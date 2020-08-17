---
title: 'Azure Event Hubs: Procesamiento de eventos de Apache Kafka'
description: 'Tutorial: En este artículo se muestra cómo procesar los eventos de Kafka que se introducen a través de Event Hubs mediante Azure Stream Analytics'
ms.topic: tutorial
ms.date: 06/23/2020
ms.openlocfilehash: 6dfca5b476dff1ee3367b5365c15869c8cff819b
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87074910"
---
# <a name="tutorial-process-apache-kafka-for-event-hubs-events-using-stream-analytics"></a>Tutorial: Procesamiento de eventos de Apache Kafka para Event Hubs mediante Stream Analytics 
En este artículo se muestra cómo transmitir en secuencias datos a Event Hubs y cómo procesarlos con Azure Stream Analytics. Estos pasos son los siguientes: 

1. Cree un espacio de nombres de Event Hubs.
2. Cree un cliente de Kafka que envíe mensajes al centro de eventos.
3. Cree un trabajo de Stream Analytics que copie datos del centro de eventos en Azure Blob Storage. 

No es necesario cambiar a los clientes de protocolo ni ejecutar sus propios clústeres al usar el punto de conexión de Kafka expuesto por un centro de eventos. Azure Event Hubs admite [Apache Kafka versión 1.0.](https://kafka.apache.org/10/documentation.html) y posteriores. 


## <a name="prerequisites"></a>Prerrequisitos

Para completar esta guía de inicio rápido, asegúrese de cumplir los siguientes requisitos previos:

* Suscripción a Azure. Si no tiene una, cree una [cuenta gratuita](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio) antes de empezar.
* [Kit de desarrollo de Java (JDK) 1.7+](https://aka.ms/azure-jdks).
* [Descargue](https://maven.apache.org/download.cgi) e [instale](https://maven.apache.org/install.html) un archivo binario de Maven.
* [Git](https://www.git-scm.com/)
* Una **cuenta de Azure Storage**. Si no tiene, [cree una](../storage/common/storage-account-create.md) antes de continuar. El trabajo de Stream Analytics en este tutorial almacena los datos de salida en Azure Blob Storage. 


## <a name="create-an-event-hubs-namespace"></a>Creación de un espacio de nombres de Event Hubs
Al crear un espacio de nombres en el nivel **estándar** de Event Hubs, se habilita automáticamente el punto de conexión de Kafka para el espacio de nombres. Puede transmitir eventos desde las aplicaciones que usan el protocolo de Kafka a instancias del nivel estándar de Event Hubs. Siga las instrucciones paso a paso de [Creación de un centro de eventos mediante Azure Portal](event-hubs-create.md) para crear un espacio de nombres de Event Hubs de nivel **estándar**. 

> [!NOTE]
> Event Hubs para Kafka solo está disponible en los niveles **estándar** y **dedicado**. El nivel **básico** no es compatible con Kafka en Event Hubs.

## <a name="send-messages-with-kafka-in-event-hubs"></a>Envío de mensajes con Kafka en Event Hubs

1. Clone el [repositorio de Azure Event Hubs para Kafka](https://github.com/Azure/azure-event-hubs-for-kafka) en la máquina.
2. Vaya a la carpeta `azure-event-hubs-for-kafka/quickstart/java/producer`. 
4. Actualice los detalles de configuración para el productor en `src/main/resources/producer.config`. Especifique el **nombre** y **cadena de conexión** para el **espacio de nombres del centro de eventos**. 

    ```xml
    bootstrap.servers={EVENT HUB NAMESPACE}.servicebus.windows.net:9093
    security.protocol=SASL_SSL
    sasl.mechanism=PLAIN
    sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required username="$ConnectionString" password="{CONNECTION STRING for EVENT HUB NAMESPACE}";
    ```

5. Vaya a `azure-event-hubs-for-kafka/quickstart/java/producer/src/main/java/` y abra el archivo **TestDataReporter.java** en el editor que prefiera. 
6. Comente las siguientes líneas de código:

    ```java
                //final ProducerRecord<Long, String> record = new ProducerRecord<Long, String>(TOPIC, time, "Test Data " + i);
    ```
3. Agregue la siguiente línea de código en lugar del código con comentarios: 

    ```java
                final ProducerRecord<Long, String> record = new ProducerRecord<Long, String>(TOPIC, time, "{ \"eventData\": \"Test Data " + i + "\" }");            
    ```

    Este código envía los datos del evento en formato **JSON**. Al configurar la entrada para un trabajo de Stream Analytics, especifique JSON como formato de los datos de entrada. 
7. **Ejecute el productor** y transmítalo a Event Hubs. En un equipo Windows, al usar un **símbolo del sistema Node.js**, cambie a la carpeta `azure-event-hubs-for-kafka/quickstart/java/producer` antes de ejecutar estos comandos. 
   
    ```shell
    mvn clean package
    mvn exec:java -Dexec.mainClass="TestProducer"                                    
    ```

## <a name="verify-that-event-hub-receives-the-data"></a>Compruebe que ese centro de eventos recibe los datos.

1. Seleccione **Event Hubs** en **ENTIDADES**. Confirme que ve un centro de eventos denominado **prueba**. 

    ![Centro de eventos: prueba](./media/event-hubs-kafka-stream-analytics/test-event-hub.png)
2. Confirme que ve los mensajes que entran en el centro de eventos. 

    ![Centro de eventos: mensajes](./media/event-hubs-kafka-stream-analytics/confirm-event-hub-messages.png)

## <a name="process-event-data-using-a-stream-analytics-job"></a>Procesamiento de los datos de eventos mediante un trabajo de Stream Analytics
En esta sección, vamos a crear un trabajo de Azure Stream Analytics. El cliente de Kafka envía eventos al centro de eventos. Cree un trabajo de Stream Analytics que tome datos de eventos como entrada y los envíe a Azure Blob Storage. Si no tiene una **cuenta de Azure Storage**, debe [crear una](../storage/common/storage-account-create.md).

La consulta en el trabajo de Stream Analytics se pasa a través de los datos sin realizar ningún análisis. Puede crear una consulta que transforme los datos de entrada para generar datos de salida en un formato diferente o con información obtenida.  

### <a name="create-a-stream-analytics-job"></a>Creación de un trabajo de Stream Analytics 

1. Seleccione **+ Crear un recurso** en [Azure Portal](https://portal.azure.com).
2. Seleccione **Analytics** en el menú **Azure Marketplace** y, luego, seleccione **Stream Analytics job** (Trabajo de Stream Analytics). 
3. En la página **Nuevo Stream Analytics**, haga lo siguiente: 
    1. Escriba un **nombre** para el trabajo. 
    2. Seleccione su **suscripción**.
    3. Seleccione **Crear nuevo** para el **grupo de recursos** y escriba el nombre. También puede **usar un grupo de recursos existente**. 
    4. Seleccione una **ubicación** para el trabajo.
    5. Seleccione **Crear** para crear el trabajo. 

        ![Nuevo trabajo de Stream Analytics](./media/event-hubs-kafka-stream-analytics/new-stream-analytics-job.png)

### <a name="configure-job-input"></a>Configuración de la entrada del trabajo

1. En el mensaje de notificación, seleccione **Ir al recurso** para ver la página **Trabajo de Stream Analytics**. 
2. Seleccione **Entradas** en la sección **TOPOLOGÍA DE TRABAJO** en el menú izquierdo.
3. Seleccione **Add stream input** (Agregar entrada de flujo) y, luego, seleccione **Event Hub** (Centro de eventos). 

    ![Adición del centro de eventos como entrada](./media/event-hubs-kafka-stream-analytics/select-event-hub-input.png)
4. En la página de configuración de la **entrada del centro de eventos**, haga lo siguiente: 

    1. Especifique un **alias** para la entrada. 
    2. Seleccione la **suscripción de Azure**.
    3. Seleccione el **espacio de nombres del centro de eventos** que ha creado anteriormente. 
    4. Seleccione **probar** para el **centro de eventos**. 
    5. Seleccione **Guardar**. 

        ![Configuración de entrada del centro de eventos](./media/event-hubs-kafka-stream-analytics/event-hub-input-configuration.png)

### <a name="configure-job-output"></a>Configuración de la salida del trabajo 

1. Seleccione **Salidas** en la sección **TOPOLOGÍA DE TRABAJO** en el menú. 
2. Seleccione **+ Agregar** en la barra de herramientas y seleccione **Blob Storage**
3. Haga lo siguiente en la página de configuración de salida de Blob Storage: 
    1. Especifique un **alias** para la salida. 
    2. Selección la **suscripción**de Azure. 
    3. Seleccione su **cuenta de Azure Storage**. 
    4. Escriba un **nombre para el contenedor** que almacena los datos de salida de la consulta de Stream Analytics.
    5. Seleccione **Guardar**.

        ![Configuración de salida de Blob Storage](./media/event-hubs-kafka-stream-analytics/output-blob-settings.png)
 

### <a name="define-a-query"></a>Definición de una consulta
Ya tiene un trabajo de Stream Analytics configurado para leer un flujo de datos de entrada. El siguiente paso consiste en crear una transformación que analice los datos en tiempo real. Definirá la consulta de transformación mediante el [lenguaje de consulta de Stream Analytics](/stream-analytics-query/stream-analytics-query-language-reference). En este tutorial, definirá una consulta que se pasa a través de los datos sin realizar ninguna transformación.

1. Seleccione **Consulta**.
2. En la ventana de consulta, reemplace `[YourOutputAlias]` por el alias de salida que creó anteriormente.
3. Reemplace `[YourInputAlias]` por el alias de entrada que creó anteriormente. 
4. Seleccione **Guardar** en la barra de herramientas. 

    ![Consultar](./media/event-hubs-kafka-stream-analytics/query.png)


### <a name="run-the-stream-analytics-job"></a>Ejecución del trabajo de Stream Analytics

1. Seleccione **Información general** en el menú de la izquierda. 
2. Seleccione **Inicio**. 

    ![Menú Inicio](./media/event-hubs-kafka-stream-analytics/start-menu.png)
1. En la página **Iniciar trabajo**, seleccione **Iniciar**. 

    ![Página Iniciar trabajo](./media/event-hubs-kafka-stream-analytics/start-job-page.png)
1. Espere hasta que el estado del trabajo cambie de **Iniciando** a **En ejecución**. 

    ![Estado del trabajo: en ejecución](./media/event-hubs-kafka-stream-analytics/running.png)

## <a name="test-the-scenario"></a>Probar el escenario
1. Vuelva a ejecutar el **productor de Kafka** para enviar eventos al centro de eventos. 

    ```shell
    mvn exec:java -Dexec.mainClass="TestProducer"                                    
    ```
1. Confirme que ve que **los datos de salida** se generan en **Azure Blob Storage**. Consulte un archivo JSON en el contenedor con 100 filas con un aspecto similar a las siguientes filas de ejemplo: 

    ```
    {"eventData":"Test Data 0","EventProcessedUtcTime":"2018-08-30T03:27:23.1592910Z","PartitionId":0,"EventEnqueuedUtcTime":"2018-08-30T03:27:22.9220000Z"}
    {"eventData":"Test Data 1","EventProcessedUtcTime":"2018-08-30T03:27:23.3936511Z","PartitionId":0,"EventEnqueuedUtcTime":"2018-08-30T03:27:22.9220000Z"}
    {"eventData":"Test Data 2","EventProcessedUtcTime":"2018-08-30T03:27:23.3936511Z","PartitionId":0,"EventEnqueuedUtcTime":"2018-08-30T03:27:22.9220000Z"}
    ```

    El trabajo de Azure Stream Analytics recibió los datos de entrada del centro de eventos y los almacenó en Azure Blob Storage en este escenario. 



## <a name="next-steps"></a>Pasos siguientes
En este artículo se muestra cómo hacer streaming a instancias de Event Hubs sin cambiar los clientes del protocolo o ejecutar sus propios clústeres. Para más información sobre Event Hubs para Apache Kafka, consulte [Guía del desarrollador de Apache Kafka para Azure Event Hubs](apache-kafka-developer-guide.md). 
