---
title: 'Inicio rápido: Extracción de la información sobre el texto y el diseño mediante Python: Form Recognizer'
titleSuffix: Azure Cognitive Services
description: En este inicio rápido, usará la API REST de diseño de Form Recognizer con Python para leer datos de texto y tablas de los formularios.
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: forms-recognizer
ms.topic: quickstart
ms.date: 05/27/2020
ms.author: pafarley
ms.custom: devx-track-python
ms.openlocfilehash: 0c7e077b3f170e3d9f1b15f78d52a1f386396983
ms.sourcegitcommit: dea88d5e28bd4bbd55f5303d7d58785fad5a341d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87876876"
---
# <a name="quickstart-extract-text-and-layout-information-using-the-form-recognizer-rest-api-with-python"></a>Inicio rápido: Extracción de la información sobre el texto y el diseño mediante la API REST de Form Recognizer con Python

En este inicio rápido, utilizará la API REST de Azure Form Recognizer con Python para extraer la información sobre el texto y el diseño, y los datos de tablas de los formularios.

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="prerequisites"></a>Requisitos previos

Para completar este inicio rápido, debe cumplir los siguientes requisitos:
- [Python](https://www.python.org/downloads/) instalado (si desea ejecutar el ejemplo localmente).
- Un formulario. Puede descargar una imagen del [conjunto de datos de ejemplo](https://go.microsoft.com/fwlink/?linkid=2090451) para este inicio rápido.

> [!NOTE]
> En este inicio rápido se usa un documento almacenado de forma local. Para aprender a usar archivos remotos con acceso mediante la dirección URL, consulte la [documentación de referencia](https://westus2.dev.cognitive.microsoft.com/docs/services/form-recognizer-api-v2/operations/AnalyzeLayoutAsync).


## <a name="create-a-form-recognizer-resource"></a>Creación de un recurso de Form Recognizer

[!INCLUDE [create resource](../includes/create-resource.md)]

## <a name="analyze-the-form-layout"></a>Análisis del diseño del formulario

Para empezar a analizar el diseño, llame a la API **[Analyze Layout](https://westus2.dev.cognitive.microsoft.com/docs/services/form-recognizer-api-v2/operations/AnalyzeLayoutAsync)** mediante el siguiente script de Python. Antes de ejecutar el script, realice estos cambios:

1. Reemplace `<Endpoint>` por el punto de conexión que obtuvo con la suscripción de Form Recognizer.
1. Reemplace `<path to your form>` por la ruta de acceso al documento de formulario local.
1. Reemplace `<subscription key>` con la clave de suscripción que copió en el paso anterior.

    ```python
    ########### Python Form Recognizer Async Layout #############

    import json
    import time
    from requests import get, post
    
    # Endpoint URL
    endpoint = r"<Endpoint>"
    apim_key = "<Subscription Key>"
    post_url = endpoint + "/formrecognizer/v2.0/Layout/analyze"
    source = r"<path to your form>"
    
    headers = {
        # Request headers
        'Content-Type': 'application/json',
        'Ocp-Apim-Subscription-Key': apim_key,
    }
    with open(source, "rb") as f:
        data_bytes = f.read()
    
    try:
        resp = post(url = post_url, data = data_bytes, headers = headers)
        if resp.status_code != 202:
            print("POST analyze failed:\n%s" % resp.text)
            quit()
        print("POST analyze succeeded:\n%s" % resp.headers)
        get_url = resp.headers["operation-location"]
    except Exception as e:
        print("POST analyze failed:\n%s" % str(e))
        quit()
    ```

1. Guarde el código en un archivo con una extensión .py. Por ejemplo, *form-recognizer-layout.py*.
1. Abra una ventana de símbolo del sistema.
1. En el símbolo del sistema, utilice el comando `python` para ejecutar el ejemplo. Por ejemplo, `python form-recognizer-layout.py`.

Recibirá una respuesta `202 (Success)` que incluye un encabezado **Operation-Location**, que el script imprimirá en la consola. Este encabezado contiene un identificador de operación que puede usar para consultar el estado de la operación asincrónica y obtener los resultados. En el siguiente valor de ejemplo, la cadena después de `operations/` es el identificador de operación.

```console
https://cognitiveservice/formrecognizer/v2.0/layout/operations/54f0b076-4e38-43e5-81bd-b85b8835fdfb
```

## <a name="get-the-layout-results"></a>Obtención de los resultados del diseño

Tras la llamada a **Analyze Layout**, llame a la API **[Get Analyze Layout Result](https://westus2.dev.cognitive.microsoft.com/docs/services/form-recognizer-api-v2/operations/GetAnalyzeLayoutResult)** para obtener el estado de la operación y los datos extraídos. Agregue el código siguiente al final del script de Python. Este código usa el valor de identificador de la operación en una nueva llamada API. El script llama a la API a intervalos regulares hasta que los resultados están disponibles. Se recomienda un intervalo de uno o varios segundos.

```python
n_tries = 10
n_try = 0
wait_sec = 6
while n_try < n_tries:
    try:
        resp = get(url = get_url, headers = {"Ocp-Apim-Subscription-Key": apim_key})
        resp_json = json.loads(resp.text)
        if resp.status_code != 200:
            print("GET Layout results failed:\n%s" % resp_json)
            quit()
        status = resp_json["status"]
        if status == "succeeded":
            print("Layout Analysis succeeded:\n%s" % resp_json)
            quit()
        if status == "failed":
            print("Layout Analysis failed:\n%s" % resp_json)
            quit()
        # Analysis still running. Wait and retry.
        time.sleep(wait_sec)
        n_try += 1     
    except Exception as e:
        msg = "GET analyze results failed:\n%s" % str(e)
        print(msg)
        quit()
```

1. Guarde el script.
1. De nuevo, use el comando `python` para ejecutar el ejemplo. Por ejemplo, `python form-recognizer-layout.py`.

### <a name="examine-the-response"></a>Examen de la respuesta

El script imprimirá las respuestas en la consola hasta que se complete la operación de **análisis de diseño**. Luego, imprimirá los datos extraídos en formato JSON. El nodo `"readResults"` contiene cada línea de texto con su correspondiente posición de cuadro de límite en la página. En el campo `"pageResults"` se muestra cada fragmento de texto dentro de las tablas, cada uno con sus coordenadas de fila y columna.

Consulte la siguiente imagen de una factura y su correspondiente salida JSON. La salida se ha abreviado por motivos de simplicidad.

> [!div class="mx-imgBorder"]
> ![Factura de Contoso con una tabla](../media/contoso-invoice.png)

```json
{ 
  "status":"succeeded",
  "createdDateTime":"2019-11-12T19:55:36Z",
  "lastUpdatedDateTime":"2019-11-12T19:55:43Z",
  "analyzeResult":{ 
    "version":"2.0.0",
    "readResults":[ 
      { 
        "page":1,
        "language":"en",
        "angle":0,
        "width":8.5,
        "height":11,
        "unit":"inch",
        "lines":[ 
          { 
            "language":"en",
            "boundingBox":[ 
              0.5384,
              1.1583,
              1.4466,
              1.1583,
              1.4466,
              1.3534,
              0.5384,
              1.3534
            ],
            "text":"Contoso",
            "words":[ 
              { 
                "boundingBox":[ 
                  0.5384,
                  1.1583,
                  1.4466,
                  1.1583,
                  1.4466,
                  1.3534,
                  0.5384,
                  1.3534
                ],
                "text":"Contoso",
                "confidence":1
              }
            ]
          },
          { 
            "language":"en",
            "boundingBox":[ 
              0.7994,
              1.5143,
              1.3836,
              1.5143,
              1.3836,
              1.6154,
              0.7994,
              1.6154
            ],
            "text":"Address:",
            "words":[ 
              { 
                "boundingBox":[ 
                  0.7994,
                  1.5143,
                  1.3836,
                  1.5143,
                  1.3836,
                  1.6154,
                  0.7994,
                  1.6154
                ],
                "text":"Address:",
                "confidence":1
              }
            ]
          },
          ...
          { 
            "language":"en",
            "boundingBox":[ 
              6.2285,
              3.4114,
              6.3919,
              3.4114,
              6.3919,
              3.5119,
              6.2285,
              3.5119
            ],
            "text":"PT",
            "words":[ 
              { 
                "boundingBox":[ 
                  6.2285,
                  3.4114,
                  6.3919,
                  3.4114,
                  6.3919,
                  3.5119,
                  6.2285,
                  3.5119
                ],
                "text":"PT",
                "confidence":1
              }
            ]
          }
        ]
      }
    ],
    "pageResults":[ 
      { 
        "page":1,
        "tables":[ 
          { 
            "rows":2,
            "columns":6,
            "cells":[ 
              { 
                "rowIndex":0,
                "columnIndex":0,
                "text":"Invoice Number",
                "boundingBox":[ 
                  0.5075,
                  2.8088,
                  1.9061,
                  2.8088,
                  1.9061,
                  3.3219,
                  0.5075,
                  3.3219
                ],
                "elements":[ 
                  "#/readResults/0/lines/8/words/0",
                  "#/readResults/0/lines/8/words/1"
                ]
              },
              ...
            ]
          }
        ]
      }
    ]
  }
}
```

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, ha usado la API REST de Form Recognizer con Python para extraer el diseño de texto de una factura. A continuación, consulte la documentación de referencia para explorar la API de Form Recognizer con mayor profundidad.

> [!div class="nextstepaction"]
> [Documentación de referencia de API REST](https://westus2.dev.cognitive.microsoft.com/docs/services/form-recognizer-api-v2/operations/AnalyzeLayoutAsync)
