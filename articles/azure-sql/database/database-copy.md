---
title: Copia de una base de datos
description: Cree una copia transaccionalmente coherente de una base de datos de Azure SQL Database existente, ya sea en el mismo servidor o en otro distinto.
services: sql-database
ms.service: sql-database
ms.subservice: data-movement
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: conceptual
author: stevestein
ms.author: sashan
ms.reviewer: carlrab
ms.date: 07/29/2020
ms.openlocfilehash: 02ff222337e1b1c22df79724c232d4ca2b8b9f67
ms.sourcegitcommit: 152c522bb5ad64e5c020b466b239cdac040b9377
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/14/2020
ms.locfileid: "88225740"
---
# <a name="copy-a-transactionally-consistent-copy-of-a-database-in-azure-sql-database"></a>Creación de una copia transaccionalmente coherente de una base de datos de Azure SQL Database

[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

Azure SQL Database proporciona varios métodos para crear una copia de una [base de datos](single-database-overview.md) existente en el mismo servidor o en un servidor diferente. Una base de datos se puede copiar usando Azure Portal, PowerShell, la CLI de Azure o T-SQL.

## <a name="overview"></a>Información general

Una copia de base de datos es una instantánea coherente con las transacciones de la base de datos de origen a partir de un momento dado después de iniciarse la solicitud de copia. Puede seleccionar el mismo servidor u otro distinto para la copia. También puede optar por conservar el nivel de servicio y tamaño de proceso de la base de datos de origen, o usar un tamaño de proceso diferente dentro del mismo nivel de servicio o uno diferente. Cuando se complete la copia, esta se convierte en una base de datos independiente y completamente funcional. Los inicios de sesión, los usuarios y los permisos de la base de datos copiada se administran independientemente de la base de datos de origen. La copia se crea mediante la tecnología de replicación geográfica. Una vez completada la propagación de réplicas, el vínculo de replicación geográfica finaliza automáticamente. Todos los requisitos para usar la replicación geográfica se aplican a la operación de copia de la base de datos. Consulte [Información general de la replicación geográfica activa](active-geo-replication-overview.md) para obtener más información.

## <a name="logins-in-the-database-copy"></a>Inicios de sesión en la copia de la base de datos

Al copiar una base de datos en el mismo servidor, los mismos inicios de sesión se pueden usar en ambas bases de datos. La entidad de seguridad que usa para copiar la base de datos se convierte en el propietario de la base de datos en la nueva base de datos.

Al copiar una base de datos en un servidor diferente, la entidad de seguridad que ha iniciado la operación de copia en el servidor de destino se convierte en el propietario de la nueva base de datos.

Independientemente del servidor de destino, todos los usuarios de base de datos, sus permisos y sus identificadores de seguridad (SID) se copian en la copia de la base de datos. Usar [usuarios de base de datos contenidos](logins-create-manage.md) para el acceso a datos asegura que la base de datos copiada tenga las mismas credenciales de usuario, de tal forma que, una vez completada la copia, pueda obtener acceso inmediato a ellas con las mismas credenciales.

Si utiliza inicios de sesión de nivel de servidor para el acceso a datos y copia la base de datos en otro servidor, es posible que el acceso basado en inicios de sesión no funcione. Esto puede ocurrir porque los inicios de sesión no existen en el servidor de destino o porque sus contraseñas e identificadores de seguridad (SID) son diferentes. Vea [Administración de la seguridad de Azure SQL Database después de la recuperación ante desastres](active-geo-replication-security-configure.md) para obtener información sobre cómo administrar inicios de sesión al copiar una base de datos en un servidor diferente. Una vez que la operación de copia en un servidor diferente se realiza correctamente y antes de que se reasignen otros usuarios, solo el inicio de sesión asociado al propietario de la base de datos o el administrador del servidor pueden iniciar sesión en la base de datos copiada. Para resolver los inicios de sesión y establecer el acceso a los datos una vez completada la operación de copia, consulte [Resolución de inicios de sesión](#resolve-logins).

## <a name="copy-using-the-azure-portal"></a>Copia con Azure Portal

Para copiar una base de datos mediante Azure Portal, abra la página de la base de datos y haga clic en **Copiar**.

   ![Copia de base de datos](./media/database-copy/database-copy.png)

## <a name="copy-using-powershell-or-the-azure-cli"></a>Copia con PowerShell o la CLI de Azure

Para copiar una base de datos, use los ejemplos siguientes.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

En el caso de PowerShell, use el cmdlet [New-AzSqlDatabaseCopy](/powershell/module/az.sql/new-azsqldatabasecopy).

> [!IMPORTANT]
> El módulo de Azure Resource Manager (RM) para PowerShell todavía es compatible con Azure SQL Database, pero todo el desarrollo futuro se realizará para el módulo Az.Sql. El módulo de AzureRM continuará recibiendo correcciones de errores hasta diciembre de 2020 como mínimo.  Los argumentos para los comandos del módulo Az y los módulos AzureRm son esencialmente idénticos. Para obtener más información sobre la compatibilidad, vea [Presentación del nuevo módulo Az de Azure PowerShell](/powershell/azure/new-azureps-module-az).

```powershell
New-AzSqlDatabaseCopy -ResourceGroupName "<resourceGroup>" -ServerName $sourceserver -DatabaseName "<databaseName>" `
    -CopyResourceGroupName "myResourceGroup" -CopyServerName $targetserver -CopyDatabaseName "CopyOfMySampleDatabase"
```

La copia de la base de datos es una operación asincrónica, pero la base de datos de destino se crea inmediatamente después de aceptar la solicitud. Si tiene que cancelar la operación de copia mientras todavía está en curso, quite la base de datos de destino mediante el cmdlet [Remove-AzSqlDatabase](/powershell/module/az.sql/new-azsqldatabase).

Para obtener un script completo de PowerShell de ejemplo, consulte [Copia de una base de datos en un nuevo servidor](scripts/copy-database-to-new-server-powershell.md).

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

```azurecli
az sql db copy --dest-name "CopyOfMySampleDatabase" --dest-resource-group "myResourceGroup" --dest-server $targetserver `
    --name "<databaseName>" --resource-group "<resourceGroup>" --server $sourceserver
```

La copia de la base de datos es una operación asincrónica, pero la base de datos de destino se crea inmediatamente después de aceptar la solicitud. Si tiene que cancelar la operación de copia mientras todavía está en curso, coloque la base de datos de destino mediante el comando [az sql db delete](/cli/azure/sql/db#az-sql-db-delete).

* * *

## <a name="copy-using-transact-sql"></a>Copia con Transact-SQL

Inicie sesión en la base de datos maestra con el inicio de sesión de administrador del servidor o el inicio de sesión que creó la base de datos que quiere copiar. Para que la copia de la base de datos sea correcta, los inicios de sesión que no son el administrador del servidor deben ser miembros del rol `dbmanager`. Para obtener más información sobre los inicios de sesión y conectarse al servidor, consulte [Administración de inicios de sesión](logins-create-manage.md).

Inicie la copia de la base de datos de origen con la instrucción [CREATE DATABASE… AS COPY OF](https://docs.microsoft.com/sql/t-sql/statements/create-database-transact-sql?view=azuresqldb-current#copy-a-database). La instrucción T-SQL continúa ejecutándose hasta que se completa la operación de copia de la base de datos.

> [!NOTE]
> La finalización de la instrucción T-SQL no finaliza la operación de copia de la base de datos. Para finalizar la operación, quite la base de datos de destino.
>

### <a name="copy-to-the-same-server"></a>Copia en el mismo servidor

Inicie sesión en la base de datos maestra con el inicio de sesión de administrador del servidor o el inicio de sesión que creó la base de datos que quiere copiar. Para que la copia de la base de datos sea correcta, los inicios de sesión que no son el administrador del servidor deben ser miembros del rol `dbmanager`.

Este comando copia Base de datos1 en una base de datos nueva denominada Base de datos2 del mismo servidor. Según el tamaño de su base de datos, la operación de copia puede tardar algún tiempo en completarse.

   ```sql
   -- execute on the master database to start copying
   CREATE DATABASE Database2 AS COPY OF Database1;
   ```

### <a name="copy-to-a-different-server"></a>Copia en un servidor diferente

Inicie sesión en la base de datos maestra del servidor de destino donde se creará la nueva base de datos. Utilice un inicio de sesión que tenga el mismo nombre y la misma contraseña que el propietario de la base de datos de origen del servidor de origen. El inicio de sesión del servidor de destino también deber ser miembro del rol `dbmanager` o ser el inicio de sesión del administrador del servidor.

Este comando copia Base de datos1 del servidor1 en una nueva base de datos denominada Base de datos2 del servidor2. Según el tamaño de su base de datos, la operación de copia puede tardar algún tiempo en completarse.

```sql
-- Execute on the master database of the target server (server2) to start copying from Server1 to Server2
CREATE DATABASE Database2 AS COPY OF server1.Database1;
```

> [!IMPORTANT]
> Los firewalls de ambos servidores deben estar configurados para permitir la conexión de entrada desde la dirección IP del cliente que emite el comando T-SQL CREATE DATABASE... AS COPY OF.

### <a name="copy-to-a-different-subscription"></a>Copia en una suscripción diferente

Puede seguir los pasos descritos en la sección [Copiar una base de datos SQL en un servidor diferente](#copy-to-a-different-server) para copiar la base de datos en un servidor en una suscripción diferente usando T-SQL. Asegúrese de utilizar un inicio de sesión que tenga el mismo nombre y la misma contraseña que el propietario de la base de datos de origen. Además, el inicio de sesión debe ser miembro del rol `dbmanager` o administrador del servidor, tanto en el servidor de origen como en el de destino.

> [!NOTE]
> [Azure Portal](https://portal.azure.com), PowerShell y la CLI de Azure no permiten copiar bases de datos en una suscripción diferente.

> [!TIP]
> La copia de base de datos mediante T-SQL permite copiar una base de datos de una suscripción en un inquilino de Azure diferente. Esto solo se admite cuando se usa un inicio de sesión de autenticación de SQL para iniciar sesión en el servidor de destino.

## <a name="monitor-the-progress-of-the-copying-operation"></a>Supervisión del progreso de la operación de copia

Para supervisar el proceso de copia, consulte las vistas [sys.databases](https://docs.microsoft.com/sql/relational-databases/system-catalog-views/sys-databases-transact-sql), [sys.dm_database_copies](https://docs.microsoft.com/sql/relational-databases/system-dynamic-management-views/sys-dm-database-copies-azure-sql-database) y [sys.dm_operation_status](https://docs.microsoft.com/sql/relational-databases/system-dynamic-management-views/sys-dm-operation-status-azure-sql-database). Mientras que la copia esté en curso, la columna **state_desc** de la vista sys.databases para la nueva base de datos se establece en **COPYING**.

* Si se produce un error en el proceso de copia, la columna **state_desc** de la vista sys.databases para la nueva base de datos se establece en **SUSPECT**. Ejecute la instrucción DROP en la nueva base de datos e inténtelo de nuevo más tarde.
* Si el proceso de copia es correcto, la columna **state_desc** de la vista sys.databases para la nueva base de datos se establece en **ONLINE**. Se completa la copia y la nueva base de datos es una base de datos normal, que se puede modificar independientemente de la base de datos de origen.

> [!NOTE]
> Si decide cancelar la copia mientras está en curso, ejecute la instrucción [DROP DATABASE](https://docs.microsoft.com/sql/t-sql/statements/drop-database-transact-sql) en la nueva base de datos.

> [!IMPORTANT]
> Si necesita crear una copia con un objetivo de servicio bastante más pequeño que el origen, es posible que la base de datos de destino no tenga suficientes recursos para completar el proceso de inicialización y que se produzca un error en la operación de copia. En este escenario, use una solicitud de restauración geográfica para crear una copia en un servidor o una región diferentes. Para más información, vea [Recuperación de una base de datos de Azure SQL mediante copias de seguridad de base de datos](recovery-using-backups.md#geo-restore).

## <a name="azure-roles-to-manage-database-copy"></a>Roles de Azure para administrar la copia de la base de datos

Para crear una copia de la base de datos, debe tener los siguientes roles:

* Propietario de la suscripción o
* Rol de colaborador de SQL Server o
* Rol personalizado en las bases de datos de origen y de destino con el siguiente permiso:

   Microsoft.Sql/servers/databases/read  Microsoft.Sql/servers/databases/write

Para cancelar una copia de la base de datos, debe tener los siguientes roles:

* Propietario de la suscripción o
* Rol de colaborador de SQL Server o
* Rol personalizado en las bases de datos de origen y de destino con el siguiente permiso:

   Microsoft.Sql/servers/databases/read  Microsoft.Sql/servers/databases/write

Para administrar la copia de la base de datos con Azure Portal, también necesitará los siguientes permisos:

   Microsoft.Resources/subscriptions/resources/read Microsoft.Resources/subscriptions/resources/write Microsoft.Resources/deployments/read Microsoft.Resources/deployments/write Microsoft.Resources/deployments/operationstatuses/read

Si quiere ver las operaciones en implementaciones en el grupo de recursos en el portal, las operaciones en varios proveedores de recursos, incluidas las operaciones de SQL, necesitará estos roles de Azure adicionales:

   Microsoft.Resources/subscriptions/resourcegroups/deployments/operations/read Microsoft.Resources/subscriptions/resourcegroups/deployments/operationstatuses/read

## <a name="resolve-logins"></a>Resolución de inicios de sesión

Después de que la nueva base de datos esté en línea en el servidor de destino, use la instrucción [ALTER USER](https://docs.microsoft.com/sql/t-sql/statements/alter-user-transact-sql?view=azuresqldb-current) para volver a asignar los usuarios de la nueva base de datos a inicios de sesión en el servidor de destino. Para resolver los usuarios huérfanos, consulte [Solucionar problemas de usuarios huérfanos (SQL Server)](https://docs.microsoft.com/sql/sql-server/failover-clusters/troubleshoot-orphaned-users-sql-server). Vea también [Administración de la seguridad de una base de datos de Azure SQL después de la recuperación ante desastres](active-geo-replication-security-configure.md).

Todos los usuarios de la nueva base de datos mantienen los permisos que tenían en la base de datos de origen. El usuario que inició la copia de la base de datos se convierte en el propietario de la nueva base de datos. Cuando la copia se realiza correctamente y antes de que se reasignen otros usuarios, solo el propietario de la base de datos puede iniciar sesión en la nueva base de datos.

Para más información sobre cómo administrar usuarios e inicios de sesión al copiar una base de datos a un servidor lógico diferente, vea [Administración de la seguridad de Azure SQL Database después de la recuperación ante desastres](active-geo-replication-security-configure.md).

## <a name="database-copy-errors"></a>Errores de copia de base de datos

Pueden encontrarse los siguientes errores al copiar una base de datos en Azure SQL Database. Para más información, vea [Copiar una base de datos de Azure SQL](database-copy.md).

| Código de error | severity | Descripción |
| ---:| ---:|:--- |
| 40635 |16 |El cliente con la dirección IP '%.&#x2a;ls' está deshabilitado temporalmente. |
| 40637 |16 |Crear copia de base de datos está deshabilitado actualmente. |
| 40561 |16 |Error al copiar la base de datos. No existe la base de datos de origen o de destino. |
| 40562 |16 |Error al copiar la base de datos. La base de datos de origen se ha quitado. |
| 40563 |16 |Error al copiar la base de datos. La base de datos de destino se ha quitado. |
| 40564 |16 |No se pudo copiar la base de datos debido a un error interno. Quite la base de datos de destino e inténtelo de nuevo. |
| 40565 |16 |Error al copiar la base de datos. No se permite más de una copia de base de datos simultánea para el mismo origen. Quite la base de datos de destino y vuelva a intentarlo más adelante. |
| 40566 |16 |No se pudo copiar la base de datos debido a un error interno. Quite la base de datos de destino e inténtelo de nuevo. |
| 40567 |16 |No se pudo copiar la base de datos debido a un error interno. Quite la base de datos de destino e inténtelo de nuevo. |
| 40568 |16 |Error al copiar la base de datos. La base de datos de origen ha dejado de estar disponible. Quite la base de datos de destino e inténtelo de nuevo. |
| 40569 |16 |Error al copiar la base de datos. La base de datos de destino ha dejado de estar disponible. Quite la base de datos de destino e inténtelo de nuevo. |
| 40570 |16 |No se pudo copiar la base de datos debido a un error interno. Quite la base de datos de destino y vuelva a intentarlo más adelante. |
| 40571 |16 |No se pudo copiar la base de datos debido a un error interno. Quite la base de datos de destino y vuelva a intentarlo más adelante. |

## <a name="next-steps"></a>Pasos siguientes

* Para más información sobre los inicios de sesión, vea [Administración de inicios de sesión](logins-create-manage.md) y [Administración de la seguridad de Azure SQL Database después de la recuperación ante desastres](active-geo-replication-security-configure.md).
* Para exportar una base de datos, consulte [Exportación de una base de datos a un archivo BACPAC](database-export.md).
