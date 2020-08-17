---
title: Cifrado de vídeo con AES-128
titleSuffix: Azure Media Services
description: Obtenga información acerca de cómo cifrar vídeo con el cifrado AES de 128 bits y cómo usar el servicio de entrega de claves en Azure Media Services.
services: media-services
documentationcenter: ''
author: Juliako
manager: femila
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 07/09/2020
ms.author: juliako
ms.openlocfilehash: 45bb8637d37c9c3789a962c9f5ac42227d547637
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87022827"
---
# <a name="tutorial-encrypt-video-with-aes-128-and-use-the-key-delivery-service"></a>Tutorial: Cifrado de vídeo con AES-128 y uso del servicio de entrega de claves

> [!NOTE]
> Aunque en este tutorial se usan los ejemplos de [SDK de .NET](/dotnet/api/microsoft.azure.management.media.models.liveevent?view=azure-dotnet), los pasos generales son los mismos para la [API REST](/rest/api/media/liveevents), la [CLI](/cli/azure/ams/live-event?view=azure-cli-latest) u otros [SDK](media-services-apis-overview.md#sdks) admitidos.

Puede usar Media Services para entregar HTTP Live Streaming (HLS), MPEG-DASH y Smooth Streaming cifrados con AES mediante las claves de cifrado de 128 bits. Media Services también proporciona el servicio de entrega de claves que distribuye claves de cifrado a los usuarios autorizados. Si quiere que Media Services cifre un vídeo de forma dinámica, debe asociar una clave de cifrado con un localizador de streaming y, además, configurar la directiva de clave de contenido. Cuando un reproductor solicita una secuencia, Media Services usa la clave especificada para cifrar de forma dinámica el contenido con AES-128. Para descifrar la secuencia, el reproductor solicitará la clave del servicio de entrega de claves. Para determinar si el usuario tiene permiso para obtener la clave, el servicio evalúa la directiva de clave de contenido que especificó para la clave.

Puede cifrar cada recurso con varios tipos de cifrado (AES-128, PlayReady, Widevine, FairPlay). Consulte [Streaming protocols and encryption types](content-protection-overview.md#streaming-protocols-and-encryption-types) (Protocolos de streaming y tipos de cifrado) para ver lo que conviene combinar. Asimismo, consulte cómo [proteger con DRM](protect-with-drm.md).

La salida del ejemplo de este artículo incluye una dirección URL para Azure Media Player, incluidos el manifiesto URL y el token de AES necesarios para reproducir el contenido. En el ejemplo se establece la expiración del token de JSON Web Token (JWT) en 1 hora. Puede abrir un explorador y pegar la dirección URL resultante para iniciar la página de demostración de Azure Media Player en la que se rellenan automáticamente la dirección URL y el token (con el siguiente formato: ```https://ampdemo.azureedge.net/?url= {dash Manifest URL} &aes=true&aestoken=Bearer%3D{ JWT Token here}```).

En este tutorial se muestra cómo realizar las siguientes acciones:

> [!div class="checklist"]
> * Descargue el ejemplo de [EncryptWithAES ](https://github.com/Azure-Samples/media-services-v3-dotnet-tutorials/blob/master/AMSV3Tutorials/EncryptWithAES) descrito en el artículo.
> * Empiece a usar las API de Media Services con el SDK de .NET.
> * Cree un recurso de salida.
> * Cree una transformación de codificación.
> * Envíe un trabajo.
> * Espere a que el trabajo se complete.
> * Crear una directiva de clave de contenido
> * Configure la directiva para usar la restricción del token JWT.
> * Cree un localizador de streaming.
> * Configure el localizador de streaming para cifrar el vídeo con AES (ClearKey).
> * Obtenga un token de prueba.
> * Cree una dirección URL de streaming.
> * Limpieza de recursos.

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Requisitos previos

Los siguientes requisitos son necesarios para completar el tutorial.

* Revise el artículo [Content protection overview](content-protection-overview.md) (Información general de la protección de contenido).
* Instalación de Visual Studio Code o Visual Studio.
* [Cree una cuenta de Media Services](./create-account-howto.md).
* Obtenga las credenciales necesarias para usar las API de Media Services siguiendo las instrucciones de [Acceso a las API](./access-api-howto.md).

## <a name="download-code"></a>Descarga de código

Clone en la máquina un repositorio GitHub que contenga el ejemplo de .NET completo que se explica en este artículo, con el siguiente comando:

 ```bash
 git clone https://github.com/Azure-Samples/media-services-v3-dotnet-tutorials.git
 ```

El ejemplo "Encrypt with AES-128" se encuentra en la carpeta [EncryptWithAES](https://github.com/Azure-Samples/media-services-v3-dotnet-tutorials/blob/master/AMSV3Tutorials/EncryptWithAES).

> [!NOTE]
> En el ejemplo se crean recursos únicos cada vez que se ejecuta la aplicación. Normalmente, se volverán a usar los recursos existentes, como las transformaciones y las directivas (si los recursos existentes tienen las configuraciones necesarias).

## <a name="start-using-media-services-apis-with-net-sdk"></a>Uso de las API de Media Services con SDK de .NET

Para empezar a usar las API de Media Services con .NET, cree un objeto **AzureMediaServicesClient**. Para crear el objeto, debe proporcionar las credenciales necesarias para que el cliente se conecte a Azure mediante Azure AD. En el código que se clonó al principio del artículo, la función **GetCredentialsAsync** crea el objeto ServiceClientCredentials basándose en las credenciales proporcionadas en el archivo de configuración local.

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/EncryptWithAES/Program.cs#CreateMediaServicesClient)]

## <a name="create-an-output-asset"></a>Creación de un recurso de salida  

El [recurso](/rest/api/media/assets) de salida almacena el resultado del trabajo de codificación.  

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/EncryptWithAES/Program.cs#CreateOutputAsset)]

## <a name="get-or-create-an-encoding-transform"></a>Obtención o creación de una transformación de codificación

Al crear una nueva instancia de la [transformación](/rest/api/media/transforms), debe especificar qué desea originar como salida. El parámetro requerido es un objeto **TransformOutput**, como se muestra en el código siguiente. Cada objeto **TransformOutput** contiene un **valor preestablecido**. El **valor preestablecido** describe las instrucciones paso a paso de las operaciones de procesamiento de vídeo o audio que se utilizarán para generar el objeto **TransformOutput** deseado. El ejemplo descrito en este artículo utiliza un valor preestablecido integrado denominado **AdaptiveStreaming**. El valor preestablecido codifica el vídeo de entrada en una escala de velocidad de bits generada automáticamente (pares velocidad de bits-resolución) basada en la resolución y la velocidad de bits y, a continuación, produce archivos ISO MP4 con vídeo H.264 y audio AAC correspondiente a cada par velocidad de bits-resolución.

Antes de crear una nueva [transformación](/rest/api/media/transforms), compruebe primero si ya existe una con el método **Get**, tal como se muestra en el código siguiente. En Media Services v3, los métodos **Get** en las entidades devuelven **null** si la entidad no existe (una comprobación sin distinción entre mayúsculas y minúsculas en el nombre).

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/EncryptWithAES/Program.cs#EnsureTransformExists)]

## <a name="submit-job"></a>Enviar el trabajo

Como se mencionó anteriormente, el objeto [Transform](/rest/api/media/transforms) es la receta y un [trabajo](/rest/api/media/jobs) es la solicitud real a Media Services para aplicar que dicho objeto **Transform** a un determinado contenido de audio o vídeo de entrada. El **trabajo** especifica información como la ubicación del vídeo de entrada y la ubicación de la salida.

En este tutorial, se crea la entrada del trabajo basada en un archivo que se ingiere directamente desde una [dirección URL de origen HTTPS](job-input-from-http-how-to.md).

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/EncryptWithAES/Program.cs#SubmitJob)]

## <a name="wait-for-the-job-to-complete"></a>Espere a que el trabajo se complete.

El trabajo tarda un tiempo en completarse. Cuando lo hace, desea recibir una notificación. En el ejemplo de código siguiente se muestra cómo sondear el servicio para conocer el estado del [trabajo](/rest/api/media/jobs). El sondeo no es un procedimiento recomendado para aplicaciones de producción debido a la posible latencia. El sondeo se puede limitar si se sobreutiliza en una cuenta. Los desarrolladores deben utilizar en su lugar Event Grid. Para más información, consulte [Enrutamiento de eventos a un punto de conexión web personalizado](job-state-events-cli-how-to.md).

El **trabajo** pasa normalmente por los siguientes estados: **Programado**, **En cola**, **Procesando**, **Finalizado** (el estado final). Si el trabajo ha encontrado un error, obtendrá el estado **Error**. Si el trabajo está en proceso de cancelación, obtendrá los mensajes **Cancelando** y **Cancelado** cuando haya terminado.

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/EncryptWithAES/Program.cs#WaitForJobToFinish)]

## <a name="create-a-content-key-policy"></a>Crear una directiva de clave de contenido

Una clave de contenido proporciona acceso seguro a los recursos. Debe crear una **directiva de clave de contenido** que configure cómo se entrega la clave de contenido a los clientes finales. La clave de contenido está asociada con el **localizador de streaming**. Media Services también proporciona el servicio de entrega de claves que distribuye claves de cifrado a los usuarios autorizados.

Cuando un reproductor solicita una secuencia, Media Services usa la clave especificada para cifrar de forma dinámica el contenido (en este caso, mediante el cifrado AES). Para descifrar la secuencia, el reproductor solicitará la clave del servicio de entrega de claves. Para determinar si el usuario tiene permiso para obtener la clave, el servicio evalúa la directiva de clave de contenido que especificó para la clave.

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/EncryptWithAES/Program.cs#GetOrCreateContentKeyPolicy)]

## <a name="create-a-streaming-locator"></a>Creación de un objeto StreamingLocator

Una vez finalizada la codificación y establecida la directiva de clave de contenido, el siguiente paso es poner el vídeo del recurso de salida a disposición de los clientes para su reproducción. El vídeo se pone a disposición de dos pasos:

1. Cree un [localizador de streaming](/rest/api/media/streaminglocators).
2. Compile direcciones URL de streaming que los clientes puedan usar.

El proceso de creación de un objeto **Localizador de streaming** se denomina publicación. De forma predeterminada, el **localizador de streaming** es válido inmediatamente después de realizar las llamadas API. Se mantiene hasta que se elimina, a menos que configure las horas de inicio y finalización opcionales.

Al crear un [localizador de streaming](/rest/api/media/streaminglocators), deberá especificar el objeto **StreamingPolicyName** deseado. En este tutorial, se usa una de las estructuras PredefinedStreamingPolicies, que indica a Azure Media Services cómo publicar el contenido de streaming. En este ejemplo, se aplica el cifrado AES Envelope (este cifrado también se conoce como cifrado ClearKey, dado que la clave se envía al cliente de reproducción a través de HTTPS y no una licencia DRM).

> [!IMPORTANT]
> Al utilizar el objeto [StreamingPolicy](/rest/api/media/streamingpolicies) personalizado, debe diseñar un conjunto limitado de dichas directivas para su cuenta de Media Service y reutilizarlas para sus objetos StreamingLocator siempre que se necesiten las mismas opciones y protocolos de cifrado. La cuenta de Media Service tiene una cuota para el número de entradas de StreamingPolicy. No debe crear un nuevo objeto StreamingPolicy para cada localizador de streaming.

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/EncryptWithAES/Program.cs#CreateStreamingLocator)]

## <a name="get-a-test-token"></a>Obtención de un token de prueba

En este tutorial, se especifica que la directiva de clave de contenido tiene una restricción de token. La directiva con restricción de token debe ir acompañada de un token emitido por un servicio de token de seguridad (STS). Media Services admite tokens en el formato [JWT](/previous-versions/azure/azure-services/gg185950(v=azure.100)#BKMK_3), y es lo que se configura en este ejemplo.

El objeto ContentKeyIdentifierClaim se usa en la **directiva de clave de contenido**, lo que significa que el token que se presenta al servicio de entrega de claves debe contener el identificador de la clave de contenido. En el ejemplo, no se especifica una clave de contenido al crear el Localizador de streaming, así el sistema crea automáticamente una aleatoria. Para generar la prueba de token, se debe obtener el elemento ContentKeyId para colocarlo en la notificación ContentKeyIdentifierClaim.

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/EncryptWithAES/Program.cs#GetToken)]

## <a name="build-a-dash-streaming-url"></a>Generación de una dirección URL de streaming de DASH

Ahora que se ha creado el [Localizador de streaming](/rest/api/media/streaminglocators), puede obtener las direcciones URL de streaming. Para crear una dirección URL, debe concatenar el nombre de host de [StreamingEndpoint](/rest/api/media/streamingendpoints) y la ruta de acceso del **Localizador de streaming**. En este ejemplo, se usa el *punto de conexión de* **streaming predeterminado**. Cuando cree su primera cuenta de Media Services, este *punto de conexión de streaming* **predeterminado** estará en un estado detenido, por lo que deberá llamar a **Iniciar**.

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/EncryptWithAES/Program.cs#GetMPEGStreamingUrl)]

## <a name="clean-up-resources-in-your-media-services-account"></a>Limpieza de los recursos en su cuenta de Media Services

Por lo general, debe limpiar todo, excepto los objetos que piensa reutilizar (normalmente, reutilizará transformaciones, localizadores de streaming, etc.). Si quiere que la cuenta esté limpia después de la experimentación, elimine los recursos que no tiene previsto reutilizar. Por ejemplo, el siguiente código elimina el trabajo, los recursos creados y la directiva de clave de contenido:

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/EncryptWithAES/Program.cs#CleanUp)]

## <a name="clean-up-resources"></a>Limpieza de recursos

Si ya no necesita ninguno de los recursos del grupo de recursos, incluida las cuentas de almacenamiento y de Media Services que ha creado en este tutorial, elimine el grupo de recursos que ha creado antes.

Ejecute el siguiente comando de la CLI:

```azurecli
az group delete --name amsResourceGroup
```

## <a name="ask-questions-give-feedback-get-updates"></a>Formule preguntas, realice comentarios y obtenga actualizaciones

Consulte el artículo [Comunidad de Azure Media Services](media-services-community.md) para ver diferentes formas de formular preguntas, enviar comentarios y obtener actualizaciones de Media Services.

## <a name="additional-notes"></a>Notas adicionales

* Widevine es un servicio que ofrece Google Inc. y que está sujeto a los términos del servicio y la directiva de privacidad de Google, Inc.

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Protección con DRM](protect-with-drm.md)
