---
title: 'Inicio rápido: Detección de anomalías en los datos de serie temporal mediante API REST Anomaly Detector y Java'
titleSuffix: Azure Cognitive Services
description: Aprenda a usar API Anomaly Detector para detectar anomalías en la serie de datos como un lote o en la transmisión de datos.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: anomaly-detector
ms.topic: quickstart
ms.date: 06/30/2020
ms.custom: devx-track-java
ms.author: aahi
ms.openlocfilehash: 42edf9cd1687314f2ef168969791cb39f43851d3
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87323696"
---
# <a name="quickstart-detect-anomalies-in-your-time-series-data-using-the-anomaly-detector-rest-api-and-java"></a>Inicio rápido: Detección de anomalías en los datos de serie temporal mediante API REST Anomaly Detector y Java

Use este inicio rápido para comenzar a usar los dos modos de detección de la API Anomaly Detector para detectar anomalías en los datos de serie temporal. Esta aplicación de Java envía dos solicitudes de API que contienen datos de serie temporal con formato JSON y obtiene las respuestas.

| Solicitud a la API                                        | Salida de la aplicación                                                                                                                         |
|----------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------|
| Detección de anomalías como un lote                        | La respuesta JSON que contiene el estado de la anomalía (y otros datos) para cada punto de datos en los datos de serie temporal y las posiciones de las anomalías detectadas. |
| Detección del estado de anomalía del punto de datos más reciente | La respuesta JSON que contiene el estado de la anomalía (y otros datos) para el punto de datos más reciente en los datos de serie temporal.                                                                                                                                         |

 Si bien esta aplicación está escrita en Java, la API es un servicio web RESTful compatible con la mayoría de los lenguajes de programación. El código fuente de este inicio rápido está disponible en [GitHub](https://github.com/Azure-Samples/AnomalyDetector/blob/master/quickstarts/java-detect-anomalies.java).

## <a name="prerequisites"></a>Requisitos previos

- Una suscripción a Azure: [cree una cuenta gratuita](https://azure.microsoft.com/free/)
- Cuando tenga la suscripción de Azure, <a href="https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesAnomalyDetector"  title="Creación de un recurso de Anomaly Detector"  target="_blank">cree un recurso de Anomaly Detector <span class="docon docon-navigate-external x-hidden-focus"></span></a> en Azure Portal para obtener la clave y el punto de conexión. Espere a que se implemente y haga clic en el botón **Ir al recurso**.
    - Necesitará la clave y el punto de conexión del recurso que cree para conectar la aplicación a Anomaly Detector API. En una sección posterior de este mismo inicio rápido pegará la clave y el punto de conexión en el código siguiente.
    Puede usar el plan de tarifa gratis (`F0`) para probar el servicio y actualizarlo más adelante a un plan de pago para producción.
- Kit de desarrollo de [Java&trade; (JDK) 7](https://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html) o superior.
- Importación de estas bibliotecas desde el repositorio de Maven
    - Paquete de [JSON en Java](https://mvnrepository.com/artifact/org.json/json)
    - Paquete de [Apache HttpClient](https://mvnrepository.com/artifact/org.apache.httpcomponents/httpclient)

- Un archivo JSON que contiene los puntos de datos de la serie temporal. Los datos de ejemplo de este inicio rápido están disponibles en [GitHub](https://github.com/Azure-Samples/anomalydetector/blob/master/example-data/request-data.json).

[!INCLUDE [anomaly-detector-environment-variables](../includes/environment-variables.md)]

## <a name="create-a-new-application"></a>Creación de una aplicación

1. Cree un proyecto de Java e importe las bibliotecas siguientes.
    
    [!code-java[Import statements](~/samples-anomaly-detector/quickstarts/java-detect-anomalies.java?name=imports)]

2. Cree variables para la clave de suscripción y el punto de conexión. A continuación se muestran los URI puede usar para la detección de anomalías. Estos se agregarán a su punto de conexión de servicio más adelante para crear las direcciones URL de solicitud de API.

    |Método de detección  |URI  |
    |---------|---------|
    |Detección por lotes    | `/anomalydetector/v1.0/timeseries/entire/detect`        |
    |Detección en el último punto de datos     | `/anomalydetector/v1.0/timeseries/last/detect`        |

    [!code-java[Initial key and endpoint variables](~/samples-anomaly-detector/quickstarts/java-detect-anomalies.java?name=vars)]

## <a name="create-a-function-to-send-requests"></a>Creación de una función para enviar solicitudes

1. Cree una función llamada `sendRequest()` que adopte las variables creadas anteriormente. Después, lleve a cabo los siguiente pasos.

2. Cree un objeto `CloseableHttpClient` que pueda enviar solicitudes a la API. Envíe la solicitud a un objeto de solicitud `HttpPost` mediante la combinación del punto de conexión y la dirección URL de Anomaly Detector.

3. Use la función `setHeader()` de la solicitud para establecer el encabezado `Content-Type` en `application/json` y agregue su clave de suscripción al encabezado `Ocp-Apim-Subscription-Key`.

4. Use la función `setEntity()` de la solicitud para los datos que se van a enviar.

5. Use la función `execute()` del cliente para enviar la solicitud y guárdela en un objeto `CloseableHttpResponse`.

6. Cree un objeto `HttpEntity` para almacenar el contenido de respuesta. Obtenga el contenido con `getEntity()`. Si la respuesta no está vacía, se devuelve.

    [!code-java[API request method](~/samples-anomaly-detector/quickstarts/java-detect-anomalies.java?name=request)]

## <a name="detect-anomalies-as-a-batch"></a>Detección de anomalías como un lote

1. Cree un método llamado `detectAnomaliesBatch()` para detectar anomalías a lo largo de los datos como un lote. Llame al método `sendRequest()` creado anteriormente con el punto de conexión, dirección URL, la clave de suscripción y datos JSON. Obtenga el resultado e imprímalo en la consola.

2. Si la respuesta contiene el campo `code`, imprima el código de error y el mensaje de error.

3. En caso contrario, busque las posiciones de las anomalías en el conjunto de datos. El campo `isAnomaly` de la respuesta contiene un valor booleano sobre si un punto de datos determinado es una anomalía. Obtenga la matriz JSON e itere mediante ella, imprimiendo el índice de cualquier valor `true`. Estos valores se corresponden con el índice de los puntos de datos anómalos, si se detecta alguno.

    [!code-java[Method for batch detection](~/samples-anomaly-detector/quickstarts/java-detect-anomalies.java?name=detectBatch)]

## <a name="detect-the-anomaly-status-of-the-latest-data-point"></a>Detección del estado de anomalía del punto de datos más reciente

Cree un método llamado `detectAnomaliesLatest()` para detectar el estado de las anomalías del último punto de datos del conjunto de datos. Llame al método `sendRequest()` creado anteriormente con el punto de conexión, dirección URL, la clave de suscripción y datos JSON. Obtenga el resultado e imprímalo en la consola.

[!code-java[Latest point detection method](~/samples-anomaly-detector/quickstarts/java-detect-anomalies.java?name=detectLatest)]

## <a name="load-your-time-series-data-and-send-the-request"></a>Carga de datos de la serie temporal y envío de la solicitud

1. En el método main de la aplicación, lea el archivo JSON que contiene los datos que se agregarán a las solicitudes de lectura.

2. Llame a las dos funciones de detección de anomalías creadas anteriormente.

    [!code-java[Main method](~/samples-anomaly-detector/quickstarts/java-detect-anomalies.java?name=main)]

### <a name="example-response"></a>Respuesta de ejemplo

Se devuelve una respuesta correcta en formato JSON. Haga clic en los siguientes vínculos para ver la respuesta JSON en GitHub:
* [Ejemplo de respuesta de detección por lotes](https://github.com/Azure-Samples/anomalydetector/blob/master/example-data/batch-response.json)
* [Ejemplo de respuesta de detección del punto más reciente](https://github.com/Azure-Samples/anomalydetector/blob/master/example-data/latest-point-response.json)

[!INCLUDE [anomaly-detector-next-steps](../includes/quickstart-cleanup-next-steps.md)]
