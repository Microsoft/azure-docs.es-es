---
title: 'Tutorial: Detección de anomalías en datos transmitidos con Azure Databricks'
description: Use la API del Detector de anomalías y Azure Databricks para supervisar anomalías en los datos.
titlesuffix: Azure Cognitive Services
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: anomaly-detector
ms.topic: article
ms.date: 05/08/2019
ms.author: aahi
ms.openlocfilehash: a00ad2523c215fa54d7d19d8c9e923b621f3081a
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65791857"
---
# <a name="tutorial-anomaly-detection-on-streaming-data-using-azure-databricks"></a>Tutorial: Detección de anomalías en datos transmitidos con Azure Databricks

Microsoft Power BI Desktop es una aplicación gratuita que le permite conectarse a los datos, transformarlos y visualizarlos. La API del Detector de anomalías, parte de Azure Cognitive Services, proporciona una manera de supervisar los datos de series temporales. Use este tutorial para ejecutar la detección de anomalías en un flujo de datos casi en tiempo real con Azure Databricks. Deberá introducir datos de twitter con Azure Event Hubs e importarlas en Azure Databricks mediante el conector de Event Hubs para Spark. Después, usará la API para detectar anomalías en los datos transmitidos. 

La siguiente ilustración muestra el flujo de la aplicación:

![Azure Databricks con Event Hubs y Cognitive Services](../media/tutorials/databricks-cognitive-services-tutorial.png "Azure Databricks with Event Hubs and Cognitive Services")

En este tutorial se describen las tareas siguientes:

> [!div class="checklist"]
> * Creación de un área de trabajo de Azure Databricks
> * Creación de un clúster de Spark en Azure Databricks
> * Creación de una aplicación de Twitter para acceder a datos de streaming
> * Creación de cuadernos en Azure Databricks
> * Asociación de bibliotecas a Event Hubs y la API de Twitter
> * Crear un recurso del Detector de anomalías y recuperar la clave de acceso
> * Envío de tweets a Event Hubs
> * Lectura de tweets desde Event Hubs
> * Ejecutar la detección de anomalías en tweets

