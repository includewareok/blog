---
title: Como automatizar firmas de correo en Office365
permalink: /firma-corre
modified: 2021-03-10
comments: true
categories:
  - Blog
  - Office365
  - correo
  - automatizar
---


# Introducción

Uno de los mayores problemas que enfrentan las empresas es la estandarización de su marca. A quien no le ha pasado que, intercambiando correos con varios empleados de una misma empresa, todos cuentan con firmas diferentes o incluso algunos ni siquiera cuentan con una firma. Ahora bien, parece un tema menor pero no lo es tanto. Hace unos meses estoy trabajando para una empresa internacional muy grande. Solo el equipo para el que trabajo tiene a su cargo 250 clientes distribuidos por el mundo. Además, intercambiamos información, soluciones técnicas y trabajamos en conjunto con otras áreas internas de la empresa. En este punto es que la firma toma un valor importante porque da a mi entender nos brinda 3 datos fundamentales para entender quien es nuestro interlocutor:
* Nombre completo
* Equipo y rol al que pertenezco
* Ubicación (dirección, país, teléfono)

Esto permite al receptor poder armar un mapa mental de quien lo está contactando y cuáles son sus intereses y responsabilidades con respecto al tema que se está tratando en el correo.

Ahora bien, como hago esto de forma que la tarea no quede en manos de un copy&paste de los nuevos empleados, como actualizo de forma centralizada mi firma para conmemorar eventos especiales como puede ser los 10 años de la empresa, Parthner of the year, best place to work, LinekdIn empresarial, etc

Office 365 y en particular Exchange online tienen una solución para este problema. Tenemos la posibilidad de agregar al final (append es la palabra en inglés) lo que queramos, en general esto es usado en bancos, empresas de seguros y demás para agregar el típico mensaje de seguridad:

**:information_source:** The information in this email is confidential and may be legally privileged. It is intended solely for the addressee. Any opinions expressed are mine and do not necessarily represent the opinions of the Company. Emails are susceptible to interference. If you are not the intended recipient, any disclosure, copying, distribution or any action taken or omitted to be taken in reliance on it, is strictly prohibited and may be unlawful. If you have received this message in error, do not open any attachments but please notify
{: .notice--info}

