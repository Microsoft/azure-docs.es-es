---
title: Configuración de la replicación entre instancias administradas
titleSuffix: Azure SQL Managed Instance
description: En este tutorial aprenderá a configurar una replicación transaccional entre un publicador o distribuidor de Instancia administrada de Azure SQL y un suscriptor de Instancia administrada de SQL.
services: sql-database
ms.service: sql-managed-instance
ms.subservice: data-movement
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: conceptual
author: MashaMSFT
ms.author: ferno
ms.reviewer: mathoma
ms.date: 04/28/2020
ms.openlocfilehash: 114d4f41ad48af3d1e585fcb01eb0794a8e349b5
ms.sourcegitcommit: 4f1c7df04a03856a756856a75e033d90757bb635
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "87920122"
---
# <a name="tutorial-configure-replication-between-two-managed-instances"></a>Tutorial: Configuración de la replicación entre dos instancias administradas

[!INCLUDE[appliesto-sqlmi](../includes/appliesto-sqlmi.md)]

La replicación transaccional permite replicar datos de una base de datos a otra hospedada tanto en SQL Server como en [Azure SQL Managed Instance](sql-managed-instance-paas-overview.md). Instancia administrada de SQL puede ser un publicador, un distribuidor o un suscriptor en la topología de replicación. Consulte las [configuraciones de la replicación transaccional](replication-transactional-overview.md#common-configurations) para ver las opciones disponibles. 

La replicación transaccional está actualmente en versión preliminar pública para SQL Managed Instance. 

En este tutorial, aprenderá a:

> [!div class="checklist"]
>
> - Configurar una instancia administrada como publicador y distribuidor de replicación.
> - Configurar una instancia administrada como distribuidor de replicación.

![Replicación entre dos instancias administradas](./media/replication-between-two-instances-configure-tutorial/sqlmi-sqlmi-repl.png)

Este tutorial va dirigido a un público experimentado, así que se da por hecho que el usuario está familiarizado con la implementación y la conexión a ambas instancias administradas, y con máquinas virtuales con SQL Server en Azure. 


> [!NOTE]
> - En este artículo se describe el uso de la [replicación transaccional](/sql/relational-databases/replication/transactional/transactional-replication) en la Instancia administrada de Azure SQL Database. Esto no está relacionado con los [grupos de conmutación por error](../database/auto-failover-group-overview.md), una característica de Instancia administrada de Azure SQL que permite crear réplicas completas legibles de instancias individuales. Hay consideraciones adicionales al configurar la [replicación transaccional con grupos de conmutación por error](replication-transactional-overview.md#with-failover-groups).



## <a name="requirements"></a>Requisitos

Para configurar Instancia administrada de SQL de forma que funcione como un publicador o distribuidor, deben darse las siguientes condiciones:

- La instancia administrada del publicador se encuentra en la misma red virtual que el distribuidor y el suscriptor o se ha configurado un [emparejamiento de red virtual](../../virtual-network/tutorial-connect-virtual-networks-powershell.md) entre las redes virtuales de las tres entidades. 
- La conectividad usa la autenticación de SQL entre los participantes de la replicación.
- Un recurso compartido de cuenta de Azure Storage para el directorio de trabajo de replicación.
- El puerto 445 (salida TCP) está abierto en las reglas de seguridad del grupo de seguridad de red de la instancia administrada para tener acceso al recurso compartido de archivos de Azure.  Si aparece el error `failed to connect to azure storage \<storage account name> with os error 53`, deberá agregar una regla de salida al grupo de seguridad de red de la subred de Instancia administrada de SQL adecuada.

## <a name="1---create-a-resource-group"></a>1 - Creación de un grupo de recursos

Use [Azure Portal](https://portal.azure.com) para crear un grupo de recursos denominado `SQLMI-Repl`.  

## <a name="2---create-managed-instances"></a>2 - Creación de instancias administradas

Use [Azure Portal](https://portal.azure.com) para crear dos [Instancias administradas de SQL](instance-create-quickstart.md) en las mismas red virtual y subred. Por ejemplo, asigne un nombre a las dos instancias administradas:

- `sql-mi-pub` (junto con algunos caracteres para la selección aleatoria)
- `sql-mi-sub` (junto con algunos caracteres para la selección aleatoria)

También deberá [configurar una máquina virtual de Azure para la conexión](connect-vm-instance-configure.md) a las instancias administradas. 

## <a name="3---create-an-azure-storage-account"></a>3\. Creación de una cuenta de Azure Storage

[Cree una cuenta de Azure Storage](/azure/storage/common/storage-create-storage-account#create-a-storage-account) para el directorio de trabajo y, después, cree un [recurso compartido de archivos](../../storage/files/storage-how-to-create-file-share.md) dentro de la cuenta de almacenamiento. 

Copie la ruta de acceso del recurso compartido de archivos en el formato `\\storage-account-name.file.core.windows.net\file-share-name`.

Ejemplo: `\\replstorage.file.core.windows.net\replshare`

Copie las claves de acceso de almacenamiento en el formato `DefaultEndpointsProtocol=https;AccountName=<Storage-Account-Name>;AccountKey=****;EndpointSuffix=core.windows.net`.

Ejemplo: `DefaultEndpointsProtocol=https;AccountName=replstorage;AccountKey=dYT5hHZVu9aTgIteGfpYE64cfis0mpKTmmc8+EP53GxuRg6TCwe5eTYWrQM4AmQSG5lb3OBskhg==;EndpointSuffix=core.windows.net`

Para obtener más información, consulte [Administración de las claves de acceso de la cuenta de almacenamiento](../../storage/common/storage-account-keys-manage.md). 

## <a name="4---create-a-publisher-database"></a>4 - Creación de una base de datos del publicador

Conéctese a la instancia administrada de `sql-mi-pub` mediante SQL Server Management Studio y ejecute el siguiente código de Transact-SQL (T-SQL) para crear la base de datos del publicador:

```sql
USE [master]
GO

CREATE DATABASE [ReplTran_PUB]
GO

USE [ReplTran_PUB]
GO
CREATE TABLE ReplTest (
  ID INT NOT NULL PRIMARY KEY,
  c1 VARCHAR(100) NOT NULL,
  dt1 DATETIME NOT NULL DEFAULT getdate()
)
GO


USE [ReplTran_PUB]
GO

INSERT INTO ReplTest (ID, c1) VALUES (6, 'pub')
INSERT INTO ReplTest (ID, c1) VALUES (2, 'pub')
INSERT INTO ReplTest (ID, c1) VALUES (3, 'pub')
INSERT INTO ReplTest (ID, c1) VALUES (4, 'pub')
INSERT INTO ReplTest (ID, c1) VALUES (5, 'pub')
GO
SELECT * FROM ReplTest
GO
```

## <a name="5---create-a-subscriber-database"></a>5 - Creación de una base de datos del suscriptor

Conéctese a la instancia administrada de `sql-mi-sub` mediante SQL Server Management Studio y ejecute el siguiente código de T-SQL para crear la base de datos del suscriptor vacía:

```sql
USE [master]
GO

CREATE DATABASE [ReplTran_SUB]
GO

USE [ReplTran_SUB]
GO
CREATE TABLE ReplTest (
  ID INT NOT NULL PRIMARY KEY,
  c1 VARCHAR(100) NOT NULL,
  dt1 DATETIME NOT NULL DEFAULT getdate()
)
GO
```

## <a name="6---configure-distribution"></a>6 - Configuración de la distribución

Conéctese a la instancia administrada de `sql-mi-pub` mediante SQL Server Management Studio y ejecute el siguiente código de T-SQL para configurar la base de datos de distribución.

```sql
USE [master]
GO

EXEC sp_adddistributor @distributor = @@ServerName;
EXEC sp_adddistributiondb @database = N'distribution';
GO
```

## <a name="7---configure-publisher-to-use-distributor"></a>7 - Configuración del publicador para usar el distribuidor

En la Instancia administrada de SQL `sql-mi-pub` del publicador, cambie la ejecución de consultas al modo [SQLCMD](/sql/ssms/scripting/edit-sqlcmd-scripts-with-query-editor) y ejecute el código siguiente para registrar el distribuidor nuevo con el publicador.

```sql
:setvar username loginUsedToAccessSourceManagedInstance
:setvar password passwordUsedToAccessSourceManagedInstance
:setvar file_storage "\\storage-account-name.file.core.windows.net\file-share-name"
-- example: file_storage "\\replstorage.file.core.windows.net\replshare"
:setvar file_storage_key "DefaultEndpointsProtocol=https;AccountName=<Storage-Account-Name>;AccountKey=****;EndpointSuffix=core.windows.net"
-- example: file_storage_key "DefaultEndpointsProtocol=https;AccountName=replstorage;AccountKey=dYT5hHZVu9aTgIteGfpYE64cfis0mpKTmmc8+EP53GxuRg6TCwe5eTYWrQM4AmQSG5lb3OBskhg==;EndpointSuffix=core.windows.net"

USE [master]
EXEC sp_adddistpublisher
  @publisher = @@ServerName,
  @distribution_db = N'distribution',
  @security_mode = 0,
  @login = N'$(username)',
  @password = N'$(password)',
  @working_directory = N'$(file_storage)',
  @storage_connection_string = N'$(file_storage_key)'; -- Remove this parameter for on-premises publishers
```

   > [!NOTE]
   > Asegúrese de usar solo barras diagonales inversas (`\`) para el parámetro file_storage. El uso de una barra diagonal (`/`) puede producir un error al conectarse al recurso compartido de archivos.

Este script configura un publicador local en la instancia administrada, agrega un servidor vinculado y crea un conjunto de trabajos del Agente SQL Server.

## <a name="8---create-publication-and-subscriber"></a>8 - Creación de la publicación y el suscriptor

A través del modo [SQLCMD](/sql/ssms/scripting/edit-sqlcmd-scripts-with-query-editor), ejecute el siguiente script de T-SQL para habilitar la replicación de la base de datos y configure la replicación entre el publicador, el distribuidor y el suscriptor.

```sql
-- Set variables
:setvar username sourceLogin
:setvar password sourcePassword
:setvar source_db ReplTran_PUB
:setvar publication_name PublishData
:setvar object ReplTest
:setvar schema dbo
:setvar target_server "sql-mi-sub.wdec33262scj9dr27.database.windows.net"
:setvar target_username targetLogin
:setvar target_password targetPassword
:setvar target_db ReplTran_SUB

-- Enable replication for your source database
USE [$(source_db)]
EXEC sp_replicationdboption
  @dbname = N'$(source_db)',
  @optname = N'publish',
  @value = N'true';

-- Create your publication
EXEC sp_addpublication
  @publication = N'$(publication_name)',
  @status = N'active';


-- Configure your log reader agent
EXEC sp_changelogreader_agent
  @publisher_security_mode = 0,
  @publisher_login = N'$(username)',
  @publisher_password = N'$(password)',
  @job_login = N'$(username)',
  @job_password = N'$(password)';

-- Add the publication snapshot
EXEC sp_addpublication_snapshot
  @publication = N'$(publication_name)',
  @frequency_type = 1,
  @publisher_security_mode = 0,
  @publisher_login = N'$(username)',
  @publisher_password = N'$(password)',
  @job_login = N'$(username)',
  @job_password = N'$(password)';

-- Add the ReplTest table to the publication
EXEC sp_addarticle
  @publication = N'$(publication_name)',
  @type = N'logbased',
  @article = N'$(object)',
  @source_object = N'$(object)',
  @source_owner = N'$(schema)';

-- Add the subscriber
EXEC sp_addsubscription
  @publication = N'$(publication_name)',
  @subscriber = N'$(target_server)',
  @destination_db = N'$(target_db)',
  @subscription_type = N'Push';

-- Create the push subscription agent
EXEC sp_addpushsubscription_agent
  @publication = N'$(publication_name)',
  @subscriber = N'$(target_server)',
  @subscriber_db = N'$(target_db)',
  @subscriber_security_mode = 0,
  @subscriber_login = N'$(target_username)',
  @subscriber_password = N'$(target_password)',
  @job_login = N'$(username)',
  @job_password = N'$(password)';

-- Initialize the snapshot
EXEC sp_startpublication_snapshot
  @publication = N'$(publication_name)';
```

## <a name="9---modify-agent-parameters"></a>9 - Modificación de los parámetros del agente

Instancia administrada de Azure SQL está experimentando algunos problemas de back-end con la conectividad con los agentes de replicación. Mientras se aborda este problema, la solución consiste en aumentar el valor de tiempo de espera de inicio de sesión para los agentes de replicación.

Ejecute el siguiente comando de T-SQL en el publicador para aumentar el tiempo de espera de inicio de sesión:

```sql
-- Increase login timeout to 150s
update msdb..sysjobsteps set command = command + N' -LoginTimeout 150'
where subsystem in ('Distribution','LogReader','Snapshot') and command not like '%-LoginTimeout %'
```

Ejecute de nuevo el siguiente comando de T-SQL para volver a establecer el tiempo de espera de inicio de sesión en el valor predeterminado, en caso de que lo necesite:

```sql
-- Increase login timeout to 30
update msdb..sysjobsteps set command = command + N' -LoginTimeout 30'
where subsystem in ('Distribution','LogReader','Snapshot') and command not like '%-LoginTimeout %'
```

Reinicie los tres agentes para aplicar estos cambios.

## <a name="10---test-replication"></a>10 - Prueba de la replicación

Una vez que se ha configurado la replicación, puede probarla mediante la inserción de nuevos elementos en el publicador y la observación de los cambios que se propagan al suscriptor.

Ejecute el siguiente fragmento de código de T-SQL para ver las filas en el suscriptor:

```sql
select * from dbo.ReplTest
```

Ejecute el siguiente fragmento de código de T-SQL para insertar filas adicionales en el publicador y, a continuación, compruebe las filas de nuevo en el suscriptor.

```sql
INSERT INTO ReplTest (ID, c1) VALUES (15, 'pub')
```

## <a name="clean-up-resources"></a>Limpieza de recursos

Para anular la publicación, ejecute el siguiente comando de T-SQL:

```sql
-- Drops the publication
USE [ReplTran_PUB]
EXEC sp_droppublication @publication = N'PublishData'
GO
```

Para quitar la opción de replicación de la base de datos, ejecute el siguiente comando de T-SQL:

```sql
-- Disables publishing of the database
USE [ReplTran_PUB]
EXEC sp_removedbreplication
GO
```

Para deshabilitar la publicación y la distribución, ejecute el siguiente comando de T-SQL:

```sql
-- Drops the distributor
USE [master]
EXEC sp_dropdistributor @no_checks = 1
GO
```

Para limpiar los recursos de Azure, [elimine los recursos de Instancia administrada de SQL del grupo de recursos](../../azure-resource-manager/management/manage-resources-portal.md#delete-resources) y, después, elimine el grupo de recursos `SQLMI-Repl`. 

## <a name="next-steps"></a>Pasos siguientes

También puede obtener más información sobre la [replicación transaccional con Instancia administrada de Azure SQL](replication-transactional-overview.md) o aprender a configurar la replicación entre un [publicador/distribuidor de Instancia administrada de SQL y un suscriptor de SQL Server en una máquina virtual de Azure](replication-two-instances-and-sql-server-configure-tutorial.md). 
