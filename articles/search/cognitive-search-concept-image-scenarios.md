---
title: Extracción de texto de las imágenes
titleSuffix: Azure Cognitive Search
description: Procese y extraiga el texto y otra información de imágenes en canalizaciones de Azure Cognitive Search.
manager: nitinme
author: LuisCabrer
ms.author: luisca
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 11/04/2019
ms.custom: devx-track-csharp
ms.openlocfilehash: 56ec893de159f4c8a90c5a229ccf7669856fb066
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89020225"
---
# <a name="how-to-process-and-extract-information-from-images-in-ai-enrichment-scenarios"></a>Procesamiento y extracción de información de imágenes en escenarios de enriquecimiento con IA

Azure Cognitive Search tiene varias funcionalidades para trabajar con imágenes y archivos de imagen. Durante la averiguación de documentos, se puede utilizar el parámetro *imageAction* para extraer el texto de fotografías o imágenes que contengan texto alfanumérico, como la palabra "STOP" en una señal de parada. Otros escenarios incluyen generar una representación en texto de una imagen, por ejemplo, "diente de león" para una foto de un diente de león o el color "amarillo". También puede extraer metadatos sobre la imagen, como su tamaño.

En este artículo se trata con más detalle el procesamiento de imágenes y se proporcionan instrucciones para trabajar con imágenes en una canalización de enriquecimiento con IA.

<a name="get-normalized-images"></a>

## <a name="get-normalized-images"></a>Obtener imágenes normalizadas

Como parte de la averiguación de documentos, hay un nuevo conjunto de parámetros de configuración de indexador para administrar archivos de imagen o imágenes incrustadas en archivos. Estos parámetros se utilizan para normalizar imágenes para un procesamiento adicional. Normalizar imágenes hace que sean más uniformes. El tamaño de las imágenes grandes se cambia a un alto y ancho máximos para que se puedan consumir. Para las imágenes que proporcionan metadatos sobre la orientación, la rotación de la imagen se ajusta para la carga vertical. Los ajustes de metadatos se capturan en un tipo complejo que se crea para cada imagen. 

No puede desactivar la normalización de imágenes. Las habilidades que iteran las imágenes esperan imágenes normalizadas. Para habilitar la normalización de imágenes en un indexador debe adjuntar un conjunto de aptitudes a ese indexador.

| Parámetro de configuración | Descripción |
|--------------------|-------------|
| imageAction   | Se establece en "none" si no debe realizar ninguna acción cuando se encuentran imágenes incrustadas o archivos de imagen. <br/>Se establece en "generateNormalizedImages" para generar una matriz de imágenes normalizadas como parte de la averiguación del documento.<br/>Se establece en "generateNormalizedImagePerPage" para generar una matriz de imágenes normalizadas donde en los archivos PDF del origen de datos, cada página se representa en una imagen de salida.  La funcionalidad es igual que "generateNormalizedImages" para tipos de archivos que no son PDF.<br/>En el caso de todas las opciones que no sean "none", las imágenes se expondrán en el campo *normalized_images*. <br/>El valor predeterminado es "none". Esta configuración solo es pertinente para los orígenes de datos de blob cuando "dataToExtract" se establece en "contentAndMetadata". <br/>Se extraerá un máximo de 1000 imágenes de un documento determinado. Si hay más de 1000 imágenes en un documento, se extraerán las 1000 primeras y se generará una advertencia. |
|  normalizedImageMaxWidth | El ancho máximo (en píxeles) para las imágenes normalizadas generadas. El valor predeterminado es 2000. El valor máximo permitido es 10 000. | 
|  normalizedImageMaxHeight | La altura máxima (en píxeles) para las imágenes normalizadas generadas. El valor predeterminado es 2000. El valor máximo permitido es 10 000.|

> [!NOTE]
> Si establece la propiedad *imageAction* en algo distinto de "none", no podrá establecer la propiedad *parsingMode* en algo distinto de "default".  Solo puede establecer una de estas dos propiedades en un valor no predeterminado en la configuración del indexador.

