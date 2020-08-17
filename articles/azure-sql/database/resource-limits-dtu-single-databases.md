---
title: Límites de recursos de la unidad de procesamiento de base de datos en bases de datos únicas
description: En esta página se describen algunos límites de recursos de DTU comunes para bases de datos únicas en Azure SQL Database.
services: sql-database
ms.service: sql-database
ms.subservice: single-database
ms.custom: references_regions, seo-lt-2019, sqldbrb=1
ms.devlang: ''
ms.topic: conceptual
author: stevestein
ms.author: sstein
ms.reviewer: ''
ms.date: 03/20/2019
ms.openlocfilehash: fa15a5f95c2163309ecfd9cc0130d1860eb3b634
ms.sourcegitcommit: 8def3249f2c216d7b9d96b154eb096640221b6b9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/03/2020
ms.locfileid: "87542694"
---
# <a name="resource-limits-for-single-databases-using-the-dtu-purchasing-model---azure-sql-database"></a>Límites de recursos de bases de datos únicas que usan el modelo de compra de DTU: Azure SQL Database
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

En este artículo se proporcionan los límites de recursos detallados para bases de datos únicas de Azure SQL Database que usan el modelo de compra de DTU.

Para saber más sobre los límites de recursos del modelo de compra de DTU para grupos elásticos, consulte [Límites de recursos de DTU: grupos elásticos](resource-limits-dtu-elastic-pools.md). En cuanto a los límites de recursos del núcleo virtual, consulte [Límites de recursos del núcleo virtual: bases de datos únicas](resource-limits-vcore-single-databases.md) y [Límites de recursos del núcleo virtual: grupos elásticos](resource-limits-vcore-elastic-pools.md). Para obtener más información sobre los diferentes modelos de compra, consulte los [niveles de servicio y modelos de compra](purchasing-models.md).

## <a name="single-database-storage-sizes-and-compute-sizes"></a>Base de datos única: tamaños de almacenamiento y tamaños de proceso

