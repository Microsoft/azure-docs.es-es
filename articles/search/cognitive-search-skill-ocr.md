---
title: 'Aptitud de búsqueda cognitiva de OCR: Azure Search'
description: Extraiga texto a partir de archivos de imagen mediante el reconocimiento óptico de caracteres (OCR) en una canalización de enriquecimiento de Azure Search.
services: search
manager: pablocas
author: luiscabrer
ms.service: search
ms.devlang: NA
ms.workload: search
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.date: 05/02/2019
ms.author: luisca
ms.custom: seodec2018
ms.openlocfilehash: 6d9b68bda2a6cff533286d9ee944abf1c92cc2bf
ms.sourcegitcommit: bb85a238f7dbe1ef2b1acf1b6d368d2abdc89f10
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/10/2019
ms.locfileid: "65523239"
---
# <a name="ocr-cognitive-skill"></a>Aptitud cognitiva de OCR

La aptitud de reconocimiento óptico de caracteres (OCR) reconoce texto impreso y manuscrito en archivos de imagen. Esta aptitud utiliza los modelos de aprendizaje automático proporcionados por [Computer Vision](https://docs.microsoft.com/azure/cognitive-services/computer-vision/home) en Cognitive Services. La aptitud de **reconocimiento óptico de caracteres** se asocia a la siguiente funcionalidad:

+ Cuando textExtractionAlgorithm está establecido en "escrito a mano", se usa la funcionalidad ["RecognizeText"](../cognitive-services/computer-vision/quickstarts-sdk/csharp-hand-text-sdk.md).
+ Cuando textExtractionAlgorithm está establecido en "impreso", se usa la funcionalidad ["OCR"](../cognitive-services/computer-vision/concept-extracting-text-ocr.md) para todos los idiomas excepto el inglés. Para el inglés, se usa la nueva funcionalidad ["Reconocer texto"](../cognitive-services/computer-vision/concept-recognizing-text.md) para texto impreso.

La aptitud de **OCR** extrae texto de los archivos de imagen. Entre otros, estos son los formatos de archivos admitidos:

+ .JPEG
+ .JPG
+ .PNG
+ .BMP
+ .GIF
+ .TIFF

> [!NOTE]
> A medida que expanda el ámbito aumentando la frecuencia de procesamiento, agregando más documentos o agregando más algoritmos de IA, tendrá que [asociar un recurso facturable de Cognitive Services](cognitive-search-attach-cognitive-services.md). Los cargos se acumulan cuando se llama a las API de Cognitive Services y para la extracción de imágenes como parte de la fase de descifrado de documentos en Azure Search. No hay ningún cargo por la extracción de texto de documentos.
>
> La ejecución de aptitudes integradas se cobra según los [precios de pago por uso de Cognitive Services](https://azure.microsoft.com/pricing/details/cognitive-services/) existentes. Los precios de la extracción de imágenes se describen en la [página de precios de Azure Search](https://go.microsoft.com/fwlink/?linkid=2042400).


## <a name="skill-parameters"></a>Parámetros de la aptitud

Los parámetros distinguen mayúsculas de minúsculas.

| Nombre de parámetro     | DESCRIPCIÓN |
|--------------------|-------------|
| detectOrientation | Permite la detección automática de la orientación de la imagen. <br/> Valores válidos: true / false.|
|defaultLanguageCode | <p>  Código de idioma del texto de entrada. Los lenguajes compatibles incluyen: <br/> zh-Hans (chino simplificado) <br/> zh-Hant (chino tradicional) <br/>cs (checo) <br/>da (danés) <br/>nl (neerlandés) <br/>en (inglés) <br/>fi (finés)  <br/>fr (francés) <br/>  de (alemán) <br/>el (griego) <br/> hu (húngaro) <br/> it (italiano) <br/>  ja (japonés) <br/> ko (coreano) <br/> nb (noruego) <br/>   pl (polaco) <br/> pt (portugués) <br/>  ru (ruso) <br/>  es (español) <br/>  sv (sueco) <br/>  tr (turco) <br/> ar (árabe) <br/> ro (rumano) <br/> sr-Cyrl (serbio cirílico) <br/> sr-Latn (serbio latino) <br/>  sk (eslovaco) <br/>  unk (desconocido) <br/><br/> Si el código de idioma no está especificado o es nulo, el idioma que se establezca será "inglés". Si el idioma se establece explícitamente en "unk", el idioma se detectará automáticamente. </p> |
| textExtractionAlgorithm | "impreso" o "manuscrito". El algoritmo OCR de reconocimiento de texto "manuscrito" se encuentra actualmente en versión preliminar y solo se admite en inglés. |
|lineEnding | El valor que se usa entre cada detectado línea. Valores posibles: «Espacio», «CarriageReturn», '/ salto de línea'.  El valor predeterminado es «Espacio» |

## <a name="skill-inputs"></a>Entradas de la aptitud

| Indicar nombre      | DESCRIPCIÓN                                          |
|---------------|------------------------------------------------------|
| image         | Tipo complejo. Actualmente, solo funciona con el campo "/document/normalized_images", que crea el indexador de Azure Blob cuando ```imageAction``` está establecido en un valor diferente a ```none```. Para obtener más información, consulte este [ejemplo](#sample-output).|


## <a name="skill-outputs"></a>Salidas de aptitudes
| Nombre de salida     | DESCRIPCIÓN                   |
|---------------|-------------------------------|
| text          | Texto sin formato extraído de la imagen.   |
| layoutText    | Tipo complejo que describe el texto extraído, así como la ubicación donde se encontró.|


## <a name="sample-definition"></a>Definición de ejemplo

```json
{
  "skills": [
    {
      "description": "Extracts text (plain and structured) from image.",
      "@odata.type": "#Microsoft.Skills.Vision.OcrSkill",
      "context": "/document/normalized_images/*",
      "defaultLanguageCode": null,
      "detectOrientation": true,
      "inputs": [
        {
          "name": "image",
          "source": "/document/normalized_images/*"
        }
      ],
      "outputs": [
        {
          "name": "text",
          "targetName": "myText"
        },
        {
          "name": "layoutText",
          "targetName": "myLayoutText"
        }
      ]
    }
  ]
}
```
<a name="sample-output"></a>

## <a name="sample-text-and-layouttext-output"></a>Texto y salida de layoutText de ejemplo

```json
{
  "text": "Hello World. -John",
  "layoutText":
  {
    "language" : "en",
    "text" : "Hello World. -John",
    "lines" : [
      {
        "boundingBox":
        [ {"x":10, "y":10}, {"x":50, "y":10}, {"x":50, "y":30},{"x":10, "y":30}],
        "text":"Hello World."
      },
      {
        "boundingBox": [ {"x":110, "y":10}, {"x":150, "y":10}, {"x":150, "y":30},{"x":110, "y":30}],
        "text":"-John"
      }
    ],
    "words": [
      {
        "boundingBox": [ {"x":110, "y":10}, {"x":150, "y":10}, {"x":150, "y":30},{"x":110, "y":30}],
        "text":"Hello"
      },
      {
        "boundingBox": [ {"x":110, "y":10}, {"x":150, "y":10}, {"x":150, "y":30},{"x":110, "y":30}],
        "text":"World."
      },
      {
        "boundingBox": [ {"x":110, "y":10}, {"x":150, "y":10}, {"x":150, "y":30},{"x":110, "y":30}],
        "text":"-John"
      }
    ]
  }
}
```

## <a name="sample-merging-text-extracted-from-embedded-images-with-the-content-of-the-document"></a>Sample: combinar el texto extraído de imágenes insertadas con el contenido del documento.

Un caso de uso común a la hora de combinar texto, es la capacidad de combinar la representación textual de imágenes (el texto de una aptitud de OCR o la leyenda de una imagen) en el campo de contenido de un documento.

El siguiente conjunto de aptitudes de ejemplo crea un campo *merged_text*. Este campo contiene el contenido textual del documento y el texto procesado con OCR de cada una de las imágenes insertadas en ese documento.

#### <a name="request-body-syntax"></a>Sintaxis del cuerpo de la solicitud
```json
{
  "description": "Extract text from images and merge with content text to produce merged_text",
  "skills":
  [
    {
      "description": "Extract text (plain and structured) from image.",
      "@odata.type": "#Microsoft.Skills.Vision.OcrSkill",
      "context": "/document/normalized_images/*",
      "defaultLanguageCode": "en",
      "detectOrientation": true,
      "inputs": [
        {
          "name": "image",
          "source": "/document/normalized_images/*"
        }
      ],
      "outputs": [
        {
          "name": "text"
        }
      ]
    },
    {
      "@odata.type": "#Microsoft.Skills.Text.MergeSkill",
      "description": "Create merged_text, which includes all the textual representation of each image inserted at the right location in the content field.",
      "context": "/document",
      "insertPreTag": " ",
      "insertPostTag": " ",
      "inputs": [
        {
          "name":"text", "source": "/document/content"
        },
        {
          "name": "itemsToInsert", "source": "/document/normalized_images/*/text"
        },
        {
          "name":"offsets", "source": "/document/normalized_images/*/contentOffset"
        }
      ],
      "outputs": [
        {
          "name": "mergedText", "targetName" : "merged_text"
        }
      ]
    }
  ]
}
```
En el ejemplo del conjunto de aptitudes anterior, se asume que existe un campo de imágenes normalizadas. Para crear este campo, establezca la configuración *imageAction* en la definición del indexador como *generateNormalizedImages*, tal como se muestra a continuación:

```json
{
  //...rest of your indexer definition goes here ...
  "parameters": {
    "configuration": {
      "dataToExtract":"contentAndMetadata",
      "imageAction":"generateNormalizedImages"
    }
  }
}
```

## <a name="see-also"></a>Vea también
+ [Aptitudes predefinidas](cognitive-search-predefined-skills.md)
+ [Aptitud TextMerger](cognitive-search-skill-textmerger.md)
+ [Definición de un conjunto de aptitudes](cognitive-search-defining-skillset.md)
+ [Create Indexer (REST)](https://docs.microsoft.com/rest/api/searchservice/create-indexer)
