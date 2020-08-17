---
title: Configuración de la directiva de autorización de claves de contenido mediante Azure Portal | Microsoft Docs
description: En este artículo se muestra cómo configurar una directiva de autorización para una clave de contenido.
services: media-services
documentationcenter: ''
author: juliako
manager: femila
editor: ''
ms.assetid: ee82a3fa-c34b-48f2-a108-8ba321f1691e
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/19/2019
ms.author: juliako
ms.openlocfilehash: 2be430fc032acd94f851072f80d14a43638c8863
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87084592"
---
# <a name="configure-a-content-key-authorization-policy"></a>Configuración de una directiva de autorización de claves de contenido
[!INCLUDE [media-services-selector-content-key-auth-policy](../../../includes/media-services-selector-content-key-auth-policy.md)]

## <a name="overview"></a>Información general
 Puede usar Azure Media Services para entregar transmisiones MPEG-DASH, Smooth Streaming y HTTP Live Streaming (HLS) protegidas con Estándar de cifrado avanzado (AES) mediante el uso de claves de cifrado de 128 bits o [administración de derechos digitales (DRM) de PlayReady](https://www.microsoft.com/playready/overview/). Con Media Services, también puede entregar transmisiones DASH cifradas con DRM de Widevine. Tanto PlayReady como Widevine se cifran según la especificación de cifrado común (ISO/IEC 23001-7 CENC).

Media Services también proporciona un servicio de entrega de claves/licencias con el que los clientes pueden obtener claves AES o licencias de PlayReady/Widevine con el fin de reproducir el contenido cifrado.

Este artículo muestra cómo usar Azure Portal para configurar la directiva de autorización de claves de contenido. La clave se puede usar posteriormente para cifrar el contenido de forma dinámica. Actualmente, puede cifrar los formatos HLS, MPEG-DASH y Smooth Streaming. No se pueden cifrar las descargas progresivas.

Cuando un reproductor solicita una transmisión que se establece para cifrarse de forma dinámica, Media Services usa la clave configurada para cifrar dinámicamente el contenido mediante cifrado AES o DRM. Para descifrar la secuencia, el reproductor solicitará la clave del servicio de entrega de claves. Para determinar si el usuario tiene permiso para obtener la clave, el servicio evalúa las directivas de autorización que especificó para la clave.

Si planea tener varias claves de contenido o desea especificar una dirección URL del servicio de entrega de claves/licencias que no sea el servicio de entrega de claves de Media Services, use el SDK de Media Services para .NET o las API de REST. Para más información, consulte:

* [Configuración de la directiva de autorización de claves mediante el SDK de Media Services para .NET](media-services-dotnet-configure-content-key-auth-policy.md)
* [Configuración de la directiva de autorización de claves mediante la API de REST de Media Services](media-services-rest-configure-content-key-auth-policy.md)

### <a name="some-considerations-apply"></a>Se aplican algunas consideraciones
* Cuando se crea la cuenta de Media Services, se agrega un punto de conexión de streaming predeterminado a la cuenta en estado Detenido. Para iniciar la transmisión del contenido y aprovechar el empaquetado dinámico y el cifrado dinámico, el punto de conexión de streaming debe estar en estado "Running" (En ejecución). 
* El recurso debe contener un conjunto de archivos MP4 de velocidad de bits adaptable o archivos Smooth Streaming de velocidad de bits adaptable. Para obtener más información, consulte [Codificación de un recurso](media-services-encode-asset.md).
* El servicio de entrega de claves almacena en caché ContentKeyAuthorizationPolicy y los objetos relacionados (opciones y restricciones de directiva) durante 15 minutos. Puede crear ContentKeyAuthorizationPolicy, especificar que se use una restricción de token, probarla y actualizar la directiva con la restricción abierta. Este proceso tarda aproximadamente unos 15 minutos antes de que la directiva cambie a su versión abierta.
* Un punto de conexión de streaming de Media Services establece el valor del encabezado CORS "Access-Control-Allow-Origin" en la respuesta de la comprobación preliminar como el comodín "\*". Este valor funciona bien con la mayoría de reproductores, incluyendo Azure Media Player, Roku y JWPlayer, entre otros. Sin embargo, algunos reproductores que usan dash.js no funcionan porque, con el modo de las credenciales establecido en "include", el elemento XMLHttpRequest de sus archivos dash.js no permite el comodín "\*" como valor de "Access-Control-Allow-Origin". Como solución alternativa a esta limitación de los archivos dash.js, si hospeda al cliente desde un solo dominio, Media Services puede especificar ese dominio en el encabezado de respuesta de la comprobación preliminar. Para obtener ayuda, abra una incidencia de soporte técnico en Azure Portal.

## <a name="configure-the-key-authorization-policy"></a>Configuración de la directiva de autorización de claves
Para configurar la directiva de autorización de claves, seleccione la página **PROTECCIÓN DE CONTENIDO** .

Media Services admite varias formas de autenticar a los usuarios que realizan solicitudes de clave. La directiva de autorización de claves de contenido pueden tener restricciones de autorización open, token, o IP. (La opción de IP se puede configurar con REST o con el SDK de .NET).

### <a name="open-restriction"></a>Restricción open
La restricción open significa que el sistema entrega la clave a cualquier persona que realice una solicitud de clave. Esta restricción puede ser útil para realizar pruebas.

![OpenPolicy][open_policy]

### <a name="token-restriction"></a>Restricción de token
Para elegir la directiva de token restringida, seleccione el botón **TOKEN**.

La directiva con restricción de token debe ir acompañada de un token emitido por un servicio de token de seguridad (STS). Media Services admite tokens en los formatos de token de web simple ([SWT](/previous-versions/azure/azure-services/gg185950(v=azure.100)#BKMK_2)) y de token de web JSON (JWT). Para más información, consulte la sección de [autenticación de JWT](http://www.gtrifonov.com/2015/01/03/jwt-token-authentication-in-azure-media-services-and-dynamic-encryption/).

Tenga en cuenta que Media Services no proporciona STS. Puede crear un STS personalizado para emitir tokens. Se debe configurar el STS para crear un token firmado con las notificaciones de clave y emisión que especificó en la configuración de restricción de tokens. Si el token es válido y las notificaciones del token coinciden con las que se han configurado en la clave de contenido, el servicio de entrega de claves de Media Services devuelve la clave de cifrado al cliente.

Al configurar la directiva de restricción de token, debe especificar los parámetros de clave de comprobación principal, el emisor y el público. La clave de comprobación principal contiene la clave con la que se firmó el token. El emisor es el STS que emite el token. El público (a veces denominado ámbito) describe la intención del token o del recurso cuyo acceso está autorizado por el token. El servicio de entrega de claves de los Media Services valida que estos valores del token coincidan con los valores de la plantilla.

### <a name="playready"></a>PlayReady
Al proteger su contenido con PlayReady, una de las cosas que debe especificar en la directiva de autorización es una cadena XML que defina la plantilla de licencias PlayReady. De forma predeterminada, se establece la siguiente directiva:

```xml
<PlayReadyLicenseResponseTemplate xmlns:i="https://www.w3.org/2001/XMLSchema-instance" xmlns="http://schemas.microsoft.com/Azure/MediaServices/KeyDelivery/PlayReadyTemplate/v1">
  <LicenseTemplates>
    <PlayReadyLicenseTemplate><AllowTestDevices>true</AllowTestDevices>
      <ContentKey i:type="ContentEncryptionKeyFromHeader" />
      <LicenseType>Nonpersistent</LicenseType>
      <PlayRight>
        <AllowPassingVideoContentToUnknownOutput>Allowed</AllowPassingVideoContentToUnknownOutput>
      </PlayRight>
    </PlayReadyLicenseTemplate>
  </LicenseTemplates>
</PlayReadyLicenseResponseTemplate>
```

Puede hacer clic en el botón **Importar directiva xml** y proporcionar un archivo XML diferente que se ajuste al esquema XML definido en [Información general de plantillas de licencias de PlayReady de Media Services](media-services-playready-license-template-overview.md).

## <a name="additional-notes"></a>Notas adicionales

* Widevine es un servicio que ofrece Google Inc. y que está sujeto a los términos del servicio y la directiva de privacidad de Google, Inc.

## <a name="next-steps"></a>Pasos siguientes
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Envío de comentarios
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

[open_policy]: ./media/media-services-portal-configure-content-key-auth-policy/media-services-protect-content-with-open-restriction.png
[token_policy]: ./media/media-services-key-authorization-policy/media-services-protect-content-with-token-restriction.png
