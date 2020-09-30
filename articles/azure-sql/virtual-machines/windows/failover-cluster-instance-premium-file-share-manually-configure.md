---
title: Creación de una FCI con un recurso compartido de archivos Premium
description: Use un recurso compartido de archivos Premium (PFS) para crear una instancia de clúster de conmutación por error (FCI) con SQL Server en Azure Virtual Machines.
services: virtual-machines
documentationCenter: na
author: MashaMSFT
editor: monicar
tags: azure-service-management
ms.service: virtual-machines-sql
ms.custom: na
ms.topic: article
ms.tgt_pltfrm: vm-windows-sql-server
ms.workload: iaas-sql-server
ms.date: 06/18/2020
ms.author: mathoma
ms.openlocfilehash: cbc6b2af98905a09324a58c92cafca0075d8a01d
ms.sourcegitcommit: 420c30c760caf5742ba2e71f18cfd7649d1ead8a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89055148"
---
# <a name="create-an-fci-with-a-premium-file-share-sql-server-on-azure-vms"></a>Creación de una FCI con un recurso compartido de archivos Premium (SQL Server en VM de Azure)
[!INCLUDE[appliesto-sqlvm](../../includes/appliesto-sqlvm.md)]

En este artículo se explica cómo crear una instancia de clúster de conmutación por error (FCI) con SQL Server en Azure Virtual Machines (VM) con un [recurso compartido de archivos Premium](../../../storage/files/storage-how-to-create-premium-fileshare.md).

Los recursos compartidos de archivos Premium son recursos compartidos de archivos de baja latencia constante con respaldo de Espacios de almacenamiento directo (SSD) que son totalmente compatibles para utilizarlos con instancias del clúster de conmutación por error en SQL Server 2012 o versiones posteriores en Windows Server 2012 o versiones posteriores. Los recursos compartidos de archivos Premium ofrecen mayor flexibilidad, lo que le permite cambiar el tamaño y escalar el recurso compartido de archivos sin tiempo de inactividad.

Para obtener más información, consulte información general de [FCI con SQL Server en VM de Azure](failover-cluster-instance-overview.md) y [procedimientos recomendados del clúster](hadr-cluster-best-practices.md). 

## <a name="prerequisites"></a>Requisitos previos

Antes de completar las instrucciones de este artículo, ya debe tener:

