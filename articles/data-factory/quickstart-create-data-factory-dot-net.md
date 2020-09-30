---
title: Crear una factoría de datos de Azure con SDK de .NET
description: Cree una factoría de datos de Azure para copiar los datos de una ubicación de Azure Blob Storage a otra.
services: data-factory
documentationcenter: ''
author: linda33wj
manager: shwang
ms.reviewer: douglasl
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: ''
ms.devlang: dotnet
ms.topic: quickstart
ms.date: 06/24/2019
ms.author: jingwang
ms.openlocfilehash: d4652ca62a82d14c7e44466161ae9b463818a71f
ms.sourcegitcommit: bf1340bb706cf31bb002128e272b8322f37d53dd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/03/2020
ms.locfileid: "89433989"
---
# <a name="quickstart-create-a-data-factory-and-pipeline-using-net-sdk"></a>Inicio rápido: Creación de una factoría de datos y una canalización con SDK de .NET

> [!div class="op_single_selector" title1="Seleccione la versión del servicio Data Factory que usa:"]
> * [Versión 1](v1/data-factory-copy-data-from-azure-blob-storage-to-sql-database.md)
> * [Versión actual](quickstart-create-data-factory-dot-net.md)

[!INCLUDE[appliesto-adf-xxx-md](includes/appliesto-adf-xxx-md.md)]

En esta guía de inicio rápido se describe cómo usar SDK de .NET para crear una instancia de Azure Data Factory. La canalización que ha creado en esta factoría de datos **copia** los datos de una carpeta a otra en Azure Blob Storage. Para ver un tutorial acerca de cómo **transformar** datos mediante Azure Data Factory, consulte [Tutorial: Transformación de datos con Spark](tutorial-transform-data-spark-portal.md).

> [!NOTE]
> En este artículo no se ofrece una introducción detallada al servicio Data Factory. Para ver una introducción al servicio Azure Data Factory, consulte [Introducción al servicio Azure Data Factory](introduction.md).

[!INCLUDE [data-factory-quickstart-prerequisites](../../includes/data-factory-quickstart-prerequisites.md)] 

### <a name="visual-studio"></a>Visual Studio

En el tutorial de este artículo se usa Visual Studio 2019. Los procedimientos para Visual Studio 2013, 2015 o 2017 son ligeramente diferentes.

### <a name="azure-net-sdk"></a>SDK de .NET para Azure

