---
title: Administración de Análisis de Azure Data Lake mediante Azure PowerShell
description: En este artículo se describe cómo usar Azure PowerShell para administrar cuentas, orígenes de datos, usuarios y trabajos de Data Lake Analytics.
services: data-lake-analytics
ms.service: data-lake-analytics
ms.reviewer: jasonh
ms.assetid: ad14d53c-fed4-478d-ab4b-6d2e14ff2097
ms.topic: how-to
ms.date: 06/29/2018
ms.openlocfilehash: 70a251db6c08f353f9c50512c41551e7a909a059
ms.sourcegitcommit: 0e8a4671aa3f5a9a54231fea48bcfb432a1e528c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/24/2020
ms.locfileid: "87125656"
---
# <a name="manage-azure-data-lake-analytics-using-azure-powershell"></a>Administración de Análisis de Azure Data Lake mediante Azure PowerShell
[!INCLUDE [manage-selector](../../includes/data-lake-analytics-selector-manage.md)]

En este artículo se describe cómo administrar cuentas, orígenes de datos, usuarios y trabajos de Azure Data Lake Analytics mediante Azure PowerShell.

## <a name="prerequisites"></a>Prerequisites

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Para usar PowerShell con Data Lake Analytics, recopile la siguiente información: 

* **Identificador de suscripción**: el identificador de la suscripción de Azure que contiene su cuenta de Data Lake Analytics.
* **Grupo de recursos**: el nombre del grupo de recursos de Azure que contiene la cuenta de Data Lake Analytics.
* **Nombre de la cuenta de Data Lake Analytics**: el nombre de la cuenta de Data Lake Analytics.
* **Nombre de la cuenta predeterminada de Data Lake Store**: cada cuenta de Data Lake Analytics tiene una cuenta de Data Lake Store predeterminada.
* **Ubicación**: la ubicación de la cuenta de Data Lake Analytics como "Este de EE. UU. 2" u otras ubicaciones admitidas.

Los fragmentos de código de PowerShell de este tutorial usan estas variables para almacenar esta información

```powershell
$subId = "<SubscriptionId>"
$rg = "<ResourceGroupName>"
$adla = "<DataLakeAnalyticsAccountName>"
$adls = "<DataLakeStoreAccountName>"
$location = "<Location>"
```

## <a name="log-in-to-azure"></a>Inicio de sesión en Azure

### <a name="log-in-using-interactive-user-authentication"></a>Inicio de sesión mediante la autenticación interactiva del usuario

Inicie sesión con un identificador de suscripción o por nombre de la suscripción

```powershell
# Using subscription id
Connect-AzAccount -SubscriptionId $subId

# Using subscription name
Connect-AzAccount -SubscriptionName $subname 
```

## <a name="saving-authentication-context"></a>Guardar el contexto de autenticación

El cmdlet `Connect-AzAccount` siempre pide las credenciales. Puede evitar que se le pidan mediante los cmdlets siguientes:

```powershell
# Save login session information
Save-AzAccounts -Path D:\profile.json  

# Load login session information
Select-AzAccounts -Path D:\profile.json 
```

### <a name="log-in-using-a-service-principal-identity-spi"></a>Inicio de sesión con una identidad de entidad de servicio (SPI)

```powershell
$tenantid = "XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX"  
$spi_appname = "appname" 
$spi_appid = "XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX" 
$spi_secret = "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX" 

$pscredential = New-Object System.Management.Automation.PSCredential ($spi_appid, (ConvertTo-SecureString $spi_secret -AsPlainText -Force))
Login-AzAccount -ServicePrincipal -TenantId $tenantid -Credential $pscredential -Subscription $subid
```

## <a name="manage-accounts"></a>Administrar cuentas


### <a name="list-accounts"></a>Enumeración de cuentas

```powershell
# List Data Lake Analytics accounts within the current subscription.
Get-AdlAnalyticsAccount

# List Data Lake Analytics accounts within a specific resource group.
Get-AdlAnalyticsAccount -ResourceGroupName $rg
```

### <a name="create-an-account"></a>Crear una cuenta

Cada cuenta de Data Lake Analytics requiere una cuenta predeterminada de Data Lake Store que se usa para almacenar registros. Puede reutilizar una cuenta existente o crear una nueva. 

