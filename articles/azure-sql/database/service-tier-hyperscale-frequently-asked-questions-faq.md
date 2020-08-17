---
title: Preguntas más frecuentes sobre Hiperescala de Azure SQL Database
description: Da respuesta a las preguntas más habituales que los clientes formulan sobre una base de datos en SQL Database en el nivel de servicio Hiperescala, normalmente conocida como base de datos de hiperescala.
services: sql-database
ms.service: sql-database
ms.subservice: ''
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: conceptual
author: dimitri-furman
ms.author: dfurman
ms.reviewer: ''
ms.date: 03/03/2020
ms.openlocfilehash: 359de25d2bdb57ad5c6386586f987942acc120ef
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87500153"
---
# <a name="azure-sql-database-hyperscale-faq"></a>Preguntas más frecuentes sobre Hiperescala de Azure SQL Database
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

Este artículo da respuesta a las preguntas más frecuentes que los clientes formulan sobre las bases de datos de Azure SQL Database en el nivel de servicio Hiperescala, a la que se hará referencia solo como Hiperescala en el resto de este documento. En este artículo se describen los escenarios que admite Hiperescala y las características con las que es compatible.

- Esta sección de preguntas más frecuentes está pensada para lectores que tienen un conocimiento básico del nivel de servicio Hiperescala y que buscan una respuesta para sus preguntas y preocupaciones concretas.
- Esta sección no pretende ser una guía ni responder a preguntas sobre cómo usar una base de datos de Hiperescala. Para una introducción a Hiperescala, le recomendamos consultar la documentación sobre [Hiperescala de Azure SQL Database](service-tier-hyperscale.md).

## <a name="general-questions"></a>Preguntas generales

### <a name="what-is-a-hyperscale-database"></a>¿Qué es una base de datos de hiperescala?

Una base de datos de hiperescala es una base de datos de SQL Database en el nivel de servicio Hiperescala, que utiliza la tecnología de almacenamiento de escalabilidad horizontal de Hiperescala. Una base de datos de hiperescala admite hasta 100 TB de datos y proporciona un alto rendimiento, así como un rápido escalado para adaptarse a los requisitos de la carga de trabajo. El escalado es transparente para la aplicación: la conectividad, el procesamiento de consultas, etc., funcionan del mismo modo que cualquier otra base de datos de Azure SQL Database.

### <a name="what-resource-types-and-purchasing-models-support-hyperscale"></a>¿Qué tipos de recursos y modelos de compra admite Hiperescala?

El nivel de servicio Hiperescala solo está disponible para bases de datos únicas que usan el modelo de compra basado en núcleo virtual de Azure SQL Database.  

### <a name="how-does-the-hyperscale-service-tier-differ-from-the-general-purpose-and-business-critical-service-tiers"></a>Diferencias entre el nivel de servicio Hiperescala y los niveles Uso general y Crítico para la empresa

Los niveles de servicio basados en núcleos virtuales se diferencian en función de la disponibilidad de la base de datos y del tipo de almacenamiento, el rendimiento y el tamaño máximo, tal como se describe en la tabla siguiente.

| | Tipo de recurso | De uso general |  Hiperescala | Crítico para la empresa |
|:---:|:---:|:---:|:---:|:---:|
| **Más adecuado para** |All|Ofrece opciones de proceso y almacenamiento equilibradas adecuadas para un presupuesto limitado.|La mayoría de las cargas de trabajo empresariales. Escalado automático del tamaño de almacenamiento hasta 100 TB, escalado de procesos vertical y horizontal rápido, restauración rápida de bases de datos.|Aplicaciones de OLTP con una alta tasa de transacciones y latencia de E/S baja. Ofrece mayor resistencia a los errores y rapidez en las conmutaciones por error mediante varias réplicas actualizadas sincrónicamente.|
|  **Tipo de recurso** ||SQL Database / Instancia administrada de SQL | Base de datos única | SQL Database / Instancia administrada de SQL |
| **Tamaño de proceso**|SQL Database* | 1 a 80 núcleos virtuales | 1 a 80 núcleos virtuales* | 1 a 80 núcleos virtuales |
| **Tamaño de proceso**|Instancia administrada de SQL | 8, 16, 24, 32, 40, 64, 80 núcleos virtuales | N/D | 8, 16, 24, 32, 40, 64, 80 núcleos virtuales |
| **Tipo de almacenamiento** | All |Almacenamiento remoto Premium (por instancia) | Almacenamiento desacoplado con caché de SSD local (por instancia) | Almacenamiento SSD local extremadamente rápido (por instancia) |
| **Tamaño de almacenamiento** | SQL Database *| 5 GB – 4 TB | Hasta 100 TB | 5 GB – 4 TB |
| **Tamaño de almacenamiento** | Instancia administrada de SQL  | 32 GB–8 TB | N/D | 32 GB – 4 TB |
| **E/S** | Base de datos única | 500 IOPS por núcleo virtual con 7000 IOPS como máximo | Hiperescala es una arquitectura de varios niveles con almacenamiento en caché en varios niveles. Los IOPS efectivos dependen de la carga de trabajo. | 5000 IOPS hasta un máximo de 200 000 IOPS|
| **E/S** | Instancia administrada de SQL | Depende del tamaño de archivo | N/D | 1375 IOPS/núcleo virtual |
|**Disponibilidad**|All|1 réplica, sin escalado horizontal de lectura, sin caché local | Varias réplicas, hasta 4 escalados horizontales de lectura, caché local parcial | 3 replicas, 1 escalado horizontal de lectura, alta disponibilidad con redundancia de zona, caché local completa |
|**Copias de seguridad**|All|RA-GRS, retención de 7 a 35 días (7 días de manera predeterminada)| RA-GRS, retención de 7 días, recuperación a un momento dado (PITR) en un tiempo constante | RA-GRS, retención de 7 a 35 días (7 días de manera predeterminada) |

