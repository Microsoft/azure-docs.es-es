---
title: Preguntas frecuentes sobre Azure Synapse Link para Azure Cosmos DB
description: Obtenga respuestas a las preguntas más frecuentes sobre Synapse Link para Azure Cosmos DB en áreas como la facturación, el almacén analítico, la seguridad y el período de vida en el almacén analítico.
author: Rodrigossz
ms.author: rosouz
ms.service: cosmos-db
ms.topic: conceptual
ms.date: 05/19/2020
ms.openlocfilehash: e65838074260a4d4a9c9668fa6ac4aa2a403929b
ms.sourcegitcommit: bfeae16fa5db56c1ec1fe75e0597d8194522b396
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/10/2020
ms.locfileid: "88037565"
---
# <a name="frequently-asked-questions-about-azure-synapse-link-for-azure-cosmos-db"></a>Preguntas frecuentes sobre Azure Synapse Link para Azure Cosmos DB

Azure Synapse Link para Azure Cosmos DB crea una integración sin contratiempos entre Azure Cosmos DB y Azure Synapse Analytics. Permite a los clientes ejecutar análisis casi en tiempo real de sus datos operativos junto con aislamiento de rendimiento completo de sus cargas de trabajo transaccionales y sin necesidad de una canalización de ETL. En este artículo se responden las preguntas más frecuentes sobre Synapse Link para Azure Cosmos DB.

## <a name="general-faq"></a>Preguntas más frecuentes generales

### <a name="is-synapse-link-supported-for-all-azure-cosmos-db-apis"></a>¿Synapse Link es compatible con todas las API de Azure Cosmos DB?
En la versión preliminar pública, Synapse Link solo es compatible con SQL (Core) API de Azure Cosmos DB. La compatibilidad con la API de Cosmos DB para MongoDB y Cassandra API se encuentra actualmente en versión preliminar validada.  Para solicitar acceso a la versión preliminar validada, póngase en contacto con el [equipo de Azure Cosmos DB](mailto:cosmosdbsynapselink@microsoft.com).

### <a name="is-synapse-link-supported-for-multi-region-azure-cosmos-accounts"></a>¿Synapse Link es compatible con las cuentas de Azure Cosmos para varias regiones?
Sí, en el caso de las cuentas de Azure Cosmos para varias regiones, los datos almacenados en el almacén analítico también se distribuyen de forma global. Independientemente de si tienen una región de escritura única (un único maestro) o varias regiones de escritura (también conocida como arquitectura multimaestro), las consultas analíticas realizadas desde Azure Synapse Analytics se pueden proporcionar desde la región local más cercana.

Al planear la configuración de una cuenta de Azure Cosmos para varias regiones compatible con el almacén analítico, se recomienda que se agreguen todas las regiones necesarias durante la creación de la cuenta.

### <a name="can-i-choose-to-enable-synapse-link-for-only-certain-region-and-not-all-regions-in-a-multi-region-account-set-up"></a>¿Puedo habilitar Synapse Link solo para regiones determinadas y no para todas las regiones de una configuración de cuenta de varias regiones?
En la versión preliminar, cuando Synapse Link está habilitado para una cuenta de varias regiones, el almacén analítico se crea en todas las regiones. Los datos subyacentes se optimizan para el rendimiento y la coherencia transaccional en el almacén transaccional.

### <a name="is-backup-and-restore-supported-for-synapse-link-enabled-accounts"></a>¿Se admite la copia de seguridad y restauración para las cuentas habilitadas para Synapse Link?
En la versión preliminar, para las cuentas de bases de datos habilitadas para Synapse Link, no se admite la copia de seguridad y restauración de contenedores. Si tiene cargas de trabajo de producción que requieren la funcionalidad de copia de seguridad y restauración, se recomienda no habilitar Synapse Link en esas cuentas de bases de datos. 

### <a name="can-i-disable-the-synapse-link-feature-for-my-azure-cosmos-account"></a>¿Puedo deshabilitar la característica Synapse Link para mi cuenta de Azure Cosmos?
En este momento, después de habilitar la funcionalidad Synapse Link en el nivel de cuenta, no se puede deshabilitar.  Si quiere desactivarla, debe eliminar la cuenta y volver a crear una nueva de Azure Cosmos.

