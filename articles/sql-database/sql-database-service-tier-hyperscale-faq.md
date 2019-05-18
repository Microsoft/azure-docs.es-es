---
title: Preguntas más frecuentes sobre Hiperescala de Azure SQL Database | Microsoft Docs
description: Da respuesta a las preguntas más habituales que los clientes formulan sobre las bases de datos de Azure SQL en el nivel de servicio Hiperescala, normalmente conocidas como bases de datos de hiperescala.
services: sql-database
ms.service: sql-database
ms.subservice: ''
ms.custom: ''
ms.devlang: ''
ms.topic: conceptual
author: stevestein
ms.author: sstein
ms.reviewer: ''
manager: craigg
ms.date: 05/06/2019
ms.openlocfilehash: 38d9ad007b67756bdca0c6f98267aa16ba38ee9d
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65791426"
---
# <a name="faq-about-azure-sql-hyperscale-databases"></a>Preguntas más frecuentes sobre las bases de datos de hiperescala de Azure SQL

En este artículo proporciona respuestas a las preguntas más frecuentes para clientes que están considerando una base de datos en el nivel de servicio de Azure SQL Database a gran escala, suele denominado una base de datos a gran escala. En este artículo se describen los escenarios que admiten Hiperescala y los servicios transversales que son compatibles con Hiperescala de SQL Database en general.

- Esta sección de preguntas más frecuentes está pensada para lectores que tienen un conocimiento básico del nivel de servicio Hiperescala y que buscan una respuesta para sus preguntas y preocupaciones concretas.
- Esta sección no pretende ser una guía ni responder a preguntas sobre cómo usar una base de datos de hiperescala de SQL Database. Para ello, se recomienda que consulte la documentación sobre [Hiperescala de Azure SQL Database](sql-database-service-tier-hyperscale.md).

## <a name="general-questions"></a>Preguntas generales

### <a name="what-is-a-hyperscale-database"></a>¿Qué es una base de datos de hiperescala?

Una base de datos de hiperescala es una base de datos de Azure SQL en el nivel de servicio Hiperescala, que utiliza la tecnología de almacenamiento de escalabilidad horizontal de Hiperescala. Una base de datos de hiperescala admite hasta 100 TB de datos y proporciona un alto rendimiento, así como un rápido escalado para adaptarse a los requisitos de la carga de trabajo. El escalado es transparente para la aplicación: la conectividad, el procesamiento de consultas, etc, es igual que cualquier otra base de datos SQL.

### <a name="what-resource-types-and-purchasing-models-support-hyperscale"></a>¿Qué tipos de recursos y modelos de compra admite Hiperescala?

El nivel de servicio Hiperescala solo está disponible para bases de datos únicas que usan el modelo de compra basado en núcleo virtual de Azure SQL Database.  

### <a name="how-does-the-hyperscale-service-tier-differ-from-the-general-purpose-and-business-critical-service-tiers"></a>Diferencias entre el nivel de servicio Hiperescala y los niveles Uso general y Crítico para la empresa

Los niveles de servicio basados en núcleos virtuales se diferencian principalmente según la disponibilidad, el tipo de almacenamiento y el número de IOPS.

- El nivel de servicio De uso general resulta adecuado para la mayoría de las cargas de trabajo empresariales ya que ofrece un conjunto equilibrado de opciones de proceso y almacenamiento en los que la latencia de E/S o los tiempos de conmutación por error no son la prioridad.
- El nivel de servicio Hiperescala está optimizado para cargas de trabajo con bases de datos de gran tamaño.
- El nivel de servicio Crítico para la empresa es adecuado para cargas de trabajo en las que la latencia de E/S es una prioridad.

