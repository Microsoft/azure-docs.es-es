---
title: Escalado elástico con Cassandra API en Azure Cosmos DB
description: Conozca las opciones disponibles para escalar una cuenta de Cassandra API de Azure Cosmos DB y sus ventajas y desventajas
author: TheovanKraay
ms.service: cosmos-db
ms.topic: conceptual
ms.date: 07/29/2020
ms.author: thvankra
ms.openlocfilehash: 26f635525afea289e2e791b802478040a7851eee
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87486517"
---
# <a name="elastically-scale-an-azure-cosmos-db-cassandra-api-account"></a>Escalado elástico de una cuenta de Cassandra API de Azure Cosmos DB

Hay varias opciones para explorar la naturaleza elástica de la API de Azure Cosmos DB para Cassandra. Para comprender cómo realizar un escalado eficaz en Azure Cosmos DB, es importante comprender cómo aprovisionar la cantidad correcta de unidades de solicitud (RU/s) para tener en cuenta las demandas de rendimiento del sistema. Para más información sobre las unidades de solicitud, consulte el artículo [Unidades de solicitud](request-units.md). 

En Cassandra API, puede recuperar el cargo de la unidad de solicitud en consultas individuales mediante los SDK de [.NET y Java](https://docs.microsoft.com/azure/cosmos-db/find-request-unit-charge#cassandra-api). Esto resulta útil para determinar la cantidad de RU/s que deberá aprovisionar en el servicio.

:::image type="content" source="./media/request-units/request-units.png" alt-text="Unidades de solicitud consumidas por operaciones de base de datos" border="false":::

## <a name="handling-rate-limiting-429-errors"></a>Administración de la limitación de frecuencia (errores 429)

Azure Cosmos DB devolverá errores de frecuencia limitada (429) si los clientes consumen más recursos (RU/s) que la cantidad que se ha aprovisionado. Cassandra API de Azure Cosmos DB traslada estas excepciones como errores sobrecargados en el protocolo nativo de Cassandra. 

Si el sistema no es sensible a la latencia, puede que baste con administrar la limitación de frecuencia del rendimiento mediante reintentos. Consulte el [ejemplo de código de Java](https://github.com/Azure-Samples/azure-cosmos-cassandra-java-retry-sample) para saber cómo administrar la limitación de frecuencia de forma transparente mediante la [extensión de Azure Cosmos DB](https://github.com/Azure/azure-cosmos-cassandra-extensions) para la [directiva de reintentos de Cassandra](https://docs.datastax.com/en/developer/java-driver/4.4/manual/core/retries/) en Java. También puede usar la [extensión de Spark](https://mvnrepository.com/artifact/com.microsoft.azure.cosmosdb/azure-cosmos-cassandra-spark-helper) para administrar la limitación de frecuencia.

## <a name="manage-scaling"></a>Administración del escalado

Si necesita reducir la latencia, existe un espectro de opciones para administrar la escala y aprovisionar el rendimiento (RU) en Cassandra API:

* [Manualmente mediante Azure Portal](#use-azure-portal)
* [Mediante programación con las características del plano de control](#use-control-plane)
* [Mediante programación con comandos de CQL con un SDK específico](#use-cql-queries)
* [Dinámicamente con escalabilidad automática](#use-autoscale)

En las siguientes secciones se explican las ventajas y desventajas de cada enfoque. Luego, puede decidir cuál es la mejor estrategia para equilibrar las necesidades de escalado del sistema, el costo general y las necesidades de eficiencia de la solución.

## <a name="use-the-azure-portal"></a><a id="use-azure-portal"></a>Uso de Azure Portal

Puede escalar los recursos en la cuenta de Cassandra API de Azure Cosmos DB mediante Azure Portal. Para más información, consulte el artículo sobre el [aprovisionamiento del rendimiento en contenedores y bases de datos](set-throughput.md). En este artículo se explican las ventajas relativas de configurar el rendimiento en el nivel de [base de datos](set-throughput.md#set-throughput-on-a-database) o [contenedor](set-throughput.md#set-throughput-on-a-container) en Azure Portal. Los términos "base de datos" y "contenedor" que se mencionan en estos artículos se asignan a "espacio" y "tabla", respectivamente, en Cassandra API.

La ventaja de este método es que es una forma sencilla de administrar la capacidad de rendimiento de la base de datos. Sin embargo, el inconveniente es que, en muchos casos, el enfoque de escalado puede requerir que ciertos niveles de automatización sean tanto económicos como de alto rendimiento. En las secciones siguientes se explican los escenarios y métodos pertinentes.

## <a name="use-the-control-plane"></a><a id="use-control-plane"></a>Uso del plano de control

La API de Azure Cosmos DB para Cassandra ofrece la posibilidad de ajustar el rendimiento mediante programación con nuestras diversas características de plano de control. Puede encontrar instrucciones y ejemplos en los artículos sobre [Azure Resource Manager](manage-cassandra-with-resource-manager.md), [PowerShell](powershell-samples.md) y la [CLI de Azure](cli-samples.md).

La ventaja de este método es que se puede automatizar el escalado o la reducción vertical de los recursos en función de un temporizador para tener en cuenta la actividad máxima o los períodos de baja actividad. Consulte [aquí](https://github.com/Azure-Samples/azure-cosmos-throughput-scheduler) un ejemplo sobre cómo hacerlo con Azure Functions y PowerShell.

Uno de los inconvenientes de este enfoque podría ser que no puede responder a las necesidades cambiantes e imprevisibles de escalado en tiempo real. En su lugar, es posible que tenga que recurrir al contexto de la aplicación dentro del sistema o en el nivel de cliente o SDK, o bien utilizar [Escalabilidad automática](provision-throughput-autoscale.md).

## <a name="use-cql-queries-with-a-specific-sdk"></a><a id="use-cql-queries"></a>Uso de consultas de CQL con un SDK específico

Puede escalar el sistema de forma dinámica con código mediante la ejecución de los [comandos CQL ALTER](cassandra-support.md#keyspace-and-table-options) para la base de datos o el contenedor dados.

La ventaja de este enfoque es que permite responder a las necesidades de escalado de forma dinámica y de una manera personalizada que se adapte a su aplicación. Con este enfoque, todavía puede aprovechar los cargos y tarifas estándar de RU/s. Si las necesidades de escalado del sistema son en su mayoría predecibles (alrededor del 70 % o más), el uso del SDK con CQL puede ser un método más rentable de escalado automático que utilizar la escalabilidad automática. El inconveniente de este enfoque es que puede ser bastante complejo implementar los reintentos, al tiempo que la limitación de frecuencia puede aumentar la latencia.

## <a name="use-autoscale-provisioned-throughput"></a><a id="use-autoscale"></a>Uso del rendimiento aprovisionado de escalabilidad automática

Además de aprovisionar el rendimiento de forma estándar (manual) o mediante programación, puede configurar contenedores de Azure Cosmos en el rendimiento aprovisionado de escalabilidad automática. La escalabilidad automática escalará el sistema de forma automática e instantánea en función de las necesidades de consumo dentro de los intervalos de RU especificados sin poner en peligro los Acuerdos de Nivel de Servicio. Para más información, consulte este artículo sobre la [creación de contenedores y bases de datos de Azure Cosmos en escalabilidad automática](provision-throughput-autoscale.md).

La ventaja de este enfoque es que es la forma más fácil de administrar las necesidades de escalado del sistema. Garantiza que no se aplica limitación de frecuencia **dentro de los intervalos de RU configurados**. El inconveniente es que, si las necesidades de escalado del sistema son predecibles, la escalabilidad automática puede resultar menos rentable a la hora de administrar las necesidades de escalado que el uso del plano de control o el nivel de SDK mencionados anteriormente.

Para establecer o modificar el rendimiento máximo (RU) para el escalado automático con CQL, use lo siguiente (reemplazando el nombre de Keyspace o tabla según corresponda):

```Bash
# to set max throughput (RUs) for autoscale at keyspace level:
create keyspace <keyspace name> WITH cosmosdb_autoscale_max_throughput=5000;

# to alter max throughput (RUs) for autoscale at keyspace level:
alter keyspace <keyspace name> WITH cosmosdb_autoscale_max_throughput=4000;

# to set max throughput (RUs) for autoscale at table level:
create table <keyspace name>.<table name> (pk int PRIMARY KEY, ck int) WITH cosmosdb_autoscale_max_throughput=5000;

# to alter max throughput (RUs) for autoscale at table level:
alter table <keyspace name>.<table name> WITH cosmosdb_autoscale_max_throughput=4000;
```

## <a name="next-steps"></a>Pasos siguientes

- Introducción a la [creación de una cuenta de Cassandra API, una base de datos y una tabla](create-cassandra-api-account-java.md) usando una aplicación de Java
