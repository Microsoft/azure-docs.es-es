---
title: Creación de una FCI con discos compartidos de Azure (versión preliminar)
description: Use discos compartidos de Azure para crear una instancia de clúster de conmutación por error (FCI) con SQL Server en Azure Virtual Machines.
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
ms.date: 06/26/2020
ms.author: mathoma
ms.openlocfilehash: ffb739affac68898f6ed5ff1d972d3fd4a70df2f
ms.sourcegitcommit: 420c30c760caf5742ba2e71f18cfd7649d1ead8a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89055267"
---
# <a name="create-an-fci-with-azure-shared-disks-sql-server-on-azure-vms"></a>Creación de una FCI con discos compartidos de Azure (SQL Server en VM de Azure)
[!INCLUDE[appliesto-sqlvm](../../includes/appliesto-sqlvm.md)]

En este artículo se explica cómo crear una instancia de clúster de conmutación por error (FCI) mediante discos compartidos de Azure con SQL Server en Azure Virtual Machines (VM). 

Para más información, consulte la información general de [FCI con SQL Server en VM de Azure](failover-cluster-instance-overview.md) y [Procedimientos recomendados de clúster](hadr-cluster-best-practices.md). 


## <a name="prerequisites"></a>Requisitos previos 

Antes de completar las instrucciones de este artículo, ya debe tener:

