---
title: Inicio rápido de la biblioteca cliente de .NET de Anomaly Detector
titleSuffix: Azure Cognitive Services
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.topic: include
ms.date: 09/04/2020
ms.author: aahi
ms.openlocfilehash: 6f1c016efb300dea2cdef91c84bb901cffd09fa0
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "91025003"
---
Comience a usar la biblioteca cliente de Anomaly Detector para .NET. Siga estos pasos para instalar el paquete y probar el código de ejemplo para realizar tareas básicas. El servicio de Anomaly Detector le permite detectar anomalías en los datos de serie temporal mediante el uso automático de los mejores modelos, independientemente del sector, el escenario o el volumen de datos.

Use la biblioteca cliente de Anomaly Detector para .NET para las siguientes acciones:

* Detectar anomalías en el conjunto de datos de serie temporal como una solicitud por lotes
* Detectar el estado de anomalía del punto de datos más reciente en la serie temporal
* Detectar puntos de cambio de tendencia en el conjunto de datos.

[Documentación de referencia de la biblioteca](https://aka.ms/anomaly-detector-dotnet-ref) | [Código fuente de la biblioteca](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/cognitiveservices/AnomalyDetector) | [Paquete (NuGet)](https://www.nuget.org/packages/Azure.AI.AnomalyDetector/3.0.0-preview.2) | [Buscar el código en GitHub](https://github.com/Azure-Samples/AnomalyDetector/blob/master/quickstarts/sdk/csharp-sdk-sample.cs)

## <a name="prerequisites"></a>Prerrequisitos

* Una suscripción a Azure: [cree una cuenta gratuita](https://azure.microsoft.com/free/cognitive-services)
* La versión actual de [.NET Core](https://dotnet.microsoft.com/download/dotnet-core)
* Cuando tenga la suscripción de Azure, <a href="https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesAnomalyDetector"  title="Creación de un recurso de Anomaly Detector"  target="_blank">cree un recurso de Anomaly Detector <span class="docon docon-navigate-external x-hidden-focus"></span></a> en Azure Portal para obtener la clave y el punto de conexión. Espere a que se implemente y haga clic en el botón **Ir al recurso**.
    * Necesitará la clave y el punto de conexión del recurso que cree para conectar la aplicación a Anomaly Detector API. En una sección posterior de este mismo inicio rápido pegará la clave y el punto de conexión en el código siguiente.
    Puede usar el plan de tarifa gratis (`F0`) para probar el servicio y actualizarlo más adelante a un plan de pago para producción.

## <a name="setting-up"></a>Instalación

[!INCLUDE [anomaly-detector-environment-variables](../environment-variables.md)]

### <a name="create-a-new-net-core-application"></a>Creación de una aplicación de .NET Core

En una ventana de consola (por ejemplo, cmd, PowerShell o Bash), use el comando `dotnet new` para crear una nueva aplicación de consola con el nombre `anomaly-detector-quickstart`. Este comando crea un sencillo proyecto "Hola mundo" con un solo archivo de origen de C#: *Program.cs*.

```dotnetcli
dotnet new console -n anomaly-detector-quickstart
```

Cambie el directorio a la carpeta de aplicaciones recién creada. Para compilar la aplicación:

```dotnetcli
dotnet build
```

La salida de la compilación no debe contener advertencias ni errores.

```output
...
Build succeeded.
 0 Warning(s)
 0 Error(s)
...
```

### <a name="install-the-client-library"></a>Instalación de la biblioteca cliente

Dentro del directorio de aplicaciones, instale la biblioteca cliente de Anomaly Detector para .NET con el siguiente comando:

```dotnetcli
dotnet add package Azure.AI.AnomalyDetector --version 3.0.0-preview.2
```

En el directorio del proyecto, abra el archivo *program.cs* y agregue lo siguiente mediante `directives`:

[!code-csharp[using statements](~/samples-anomaly-detector/quickstarts/sdk/csharp-sdk-sample.cs?name=usingStatements)]

En el método `main()` de la aplicación, cree variables para la ubicación de Azure del recurso y la clave como una variable de entorno. Si ha creado la variable de entorno una vez iniciada la aplicación, el editor, IDE o shell que se esté ejecutando se deberá cerrar y volver a cargar para tener acceso a la variable.

[!code-csharp[Main method](~/samples-anomaly-detector/quickstarts/sdk/csharp-sdk-sample.cs?name=mainMethod)]

## <a name="object-model"></a>Modelo de objetos

El cliente de Anomaly Detector es un objeto [AnomalyDetectorClient](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.anomalydetector.anomalydetectorclient) que se autentica en Azure mediante [ApiKeyServiceClientCredentials](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.anomalydetector.apikeyserviceclientcredentials), que contiene la clave. El cliente puede realizar detección de anomalías en un conjunto de datos completo mediante [EntireDetectAsync()](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.anomalydetector.anomalydetectorclientextensions.entiredetectasync), o en el punto de datos más reciente mediante [LastDetectAsync()](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.anomalydetector.anomalydetectorclientextensions.lastdetectasync). El método [ChangePointDetectAsync](https://aka.ms/anomaly-detector-dotnet-ref) detecta puntos que marcan los cambios en una tendencia.

Los datos de serie temporal se envían como una serie de objetos [Point](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.anomalydetector.models.request.series?view=azure-dotnet-preview#Microsoft_Azure_CognitiveServices_AnomalyDetector_Models_Request_Series) en un objeto [Request](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.anomalydetector.models.request). El objeto `Request` contiene propiedades para describir los datos ([Granularidad](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.anomalydetector.models.request.granularity), por ejemplo), así como los parámetros para la detección de anomalías.

La respuesta de Anomaly Detector es un objeto [EntireDetectResponse](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.anomalydetector.models.entiredetectresponse), [LastDetectResponse](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.anomalydetector.models.lastdetectresponse) o [changePointDetectResponse](https://aka.ms/anomaly-detector-dotnet-ref), según el método usado.

## <a name="code-examples"></a>Ejemplos de código

Estos fragmentos de código muestran cómo realizar las siguientes acciones con la biblioteca cliente de Anomaly Detector para .NET:

* [Autenticar el cliente](#authenticate-the-client)
* [Cargar un conjunto de datos de serie temporal desde un archivo](#load-time-series-data-from-a-file)
* [Detectar anomalías en todo el conjunto de datos](#detect-anomalies-in-the-entire-data-set)
* [Detectar el estado de anomalía del punto de datos más reciente](#detect-the-anomaly-status-of-the-latest-data-point)
* [Detección de los puntos de cambio en el conjunto de datos](#detect-change-points-in-the-data-set)

## <a name="authenticate-the-client"></a>Autenticar el cliente

En un nuevo método, cree una instancia de un cliente con la clave y el punto de conexión. Cree un objeto [ApiKeyServiceClientCredentials](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.anomalydetector.apikeyserviceclientcredentials?view=azure-dotnet-preview) con la clave y úselo con el punto de conexión para crear un objeto [AnomalyDetectorClient](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.anomalydetector.anomalydetectorclient?view=azure-dotnet-preview).

[!code-csharp[Client authentication function](~/samples-anomaly-detector/quickstarts/sdk/csharp-sdk-sample.cs?name=createClient)]

## <a name="load-time-series-data-from-a-file"></a>Cargar datos de serie temporal desde un archivo

Descargue los datos de ejemplo de este inicio rápido desde [GitHub](https://github.com/Azure-Samples/AnomalyDetector/blob/master/example-data/request-data.csv):
1. En el explorador, haga clic con el botón derecho en **Raw** (Sin formato).
2. Haga clic en **Save link as** (Guardar vínculo como).
3. Guarde el archivo como .csv en el directorio de aplicaciones.

Estos datos de serie temporal tienen el formato de un archivo .csv y se enviarán a Anomaly Detector API.

Cree un nuevo método para leer los datos de serie temporal y agréguelo a un objeto [Request](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.anomalydetector.models.request?view=azure-dotnet-preview). Llame a `File.ReadAllLines()` con la ruta de acceso del archivo y cree una lista de objetos [Point](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.anomalydetector.models.point?view=azure-dotnet-preview) y realice el script de los nuevos caracteres de línea. Extraiga los valores y separe la marca de fecha de su valor numérico y agréguelos a un nuevo objeto `Point`.

Cree un objeto `Request` con la serie de puntos y `Granularity.Daily` para [Granularidad](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.anomalydetector.models.granularity?view=azure-dotnet-preview) (o periodicidad) de los puntos de datos.

[!code-csharp[load the time series data file](~/samples-anomaly-detector/quickstarts/sdk/csharp-sdk-sample.cs?name=GetSeriesFromFile)]

## <a name="detect-anomalies-in-the-entire-data-set"></a>Detectar anomalías en todo el conjunto de datos

Cree un método para llamar al método [EntireDetectAsync()](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.anomalydetector.anomalydetectorclientextensions.entiredetectasync?view=azure-dotnet-preview#Microsoft_Azure_CognitiveServices_AnomalyDetector_AnomalyDetectorClientExtensions_EntireDetectAsync_Microsoft_Azure_CognitiveServices_AnomalyDetector_IAnomalyDetectorClient_Microsoft_Azure_CognitiveServices_AnomalyDetector_Models_Request_System_Threading_CancellationToken_) del cliente con el objeto `Request` y espere la respuesta como un objeto [EntireDetectResponse](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.anomalydetector.models.entiredetectresponse?view=azure-dotnet-preview). Si la serie temporal contiene alguna anomalía, itere a través de los valores [IsAnomaly](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.anomalydetector.models.entiredetectresponse.isanomaly?view=azure-dotnet-preview) de la respuesta e imprima cualquiera que sea `true`. Estos valores se corresponden con el índice de los puntos de datos anómalos, si se detecta alguno.

[!code-csharp[EntireDetectSampleAsync() function](~/samples-anomaly-detector/quickstarts/sdk/csharp-sdk-sample.cs?name=entireDatasetExample)]

## <a name="detect-the-anomaly-status-of-the-latest-data-point"></a>Detección del estado de anomalía del punto de datos más reciente

Cree un método para llamar al método [LastDetectAsync()](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.anomalydetector.anomalydetectorclientextensions.lastdetectasync?view=azure-dotnet-preview#Microsoft_Azure_CognitiveServices_AnomalyDetector_AnomalyDetectorClientExtensions_LastDetectAsync_Microsoft_Azure_CognitiveServices_AnomalyDetector_IAnomalyDetectorClient_Microsoft_Azure_CognitiveServices_AnomalyDetector_Models_Request_System_Threading_CancellationToken_) del cliente con el objeto `Request` y espere la respuesta como un objeto [LastDetectResponse](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.anomalydetector.models.lastdetectresponse?view=azure-dotnet-preview). Compruebe el atributo [IsAnomaly](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.anomalydetector.models.lastdetectresponse.isanomaly?view=azure-dotnet-preview) de la respuesta para determinar si el punto de datos más reciente enviado era una anomalía o no.

[!code-csharp[LastDetectSampleAsync() function](~/samples-anomaly-detector/quickstarts/sdk/csharp-sdk-sample.cs?name=latestPointExample)]

## <a name="detect-change-points-in-the-data-set"></a>Detección de los puntos de cambio en el conjunto de datos

Cree un método para llamar al método [DetectChangePointAsync](https://aka.ms/anomaly-detector-dotnet-ref) del cliente con el objeto `Request` y espere la respuesta como un objeto [ChangePointDetectResponse](https://aka.ms/anomaly-detector-dotnet-ref). Compruebe los valores de IsChangePoint de la respuesta e imprima todos los que sean `true`. Estos valores corresponden a los puntos de cambio de tendencia, si se encuentra alguno.

[!code-csharp[DetectChangePoint() function](~/samples-anomaly-detector/quickstarts/sdk/csharp-sdk-sample.cs?name=changePointExample)]

## <a name="run-the-application"></a>Ejecución de la aplicación

Ejecute la aplicación con el comando `dotnet run` desde el directorio de la aplicación.

```dotnetcli
dotnet run
```

[!INCLUDE [anomaly-detector-next-steps](../quickstart-cleanup-next-steps.md)]
