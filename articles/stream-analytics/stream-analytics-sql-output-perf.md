---
title: Salida de Azure Stream Analytics a Azure SQL Database
description: Obtenga información sobre cómo enviar datos a SQL Azure desde Azure Stream Analytics y lograr mayores tasas de rendimiento de escritura.
author: chetanmsft
ms.author: chetang
ms.reviewer: mamccrea
ms.service: stream-analytics
ms.topic: conceptual
ms.date: 03/18/2019
ms.openlocfilehash: b760ad03318b3c31b39b6470251847150dc5a70a
ms.sourcegitcommit: 927dd0e3d44d48b413b446384214f4661f33db04
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88869429"
---
# <a name="azure-stream-analytics-output-to-azure-sql-database"></a>Salida de Azure Stream Analytics a Azure SQL Database

En este artículo se tratan las sugerencias para lograr un mejor rendimiento de escritura al cargar datos en Azure SQL Database mediante Azure Stream Analytics.

La salida de SQL en Azure Stream Analytics admite la escritura en paralelo como una opción. Esta opción permite topologías de trabajo [totalmente paralelas](stream-analytics-parallelization.md#embarrassingly-parallel-jobs), donde varias particiones de salida escriben en la tabla de destino en paralelo. No obstante, es posible que habilitar esta opción en Azure Stream Analytics no sea suficiente para lograr un mayor rendimiento, ya que depende significativamente la configuración de la base de datos y el esquema de tabla. La elección de índices, clave de agrupación en clústeres, factor de relleno de índice y compresión influye en el tiempo de carga de las tablas. Para más información acerca de cómo optimizar la base de datos para mejorar el rendimiento de carga y las consultas de acuerdo con las pruebas comparativas internas, consulte la [guía sobre el rendimiento de SQL Database](../azure-sql/database/performance-guidance.md). No se garantiza el orden de escritura al escribir en paralelo en una instancia de SQL Database.

A continuación se indican algunas configuraciones de cada servicio que pueden ayudar a mejorar el rendimiento global de la solución.

## <a name="azure-stream-analytics"></a>Azure Stream Analytics

- **Inherit Partitioning** (Heredar la creación de particiones): esta opción de configuración de salida de SQL habilita la herencia del esquema de partición de la entrada o el paso anterior de la consulta. Con esta opción habilitada, al escribir en una tabla basada en disco y tener una topología [totalmente paralela](stream-analytics-parallelization.md#embarrassingly-parallel-jobs) para su trabajo, espere ver un rendimiento superior. Esta creación de particiones se produce automáticamente para muchas otras [salidas](stream-analytics-parallelization.md#partitions-in-inputs-and-outputs). El bloqueo de tabla (TABLOCK) también se deshabilita para las inserciones masivas realizadas con esta opción.

> [!NOTE] 
> Cuando hay más de 8 particiones de entrada, es posible que heredar el esquema de partición de entrada no sea una opción adecuada. Se ha observado este límite superior en una tabla con una columna de identidad única y un índice agrupado. En este caso, considere el uso de [INTO](https://docs.microsoft.com/stream-analytics-query/into-azure-stream-analytics#into-shard-count) 8 en la consulta para especificar explícitamente el número de redactores de salida. Según el esquema y la elección de índices, sus observaciones pueden variar.

- **Tamaño del lote**: la configuración de salida de SQL le permite especificar el tamaño del lote máximo en una salida de SQL de Azure Stream Analytics según la naturaleza de la carga de trabajo o la tabla de destino. El tamaño del lote es el número máximo de registros que se envía con cada transacción de inserción masiva. En los índices de almacén de columnas agrupados, los tamaños de lote en torno a [100 000](https://docs.microsoft.com/sql/relational-databases/indexes/columnstore-indexes-data-loading-guidance) permiten más paralelización, registros mínimos y optimizaciones de bloqueo. En las tablas basadas en disco, un tamaño de 10 000 (valor predeterminado) o inferior puede ser óptimo para su solución, ya que los tamaños de lote mayores pueden desencadenar la extensión de bloqueo durante las inserciones masivas.

- **Input Message Tuning** (Optimización de mensajes de entrada): si ha realizado la optimización con la herencia de particiones y el tamaño del lote, aumentar el número de eventos de entrada por mensaje y por partición ayuda a aumentar aún más el rendimiento de escritura. La opción de optimización de mensajes de entrada permite que los tamaños de lote en Azure Stream Analytics alcancen el tamaño del lote especificado, lo que mejora el rendimiento. Esto puede lograrse mediante la [compresión](stream-analytics-define-inputs.md) o el aumento de los tamaños de los mensajes de entrada en EventHub o Blob.

## <a name="sql-azure"></a>SQL Azure

- **Partitioned Table and Indexes** (Tabla e índices con particiones): con una tabla SQL [con particiones](https://docs.microsoft.com/sql/relational-databases/partitions/partitioned-tables-and-indexes?view=sql-server-2017) e índices con particiones en la tabla con la misma columna que la clave de partición (por ejemplo, PartitionId), puede reducir significativamente las contenciones entre particiones durante las operaciones de escritura. Para una tabla con particiones, deberá crear un [función de partición](https://docs.microsoft.com/sql/t-sql/statements/create-partition-function-transact-sql?view=sql-server-2017) y un [esquema de partición](https://docs.microsoft.com/sql/t-sql/statements/create-partition-scheme-transact-sql?view=sql-server-2017) en el grupo de archivos PRIMARY. Esto también aumentará la disponibilidad de los datos existentes mientras se cargan datos nuevos. Es posible que se haya alcanzado el límite de E/S de registro basado en el número de particiones, que se puede incrementar mediante la actualización de la SKU.

- **Avoid unique key violations** (Evitar infracciones de clave únicas): si recibe [varios mensajes de advertencia de infracción de claves](stream-analytics-troubleshoot-output.md#key-violation-warning-with-azure-sql-database-output) en el registro de actividad de Azure Stream Analytics, asegúrese de que el trabajo no se ve afectado por las infracciones de restricción únicas que pueden producirse durante los casos de recuperación. Para evitarlo, puede definir la opción [IGNORE\_DUP\_KEY](stream-analytics-troubleshoot-output.md#key-violation-warning-with-azure-sql-database-output) en sus índices.

## <a name="azure-data-factory-and-in-memory-tables"></a>Azure Data Factory y tablas en memoria

- **In-Memory Table as temp table** (Tabla en memoria como tabla temporal): [las tablas en memoria](/sql/relational-databases/in-memory-oltp/in-memory-oltp-in-memory-optimization) permiten cargas de datos de alta velocidad, pero los datos deben caber en la memoria. Las pruebas comparativas muestran que la carga masiva de una tabla en memoria a una tabla basada en disco es aproximadamente 10 veces más rápida que la inserción masiva directamente con un único escritor en la tabla basada en disco con una columna de identidad y un índice agrupado. Para aprovechar este rendimiento de inserción masiva, configure un [trabajo de copia mediante Azure Data Factory](../data-factory/connector-azure-sql-database.md) que copie datos de la tabla en memoria a la tabla basada en disco.

## <a name="avoiding-performance-pitfalls"></a>Evitación de los errores de rendimiento
La inserción masiva de datos es un proceso más rápido que la carga de datos con inserciones sencillas, porque se evita la sobrecarga repetida que supone transferir los datos, analizar la instrucción insert, ejecutar la instrucción y emitir un registro de transacciones. En su lugar, se usa una ruta de acceso más eficaz en el motor de almacenamiento para transmitir los datos. Sin embargo, el costo de la configuración de esta ruta de acceso es mucho mayor que una sola instrucción insert en una tabla basada en disco. El punto de equilibrio es normalmente alrededor de 100 filas; para cantidades superiores, la carga masiva es casi siempre más eficaz. 

Si la tasa de eventos entrantes es baja, puede crear fácilmente tamaños de lote con menos de 100 filas, lo que convierte en ineficaz la inserción masiva y utiliza demasiado espacio en disco. Para solucionar esta limitación, puede realizar una de estas acciones:
* Cree un [desencadenador](/sql/t-sql/statements/create-trigger-transact-sql) INSTEAD OF para usar la inserción sencilla para cada fila.
* Use una tabla temporal en memoria como se describe en la sección anterior.

Otro escenario similar ocurre cuando se escribe en un índice de almacén de columnas no en clúster, donde las inserciones masivas más pequeñas pueden crear demasiados segmentos, lo cual puede bloquear el índice. En este caso, la recomendación es usar un índice de almacén de columnas agrupado en su lugar.

## <a name="summary"></a>Resumen

En resumen, con la característica de salida con particiones en Azure Stream Analytics para la salida SQL, la paralelización alineada de su trabajo con una tabla con particiones en SQL Azure debería proporcionarle mejoras de rendimiento significativas. Aprovechar Azure Data Factory para orquestar el movimiento de datos de una tabla en memoria en las tablas basadas en disco puede proporcionar mejoras de rendimiento de orden de magnitud. Si es factible, mejorar la densidad de mensajes también puede ser un factor importante para mejorar el rendimiento global.