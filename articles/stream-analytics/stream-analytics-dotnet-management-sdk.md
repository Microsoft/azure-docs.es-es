---
title: SDK de .NET de administración para Azure Stream Analytics
description: Introducción al uso del SDK de .NET de administración de Stream Analytics Aprenda a configurar y ejecutar trabajos de análisis. Cree un proyecto, entradas, salidas y transformaciones.
author: jseb225
ms.author: jeanb
ms.reviewer: mamccrea
ms.service: stream-analytics
ms.topic: how-to
ms.date: 12/06/2018
ms.custom: seodec18, devx-track-csharp
ms.openlocfilehash: 5776c741bb69d345f191e5673fe388328d0fe7bb
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89015601"
---
# <a name="management-net-sdk-set-up-and-run-analytics-jobs-using-the-azure-stream-analytics-api-for-net"></a>SDK .NET de administración: Configuración y ejecución de trabajos de análisis con la API de Azure Stream Analytics para .NET
Aprenda a configurar y ejecutar trabajos de análisis con la API de Stream Analytics para .NET mediante el SDK de .NET de administración. Configure un proyecto, cree orígenes de entrada y salida, transformaciones, e inicie y detenga trabajos. En los trabajos de análisis puede transmitir datos desde el almacenamiento de blobs o desde un centro de eventos.