- Suscripción a Azure.
- Una cuenta que tenga permisos para crear objetos en máquinas virtuales de Azure y en Active Directory.
- [Dos o más instancias de Microsoft Azure Virtual Machines preparadas](failover-cluster-instance-prepare-vm.md) en un [conjunto de disponibilidad](../../../virtual-machines/windows/tutorial-availability-sets.md#create-an-availability-set) o [zonas de disponibilidad](../../../virtual-machines/windows/create-portal-availability-zone.md#confirm-zone-for-managed-disk-and-ip-address) diferentes.
- Un [recurso compartido de archivos premium](../../../storage/files/storage-how-to-create-premium-fileshare.md) que se usará como unidad en clúster, en función de la cuota de almacenamiento de la base de datos de los archivos de datos.
- La versión más reciente de [PowerShell](/powershell/azure/install-az-ps?view=azps-4.2.0). 

## <a name="mount-premium-file-share"></a>Montaje del recurso compartido de archivos Premium

1. Inicie sesión en [Azure Portal](https://portal.azure.com). y vaya a la cuenta de almacenamiento.
1. Vaya a **Recursos compartidos de archivos** en **Servicio Archivo** y, a continuación, seleccione el recurso compartido de archivos Premium que quiere usar para el almacenamiento de SQL.
1. Seleccione **Conectar** para mostrar la cadena de conexión del recurso compartido de archivos.
1. En la lista desplegable, seleccione la letra de la unidad que desea usar y, a continuación, copie ambos bloques de código en un bloc de notas.

   :::image type="content" source="media/failover-cluster-instance-premium-file-share-manually-configure/premium-file-storage-commands.png" alt-text="Copia de ambos comandos de PowerShell desde el portal de conexión del recurso compartido de archivos":::

1. Aplique el protocolo de escritorio remoto (RDP) para conectarse a la VM con SQL Server con la cuenta que la FCI de SQL Server usará para la cuenta de servicio.
1. Abra una consola de comandos de PowerShell de administración.
1. Ejecute los comandos que guardó anteriormente cuando trabajaba en el portal.
1. Vaya hasta el recurso compartido con el explorador de archivos o el cuadro de diálogo **Ejecutar** (seleccione Windows + R). Use la ruta de acceso a la red `\\storageaccountname.file.core.windows.net\filesharename`. Por ejemplo: `\\sqlvmstorageaccount.file.core.windows.net\sqlpremiumfileshare`

1. Cree al menos una carpeta en el recurso compartido de archivos recién conectado para colocar los archivos de datos de SQL.
1. Repita estos pasos en cada VM con SQL Server que participará en el clúster.

  > [!IMPORTANT]
  > - Considere la posibilidad de usar un recurso compartido de archivos independiente para los archivos de copia de seguridad a fin de ahorrar la capacidad de operaciones de entrada/salida por segundo (IOPS) y espacio de este recurso compartido para usarla para el archivo de registro y de datos. Puede usar un recurso compartido de archivos Premium o Estándar para los archivos de copia de seguridad.
  > - Si tiene Windows 2012 R2 o versiones anteriores, siga estos mismos pasos para montar el recurso compartido de archivos que va a usar como testigo de recurso compartido de archivos. 
  > 


## <a name="add-windows-cluster-feature"></a>Adición de la característica del clúster de Windows

1. Conéctese a la primera máquina virtual con RDP mediante una cuenta de dominio que sea miembro de los administradores locales y tenga permisos para crear objetos en Active Directory. Utilice esta cuenta para el resto de la configuración.

1. [Agregue Clústeres de conmutación por error a todas las máquinas virtuales](availability-group-manually-configure-prerequisites-tutorial.md#add-failover-clustering-features-to-both-sql-server-vms).

   Para instalar Clústeres de conmutación por error desde la interfaz de usuario, realice lo siguiente en las dos máquinas virtuales:
   1. En el **Administrador del servidor**, seleccione **Administrar** y, a continuación, seleccione **Agregar roles y características**.
   1. En el Asistente para **agregar roles y características**, seleccione **Siguiente** hasta llegar a **Seleccionar características**.
   1. En **Seleccionar características**, seleccione **Clúster de conmutación por error**. Incluya todas las características y herramientas de administración requeridas. 
   1. Seleccione **Agregar características**.
   1. Seleccione **Siguiente** y, después, **Finalizar** para instalar las características.

   Para instalar Clústeres de conmutación por error con PowerShell, ejecute el siguiente script en una sesión de PowerShell de administrador de una de las máquinas virtuales:

   ```powershell
   $nodes = ("<node1>","<node2>")
   Invoke-Command  $nodes {Install-WindowsFeature Failover-Clustering -IncludeAllSubFeature -IncludeManagementTools}
   ```

## <a name="validate-cluster"></a>Validar clúster

Valide el clúster en la interfaz de usuario o con PowerShell.

Para validar el clúster con la interfaz de usuario, realice lo siguiente en una de las máquinas virtuales:

1. En **Administrador del servidor**, seleccione **Herramientas** y, después, seleccione **Administrador de clústeres de conmutación por error**.
1. En **Administrador de clústeres de conmutación por error**, seleccione **Acción**y, a continuación, seleccione **Validar configuración**.
1. Seleccione **Next** (Siguiente).
1. En **Seleccionar servidores o un clúster**, escriba el nombre de ambas máquinas virtuales.
1. En **Opciones de pruebas**, seleccione **Ejecutar solo las pruebas que seleccione**. 
1. Seleccione **Next** (Siguiente).
1. En **Selección de pruebas**, seleccione todas las pruebas excepto **Almacenamiento** y **Espacios de almacenamiento directo** como se muestra aquí:

   :::image type="content" source="media/failover-cluster-instance-premium-file-share-manually-configure/cluster-validation.png" alt-text="Seleccionar pruebas de validación de clústeres":::

1. Seleccione **Next** (Siguiente).
1. En **Confirmación**, seleccione **Siguiente**.

El Asistente para **validar una configuración** ejecuta las pruebas de validación.

Para validar el clúster con PowerShell, ejecute el siguiente script en una sesión de PowerShell de administrador de una de las máquinas virtuales:

   ```powershell
   Test-Cluster –Node ("<node1>","<node2>") –Include "Inventory", "Network", "System Configuration"
   ```

Después de validar el clúster, cree el clúster de conmutación por error.


## <a name="create-failover-cluster"></a>Creación de un clúster de conmutación por error

Para crear el clúster de conmutación por error, necesita:

- Los nombres de las máquinas virtuales que se convertirán en nodos del clúster.
- Un nombre para el clúster de conmutación por error.
- Una dirección IP para el clúster de conmutación por error. Puede usar una dirección IP que no se utilice en la misma red virtual de Azure y subred como nodos del clúster.


# <a name="windows-server-2012---2016"></a>[Windows Server 2012 a 2016](#tab/windows2012)

El siguiente script de PowerShell crea un clúster de conmutación por error para Windows Server 2012 a través de Windows Server 2016. Actualice el script con los nombres de los nodos (los nombres de las máquinas virtuales) y una dirección IP disponible desde la red virtual de Azure.

```powershell
New-Cluster -Name <FailoverCluster-Name> -Node ("<node1>","<node2>") –StaticAddress <n.n.n.n> -NoStorage
```   

# <a name="windows-server-2019"></a>[Windows Server 2019](#tab/windows2019)

El siguiente script de PowerShell crea un clúster de conmutación por error para Windows Server 2019.  Actualice el script con los nombres de los nodos (los nombres de las máquinas virtuales) y una dirección IP disponible desde la red virtual de Azure.

```powershell
New-Cluster -Name <FailoverCluster-Name> -Node ("<node1>","<node2>") –StaticAddress <n.n.n.n> -NoStorage -ManagementPointNetworkType Singleton 
```

Para más información, consulte [Clúster de conmutación por error: objeto de red en clúster](https://blogs.windows.com/windowsexperience/2018/08/14/announcing-windows-server-2019-insider-preview-build-17733/#W0YAxO8BfwBRbkzG.97).

---


## <a name="configure-quorum"></a>Configuración de un cuórum

Configure la solución del cuórum que mejor se adapte a sus necesidades empresariales. Puede configurar un [testigo de disco](/windows-server/failover-clustering/manage-cluster-quorum#configure-the-cluster-quorum), un [testigo en la nube](/windows-server/failover-clustering/deploy-cloud-witness) o un [testigo del recurso compartido de archivos](/windows-server/failover-clustering/manage-cluster-quorum#configure-the-cluster-quorum). Para obtener más información, consulte [Quorum con VM con SQL Server](hadr-cluster-best-practices.md#quorum). 


## <a name="test-cluster-failover"></a>Conmutación por error del clúster de prueba

Pruebe la conmutación por error del clúster. En **Administrador de clústeres de conmutación por error**, haga clic con el botón derecho en el clúster, seleccione **Más acciones** > **Mover recurso del clúster del recurso principal** > **Seleccionar nodo**, y después, seleccione el otro nodo del clúster. Mueva el recurso de clúster principal a cada nodo del clúster y, después, devuélvalo al nodo principal. Si puede mover correctamente el clúster a cada nodo, está listo para instalar SQL Server.  

:::image type="content" source="media/failover-cluster-instance-premium-file-share-manually-configure/test-cluster-failover.png" alt-text="Prueba de la conmutación por error del clúster moviendo el recurso principal a los demás nodos":::


## <a name="create-sql-server-fci"></a>Crear la FCI de SQL Server

Después de haber configurado el clúster de conmutación por error, puede crear la FCI de SQL Server.

1. Conéctese a la primera máquina virtual con RDP.

1. En **Administrador de clústeres de conmutación por error**, asegúrese de que todos los recursos principales de clúster estén en la primera máquina virtual. Si es necesario, mueva todos los recursos a esta máquina virtual.

1. Localice los medios de instalación. Si la máquina virtual usa una de las imágenes de Azure Marketplace, los medios se encuentran en `C:\SQLServer_<version number>_Full`. 

1. Seleccione **Setup** (Configuración).

1. En el **Centro de instalación de SQL Server**, seleccione **Instalación**.

1. Seleccione **Nueva instalación de clúster de conmutación por error de SQL Server** y, a continuación, siga las instrucciones del asistente para instalar la FCI de SQL Server.

   Es preciso que los directorios de datos de FCI estén en el recurso compartido de archivos premium. Escriba la ruta de acceso completa del recurso compartido,con este formato: `\\storageaccountname.file.core.windows.net\filesharename\foldername`. Aparecerá una advertencia que le notificará que ha especificado un servidor de archivos como directorio de datos. Se espera esta advertencia. Para evitar posibles errores, asegúrese de que la cuenta de usuario que usó para tener acceso a la máquina virtual mediante RDP cuando conservó el recurso compartido de archivos sea la misma que usa el servicio SQL Server.

   :::image type="content" source="media/failover-cluster-instance-premium-file-share-manually-configure/use-file-share-as-data-directories.png" alt-text="Uso del recurso compartido de archivos como directorios de datos SQL":::

1. Después de completar los pasos del asistente, el programa de instalación instalará una FCI de SQL Server en el primer nodo.

1. Una vez que el programa de instalación instale la FCI en el primer nodo, conéctese al segundo nodo con RDP.

1. Abra el **Centro de instalación de SQL Server** y, a continuación, seleccione **Instalación**.

1. Seleccione **Agregar nodo a clúster de conmutación por error de SQL Server**. Siga las instrucciones del asistente para instalar el servidor de SQL Server y agregarlo a la FCI.

   >[!NOTE]
   >Si usó una imagen de la galería de Azure Marketplace con SQL Server, las herramientas de SQL Server estaban incluidas en la imagen. Si no usó alguna de estas imágenes, instale las herramientas de SQL Server por separado. Para más información, consulte [Descargar SQL Server Management Studio (SSMS)](/sql/ssms/download-sql-server-management-studio-ssms).

1. Repita estos pasos en cualquier otro nodo que desee agregar a la instancia de clúster de conmutación por error de SQL Server. 

## <a name="register-with-the-sql-vm-rp"></a>Registro con el proveedor de recursos de máquina virtual con SQL

Para administrar la VM con SQL Server desde el portal, regístrela con el proveedor de recursos (RP) de máquina virtual con SQL en [modo de administración ligera](sql-vm-resource-provider-register.md#lightweight-management-mode), que actualmente es el único modo que se admite con la FCI y SQL Server en VM de Azure. 

Registre una máquina virtual con SQL Server en modo ligero con PowerShell ((-LicenseType puede ser `PAYG` o `AHUB`):

```powershell-interactive
# Get the existing compute VM
$vm = Get-AzVM -Name <vm_name> -ResourceGroupName <resource_group_name>
         
# Register SQL VM with 'Lightweight' SQL IaaS agent
New-AzSqlVM -Name $vm.Name -ResourceGroupName $vm.ResourceGroupName -Location $vm.Location `
   -LicenseType ???? -SqlManagementType LightWeight  
```

## <a name="configure-connectivity"></a>Configuración de la conectividad 

Para enrutar el tráfico de forma adecuada al nodo principal actual, configure la opción de conectividad adecuada para su entorno. Puede crear un [equilibrador de carga de Azure](hadr-vnn-azure-load-balancer-configure.md) o bien, si usa SQL Server 2019 y Windows Server 2016 (o posterior), puede obtener una versión preliminar de la característica [nombre de red distribuida](hadr-distributed-network-name-dnn-configure.md) en su lugar. 

## <a name="limitations"></a>Limitaciones

- No se admite el Coordinador de transacciones distribuidas de Microsoft (MSDTC) en Windows Server 2016 y versiones anteriores. 
- La secuencia de archivos no se admite en los clústeres de conmutación por error con un recurso compartido de archivos Premium. Para usar la secuencia de archivos, implemente el clúster con [Espacios de almacenamiento directo](failover-cluster-instance-storage-spaces-direct-manually-configure.md) o [discos compartidos de Azure](failover-cluster-instance-azure-shared-disks-manually-configure.md) en su lugar.
- Solo se admite el registro con el proveedor de recursos de máquina virtual con SQL en [modo de administración ligera](sql-vm-resource-provider-register.md#management-modes). 

## <a name="next-steps"></a>Pasos siguientes

Si aún no lo ha hecho, configure la conectividad a la FCI con un [nombre de la red virtual y una instancia de Azure Load Balancer](hadr-vnn-azure-load-balancer-configure.md) o un [nombre de red distribuido (DNN)](hadr-distributed-network-name-dnn-configure.md). 

Si los recursos compartidos de archivos Premium no son la solución de almacenamiento de la FCI adecuada para usted, considere la posibilidad de crear la FCI mediante [discos compartidos de Azure](failover-cluster-instance-azure-shared-disks-manually-configure.md) o [Espacios de almacenamiento directo](failover-cluster-instance-storage-spaces-direct-manually-configure.md) en su lugar. 

Para obtener más información, consulte información general de [FCI con SQL Server en VM de Azure](failover-cluster-instance-overview.md) y [procedimientos recomendados de configuración del clúster](hadr-cluster-best-practices.md). 

Para más información, consulte: 
- [Tecnologías de clúster de Windows](/windows-server/failover-clustering/failover-clustering-overview)   
- [Instancias del clúster de conmutación por error de SQL Server](/sql/sql-server/failover-clusters/windows/always-on-failover-cluster-instances-sql-server)
