---
title: Bifurcación de la canalización de Azure Data Factory
description: Obtenga información sobre cómo controlar el flujo de datos en Azure Data Factory mediante la bifurcación y el encadenamiento de actividades.
services: data-factory
author: djpmsft
ms.author: daperlov
manager: anandsub
ms.reviewer: maghan
ms.service: data-factory
ms.workload: data-services
ms.topic: tutorial
ms.custom: seo-lt-2019; seo-dt-2019
ms.date: 9/27/2019
ms.openlocfilehash: 0330e72ad74726f97bfdfd78ef8d5f9b24a5d172
ms.sourcegitcommit: 62717591c3ab871365a783b7221851758f4ec9a4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/22/2020
ms.locfileid: "85513310"
---
# <a name="branching-and-chaining-activities-in-a-data-factory-pipeline"></a>Actividades de bifurcación y encadenamiento en una canalización de Data Factory

[!INCLUDE[appliesto-adf-xxx-md](includes/appliesto-adf-xxx-md.md)]

En este tutorial creará una canalización de Data Factory que muestra algunas de las características del flujo de control. Esta canalización copia de un contenedor en Azure Blob Storage a otro contenedor en la misma cuenta de almacenamiento. Si la actividad de copia se realiza correctamente, la canalización envía los detalles de la operación de copia correcta en un correo electrónico. Esa información puede incluir la cantidad de datos escritos. Si se produce un error en la actividad de copia, la canalización envía los detalles del error de copia, como el mensaje de error, en un correo electrónico. A lo largo del tutorial, verá cómo pasar parámetros.

En este gráfico se proporciona información general sobre el escenario:

![Información general](media/tutorial-control-flow/overview.png)

En el tutorial se muestra cómo realizar las siguientes tareas:

> [!div class="checklist"]
> * Crear una factoría de datos
> * Creación de un servicio vinculado de Azure Storage
> * Creación de un conjunto de datos del blob de Azure
> * Creación de una canalización que contiene una actividad de copia y una actividad web
> * Envío de los resultados de las actividades en actividades subsiguientes
> * Uso del paso de parámetros y de las variables del sistema
> * Inicio de la ejecución de una canalización
> * Supervisión de las ejecuciones de canalización y actividad

En este tutorial se usa SDK de .NET. Puede usar otros mecanismos para interactuar con Azure Data Factory. Si quiere acceder a inicios rápidos de Data Factory, consulte [Inicios rápidos en 5 minutos](/azure/data-factory/quickstart-create-data-factory-portal).

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="prerequisites"></a>Requisitos previos

