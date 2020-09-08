---
title: Streaming en vivo con Media Services v3
titleSuffix: Azure Media Services
description: Obtenga información sobre como hacer streaming en vivo con Azure Media Services v3.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.custom: mvc, devx-track-csharp
ms.date: 06/13/2019
ms.author: inhenkel
ms.openlocfilehash: b2e456474a9d052d9515c8169ce233e9577a5c53
ms.sourcegitcommit: bcda98171d6e81795e723e525f81e6235f044e52
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89256572"
---
# <a name="tutorial-stream-live-with-media-services"></a>Tutorial: Streaming en vivo con Media Services v3

> [!NOTE]
> Aunque en este tutorial se usan los ejemplos de [SDK de .NET](/dotnet/api/microsoft.azure.management.media.models.liveevent?view=azure-dotnet), los pasos generales son los mismos para la [API REST](/rest/api/media/liveevents), la [CLI](/cli/azure/ams/live-event?view=azure-cli-latest) u otros [SDK](media-services-apis-overview.md#sdks) admitidos.

En Azure Media Services, los objetos [LiveEvent](/rest/api/media/liveevents) son los responsables de procesar el contenido de streaming en vivo. Un objeto LiveEvent proporciona un punto de conexión de entrada (dirección URL de ingesta) que luego se ofrece a un codificador en directo. El objeto LiveEvent recibe flujos de entrada en vivo del codificador en directo y hace que estén disponibles para streaming con uno o varios [puntos de conexión de streaming](/rest/api/media/streamingendpoints). Los objetos LiveEvent también proporcionan un punto de conexión de vista previa (dirección URL de vista previa) que se puede utilizar para obtener una vista previa y validar el flujo antes de un procesamiento y entrega ulteriores. En este tutorial se muestra cómo usar .NET Core para un tipo de **paso a través** de un evento en directo.

En este tutorial se muestra cómo realizar las siguientes acciones:

> [!div class="checklist"]
> * Descargue la aplicación de ejemplo que se describe en el tema
> * Examinar el código que realiza el streaming en vivo.
> * Ver el evento con [Azure Media Player](https://amp.azure.net/libs/amp/latest/docs/index.html) en [https://ampdemo.azureedge.net](https://ampdemo.azureedge.net).
> * Limpieza de recursos.

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Prerrequisitos

Los siguientes elementos son necesarios para completar el tutorial:

- Instalación de Visual Studio Code o Visual Studio.
- [Cree una cuenta de Media Services](./create-account-howto.md).<br/>Asegúrese de recordar los valores que usó para el nombre del grupo de recursos y de la cuenta de Media Services.
- Siga los pasos de [Acceso a la API de Azure Media Services con la CLI de Azure](./access-api-howto.md) y guarde las credenciales. Deberá usarlas para acceder a la API.
- Una cámara o un dispositivo (como un equipo portátil) que se utiliza para difundir un evento.
- Un codificador en directo local que convierte las señales de la cámara en secuencias que se envían a un servicio de streaming en vivo de Media Services, consulte [Codificadores en directo locales recomendados](recommended-on-premises-live-encoders.md). La secuencia debe estar en formato **RTMP** o **Smooth Streaming**.

> [!TIP]
> Asegúrese de revisar [Streaming en vivo con Media Services v3](live-streaming-overview.md) antes de continuar. 

## <a name="download-and-configure-the-sample"></a>Descarga y configuración del ejemplo

Clone un repositorio GitHub que contenga el ejemplo de .NET de streaming en la máquina con el siguiente comando:  

 ```bash
 git clone https://github.com/Azure-Samples/media-services-v3-dotnet-core-tutorials.git
 ```

El ejemplo de streaming en vivo se encuentra en carpeta [Live](https://github.com/Azure-Samples/media-services-v3-dotnet-core-tutorials/tree/master/NETCore/Live/MediaV3LiveApp).

Abra [appsettings.json](https://github.com/Azure-Samples/media-services-v3-dotnet-core-tutorials/blob/master/NETCore/Live/MediaV3LiveApp/appsettings.json) en el proyecto que ha descargado. Sustituya los valores por las credenciales que obtuvo al [acceder a las API](./access-api-howto.md).

> [!IMPORTANT]
> Este ejemplo utiliza un sufijo único para cada recurso. Si cancela la depuración o termina la aplicación sin ejecutarla, acabará con varios objetos eventos en directo en la cuenta. <br/>Asegúrese de detener los objetos LiveEvent en ejecución. En caso contrario, se le **facturará** por ellos.

## <a name="examine-the-code-that-performs-live-streaming"></a>Examen del código que realiza el streaming en vivo

En esta sección se examinan las funciones definidas en el archivo [Program.cs](https://github.com/Azure-Samples/media-services-v3-dotnet-core-tutorials/blob/master/NETCore/Live/MediaV3LiveApp/Program.cs) del proyecto *MediaV3LiveApp*.

El ejemplo crea un sufijo único para cada recurso, de modo que no tengamos conflictos de nombres si ejecuta el ejemplo varias veces sin limpiar.

> [!IMPORTANT]
> Este ejemplo utiliza un sufijo único para cada recurso. Si cancela la depuración o termina la aplicación sin ejecutarla, acabará con varios objetos eventos en directo en la cuenta. <br/>
> Asegúrese de detener los objetos LiveEvent en ejecución. En caso contrario, se le **facturará** por ellos.

### <a name="start-using-media-services-apis-with-net-sdk"></a>Uso de las API de Media Services con SDK de .NET

Para empezar a usar las API de Media Services con. NET, debe crear un objeto **AzureMediaServicesClient**. Para crear el objeto, debe proporcionar las credenciales necesarias para que el cliente se conecte a Azure mediante Azure AD. En el código que se clonó al principio del artículo, la función **GetCredentialsAsync** crea el objeto ServiceClientCredentials basándose en las credenciales proporcionadas en el archivo de configuración local. 

[!code-csharp[Main](../../../media-services-v3-dotnet-core-tutorials/NETCore/Live/MediaV3LiveApp/Program.cs#CreateMediaServicesClient)]

### <a name="create-a-live-event"></a>Creación de un evento en directo

En esta sección se muestra cómo crear un tipo de objeto LiveEvent **paso a través** (el valor de LiveEventEncodingType es None). Para obtener más información sobre los tipos disponibles de objetos LiveEvent, consulte [Tipos de objetos LiveEvent](live-events-outputs-concept.md#live-event-types). 
 
Algunos de los aspectos que podría especificar al crear el evento en directo son:

* Ubicación de Media Services.
* El protocolo de streaming para el objeto LiveEvent (actualmente, se admiten los protocolos RTMP y Smooth Streaming).<br/>No puede cambiar la opción de protocolo mientras estén en ejecución el evento en directo o sus salidas activas asociadas. Si necesita diferentes protocolos, debe crear un evento en directo independiente para cada protocolo de streaming.  
* Restricciones de IP en la ingesta y vista previa. Puede definir las direcciones IP a las que se permite ingerir un vídeo en este objeto LiveEvent. Las direcciones IP permitidas se pueden especificar como una dirección IP única (por ejemplo, 10.0.0.1), un intervalo IP que usa una dirección IP y una máscara de subred CIDR (por ejemplo, 10.0.0.1/22) o un intervalo de IP que usa una máscara de subred decimal con puntos; por ejemplo, 10.0.0.1(255.255.252.0).<br/>Si no se especifica ninguna dirección IP y no hay ninguna definición de regla, no se permitirá ninguna dirección IP. Para permitir las direcciones IP, cree una regla y establezca 0.0.0.0/0.<br/>Las direcciones IP deben estar en uno de los siguientes formatos: dirección IpV4 con cuatro números o intervalo de direcciones CIDR.
* Al crear el evento, puede especificar que se inicie automáticamente. <br/>Cuando el inicio automático está establecido en true, el evento en directo se inicia después de la creación. Es decir, la facturación comienza en cuanto el evento en directo empieza a ejecutarse. Debe llamar explícitamente a Stop en el recurso del objeto LiveEvent para evitar que continúe la facturación. Para más información, consulte [Estados y facturación de LiveEvent](live-event-states-billing.md).
* Para que una dirección URL de ingesta sea predictiva, establezca el modo "mnemónica". Para información detallada, consulte [Direcciones URL de ingesta de objetos LiveEvent](live-events-outputs-concept.md#live-event-ingest-urls).

[!code-csharp[Main](../../../media-services-v3-dotnet-core-tutorials/NETCore/Live/MediaV3LiveApp/Program.cs#CreateLiveEvent)]

### <a name="get-ingest-urls"></a>Obtención de direcciones URL de introducción

Una vez que se crea el evento en directo, puede obtener las direcciones URL de ingesta que proporcionará al codificador en directo. El codificador usa estas direcciones URL para introducir una secuencia en vivo.

[!code-csharp[Main](../../../media-services-v3-dotnet-core-tutorials/NETCore/Live/MediaV3LiveApp/Program.cs#GetIngestURL)]

### <a name="get-the-preview-url"></a>Obtención de la URL de vista previa

Utilice el objeto previewEndpoint para obtener una vista previa y comprobar que la entrada del codificador se está recibiendo realmente.

> [!IMPORTANT]
> Asegúrese de que el vídeo fluye a la dirección URL de vista previa antes de continuar.

[!code-csharp[Main](../../../media-services-v3-dotnet-core-tutorials/NETCore/Live/MediaV3LiveApp/Program.cs#GetPreviewURLs)]

### <a name="create-and-manage-live-events-and-live-outputs"></a>Creación y administración de objetos LiveEvent y LiveOutput

Una vez que la secuencia fluye en el objeto LiveEvent, puede comenzar el evento de streaming mediante la creación de un recurso, un objeto LiveOutput y un objeto StreamingLocator. Se archivará la secuencia y estará disponible a los usuarios a través del extremo de streaming.

#### <a name="create-an-asset"></a>Creación de un recurso

Cree un recurso para que lo use el objeto LiveOutput.

[!code-csharp[Main](../../../media-services-v3-dotnet-core-tutorials/NETCore/Live/MediaV3LiveApp/Program.cs#CreateAsset)]

#### <a name="create-a-live-output"></a>Creación de un objeto LiveOutput

Los objetos LiveOutput comienzan al crearlos y se detienen cuando se eliminan. Cuando se elimina la salida en directo, no se eliminan el recurso subyacente ni el contenido de dicho recurso.

[!code-csharp[Main](../../../media-services-v3-dotnet-core-tutorials/NETCore/Live/MediaV3LiveApp/Program.cs#CreateLiveOutput)]

#### <a name="create-a-streaming-locator"></a>Creación de un objeto StreamingLocator

> [!NOTE]
> Cuando se crea la cuenta de Media Services, se agrega un punto de conexión de streaming **predeterminado** a la cuenta en estado **Detenido**. Para iniciar la transmisión del contenido y aprovechar el [empaquetado dinámico](dynamic-packaging-overview.md) y el cifrado dinámico, el punto de conexión de streaming desde el que va a transmitir el contenido debe estar en estado **Running** (En ejecución).

Cuando publique el recurso del objeto LiveOutput mediante un objeto StreamingLocator, el objeto LiveEvent (hasta la longitud de la ventana de DVR) seguirá estando visible hasta la expiración o eliminación del objeto StreamingLocator, lo que ocurra primero.

[!code-csharp[Main](../../../media-services-v3-dotnet-core-tutorials/NETCore/Live/MediaV3LiveApp/Program.cs#CreateStreamingLocator)]

```csharp

// Get the url to stream the output
ListPathsResponse paths = await client.StreamingLocators.ListPathsAsync(resourceGroupName, accountName, locatorName);

foreach (StreamingPath path in paths.StreamingPaths)
{
    UriBuilder uriBuilder = new UriBuilder();
    uriBuilder.Scheme = "https";
    uriBuilder.Host = streamingEndpoint.HostName;

    uriBuilder.Path = path.Paths[0];
    // Get the URL from the uriBuilder: uriBuilder.ToString()
}
```

### <a name="cleaning-up-resources-in-your-media-services-account"></a>Limpieza de los recursos en su cuenta de Media Services

Si has terminado de hacer streaming de los eventos y quiere limpiar los recursos aprovisionados anteriormente, siga el procedimiento siguiente:

* Detenga la inserción de la secuencia en el codificador.
* Detenga el objeto LiveEvent. Una vez que el evento en directo se detenga, dejará de suponer un coste. Cuando necesite iniciarlo de nuevo, tendrá la misma URL de introducción, por lo que no necesitará volver a configurar su codificador.
* Puede detener el extremo de streaming, a menos que desee seguir proporcionando el archivo de su evento en vivo como una secuencia a petición. Si el evento en directo se encuentra en estado detenido, dejará de suponer un coste.

[!code-csharp[Main](../../../media-services-v3-dotnet-core-tutorials/NETCore/Live/MediaV3LiveApp/Program.cs#CleanupLiveEventAndOutput)]

[!code-csharp[Main](../../../media-services-v3-dotnet-core-tutorials/NETCore/Live/MediaV3LiveApp/Program.cs#CleanupLocatorAssetAndStreamingEndpoint)]

## <a name="watch-the-event"></a>Visualización del evento

Para ver el evento, copie la dirección URL de streaming que obtuvo al ejecutar el código descrito en Creación de un localizador de streaming. Puede usar el reproductor multimedia que quiera. Puede usar [Azure Media Player](https://amp.azure.net/libs/amp/latest/docs/index.html) para probar la secuencia en https://ampdemo.azureedge.net.

El evento en directo convierte automáticamente los eventos en contenido a petición cuando se detiene. Incluso después de detener y eliminar el evento, los usuarios podrán hacer streaming del contenido archivado como un vídeo bajo demanda, siempre que no se elimine el recurso. No se puede eliminar un recurso si lo está usando un evento. Primero se debe eliminar el evento.

## <a name="clean-up-resources"></a>Limpieza de recursos

Si ya no necesita ninguno de los recursos del grupo de recursos, incluida las cuentas de almacenamiento y de Media Services que ha creado en este tutorial, elimine el grupo de recursos que ha creado antes.

Ejecute el siguiente comando de la CLI:

```azurecli-interactive
az group delete --name amsResourceGroup
```

> [!IMPORTANT]
> Si se deja el evento en directo en ejecución, este incurrirá en costos de facturación. Tenga en cuenta que si el proyecto o programa se bloquea o se cierra por cualquier motivo, es posible que el evento en directo se quede en ejecución en un estado de facturación.

## <a name="ask-questions-give-feedback-get-updates"></a>Formule preguntas, realice comentarios y obtenga actualizaciones

Consulte el artículo [Comunidad de Azure Media Services](media-services-community.md) para ver diferentes formas de formular preguntas, enviar comentarios y obtener actualizaciones de Media Services.

## <a name="next-steps"></a>Pasos siguientes

[Streaming de archivos](stream-files-tutorial-with-api.md)
 
