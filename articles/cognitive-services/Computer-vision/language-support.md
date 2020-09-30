---
title: 'Compatibilidad con idiomas: Computer Vision'
titleSuffix: Azure Cognitive Services
description: 'En este artículo se proporciona una lista de lenguajes naturales que admiten las características de Computer Vision: OCR y análisis de imagen.'
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: computer-vision
ms.topic: conceptual
ms.date: 04/17/2019
ms.author: pafarley
ms.openlocfilehash: 844ef8c09f10e1df888ef4990db22be8705d5c01
ms.sourcegitcommit: 03662d76a816e98cfc85462cbe9705f6890ed638
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/15/2020
ms.locfileid: "90530206"
---
# <a name="language-support-for-computer-vision"></a>Compatibilidad con idiomas para Computer Vision

Algunas características de Computer Vision admiten varios idiomas; las características que no se mencionan aquí solo admiten inglés.

## <a name="optical-character-recognition-ocr"></a>Reconocimiento óptico de caracteres (OCR)

Las API de OCR de Computer Vision admiten varios idiomas. No requieren que especifique un código de idioma. Consulte [Reconocimiento óptico de caracteres (OCR)](concept-recognizing-text.md) para obtener más información.

|Idioma| Código de lenguaje | API de OCR | Read v3.0 | Versión preliminar pública de Read v3.1 |
|:-----|:----:|:-----:|:---:|:---:|
|Árabe | `ar`|✔ | | |
|Chino (simplificado) | `zh-Hans`|✔ | |✔ |
|Chino (tradicional) | `zh-Hant`|✔ | | |
|Checo | `cs` |✔ | | |
|Danés | `da` |✔ | | |
|Neerlandés | `nl` |✔ |✔ |✔ |
|Inglés | `en` |✔ |✔ |✔ |
|Finés | `fi` |✔ | | |
|Francés | `fr` |✔ |✔ |✔ |
|Alemán | `de` |✔ |✔ |✔ |
|Griego | `el` |✔ | | |
|Húngaro | `hu` |✔ | | |
|Italiano | `it` |✔ |✔ |✔ |
|Japonés | `ja` |✔ | |✔ |
|Coreano | `ko` |✔ | | |
|Noruego | `nb` |✔ | | |
|Polaco | `pl` |✔ | | |
|Portugués | `pt` |✔ |✔ |✔ |
|Rumano | `ro` |✔ | | |
|Ruso | `ru` |✔ | | |
|Serbio (cirílico) | `sr-Cyrl` |✔ | | |
|Serbio (latino) | `sr-Latn` |✔ | | |
|Eslovaco | `sk` |✔ | | |
|Español | `es` |✔ |✔ |✔ |
|Sueco | `sw` |✔ | | |
|Turco | `tr` |✔ | | |

## <a name="image-analysis"></a>Análisis de imágenes

Algunas acciones de la API [analizar: imagen](https://westus.dev.cognitive.microsoft.com/docs/services/5adf991815e1060e6355ad44/operations/56f91f2e778daf14a499e1fa) pueden devolver resultados en otros idiomas, especificados con el parámetro de consulta `language`. Otras acciones devuelven resultados en inglés, independientemente del idioma que se especifique, y otros generan una excepción para los idiomas no admitidos. Las acciones se especifican con los parámetros de consulta `visualFeatures` y `details`; vea la [información general](overview.md) para obtener una lista de todas las acciones que puede hacer con el análisis de imágenes.

|Idioma | Código de lenguaje | Categorías | Etiquetas | Descripción | Adultos | Marcas | Color | Caras | ImageType | Objetos | Celebridades | Puntos de referencia |
|:---|:---:|:----:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
|Chino | `zh`    | ✔ | ✔| ✔|-|-|-|-|-|❌|✔|✔|
|Inglés | `en`   | ✔ | ✔| ✔|✔|✔|✔|✔|✔|✔|✔|✔|
|Japonés | `ja`   | ✔ | ✔| ✔|-|-|-|-|-|❌|✔|✔|
|Portugués | `pt` | ✔ | ✔| ✔|-|-|-|-|-|❌|✔|✔|
|Español | `es`    | ✔ | ✔| ✔|-|-|-|-|-|❌|✔|✔|

## <a name="next-steps"></a>Pasos siguientes

Conozca el uso de las características de Computer Vision mencionadas en esta guía.

* [Análisis de imágenes locales (REST)](./quickstarts/csharp-analyze.md)
* [Extracción de texto impreso (REST)](./quickstarts/csharp-print-text.md)
