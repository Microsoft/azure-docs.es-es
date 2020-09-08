---
title: Análisis multimedia en la plataforma Media Services | Microsoft Docs
description: Información general sobre la versión preliminar pública de Análisis multimedia, una colección de servicios de voz y visión informática a escala empresarial, cumplimiento, seguridad y alcance global
services: media-services
documentationcenter: ''
author: juliako
manager: femila
editor: ''
ms.assetid: c56e3781-8510-4f7f-b5ff-a218c1bb6f4c
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: dotnet
ms.topic: article
ms.date: 11/13/2019
ms.author: juliako
ms.reviewer: milanga; johndeu
ms.custom: devx-track-csharp
ms.openlocfilehash: 79665531e5faa9766c62b87a002efafdea2f3221
ms.sourcegitcommit: bcda98171d6e81795e723e525f81e6235f044e52
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89264732"
---
# <a name="media-analytics-on-the-media-services-platform"></a>Análisis multimedia en la plataforma Media Services

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

## <a name="retirement-plans"></a>Planes de retirada

> [!IMPORTANT]
> Algunos procesadores se retirarán. Para ver las fechas de retirada y más información, consulte el tema sobre [componentes heredados](legacy-components.md). 

## <a name="overview"></a>Información general

Cada vez más organizaciones adoptan el vídeo como medio preferido para formar a sus empleados, atraer a sus clientes y documentar las funciones empresariales. La informática en la nube proporciona una manera de almacenar estos archivos multimedia de gran tamaño, transmitirlos y acceder a ellos. Pero a medida que la biblioteca de contenido de vídeo de una empresa crece, necesita un medio igualmente eficaz de extraer información de ese contenido. 

Para satisfacer esta necesidad creciente, Azure Media Services ofrece Análisis multimedia de Azure. Análisis multimedia es una colección de componentes de voz y visión que facilita a las empresas y organizaciones obtener conocimiento útil de sus archivos de vídeo. Generado con los componentes principales de la plataforma Media Services, Análisis multimedia puede ocuparse del procesamiento multimedia a escala desde el primer día.

Con Análisis multimedia, los desarrolladores pueden integrar funcionalidad de vídeo avanzada en las aplicaciones rápidamente. Proporciona entornos empresariales con la escala total, el cumplimiento, la seguridad y el alcance global que necesitan las grandes organizaciones.

El siguiente diagrama muestra Análisis multimedia y otras partes principales de la plataforma Media Services. 

![Flujo de trabajo de VoD](./media/media-services-analytics-overview/media-services-analytics-overview01.png)

Los procesadores multimedia de Análisis multimedia generan archivos MP4 o JSON. Si un procesador multimedia genera un archivo MP4, puede descargar progresivamente el archivo. Si un procesador multimedia genera un archivo JSON, puede descargar el archivo desde Azure Blob Storage. 

## <a name="media-analytics-services"></a>Servicios de Análisis multimedia

### <a name="indexer"></a>Indexer
Con Azure Media Indexer puede realizar búsquedas en el contenido, así como generar pistas de subtítulos. Para más información y examinar algunos ejemplos, consulte [Indexación de archivos multimedia con Azure Media Indexer](media-services-index-content.md).

### <a name="motion-detector"></a>Detector de movimientos
Puede usar Motion Detector para detectar movimiento en un vídeo con fondos quietos. Esto permite buscar falsos positivos en eventos de movimiento detectados por cámaras de vigilancia. Para obtener información detallada y ejemplos, vea [Motion Detection for Azure Media Analytics (Detección de movimiento para Análisis multimedia de Azure)](media-services-motion-detection.md).

### <a name="face-detector"></a>Detector de caras
Con Face Detector puede detectar las caras y las emociones de las personas, lo que incluye felicidad, tristeza y sorpresa. Esto tiene varias aplicaciones útiles para el sector, que se describen más adelante, como la adición y el análisis de reacciones de personas que asisten a un evento. Para obtener información detallada y ejemplos, vea [Face and Emotion Detection for Azure Media Analytics (Detección de caras y emociones para Análisis multimedia de Azure)](media-services-face-and-emotion-detection.md).

### <a name="video-summarization"></a>Resumen de vídeo
El resumen de vídeo puede ayudarle a crear resúmenes de vídeos largos al seleccionar automáticamente fragmentos interesantes del vídeo original. Esta capacidad es útil si quiere proporcionar una rápida descripción de lo que se va a encontrar en un vídeo largo. Para obtener información detallada y ejemplos, vea [Usar Azure Media Video Thumbnails para crear un resumen de vídeo](media-services-video-summarization.md).
### <a name="optical-character-recognition"></a>Reconocimiento óptico de caracteres
Con Azure Media OCR (reconocimiento óptico de caracteres), puede convertir el contenido de texto de archivos de vídeo en texto digital modificable y utilizable en búsquedas. Luego puede automatizar la extracción de metadatos significativos de la señal de vídeo de los elementos multimedia.
### <a name="scalable-face-redaction"></a>Censura de rostros escalable
Azure Media Redactor es un procesador multimedia de Análisis multimedia de Azure que ofrece censura de rostros escalable en la nube. Con la censura de rostros, puede modificar un vídeo para difuminar las caras de personas seleccionadas. Es posible que quiera usar el servicio de censura de rostros en noticias en los medios de comunicación o cuando se trate de la seguridad pública. Unos minutos de material de archivo que contenga varias caras puede tardar horas en censurarse manualmente, pero con este servicio, la censura de caras solo conlleva unos cuantos pasos sencillos. Para más información, vea el artículo [Censura de rostros con Azure Media Analytics](media-services-face-redaction.md).

