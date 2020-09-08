---
title: 'Tutorial: Detección de anomalías en datos transmitidos con Azure Databricks'
titleSuffix: Azure Cognitive Services
description: Aprenda a usar la API Anomaly Detector y Azure Databricks para supervisar anomalías en los datos.
titlesuffix: Azure Cognitive Services
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: anomaly-detector
ms.topic: tutorial
ms.date: 03/05/2020
ms.author: aahi
ms.openlocfilehash: 95ab400d645a2a2761e39c191cdb85e49e1c7a27
ms.sourcegitcommit: c293217e2d829b752771dab52b96529a5442a190
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/15/2020
ms.locfileid: "88245611"
---
# <a name="tutorial-anomaly-detection-on-streaming-data-using-azure-databricks"></a>Tutorial: Detección de anomalías en datos transmitidos con Azure Databricks

[Azure Databricks](https://azure.microsoft.com/services/databricks/) es un servicio de análisis rápido, sencillo y de colaboración basado en Apache Spark. API Anomaly Detector forma parte de Azure Cognitive Services y proporciona una forma de supervisar los datos de series temporales. Use este tutorial para ejecutar la detección de anomalías en un flujo de datos casi en tiempo real mediante Azure Databricks. En él, ingerirá datos de Twitter mediante Azure Event Hubs y los importará en Azure Databricks mediante el conector de Event Hubs para Spark. Y después, usará la API para detectar anomalías en los datos transmitidos.

La siguiente ilustración muestra el flujo de la aplicación:

![Azure Databricks con Event Hubs y Cognitive Services](../media/tutorials/databricks-cognitive-services-tutorial.png "Azure Databricks con Event Hubs y Cognitive Services")

En este tutorial se describen las tareas siguientes:

> [!div class="checklist"]
> * Creación de un área de trabajo de Azure Databricks
> * Creación de un clúster de Spark en Azure Databricks
> * Creación de una aplicación de Twitter para acceder a datos de streaming
> * Creación de cuadernos en Azure Databricks
> * Asociación de bibliotecas a Event Hubs y la API de Twitter
> * Creación de un recurso de Anomaly Detector y recuperación de la clave de acceso
> * Envío de tweets a Event Hubs
> * Lectura de tweets desde Event Hubs
> * Ejecución de la detección de anomalías en tweets

> [!Note]
> * En este tutorial se presenta un enfoque para implementar la [arquitectura de la solución](https://azure.microsoft.com/solutions/architecture/anomaly-detector-process/) recomendada para API Anomaly Detector.
> * Este tutorial no se puede completar con una suscripción de nivel Gratis (`F0`) de Anomaly Detector API o Azure Databricks.

Cree una [suscripción a Azure](https://azure.microsoft.com/free/cognitive-services), en caso de que no tenga.

## <a name="prerequisites"></a>Requisitos previos

- Un [espacio de nombres de Azure Event Hubs](https://docs.microsoft.com/azure/event-hubs/event-hubs-create) y un centro de eventos.

- La [cadena de conexión](../../../event-hubs/event-hubs-get-connection-string.md) para acceder al espacio de nombres de Event Hubs. El formato de la cadena de conexión debe ser similar a este:

    `Endpoint=sb://<namespace>.servicebus.windows.net/;SharedAccessKeyName=<key name>;SharedAccessKey=<key value>`.

- El nombre de la directiva de acceso compartido y la clave de la directiva de Event Hubs.

Para más información acerca de la creación de un espacio de nombres y un centro de eventos, consulte el [tutorial de inicio rápido](../../../event-hubs/event-hubs-create.md) de Azure Event Hubs.

## <a name="create-an-azure-databricks-workspace"></a>Creación de un área de trabajo de Azure Databricks

En esta sección, se crea un área de trabajo de Azure Databricks desde [Azure Portal](https://portal.azure.com/).

1. En Azure Portal, seleccione **Crear un recurso** > **Análisis** > **Azure Databricks**.

    ![Databricks en Azure Portal](../media/tutorials/azure-databricks-on-portal.png "Databricks de Azure Portal")

3. En **Azure Databricks Service**, proporcione los valores siguientes para crear un área de trabajo de Databricks.


    |Propiedad  |Descripción  |
    |---------|---------|
    |**Workspace name** (Nombre del área de trabajo)     | Proporcione un nombre para el área de trabajo de Databricks        |
    |**Suscripción**     | En el cuadro desplegable, seleccione la suscripción de Azure.        |
    |**Grupos de recursos**     | Especifique si desea crear un nuevo grupo de recursos o utilizar uno existente. Un grupo de recursos es un contenedor que almacena los recursos relacionados con una solución de Azure. Para más información, consulte [Información general del grupo de recursos de Azure](../../../azure-resource-manager/management/overview.md). |
    |**Ubicación**     | Seleccione **Este de EE. UU. 2** o cualquiera de las otras regiones disponibles. Para conocer la disponibilidad de las regiones, consulte [Productos disponibles por región](https://azure.microsoft.com/regions/services/).        |
    |**Plan de tarifa**     |  Elija entre **Standard** o **Premium**. NO Elija **Versión de evaluación**. Para más información sobre estos planes, consulte la [página de precios de Databricks](https://azure.microsoft.com/pricing/details/databricks/).       |

    Seleccione **Crear**.

4. Se tarda unos minutos en crear el área de trabajo.

## <a name="create-a-spark-cluster-in-databricks"></a>Creación de un clúster de Spark en Databricks

1. En Azure Portal, vaya al área de trabajo de Databricks que ha creado y, después, seleccione **Launch Workspace** (Iniciar área de trabajo).

2. Se le redirigirá al portal de Azure Databricks. En el portal, seleccione **Nuevo clúster**.

    ![Databricks en Azure](../media/tutorials/databricks-on-azure.png "Databricks en Azure")

3. En la página **Nuevo clúster**, especifique los valores necesarios para crear un clúster.

    ![Creación de clústeres de Spark para Databricks en Azure](../media/tutorials/create-databricks-spark-cluster.png "Creación de clústeres de Spark para Databricks en Azure")

    Acepte los demás valores predeterminados, salvo los siguientes:

   * Escriba un nombre para el clúster.
   * Para este artículo, cree un clúster con el entorno de ejecución **5.2**. NO seleccione el runtime **5.3**.
   * Asegúrese de que la casilla **Terminar después de \_\_ minutos de inactividad** está activada. Si no se usa el clúster, especifique una duración (en minutos) para que se termine.

     Seleccione **Create cluster** (Crear clúster).
4. La creación del clúster tarda varios minutos. Una vez que el clúster se está ejecutando, puede asociarle notebooks y ejecutar trabajos de Spark.

## <a name="create-a-twitter-application"></a>Creación de una aplicación de Twitter

Para recibir un flujo de tweets, debe crear una aplicación en Twitter. Siga los pasos para crear una aplicación de Twitter y anote los valores que necesita para finalizar este tutorial.

1. Desde un explorador web, vaya a [Twitter Application Management](https://apps.twitter.com/) (Administración de aplicaciones de Twitter) y seleccione **Create New App** (Crear nueva aplicación).

    ![Crear una aplicación de Twitter](../media/tutorials/databricks-create-twitter-app.png "Creación de una aplicación de Twitter")

2. En la página **Create an application** (Crear una aplicación), proporcione los detalles de la nueva aplicación y luego seleccione **Create your Twitter application** (Crear su aplicación de Twitter).

    ![Detalles de la aplicación Twitter](../media/tutorials/databricks-provide-twitter-app-details.png "Detalles de la aplicación Twitter")

3. En la página de la aplicación, seleccione la pestaña **Claves y tokens de acceso** y copie los valores de **Consumer Key** (Clave de consumidor) y **Consumer Secret** (Secreto de consumidor). Además, seleccione **Create my access token** (Crear mi token de acceso) para generar los tokens de acceso. Copie los valores de **Access Token** (Token de acceso) y **Access Token Secret** (Secreto del token de acceso).

    ![Detalles de la aplicación Twitter](../media/tutorials/twitter-app-key-secret.png "Detalles de la aplicación Twitter")

Guarde los valores que ha recuperado de la aplicación de Twitter. Necesitará estos valores más adelante en el tutorial.

## <a name="attach-libraries-to-spark-cluster"></a>Asociación de bibliotecas a un clúster de Spark

En este tutorial, se utilizan las API de Twitter para enviar tweets a Event Hubs. Se utiliza también el [conector de Event Hubs para Apache Spark](https://github.com/Azure/azure-event-hubs-spark) para leer y escribir datos en Azure Event Hubs. Para usar estas API como parte del clúster, puede agregarlas como bibliotecas a Azure Databricks y, a continuación, asociarlas al clúster de Spark. Las siguientes instrucciones muestran cómo agregar las bibliotecas a la carpeta **Shared** del área de trabajo.

1. En el área de trabajo de Azure Databricks, seleccione **Workspace** (Área de trabajo) y, a continuación, haga clic con el botón derecho en **Shared** (Compartidos). En el menú contextual, seleccione **Create** > **Library** (Crear > Biblioteca).

   ![Cuadro de diálogo Add library (Agregar biblioteca)](../media/tutorials/databricks-add-library-option.png "Cuadro de diálogo Add library (Agregar biblioteca)")

2. En la página de nueva biblioteca, en **Source** (Origen) seleccione **Maven**. En **Coordinates** (Coordenadas), escriba la coordenada del paquete que desea agregar. Estas son las coordenadas de Maven para las bibliotecas que se utilizan en este tutorial:

   * Conector de Event Hubs para Spark: `com.microsoft.azure:azure-eventhubs-spark_2.11:2.3.10`
   * API de Twitter: `org.twitter4j:twitter4j-core:4.0.7`

     ![Proporcionar coordenadas de Maven](../media/tutorials/databricks-eventhub-specify-maven-coordinate.png "Proporcionar coordenadas de Maven")

3. Seleccione **Crear**.

4. Seleccione la carpeta en la que ha agregado la biblioteca y, a continuación, seleccione el nombre de la biblioteca.

    ![Seleccionar la biblioteca que desea agregar](../media/tutorials/select-library.png "Selección de la biblioteca que desea agregar")

5. Si no hay ningún clúster en la página de la biblioteca, seleccione **Clusters** (Clústeres) y ejecute el clúster que ha creado. Espere hasta que el estado muestre "Running" (En ejecución) y, después, vuelva a la página de la biblioteca.
En esta página, seleccione el clúster en el que desea utilizar la biblioteca y, después, seleccione **Install** (Instalar). Una vez que la biblioteca esté correctamente asociada con el clúster, el estado cambia inmediatamente a **Installed** (Instalada).

    ![Instalar la biblioteca en el clúster](../media/tutorials/databricks-library-attached.png "Instalación de la biblioteca en el clúster")

6. Repita estos pasos para el paquete de Twitter, `twitter4j-core:4.0.7`.

## <a name="get-a-cognitive-services-access-key"></a>Obtención de una clave de acceso de Cognitive Services

En este tutorial, las [API Anomaly Detector de Azure Cognitive Services](../overview.md) se usan para ejecutar la detección de anomalías en un flujo de tweets casi en tiempo real. Antes de usar las API, debe crear un recurso de Anomaly Detector en Azure y recuperar una clave de acceso para usar las API Anomaly Detector.

1. Inicie sesión en [Azure Portal](https://portal.azure.com/).

2. Seleccione **+ Crear un recurso**.

3. En Azure Marketplace, seleccione **IA y Machine Learning**  > **Ver todo** > **Cognitive Services - Más** > **Anomaly Detector**. O bien puede usar [este vínculo](https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesAnomalyDetector) para ir directamente al cuadro de diálogo **Crear**.

    ![Crear un recurso de Anomaly Detector](../media/tutorials/databricks-cognitive-services-anomaly-detector.png "Creación de un recurso de Anomaly Detector")

4. En el cuadro de diálogo **Create** (Crear), proporcione los siguientes valores:

    |Value |Descripción  |
    |---------|---------|
    |Nombre     | Nombre del recurso de Anomaly Detector.        |
    |Suscripción     | Suscripción de Azure con la que se asociará el recurso.        |
    |Location     | Una ubicación de Azure.        |
    |Plan de tarifa     | El plan de tarifa del servicio. Para más información acerca de los precios de Anomaly Detector, consulte la [página de precios](https://azure.microsoft.com/pricing/details/cognitive-services/anomaly-detector/).        |
    |Resource group     | Especifique si desea crear un nuevo grupo de recursos o seleccionar uno existente.        |


     Seleccione **Crear**.

5. Después de crear el recurso, en la pestaña **Información general**, copie y guarde la dirección URL de **Punto de conexión**, como se muestra en la captura de pantalla. Luego, seleccione **Mostrar claves de acceso**.

    ![Mostrar claves de acceso](../media/tutorials/cognitive-services-get-access-keys.png "Mostrar claves de acceso")

6. En **Claves**, seleccione el icono de copia en la clave que desee usar. Guarde la clave de acceso.

    ![Copia de las claves de acceso](../media/tutorials/cognitive-services-copy-access-keys.png "Copia de las claves de acceso")

## <a name="create-notebooks-in-databricks"></a>Creación de cuadernos en Databricks

En esta sección, se crearán dos cuadernos en el área de trabajo de Databricks con los nombres siguientes:

- **SendTweetsToEventHub**: un cuaderno de productor que se usa para obtener tweets de Twitter y transmitirlos a Event Hubs.
- **AnalyzeTweetsFromEventHub**: un cuaderno de consumidor que se usa para leer los tweets de Event Hubs y ejecutar la detección de anomalías.

1. En el área de trabajo de Azure Databricks, seleccione **Workspace** (Área de trabajo) en el panel izquierdo. En la lista desplegable **Workspace** (Área de trabajo), seleccione **Create** (Crear) y, a continuación, haga clic en **Notebook** (Cuaderno).

    ![Creación de un cuaderno en Databricks](../media/tutorials/databricks-create-notebook.png "Creación de un cuaderno en Databricks")

2. En el cuadro de diálogo **Create Notebook** (Crear cuaderno), escriba **SendTweetsToEventHub** como nombre, seleccione **Scala** como lenguaje y seleccione el clúster de Spark que creó anteriormente.

    ![Creación de un cuaderno en Databricks](../media/tutorials/databricks-notebook-details.png "Creación de un cuaderno en Databricks")

    Seleccione **Crear**.

3. Repita los pasos para crear el cuaderno **AnalyzeTweetsFromEventHub**.

## <a name="send-tweets-to-event-hubs"></a>Envío de tweets a Event Hubs

En el cuaderno **SendTweetsToEventHub**, pegue el código siguiente y reemplace el marcador de posición por los valores del espacio de nombres de Event Hubs y la aplicación de Twitter que creó anteriormente. Este cuaderno extrae la hora de creación y el número de "Me gusta" de los tweets con la palabra clave "Azure" y los transmite como eventos a Event Hubs en tiempo real.

```scala
//
// Send Data to Eventhub
//

import scala.collection.JavaConverters._
import com.microsoft.azure.eventhubs._
import java.util.concurrent._
import com.google.gson.{Gson, GsonBuilder, JsonParser}
import java.util.Date
import scala.util.control._
import twitter4j._
import twitter4j.TwitterFactory
import twitter4j.Twitter
import twitter4j.conf.ConfigurationBuilder

// Event Hub Config
val namespaceName = "[Placeholder: EventHub namespace]"
val eventHubName = "[Placeholder: EventHub name]"
val sasKeyName = "[Placeholder: EventHub access key name]"
val sasKey = "[Placeholder: EventHub access key key]"
val connStr = new ConnectionStringBuilder()
  .setNamespaceName(namespaceName)
  .setEventHubName(eventHubName)
  .setSasKeyName(sasKeyName)
  .setSasKey(sasKey)

// Connect to the Event Hub
val pool = Executors.newScheduledThreadPool(1)
val eventHubClient = EventHubClient.create(connStr.toString(), pool)

def sendEvent(message: String) = {
  val messageData = EventData.create(message.getBytes("UTF-8"))
  eventHubClient.get().send(messageData)
  System.out.println("Sent event: " + message + "\n")
}

case class MessageBody(var timestamp: Date, var favorite: Int)
val gson: Gson = new GsonBuilder().setDateFormat("yyyy-MM-dd'T'HH:mm:ss.SSS'Z'").create()

val twitterConsumerKey = "[Placeholder: Twitter consumer key]"
val twitterConsumerSecret = "[Placeholder: Twitter consumer seceret]"
val twitterOauthAccessToken = "[Placeholder: Twitter oauth access token]"
val twitterOauthTokenSecret = "[Placeholder: Twitter oauth token secret]"

val cb = new ConfigurationBuilder()
cb.setDebugEnabled(true)
  .setOAuthConsumerKey(twitterConsumerKey)
  .setOAuthConsumerSecret(twitterConsumerSecret)
  .setOAuthAccessToken(twitterOauthAccessToken)
  .setOAuthAccessTokenSecret(twitterOauthTokenSecret)

val twitterFactory = new TwitterFactory(cb.build())
val twitter = twitterFactory.getInstance()

// Getting tweets with keyword "Azure" and sending them to the Event Hub in realtime!

val query = new Query(" #Azure ")
query.setCount(100)
query.lang("en")

var finished = false
var maxStatusId = Long.MinValue
var preMaxStatusId = Long.MinValue
val innerLoop = new Breaks
while (!finished) {
  val result = twitter.search(query)
  val statuses = result.getTweets()
  var lowestStatusId = Long.MaxValue
  innerLoop.breakable {
    for (status <- statuses.asScala) {
      if (status.getId() <= preMaxStatusId) {
        preMaxStatusId = maxStatusId
        innerLoop.break
      }
      if(!status.isRetweet()) {
        sendEvent(gson.toJson(new MessageBody(status.getCreatedAt(), status.getFavoriteCount())))
      }
      lowestStatusId = Math.min(status.getId(), lowestStatusId)
      maxStatusId = Math.max(status.getId(), maxStatusId)
    }
  }

  if (lowestStatusId == Long.MaxValue) {
    preMaxStatusId = maxStatusId
  }
  Thread.sleep(10000)
  query.setMaxId(lowestStatusId - 1)
}

// Close connection to the Event Hub
eventHubClient.get().close()
pool.shutdown()
```

Presione **MAYÚS + ENTRAR** para ejecutar el cuaderno. Verá una salida similar al siguiente fragmento de código. Cada evento de la salida es una combinación de marca de tiempo y número de "Me gusta" ingeridos en Event Hubs.

```output
    Sent event: {"timestamp":"2019-04-24T09:39:40.000Z","favorite":0}

    Sent event: {"timestamp":"2019-04-24T09:38:48.000Z","favorite":1}

    Sent event: {"timestamp":"2019-04-24T09:38:36.000Z","favorite":0}

    Sent event: {"timestamp":"2019-04-24T09:37:27.000Z","favorite":0}

    Sent event: {"timestamp":"2019-04-24T09:37:00.000Z","favorite":2}

    Sent event: {"timestamp":"2019-04-24T09:31:11.000Z","favorite":0}

    Sent event: {"timestamp":"2019-04-24T09:30:15.000Z","favorite":0}

    Sent event: {"timestamp":"2019-04-24T09:30:02.000Z","favorite":1}

    ...
    ...
```

## <a name="read-tweets-from-event-hubs"></a>Lectura de tweets desde Event Hubs

En el cuaderno **AnalyzeTweetsFromEventHub**, pegue el código siguiente y reemplace el marcador de posición por los valores del recurso de Anomaly Detector que creó anteriormente. Este cuaderno lee los tweets que transmitió anteriormente a Event Hubs usando el cuaderno **SendTweetsToEventHub**.

En primer lugar, escriba un cliente para llamar a Anomaly Detector.
```scala

//
// Anomaly Detection Client
//

import java.io.{BufferedReader, DataOutputStream, InputStreamReader}
import java.net.URL
import java.sql.Timestamp

import com.google.gson.{Gson, GsonBuilder, JsonParser}
import javax.net.ssl.HttpsURLConnection

case class Point(var timestamp: Timestamp, var value: Double)
case class Series(var series: Array[Point], var maxAnomalyRatio: Double, var sensitivity: Int, var granularity: String)
case class AnomalySingleResponse(var isAnomaly: Boolean, var isPositiveAnomaly: Boolean, var isNegativeAnomaly: Boolean, var period: Int, var expectedValue: Double, var upperMargin: Double, var lowerMargin: Double, var suggestedWindow: Int)
case class AnomalyBatchResponse(var expectedValues: Array[Double], var upperMargins: Array[Double], var lowerMargins: Array[Double], var isAnomaly: Array[Boolean], var isPositiveAnomaly: Array[Boolean], var isNegativeAnomaly: Array[Boolean], var period: Int)

object AnomalyDetector extends Serializable {

  // Cognitive Services API connection settings
  val subscriptionKey = "[Placeholder: Your Anomaly Detector resource access key]"
  val endpoint = "[Placeholder: Your Anomaly Detector resource endpoint]"
  val latestPointDetectionPath = "/anomalydetector/v1.0/timeseries/last/detect"
  val batchDetectionPath = "/anomalydetector/v1.0/timeseries/entire/detect";
  val latestPointDetectionUrl = new URL(endpoint + latestPointDetectionPath)
  val batchDetectionUrl = new URL(endpoint + batchDetectionPath)
  val gson: Gson = new GsonBuilder().setDateFormat("yyyy-MM-dd'T'HH:mm:ss.SSS'Z'").setPrettyPrinting().create()

  def getConnection(path: URL): HttpsURLConnection = {
    val connection = path.openConnection().asInstanceOf[HttpsURLConnection]
    connection.setRequestMethod("POST")
    connection.setRequestProperty("Content-Type", "text/json")
    connection.setRequestProperty("Ocp-Apim-Subscription-Key", subscriptionKey)
    connection.setDoOutput(true)
    return connection
  }

  // Handles the call to Cognitive Services API.
  def processUsingApi(request: String, path: URL): String = {
    println(request)
    val encoded_text = request.getBytes("UTF-8")
    val connection = getConnection(path)
    val wr = new DataOutputStream(connection.getOutputStream())
    wr.write(encoded_text, 0, encoded_text.length)
    wr.flush()
    wr.close()

    val response = new StringBuilder()
    val in = new BufferedReader(new InputStreamReader(connection.getInputStream()))
    var line = in.readLine()
    while (line != null) {
      response.append(line)
      line = in.readLine()
    }
    in.close()
    return response.toString()
  }

  // Calls the Latest Point Detection API.
  def detectLatestPoint(series: Series): Option[AnomalySingleResponse] = {
    try {
      println("Process Timestamp: " + series.series.apply(series.series.length-1).timestamp.toString + ", size: " + series.series.length)
      val response = processUsingApi(gson.toJson(series), latestPointDetectionUrl)
      println(response)
      // Deserializing the JSON response from the API into Scala types
      val anomaly = gson.fromJson(response, classOf[AnomalySingleResponse])
      Thread.sleep(5000)
      return Some(anomaly)
    } catch {
      case e: Exception => {
        println(e)
        e.printStackTrace()
        return None
      }
    }
  }

  // Calls the Batch Detection API.
  def detectBatch(series: Series): Option[AnomalyBatchResponse] = {
    try {
      val response = processUsingApi(gson.toJson(series), batchDetectionUrl)
      println(response)
      // Deserializing the JSON response from the API into Scala types
      val anomaly = gson.fromJson(response, classOf[AnomalyBatchResponse])
      Thread.sleep(5000)
      return Some(anomaly)
    } catch {
      case e: Exception => {
        println(e)
        return None
      }
    }
  }
}
```

Presione **MAYÚS + ENTRAR** para ejecutar el cuaderno. Verá una salida similar al siguiente fragmento de código.

```scala
import java.io.{BufferedReader, DataOutputStream, InputStreamReader}
import java.net.URL
import java.sql.Timestamp
import com.google.gson.{Gson, GsonBuilder, JsonParser}
import javax.net.ssl.HttpsURLConnection
defined class Point
defined class Series
defined class AnomalySingleResponse
defined class AnomalyBatchResponse
defined object AnomalyDetector
```

Luego, prepare una función de agregación para usarla más adelante.
```scala
//
// User Defined Aggregation Function for Anomaly Detection
//

import org.apache.spark.sql.Row
import org.apache.spark.sql.expressions.{MutableAggregationBuffer, UserDefinedAggregateFunction}
import org.apache.spark.sql.types.{StructType, TimestampType, FloatType, MapType, BooleanType, DataType}
import scala.collection.immutable.ListMap

class AnomalyDetectorAggregationFunction extends UserDefinedAggregateFunction {
  override def inputSchema: StructType = new StructType().add("timestamp", TimestampType).add("value", FloatType)

  override def bufferSchema: StructType = new StructType().add("point", MapType(TimestampType, FloatType))

  override def dataType: DataType = BooleanType

  override def deterministic: Boolean = false

  override def initialize(buffer: MutableAggregationBuffer): Unit = {
    buffer(0) = Map()
  }

  override def update(buffer: MutableAggregationBuffer, input: Row): Unit = {
    buffer(0) = buffer.getAs[Map[java.sql.Timestamp, Float]](0) + (input.getTimestamp(0) -> input.getFloat(1))
  }

  override def merge(buffer1: MutableAggregationBuffer, buffer2: Row): Unit = {
    buffer1(0) = buffer1.getAs[Map[java.sql.Timestamp, Float]](0) ++ buffer2.getAs[Map[java.sql.Timestamp, Float]](0)
  }

  override def evaluate(buffer: Row): Any = {
    val points = buffer.getAs[Map[java.sql.Timestamp, Float]](0)
    if (points.size > 12) {
      val sorted_points = ListMap(points.toSeq.sortBy(_._1.getTime):_*)
      var detect_points: List[Point] = List()
      sorted_points.keys.foreach {
        key => detect_points = detect_points :+ new Point(key, sorted_points(key))
      }


      // 0.25 is maxAnomalyRatio. It represents 25%, max anomaly ratio in a time series.
      // 95 is the sensitivity of the algorithms.
      // Check Anomaly detector API reference (https://aka.ms/anomaly-detector-rest-api-ref)

      val series: Series = new Series(detect_points.toArray, 0.25, 95, "hourly")
      val response: Option[AnomalySingleResponse] = AnomalyDetector.detectLatestPoint(series)
      if (!response.isEmpty) {
        return response.get.isAnomaly
      }
    }

    return None
  }
}

```

Presione **MAYÚS + ENTRAR** para ejecutar el cuaderno. Verá una salida similar al siguiente fragmento de código.

```scala
import org.apache.spark.sql.Row
import org.apache.spark.sql.expressions.{MutableAggregationBuffer, UserDefinedAggregateFunction}
import org.apache.spark.sql.types.{StructType, TimestampType, FloatType, MapType, BooleanType, DataType}
import scala.collection.immutable.ListMap
defined class AnomalyDetectorAggregationFunction
```

Después, cargue del centro de eventos los datos de la detección de anomalías. Reemplace el marcador de posición por los valores de Azure Event Hubs que creó antes.

```scala
//
// Load Data from Eventhub
//

import org.apache.spark.eventhubs._
import org.apache.spark.sql.types._
import org.apache.spark.sql.functions._

val connectionString = ConnectionStringBuilder("[Placeholder: EventHub namespace connection string]")
  .setEventHubName("[Placeholder: EventHub name]")
  .build

val customEventhubParameters =
  EventHubsConf(connectionString)
  .setConsumerGroup("$Default")
  .setMaxEventsPerTrigger(100)

val incomingStream = spark.readStream.format("eventhubs").options(customEventhubParameters.toMap).load()

val messages =
  incomingStream
  .withColumn("enqueuedTime", $"enqueuedTime".cast(TimestampType))
  .withColumn("body", $"body".cast(StringType))
  .select("enqueuedTime", "body")

val bodySchema = new StructType().add("timestamp", TimestampType).add("favorite", IntegerType)

val msgStream = messages.select(from_json('body, bodySchema) as 'fields).select("fields.*")

msgStream.printSchema

display(msgStream)

```

Ahora, la salida será similar a la de la siguiente imagen. Tenga en cuenta que la fecha de la tabla puede no coincidir con la de este tutorial, ya que los datos son en tiempo real.
![Cargar datos de Event Hubs](../media/tutorials/load-data-from-eventhub.png "Carga de datos de Event Hubs")

Ahora ha transmitido datos de Azure Event Hubs a Azure Databricks casi en tiempo real mediante el conector de Event Hubs para Apache Spark. Para más información sobre cómo usar el conector de Event Hubs para Spark, consulte la [documentación del conector](https://github.com/Azure/azure-event-hubs-spark/tree/master/docs).



## <a name="run-anomaly-detection-on-tweets"></a>Ejecución de la detección de anomalías en tweets

En esta sección, se ejecuta la detección de anomalías en los tweets recibidos mediante API Anomaly Detector. En esta sección, agregará los fragmentos de código al mismo cuaderno **AnalyzeTweetsFromEventHub**.

Para realizar la detección de anomalías, en primer lugar, es preciso agregar el número de métricas por hora.
```scala
//
// Aggregate Metric Count by Hour
//

// If you want to change granularity, change the groupBy window.
val groupStream = msgStream.groupBy(window($"timestamp", "1 hour"))
  .agg(avg("favorite").alias("average"))
  .withColumn("groupTime", $"window.start")
  .select("groupTime", "average")

groupStream.printSchema

display(groupStream)
```
Ahora, la salida será similar a los siguientes fragmentos de código.
```
groupTime                       average
2019-04-23T04:00:00.000+0000    24
2019-04-26T19:00:00.000+0000    47.888888888888886
2019-04-25T12:00:00.000+0000    32.25
2019-04-26T09:00:00.000+0000    63.4
...
...

```

Luego, lleve el resultado de la salida agregada a Delta. Como la detección de anomalías requiere una ventana de historial mayor, se va a usar Delta para mantener los datos del historial del punto en que desea realizar la detección.
Reemplace "[Placeholder: table name]" por un nombre de tabla de Delta completo que debe crear (por ejemplo, "tweets"). Reemplace "[Placeholder: folder name for checkpoints]" por un valor de cadena que es único cada vez que se ejecuta este código (por ejemplo, "etl-from-eventhub-20190605").
Para más información acerca de Delta Lake en Azure Databricks, consulte la [guía de Delta Lake](https://docs.azuredatabricks.net/delta/index.html)


```scala
//
// Output Aggregation Result to Delta
//

groupStream.writeStream
  .format("delta")
  .outputMode("complete")
  .option("checkpointLocation", "/delta/[Placeholder: table name]/_checkpoints/[Placeholder: folder name for checkpoints]")
  .table("[Placeholder: table name]")

```

Reemplace "[Placeholder: table name]" por el nombre de tabla Delta que seleccionó antes.
```scala
//
// Show Aggregate Result
//

val twitterCount = spark.sql("SELECT COUNT(*) FROM [Placeholder: table name]")
twitterCount.show()

val twitterData = spark.sql("SELECT * FROM [Placeholder: table name] ORDER BY groupTime")
twitterData.show(200, false)

display(twitterData)
```
La salida es similar a la siguiente:
```
groupTime                       average
2019-04-08T01:00:00.000+0000    25.6
2019-04-08T02:00:00.000+0000    6857
2019-04-08T03:00:00.000+0000    71
2019-04-08T04:00:00.000+0000    55.111111111111114
2019-04-08T05:00:00.000+0000    2203.8
...
...

```

Ahora Delta ingiere continuamente los datos de las series temporales agregados. Luego, puede programar un trabajo por horas para detectar la anomalía del punto más reciente.
Reemplace "[Placeholder: table name]" por el nombre de tabla Delta que seleccionó antes.

```scala
//
// Anomaly Detection
//

import java.time.Instant
import java.time.format.DateTimeFormatter
import java.time.ZoneOffset
import java.time.temporal.ChronoUnit

val detectData = spark.read.format("delta").table("[Placeholder: table name]")

// You could use Databricks to schedule an hourly job and always monitor the latest data point
// Or you could specify a const value here for testing purpose
// For example, val endTime = Instant.parse("2019-04-16T00:00:00Z")
val endTime = Instant.now()

// This is when your input of anomaly detection starts. It is hourly time series in this tutorial, so 72 means 72 hours ago from endTime.
val batchSize = 72
val startTime = endTime.minus(batchSize, ChronoUnit.HOURS)

val DATE_TIME_FORMATTER = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss").withZone(ZoneOffset.UTC);

val series = detectData.filter($"groupTime" <= DATE_TIME_FORMATTER.format(endTime))
  .filter($"groupTime" > DATE_TIME_FORMATTER.format(startTime))
  .sort($"groupTime")

series.createOrReplaceTempView("series")

//series.show()

// Register the function to access it
spark.udf.register("anomalydetect", new AnomalyDetectorAggregationFunction)

val adResult = spark.sql("SELECT '" + endTime.toString + "' as datetime, anomalydetect(groupTime, average) as anomaly FROM series")
adResult.show()
```
El resultado es similar a este:

```
+--------------------+-------+
|           timestamp|anomaly|
+--------------------+-------+
|2019-04-16T00:00:00Z|  false|
+--------------------+-------+
```

Eso es todo. Mediante Azure Databricks ha transmitido correctamente datos en secuencias a Azure Event Hubs, ha consumido los datos de los flujos mediante el conector de Event Hubs y, después, ha ejecutado una detección de anomalías casi en tiempo real.
Aunque en este tutorial, la granularidad es horaria, la puede cambiar para ajustarla a sus necesidades.

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando haya terminado de ejecutar el tutorial, puede finalizar el clúster. Para ello, en el área de trabajo de Azure Databricks, seleccione **Clusters** (Clústeres) en el panel izquierdo. En el clúster que desea finalizar, mueva el cursor sobre el botón de puntos suspensivos en la columna **Actions** (Acciones), seleccione el icono **Terminate** (Finalizar) y, luego. seleccione **Confirm** (Confirmar).

![Detención de un clúster de Databricks](../media/tutorials/terminate-databricks-cluster.png "Detención de un clúster de Databricks")

Si no finaliza manualmente el clúster, este se detendrá automáticamente si seleccionó la casilla **Terminate after \_\_ minutes of inactivity** (Finalizar después de \_\_ minutos de inactividad) al crear el clúster. En tal caso, el clúster se detendrá automáticamente si ha estado inactivo durante el tiempo especificado.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, aprendió a utilizar Azure Databricks para transmitir datos a Azure Event Hubs y, después, leyó los datos transmitidos desde Event Hubs en tiempo real. Pase al siguiente tutorial, donde aprenderá a llamar a API Anomaly Detector y a visualizar anomalías mediante Power BI Desktop.

> [!div class="nextstepaction"]
>[Detección de anomalías por lotes con Power BI Desktop](batch-anomaly-detection-powerbi.md)
