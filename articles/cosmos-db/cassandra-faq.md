---
title: Preguntas frecuentes sobre Cassandra API para Azure Cosmos DB
description: Obtenga respuestas a las preguntas frecuentes sobre Cassandra API para Azure Cosmos DB.
author: TheovanKraay
ms.service: cosmos-db
ms.topic: conceptual
ms.date: 08/12/2020
ms.author: thvankra
ms.openlocfilehash: b327c0786fb07488fd8863272598dbffe19bfe07
ms.sourcegitcommit: c28fc1ec7d90f7e8b2e8775f5a250dd14a1622a6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/13/2020
ms.locfileid: "88167613"
---
# <a name="frequently-asked-questions-about-the-cassandra-api-in-azure-cosmos-db"></a>Preguntas más frecuentes sobre Cassandra API en Azure Cosmos DB

En este artículo se describen las diferencias de funcionalidad entre Apache Cassandra y Cassandra API en Azure Cosmos DB. Además incluye respuestas a preguntas más frecuentes sobre Cassandra API en Azure Cosmos DB.

## <a name="key-differences-between-apache-cassandra-and-the-cassandra-api"></a>Diferencias clave entre Apache Cassandra y Cassandra API

- Apache Cassandra recomienda un límite de 100 MB para el tamaño de una clave de partición. Cassandra API para Azure Cosmos DB permite hasta 20 GB por partición.
- Apache Cassandra le permite deshabilitar las confirmaciones duraderas. Puede omitir la escritura en el registro de confirmaciones e ir directamente a las memtables. Esto puede dar lugar a la pérdida de datos si el nodo deja de funcionar antes de que las memtables se vacíen en SSTables en el disco. Azure Cosmos DB siempre realiza confirmaciones duraderas para evitar la pérdida de datos.
- Apache Cassandra puede experimentar un rendimiento reducido si la carga de trabajo implica muchos reemplazos o eliminaciones. El motivo se debe a los marcadores de exclusión que la carga de trabajo de lectura debe omitir para capturar los datos más recientes. Cassandra API no experimentará un rendimiento de lectura reducido cuando la carga de trabajo tenga muchos reemplazos o eliminaciones.
- Durante los escenarios de cargas de trabajo de gran reemplazo, la compactación debe ejecutarse para combinar SSTables en el disco. (Se necesita una combinación porque las escrituras de Apache Cassandra solo se anexan. Varias actualizaciones se almacenan como entradas SSTable individuales que deben combinarse periódicamente). Esta situación también puede provocar un rendimiento de lectura reducido durante la compactación. Este impacto en el rendimiento no se produce en Cassandra API porque la API no implementa la compactación.
- Con Apache Cassandra, es posible establecer un factor de replicación de 1. Sin embargo, esto da lugar a una disponibilidad baja si el único nodo con datos deja de funcionar. Esto no supone un problema con Cassandra API para Azure Cosmos DB porque siempre hay un factor de replicación de 4 (cuórum de 3).
- La adición o eliminación de nodos en Apache Cassandra requiere intervención manual, junto con un elevado uso de CPU en el nuevo nodo, mientras que los nodos existentes mueven algunos de sus intervalos de tokens al nuevo nodo. Esta situación es la misma cuando retira un nodo existente. Sin embargo, Cassandra API se escala horizontalmente sin que se observe ningún problema en el servicio o la aplicación.
- No es necesario establecer **num_tokens** en cada nodo del clúster como en Apache Cassandra. Azure Cosmos DB administra totalmente los nodos y los intervalos de tokens.
- Cassandra API está totalmente administrada. No necesita los comandos **nodetool**, como repair y decommission, que se usan en Apache Cassandra.

## <a name="other-frequently-asked-questions"></a>Otras preguntas frecuentes

### <a name="what-protocol-version-does-the-cassandra-api-support"></a>¿Qué versión de protocolo admite Cassandra API?

