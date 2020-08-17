---
title: Protección de su contenido con Azure Media Services | Microsoft Docs
description: En este artículo se ofrece información general sobre la protección de contenido con Azure Media Services v2.
services: media-services
documentationcenter: ''
author: Juliako
manager: femila
editor: ''
ms.assetid: 81bc00e1-dcda-4d69-b9ab-8768b793422b
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 04/01/2019
ms.author: juliako
ms.openlocfilehash: a3a931955451722d7d925be482a78e1f6cc6d43d
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87042989"
---
# <a name="content-protection-overview"></a>Introducción a la protección de contenido 

> [!NOTE]
> No hay características o funcionalidades nuevas para agregar a Media Services, versión 2. <br/>Finalice la compra de la versión más reciente, [Media Services v3](../latest/index.yml). Consulte también la [guía de migración de la versión v2 a v3](../latest/migrate-from-v2-to-v3.md).

Puede usar Azure Media Services para proteger su contenido multimedia desde el momento en que este deja el equipo y pasa a través del almacenamiento, el procesamiento y la entrega. Con Media Services puede entregar el contenido cifrado de forma dinámica en vivo y a petición con el Estándar de cifrado avanzado (AES-128) o cualquiera de los tres sistemas de administración de derechos digitales (DRM) principales: Microsoft PlayReady, Google Widevine y Apple FairPlay. Media Services también proporciona un servicio para entregar claves AES y licencias de DMR (PlayReady, Widevine y FairPlay) a los clientes autorizados. 

En la siguiente imagen se ilustra el flujo de trabajo de protección de contenido de Media Services: 

![Protección con PlayReady](./media/media-services-content-protection-overview/media-services-content-protection-with-multi-drm.png)

En este artículo se explican los conceptos y terminología pertinentes para conocer la protección de contenido con Media Services. En el artículo también se proporcionan vínculos a artículos donde se indica cómo proteger el contenido. 

## <a name="dynamic-encryption"></a>Cifrado dinámico

Puede usar Media Services para entregar el contenido cifrado de forma dinámica con la clave sin cifrado AES o el cifrado DRM mediante PlayReady, Widevine o FairPlay. Si el contenido está cifrado con una clave sin cifrado AES y se envía a través de HTTPS, no será claro hasta que llegue al cliente. 

Cada método de cifrado admite los siguientes protocolos de streaming:
 
- AES: MPEG-DASH, Smooth Streaming y HLS
- PlayReady: MPEG-DASH, Smooth Streaming y HLS
- Widevine: MPEG-DASH
- FairPlay: HLS

No se admite el cifrado en descargas progresivas. 

Para cifrar un recurso, debe asociar una clave de contenido de cifrado con el recurso y, además, configurar una directiva de autorización para la clave. Media Services puede especificar o generar automáticamente las claves de contenido.

También necesita configurar la directiva de entrega del recurso. Si desea transmitir un recurso cifrado de almacenamiento, asegúrese de configurar la directiva de entrega del recurso para especificar cómo desea entregarlo.

Cuando un reproductor solicita una transmisión, Media Services usa la clave especificada para cifrar de forma dinámica el contenido mediante la clave sin cifrado de AES o el cifrado DRM. Para descifrar la secuencia, el reproductor solicitará la clave del servicio de entrega de claves de Media Services. Para decidir si el usuario está o no autorizado para obtener la clave, el servicio evalúa las directivas de autorización que especificó para la clave.

## <a name="aes-128-clear-key-vs-drm"></a>Clave sin cifrado AES-128 frente a DRM
Los clientes suelen preguntarse si deben usar el cifrado de AES o un sistema de DRM. La diferencia principal entre los dos sistemas es que, con el cifrado de AES, la clave de contenido se transmite al cliente en un formato sin cifrar ("fuera de peligro"). Como resultado, la clave usada para cifrar el contenido se puede ver en un seguimiento de la red en el cliente en texto sin formato. La clave sin cifrado AES-128 es adecuada para los casos de uso en los que el destinatario es una entidad de confianza (p. ej., cifrado de vídeos corporativos distribuidos dentro de una empresa para su visualización por parte de los empleados).

PlayReady, Widevine y FairPlay proporcionan un nivel de cifrado más alto en comparación con el cifrado de clave sin cifrado AES-128. La clave de contenido se transmite en un formato cifrado. Además, el descifrado se controla en un entorno seguro en el nivel de sistema operativo donde a un usuario malintencionado le resulta más difícil atacar. DRM se recomienda para los casos de uso en los que es posible que el destinatario no sea una entidad de confianza y usted requiere el nivel de seguridad más alto.

## <a name="storage-encryption"></a>Cifrado de almacenamiento
Puede usar el cifrado de almacenamiento para cifrar localmente el contenido sin cifrar mediante el cifrado AES de 256 bits. A continuación puede cargarlo en Azure Storage, donde se almacenará cifrado y en reposo. Los recursos protegidos con el cifrado de almacenamiento se descifran automáticamente y se colocan en un sistema de archivos cifrados antes de la codificación. Los recursos, opcionalmente, se vuelven a cifrar antes de volver a cargarlos como un nuevo recurso de salida. El caso de uso principal para el cifrado de almacenamiento es cuando desea proteger los archivos multimedia de entrada de alta calidad con un sólido cifrado en reposo en disco.

Para entregar un recurso cifrado de almacenamiento, debe configurar la directiva de entrega del recurso para que Media Services sepa cómo desea entregar el contenido. Antes de poder transmitir el recurso, el servidor de streaming descifra y transmite el contenido mediante la directiva de entrega especificada (por ejemplo, AES, cifrado común o sin cifrado).

