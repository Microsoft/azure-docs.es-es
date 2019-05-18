---
title: Transmisión de archivos de vídeo con Azure Media Services y la CLI de Azure | Microsoft Docs
description: Siga los pasos de este inicio rápido para crear una nueva cuenta de Azure Media Services, codificar un archivo y hacer streaming a Azure Media Player.
services: media-services
documentationcenter: ''
author: Juliako
manager: femila
editor: ''
keywords: azure media services, streaming
ms.service: media-services
ms.workload: media
ms.topic: quickstart
ms.custom: ''
ms.date: 02/19/2019
ms.author: juliako
ms.openlocfilehash: bd818135f289c430416909d16c8568208936a9f8
ms.sourcegitcommit: 0568c7aefd67185fd8e1400aed84c5af4f1597f9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2019
ms.locfileid: "65205418"
---
# <a name="quickstart-stream-video-files---cli"></a>Inicio rápido: Hacer streaming de archivos de vídeo: CLI

En este inicio rápido se muestra lo fácil que es codificar y transmitir vídeos en una amplia variedad de exploradores y dispositivos con Azure Media Services y la CLI de Azure. Se puede especificar contenido de entrada mediante direcciones URL HTTPS o SAS o rutas de acceso a archivos de Azure Blob Storage.

En el ejemplo de este artículo se codifica contenido que se hace accesible a través de una dirección URL HTTPS. Media Services v3 no admite actualmente la codificación de transferencia fragmentada a través de direcciones URL HTTPS.

Al final de este inicio rápido, podrá transmitir un vídeo.  

![Reproducción del vídeo](./media/stream-files-dotnet-quickstart/final-video.png)

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="create-a-media-services-account"></a>Creación de una cuenta de Media Services

Para poder cifrar, codificar, analizar, administrar y transmitir contenido multimedia en Azure, debe crear una cuenta de Media Services. Esa cuenta debe estar asociada con una o varias cuentas de almacenamiento.

La cuenta de Media Services y todas las cuentas de almacenamiento asociadas deben estar en la misma suscripción de Azure. Se recomienda usar cuentas de almacenamiento que estén en el mismo lugar que la cuenta de Media Services para limitar los costos de salida de datos y latencia.

### <a name="create-a-resource-group"></a>Crear un grupo de recursos

```azurecli
az group create -n amsResourceGroup -l westus2
```

### <a name="create-an-azure-storage-account"></a>Creación de una cuenta de Azure Storage

En este ejemplo, se va a crear una cuenta LRS estándar de uso general v2.