* Cuenta de Azure Storage. Usará el almacenamiento de blobs como almacén de datos de origen. Si no dispone de una, consulte [Crear una cuenta de almacenamiento](../storage/common/storage-account-create.md).
* Explorador de Azure Storage Para instalar esta herramienta, consulte [Explorador de Azure Storage](https://storageexplorer.com/).
* Azure SQL Database. Usará la base de datos como un almacén de datos receptor. Si no tiene una base de datos en Azure SQL Database, consulte el artículo [Creación de una base de datos en Azure SQL Database](../azure-sql/database/single-database-create-quickstart.md).
* Visual Studio. En este artículo se utiliza Visual Studio 2019.
* SDK de Azure para .NET. Descargue e instale el [SDK de Azure para .NET](https://azure.microsoft.com/downloads/).

Para obtener una lista de las regiones de Azure en las que Data Factory está disponible actualmente, consulte [Productos disponibles por región](https://azure.microsoft.com/global-infrastructure/services/). Los almacenes de datos y los procesos pueden estar en otras regiones. Los almacenes incluyen Azure Storage y Azure SQL Database. Los procesos incluyen HDInsight, que se usa en Data Factory.

Cree una aplicación tal como se describe en [Creación de una aplicación de Azure Active Directory](../active-directory/develop/howto-create-service-principal-portal.md#register-an-application-with-azure-ad-and-create-a-service-principal). Siga las instrucciones del mismo artículo para asignar la aplicación al rol **Colaborador**. Necesitará varios valores para partes posteriores de este tutorial, como **Id. de aplicación (cliente)** e **Id. de directorio (inquilino)** .

### <a name="create-a-blob-table"></a>Creación de una tabla de blobs

1. Abra un editor de texto. Copie el siguiente texto y guárdelo localmente como *input.txt*.

   ```
   Ethel|Berg
   Tamika|Walsh
   ```

1. Abra el Explorador de Azure Storage. Expanda la cuenta de almacenamiento. Haga clic con el botón derecho en **Contenedores de blob** y seleccione **Crear contenedor de blobs**.
1. Asigne al nuevo contenedor el nombre *adfv2branch* y seleccione **Cargar** para agregar el archivo *input.txt* al contenedor.

## <a name="create-visual-studio-project"></a>Creación de un proyecto de Visual Studio<a name="create-visual-studio-project"></a>

Cree una aplicación de consola .NET de C#:

1. Inicie Visual Studio y seleccione **Crear un proyecto**.
1. En **Crear un proyecto**, elija **Aplicación de consola (.NET Framework)** para C# y seleccione **Siguiente**.
1. Asigne al proyecto el nombre *ADFv2BranchTutorial*.
1. Seleccione **Versión de .NET 4.5.2** o superior y, a continuación, seleccione **Crear**.

### <a name="install-nuget-packages"></a>Instalación de paquetes NuGet

1. Seleccione **Herramientas** > **Administrador de paquetes NuGet** > **Consola del Administrador de paquetes**.
1. En la **Consola del Administrador de paquetes**, ejecute los comandos siguientes para instalar los paquetes. Consulte el [paquete NuGet Microsoft.Azure.Management.DataFactory](https://www.nuget.org/packages/Microsoft.Azure.Management.DataFactory/) para ver los detalles.

   ```powershell
   Install-Package Microsoft.Azure.Management.DataFactory
   Install-Package Microsoft.Azure.Management.ResourceManager -IncludePrerelease
   Install-Package Microsoft.IdentityModel.Clients.ActiveDirectory
   ```

### <a name="create-a-data-factory-client"></a>Creación de un cliente de factoría de datos

1. Abra el archivo *Program.cs* y agregue las siguientes instrucciones:

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

1. Agregue estas variables estáticas a la clase `Program`. Reemplace los marcadores de posición con sus propios valores.

   ```csharp
   // Set variables
   static string tenantID = "<tenant ID>";
   static string applicationId = "<application ID>";
   static string authenticationKey = "<Authentication key for your application>";
   static string subscriptionId = "<Azure subscription ID>";
   static string resourceGroup = "<Azure resource group name>";

   static string region = "East US";
   static string dataFactoryName = "<Data factory name>";

   // Specify the source Azure Blob information
   static string storageAccount = "<Azure Storage account name>";
   static string storageKey = "<Azure Storage account key>";
   // confirm that you have the input.txt file placed in th input folder of the adfv2branch container.
   static string inputBlobPath = "adfv2branch/input";
   static string inputBlobName = "input.txt";
   static string outputBlobPath = "adfv2branch/output";
   static string emailReceiver = "<specify email address of the receiver>";

   static string storageLinkedServiceName = "AzureStorageLinkedService";
   static string blobSourceDatasetName = "SourceStorageDataset";
   static string blobSinkDatasetName = "SinkStorageDataset";
   static string pipelineName = "Adfv2TutorialBranchCopy";

   static string copyBlobActivity = "CopyBlobtoBlob";
   static string sendFailEmailActivity = "SendFailEmailActivity";
   static string sendSuccessEmailActivity = "SendSuccessEmailActivity";
   ```

1. Agregue el siguiente código al método `Main`. Este código crea una instancia de la clase `DataFactoryManagementClient`. Este objeto se usa después para crear la factoría de datos, el servicio vinculado, los conjuntos de datos y la canalización. También se puede usar para supervisar los detalles de ejecución de la canalización.

   ```csharp
   // Authenticate and create a data factory management client
   var context = new AuthenticationContext("https://login.windows.net/" + tenantID);
   ClientCredential cc = new ClientCredential(applicationId, authenticationKey);
   AuthenticationResult result = context.AcquireTokenAsync("https://management.azure.com/", cc).Result;
   ServiceClientCredentials cred = new TokenCredentials(result.AccessToken);
   var client = new DataFactoryManagementClient(cred) { SubscriptionId = subscriptionId };
   ```

### <a name="create-a-data-factory"></a>Crear una factoría de datos

1. Agregue un método `CreateOrUpdateDataFactory` al archivo *Program.cs*:

   ```csharp
   static Factory CreateOrUpdateDataFactory(DataFactoryManagementClient client)
   {
       Console.WriteLine("Creating data factory " + dataFactoryName + "...");
       Factory resource = new Factory
       {
           Location = region
       };
       Console.WriteLine(SafeJsonConvert.SerializeObject(resource, client.SerializationSettings));

       Factory response;
       {
           response = client.Factories.CreateOrUpdate(resourceGroup, dataFactoryName, resource);
       }

       while (client.Factories.Get(resourceGroup, dataFactoryName).ProvisioningState == "PendingCreation")
       {
           System.Threading.Thread.Sleep(1000);
       }
       return response;
   }
   ```

1. Agregue la siguiente línea al método `Main` que crea una factoría de datos:

   ```csharp
   Factory df = CreateOrUpdateDataFactory(client);
   ```

## <a name="create-an-azure-storage-linked-service"></a>Creación de un servicio vinculado de Azure Storage

1. Agregue un método `StorageLinkedServiceDefinition` al archivo *Program.cs*:

   ```csharp
   static LinkedServiceResource StorageLinkedServiceDefinition(DataFactoryManagementClient client)
   {
      Console.WriteLine("Creating linked service " + storageLinkedServiceName + "...");
      AzureStorageLinkedService storageLinkedService = new AzureStorageLinkedService
      {
          ConnectionString = new SecureString("DefaultEndpointsProtocol=https;AccountName=" + storageAccount + ";AccountKey=" + storageKey)
      };
      Console.WriteLine(SafeJsonConvert.SerializeObject(storageLinkedService, client.SerializationSettings));
      LinkedServiceResource linkedService = new LinkedServiceResource(storageLinkedService, name:storageLinkedServiceName);
      return linkedService;
   }
   ```

1. Agregue la siguiente línea al método `Main` que crea un servicio vinculado de Azure Storage:

   ```csharp
   client.LinkedServices.CreateOrUpdate(resourceGroup, dataFactoryName, storageLinkedServiceName, StorageLinkedServiceDefinition(client));
   ```

Para más información acerca de las propiedades admitidas y sus detalles, consulte [Propiedades del servicio vinculado](connector-azure-blob-storage.md#linked-service-properties).

## <a name="create-datasets"></a>Creación de conjuntos de datos

En esta sección se crean dos conjuntos de datos: uno para el origen y otro para el receptor.

### <a name="create-a-dataset-for-a-source-azure-blob"></a>Creación de un conjunto de datos para un blob de Azure de origen

Agregue un método que cree un *conjunto de datos de blob de Azure*. Para más información acerca de las propiedades admitidas y sus detalles, consulte [Propiedades del conjunto de datos de blob de Azure](connector-azure-blob-storage.md#dataset-properties).

Agregue un método `SourceBlobDatasetDefinition` al archivo *Program.cs*:

```csharp
static DatasetResource SourceBlobDatasetDefinition(DataFactoryManagementClient client)
{
    Console.WriteLine("Creating dataset " + blobSourceDatasetName + "...");
    AzureBlobDataset blobDataset = new AzureBlobDataset
    {
        FolderPath = new Expression { Value = "@pipeline().parameters.sourceBlobContainer" },
        FileName = inputBlobName,
        LinkedServiceName = new LinkedServiceReference
        {
            ReferenceName = storageLinkedServiceName
        }
    };
    Console.WriteLine(SafeJsonConvert.SerializeObject(blobDataset, client.SerializationSettings));
    DatasetResource dataset = new DatasetResource(blobDataset, name:blobSourceDatasetName);
    return dataset;
}
```

Se define un conjunto de datos que representa los datos de origen del blob de Azure. Este conjunto de datos de blob hace referencia al servicio vinculado de Azure Storage admitido del paso anterior. El conjunto de datos de blob describe la ubicación del blob desde el que se va a copiar: *FolderPath* y *FileName*.

Tenga en cuenta el uso de los parámetros en *FolderPath*. `sourceBlobContainer` es el nombre del parámetro; la expresión se reemplaza con los valores pasados en la ejecución de la canalización. La sintaxis para definir los parámetros es `@pipeline().parameters.<parameterName>`.

### <a name="create-a-dataset-for-a-sink-azure-blob"></a>Creación de un conjunto de datos para un blob de Azure receptor

1. Agregue un método `SourceBlobDatasetDefinition` al archivo *Program.cs*:

   ```csharp
   static DatasetResource SinkBlobDatasetDefinition(DataFactoryManagementClient client)
   {
       Console.WriteLine("Creating dataset " + blobSinkDatasetName + "...");
       AzureBlobDataset blobDataset = new AzureBlobDataset
       {
           FolderPath = new Expression { Value = "@pipeline().parameters.sinkBlobContainer" },
           LinkedServiceName = new LinkedServiceReference
           {
               ReferenceName = storageLinkedServiceName
           }
       };
       Console.WriteLine(SafeJsonConvert.SerializeObject(blobDataset, client.SerializationSettings));
       DatasetResource dataset = new DatasetResource(blobDataset, name: blobSinkDatasetName);
       return dataset;
   }
   ```

1. Agregue el siguiente código al método `Main` que crea los conjuntos de datos de blob de Azure de origen y recepción.

   ```csharp
   client.Datasets.CreateOrUpdate(resourceGroup, dataFactoryName, blobSourceDatasetName, SourceBlobDatasetDefinition(client));

   client.Datasets.CreateOrUpdate(resourceGroup, dataFactoryName, blobSinkDatasetName, SinkBlobDatasetDefinition(client));
   ```

## <a name="create-a-c-class-emailrequest"></a>Cree una clase de C#: EmailRequest

En el proyecto de C#, cree una clase denominada `EmailRequest`. Esta clase define qué propiedades envía la canalización en la solicitud del cuerpo al enviar un correo electrónico. En este tutorial, la canalización envía cuatro propiedades desde la canalización al correo electrónico:

* Message. Cuerpo del correo electrónico. Si la copia se realiza correctamente, esta propiedad contiene la cantidad de datos escritos. Si la copia no se realiza correctamente, esta propiedad contiene los detalles del error.
* Nombre de factoría de datos. El nombre de la factoría de datos.
* Nombre de canalización. Nombre de la canalización.
* Receptor. Parámetro que se pasa. Esta propiedad especifica el destinatario del correo electrónico.

```csharp
    class EmailRequest
    {
        [Newtonsoft.Json.JsonProperty(PropertyName = "message")]
        public string message;

        [Newtonsoft.Json.JsonProperty(PropertyName = "dataFactoryName")]
        public string dataFactoryName;

        [Newtonsoft.Json.JsonProperty(PropertyName = "pipelineName")]
        public string pipelineName;

        [Newtonsoft.Json.JsonProperty(PropertyName = "receiver")]
        public string receiver;

        public EmailRequest(string input, string df, string pipeline, string receiverName)
        {
            message = input;
            dataFactoryName = df;
            pipelineName = pipeline;
            receiver = receiverName;
        }
    }
```

## <a name="create-email-workflow-endpoints"></a>Creación de puntos de conexión de flujo de trabajo del correo electrónico

Para desencadenar el envío de un correo electrónico, use [Logic Apps](../logic-apps/logic-apps-overview.md) a fin de definir el flujo de trabajo. Para más información sobre la creación de un flujo de trabajo de Logic Apps, consulte [Creación de una aplicación lógica](../logic-apps/quickstart-create-first-logic-app-workflow.md).

### <a name="success-email-workflow"></a>Flujo de trabajo del correo electrónico de operación correcta

En [Azure Portal](https://portal.azure.com), cree un flujo de trabajo de Logic Apps denominado *CopySuccessEmail*. Defina el desencadenador del flujo de trabajo como `When an HTTP request is received`. Para el desencadenador de la solicitud, rellene `Request Body JSON Schema` con el JSON siguiente:

```json
{
    "properties": {
        "dataFactoryName": {
            "type": "string"
        },
        "message": {
            "type": "string"
        },
        "pipelineName": {
            "type": "string"
        },
        "receiver": {
            "type": "string"
        }
    },
    "type": "object"
}
```

El flujo de trabajo tiene un aspecto similar al del ejemplo siguiente:

![Flujo de trabajo del correo electrónico de operación correcta](media/tutorial-control-flow/success-email-workflow-trigger.png)

Este contenido JSON se corresponde con la clase `EmailRequest` creada en la sección anterior.

Agregue una acción `Office 365 Outlook – Send an email`. Para la acción **Enviar un correo electrónico**, personalice el formato del correo electrónico. Para ello, use las propiedades que se pasan en el esquema JSON del **Cuerpo** de solicitud. Este es un ejemplo:

![Diseñador de aplicación lógica: acción de envío de correo electrónico](media/tutorial-control-flow/customize-send-email-action.png)

Después de guardar el flujo de trabajo, copie y guarde el valor de **Dirección URL de HTTP POST** del desencadenador.

## <a name="fail-email-workflow"></a>Flujo de trabajo del correo electrónico de operación incorrecta

Clone **CopySuccessEmail** como otro flujo de trabajo de Logic Apps denominado *CopyFailEmail*. En el desencadenador de solicitudes, `Request Body JSON schema` es el mismo. Cambie el formato del correo electrónico, por ejemplo, la parte `Subject`, para adaptarlo para que sea un correo electrónico de operación incorrecta. Este es un ejemplo:

![Diseñador de aplicación lógica: flujo de trabajo del correo electrónico de operación incorrecta](media/tutorial-control-flow/fail-email-workflow.png)

Después de guardar el flujo de trabajo, copie y guarde el valor de **Dirección URL de HTTP POST** del desencadenador.

Ahora debería tener dos direcciones URL de flujo de trabajo, como en los ejemplos siguientes:

```csharp
//Success Request Url
https://prodxxx.eastus.logic.azure.com:443/workflows/000000/triggers/manual/paths/invoke?api-version=2016-10-01&sp=%2Ftriggers%2Fmanual%2Frun&sv=1.0&sig=000000

//Fail Request Url
https://prodxxx.eastus.logic.azure.com:443/workflows/000000/triggers/manual/paths/invoke?api-version=2016-10-01&sp=%2Ftriggers%2Fmanual%2Frun&sv=1.0&sig=000000
```

## <a name="create-a-pipeline"></a>Crear una canalización

Vuelva al proyecto en Visual Studio. Ahora vamos a agregar el código que crea una canalización con una actividad de copia y la propiedad `DependsOn`. En este tutorial, la canalización contiene una actividad, la actividad de copia, que toma el conjunto de datos del blob como origen y otro conjunto de datos del blob como receptor. Si la actividad de copia se realiza de forma correcta o incorrecta, llama a distintas tareas de correo electrónico.

En esta canalización, use las siguientes características:

* Parámetros
* Actividad web
* Dependencia de actividades
* Uso de la salida de una actividad como entrada de otra actividad

1. Agregue este método al proyecto. En las siguientes secciones se ofrecen más detalles.

    ```csharp
    static PipelineResource PipelineDefinition(DataFactoryManagementClient client)
            {
                Console.WriteLine("Creating pipeline " + pipelineName + "...");
                PipelineResource resource = new PipelineResource
                {
                    Parameters = new Dictionary<string, ParameterSpecification>
                    {
                        { "sourceBlobContainer", new ParameterSpecification { Type = ParameterType.String } },
                        { "sinkBlobContainer", new ParameterSpecification { Type = ParameterType.String } },
                        { "receiver", new ParameterSpecification { Type = ParameterType.String } }

                    },
                    Activities = new List<Activity>
                    {
                        new CopyActivity
                        {
                            Name = copyBlobActivity,
                            Inputs = new List<DatasetReference>
                            {
                                new DatasetReference
                                {
                                    ReferenceName = blobSourceDatasetName
                                }
                            },
                            Outputs = new List<DatasetReference>
                            {
                                new DatasetReference
                                {
                                    ReferenceName = blobSinkDatasetName
                                }
                            },
                            Source = new BlobSource { },
                            Sink = new BlobSink { }
                        },
                        new WebActivity
                        {
                            Name = sendSuccessEmailActivity,
                            Method = WebActivityMethod.POST,
                            Url = "https://prodxxx.eastus.logic.azure.com:443/workflows/00000000000000000000000000000000000/triggers/manual/paths/invoke?api-version=2016-10-01&sp=%2Ftriggers%2Fmanual%2Frun&sv=1.0&sig=0000000000000000000000000000000000000000000000",
                            Body = new EmailRequest("@{activity('CopyBlobtoBlob').output.dataWritten}", "@{pipeline().DataFactory}", "@{pipeline().Pipeline}", "@pipeline().parameters.receiver"),
                            DependsOn = new List<ActivityDependency>
                            {
                                new ActivityDependency
                                {
                                    Activity = copyBlobActivity,
                                    DependencyConditions = new List<String> { "Succeeded" }
                                }
                            }
                        },
                        new WebActivity
                        {
                            Name = sendFailEmailActivity,
                            Method =WebActivityMethod.POST,
                            Url = "https://prodxxx.eastus.logic.azure.com:443/workflows/000000000000000000000000000000000/triggers/manual/paths/invoke?api-version=2016-10-01&sp=%2Ftriggers%2Fmanual%2Frun&sv=1.0&sig=0000000000000000000000000000000000000000000",
                            Body = new EmailRequest("@{activity('CopyBlobtoBlob').error.message}", "@{pipeline().DataFactory}", "@{pipeline().Pipeline}", "@pipeline().parameters.receiver"),
                            DependsOn = new List<ActivityDependency>
                            {
                                new ActivityDependency
                                {
                                    Activity = copyBlobActivity,
                                    DependencyConditions = new List<String> { "Failed" }
                                }
                            }
                        }
                    }
                };
                Console.WriteLine(SafeJsonConvert.SerializeObject(resource, client.SerializationSettings));
                return resource;
            }
    ```

1. Agregue la siguiente línea al método `Main` que crea la canalización:

   ```csharp
   client.Pipelines.CreateOrUpdate(resourceGroup, dataFactoryName, pipelineName, PipelineDefinition(client));
   ```

### <a name="parameters"></a>Parámetros

La primera sección del código de la canalización define parámetros.

* `sourceBlobContainer`. El conjunto de datos de blob de origen consume este parámetro en la canalización.
* `sinkBlobContainer`. El conjunto de datos de blob de receptor consume este parámetro en la canalización.
* `receiver`. Las dos actividades web de la canalización que envían correos electrónicos de fin correcto o error al receptor usan este parámetro.

```csharp
Parameters = new Dictionary<string, ParameterSpecification>
    {
        { "sourceBlobContainer", new ParameterSpecification { Type = ParameterType.String } },
        { "sinkBlobContainer", new ParameterSpecification { Type = ParameterType.String } },
        { "receiver", new ParameterSpecification { Type = ParameterType.String } }
    },
```

### <a name="web-activity"></a>Actividad web

Una actividad web permite efectuar una llamada a cualquier punto de conexión REST. Para más información sobre la actividad, consulte [Actividad web en Azure Data Factory](control-flow-web-activity.md). Esta canalización usa actividades web para llamar al flujo de trabajo de correo electrónico de Logic Apps. Se crean dos actividades web: una que llama al flujo de trabajo `CopySuccessEmail` y otra que llama a `CopyFailWorkFlow`.

```csharp
        new WebActivity
        {
            Name = sendCopyEmailActivity,
            Method = WebActivityMethod.POST,
            Url = "https://prodxxx.eastus.logic.azure.com:443/workflows/12345",
            Body = new EmailRequest("@{activity('CopyBlobtoBlob').output.dataWritten}", "@{pipeline().DataFactory}", "@{pipeline().Pipeline}", "@pipeline().parameters.receiver"),
            DependsOn = new List<ActivityDependency>
            {
                new ActivityDependency
                {
                    Activity = copyBlobActivity,
                    DependencyConditions = new List<String> { "Succeeded" }
                }
            }
        }
```

En la propiedad `Url`, pegue los puntos de conexión de **Dirección URL HTTP POST** de los flujos de trabajo de Logic Apps. En la propiedad `Body`, pase una instancia de la clase `EmailRequest`. La solicitud de correo electrónico contiene las siguientes propiedades:

* Message. Pasa el valor de `@{activity('CopyBlobtoBlob').output.dataWritten`. Accede a una propiedad de la actividad de copia anterior y pasa el valor de `dataWritten`. En caso de error, pasa la salida de error en lugar de `@{activity('CopyBlobtoBlob').error.message`.
* Nombre de factoría de datos. Pasa el valor de `@{pipeline().DataFactory}`. Esta variable del sistema permite acceder al nombre de la factoría de datos correspondiente. Para obtener una lista de las variables del sistema, consulte [Variables del sistema](control-flow-system-variables.md).
* Nombre de canalización. Pasa el valor de `@{pipeline().Pipeline}`. Esta variable del sistema permite acceder al nombre de la canalización correspondiente.
* Receptor. Pasa el valor de `"@pipeline().parameters.receiver"`. Accede a los parámetros de la canalización.

Este código crea una nueva dependencia de actividad que depende de la actividad de copia anterior.

## <a name="create-a-pipeline-run"></a>Creación de una ejecución de canalización

Agregue el siguiente código al método `Main` que desencadena una ejecución de canalización.

```csharp
// Create a pipeline run
Console.WriteLine("Creating pipeline run...");
Dictionary<string, object> arguments = new Dictionary<string, object>
{
    { "sourceBlobContainer", inputBlobPath },
    { "sinkBlobContainer", outputBlobPath },
    { "receiver", emailReceiver }
};

CreateRunResponse runResponse = client.Pipelines.CreateRunWithHttpMessagesAsync(resourceGroup, dataFactoryName, pipelineName, arguments).Result.Body;
Console.WriteLine("Pipeline run ID: " + runResponse.RunId);
```

## <a name="main-class"></a>Clase Main

El método `Main` final debe tener el aspecto siguiente.

```csharp
// Authenticate and create a data factory management client
var context = new AuthenticationContext("https://login.windows.net/" + tenantID);
ClientCredential cc = new ClientCredential(applicationId, authenticationKey);
AuthenticationResult result = context.AcquireTokenAsync("https://management.azure.com/", cc).Result;
ServiceClientCredentials cred = new TokenCredentials(result.AccessToken);
var client = new DataFactoryManagementClient(cred) { SubscriptionId = subscriptionId };

Factory df = CreateOrUpdateDataFactory(client);

client.LinkedServices.CreateOrUpdate(resourceGroup, dataFactoryName, storageLinkedServiceName, StorageLinkedServiceDefinition(client));
client.Datasets.CreateOrUpdate(resourceGroup, dataFactoryName, blobSourceDatasetName, SourceBlobDatasetDefinition(client));
client.Datasets.CreateOrUpdate(resourceGroup, dataFactoryName, blobSinkDatasetName, SinkBlobDatasetDefinition(client));

client.Pipelines.CreateOrUpdate(resourceGroup, dataFactoryName, pipelineName, PipelineDefinition(client));

Console.WriteLine("Creating pipeline run...");
Dictionary<string, object> arguments = new Dictionary<string, object>
{
    { "sourceBlobContainer", inputBlobPath },
    { "sinkBlobContainer", outputBlobPath },
    { "receiver", emailReceiver }
};

CreateRunResponse runResponse = client.Pipelines.CreateRunWithHttpMessagesAsync(resourceGroup, dataFactoryName, pipelineName, arguments).Result.Body;
Console.WriteLine("Pipeline run ID: " + runResponse.RunId);
```

Compile y ejecute su programa para desencadenar una ejecución de canalización.

## <a name="monitor-a-pipeline-run"></a>Supervisar una ejecución de canalización

1. Agregue el siguiente código al método `Main`:

    ```csharp
    // Monitor the pipeline run
    Console.WriteLine("Checking pipeline run status...");
    PipelineRun pipelineRun;
    while (true)
    {
        pipelineRun = client.PipelineRuns.Get(resourceGroup, dataFactoryName, runResponse.RunId);
        Console.WriteLine("Status: " + pipelineRun.Status);
        if (pipelineRun.Status == "InProgress")
            System.Threading.Thread.Sleep(15000);
        else
            break;
    }
    ```

    Este código comprueba continuamente el estado de la ejecución hasta que termina de copiar los datos.

1. Agregue el siguiente código al método `Main` que recupera detalles de la ejecución de la actividad de copia, como el tamaño de los datos leídos o escritos:

    ```csharp
    // Check the copy activity run details
    Console.WriteLine("Checking copy activity run details...");

    List<ActivityRun> activityRuns = client.ActivityRuns.ListByPipelineRun(
    resourceGroup, dataFactoryName, runResponse.RunId, DateTime.UtcNow.AddMinutes(-10), DateTime.UtcNow.AddMinutes(10)).ToList();

    if (pipelineRun.Status == "Succeeded")
    {
        Console.WriteLine(activityRuns.First().Output);
        //SaveToJson(SafeJsonConvert.SerializeObject(activityRuns.First().Output, client.SerializationSettings), "ActivityRunResult.json", folderForJsons);
    }
    else
        Console.WriteLine(activityRuns.First().Error);

    Console.WriteLine("\nPress any key to exit...");
    Console.ReadKey();
    ```

## <a name="run-the-code"></a>Ejecución del código

Compile e inicie la aplicación y, a continuación, compruebe la ejecución de la canalización.

La aplicación muestra el progreso de la creación de la factoría de datos, el servicio vinculado, los conjuntos de datos, la canalización y la ejecución de la canalización. A continuación, comprueba el estado de la ejecución de canalización. Espere hasta que vea los detalles de ejecución de actividad de copia con el tamaño de los datos leídos/escritos. A continuación, use herramientas como el Explorador de Azure Storage para comprobar que los blobs se copian a *outputBlobPath* desde *inputBlobPath*, como se especificó en las variables.

La salida debe ser similar al siguiente ejemplo:

```json
Creating data factory DFTutorialTest...
{
  "location": "East US"
}
Creating linked service AzureStorageLinkedService...
{
  "type": "AzureStorage",
  "typeProperties": {
    "connectionString": "DefaultEndpointsProtocol=https;AccountName=***;AccountKey=***"
  }
}
Creating dataset SourceStorageDataset...
{
  "type": "AzureBlob",
  "typeProperties": {
    "folderPath": {
      "type": "Expression",
      "value": "@pipeline().parameters.sourceBlobContainer"
    },
    "fileName": "input.txt"
  },
  "linkedServiceName": {
    "type": "LinkedServiceReference",
    "referenceName": "AzureStorageLinkedService"
  }
}
Creating dataset SinkStorageDataset...
{
  "type": "AzureBlob",
  "typeProperties": {
    "folderPath": {
      "type": "Expression",
      "value": "@pipeline().parameters.sinkBlobContainer"
    }
  },
  "linkedServiceName": {
    "type": "LinkedServiceReference",
    "referenceName": "AzureStorageLinkedService"
  }
}
Creating pipeline Adfv2TutorialBranchCopy...
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
            "type": "DatasetReference",
            "referenceName": "SourceStorageDataset"
          }
        ],
        "outputs": [
          {
            "type": "DatasetReference",
            "referenceName": "SinkStorageDataset"
          }
        ],
        "name": "CopyBlobtoBlob"
      },
      {
        "type": "WebActivity",
        "typeProperties": {
          "method": "POST",
          "url": "https://xxxx.eastus.logic.azure.com:443/workflows/... ",
          "body": {
            "message": "@{activity('CopyBlobtoBlob').output.dataWritten}",
            "dataFactoryName": "@{pipeline().DataFactory}",
            "pipelineName": "@{pipeline().Pipeline}",
            "receiver": "@pipeline().parameters.receiver"
          }
        },
        "name": "SendSuccessEmailActivity",
        "dependsOn": [
          {
            "activity": "CopyBlobtoBlob",
            "dependencyConditions": [
              "Succeeded"
            ]
          }
        ]
      },
      {
        "type": "WebActivity",
        "typeProperties": {
          "method": "POST",
          "url": "https://xxx.eastus.logic.azure.com:443/workflows/... ",
          "body": {
            "message": "@{activity('CopyBlobtoBlob').error.message}",
            "dataFactoryName": "@{pipeline().DataFactory}",
            "pipelineName": "@{pipeline().Pipeline}",
            "receiver": "@pipeline().parameters.receiver"
          }
        },
        "name": "SendFailEmailActivity",
        "dependsOn": [
          {
            "activity": "CopyBlobtoBlob",
            "dependencyConditions": [
              "Failed"
            ]
          }
        ]
      }
    ],
    "parameters": {
      "sourceBlobContainer": {
        "type": "String"
      },
      "sinkBlobContainer": {
        "type": "String"
      },
      "receiver": {
        "type": "String"
      }
    }
  }
}
Creating pipeline run...
Pipeline run ID: 00000000-0000-0000-0000-0000000000000
Checking pipeline run status...
Status: InProgress
Status: InProgress
Status: Succeeded
Checking copy activity run details...
{
  "dataRead": 20,
  "dataWritten": 20,
  "copyDuration": 4,
  "throughput": 0.01,
  "errors": [],
  "effectiveIntegrationRuntime": "DefaultIntegrationRuntime (East US)"
}
{}

Press any key to exit...
```

## <a name="next-steps"></a>Pasos siguientes

En este tutorial ha realizado las tareas siguientes:

> [!div class="checklist"]
> * Crear una factoría de datos
> * Creación de un servicio vinculado de Azure Storage
> * Creación de un conjunto de datos del blob de Azure
> * Creación de una canalización que contiene una actividad de copia y una actividad web
> * Envío de los resultados de las actividades en actividades subsiguientes
> * Uso del paso de parámetros y de las variables del sistema
> * Inicio de la ejecución de una canalización
> * Supervisión de las ejecuciones de canalización y actividad

Ahora puede continuar en la sección de conceptos para más información sobre Azure Data Factory.
> [!div class="nextstepaction"]
>[Canalizaciones y actividades](concepts-pipelines-activities.md)
