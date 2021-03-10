---
title: "¿Cómo obtener datos de nuestro SQL?"
excerpt_separator: "<!--more-->"
classes: wide
categories:
  - Blog
  - SQL
  - Azure
tags:
  - Post Formats
  - readability
  - standard
---

![visitors](https://visitor-badge.glitch.me/badge?page_id=includewareok.blog.2021-03-08-get-azuresql")

Hace un tiempo escribí como actualizar las reglas del firewall del Azure SQL en [el blog de Victor](https://blog.victorsilva.com.uy/powershell-firewall-azure-sql/). En esta ocación vamos a ver como obtener algunos datos más interesantes.

<!--more-->
Uno de los desafios a nivel de SQL en particular pero de toda infra en general es la capacidad de preever problemas de capacidad. Para poder hacer eso primero necesitamos saber donde estamos parando.

Lo primero que debemos hacer es instalar el modulo de azure 
```powershell
Import-Module az
```

Una vez importado vamos a crear nuestro objeto credenciales y pasarlo al comando `Connect-AzAccount`

```powershell
$secpasswd = ConvertTo-SecureString "MyPassword" -AsPlainText -Force
$mycreds = New-Object System.Management.Automation.PSCredential ("username", $secpasswd)

Connect-AzAccount -Credential $mycreds
```

Una vez conectado debemos elegir la subscripción donde se encuentra la base de datos y cargar los valores de `resource group`, `nombre servidor` y `nombre de la base de datos`

```powershell
$subId = "MySubId"
Select-AzSubscription -SubscriptionId $subId

$dbName = "DatabaseName"
$serverName = "ServerName"
$resourceGroupName = "resourceGroupName"
```

Una vez que tenemos esto definido podemos ir a obtener nuestros datos, pero antes vamos a crear una [propiedad calculada](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.utility/select-object?view=powershell-7.1#example-10--create-calculated-properties-for-each-inputobject) para pasar de bytes a GB:
```powershell
$size = @{label="Size(GB)";expression={$_.MaxSizeBytes/1GB}}
Get-AzSqlDatabase -ServerName $serverName -DatabaseName $dbName -ResourceGroupName $resourceGroupName | select Capacity, Edition, $size, MaxSizeBytes
```
![output](/assets/images/2021-03-08-get-azuresql-1.png)

En este caso es una base de datos de ejemplo con 5DTU y 2 GB de tamaño máximo. Lamentablemente al momento de escribir este manual, no hay una forma facil de obtener cuanto de eso está usado sin tener que consultar al SQL mismo