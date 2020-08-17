---
title: archivo de inclusión
description: archivo de inclusión
services: cognitive-services
manager: nitinme
ms.service: cognitive-services
ms.subservice: luis
ms.topic: include
ms.custom: include file
ms.date: 04/27/2020
ms.openlocfilehash: fabd79829425147667c46f686a1ec1ceb6a29b00
ms.sourcegitcommit: 0e8a4671aa3f5a9a54231fea48bcfb432a1e528c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/24/2020
ms.locfileid: "87132910"
---
En esta guía de inicio rápido basada en Postman se ofrece orientación sobre cómo obtener una respuesta de una base de conocimiento.

## <a name="prerequisites"></a>Requisitos previos

* Última versión de [**Postman**](https://www.getpostman.com/).
* Debe disponer de lo siguiente:
    * Un [servicio QnA Maker](../How-To/set-up-qnamaker-service-azure.md)
    * Una [base de conocimiento entrenada y publicada con preguntas y respuestas](../Quickstarts/add-question-metadata-portal.md) creada en un inicio rápido y configurada con metadatos y charla.

> [!NOTE]
> Cuando esté listo para generar una respuesta a una pregunta de la base de conocimiento, debe [entrenar](../Quickstarts/create-publish-knowledge-base.md#save-and-train) y [publicar](../Quickstarts/create-publish-knowledge-base.md#publish-the-knowledge-base) la base de conocimiento. Cuando se publica la base de conocimiento, en la página **Publicar** se muestra la configuración de la solicitud HTTP para generar una respuesta. La pestaña **Postman** muestra la configuración necesaria para generar una respuesta.

## <a name="set-up-postman-for-requests"></a>Configuración de Postman para solicitudes

En este inicio rápido se usa la misma configuración que para la solicitud **POST** de Postman y, a continuación, se configura en el código JSON de la sección body de la solicitud POST que se envía al servicio en función de lo que se intenta consultar.

Use este procedimiento para configurar Postman y, luego, lea todas las secciones posteriores para configurar el código JSON de la sección body de la solicitud POST.

1. En la página **Configuración** de la base de conocimiento, seleccione la pestaña **Postman** para ver la configuración que se usa para generar una respuesta de la base de conocimiento. Copie la siguiente información para usarla en Postman.

    |Nombre|Configuración|Propósito y valor|
    |--|--|--|
    |`POST`| `/knowledgebases/replace-with-your-knowledge-base-id/generateAnswer`|Este es el método HTTP y la ruta de la dirección URL.|
    |`Host`|`https://YOUR-RESOURCE_NAME.azurewebsites.net/qnamaker`|Este es el host de la dirección URL. Concatene los valores de Host y Post para obtener la dirección URL completa de generateAnswer.|
    |`Authorization`|`EndpointKey xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx`|El valor del encabezado para autorizar la solicitud en Azure. |
    |`Content-type`|`application/json`|El valor del encabezado para el contenido.|
    ||`{"question":"<Your question>"}`|El cuerpo de la solicitud POST como un objeto JSON. Este valor cambiará en cada sección posterior, en función de aquello para lo que esté pensada la consulta.|

1. Abra Postman y cree una nueva solicitud **POST** básica con la configuración de la base de conocimiento publicada. En las secciones siguientes, modifique el código JSON de la sección body de la solicitud POST para cambiar la consulta a la base de conocimiento.

## <a name="use-metadata-to-filter-answer"></a>Uso de metadatos para filtrar las respuestas

En un inicio rápido anterior, se agregaron los metadatos a dos pares de QnA para distinguir entre dos preguntas diferentes. Agregue los metadatos a la consulta para restringir el filtro solo al par de QnA pertinente.

1. En Postman, cambie solo el código JSON de la consulta agregando la propiedad `strictFilters` con el par nombre-valor de `service:qna_maker`. El código JSON del cuerpo debe ser:

    ```json
    {
        'question':'size',
        'strictFilters': [
            {
                'name':'service','value':'qna_maker'
            }
        ]
    }
    ```

    La pregunta es solo una palabra, `size`, que puede devolver cualquiera de los dos pares de preguntas y respuestas. La matriz `strictFilters` indica a la respuesta que se reduzca a solo las contestaciones de `qna_maker`.

1. La respuesta incluye solo la contestación que cumple los criterios de filtro.

    A la siguiente respuesta se le ha cambiado el formato para mayor legibilidad:

    ```JSON
    {
        "answers": [
            {
                "questions": [
                    "How large a knowledge base can I create?",
                    "What is the max size of a knowledge base?",
                    "How many GB of data can a knowledge base hold?"
                ],
                "answer": "The size of the knowledge base depends on the SKU of Azure search you choose when creating the QnA Maker service. Read [here](https://docs.microsoft.com/azure/cognitive-services/qnamaker/tutorials/choosing-capacity-qnamaker-deployment) for more details.",
                "score": 68.76,
                "id": 3,
                "source": "https://docs.microsoft.com/azure/cognitive-services/qnamaker/troubleshooting",
                "metadata": [
                    {
                        "name": "link_in_answer",
                        "value": "true"
                    },
                    {
                        "name": "service",
                        "value": "qna_maker"
                    }
                ],
                "context": {
                    "isContextOnly": false,
                    "prompts": []
                }
            }
        ],
        "debugInfo": null
    }
    ```

    Si hay un par de preguntas y respuestas que no cumple el término de búsqueda, pero sí el filtro, no se devolverá. En su lugar, se devuelve la respuesta general `No good match found in KB.`.

## <a name="use-debug-query-property"></a>Uso de la propiedad de consulta debug

> [!NOTE]
>No se recomienda usar la propiedad Debug con ninguna dependencia. Esta propiedad se ha agregado para ayudar al equipo del producto a solucionar problemas.

La información de depuración le ayuda a comprender cómo se determinó la respuesta devuelta. Aunque es útil, no es necesaria. Para generar una respuesta con la información de depuración, agregue la propiedad `debug`:

1. En Postman, cambie solo el código JSON de la sección body agregando la propiedad `debug`. El código JSON debe ser:

    ```json
    {
        'question':'size',
        'Debug': {
            'Enable':true
        }

    }
    ```

1. La respuesta incluye la información pertinente acerca de la respuesta. En la siguiente salida JSON, algunos detalles de la depuración se han reemplazado por puntos suspensivos.

    ```console
    {
        "answers": [
            {
                "questions": [
                    "How do I share a knowledge base with others?"
                ],
                "answer": "Sharing works at the level of a QnA Maker service, that is, all knowledge bases in the service will be shared. Read [here](https://docs.microsoft.com/azure/cognitive-services/qnamaker/how-to/collaborate-knowledge-base) how to collaborate on a knowledge base.",
                "score": 56.07,
                "id": 5,
                "source": "https://docs.microsoft.com/azure/cognitive-services/qnamaker/troubleshooting",
                "metadata": [],
                "context": {
                    "isContextOnly": false,
                    "prompts": []
                }
            }
        ],
        "debugInfo": {
            "userQuery": {
                "question": "How do I programmatically update my Knowledge Base?",
                "top": 1,
                "userId": null,
                "strictFilters": [],
                "isTest": false,
                "debug": {
                    "enable": true,
                    "recordL1SearchLatency": false,
                    "mockQnaL1Content": null
                },
                "rankerType": 0,
                "context": null,
                "qnaId": 0,
                "scoreThreshold": 0.0
            },
            "rankerInfo": {
                "specialFuzzyQuery": "how do i programmatically~6 update my knowledge base",
                "synonyms": "what s...",
                "rankerLanguage": "English",
                "rankerFileName": "https://qnamakerstore.blob.core.windows.net/qnamakerdata/rankers/ranker-English.ini",
                "rankersDirectory": "D:\\home\\site\\wwwroot\\Data\\QnAMaker\\rd0003ffa60fc45.24.0\\RankerData\\Rankers",
                "allQnAsfeatureValues": {
                    "WordnetSimilarity": {
                        "5": 0.54706300120043716,...
                    },
                    ...
                },
                "rankerVersion": "V2",
                "rankerModelType": "TreeEnsemble",
                "rankerType": 0,
                "indexResultsCount": 25,
                "reRankerResultsCount": 1
            },
            "runtimeVersion": "5.24.0",
            "indexDebugInfo": {
                "indexDefinition": {
                    "name": "064a4112-bd65-42e8-b01d-141c4c9cd09e",
                    "fields": [...
                    ],
                    "scoringProfiles": [],
                    "defaultScoringProfile": null,
                    "corsOptions": null,
                    "suggesters": [],
                    "analyzers": [],
                    "tokenizers": [],
                    "tokenFilters": [],
                    "charFilters": [],
                    "@odata.etag": "\"0x8D7A920EA5EE6FE\""
                },
                "qnaCount": 117,
                "parameters": {},
                "azureSearchResult": {
                    "continuationToken": null,
                    "@odata.count": null,
                    "@search.coverage": null,
                    "@search.facets": null,
                    "@search.nextPageParameters": null,
                    "value": [...],
                    "@odata.nextLink": null
                }
            },
            "l1SearchLatencyInMs": 0,
            "qnaL1Results": {...}
        },
        "activeLearningEnabled": true
    }
    ```

## <a name="use-test-knowledge-base"></a>Uso de la base de conocimiento de prueba

Si quiere obtener una respuesta de la base de conocimiento de prueba, use la propiedad de cuerpo `isTest`.

En Postman, cambie solo el código JSON de la sección body agregando la propiedad `isTest`. El código JSON debe ser:

```json
{
    'question':'size',
    'isTest': true
}
```

La respuesta JSON usa el mismo esquema que la consulta de la base de conocimiento publicada.

> [!NOTE]
> Aunque la prueba y las bases de conocimiento publicadas sean exactamente iguales, puede haber algunas pequeñas diferencias ya que el índice de pruebas se comparte entre todas las bases de conocimiento del recurso.

## <a name="query-for-a-chit-chat-answer"></a>Consulta de una respuesta de charla

1. En Postman, cambie solo el código JSON de la sección body a una instrucción de fin de conversación del usuario. El código JSON debe ser:

    ```json
    {
        'question':'thank you'
    }
    ```

1. La respuesta incluye la puntuación y la respuesta.

    ```json
    {
      "answers": [
          {
              "questions": [
                  "I thank you",
                  "Oh, thank you",
                  "My sincere thanks",
                  "My humblest thanks to you",
                  "Marvelous, thanks",
                  "Marvelous, thank you kindly",
                  "Marvelous, thank you",
                  "Many thanks to you",
                  "Many thanks",
                  "Kthx",
                  "I'm grateful, thanks",
                  "Ahh, thanks",
                  "I'm grateful for that, thank you",
                  "Perfecto, thanks",
                  "I appreciate you",
                  "I appreciate that",
                  "I appreciate it",
                  "I am very thankful for that",
                  "How kind, thank you",
                  "Great, thanks",
                  "Great, thank you",
                  "Gracias",
                  "Gotcha, thanks",
                  "Gotcha, thank you",
                  "Awesome thanks!",
                  "I'm grateful for that, thank you kindly",
                  "thank you pal",
                  "Wonderful, thank you!",
                  "Wonderful, thank you very much",
                  "Why thank you",
                  "Thx",
                  "Thnx",
                  "That's very kind",
                  "That's great, thanks",
                  "That is lovely, thanks",
                  "That is awesome, thanks!",
                  "Thanks bot",
                  "Thanks a lot",
                  "Okay, thanks!",
                  "Thank you so much",
                  "Perfect, thanks",
                  "Thank you my friend",
                  "Thank you kindly",
                  "Thank you for that",
                  "Thank you bot",
                  "Thank you",
                  "Right on, thanks very much",
                  "Right on, thanks a lot",
                  "Radical, thanks",
                  "Rad, thanks",
                  "Rad thank you",
                  "Wonderful, thanks!",
                  "Thanks"
              ],
              "answer": "You're welcome.",
              "score": 100.0,
              "id": 75,
              "source": "qna_chitchat_Professional.tsv",
              "metadata": [
                  {
                      "name": "editorial",
                      "value": "chitchat"
                  }
              ],
              "context": {
                  "isContextOnly": false,
                  "prompts": []
              }
          }
      ],
      "debugInfo": null,
      "activeLearningEnabled": true
    }
    ```

    Dado que la pregunta de `Thank you` coincidía exactamente con una pregunta de charla, QnA Maker está completamente seguro con la puntuación de 100. QnA Maker también ha devuelto todas las preguntas relacionadas, así como la propiedad metadata que contiene la información de la etiqueta de metadatos de charla.

## <a name="use-threshold-and-default-answer"></a>Uso de un umbral y una respuesta predeterminada

Puede solicitar un umbral mínimo para la respuesta. Si no se cumple el umbral, se devuelve la respuesta predeterminada.

1. En Postman, cambie solo el código JSON de la sección body a una instrucción de fin de conversación del usuario. El código JSON debe ser:

    ```json
    {
        'question':'size',
        'scoreThreshold':80.00
    }
    ```

    La base de conocimiento no encuentra esa respuesta ya que la puntuación de la pregunta es del 71 % y, en su lugar, devuelve la respuesta predeterminada que proporcionó al crear la base de conocimiento.

    La respuesta JSON devuelta, incluida la puntuación y la respuesta, es:

    ```json
    {
        "answers": [
            {
                "questions": [],
                "answer": "No good match found in KB.",
                "score": 0.0,
                "id": -1,
                "source": null,
                "metadata": []
            }
        ],
        "debugInfo": null,
        "activeLearningEnabled": true
    }
    ```

    QnA Maker ha devuelto una puntuación de `0`, lo que significa ninguna confianza. También ha devuelto la respuesta predeterminada.

1. Cambie el valor de umbral a 60 % y vuelva a realizar la consulta:

    ```json
    {
        'question':'size',
        'scoreThreshold':60.00
    }
    ```

    El código JSON devuelto encontró la respuesta.

    ```json
    {
        "answers": [
            {
                "questions": [
                    "How large a knowledge base can I create?",
                    "What is the max size of a knowledge base?",
                    "How many GB of data can a knowledge base hold?"
                ],
                "answer": "The size of the knowledge base depends on the SKU of Azure search you choose when creating the QnA Maker service. Read [here](https://docs.microsoft.com/azure/cognitive-services/qnamaker/tutorials/choosing-capacity-qnamaker-deployment) for more details.",
                "score": 71.1,
                "id": 3,
                "source": "https://docs.microsoft.com/azure/cognitive-services/qnamaker/troubleshooting",
                "metadata": [
                    {
                        "name": "link_in_answer",
                        "value": "true"
                    },
                    {
                        "name": "server",
                        "value": "qna_maker"
                    }
                ],
                "context": {
                    "isContextOnly": false,
                    "prompts": []
                }
            }
        ],
        "debugInfo": null,
        "activeLearningEnabled": true
    }
    ```
