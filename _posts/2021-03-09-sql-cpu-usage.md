---
title: "Como SQL Server gestiona el CPU"
excerpt_separator: "<!--more-->"
modified: 2021-03-09
categories:
  - Blog
  - SQL
  - internals
tags:
  - Post Formats
  - readability
  - standard
---

En el campo de la computación existe la llamada jerarquias de memorias. Los tipos de memoria más rapido son los registros pero son muy costosos. Por eso se deben gestionar de la forma más optima:

Esto junto a la capacidad de computo, hace que el CPU sea el recurso más valioso y rápido y por lo tanto el que se debe aprovechas al máximo. Desde la introducción de los sistemas multi proceso y multi usuario, los SO deben organizar de la forma más efectiva posible el uso de ellos. Debido a que SQL Server es un RDBMS, uno de sus componentes se encarga de eso. Se llama internamente SQL Scheduler.

**Changes in Service:** Para este simplificar el ejemplo vamos a suponer un servidor que solo cuenta con un solo core y un solo CPU.
{: .notice}


Internamente el ciclo de vida de una consulta en SQL Server tiene 3 estados posibles:
* Running
* Suspended
* Runnable

Una consulta solo puede estar en el estado Running mientras tienga trabajo que pueda ejecutar, en el momento en que la misma tiene que esperar por un recurso (páginas, locks, red, etc) que no están disponibles, la consulta pasa al estado Suspended. La consulta tiene que esperar en este estado hasta que el recurso solicitado este disponible. Luego de esto la consulta es movida al estado Runnable. La consulta se va a quedar en este estado mientras espera que un núcleo este libre y finalmente la consulta vuelve al estado Running. Todas estos cambios de estados son registrados y monitoreados mediante los waits stats internamente.

**Info Notice:** Como SQL Server está contruido con un gestión de procesos llamado [Cooperative Scheduling](https://www.sqlpassion.at/archive/2015/04/13/introduction-to-wait-statistics-in-sql-server/) (SQL Server bypassea el agendado preventivo del SO y agenda sus hilos el mismo) y tiene definido su Quantum (tiempo máximo de una consulta en un procesador,  en un procesador de 2Ghz son 1 x 10 a las 6 instrucciones por  milisegundo) en 4ms. Después de ese tiempo, la consulta cede voluntariamente el CPU y se cambia al estado Runnable. Este tipo de comportamientos es registrado por SQL Server como un tipo de espera SOS_SCHEDULER. Por más información se puede [leer este](https://sqlperformance.com/2014/02/sql-performance/knee-jerk-waits-sos-scheduler-yield) o [este articulo](https://www.sqlskills.com/help/waits/sos_scheduler_yield/) de Paul Randall.
{: .notice--info}


![Imagen-002](/assets/images/como-sql-gestiona-el-cpu-002.png)

Vamos a ver un ejemplo práctico. Supongamos que llega la consulta de Felipe:

![Imagen-003](/assets/images/como-sql-gestiona-el-cpu-003.png)

Mientas see ejecuta esa, llega la consutla de Mauro y de Damian:

![Imagen-004](/assets/images/como-sql-gestiona-el-cpu-004.png)


Termina la consulta de Felipe y se le da lugar a la consulta de Mauro:

![Imagen-005](/assets/images/como-sql-gestiona-el-cpu-005.png)

La consulta de Mauro Necesita un recurso y se pasa a esperar:

![Imagen-006](/assets/images/como-sql-gestiona-el-cpu-006.png)

**Info Notice:** La consulta de Mauro genero un aumento en las estadisticas de espera waits statitics. SQL Server puede esperar por diferentes tipos de cosas como son
* Recursos: CPU, memoria, almacenanmiento, red, bloqueos, latches
* Cosas fuer de SQL Server: OLEDB, Objetos COM, CLR
* Tareas del sistema: Lazy writer, trace, full text search
{: .notice--info}


Llegan otras consultas y la consulta de Mauro obtiene el recurso y vuelve a la lista de runnable:

![Imagen-007](/assets/images/como-sql-gestiona-el-cpu-007.png)