Descargue e instale el [SDK de .NET para Azure](https://azure.microsoft.com/downloads/) en la máquina.

## <a name="create-an-application-in-azure-active-directory"></a>Creación de una aplicación en Azure Active Directory

En las secciones de *Instrucciones: Uso del portal para crear una aplicación de Azure AD y una entidad de servicio con acceso a los recursos*, siga las instrucciones para realizar las siguientes tareas:

1. En [Crear una aplicación de Azure Active Directory](../active-directory/develop/howto-create-service-principal-portal.md#register-an-application-with-azure-ad-and-create-a-service-principal), cree una aplicación que represente la aplicación .NET que se va a crear en este tutorial. Para la dirección URL de inicio de sesión, puede proporcionar una dirección URL ficticia, tal como se muestra en el artículo (`https://contoso.org/exampleapp`).
2. En [Obtener valores para iniciar sesión](../active-directory/develop/howto-create-service-principal-portal.md#get-tenant-and-app-id-values-for-signing-in), obtenga el **identificador de aplicación** y el **identificador de inquilino** y tome nota de estos valores que se usa más adelante en este tutorial. 
3. En [Certificados y secretos](../active-directory/develop/howto-create-service-principal-portal.md#upload-a-certificate-or-create-a-secret-for-signing-in), obtenga la **clave de autenticación** y tome nota de este valor que se usa más adelante en este tutorial.
4. En [Asignar la aplicación a un rol](../active-directory/develop/howto-create-service-principal-portal.md#assign-a-role-to-the-application), asigne la aplicación al rol **Colaborador** en el nivel de suscripción para que la aplicación pueda crear factorías de datos en la suscripción.

## <a name="create-a-visual-studio-project"></a>Creación de un proyecto de Visual Studio

A continuación, cree una aplicación de consola .NET de C# en Visual Studio:

1. Inicie **Visual Studio**.
2. En la ventana Inicio, seleccione **Crear un nuevo proyecto** > **Aplicación de consola (.NET Framework)** . Se requiere .NET versión 4.5.2 o posterior.
3. En **Nombre del proyecto**, escriba **ADFv2QuickStart**.
4. Seleccione **Crear** para crear el proyecto.

## <a name="install-nuget-packages"></a>Instalación de paquetes NuGet

1. Seleccione **Herramientas** > **Administrador de paquetes NuGet** > **Consola del Administrador de paquetes**.
2. En el panel **Consola del Administrador de paquetes**, ejecute los comandos siguientes para instalar los paquetes. Para obtener más información, consulte el [paquete NuGet Microsoft.Azure.Management.DataFactory](https://www.nuget.org/packages/Microsoft.Azure.Management.DataFactory/).

    ```powershell
    Install-Package Microsoft.Azure.Management.DataFactory
    Install-Package Microsoft.Azure.Management.ResourceManager -IncludePrerelease
    Install-Package Microsoft.IdentityModel.Clients.ActiveDirectory
    ```

## <a name="create-a-data-factory-client"></a>Creación de un cliente de factoría de datos

1. Abra **Program.cs** e incluya las siguientes instrucciones para agregar referencias a espacios de nombres.

    ```csharp
    using System;
    using System.Collections.Generic;
    using System.Linq;
    using Microsoft.Rest;
    using Microsoft.Azure.Management.ResourceManager;
    using Microsoft.Azure.Management.DataFactory;
    using Microsoft.Azure.Management.DataFactory.Models;
    using Microsoft.IdentityModel.Clients.ActiveDirectory;
    ```

2. Agregue el siguiente código al método **main** que define las variables. Reemplace los marcadores de posición por sus propios valores. Para una lista de las regiones de Azure en las que Data Factory está disponible actualmente, seleccione las regiones que le interesen en la página siguiente y expanda **Análisis** para poder encontrar **Data Factory**: [Productos disponibles por región](https://azure.microsoft.com/global-infrastructure/services/). Los almacenes de datos (Azure Storage, Azure SQL Database, etc.) y los procesos (HDInsight, etc.) que usa la factoría de datos pueden encontrarse en otras regiones.

   ```csharp
   // Set variables
   string tenantID = "<your tenant ID>";
   string applicationId = "<your application ID>";
   string authenticationKey = "<your authentication key for the application>";
   string subscriptionId = "<your subscription ID where the data factory resides>";
   string resourceGroup = "<your resource group where the data factory resides>";
   string region = "<the location of your resource group>";
   string dataFactoryName = 
       "<specify the name of data factory to create. It must be globally unique.>";
   string storageAccount = "<your storage account name to copy data>";
   string storageKey = "<your storage account key>";
   // specify the container and input folder from which all files 
   // need to be copied to the output folder. 
   string inputBlobPath =
       "<path to existing blob(s) to copy data from, e.g. containername/inputdir>";
   //specify the contains and output folder where the files are copied
   string outputBlobPath =
       "<the blob path to copy data to, e.g. containername/outputdir>";

   // name of the Azure Storage linked service, blob dataset, and the pipeline
   string storageLinkedServiceName = "AzureStorageLinkedService";
   string blobDatasetName = "BlobDataset";
   string pipelineName = "Adfv2QuickStartPipeline";
   ```

3. Agregue el código siguiente al método **Main** que crea una instancia de la clase **DataFactoryManagementClient**. Este objeto se usa para crear una factoría de datos, un servicio vinculado, conjuntos de datos y una canalización. También se usa para supervisar los detalles de ejecución de la canalización.

   ```csharp
   // Authenticate and create a data factory management client
   var context = new AuthenticationContext("https://login.windows.net/" + tenantID);
   ClientCredential cc = new ClientCredential(applicationId, authenticationKey);
   AuthenticationResult result = context.AcquireTokenAsync(
       "https://management.azure.com/", cc).Result;
   ServiceClientCredentials cred = new TokenCredentials(result.AccessToken);
   var client = new DataFactoryManagementClient(cred) {
       SubscriptionId = subscriptionId };
   ```

## <a name="create-a-data-factory"></a>Crear una factoría de datos

Agregue el código siguiente al método **main** que crea una **factoría de datos**. 

```csharp
// Create a data factory
Console.WriteLine("Creating data factory " + dataFactoryName + "...");
Factory dataFactory = new Factory
{
    Location = region,
    Identity = new FactoryIdentity()
};
client.Factories.CreateOrUpdate(resourceGroup, dataFactoryName, dataFactory);
Console.WriteLine(
    SafeJsonConvert.SerializeObject(dataFactory, client.SerializationSettings));

while (client.Factories.Get(resourceGroup, dataFactoryName).ProvisioningState ==
       "PendingCreation")
{
    System.Threading.Thread.Sleep(1000);
}
```

## <a name="create-a-linked-service"></a>Creación de un servicio vinculado

Agregue el código siguiente al método **main** que crea un **servicio vinculado de Azure Storage**.

Los servicios vinculados se crean en una factoría de datos para vincular los almacenes de datos y los servicios de proceso con la factoría de datos. En esta guía de inicio rápido, basta con crear un servicio vinculado de Azure Storage para copiar el almacén de origen y el receptor. En el ejemplo, se denomina "AzureStorageLinkedService".

```csharp
// Create an Azure Storage linked service
Console.WriteLine("Creating linked service " + storageLinkedServiceName + "...");

LinkedServiceResource storageLinkedService = new LinkedServiceResource(
    new AzureStorageLinkedService
    {
        ConnectionString = new SecureString(
            "DefaultEndpointsProtocol=https;AccountName=" + storageAccount +
            ";AccountKey=" + storageKey)
    }
);
client.LinkedServices.CreateOrUpdate(
    resourceGroup, dataFactoryName, storageLinkedServiceName, storageLinkedService);
Console.WriteLine(SafeJsonConvert.SerializeObject(
    storageLinkedService, client.SerializationSettings));
```

## <a name="create-a-dataset"></a>Crear un conjunto de datos

Agregue el código siguiente al método **Main** que crea un **conjunto de datos de blob de Azure**.

Se define un conjunto de datos que representa los datos que se copian de un origen a un receptor. En este ejemplo, este conjunto de datos de blob hace referencia al servicio vinculado de Azure Storage que creó en el paso anterior. En el conjunto de datos se usa un parámetro cuyo valor se establece en una actividad que consume el conjunto de datos. El parámetro se usa para construir la ruta "folderPath" que apunta a la ubicación o al almacén de los datos.

```csharp
// Create an Azure Blob dataset
Console.WriteLine("Creating dataset " + blobDatasetName + "...");
DatasetResource blobDataset = new DatasetResource(
    new AzureBlobDataset
    {
        LinkedServiceName = new LinkedServiceReference
        {
            ReferenceName = storageLinkedServiceName
        },
        FolderPath = new Expression { Value = "@{dataset().path}" },
        Parameters = new Dictionary<string, ParameterSpecification>
        {
            { "path", new ParameterSpecification { Type = ParameterType.String } }
        }
    }
);
client.Datasets.CreateOrUpdate(
    resourceGroup, dataFactoryName, blobDatasetName, blobDataset);
Console.WriteLine(
    SafeJsonConvert.SerializeObject(blobDataset, client.SerializationSettings));
```

## <a name="create-a-pipeline"></a>Crear una canalización

Agregue el código siguiente al método **main** que crea una **canalización con una actividad de copia**.

En este ejemplo, esta canalización contiene una actividad y se usan dos parámetros: la ruta de acceso de blob de entrada y la de salida. Los valores para estos parámetros se establecen cuando se desencadena/ejecuta la canalización. La actividad de copia hace referencia al mismo conjunto de datos de blob creado en el paso anterior como entrada y salida. Cuando se usa el conjunto de datos como conjunto de datos de entrada, se especifica una ruta de acceso de entrada. Cuando se usa el conjunto de datos como conjunto de datos de salida, se especifica una ruta de acceso de salida. 

```csharp
// Create a pipeline with a copy activity
Console.WriteLine("Creating pipeline " + pipelineName + "...");
PipelineResource pipeline = new PipelineResource
{
    Parameters = new Dictionary<string, ParameterSpecification>
    {
        { "inputPath", new ParameterSpecification { Type = ParameterType.String } },
        { "outputPath", new ParameterSpecification { Type = ParameterType.String } }
    },
    Activities = new List<Activity>
    {
        new CopyActivity
        {
            Name = "CopyFromBlobToBlob",
            Inputs = new List<DatasetReference>
            {
                new DatasetReference()
                {
                    ReferenceName = blobDatasetName,
                    Parameters = new Dictionary<string, object>
                    {
                        { "path", "@pipeline().parameters.inputPath" }
                    }
                }
            },
            Outputs = new List<DatasetReference>
            {
                new DatasetReference
                {
                    ReferenceName = blobDatasetName,
                    Parameters = new Dictionary<string, object>
                    {
                        { "path", "@pipeline().parameters.outputPath" }
                    }
                }
            },
            Source = new BlobSource { },
            Sink = new BlobSink { }
        }
    }
};
client.Pipelines.CreateOrUpdate(resourceGroup, dataFactoryName, pipelineName, pipeline);
Console.WriteLine(SafeJsonConvert.SerializeObject(pipeline, client.SerializationSettings));
```

## <a name="create-a-pipeline-run"></a>Creación de una ejecución de canalización

Agregue el código siguiente al método **Main** que **desencadena una ejecución de canalización**.

Este código también establece los valores de los parámetros **inputPath** y **outputPath** especificados en la canalización con los valores reales de las rutas de acceso de blob de origen y receptor.

```csharp
// Create a pipeline run
Console.WriteLine("Creating pipeline run...");
Dictionary<string, object> parameters = new Dictionary<string, object>
{
    { "inputPath", inputBlobPath },
    { "outputPath", outputBlobPath }
};
CreateRunResponse runResponse = client.Pipelines.CreateRunWithHttpMessagesAsync(
    resourceGroup, dataFactoryName, pipelineName, parameters: parameters
).Result.Body;
Console.WriteLine("Pipeline run ID: " + runResponse.RunId);
```

## <a name="monitor-a-pipeline-run"></a>Supervisar una ejecución de canalización

1. Agregue el código siguiente al método **Main** para comprobar continuamente el estado hasta que termine de copiar los datos.

   ```csharp
   // Monitor the pipeline run
   Console.WriteLine("Checking pipeline run status...");
   PipelineRun pipelineRun;
   while (true)
   {
       pipelineRun = client.PipelineRuns.Get(
           resourceGroup, dataFactoryName, runResponse.RunId);
       Console.WriteLine("Status: " + pipelineRun.Status);
       if (pipelineRun.Status == "InProgress" || pipelineRun.Status == "Queued")
           System.Threading.Thread.Sleep(15000);
       else
           break;
   }
   ```

2. Agregue el código siguiente al método **Main** que recupera detalles de la ejecución de actividad de copia, como el tamaño de los datos que se leen o escriben.

   ```csharp
   // Check the copy activity run details
   Console.WriteLine("Checking copy activity run details...");

   RunFilterParameters filterParams = new RunFilterParameters(
       DateTime.UtcNow.AddMinutes(-10), DateTime.UtcNow.AddMinutes(10));
   ActivityRunsQueryResponse queryResponse = client.ActivityRuns.QueryByPipelineRun(
       resourceGroup, dataFactoryName, runResponse.RunId, filterParams);
   if (pipelineRun.Status == "Succeeded")
       Console.WriteLine(queryResponse.Value.First().Output);
   else
       Console.WriteLine(queryResponse.Value.First().Error);
   Console.WriteLine("\nPress any key to exit...");
   Console.ReadKey();
   ```

## <a name="run-the-code"></a>Ejecución del código

Compile e inicie la aplicación y, a continuación, compruebe la ejecución de la canalización.

La consola imprime el progreso de la creación de la factoría de datos, el servicio vinculado, los conjuntos de datos, la canalización y la ejecución de canalización. A continuación, comprueba el estado de la ejecución de canalización. Espere hasta que vea los detalles de ejecución de actividad de copia con el tamaño de los datos leídos o escritos. Luego, use herramientas como el [Explorador de Azure Storage](https://azure.microsoft.com/features/storage-explorer/) para comprobar que los blobs se copian a "outputBlobPath" desde "inputBlobPath", como especificó en las variables.

### <a name="sample-output"></a>Salida de ejemplo

```json
Creating data factory SPv2Factory0907...
{
  "identity": {
    "type": "SystemAssigned"
  },
  "location": "East US"
}
Creating linked service AzureStorageLinkedService...
{
  "properties": {
    "type": "AzureStorage",
    "typeProperties": {
      "connectionString": {
        "value": "DefaultEndpointsProtocol=https;AccountName=<storageAccountName>;AccountKey=<storageAccountKey>",
        "type": "SecureString"
      }
    }
  }
}
Creating dataset BlobDataset...
{
  "properties": {
    "type": "AzureBlob",
    "typeProperties": {
      "folderPath": {
        "value": "@{dataset().path}",
        "type": "Expression"
      }
    },
    "linkedServiceName": {
      "referenceName": "AzureStorageLinkedService",
      "type": "LinkedServiceReference"
    },
    "parameters": {
      "path": {
        "type": "String"
      }
    }
  }
}
Creating pipeline Adfv2QuickStartPipeline...
{
  "properties": {
    "activities": [
      {
        "type": "Copy",
        "typeProperties": {
          "source": {
            "type": "BlobSource"
          },
          "sink": {
            "type": "BlobSink"
          }
        },
        "inputs": [
          {
            "referenceName": "BlobDataset",
            "parameters": {
              "path": "@pipeline().parameters.inputPath"
            },
            "type": "DatasetReference"
          }
        ],
        "outputs": [
          {
            "referenceName": "BlobDataset",
            "parameters": {
              "path": "@pipeline().parameters.outputPath"
            },
            "type": "DatasetReference"
          }
        ],
        "name": "CopyFromBlobToBlob"
      }
    ],
    "parameters": {
      "inputPath": {
        "type": "String"
      },
      "outputPath": {
        "type": "String"
      }
    }
  }
}
Creating pipeline run...
Pipeline run ID: 308d222d-3858-48b1-9e66-acd921feaa09
Checking pipeline run status...
Status: InProgress
Status: InProgress
Checking copy activity run details...
{
    "dataRead": 331452208,
    "dataWritten": 331452208,
    "copyDuration": 23,
    "throughput": 14073.209,
    "errors": [],
    "effectiveIntegrationRuntime": "DefaultIntegrationRuntime (West US)",
    "usedDataIntegrationUnits": 2,
    "billedDuration": 23
}

Press any key to exit...
```

## <a name="verify-the-output"></a>Comprobación del resultado

La canalización crea automáticamente la carpeta de salida en el contenedor de blobs **adftutorial**. A continuación, copia el archivo **emp.txt** de la carpeta de entrada a la carpeta de salida. 

1. En Azure Portal, en la página del contenedor **adftutorial** que detuvo en la sección anterior [Agregar una carpeta de entrada y un archivo para el contenedor de blobs](#add-an-input-folder-and-file-for-the-blob-container), seleccione **Actualizar** para ver la carpeta de salida. 
2. En la lista de carpetas, haga clic en **output**.
3. Confirme que **emp.txt** se ha copiado en la carpeta de salida. 

## <a name="clean-up-resources"></a>Limpieza de recursos

Para eliminar la factoría de datos mediante programación, agregue las líneas de código siguientes al programa: 

```csharp
Console.WriteLine("Deleting the data factory");
client.Factories.Delete(resourceGroup, dataFactoryName);
```

## <a name="next-steps"></a>Pasos siguientes

La canalización de este ejemplo copia los datos de una ubicación a otra en una instancia de Azure Blob Storage. Consulte los [tutoriales](tutorial-copy-data-dot-net.md) para obtener información acerca del uso de Data Factory en otros escenarios. 
