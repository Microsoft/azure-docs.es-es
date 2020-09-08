---
title: Llamada a Computer Vision API
titleSuffix: Azure Cognitive Services
description: Obtenga información sobre cómo llamar a la API Computer Vision mediante la API REST en Azure Cognitive Services.
services: cognitive-services
author: KellyDF
manager: nitinme
ms.service: cognitive-services
ms.subservice: computer-vision
ms.topic: sample
ms.date: 09/09/2019
ms.author: kefre
ms.custom: seodec18, devx-track-csharp
ms.openlocfilehash: 085da17a29e1d5ff1fa69a62e0029fb917d56bb1
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88936340"
---
# <a name="call-the-computer-vision-api"></a>Llamada a Computer Vision API

En este artículo se muestra cómo llamar a la API Computer Vision mediante la API REST. Los ejemplos están escritos en C# mediante la biblioteca cliente de la API Computer Vision y como llamadas HTTP POST o GET. Este artículo se centra en:

- Obtención de etiquetas, descripciones y categorías
- Obtención de información específica de dominio o "celebridades"

En los ejemplos de este artículo se muestran las características siguientes:

* Análisis de una imagen para devolver una matriz de etiquetas y una descripción
* Análisis de una imagen con un modelo específico de domino (en concreto, el modelo "celebridades") para devolver el resultado correspondiente en JSON

Las características ofrecen las siguientes opciones:

- **Opción 1**: Análisis con ámbito. Análisis de solo un modelo determinado
- **Opción 2**: Análisis mejorado. Análisis para ofrecer detalles adicionales con una [taxonomía de 86 categorías](../Category-Taxonomy.md)

## <a name="prerequisites"></a>Requisitos previos

