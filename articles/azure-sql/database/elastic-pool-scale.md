---
title: Escalado de los recursos de un grupo elástico
description: En esta página se describe el proceso de escalado de recursos de grupos elásticos en Azure SQL Database.
services: sql-database
ms.service: sql-database
ms.subservice: elastic-pools
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: conceptual
author: oslake
ms.author: moslake
ms.reviewer: carlrab
ms.date: 7/31/2020
ms.openlocfilehash: d8055c89af8adcb88a2055e617e27c030e05d5ae
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87504388"
---
# <a name="scale-elastic-pool-resources-in-azure-sql-database"></a>Escalar recursos de grupos elásticos en Azure SQL Database
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

En este artículo se describe cómo escalar los recursos de proceso y almacenamiento disponibles para los grupos elásticos y las bases de datos agrupadas en Azure SQL Database.

## <a name="change-compute-resources-vcores-or-dtus"></a>Cambio de los recursos de proceso (núcleos virtuales o DTU)

Después de elegir inicialmente el número de núcleos virtuales o de eDTU, puede escalar o reducir un grupo elástico verticalmente de manera dinámica en función de la experiencia real mediante [Azure Portal](elastic-pool-manage.md#azure-portal), [PowerShell](/powershell/module/az.sql/Get-AzSqlElasticPool), la [CLI de Azure](/cli/azure/sql/elastic-pool#az-sql-elastic-pool-update) o la [API REST](https://docs.microsoft.com/rest/api/sql/elasticpools/update).

### <a name="impact-of-changing-service-tier-or-rescaling-compute-size"></a>Impacto de cambiar el nivel de servicio o la escala del tamaño de proceso

El cambio en el nivel de servicio o el tamaño de proceso de un grupo elástico sigue un patrón similar en cuanto a bases de datos únicas y principalmente implica que el servicio realice los pasos siguientes:

1. Creación de una instancia de proceso nueva para el grupo elástico  

    Se crea una instancia de proceso nueva para el grupo elástico con el tamaño de proceso y el nivel de servicio solicitados. Para algunas combinaciones de cambios en el nivel de servicio y el tamaño de proceso, se debe crear una réplica de cada base de datos en la nueva instancia de proceso que implique copiar los datos y pueda influir en gran medida en la latencia general. En cualquier caso, las bases de datos permanecen en línea durante este paso y las conexiones se continúan dirigiendo a las bases de datos en la instancia de proceso original.

2. Cambio en el enrutamiento de conexiones a la nueva instancia de proceso

    Se colocan las conexiones existentes en las bases de datos en la instancia de proceso original. En la nueva instancia de proceso las nuevas conexiones se establecen en las bases de datos. Para algunas combinaciones de cambios de nivel de servicio y de tamaño de proceso, los archivos de base de datos se desasocian y se vuelven a asociar durante la modificación.  No obstante, el modificador puede provocar una breve interrupción del servicio cuando las bases de datos no estén disponibles de forma general durante menos de 30 segundos y, a menudo, durante solo unos segundos. Si hay transacciones de larga duración que se ejecutan cuando se colocan las conexiones, la duración de este paso puede ser mayor con el fin de recuperar las transacciones anuladas. La [Recuperación de base de datos acelerada](../accelerated-database-recovery.md) puede reducir el impacto de la anulación de transacciones de larga duración.

> [!IMPORTANT]
> Durante los pasos del flujo de trabajo no se pierden datos.

### <a name="latency-of-changing-service-tier-or-rescaling-compute-size"></a>Latencia de cambiar el nivel de servicio o la escala del tamaño de proceso

La latencia estimada de cambiar el nivel de servicio o la escala del tamaño de proceso de una base de datos única o un grupo elástico se parametriza como sigue:

|Nivel de servicio|Base de datos única básica,</br>estándar (S0-S1)|Grupo elástico básico,</br>estándar (S2-S12), </br>base de datos única o grupo elástico de uso general|Base de datos única o grupo elástico Premium o Crítico para la empresa|Hiperescala
|:---|:---|:---|:---|:---|
|**Base de datos único básica,</br> estándar (S0-S1)**|&bull; &nbsp;Latencia de tiempo constante independientemente del espacio usado</br>&bull; &nbsp;Normalmente, menos de 5 minutos|&bull; &nbsp;Latencia proporcional al espacio usado en la base de datos debido a la copia de datos</br>&bull; &nbsp;Normalmente, menos de 1 minuto por GB de espacio usado|&bull; &nbsp;Latencia proporcional al espacio usado en la base de datos debido a la copia de datos</br>&bull; &nbsp;Normalmente, menos de 1 minuto por GB de espacio usado|&bull; &nbsp;Latencia proporcional al espacio usado en la base de datos debido a la copia de datos</br>&bull; &nbsp;Normalmente, menos de 1 minuto por GB de espacio usado|
|**Grupo elástico básico, </br>estándar (S2-S12) </br>base de datos única o grupo elástico de uso general**|&bull; &nbsp;Latencia proporcional al espacio usado en la base de datos debido a la copia de datos</br>&bull; &nbsp;Normalmente, menos de 1 minuto por GB de espacio usado|&bull; &nbsp;Latencia de tiempo constante independientemente del espacio usado</br>&bull; &nbsp;Normalmente, menos de 5 minutos|&bull; &nbsp;Latencia proporcional al espacio usado en la base de datos debido a la copia de datos</br>&bull; &nbsp;Normalmente, menos de 1 minuto por GB de espacio usado|&bull; &nbsp;Latencia proporcional al espacio usado en la base de datos debido a la copia de datos</br>&bull; &nbsp;Normalmente, menos de 1 minuto por GB de espacio usado|
|**Base de datos única o grupo elástico Premium o Crítico para la empresa**|&bull; &nbsp;Latencia proporcional al espacio usado en la base de datos debido a la copia de datos</br>&bull; &nbsp;Normalmente, menos de 1 minuto por GB de espacio usado|&bull; &nbsp;Latencia proporcional al espacio usado en la base de datos debido a la copia de datos</br>&bull; &nbsp;Normalmente, menos de 1 minuto por GB de espacio usado|&bull; &nbsp;Latencia proporcional al espacio usado en la base de datos debido a la copia de datos</br>&bull; &nbsp;Normalmente, menos de 1 minuto por GB de espacio usado|&bull; &nbsp;Latencia proporcional al espacio usado en la base de datos debido a la copia de datos</br>&bull; &nbsp;Normalmente, menos de 1 minuto por GB de espacio usado|
|**Hiperescala**|N/D|N/D|N/D|&bull; &nbsp;Latencia de tiempo constante independientemente del espacio usado</br>&bull; &nbsp;Normalmente, menos de 2 minutos|

> [!NOTE]
>
> - En caso de cambiar el nivel de servicio o la escala de proceso para un grupo elástico, para calcular la estimación se debe usar la suma del espacio usado en todas las bases de datos en el grupo.
> - En caso de mover una base de datos a un grupo elástico o desde este, solo el espacio que usa la base de datos, y no el del grupo elástico, afecta a la latencia.
> - En el caso de los grupos elásticos Estándar y De uso general, la latencia para mover una base de datos dentro o fuera de un grupo elástico o entre grupos elásticos será proporcional al tamaño de la base de datos si el grupo elástico usa almacenamiento de recursos compartidos de archivos Premium ([PFS](https://docs.microsoft.com/azure/storage/files/storage-files-introduction)). Para determinar si un grupo usa el almacenamiento PFS, ejecute la siguiente consulta en el contexto de cualquier base de datos del grupo. Si el valor de la columna AccountType es `PremiumFileStorage`, el grupo utiliza el almacenamiento PFS.

```sql
SELECT s.file_id,
       s.type_desc,
       s.name,
       FILEPROPERTYEX(s.name, 'AccountType') AS AccountType
FROM sys.database_files AS s
WHERE s.type_desc IN ('ROWS', 'LOG');
```

> [!TIP]
> Para supervisar las operaciones en curso, consulte: [Administración de operaciones mediante la API REST de SQL](https://docs.microsoft.com/rest/api/sql/operations/list), [Administración de operaciones mediante la CLI](/cli/azure/sql/db/op), [Supervisión de operaciones mediante T-SQL](/sql/relational-databases/system-dynamic-management-views/sys-dm-operation-status-azure-sql-database) y estos dos comandos de PowerShell: [Get-AzSqlDatabaseActivity](/powershell/module/az.sql/get-azsqldatabaseactivity) y [Stop AzSqlDatabaseActivity](/powershell/module/az.sql/stop-azsqldatabaseactivity).

### <a name="additional-considerations-when-changing-service-tier-or-rescaling-compute-size"></a>Consideraciones adicionales cuando se cambia el nivel de servicio o la escala de tamaño de proceso

- Cuando se reducen los núcleos virtuales o las eDTU del grupo elástico, el grupo que se utilice debe ser menor que el tamaño máximo permitido del nivel de servicio de destino y de las eDTU de grupo.
- Cuando se cambia la escala de las eDTU de un grupo elástico, se aplica un costo de almacenamiento adicional si (1) el tamaño máximo de almacenamiento del grupo de es compatible con el grupo de destino y (2) el tamaño máximo de almacenamiento supera la cantidad de almacenamiento incluido del grupo de destino. Por ejemplo, si un grupo Estándar de 100 eDTU con un tamaño máximo de 100 GB se reduce a un grupo Estándar de 50 eDTU, se aplica un costo de almacenamiento adicional porque el grupo de destino admite un tamaño máximo de 100 GB y su cantidad de almacenamiento incluido es solo de 50 GB. Por lo tanto, la cantidad de almacenamiento adicional es 100 GB – 50 GB = 50 GB. Para conocer el precio del almacenamiento adicional, consulte los [precios de SQL Database](https://azure.microsoft.com/pricing/details/sql-database/). Si la cantidad de espacio real utilizada es menor que la cantidad de almacenamiento incluido, este costo adicional puede evitarse si se reduce el tamaño máximo de la base de datos a la cantidad incluida.

### <a name="billing-during-rescaling"></a>Facturación durante el cambio de escala

Se le cobrará por cada hora que una base de datos exista con el mayor nivel de servicio + tamaño de proceso aplicable durante esa hora, independientemente del uso o de si la base de datos estuvo activa durante menos tiempo. Por ejemplo, si crea una base de datos única y la elimina a los cinco minutos, se le efectuará un cargo de una hora por usar la base de datos.

## <a name="change-elastic-pool-storage-size"></a>cambiar el tamaño de almacenamiento de grupos elásticos

> [!IMPORTANT]
> En algunas circunstancias, puede que deba reducir una base de datos para reclamar el espacio no utilizado. Para obtener más información, consulte [Administración del espacio de archivo en Azure SQL Database](file-space-manage.md).

### <a name="vcore-based-purchasing-model"></a>Modelo de compra basado en núcleo virtual

- Se puede aprovisionar el almacenamiento hasta el límite de tamaño máximo:

  - En el caso del almacenamiento en los niveles de servicio Estándar o Uso general, aumente o disminuya el tamaño en incrementos de 10 GB.
  - Para el almacenamiento en niveles de servicio Premium o Crítico para la empresa, aumente o disminuya el tamaño en incrementos de 250 GB.
- El almacenamiento de un grupo elástico se puede aprovisionar aumentando o reduciendo su tamaño máximo.
- El precio del almacenamiento para un grupo elástico es la cantidad de almacenamiento multiplicada por el precio de la unidad de almacenamiento del nivel de servicio. Para más información sobre el precio del almacenamiento adicional, consulte los [precios de SQL Database](https://azure.microsoft.com/pricing/details/sql-database/).

> [!IMPORTANT]
> En algunas circunstancias, puede que deba reducir una base de datos para reclamar el espacio no utilizado. Para obtener más información, consulte [Administración del espacio de archivo en Azure SQL Database](file-space-manage.md).

### <a name="dtu-based-purchasing-model"></a>Modelo de compra basado en DTU

- El precio de la eDTU de un grupo elástico incluye una cierta cantidad de almacenamiento sin ningún costo adicional. El almacenamiento adicional que supere la cantidad incluida se puede aprovisionar por un costo extra hasta el límite de tamaño máximo en incrementos de 250 GB hasta 1 TB, y luego en incrementos de 256 GB superando 1 TB. Para más información sobre los límites de tamaño máximo y las cantidades de almacenamiento incluidas, consulte [Grupos elásticos: tamaños de almacenamiento y de proceso](resource-limits-dtu-elastic-pools.md#elastic-pool-storage-sizes-and-compute-sizes).
- Se puede aprovisionar el almacenamiento adicional para un grupo elástico si se aumenta su tamaño máximo mediante [Azure Portal](elastic-pool-manage.md#azure-portal), [PowerShell](/powershell/module/az.sql/Get-AzSqlElasticPool), la [CLI de Azure](/cli/azure/sql/elastic-pool#az-sql-elastic-pool-update) o la [API de REST](https://docs.microsoft.com/rest/api/sql/elasticpools/update).
- El precio del almacenamiento adicional para un grupo de bases de datos elásticas es la cantidad de almacenamiento adicional multiplicada por el precio de la unidad de almacenamiento adicional del nivel de servicio. Para más información sobre el precio del almacenamiento adicional, consulte los [precios de SQL Database](https://azure.microsoft.com/pricing/details/sql-database/).

> [!IMPORTANT]
> En algunas circunstancias, puede que deba reducir una base de datos para reclamar el espacio no utilizado. Para obtener más información, consulte [Administración del espacio de archivo en Azure SQL Database](file-space-manage.md).

## <a name="next-steps"></a>Pasos siguientes

Para conocer los límites de recursos globales, consulte [Límites de recursos basados en núcleos virtuales de SQL Database para grupos elásticos](resource-limits-vcore-elastic-pools.md) y [SQL Database DTU-based resource limits - elastic pools](resource-limits-dtu-elastic-pools.md) (Límites de recursos basados en DTU de SQL Database para grupos elásticos).
