---
title: Exportación de una base de datos de Azure SQL única o agrupada a un archivo BACPAC | Microsoft Docs
description: Exportación de una base de datos de Azure SQL Database a un archivo BACPAC mediante Azure Portal
services: sql-database
ms.service: sql-database
ms.subservice: data-movement
ms.custom: ''
ms.devlang: ''
ms.topic: conceptual
author: stevestein
ms.author: sstein
ms.reviewer: carlrab
manager: craigg
ms.date: 03/11/2019
ms.openlocfilehash: c87979760730cbe8f57d8f65463c94d08888aa2b
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65762754"
---
# <a name="export-an-azure-sql-database-to-a-bacpac-file"></a>Exportación de una base de datos de Azure SQL Database a un archivo BACPAC

Cuando necesite exportar una base de datos para archivar o migrar a otra plataforma, puede exportar el esquema de base de datos y los datos a un archivo [BACPAC](https://msdn.microsoft.com/library/ee210546.aspx#Anchor_4). Un archivo BACPAC es un archivo ZIP con una extensión de BACPAC que contiene los metadatos y los datos de una base de datos de SQL Server. Un archivo BACPAC se puede almacenar en Azure Blob Storage o en una ubicación local y, luego, volver a importarlo en Azure SQL Database o en una instalación local de SQL Server.

## <a name="considerations-when-exporting-an-azure-sql-database"></a>Consideraciones al exportar una base de datos de Azure SQL

- Para que una exportación sea transaccionalmente coherente, debe asegurarse de que no haya ninguna actividad de escritura durante la exportación, o bien de exportar desde una [copia transaccionalmente coherente](sql-database-copy.md) de su base de datos de Azure SQL.
- Si va a exportar a Blob Storage, el tamaño máximo de un archivo BACPAC es 200 GB. Para archivar un archivo BACPAC de mayor tamaño, exporte al almacenamiento local.
- No se admite la exportación de un archivo BACPAC en Azure Premium Storage con los métodos que se describen en este artículo.
- Almacenamiento detrás de un firewall no se admite actualmente.
- Si la operación de exportación desde Azure SQL Database demora más de 20 horas, es posible que se cancele. Para aumentar el rendimiento durante la exportación, puede hacer lo siguiente:

  - Aumentar temporalmente el tamaño de proceso.
  - Detener toda actividad de lectura y escritura durante la exportación.
  - Use un [índice agrupado](https://msdn.microsoft.com/library/ms190457.aspx) con valores distintos de NULL en todas las tablas de gran tamaño. Sin índices agrupados, la exportación podría no producirse si tarda más de 6-12 horas. Esto se debe a que el servicio de exportación necesita completar el recorrido de tabla para tratar de exportar toda la tabla. Una buena forma de determinar si las tablas están optimizadas para la exportación es ejecutar **DBCC SHOW_STATISTICS** y asegurarse de que *RANGE_HI_KEY* no es NULL y su valor tiene buena distribución. Para obtener más información, consulte [DBCC SHOW_STATISTICS](https://msdn.microsoft.com/library/ms174384.aspx).

> [!NOTE]
> Los BACPAC no están diseñados para usarse en operaciones de copia de seguridad y restauración. Azure SQL Database crea automáticamente copias de seguridad para cada base de datos de usuario. Para detalles, consulte los artículos de [introducción a la continuidad empresarial](sql-database-business-continuity.md) y sobre las [copias de seguridad de SQL Database](sql-database-automated-backups.md).

## <a name="export-to-a-bacpac-file-using-the-azure-portal"></a>Exportar a un archivo BACPAC mediante Azure Portal

> [!NOTE]
> [Una instancia administrada](sql-database-managed-instance.md) no admite actualmente la exportación de una base de datos a un archivo BACPAC mediante Azure Portal. Para exportar una instancia administrada a un archivo BACPAC, utilice SQL Server Management Studio o SQLPackage.

1. Para exportar una base de datos mediante [Azure Portal](https://portal.azure.com), abra la página de la base de datos y haga clic en **Exportar** en la barra de herramientas.

   ![Exportación de base de datos](./media/sql-database-export/database-export1.png)

2. Especifique el nombre de archivo BACPAC, seleccione una cuenta de Azure Storage existente y el contenedor para la exportación, y escriba las credenciales correspondientes para acceder a la base de datos de origen.

    ![Exportación de base de datos](./media/sql-database-export/database-export2.png)

3. Haga clic en **OK**.

4. Para supervisar el progreso de la operación de exportación, abra la página del servidor de SQL Database que contiene la base de datos que se va a exportar. Vaya a **Configuración** y haga clic en **Historial de importación y exportación**.

   ![historial de exportación](./media/sql-database-export/export-history.png)

## <a name="export-to-a-bacpac-file-using-the-sqlpackage-utility"></a>Exportar a un archivo BACPAC mediante la utilidad SQLPackage

Para exportar una base de datos SQL mediante la utilidad de línea de comandos [SqlPackage](https://docs.microsoft.com/sql/tools/sqlpackage), consulte [Parámetros y propiedades de la exportación](https://docs.microsoft.com/sql/tools/sqlpackage#export-parameters-and-properties). La utilidad SQLPackage incluye las versiones más recientes de [SQL Server Management Studio](https://msdn.microsoft.com/library/mt238290.aspx) y [SQL Server Data Tools para Visual Studio](https://msdn.microsoft.com/library/mt204009.aspx), o bien puede descargar la versión más reciente de [SqlPackage](https://www.microsoft.com/download/details.aspx?id=53876) directamente desde el Centro de descarga de Microsoft.

Se recomienda usar la utilidad SQLPackage para la escala y el rendimiento en la mayoría de los entornos de producción. Consulte cómo [migrar de SQL Server a Azure SQL Database con archivos BACPAC](https://blogs.msdn.microsoft.com/sqlcat/20../../migrating-from-sql-server-to-azure-sql-database-using-bacpac-files/) en el blog de Customer Advisory Team de SQL Server.

Este ejemplo muestra cómo exportar una base de datos con SqlPackage.exe con Autenticación universal de Active Directory:

```cmd
SqlPackage.exe /a:Export /tf:testExport.bacpac /scs:"Data Source=apptestserver.database.windows.net;Initial Catalog=MyDB;" /ua:True /tid:"apptest.onmicrosoft.com"
```

## <a name="export-to-a-bacpac-file-using-sql-server-management-studio-ssms"></a>Exportar a un archivo BACPAC mediante SQL Server Management Studio (SSMS)

Las versiones más recientes de SQL Server Management Studio proporcionan un asistente para exportar una base de datos de Azure SQL a un archivo BACPAC. Consulte [Export a Data-tier Application](https://docs.microsoft.com/sql/relational-databases/data-tier-applications/export-a-data-tier-application) (Exportación de una aplicación de la capa de datos).

## <a name="export-to-a-bacpac-file-using-powershell"></a>Exportar a un archivo BACPAC mediante PowerShell

> [!NOTE]
> [Una instancia administrada](sql-database-managed-instance.md) no admite actualmente la exportación de una base de datos a un archivo BACPAC mediante Azure PowerShell. Para exportar una instancia administrada a un archivo BACPAC, utilice SQL Server Management Studio o SQLPackage.

Use la [New AzSqlDatabaseExport](/powershell/module/az.sql/new-azsqldatabaseexport) cmdlet para enviar una solicitud de base de datos de exportación para el servicio de Azure SQL Database. Según el tamaño de la base de datos, la operación de exportación puede tardar algún tiempo en completarse.

```powershell
$exportRequest = New-AzSqlDatabaseExport -ResourceGroupName $ResourceGroupName -ServerName $ServerName `
  -DatabaseName $DatabaseName -StorageKeytype $StorageKeytype -StorageKey $StorageKey -StorageUri $BacpacUri `
  -AdministratorLogin $creds.UserName -AdministratorLoginPassword $creds.Password
```

Para comprobar el estado de la solicitud de exportación, use el [Get AzSqlDatabaseImportExportStatus](/powershell/module/az.sql/get-azsqldatabaseimportexportstatus) cmdlet. Si se ejecuta inmediatamente después de la solicitud, normalmente devuelve **Estado: En curso**. Cuando vea **Estado: Correcto**, la exportación se ha terminado.

```powershell
$exportStatus = Get-AzSqlDatabaseImportExportStatus -OperationStatusLink $exportRequest.OperationStatusLink
[Console]::Write("Exporting")
while ($exportStatus.Status -eq "InProgress")
{
    Start-Sleep -s 10
    $exportStatus = Get-AzSqlDatabaseImportExportStatus -OperationStatusLink $exportRequest.OperationStatusLink
    [Console]::Write(".")
}
[Console]::WriteLine("")
$exportStatus
```

## <a name="next-steps"></a>Pasos siguientes

- Para información sobre la retención a largo plazo de la copia de seguridad de bases de datos únicas y agrupadas como alternativa a la exportación de una base de datos para archivarla, consulte el artículo sobre la [Retención a largo plazo de copias de seguridad](sql-database-long-term-retention.md). Puede usar trabajos del Agente SQL para programar [copias de seguridad de base de datos de solo copia](https://docs.microsoft.com/sql/relational-databases/backup-restore/copy-only-backups-sql-server) como alternativa a la retención a largo plazo de las copias de seguridad.
- Consulte cómo [migrar de SQL Server a Azure SQL Database con archivos BACPAC](https://blogs.msdn.microsoft.com/sqlcat/2016/10/20/migrating-from-sql-server-to-azure-sql-database-using-bacpac-files/) en el blog de Customer Advisory Team de SQL Server.
- Para aprender a importar un BACPAC a una base de datos de SQL Server, consulte [Importación de un BACPAC en una base de datos de SQL Server](https://msdn.microsoft.com/library/hh710052.aspx).
- Para aprender a exportar un BACPAC desde una base de datos de SQL Server, consulte [Exportar una aplicación de capa de datos](https://docs.microsoft.com/sql/relational-databases/data-tier-applications/export-a-data-tier-application).
- Para más información acerca de cómo usar el servicio de migración de datos para migrar una base de datos, consulte [Migración de SQL Server a Azure SQL Database sin conexión mediante DMS](../dms/tutorial-sql-server-to-azure-sql.md).
- Si va a exportar desde SQL Server como paso previo a la migración a Azure SQL Database, consulte [Migración de una base de datos de SQL Server a Azure SQL Database](sql-database-single-database-migrate.md).
- Para obtener información sobre cómo administrar y compartir de forma segura claves de almacenamiento y firmas de acceso compartido, vea la [Guía de seguridad de Azure Storage](https://docs.microsoft.com/azure/storage/common/storage-security-guide).
