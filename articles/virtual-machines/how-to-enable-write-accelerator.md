---
title: Acelerador de escritura de Azure
description: Documentación sobre cómo habilitar y utilizar el acelerador de escritura
author: raiye
manager: markkie
ms.service: virtual-machines
ms.topic: how-to
ms.workload: infrastructure
ms.date: 2/20/2019
ms.author: raiye
ms.subservice: disks
ms.openlocfilehash: 0b5e6134de2260998e599bad0d1bf6b381898ffd
ms.sourcegitcommit: 54d8052c09e847a6565ec978f352769e8955aead
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88513060"
---
# <a name="enable-write-accelerator"></a>Habilitar el acelerador de escritura

El Acelerador de escritura es una capacidad de disco para las máquinas virtuales (VM) de la serie M en Premium Storage con Azure Managed Disks exclusivamente. Como el nombre indica, el propósito de la funcionalidad es mejorar la latencia de E/S de las escrituras en Azure Premium Storage. El Acelerador de escritura se adecua perfectamente a las situaciones en las que las actualizaciones del archivo de registro deben persistir en el disco de una manera muy eficiente para bases de datos modernas.

En general, el Acelerador de escritura está disponible para VM de serie la M en la nube pública.

## <a name="planning-for-using-write-accelerator"></a>Planeación del uso del Acelerador de escritura

El Acelerador de escritura se debe usar para los volúmenes que contienen el registro de transacciones o los registros de puestas al día de un DBMS. No se recomienda utilizar el Acelerador de escritura para los volúmenes de datos de un DBMS, ya que la característica se ha optimizado para su uso en los discos de registro.

