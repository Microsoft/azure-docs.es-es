---
title: 'Tarjetas de presentación: Form Recognizer'
titleSuffix: Azure Cognitive Services
description: 'Conozca los conceptos relacionados con el análisis de tarjetas de presentación con la API Form Recognizer: uso y límites.'
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: forms-recognizer
ms.topic: conceptual
ms.date: 08/17/2019
ms.author: pafarley
ms.openlocfilehash: 1163531fb5a6aa7158bd81ff9095ed1ee29e73c1
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89004908"
---
# <a name="business-card-concepts"></a>Conceptos de tarjeta de presentación

Azure Form Recognizer puede analizar y extraer información de contacto de tarjetas de presentación mediante alguno de los modelos precompilados. La API de tarjetas de presentación combina funcionalidades eficaces de reconocimiento óptico de caracteres (OCR) con nuestro modelo de comprensión de tarjetas de presentación para extraer información clave de las tarjetas de presentación en inglés. Extrae información de contacto personal, el nombre de la empresa, el puesto y mucho más. La API de tarjeta de presentación precompilada está disponible públicamente en la versión preliminar de Form Recognizer v2.1. 

## <a name="what-does-the-business-card-api-do"></a>¿Qué hace la API de tarjeta de presentación?

La API de tarjeta de presentación extrae campos clave de las tarjetas de presentación y los devuelve en una respuesta JSON organizada.

![Imagen de Contoso detallada de FOTT y salida JSON](./media/business-card-english.jpg)

### <a name="fields-extracted"></a>Campos extraídos:

* Nombres de contacto 
  * Nombres
  * Apellidos
* Nombres de compañía 
* Departments 
* Puestos de trabajo 
* Correos electrónicos 
* Websites 
* Direcciones 
* Números de teléfono 
  * Teléfonos móviles 
  * Faxes 
  * Teléfonos de trabajo 
  * Otros teléfonos 

La API de tarjeta de presentación también puede devolver todo el texto reconocido de la tarjeta de presentación. Esta salida de OCR se incluye en la respuesta JSON.  

### <a name="input-requirements"></a>Requisitos de entrada 

[!INCLUDE [input reqs](./includes/input-requirements-receipts.md)]

## <a name="the-analyze-business-card-operation"></a>Operación Analyze Business Card

