---
title: "En tiempos de pandemia, tus datos también deberían usar máscara"
excerpt_separator: "<!--more-->"
modified: 2021-04-13
classes: wide
comments: true
categories:
  - "security"
tags:
  - "security"
  - "data masking"
  - database
---

![visitors](https://visitor-badge.glitch.me/badge?page_id=includewareok.blog.2021-04-13-seguridad")

## Introducción

Con la llegada de la pandemia en 2020 muchas cosas cambiaron para todos, la gran mayoría empezó a trabajar desde su casa, aprendimos nuevos términos como `burbuja social`, `aplanar la curva`. El nuevo accesorio de todo el mundo paso a ser una máscara facial para protegernos y proteger a los demás, también dispositivo como caretas o ropa descartable y volvimos a lavarnos las manos y lavar los productos cada vez que volvemos a nuestro hogares. Lo que no cambio, es más, empeoró son los ciber ataques. Suplantación de identidad, robo de datos sensibles por nombrar algunos. Entonces cabe plantearnos esta pregunta, ¿No será necesario que nuestros datos también uses sistemas para protegerse? ¿Cúales son? Y, por último, ¿Por qué no enmascarar nuestros datos? 
<!--more-->

## El nuevo petróleo

Hace un tiempo que escucho el tema de que los datos con el petróleo del siglo XXI, si ese es el caso. El paralelismo nace del hecho que a principios del siglo XX, el petroleo revolucionó el mundo, el surgimiento de aviones, la transformación del carbón al fueloil o gasoil en el medio marítimo. Eso sin hablar del plástico que también es un derivado de él. Tal fue el impacto de este oro negro, que guerras a lo largo y ancho del planeta, junto con economías que se lograron posicionar en la cima debido es este recurso. 

Por lo valioso del recurso en sí, es que nadie deja a su voluntad una cañería de petróleo para que pueda ser saboteada, ya sea contaminado su contenido  o sea robado. Por otro lado, ¿Ustedes se imaginan un operario de una refinería al norte de Canadá, fumando mientras arregla una fuga en una tubería?. Claro que no, de la misma forma que tampoco nos imaginamos que Venezuela hagan una fiesta con fuegos artificiales encima de una refinería con cientos o miles de tanques llenos de crudo.

 ![Casa en llamas](https://www.ecestaticos.com/image/clipping/557/418/af89b92e145ca0c0b3ddbbd2abbd7e2b/la-protagonista-del-meme-de-disaster-girl-explica-como-surgio-la-foto-del-incendio.jpg)

## Tipos de seguridad

De igual forma que en los casos anteriores, no podemos dejar nuestros datos flotando en el éter de la red de redes liberados a su suerte. Tenemos entonces que preocuparnos de los datos en sus 2 estados. Cuando está  **viajando en una tubería** se llama _security at transit_ y cuando está en un **depositada en un tanque** se llama _security at rest_.

### Securiry at rest

Se entiende por **security at rest** todos datos que se encuentran en un lugar. Esto pueden ser bien los archivos de transacciones de una base de datos, un almacén de datos (`lago de datos` o simples carpetas tipo `storage account` o `S3`) , las carpetas bien pueden tener archivos con datos en crudo de un [proceso ETL]({% post_url 2021-03-22-etl %}) o los respaldos de un motor. 

Para este tipo de problemas, en general nos enfocamos en `RBAC` o `ACLs` para dar o quitar permisos para las carpetas de respaldos o de datos intermedios. Para los archivos de la base de datos, se utiliza **TDE**, por sus siglas en inlgés que significa _Transparent Data Encryption_ y en el caso de [SQL Server](https://docs.microsoft.com/en-us/sql/relational-databases/security/encryption/transparent-data-encryption?view=sql-server-ver15) el cifrado se hace a nivel de las páginas de datos, esto también se puede aplicar a nivel de respaldos y lo que debemos proteger es el certificado con el cual hicimos el cifrado para poder hacer una restauración de dicho respaldo. Esto que parece bastante simple, pero por ejemplo para otro motor famoso como [PostgrSQL](https://www.highgo.ca/2020/12/14/rise-and-fall-for-an-expected-feature-in-postgresql-transparent-data-encryption/) es algo que siguen trabajando porque no hay un acuerdo común entre los principales lideres de dicha comunidad.

### Securiry at transit

De manera similar, se entiende como `security at transit` cuando los datos están viajando de un servidor a otro, acá puede ser bien que los datos están yendo desde el motor al frontend que está procesando un request en una API o bien cuando el dato viaje desde un sitio web hasta el equipo del cliente final. 

Este tema es un poco más conocido ya que no solo en el mundo de las bases de datos se ven estos problemas. Hasta hace no tanto, era bastante común ver sitios web sin uso de certificados y por lo tanto susceptibles a que alguien en la misma red pudiera ver la información que viaja en la comunicación entre el cliente y el servodr o ataques de **MITM**, por sus siglas en inglés _Man in the Middle_. Pero va un poco más, en este punto podemos estar hablando de VPN site-to-site o site-to-client para proteger el tráfico, también podemos implementar **TLS**,  _Transport Layer Security_ sobre la conexión entre la API y el servidor de base de datos. [SQL Server soporta desde hace un tiempo](https://docs.microsoft.com/en-us/sql/database-engine/configure-windows/enable-encrypted-connections-to-the-database-engine?view=sql-server-ver15) **TLS** desde su versión 2012.


### Data masking

El enmascarado de datos es una técnica que se usa bastante en la ciencia de datos, sobre todo en la etapa anonimización de datos, pero no exclusivamente. Existen 2 tipos de ofuscado de datos:
* Estático: Es el más simple e implica cambiar el dato final por uno nuevo. Por ejemplo cambiar el correo de todos los usuarios por _algo@correo.com_, el problema es que no tiene vuelta atras y para obtener el dato real debemos recurrir a la fuente.
* Dinámico: Es cuando el datos está en su forma original y dependiendo de si quien lo accede tiene permisos o no, puede ver el valor real o una _mascara_, esto se puede hacer bien mediante vistas o algunos motores como [SQL Server implemetan DDM](https://docs.microsoft.com/en-us/sql/relational-databases/security/dynamic-data-masking?view=sql-server-ver15), lo bueno que se pueden definir máscaras de forma dinámica por ejemplo si tengo un documentado de identidad que su valor es 1.234.567-8, puedo aplicarle una máscara que me muestre 1.XXX.YYY-8 un valor fijo o un valor al azar dentro de un rango. Nos permite acceder al dato real si contamos con el permiso.

```sql
--Dato real
select email, edad
from dbo.usuarios

--Dato enmascarado con una vista
CREATE VIEW vista_usuarios AS
select "algo@corre.com", edad
from dbo.usuarios
```

### Always Encrypted

Por último me gusta mucho el feature de Microsoft SQL Server llamad [Always Encrypted](https://docs.microsoft.com/en-us/sql/relational-databases/security/encryption/always-encrypted-database-engine?view=sql-server-ver15#:~:text=Always%20Encrypted%20allows%20clients%20to,SQL%20Database%20or%20SQL%20Server) que nos permite que cifrar datos y los mismos están protegidos hasta que son procesados por el motor mismo. Definir una tabla es un poco diferente porque debemos cambiar la collation de la columna a BIN2 y además decir que algoritmo vamos a usar y que tipo:
```sql
CREATE TABLE Customers (  
    CustName nvarchar(60)   
        COLLATE  Latin1_General_BIN2 ENCRYPTED WITH (COLUMN_ENCRYPTION_KEY = MyCEK,  
        ENCRYPTION_TYPE = RANDOMIZED,  
        ALGORITHM = 'AEAD_AES_256_CBC_HMAC_SHA_256'),   
    SSN varchar(11)   
        COLLATE  Latin1_General_BIN2 ENCRYPTED WITH (COLUMN_ENCRYPTION_KEY = MyCEK,  
        ENCRYPTION_TYPE = DETERMINISTIC ,  
        ALGORITHM = 'AEAD_AES_256_CBC_HMAC_SHA_256'),   
    Age int NULL  
);  
```
[Obtenido de la documentación de MS](https://docs.microsoft.com/en-us/sql/relational-databases/security/encryption/always-encrypted-database-engine?view=sql-server-ver15#example)

## CIA en seguridad

La base de la seguridad informática como la conocemos está formada por el triángulo CIA. Este es una forma de modelar la seguridad que nos permite ver diferentes dimensiones del problema. La sigla viene del inglés por `Confidentiality`, `Integrity` y  `Availability`:

* _Confidencialidad_: Es la capacidad de un sistema o programa de solo permitir acceso a aquellos usuarios que corresponda.
* _Integridad_: Los datos son consistentes y solo son modificados de acuerdo a lo esperado por el sistema, acá sin embargo está el problema de posibles bugs pero no nos enfoquemos en esto ahora. 
* _Disponibilidad_: Cualquier usuario, en cualquier momento puede acceder a los datos cuando los necesite.

El mayor problema relacionado a la seguridad en las bases de datos es que siempre pensamos en el sistema activo. Lo bueno de este modelo es que podemos abordar los diferentes desafíos o partes del sistema desde diferentes ángulos. Podemos entonces dejar de ver un respaldo como algo sin problemas y pensar en la **Confidencialidad** ya que puede quedar comprometida si nuestro respaldo está en una carpeta que todos tiene permiso de escritura, o si al intentar hacer una recuperación alguien lo borro vemos como nuestra  solución puede tener problemas de **Disponibilidad**. 

## Conclusiones

La seguridad no es algo que debe ser tomado de forma trivial y debe ser visto desde varios ángulos. Es importante construir sistemas que cuenten con varias capaz para asegurar el cumplimiento de la tríada **CIA**. Es importante por lo tanto tener en cuenta todo el ciclo de vida de nuestros datos y no solo los que se encuentran dentro de una base de datos. Ningún sistema es infalible, pero adoptando estas medidas, estamos reduciendo los vectores de ataque, desincentivando a usuarios maliciosos o hackers con menor nivel de conocimiento técnico. 