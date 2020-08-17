---
title: Restauración de una base de datos desde una copia de seguridad
titleSuffix: Azure SQL Database & SQL Managed Instance
description: Información acerca de la restauración a un momento dado, que le permite revertir una base de datos de Azure SQL Database o una instancia de Instancia administrada de Azure SQL hasta 35 días.
services: sql-database
ms.service: sql-db-mi
ms.subservice: service
ms.custom: ''
ms.devlang: ''
ms.topic: conceptual
author: anosov1960
ms.author: sashan
ms.reviewer: mathoma, carlrab, danil
ms.date: 09/26/2019
ms.openlocfilehash: 6b07b6c3e54f4aebcda6c2e84047ecd1a27b3d5b
ms.sourcegitcommit: 85eb6e79599a78573db2082fe6f3beee497ad316
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/05/2020
ms.locfileid: "87809500"
---
# <a name="recover-using-automated-database-backups---azure-sql-database--sql-managed-instance"></a>Recuperación de una base de datos de Azure SQL Database o Instancia administrada de Azure SQL mediante copias de seguridad automatizadas
[!INCLUDE[appliesto-sqldb-sqlmi](../includes/appliesto-sqldb-sqlmi.md)]

Las opciones a continuación están disponibles para la recuperación de bases de datos mediante las [copias de seguridad de base de datos automatizadas](automated-backups-overview.md). Puede:

- Crear una nueva base de datos en el mismo servidor, recuperada en un punto especificado en el tiempo durante el período de retención.
- Crear una base de datos en el mismo servidor, recuperada a la hora de eliminación de una base de datos eliminada.
- Crear una nueva base de datos en cualquier servidor de la misma región, recuperada en el momento de las copias de seguridad más recientes.
- Crear una nueva base de datos en cualquier servidor de cualquier región, recuperada en el momento de las copias de seguridad con replicación más recientes.

Si ha configurado la [retención de copia de seguridad a largo plazo](long-term-retention-overview.md) también puede crear una nueva base de datos desde cualquier copia de seguridad de retención a largo plazo en cualquier servidor.

> [!IMPORTANT]
> Durante la restauración no se puede sobrescribir ninguna base de datos existente.

