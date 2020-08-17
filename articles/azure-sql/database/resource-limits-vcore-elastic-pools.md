---
title: Límites de recursos de núcleo virtual de grupos elásticos
description: En esta página se describen algunos límites de recursos de núcleos virtuales comunes para grupos elásticos en Azure SQL Database.
services: sql-database
ms.service: sql-database
ms.subservice: elastic-pools
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: conceptual
author: oslake
ms.author: moslake
ms.reviewer: carlrab, sstein
ms.date: 07/21/2020
ms.openlocfilehash: f71daab55139f6b4690df50472928db466774cb3
ms.sourcegitcommit: 0e8a4671aa3f5a9a54231fea48bcfb432a1e528c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/24/2020
ms.locfileid: "87128274"
---
# <a name="resource-limits-for-elastic-pools-using-the-vcore-purchasing-model"></a>Límites de recursos para grupos elásticos que usan el modelo de compra de núcleo virtual
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

En este artículo se proporcionan los límites de recursos detallados para grupos elásticos y bases de datos agrupadas de Azure SQL Database que utilizan el modelo de compra de núcleo virtual.

Para información sobre los límites del modelo de compra de DTU, consulte [Límites de recursos para grupos elásticos que utilizan el modelo de compra de DTU](resource-limits-dtu-elastic-pools.md).

> [!IMPORTANT]
> En algunas circunstancias, puede que deba reducir una base de datos para reclamar el espacio no utilizado. Para obtener más información, consulte [Administración del espacio de archivo en Azure SQL Database](file-space-manage.md).