* Una suscripción a Azure: [cree una cuenta gratuita](https://azure.microsoft.com/free/cognitive-services/).
* Una vez que tenga la suscripción de Azure, <a href="https://portal.azure.com/#create/Microsoft.CognitiveServicesComputerVision"  title="Creación de un recurso de Computer Vision"  target="_blank">cree un recurso de Computer Vision <span class="docon docon-navigate-external x-hidden-focus"></span></a> en Azure Portal para obtener la clave y el punto de conexión. Una vez que se implemente, haga clic en **Ir al recurso**.
    * Necesitará la clave y el punto de conexión del recurso que cree para conectar la aplicación al servicio Computer Vision. En una sección posterior de este mismo inicio rápido pegará la clave y el punto de conexión en el código siguiente.
    * Puede usar el plan de tarifa gratis (`F0`) para probar el servicio y actualizarlo más adelante a un plan de pago para producción.
* Una dirección URL de la imagen o ruta de acceso a la imagen almacenada localmente
* Métodos de entrada admitidos: archivos binarios de imagen raw en forma de application/octet-stream o URL de imagen
* Formatos de archivos de imagen admitidos: JPEG, PNG, GIF y BMP
* Tamaño del archivo de imagen: 4 MB o menos
* Dimensiones de la imagen: 50 &times; 50 píxeles o superior
  
## <a name="authorize-the-api-call"></a>Autorización de la llamada a la API

Cada llamada a Computer Vision API requiere una clave de suscripción. Esta clave debe pasarse a través de un parámetro de cadena de consulta o especificarse en la cabecera de la solicitud.

Para pasar la clave de suscripción, realice una de las acciones siguientes:

* Pásela mediante una cadena de consulta, como en este ejemplo de la API Computer Vision:

  ```
  https://westus.api.cognitive.microsoft.com/vision/v2.1/analyze?visualFeatures=Description,Tags&subscription-key=<Your subscription key>
  ```

* Especifíquela en el encabezado de solicitud HTTP:

  ```
  ocp-apim-subscription-key: <Your subscription key>
  ```

* Si se usa la biblioteca de cliente, pase la clave mediante el constructor de ComputerVisionClient y especifique la región en una propiedad del cliente:

    ```
    var visionClient = new ComputerVisionClient(new ApiKeyServiceClientCredentials("Your subscriptionKey"))
    {
        Endpoint = "https://westus.api.cognitive.microsoft.com"
    }
    ```

## <a name="upload-an-image-to-the-computer-vision-api-service"></a>Carga de una imagen en el servicio de la API Computer Vision

La forma básica para realizar la llamada a la API Computer Vision es cargar directamente una imagen para devolver etiquetas, una descripción y las celebridades. Para ello, envíe una solicitud "POST" con la imagen binaria en el cuerpo HTTP junto con los datos leídos de la imagen. El método de carga es el mismo para todas las llamadas a la API Computer Vision. La única diferencia radicará en los parámetros de consulta que se especifican. 

En el caso de una imagen especificada, obtenga etiquetas y una descripción mediante cualquiera de las siguientes opciones:

### <a name="option-1-get-a-list-of-tags-and-a-description"></a>Opción 1: Obtención de una lista de etiquetas y una descripción

```
POST https://westus.api.cognitive.microsoft.com/vision/v2.1/analyze?visualFeatures=Description,Tags&subscription-key=<Your subscription key>
```

```csharp
using System.IO;
using Microsoft.Azure.CognitiveServices.Vision.ComputerVision;
using Microsoft.Azure.CognitiveServices.Vision.ComputerVision.Models;

ImageAnalysis imageAnalysis;
var features = new VisualFeatureTypes[] { VisualFeatureTypes.Tags, VisualFeatureTypes.Description };

using (var fs = new FileStream(@"C:\Vision\Sample.jpg", FileMode.Open))
{
  imageAnalysis = await visionClient.AnalyzeImageInStreamAsync(fs, features);
}
```

### <a name="option-2-get-a-list-of-tags-only-or-a-description-only"></a>Opción 2: Obtención de una lista solo de etiquetas o de solo una descripción

Solo para las etiquetas, ejecute:

```
POST https://westus.api.cognitive.microsoft.com/vision/v2.1/tag?subscription-key=<Your subscription key>
var tagResults = await visionClient.TagImageAsync("http://contoso.com/example.jpg");
```

Solo para una descripción, ejecute:

```
POST https://westus.api.cognitive.microsoft.com/vision/v2.1/describe?subscription-key=<Your subscription key>
using (var fs = new FileStream(@"C:\Vision\Sample.jpg", FileMode.Open))
{
  imageDescription = await visionClient.DescribeImageInStreamAsync(fs);
}
```

## <a name="get-domain-specific-analysis-celebrities"></a>Obtiene un análisis específico del dominio (celebridades)

### <a name="option-1-scoped-analysis---analyze-only-a-specified-model"></a>Opción 1: Análisis con ámbito. Análisis de solo un modelo determinado
```
POST https://westus.api.cognitive.microsoft.com/vision/v2.1/models/celebrities/analyze
var celebritiesResult = await visionClient.AnalyzeImageInDomainAsync(url, "celebrities");
```

Para esta opción, todos los demás parámetros de consulta {visualFeatures, details} no son válidos. Si desea ver todos los modelos compatibles, use:

```
GET https://westus.api.cognitive.microsoft.com/vision/v2.1/models 
var models = await visionClient.ListModelsAsync();
```

### <a name="option-2-enhanced-analysis---analyze-to-provide-additional-details-by-using-86-categories-taxonomy"></a>Opción 2: Análisis mejorado. Análisis para ofrecer detalles adicionales con una taxonomía de 86 categorías

Para aplicaciones donde desea obtener un análisis genérico de imágenes adicional a los detalles de uno o más modelos específicos del dominio, amplíe la API v1 mediante el parámetro de consulta de modelos.

```
POST https://westus.api.cognitive.microsoft.com/vision/v2.1/analyze?details=celebrities
```

Al invocar este método, primero se llama al clasificador [86-category](../Category-Taxonomy.md). Si ninguna de las categorías concuerda con los modelos conocidos o coincidentes, se procede a una segunda pasada de invocaciones de clasificador. Por ejemplo, si "details=all" o "details" incluye "celebrities", llame al modelo de celebridades después de llamar al clasificador 86-category. El resultado incluye la categoría "person". A diferencia de la opción 1, este método aumenta la latencia de los usuarios que están interesados en celebridades.

En este caso, todos los parámetros de consulta de v1 se comportarán igual. Si no se especifica visualFeatures=categories, se habilita implícitamente.

## <a name="retrieve-and-understand-the-json-output-for-analysis"></a>Recuperar y describir la salida JSON para el análisis

Este es un ejemplo:

```json
{  
  "tags":[  
    {  
      "name":"outdoor",
      "score":0.976
    },
    {  
      "name":"bird",
      "score":0.95
    }
  ],
  "description":{  
    "tags":[  
      "outdoor",
      "bird"
    ],
    "captions":[  
      {  
        "text":"partridge in a pear tree",
        "confidence":0.96
      }
    ]
  }
}
```

Campo | Tipo | Contenido
------|------|------|
Etiquetas  | `object` | Objeto de nivel superior de una matriz de etiquetas
tags[].Name | `string`    | Palabra clave del clasificador de etiquetas
tags[].Score    | `number`    | Puntuación de confianza, entre 0 y 1
description     | `object`    | Objeto de nivel superior de la descripción
description.tags[] |    `string`    | Lista de etiquetas.  Si la confianza es insuficiente para poder producir un título, las etiquetas pueden ser la única información disponible para el llamador.
description.captions[].text    | `string`    | Una frase que describe la imagen
description.captions[].confidence    | `number`    | Puntuación de confianza para la frase

## <a name="retrieve-and-understand-the-json-output-of-domain-specific-models"></a>Recuperar y describir la salida JSON de modelos específicos del dominio

### <a name="option-1-scoped-analysis---analyze-only-a-specified-model"></a>Opción 1: Análisis con ámbito. Análisis de solo un modelo determinado

La salida es una matriz de etiquetas, tal como se muestra en el ejemplo siguiente:

```json
{  
  "result":[  
    {  
      "name":"golden retriever",
      "score":0.98
    },
    {  
      "name":"Labrador retriever",
      "score":0.78
    }
  ]
}
```

### <a name="option-2-enhanced-analysis---analyze-to-provide-additional-details-by-using-the-86-categories-taxonomy"></a>Opción 2: Análisis mejorado. Análisis para ofrecer detalles adicionales con una taxonomía de 86 categorías

Para modelos específicos del dominio con la opción 2 (Análisis mejorado), el tipo de devolución de categorías se amplía, tal como se muestra en el ejemplo siguiente:

```json
{  
  "requestId":"87e44580-925a-49c8-b661-d1c54d1b83b5",
  "metadata":{  
    "width":640,
    "height":430,
    "format":"Jpeg"
  },
  "result":{  
    "celebrities":[  
      {  
        "name":"Richard Nixon",
        "faceRectangle":{  
          "left":107,
          "top":98,
          "width":165,
          "height":165
        },
        "confidence":0.9999827
      }
    ]
  }
}
```

El campo de categorías es una lista de una o varias de las [86 categorías](../Category-Taxonomy.md) de la taxonomía original. Las categorías que terminan con un guión bajo coincidirán con esa categoría y sus elementos secundarios (por ejemplo, "people_" o "people_group", para el modelo de celebridades).

Campo    | Tipo    | Contenido
------|------|------|
categories | `object`    | Objeto de nivel superior
categories[].name     | `string`    | Nombre de la lista de taxonomía de 86 categorías
categories[].score    | `number`    | Puntuación de confianza, entre 0 y 1
categories[].detail     | `object?`      | (Opcional) Objeto de detalle.

Si varias categorías coinciden (por ejemplo, el clasificador "86-category" devuelve una puntuación para "people_" y "people_young" si model=celebrities), los detalles se asocian a la coincidencia de nivel más general ("people_" en dicho ejemplo).

## <a name="error-responses"></a>Respuestas de errores

Estos errores son idénticos a los de vision.analyze, con el error adicional de NotSupportedModel (HTTP 400), que puede devolverse en escenarios de la opción 1 y la opción 2. Para la opción 2 (análisis mejorado), si no se reconoce ninguno de los modelos especificados en los detalles, la API devolverá NotSupportedModel, incluso si uno o varios de ellos son válidos. Para averiguar qué modelos se admiten, se puede llamar a listModels.

## <a name="next-steps"></a>Pasos siguientes

Para usar la API de REST, vaya a la [referencia de Computer Vision API](https://westus.dev.cognitive.microsoft.com/docs/services/computer-vision-v3-ga/operations/56f91f2e778daf14a499f21b).