```powershell
# Create a data lake store if needed, or you can re-use an existing one
New-AdlStore -ResourceGroupName $rg -Name $adls -Location $location
New-AdlAnalyticsAccount -ResourceGroupName $rg -Name $adla -Location $location -DefaultDataLake $adls
```

### <a name="get-account-information"></a>Obtención de información de la cuenta

Obtener detalles acerca de una cuenta.

```powershell
Get-AdlAnalyticsAccount -Name $adla
```

### <a name="check-if-an-account-exists"></a>Comprobando de la existencia de una cuenta

```powershell
Test-AdlAnalyticsAccount -Name $adla
```

## <a name="manage-data-sources"></a>Administración de orígenes de datos
Actualmente, Azure Data Lake Analytics admite los siguientes orígenes de datos:

* [Azure Data Lake Store](../data-lake-store/data-lake-store-overview.md)
* [Almacenamiento de Azure](../storage/common/storage-introduction.md)

Cada cuenta de Data Lake Analytics tiene una cuenta de Data Lake Store predeterminada. La cuenta predeterminada de Almacén de Data Lake sirve para almacenar los registros de auditoría y de metadatos de trabajos. 

### <a name="find-the-default-data-lake-store-account"></a>Búsqueda de la cuenta predeterminada de Almacén de Data Lake

```powershell
$adla_acct = Get-AdlAnalyticsAccount -Name $adla
$dataLakeStoreName = $adla_acct.DefaultDataLakeAccount
```

Puede encontrar la cuenta predeterminada de Data Lake Store si filtra la lista de orígenes de datos por la propiedad `IsDefault`:

```powershell
Get-AdlAnalyticsDataSource -Account $adla  | ? { $_.IsDefault } 
```

### <a name="add-a-data-source"></a>Agregar un origen de datos

```powershell

# Add an additional Storage (Blob) account.
$AzureStorageAccountName = "<AzureStorageAccountName>"
$AzureStorageAccountKey = "<AzureStorageAccountKey>"
Add-AdlAnalyticsDataSource -Account $adla -Blob $AzureStorageAccountName -AccessKey $AzureStorageAccountKey

# Add an additional Data Lake Store account.
$AzureDataLakeStoreName = "<AzureDataLakeStoreAccountName"
Add-AdlAnalyticsDataSource -Account $adla -DataLakeStore $AzureDataLakeStoreName 
```

### <a name="list-data-sources"></a>Enumerar orígenes de datos

```powershell
# List all the data sources
Get-AdlAnalyticsDataSource -Account $adla

# List attached Data Lake Store accounts
Get-AdlAnalyticsDataSource -Account $adla | where -Property Type -EQ "DataLakeStore"

# List attached Storage accounts
Get-AdlAnalyticsDataSource -Account $adla | where -Property Type -EQ "Blob"
```

## <a name="submit-u-sql-jobs"></a>Envío de trabajos de U-SQL

### <a name="submit-a-string-as-a-u-sql-job"></a>Envío de una cadena como un trabajo de U-SQL

```powershell
$script = @"
@a  = 
    SELECT * FROM 
        (VALUES
            ("Contoso", 1500.0),
            ("Woodgrove", 2700.0)
        ) AS D( customer, amount );
OUTPUT @a
    TO "/data.csv"
    USING Outputters.Csv();
"@

$scriptpath = "d:\test.usql"
$script | Out-File $scriptpath 

Submit-AdlJob -AccountName $adla -Script $script -Name "Demo"
```

### <a name="submit-a-file-as-a-u-sql-job"></a>Envío de un archivo como un trabajo de U-SQL

```powershell
$scriptpath = "d:\test.usql"
$script | Out-File $scriptpath 
Submit-AdlJob -AccountName $adla –ScriptPath $scriptpath -Name "Demo"
```

### <a name="list-jobs"></a>Enumeración de trabajos

El resultado incluye los trabajos actualmente en ejecución y aquellos que se han completado recientemente.

```powershell
Get-AdlJob -Account $adla
```

### <a name="list-the-top-n-jobs"></a>Enumere los principales trabajos de N

De forma predeterminada, la lista de trabajos se ordena por hora de envío. Así, los trabajos enviados más recientemente aparecen en primer lugar. De forma predeterminada, la cuenta de ADLA recuerda los trabajos durante 180 días, pero el cmdlet Get-AdlJob devuelve solo los primeros 500. Use el parámetro -Top para mostrar un número específico de trabajos.