Tenga en cuenta que **no** conlleva ningún costo que la capacidad Synapse Link esté habilitada en el nivel de cuenta, aunque no haya ningún contenedor habilitado para el almacén analítico.

## <a name="azure-cosmos-db-analytical-store"></a>Almacén analítico de Azure Cosmos DB

### <a name="can-i-enable-analytical-store-on-existing-containers"></a>¿Puedo habilitar el almacén analítico en contenedores existentes?
En este momento, solo se puede habilitar el almacén analítico para los contenedores nuevos (tanto en cuentas nuevas como ya existentes).

### <a name="can-i-disable-analytical-store-on-my-azure-cosmos-containers-after-enabling-it-during-container-creation"></a>¿Puedo deshabilitar el almacén analítico en mis contenedores de Azure Cosmos después de haberlo habilitado durante la creación del contenedor?
En este momento, el almacén analítico no se puede deshabilitar en un contenedor de Azure Cosmos después de haberlo habilitado durante la creación del contenedor.

### <a name="is-analytical-store-supported-for-azure-cosmos-containers-with-autoscale-provisioned-throughput"></a>¿El almacén analítico es compatible con los contenedores de Azure Cosmos que tienen rendimiento aprovisionado de escalabilidad automática?
Sí, el almacén analítico se puede habilitar en los contenedores con rendimiento aprovisionado de escalabilidad automática.