[Analyze Business Card](https://westcentralus.dev.cognitive.microsoft.com/docs/services/form-recognizer-api-v2-1-preview-1/operations/AnalyzeBusinessCardAsync) toma una imagen o PDF de una tarjeta de presentación como entrada y extrae los valores de interés. La llamada devuelve un campo de encabezado de respuesta denominado `Operation-Location`. El valor `Operation-Location` es una dirección URL que contiene el id. de resultado que se va a usar en el paso siguiente.

|Encabezado de respuesta| Dirección URL del resultado |
|:-----|:----|
|Operation-Location | `https://cognitiveservice/formrecognizer/v2.1-preview.1/prebuilt/businessCard/analyzeResults/49a36324-fc4b-4387-aa06-090cfbf0064f` |

## <a name="the-get-analyze-business-card-result-operation"></a>Operación Get Analyze Business Card Result

El segundo paso consiste en llamar a la operación [Get Analyze Business Card Result](https://westcentralus.dev.cognitive.microsoft.com/docs/services/form-recognizer-api-v2-1-preview-1/operations/GetAnalyzeBusinessCardResult). Esta operación toma como entrada el id. de resultado que la operación Analyze Business Card ha creado. Devuelve una respuesta JSON que contiene un campo de **estado** con los siguientes valores posibles. Llamará a esta operación de forma iterativa hasta que se devuelva con el valor **correcto**. Use un intervalo de 3 a 5 segundos para evitar superar la tasa de solicitudes por segundo (RPS).

|Campo| Tipo | Valores posibles |
|:-----|:----:|:----|
|status | string | notStarted: la operación de análisis no se ha iniciado.<br /><br />running: la operación de análisis está en curso.<br /><br />failed: error en la operación de análisis.<br /><br />succeeded: la operación de análisis se realizó correctamente.|

Cuando el campo **status** tenga el valor **succeeded**, la respuesta JSON incluirá los resultados del reconocimiento de texto opcional y la tarjeta de presentación, si es necesario. El resultado de la descripción de la tarjeta de presentación se organiza como un diccionario de valores de campo con nombre, en el que cada valor contiene el texto extraído, el valor normalizado, el cuadro de límite, el nivel de confianza y los elementos de la palabra correspondiente. El resultado del reconocimiento de texto se organiza como una jerarquía de líneas y palabras, con texto, cuadro de límite e información de confianza.

![salida de tarjeta de presentación de ejemplo](./media/business-card-results.png)

### <a name="sample-json-output"></a>Salida de JSON de ejemplo

Consulte el siguiente ejemplo de una respuesta JSON correcta: El nodo "readResults" contiene todo el texto reconocido. El texto se organiza por página, después, por líneas y, finalmente, por palabras individuales. El nodo "documentResults" contiene los valores específicos de la tarjeta de presentación que el modelo ha detectado. Aquí encontrará información de contacto útil como el nombre, el apellido, el nombre de la compañía y más.

```json
{
    "status": "succeeded",
    "createdDateTime": "2020-08-20T17:41:19Z",
    "lastUpdatedDateTime": "2020-08-20T17:41:24Z",
    "analyzeResult": {
        "version": "2.1.0",
        "readResults": [
            {
                "page": 1,
                "angle": -17.0956,
                "width": 4032,
                "height": 3024,
                "unit": "pixel",
                "lines": 
                          {
                        "text": "Dr. Avery Smith",
                        "boundingBox": [
                            419.3,
                            1154.6,
                            1589.6,
                            877.9,
                            1618.9,
                            1001.7,
                            448.6,
                            1278.4
                        ],
                        "words": [
                            {
                                "text": "Dr.",
                                "boundingBox": [
                                    419,
                            ]
    
            }
        ],
        "documentResults": [
            {
                "docType": "prebuilt:businesscard",
                "pageRange": [
                    1,
                    1
                ],
                "fields": {
                    "ContactNames": {
                        "type": "array",
                        "valueArray": [
                            {
                                "type": "object",
                                "valueObject": {
                                    "FirstName": {
                                        "type": "string",
                                        "valueString": "Avery",
                                        "text": "Avery",
                                        "boundingBox": [
                                            703,
                                            1096,
                                            1134,
                                            989,
                                            1165,
                                            1109,
                                            733,
                                            1206
                                        ],
                                        "page": 1
                                    },
                                    "LastName": {
                                        "type": "string",
                                        "valueString": "Smith",
                                        "text": "Smith",
                                        "boundingBox": [
                                            1186,
                                            976,
                                            1585,
                                            879,
                                            1618,
                                            998,
                                            1218,
                                            1096
                                        ],
                                        "page": 1
                                    }
                                },
                                "text": "Dr. Avery Smith",
                                "boundingBox": [
                                    419.3,
                                    1154.6,
                                    1589.6,
                                    877.9,
                                    1618.9,
                                    1001.7,
                                    448.6,
                                    1278.4
                                ],
                                "confidence": 0.97
                            }
                        ]
                    },
                    "JobTitles": {
                        "type": "array",
                        "valueArray": [
                            {
                                "type": "string",
                                "valueString": "Senior Researcher",
                                "text": "Senior Researcher",
                                "boundingBox": [
                                    451.8,
                                    1301.9,
                                    1313.5,
                                    1099.9,
                                    1333.8,
                                    1186.7,
                                    472.2,
                                    1388.7
                                ],
                                "page": 1,
                                "confidence": 0.99
                            }
                        ]
                    },
                    "Departments": {
                        "type": "array",
                        "valueArray": [
                            {
                                "type": "string",
                                "valueString": "Cloud & Al Department",
                                "text": "Cloud & Al Department",
                                "boundingBox": [
                                    480.1,
                                    1403.3,
                                    1590.5,
                                    1129.6,
                                    1612.6,
                                    1219.6,
                                    502.3,
                                    1493.3
                                ],
                                "page": 1,
                                "confidence": 0.99
                            }
                        ]
                    },
                    "Emails": {
                        "type": "array",
                        "valueArray": [
                            {
                                "type": "string",
                                "valueString": "avery.smith@contoso.com",
                                "text": "avery.smith@contoso.com",
                                "boundingBox": [
                                    2107,
                                    934,
                                    2917,
                                    696,
                                    2935,
                                    764,
                                    2126,
                                    995
                                ],
                                "page": 1,
                                "confidence": 0.99
                            }
                        ]
                    },
                    "Websites": {
                        "type": "array",
                        "valueArray": [
                            {
                                "type": "string",
                                "valueString": "https://www.contoso.com/",
                                "text": "https://www.contoso.com/",
                                "boundingBox": [
                                    2121,
                                    1002,
                                    2992,
                                    755,
                                    3014,
                                    826,
                                    2143,
                                    1077
                                ],
                                "page": 1,
                                "confidence": 0.995
                            }
                        ]
                    },
                    "MobilePhones": {
                        "type": "array",
                        "valueArray": [
                            {
                                "type": "phoneNumber",
                                "text": "+44 (0) 7911 123456",
                                "boundingBox": [
                                    2434.9,
                                    1033.3,
                                    3072,
                                    836,
                                    3096.2,
                                    914.3,
                                    2459.1,
                                    1111.6
                                ],
                                "page": 1,
                                "confidence": 0.99
                            }
                        ]
                    },
                    "OtherPhones": {
                        "type": "array",
                        "valueArray": [
                            {
                                "type": "phoneNumber",
                                "text": "+44 (0) 20 9876 5432",
                                "boundingBox": [
                                    2473.2,
                                    1115.4,
                                    3139.2,
                                    907.7,
                                    3163.2,
                                    984.7,
                                    2497.2,
                                    1192.4
                                ],
                                "page": 1,
                                "confidence": 0.99
                            }
                        ]
                    },
                    "Faxes": {
                        "type": "array",
                        "valueArray": [
                            {
                                "type": "phoneNumber",
                                "text": "+44 (0) 20 6789 2345",
                                "boundingBox": [
                                    2525,
                                    1185.4,
                                    3192.4,
                                    977.9,
                                    3217.9,
                                    1060,
                                    2550.5,
                                    1267.5
                                ],
                                "page": 1,
                                "confidence": 0.99
                            }
                        ]
                    },
                    "Addresses": {
                        "type": "array",
                        "valueArray": [
                            {
                                "type": "string",
                                "valueString": "2 Kingdom Street Paddington, London, W2 6BD",
                                "text": "2 Kingdom Street Paddington, London, W2 6BD",
                                "boundingBox": [
                                    1230,
                                    2138,
                                    2535.2,
                                    1678.6,
                                    2614.2,
                                    1903.1,
                                    1309,
                                    2362.5
                                ],
                                "page": 1,
                                "confidence": 0.977
                            }
                        ]
                    },
                    "CompanyNames": {
                        "type": "array",
                        "valueArray": [
                            {
                                "type": "string",
                                "valueString": "Contoso",
                                "text": "Contoso",
                                "boundingBox": [
                                    1152,
                                    1916,
                                    2293,
                                    1552,
                                    2358,
                                    1733,
                                    1219,
                                    2105
                                ],
                                "page": 1,
                                "confidence": 0.97
                            }
                        ]
                    }
                }
            }
        ]
    }
}
```

Siga la guía de inicio rápido [Extracción de datos de tarjetas de presentación](./QuickStarts/python-business-cards.md) para implementar la extracción de datos de tarjeta de presentación con Python y la API REST.

## <a name="customer-scenarios"></a>Escenarios de cliente  

Los datos extraídos con la API de tarjeta de presentación se pueden usar para realizar varias tareas. Si extrae esta información de contacto, automáticamente ahorrará tiempo en los roles orientados al cliente. A continuación se muestran algunos ejemplos de lo que nuestros clientes han realizado con la API de tarjeta de presentación:

* Extraer información de contacto de tarjetas de presentación y crear contactos de teléfono rápidamente. 
* Integrarse con CRM para crear automáticamente el contacto mediante las imágenes de la tarjeta de presentación. 
* Realizar un seguimiento de los clientes potenciales.  
* Extraer información de contacto en masa de imágenes de tarjetas de presentación existentes. 

La API de tarjeta de presentación también impulsa la [característica de procesamiento de tarjetas de presentación de AI Builder](https://docs.microsoft.com/ai-builder/prebuilt-business-card).

## <a name="next-steps"></a>Pasos siguientes

- Siga las guías de inicio rápido de [Python para la API de tarjeta de presentación](./quickstarts/python-business-cards.md) para empezar
- Obtenga más información acerca de la [API REST Form Recognizer](https://westcentralus.dev.cognitive.microsoft.com/docs/services/form-recognizer-api-v2-1-preview-1/operations/AnalyzeBusinessCardAsync)
- Obtenga más información sobre [Form Recognizer](overview.md)

