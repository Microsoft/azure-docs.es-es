---
title: Tecnologías en memoria de Azure SQL Database | Microsoft Docs
description: Las tecnologías en memoria de Azure SQL Database mejoran notablemente el rendimiento de las cargas de trabajo transaccionales y de análisis.
services: sql-database
ms.service: sql-database
ms.subservice: development
ms.custom: ''
ms.devlang: ''
ms.topic: conceptual
author: stevestein
ms.author: sstein
ms.reviewer: ''
manager: craigg
ms.date: 03/19/2019
ms.openlocfilehash: 5681b5aa46acc1192675da0b1cceee596dfa0105
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65799903"
---
# <a name="optimize-performance-by-using-in-memory-technologies-in-sql-database"></a>Optimización del rendimiento mediante las tecnologías en memoria de SQL Database

Las tecnologías en memoria de Azure SQL Database permiten mejorar el rendimiento de la aplicación y reducen el costo de la base de datos. 

## <a name="when-to-use-in-memory-technologies"></a>Cuándo usar tecnologías en memoria

Mediante el uso de las tecnologías en memoria de Azure SQL Database, puede lograr mejoras de rendimiento con diversas cargas de trabajo:

- **Transaccionales** (procesamiento transaccional en línea (OLTP)) en las que la mayoría de las solicitudes leen o actualizan conjuntos de datos más pequeños (por ejemplo, operaciones CRUD).
- **Analíticas** (procesamiento analítico en línea (OLAP)) en las que la mayoría de las consultas tienen cálculos complejos para los informes e incluyen un determinado número de consultas que cargan y anexan datos a las tablas existentes (denominadas "cargas masivas") o los eliminan de las mismas. 
- **Mixtas** (procesamiento analítico-transaccional híbrido (HTAP)) en las que las consultas OLTP y OLAP se ejecutan en el mismo conjunto de datos.

Las tecnologías en memoria pueden mejorar el rendimiento de dichas cargas de trabajo al conservar en la memoria los datos que se deben procesar mediante la compilación nativa de las consultas o el procesamiento avanzado (como el procesamiento por lotes y las instrucciones SIMD) disponibles en el hardware subyacente. 

## <a name="overview"></a>Información general

