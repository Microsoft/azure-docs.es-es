---
title: 'Inicio rápido: Consulta de predicciones con el explorador: LUIS'
description: En este inicio rápido, usará una aplicación de LUIS disponible públicamente para determinar la intención de un usuario a partir de texto conversacional en un explorador.
ms.topic: quickstart
ms.date: 04/21/2020
ms.openlocfilehash: 24c7a8aca410ad10b8a9299c4bca2f13d0feb780
ms.sourcegitcommit: c293217e2d829b752771dab52b96529a5442a190
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/15/2020
ms.locfileid: "88246002"
---
# <a name="quickstart-query-prediction-runtime-with-user-text"></a>Inicio rápido: Consulta del entorno de ejecución de predicción con texto del usuario

Para entender lo que devuelve un punto de conexión de predicción de LUIS, vea un resultado de predicción en un explorador web.

## <a name="prerequisites"></a>Prerrequisitos

Para consultar una aplicación pública, necesita:

* La información de recursos de Language Understanding (LUIS):
    * **Clave de predicción**, que se puede obtener en el [portal de LUIS](https://www.luis.ai/). Si aún no tiene una suscripción para crear una clave, puede registrarse para obtener una [cuenta gratuita](https://azure.microsoft.com/free/cognitive-services).
    * **Subdominio de punto de conexión de predicción**: el subdominio es también el **nombre** del recurso de LUIS.
* Un identificador de aplicación de LUIS: use el identificador de aplicación de IoT público de `df67dcdb-c37d-46af-88e1-8b97951ca1c2`. La consulta de usuario que se usa en el código del inicio rápido es específica de esa aplicación.

## <a name="use-the-browser-to-see-predictions"></a>Uso del explorador para ver las predicciones

1. Abra un explorador web.
1. Use las direcciones URL completas que se indican a continuación y reemplace `YOUR-KEY` por su propia clave de predicción de LUIS. Las solicitudes son solicitudes GET e incluyen la autorización, con la clave de predicción de LUIS, como parámetro de la cadena de consulta.

    #### <a name="v3-prediction-request"></a>[Solicitud de predicción de V3](#tab/V3-1-1)


    El formato de la dirección URL de V3 para una solicitud de punto de conexión (por ranuras) **GET** es:

    `
    https://YOUR-LUIS-ENDPOINT-SUBDOMAIN.api.cognitive.microsoft.com/luis/prediction/v3.0/apps/df67dcdb-c37d-46af-88e1-8b97951ca1c2/slots/production/predict?query=turn on all lights&subscription-key=YOUR-LUIS-PREDICTION-KEY
    `

    #### <a name="v2-prediction-request"></a>[Solicitud de predicción de V2](#tab/V2-1-2)

    El formato de la dirección URL de V2 para una solicitud de punto de conexión **GET** es:

    `
    https://YOUR-LUIS-ENDPOINT-SUBDOMAIN.api.cognitive.microsoft.com/luis/v2.0/apps/df67dcdb-c37d-46af-88e1-8b97951ca1c2?subscription-key=YOUR-LUIS-PREDICTION-KEY&q=turn on all lights
    `

1. Pegue la dirección URL en una ventana del explorador y presione ENTRAR. El explorador muestra el resultado JSON que indica que LUIS detecta la intención `HomeAutomation.TurnOn` como intención principal y la entidad `HomeAutomation.Operation` con el valor `on`.

    #### <a name="v3-prediction-response"></a>[Respuesta de predicción de V3](#tab/V3-2-1)

    ```JSON
    {
        "query": "turn on all lights",
        "prediction": {
            "topIntent": "HomeAutomation.TurnOn",
            "intents": {
                "HomeAutomation.TurnOn": {
                    "score": 0.5375382
                }
            },
            "entities": {
                "HomeAutomation.Operation": [
                    "on"
                ]
            }
        }
    }
    ```

    #### <a name="v2-prediction-response"></a>[Respuesta de predicción de V2](#tab/V2-2-2)

    ```json
    {
        "query": "turn on all lights",
        "topScoringIntent": {
            "intent": "HomeAutomation.TurnOn",
            "score": 0.5375382
        },
        "entities": [
            {
                "entity": "on",
                "type": "HomeAutomation.Operation",
                "startIndex": 5,
                "endIndex": 6,
                "score": 0.724984169
            }
        ]
    }
    ```

    * * *

1. Para ver todas las intenciones, agregue el parámetro de cadena de consulta adecuado.

    #### <a name="v3-prediction-endpoint"></a>[Punto de conexión de predicción de V3](#tab/V3-3-1)

    Agregue `show-all-intents=true` al final de la cadena de QueryString para **mostrar todas las intenciones**:

    `
    https://YOUR-LUIS-ENDPOINT-SUBDOMAIN.api.cognitive.microsoft.com/luis/predict/v3.0/apps/df67dcdb-c37d-46af-88e1-8b97951ca1c2/slots/production/predict?query=turn on all lights&subscription-key=YOUR-LUIS-PREDICTION-KEY&show-all-intents=true
    `

    ```JSON
    {
        "query": "turn off the living room light",
        "prediction": {
            "topIntent": "HomeAutomation.TurnOn",
            "intents": {
                "HomeAutomation.TurnOn": {
                    "score": 0.5375382
                },
                "None": {
                    "score": 0.08687421
                },
                "HomeAutomation.TurnOff": {
                    "score": 0.0207554
                }
            },
            "entities": {
                "HomeAutomation.Operation": [
                    "on"
                ]
            }
        }
    }
    ```

    #### <a name="v2-prediction-endpoint"></a>[Punto de conexión de predicción de V2](#tab/V2)

    Agregue `verbose=true` al final de la cadena de QueryString para **mostrar todas las intenciones**:

    `
    https://YOUR-LUIS-ENDPOINT-SUBDOMAIN.api.cognitive.microsoft.com/luis/v2.0/apps/df67dcdb-c37d-46af-88e1-8b97951ca1c2?q=turn on all lights&subscription-key=YOUR-LUIS-PREDICTION-KEY&verbose=true
    `

    ```json
    {
        "query": "turn on all lights",
        "topScoringIntent": {
            "intent": "HomeAutomation.TurnOn",
            "score": 0.5375382
        },
        "intents": [
            {
                "intent": "HomeAutomation.TurnOn",
                "score": 0.5375382
            },
            {
                "intent": "None",
                "score": 0.08687421
            },
            {
                "intent": "HomeAutomation.TurnOff",
                "score": 0.0207554
            }
        ],
        "entities": [
            {
                "entity": "on",
                "type": "HomeAutomation.Operation",
                "startIndex": 5,
                "endIndex": 6,
                "score": 0.724984169
            }
        ]
    }
    ```

## <a name="next-steps"></a>Pasos siguientes

Más información sobre:
* [Punto de conexión de predicción de V3](luis-migration-api-v3.md)
* [Subdominios personalizados](../cognitive-services-custom-subdomains.md)

> [!div class="nextstepaction"]
> [Creación de una aplicación en el portal de LUIS](get-started-portal-build-app.md)