```powershell
$jobs = Get-AdlJob -Account $adla -Top 10
```

### <a name="list-jobs-by-job-state"></a>Enumeración de trabajos por estado de trabajo

Mediante el parámetro `-State`. Puede combinar cualquiera de estos valores:

* `Accepted`
* `Compiling`
* `Ended`
* `New`
* `Paused`
* `Queued`
* `Running`
* `Scheduling`
* `Start`

```powershell
# List the running jobs
Get-AdlJob -Account $adla -State Running

# List the jobs that have completed
Get-AdlJob -Account $adla -State Ended

# List the jobs that have not started yet
Get-AdlJob -Account $adla -State Accepted,Compiling,New,Paused,Scheduling,Start
```

### <a name="list-jobs-by-job-result"></a>Enumeración de trabajos por resultado del trabajo

Use el parámetro `-Result` para detectar si los trabajos finalizados se han realizado correctamente. Tiene estos valores:

* Cancelado
* Con error
* None
* Correcto

``` powershell
# List Successful jobs.
Get-AdlJob -Account $adla -State Ended -Result Succeeded

# List Failed jobs.
Get-AdlJob -Account $adla -State Ended -Result Failed
```

### <a name="list-jobs-by-job-submitter"></a>Enumeración de trabajos por remitente del trabajo

El parámetro `-Submitter` ayuda a identificar quién ha enviado un trabajo.

```powershell
Get-AdlJob -Account $adla -Submitter "joe@contoso.com"
```

### <a name="list-jobs-by-submission-time"></a>Enumeración de trabajos por hora de envío

`-SubmittedAfter` es útil al filtrar por un intervalo de tiempo.


```powershell
# List  jobs submitted in the last day.
$d = [DateTime]::Now.AddDays(-1)
Get-AdlJob -Account $adla -SubmittedAfter $d

# List  jobs submitted in the last seven day.
$d = [DateTime]::Now.AddDays(-7)
Get-AdlJob -Account $adla -SubmittedAfter $d
```

### <a name="get-job-status"></a>Obtención de estado del trabajo

Obtenga el estado de un trabajo específico.

```powershell
Get-AdlJob -AccountName $adla -JobId $job.JobId
```


### <a name="cancel-a-job"></a>Cancelación de un trabajo

```powershell
Stop-AdlJob -Account $adla -JobID $jobID
```

### <a name="wait-for-a-job-to-finish"></a>Espera de fin de un trabajo

En lugar de repetir `Get-AdlAnalyticsJob` hasta que finalice un trabajo, puede usar el cmdlet `Wait-AdlJob` para esperar a que finalice el trabajo.

```powershell
Wait-AdlJob -Account $adla -JobId $job.JobId
```

## <a name="analyzing-job-history"></a>Análisis del historial de trabajo