\* Los grupos elásticos no se admiten en el nivel de servicio Hiperescala

### <a name="who-should-use-the-hyperscale-service-tier"></a>Destinatarios del nivel de servicio Hiperescala

El nivel de servicio Hiperescala está pensado para clientes que tienen grandes bases de datos locales de SQL Server y quieren modernizar las aplicaciones mediante su traslado a la nube o para clientes que ya utilizan Azure SQL Database y desean ampliar significativamente las posibilidades de crecimiento de la base de datos. También está pensado para clientes que buscan un alto rendimiento y escalabilidad. Con Hiperescala, obtendrá:

- Tamaño de base de datos hasta 100 TB.
- Copias de seguridad rápidas de bases de datos, independientemente del tamaño de la base de datos (las copias de seguridad se basan en instantáneas de almacenamiento).
- Restauraciones rápidas de bases de datos, independientemente del tamaño de la base de datos (las restauraciones se basan en instantáneas de almacenamiento).
- Mayor rendimiento de registro independientemente del tamaño de la base de datos y del número de núcleos virtuales.
- Escalado horizontal con una o varias réplicas de solo lectura, que se usan para la descarga de lectura y como servidores en espera activa.
- Rápido escalado vertical de procesos, en tiempo constante, para que tenga más potencia para acomodar la pesada carga de trabajo, seguida de una reducción vertical, en tiempo constante. Esto se parece a un escalado y reducción vertical entre un nivel P6 y un nivel P11, por ejemplo, pero es mucho más rápido ya que esto no es un tamaño de operación de datos.

### <a name="what-regions-currently-support-hyperscale"></a>¿En qué regiones se admite actualmente Hiperescala?