## <a name="types-of-encryption"></a>Tipos de cifrado
PlayReady y Widevine utilizan cifrado común (modo AES-CTR). FairPlay utiliza el cifrado de modo AES-CBC. El cifrado de clave sin cifrado AES-128 utiliza cifrado de sobre.

## <a name="licenses-and-keys-delivery-service"></a>Servicio de entrega de licencias y claves
Media Services proporciona un servicio de entrega de claves para proporcionar licencias DRM (PlayReady, Widevine, FairPlay) y claves AES a clientes autorizados. Puede utilizar [Azure Portal](media-services-portal-protect-content.md), la API de REST o SDK de Media Services para .NET para configurar las directivas de autorización y autenticación de sus licencias y claves.

## <a name="control-content-access"></a>Acceso al contenido de control
Puede controlar quién tiene acceso a su contenido configurando la directiva de autorización de claves de contenido. La directiva de autorización de claves de contenido admite la restricción open o la restricción de token.

### <a name="open-authorization"></a>Autorización open
Con una directiva de autorización open, la clave de contenido se envía a cualquier cliente (sin restricción).

### <a name="token-authorization"></a>Autorización de token
Con una directiva de autorización con restricción de token, la clave de contenido solo se enviará a un cliente que presente JSON Web Token (JWT) o Simple Web Token (SWT) válidos en la solicitud de clave/licencia. Un servicio de token seguro (STS) debe emitir este token. Puede usar Azure Active Directory como un STS o implementar un STS personalizado. Se debe configurar el STS para crear un token firmado con las notificaciones de clave y emisión que especificó en la configuración de restricción de tokens. El servicio de entrega de claves de Media Services devolverá la clave/licencia solicitada al cliente si el token es válido y las reclamaciones del token coinciden con las configuradas para la clave/licencia.

Al configurar la directiva de restricción de token, debe especificar los parámetros de clave de comprobación principal, el emisor y el público. La clave de comprobación principal contiene la clave con la que se firmó el token. El emisor es el servicio de token seguro que emite el token. El público, a veces denominado ámbito, describe la intención del token o del recurso cuyo acceso está autorizado por el token. El servicio de entrega de claves de los Media Services valida que estos valores del token coincidan con los valores de la plantilla.

### <a name="token-replay-prevention"></a>Prevención de reproducción de tokens

La característica de *prevención de reproducción de tokens* permite a los clientes de Media Services establecer un límite en el número de veces que se puede usar el mismo token para solicitar una clave o una licencia. El cliente puede agregar una notificación de tipo `urn:microsoft:azure:mediaservices:maxuses` en el token, donde el valor es el número de veces que el token se puede usar para adquirir una licencia o una clave. Todas las solicitudes subsiguientes con el mismo token para la entrega de claves devolverán una respuesta no autorizada. Consulte cómo agregar la notificación en el [ejemplo de DRM](https://github.com/Azure-Samples/media-services-v3-dotnet-tutorials/blob/master/AMSV3Tutorials/EncryptWithDRM/Program.cs#L601).
 
#### <a name="considerations"></a>Consideraciones

* Los clientes deben tener control sobre la generación de tokens. La notificación debe colocarse en el propio token.
* Cuando se usa esta característica, las solicitudes con tokens cuya hora de expiración sea superior a una hora desde el momento en que se reciba la solicitud se rechazan con una respuesta no autorizada.
* Los tokens se identifican de forma única mediante su firma. Cualquier cambio en la carga útil (por ejemplo, la actualización a la hora de expiración o la notificación) cambia la firma del token y se contará como un nuevo token con el que la entrega de claves no se ha encontrado antes.
* Se produce un error en la reproducción si el token ha superado el valor de `maxuses` establecido por el cliente.
* Esta característica se puede usar para todo el contenido protegido existente (solo se debe cambiar el token emitido).
* Esta característica funciona con JWT y también con SWT.

## <a name="streaming-urls"></a>Direcciones URL de streaming
Si el recurso se cifró con más de un DRM, use una etiqueta de cifrado en la dirección URL de streaming: (formato = 'm3u8-aapl', cifrado = 'xxx').

Se aplican las siguientes consideraciones:

* No se puede especificar más de un tipo de cifrado.
* El tipo de cifrado no tiene que especificarse en la dirección URL si solo se aplicó un cifrado al recurso.
* El tipo de cifrado distingue mayúsculas de minúsculas.
* Se pueden especificar los siguientes tipos de cifrado:

  * **cenc**: Para PlayReady o Widevine (cifrado común)
  * **cbcs-aapl**: Para FairPlay (cifrado AES-CBC)
  * **cbc**: Cifrado de sobre AES

## <a name="additional-notes"></a>Notas adicionales

* Widevine es un servicio que ofrece Google Inc. y que está sujeto a los términos del servicio y la directiva de privacidad de Google, Inc.

## <a name="next-steps"></a>Pasos siguientes
En los próximos artículos se describen los siguientes pasos para empezar con la protección de contenido:

* [Protección con el cifrado de almacenamiento](media-services-rest-storage-encryption.md)
* [Protección con el cifrado AES](media-services-protect-with-aes128.md)
* [Protección con PlayReady y/o Widevine](media-services-protect-with-playready-widevine.md)
* [Protección con FairPlay](media-services-protect-hls-with-FairPlay.md)

## <a name="related-links"></a>Vínculos relacionados

* [Autenticación de token JWT](http://www.gtrifonov.com/2015/01/03/jwt-token-authentication-in-azure-media-services-and-dynamic-encryption/)
* [Integración de la aplicación OWIN basada en MVC de Azure Media Services con Azure Active Directory y restricción de la entrega de claves de contenido basada en notificaciones de JWT](http://www.gtrifonov.com/2015/01/24/mvc-owin-azure-media-services-ad-integration/)

[content-protection]: ./media/media-services-content-protection-overview/media-services-content-protection.png