| | Tipo de recurso | Uso general |  Hiperescala | Crítico para la empresa |
|:---|:---:|:---:|:---:|:---:|:---:|
| **Más adecuado para** |Todo|  La mayoría de las cargas de trabajo empresariales. Ofrece opciones de proceso y almacenamiento equilibradas adecuadas para un presupuesto limitado. | Aplicaciones de datos con requisitos de gran capacidad de datos y la posibilidad de escalar automáticamente el almacenamiento y de escalar los procesos con fluidez. | Aplicaciones de OLTP con una alta tasa de transacciones y la latencia de E/S más baja. Ofrece la máxima resistencia a errores mediante varias réplicas aisladas.|
|  **Tipo de recurso** ||Base de datos única / grupo elástico / instancia administrada | Base de datos única | Base de datos única / grupo elástico / instancia administrada |
| **Tamaño de proceso**|Base de datos única / grupo elástico * | 1 a 80 núcleos virtuales | 1 a 80 núcleos virtuales* | 1 a 80 núcleos virtuales |
| |Instancia administrada | 8, 16, 24, 32, 40, 64, 80 núcleos virtuales | N/D | 8, 16, 24, 32, 40, 64, 80 núcleos virtuales |
| **Tipo de almacenamiento** | Todo |Almacenamiento remoto Premium (por instancia) | Almacenamiento desacoplado con caché de SSD local (por instancia) | Almacenamiento SSD local extremadamente rápido (por instancia) |
| **Tamaño de almacenamiento** | Base de datos única / grupo elástico | 5 GB – 4 TB | Hasta 100 TB | 5 GB – 4 TB |
| | Instancia administrada  | 32 GB–8 TB | N/D | 32 GB – 4 TB |
| **Rendimiento de E/S** | Base de datos única** | 500 IOPS por núcleo virtual con 7000 IOPS como máximo | Desconocido todavía | 5000 IOPS hasta un máximo de 200 000 IOPS|
| | Instancia administrada | Depende del tamaño del archivo | N/D | Instancia administrada: Depende del tamaño del archivo|
|**Disponibilidad**|Todo|1 réplica, sin escalado de lectura, sin caché local | Varias réplicas, hasta el 15 escalados de lectura, caché local parcial | 3 replicas, 1 escalado de lectura, alta disponibilidad con redundancia de zona, caché local completa |
|**Copias de seguridad**|Todo|RA-GRS, de 7 a 35 días (7 días de forma predeterminada)| RA-GRS, 7 a 35 días (7 días de forma predeterminada), recuperación en el momento de tiempo constante (PITR) | RA-GRS, de 7 a 35 días (7 días de forma predeterminada) |

\* Los grupos elásticos no se admiten en el nivel de servicio Hiperescala

### <a name="who-should-use-the-hyperscale-service-tier"></a>Destinatarios del nivel de servicio Hiperescala

El nivel de servicio Hiperescala está pensado principalmente para clientes que tienen grandes bases de datos locales de SQL Server y quieren modernizar sus aplicaciones mediante su traslado a la nube, o para clientes que ya están utilizando Azure SQL Database y desean ampliar significativamente las posibilidades de crecimiento de la base de datos. También está pensado para clientes que buscan un alto rendimiento y escalabilidad. Con Hiperescala, obtendrá:

- Compatibilidad con bases de datos con un tamaño de hasta 100 TB
- Copias de seguridad rápidas de bases de datos, independientemente del tamaño de la base de datos (las copias de seguridad se basan en instantáneas de archivos)
- Restauraciones rápidas de bases de datos, independientemente del tamaño de la base de datos (las restauraciones se basan en instantáneas de archivos)
- Un mayor rendimiento de registro da como resultado unos tiempos de confirmación de transacciones más rápidos independientemente del tamaño de la base de datos
- Un escalado horizontal de lectura o más nodos de solo lectura para descargar la carga de trabajo de lectura y para las esperas activas.
- Rápido escalado vertical de procesos, en tiempo constante, para que tenga más potencia para acomodar la pesada carga de trabajo, seguida de una reducción vertical, en tiempo constante. Esto se parece a un escalado y reducción vertical entre un nivel P6 y P11, por ejemplo, pero es mucho más rápido ya que esto no es un tamaño de operación de datos.

### <a name="what-regions-currently-support-hyperscale"></a>¿En qué regiones se admite actualmente Hiperescala?

