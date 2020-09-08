---
title: 'Inicio rápido: Detección de anomalías en los datos de serie temporal mediante API REST Anomaly Detector y C#'
titleSuffix: Azure Cognitive Services
description: Aprenda a usar API Anomaly Detector para detectar anomalías en la serie de datos como un lote o en la transmisión de datos.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: anomaly-detector
ms.topic: quickstart
ms.date: 06/30/2020
ms.author: aahi
ms.custom: devx-track-csharp
ms.openlocfilehash: a364588d77fb24e96c831ce541c5bb4e63d93e98
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88922351"
---
# <a name="quickstart-detect-anomalies-in-your-time-series-data-using-the-anomaly-detector-rest-api-and-c"></a>Inicio rápido: Detección de anomalías en los datos de serie temporal mediante API REST Anomaly Detector y C#

Use este inicio rápido para comenzar a usar los dos modos de detección de la API Anomaly Detector para detectar anomalías en los datos de serie temporal. Esta aplicación de C# envía dos solicitudes de API que contienen datos de serie temporal con formato JSON y obtiene las respuestas.

| Solicitud a la API                                        | Salida de la aplicación                                                                                                                                         |
|----------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Detección de anomalías como un lote                        | La respuesta JSON que contiene el estado de la anomalía (y otros datos) para cada punto de datos en los datos de serie temporal y las posiciones de las anomalías detectadas. |
| Detección del estado de anomalía del punto de datos más reciente | La respuesta JSON que contiene el estado de la anomalía (y otros datos) para el punto de datos más reciente en los datos de serie temporal.                                        |

 Si bien esta aplicación está escrita en C#, la API es un servicio web RESTful compatible con la mayoría de los lenguajes de programación. El código fuente de este inicio rápido está disponible en [GitHub](https://github.com/Azure-Samples/AnomalyDetector/blob/master/quickstarts/csharp-detect-anomalies.cs).

## <a name="prerequisites"></a>Requisitos previos

