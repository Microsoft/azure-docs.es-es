---
title: 'Inicio rápido: Extracción de datos de recibos con cURL: Form Recognizer'
titleSuffix: Azure Cognitive Services
description: En este inicio rápido, usará la API REST de Form Recognizer con cURL para extraer datos de imágenes de recibos de compras de EE. UU.
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: forms-recognizer
ms.topic: quickstart
ms.date: 05/27/2020
ms.author: pafarley
ms.openlocfilehash: 9c733c3c21d76ea6b9b13ac19c349c26b1ea1b4a
ms.sourcegitcommit: 4e5560887b8f10539d7564eedaff4316adb27e2c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87902163"
---
# <a name="quickstart-extract-receipt-data-using-the-form-recognizer-rest-api-with-curl"></a>Inicio rápido: Extracción de datos de recibos con API REST Form Recognizer con cURL

En este inicio rápido, usará la API REST de Azure Form Recognizer con cURL para extraer e identificar la información pertinente de los recibos de compras de EE. UU.

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="prerequisites"></a>Requisitos previos

Para completar este inicio rápido, debe cumplir los siguientes requisitos:
- [cURL](https://curl.haxx.se/windows/) instalado.
- Una dirección URL para la imagen de un recibo. En este inicio rápido, puede usar una [imagen de ejemplo](https://raw.githubusercontent.com/Azure-Samples/cognitive-services-REST-api-samples/master/curl/form-recognizer/contoso-allinone.jpg).

## <a name="create-a-form-recognizer-resource"></a>Creación de un recurso de Form Recognizer

[!INCLUDE [create resource](../includes/create-resource.md)]

## <a name="analyze-a-receipt"></a>Análisis de un recibo

Para empezar a analizar un recibo, llame a la API **[Analyze Receipt](https://westus2.dev.cognitive.microsoft.com/docs/services/form-recognizer-api-v2/operations/AnalyzeReceiptAsync)** con el comando cURL siguiente. Antes de ejecutar el comando, realice estos cambios:

1. Reemplace `<Endpoint>` por el punto de conexión que obtuvo con la suscripción de Form Recognizer.
1. Reemplace `<your receipt URL>` por la dirección URL de una imagen del recibo.
1. Reemplace `<subscription key>` con la clave de suscripción que copió en el paso anterior.

```bash
curl -i -X POST "https://<Endpoint>/formrecognizer/v2.0/prebuilt/receipt/analyze" -H "Content-Type: application/json" -H "Ocp-Apim-Subscription-Key: <subscription key>" --data-ascii "{ \"source\": \"<your receipt URL>\"}"
```

Recibirá una respuesta `202 (Success)` que incluye un encabezado **Operation-Location**. El valor de este encabezado contiene un identificador de operación que puede usar para consultar el estado de la operación asincrónica y obtener los resultados. En el siguiente ejemplo, la cadena después de `operations/` es el identificador de operación.

```console
https://cognitiveservice/formrecognizer/v2.0/prebuilt/receipt/operations/54f0b076-4e38-43e5-81bd-b85b8835fdfb
```

## <a name="get-the-receipt-results"></a>Obtención de los resultados del recibo

Tras la llamada a la API **Analyze Receipt**, llame a la API **[Get Receipt Result](https://westus2.dev.cognitive.microsoft.com/docs/services/form-recognizer-api-v2/operations/GetAnalyzeReceiptResult)** para obtener el estado de la operación y los datos extraídos. Antes de ejecutar el comando, realice estos cambios:

1. Reemplace `<Endpoint>` por el punto de conexión que obtuvo con la clave de suscripción de Form Recognizer. Lo encontrará en la pestaña **Información general** del recurso Form Recognizer.
1. Reemplace `<operationId>` por el identificador de operación del paso anterior.
1. Reemplace `<subscription key>` por la clave de suscripción.

```bash
curl -X GET "https://<Endpoint>/formrecognizer/v2.0/prebuilt/receipt/analyzeResults/<operationId>" -H "Ocp-Apim-Subscription-Key: <subscription key>"
```

### <a name="examine-the-response"></a>Examen de la respuesta

Recibirá una respuesta `200 (Success)` con la salida JSON. El primer campo, `"status"`, indica el estado de la operación. Si la operación se ha completado, e campo `"recognitionResults"` contiene todas las líneas de texto que se ha extrajeron del recibo y el campo `"understandingResults"` contiene información de del par clave-valor de las partes más importantes del recibo. Si la operación no está completa, el valor de `"status"` será `"running"` o `"notStarted"`, y debe llamar a la API de nuevo, ya sea manualmente o a través de un script. Se recomienda un intervalo de uno o varios segundos entre llamadas.

Consulte la siguiente imagen del recibo y su correspondiente salida JSON. La salida se ha abreviado para que se pueda leer mejor.

![Un recibo del almacén de Contoso](../media/contoso-allinone.jpg)

El nodo `"recognitionResults"` contiene todo el texto reconocido. El texto se organiza por página, después, por líneas y, finalmente, por palabras individuales. El nodo `"understandingResults"` contiene los valores específicos del recibo que el modelo ha descubierto. Aquí encontrará pares clave-valor útiles como los impuestos, el total, la dirección del vendedor, etc.

```json
{ 
  "status":"succeeded",
  "createdDateTime":"2019-12-17T04:11:24Z",
  "lastUpdatedDateTime":"2019-12-17T04:11:32Z",
  "analyzeResult":{ 
    "version":"2.0.0",
    "readResults":[ 
      { 
        "page":1,
        "angle":0.6893,
        "width":1688,
        "height":3000,
        "unit":"pixel",
        "language":"en",
        "lines":[ 
          { 
            "text":"Contoso",
            "boundingBox":[ 
              635,
              510,
              1086,
              461,
              1098,
              558,
              643,
              604
            ],
            "words":[ 
              { 
                "text":"Contoso",
                "boundingBox":[ 
                  639,
                  510,
                  1087,
                  461,
                  1098,
                  551,
                  646,
                  604
                ],
                "confidence":0.955
              }
            ]
          },
          ...
        ]
      }
    ],
    "documentResults":[ 
      { 
        "docType":"prebuilt:receipt",
        "pageRange":[ 
          1,
          1
        ],
        "fields":{ 
          "ReceiptType":{ 
            "type":"string",
            "valueString":"Itemized",
            "confidence":0.692
          },
          "MerchantName":{ 
            "type":"string",
            "valueString":"Contoso Contoso",
            "text":"Contoso Contoso",
            "boundingBox":[ 
              378.2,
              292.4,
              1117.7,
              468.3,
              1035.7,
              812.7,
              296.3,
              636.8
            ],
            "page":1,
            "confidence":0.613,
            "elements":[ 
              "#/readResults/0/lines/0/words/0",
              "#/readResults/0/lines/1/words/0"
            ]
          },
          "MerchantAddress":{ 
            "type":"string",
            "valueString":"123 Main Street Redmond, WA 98052",
            "text":"123 Main Street Redmond, WA 98052",
            "boundingBox":[ 
              302,
              675.8,
              848.1,
              793.7,
              809.9,
              970.4,
              263.9,
              852.5
            ],
            "page":1,
            "confidence":0.99,
            "elements":[ 
              "#/readResults/0/lines/2/words/0",
              "#/readResults/0/lines/2/words/1",
              "#/readResults/0/lines/2/words/2",
              "#/readResults/0/lines/3/words/0",
              "#/readResults/0/lines/3/words/1",
              "#/readResults/0/lines/3/words/2"
            ]
          },
          "MerchantPhoneNumber":{ 
            "type":"phoneNumber",
            "valuePhoneNumber":"+19876543210",
            "text":"987-654-3210",
            "boundingBox":[ 
              278,
              1004,
              656.3,
              1054.7,
              646.8,
              1125.3,
              268.5,
              1074.7
            ],
            "page":1,
            "confidence":0.99,
            "elements":[ 
              "#/readResults/0/lines/4/words/0"
            ]
          },
          "TransactionDate":{ 
            "type":"date",
            "valueDate":"2019-06-10",
            "text":"6/10/2019",
            "boundingBox":[ 
              265.1,
              1228.4,
              525,
              1247,
              518.9,
              1332.1,
              259,
              1313.5
            ],
            "page":1,
            "confidence":0.99,
            "elements":[ 
              "#/readResults/0/lines/5/words/0"
            ]
          },
          "TransactionTime":{ 
            "type":"time",
            "valueTime":"13:59:00",
            "text":"13:59",
            "boundingBox":[ 
              541,
              1248,
              677.3,
              1261.5,
              668.9,
              1346.5,
              532.6,
              1333
            ],
            "page":1,
            "confidence":0.977,
            "elements":[ 
              "#/readResults/0/lines/5/words/1"
            ]
          },
          "Items":{ 
            "type":"array",
            "valueArray":[ 
              { 
                "type":"object",
                "valueObject":{ 
                  "Quantity":{ 
                    "type":"number",
                    "text":"1",
                    "boundingBox":[ 
                      245.1,
                      1581.5,
                      300.9,
                      1585.1,
                      295,
                      1676,
                      239.2,
                      1672.4
                    ],
                    "page":1,
                    "confidence":0.92,
                    "elements":[ 
                      "#/readResults/0/lines/7/words/0"
                    ]
                  },
                  "Name":{ 
                    "type":"string",
                    "valueString":"Cappuccino",
                    "text":"Cappuccino",
                    "boundingBox":[ 
                      322,
                      1586,
                      654.2,
                      1601.1,
                      650,
                      1693,
                      317.8,
                      1678
                    ],
                    "page":1,
                    "confidence":0.923,
                    "elements":[ 
                      "#/readResults/0/lines/7/words/1"
                    ]
                  },
                  "TotalPrice":{ 
                    "type":"number",
                    "valueNumber":2.2,
                    "text":"$2.20",
                    "boundingBox":[ 
                      1107.7,
                      1584,
                      1263,
                      1574,
                      1268.3,
                      1656,
                      1113,
                      1666
                    ],
                    "page":1,
                    "confidence":0.918,
                    "elements":[ 
                      "#/readResults/0/lines/8/words/0"
                    ]
                  }
                }
              },
              ...
            ]
          },
          "Subtotal":{ 
            "type":"number",
            "valueNumber":11.7,
            "text":"11.70",
            "boundingBox":[ 
              1146,
              2221,
              1297.3,
              2223,
              1296,
              2319,
              1144.7,
              2317
            ],
            "page":1,
            "confidence":0.955,
            "elements":[ 
              "#/readResults/0/lines/13/words/1"
            ]
          },
          "Tax":{ 
            "type":"number",
            "valueNumber":1.17,
            "text":"1.17",
            "boundingBox":[ 
              1190,
              2359,
              1304,
              2359,
              1304,
              2456,
              1190,
              2456
            ],
            "page":1,
            "confidence":0.979,
            "elements":[ 
              "#/readResults/0/lines/15/words/1"
            ]
          },
          "Tip":{ 
            "type":"number",
            "valueNumber":1.63,
            "text":"1.63",
            "boundingBox":[ 
              1094,
              2479,
              1267.7,
              2485,
              1264,
              2591,
              1090.3,
              2585
            ],
            "page":1,
            "confidence":0.941,
            "elements":[ 
              "#/readResults/0/lines/17/words/1"
            ]
          },
          "Total":{ 
            "type":"number",
            "valueNumber":14.5,
            "text":"$14.50",
            "boundingBox":[ 
              1034.2,
              2617,
              1387.5,
              2638.2,
              1380,
              2763,
              1026.7,
              2741.8
            ],
            "page":1,
            "confidence":0.985,
            "elements":[ 
              "#/readResults/0/lines/19/words/0"
            ]
          }
        }
      }
    ]
  }
}
```

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, usará la API REST de Form Recognizer con cURL para extraer el contenido de un recibo de ventas. A continuación, consulte la documentación de referencia para explorar la API de Form Recognizer con mayor profundidad.

> [!div class="nextstepaction"]
> [Documentación de referencia de API REST](https://westus2.dev.cognitive.microsoft.com/docs/services/form-recognizer-api-v2/operations/AnalyzeReceiptAsync)