Establezca el parámetro **parsingMode** en `json` (para indexar cada blob como un documento individual) o en `jsonArray` (si los blobs contienen matrices JSON y necesita que cada elemento de una matriz se trate como un documento independiente).

El valor predeterminado es de 2000 píxeles para el ancho máximo de las imágenes normalizadas, y la altura se basa en los tamaños máximos admitidos por la [habilidad de OCR](cognitive-search-skill-ocr.md) y la [habilidad de análisis de imágenes](cognitive-search-skill-image-analysis.md). La [aptitud de OCR](cognitive-search-skill-ocr.md) admite un ancho y un alto máximos de 4200 para los idiomas distintos del inglés y 10 000 para el inglés.  Si aumenta los límites máximos, el procesamiento podría generar un error en imágenes de mayor tamaño en función de la definición del conjunto de aptitudes y del idioma de los documentos. 

Especifique imageAction en su [definición de indexador](/rest/api/searchservice/create-indexer) como se indica a continuación:

```json
{
  //...rest of your indexer definition goes here ...
  "parameters":
  {
    "configuration": 
    {
        "dataToExtract": "contentAndMetadata",
        "imageAction": "generateNormalizedImages"
    }
  }
}
```

Cuando *imageAction* se establece en un valor distinto de "none", el nuevo campo *normalized_images* contendrá una matriz de imágenes. Cada imagen es un tipo complejo que tiene los siguientes miembros:

| Miembro de la imagen       | Descripción                             |
|--------------------|-----------------------------------------|
| datos               | Cadena codificada en BASE64 de la imagen normalizada en formato JPEG.   |
| width              | Ancho de la imagen normalizada en píxeles. |
| height             | Altura de la imagen normalizada en píxeles. |
| originalWidth      | El ancho original de la imagen antes de la normalización. |
| originalHeight      | La altura original de la imagen antes de la normalización. |
| rotationFromOriginal |  Rotación a la izquierda en grados, que se produjo para crear la imagen normalizada. Un valor entre 0 y 360 grados. Este paso lee mediante una cámara o escáner los metadatos de la imagen que se genera. Normalmente, es un múltiplo de 90 grados. |
| contentOffset | El desplazamiento de caracteres en el campo de contenido desde donde se extrajo la imagen. Este campo solo es aplicable para los archivos con imágenes incrustadas. |
| pageNumber | Si la imagen se extrajo o se representó desde un archivo PDF, este campo contiene el número de página en el PDF del que se extrajo o se representó, empezando por 1.  Si la imagen no era de un archivo PDF, este campo será 0.  |

 Muestra el valor de *normalized_images*:
```json
[
  {
    "data": "BASE64 ENCODED STRING OF A JPEG IMAGE",
    "width": 500,
    "height": 300,
    "originalWidth": 5000,  
    "originalHeight": 3000,
    "rotationFromOriginal": 90,
    "contentOffset": 500,
    "pageNumber": 2
  }
]
```

## <a name="image-related-skills"></a>Habilidades relacionadas con la imagen

Hay dos habilidades cognitivas integradas que se usan para tomar imágenes como entrada: [OCR](cognitive-search-skill-ocr.md) y [Análisis de imágenes](cognitive-search-skill-image-analysis.md). 

Actualmente, estas habilidades solo funcionan con imágenes generadas a partir del paso de averiguación de documentos. Por lo tanto, la única entrada compatible es `"/document/normalized_images"`.

### <a name="image-analysis-skill"></a>Habilidad de Análisis de imágenes

La [habilidad de Análisis de imágenes](cognitive-search-skill-image-analysis.md) extrae un amplio conjunto de características visuales en función del contenido de la imagen. Por ejemplo, puede generar un título de una imagen, generar etiquetas o identificar celebridades y lugares de referencia.

### <a name="ocr-skill"></a>Habilidad de OCR

