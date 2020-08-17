---
title: Procedimiento para configurar una canalización de CI/CD para Azure Data Lake Analytics
description: Aprenda a configurar una integración continua y una implementación continua para Azure Data Lake Analytics.
services: data-lake-analytics
author: liudan66
ms.author: liud
ms.reviewer: jasonh
ms.assetid: 66dd58b1-0b28-46d1-aaae-43ee2739ae0a
ms.service: data-lake-analytics
ms.topic: how-to
ms.workload: big-data
ms.date: 09/14/2018
ms.openlocfilehash: 3517938ae0e08af62a6fcf0d3d0a43a5eaee48dd
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87496124"
---
# <a name="how-to-set-up-a-cicd-pipeline-for-azure-data-lake-analytics"></a>Procedimiento para configurar una canalización de CI/CD para Azure Data Lake Analytics  

En este artículo, aprenderá a configurar una canalización de integración e implementación continua (CI/CD) para trabajos U-SQL y bases de datos U-SQL.  

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="use-cicd-for-u-sql-jobs"></a>Uso de CI/CD para trabajos U-SQL

Las Herramientas de Azure Data Lake para Visual Studio proporcionan un tipo de proyecto de U-SQL que le ayuda a organizar los scripts de U-SQL. El uso del proyecto de U-SQL para administrar código U-SQL facilita los escenarios de CI/CD.

## <a name="build-a-u-sql-project"></a>Compilación de un proyecto de U-SQL

Un proyecto de U-SQL se puede compilar con Microsoft Build Engine (MSBuild). Para ello, es preciso usar los parámetros correspondientes. Siga los pasos descritos en este artículo para configurar un proceso de compilación de un proyecto de U-SQL.

### <a name="project-migration"></a>Migración del proyecto

Antes de configurar una tarea de compilación de un proyecto de U-SQL, asegúrese de usar la versión más reciente del mismo. Abra el archivo de proyecto de U-SQL en el editor y compruebe que dispone de estos elementos de importación:

```xml
<!-- check for SDK Build target in current path then in USQLSDKPath-->
<Import Project="UsqlSDKBuild.targets" Condition="Exists('UsqlSDKBuild.targets')" />
<Import Project="$(USQLSDKPath)\UsqlSDKBuild.targets" Condition="!Exists('UsqlSDKBuild.targets') And '$(USQLSDKPath)' != '' And Exists('$(USQLSDKPath)\UsqlSDKBuild.targets')" />
``` 

En caso negativo, tiene dos opciones para migrar el proyecto:

- Opción 1: Cambiar el elemento de importación antiguo por el de anterior.
- Opción 2: Abrir el proyecto antiguo en Herramientas de Azure Data Lake para Visual Studio posterior. Use una versión más reciente que 2.3.3000.0. La plantilla del proyecto antiguo se actualizará automáticamente a la versión más reciente. Los nuevos proyectos creados con versiones más recientes que 2.3.3000.0 utilizan la nueva plantilla.

### <a name="get-nuget"></a>Obtención de NuGet

