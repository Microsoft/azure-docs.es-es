---
title: 'Tutorial: Uso de Streams API de Apache Kafka: Azure HDInsight '
description: 'Tutorial: Aprenda a usar Streams API de Apache Kafka con Kafka en HDInsight. Esta API permite realizar el procesamiento de flujos entre temas de Kafka.'
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: tutorial
ms.custom: hdinsightactive
ms.date: 03/20/2020
ms.openlocfilehash: 0174c40a0fada0f78cc8d52f5c45b991c3851da0
ms.sourcegitcommit: 62717591c3ab871365a783b7221851758f4ec9a4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/22/2020
ms.locfileid: "85850566"
---
# <a name="tutorial-use-apache-kafka-streams-api-in-azure-hdinsight"></a>Tutorial: Uso de Streams API de Apache Kafka en Azure HDInsight

Aprenda a crear una aplicación que use Apache Kafka Streams API y ejecútela con Kafka en HDInsight.

La aplicación que se usa en este tutorial es un recuento de palabras de streaming. Lee datos de texto de un tema de Kafka, extrae las palabras individuales y, a continuación, almacena el recuento de palabras en otro tema de Kafka.

El procesamiento de flujos de Kafka a menudo se realiza con Apache Spark o Apache Storm. Kafka Streams API se incluyó por primera vez en la versión 1.1.0 de Kafka (en HDInsight 3.5 y 3.6). Esta API le permite transformar flujos de datos entre los temas de entrada y de salida. En algunos casos, esta puede ser una alternativa a la creación de una solución de streaming de Spark o Storm.

