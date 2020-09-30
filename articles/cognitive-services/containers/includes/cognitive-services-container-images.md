---
title: Repositorios e imágenes de contenedor
services: cognitive-services
author: aahill
manager: nitinme
description: Dos tablas que representan los registros de contenedor, los repositorios y los nombres de imagen de todas las ofertas de Cognitive Service.
ms.service: cognitive-services
ms.topic: include
ms.date: 09/03/2020
ms.author: aahi
ms.openlocfilehash: d514460f3cd80b5b85604251743abbbbcd1adc8c
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90906991"
---
### <a name="container-repositories-and-images"></a>Repositorios e imágenes de contenedor

Las tablas siguientes son una lista de las imágenes de contenedor disponibles que ofrece Azure Cognitive Services. Para obtener una lista completa de todos los nombres de imagen de contenedor y sus etiquetas disponibles, consulte [Etiquetas de imágenes de contenedor de Azure Cognitive Services](../container-image-tags.md). 

#### <a name="generally-available"></a>Disponibilidad general 

Microsoft Container Registry (MCR) sindica todos los contenedores con disponibilidad general para Cognitive Services. Los contenedores también están disponibles directamente desde [Docker Hub](https://hub.docker.com/_/microsoft-azure-cognitive-services).

**LUIS**

| Contenedor | Container Registry/Repositorio/Nombre de imagen |
|--|--|
| LUIS | `mcr.microsoft.com/azure-cognitive-services/language/luis` |

Consulte [Cómo ejecutar e instalar contenedores de LUIS](../../LUIS/luis-container-howto.md) para obtener más información.

**Text Analytics**

| Contenedor | Container Registry/Repositorio/Nombre de imagen |
|--|--|
| Análisis de sentimiento, versión 3 (inglés) | `mcr.microsoft.com/azure-cognitive-services/textanalytics/sentiment:3.0-en` |
| Análisis de sentimiento, versión 3 (español) | `mcr.microsoft.com/azure-cognitive-services/textanalytics/sentiment:3.0-es` |
| Análisis de sentimiento, versión 3 (francés) | `mcr.microsoft.com/azure-cognitive-services/textanalytics/sentiment:3.0-fr` |
| Análisis de sentimiento, versión 3 (italiano) | `mcr.microsoft.com/azure-cognitive-services/textanalytics/sentiment:3.0-it` |
| Análisis de sentimiento, versión 3 (alemán) | `mcr.microsoft.com/azure-cognitive-services/textanalytics/sentiment:3.0-de` |
| Análisis de sentimiento, versión 3 (chino simplificado) | `mcr.microsoft.com/azure-cognitive-services/textanalytics/sentiment:3.0-zh` |
| Análisis de sentimiento, versión 3 (chino tradicional) | `mcr.microsoft.com/azure-cognitive-services/textanalytics/sentiment:3.0-zht` |
| Análisis de sentimiento, versión 3 (japonés) | `mcr.microsoft.com/azure-cognitive-services/textanalytics/sentiment:3.0-ja` |
| Análisis de sentimiento, versión 3 (portugués) | `mcr.microsoft.com/azure-cognitive-services/textanalytics/sentiment:3.0-pt` |
| Análisis de sentimiento, versión 3 (holandés) | `mcr.microsoft.com/azure-cognitive-services/textanalytics/sentiment:3.0-nl` |

Consulte [Cómo ejecutar e instalar contenedores de Text Analytics](../../text-analytics/how-tos/text-analytics-how-to-install-containers.md) para obtener más información.

**Anomaly Detector** 

| Contenedor | Container Registry/Repositorio/Nombre de imagen |
|--|--|
| Anomaly Detector | `mcr.microsoft.com/azure-cognitive-services/decision/anomaly-detector` |

Consulte el artículo sobre [cómo ejecutar e instalar contenedores de Anomaly Detector](../../anomaly-detector/anomaly-detector-container-howto.md) para más información.

**Servicio Voz**

> [!NOTE]
> Para usar los contenedores de Voz, debe completar un [formulario de solicitud en línea](https://aka.ms/csgate).

| Contenedor | Container Registry/Repositorio/Nombre de imagen |
|--|--|
| [Voz a texto](../../speech-service/speech-container-howto.md?tab=stt) | `mcr.microsoft.com/azure-cognitive-services/speechservices/speech-to-text` |
| [Conversión de voz a texto personalizada](../../speech-service/speech-container-howto.md?tab=cstt) | `mcr.microsoft.com/azure-cognitive-services/speechservices/custom-speech-to-text` |
| [Texto a voz](../../speech-service/speech-container-howto.md?tab=tts) | `mcr.microsoft.com/azure-cognitive-services/speechservices/text-to-speech` |

#### <a name="ungated-preview"></a>De versión preliminar "no validados" 

Los siguientes contenedores de versión preliminar están disponibles públicamente. Microsoft Container Registry sindica todos los contenedores no validados con disponibilidad general para Cognitive Services. Los contenedores también están disponibles directamente desde [Docker Hub](https://hub.docker.com/_/microsoft-azure-cognitive-services).

| Servicio | Contenedor | Container Registry/Repositorio/Nombre de imagen |
|--|--|--|
| [Text Analytics](../../text-analytics/how-tos/text-analytics-how-to-install-containers.md) | Extracción de frases clave | `mcr.microsoft.com/azure-cognitive-services/textanalytics/keyphrase` |
| [Text Analytics](../../text-analytics/how-tos/text-analytics-how-to-install-containers.md) | Detección de idiomas | `mcr.microsoft.com/azure-cognitive-services/textanalytics/language` |


#### <a name="gated-preview"></a>De versión preliminar "validados"

Anteriormente, los contenedores de versión preliminar validados se hospedaban en el repositorio de `containerpreview.azurecr.io`. A partir del 22 de septiembre de 2020, estos contenedores (excepto Text Analytics for Health) se hospedan en la instancia de Microsoft Container Registry (MCR) y su descarga no requiere el uso del comando login de Docker. Para usar el contenedor, tendrá que:

1. Completar un [formulario de solicitud](https://aka.ms/csgate) con su identificador de suscripción de Azure y el escenario de usuario. 
2. Tras la aprobación, descargar el contenedor desde MCR. 
3. Usar la clave y el punto de conexión de un recurso de Azure adecuado para autenticar el contenedor en tiempo de ejecución. 

| Servicio | Contenedor | Container Registry/Repositorio/Nombre de imagen |
|--|--|--|
| [Computer Vision](../../Computer-vision/computer-vision-how-to-install-containers.md) | Read v3.0 | `mcr.microsoft.com/azure-cognitive-services/vision/read:3.0-preview` |
| [Computer Vision](../../Computer-vision/computer-vision-how-to-install-containers.md) | Read v3.1 | `mcr.microsoft.com/azure-cognitive-services/vision/read:3.1-preview` |
| [Computer Vision](https://docs.microsoft.com/azure/cognitive-services/computer-vision/spatial-analysis-container) | Análisis espacial | `mcr.microsoft.com/azure-cognitive-services/vision/spatial-analysis` |
| [Speech Service API](../../speech-service/speech-container-howto.md?tab=ctts) | Conversión de texto a voz personalizada | `mcr.microsoft.com/azure-cognitive-services/speechservices/custom-text-to-speech` |
| [Speech Service API](../../speech-service/speech-container-howto.md?tab=lid) | Detección de idiomas | `mcr.microsoft.com/azure-cognitive-services/speechservices/language-detection` |
| [Speech Service API](../../speech-service/speech-container-howto.md?tab=ntts) | Texto a voz neuronal | `mcr.microsoft.com/azure-cognitive-services/speechservices/neural-text-to-speech` |
| [Text Analytics for Health](../../text-analytics/how-tos/text-analytics-how-to-install-containers.md?tabs=health) | Text Analytics for Health | `containerpreview.azurecr.io/microsoft/cognitive-services-healthcare` |