> [!Note]
> Este tutorial presenta un enfoque para implementar la recomendada [arquitectura de la solución](https://azure.microsoft.com/solutions/architecture/anomaly-detector-process/) para la API del Detector de anomalías.

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

> [!Note]
> No se puede completar este tutorial con una clave de prueba gratuita para la API del Detector de anomalías. Para usar una cuenta gratuita para crear el clúster de Azure Databricks, antes de crear el clúster, vaya a su perfil y cambie su suscripción a **pago por uso**. Para más información consulte el sitio de [cuentas gratuitas de Azure](https://azure.microsoft.com/free/).

## <a name="prerequisites"></a>Requisitos previos

- Un [espacio de nombres de Azure Event Hubs](https://docs.microsoft.com/azure/event-hubs/event-hubs-create) y centro de eventos.

- El [cadena de conexión](../../../event-hubs/event-hubs-get-connection-string.md) para tener acceso al espacio de nombres de Event Hubs. La cadena de conexión debe tener un formato similar al:

    `Endpoint=sb://<namespace>.servicebus.windows.net/;SharedAccessKeyName=<key name>;SharedAccessKey=<key value>`  

- El nombre de la directiva de acceso compartido y la directiva de clave para Event Hubs.

Vea Azure Event Hubs [quickstart](../../../event-hubs/event-hubs-create.md) para obtener información acerca de cómo crear un espacio de nombres y centro de eventos.

## <a name="create-an-azure-databricks-workspace"></a>Creación de un área de trabajo de Azure Databricks

En esta sección, creará un área de trabajo de Azure Databricks mediante el [portal Azure](https://portal.azure.com/).

1. En Azure Portal, seleccione **Crear un recurso** > **Análisis** > **Azure Databricks**.

    ![Databricks en Azure Portal](../media/tutorials/azure-databricks-on-portal.png "Databricks en Azure Portal")

3. En **Azure Databricks Service**, proporcione los valores siguientes para crear un área de trabajo de Databricks.


    |Propiedad  |Descripción  |
    |---------|---------|
    |**Workspace name** (Nombre del área de trabajo)     | Proporcione un nombre para el área de trabajo de Databricks        |
    |**Suscripción**     | En el cuadro desplegable, seleccione la suscripción de Azure.        |
    |**Grupos de recursos**     | Especifique si desea crear un nuevo grupo de recursos o utilizar uno existente. Un grupo de recursos es un contenedor que almacena los recursos relacionados con una solución de Azure. Para más información, consulte [Información general del grupo de recursos de Azure](../../../azure-resource-manager/resource-group-overview.md). |
    |**Ubicación**     | Seleccione **East US 2** o una de las otras regiones disponibles. Consulte [productos disponibles por región](https://azure.microsoft.com/regions/services/) para disponibilidad por región.        |
    |**Plan de tarifa**     |  Elija entre **Standard** o **Premium**. No elija **prueba**. Para más información sobre estos planes, consulte la [página de precios de Databricks](https://azure.microsoft.com/pricing/details/databricks/).       |

    Seleccione **Crear**.

4. La cuenta tarda unos minutos en crearse. 

## <a name="create-a-spark-cluster-in-databricks"></a>Creación de un clúster de Spark en Databricks

1. En Azure Portal, vaya al área de trabajo de Databricks que ha creado y, después, seleccione **Launch Workspace** (Iniciar área de trabajo).

2. Se le redirigirá al portal de Azure Databricks. En el portal, seleccione **nuevo clúster**.

    ![Databricks en Azure](../media/tutorials/databricks-on-azure.png "Databricks en Azure")

3. En el **nuevo clúster** , proporcione los valores para crear un clúster.

    ![Creación de un clúster de Databricks Spark en Azure](../media/tutorials/create-databricks-spark-cluster.png "Creación de un clúster de Databricks Spark en Azure")

    Acepte los demás valores predeterminados, salvo los siguientes:

   * Escriba un nombre para el clúster.
   * En este artículo, cree un clúster con **5.2** en tiempo de ejecución. No seleccione **5.3** en tiempo de ejecución.
   * Asegúrese de que el **finalizan \_ \_ minutos de inactividad** está activada la casilla. Proporcionar una duración (en minutos) para terminar el clúster, si no se usa el clúster.

     Seleccione **Create cluster** (Crear clúster). Una vez que el clúster se está ejecutando, puede asociarle notebooks y ejecutar trabajos de Spark.

## <a name="create-a-twitter-application"></a>Crear una aplicación de Twitter

Para recibir un flujo de tweets, debe crear una aplicación en Twitter. Siga los pasos para crear una aplicación de Twitter y anote los valores que necesita para finalizar este tutorial.

1. Desde un explorador web, vaya a [Twitter Application Management](https://apps.twitter.com/) (Administración de aplicaciones de Twitter) y seleccione **Create New App** (Crear nueva aplicación).

    ![Crear aplicación de Twitter](../media/tutorials/databricks-create-twitter-app.png "Create Twitter application")

2. En la página **Create an application** (Crear una aplicación), proporcione los detalles de la nueva aplicación y, a continuación, seleccione **Create your Twitter application** (Crear su aplicación de Twitter).

    ![Detalles de la aplicación de Twitter](../media/tutorials/databricks-provide-twitter-app-details.png "Twitter application details")

3. En la página de la aplicación, seleccione la pestaña **Claves y tokens de acceso** y copie los valores de **Consumer Key** (Clave de consumidor) y **Consumer Secret** (Secreto de consumidor). Además, seleccione **Create my access token** (Crear mi token de acceso) para generar los tokens de acceso. Copie los valores de **Access Token** (Token de acceso) y **Access Token Secret** (Secreto del token de acceso).

    ![Detalles de la aplicación de Twitter](../media/tutorials/twitter-app-key-secret.png "Twitter application details")

Guarde los valores que ha recuperado de la aplicación de Twitter. Necesitará estos valores más adelante en el tutorial.

## <a name="attach-libraries-to-spark-cluster"></a>Asociación de bibliotecas a un clúster de Spark

En este tutorial, se utilizan las API de Twitter para enviar tweets a Event Hubs. Se utiliza también el [conector de Event Hubs para Apache Spark](https://github.com/Azure/azure-event-hubs-spark) para leer y escribir datos en Azure Event Hubs. Para usar estas API como parte del clúster, puede agregarlas como bibliotecas a Azure Databricks y, a continuación, asociarlas al clúster de Spark. Las instrucciones siguientes muestran cómo agregar las bibliotecas para el **Shared** carpeta en el área de trabajo.

1. En el área de trabajo de Azure Databricks, seleccione **Workspace** (Área de trabajo) y, a continuación, haga clic con el botón derecho en **Shared** (Compartidos). En el menú contextual, seleccione **Create** > **Library** (Crear > Biblioteca).

   ![Cuadro de diálogo Agregar biblioteca](../media/tutorials/databricks-add-library-option.png "Add library dialog box")

2. En la página New Library (Nueva biblioteca), en **Source** (Origen), seleccione **Maven Coordinate** (Coordenada de Maven). En **Coordinate** (Coordenada), escriba la coordenada del paquete que desea agregar. Estas son las coordenadas de Maven para las bibliotecas que se utilizan en este tutorial:

   * Conector de Event Hubs para Spark: `com.microsoft.azure:azure-eventhubs-spark_2.11:2.3.10`
   * API de Twitter: `org.twitter4j:twitter4j-core:4.0.7`

     ![Proporcionar coordenadas Maven](../media/tutorials/databricks-eventhub-specify-maven-coordinate.png "Provide Maven coordinates")

3. Seleccione **Crear**.

4. Seleccione la carpeta en la que ha agregado la biblioteca y, a continuación, seleccione el nombre de la biblioteca.

    ![Seleccionar biblioteca para agregar](../media/tutorials/select-library.png "Select library to add")

5. Si no hay ningún clúster en la página de la biblioteca, seleccione **clústeres** y ejecute el clúster que ha creado. Espere hasta que el estado muestra "Running" y, a continuación, vuelva a la página de la biblioteca.
En la página de la biblioteca, seleccione el clúster donde desea usar la biblioteca y, a continuación, seleccione **instalar**. Una vez que la biblioteca está correctamente asociada con el clúster, el estado cambia inmediatamente a **instalado**.

    ![Instalar la biblioteca al clúster](../media/tutorials/databricks-library-attached.png "biblioteca de instalación en clúster")

6. Repita estos pasos para el paquete de Twitter, `twitter4j-core:4.0.7`.

## <a name="get-a-cognitive-services-access-key"></a>Obtención de una clave de acceso de Cognitive Services

En este tutorial, usará el [API de Azure Cognitive Services anomalías Detector](../overview.md) para ejecutar la detección de anomalías en un flujo de tweets casi en tiempo real. Antes de usar las API, debe crear un recurso del Detector de anomalías en Azure y recuperar una clave de acceso para usar las API del Detector de anomalías.

1. Inicie sesión en el [Azure Portal](https://portal.azure.com/).

2. Seleccione **+ Crear un recurso**.

3. En Azure Marketplace, seleccione **AI + aprendizaje automático** > **ver todo** > **Cognitive Services - más**  >  **Detector de anomalías**. O bien podría usar [este vínculo](https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesAnomalyDetector) para ir a la **crear** directamente el cuadro de diálogo.

    ![Crear recurso de Detector de anomalías](../media/tutorials/databricks-cognitive-services-anomaly-detector.png "recursos crear Detector de anomalías")

4. En el cuadro de diálogo **Create** (Crear), proporcione los siguientes valores:

    |Valor |DESCRIPCIÓN  |
    |---------|---------|
    |NOMBRE     | Un nombre para el recurso del Detector de anomalías.        |
    |Subscription     | Se asociará la suscripción de Azure del recurso.        |
    |Ubicación     | Una ubicación de Azure.        |
    |Plan de tarifa     | Un plan de tarifa para el servicio. Para obtener más información sobre los precios de Detector de anomalías, vea [página de precios](https://azure.microsoft.com/pricing/details/cognitive-services/anomaly-detector/).        |
    |Grupos de recursos     | Especifique si desea crear un nuevo grupo de recursos o seleccionar uno existente.        |


     Seleccione **Crear**.

5. Una vez creado el recurso, desde el **Introducción** ficha, seleccione **Mostrar teclas de acceso**.

    ![Mostrar claves de acceso](../media/tutorials/cognitive-services-get-access-keys.png "Show access keys")

    Copie también una parte de la dirección URL del punto de conexión, tal y como se muestra en la captura de pantalla. Necesita esta dirección URL en el tutorial.

6. En **claves**, seleccione el icono de copiar en relación con la clave que desea usar.

    ![Copiar las claves de acceso](../media/tutorials/cognitive-services-copy-access-keys.png "Copy access keys")

7. Guarde los valores de la dirección URL del punto de conexión y la clave de acceso que recuperó en este paso. Los necesitará más adelante en este tutorial.

## <a name="create-notebooks-in-databricks"></a>Creación de cuadernos en Databricks

En esta sección, se crearán dos cuadernos en el área de trabajo de Databricks con los nombres siguientes:

- **SendTweetsToEventHub**: un cuaderno de productor que se usa para obtener tweets de Twitter y transmitirlos a Event Hubs.
- **AnalyzeTweetsFromEventHub** -un cuaderno de consumidor utiliza para leer los tweets de Event Hubs y ejecutar la detección de anomalías.

1. En el panel izquierdo, seleccione **Workspace** (Área de trabajo). En la lista desplegable **Workspace** (Área de trabajo), seleccione **Create** (Crear) y, a continuación, haga clic en **Notebook** (Cuaderno).

    ![Creación de notebooks en Databricks](../media/tutorials/databricks-create-notebook.png "Creación de notebooks en Databricks")

2. En el **crear Bloc de notas** diálogo cuadro, escriba **SendTweetsToEventHub** como nombre, seleccione **Scala** como el idioma y seleccione el clúster de Spark que creó anteriormente.

    ![Creación de notebooks en Databricks](../media/tutorials/databricks-notebook-details.png "Creación de notebooks en Databricks")

    Seleccione **Crear**.

3. Repita los pasos para crear el cuaderno **AnalyzeTweetsFromEventHub**.

## <a name="send-tweets-to-event-hubs"></a>Envío de tweets a Event Hubs

En el cuaderno **SendTweetsToEventHub**, pegue el código siguiente y reemplace el marcador de posición por los valores del espacio de nombres de Event Hubs y la aplicación de Twitter que creó anteriormente. Este cuaderno transmite los tweets con la palabra clave "Azure" a Event Hubs en tiempo real.

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

Presione **MAYÚS + ENTRAR** para ejecutar el cuaderno. Verá una salida similar al siguiente fragmento de código. Cada evento de la salida es un tweet que se ingiere en Event Hubs.

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

## <a name="read-tweets-from-event-hubs"></a>Lectura de tweets desde Event Hubs

En el cuaderno **AnalyzeTweetsFromEventHub**, pegue el código siguiente y reemplace los marcadores de posición por los valores de la instancia de Azure Event Hubs que creó anteriormente. Este cuaderno lee los tweets que transmitió anteriormente a Event Hubs usando el cuaderno **SendTweetsToEventHub**.

En primer lugar, escribir un cliente para llamar al detector de anomalías. 
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

  // Calls the Latest Point Detection API for timeserie.
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

  // Calls the Batch Detection API for timeserie.
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

Presione **MAYÚS + ENTRAR** para ejecutar el cuaderno. Verá una salida similar al siguiente fragmento de código. :

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

A continuación, prepare una función de agregación en el futuro.
```scala
//
// User Defined Aggregation Function for Anomaly Detection
//

import org.apache.spark.sql.Row
import org.apache.spark.sql.expressions.{MutableAggregationBuffer, UserDefinedAggregateFunction}
import org.apache.spark.sql.types.{StructType, TimestampType, FloatType, MapType, BooleanType, DataType}
//import org.apache.spark.sql.functions._
import scala.collection.immutable.ListMap

class AnomalyDetectorAggregationFunction_Hourly extends UserDefinedAggregateFunction {
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
      // Check Anomaly detector API reference (https://westus2.dev.cognitive.microsoft.com/docs/services/AnomalyDetector/operations/post-timeseries-last-detect)
      
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

    import org.apache.spark.sql.Row
    import org.apache.spark.sql.expressions.{MutableAggregationBuffer, UserDefinedAggregateFunction}
    import org.apache.spark.sql.types.{StructType, TimestampType, FloatType, MapType, BooleanType, DataType}
    import scala.collection.immutable.ListMap
    defined class AnomalyDetectorAggregationFunction

A continuación, cargar datos desde el centro de eventos para la detección de anomalías.

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

Ahora, la salida es similar a la siguiente imagen. Preste atención a la fecha de la tabla pueden diferir de la fecha en este tutorial que los datos en tiempo real.
![Centro de datos de eventos de carga](../media/tutorials/load-data-from-eventhub.png "carga datos desde centro de eventos")

Ahora ha transmitido datos de Azure Event Hubs a Azure Databricks casi en tiempo real mediante el conector de Event Hubs para Apache Spark. Para más información sobre cómo usar el conector de Event Hubs para Spark, consulte la [documentación del conector](https://github.com/Azure/azure-event-hubs-spark/tree/master/docs).



## <a name="run-anomaly-detection-on-tweets"></a>Ejecutar la detección de anomalías en tweets

En esta sección, ejecutará la detección de anomalías en los tweets recibidos mediante la API del detector de anomalías. En esta sección, agregará los fragmentos de código al mismo cuaderno **AnalyzeTweetsFromEventHub**.

Para realizar la detección de anomalías, en primer lugar, debe agregar el número de métricas por hora.
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
Ahora, la salida es similar a los fragmentos de código siguientes.
```
groupTime                       average
2019-04-23T04:00:00.000+0000    24
2019-04-26T19:00:00.000+0000    47.888888888888886
2019-04-25T12:00:00.000+0000    32.25
2019-04-26T09:00:00.000+0000    63.4
...
...

```

A continuación, obtener el resultado de salida agregados a Delta. Dado que la detección de anomalías requiere una ventana de historial más largo, vamos a usar Delta para mantener los datos del historial para el punto que desea detectar. 


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
La salida como sigue: 
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

Ahora los datos de series de tiempo agregados continuamente se ingieren en el Delta. A continuación, puede programar un trabajo cada hora para detectar las anomalías de punto más reciente. 

```scala
//
// Anomaly Detection with Batch query
//

import java.time.Instant
import java.time.temporal.ChronoUnit

val detectData = spark.read.format("delta").table("[Placeholder: table name]")

// How long history you want to use in anomaly detection. It is hourly time series in this tutorial, so 72 means 72 hours. 
val batchSize = 72

// Change the endTime to where you want to detect. You could use Databricks to schedule a job and change it to the latest hour. 
val endTime = Instant.parse("2019-04-16T00:00:00Z")
val startTime = endTime.minus(batchSize, ChronoUnit.HOURS)

val series = detectData.filter($"groupTime" < endTime.toString && $"groupTime" >= startTime.toString).sort($"groupTime")

series.createOrReplaceTempView("series")

//series.show()

// Register the function to access it
spark.udf.register("anomalydetect", new AnomalyDetectorAggregationFunction)

val adResult = spark.sql("SELECT '" + endTime.toString + "' as timestamp, anomalydetect(groupTime, average) as anomaly FROM series")
adResult.show()
```
Como resultado como sigue: 

```
+--------------------+-------+
|           timestamp|anomaly|
+--------------------+-------+
|2019-04-16T00:00:00Z|  false|
+--------------------+-------+

```
El resultado de la detección de anomalías a las diferencias de salida. 
```scala
//
// Output Batch AD Result to delta
//

adResult.writeStream
  .format("delta")
  .outputMode("complete")
  .option("checkpointLocation", "/delta/[Placeholder: table name]/_checkpoints/[Placeholder: folder name for checkpoints]")
  .table("[Placeholder: table name]")
  
```


Eso es todo. Con Azure Databricks, ha transmitido correctamente datos a Azure Event Hubs, consumen los flujos de datos mediante el conector de Event Hubs y, a continuación, se ejecutó la detección de anomalías en la transmisión de datos en tiempo real.
Aunque en este tutorial, la granularidad es cada hora, siempre puede cambiar la granularidad para satisfacer sus necesidades. 

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando haya terminado de ejecutar el tutorial, puede finalizar el clúster. Para ello, desde el área de trabajo de Azure Databricks, en el panel izquierdo, seleccione **Clusters** (Clústeres). En el clúster que desea finalizar, mueva el cursor sobre el botón de puntos suspensivos en la columna **Actions** (Acciones) y seleccione el icono **Terminate** (Finalizar).

![Detener un clúster de Databricks](../media/tutorials/terminate-databricks-cluster.png "Stop a Databricks cluster")

Si no finaliza manualmente el clúster se detendrá automáticamente, proporciona seleccionó el **finalizan \_ \_ minutos de inactividad** casilla de verificación al crear el clúster. En tal caso, el clúster se detendrá automáticamente si ha estado inactivo durante el tiempo especificado.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, aprendió a utilizar Azure Databricks para transmitir datos a Azure Event Hubs y, después, leyó los datos transmitidos desde Event Hubs en tiempo real. En el tutorial siguiente para obtener información sobre cómo llamar a la API del Detector de anomalías y visualizar las anomalías con Power BI desktop. 

> [!div class="nextstepaction"]
>[Detección de anomalías de batch con Power BI desktop](batch-anomaly-detection-powerbi.md)
