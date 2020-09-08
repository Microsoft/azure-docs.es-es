---
title: Realización de análisis de opiniones con Text Analytics API de REST
titleSuffix: Azure Cognitive Services
description: En este artículo se muestra cómo detectar opiniones en texto con la API de REST de Azure Cognitive Services Text Analytics.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: text-analytics
ms.topic: sample
ms.date: 08/25/2020
ms.author: aahi
ms.openlocfilehash: a0557c3ccf6510ab3ee2ae29cbef1fc754473345
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88933025"
---
# <a name="how-to-detect-sentiment-using-the-text-analytics-api"></a>Procedimientos: Detección de opiniones mediante la API Text Analytics

La característica Análisis de sentimiento de la API Text Analytics evalúa el texto y devuelve las etiquetas y puntuaciones de opinión de cada oración, lo que resulta útil para detectar opiniones positivas y negativas tanto en redes sociales como en reseñas de los clientes o en foros de discusión, entre otros. El servicio proporciona los modelos de AI que usa la API, el usuario solo tiene que enviar contenido para su análisis.

Después de enviar una solicitud de análisis de sentimiento, la API devuelve etiquetas de sentimiento (como "negative", "neutral" y "positive") y puntuaciones de confianza en el nivel de oración y de documento.

Análisis de sentimiento admite una amplia gama de idiomas, y hay más en la versión preliminar. Para más información, consulte [Lenguajes admitidos](../text-analytics-supported-languages.md).

## <a name="sentiment-analysis-versions-and-features"></a>Versiones y características de Análisis de sentimiento

[!INCLUDE [v3 region availability](../includes/v3-region-availability.md)]

| Característica                                   | Análisis de sentimiento, versión 3 | Análisis de sentimiento, versión 3.1 (versión preliminar) |
|-------------------------------------------|-----------------------|-----------------------------------|
| Métodos para solicitudes individuales y por lotes    | X                     | X                                 |
| Puntuaciones y etiquetado de sentimiento             | X                     | X                                 |
| [Contenedor de Docker](text-analytics-how-to-install-containers.md) basado en Linux | X  |  |
| Minería de opiniones                            |                       | X                                 |

### <a name="sentiment-scoring-and-labeling"></a>Puntuación y etiquetado de sentimiento

La versión 3 de Análisis de sentimiento aplica etiquetas de sentimiento al texto, que se devuelven en el nivel de oración y de documento, con una puntuación de confianza para cada una de ellas. 

Las etiquetas son *positive*, *negative* y *neutral*. En el nivel de documento, también se puede devolver la etiqueta de opinión *mixed*. La opinión del documento se determina a continuación:

| Opiniones de frases                                                                            | Etiqueta de documento devuelta |
|-----------------------------------------------------------------------------------------------|-------------------------|
| Hay al menos una oración `positive` en el documento. El resto de las oraciones son `neutral`. | `positive`              |
| Hay al menos una oración `negative` en el documento. El resto de las oraciones son `neutral`. | `negative`              |
| Hay al menos una oración `negative` y al menos una oración `positive` en el documento.    | `mixed`                 |
| Todas las oraciones del documento son `neutral`.                                                  | `neutral`               |

Las puntuaciones de confianza oscilan entre 1 y 0. Las puntuaciones más próximas a 1 indican una mayor confianza en la clasificación de la etiqueta, mientras que las puntuaciones inferiores indican una menor confianza. Para cada documento o frase, las puntuaciones previstas asociadas a las etiquetas (positive, negative y neutral) se suman hasta un máximo de 1.

### <a name="opinion-mining"></a>Minería de opiniones

La minería de opiniones es una característica de Análisis de sentimiento, a partir de la versión 3.1-preview.1. Esta característica, también conocida como Análisis de sentimiento basada en aspectos en el procesamiento de lenguaje natural (NLP), proporciona información más detallada sobre las opiniones relacionadas con los aspectos (como los atributos de los productos o servicios) en el texto.

Por ejemplo, si un cliente deja comentarios sobre un hotel, como "la habitación era fantástica, pero el personal era antipático", la minería de opiniones ubicará aspectos en el texto, así como sus opiniones y sentimientos asociados:

| Aspecto | Opinión    | Sentimiento |
|--------|------------|-----------|
| habitación   | fantástica      | positiva  |
| personal  | antipático | negative  |

Para obtener minería de opiniones en los resultados, debe incluir la marca `opinionMining=true` en las solicitudes de análisis de sentimiento. Los resultados de la minería de opiniones se incluirán en la respuesta del análisis de sentimiento.

## <a name="sending-a-rest-api-request"></a>Envío de una solicitud de API REST 

### <a name="preparation"></a>Preparación

