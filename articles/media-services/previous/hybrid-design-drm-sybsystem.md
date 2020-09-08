---
title: Diseño híbrido de subsistemas DRM con Azure Media Services | Microsoft Docs
description: En este tema se describe el diseño híbrido de subsistemas DRM con Azure Media Services.
services: media-services
documentationcenter: ''
author: willzhan
manager: femila
editor: ''
ms.assetid: 18213fc1-74f5-4074-a32b-02846fe90601
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 08/31/2020
ms.author: willzhan
ms.reviewer: juliako
ms.openlocfilehash: 1bf01e98e418edfe8f15043d16a199b276b71020
ms.sourcegitcommit: bcda98171d6e81795e723e525f81e6235f044e52
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89266908"
---
# <a name="hybrid-design-of-drm-subsystems"></a>Diseño híbrido de subsistemas de DRM

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

En este tema se describe el diseño híbrido de subsistemas DRM con Azure Media Services.

## <a name="overview"></a>Información general

Azure Media Services proporciona compatibilidad para los tres siguientes sistemas DRM:

* PlayReady
* Widevine (Modular)
* FairPlay

La compatibilidad con DRM incluye cifrado de DRM (cifrado dinámico) y la entrega de licencias, y Azure Media Player admite los tres DRM como un SDK de reproductor de explorador.

Para obtener un tratamiento detallado del diseño y la implementación de subsistemas DRM/CENC, vea el documento titulado [CENC con varios DRM y control de acceso](media-services-cenc-with-multidrm-access-control.md).

Aunque se ofrece compatibilidad total con tres sistemas DRM, en ocasiones los clientes necesitan usar distintas partes de sus propia infraestructura o subsistemas, además de Azure Media Services para crear un subsistema DRM híbrido.

A continuación se incluyen algunas preguntas comunes planteadas por los clientes:

* "¿Puedo usar mis propio servidores de licencias DRM?" (En este caso, los clientes han invertido en una granja de servidores de licencias DRM con lógica de negocios insertada).
* "¿Puedo usar únicamente la entrega de licencias DRM en Azure Media Services sin hospedar contenido de AMS?"

## <a name="modularity-of-the-ams-drm-platform"></a>Modularidad de la plataforma DRM de AMS

Como parte de una plataforma integral de vídeo en la nube, DRM de Azure Media Services tiene en mente un diseño flexible modular. Puede usar Azure Media Services con cualquiera de las siguientes combinaciones diferentes descritas en la tabla siguiente (se incluye una explicación de la notación que se usa en la tabla). 

|**Hospedaje y origen del contenido**|**Cifrado de contenido**|**Entrega de licencia DRM**|
|---|---|---|
|AMS|AMS|AMS|
|AMS|AMS|Aplicaciones de terceros|
|AMS|Aplicaciones de terceros|AMS|
|AMS|Aplicaciones de terceros|Aplicaciones de terceros|
|Aplicaciones de terceros|Aplicaciones de terceros|AMS|

### <a name="content-hosting--origin"></a>Hospedaje y origen del contenido

* AMS: el recurso de vídeo se hospeda en AMS y el streaming se realiza a través de puntos de conexión de streaming de AMS (pero no necesariamente de empaquetado dinámico).
* Aplicaciones de terceros: el vídeo se hospeda y se entrega en una plataforma de streaming de terceros fuera AMS.

### <a name="content-encryption"></a>Cifrado de contenido

* AMS: el cifrado de contenido se realiza de forma dinámica o a petición mediante cifrado dinámico de AMS.
* Aplicaciones de terceros: el cifrado de contenido se realiza fuera de AMS mediante un flujo de trabajo previo al procesamiento.

### <a name="drm-license-delivery"></a>Entrega de licencias DRM

* AMS: el servicio de entrega de licencias de AMS entrega la licencia DRM.
* Aplicaciones de terceros: la licencia DRM se entrega mediante un servidor de licencias DRM de terceros fuera de AMS.

## <a name="configure-based-on-your-hybrid-scenario"></a>Configuración en función del escenario híbrido

### <a name="content-key"></a>Clave de contenido

A través de la configuración de una clave de contenido, puede controlar los siguientes atributos del cifrado dinámico de AMS y el servicio de entrega de licencias de AMS:

* La clave de contenido que se usa para el cifrado dinámico de DRM.
* El contenido de la licencia DRM que se entrega mediante servicios de entrega de licencias: derechos, clave de contenido y restricciones.
* Tipo de **restricción de la directiva de autorización de claves de contenido**: abierta, IP o restricción de token.
* Si se usa el tipo **token** de la **restricción de la directiva de autorización de claves de contenido**, la **restricción de la directiva de autorización de claves de contenido** debe cumplirse antes de que se emita una licencia.

### <a name="asset-delivery-policy"></a>Directiva de entrega de recursos

A través de la configuración de una directiva de entrega de activos, puede controlar los siguientes atributos que usa el empaquetador dinámico de AMS y el cifrado dinámico de un punto de conexión de streaming de AMS:

