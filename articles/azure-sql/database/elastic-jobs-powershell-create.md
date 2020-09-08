---
title: Creación de un agente de trabajos elásticos mediante PowerShell
description: Aprenda a crear un agente de trabajos elásticos mediante PowerShell.
services: sql-database
ms.service: sql-database
ms.subservice: scale-out
ms.custom: seo-lt-2019, sqldbrb=1, devx-track-azurepowershell
ms.devlang: ''
ms.topic: tutorial
author: johnpaulkee
ms.author: joke
ms.reviwer: sstein
ms.date: 03/13/2019
ms.openlocfilehash: 0fe6a3b1d0e5216d923cfe3997826e7a727f1a2b
ms.sourcegitcommit: 656c0c38cf550327a9ee10cc936029378bc7b5a2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89077354"
---
# <a name="create-an-elastic-job-agent-using-powershell"></a>Creación de un agente de trabajos elásticos mediante PowerShell
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

Los [trabajos elásticos (versión preliminar)](job-automation-overview.md#elastic-database-jobs-preview) permiten la ejecución de uno o más scripts de Transact-SQL (T-SQL) en paralelo en varias bases de datos.

En este tutorial aprenderá los pasos necesarios para ejecutar una consulta en varias bases de datos:

> [!div class="checklist"]
> * Creación de un agente de trabajos elásticos
> * Creación de credenciales de trabajo para que los trabajos puedan ejecutar scripts en sus destinos
> * Definición de los destinos (servidores, grupos elásticos, bases de datos, mapas de particiones) en los que desea ejecutar el trabajo
> * Creación de credenciales de ámbito de base de datos en las bases de datos de destino para que el agente pueda conectar y ejecutar trabajos
> * Creación de un trabajo
> * Incorporación de pasos de trabajo a un trabajo
> * Inicio de la ejecución de un trabajo
> * Supervisión de un trabajo

## <a name="prerequisites"></a>Requisitos previos

La versión actualizada de los trabajos de Elastic Database tiene un nuevo conjunto de cmdlets de PowerShell para su utilización durante la migración. Estos nuevos cmdlets transfieren todas las credenciales de trabajos, destinos (incluidas bases de datos, servidores y colecciones personalizadas), desencadenadores de trabajos, programaciones de trabajos, contenido de trabajos y trabajos existentes a un nuevo agente de trabajos elásticos.

### <a name="install-the-latest-elastic-jobs-cmdlets"></a>Instalación de los últimos cmdlets de trabajos elásticos

Si no tiene una suscripción de Azure, [cree una cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

Instale el módulo **Az.Sql** para obtener los cmdlets de trabajos elásticos más recientes. Ejecute los comandos siguientes en PowerShell con acceso administrativo.

```powershell
# installs the latest PackageManagement and PowerShellGet packages
Find-Package PackageManagement | Install-Package -Force
Find-Package PowerShellGet | Install-Package -Force

# Restart your powershell session with administrative access

# Install and import the Az.Sql module, then confirm
Install-Module -Name Az.Sql
Import-Module Az.Sql

Get-Module Az.Sql
```

Además del módulo **Az.Sql**, este tutorial también requiere el módulo *SqlServer* de PowerShell. Para más información, consulte [Instalación de un módulo de SQL Server PowerShell](/sql/powershell/download-sql-server-ps-module).

## <a name="create-required-resources"></a>Creación de los recursos necesarios

La creación de un agente de trabajos elásticos requiere una base de datos (S0 o superior) para usarla como [base de datos de trabajos](job-automation-overview.md#job-database).

El script siguiente crea un grupo de recursos, un servidor y una base de datos que se usará como base de datos de trabajos. El segundo script crea un segundo servidor con dos bases de datos en blanco en las que ejecutar los trabajos.

Los trabajos elásticos no tienen ningún requisito de nomenclatura específico, por lo que puede usar las convenciones de nomenclatura que desee, siempre y cuando cumplan con los [requisitos de Azure](/azure/architecture/best-practices/resource-naming).

```powershell
# sign in to Azure account
Connect-AzAccount

# create a resource group
Write-Output "Creating a resource group..."
$resourceGroupName = Read-Host "Please enter a resource group name"
$location = Read-Host "Please enter an Azure Region"
$rg = New-AzResourceGroup -Name $resourceGroupName -Location $location
$rg

# create a server
Write-Output "Creating a server..."
$agentServerName = Read-Host "Please enter an agent server name"
$agentServerName = $agentServerName + "-" + [guid]::NewGuid()
$adminLogin = Read-Host "Please enter the server admin name"
$adminPassword = Read-Host "Please enter the server admin password"
$adminPasswordSecure = ConvertTo-SecureString -String $AdminPassword -AsPlainText -Force
$adminCred = New-Object -TypeName "System.Management.Automation.PSCredential" -ArgumentList $adminLogin, $adminPasswordSecure
$agentServer = New-AzSqlServer -ResourceGroupName $resourceGroupName -Location $location `
    -ServerName $agentServerName -ServerVersion "12.0" -SqlAdministratorCredentials ($adminCred)

# set server firewall rules to allow all Azure IPs
Write-Output "Creating a server firewall rule..."
$agentServer | New-AzSqlServerFirewallRule -AllowAllAzureIPs
$agentServer

# create the job database
Write-Output "Creating a blank database to be used as the Job Database..."
$jobDatabaseName = "JobDatabase"
$jobDatabase = New-AzSqlDatabase -ResourceGroupName $resourceGroupName -ServerName $agentServerName -DatabaseName $jobDatabaseName -RequestedServiceObjectiveName "S0"
$jobDatabase
```

```powershell
# create a target server and sample databases - uses the same credentials
Write-Output "Creating target server..."
$targetServerName = Read-Host "Please enter a target server name"
$targetServerName = $targetServerName + "-" + [guid]::NewGuid()
$targetServer = New-AzSqlServer -ResourceGroupName $resourceGroupName -Location $location `
    -ServerName $targetServerName -ServerVersion "12.0" -SqlAdministratorCredentials ($adminCred)

# set target server firewall rules to allow all Azure IPs
$targetServer | New-AzSqlServerFirewallRule -AllowAllAzureIPs
$targetServer | New-AzSqlServerFirewallRule -StartIpAddress 0.0.0.0 -EndIpAddress 255.255.255.255 -FirewallRuleName AllowAll
$targetServer

# create sample databases to execute jobs against
$db1 = New-AzSqlDatabase -ResourceGroupName $resourceGroupName -ServerName $targetServerName -DatabaseName "database1"
$db1
$db2 = New-AzSqlDatabase -ResourceGroupName $resourceGroupName -ServerName $targetServerName -DatabaseName "database2"
$db2
```

## <a name="use-elastic-jobs"></a>Uso de trabajos elásticos

Para usar trabajos elásticos, registre la característica en su suscripción de Azure mediante la ejecución del siguiente comando. Ejecute este comando una vez para la suscripción en la que tiene pensado aprovisionar el agente de trabajos elásticos. No es necesario registrar las suscripciones que solo contengan bases de datos que son destinos de trabajos.

```powershell
Register-AzProviderFeature -FeatureName sqldb-JobAccounts -ProviderNamespace Microsoft.Sql
```

### <a name="create-the-elastic-job-agent"></a>Creación del agente de trabajos elásticos

Un agente de trabajos elásticos es un recurso de Azure para crear, ejecutar y administrar trabajos. El agente ejecuta los trabajos según una programación o como un trabajo único.

El cmdlet **New-AzSqlElasticJobAgent** requiere que exista ya una base de datos de Azure SQL Database, por lo que los parámetros *resourceGroupName*, *serverName* y *databaseName* deben todos apuntar a recursos ya existentes.

```powershell
Write-Output "Creating job agent..."
$agentName = Read-Host "Please enter a name for your new Elastic Job agent"
$jobAgent = $jobDatabase | New-AzSqlElasticJobAgent -Name $agentName
$jobAgent
```

### <a name="create-the-job-credentials"></a>Creación de credenciales de trabajos

Los trabajos usan credenciales de ámbito de base de datos para conectarse a las bases de datos de destino que especifica el grupo de destino después de la ejecución y ejecutar los scripts. Estas credenciales de ámbito de base de datos también se usan para conectarse a la base de datos maestra para enumerar todas las bases de datos de un servidor o un grupo elástico, cuando cualquiera de estos se utiliza como el tipo de miembro del grupo de destino.

Las credenciales de ámbito de base de datos se deben crear en la base de datos del trabajo. Todas las bases de datos de destino deben tener un inicio de sesión con permisos suficientes para que el trabajo se complete correctamente.

![Credenciales de trabajos elásticos](./media/elastic-jobs-powershell-create/job-credentials.png)

Además de las credenciales de la imagen, observe la adición de los comandos **GRANT** en el siguiente script. Estos permisos son necesarios para el script que hemos elegido para este trabajo de ejemplo. Dado que en el ejemplo se crea una tabla en las bases de datos de destino, cada una de estas base de datos necesita los permisos adecuados para ejecutarse correctamente.

Para crear las credenciales de trabajos necesarias (en la base de datos de trabajo), ejecute el script siguiente:

```powershell
# in the master database (target server)
# create the master user login, master user, and job user login
$params = @{
  'database' = 'master'
  'serverInstance' =  $targetServer.ServerName + '.database.windows.net'
  'username' = $adminLogin
  'password' = $adminPassword
  'outputSqlErrors' = $true
  'query' = "CREATE LOGIN masteruser WITH PASSWORD='password!123'"
}
Invoke-SqlCmd @params
$params.query = "CREATE USER masteruser FROM LOGIN masteruser"
Invoke-SqlCmd @params
$params.query = "CREATE LOGIN jobuser WITH PASSWORD='password!123'"
Invoke-SqlCmd @params

# for each target database
# create the jobuser from jobuser login and check permission for script execution
$targetDatabases = @( $db1.DatabaseName, $Db2.DatabaseName )
$createJobUserScript =  "CREATE USER jobuser FROM LOGIN jobuser"
$grantAlterSchemaScript = "GRANT ALTER ON SCHEMA::dbo TO jobuser"
$grantCreateScript = "GRANT CREATE TABLE TO jobuser"

$targetDatabases | % {
  $params.database = $_
  $params.query = $createJobUserScript
  Invoke-SqlCmd @params
  $params.query = $grantAlterSchemaScript
  Invoke-SqlCmd @params
  $params.query = $grantCreateScript
  Invoke-SqlCmd @params
}

# create job credential in Job database for master user
Write-Output "Creating job credentials..."
$loginPasswordSecure = (ConvertTo-SecureString -String "password!123" -AsPlainText -Force)

$masterCred = New-Object -TypeName "System.Management.Automation.PSCredential" -ArgumentList "masteruser", $loginPasswordSecure
$masterCred = $jobAgent | New-AzSqlElasticJobCredential -Name "masteruser" -Credential $masterCred

$jobCred = New-Object -TypeName "System.Management.Automation.PSCredential" -ArgumentList "jobuser", $loginPasswordSecure
$jobCred = $jobAgent | New-AzSqlElasticJobCredential -Name "jobuser" -Credential $jobCred
```

### <a name="define-the-target-databases-to-run-the-job-against"></a>Definición de las bases de datos de destino en las que se ejecutan los trabajos

Un [grupo de destino](job-automation-overview.md#target-group) define el conjunto de una o varias bases de datos en las que se ejecutará un paso de un trabajo.

El siguiente fragmento de código crea dos grupos de destino: *serverGroup* y *serverGroupExcludingDb2*. *serverGroup* tiene como destino todas las bases de datos que existen en el servidor en el momento de la ejecución, mientras que el destino de *serverGroupExcludingDb2* son todas las bases de datos del servidor, excepto *TargetDb2*:

```powershell
Write-Output "Creating test target groups..."
# create ServerGroup target group
$serverGroup = $jobAgent | New-AzSqlElasticJobTargetGroup -Name 'ServerGroup'
$serverGroup | Add-AzSqlElasticJobTarget -ServerName $targetServerName -RefreshCredentialName $masterCred.CredentialName

# create ServerGroup with an exclusion of db2
$serverGroupExcludingDb2 = $jobAgent | New-AzSqlElasticJobTargetGroup -Name 'ServerGroupExcludingDb2'
$serverGroupExcludingDb2 | Add-AzSqlElasticJobTarget -ServerName $targetServerName -RefreshCredentialName $masterCred.CredentialName
$serverGroupExcludingDb2 | Add-AzSqlElasticJobTarget -ServerName $targetServerName -Database $db2.DatabaseName -Exclude
```

### <a name="create-a-job-and-steps"></a>Creación de un trabajo y pasos

En este ejemplo se definen un trabajo y dos pasos para que ejecute el trabajo. El primer paso de trabajo (*step1*) crea una nueva tabla (*Step1Table*) en cada base de datos del grupo de destino *ServerGroup*. El segundo paso (*step2*) crea una tabla (*Step2Table*) en cada base de datos excepto en *TargetDb2*, ya que el grupo de destino definido anteriormente especificó que se excluyera.

```powershell
Write-Output "Creating a new job..."
$jobName = "Job1"
$job = $jobAgent | New-AzSqlElasticJob -Name $jobName -RunOnce
$job

Write-Output "Creating job steps..."
$sqlText1 = "IF NOT EXISTS (SELECT * FROM sys.tables WHERE object_id = object_id('Step1Table')) CREATE TABLE [dbo].[Step1Table]([TestId] [int] NOT NULL);"
$sqlText2 = "IF NOT EXISTS (SELECT * FROM sys.tables WHERE object_id = object_id('Step2Table')) CREATE TABLE [dbo].[Step2Table]([TestId] [int] NOT NULL);"

$job | Add-AzSqlElasticJobStep -Name "step1" -TargetGroupName $serverGroup.TargetGroupName -CredentialName $jobCred.CredentialName -CommandText $sqlText1
$job | Add-AzSqlElasticJobStep -Name "step2" -TargetGroupName $serverGroupExcludingDb2.TargetGroupName -CredentialName $jobCred.CredentialName -CommandText $sqlText2
```

### <a name="run-the-job"></a>Ejecutar el trabajo

Para iniciar el trabajo inmediatamente, ejecute el comando siguiente:

```powershell
Write-Output "Start a new execution of the job..."
$jobExecution = $job | Start-AzSqlElasticJob
$jobExecution
```

Después de una correcta finalización debería ver dos nuevas tablas en TargetDb1 y solo una en TargetDb2:

   ![comprobación de las tablas nuevas en SSMS](./media/elastic-jobs-powershell-create/job-execution-verification.png)

También puede programar que el trabajo se ejecute más tarde. Para programar un trabajo para que se ejecute en un momento determinado, ejecute el siguiente comando:

```powershell
# run every hour starting from now
$job | Set-AzSqlElasticJob -IntervalType Hour -IntervalCount 1 -StartTime (Get-Date) -Enable
```

### <a name="monitor-status-of-job-executions"></a>Supervisión del estado de las ejecuciones de trabajos

Los siguientes fragmentos de código permiten obtener la información sobre las ejecuciones:

```powershell
# get the latest 10 executions run
$jobAgent | Get-AzSqlElasticJobExecution -Count 10

# get the job step execution details
$jobExecution | Get-AzSqlElasticJobStepExecution

# get the job target execution details
$jobExecution | Get-AzSqlElasticJobTargetExecution -Count 2
```

En la tabla siguiente se muestran los posibles estados de ejecución de los trabajos:

|State|Descripción|
|:---|:---|
|**Creado** | La ejecución del trabajo se acaba de crear y aún no está en curso.|
|**InProgress** | La ejecución del trabajo está en curso.|
|**WaitingForRetry** | La ejecución del trabajo no pudo completar la acción y está esperando para intentarlo de nuevo.|
|**Correcto** | La ejecución del trabajo se ha completado correctamente.|
|**SucceededWithSkipped** | La ejecución del trabajo se ha completado correctamente, pero se omitieron algunos de sus elementos secundarios.|
|**Erróneo** | La ejecución del trabajo ha dado error y ha agotado sus reintentos.|
|**TimedOut** | Se agotó el tiempo de espera para la ejecución del trabajo.|
|**Canceled** | Se canceló la ejecución del trabajo.|
|**Omitido** | Se omitió la ejecución del trabajo porque otra ejecución del mismo paso de trabajo ya se estaba ejecutando en el mismo destino.|
|**WaitingForChildJobExecutions** | La ejecución del trabajo está esperando a que se completen sus ejecuciones secundarias.|

## <a name="clean-up-resources"></a>Limpieza de recursos

Elimine los recursos de Azure que ha creado en este tutorial eliminando el grupo de recursos.

> [!TIP]
> Si planea seguir usando estos trabajos, no limpie los recursos creados en este artículo.

```powershell
Remove-AzResourceGroup -ResourceGroupName $resourceGroupName
```

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ejecutó un script de Transact-SQL en un conjunto de bases de datos. Ha aprendido a realizar las siguientes tareas:

> [!div class="checklist"]
> * Creación de un agente de trabajos elásticos
> * Creación de credenciales de trabajo para que los trabajos puedan ejecutar scripts en sus destinos
> * Definición de los destinos (servidores, grupos elásticos, bases de datos, mapas de particiones) en los que desea ejecutar el trabajo
> * Creación de credenciales de ámbito de base de datos en las bases de datos de destino para que el agente pueda conectar y ejecutar trabajos
> * Creación de un trabajo
> * Adición de un paso de trabajo al trabajo
> * Inicio de la ejecución del trabajo
> * Supervisión del trabajo

> [!div class="nextstepaction"]
> [Administración de trabajos elásticos mediante Transact-SQL](elastic-jobs-tsql-create-manage.md)