Cassandra API para Azure Cosmos DB admite CQL, versión 3.x. La compatibilidad con CQL se basa en el [repositorio público de GitHub de Apache Cassandra](https://github.com/apache/cassandra/blob/trunk/doc/cql3/CQL.textile). Si tiene comentarios sobre la compatibilidad con otros protocolos, háganoslo saber a través de [comentarios de UserVoice](https://feedback.azure.com/forums/263030-azure-cosmos-db) o envíe un correo electrónico a [askcosmosdbcassandra@microsoft.com](mailto:askcosmosdbcassandra@microsoft.com).

### <a name="why-is-choosing-throughput-for-a-table-a-requirement"></a>¿Por qué es un requisito elegir un rendimiento para una tabla?

Azure Cosmos DB establece el rendimiento predeterminado para el contenedor en función de dónde crear la tabla: Azure Portal o CQL.

Azure Cosmos DB ofrece garantías de rendimiento y latencia con límites superiores en las operaciones. Estas garantías son posibles cuando un motor puede aplicar la gobernanza de las operaciones del inquilino. Al establecer el rendimiento, se asegura de obtener el rendimiento y la latencia garantizados, ya que la plataforma reserva esta capacidad y asegura el éxito operativo.
Puede [cambiar con total flexibilidad el rendimiento](manage-scale-cassandra.md) para beneficiarse de la estacionalidad de su aplicación y ahorrar costos.

El concepto de rendimiento se explica en el artículo [Unidades de solicitud en Azure Cosmos DB](request-units.md). El rendimiento de una tabla se distribuye equitativamente entre las particiones físicas subyacentes.

### <a name="what-is-the-throughput-of-a-table-thats-created-through-cql"></a>¿Cuál es el rendimiento de una tabla que se crea mediante CQL?

Azure Cosmos DB utiliza Unidades de solicitud por segundo (RU/s) como medida para proporcionar el rendimiento. Las tablas creadas a través de CQL tienen 400 RU de manera predeterminada. Puede cambiar la RU en Azure Portal.

CQL

```shell
CREATE TABLE keyspaceName.tablename (user_id int PRIMARY KEY, lastname text) WITH cosmosdb_provisioned_throughput=1200
```

.NET

```csharp
int provisionedThroughput = 400;
var simpleStatement = new SimpleStatement($"CREATE TABLE {keyspaceName}.{tableName} (user_id int PRIMARY KEY, lastname text)");
var outgoingPayload = new Dictionary<string, byte[]>();
outgoingPayload["cosmosdb_provisioned_throughput"] = Encoding.UTF8.GetBytes(provisionedThroughput.ToString());
simpleStatement.SetOutgoingPayload(outgoingPayload);
```

### <a name="what-happens-when-throughput-is-used-up"></a>¿Qué ocurre cuando se usa el rendimiento?

Azure Cosmos DB ofrece garantías de rendimiento y latencia con límites superiores en las operaciones. Estas garantías son posibles cuando un motor puede aplicar la gobernanza de las operaciones del inquilino. Al establecer el rendimiento, se asegura de obtener el rendimiento y la latencia garantizados, ya que la plataforma reserva esta capacidad y asegura el éxito operativo.

Al superar esta capacidad, obtiene el siguiente mensaje de error que indica que se agotó la capacidad:

**0x1001 Sobrecargado: No se puede procesar la solicitud porque "La tasa de solicitudes es grande".** 

Es esencial ver qué operaciones ocasionan este problema (y su volumen). Puede hacerse una idea sobre el exceso de capacidad consumida que supera la aprovisionada con las métricas de Azure Portal. A continuación, debe asegurarse de que se consume la capacidad prácticamente igual en todas las particiones subyacentes. Si ve que una partición consume la mayor parte del rendimiento, tiene un sesgo de la carga de trabajo.

Hay disponibles métricas que muestran cómo se usa el rendimiento en horas, días y cada siete días, entre particiones o en total. Para más información, consulte [Supervisión y depuración con métricas de Azure Cosmos DB](use-metrics.md).

Los registros de diagnóstico se explican en el artículo [Registro de diagnóstico de Azure Cosmos DB](logging.md).

### <a name="does-the-primary-key-map-to-the-partition-key-concept-of-azure-cosmos-db"></a>¿Se corresponde la clave principal con el concepto de clave de partición de Azure Cosmos DB?

Sí, la clave de partición se utiliza para colocar la entidad en la ubicación correcta. En Azure Cosmos DB, se usa para encontrar una partición lógica apropiada que esté almacenada en una partición física. El concepto de creación de particiones también se explica en el artículo [Partición y escalado en Azure Cosmos DB](partition-data.md). La lección esencial de esto es que una partición lógica no debe superar el límite de 20 GB.

### <a name="what-happens-when-i-get-a-notification-that-a-partition-is-full"></a>¿Qué ocurre cuando recibo una notificación que indica que una partición está llena?

Azure Cosmos DB es un sistema basado en acuerdos de nivel de servicio (SLA). Ofrece un escalado ilimitado, con garantías de latencia, rendimiento, disponibilidad y coherencia. Este almacenamiento ilimitado se basa en la escalabilidad horizontal de los datos mediante la creación de particiones como concepto clave. El concepto de creación de particiones también se explica en el artículo [Partición y escalado en Azure Cosmos DB](partition-data.md).

Debe cumplir con el límite de 20 GB en el número de entidades o elementos por partición lógica. Para asegurarse de que la aplicación se escala correctamente incluso con Azure Storage, le pedimos que *no* cree una partición muy activa almacenando toda la información en una partición y realizando consultas en ella. Este error solo puede producirse si los datos están sesgados; es decir, tiene una gran cantidad de datos para una clave de partición (más de 20 GB). Puede encontrar la distribución de datos mediante el portal de almacenamiento. La manera de corregir este error consiste en volver a crear la tabla y elegir una (clave de partición) principal granular, lo que permite una mejor distribución de datos.

### <a name="can-i-use-the-cassandra-api-as-a-key-value-store-with-millions-or-billions-of-partition-keys"></a>¿Es posible usar Cassandra API como almacén de valores clave con millones o miles de millones de claves de partición?

Azure Cosmos DB puede almacenar datos ilimitados mediante el escalado horizontal del almacenamiento. Este almacenamiento es independiente del rendimiento. Sí, puede usar siempre Cassandra API para simplemente almacenar y recuperar las claves y los valores especificando la clave de partición y principal correcta. Estas claves individuales obtienen su propia partición lógica y se colocan encima de la partición física sin problemas.

### <a name="can-i-create-more-than-one-table-with-the-cassandra-api"></a>¿Puedo crear más de una tabla con Cassandra API?

Sí, es posible crear más de una tabla con Cassandra API. Cada una de esas tablas se trata como unidad de rendimiento y almacenamiento.

### <a name="can-i-create-more-than-one-table-in-succession"></a>¿Puedo crear más de una tabla en sucesión?

Azure Cosmos DB es un sistema regulado por recursos tanto para las actividades de plano de control como de datos. Los contenedores, como las colecciones y las tablas, son entidades de runtime que se han aprovisionado para una capacidad de rendimiento dada. La creación de estos contenedores en sucesión rápida no es una actividad esperada y puede estar limitada. Si tiene pruebas que eliminan o crean tablas inmediatamente, intente espaciarlas.

### <a name="what-is-the-maximum-number-of-tables-that-i-can-create"></a>¿Cuál es el número máximo de tablas que puedo crear?

No hay ningún límite físico en el número de tablas. Si tiene un gran número de tablas (donde el tamaño constante total supera los 10 TB de datos) que deben crearse, no las decenas ni centenas habituales, envíe un correo electrónico a [askcosmosdbcassandra@microsoft.com](mailto:askcosmosdbcassandra@microsoft.com).

### <a name="what-is-the-maximum-number-of-keyspaces-that-i-can-create"></a>¿Cuál es el número máximo de espacios de claves que puedo crear?

No hay ningún límite físico en el número de espacios de claves, ya que son contenedores de metadatos. Si tiene un gran número de espacios de claves, envíe un correo electrónico a [askcosmosdbcassandra@microsoft.com](mailto:askcosmosdbcassandra@microsoft.com).

### <a name="can-i-bring-in-a-lot-of-data-after-starting-from-a-normal-table"></a>¿Puedo tener una gran cantidad de datos tras comenzar con una tabla normal?

Sí. En particiones distribuidas uniformemente, la capacidad de almacenamiento se administra automáticamente y aumenta a medida que inserta más datos en ella. Así, puede importar con confianza tantos datos como sea necesario sin administrar ni aprovisionar los nodos y mucho más. Pero si prevé un gran crecimiento inmediato de datos, tiene más sentido [aprovisionar directamente para el rendimiento previsto](set-throughput.md) en lugar de empezar con menos y aumentar inmediatamente.

### <a name="can-i-use-yaml-file-settings-to-configure-api-behavior"></a>¿Puedo usar la configuración de archivos de YAML para configurar el comportamiento de la API?

Cassandra API para Azure Cosmos DB proporciona compatibilidad de nivel de protocolo para la ejecución de operaciones. Oculta la complejidad de la administración, supervisión y configuración. Como desarrollador y usuario, no tiene que preocuparse por la disponibilidad, los marcadores de exclusión, la memoria caché de claves, la memoria caché de filas, los filtros de Bloom y muchas otras opciones. Cassandra API se centra en proporcionar el rendimiento de lectura y escritura que necesite, sin la sobrecarga de la configuración y administración.

### <a name="will-the-cassandra-api-support-node-addition-cluster-status-and-node-status-commands"></a>¿Admitirá Cassandra API los comandos de estado de nodo, estado de clústeres y adición de nodos?

Cassandra API simplifica el planeamiento de capacidad y la respuesta a las exigencias de elasticidad en el rendimiento y el almacenamiento. Con Azure Cosmos DB, aprovisiona el rendimiento que necesita. A continuación, puede escalar o reducir verticalmente la cantidad de veces que necesite a lo largo del día, sin preocuparse de agregar o eliminar ni administrar nodos. No es necesario usar herramientas para la administración de nodos y clústeres.

### <a name="what-happens-with-various-configuration-settings-for-keyspace-creation-like-simplenetwork"></a>¿Qué ocurre con los distintos valores de configuración para la creación de espacios de claves, como una red o un elemento sencillo?

Azure Cosmos DB proporciona una distribución global directamente por motivos de disponibilidad y baja latencia. No es necesario configurar réplicas ni nada más. Las escrituras se confirman siempre mediante cuórum duradero en cualquier región donde escriba, a la vez que proporcionan garantías de rendimiento.

### <a name="what-happens-with-various-settings-for-table-metadata"></a>¿Qué ocurre con las distintas opciones de los metadatos de una tabla?

Azure Cosmos DB proporciona garantías de rendimiento para lecturas, escrituras y rendimiento. Así que no tiene que preocuparse por tocar ninguna de las opciones de configuración y manipularlas por accidente. Estas opciones incluyen filtro de Bloom, almacenamiento en caché, oportunidad de reparación de lectura, gc_grace y compresión memtable_flush_period.

### <a name="is-time-to-live-supported-for-cassandra-tables"></a>¿Se admite el período de vida en las tablas de Cassandra?

Sí, TTL se admite.

### <a name="how-can-i-monitor-infrastructure-along-with-throughput"></a>¿Cómo puedo supervisar la infraestructura junto con el rendimiento?

Azure Cosmos DB es un servicio de plataforma que le ayuda a aumentar la productividad y no preocuparse de la administración y supervisión de la infraestructura. Por ejemplo, no es necesario supervisar los parámetros anteriores de estado de nodo, estado de réplica, gc y sistema operativo con varias herramientas. Solo debe vigilar el rendimiento que está disponible en las métricas del portal para ver si está siendo limitado, y aumentar o disminuir el rendimiento. Puede:

- Supervisar los [SLA](monitor-accounts.md)
- Usar [métricas](use-metrics.md)
- Usar [registros de diagnóstico](logging.md)

### <a name="which-client-sdks-can-work-with-the-cassandra-api"></a>¿Qué SDK de cliente pueden trabajar con Cassandra API?

Los controladores del cliente del SDK de Apache Cassandra que usan CQLv3 se utilizaban para los programas cliente. Si usa otros controladores o si está teniendo problemas, envíe un correo electrónico a [askcosmosdbcassandra@microsoft.com](mailto:askcosmosdbcassandra@microsoft.com).

### <a name="are-composite-partition-keys-supported"></a>¿Se admiten claves de partición compuestas?

Sí, puede usar la sintaxis normal para crear claves de partición compuestas.

### <a name="can-i-use-sstableloader-for-data-loading"></a>¿Puedo usar sstableloader para cargar los datos?

No, sstableloader no se admite.

### <a name="can-i-pair-an-on-premises-apache-cassandra-cluster-with-the-cassandra-api"></a>¿Puedo emparejar un clúster de Apache Cassandra local con Cassandra API?

En la actualidad, Azure Cosmos DB tiene una experiencia optimizada para el entorno en la nube sin sobrecarga de las operaciones. Si requiere del emparejamiento, envíe un correo electrónico a [askcosmosdbcassandra@microsoft.com](mailto:askcosmosdbcassandra@microsoft.com) con una descripción de su escenario. Estamos trabajando en una oferta para ayudar a emparejar el clúster de Cassandra local o en la nube con Cassandra API para Azure Cosmos DB.

### <a name="does-the-cassandra-api-provide-full-backups"></a>¿Proporciona Cassandra API copias de seguridad completas?

Azure Cosmos DB ofrece dos copias de seguridad completas gratis realizadas en intervalos de cuatro horas entre todas las API. Por lo tanto, no es necesario configurar una programación de copias de seguridad. 

Si desea modificar la retención y la frecuencia, envíe un correo electrónico a [askcosmosdbcassandra@microsoft.com](mailto:askcosmosdbcassandra@microsoft.com) o genere una incidencia de soporte técnico. En el artículo [Copias de seguridad y restauración automáticas en línea con Azure Cosmos DB](online-backup-and-restore.md), se proporciona información sobre la funcionalidad de copia de seguridad.

### <a name="how-does-the-cassandra-api-account-handle-failover-if-a-region-goes-down"></a>¿Cómo trata la cuenta de la API de Cassandra la conmutación por error en caso de que una región deje de funcionar?

Cassandra API adopta la plataforma distribuida globalmente de Azure Cosmos DB. Para garantizar que la aplicación pueda tolerar un tiempo de inactividad del centro de datos, habilite al menos una región adicional para la cuenta en Azure Portal. Para obtener más información, consulte [Alta disponibilidad con Azure Cosmos DB](high-availability.md).

Puede agregar todas las regiones que quiera para la cuenta y controlar adónde puede conmutar por error especificando una prioridad. Para usar la base de datos, debe especificar también una aplicación allí. Al hacerlo, los clientes no experimentarán tiempo de inactividad.

### <a name="does-the-cassandra-api-index-all-attributes-of-an-entity-by-default"></a>¿Indexa Cassandra API todos los atributos de una entidad de forma predeterminada?

No. Cassandra API admite [índices secundarios](cassandra-secondary-index.md), que tienen un comportamiento similar al que tienen en Apache Cassandra. La API no indexa cada atributo de forma predeterminada.  


### <a name="can-i-use-the-new-cassandra-api-sdk-locally-with-the-emulator"></a>¿Se puede usar el nuevo SDK de la API de Cassandra localmente con el emulador?

Sí, este procedimiento se admite. Puede encontrar información sobre cómo habilitar esta opción en el artículo [Uso del emulador de Azure Cosmos para desarrollo y pruebas locales](local-emulator.md#cassandra-api).


### <a name="how-can-i-migrate-data-from-apache-cassandra-clusters-to-azure-cosmos-db"></a>¿Cómo puedo migrar datos de clústeres de Apache Cassandra a Azure Cosmos DB?

Puede leer acerca de las opciones de migración en el tutorial [Migración de los datos a una cuenta de Cassandra API en Azure Cosmos DB](cassandra-import-data.md).


### <a name="where-can-i-give-feedback-on-cassandra-api-features"></a>¿Dónde puedo enviar comentarios sobre las características de Cassandra API?

Proporcione sus comentarios a través de [comentarios de UserVoice](https://feedback.azure.com/forums/263030-azure-cosmos-db).

[azure-portal]: https://portal.azure.com
[query]: sql-api-sql-query.md

## <a name="next-steps"></a>Pasos siguientes

- Introducción al [escalado elástico de una cuenta de Cassandra API de Azure Cosmos DB](manage-scale-cassandra.md).