De forma predeterminada, las copias de seguridad de Azure SQL Database e Instancia administrada de Azure SQL se almacenan en un almacenamiento de blobs de replicación geográfica (tipo de almacenamiento RA-GRS). Además, SQL Managed Instance también admite el almacenamiento de copia de seguridad con redundancia local (LRS) y con redundancia de zona (ZRS). La redundancia garantiza que los datos estén protegidos frente a eventos previstos e imprevistos, como errores transitorios del hardware, interrupciones del suministro eléctrico o de la red y desastres naturales masivos. El almacenamiento con redundancia de zona (ZRS) solo está disponible en [determinadas regiones](../../storage/common/storage-redundancy.md#zone-redundant-storage).

> [!IMPORTANT]
> La configuración de la redundancia de almacenamiento para las copias de seguridad solo está disponible para la instancia administrada y se permite durante el proceso de creación. Una vez que se ha aprovisionado el recurso, no se puede cambiar la opción de redundancia del almacenamiento de copia de seguridad.

Cuando se usan los niveles de servicio Estándar o Premium, la restauración de la base de datos puede suponer un costo de almacenamiento adicional. El costo adicional se genera cuando el tamaño máximo de la base de datos restaurada es mayor que la cantidad de almacenamiento incluida en el nivel de rendimiento y el nivel de servicio de la base de datos de destino. Para más información sobre los precios del almacenamiento adicional, consulte la [página de precios de SQL Database](https://azure.microsoft.com/pricing/details/sql-database/). Cuando la cantidad de espacio real usado es menor que la cantidad de almacenamiento incluido, se puede evitar este costo adicional con el establecimiento del tamaño máximo de la base de datos en la cantidad incluida.

## <a name="recovery-time"></a>Tiempo de recuperación

El tiempo de recuperación para restaurar una base de datos mediante copias de seguridad de base de datos automatizadas se ve afectado por una serie de factores:

- El tamaño de la base de datos
- El tamaño de proceso de la base de datos
- El número de registros de transacciones implicados
- La cantidad de actividad que tiene que reproducirse para la recuperación hasta el punto de restauración
- El ancho de banda de red si la restauración se realiza en una región diferente
- El número de solicitudes de restauración simultáneas que se están procesando en la región de destino.

En bases de datos grandes o muy activas, la restauración puede tardar varias horas. Si se produce una interrupción prolongada en una región, es posible que se inicie un elevado número de solicitudes de restauración geográfica para la recuperación ante desastres. Si hay muchas solicitudes, el tiempo de recuperación de las bases de datos individuales puede aumentar. La mayoría de las restauraciones de bases de datos se completan en menos de 12 horas.

Para una única suscripción, existen limitaciones en el número de solicitudes simultáneas de restauración. Estas limitaciones se aplican a cualquier combinación de restauraciones a un momento dado, restauraciones geográficas y restauraciones a partir de copias de seguridad de retención a largo plazo.

| **Opción de implementación** | **Número máximo de solicitudes simultáneas que se van a procesar** | **Número máximo de solicitudes simultáneas que se van a enviar** |
| :--- | --: | --: |
|**Base de datos única (por suscripción)**|10|60|
|**Grupo elástico (por grupo)**|4|200|


No hay ningún método integrado para restaurar todo el servidor. Para obtener un ejemplo de cómo realizar esta tarea, consulte [Azure SQL Database: Recuperación completa del servidor](https://gallery.technet.microsoft.com/Azure-SQL-Database-Full-82941666).

> [!IMPORTANT]
> Para poder efectuar una recuperación con copias de seguridad automatizadas, tiene que ser miembro del rol de colaborador de SQL Server o de Instancia administrada de SQL (en función del destino de recuperación) en la suscripción, o debe ser el propietario de la suscripción. Para más información, vea [RBAC: roles integrados](../../role-based-access-control/built-in-roles.md). Las recuperaciones se pueden realizar a través de Azure Portal, PowerShell o la API REST. No se puede utilizar Transact-SQL.

## <a name="point-in-time-restore"></a>Restauración a un momento dado

Puede restaurar una base de datos independiente, agrupada o de instancia a un momento dado anterior mediante Azure Portal, [PowerShell](https://docs.microsoft.com/powershell/module/az.sql/restore-azsqldatabase) o la [API REST](https://docs.microsoft.com/rest/api/sql/databases/createorupdate#creates-a-database-from-pointintimerestore.). La solicitud puede especificar cualquier nivel de servicio o calcular el tamaño de la base de datos restaurada. Asegúrese de que tiene suficientes recursos en el servidor en el que se va a restaurar la base de datos. 

Una vez completada, la restauración creará una nueva base de datos en el mismo servidor que la base de datos original. La base de datos restaurada se cobra según la tarifa normal en función de su nivel de servicio y tamaño de proceso. No se aplican cargos hasta que finaliza la restauración de la base de datos.

Por lo general, una base de datos se restaura a un punto anterior para fines de recuperación. Puede tratar la base de datos restaurada como sustituta de la base de datos original o usarla como origen de datos para actualizar la base de datos original.

- **Sustituto de base de datos**

  Si la base de datos restaurada está pensada como sustituto de la base de datos original, debe especificar el tamaño de proceso y el nivel de servicio de la base de datos original. Luego puede cambiar el nombre de la base de datos original y asignar a la base de datos restaurada el nombre original mediante el comando [ALTER DATABASE](/sql/t-sql/statements/alter-database-azure-sql-database) en T-SQL.

- **Recuperación de datos**

  Si va a recuperar datos de la base de datos restaurada para recuperarse de un error de usuario o de aplicación, debe escribir y ejecutar un script de recuperación de datos que extraiga datos de la base de datos restaurada y se aplique a la base de datos original. Aunque la operación de restauración puede tardar mucho tiempo en finalizar, la base de datos restaurada será visible en la lista de bases de datos en todo el proceso de restauración. Si elimina la base de datos durante la restauración, la operación de restauración se cancelará y no se le cobrará por la base de datos que no finalizó la restauración.
  
### <a name="point-in-time-restore-by-using-azure-portal"></a>Restauración a un momento dado con Azure Portal

Puede recuperar una base de datos única o una base de datos de instancia a un momento dado desde la hoja de información general de la base de datos que desea restaurar en Azure Portal.

#### <a name="sql-database"></a>SQL Database

Para recuperar una base de datos en un momento dado mediante Azure Portal, abra la página de información general de la base de datos y seleccione **Restaurar** en la barra de herramientas. Elija el origen de la copia de seguridad y seleccione el punto de copia de seguridad a un momento dado a partir del cual se creará una nueva base de datos.

  ![Captura de pantalla de opciones de restauración de base de datos](./media/recovery-using-backups/pitr-backup-sql-database-annotated.png)

#### <a name="sql-managed-instance"></a>Instancia administrada de SQL

Para recuperar una base de datos de instancia administrada a un momento dado mediante Azure Portal, abra la página de información general de la base de datos y seleccione **Restaurar** en la barra de herramientas. Elija el punto de copia de seguridad a un momento dado a partir del que se creará una nueva base de datos.

  ![Captura de pantalla de opciones de restauración de base de datos](./media/recovery-using-backups/pitr-backup-managed-instance-annotated.png)

> [!TIP]
> Para restaurar una base de datos mediante programación desde una copia de seguridad, consulte [Recuperación mediante programación con copias de seguridad automatizadas](recovery-using-backups.md).

## <a name="deleted-database-restore"></a>Restauración de la base de datos eliminada

Puede restaurar una base de datos eliminada al momento en que se eliminó o a un momento anterior en el mismo servidor o la misma instancia administrada. Para ello puede usar Azure Portal, [PowerShell](https://docs.microsoft.com/powershell/module/az.sql/restore-azsqldatabase) o [REST (createMode=Restore)](https://docs.microsoft.com/rest/api/sql/databases/createorupdate). La restauración de una base de datos eliminada se realiza mediante la creación de una nueva base de datos a partir de la copia de seguridad.

> [!IMPORTANT]
> Si elimina un servidor o una instancia administrada, todas sus bases de datos también se eliminan y no se pueden recuperar. No se puede restaurar un servidor o una instancia administrada que se hayan eliminado.

### <a name="deleted-database-restore-by-using-the-azure-portal"></a>Restauración de bases de datos eliminadas mediante Azure Portal

La restauración de bases de datos eliminadas de Azure Portal se realiza desde el servidor o el recurso de instancia administrada.

#### <a name="sql-database"></a>SQL Database

Para recuperar una base de datos eliminada al momento de eliminación con Azure Portal, abra la página de información general del servidor y seleccione **Bases de datos eliminadas**. Seleccione la base de datos eliminada que desea restaurar y escriba el nombre de la nueva base de datos que se creará con los datos restaurados desde la copia de seguridad.

  ![Captura de pantalla de restaurar base de datos eliminada](./media/recovery-using-backups/restore-deleted-sql-database-annotated.png)

#### <a name="sql-managed-instance"></a>Instancia administrada de SQL

Para recuperar una base de datos administrada mediante Azure Portal, abra la página información general de instancia administrada y seleccione bases de datos **eliminadas**. Seleccione la base de datos eliminada que desea restaurar y escriba el nombre de la nueva base de datos que se creará con los datos restaurados desde la copia de seguridad.

  ![Captura de pantalla de restauración de base de datos de Instancia administrada de Azure SQL eliminada](./media/recovery-using-backups/restore-deleted-sql-managed-instance-annotated.png)

### <a name="deleted-database-restore-by-using-powershell"></a>Restauración de bases de datos eliminadas mediante PowerShell

Use los siguientes scripts de ejemplo para restaurar una base de datos eliminada para SQL Database o la Instancia administrada de SQL mediante PowerShell.

#### <a name="sql-database"></a>SQL Database

Para obtener un script de PowerShell de ejemplo que muestre cómo realizar una restauración de una base de datos eliminada, consulte [Restauración de una base de datos con PowerShell](scripts/restore-database-powershell.md).

#### <a name="sql-managed-instance"></a>Instancia administrada de SQL

Para ver un script de PowerShell de ejemplo que muestre cómo restaurar una base de datos de instancia eliminada, consulte [Restauración una base de datos de instancia eliminada mediante PowerShell](../managed-instance/point-in-time-restore.md#restore-a-deleted-database).

> [!TIP]
> Para restaurar una base de datos eliminada mediante programación, consulte [Recuperación mediante programación con copias de seguridad automatizadas](recovery-using-backups.md).

## <a name="geo-restore"></a>Geo-restore

> [!IMPORTANT]
> La restauración geográfica solo está disponible para las instancias administradas configuradas con el tipo de almacenamiento de copia de seguridad con redundancia geográfica (RA-GRS). Las instancias administradas configuradas con tipos de almacenamiento de copia de seguridad con redundancia local o con redundancia de zona no admiten la restauración geográfica.

Puede restaurar una base de datos en cualquier servidor de SQL Database o en una base de datos de instancia en cualquier Instancia administrada de cualquier región de Azure a partir de las copias de seguridad con replicación geográfica más recientes. La restauración geográfica usa una copia de seguridad con replicación geográfica como su origen. Puede solicitar una restauración geográfica, aunque la base de datos o el centro de datos sea inaccesible debido a una interrupción.

La función de restauración geográfica proporciona la opción de recuperación predeterminada cuando la base de datos no está disponible debido a una incidencia en la región de hospedaje. Puede restaurar la base de datos a un servidor de cualquier otra región. Hay un retraso entre el momento en que se realiza una copia de seguridad y el momento en que se replica geográficamente en un blob de Azure de una región diferente. Como resultado, la base de datos restaurada puede encontrarse hasta una hora por detrás de la base de datos original. En la siguiente ilustración se muestra la restauración de una base de datos a partir de la última copia de seguridad disponible en otra región.

![Gráfico de restauración geográfica](./media/recovery-using-backups/geo-restore-2.png)

### <a name="geo-restore-by-using-the-azure-portal"></a>Restauración geográfica mediante Azure Portal

En Azure Portal, cree una nueva base de datos de instancia única o administrada y seleccione una copia de seguridad de restauración geográfica que esté disponible. La base de datos recién creada contiene los datos de la copia de seguridad de la restauración geográfica.

#### <a name="sql-database"></a>SQL Database

Para realizar una restauración geográfica de una base de datos única a partir de Azure Portal en la región y el servidor de su elección, siga estos pasos:

1. En **Panel**, seleccione **Agregar** > **Crear base de datos SQL**. En la pestaña **Aspectos básicos**, escriba la información necesaria.
2. Seleccione **Configuración adicional**.
3. Para **Usar datos existentes**, seleccione **Copia de seguridad**.
4. En **Copia de seguridad** en la lista desplegable de copias de seguridad de restauración geográfica disponibles seleccione una.

    ![Captura de pantalla de las opciones de Crear base de datos SQL](./media/recovery-using-backups/geo-restore-azure-sql-database-list-annotated.png)

Complete el proceso de creación de una nueva base de datos desde la copia de seguridad. Cuando cree una base de datos en Azure SQL Database, esta contendrá la copia de seguridad restaurada geográficamente.

#### <a name="sql-managed-instance"></a>Instancia administrada de SQL

Para realizar la restauración geográfica de una base de datos de instancia administrada de Azure Portal en una instancia administrada ya existente de una región de su elección, seleccione la instancia administrada en la que desea que se restaure la base de datos. Siga estos pasos:

1. Seleccione **Nueva base de datos**.
2. Escriba el nombre que desee para la base de datos.
3. En **Usar datos existentes**, seleccione **Copia de seguridad**.
4. En la lista desplegable de copias de seguridad de restauración geográfica disponibles seleccione una.

    ![Captura de pantalla de las opciones de Nueva base de datos](./media/recovery-using-backups/geo-restore-sql-managed-instance-list-annotated.png)

Complete el proceso de creación de una nueva base de datos. Cuando cree la base de datos de instancia, esta contendrá la copia de seguridad restaurada geográficamente.

### <a name="geo-restore-by-using-powershell"></a>Restauración geográfica mediante PowerShell

#### <a name="sql-database"></a>SQL Database

Para ver un script de PowerShell que indica cómo realizar la restauración geográfica de una base de datos única, consulte [Uso de PowerShell para restaurar una base de datos única a un momento anterior](scripts/restore-database-powershell.md).

#### <a name="sql-managed-instance"></a>Instancia administrada de SQL

Para ver un script de PowerShell que indica cómo realizar la restauración geográfica de una base de datos de instancia administrada, consulte [Uso de PowerShell para restaurar una base de datos de Instancia administrada en otra región con replicación geográfica](../managed-instance/scripts/restore-geo-backup.md).

### <a name="geo-restore-considerations"></a>Consideraciones sobre la restauración geográfica

No se puede realizar una restauración a un momento dado en una base de datos de replicación geográfica secundaria. Solo puede hacerlo en una base de datos principal. Para obtener información detallada sobre cómo usar la restauración geográfica a fin de recuperarse de una interrupción, consulte [Restauración de una base de datos SQL de Azure o una conmutación por error en una secundaria](../../key-vault/general/disaster-recovery-guidance.md).

> [!IMPORTANT]
> La restauración geográfica es la solución de recuperación ante desastres más básica disponible en SQL Database e Instancia administrada de SQL. Depende de copias de seguridad con replicación geográfica creadas automáticamente con el objetivo de punto de recuperación (RPO) igual a 1 hora y un tiempo de recuperación estimado de hasta 12 horas. No garantiza que la región de destino tenga la capacidad de restaurar las bases de datos después de una interrupción regional porque es probable que se produzca un fuerte aumento de la demanda. Si la aplicación usa bases de datos relativamente pequeñas y no es de importancia crítica para la empresa, la restauración geográfica es una solución de recuperación ante desastres adecuada. 
>
> En el caso de las aplicaciones críticas para la empresa que necesitan bases de datos grandes y deben garantizar la continuidad empresarial, use [Grupos de conmutación por error automática](auto-failover-group-overview.md). Ofrecen un RPO y un objetivo de tiempo de recuperación mucho menor, y la capacidad siempre está garantizada. 
>
> Para más información sobre las opciones de continuidad empresarial, vea [Introducción a la continuidad empresarial](business-continuity-high-availability-disaster-recover-hadr-overview.md).

## <a name="programmatic-recovery-using-automated-backups"></a>Recuperación mediante programación con copias de seguridad automatizadas

También puede usar Azure PowerShell o la API REST para la recuperación. En las tablas siguientes se describe el conjunto de comandos disponibles.

### <a name="powershell"></a>PowerShell

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]
> [!IMPORTANT]
> El módulo de Azure Resource Manager para PowerShell todavía es compatible con SQL Database e Instancia administrada de Azure SQL, pero todo el desarrollo futuro se realizará para el módulo Az.Sql. Para estos cmdlets, consulte [AzureRM.Sql](https://docs.microsoft.com/powershell/module/AzureRM.Sql/). Los argumentos para los comandos en el módulo Az y en los módulos de Azure Resource Manager son en gran medida idénticos.

#### <a name="sql-database"></a>SQL Database

Para restaurar una base de datos independiente o agrupada, vea [Restore-AzSqlDatabase](/powershell/module/az.sql/restore-azsqldatabase).

  | Cmdlet | Descripción |
  | --- | --- |
  | [Get-AzSqlDatabase](/powershell/module/az.sql/get-azsqldatabase) |Obtiene una o más bases de datos. |
  | [Get-AzSqlDeletedDatabaseBackup](/powershell/module/az.sql/get-azsqldeleteddatabasebackup) | Obtiene una base de datos eliminada que se puede restaurar. |
  | [Get-AzSqlDatabaseGeoBackup](/powershell/module/az.sql/get-azsqldatabasegeobackup) |Obtiene una copia de seguridad con redundancia geográfica de una base de datos. |
  | [Restore-AzSqlDatabase](/powershell/module/az.sql/restore-azsqldatabase) |Restaura una base de datos. |

  > [!TIP]
  > Para obtener un script de PowerShell de ejemplo que muestre cómo realizar una restauración de una base de datos a un momento dado, consulte [Restauración de una base de datos con PowerShell](scripts/restore-database-powershell.md).

#### <a name="sql-managed-instance"></a>Instancia administrada de SQL

Para restaurar una base de datos de instancia administrada, consulte [Restore-AzSqlInstanceDatabase](/powershell/module/az.sql/restore-azsqlinstancedatabase).

  | Cmdlet | Descripción |
  | --- | --- |
  | [Get-AzSqlInstance](/powershell/module/az.sql/get-azsqlinstance) |Obtiene una o más instancias administradas. |
  | [Get-AzSqlInstanceDatabase](/powershell/module/az.sql/get-azsqlinstancedatabase) | Obtiene una base de datos de instancias. |
  | [Restore-AzSqlInstanceDatabase](/powershell/module/az.sql/restore-azsqlinstancedatabase) |Restaura una base de datos de una instancia. |

### <a name="rest-api"></a>API DE REST

Restaurar una base de datos mediante API de REST:

| API | Descripción |
| --- | --- |
| [REST (createMode=Recovery)](https://docs.microsoft.com/rest/api/sql/databases) |Restaura una base de datos. |
| [Obtener el estado de creación o actualización de la base de datos](https://docs.microsoft.com/rest/api/sql/operations) |Devuelve el estado durante una operación de restauración. |

### <a name="azure-cli"></a>Azure CLI

#### <a name="sql-database"></a>SQL Database

Para restaurar una base de datos mediante la CLI de Azure, consulte la referencia sobre el comando [az sql db restore](/cli/azure/sql/db#az-sql-db-restore).

#### <a name="sql-managed-instance"></a>Instancia administrada de SQL

Para restaurar una base de datos de instancia administrada mediante la CLI de Azure, consulte la referencia sobre el comando [aaz sql midb restore](/cli/azure/sql/midb#az-sql-midb-restore).

## <a name="summary"></a>Resumen

Las copias de seguridad automáticas protegen las bases de datos de los errores de usuario y de aplicación, la eliminación accidental de la base de datos y las interrupciones prolongadas. Esta funcionalidad integrada está disponible para todos los niveles de servicio y tamaños de proceso.

## <a name="next-steps"></a>Pasos siguientes

- [Información general sobre la continuidad empresarial](business-continuity-high-availability-disaster-recover-hadr-overview.md)
- [Información general: copias de seguridad automatizadas de SQL Database](automated-backups-overview.md)
- [Retención a largo plazo](long-term-retention-overview.md)
- Para conocer las opciones de recuperación más rápidas, consulte el artículo sobre la [replicación geográfica activa](active-geo-replication-overview.md) y los [grupos de conmutación por error automáticos](auto-failover-group-overview.md).