El nivel de hiperescala de base de datos de SQL Azure está disponible actualmente en las regiones incluidas en [información general de Azure SQL Database a gran escala](sql-database-service-tier-hyperscale.md#regions).

### <a name="can-i-create-multiple-hyperscale-databases-per-logical-server"></a>¿Puedo crear varias bases de datos de hiperescala por servidor lógico?

Sí. Para más información y para conocer los límites en el número de bases de datos de hiperescala por servidor lógico, consulte [Límites de recursos de SQL Database para bases de datos individuales y agrupadas en un servidor lógico](sql-database-resource-limits-logical-server.md).

### <a name="what-are-the-performance-characteristic-of-a-hyperscale-database"></a>¿Cuáles son las características de rendimiento de una base de datos de hiperescala?

La arquitectura de Hiperescala de SQL Database proporciona un alto rendimiento al tiempo que admite tamaños de bases de datos grandes. 

### <a name="what-is-the-scalability-of-a-hyperscale-database"></a>¿Cuál es la escalabilidad de una base de datos de hiperescala?

Hiperescala de SQL Database proporciona una rápida escalabilidad según la demanda de la carga de trabajo.

- **Escalado y reducción vertical**

  Con Hiperescala, puede escalar verticalmente el tamaño de proceso principal en términos de recursos como la CPU o la memoria y, posteriormente, reducir verticalmente, en tiempo constante. Dado que el almacenamiento se comparte, el escalado y reducción vertical no constituye un tamaño de operación de datos.  
- **Escalado y reducción horizontal**

  Con Hiperescala también tiene la posibilidad de aprovisionar uno o varios nodos de proceso adicionales que puede usar para atender las solicitudes de lectura. Esto significa que puede usar estos nodos de proceso adicionales como nodos de solo lectura para descargar la carga de trabajo de lectura del proceso principal. Además de solo lectura, estos nodos también pueden funcionar como nodos de espera activa en caso de una conmutación por error del nodo principal.

  El aprovisionamiento de cada uno de estos nodos de proceso adicionales se puede realizar en tiempo constante y se trata de una operación en línea. Puede conectarse a estos nodos de proceso adicionales de solo lectura estableciendo el argumento `ApplicationIntent` de la cadena de conexión en `read_only`. Todas las conexiones marcadas con `read-only` se enrutan automáticamente a uno de los nodos de proceso adicionales de solo lectura.

## <a name="deep-dive-questions"></a>Preguntas de profundización

### <a name="can-i-mix-hyperscale-and-single-databases-in-a-single-logical-server"></a>¿Puedo mezclar a gran escala y las bases de datos únicas en un único servidor lógico

 Sí, puede hacerlo.

### <a name="does-hyperscale-require-my-application-programming-model-to-change"></a>¿Requiere Hiperescala que cambie mi modelo de programación de la aplicación?

No, el modelo de programación de la aplicación permanece tal cual. Puede usar la cadena de conexión y los demás modos normales como de costumbre para interactuar con su base de datos de Azure SQL.

### <a name="what-transaction-isolation-levels-are-going-to-be-default-on-sql-database-hyperscale-database"></a>¿Qué niveles de aislamiento de transacción van a ser los predeterminados en las bases de datos de hiperescala de SQL Database?

En el nodo principal, el nivel de aislamiento de transacción es RCSI (nivel de aislamiento instantáneo de lectura confirmada). En los nodos secundarios de escalado de lectura, el nivel de aislamiento es Instantáneo.

### <a name="can-i-bring-my-on-premises-or-iaas-sql-server-license-to-sql-database-hyperscale"></a>¿Puedo traer mi licencia de SQL Server local o de IaaS a Hiperescala de SQL Database?

Sí, [Ventaja híbrida de Azure](https://azure.microsoft.com/pricing/hybrid-benefit/) está disponible para Hiperescala. Cada núcleo de SQL Server Standard se puede asignar a 1 núcleo virtual de Hiperescala. Cada núcleo de SQL Server Enterprise se puede asignar a 4 núcleos virtuales de Hiperescala. No necesita una licencia de SQL para las réplicas secundarias. El precio de la ventaja híbrida de Azure se aplicará automáticamente a las réplicas de escalado de lectura (secundarias).

### <a name="what-kind-of-workloads-is-sql-database-hyperscale-designed-for"></a>¿Para qué tipo de cargas de trabajo está diseñado el nivel de servicio Hiperescala de SQL Database?

Hiperescala de SQL Database es compatible con todas las cargas de trabajo de SQL Server, pero está optimizado principalmente para OLTP. Puede aportar híbrido (HTAP) y analítico (data mart.) las cargas de trabajo también.

### <a name="how-can-i-choose-between-azure-sql-data-warehouse-and-sql-database-hyperscale"></a>¿Cómo puedo elegir entre Azure SQL Data Warehouse e Hiperescala de SQL Database?

Si actualmente está ejecutando consultas de análisis interactivas mediante SQL Server como almacenamiento de datos, Hiperescala de SQL Database es una magnífica opción ya que puede hospedar almacenamientos de datos relativamente pequeños (desde unos pocos TB hasta 10 TB) a un costo reducido y puede migrar la carga de trabajo del almacenamiento de datos a Hiperescala de SQL Database sin cambios en el código T-SQL.

Si va a ejecutar análisis de datos a gran escala con consultas complejas y mediante almacenamiento de datos paralelos (PDW), Teradata o cualquier otro almacenamiento de datos con procesador paralelo masivo (MPP), SQL Data Warehouse puede ser la mejor opción.
  
## <a name="sql-database-hyperscale-compute-questions"></a>Preguntas sobre proceso en Hiperescala de SQL Database

### <a name="can-i-pause-my-compute-at-any-time"></a>¿Puedo detener el proceso en cualquier momento?

No en este momento, sin embargo, puede escalar el proceso y el número de réplicas hacia abajo para reducir el costo durante las horas de poca actividad.

### <a name="can-i-provision-a-compute-with-extra-ram-for-my-memory-intensive-workload"></a>¿Puedo aprovisionar un proceso con memoria RAM adicional para mi carga de trabajo con uso intensivo de memoria?

No. Para obtener más memoria RAM, deberá actualizar a un tamaño de proceso mayor. Para más información, consulte los [tamaños de almacenamiento y proceso de Hiperescala](sql-database-vcore-resource-limits-single-databases.md#hyperscale-service-tier).

### <a name="can-i-provision-multiple-compute-nodes-of-different-sizes"></a>¿Puedo aprovisionar varios nodos de proceso de diferentes tamaños?

No.

### <a name="how-many-read-scale-replicas-are-supported"></a>¿Cuántas réplicas de escalado de lectura se admiten?

Se crean las bases de datos a gran escala con una réplica de escalado de lectura (dos réplicas en total) de forma predeterminada. Puede escalar el número de réplicas de solo lectura entre 0 y 4 mediante la [portal Azure](https://portal.azure.com), [T-SQL](https://docs.microsoft.com/sql/t-sql/statements/alter-database-transact-sql?view=azuresqldb-current), [Powershell](https://docs.microsoft.com/powershell/module/azurerm.sql/set-azurermsqldatabase) o [CLI](https://docs.microsoft.com/cli/azure/sql/db#az-sql-db-update)...

### <a name="for-high-availability-do-i-need-to-provision-additional-compute-nodes"></a>¿Tengo que aprovisionar nodos de proceso adicionales para lograr una alta disponibilidad?

En las bases de datos a gran escala, la resistencia se proporciona en el nivel de almacenamiento. Solo necesita una réplica para proporcionar resistencia. Cuando la réplica de proceso está inactiva, se crea automáticamente una nueva réplica sin ninguna pérdida de datos.

Sin embargo, si hay solo una réplica, puede que tarde algún tiempo en generar la caché local en la nueva réplica después de la conmutación por error. Durante la fase de recompilación de la caché, la base de datos captura los datos directamente desde los servidores de páginas, lo que produce una disminución del rendimiento de IOPS y de consulta.

Para las aplicaciones críticas que requieren alta disponibilidad, debe aprovisionar al menos 2 nodos de proceso, incluido el nodo de proceso principal (valor predeterminado). De esta forma, hay una espera activa disponible en caso de una conmutación por error.

## <a name="data-size-and-storage-questions"></a>Preguntas sobre el tamaño y el almacenamiento de datos

### <a name="what-is-the-max-db-size-supported-with-sql-database-hyperscale"></a>¿Cuál es el tamaño máximo de la base de datos compatible con Hiperescala de SQL Database?

100 TB

### <a name="what-is-the-size-of-the-transaction-log-with-hyperscale"></a>¿Cuál es el tamaño del registro de transacciones con Hiperescala?

El registro de transacciones con Hiperescala es prácticamente infinito. No es necesario preocuparse por quedarse sin espacio en el registro en un sistema que tenga un alto rendimiento. No obstante, la velocidad de generación de registros se puede ver limitada por cargas de trabajo intensivas continuas. La velocidad de generación de registro sostenido de pico es aproximadamente 100 MB/seg.

### <a name="does-my-temp-db-scale-as-my-database-grows"></a>¿Escala mi base de datos temporal a medida que crece mi base de datos?

La base de datos `tempdb` está ubicada en el almacenamiento SSD local y está configurada según el tamaño de proceso que aprovisione. `tempdb` se ha optimizado y está lista para proporcionar ventajas de rendimiento máximo. No puede configurar el tamaño de `tempdb` ya que de su administración se encarga el subsistema de almacenamiento.

### <a name="does-my-database-size-automatically-grow-or-do-i-have-to-manage-the-size-of-the-data-files"></a>¿Crece automáticamente el tamaño de mi base de datos o tengo que administrar el tamaño de los archivos de datos?

El tamaño de la base de datos crece automáticamente a medida que inserta o ingiere más datos.

### <a name="what-is-the-smallest-database-size-that-sql-database-hyperscale-supports-or-starts-with"></a>¿Cuál es el tamaño de base de datos más pequeño que admite Hiperescala de SQL Database?

10 GB

### <a name="in-what-increments-does-my-database-size-grow"></a>¿En cuantos GB se incrementa el tamaño de mi base de datos cada vez?

1 GB

### <a name="is-the-storage-in-sql-database-hyperscale-local-or-remote"></a>El almacenamiento en Hiperescala de SQL Database ¿es local o remoto?

En Hiperescala, los archivos de datos se almacenan en el almacenamiento estándar de Azure. Los datos se almacenan principalmente en caché en el almacenamiento SSD local, en máquinas cercanas a los nodos de proceso. Además, los nodos de proceso tienen una caché en el SSD local y en memoria (caché del búfer, etc) para reducir la frecuencia de la captura de datos de los nodos remotos.

### <a name="can-i-manage-or-define-files-or-filegroups-with-hyperscale"></a>¿Puedo administrar o definir archivos o grupos de archivos con Hiperescala?

Sin 
  
### <a name="can-i-provision-a-hard-cap-on-the-data-growth-for-my-database"></a>¿Puedo aprovisionar un límite duro en el crecimiento de datos de mi base de datos?

Sin 

### <a name="how-are-data-files-laid-out-with-sql-database-hyperscale"></a>¿Cómo se disponen los archivos de datos con Hiperescala de SQL Database?

Los servidores de páginas controlan los archivos de datos. A medida que crece el tamaño de los datos, se agregan archivos de datos y nodos de servidor de páginas asociados.

### <a name="is-database-shrink-supported"></a>¿Se admite la reducción de la base de datos?

Sin 

### <a name="is-database-compression-supported"></a>¿Se admite la compresión de la base de datos?

Sí

### <a name="if-i-have-a-huge-table-does-my-table-data-get-spread-out-across-multiple-data-files"></a>Si tengo una tabla de gran tamaño, ¿se reparten los datos de la tabla entre varios archivos de datos?

Sí. Las páginas de datos asociadas a una determinada tabla pueden acabar en varios archivos de datos, los cuales forman todos parte del mismo grupo de archivos. SQL Server usa una [estrategia de relleno proporcional](https://docs.microsoft.com/sql/relational-databases/databases/database-files-and-filegroups#file-and-filegroup-fill-strategy) para distribuir los datos entre los archivos de datos.

## <a name="data-migration-questions"></a>Preguntas sobre migración de datos

### <a name="can-i-move-my-existing-azure-sql-databases-to-the-hyperscale-service-tier"></a>¿Puedo trasladar las bases de datos de Azure SQL existentes al nivel de servicio Hiperescala?

Sí. Puede trasladar las bases de datos de Azure SQL existentes a Hiperescala. Se trata de una migración unidireccional. No puede trasladar las bases de datos del nivel de servicio Hiperescala a ningún otro. Es recomendable que haga una copia de las bases de datos de producción y migre a Hiperescala para realizar las pruebas de concepto.
  
### <a name="can-i-move-my-hyperscale-databases-to-other-editions"></a>¿Puedo mover mis bases de datos de hiperescala a otras ediciones?

No. En este momento, no se puede mover una base de datos a gran escala a otro nivel de servicio.

### <a name="do-i-lose-any-functionality-or-capabilities-after-migration-to-the-hyperscale-service-tier"></a>¿Pierdo alguna funcionalidad al migrar al nivel de servicio Hiperescala?

Sí. Algunas de las características de Azure SQL Database aún no se admiten en gran escala, incluyendo pero sin carácter limitativo backups a retención largo plazo. Después de migrar las bases de datos a Hiperescala, esas características dejan de funcionar.  Esperamos que estas limitaciones para ser temporal.

### <a name="can-i-move-my--on-premises-sql-server-database-or-my-sql-server-virtual-machine-database-to-hyperscale"></a>¿Puedo mover mi base de datos de SQL Server local o mi base de datos de máquina virtual de SQL Server a Hiperescala?

Sí. Puede usar todas las tecnologías existentes de migración para migrar a Hiperescala, incluidas BACPAC, replicación transaccional y carga de datos lógica. Consulte también [Azure Database Migration Service](../dms/dms-overview.md).

### <a name="what-is-my-downtime-during-migration-from-an-on-premises-or-virtual-machine-environment-to-hyperscale-and-how-can-i-minimize-it"></a>¿Cuánto tiempo de inactividad experimentaré durante la migración de un entorno local o máquina virtual a Hiperescala y cómo puedo minimizarlo?

El tiempo de inactividad es el mismo que el tiempo de inactividad que se produce cuando migra la base de datos a una base de datos única de Azure SQL Database. Puede usar la [replicación transaccional](replication-to-sql-database.md#data-migration-scenario
) para minimizar el tiempo de inactividad de la migración para bases de datos de pocos TB de tamaño. Para bases de datos de gran tamaño (más de 10 TB), puede considerar la opción de migrar datos con ADF, Spark u otras tecnologías de movimiento de datos.

### <a name="how-much-time-would-it-take-to-bring-in-x-amount-of-data-to-sql-database-hyperscale"></a>¿Cuánto tiempo se tardará en llevar una cantidad X de datos a Hiperescala de SQL Database?

Es capaz de consumir 100 MB/s de nuevo o modificar datos a gran escala.

### <a name="can-i-read-data-from-blob-storage-and-do-fast-load-like-polybase-and-sql-data-warehouse"></a>¿Puedo leer datos de Blob Storage y realizar una carga rápida (como con Polybase y SQL Data Warehouse)?

Puede leer datos de Azure Storage y cargar datos en una base de datos de hiperescala (al igual que lo haría con una base de datos única normal). Actualmente, Polybase no se admite en Azure SQL Database. Puede hacer que Polybase use [Azure Data Factory](https://docs.microsoft.com/azure/data-factory/) o ejecute un trabajo de Spark en [Azure Databricks](https://docs.microsoft.com/azure/azure-databricks/) con el [conector de Spark para SQL](sql-database-spark-connector.md). Este conector admite la inserción masiva.

El modelo de recuperación simple o de registro masivo no se admite en Hiperescala. Se necesita el modelo de recuperación completa para proporcionar alta disponibilidad. Sin embargo, Hiperescala proporciona una ingesta de datos mejor en comparación con una base de datos de Azure SQL única gracias a la nueva arquitectura de registro.

### <a name="does-sql-database-hyperscale-allow-provisioning-multiple-nodes-for-ingesting-large-amounts-of-data"></a>¿Permite Hiperescala de SQL Database el aprovisionamiento de varios nodos para ingerir grandes cantidades de datos?

No. Hiperescala de SQL Database es una arquitectura SMP y no una arquitectura de multiprocesos asimétricos o multimaestro. Solo se pueden crear varias réplicas para escalar horizontalmente cargas de trabajo de solo lectura.

### <a name="what-is-the-oldest-sql-server-version-will-sql-database-hyperscale-support-migration-from"></a>¿Cuál es la versión más antigua de SQL Server desde la que se admiten migraciones a Hiperescala de SQL Database?

SQL Server 2005. Para más información, consulte [Migración a una base de datos única o a una base de datos agrupada](sql-database-single-database-migrate.md#migrate-to-a-single-database-or-a-pooled-database). Para más información sobre problemas de compatibilidad, consulte [Solución de problemas de compatibilidad de migración de bases de datos](sql-database-single-database-migrate.md#resolving-database-migration-compatibility-issues).

### <a name="does-sql-database-hyperscale-support-migration-from-other-data-sources-such-as-aurora-mysql-oracle-db2-and-other-database-platforms"></a>¿Admite Hiperescala de SQL Database la migración desde otros orígenes de datos como Aurora, MySQL, Oracle, DB2 y desde otras plataformas de base de datos?

Sí. Para realizar una migración desde orígenes de datos distintos a SQL Server, se necesita una migración lógica. Puede usar [Azure Database Migration Service](../dms/dms-overview.md) para una migración lógica.

## <a name="business-continuity-and-disaster-recovery-questions"></a>Preguntas sobre recuperación ante desastres y continuidad empresarial

### <a name="what-slas-are-provided-for-a-hyperscale-database"></a>¿Cuál es el Acuerdo de Nivel de Servicio que se proporciona para una base de datos de hiperescala?

Con el valor predeterminado principal y 1 base de datos secundaria legible, el SLA es la disponibilidad del 99,95%.  Con varias réplicas, el SLA va hasta un 99,99%.  

### <a name="are-the-database-backups-managed-for-me-by-the-azure-sql-database-service"></a>¿Administra el servicio Azure SQL Database las copias de seguridad de las bases de datos en mi lugar?

Sí

### <a name="how-often-are-the-database-backups-taken"></a>¿Con qué frecuencia se realizan las copias de seguridad de las bases de datos?

No hay copias de seguridad de registros tradicionales, completas y diferenciales para las bases de datos de hiperescala de SQL Database. En su lugar, hay instantáneas regulares de los archivos de datos y el registro que se genera se conserva tal cual está durante el período de retención configurado o que esté disponible para el usuario.

### <a name="does-sql-database-hyperscale-support-point-in-time-restore"></a>¿Admite Hiperescala de SQL Database la restauración a un momento dado?

Sí

### <a name="what-is-the-recovery-point-objective-rporecovery-time-objective-rto-with-backuprestore-in-sql-database-hyperscale"></a>¿Cuál es el objetivo de punto de recuperación (RPO) y el objetivo de tiempo de recuperación (RTO) con la copia de seguridad y restauración en Hiperescala de SQL Database?

El RPO es 0 minutos. El objetivo de RTO es inferior a 10 minutos, sin importar el tamaño de la base de datos. 

### <a name="do-backups-of-large-databases-affect-compute-performance-on-my-primary"></a>¿Afectan las copias de seguridad de las bases de datos de gran tamaño al rendimiento de proceso del nodo principal?

No. El subsistema de almacenamiento administra las copias de seguridad y aprovecha las instantáneas de archivos. Estas no afectan a la carga de trabajo del usuario en el nodo principal.

### <a name="can-i-perform-geo-restore-with-a-sql-database-hyperscale-database"></a>¿Puedo realizar la restauración geográfica con una base de datos de hiperescala de SQL Database?

Sí.  Restauración geográfica es totalmente compatible.

### <a name="can-i-setup-geo-replication-with-sql-database-hyperscale-database"></a>¿Puedo configurar la replicación geográfica con una base de datos de hiperescala de SQL Database?

De momento, no.

### <a name="do-my-secondary-compute-nodes-get-geo-replicated-with-sql-database-hyperscale"></a>¿Se realiza la replicación geográfica de mis nodos de proceso secundarios con Hiperescala de SQL Database?

De momento, no.

### <a name="can-i-take-a-sql-database-hyperscale-database-backup-and-restore-it-to-my-on-premises-server-or-sql-server-in-vm"></a>¿Puedo realizar una copia de seguridad de una base de datos de hiperescala de SQL Database y restaurarla en mi servidor local o en SQL Server en una máquina virtual?

No. El formato de almacenamiento de las bases de datos de hiperescala es diferente al tradicional de SQL Server y no podría controlar las copias de seguridad ni tener acceso a ellas. Para sacar los datos de una base de datos de hiperescala de SQL Database, use el servicio de exportación o use scripting más BCP.

## <a name="cross-feature-questions"></a>Preguntas sobre características transversales

### <a name="do-i-lose-any-functionality-or-capabilities-after-migration-to-the-hyperscale-service-tier"></a>¿Pierdo alguna funcionalidad al migrar al nivel de servicio Hiperescala?

Sí. Algunas de las características de Azure SQL Database no se admiten en gran escala, incluyendo pero la copia de seguridad de retención no se limita a largo plazo. Después de migrar las bases de datos a Hiperescala, esas características dejan de funcionar.

### <a name="will-polybase-work-with-sql-database-hyperscale"></a>¿Funciona Polybase con Hiperescala de SQL Database?

No. Polybase no se admite en Azure SQL Database.

### <a name="does-the-compute-have-support-for-r-and-python"></a>¿Es compatible el proceso con R y Python?

No. R y Python no se admiten en Azure SQL Database.

### <a name="are-the-compute-nodes-containerized"></a>¿Están los nodos de proceso en contenedores?

No. La base de datos reside en una máquina virtual de proceso y no en un contenedor.

## <a name="performance-questions"></a>Preguntas sobre rendimiento

### <a name="how-much-throughput-can-i-push-on-the-largest-sql-database-hyperscale-compute"></a>¿Qué capacidad de procesamiento puedo insertar en el mayor tamaño de proceso de Hiperescala de SQL Database?

Hemos visto una coherente 100 MB/s de datos modificados (generación de datos de registro de transacciones)

### <a name="how-many-iops-do-i-get-on-the-largest-sql-database-hyperscale-compute"></a>¿Cuántas IOPS puedo obtener en el mayor tamaño de proceso de Hiperescala de SQL Database?

IOPS y latencia de E/S variará según los patrones de carga de trabajo.  Si los datos que necesitan tener acceso están locales en memoria caché del proceso, será los mismos patrones de E/S como SSD local.   

### <a name="does-my-throughput-get-affected-by-backups"></a>¿Afectan las copias de seguridad a mi rendimiento?

No. El proceso está desacoplado de la capa de almacenamiento para evitar cualquier efecto en el proceso.

### <a name="does-my-throughput-get-affected-as-i-provision-additional-compute-nodes"></a>¿Se ve afectado el rendimiento cuando aprovisiono nodos de proceso adicionales?

Dado que el almacenamiento se comparte y no hay ninguna replicación física directa que sucede entre nodos de proceso principal y secundaria, técnicamente, el rendimiento en el nodo principal no se verán afectado mediante la adición de nodos de escalado de lectura. Sin embargo, podemos limitar las cargas de trabajo intensivas continuas para permitir que la aplicación del registro en los nodos secundarios y en los servidores de páginas se ponga al día y evite un mal rendimiento de lectura en los nodos secundarios.

## <a name="scalability-questions"></a>Preguntas sobre escalabilidad

### <a name="how-long-would-it-take-to-scale-up-and-down-a-compute-node"></a>¿Cuánto tiempo se tarda en realizar un escalado y reducción vertical de un nodo de proceso?

Proceso de escalado vertical o hacia abajo, tardará entre 5 y 10 minutos, independientemente del tamaño de los datos.

### <a name="is-my-database-offline-while-the-scaling-updown-operation-is-in-progress"></a>¿Está mi base de datos sin conexión mientras que la operación de escalado y reducción vertical está en curso?

No. El escalado y reducción vertical se realiza en línea.

### <a name="should-i-expect-connection-drop-when-the-scaling-operations-are-in-progress"></a>¿Se puede esperar una interrupción de la conexión cuando las operaciones de escalado están en curso?

El escalado y la reducción vertical tienen como resultado la interrupción de las conexiones existentes cuando se produce la conmutación por error en el nodo de proceso con el tamaño de destino. La incorporación de réplicas de lectura no produce interrupciones de conexión.

### <a name="is-the-scaling-up-and-down-of-compute-nodes-automatic-or-end-user-triggered-operation"></a>¿Se produce el escalado y reducción vertical de los nodos de proceso de forma automática o se trata de una operación desencadenada por el usuario final?

Por el usuario final. No es una operación automática.  

### <a name="does-my-tempb-also-grow-as-the-compute-is-scaled-up"></a>¿Crece `tempb` a medida que el proceso se escala verticalmente?

Sí. Temp db se escalará verticalmente de forma automática a medida que crezca el proceso.  

### <a name="can-i-provision-multiple-primary-computes-such-as-a-multi-master-system-where-multiple-primary-compute-heads-can-drive-a-higher-level-of-concurrency"></a>¿Puedo aprovisionar varios procesos principales como un sistema multimaestro en los que los nodos principales pueden impulsar un mayor nivel de simultaneidad?

No. Solo el nodo de proceso principal acepta solicitudes de lectura y escritura. Los nodos de proceso secundarios aceptan únicamente solicitudes de solo lectura.

## <a name="read-scale-questions"></a>Preguntas sobre escalado de lectura

### <a name="how-many-secondary-compute-nodes-can-i-provision"></a>¿Cuántos nodos de proceso secundarios puedo aprovisionar?

Creamos 2 réplicas para las bases de datos a gran escala de forma predeterminada. Si desea ajustar el número de réplicas, puede hacerlo mediante [portal Azure](https://portal.azure.com).

### <a name="how-do-i-connect-to-these-secondary-compute-nodes"></a>¿Cómo me conecto a estos nodos de proceso secundarios?

Puede conectarse a estos nodos de proceso adicionales de solo lectura estableciendo el argumento `ApplicationIntent` de la cadena de conexión en `read_only`. Todas las conexiones marcadas con `read-only` se enrutan automáticamente a uno de los nodos de proceso adicionales de solo lectura.  

### <a name="can-i-create-a-dedicated-endpoint-for-the-read-scale-replica"></a>¿Puedo crear un punto de conexión dedicado para la réplica de escalado de lectura?

No. Réplica de escalado de lectura solo se puede conectar mediante la especificación de `ApplicationIntent=ReadOnly`.

### <a name="does-the-system-do-intelligent-load-balancing-of-the-read-workload"></a>¿Realiza el sistema un equilibrio de carga inteligente de la carga de trabajo de lectura?

No. La carga de trabajo solo lectura se redirige a una réplica aleatoria de escalado de lectura.

### <a name="can-i-scale-updown-the-secondary-compute-nodes-independently-of-the-primary-compute"></a>¿Se puede realizar un escalado y reducción vertical de los nodos de proceso secundarios independientemente del nodo de proceso principal?

No. Los nodos de proceso secundario también se utilizan para alta disponibilidad, por lo que necesitan para ser la misma configuración que el servidor principal, en el caso de una conmutación por error.

### <a name="do-i-get-different-temp-db-sizing-for-my-primary-compute-and-my-additional-secondary-compute-nodes"></a>¿Puedo obtener un tamaño de temp db diferente para mi nodo de proceso principal y los secundarios adicionales?

No. Su `tempdb` se configura basándose en el aprovisionamiento de tamaño de proceso, los nodos de proceso secundario son el mismo tamaño que el proceso principal.

### <a name="can-i-add-indexes-and-views-on-my-secondary-compute-nodes"></a>¿Puedo agregar índices y vistas en mis nodos de proceso secundarios?

No. Las bases de datos de hiperescala tienen el almacenamiento compartido, lo que significa que todos los nodos de proceso ven las mismas tablas, índices y vistas. Si desea índices adicionales optimizados para lecturas en el nodo secundario, debe primero agregarlos en el nodo principal.

### <a name="how-much-delay-is-there-going-to-be-between-the-primary-and-secondary-compute-node"></a>¿Cuánto retraso va a haber entre el nodo de proceso principal y los secundarios?

Desde el momento en que se confirma una transacción en el nodo principal, y según la velocidad de generación del registro, puede ser instantáneo o tener un retraso de milisegundos.

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información sobre el nivel de servicio a gran escala, consulte [nivel de servicio de hiperescala](sql-database-service-tier-hyperscale.md).
