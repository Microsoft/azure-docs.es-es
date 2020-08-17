---
title: Uso de Azure WebHooks para supervisar las notificaciones de trabajo de Media Services con .NET | Microsoft Docs
description: Obtenga información sobre cómo usar Azure WebHooks para supervisar las notificaciones de trabajo de Media Services. El ejemplo de código está escrito en C# y utiliza el SDK de Media Services para .NET.
services: media-services
documentationcenter: ''
author: juliako
manager: femila
editor: ''
ms.assetid: a61fe157-81b1-45c1-89f2-224b7ef55869
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: dotnet
ms.topic: article
ms.date: 03/18/2019
ms.author: juliako
ms.openlocfilehash: 0a82050bef36e139c122ea97f777fb5cb1906974
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87052957"
---
# <a name="use-azure-webhooks-to-monitor-media-services-job-notifications-with-net"></a>Uso de Azure WebHooks para supervisar las notificaciones de trabajo de Media Services con .NET 

> [!NOTE]
> No hay características o funcionalidades nuevas para agregar a Media Services, versión 2. <br/>Finalice la compra de la versión más reciente, [Media Services v3](../latest/index.yml). Consulte también la [guía de migración de la versión v2 a v3](../latest/migrate-from-v2-to-v3.md).

Al ejecutar trabajos, muchas veces se requiere una forma de hacer un seguimiento al progreso del trabajo. Puede supervisar las notificaciones de trabajo de Media Services con Azure WebHooks o con [Azure Queue Storage](media-services-dotnet-check-job-progress-with-queues.md). En este artículo se explica cómo trabajar con webhooks.

Este artículo le muestra cómo

*  Definir una función de Azure que está personalizada para responder a webhook. 
    
    En este caso, Media Services desencadena el webhook cuando el trabajo de codificación cambia de estado. La función escucha la devolución de llamada al webhook desde las notificaciones de Media Services y publica el recurso de salida cuando el trabajo ha finalizado. 
    
    >[!TIP]
    >Antes de continuar, asegúrese de que comprende cómo funcionan los [enlaces HTTP y webhook en Azure Functions](../../azure-functions/functions-bindings-http-webhook.md).
    >
    
* Agregue un webhook a la tarea de codificación y especifique la dirección URL del webhook y la clave secreta a la que este webhook responde. Al final del artículo encontrará un ejemplo que agrega un webhook a la tarea de codificación.  

