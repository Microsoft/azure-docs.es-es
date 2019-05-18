---
title: 'Terminología de Media Services de Azure y conceptos: Azure | Microsoft Docs'
description: En este tema se ofrece una breve descripción de los conceptos y terminología de Azure Media Services y proporciona vínculos para obtener más detalles.
services: media-services
documentationcenter: ''
author: Juliako
manager: femila
editor: ''
ms.service: media-services
ms.workload: ''
ms.topic: article
ms.date: 05/13/2019
ms.author: juliako
ms.custom: seodec18
ms.openlocfilehash: 1e76569c7f5157dce681d15ec8d499b90e080102
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65762307"
---
# <a name="media-services-concepts"></a>Conceptos de Media Services

En este tema se ofrece una breve descripción de los conceptos y terminología de Azure Media Services. El artículo también proporciona vínculos a artículos con una explicación detallada de los conceptos de Media Services v3 y funcionalidad. 

Deben revisarse los conceptos fundamentales que se describen en estos temas antes de comenzar a desarrollar.

> [!NOTE]
> Actualmente, no puede usar Azure Portal para administrar recursos de v3. Use la [API REST](https://aka.ms/ams-v3-rest-ref), la [CLI](https://aka.ms/ams-v3-cli-ref) o uno de los [SDK](media-services-apis-overview.md#sdks) admitidos.

## <a name="terminology"></a>Terminología

En esta sección se muestra cómo se asignan a la API de Media Services v3 algunos términos comunes de la industria.

### <a name="live-event"></a>Evento en directo

Un **evento en directo** representa una canalización para ingerir, transcodificación (opcionalmente) y empaquetar secuencias en directo de metadatos en tiempo real, audio y vídeo.

Para los clientes que migran desde las API de Media Services v2, la **evento en directo** reemplaza el **canal** entidad en la versión 2. Para obtener más información, consulte [migración desde v2 a v3](migrate-from-v2-to-v3.md).

### <a name="streaming-endpoint-packaging-and-origin"></a>Extremo de streaming (empaquetado y el origen)

Un **Streamingendpoint** representa un dinámico (just-in-time) empaquetado y el origen de servicio que puede entregar el contenido en directo y a petición directamente a una aplicación de Reproductor de cliente, mediante uno de los protocolos comunes de media streaming (HLS o DASH). Además, el **Streamingendpoint** proporciona el cifrado dinámico (just-in-time) para DRM de líderes del sector.

En los medios de transmisión por secuencias del sector, este servicio es comúnmente contemplado como un **Packager** o **origen**.  Otros términos comunes usados en la industria para esta funcionalidad incluyen JITP (Just-en-tiempo-packager) o JITE (Just-en-tiempo de cifrado). 
 
## <a name="cloud-upload-and-storage"></a>Carga y almacenamiento en la nube

Para empezar a administrar, cifrado, codificación, analizar y transmisión por secuencias contenido multimedia en Azure, deberá crear una cuenta de Media Services y cargue los archivos digitales en **activos**.

- [Carga y almacenamiento en la nube](storage-account-concept.md)
- [Concepto de activos](assets-concept.md)

## <a name="encoding"></a>Encoding

Una vez que cargue los archivos multimedia digitales de alta calidad en recursos, se puede codificar en formatos que se pueden reproducir en una amplia variedad de exploradores y dispositivos. 

Para codificar con Media Services v3, deberá crear **transforma** y **trabajos**.

![Transformaciones](./media/encoding/transforms-jobs.png)

- [Transformaciones y trabajos](transforms-jobs-concept.md)
- [Codificación con Media Services](encoding-concept.md)

## <a name="media-analytics"></a>Media Analytics

Para analizar los archivos de audio y vídeos, también deberá crear **transforma** y **trabajos**.

- [Analizar archivos de audio y vídeos](analyzing-video-audio-files-concept.md)

## <a name="packaging-delivery-protection"></a>Empaquetado, entrega, protección

Una vez que se codifica el contenido, puede sacar partido de **empaquetado dinámico**. En Media Services, un **Streamingendpoint**  /origen es el servicio de empaquetado dinámico utilizado para entregar contenido multimedia a los jugadores del cliente. Para que los vídeos en el recurso de salida disponibles para los clientes para la reproducción, tiene que crear un **localizador de Streaming** y, a continuación, generar direcciones URL de streaming. 

Al crear el **localizador de Streaming**, además del nombre del recurso, deberá especificar **directiva Streaming**. **Las directivas de transmisión por secuencias** le permiten definir los protocolos de streaming y cifrado opciones (si existe) para su **localizadores de Streaming**.

Empaquetado dinámico se utiliza si transmitir el contenido en vivo o bajo demanda. El siguiente diagrama muestra el streaming a petición con el flujo de trabajo de empaquetado dinámico.

![Empaquetado dinámico](./media/dynamic-packaging-overview/media-services-dynamic-packaging.png)

Con Media Services, puede entregar el contenido en directo y a petición, cifrado de forma dinámica con estándar de cifrado avanzado (AES-128) o / y cualquiera de los tres sistemas DRM (administración) de derechos digitales principales: Microsoft PlayReady, Google Widevine y Apple FairPlay. Media Services también proporciona un servicio para entregar claves AES y licencias de DMR (PlayReady, Widevine y FairPlay) a los clientes autorizados.

Si especifica las opciones de cifrado en la secuencia, cree el **directiva de clave de contenido** y asócielo con su **localizador de Streaming**. El **directiva de clave de contenido** le permite configurar cómo se entrega la clave de contenido para los clientes finales.

En la siguiente imagen se ilustra el flujo de trabajo de protección de contenido de Media Services: 

![Proteger contenido](./media/content-protection/content-protection.svg)

&#42;es compatible con cifrado dinámico AES-128 "clave sin cifrado", CBCS y CENC. 

Puede usar Media Services **los manifiestos dinámicos** transmitir sólo una copia específica o clips secundarios del vídeo. En el ejemplo siguiente, se usó un codificador para codificar un recurso intermedio en siete representaciones de vídeo MP4 ISO (de 180p a 1080p). El recurso codificado puede empaquetarse dinámicamente en cualquiera de los siguientes protocolos de transmisión: HLS, MPEG DASH y Smooth.  En la parte superior del diagrama, se muestra el manifiesto HLS para el activo sin filtros (contiene las siete representaciones).  En la parte inferior izquierda, se muestra el manifiesto HLS al que se aplicó un filtro denominado "ott". El filtro de "ott" especifica la eliminación de todas las velocidades de bits por debajo de 1 Mbps, lo que dio lugar a que se quitaran los dos niveles de calidad inferiores en la respuesta. En la parte inferior derecha se muestra el manifiesto HLS al que se aplicó un filtro denominado "móvil". El filtro "móvil" especifica la eliminación de las representaciones donde la resolución es mayor que 720p, lo que hizo que se quitaran las dos representaciones de 1080p.

![Filtrado de representaciones](./media/filters-dynamic-manifest-overview/media-services-rendition-filter.png)

- [Empaquetado dinámico](dynamic-packaging-overview.md)
- [Puntos de conexión de streaming](streaming-endpoint-concept.md)
- [Localizadores de streaming](streaming-locators-concept.md)
- [Directivas de streaming](streaming-policy-concept.md)
- [Directivas de claves de contenido](content-key-policy-concept.md)
- [Protección de contenido](content-protection-overview.md)
- [Manifiestos dinámicos](filters-dynamic-manifest-overview.md)
- [Filtros](filters-concept.md)

## <a name="live-streaming"></a>Streaming en vivo

Azure Media Services permite entregar eventos en directo a sus clientes en la nube de Azure. Los objetos **LiveEvents** son responsables de la ingesta y el procesamiento de las fuentes de vídeo en directo. Cuando creas un **evento en directo**, se crea un extremo de entrada que puede usar para enviar una señal en directo desde un codificador remoto. Una vez que la secuencia fluye en el **evento en directo**, puede comenzar el evento de streaming mediante la creación de un **activos**, **Live salida**, y **localizador de Streaming** . **Salida de Live** archivará la secuencia en la **activos** y ponerlo a disposición de los usuarios a través del **extremo de Streaming**. Un **evento en directo** puede ser uno de los dos tipos: **paso a través** y **codificación en directo**.

La siguiente imagen ilustra el flujo de trabajo de tipo de paso a través:

![paso a través](./media/live-streaming/pass-through.svg)

- [Introducción al streaming en vivo](live-streaming-overview.md)
- [Objetos LiveEvent y LiveOutput](live-events-outputs-concept.md)

## <a name="monitoring"></a>Supervisión

### <a name="event-grid"></a>Event Grid

Para ver el progreso del trabajo, debe usar **Event Grid**. Media Services también emite los tipos de evento en directo. Con Event Grid, sus aplicaciones pueden escuchar y reaccionar a eventos de casi todos los servicios de Azure y de orígenes personalizados. 

- [Control de eventos de Event Grid](reacting-to-media-services-events.md)
- [Schemas](media-services-event-schemas.md)

### <a name="azure-monitor"></a>Azure Monitor

Supervisión de las métricas y registros de diagnóstico que le ayudarán a comprenden cómo funcionan las aplicaciones con Azure Monitor.

- [Las métricas y registros de diagnóstico](media-services-metrics-diagnostic-logs.md)
- [Esquemas de los registros de diagnóstico](media-services-diagnostic-logs-schema.md)

## <a name="player-clients"></a>Clientes de Player

Puede usar Azure Media Player para reproducir contenido multimedia que se transmiten por Media Services en una amplia variedad de exploradores y dispositivos. Explorador multimedia de Azure utiliza los estándares del sector, como HTML5, Media Source Extensions (MSE, extensiones de origen multimedia) y Encrypted Media Extensions (EME, extensiones multimedia cifradas) para proporcionar una experiencia de streaming adaptativa enriquecida. 

- [Información general sobre Azure Media Player](use-azure-media-player.md)

## <a name="ask-questions-give-feedback-get-updates"></a>Formule preguntas, realice comentarios y obtenga actualizaciones

Consulte el artículo [Comunidad de Azure Media Services](media-services-community.md) para ver diferentes formas de formular preguntas, enviar comentarios y obtener actualizaciones de Media Services.

## <a name="next-steps"></a>Pasos siguientes

* [Codificación de archivos remotos y streaming de vídeo: REST](stream-files-tutorial-with-rest.md)
* [Codificación de archivos cargados y streaming de vídeo: .NET](stream-files-tutorial-with-api.md)
* [Streaming en vivo: .NET](stream-live-tutorial-with-api.md)
* [Análisis del vídeo: .NET](analyze-videos-tutorial-with-api.md)
* [Cifrado dinámico AES-128: .NET](protect-with-aes128.md)
* [Cifrado dinámico con varias soluciones de DRM: .NET](protect-with-drm.md) 
