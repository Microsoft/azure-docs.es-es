---
title: Importación de un archivo BACPAC para crear una base de datos de Azure SQL | Microsoft Docs
description: Cree una base de datos de Azure SQL Database nueva mediante la importación de archivo BACPAC.
services: sql-database
ms.service: sql-database
ms.subservice: migration
ms.custom: ''
ms.devlang: ''
ms.topic: conceptual
author: stevestein
ms.author: sstein
ms.reviewer: ''
manager: craigg
ms.date: 03/12/2019
ms.openlocfilehash: 98b316f8a9c1c8ceba91870af4ff67b1aa854a9b
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65785338"
---
# <a name="quickstart-import-a-bacpac-file-to-a-database-in-azure-sql-database"></a>Inicio rápido: Importación de un archivo BACPAC en una base de datos de Azure SQL Database

Puede importar una base de datos de SQL Server a una base de datos de Azure SQL Database mediante un archivo [BACPAC](https://docs.microsoft.com/sql/relational-databases/data-tier-applications/data-tier-applications#bacpac). Puede importar los datos de un archivo `BACPAC` almacenados en Azure Blob Storage (solo almacenamiento estándar) o desde el almacenamiento local en una ubicación local. Para maximizar la velocidad de importación al proporcionar más recursos y más rápidos, escale la base de datos a un nivel de servicio más alto y calcule el tamaño de proceso durante la importación. A continuación, puede reducir verticalmente cuando la importación se lleve a cabo de forma correcta.

> [!NOTE]
> El nivel de compatibilidad de la base de datos importada se basa en el nivel de compatibilidad de la base de datos de origen.
> [!IMPORTANT]
> Después de importar la base de datos, puede elegir usar la base de datos en el nivel de compatibilidad actual (nivel 100 para la base de datos AdventureWorks2008R2) o en un nivel superior. Para más información acerca de las implicaciones y las opciones para la utilización de una base de datos en un nivel de compatibilidad específico, consulte [ALTER DATABASE (Transact-SQL) Compatibility Level](https://docs.microsoft.com/sql/t-sql/statements/alter-database-transact-sql-compatibility-level) [Nivel de compatibilidad de ALTER DATABASE (Transact-SQL)]. Vea también [ALTER DATABASE SCOPED CONFIGURATION](https://docs.microsoft.com/sql/t-sql/statements/alter-database-scoped-configuration-transact-sql) para obtener información sobre los valores de nivel de base de datos adicionales relacionados con los niveles de compatibilidad.

## <a name="import-from-a-bacpac-file-in-the-azure-portal"></a>Importación de un archivo BACPAC en Azure Portal

[Azure Portal](https://portal.azure.com) *solo* admite la creación de una base de datos única en Azure SQL Database y *solo* desde un archivo BACPAC almacenado en Azure Blob Storage.

> [!NOTE]
> [Una instancia administrada](sql-database-managed-instance.md) no admite actualmente la migración de una base de datos a una base de datos de instancias desde un archivo BACPAC mediante Azure Portal. Para importar en una instancia administrada, utilice SQL Server Management Studio o SQLPackage.

1. Para importar a partir de un archivo BACPAC en una base de datos única mediante Azure Portal, abra la página del servidor de bases de datos que corresponda y, en la barra de herramientas, seleccione **Importar base de datos**.  

   ![Base de datos import1](./media/sql-database-import/import1.png)

2. Seleccione la cuenta de almacenamiento y el contenedor para el archivo BACPAC y el archivo BACPAC desde el que se va a importar.
3. Especifique el tamaño de la nueva base de datos (por lo general, el mismo que el origen) y proporcione las credenciales del servidor SQL Server de destino. Para obtener una lista de valores posibles para una nueva base de datos de Azure SQL, consulte [Creación de una base de datos](https://docs.microsoft.com/sql/t-sql/statements/create-database-transact-sql?view=azuresqldb-current).

   ![Base de datos import2](./media/sql-database-import/import2.png)

4. Haga clic en **OK**.

5. Para supervisar el progreso de una importación, abra la página del servidor de la base de datos y, en **Configuración**, seleccione **Historial de importación y exportación**. Cuando se realiza correctamente, la importación tiene el estado **Completada**.

   ![Estado de importación de la base de datos](./media/sql-database-import/import-status.png)

6. Para comprobar que la base de datos está activa en el servidor de bases de datos, seleccione **Bases de datos SQL** y compruebe que la base de datos nueva está **En línea**.

## <a name="import-from-a-bacpac-file-using-sqlpackage"></a>Importación de un archivo BACPAC mediante SqlPackage

Para importar una base de datos de SQL Server mediante la utilidad de línea de comandos [SqlPackage](https://docs.microsoft.com/sql/tools/sqlpackage), consulte [Parámetros y propiedades de la importación](https://docs.microsoft.com/sql/tools/sqlpackage#import-parameters-and-properties). SqlPackage incluye las versiones más recientes de [SQL Server Management Studio](https://docs.microsoft.com/sql/ssms/download-sql-server-management-studio-ssms) y [SQL Server Data Tools para Visual Studio](https://msdn.microsoft.com/library/mt204009.aspx). También puede descargar la versión más reciente de [SqlPackage](https://www.microsoft.com/download/details.aspx?id=53876) desde el Centro de descarga de Microsoft.

Para el escalado y rendimiento, se recomienda usar SqlPackage en la mayoría de los entornos de producción en lugar de usar Azure Portal. Consulte cómo [migrar de SQL Server a Azure SQL Database con archivos BACPAC](https://blogs.msdn.microsoft.com/sqlcat/2016/10/20/migrating-from-sql-server-to-azure-sql-database-using-bacpac-files/) en el blog de Customer Advisory Team de SQL Server sobre cómo migrar usuarios con archivos `BACPAC`.

Para escalado y rendimiento, se recomienda usar SqlPackage en la mayoría de los entornos de producción. Consulte cómo [migrar de SQL Server a Azure SQL Database con archivos BACPAC](https://blogs.msdn.microsoft.com/sqlcat/20../../migrating-from-sql-server-to-azure-sql-database-using-bacpac-files/) en el blog de Customer Advisory Team de SQL Server.

El comando SqlPackage siguiente importa la base de datos **AdventureWorks2008R2** desde el almacenamiento local a un servidor de Azure SQL Database denominado **mynewserver20170403**. Crea una base de datos llamada **myMigratedDatabase** con un nivel de servicio **Premium** y un objetivo de servicio **P6**. Cambie estos valores según corresponda para su entorno.

```cmd
SqlPackage.exe /a:import /tcs:"Data Source=mynewserver20170403.database.windows.net;Initial Catalog=myMigratedDatabase;User Id=<your_server_admin_account_user_id>;Password=<your_server_admin_account_password>" /sf:AdventureWorks2008R2.bacpac /p:DatabaseEdition=Premium /p:DatabaseServiceObjective=P6
```

> [!IMPORTANT]
> Para conectarse a un servidor de SQL Database que administra una base de datos única desde detrás de un firewall corporativo, este debe tener abierto el puerto 1433. Para conectarse a una instancia administrada, debe tener una [conexión de punto a sitio](sql-database-managed-instance-configure-p2s.md) o una conexión de ExpressRoute.
>

Este ejemplo muestra cómo importar una base de datos con SqlPackage con Autenticación universal de Active Directory.

```cmd
SqlPackage.exe /a:Import /sf:testExport.bacpac /tdn:NewDacFX /tsn:apptestserver.database.windows.net /ua:True /tid:"apptest.onmicrosoft.com"
```

## <a name="import-into-a-single-database-from-a-bacpac-file-using-powershell"></a>Importación en una base de datos única a partir de un archivo BACPAC mediante PowerShell

> [!NOTE]
> [Una instancia administrada](sql-database-managed-instance.md) no admite actualmente la migración de una base de datos en una base de datos de instancia de un archivo BACPAC mediante Azure PowerShell. Para importar en una instancia administrada, utilice SQL Server Management Studio o SQLPackage.


[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]
> [!IMPORTANT]
> El módulo de PowerShell de Azure Resource Manager es compatible aún con Azure SQL Database, pero todo el desarrollo futuro es para el módulo Az.Sql. Para estos cmdlets, consulte [AzureRM.Sql](https://docs.microsoft.com/powershell/module/AzureRM.Sql/). Los argumentos para los comandos en el módulo de Az y en los módulos AzureRm son esencialmente idénticos.

Use la [New AzSqlDatabaseImport](/powershell/module/az.sql/new-azsqldatabaseimport) cmdlet para enviar una solicitud de base de datos de importación para el servicio de Azure SQL Database. En función del tamaño de la base de datos, la importación puede tardar en completarse.

 ```powershell
 $importRequest = New-AzSqlDatabaseImport 
    -ResourceGroupName "<your_resource_group>" `
    -ServerName "<your_server>" `
    -DatabaseName "<your_database>" `
    -DatabaseMaxSizeBytes "<database_size_in_bytes>" `
    -StorageKeyType "StorageAccessKey" `
    -StorageKey $(Get-AzStorageAccountKey -ResourceGroupName "<your_resource_group>" -StorageAccountName "<your_storage_account").Value[0] `
    -StorageUri "https://myStorageAccount.blob.core.windows.net/importsample/sample.bacpac" `
    -Edition "Standard" `
    -ServiceObjectiveName "P6" `
    -AdministratorLogin "<your_server_admin_account_user_id>" `
    -AdministratorLoginPassword $(ConvertTo-SecureString -String "<your_server_admin_account_password>" -AsPlainText -Force)

 ```

 Puede usar el [Get AzSqlDatabaseImportExportStatus](/powershell/module/az.sql/get-azsqldatabaseimportexportstatus) cmdlet comprobación del progreso de la importación. Si el cmdlet se ejecuta inmediatamente después de la solicitud, por lo general devuelve **Estado: En curso**. La importación se completa cuando se ve **Estado: Correcto**.

```powershell
$importStatus = Get-AzSqlDatabaseImportExportStatus -OperationStatusLink $importRequest.OperationStatusLink
[Console]::Write("Importing")
while ($importStatus.Status -eq "InProgress")
{
    $importStatus = Get-AzSqlDatabaseImportExportStatus -OperationStatusLink $importRequest.OperationStatusLink
    [Console]::Write(".")
    Start-Sleep -s 10
}
[Console]::WriteLine("")
$importStatus
```

> [!TIP]
> Para ver otro ejemplo de script, consulte [Importación de una base de datos desde un archivo BACPAC](scripts/sql-database-import-from-bacpac-powershell.md).

## <a name="limitations"></a>Limitaciones

No se admite la importación a una base de datos que esté en un grupo elástico. Puede importar los datos en una base de datos única y después moverla a un grupo elástico.

## <a name="import-using-wizards"></a>Importación mediante asistentes

También puede usar estos asistentes.

- [Asistente para importación de una aplicación de capa de datos en SQL Server Management Studio](https://docs.microsoft.com/sql/relational-databases/data-tier-applications/import-a-bacpac-file-to-create-a-new-user-database#using-the-import-data-tier-application-wizard).
- [Asistente para importación y exportación de SQL Server](https://docs.microsoft.com/sql/integration-services/import-export-data/start-the-sql-server-import-and-export-wizard)

## <a name="next-steps"></a>Pasos siguientes

- Para información sobre cómo conectarse a una base de datos SQL importada y cómo realizar consultas en ella, lea [Inicio rápido: Azure SQL Database: uso de SQL Server Management Studio para conectarse a los datos y realizar consultas en ellos](sql-database-connect-query-ssms.md).
- Consulte cómo [migrar de SQL Server a Azure SQL Database con archivos BACPAC](https://techcommunity.microsoft.com/t5/DataCAT/Migrating-from-SQL-Server-to-Azure-SQL-Database-using-Bacpac/ba-p/305407) en el blog de Customer Advisory Team de SQL Server.
- Para información sobre el proceso de migración de bases de datos SQL Server completo, incluidas las recomendaciones relacionadas con el rendimiento, consulte [Migración de una base de datos de SQL Server a Azure SQL Database](sql-database-single-database-migrate.md).
- Para obtener información sobre cómo administrar y compartir de forma segura claves de almacenamiento y firmas de acceso compartido, vea la [Guía de seguridad de Azure Storage](https://docs.microsoft.com/azure/storage/common/storage-security-guide).