El uso de Azure PowerShell para analizar el historial de trabajos que se han ejecutado en Data Lake Analytics es una técnica eficaz. Puede usarla para obtener información sobre el uso y el costo. Puede obtener más información examinando el [repositorio de ejemplo del análisis del historial de trabajo](https://github.com/Azure-Samples/data-lake-analytics-powershell-job-history-analysis)  

## <a name="list-job-pipelines-and-recurrences"></a>Enumeración de canalizaciones y repeticiones de trabajos

Use el cmdlet `Get-AdlJobPipeline` para ver la información de canalización de trabajos enviados previamente.

```powershell
$pipelines = Get-AdlJobPipeline -Account $adla
$pipeline = Get-AdlJobPipeline -Account $adla -PipelineId "<pipeline ID>"
```

Use el cmdlet `Get-AdlJobRecurrence` para ver la información de repetición de trabajos enviados previamente.

```powershell
$recurrences = Get-AdlJobRecurrence -Account $adla

$recurrence = Get-AdlJobRecurrence -Account $adla -RecurrenceId "<recurrence ID>"
```


## <a name="manage-compute-policies"></a>Administración de nodos directivas de proceso

### <a name="list-existing-compute-policies"></a>Lista de directivas de proceso existentes

El cmdlet `Get-AdlAnalyticsComputePolicy` recupera información sobre las directivas de proceso de una cuenta de Data Lake Analytics.

```powershell
$policies = Get-AdlAnalyticsComputePolicy -Account $adla
```

### <a name="create-a-compute-policy"></a>Creación de una directiva de proceso

El cmdlet `New-AdlAnalyticsComputePolicy` crea una nueva directiva de proceso para una cuenta de Data Lake Analytics. En este ejemplo se establece el número máximo de unidades de asignación (AU) disponibles para el usuario especificado en 50 y la prioridad mínima del trabajo en 250.

```powershell
$userObjectId = (Get-AzAdUser -SearchString "garymcdaniel@contoso.com").Id

New-AdlAnalyticsComputePolicy -Account $adla -Name "GaryMcDaniel" -ObjectId $objectId -ObjectType User -MaxDegreeOfParallelismPerJob 50 -MinPriorityPerJob 250
```
## <a name="manage-files"></a>Administración de archivos

### <a name="check-for-the-existence-of-a-file"></a>Compruebe la existencia de un archivo.

```powershell
Test-AdlStoreItem -Account $adls -Path "/data.csv"
```

### <a name="uploading-and-downloading"></a>Carga y descarga de archivos

Cargar un archivo.

```powershell
Import-AdlStoreItem -AccountName $adls -Path "c:\data.tsv" -Destination "/data_copy.csv" 
```

Cargue una carpeta completa de forma recursiva.

```powershell
Import-AdlStoreItem -AccountName $adls -Path "c:\myData\" -Destination "/myData/" -Recurse
```

Descargar un archivo.

```powershell
Export-AdlStoreItem -AccountName $adls -Path "/data.csv" -Destination "c:\data.csv"
```

Descargue una carpeta completa de forma recursiva.

```powershell
Export-AdlStoreItem -AccountName $adls -Path "/" -Destination "c:\myData\" -Recurse
```

> [!NOTE]
> Si se interrumpe el proceso de carga o descarga, puede intentar reanudar el proceso ejecutando el cmdlet de nuevo con la opción ``-Resume``.

## <a name="manage-the-u-sql-catalog"></a>Administración del catálogo de U-SQL

El catálogo de U-SQL se usa para estructurar datos y código, para que puedan compartirse mediante scripts de U-SQL. El catálogo permite el mayor rendimiento posible con los datos en Azure Data Lake. Para obtener más información, consulte [Uso del catálogo de U-SQL](data-lake-analytics-use-u-sql-catalog.md).

### <a name="list-items-in-the-u-sql-catalog"></a>Enumerar elementos en el catálogo de U-SQL

```powershell
# List U-SQL databases
Get-AdlCatalogItem -Account $adla -ItemType Database 

# List tables within a database
Get-AdlCatalogItem -Account $adla -ItemType Table -Path "database"

# List tables within a schema.
Get-AdlCatalogItem -Account $adla -ItemType Table -Path "database.schema"
```

### <a name="list-all-the-assemblies-the-u-sql-catalog"></a>Enumeración de todos los conjuntos del catálogo de U-SQL

```powershell
$dbs = Get-AdlCatalogItem -Account $adla -ItemType Database

foreach ($db in $dbs)
{
    $asms = Get-AdlCatalogItem -Account $adla -ItemType Assembly -Path $db.Name

    foreach ($asm in $asms)
    {
        $asmname = "[" + $db.Name + "].[" + $asm.Name + "]"
        Write-Host $asmname
    }
}
```

### <a name="get-details-about-a-catalog-item"></a>Obtener detalles sobre un elemento de catálogo

```powershell
# Get details of a table
Get-AdlCatalogItem  -Account $adla -ItemType Table -Path "master.dbo.mytable"

# Test existence of a U-SQL database.
Test-AdlCatalogItem  -Account $adla -ItemType Database -Path "master"
```

### <a name="store-credentials-in-the-catalog"></a>Almacenamiento de credenciales en el catálogo

Dentro de una base de datos U-SQL, cree un objeto de credenciales para una base de datos hospedada en Azure. Actualmente, las credenciales U-SQL son el único tipo de elemento de catálogo que se puede crear a través de PowerShell.

```powershell
$dbName = "master"
$credentialName = "ContosoDbCreds"
$dbUri = "https://contoso.database.windows.net:8080"

New-AdlCatalogCredential -AccountName $adla `
          -DatabaseName $db `
          -CredentialName $credentialName `
          -Credential (Get-Credential) `
          -Uri $dbUri
```

## <a name="manage-firewall-rules"></a>Administración de las reglas del firewall

### <a name="list-firewall-rules"></a>Enumerar reglas de firewall

```powershell
Get-AdlAnalyticsFirewallRule -Account $adla
```

### <a name="add-a-firewall-rule"></a>Incorporación de una regla de firewall

```powershell
$ruleName = "Allow access from on-prem server"
$startIpAddress = "<start IP address>"
$endIpAddress = "<end IP address>"

Add-AdlAnalyticsFirewallRule -Account $adla -Name $ruleName -StartIpAddress $startIpAddress -EndIpAddress $endIpAddress
```

### <a name="modify-a-firewall-rule"></a>Modificación de una regla de firewall

```powershell
Set-AdlAnalyticsFirewallRule -Account $adla -Name $ruleName -StartIpAddress $startIpAddress -EndIpAddress $endIpAddress
```

### <a name="remove-a-firewall-rule"></a>Eliminación de una regla de firewall

```powershell
Remove-AdlAnalyticsFirewallRule -Account $adla -Name $ruleName
```

### <a name="allow-azure-ip-addresses"></a>Permitir direcciones IP de Azure

```powershell
Set-AdlAnalyticsAccount -Name $adla -AllowAzureIpState Enabled
```

```powershell
Set-AdlAnalyticsAccount -Name $adla -FirewallState Enabled
Set-AdlAnalyticsAccount -Name $adla -FirewallState Disabled
```


## <a name="working-with-azure"></a>Trabajo con Azure

### <a name="get-error-details"></a>Ver detalles del error

```powershell
Resolve-AzError -Last
```

### <a name="verify-if-you-are-running-as-an-administrator-on-your-windows-machine"></a>Comprobación de si se ejecuta un administrador en el equipo Windows

```powershell
function Test-Administrator  
{  
    $user = [Security.Principal.WindowsIdentity]::GetCurrent();
    $p = New-Object Security.Principal.WindowsPrincipal $user
    $p.IsInRole([Security.Principal.WindowsBuiltinRole]::Administrator)  
}
```

### <a name="find-a-tenantid"></a>Buscar un elemento TenantID

A partir de un nombre de suscripción:

```powershell
function Get-TenantIdFromSubscriptionName( [string] $subname )
{
    $sub = (Get-AzSubscription -SubscriptionName $subname)
    $sub.TenantId
}

Get-TenantIdFromSubscriptionName "ADLTrainingMS"
```

A partir de un identificador de suscripción:

```powershell
function Get-TenantIdFromSubscriptionId( [string] $subid )
{
    $sub = (Get-AzSubscription -SubscriptionId $subid)
    $sub.TenantId
}

$subid = "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
Get-TenantIdFromSubscriptionId $subid
```

A partir de una dirección de dominio como "contoso.com"

```powershell
function Get-TenantIdFromDomain( $domain )
{
    $url = "https://login.windows.net/" + $domain + "/.well-known/openid-configuration"
    return (Invoke-WebRequest $url|ConvertFrom-Json).token_endpoint.Split('/')[3]
}

$domain = "contoso.com"
Get-TenantIdFromDomain $domain
```

### <a name="list-all-your-subscriptions-and-tenant-ids"></a>Enumeración de todas las suscripciones e identificadores de inquilino

```powershell
$subs = Get-AzSubscription
foreach ($sub in $subs)
{
    Write-Host $sub.Name "("  $sub.Id ")"
    Write-Host "`tTenant Id" $sub.TenantId
}
```

## <a name="create-a-data-lake-analytics-account-using-a-template"></a>Creación de una cuenta de Data Lake Analytics mediante una plantilla

También puede usar una plantilla de Grupo de recursos de Azure con el siguiente ejemplo: [Creación de una cuenta de Data Lake Analytics mediante una plantilla](https://github.com/Azure-Samples/data-lake-analytics-create-account-with-arm-template)

## <a name="next-steps"></a>Pasos siguientes
* [Información general de Análisis de Microsoft Azure Data Lake](data-lake-analytics-overview.md)
* Introducción a Data Lake Analytics mediante [Azure Portal](data-lake-analytics-get-started-portal.md) | [Azure PowerShell](data-lake-analytics-get-started-powershell.md) | [CLI de Azure](data-lake-analytics-get-started-cli.md)
* Administración de Azure Data Lake Analytics mediante [Azure Portal](data-lake-analytics-manage-use-portal.md) | [Azure PowerShell](data-lake-analytics-manage-use-powershell.md) | [CLI](data-lake-analytics-manage-use-cli.md) 
