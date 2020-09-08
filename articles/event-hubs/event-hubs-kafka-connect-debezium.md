---
title: Integración de Apache Kafka Connect en Azure Event Hubs (versión preliminar) con Debezium para operaciones de captura de datos modificados
description: En este artículo se proporciona información sobre cómo usar Apache Spark con Azure Event Hubs para Kafka.
ms.topic: how-to
author: abhirockzz
ms.author: abhishgu
ms.date: 08/11/2020
ms.openlocfilehash: a11ec882a50d051a34758562ac84dcef5b799f5f
ms.sourcegitcommit: 1aef4235aec3fd326ded18df7fdb750883809ae8
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/12/2020
ms.locfileid: "88136672"
---
# <a name="integrate-apache-kafka-connect-support-on-azure-event-hubs-preview-with-debezium-for-change-data-capture"></a>Integración de la compatibilidad de Apache Kafka Connect en Azure Event Hubs (versión preliminar) con Debezium para operaciones de captura de datos modificados

La **captura de datos modificados (CDC)** es una técnica que se usa para realizar un seguimiento de los cambios de nivel de fila en tablas de base de datos como respuesta a las operaciones de creación, actualización y eliminación. [Debezium](https://debezium.io/) es una plataforma distribuida que se basa en las características de captura de datos modificados que hay disponibles en distintas bases de datos (por ejemplo, la [descodificación lógica en PostgreSQL](https://www.postgresql.org/docs/current/static/logicaldecoding-explanation.html)). Proporciona un conjunto de [conectores Kafka Connect](https://debezium.io/documentation/reference/1.2/connectors/index.html) que aprovechan los cambios de nivel de fila en tablas de base de datos y los convierten en secuencias de eventos que se envían a [Apache Kafka](https://kafka.apache.org/).

Este tutorial lo guiará a través de la configuración de un sistema basado en la captura de datos modificados en Azure con [Azure Event Hubs](https://docs.microsoft.com/azure/event-hubs/event-hubs-about?WT.mc_id=devto-blog-abhishgu) (para Kafka), [Azure Database for PostgreSQL](../postgresql/overview.md) y Debezium. Usará el [conector de PostgreSQL de Debezium ](https://debezium.io/documentation/reference/1.2/connectors/postgresql.html) para transmitir modificaciones de base de datos desde PostgreSQL a temas de Kafka en Azure Event Hubs.

En este tutorial, realizará los siguientes pasos:

> [!div class="checklist"]
> * Creación de un espacio de nombres de Event Hubs
> * Instalación y configuración de Azure Database for PostgreSQL
> * Configuración y ejecución de Kafka Connect con el conector de Debezium PostgreSQL
> * Prueba de la captura de datos modificados
> * Consumo de eventos de datos modificados con un conector `FileStreamSink` (opcional)

## <a name="pre-requisites"></a>Requisitos previos
Para completar este tutorial, necesitará lo siguiente:

- Suscripción de Azure. En caso de no tener ninguna, [cree una cuenta gratuita](https://azure.microsoft.com/free/).
- Linux/MacOS
- Versión de Kafka (versión 1.1.1, Scala versión 2.11), disponible en [kafka.apache.org](https://kafka.apache.org/downloads#1.1.1)
- Lea el artículo de introducción [Event Hubs para Apache Kafka](./event-hubs-for-kafka-ecosystem-overview.md).

## <a name="create-an-event-hubs-namespace"></a>Creación de un espacio de nombres de Event Hubs
Se requiere un espacio de nombres de Event Hubs para enviar y recibir de cualquier servicio de Event Hubs. Vea [Creación de un centro de eventos](event-hubs-create.md) para obtener instrucciones sobre cómo crear un espacio de nombres y un centro de eventos. Obtenga la cadena de conexión de Event Hubs y el nombre de dominio completo (FQDN) para su uso posterior. Para obtener instrucciones, consulte [Get an Event Hubs connection string](event-hubs-get-connection-string.md) (Obtención de una cadena de conexión de Event Hubs). 

## <a name="setup-and-configure-azure-database-for-postgresql"></a>Instalación y configuración de Azure Database for PostgreSQL
[Azure Database for PostgreSQL](../postgresql/overview.md) es un servicio de base de datos relacional basado en la versión de la comunidad del motor de base de datos PostgreSQL de código abierto, y se encuentra disponible en dos opciones de implementación: Un solo servidor e Hiperescala (Citus) [Siga las instrucciones](../postgresql/quickstart-create-server-database-portal.md) para crear un servidor de Azure Database for PostgreSQL a través de Azure Portal. 

## <a name="setup-and-run-kafka-connect"></a>Instalación y ejecución de Kafka Connect
En esta sección se tratarán los siguientes temas:

- Instalación del conector de Debezium
- Configuración de Kafka Connect para Event Hubs
- Inicio del clúster de Kafka Connect con el conector de Debezium

### <a name="download-and-setup-debezium-connector"></a>Descarga y configuración del conector de Debezium
Siga las instrucciones más recientes de la [documentación de Debezium](https://debezium.io/documentation/reference/1.2/connectors/postgresql.html#postgresql-deploying-a-connector) para descargar y configurar el conector.

- Descargue el archivo de complemento del conector. Por ejemplo, para descargar la versión `1.2.0` del conector, use este vínculo: https://repo1.maven.org/maven2/io/debezium/debezium-connector-postgres/1.2.0.Final/debezium-connector-postgres-1.2.0.Final-plugin.tar.gz.
- Extraiga los archivos JAR y cópielos en el archivo [plugin.path de Kafka Connect](https://kafka.apache.org/documentation/#connectconfigs).


### <a name="configure-kafka-connect-for-event-hubs"></a>Configuración de Kafka Connect para Event Hubs
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

### <a name="run-kafka-connect"></a>Ejecución de Kafka Connect
En este paso, un trabajo de Kafka Connect se inicia localmente en modo distribuido, con Event Hubs para mantener el estado del clúster.

1. Guarde el archivo anterior `connect-distributed.properties` localmente.  Asegúrese de reemplazar todos los valores entre llaves.
2. Vaya a la ubicación de la versión de Kafka en la máquina.
3. Ejecute `./bin/connect-distributed.sh /PATH/TO/connect-distributed.properties` y espere a que se inicie el clúster.

> [!NOTE]
> Kafka Connect utiliza Kafka AdminClient API para crear temas automáticamente con configuraciones recomendadas, incluida la compactación. Una rápida comprobación del espacio de nombres en Azure Portal revela que los temas internos del trabajo de Connect se han creado automáticamente.
>
> Los temas internos de Kafka Connect **deben usar compactación**.  El equipo de Event Hubs no es responsable de corregir configuraciones incorrectas si los temas internos de Connect no están configurados correctamente.

### <a name="configure-and-start-the-debezium-postgresql-source-connector"></a>Configuración e inicio del conector de origen de PostgreSQL de Debezium

Cree un archivo de configuración (`pg-source-connector.json`) para el conector de origen de PostgreSQL. Reemplace los valores según su instancia de Azure PostgreSQL.

```json
{
    "name": "todo-connector",
    "config": {
        "connector.class": "io.debezium.connector.postgresql.PostgresConnector",
        "database.hostname": "<replace with Azure PostgreSQL instance name>.postgres.database.azure.com",
        "database.port": "5432",
        "database.user": "<replace with database user name>",
        "database.password": "<replace with database password>",
        "database.dbname": "postgres",
        "database.server.name": "my-server",
        "plugin.name": "wal2json",
        "table.whitelist": "public.todos"
    }
}
```

> [!TIP]
> El atributo `database.server.name` es un nombre lógico que identifica y proporciona un espacio de nombres para el clúster o servidor de base de datos de PostgreSQL que se está supervisando. Para obtener información detallada, consulte la [documentación de Debezium](https://debezium.io/documentation/reference/1.2/connectors/postgresql.html#postgresql-property-database-server-name).

Para crear una instancia del conector, use el punto de conexión de la API REST de Kafka Connect:

```bash
curl -X POST -H "Content-Type: application/json" --data @pg-source-connector.json http://localhost:8083/connectors
```

Para comprobar el estado del conector:

```bash
curl -s http://localhost:8083/connectors/todo-connector/status
```

## <a name="test-change-data-capture"></a>Prueba de la captura de datos modificados
Para ver la captura de datos modificados en acción, debe crear, actualizar o eliminar registros en la base de datos de Azure PostgreSQL.

Para empezar, conéctese a la base de datos de Azure PostgreSQL (el ejemplo siguiente usa [psql](https://www.postgresql.org/docs/12/app-psql.html)).

```bash
psql -h <POSTGRES_INSTANCE_NAME>.postgres.database.azure.com -p 5432 -U <POSTGRES_USER_NAME> -W -d <POSTGRES_DB_NAME> --set=sslmode=require

e.g. 

psql -h my-postgres.postgres.database.azure.com -p 5432 -U testuser@my-postgres -W -d postgres --set=sslmode=require
```

**Creación de una tabla e inserción de datos**

```sql
CREATE TABLE todos (id SERIAL, description VARCHAR(50), todo_status VARCHAR(10), PRIMARY KEY(id));

INSERT INTO todos (description, todo_status) VALUES ('setup postgresql on azure', 'complete');
INSERT INTO todos (description, todo_status) VALUES ('setup kafka connect', 'complete');
INSERT INTO todos (description, todo_status) VALUES ('configure and install connector', 'in-progress');
INSERT INTO todos (description, todo_status) VALUES ('start connector', 'pending');
```

El conector ahora debería pasar a la acción y enviar los eventos de los datos modificados a un tema de Event Hubs con el nombre `my-server.public.todos`, dando por hecho que `my-server` es el valor de `database.server.name` y que `public.todos` es la tabla de cuyos cambios está realizando el seguimiento (según la configuración de `table.whitelist`).

**Consulta del tema de Event Hubs**

Inspeccionemos el contenido del tema para asegurarnos de que todo funciona según lo previsto. En el ejemplo siguiente se usa [`kafkacat`](https://github.com/Azure/azure-event-hubs-for-kafka/tree/master/quickstart/kafkacat), pero también puede [crear un consumidor con cualquiera de las opciones que se muestran aquí](apache-kafka-developer-guide.md).

Cree un nuevo archivo denominado `kafkacat.conf` con el contenido siguiente:

```
metadata.broker.list=<enter event hubs namespace>.servicebus.windows.net:9093
security.protocol=SASL_SSL
sasl.mechanisms=PLAIN
sasl.username=$ConnectionString
sasl.password=<enter event hubs connection string>
```

> [!NOTE]
> Actualice los atributos `metadata.broker.list` y `sasl.password` de `kafkacat.conf` según la información de Event Hubs. 

En un terminal diferente, inicie un consumidor:

```bash
export KAFKACAT_CONFIG=kafkacat.conf
export BROKER=<enter event hubs namespace>.servicebus.windows.net:9093
export TOPIC=my-server.public.todos

kafkacat -b $BROKER -t $TOPIC -o beginning
```

Debería ver las cargas JSON que representan los eventos de datos modificados generados en PostgreSQL como respuesta a las filas que acaba de agregar a la tabla `todos`. A continuación se muestra un fragmento de código de la carga:


```json
{
    "schema": {...},
    "payload": {
        "before": null,
        "after": {
            "id": 1,
            "description": "setup postgresql on azure",
            "todo_status": "complete"
        },
        "source": {
            "version": "1.2.0.Final",
            "connector": "postgresql",
            "name": "fullfillment",
            "ts_ms": 1593018069944,
            "snapshot": "last",
            "db": "postgres",
            "schema": "public",
            "table": "todos",
            "txId": 602,
            "lsn": 184579736,
            "xmin": null
        },
        "op": "c",
        "ts_ms": 1593018069947,
        "transaction": null
    }
```

El evento está formado por `payload` junto con su `schema` (se omite, para ser breves). En la sección `payload`, observe cómo se representa la operación de creación (`"op": "c"`); `"before": null` significa que se trata de una fila en la que se acaba de ejecutar `INSERT`, `after` proporciona valores para las columnas de la fila, `source` ofrece los metadatos de la instancia de PostgreSQL desde donde se obtuvo este evento, etc.

También puede probar lo mismo con las operaciones de actualización o eliminación e inspeccionar los eventos de datos modificados. Por ejemplo, para actualizar el estado de la tarea de `configure and install connector` (suponiendo que su `id` es `3`):

```sql
UPDATE todos SET todo_status = 'complete' WHERE id = 3;
```

## <a name="optional-install-filestreamsink-connector"></a>Instalación del conector de FileStreamSink (opcional)
Ahora que se están capturando todos los cambios de la tabla `todos` en el tema Event Hubs, usaremos el conector de FileStreamSink (que está disponible de forma predeterminada en Kafka Connect) para consumir estos eventos.

Cree un archivo de configuración (`file-sink-connector.json`) para el conector; reemplace el atributo `file` según su sistema de archivos.

```json
{
    "name": "cdc-file-sink",
    "config": {
        "connector.class": "org.apache.kafka.connect.file.FileStreamSinkConnector",
        "tasks.max": "1",
        "topics": "my-server.public.todos",
        "file": "<enter full path to file e.g. /Users/foo/todos-cdc.txt>"
    }
}
```

Para crear el conector y comprobar su estado:

```bash
curl -X POST -H "Content-Type: application/json" --data @file-sink-connector.json http://localhost:8083/connectors

curl http://localhost:8083/connectors/cdc-file-sink/status
```

Inserte, actualice o elimine registros de base de datos y supervise los registros en el archivo de receptor de salida configurado:

```bash
tail -f /Users/foo/todos-cdc.txt
```


## <a name="cleanup"></a>Limpieza
Kafka Connect crea temas de centro de eventos para almacenar las configuraciones, los desplazamientos y los estados que persisten incluso después de que el clúster de Connect se haya desactivado. A menos que se desee esta persistencia, se recomienda eliminar estos temas. También recomendamos eliminar el centro de eventos `my-server.public.todos` que se creó durante el transcurso de este tutorial.

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre Event Hubs para Kafka, vea los artículos siguientes:  

- [Reflejo de un agente de Kafka en un centro de eventos](event-hubs-kafka-mirror-maker-tutorial.md)
- [Conexión de Apache Spark a un centro de eventos](event-hubs-kafka-spark-tutorial.md)
- [Conexión de Apache Flink a un centro de eventos](event-hubs-kafka-flink-tutorial.md)
- [Exploración de ejemplos en nuestro GitHub](https://github.com/Azure/azure-event-hubs-for-kafka)
- [Conexión de Akka Streams a un centro de eventos](event-hubs-kafka-akka-streams-tutorial.md)
- [Guía del desarrollador de Apache Kafka para Azure Event Hubs](apache-kafka-developer-guide.md)
