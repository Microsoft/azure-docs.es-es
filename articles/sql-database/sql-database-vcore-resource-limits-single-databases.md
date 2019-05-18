---
title: Límites de recursos basados en núcleos virtuales de Azure SQL Database para bases de datos únicas | Microsoft Docs
description: En esta página se describen algunos límites de recursos basados en núcleos virtuales comunes para una base de datos única en Azure SQL Database.
services: sql-database
ms.service: sql-database
ms.subservice: single-database
ms.custom: ''
ms.devlang: ''
ms.topic: conceptual
author: stevestein
ms.author: sstein
ms.reviewer: ''
manager: craigg
ms.date: 04/22/2019
ms.openlocfilehash: 15aedaa7eedad4940945e9b1e7eff0e1780da796
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65790399"
---
# <a name="resource-limits-for-single-databases-using-the-vcore-based-purchasing-model"></a>Límites de recursos para las bases de datos únicas mediante el modelo de compra basado en núcleos virtuales

En este artículo se proporcionan los límites de recursos detallados para bases de datos únicas de Azure SQL Database que utilizan el modelo de compra basado en núcleo virtual.

Para conocer los límites del modelo de compra basado en DTU para las bases de datos en un servidor de SQL Database, consulte la [Overview of resource limits on a SQL Database server](sql-database-resource-limits-database-server.md) (Introducción a los límites de recursos en un servidor de SQL Database).

> [!IMPORTANT]
> En algunas circunstancias, puede que deba reducir una base de datos para reclamar el espacio no utilizado. Para obtener más información, consulte [Administración del espacio de archivo en Azure SQL Database](sql-database-file-space-management.md).

