---
title: 'Procedimientos recomendados de rendimiento e instrucciones de configuración: Azure SQL Edge'
description: Información sobre procedimientos recomendados de rendimiento e instrucciones de configuración en Azure SQL Edge
keywords: SQL Edge, retención de datos
services: sql-edge
ms.service: sql-edge
ms.topic: conceptual
author: SQLSourabh
ms.author: sourabha
ms.reviewer: sstein
ms.date: 09/22/2020
ms.openlocfilehash: 35985404d5ac97940c324c3ad7f7d46c959b4902
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90932565"
---
# <a name="performance-best-practices-and-configuration-guidelines"></a>Procedimientos recomendados de rendimiento e instrucciones de configuración

Azure SQL Edge ofrece varias características y funcionalidades que se pueden usar para mejorar el rendimiento de la implementación de SQL Edge. En este artículo se proporcionan algunos procedimientos recomendados y sugerencias para maximizar el rendimiento. 

## <a name="best-practices"></a>Procedimientos recomendados 

### <a name="configure-multiple-tempdb-data-files"></a>Configuración de varios archivos de datos de tempdb

Azure SQL Edge crea de forma predeterminada un solo archivo de datos tempdb como parte de la inicialización del contenedor. Se recomienda que considere la posibilidad de crear varios archivos de datos tempdb después de la implementación. Para obtener más información, consulte las instrucciones del artículo [Recomendaciones para reducir la contención de asignación en la base de datos tempdb de SQL Server](https://support.microsoft.com/help/2154845/recommendations-to-reduce-allocation-contention-in-sql-server-tempdb-d).

### <a name="use-clustered-columnstore-indexes-where-possible"></a>Uso de índices de almacén de columnas agrupados siempre que sea posible

Los dispositivos IoT y Edge tienden a generar un gran volumen de datos que normalmente se agregan durante un período de tiempo para su análisis. Para el análisis rara vez se usan filas de datos individuales. Los índices de almacén de columnas son idóneos para almacenar y consultar este tipo de conjuntos de datos grandes. Este índice usa el almacenamiento de datos basado en columnas y el procesamiento de consultas para lograr ganancias de rendimiento de las consultas de hasta 10 veces superior al almacenamiento tradicional orientado a filas. También puede lograr ganancias de hasta 10 veces la compresión de datos sobre el tamaño de los datos sin comprimir. Para obtener más información, vea [Índices de almacén de columnas](https://docs.microsoft.com/sql/relational-databases/indexes/columnstore-indexes-overview).

Además, otras características de Azure SQL Edge como el streaming y la retención de datos se benefician de las optimizaciones de almacén de columnas relacionadas con la inserción y eliminación de datos. 

### <a name="simple-recovery-model"></a>Modelo de recuperación simple

Como el almacenamiento se puede restringir en dispositivos perimetrales, en todas las bases de datos de usuario de Azure SQL Edge se usa de forma predeterminada el modelo de recuperación simple. Este modelo recupera de forma automática el espacio de registro para mantener al mínimo los requisitos de espacio, lo que elimina, en esencia, la necesidad de administrar el espacio del registro de transacciones. Esto puede resultar útil en dispositivos perimetrales con almacenamiento limitado disponible. Para obtener más información sobre el modelo de recuperación simple y otros disponibles, vea [Modelos de recuperación](https://docs.microsoft.com/sql/relational-databases/backup-restore/recovery-models-sql-server).

El modelo de recuperación simple no admite operaciones como el trasvase de registros y las de restauración a un momento dado, para las que se necesitan copias de seguridad del registro de transacciones.  

## <a name="advanced-configuration"></a>Configuración avanzada 

### <a name="setting-memory-limits"></a>Establecimiento de límites de memoria

Azure SQL Edge admite hasta 64 GB de memoria para el grupo de búferes, mientras que los procesos satélite que se ejecutan en el contenedor de SQL Edge pueden necesitar memoria adicional. En dispositivos perimetrales más pequeños con memoria limitada, es aconsejable limitar la memoria disponible para los contenedores de SQL Edge, de modo que el host de Docker y otros procesos o módulos perimetrales puedan funcionar correctamente. La memoria total disponible para SQL Edge se puede controlar mediante los mecanismos siguientes. 

- Limitación de la memoria disponible para los contenedores de SQL Edge: la memoria total disponible para el contenedor de SQL Edge se puede limitar mediante la opción de configuración de tiempo de ejecución del contenedor `--memory`. Para obtener más información sobre la limitación de la memoria disponible para el contenedor de SQL Edge, vea [Opciones de tiempo de ejecución con memoria, CPU y GPU](https://docs.docker.com/config/containers/resource_constraints/).

- Limitación de la memoria disponible para el proceso SQL dentro del contenedor: de forma predeterminada, el proceso SQL dentro del contenedor solo usa el 80 % de la RAM física disponible. Para la mayoría de las implementaciones, la configuración predeterminada será suficiente. Pero puede haber escenarios en los que se necesite memoria adicional para el streaming de datos y los procesos de ONNX que se ejecutan dentro de los contenedores de SQL Edge. En estos escenarios, la memoria disponible para el proceso SQL se puede controlar mediante el valor `memory.memorylimitmb` del archivo mssql-conf. Para obtener más información, vea [Configuración mediante el archivo mssql-conf](configure.md#configure-by-using-an-mssqlconf-file).

Al establecer los límites de memoria, tenga cuidado de no usar un valor demasiado bajo. Si no establece memoria suficiente para el proceso SQL, el rendimiento de SQL puede verse afectado gravemente.

### <a name="delayed-durability"></a>Durabilidad diferida

En Azure SQL Edge, las transacciones pueden ser totalmente durables (el valor predeterminado de SQL Server), o bien durables diferidas (también conocidas como confirmaciones diferidas).

Las confirmaciones de transacciones totalmente durables son sincrónicas y notifican una instrucción COMMIT como correcta para devolver el control al cliente únicamente tras escribir en disco los registros de la transacción. Las confirmaciones de transacciones durables diferidas son asincrónicas y notifican una instrucción COMMIT como correcta antes de escribir en disco los registros de la transacción. Para que una transacción sea durable, es necesario escribir las entradas del registro de transacciones en el disco. Las transacciones durables diferidas pasan a ser durables cuando las entradas del registro de transacciones se vacían en el disco. 

En las implementaciones en las que se puede tolerar **cierta pérdida de datos** o en dispositivos perimetrales con almacenamiento lento, se puede usar la durabilidad diferida para optimizar la ingesta de datos y la limpieza basada en la retención de datos. Para saber más, vea [Control de la durabilidad de las transacciones](https://docs.microsoft.com/sql/relational-databases/logs/control-transaction-durability).


### <a name="linux-os-configurations"></a>Configuraciones del sistema operativo Linux 

Considere la posibilidad de usar las siguientes opciones de [configuración del sistema operativo Linux](https://docs.microsoft.com/sql/linux/sql-server-linux-performance-best-practices#linux-os-configuration) para experimentar el mejor rendimiento en una instalación de SQL.







