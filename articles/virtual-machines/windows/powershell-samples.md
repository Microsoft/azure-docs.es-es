---
title: Ejemplos de PowerShell de máquina virtual de Azure
description: Ejemplos de PowerShell de máquina virtual de Azure
author: cynthn
ms.service: virtual-machines-windows
ms.topic: sample
ms.workload: infrastructure
ms.date: 03/01/2019
ms.author: cynthn
ms.custom: mvc
ms.openlocfilehash: e8489ecb9343cf70bd8523aa45f04a6bde171b98
ms.sourcegitcommit: 5ed504a9ddfbd69d4f2d256ec431e634eb38813e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89321759"
---
# <a name="azure-virtual-machine-powershell-samples"></a>Ejemplos de PowerShell de máquina virtual de Azure

En la tabla siguiente se proporcionan vínculos a ejemplos de scripts de PowerShell que crean y administran máquinas virtuales (VM) Windows.

| Script | Descripción |
|---|---|
|**Creación de máquinas virtuales**||
| [Creación rápida de una máquina virtual](./../scripts/virtual-machines-windows-powershell-sample-create-vm-quick.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json) | Crea un grupo de recursos, una máquina virtual y todos los recursos relacionados con un mínimo de mensajes.|
| [Creación una máquina virtual completamente configurada](./../scripts/virtual-machines-windows-powershell-sample-create-vm.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json) | Crea un grupo de recursos, una máquina virtual y todos los recursos relacionados.|
| [Creación de máquinas virtuales de alta disponibilidad](./../scripts/virtual-machines-windows-powershell-sample-create-nlb-vm.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json) | Crea varias máquinas virtuales en una configuración de alta disponibilidad y de equilibrio de carga.|
| [Creación de una VM y ejecución de un script de configuración](./../scripts/virtual-machines-windows-powershell-sample-create-vm-iis.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json) | Crea una máquina virtual y usa la extensión de script personalizado de Azure para instalar IIS. |
| [Creación de una VM y ejecución de una configuración de DSC](./../scripts/virtual-machines-windows-powershell-sample-create-iis-using-dsc.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json) | Crea una máquina virtual y usa la extensión de configuración de estado deseado (DSC) de Azure para instalar IIS. |
| [Carga de un disco duro virtual y creación de máquinas virtuales](./../scripts/virtual-machines-windows-powershell-upload-generalized-script.md) | Carga un archivo de VHD local en Azure, crea una imagen desde el VHD y, después, crea una VM a partir de esa imagen. |
| [Creación de una máquina virtual desde un disco administrado](./../scripts/virtual-machines-powershell-sample-create-vm-from-managed-os-disks.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json) | Crea una máquina virtual conectando un disco administrado como disco del SO. |
| [Creación de una VM a partir de una instantánea](./../scripts/virtual-machines-windows-powershell-sample-create-vm-from-snapshot.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json) | Crea una máquina virtual desde una instantánea creando primero un disco administrado de la instantánea y conectando luego el nuevo disco administrado como disco del sistema operativo. |
|**Administrar el almacenamiento**||
| [Creación de un disco administrado desde un VHD en la misma suscripción o en otra](../scripts/virtual-machines-powershell-sample-create-managed-disk-from-vhd.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json) | Crea un disco administrado a partir de un VHD especializado como un disco del sistema operativo, o a partir de un VHD de datos como un disco de datos en la misma suscripción o en otra.  |
| [Crear un disco administrado a partir de una instantánea](../scripts/virtual-machines-powershell-sample-create-managed-disk-from-snapshot.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json) | Crea un disco administrado a partir de una instantánea. |
| [Copia de un disco administrado en la misma suscripción o en otra](../scripts/virtual-machines-powershell-sample-copy-managed-disks-to-same-or-different-subscription.md?toc=%2fcli%2fmodule%2ftoc.json) | Copia un disco administrado en la misma suscripción o en otra, que está en la misma región que el disco administrado primario.
| [Exportación de una instantánea como VHD a una cuenta de almacenamiento](../scripts/virtual-machines-powershell-sample-copy-snapshot-to-storage-account.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json) | Exporta una instantánea administrada como VHD a una cuenta de almacenamiento en otra región. |
| [Exportar el disco duro virtual de un disco administrado a una cuenta de almacenamiento](../scripts/virtual-machines-powershell-sample-copy-managed-disks-vhd.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json) | Exporta el VHD subyacente de un disco administrado a una cuenta de almacenamiento en otra región. |
| [Creación de una instantánea a partir de un disco duro virtual](../scripts/virtual-machines-powershell-sample-create-snapshot-from-vhd.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json) | Crea una instantánea a partir de un VHD y luego usa esa instantánea para crear varios discos administrados idénticos rápidamente.  |
| [Copia de una instantánea en la misma suscripción o en otra](../scripts/virtual-machines-powershell-sample-copy-snapshot-to-same-or-different-subscription.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json) | Copia la instantánea en la misma suscripción o en otra, que está en la misma región que la instantánea primaria. |
|**Protección de las máquinas virtuales**||
| [Cifrado de una VM y sus discos de datos](./../scripts/virtual-machines-windows-powershell-sample-encrypt-vm.md?toc=%2fpowershell%2fazure%2ftoc.json) | Crea un almacén de claves de Azure, una clave de cifrado y una entidad de servicio, y luego cifra una VM. |
|**Supervisión de máquinas virtuales**||
| [Supervisión de una máquina virtual con Azure Monitor](./../scripts/virtual-machines-windows-powershell-sample-create-vm-oms.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json) | Crea una máquina virtual, instala el agente de Azure Log Analytics e inscribe la VM en un área de trabajo de Log Analytics.  |
| [Recopilación de detalles sobre todas las máquinas virtuales de una suscripción con PowerShell](../scripts/virtual-machines-powershell-sample-collect-vm-details.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json) | Crea un csv que contiene el nombre de la máquina virtual, el nombre del grupo de recursos, la región, la red virtual, la subred, la dirección IP privada, el tipo de sistema operativo y la dirección IP pública de las máquinas virtuales de la suscripción proporcionada.
| | |