Puede establecer el nivel de servicio, el tamaño de proceso y la cantidad de almacenamiento para una base de datos única mediante [Azure Portal](sql-database-single-databases-manage.md#manage-an-existing-sql-database-server), [Transact-SQL](sql-database-single-databases-manage.md#transact-sql-manage-sql-database-servers-and-single-databases), [PowerShell](sql-database-single-databases-manage.md#powershell-manage-sql-database-servers-and-single-databases), la [CLI de Azure](sql-database-single-databases-manage.md#azure-cli-manage-sql-database-servers-and-single-databases) o la [API REST](sql-database-single-databases-manage.md#rest-api-manage-sql-database-servers-and-single-databases).

> [!IMPORTANT]
> Para escalar de instrucciones y otras consideraciones, consulte [escalar una base de datos](sql-database-single-database-scale.md).

## <a name="general-purpose-service-tier-storage-sizes-and-compute-sizes"></a>Nivel de servicio Uso general: tamaños de almacenamiento y tamaños de proceso

### <a name="general-purpose-service-tier-generation-4-compute-platform-part-1"></a>Nivel de servicio Uso general: Plataforma de procesos de generación 4 (parte 1)

|Tamaño de proceso|GP_Gen4_1|GP_Gen4_2|GP_Gen4_3|GP_Gen4_4|GP_Gen4_5|GP_Gen4_6
|:--- | --: |--: |--: |--: |--: |--: |
|Generación de H/W|4|4|4|4|4|4|
|Núcleos virtuales|1|2|3|4|5|6|
|Memoria (GB)|7|14|21|28|35|42|
|Compatible con almacén de columnas|Sí|Sí|Sí|Sí|Sí|Sí|
|Almacenamiento OLTP en memoria (GB)|N/D|N/D|N/D|N/D|N/D|N/D|
|Tamaño máximo de datos (GB)|1024|1024|1024|1536|1536|1536|
|Tamaño máximo de registro (GB)|307|307|307|461|461|461|
|Tamaño de TempDB (GB)|32|64|96|128|160|192|
|Tipo de almacenamiento|Premium Storage (remoto)|Premium Storage (remoto)|Premium Storage (remoto)|Premium Storage (remoto)|Premium Storage (remoto)|Premium Storage (remoto)|
|Latencia de E/S (aproximada)|5-7 ms (escritura)<br>5-10 ms (lectura)|5-7 ms (escritura)<br>5-10 ms (lectura)|5-7 ms (escritura)<br>5-10 ms (lectura)|5-7 ms (escritura)<br>5-10 ms (lectura)|5-7 ms (escritura)<br>5-10 ms (lectura)|5-7 ms (escritura)<br>5-10 ms (lectura)|
|IOPS de destino (64 KB)|500|1000|1.500|2000|2.500|3000|
|Registro de los límites de frecuencia (MBps)|3,75|7.5|11,25|15|18.75|22.5|
|Cantidad máxima de trabajos (solicitudes) simultáneos|200|400|600|800|1000|1200|
|Sesiones máximas permitidas|30000|30000|30000|30000|30000|30000|
|Número de réplicas|1|1|1|1|1|1|
|AZ múltiple|N/D|N/D|N/D|N/D|N/D|N/D|
|Escalado horizontal de lectura|N/D|N/D|N/D|N/D|N/D|N/D|
|Almacenamiento de copia de seguridad incluido|1X el tamaño de base de datos|1X el tamaño de base de datos|1X el tamaño de base de datos|1X el tamaño de base de datos|1X el tamaño de base de datos|1X el tamaño de base de datos|

### <a name="general-purpose-service-tier-generation-4-compute-platform-part-2"></a>Nivel de servicio Uso general: Plataforma de procesos de generación 4 (parte 2)

|Tamaño de proceso|GP_Gen4_7|GP_Gen4_8|GP_Gen4_9|GP_Gen4_10|GP_Gen4_16|GP_Gen4_24
|:--- | --: |--: |--: |--: |--: |--: |
|Generación de H/W|4|4|4|4|4|4|
|Núcleos virtuales|7|8|9|10|16|24|
|Memoria (GB)|49|56|63|70|112|168|
|Compatible con almacén de columnas|Sí|Sí|Sí|Sí|Sí|Sí|
|Almacenamiento OLTP en memoria (GB)|N/D|N/D|N/D|N/D|N/D|N/D|
|Tamaño máximo de datos (GB)|1536|3072|3072|3072|4096|4096|
|Tamaño máximo de registro (GB)|461|922|922|922|1229|1229|
|Tamaño de TempDB (GB)|224|256|288|320|384|384|
|Tipo de almacenamiento|Premium Storage (remoto)|Premium Storage (remoto)|Premium Storage (remoto)|Premium Storage (remoto)|Premium Storage (remoto)|Premium Storage (remoto)|
|Latencia de E/S (aproximada)|5-7 ms (escritura)<br>5-10 ms (lectura)|5-7 ms (escritura)<br>5-10 ms (lectura)|5-7 ms (escritura)<br>5-10 ms (lectura)|5-7 ms (escritura)<br>5-10 ms (lectura)|5-7 ms (escritura)<br>5-10 ms (lectura)|5-7 ms (escritura)<br>5-10 ms (lectura)
|IOPS de destino (64 KB)|3500|4000|4500|5000|7000|7000|
|Registro de los límites de frecuencia (MBps)|26.25|30|30|30|30|30|
|Cantidad máxima de trabajos (solicitudes) simultáneos|1400|1600|1.800|2000|3200|4800|
|Sesiones máximas permitidas|30000|30000|30000|30000|30000|30000|
|Número de réplicas|1|1|1|1|1|1|
|AZ múltiple|N/D|N/D|N/D|N/D|N/D|N/D|
|Escalado horizontal de lectura|N/D|N/D|N/D|N/D|N/D|N/D|
|Almacenamiento de copia de seguridad incluido|1X el tamaño de base de datos|1X el tamaño de base de datos|1X el tamaño de base de datos|1X el tamaño de base de datos|1X el tamaño de base de datos|1X el tamaño de base de datos|

### <a name="general-purpose-service-tier-generation-5-compute-platform-part-1"></a>Nivel de servicio Uso general: Plataforma de procesos de generación 5 (parte 1)

|Tamaño de proceso|GP_Gen5_2|GP_Gen5_4|GP_Gen5_6|GP_Gen5_8|GP_Gen5_10|GP_Gen5_12|GP_Gen5_14|
|:--- | --: |--: |--: |--: |---: | --: |--: |
|Generación de H/W|5|5|5|5|5|5|5|
|Núcleos virtuales|2|4|6|8|10|12|14|
|Memoria (GB)|10.2|20.4|30.6|40.8|51|61.2|71.4|
|Compatible con almacén de columnas|Sí|Sí|Sí|Sí|Sí|Sí|Sí|
|Almacenamiento OLTP en memoria (GB)|N/D|N/D|N/D|N/D|N/D|N/D|N/D|
|Tamaño máximo de datos (GB)|1024|1024|1536|1536|1536|3072|3072|
|Tamaño máximo de registro (GB)|307|307|307|461|461|461|461|
|Tamaño de TempDB (GB)|64|128|192|256|320|384|384|
|Tipo de almacenamiento|Premium Storage (remoto)|Premium Storage (remoto)|Premium Storage (remoto)|Premium Storage (remoto)|Premium Storage (remoto)|Premium Storage (remoto)|Premium Storage (remoto)|
|Latencia de E/S (aproximada)|5-7 ms (escritura)<br>5-10 ms (lectura)|5-7 ms (escritura)<br>5-10 ms (lectura)|5-7 ms (escritura)<br>5-10 ms (lectura)|5-7 ms (escritura)<br>5-10 ms (lectura)|5-7 ms (escritura)<br>5-10 ms (lectura)|5-7 ms (escritura)<br>5-10 ms (lectura)|5-7 ms (escritura)<br>5-10 ms (lectura)|
|IOPS de destino (64 KB)|1000|2000|3000|4000|5000|6000|7000|
|Registro de los límites de frecuencia (MBps)|3,75|7.5|11,25|15|18.75|22.5|26.25|
|Cantidad máxima de trabajos (solicitudes) simultáneos|200|400|600|800|1000|1200|1400|
|Sesiones máximas permitidas|30000|30000|30000|30000|30000|30000|30000|
|Número de réplicas|1|1|1|1|1|1|1|
|AZ múltiple|N/D|N/D|N/D|N/D|N/D|N/D|N/D|
|Escalado horizontal de lectura|N/D|N/D|N/D|N/D|N/D|N/D|N/D|
|Almacenamiento de copia de seguridad incluido|1X el tamaño de base de datos|1X el tamaño de base de datos|1X el tamaño de base de datos|1X el tamaño de base de datos|1X el tamaño de base de datos|1X el tamaño de base de datos|1X el tamaño de base de datos|

### <a name="general-purpose-service-tier-generation-5-compute-platform-part-2"></a>Nivel de servicio Uso general: Plataforma de procesos de generación 5 (parte 2)

|Tamaño de proceso|GP_Gen5_16|GP_Gen5_18|GP_Gen5_20|GP_Gen5_24|GP_Gen5_32|GP_Gen5_40|GP_Gen5_80|
|:--- | --: |--: |--: |--: |---: | --: |--: |
|Generación de H/W|5|5|5|5|5|5|5|
|Núcleos virtuales|16|18|20|24|32|40|80|
|Memoria (GB)|81.6|91.8|102|122.4|163.2|204|408|
|Compatible con almacén de columnas|Sí|Sí|Sí|Sí|Sí|Sí|Sí|
|Almacenamiento OLTP en memoria (GB)|N/D|N/D|N/D|N/D|N/D|N/D|N/D|
|Tamaño máximo de datos (GB)|3072|3072|3072|4096|4096|4096|4096|
|Tamaño máximo de registro (GB)|922|922|922|1229|1229|1229|1229|
|Tamaño de TempDB (GB)|384|384|384|384|384|384|384|
|Tipo de almacenamiento|Premium Storage (remoto)|Premium Storage (remoto)|Premium Storage (remoto)|Premium Storage (remoto)|Premium Storage (remoto)|Premium Storage (remoto)|Premium Storage (remoto)|
|Latencia de E/S (aproximada)|5-7 ms (escritura)<br>5-10 ms (lectura)|5-7 ms (escritura)<br>5-10 ms (lectura)|5-7 ms (escritura)<br>5-10 ms (lectura)|5-7 ms (escritura)<br>5-10 ms (lectura)|5-7 ms (escritura)<br>5-10 ms (lectura)|5-7 ms (escritura)<br>5-10 ms (lectura)|5-7 ms (escritura)<br>5-10 ms (lectura)|
|IOPS de destino (64 KB)|7000|7000|7000|7000|7000|7000|7000|
|Registro de los límites de frecuencia (MBps)|30|30|30|30|30|30|30|
|Cantidad máxima de trabajos (solicitudes) simultáneos|1600|1.800|2000|2400|3200|4000|8000|
|Sesiones máximas permitidas|30000|30000|30000|30000|30000|30000|30000|
|Número de réplicas|1|1|1|1|1|1|1|
|AZ múltiple|N/D|N/D|N/D|N/D|N/D|N/D|N/D|
|Escalado horizontal de lectura|N/D|N/D|N/D|N/D|N/D|N/D|N/D|
|Almacenamiento de copia de seguridad incluido|1X el tamaño de base de datos|1X el tamaño de base de datos|1X el tamaño de base de datos|1X el tamaño de base de datos|1X el tamaño de base de datos|1X el tamaño de base de datos|1X el tamaño de base de datos|

### <a name="serverless-compute-tier"></a>Nivel de servicio de informática sin servidor

El [nivel de proceso sin servidor](sql-database-serverless.md) está en versión preliminar y es sólo para bases de datos únicas mediante la compra de núcleos virtuales del modelo.

#### <a name="generation-5-compute-platform"></a>Plataforma de procesos de generación 5

|Tamaño de proceso|GP_S_Gen5_1|GP_S_Gen5_2|GP_S_Gen5_4|
|:--- | --: |--: |--: |
|Generación de H/W|5|5|5|
|Núcleos virtuales mín. máx.|0.5-1|0.5-2|0.5-4|
|Memoria mínima / máxima (GB)|2.02-3|2.05-6|2.10-12|
|Retraso de pausa automática min (horas)|6|6|6|
|Compatible con almacén de columnas|Sí|Sí|Sí|
|Almacenamiento OLTP en memoria (GB)|N/D|N/D|N/D|
|Tamaño máximo de datos (GB)|512|1024|1024|
|Tamaño máximo de registro (GB)|12|24|48|
|Tamaño de TempDB (GB)|32|64|128|
|Tipo de almacenamiento|Premium Storage (remoto)|Premium Storage (remoto)|Premium Storage (remoto)|
|Latencia de E/S (aproximada)|5-7 ms (escritura)<br>5-10 ms (lectura)|5-7 ms (escritura)<br>5-10 ms (lectura)|5-7 ms (escritura)<br>5-10 ms (lectura)|
|IOPS de destino (64 KB)|250|500|1000|
|Registro de los límites de frecuencia (MBps)|2.5|5.6|10|
|Cantidad máxima de trabajos (solicitudes) simultáneos|75|150|300|
|Sesiones máximas permitidas|30000|30000|30000|
|Número de réplicas|1|1|1|
|AZ múltiple|N/D|N/D|N/D|
|Escalado horizontal de lectura|N/D|N/D|N/D|
|Almacenamiento de copia de seguridad incluido|1X el tamaño de base de datos|1X el tamaño de base de datos|1X el tamaño de base de datos|

## <a name="business-critical-service-tier-for-provisioned-compute-tier"></a>Nivel de servicio crítico de negocio para el nivel de proceso aprovisionada

### <a name="business-critical-service-tier-generation-4-compute-platform-part-1"></a>Nivel de servicio Crítico para la empresa: Plataforma de procesos de generación 4 (parte 1)

|Tamaño de proceso|BC_Gen4_1|BC_Gen4_2|BC_Gen4_3|BC_Gen4_4|BC_Gen4_5|BC_Gen4_6|
|:--- | --: |--: |--: |--: |--: |--: |
|Generación de H/W|4|4|4|4|4|4|
|Núcleos virtuales|1|2|3|4|5|6|
|Memoria (GB)|7|14|21|28|35|42|
|Compatible con almacén de columnas|Sí|Sí|Sí|Sí|Sí|Sí|
|Almacenamiento OLTP en memoria (GB)|1|2|3|4|5|6|
|Tipo de almacenamiento|SSD local|SSD local|SSD local|SSD local|SSD local|SSD local|
|Tamaño máximo de datos (GB)|650|650|650|650|650|650|
|Tamaño máximo de registro (GB)|195|195|195|195|195|195|
|Tamaño de TempDB (GB)|32|64|96|128|160|192|
|Latencia de E/S (aproximada)|1-2 ms (escritura)<br>1-2 ms (lectura)|1-2 ms (escritura)<br>1-2 ms (lectura)|1-2 ms (escritura)<br>1-2 ms (lectura)|1-2 ms (escritura)<br>1-2 ms (lectura)|1-2 ms (escritura)<br>1-2 ms (lectura)|1-2 ms (escritura)<br>1-2 ms (lectura)|
|IOPS de destino (64 KB)|5000|10000|15000|20000|25000|30000|
|Registro de los límites de frecuencia (MBps)|8|16|24|32|40|48|
|Cantidad máxima de trabajos (solicitudes) simultáneos|200|400|600|800|1000|1200|
|Máximo de inicios de sesión simultáneos|200|400|600|800|1000|1200|
|Sesiones máximas permitidas|30000|30000|30000|30000|30000|30000|
|Número de réplicas|4|4|4|4|4|4|
|AZ múltiple|Sí|Sí|Sí|Sí|Sí|Sí|
|Escalado horizontal de lectura|Sí|Sí|Sí|Sí|Sí|Sí|
|Almacenamiento de copia de seguridad incluido|1X el tamaño de base de datos|1X el tamaño de base de datos|1X el tamaño de base de datos|1X el tamaño de base de datos|1X el tamaño de base de datos|1X el tamaño de base de datos|

### <a name="business-critical-service-tier-generation-4-compute-platform-part-2"></a>Nivel de servicio Crítico para la empresa: Plataforma de procesos de generación 4 (parte 2)

|Tamaño de proceso|BC_Gen4_7|BC_Gen4_8|BC_Gen4_9|BC_Gen4_10|BC_Gen4_16|BC_Gen4_24|
|:--- | --: |--: |--: |--: |--: |--: |
|Generación de H/W|4|4|4|4|4|4|
|Núcleos virtuales|7|8|9|10|16|24|
|Memoria (GB)|49|56|63|70|112|168|
|Compatible con almacén de columnas|Sí|Sí|Sí|Sí|Sí|Sí|
|Almacenamiento OLTP en memoria (GB)|7|8|9.5|11|20|36|
|Tipo de almacenamiento|SSD local|SSD local|SSD local|SSD local|SSD local|SSD local|
|Tamaño máximo de datos (GB)|650|650|650|650|1024|1024|
|Tamaño máximo de registro (GB)|195|195|195|195|307|307|
|Tamaño de TempDB (GB)|224|256|288|320|384|384|
|Latencia de E/S (aproximada)|1-2 ms (escritura)<br>1-2 ms (lectura)|1-2 ms (escritura)<br>1-2 ms (lectura)|1-2 ms (escritura)<br>1-2 ms (lectura)|1-2 ms (escritura)<br>1-2 ms (lectura)|1-2 ms (escritura)<br>1-2 ms (lectura)|1-2 ms (escritura)<br>1-2 ms (lectura)|
|IOPS de destino (64 KB)|35000|40000|45000|50000|80000|120000|
|Registro de los límites de frecuencia (MBps)|56|64|64|64|64|64|
|Cantidad máxima de trabajos (solicitudes) simultáneos|1400|1600|1.800|2000|3200|4800|
|Inicios de sesión máxima simultáneos (solicitudes)|1400|1600|1.800|2000|3200|4800|
|Sesiones máximas permitidas|30000|30000|30000|30000|30000|30000|
|Número de réplicas|4|4|4|4|4|4|
|AZ múltiple|Sí|Sí|Sí|Sí|Sí|Sí|
|Escalado horizontal de lectura|Sí|Sí|Sí|Sí|Sí|Sí|
|Almacenamiento de copia de seguridad incluido|1X el tamaño de base de datos|1X el tamaño de base de datos|1X el tamaño de base de datos|1X el tamaño de base de datos|1X el tamaño de base de datos|1X el tamaño de base de datos|

### <a name="business-critical-service-tier-generation-5-compute-platform-part-1"></a>Nivel de servicio Crítico para la empresa: Plataforma de procesos de generación 5 (parte 1)

|Tamaño de proceso|BC_Gen5_2|BC_Gen5_4|BC_Gen5_6|BC_Gen5_8|BC_Gen5_10|BC_Gen5_12|BC_Gen5_14|
|:--- | --: |--: |--: |--: |---: | --: |--: |
|Generación de H/W|5|5|5|5|5|5|5|
|Núcleos virtuales|2|4|6|8|10|12|14|
|Memoria (GB)|10.2|20.4|30.6|40.8|51|61.2|71.4|
|Compatible con almacén de columnas|Sí|Sí|Sí|Sí|Sí|Sí|Sí|
|Almacenamiento OLTP en memoria (GB)|1571|3142|4.713|6284|8.655|11.026|13.397|
|Tamaño máximo de datos (GB)|1024|1024|1536|1536|1536|3072|3072|
|Tamaño máximo de registro (GB)|307|307|307|461|461|922|922|
|Tamaño de TempDB (GB)|64|128|192|256|320|384|384|
|Tipo de almacenamiento|SSD local|SSD local|SSD local|SSD local|SSD local|SSD local|SSD local|
|Latencia de E/S (aproximada)|1-2 ms (escritura)<br>1-2 ms (lectura)|1-2 ms (escritura)<br>1-2 ms (lectura)|1-2 ms (escritura)<br>1-2 ms (lectura)|1-2 ms (escritura)<br>1-2 ms (lectura)|1-2 ms (escritura)<br>1-2 ms (lectura)|1-2 ms (escritura)<br>1-2 ms (lectura)|1-2 ms (escritura)<br>1-2 ms (lectura)|
|IOPS de destino (64 KB)|8000|16000|24000|32000|40000|48000|56000|
|Registro de los límites de frecuencia (MBps)|12|24|36|48|60|72|84|
|Cantidad máxima de trabajos (solicitudes) simultáneos|200|400|600|800|1000|1200|1400|
|Máximo de inicios de sesión simultáneos|200|400|600|800|1000|1200|1400|
|Sesiones máximas permitidas|30000|30000|30000|30000|30000|30000|30000|
|Número de réplicas|4|4|4|4|4|4|4|
|AZ múltiple|Sí|Sí|Sí|Sí|Sí|Sí|Sí|
|Escalado horizontal de lectura|Sí|Sí|Sí|Sí|Sí|Sí|Sí|
|Almacenamiento de copia de seguridad incluido|1X el tamaño de base de datos|1X el tamaño de base de datos|1X el tamaño de base de datos|1X el tamaño de base de datos|1X el tamaño de base de datos|1X el tamaño de base de datos|1X el tamaño de base de datos|

### <a name="business-critical-service-tier-generation-5-compute-platform-part-2"></a>Nivel de servicio Crítico para la empresa: Plataforma de procesos de generación 5 (parte 2)

|Tamaño de proceso|BC_Gen5_16|BC_Gen5_18|BC_Gen5_20|BC_Gen5_24|BC_Gen5_32|BC_Gen5_40|BC_Gen5_80|
|:--- | --: |--: |--: |--: |---: | --: |--: |
|Generación de H/W|5|5|5|5|5|5|5|
|Núcleos virtuales|16|18|20|24|32|40|80|
|Memoria (GB)|81.6|91.8|102|122.4|163.2|204|408|
|Compatible con almacén de columnas|Sí|Sí|Sí|Sí|Sí|Sí|Sí|
|Almacenamiento OLTP en memoria (GB)|15 768|18.139|20.51|25 252|37 936|52.22|131,64|
|Tamaño máximo de datos (GB)|3072|3072|3072|4096|4096|4096|4096|
|Tamaño máximo de registro (GB)|922|922|922|1229|1229|1229|1229|
|Tamaño de TempDB (GB)|384|384|384|384|384|384|384|
|Tipo de almacenamiento|SSD local|SSD local|SSD local|SSD local|SSD local|SSD local|SSD local|
|Latencia de E/S (aproximada)|1-2 ms (escritura)<br>1-2 ms (lectura)|1-2 ms (escritura)<br>1-2 ms (lectura)|1-2 ms (escritura)<br>1-2 ms (lectura)|1-2 ms (escritura)<br>1-2 ms (lectura)|1-2 ms (escritura)<br>1-2 ms (lectura)|1-2 ms (escritura)<br>1-2 ms (lectura)|1-2 ms (escritura)<br>1-2 ms (lectura)|
|IOPS de destino (64 KB)|64000|72000|80000|96000|128000|160000|320000|
|Registro de los límites de frecuencia (MBps)|96|96|96|96|96|96|96|
|Cantidad máxima de trabajos (solicitudes) simultáneos|1600|1.800|2000|2400|3200|4000|8000|
|Máximo de inicios de sesión simultáneos|1600|1.800|2000|2400|3200|4000|8000|
|Sesiones máximas permitidas|30000|30000|30000|30000|30000|30000|30000|
|Número de réplicas|4|4|4|4|4|4|4|
|AZ múltiple|Sí|Sí|Sí|Sí|Sí|Sí|Sí|
|Escalado horizontal de lectura|Sí|Sí|Sí|Sí|Sí|Sí|Sí|
|Almacenamiento de copia de seguridad incluido|1X el tamaño de base de datos|1X el tamaño de base de datos|1X el tamaño de base de datos|1X el tamaño de base de datos|1X el tamaño de base de datos|1X el tamaño de base de datos|1X el tamaño de base de datos|

## <a name="hyperscale-service-tier"></a>Nivel de servicio Hiperescala

### <a name="generation-5-compute-platform"></a>Plataforma de procesos de generación 5

|Nivel de rendimiento|HS_Gen5_2|HS_Gen5_4|HS_Gen5_8|HS_Gen5_16|HS_Gen5_24|HS_Gen5_32|HS_Gen5_40|HS_Gen5_80|
|:--- | --: |--: |--: |--: |---: | --: |--: |--: |
|Generación de H/W|5|5|5|5|5|5|5|5|
|Núcleos virtuales|2|4|8|16|24|32|40|80|
|Memoria (GB)|10.2|20.4|40.8|81.6|122.4|163.2|204|408|
|Compatible con almacén de columnas|Sí|Sí|Sí|Sí|Sí|Sí|Sí|Sí|
|Almacenamiento OLTP en memoria (GB)|N/D|N/D|N/D|N/D|N/D|N/D|N/D|N/D|
|Tamaño máximo de datos (TB)|100 |100 |100 |100 |100 |100 |100 |100 |
|Tamaño máximo de registro (TB)|1 |1 |1 |1 |1 |1 |1 |1 |
|Tamaño de TempDB (GB)|64|128|256|384|384|384|384|384|
|Tipo de almacenamiento|SSD local|SSD local|SSD local|SSD local|SSD local|SSD local|SSD local|SSD local|
|IOPS de destino (64 KB)|Por determinar|Por determinar|Por determinar|Por determinar|Por determinar|Por determinar|Por determinar|Por determinar|
|Latencia de E/S (aproximada)|Por determinar|Por determinar|Por determinar|Por determinar|Por determinar|Por determinar|Por determinar|Por determinar|
|Cantidad máxima de trabajos (solicitudes) simultáneos|200|400|800|1600|2400|3200|4000|8000|
|Sesiones máximas permitidas|30000|30000|30000|30000|30000|30000|30000|30000|
|Número de réplicas|2|2|2|2|2|2|2|2|
|AZ múltiple|N/D|N/D|N/D|N/D|N/D|N/D|N/D|N/D|
|Escalado horizontal de lectura|Sí|Sí|Sí|Sí|Sí|Sí|Sí|Sí|
|Almacenamiento de copia de seguridad incluido |7|7|7|7|7|7|7|7|
|||

## <a name="next-steps"></a>Pasos siguientes

- Para conocer los límites de recursos de DTU para una base de datos única, consulte los [límites de recursos para bases de datos únicas con el modelo de compra basado en DTU](sql-database-dtu-resource-limits-single-databases.md).
- Para conocer los límites de recursos de núcleos virtuales para grupos elásticos, consulte los [límites de recursos para grupos elásticos con el modelo de compra basado en núcleo virtual](sql-database-vcore-resource-limits-elastic-pools.md).
- Para conocer los límites de recursos de DTU para grupos elásticos, consulte los [límites de recursos para grupos elásticos con el modelo de compra basado en DTU](sql-database-dtu-resource-limits-elastic-pools.md).
- Para conocer los límites de recursos para instancias administradas, consulte los [límites de recursos para instancias administradas](sql-database-managed-instance-resource-limits.md).
- Para más información sobre los límites generales de Azure, consulte [Límites, cuotas y restricciones de suscripción y servicios de Microsoft Azure](../azure-subscription-service-limits.md).
- Para más información sobre los límites de recursos en un servidor de bases de datos, consulte [Límites de recursos en un servidor de SQL Database](sql-database-resource-limits-database-server.md) para obtener información acerca de los límites en los niveles de servidor y suscripción.