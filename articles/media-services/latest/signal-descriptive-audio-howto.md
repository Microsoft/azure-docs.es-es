---
title: Señalización de pistas descriptivas de audio con Azure Media Services v3 | Microsoft Docs
description: Siga los pasos de este tutorial para cargar un archivo, codificar el vídeo, agregar pistas descriptivas de audio y transmitir en secuencias su contenido con Media Services v3.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.service: media-services
ms.workload: ''
ms.topic: how-to
ms.custom: devx-track-csharp
ms.date: 08/31/2020
ms.author: inhenkel
ms.openlocfilehash: 0bd2002e8f6f233361dd85cc08cfbd97ca23ed60
ms.sourcegitcommit: 58d3b3314df4ba3cabd4d4a6016b22fa5264f05a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89291269"
---
# <a name="signal-descriptive-audio-tracks"></a>Señalización de pistas descriptivas de audio

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

Puede agregar una pista de narración al vídeo que ayude a los clientes con problemas visuales a seguir la grabación de vídeo escuchando la narración. En Media Services v3, se señalan pistas de audio descriptivas al anotar la pista de audio en el archivo de manifiesto.

En este artículo se muestra cómo codificar un vídeo, cargar un archivo MP4 de solo audio (códec AAC) que contiene audio descriptivo en el recurso de salida y editar el archivo .ism para incluirlo.

## <a name="prerequisites"></a>Prerrequisitos

- [Cree una cuenta de Media Services](./create-account-howto.md).
- Siga los pasos de [Acceso a la API de Azure Media Services con la CLI de Azure](./access-api-howto.md) y guarde las credenciales. Deberá usarlas para acceder a la API.
- Consulte [Empaquetado dinámico](dynamic-packaging-overview.md).
- Consulte el tutorial [Carga, codificación y transmisión de vídeos con REST](stream-files-tutorial-with-api.md).

## <a name="create-an-input-asset-and-upload-a-local-file-into-it"></a>Creación de un recurso de entrada y carga de un archivo local en él 

La función **CreateInputAsset** crea un nuevo [recurso](/rest/api/media/assets) de entrada y carga en él el archivo de vídeo local especificado. Este **recurso** se utiliza como entrada para el trabajo de codificación. En Media Services v3, la entrada a un **trabajo** puede ser tanto un **recurso** como contenido que se pone a disposición de la cuenta de Media Services a través de direcciones URL HTTPS. 

Para más información sobre cómo codificar desde una dirección URL HTTPS, consulte [este artículo](job-input-from-http-how-to.md).  

En Media Services v3, se utilizan las API de Azure Storage para cargar archivos. En el siguiente fragmento de código de .NET se muestra cómo hacerlo.

La función siguiente realiza estas acciones:

* Crea un **recurso** 
* Obtiene una [dirección URL de SAS](../../storage/common/storage-sas-overview.md) que se puede escribir para el [contenedor de almacenamiento](../../storage/blobs/storage-quickstart-blobs-dotnet.md#upload-blobs-to-a-container) del recurso.
* Carga el archivo en el contenedor de almacenamiento mediante la dirección URL de SAS

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/UploadEncodeAndStreamFiles/Program.cs#CreateInputAsset)]

Si necesita pasar el nombre del recurso de entrada creado a otros métodos, asegúrese de usar la propiedad `Name` en el objeto de recurso devuelto de `CreateInputAssetAsync`, por ejemplo, inputAsset.Name. 

## <a name="create-an-output-asset-to-store-the-result-of-the-encoding-job"></a>Creación de un recurso de salida para almacenar el resultado del trabajo de codificación

El [recurso](/rest/api/media/assets) de salida almacena el resultado del trabajo de codificación. La siguiente función muestra cómo crear un recurso de salida.

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/UploadEncodeAndStreamFiles/Program.cs#CreateOutputAsset)]

Si tiene que pasar el nombre del recurso de salida creado a otros métodos, asegúrese de usar la propiedad `Name` en el objeto de recurso devuelto de `CreateIOutputAssetAsync`, por ejemplo, outputAsset.Name. 

En el caso de este artículo, pase el valor `outputAsset.Name` a las funciones `SubmitJobAsync` y `UploadAudioIntoOutputAsset`.

## <a name="create-a-transform-and-a-job-that-encodes-the-uploaded-file"></a>Creación de una transformación y de un trabajo que codifican el archivo cargado

Cuando se codifica o procesa contenido en Media Services, es un patrón común configurar los ajustes de codificación como una receta. Después, podría enviar un **trabajo** para aplicar esa receta a un vídeo. Al enviar nuevos trabajos en cada nuevo vídeo, está aplicando dicha receta a todos los vídeos de la biblioteca. Una receta en Media Services se llama **transformación**. Para obtener más información, consulte [Transformaciones y trabajos](./transforms-jobs-concept.md). El ejemplo descrito en este tutorial define una receta que codifica el vídeo para transmitirlo a varios dispositivos iOS y Android. 

En el ejemplo siguiente se crea una transformación (si no existe ninguna).

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/UploadEncodeAndStreamFiles/Program.cs#EnsureTransformExists)]

La función siguiente envía un trabajo.

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/UploadEncodeAndStreamFiles/Program.cs#SubmitJob)]

## <a name="wait-for-the-job-to-complete"></a>Espere a que el trabajo se complete

El trabajo tarda algún tiempo en completarse y cuando lo hace querrá recibir una notificación. Se recomienda usar Event Grid para esperar a que el trabajo se complete.

El trabajo pasa normalmente por los siguientes estados: **Programado**, **En cola**, **Procesando**, **Finalizado** (el estado final). Si el trabajo ha encontrado un error, obtendrá el estado **Error**. Si el trabajo está en proceso de cancelación, obtendrá **Cancelando** y **Cancelado** cuando haya terminado.

Para más información, consulte [Control de los eventos de Azure Event Grid](reacting-to-media-services-events.md).

## <a name="upload-the-audio-only-mp4-file"></a>Carga del archivo MP4 de solo audio

Cargue el archivo MP4 de solo audio adicional (códec AAC) que contiene el audio descriptivo en el recurso de salida.  

```csharp
private static async Task UpoadAudioIntoOutputAsset(
    IAzureMediaServicesClient client,
    string resourceGroupName,
    string accountName,
    string outputAssetName,
    string fileToUpload)
{
    // Use the Assets.Get method to get the existing asset. 
    // In Media Services v3, the Get method on entities returns null 
    // if the entity doesn't exist (a case-insensitive check on the name).

    // Call Media Services API to create an Asset.
    // This method creates a container in storage for the Asset.
    // The files (blobs) associated with the asset will be stored in this container.
    Asset asset = await client.Assets.GetAsync(resourceGroupName, accountName, outputAssetName);
    
    if (asset != null)
    {
      // Use Media Services API to get back a response that contains
      // SAS URL for the Asset container into which to upload blobs.
      // That is where you would specify read-write permissions 
      // and the exparation time for the SAS URL.
      var response = await client.Assets.ListContainerSasAsync(
          resourceGroupName,
          accountName,
          outputAssetName,
          permissions: AssetContainerPermission.ReadWrite,
          expiryTime: DateTime.UtcNow.AddHours(4).ToUniversalTime());

      var sasUri = new Uri(response.AssetContainerSasUrls.First());

      // Use Storage API to get a reference to the Asset container
      // that was created by calling Asset's CreateOrUpdate method.  
      CloudBlobContainer container = new CloudBlobContainer(sasUri);
      var blob = container.GetBlockBlobReference(Path.GetFileName(fileToUpload));

      // Use Strorage API to upload the file into the container in storage.
      await blob.UploadFromFileAsync(fileToUpload);
    }
}
```

Este es un ejemplo de una llamada a la función `UpoadAudioIntoOutputAsset`:

```csharp
await UpoadAudioIntoOutputAsset(client, config.ResourceGroup, config.AccountName, outputAsset.Name, "audio_description.m4a");
```

## <a name="edit-the-ism-file"></a>Edición del archivo .ism

Cuando haya finalizado el trabajo de codificación, el recurso de salida contendrá los archivos generados por el trabajo de codificación. 

1. En Azure Portal, vaya a la cuenta de almacenamiento asociada a su cuenta de Media Services. 
1. Busque el contenedor con el nombre del recurso de salida. 
1. En el contenedor, busque el archivo .ism y haga clic en **Editar blob** (en la ventana derecha). 
1. Para editar el archivo .ism, agregue la información sobre el archivo MP4 de solo audio cargado (códec AAC) que contenga el audio descriptivo y presione **Guardar** cuando haya terminado.

    Para señalizar las pistas de audio descriptivas, debe agregar los parámetros "accessibility" y "role" al archivo .ism. Debe establecer estos parámetros correctamente para señalizar las pistas de audio como descripciones de audio. Por ejemplo, agregue `<param name="accessibility" value="description" />` y `<param name="role" value="alternate" />` al archivo .ism para una pista de audio concreta, según se muestra en el ejemplo siguiente.
 
```xml
<?xml version="1.0" encoding="utf-8"?>
<smil xmlns="http://www.w3.org/2001/SMIL20/Language">
  <head>
    <meta name="clientManifestRelativePath" content="ignite.ismc" />
    <meta name="formats" content="mp4-v3" />
  </head>
  <body>
    <switch>
      <audio src="ignite_320x180_AACAudio_381.mp4" systemBitrate="128041" systemLanguage="eng">
        <param name="systemBitrate" value="128041" valuetype="data" />
        <param name="trackID" value="2" valuetype="data" />
        <param name="trackName" value="aac_eng_2_128041_2_1" valuetype="data" />
        <param name="systemLanguage" value="eng" valuetype="data" />
        <param name="trackIndex" value="ignite_320x180_AACAudio_381_2.mpi" valuetype="data" />
      </audio>
      <audio src="audio_description.m4a" systemBitrate="194000" systemLanguage="eng">
        <param name="trackName" value="aac_eng_audio_description" />
        <param name="accessibility" value="description" />
        <param name="role" value="alternate" />     
      </audio>          
      <video src="ignite_1280x720_AACAudio_3549.mp4" systemBitrate="3549855">
        <param name="systemBitrate" value="3549855" valuetype="data" />
        <param name="trackID" value="1" valuetype="data" />
        <param name="trackName" value="video" valuetype="data" />
        <param name="trackIndex" value="ignite_1280x720_AACAudio_3549_1.mpi" valuetype="data" />
      </video>
      <video src="ignite_960x540_AACAudio_2216.mp4" systemBitrate="2216764">
        <param name="systemBitrate" value="2216764" valuetype="data" />
        <param name="trackID" value="1" valuetype="data" />
        <param name="trackName" value="video" valuetype="data" />
        <param name="trackIndex" value="ignite_960x540_AACAudio_2216_1.mpi" valuetype="data" />
      </video>
      <video src="ignite_640x360_AACAudio_1154.mp4" systemBitrate="1154569">
        <param name="systemBitrate" value="1154569" valuetype="data" />
        <param name="trackID" value="1" valuetype="data" />
        <param name="trackName" value="video" valuetype="data" />
        <param name="trackIndex" value="ignite_640x360_AACAudio_1154_1.mpi" valuetype="data" />
      </video>
      <video src="ignite_480x270_AACAudio_721.mp4" systemBitrate="721893">
        <param name="systemBitrate" value="721893" valuetype="data" />
        <param name="trackID" value="1" valuetype="data" />
        <param name="trackName" value="video" valuetype="data" />
        <param name="trackIndex" value="ignite_480x270_AACAudio_721_1.mpi" valuetype="data" />
      </video>
      <video src="ignite_320x180_AACAudio_381.mp4" systemBitrate="381027">
        <param name="systemBitrate" value="381027" valuetype="data" />
        <param name="trackID" value="1" valuetype="data" />
        <param name="trackName" value="video" valuetype="data" />
        <param name="trackIndex" value="ignite_320x180_AACAudio_381_1.mpi" valuetype="data" />
      </video>
    </switch>
  </body>
</smil>
```

## <a name="get-a-streaming-locator"></a>Obtención de un localizador de streaming

Una vez finalizada la codificación, el siguiente paso es poner el vídeo del recurso de salida a disposición de los clientes para su reproducción. Puede hacerlo en dos pasos: en primer lugar, cree un objeto [StreamingLocator](/rest/api/media/streaminglocators) y, después, cree las direcciones URL de streaming que pueden usar los clientes. 

El proceso de creación de un objeto **StreamingLocator** se denomina publicación. De forma predeterminada, el objeto **StreamingLocator** es válido inmediatamente después de realizar las llamadas a la API y dura hasta que se elimina, salvo que configure las horas de inicio y de finalización opcionales. 

Al crear un objeto [StreamingLocator](/rest/api/media/streaminglocators), debe especificar el objeto **StreamingPolicyName** deseado. En este ejemplo, estará transmitiendo contenido no cifrado, de modo que se utiliza la directiva de streaming sin cifrar predefinida (**PredefinedStreamingPolicy.ClearStreamingOnly**).

> [!IMPORTANT]
> Al utilizar un objeto [StreamingPolicy](/rest/api/media/streamingpolicies) personalizado, debe diseñar un conjunto limitado de dichas directivas para su cuenta de Media Service y reutilizarlas para sus localizadores de streaming siempre que se necesiten las mismas opciones y protocolos de cifrado. La cuenta de Media Service tiene una cuota para el número de entradas de Streaming Policy. No debe crear una nueva directiva de streaming para cada localizador de streaming.

El código siguiente da por supuesto que está llamando a la función con un único objeto locatorName.

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/UploadEncodeAndStreamFiles/Program.cs#CreateStreamingLocator)]

Aunque el ejemplo de este tema trata del streaming, puede utilizar la misma llamada para crear un objeto StreamingLocator para la entrega de vídeo mediante descarga progresiva.

### <a name="get-streaming-urls"></a>Obtención de direcciones URL de streaming

Ahora que se ha creado el objeto [StreamingLocator](/rest/api/media/streaminglocators), puede obtener las direcciones URL de streaming, como se muestra en **GetStreamingURLs**. Para crear una dirección URL, debe concatenar el nombre de host del [punto de conexión de streaming](/rest/api/media/streamingendpoints) y la ruta de acceso del objeto **StreamingLocator**. En este ejemplo, se usa el *punto de conexión de* **streaming predeterminado**. Cuando cree su primera cuenta de Media Services, este *punto de conexión de streaming* **predeterminado** estará en un estado detenido, por lo que deberá llamar a **Iniciar**.

> [!NOTE]
> En este método, se necesita el objeto locatorName que se utilizó al crear el objeto **StreamingLocator** del recurso de salida.

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/UploadEncodeAndStreamFiles/Program.cs#GetStreamingURLs)]

## <a name="test-with-azure-media-player"></a>Prueba con Azure Media Player

Para probar el streaming, este artículo usa Azure Media Player. 

> [!NOTE]
> Si el reproductor está hospedado en un sitio https, asegúrese de actualizar la dirección URL a "https".

1. Abra un explorador web y vaya a [https://aka.ms/azuremediaplayer/](https://aka.ms/azuremediaplayer/).
2. En el cuadro **Dirección URL:** , pegue uno de los valores de la dirección URL de streaming que se obtuvo en la aplicación. 
 
     Puede pegar la dirección URL en formato HLS, Dash o Smooth y Azure Media Player cambiará automáticamente a un protocolo de streaming adecuado en su dispositivo.
3. Presione **Actualizar Player**.

Azure Media Player puede usarse para realizar pruebas, pero no debe usarse en un entorno de producción. 

## <a name="next-steps"></a>Pasos siguientes

[Análisis de vídeos](analyze-videos-tutorial-with-api.md)
