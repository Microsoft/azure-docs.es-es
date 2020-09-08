---
title: Uso del SDK de .NET para trabajos de Microsoft Azure StorSimple Data Manager
description: Aprenda a usar el SDK de .NET en el servicio StorSimple Data Manager para transformar los datos del dispositivo StorSimple.
author: alkohli
ms.service: storsimple
ms.topic: how-to
ms.date: 01/16/2018
ms.author: alkohli
ms.openlocfilehash: e34701640de24a4c3c13f8d10073b4392af2f28a
ms.sourcegitcommit: faeabfc2fffc33be7de6e1e93271ae214099517f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/13/2020
ms.locfileid: "88183656"
---
# <a name="use-the-net-sdk-to-initiate-data-transformation"></a>Uso del SDK de .NET para iniciar la transformación de datos

## <a name="overview"></a>Información general

En este artículo se explica cómo se usa la característica de transformación de datos en StorSimple Data Manager para transformar los datos de los dispositivos StorSimple. Luego, los datos transformados los consumen otros servicios de Azure en la nube.

Puede iniciar un trabajo de transformación de datos de dos maneras:

- Uso del SDK de .NET
- Uso de un runbook de Azure Automation
 
  En este artículo se detalla cómo crear una aplicación de consola .NET de ejemplo para iniciar un trabajo de transformación de datos y, después, realizar su seguimiento para asegurarse de que finaliza. Para más información sobre cómo iniciar la transformación de datos mediante Automation, vaya al artículo sobre cómo [usar un runbook de Azure Automation para desencadenar trabajos de transformación de datos](storsimple-data-manager-job-using-automation.md).

## <a name="prerequisites"></a>Prerequisites