- Suscripción a Azure. Comience a usarla [gratis](https://azure.microsoft.com/free/). 
- [Dos o más máquinas virtuales de Microsoft Azure preparadas para el Centro-oeste de EE. UU.](failover-cluster-instance-prepare-vm.md) en el mismo [conjunto de disponibilidad](../../../virtual-machines/linux/tutorial-availability-sets.md) y un [grupo de ubicación de proximidad](../../../virtual-machines/windows/co-location.md#proximity-placement-groups), con el conjunto de disponibilidad que se crea con el dominio de error y el dominio de actualización establecidos en **1**. 
- Una cuenta que tenga permisos para crear objetos en máquinas virtuales de Azure y en Active Directory.
- La versión más reciente de [PowerShell](/powershell/azure/install-az-ps?view=azps-4.2.0). 


## <a name="add-azure-shared-disk"></a>Adición de un disco compartido de Azure
Implemente un disco SSD Premium administrado con la característica de disco compartido habilitada. Establezca `maxShares` en **2** para que el disco pueda compartirse en ambos nodos de FCI. 

Agregue un disco compartido de Azure mediante el procedimiento siguiente: 


1. Guarde el siguiente script como *SharedDiskConfig.json*: 

   ```JSON
   { 
     "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
     "contentVersion": "1.0.0.0",
     "parameters": {
       "dataDiskName": {
         "type": "string",
         "defaultValue": "mySharedDisk"
       },
       "dataDiskSizeGB": {
         "type": "int",
         "defaultValue": 1024
       },
       "maxShares": {
         "type": "int",
         "defaultValue": 2
       }
     },
     "resources": [
       {
         "type": "Microsoft.Compute/disks",
         "name": "[parameters('dataDiskName')]",
         "location": "[resourceGroup().location]",
         "apiVersion": "2019-07-01",
         "sku": {
           "name": "Premium_LRS"
         },
         "properties": {
           "creationData": {
             "createOption": "Empty"
           },
           "diskSizeGB": "[parameters('dataDiskSizeGB')]",
           "maxShares": "[parameters('maxShares')]"
         }
       }
     ] 
   }
   ```


2. Ejecute *SharedDiskConfig.json* mediante PowerShell: 

   ```powershell
   $rgName = < specify your resource group name>
       $location = 'westcentralus'
       New-AzResourceGroupDeployment -ResourceGroupName $rgName `
   -TemplateFile "SharedDiskConfig.json"
   ```

3. Para cada VM, inicialice los discos compartidos adjuntos como tabla de particiones GUID (GPT) y formatéelos como un nuevo sistema de archivos de tecnología (NTFS). Para ello, ejecute este comando: 

   ```powershell
   $resourceGroup = "<your resource group name>"
       $location = "<region of your shared disk>"
       $ppgName = "<your proximity placement groups name>"
       $vm = Get-AzVM -ResourceGroupName "<your resource group name>" `
           -Name "<your VM node name>"
       $dataDisk = Get-AzDisk -ResourceGroupName $resourceGroup `
           -DiskName "<your shared disk name>"
       $vm = Add-AzVMDataDisk -VM $vm -Name "<your shared disk name>" `
           -CreateOption Attach -ManagedDiskId $dataDisk.Id `
           -Lun <available LUN  check disk setting of the VM>
    update-AzVm -VM $vm -ResourceGroupName $resourceGroup
   ```


## <a name="create-failover-cluster"></a>Creación de un clúster de conmutación por error

Para crear el clúster de conmutación por error, necesita:

- Los nombres de las máquinas virtuales que se convertirán en nodos del clúster.
- Un nombre para el clúster de conmutación por error.
- Una dirección IP para el clúster de conmutación por error. Puede usar una dirección IP que no se utilice en la misma red virtual de Azure y subred como nodos del clúster.


# <a name="windows-server-2012-2016"></a>[Windows Server 2012-2016](#tab/windows2012)

El siguiente script de PowerShell crea un clúster de conmutación por error. Actualice el script con los nombres de los nodos (los nombres de las máquinas virtuales) y una dirección IP disponible desde la red virtual de Azure.

```powershell
New-Cluster -Name <FailoverCluster-Name> -Node ("<node1>","<node2>") –StaticAddress <n.n.n.n> -NoStorage
```   

# <a name="windows-server-2019"></a>[Windows Server 2019](#tab/windows2019)

El siguiente script de PowerShell crea un clúster de conmutación por error. Actualice el script con los nombres de los nodos (los nombres de las máquinas virtuales) y una dirección IP disponible desde la red virtual de Azure.

```powershell
New-Cluster -Name <FailoverCluster-Name> -Node ("<node1>","<node2>") –StaticAddress <n.n.n.n> -NoStorage -ManagementPointNetworkType Singleton 
```

Para más información, consulte [Clúster de conmutación por error: objeto de red en clúster](https://blogs.windows.com/windowsexperience/2018/08/14/announcing-windows-server-2019-insider-preview-build-17733/#W0YAxO8BfwBRbkzG.97).

---


## <a name="configure-quorum"></a>Configuración de un cuórum

Configure la solución de cuórum que mejor se adapte a sus necesidades empresariales. Puede configurar un [testigo de disco](/windows-server/failover-clustering/manage-cluster-quorum#configure-the-cluster-quorum), un [testigo en la nube](/windows-server/failover-clustering/deploy-cloud-witness) o un [testigo del recurso compartido de archivos](/windows-server/failover-clustering/manage-cluster-quorum#configure-the-cluster-quorum). Para más información, consulte [Cuórum con VM SQL Server](hadr-cluster-best-practices.md#quorum). 

## <a name="validate-cluster"></a>Validar el clúster
Valide el clúster en la interfaz de usuario o con PowerShell.

Para validar el clúster con la interfaz de usuario, realice los pasos siguientes en una de las máquinas virtuales:

1. En **Administrador del servidor**, seleccione **Herramientas** y, después, seleccione **Administrador de clústeres de conmutación por error**.
1. En **Administrador de clústeres de conmutación por error**, seleccione **Acción**y, a continuación, seleccione **Validar configuración**.
1. Seleccione **Next** (Siguiente).
1. En **Seleccionar servidores o un clúster**, escriba el nombre de ambas máquinas virtuales.
1. En **Opciones de pruebas**, seleccione **Ejecutar solo las pruebas que seleccione**. 
1. Seleccione **Next** (Siguiente).
1. En **Selección de pruebas**, elija todas las pruebas, *excepto* **Almacenamiento**.

## <a name="test-cluster-failover"></a>Conmutación por error del clúster de prueba

Pruebe la conmutación por error del clúster. En **Administrador de clústeres de conmutación por error**, haga clic con el botón derecho en el clúster, seleccione **Más acciones** > **Mover recurso del clúster del recurso principal** > **Seleccionar nodo**, y después, seleccione el otro nodo del clúster. Mueva el recurso de clúster principal a cada nodo del clúster y, después, devuélvalo al nodo principal. Si puede mover correctamente el clúster a cada nodo, está listo para instalar SQL Server.  

:::image type="content" source="media/failover-cluster-instance-premium-file-share-manually-configure/test-cluster-failover.png" alt-text="Prueba de la conmutación por error del clúster moviendo el recurso principal a los demás nodos":::

## <a name="create-sql-server-fci"></a>Crear la FCI de SQL Server

Después de haber configurado el clúster de conmutación por error y todos los componentes del clúster, incluido el almacenamiento, puede crear la FCI de SQL Server.

1. Establezca la conexión a la primera máquina virtual mediante el Protocolo de escritorio remoto (RDP).

1. En **Administrador de clústeres de conmutación por error**, asegúrese de que todos los recursos principales de clúster estén en la primera máquina virtual. Si es necesario, mueva todos los recursos a esa máquina virtual.

1. Localice los medios de instalación. Si la máquina virtual usa una de las imágenes de Azure Marketplace, los medios se encuentran en `C:\SQLServer_<version number>_Full`. 

1. Seleccione **Setup** (Configuración).

1. En **Centro de instalación de SQL Server**, seleccione **Instalación**.

1. Seleccione **Nueva instalación de clúster de conmutación por error de SQL Server**. Siga las instrucciones del asistente para instalar la FCI de SQL Server.

Es preciso que los directorios de datos de FCI estén en los discos compartidos de Azure. 

1. Después de completar las instrucciones en el asistente, el programa de instalación instalará una FCI de SQL Server en el primer nodo.

1. Una vez que el programa de instalación instale la FCI en el primer nodo, conéctese al segundo nodo con RDP.

1. Abra el **Centro de instalación de SQL Server** y, a continuación, seleccione **Instalación**.

1. Seleccione **Agregar nodo a clúster de conmutación por error de SQL Server**. Siga las instrucciones del asistente para instalar el servidor de SQL Server y agregarlo a la FCI.

   >[!NOTE]
   >Si usó una imagen de la galería de Azure Marketplace con SQL Server, las herramientas de SQL Server estaban incluidas en la imagen. Si no usó alguna de estas imágenes, instale las herramientas de SQL Server por separado. Para más información, consulte [Descargar SQL Server Management Studio (SSMS)](https://msdn.microsoft.com/library/mt238290.aspx).
   >

## <a name="register-with-the-sql-vm-rp"></a>Registro con el proveedor de recursos de VM de SQL

Para administrar la VM de SQL Server desde el portal, regístrela con el proveedor de recursos (RP) de VM de SQL en [modo de administración ligera](sql-vm-resource-provider-register.md#lightweight-management-mode); actualmente, es el único modo que se admite con FCI y SQL Server en las VM de Azure. 


Registre una VM con SQL Server en modo ligero con PowerShell:  

```powershell-interactive
# Get the existing compute VM
$vm = Get-AzVM -Name <vm_name> -ResourceGroupName <resource_group_name>
         
# Register SQL VM with 'Lightweight' SQL IaaS agent
New-AzSqlVM -Name $vm.Name -ResourceGroupName $vm.ResourceGroupName -Location $vm.Location `
   -LicenseType PAYG -SqlManagementType LightWeight  
```

## <a name="configure-connectivity"></a>Configuración de la conectividad 

Para enrutar el tráfico de forma adecuada al nodo principal actual, configure la opción de conectividad apropiada para su entorno. Puede crear un [equilibrador de carga de Azure](hadr-vnn-azure-load-balancer-configure.md) o bien, si usa SQL Server 2019 y Windows Server 2016 (o posterior), puede obtener una versión preliminar de la característica [nombre de red distribuida](hadr-distributed-network-name-dnn-configure.md) en su lugar. 

## <a name="limitations"></a>Limitaciones

- Solo se admite SQL Server 2019 en Windows Server 2019. 
- Solo se admite el registro con el proveedor de recursos de VM de SQL en [modo de administración ligero](sql-vm-resource-provider-register.md#management-modes).

## <a name="next-steps"></a>Pasos siguientes

Si aún no lo ha hecho, configure la conectividad a su FCI con un [nombre de red virtual y un equilibrador de carga de Azure](hadr-vnn-azure-load-balancer-configure.md) o un [nombre de red distribuida (DNN)](hadr-distributed-network-name-dnn-configure.md). 

Si los discos compartidos de Azure no son la solución de almacenamiento de la FCI adecuada para usted, considere la posibilidad de crear la FCI mediante [recursos compartidos de archivos Premium](failover-cluster-instance-premium-file-share-manually-configure.md) o [Espacios de almacenamiento directo](failover-cluster-instance-storage-spaces-direct-manually-configure.md) en su lugar. 

Para más información, consulte la introducción a [FCI con SQL Server en VM de Azure](failover-cluster-instance-overview.md) y los [procedimientos recomendados de configuración de clúster](hadr-cluster-best-practices.md).

Para obtener más información, consulte: 
- [Tecnologías de clúster de Windows](/windows-server/failover-clustering/failover-clustering-overview)   
- [Instancias del clúster de conmutación por error de SQL Server](/sql/sql-server/failover-clusters/windows/always-on-failover-cluster-instances-sql-server)