La [habilidad de OCR](cognitive-search-skill-ocr.md) extrae el texto de los archivos de imagen, como JPG, PNG y mapas de bits. Puede extraer el texto, así como información de diseño. La información de diseño proporciona rectángulos delimitadores para cada una de las cadenas identificadas.

## <a name="embedded-image-scenario"></a>Escenario de imagen incrustada

Un escenario común implica la creación de una única cadena que contenga todo el contenido del archivo, tanto texto como texto originado en imágenes, a través de los pasos siguientes:  

1. [Extraer normalized_images](#get-normalized-images)
1. Ejecute la habilidad de OCR utilizando `"/document/normalized_images"` como entrada.
1. Combine la representación de texto de esas imágenes con el texto sin formato extraído del archivo. Puede usar la habilidad [Combinación de texto](cognitive-search-skill-textmerger.md) para consolidar ambos fragmentos de texto en una única cadena grande.

El conjunto de habilidades de ejemplo siguiente crea un campo *merged_text* con el contenido textual del documento. También incluye el texto procesado con OCR de cada una de las imágenes incrustadas. 

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

Ahora que tiene un campo merged_text, podría asignarlo como campo que admita búsquedas en la definición del indexador. Podrá buscar todo el contenido de los archivos, incluido el texto de las imágenes.

## <a name="visualize-bounding-boxes-of-extracted-text"></a>Visualización de rectángulos delimitadores de texto extraído

Otro escenario común es la visualización de información de diseño de los resultados de búsqueda. Por ejemplo, puede querer resaltar donde se encontró un fragmento de texto en una imagen como parte de los resultados de búsqueda.

Puesto que el paso de OCR se realiza en las imágenes normalizadas, las coordenadas de diseño están en el espacio de la imagen normalizada. Cuando se muestra la imagen normalizada, la presencia de coordenadas no suele ser un problema, pero en algunas situaciones puede querer mostrar la imagen original. En este caso, convierta cada uno de los puntos de coordenadas en el diseño al sistema de coordenadas de la imagen original. 

Como ayuda adicional, si tiene que transformar las coordenadas normalizadas al espacio de coordenadas original, podría utilizar el siguiente algoritmo:

```csharp
        /// <summary>
        ///  Converts a point in the normalized coordinate space to the original coordinate space.
        ///  This method assumes the rotation angles are multiples of 90 degrees.
        /// </summary>
        public static Point GetOriginalCoordinates(Point normalized,
                                    int originalWidth,
                                    int originalHeight,
                                    int width,
                                    int height,
                                    double rotationFromOriginal)
        {
            Point original = new Point();
            double angle = rotationFromOriginal % 360;

            if (angle == 0 )
            {
                original.X = normalized.X;
                original.Y = normalized.Y;
            } else if (angle == 90)
            {
                original.X = normalized.Y;
                original.Y = (width - normalized.X);
            } else if (angle == 180)
            {
                original.X = (width -  normalized.X);
                original.Y = (height - normalized.Y);
            } else if (angle == 270)
            {
                original.X = height - normalized.Y;
                original.Y = normalized.X;
            }

            double scalingFactor = (angle % 180 == 0) ? originalHeight / height : originalHeight / width;
            original.X = (int) (original.X * scalingFactor);
            original.Y = (int)(original.Y * scalingFactor);

            return original;
        }
```

## <a name="see-also"></a>Consulte también
+ [Create indexer (REST)](/rest/api/searchservice/create-indexer)
+ [Aptitud de análisis de imágenes](cognitive-search-skill-image-analysis.md)
+ [Habilidad de OCR](cognitive-search-skill-ocr.md)
+ [Habilidad de Combinación de texto](cognitive-search-skill-textmerger.md)
+ [Definición de un conjunto de aptitudes](cognitive-search-defining-skillset.md)
+ [Cómo asignar campos enriquecidos](cognitive-search-output-field-mapping.md)