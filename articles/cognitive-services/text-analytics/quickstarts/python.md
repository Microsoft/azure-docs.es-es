---
title: 'Inicio rápido: Uso de Python para llamar a Text Analytics API'
titleSuffix: Azure Cognitive Services
description: En este inicio rápido se muestra cómo obtener información y ejemplos de código que le ayuden a empezar a usar rápidamente la API Text Analytics en Azure Cognitive Services.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: text-analytics
ms.topic: quickstart
ms.date: 07/20/2020
ms.author: aahi
ms.custom: devx-track-python
ms.openlocfilehash: 34d1b62ed97b966c000ff81e8f7676c30338b6a1
ms.sourcegitcommit: dea88d5e28bd4bbd55f5303d7d58785fad5a341d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87876774"
---
# <a name="quickstart-using-the-python-rest-api-to-call-the-text-analytics-cognitive-service"></a>Inicio rápido: Uso de la API REST de Python para llamar a Text Analytics de Cognitive Services 
<a name="HOLTop"></a>

Use esta guía de inicio rápido para empezar a analizar el lenguaje con la API REST de Text Analytics y Python. En este artículo se muestra cómo [detectar el idioma](#Detect), [analizar las opiniones](#SentimentAnalysis), [extraer las frases clave](#KeyPhraseExtraction) e [identificar las entidades vinculadas](#Entities).

[!INCLUDE [text-analytics-api-references](../includes/text-analytics-api-references.md)]

## <a name="prerequisites"></a>Requisitos previos

* [Python 3.x](https://python.org)

* Biblioteca de solicitudes de Python
    
    Puede instalar la biblioteca con este comando:

    ```console
    pip install --upgrade requests
    ```

[!INCLUDE [cognitive-services-text-analytics-signup-requirements](../../../../includes/cognitive-services-text-analytics-signup-requirements.md)]


## <a name="create-a-new-python-application"></a>Creación de una nueva aplicación de Python

Cree una aplicación de Python en su editor o IDE favoritos. Agregue las importaciones siguientes al archivo.

```python
import requests
# pprint is used to format the JSON response
from pprint import pprint
```

Cree variables para la clave de suscripción y el punto de conexión de Azure del recurso.
    
```python
import os

subscription_key = "<paste-your-text-analytics-key-here>"
endpoint = "<paste-your-text-analytics-endpoint-here>"
```

En las siguientes secciones se describe cómo llamar a cada una de las características de API.

<a name="Detect"></a>

## <a name="detect-languages"></a>Detección de idiomas

Anexe `/text/analytics/v3.0/languages` al punto de conexión de base de Text Analytics para formar la dirección URL de detección de idioma. Por ejemplo: `https://<your-custom-subdomain>.cognitiveservices.azure.com/text/analytics/v3.0/languages`
    
```python
language_api_url = endpoint + "/text/analytics/v3.0/languages"
```

La carga a la API consta de una lista de `documents`, que son tuplas que contienen `id` y un atributo `text`. El atributo `text` almacena el texto que se va a analizar y el `id` puede ser cualquier valor. 

```python
documents = {"documents": [
    {"id": "1", "text": "This is a document written in English."},
    {"id": "2", "text": "Este es un document escrito en Español."},
    {"id": "3", "text": "这是一个用中文写的文件"}
]}
```

Utilice la biblioteca de solicitudes para enviar los documentos a la API. Agregue su clave de suscripción al encabezado `Ocp-Apim-Subscription-Key` y devuelva la solicitud con `requests.post()`. 

```python
headers = {"Ocp-Apim-Subscription-Key": subscription_key}
response = requests.post(language_api_url, headers=headers, json=documents)
languages = response.json()
pprint(languages)
```

### <a name="output"></a>Output

```json
{
    "documents": [
        {
            "id": "1",
            "detectedLanguage": {
                "name": "English",
                "iso6391Name": "en",
                "confidenceScore": 1.0
            },
            "warnings": []
        },
        {
            "id": "2",
            "detectedLanguage": {
                "name": "Spanish",
                "iso6391Name": "es",
                "confidenceScore": 1.0
            },
            "warnings": []
        },
        {
            "id": "3",
            "detectedLanguage": {
                "name": "Chinese_Simplified",
                "iso6391Name": "zh_chs",
                "confidenceScore": 1.0
            },
            "warnings": []
        }
    ],
    "errors": [],
    "modelVersion": "2019-10-01"
}
```

<a name="SentimentAnalysis"></a>

## <a name="analyze-sentiment"></a>Análisis de opinión

Para detectar la opinión (que oscila entre positiva o negativa) de un conjunto de documentos, anexe `/text/analytics/v3.0/sentiment` al punto de conexión de base de Text Analytics para formar la dirección URL de detección de idioma. Por ejemplo: `https://<your-custom-subdomain>.cognitiveservices.azure.com/text/analytics/v3.0/sentiment`
    
```python
sentiment_url = endpoint + "/text/analytics/v3.0/sentiment"
```

Como en el ejemplo de detección de idioma, cree un diccionario con una clave `documents` que consta de una lista de documentos. Cada documento es una tupla que tiene el `id`, el `text` que se va a analizar y el `language` del texto. 

```python
documents = {"documents": [
    {"id": "1", "language": "en",
        "text": "I really enjoy the new XBox One S. It has a clean look, it has 4K/HDR resolution and it is affordable."},
    {"id": "2", "language": "es",
        "text": "Este ha sido un dia terrible, llegué tarde al trabajo debido a un accidente automobilistico."}
]}
```

Utilice la biblioteca de solicitudes para enviar los documentos a la API. Agregue su clave de suscripción al encabezado `Ocp-Apim-Subscription-Key` y devuelva la solicitud con `requests.post()`. 

```python
headers = {"Ocp-Apim-Subscription-Key": subscription_key}
response = requests.post(sentiment_url, headers=headers, json=documents)
sentiments = response.json()
pprint(sentiments)
```

### <a name="output"></a>Output

La puntuación de las opiniones de un documento oscila entre 0,0 y 1,0: una puntuación más alta indica una opinión más positiva.

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
                    "length": 102,
                    "text": "I really enjoy the new XBox One S. It has a clean look, it has 4K/HDR resolution and it is affordable."
                }
            ],
            "warnings": []
        },
        {
            "id": "2",
            "sentiment": "negative",
            "confidenceScores": {
                "positive": 0.02,
                "neutral": 0.05,
                "negative": 0.93
            },
            "sentences": [
                {
                    "sentiment": "negative",
                    "confidenceScores": {
                        "positive": 0.02,
                        "neutral": 0.05,
                        "negative": 0.93
                    },
                    "offset": 0,
                    "length": 92,
                    "text": "Este ha sido un dia terrible, llegué tarde al trabajo debido a un accidente automobilistico."
                }
            ],
            "warnings": []
        }
    ],
    "errors": [],
    "modelVersion": "2020-04-01"
}
```

<a name="KeyPhraseExtraction"></a>

## <a name="extract-key-phrases"></a>Extracción de frases clave
 
Para extraer las frases clave de un conjunto de documentos, anexe `/text/analytics/v3.0/keyPhrases` al punto de conexión de base de Text Analytics para formar la dirección URL de detección de idioma. Por ejemplo: `https://<your-custom-subdomain>.cognitiveservices.azure.com/text/analytics/v3.0/keyPhrases`
    
