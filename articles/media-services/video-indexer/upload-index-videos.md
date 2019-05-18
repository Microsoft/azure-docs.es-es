---
title: Carga e indexación de vídeos con Video Indexer
titlesuffix: Azure Media Services
description: En este tema se muestra cómo usar las API para cargar e indexar los vídeos con Video Indexer.
services: media-services
author: Juliako
manager: femila
ms.service: media-services
ms.subservice: video-indexer
ms.topic: article
ms.date: 05/15/2019
ms.author: juliako
ms.openlocfilehash: e92086ca18887b9b2c2362e97d855c33834b83bb
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65799203"
---
# <a name="upload-and-index-your-videos"></a>Carga e indexación de los vídeos  

Al cargar vídeos con Video Indexer API, tiene las siguientes opciones de carga: 

* cargar el vídeo desde una dirección URL (opción preferida);
* enviar el archivo de vídeo como una matriz de bytes en el cuerpo de la solicitud.
* Usar el recurso de Azure Media Services existente proporcionando el [Id. de recurso](https://docs.microsoft.com/azure/media-services/latest/assets-concept) (compatible con solo cuentas de pago).

En el artículo se muestra cómo usar la API [Cargar vídeo](https://api-portal.videoindexer.ai/docs/services/operations/operations/Upload-video?) para cargar e indexar los vídeos según una dirección URL. En el ejemplo de código del artículo se incluye el código con comentarios que muestra cómo cargar la matriz de bytes. <br/>En el artículo también se explican algunos de los parámetros que puede establecer en la API para cambiar el proceso y la salida de la API.

Una vez cargado el vídeo, Video Indexer codifica opcionalmente el vídeo (lo cual se explica en este artículo). Al crear una cuenta de Video Indexer, puede elegir una cuenta de evaluación gratuita (donde obtendrá un número determinado de minutos gratuitos de indexación) o una opción de pago (donde no está limitado por la cuota). Con la versión de evaluación gratuita, Video Indexer proporciona hasta 600 horas de indexación gratuita a los usuarios de sitios web y hasta 2400 horas de indexación gratuita a los usuarios de API. Con la opción de pago, se crea una cuenta de Video Indexer que está [conectada a su suscripción de Azure y a una cuenta de Azure Media Services](connect-to-azure.md). Se paga por minutos de indexación y por los cargos relacionados con la cuenta de Media Services. 

## <a name="uploading-considerations"></a>Consideraciones de la carga

- Al cargar el vídeo según la dirección URL (opción preferida) el punto de conexión debe estar protegido con TLS 1.2 (o una versión posterior).
- El tamaño de carga con la opción de dirección URL se limita a 30GB
- La longitud de la dirección URL de solicitud se limita a 2048 caracteres
- El tamaño de carga con la opción de matriz de bytes se limita a 2 GB
- La opción de matriz de bytes agota el tiempo de espera a los 30 minutos
- La dirección URL que se proporciona en el parámetro `videoURL` debe estar codificada.
- Indización de recursos de Media Services tiene la misma limitación que la indización de dirección URL
- Indexador de vídeo tiene un límite de duración máxima de 4 horas para un solo archivo

> [!Tip]
> Es recomendable que use .NET Framework versión 4.6.2 o superior porque las versiones anteriores de .NET Framework no usan de forma predeterminada TLS 1.2.
>
> Si debe usar versiones anteriores de .NET Framework, agregue una línea en el código antes de realizar la llamada a la API REST:  <br/> System.Net.ServicePointManager.SecurityProtocol = SecurityProtocolType.Tls | SecurityProtocolType.Tls11 | SecurityProtocolType.Tls12;

## <a name="configurations-and-params"></a>Configuración y parámetros

En esta sección se describen algunos de los parámetros opcionales y cuándo es conveniente establecerlos.

### <a name="externalid"></a>externalId 

Este parámetro permite especificar un identificador que se asociará con el vídeo. El identificador puede aplicarse a la integración del sistema de "Administración de contenido de vídeo" (VCM) externa. Los vídeos del portal de Video Indexer se pueden buscar con el identificador externo especificado.

### <a name="callbackurl"></a>callbackUrl

Una dirección URL que se usa para notificar al cliente (mediante una solicitud POST) los siguientes eventos:

- Indexación de los cambios de estado: 
    - Propiedades:    
    
        |NOMBRE|Descripción|
        |---|---|
        |id|Id. de vídeo|
        |state|El estado del vídeo|  
    - Ejemplo: https://test.com/notifyme?projectName=MyProject&id=1234abcd&state=Processed
- Persona identificada en el vídeo:
  - Properties (Propiedades)
    
      |NOMBRE|Descripción|
      |---|---|
      |id| Id. de vídeo|
      |faceId|El identificador de caras que aparece en el índice de vídeo|
      |knownPersonId|El identificador de persona que es único dentro de un modelo de cara|
      |personName|El nombre de la persona|
        
    - Ejemplo: https://test.com/notifyme?projectName=MyProject&id=1234abcd&faceid=12&knownPersonId=CCA84350-89B7-4262-861C-3CAC796542A5&personName=Inigo_Montoya 

#### <a name="notes"></a>Notas

- Video Indexer devuelve los parámetros existentes proporcionados en la dirección URL original.
- La dirección URL proporcionada debe estar codificada.

### <a name="indexingpreset"></a>indexingPreset

Utilice este parámetro si hay grabaciones sin formato o externas que contengan ruido de fondo. Este parámetro se utiliza para configurar el proceso de indexación. Puede especificar los siguientes valores:

- `Default`: indexar y extraer información detallada con audio y vídeo
- `AudioOnly`: indexar y extraer información detallada solo con audio (e ignorar el vídeo)
- `DefaultWithNoiseReduction`: indexar y extraer información detallada de audio y vídeo, mientras se aplican algoritmos de reducción de ruido en la transmisión de audio

El precio depende de la opción de indexación seleccionada.  

### <a name="priority"></a>prioridad

Video Indexer indexa los vídeos según su prioridad. Use el parámetro **priority** para especificar la prioridad del índice. Los siguientes valores son válidos: **Low**, **Normal** (valor predeterminado) y **High**.

El parámetro **priority** solo se admite en cuentas de pago.

### <a name="streamingpreset"></a>streamingPreset

Una vez cargado el vídeo, Video Indexer, codifica opcionalmente el vídeo. Después, continúa con la indexación y el análisis del vídeo. Cuando Video Indexer ha terminado de analizar, recibirá una notificación con el identificador de vídeo.  

Cuando se usa [Upload Video](https://api-portal.videoindexer.ai/docs/services/operations/operations/Upload-video?) o [Re-Index Video](https://api-portal.videoindexer.ai/docs/services/operations/operations/Re-index-video?) API, uno de los parámetros opcionales es `streamingPreset`. Si establece `streamingPreset` en `Default`, `SingleBitrate` o en `AdaptiveBitrate`, se desencadenará el proceso de codificación. Una vez realizados los trabajos de indexación y codificación, el vídeo se publica para que también pueda transmitirlo. El punto de conexión de streaming desde el que va a transmitir el vídeo debe estar en estado **Running** (En ejecución).

Para ejecutar los trabajos de indexación y codificación, la cuenta de [Azure Media Services conectada a la cuenta de Video Indexer](connect-to-azure.md) requiere unidades reservadas. Para más información, consulte [Escalado del procesamiento de elementos multimedia](https://docs.microsoft.com/azure/media-services/previous/media-services-scale-media-processing-overview). Como son trabajos de proceso intensivo, se recomienda encarecidamente el tipo de unidad S3. El número de unidades reservadas define el número máximo de trabajos que se pueden ejecutar en paralelo. La recomendación de referencia son 10 unidades reservadas S3. 

Si solo desea indexar el vídeo pero no codificarlo, establezca `streamingPreset` en `NoStreaming`.

### <a name="videourl"></a>videoUrl

Una dirección URL del archivo de audio y vídeo que se va a indexar. La dirección URL debe apuntar a un archivo multimedia (no se admiten páginas HTML). El archivo se puede proteger mediante un token de acceso que se proporciona como parte del URI y el punto de conexión que se aplica al archivo debe estar protegido con TLS 1.2 o una versión posterior. La dirección URL debe estar codificada. 

Si el parámetro `videoUrl` no se especifica, Video Indexer espera que pase el archivo como contenido del cuerpo de varias partes o formulario.

## <a name="code-sample"></a>Código de ejemplo

El siguiente fragmento de código de C# muestra el uso de todas las API de Video Indexer juntas.

```csharp
public async Task Sample()
{
    var apiUrl = "https://api.videoindexer.ai";
    var location = "westus2";
    var apiKey = "...";

    System.Net.ServicePointManager.SecurityProtocol =
        System.Net.ServicePointManager.SecurityProtocol | System.Net.SecurityProtocolType.Tls12;

    // create the http client
    var handler = new HttpClientHandler();
    handler.AllowAutoRedirect = false;
    var client = new HttpClient(handler);
    client.DefaultRequestHeaders.Add("Ocp-Apim-Subscription-Key", apiKey);

    // obtain account information and access token
    string queryParams = CreateQueryString(
        new Dictionary<string, string>()
        {
            {"generateAccessTokens", "true"},
            {"allowEdit", "true"},
        });
    HttpResponseMessage result = await client.GetAsync($"{apiUrl}/auth/trial/Accounts?{queryParams}");
    var json = await result.Content.ReadAsStringAsync();
    var accounts = JsonConvert.DeserializeObject<AccountContractSlim[]>(json);
    // take the relevant account, here we simply take the first
    var accountInfo = accounts.First();

    // we will use the access token from here on, no need for the apim key
    client.DefaultRequestHeaders.Remove("Ocp-Apim-Subscription-Key");

    // upload a video
    var content = new MultipartFormDataContent();
    Debug.WriteLine("Uploading...");
    // get the video from URL
    var videoUrl = "VIDEO_URL"; // replace with the video URL

    // as an alternative to specifying video URL, you can upload a file.
    // remove the videoUrl parameter from the query params below and add the following lines:
    //FileStream video =File.OpenRead(Globals.VIDEOFILE_PATH);
    //byte[] buffer =newbyte[video.Length];
    //video.Read(buffer, 0, buffer.Length);
    //content.Add(newByteArrayContent(buffer));

    queryParams = CreateQueryString(
        new Dictionary<string, string>()
        {
            {"accessToken", accountInfo.AccessToken},
            {"name", "video_name"},
            {"description", "video_description"},
            {"privacy", "private"},
            {"partition", "partition"},
            {"videoUrl", videoUrl},
        });
    var uploadRequestResult = await client.PostAsync($"{apiUrl}/{accountInfo.Location}/Accounts/{accountInfo.Id}/Videos?{queryParams}", content);
    var uploadResult = await uploadRequestResult.Content.ReadAsStringAsync();

    // get the video ID from the upload result
    string videoId = JsonConvert.DeserializeObject<dynamic>(uploadResult)["id"];
    Debug.WriteLine("Uploaded");
    Debug.WriteLine("Video ID:");
    Debug.WriteLine(videoId);

    // wait for the video index to finish
    while (true)
    {
        await Task.Delay(10000);

        queryParams = CreateQueryString(
            new Dictionary<string, string>()
            {
                {"accessToken", accountInfo.AccessToken},
                {"language", "English"},
            });

        var videoGetIndexRequestResult = await client.GetAsync($"{apiUrl}/{accountInfo.Location}/Accounts/{accountInfo.Id}/Videos/{videoId}/Index?{queryParams}");
        var videoGetIndexResult = await videoGetIndexRequestResult.Content.ReadAsStringAsync();

        string processingState = JsonConvert.DeserializeObject<dynamic>(videoGetIndexResult)["state"];

        Debug.WriteLine("");
        Debug.WriteLine("State:");
        Debug.WriteLine(processingState);

        // job is finished
        if (processingState != "Uploaded" && processingState != "Processing")
        {
            Debug.WriteLine("");
            Debug.WriteLine("Full JSON:");
            Debug.WriteLine(videoGetIndexResult);
            break;
        }
    }

    // search for the video
    queryParams = CreateQueryString(
        new Dictionary<string, string>()
        {
            {"accessToken", accountInfo.AccessToken},
            {"id", videoId},
        });

    var searchRequestResult = await client.GetAsync($"{apiUrl}/{accountInfo.Location}/Accounts/{accountInfo.Id}/Videos/Search?{queryParams}");
    var searchResult = await searchRequestResult.Content.ReadAsStringAsync();
    Debug.WriteLine("");
    Debug.WriteLine("Search:");
    Debug.WriteLine(searchResult);

    // Generate video access token (used for get widget calls)
    client.DefaultRequestHeaders.Add("Ocp-Apim-Subscription-Key", apiKey);
    var videoTokenRequestResult = await client.GetAsync($"{apiUrl}/auth/{accountInfo.Location}/Accounts/{accountInfo.Id}/Videos/{videoId}/AccessToken?allowEdit=true");
    var videoAccessToken = (await videoTokenRequestResult.Content.ReadAsStringAsync()).Replace("\"", "");
    client.DefaultRequestHeaders.Remove("Ocp-Apim-Subscription-Key");

    // get insights widget url
    queryParams = CreateQueryString(
        new Dictionary<string, string>()
        {
            {"accessToken", videoAccessToken},
            {"widgetType", "Keywords"},
            {"allowEdit", "true"},
        });
    var insightsWidgetRequestResult = await client.GetAsync($"{apiUrl}/{accountInfo.Location}/Accounts/{accountInfo.Id}/Videos/{videoId}/InsightsWidget?{queryParams}");
    var insightsWidgetLink = insightsWidgetRequestResult.Headers.Location;
    Debug.WriteLine("Insights Widget url:");
    Debug.WriteLine(insightsWidgetLink);

    // get player widget url
    queryParams = CreateQueryString(
        new Dictionary<string, string>()
        {
            {"accessToken", videoAccessToken},
        });
    var playerWidgetRequestResult = await client.GetAsync($"{apiUrl}/{accountInfo.Location}/Accounts/{accountInfo.Id}/Videos/{videoId}/PlayerWidget?{queryParams}");
    var playerWidgetLink = playerWidgetRequestResult.Headers.Location;
    Debug.WriteLine("");
    Debug.WriteLine("Player Widget url:");
    Debug.WriteLine(playerWidgetLink);
}

private string CreateQueryString(IDictionary<string, string> parameters)
{
    var queryParameters = HttpUtility.ParseQueryString(string.Empty);
    foreach (var parameter in parameters)
    {
        queryParameters[parameter.Key] = parameter.Value;
    }

    return queryParameters.ToString();
}

public class AccountContractSlim
{
    public Guid Id { get; set; }
    public string Name { get; set; }
    public string Location { get; set; }
    public string AccountType { get; set; }
    public string Url { get; set; }
    public string AccessToken { get; set; }
}
```
## <a name="common-errors"></a>Errores comunes

La operación Upload puede devolver los códigos de estado que aparecen en la siguiente tabla.

|Código de estado|ErrorType (en el cuerpo de la respuesta)|DESCRIPCIÓN|
|---|---|---|
|400|VIDEO_ALREADY_IN_PROGRESS|El mismo vídeo ya se está procesando en la cuenta especificada.|
|400|VIDEO_ALREADY_FAILED|El mismo vídeo no se pudo procesar en la cuenta especificada hace menos de 2 horas. Los clientes de API deberán esperar al menos 2 horas antes de volver a cargar un vídeo.|

## <a name="next-steps"></a>Pasos siguientes

[Examine la salida de Azure Video Indexer producida por la API](video-indexer-output-json-v2.md)