Si quiere experimentar con las cuentas de almacenamiento, use `--sku Standard_LRS`. Al seleccionar una SKU para producción, considere la posibilidad de usar `--sku Standard_RAGRS`, que proporciona replicación geográfica para la continuidad empresarial. Para más información, consulte los comandos [storage accounts](https://docs.microsoft.com/cli/azure/storage/account?view=azure-cli-latest).
 
```azurecli
az storage account create -n amsstorageaccount --kind StorageV2 --sku Standard_LRS -l westus2 -g amsResourceGroup
```

### <a name="create-an-azure-media-services-account"></a>Creación de una cuenta de Azure Media Services

```azurecli
az ams account create --n amsaccount -g amsResourceGroup --storage-account amsstorageaccount -l westus2
```

Obtendrá una respuesta parecida a esta:

```
{
  "id": "/subscriptions/<id>/resourceGroups/amsResourceGroup/providers/Microsoft.Media/mediaservices/amsaccount",
  "location": "West US 2",
  "mediaServiceId": "8b569c2e-d648-4fcb-9035-c7fcc3aa7ddf",
  "name": "amsaccount",
  "resourceGroup": "amsResourceGroupTest",
  "storageAccounts": [
    {
      "id": "/subscriptions/<id>/resourceGroups/amsResourceGroup/providers/Microsoft.Storage/storageAccounts/amsstorageaccount",
      "resourceGroup": "amsResourceGroupTest",
      "type": "Primary"
    }
  ],
  "tags": null,
  "type": "Microsoft.Media/mediaservices"
}
```

## <a name="start-the-streaming-endpoint"></a>Inicio del punto de conexión de streaming

El siguiente comando de la CLI de Azure inicia el **punto de conexión de streaming** predeterminado.

```azurecli
az ams streaming-endpoint start  -n default -a amsaccount -g amsResourceGroup
```

Obtendrá una respuesta parecida a esta:

```
az ams streaming-endpoint start  -n default -a amsaccount -g amsResourceGroup
{
  "accessControl": null,
  "availabilitySetName": null,
  "cdnEnabled": true,
  "cdnProfile": "AzureMediaStreamingPlatformCdnProfile-StandardVerizon",
  "cdnProvider": "StandardVerizon",
  "created": "2019-02-06T21:58:03.604954+00:00",
  "crossSiteAccessPolicies": null,
  "customHostNames": [],
  "description": "",
  "freeTrialEndTime": "2019-02-21T22:05:31.277936+00:00",
  "hostName": "amsaccount-usw22.streaming.media.azure.net",
  "id": "/subscriptions/<id>/resourceGroups/amsResourceGroup/providers/Microsoft.Media/mediaservices/amsaccount/streamingendpoints/default",
  "lastModified": "2019-02-06T21:58:03.604954+00:00",
  "location": "West US 2",
  "maxCacheAge": null,
  "name": "default",
  "provisioningState": "Succeeded",
  "resourceGroup": "amsResourceGroup",
  "resourceState": "Running",
  "scaleUnits": 0,
  "tags": {},
  "type": "Microsoft.Media/mediaservices/streamingEndpoints"
}
```

Si el punto de conexión de streaming ya está en ejecución, recibirá este mensaje:

```
(InvalidOperation) The server cannot execute the operation in its current state.
```

## <a name="create-a-transform-for-adaptive-bitrate-encoding"></a>Creación de una transformación para la codificación de velocidad de bits adaptable

Use una **transformación** para configurar tareas comunes de codificación o análisis de vídeos. En este ejemplo, se va a realizar una codificación de velocidad de bits adaptable. Luego, se enviará un trabajo con la transformación que se ha creado. El trabajo es la solicitud real a Media Services de que aplique la transformación a una entrada de contenido de vídeo o audio determinado.

```azurecli
az ams transform create --name testEncodingTransform --preset AdaptiveStreaming --description 'a simple Transform for Adaptive Bitrate Encoding' -g amsResourceGroup -a amsaccount
```

Obtendrá una respuesta parecida a esta:

```
{
  "created": "2019-02-15T00:11:18.506019+00:00",
  "description": "a simple Transform for Adaptive Bitrate Encoding",
  "id": "/subscriptions/<id>/resourceGroups/amsResourceGroup/providers/Microsoft.Media/mediaservices/amsaccount/transforms/testEncodingTransform",
  "lastModified": "2019-02-15T00:11:18.506019+00:00",
  "name": "testEncodingTransform",
  "outputs": [
    {
      "onError": "StopProcessingJob",
      "preset": {
        "odatatype": "#Microsoft.Media.BuiltInStandardEncoderPreset",
        "presetName": "AdaptiveStreaming"
      },
      "relativePriority": "Normal"
    }
  ],
  "resourceGroup": "amsResourceGroup",
  "type": "Microsoft.Media/mediaservices/transforms"
}
```

## <a name="create-an-output-asset"></a>Creación de un recurso de salida

Cree un **recurso** para usarlo como salida del trabajo de codificación.

```azurecli
az ams asset create -n testOutputAssetName -a amsaccount -g amsResourceGroup
```

Obtendrá una respuesta parecida a esta:

```
{
  "alternateId": null,
  "assetId": "96427438-bbce-4a74-ba91-e38179b72f36",
  "container": null,
  "created": "2019-02-14T23:58:19.127000+00:00",
  "description": null,
  "id": "/subscriptions/<id>/resourceGroups/amsResourceGroup/providers/Microsoft.Media/mediaservices/amsaccount/assets/testOutputAssetName",
  "lastModified": "2019-02-14T23:58:19.127000+00:00",
  "name": "testOutputAssetName",
  "resourceGroup": "amsResourceGroup",
  "storageAccountName": "amsstorageaccount",
  "storageEncryptionFormat": "None",
  "type": "Microsoft.Media/mediaservices/assets"
}
```

## <a name="start-a-job-by-using-https-input"></a>Inicio de un trabajo mediante la entrada HTTPS

Cuando se envían trabajos para procesar los vídeos, es necesario indicar a Media Services dónde encontrar el vídeo de entrada. Una de las opciones es especificar una dirección URL HTTPS como entrada de trabajo, como se muestra en este ejemplo.

Al ejecutar `az ams job start`, puede establecer una etiqueta en la salida del trabajo. Luego, puede usar la etiqueta para identificar la finalidad del recurso de salida.

- Si asigna un valor a la etiqueta, establezca "--output-assets" en "assetname=label".
- En caso contrario, establezca "--output-assets" en "assetname=".

  Observe que se agrega "=" a `output-assets`.

```azurecli
az ams job start --name testJob001 --transform-name testEncodingTransform --base-uri 'https://nimbuscdn-nimbuspm.streaming.mediaservices.windows.net/2b533311-b215-4409-80af-529c3e853622/' --files 'Ignite-short.mp4' --output-assets testOutputAssetName= -a amsaccount -g amsResourceGroup 
```

Obtendrá una respuesta parecida a esta:

```
{
  "correlationData": {},
  "created": "2019-02-15T05:08:26.266104+00:00",
  "description": null,
  "id": "/subscriptions/<id>/resourceGroups/amsResourceGroup/providers/Microsoft.Media/mediaservices/amsaccount/transforms/testEncodingTransform/jobs/testJob001",
  "input": {
    "baseUri": "https://nimbuscdn-nimbuspm.streaming.mediaservices.windows.net/2b533311-b215-4409-80af-529c3e853622/",
    "files": [
      "Ignite-short.mp4"
    ],
    "label": null,
    "odatatype": "#Microsoft.Media.JobInputHttp"
  },
  "lastModified": "2019-02-15T05:08:26.266104+00:00",
  "name": "testJob001",
  "outputs": [
    {
      "assetName": "testOutputAssetName",
      "error": null,
      "label": "",
      "odatatype": "#Microsoft.Media.JobOutputAsset",
      "progress": 0,
      "state": "Queued"
    }
  ],
  "priority": "Normal",
  "resourceGroup": "amsResourceGroup",
  "state": "Queued",
  "type": "Microsoft.Media/mediaservices/transforms/jobs"
}
```

### <a name="check-status"></a>Comprobar estado

En cinco minutos, compruebe el estado del trabajo. Debe mostrarse como finalizado. Si no, vuelva a comprobarlo en unos minutos. Cuando se muestre como finalizado, vaya al paso siguiente y cree un **localizador de streaming**.

```azurecli
az ams job show -a amsaccount -g amsResourceGroup -t testEncodingTransform -n testJob001
```

## <a name="create-a-streaming-locator-and-get-a-path"></a>Creación de un localizador de streaming y obtención de la ruta de acceso

Tras finalizar la codificación, el siguiente paso consiste en poner el vídeo del recurso de salida a disposición de los clientes para su reproducción. Para ello, seleccione "Create a Streaming Locator" (Crear un localizador de streaming). A continuación, cree direcciones URL de streaming que los clientes puedan usar.

### <a name="create-a-streaming-locator"></a>Creación de un localizador de streaming

```azurecli
az ams streaming-locator create -n testStreamingLocator --asset-name testOutputAssetName --streaming-policy-name Predefined_ClearStreamingOnly  -g amsResourceGroup -a amsaccount 
```

Obtendrá una respuesta parecida a esta:

```
{
  "alternativeMediaId": null,
  "assetName": "output-3b6d7b1dffe9419fa104b952f7f6ab76",
  "contentKeys": [],
  "created": "2019-02-15T04:35:46.270750+00:00",
  "defaultContentKeyPolicyName": null,
  "endTime": "9999-12-31T23:59:59.999999+00:00",
  "id": "/subscriptions/<id>/resourceGroups/amsResourceGroup/providers/Microsoft.Media/mediaservices/amsaccount/streamingLocators/testStreamingLocator",
  "name": "testStreamingLocator",
  "resourceGroup": "amsResourceGroup",
  "startTime": null,
  "streamingLocatorId": "e01b2be1-5ea4-42ca-ae5d-7fe704a5962f",
  "streamingPolicyName": "Predefined_ClearStreamingOnly",
  "type": "Microsoft.Media/mediaservices/streamingLocators"
}
```

### <a name="get-streaming-locator-paths"></a>Obtención de las rutas de acceso del localizador de streaming

```azurecli
az ams streaming-locator get-paths -a amsaccount -g amsResourceGroup -n testStreamingLocator
```

Obtendrá una respuesta parecida a esta:

```
{
  "downloadPaths": [],
  "streamingPaths": [
    {
      "encryptionScheme": "NoEncryption",
      "paths": [
        "/e01b2be1-5ea4-42ca-ae5d-7fe704a5962f/ignite.ism/manifest(format=m3u8-aapl)"
      ],
      "streamingProtocol": "Hls"
    },
    {
      "encryptionScheme": "NoEncryption",
      "paths": [
        "/e01b2be1-5ea4-42ca-ae5d-7fe704a5962f/ignite.ism/manifest(format=mpd-time-csf)"
      ],
      "streamingProtocol": "Dash"
    },
    {
      "encryptionScheme": "NoEncryption",
      "paths": [
        "/e01b2be1-5ea4-42ca-ae5d-7fe704a5962f/ignite.ism/manifest"
      ],
      "streamingProtocol": "SmoothStreaming"
    }
  ]
}
```

Copie la ruta de acceso de streaming en vivo (HLS) HTTP. En este caso, es `/e01b2be1-5ea4-42ca-ae5d-7fe704a5962f/ignite.ism/manifest(format=m3u8-aapl)`.

## <a name="build-the-url"></a>Creación de la dirección URL 

### <a name="get-the-streaming-endpoint-host-name"></a>Obtención del nombre de host del punto de conexión de streaming

```azurecli
az ams streaming-endpoint list -a amsaccount -g amsResourceGroup -n default
```
Copie el valor `hostName`. En este caso, es `amsaccount-usw22.streaming.media.azure.net`.

### <a name="assemble-the-url"></a>Ensamblado de la dirección URL

"https:// " + &lt;valor de hostName&gt; + &lt;valor de ruta de acceso de HLS&gt;

Este es un ejemplo:

`https://amsaccount-usw22.streaming.media.azure.net/7f19e783-927b-4e0a-a1c0-8a140c49856c/ignite.ism/manifest(format=m3u8-aapl)`

## <a name="test-playback-by-using-azure-media-player"></a>Prueba de reproducción con Azure Media Player

> [!NOTE]
> Si el reproductor está hospedado en un sitio HTTPS, asegúrese de iniciar la dirección URL con "https".

1. Abra un explorador web y vaya a [https://aka.ms/azuremediaplayer/](https://aka.ms/azuremediaplayer/).
2. En el cuadro **URL:**, pegue la dirección URL que creó en la sección anterior. Puede pegar la dirección URL en formato Smooth, Dash o HLS. Azure Media Player usarán automáticamente un protocolo de transmisión adecuado para la reproducción en el dispositivo.
3. Seleccione **Update Player** (Actualizar reproductor).

>[!NOTE]
>Azure Media Player puede usarse para realizar pruebas, pero no debe usarse en un entorno de producción.

## <a name="clean-up-resources"></a>Limpieza de recursos

Si ya no necesita ninguno de los recursos del grupo de recursos, incluidas las cuentas de almacenamiento y de Media Services que creó en este inicio rápido, elimine el grupo de recursos.

Ejecute este comando de la CLI:

```azurecli
az group delete --name amsResourceGroup
```

## <a name="see-also"></a>Otras referencias

Consulte [Códigos de error de trabajo](https://docs.microsoft.com/rest/api/media/jobs/get#joberrorcode).

## <a name="next-steps"></a>Pasos siguientes

[Introducción a Media Services](media-services-overview.md)