Antes de comenzar, asegúrese de que dispone de:
*   Un equipo que ejecute:

    - Visual Studio 2012, 2013, 2015 o 2017.

    - Azure Powershell. [Descarga de Azure PowerShell](https://azure.microsoft.com/documentation/articles/powershell-install-configure/).
*   Una definición de trabajo configurada correctamente en StorSimple Data Manager dentro de un grupo de recursos.
*   Todos los archivos dll requeridos. Dichos archivos se pueden descargar del [repositorio de GitHub](https://github.com/Azure-Samples/storsimple-dotnet-data-manager-get-started/tree/master/Data_Manager_Job_Run/dlls).
*   El script [`Get-ConfigurationParams.ps1`](https://github.com/Azure-Samples/storsimple-dotnet-data-manager-get-started/blob/master/Data_Manager_Job_Run/Get-ConfigurationParams.ps1) desde el repositorio GitHub.

## <a name="step-by-step-procedure"></a>Procedimiento paso a paso

Realice los pasos siguientes si desea usar .NET para iniciar un trabajo de transformación de datos.

1. Para recuperar los parámetros de configuración, siga estos pasos:
    1. Descargue `Get-ConfigurationParams.ps1` del script del repositorio GitHub que se encuentra en la ubicación `C:\DataTransformation`.
    1. Ejecute el script `Get-ConfigurationParams.ps1` del repositorio GitHub. Escriba el siguiente comando:

        ```
        C:\DataTransformation\Get-ConfigurationParams.ps1 -SubscriptionName "AzureSubscriptionName" -ActiveDirectoryKey "AnyRandomPassword" -AppName "ApplicationName"
         ```
        Puede usar cualquier valor para ActiveDirectoryKey y AppName.

2. Este script genera los siguientes valores:
    * Id. de cliente
    * Id. de inquilino
    * Clave de Active Directory (la misma que escribió anteriormente)
    * Id. de suscripción

        ![Salida del script de los parámetros de configuración](media/storsimple-data-manager-dotnet-jobs/get-config-parameters.png)

3. Con Visual Studio 2012, 2013 o 2015, cree una aplicación de consola .NET de C#.

    1. Inicie **Visual Studio 2012/2013/2015**.
    1. Seleccione **Archivo > Nuevo > Proyecto**.

        ![Creación de un proyecto 1](media/storsimple-data-manager-dotnet-jobs/create-new-project-7.png)        
    2. Seleccione **Instalado > Plantillas > Visual C# > Aplicación de consola**.
    3. Escriba **DataTransformationApp** en **Nombre**.
    4. Seleccione **C:\DataTransformation** en **Ubicación**.
    6. Haga clic en **Aceptar** para crear el proyecto.

        ![Creación de un proyecto 2](media/storsimple-data-manager-dotnet-jobs/create-new-project-1.png)

4. Ahora, agregue todos los archivos DLL de la carpeta [dlls](https://github.com/Azure-Samples/storsimple-dotnet-data-manager-get-started/tree/master/Data_Manager_Job_Run/dlls) como **referencias** en el proyecto que ha creado. Para agregar los archivos DLL, haga lo siguiente:

   1. En Visual Studio, vaya a **Vista > Explorador de soluciones**.
   2. Haga clic en la flecha situada a la izquierda del proyecto de la aplicación de transformación de datos. Haga clic en **Referencias** y, después, haga clic con el botón derecho en **Agregar referencia**.
    
       ![Agregar archivos DLL 1](media/storsimple-data-manager-dotnet-jobs/create-new-project-4.png)

   3. Vaya a la ubicación de la carpeta de paquetes, seleccione todos los archivos DLL, haga clic en **Agregar** y, luego, haga clic en **Aceptar**.

       ![Agregar archivos DLL 2](media/storsimple-data-manager-dotnet-jobs/create-new-project-6.png)

5. Agregue las siguientes instrucciones **using** al archivo de origen (Program.cs) en el proyecto.

    ```
    using System;
    using System.Collections.Generic;
    using System.Threading;
    using Microsoft.Azure.Management.HybridData.Models;
    using Microsoft.Internal.Dms.DmsWebJob;
    using Microsoft.Internal.Dms.DmsWebJob.Contracts;
    ```
    
6. El siguiente código inicializa la instancia del trabajo de transformación de datos. Agréguelo al **Método principal**. Reemplace los valores de los parámetros de configuración que obtuvo antes. Emplee los valores de **ResourceGroupName** y **ResourceName**. El valor **ResourceGroupName** está asociado con la instancia de StorSimple Data Manager en que se configuró la definición del trabajo. El valor **ResourceName** es el nombre del servicio de StorSimple Data Manager.

    ```
    // Setup the configuration parameters.
    var configParams = new ConfigurationParams
    {
        ClientId = "client-id",
        TenantId = "tenant-id",
        ActiveDirectoryKey = "active-directory-key",
        SubscriptionId = "subscription-id",
        ResourceGroupName = "resource-group-name",
        ResourceName = "resource-name"
    };

    // Initialize the Data Transformation Job instance.
    DataTransformationJob dataTransformationJob = new DataTransformationJob(configParams);
    ```
   
7. Especifique los parámetros con los que hay que ejecutar la definición del trabajo

    ```
    string jobDefinitionName = "job-definition-name";

    DataTransformationInput dataTransformationInput = dataTransformationJob.GetJobDefinitionParameters(jobDefinitionName);
    ```

    O BIEN

    Si desea cambiar los parámetros de la definición del trabajo en tiempo de ejecución, agregue después el código siguiente:

    ```
    string jobDefinitionName = "job-definition-name";
    // Must start with a '\'
    var rootDirectories = new List<string> {@"\root"};

    // Name of the volume on the StorSimple device.
    var volumeNames = new List<string> {"volume-name"};

    var dataTransformationInput = new DataTransformationInput
    {
        // If you require the latest existing backup to be picked else use TakeNow to trigger a new backup.
        BackupChoice = BackupChoice.UseExistingLatest.ToString(),
        // Name of the StorSimple device.
        DeviceName = "device-name",
        // Name of the container in Azure storage where the files will be placed after execution.
        ContainerName = "container-name",
        // File name filter (search pattern) to be applied on files under the root directory. * - Match all files.
        FileNameFilter = "*",
        // List of root directories.
        RootDirectories = rootDirectories,
        // Name of the volume on StorSimple device on which the relevant data is present. 
        VolumeNames = volumeNames
    };
    ```

8. Después de la inicialización, agregue el código siguiente para desencadenar un trabajo de transformación de datos en la definición del trabajo. Emplee el **nombre de definición de trabajo** apropiado.

    ```
    // Trigger a job, retrieve the jobId and the retry interval for polling.
    int retryAfter;
    string jobId = dataTransformationJob.RunJobAsync(jobDefinitionName, 
    dataTransformationInput, out retryAfter);
    Console.WriteLine("jobid: ", jobId);
    Console.ReadLine();

    ```
    Una vez que pegue el código, compile la solución. A continuación, aparece una captura de pantalla del fragmento de código para inicializar la instancia del trabajo de transformación de datos.

   ![Fragmento de código para inicializar el trabajo de transformación de datos](media/storsimple-data-manager-dotnet-jobs/start-dotnet-job-code-snippet-1.png)

9. Este trabajo transforma los datos que coinciden con el directorio raíz y los filtros de archivo dentro del volumen de StorSimple y los coloca en el recurso compartido de archivos o el contenedor especificado. Cuando se transforma un archivo, se agrega un mensaje en la cola de almacenamiento (en la misma cuenta de almacenamiento que el recurso compartido o el contenedor) con el mismo nombre que la definición del trabajo. Dicho mensaje se puede usar como desencadenador para iniciar el posterior procesamiento del archivo.

10. Una vez que el trabajo se haya desencadenado, puede usar el código siguiente para realizar un seguimiento de la finalización del mismo. No es obligatorio agregar este código para la ejecución del trabajo.

    ```
    Job jobDetails = null;

    // Poll the job.
    do
    {
        jobDetails = dataTransformationJob.GetJob(jobDefinitionName, jobId);

        // Wait before polling for the status again.
        Thread.Sleep(TimeSpan.FromSeconds(retryAfter));

    } while (jobDetails.Status == JobStatus.InProgress);

    // Completion status of the job.
    Console.WriteLine("JobStatus: {0}", jobDetails.Status);
    
    // To hold the console before exiting.
    Console.Read();

    ```
    Esta es una captura de pantalla del ejemplo de código completo que se usa para desencadenar el trabajo con .NET.

    ![Fragmento de código completo para desencadenar un trabajo de .NET](media/storsimple-data-manager-dotnet-jobs/start-dotnet-job-code-snippet.png)

## <a name="next-steps"></a>Pasos siguientes

[Manage using the StorSimple Data Manager service UI](storsimple-data-manager-ui.md) (Administración del uso de la interfaz de usuario del servicio StorSimple Data Manager).
