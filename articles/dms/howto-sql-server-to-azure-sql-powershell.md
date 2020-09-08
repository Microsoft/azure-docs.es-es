---
title: 'PowerShell: Migración de SQL Server a SQL Database'
titleSuffix: Azure Database Migration Service
description: Obtenga información sobre cómo migrar una base de datos de SQL Server a Azure SQL Database mediante Azure PowerShell con Azure Database Migration Service.
services: database-migration
author: pochiraju
ms.author: rajpo
manager: craigg
ms.reviewer: craigg
ms.service: dms
ms.workload: data-services
ms.custom: seo-lt-2019, devx-track-azurepowershell
ms.topic: article
ms.date: 02/20/2020
ms.openlocfilehash: e83f69b2a46b54f92bbfe35ac96f870edf5cb807
ms.sourcegitcommit: 656c0c38cf550327a9ee10cc936029378bc7b5a2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89078034"
---
# <a name="migrate-a-sql-server-database-to-azure-sql-database-using-azure-powershell"></a>Migración de una base de datos de SQL Server a Azure SQL Database mediante Azure PowerShell

En este artículo, migrará la base de datos **Adventureworks2012** restaurada en una instancia local de SQL Server 2016 o posterior a Azure SQL mediante Microsoft Azure PowerShell. Puede migrar bases de datos desde una instancia de SQL Server a Azure SQL Database mediante el módulo `Az.DataMigration` en Microsoft Azure PowerShell.

En este artículo aprenderá a:
> [!div class="checklist"]
>
> * Cree un grupo de recursos.
> * Crear una instancia de Azure Database Migration Service.
> * Crear un proyecto de migración en una instancia de Azure Database Migration Service.
> * Ejecutar la migración.

## <a name="prerequisites"></a>Requisitos previos

Para completar estos pasos, necesitará lo siguiente:

