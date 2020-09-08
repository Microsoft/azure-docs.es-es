---
title: Configuración de la replicación transaccional entre Instancia administrada de Azure SQL y SQL Server
description: En este tutorial se configura la replicación entre una instancia administrada del publicador, una instancia administrada del distribuidor y un suscriptor de SQL Server en una máquina virtual de Azure, junto con los componentes de red necesarios, como el emparejamiento de VPN y la zona DNS privada.
services: sql-database
ms.service: sql-managed-instance
ms.subservice: security
ms.custom: sqldbrb=1
ms.topic: tutorial
author: MashaMSFT
ms.author: mathoma
ms.reviewer: carlrab
ms.date: 11/21/2019
ms.openlocfilehash: d89bc33b0ddd0793a3c55dbd64bef9678bd723e7
ms.sourcegitcommit: 4f1c7df04a03856a756856a75e033d90757bb635
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "87920150"
---
# <a name="tutorial-configure-transactional-replication-between-azure-sql-managed-instance-and-sql-server"></a>Tutorial: Configuración de la replicación transaccional entre Instancia administrada de Azure SQL y SQL Server
[!INCLUDE[appliesto-sqlmi](../includes/appliesto-sqlmi.md)]

La replicación transaccional permite replicar datos de una base de datos a otra hospedada tanto en SQL Server como en [Azure SQL Managed Instance](sql-managed-instance-paas-overview.md). Instancia administrada de SQL puede ser un publicador, un distribuidor o un suscriptor en la topología de replicación. Consulte las [configuraciones de la replicación transaccional](replication-transactional-overview.md#common-configurations) para ver las opciones disponibles. 

La replicación transaccional está actualmente en versión preliminar pública para SQL Managed Instance. 

En este tutorial, aprenderá a:

> [!div class="checklist"]
>
> - Configurar una instancia administrada como publicador de replicación.
> - Configurar una instancia administrada como distribuidor de replicación.
> - Configurar SQL Server como suscriptor.

![Replicación entre un publicador de instancia administrada, un distribuidor de instancia administrada y un suscriptor de SQL Server](./media/replication-two-instances-and-sql-server-configure-tutorial/sqlmi-to-sql-replication.png)

Este tutorial va dirigido a un público experimentado, así que se da por hecho que el usuario está familiarizado con la implementación y la conexión a ambas instancias administradas, y con máquinas virtuales con SQL Server en Azure. 


> [!NOTE]
> En este artículo se describe el uso de la [replicación transaccional](https://docs.microsoft.com/sql/relational-databases/replication/transactional/transactional-replication) en la Instancia administrada de Azure SQL Database. Esto no está relacionado con los [grupos de conmutación por error](https://docs.microsoft.com/azure/sql-database/sql-database-auto-failover-group), una característica de Instancia administrada de Azure SQL que permite crear réplicas completas legibles de instancias individuales. Hay consideraciones adicionales al configurar la [replicación transaccional con grupos de conmutación por error](replication-transactional-overview.md#with-failover-groups).

## <a name="prerequisites"></a>Requisitos previos

Para completar el tutorial, asegúrese de que cuenta con estos requisitos previos:

- Una [suscripción de Azure](https://azure.microsoft.com/free/).
- Experiencia con la implementación de dos instancias administradas dentro de la misma red virtual.
- Un suscriptor de SQL Server, ya sea local o en una máquina virtual de Azure. En este tutorial se usa una máquina virtual de Azure.  
- [SQL Server Management Studio (SSMS) 18.0 o versiones posteriores](/sql/ssms/download-sql-server-management-studio-ssms).
- La versión más reciente de [Azure PowerShell](/powershell/azure/install-az-ps?view=azps-1.7.0).
- Los puertos 445 y 1433 permiten el tráfico SQL en Azure Firewall y en el Firewall de Windows.

## <a name="1---create-the-resource-group"></a>1 - Creación del grupo de recursos

Use el siguiente fragmento de código de PowerShell para crear un grupo de recursos:

```powershell-interactive
# set variables
$ResourceGroupName = "SQLMI-Repl"
$Location = "East US 2"

# Create a new resource group
New-AzResourceGroup -Name  $ResourceGroupName -Location $Location
```

## <a name="2---create-two-managed-instances"></a>2 - Creación de dos instancias administradas

Cree dos instancias administradas dentro de este nuevo grupo de recursos mediante [Azure Portal](https://portal.azure.com).

- El nombre de la instancia administrada del publicador debe ser `sql-mi-publisher` (junto con algunos caracteres para la selección aleatoria) y el nombre de la red virtual debe ser `vnet-sql-mi-publisher`.
- El nombre de la instancia administrada del distribuidor debe ser `sql-mi-distributor` (junto con algunos caracteres para la selección aleatoria) y debe estar _en la misma red virtual que la instancia administrada del publicador_.

   ![Uso de la red virtual del publicador para el distribuidor](./media/replication-two-instances-and-sql-server-configure-tutorial/use-same-vnet-for-distributor.png)

Para más información sobre cómo crear una instancia administrada, consulte [Creación de una instancia administrada en el portal](instance-create-quickstart.md).

  > [!NOTE]
  > Por motivos de simplicidad, y dado que es la configuración más común, en este tutorial se sugiere colocar la instancia administrada del distribuidor dentro de la misma red virtual que la del publicador. Sin embargo, es posible crear el distribuidor en una red virtual distinta. Para ello, tendrá que configurar el emparejamiento de VPN entre las redes virtuales del publicador y del distribuidor y, luego, configurar el emparejamiento de VPN entre las redes virtuales del distribuidor y del suscriptor.

## <a name="3---create-a-sql-server-vm"></a>3 - Creación de una máquina virtual con SQL Server

Cree una máquina virtual con SQL Server en [Azure Portal](https://portal.azure.com). La máquina virtual con SQL Server debe tener las siguientes características:

- Nombre: `sql-vm-sub`
- Imagen: SQL Server 2016 o superior.
- Grupo de recursos: el mismo que el de la instancia administrada.
- Red virtual: `sql-vm-sub-vnet`.

Para más información sobre la implementación de una máquina virtual con SQL Server en Azure, consulte [Inicio rápido: Creación de una máquina virtual con SQL Server](../virtual-machines/windows/sql-vm-create-portal-quickstart.md).

## <a name="4---configure-vpn-peering"></a>4 - Configuración del emparejamiento de VPN

Configure el emparejamiento de VPN para permitir la comunicación entre la red virtual de las dos instancias administradas y la red virtual de SQL Server. Para ello, use este fragmento de código de PowerShell:

```powershell-interactive
# Set variables
$SubscriptionId = '<SubscriptionID>'
$resourceGroup = 'SQLMI-Repl'
$pubvNet = 'sql-mi-publisher-vnet'
$subvNet = 'sql-vm-sub-vnet'
$pubsubName = 'Pub-to-Sub-Peer'
$subpubName = 'Sub-to-Pub-Peer'

$virtualNetwork1 = Get-AzVirtualNetwork `
  -ResourceGroupName $resourceGroup `
  -Name $pubvNet

 $virtualNetwork2 = Get-AzVirtualNetwork `
  -ResourceGroupName $resourceGroup `
  -Name $subvNet  

# Configure VPN peering from publisher to subscriber
Add-AzVirtualNetworkPeering `
  -Name $pubsubName `
  -VirtualNetwork $virtualNetwork1 `
  -RemoteVirtualNetworkId $virtualNetwork2.Id

# Configure VPN peering from subscriber to publisher
Add-AzVirtualNetworkPeering `
  -Name $subpubName `
  -VirtualNetwork $virtualNetwork2 `
  -RemoteVirtualNetworkId $virtualNetwork1.Id

# Check status of peering on the publisher VNet; should say connected
Get-AzVirtualNetworkPeering `
 -ResourceGroupName $resourceGroup `
 -VirtualNetworkName $pubvNet `
 | Select PeeringState

# Check status of peering on the subscriber VNet; should say connected
Get-AzVirtualNetworkPeering `
 -ResourceGroupName $resourceGroup `
 -VirtualNetworkName $subvNet `
 | Select PeeringState

```

Una vez establecido el emparejamiento de VPN, inicie SQL Server Management Studio (SSMS) en la instancia de SQL Server y conéctese a ambas instancias administradas para probar la conectividad. Para más información sobre cómo conectarse a una instancia administrada mediante SSMS, consulte [Uso de SSMS para conectarse a Instancia administrada de SQL](point-to-site-p2s-configure.md#connect-with-ssms).

![Prueba de la conectividad a las instancias administradas](./media/replication-two-instances-and-sql-server-configure-tutorial/test-connectivity-to-mi.png)

## <a name="5---create-a-private-dns-zone"></a>5\. Creación de una zona DNS privada

Una zona DNS privada permite el enrutamiento DNS entre las instancias administradas y SQL Server.

### <a name="create-a-private-dns-zone"></a>Crear una zona DNS privada

1. Inicie sesión en el [Portal de Azure](https://portal.azure.com).
1. Seleccione **Crear un recurso** para crear un recurso de Azure.
1. Busque `private dns zone` en Azure Marketplace.
1. Elija el recurso **Zona DNS privada** publicado por Microsoft y, luego, seleccione **Crear** para crear la zona DNS.
1. Elija la suscripción y el grupo de recursos en la lista desplegable.
1. Proporcione un nombre arbitrario para la zona DNS, como `repldns.com`.

   ![Creación de una zona DNS privada](./media/replication-two-instances-and-sql-server-configure-tutorial/create-private-dns-zone.png)

1. Seleccione **Revisar + crear**. Revise los parámetros de la zona DNS privada y, luego, seleccione **Crear** para crear el recurso.

### <a name="create-an-a-record"></a>Creación de un registro A

1. Vaya a la nueva **zona DNS privada** y seleccione **Información general**.
1. Seleccione **+ Conjunto de registros** para crear un nuevo registro D.
1. Proporcione el nombre de la VM con SQL Server, así como la dirección IP interna privada.

   ![Configuración de un registro D](./media/replication-two-instances-and-sql-server-configure-tutorial/configure-a-record.png)

1. Seleccione **Aceptar** para crear el registro D.

### <a name="link-the-virtual-network"></a>Vincular la red virtual

1. Vaya a la nueva **zona DNS privada** y seleccione **Vínculos de red virtual**.
1. Seleccione **+Agregar**.
1. Proporcione un nombre para el vínculo, como `Pub-link`.
1. Seleccione la suscripción en la lista desplegable y, luego, seleccione la red virtual de la instancia administrada del publicador.
1. Active la casilla junto a **Habilitar el registro automático**.

   ![Creación de un vínculo de red virtual](./media/replication-two-instances-and-sql-server-configure-tutorial/configure-vnet-link.png)

1. Seleccione **Aceptar** para vincular la red virtual.
1. Repita estos pasos para agregar un vínculo para la red virtual del suscriptor, con un nombre como `Sub-link`.

## <a name="6---create-an-azure-storage-account"></a>6\. Creación de una cuenta de Azure Storage

[Cree una cuenta de Azure Storage](https://docs.microsoft.com/azure/storage/common/storage-create-storage-account#create-a-storage-account) para el directorio de trabajo y, a continuación, cree un [recurso compartido de archivos](../../storage/files/storage-how-to-create-file-share.md) dentro de la cuenta de almacenamiento.

Copie la ruta de acceso del recurso compartido de archivos en el formato `\\storage-account-name.file.core.windows.net\file-share-name`.

Ejemplo: `\\replstorage.file.core.windows.net\replshare`

Copie la cadena de conexión de la clave de acceso de almacenamiento en el formato `DefaultEndpointsProtocol=https;AccountName=<Storage-Account-Name>;AccountKey=****;EndpointSuffix=core.windows.net`.

Ejemplo: `DefaultEndpointsProtocol=https;AccountName=replstorage;AccountKey=dYT5hHZVu9aTgIteGfpYE64cfis0mpKTmmc8+EP53GxuRg6TCwe5eTYWrQM4AmQSG5lb3OBskhg==;EndpointSuffix=core.windows.net`

Para más información, consulte [Administración de las claves de acceso de la cuenta de almacenamiento](../../storage/common/storage-account-keys-manage.md).

## <a name="7---create-a-database"></a>7 - Creación de una base de datos

Cree una base de datos en la instancia administrada del publicador. Para hacerlo, siga estos pasos:

1. Inicie SQL Server Management Studio en SQL Server.
1. Conéctese a la instancia administrada `sql-mi-publisher`.
1. Abra una ventana **Nueva consulta** y ejecute la siguiente consulta de T-SQL para crear la base de datos.

```sql
-- Create the databases
USE [master]
GO

-- Drop database if it exists
IF EXISTS (SELECT * FROM sys.sysdatabases WHERE name = 'ReplTutorial')
BEGIN
    DROP DATABASE ReplTutorial
END
GO

-- Create new database
CREATE DATABASE [ReplTutorial]
GO

-- Create table
USE [ReplTutorial]
GO
CREATE TABLE ReplTest (
   ID INT NOT NULL PRIMARY KEY,
   c1 VARCHAR(100) NOT NULL,
   dt1 DATETIME NOT NULL DEFAULT getdate()
)
GO

-- Populate table with data
USE [ReplTutorial]
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

## <a name="8---configure-distribution"></a>8 - Configuración de la distribución

Una vez que ha establecido la conectividad y tiene una base de datos de ejemplo, puede configurar la distribución en la instancia administrada `sql-mi-distributor`. Para hacerlo, siga estos pasos:

1. Inicie SQL Server Management Studio en SQL Server.
1. Conéctese a la instancia administrada `sql-mi-distributor`.
1. Abra una ventana **Nueva consulta** y ejecute el siguiente código de Transact-SQL para configurar la distribución en la instancia administrada del distribuidor:

   ```sql
   EXEC sp_adddistributor @distributor = 'sql-mi-distributor.b6bf57.database.windows.net', @password = '<distributor_admin_password>'
   
   EXEC sp_adddistributiondb @database = N'distribution'
   
   EXEC sp_adddistpublisher @publisher = 'sql-mi-publisher.b6bf57.database.windows.net', -- primary publisher
        @distribution_db = N'distribution',
        @security_mode = 0,
        @login = N'azureuser',
        @password = N'<publisher_password>',
        @working_directory = N'\\replstorage.file.core.windows.net\replshare',
        @storage_connection_string = N'<storage_connection_string>'
        -- example: @storage_connection_string = N'DefaultEndpointsProtocol=https;AccountName=replstorage;AccountKey=dYT5hHZVu9aTgIteGfpYE64cfis0mpKTmmc8+EP53GxuRg6TCwe5eTYWrQM4AmQSG5lb3OBskhg==;EndpointSuffix=core.windows.net'

   ```

   > [!NOTE]
   > Asegúrese de usar solo barras diagonales inversas (`\`) para el parámetro @working_directory. El uso de una barra diagonal (`/`) puede producir un error al conectarse al recurso compartido de archivos.

1. Conéctese a la instancia administrada `sql-mi-publisher`.
1. Abra una ventana **Nueva consulta** y ejecute el siguiente código de Transact-SQL para registrar el distribuidor en el publicador:

   ```sql
   Use MASTER
   EXEC sys.sp_adddistributor @distributor = 'sql-mi-distributor.b6bf57.database.windows.net', @password = '<distributor_admin_password>'
   ```

## <a name="9---create-the-publication"></a>9 - Creación de la publicación

Una vez configurada la distribución, ahora puede crear la publicación. Para hacerlo, siga estos pasos:

1. Inicie SQL Server Management Studio en SQL Server.
1. Conéctese a la instancia administrada `sql-mi-publisher`.
1. En el **Explorador de objetos**, expanda el nodo **Replicación** y haga clic con el botón derecho en la carpeta **Local Publication** (Publicación local). Seleccione **Nueva publicación...** .
1. Seleccione **Siguiente** para dejar atrás la página principal.
1. En la página **Base de datos de publicación**, seleccione la base de datos `ReplTutorial` que creó anteriormente. Seleccione **Next** (Siguiente).
1. En la página **Tipo de publicación**, seleccione **Publicación transaccional**. Seleccione **Next** (Siguiente).
1. En la página **Artículos**, active la casilla situada junto a **Tablas**. Seleccione **Next** (Siguiente).
1. En la página **Filtrar filas de tabla**, seleccione **Siguiente** sin agregar ningún filtro.
1. En la página **Agente de instantáneas**, active la casilla junto a **Crear una instantánea inmediatamente y mantenerla disponible para inicializar suscripciones**. Seleccione **Next** (Siguiente).
1. En la página **Seguridad del agente**, seleccione **Configuración de seguridad**. Proporcione credenciales de inicio de sesión de SQL Server para usar con el Agente de instantáneas y para conectarse al publicador. Seleccione **Aceptar** para cerrar la página **Seguridad del Agente de instantáneas**. Seleccione **Next** (Siguiente).

   ![Configuración de la seguridad del Agente de instantáneas](./media/replication-two-instances-and-sql-server-configure-tutorial/snapshot-agent-security.png)

1. En la página **Acciones del asistente**, elija **Crear la publicación** y (opcionalmente) elija **Generar un archivo de script con los pasos para crear la publicación** si quiere guardar este script para más tarde.
1. En la página **Finalización del asistente**, asigne a la publicación el nombre `ReplTest` y seleccione **Siguiente** para crear la publicación.
1. Una vez creada la publicación, actualice el nodo **Replication** en el **Explorador de objetos** y expanda **Publicaciones locales** para ver la nueva publicación.

## <a name="10---create-the-subscription"></a>10 - Creación de la suscripción

Una vez creada la publicación, puede crear la suscripción. Para hacerlo, siga estos pasos:

1. Inicie SQL Server Management Studio en SQL Server.
1. Conéctese a la instancia administrada `sql-mi-publisher`.
1. Abra una ventana **Nueva consulta** y ejecute el siguiente código de Transact-SQL para agregar la suscripción y el agente de distribución. Use el DNS como parte del nombre del suscriptor.

```sql
use [ReplTutorial]
exec sp_addsubscription
@publication = N'ReplTest',
@subscriber = N'sql-vm-sub.repldns.com', -- include the DNS configured in the private DNS zone
@destination_db = N'ReplSub',
@subscription_type = N'Push',
@sync_type = N'automatic',
@article = N'all',
@update_mode = N'read only',
@subscriber_type = 0

exec sp_addpushsubscription_agent
@publication = N'ReplTest',
@subscriber = N'sql-vm-sub.repldns.com', -- include the DNS configured in the private DNS zone
@subscriber_db = N'ReplSub',
@job_login = N'azureuser',
@job_password = '<Complex Password>',
@subscriber_security_mode = 0,
@subscriber_login = N'azureuser',
@subscriber_password = '<Complex Password>',
@dts_package_location = N'Distributor'
GO
```

## <a name="11---test-replication"></a>11 - Prueba de la replicación

Una vez que se ha configurado la replicación, puede probarla mediante la inserción de nuevos elementos en el publicador y la observación de los cambios que se propagan al suscriptor.

Ejecute el siguiente fragmento de código de T-SQL para ver las filas en el suscriptor:

```sql
Use ReplSub
select * from dbo.ReplTest
```

Ejecute el siguiente fragmento de código de T-SQL para insertar filas adicionales en el publicador y, a continuación, compruebe las filas de nuevo en el suscriptor.

```sql
Use ReplTutorial
INSERT INTO ReplTest (ID, c1) VALUES (15, 'pub')
```

## <a name="clean-up-resources"></a>Limpieza de recursos

1. Vaya a su grupo de recursos en [Azure Portal](https://portal.azure.com).
1. Seleccione las instancias administradas y después seleccione **Eliminar**. Escriba `yes` en el cuadro de texto para confirmar que quiere eliminar el recurso y después seleccione **Eliminar**. Este proceso puede tardar algún tiempo en completarse en segundo plano y, hasta que finalice, no podrá eliminar el *clúster virtual* ni ningún otro recurso dependiente. Supervise la eliminación en la pestaña **Actividad** para confirmar que la instancia administrada se ha eliminado.
1. Una vez que se elimine la instancia administrada, puede eliminar el *clúster virtual* si lo selecciona en el grupo de recursos y, después, elige **Eliminar**. Escriba `yes` en el cuadro de texto para confirmar que quiere eliminar el recurso y después seleccione **Eliminar**.
1. Elimine todos los recursos restantes. Escriba `yes` en el cuadro de texto para confirmar que quiere eliminar el recurso y después seleccione **Eliminar**.
1. Para eliminar el grupo de recursos, seleccione **Eliminar grupo de recursos**, escriba el nombre del grupo de recursos (`myResourceGroup`) y, a continuación, seleccione **Eliminar**.

## <a name="known-errors"></a>Errores conocidos

### <a name="windows-logins-are-not-supported"></a>No se admiten inicios de sesión de Windows

`Exception Message: Windows logins are not supported in this version of SQL Server.`

El agente se configuró con un inicio de sesión de Windows y lo que necesita es usar un inicio de sesión de SQL Server. Use la página **Seguridad del agente** de **Propiedades de la publicación** para cambiar las credenciales de inicio de sesión a un inicio de sesión de SQL Server.

### <a name="failed-to-connect-to-azure-storage"></a>No se pudo conectar a Azure Storage

`Connecting to Azure Files Storage '\\replstorage.file.core.windows.net\replshare' Failed to connect to Azure Storage '' with OS error: 53.`

2019-11-19 02:21:05.07 Se obtuvo la cadena de conexión de Azure Storage para replstorage 2019-11-19 02:21:05.07 Conectando a Azure Files Storage "\\replstorage.file.core.windows.net\replshare" 2019-11-19 02:21:31.21 No se pudo conectar a Azure Storage" con el error del SO: 53.

Puede que el puerto 445 esté cerrado en Azure Firewall, en el Firewall de Windows o en ambos.

`Connecting to Azure Files Storage '\\replstorage.file.core.windows.net\replshare' Failed to connect to Azure Storage '' with OS error: 55.`

El uso de una barra diagonal en lugar de una barra diagonal inversa en la ruta de acceso del archivo del el recurso compartido de archivos puede producir este error.
  
  - Esto es correcto: `\\replstorage.file.core.windows.net\replshare`
  - Esto puede producir un error 55 del sistema operativo: `'\\replstorage.file.core.windows.net/replshare'`

### <a name="could-not-connect-to-subscriber"></a>No se pudo conectar al suscriptor

`The process could not connect to Subscriber 'SQL-VM-SUB`
`Could not open a connection to SQL Server [53].`
`A network-related or instance-specific error has occurred while establishing a connection to SQL Server. Server is not found or not accessible. Check if instance name is correct and if SQL Server is configured to allow remote connections.`

Posibles soluciones:

- Asegúrese de que el puerto 1433 esté abierto.
- Asegúrese de que TCP/IP esté habilitado en el suscriptor.
- Confirme que se usó el nombre DNS al crear el suscriptor.
- Compruebe que las redes virtuales estén vinculadas correctamente en la zona DNS privada.
- Compruebe que el registro D esté configurado correctamente.
- Compruebe que el emparejamiento de VPN esté configurado correctamente.

### <a name="no-publications-to-which-you-can-subscribe"></a>No hay publicaciones a las que pueda suscribirse

Al agregar una nueva suscripción mediante el **Asistente para nueva suscripción**, en la página **Publicación**, es posible que no se muestren las bases de datos y las publicaciones como opciones disponibles, y podría aparecer el mensaje de error siguiente:

`There are no publications to which you can subscribe, either because this server has no publications or because you do not have sufficient privileges to access the publications.`

Aunque puede que este mensaje de error sea verdad y no haya realmente publicaciones disponibles en el publicador al que se ha conectado, o que no tenga permisos suficientes, este error también podría deberse a una versión anterior de SQL Server Management Studio. Intente actualizar a SQL Server Management Studio 18.0 o superior para que esto se descarte como causa principal.

## <a name="next-steps"></a>Pasos siguientes

### <a name="enable-security-features"></a>Habilitar características de seguridad

Consulte en el artículo [¿Qué es Instancia administrada de Azure SQL?](sql-managed-instance-paas-overview.md#advanced-security-and-compliance) la lista completa de las formas de proteger la base de datos. Se abordan las características de seguridad siguientes:

- [Auditoría de Instancia administrada de SQL](auditing-configure.md)
- [Always Encrypted](/sql/relational-databases/security/encryption/always-encrypted-database-engine)
- [Detección de amenazas](threat-detection-configure.md)
- [Enmascaramiento de datos dinámicos](/sql/relational-databases/security/dynamic-data-masking)
- [Seguridad de nivel de fila](/sql/relational-databases/security/row-level-security)
- [Cifrado de datos transparente (TDE)](https://docs.microsoft.com/sql/relational-databases/security/encryption/transparent-data-encryption-azure-sql)

### <a name="sql-managed-instance-capabilities"></a>Funcionalidades de Instancia administrada de SQL

Para obtener una información general completa de las funcionalidades de una instancia administrada, consulte:

> [!div class="nextstepaction"]
> [Funcionalidades de Instancia administrada de SQL](sql-managed-instance-paas-overview.md)
