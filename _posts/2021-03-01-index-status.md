---
title: "Analizando el estado de los índices"
excerpt_separator: "<!--more-->"
modified: 2021-03-09
permalink: /index-status
classes: wide
categories:
  - Blog
  - SQL
  - SQL Server
  - internals
tags:
  - Post Formats
  - readability
  - standard
---

![visitors](https://visitor-badge.glitch.me/badge?page_id=includewareok.blog.2021-03-01-index-status")

## Introducción
Los indices juegan una parte importante de un sistema, impacatando directamente en su performance. En esta sección vamos a analizar el estado de los mismos y sugerir posibles soluciones a problemas comunes. Como comentaria vamos solo a analizar los tipos CLUSTERED INDEX y NONCLUSTERED INDEX
<!--more-->

**:information_source:** Cada vez que creamos una tabla y no aclaramos una Primary Key (PK) o algun indice CLUSTED, la tabla a nivel de SQL Server se crea de tipo HEAP (pila en español), esos datos tienen el orden como si fuera un STACK, en primer dato ingresado es la base y se van “apilando”, esto tiene la ventaja de que es super performance para insert y puede ser una buena idea para tablas que no son acutalizadas y/o consultadas frecuentemente y con bajo volumen de datos.
{: .notice}


Vamos a trabajar en 2 arístas del problema por un lado la no existencia de indices CLUSTERED y la otra es el estado del mismo ya que un indice en mal estado puede igual de malo que no tener ningúno.

Cada base de datos de SQL Server cuenta con varias tablas internas del sistema, vamos a usar en este caso 2: 

* sys.indexes, donde se almacena la información de los indices que existen en cada 
* sys.objects: Están todos los objetos de la base de datos, tablas de usuario y sistema, PK, vistas, Store Procedure, constraint, etc.

Vamos a ver los tipos de indices de una base de datos, para eso ejecutamos la siguiente query:

``` sql
USE  NOMBREDELADB --Cambiar por la DB que se quiera comprobar
GO;
SELECT DISTINCT(type_desc) 
FROM sys.indexes i
```

El resultado es el siguente:

|type_desc|
|-|
|CLUSTERED|
|HEAP|
|NONCLUSTERED|

## Identificar si existe indices en las tablas

**:information_source:** La estructura de los indices SQL Server son B+ Tree (árboles binarios de búqueda), la diferencia sustancial entre los CLUSTERED y NONCLUSTERED es que el nivel más bajo, leaf level, en los CLUSTERED es el dato en si, mientras que en los NONCLUSTERED es un puntero al dato.
{: .notice}

Cuando una tabla no cuenta con un indice CLUSTERED los datos están dispersos de tal forma que consultas o actualizaciones al mismo son poco performantes. Por contra posición el indice CLUSTERED puede hacer que los INSERT sean mas lentos. 

![Imagen-001](/assets/images/2021-03-01-index-status-001.png)
[Imagen obtenida de la documentación de MS](https://docs.microsoft.com/en-us/sql/relational-databases/sql-server-index-design-guide?view=sql-server-ver15#clustered-index-architecture)

``` sql
--Obotener los nombres de las tablas que no tiene indices clustered
USE  NOMBREDELADB --Cambiar por la DB que se quiera comprobar
GO;
SELECT o.name, o.object_id
FROM sys.objects o
WHERE o.type='U'
  AND NOT EXISTS(SELECT 1 FROM sys.indexes i
                 WHERE o.object_id = i.object_id
                   AND i.type_desc = 'CLUSTERED')
```

Esto nos devolverá una lista con todas las tablas que no cuentan con un indice CLUSTERED y sirve para pasarle al equipo de desarrollo y validar si es por diseño que se optó por esa opción o no. 


## Obtener el estado de los indices
El tema de los indices al ser estructuras lógicas de árboles binarios de búsqueda y la forma de agrupación de datos en disco (páginas de 8kb) de SQL Server, cada vez que se ingresan datos se puede proceder a un page split, sumado a esto si los datos no están contiguos se va generando una degradación del indice que se llama fragementación.

Con este [script de Glen Barry](https://www.sqlskills.com/blogs/glenn/category/dmv-queries/) podemos obtener el estado de  fragemtanción los indices y validar que tan correcta es nuestra estrategía de mantenimiento de indices:

``` sql
USE  NOMBREDELADB --Cambiar por la DB que se quiera comprobar
GO;
SELECT DB_NAME(ps.database_id) AS [Database Name], SCHEMA_NAME(o.[schema_id]) AS [Schema Name],
OBJECT_NAME(ps.OBJECT_ID) AS [Object Name], 
i.name AS [Index Name], ps.index_id, ps.index_type_desc, ps.avg_fragmentation_in_percent, 
ps.fragment_count, ps.page_count, i.fill_factor, i.has_filter, i.filter_definition, i.allow_page_locks
FROM sys.dm_db_index_physical_stats(DB_ID(),NULL, NULL, NULL , N'LIMITED') AS ps
INNER JOIN sys.indexes AS i WITH (NOLOCK)
ON ps.[object_id] = i.[object_id] 
AND ps.index_id = i.index_id
INNER JOIN sys.objects AS o WITH (NOLOCK)
ON i.[object_id] = o.[object_id]
WHERE ps.database_id = DB_ID()
AND ps.page_count > 2500
ORDER BY ps.avg_fragmentation_in_percent DESC OPTION (RECOMPILE)
```

![Imagen-002](/assets/images/2021-03-01-index-status-002.png)

Del mundo de datos a fijarnos los que más interesan son:
* Schema Name, Objet Name, Index Name para identiifcar el objeto en particular de forma rapida.
* El tipo para saber que tipo de indice es. Por ejemplo en el caso de los HEAP no es posible corregir la fragemtanción, es necesarió crear un CLUSTERED index para solucionar el problema.
* El % de fragmetación del indice, como regla general 0 a 10% no hay mucho problema, de 11 a 35 % puede verse degradada la perofmrnace y se sugiere hacer un rebuild del mismo, más de 35 % el indice no es efectivo y es necesario hacer un rebuild o reorganize del mismo.
* page_count y fragmentation_count nos dicen de que volumen de datos estamos hablando, por ejemplo:
  * para el primer indice load_items_co.., que es una tabla HEAP,el tamaño de a tabla es de 590MB y en caso de hacer consultas en esta tabla (SELECT, UPDATE, DELETE) SQL Server a a necesitar todos los 590MB desde disco (generando I/Os) para poder realizar la consulta en cuestión
  * Para el índice load_item_au…, de tipo NONCLUSTERED, vemos que el tamaño es de 185MB
* Otro dato interesante de ver el fill factor, básicamente cuando uno define un indice puede decir cuanto quiero llenar cada página de datos, en este caso en particular, un fill factor de 95 significa que se va a dejar libre un 5% de la misma para nuevos datos, y reducir los nivel de [page split]({% post_url 2021-03-09-page-split %}).


Para corregir la fragmentación hay 3 posibles caminos
* Hacer clic derecho desde el Managment studio en el indice en particular y optar por la opción de rebuild o recreate, el problema de esto es que es un proceso a demanda y si son muchos los indices/tablas puede llevar bastante tiempo: 
![Imagen-003](/assets/images/2021-03-01-index-status-003.png)
* Crear un plan de mantenimiento desde el Managment studio usando los objetos nativos de SQL Server, la desventaja de este camino es que  no tiene inteligencia, no usa el estado de los indices y en general va por todos o los que se le especifique, haciendo que sea o muy intensivo a nivel del sistema o tener que estar constantemente adaptandolo para tener en cuenta los nuevos indices con altos indices de fragmentación.
* Usar el [scipt de Ola Hallengren](https://ola.hallengren.com/) para hacer un mantenimiento inteligente, por ejemplo:

``` sql
EXECUTE dbo.IndexOptimize
@Databases = 'USER_DATABASES',
@FragmentationLow = NULL,
@FragmentationMedium = 'INDEX_REORGANIZE,INDEX_REBUILD_ONLINE,INDEX_REBUILD_OFFLINE',
@FragmentationHigh = 'INDEX_REBUILD_ONLINE,INDEX_REBUILD_OFFLINE',
@FragmentationLevel1 = 5,
@FragmentationLevel2 = 30
```