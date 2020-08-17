---
title: Integración de Apache Kafka Connect con Azure Event Hubs | Microsoft Docs
description: En este artículo se proporciona información sobre cómo usar Apache Spark con Azure Event Hubs para Kafka.
ms.topic: how-to
ms.date: 06/23/2020
ms.openlocfilehash: 4c63d27549df40120a90b2594ab54337c11168b6
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87079093"
---
# <a name="integrate-apache-kafka-connect-support-on-azure-event-hubs-preview"></a>Integración de la compatibilidad con Apache Kafka Connect en Azure Event Hubs (versión preliminar)
A medida que aumenta la ingesta para las necesidades del negocio, también aumenta la necesidad de ingerir para varios orígenes y receptores externos. [Apache Kafka Connect](https://kafka.apache.org/documentation/#connect) proporciona dicha plataforma para conectar e importar o exportar datos desde y hacia cualquier sistema externo como MySQL, HDFS y sistema de archivos mediante un clúster de Kafka. Este tutorial le guiará por el uso del marco Kafka Connect con Event Hubs.

Este tutorial le guiará por la integración de Kafka Connect con un centro de eventos de Azure y la implementación de conectores básicos FileStreamSource y FileStreamSink. Esta funcionalidad actualmente está en su versión preliminar. Aunque estos conectores no están pensados para su uso en la producción, muestran un escenario de un extremo a otro de Kafka Connect en el que Azure Event Hubs actúa como un agente de Kafka.

> [!NOTE]
> Este ejemplo está disponible en [GitHub](https://github.com/Azure/azure-event-hubs-for-kafka/tree/master/tutorials/connect).

En este tutorial, realizará los siguientes pasos:

> [!div class="checklist"]
> * Creación de un espacio de nombres de Event Hubs
> * Clonación del proyecto de ejemplo
> * Configuración de Kafka Connect para Event Hubs
> * Ejecución de Kafka Connect
> * Creación de conectores

## <a name="prerequisites"></a>Prerrequisitos
Para completar este tutorial, asegúrese de cumplir estos requisitos previos:

- Suscripción de Azure. En caso de no tener ninguna, [cree una cuenta gratuita](https://azure.microsoft.com/free/).
- [Git](https://www.git-scm.com/downloads)
- Linux/MacOS
- Versión de Kafka (versión 1.1.1, Scala versión 2.11), disponible en [kafka.apache.org](https://kafka.apache.org/downloads#1.1.1)
- Lea el artículo de introducción [Event Hubs para Apache Kafka](./event-hubs-for-kafka-ecosystem-overview.md).

## <a name="create-an-event-hubs-namespace"></a>Creación de un espacio de nombres de Event Hubs
Se requiere un espacio de nombres de Event Hubs para enviar y recibir de cualquier servicio de Event Hubs. Consulte [Creación de un centro de eventos](event-hubs-create.md) para obtener instrucciones sobre cómo crear un espacio de nombres y un centro de eventos. Obtenga la cadena de conexión de Event Hubs y el nombre de dominio completo (FQDN) para su uso posterior. Para obtener instrucciones, consulte [Get an Event Hubs connection string](event-hubs-get-connection-string.md) (Obtención de una cadena de conexión de Event Hubs). 

## <a name="clone-the-example-project"></a>Clonación del proyecto de ejemplo
Clone el repositorio de Azure Event Hubs y vaya a la carpeta tutorials/connect: 

```
git clone https://github.com/Azure/azure-event-hubs-for-kafka.git
cd azure-event-hubs-for-kafka/tutorials/connect
```

## <a name="configure-kafka-connect-for-event-hubs"></a>Configuración de Kafka Connect para Event Hubs
Es necesaria una reconfiguración mínima cuando se redirige el rendimiento de Kafka Connect desde Kafka a Event Hubs.  En el siguiente ejemplo `connect-distributed.properties` se muestra cómo configurar Connect para autenticar y comunicarse con el punto de conexión de Kafka en Event Hubs:

```properties
bootstrap.servers={YOUR.EVENTHUBS.FQDN}:9093 # e.g. namespace.servicebus.windows.net:9093
group.id=connect-cluster-group

# connect internal topic names, auto-created if not exists
config.storage.topic=connect-cluster-configs
offset.storage.topic=connect-cluster-offsets
status.storage.topic=connect-cluster-status

# internal topic replication factors - auto 3x replication in Azure Storage
config.storage.replication.factor=1
offset.storage.replication.factor=1
status.storage.replication.factor=1

rest.advertised.host.name=connect
offset.flush.interval.ms=10000

key.converter=org.apache.kafka.connect.json.JsonConverter
value.converter=org.apache.kafka.connect.json.JsonConverter
internal.key.converter=org.apache.kafka.connect.json.JsonConverter
internal.value.converter=org.apache.kafka.connect.json.JsonConverter

internal.key.converter.schemas.enable=false
internal.value.converter.schemas.enable=false

# required EH Kafka security settings
security.protocol=SASL_SSL
sasl.mechanism=PLAIN
sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required username="$ConnectionString" password="{YOUR.EVENTHUBS.CONNECTION.STRING}";

producer.security.protocol=SASL_SSL
producer.sasl.mechanism=PLAIN
producer.sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required username="$ConnectionString" password="{YOUR.EVENTHUBS.CONNECTION.STRING}";

consumer.security.protocol=SASL_SSL
consumer.sasl.mechanism=PLAIN
consumer.sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required username="$ConnectionString" password="{YOUR.EVENTHUBS.CONNECTION.STRING}";

plugin.path={KAFKA.DIRECTORY}/libs # path to the libs directory within the Kafka release
```

## <a name="run-kafka-connect"></a>Ejecución de Kafka Connect

En este paso, un trabajo de Kafka Connect se inicia localmente en modo distribuido, con Event Hubs para mantener el estado del clúster.

1. Guarde el archivo anterior `connect-distributed.properties` localmente.  Asegúrese de reemplazar todos los valores entre llaves.
2. Vaya a la ubicación de la versión de Kafka en la máquina.
4. Ejecute `./bin/connect-distributed.sh /PATH/TO/connect-distributed.properties`.  La API REST del trabajo de Connect está lista para interactuar cuando vea `'INFO Finished starting connectors and tasks'`. 

> [!NOTE]
> Kafka Connect utiliza Kafka AdminClient API para crear temas automáticamente con configuraciones recomendadas, incluida la compactación. Una rápida comprobación del espacio de nombres en Azure Portal revela que los temas internos del trabajo de Connect se han creado automáticamente.
>
>Los temas internos de Kafka Connect **deben usar compactación**.  El equipo de Event Hubs no es responsable de corregir configuraciones incorrectas si los temas internos de Connect no están configurados correctamente.

### <a name="create-connectors"></a>Creación de conectores
Esta sección le guiará por la rotación de los conectores FileStreamSource y FileStreamSink. 

1. Cree un directorio para archivos de datos de entrada y salida.
    ```bash
    mkdir ~/connect-quickstart
    ```

2. Cree dos archivos: un archivo con datos de inicialización de los que lee el conector FileStreamSource y otro en el que escribe nuestro conector FileStreamSink.
    ```bash
    seq 1000 > ~/connect-quickstart/input.txt
    touch ~/connect-quickstart/output.txt
    ```

3. Cree un conector FileStreamSource.  Asegúrese de reemplazar las llaves por la ruta de acceso del directorio particular.
    ```bash
    curl -s -X POST -H "Content-Type: application/json" --data '{"name": "file-source","config": {"connector.class":"org.apache.kafka.connect.file.FileStreamSourceConnector","tasks.max":"1","topic":"connect-quickstart","file": "{YOUR/HOME/PATH}/connect-quickstart/input.txt"}}' http://localhost:8083/connectors
    ```
    Debería ver el centro de eventos `connect-quickstart` en la instancia de Event Hubs después de ejecutar el comando anterior.
4. Compruebe el estado del conector de origen.
    ```bash
    curl -s http://localhost:8083/connectors/file-source/status
    ```
    Si lo desea, puede usar [Explorador de Service Bus](https://github.com/paolosalvatori/ServiceBusExplorer/releases) para comprobar que los eventos han llegado al tema `connect-quickstart`.

5. Cree un conector FileStreamSink.  Una vez más, asegúrese de reemplazar las llaves por la ruta de acceso del directorio particular.
    ```bash
    curl -X POST -H "Content-Type: application/json" --data '{"name": "file-sink", "config": {"connector.class":"org.apache.kafka.connect.file.FileStreamSinkConnector", "tasks.max":"1", "topics":"connect-quickstart", "file": "{YOUR/HOME/PATH}/connect-quickstart/output.txt"}}' http://localhost:8083/connectors
    ```
 
6. Compruebe el estado del conector del receptor.
    ```bash
    curl -s http://localhost:8083/connectors/file-sink/status
    ```

7. Compruebe que se han replicado los datos entre los archivos y que son idénticos en ambos archivos.
    ```bash
    # read the file
    cat ~/connect-quickstart/output.txt
    # diff the input and output files
    diff ~/connect-quickstart/input.txt ~/connect-quickstart/output.txt
    ```

### <a name="cleanup"></a>Limpieza
Kafka Connect crea temas de centro de eventos para almacenar las configuraciones, los desplazamientos y los estados que persisten incluso después de que el clúster de Connect se haya desactivado. A menos que se desee esta persistencia, se recomienda eliminar estos temas. También puede eliminar el centro de eventos `connect-quickstart` que se creó durante el transcurso de este tutorial.

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información acerca de Event Hubs para Kafka, consulte los artículos siguientes:  

- [Reflejo de un agente de Kafka en un centro de eventos](event-hubs-kafka-mirror-maker-tutorial.md)
- [Conexión de Apache Spark a un centro de eventos](event-hubs-kafka-spark-tutorial.md)
- [Conexión de Apache Flink a un centro de eventos](event-hubs-kafka-flink-tutorial.md)
- [Exploración de ejemplos en nuestro GitHub](https://github.com/Azure/azure-event-hubs-for-kafka)
- [Conexión de Akka Streams a un centro de eventos](event-hubs-kafka-akka-streams-tutorial.md)
- [Guía del desarrollador de Apache Kafka para Azure Event Hubs](apache-kafka-developer-guide.md)
