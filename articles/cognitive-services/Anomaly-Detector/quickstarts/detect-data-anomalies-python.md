---
title: 'Inicio rápido: Detección de anomalías como un lote mediante la API REST Anomaly Detector y Python'
titleSuffix: Azure Cognitive Services
description: Use la API Anomaly Detector para detectar anomalías en la serie de datos como un lote o en la transmisión de datos con este inicio rápido.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: anomaly-detector
ms.topic: quickstart
ms.date: 06/30/2020
ms.author: aahi
ms.custom: devx-track-python
ms.openlocfilehash: daee5a921e25dc74d6e869ca3feb6c66d8c4ce7d
ms.sourcegitcommit: 7fe8df79526a0067be4651ce6fa96fa9d4f21355
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87851038"
---
# <a name="quickstart-detect-anomalies-in-your-time-series-data-using-the-anomaly-detector-rest-api-and-python"></a>Inicio rápido: Detección de anomalías en los datos de serie temporal mediante API REST Anomaly Detector y Python

Use este inicio rápido para comenzar a usar los dos modos de detección de la API Anomaly Detector para detectar anomalías en los datos de serie temporal. Esta aplicación de Python envía dos solicitudes de API que contienen datos de serie temporal con formato JSON y obtiene las respuestas.

| Solicitud a la API                                        | Salida de la aplicación                                                                                                                         |
|----------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------|
| Detección de anomalías como un lote                        | La respuesta JSON que contiene el estado de la anomalía (y otros datos) para cada punto de datos en los datos de serie temporal y las posiciones de las anomalías detectadas. |
| Detección del estado de anomalía del punto de datos más reciente | La respuesta JSON que contiene el estado de la anomalía (y otros datos) para el punto de datos más reciente en los datos de serie temporal.                                                                                                                                         |

 Si bien esta aplicación está escrita en Python, la API es un servicio web RESTful compatible con la mayoría de los lenguajes de programación. El código fuente de este inicio rápido está disponible en [GitHub](https://github.com/Azure-Samples/AnomalyDetector/blob/master/quickstarts/python-detect-anomalies.py).

## <a name="prerequisites"></a>Requisitos previos

- Una suscripción a Azure: [cree una cuenta gratuita](https://azure.microsoft.com/free/)
- Cuando tenga la suscripción de Azure, <a href="https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesAnomalyDetector"  title="Creación de un recurso de Anomaly Detector"  target="_blank">cree un recurso de Anomaly Detector <span class="docon docon-navigate-external x-hidden-focus"></span></a> en Azure Portal para obtener la clave y el punto de conexión. Espere a que se implemente y haga clic en el botón **Ir al recurso**.
    - Necesitará la clave y el punto de conexión del recurso que cree para conectar la aplicación a Anomaly Detector API. En una sección posterior de este mismo inicio rápido pegará la clave y el punto de conexión en el código siguiente.
    Puede usar el plan de tarifa gratis (`F0`) para probar el servicio y actualizarlo más adelante a un plan de pago para producción.
- [Python 2.x o 3.x](https://www.python.org/downloads/)
- La [Biblioteca de solicitudes](https://pypi.org/project/requests/) para Python

- Un archivo JSON que contiene los puntos de datos de la serie temporal. Los datos de ejemplo de este inicio rápido están disponibles en [GitHub](https://github.com/Azure-Samples/anomalydetector/blob/master/example-data/request-data.json).

[!INCLUDE [anomaly-detector-environment-variables](../includes/environment-variables.md)]

## <a name="create-a-new-application"></a>Creación de una aplicación

1. Cree un archivo de Python y agregue las importaciones siguientes.

    [!code-python[import statements](~/samples-anomaly-detector/quickstarts/python-detect-anomalies.py?name=imports)]

2. Cree variables para la clave de suscripción y el punto de conexión. A continuación se muestran los URI puede usar para la detección de anomalías. Estos se agregarán a su punto de conexión de servicio más adelante para crear las direcciones URL de solicitud de API.

    |Método de detección  |URI  |
    |---------|---------|
    |Detección por lotes    | `/anomalydetector/v1.0/timeseries/entire/detect`        |
    |Detección en el último punto de datos     | `/anomalydetector/v1.0/timeseries/last/detect`        |

    [!code-python[initial endpoint and key variables](~/samples-anomaly-detector/quickstarts/python-detect-anomalies.py?name=vars)]

3. Lea el archivo de datos JSON; para ello, ábralo y use `json.load()`.

    [!code-python[Open JSON file and read in the data](~/samples-anomaly-detector/quickstarts/python-detect-anomalies.py?name=fileLoad)]

## <a name="create-a-function-to-send-requests"></a>Creación de una función para enviar solicitudes

1. Cree una función llamada `send_request()` que adopte las variables creadas anteriormente. Después, lleve a cabo los siguiente pasos.

2. Cree un diccionario para los encabezados de solicitud. Establezca el `Content-Type` en `application/json` y agregue su clave de suscripción al encabezado `Ocp-Apim-Subscription-Key`.

3. Envíe la solicitud con `requests.post()`. Combine el punto de conexión y la dirección URL de detección de anomalías para la dirección URL de solicitud completa, e incluya los encabezados y los datos de la solicitud JSON. Y, a continuación, devuelva la respuesta.

    [!code-python[request method](~/samples-anomaly-detector/quickstarts/python-detect-anomalies.py?name=request)]

## <a name="detect-anomalies-as-a-batch"></a>Detección de anomalías como un lote

1. Cree un método llamado `detect_batch()` para detectar anomalías a lo largo de los datos como un lote. Llame al método `send_request()` creado anteriormente con el punto de conexión, dirección URL, la clave de suscripción y datos JSON.

2. Llame a `json.dumps()` en el resultado para aplicarle formato e imprimirlo en la consola.

3. Si la respuesta contiene el campo `code`, imprima el código de error y el mensaje de error.

4. En caso contrario, busque las posiciones de las anomalías en el conjunto de datos. El campo `isAnomaly` de la respuesta contiene un valor booleano sobre si un punto de datos determinado es una anomalía. Recorra en iteración la lista e imprima el índice de los valores `True`. Estos valores se corresponden con el índice de los puntos de datos anómalos, si se detecta alguno.

    [!code-python[detection as a batch](~/samples-anomaly-detector/quickstarts/python-detect-anomalies.py?name=detectBatch)]

## <a name="detect-the-anomaly-status-of-the-latest-data-point"></a>Detección del estado de anomalía del punto de datos más reciente

1. Cree un método llamado `detect_latest()` para determinar si el punto de datos más reciente de la serie de tiempo es una anomalía. Llame al método `send_request()` anterior con el punto de conexión, la dirección URL, la clave de suscripción y los datos JSON. 

2. Llame a `json.dumps()` en el resultado para aplicarle formato e imprimirlo en la consola.

    [!code-python[Latest point detection](~/samples-anomaly-detector/quickstarts/python-detect-anomalies.py?name=detectLatest)]

## <a name="send-the-request"></a>Envío de la solicitud

Llame a los métodos de detección de anomalías que creó anteriormente.

[!code-python[Method calls](~/samples-anomaly-detector/quickstarts/python-detect-anomalies.py?name=methodCalls)]

### <a name="example-response"></a>Respuesta de ejemplo

Se devuelve una respuesta correcta en formato JSON. Haga clic en los siguientes vínculos para ver la respuesta JSON en GitHub:
* [Ejemplo de respuesta de detección por lotes](https://github.com/Azure-Samples/anomalydetector/blob/master/example-data/batch-response.json)
* [Ejemplo de respuesta de detección del punto más reciente](https://github.com/Azure-Samples/anomalydetector/blob/master/example-data/latest-point-response.json)

[!INCLUDE [anomaly-detector-next-steps](../includes/quickstart-cleanup-next-steps.md)]
