---
title: Análisis de los medios mediante Azure Portal| Microsoft Docs
description: En este tema se describe cómo procesar los medios con procesadores de multimedia (MP) de Análisis multimedia mediante Azure Portal.
services: media-services
documentationcenter: ''
author: Juliako
manager: femila
editor: ''
ms.assetid: 18213fc1-74f5-4074-a32b-02846fe90601
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/19/2019
ms.author: juliako
ms.openlocfilehash: 0bc8233110c02775e208470591c3e0c6eb619294
ms.sourcegitcommit: bcda98171d6e81795e723e525f81e6235f044e52
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89260992"
---
# <a name="analyze-your-media-using-the-azure-portal"></a>Análisis del contenido multimedia mediante Azure Portal

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

> [!IMPORTANT]
> Revise los [planes de retirada](media-services-analytics-overview.md#retirement-plans) de algunos procesadores de multimedia.

## <a name="overview"></a>Información general
Análisis de Azure Media Services es una colección de componentes de voz y visión (a escala empresarial, conforme, segura y de alcance global) que facilita a las organizaciones y empresas la derivación de conocimientos prácticos a partir de sus archivos de vídeo. Para obtener información más detallada de Análisis de Azure Media Services, consulte [este](media-services-analytics-overview.md) tema. 

En este tema se describe cómo procesar los medios con procesadores de multimedia (MP) de Análisis multimedia mediante Azure Portal. Los procesadores de multimedia de Análisis multimedia generan archivos MP4 o JSON. Si un procesador de multimedia genera un archivo MP4, puede descargarlo progresivamente. Si un procesador de multimedia genera un archivo JSON, puede descargarlo desde Azure Blob Storage. 

## <a name="choose-an-asset-that-you-want-to-analyze"></a>Selección de recursos para analizar
1. En [Azure Portal](https://portal.azure.com/), seleccione la cuenta de Azure Media Services.
2. En la ventana **Configuración**, seleccione **Activos**.  
   
    ![Análisis de vídeos](./media/media-services-portal-analyze/media-services-portal-analyze001.png)
3. Seleccione el recurso que desea analizar y haga clic en el botón **Analizar**.
   
    ![Análisis de vídeos](./media/media-services-portal-analyze/media-services-portal-analyze002.png)
4. En la ventana **Procesar un recurso multimedia con Análisis multimedia**, seleccione el procesador. 
   
    En el resto del artículo se explica por qué y cómo usar cada procesador. 
5. Haga clic en **Crear** para iniciar un trabajo.

## <a name="azure-media-indexer"></a>Azure Media Indexer
El procesador de multimedia **Azure Media Indexer** permite crear archivos multimedia y contenido que se puede buscar, así como generar pistas de subtítulos. En esta sección se proporcionan algunos detalles sobre las opciones que se especifican para este MP.

![Análisis de vídeos](./media/media-services-portal-analyze/media-services-portal-analyze003.png)

### <a name="language"></a>Idioma
El lenguaje natural que se reconocerá en el archivo multimedia. Por ejemplo, inglés o español. 

### <a name="captions"></a>Subtítulos
Puede elegir un formato de subtitulado que se generará a partir del contenido. Un trabajo de indexación puede generar archivos de subtítulos en los siguientes formatos:  
 
* **TTML**
* **WebVTT**

Los archivos de subtítulos (CC) en estos formatos se pueden usar para crear archivos de audio y vídeo accesibles para personas con discapacidades auditivas.

### <a name="keywords"></a>Palabras clave
Seleccione esta opción si desea generar un archivo XML de palabras clave. Este archivo contiene las palabras clave que se extraen del contenido de voz, con información de frecuencia y desplazamiento.

### <a name="job-name"></a>Nombre del trabajo
Un nombre descriptivo que permite identificar el trabajo. [Este](media-services-portal-check-job-progress.md) artículo describe cómo puede supervisar el progreso de un trabajo. 

### <a name="output-file"></a>Archivo de salida
Nombre descriptivo que permite identificar el contenido de salida. 

### <a name="speed"></a>Velocidad
Especifique la velocidad con la que se acelerará el vídeo de entrada. El resultado es una representación estabilizada y en time-lapse del vídeo de entrada.

### <a name="job-name"></a>Nombre del trabajo
Un nombre descriptivo que permite identificar el trabajo. [Este](media-services-portal-check-job-progress.md) artículo describe cómo puede supervisar el progreso de un trabajo. 

### <a name="output-file"></a>Archivo de salida
Nombre descriptivo que permite identificar el contenido de salida. 

## <a name="azure-media-face-detector"></a>Azure Media Face Detector
El procesador de multimedia (MP) **Azure Media Face Detector** (Detector de caras multimedia de Azure) le permite contar y seguir los movimientos, e incluso evaluar la participación y reacción de la audiencia a través de expresiones faciales. Este servicio contiene dos características: 

* **Detección de caras**
  
    La detección de caras busca y realiza el seguimiento de caras humanas dentro de un vídeo. Se pueden detectar varias caras y posteriormente realizar un seguimiento de las mismas a medida que se mueven, con los metadatos de tiempo y ubicación devueltos en un archivo JSON. Durante el seguimiento, intentará dar un identificador constante a la misma cara mientras la persona se mueve alrededor de la pantalla, aunque se tape o salga brevemente del fotograma.
  
  > [!NOTE]
  > Este servicio no lleva a cabo el reconocimiento facial. Si una persona abandona el fotograma o se tapa durante demasiado tiempo se le asigna un nuevo identificador cuando regresa.
  > 
  > 
* **Detección de emociones**
  
    Detección de emociones es un componente opcional del procesador de multimedia de detección de caras que devuelve análisis sobre varios atributos emocionales a partir de las caras detectadas, como felicidad, tristeza, temor, enojo, etc. 

![Análisis de vídeos](./media/media-services-portal-analyze/media-services-portal-analyze005.png)

### <a name="detection-mode"></a>Modo de detección
El procesador puede utilizar uno de los siguientes modos:

* Detección de caras
* Detección de emociones por cara
* Detección de emociones agregadas

### <a name="job-name"></a>Nombre del trabajo
Un nombre descriptivo que permite identificar el trabajo. [Este](media-services-portal-check-job-progress.md) artículo describe cómo puede supervisar el progreso de un trabajo. 

### <a name="output-file"></a>Archivo de salida
Nombre descriptivo que permite identificar el contenido de salida. 

## <a name="azure-media-motion-detector"></a>Azure Media Motion Detector
El procesador de multimedia (MP) **Detector de movimiento multimedia de Azure** permite identificar de manera eficaz las secciones de interés dentro de un vídeo que, de lo contrario, sería extenso y monótono. La detección de movimiento se puede usar en grabaciones con cámara estática para identificar las secciones del vídeo donde se produjo el movimiento. Genera un archivo JSON que contiene metadatos con marcas de tiempo y la región delimitadora donde se produjo el evento.

Orientada a las fuentes de vídeo de seguridad, esta tecnología puede clasificar el movimiento en eventos relevantes y falsos positivos, como sombras y cambios en la iluminación. Esto le permite generar alertas de seguridad a partir de las fuentes de las cámaras sin recibir correos no deseados con una cantidad interminable de eventos irrelevantes, a la vez que puede extraer momentos de interés a partir vídeos de vigilancia considerablemente extensos.

![Análisis de vídeos](./media/media-services-portal-analyze/media-services-portal-analyze006.png)

## <a name="azure-media-video-thumbnails"></a>Azure Media Video Thumbnails
Este procesador puede ayudarlo a crear resúmenes de vídeos largos al seleccionar automáticamente fragmentos interesantes del vídeo original. Esto es útil si quiere proporcionar una rápida descripción de lo que se va a encontrar en un vídeo largo. Para obtener información detallada y ejemplos, consulte [Use Azure Media Video Thumbnails to Create a Video Summarization (Usar Miniaturas de vídeo multimedia de Azure para crear un resumen de vídeo)](media-services-video-summarization.md)

![Análisis de vídeos](./media/media-services-portal-analyze/media-services-portal-analyze008.png)

### <a name="job-name"></a>Nombre del trabajo
Un nombre descriptivo que permite identificar el trabajo. [Este](media-services-portal-check-job-progress.md) artículo describe cómo puede supervisar el progreso de un trabajo. 

### <a name="output-file"></a>Archivo de salida
Nombre descriptivo que permite identificar el contenido de salida. 

## <a name="azure-media-content-moderator"></a>Azure Media Content Moderator
Este procesador le ayudará a detectar posible contenido para adultos en los vídeos. El procesador detecta automáticamente las capturas y los fotogramas clave en el vídeo. Puntúa los fotogramas clave en función del contenido para adultos y sugiere revisiones según los umbrales predeterminados. Para información detallada y ejemplos, consulte el artículo sobre la [moderación de vídeos con Azure Media Content Moderator](media-services-content-moderation.md)

![Vídeos moderados](./media/media-services-portal-analyze/media-services-portal-analyze-content-moderator.PNG)

### <a name="version"></a>Versión 
Use la "2.0".

### <a name="mode"></a>Mode
La versión 2.0 prescinde de la configuración `Mode`.

## <a name="next-steps"></a>Pasos siguientes
Ver las rutas de aprendizaje de Media Services

[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Envío de comentarios
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]
