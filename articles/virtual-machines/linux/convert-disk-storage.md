---
title: Conversión del almacenamiento de Managed Disks entre SSD estándar y Premium
description: Conversión de almacenamiento de Azure Managed Disks de estándar a Premium o de Premium a estándar mediante la CLI de Azure.
author: roygara
ms.service: virtual-machines-linux
ms.topic: how-to
ms.date: 07/12/2018
ms.author: rogarana
ms.subservice: disks
ms.openlocfilehash: bf16ee0af1c5816822f6a201ad1d5a70127082db
ms.sourcegitcommit: 2ff0d073607bc746ffc638a84bb026d1705e543e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87825385"
---
# <a name="convert-azure-managed-disks-storage-from-standard-to-premium-or-premium-to-standard"></a>Conversión de almacenamiento de Azure Managed Disks de estándar a Premium o de Premium a estándar

Hay cuatro tipos de discos de Azure Managed Disks: SSD Ultra de Azure (versión preliminar), SSD Premium, SSD estándar y HDD estándar. Puede cambiar entre los tres tipos de disco de disponibilidad general (SSD Premium, SSD estándar y HDD estándar) según las necesidades de rendimiento. Todavía no puede cambiar de o a un SSD Ultra; debe implementar uno nuevo.

Esta funcionalidad no se admite para discos no administrados. Sin embargo, puede [convertir un disco no administrado en un disco administrado](convert-unmanaged-to-managed-disks.md) de manera sencilla para poder cambiar entre los tipos de disco.

En este artículo se muestra cómo convertir discos administrados de estándar a Premium o de Premium a estándar mediante la CLI de Azure. Para instalar o actualizar la herramienta, consulte [Instalación de la CLI de Azure](/cli/azure/install-azure-cli).

## <a name="before-you-begin"></a>Antes de empezar

* La conversión de disco requiere reiniciar la máquina virtual (VM), por lo que debe programar la migración del almacenamiento de discos durante una ventana de mantenimiento existente previamente.
* En el caso de los discos no administrados, primero [debe convertirlos en discos administrados](convert-unmanaged-to-managed-disks.md) para poder cambiar entre las opciones de almacenamiento.


## <a name="switch-all-managed-disks-of-a-vm-between-premium-and-standard"></a>Cambio de todos los discos administrados de una máquina virtual entre Premium y estándar

En este ejemplo se muestra cómo convertir todos los discos de una máquina virtual de almacenamiento estándar a Premium o de almacenamiento Premium a estándar. Para usar Managed Disks premium, la máquina virtual debe usar un [Tamaño de máquina virtual](../sizes.md) que admita Premium Storage. En este ejemplo también se pasa a un tamaño que admite el almacenamiento Premium.

 ```azurecli

#resource group that contains the virtual machine
rgName='yourResourceGroup'

#Name of the virtual machine
vmName='yourVM'

#Premium capable size 
#Required only if converting from Standard to Premium
size='Standard_DS2_v2'

#Choose between Standard_LRS and Premium_LRS based on your scenario
sku='Premium_LRS'

#Deallocate the VM before changing the size of the VM
az vm deallocate --name $vmName --resource-group $rgName

#Change the VM size to a size that supports Premium storage 
#Skip this step if converting storage from Premium to Standard
az vm resize --resource-group $rgName --name $vmName --size $size

#Update the SKU of all the data disks 
az vm show -n $vmName -g $rgName --query storageProfile.dataDisks[*].managedDisk -o tsv \
 | awk -v sku=$sku '{system("az disk update --sku "sku" --ids "$1)}'

#Update the SKU of the OS disk
az vm show -n $vmName -g $rgName --query storageProfile.osDisk.managedDisk -o tsv \
| awk -v sku=$sku '{system("az disk update --sku "sku" --ids "$1)}'

az vm start --name $vmName --resource-group $rgName

```
## <a name="switch-individual-managed-disks-between-standard-and-premium"></a>Cambio de discos administrados individuales entre estándar y Premium

En el caso de la carga de trabajo de desarrollo y pruebas, puede que prefiera tener una combinación de discos estándar y Premium para reducir los costos. Puede optar por actualizar solamente los discos que necesitan un rendimiento más eficaz. En este ejemplo se muestra cómo convertir un disco de una máquina virtual de almacenamiento estándar a Premium o de almacenamiento Premium a estándar. Para usar Managed Disks premium, la máquina virtual debe usar un [Tamaño de máquina virtual](../sizes.md) que admita Premium Storage. En este ejemplo también se pasa a un tamaño que admite el almacenamiento Premium.

 ```azurecli

#resource group that contains the managed disk
rgName='yourResourceGroup'

#Name of your managed disk
diskName='yourManagedDiskName'

#Premium capable size 
#Required only if converting from Standard to Premium
size='Standard_DS2_v2'

#Choose between Standard_LRS and Premium_LRS based on your scenario
sku='Premium_LRS'

#Get the parent VM Id 
vmId=$(az disk show --name $diskName --resource-group $rgName --query managedBy --output tsv)

#Deallocate the VM before changing the size of the VM
az vm deallocate --ids $vmId 

#Change the VM size to a size that supports Premium storage 
#Skip this step if converting storage from Premium to Standard
az vm resize --ids $vmId --size $size

# Update the SKU
az disk update --sku $sku --name $diskName --resource-group $rgName 

az vm start --ids $vmId 
```

## <a name="switch-managed-disks-between-standard-hdd-and-standard-ssd"></a>Cambio de discos administrados entre HDD estándar y SSD estándar

En este ejemplo se muestra cómo convertir un disco de una máquina virtual de HDD estándar a SSD estándar o de SSD estándar a HDD estándar.

 ```azurecli

#resource group that contains the managed disk
rgName='yourResourceGroup'

#Name of your managed disk
diskName='yourManagedDiskName'

#Choose between Standard_LRS and StandardSSD_LRS based on your scenario
sku='StandardSSD_LRS'

#Get the parent VM ID 
vmId=$(az disk show --name $diskName --resource-group $rgName --query managedBy --output tsv)

#Deallocate the VM before changing the disk type
az vm deallocate --ids $vmId 

# Update the SKU
az disk update --sku $sku --name $diskName --resource-group $rgName 

az vm start --ids $vmId 
```

## <a name="switch-managed-disks-between-standard-and-premium-in-azure-portal"></a>Cambio de discos administrados individuales entre estándar y Premium en Azure Portal

Siga estos pasos:

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
2. Seleccione la máquina virtual de la lista de **Virtual Machines**.
3. Si la máquina virtual no se detiene, seleccione la opción **Detener** que se encuentra en la parte superior del panel **Información general** de la máquina virtual y espere a que se detenga.
4. En el panel de la máquina virtual, seleccione **Discos** del menú.
5. Seleccione el disco que quiera convertir.
6. Seleccione **Configuración** del menú.
7. Cambie el **Tipo de cuenta** de **HDD estándar** a **SSD Premium** o de **SSD Premium** a **HDD estándar**.
8. Seleccione **Guardar** y cierre el panel de discos.

La actualización del tipo de disco es instantánea. Puede reiniciar la máquina virtual después de la conversión.

## <a name="next-steps"></a>Pasos siguientes

Realice una copia de solo lectura de una máquina virtual mediante [instantáneas](snapshot-copy-managed-disk.md).