Puede establecer el nivel de servicio, el tamaño de proceso (objetivo de servicio) y la cantidad de almacenamiento mediante [Azure Portal](elastic-pool-manage.md#azure-portal), [PowerShell](elastic-pool-manage.md#powershell), la [CLI de Azure](elastic-pool-manage.md#azure-cli) o la [API REST](elastic-pool-manage.md#rest-api).

> [!IMPORTANT]
> Para obtener información sobre la guía y otras consideraciones del escalado, consulte [Escalar un grupo elástico](elastic-pool-scale.md).

## <a name="general-purpose---provisioned-compute---gen4"></a>Uso general: proceso aprovisionado: Gen4

> [!IMPORTANT]
> Las nuevas bases de datos de Gen4 ya no se admiten en las regiones Este de Australia o Sur de Brasil.

### <a name="general-purpose-service-tier-generation-4-compute-platform-part-1"></a>Nivel de servicio de uso general: Plataforma de procesos de generación 4 (parte 1)

|Tamaño de proceso (objetivo de servicio)|GP_Gen4_1|GP_Gen4_2|GP_Gen4_3|GP_Gen4_4|GP_Gen4_5|GP_Gen4_6
|:--- | --: |--: |--: |--: |--: |--: |
|Generación de procesos|Gen4|Gen4|Gen4|Gen4|Gen4|Gen4|
|Núcleos virtuales|1|2|3|4|5|6|
|Memoria (GB)|7|14|21|28|35|42|
|Máximo número de bases de datos por grupo <sup>1</sup>|100|200|500|500|500|500|
|Compatible con almacén de columnas|Sí|Sí|Sí|Sí|Sí|Sí|
|Almacenamiento OLTP en memoria (GB)|N/D|N/D|N/D|N/D|N/D|N/D|
|Tamaño máximo de datos (GB)|512|756|1536|1536|1536|2048|
|Tamaño máximo de registro|154|227|461|461|461|614|
|Tamaño máximo de datos de TempDB (GB)|32|64|96|128|160|192|
|Tipo de almacenamiento|Premium Storage (remoto)|Premium Storage (remoto)|Premium Storage (remoto)|Premium Storage (remoto)|Premium Storage (remoto)|Premium Storage (remoto)|
|Latencia de E/S (aproximada)|5-7 ms (escritura)<br>5-10 ms (lectura)|5-7 ms (escritura)<br>5-10 ms (lectura)|5-7 ms (escritura)<br>5-10 ms (lectura)|5-7 ms (escritura)<br>5-10 ms (lectura)|5-7 ms (escritura)<br>5-10 ms (lectura)|5-7 ms (escritura)<br>5-10 ms (lectura)|
|Número máximo de IOPS de datos por grupo <sup>2</sup> |400|800|1200|1600|2000|2400|
|Velocidad de registro máxima por grupo (MBps)|4,7|9,4|14,1|18,8|23,4|28,1|
|Número máximo de trabajos simultáneos por grupo (solicitudes) <sup>3</sup> |210|420|630|840|1050|1260|
|Número máximo de inicios de sesión simultáneos por grupo <sup>3</sup> |210|420|630|840|1050|1260|
|N.º máximo de sesiones simultáneas|30,000|30,000|30,000|30,000|30,000|30,000|
|Opciones de núcleo virtual mín./máx. de grupos elásticos por base de datos|0, 0.25, 0.5, 1|0, 0.25, 0.5, 1, 2|0, 0.25, 0.5, 1...3|0, 0.25, 0.5, 1...4|0, 0.25, 0.5, 1...5|0, 0.25, 0.5, 1...6|
|Número de réplicas|1|1|1|1|1|1|
|AZ múltiple|N/D|N/D|N/D|N/D|N/D|N/D|
|Escalado horizontal de lectura|N/D|N/D|N/D|N/D|N/D|N/D|
|Almacenamiento de copia de seguridad incluido|1X el tamaño de base de datos|1X el tamaño de base de datos|1X el tamaño de base de datos|1X el tamaño de base de datos|1X el tamaño de base de datos|1X el tamaño de base de datos|

<sup>1</sup> Consulte [Administración de recursos en grupos elásticos densos](elastic-pool-resource-management.md) para conocer las consideraciones adicionales.

<sup>2</sup> El valor máximo de los tamaños de E/S que oscilan entre 8 KB y 64 KB. Las IOPS reales dependen de la carga de trabajo. Para más información, consulte [Regulación de E/S de los datos](resource-limits-logical-server.md#resource-governance).

<sup>3</sup> Para obtener el máximo de trabajos simultáneos (solicitudes) para cualquier base de datos individual, consulte [Límites de recursos de base de datos única](resource-limits-vcore-single-databases.md). Por ejemplo, si el grupo elástico usa Gen5 y el valor máximo de núcleo virtual por base de datos se establece en 2, el valor de máximo de trabajos simultáneos es 200.  Si el número máximo de núcleo virtual por base de datos se establece en 0,5, el valor de máximo de trabajos simultáneos es 50, ya que en Gen5 hay un máximo de 100 trabajos simultáneos por núcleo virtual. Para otras configuraciones de memoria con núcleo virtual máximo por base de datos que sean un núcleo virtual o menos, la cantidad máxima de trabajos simultáneos se escala de forma similar.

### <a name="general-purpose-service-tier-generation-4-compute-platform-part-2"></a>Nivel de servicio de uso general: Plataforma de procesos de generación 4 (parte 2)

|Tamaño de proceso (objetivo de servicio)|GP_Gen4_7|GP_Gen4_8|GP_Gen4_9|GP_Gen4_10|GP_Gen4_16|GP_Gen4_24|
|:--- | --: |--: |--: |--: |--: |--: |
|Generación de procesos|Gen4|Gen4|Gen4|Gen4|Gen4|Gen4|
|Núcleos virtuales|7|8|9|10|16|24|
|Memoria (GB)|49|56|63|70|112|159,5|
|Máximo número de bases de datos por grupo <sup>1</sup>|500|500|500|500|500|500|
|Compatible con almacén de columnas|Sí|Sí|Sí|Sí|Sí|Sí|
|Almacenamiento OLTP en memoria (GB)|N/D|N/D|N/D|N/D|N/D|N/D|
|Tamaño máximo de datos (GB)|2048|2048|2048|2048|3584|4096|
|Tamaño máximo de registro (GB)|614|614|614|614|1075|1229|
|Tamaño máximo de datos de TempDB (GB)|224|256|288|320|512|768|
|Tipo de almacenamiento|Premium Storage (remoto)|Premium Storage (remoto)|Premium Storage (remoto)|Premium Storage (remoto)|Premium Storage (remoto)|Premium Storage (remoto)|
|Latencia de E/S (aproximada)|5-7 ms (escritura)<br>5-10 ms (lectura)|5-7 ms (escritura)<br>5-10 ms (lectura)|5-7 ms (escritura)<br>5-10 ms (lectura)|5-7 ms (escritura)<br>5-10 ms (lectura)|5-7 ms (escritura)<br>5-10 ms (lectura)|5-7 ms (escritura)<br>5-10 ms (lectura)|
|Número máximo de IOPS de datos por grupo <sup>2</sup>|2800|3200|3600|4000|6400|9600|
|Velocidad de registro máxima por grupo (MBps)|32,8|37.5|37.5|37.5|37.5|37.5|
|Número máximo de trabajos simultáneos por grupo (solicitudes) <sup>3</sup>|1470|1680|1890|2100|3360|5040|
|Cantidad máxima de inicios de sesión simultáneos por grupo (solicitudes) <sup>3</sup>|1470|1680|1890|2100|3360|5040|
|N.º máximo de sesiones simultáneas|30,000|30,000|30,000|30,000|30,000|30,000|
|Opciones de núcleo virtual mín./máx. de grupos elásticos por base de datos|0, 0.25, 0.5, 1...7|0, 0.25, 0.5, 1...8|0, 0.25, 0.5, 1...9|0, 0.25, 0.5, 1...10|0, 0.25, 0.5, 1...10, 16|0, 0.25, 0.5, 1...10, 16, 24|
|Número de réplicas|1|1|1|1|1|1|
|AZ múltiple|N/D|N/D|N/D|N/D|N/D|N/D|
|Escalado horizontal de lectura|N/D|N/D|N/D|N/D|N/D|N/D|
|Almacenamiento de copia de seguridad incluido|1X el tamaño de base de datos|1X el tamaño de base de datos|1X el tamaño de base de datos|1X el tamaño de base de datos|1X el tamaño de base de datos|1X el tamaño de base de datos|

<sup>1</sup> Consulte [Administración de recursos en grupos elásticos densos](elastic-pool-resource-management.md) para conocer las consideraciones adicionales.

<sup>2</sup> El valor máximo de los tamaños de E/S que oscilan entre 8 KB y 64 KB. Las IOPS reales dependen de la carga de trabajo. Para más información, consulte [Regulación de E/S de los datos](resource-limits-logical-server.md#resource-governance).    

<sup>3</sup> Para obtener el máximo de trabajos simultáneos (solicitudes) para cualquier base de datos individual, consulte [Límites de recursos de base de datos única](resource-limits-vcore-single-databases.md). Por ejemplo, si el grupo elástico usa Gen5 y el valor máximo de núcleo virtual por base de datos se establece en 2, el valor de máximo de trabajos simultáneos es 200.  Si el número máximo de núcleo virtual por base de datos se establece en 0,5, el valor de máximo de trabajos simultáneos es 50, ya que en Gen5 hay un máximo de 100 trabajos simultáneos por núcleo virtual. Para otras configuraciones de memoria con núcleo virtual máximo por base de datos que sean un núcleo virtual o menos, la cantidad máxima de trabajos simultáneos se escala de forma similar.

## <a name="general-purpose---provisioned-compute---gen5"></a>Uso general: proceso aprovisionado: Gen5

### <a name="general-purpose-service-tier-generation-5-compute-platform-part-1"></a>Nivel de servicio de uso general: Plataforma de procesos de generación 5 (parte 1)

|Tamaño de proceso (objetivo de servicio)|GP_Gen5_2|GP_Gen5_4|GP_Gen5_6|GP_Gen5_8|GP_Gen5_10|GP_Gen5_12|GP_Gen5_14|
|:--- | --: |--: |--: |--: |---: | --: |--: |
|Generación de procesos|Gen5|Gen5|Gen5|Gen5|Gen5|Gen5|Gen5|
|Núcleos virtuales|2|4|6|8|10|12|14|
|Memoria (GB)|10,4|20,8|31,1|41,5|51,9|62,3|72,7|
|Máximo número de bases de datos por grupo <sup>1</sup>|100|200|500|500|500|500|500|
|Compatible con almacén de columnas|Sí|Sí|Sí|Sí|Sí|Sí|Sí|
|Almacenamiento OLTP en memoria (GB)|N/D|N/D|N/D|N/D|N/D|N/D|N/D|
|Tamaño máximo de datos (GB)|512|756|1536|1536|1536|2048|2048|
|Tamaño máximo de registro (GB)|154|227|461|461|461|614|614|
|Tamaño máximo de datos de TempDB (GB)|64|128|192|256|320|384|448|
|Tipo de almacenamiento|Premium Storage (remoto)|Premium Storage (remoto)|Premium Storage (remoto)|Premium Storage (remoto)|Premium Storage (remoto)|Premium Storage (remoto)|Premium Storage (remoto)|
|Latencia de E/S (aproximada)|5-7 ms (escritura)<br>5-10 ms (lectura)|5-7 ms (escritura)<br>5-10 ms (lectura)|5-7 ms (escritura)<br>5-10 ms (lectura)|5-7 ms (escritura)<br>5-10 ms (lectura)|5-7 ms (escritura)<br>5-10 ms (lectura)|5-7 ms (escritura)<br>5-10 ms (lectura)|5-7 ms (escritura)<br>5-10 ms (lectura)|
|Número máximo de IOPS de datos por grupo <sup>2</sup>|800|1600|2400|3200|4000|4800|5600|
|Velocidad de registro máxima por grupo (MBps)|9,4|18,8|28,1|37.5|37.5|37.5|37.5|
|Número máximo de trabajos simultáneos por grupo (solicitudes) <sup>3</sup>|210|420|630|840|1050|1260|1470|
|Número máximo de inicios de sesión simultáneos por grupo (solicitudes) <sup>3</sup>|210|420|630|840|1050|1260|1470|
|N.º máximo de sesiones simultáneas|30,000|30,000|30,000|30,000|30,000|30,000|30,000|
|Opciones de núcleo virtual mín./máx. de grupos elásticos por base de datos|0, 0.25, 0.5, 1, 2|0, 0.25, 0.5, 1...4|0, 0.25, 0.5, 1...6|0, 0.25, 0.5, 1...8|0, 0.25, 0.5, 1...10|0, 0.25, 0.5, 1...12|0, 0.25, 0.5, 1...14|
|Número de réplicas|1|1|1|1|1|1|1|
|AZ múltiple|N/D|N/D|N/D|N/D|N/D|N/D|N/D|
|Escalado horizontal de lectura|N/D|N/D|N/D|N/D|N/D|N/D|N/D|
|Almacenamiento de copia de seguridad incluido|1X el tamaño de base de datos|1X el tamaño de base de datos|1X el tamaño de base de datos|1X el tamaño de base de datos|1X el tamaño de base de datos|1X el tamaño de base de datos|1X el tamaño de base de datos|

<sup>1</sup> Consulte [Administración de recursos en grupos elásticos densos](elastic-pool-resource-management.md) para conocer las consideraciones adicionales.

<sup>2</sup> El valor máximo de los tamaños de E/S que oscilan entre 8 KB y 64 KB. Las IOPS reales dependen de la carga de trabajo. Para más información, consulte [Regulación de E/S de los datos](resource-limits-logical-server.md#resource-governance).

<sup>3</sup> Para obtener el máximo de trabajos simultáneos (solicitudes) para cualquier base de datos individual, consulte [Límites de recursos de base de datos única](resource-limits-vcore-single-databases.md). Por ejemplo, si el grupo elástico usa Gen5 y el valor máximo de núcleo virtual por base de datos se establece en 2, el valor de máximo de trabajos simultáneos es 200.  Si el número máximo de núcleo virtual por base de datos se establece en 0,5, el valor de máximo de trabajos simultáneos es 50, ya que en Gen5 hay un máximo de 100 trabajos simultáneos por núcleo virtual. Para otras configuraciones de memoria con núcleo virtual máximo por base de datos que sean un núcleo virtual o menos, la cantidad máxima de trabajos simultáneos se escala de forma similar.

### <a name="general-purpose-service-tier-generation-5-compute-platform-part-2"></a>Nivel de servicio de uso general: Plataforma de procesos de generación 5 (parte 2)

|Tamaño de proceso (objetivo de servicio)|GP_Gen5_16|GP_Gen5_18|GP_Gen5_20|GP_Gen5_24|GP_Gen5_32|GP_Gen5_40|GP_Gen5_80|
|:--- | --: |--: |--: |--: |---: | --: |--: |
|Generación de procesos|Gen5|Gen5|Gen5|Gen5|Gen5|Gen5|Gen5|
|Núcleos virtuales|16|18|20|24|32|40|80|
|Memoria (GB)|83|93,4|103,8|124,6|166,1|207,6|415,2|
|Máximo número de bases de datos por grupo <sup>1</sup>|500|500|500|500|500|500|500|
|Compatible con almacén de columnas|Sí|Sí|Sí|Sí|Sí|Sí|Sí|
|Almacenamiento OLTP en memoria (GB)|N/D|N/D|N/D|N/D|N/D|N/D|N/D|
|Tamaño máximo de datos (GB)|2048|3072|3072|3072|4096|4096|4096|
|Tamaño máximo de registro (GB)|614|922|922|922|1229|1229|1229|
|Tamaño máximo de datos de TempDB (GB)|512|576|640|768|1024|1280|2560|
|Tipo de almacenamiento|Premium Storage (remoto)|Premium Storage (remoto)|Premium Storage (remoto)|Premium Storage (remoto)|Premium Storage (remoto)|Premium Storage (remoto)|Premium Storage (remoto)|
|Latencia de E/S (aproximada)|5-7 ms (escritura)<br>5-10 ms (lectura)|5-7 ms (escritura)<br>5-10 ms (lectura)|5-7 ms (escritura)<br>5-10 ms (lectura)|5-7 ms (escritura)<br>5-10 ms (lectura)|5-7 ms (escritura)<br>5-10 ms (lectura)|5-7 ms (escritura)<br>5-10 ms (lectura)|5-7 ms (escritura)<br>5-10 ms (lectura)|
|Número máximo de IOPS de datos por grupo <sup>2</sup> |6\.400|7200|8,000|9600|12.800|16 000|16 000|
|Velocidad de registro máxima por grupo (MBps)|37.5|37.5|37.5|37.5|37.5|37.5|37.5|
|Número máximo de trabajos simultáneos por grupo (solicitudes) <sup>3</sup>|1680|1890|2100|2520|3360|4200|8400|
|Número máximo de inicios de sesión simultáneos por grupo (solicitudes) <sup>3</sup>|1680|1890|2100|2520|3360|4200|8400|
|N.º máximo de sesiones simultáneas|30,000|30,000|30,000|30,000|30,000|30,000|30,000|
|Opciones de núcleo virtual mín./máx. de grupos elásticos por base de datos|0, 0.25, 0.5, 1...16|0, 0.25, 0.5, 1...18|0, 0.25, 0.5, 1...20|0, 0.25, 0.5, 1...20, 24|0, 0.25, 0.5, 1...20, 24, 32|0, 0.25, 0.5, 1...16, 24, 32, 40|0, 0.25, 0.5, 1...16, 24, 32, 40, 80|
|Número de réplicas|1|1|1|1|1|1|1|
|AZ múltiple|N/D|N/D|N/D|N/D|N/D|N/D|N/D|
|Escalado horizontal de lectura|N/D|N/D|N/D|N/D|N/D|N/D|N/D|
|Almacenamiento de copia de seguridad incluido|1X el tamaño de base de datos|1X el tamaño de base de datos|1X el tamaño de base de datos|1X el tamaño de base de datos|1X el tamaño de base de datos|1X el tamaño de base de datos|1X el tamaño de base de datos|

<sup>1</sup> Consulte [Administración de recursos en grupos elásticos densos](elastic-pool-resource-management.md) para conocer las consideraciones adicionales.

<sup>2</sup> El valor máximo de los tamaños de E/S que oscilan entre 8 KB y 64 KB. Las IOPS reales dependen de la carga de trabajo. Para más información, consulte [Regulación de E/S de los datos](resource-limits-logical-server.md#resource-governance).

<sup>3</sup> Para obtener el máximo de trabajos simultáneos (solicitudes) para cualquier base de datos individual, consulte [Límites de recursos de base de datos única](resource-limits-vcore-single-databases.md). Por ejemplo, si el grupo elástico usa Gen5 y el valor máximo de núcleo virtual por base de datos se establece en 2, el valor de máximo de trabajos simultáneos es 200.  Si el número máximo de núcleo virtual por base de datos se establece en 0,5, el valor de máximo de trabajos simultáneos es 50, ya que en Gen5 hay un máximo de 100 trabajos simultáneos por núcleo virtual. Para otras configuraciones de memoria con núcleo virtual máximo por base de datos que sean un núcleo virtual o menos, la cantidad máxima de trabajos simultáneos se escala de forma similar.

## <a name="general-purpose---provisioned-compute---fsv2-series"></a>Uso general: proceso aprovisionado: serie Fsv2

### <a name="fsv2-series-compute-generation-part-1"></a>Generación de proceso de la serie Fsv2 (parte 1)

|Tamaño de proceso (objetivo de servicio)|GP_Fsv2_8|GP_Fsv2_10|GP_Fsv2_12|GP_Fsv2_14| GP_Fsv2_16|
|:---| ---:|---:|---:|---:|---:|
|Generación de procesos|Serie Fsv2|Serie Fsv2|Serie Fsv2|Serie Fsv2|Serie Fsv2|
|Núcleos virtuales|8|10|12|14|16|
|Memoria (GB)|15,1|18,9|22,7|26,5|30,2|
|Máximo número de bases de datos por grupo <sup>1</sup>|500|500|500|500|500|
|Compatible con almacén de columnas|Sí|Sí|Sí|Sí|Sí|
|Almacenamiento OLTP en memoria (GB)|N/D|No aplicable|No aplicable|N/D|N/D|
|Tamaño máximo de datos (GB)|1024|1024|1024|1024|1536|
|Tamaño máximo de registro (GB)|336|336|336|336|512|
|Tamaño máximo de datos de TempDB (GB)|333|333|333|333|333|
|Tipo de almacenamiento|SSD remoto|SSD remoto|SSD remoto|SSD remoto|SSD remoto|
|Latencia de E/S (aproximada)|5-7 ms (escritura)<br>5-10 ms (lectura)|5-7 ms (escritura)<br>5-10 ms (lectura)|5-7 ms (escritura)<br>5-10 ms (lectura)|5-7 ms (escritura)<br>5-10 ms (lectura)|5-7 ms (escritura)<br>5-10 ms (lectura)|
|Número máximo de IOPS de datos por grupo <sup>2</sup>|2560|3200|3840|4480|5120|
|Velocidad de registro máxima por grupo (MBps)|30|30|30|30|30|
|Número máximo de trabajos simultáneos por grupo (solicitudes) <sup>3</sup>|400|500|600|700|800|
|Número máximo de inicios de sesión simultáneos por grupo (solicitudes) <sup>3</sup>|800|1000|1200|1400|1600|
|N.º máximo de sesiones simultáneas|30,000|30,000|30,000|30,000|30,000|
|Opciones de núcleo virtual mín./máx. de grupos elásticos por base de datos|0 a 8|0 a 10|0 a 12|0 a 14|0 a 16|
|Número de réplicas|1|1|1|1|1|
|AZ múltiple|No aplicable|No aplicable|No aplicable|No aplicable|No aplicable|
|Escalado horizontal de lectura|N/D|No aplicable|No aplicable|No aplicable|N/D|
|Almacenamiento de copia de seguridad incluido|1X el tamaño de base de datos|1X el tamaño de base de datos|1X el tamaño de base de datos|1X el tamaño de base de datos|1X el tamaño de base de datos|


<sup>1</sup> Consulte [Administración de recursos en grupos elásticos densos](elastic-pool-resource-management.md) para conocer las consideraciones adicionales.

<sup>2</sup> El valor máximo de los tamaños de E/S que oscilan entre 8 KB y 64 KB. Las IOPS reales dependen de la carga de trabajo. Para más información, consulte [Regulación de E/S de los datos](resource-limits-logical-server.md#resource-governance).

<sup>3</sup> Para obtener el máximo de trabajos simultáneos (solicitudes) para cualquier base de datos individual, consulte [Límites de recursos de base de datos única](resource-limits-vcore-single-databases.md). Por ejemplo, si el grupo elástico usa Gen5 y el valor máximo de núcleo virtual por base de datos se establece en 2, el valor de máximo de trabajos simultáneos es 200.  Si el número máximo de núcleo virtual por base de datos se establece en 0,5, el valor de máximo de trabajos simultáneos es 50, ya que en Gen5 hay un máximo de 100 trabajos simultáneos por núcleo virtual. Para otras configuraciones de memoria con núcleo virtual máximo por base de datos que sean un núcleo virtual o menos, la cantidad máxima de trabajos simultáneos se escala de forma similar.

### <a name="fsv2-series-compute-generation-part-2"></a>Generación de proceso de la serie Fsv2 (parte 2)

|Tamaño de proceso (objetivo de servicio)|GP_Fsv2_18|GP_Fsv2_20|GP_Fsv2_24|GP_Fsv2_32| GP_Fsv2_36|GP_Fsv2_72|
|:---| ---:|---:|---:|---:|---:|---:|
|Generación de procesos|Serie Fsv2|Serie Fsv2|Serie Fsv2|Serie Fsv2|Serie Fsv2|Serie Fsv2|
|Núcleos virtuales|18|20|24|32|36|72|
|Memoria (GB)|34,0|37,8|45,4|60,5|68,0|136,0|
|Máximo número de bases de datos por grupo <sup>1</sup>|500|500|500|500|500|
|Compatible con almacén de columnas|Sí|Sí|Sí|Sí|Sí|Sí|
|Almacenamiento OLTP en memoria (GB)|No aplicable|No aplicable|No aplicable|No aplicable|No aplicable|No aplicable|
|Tamaño máximo de datos (GB)|1536|1536|1536|3072|3072|4096|
|Tamaño máximo de registro (GB)|512|512|512|1024|1024|1024|
|Tamaño máximo de datos de TempDB (GB)|83,25|92,5|111|148|166,5|333|
|Tipo de almacenamiento|SSD remoto|SSD remoto|SSD remoto|SSD remoto|SSD remoto|SSD remoto|
|Latencia de E/S (aproximada)|5-7 ms (escritura)<br>5-10 ms (lectura)|5-7 ms (escritura)<br>5-10 ms (lectura)|5-7 ms (escritura)<br>5-10 ms (lectura)|5-7 ms (escritura)<br>5-10 ms (lectura)|5-7 ms (escritura)<br>5-10 ms (lectura)|5-7 ms (escritura)<br>5-10 ms (lectura)|
|Número máximo de IOPS de datos por grupo <sup>2</sup>|5760|6400|7680|10240|11 520|23 040|
|Velocidad de registro máxima por grupo (MBps)|30|30|30|30|30|30|
|Número máximo de trabajos simultáneos por grupo (solicitudes) <sup>3</sup>|900|1000|1200|1600|1800|3600|
|Número máximo de inicios de sesión simultáneos por grupo (solicitudes) <sup>3</sup>|1800|2000|2400|3200|3600|7200|
|N.º máximo de sesiones simultáneas|30,000|30,000|30,000|30,000|30,000|30,000|
|Opciones de núcleo virtual mín./máx. de grupos elásticos por base de datos|0 a 18|0-20|0-24|0 a 32|0 a 36|0-72|
|Número de réplicas|1|1|1|1|1|1|
|AZ múltiple|No aplicable|No aplicable|No aplicable|No aplicable|No aplicable|No aplicable|
|Escalado horizontal de lectura|N/D|No aplicable|No aplicable|No aplicable|No aplicable|N/D|
|Almacenamiento de copia de seguridad incluido|1X el tamaño de base de datos|1X el tamaño de base de datos|1X el tamaño de base de datos|1X el tamaño de base de datos|1X el tamaño de base de datos|1X el tamaño de base de datos|

<sup>1</sup> Consulte [Administración de recursos en grupos elásticos densos](elastic-pool-resource-management.md) para conocer las consideraciones adicionales.

<sup>2</sup> El valor máximo de los tamaños de E/S que oscilan entre 8 KB y 64 KB. Las IOPS reales dependen de la carga de trabajo. Para más información, consulte [Regulación de E/S de los datos](resource-limits-logical-server.md#resource-governance).

<sup>3</sup> Para obtener el máximo de trabajos simultáneos (solicitudes) para cualquier base de datos individual, consulte [Límites de recursos de base de datos única](resource-limits-vcore-single-databases.md). Por ejemplo, si el grupo elástico usa Gen5 y el valor máximo de núcleo virtual por base de datos se establece en 2, el valor de máximo de trabajos simultáneos es 200.  Si el número máximo de núcleo virtual por base de datos se establece en 0,5, el valor de máximo de trabajos simultáneos es 50, ya que en Gen5 hay un máximo de 100 trabajos simultáneos por núcleo virtual. Para otras configuraciones de memoria con núcleo virtual máximo por base de datos que sean un núcleo virtual o menos, la cantidad máxima de trabajos simultáneos se escala de forma similar.

## <a name="business-critical---provisioned-compute---gen4"></a>Crítico para la empresa: proceso aprovisionado: Gen4

> [!IMPORTANT]
> Las nuevas bases de datos de Gen4 ya no se admiten en las regiones Este de Australia o Sur de Brasil.

### <a name="business-critical-service-tier-generation-4-compute-platform-part-1"></a>Nivel de servicio crítico para la empresa: Plataforma de procesos de generación 4 (parte 1)

|Tamaño de proceso (objetivo de servicio)|BC_Gen4_2|BC_Gen4_3|BC_Gen4_4|BC_Gen4_5|BC_Gen4_6|
|:--- | --: |--: |--: |--: |--: |--: |
|Generación de procesos|Gen4|Gen4|Gen4|Gen4|Gen4|
|Núcleos virtuales|2|3|4|5|6|
|Memoria (GB)|14|21|28|35|42|
|Máximo número de bases de datos por grupo <sup>1</sup>|50|100|100|100|100|
|Compatible con almacén de columnas|Sí|Sí|Sí|Sí|Sí|
|Almacenamiento OLTP en memoria (GB)|2|3|4|5|6|
|Tipo de almacenamiento|SSD local|SSD local|SSD local|SSD local|SSD local|
|Tamaño máximo de datos (GB)|1024|1024|1024|1024|1024|
|Tamaño máximo de registro (GB)|307|307|307|307|307|
|Tamaño máximo de datos de TempDB (GB)|64|96|128|160|192|
|Latencia de E/S (aproximada)|1-2 ms (escritura)<br>1-2 ms (lectura)|1-2 ms (escritura)<br>1-2 ms (lectura)|1-2 ms (escritura)<br>1-2 ms (lectura)|1-2 ms (escritura)<br>1-2 ms (lectura)|1-2 ms (escritura)<br>1-2 ms (lectura)|
|Número máximo de IOPS de datos por grupo <sup>2</sup>|9000|13 500|18 000|22 500|27 000|
|Velocidad de registro máxima por grupo (MBps)|20|30|40|50|60|
|Número máximo de trabajos simultáneos por grupo (solicitudes) <sup>3</sup>|420|630|840|1050|1260|
|Número máximo de inicios de sesión simultáneos por grupo (solicitudes) <sup>3</sup>|420|630|840|1050|1260|
|N.º máximo de sesiones simultáneas|30,000|30,000|30,000|30,000|30,000|
|Opciones de núcleo virtual mín./máx. de grupos elásticos por base de datos|0, 0.25, 0.5, 1, 2|0, 0.25, 0.5, 1...3|0, 0.25, 0.5, 1...4|0, 0.25, 0.5, 1...5|0, 0.25, 0.5, 1...6|
|Número de réplicas|4|4|4|4|4|
|AZ múltiple|Sí|Sí|Sí|Sí|Sí|
|Escalado horizontal de lectura|Sí|Sí|Sí|Sí|Sí|
|Almacenamiento de copia de seguridad incluido|1X el tamaño de base de datos|1X el tamaño de base de datos|1X el tamaño de base de datos|1X el tamaño de base de datos|1X el tamaño de base de datos|

<sup>1</sup> Consulte [Administración de recursos en grupos elásticos densos](elastic-pool-resource-management.md) para conocer las consideraciones adicionales.

<sup>2</sup> El valor máximo de los tamaños de E/S que oscilan entre 8 KB y 64 KB. Las IOPS reales dependen de la carga de trabajo. Para más información, consulte [Regulación de E/S de los datos](resource-limits-logical-server.md#resource-governance).

<sup>3</sup> Para obtener el máximo de trabajos simultáneos (solicitudes) para cualquier base de datos individual, consulte [Límites de recursos de base de datos única](resource-limits-vcore-single-databases.md). Por ejemplo, si el grupo elástico usa Gen5 y el valor máximo de núcleo virtual por base de datos se establece en 2, el valor de máximo de trabajos simultáneos es 200.  Si el número máximo de núcleo virtual por base de datos se establece en 0,5, el valor de máximo de trabajos simultáneos es 50, ya que en Gen5 hay un máximo de 100 trabajos simultáneos por núcleo virtual. Para otras configuraciones de memoria con núcleo virtual máximo por base de datos que sean un núcleo virtual o menos, la cantidad máxima de trabajos simultáneos se escala de forma similar.

### <a name="business-critical-service-tier-generation-4-compute-platform-part-2"></a>Nivel de servicio crítico para la empresa: Plataforma de procesos de generación 4 (parte 2)

|Tamaño de proceso (objetivo de servicio)|BC_Gen4_7|BC_Gen4_8|BC_Gen4_9|BC_Gen4_10|BC_Gen4_16|BC_Gen4_24|
|:--- | --: |--: |--: |--: |--: |--: |
|Generación de procesos|Gen4|Gen4|Gen4|Gen4|Gen4|Gen4|
|Núcleos virtuales|7|8|9|10|16|24|
|Memoria (GB)|49|56|63|70|112|159,5|
|Máximo número de bases de datos por grupo <sup>1</sup>|100|100|100|100|100|100|
|Compatible con almacén de columnas|N/D|No aplicable|No aplicable|No aplicable|No aplicable|N/D|
|Almacenamiento OLTP en memoria (GB)|7|8|9.5|11|20|36|
|Tipo de almacenamiento|SSD local|SSD local|SSD local|SSD local|SSD local|SSD local|
|Tamaño máximo de datos (GB)|1024|1024|1024|1024|1024|1024|
|Tamaño máximo de registro (GB)|307|307|307|307|307|307|
|Tamaño máximo de datos de TempDB (GB)|224|256|288|320|512|768|
|Latencia de E/S (aproximada)|1-2 ms (escritura)<br>1-2 ms (lectura)|1-2 ms (escritura)<br>1-2 ms (lectura)|1-2 ms (escritura)<br>1-2 ms (lectura)|1-2 ms (escritura)<br>1-2 ms (lectura)|1-2 ms (escritura)<br>1-2 ms (lectura)|1-2 ms (escritura)<br>1-2 ms (lectura)|
|Número máximo de IOPS de datos por grupo <sup>2</sup>|31 500|36 000|40 500|45 000|72 000|96 000|
|Velocidad de registro máxima por grupo (MBps)|70|80|80|80|80|80|
|Número máximo de trabajos simultáneos por grupo (solicitudes) <sup>3</sup>|1470|1680|1890|2100|3360|5040|
|Número máximo de inicios de sesión simultáneos por grupo (solicitudes) <sup>3</sup>|1470|1680|1890|2100|3360|5040|
|N.º máximo de sesiones simultáneas|30,000|30,000|30,000|30,000|30,000|30,000|
|Opciones de núcleo virtual mín./máx. de grupos elásticos por base de datos|0, 0.25, 0.5, 1...7|0, 0.25, 0.5, 1...8|0, 0.25, 0.5, 1...9|0, 0.25, 0.5, 1...10|0, 0.25, 0.5, 1...10, 16|0, 0.25, 0.5, 1...10, 16, 24|
|Número de réplicas|4|4|4|4|4|4|
|AZ múltiple|Sí|Sí|Sí|Sí|Sí|Sí|
|Escalado horizontal de lectura|Sí|Sí|Sí|Sí|Sí|Sí|
|Almacenamiento de copia de seguridad incluido|1X el tamaño de base de datos|1X el tamaño de base de datos|1X el tamaño de base de datos|1X el tamaño de base de datos|1X el tamaño de base de datos|1X el tamaño de base de datos|

<sup>1</sup> Consulte [Administración de recursos en grupos elásticos densos](elastic-pool-resource-management.md) para conocer las consideraciones adicionales.

<sup>2</sup> El valor máximo de los tamaños de E/S que oscilan entre 8 KB y 64 KB. Las IOPS reales dependen de la carga de trabajo. Para más información, consulte [Regulación de E/S de los datos](resource-limits-logical-server.md#resource-governance).

<sup>3</sup> Para obtener el máximo de trabajos simultáneos (solicitudes) para cualquier base de datos individual, consulte [Límites de recursos de base de datos única](resource-limits-vcore-single-databases.md). Por ejemplo, si el grupo elástico usa Gen5 y el valor máximo de núcleo virtual por base de datos se establece en 2, el valor de máximo de trabajos simultáneos es 200.  Si el número máximo de núcleo virtual por base de datos se establece en 0,5, el valor de máximo de trabajos simultáneos es 50, ya que en Gen5 hay un máximo de 100 trabajos simultáneos por núcleo virtual. Para otras configuraciones de memoria con núcleo virtual máximo por base de datos que sean un núcleo virtual o menos, la cantidad máxima de trabajos simultáneos se escala de forma similar.

## <a name="business-critical---provisioned-compute---gen5"></a>Crítico para la empresa: proceso aprovisionado: Gen5

### <a name="business-critical-service-tier-generation-5-compute-platform-part-1"></a>Nivel de servicio crítico para la empresa: Plataforma de procesos de generación 5 (parte 1)

|Tamaño de proceso (objetivo de servicio)|BC_Gen5_4|BC_Gen5_6|BC_Gen5_8|BC_Gen5_10|BC_Gen5_12|BC_Gen5_14|
|:--- | --: |--: |--: |--: |---: | --: |--: |
|Generación de procesos|Gen5|Gen5|Gen5|Gen5|Gen5|Gen5|
|Núcleos virtuales|4|6|8|10|12|14|
|Memoria (GB)|20,8|31,1|41,5|51,9|62,3|72,7|
|Máximo número de bases de datos por grupo <sup>1</sup>|50|100|100|100|100|100|
|Compatible con almacén de columnas|Sí|Sí|Sí|Sí|Sí|Sí|
|Almacenamiento OLTP en memoria (GB)|3,14|4.71|6,28|8,65|11,02|13,39|
|Tamaño máximo de datos (GB)|1024|1536|1536|1536|3072|3072|
|Tamaño máximo de registro (GB)|307|307|461|461|922|922|
|Tamaño máximo de datos de TempDB (GB)|128|192|256|320|384|448|
|Tipo de almacenamiento|SSD local|SSD local|SSD local|SSD local|SSD local|SSD local|
|Latencia de E/S (aproximada)|1-2 ms (escritura)<br>1-2 ms (lectura)|1-2 ms (escritura)<br>1-2 ms (lectura)|1-2 ms (escritura)<br>1-2 ms (lectura)|1-2 ms (escritura)<br>1-2 ms (lectura)|1-2 ms (escritura)<br>1-2 ms (lectura)|1-2 ms (escritura)<br>1-2 ms (lectura)|
|Número máximo de IOPS de datos por grupo <sup>2</sup>|18 000|27 000|36 000|45 000|54 000|63 000|
|Velocidad de registro máxima por grupo (MBps)|60|90|120|120|120|120|
|Número máximo de trabajos simultáneos por grupo (solicitudes) <sup>3</sup>|420|630|840|1050|1260|1470|
|Número máximo de inicios de sesión simultáneos por grupo (solicitudes) <sup>3</sup>|420|630|840|1050|1260|1470|
|N.º máximo de sesiones simultáneas|30,000|30,000|30,000|30,000|30,000|30,000|
|Opciones de núcleo virtual mín./máx. de grupos elásticos por base de datos|0, 0.25, 0.5, 1...4|0, 0.25, 0.5, 1...6|0, 0.25, 0.5, 1...8|0, 0.25, 0.5, 1...10|0, 0.25, 0.5, 1...12|0, 0.25, 0.5, 1...14|
|Número de réplicas|4|4|4|4|4|4|
|AZ múltiple|Sí|Sí|Sí|Sí|Sí|Sí|
|Escalado horizontal de lectura|Sí|Sí|Sí|Sí|Sí|Sí|
|Almacenamiento de copia de seguridad incluido|1X el tamaño de base de datos|1X el tamaño de base de datos|1X el tamaño de base de datos|1X el tamaño de base de datos|1X el tamaño de base de datos|1X el tamaño de base de datos|

<sup>1</sup> Consulte [Administración de recursos en grupos elásticos densos](elastic-pool-resource-management.md) para conocer las consideraciones adicionales.

<sup>2</sup> El valor máximo de los tamaños de E/S que oscilan entre 8 KB y 64 KB. Las IOPS reales dependen de la carga de trabajo. Para más información, consulte [Regulación de E/S de los datos](resource-limits-logical-server.md#resource-governance).

<sup>3</sup> Para obtener el máximo de trabajos simultáneos (solicitudes) para cualquier base de datos individual, consulte [Límites de recursos de base de datos única](resource-limits-vcore-single-databases.md). Por ejemplo, si el grupo elástico usa Gen5 y el valor máximo de núcleo virtual por base de datos se establece en 2, el valor de máximo de trabajos simultáneos es 200.  Si el número máximo de núcleo virtual por base de datos se establece en 0,5, el valor de máximo de trabajos simultáneos es 50, ya que en Gen5 hay un máximo de 100 trabajos simultáneos por núcleo virtual. Para otras configuraciones de memoria con núcleo virtual máximo por base de datos que sean un núcleo virtual o menos, la cantidad máxima de trabajos simultáneos se escala de forma similar.

### <a name="business-critical-service-tier-generation-5-compute-platform-part-2"></a>Nivel de servicio crítico para la empresa: Plataforma de procesos de generación 5 (parte 2)

|Tamaño de proceso (objetivo de servicio)|BC_Gen5_16|BC_Gen5_18|BC_Gen5_20|BC_Gen5_24|BC_Gen5_32|BC_Gen5_40|BC_Gen5_80|
|:--- | --: |--: |--: |--: |---: | --: |--: |
|Generación de procesos|Gen5|Gen5|Gen5|Gen5|Gen5|Gen5|Gen5|
|Núcleos virtuales|16|18|20|24|32|40|80|
|Memoria (GB)|83|93,4|103,8|124,6|166,1|207,6|415,2|
|Máximo número de bases de datos por grupo <sup>1</sup>|100|100|100|100|100|100|100|
|Compatible con almacén de columnas|Sí|Sí|Sí|Sí|Sí|Sí|Sí|
|Almacenamiento OLTP en memoria (GB)|15,77|18,14|20.51|25,25|37,94|52,23|131,68|
|Tamaño máximo de datos (GB)|3072|3072|3072|4096|4096|4096|4096|
|Tamaño máximo de registro (GB)|922|922|922|1229|1229|1229|1229|
|Tamaño máximo de datos de TempDB (GB)|512|576|640|768|1024|1280|2560|
|Tipo de almacenamiento|SSD local|SSD local|SSD local|SSD local|SSD local|SSD local|SSD local|
|Latencia de E/S (aproximada)|1-2 ms (escritura)<br>1-2 ms (lectura)|1-2 ms (escritura)<br>1-2 ms (lectura)|1-2 ms (escritura)<br>1-2 ms (lectura)|1-2 ms (escritura)<br>1-2 ms (lectura)|1-2 ms (escritura)<br>1-2 ms (lectura)|1-2 ms (escritura)<br>1-2 ms (lectura)|1-2 ms (escritura)<br>1-2 ms (lectura)|
|Número máximo de IOPS de datos por grupo <sup>2</sup>|72 000|81 000|90 000|108 000|144 000|180,000|256 000|
|Velocidad de registro máxima por grupo (MBps)|120|120|120|120|120|120|120|
|Número máximo de trabajos simultáneos por grupo (solicitudes) <sup>3</sup>|1680|1890|2100|2520|3360|4200|8400|
|Número máximo de inicios de sesión simultáneos por grupo (solicitudes) <sup>3</sup>|1680|1890|2100|2520|3360|4200|8400|
|N.º máximo de sesiones simultáneas|30,000|30,000|30,000|30,000|30,000|30,000|30,000|
|Opciones de núcleo virtual mín./máx. de grupos elásticos por base de datos|0, 0.25, 0.5, 1...16|0, 0.25, 0.5, 1...18|0, 0.25, 0.5, 1...20|0, 0.25, 0.5, 1...20, 24|0, 0.25, 0.5, 1...20, 24, 32|0, 0.25, 0.5, 1...20, 24, 32, 40|0, 0.25, 0.5, 1...20, 24, 32, 40, 80|
|Número de réplicas|4|4|4|4|4|4|4|
|AZ múltiple|Sí|Sí|Sí|Sí|Sí|Sí|Sí|
|Escalado horizontal de lectura|Sí|Sí|Sí|Sí|Sí|Sí|Sí|
|Almacenamiento de copia de seguridad incluido|1X el tamaño de base de datos|1X el tamaño de base de datos|1X el tamaño de base de datos|1X el tamaño de base de datos|1X el tamaño de base de datos|1X el tamaño de base de datos|1X el tamaño de base de datos|

<sup>1</sup> Consulte [Administración de recursos en grupos elásticos densos](elastic-pool-resource-management.md) para conocer las consideraciones adicionales.

<sup>2</sup> El valor máximo de los tamaños de E/S que oscilan entre 8 KB y 64 KB. Las IOPS reales dependen de la carga de trabajo. Para más información, consulte [Regulación de E/S de los datos](resource-limits-logical-server.md#resource-governance).

<sup>3</sup> Para obtener el máximo de trabajos simultáneos (solicitudes) para cualquier base de datos individual, consulte [Límites de recursos de base de datos única](resource-limits-vcore-single-databases.md). Por ejemplo, si el grupo elástico usa Gen5 y el valor máximo de núcleo virtual por base de datos se establece en 2, el valor de máximo de trabajos simultáneos es 200.  Si el número máximo de núcleo virtual por base de datos se establece en 0,5, el valor de máximo de trabajos simultáneos es 50, ya que en Gen5 hay un máximo de 100 trabajos simultáneos por núcleo virtual. Para otras configuraciones de memoria con núcleo virtual máximo por base de datos que sean un núcleo virtual o menos, la cantidad máxima de trabajos simultáneos se escala de forma similar.

## <a name="business-critical---provisioned-compute---m-series"></a>Crítico para la empresa: proceso aprovisionado: serie M

### <a name="m-series-compute-generation-part-1"></a>Generación de proceso de la serie M (parte 1)

|Tamaño de proceso (objetivo de servicio)|BC_M_8|BC_M_10|BC_M_12|BC_M_14|BC_M_16|BC_M_18|
|:---| ---:|---:|---:|---:|---:|---:|
|Generación de procesos|Serie M|Serie M|Serie M|Serie M|Serie M|Serie M|
|Núcleos virtuales|8|10|12|14|16|18|
|Memoria (GB)|235,4|294,3|353,2|412,0|470,9|529,7|
|Máximo número de bases de datos por grupo <sup>1</sup>|100|100|100|100|100|100|
|Compatible con almacén de columnas|Sí|Sí|Sí|Sí|Sí|Sí|
|Almacenamiento OLTP en memoria (GB)|64|80|96|112|128|150|
|Tamaño máximo de datos (GB)|512|640|768|896|1024|1152|
|Tamaño máximo de registro (GB)|171|213|256|299|341|384|
|Tamaño máximo de datos de TempDB (GB)|256|320|384|448|512|576|
|Tipo de almacenamiento|SSD local|SSD local|SSD local|SSD local|SSD local|SSD local|
|Latencia de E/S (aproximada)|1-2 ms (escritura)<br>1-2 ms (lectura)|1-2 ms (escritura)<br>1-2 ms (lectura)|1-2 ms (escritura)<br>1-2 ms (lectura)|1-2 ms (escritura)<br>1-2 ms (lectura)|1-2 ms (escritura)<br>1-2 ms (lectura)|1-2 ms (escritura)<br>1-2 ms (lectura)|
|Número máximo de IOPS de datos por grupo <sup>2</sup>|12 499|15 624|18 748|21 873|24 998|28 123|
|Velocidad de registro máxima por grupo (MBps)|48|60|72|84|96|108|
|Número máximo de trabajos simultáneos por grupo (solicitudes) <sup>3</sup>|800|1,000|1,200|1400|1600|1800|
|Número máximo de inicios de sesión simultáneos por grupo (solicitudes) <sup>3</sup>|800|1,000|1,200|1400|1600|1800|
|N.º máximo de sesiones simultáneas|30000|30000|30000|30000|30000|30000|
|Opciones de núcleo virtual mín./máx. de grupos elásticos por base de datos|0 a 8|0 a 10|0 a 12|0 a 14|0 a 16|0 a 18|
|Número de réplicas|4|4|4|4|4|4|
|AZ múltiple|No|No|No|No|No|No|
|Escalado horizontal de lectura|Sí|Sí|Sí|Sí|Sí|Sí|
|Almacenamiento de copia de seguridad incluido|1X el tamaño de base de datos|1X el tamaño de base de datos|1X el tamaño de base de datos|1X el tamaño de base de datos|1X el tamaño de base de datos|1X el tamaño de base de datos|

<sup>1</sup> Consulte [Administración de recursos en grupos elásticos densos](elastic-pool-resource-management.md) para conocer las consideraciones adicionales.

<sup>2</sup> El valor máximo de los tamaños de E/S que oscilan entre 8 KB y 64 KB. Las IOPS reales dependen de la carga de trabajo. Para más información, consulte [Regulación de E/S de los datos](resource-limits-logical-server.md#resource-governance).

<sup>3</sup> Para obtener el máximo de trabajos simultáneos (solicitudes) para cualquier base de datos individual, consulte [Límites de recursos de base de datos única](resource-limits-vcore-single-databases.md). Por ejemplo, si el grupo elástico usa Gen5 y el valor máximo de núcleo virtual por base de datos se establece en 2, el valor de máximo de trabajos simultáneos es 200.  Si el número máximo de núcleo virtual por base de datos se establece en 0,5, el valor de máximo de trabajos simultáneos es 50, ya que en Gen5 hay un máximo de 100 trabajos simultáneos por núcleo virtual. Para otras configuraciones de memoria con núcleo virtual máximo por base de datos que sean un núcleo virtual o menos, la cantidad máxima de trabajos simultáneos se escala de forma similar.

Si todos los núcleos virtuales de un grupo elástico están ocupados, cada una de las bases de datos del grupo recibe la misma cantidad de recursos de proceso para procesar las consultas. Azure SQL Database proporciona ecuanimidad de uso compartido de recursos entre bases de datos garantizando los mismos segmentos de tiempo de proceso. La ecuanimidad de uso compartido de recursos del grupo elástico es adicional a cualquier cantidad de recursos garantizados de otro modo a cada base de datos cuando el número mínimo de núcleos virtuales por base de datos se establece en un valor distinto de cero.



### <a name="m-series-compute-generation-part-2"></a>Generación de proceso de la serie M (parte 2)

|Tamaño de proceso (objetivo de servicio)|BC_M_20|BC_M_24|BC_M_32|BC_M_64|BC_M_128|
|:---| ---:|---:|---:|---:|---:|
|Generación de procesos|Serie M|Serie M|Serie M|Serie M|Serie M|
|Núcleos virtuales|20|24|32|64|128|
|Memoria (GB)|588,6|706,3|941,8|1883,5|3767,0|
|Máximo número de bases de datos por grupo <sup>1</sup>|100|100|100|100|100|100|
|Compatible con almacén de columnas|Sí|Sí|Sí|Sí|Sí|
|Almacenamiento OLTP en memoria (GB)|172|216|304|704|1768|
|Tamaño máximo de datos (GB)|1280|1536|2048|4096|4096|
|Tamaño máximo de registro (GB)|427|512|683|1024|1024|
|Tamaño máximo de datos de TempDB (GB)|4096|2048|1024|768|640|
|Tipo de almacenamiento|SSD local|SSD local|SSD local|SSD local|SSD local|
|Latencia de E/S (aproximada)|1-2 ms (escritura)<br>1-2 ms (lectura)|1-2 ms (escritura)<br>1-2 ms (lectura)|1-2 ms (escritura)<br>1-2 ms (lectura)|1-2 ms (escritura)<br>1-2 ms (lectura)|1-2 ms (escritura)<br>1-2 ms (lectura)|
|Número máximo de IOPS de datos por grupo <sup>2</sup>|31 248|37 497|49 996|99 993|160 000|
|Velocidad de registro máxima por grupo (MBps)|120|144|192|264|264|
|Número máximo de trabajos simultáneos por grupo (solicitudes) <sup>3</sup>|2\.000|2,400|3\.200|6\.400|12.800|
|Número máximo de inicios de sesión simultáneos por grupo (solicitudes) <sup>3</sup>|2\.000|2,400|3\.200|6\.400|12.800|
|N.º máximo de sesiones simultáneas|30000|30000|30000|30000|30000|
|Número de réplicas|4|4|4|4|4|
|AZ múltiple|No|No|No|No|No|
|Escalado horizontal de lectura|Sí|Sí|Sí|Sí|Sí|
|Almacenamiento de copia de seguridad incluido|1X el tamaño de base de datos|1X el tamaño de base de datos|1X el tamaño de base de datos|1X el tamaño de base de datos|1X el tamaño de base de datos|

<sup>1</sup> Consulte [Administración de recursos en grupos elásticos densos](elastic-pool-resource-management.md) para conocer las consideraciones adicionales.

<sup>2</sup> El valor máximo de los tamaños de E/S que oscilan entre 8 KB y 64 KB. Las IOPS reales dependen de la carga de trabajo. Para más información, consulte [Regulación de E/S de los datos](resource-limits-logical-server.md#resource-governance).

<sup>3</sup> Para obtener el máximo de trabajos simultáneos (solicitudes) para cualquier base de datos individual, consulte [Límites de recursos de base de datos única](resource-limits-vcore-single-databases.md). Por ejemplo, si el grupo elástico usa Gen5 y el valor máximo de núcleo virtual por base de datos se establece en 2, el valor de máximo de trabajos simultáneos es 200.  Si el número máximo de núcleo virtual por base de datos se establece en 0,5, el valor de máximo de trabajos simultáneos es 50, ya que en Gen5 hay un máximo de 100 trabajos simultáneos por núcleo virtual. Para otras configuraciones de memoria con núcleo virtual máximo por base de datos que sean un núcleo virtual o menos, la cantidad máxima de trabajos simultáneos se escala de forma similar.

Si todos los núcleos virtuales de un grupo elástico están ocupados, cada una de las bases de datos del grupo recibe la misma cantidad de recursos de proceso para procesar las consultas. Azure SQL Database proporciona ecuanimidad de uso compartido de recursos entre bases de datos garantizando los mismos segmentos de tiempo de proceso. La ecuanimidad de uso compartido de recursos del grupo elástico es adicional a cualquier cantidad de recursos garantizados de otro modo a cada base de datos cuando el número mínimo de núcleos virtuales por base de datos se establece en un valor distinto de cero.


## <a name="database-properties-for-pooled-databases"></a>Propiedades de base de datos para bases de datos agrupadas

En la tabla siguiente se describen las propiedades de las bases de datos agrupadas.

> [!NOTE]
> Los límites de recursos de las bases de datos individuales de los grupos elásticos suelen ser los mismos que los de las bases de datos únicas fuera de los grupos que tienen el mismo tamaño de proceso (objetivo de servicio). Por ejemplo, el número máximo de trabajos simultáneos en una base de datos GP_Gen4_1 es 200 trabajos. Por lo tanto, el número máximo de trabajos simultáneos en una base de datos de un grupo GP_Gen4_1 también es 200 trabajos. Tenga en cuenta que el número total de trabajos simultáneos en el grupo GP_Gen4_1 es 210.

| Propiedad | Descripción |
|:--- |:--- |
| Núcleos virtuales máximos por base de datos |El número máximo de núcleos virtuales que puede usar cualquier base de datos del grupo, si está disponible según el uso que hacen otras bases de datos del grupo. El número de núcleos virtuales por base de datos no garantiza la disponibilidad de recursos para una base de datos. Se trata de una configuración global que se aplica a todas las bases de datos del grupo. Establezca un número de núcleos virtuales por base de datos lo suficientemente alto como para gestionar los picos de uso de la base de datos. Se admite cierto grado de exceso de asignación de recursos, ya que el grupo suele basarse en patrones de uso en frío y caliente de las bases de datos, cuando en realidad los picos de uso no tienen lugar en todas las bases de datos a la vez.|
| Número mínimo de núcleos virtuales por base de datos |El número mínimo de núcleos virtuales que se garantiza en cualquier base de datos del grupo. Se trata de una configuración global que se aplica a todas las bases de datos del grupo. El número mínimo de núcleos virtuales por base de datos se puede establecer en 0, y también se trata del valor predeterminado. Esta propiedad se establece en cualquier valor entre 0 y el uso medio de núcleos virtuales por base de datos. El resultado de multiplicar el número de bases de datos del grupo y el número mínimo de núcleos virtuales por base de datos no puede superar el número de núcleos virtuales por grupo.|
| Almacenamiento máximo por base de datos |El tamaño máximo de base de datos establecido por el usuario para una base de datos de un grupo. Las bases de datos agrupadas comparten el almacenamiento del grupo asignado, de modo que el tamaño que puede alcanzar una base de datos se limita a la menor cantidad de almacenamiento restante del grupo y el tamaño de la base de datos. El tamaño máximo de la base de datos hace referencia al tamaño máximo de los archivos de datos, y no incluye el espacio utilizado por los archivos de registro. |
|||

## <a name="next-steps"></a>Pasos siguientes

- Para conocer los límites de recursos de los núcleos virtuales de una base de datos única, consulte los [límites de recursos para bases de datos únicas con el modelo de compra del núcleo virtual](resource-limits-vcore-single-databases.md).
- Para conocer los límites de recursos de DTU para una base de datos única, consulte los [límites de recursos para bases de datos únicas que usan el modelo de compra de DTU](resource-limits-dtu-single-databases.md).
- Para conocer los límites de recursos de DTU para grupos elásticos, consulte los [límites de recursos para grupos elásticos que usan el modelo de compra de DTU](resource-limits-dtu-elastic-pools.md).
- Para conocer los límites de recursos para instancias administradas, consulte los [límites de recursos para instancias administradas](../managed-instance/resource-limits.md).
- Para más información sobre los límites generales de Azure, consulte [Límites, cuotas y restricciones de suscripción y servicios de Microsoft Azure](../../azure-resource-manager/management/azure-subscription-service-limits.md).
- Para más información sobre los límites de recursos en un servidor SQL lógico, vea la [información general sobre los límites de recursos en un servidor SQL lógico](resource-limits-logical-server.md), donde encontrará datos sobre los límites en los niveles de servidor y suscripción.
