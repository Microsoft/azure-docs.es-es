---
title: Uso del cifrado dinámico AES-128 y el servicio de entrega de claves | Microsoft Docs
description: En este tema se muestra cómo cifrar dinámicamente con AES-128 y usar el servicio de entrega de claves.
services: media-services
documentationcenter: ''
author: Juliako
manager: femila
editor: ''
ms.assetid: 4d2c10af-9ee0-408f-899b-33fa4c1d89b9
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 04/01/2019
ms.author: juliako
ms.openlocfilehash: 143583cf11da7687ae99121a2893be1c54810add
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87087376"
---
# <a name="use-aes-128-dynamic-encryption-and-the-key-delivery-service"></a>Uso del cifrado dinámico AES-128 y el servicio de entrega de claves
> [!div class="op_single_selector"]
> * [.NET](media-services-protect-with-aes128.md)
> * [Java](https://github.com/southworkscom/azure-sdk-for-media-services-java-samples)
> * [PHP](https://github.com/Azure/azure-sdk-for-php/tree/master/examples/MediaServices)
>  

> [!NOTE]
> No hay características o funcionalidades nuevas para agregar a Media Services, versión 2. <br/>Finalice la compra de la versión más reciente, [Media Services v3](../latest/index.yml). Consulte también la [guía de migración de la versión v2 a v3](../latest/migrate-from-v2-to-v3.md).

Puede usar Media Services para entregar HTTP Live Streaming (HLS) y Smooth Streaming cifrados con AES mediante las claves de cifrado de 128 bits. Media Services también proporciona el servicio de entrega de claves que distribuye claves de cifrado a los usuarios autorizados. Si desea que Media Services cifre un recurso, debe asociar una clave de cifrado con el recurso y, además, configurar directivas de autorización para la clave. Cuando un reproductor solicita una secuencia, Media Services usa la clave especificada para cifrar de forma dinámica el contenido mediante el cifrado AES. Para descifrar la secuencia, el reproductor solicitará la clave del servicio de entrega de claves. Para determinar si el usuario tiene permiso para obtener la clave, el servicio evalúa las directivas de autorización que especificó para la clave.

Media Services admite varias formas de autenticar a los usuarios que realizan solicitudes de clave. La directiva de autorización de claves de acceso puede tener una o más restricciones de autorización, ya sea restricciones de apertura o de token. La directiva con restricción de token debe ir acompañada de un token emitido por un servicio de token de seguridad (STS). Media Services admite tokens en los formatos de [token de web simple](/previous-versions/azure/azure-services/gg185950(v=azure.100)#BKMK_2) (SWT) y de [token de web JSON](/previous-versions/azure/azure-services/gg185950(v=azure.100)#BKMK_3) (JWT). Para más información, consulte [Configuración de la directiva de autorización de claves de contenido](media-services-protect-with-aes128.md#configure_key_auth_policy).

Para aprovechar las ventajas del cifrado dinámico, debe disponer de un recurso que contenga un conjunto de archivos MP4 o archivos de origen Smooth Streaming, de varias velocidades de bits. También debe configurar la directiva de entrega para el recurso (se describe más adelante en este artículo). Luego, según el formato especificado en la URL de streaming, el servidor de streaming a petición se asegura de que se reciba la secuencia en el protocolo seleccionado. Como resultado, necesita almacenar y pagar solo los archivos que estén en un formato de almacenamiento único. Media Services crea y publica la respuesta adecuada en función de las solicitudes de un cliente.

Este artículo será útil para los desarrolladores que trabajan en aplicaciones que proporcionan contenido multimedia protegido. Se describe cómo configurar el servicio de entrega de claves con directivas de autorización para que solo los clientes autorizados puedan recibir las claves de cifrado. También se describe cómo usar el cifrado dinámico.

Para información sobre cómo cifrar contenido con el Estándar de cifrado avanzado (AES) para su entrega en Safari de macOS, consulte [esta entrada de blog](https://azure.microsoft.com/blog/how-to-make-token-authorized-aes-encrypted-hls-stream-working-in-safari/).
Para información general sobre cómo proteger el contenido multimedia con el cifrado AES, vea [este vídeo](https://channel9.msdn.com/Shows/Azure-Friday/Azure-Media-Services-Protecting-your-Media-Content-with-AES-Encryption).


## <a name="aes-128-dynamic-encryption-and-key-delivery-service-workflow"></a>Flujo de trabajo de cifrado dinámico AES-128 y del servicio de entrega de claves

Realice estos pasos generales cuando cifre los recursos con AES, mediante el servicio de entrega de claves de Media Services y también mediante cifrado dinámico:

1. [Cree un recurso y cargue los archivos en él](media-services-protect-with-aes128.md#create_asset).

2. [Codifique el recurso que contiene el archivo para el conjunto de MP4 de velocidad de bits adaptable](media-services-protect-with-aes128.md#encode_asset).

3. [Cree una clave de contenido y asóciela con el recurso codificado](media-services-protect-with-aes128.md#create_contentkey). En Media Services, la clave de contenido contiene la clave de cifrado del recurso.

4. [Configura la directiva de autorización de la clave de contenido](media-services-protect-with-aes128.md#configure_key_auth_policy). Debe configurar la directiva de autorización de claves de contenido. El cliente debe cumplir la directiva antes de que la clave de contenido se entregue al cliente.

5. [Configuración de la directiva de entrega para un recurso](media-services-protect-with-aes128.md#configure_asset_delivery_policy). La configuración de directiva de entrega incluye la dirección URL de adquisición de claves y un vector de inicialización (IV). (AES-128 requiere el mismo vector de inicialización para cifrado y descifrado). La configuración también incluye el protocolo de entrega (por ejemplo, MPEG-DASH, HSL, Smooth Streaming o todos) y el tipo de cifrado dinámico (por ejemplo, cifrado Envelope o cifrado no dinámico).

    Puede aplicar una directiva diferente a cada protocolo en el mismo recurso. Por ejemplo, puede aplicar cifrado PlayReady a Smooth/DASH y un sobre AES a HLS. Los protocolos que no están definidos en una directiva de entrega se bloquean del streaming. (Por ejemplo, si agrega una directiva única que especifica que solo HLS es el protocolo). La excepción se produce en el caso de que no haya definido ninguna directiva de entrega de recursos. En tal caso, todos los protocolos están habilitados sin cifrar.

6. [Cree un localizador OnDemand](media-services-protect-with-aes128.md#create_locator) para obtener una dirección URL de streaming.

El artículo también muestra [cómo una aplicación cliente puede solicitar una clave al servicio de entrega de claves](media-services-protect-with-aes128.md#client_request).

Encontrará un [ejemplo completo para .NET](media-services-protect-with-aes128.md#example) al final del artículo.

La siguiente imagen muestra el flujo de trabajo que se ha descrito anteriormente. Aquí el token se usa para la autenticación.

![Protección con AES-128](./media/media-services-content-protection-overview/media-services-content-protection-with-aes.png)

El resto de este artículo proporciona explicaciones, ejemplos de código y vínculos a temas que le muestran cómo lograr las tareas descritas anteriormente.

## <a name="current-limitations"></a>Limitaciones actuales
Si agrega o actualiza la directiva de entrega de recursos, debe eliminar un localizador existente y crear uno nuevo.

## <a name="create-an-asset-and-upload-files-into-the-asset"></a><a id="create_asset"></a>Creación de un recurso y carga de los archivos en el recurso
Para administrar, codificar y transmitir vídeos, primero debe cargar el contenido en Media Services. Una vez cargado, el contenido se almacena de forma segura en la nube para su posterior procesamiento y streaming. 

Para más información, consulte [Carga de archivos en una cuenta de Media Services](media-services-dotnet-upload-files.md).

## <a name="encode-the-asset-that-contains-the-file-to-the-adaptive-bitrate-mp4-set"></a><a id="encode_asset"></a>Codificación del recurso que contiene el archivo para el conjunto de MP4 de velocidad de bits adaptable
Con el cifrado dinámico, crea un recurso que contiene un conjunto de archivos MP4 o archivos de origen Smooth Streaming, de múltiples velocidades de bits. Luego, según el formato especificado en la solicitud de manifiesto o fragmento, el servidor de streaming a petición se asegura de que reciba la secuencia en el protocolo que se ha seleccionado. Después, solo debe almacenar y pagar los archivos en un solo formato de almacenamiento. Media Services crea y publica la respuesta adecuada en función de las solicitudes de un cliente. Para más información, consulte [Empaquetado dinámico](media-services-dynamic-packaging-overview.md).

>[!NOTE]
>Cuando se crea la cuenta de Media Services, se agrega un punto de conexión de streaming predeterminado a la cuenta en estado Detenido. Para iniciar la transmisión del contenido y aprovechar el empaquetado dinámico y el cifrado dinámico, el punto de conexión de streaming desde el que va a transmitir el contenido debe estar en estado En ejecución. 
>
>Además, para usar el empaquetado dinámico y el cifrado dinámico, el recurso debe contener un conjunto de archivos MP4 de velocidad de bits adaptable o archivos Smooth Streaming de velocidad de bits adaptable.

Para obtener instrucciones acerca de cómo codificar, consulte [Codificación de un recurso mediante Media Encoder Standard](media-services-dotnet-encode-with-media-encoder-standard.md).

## <a name="create-a-content-key-and-associate-it-with-the-encoded-asset"></a><a id="create_contentkey"></a>Creación de una clave de contenido y su asociación con el activo codificado
En Media Services, la clave de contenido contiene la clave con la que desea cifrar un recurso.

Para más información, consulte [Creación de una clave de contenido](media-services-dotnet-create-contentkey.md).

## <a name="configure-the-content-keys-authorization-policy"></a><a id="configure_key_auth_policy"></a>Configuración de la directiva de autorización de claves de contenido
Media Services admite varias formas de autenticar a los usuarios que realizan solicitudes de clave. Debe configurar la directiva de autorización de claves de contenido. El cliente (reproductor) debe cumplir la directiva antes de que se pueda entregar la clave al cliente. La directiva de autorización de clave de acceso puede tener una o más restricciones de autorización, ya sea abrir, restricción de token o restricción de IP.

Para más información, consulte [Configuración de la directiva de autorización de claves de contenido](media-services-dotnet-configure-content-key-auth-policy.md).

## <a name="configure-an-asset-delivery-policy"></a><a id="configure_asset_delivery_policy"></a>Configuración de una directiva de entrega de recursos
Configure la directiva de entrega de sus recursos. Algunos de los elementos que incluye la configuración de la directiva de entrega de recursos son los siguientes:

* La dirección URL de adquisición de clave. 
* El vector de inicialización (IV) que se usará para el cifrado Envelope. AES-128 requiere el mismo vector de inicialización para cifrado y descifrado. 
* El protocolo de entrega de recursos (por ejemplo, MPEG DASH, HLS, Smooth Streaming o todos).
* El tipo de cifrado dinámico (por ejemplo, AES Envelope) o sin cifrado dinámico. 

Para más información, consulte [Configuración de una directiva de entrega de recursos](media-services-dotnet-configure-asset-delivery-policy.md).

## <a name="create-an-ondemand-streaming-locator-to-get-a-streaming-url"></a><a id="create_locator"></a>Creación de un localizador de streaming a petición para obtener una URL de streaming
Debe suministrar al usuario la URL de streaming para Smooth Streaming, DASH o HLS.

> [!NOTE]
> Si agrega o actualiza la directiva de entrega de recursos, debe eliminar un localizador existente y crear uno nuevo.
> 
> 

Para obtener instrucciones sobre cómo publicar un recurso y generar una dirección URL de streaming, vea [Creación de una dirección URL de streaming](media-services-deliver-streaming-content.md).

## <a name="get-a-test-token"></a>Obtención de un token de prueba
Obtenga un token de prueba basado en la restricción de token que se usó para la directiva de autorización de claves.

```csharp
    // Deserializes a string containing an Xml representation of a TokenRestrictionTemplate
    // back into a TokenRestrictionTemplate class instance.
    TokenRestrictionTemplate tokenTemplate = 
        TokenRestrictionTemplateSerializer.Deserialize(tokenTemplateString);

    // Generate a test token based on the data in the given TokenRestrictionTemplate.
    //The GenerateTestToken method returns the token without the word "Bearer" in front
    //so you have to add it in front of the token string. 
    string testToken = TokenRestrictionTemplateSerializer.GenerateTestToken(tokenTemplate);
    Console.WriteLine("The authorization token is:\nBearer {0}", testToken);
```

Puede usar [Azure Media Services Player](https://aka.ms/azuremediaplayer) para probar la secuencia.

## <a name="how-can-your-client-request-a-key-from-the-key-delivery-service"></a><a id="client_request"></a>Cómo el cliente puede solicitar una clave al servicio de entrega de claves
En el paso anterior, creó la URL que apunta a un archivo de manifiesto. El cliente debe extraer la información necesaria de los archivos de manifiesto de streaming para realizar una solicitud al servicio de entrega de claves.

### <a name="manifest-files"></a>Archivos de manifiesto
El cliente debe extraer el valor de la URL (que también contiene el identificador de la clave de contenido) del archivo de manifiesto. Luego, el cliente intenta obtener la clave de cifrado del servicio de entrega de claves. El cliente también debe extraer el valor IV y usarlo para descifrar la secuencia. En el siguiente fragmento se muestra el elemento `<Protection>` del manifiesto de Smooth Streaming:

```xml
    <Protection>
      <ProtectionHeader SystemID="B47B251A-2409-4B42-958E-08DBAE7B4EE9">
        <ContentProtection xmlns:sea="urn:mpeg:dash:schema:sea:2012" schemeIdUri="urn:mpeg:dash:sea:2012">
          <sea:SegmentEncryption schemeIdUri="urn:mpeg:dash:sea:aes128-cbc:2013"/>
          <sea:KeySystem keySystemUri="urn:mpeg:dash:sea:keysys:http:2013"/>
          <sea:CryptoPeriod IV="0xD7D7D7D7D7D7D7D7D7D7D7D7D7D7D7D7" 
                            keyUriTemplate="https://wamsbayclus001kd-hs.cloudapp.net/HlsHandler.ashx?
                                            kid=da3813af-55e6-48e7-aa9f-a4d6031f7b4d"/>
        </ContentProtection>
      </ProtectionHeader>
    </Protection>
```

En el caso de HLS, el manifiesto raíz se divide en archivos de segmento. 

Por ejemplo, el manifiesto raíz es: http:\//test001.origin.mediaservices.windows.net/8bfe7d6f-34e3-4d1a-b289-3e48a8762490/BigBuckBunny.ism/manifest(format=m3u8-aapl). que contiene una lista de nombres de archivo de segmento.

```text
. . . 
#EXT-X-STREAM-INF:PROGRAM-ID=1,BANDWIDTH=630133,RESOLUTION=424x240,CODECS="avc1.4d4015,mp4a.40.2",AUDIO="audio"
QualityLevels(514369)/Manifest(video,format=m3u8-aapl)
#EXT-X-STREAM-INF:PROGRAM-ID=1,BANDWIDTH=965441,RESOLUTION=636x356,CODECS="avc1.4d401e,mp4a.40.2",AUDIO="audio"
QualityLevels(842459)/Manifest(video,format=m3u8-aapl)
…
```

Si abre uno de los archivos de segmento en un editor de texto (por ejemplo, http:\//test001.origin.mediaservices.windows.net/8bfe7d6f-34e3-4d1a-b289-3e48a8762490/BigBuckBunny.ism/QualityLevels(514369)/Manifest(video,format=m3u8-aapl), contiene #EXT-X-KEY, que indica que el archivo está cifrado.

```text
#EXTM3U
#EXT-X-VERSION:4
#EXT-X-ALLOW-CACHE:NO
#EXT-X-MEDIA-SEQUENCE:0
#EXT-X-TARGETDURATION:9
#EXT-X-KEY:METHOD=AES-128,
URI="https://wamsbayclus001kd-hs.cloudapp.net/HlsHandler.ashx?
        kid=da3813af-55e6-48e7-aa9f-a4d6031f7b4d",
        IV=0XD7D7D7D7D7D7D7D7D7D7D7D7D7D7D7D7
#EXT-X-PROGRAM-DATE-TIME:1970-01-01T00:00:00.000+00:00
#EXTINF:8.425708,no-desc
Fragments(video=0,format=m3u8-aapl)
#EXT-X-ENDLIST
```

>[!NOTE] 
>Si planea la reproducción de una instancia de HLS cifrada mediante AES en Safari, consulte [este blog](https://azure.microsoft.com/blog/how-to-make-token-authorized-aes-encrypted-hls-stream-working-in-safari/).

### <a name="request-the-key-from-the-key-delivery-service"></a>Solicitud de la clave al servicio de entrega de claves

El código siguiente muestra cómo enviar una solicitud al servicio de entrega de claves de Media Services mediante un URI de entrega de claves (que se extrajo del manifiesto) y un token. (En este artículo no se explica cómo obtener SWT a partir de un STS).

```csharp
    private byte[] GetDeliveryKey(Uri keyDeliveryUri, string token)
    {
        HttpWebRequest request = (HttpWebRequest)WebRequest.Create(keyDeliveryUri);

        request.Method = "POST";
        request.ContentType = "text/xml";
        if (!string.IsNullOrEmpty(token))
        {
            request.Headers[AuthorizationHeader] = token;
        }
        request.ContentLength = 0;

        var response = request.GetResponse();

        var stream = response.GetResponseStream();
        if (stream == null)
        {
            throw new NullReferenceException("Response stream is null");
        }

        var buffer = new byte[256];
        var length = 0;
        while (stream.CanRead && length <= buffer.Length)
        {
            var nexByte = stream.ReadByte();
            if (nexByte == -1)
            {
                break;
            }
            buffer[length] = (byte)nexByte;
            length++;
        }
        response.Close();

        // AES keys must be exactly 16 bytes (128 bits).
        var key = new byte[length];
        Array.Copy(buffer, key, length);
        return key;
    }
```

## <a name="protect-your-content-with-aes-128-by-using-net"></a>Protección del contenido con AES-128 mediante .NET

### <a name="create-and-configure-a-visual-studio-project"></a>Creación y configuración de un proyecto de Visual Studio

1. Configure el entorno de desarrollo y rellene el archivo app.config con la información de la conexión, como se describe en [Desarrollo de Media Services con .NET](media-services-dotnet-how-to-use.md).

2. Agregue los siguientes elementos a appSettings, tal como se define en el archivo app.config:

    ```xml
    <add key="Issuer" value="http://testissuer.com"/>
    <add key="Audience" value="urn:test"/>
    ```

### <a name="example"></a><a id="example"></a>Ejemplo

Sobrescriba el código del archivo Program.cs con el código mostrado en esta sección.
 
>[!NOTE]
>Hay un límite de 1 000 000 directivas para diferentes directivas de Media Services (por ejemplo, para la directiva de localizador o ContentKeyAuthorizationPolicy). Use el mismo identificador de directiva si siempre usa los mismos permisos de acceso/días. Un ejemplo de ello son las directivas para localizadores que van a permanecer durante mucho tiempo (directivas no de carga). Para más información, consulte la sección "Directivas de limitación del acceso" en el artículo sobre cómo [administrar los recursos y entidades relacionadas con el SDK de .NET de Media Services](media-services-dotnet-manage-entities.md#limit-access-policies).

Asegúrese de actualizar las variables para que apunten a las carpetas donde se encuentran los archivos de entrada.

[!code-csharp[Main](../../../samples-mediaservices-encryptionaes/DynamicEncryptionWithAES/DynamicEncryptionWithAES/Program.cs)]

## <a name="media-services-learning-paths"></a>Rutas de aprendizaje de Media Services
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Envío de comentarios
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]