* [SQL Server 2016 o posterior](https://www.microsoft.com/sql-server/sql-server-downloads) (cualquier edición).
* Para habilitar el protocolo TCP/IP, que se deshabilita de forma predeterminada con la instalación de SQL Server Express. Habilite el protocolo TCP/IP siguiendo el artículo [Habilitar o deshabilitar un protocolo de red de servidor](https://docs.microsoft.com/sql/database-engine/configure-windows/enable-or-disable-a-server-network-protocol#SSMSProcedure).
* Configurar [Firewall de Windows para el acceso al motor de base de datos](https://docs.microsoft.com/sql/database-engine/configure-windows/configure-a-windows-firewall-for-database-engine-access).
* Una base de datos de Azure SQL. Puede crear una instancia de Azure SQL Database mediante la información del artículo [Creación de una base de datos de Azure SQL Database en Azure Portal](https://docs.microsoft.com/azure/sql-database/sql-database-get-started-portal).
* [Data Migration Assistant](https://www.microsoft.com/download/details.aspx?id=53595) (versión 3.3 o posterior).
* Haber creado una instancia de Microsoft Azure Virtual Network con el modelo de implementación de Azure Resource Manager, que proporciona a Azure Database Migration Service conectividad de sitio a sitio a los servidores de origen locales a través de [ExpressRoute](https://docs.microsoft.com/azure/expressroute/expressroute-introduction) o [VPN](https://docs.microsoft.com/azure/vpn-gateway/vpn-gateway-about-vpngateways).
* Haber completado la evaluación de su base de datos local y la migración de esquemas con Data Migration Assistant, tal y como se describe en el artículo [Realizar una evaluación de migración de SQL Server con Data Migration Assistant](https://docs.microsoft.com/sql/dma/dma-assesssqlonprem).
* Descargar e instalar el módulo Az.DataMigration de la Galería de PowerShell con el [cmdlet Install-Module de PowerShell](https://docs.microsoft.com/powershell/module/powershellget/Install-Module?view=powershell-5.1); asegurarse de abrir la ventana Comandos de PowerShell ejecutándola como administrador.
* Asegurarse de que las credenciales usadas para conectarse a la instancia de SQL Server de origen tenga el permiso [CONTROL SERVER](https://docs.microsoft.com/sql/t-sql/statements/grant-server-permissions-transact-sql).
* Asegurarse de que las credenciales usadas para conectarse a la instancia de Azure SQL DB de destino tenga el permiso CONTROL DATABASE en las bases de datos de Azure SQL de destino.
* Suscripción a Azure. Si no tiene una, cree una cuenta [gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="log-in-to-your-microsoft-azure-subscription"></a>Inicio de sesión en la suscripción de Microsoft Azure

Utilice las instrucciones que aparecen en el artículo [Inicio de sesión con Azure PowerShell](https://docs.microsoft.com/powershell/azure/authenticate-azureps) para iniciar sesión en su suscripción de Azure mediante PowerShell.

## <a name="create-a-resource-group"></a>Crear un grupo de recursos

Un grupo de recursos de Azure es un contenedor lógico en el que se implementan y se administran los recursos de Azure. Cree un grupo de recursos para poder crear una máquina virtual.

Cree un grupo de recursos de Azure con el comando [New-AzResourceGroup](https://docs.microsoft.com/powershell/module/az.resources/new-azresourcegroup).

En el ejemplo siguiente, se crea un grupo de recursos denominado *myResourceGroup* en la región *EastUS*.

```powershell
New-AzResourceGroup -ResourceGroupName myResourceGroup -Location EastUS
```

## <a name="create-an-instance-of-azure-database-migration-service"></a>Creación de una instancia de Azure Database Migration Service

Puede crear una instancia nueva de Azure Database Migration Service mediante el cmdlet `New-AzDataMigrationService`. Este cmdlet espera los siguientes parámetros requeridos:

* *Nombre del grupo de recursos de Azure*. Puede usar el comando [New-AzResourceGroup](https://docs.microsoft.com/powershell/module/az.resources/new-azresourcegroup) para crear el grupo de recursos de Azure como se mostró anteriormente y proporcionar su nombre como un parámetro.
* *Nombre del servicio*. Cadena que se corresponde con el nombre de servicio único para Azure Database Migration Service. 
* *Ubicación*. Especifica la ubicación del servicio. Especifique una ubicación de centro de datos de Azure, por ejemplo, Oeste de EE. UU. o Sudeste de Asia.
* *SKU*. Este parámetro corresponde al nombre de SKU de DMS. El nombre de SKU admitido actualmente es *GeneralPurpose_4vCores*.
* *Identificador de subred virtual*. Puede usar el cmdlet [New-AzVirtualNetworkSubnetConfig](https://docs.microsoft.com/powershell/module/az.network/new-azvirtualnetworksubnetconfig) para crear una subred. 

En el ejemplo siguiente se crea un servicio de nombre *MyDMS* en el grupo de recursos *MyDMSResourceGroup* que se encuentra en la región *Este de EE. UU.* con una red virtual denominada *MyVNET* y una subred llamada *MySubnet*.

```powershell
 $vNet = Get-AzVirtualNetwork -ResourceGroupName MyDMSResourceGroup -Name MyVNET

$vSubNet = Get-AzVirtualNetworkSubnetConfig -VirtualNetwork $vNet -Name MySubnet

$service = New-AzDms -ResourceGroupName myResourceGroup `
  -ServiceName MyDMS `
  -Location EastUS `
  -Sku Basic_2vCores `  
  -VirtualSubnetId $vSubNet.Id`
```

## <a name="create-a-migration-project"></a>Creación de un proyecto de migración

Cree un proyecto de migración después de crear una instancia de Azure Database Migration Service. Un proyecto de Azure Database Migration Service requiere información de conexión para las instancias de origen y de destino, así como una lista de bases de datos que desee migrar como parte del proyecto.

### <a name="create-a-database-connection-info-object-for-the-source-and-target-connections"></a>Creación de un objeto de información de conexión de base de datos para las conexiones de origen y de destino

Puede crear un objeto de información de conexión de base de datos mediante el cmdlet `New-AzDmsConnInfo`. Este cmdlet espera los siguientes parámetros:

* *ServerType*. El tipo de conexión de base de datos solicitado, por ejemplo, SQL, Oracle o MySQL. Utilice SQL para SQL Server y Azure SQL.
* *DataSource*. Nombre o dirección IP de una instancia de SQL Server o Azure SQL Database.
* *AuthType*. El tipo de autenticación para la conexión, que puede ser SqlAuthentication o WindowsAuthentication.
* El parámetro *TrustServerCertificate* establece un valor que indica si el canal se cifra mientras se omite recorrer la cadena de certificados para validar la confianza. El valor puede ser "true" o "false".

En el ejemplo siguiente se crea un objeto Connection Info para el servidor SQL Server de origen denominado MySourceSQLServer con autenticación de SQL:

```powershell
$sourceConnInfo = New-AzDmsConnInfo -ServerType SQL `
  -DataSource MySourceSQLServer `
  -AuthType SqlAuthentication `
  -TrustServerCertificate:$true
```

En el ejemplo siguiente se muestra la creación de la información de conexión para el servidor denominado SQLAzureTarget mediante la autenticación de SQL:

```powershell
$targetConnInfo = New-AzDmsConnInfo -ServerType SQL `
  -DataSource "sqlazuretarget.database.windows.net" `
  -AuthType SqlAuthentication `
  -TrustServerCertificate:$false
```

### <a name="provide-databases-for-the-migration-project"></a>Ofrecimiento de bases de datos para el proyecto de migración

Cree una lista de objetos `AzDataMigrationDatabaseInfo` que especifique las bases de datos como parte del proyecto de Azure Database Migration que puede proporcionarse como parámetro para la creación del proyecto. Puede usarse el cmdlet `New-AzDataMigrationDatabaseInfo` para crear AzDataMigrationDatabaseInfo. 

En el siguiente ejemplo se crea el proyecto `AzDataMigrationDatabaseInfo` en la base de datos **AdventureWorks2016** y se agrega a la lista que se va a proporcionar como parámetro para la creación del proyecto.

```powershell
$dbInfo1 = New-AzDataMigrationDatabaseInfo -SourceDatabaseName AdventureWorks2016
$dbList = @($dbInfo1)
```

### <a name="create-a-project-object"></a>Creación de un objeto del proyecto

Por último, puede crear el proyecto de Azure Database Migration denominado *MyDMSProject* ubicado en el *Este de EE. UU.* con `New-AzDataMigrationProject` y agregar las conexiones de origen y de destino creadas anteriormente y la lista de bases de datos que migrar.

```powershell
$project = New-AzDataMigrationProject -ResourceGroupName myResourceGroup `
  -ServiceName $service.Name `
  -ProjectName MyDMSProject `
  -Location EastUS `
  -SourceType SQL `
  -TargetType SQLDB `
  -SourceConnection $sourceConnInfo `
  -TargetConnection $targetConnInfo `
  -DatabaseInfo $dbList
```

## <a name="create-and-start-a-migration-task"></a>Creación e inicio de una tarea de migración

Por último, cree e inicie la tarea de Azure Database Migration. La tarea de Azure Database Migration requiere información de credenciales de conexión de origen y destino, y una lista de las tablas de base de datos que se van a migrar, además de la información ya proporcionada con el proyecto creado como requisito previo.

### <a name="create-credential-parameters-for-source-and-target"></a>Creación de parámetros de credenciales de origen y de destino

Las credenciales de seguridad de conexión pueden crearse como un objeto [PSCredential](https://docs.microsoft.com/dotnet/api/system.management.automation.pscredential?redirectedfrom=MSDN&view=powershellsdk-1.1.0).

En el ejemplo siguiente se muestra la creación de objetos *PSCredential* para las conexiones de origen y destino proporcionando contraseñas como variables de cadena *$sourcePassword* y *$targetPassword*.

```powershell
$secpasswd = ConvertTo-SecureString -String $sourcePassword -AsPlainText -Force
$sourceCred = New-Object System.Management.Automation.PSCredential ($sourceUserName, $secpasswd)
$secpasswd = ConvertTo-SecureString -String $targetPassword -AsPlainText -Force
$targetCred = New-Object System.Management.Automation.PSCredential ($targetUserName, $secpasswd)
```

### <a name="create-a-table-map-and-select-source-and-target-parameters-for-migration"></a>Crear una asignación de tabla y selección de parámetros de origen y de destino para la migración

Otro parámetro necesario para la migración es la asignación de tablas del origen al destino que se va a migrar. Cree un diccionario de tablas que proporcione una asignación entre las tablas de origen y de destino para la migración. En el ejemplo siguiente se muestra la asignación entre las tablas de origen y de destino de esquema de Recursos Humanos para la base de datos de AdventureWorks 2016.

```powershell
$tableMap = New-Object 'system.collections.generic.dictionary[string,string]'
$tableMap.Add("HumanResources.Department", "HumanResources.Department")
$tableMap.Add("HumanResources.Employee","HumanResources.Employee")
$tableMap.Add("HumanResources.EmployeeDepartmentHistory","HumanResources.EmployeeDepartmentHistory")
$tableMap.Add("HumanResources.EmployeePayHistory","HumanResources.EmployeePayHistory")
$tableMap.Add("HumanResources.JobCandidate","HumanResources.JobCandidate")
$tableMap.Add("HumanResources.Shift","HumanResources.Shift")
```

El paso siguiente consiste en seleccionar las bases de datos de origen y de destino y proporcionar asignaciones de tabla que migrar como un parámetro con el cmdlet `New-AzDmsSelectedDB`, tal y como se muestra en el ejemplo siguiente:

```powershell
$selectedDbs = New-AzDmsSelectedDB -MigrateSqlServerSqlDb -Name AdventureWorks2016 `
  -TargetDatabaseName AdventureWorks2016 `
  -TableMap $tableMap
```

### <a name="create-the-migration-task-and-start-it"></a>Creación e inicio de la tarea de migración

Utilice el cmdlet `New-AzDataMigrationTask` para crear e iniciar una tarea de migración. Este cmdlet espera los siguientes parámetros:

* *TaskType*. Se espera el tipo de tarea de migración que se creará para SQL Server, en el tipo de migración *MigrateSqlServerSqlDb* de Azure SQL Database. 
* *Nombre del grupo de recursos*. Nombre del grupo de recursos de Azure en el que se crea la tarea.
* *ServiceName*. Instancia de Azure Database Migration Service en la que crear la tarea.
* *ProjectName*. Nombre del proyecto de Azure Database Migration Service en el que se va a crear la tarea. 
* *TaskName*. Nombre de la tarea que se va a crear. 
* *SourceConnection*. Objeto AzDmsConnInfo que representa la conexión de origen de SQL Server.
* *TargetConnection*. Objeto AzDmsConnInfo que representa la conexión de destino de Azure SQL Database.
* *SourceCred*. Objeto [PSCredential](https://docs.microsoft.com/dotnet/api/system.management.automation.pscredential?redirectedfrom=MSDN&view=powershellsdk-1.1.0) para la conexión al servidor de origen.
* *TargetCred*. Objeto [PSCredential](https://docs.microsoft.com/dotnet/api/system.management.automation.pscredential?redirectedfrom=MSDN&view=powershellsdk-1.1.0) para la conexión al servidor de destino.
* *SelectedDatabase*. Objeto AzDataMigrationSelectedDB que representa la asignación de base de datos de origen y de destino.
* *SchemaValidation*. (opcional, parámetro de modificador) Tras la migración, realiza una comparación de la información del esquema entre el origen y el destino.
* *DataIntegrityValidation*. (opcional, parámetro de modificador) Tras la migración, realiza una validación de la integridad de los datos basada en la suma de comprobación entre el origen y el destino.
* *QueryAnalysisValidation*. (opcional, parámetro de modificador) Tras la migración, realiza un análisis de consultas rápido e inteligente recuperando las consultas de la base de datos de origen y las ejecuta en el destino.

En el siguiente ejemplo se crea y se inicia una tarea de migración denominada myDMSTask:

```powershell
$migTask = New-AzDataMigrationTask -TaskType MigrateSqlServerSqlDb `
  -ResourceGroupName myResourceGroup `
  -ServiceName $service.Name `
  -ProjectName $project.Name `
  -TaskName myDMSTask `
  -SourceConnection $sourceConnInfo `
  -SourceCred $sourceCred `
  -TargetConnection $targetConnInfo `
  -TargetCred $targetCred `
  -SelectedDatabase  $selectedDbs `
```

En el ejemplo siguiente se crea e inicia la misma tarea de migración como antes, pero también realiza las tres validaciones:

```powershell
$migTask = New-AzDataMigrationTask -TaskType MigrateSqlServerSqlDb `
  -ResourceGroupName myResourceGroup `
  -ServiceName $service.Name `
  -ProjectName $project.Name `
  -TaskName myDMSTask `
  -SourceConnection $sourceConnInfo `
  -SourceCred $sourceCred `
  -TargetConnection $targetConnInfo `
  -TargetCred $targetCred `
  -SelectedDatabase  $selectedDbs `
  -SchemaValidation `
  -DataIntegrityValidation `
  -QueryAnalysisValidation `
```

## <a name="monitor-the-migration"></a>Supervisión de la migración

Puede supervisar la tarea de migración que se ejecuta mediante la consulta de la propiedad de estado de la tarea como se muestra en el siguiente ejemplo:

```powershell
if (($mytask.ProjectTask.Properties.State -eq "Running") -or ($mytask.ProjectTask.Properties.State -eq "Queued"))
{
  write-host "migration task running"
}
```

## <a name="deleting-the-dms-instance"></a>Eliminación de la instancia DMS

Una vez completada la migración, puede eliminar la instancia de Azure DMS:

```powershell
Remove-AzDms -ResourceGroupName myResourceGroup -ServiceName MyDMS
```

## <a name="next-step"></a>Paso siguiente

* Revise las directrices de migración en la [Guía de migración de bases de datos](https://datamigration.microsoft.com/).