Análisis de sentimiento genera un resultado de mayor calidad cuando se le proporcionan fragmentos de texto más pequeños para trabajar. Esto es opuesto a la extracción de frases clave, que funciona mejor en bloques de texto más grandes. Para obtener los mejores resultados de ambas operaciones, considere la posibilidad de reestructurar las entradas en consecuencia.

Debe tener documentos JSON en este formato: identificador, texto e idioma.

El tamaño del documento debe ser inferior a 5120 caracteres por documento. Puede tener hasta 1000 elementos (identificadores) por colección. La colección se envía en el cuerpo de la solicitud.

## <a name="structure-the-request"></a>Estructurar la solicitud

Cree una solicitud POST. Puede [usar Postman](text-analytics-how-to-call-api.md) o la **consola de pruebas de API** en los siguientes vínculos de referencia para estructurar y enviar rápidamente una. 

#### <a name="version-30"></a>[Versión 3.0](#tab/version-3)

[Referencia de Análisis de sentimiento, versión 3](https://westus2.dev.cognitive.microsoft.com/docs/services/TextAnalytics-v3-0/operations/Sentiment)

#### <a name="version-31-preview1"></a>[Versión 3.1-preview.1](#tab/version-3-1)

[Referencia de Análisis de sentimiento, versión 3.1](https://westcentralus.dev.cognitive.microsoft.com/docs/services/TextAnalytics-v3-1-preview-1/operations/Sentiment)

---

### <a name="request-endpoints"></a>Puntos de conexión de solicitud

Establezca el punto de conexión HTTPS para el análisis de sentimiento, para lo que puede usar un recurso de Text Analytics en Azure o un [contenedor de Text Analytics](text-analytics-how-to-install-containers.md) con instancias. Debe incluir la dirección URL correcta para la versión que desea usar. Por ejemplo:

> [!NOTE]
> Puede encontrar la clave y el punto de conexión del recurso de Text Analytics en Azure Portal. Están en la página **Inicio rápido** del recurso, en **Administración de recursos**. 

#### <a name="version-30"></a>[Versión 3.0](#tab/version-3)

`https://<your-custom-subdomain>.cognitiveservices.azure.com/text/analytics/v3.0/sentiment`

#### <a name="version-31-preview1"></a>[Versión 3.1-preview.1](#tab/version-3-1)

`https://<your-custom-subdomain>.cognitiveservices.azure.com/text/analytics/v3.1-preview.1/sentiment`

Para obtener resultados de la minería de opiniones, debe incluir el parámetro `opinionMining=true`. Por ejemplo:

`https://<your-custom-subdomain>.cognitiveservices.azure.com/text/analytics/v3.1-preview.1/sentiment?opinionMining=true`

De manera predeterminada, este parámetro se establece en `false`. 

---

Establezca un encabezado de la solicitud para incluir la clave de la API Text Analytics. En el cuerpo de la solicitud, proporcione la colección de documentos JSON que preparó para este análisis.

### <a name="example-sentiment-analysis-request"></a>Solicitud de ejemplo de Análisis de sentimiento 

En el siguiente ejemplo se presenta contenido que se podría enviar para el análisis de opiniones. El formato de la solicitud es el mismo para ambas versiones.
    
```json
{
  "documents": [
    {
      "language": "en",
      "id": "1",
      "text": "The restaurant had great food and our waiter was friendly."
    }
  ]
}
```

### <a name="post-the-request"></a>Publicar la solicitud

El análisis se realiza tras la recepción de la solicitud. Para obtener información sobre el tamaño y el número de solicitudes que puede enviar por minuto y segundo, consulte la sección de [límites de datos](../overview.md#data-limits) de la introducción.

La API Text Analytics no tiene estado. No se almacenan datos en la cuenta y los resultados se devuelven inmediatamente en la respuesta.


### <a name="view-the-results"></a>View the results

El análisis de sentimiento devuelve una etiqueta de sentimiento y una puntuación de confianza para todo el documento, y para cada oración del mismo. Las puntuaciones más próximas a 1 indican una mayor confianza en la clasificación de la etiqueta, mientras que las puntuaciones inferiores indican una menor confianza. Un documento puede tener varias oraciones y las puntuaciones de confianza de cada documento u oración suman 1.

La salida se devuelve inmediatamente. Puede transmitir los resultados a una aplicación que acepte JSON o guardar la salida en un archivo en el sistema local. Después, importe el resultado en una aplicación que pueda usar para ordenar los datos, realizar búsquedas en ellos y manipularlos. Debido a la compatibilidad con varios idiomas y con Emojis, la respuesta puede contener desplazamientos de texto. Consulte [cómo procesar desplazamientos](../concepts/text-offsets.md) para más información.

#### <a name="version-30"></a>[Versión 3.0](#tab/version-3)

### <a name="sentiment-analysis-v30-example-response"></a>Respuesta del ejemplo de la versión 3.0 de Análisis de sentimiento

Las respuestas de la versión 3 de Análisis de sentimiento contienen puntuaciones y etiquetas de opinión de cada oración y documento analizados.

```json
{
    "documents": [
        {
            "id": "1",
            "sentiment": "positive",
            "confidenceScores": {
                "positive": 1.0,
                "neutral": 0.0,
                "negative": 0.0
            },
            "sentences": [
                {
                    "sentiment": "positive",
                    "confidenceScores": {
                        "positive": 1.0,
                        "neutral": 0.0,
                        "negative": 0.0
                    },
                    "offset": 0,
                    "length": 58,
                    "text": "The restaurant had great food and our waiter was friendly."
                }
            ],
            "warnings": []
        }
    ],
    "errors": [],
    "modelVersion": "2020-04-01"
}
```

#### <a name="version-31-preview1"></a>[Versión 3.1-preview.1](#tab/version-3-1)

### <a name="sentiment-analysis-v31-example-response"></a>Respuesta del ejemplo de la versión 3.1 de Análisis de sentimiento

Análisis de sentimiento v3.1 ofrece minería de opiniones, además del objeto de respuesta en la pestaña **Version 3.0**. En la respuesta siguiente, la oración *The restaurant had great food and our waiter was friendly* tiene dos aspectos: *food* y *waiter*. La propiedad `relations` de cada aspecto contiene un valor `ref` con la referencia del identificador URI a los objetos `documents`, `sentences` y `opinions` asociados.

```json
{
    "documents": [
        {
            "id": "1",
            "sentiment": "positive",
            "confidenceScores": {
                "positive": 1.0,
                "neutral": 0.0,
                "negative": 0.0
            },
            "sentences": [
                {
                    "sentiment": "positive",
                    "confidenceScores": {
                        "positive": 1.0,
                        "neutral": 0.0,
                        "negative": 0.0
                    },
                    "offset": 0,
                    "length": 58,
                    "text": "The restaurant had great food and our waiter was friendly.",
                    "aspects": [
                        {
                            "sentiment": "positive",
                            "confidenceScores": {
                                "positive": 1.0,
                                "negative": 0.0
                            },
                            "offset": 25,
                            "length": 4,
                            "text": "food",
                            "relations": [
                                {
                                    "relationType": "opinion",
                                    "ref": "#/documents/0/sentences/0/opinions/0"
                                }
                            ]
                        },
                        {
                            "sentiment": "positive",
                            "confidenceScores": {
                                "positive": 1.0,
                                "negative": 0.0
                            },
                            "offset": 38,
                            "length": 6,
                            "text": "waiter",
                            "relations": [
                                {
                                    "relationType": "opinion",
                                    "ref": "#/documents/0/sentences/0/opinions/1"
                                }
                            ]
                        }
                    ],
                    "opinions": [
                        {
                            "sentiment": "positive",
                            "confidenceScores": {
                                "positive": 1.0,
                                "negative": 0.0
                            },
                            "offset": 19,
                            "length": 5,
                            "text": "great",
                            "isNegated": false
                        },
                        {
                            "sentiment": "positive",
                            "confidenceScores": {
                                "positive": 1.0,
                                "negative": 0.0
                            },
                            "offset": 49,
                            "length": 8,
                            "text": "friendly",
                            "isNegated": false
                        }
                    ]
                }
            ],
            "warnings": []
        }
    ],
    "errors": [],
    "modelVersion": "2020-04-01"
}
```

---

## <a name="summary"></a>Resumen

En este artículo, ha aprendido los conceptos y el flujo de trabajo del análisis de sentimiento mediante la API Text Analytics. En resumen:

+ Análisis de sentimiento está disponible para determinados idiomas.
+ Los documentos JSON del cuerpo de la solicitud incluyen un identificador, un texto y el código de idioma.
+ La solicitud POST se realiza a un punto de conexión `/sentiment`, mediante una [clave de acceso y un punto de conexión](../../cognitive-services-apis-create-account.md#get-the-keys-for-your-resource) personalizados, que sean válidos para la suscripción.
+ La salida de la respuesta, que consta de una puntuación de las opiniones de cada identificador de documento, se puede transmitir en secuencias a cualquier aplicación que acepte JSON. Por ejemplo, Excel y Power BI.

## <a name="see-also"></a>Consulte también

* [Información general de Text Analytics](../overview.md)
* [Uso de la biblioteca cliente de Text Analytics](../quickstarts/text-analytics-sdk.md)
* [Novedades](../whats-new.md)
