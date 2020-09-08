---
title: Administración de Azure Data Lake Analytics con el SDK de .NET para Azure
description: En este artículo se describe cómo usar el SDK de .NET para Azure para escribir aplicaciones que administran trabajos, orígenes de datos y usuarios de Data Lake Analytics.
services: data-lake-analytics
ms.reviewer: jasonh
ms.assetid: 811d172d-9873-4ce9-a6d5-c1a26b374c79
ms.service: data-lake-analytics
ms.topic: how-to
ms.date: 06/18/2017
ms.custom: devx-track-csharp
ms.openlocfilehash: b06530488e74120f3cc511694d290840269db453
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89020072"
---
# <a name="manage-azure-data-lake-analytics-a-net-app"></a>Administración de Azure Data Lake Analytics mediante una aplicación .NET

[!INCLUDE [manage-selector](../../includes/data-lake-analytics-selector-manage.md)]

En este artículo se describe cómo administrar cuentas, orígenes de datos, usuarios y trabajos de Azure Data Lake Analytics con una aplicación escrita mediante el SDK de Azure .NET. 

## <a name="prerequisites"></a>Prerequisites

* **Visual Studio 2015, Visual Studio 2013 Update 4 o Visual Studio 2012 con Visual C++ instalado**.
* **SDK de Microsoft Azure para .NET versión 2.5 o posterior**.  Instálelo usando el [instalador de plataforma web](https://www.microsoft.com/web/downloads/platform.aspx).
* **Paquetes de NuGet requeridos**

### <a name="install-nuget-packages"></a>Instalación de paquetes NuGet

|Paquete|Versión|
|-------|-------|
|[Microsoft.Rest.ClientRuntime.Azure.Authentication](https://www.nuget.org/packages/Microsoft.Rest.ClientRuntime.Azure.Authentication)| 2.3.1|
|[Microsoft.Azure.Management.DataLake.Analytics](https://www.nuget.org/packages/Microsoft.Azure.Management.DataLake.Analytics)|3.0.0|
|[Microsoft.Azure.Management.DataLake.Store](https://www.nuget.org/packages/Microsoft.Azure.Management.DataLake.Store)|2.2.0|
|[Microsoft.Azure.Management.ResourceManager](https://www.nuget.org/packages/Microsoft.Azure.Management.ResourceManager)|1.6.0-preview|
|[Microsoft.Azure.Graph.RBAC](https://www.nuget.org/packages/Microsoft.Azure.Management.ResourceManager)|3.4.0-preview|

Estos paquetes se pueden instalar desde la línea de comandos de NuGet con los siguientes comandos:

```powershell
Install-Package -Id Microsoft.Rest.ClientRuntime.Azure.Authentication  -Version 2.3.1
Install-Package -Id Microsoft.Azure.Management.DataLake.Analytics  -Version 3.0.0
Install-Package -Id Microsoft.Azure.Management.DataLake.Store  -Version 2.2.0
Install-Package -Id Microsoft.Azure.Management.ResourceManager  -Version 1.6.0-preview
Install-Package -Id Microsoft.Azure.Graph.RBAC -Version 3.4.0-preview
```

## <a name="common-variables"></a>Variables comunes

```csharp
string subid = "<Subscription ID>"; // Subscription ID (a GUID)
string tenantid = "<Tenant ID>"; // AAD tenant ID or domain. For example, "contoso.onmicrosoft.com"
string rg == "<value>"; // Resource  group name
string clientid = "1950a258-227b-4e31-a9cf-717495945fc2"; // Sample client ID (this will work, but you should pick your own)
```

## <a name="authentication"></a>Authentication

Tiene varias opciones para registrarse en Azure Data Lake Analytics. El fragmento de código siguiente muestra un ejemplo de autenticación con la autenticación de usuario interactiva con un elemento emergente.

``` csharp
using System;
using System.IO;
using System.Threading;
using System.Security.Cryptography.X509Certificates;

using Microsoft.Rest;
using Microsoft.Rest.Azure.Authentication;
using Microsoft.Azure.Management.DataLake.Analytics;
using Microsoft.Azure.Management.DataLake.Analytics.Models;
using Microsoft.Azure.Management.DataLake.Store;
using Microsoft.Azure.Management.DataLake.Store.Models;
using Microsoft.IdentityModel.Clients.ActiveDirectory;
using Microsoft.Azure.Graph.RBAC;

public static Program
{
   public static string TENANT = "microsoft.onmicrosoft.com";
   public static string CLIENTID = "1950a258-227b-4e31-a9cf-717495945fc2";
   public static System.Uri ARM_TOKEN_AUDIENCE = new System.Uri( @"https://management.core.windows.net/");
   public static System.Uri ADL_TOKEN_AUDIENCE = new System.Uri( @"https://datalake.azure.net/" );
   public static System.Uri GRAPH_TOKEN_AUDIENCE = new System.Uri( @"https://graph.windows.net/" );

   static void Main(string[] args)
   {
      string MY_DOCUMENTS= System.Environment.GetFolderPath( System.Environment.SpecialFolder.MyDocuments);
      string TOKEN_CACHE_PATH = System.IO.Path.Combine(MY_DOCUMENTS, "my.tokencache");

      var tokenCache = GetTokenCache(TOKEN_CACHE_PATH);
      var armCreds = GetCreds_User_Popup(TENANT, ARM_TOKEN_AUDIENCE, CLIENTID, tokenCache);
      var adlCreds = GetCreds_User_Popup(TENANT, ADL_TOKEN_AUDIENCE, CLIENTID, tokenCache);
      var graphCreds = GetCreds_User_Popup(TENANT, GRAPH_TOKEN_AUDIENCE, CLIENTID, tokenCache);
   }
}
```

Tanto el código fuente de **GetCreds_User_Popup** como el código de otras opciones de autenticación se tratan en [Data Lake Analytics .NET authentication options](https://github.com/Azure-Samples/data-lake-analytics-dotnet-auth-options) (Opciones de autenticación de .NET de Data Lake Analytics)


## <a name="create-the-client-management-objects"></a>Creación de objetos de administración de cliente

``` csharp
var resourceManagementClient = new ResourceManagementClient(armCreds) { SubscriptionId = subid };

var adlaAccountClient = new DataLakeAnalyticsAccountManagementClient(armCreds);
adlaAccountClient.SubscriptionId = subid;

var adlsAccountClient = new DataLakeStoreAccountManagementClient(armCreds);
adlsAccountClient.SubscriptionId = subid;

var adlaCatalogClient = new DataLakeAnalyticsCatalogManagementClient(adlCreds);
var adlaJobClient = new DataLakeAnalyticsJobManagementClient(adlCreds);

var adlsFileSystemClient = new DataLakeStoreFileSystemManagementClient(adlCreds);

var  graphClient = new GraphRbacManagementClient(graphCreds);
graphClient.TenantID = domain;

```

## <a name="manage-accounts"></a>Administrar cuentas

### <a name="create-an-azure-resource-group"></a>Creación de un grupo de recursos de Azure

Si aún no ha creado ninguno, debe tener un grupo de recursos de Azure para crear los componentes de Data Lake Analytics. Necesita las credenciales de autenticación, el identificador de la suscripción y una ubicación. El código siguiente muestra cómo crear un grupo de recursos:

``` csharp
var resourceGroup = new ResourceGroup { Location = location };
resourceManagementClient.ResourceGroups.CreateOrUpdate(groupName, rg);
```

Para más información, vea Grupos de recursos de Azure y Data Lake Analytics.

### <a name="create-a-data-lake-store-account"></a>Crear una cuenta de Almacén de Data Lake

Todas las cuenta ADLA requieren una cuenta ADLS. Si no tiene ninguna, puede crearla con el siguiente código:

``` csharp
var new_adls_params = new DataLakeStoreAccount(location: _location);
adlsAccountClient.Account.Create(rg, adls, new_adls_params);
```

### <a name="create-a-data-lake-analytics-account"></a>Creación de una cuenta de Análisis de Data Lake

El siguiente código crea una cuenta ADLS

``` csharp
var new_adla_params = new DataLakeAnalyticsAccount()
{
   DefaultDataLakeStoreAccount = adls,
   Location = location
};

adlaClient.Account.Create(rg, adla, new_adla_params);
```

### <a name="list-data-lake-store-accounts"></a>Enumeración de cuentas de Data Lake Store

``` csharp
var adlsAccounts = adlsAccountClient.Account.List().ToList();
foreach (var adls in adlsAccounts)
{
   Console.WriteLine($"ADLS: {0}", adls.Name);
}
```

### <a name="list-data-lake-analytics-accounts"></a>Enumeración de cuentas de Análisis de Data Lake

``` csharp
var adlaAccounts = adlaClient.Account.List().ToList();

for (var adla in AdlaAccounts)
{
   Console.WriteLine($"ADLA: {0}, adla.Name");
}
```

### <a name="checking-if-an-account-exists"></a>Comprobando si existe una cuenta

``` csharp
bool exists = adlaClient.Account.Exists(rg, adla));
```

### <a name="get-information-about-an-account"></a>Obtener información de una cuenta

``` csharp
bool exists = adlaClient.Account.Exists(rg, adla));
if (exists)
{
   var adla_accnt = adlaClient.Account.Get(rg, adla);
}
```

### <a name="delete-an-account"></a>Eliminación de una cuenta

``` csharp
if (adlaClient.Account.Exists(rg, adla))
{
   adlaClient.Account.Delete(rg, adla);
}
```

### <a name="get-the-default-data-lake-store-account"></a>Obtención de la cuenta predeterminada de Data Lake Store

Cada cuenta de Data Lake Analytics requiere una cuenta de Data Lake Store predeterminada. Utilice este código para determinar la cuenta de Store predeterminada para una cuenta de Analytics.

``` csharp
if (adlaClient.Account.Exists(rg, adla))
{
  var adla_accnt = adlaClient.Account.Get(rg, adla);
  string def_adls_account = adla_accnt.DefaultDataLakeStoreAccount;
}
```

## <a name="manage-data-sources"></a>Administración de orígenes de datos

Actualmente, Análisis de Data Lake admite los siguientes orígenes de datos:

* [Azure Data Lake Store](../data-lake-store/data-lake-store-overview.md)
* [Cuenta de Azure Storage](../storage/common/storage-introduction.md)

### <a name="link-to-an-azure-storage-account"></a>Vínculo a una cuenta de Azure Storage

Se pueden crear vínculos a cuentas de Azure Storage.

``` csharp
string storage_key = "xxxxxxxxxxxxxxxxxxxx";
string storage_account = "mystorageaccount";
var addParams = new AddStorageAccountParameters(storage_key);            
adlaClient.StorageAccounts.Add(rg, adla, storage_account, addParams);
```

### <a name="list-azure-storage-data-sources"></a>Enumeración de orígenes de datos de Azure Storage

``` csharp
var stg_accounts = adlaAccountClient.StorageAccounts.ListByAccount(rg, adla);

if (stg_accounts != null)
{
  foreach (var stg_account in stg_accounts)
  {
      Console.WriteLine($"Storage account: {0}", stg_account.Name);
  }
}
```

### <a name="list-data-lake-store-data-sources"></a>Lista de orígenes de datos de Data Lake Store

``` csharp
var adls_accounts = adlsClient.Account.List();

if (adls_accounts != null)
{
  foreach (var adls_accnt in adls_accounts)
  {
      Console.WriteLine($"ADLS account: {0}", adls_accnt.Name);
  }
}
```

### <a name="upload-and-download-folders-and-files"></a>Cargar y descargar archivos y carpetas

El objeto de administración de cliente del sistema de archivos de Data Lake Store se puede usar para cargar y descargar archivos o carpetas individuales desde Azure al equipo local, usando los métodos siguientes:

- UploadFolder
- UploadFile
- DownloadFolder
- DownloadFile

El primer parámetro de estos métodos es el nombre de la cuenta de Data Lake Store, seguido de parámetros para la ruta de acceso de origen y la ruta de acceso de destino.

En el ejemplo siguiente se muestra cómo descargar una carpeta en Data Lake Store.

``` csharp
adlsFileSystemClient.FileSystem.DownloadFolder(adls, sourcePath, destinationPath);
```

### <a name="create-a-file-in-a-data-lake-store-account"></a>Crear un archivo en una cuenta de Data Lake Store

``` csharp
using (var memstream = new MemoryStream())
{
   using (var sw = new StreamWriter(memstream, UTF8Encoding.UTF8))
   {
      sw.WriteLine("Hello World");
      sw.Flush();
      
      memstream.Position = 0;

      adlsFileSystemClient.FileSystem.Create(adls, "/Samples/Output/randombytes.csv", memstream);
   }
}
```

### <a name="verify-azure-storage-account-paths"></a>Verificación de las rutas de acceso a la cuenta de Azure Storage

El código siguiente comprueba si hay una cuenta de Azure Storage (storageAccntName) en una cuenta de Data Lake Analytics (analyticsAccountName), y si hay un contenedor (containerName) en la cuenta de Azure Storage.

``` csharp
string storage_account = "mystorageaccount";
string storage_container = "mycontainer";
bool accountExists = adlaClient.Account.StorageAccountExists(rg, adla, storage_account));
bool containerExists = adlaClient.Account.StorageContainerExists(rg, adla, storage_account, storage_container));
```

## <a name="manage-catalog-and-jobs"></a>Administración del catálogo y los trabajos

El objeto DataLakeAnalyticsCatalogManagementClient proporciona métodos para administrar la base de datos SQL que se proporciona a cada cuenta de Azure Data Lake Analytics. DataLakeAnalyticsJobManagementClient proporciona métodos para enviar y administrar trabajos ejecutados en la base de datos con scripts de U-SQL.

### <a name="list-databases-and-schemas"></a>Lista de bases de datos y esquemas

Entre las diversas cosas que puede incluir en la lista, las más comunes son las bases de datos y su esquema. El código siguiente obtiene una colección de bases de datos y, a continuación, enumera el esquema de cada base de datos.

``` csharp
var databases = adlaCatalogClient.Catalog.ListDatabases(adla);
foreach (var db in databases)
{
  Console.WriteLine($"Database: {db.Name}");
  Console.WriteLine(" - Schemas:");
  var schemas = adlaCatalogClient.Catalog.ListSchemas(adla, db.Name);
  foreach (var schm in schemas)
  {
      Console.WriteLine($"\t{schm.Name}");
  }
}
```

### <a name="list-table-columns"></a>Lista de columnas de la tabla

El código siguiente muestra cómo acceder a la base de datos con un cliente de administración del catálogo de Data Lake Analytics para mostrar las columnas de una tabla especificada.

```csharp
var tbl = adlaCatalogClient.Catalog.GetTable(adla, "master", "dbo", "MyTableName");
IEnumerable<USqlTableColumn> columns = tbl.ColumnList;

foreach (USqlTableColumn utc in columns)
{
  Console.WriteLine($"\t{utc.Name}");
}
```

### <a name="submit-a-u-sql-job"></a>Envío de un trabajo de U-SQL

El código siguiente muestra cómo usar un cliente de administración de trabajo de Data Lake Analytics para enviar un trabajo.

``` csharp
string scriptPath = "/Samples/Scripts/SearchResults_Wikipedia_Script.txt";
Stream scriptStrm = adlsFileSystemClient.FileSystem.Open(_adlsAccountName, scriptPath);
string scriptTxt = string.Empty;
using (StreamReader sr = new StreamReader(scriptStrm))
{
    scriptTxt = sr.ReadToEnd();
}

var jobName = "SR_Wikipedia";
var jobId = Guid.NewGuid();
var properties = new USqlJobProperties(scriptTxt);
var parameters = new JobInformation(jobName, JobType.USql, properties, priority: 1, degreeOfParallelism: 1, jobId: jobId);
var jobInfo = adlaJobClient.Job.Create(adla, jobId, parameters);
Console.WriteLine($"Job {jobName} submitted.");
```

### <a name="list-failed-jobs"></a>Lista de trabajos con errores

El código siguiente muestra información acerca de trabajos con errores.

```csharp
var odq = new ODataQuery<JobInformation> { Filter = "result eq 'Failed'" };
var jobs = adlaJobClient.Job.List(adla, odq);
foreach (var j in jobs)
{
   Console.WriteLine($"{j.Name}\t{j.JobId}\t{j.Type}\t{j.StartTime}\t{j.EndTime}");
}
```

### <a name="list-pipelines"></a>Enumerar canalizaciones

El código siguiente muestra información sobre cada canalización de los trabajos enviados a la cuenta.

``` csharp
var pipelines = adlaJobClient.Pipeline.List(adla);
foreach (var p in pipelines)
{
   Console.WriteLine($"Pipeline: {p.Name}\t{p.PipelineId}\t{p.LastSubmitTime}");
}
```

### <a name="list-recurrences"></a>Enumerar repeticiones

El código siguiente muestra información sobre cada repetición de los trabajos enviados a la cuenta.

``` csharp
var recurrences = adlaJobClient.Recurrence.List(adla);
foreach (var r in recurrences)
{
   Console.WriteLine($"Recurrence: {r.Name}\t{r.RecurrenceId}\t{r.LastSubmitTime}");
}
```

## <a name="common-graph-scenarios"></a>Escenarios comunes de grafos

### <a name="look-up-user-in-the-aad-directory"></a>Buscar usuario en el directorio AAD

``` csharp
var userinfo = graphClient.Users.Get( "bill@contoso.com" );
```

### <a name="get-the-objectid-of-a-user-in-the-aad-directory"></a>Obtención de ObjectId de usuario en el directorio de AAD

``` csharp
var userinfo = graphClient.Users.Get( "bill@contoso.com" );
Console.WriteLine( userinfo.ObjectId )
```

## <a name="manage-compute-policies"></a>Administración de nodos directivas de proceso

El objeto de DataLakeAnalyticsAccountManagementClient proporciona métodos para administrar las directivas de proceso de una cuenta de Data Lake Analytics.

### <a name="list-compute-policies"></a>Enumeración de directivas de proceso

El código siguiente recupera una lista de directivas de proceso de una cuenta de Data Lake Analytics.

``` csharp
var policies = adlaAccountClient.ComputePolicies.ListByAccount(rg, adla);
foreach (var p in policies)
{
   Console.WriteLine($"Name: {p.Name}\tType: {p.ObjectType}\tMax AUs / job: {p.MaxDegreeOfParallelismPerJob}\tMin priority / job: {p.MinPriorityPerJob}");
}
```

### <a name="create-a-new-compute-policy"></a>Creación de una nueva directiva de proceso

El siguiente código crea una nueva directiva de cálculo para una cuenta de análisis de Data Lake y establece que el número máximo de AU disponibles para el usuario especificado en 50 y la prioridad del trabajo mínimo en 250.

``` csharp
var userAadObjectId = "3b097601-4912-4d41-b9d2-78672fc2acde";
var newPolicyParams = new ComputePolicyCreateOrUpdateParameters(userAadObjectId, "User", 50, 250);
adlaAccountClient.ComputePolicies.CreateOrUpdate(rg, adla, "GaryMcDaniel", newPolicyParams);
```

## <a name="next-steps"></a>Pasos siguientes

* [Información general de Análisis de Microsoft Azure Data Lake](data-lake-analytics-overview.md)
* [Administración de Azure Data Lake Analytics con Azure Portal](data-lake-analytics-manage-use-portal.md)
* [Supervisión y solución de problemas de trabajos de Azure Data Lake Analytics con Azure Portal](data-lake-analytics-monitor-and-troubleshoot-jobs-tutorial.md)