Azure SQL Database cuenta con las siguientes tecnologías en memoria:
- *[OLTP en memoria](https://docs.microsoft.com/sql/relational-databases/in-memory-oltp/in-memory-oltp-in-memory-optimization)* aumenta el número de transacciones por segundo y reduce la latencia de su procesamiento. Estas son las situaciones en las que se obtienen ventajas con OLTP en memoria: procesamiento de transacciones de alto rendimiento, como operaciones comerciales y juegos, ingesta de datos de eventos o dispositivos de IoT, almacenamiento en caché, carga de datos y escenarios de tablas temporales y variables de tablas.
- Los *índices de almacén de columnas en clúster* reducen el espacio de almacenamiento necesario (hasta 10 veces) y mejoran el rendimiento de las consultas de análisis e informes. Puede usarlos con las tablas de hechos de sus data marts para incluir más datos en la base de datos y mejorar el rendimiento. También puede usarlos con los datos históricos de la base de datos operativa para archivar hasta 10 veces más datos, así como para disfrutar de un incremento equivalente en el número de consultas realizadas sobre ellos.
- Con los *índices de almacén de columnas no clúster* para HTAP, podrá obtener información en tiempo real sobre su negocio realizando consultas directamente a la base de datos operativa, sin necesidad de ejecutar un caro proceso de extracción, transformación y carga (ETL) ni esperar a que se rellene el almacén de datos. Los índices de almacén de columnas no en clúster permiten una ejecución rápida de las consultas de análisis en la base de datos OLTP y, a la vez, reducen el impacto en la carga de trabajo operativa.
- Los *índices de almacén de columnas en clúster optimizados para memoria* para HTAP le permiten realizar el procesamiento de transacciones de manera rápida y ejecutar consultas de análisis *simultáneamente* de manera muy rápida en los mismos datos.

Las opciones de índices de almacén de columnas y OLTP en memoria forman parte de SQL Server desde 2012 y 2014, respectivamente. Azure SQL Database y SQL Server comparten la misma implementación de tecnologías en memoria. A partir de ahora, las nuevas funciones para estas tecnologías se publican primero en Azure SQL Database y después, en SQL Server.

## <a name="benefits-of-in-memory-technology"></a>Ventajas de la tecnología en memoria

Gracias al procesamiento más eficiente de las consultas y las transacciones, las tecnologías en memoria también lo ayudan a reducir costos. Normalmente no necesita actualizar el plan de tarifa de la base de datos para lograr mejoras de rendimiento. En algunos casos, tal vez pueda reducir incluso el plan de tarifa sin dejar de observar mejoras de rendimiento con las tecnologías en memoria.

A continuación se muestran dos ejemplos de cómo OLTP en memoria ayudó significativamente a mejorar el rendimiento:

- Gracias al uso de OLTP en memoria, [Quorum Business Solutions pudo duplicar la carga de trabajo al mismo tiempo que mejoró las DTU en un 70 %](https://customers.microsoft.com/story/quorum-doubles-key-databases-workload-while-lowering-dtu-with-sql-database).

  - DTU significa *unidad de transmisión de datos* e incluye una medida del consumo de recursos.
- El vídeo siguiente muestra la mejora significativa del consumo de recursos con una carga de trabajo de ejemplo: [In-Memory OLTP in Azure SQL Database Video](https://channel9.msdn.com/Shows/Data-Exposed/In-Memory-OTLP-in-Azure-SQL-DB) (Vídeo sobre OLTP en memoria de Azure SQL Database).
  - Para obtener más información, consulte la entrada de blog: [In-Memory OLTP in Azure SQL Database Blog Post](https://azure.microsoft.com/blog/in-memory-oltp-in-azure-sql-database/) (OLTP en memoria en la entrada de Blog de base de datos de SQL de Azure)

> [!NOTE]  
> Las tecnologías en memoria están disponibles en las bases de datos de los niveles Premium y Crítico para la empresa de Azure SQL Database y en los grupos elásticos Premium.

En el siguiente vídeo se explican posibles mejoras de rendimiento obtenidas con las tecnologías en memoria de Azure SQL Database. Tenga presente que la mejora de rendimiento que obtenga depende siempre de muchos factores, como la naturaleza de la carga de trabajo y los datos, los patrones de acceso de la base de datos, etc.

> [!VIDEO https://channel9.msdn.com/Blogs/Azure/Azure-SQL-Database-In-Memory-Technologies/player]
>
>

En este artículo, se describen aspectos de OLTP en memoria y los índices de almacén de columnas específicos de Azure SQL Database junto con algunos ejemplos:

- Veremos la repercusión de estas tecnologías en el almacenamiento, así como en los límites de tamaño de los datos.
- Después trataremos cómo administrar el movimiento de bases de datos que usan estas tecnologías entre los distintos planes de tarifa.
- Y también eremos dos ejemplos que ilustran el uso de OLTP en memoria y de los índices del almacén de columnas en Azure SQL Database.

Para más información, consulte:

- [Información general y escenarios de uso de OLTP en memoria](https://msdn.microsoft.com/library/mt774593.aspx) (incluye referencias a información y casos prácticos de clientes para familiarizarse)
- [Documentación de In-Memory OLTP](https://msdn.microsoft.com/library/dn133186.aspx)
- [Descripción de los índices de almacén de columnas](https://msdn.microsoft.com/library/gg492088.aspx)
- Procesamiento analítico y transaccional híbrido (HTAP), también conocido como [análisis operativo en tiempo real](https://msdn.microsoft.com/library/dn817827.aspx)

## <a name="in-memory-oltp"></a>OLTP en memoria (optimización en memoria

La tecnología OLTP en memoria proporciona operaciones de acceso a datos sumamente rápidas al mantener todos los datos en memoria. Además, usa índices especializados, compilación nativa de consultas y acceso a datos libre de bloqueos temporales para mejorar el rendimiento de la carga de trabajo OLTP. Hay dos maneras de organizar los datos de OLTP en memoria:

- El formato **almacén de filas optimizadas para memoria**, en el que cada fila es un objeto de memoria independiente. Se trata de un formato clásico de OLTP en memoria optimizado para cargas de trabajo OLTP de alto rendimiento. Existen dos tipos de tablas optimizadas para memoria que se pueden usar en el formato de almacén de filas optimizadas para memoria:
  - *Tablas duraderas* (SCHEMA_AND_DATA), en las que las filas que se encuentran en la memoria se conservan después de reiniciar el servidor. Este tipo de tablas se comporta como una tabla de almacén de filas tradicional, con las ventajas adicionales de las optimizaciones en memoria.
  - *Tablas no duraderas* (SCHEMA_ONLY) donde las filas son no se conservan después del reinicio. Este tipo de tabla está diseñado para datos temporales (por ejemplo, tablas temporales o de reemplazo) o para tablas en las que necesite cargar datos rápidamente antes de moverlos a alguna tabla persistente (denominadas "tablas de almacenamiento provisional").
- El formato **Almacén de columnas optimizadas para memoria**, en el que los datos se organizan en un formato de columnas. Esta estructura está diseñada para escenarios HTAP donde es necesario ejecutar consultas analíticas en la misma estructura de datos en la que se está ejecutando la carga de trabajo OLTP.

> [!Note]
> La tecnología de OLTP en memoria está diseñada para las estructuras de datos que pueden residir completamente en memoria. Puesto que no se pueden descargar los datos en memoria en el disco, asegúrese de usar una base de datos que tenga memoria suficiente. Consulte [Límite de almacenamiento y tamaño de datos para OLTP en memoria](#data-size-and-storage-cap-for-in-memory-oltp) para conocer más detalles.

Una guía rápida sobre OLTP en memoria: [Inicio rápido 1: Tecnologías de OLTP en memoria para acelerar el rendimiento de Transact-SQL](https://msdn.microsoft.com/library/mt694156.aspx) es otro artículo que le ayudará a empezar a trabajar

Vídeos detallados sobre las tecnologías:

- [In-Memory OLTP in Azure SQL Database](https://channel9.msdn.com/Shows/Data-Exposed/In-Memory-OTLP-in-Azure-SQL-DB) (OLTP en memoria en Azure SQL Database), que contiene una demostración de las ventajas de rendimiento y los pasos para reproducir estos resultados
- [In-Memory OLTP Videos: What it is and When/How to use it](https://blogs.msdn.microsoft.com/sqlserverstorageengine/20../../in-memory-oltp-video-what-it-is-and-whenhow-to-use-it/) (Vídeos de OLTP en memoria: qué es y cuándo/cómo usarlo)

No existe ningún mecanismo de programación para comprender si una base de datos específica admite OLTP en memoria. Puede ejecutar la siguiente consulta de Transact-SQL:
```
SELECT DatabasePropertyEx(DB_NAME(), 'IsXTPSupported');
```
Si la consulta devuelve **1**, OLTP en memoria se admite en esta base de datos. Las siguientes consultas identifican todos los objetos que deben quitarse antes de que se una base de datos de nivel Premium pase al nivel Estándar/Básico:
```
SELECT * FROM sys.tables WHERE is_memory_optimized=1
SELECT * FROM sys.table_types WHERE is_memory_optimized=1
SELECT * FROM sys.sql_modules WHERE uses_native_compilation=1
```

### <a name="data-size-and-storage-cap-for-in-memory-oltp"></a>Límite de almacenamiento y tamaño de datos para OLTP en memoria

OLTP en memoria incluye tablas optimizadas para memoria, que se usan para almacenar los datos de los usuarios. Estas tablas deben caber en la memoria. Dado que administra la memoria directamente en el servicio de SQL Database, tenemos el concepto de una cuota para datos de usuario. Esta idea se conoce como *almacenamiento de OLTP en memoria*.

Cada plan de tarifa de grupo elástico y de base de datos única admitido incluye una cantidad determinada de almacenamiento de OLTP en memoria. Consulte [Límites de los recursos basados en DTU para bases de datos únicas](sql-database-dtu-resource-limits-single-databases.md), [Límites de los recursos basados en DTU para grupos elásticos](sql-database-dtu-resource-limits-elastic-pools.md), [Límites de los recursos basados en núcleos virtuales para bases de datos únicas](sql-database-vcore-resource-limits-single-databases.md) y [Límites de los recursos basados en núcleos virtuales para grupos elásticos](sql-database-vcore-resource-limits-elastic-pools.md).

Los siguientes elementos cuentan para su límite de almacenamiento de OLTP en memoria:

- Las filas de datos de usuarios activos en tablas optimizadas para memoria y variables de tabla. Tenga en cuenta que las versiones antiguas de las filas no cuentan para el límite.
- Los índices de tablas optimizadas para memoria.
- La sobrecarga operacional de operaciones ALTER TABLE.

Si alcanza el límite, recibirá un error que le notificará que se ha quedado sin cuota y no podrá volver a insertar o actualizar datos. Para mitigar este error, elimine datos o aumente el plan de tarifa de la base de datos o del grupo.

Para obtener más información sobre cómo supervisar la utilización del almacenamiento de OLTP en memoria y configurar alertas que se activen cuando casi haya alcanzado el límite, consulte [Supervisión del almacenamiento en memoria](sql-database-in-memory-oltp-monitoring.md).

#### <a name="about-elastic-pools"></a>Acerca de los grupos elásticos

Con grupos elásticos, el almacenamiento de OLTP en memoria se comparte entre todas las bases de datos del grupo. Por lo tanto, el uso de una base de datos puede afectar a otras bases de datos. Existen dos formas de mitigar este problema:

- Configure un valor `Max-eDTU` o `MaxvCore` para las bases de datos que sea inferior al recuento de eDTU o núcleos virtuales del grupo como un todo. De este modo, se limita la utilización del almacenamiento de OLTP en memoria en cualquier base de datos del grupo al tamaño correspondiente al número de eDTU.
- Configure un valor `Min-eDTU` o `MinvCore` que sea mayor que 0. Este mínimo garantiza que cada base de datos del grupo tenga la cantidad de almacenamiento de OLTP en memoria disponible que corresponda al valor `Min-eDTU` o `vCore` configurado.

### <a name="changing-service-tiers-of-databases-that-use-in-memory-oltp-technologies"></a>Cambio de los niveles de servicio de las bases de datos que usan tecnologías de OLTP en memoria

Siempre puede actualizar su base de datos o instancia a un plan superior, como de uso General a Crítico para la empresa (o de Estándar a Premium). Solo aumenta la funcionalidad y los recursos disponibles.

Pero cambiar a un nivel inferior puede repercutir negativamente en la base de datos. El impacto es especialmente evidente al cambiar de Crítico para la empresa a De uso general (o de Premium a Estándar o Básico) cuando la base de datos contiene objetos de OLTP en memoria. Las tablas optimizadas para memoria no están disponibles después del cambio a una versión anterior (aunque sigan estando visibles). Lo mismo se aplica al reducir el plan de tarifa de un grupo elástico o mover bases de datos con tecnologías en memoria a un grupo elástico Estándar o Básico.

> [!Important]
> OLTP en memoria no se admite en los planes De uso general, Estándar o Básico. Por lo tanto, no es posible mover una base de datos con objetos de OLTP en memoria a los planes Estándar o Básico.

Antes de degradar el plan de tarifa de una base de datos a Estándar o Básico, quite todos los tipos de tabla y las tablas optimizadas para memoria, así como todos los módulos de Transact-SQL compilados de forma nativa. 

*Reducir verticalmente los recursos en el plan Crítico para la empresa*: Los datos de las tablas optimizadas para memoria deben caber en el almacenamiento de OLTP en memoria asociado al plan de la base de datos o Instancia administrada o disponible en el grupo elástico. Si trata de reducir verticalmente el plan o mover la base de datos a un grupo que no disponga de almacenamiento de OLTP en memoria suficiente, la operación no se desarrolla correctamente.

## <a name="in-memory-columnstore"></a>Almacén de columnas en memoria

La tecnología de almacén de columnas en memoria es lo que le permite almacenar y consultar una gran cantidad de datos en las tablas. La tecnología de almacén de columnas usa el formato de almacenamiento de datos basado en columnas y procesamiento de consultas por lotes para lograr hasta 10 veces el rendimiento de las consultas en las cargas de trabajo OLAP con almacenamiento tradicional orientado a filas. También puede lograr ganancias de hasta 10 veces la compresión de datos sobre el tamaño de los datos sin comprimir.
Hay dos tipos de modelos de almacén de columnas que puede usar para organizar los datos:

- **Almacén de columnas en clúster** donde todos los datos en la tabla se organizan con el formato de columnas. En este modelo, todas las filas de la tabla se colocan en un formato de columnas que comprime enormemente los datos y le permite ejecutar informes y consultas analíticas rápidas en la tabla. Según la naturaleza de los datos, el tamaño de los datos puede disminuirse entre 10 y 100 veces. El modelo de almacén de columnas en clúster también permite la ingesta rápida de grandes cantidades de datos (carga masiva), ya que los lotes grandes de datos con más de 100 000 filas se comprimen antes de almacenarse en el disco. Este modelo es una buena elección para los escenarios de almacenamiento de datos clásicos. 
- **Almacén de columnas no en clúster**, donde los datos se almacenan en una tabla de almacén de filas tradicional y hay un índice en formato de almacén de columnas que se usa para las consultas analíticas. Este modelo permite el procesamiento analítico-transaccional híbrido (HTAP): la capacidad de ejecutar análisis en tiempo real de alto rendimiento en una carga de trabajo transaccional. Las consultas OLTP se ejecutan en la tabla de almacén de filas que está optimizada para tener acceso a un pequeño conjunto de filas, mientras que las consultas OLAP se ejecutan en el índice de almacén de columnas, que es la mejor opción para exámenes y análisis. El optimizador de consultas de Azure SQL Database elige dinámicamente el formato de almacén de filas o almacén de columnas en función de la consulta. Los índices de almacén de columnas no en clúster no reducen el tamaño de los datos, ya que el conjunto de datos original se conserva en la tabla de almacén de filas original sin realizar ningún cambio. Sin embargo, el tamaño del índice de almacén de columnas adicional debe ser, en orden de magnitud, menor que el índice de árbol B equivalente.

> [!Note]
> La tecnología de almacén de columnas en memoria conserva únicamente los datos que se necesitan para su procesamiento en la memoria, mientras que los datos que no quepan en la memoria se almacenan en disco. Por lo tanto, la cantidad de datos en las estructuras de almacén de columnas en memoria pueden superar la cantidad de memoria disponible. 

Vídeo detallado sobre la tecnología:

- [Columnstore Index: In-Memory Analytics Videos from Ignite 2016](https://blogs.msdn.microsoft.com/sqlserverstorageengine/20../../columnstore-index-in-memory-analytics-i-e-columnstore-index-videos-from-ignite-2016/) (Índice de almacén de columnas: Vídeos de análisis en memoria de Ignite 2016)

### <a name="data-size-and-storage-for-columnstore-indexes"></a>Almacenamiento y tamaño de datos para los índices de almacén de columnas

No se requiere que los índices de almacén de columnas quepan en la memoria. Por lo tanto, el único límite del tamaño de los índices es el tamaño máximo global de la base de datos, que está documentado en los artículos sobre el [modelo de compra basado en DTU](sql-database-service-tiers-dtu.md) y el [modelo de compra basado en núcleo virtual](sql-database-service-tiers-vcore.md).

Al utilizar los índices de almacén de columnas en clúster, se emplea una compresión de columnas para el almacenamiento de la tabla base. Esta compresión puede reducir considerablemente el consumo de almacenamiento de sus datos de usuario, lo que significa que la base de datos podrá albergar más información. Y es posible aumentar este compresión aún más con la [compresión de archivo de columnas](https://msdn.microsoft.com/library/cc280449.aspx#using-columnstore-and-columnstore-archive-compression). La cantidad de compresión que puede lograr depende de la naturaleza de los datos, pero no es raro obtener una compresión que reduzca el tamaño en 10 veces.

Por ejemplo, si tiene una base de datos con el tamaño máximo de 1 terabyte (TB) y logra una compresión de 10 veces con índices de almacén de columnas, puede incluir un total de 10 TB de datos de usuario en la base de datos.

Al utilizar índices de almacén de columnas no agrupados, la tabla base sigue almacenada en el formato de almacenamiento de filas tradicional. Por lo tanto, el ahorro de almacenamiento no es tan grande como con los índices de almacén de columnas agrupados. Pero si sustituye diversos índices no agrupados tradicionales por un único índice de almacén de columnas, aún podrá obtener un ahorro global en el espacio de almacenamiento de la tabla.

### <a name="changing-service-tiers-of-databases-containing-columnstore-indexes"></a>Cambio de los niveles de servicio de las bases de datos que contienen índices de almacén de columnas

*Cambiar una base de datos única a un plan Básico o Estándar* no sería posible si el nivel de destino está por debajo de S3. Los índices de almacén de columnas solo se admiten en los planes de tarifa Premium, Crítico para la empresa y Estándar (S3 y superior), no en el plan Básico. Si se cambia la base de datos a un nivel inferior incompatible, el índice de almacén de columnas dejará de estar disponible. El sistema mantiene el índice de almacén de columnas, pero no aprovecha el índice. Si, más tarde, vuelve a actualizar a un plan o nivel superior compatible, el almacén de columnas estará listo inmediatamente para volver a sacar el máximo partido.

Si tiene un índice de almacén de columnas **en clúster**, toda la tabla deja de estar disponible después del cambio a un nivel inferior. Por lo tanto, se recomienda quitar todos los índices de almacén de columnas *en clúster* antes de cambiar la base de datos a un nivel o plan inferior incompatible.

> [!Note]
> La instancia administrada admite índices de tipo ColumnStore en todos los niveles.

<a id="install_oltp_manuallink" name="install_oltp_manuallink"></a>

## <a name="next-steps"></a>Pasos siguientes

- [Inicio rápido 1: Tecnologías de OLTP en memoria para acelerar el rendimiento de Transact-SQL](https://msdn.microsoft.com/library/mt694156.aspx)
- [Uso de OLTP en memoria para mejorar el rendimiento de las aplicaciones en Azure SQL](sql-database-in-memory-oltp-migration.md)
- [Supervisión del almacenamiento de OLTP en memoria](sql-database-in-memory-oltp-monitoring.md)
- [Pruebe las características en memoria de Azure SQL Database](sql-database-in-memory-sample.md)

## <a name="additional-resources"></a>Recursos adicionales

### <a name="deeper-information"></a>Información más detallada

- [Más información sobre cómo Quorum duplica cargas de trabajo clave de las bases de datos a la vez que reduce las DTU en un 70 % con OLTP en memoria en SQL Database](https://customers.microsoft.com/story/quorum-doubles-key-databases-workload-while-lowering-dtu-with-sql-database)
- [In-Memory OLTP in Azure SQL Database Blog Post](https://azure.microsoft.com/blog/in-memory-oltp-in-azure-sql-database/) (OLTP en memoria en la entrada de Blog de base de datos de SQL de Azure)
- [Más información sobre OLTP en memoria](https://msdn.microsoft.com/library/dn133186.aspx)
- [Más información sobre los índices de almacén de columnas](https://msdn.microsoft.com/library/gg492088.aspx)
- [Más información sobre los análisis operativos en tiempo real](https://msdn.microsoft.com/library/dn817827.aspx)
- Consulte [Common Workload Patterns and Migration Considerations](https://msdn.microsoft.com/library/dn673538.aspx) (Patrones de cargas de trabajo comunes y consideraciones de migración), que describe los patrones de carga de trabajo donde In-Memory OLTP normalmente proporciona importantes mejoras de rendimiento.

### <a name="application-design"></a>Diseño de aplicación

- [In-Memory OLTP (optimización In-Memory)](https://msdn.microsoft.com/library/dn133186.aspx)
- [Uso de OLTP en memoria para mejorar el rendimiento de las aplicaciones en Azure SQL](sql-database-in-memory-oltp-migration.md)

### <a name="tools"></a>Herramientas

- [Azure Portal](https://portal.azure.com/)
- [SQL Server Management Studio (SSMS)](https://msdn.microsoft.com/library/mt238290.aspx)
- [SQL Server Data Tools (SSDT)](https://msdn.microsoft.com/library/mt204009.aspx)
