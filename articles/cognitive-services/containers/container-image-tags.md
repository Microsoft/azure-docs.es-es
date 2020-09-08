---
title: Etiquetas de imágenes de contenedor de Cognitive Services
titleSuffix: Azure Cognitive Services
description: Una lista completa de todas las etiquetas de imágenes de contenedor de Cognitive Services.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.topic: reference
ms.date: 04/01/2020
ms.author: aahi
ms.openlocfilehash: e0df3de5eadfd2cc5c00c52da5c4942b42a68b2b
ms.sourcegitcommit: 5b6acff3d1d0603904929cc529ecbcfcde90d88b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/21/2020
ms.locfileid: "88722575"
---
# <a name="azure-cognitive-services-container-image-tags"></a>Etiquetas de imágenes de contenedor de Azure Cognitive Services

Azure Cognitive Services ofrece muchas imágenes de contenedor. Los registros de contenedor y los repositorios correspondientes varían entre las imágenes de contenedor. Cada nombre de imagen de contenedor ofrece varias etiquetas. Una etiqueta de imagen de contenedor es un mecanismo de control de versiones de la imagen de contenedor. Este artículo está pensado para usarse como una referencia completa para mostrar todas las imágenes de contenedor de Cognitive Services y sus etiquetas disponibles.

> [!TIP]
> Al usar [`docker pull`](https://docs.docker.com/engine/reference/commandline/pull/), preste especial atención a las mayúsculas y minúsculas del registro de contenedor, el repositorio, el nombre de la imagen del contenedor y la etiqueta correspondiente, ya que se **distingue mayúsculas de minúsculas**.

## <a name="anomaly-detector"></a>Anomaly Detector

La imagen del contenedor de [Anomaly Detector][ad-containers] se puede encontrar en el sindicato del registro de contenedor de `mcr.microsoft.com`. Reside en el repositorio `azure-cognitive-services` y se denomina `anomaly-detector`. El nombre completo de la imagen de contenedor es `mcr.microsoft.com/azure-cognitive-services/anomaly-detector`.

Esta imagen de contenedor tiene disponibles las siguientes etiquetas:

| Etiquetas de imagen                    | Notas |
|-------------------------------|:------|
| `latest`                      |       |

## <a name="computer-vision"></a>Computer Vision

La imagen del contenedor de lectura de OCR de [Computer Vision][cv-containers] se puede encontrar en el registro de contenedor de `containerpreview.azurecr.io`. Reside en el repositorio `microsoft` y se denomina `cognitive-services-read`. El nombre completo de la imagen de contenedor es `containerpreview.azurecr.io/microsoft/cognitive-services-read`.

Esta imagen de contenedor tiene disponibles las siguientes etiquetas:

| Etiquetas de imagen                    | Notas |
|-------------------------------|:------|
| `latest ( (2.0.013250001-amd64-preview)` | • Se reduce aún más el uso de memoria para el contenedor. |
|                                          | • La memoria caché externa es necesaria para la configuración de varios pods. Por ejemplo, configure Redis para el almacenamiento en caché. |
|                                          | • Se corrige el problema de los resultados que faltan cuando se configura Redis Cache y ResultExpirationPeriod=0.  |
|                                          | • Se quita el límite de 26 MB para el tamaño del cuerpo de la solicitud. Ahora, el contenedor puede aceptar archivos de más de 26 MB.  |
|                                          | • Se agrega una marca de tiempo y una versión de compilación al registro de la consola.  |
| `1.1.013050001-amd64-preview`            | * Se ha agregado la configuración de inicialización de contenedores ReadEngineConfig:ResultExpirationPeriod para especificar el momento en que el sistema debe limpiar los resultados de reconocimiento. |
|                                          | La configuración se encuentra en horas y el valor predeterminado es 48 h.   |
|                                          |   La configuración puede reducir el uso de memoria para el almacenamiento de resultados, en especial cuando se usa el almacenamiento en memoria del contenedor.  |
|                                          |    * Ejemplo 1. ReadEngineConfig:ResultExpirationPeriod=1: el sistema borrará el resultado del reconocimiento 1 h después del proceso.   |
|                                          |    * Ejemplo 2. ReadEngineConfig:ResultExpirationPeriod=0: el sistema borrará el resultado del reconocimiento después de recuperarlo.  |
|                                          | Se corrigió un error interno del servidor (500) cuando se pasa un formato de imagen no válido al sistema. Ahora se devolverá un error 400:   |
|                                          | `{`  |
|                                          | `"error": {`  |
|                                          |      `"code": "InvalidImageSize",`  |
|                                          |      `"message": "Image must be between 1024 and 209715200 bytes."`  |
|                                          |          `}`  |
|                                          | `}`  |
| `1.1.011580001-amd64-preview` |       |
| `1.1.009920003-amd64-preview` |       |
| `1.1.009910003-amd64-preview` |       |

## <a name="face"></a>Caras

La imagen de contenedor [Caras][fa-containers] se puede encontrar en el registro de contenedor `containerpreview.azurecr.io`. Reside en el repositorio `microsoft` y se denomina `cognitive-services-face`. El nombre completo de la imagen de contenedor es `containerpreview.azurecr.io/microsoft/cognitive-services-face`.

Esta imagen de contenedor tiene disponibles las siguientes etiquetas:

| Etiquetas de imagen                    | Notas |
|-------------------------------|:------|
| `latest`                      |       |
| `1.1.009301-amd64-preview`    |       |
| `1.1.008710001-amd64-preview` |       |
| `1.1.007750002-amd64-preview` |       |
| `1.1.007360001-amd64-preview` |       |
| `1.1.006770001-amd64-preview` |       |
| `1.1.006490002-amd64-preview` |       |
| `1.0.005940002-amd64-preview` |       |
| `1.0.005550001-amd64-preview` |       |

## <a name="form-recognizer"></a>Form Recognizer

La imagen del contenedor de [Form Recognizer][fr-containers] se puede encontrar en el registro de contenedor `containerpreview.azurecr.io`. Reside en el repositorio `microsoft` y se denomina `cognitive-services-form-recognizer`. El nombre completo de la imagen de contenedor es `containerpreview.azurecr.io/microsoft/cognitive-services-form-recognizer`.

Esta imagen de contenedor tiene disponibles las siguientes etiquetas:

| Etiquetas de imagen                    | Notas |
|-------------------------------|:------|
| `latest`                      |       |
| `1.1.009301-amd64-preview`    |       |
| `1.1.008640001-amd64-preview` |       |
| `1.1.008510001-amd64-preview` |       |

## <a name="language-understanding-luis"></a>Language Understanding (LUIS)

La imagen de contenedor [LUIS][lu-containers] se puede encontrar en el sindicato de registro de contenedor `mcr.microsoft.com`. Reside en el repositorio `azure-cognitive-services` y se denomina `luis`. El nombre completo de la imagen de contenedor es `mcr.microsoft.com/azure-cognitive-services/luis`.

Esta imagen de contenedor tiene disponibles las siguientes etiquetas:

| Etiquetas de imagen                    | Notas |
|-------------------------------|:------|
| `latest`                      |       |
| `1.1.010330004-amd64-preview` |       |
| `1.1.009301-amd64-preview`    |       |
| `1.1.008710001-amd64-preview` |       |
| `1.1.008510001-amd64-preview` |       |
| `1.1.008010002-amd64-preview` |       |
| `1.1.007750002-amd64-preview` |       |
| `1.1.007360001-amd64-preview` |       |
| `1.1.007020001-amd64-preview` |       |

## <a name="custom-speech-to-text"></a>Conversión de voz a texto personalizada

La imagen de contenedor [Conversión de voz en texto personalizada][sp-cstt] se puede encontrar en el registro de contenedor `containerpreview.azurecr.io`. Reside en el repositorio `microsoft` y se denomina `cognitive-services-custom-speech-to-text`. El nombre completo de la imagen de contenedor es `containerpreview.azurecr.io/microsoft/cognitive-services-custom-speech-to-text`.

Esta imagen de contenedor tiene disponibles las siguientes etiquetas:

| Etiquetas de imagen            | Notas |
|-----------------------|:------|
| `latest`              |       |
| `2.2.0-amd64-preview` |       |
| `2.1.1-amd64-preview` |       |
| `2.1.0-amd64-preview` |       |
| `2.0.2-amd64-preview` |       |
| `2.0.0-amd64-preview` |       |

## <a name="custom-text-to-speech"></a>Conversión de texto a voz personalizada

La imagen de contenedor [Conversión de texto a voz personalizada][sp-ctts] se puede encontrar en el registro de contenedor `containerpreview.azurecr.io`. Reside en el repositorio `microsoft` y se denomina `cognitive-services-custom-text-to-speech`. El nombre completo de la imagen de contenedor es `containerpreview.azurecr.io/microsoft/cognitive-services-custom-text-to-speech`.

Esta imagen de contenedor tiene disponibles las siguientes etiquetas:

| Etiquetas de imagen            | Notas |
|-----------------------|:------|
| `latest`              |       |
| `1.3.0-amd64-preview` |       |

## <a name="speech-to-text"></a>Voz a texto

La imagen de contenedor [Conversión de voz en texto][sp-stt] se puede encontrar en el registro de contenedor `containerpreview.azurecr.io`. Reside en el repositorio `microsoft` y se denomina `cognitive-services-speech-to-text`. El nombre completo de la imagen de contenedor es `containerpreview.azurecr.io/microsoft/cognitive-services-speech-to-text`.

Esta imagen de contenedor tiene disponibles las siguientes etiquetas:

| Etiquetas de imagen                  | Notas                                    |
|-----------------------------|:-----------------------------------------|
| `latest`                    | Imagen de contenedor con la configuración regional `en-US`. |
| `2.2.0-amd64-ar-ae-preview` | Imagen de contenedor con la configuración regional `ar-AE`. |
| `2.2.0-amd64-ar-eg-preview` | Imagen de contenedor con la configuración regional `ar-EG`. |
| `2.2.0-amd64-ar-kw-preview` | Imagen de contenedor con la configuración regional `ar-KW`. |
| `2.2.0-amd64-ar-qa-preview` | Imagen de contenedor con la configuración regional `ar-QA`. |
| `2.2.0-amd64-ar-sa-preview` | Imagen de contenedor con la configuración regional `ar-SA`. |
| `2.2.0-amd64-ca-es-preview` | Imagen de contenedor con la configuración regional `ca-ES`. |
| `2.2.0-amd64-da-dk-preview` | Imagen de contenedor con la configuración regional `da-DK`. |
| `2.2.0-amd64-de-de-preview` | Imagen de contenedor con la configuración regional `de-DE`. |
| `2.2.0-amd64-en-au-preview` | Imagen de contenedor con la configuración regional `en-AU`. |
| `2.2.0-amd64-en-ca-preview` | Imagen de contenedor con la configuración regional `en-CA`. |
| `2.2.0-amd64-en-gb-preview` | Imagen de contenedor con la configuración regional `en-GB`. |
| `2.2.0-amd64-en-in-preview` | Imagen de contenedor con la configuración regional `en-IN`. |
| `2.2.0-amd64-en-nz-preview` | Imagen de contenedor con la configuración regional `en-NZ`. |
| `2.2.0-amd64-en-us-preview` | Imagen de contenedor con la configuración regional `en-US`. |
| `2.2.0-amd64-es-es-preview` | Imagen de contenedor con la configuración regional `es-ES`. |
| `2.2.0-amd64-es-mx-preview` | Imagen de contenedor con la configuración regional `es-MX`. |
| `2.2.0-amd64-fi-fi-preview` | Imagen de contenedor con la configuración regional `fi-FI`. |
| `2.2.0-amd64-fr-ca-preview` | Imagen de contenedor con la configuración regional `fr-CA`. |
| `2.2.0-amd64-fr-fr-preview` | Imagen de contenedor con la configuración regional `fr-FR`. |
| `2.2.0-amd64-gu-in-preview` | Imagen de contenedor con la configuración regional `gu-IN`. |
| `2.2.0-amd64-hi-in-preview` | Imagen de contenedor con la configuración regional `hi-IN`. |
| `2.2.0-amd64-it-it-preview` | Imagen de contenedor con la configuración regional `it-IT`. |
| `2.2.0-amd64-ja-jp-preview` | Imagen de contenedor con la configuración regional `ja-JP`. |
| `2.2.0-amd64-ko-kr-preview` | Imagen de contenedor con la configuración regional `ko-KR`. |
| `2.2.0-amd64-mr-in-preview` | Imagen de contenedor con la configuración regional `mr-IN`. |
| `2.2.0-amd64-nb-no-preview` | Imagen de contenedor con la configuración regional `nb-NO`. |
| `2.2.0-amd64-nl-nl-preview` | Imagen de contenedor con la configuración regional `nl-NL`. |
| `2.2.0-amd64-pl-pl-preview` | Imagen de contenedor con la configuración regional `pl-PL`. |
| `2.2.0-amd64-pt-br-preview` | Imagen de contenedor con la configuración regional `pt-BR`. |
| `2.2.0-amd64-pt-pt-preview` | Imagen de contenedor con la configuración regional `pt-PT`. |
| `2.2.0-amd64-ru-ru-preview` | Imagen de contenedor con la configuración regional `ru-RU`. |
| `2.2.0-amd64-sv-se-preview` | Imagen de contenedor con la configuración regional `sv-SE`. |
| `2.2.0-amd64-ta-in-preview` | Imagen de contenedor con la configuración regional `ta-IN`. |
| `2.2.0-amd64-te-in-preview` | Imagen de contenedor con la configuración regional `te-IN`. |
| `2.2.0-amd64-th-th-preview` | Imagen de contenedor con la configuración regional `th-TH`. |
| `2.2.0-amd64-tr-tr-preview` | Imagen de contenedor con la configuración regional `tr-TR`. |
| `2.2.0-amd64-zh-cn-preview` | Imagen de contenedor con la configuración regional `zh-CN`. |
| `2.2.0-amd64-zh-hk-preview` | Imagen de contenedor con la configuración regional `zh-HK`. |
| `2.2.0-amd64-zh-tw-preview` | Imagen de contenedor con la configuración regional `zh-TW`. |
| `2.1.1-amd64-en-us-preview` | Imagen de contenedor con la configuración regional `en-US`. |
| `2.1.1-amd64-ar-ae-preview` | Imagen de contenedor con la configuración regional `ar-AE`. |
| `2.1.1-amd64-ar-eg-preview` | Imagen de contenedor con la configuración regional `ar-EG`. |
| `2.1.1-amd64-ar-kw-preview` | Imagen de contenedor con la configuración regional `ar-KW`. |
| `2.1.1-amd64-ar-qa-preview` | Imagen de contenedor con la configuración regional `ar-QA`. |
| `2.1.1-amd64-ar-sa-preview` | Imagen de contenedor con la configuración regional `ar-SA`. |
| `2.1.1-amd64-ca-es-preview` | Imagen de contenedor con la configuración regional `ca-ES`. |
| `2.1.1-amd64-da-dk-preview` | Imagen de contenedor con la configuración regional `da-DK`. |
| `2.1.1-amd64-de-de-preview` | Imagen de contenedor con la configuración regional `de-DE`. |
| `2.1.1-amd64-en-au-preview` | Imagen de contenedor con la configuración regional `en-AU`. |
| `2.1.1-amd64-en-ca-preview` | Imagen de contenedor con la configuración regional `en-CA`. |
| `2.1.1-amd64-en-gb-preview` | Imagen de contenedor con la configuración regional `en-GB`. |
| `2.1.1-amd64-en-in-preview` | Imagen de contenedor con la configuración regional `en-IN`. |
| `2.1.1-amd64-en-nz-preview` | Imagen de contenedor con la configuración regional `en-NZ`. |
| `2.1.1-amd64-en-us-preview` | Imagen de contenedor con la configuración regional `en-US`. |
| `2.1.1-amd64-es-es-preview` | Imagen de contenedor con la configuración regional `es-ES`. |
| `2.1.1-amd64-es-mx-preview` | Imagen de contenedor con la configuración regional `es-MX`. |
| `2.1.1-amd64-fi-fi-preview` | Imagen de contenedor con la configuración regional `fi-FI`. |
| `2.1.1-amd64-fr-ca-preview` | Imagen de contenedor con la configuración regional `fr-CA`. |
| `2.1.1-amd64-fr-fr-preview` | Imagen de contenedor con la configuración regional `fr-FR`. |
| `2.1.1-amd64-gu-in-preview` | Imagen de contenedor con la configuración regional `gu-IN`. |
| `2.1.1-amd64-hi-in-preview` | Imagen de contenedor con la configuración regional `hi-IN`. |
| `2.1.1-amd64-it-it-preview` | Imagen de contenedor con la configuración regional `it-IT`. |
| `2.1.1-amd64-ja-jp-preview` | Imagen de contenedor con la configuración regional `ja-JP`. |
| `2.1.1-amd64-ko-kr-preview` | Imagen de contenedor con la configuración regional `ko-KR`. |
| `2.1.1-amd64-mr-in-preview` | Imagen de contenedor con la configuración regional `mr-IN`. |
| `2.1.1-amd64-nb-no-preview` | Imagen de contenedor con la configuración regional `nb-NO`. |
| `2.1.1-amd64-nl-nl-preview` | Imagen de contenedor con la configuración regional `nl-NL`. |
| `2.1.1-amd64-pl-pl-preview` | Imagen de contenedor con la configuración regional `pl-PL`. |
| `2.1.1-amd64-pt-br-preview` | Imagen de contenedor con la configuración regional `pt-BR`. |
| `2.1.1-amd64-pt-pt-preview` | Imagen de contenedor con la configuración regional `pt-PT`. |
| `2.1.1-amd64-ru-ru-preview` | Imagen de contenedor con la configuración regional `ru-RU`. |
| `2.1.1-amd64-sv-se-preview` | Imagen de contenedor con la configuración regional `sv-SE`. |
| `2.1.1-amd64-ta-in-preview` | Imagen de contenedor con la configuración regional `ta-IN`. |
| `2.1.1-amd64-te-in-preview` | Imagen de contenedor con la configuración regional `te-IN`. |
| `2.1.1-amd64-th-th-preview` | Imagen de contenedor con la configuración regional `th-TH`. |
| `2.1.1-amd64-tr-tr-preview` | Imagen de contenedor con la configuración regional `tr-TR`. |
| `2.1.1-amd64-zh-cn-preview` | Imagen de contenedor con la configuración regional `zh-CN`. |
| `2.1.1-amd64-zh-hk-preview` | Imagen de contenedor con la configuración regional `zh-HK`. |
| `2.1.1-amd64-zh-tw-preview` | Imagen de contenedor con la configuración regional `zh-TW`. |
| `2.1.0-amd64-ar-ae-preview` | Imagen de contenedor con la configuración regional `ar-AE`. |
| `2.1.0-amd64-ar-eg-preview` | Imagen de contenedor con la configuración regional `ar-EG`. |
| `2.1.0-amd64-ar-kw-preview` | Imagen de contenedor con la configuración regional `ar-KW`. |
| `2.1.0-amd64-ar-qa-preview` | Imagen de contenedor con la configuración regional `ar-QA`. |
| `2.1.0-amd64-ar-sa-preview` | Imagen de contenedor con la configuración regional `ar-SA`. |
| `2.1.0-amd64-ca-es-preview` | Imagen de contenedor con la configuración regional `ca-ES`. |
| `2.1.0-amd64-da-dk-preview` | Imagen de contenedor con la configuración regional `da-DK`. |
| `2.1.0-amd64-de-de-preview` | Imagen de contenedor con la configuración regional `de-DE`. |
| `2.1.0-amd64-en-au-preview` | Imagen de contenedor con la configuración regional `en-AU`. |
| `2.1.0-amd64-en-ca-preview` | Imagen de contenedor con la configuración regional `en-CA`. |
| `2.1.0-amd64-en-gb-preview` | Imagen de contenedor con la configuración regional `en-GB`. |
| `2.1.0-amd64-en-in-preview` | Imagen de contenedor con la configuración regional `en-IN`. |
| `2.1.0-amd64-en-nz-preview` | Imagen de contenedor con la configuración regional `en-NZ`. |
| `2.1.0-amd64-en-us-preview` | Imagen de contenedor con la configuración regional `en-US`. |
| `2.1.0-amd64-es-es-preview` | Imagen de contenedor con la configuración regional `es-ES`. |
| `2.1.0-amd64-es-mx-preview` | Imagen de contenedor con la configuración regional `es-MX`. |
| `2.1.0-amd64-fi-fi-preview` | Imagen de contenedor con la configuración regional `fi-FI`. |
| `2.1.0-amd64-fr-ca-preview` | Imagen de contenedor con la configuración regional `fr-CA`. |
| `2.1.0-amd64-fr-fr-preview` | Imagen de contenedor con la configuración regional `fr-FR`. |
| `2.1.0-amd64-gu-in-preview` | Imagen de contenedor con la configuración regional `gu-IN`. |
| `2.1.0-amd64-hi-in-preview` | Imagen de contenedor con la configuración regional `hi-IN`. |
| `2.1.0-amd64-it-it-preview` | Imagen de contenedor con la configuración regional `it-IT`. |
| `2.1.0-amd64-ja-jp-preview` | Imagen de contenedor con la configuración regional `ja-JP`. |
| `2.1.0-amd64-ko-kr-preview` | Imagen de contenedor con la configuración regional `ko-KR`. |
| `2.1.0-amd64-mr-in-preview` | Imagen de contenedor con la configuración regional `mr-IN`. |
| `2.1.0-amd64-nb-no-preview` | Imagen de contenedor con la configuración regional `nb-NO`. |
| `2.1.0-amd64-nl-nl-preview` | Imagen de contenedor con la configuración regional `nl-NL`. |
| `2.1.0-amd64-pl-pl-preview` | Imagen de contenedor con la configuración regional `pl-PL`. |
| `2.1.0-amd64-pt-br-preview` | Imagen de contenedor con la configuración regional `pt-BR`. |
| `2.1.0-amd64-pt-pt-preview` | Imagen de contenedor con la configuración regional `pt-PT`. |
| `2.1.0-amd64-ru-ru-preview` | Imagen de contenedor con la configuración regional `ru-RU`. |
| `2.1.0-amd64-sv-se-preview` | Imagen de contenedor con la configuración regional `sv-SE`. |
| `2.1.0-amd64-ta-in-preview` | Imagen de contenedor con la configuración regional `ta-IN`. |
| `2.1.0-amd64-te-in-preview` | Imagen de contenedor con la configuración regional `te-IN`. |
| `2.1.0-amd64-th-th-preview` | Imagen de contenedor con la configuración regional `th-TH`. |
| `2.1.0-amd64-tr-tr-preview` | Imagen de contenedor con la configuración regional `tr-TR`. |
| `2.1.0-amd64-zh-cn-preview` | Imagen de contenedor con la configuración regional `zh-CN`. |
| `2.1.0-amd64-zh-hk-preview` | Imagen de contenedor con la configuración regional `zh-HK`. |
| `2.1.0-amd64-zh-tw-preview` | Imagen de contenedor con la configuración regional `zh-TW`. |
| `2.0.3-amd64-ja-jp-preview` | Imagen de contenedor con la configuración regional `ja-JP`. |
| `2.0.2-amd64-ar-ae-preview` | Imagen de contenedor con la configuración regional `ar-AE`. |
| `2.0.2-amd64-ar-eg-preview` | Imagen de contenedor con la configuración regional `ar-EG`. |
| `2.0.2-amd64-ar-kw-preview` | Imagen de contenedor con la configuración regional `ar-KW`. |
| `2.0.2-amd64-ar-qa-preview` | Imagen de contenedor con la configuración regional `ar-QA`. |
| `2.0.2-amd64-ar-sa-preview` | Imagen de contenedor con la configuración regional `ar-SA`. |
| `2.0.2-amd64-ca-es-preview` | Imagen de contenedor con la configuración regional `ca-ES`. |
| `2.0.2-amd64-da-dk-preview` | Imagen de contenedor con la configuración regional `da-DK`. |
| `2.0.2-amd64-de-de-preview` | Imagen de contenedor con la configuración regional `de-DE`. |
| `2.0.2-amd64-en-au-preview` | Imagen de contenedor con la configuración regional `en-AU`. |
| `2.0.2-amd64-en-ca-preview` | Imagen de contenedor con la configuración regional `en-CA`. |
| `2.0.2-amd64-en-gb-preview` | Imagen de contenedor con la configuración regional `en-GB`. |
| `2.0.2-amd64-en-in-preview` | Imagen de contenedor con la configuración regional `en-IN`. |
| `2.0.2-amd64-en-nz-preview` | Imagen de contenedor con la configuración regional `en-NZ`. |
| `2.0.2-amd64-en-us-preview` | Imagen de contenedor con la configuración regional `en-US`. |
| `2.0.2-amd64-es-es-preview` | Imagen de contenedor con la configuración regional `es-ES`. |
| `2.0.2-amd64-es-mx-preview` | Imagen de contenedor con la configuración regional `es-MX`. |
| `2.0.2-amd64-fi-fi-preview` | Imagen de contenedor con la configuración regional `fi-FI`. |
| `2.0.2-amd64-fr-ca-preview` | Imagen de contenedor con la configuración regional `fr-CA`. |
| `2.0.2-amd64-fr-fr-preview` | Imagen de contenedor con la configuración regional `fr-FR`. |
| `2.0.2-amd64-gu-in-preview` | Imagen de contenedor con la configuración regional `gu-IN`. |
| `2.0.2-amd64-hi-in-preview` | Imagen de contenedor con la configuración regional `hi-IN`. |
| `2.0.2-amd64-it-it-preview` | Imagen de contenedor con la configuración regional `it-IT`. |
| `2.0.2-amd64-ja-jp-preview` | Imagen de contenedor con la configuración regional `ja-JP`. |
| `2.0.2-amd64-ko-kr-preview` | Imagen de contenedor con la configuración regional `ko-KR`. |
| `2.0.2-amd64-mr-in-preview` | Imagen de contenedor con la configuración regional `mr-IN`. |
| `2.0.2-amd64-nb-no-preview` | Imagen de contenedor con la configuración regional `nb-NO`. |
| `2.0.2-amd64-nl-nl-preview` | Imagen de contenedor con la configuración regional `nl-NL`. |
| `2.0.2-amd64-pl-pl-preview` | Imagen de contenedor con la configuración regional `pl-PL`. |
| `2.0.2-amd64-pt-br-preview` | Imagen de contenedor con la configuración regional `pt-BR`. |
| `2.0.2-amd64-pt-pt-preview` | Imagen de contenedor con la configuración regional `pt-PT`. |
| `2.0.2-amd64-ru-ru-preview` | Imagen de contenedor con la configuración regional `ru-RU`. |
| `2.0.2-amd64-sv-se-preview` | Imagen de contenedor con la configuración regional `sv-SE`. |
| `2.0.2-amd64-ta-in-preview` | Imagen de contenedor con la configuración regional `ta-IN`. |
| `2.0.2-amd64-te-in-preview` | Imagen de contenedor con la configuración regional `te-IN`. |
| `2.0.2-amd64-th-th-preview` | Imagen de contenedor con la configuración regional `th-TH`. |
| `2.0.2-amd64-tr-tr-preview` | Imagen de contenedor con la configuración regional `tr-TR`. |
| `2.0.2-amd64-zh-cn-preview` | Imagen de contenedor con la configuración regional `zh-CN`. |
| `2.0.2-amd64-zh-hk-preview` | Imagen de contenedor con la configuración regional `zh-HK`. |
| `2.0.2-amd64-zh-tw-preview` | Imagen de contenedor con la configuración regional `zh-TW`. |
| `2.0.1-amd64-ar-ae-preview` | Imagen de contenedor con la configuración regional `ar-AE`. |
| `2.0.1-amd64-ar-eg-preview` | Imagen de contenedor con la configuración regional `ar-EG`. |
| `2.0.1-amd64-ar-kw-preview` | Imagen de contenedor con la configuración regional `ar-KW`. |
| `2.0.1-amd64-ar-qa-preview` | Imagen de contenedor con la configuración regional `ar-QA`. |
| `2.0.1-amd64-ar-sa-preview` | Imagen de contenedor con la configuración regional `ar-SA`. |
| `2.0.1-amd64-ca-es-preview` | Imagen de contenedor con la configuración regional `ca-ES`. |
| `2.0.1-amd64-da-dk-preview` | Imagen de contenedor con la configuración regional `da-DK`. |
| `2.0.1-amd64-de-de-preview` | Imagen de contenedor con la configuración regional `de-DE`. |
| `2.0.1-amd64-en-au-preview` | Imagen de contenedor con la configuración regional `en-AU`. |
| `2.0.1-amd64-en-ca-preview` | Imagen de contenedor con la configuración regional `en-CA`. |
| `2.0.1-amd64-en-gb-preview` | Imagen de contenedor con la configuración regional `en-GB`. |
| `2.0.1-amd64-en-in-preview` | Imagen de contenedor con la configuración regional `en-IN`. |
| `2.0.1-amd64-en-nz-preview` | Imagen de contenedor con la configuración regional `en-NZ`. |
| `2.0.1-amd64-en-us-preview` | Imagen de contenedor con la configuración regional `en-US`. |
| `2.0.1-amd64-es-es-preview` | Imagen de contenedor con la configuración regional `es-ES`. |
| `2.0.1-amd64-es-mx-preview` | Imagen de contenedor con la configuración regional `es-MX`. |
| `2.0.1-amd64-fi-fi-preview` | Imagen de contenedor con la configuración regional `fi-FI`. |
| `2.0.1-amd64-fr-ca-preview` | Imagen de contenedor con la configuración regional `fr-CA`. |
| `2.0.1-amd64-fr-fr-preview` | Imagen de contenedor con la configuración regional `fr-FR`. |
| `2.0.1-amd64-gu-in-preview` | Imagen de contenedor con la configuración regional `gu-IN`. |
| `2.0.1-amd64-hi-in-preview` | Imagen de contenedor con la configuración regional `hi-IN`. |
| `2.0.1-amd64-it-it-preview` | Imagen de contenedor con la configuración regional `it-IT`. |
| `2.0.1-amd64-ja-jp-preview` | Imagen de contenedor con la configuración regional `ja-JP`. |
| `2.0.1-amd64-ko-kr-preview` | Imagen de contenedor con la configuración regional `ko-KR`. |
| `2.0.1-amd64-mr-in-preview` | Imagen de contenedor con la configuración regional `mr-IN`. |
| `2.0.1-amd64-nb-no-preview` | Imagen de contenedor con la configuración regional `nb-NO`. |
| `2.0.1-amd64-nl-nl-preview` | Imagen de contenedor con la configuración regional `nl-NL`. |
| `2.0.1-amd64-pl-pl-preview` | Imagen de contenedor con la configuración regional `pl-PL`. |
| `2.0.1-amd64-pt-br-preview` | Imagen de contenedor con la configuración regional `pt-BR`. |
| `2.0.1-amd64-pt-pt-preview` | Imagen de contenedor con la configuración regional `pt-PT`. |
| `2.0.1-amd64-ru-ru-preview` | Imagen de contenedor con la configuración regional `ru-RU`. |
| `2.0.1-amd64-sv-se-preview` | Imagen de contenedor con la configuración regional `sv-SE`. |
| `2.0.1-amd64-ta-in-preview` | Imagen de contenedor con la configuración regional `ta-IN`. |
| `2.0.1-amd64-te-in-preview` | Imagen de contenedor con la configuración regional `te-IN`. |
| `2.0.1-amd64-th-th-preview` | Imagen de contenedor con la configuración regional `th-TH`. |
| `2.0.1-amd64-tr-tr-preview` | Imagen de contenedor con la configuración regional `tr-TR`. |
| `2.0.1-amd64-zh-cn-preview` | Imagen de contenedor con la configuración regional `zh-CN`. |
| `2.0.1-amd64-zh-hk-preview` | Imagen de contenedor con la configuración regional `zh-HK`. |
| `2.0.1-amd64-zh-tw-preview` | Imagen de contenedor con la configuración regional `zh-TW`. |
| `2.0.0-amd64-ar-eg-preview` | Imagen de contenedor con la configuración regional `ar-EG`. |
| `2.0.0-amd64-ca-es-preview` | Imagen de contenedor con la configuración regional `ca-ES`. |
| `2.0.0-amd64-da-dk-preview` | Imagen de contenedor con la configuración regional `da-DK`. |
| `2.0.0-amd64-de-de-preview` | Imagen de contenedor con la configuración regional `de-DE`. |
| `2.0.0-amd64-en-au-preview` | Imagen de contenedor con la configuración regional `en-AU`. |
| `2.0.0-amd64-en-ca-preview` | Imagen de contenedor con la configuración regional `en-CA`. |
| `2.0.0-amd64-en-gb-preview` | Imagen de contenedor con la configuración regional `en-GB`. |
| `2.0.0-amd64-en-in-preview` | Imagen de contenedor con la configuración regional `en-IN`. |
| `2.0.0-amd64-en-nz-preview` | Imagen de contenedor con la configuración regional `en-NZ`. |
| `2.0.0-amd64-en-us-preview` | Imagen de contenedor con la configuración regional `en-US`. |
| `2.0.0-amd64-es-es-preview` | Imagen de contenedor con la configuración regional `es-ES`. |
| `2.0.0-amd64-es-mx-preview` | Imagen de contenedor con la configuración regional `es-MX`. |
| `2.0.0-amd64-fi-fi-preview` | Imagen de contenedor con la configuración regional `fi-FI`. |
| `2.0.0-amd64-fr-ca-preview` | Imagen de contenedor con la configuración regional `fr-CA`. |
| `2.0.0-amd64-fr-fr-preview` | Imagen de contenedor con la configuración regional `fr-FR`. |
| `2.0.0-amd64-hi-in-preview` | Imagen de contenedor con la configuración regional `hi-IN`. |
| `2.0.0-amd64-it-it-preview` | Imagen de contenedor con la configuración regional `it-IT`. |
| `2.0.0-amd64-ja-jp-preview` | Imagen de contenedor con la configuración regional `ja-JP`. |
| `2.0.0-amd64-ko-kr-preview` | Imagen de contenedor con la configuración regional `ko-KR`. |
| `2.0.0-amd64-nb-no-preview` | Imagen de contenedor con la configuración regional `nb-NO`. |
| `2.0.0-amd64-nl-nl-preview` | Imagen de contenedor con la configuración regional `nl-NL`. |
| `2.0.0-amd64-pl-pl-preview` | Imagen de contenedor con la configuración regional `pl-PL`. |
| `2.0.0-amd64-pt-br-preview` | Imagen de contenedor con la configuración regional `pt-BR`. |
| `2.0.0-amd64-pt-pt-preview` | Imagen de contenedor con la configuración regional `pt-PT`. |
| `2.0.0-amd64-ru-ru-preview` | Imagen de contenedor con la configuración regional `ru-RU`. |
| `2.0.0-amd64-sv-se-preview` | Imagen de contenedor con la configuración regional `sv-SE`. |
| `2.0.0-amd64-th-th-preview` | Imagen de contenedor con la configuración regional `th-TH`. |
| `2.0.0-amd64-tr-tr-preview` | Imagen de contenedor con la configuración regional `tr-TR`. |
| `2.0.0-amd64-zh-cn-preview` | Imagen de contenedor con la configuración regional `zh-CN`. |
| `2.0.0-amd64-zh-hk-preview` | Imagen de contenedor con la configuración regional `zh-HK`. |
| `2.0.0-amd64-zh-tw-preview` | Imagen de contenedor con la configuración regional `zh-TW`. |
| `1.2.0-amd64-de-de-preview` | Imagen de contenedor con la configuración regional `de-DE`. |
| `1.2.0-amd64-en-au-preview` | Imagen de contenedor con la configuración regional `en-AU`. |
| `1.2.0-amd64-en-ca-preview` | Imagen de contenedor con la configuración regional `en-CA`. |
| `1.2.0-amd64-en-gb-preview` | Imagen de contenedor con la configuración regional `en-GB`. |
| `1.2.0-amd64-en-in-preview` | Imagen de contenedor con la configuración regional `en-IN`. |
| `1.2.0-amd64-en-us-preview` | Imagen de contenedor con la configuración regional `en-US`. |
| `1.2.0-amd64-es-es-preview` | Imagen de contenedor con la configuración regional `es-ES`. |
| `1.2.0-amd64-es-mx-preview` | Imagen de contenedor con la configuración regional `es-MX`. |
| `1.2.0-amd64-fr-ca-preview` | Imagen de contenedor con la configuración regional `fr-CA`. |
| `1.2.0-amd64-fr-fr-preview` | Imagen de contenedor con la configuración regional `fr-FR`. |
| `1.2.0-amd64-it-it-preview` | Imagen de contenedor con la configuración regional `it-IT`. |
| `1.2.0-amd64-ja-jp-preview` | Imagen de contenedor con la configuración regional `ja-JP`. |
| `1.2.0-amd64-pt-br-preview` | Imagen de contenedor con la configuración regional `pt-BR`. |
| `1.2.0-amd64-zh-cn-preview` | Imagen de contenedor con la configuración regional `zh-CN`. |
| `1.1.3-amd64-de-de-preview` | Imagen de contenedor con la configuración regional `de-DE`. |
| `1.1.3-amd64-en-au-preview` | Imagen de contenedor con la configuración regional `en-AU`. |
| `1.1.3-amd64-en-ca-preview` | Imagen de contenedor con la configuración regional `en-CA`. |
| `1.1.3-amd64-en-gb-preview` | Imagen de contenedor con la configuración regional `en-GB`. |
| `1.1.3-amd64-en-in-preview` | Imagen de contenedor con la configuración regional `en-IN`. |
| `1.1.3-amd64-en-us-preview` | Imagen de contenedor con la configuración regional `en-US`. |
| `1.1.3-amd64-es-es-preview` | Imagen de contenedor con la configuración regional `es-ES`. |
| `1.1.3-amd64-es-mx-preview` | Imagen de contenedor con la configuración regional `es-MX`. |
| `1.1.3-amd64-fr-ca-preview` | Imagen de contenedor con la configuración regional `fr-CA`. |
| `1.1.3-amd64-fr-fr-preview` | Imagen de contenedor con la configuración regional `fr-FR`. |
| `1.1.3-amd64-it-it-preview` | Imagen de contenedor con la configuración regional `it-IT`. |
| `1.1.3-amd64-ja-jp-preview` | Imagen de contenedor con la configuración regional `ja-JP`. |
| `1.1.3-amd64-pt-br-preview` | Imagen de contenedor con la configuración regional `pt-BR`. |
| `1.1.3-amd64-zh-cn-preview` | Imagen de contenedor con la configuración regional `zh-CN`. |
| `1.1.2-amd64-de-de-preview` | Imagen de contenedor con la configuración regional `de-DE`. |
| `1.1.2-amd64-en-au-preview` | Imagen de contenedor con la configuración regional `en-AU`. |
| `1.1.2-amd64-en-ca-preview` | Imagen de contenedor con la configuración regional `en-CA`. |
| `1.1.2-amd64-en-gb-preview` | Imagen de contenedor con la configuración regional `en-GB`. |
| `1.1.2-amd64-en-in-preview` | Imagen de contenedor con la configuración regional `en-IN`. |
| `1.1.2-amd64-en-us-preview` | Imagen de contenedor con la configuración regional `en-US`. |
| `1.1.2-amd64-es-es-preview` | Imagen de contenedor con la configuración regional `es-ES`. |
| `1.1.2-amd64-es-mx-preview` | Imagen de contenedor con la configuración regional `es-MX`. |
| `1.1.2-amd64-fr-ca-preview` | Imagen de contenedor con la configuración regional `fr-CA`. |
| `1.1.2-amd64-fr-fr-preview` | Imagen de contenedor con la configuración regional `fr-FR`. |
| `1.1.2-amd64-it-it-preview` | Imagen de contenedor con la configuración regional `it-IT`. |
| `1.1.2-amd64-ja-jp-preview` | Imagen de contenedor con la configuración regional `ja-JP`. |
| `1.1.2-amd64-pt-br-preview` | Imagen de contenedor con la configuración regional `pt-BR`. |
| `1.1.2-amd64-zh-cn-preview` | Imagen de contenedor con la configuración regional `zh-CN`. |
| `1.1.1-amd64-de-de-preview` | Imagen de contenedor con la configuración regional `de-DE`. |
| `1.1.1-amd64-en-au-preview` | Imagen de contenedor con la configuración regional `en-AU`. |
| `1.1.1-amd64-en-ca-preview` | Imagen de contenedor con la configuración regional `en-CA`. |
| `1.1.1-amd64-en-gb-preview` | Imagen de contenedor con la configuración regional `en-GB`. |
| `1.1.1-amd64-en-in-preview` | Imagen de contenedor con la configuración regional `en-IN`. |
| `1.1.1-amd64-en-us-preview` | Imagen de contenedor con la configuración regional `en-US`. |
| `1.1.1-amd64-es-es-preview` | Imagen de contenedor con la configuración regional `es-ES`. |
| `1.1.1-amd64-es-mx-preview` | Imagen de contenedor con la configuración regional `es-MX`. |
| `1.1.1-amd64-fr-ca-preview` | Imagen de contenedor con la configuración regional `fr-CA`. |
| `1.1.1-amd64-fr-fr-preview` | Imagen de contenedor con la configuración regional `fr-FR`. |
| `1.1.1-amd64-it-it-preview` | Imagen de contenedor con la configuración regional `it-IT`. |
| `1.1.1-amd64-ja-jp-preview` | Imagen de contenedor con la configuración regional `ja-JP`. |
| `1.1.1-amd64-pt-br-preview` | Imagen de contenedor con la configuración regional `pt-BR`. |
| `1.1.1-amd64-zh-cn-preview` | Imagen de contenedor con la configuración regional `zh-CN`. |
| `1.1.0-amd64-de-de-preview` | Imagen de contenedor con la configuración regional `de-DE`. |
| `1.1.0-amd64-en-au-preview` | Imagen de contenedor con la configuración regional `en-AU`. |
| `1.1.0-amd64-en-ca-preview` | Imagen de contenedor con la configuración regional `en-CA`. |
| `1.1.0-amd64-en-gb-preview` | Imagen de contenedor con la configuración regional `en-GB`. |
| `1.1.0-amd64-en-in-preview` | Imagen de contenedor con la configuración regional `en-IN`. |
| `1.1.0-amd64-en-us-preview` | Imagen de contenedor con la configuración regional `en-US`. |
| `1.1.0-amd64-es-es-preview` | Imagen de contenedor con la configuración regional `es-ES`. |
| `1.1.0-amd64-es-mx-preview` | Imagen de contenedor con la configuración regional `es-MX`. |
| `1.1.0-amd64-fr-ca-preview` | Imagen de contenedor con la configuración regional `fr-CA`. |
| `1.1.0-amd64-fr-fr-preview` | Imagen de contenedor con la configuración regional `fr-FR`. |
| `1.1.0-amd64-it-it-preview` | Imagen de contenedor con la configuración regional `it-IT`. |
| `1.1.0-amd64-ja-jp-preview` | Imagen de contenedor con la configuración regional `ja-JP`. |
| `1.1.0-amd64-pt-br-preview` | Imagen de contenedor con la configuración regional `pt-BR`. |
| `1.1.0-amd64-zh-cn-preview` | Imagen de contenedor con la configuración regional `zh-CN`. |
| `1.0.0-amd64-de-de-preview` | Imagen de contenedor con la configuración regional `de-DE`. |
| `1.0.0-amd64-en-au-preview` | Imagen de contenedor con la configuración regional `en-AU`. |
| `1.0.0-amd64-en-ca-preview` | Imagen de contenedor con la configuración regional `en-CA`. |
| `1.0.0-amd64-en-gb-preview` | Imagen de contenedor con la configuración regional `en-GB`. |
| `1.0.0-amd64-en-in-preview` | Imagen de contenedor con la configuración regional `en-IN`. |
| `1.0.0-amd64-en-us-preview` | Imagen de contenedor con la configuración regional `en-US`. |
| `1.0.0-amd64-es-es-preview` | Imagen de contenedor con la configuración regional `es-ES`. |
| `1.0.0-amd64-es-mx-preview` | Imagen de contenedor con la configuración regional `es-MX`. |
| `1.0.0-amd64-fr-ca-preview` | Imagen de contenedor con la configuración regional `fr-CA`. |
| `1.0.0-amd64-fr-fr-preview` | Imagen de contenedor con la configuración regional `fr-FR`. |
| `1.0.0-amd64-it-it-preview` | Imagen de contenedor con la configuración regional `it-IT`. |
| `1.0.0-amd64-ja-jp-preview` | Imagen de contenedor con la configuración regional `ja-JP`. |
| `1.0.0-amd64-pt-br-preview` | Imagen de contenedor con la configuración regional `pt-BR`. |
| `1.0.0-amd64-zh-cn-preview` | Imagen de contenedor con la configuración regional `zh-CN`. |

## <a name="text-to-speech"></a>Texto a voz

La imagen de contenedor [Conversión de texto a voz][sp-tts] se puede encontrar en el registro de contenedor `containerpreview.azurecr.io`. Reside en el repositorio `microsoft` y se denomina `cognitive-services-text-to-speech`. El nombre completo de la imagen de contenedor es `containerpreview.azurecr.io/microsoft/cognitive-services-text-to-speech`.

Esta imagen de contenedor tiene disponibles las siguientes etiquetas:

| Etiquetas de imagen                                  | Notas                                                                      |
|---------------------------------------------|:---------------------------------------------------------------------------|
| `latest`                                    | Imagen de contenedor con la configuración regional `en-US` y voz `en-US-JessaRUS`.        |
| `1.3.0-amd64-ar-eg-hoda-preview`            | Imagen de contenedor con la configuración regional `ar-EG` y voz `ar-EG-Hoda`.            |
| `1.3.0-amd64-ar-sa-naayf-preview`           | Imagen de contenedor con la configuración regional `ar-SA` y voz `ar-SA-Naayf`.           |
| `1.3.0-amd64-bg-bg-ivan-preview`            | Imagen de contenedor con la configuración regional `bg-BG` y voz `bg-BG-Ivan`.            |
| `1.3.0-amd64-ca-es-herenarus-preview`       | Imagen de contenedor con la configuración regional `ca-ES` y voz `ca-ES-HerenaRUS`.       |
| `1.3.0-amd64-cs-cz-jakub-preview`           | Imagen de contenedor con la configuración regional `cs-CZ` y voz `cs-CZ-Jakub`.           |
| `1.3.0-amd64-da-dk-hellerus-preview`        | Imagen de contenedor con la configuración regional `da-DK` y voz `da-DK-HelleRUS`.        |
| `1.3.0-amd64-de-at-michael-preview`         | Imagen de contenedor con la configuración regional `de-AT` y voz `de-AT-Michael`.         |
| `1.3.0-amd64-de-ch-karsten-preview`         | Imagen de contenedor con la configuración regional `de-CH` y voz `de-CH-Karsten`.         |
| `1.3.0-amd64-de-de-hedda-preview`           | Imagen de contenedor con la configuración regional `de-DE` y voz `de-DE-Hedda`.           |
| `1.3.0-amd64-de-de-heddarus-preview`        | Imagen de contenedor con la configuración regional `de-DE` y voz `de-DE-Hedda`.           |
| `1.3.0-amd64-de-de-heddarus-preview`        | Imagen de contenedor con la configuración regional `de-DE` y voz `de-DE-HeddaRUS`.        |
| `1.3.0-amd64-de-de-stefan-apollo-preview`   | Imagen de contenedor con la configuración regional `de-DE` y voz `de-DE-Stefan-Apollo`.   |
| `1.3.0-amd64-el-gr-stefanos-preview`        | Imagen de contenedor con la configuración regional `el-GR` y voz `el-GR-Stefanos`.        |
| `1.3.0-amd64-en-au-catherine-preview`       | Imagen de contenedor con la configuración regional `en-AU` y voz `en-AU-Catherine`.       |
| `1.3.0-amd64-en-au-hayleyrus-preview`       | Imagen de contenedor con la configuración regional `en-AU` y voz `en-AU-HayleyRUS`.       |
| `1.3.0-amd64-en-ca-heatherrus-preview`      | Imagen de contenedor con la configuración regional `en-CA` y voz `en-CA-HeatherRUS`.      |
| `1.3.0-amd64-en-ca-linda-preview`           | Imagen de contenedor con la configuración regional `en-CA` y voz `en-CA-Linda`.           |
| `1.3.0-amd64-en-gb-george-apollo-preview`   | Imagen de contenedor con la configuración regional `en-GB` y voz `en-GB-George-Apollo`.   |
| `1.3.0-amd64-en-gb-hazelrus-preview`        | Imagen de contenedor con la configuración regional `en-GB` y voz `en-GB-HazelRUS`.        |
| `1.3.0-amd64-en-gb-susan-apollo-preview`    | Imagen de contenedor con la configuración regional `en-GB` y voz `en-GB-Susan-Apollo`.    |
| `1.3.0-amd64-en-ie-sean-preview`            | Imagen de contenedor con la configuración regional `en-IE` y voz `en-IE-Sean`.            |
| `1.3.0-amd64-en-in-heera-apollo-preview`    | Imagen de contenedor con la configuración regional `en-IN` y voz `en-IN-Heera-Apollo`.    |
| `1.3.0-amd64-en-in-priyarus-preview`        | Imagen de contenedor con la configuración regional `en-IN` y voz `en-IN-PriyaRUS`.        |
| `1.3.0-amd64-en-in-ravi-apollo-preview`     | Imagen de contenedor con la configuración regional `en-IN` y voz `en-IN-Ravi-Apollo`.     |
| `1.3.0-amd64-en-us-benjaminrus-preview`     | Imagen de contenedor con la configuración regional `en-US` y voz `en-US-BenjaminRUS`.     |
| `1.3.0-amd64-en-us-guy24krus-preview`       | Imagen de contenedor con la configuración regional `en-US` y voz `en-US-Guy24kRUS`.       |
| `1.3.0-amd64-en-us-jessa24krus-preview`     | Imagen de contenedor con la configuración regional `en-US` y voz `en-US-Jessa24kRUS`.     |
| `1.3.0-amd64-en-us-jessarus-preview`        | Imagen de contenedor con la configuración regional `en-US` y voz `en-US-JessaRUS`.        |
| `1.3.0-amd64-en-us-zirarus-preview`         | Imagen de contenedor con la configuración regional `en-US` y voz `en-US-ZiraRUS`.         |
| `1.3.0-amd64-es-es-helenarus-preview`       | Imagen de contenedor con la configuración regional `es-ES` y voz `es-ES-HelenaRUS`.       |
| `1.3.0-amd64-es-es-laura-apollo-preview`    | Imagen de contenedor con la configuración regional `es-ES` y voz `es-ES-Laura-Apollo`.    |
| `1.3.0-amd64-es-es-pablo-apollo-preview`    | Imagen de contenedor con la configuración regional `es-ES` y voz `es-ES-Pablo-Apollo`.    |
| `1.3.0-amd64-es-mx-hildarus-preview`        | Imagen de contenedor con la configuración regional `es-MX` y voz `es-MX-HildaRUS`.        |
| `1.3.0-amd64-es-mx-raul-apollo-preview`     | Imagen de contenedor con la configuración regional `es-MX` y voz `es-MX-Raul-Apollo`.     |
| `1.3.0-amd64-fi-fi-heidirus-preview`        | Imagen de contenedor con la configuración regional `fi-FI` y voz `fi-FI-HeidiRUS`.        |
| `1.3.0-amd64-fr-ca-caroline-preview`        | Imagen de contenedor con la configuración regional `fr-CA` y voz `fr-CA-Caroline`.        |
| `1.3.0-amd64-fr-ca-harmonierus-preview`     | Imagen de contenedor con la configuración regional `fr-CA` y voz `fr-CA-HarmonieRUS`.     |
| `1.3.0-amd64-fr-ch-guillaume-preview`       | Imagen de contenedor con la configuración regional `fr-CH` y voz `fr-CH-Guillaume`.       |
| `1.3.0-amd64-fr-fr-hortenserus-preview`     | Imagen de contenedor con la configuración regional `fr-FR` y voz `fr-FR-HortenseRUS`.     |
| `1.3.0-amd64-fr-fr-julie-apollo-preview`    | Imagen de contenedor con la configuración regional `fr-FR` y voz `fr-FR-Julie-Apollo`.    |
| `1.3.0-amd64-fr-fr-paul-apollo-preview`     | Imagen de contenedor con la configuración regional `fr-FR` y voz `fr-FR-Paul-Apollo`.     |
| `1.3.0-amd64-he-il-asaf-preview`            | Imagen de contenedor con la configuración regional `he-IL` y voz `he-IL-Asaf`.            |
| `1.3.0-amd64-hi-in-hemant-preview`          | Imagen de contenedor con la configuración regional `hi-IN` y voz `hi-IN-Hemant`.          |
| `1.3.0-amd64-hi-in-kalpana-apollo-preview`  | Imagen de contenedor con la configuración regional `hi-IN` y voz `hi-IN-Kalpana-Apollo`.  |
| `1.3.0-amd64-hi-in-kalpana-apollo-preview`  | Imagen de contenedor con la configuración regional `hi-IN` y voz `hi-IN-Kalpana`.         |
| `1.3.0-amd64-hi-in-kalpana-preview`         | Imagen de contenedor con la configuración regional `hi-IN` y voz `hi-IN-Kalpana`.         |
| `1.3.0-amd64-hr-hr-matej-preview`           | Imagen de contenedor con la configuración regional `hr-HR` y voz `hr-HR-Matej`.           |
| `1.3.0-amd64-hu-hu-szabolcs-preview`        | Imagen de contenedor con la configuración regional `hu-HU` y voz `hu-HU-Szabolcs`.        |
| `1.3.0-amd64-id-id-andika-preview`          | Imagen de contenedor con la configuración regional `id-ID` y voz `id-ID-Andika`.          |
| `1.3.0-amd64-it-it-cosimo-apollo-preview`   | Imagen de contenedor con la configuración regional `it-IT` y voz `it-IT-Cosimo-Apollo`.   |
| `1.3.0-amd64-it-it-luciarus-preview`        | Imagen de contenedor con la configuración regional `it-IT` y voz `it-IT-LuciaRUS`.        |
| `1.3.0-amd64-ja-jp-ayumi-apollo-preview`    | Imagen de contenedor con la configuración regional `ja-JP` y voz `ja-JP-Ayumi-Apollo`.    |
| `1.3.0-amd64-ja-jp-harukarus-preview`       | Imagen de contenedor con la configuración regional `ja-JP` y voz `ja-JP-HarukaRUS`.       |
| `1.3.0-amd64-ja-jp-ichiro-apollo-preview`   | Imagen de contenedor con la configuración regional `ja-JP` y voz `ja-JP-Ichiro-Apollo`.   |
| `1.3.0-amd64-ko-kr-heamirus-preview`        | Imagen de contenedor con la configuración regional `ko-KR` y voz `ko-KR-HeamiRUS`.        |
| `1.3.0-amd64-ms-my-rizwan-preview`          | Imagen de contenedor con la configuración regional `ms-MY` y voz `ms-MY-Rizwan`.          |
| `1.3.0-amd64-nb-no-huldarus-preview`        | Imagen de contenedor con la configuración regional `nb-NO` y voz `nb-NO-HuldaRUS`.        |
| `1.3.0-amd64-nl-nl-hannarus-preview`        | Imagen de contenedor con la configuración regional `nl-NL` y voz `nl-NL-HannaRUS`.        |
| `1.3.0-amd64-pl-pl-paulinarus-preview`      | Imagen de contenedor con la configuración regional `pl-PL` y voz `pl-PL-PaulinaRUS`.      |
| `1.3.0-amd64-pt-br-daniel-apollo-preview`   | Imagen de contenedor con la configuración regional `pt-BR` y voz `pt-BR-Daniel-Apollo`.   |
| `1.3.0-amd64-pt-br-heloisarus-preview`      | Imagen de contenedor con la configuración regional `pt-BR` y voz `pt-BR-HeloisaRUS`.      |
| `1.3.0-amd64-pt-pt-heliarus-preview`        | Imagen de contenedor con la configuración regional `pt-PT` y voz `pt-PT-HeliaRUS`.        |
| `1.3.0-amd64-ro-ro-andrei-preview`          | Imagen de contenedor con la configuración regional `ro-RO` y voz `ro-RO-Andrei`.          |
| `1.3.0-amd64-ru-ru-ekaterinarus-preview`    | Imagen de contenedor con la configuración regional `ru-RU` y voz `ru-RU-EkaterinaRUS`.    |
| `1.3.0-amd64-ru-ru-irina-apollo-preview`    | Imagen de contenedor con la configuración regional `ru-RU` y voz `ru-RU-Irina-Apollo`.    |
| `1.3.0-amd64-ru-ru-pavel-apollo-preview`    | Imagen de contenedor con la configuración regional `ru-RU` y voz `ru-RU-Pavel-Apollo`.    |
| `1.3.0-amd64-sk-sk-filip-preview`           | Imagen de contenedor con la configuración regional `sk-SK` y voz `sk-SK-Filip`.           |
| `1.3.0-amd64-sl-si-lado-preview`            | Imagen de contenedor con la configuración regional `sl-SI` y voz `sl-SI-Lado`.            |
| `1.3.0-amd64-sv-se-hedvigrus-preview`       | Imagen de contenedor con la configuración regional `sv-SE` y voz `sv-SE-HedvigRUS`.       |
| `1.3.0-amd64-ta-in-valluvar-preview`        | Imagen de contenedor con la configuración regional `ta-IN` y voz `ta-IN-Valluvar`.        |
| `1.3.0-amd64-te-in-chitra-preview`          | Imagen de contenedor con la configuración regional `te-IN` y voz `te-IN-Chitra`.          |
| `1.3.0-amd64-th-th-pattara-preview`         | Imagen de contenedor con la configuración regional `th-TH` y voz `th-TH-Pattara`.         |
| `1.3.0-amd64-tr-tr-sedarus-preview`         | Imagen de contenedor con la configuración regional `tr-TR` y voz `tr-TR-SedaRUS`.         |
| `1.3.0-amd64-vi-vn-an-preview`              | Imagen de contenedor con la configuración regional `vi-VN` y voz `vi-VN-An`.              |
| `1.3.0-amd64-zh-cn-huihuirus-preview`       | Imagen de contenedor con la configuración regional `zh-CN` y voz `zh-CN-HuihuiRUS`.       |
| `1.3.0-amd64-zh-cn-kangkang-apollo-preview` | Imagen de contenedor con la configuración regional `zh-CN` y voz `zh-CN-Kangkang-Apollo`. |
| `1.3.0-amd64-zh-cn-yaoyao-apollo-preview`   | Imagen de contenedor con la configuración regional `zh-CN` y voz `zh-CN-Yaoyao-Apollo`.   |
| `1.3.0-amd64-zh-hk-danny-apollo-preview`    | Imagen de contenedor con la configuración regional `zh-HK` y voz `zh-HK-Danny-Apollo`.    |
| `1.3.0-amd64-zh-hk-tracy-apollo-preview`    | Imagen de contenedor con la configuración regional `zh-HK` y voz `zh-HK-Tracy-Apollo`.    |
| `1.3.0-amd64-zh-hk-tracyrus-preview`        | Imagen de contenedor con la configuración regional `zh-HK` y voz `zh-HK-TracyRUS`.        |
| `1.3.0-amd64-zh-tw-hanhanrus-preview`       | Imagen de contenedor con la configuración regional `zh-TW` y voz `zh-TW-HanHanRUS`.       |
| `1.3.0-amd64-zh-tw-yating-apollo-preview`   | Imagen de contenedor con la configuración regional `zh-TW` y voz `zh-TW-Yating-Apollo`.   |
| `1.3.0-amd64-zh-tw-zhiwei-apollo-preview`   | Imagen de contenedor con la configuración regional `zh-TW` y voz `zh-TW-Zhiwei-Apollo`.   |
| `1.2.0-amd64-de-de-heddarus-preview`        | Imagen de contenedor con la configuración regional `de-DE` y voz `de-DE-Hedda`.           |
| `1.2.0-amd64-de-de-heddarus-preview`        | Imagen de contenedor con la configuración regional `de-DE` y voz `de-DE-HeddaRUS`.        |
| `1.2.0-amd64-de-de-stefan-apollo-preview`   | Imagen de contenedor con la configuración regional `de-DE` y voz `de-DE-Stefan-Apollo`.   |
| `1.2.0-amd64-en-au-catherine-preview`       | Imagen de contenedor con la configuración regional `en-AU` y voz `en-AU-Catherine`.       |
| `1.2.0-amd64-en-au-hayleyrus-preview`       | Imagen de contenedor con la configuración regional `en-AU` y voz `en-AU-HayleyRUS`.       |
| `1.2.0-amd64-en-gb-george-apollo-preview`   | Imagen de contenedor con la configuración regional `en-GB` y voz `en-GB-George-Apollo`.   |
| `1.2.0-amd64-en-gb-hazelrus-preview`        | Imagen de contenedor con la configuración regional `en-GB` y voz `en-GB-HazelRUS`.        |
| `1.2.0-amd64-en-gb-susan-apollo-preview`    | Imagen de contenedor con la configuración regional `en-GB` y voz `en-GB-Susan-Apollo`.    |
| `1.2.0-amd64-en-in-heera-apollo-preview`    | Imagen de contenedor con la configuración regional `en-IN` y voz `en-IN-Heera-Apollo`.    |
| `1.2.0-amd64-en-in-priyarus-preview`        | Imagen de contenedor con la configuración regional `en-IN` y voz `en-IN-PriyaRUS`.        |
| `1.2.0-amd64-en-in-ravi-apollo-preview`     | Imagen de contenedor con la configuración regional `en-IN` y voz `en-IN-Ravi-Apollo`.     |
| `1.2.0-amd64-en-us-benjaminrus-preview`     | Imagen de contenedor con la configuración regional `en-US` y voz `en-US-BenjaminRUS`.     |
| `1.2.0-amd64-en-us-guy24krus-preview`       | Imagen de contenedor con la configuración regional `en-US` y voz `en-US-Guy24kRUS`.       |
| `1.2.0-amd64-en-us-jessa24krus-preview`     | Imagen de contenedor con la configuración regional `en-US` y voz `en-US-Jessa24kRUS`.     |
| `1.2.0-amd64-en-us-jessarus-preview`        | Imagen de contenedor con la configuración regional `en-US` y voz `en-US-JessaRUS`.        |
| `1.2.0-amd64-en-us-zirarus-preview`         | Imagen de contenedor con la configuración regional `en-US` y voz `en-US-ZiraRUS`.         |
| `1.2.0-amd64-es-es-helenarus-preview`       | Imagen de contenedor con la configuración regional `es-ES` y voz `es-ES-HelenaRUS`.       |
| `1.2.0-amd64-es-es-laura-apollo-preview`    | Imagen de contenedor con la configuración regional `es-ES` y voz `es-ES-Laura-Apollo`.    |
| `1.2.0-amd64-es-es-pablo-apollo-preview`    | Imagen de contenedor con la configuración regional `es-ES` y voz `es-ES-Pablo-Apollo`.    |
| `1.2.0-amd64-es-mx-hildarus-preview`        | Imagen de contenedor con la configuración regional `es-MX` y voz `es-MX-HildaRUS`.        |
| `1.2.0-amd64-es-mx-raul-apollo-preview`     | Imagen de contenedor con la configuración regional `es-MX` y voz `es-MX-Raul-Apollo`.     |
| `1.2.0-amd64-fr-ca-caroline-preview`        | Imagen de contenedor con la configuración regional `fr-CA` y voz `fr-CA-Caroline`.        |
| `1.2.0-amd64-fr-ca-harmonierus-preview`     | Imagen de contenedor con la configuración regional `fr-CA` y voz `fr-CA-HarmonieRUS`.     |
| `1.2.0-amd64-fr-fr-hortenserus-preview`     | Imagen de contenedor con la configuración regional `fr-FR` y voz `fr-FR-HortenseRUS`.     |
| `1.2.0-amd64-fr-fr-julie-apollo-preview`    | Imagen de contenedor con la configuración regional `fr-FR` y voz `fr-FR-Julie-Apollo`.    |
| `1.2.0-amd64-fr-fr-paul-apollo-preview`     | Imagen de contenedor con la configuración regional `fr-FR` y voz `fr-FR-Paul-Apollo`.     |
| `1.2.0-amd64-it-it-cosimo-apollo-preview`   | Imagen de contenedor con la configuración regional `it-IT` y voz `it-IT-Cosimo-Apollo`.   |
| `1.2.0-amd64-it-it-luciarus-preview`        | Imagen de contenedor con la configuración regional `it-IT` y voz `it-IT-LuciaRUS`.        |
| `1.2.0-amd64-ja-jp-ayumi-apollo-preview`    | Imagen de contenedor con la configuración regional `ja-JP` y voz `ja-JP-Ayumi-Apollo`.    |
| `1.2.0-amd64-ja-jp-harukarus-preview`       | Imagen de contenedor con la configuración regional `ja-JP` y voz `ja-JP-HarukaRUS`.       |
| `1.2.0-amd64-ja-jp-ichiro-apollo-preview`   | Imagen de contenedor con la configuración regional `ja-JP` y voz `ja-JP-Ichiro-Apollo`.   |
| `1.2.0-amd64-ko-kr-heamirus-preview`        | Imagen de contenedor con la configuración regional `ko-KR` y voz `ko-KR-HeamiRUS`.        |
| `1.2.0-amd64-pt-br-daniel-apollo-preview`   | Imagen de contenedor con la configuración regional `pt-BR` y voz `pt-BR-Daniel-Apollo`.   |
| `1.2.0-amd64-pt-br-heloisarus-preview`      | Imagen de contenedor con la configuración regional `pt-BR` y voz `pt-BR-HeloisaRUS`.      |
| `1.2.0-amd64-zh-cn-huihuirus-preview`       | Imagen de contenedor con la configuración regional `zh-CN` y voz `zh-CN-HuihuiRUS`.       |
| `1.2.0-amd64-zh-cn-kangkang-apollo-preview` | Imagen de contenedor con la configuración regional `zh-CN` y voz `zh-CN-Kangkang-Apollo`. |
| `1.2.0-amd64-zh-cn-yaoyao-apollo-preview`   | Imagen de contenedor con la configuración regional `zh-CN` y voz `zh-CN-Yaoyao-Apollo`.   |
| `1.1.0-amd64-de-de-hedda-preview`           | Imagen de contenedor con la configuración regional `de-DE` y voz `de-DE-Hedda`.           |
| `1.1.0-amd64-de-de-heddarus-preview`        | Imagen de contenedor con la configuración regional `de-DE` y voz `de-DE-Hedda`.           |
| `1.1.0-amd64-de-de-heddarus-preview`        | Imagen de contenedor con la configuración regional `de-DE` y voz `de-DE-HeddaRUS`.        |
| `1.1.0-amd64-de-de-stefan-apollo-preview`   | Imagen de contenedor con la configuración regional `de-DE` y voz `de-DE-Stefan-Apollo`.   |
| `1.1.0-amd64-en-au-catherine-preview`       | Imagen de contenedor con la configuración regional `en-AU` y voz `en-AU-Catherine`.       |
| `1.1.0-amd64-en-au-hayleyrus-preview`       | Imagen de contenedor con la configuración regional `en-AU` y voz `en-AU-HayleyRUS`.       |
| `1.1.0-amd64-en-gb-george-apollo-preview`   | Imagen de contenedor con la configuración regional `en-GB` y voz `en-GB-George-Apollo`.   |
| `1.1.0-amd64-en-gb-hazelrus-preview`        | Imagen de contenedor con la configuración regional `en-GB` y voz `en-GB-HazelRUS`.        |
| `1.1.0-amd64-en-gb-susan-apollo-preview`    | Imagen de contenedor con la configuración regional `en-GB` y voz `en-GB-Susan-Apollo`.    |
| `1.1.0-amd64-en-in-heera-apollo-preview`    | Imagen de contenedor con la configuración regional `en-IN` y voz `en-IN-Heera-Apollo`.    |
| `1.1.0-amd64-en-in-priyarus-preview`        | Imagen de contenedor con la configuración regional `en-IN` y voz `en-IN-PriyaRUS`.        |
| `1.1.0-amd64-en-in-ravi-apollo-preview`     | Imagen de contenedor con la configuración regional `en-IN` y voz `en-IN-Ravi-Apollo`.     |
| `1.1.0-amd64-en-us-benjaminrus-preview`     | Imagen de contenedor con la configuración regional `en-US` y voz `en-US-BenjaminRUS`.     |
| `1.1.0-amd64-en-us-guy24krus-preview`       | Imagen de contenedor con la configuración regional `en-US` y voz `en-US-Guy24kRUS`.       |
| `1.1.0-amd64-en-us-jessa24krus-preview`     | Imagen de contenedor con la configuración regional `en-US` y voz `en-US-Jessa24kRUS`.     |
| `1.1.0-amd64-en-us-jessarus-preview`        | Imagen de contenedor con la configuración regional `en-US` y voz `en-US-JessaRUS`.        |
| `1.1.0-amd64-en-us-zirarus-preview`         | Imagen de contenedor con la configuración regional `en-US` y voz `en-US-ZiraRUS`.         |
| `1.1.0-amd64-es-es-helenarus-preview`       | Imagen de contenedor con la configuración regional `es-ES` y voz `es-ES-HelenaRUS`.       |
| `1.1.0-amd64-es-es-laura-apollo-preview`    | Imagen de contenedor con la configuración regional `es-ES` y voz `es-ES-Laura-Apollo`.    |
| `1.1.0-amd64-es-es-pablo-apollo-preview`    | Imagen de contenedor con la configuración regional `es-ES` y voz `es-ES-Pablo-Apollo`.    |
| `1.1.0-amd64-es-mx-hildarus-preview`        | Imagen de contenedor con la configuración regional `es-MX` y voz `es-MX-HildaRUS`.        |
| `1.1.0-amd64-es-mx-raul-apollo-preview`     | Imagen de contenedor con la configuración regional `es-MX` y voz `es-MX-Raul-Apollo`.     |
| `1.1.0-amd64-fr-ca-caroline-preview`        | Imagen de contenedor con la configuración regional `fr-CA` y voz `fr-CA-Caroline`.        |
| `1.1.0-amd64-fr-ca-harmonierus-preview`     | Imagen de contenedor con la configuración regional `fr-CA` y voz `fr-CA-HarmonieRUS`.     |
| `1.1.0-amd64-fr-fr-hortenserus-preview`     | Imagen de contenedor con la configuración regional `fr-FR` y voz `fr-FR-HortenseRUS`.     |
| `1.1.0-amd64-fr-fr-julie-apollo-preview`    | Imagen de contenedor con la configuración regional `fr-FR` y voz `fr-FR-Julie-Apollo`.    |
| `1.1.0-amd64-fr-fr-paul-apollo-preview`     | Imagen de contenedor con la configuración regional `fr-FR` y voz `fr-FR-Paul-Apollo`.     |
| `1.1.0-amd64-it-it-cosimo-apollo-preview`   | Imagen de contenedor con la configuración regional `it-IT` y voz `it-IT-Cosimo-Apollo`.   |
| `1.1.0-amd64-it-it-luciarus-preview`        | Imagen de contenedor con la configuración regional `it-IT` y voz `it-IT-LuciaRUS`.        |
| `1.1.0-amd64-ja-jp-ayumi-apollo-preview`    | Imagen de contenedor con la configuración regional `ja-JP` y voz `ja-JP-Ayumi-Apollo`.    |
| `1.1.0-amd64-ja-jp-harukarus-preview`       | Imagen de contenedor con la configuración regional `ja-JP` y voz `ja-JP-HarukaRUS`.       |
| `1.1.0-amd64-ja-jp-ichiro-apollo-preview`   | Imagen de contenedor con la configuración regional `ja-JP` y voz `ja-JP-Ichiro-Apollo`.   |
| `1.1.0-amd64-ko-kr-heamirus-preview`        | Imagen de contenedor con la configuración regional `ko-KR` y voz `ko-KR-HeamiRUS`.        |
| `1.1.0-amd64-pt-br-daniel-apollo-preview`   | Imagen de contenedor con la configuración regional `pt-BR` y voz `pt-BR-Daniel-Apollo`.   |
| `1.1.0-amd64-pt-br-heloisarus-preview`      | Imagen de contenedor con la configuración regional `pt-BR` y voz `pt-BR-HeloisaRUS`.      |
| `1.1.0-amd64-zh-cn-huihuirus-preview`       | Imagen de contenedor con la configuración regional `zh-CN` y voz `zh-CN-HuihuiRUS`.       |
| `1.1.0-amd64-zh-cn-kangkang-apollo-preview` | Imagen de contenedor con la configuración regional `zh-CN` y voz `zh-CN-Kangkang-Apollo`. |
| `1.1.0-amd64-zh-cn-yaoyao-apollo-preview`   | Imagen de contenedor con la configuración regional `zh-CN` y voz `zh-CN-Yaoyao-Apollo`.   |
| `1.0.0-amd64-en-us-benjaminrus-preview`     | Imagen de contenedor con la configuración regional `en-US` y voz `en-US-BenjaminRUS`.     |
| `1.0.0-amd64-en-us-guy24krus-preview`       | Imagen de contenedor con la configuración regional `en-US` y voz `en-US-Guy24kRUS`.       |
| `1.0.0-amd64-en-us-jessa24krus-preview`     | Imagen de contenedor con la configuración regional `en-US` y voz `en-US-Jessa24kRUS`.     |
| `1.0.0-amd64-en-us-jessarus-preview`        | Imagen de contenedor con la configuración regional `en-US` y voz `en-US-JessaRUS`.        |
| `1.0.0-amd64-en-us-zirarus-preview`         | Imagen de contenedor con la configuración regional `en-US` y voz `en-US-ZiraRUS`.         |
| `1.0.0-amd64-zh-cn-huihuirus-preview`       | Imagen de contenedor con la configuración regional `zh-CN` y voz `zh-CN-HuihuiRUS`.       |
| `1.0.0-amd64-zh-cn-kangkang-apollo-preview` | Imagen de contenedor con la configuración regional `zh-CN` y voz `zh-CN-Kangkang-Apollo`. |
| `1.0.0-amd64-zh-cn-yaoyao-apollo-preview`   | Imagen de contenedor con la configuración regional `zh-CN` y voz `zh-CN-Yaoyao-Apollo`.   |

## <a name="key-phrase-extraction"></a>Extracción de frases clave

La imagen de contenedor [Extracción de frases clave][ta-kp] se puede encontrar en el sindicato de registro de contenedor `mcr.microsoft.com`. Reside en el repositorio `azure-cognitive-services` y se denomina `keyphrase`. El nombre completo de la imagen de contenedor es `mcr.microsoft.com/azure-cognitive-services/keyphrase`.

Esta imagen de contenedor tiene disponibles las siguientes etiquetas:

| Etiquetas de imagen                    | Notas |
|-------------------------------|:------|
| `latest`                      |       |
| `1.1.009301-amd64-preview`    |       |
| `1.1.008510001-amd64-preview` |       |
| `1.1.007750002-amd64-preview` |       |
| `1.1.007360001-amd64-preview` |       |
| `1.1.006770001-amd64-preview` |       |

## <a name="language-detection"></a>Detección de idiomas

La imagen de contenedor [Detección de idioma][ta-la] se puede encontrar en el sindicato de registro de contenedor `mcr.microsoft.com`. Reside en el repositorio `azure-cognitive-services` y se denomina `language`. El nombre completo de la imagen de contenedor es `mcr.microsoft.com/azure-cognitive-services/language`.

Esta imagen de contenedor tiene disponibles las siguientes etiquetas:

| Etiquetas de imagen                    | Notas |
|-------------------------------|:------|
| `latest`                      |       |
| `1.1.009301-amd64-preview`    |       |
| `1.1.008510001-amd64-preview` |       |
| `1.1.007750002-amd64-preview` |       |
| `1.1.007360001-amd64-preview` |       |
| `1.1.006770001-amd64-preview` |       |

## <a name="sentiment-analysis"></a>Análisis de sentimiento

La imagen de contenedor [Análisis de sentimiento][ta-se] se puede encontrar en el sindicato de registro de contenedor `mcr.microsoft.com`. Reside en el repositorio `azure-cognitive-services` y se denomina `sentiment`. El nombre completo de la imagen de contenedor es `mcr.microsoft.com/azure-cognitive-services/sentiment`.

Esta imagen de contenedor tiene disponibles las siguientes etiquetas:

| Etiquetas de imagen | Notas                                         |
|------------|:----------------------------------------------|
| `latest`   |                                               |
| `3.0-en`   | Análisis de sentimiento, versión 3 (inglés)               |
| `3.0-es`   | Análisis de sentimiento, versión 3 (español)               |
| `3.0-fr`   | Análisis de sentimiento, versión 3 (francés)                |
| `3.0-it`   | Análisis de sentimiento, versión 3 (italiano)               |
| `3.0-de`   | Análisis de sentimiento, versión 3 (alemán)                |
| `3.0-zh`   | Análisis de sentimiento, versión 3 (chino simplificado)  |
| `3.0-zht`  | Análisis de sentimiento, versión 3 (chino tradicional) |
| `3.0-ja`   | Análisis de sentimiento, versión 3 (japonés)              |
| `3.0-pt`   | Análisis de sentimiento, versión 3 (portugués)            |
| `3.0-nl`   | Análisis de sentimiento, versión 3 (holandés)                 |
| `1.1.009301-amd64-preview`    | Análisis de sentimiento, versión 2      |
| `1.1.008510001-amd64-preview` |       |
| `1.1.007750002-amd64-preview` |       |
| `1.1.007360001-amd64-preview` |       |
| `1.1.006770001-amd64-preview` |       |

[ad-containers]: ../anomaly-Detector/anomaly-detector-container-howto.md
[cv-containers]: ../computer-vision/computer-vision-how-to-install-containers.md
[fa-containers]: ../face/face-how-to-install-containers.md
[fr-containers]: ../form-recognizer/form-recognizer-container-howto.md
[lu-containers]: ../luis/luis-container-howto.md
[sp-stt]: ../speech-service/speech-container-howto.md?tabs=stt
[sp-cstt]: ../speech-service/speech-container-howto.md?tabs=cstt
[sp-tts]: ../speech-service/speech-container-howto.md?tabs=tts
[sp-ctts]: ../speech-service/speech-container-howto.md?tabs=ctts
[ta-kp]: ../text-analytics/how-tos/text-analytics-how-to-install-containers.md?tabs=keyphrase
[ta-la]: ../text-analytics/how-tos/text-analytics-how-to-install-containers.md?tabs=language
[ta-se]: ../text-analytics/how-tos/text-analytics-how-to-install-containers.md?tabs=sentiment