Pero nosotros lo vamos a usar para crear de forma automática nuestra firma. Primero tenemos que ir a Exchange online y acceder a la sección de [message Flow](https://outlook.office365.com/ecp/?rfr=Admin_o365&exsvurl=1&mkt=en-US)

**⚠️** Para poder realziar esta tarea es necesario contar con permisos de Exchange Admin.
{: .notice}

![Imagen-001](/assets/images/2021-03/001.png)

Una vez en tab de reglas vamos a crear una nueva regla:

![Imagen-002](/assets/images/2021-03/002.png)

En nuestro caso vamos a filtrar solo por mi usuario, pero podríamos ir por el emisor está dentro de mi organización (todos mis empleados)


![Imagen-003](/assets/images/2021-03/003.png)


Por último tenemos que aclarar cual es la acción que queremos que se ejecute una vez cumplida la condición, en nuestro caso es “append the disclaimer”

![Imagen-004](/assets/images/2021-03/004.png)

Primero le ponemos un nombre para identificar, ya que esto es una prueba le vamos a poner **Prueba Firma Felipe**, después tenemos que elegir el criterio de aplicación. Existen un montón de posibles criterios posibles:

![Imagen-005](/assets/images/2021-03/005.png)

En nuestro caso vamos a filtrar solo por mi usuario, pero podríamos ir por el emisor está dentro de mi organización (todos mis empleados)

![Imagen-006](/assets/images/2021-03/006.png)

Por último tenemos que aclarar cual es la acción que queremos que se ejecute una vez cumplida la condición, en nuestro caso es **append the disclaimer**

![Imagen-007](/assets/images/2021-03/007.png)

Los correos de Office365 son por defecto de contenido HTML, por lo tanto, tenemos que crear un objeto (en nuestro caso una tabla) que contenga los datos de la firma. Un dato interesante que me encontré Armando esta POC es que existen un set de variables relacionadas con el usuario que está enviado el correo. Por lo tanto, podemos acceder a datos de dicha persona como por ejemplo %%DisplayName%%, todos estos datos son ni más ni menos que atributos del Azure AD, así que si tenemos bien configurados nuestros usuarios en el AD, podemos fácilmente tener una firma linda. Acá les paso un ejemplo con una tabla para mostrar mi firma. La misma en los correos salientes se ve así (no tuve tiempo de jugar con el tamaño de la foto ya que no soy un fanático de HTML:

![Imagen-008](/assets/images/2021-03/008.png)


```html
<table border="0" cellspacing="0" cellpadding="0" style="border-collapse:collapse;margin-left:5.4pt;">
   <tbody>
      <tr>
         <td rowspan="3" style="height:50px;width:101px;padding:0 5.4pt;"><img data-imagetype="External" src="https://yetanotherbloginthehood.files.wordpress.com/2019/10/firma.jpg" height=97 width=313 alt="https://yetanotherbloginthehood.files.wordpress.com/2019/10/firma.jpg"><br>
         </td>
         <td valign="top" style="width:251.35pt;padding:0 5.4pt;">
            <p style="margin-top:0;margin-bottom:0;"><strong><span style="color:#1F497D;">%%DisplayName%%</span></strong></p>
            <p style="margin-top:0;margin-bottom:0;"><span style="color:#1F497D;"><span style="color: rgb(31, 73, 125); font-size: 10pt; font-family: Calibri, Arial, Helvetica, sans-serif, serif, EmojiFont; text-align: center;">%%Title%%</span><br>
               </span>
            </p>
         </td>
      </tr>
      <tr>
         <td valign="top" style="width:251.35pt;padding:0 5.4pt;">
            <p style="margin-top:0;margin-bottom:0;"><span style="color:#1F497D;font-size:10pt;"></span><span style="color:#1F497D;font-size:10pt;">%%StreetAddress%%, %%City%% %%Country%%</span></p>
            <p style="margin-top:0;margin-bottom:0;"></p>
         </td>
      </tr>
      <tr>
         <td valign="top" style="width:251.35pt;padding:0 5.4pt;">
            <p style="margin-top:0;margin-bottom:0;"><span style="color:#1F497D;font-size:10pt;">%%Phone%% Ext. %%Office%% <br>
               </span>
            </p>
            <p style="margin-top:0;margin-bottom:0;"><span style="color:#1F497D;font-size:10pt;"><a href="mailto:%%Email%%" target="_blank" rel="noopener noreferrer" data-auth="NotApplicable"><span style="color:#0563C1;">%%Email%%</span></a>
               |&nbsp;</span><span style="color:#0563C1;font-size:10pt;">www.pyxis.com.uy</span>
            </p>
         </td>
      </tr>
   </tbody>
</table>
```

Demoré tanto tiempo (creo que tengo la firma hace 3 años creada) en escribir este post que ahora MS tiene una [página donde explica esto mismo](https://docs.microsoft.com/en-us/exchange/policy-and-compliance/mail-flow-rules/signatures?view=exchserver-2019)


La Buena noticia, Podemos hacer esto mismo desde [powershell](https://docs.microsoft.com/en-us/exchange/security-and-compliance/mail-flow-rules/disclaimers-signatures-footers-or-headers)

```powershell
New-TransportRule -Name "Prueba 2 Felipe" -SentToScope NotInOrganization -ApplyHtmlDisclaimerText "<h3>Esto es una prueba</h3><p>Este es el texto de la <a href=’https://blog.victorsilva.com.uy/’>un gran blog</a></p><img alt='Contoso logo' src='http://www.contoso.com/images/logo.gif'>"
```