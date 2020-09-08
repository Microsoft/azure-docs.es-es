---
title: Protección del contenido mediante el cifrado dinámico de Media Services v3
titleSuffix: Azure Media Services
description: Obtenga información sobre la protección de contenido con cifrado dinámico, protocolos de streaming y tipos de cifrado en Azure Media Services.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 08/31/2020
ms.author: inhenkel
ms.custom: seodec18, devx-track-csharp
ms.openlocfilehash: d0f040961bfb72082f8c5accb86999d489a93de5
ms.sourcegitcommit: 5a3b9f35d47355d026ee39d398c614ca4dae51c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89401391"
---
# <a name="protect-your-content-with-media-services-dynamic-encryption"></a>Protección del contenido mediante el cifrado dinámico de Media Services

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

Use Azure Media Services para proteger su contenido multimedia desde el momento en que este deja el equipo hasta el almacenamiento, el procesamiento y la entrega. Con Media Services puede entregar el contenido cifrado de forma dinámica en vivo y a petición con el Estándar de cifrado avanzado (AES-128) o cualquiera de los tres sistemas de administración de derechos digitales (DRM) principales: Microsoft PlayReady, Google Widevine y Apple FairPlay. Media Services también proporciona un servicio para entregar claves AES y licencias de DMR (PlayReady, Widevine y FairPlay) a los clientes autorizados. Si el contenido está cifrado con una clave sin cifrado AES y se envía a través de HTTPS, no será claro hasta que llegue al cliente. 

En Media Services v3, una clave de contenido se asocia con un localizador de streaming (consulte [este ejemplo](protect-with-aes128.md)). Si usa el servicio de entrega de claves de Media Services, puede permitir que Azure Media Services genere la clave de contenido automáticamente. La clave de contenido la debería generar usted mismo si usa su propio servicio de claves o si tiene que controlar un escenario de alta disponibilidad en el que necesita tener la misma clave de contenido en dos centros de datos.

Cuando un reproductor solicita una transmisión, Media Services usa la clave especificada para cifrar de forma dinámica el contenido mediante la clave sin cifrado de AES o el cifrado DRM. Para descifrar la secuencia, el reproductor solicitará la clave del servicio de entrega de claves de Media Services o la del servicio que especifique. Para determinar si el usuario tiene autorización para obtener la clave, el servicio evalúa la directiva de claves de contenido que especificó para la clave.

Puede usar la API REST o una biblioteca cliente de Media Services para configurar las directivas de autorización y autenticación de sus licencias y claves.

En la siguiente imagen se ilustra el flujo de trabajo de protección de contenido de Media Services:

![Flujo de trabajo para la protección de contenido de Media Services](./media/content-protection/content-protection.svg)
  
&#42; *El cifrado dinámico es compatible con la "clave sin cifrado" de AES-128, CBCS y CENC. Para información detallada, consulte [la matriz de compatibilidad](#streaming-protocols-and-encryption-types).*

En este artículo se explican los conceptos y la terminología para ayudar a conocer la protección de contenido con Media Services.

## <a name="main-components-of-a-content-protection-system"></a>Componentes principales del sistema de protección de contenido

Para completar correctamente el sistema de protección de contenido, debe entender completamente el ámbito del esfuerzo. En las secciones siguientes se ofrece información general sobre las tres partes que necesitaría implementar.

> [!NOTE]
> Le recomendamos encarecidamente que pruebe completamente cada parte en las secciones siguientes antes de pasar a la siguiente parte. Para probar el sistema de protección de contenido, use las herramientas especificadas en las secciones.

### <a name="media-services-code"></a>Código de Media Services
  