El nivel de Hiperescala está disponible actualmente en las regiones indicadas en la [información general sobre el nivel de Hiperescala de Azure SQL Database](service-tier-hyperscale.md#regions).

### <a name="can-i-create-multiple-hyperscale-databases-per-server"></a>¿Puedo crear varias bases de datos de hiperescala por servidor?

Sí. Para obtener más información y conocer los límites en el número de bases de datos de hiperescala por servidor, consulte [Límites de recursos de SQL Database para bases de datos individuales y agrupadas en un servidor](resource-limits-logical-server.md).

### <a name="what-are-the-performance-characteristics-of-a-hyperscale-database"></a>¿Cuáles son las características de rendimiento de una base de datos de Hiperescala?

La arquitectura de Hiperescala proporciona un alto rendimiento al tiempo que admite tamaños de bases de datos grandes.

### <a name="what-is-the-scalability-of-a-hyperscale-database"></a>¿Cuál es la escalabilidad de una base de datos de hiperescala?

Hiperescala proporciona una rápida escalabilidad según la demanda de la carga de trabajo.

- **Escalado y reducción vertical**

  Con Hiperescala, puede escalar verticalmente el tamaño de proceso principal en términos de recursos como la CPU y la memoria y, posteriormente, reducir verticalmente, en tiempo constante. Dado que el almacenamiento se comparte, el escalado y reducción vertical no constituye un tamaño de operación de datos.  
- **Escalado y reducción horizontal**

  Con Hiperescala también tiene la posibilidad de aprovisionar una o varias réplicas de proceso adicionales que puede usar para atender las solicitudes de lectura. Esto significa que puede usar estas réplicas de proceso adicionales como réplicas de solo lectura para descargar la carga de trabajo de lectura del proceso principal. Además de solo lectura, estas réplicas también pueden funcionar como servidores en espera activa en caso de una conmutación por error de la réplica principal.

  El aprovisionamiento de cada una de estas réplicas de proceso adicionales se puede realizar en tiempo constante y se trata de una operación en línea. Puede conectarse a estas réplicas de proceso adicionales de solo lectura estableciendo el argumento `ApplicationIntent` de la cadena de conexión en `ReadOnly`. Todas las conexiones con la intención de aplicación `ReadOnly` se enrutan automáticamente a una de las réplicas de proceso de solo lectura adicionales.

## <a name="deep-dive-questions"></a>Preguntas de profundización

### <a name="can-i-mix-hyperscale-and-single-databases-in-a-single-server"></a>¿Puedo mezclar bases de datos de hiperescala y bases de datos únicas en un solo servidor?

Sí, puede hacerlo.

### <a name="does-hyperscale-require-my-application-programming-model-to-change"></a>¿Requiere Hiperescala que cambie mi modelo de programación de la aplicación?

No, el modelo de programación de la aplicación permanece tal cual. Puede usar la cadena de conexión y los demás modos normales como de costumbre para interactuar con su base de datos de Hiperescala.

### <a name="what-transaction-isolation-level-is-the-default-in-a-hyperscale-database"></a>¿Cuál es el nivel de aislamiento de transacción predeterminado en una base de datos de Hiperescala?

En la réplica principal, el nivel de aislamiento de transacción predeterminado es RCSI (aislamiento de instantánea de lectura confirmada). En las réplicas secundarias del escalado horizontal de lectura, el nivel de aislamiento predeterminado es Instantánea.

### <a name="can-i-bring-my-on-premises-or-iaas-sql-server-license-to-hyperscale"></a>¿Puedo traer mi licencia de SQL Server local o de IaaS a Hiperescala?

Sí, [Ventaja híbrida de Azure](https://azure.microsoft.com/pricing/hybrid-benefit/) está disponible para Hiperescala. Cada núcleo de SQL Server Standard se puede asignar a 1 núcleo virtual de Hiperescala. Cada núcleo de SQL Server Enterprise se puede asignar a 4 núcleos virtuales de Hiperescala. No necesita una licencia de SQL para las réplicas secundarias. El precio de la Ventaja híbrida de Azure se aplicará automáticamente a las réplicas de escalado horizontal de lectura (secundarias).

### <a name="what-kind-of-workloads-is-hyperscale-designed-for"></a>¿Para qué tipo de cargas de trabajo está diseñado el nivel de servicio Hiperescala?

Hiperescala es compatible con todas las cargas de trabajo de SQL Server, pero está optimizado principalmente para OLTP. Puede traer también cargas de trabajo híbridas (HTAP) y analíticas (data mart).

### <a name="how-can-i-choose-between-azure-sql-data-warehouse-and-azure-sql-database-hyperscale"></a>¿Cómo puedo elegir entre Azure SQL Data Warehouse e Hiperescala de Azure SQL Database?

Si actualmente está ejecutando consultas de análisis interactivas mediante SQL°Server como almacenamiento de datos, Hiperescala es una excelente opción porque se pueden hospedar almacenamientos de datos pequeños y medianos (desde unos pocos TB hasta 100 TB) a un menor costo y puede migrar sus cargas de trabajo del almacenamiento de datos de SQL Server a Hiperescala con el mínimo de cambios en el código T-SQL.

Si va a ejecutar análisis de datos a gran escala con consultas complejas y tasas de ingesta sostenidas que superen los 100 MB/s o mediante almacenamiento de datos paralelos (PDW), Teradata o cualquier otro almacenamiento de datos con procesamiento paralelo masivo (MPP), SQL Data Warehouse puede ser la mejor opción.
  
## <a name="hyperscale-compute-questions"></a>Preguntas sobre el proceso de Hiperescala

### <a name="can-i-pause-my-compute-at-any-time"></a>¿Puedo detener el proceso en cualquier momento?

No por el momento, pero puede reducir verticalmente el proceso y el número de réplicas para disminuir el coste en horas de poca actividad.

### <a name="can-i-provision-a-compute-replica-with-extra-ram-for-my-memory-intensive-workload"></a>¿Puedo aprovisionar una réplica de proceso con memoria RAM adicional para mi carga de trabajo con uso intensivo de memoria?

No. Para obtener más memoria RAM, deberá actualizar a un tamaño de proceso mayor. Para más información, consulte los [tamaños de almacenamiento y proceso de Hiperescala](resource-limits-vcore-single-databases.md#hyperscale---provisioned-compute---gen5).

### <a name="can-i-provision-multiple-compute-replicas-of-different-sizes"></a>¿Puedo aprovisionar varias réplicas de proceso de diferentes tamaños?

No.

### <a name="how-many-read-scale-out-replicas-are-supported"></a>¿Cuántas réplicas de escalado horizontal de lectura se admiten?

De manera predeterminada, las bases de datos de Hiperescala se crean con una réplica de escalado horizontal de lectura (dos réplicas, incluida la primaria). Puede escalar el número de réplicas de solo lectura entre 0 y 4 mediante [Azure Portal ](https://portal.azure.com) o la [API REST](https://docs.microsoft.com/rest/api/sql/databases/createorupdate).

### <a name="for-high-availability-do-i-need-to-provision-additional-compute-replicas"></a>¿Tengo que aprovisionar réplicas de proceso adicionales para lograr una alta disponibilidad?

En las bases de datos de Hiperescala, se proporciona resistencia de los datos en el nivel de almacenamiento. Solo necesita una réplica para proporcionar resistencia. Cuando la réplica de proceso está inactiva, se crea automáticamente una nueva réplica sin ninguna pérdida de datos.

Sin embargo, si hay solo una réplica, puede que tarde algún tiempo en generar la caché local en la nueva réplica después de la conmutación por error. Durante la fase de recompilación de la caché, la base de datos captura los datos directamente desde los servidores de páginas, lo que genera una mayor latencia de almacenamiento y un menor rendimiento de consultas.

En el caso de las aplicaciones críticas que requieren alta disponibilidad con un impacto mínimo de conmutación por error, debe aprovisionar al menos 2 réplicas de proceso, incluida la réplica de proceso primaria. Esta es la configuración predeterminada. De este modo, hay disponible una réplica en espera activa que actúa como destino de la conmutación por error.

## <a name="data-size-and-storage-questions"></a>Preguntas sobre el tamaño y el almacenamiento de datos

### <a name="what-is-the-maximum-database-size-supported-with-hyperscale"></a>¿Cuál es el tamaño máximo de base de datos compatible con Hiperescala?

100 TB.

### <a name="what-is-the-size-of-the-transaction-log-with-hyperscale"></a>¿Cuál es el tamaño del registro de transacciones con Hiperescala?

El registro de transacciones con Hiperescala es prácticamente infinito. No es necesario preocuparse por quedarse sin espacio en el registro en un sistema que tenga un alto rendimiento. No obstante, la velocidad de generación de registros se puede ver limitada por cargas de trabajo de escritura intensamente continuas. La velocidad máxima de generación de registros sostenidos es de 100 MB/s.

### <a name="does-my-tempdb-scale-as-my-database-grows"></a>¿Escala mi `tempdb` a medida que crece mi base de datos?

La base de datos `tempdb` está ubicada en el almacenamiento SSD local y tiene un tamaño proporcional al tamaño de proceso que aprovisione. La `tempdb` se ha optimizado para proporcionar ventajas de rendimiento máximo. No puede configurar el tamaño de `tempdb` porque no es usted quien lo administra.

### <a name="does-my-database-size-automatically-grow-or-do-i-have-to-manage-the-size-of-data-files"></a>¿Crece automáticamente el tamaño de mi base de datos o tengo que administrar el tamaño de los archivos de datos?

El tamaño de la base de datos crece automáticamente a medida que inserta o ingiere más datos.

### <a name="what-is-the-smallest-database-size-that-hyperscale-supports-or-starts-with"></a>¿Cuál es el tamaño de base de datos más pequeño que admite Hiperescala?

40 GB. Una base de datos de Hiperescala se crea con un tamaño inicial de 10 GB. Después, comienza a crecer 10 GB cada 10 minutos, hasta que alcanza el tamaño de 40 GB. Cada uno de estos bloques de 10 GB se asigna en un servidor de páginas diferente para proporcionar más IOPS y más paralelismo de E/S. Debido a esta optimización, incluso si elige un tamaño de base de datos inicial inferior a 40 GB, la base de datos crecerá automáticamente al menos 40 GB.

### <a name="in-what-increments-does-my-database-size-grow"></a>¿En cuantos GB se incrementa el tamaño de mi base de datos cada vez?

Cada archivo de datos crece en 10 GB. Varios archivos de datos pueden crecer al mismo tiempo.

### <a name="is-the-storage-in-hyperscale-local-or-remote"></a>¿El almacenamiento de Hiperescala es local o remoto?

En Hiperescala, los archivos de datos se almacenan en el almacenamiento estándar de Azure. Los datos se almacenan completamente en caché en el almacenamiento SSD local, en servidores de página que están cerca de las réplicas de proceso. Además, las réplicas de proceso tienen cachés de datos en el almacenamiento SSD local y en memoria, para disminuir la frecuencia de la captura de datos desde servidores de página remotos.

### <a name="can-i-manage-or-define-files-or-filegroups-with-hyperscale"></a>¿Puedo administrar o definir archivos o grupos de archivos con Hiperescala?

No. Los archivos de datos se agregan automáticamente. Las razones comunes para crear grupos de archivos adicionales no se aplican a la arquitectura de almacenamiento de Hiperescala.

### <a name="can-i-provision-a-hard-cap-on-the-data-growth-for-my-database"></a>¿Puedo aprovisionar un límite duro en el crecimiento de datos de mi base de datos?

No.

### <a name="how-are-data-files-laid-out-with-hyperscale"></a>¿Cómo se distribuyen los archivos de datos con Hiperescala?

Los archivos de datos se controlan mediante servidores de páginas, con un servidor de páginas por cada archivo de datos. A medida que crece el tamaño de los datos, se agregan archivos de datos y servidores de páginas asociados.

### <a name="is-database-shrink-supported"></a>¿Se admite la reducción de la base de datos?

No.

### <a name="is-data-compression-supported"></a>¿Se admite la compresión de datos?

Sí, incluida la compresión de fila, página y almacén de columnas.

### <a name="if-i-have-a-huge-table-does-my-table-data-get-spread-out-across-multiple-data-files"></a>Si tengo una tabla de gran tamaño, ¿se reparten los datos de la tabla entre varios archivos de datos?

Sí. Las páginas de datos asociadas a una determinada tabla pueden acabar en varios archivos de datos, los cuales forman todos parte del mismo grupo de archivos. SQL Server usa una [estrategia de relleno proporcional](https://docs.microsoft.com/sql/relational-databases/databases/database-files-and-filegroups#file-and-filegroup-fill-strategy) para distribuir los datos entre los archivos de datos.

## <a name="data-migration-questions"></a>Preguntas sobre migración de datos

### <a name="can-i-move-my-existing-databases-in-azure-sql-database-to-the-hyperscale-service-tier"></a>¿Puedo trasladar las bases de datos existentes de Azure SQL al nivel de servicio Hiperescala?

Sí. Puede trasladar las bases de datos existentes de Azure SQL a Hiperescala. Esta es una migración unidireccional. No puede trasladar las bases de datos del nivel de servicio Hiperescala a ningún otro. En el caso de las pruebas de concepto (POC), se recomienda hacer una copia de la base de datos y migrarla a Hiperescala. 

El tiempo necesario para mover una base de datos existente a Hiperescala consta del tiempo para copiar los datos y el tiempo para reproducir los cambios realizados en la base de datos de origen mientras se copian los datos. El tiempo de la copia de datos es proporcional al tamaño de los datos. El tiempo de reproducción de los cambios será más breve si el movimiento se realiza durante un período de escasa actividad de escritura.
  
### <a name="can-i-move-my-hyperscale-databases-to-other-service-tiers"></a>¿Puedo mover mis bases de datos de Hiperescala a otros niveles de servicio?

No. Actualmente, no se puede mover una base de datos de hiperescala a ningún otro nivel de servicio.

### <a name="do-i-lose-any-functionality-or-capabilities-after-migration-to-the-hyperscale-service-tier"></a>¿Pierdo alguna funcionalidad al migrar al nivel de servicio Hiperescala?

Sí. Algunas características de Azure SQL Database todavía no se admiten en Hiperescala, incluida la retención de copias de seguridad a largo plazo, entre otras. Después de migrar las bases de datos a Hiperescala, esas características dejan de funcionar.  Esperamos que estas limitaciones sean temporales.

### <a name="can-i-move-my-on-premises-sql-server-database-or-my-sql-server-database-in-a-cloud-virtual-machine-to-hyperscale"></a>¿Puedo mover mi base de datos de SQL Server local o mi base de datos de SQL Server en una máquina virtual de nube a Hiperescala?

Sí. Puede usar todas las tecnologías de migración existentes para migrar a Hiperescala, incluida la replicación transaccional y cualquier otra tecnología de movimiento de datos (la copia masiva, Azure Data Factory, Azure Databricks, SSIS). Consulte también [Azure Database Migration Service](../../dms/dms-overview.md), que admite muchos escenarios de migración.

### <a name="what-is-my-downtime-during-migration-from-an-on-premises-or-virtual-machine-environment-to-hyperscale-and-how-can-i-minimize-it"></a>¿Cuánto tiempo de inactividad experimentaré durante la migración de un entorno local o máquina virtual a Hiperescala y cómo puedo minimizarlo?

El tiempo de inactividad para la migración a Hiperescala es igual que el tiempo de inactividad al migrar las bases de datos a otros niveles de servicio de Azure SQL Database. Puede usar la [replicación transaccional](replication-to-sql-database.md#data-migration-scenario
) para minimizar el tiempo de inactividad de la migración para bases de datos de pocos TB de tamaño. Para bases de datos de gran tamaño (más de 10 TB), puede considerar la opción de migrar datos con ADF, Spark u otras tecnologías de movimiento de datos.

### <a name="how-much-time-would-it-take-to-bring-in-x-amount-of-data-to-hyperscale"></a>¿Cuánto tiempo se tardará en llevar una cantidad X de datos a Hiperescala?

Hiperescala es capaz de consumir 100 MB/s de datos nuevos o modificados, pero el tiempo necesario para trasladar los datos a las bases de datos de Azure SQL Database también se ve afectado por el rendimiento de la red disponible, la velocidad de lectura del origen y el objetivo de nivel de servicio de la base de datos de destino.

### <a name="can-i-read-data-from-blob-storage-and-do-fast-load-like-polybase-in-sql-data-warehouse"></a>¿Puedo leer datos de Blob Storage y realizar una carga rápida (como Polybase en SQL Data Warehouse)?

Puede hacer que una aplicación cliente lea datos de Azure Storage y cargue la carga de datos en una base de datos de Hiperescala (al igual que con cualquier otra base de datos de Azure SQL Database). Actualmente, Polybase no se admite en Azure SQL Database. Como alternativa a proporcionar una carga rápida, puede usar [Azure Data Factory ](https://docs.microsoft.com/azure/data-factory/) o usar un trabajo de Spark en [Azure Databricks](https://docs.microsoft.com/azure/azure-databricks/) con el [conector de Spark para SQL](spark-connector.md). Este conector admite la inserción masiva.

También es posible leer datos de forma masiva desde el almacén de blobs de Azure mediante BULK INSERT o OPENROWSET: [Ejemplos de acceso masivo a datos en Azure Blob Storage](https://docs.microsoft.com/sql/relational-databases/import-export/examples-of-bulk-access-to-data-in-azure-blob-storage?view=sql-server-2017#accessing-data-in-a-csv-file-referencing-an-azure-blob-storage-location).

El modelo de recuperación simple o de registro masivo no se admite en Hiperescala. Se requiere el modelo de recuperación completa para proporcionar alta disponibilidad y recuperación a un momento dado. Sin embargo, la arquitectura de registro de Hiperescala proporciona una mejor tasa de ingesta de datos en comparación con otros niveles de servicio de Azure SQL Database.

### <a name="does-hyperscale-allow-provisioning-multiple-nodes-for-parallel-ingesting-of-large-amounts-of-data"></a>¿Permite Hiperescala el aprovisionamiento de varios nodos para la ingesta paralela de grandes cantidades de datos?

No. Hiperescala es una arquitectura de procesamiento múltiple simétrico (SMP) y no un procesamiento paralelo masivo (MPP) ni una arquitectura multimaestro. Solo se pueden crear varias réplicas para escalar horizontalmente cargas de trabajo de solo lectura.

### <a name="what-is-the-oldest-sql-server-version-supported-for-migration-to-hyperscale"></a>¿Cuál es la versión de SQL Server más antigua compatible con la migración a Hiperescala?

SQL Server 2005. Para más información, consulte [Migración a una base de datos única o a una base de datos agrupada](migrate-to-database-from-sql-server.md#migrate-to-a-single-database-or-a-pooled-database). Para más información sobre problemas de compatibilidad, consulte [Solución de problemas de compatibilidad de migración de bases de datos](migrate-to-database-from-sql-server.md#resolving-database-migration-compatibility-issues).

### <a name="does-hyperscale-support-migration-from-other-data-sources-such-as-amazon-aurora-mysql-postgresql-oracle-db2-and-other-database-platforms"></a>¿Admite Hiperescala la migración desde otros orígenes de datos como Amazon Aurora, MySQL, PostgreSQL, Oracle, DB2 y otras plataformas de base de datos?

Sí. [Azure Database Migration Service](../../dms/dms-overview.md) admite muchos escenarios de migración.

## <a name="business-continuity-and-disaster-recovery-questions"></a>Preguntas sobre recuperación ante desastres y continuidad empresarial

### <a name="what-slas-are-provided-for-a-hyperscale-database"></a>¿Cuáles son los contratos de nivel de servicio que se proporcionan para una base de datos de Hiperescala?

Consulte [Contrato de nivel de servicio para Azure SQL Database](https://azure.microsoft.com/support/legal/sla/sql-database/v1_4/). Las réplicas de proceso secundarias adicionales aumentan la disponibilidad, hasta un 99,99 % para una base de datos con dos o más réplicas de proceso secundarias.

### <a name="are-the-database-backups-managed-for-me-by-azure-sql-database"></a>¿Administra Azure SQL Database las copias de seguridad de las bases de datos en mi lugar?

Sí.

### <a name="how-often-are-the-database-backups-taken"></a>¿Con qué frecuencia se realizan las copias de seguridad de las bases de datos?

No hay copias de seguridad de registros tradicionales, completas y diferenciales para las bases de datos de Hiperescala. En su lugar, hay instantáneas de almacenamiento normales de archivos de datos. El registro que se genera simplemente se conserva tal cual durante el período de retención configurado, lo que permite la restauración a cualquier punto en el tiempo dentro del período de retención.

### <a name="does-hyperscale-support-point-in-time-restore"></a>¿Admite Hiperescala la restauración a un momento dado?

Sí.

### <a name="what-is-the-recovery-point-objective-rporecovery-time-objective-rto-for-database-restore-in-hyperscale"></a>¿Cuál es el objetivo de punto de recuperación (RPO) y el objetivo de tiempo de recuperación (RTO) para la restauración de base de datos en Hiperescala?

El RPO es 0 minutos. El objetivo de RTO es inferior a 10 minutos, sin importar el tamaño de la base de datos.

### <a name="does-database-backup-affect-compute-performance-on-my-primary-or-secondary-replicas"></a>¿La copia de seguridad de bases de datos afecta al rendimiento de proceso en mis réplicas principales o secundarias?

No. El subsistema de almacenamiento administra las copias de seguridad y estas aprovechan las instantáneas de almacenamiento. No afectan la carga de trabajo del usuario.

### <a name="can-i-perform-geo-restore-with-a-hyperscale-database"></a>¿Puedo realizar una restauración geográfica con una base de datos de Hiperescala?

Sí. La restauración geográfica es totalmente compatible. A diferencia de la restauración a un momento dado, la restauración geográfica requiere una operación de tamaño de datos. Los archivos de datos se copian en paralelo, por lo que la duración de esta operación depende sobre todo del tamaño del archivo más grande en la base de datos, en lugar del tamaño total de la base de datos. Los tiempos de la restauración geográfica se reducirán de manera significativa si la base de datos se restaura en la región de Azure que está [emparejada](https://docs.microsoft.com/azure/best-practices-availability-paired-regions) con la región de la base de datos de origen.

### <a name="can-i-set-up-geo-replication-with-hyperscale-database"></a>¿Puedo configurar la replicación geográfica con la base de datos de Hiperescala?

De momento, no.

### <a name="can-i-take-a-hyperscale-database-backup-and-restore-it-to-my-on-premises-server-or-on-sql-server-in-a-vm"></a>¿Puedo realizar una copia de seguridad de una base de datos de Hiperescala y restaurarla en mi servidor local o en SQL Server en una máquina virtual?

No. El formato de almacenamiento de las bases de datos de Hiperescala es diferente de cualquier versión de SQL Server y no podría controlar las copias de seguridad ni tener acceso a ellas. Para sacar los datos de una base de datos de Hiperescala, puede extraer datos mediante cualquier tecnología de movimiento de datos, es decir, Azure Data Factory, Azure Databricks, SSIS, etc.

## <a name="cross-feature-questions"></a>Preguntas sobre las distintas características

### <a name="do-i-lose-any-functionality-or-capabilities-after-migration-to-the-hyperscale-service-tier"></a>¿Pierdo alguna funcionalidad al migrar al nivel de servicio Hiperescala?

Sí. Algunas características de Azure SQL Database no se admiten en Hiperescala, incluida la retención de copias de seguridad a largo plazo, entre otras. Después de migrar las bases de datos a Hiperescala, esas características dejan de funcionar.

### <a name="will-polybase-work-with-hyperscale"></a>¿Funcionará Polybase con Hiperescala?

No. Polybase no se admite en Azure SQL Database.

### <a name="does-hyperscale-have-support-for-r-and-python"></a>¿Hiperescala es compatible con R y Python?

De momento, no.

### <a name="are-compute-nodes-containerized"></a>¿Están los nodos de proceso en contenedores?

No. Los procesos de Hiperescala se ejecutan en nodos (VM) de [Service Fabric](https://azure.microsoft.com/services/service-fabric/), no en contenedores.

## <a name="performance-questions"></a>Preguntas sobre rendimiento

### <a name="how-much-write-throughput-can-i-push-in-a-hyperscale-database"></a>¿Cuánto rendimiento de escritura puedo introducir en una base de datos de Hiperescala?

El límite de rendimiento del registro de transacciones se establece en 100 MB/s para cualquier tamaño de proceso de Hiperescala. La capacidad de lograr esta tasa depende de varios factores, entre los que se incluyen, entre otros, el tipo de carga de trabajo, la configuración de cliente y tener la capacidad de proceso suficiente en la réplica de proceso principal para producir el registro a esta velocidad.

### <a name="how-many-iops-do-i-get-on-the-largest-compute"></a>¿Cuántas IOPS obtengo en el proceso más grande?

Las IOPS y la latencia de E/S variarán según los patrones de carga de trabajo. Si los datos a los que se accede se almacenan en caché en la réplica de proceso, verá un rendimiento similar de E/S que con el almacenamiento SSD local.

### <a name="does-my-throughput-get-affected-by-backups"></a>¿Afectan las copias de seguridad a mi rendimiento?

No. El proceso no está acoplado con la capa de almacenamiento. Esto elimina el impacto en el rendimiento de la copia de seguridad.

### <a name="does-my-throughput-get-affected-as-i-provision-additional-compute-replicas"></a>¿Se ve afectado el rendimiento cuando aprovisiono réplicas de proceso adicionales?

Dado que el almacenamiento se comparte y no hay ninguna replicación física directa que tenga lugar entre la réplica de proceso principal y las secundarias, el rendimiento de la réplica principal no se verá afectado directamente al agregar réplicas secundarias. Sin embargo, podemos limitar las cargas de trabajo de escritura intensivas continuas en la principal para permitir que la aplicación del registro en las réplicas secundarias y en los servidores de páginas se ponga al día y evitar un rendimiento de lectura insuficiente en las réplicas secundarias.

### <a name="how-do-i-diagnose-and-troubleshoot-performance-problems-in-a-hyperscale-database"></a>¿Cómo diagnostico y soluciono los problemas de rendimiento en una base de datos de Hiperescala?

Para la mayoría de los problemas de rendimiento, especialmente los que no tienen el origen en el rendimiento del almacenamiento, se aplican los pasos de diagnóstico y solución de problemas habituales de SQL. Para obtener información sobre el diagnóstico de almacenamiento específico de Hiperescala, vea [Diagnóstico de la solución de problemas de rendimiento de Hiperescala de SQL](hyperscale-performance-diagnostics.md).

## <a name="scalability-questions"></a>Preguntas sobre escalabilidad

### <a name="how-long-would-it-take-to-scale-up-and-down-a-compute-replica"></a>¿Cuánto tiempo se tarda en realizar un escalado y reducción vertical de una réplica de proceso?

El proceso de escalado o reducción vertical tarda hasta 2 minutos, independientemente del tamaño de los datos.

### <a name="is-my-database-offline-while-the-scaling-updown-operation-is-in-progress"></a>¿Está mi base de datos sin conexión mientras que la operación de escalado y reducción vertical está en curso?

No. El escalado y reducción vertical se realiza en línea.

### <a name="should-i-expect-connection-drop-when-the-scaling-operations-are-in-progress"></a>¿Se puede esperar una interrupción de la conexión cuando las operaciones de escalado están en curso?

El escalado o la reducción vertical hace que las conexiones existentes se bloqueen cuando se produce una conmutación por error al final de la operación de escalado. La incorporación de réplicas secundarias no genera interrupciones de conexión.

### <a name="is-the-scaling-up-and-down-of-compute-replicas-automatic-or-end-user-triggered-operation"></a>¿Se produce el escalado y reducción vertical de las réplicas de proceso de forma automática o se trata de una operación desencadenada por el usuario final?

Por el usuario final. No es una operación automática.  

### <a name="does-the-size-of-my-tempdb-database-also-grow-as-the-compute-is-scaled-up"></a>¿El tamaño de la base de datos de `tempdb` también aumenta a medida que el proceso se escala verticalmente?

Sí. La base de datos `tempdb` se escalará verticalmente de forma automática a medida que crezca el proceso.  

### <a name="can-i-provision-multiple-primary-compute-replicas-such-as-a-multi-master-system-where-multiple-primary-compute-heads-can-drive-a-higher-level-of-concurrency"></a>¿Puedo aprovisionar varias réplicas de proceso principales como un sistema de arquitectura multimaestro en los que los nodos principales pueden impulsar un mayor nivel de simultaneidad?

No. Solo la réplica de proceso principal acepta solicitudes de lectura y escritura. Las réplicas de proceso secundarias aceptan únicamente solicitudes de solo lectura.

## <a name="read-scale-out-questions"></a>Preguntas sobre el escalado horizontal de lectura

### <a name="how-many-secondary-compute-replicas-can-i-provision"></a>¿Cuántas réplicas de proceso secundarias puedo aprovisionar?

De manera predeterminada, se crea una réplica secundaria para las bases de datos de Hiperescala. Si quiere ajustar el número de réplicas, puede hacerlo a través de [Azure Portal](https://portal.azure.com) o la [API REST](https://docs.microsoft.com/rest/api/sql/databases/createorupdate).

### <a name="how-do-i-connect-to-these-secondary-compute-replicas"></a>¿Cómo me conecto a estas réplicas de proceso secundarias?

Puede conectarse a estas réplicas de proceso adicionales de solo lectura estableciendo el argumento `ApplicationIntent` de la cadena de conexión en `ReadOnly`. Todas las conexiones marcadas con `ReadOnly` se enrutan automáticamente a uno de las réplicas de proceso adicionales de solo lectura.  

### <a name="how-do-i-validate-if-i-have-successfully-connected-to-secondary-compute-replica-using-ssms-or-other-client-tools"></a>¿Cómo puedo validar si me he conectado correctamente a la réplica de proceso secundaria mediante SSMS u otras herramientas de cliente?

Puede ejecutar la siguiente consulta de Transact-SQL: `SELECT DATABASEPROPERTYEX ('<database_name>', 'Updateability')`.
El resultado es `READ_ONLY` si está conectado a una réplica secundaria de solo lectura y `READ_WRITE`, si está conectado a la réplica principal. Tenga en cuenta que el contexto de la base de datos debe establecerse en el nombre de la base de datos de Hiperescala, no en la base de datos de `master`.

### <a name="can-i-create-a-dedicated-endpoint-for-a-read-scale-out-replica"></a>¿Puedo crear un punto de conexión dedicado para la réplica de escalado horizontal de lectura?

No. Solo puede conectarse a la réplica de escalado horizontal de lectura si especifica `ApplicationIntent=ReadOnly`.

### <a name="does-the-system-do-intelligent-load-balancing-of-the-read-workload"></a>¿Realiza el sistema un equilibrio de carga inteligente de la carga de trabajo de lectura?

No. Una nueva conexión con intención de solo lectura se redirige a una réplica de escalado horizontal de lectura arbitraria.

### <a name="can-i-scale-updown-the-secondary-compute-replicas-independently-of-the-primary-replica"></a>¿Se puede realizar un escalado y reducción vertical de las réplicas de proceso secundarias independientemente de la réplica principal?

No. La réplica de proceso secundaria también se usa como destinos de conmutación por error de alta disponibilidad, por lo que deben tener la misma configuración que la principal para proporcionar el rendimiento esperado después de la conmutación por error.

### <a name="do-i-get-different-tempdb-sizing-for-my-primary-compute-and-my-additional-secondary-compute-replicas"></a>¿Puedo obtener un tamaño de `tempdb` diferente para mi réplica de proceso principal y las secundarias adicionales?

No. La base de datos `tempdb` está configurada según el aprovisionamiento del tamaño de proceso y las réplicas de proceso secundarias son del mismo tamaño que la principal.

### <a name="can-i-add-indexes-and-views-on-my-secondary-compute-replicas"></a>¿Puedo agregar índices y vistas en mis réplicas de proceso secundarias?

No. Las bases de datos de Hiperescala tienen el almacenamiento compartido, lo que significa que todas las réplicas de proceso ven las mismas tablas, índices y vistas. Si desea índices adicionales optimizados para lecturas en la réplica secundaria, debe primero agregarlos a la réplica principal.

### <a name="how-much-delay-is-there-going-to-be-between-the-primary-and-secondary-compute-replicas"></a>¿Cuánto retraso va a haber entre la réplica de proceso principal y las secundarias?

La latencia de los datos desde el momento en que se confirma una transacción en la réplica principal hasta el momento en que es visible en la secundaria depende de la velocidad de generación de registro actual. La latencia de datos típica es de pocos milisegundos.

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información sobre el nivel de servicio Hiperescala, vea [Nivel de servicio Hiperescala](service-tier-hyperscale.md).