- Una suscripción a Azure: [cree una cuenta gratuita](https://azure.microsoft.com/free/cognitive-services)
- Cuando tenga la suscripción de Azure, <a href="https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesAnomalyDetector"  title="Creación de un recurso de Anomaly Detector"  target="_blank">cree un recurso de Anomaly Detector <span class="docon docon-navigate-external x-hidden-focus"></span></a> en Azure Portal para obtener la clave y el punto de conexión. Espere a que se implemente y haga clic en el botón **Ir al recurso**.
    - Necesitará la clave y el punto de conexión del recurso que cree para conectar la aplicación a Anomaly Detector API. En una sección posterior de este mismo inicio rápido pegará la clave y el punto de conexión en el código siguiente.
    Puede usar el plan de tarifa gratis (`F0`) para probar el servicio y actualizarlo más adelante a un plan de pago para producción.
- Cualquier edición de [Visual Studio 2017 o versiones posteriores](https://visualstudio.microsoft.com/downloads/).
- El marco [Json.NET](https://www.newtonsoft.com/json), disponible como un paquete NuGet. Para instalar Newtonsoft.Json como un paquete NuGet en Visual Studio, siga estos pasos:

    1. Haga clic con el botón derecho en el proyecto en el **Explorador de soluciones**.
    2. Seleccione **Administrar paquetes NuGet**.
    3. Busque *Newtonsoft.json* e instale el paquete.

- Si usa Linux o MacOS, esta aplicación puede ejecutarse con [Mono](https://www.mono-project.com/).

- Un archivo JSON que contiene los puntos de datos de la serie temporal. Los datos de ejemplo de este inicio rápido están disponibles en [GitHub](https://github.com/Azure-Samples/anomalydetector/blob/master/example-data/request-data.json).

[!INCLUDE [anomaly-detector-environment-variables](../includes/environment-variables.md)]

## <a name="create-a-new-application"></a>Creación de una aplicación

1. En Visual Studio, cree una solución de consola y agregue los siguientes paquetes.

    [!code-csharp[using statements](~/samples-anomaly-detector/quickstarts/csharp-detect-anomalies.cs?name=usingStatements)]


2. Cree variables para la clave de suscripción y el punto de conexión. A continuación se muestran los URI puede usar para la detección de anomalías. Estos se agregarán a su punto de conexión de servicio más adelante para crear las direcciones URL de solicitud de API.

    | Método de detección                   | URI                                              |
    |------------------------------------|--------------------------------------------------|
    | Detección por lotes                    | `/anomalydetector/v1.0/timeseries/entire/detect` |
    | Detección en el último punto de datos | `/anomalydetector/v1.0/timeseries/last/detect`   |

    [!code-csharp[initial variables for endpoint, key and data file](~/samples-anomaly-detector/quickstarts/csharp-detect-anomalies.cs?name=vars)]

## <a name="create-a-function-to-send-requests"></a>Creación de una función para enviar solicitudes

1. Cree una función asincrónica llamada `Request` que adopte las variables creadas anteriormente.

2. Establezca el protocolo de seguridad del cliente y la información de encabezado con un objeto `HttpClient`. No olvide agregar la clave de suscripción al encabezado `Ocp-Apim-Subscription-Key`. A continuación, cree un objeto `StringContent` para la solicitud.

3. Envíe la solicitud con `PostAsync()` y, a continuación, devuelva la respuesta.

    [!code-csharp[Request method](~/samples-anomaly-detector/quickstarts/csharp-detect-anomalies.cs?name=requestMethod)]

## <a name="detect-anomalies-as-a-batch"></a>Detección de anomalías como un lote

1. Cree una nueva función llamada `detectAnomaliesBatch()`. Construya la solicitud y envíela mediante una llamada a la función `Request()` con el punto de conexión, la clave de suscripción, la dirección URL de la detección de anomalías por lotes y los datos de la serie temporal.

2. Deserialice el objeto JSON y escríbalo en la consola.

3. Si la respuesta contiene el campo `code`, imprima el código de error y el mensaje de error.

4. En caso contrario, busque las posiciones de las anomalías en el conjunto de datos. El campo `isAnomaly` de la respuesta contiene una matriz de valores booleanos, cada uno de los cuales indica si un punto de datos es una anomalía. Convierta esto en una matriz de cadenas con la función `ToObject<bool[]>()` del objeto de respuesta. Itere la matriz e imprima el índice de los valores `true`. Estos valores se corresponden con el índice de los puntos de datos anómalos, si se detecta alguno.

    [!code-csharp[Detect anomalies batch](~/samples-anomaly-detector/quickstarts/csharp-detect-anomalies.cs?name=detectAnomaliesBatch)]


## <a name="detect-the-anomaly-status-of-the-latest-data-point"></a>Detección del estado de anomalía del punto de datos más reciente

1. Cree una nueva función llamada `detectAnomaliesLatest()`. Construya la solicitud y envíela mediante una llamada a la función `Request()` con el punto de conexión, la clave de suscripción, la dirección URL de la última detección de anomalías de puntos y los datos de la serie temporal.

2. Deserialice el objeto JSON y escríbalo en la consola.

    [!code-csharp[Detect anomalies latest](~/samples-anomaly-detector/quickstarts/csharp-detect-anomalies.cs?name=detectAnomaliesLatest)]

## <a name="load-your-time-series-data-and-send-the-request"></a>Carga de datos de la serie temporal y envío de la solicitud

1. En el método main de la aplicación, cargue los datos JSON de la serie temporal con `File.ReadAllText()`.

2. Llame a las funciones de detección de anomalías creadas anteriormente. Use `System.Console.ReadKey()` para mantener abierta la ventana de consola después de ejecutar la aplicación.

    [!code-csharp[Main method](~/samples-anomaly-detector/quickstarts/csharp-detect-anomalies.cs?name=main)]

### <a name="example-response"></a>Respuesta de ejemplo

Se devuelve una respuesta correcta en formato JSON. Haga clic en los siguientes vínculos para ver la respuesta JSON en GitHub:
* [Ejemplo de respuesta de detección por lotes](https://github.com/Azure-Samples/anomalydetector/blob/master/example-data/batch-response.json)
* [Ejemplo de respuesta de detección del punto más reciente](https://github.com/Azure-Samples/anomalydetector/blob/master/example-data/latest-point-response.json)

[!INCLUDE [anomaly-detector-next-steps](../includes/quickstart-cleanup-next-steps.md)]