Para más información sobre Kafka Streams, consulte la documentación de [introducción a Kafka Streams](https://kafka.apache.org/10/documentation/streams/) en Apache.org.

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Comprendiendo el código
> * Compilar e implementar la aplicación
> * Configurar temas de Kafka
> * Ejecución del código

## <a name="prerequisites"></a>Prerrequisitos

* Un clúster de Kafka en HDInsight 3.6. Para aprender a crear un clúster de Kafka en HDInsight, consulte el documento [Inicio de Apache Kafka en HDInsight](apache-kafka-get-started.md).

* Complete los pasos que se indican en el documento [Producer API y Consumer API de Apache Kafka](apache-kafka-producer-consumer-api.md). Los pasos de este documento utilizan la aplicación y los temas de ejemplo que creó en este tutorial.

* [Java Developer Kit (JDK) versión 8](https://aka.ms/azure-jdks), o un equivalente, como OpenJDK.

* [Apache Maven](https://maven.apache.org/download.cgi) correctamente [instalado](https://maven.apache.org/install.html) según Apache.  Maven es un sistema de compilación de proyectos de Java.

* Un cliente SSH. Para más información, consulte [Conexión a través de SSH con HDInsight (Apache Hadoop)](../hdinsight-hadoop-linux-use-ssh-unix.md).

## <a name="understand-the-code"></a>Comprendiendo el código

La aplicación de ejemplo se encuentra en [https://github.com/Azure-Samples/hdinsight-kafka-java-get-started](https://github.com/Azure-Samples/hdinsight-kafka-java-get-started), en el subdirectorio `Streaming`. La aplicación consta de dos archivos:

* `pom.xml`: este archivo define las dependencias del proyecto, la versión de Java y los métodos de empaquetado.
* `Stream.java`: este archivo implementa la lógica de streaming.

### <a name="pomxml"></a>Pom.xml

Esto es lo más importante que hay que saber del archivo `pom.xml`:

* Dependencias: este proyecto utiliza Streams API de Kafka, que el paquete `kafka-clients` proporciona. El siguiente código XML define esta dependencia:

    ```xml
    <!-- Kafka client for producer/consumer operations -->
    <dependency>
            <groupId>org.apache.kafka</groupId>
            <artifactId>kafka-clients</artifactId>
            <version>${kafka.version}</version>
    </dependency>
    ```

    La entrada `${kafka.version}` se declara en la sección `<properties>..</properties>` de `pom.xml` y está configurada para la versión Kafka del clúster de HDInsight.

* Complementos: los complementos de Maven proporcionan varias funcionalidades. En este proyecto, se utilizan los siguientes complementos:

    * `maven-compiler-plugin`: se utiliza para establecer que el proyecto utiliza la versión 8 de Java. HDInsight 3.6 requiere Java 8.
    * `maven-shade-plugin`: se utiliza para generar un archivo uber-jar que contiene esta aplicación y todas las dependencias. También se usa para establecer el punto de entrada de la aplicación, con el fin de que pueda ejecutar directamente el archivo Jar sin tener que especificar la clase principal.

### <a name="streamjava"></a>Stream.java

El archivo [Stream.java](https://github.com/Azure-Samples/hdinsight-kafka-java-get-started/blob/master/Streaming/src/main/java/com/microsoft/example/Stream.java) utiliza Streams API para implementar una aplicación de recuento de palabras. Lee datos de un tema de Kafka llamado `test` y escribe los recuentos de palabras en un tema llamado `wordcounts`.

El código siguiente define la aplicación de recuento de palabras:

```java
package com.microsoft.example;

import org.apache.kafka.common.serialization.Serde;
import org.apache.kafka.common.serialization.Serdes;
import org.apache.kafka.streams.KafkaStreams;
import org.apache.kafka.streams.KeyValue;
import org.apache.kafka.streams.StreamsConfig;
import org.apache.kafka.streams.kstream.KStream;
import org.apache.kafka.streams.kstream.KStreamBuilder;

import java.util.Arrays;
import java.util.Properties;

public class Stream
{
    public static void main( String[] args ) {
        Properties streamsConfig = new Properties();
        // The name must be unique on the Kafka cluster
        streamsConfig.put(StreamsConfig.APPLICATION_ID_CONFIG, "wordcount-example");
        // Brokers
        streamsConfig.put(StreamsConfig.BOOTSTRAP_SERVERS_CONFIG, args[0]);
        // SerDes for key and values
        streamsConfig.put(StreamsConfig.KEY_SERDE_CLASS_CONFIG, Serdes.String().getClass().getName());
        streamsConfig.put(StreamsConfig.VALUE_SERDE_CLASS_CONFIG, Serdes.String().getClass().getName());

        // Serdes for the word and count
        Serde<String> stringSerde = Serdes.String();
        Serde<Long> longSerde = Serdes.Long();

        KStreamBuilder builder = new KStreamBuilder();
        KStream<String, String> sentences = builder.stream(stringSerde, stringSerde, "test");
        KStream<String, Long> wordCounts = sentences
                .flatMapValues(value -> Arrays.asList(value.toLowerCase().split("\\W+")))
                .map((key, word) -> new KeyValue<>(word, word))
                .countByKey("Counts")
                .toStream();
        wordCounts.to(stringSerde, longSerde, "wordcounts");

        KafkaStreams streams = new KafkaStreams(builder, streamsConfig);
        streams.start();

        Runtime.getRuntime().addShutdownHook(new Thread(streams::close));
    }
}
```

## <a name="build-and-deploy-the-example"></a>Compilación e implementación del ejemplo

Use los siguientes pasos para compilar e implementar el proyecto en el clúster de Kafka en HDInsight:

1. Establezca el directorio actual en la ubicación del directorio `hdinsight-kafka-java-get-started-master\Streaming` y luego use el siguiente comando para crear un paquete jar:

    ```cmd
    mvn clean package
    ```

    Este comando crea el paquete en `target/kafka-streaming-1.0-SNAPSHOT.jar`.

2. Reemplace `sshuser` por el usuario de SSH del clúster y `clustername` por el nombre de su clúster. Use el siguiente comando para copiar el archivo `kafka-streaming-1.0-SNAPSHOT.jar` en el clúster de HDInsight. Si se le solicita, escriba la contraseña de la cuenta de usuario de SSH.

    ```cmd
    scp ./target/kafka-streaming-1.0-SNAPSHOT.jar sshuser@clustername-ssh.azurehdinsight.net:kafka-streaming.jar
    ```

## <a name="create-apache-kafka-topics"></a>Creación de temas de Apache Kafka

1. Reemplace `sshuser` por el usuario de SSH del clúster y `CLUSTERNAME` por el nombre de su clúster. Abra una conexión de SSH con el clúster, para lo que debe escribir el siguiente comando. Si se le solicita, escriba la contraseña de la cuenta de usuario de SSH.

    ```bash
    ssh sshuser@CLUSTERNAME-ssh.azurehdinsight.net
    ```

2. Instale [jq](https://stedolan.github.io/jq/), un procesador JSON de línea de comandos. En la conexión SSH abierta, escriba el siguiente comando para instalar `jq`:

    ```bash
    sudo apt -y install jq
    ```

3. Configure una variable de contraseña. Reemplace `PASSWORD` por la contraseña de inicio de sesión del clúster y, después, escriba el comando:

    ```bash
    export password='PASSWORD'
    ```

4. Extraiga el nombre del clúster con las mayúsculas y minúsculas correctas. Las mayúsculas y minúsculas reales del nombre del clúster pueden no ser como cabría esperar, dependen de la forma en que se haya creado el clúster. Este comando obtendrá las mayúsculas y minúsculas reales y después las almacenará en una variable. Escriba el comando siguiente:

    ```bash
    export clusterName=$(curl -u admin:$password -sS -G "http://headnodehost:8080/api/v1/clusters" | jq -r '.items[].Clusters.cluster_name')
    ```

    > [!Note]  
    > Si está realizando este proceso desde fuera del clúster, hay un procedimiento diferente para almacenar el nombre del clúster. Obtenga el nombre del clúster en minúsculas desde Azure Portal. A continuación, sustituya el nombre del clúster por `<clustername>` en el siguiente comando y ejecútelo: `export clusterName='<clustername>'`.  

5. Para obtener tanto los hosts del agente de Kafka como los hosts de Apache Zookeeper, use los siguientes comandos. Cuando se le solicite, escriba la contraseña de administrador del clúster.

    ```bash
    export KAFKAZKHOSTS=$(curl -sS -u admin:$password -G https://$clusterName.azurehdinsight.net/api/v1/clusters/$clusterName/services/ZOOKEEPER/components/ZOOKEEPER_SERVER | jq -r '["\(.host_components[].HostRoles.host_name):2181"] | join(",")' | cut -d',' -f1,2);

    export KAFKABROKERS=$(curl -sS -u admin:$password -G https://$clusterName.azurehdinsight.net/api/v1/clusters/$clusterName/services/KAFKA/components/KAFKA_BROKER | jq -r '["\(.host_components[].HostRoles.host_name):9092"] | join(",")' | cut -d',' -f1,2);
    ```

    > [!Note]  
    > Estos comandos requieren acceso a Ambari. Si el clúster se encuentra detrás de un grupo de seguridad de red, ejecute estos comandos desde una máquina que pueda acceder a Ambari.

6. Para crear los temas que emplea la operación de streaming, use los siguientes comandos:

    > [!NOTE]  
    > Puede recibir un error que indica que el tema `test` ya existe. Esto es normal, ya que puede que lo haya creado en el tutorial sobre Producer y Consumer API.

    ```bash
    /usr/hdp/current/kafka-broker/bin/kafka-topics.sh --create --replication-factor 3 --partitions 8 --topic test --zookeeper $KAFKAZKHOSTS
    /usr/hdp/current/kafka-broker/bin/kafka-topics.sh --create --replication-factor 3 --partitions 8 --topic wordcounts --zookeeper $KAFKAZKHOSTS
    /usr/hdp/current/kafka-broker/bin/kafka-topics.sh --create --replication-factor 3 --partitions 8 --topic RekeyedIntermediateTopic --zookeeper $KAFKAZKHOSTS
    /usr/hdp/current/kafka-broker/bin/kafka-topics.sh --create --replication-factor 3 --partitions 8 --topic wordcount-example-Counts-changelog --zookeeper $KAFKAZKHOSTS
    ```

    Los temas se utilizan para los siguientes fines:

   * `test`: este es el tema en el que se reciben los registros. La aplicación de streaming los lee de aquí.
   * `wordcounts`: este es el tema en el que la aplicación de streaming almacena su salida.
   * `RekeyedIntermediateTopic`: este tema se usa para volver a particionar los datos a medida que el operador `countByKey` actualiza el recuento.
   * `wordcount-example-Counts-changelog`: este tema es un almacén de estados que la operación `countByKey` usa.

    Kafka en HDInsight se puede también configurar para que cree temas automáticamente. Para más información, consulte el documento [Configure automatic topic creation](apache-kafka-auto-create-topics.md) (Configuración de la creación automática de temas).

## <a name="run-the-code"></a>Ejecución del código

1. Para iniciar la aplicación de streaming como un proceso en segundo plano, use el comando siguiente:

    ```bash
    java -jar kafka-streaming.jar $KAFKABROKERS $KAFKAZKHOSTS &
    ```

    Puede recibir una advertencia sobre log4j de Apache. Puede pasarla por alto.

2. Para enviar registros al tema `test`, use el comando siguiente para iniciar la aplicación de producción:

    ```bash
    java -jar kafka-producer-consumer.jar producer test $KAFKABROKERS
    ```

3. Una vez finalizada la producción, use el siguiente comando para ver la información almacenada en el tema `wordcounts`:

    ```bash
    /usr/hdp/current/kafka-broker/bin/kafka-console-consumer.sh --bootstrap-server $KAFKABROKERS --topic wordcounts --formatter kafka.tools.DefaultMessageFormatter --property print.key=true --property key.deserializer=org.apache.kafka.common.serialization.StringDeserializer --property value.deserializer=org.apache.kafka.common.serialization.LongDeserializer --from-beginning
    ```

    Los parámetros `--property` indican al consumidor de consola que imprima la clave (palabra) junto con el número (valor). Este parámetro también configura el deserializador que se utilizará al leer estos valores de Kafka.

    La salida será similar al siguiente texto:

    ```output
    dwarfs  13635
    ago     13664
    snow    13636
    dwarfs  13636
    ago     13665
    a       13803
    ago     13666
    a       13804
    ago     13667
    ago     13668
    jumped  13640
    jumped  13641
    ```

    El parámetro `--from-beginning` configura el consumidor para que empiece por el primero de los registros almacenados en el tema. El recuento se incrementa cada vez que se encuentra una palabra, por lo que el tema contiene varias entradas para cada palabra, con un recuento creciente.

4. Use __Ctrl + C__ para salir del productor. Siga usando __Ctrl + C__ para salir de la aplicación y del consumidor.

5. Para eliminar los temas que emplea la operación de streaming, use los siguientes comandos:

    ```bash
    /usr/hdp/current/kafka-broker/bin/kafka-topics.sh --delete --topic test --zookeeper $KAFKAZKHOSTS
    /usr/hdp/current/kafka-broker/bin/kafka-topics.sh --delete --topic wordcounts --zookeeper $KAFKAZKHOSTS
    /usr/hdp/current/kafka-broker/bin/kafka-topics.sh --delete --topic RekeyedIntermediateTopic --zookeeper $KAFKAZKHOSTS
    /usr/hdp/current/kafka-broker/bin/kafka-topics.sh --delete --topic wordcount-example-Counts-changelog --zookeeper $KAFKAZKHOSTS
    ```

## <a name="clean-up-resources"></a>Limpieza de recursos

Para limpiar los recursos creados por este tutorial puede eliminar el grupo de recursos. Al eliminar el grupo de recursos, también se elimina el clúster de HDInsight asociado y otros recursos asociados al grupo.

Para quitar el grupo de recursos mediante Azure Portal:

1. En Azure Portal, expanda el menú en el lado izquierdo para abrir el menú de servicios y elija __Grupos de recursos__ para mostrar la lista de sus grupos de recursos.
2. Busque el grupo de recursos que desea eliminar y haga clic con el botón derecho en __Más__ (...) en el lado derecho de la lista.
3. Seleccione __Eliminar grupo de recursos__ y confirme la elección.

## <a name="next-steps"></a>Pasos siguientes

En este documento, ha aprendido a usar Streams API de Apache Kafka con Kafka en HDInsight. Use los recursos siguientes para obtener más información sobre cómo trabajar con Kafka.

> [!div class="nextstepaction"]
> [Análisis de registros de Apache Kafka](apache-kafka-log-analytics-operations-management.md)