Consulte la [documentación de referencia de administración de la API de Stream Analytics para .NET](https://msdn.microsoft.com/library/azure/dn889315.aspx).

Azure Stream Analytics es un servicio totalmente administrado que proporciona un procesamiento completo de eventos de baja latencia, alta disponibilidad y escalable a través de la transmisión de datos en la nube. Stream Analytics permite a los clientes configurar trabajos de streaming para analizar flujos de datos y realizar análisis casi en tiempo real.  

> [!NOTE]
> El código de ejemplo de este artículo se ha actualizado con la versión v2.x del SDK de .NET de administración de Azure Stream Analytics. Para código de ejemplo con la versión heredada (1.x) del SDK, vea [Uso del SDK v1.x de .NET de administración para Stream Analytics](https://docs.microsoft.com/azure/stream-analytics/stream-analytics-dotnet-management-sdk-v1).

## <a name="prerequisites"></a>Requisitos previos
Antes de comenzar con este artículo, debe realizar los siguientes pasos:

* Instale Visual Studio 2019 o 2015.
* Descargue e instale el [SDK de .NET de Azure](https://azure.microsoft.com/downloads/).
* Cree un grupo de recursos de Azure en su suscripción. A continuación se muestra un ejemplo de script de Azure PowerShell. Para obtener información sobre Azure PowerShell, consulte [Instalación y configuración de Azure PowerShell](/powershell/azure/).  

   ```powershell
   # Log in to your Azure account
   Add-AzureAccount
   
   # Select the Azure subscription you want to use to create the resource group
   Select-AzureSubscription -SubscriptionName <subscription name>
   
   # If Stream Analytics has not been registered to the subscription, remove the remark    symbol (#) to run the Register-AzProvider cmdlet to register the provider namespace
   #Register-AzProvider -Force -ProviderNamespace 'Microsoft.StreamAnalytics'
   
   # Create an Azure resource group
   New-AzureResourceGroup -Name <YOUR RESOURCE GROUP NAME> -Location <LOCATION>
   ```

* Configure un origen de entrada y un destino de salida a los que se conectará el trabajo.

## <a name="set-up-a-project"></a>Configuración de un proyecto
Para crear un trabajo de análisis que use la API de Stream Analytics para. NET, configure primero el proyecto.

1. Cree una aplicación de consola .NET de Visual Studio C#.
2. En la consola del administrador de paquetes, ejecute los siguientes comandos para instalar los paquetes NuGet. El primero es el SDK de .NET de administración de Azure Stream Analytics. El segundo corresponde a la autenticación de cliente de Azure.

   ```powershell   
   Install-Package Microsoft.Azure.Management.StreamAnalytics -Version 2.0.0
   Install-Package Microsoft.Rest.ClientRuntime.Azure.Authentication -Version 2.3.1
   ```

3. Agregue la siguiente sección **appSettings** al archivo App.config:
   
   ```powershell
   <appSettings>
       <add key="ClientId" value="1950a258-227b-4e31-a9cf-717495945fc2" />
       <add key="RedirectUri" value="urn:ietf:wg:oauth:2.0:oob" />
       <add key="SubscriptionId" value="YOUR SUBSCRIPTION ID" />
       <add key="ActiveDirectoryTenantId" value="YOUR TENANT ID" />
   </appSettings>
   ```

    Reemplace los valores para **SubscriptionId** y **ActiveDirectoryTenantId** por sus identificadores de inquilino y de suscripción de Azure. Para obtener estos valores, ejecute el siguiente cmdlet de Azure PowerShell:

   ```powershell
      Get-AzureAccount
   ```

4. Agregue la siguiente referencia al archivo .csproj:

   ```csharp
   <Reference Include="System.Configuration" />
   ```

5. Agregue las siguientes instrucciones **using** al archivo de origen (Program.cs) en el proyecto:
   
   ```csharp
   using System;
   using System.Collections.Generic;
   using System.Configuration;
   using System.Threading;
   using System.Threading.Tasks;
   
   using Microsoft.Azure.Management.StreamAnalytics;
   using Microsoft.Azure.Management.StreamAnalytics.Models;
   using Microsoft.Rest.Azure.Authentication;
   using Microsoft.Rest;
   ```

6. Agregue un método auxiliar de autenticación:

   ```csharp
   private static async Task<ServiceClientCredentials> GetCredentials()
   {
       var activeDirectoryClientSettings = ActiveDirectoryClientSettings.UsePromptOnly(ConfigurationManager.AppSettings["ClientId"], new Uri("urn:ietf:wg:oauth:2.0:oob"));
       ServiceClientCredentials credentials = await UserTokenProvider.LoginWithPromptAsync(ConfigurationManager.AppSettings["ActiveDirectoryTenantId"], activeDirectoryClientSettings);
       
       return credentials;
    }
   ```

## <a name="create-a-stream-analytics-management-client"></a>Cree un cliente de administración de Stream Analytics
Un objeto **StreamAnalyticsManagementClient** le permite administrar el trabajo y los componentes del trabajo, como la entrada, la salida y la transformación.

Agregue el siguiente código al comienzo del método **Main** :

   ```csharp
    string resourceGroupName = "<YOUR AZURE RESOURCE GROUP NAME>";
    string streamingJobName = "<YOUR STREAMING JOB NAME>";
    string inputName = "<YOUR JOB INPUT NAME>";
    string transformationName = "<YOUR JOB TRANSFORMATION NAME>";
    string outputName = "<YOUR JOB OUTPUT NAME>";
    
    SynchronizationContext.SetSynchronizationContext(new SynchronizationContext());
    
    // Get credentials
    ServiceClientCredentials credentials = GetCredentials().Result;
    
    // Create Stream Analytics management client
    StreamAnalyticsManagementClient streamAnalyticsManagementClient = new StreamAnalyticsManagementClient(credentials)
    {
        SubscriptionId = ConfigurationManager.AppSettings["SubscriptionId"]
    };
   ```

El valor de la variable **resourceGroupName** debe ser el mismo que el nombre del grupo de recursos que creó o eligió en los pasos de requisitos previos.

Para automatizar el aspecto de la presentación de credenciales de creación del trabajo, consulte [Autenticación de una entidad de servicio con el Administrador de recursos de Azure](../active-directory/develop/howto-authenticate-service-principal-powershell.md).

Las secciones restantes de este artículo suponen que este código se encuentra al comienzo del método **Main** .

## <a name="create-a-stream-analytics-job"></a>Creación de un trabajo de Stream Analytics
El siguiente código crea un trabajo de Stream Analytics bajo el grupo de recursos que ha definido. Agregará una entrada, salida y transformación al trabajo más adelante.

   ```csharp
   // Create a streaming job
   StreamingJob streamingJob = new StreamingJob()
   {
       Tags = new Dictionary<string, string>()
       {
           { "Origin", ".NET SDK" },
           { "ReasonCreated", "Getting started tutorial" }
       },
       Location = "West US",
       EventsOutOfOrderPolicy = EventsOutOfOrderPolicy.Drop,
       EventsOutOfOrderMaxDelayInSeconds = 5,
       EventsLateArrivalMaxDelayInSeconds = 16,
       OutputErrorPolicy = OutputErrorPolicy.Drop,
       DataLocale = "en-US",
       CompatibilityLevel = CompatibilityLevel.OneFullStopZero,
       Sku = new Sku()
       {
           Name = SkuName.Standard
       }
   };
   StreamingJob createStreamingJobResult = streamAnalyticsManagementClient.StreamingJobs.CreateOrReplace(streamingJob, resourceGroupName, streamingJobName);
   ```

## <a name="create-a-stream-analytics-input-source"></a>Creación de un origen de entrada de Stream Analytics
El código siguiente crea un origen de entrada de Stream Analytics con el tipo de origen de entrada de blob y la serialización de CSV. Para crear un origen de entrada de centro de eventos, use **EventHubStreamInputDataSource** en lugar de **BlobStreamInputDataSource**. De manera similar, puede personalizar el tipo de serialización del origen de entrada.

   ```csharp
   // Create an input
   StorageAccount storageAccount = new StorageAccount()
   {
       AccountName = "<YOUR STORAGE ACCOUNT NAME>",
       AccountKey = "<YOUR STORAGE ACCOUNT KEY>"
   };
   Input input = new Input()
   {
       Properties = new StreamInputProperties()
       {
           Serialization = new CsvSerialization()
           {
               FieldDelimiter = ",",
               Encoding = Encoding.UTF8
           },
           Datasource = new BlobStreamInputDataSource()
           {
               StorageAccounts = new[] { storageAccount },
               Container = "<YOUR STORAGE BLOB CONTAINER>",
               PathPattern = "{date}/{time}",
               DateFormat = "yyyy/MM/dd",
               TimeFormat = "HH",
               SourcePartitionCount = 16
           }
       }
   };
   Input createInputResult = streamAnalyticsManagementClient.Inputs.CreateOrReplace(input, resourceGroupName, streamingJobName, inputName);
   ```

Los orígenes de entrada, ya sean desde el almacenamiento de blobs o un centro de eventos, están vinculados a un trabajo específico. Para usar el mismo origen de entrada para distintos trabajos, debe llamar nuevamente al método y especificar un nombre de trabajo distinto.

## <a name="test-a-stream-analytics-input-source"></a>Prueba del origen de entrada de Stream Analytics
El método **TestConnection** prueba si el trabajo de Stream Analytics puede conectarse al origen de entrada así como otros aspectos específicos para el tipo de origen de entrada. Por ejemplo, en el origen de entrada de blob que creó en un paso anterior, el método comprobará que el par de claves y el nombre de cuenta de almacenamiento se pueden usar para conectarse a la cuenta de almacenamiento, así como para comprobar que existe el contenedor especificado.

   ```csharp
   // Test the connection to the input
   ResourceTestStatus testInputResult = streamAnalyticsManagementClient.Inputs.Test(resourceGroupName, streamingJobName, inputName);
   ```

## <a name="create-a-stream-analytics-output-target"></a>Creación de un destino de salida de Stream Analytics
La creación de un destino de salida es similar al proceso de creación de un origen de entrada de Stream Analytics. Al igual que los orígenes de entrada, los destinos de salida están vinculados a un trabajo específico. Para usar el mismo destino de salida para distintos trabajos, debe llamar nuevamente al método y especificar un nombre de trabajo distinto.

El código siguiente crea un destino de salida (Azure SQL Database). Puede personalizar el tipo de datos y/o el tipo de serialización del destino de salida.

   ```csharp
   // Create an output
   Output output = new Output()
   {
       Datasource = new AzureSqlDatabaseOutputDataSource()
       {
           Server = "<YOUR DATABASE SERVER NAME>",
           Database = "<YOUR DATABASE NAME>",
           User = "<YOUR DATABASE LOGIN>",
           Password = "<YOUR DATABASE LOGIN PASSWORD>",
           Table = "<YOUR DATABASE TABLE NAME>"
       }
   };
   Output createOutputResult = streamAnalyticsManagementClient.Outputs.CreateOrReplace(output, resourceGroupName, streamingJobName, outputName);
   ```

## <a name="test-a-stream-analytics-output-target"></a>Prueba de un destino de salida de Stream Analytics
Un destino de salida de Stream Analytics también tiene el método **TestConnection** para probar conexiones.

   ```csharp
   // Test the connection to the output
   ResourceTestStatus testOutputResult = streamAnalyticsManagementClient.Outputs.Test(resourceGroupName, streamingJobName, outputName);
   ```

## <a name="create-a-stream-analytics-transformation"></a>Creación de una transformación de Stream Analytics
El siguiente código crea una transformación de Stream Analytics con la consulta "select * from Input" y especifica para asignar una unidad de streaming para el trabajo de Stream Analytics. Para obtener más información sobre el ajuste de las unidades de streaming, consulte [Escalación de trabajos de Stream Analytics](stream-analytics-scale-jobs.md).

   ```csharp
   // Create a transformation
   Transformation transformation = new Transformation()
   {
       Query = "Select Id, Name from <your input name>", // '<your input name>' should be replaced with the value you put for the 'inputName' variable above or in a previous step
       StreamingUnits = 1
   };
   Transformation createTransformationResult = streamAnalyticsManagementClient.Transformations.CreateOrReplace(transformation, resourceGroupName, streamingJobName, transformationName);
   ```

Al igual que la entrada y la salida, una transformación también está vinculada al trabajo de Stream Analytics específico en el que se creó.

## <a name="start-a-stream-analytics-job"></a>Inicio de un trabajo de Stream Analytics
Después de crear un trabajo de Stream Analytics y sus entradas, salidas y transformaciones, puede iniciar el trabajo si llama al método **Start** .

El siguiente código de ejemplo inicia un trabajo de Stream Analytics con una hora de inicio de salida personalizada definida para el 12 de diciembre de 2012, 12:12:12 UTC:

   ```csharp
   // Start a streaming job
   StartStreamingJobParameters startStreamingJobParameters = new StartStreamingJobParameters()
   {
       OutputStartMode = OutputStartMode.CustomTime,
       OutputStartTime = new DateTime(2012, 12, 12, 12, 12, 12, DateTimeKind.Utc)
   };
   streamAnalyticsManagementClient.StreamingJobs.Start(resourceGroupName, streamingJobName, startStreamingJobParameters);
   ```

## <a name="stop-a-stream-analytics-job"></a>Detención de un trabajo de Stream Analytics
Puede detener un trabajo de Stream Analytics en ejecución si llama al método **Stop** .

   ```csharp
   // Stop a streaming job
   streamAnalyticsManagementClient.StreamingJobs.Stop(resourceGroupName, streamingJobName);
   ```

## <a name="delete-a-stream-analytics-job"></a>Eliminación de un trabajo de Stream Analytics
El método **Delete** eliminará el trabajo, además de los subrecursos subyacentes, incluidas las entradas, salidas y transformaciones del trabajo.

   ```csharp
   // Delete a streaming job
   streamAnalyticsManagementClient.StreamingJobs.Delete(resourceGroupName, streamingJobName);
   ```

## <a name="get-support"></a>Obtención de soporte técnico
Para más ayuda, pruebe nuestra [página de preguntas y respuestas de Microsoft sobre Azure Stream Analytics](https://docs.microsoft.com/answers/topics/azure-stream-analytics.html).

## <a name="next-steps"></a>Pasos siguientes
Ha aprendido los conceptos básicos del uso de un SDK de .NET para crear y ejecutar trabajos de análisis. Para más información, vea los siguientes artículos:

* [Introducción a Azure Stream Analytics](stream-analytics-introduction.md)
* [Introducción al uso de Azure Stream Analytics](stream-analytics-real-time-fraud-detection.md)
* [Escalado de trabajos de Azure Stream Analytics](stream-analytics-scale-jobs.md)
* [SDK de .NET de administración de Azure Stream Analytics](https://msdn.microsoft.com/library/azure/dn889315.aspx)
* [Referencia del lenguaje de consulta de Azure Stream Analytics](https://docs.microsoft.com/stream-analytics-query/stream-analytics-query-language-reference)
* [Referencia de API de REST de administración de Azure Stream Analytics](https://msdn.microsoft.com/library/azure/dn835031.aspx)

<!--Image references-->
[5]: ./media/markdown-template-for-new-articles/octocats.png
[6]: ./media/markdown-template-for-new-articles/pretty49.png
[7]: ./media/markdown-template-for-new-articles/channel-9.png


<!--Link references-->
[azure.blob.storage]: https://azure.microsoft.com/documentation/services/storage/
[azure.blob.storage.use]: https://azure.microsoft.com/documentation/articles/storage-dotnet-how-to-use-blobs/

[azure.event.hubs]: https://azure.microsoft.com/services/event-hubs/
[azure.event.hubs.developer.guide]: https://msdn.microsoft.com/library/azure/dn789972.aspx

[stream.analytics.query.language.reference]: https://go.microsoft.com/fwlink/?LinkID=513299
[stream.analytics.forum]: https://go.microsoft.com/fwlink/?LinkId=512151

[stream.analytics.introduction]: stream-analytics-introduction.md
[stream.analytics.get.started]: stream-analytics-real-time-fraud-detection.md
[stream.analytics.developer.guide]: stream-analytics-developer-guide.md
[stream.analytics.scale.jobs]: stream-analytics-scale-jobs.md
[stream.analytics.query.language.reference]: https://go.microsoft.com/fwlink/?LinkID=513299
[stream.analytics.rest.api.reference]: https://go.microsoft.com/fwlink/?LinkId=517301