* Combinación de protocolo de streaming y cifrado de DRM, como DASH en CENC (PlayReady y Widevine), Smooth Streaming en PlayReady, HLS en Widevine o PlayReady.
* Las direcciones URL de entrega de licencias predeterminadas o insertadas para cada uno de los DRM implicados.
* Si las direcciones URL de adquisición de licencia (LA_URL) en DASH MPD o la lista de reproducción de HLS contienen la cadena de consulta del identificador de clave (KID) para Widevine y FairPlay, respectivamente.

## <a name="scenarios-and-samples"></a>Escenarios y ejemplos

Según las explicaciones de la sección anterior, en los cinco siguientes escenarios híbridos se usan las respectivas combinaciones de configuración de **clave de contenido**-**directiva de entrega de activos** (los ejemplos mencionados en la última columna aparecen después de la tabla):

|**Hospedaje y origen del contenido**|**Cifrado de DRM**|**Entrega de licencia DRM**|**Configurar la clave de contenido**|**Configuración de directivas de entrega de activos**|**Ejemplo**|
|---|---|---|---|---|---|
|AMS|AMS|AMS|Sí|Sí|Ejemplo 1|
|AMS|AMS|Aplicaciones de terceros|Sí|Sí|Ejemplo 2|
|AMS|Aplicaciones de terceros|AMS|Sí|No|Ejemplo 3|
|AMS|Aplicaciones de terceros|Externo|No|No|Ejemplo 4|
|Aplicaciones de terceros|Aplicaciones de terceros|AMS|Sí|No|    

En los ejemplos, la protección de PlayReady funciona para DASH y Smooth Streaming. Las direcciones URL de vídeo siguientes son direcciones URL de Smooth Streaming. Para obtener las direcciones URL de DASH correspondientes, simplemente anexe "(format=mpd-time-csf)". Puede usar el [reproductor multimedia de prueba de Azure](https://aka.ms/amtest) para probar en un explorador. Permite configurar qué protocolo de streaming usar y en qué tecnología. IE11 y Microsoft Edge en Windows 10 admiten PlayReady a través de EME. Para más información, vea los [detalles sobre la herramienta de prueba](./offline-playready-streaming-windows-10.md).

### <a name="sample-1"></a>Ejemplo 1

* Dirección URL de origen (base): `https://willzhanmswest.streaming.mediaservices.windows.net/1efbd6bb-1e66-4e53-88c3-f7e5657a9bbd/RussianWaltz.ism/manifest` 
* PlayReady LA_URL (DASH & smooth): `https://willzhanmswest.keydelivery.mediaservices.windows.net/PlayReady/` 
* Widevine LA_URL (DASH): `https://willzhanmswest.keydelivery.mediaservices.windows.net/Widevine/?kid=78de73ae-6d0f-470a-8f13-5c91f7c4` 
* FairPlay LA_URL (HLS): `https://willzhanmswest.keydelivery.mediaservices.windows.net/FairPlay/?kid=ba7e8fb0-ee22-4291-9654-6222ac611bd8` 

### <a name="sample-2"></a>Ejemplo 2

* Dirección URL de origen (base): https://willzhanmswest.streaming.mediaservices.windows.net/1a670626-4515-49ee-9e7f-cd50853e41d8/Microsoft_HoloLens_TransformYourWorld_816p23.ism/Manifest 
* PlayReady LA_URL (DASH & smooth): `http://willzhan12.cloudapp.net/PlayReady/RightsManager.asmx` 

### <a name="sample-3"></a>Ejemplo 3

* Dirección URL de origen: https://willzhanmswest.streaming.mediaservices.windows.net/8d078cf8-d621-406c-84ca-88e6b9454acc/20150807-bridges-2500.ism/manifest 
* PlayReady LA_URL (DASH & smooth): `https://willzhanmswest.keydelivery.mediaservices.windows.net/PlayReady/` 

### <a name="sample-4"></a>Ejemplo 4

* Dirección URL de origen: https://willzhanmswest.streaming.mediaservices.windows.net/7c085a59-ae9a-411e-842c-ef10f96c3f89/20150807-bridges-2500.ism/manifest 
* PlayReady LA_URL (DASH & smooth): `https://willzhan12.cloudapp.net/playready/rightsmanager.asmx` 

## <a name="additional-notes"></a>Notas adicionales

* Widevine es un servicio que ofrece Google Inc. y que está sujeto a los términos del servicio y la directiva de privacidad de Google, Inc.

## <a name="summary"></a>Resumen

En resumen, los componentes DRM de Azure Media Services son flexibles y se pueden usar en un escenario híbrido mediante la configuración correcta de la clave de contenido y la directiva de entrega de activos, como se describe en este tema.

## <a name="next-steps"></a>Pasos siguientes
Ver las rutas de aprendizaje de Media Services

[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Envío de comentarios
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]