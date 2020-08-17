---
title: Configuración de directivas de protección de contenido mediante Azure Portal | Microsoft Docs
description: En este artículo se muestra cómo usar Azure Portal para configurar las directivas de protección de contenido. El artículo también muestra cómo habilitar el cifrado dinámico para los recursos.
services: media-services
documentationcenter: ''
author: Juliako
manager: femila
editor: ''
ms.assetid: 270b3272-7411-40a9-ad42-5acdbba31154
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/19/2019
ms.author: juliako
ms.openlocfilehash: 8d0e2487632971579a1a473375eb2200f3a63a46
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87084541"
---
# <a name="configure-content-protection-policies-by-using-the-azure-portal"></a>Configuración de directivas de protección de contenido mediante Azure Portal

> [!NOTE]
> Para completar este tutorial, deberá tener una cuenta de Azure. Para obtener más información, consulte [Evaluación gratuita de Azure](https://azure.microsoft.com/pricing/free-trial/).   > No se van a agregar características o funcionalidades nuevas a Media Services, versión 2. <br/>Finalice la compra de la versión más reciente, [Media Services v3](../latest/index.yml). Consulte también la [guía de migración de la versión v2 a v3](../latest/migrate-from-v2-to-v3.md).
>

 Con Azure Media Services, puede proteger su contenido multimedia desde el momento en que deja el equipo a través de almacenamiento, procesamiento y entrega. Puede usar Media Services para entregar el contenido cifrado de forma dinámica con el Estándar de cifrado avanzado (AES) mediante claves de cifrado de 128 bits. También puede usarlo con el cifrado común (CENC) mediante el uso de PlayReady o administración de derechos digitales (DRM) Widevine y Apple FairPlay. 

Media Services proporciona un servicio para proporcionar licencias de DRM y claves sin cifrado de AES a clientes autorizados. Puede usar Azure Portal para crear una directiva de autorización de licencias o claves para todos los tipos de cifrados.

En este artículo se muestra cómo configurar una directiva de protección de contenido mediante el uso del portal. El artículo también muestra cómo aplicar el cifrado dinámico a los recursos.

## <a name="start-to-configure-content-protection"></a>Configuración de la protección de contenido
Para usar el portal para configurar la protección de contenido global con su cuenta de Media Services, siga estos pasos:

1. En el [portal](https://portal.azure.com/), seleccione la cuenta de Media Services.

1. Seleccione **Ajustes** > **Protección de contenido**.

    ![Protección de contenido](./media/media-services-portal-content-protection/media-services-content-protection001.png)

## <a name="keylicense-authorization-policy"></a>directiva de autorización de licencias o claves
Media Services admite varias formas de autenticar a los usuarios que realizan solicitudes de clave o licencia. Debe configurar la directiva de autorización de claves de contenido. Luego, el cliente debe cumplir con la directiva antes de que se pueda entregar la clave o licencia al cliente. La directiva de autorización de claves de acceso puede tener una o más restricciones de autorización, ya sea restricciones de apertura o de token.

Puede usar el portal para crear una directiva de autorización de licencias o claves para todos los tipos de cifrados.

### <a name="open-authorization"></a>Autorización open
La restricción open significa que el sistema entrega la clave a cualquier persona que realice una solicitud de clave. Esta restricción puede ser útil para realizar pruebas. 

### <a name="token-authorization"></a>Autorización de token
La directiva con restricción de token debe ir acompañada de un token emitido por un servicio de token de seguridad (STS). Media Services admite tokens en los formatos de token de web simple (SWT) y JSON Web Token (JWT). Tenga en cuenta que Media Services no proporciona STS. Puede crear un STS personalizado o usar Azure Access Control Service para emitir tokens. Se debe configurar el STS para crear un token firmado con las notificaciones de clave y emisión que especificó en la configuración de restricción de tokens. Si el token es válido y las notificaciones del token coinciden con las que hay para la clave (o licencia), el servicio de entrega de claves de Media Services devuelve la clave (o licencia) solicitada al cliente.

Al configurar la directiva de restricción de token, debe especificar los parámetros de clave de comprobación principal, el emisor y el público. La clave de comprobación principal contiene la clave con la que se firmó el token. El emisor es el servicio de token seguro que emite el token. El público (a veces denominado ámbito) describe la intención del token o del recurso cuyo acceso está autorizado por el token. El servicio de entrega de claves de los Media Services valida que estos valores del token coincidan con los valores de la plantilla.

![directiva de autorización de licencias o claves](./media/media-services-portal-content-protection/media-services-content-protection002.png)

## <a name="playready-license-template"></a>Plantilla de licencia de PlayReady
La plantilla de licencia de PlayReady establece la funcionalidad habilitada en la licencia de PlayReady. Para más información sobre la plantilla de licencia de PlayReady, consulte [Información general de plantillas de licencias de PlayReady de Media Services](media-services-playready-license-template-overview.md).

### <a name="nonpersistent"></a>No persistente
Si configura una licencia como no persistente, solo se mantiene en la memoria mientras el reproductor usa la licencia.  

![Protección de contenido no persistente](./media/media-services-portal-content-protection/media-services-content-protection003.png)

### <a name="persistent"></a>Persistente
Si configura una licencia como persistente, se guarda en el almacenamiento persistente en el cliente.

![Protección de contenido persistente](./media/media-services-portal-content-protection/media-services-content-protection004.png)

## <a name="widevine-license-template"></a>Plantilla de licencia de Widevine
La plantilla de licencia de Widevine establece la funcionalidad habilitada en las licencias de Widevine.

### <a name="basic"></a>Básica
Cuando selecciona **Básico**, la plantilla se crea con todos los valores predeterminados.

### <a name="advanced"></a>Avanzado
Para más información sobre la plantilla de derechos de Widevine, consulte [Información general sobre las plantillas de licencias de Widevine](media-services-widevine-license-template-overview.md).

![Protección avanzada de contenido](./media/media-services-portal-content-protection/media-services-content-protection005.png)

## <a name="fairplay-configuration"></a>Configuración de FairPlay
Para habilitar el cifrado FairPlay, seleccione **Configuración de FairPlay**. Luego, seleccione **Certificado de aplicación** y escriba la **clave secreta de aplicación**. Para más información sobre la configuración de FairPlay y los requisitos, consulte [Protección del contenido HLS con Apple FairPlay o Microsoft PlayReady](media-services-protect-hls-with-FairPlay.md).

![Configuración de FairPlay](./media/media-services-portal-content-protection/media-services-content-protection006.png)

## <a name="apply-dynamic-encryption-to-your-asset"></a>Aplicación del cifrado dinámico al recurso
Para aprovechar las ventajas del cifrado dinámico, codifique el archivo de origen en un conjunto de archivos MP4 de velocidad de bits adaptable.

### <a name="select-an-asset-that-you-want-to-encrypt"></a>Seleccione el recurso que desea cifrar.
Para ver todos sus recursos, seleccione **Configuración** > **Recursos**.

![Opción de recursos](./media/media-services-portal-content-protection/media-services-content-protection007.png)

### <a name="encrypt-with-aes-or-drm"></a>Cifrado con AES o DRM
Cuando selecciona **Cifrar** para un recurso, puede ver dos opciones: **AES** o **DRM**. 

#### <a name="aes"></a>AES
El cifrado de claves sin cifrado AES se habilita en todos los protocolos de streaming: Smooth Streaming, HLS y MPEG-DASH.

![Configuración de cifrado](./media/media-services-portal-content-protection/media-services-content-protection008.png)

#### <a name="drm"></a>DRM
1. Al seleccionar **DRM**, verá distintas directivas de protección de contenido (que se deben configurar según este punto) y un conjunto de protocolos de streaming:

    a. **PlayReady y Widevine con MPEG-DASH**: cifra dinámicamente su transmisión MPEG-DASH con DRM de PlayReady y Widevine.

    b. **PlayReady y Widevine con MPEG-DASH más FairPlay con HLS**: cifra dinámicamente su transmisión MPEG-DASH con DRM de PlayReady y Widevine. Esta opción también cifra las transmisiones HLS con FairPlay.

    c. **PlayReady solo con Smooth Streaming, HLS y MPEG-DASH**: cifra dinámicamente Smooth Streaming, HLS, transmisiones MPEG DASH con DRM de PlayReady.

    d. **Widevine solo con MPEG-DASH**: cifra dinámicamente MPEG-DASH con DRM de Widevine.
    
    e. **FairPlay solo con HLS**: cifra dinámicamente su transmisión HLS con FairPlay.

1. Para habilitar el cifrado de FairPlay, en la hoja **Configuración global de protección de contenido**, seleccione **Configuración de FairPlay**. Luego, seleccione **Certificado de aplicación** y escriba la **clave secreta de aplicación**.

    ![Tipo de cifrado](./media/media-services-portal-content-protection/media-services-content-protection009.png)

1. Una vez realizada la selección de cifrado, seleccione **Aplicar**.

>[!NOTE] 
>Si planea reproducir una instancia de HLS cifrada mediante AES en Safari, consulte la entrada de blog sobre [HLS cifrado en Safari](https://azure.microsoft.com/blog/how-to-make-token-authorized-aes-encrypted-hls-stream-working-in-safari/).

## <a name="additional-notes"></a>Notas adicionales

* Widevine es un servicio que ofrece Google Inc. y que está sujeto a los términos del servicio y la directiva de privacidad de Google, Inc.

## <a name="next-steps"></a>Pasos siguientes
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Envío de comentarios
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]
