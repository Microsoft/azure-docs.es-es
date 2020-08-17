---
title: 'Inicio rápido: recepción de eventos mediante Apache Storm: Azure Event Hubs'
description: 'Inicio rápido: En este artículo se proporciona información sobre cómo usar Apache Storm para recibir eventos de Azure Event Hubs.'
ms.topic: quickstart
ms.date: 06/23/2020
ms.openlocfilehash: 099a3ff1ad57616de0d7929d787b82e9d55c9e00
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87087584"
---
# <a name="quickstart-receive-events-from-event-hubs-using-apache-storm"></a>Inicio rápido: Recepción de eventos desde Event Hubs mediante Apache Storm

[Apache Storm](https://storm.incubator.apache.org) es un sistema distribuido de cálculo en tiempo real que simplifica el procesamiento confiable de flujos de datos sin enlazar. Esta sección muestra cómo utilizar un spout de Storm para Azure Event Hubs a fin de recibir eventos de Event Hubs. Con Apache Storm, se pueden dividir los eventos en varios procesos hospedados en distintos nodos. La integración de Event Hubs con Storm simplifica el consumo de eventos al comprobar de forma transparente el progreso mediante la instalación de Zookeeper de Storm, la administración de puntos de comprobación persistentes y las recepciones en paralelo de Event Hubs.

Para más información sobre los patrones de recepción de Event Hubs, vea la [Información general de Event Hubs][Event Hubs overview].

## <a name="prerequisites"></a>Prerrequisitos
Antes de empezar con el inicio rápido, **cree un espacio de nombres de Event Hubs y un centro de eventos**. Use [Azure Portal](https://portal.azure.com) para crear un espacio de nombres de tipo Event Hubs y obtener las credenciales de administración que la aplicación necesita para comunicarse con el centro de eventos. Para crear un espacio de nombres y un centro de eventos, siga el procedimiento que se indica en [este artículo](event-hubs-create.md). 

## <a name="create-project-and-add-code"></a>Creación del proyecto y adición de código

Este tutorial usa una instalación de [HDInsight Storm][HDInsight Storm] , que integra el emisor de Event Hubs que ya se encuentra disponible.

1. Siga el procedimiento descrito en [Introducción a HDInsight Storm](../hdinsight/storm/apache-storm-overview.md) para crear un clúster nuevo de HDInsight y conectarlo a través del Escritorio remoto.
2. Copie el archivo `%STORM_HOME%\examples\eventhubspout\eventhubs-storm-spout-0.9-jar-with-dependencies.jar` en su entorno de desarrollo local. Contiene events-storm-spout.
3. Utilice el comando siguiente para instalar el paquete en el almacén Maven local. Esto permite agregarlo como referencia en el proyecto de Storm en un paso posterior.

    ```shell
    mvn install:install-file -Dfile=target\eventhubs-storm-spout-0.9-jar-with-dependencies.jar -DgroupId=com.microsoft.eventhubs -DartifactId=eventhubs-storm-spout -Dversion=0.9 -Dpackaging=jar
    ```
4. En Eclipse, cree un proyecto Maven nuevo (haga clic en **Archivo**, **Nuevo** y, a continuación, en **Proyecto**).
   
    ![Archivo > Nuevo > Proyecto][12]
5. Seleccione **Usar ubicación del área de trabajo predeterminada** y, a continuación, haga clic en **Siguiente**
6. Seleccione el arquetipo **maven-archetype-quickstart** y, a continuación, haga clic en **Siguiente**
7. Inserte un **GroupId** y **ArtifactId** y, a continuación, haga clic en **Finalizar**
8. En **pom.xml**, agregue las siguientes dependencias en el nodo `<dependency>`.

    ```xml  
    <dependency>
        <groupId>org.apache.storm</groupId>
        <artifactId>storm-core</artifactId>
        <version>0.9.2-incubating</version>
        <scope>provided</scope>
    </dependency>
    <dependency>
        <groupId>com.microsoft.eventhubs</groupId>
        <artifactId>eventhubs-storm-spout</artifactId>
        <version>0.9</version>
    </dependency>
    <dependency>
        <groupId>com.netflix.curator</groupId>
        <artifactId>curator-framework</artifactId>
        <version>1.3.3</version>
        <exclusions>
            <exclusion>
                <groupId>log4j</groupId>
                <artifactId>log4j</artifactId>
            </exclusion>
            <exclusion>
                <groupId>org.slf4j</groupId>
                <artifactId>slf4j-log4j12</artifactId>
            </exclusion>
        </exclusions>
        <scope>provided</scope>
    </dependency>
    ```

9. En la carpeta **src**, cree un archivo llamado **Config.properties** y copie el siguiente contenido, sustituyendo los valores `receive rule key` y `event hub name`:

    ```java
    eventhubspout.username = ReceiveRule
    eventhubspout.password = {receive rule key}
    eventhubspout.namespace = ioteventhub-ns
    eventhubspout.entitypath = {event hub name}
    eventhubspout.partitions.count = 16
       
    # if not provided, will use storm's zookeeper settings
    # zookeeper.connectionstring=localhost:2181
       
    eventhubspout.checkpoint.interval = 10
    eventhub.receiver.credits = 10
    ```
    El valor de **eventhub.receiver.credits** determina cuántos eventos se procesan por lotes antes de liberarlos a la canalización de Storm. Por simplicidad, este ejemplo establece el valor en 10. En producción, se debe normalmente establecer en valores más altos; Por ejemplo, 1024.
10. Cree una clase nueva denominada **LoggerBolt** con el código siguiente:
    
    ```java
    import java.util.Map;
    import org.slf4j.Logger;
    import org.slf4j.LoggerFactory;
    import backtype.storm.task.OutputCollector;
    import backtype.storm.task.TopologyContext;
    import backtype.storm.topology.OutputFieldsDeclarer;
    import backtype.storm.topology.base.BaseRichBolt;
    import backtype.storm.tuple.Tuple;
    
    public class LoggerBolt extends BaseRichBolt {
        private OutputCollector collector;
        private static final Logger logger = LoggerFactory
                  .getLogger(LoggerBolt.class);
    
        @Override
        public void execute(Tuple tuple) {
            String value = tuple.getString(0);
            logger.info("Tuple value: " + value);
   
            collector.ack(tuple);
        }
   
        @Override
        public void prepare(Map map, TopologyContext context, OutputCollector collector) {
            this.collector = collector;
            this.count = 0;
        }
        
        @Override
        public void declareOutputFields(OutputFieldsDeclarer declarer) {
            // no output fields
        }
    
    }
    ```
    
    Este elemento de Storm registra el contenido de los eventos recibidos. Esto se puede ampliar fácilmente para almacenar las tuplas en un servicio de almacenamiento. El [ejemplo de HDInsight Storm con Event Hubs] usa este mismo enfoque para almacenar datos en Azure Storage y Power BI.
11. Cree una clase denominada **LogTopology** con el código siguiente:
    
    ```java
    import java.io.FileReader;
    import java.util.Properties;
    import backtype.storm.Config;
    import backtype.storm.LocalCluster;
    import backtype.storm.StormSubmitter;
    import backtype.storm.generated.StormTopology;
    import backtype.storm.topology.TopologyBuilder;
    import com.microsoft.eventhubs.samples.EventCount;
    import com.microsoft.eventhubs.spout.EventHubSpout;
    import com.microsoft.eventhubs.spout.EventHubSpoutConfig;
        
    public class LogTopology {
        protected EventHubSpoutConfig spoutConfig;
        protected int numWorkers;
        
        protected void readEHConfig(String[] args) throws Exception {
            Properties properties = new Properties();
            if (args.length > 1) {
                properties.load(new FileReader(args[1]));
            } else {
                properties.load(EventCount.class.getClassLoader()
                        .getResourceAsStream("Config.properties"));
            }
        
            String username = properties.getProperty("eventhubspout.username");
            String password = properties.getProperty("eventhubspout.password");
            String namespaceName = properties
                    .getProperty("eventhubspout.namespace");
            String entityPath = properties.getProperty("eventhubspout.entitypath");
            String zkEndpointAddress = properties
                    .getProperty("zookeeper.connectionstring"); // opt
            int partitionCount = Integer.parseInt(properties
                    .getProperty("eventhubspout.partitions.count"));
            int checkpointIntervalInSeconds = Integer.parseInt(properties
                    .getProperty("eventhubspout.checkpoint.interval"));
            int receiverCredits = Integer.parseInt(properties
                    .getProperty("eventhub.receiver.credits")); // prefetch count
                                                                // (opt)
            System.out.println("Eventhub spout config: ");
            System.out.println("  partition count: " + partitionCount);
            System.out.println("  checkpoint interval: "
                    + checkpointIntervalInSeconds);
            System.out.println("  receiver credits: " + receiverCredits);
     
            spoutConfig = new EventHubSpoutConfig(username, password,
                    namespaceName, entityPath, partitionCount, zkEndpointAddress,
                    checkpointIntervalInSeconds, receiverCredits);
        
            // set the number of workers to be the same as partition number.
            // the idea is to have a spout and a logger bolt co-exist in one
            // worker to avoid shuffling messages across workers in storm cluster.
            numWorkers = spoutConfig.getPartitionCount();
        
            if (args.length > 0) {
                // set topology name so that sample Trident topology can use it as
                // stream name.
                spoutConfig.setTopologyName(args[0]);
            }
        }
        
        protected StormTopology buildTopology() {
            TopologyBuilder topologyBuilder = new TopologyBuilder();
       
            EventHubSpout eventHubSpout = new EventHubSpout(spoutConfig);
            topologyBuilder.setSpout("EventHubsSpout", eventHubSpout,
                    spoutConfig.getPartitionCount()).setNumTasks(
                    spoutConfig.getPartitionCount());
            topologyBuilder
                    .setBolt("LoggerBolt", new LoggerBolt(),
                            spoutConfig.getPartitionCount())
                    .localOrShuffleGrouping("EventHubsSpout")
                    .setNumTasks(spoutConfig.getPartitionCount());
            return topologyBuilder.createTopology();
        }
        
        protected void runScenario(String[] args) throws Exception {
            boolean runLocal = true;
            readEHConfig(args);
            StormTopology topology = buildTopology();
            Config config = new Config();
            config.setDebug(false);
        
            if (runLocal) {
                config.setMaxTaskParallelism(2);
                LocalCluster localCluster = new LocalCluster();
                localCluster.submitTopology("test", config, topology);
                Thread.sleep(5000000);
                localCluster.shutdown();
            } else {
                config.setNumWorkers(numWorkers);
                StormSubmitter.submitTopology(args[0], config, topology);
            }
        }
        
        public static void main(String[] args) throws Exception {
            LogTopology topology = new LogTopology();
            topology.runScenario(args);
        }
    }
    ```

    Esta clase crea un emisor de Event Hubs, utilizando las propiedades del archivo de configuración para crear una instancia. Es importante tener en cuenta que este ejemplo crea tantas tareas de spout como número de particiones hay en el centro de eventos, para poder usar el paralelismo máximo permitido por ese centro de eventos.

## <a name="next-steps"></a>Pasos siguientes
Para más información acerca de Event Hubs, visite los vínculos siguientes:

* [Información general de Event Hubs][Event Hubs overview]
* [Creación de un centro de eventos](event-hubs-create.md)
* [Preguntas más frecuentes sobre Event Hubs](event-hubs-faq.md)

<!-- Links -->
[Event Hubs overview]: ./event-hubs-about.md
[HDInsight Storm]: ../hdinsight/storm/apache-storm-overview.md
[Ejemplo de HDInsight Storm con Event Hubs]: https://github.com/Azure-Samples/hdinsight-java-storm-eventhub

<!-- Images -->

[12]: ./media/event-hubs-get-started-receive-storm/create-storm1.png