Las siguientes tablas muestran los recursos disponibles para una base de datos única en cada nivel de servicio y tamaño de proceso. Puede establecer el nivel de servicio, el tamaño de proceso y la cantidad de almacenamiento para una base de datos única mediante [Azure Portal](single-database-manage.md#the-azure-portal), [Transact-SQL](single-database-manage.md#transact-sql-t-sql), [PowerShell](single-database-manage.md#powershell), la [CLI de Azure](single-database-manage.md#the-azure-cli) o la [API REST](single-database-manage.md#rest-api).

> [!IMPORTANT]
> Para información y consideraciones sobre el escalado, consulte [Escalado de una base de datos única](single-database-scale.md).

### <a name="basic-service-tier"></a>Nivel de servicio Básico

| **Tamaño de proceso** | **Basic** |
| :--- | --: |
| DTU máx. | 5 |
| Almacenamiento incluido (GB) | 2 |
| Almacenamiento máximo (GB) | 2 |
| Almacenamiento máximo de OLTP en memoria (GB) |N/D |
| Cantidad máxima de trabajos (solicitudes) simultáneos | 30 |
| N.º máximo de sesiones simultáneas | 300 |
|||

> [!IMPORTANT]
> El nivel de servicio Básico proporciona menos de una núcleo virtual (CPU).  En el caso de las cargas de trabajo con un uso intensivo de CPU, se recomienda un nivel de servicio S3 o superior.
>
>En lo que respecta al almacenamiento de datos, el nivel de servicio Básico se coloca en blobs en páginas estándar. Los blobs en páginas estándar usan medios de almacenamiento basados en discos duros (HDD) y son más adecuados para el desarrollo, las pruebas y otras cargas de trabajo de acceso poco frecuente que no dan tanta importancia a la variabilidad del rendimiento.
>

### <a name="standard-service-tier"></a>Nivel de servicio Estándar

| **Tamaño de proceso** | **S0** | **S1** | **S2** | **S3** |
| :--- |---:| ---:|---:|---:|
| DTU máx. | 10 | 20 | 50 | 100 |
| Almacenamiento incluido (GB) <sup>1</sup> | 250 | 250 | 250 | 250 |
| Almacenamiento máximo (GB) | 250 | 250 | 250 | 1024 |
| Almacenamiento máximo de OLTP en memoria (GB) | N/D | N/D | N/D | N/D |
| Cantidad máxima de trabajos (solicitudes) simultáneos| 60 | 90 | 120 | 200 |
| N.º máximo de sesiones simultáneas |600 | 900 | 1200 | 2400 |
||||||

<sup>1</sup> Consulte [Precios de Azure SQL Database](https://azure.microsoft.com/pricing/details/sql-database/single/) para más información sobre los costos extra que se generan al aprovisionar almacenamiento adicional.

> [!IMPORTANT]
> Los niveles de servicio Estándar S0, S1 y S2 proporcionan menos de una núcleo virtual (CPU).  En el caso de las cargas de trabajo con un uso intensivo de CPU, se recomienda un nivel de servicio S3 o superior.
>
>En lo que respecta al almacenamiento de datos, los niveles de servicio Estándar S0 y S1 se colocan en blobs en páginas estándar. Los blobs en páginas estándar usan medios de almacenamiento basados en discos duros (HDD) y son más adecuados para el desarrollo, las pruebas y otras cargas de trabajo de acceso poco frecuente que no dan tanta importancia a la variabilidad del rendimiento.
>

### <a name="standard-service-tier-continued"></a>Nivel de servicio Estándar (continuación)

| **Tamaño de proceso** | **S4** | **S6** | **S7** | **S9** | **S12** |
| :--- |---:| ---:|---:|---:|---:|
| DTU máx. | 200 | 400 | 800 | 1600 | 3000 |
| Almacenamiento incluido (GB) <sup>1</sup> | 250 | 250 | 250 | 250 | 250 |
| Almacenamiento máximo (GB) | 1024 | 1024 | 1024 | 1024 | 1024 |
| Almacenamiento máximo de OLTP en memoria (GB) | N/D | N/D | N/D | N/D |N/D |
| Cantidad máxima de trabajos (solicitudes) simultáneos| 400 | 800 | 1600 | 3200 |6000 |
| N.º máximo de sesiones simultáneas |4800 | 9600 | 19200 | 30000 |30000 |
|||||||

<sup>1</sup> Consulte [Precios de Azure SQL Database](https://azure.microsoft.com/pricing/details/sql-database/single/) para más información sobre los costos extra que se generan al aprovisionar almacenamiento adicional.

### <a name="premium-service-tier"></a>Nivel de servicio Premium

| **Tamaño de proceso** | **P1** | **P2** | **P4** | **P6** | **P11** | **P15** |
| :--- |---:|---:|---:|---:|---:|---:|
| DTU máx. | 125 | 250 | 500 | 1000 | 1750 | 4000 |
| Almacenamiento incluido (GB) <sup>1</sup> | 500 | 500 | 500 | 500 | 4096 <sup>2</sup> | 4096 <sup>2</sup> |
| Almacenamiento máximo (GB) | 1024 | 1024 | 1024 | 1024 | 4096 <sup>2</sup> | 4096 <sup>2</sup> |
| Almacenamiento máximo de OLTP en memoria (GB) | 1 | 2 | 4 | 8 | 14 | 32 |
| Cantidad máxima de trabajos (solicitudes) simultáneos| 200 | 400 | 800 | 1600 | 2800 | 6400 |
| N.º máximo de sesiones simultáneas | 30000 | 30000 | 30000 | 30000 | 30000 | 30000 |
|||||||

<sup>1</sup> Consulte [Precios de Azure SQL Database](https://azure.microsoft.com/pricing/details/sql-database/single/) para más información sobre los costos extra que se generan al aprovisionar almacenamiento adicional.

<sup>2</sup> Desde 1024 GB hasta 4096 GB en incrementos de 256 GB.

> [!IMPORTANT]
> Existe más de 1 TB de almacenamiento en el nivel Premium actualmente disponible en todas las regiones excepto: Este de China, Norte de China, Centro de Alemania y Nordeste de Alemania. En estas regiones, el almacenamiento máximo en el nivel Prémium está limitado a 1 TB.  Para más información, consulte las [limitaciones actuales de P11 y P15](single-database-scale.md#p11-and-p15-constraints-when-max-size-greater-than-1-tb).
> [!NOTE]
> Para obtener información sobre los límites de `tempdb`, consulte los [límites de tempdb](https://docs.microsoft.com/sql/relational-databases/databases/tempdb-database?view=sql-server-2017#tempdb-database-in-sql-database).

## <a name="next-steps"></a>Pasos siguientes

- Para conocer los límites de recursos de los núcleos virtuales de una base de datos única, consulte los [límites de recursos para bases de datos únicas con el modelo de compra del núcleo virtual](resource-limits-vcore-single-databases.md).
- Para conocer los límites de recursos de núcleos virtuales para grupos elásticos, consulte los [límites de recursos para grupos elásticos con el modelo de compra del núcleo virtual](resource-limits-vcore-elastic-pools.md).
- Para conocer los límites de recursos de DTU para grupos elásticos, consulte los [límites de recursos para grupos elásticos que usan el modelo de compra de DTU](resource-limits-dtu-elastic-pools.md).
- Para conocer los límites de recursos de las instancias administradas de Instancia administrada de Azure SQL, vea [Límites de recursos para Instancia administrada de SQL](../managed-instance/resource-limits.md).
- Para más información sobre los límites generales de Azure, consulte [Límites, cuotas y restricciones de suscripción y servicios de Microsoft Azure](../../azure-resource-manager/management/azure-subscription-service-limits.md).
- Para más información sobre los límites de recursos en un servidor SQL lógico, vea la [información general sobre los límites de recursos en un servidor SQL lógico](resource-limits-logical-server.md), donde encontrará datos sobre los límites en los niveles de servidor y suscripción.