### <a name="is-there-any-effect-on-azure-cosmos-db-transactional-store-provisioned-rus"></a>¿Tiene algún efecto en las RU aprovisionadas por el almacén transaccional de Azure Cosmos DB?
Azure Cosmos DB garantiza aislamiento del rendimiento entre las cargas de trabajo transaccionales y analíticas. La habilitación del almacén analítico en un contenedor no afectará a las RU/s aprovisionadas en el almacén transaccional de Azure Cosmos DB. Las transacciones (lectura y escritura) y los costos de almacenamiento del almacén analítico se cobrarán por separado. Consulte los [precios del almacén analítico de Azure Cosmos DB](analytical-store-introduction.md#analytical-store-pricing) para más información.

### <a name="are-delete-and-update-operations-on-the-transactional-store-reflected-in-the-analytical-store"></a>¿Las operaciones de eliminación y actualización del almacén transaccional se reflejan en el almacén analítico? 
Sí, las eliminaciones y actualizaciones de datos en el almacén transaccional se reflejarán en el almacén analítico. Puede configurar el período de vida (TTL) en el contenedor para que incluya datos históricos. De este modo, el almacén analítico conservará todas las versiones de los elementos que cumplan los criterios del TTL analítico. Consulte la [información general sobre TTL analítico](analytical-store-introduction.md#analytical-ttl) para obtener más detalles.

### <a name="can-i-connect-to-analytical-store-from-analytics-engines-other-than-azure-synapse-analytics"></a>¿Puedo conectarme al almacén analítico desde motores de análisis distintos de Azure Synapse Analytics?
Solo puede acceder a las consultas en el almacén analítico y ejecutarlas mediante los diversos entornos de ejecución que proporciona Azure Synapse Analytics. El almacén analítico se puede consultar y analizar mediante:

* Spark en Synapse con compatibilidad completa con Scala, Python, SparkSQL y C#. Spark en Synapse es fundamental para los escenarios de ciencia e ingeniería de datos.
* SQL sin servidor con lenguaje T-SQL y compatibilidad con herramientas de inteligencia empresarial conocidas (por ejemplo, Power BI Premium, etc.).

### <a name="can-i-connect-to-analytical-store-from-synapse-sql-provisioned"></a>¿Puedo conectarme al almacén analítico desde Synapse SQL aprovisionado?
En este momento, no se puede acceder al almacén analítico desde Synapse SQL aprovisionado.

### <a name="can-i-write-back-the-query-aggregation-results-from-synapse-back-to-the-analytical-store"></a>¿Puedo reescribir los resultados de la agregación de consultas de Synapse en el almacén analítico?
El almacén analítico es un almacén de solo lectura de un contenedor de Azure Cosmos. Por lo tanto, no puede volver a escribir directamente los resultados de la agregación en el almacén analítico, pero puede escribirlos en el almacén transaccional de Azure Cosmos DB de otro contenedor, que posteriormente se puede aprovechar como capa de servicio.

### <a name="is-the-autosync-replication-from-transactional-store-to-the-analytical-store-asynchronous-or-synchronous-and-what-are-the-latencies"></a>¿La replicación de sincronización automática del almacén transaccional al almacén analítico es asincrónica o sincrónica, y cuáles son las latencias? 
La replicación es asincrónica y actualmente la latencia esperada es de aproximadamente 2 minutos.

### <a name="are-there-any-scenarios-where-the-items-from-the-transactional-store-are-not-automatically-propagated-to-the-analytical-store"></a>¿Hay algún escenario en el que los elementos del almacén transaccional no se propaguen automáticamente al almacén analítico?
Si los elementos específicos del contenedor infringen el [esquema bien definido para análisis](analytical-store-introduction.md#analytical-schema), no se incluirán en el almacén analítico. Si tiene escenarios bloqueados por un esquema bien definido para análisis, envíe un correo electrónico al [equipo de Azure Cosmos DB](mailto:cosmosdbsynapselink@microsoft.com) para obtener ayuda.

### <a name="can-i-partition-the-data-in-analytical-store-differently-from-transactional-store"></a>¿Se pueden crear particiones de los datos en el almacén analítico de forma distinta a la del almacén transaccional?
Las particiones de los datos del almacén analítico se crean en función de las particiones horizontales en el almacén transaccional. Actualmente, no se puede elegir una estrategia de creación de particiones diferente para el almacén analítico.

### <a name="can-i-customize-or-override-the-way-transactional-data-is-transformed-into-columnar-format-in-the-analytical-store"></a>¿Puedo personalizar o invalidar la manera en que los datos transaccionales se transforman al formato de columnas en el almacén analítico?
Actualmente no se pueden transformar los elementos de datos cuando se propagan de forma automática del almacén transaccional al almacén analítico. Si tiene escenarios bloqueados por esta limitación, envíe un correo electrónico al [equipo de Azure Cosmos DB](mailto:cosmosdbsynapselink@microsoft.com).

## <a name="analytical-time-to-live-ttl"></a>Período de vida (TTL) analítico

### <a name="is-ttl-for-analytical-data-supported-at-both-container-and-item-level"></a>¿El TTL de los datos analíticos se admite en los niveles de contenedor y elemento?
En este momento, el TTL de los datos analíticos solo se puede configurar en el nivel de contenedor y no hay compatibilidad para establecerlo en el nivel de elemento.

### <a name="after-setting-the-container-level--analytical-ttl-on-an-azure-cosmos-db-container-can-i-change-to-a-different-value-later"></a>Después de establecer el TTL analítico de nivel de contenedor en un contenedor de Azure Cosmos DB, ¿puedo cambiar a un valor diferente más adelante?
Sí, el TTL analítico puede actualizarse a cualquier valor válido. Consulte el artículo [TTL analítico](analytical-store-introduction.md#analytical-ttl) para obtener más detalles sobre el TTL analítico.

### <a name="can-i-update-or-delete-an-item-from-the-analytical-store-after-it-has-been-ttld-out-from-the-transactional-store"></a>¿Se puede actualizar o eliminar un elemento del almacén analítico una vez que se ha agotado el TTL del almacén transaccional?
Todas las actualizaciones y eliminaciones de transacciones se copian en el almacén analítico pero, si el elemento se ha purgado del almacén transaccional, no se podrá actualizar en el almacén analítico. Para obtener más información, consulte el artículo [TTL analítico](analytical-store-introduction.md#analytical-ttl).

## <a name="billing"></a>Facturación

### <a name="what-is-the-billing-model-of-synapse-link-for-azure-cosmos-db"></a>¿Cuál es el modelo de facturación de Synapse Link para Azure Cosmos DB?
El [almacén analítico de Azure Cosmos DB](analytical-store-introduction.md) está disponible en versión preliminar pública sin cargos por el almacén analítico hasta el 30 de agosto de 2020. Spark en Synapse y Synapse SQL se facturan según el [consumo del servicio Synapse](https://azure.microsoft.com/pricing/details/synapse-analytics/).

## <a name="security"></a>Seguridad

### <a name="what-are-the-ways-to-authenticate-with-the-analytical-store"></a>¿De qué maneras hay para autenticarse con el almacén analítico?
La autenticación con el almacén analítico es igual que en un almacén transaccional. En una base de datos determinada, puede autenticarse con la clave maestra o de solo lectura. Puede aprovechar el servicio vinculado en Synapse Studio para evitar pegar las claves de Azure Cosmos DB en los cuadernos de Spark. El acceso a este servicio vinculado está disponible para todos los usuarios que tienen acceso al área de trabajo.

## <a name="synapse-run-times"></a>Tiempos de ejecución de Synapse

### <a name="what-are-the-currently-supported-synapse-run-times-to-access-azure-cosmos-db-analytical-store"></a>¿Cuáles son los tiempos de ejecución de Synapse admitidos actualmente para acceder al almacén analítico de Azure Cosmos DB?

|Tiempo de ejecución de Synapse |Compatibilidad actual |
|---------|---------|
|Grupos de Spark en Synapse | Lectura, escritura (a través del almacén transaccional), tablas, vista temporal |
|Synapse SQL sin servidor    | Lectura, vista (versión preliminar validada)  |
|Synapse SQL aprovisionado   |  No disponible |

### <a name="do-my-synapse-spark-tables-sync-with-my-synapse-sql-serverless-tables-the-same-way-they-do-with-azure-data-lake"></a>¿Las tablas de Spark en Synapse se sincronizan con las tablas de Synapse SQL sin servidor de la misma forma que con Azure Data Lake?
En este momento, esta característica no está disponible.

### <a name="can-i-do-spark-structured-streaming-from-analytical-store"></a>¿Puedo realizar un flujo estructurado de Spark desde el almacén analítico?
En este momento, la compatibilidad con el flujo estructurado de Spark para Azure Cosmos DB está implementada mediante la funcionalidad de fuente de cambios del almacén transaccional y aún no se admite desde el almacén analítico.

## <a name="synapse-studio"></a>Synapse Studio

### <a name="in-the-synapse-studio-how-do-i-recognize-if-im-connected-to-an-azure-cosmos-db-container-with-the-analytics-store-enabled"></a>En Synapse Studio, ¿cómo puedo reconocer si estoy conectado a un contenedor de Azure Cosmos DB con el almacén analítico habilitado?
Un contenedor de Azure Cosmos DB habilitado con el almacén analítico tiene el siguiente icono:

:::image type="content" source="./media/synapse-link-frequently-asked-questions/analytical-store-icon.png" alt-text="Icono: contenedor de Azure Cosmos DB habilitado con el almacén analítico":::

Un contenedor de almacén transaccional se representará con el siguiente icono:

:::image type="content" source="./media/synapse-link-frequently-asked-questions/transactional-store-icon.png" alt-text="Icono: contenedor de Azure Cosmos DB habilitado con el almacén transaccional":::
 
### <a name="how-do-you-pass-azure-cosmos-db-credentials-from-synapse-studio"></a>¿Cómo se pasan las credenciales de Azure Cosmos DB desde Synapse Studio?
En este momento, las credenciales de Azure Cosmos DB se pasan durante la creación del servicio que ha vinculado el usuario con acceso a las bases de datos de Azure Cosmos DB. El acceso a ese almacén está disponible para otros usuarios que tienen acceso al área de trabajo.

## <a name="next-steps"></a>Pasos siguientes

* Más información sobre las [ventajas de Synapse Link](synapse-link.md#synapse-link-benefits).

* Obtenga información sobre la [integración entre Synapse Link y Azure Cosmos DB](synapse-link.md#synapse-link-integration).
