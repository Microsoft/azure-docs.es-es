---
title: Límites de recursos de núcleo virtual de bases de datos únicas
description: En esta página se describen algunos límites de recursos en núcleos virtuales comunes para una base de datos única en Azure SQL Database.
services: sql-database
ms.service: sql-database
ms.subservice: single-database
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: conceptual
author: stevestein
ms.author: sstein
ms.reviewer: carlrab
ms.date: 07/21/2020
ms.openlocfilehash: 64a21c0d0edcd035bdf42c3b17c5f2c0131dabfa
ms.sourcegitcommit: 0820c743038459a218c40ecfb6f60d12cbf538b3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87117022"
---
# <a name="resource-limits-for-single-databases-using-the-vcore-purchasing-model"></a>Límites de recursos para bases de datos únicas que utilizan el modelo de compra en núcleos virtuales
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

En este artículo se proporcionan los límites de recursos detallados para bases de datos únicas en Azure SQL Database que utilizan el modelo de compra en núcleos virtuales.

Para conocer los límites del modelo de compra en DTU para las bases de datos únicas en un servidor, consulte [Información general de los límites de recursos para un servidor](resource-limits-logical-server.md).

Puede establecer el nivel de servicio, el tamaño de proceso (objetivo de servicio) y la cantidad de almacenamiento para una base de datos única mediante [Azure Portal](single-database-manage.md#the-azure-portal), [Transact-SQL](single-database-manage.md#transact-sql-t-sql), [PowerShell](single-database-manage.md#powershell), la [CLI de Azure](single-database-manage.md#the-azure-cli) o la [API REST](single-database-manage.md#rest-api).

> [!IMPORTANT]
> Para información y consideraciones sobre el escalado, consulte [Escalado de una base de datos única](single-database-scale.md).

## <a name="general-purpose---serverless-compute---gen5"></a>Uso general: proceso sin servidor: Gen5

El [nivel de proceso sin servidor](serverless-tier-overview.md) está actualmente disponible en el hardware de Gen5 solo.

### <a name="gen5-compute-generation-part-1"></a>Generación de proceso Gen5 (parte 1)

|Tamaño de proceso (objetivo de servicio)|GP_S_Gen5_1|GP_S_Gen5_2|GP_S_Gen5_4|GP_S_Gen5_6|GP_S_Gen5_8|
|:--- | --: |--: |--: |--: |--: |
|Generación de procesos|Gen5|Gen5|Gen5|Gen5|Gen5|
|Mínimo y máximo de núcleos virtuales|0,5 - 1|0,5 - 2|0,5 - 4|0,75 - 6|1,0 - 8|
|Mínimo y máximo de memoria (GB)|2,02 - 3|2,05 - 6|2,10 - 12|2,25 - 18|3,00 - 24|
|Retraso mínimo y máximo de la pausa automática (minutos)|60 a 10 080|60 a 10 080|60 a 10 080|60 a 10 080|60 a 10 080|
|Compatible con almacén de columnas|Sí|Sí|Sí|Sí|Sí|
|Almacenamiento OLTP en memoria (GB)|N/D|N/D|N/D|N/D|N/D|
|Tamaño máximo de datos (GB)|512|1024|1024|1024|1536|
|Tamaño máximo de registro (GB)|154|307|307|307|461|
|Tamaño máximo de datos de TempDB (GB)|32|64|128|192|256|
|Tipo de almacenamiento|SSD remoto|SSD remoto|SSD remoto|SSD remoto|SSD remoto|
|Latencia de E/S (aproximada)|5-7 ms (escritura)<br>5-10 ms (lectura)|5-7 ms (escritura)<br>5-10 ms (lectura)|5-7 ms (escritura)<br>5-10 ms (lectura)|5-7 ms (escritura)<br>5-10 ms (lectura)|5-7 ms (escritura)<br>5-10 ms (lectura)|
|Número máx. de IOPS de datos *|320|640|1280|1920|2560|
|Velocidad de registro máx. (Mbps)|3.8|7.5|15|22.5|30|
|Cantidad máxima de trabajos (solicitudes) simultáneos|75|150|300|450|600|
|N.º máximo de sesiones simultáneas|30,000|30,000|30,000|30,000|30,000|
|Número de réplicas|1|1|1|1|1|
|AZ múltiple|N/D|N/D|N/D|N/D|N/D|
|Escalado horizontal de lectura|N/D|N/D|N/D|N/D|N/D|
|Almacenamiento de copia de seguridad incluido|1X el tamaño de base de datos|1X el tamaño de base de datos|1X el tamaño de base de datos|1X el tamaño de base de datos|1X el tamaño de base de datos|

\* El valor máximo de los tamaños de e/s que oscilan entre 8 KB y 64 KB. Las IOPS reales dependen de la carga de trabajo. Para obtener más información, consulte [Regulación de E/S de los datos](resource-limits-logical-server.md#resource-governance).

### <a name="gen5-compute-generation-part-2"></a>Generación de proceso Gen5 (parte 2)

|Tamaño de proceso (objetivo de servicio)|GP_S_Gen5_10|GP_S_Gen5_12|GP_S_Gen5_14|GP_S_Gen5_16|
|:--- | --: |--: |--: |--: |
|Generación de procesos|Gen5|Gen5|Gen5|Gen5|
|Mínimo y máximo de núcleos virtuales|1,25 - 10|1,50 - 12|1,75-14|2,00 - 16|
|Mínimo y máximo de memoria (GB)|3,75 - 30|4,50 - 36|5,25 - 42|6,00 - 48|
|Retraso mínimo y máximo de la pausa automática (minutos)|60 a 10 080|60 a 10 080|60 a 10 080|60 a 10 080|
|Compatible con almacén de columnas|Sí|Sí|Sí|Sí|
|Almacenamiento OLTP en memoria (GB)|N/D|N/D|N/D|N/D|
|Tamaño máximo de datos (GB)|1536|3072|3072|3072|
|Tamaño máximo de registro (GB)|461|461|461|922|
|Tamaño máximo de datos de TempDB (GB)|320|384|448|512|
|Tipo de almacenamiento|SSD remoto|SSD remoto|SSD remoto|SSD remoto|
|Latencia de E/S (aproximada)|5-7 ms (escritura)<br>5-10 ms (lectura)|5-7 ms (escritura)<br>5-10 ms (lectura)|5-7 ms (escritura)<br>5-10 ms (lectura)|5-7 ms (escritura)<br>5-10 ms (lectura)|
|Número máx. de IOPS de datos *|3200|3840|4480|5120|
|Velocidad de registro máx. (Mbps)|30|30|30|30|
|Cantidad máxima de trabajos (solicitudes) simultáneos|750|900|1050|1200|
|N.º máximo de sesiones simultáneas|30,000|30,000|30,000|30,000|
|Número de réplicas|1|1|1|1|
|AZ múltiple|N/D|N/D|N/D|N/D|
|Escalado horizontal de lectura|N/D|N/D|N/D|N/D|
|Almacenamiento de copia de seguridad incluido|1X el tamaño de base de datos|1X el tamaño de base de datos|1X el tamaño de base de datos|1X el tamaño de base de datos|

\* El valor máximo de los tamaños de e/s que oscilan entre 8 KB y 64 KB. Las IOPS reales dependen de la carga de trabajo. Para más información, consulte [Regulación de E/S de los datos](resource-limits-logical-server.md#resource-governance).

### <a name="gen5-compute-generation-part-3"></a>Generación de proceso Gen5 (parte 3)

|Tamaño de proceso (objetivo de servicio)|GP_S_Gen5_18|GP_S_Gen5_20|GP_S_Gen5_24|GP_S_Gen5_32|GP_S_Gen5_40|
|:--- | --: |--: |--: |--: |--:|
|Generación de procesos|Gen5|Gen5|Gen5|Gen5|Gen5|
|Mínimo y máximo de núcleos virtuales|2,25 - 18|2,5 a 20|3-24|4 a 32|5 a 40|
|Mínimo y máximo de memoria (GB)|6,75 a 54|7,5 a 60|9 a 72|12 a 96|15 a 120|
|Retraso mínimo y máximo de la pausa automática (minutos)|60 a 10 080|60 a 10 080|60 a 10 080|60 a 10 080|60 a 10 080|
|Compatible con almacén de columnas|Sí|Sí|Sí|Sí|Sí|
|Almacenamiento OLTP en memoria (GB)|N/D|N/D|N/D|N/D|N/D|
|Tamaño máximo de datos (GB)|3072|3072|4096|4096|4096|
|Tamaño máximo de registro (GB)|922|922|1229|1229|1229|
|Tamaño máximo de datos de TempDB (GB)|576|640|768|1024|1280|
|Tipo de almacenamiento|SSD remoto|SSD remoto|SSD remoto|SSD remoto|SSD remoto|
|Latencia de E/S (aproximada)|5-7 ms (escritura)<br>5-10 ms (lectura)|5-7 ms (escritura)<br>5-10 ms (lectura)|5-7 ms (escritura)<br>5-10 ms (lectura)|5-7 ms (escritura)<br>5-10 ms (lectura)|5-7 ms (escritura)<br>5-10 ms (lectura)|
|Número máx. de IOPS de datos *|5760|6400|7680|10240|12800|
|Velocidad de registro máx. (Mbps)|30|30|30|30|30|
|Cantidad máxima de trabajos (solicitudes) simultáneos|1350|1\.500|1800|2400|3000|
|N.º máximo de sesiones simultáneas|30,000|30,000|30,000|30,000|30,000|
|Número de réplicas|1|1|1|1|1|
|AZ múltiple|N/D|N/D|N/D|N/D|N/D|
|Escalado horizontal de lectura|N/D|N/D|N/D|N/D|N/D|
|Almacenamiento de copia de seguridad incluido|1X el tamaño de base de datos|1X el tamaño de base de datos|1X el tamaño de base de datos|1X el tamaño de base de datos|1X el tamaño de base de datos|

\* El valor máximo de los tamaños de e/s que oscilan entre 8 KB y 64 KB. Las IOPS reales dependen de la carga de trabajo. Para obtener más información, consulte [Regulación de E/S de los datos](resource-limits-logical-server.md#resource-governance).


## <a name="hyperscale---provisioned-compute---gen4"></a>Hiperescala: proceso aprovisionado: Gen4

### <a name="gen4-compute-generation-part-1"></a>Generación de proceso Gen4 (parte 1)

|Tamaño de proceso (objetivo de servicio)|HS_Gen4_1|HS_Gen4_2|HS_Gen4_3|HS_Gen4_4|HS_Gen4_5|HS_Gen4_6|
|:--- | --: |--: |--: |---: | --: |--: |
|Generación de procesos|Gen4|Gen4|Gen4|Gen4|Gen4|Gen4|
|Núcleos virtuales|1|2|3|4|5|6|
|Memoria (GB)|7|14|21|28|35|42|
|Tamaño de [RBPEX](service-tier-hyperscale.md#compute)|Memoria x3|Memoria x3|Memoria x3|Memoria x3|Memoria x3|Memoria x3|
|Compatible con almacén de columnas|Sí|Sí|Sí|Sí|Sí|Sí|
|Almacenamiento OLTP en memoria (GB)|N/D|N/D|N/D|N/D|N/D|N/D|
|Tamaño máximo de datos (TB)|100 |100 |100 |100 |100 |100|
|Tamaño máximo de registro (TB)|Sin límite |Sin límite |Sin límite |Sin límite |Sin límite |Sin límite |
|Tamaño máximo de datos de TempDB (GB)|32|64|96|128|160|192|
|Tipo de almacenamiento| [Nota 1](#notes) |[Nota 1](#notes)|[Nota 1](#notes) |[Nota 1](#notes) |[Nota 1](#notes) |[Nota 1](#notes) |
|IOPS de SSD local máx.*|4000 |8000 |12000 |16000 |20 000 |24000 |
|Velocidad de registro máx. (Mbps)|100 |100 |100 |100 |100 |100 |
|Latencia de E/S (aproximada)|[Nota 2](#notes)|[Nota 2](#notes)|[Nota 2](#notes)|[Nota 2](#notes)|[Nota 2](#notes)|[Nota 2](#notes)|
|Cantidad máxima de trabajos (solicitudes) simultáneos|200|400|600|800|1000|1200|
|N.º máximo de sesiones simultáneas|30,000|30,000|30,000|30,000|30,000|30,000|
|Réplicas secundarias|0-4|0-4|0-4|0-4|0-4|0-4|
|AZ múltiple|N/D|N/D|N/D|N/D|N/D|N/D|
|Escalado horizontal de lectura|Sí|Sí|Sí|Sí|Sí|Sí|
|Retención de almacenamiento de copia de seguridad|7 días|7 días|7 días|7 días|7 días|7 días|
|||

\* Además de las E/S de la unidad de estado sólido local, las cargas de trabajo usarán las E/S del [servidor de páginas](service-tier-hyperscale.md#page-server) remoto. Los IOPS efectivos dependerán de la carga de trabajo. Para obtener información, vea [Gobierno de E/S de datos](resource-limits-logical-server.md#resource-governance) y [E/S de datos en estadísticas de uso de recursos](hyperscale-performance-diagnostics.md#data-io-in-resource-utilization-statistics).

### <a name="gen4-compute-generation-part-2"></a>Generación de proceso Gen4 (parte 2)

|Tamaño de proceso (objetivo de servicio)|HS_Gen4_7|HS_Gen4_8|HS_Gen4_9|HS_Gen4_10|HS_Gen4_16|HS_Gen4_24|
|:--- | ---: |--: |--: | --: |--: |--: |
|Generación de procesos|Gen4|Gen4|Gen4|Gen4|Gen4|Gen4|
|Núcleos virtuales|7|8|9|10|16|24|
|Memoria (GB)|49|56|63|70|112|159,5|
|Tamaño de [RBPEX](service-tier-hyperscale.md#compute)|Memoria x3|Memoria x3|Memoria x3|Memoria x3|Memoria x3|Memoria x3|
|Compatible con almacén de columnas|Sí|Sí|Sí|Sí|Sí|Sí|
|Almacenamiento OLTP en memoria (GB)|N/D|N/D|N/D|N/D|N/D|N/D|
|Tamaño máximo de datos (TB)|100 |100 |100 |100 |100 |100 |
|Tamaño máximo de registro (TB)|Sin límite |Sin límite |Sin límite |Sin límite |Sin límite |Sin límite |
|Tamaño máximo de datos de TempDB (GB)|224|256|288|320|512|768|
|Tipo de almacenamiento| [Nota 1](#notes) |[Nota 1](#notes) |[Nota 1](#notes) |[Nota 1](#notes) |[Nota 1](#notes) |[Nota 1](#notes) |
|IOPS de SSD local máx.*|28000 |32000 |36000 |40000 |64000 |76 800 |
|Velocidad de registro máx. (Mbps)|100 |100 |100 |100 |100 |100 |
|Latencia de E/S (aproximada)|[Nota 2](#notes)|[Nota 2](#notes)|[Nota 2](#notes)|[Nota 2](#notes)|[Nota 2](#notes)|[Nota 2](#notes)|
|Cantidad máxima de trabajos (solicitudes) simultáneos|1400|1600|1800|2000|3200|4800|
|N.º máximo de sesiones simultáneas|30,000|30,000|30,000|30,000|30,000|30,000|
|Réplicas secundarias|0-4|0-4|0-4|0-4|0-4|0-4|
|AZ múltiple|N/D|N/D|N/D|N/D|N/D|N/D|
|Escalado horizontal de lectura|Sí|Sí|Sí|Sí|Sí|Sí|
|Retención de almacenamiento de copia de seguridad|7 días|7 días|7 días|7 días|7 días|7 días|
|||

\* Además de las E/S de la unidad de estado sólido local, las cargas de trabajo usarán las E/S del [servidor de páginas](service-tier-hyperscale.md#page-server) remoto. Los IOPS efectivos dependerán de la carga de trabajo. Para obtener información, vea [Gobierno de E/S de datos](resource-limits-logical-server.md#resource-governance) y [E/S de datos en estadísticas de uso de recursos](hyperscale-performance-diagnostics.md#data-io-in-resource-utilization-statistics).

## <a name="hyperscale---provisioned-compute---gen5"></a>Hiperescala: proceso aprovisionado: Gen5

### <a name="gen5-compute-generation-part-1"></a>Generación de proceso Gen5 (parte 1)

|Tamaño de proceso (objetivo de servicio)|HS_Gen5_2|HS_Gen5_4|HS_Gen5_6|HS_Gen_8|HS_Gen5_10|HS_Gen5_12|HS_Gen5_14|
|:--- | --: |--: |--: |--: |---: | --: |--: |--: |
|Generación de procesos|Gen5|Gen5|Gen5|Gen5|Gen5|Gen5|Gen5|
|Núcleos virtuales|2|4|6|8|10|12|14|
|Memoria (GB)|10,4|20,8|31,1|41,5|51,9|62,3|72,7|
|Tamaño de [RBPEX](service-tier-hyperscale.md#compute)|Memoria x3|Memoria x3|Memoria x3|Memoria x3|Memoria x3|Memoria x3|Memoria x3|
|Compatible con almacén de columnas|Sí|Sí|Sí|Sí|Sí|Sí|Sí|
|Almacenamiento OLTP en memoria (GB)|N/D|N/D|N/D|N/D|N/D|N/D|N/D|
|Tamaño máximo de datos (TB)|100 |100 |100 |100 |100 |100 |100|
|Tamaño máximo de registro (TB)|Sin límite |Sin límite |Sin límite |Sin límite |Sin límite |Sin límite |Sin límite |
|Tamaño máximo de datos de TempDB (GB)|64|128|192|256|320|384|448|
|Tipo de almacenamiento| [Nota 1](#notes) |[Nota 1](#notes)|[Nota 1](#notes) |[Nota 1](#notes) |[Nota 1](#notes) |[Nota 1](#notes) |[Nota 1](#notes) |
|IOPS de SSD local máx. *|8000 |16000 |24000 |32000 |40000 |48000 |56 000 |
|Velocidad de registro máx. (Mbps)|100 |100 |100 |100 |100 |100 |100 |
|Latencia de E/S (aproximada)|[Nota 2](#notes)|[Nota 2](#notes)|[Nota 2](#notes)|[Nota 2](#notes)|[Nota 2](#notes)|[Nota 2](#notes)|[Nota 2](#notes)|
|Cantidad máxima de trabajos (solicitudes) simultáneos|200|400|600|800|1000|1200|1400|
|N.º máximo de sesiones simultáneas|30,000|30,000|30,000|30,000|30,000|30,000|30,000|
|Réplicas secundarias|0-4|0-4|0-4|0-4|0-4|0-4|0-4|
|AZ múltiple|N/D|N/D|N/D|N/D|N/D|N/D|N/D|
|Escalado horizontal de lectura|Sí|Sí|Sí|Sí|Sí|Sí|Sí|
|Retención de almacenamiento de copia de seguridad|7 días|7 días|7 días|7 días|7 días|7 días|7 días|
|||

\* Además de las E/S de la unidad de estado sólido local, las cargas de trabajo usarán las E/S del [servidor de páginas](service-tier-hyperscale.md#page-server) remoto. Los IOPS efectivos dependerán de la carga de trabajo. Para obtener información, vea [Gobierno de E/S de datos](resource-limits-logical-server.md#resource-governance) y [E/S de datos en estadísticas de uso de recursos](hyperscale-performance-diagnostics.md#data-io-in-resource-utilization-statistics).

### <a name="gen5-compute-generation-part-2"></a>Generación de proceso Gen5 (parte 2)

|Tamaño de proceso (objetivo de servicio)|HS_Gen5_16|HS_Gen5_18|HS_Gen5_20|HS_Gen_24|HS_Gen5_32|HS_Gen5_40|HS_Gen5_80|
|:--- | --: |--: |--: |--: |---: |--: |--: |
|Generación de procesos|Gen5|Gen5|Gen5|Gen5|Gen5|Gen5|Gen5|
|Núcleos virtuales|16|18|20|24|32|40|80|
|Memoria (GB)|83|93,4|103,8|124,6|166,1|207,6|415,2|
|Tamaño de [RBPEX](service-tier-hyperscale.md#compute)|Memoria x3|Memoria x3|Memoria x3|Memoria x3|Memoria x3|Memoria x3|Memoria x3|
|Compatible con almacén de columnas|Sí|Sí|Sí|Sí|Sí|Sí|Sí|
|Almacenamiento OLTP en memoria (GB)|N/D|N/D|N/D|N/D|N/D|N/D|N/D|
|Tamaño máximo de datos (TB)|100 |100 |100 |100 |100 |100 |100 |
|Tamaño máximo de registro (TB)|Sin límite |Sin límite |Sin límite |Sin límite |Sin límite |Sin límite |Sin límite |
|Tamaño máximo de datos de TempDB (GB)|512|576|640|768|1024|1280|2560|
|Tipo de almacenamiento| [Nota 1](#notes) |[Nota 1](#notes)|[Nota 1](#notes)|[Nota 1](#notes) |[Nota 1](#notes) |[Nota 1](#notes) |[Nota 1](#notes) |
|IOPS de SSD local máx. *|64000 |72 000 |80000 |96 000 |160 000 |192 000 |204800 |
|Velocidad de registro máx. (Mbps)|100 |100 |100 |100 |100 |100 |100 |
|Latencia de E/S (aproximada)|[Nota 2](#notes)|[Nota 2](#notes)|[Nota 2](#notes)|[Nota 2](#notes)|[Nota 2](#notes)|[Nota 2](#notes)|[Nota 2](#notes)|
|Cantidad máxima de trabajos (solicitudes) simultáneos|1600|1800|2000|2400|3200|4000|8000|
|N.º máximo de sesiones simultáneas|30,000|30,000|30,000|30,000|30,000|30,000|30,000|
|Réplicas secundarias|0-4|0-4|0-4|0-4|0-4|0-4|0-4|
|AZ múltiple|N/D|N/D|N/D|N/D|N/D|N/D|N/D|
|Escalado horizontal de lectura|Sí|Sí|Sí|Sí|Sí|Sí|Sí|
|Retención de almacenamiento de copia de seguridad|7 días|7 días|7 días|7 días|7 días|7 días|7 días|
|||

\* Además de las E/S de la unidad de estado sólido local, las cargas de trabajo usarán las E/S del [servidor de páginas](service-tier-hyperscale.md#page-server) remoto. Los IOPS efectivos dependerán de la carga de trabajo. Para obtener información, vea [Gobierno de E/S de datos](resource-limits-logical-server.md#resource-governance) y [E/S de datos en estadísticas de uso de recursos](hyperscale-performance-diagnostics.md#data-io-in-resource-utilization-statistics).

#### <a name="notes"></a>Notas

**Nota 1**: La hiperescala es una arquitectura de varios niveles con componentes de proceso y almacenamiento independientes: [Arquitectura de nivel de servicio Hiperescala](service-tier-hyperscale.md#distributed-functions-architecture)

**Nota 2**: La latencia es de 1 o 2 ms para los datos de la unidad de estado sólido de la réplica de proceso local, que almacena en caché las páginas de datos más usadas. Mayor latencia de los datos recuperados de los servidores de páginas.

## <a name="general-purpose---provisioned-compute---gen4"></a>Uso general: proceso aprovisionado: Gen4

> [!IMPORTANT]
> Las nuevas bases de datos de Gen4 ya no se admiten en las regiones Este de Australia o Sur de Brasil.

### <a name="gen4-compute-generation-part-1"></a>Generación de proceso Gen4 (parte 1)

|Tamaño de proceso (objetivo de servicio)|GP_Gen4_1|GP_Gen4_2|GP_Gen4_3|GP_Gen4_4|GP_Gen4_5|GP_Gen4_6
|:--- | --: |--: |--: |--: |--: |--: |
|Generación de procesos|Gen4|Gen4|Gen4|Gen4|Gen4|Gen4|
|Núcleos virtuales|1|2|3|4|5|6|
|Memoria (GB)|7|14|21|28|35|42|
|Compatible con almacén de columnas|Sí|Sí|Sí|Sí|Sí|Sí|
|Almacenamiento OLTP en memoria (GB)|N/D|N/D|N/D|N/D|N/D|N/D|
|Tamaño máximo de datos (GB)|1024|1024|1536|1536|1536|3072|
|Tamaño máximo de registro (GB)|307|307|461|461|461|922|
|Tamaño máximo de datos de TempDB (GB)|32|64|96|128|160|192|
|Tipo de almacenamiento|SSD remoto|SSD remoto|SSD remoto|SSD remoto|SSD remoto|SSD remoto|
|Latencia de E/S (aproximada)|5-7 ms (escritura)<br>5-10 ms (lectura)|5-7 ms (escritura)<br>5-10 ms (lectura)|5-7 ms (escritura)<br>5-10 ms (lectura)|5-7 ms (escritura)<br>5-10 ms (lectura)|5-7 ms (escritura)<br>5-10 ms (lectura)|5-7 ms (escritura)<br>5-10 ms (lectura)|
|Número máx. de IOPS de datos *|320|640|960|1280|1600|1920|
|Velocidad de registro máx. (Mbps)|3,75|7.5|11,25|15|18,75|22.5|
|Cantidad máxima de trabajos (solicitudes) simultáneos|200|400|600|800|1000|1200|
|N.º máximo de sesiones simultáneas|30,000|30,000|30,000|30,000|30,000|30,000|
|Número de réplicas|1|1|1|1|1|1|
|AZ múltiple|N/D|N/D|N/D|N/D|N/D|N/D|
|Escalado horizontal de lectura|N/D|N/D|N/D|N/D|N/D|N/D|
|Almacenamiento de copia de seguridad incluido|1X el tamaño de base de datos|1X el tamaño de base de datos|1X el tamaño de base de datos|1X el tamaño de base de datos|1X el tamaño de base de datos|1X el tamaño de base de datos|

\* El valor máximo de los tamaños de e/s que oscilan entre 8 KB y 64 KB. Las IOPS reales dependen de la carga de trabajo. Para obtener más información, consulte [Regulación de E/S de los datos](resource-limits-logical-server.md#resource-governance).

### <a name="gen4-compute-generation-part-2"></a>Generación de proceso Gen4 (parte 2)

|Tamaño de proceso (objetivo de servicio)|GP_Gen4_7|GP_Gen4_8|GP_Gen4_9|GP_Gen4_10|GP_Gen4_16|GP_Gen4_24
|:--- | --: |--: |--: |--: |--: |--: |
|Generación de procesos|Gen4|Gen4|Gen4|Gen4|Gen4|Gen4|
|Núcleos virtuales|7|8|9|10|16|24|
|Memoria (GB)|49|56|63|70|112|159,5|
|Compatible con almacén de columnas|Sí|Sí|Sí|Sí|Sí|Sí|
|Almacenamiento OLTP en memoria (GB)|N/D|N/D|N/D|N/D|N/D|N/D|
|Tamaño máximo de datos (GB)|3072|3072|3072|3072|4096|4096|
|Tamaño máximo de registro (GB)|922|922|922|922|1229|1229|
|Tamaño máximo de datos de TempDB (GB)|224|256|288|320|512|768|
|Tipo de almacenamiento|SSD remoto|SSD remoto|SSD remoto|SSD remoto|SSD remoto|SSD remoto|
|Latencia de E/S (aproximada)|5-7 ms (escritura)<br>5-10 ms (lectura)|5-7 ms (escritura)<br>5-10 ms (lectura)|5-7 ms (escritura)<br>5-10 ms (lectura)|5-7 ms (escritura)<br>5-10 ms (lectura)|5-7 ms (escritura)<br>5-10 ms (lectura)|5-7 ms (escritura)<br>5-10 ms (lectura)
|Número máx. de IOPS de datos *|2240|2560|2880|3200|5120|7680|
|Velocidad de registro máx. (Mbps)|26,3|30|30|30|30|30|
|Cantidad máxima de trabajos (solicitudes) simultáneos|1400|1600|1800|2000|3200|4800|
|N.º máximo de sesiones simultáneas|30,000|30,000|30,000|30,000|30,000|30,000|
|Número de réplicas|1|1|1|1|1|1|
|AZ múltiple|N/D|N/D|N/D|N/D|N/D|N/D|
|Escalado horizontal de lectura|N/D|N/D|N/D|N/D|N/D|N/D|
|Almacenamiento de copia de seguridad incluido|1X el tamaño de base de datos|1X el tamaño de base de datos|1X el tamaño de base de datos|1X el tamaño de base de datos|1X el tamaño de base de datos|1X el tamaño de base de datos|

\* El valor máximo de los tamaños de e/s que oscilan entre 8 KB y 64 KB. Las IOPS reales dependen de la carga de trabajo. Para obtener más información, consulte [Regulación de E/S de los datos](resource-limits-logical-server.md#resource-governance).

## <a name="general-purpose---provisioned-compute---gen5"></a>Uso general: proceso aprovisionado: Gen5

### <a name="gen5-compute-generation-part-1"></a>Generación de proceso Gen5 (parte 1)

|Tamaño de proceso (objetivo de servicio)|GP_Gen5_2|GP_Gen5_4|GP_Gen5_6|GP_Gen5_8|GP_Gen5_10|GP_Gen5_12|GP_Gen5_14|
|:--- | --: |--: |--: |--: |---: | --: |--: |
|Generación de procesos|Gen5|Gen5|Gen5|Gen5|Gen5|Gen5|Gen5|
|Núcleos virtuales|2|4|6|8|10|12|14|
|Memoria (GB)|10,4|20,8|31,1|41,5|51,9|62,3|72,7|
|Compatible con almacén de columnas|Sí|Sí|Sí|Sí|Sí|Sí|Sí|
|Almacenamiento OLTP en memoria (GB)|N/D|N/D|N/D|N/D|N/D|N/D|N/D|
|Tamaño máximo de datos (GB)|1024|1024|1536|1536|1536|3072|3072|
|Tamaño máximo de registro (GB)|307|307|461|461|461|922|922|
|Tamaño máximo de datos de TempDB (GB)|64|128|192|256|320|384|384|
|Tipo de almacenamiento|SSD remoto|SSD remoto|SSD remoto|SSD remoto|SSD remoto|SSD remoto|SSD remoto|
|Latencia de E/S (aproximada)|5-7 ms (escritura)<br>5-10 ms (lectura)|5-7 ms (escritura)<br>5-10 ms (lectura)|5-7 ms (escritura)<br>5-10 ms (lectura)|5-7 ms (escritura)<br>5-10 ms (lectura)|5-7 ms (escritura)<br>5-10 ms (lectura)|5-7 ms (escritura)<br>5-10 ms (lectura)|5-7 ms (escritura)<br>5-10 ms (lectura)|
|Número máx. de IOPS de datos *|640|1280|1920|2560|3200|3840|4480|
|Velocidad de registro máx. (Mbps)|7.5|15|22.5|30|30|30|30|
|Cantidad máxima de trabajos (solicitudes) simultáneos|200|400|600|800|1000|1200|1400|
|N.º máximo de sesiones simultáneas|30,000|30,000|30,000|30,000|30,000|30,000|30,000|
|Número de réplicas|1|1|1|1|1|1|1|
|AZ múltiple|N/D|N/D|N/D|N/D|N/D|N/D|N/D|
|Escalado horizontal de lectura|N/D|N/D|N/D|N/D|N/D|N/D|N/D|
|Almacenamiento de copia de seguridad incluido|1X el tamaño de base de datos|1X el tamaño de base de datos|1X el tamaño de base de datos|1X el tamaño de base de datos|1X el tamaño de base de datos|1X el tamaño de base de datos|1X el tamaño de base de datos|

\* El valor máximo de los tamaños de e/s que oscilan entre 8 KB y 64 KB. Las IOPS reales dependen de la carga de trabajo. Para obtener más información, consulte [Regulación de E/S de los datos](resource-limits-logical-server.md#resource-governance).

### <a name="gen5-compute-generation-part-2"></a>Generación de proceso Gen5 (parte 2)

|Tamaño de proceso (objetivo de servicio)|GP_Gen5_16|GP_Gen5_18|GP_Gen5_20|GP_Gen5_24|GP_Gen5_32|GP_Gen5_40|GP_Gen5_80|
|:--- | --: |--: |--: |--: |---: | --: |--: |
|Generación de procesos|Gen5|Gen5|Gen5|Gen5|Gen5|Gen5|Gen5|
|Núcleos virtuales|16|18|20|24|32|40|80|
|Memoria (GB)|83|93,4|103,8|124,6|166,1|207,6|415,2|
|Compatible con almacén de columnas|Sí|Sí|Sí|Sí|Sí|Sí|Sí|
|Almacenamiento OLTP en memoria (GB)|N/D|N/D|N/D|N/D|N/D|N/D|N/D|
|Tamaño máximo de datos (GB)|3072|3072|3072|4096|4096|4096|4096|
|Tamaño máximo de registro (GB)|922|922|922|1229|1229|1229|1229|
|Tamaño máximo de datos de TempDB (GB)|512|576|640|768|1024|1280|2560|
|Tipo de almacenamiento|SSD remoto|SSD remoto|SSD remoto|SSD remoto|SSD remoto|SSD remoto|SSD remoto|
|Latencia de E/S (aproximada)|5-7 ms (escritura)<br>5-10 ms (lectura)|5-7 ms (escritura)<br>5-10 ms (lectura)|5-7 ms (escritura)<br>5-10 ms (lectura)|5-7 ms (escritura)<br>5-10 ms (lectura)|5-7 ms (escritura)<br>5-10 ms (lectura)|5-7 ms (escritura)<br>5-10 ms (lectura)|5-7 ms (escritura)<br>5-10 ms (lectura)|
|Número máx. de IOPS de datos *|5120|5760|6400|7680|10240|12800|12800|
|Velocidad de registro máx. (Mbps)|30|30|30|30|30|30|30|
|Cantidad máxima de trabajos (solicitudes) simultáneos|1600|1800|2000|2400|3200|4000|8000|
|N.º máximo de sesiones simultáneas|30,000|30,000|30,000|30,000|30,000|30,000|30,000|
|Número de réplicas|1|1|1|1|1|1|1|
|AZ múltiple|N/D|N/D|N/D|N/D|N/D|N/D|N/D|
|Escalado horizontal de lectura|N/D|N/D|N/D|N/D|N/D|N/D|N/D|
|Almacenamiento de copia de seguridad incluido|1X el tamaño de base de datos|1X el tamaño de base de datos|1X el tamaño de base de datos|1X el tamaño de base de datos|1X el tamaño de base de datos|1X el tamaño de base de datos|1X el tamaño de base de datos|

\* El valor máximo de los tamaños de e/s que oscilan entre 8 KB y 64 KB. Las IOPS reales dependen de la carga de trabajo. Para obtener más información, consulte [Regulación de E/S de los datos](resource-limits-logical-server.md#resource-governance).

## <a name="general-purpose---provisioned-compute---fsv2-series"></a>Uso general: proceso aprovisionado: serie Fsv2

### <a name="fsv2-series-compute-generation-part-1"></a>Generación de proceso de la serie Fsv2 (parte 1)

|Tamaño de proceso (objetivo de servicio)|GP_Fsv2_8|GP_Fsv2_10|GP_Fsv2_12|GP_Fsv2_14| GP_Fsv2_16|
|:---| ---:|---:|---:|---:|---:|
|Generación de procesos|Serie Fsv2|Serie Fsv2|Serie Fsv2|Serie Fsv2|Serie Fsv2|
|Núcleos virtuales|8|10|12|14|16|
|Memoria (GB)|15,1|18,9|22,7|26,5|30,2|
|Compatible con almacén de columnas|Sí|Sí|Sí|Sí|Sí|
|Almacenamiento OLTP en memoria (GB)|N/D|No aplicable|No aplicable|No aplicable|N/D|
|Tamaño máximo de datos (GB)|1024|1024|1024|1024|1536|
|Tamaño máximo de registro (GB)|336|336|336|336|512|
|Tamaño máximo de datos de TempDB (GB)|333|333|333|333|333|
|Tipo de almacenamiento|SSD remoto|SSD remoto|SSD remoto|SSD remoto|SSD remoto|
|Latencia de E/S (aproximada)|5-7 ms (escritura)<br>5-10 ms (lectura)|5-7 ms (escritura)<br>5-10 ms (lectura)|5-7 ms (escritura)<br>5-10 ms (lectura)|5-7 ms (escritura)<br>5-10 ms (lectura)|5-7 ms (escritura)<br>5-10 ms (lectura)|
|Número máx. de IOPS de datos *|2560|3200|3840|4480|5120|
|Velocidad de registro máx. (Mbps)|30|30|30|30|30|
|Cantidad máxima de trabajos (solicitudes) simultáneos|400|500|600|700|800|
|Máximo de inicios de sesión simultáneos|800|1000|1200|1400|1600|
|N.º máximo de sesiones simultáneas|30,000|30,000|30,000|30,000|30,000|
|Número de réplicas|1|1|1|1|1|
|AZ múltiple|No aplicable|No aplicable|No aplicable|No aplicable|No aplicable|
|Escalado horizontal de lectura|N/D|No aplicable|No aplicable|No aplicable|N/D|
|Almacenamiento de copia de seguridad incluido|1X el tamaño de base de datos|1X el tamaño de base de datos|1X el tamaño de base de datos|1X el tamaño de base de datos|1X el tamaño de base de datos|

\* El valor máximo de los tamaños de e/s que oscilan entre 8 KB y 64 KB. Las IOPS reales dependen de la carga de trabajo. Para obtener más información, consulte [Regulación de E/S de los datos](resource-limits-logical-server.md#resource-governance).

### <a name="fsv2-series-compute-generation-part-2"></a>Generación de proceso de la serie Fsv2 (parte 2)

|Tamaño de proceso (objetivo de servicio)|GP_Fsv2_18|GP_Fsv2_20|GP_Fsv2_24|GP_Fsv2_32| GP_Fsv2_36|GP_Fsv2_72|
|:---| ---:|---:|---:|---:|---:|---:|
|Generación de procesos|Serie Fsv2|Serie Fsv2|Serie Fsv2|Serie Fsv2|Serie Fsv2|Serie Fsv2|
|Núcleos virtuales|18|20|24|32|36|72|
|Memoria (GB)|34,0|37,8|45,4|60,5|68,0|136,0|
|Compatible con almacén de columnas|Sí|Sí|Sí|Sí|Sí|Sí|
|Almacenamiento OLTP en memoria (GB)|No aplicable|No aplicable|No aplicable|No aplicable|No aplicable|No aplicable|
|Tamaño máximo de datos (GB)|1536|1536|1536|3072|3072|4096|
|Tamaño máximo de registro (GB)|512|512|512|1024|1024|1024|
|Tamaño máximo de datos de TempDB (GB)|83,25|92,5|111|148|166,5|333|
|Tipo de almacenamiento|SSD remoto|SSD remoto|SSD remoto|SSD remoto|SSD remoto|SSD remoto|
|Latencia de E/S (aproximada)|5-7 ms (escritura)<br>5-10 ms (lectura)|5-7 ms (escritura)<br>5-10 ms (lectura)|5-7 ms (escritura)<br>5-10 ms (lectura)|5-7 ms (escritura)<br>5-10 ms (lectura)|5-7 ms (escritura)<br>5-10 ms (lectura)|5-7 ms (escritura)<br>5-10 ms (lectura)|
|Número máx. de IOPS de datos *|5760|6400|7680|10240|11 520|23 040|
|Velocidad de registro máx. (Mbps)|30|30|30|30|30|30|
|Cantidad máxima de trabajos (solicitudes) simultáneos|900|1000|1200|1600|1800|3600|
|Máximo de inicios de sesión simultáneos|1800|2000|2400|3200|3600|7200|
|N.º máximo de sesiones simultáneas|30,000|30,000|30,000|30,000|30,000|30,000|
|Número de réplicas|1|1|1|1|1|1|
|AZ múltiple|No aplicable|No aplicable|No aplicable|No aplicable|No aplicable|No aplicable|
|Escalado horizontal de lectura|N/D|No aplicable|No aplicable|No aplicable|No aplicable|N/D|
|Almacenamiento de copia de seguridad incluido|1X el tamaño de base de datos|1X el tamaño de base de datos|1X el tamaño de base de datos|1X el tamaño de base de datos|1X el tamaño de base de datos|1X el tamaño de base de datos|

\* El valor máximo de los tamaños de e/s que oscilan entre 8 KB y 64 KB. Las IOPS reales dependen de la carga de trabajo. Para obtener más información, consulte [Regulación de E/S de los datos](resource-limits-logical-server.md#resource-governance).

## <a name="business-critical---provisioned-compute---gen4"></a>Crítico para la empresa: proceso aprovisionado: Gen4

> [!IMPORTANT]
> Las nuevas bases de datos de Gen4 ya no se admiten en las regiones Este de Australia o Sur de Brasil.

### <a name="gen4-compute-generation-part-1"></a>Generación de proceso Gen4 (parte 1)

|Tamaño de proceso (objetivo de servicio)|BC_Gen4_1|BC_Gen4_2|BC_Gen4_3|BC_Gen4_4|BC_Gen4_5|BC_Gen4_6|
|:--- | --: |--: |--: |--: |--: |--: |
|Generación de procesos|Gen4|Gen4|Gen4|Gen4|Gen4|Gen4|
|Núcleos virtuales|1|2|3|4|5|6|
|Memoria (GB)|7|14|21|28|35|42|
|Compatible con almacén de columnas|Sí|Sí|Sí|Sí|Sí|Sí|
|Almacenamiento OLTP en memoria (GB)|1|2|3|4|5|6|
|Tipo de almacenamiento|SSD local|SSD local|SSD local|SSD local|SSD local|SSD local|
|Tamaño máximo de datos (GB)|1024|1024|1024|1024|1024|1024|
|Tamaño máximo de registro (GB)|307|307|307|307|307|307|
|Tamaño máximo de datos de TempDB (GB)|32|64|96|128|160|192|
|Latencia de E/S (aproximada)|1-2 ms (escritura)<br>1-2 ms (lectura)|1-2 ms (escritura)<br>1-2 ms (lectura)|1-2 ms (escritura)<br>1-2 ms (lectura)|1-2 ms (escritura)<br>1-2 ms (lectura)|1-2 ms (escritura)<br>1-2 ms (lectura)|1-2 ms (escritura)<br>1-2 ms (lectura)|
|Número máx. de IOPS de datos *|4\.000|8,000|12,000|16 000|20.000|24,000|
|Velocidad de registro máx. (Mbps)|8|16|24|32|40|48|
|Cantidad máxima de trabajos (solicitudes) simultáneos|200|400|600|800|1000|1200|
|Máximo de inicios de sesión simultáneos|200|400|600|800|1000|1200|
|N.º máximo de sesiones simultáneas|30,000|30,000|30,000|30,000|30,000|30,000|
|Número de réplicas|4|4|4|4|4|4|
|AZ múltiple|Sí|Sí|Sí|Sí|Sí|Sí|
|Escalado horizontal de lectura|Sí|Sí|Sí|Sí|Sí|Sí|
|Almacenamiento de copia de seguridad incluido|1X el tamaño de base de datos|1X el tamaño de base de datos|1X el tamaño de base de datos|1X el tamaño de base de datos|1X el tamaño de base de datos|1X el tamaño de base de datos|

\* El valor máximo de los tamaños de e/s que oscilan entre 8 KB y 64 KB. Las IOPS reales dependen de la carga de trabajo. Para obtener más información, consulte [Regulación de E/S de los datos](resource-limits-logical-server.md#resource-governance).

### <a name="gen4-compute-generation-part-2"></a>Generación de proceso Gen4 (parte 2)

|Tamaño de proceso (objetivo de servicio)|BC_Gen4_7|BC_Gen4_8|BC_Gen4_9|BC_Gen4_10|BC_Gen4_16|BC_Gen4_24|
|:--- | --: |--: |--: |--: |--: |--: |
|Generación de procesos|Gen4|Gen4|Gen4|Gen4|Gen4|Gen4|
|Núcleos virtuales|7|8|9|10|16|24|
|Memoria (GB)|49|56|63|70|112|159,5|
|Compatible con almacén de columnas|Sí|Sí|Sí|Sí|Sí|Sí|
|Almacenamiento OLTP en memoria (GB)|7|8|9.5|11|20|36|
|Tipo de almacenamiento|SSD local|SSD local|SSD local|SSD local|SSD local|SSD local|
|Tamaño máximo de datos (GB)|1024|1024|1024|1024|1024|1024|
|Tamaño máximo de registro (GB)|307|307|307|307|307|307|
|Tamaño máximo de datos de TempDB (GB)|224|256|288|320|512|768|
|Latencia de E/S (aproximada)|1-2 ms (escritura)<br>1-2 ms (lectura)|1-2 ms (escritura)<br>1-2 ms (lectura)|1-2 ms (escritura)<br>1-2 ms (lectura)|1-2 ms (escritura)<br>1-2 ms (lectura)|1-2 ms (escritura)<br>1-2 ms (lectura)|1-2 ms (escritura)<br>1-2 ms (lectura)|
|Número máx. de IOPS de datos |28 000|32 000|36 000|40.000|64 000|76 800|
|Velocidad de registro máx. (Mbps)|56|64|64|64|64|64|
|Cantidad máxima de trabajos (solicitudes) simultáneos|1400|1600|1800|2000|3200|4800|
|Máximo de inicios de sesión simultáneos (solicitudes)|1400|1600|1800|2000|3200|4800|
|N.º máximo de sesiones simultáneas|30,000|30,000|30,000|30,000|30,000|30,000|
|Número de réplicas|4|4|4|4|4|4|
|AZ múltiple|Sí|Sí|Sí|Sí|Sí|Sí|
|Escalado horizontal de lectura|Sí|Sí|Sí|Sí|Sí|Sí|
|Almacenamiento de copia de seguridad incluido|1X el tamaño de base de datos|1X el tamaño de base de datos|1X el tamaño de base de datos|1X el tamaño de base de datos|1X el tamaño de base de datos|1X el tamaño de base de datos|

\* El valor máximo de los tamaños de e/s que oscilan entre 8 KB y 64 KB. Las IOPS reales dependen de la carga de trabajo. Para obtener más información, consulte [Regulación de E/S de los datos](resource-limits-logical-server.md#resource-governance).

## <a name="business-critical---provisioned-compute---gen5"></a>Crítico para la empresa: proceso aprovisionado: Gen5

### <a name="gen5-compute-generation-part-1"></a>Generación de proceso Gen5 (parte 1)

|Tamaño de proceso (objetivo de servicio)|BC_Gen5_2|BC_Gen5_4|BC_Gen5_6|BC_Gen5_8|BC_Gen5_10|BC_Gen5_12|BC_Gen5_14|
|:--- | --: |--: |--: |--: |---: | --: |--: |
|Generación de procesos|Gen5|Gen5|Gen5|Gen5|Gen5|Gen5|Gen5|
|Núcleos virtuales|2|4|6|8|10|12|14|
|Memoria (GB)|10,4|20,8|31,1|41,5|51,9|62,3|72,7|
|Compatible con almacén de columnas|Sí|Sí|Sí|Sí|Sí|Sí|Sí|
|Almacenamiento OLTP en memoria (GB)|1,57|3,14|4.71|6,28|8,65|11,02|13,39|
|Tamaño máximo de datos (GB)|1024|1024|1536|1536|1536|3072|3072|
|Tamaño máximo de registro (GB)|307|307|461|461|461|922|922|
|Tamaño máximo de datos de TempDB (GB)|64|128|192|256|320|384|448|
|Tipo de almacenamiento|SSD local|SSD local|SSD local|SSD local|SSD local|SSD local|SSD local|
|Latencia de E/S (aproximada)|1-2 ms (escritura)<br>1-2 ms (lectura)|1-2 ms (escritura)<br>1-2 ms (lectura)|1-2 ms (escritura)<br>1-2 ms (lectura)|1-2 ms (escritura)<br>1-2 ms (lectura)|1-2 ms (escritura)<br>1-2 ms (lectura)|1-2 ms (escritura)<br>1-2 ms (lectura)|1-2 ms (escritura)<br>1-2 ms (lectura)|
|Número máx. de IOPS de datos *|8000|16 000|24,000|32 000|40.000|48 000|56 000|
|Velocidad de registro máx. (Mbps)|24|48|72|96|96|96|96|
|Cantidad máxima de trabajos (solicitudes) simultáneos|200|400|600|800|1000|1200|1400|
|Máximo de inicios de sesión simultáneos|200|400|600|800|1000|1200|1400|
|N.º máximo de sesiones simultáneas|30,000|30,000|30,000|30,000|30,000|30,000|30,000|
|Número de réplicas|4|4|4|4|4|4|4|
|AZ múltiple|Sí|Sí|Sí|Sí|Sí|Sí|Sí|
|Escalado horizontal de lectura|Sí|Sí|Sí|Sí|Sí|Sí|Sí|
|Almacenamiento de copia de seguridad incluido|1X el tamaño de base de datos|1X el tamaño de base de datos|1X el tamaño de base de datos|1X el tamaño de base de datos|1X el tamaño de base de datos|1X el tamaño de base de datos|1X el tamaño de base de datos|

\* El valor máximo de los tamaños de e/s que oscilan entre 8 KB y 64 KB. Las IOPS reales dependen de la carga de trabajo. Para obtener más información, consulte [Regulación de E/S de los datos](resource-limits-logical-server.md#resource-governance).

### <a name="gen5-compute-generation-part-2"></a>Generación de proceso Gen5 (parte 2)

|Tamaño de proceso (objetivo de servicio)|BC_Gen5_16|BC_Gen5_18|BC_Gen5_20|BC_Gen5_24|BC_Gen5_32|BC_Gen5_40|BC_Gen5_80|
|:--- | --: |--: |--: |--: |---: | --: |--: |
|Generación de procesos|Gen5|Gen5|Gen5|Gen5|Gen5|Gen5|Gen5|
|Núcleos virtuales|16|18|20|24|32|40|80|
|Memoria (GB)|83|93,4|103,8|124,6|166,1|207,6|415,2|
|Compatible con almacén de columnas|Sí|Sí|Sí|Sí|Sí|Sí|Sí|
|Almacenamiento OLTP en memoria (GB)|15,77|18,14|20.51|25,25|37,94|52,23|131,64|
|Tamaño máximo de datos (GB)|3072|3072|3072|4096|4096|4096|4096|
|Tamaño máximo de registro (GB)|922|922|922|1229|1229|1229|1229|
|Tamaño máximo de datos de TempDB (GB)|512|576|640|768|1024|1280|2560|
|Tipo de almacenamiento|SSD local|SSD local|SSD local|SSD local|SSD local|SSD local|SSD local|
|Latencia de E/S (aproximada)|1-2 ms (escritura)<br>1-2 ms (lectura)|1-2 ms (escritura)<br>1-2 ms (lectura)|1-2 ms (escritura)<br>1-2 ms (lectura)|1-2 ms (escritura)<br>1-2 ms (lectura)|1-2 ms (escritura)<br>1-2 ms (lectura)|1-2 ms (escritura)<br>1-2 ms (lectura)|1-2 ms (escritura)<br>1-2 ms (lectura)|
|Número máx. de IOPS de datos *|64 000|72 000|80 000|96 000|128 000|160 000|204 800|
|Velocidad de registro máx. (Mbps)|96|96|96|96|96|96|96|
|Cantidad máxima de trabajos (solicitudes) simultáneos|1600|1800|2000|2400|3200|4000|8000|
|Máximo de inicios de sesión simultáneos|1600|1800|2000|2400|3200|4000|8000|
|N.º máximo de sesiones simultáneas|30,000|30,000|30,000|30,000|30,000|30,000|30,000|
|Número de réplicas|4|4|4|4|4|4|4|
|AZ múltiple|Sí|Sí|Sí|Sí|Sí|Sí|Sí|
|Escalado horizontal de lectura|Sí|Sí|Sí|Sí|Sí|Sí|Sí|
|Almacenamiento de copia de seguridad incluido|1X el tamaño de base de datos|1X el tamaño de base de datos|1X el tamaño de base de datos|1X el tamaño de base de datos|1X el tamaño de base de datos|1X el tamaño de base de datos|1X el tamaño de base de datos|

\* El valor máximo de los tamaños de e/s que oscilan entre 8 KB y 64 KB. Las IOPS reales dependen de la carga de trabajo. Para obtener más información, consulte [Regulación de E/S de los datos](resource-limits-logical-server.md#resource-governance).

## <a name="business-critical---provisioned-compute---m-series"></a>Crítico para la empresa: proceso aprovisionado: serie M

### <a name="m-series-compute-generation-part-1"></a>Generación de proceso de la serie M (parte 1)

|Tamaño de proceso (objetivo de servicio)|BC_M_8|BC_M_10|BC_M_12|BC_M_14|BC_M_16|BC_M_18|
|:---| ---:|---:|---:|---:|---:|---:|
|Generación de procesos|Serie M|Serie M|Serie M|Serie M|Serie M|Serie M|
|Núcleos virtuales|8|10|12|14|16|18|
|Memoria (GB)|235,4|294,3|353,2|412,0|470,9|529,7|
|Compatible con almacén de columnas|Sí|Sí|Sí|Sí|Sí|Sí|
|Almacenamiento OLTP en memoria (GB)|64|80|96|112|128|150|
|Tamaño máximo de datos (GB)|512|640|768|896|1024|1152|
|Tamaño máximo de registro (GB)|171|213|256|299|341|384|
|Tamaño máximo de datos de TempDB (GB)|256|320|384|448|512|576|
|Tipo de almacenamiento|SSD local|SSD local|SSD local|SSD local|SSD local|SSD local|
|Latencia de E/S (aproximada)|1-2 ms (escritura)<br>1-2 ms (lectura)|1-2 ms (escritura)<br>1-2 ms (lectura)|1-2 ms (escritura)<br>1-2 ms (lectura)|1-2 ms (escritura)<br>1-2 ms (lectura)|1-2 ms (escritura)<br>1-2 ms (lectura)|1-2 ms (escritura)<br>1-2 ms (lectura)|
|Número máx. de IOPS de datos *|12 499|15 624|18 748|21 873|24 998|28 123|
|Velocidad de registro máx. (Mbps)|48|60|72|84|96|108|
|Cantidad máxima de trabajos (solicitudes) simultáneos|800|1,000|1,200|1400|1600|1800|
|Máximo de inicios de sesión simultáneos|800|1,000|1,200|1400|1600|1800|
|N.º máximo de sesiones simultáneas|30000|30000|30000|30000|30000|30000|
|Número de réplicas|4|4|4|4|4|4|
|AZ múltiple|No|No|No|No|No|No|
|Escalado horizontal de lectura|Sí|Sí|Sí|Sí|Sí|Sí|
|Almacenamiento de copia de seguridad incluido|1X el tamaño de base de datos|1X el tamaño de base de datos|1X el tamaño de base de datos|1X el tamaño de base de datos|1X el tamaño de base de datos|1X el tamaño de base de datos|

\* El valor máximo de los tamaños de e/s que oscilan entre 8 KB y 64 KB. Las IOPS reales dependen de la carga de trabajo. Para obtener más información, consulte [Regulación de E/S de los datos](resource-limits-logical-server.md#resource-governance).

> [!IMPORTANT]
> En algunas circunstancias, puede que deba reducir una base de datos para reclamar el espacio no utilizado. Para obtener más información, consulte [Administración del espacio de archivo en Azure SQL Database](file-space-manage.md).

### <a name="m-series-compute-generation-part-2"></a>Generación de proceso de la serie M (parte 2)

|Tamaño de proceso (objetivo de servicio)|BC_M_20|BC_M_24|BC_M_32|BC_M_64|BC_M_128|
|:---| ---:|---:|---:|---:|---:|
|Generación de procesos|Serie M|Serie M|Serie M|Serie M|Serie M|
|Núcleos virtuales|20|24|32|64|128|
|Memoria (GB)|588,6|706,3|941,8|1883,5|3767,0|
|Compatible con almacén de columnas|Sí|Sí|Sí|Sí|Sí|
|Almacenamiento OLTP en memoria (GB)|172|216|304|704|1768|
|Tamaño máximo de datos (GB)|1280|1536|2048|4096|4096|
|Tamaño máximo de registro (GB)|427|512|683|1024|1024|
|Tamaño máximo de datos de TempDB (GB)|4096|2048|1024|768|640|
|Tipo de almacenamiento|SSD local|SSD local|SSD local|SSD local|SSD local|
|Latencia de E/S (aproximada)|1-2 ms (escritura)<br>1-2 ms (lectura)|1-2 ms (escritura)<br>1-2 ms (lectura)|1-2 ms (escritura)<br>1-2 ms (lectura)|1-2 ms (escritura)<br>1-2 ms (lectura)|1-2 ms (escritura)<br>1-2 ms (lectura)|
|Número máx. de IOPS de datos *|31 248|37 497|49 996|99 993|160 000|
|Velocidad de registro máx. (Mbps)|120|144|192|264|264|
|Cantidad máxima de trabajos (solicitudes) simultáneos|2\.000|2,400|3\.200|6\.400|12.800|
|Máximo de inicios de sesión simultáneos|2\.000|2,400|3\.200|6\.400|12.800|
|N.º máximo de sesiones simultáneas|30000|30000|30000|30000|30000|
|Número de réplicas|4|4|4|4|4|
|AZ múltiple|No|No|No|No|No|
|Escalado horizontal de lectura|Sí|Sí|Sí|Sí|Sí|
|Almacenamiento de copia de seguridad incluido|1X el tamaño de base de datos|1X el tamaño de base de datos|1X el tamaño de base de datos|1X el tamaño de base de datos|1X el tamaño de base de datos|

\* El valor máximo de los tamaños de e/s que oscilan entre 8 KB y 64 KB. Las IOPS reales dependen de la carga de trabajo. Para obtener más información, consulte [Regulación de E/S de los datos](resource-limits-logical-server.md#resource-governance).

> [!IMPORTANT]
> En algunas circunstancias, puede que deba reducir una base de datos para reclamar el espacio no utilizado. Para obtener más información, consulte [Administración del espacio de archivo en Azure SQL Database](file-space-manage.md).



## <a name="next-steps"></a>Pasos siguientes

- Para conocer los límites de recursos de DTU para una base de datos única, consulte los [límites de recursos para bases de datos únicas que usan el modelo de compra de DTU](resource-limits-dtu-single-databases.md).
- Para conocer los límites de recursos de núcleos virtuales para grupos elásticos, consulte los [límites de recursos para grupos elásticos con el modelo de compra del núcleo virtual](resource-limits-vcore-elastic-pools.md).
- Para conocer los límites de recursos de DTU para grupos elásticos, consulte los [límites de recursos para grupos elásticos que usan el modelo de compra de DTU](resource-limits-dtu-elastic-pools.md).
- Para conocer los límites de recursos para Instancia administrada de SQL, consulte los [límites de recursos para Instancia administrada de SQL](../managed-instance/resource-limits.md).
- Para más información sobre los límites generales de Azure, consulte [Límites, cuotas y restricciones de suscripción y servicios de Microsoft Azure](../../azure-resource-manager/management/azure-subscription-service-limits.md).
- Para más información sobre los límites de recursos en un servidor, consulte la [información general sobre límites de recursos en un servidor](resource-limits-logical-server.md) relacionada con los niveles de servidor y suscripción.
