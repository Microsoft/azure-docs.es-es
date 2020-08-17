---
title: Límites de recursos de la unidad de procesamiento de base de datos en grupos elásticos
description: En esta página se describen algunos límites de recursos de DTU comunes para grupos elásticos en Azure SQL Database.
services: sql-database
ms.service: sql-database
ms.subservice: elastic-pools
ms.custom: seo-lt-2019 sqldbrb=1 references_regions
ms.devlang: ''
ms.topic: conceptual
author: sachinpMSFT
ms.author: sachinp
ms.reviewer: carlrab
ms.date: 07/28/2020
ms.openlocfilehash: f3c7420e1f33a7c25e7d3bfdffff14019d0b606a
ms.sourcegitcommit: 8def3249f2c216d7b9d96b154eb096640221b6b9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/03/2020
ms.locfileid: "87542677"
---
# <a name="resources-limits-for-elastic-pools-using-the-dtu-purchasing-model"></a>Límites de recursos para grupos elásticos que utilizan el modelo de compra de DTU
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

En este artículo se proporcionan los límites de recursos detallados para bases de datos en Azure SQL Database que están dentro de un grupo elástico mediante el modelo de compra de DTU.

* Para conocer los límites de recursos del modelo de compra de DTU para Azure SQL Database, consulte [Límites de recursos de DTU: Azure SQL Database](resource-limits-vcore-elastic-pools.md).
* Para conocer los límites de recursos de núcleo virtual, consulte [Límites de recursos del núcleo virtual: Azure SQL Database](resource-limits-vcore-single-databases.md) y [Límites de recursos del núcleo virtual: grupos elásticos](resource-limits-vcore-elastic-pools.md).

## <a name="elastic-pool-storage-sizes-and-compute-sizes"></a>Grupo elástico: tamaños de almacenamiento y tamaños de proceso

Para los grupos elásticos de Azure SQL Database, las siguientes tablas muestran los recursos disponibles en cada nivel de servicio y tamaño de proceso. Puede establecer el nivel de servicio, el tamaño de proceso y la cantidad de almacenamiento mediante:

* [Azure Portal](elastic-pool-manage.md#azure-portal)
* [PowerShell](elastic-pool-manage.md#powershell)
* [CLI de Azure](elastic-pool-manage.md#azure-cli)
* [API DE REST](elastic-pool-manage.md#rest-api).

> [!IMPORTANT]
> Para obtener información sobre la guía y otras consideraciones del escalado, consulte [Escalar un grupo elástico](elastic-pool-scale.md).

Los límites de recursos de las bases de datos individuales de los grupos elásticos suelen ser los mismos que para las bases de datos únicas fuera de los grupos basados en DTU y el nivel de servicio. Por ejemplo, el número máximo de trabajadores simultáneos de una base de datos S2 es de 120 trabajadores. Por lo tanto, el número máximo de trabajadores simultáneos de una base de datos de un grupo estándar también es de 120 trabajadores si el número máximo de DTU por base de datos del grupo es de 50 DTU (que es equivalente a S2).
 
Para el mismo número de DTU, los recursos proporcionados a un grupo elástico pueden superar los recursos proporcionados a una única base de datos fuera de un grupo elástico. Esto significa que es posible que el uso de eDTU de un grupo elástico sea inferior a la suma del uso de DTU entre las bases de datos del grupo, en función de los patrones de carga de trabajo. Por ejemplo, en un caso extremo en el que solo hay una base de datos en un grupo elástico donde el uso de DTU de la base de datos es del 100 %, es posible que el uso de eDTU del grupo sea del 50 % para determinados patrones de carga de trabajo. Esto puede ocurrir incluso si la DTU máxima por base de datos permanece en el valor máximo admitido para el tamaño de grupo especificado.

> [!NOTE]
> El límite de recursos de almacenamiento por grupo de cada una de las tablas siguientes no incluye tempdb ni el almacenamiento de registros.

### <a name="basic-elastic-pool-limits"></a>Límites de grupo elástico básico

| eDTU por grupo | **50** | **100** | **200** | **300** | **400** | **800** | **1200** | **1600** |
|:---|---:|---:|---:| ---: | ---: | ---: | ---: | ---: |
| Almacenamiento incluido por grupo (GB) | 5 | 10 | 20 | 29 | 39 | 78 | 117 | 156 |
| Almacenamiento máximo por grupo (GB) | 5 | 10 | 20 | 29 | 39 | 78 | 117 | 156 |
| Almacenamiento máximo de OLTP en memoria por grupo (GB) | N/D | N/D | N/D | N/D | N/D | N/D | N/D | N/D |
| Máximo número de bases de datos por grupo <sup>1</sup> | 100 | 200 | 500 | 500 | 500 | 500 | 500 | 500 |
| Cantidad máxima de trabajos (solicitudes) simultáneos por grupo <sup>2</sup> | 100 | 200 | 400 | 600 | 800 | 1600 | 2400 | 3200 |
| Cantidad máxima de sesiones simultáneas por grupo <sup>2</sup> | 30000 | 30000 | 30000 | 30000 |30000 | 30000 | 30000 | 30000 |
| Opciones de DTU mínima por base de datos | 0, 5 | 0, 5 | 0, 5 | 0, 5 | 0, 5 | 0, 5 | 0, 5 | 0, 5 |
| Opciones de DTU máxima por base de datos | 5 | 5 | 5 | 5 | 5 | 5 | 5 | 5 |
| Almacenamiento máximo por base de datos (GB) | 2 | 2 | 2 | 2 | 2 | 2 | 2 | 2 |
||||||||

<sup>1</sup> Consulte [Administración de recursos en grupos elásticos densos](elastic-pool-resource-management.md) para conocer las consideraciones adicionales.

<sup>2</sup> Para obtener el máximo de trabajos simultáneos (solicitudes) para cualquier base de datos individual, consulte [Límites de recursos de base de datos única](resource-limits-vcore-single-databases.md). Por ejemplo, si el grupo elástico usa Gen5 y el valor máximo de núcleo virtual por base de datos se establece en 2, el valor de máximo de trabajos simultáneos es 200.  Si el número máximo de núcleo virtual por base de datos se establece en 0,5, el valor de máximo de trabajos simultáneos es 50, ya que en Gen5 hay un máximo de 100 trabajos simultáneos por núcleo virtual. Para otras configuraciones de memoria con núcleo virtual máximo por base de datos que sean un núcleo virtual o menos, la cantidad máxima de trabajos simultáneos se escala de forma similar.

### <a name="standard-elastic-pool-limits"></a>Límites de grupo elástico estándar

| eDTU por grupo | **50** | **100** | **200** | **300** | **400** | **800**|
|:---|---:|---:|---:| ---: | ---: | ---: |
| Almacenamiento incluido por grupo (GB) <sup>1</sup> | 50 | 100 | 200 | 300 | 400 | 800 |
| Almacenamiento máximo por grupo (GB) | 500 | 750 | 1024 | 1280 | 1536 | 2048 |
| Almacenamiento máximo de OLTP en memoria por grupo (GB) | N/D | N/D | N/D | N/D | N/D | N/D |
| Número máximo de bases de datos por grupo <sup>2</sup> | 100 | 200 | 500 | 500 | 500 | 500 |
| Cantidad máxima de trabajos (solicitudes) simultáneos por grupo <sup>3</sup> | 100 | 200 | 400 | 600 | 800 | 1600 |
| Cantidad máxima de sesiones simultáneas por grupo <sup>3</sup> | 30000 | 30000 | 30000 | 30000 | 30000 | 30000 |
| Opciones de DTU mínima por base de datos | 0, 10, 20, 50 | 0, 10, 20, 50, 100 | 0, 10, 20, 50, 100, 200 | 0, 10, 20, 50, 100, 200, 300 | 0, 10, 20, 50, 100, 200, 300, 400 | 0, 10, 20, 50, 100, 200, 300, 400, 800 |
| Opciones de DTU máxima por base de datos | 10, 20, 50 | 10, 20, 50, 100 | 10, 20, 50, 100, 200 | 10, 20, 50, 100, 200, 300 | 10, 20, 50, 100, 200, 300, 400 | 10, 20, 50, 100, 200, 300, 400, 800 |
| Almacenamiento máximo por base de datos (GB) | 500 | 750 | 1024 | 1024 | 1024 | 1024 |
||||||||

<sup>1</sup> Consulte [Precios de Azure SQL Database](https://azure.microsoft.com/pricing/details/sql-database/elastic/) para más información sobre los costos extra que se generan al aprovisionar almacenamiento adicional.

<sup>2</sup> Consulte [Administración de recursos en grupos elásticos densos](elastic-pool-resource-management.md) para ver otras consideraciones.

<sup>3</sup> Para obtener el máximo de trabajos simultáneos (solicitudes) para cualquier base de datos individual, consulte [Límites de recursos de base de datos única](resource-limits-vcore-single-databases.md). Por ejemplo, si el grupo elástico usa Gen5 y el valor máximo de núcleo virtual por base de datos se establece en 2, el valor de máximo de trabajos simultáneos es 200.  Si el número máximo de núcleo virtual por base de datos se establece en 0,5, el valor de máximo de trabajos simultáneos es 50, ya que en Gen5 hay un máximo de 100 trabajos simultáneos por núcleo virtual. Para otras configuraciones de memoria con núcleo virtual máximo por base de datos que sean un núcleo virtual o menos, la cantidad máxima de trabajos simultáneos se escala de forma similar.

### <a name="standard-elastic-pool-limits-continued"></a>Límites de grupo elástico estándar (continuación)

| eDTU por grupo | **1200** | **1600** | **2000** | **2500** | **3000** |
|:---|---:|---:|---:| ---: | ---: |
| Almacenamiento incluido por grupo (GB) <sup>1</sup> | 1200 | 1600 | 2000 | 2500 | 3000 |
| Almacenamiento máximo por grupo (GB) | 2560 | 3072 | 3584 | 4096 | 4096 |
| Almacenamiento máximo de OLTP en memoria por grupo (GB) | N/D | N/D | N/D | N/D | N/D |
| Número máximo de bases de datos por grupo <sup>2</sup> | 500 | 500 | 500 | 500 | 500 |
| Cantidad máxima de trabajos (solicitudes) simultáneos por grupo <sup>3</sup> | 2400 | 3200 | 4000 | 5000 | 6000 |
| Cantidad máxima de sesiones simultáneas por grupo <sup>3</sup> | 30000 | 30000 | 30000 | 30000 | 30000 |
| Opciones de DTU mínima por base de datos | 0, 10, 20, 50, 100, 200, 300, 400, 800, 1200 | 0, 10, 20, 50, 100, 200, 300, 400, 800, 1200, 1600 | 0, 10, 20, 50, 100, 200, 300, 400, 800, 1200, 1600, 2000 | 0, 10, 20, 50, 100, 200, 300, 400, 800, 1200, 1600, 2000, 2500 | 0, 10, 20, 50, 100, 200, 300, 400, 800, 1200, 1600, 2000, 2500, 3000 |
| Opciones de DTU máxima por base de datos | 10, 20, 50, 100, 200, 300, 400, 800, 1200 | 10, 20, 50, 100, 200, 300, 400, 800, 1200, 1600 | 10, 20, 50, 100, 200, 300, 400, 800, 1200, 1600, 2000 | 10, 20, 50, 100, 200, 300, 400, 800, 1200, 1600, 2000, 2500 | 10, 20, 50, 100, 200, 300, 400, 800, 1200, 1600, 2000, 2500, 3000 |
| Almacenamiento máximo por base de datos (GB) | 1024 | 1024 | 1024 | 1024 | 1024 |
|||||||

<sup>1</sup> Consulte [Precios de Azure SQL Database](https://azure.microsoft.com/pricing/details/sql-database/elastic/) para más información sobre los costos extra que se generan al aprovisionar almacenamiento adicional.

<sup>2</sup> Consulte [Administración de recursos en grupos elásticos densos](elastic-pool-resource-management.md) para ver otras consideraciones.

<sup>3</sup> Para obtener el máximo de trabajos simultáneos (solicitudes) para cualquier base de datos individual, consulte [Límites de recursos de base de datos única](resource-limits-vcore-single-databases.md). Por ejemplo, si el grupo elástico usa Gen5 y el valor máximo de núcleo virtual por base de datos se establece en 2, el valor de máximo de trabajos simultáneos es 200.  Si el número máximo de núcleo virtual por base de datos se establece en 0,5, el valor de máximo de trabajos simultáneos es 50, ya que en Gen5 hay un máximo de 100 trabajos simultáneos por núcleo virtual. Para otras configuraciones de memoria con núcleo virtual máximo por base de datos que sean un núcleo virtual o menos, la cantidad máxima de trabajos simultáneos se escala de forma similar.

### <a name="premium-elastic-pool-limits"></a>Límites de grupo elástico premium

| eDTU por grupo | **125** | **250** | **500** | **1000** | **1500**|
|:---|---:|---:|---:| ---: | ---: |
| Almacenamiento incluido por grupo (GB) <sup>1</sup> | 250 | 500 | 750 | 1024 | 1536 |
| Almacenamiento máximo por grupo (GB) | 1024 | 1024 | 1024 | 1024 | 1536 |
| Almacenamiento máximo de OLTP en memoria por grupo (GB) | 1 | 2 | 4 | 10 | 12 |
| Número máximo de bases de datos por grupo <sup>2</sup> | 50 | 100 | 100 | 100 | 100 |
| Número máximo de trabajos simultáneos por grupo (solicitudes) <sup>3</sup> | 200 | 400 | 800 | 1600 | 2400 |
| Cantidad máxima de sesiones simultáneas por grupo <sup>3</sup> | 30000 | 30000 | 30000 | 30000 | 30000 |
| Cantidad mínima de eDTU por base de datos | 0, 25, 50, 75, 125 | 0, 25, 50, 75, 125, 250 | 0, 25, 50, 75, 125, 250, 500 | 0, 25, 50, 75, 125, 250, 500, 1000 | 0, 25, 50, 75, 125, 250, 500, 1000|
| Cantidad máxima de eDTU por base de datos | 25, 50, 75, 125 | 25, 50, 75, 125, 250 | 25, 50, 75, 125, 250, 500 | 25, 50, 75, 125, 250, 500, 1000 | 25, 50, 75, 125, 250, 500, 1000|
| Almacenamiento máximo por base de datos (GB) | 1024 | 1024 | 1024 | 1024 | 1024 |
|||||||

<sup>1</sup> Consulte [Precios de Azure SQL Database](https://azure.microsoft.com/pricing/details/sql-database/elastic/) para más información sobre los costos extra que se generan al aprovisionar almacenamiento adicional.

<sup>2</sup> Consulte [Administración de recursos en grupos elásticos densos](elastic-pool-resource-management.md) para ver otras consideraciones.

<sup>3</sup> Para obtener el máximo de trabajos simultáneos (solicitudes) para cualquier base de datos individual, consulte [Límites de recursos de base de datos única](resource-limits-vcore-single-databases.md). Por ejemplo, si el grupo elástico usa Gen5 y el valor máximo de núcleo virtual por base de datos se establece en 2, el valor de máximo de trabajos simultáneos es 200.  Si el número máximo de núcleo virtual por base de datos se establece en 0,5, el valor de máximo de trabajos simultáneos es 50, ya que en Gen5 hay un máximo de 100 trabajos simultáneos por núcleo virtual. Para otras configuraciones de memoria con núcleo virtual máximo por base de datos que sean un núcleo virtual o menos, la cantidad máxima de trabajos simultáneos se escala de forma similar.

### <a name="premium-elastic-pool-limits-continued"></a>Límites de grupo elástico premium (continuación)

| eDTU por grupo | **2000** | **2500** | **3000** | **3500** | **4000**|
|:---|---:|---:|---:| ---: | ---: |
| Almacenamiento incluido por grupo (GB) <sup>1</sup> | 2048 | 2560 | 3072 | 3548 | 4096 |
| Almacenamiento máximo por grupo (GB) | 2048 | 2560 | 3072 | 3548 | 4096|
| Almacenamiento máximo de OLTP en memoria por grupo (GB) | 16 | 20 | 24 | 28 | 32 |
| Número máximo de bases de datos por grupo <sup>2</sup> | 100 | 100 | 100 | 100 | 100 |
| Cantidad máxima de trabajos (solicitudes) simultáneos por grupo <sup>3</sup> | 3200 | 4000 | 4800 | 5600 | 6400 |
| Cantidad máxima de sesiones simultáneas por grupo <sup>3</sup> | 30000 | 30000 | 30000 | 30000 | 30000 |
| Opciones de DTU mínima por base de datos | 0, 25, 50, 75, 125, 250, 500, 1000, 1750 | 0, 25, 50, 75, 125, 250, 500, 1000, 1750 | 0, 25, 50, 75, 125, 250, 500, 1000, 1750 | 0, 25, 50, 75, 125, 250, 500, 1000, 1750 | 0, 25, 50, 75, 125, 250, 500, 1000, 1750, 4000 |
| Opciones de DTU máxima por base de datos | 25, 50, 75, 125, 250, 500, 1000, 1750 | 25, 50, 75, 125, 250, 500, 1000, 1750 | 25, 50, 75, 125, 250, 500, 1000, 1750 | 25, 50, 75, 125, 250, 500, 1000, 1750 | 25, 50, 75, 125, 250, 500, 1000, 1750, 4000 |
| Almacenamiento máximo por base de datos (GB) | 1024 | 1024 | 1024 | 1024 | 1024 |
|||||||

<sup>1</sup> Consulte [Precios de Azure SQL Database](https://azure.microsoft.com/pricing/details/sql-database/elastic/) para más información sobre los costos extra que se generan al aprovisionar almacenamiento adicional.

<sup>2</sup> Consulte [Administración de recursos en grupos elásticos densos](elastic-pool-resource-management.md) para ver otras consideraciones.

<sup>3</sup> Para obtener el máximo de trabajos simultáneos (solicitudes) para cualquier base de datos individual, consulte [Límites de recursos de base de datos única](resource-limits-vcore-single-databases.md). Por ejemplo, si el grupo elástico usa Gen5 y el valor máximo de núcleo virtual por base de datos se establece en 2, el valor de máximo de trabajos simultáneos es 200.  Si el número máximo de núcleo virtual por base de datos se establece en 0,5, el valor de máximo de trabajos simultáneos es 50, ya que en Gen5 hay un máximo de 100 trabajos simultáneos por núcleo virtual. Para otras configuraciones de memoria con núcleo virtual máximo por base de datos que sean un núcleo virtual o menos, la cantidad máxima de trabajos simultáneos se escala de forma similar.

> [!IMPORTANT]
> Existe más de 1 TB de almacenamiento en el nivel Premium actualmente disponible en todas las regiones excepto: Este de China, Norte de China, Centro de Alemania y Noreste de Alemania. En estas regiones, el almacenamiento máximo en el nivel Prémium está limitado a 1 TB.  Para más información, consulte las [limitaciones actuales de P11 y P15](single-database-scale.md#p11-and-p15-constraints-when-max-size-greater-than-1-tb).

Si se usan todas las unidades DTU de un grupo elástico, cada una de las bases de datos del grupo recibe una misma cantidad de recursos para procesar consultas. El servicio SQL Database proporciona ecuanimidad de uso compartido de recursos entre bases de datos garantizando los mismos segmentos de tiempo de proceso. La ecuanimidad de uso compartido de recursos del grupo elástico es adicional a cualquier cantidad de recursos garantizados de otro modo a cada base de datos cuando el número mínimo de DTU por base de datos se establece en un valor distinto de cero.

> [!NOTE]
> Para obtener información sobre los límites de `tempdb`, consulte los [límites de tempdb](https://docs.microsoft.com/sql/relational-databases/databases/tempdb-database?view=sql-server-2017#tempdb-database-in-sql-database).

### <a name="database-properties-for-pooled-databases"></a>Propiedades de base de datos para bases de datos agrupadas

En la tabla siguiente se describen las propiedades de las bases de datos agrupadas.

| Propiedad | Descripción |
|:--- |:--- |
| Cantidad máxima de eDTU por base de datos |Cantidad máxima de eDTU que puede utilizar cualquier base de datos del grupo, si está disponible según el uso que hacen otras bases de datos del grupo. La cantidad máxima de eDTU por base de datos no garantiza la disponibilidad de recursos. Se trata de una configuración global que se aplica a todas las bases de datos del grupo. Establezca una cantidad máxima de eDTU por base de datos lo suficientemente alta como para gestionar los picos de uso de la base de datos. Se admite cierto grado de exceso de asignación de recursos, ya que el grupo suele basarse en patrones de uso en frío y caliente de las bases de datos, cuando en realidad los picos de uso no tienen lugar en todas las bases de datos a la vez. Por ejemplo, supongamos que el pico de uso de cada base de datos es 20 eDTU y solo tiene lugar en el 20 % de las 100 bases de datos del grupo a la vez. Si la cantidad máxima de eDTU por base de datos se establece en 20, puede asignar al grupo una cantidad 5 veces mayor y establecer las eDTU por grupo en 400. |
| Cantidad mínima de eDTU por base de datos |Cantidad mínima de eDTU que se garantiza en cualquier base de datos del grupo. Se trata de una configuración global que se aplica a todas las bases de datos del grupo. La cantidad mínima de eDTU por base de datos se puede establecer en 0, y también se trata del valor predeterminado. Esta propiedad se establece en cualquier valor entre 0 y el uso medio de eDTU por base de datos. El resultado de multiplicar la cantidad de bases de datos del grupo y la cantidad mínima de eDTU por base de datos no puede superar el número de eDTU por grupo. Por ejemplo, si un grupo tiene 20 bases de datos y la cantidad mínima de eDTU por base de datos establecida es 10, el número de eDTU por grupo debe ser de, al menos, 200. |
| Almacenamiento máximo por base de datos |El tamaño máximo de base de datos establecido por el usuario para una base de datos de un grupo. Sin embargo, las bases de datos agrupadas comparten almacenamiento en grupo asignado. Aunque el almacenamiento total máximo *por base de datos* se establezca en un valor mayor que el *espacio del grupo* de almacenamiento total disponible, el espacio total que realmente usan todas las bases de datos no podrá superar el límite de grupo disponible. El tamaño máximo de la base de datos hace referencia al tamaño máximo de los archivos de datos, y no incluye el espacio utilizado por los archivos de registro. |
|||

## <a name="next-steps"></a>Pasos siguientes

* Para conocer los límites de recursos de los núcleos virtuales de una base de datos única, consulte los [límites de recursos para bases de datos únicas con el modelo de compra del núcleo virtual](resource-limits-vcore-single-databases.md).
* Para conocer los límites de recursos de DTU para una base de datos única, consulte los [límites de recursos para bases de datos únicas que usan el modelo de compra de DTU](resource-limits-dtu-single-databases.md).
* Para conocer los límites de recursos de núcleos virtuales para grupos elásticos, consulte los [límites de recursos para grupos elásticos con el modelo de compra del núcleo virtual](resource-limits-vcore-elastic-pools.md).
* Para conocer los límites de recursos para instancias administradas en Instancia administrada de Azure SQL, consulte los [límites de recursos para Instancia administrada de SQL](../managed-instance/resource-limits.md).
* Para más información sobre los límites generales de Azure, consulte [Límites, cuotas y restricciones de suscripción y servicios de Microsoft Azure](../../azure-resource-manager/management/azure-subscription-service-limits.md).
* Para más información sobre los límites de recursos en un servidor SQL lógico, vea la [información general sobre los límites de recursos en un servidor SQL lógico](resource-limits-logical-server.md), donde encontrará datos sobre los límites en los niveles de servidor y suscripción.
