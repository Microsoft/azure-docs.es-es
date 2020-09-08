---
title: 'Inicio rápido: Entrenamiento con etiquetas mediante la API REST y Python: Form Recognizer'
titleSuffix: Azure Cognitive Services
description: Aprenda a usar la característica de datos etiquetados Form Recognizer con la API REST y Python para entrenar un modelo personalizado.
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: forms-recognizer
ms.topic: quickstart
ms.date: 05/27/2020
ms.author: pafarley
ms.custom: devx-track-python
ms.openlocfilehash: e96940960b6ee131068b77bca4818499377ea3dd
ms.sourcegitcommit: 5b6acff3d1d0603904929cc529ecbcfcde90d88b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/21/2020
ms.locfileid: "88723508"
---
# <a name="train-a-form-recognizer-model-with-labels-using-rest-api-and-python"></a>Entrenamiento de un modelo de Form Recognizer con etiquetas mediante la API REST y Python

En este inicio rápido, usará la API REST de Form Recognizer con Python para entrenar un modelo personalizado con datos etiquetados manualmente. Consulte la sección [Entrenamiento con etiquetas](../overview.md#train-with-labels) de la introducción para más información acerca de esta característica.

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/cognitive-services/) antes de empezar.

## <a name="prerequisites"></a>Requisitos previos