El Acelerador de escritura solo funciona en conjunto con [Azure Managed Disks](https://azure.microsoft.com/services/managed-disks/).

> [!IMPORTANT]
> La habilitación del Acelerador de escritura para el disco del sistema operativo de la máquina virtual hará que esta se reinicie.
>
> Para habilitar el Acelerador de escritura en un disco existente de Azure que NO forme parte de un volumen compuesto de varios discos con administradores de volúmenes o discos de Windows, espacios de almacenamiento de Windows, servidor de archivos de escalabilidad horizontal (SOFS) de Windows, LVM o MDADM de Linux, la carga de trabajo que accede al disco de Azure debe estar apagada. Las aplicaciones de base de datos que usan el disco de Azure DEBEN estar apagadas.
>
> Si desea habilitar o deshabilitar el Acelerador de escritura para un volumen existente que conste de varios discos de Azure Premium Storage y se segmente con administradores de volúmenes o discos de Windows, espacios de almacenamiento de Windows, servidor de archivos de escalabilidad horizontal (SOFS) de Windows, LVM o MDADM de Linux, todos los discos que componen el volumen se deben habilitar o deshabilitar para el Acelerador de escritura en pasos independientes. **Antes de habilitar o deshabilitar el Acelerador de escritura en este tipo de configuración, apague la máquina virtual de Azure**.

La habilitación del Acelerador de escritura para los discos de sistema operativo no debería ser necesaria para las configuraciones de máquinas virtuales relacionadas con SAP.

### <a name="restrictions-when-using-write-accelerator"></a>Restricciones del uso del Acelerador de escritura

Estas restricciones se aplican al usar el Acelerador de escritura para un VHD o disco de Azure:

- El almacenamiento en caché de discos Premium debe establecerse en "Ninguno" o "Solo lectura". No se admite ningún otro modo de almacenamiento en caché.
- Actualmente, los discos con el Acelerador de escritura habilitado no admiten las instantáneas. Durante la copia de seguridad, el servicio Azure Backup excluye automáticamente los discos con el Acelerador de escritura habilitado conectados a la VM.
- Solo los tamaños de E/S más pequeños (<=512 KiB) toman la ruta de acceso acelerada. En situaciones de carga de trabajo donde los datos se cargan de forma masiva o donde los búferes de registros de transacción de los diferentes sistemas de administración de bases de datos (DBMS) se llenan hasta un mayor grado antes de conservarse en el almacenamiento, existe la probabilidad de que la E/S escrita en el disco no tome la ruta de acceso acelerada.

Hay límites en los discos duros virtuales de Azure Premium Storage por máquina virtual que el Acelerador de escritura puede admitir. Los límites actuales son:

| SKU de la máquina virtual | Número de discos de Acelerador de escritura | IOPS de discos de Acelerador de escritura por máquina virtual |
| --- | --- | --- |
| M416ms_v2, M416s_v2| 16 | 20000 |
| M208ms_v2, M208s_v2| 8 | 10000 |
| M128ms, M128s | 16 | 20000 |
| M64ms, M64ls, M64s | 8 | 10000 |
| M32ms, M32ls, M32ts, M32s | 4 | 5000 |
| M16ms, M16s | 2 | 2.500 |
| M8ms, M8s | 1 | 1250 |

Los límites de IOPS son por máquina virtual y *no* por disco. Todos los discos del Acelerador de escritura comparten el mismo límite de IOPS por máquina virtual.

## <a name="enabling-write-accelerator-on-a-specific-disk"></a>Habilitación del Acelerador de escritura en un disco específico

En las secciones siguientes se describe cómo el Acelerador de escritura se pueden habilitar en los VHD de Azure Premium Storage.

### <a name="prerequisites"></a>Prerrequisitos

En este momento, los requisitos previos siguientes se aplican al uso del Acelerador de escritura:

- Los discos en los que desea aplicar el Acelerador de escritura de Azure deben ser [Azure Managed Disks](https://azure.microsoft.com/services/managed-disks/) en Premium Storage.
- Debe usar una máquina virtual de la serie M

## <a name="enabling-azure-write-accelerator-using-azure-powershell"></a>Habilitación del Acelerador de escritura mediante Azure PowerShell

El módulo de Azure PowerShell a partir de la versión 5.5.0 incluye los cambios en los cmdlets pertinentes para habilitar o deshabilitar el Acelerador de escritura para discos específicos de Azure Premium Storage.
Con el fin de habilitar o implementar los discos compatibles con el Acelerador de escritura, se modificaron los comandos de PowerShell siguientes y se extendieron para que aceptaran un parámetro para el Acelerador de escritura.

Se ha agregado **-WriteAccelerator**, un nuevo parámetro de modificador, a los siguientes cmdlets:

- [Set-AzVMOsDisk](https://docs.microsoft.com/powershell/module/az.compute/set-azvmosdisk?view=azurermps-6.0.0)
- [Add-AzVMDataDisk](https://docs.microsoft.com/powershell/module/az.compute/Add-AzVMDataDisk?view=azurermps-6.0.0)
- [Set-AzVMDataDisk](https://docs.microsoft.com/powershell/module/az.compute/Set-AzVMDataDisk?view=azurermps-6.0.0)
- [Add-AzVmssDataDisk](https://docs.microsoft.com/powershell/module/az.compute/Add-AzVmssDataDisk?view=azurermps-6.0.0)

Si no se proporciona el parámetro, la propiedad se establece en false y se implementarán discos no compatibles con el Acelerador de escritura.

Se agregó **-OsDiskWriteAccelerator**, un nuevo parámetro de modificador, a los siguientes cmdlets:

- [Set-AzVmssStorageProfile](https://docs.microsoft.com/powershell/module/az.compute/Set-AzVmssStorageProfile?view=azurermps-6.0.0)

Si no se especifica el parámetro, la propiedad se establece en false de forma predeterminada y se devuelven los discos que no usan el Acelerador de escritura.

Se agregó **-OsDiskWriteAccelerator**, un nuevo parámetro booleano opcional (que no admite valores NULL), a los siguientes cmdlets:

- [Update-AzVM](https://docs.microsoft.com/powershell/module/az.compute/Update-AzVM?view=azurermps-6.0.0)
- [Update-AzVmss](https://docs.microsoft.com/powershell/module/az.compute/Update-AzVmss?view=azurermps-6.0.0)

Especifique $true o $false para controlar la compatibilidad del Acelerador de escritura con los discos.

Algunos ejemplos de los comandos podrían ser:

```powershell
New-AzVMConfig | Set-AzVMOsDisk | Add-AzVMDataDisk -Name "datadisk1" | Add-AzVMDataDisk -Name "logdisk1" -WriteAccelerator | New-AzVM

Get-AzVM | Update-AzVM -OsDiskWriteAccelerator $true

New-AzVmssConfig | Set-AzVmssStorageProfile -OsDiskWriteAccelerator | Add-AzVmssDataDisk -Name "datadisk1" -WriteAccelerator:$false | Add-AzVmssDataDisk -Name "logdisk1" -WriteAccelerator | New-AzVmss

Get-AzVmss | Update-AzVmss -OsDiskWriteAccelerator:$false
```

Se pueden ejecutar dos escenarios principales mediante scripts, tal como se muestra en las secciones siguientes.

### <a name="adding-a-new-disk-supported-by-write-accelerator-using-powershell"></a>Incorporación de un disco nuevo compatible con el Acelerador de escritura utilizando PowerShell

Puede usar este script para agregar un disco nuevo a la máquina virtual. El disco que se crea con este script usa el Acelerador de escritura.

Reemplace `myVM`, `myWAVMs`, `log001`, el tamaño del disco y el LunID del disco con los valores adecuados para su implementación.

```powershell
# Specify your VM Name
$vmName="myVM"
#Specify your Resource Group
$rgName = "myWAVMs"
#data disk name
$datadiskname = "log001"
#LUN Id
$lunid=8
#size
$size=1023
#Pulls the VM info for later
$vm=Get-AzVM -ResourceGroupName $rgname -Name $vmname
#add a new VM data disk
Add-AzVMDataDisk -CreateOption empty -DiskSizeInGB $size -Name $vmname-$datadiskname -VM $vm -Caching None -WriteAccelerator:$true -lun $lunid
#Updates the VM with the disk config - does not require a reboot
Update-AzVM -ResourceGroupName $rgname -VM $vm
```

### <a name="enabling-write-accelerator-on-an-existing-azure-disk-using-powershell"></a>Habilitación del Acelerador de escritura en un disco de Azure existente utilizando PowerShell

Pueden usar este script para habilitar el Acelerador de escritura en un disco existente. Reemplace `myVM`, `myWAVMs` y `test-log001` con los valores adecuados para su implementación. El script agrega el Acelerador de escritura a un disco existente cuando el valor de **$newstatus** está establecido en "$true". Si usa el valor "$false", el Acelerador de escritura se deshabilitará en un disco determinado.

```powershell
#Specify your VM Name
$vmName="myVM"
#Specify your Resource Group
$rgName = "myWAVMs"
#data disk name
$datadiskname = "test-log001" 
#new Write Accelerator status ($true for enabled, $false for disabled) 
$newstatus = $true
#Pulls the VM info for later
$vm=Get-AzVM -ResourceGroupName $rgname -Name $vmname
#add a new VM data disk
Set-AzVMDataDisk -VM $vm -Name $datadiskname -Caching None -WriteAccelerator:$newstatus
#Updates the VM with the disk config - does not require a reboot
Update-AzVM -ResourceGroupName $rgname -VM $vm
```

> [!Note]
> Si ejecuta el script anterior, desconectará el disco especificado, habilitará el Acelerador de escritura en el disco y, luego, lo volverá a adjuntar.

## <a name="enabling-write-accelerator-using-the-azure-portal"></a>Habilitación del Acelerador de escritura mediante Azure Portal

Puede habilitar el Acelerador de escritura a través del portal, donde especifique la configuración de caché del disco:

![Acelerador de escritura en Azure Portal](./media/virtual-machines-common-how-to-enable-write-accelerator/wa_scrnsht.png)

## <a name="enabling-write-accelerator-using-the-azure-cli"></a>Habilitación del Acelerador de escritura mediante la CLI de Azure

Puede usar la [CLI de Azure](https://docs.microsoft.com/cli/azure/?view=azure-cli-latest) para habilitar el acelerador de escritura.

Para habilitar el Acelerador de escritura en un disco existente, use [az vm update](https://docs.microsoft.com/cli/azure/vm?view=azure-cli-latest#az-vm-update). Puede usar los siguientes ejemplos si sustituye los valores diskName, VMName y ResourceGroup por los suyos propios: `az vm update -g group1 -n vm1 -write-accelerator 1=true`

Para conectar un disco con el Acelerador de escritura habilitado, use [az vm disk attach](https://docs.microsoft.com/cli/azure/vm/disk?view=azure-cli-latest#az-vm-disk-attach), puede utilizar el ejemplo siguiente si sustituye los valores por los suyos propios: `az vm disk attach -g group1 -vm-name vm1 -disk d1 --enable-write-accelerator`

Para deshabilitar el Acelerador de escritura, use [az vm update](https://docs.microsoft.com/cli/azure/vm?view=azure-cli-latest#az-vm-update), estableciendo las propiedades en false: `az vm update -g group1 -n vm1 -write-accelerator 0=false 1=false`

## <a name="enabling-write-accelerator-using-rest-apis"></a>Habilitación del Acelerador de escritura mediante las API REST

Para implementar a través de la API REST de Azure, necesita instalar ARMclient de Azure.

### <a name="install-armclient"></a>Instalación de ARMclient

Para ejecutar ARMclient, debe instalarlo a través de Chocolatey. Puede instalarlo a través de cmd.exe o PowerShell. Use derechos elevados para estos comandos ("Ejecutar como administrador").

Con cmd.exe, ejecute este comando: `@"%SystemRoot%\System32\WindowsPowerShell\v1.0\powershell.exe" -NoProfile -InputFormat None -ExecutionPolicy Bypass -Command "iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))" && SET "PATH=%PATH%;%ALLUSERSPROFILE%\chocolatey\bin"`

Con PowerShell, ejecute este comando: `Set-ExecutionPolicy Bypass -Scope Process -Force; iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))`

Ahora puede instalar armclient con el siguiente comando en cmd.exe o PowerShell `choco install armclient`

### <a name="getting-your-current-vm-configuration"></a>Obtención de la configuración de la máquina virtual actual

Para cambiar los atributos de la configuración de disco, primero debe obtener la configuración actual en un archivo JSON. Puede obtener la configuración actual ejecutando el siguiente comando: `armclient GET /subscriptions/<<subscription-ID<</resourceGroups/<<ResourceGroup>>/providers/Microsoft.Compute/virtualMachines/<<virtualmachinename>>?api-version=2017-12-01 > <<filename.json>>`

Reemplace los términos dentro de "<<   >>" con los datos, incluido el nombre que debe tener el archivo JSON.

La salida debería parecerse a esta:

```JSON
{
  "properties": {
    "vmId": "2444c93e-f8bb-4a20-af2d-1658d9dbbbcb",
    "hardwareProfile": {
      "vmSize": "Standard_M64s"
    },
    "storageProfile": {
      "imageReference": {
        "publisher": "SUSE",
        "offer": "SLES-SAP",
        "sku": "12-SP3",
        "version": "latest"
      },
      "osDisk": {
        "osType": "Linux",
        "name": "mylittlesap_OsDisk_1_754a1b8bb390468e9b4c429b81cc5f5a",
        "createOption": "FromImage",
        "caching": "ReadWrite",
        "managedDisk": {
          "storageAccountType": "Premium_LRS",
          "id": "/subscriptions/XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX/resourceGroups/mylittlesap/providers/Microsoft.Compute/disks/mylittlesap_OsDisk_1_754a1b8bb390468e9b4c429b81cc5f5a"
        },
        "diskSizeGB": 30
      },
      "dataDisks": [
        {
          "lun": 0,
          "name": "data1",
          "createOption": "Attach",
          "caching": "None",
          "managedDisk": {
            "storageAccountType": "Premium_LRS",
            "id": "/subscriptions/XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX/resourceGroups/mylittlesap/providers/Microsoft.Compute/disks/data1"
          },
          "diskSizeGB": 1023
        },
        {
          "lun": 1,
          "name": "log1",
          "createOption": "Attach",
          "caching": "None",
          "managedDisk": {
            "storageAccountType": "Premium_LRS",
            "id": "/subscriptions/XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX/resourceGroups/mylittlesap/providers/Microsoft.Compute/disks/data2"
          },
          "diskSizeGB": 1023
        }
      ]
    },
    "osProfile": {
      "computerName": "mylittlesapVM",
      "adminUsername": "pl",
      "linuxConfiguration": {
        "disablePasswordAuthentication": false
      },
      "secrets": []
    },
    "networkProfile": {
      "networkInterfaces": [
        {
          "id": "/subscriptions/XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX/resourceGroups/mylittlesap/providers/Microsoft.Network/networkInterfaces/mylittlesap518"
        }
      ]
    },
    "diagnosticsProfile": {
      "bootDiagnostics": {
        "enabled": true,
        "storageUri": "https://mylittlesapdiag895.blob.core.windows.net/"
      }
    },
    "provisioningState": "Succeeded"
  },
  "type": "Microsoft.Compute/virtualMachines",
  "location": "westeurope",
  "id": "/subscriptions/XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX/resourceGroups/mylittlesap/providers/Microsoft.Compute/virtualMachines/mylittlesapVM",
  "name": "mylittlesapVM"

```

Después, actualice el archivo JSON y habilite el Acelerador de escritura en el disco llamado "log1". Para ello, agregue este atributo en el archivo JSON después de la entrada de caché del disco.

```JSON
        {
          "lun": 1,
          "name": "log1",
          "createOption": "Attach",
          "caching": "None",
          "writeAcceleratorEnabled": true,
          "managedDisk": {
            "storageAccountType": "Premium_LRS",
            "id": "/subscriptions/XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX/resourceGroups/mylittlesap/providers/Microsoft.Compute/disks/data2"
          },
          "diskSizeGB": 1023
        }
```

Luego, actualice la implementación existente con este comando: `armclient PUT /subscriptions/<<subscription-ID<</resourceGroups/<<ResourceGroup>>/providers/Microsoft.Compute/virtualMachines/<<virtualmachinename>>?api-version=2017-12-01 @<<filename.json>>`

La salida debe parecerse a la que aparece a continuación. Puede ver que el Acelerador de escritura está habilitado para un disco.

```JSON
{
  "properties": {
    "vmId": "2444c93e-f8bb-4a20-af2d-1658d9dbbbcb",
    "hardwareProfile": {
      "vmSize": "Standard_M64s"
    },
    "storageProfile": {
      "imageReference": {
        "publisher": "SUSE",
        "offer": "SLES-SAP",
        "sku": "12-SP3",
        "version": "latest"
      },
      "osDisk": {
        "osType": "Linux",
        "name": "mylittlesap_OsDisk_1_754a1b8bb390468e9b4c429b81cc5f5a",
        "createOption": "FromImage",
        "caching": "ReadWrite",
        "managedDisk": {
          "storageAccountType": "Premium_LRS",
          "id": "/subscriptions/XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX/resourceGroups/mylittlesap/providers/Microsoft.Compute/disks/mylittlesap_OsDisk_1_754a1b8bb390468e9b4c429b81cc5f5a"
        },
        "diskSizeGB": 30
      },
      "dataDisks": [
        {
          "lun": 0,
          "name": "data1",
          "createOption": "Attach",
          "caching": "None",
          "managedDisk": {
            "storageAccountType": "Premium_LRS",
            "id": "/subscriptions/XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX/resourceGroups/mylittlesap/providers/Microsoft.Compute/disks/data1"
          },
          "diskSizeGB": 1023
        },
        {
          "lun": 1,
          "name": "log1",
          "createOption": "Attach",
          "caching": "None",
          "writeAcceleratorEnabled": true,
          "managedDisk": {
            "storageAccountType": "Premium_LRS",
            "id": "/subscriptions/XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX/resourceGroups/mylittlesap/providers/Microsoft.Compute/disks/data2"
          },
          "diskSizeGB": 1023
        }
      ]
    },
    "osProfile": {
      "computerName": "mylittlesapVM",
      "adminUsername": "pl",
      "linuxConfiguration": {
        "disablePasswordAuthentication": false
      },
      "secrets": []
    },
    "networkProfile": {
      "networkInterfaces": [
        {
          "id": "/subscriptions/XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX/resourceGroups/mylittlesap/providers/Microsoft.Network/networkInterfaces/mylittlesap518"
        }
      ]
    },
    "diagnosticsProfile": {
      "bootDiagnostics": {
        "enabled": true,
        "storageUri": "https://mylittlesapdiag895.blob.core.windows.net/"
      }
    },
    "provisioningState": "Succeeded"
  },
  "type": "Microsoft.Compute/virtualMachines",
  "location": "westeurope",
  "id": "/subscriptions/XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX/resourceGroups/mylittlesap/providers/Microsoft.Compute/virtualMachines/mylittlesapVM",
  "name": "mylittlesapVM"
```

Una vez haya hecho el cambio, la unidad debería admitir el Acelerador de escritura.
