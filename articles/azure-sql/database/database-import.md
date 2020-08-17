---
title: Importación de un archivo BACPAC para crear una base de datos en Azure SQL Database
description: Cree una base de datos en Azure SQL Database o en Instancia administrada de Azure SQL a partir de un archivo BACPAC.
services: sql-database
ms.service: sql-db-mi
ms.subservice: migrate
ms.custom: sqldbrb=1, devx-track-azurecli
ms.devlang: ''
ms.topic: conceptual
author: stevestein
ms.author: sstein
ms.reviewer: ''
ms.date: 06/20/2019
ms.openlocfilehash: a1e349d284d8f2c0c9c5194ac9f52534cd1ab973
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87503249"
---
# <a name="quickstart-import-a-bacpac-file-to-a-database-in-azure-sql-database-or-azure-sql-managed-instance"></a>Inicio rápido: Importación de un archivo BACPAC a una base de datos de Azure SQL Database o Instancia administrada de Azure SQL
[!INCLUDE[appliesto-sqldb-sqlmi](../includes/appliesto-sqldb-sqlmi.md)]

Puede importar una base de datos de SQL Server en Azure SQL Database o Instancia administrada de SQL mediante un archivo [BACPAC](https://docs.microsoft.com/sql/relational-databases/data-tier-applications/data-tier-applications#bacpac). Puede importar los datos de un archivo BACPAC almacenado en Azure Blob Storage (solo almacenamiento estándar) o desde el almacenamiento local en una ubicación local. Para maximizar la velocidad de importación al proporcionar más recursos y más rápidos, escale la base de datos a un nivel de servicio más alto y calcule el tamaño de proceso durante la importación. A continuación, puede reducir verticalmente cuando la importación se lleve a cabo de forma correcta.

> [!NOTE]
> El nivel de compatibilidad de la base de datos importada se basa en el nivel de compatibilidad de la base de datos de origen.

> [!IMPORTANT]
> Después de importar la base de datos, puede elegir usar la base de datos en el nivel de compatibilidad actual (nivel 100 para la base de datos AdventureWorks2008R2) o en un nivel superior. Para más información acerca de las implicaciones y las opciones para la utilización de una base de datos en un nivel de compatibilidad específico, consulte [ALTER DATABASE (Transact-SQL) Compatibility Level](https://docs.microsoft.com/sql/t-sql/statements/alter-database-transact-sql-compatibility-level) [Nivel de compatibilidad de ALTER DATABASE (Transact-SQL)]. Vea también [ALTER DATABASE SCOPED CONFIGURATION](https://docs.microsoft.com/sql/t-sql/statements/alter-database-scoped-configuration-transact-sql) para obtener información sobre los valores de nivel de base de datos adicionales relacionados con los niveles de compatibilidad.

## <a name="using-azure-portal"></a>Uso de Azure Portal

Vea este vídeo para ver cómo importar desde un archivo BACPAC en Azure Portal o siga leyendo a continuación:

> [!VIDEO https://channel9.msdn.com/Shows/Data-Exposed/Its-just-SQL-Restoring-a-database-to-Azure-SQL-DB-from-backup/player?WT.mc_id=dataexposed-c9-niner]

[Azure Portal](https://portal.azure.com)*solo* admite la creación de una base de datos única en Azure SQL Database y *solo* desde un archivo BACPAC almacenado en Azure Blob Storage.

Para migrar una base de datos a una [Instancia administrada de Azure SQL](../managed-instance/sql-managed-instance-paas-overview.md) a partir de un archivo BACPAC, use SQL Server Management Studio o SQLPackage. Actualmente no se admite el uso de Azure Portal o Azure PowerShell.

> [!NOTE]
> Los equipos que procesan las solicitudes de importación o exportación que se envían a través de Azure Portal o PowerShell deben almacenar el archivo BACPAC, así como los archivos temporales generados por Data-Tier Application Framework (DacFX). El espacio en disco necesario varía considerablemente entre las bases de datos del mismo tamaño y puede requerir un espacio en disco de hasta tres veces el tamaño de la base de datos. Los equipos que ejecutan la solicitud de importación o exportación solo tienen 450 GB de espacio en disco local. Como resultado, puede que es produzca el error `There is not enough space on the disk` en algunas solicitudes. En este caso, la solución alternativa es ejecutar sqlpackage.exe en un equipo con suficiente espacio en disco local. Se recomienda usar SqlPackage para importar o exportar bases de datos superiores a 150 GB para evitar este problema.

1. Para importar a partir de un archivo BACPAC en una nueva base de datos única mediante Azure Portal, abra la página del servidor que corresponda y, en la barra de herramientas, seleccione **Importar base de datos**.  

   ![Base de datos import1](./media/database-import/sql-server-import-database.png)

1. Seleccione la cuenta de almacenamiento y el contenedor para el archivo BACPAC y el archivo BACPAC desde el que se va a importar.

1. Especifique el tamaño de la nueva base de datos (por lo general, el mismo que el origen) y proporcione las credenciales del servidor SQL Server de destino. Para obtener una lista de valores posibles para una nueva base de datos de Azure SQL Database, vea [Creación de una base de datos](https://docs.microsoft.com/sql/t-sql/statements/create-database-transact-sql?view=azuresqldb-current).

   ![Base de datos import2](./media/database-import/sql-server-import-database-settings.png)

1. Haga clic en **OK**.

1. Para supervisar el progreso de una importación, abra la página del servidor de la base de datos y, en **Configuración**, seleccione **Historial de importación y exportación**. Cuando se realiza correctamente, la importación tiene el estado **Completada**.

   ![Estado de importación de la base de datos](./media/database-import/sql-server-import-database-history.png)

1. Para comprobar que la base de datos está activa en el servidor, seleccione **Bases de datos SQL** y compruebe que la base de datos nueva está **En línea**.

## <a name="using-sqlpackage"></a>Uso de SqlPackage

Para importar una base de datos de SQL Server mediante la utilidad de línea de comandos [SqlPackage](https://docs.microsoft.com/sql/tools/sqlpackage), consulte [Parámetros y propiedades de la importación](https://docs.microsoft.com/sql/tools/sqlpackage#import-parameters-and-properties). [SQL Server Management Studio](https://docs.microsoft.com/sql/ssms/download-sql-server-management-studio-ssms) y [SQL Server Data Tools para Visual Studio](https://msdn.microsoft.com/library/mt204009.aspx). También puede descargar la versión más reciente de [SqlPackage](https://www.microsoft.com/download/details.aspx?id=53876) desde el Centro de descarga de Microsoft.

Para el escalado y rendimiento, se recomienda usar SqlPackage en la mayoría de los entornos de producción en lugar de usar Azure Portal. Consulte cómo [migrar de SQL Server a Azure SQL Database con archivos BACPAC](https://blogs.msdn.microsoft.com/sqlcat/2016/10/20/migrating-from-sql-server-to-azure-sql-database-using-bacpac-files/) en el blog de Customer Advisory Team de SQL Server sobre cómo migrar usuarios con archivos `BACPAC`.

El comando SqlPackage siguiente importa la base de datos **AdventureWorks2008R2** desde el almacenamiento local a un servidor SQL lógico denominado **mynewserver20170403**. Crea una base de datos llamada **myMigratedDatabase** con un nivel de servicio **Premium** y un objetivo de servicio **P6**. Cambie estos valores según corresponda para su entorno.

```cmd
sqlpackage.exe /a:import /tcs:"Data Source=<serverName>.database.windows.net;Initial Catalog=<migratedDatabase>;User Id=<userId>;Password=<password>" /sf:AdventureWorks2008R2.bacpac /p:DatabaseEdition=Premium /p:DatabaseServiceObjective=P6
```

> [!IMPORTANT]
> Para conectarse a Azure SQL Database desde detrás de un firewall corporativo, el firewall debe tener abierto el puerto 1433. Para conectarse a Instancia administrada de SQL, debe tener una [conexión de punto a sitio](../managed-instance/point-to-site-p2s-configure.md) o una conexión de ExpressRoute.

Este ejemplo muestra cómo importar una base de datos con SqlPackage con Autenticación universal de Active Directory.

```cmd
sqlpackage.exe /a:Import /sf:testExport.bacpac /tdn:NewDacFX /tsn:apptestserver.database.windows.net /ua:True /tid:"apptest.onmicrosoft.com"
```

## <a name="using-powershell"></a>Usar PowerShell

> [!NOTE]
> [Instancia administrada de SQL](../managed-instance/sql-managed-instance-paas-overview.md) no admite actualmente la migración de una base de datos a una base de datos de instancia desde un archivo BACPAC mediante Azure PowerShell. Para importar en Instancia administrada de SQL, use SQL Server Management Studio o SQLPackage.

> [!NOTE]
> Los equipos que procesan las solicitudes de importación o exportación que se envían a través del portal o Powershell deben almacenar el archivo bacpac, así como los archivos temporales generados por Data-Tier Application Framework (DacFX). El espacio en disco necesario varía considerablemente entre las bases de datos del mismo tamaño y puede tardar hasta 3 veces el tamaño de la base de datos. Los equipos que ejecutan la solicitud de importación o exportación solo tienen 450 GB de espacio en disco local. Como resultado, en algunas solicitudes se puede producir el error "No hay suficiente espacio en el disco". En este caso, la solución alternativa es ejecutar sqlpackage.exe en un equipo con suficiente espacio en disco local. Al importar o exportar bases de datos superiores a 150 GB, use SqlPackage para evitar este problema.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

> [!IMPORTANT]
> El módulo de Azure Resource Manager (RM) para PowerShell todavía es compatible, pero todo el desarrollo futuro se realizará para el módulo Az.Sql. El módulo de AzureRM continuará recibiendo correcciones de errores hasta diciembre de 2020 como mínimo.  Los argumentos para los comandos del módulo Az y los módulos AzureRm son esencialmente idénticos. Para obtener más información sobre la compatibilidad, vea [Presentación del nuevo módulo Az de Azure PowerShell](/powershell/azure/new-azureps-module-az).

Use el cmdlet [New-AzSqlDatabaseImport](/powershell/module/az.sql/new-azsqldatabaseimport) para enviar una solicitud de importación de base de datos a Azure. En función del tamaño de la base de datos, la importación puede tardar en completarse.

```powershell
$importRequest = New-AzSqlDatabaseImport -ResourceGroupName "<resourceGroupName>" `
    -ServerName "<serverName>" -DatabaseName "<databaseName>" `
    -DatabaseMaxSizeBytes "<databaseSizeInBytes>" -StorageKeyType "StorageAccessKey" `
    -StorageKey $(Get-AzStorageAccountKey `
        -ResourceGroupName "<resourceGroupName>" -StorageAccountName "<storageAccountName>").Value[0] `
        -StorageUri "https://myStorageAccount.blob.core.windows.net/importsample/sample.bacpac" `
        -Edition "Standard" -ServiceObjectiveName "P6" `
        -AdministratorLogin "<userId>" `
        -AdministratorLoginPassword $(ConvertTo-SecureString -String "<password>" -AsPlainText -Force)
```

Puede usar el cmdlet [Get-AzSqlDatabaseImportExportStatus](/powershell/module/az.sql/get-azsqldatabaseimportexportstatus) para comprobar el progreso de la importación. Si el cmdlet se ejecuta inmediatamente después de la solicitud, por lo general devuelve `Status: InProgress`: La importación se completa cuando se ve `Status: Succeeded`.

```powershell
$importStatus = Get-AzSqlDatabaseImportExportStatus -OperationStatusLink $importRequest.OperationStatusLink

[Console]::Write("Importing")
while ($importStatus.Status -eq "InProgress") {
    $importStatus = Get-AzSqlDatabaseImportExportStatus -OperationStatusLink $importRequest.OperationStatusLink
    [Console]::Write(".")
    Start-Sleep -s 10
}

[Console]::WriteLine("")
$importStatus
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

Use el comando [az-sql-db-import](/cli/azure/sql/db#az-sql-db-import) para enviar una solicitud de importación de base de datos a Azure. En función del tamaño de la base de datos, la importación puede tardar en completarse.

```azurecli
# get the storage account key
az storage account keys list --resource-group "<resourceGroup>" --account-name "<storageAccount>"

az sql db import --resource-group "<resourceGroup>" --server "<server>" --name "<database>" `
    --storage-key-type "StorageAccessKey" --storage-key "<storageAccountKey>" `
    --storage-uri "https://myStorageAccount.blob.core.windows.net/importsample/sample.bacpac" `
    -u "<userId>" -p "<password>"
```

* * *

> [!TIP]
> Para ver otro ejemplo de script, consulte [Importación de una base de datos desde un archivo BACPAC](scripts/import-from-bacpac-powershell.md).

## <a name="limitations"></a>Limitaciones

- No se admite la importación a una base de datos que esté en un grupo elástico. Puede importar los datos en una base de datos única y después moverla a un grupo elástico.
- El servicio de importación y exportación no funciona cuando "Permitir el acceso a servicios de Azure" se establece en "DESACTIVADO". Sin embargo, puede solucionar el problema con la ejecución manual de sqlpackage.exe desde una máquina virtual de Azure o la realización de la exportación directamente en el código mediante la API de DACFx.

## <a name="import-using-wizards"></a>Importación mediante asistentes

También puede usar estos asistentes.

- [Asistente para importación de una aplicación de capa de datos en SQL Server Management Studio](https://docs.microsoft.com/sql/relational-databases/data-tier-applications/import-a-bacpac-file-to-create-a-new-user-database#using-the-import-data-tier-application-wizard).
- [Asistente para importación y exportación de SQL Server](https://docs.microsoft.com/sql/integration-services/import-export-data/start-the-sql-server-import-and-export-wizard)

## <a name="next-steps"></a>Pasos siguientes

- Para obtener información sobre cómo conectarse a una base de datos en Azure SQL Database y cómo realizar consultas en ella, vea [Inicio rápido: Azure SQL Database: uso de SQL Server Management Studio para conectarse a los datos y realizar consultas en ellos](connect-query-ssms.md).
- Consulte cómo [migrar de SQL Server a Azure SQL Database con archivos BACPAC](https://techcommunity.microsoft.com/t5/DataCAT/Migrating-from-SQL-Server-to-Azure-SQL-Database-using-Bacpac/ba-p/305407) en el blog de Customer Advisory Team de SQL Server.
- Para información sobre el proceso de migración de bases de datos SQL Server completo, incluidas las recomendaciones relacionadas con el rendimiento, consulte [Migración de una base de datos de SQL Server a Azure SQL Database](migrate-to-database-from-sql-server.md).
- Para obtener información sobre cómo administrar y compartir de forma segura claves de almacenamiento y firmas de acceso compartido, vea la [Guía de seguridad de Azure Storage](https://docs.microsoft.com/azure/storage/common/storage-security-guide).