Puede encontrar las definiciones de diversas instancias de Azure Functions con .NET para Media Services (incluidas las que se muestran en este artículo) [aquí](https://github.com/Azure-Samples/media-services-dotnet-functions-integration).

## <a name="prerequisites"></a>Prerrequisitos

Estos son los requisitos previos para completar el tutorial.

* Una cuenta de Azure. Para obtener más información, consulte [Evaluación gratuita de Azure](https://azure.microsoft.com/pricing/free-trial/).
* Una cuenta de Media Services. Para crear una cuenta de Media Services, consulte el tema [Creación de una cuenta de Media Services](media-services-portal-create-account.md).
* Información sobre [cómo usar Azure Functions](../../azure-functions/functions-overview.md). Revise también [Enlaces de webhook y HTTP de Azure Functions](../../azure-functions/functions-bindings-http-webhook.md).

## <a name="create-a-function-app"></a>Creación de una aplicación de función

1. Vaya a [Azure Portal](https://portal.azure.com) e inicie sesión con su cuenta de Azure.
2. Cree una aplicación de función como se describe [aquí](../../azure-functions/functions-create-function-app-portal.md).

## <a name="configure-function-app-settings"></a>Configuración de la aplicación de función

Cuando desarrolle funciones de Media Services, es útil agregar variables de entorno que se usarán en todas sus funciones. Para definir la configuración de la aplicación, haga clic en el vínculo Configurar las opciones de la aplicación. 

En la sección de [configuración de la aplicación](media-services-dotnet-how-to-use-azure-functions.md#configure-function-app-settings) se definen los parámetros usados en el webhook definido en este artículo. Agregue también los siguientes parámetros a la configuración de la aplicación. 

|Nombre|Definición|Ejemplo| 
|---|---|---|
|SigningKey |Una clave de firma.| j0txf1f8msjytzvpe40nxbpxdcxtqcgxy0nt|
|WebHookEndpoint | Una dirección de punto de conexión webhook. Una vez creada la función del webhook, puede copiar la dirección URL del vínculo **Obtener la dirección URL de la función**. | https:\//juliakofuncapp.azurewebsites.net/api/Notification_Webhook_Function?code=iN2phdrTnCxmvaKExFWOTulfnm4C71mMLIy8tzLr7Zvf6Z22HHIK5g==.|

## <a name="create-a-function"></a>Creación de una función

Una vez implementada su instancia de Function App, puede encontrarla entre Azure Functions de **App Services**.

1. Seleccione la aplicación de función y haga clic en **Nueva función**.
2. Seleccione el código **C#** y el escenario **API y WebHooks**. 
3. Seleccione **Webhook genérico: C#** .
4. Asigne un nombre al webhook y presione **Crear**.

### <a name="files"></a>Archivos

La función de Azure está asociada a archivos de código y otros archivos que se describen en esta sección. De forma predeterminada, una función está asociada con los archivos **function.json** y **run.csx** (C#). Debe agregar un archivo **project.json**. El resto de esta sección muestra las definiciones de estos archivos.

![files](./media/media-services-azure-functions/media-services-azure-functions003.png)

#### <a name="functionjson"></a>function.json

El archivo function.json define los enlaces de función y otras opciones de configuración. Este archivo se usa en tiempo de ejecución para determinar los eventos que se supervisarán y cómo pasar datos y devolverlos al ejecutarse una función. 

```json
{
  "bindings": [
    {
      "type": "httpTrigger",
      "direction": "in",
      "webHookType": "genericJson",
      "name": "req"
    },
    {
      "type": "http",
      "direction": "out",
      "name": "res"
    }
  ],
  "disabled": false
}
```

#### <a name="projectjson"></a>project.json

El archivo project.json contiene dependencias. 

```json
{
  "frameworks": {
    "net46":{
      "dependencies": {
        "windowsazure.mediaservices": "4.0.0.4",
        "windowsazure.mediaservices.extensions": "4.0.0.4",
        "Microsoft.IdentityModel.Clients.ActiveDirectory": "3.13.1",
        "Microsoft.IdentityModel.Protocol.Extensions": "1.0.2.206221351"
      }
    }
   }
}
```
    
#### <a name="runcsx"></a>run.csx

El código de esta sección muestra una implementación de una función de Azure que es un webhook. En este ejemplo, la función escucha la devolución de llamada al webhook desde las notificaciones de Media Services y publica el recurso de salida cuando el trabajo ha finalizado.

El webhook espera que una clave de firma (credencial) coincida con la especificada al configurar el punto de conexión de la notificación. La clave de firma es el valor codificado en Base64 de 64 bits que se usa para proteger y asegurar sus devoluciones de llamada de WebHooks desde Azure Media Services. 

En el código de la definición del webhook, el método **VerifyWebHookRequestSignature** realiza la verificación del mensaje de notificación. El propósito de esta validación es asegurarse de que Azure Media Services envió el mensaje y que no se manipuló. La firma es opcional para Azure Functions, ya que tiene el valor **Código** como un parámetro de consulta a través de la Seguridad de la capa de transporte (TLS). 

>[!NOTE]
>Hay un límite de 1 000 000 directivas para diferentes directivas de AMS (por ejemplo, para la directiva de localizador o ContentKeyAuthorizationPolicy). Debe usar el mismo identificador de directiva si siempre usa los mismos permisos de acceso y días, por ejemplo, directivas para localizadores que vayan a aplicarse durante mucho tiempo (directivas distintas a carga). Para obtener más información, consulte [este tema](media-services-dotnet-manage-entities.md#limit-access-policies) .

```csharp
///////////////////////////////////////////////////
#r "Newtonsoft.Json"

using System;
using Microsoft.WindowsAzure.MediaServices.Client;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading;
using System.Threading.Tasks;
using System.IO;
using System.Globalization;
using Newtonsoft.Json;
using Microsoft.Azure;
using System.Net;
using System.Security.Cryptography;
using Microsoft.Azure.WebJobs;
using Microsoft.IdentityModel.Clients.ActiveDirectory;

internal const string SignatureHeaderKey = "sha256";
internal const string SignatureHeaderValueTemplate = SignatureHeaderKey + "={0}";
static string _webHookEndpoint = Environment.GetEnvironmentVariable("WebHookEndpoint");
static string _signingKey = Environment.GetEnvironmentVariable("SigningKey");

static readonly string _AADTenantDomain = Environment.GetEnvironmentVariable("AMSAADTenantDomain");
static readonly string _RESTAPIEndpoint = Environment.GetEnvironmentVariable("AMSRESTAPIEndpoint");

static readonly string _AMSClientId = Environment.GetEnvironmentVariable("AMSClientId");
static readonly string _AMSClientSecret = Environment.GetEnvironmentVariable("AMSClientSecret");

static CloudMediaContext _context = null;

public static async Task<HttpResponseMessage> Run(HttpRequestMessage req, TraceWriter log)
{
    log.Info($"C# HTTP trigger function processed a request. RequestUri={req.RequestUri}");

    Task<byte[]> taskForRequestBody = req.Content.ReadAsByteArrayAsync();
    byte[] requestBody = await taskForRequestBody;

    string jsonContent = await req.Content.ReadAsStringAsync();
    log.Info($"Request Body = {jsonContent}");

    IEnumerable<string> values = null;
    if (req.Headers.TryGetValues("ms-signature", out values))
    {
        byte[] signingKey = Convert.FromBase64String(_signingKey);
        string signatureFromHeader = values.FirstOrDefault();

        if (VerifyWebHookRequestSignature(requestBody, signatureFromHeader, signingKey))
        {
            string requestMessageContents = Encoding.UTF8.GetString(requestBody);

            NotificationMessage msg = JsonConvert.DeserializeObject<NotificationMessage>(requestMessageContents);

            if (VerifyHeaders(req, msg, log))
            { 
                string newJobStateStr = (string)msg.Properties.Where(j => j.Key == "NewState").FirstOrDefault().Value;
                if (newJobStateStr == "Finished")
                {
                    AzureAdTokenCredentials tokenCredentials = new AzureAdTokenCredentials(_AADTenantDomain,
                                new AzureAdClientSymmetricKey(_AMSClientId, _AMSClientSecret),
                                AzureEnvironments.AzureCloudEnvironment);

                    AzureAdTokenProvider tokenProvider = new AzureAdTokenProvider(tokenCredentials);

                    _context = new CloudMediaContext(new Uri(_RESTAPIEndpoint), tokenProvider);

                    if(_context!=null)   
                    {                        
                        string urlForClientStreaming = PublishAndBuildStreamingURLs(msg.Properties["JobId"]);
                        log.Info($"URL to the manifest for client streaming using HLS protocol: {urlForClientStreaming}");
                    }
                }

                return req.CreateResponse(HttpStatusCode.OK, string.Empty);
            }
            else
            {
                log.Info($"VerifyHeaders failed.");
                return req.CreateResponse(HttpStatusCode.BadRequest, "VerifyHeaders failed.");
            }
        }
        else
        {
            log.Info($"VerifyWebHookRequestSignature failed.");
            return req.CreateResponse(HttpStatusCode.BadRequest, "VerifyWebHookRequestSignature failed.");
        }
    }

    return req.CreateResponse(HttpStatusCode.BadRequest, "Generic Error.");
}

private static string PublishAndBuildStreamingURLs(String jobID)
{
    IJob job = _context.Jobs.Where(j => j.Id == jobID).FirstOrDefault();
    IAsset asset = job.OutputMediaAssets.FirstOrDefault();

    // Create a 30-day readonly access policy. 
    // You cannot create a streaming locator using an AccessPolicy that includes write or delete permissions.
    IAccessPolicy policy = _context.AccessPolicies.Create("Streaming policy",
    TimeSpan.FromDays(30),
    AccessPermissions.Read);

    // Create a locator to the streaming content on an origin. 
    ILocator originLocator = _context.Locators.CreateLocator(LocatorType.OnDemandOrigin, asset,
    policy,
    DateTime.UtcNow.AddMinutes(-5));

    // Get a reference to the streaming manifest file from the  
    // collection of files in the asset. 
    var manifestFile = asset.AssetFiles.ToList().Where(f => f.Name.ToLower().
                EndsWith(".ism")).
                FirstOrDefault();

    // Create a full URL to the manifest file. Use this for playback
    // in streaming media clients. 
    string urlForClientStreaming = originLocator.Path + manifestFile.Name + "/manifest" +  "(format=m3u8-aapl)";
    return urlForClientStreaming;

}

private static bool VerifyWebHookRequestSignature(byte[] data, string actualValue, byte[] verificationKey)
{
    using (var hasher = new HMACSHA256(verificationKey))
    {
        byte[] sha256 = hasher.ComputeHash(data);
        string expectedValue = string.Format(CultureInfo.InvariantCulture, SignatureHeaderValueTemplate, ToHex(sha256));

        return (0 == String.Compare(actualValue, expectedValue, System.StringComparison.Ordinal));
    }
}

private static bool VerifyHeaders(HttpRequestMessage req, NotificationMessage msg, TraceWriter log)
{
    bool headersVerified = false;

    try
    {
        IEnumerable<string> values = null;
        if (req.Headers.TryGetValues("ms-mediaservices-accountid", out values))
        {
            string accountIdHeader = values.FirstOrDefault();
            string accountIdFromMessage = msg.Properties["AccountId"];

            if (0 == string.Compare(accountIdHeader, accountIdFromMessage, StringComparison.OrdinalIgnoreCase))
            {
                headersVerified = true;
            }
            else
            {
                log.Info($"accountIdHeader={accountIdHeader} does not match accountIdFromMessage={accountIdFromMessage}");
            }
        }
        else
        {
            log.Info($"Header ms-mediaservices-accountid not found.");
        }
    }
    catch (Exception e)
    {
        log.Info($"VerifyHeaders hit exception {e}");
        headersVerified = false;
    }

    return headersVerified;
}

private static readonly char[] HexLookup = new char[] { '0', '1', '2', '3', '4', '5', '6', '7', '8', '9', 'A', 'B', 'C', 'D', 'E', 'F' };

/// <summary>
/// Converts a <see cref="T:byte[]"/> to a hex-encoded string.
/// </summary>
private static string ToHex(byte[] data)
{
    if (data == null)
    {
        return string.Empty;
    }

    char[] content = new char[data.Length * 2];
    int output = 0;
    byte d;

    for (int input = 0; input < data.Length; input++)
    {
        d = data[input];
        content[output++] = HexLookup[d / 0x10];
        content[output++] = HexLookup[d % 0x10];
    }

    return new string(content);
}

internal enum NotificationEventType
{
    None = 0,
    JobStateChange = 1,
    NotificationEndPointRegistration = 2,
    NotificationEndPointUnregistration = 3,
    TaskStateChange = 4,
    TaskProgress = 5
}

internal sealed class NotificationMessage
{
    public string MessageVersion { get; set; }
    public string ETag { get; set; }
    public NotificationEventType EventType { get; set; }
    public DateTime TimeStamp { get; set; }
    public IDictionary<string, string> Properties { get; set; }
}
```

Guarde y ejecute la función.

### <a name="function-output"></a>Salida de la función

Una vez desencadenado el webhook, el ejemplo anterior genera el siguiente resultado, pero sus valores variarán.

```output
C# HTTP trigger function processed a request. RequestUri=https://juliako001-functions.azurewebsites.net/api/otification_Webhook_Function?code=9376d69kygoy49oft81nel8frty5cme8hb9xsjslxjhalwhfrqd79awz8ic4ieku74dvkdfgvi
Request Body = 
{
  "MessageVersion": "1.1",
  "ETag": "b8977308f48858a8f224708bc963e1a09ff917ce730316b4e7ae9137f78f3b20",
  "EventType": 4,
  "TimeStamp": "2017-02-16T03:59:53.3041122Z",
  "Properties": {
    "JobId": "nb:jid:UUID:badd996c-8d7c-4ae0-9bc1-bd7f1902dbdd",
    "TaskId": "nb:tid:UUID:80e26fb9-ee04-4739-abd8-2555dc24639f",
    "NewState": "Finished",
    "OldState": "Processing",
    "AccountName": "mediapkeewmg5c3peq",
    "AccountId": "301912b0-659e-47e0-9bc4-6973f2be3424",
    "NotificationEndPointId": "nb:nepid:UUID:cb5d707b-4db8-45fe-a558-19f8d3306093"
  }
}

URL to the manifest for client streaming using HLS protocol: http://mediapkeewmg5c3peq.streaming.mediaservices.windows.net/0ac98077-2b58-4db7-a8da-789a13ac6167/BigBuckBunny.ism/manifest(format=m3u8-aapl)
```

## <a name="add-a-webhook-to-your-encoding-task"></a>Adición de un webhook a la tarea de codificación

En esta sección, se muestra el código que agrega una notificación de webhook a una tarea. También puede agregar una notificación de nivel de trabajo, que podría ser más útil para un trabajo con tareas encadenadas.  

1. Cree una aplicación de consola en C# mediante Visual Studio. Escriba el nombre, la ubicación y el nombre de la solución y, a continuación, haga clic en Aceptar.
2. Use [NuGet](https://www.nuget.org/packages/windowsazure.mediaservices) para instalar Azure Media Services.
3. Actualice el archivo App.config con los valores adecuados: 
    
   * información de conexión de Azure Media Services, 
   * dirección URL del webhook que espera recibir las notificaciones, 
   * la clave de firma que coincide con la clave que el webhook espera. La clave de firma es el valor codificado en Base64 de 64 bytes que se usa para proteger y asegurar sus devoluciones de llamada de WebHooks desde Azure Media Services. 

     ```xml
           <appSettings>
               <add key="AMSAADTenantDomain" value="domain" />
               <add key="AMSRESTAPIEndpoint" value="endpoint" />

               <add key="AMSClientId" value="clinet id" />
               <add key="AMSClientSecret" value="client secret" />

               <add key="WebhookURL" value="https://yourapp.azurewebsites.net/api/functionname?code=ApiKey" />
               <add key="WebhookSigningKey" value="j0txf1f8msjytzvpe40nxbpxdcxtqcgxy0nt" />
           </appSettings>
     ```

4. Actualice el archivo Program.cs con el código siguiente:

    ```csharp
            using System;
            using System.Configuration;
            using System.Linq;
            using Microsoft.WindowsAzure.MediaServices.Client;

            namespace NotificationWebHook
            {
                class Program
                {
                // Read values from the App.config file.
                private static readonly string _AMSAADTenantDomain =
                    ConfigurationManager.AppSettings["AMSAADTenantDomain"];
                private static readonly string _AMSRESTAPIEndpoint =
                    ConfigurationManager.AppSettings["AMSRESTAPIEndpoint"];

                private static readonly string _AMSClientId =
                    ConfigurationManager.AppSettings["AMSClientId"];
                private static readonly string _AMSClientSecret =
                    ConfigurationManager.AppSettings["AMSClientSecret"];

                private static readonly string _webHookEndpoint =
                    ConfigurationManager.AppSettings["WebhookURL"];
                private static readonly string _signingKey =
                    ConfigurationManager.AppSettings["WebhookSigningKey"];

                // Field for service context.
                private static CloudMediaContext _context = null;

                static void Main(string[] args)
                {
                    AzureAdTokenCredentials tokenCredentials = new AzureAdTokenCredentials(_AMSAADTenantDomain,
                        new AzureAdClientSymmetricKey(_AMSClientId, _AMSClientSecret),
                        AzureEnvironments.AzureCloudEnvironment);

                    AzureAdTokenProvider tokenProvider = new AzureAdTokenProvider(tokenCredentials);

                    _context = new CloudMediaContext(new Uri(_AMSRESTAPIEndpoint), tokenProvider);

                    byte[] keyBytes = Convert.FromBase64String(_signingKey);

                    IAsset newAsset = _context.Assets.FirstOrDefault();

                    // Check for existing Notification Endpoint with the name "FunctionWebHook"

                    var existingEndpoint = _context.NotificationEndPoints.Where(e => e.Name == "FunctionWebHook").FirstOrDefault();
                    INotificationEndPoint endpoint = null;

                    if (existingEndpoint != null)
                    {
                    Console.WriteLine("webhook endpoint already exists");
                    endpoint = (INotificationEndPoint)existingEndpoint;
                    }
                    else
                    {
                    endpoint = _context.NotificationEndPoints.Create("FunctionWebHook",
                        NotificationEndPointType.WebHook, _webHookEndpoint, keyBytes);
                    Console.WriteLine("Notification Endpoint Created with Key : {0}", keyBytes.ToString());
                    }

                    // Declare a new encoding job with the Standard encoder
                    IJob job = _context.Jobs.Create("MES Job");

                    // Get a media processor reference, and pass to it the name of the 
                    // processor to use for the specific task.
                    IMediaProcessor processor = GetLatestMediaProcessorByName("Media Encoder Standard");

                    ITask task = job.Tasks.AddNew("My encoding task",
                    processor,
                    "Adaptive Streaming",
                    TaskOptions.None);

                    // Specify the input asset to be encoded.
                    task.InputAssets.Add(newAsset);

                    // Add an output asset to contain the results of the job. 
                    // This output is specified as AssetCreationOptions.None, which 
                    // means the output asset is not encrypted. 
                    task.OutputAssets.AddNew(newAsset.Name, AssetCreationOptions.None);

                    // Add the WebHook notification to this Task and request all notification state changes.
                    // Note that you can also add a job level notification
                    // which would be more useful for a job with chained tasks.  
                    if (endpoint != null)
                    {
                    task.TaskNotificationSubscriptions.AddNew(NotificationJobState.All, endpoint, true);
                    Console.WriteLine("Created Notification Subscription for endpoint: {0}", _webHookEndpoint);
                    }
                    else
                    {
                    Console.WriteLine("No Notification Endpoint is being used");
                    }

                    job.Submit();

                    Console.WriteLine("Expect WebHook to be triggered for the Job ID: {0}", job.Id);
                    Console.WriteLine("Expect WebHook to be triggered for the Task ID: {0}", task.Id);

                    Console.WriteLine("Job Submitted");

                }
                private static IMediaProcessor GetLatestMediaProcessorByName(string mediaProcessorName)
                {
                    var processor = _context.MediaProcessors.Where(p => p.Name == mediaProcessorName).
                    ToList().OrderBy(p => new Version(p.Version)).LastOrDefault();

                    if (processor == null)
                    throw new ArgumentException(string.Format("Unknown media processor", mediaProcessorName));

                    return processor;
                }
                }
            }
    ```

## <a name="next-steps"></a>Pasos siguientes

[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Envío de comentarios
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]