El [ejemplo de DRM](https://github.com/Azure-Samples/media-services-v3-dotnet-tutorials/blob/master/AMSV3Tutorials/EncryptWithDRM/Program.cs) muestra cómo implementar el sistema de varios DRM con Media Services v3 con .NET. También muestra cómo usar el servicio de entrega de claves o licencias de Media Services.
  
Puede cifrar cada recurso con varios tipos de cifrado (AES-128, PlayReady, Widevine, FairPlay). Para ver lo que conviene combinar, consulte [Protocolos de streaming y tipos de cifrado](#streaming-protocols-and-encryption-types).

En el ejemplo se muestra cómo:

1. Crear y configurar una [directiva de clave de contenido](content-key-policy-concept.md).

   Se crea una directiva de clave de contenido para configurar el modo en que la clave de contenido (que proporciona acceso seguro a los recursos) se entrega a los clientes finales:  

   * Defina la autorización de la entrega de licencias. Especifique la lógica de la comprobación de autorizaciones en función de las notificaciones de JSON Web Token (JWT).
   * Configure las licencias de [PlayReady](playready-license-template-overview.md), [Widevine](widevine-license-template-overview.md) o [FairPlay](fairplay-license-overview.md). Las plantillas le permiten configurar los derechos y los permisos para cada uno de los DRM.

     ```
     ContentKeyPolicyPlayReadyConfiguration playReadyConfig = ConfigurePlayReadyLicenseTemplate();
     ContentKeyPolicyWidevineConfiguration widevineConfig = ConfigureWidevineLicenseTempate();
     ContentKeyPolicyFairPlayConfiguration fairPlayConfig = ConfigureFairPlayPolicyOptions();
     ```

2. Cree un [localizador de streaming](streaming-locators-concept.md) configurado para transmitir un recurso cifrado.
  
   El localizador de streaming tiene que estar asociado a una [directiva de streaming](streaming-policy-concept.md). En el ejemplo, establecemos `StreamingLocator.StreamingPolicyName` en la directiva "Predefined_MultiDrmCencStreaming".

   Se aplican los cifrados PlayReady y Widevine, y la clave se entrega al cliente de reproducción en función de las licencias DRM configuradas. Si también quiere cifrar su transmisión con CBCS (FairPlay), use la directiva "Predefined_MultiDrmStreaming".

   El localizador de streaming también está asociado a la directiva de clave de contenido que se ha definido.

3. Crear un token de prueba.

   El método `GetTokenAsync` muestra cómo crear una prueba de token.
4. Crear las direcciones URL de streaming.

   El método `GetDASHStreamingUrlAsync` muestra cómo crear la dirección URL de streaming. En este caso, la dirección URL transmite el contenido de DASH.

### <a name="player-with-an-aes-or-drm-client"></a>Un reproductor con un cliente de AES o DRM.

Un reproductor de vídeo basado en un SDK de reproductor (nativo o basado en un explorador) necesita reunir los siguientes requisitos:

* El SDK del reproductor admite los clientes de DRM necesarios.
* El SDK del reproductor admite los protocolos necesarios de streaming: Smooth, DASH y/o HTTP Live Streaming (HLS).
* El SDK del reproductor puede controlar el paso de un token JWT en la solicitud de adquisición de licencia.

Puede crear un reproductor mediante la [API de Azure Media Player](https://amp.azure.net/libs/amp/latest/docs/). Utilice la [API ProtectionInfo de Azure Media Player](https://amp.azure.net/libs/amp/latest/docs/) para especificar la tecnología DRM que se usará en distintas plataformas DRM.

Para probar el contenido cifrado de AES o CENC (Widevine o PlayReady), puede usar [Azure Media Player](https://aka.ms/azuremediaplayer). No olvide seleccionar **Opciones avanzadas** y comprobar las opciones de cifrado.

Si desea probar el contenido cifrado de FairPlay, utilice [este reproductor de prueba](https://aka.ms/amtest). El reproductor admite los DRM de Widevine, PlayReady y FairPlay, así como el cifrado de claves sin cifrado AES-128.

Elija el explorador adecuado para probar diferentes DRM:

* Chrome, Opera o Firefox para Widevine.
* Microsoft Edge o Internet Explorer 11 para PlayReady.
* Safari en macOS para FairPlay.

### <a name="security-token-service"></a>Servicio de token de seguridad

Un servicio de token de seguridad (STS) emite JWT como el token de acceso para el acceso a los recursos de back-end. Puede usar el servicio de entrega de claves o licencias de Azure Media Services como recurso de back-end. Un STS debe definir lo siguiente:

* Emisor y audiencia (o ámbito).
* Notificaciones, que dependen de los requisitos empresariales de protección de contenido.
* Comprobación simétrica o asimétrica para la comprobación de firma.
* Compatibilidad con la sustitución de claves (si es necesario).

Puede usar [esta herramienta STS](https://openidconnectweb.azurewebsites.net/DRMTool/Jwt) para probar el STS. Admite los tres tipos de claves de comprobación: simétrica, asimétrica o Azure Active Directory (Azure AD) con sustitución de claves.

## <a name="streaming-protocols-and-encryption-types"></a>Protocolos de streaming y tipos de cifrado

Puede usar Media Services para entregar el contenido cifrado de forma dinámica con la clave sin cifrado AES o el cifrado DRM mediante PlayReady, Widevine o FairPlay. Actualmente, puede cifrar los formatos HLS, MPEG-DASH y Smooth Streaming. Cada protocolo admite los siguientes métodos de cifrado.

### <a name="hls"></a>HLS

El protocolo HLS admite los siguientes formatos de contenedor y esquemas de cifrado:

|Formato de contenedor|Esquema de cifrado|Ejemplo de dirección URL|
|---|---|---|
|All|AES|`https://amsv3account-usw22.streaming.media.azure.net/00000000-0000-0000-0000-000000000000/ignite.ism/manifest(format=m3u8-aapl,encryption=cbc)`|
|MPG2-TS |CBCS (FairPlay) |`https://amsv3account-usw22.streaming.media.azure.net/00000000-0000-0000-0000-000000000000/ignite.ism/manifest(format=m3u8-aapl,encryption=cbcs-aapl)`|
|CMAF(fmp4) |CBCS (FairPlay) |`https://amsv3account-usw22.streaming.media.azure.net/00000000-0000-0000-0000-000000000000/ignite.ism/manifest(format=m3u8-cmaf,encryption=cbcs-aapl)`|
|MPG2-TS |CENC (PlayReady) |`https://amsv3account-usw22.streaming.media.azure.net/00000000-0000-0000-0000-000000000000/ignite.ism/manifest(format=m3u8-aapl,encryption=cenc)`|
|CMAF(fmp4) |CENC (PlayReady) |`https://amsv3account-usw22.streaming.media.azure.net/00000000-0000-0000-0000-000000000000/ignite.ism/manifest(format=m3u8-cmaf,encryption=cenc)`|

HLS/CMAF + FairPlay (incluido HEVC/H.265) se admite en los siguientes dispositivos:

* iOS 11 o posterior.
* iPhone 8 o posterior.
* MacOS High Sierra con CPU Intel de séptima generación.

### <a name="mpeg-dash"></a>MPEG-DASH

El protocolo MPEG-DASH admite los siguientes formatos de contenedor y esquemas de cifrado:

|Formato de contenedor|Esquema de cifrado|Ejemplos de direcciones URL
|---|---|---|
|All|AES|`https://amsv3account-usw22.streaming.media.azure.net/00000000-0000-0000-0000-000000000000/ignite.ism/manifest(format=mpd-time-csf,encryption=cbc)`|
|CSF(fmp4) |CENC (Widevine + PlayReady) |`https://amsv3account-usw22.streaming.media.azure.net/00000000-0000-0000-0000-000000000000/ignite.ism/manifest(format=mpd-time-csf,encryption=cenc)`|
|CMAF(fmp4)|CENC (Widevine + PlayReady)|`https://amsv3account-usw22.streaming.media.azure.net/00000000-0000-0000-0000-000000000000/ignite.ism/manifest(format=mpd-time-cmaf,encryption=cenc)`|

### <a name="smooth-streaming"></a>Smooth Streaming

El protocolo Smooth Streaming admite los siguientes formatos de contenedor y esquemas de cifrado.

|Protocolo|Formato de contenedor|Esquema de cifrado|
|---|---|---|
|fMP4|AES|`https://amsv3account-usw22.streaming.media.azure.net/00000000-0000-0000-0000-000000000000/ignite.ism/manifest(encryption=cbc)`|
|fMP4 | CENC (PlayReady) |`https://amsv3account-usw22.streaming.media.azure.net/00000000-0000-0000-0000-000000000000/ignite.ism/manifest(encryption=cenc)`|
|fMP4 | PIFF 1.1 (PlayReady) |`https://amsv3account-usw22.streaming.media.azure.net/00000000-0000-0000-0000-000000000000/ignite.ism/manifest(encryption=piff)`|

> [!NOTE]
> La compatibilidad con PIFF 1.1 se proporciona como una solución compatible con versiones anteriores de televisores inteligentes (Samsung y LG) que implementó la versión "Silverlight" anterior de Common Encryption. Se recomienda usar solo el formato PIFF cuando sea necesario para la compatibilidad con los televisores inteligentes Samsung o LG heredados vendidos entre 2009 y 2015 compatibles con la versión PIFF 1.1 del cifrado de PlayReady. 

### <a name="browsers"></a>Exploradores

Los exploradores comunes admiten los siguientes clientes DRM:

|Browser|Cifrado|
|---|---|
|Chrome|Widevine|
|Microsoft Edge, Internet Explorer 11|PlayReady|
|Firefox|Widevine|
|Opera|Widevine|
|Safari|FairPlay|

## <a name="controlling-content-access"></a>Control del acceso al contenido

Puede controlar quién tiene acceso a su contenido configurando la directiva de claves de contenido. Media Services admite varias formas de autorizar a los usuarios que realizan solicitudes de clave. El cliente (reproductor) debe cumplir la directiva antes de que se pueda entregar la clave al cliente. La directiva de claves de contenido puede tener una restricción de *apertura* o de *token*.

Se puede usar una directiva de clave de contenido con restricción abierta cuando se desea emitir una licencia a cualquier persona sin autorización. Por ejemplo, si los ingresos se basan en AD y no en la suscripción.  

Con una directiva de clave de contenido con restricción de token, la clave de contenido solo se enviará a un cliente que presente un token JWT o un token web simple (SWT) válidos en la solicitud de clave o licencia. Este token debe ser emitido por un STS.

Puede usar Azure AD como un STS o implementar un [STS personalizado](#using-a-custom-sts). Se debe configurar el STS para crear un token firmado con las notificaciones de clave y emisión que especificó en la configuración de restricción de tokens. El servicio de entrega de claves o licencias de Media Services devuelve la licencia o clave solicitada al cliente si se cumplen estas dos condiciones:

* El token es válido.
* Las notificaciones del token coinciden con las configuradas para la licencia o la clave.

Al configurar la directiva de restricción de token, debe especificar los parámetros de clave de comprobación principal, el emisor y el público. La clave de comprobación principal contiene la clave con la que se firmó el token. El emisor es el STS que emite el token. El público, a veces denominado ámbito, describe la intención del token o del recurso cuyo acceso está autorizado por el token. El servicio de entrega de claves o licencias de Media Services valida que estos valores del token coincidan con los de la plantilla.

### <a name="token-replay-prevention"></a>Prevención de reproducción de tokens

La característica de *prevención de reproducción de tokens* permite a los clientes de Media Services establecer un límite en el número de veces que se puede usar el mismo token para solicitar una clave o una licencia. El cliente puede agregar una notificación de tipo `urn:microsoft:azure:mediaservices:maxuses` en el token, donde el valor es el número de veces que el token se puede usar para adquirir una licencia o una clave. Todas las solicitudes subsiguientes con el mismo token para la entrega de claves devolverán una respuesta no autorizada. Consulte cómo agregar la notificación en el [ejemplo de DRM](https://github.com/Azure-Samples/media-services-v3-dotnet-tutorials/blob/master/AMSV3Tutorials/EncryptWithDRM/Program.cs#L601).
 
#### <a name="considerations"></a>Consideraciones

* Los clientes deben tener control sobre la generación de tokens. La notificación debe colocarse en el propio token.
* Cuando se usa esta característica, las solicitudes con tokens cuya hora de expiración sea superior a una hora desde el momento en que se reciba la solicitud se rechazan con una respuesta no autorizada.
* Los tokens se identifican de forma única mediante su firma. Cualquier cambio en la carga útil (por ejemplo, la actualización a la hora de expiración o la notificación) cambia la firma del token y se contará como un nuevo token con el que la entrega de claves no se ha encontrado antes.
* Se produce un error en la reproducción si el token ha superado el valor de `maxuses` establecido por el cliente.
* Esta característica se puede usar para todo el contenido protegido existente (solo se debe cambiar el token emitido).
* Esta característica funciona con JWT y también con SWT.

## <a name="using-a-custom-sts"></a>Uso de un STS personalizado

Un cliente puede optar por usar un servicio de token de seguridad personalizado para proporcionar tokens. Entre los motivos se incluyen:

* El proveedor de identidades (IDP) utilizado por el cliente no admite STS. En este caso, un servicio de token de seguridad personalizado podría ser una opción.
* El cliente puede necesitar un control más flexible o más estricto para integrar el servicio de token de seguridad con el sistema de facturación de los suscriptores del cliente.

   Por ejemplo, un operador de serviciod [OTT](https://en.wikipedia.org/wiki/Over-the-top_media_services) puede ofrecer varios paquetes de suscriptor, por ejemplo, prémium, básico y deportes. El operador puede querer hacer coincidir las notificaciones de un token con el paquete de un suscriptor de forma que solo el contenido de un paquete específico esté disponible. En este caso, un STS personalizado proporciona la flexibilidad y el control necesarios.

* Para incluir las notificaciones personalizadas en el token y seleccionar entre diferentes ContentKeyPolicyOptions con distintos parámetros de licencia DRM (una licencia de suscripción en lugar de una licencia de alquiler).
* Para incluir una notificación que represente el identificador de clave de contenido de la clave a la que el token concede acceso.

Cuando se utiliza un servicio de token de seguridad personalizado, se deben realizar dos cambios:

* Al configurar el servicio de entrega de licencias para un recurso, debe especificar la clave de seguridad utilizada para la comprobación por el servicio de token de seguridad personalizado en lugar de la clave actual de Azure AD.
* Cuando se genera un token JTW, se especifica una clave de seguridad en lugar de la clave privada del certificado X509 actual en Azure AD.

Hay dos tipos de claves de seguridad:

* Clave simétrica: La misma clave se usa para generar y verificar un token JWT.
* Clave asimétrica: Se usa un par de claves público-privadas de un certificado X509 junto con la clave privada para cifrar o generar un token JWT y con la clave pública para verificar el token.

Si utiliza .NET Framework o C# como plataforma de desarrollo, el certificado X509 usado en una clave de seguridad asimétrica debe tener una longitud de clave de al menos 2048. Esta longitud de clave es un requisito de la clase System.IdentityModel.Tokens.X509AsymmetricSecurityKey en .NET Framework. De lo contrario, se produce la siguiente excepción: IDX10630: El valor 
"System.IdentityModel.Tokens.X509AsymmetricSecurityKey" para la firma no puede ser menor que 2048 bits.

## <a name="custom-key-and-license-acquisition-url"></a>Dirección URL de adquisición de licencias y claves personalizadas

Utilice las siguientes plantillas si desea especificar un servicio de entrega diferente de claves y licencias (no Media Services). Los dos campos reemplazables en las plantillas están ahí para que pueda compartir la directiva de streaming a través de muchos recursos en lugar de crear una directiva de streaming por cada recurso. 

* `EnvelopeEncryption.CustomKeyAcquisitionUrlTemplate`: plantilla para la dirección URL del servicio personalizado que proporciona claves a los reproductores del usuario final. No se requiere cuando se usa Azure Media Services para la emisión de claves. 

   La plantilla admite tokens reemplazables que el servicio actualizará en tiempo de ejecución con el valor específico de la solicitud.  Los valores de token admitidos actualmente son:
   * `{AlternativeMediaId}`, que se reemplaza por el valor de StreamingLocatorId.AlternativeMediaId.
   * `{ContentKeyId}`, que se reemplaza por el valor del identificador de la clave solicitada.
* `StreamingPolicyPlayReadyConfiguration.CustomLicenseAcquisitionUrlTemplate`: plantilla para la dirección URL del servicio personalizado que proporciona licencias a los reproductores del usuario final. No se requiere cuando se usa Azure Media Services para la emisión de licencias.

   La plantilla admite tokens reemplazables que el servicio actualizará en tiempo de ejecución con el valor específico de la solicitud. Los valores de token admitidos actualmente son:  
   * `{AlternativeMediaId}`, que se reemplaza por el valor de StreamingLocatorId.AlternativeMediaId.
   * `{ContentKeyId}`, que se reemplaza por el valor del identificador de la clave solicitada. 
* `StreamingPolicyWidevineConfiguration.CustomLicenseAcquisitionUrlTemplate`: Igual que la plantilla anterior, solo para Widevine. 
* `StreamingPolicyFairPlayConfiguration.CustomLicenseAcquisitionUrlTemplate`: Igual que la plantilla anterior, solo para FairPlay.  

Por ejemplo:

```csharp
streamingPolicy.EnvelopEncryption.customKeyAcquisitionUrlTemplate = "https://mykeyserver.hostname.com/envelopekey/{AlternativeMediaId}/{ContentKeyId}";
```

`ContentKeyId` tiene un valor de la clave solicitada. Puede usar el identificador `AlternativeMediaId` si desea asignar la solicitud a una entidad en su lado. Por ejemplo, `AlternativeMediaId` se puede usar para ayudarle a buscar los permisos.

Para ejemplos de REST que utilizan direcciones URL de adquisición de claves y licencias personalizadas, consulte [Directivas de streaming: creación](/rest/api/media/streamingpolicies/create).

> [!NOTE]
> Widevine es un servicio que ofrece Google Inc. y que está sujeto a los términos del servicio y la directiva de privacidad de Google, Inc.

## <a name="troubleshoot"></a>Solución de problemas

Si obtiene el error `MPE_ENC_ENCRYPTION_NOT_SET_IN_DELIVERY_POLICY`, asegúrese de especificar la directiva de streaming adecuada.

Si se producen errores que terminan con `_NOT_SPECIFIED_IN_URL`, asegúrese de especificar el formato de cifrado en la dirección URL. Un ejemplo es `…/manifest(format=m3u8-cmaf,encryption=cbcs-aapl)`. Consulte [Protocolos de streaming y tipos de cifrado](#streaming-protocols-and-encryption-types).

## <a name="ask-questions-give-feedback-get-updates"></a>Formule preguntas, realice comentarios y obtenga actualizaciones

Consulte el artículo [Comunidad de Azure Media Services](media-services-community.md) para ver diferentes formas de formular preguntas, enviar comentarios y obtener actualizaciones de Media Services.

## <a name="next-steps"></a>Pasos siguientes

* [Protección con el cifrado AES](protect-with-aes128.md)
* [Protección con DRM](protect-with-drm.md)
* [Diseño del sistema de protección de contenido con DRM múltiple con control de acceso](design-multi-drm-system-with-access-control.md)
* [Cifrado del lado de almacenamiento](storage-account-concept.md#storage-side-encryption)
* [Preguntas más frecuentes](frequently-asked-questions.md)
* [Controlador de JSON Web Token](/dotnet/framework/security/json-web-token-handler)