### <a name="content-moderation"></a>Moderación de contenido
Azure Content Moderator le permite usar la moderación automatizada en sus vídeos. Por ejemplo, puede usarlo para detectar posible contenido para adultos en los vídeos y revisar el contenido que hayan indicado sus equipos de moderadores. La moderación manual de vídeos para detectar contenido no deseado es una tarea lenta y costosa. Con este servicio y las herramientas de revisión asociadas, se combina la moderación automática con funcionalidades de intervención humana para obtener los mejores resultados en cuanto a eficacia y rentabilidad. Para más información, consulte el artículo sobre el [procesamiento de vídeos con Azure Content Moderator](media-services-content-moderation.md).

## <a name="common-scenarios"></a>Escenarios frecuentes
Análisis multimedia puede ayudar a las organizaciones y empresas a recopilar nuevos datos relevantes a partir del vídeo y a administrar más eficazmente grandes volúmenes de contenido de vídeo. Estos son algunos escenarios:

* **Centros de atención al cliente**. Incluso con la aparición de las redes sociales, los centros de atención al cliente todavía gestionan un gran porcentaje de transacciones de servicios de clientes. En estos datos de audio hay una gran cantidad de información del cliente codificada que se puede analizar para lograr una mayor satisfacción de este. Con Media Indexer, las organizaciones pueden extraer texto y crear índices y paneles de búsqueda. Luego pueden extraer inteligencia de las quejas habituales, los orígenes de las quejas y otros datos relevantes.
* **Moderación de contenido generado por el usuario**. Desde medios de comunicación a departamentos de policía, muchas organizaciones cuentan con portales orientados al público que aceptan elementos multimedia generados por el usuario, como imágenes y vídeos. El volumen de contenido puede tener picos debido a sucesos inesperados. En estos casos, es difícil llevar a cabo revisiones manuales eficaces del contenido para garantizar su idoneidad. Los clientes pueden basarse en el servicio de moderación de contenido para centrarse en el contenido adecuado.

## <a name="media-analytics-media-processors"></a>Procesadores de multimedia de Análisis multimedia
En esta sección se enumeran los procesadores de multimedia de Análisis multimedia y se muestra cómo usar .NET o REST para obtener un objeto de procesador de multimedia (MP).

### <a name="mp-names"></a>Nombres de MP

* Azure Media Indexer
* Azure Media Face Detector
* Azure Media Motion Detector
* Azure Media Video Thumbnails
* Azure Media OCR
* Azure Media Content Moderator

### <a name="net"></a>.NET
La siguiente función toma uno de los nombres de MP especificados y devuelve un objeto MP.

```csharp
static IMediaProcessor GetLatestMediaProcessorByName(string mediaProcessorName)
{
    var processor = _context.MediaProcessors
        .Where(p => p.Name == mediaProcessorName)
        .ToList()
        .OrderBy(p => new Version(p.Version))
        .LastOrDefault();

    if (processor == null)
        throw new ArgumentException(string.Format("Unknown media processor",
                                                  mediaProcessorName));

    return processor;
}
```


### <a name="rest"></a>REST
Solicitud:

```http
GET https://media.windows.net/api/MediaProcessors()?$filter=Name%20eq%20'Azure%20Media%20OCR' HTTP/1.1
DataServiceVersion: 1.0;NetFx
MaxDataServiceVersion: 3.0;NetFx
Accept: application/json
Accept-Charset: UTF-8
User-Agent: Microsoft ADO.NET Data Services
Authorization: Bearer <token>
x-ms-version: 2.19
Host: media.windows.net
```

Respuesta:

```http
. . .

{  
    "odata.metadata":"https://media.windows.net/api/$metadata#MediaProcessors",
    "value":[  
        {  
            "Id":"nb:mpid:UUID:074c3899-d9fb-448f-9ae1-4ebcbe633056",
            "Description":"Azure Media OCR",
            "Name":"Azure Media OCR",
            "Sku":"",
            "Vendor":"Microsoft",
            "Version":"1.1"
        }
    ]
}
```

## <a name="demos"></a>Demostraciones
Vea [Demostraciones de Análisis multimedia de Azure](https://azuremedialabs.azurewebsites.net/demos/Analytics.html).

## <a name="provide-feedback"></a>Envío de comentarios
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

## <a name="related-articles"></a>Artículos relacionados
Vea [Media Services Analytics announcement (Anuncio de análisis de Media Services)](https://azure.microsoft.com/blog/introducing-azure-media-analytics/).

<!-- Images -->

[overview]: ./media/media-services-video-on-demand-workflow/media-services-video-on-demand.png

## <a name="next-steps"></a>Pasos siguientes
Consulte las rutas de aprendizaje de Media Services.

[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]
