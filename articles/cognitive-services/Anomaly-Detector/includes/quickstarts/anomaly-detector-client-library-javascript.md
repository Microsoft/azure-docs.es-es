---
title: Inicio rápido de la biblioteca cliente de JavaScript de Anomaly Detector
titleSuffix: Azure Cognitive Services
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.topic: include
ms.date: 09/10/2020
ms.author: aahi
ms.custom: devx-track-javascript
ms.openlocfilehash: 4a4b6d02845c9767b7ab668dd172da38150fc89e
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "91025009"
---
Comience a usar la biblioteca cliente de Anomaly Detector para JavaScript. Siga estos pasos para instalar el paquete y probar el código de ejemplo para realizar tareas básicas. El servicio de Anomaly Detector le permite detectar anomalías en los datos de serie temporal mediante el uso automático de los mejores modelos, independientemente del sector, el escenario o el volumen de datos.

Use la biblioteca cliente de Anomaly Detector para JavaScript para hacer lo siguiente:

* Detectar anomalías en el conjunto de datos de serie temporal como una solicitud por lotes
* Detectar el estado de anomalía del punto de datos más reciente en la serie temporal
* Detectar puntos de cambio de tendencia en el conjunto de datos.

[Documentación de referencia de la biblioteca](https://go.microsoft.com/fwlink/?linkid=2090788) | [Código fuente de la biblioteca](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/cognitiveservices/AnomalyDetector) | [Paquete (npm)](https://www.npmjs.com/package/%40azure/ai-anomaly-detector) | [Buscar el código en GitHub](https://github.com/Azure-Samples/cognitive-services-quickstart-code/tree/master/javascript/AnomalyDetector)

## <a name="prerequisites"></a>Requisitos previos

* Una suscripción a Azure: [cree una cuenta gratuita](https://azure.microsoft.com/free/cognitive-services)
* La versión actual de [Node.js](https://nodejs.org/)
* Cuando tenga la suscripción de Azure, <a href="https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesAnomalyDetector"  title="Creación de un recurso de Anomaly Detector"  target="_blank">cree un recurso de Anomaly Detector <span class="docon docon-navigate-external x-hidden-focus"></span></a> en Azure Portal para obtener la clave y el punto de conexión. Espere a que se implemente y haga clic en el botón **Ir al recurso**.
    * Necesitará la clave y el punto de conexión del recurso que cree para conectar la aplicación a Anomaly Detector API. En una sección posterior de este mismo inicio rápido pegará la clave y el punto de conexión en el código siguiente.
    Puede usar el plan de tarifa gratis (`F0`) para probar el servicio y actualizarlo más adelante a un plan de pago para producción.

## <a name="setting-up"></a>Instalación

[!INCLUDE [anomaly-detector-environment-variables](../environment-variables.md)]

### <a name="create-a-new-nodejs-application"></a>Creación de una aplicación Node.js

En una ventana de la consola (como cmd, PowerShell o Bash), cree un directorio para la aplicación y vaya a él. 

```console
mkdir myapp && cd myapp
```

Ejecute el comando `npm init` para crear una aplicación de nodo con un archivo `package.json`. 

```console
npm init
```

Cree un archivo llamado `index.js` e importe las bibliotecas siguientes:

[!code-javascript[Import statements](~/cognitive-services-quickstart-code/javascript/AnomalyDetector/anomaly_detector_quickstart.js?name=imports)]

Cree variables para el punto de conexión y la clave de Azure del recurso. Si ha creado la variable de entorno después de haber iniciado la aplicación, deberá cerrar y volver a abrir el editor, el IDE o el shell que lo ejecuta para acceder a la variable. Cree otra variable para el archivo de datos de ejemplo que se descargará en un paso posterior y una lista vacía para los puntos de datos. Después, cree un objeto `ApiKeyCredentials` para que contenga la clave.

[!code-javascript[Initial endpoint and key variables](~/cognitive-services-quickstart-code/javascript/AnomalyDetector/anomaly_detector_quickstart.js?name=vars)]

### <a name="install-the-client-library"></a>Instalación de la biblioteca cliente

Instale los paquetes `ms-rest-azure` y `azure-cognitiveservices-anomalydetector` de NPM. La biblioteca de análisis de CSV también se usa en este inicio rápido:

```console
npm install @azure/ai-anomaly-detector @azure/ms-rest-js csv-parse
```

el archivo `package.json` de la aplicación se actualizará con las dependencias.

## <a name="object-model"></a>Modelo de objetos

El cliente de Anomaly Detector es un objeto [AnomalyDetectorClient](https://docs.microsoft.com/javascript/api/@azure/cognitiveservices-anomalydetector/anomalydetectorclient?view=azure-node-latest) que se autentica en Azure mediante la clave. El cliente puede realizar detección de anomalías en un conjunto de datos completo mediante [entireDetect()](https://docs.microsoft.com/javascript/api/@azure/cognitiveservices-anomalydetector/anomalydetectorclient?view=azure-node-latest#entiredetect-request--servicecallback-entiredetectresponse--), o en el punto de datos más reciente mediante [LastDetect()](https://docs.microsoft.com/javascript/api/@azure/cognitiveservices-anomalydetector/anomalydetectorclient?view=azure-node-latest#lastdetect-request--msrest-requestoptionsbase-). El método [ChangePointDetectAsync](https://go.microsoft.com/fwlink/?linkid=2090788) detecta puntos que marcan los cambios en una tendencia. 

Los datos de serie temporal se envían como una serie de objetos [Point](https://docs.microsoft.com/javascript/api/@azure/cognitiveservices-anomalydetector/point?view=azure-node-latest) en un objeto [Request](https://docs.microsoft.com/javascript/api/@azure/cognitiveservices-anomalydetector/request?view=azure-node-latest). El objeto `Request` contiene propiedades para describir los datos ([Granularidad](https://docs.microsoft.com/javascript/api/@azure/cognitiveservices-anomalydetector/request?view=azure-node-latest#granularity), por ejemplo), así como los parámetros para la detección de anomalías. 

La respuesta de Anomaly Detector es un objeto [LastDetectResponse](https://docs.microsoft.com/javascript/api/@azure/cognitiveservices-anomalydetector/lastdetectresponse?view=azure-node-latest), [EntireDetectResponse](https://docs.microsoft.com/javascript/api/@azure/cognitiveservices-anomalydetector/entiredetectresponse?view=azure-node-latest) o [ChangePointDetectResponse](https://go.microsoft.com/fwlink/?linkid=2090788), según el método usado. 

## <a name="code-examples"></a>Ejemplos de código 

Estos fragmentos de código muestran cómo realizar las siguientes acciones con la biblioteca cliente de Anomaly Detector para Node.js:

* [Autenticar el cliente](#authenticate-the-client)
* [Cargar un conjunto de datos de serie temporal desde un archivo](#load-time-series-data-from-a-file)
* [Detectar anomalías en todo el conjunto de datos](#detect-anomalies-in-the-entire-data-set) 
* [Detectar el estado de anomalía del punto de datos más reciente](#detect-the-anomaly-status-of-the-latest-data-point)
* [Detección de los puntos de cambio en el conjunto de datos](#detect-change-points-in-the-data-set)

## <a name="authenticate-the-client"></a>Autenticar el cliente

Cree una instancia de un objeto [AnomalyDetectorClient](https://docs.microsoft.com/javascript/api/@azure/cognitiveservices-anomalydetector/anomalydetectorclient?view=azure-node-latest) con el punto de conexión y las credenciales.

[!code-javascript[Authentication](~/cognitive-services-quickstart-code/javascript/AnomalyDetector/anomaly_detector_quickstart.js?name=authentication)]

## <a name="load-time-series-data-from-a-file"></a>Cargar datos de serie temporal desde un archivo

Descargue los datos de ejemplo de este inicio rápido desde [GitHub](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/javascript/AnomalyDetector/request-data.csv):
1. En el explorador, haga clic con el botón derecho en **Raw** (Sin formato).
2. Haga clic en **Save link as** (Guardar vínculo como).
3. Guarde el archivo como .csv en el directorio de aplicaciones.

Estos datos de serie temporal tienen el formato de un archivo .csv y se enviarán a Anomaly Detector API.

Lea el archivo de datos con el método `readFileSync()` de la biblioteca de análisis de CSV y analice el archivo con `parse()`. Para cada línea, inserte un objeto [Point](https://docs.microsoft.com/javascript/api/@azure/cognitiveservices-anomalydetector/point?view=azure-node-latest) que contenga la marca de tiempo y el valor numérico.

[!code-javascript[Read the data file](~/cognitive-services-quickstart-code/javascript/AnomalyDetector/anomaly_detector_quickstart.js?name=readFile)]

## <a name="detect-anomalies-in-the-entire-data-set"></a>Detectar anomalías en todo el conjunto de datos 

Llame a la API para detectar anomalías en la serie temporal completa como un lote mediante el método [entireDetect()](https://docs.microsoft.com/javascript/api/@azure/cognitiveservices-anomalydetector/anomalydetectorclient?view=azure-node-latest#entiredetect-request--msrest-requestoptionsbase-) del cliente. Almacene el objeto [EntireDetectResponse](https://docs.microsoft.com/javascript/api/@azure/cognitiveservices-anomalydetector/entiredetectresponse?view=azure-node-latest) devuelto. Itere la lista `isAnomaly` de la respuesta e imprima el índice de los valores `true`. Estos valores se corresponden con el índice de los puntos de datos anómalos, si se detecta alguno.

[!code-javascript[Batch detection function](~/cognitive-services-quickstart-code/javascript/AnomalyDetector/anomaly_detector_quickstart.js?name=batchCall)]

## <a name="detect-the-anomaly-status-of-the-latest-data-point"></a>Detección del estado de anomalía del punto de datos más reciente

Llame a la API Anomaly Detector para determinar si el punto de datos más reciente es una anomalía mediante el método [lastDetect()](https://docs.microsoft.com/javascript/api/@azure/cognitiveservices-anomalydetector/anomalydetectorclient?view=azure-node-latest#lastdetect-request--msrest-requestoptionsbase-) del cliente y almacene el objeto [LastDetectResponse](https://docs.microsoft.com/javascript/api/@azure/cognitiveservices-anomalydetector/lastdetectresponse?view=azure-node-latest) devuelto. El valor `isAnomaly` de la respuesta es un valor booleano que especifica el estado de anomalía de ese punto.  

[!code-javascript[Last point detection function](~/cognitive-services-quickstart-code/javascript/AnomalyDetector/anomaly_detector_quickstart.js?name=lastDetection)]

## <a name="detect-change-points-in-the-data-set"></a>Detección de los puntos de cambio en el conjunto de datos

Llame a la API para detectar los puntos de cambio de la serie temporal con el método[detectChangePoint()](https://go.microsoft.com/fwlink/?linkid=2090788) del cliente. Almacene el objeto [ChangePointDetectResponse](https://go.microsoft.com/fwlink/?linkid=2090788) devuelto. Itere la lista `isChangePoint` de la respuesta e imprima el índice de los valores `true`. Estos valores corresponden a los índices de los puntos de cambio de tendencia, si se encuentra alguno.

[!code-javascript[detect change points](~/cognitive-services-quickstart-code/javascript/AnomalyDetector/anomaly_detector_quickstart.js?name=changePointDetection)]

## <a name="run-the-application"></a>Ejecución de la aplicación

Ejecute la aplicación con el comando `node` en el archivo de inicio rápido.

```console
node index.js
```

[!INCLUDE [anomaly-detector-next-steps](../quickstart-cleanup-next-steps.md)]