Para completar este inicio rápido, debe cumplir los siguientes requisitos:
- [Python](https://www.python.org/downloads/) instalado (si desea ejecutar el ejemplo localmente).
- Un conjunto de al menos seis formularios del mismo tipo. Usará estos datos para entrenar el modelo y probar un formulario. En este inicio rápido puede usar un [conjunto de datos de ejemplo](https://go.microsoft.com/fwlink/?linkid=2090451). Cargue los archivos de entrenamiento en la raíz de un contenedor de almacenamiento de blobs de una cuenta de Azure Storage de nivel de rendimiento estándar.

> [!NOTE]
> En este inicio rápido se usan documentos remotos con acceso mediante la dirección URL. Para usar archivos locales en su lugar, consulte la [documentación de referencia de la versión v2.0](https://westus2.dev.cognitive.microsoft.com/docs/services/form-recognizer-api-v2/operations/TrainCustomModelAsync) y la [documentación de referencia de la versión v2.1](https://westcentralus.dev.cognitive.microsoft.com/docs/services/form-recognizer-api-v2-1-preview-1/operations/TrainCustomModelAsync).

## <a name="create-a-form-recognizer-resource"></a>Creación de un recurso de Form Recognizer

[!INCLUDE [create resource](../includes/create-resource.md)]

## <a name="set-up-training-data"></a>Configuración de los datos de entrenamiento

Después, deberá configurar los datos de entrada necesarios. La característica de datos etiquetados tiene requisitos de entrada especiales que van más allá de los que se necesitan para entrenar un modelo personalizado sin etiquetas.

Asegúrese de que todos los documentos de entrenamiento tienen el mismo formato. Si tiene formularios en varios formatos, organícelos en subcarpetas basadas en un formato común. Al entrenar, deberá dirigir la API a una subcarpeta.

Para entrenar un modelo con datos etiquetados, necesitará los siguientes archivos como entradas en la subcarpeta. A continuación, aprenderá a crear estos archivos.

* **Formularios de origen**: los formularios de los que se van a extraer datos. Los tipos admitidos son JPEG, PNG, PDF o TIFF.
* **Archivos de diseño OCR**: archivos JSON que describen los tamaños y las posiciones de todo el texto legible de cada formulario de origen. Usará la API de diseño de Form Recognizer para generar estos datos. 
* **Archivos de etiqueta**: archivos JSON que describen las etiquetas de datos que un usuario ha escrito manualmente.

Todos estos archivos deben ocupar la misma subcarpeta y tener el siguiente formato:

* input_file1.pdf 
* input_file1.pdf.ocr.json
* input_file1.pdf.labels.json 
* input_file2.pdf 
* input_file2.pdf.ocr.json
* input_file2.pdf.labels.json
* …

> [!TIP]
> Cuando etiqueta formularios mediante la [herramienta de etiquetado de ejemplo](./label-tool.md) Form Recognizer, dicha herramienta crea estas etiquetas y los archivos de diseño OCR automáticamente.

### <a name="create-the-ocr-output-files"></a>Creación de los archivos de salida de OCR

Necesita archivos de resultados de OCR para que el servicio tenga en cuenta los archivos de entrada correspondientes para el entrenamiento etiquetado. Para obtener los resultados de OCR de un formulario de origen determinado, siga estos pasos:

1. Llame a la API **[Analyze Layout](https://westus2.dev.cognitive.microsoft.com/docs/services/form-recognizer-api-v2/operations/AnalyzeLayoutAsync)** en el contenedor de diseño de lectura con el archivo de entrada como parte del cuerpo de la solicitud. Guarde el identificador que se encontró en el encabezado **Operation-Location** de la respuesta.
1. Llame a la API **[Get Analyze Layout Result](https://westus2.dev.cognitive.microsoft.com/docs/services/form-recognizer-api-v2/operations/GetAnalyzeLayoutResult)** mediante el identificador de la operación del paso anterior.
1. Obtenga la respuesta y escriba el contenido en un archivo. Para cada formulario de origen, el archivo de OCR correspondiente debe tener el nombre del archivo original anexado a `.ocr.json`. La salida JSON del OCR debe tener el siguiente formato. Para ver un ejemplo completo, consulte el [archivo OCR de ejemplo](https://github.com/Azure-Samples/cognitive-services-REST-api-samples/blob/master/curl/form-recognizer/Invoice_1.pdf.ocr.json). 

    # <a name="v20"></a>[v2.0](#tab/v2-0)
    ```json
    {
    "status": "succeeded",
    "createdDateTime": "2019-11-12T21:18:12Z",
    "lastUpdatedDateTime": "2019-11-12T21:18:17Z",
    "analyzeResult": {
        "version": "2.0.0",
        "readResults": [
            {
                "page": 1,
                "language": "en",
                "angle": 0,
                "width": 8.5,
                "height": 11,
                "unit": "inch",
                "lines": [
                    {
                        "language": "en",
                        "boundingBox": [
                            0.5384,
                            1.1583,
                            1.4466,
                            1.1583,
                            1.4466,
                            1.3534,
                            0.5384,
                            1.3534
                        ],
                        "text": "Contoso",
                        "words": [
                            {
                                "boundingBox": [
                                    0.5384,
                                    1.1583,
                                    1.4466,
                                    1.1583,
                                    1.4466,
                                    1.3534,
                                    0.5384,
                                    1.3534
                                ],
                                "text": "Contoso",
                                "confidence": 1
                            }
                        ]
                    },
                    ...
    ```    
    # <a name="v21-preview"></a>[Versión preliminar v2.1](#tab/v2-1)
    ```json
    {
    "status": "succeeded",
    "createdDateTime": "2019-11-12T21:18:12Z",
    "lastUpdatedDateTime": "2019-11-12T21:18:17Z",
    "analyzeResult": {
        "version": "2.1.0",
        "readResults": [
            {
                "page": 1,
                "language": "en",
                "angle": 0,
                "width": 8.5,
                "height": 11,
                "unit": "inch",
                "lines": [
                    {
                        "language": "en",
                        "boundingBox": [
                            0.5384,
                            1.1583,
                            1.4466,
                            1.1583,
                            1.4466,
                            1.3534,
                            0.5384,
                            1.3534
                        ],
                        "text": "Contoso",
                        "words": [
                            {
                                "boundingBox": [
                                    0.5384,
                                    1.1583,
                                    1.4466,
                                    1.1583,
                                    1.4466,
                                    1.3534,
                                    0.5384,
                                    1.3534
                                ],
                                "text": "Contoso",
                                "confidence": 1
                            }
                        ]
                    },
                    ...
    ```    


    ---



### <a name="create-the-label-files"></a>Creación de archivos de etiqueta

Los archivos de etiqueta contienen asociaciones de clave-valor que un usuario ha especificado manualmente. Son necesarios para el entrenamiento con datos etiquetados, pero no todos los archivos de origen deben tener un archivo de etiqueta correspondiente. Los archivos de origen sin etiquetas se tratarán como documentos de entrenamiento ordinarios. Se recomiendan cinco o más archivos etiquetados para un entrenamiento confiable. Puede usar una herramienta de interfaz de usuario como la [herramienta de etiquetado de ejemplo](./label-tool.md) para generar estos archivos.

Al crear un archivo de etiquetas, puede especificar de forma opcional las posiciones de regiones exactas de los valores del documento. Esto proporcionará al entrenamiento aún más precisión. Las regiones tienen un formato de conjunto de ocho valores que se corresponden con las cuatro coordenadas X, Y: superior izquierda, superior derecha, inferior derecha e inferior izquierda. Los valores de las coordenadas están entre cero y uno, escalados a las dimensiones de la página.

Para cada formulario de origen, el archivo de etiquetas correspondiente debe tener el nombre del archivo original anexado a `.labels.json`. El archivo de etiquetas debe tener el siguiente formato. Para ver un ejemplo completo, consulte el [archivo de etiquetas de ejemplo](https://github.com/Azure-Samples/cognitive-services-REST-api-samples/blob/master/curl/form-recognizer/Invoice_1.pdf.labels.json).

```json
{
    "document": "Invoice_1.pdf",
    "labels": [
        {
            "label": "Provider",
            "key": null,
            "value": [
                {
                    "page": 1,
                    "text": "Contoso",
                    "boundingBoxes": [
                        [
                            0.06334117647058823,
                            0.1053,
                            0.17018823529411767,
                            0.1053,
                            0.17018823529411767,
                            0.12303636363636362,
                            0.06334117647058823,
                            0.12303636363636362
                        ]
                    ]
                }
            ]
        },
        {
            "label": "For",
            "key": null,
            "value": [
                {
                    "page": 1,
                    "text": "Microsoft",
                    "boundingBoxes": [
                        [
                            0.6122941176470589,
                            0.1374,
                            0.6841764705882353,
                            0.1374,
                            0.6841764705882353,
                            0.14682727272727272,
                            0.6122941176470589,
                            0.14682727272727272
                        ]
                    ]
                },
                {
                    "page": 1,
                    "text": "1020",
                    "boundingBoxes": [
                        [
                            0.6121882352941176,
                            0.156,
                            0.6462941176470588,
                            0.156,
                            0.6462941176470588,
                            0.1653181818181818,
                            0.6121882352941176,
                            0.1653181818181818
                        ]
                    ]
                },
                ...
```

> [!IMPORTANT]
> Solo se puede aplicar una etiqueta a cada elemento de texto y cada etiqueta solo se puede aplicar una vez por página. Actualmente no se puede aplicar una etiqueta a varias páginas.


## <a name="train-a-model-using-labeled-data"></a>Entrenamiento de un modelo con datos etiquetados

Para entrenar un modelo con datos etiquetados, llame a la API **[Train Custom Model](https://westus2.dev.cognitive.microsoft.com/docs/services/form-recognizer-api-v2/operations/TrainCustomModelAsync)** ejecutando el siguiente código de Python. Antes de ejecutar el código, realice estos cambios:

1. Reemplace `<Endpoint>` por la dirección URL del punto de conexión del recurso de Form Recognizer.
1. Reemplace `<SAS URL>` por la dirección URL de la firma de acceso compartido (SAS) del contenedor de almacenamiento de blobs de Azure. Para recuperar la dirección URL de SAS, abra el Explorador de Microsoft Azure Storage, haga clic con el botón derecho en el contenedor y seleccione **Get shared access signature** (Obtener firma de acceso compartido). Asegúrese de que los permisos de **lectura** y **enumeración** están marcados y haga clic en **Create** (Crear). A continuación, copie el valor en la sección **URL**. Debe tener el formato `https://<storage account>.blob.core.windows.net/<container name>?<SAS value>`.
1. Reemplace `<Blob folder name>` por el nombre de la carpeta del contenedor de blobs donde se encuentran los datos de entrada. O bien, si los datos están en la raíz, déjelo en blanco y quite el campo `"prefix"` del cuerpo de la solicitud HTTP.

# <a name="v20"></a>[v2.0](#tab/v2-0)
```python
########### Python Form Recognizer Labeled Async Train #############
import json
import time
from requests import get, post

# Endpoint URL
endpoint = r"<Endpoint>"
post_url = endpoint + r"/formrecognizer/v2.0/custom/models"
source = r"<SAS URL>"
prefix = "<Blob folder name>"
includeSubFolders = False
useLabelFile = True

headers = {
    # Request headers
    'Content-Type': 'application/json',
    'Ocp-Apim-Subscription-Key': '<subsription key>',
}

body =     {
    "source": source,
    "sourceFilter": {
        "prefix": prefix,
        "includeSubFolders": includeSubFolders
    },
    "useLabelFile": useLabelFile
}

try:
    resp = post(url = post_url, json = body, headers = headers)
    if resp.status_code != 201:
        print("POST model failed (%s):\n%s" % (resp.status_code, json.dumps(resp.json())))
        quit()
    print("POST model succeeded:\n%s" % resp.headers)
    get_url = resp.headers["location"]
except Exception as e:
    print("POST model failed:\n%s" % str(e))
    quit() 
```    
# <a name="v21-preview"></a>[Versión preliminar v2.1](#tab/v2-1)    
```python
########### Python Form Recognizer Labeled Async Train #############
import json
import time
from requests import get, post

# Endpoint URL
endpoint = r"<Endpoint>"
post_url = endpoint + r"/formrecognizer/v2.1-preview.1/custom/models"
source = r"<SAS URL>"
prefix = "<Blob folder name>"
includeSubFolders = False
useLabelFile = True

headers = {
    # Request headers
    'Content-Type': 'application/json',
    'Ocp-Apim-Subscription-Key': '<subsription key>',
}

body =     {
    "source": source,
    "sourceFilter": {
        "prefix": prefix,
        "includeSubFolders": includeSubFolders
    },
    "useLabelFile": useLabelFile
}

try:
    resp = post(url = post_url, json = body, headers = headers)
    if resp.status_code != 201:
        print("POST model failed (%s):\n%s" % (resp.status_code, json.dumps(resp.json())))
        quit()
    print("POST model succeeded:\n%s" % resp.headers)
    get_url = resp.headers["location"]
except Exception as e:
    print("POST model failed:\n%s" % str(e))
    quit() 
```   

---



## <a name="get-training-results"></a>Obtención de los resultados del entrenamiento

Una vez que haya iniciado la operación de entrenamiento, use el identificador devuelto para obtener el estado de esta. Agregue el código siguiente al final del script de Python. Este utilizará el valor del identificador de la llamada de entrenamiento en una nueva llamada API. La operación de entrenamiento es asincrónica, así que este script llama a la API a intervalos regulares hasta que el estado del entrenamiento es "Completado". Se recomienda un intervalo de uno o varios segundos.

```python 
n_tries = 15
n_try = 0
wait_sec = 5
max_wait_sec = 60
while n_try < n_tries:
    try:
        resp = get(url = get_url, headers = headers)
        resp_json = resp.json()
        if resp.status_code != 200:
            print("GET model failed (%s):\n%s" % (resp.status_code, json.dumps(resp_json)))
            quit()
        model_status = resp_json["modelInfo"]["status"]
        if model_status == "ready":
            print("Training succeeded:\n%s" % json.dumps(resp_json))
            quit()
        if model_status == "invalid":
            print("Training failed. Model is invalid:\n%s" % json.dumps(resp_json))
            quit()
        # Training still running. Wait and retry.
        time.sleep(wait_sec)
        n_try += 1
        wait_sec = min(2*wait_sec, max_wait_sec)     
    except Exception as e:
        msg = "GET model failed:\n%s" % str(e)
        print(msg)
        quit()
print("Train operation did not complete within the allocated time.")
```

Cuando se complete el proceso de entrenamiento, recibirá una respuesta `201 (Success)` con contenido JSON similar al siguiente. La respuesta se ha abreviado por motivos de simplicidad.

```json
{ 
  "modelInfo":{ 
    "status":"ready",
    "createdDateTime":"2019-10-08T10:20:31.957784",
    "lastUpdatedDateTime":"2019-10-08T14:20:41+00:00",
    "modelId":"1cfb372bab404ba3aa59481ab2c63da5"
  },
  "trainResult":{ 
    "trainingDocuments":[ 
      { 
        "documentName":"invoices\\Invoice_1.pdf",
        "pages":1,
        "errors":[ 

        ],
        "status":"succeeded"
      },
      { 
        "documentName":"invoices\\Invoice_2.pdf",
        "pages":1,
        "errors":[ 

        ],
        "status":"succeeded"
      },
      { 
        "documentName":"invoices\\Invoice_3.pdf",
        "pages":1,
        "errors":[ 

        ],
        "status":"succeeded"
      },
      { 
        "documentName":"invoices\\Invoice_4.pdf",
        "pages":1,
        "errors":[ 

        ],
        "status":"succeeded"
      },
      { 
        "documentName":"invoices\\Invoice_5.pdf",
        "pages":1,
        "errors":[ 

        ],
        "status":"succeeded"
      }
    ],
    "errors":[ 

    ]
  },
  "keys":{ 
    "0":[ 
      "Address:",
      "Invoice For:",
      "Microsoft",
      "Page"
    ]
  }
}
```

Copie el valor `"modelId"` para usarlo en los pasos siguientes.

[!INCLUDE [analyze forms](../includes/python-custom-analyze.md)]

Cuando se complete el proceso, recibirá una respuesta `202 (Success)` con contenido JSON con el siguiente formato. La respuesta se ha abreviado por motivos de simplicidad. Las asociaciones de clave-valor principales se encuentran en el nodo `"documentResults"`. El nodo `"selectionMarks"` (en la versión preliminar v2.1) muestra todas las casillas de selección (casilla, botón de opción) y si su estado es "seleccionado" o "no seleccionado". Los resultados de la API de diseño (el contenido y las posiciones de todo el texto del documento) se encuentran en el nodo `"readResults"`.

# <a name="v20"></a>[v2.0](#tab/v2-0)
```json
{
  "status": "succeeded",
  "createdDateTime": "2020-08-21T02:16:28Z",
  "lastUpdatedDateTime": "2020-08-21T02:16:35Z",
  "analyzeResult": {
    "version": "2.0.0",
    "readResults": [
      {
        "page": 1,
        "language": "en",
        "angle": 0,
        "width": 8.5,
        "height": 11,
        "unit": "inch",
        "lines": [
          {
            "boundingBox": [
              0.5826,
              0.4411,
              2.3387,
              0.4411,
              2.3387,
              0.7969,
              0.5826,
              0.7969
            ],
            "text": "Contoso, Ltd.",
            "words": [
              {
                "boundingBox": [
                  0.5826,
                  0.4411,
                  1.744,
                  0.4411,
                  1.744,
                  0.7969,
                  0.5826,
                  0.7969
                ],
                "text": "Contoso,",
                "confidence": 1
              },
              {
                "boundingBox": [
                  1.8448,
                  0.4446,
                  2.3387,
                  0.4446,
                  2.3387,
                  0.7631,
                  1.8448,
                  0.7631
                ],
                "text": "Ltd.",
                "confidence": 1
              }
            ]
          },
          ...
            ]
          }
        ] 
      }
    ],
    "pageResults": [
      {
        "page": 1,
        "tables": [
          {
            "rows": 5,
            "columns": 5,
            "cells": [
              {
                "rowIndex": 0,
                "columnIndex": 0,
                "text": "Training Date",
                "boundingBox": [
                  0.5133,
                  4.2167,
                  1.7567,
                  4.2167,
                  1.7567,
                  4.4492,
                  0.5133,
                  4.4492
                ],
                "elements": [
                  "#/readResults/0/lines/14/words/0",
                  "#/readResults/0/lines/14/words/1"
                ]
              },
              ...
            ]
          },
        ]
      }
    ],
    "documentResults": [
      {
        "docType": "custom:form",
        "pageRange": [
          1,
          1
        ],
        "fields": {
          "Receipt No": {
            "type": "string",
            "valueString": "9876",
            "text": "9876",
            "page": 1,
            "boundingBox": [
              7.615,
              1.245,
              7.915,
              1.245,
              7.915,
              1.35,
              7.615,
              1.35
            ],
            "confidence": 1,
            "elements": [
              "#/readResults/0/lines/3/words/3"
            ]
          },
          ...
        }
      }
    ],
    "errors": []
  }
}
```
# <a name="v21-preview"></a>[Versión preliminar v2.1](#tab/v2-1) 
```json   
{
  "status": "succeeded",
  "createdDateTime": "2020-08-21T02:29:42Z",
  "lastUpdatedDateTime": "2020-08-21T02:29:50Z",
  "analyzeResult": {
    "version": "2.1.0",
    "readResults": [
      {
        "page": 1,
        "angle": 0,
        "width": 8.5,
        "height": 11,
        "unit": "inch",
        "lines": [
          {
            "boundingBox": [
              0.5826,
              0.4411,
              2.3387,
              0.4411,
              2.3387,
              0.7969,
              0.5826,
              0.7969
            ],
            "text": "Contoso, Ltd.",
            "words": [
              {
                "boundingBox": [
                  0.5826,
                  0.4411,
                  1.744,
                  0.4411,
                  1.744,
                  0.7969,
                  0.5826,
                  0.7969
                ],
                "text": "Contoso,",
                "confidence": 1
              },
              {
                "boundingBox": [
                  1.8448,
                  0.4446,
                  2.3387,
                  0.4446,
                  2.3387,
                  0.7631,
                  1.8448,
                  0.7631
                ],
                "text": "Ltd.",
                "confidence": 1
              }
            ]
          },
          ...
        ], 
        "selectionMarks": [
          {
            "boundingBox": [
              3.9737,
              3.7475,
              4.1693,
              3.7475,
              4.1693,
              3.9428,
              3.9737,
              3.9428
            ],
            ...
        ] 
      }
    ],
    "pageResults": [
      {
        "page": 1,
        "tables": [
          {
            "rows": 5,
            "columns": 5,
            "cells": [
              {
                "rowIndex": 0,
                "columnIndex": 0,
                "text": "Training Date",
                "boundingBox": [
                  0.5133,
                  4.2167,
                  1.7567,
                  4.2167,
                  1.7567,
                  4.4492,
                  0.5133,
                  4.4492
                ],
                "elements": [
                  "#/readResults/0/lines/12/words/0",
                  "#/readResults/0/lines/12/words/1"
                ]
              },
              ...
            ]
          }
        ] 
      }
    ], 
    "documentResults": [
      {
        "docType": "custom:e1073364-4f3d-4797-8cc4-4bdbcd0dab6b",
        "modelId": "e1073364-4f3d-4797-8cc4-4bdbcd0dab6b",
        "pageRange": [
          1,
          1
        ],
        "fields": {
          "ID #": {
            "type": "string",
            "valueString": "5554443",
            "text": "5554443",
            "page": 1,
            "boundingBox": [
              2.315,
              2.43,
              2.74,
              2.43,
              2.74,
              2.515,
              2.315,
              2.515
            ],
            "confidence": 1,
            "elements": [
              "#/readResults/0/lines/8/words/1"
            ]
          },
          ...
        },
        "docTypeConfidence": 1
      }
    ],
    "errors": []
  }
}
```

---


## <a name="improve-results"></a>Mejora de los resultados

Examine los valores de `"confidence"` de cada resultado de clave-valor en el nodo `"documentResults"`. También debe examinar las puntuaciones de confianza del nodo `"readResults"`, que se corresponden a la operación de diseño. La confianza de los resultados del diseño no afecta a la confianza de los resultados de la extracción de los pares clave-valor, por lo que debe comprobar ambos.
* Si las puntuaciones de confianza de la operación de diseño son bajas, intente mejorar la calidad de los documentos de entrada (consulte [Requisitos de entrada](../overview.md#input-requirements)).
* Si las puntuaciones de confianza de la operación de extracción de pares clave-valor son bajas, asegúrese de que los documentos que se están analizando sean del mismo tipo que los documentos usados en el conjunto de entrenamiento. Si los documentos del conjunto de entrenamiento tienen variantes de apariencia, considere la posibilidad de dividirlos en carpetas diferentes y entrenar un modelo para cada variante.

### <a name="avoid-cluttered-labels"></a>Evitar etiquetas desordenadas

A veces, cuando se aplican etiquetas diferentes en la misma línea de texto, el servicio puede combinar esas etiquetas en un campo. Por ejemplo, en una dirección, podría etiquetar la ciudad, el estado y el código postal como campos diferentes, pero durante la predicción esos campos no se reconocerían por separado.

Somos conscientes de que esta posibilidad es fundamental para nuestros clientes y estamos trabajando para mejorarla en el futuro. Actualmente, recomendamos que los usuarios etiqueten varios campos desordenados como un solo campo y, a continuación, separe los términos en un procesamiento posterior de los resultados de la extracción.

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, ha aprendido a usar la API REST de Form Recognizer con Python para entrenar un modelo con datos etiquetados manualmente. A continuación, consulte la documentación de referencia para explorar la API de Form Recognizer con mayor profundidad.

> [!div class="nextstepaction"]
> [Documentación de referencia de API REST](https://westus2.dev.cognitive.microsoft.com/docs/services/form-recognizer-api-v2/operations/AnalyzeWithCustomForm)
