---
title: Escalado de recursos de bases de datos únicas
description: En este artículo se describe cómo escalar los recursos de proceso y almacenamiento disponibles para una base de datos única en Azure SQL Database.
services: sql-database
ms.service: sql-database
ms.subservice: performance
ms.custom: sqldbrb=1, references_regions
ms.devlang: ''
ms.topic: conceptual
author: stevestein
ms.author: sstein
ms.reviewer: carlrab
ms.date: 07/31/2020
ms.openlocfilehash: 39869e74fcb3e8f3deae1273721093f3f85e8d78
ms.sourcegitcommit: 8def3249f2c216d7b9d96b154eb096640221b6b9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/03/2020
ms.locfileid: "87541692"
---
# <a name="scale-single-database-resources-in-azure-sql-database"></a>Escalar recursos de base de datos única en Azure SQL Database

En este artículo se describe cómo escalar los recursos de proceso y almacenamiento disponibles para una instancia de Azure SQL Database en el nivel de proceso aprovisionado. Como alternativa, el [nivel de proceso sin servidor](serverless-tier-overview.md) proporciona escalado automático de proceso y se factura por segundo el proceso que se usa.

Después de elegir inicialmente el número de núcleos virtuales o DTU, puede escalar o reducir una base de datos verticalmente de manera dinámica en función de la experiencia real mediante [Azure Portal](single-database-manage.md#the-azure-portal), [Transact-SQL](https://docs.microsoft.com/sql/t-sql/statements/alter-database-transact-sql?view=azuresqldb-current#examples-1), [PowerShell](/powershell/module/az.sql/set-azsqldatabase), la [CLI de Azure](/cli/azure/sql/db#az-sql-db-update) o la [API REST](https://docs.microsoft.com/rest/api/sql/databases/update).

El vídeo siguiente muestra cómo cambiar de manera dinámica el nivel de servicio y el tamaño de proceso para aumentar las DTU disponibles para una base de datos única.

> [!VIDEO https://channel9.msdn.com/Blogs/Azure/Azure-SQL-Database-dynamically-scale-up-or-scale-down/player]

> [!IMPORTANT]
> En algunas circunstancias, puede que deba reducir una base de datos para reclamar el espacio no utilizado. Para obtener más información, consulte [Administración del espacio de archivo en Azure SQL Database](file-space-manage.md).

## <a name="impact"></a>Impacto

El cambio en el nivel de servicio o el tamaño de proceso implica principalmente que el servicio realice los pasos siguientes:

1. Cree una instancia de proceso nueva para la base de datos. 

    Se crea una instancia de proceso nueva con el tamaño de proceso y el nivel de servicio solicitados. Para algunas combinaciones de cambios en el nivel de servicio y el tamaño de proceso, se debe crear una réplica de la base de datos en la nueva instancia de proceso, lo que implica copiar los datos y puede influir en gran medida en la latencia general. En cualquier caso, la base de datos permanece en línea durante este paso y las conexiones se continúan dirigiendo a la base de datos en la instancia de proceso original.

2. Cambie el enrutamiento de las conexiones a una nueva instancia de proceso.

    Se colocan las conexiones existentes en la base de datos en la instancia de proceso original. En la nueva instancia de proceso las nuevas conexiones se establecen en la base de datos. Para algunas combinaciones de cambios de nivel de servicio y de tamaño de proceso, los archivos de base de datos se desasocian y se vuelven a asociar durante la modificación.  No obstante, el modificador puede provocar una breve interrupción del servicio cuando la base de datos no esté disponible de forma general durante menos de 30 segundos y, a menudo, durante solo unos segundos. Si hay transacciones de ejecución prolongada que se ejecutan cuando se eliminan las conexiones, la duración de este paso puede ser mayor con el fin de recuperar las transacciones anuladas. La [Recuperación de base de datos acelerada](../accelerated-database-recovery.md) puede reducir el impacto de la anulación de transacciones de larga duración.

> [!IMPORTANT]
> Durante los pasos del flujo de trabajo no se pierden datos. Asegúrese de que ha implementado alguna [lógica de reintento](troubleshoot-common-connectivity-issues.md) en las aplicaciones y los componentes que utilizan Azure SQL Database mientras se cambia el nivel de servicio.

## <a name="latency"></a>Latencia

La latencia estimada para cambiar el nivel de servicio, escalar el tamaño de proceso de una sola base de datos o grupo elástico, trasladar una base de datos dentro o fuera de un grupo elástico o trasladar una base de datos entre grupos elásticos incluye los parámetros que se indican a continuación:

|Nivel de servicio|Base de datos única básica,</br>estándar (S0-S1)|Grupo elástico básico,</br>estándar (S2-S12), </br>base de datos única o grupo elástico de uso general|Base de datos única o grupo elástico Premium o Crítico para la empresa|Hiperescala
|:---|:---|:---|:---|:---|
|**Base de datos único básica,</br> estándar (S0-S1)**|&bull; &nbsp;Latencia de tiempo constante independientemente del espacio usado</br>&bull; &nbsp;Normalmente, menos de 5 minutos|&bull; &nbsp;Latencia proporcional al espacio usado en la base de datos debido a la copia de datos</br>&bull; &nbsp;Normalmente, menos de 1 minuto por GB de espacio usado|&bull; &nbsp;Latencia proporcional al espacio usado en la base de datos debido a la copia de datos</br>&bull; &nbsp;Normalmente, menos de 1 minuto por GB de espacio usado|&bull; &nbsp;Latencia proporcional al espacio usado en la base de datos debido a la copia de datos</br>&bull; &nbsp;Normalmente, menos de 1 minuto por GB de espacio usado|
|**Grupo elástico básico, </br>estándar (S2-S12) </br>base de datos única o grupo elástico de uso general**|&bull; &nbsp;Latencia proporcional al espacio usado en la base de datos debido a la copia de datos</br>&bull; &nbsp;Normalmente, menos de 1 minuto por GB de espacio usado|&bull; &nbsp;Latencia de tiempo constante independientemente del espacio usado</br>&bull; &nbsp;Normalmente, menos de 5 minutos|&bull; &nbsp;Latencia proporcional al espacio usado en la base de datos debido a la copia de datos</br>&bull; &nbsp;Normalmente, menos de 1 minuto por GB de espacio usado|&bull; &nbsp;Latencia proporcional al espacio usado en la base de datos debido a la copia de datos</br>&bull; &nbsp;Normalmente, menos de 1 minuto por GB de espacio usado|
|**Base de datos única o grupo elástico Premium o Crítico para la empresa**|&bull; &nbsp;Latencia proporcional al espacio usado en la base de datos debido a la copia de datos</br>&bull; &nbsp;Normalmente, menos de 1 minuto por GB de espacio usado|&bull; &nbsp;Latencia proporcional al espacio usado en la base de datos debido a la copia de datos</br>&bull; &nbsp;Normalmente, menos de 1 minuto por GB de espacio usado|&bull; &nbsp;Latencia proporcional al espacio usado en la base de datos debido a la copia de datos</br>&bull; &nbsp;Normalmente, menos de 1 minuto por GB de espacio usado|&bull; &nbsp;Latencia proporcional al espacio usado en la base de datos debido a la copia de datos</br>&bull; &nbsp;Normalmente, menos de 1 minuto por GB de espacio usado|
|**Hiperescala**|N/D|N/D|N/D|&bull; &nbsp;Latencia de tiempo constante independientemente del espacio usado</br>&bull; &nbsp;Normalmente, menos de 2 minutos|

> [!NOTE]
> Además, en el caso de las bases de datos Estándar (S2-S12) y De uso general, la latencia para mover una base de datos dentro o fuera de un grupo elástico o entre grupos elásticos será proporcional al tamaño de la base de datos si esta usa almacenamiento de recursos compartidos de archivos Premium ([PFS](https://docs.microsoft.com/azure/storage/files/storage-files-introduction)).
>
> Para determinar si una base de datos usa el almacenamiento PFS, ejecute la siguiente consulta en el contexto de la base de datos. Si el valor de la columna AccountType es `PremiumFileStorage`, la base de datos utiliza el almacenamiento PFS.
 
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

## <a name="cancelling-changes"></a>Cancelación de cambios

Un cambio del nivel de servicio o de la escala del proceso se puede cancelar.

### <a name="the-azure-portal"></a>El Portal de Azure

En la hoja de información general de la base de datos, vaya a **Notificaciones** y haga clic en el icono que indica que hay una operación en curso:

![Operación en curso](./media/single-database-scale/ongoing-operations.png)

A continuación, haga clic en el botón **Cancelar esta operación**.

![Cancelación de la operación en curso](./media/single-database-scale/cancel-ongoing-operation.png)

### <a name="powershell"></a>PowerShell

Desde un símbolo del sistema de PowerShell, defina los valores `$resourceGroupName`, `$serverName` y `$databaseName` y, después, ejecute el comando siguiente:

```azurecli
$operationName = (az sql db op list --resource-group $resourceGroupName --server $serverName --database $databaseName --query "[?state=='InProgress'].name" --out tsv)
if (-not [string]::IsNullOrEmpty($operationName)) {
    (az sql db op cancel --resource-group $resourceGroupName --server $serverName --database $databaseName --name $operationName)
        "Operation " + $operationName + " has been canceled"
}
else {
    "No service tier change or compute rescaling operation found"
}
```

## <a name="additional-considerations"></a>Consideraciones adicionales

- Si va a actualizar a un nivel de servicio o tamaño de proceso más elevado, el tamaño máximo de la base de datos no aumenta a menos que especifique un tamaño mayor (maxsize).
- Para cambiar una base de datos a una versión anterior, su espacio usado no debe alcanzar el tamaño máximo permitido del nivel de servicio de destino y del tamaño de proceso.
- Al pasar del nivel **Premium** al nivel **Estándar**, se aplica un costo de almacenamiento adicional si (1) el tamaño máximo de la base de datos es compatible con el tamaño de proceso de destino y (2) el tamaño máximo supera la cantidad de almacenamiento incluido del tamaño de proceso de destino. Por ejemplo, si una base de datos P1 con un tamaño máximo de 500 GB se reduce a S3, se aplica un costo de almacenamiento adicional porque S3 admite un tamaño máximo de 1 TB y su cantidad de almacenamiento incluido es solo de 250 GB. Por lo tanto, la cantidad de almacenamiento adicional es 500 GB – 250 GB = 250 GB. Para conocer el precio del almacenamiento adicional, consulte los [precios de Azure SQL Database](https://azure.microsoft.com/pricing/details/sql-database/). Si la cantidad de espacio real utilizada es menor que la cantidad de almacenamiento incluido, este costo adicional puede evitarse si se reduce el tamaño máximo de la base de datos a la cantidad incluida.
- Al actualizar una base de datos con la [replicación geográfica](active-geo-replication-configure-portal.md) habilitada, actualice sus bases de datos secundarias al nivel de servicio y al tamaño de proceso deseados antes de actualizar la base de datos principal (instrucciones generales para mejorar el rendimiento). Si se realiza la actualización a una edición diferente, es necesario actualizar primero la base de datos secundaria.
- Al cambiar a una versión anterior la base de datos con la [replicación geográfica](active-geo-replication-configure-portal.md) habilitada, se cambian también sus bases de datos principales al nivel de servicio y tamaño de proceso deseados antes de cambiar a una versión anterior la base de datos secundaria (regla general para un mejor rendimiento). Si se realiza la actualización a una edición diferente, es necesario actualizar primero la base de datos secundaria.
- Las ofertas del servicio de restauración son diferentes para los distintos niveles de servicio. Si va a cambiar al nivel **Básico**, hay un período de retención de copia de seguridad más bajo. Consulte el artículo sobre [Copias de seguridad de Azure SQL Database](automated-backups-overview.md).
- Las nuevas propiedades de la base de datos no se aplican hasta que se completan los cambios.

## <a name="billing"></a>Facturación

Se le cobrará por cada hora que una base de datos exista con el mayor nivel de servicio + tamaño de proceso aplicable durante esa hora, independientemente del uso o de si la base de datos estuvo activa durante menos de una hora. Por ejemplo, si crea una base de datos única y la elimina a los cinco minutos, se le efectuará un cargo de una hora por usar la base de datos.

## <a name="change-storage-size"></a>Modificar el tamaño de almacenamiento

### <a name="vcore-based-purchasing-model"></a>Modelo de compra basado en núcleo virtual

- Se puede aprovisionar el almacenamiento hasta el límite máximo de tamaño del almacenamiento de datos con incrementos de 1 GB. El almacenamiento de datos mínimo configurable es 1 GB. Consulte las páginas de documentación del límite de recursos para [bases de datos únicas](resource-limits-vcore-single-databases.md) y [grupos elásticos](resource-limits-vcore-elastic-pools.md) a fin de ver los límites máximos de tamaño del almacenamiento de datos en cada objetivo del servicio.
- Se puede aprovisionar almacenamiento de datos para una base de datos única mediante el aumento o disminución de su tamaño máximo con [Azure Portal](https://portal.azure.com), [Transact-SQL](https://docs.microsoft.com/sql/t-sql/statements/alter-database-transact-sql?view=azuresqldb-current#examples-1), [PowerShell](/powershell/module/az.sql/set-azsqldatabase), la [CLI de Azure](/cli/azure/sql/db#az-sql-db-update) o la [API de REST](https://docs.microsoft.com/rest/api/sql/databases/update). Si el valor de tamaño máximo se especifica en bytes, debe ser un múltiplo de 1 GB (1073741824 bytes).
- La cantidad de datos que se pueden almacenar en los archivos de datos de una base de datos está limitada por el tamaño máximo del almacenamiento de datos configurado. Además de ese almacenamiento, Azure SQL Database asigna automáticamente un 30 % de almacenamiento adicional que se va a usar para el registro de transacciones.
- Azure SQL Database asigna automáticamente 32 GB por núcleo virtual para la base de datos `tempdb`. `tempdb` se encuentra en el almacenamiento local de SSD en todos los niveles de servicio.
- El precio de almacenamiento para una única base de datos o un grupo elástico es la suma de las cantidades de almacenamiento de datos y de registros de transacciones, multiplicada por el precio de la unidad de almacenamiento del nivel de servicio. El costo de `tempdb` se incluye en el precio. Para más información sobre el precio del almacenamiento, consulte los [precios de Azure SQL Database](https://azure.microsoft.com/pricing/details/sql-database/).

> [!IMPORTANT]
> En algunas circunstancias, puede que deba reducir una base de datos para reclamar el espacio no utilizado. Para obtener más información, consulte [Administración del espacio de archivo en Azure SQL Database](file-space-manage.md).

### <a name="dtu-based-purchasing-model"></a>Modelo de compra basado en DTU

- El precio de la DTU para una base de datos única incluye una cierta cantidad de almacenamiento sin costo adicional. El almacenamiento adicional que supere la cantidad incluida se puede aprovisionar por un costo extra hasta el límite de tamaño máximo en incrementos de 250 GB hasta 1 TB, y luego en incrementos de 256 GB superando 1 TB. Para más información sobre los límites de tamaño máximo y las cantidades de almacenamiento incluidas, consulte [Base de datos única: tamaños de almacenamiento y tamaños de proceso](resource-limits-dtu-single-databases.md#single-database-storage-sizes-and-compute-sizes).
- Se puede aprovisionar almacenamiento adicional para una base de datos única aumentando su tamaño máximo con Azure Portal, [Transact-SQL](https://docs.microsoft.com/sql/t-sql/statements/alter-database-transact-sql?view=azuresqldb-current#examples-1), [PowerShell](/powershell/module/az.sql/set-azsqldatabase), la [CLI de Azure](/cli/azure/sql/db#az-sql-db-update) o la [API de REST](https://docs.microsoft.com/rest/api/sql/databases/update).
- El precio del almacenamiento adicional para una base de datos única es la cantidad de almacenamiento adicional multiplicado por el precio de la unidad de almacenamiento adicional del nivel de servicio. Para más información sobre el precio del almacenamiento adicional, consulte los [precios de Azure SQL Database](https://azure.microsoft.com/pricing/details/sql-database/).

> [!IMPORTANT]
> En algunas circunstancias, puede que deba reducir una base de datos para reclamar el espacio no utilizado. Para obtener más información, consulte [Administración del espacio de archivo en Azure SQL Database](file-space-manage.md).

### <a name="geo-replicated-database"></a>Base de datos con replicación geográfica

Para cambiar el tamaño de una base de datos secundaria replicada, cambie el tamaño de la base de datos principal. A continuación, este cambio se replicará y se implementará también en la base de datos secundaria.

## <a name="p11-and-p15-constraints-when-max-size-greater-than-1-tb"></a>P11 y P15 se restringen cuando el tamaño máximo es mayor de 1 TB

Existe más de 1 TB de almacenamiento en el nivel Premium actualmente disponible en todas las regiones excepto: Este de China, Norte de China, Centro de Alemania y Noreste de Alemania. En estas regiones, el almacenamiento máximo en el nivel Prémium está limitado a 1 TB. Las siguientes consideraciones y limitaciones se aplican a las bases de datos P11 y P15 con un tamaño máximo mayor de 1 TB:

- Si el tamaño máximo de una base de datos P11 o P15 nunca se estableció en un valor mayor que 1 TB, entonces solo puede restablecerse o copiarse a una base de datos P11 o P15.  Posteriormente, la base de datos puede escalarse de nuevo a un tamaño de proceso diferente siempre que la cantidad de espacio asignado en el momento de la operación de cambio de escala no supere los límites de tamaño máximo con el nuevo tamaño de proceso.
- En escenarios de replicación geográfica activa:
  - Configuración de una relación de replicación geográfica: Si la base de datos principal es P11 o P15, las secundarias también deben ser P11 o P15. Un menor tamaño de proceso se rechaza como secundarias, ya que no admiten más de 1 TB.
  - Actualización de la base de datos principal en una relación de replicación geográfica: al cambiar el tamaño máximo a 1 TB en una base de datos principal, se desencadenará el mismo cambio en la base de datos secundaria. Ambas actualizaciones deben realizarse correctamente para que el cambio en la principal surta efecto. Se aplican limitaciones por región para la opción de más de 1 TB. Si la base de datos secundaria está en una región que no admite más de 1 TB, no se actualizará la principal.
- No se admite el uso del servicio Import/Export para cargar bases de datos P11 y P15 con más de 1 TB. Use SqlPackage.exe para [importar](database-import.md) y [exportar](database-export.md) datos.

## <a name="next-steps"></a>Pasos siguientes

Para conocer los límites de recursos globales, consulte [Límites de recursos basados en núcleos virtuales de Azure SQL Database para bases de datos únicas](resource-limits-vcore-single-databases.md) y [Límites de recursos de bases de datos únicas que usan el modelo de compra de DTU: Azure SQL Database](resource-limits-dtu-single-databases.md).
 