MSBuild no proporciona compatibilidad integrada con proyectos de U-SQL. Para conseguir esta compatibilidad, es preciso agregar una referencia para la solución al paquete NuGet [Microsoft.Azure.DataLake.USQL.SDK](https://www.nuget.org/packages/Microsoft.Azure.DataLake.USQL.SDK/) que agregue el servicio de lenguaje requerido.

Para agregar dicha referencia al paquete NuGet, puede hacer clic con el botón derecho en la solución en el Explorador de soluciones de Visual Studio y elegir **Administrar paquetes NuGet**. O bien puede agregar un archivo llamado `packages.config` a la carpeta de la solución y agregarle el contenido siguiente:

```xml 
<?xml version="1.0" encoding="utf-8"?>
<packages>
  <package id="Microsoft.Azure.DataLake.USQL.SDK" version="1.3.180620" targetFramework="net452" />
</packages>
``` 

### <a name="manage-u-sql-database-references"></a>Administración de referencias de la base de datos de U-SQL

Los scripts de U-SQL en un proyecto de U-SQL podrían tener las instrucciones de consulta para los objetos de base de datos U-SQL. En ese caso, deberá hacer referencia al proyecto de base de datos U-SQL correspondiente que incluya la definición de los objetos antes de compilar el proyecto de U-SQL. Un ejemplo es cuando se consulta una tabla U-SQL o una referencia a un ensamblado. 

Más información acerca del [proyecto de base de datos U-SQL](data-lake-analytics-data-lake-tools-develop-usql-database.md).

>[!NOTE]
> La instrucción DROP puede causar una eliminación accidental. Para habilitar la instrucción DROP, deberá especificar explícitamente los argumentos de MSBuild. **AllowDropStatement** permitirá una operación DROP no relacionada con los datos, como la función con valor de DROP ASSEMBLY y DROP TABLE. **AllowDataDropStatement** permitirá una operación DROP relacionada con los datos, como DROP TABLE y DROP SCHEMA. Tendrá que habilitar AllowDropStatement antes de usar AllowDataDropStatement.
>

### <a name="build-a-u-sql-project-with-the-msbuild-command-line"></a>Compilación de un proyecto de U-SQL con la línea de comandos de MSBuild

En primer lugar, migre el proyecto y obtenga el paquete NuGet. A continuación, llame a la línea de comandos de MSBuild estándar con los argumentos adicionales siguientes para compilar el proyecto de U-SQL: 

```console
msbuild USQLBuild.usqlproj /p:USQLSDKPath=packages\Microsoft.Azure.DataLake.USQL.SDK.1.3.180615\build\runtime;USQLTargetType=SyntaxCheck;DataRoot=datarootfolder;/p:EnableDeployment=true
``` 

La definición de los argumentos y los valores son los siguientes:

* **USQLSDKPath=\<U-SQL Nuget package>\build\runtime**. Este parámetro hace referencia a la ruta de instalación del paquete NuGet para el servicio de lenguaje de U-SQL.
* **USQLTargetType=Merge o SyntaxCheck**:
    * **Merge**. El modo Merge compila los archivos de código subyacente. Algunos ejemplos son los archivos **.cs**, **.py** y **.r**. Incorpora la biblioteca de código definido por el usuario resultante en el script U-SQL. Algunos ejemplos son un archivo binario .dll, Python o código de R.
    * **SyntaxCheck**. El modo SyntaxCheck primero combina los archivos de código subyacente en el script U-SQL. A continuación, compila el script U-SQL para validar el código.
* **DataRoot=\<DataRoot path>** . DataRoot solo se necesita para el modo SyntaxCheck. Al compilar el script con el modo SyntaxCheck, MSBuild comprueba las referencias a objetos de base de datos en el script. Antes de compilar, configure un entorno local coincidente que contenga los objetos a los que se hace referencia desde la base de datos U-SQL en la carpeta DataRoot del equipo en que se realiza compilación. Para administrar estas dependencias de la base de datos, también se puede [hacer referencia a un proyecto de base de datos de U-SQL](data-lake-analytics-data-lake-tools-develop-usql-database.md#reference-a-u-sql-database-project). MSBuild solo comprueba la referencia de los objetos de base de datos, no los archivos.
* **EnableDeployment=true** o **false**. EnableDeployment indica si se permite implementar bases de datos U-SQL de referencia durante el proceso de compilación. Si hace referencia al proyecto de base de datos U-SQL y consume los objetos de base de datos en el script U-SQL, establezca este parámetro en **true**.

### <a name="continuous-integration-through-azure-pipelines"></a>Integración continua mediante Azure Pipelines

Además de la línea de comandos, puede utilizar la tarea de compilación de Visual Studio Build o una tarea de MSBuild para compilar proyectos de U-SQL en Azure Pipelines. Para configurar una canalización de compilación, asegúrese de agregar dos tareas en la canalización de compilación: una tarea de restauración de NuGet y una tarea de MSBuild.

![Tarea de MSBuild para un proyecto de U-SQL](./media/data-lake-analytics-cicd-overview/data-lake-analytics-set-vsts-msbuild-task.png) 

1.  Agregue una tarea de restauración de NuGet para obtener el paquete NuGet al que hace referencia la solución que incluye `Azure.DataLake.USQL.SDK`, con el fin de que MSBuild pueda encontrar los destinos del lenguaje U-SQL. Establezca **Opciones avanzadas** > **Directorio de destino** en `$(Build.SourcesDirectory)/packages` si desea usar el ejemplo de argumentos de MSBuild directamente en el paso 2.

    ![Tarea de restauración de NuGet para un proyecto de U-SQL](./media/data-lake-analytics-cicd-overview/data-lake-analytics-set-vsts-nuget-task.png)

2.  Establezca los argumentos de MSBuild en herramientas de compilación de Visual Studio o en una tarea de MSBuild como se muestra en el ejemplo siguiente. O bien, puede definir variables para estos argumentos en la canalización de compilación de Azure Pipelines.

    ![Definición de variables de MSBuild de CI/CD para un proyecto de U-SQL](./media/data-lake-analytics-cicd-overview/data-lake-analytics-set-vsts-msbuild-variables.png) 

    ```console
    /p:USQLSDKPath=$(Build.SourcesDirectory)/packages/Microsoft.Azure.DataLake.USQL.SDK.1.3.180615/build/runtime /p:USQLTargetType=SyntaxCheck /p:DataRoot=$(Build.SourcesDirectory) /p:EnableDeployment=true
    ```

### <a name="u-sql-project-build-output"></a>Salida de compilación del proyecto de U-SQL

Después de ejecutar una compilación, se compilan todos los scripts del proyecto de U-SQL y la salida se comprime en un archivo zip llamado `USQLProjectName.usqlpack`. La estructura de carpetas del proyecto se mantiene en la salida de la compilación comprimida.

> [!NOTE]
> Los archivos de código subyacente de cada script U-SQL se combinarán como una instrucción insertada en la salida de la compilación del script.

## <a name="test-u-sql-scripts"></a>Prueba de scripts U-SQL

Azure Data Lake proporciona proyectos de prueba para los scripts U-SQL y para UDO/UDAG/UDF de C#:
* Aprenda a [agregar casos de prueba para scripts U-SQL y código de C# extendido](data-lake-analytics-cicd-test.md#test-u-sql-scripts).
* Aprenda a [ejecutar casos de prueba en Azure Pipelines](data-lake-analytics-cicd-test.md#run-test-cases-in-azure-devops).

## <a name="deploy-a-u-sql-job"></a>Implementación de un trabajo de U-SQL

Después de comprobar el código en el proceso de compilación y prueba, puede enviar directamente los trabajos de U-SQL desde Azure Pipelines mediante una tarea de Azure PowerShell. También puede implementar el script en Azure Data Lake Store o Azure Blob Storage, y [ejecutar los trabajos programados con Azure Data Factory](https://docs.microsoft.com/azure/data-factory/transform-data-using-data-lake-analytics).

### <a name="submit-u-sql-jobs-through-azure-pipelines"></a>Envío de trabajos de U-SQL a través de Azure Pipelines

La salida de la compilación del proyecto de U-SQL es un archivo ZIP llamado **USQLProjectName.usqlpack**. El archivo ZIP incluye todos los scripts U-SQL en el proyecto. Puede usar la [tarea de Azure PowerShell](https://docs.microsoft.com/azure/devops/pipelines/tasks/deploy/azure-powershell?view=vsts) en Pipelines con el siguiente script de PowerShell de ejemplo para enviar trabajos de U-SQL directamente desde Azure Pipelines.

```powershell
<#
    This script can be used to submit U-SQL Jobs with given U-SQL project build output(.usqlpack file).
    This will unzip the U-SQL project build output, and submit all scripts one-by-one.

    Note: the code behind file for each U-SQL script will be merged into the built U-SQL script in build output.
          
    Example :
        USQLJobSubmission.ps1 -ADLAAccountName "myadlaaccount" -ArtifactsRoot "C:\USQLProject\bin\debug\" -DegreeOfParallelism 2
#>

param(
    [Parameter(Mandatory=$true)][string]$ADLAAccountName, # ADLA account name to submit U-SQL jobs
    [Parameter(Mandatory=$true)][string]$ArtifactsRoot, # Root folder of U-SQL project build output
    [Parameter(Mandatory=$false)][string]$DegreeOfParallelism = 1
)

function Unzip($USQLPackfile, $UnzipOutput)
{
    $USQLPackfileZip = Rename-Item -Path $USQLPackfile -NewName $([System.IO.Path]::ChangeExtension($USQLPackfile, ".zip")) -Force -PassThru
    Expand-Archive -Path $USQLPackfileZip -DestinationPath $UnzipOutput -Force
    Rename-Item -Path $USQLPackfileZip -NewName $([System.IO.Path]::ChangeExtension($USQLPackfileZip, ".usqlpack")) -Force
}

## Get U-SQL scripts in U-SQL project build output(.usqlpack file)
Function GetUsqlFiles()
{

    $USQLPackfiles = Get-ChildItem -Path $ArtifactsRoot -Include *.usqlpack -File -Recurse -ErrorAction SilentlyContinue

    $UnzipOutput = Join-Path $ArtifactsRoot -ChildPath "UnzipUSQLScripts"

    foreach ($USQLPackfile in $USQLPackfiles)
    {
        Unzip $USQLPackfile $UnzipOutput
    }

    $USQLFiles = Get-ChildItem -Path $UnzipOutput -Include *.usql -File -Recurse -ErrorAction SilentlyContinue

    return $USQLFiles
}

## Submit U-SQL scripts to ADLA account one-by-one
Function SubmitAnalyticsJob()
{
    $usqlFiles = GetUsqlFiles

    Write-Output "$($usqlFiles.Count) jobs to be submitted..."

    # Submit each usql script and wait for completion before moving ahead.
    foreach ($usqlFile in $usqlFiles)
    {
        $scriptName = "[Release].[$([System.IO.Path]::GetFileNameWithoutExtension($usqlFile.fullname))]"

        Write-Output "Submitting job for '{$usqlFile}'"

        $jobToSubmit = Submit-AzDataLakeAnalyticsJob -Account $ADLAAccountName -Name $scriptName -ScriptPath $usqlFile -DegreeOfParallelism $DegreeOfParallelism
        
        LogJobInformation $jobToSubmit
        
        Write-Output "Waiting for job to complete. Job ID:'{$($jobToSubmit.JobId)}', Name: '$($jobToSubmit.Name)' "
        $jobResult = Wait-AzDataLakeAnalyticsJob -Account $ADLAAccountName -JobId $jobToSubmit.JobId  
        LogJobInformation $jobResult
    }
}

Function LogJobInformation($jobInfo)
{
    Write-Output "************************************************************************"
    Write-Output ([string]::Format("Job Id: {0}", $(DefaultIfNull $jobInfo.JobId)))
    Write-Output ([string]::Format("Job Name: {0}", $(DefaultIfNull $jobInfo.Name)))
    Write-Output ([string]::Format("Job State: {0}", $(DefaultIfNull $jobInfo.State)))
    Write-Output ([string]::Format("Job Started at: {0}", $(DefaultIfNull  $jobInfo.StartTime)))
    Write-Output ([string]::Format("Job Ended at: {0}", $(DefaultIfNull  $jobInfo.EndTime)))
    Write-Output ([string]::Format("Job Result: {0}", $(DefaultIfNull $jobInfo.Result)))
    Write-Output "************************************************************************"
}

Function DefaultIfNull($item)
{
    if ($item -ne $null)
    {
        return $item
    }
    return ""
}

Function Main()
{
    Write-Output ([string]::Format("ADLA account: {0}", $ADLAAccountName))
    Write-Output ([string]::Format("Root folde for usqlpack: {0}", $ArtifactsRoot))
    Write-Output ([string]::Format("AU count: {0}", $DegreeOfParallelism))

    Write-Output "Starting USQL script deployment..."
    
    SubmitAnalyticsJob

    Write-Output "Finished deployment..."
}

Main
```

>[!NOTE]
> Los comandos: tanto `Submit-AzDataLakeAnalyticsJob` como `Wait-AzDataLakeAnalyticsJob` son cmdlets de Azure PowerShell para Azure Data Lake Analytics en el marco de Azure Resource Manager. Necesitará una estación de trabajo con Azure PowerShell instalado. Puede consultar la [lista de comandos](https://docs.microsoft.com/powershell/module/Az.DataLakeAnalytics/?view=azps-4.3.0) para ver más comandos y ejemplos.
>

### <a name="deploy-u-sql-jobs-through-azure-data-factory"></a>Implementación de trabajos de U-SQL a través de Azure Data Factory

Puede enviar trabajos de U-SQL directamente desde Azure Pipelines. También puede cargar scripts en Azure Data Lake Store o Azure Blob Storage, y [ejecutar los trabajos programados con Azure Data Factory](https://docs.microsoft.com/azure/data-factory/transform-data-using-data-lake-analytics).

Use la [tarea de Azure PowerShell](https://docs.microsoft.com/azure/devops/pipelines/tasks/deploy/azure-powershell?view=vsts) en Azure Pipelines con el siguiente script de PowerShell de ejemplo para cargar los scripts U-SQL en una cuenta de Azure Data Lake Store:

```powershell
<#
    This script can be used to upload U-SQL files to ADLS with given U-SQL project build output(.usqlpack file).
    This will unzip the U-SQL project build output, and upload all scripts to ADLS one-by-one.
          
    Example :
        FileUpload.ps1 -ADLSName "myadlsaccount" -ArtifactsRoot "C:\USQLProject\bin\debug\"
#>

param(
    [Parameter(Mandatory=$true)][string]$ADLSName, # ADLS account name to upload U-SQL scripts
    [Parameter(Mandatory=$true)][string]$ArtifactsRoot, # Root folder of U-SQL project build output
    [Parameter(Mandatory=$false)][string]$DestinationFolder = "USQLScriptSource" # Destination folder in ADLS
)

Function UploadResources()
{
    Write-Host "************************************************************************"
    Write-Host "Uploading files to $ADLSName"
    Write-Host "***********************************************************************"

    $usqlScripts = GetUsqlFiles

    $files = @(get-childitem $usqlScripts -recurse)
    foreach($file in $files)
    {
        Write-Host "Uploading file: $($file.Name)"
        Import-AzDataLakeStoreItem -AccountName $ADLSName -Path $file.FullName -Destination "/$(Join-Path $DestinationFolder $file)" -Force
    }
}

function Unzip($USQLPackfile, $UnzipOutput)
{
    $USQLPackfileZip = Rename-Item -Path $USQLPackfile -NewName $([System.IO.Path]::ChangeExtension($USQLPackfile, ".zip")) -Force -PassThru
    Expand-Archive -Path $USQLPackfileZip -DestinationPath $UnzipOutput -Force
    Rename-Item -Path $USQLPackfileZip -NewName $([System.IO.Path]::ChangeExtension($USQLPackfileZip, ".usqlpack")) -Force
}

Function GetUsqlFiles()
{

    $USQLPackfiles = Get-ChildItem -Path $ArtifactsRoot -Include *.usqlpack -File -Recurse -ErrorAction SilentlyContinue

    $UnzipOutput = Join-Path $ArtifactsRoot -ChildPath "UnzipUSQLScripts"

    foreach ($USQLPackfile in $USQLPackfiles)
    {
        Unzip $USQLPackfile $UnzipOutput
    }

    return Get-ChildItem -Path $UnzipOutput -Include *.usql -File -Recurse -ErrorAction SilentlyContinue
}

UploadResources
```

## <a name="cicd-for-a-u-sql-database"></a>CI/CD para una base de datos U-SQL

Herramientas de Azure Data Lake para Visual Studio proporciona plantillas de proyecto de base de datos U-SQL que ayudan a desarrollar, administrar e implementar las bases de datos U-SQL de forma rápida y sencilla. Más información acerca de un [proyecto de base de datos U-SQL](data-lake-analytics-data-lake-tools-develop-usql-database.md).

## <a name="build-u-sql-database-project"></a>Compilación de un proyecto de base de datos de U-SQL

### <a name="get-the-nuget-package"></a>Obtención del paquete NuGet

MSBuild no proporciona compatibilidad integrada con los proyectos de base de datos U-SQL. Para obtener esta capacidad, es preciso agregar una referencia para la solución al paquete NuGet [Microsoft.Azure.DataLake.USQL.SDK](https://www.nuget.org/packages/Microsoft.Azure.DataLake.USQL.SDK/) que agregue el servicio de lenguaje requerido.

Para agregar dicha referencia al paquete NuGet, haga clic con el botón derecho en la solución en el Explorador de soluciones de Visual Studio. Elija **Administrar paquetes NuGet**. A continuación, busque e instale el paquete NuGet. O bien puede agregar un archivo llamado **packages.config** a la carpeta de la solución y agregarle el contenido siguiente:

```xml
<?xml version="1.0" encoding="utf-8"?>
<packages>
  <package id="Microsoft.Azure.DataLake.USQL.SDK" version="1.3.180615" targetFramework="net452" />
</packages>
```

### <a name="build-u-sql-a-database-project-with-the-msbuild-command-line"></a>Compilación de un proyecto de base de datos U-SQL con la línea de comandos de MSBuild

Para compilar un proyecto de base de datos U-SQL, llame a la línea de comandos de MSBuild estándar y pase la referencia del paquete NuGet del SDK de U-SQL como argumento adicional. Observe el ejemplo siguiente: 

```console
msbuild DatabaseProject.usqldbproj /p:USQLSDKPath=packages\Microsoft.Azure.DataLake.USQL.SDK.1.3.180615\build\runtime
```

El argumento `USQLSDKPath=<U-SQL Nuget package>\build\runtime` hace referencia a la ruta de instalación del paquete NuGet para el servicio de lenguaje de U-SQL.

### <a name="continuous-integration-with-azure-pipelines"></a>Integración continua con Azure Pipelines

Además de la línea de comandos, puede utilizar Visual Studio Build o una tarea de MSBuild para compilar proyectos de base de datos U-SQL en Azure Pipelines. Para configurar una tarea de compilación, asegúrese de agregar dos tareas en la canalización de compilación: una tarea de restauración de NuGet y una tarea de MSBuild.

   ![Tarea de MSBuild de CI/CD para un proyecto de U-SQL](./media/data-lake-analytics-cicd-overview/data-lake-analytics-set-vsts-msbuild-task.png)

1. Agregue una tarea de restauración de NuGet para obtener el paquete NuGet al que hace referencia la solución que incluye `Azure.DataLake.USQL.SDK`, con el fin de que MSBuild pueda encontrar los destinos del lenguaje de U-SQL. Establezca **Opciones avanzadas** > **Directorio de destino** en `$(Build.SourcesDirectory)/packages` si desea usar el ejemplo de argumentos de MSBuild directamente en el paso 2.

   ![Tarea de NuGet de CI/CD para un proyecto de U-SQL](./media/data-lake-analytics-cicd-overview/data-lake-analytics-set-vsts-nuget-task.png)

2. Establezca los argumentos de MSBuild en herramientas de compilación de Visual Studio o en una tarea de MSBuild como se muestra en el ejemplo siguiente. O bien, puede definir variables para estos argumentos en la canalización de compilación de Azure Pipelines.

   ![Definición de variables de MSBuild de CI/CD para un proyecto de base de datos U-SQL](./media/data-lake-analytics-cicd-overview/data-lake-analytics-set-vsts-msbuild-variables-database-project.png)

   ```console
   /p:USQLSDKPath=$(Build.SourcesDirectory)/packages/Microsoft.Azure.DataLake.USQL.SDK.1.3.180615/build/runtime
   ```

### <a name="u-sql-database-project-build-output"></a>Salida de compilación de un proyecto de base de datos de U-SQL

La salida de compilación del proyecto de base de datos U-SQL es un paquete de implementación de la base de datos U-SQL, denominado con el sufijo `.usqldbpack`. El paquete `.usqldbpack` es un archivo ZIP que incluye todas las instrucciones DDL de un único script U-SQL en una carpeta DDL. Incluye todos los archivos **.dll** y adicionales para el ensamblado en una carpeta temporal.

## <a name="test-table-valued-functions-and-stored-procedures"></a>Prueba de las funciones con valores de tabla y de los procedimientos almacenados

En este momento, no se admite la adición directa de casos de prueba a los procedimientos almacenados y las funciones con valores de tabla. Como alternativa, puede crear un proyecto de U-SQL que tenga scripts U-SQL que llamen a dichas funciones y escribir casos de prueba para ellos. Siga estos pasos para configurar casos de prueba para las funciones con valores de tabla y para los procedimientos almacenados definidos en el proyecto de base de datos U-SQL:

1.  Cree un proyecto de U-SQL para realizar las pruebas y escriba scripts U-SQL que llamen a las funciones con valores de tabla y a los procedimientos almacenados.
2.  Agregue una referencia de base de datos a este proyecto de U-SQL. Para obtener la definición de la función con valores de tabla y del procedimiento almacenado, es preciso hacer referencia al proyecto de base de datos que contiene la instrucción de DDL. Más información acerca de [las referencias de base de datos](data-lake-analytics-data-lake-tools-develop-usql-database.md#reference-a-u-sql-database-project).
3.  Agregue casos de prueba para los scripts U-SQL que llamen a procedimientos almacenados y a funciones con valores de tabla. Aprenda a [agregar casos de prueba para scripts U-SQL](data-lake-analytics-cicd-test.md#test-u-sql-scripts).

## <a name="deploy-u-sql-database-through-azure-pipelines"></a>Implementación de una base de datos U-SQL mediante Azure Pipelines

`PackageDeploymentTool.exe` proporciona interfaces de programación y de línea de comandos que ayudan a implementar paquetes de implementación de bases de datos U-SQL **.usqldbpack**. El SDK se incluye en el [paquete NuGet del SDK de U-SQL](https://www.nuget.org/packages/Microsoft.Azure.DataLake.USQL.SDK/), que se encuentra en **build/runtime/PackageDeploymentTool.exe**. Mediante el uso de `PackageDeploymentTool.exe`, puede implementar bases de datos U-SQL tanto para cuentas locales como para Azure Data Lake Analytics.

> [!NOTE]
>
> La compatibilidad de la línea de comandos de PowerShell y de la tarea de liberación de Azure Pipelines para la implementación de la base de datos U-SQL está pendiente en este momento.
>

Siga estos pasos para configurar una tarea de implementación de la base de datos en Azure Pipelines:

1. Agregue una tarea de PowerShell Script en una canalización de versión o de compilación, y ejecute el siguiente script de PowerShell. Esta tarea ayuda a obtener las dependencias del SDK de Azure para `PackageDeploymentTool.exe` y `PackageDeploymentTool.exe`. Puede establecer los parámetros **-AzureSDK** y **-DBDeploymentTool** para cargar las dependencias y la herramienta de implementación en carpetas específicas. Pase la ruta de acceso **-AzureSDK** a `PackageDeploymentTool.exe` como el parámetro **-AzureSDKPath** en el paso 2. 

    ```powershell
    <#
        This script is used for getting dependencies and SDKs for U-SQL database deployment.
        PowerShell command line support for deploying U-SQL database package(.usqldbpack file) will come soon.

        Example :
            GetUSQLDBDeploymentSDK.ps1 -AzureSDK "AzureSDKFolderPath" -DBDeploymentTool "DBDeploymentToolFolderPath"
    #>

    param (
        [string]$AzureSDK = "AzureSDK", # Folder to cache Azure SDK dependencies
        [string]$DBDeploymentTool = "DBDeploymentTool", # Folder to cache U-SQL database deployment tool
        [string]$workingfolder = "" # Folder to execute these command lines
    )

    if ([string]::IsNullOrEmpty($workingfolder))
    {
        $scriptpath = $MyInvocation.MyCommand.Path
        $workingfolder = Split-Path $scriptpath
    }
    cd $workingfolder

    echo "workingfolder=$workingfolder, outputfolder=$outputfolder"
    echo "Downloading required packages..."

    iwr https://www.nuget.org/api/v2/package/Microsoft.Azure.Management.DataLake.Analytics/3.5.1-preview -outf Microsoft.Azure.Management.DataLake.Analytics.3.5.1-preview.zip
    iwr https://www.nuget.org/api/v2/package/Microsoft.Azure.Management.DataLake.Store/2.4.1-preview -outf Microsoft.Azure.Management.DataLake.Store.2.4.1-preview.zip
    iwr https://www.nuget.org/api/v2/package/Microsoft.IdentityModel.Clients.ActiveDirectory/2.28.3 -outf Microsoft.IdentityModel.Clients.ActiveDirectory.2.28.3.zip
    iwr https://www.nuget.org/api/v2/package/Microsoft.Rest.ClientRuntime/2.3.11 -outf Microsoft.Rest.ClientRuntime.2.3.11.zip
    iwr https://www.nuget.org/api/v2/package/Microsoft.Rest.ClientRuntime.Azure/3.3.7 -outf Microsoft.Rest.ClientRuntime.Azure.3.3.7.zip
    iwr https://www.nuget.org/api/v2/package/Microsoft.Rest.ClientRuntime.Azure.Authentication/2.3.3 -outf Microsoft.Rest.ClientRuntime.Azure.Authentication.2.3.3.zip
    iwr https://www.nuget.org/api/v2/package/Newtonsoft.Json/6.0.8 -outf Newtonsoft.Json.6.0.8.zip
    iwr https://www.nuget.org/api/v2/package/Microsoft.Azure.DataLake.USQL.SDK/ -outf USQLSDK.zip

    echo "Extracting packages..."

    Expand-Archive Microsoft.Azure.Management.DataLake.Analytics.3.5.1-preview.zip -DestinationPath Microsoft.Azure.Management.DataLake.Analytics.3.5.1-preview -Force
    Expand-Archive Microsoft.Azure.Management.DataLake.Store.2.4.1-preview.zip -DestinationPath Microsoft.Azure.Management.DataLake.Store.2.4.1-preview -Force
    Expand-Archive Microsoft.IdentityModel.Clients.ActiveDirectory.2.28.3.zip -DestinationPath Microsoft.IdentityModel.Clients.ActiveDirectory.2.28.3 -Force
    Expand-Archive Microsoft.Rest.ClientRuntime.2.3.11.zip -DestinationPath Microsoft.Rest.ClientRuntime.2.3.11 -Force
    Expand-Archive Microsoft.Rest.ClientRuntime.Azure.3.3.7.zip -DestinationPath Microsoft.Rest.ClientRuntime.Azure.3.3.7 -Force
    Expand-Archive Microsoft.Rest.ClientRuntime.Azure.Authentication.2.3.3.zip -DestinationPath Microsoft.Rest.ClientRuntime.Azure.Authentication.2.3.3 -Force
    Expand-Archive Newtonsoft.Json.6.0.8.zip -DestinationPath Newtonsoft.Json.6.0.8 -Force
    Expand-Archive USQLSDK.zip -DestinationPath USQLSDK -Force

    echo "Copy required DLLs to output folder..."

    mkdir $AzureSDK -Force
    mkdir $DBDeploymentTool -Force
    copy Microsoft.Azure.Management.DataLake.Analytics.3.5.1-preview\lib\net452\*.dll $AzureSDK
    copy Microsoft.Azure.Management.DataLake.Store.2.4.1-preview\lib\net452\*.dll $AzureSDK
    copy Microsoft.IdentityModel.Clients.ActiveDirectory.2.28.3\lib\net45\*.dll $AzureSDK
    copy Microsoft.Rest.ClientRuntime.2.3.11\lib\net452\*.dll $AzureSDK
    copy Microsoft.Rest.ClientRuntime.Azure.3.3.7\lib\net452\*.dll $AzureSDK
    copy Microsoft.Rest.ClientRuntime.Azure.Authentication.2.3.3\lib\net452\*.dll $AzureSDK
    copy Newtonsoft.Json.6.0.8\lib\net45\*.dll $AzureSDK
    copy USQLSDK\build\runtime\*.* $DBDeploymentTool
    ```

2. Llame a `PackageDeploymentTool.exe` para agregar una **tarea de línea de comandos** en una canalización de versión o de compilación, y rellenar el script. `PackageDeploymentTool.exe` se encuentra bajo la carpeta **$DBDeploymentTool** definida. Este es el script de ejemplo: 

    * Implementación local de una base de datos U-SQL:

        ```
        PackageDeploymentTool.exe deploylocal -Package <package path> -Database <database name> -DataRoot <data root path>
        ```

    * Use el modo de autenticación interactiva para implementar una base de datos U-SQL en una cuenta de Azure Data Lake Analytics:

        ```
        PackageDeploymentTool.exe deploycluster -Package <package path> -Database <database name> -Account <account name> -ResourceGroup <resource group name> -SubscriptionId <subscript id> -Tenant <tenant name> -AzureSDKPath <azure sdk path> -Interactive
        ```

    * Use la autenticación **secrete** para implementar una base de datos U-SQL en una cuenta de Azure Data Lake Analytics:

        ```
        PackageDeploymentTool.exe deploycluster -Package <package path> -Database <database name> -Account <account name> -ResourceGroup <resource group name> -SubscriptionId <subscript id> -Tenant <tenant name> -ClientId <client id> -Secrete <secrete>
        ```

    * Use el modo de autenticación **certFile** para implementar una base de datos U-SQL en una cuenta de Azure Data Lake Analytics:

        ```
        PackageDeploymentTool.exe deploycluster -Package <package path> -Database <database name> -Account <account name> -ResourceGroup <resource group name> -SubscriptionId <subscript id> -Tenant <tenant name> -ClientId <client id> -Secrete <secrete> -CertFile <certFile>
        ```

### <a name="packagedeploymenttoolexe-parameter-descriptions"></a>Descripciones de parámetros de PackageDeploymentTool.exe

#### <a name="common-parameters"></a>Parámetros comunes

| Parámetro | Descripción | Valor predeterminado | Obligatorio |
|---------|-----------|-------------|--------|
|Paquete|Ruta de acceso del paquete de implementación de la base de datos U-SQL que se va a implementar.|null|true|
|Base de datos|Nombre de base de datos que se va a implementar o crear.|maestro|false|
|LogFile|Ruta de acceso del archivo para registro. Salida estándar de forma predeterminada (consola).|null|false|
|LogLevel|Nivel de registro: Detallado, Normal, Advertencia o Error.|LogLevel.Normal|false|

#### <a name="parameter-for-local-deployment"></a>Parámetro para la implementación local

|Parámetro|Descripción|Valor predeterminado|Obligatorio|
|---------|-----------|-------------|--------|
|DataRoot|Ruta de acceso de la carpeta raíz de datos local.|null|true|

#### <a name="parameters-for-azure-data-lake-analytics-deployment"></a>Parámetros para la implementación de Azure Data Lake Analytics

|Parámetro|Descripción|Valor predeterminado|Obligatorio|
|---------|-----------|-------------|--------|
|Cuenta|Especifica la cuenta de Azure Data Lake Analytics en la que se realiza la implementación, por número de cuenta.|null|true|
|ResourceGroup|Nombre del grupo de recursos de Azure para la cuenta de Azure Data Lake Analytics.|null|true|
|SubscriptionId|Identificador de la suscripción de Azure para la cuenta de Azure Data Lake Analytics.|null|true|
|Inquilino|El nombre del inquilino es el nombre de dominio de Azure Active Directory (Azure AD). Encuéntrelo en la página de administración de la suscripción en Azure Portal.|null|true|
|AzureSDKPath|Ruta de acceso para buscar los ensamblados dependientes en el SDK de Azure.|null|true|
|Interactive|Si se usa o no el modo interactivo para la autenticación.|false|false|
|ClientId|El identificador de aplicación de Azure AD necesario para la autenticación no interactiva.|null|Se requiere para la autenticación no interactiva.|
|Secrete|El secreto o la contraseña para la autenticación no interactiva. Se debe usar únicamente en un entorno seguro y de confianza.|null|Se requiere para la autenticación no interactiva, o bien use SecreteFile.|
|SecreteFile|El archivo guarda el secreto o la contraseña para la autenticación no interactiva. Asegúrese de mantenerlo legible solo para el usuario actual.|null|Se requiere para la autenticación no interactiva, o bien use Secrete.|
|CertFile|El archivo guarda los certificados X.509 para la autenticación no interactiva. El valor predeterminado es usar la autenticación secreta del cliente.|null|false|
| JobPrefix | El prefijo para la implementación de la base de datos de un trabajo de DDL de U-SQL. | Deploy_ + DateTime.Now | false |

## <a name="next-steps"></a>Pasos siguientes

- [Procedimiento para probar el código de Azure Data Lake Analytics](data-lake-analytics-cicd-test.md).
- [Ejecución del script U-SQL en la máquina local](data-lake-analytics-data-lake-tools-local-run.md).
- [Uso del proyecto de base de datos U-SQL para desarrollar la base de datos U-SQL](data-lake-analytics-data-lake-tools-develop-usql-database.md).
