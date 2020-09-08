---
title: Uso de castLabs para proporcionar licencias de Widevine a Azure Media Services | Microsoft Docs
description: En este artículo se describe cómo puede usar Azure Media Services (AMS) para entregar una secuencia que se cifra dinámicamente por AMS con DRM tanto de PlayReady como Widevine.
services: media-services
documentationcenter: ''
author: Mingfeiy
manager: femila
editor: ''
ms.assetid: 2a9a408a-a995-49e1-8d8f-ac5b51e17d40
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/14/2019
ms.author: Juliako
ms.reviewer: willzhan
ms.openlocfilehash: 17b54b2adb21419de61d2309752987f6e4a48e41
ms.sourcegitcommit: bcda98171d6e81795e723e525f81e6235f044e52
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89264494"
---
# <a name="using-castlabs-to-deliver-widevine-licenses-to-azure-media-services"></a>Uso de castLabs para entregar licencias de Widevine a Azure Media Services

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]
 
> [!div class="op_single_selector"]
> * [Axinom](media-services-axinom-integration.md)
> * [castLabs](media-services-castlabs-integration.md)
> 
> 

## <a name="overview"></a>Información general

En este artículo se describe cómo puede usar Azure Media Services (AMS) para entregar una secuencia que se cifra dinámicamente por AMS con DRM tanto de PlayReady como Widevine. La licencia de PlayReady procede del servidor de licencias de PlayReady de Media Services y la licencia de Widevine se entrega por el servidor de licencias **castLabs** .

Para la reproducción de contenido en streaming protegido por CENC (PlayReady o Widevine), puede usar [Azure Media Player](https://aka.ms/azuremediaplayer). Consulte el [documento AMP](https://amp.azure.net/libs/amp/latest/docs/) para obtener información detallada.

En el siguiente diagrama se muestra una arquitectura de integración de castLabs y Azure Media Services de alto nivel.

![integración](./media/media-services-castlabs-integration/media-services-castlabs-integration.png)

## <a name="typical-system-set-up"></a>Configuración de sistema típico

* El contenido multimedia se almacena en AMS.
* Los id. de clave de claves de contenido se almacenan en castLabs y AMS.
* Tanto castLabs como AMS tienen la autenticación de tokens integrada. En las secciones siguientes se analizan los tokens de autenticación. 
* Cuando un cliente solicita transmitir el vídeo, el contenido se cifra dinámicamente con **cifrado común** (CENC) y se empaqueta dinámicamente por AMS a Smooth Streaming y DASH. También ofrecemos cifrado de streaming elemental de PlayReady M2TS para el protocolo de streaming HLS.
* La licencia de PlayReady se recupera del servidor de licencias de AMS y la licencia de Widevine se recupera del servidor de licencias de castLabs. 
* El reproductor multimedia decide automáticamente qué licencia capturar basándose en la capacidad de la plataforma del cliente. 

## <a name="authentication-token-generation-for-getting-a-license"></a>Generación de tokens de autenticación para obtener una licencia

Tanto castLabs como AMS admiten el formato de token JWT (token web JSON) usado para autorizar una licencia. 

### <a name="jwt-token-in-ams"></a>Token JWT en AMS

En la tabla siguiente se describen los tokens JWT en AMS. 

| Emisor | Cadena de emisor desde el servicio de tokens seguro (STS) elegido |
| --- | --- |
| Público |Cadena de audiencia del STS usado |
| Notificaciones |Un conjunto de notificaciones |
| NotBefore |Iniciar la validez del token |
| Expira |Finalizar la validez del token |
| SigningCredentials |La clave que se comparte entre el servidor de licencias de PlayReady, el servidor de licencias de castLabs y STS; podría ser una clave simétrica o asimétrica. |

### <a name="jwt-token-in-castlabs"></a>Token JWT en castLabs

En la tabla siguiente se describen los tokens JWT en castLabs. 

| Nombre | Descripción |
| --- | --- |
| optData |Cadena JSON que contiene información sobre el usuario. |
| crt |Cadena JSON que contiene información sobre el activo, su información de licencia y derechos de reproducción. |
| iat |La fecha y hora actual en la época. |
| jti |Identificador único sobre este token (cada token solo puede usarse una vez en el sistema castLabs). |

## <a name="sample-solution-setup"></a>Configuración de soluciones de ejemplo

La [solución de ejemplo](https://github.com/AzureMediaServicesSamples/CastlabsIntegration) consta de dos proyectos:

* Una aplicación de consola que puede usarse para establecer restricciones de DRM en un activo ya introducido, tanto para PlayReady como para Widevine.
* Una aplicación web que distribuye tokens, que podrían considerarse como una versión MUY SIMPLIFICADA de un STS.

Para usar la aplicación de consola:

1. Cambie el archivo app.config para configurar las credenciales de AMS, las credenciales de castLabs, la configuración de STS y la clave compartida.
2. Cargue un activo en AMS.
3. Obtenga el UUID del recurso cargado y cambie la línea 32 del archivo Program.cs:
   
      var objIAsset = _context.Assets.Where(x => x.Id == "nb:cid:UUID:dac53a5d-1500-80bd-b864-f1e4b62594cf").FirstOrDefault();
4. Use un AssetId para asignar un nombre al activo del sistema castLabs (línea 44 del archivo Program.cs).
   
   Debe establecer AssetId para **castLabs**; debe ser una cadena alfanumérica única.
5. Ejecute el programa.

Para usar la aplicación web (STS):

1. Cambie el archivo web.config para configurar el id. de comerciante de castlabs, la configuración de STS y la clave compartida.
2. Implemente en Azure Websites.
3. Vaya al sitio web.

## <a name="playing-back-a-video"></a>Reproducir un vídeo

Para reproducir un vídeo cifrado con cifrado común (PlayReady o Widevine), puede usar [Azure Media Player](https://aka.ms/azuremediaplayer). Cuando se ejecuta la aplicación de consola, se reflejan el id. de clave de contenido y la dirección URL del manifiesto.

1. Abra una nueva pestaña e inicie STS: http://[yourStsName].azurewebsites.net/api/token/assetid/[yourCastLabsAssetId]/contentkeyid/[thecontentkeyid].
2. Vaya a [Azure Media Player](https://aka.ms/azuremediaplayer).
3. Pegue la dirección URL de streaming.
4. Haga clic en la casilla **Opciones avanzadas** .
5. En el menú desplegable **Protección** , seleccione PlayReady o Widevine.
6. Pegue el token que obtuvo de su STS en el cuadro de texto Token. 
   
   El servidor de licencias de castLab no necesita el prefijo “Bearer=” delante del token. Por tanto, quítelo antes de enviar el token.
7. Actualice el reproductor.
8. El vídeo se debe estar reproduciendo.

## <a name="additional-notes"></a>Notas adicionales

* Widevine es un servicio que ofrece Google Inc. y que está sujeto a los términos del servicio y la directiva de privacidad de Google, Inc.

## <a name="media-services-learning-paths"></a>Rutas de aprendizaje de Media Services
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Envío de comentarios
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