```python
keyphrase_url = endpoint + "/text/analytics/v3.0/keyphrases"
```

Esta colección de documentos es igual a la que se usó para el ejemplo del análisis de opiniones.

```python
documents = {"documents": [
    {"id": "1", "language": "en",
        "text": "I really enjoy the new XBox One S. It has a clean look, it has 4K/HDR resolution and it is affordable."},
    {"id": "2", "language": "es",
        "text": "Si usted quiere comunicarse con Carlos, usted debe de llamarlo a su telefono movil. Carlos es muy responsable, pero necesita recibir una notificacion si hay algun problema."},
    {"id": "3", "language": "en",
        "text": "The Grand Hotel is a new hotel in the center of Seattle. It earned 5 stars in my review, and has the classiest decor I've ever seen."}
]}
```

Utilice la biblioteca de solicitudes para enviar los documentos a la API. Agregue su clave de suscripción al encabezado `Ocp-Apim-Subscription-Key` y devuelva la solicitud con `requests.post()`. 

```python
headers = {"Ocp-Apim-Subscription-Key": subscription_key}
response = requests.post(keyphrase_url, headers=headers, json=documents)
key_phrases = response.json()
pprint(key_phrases)
```

### <a name="output"></a>Output

```json
{
    "documents": [
        {
            "id": "1",
            "keyPhrases": [
                "HDR resolution",
                "new XBox",
                "clean look"
            ],
            "warnings": []
        },
        {
            "id": "2",
            "keyPhrases": [
                "Carlos",
                "notificacion",
                "algun problema",
                "telefono movil"
            ],
            "warnings": []
        },
        {
            "id": "3",
            "keyPhrases": [
                "new hotel",
                "Grand Hotel",
                "review",
                "center of Seattle",
                "classiest decor",
                "stars"
            ],
            "warnings": []
        }
    ],
    "errors": [],
    "modelVersion": "2019-10-01"
}
```

<a name="Entities"></a>

## <a name="identify-entities"></a>Identificación de entidades

Para identificar las entidades conocidas (personas, lugares y cosas) en documentos de texto, anexe `/text/analytics/v3.0/entities/recognition/general` al punto de conexión de base de Text Analytics para formar la dirección URL de detección de idioma. Por ejemplo: `https://<your-custom-subdomain>.cognitiveservices.azure.com/text/analytics/v3.0/entities/recognition/general`
    
```python
entities_url = endpoint + "/text/analytics/v3.0/entities/recognition/general"
```

Cree una colección de documentos, como en los ejemplos anteriores. 

```python
documents = {"documents": [
    {"id": "1", "text": "Microsoft is an It company."}
]}
```

Utilice la biblioteca de solicitudes para enviar los documentos a la API. Agregue su clave de suscripción al encabezado `Ocp-Apim-Subscription-Key` y devuelva la solicitud con `requests.post()`.

```python
headers = {"Ocp-Apim-Subscription-Key": subscription_key}
response = requests.post(entities_url, headers=headers, json=documents)
entities = response.json()
pprint(entities)
```

### <a name="output"></a>Output

```json
{
    "documents": [
        {
            "id": "1",
            "entities": [
                {
                    "text": "Microsoft",
                    "category": "Organization",
                    "offset": 0,
                    "length": 9,
                    "confidenceScore": 0.86
                },
                {
                    "text": "IT",
                    "category": "Skill",
                    "offset": 16,
                    "length": 2,
                    "confidenceScore": 0.8
                }
            ],
            "warnings": []
        }
    ],
    "errors": [],
    "modelVersion": "2020-04-01"
}
```

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Text Analytics con Power BI](../tutorials/tutorial-power-bi-key-phrases.md)

## <a name="see-also"></a>Consulte también 

 [Información general de Text Analytics](../overview.md)  
 [Preguntas más frecuentes](../text-analytics-resource-faq.md)
