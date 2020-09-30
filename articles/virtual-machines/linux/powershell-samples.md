---
title: Ejemplos de PowerShell
description: Lista de ejemplos de PowerShell para máquinas virtuales
author: cynthn
ms.service: virtual-machines
ms.topic: how-to
ms.workload: infrastructure
ms.date: 03/01/2019
ms.author: cynthn
ms.openlocfilehash: b1a154e562db9f8fdd74300bb8eb3d6839cb2ef4
ms.sourcegitcommit: 5ed504a9ddfbd69d4f2d256ec431e634eb38813e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89321929"
---
# <a name="azure-vm-powershell-samples-for-creating-and-managing-linux-vms"></a>Ejemplos de PowerShell de máquina virtual de Azure para crear y administrar máquinas virtuales con Linux

En la tabla siguiente se incluyen vínculos a ejemplos de scripts de PowerShell que crean y administran máquinas virtuales Linux.

| Script | Descripción |
|---|---|
|**Creación de máquinas virtuales**||
| [Creación una máquina virtual completamente configurada](./../scripts/virtual-machines-linux-powershell-sample-create-vm.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) | Crea un grupo de recursos, una máquina virtual y todos los recursos relacionados.|
| [Creación de una máquina virtual con Docker habilitado](./../scripts/virtual-machines-linux-powershell-sample-create-docker-host.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) | Crea una máquina virtual, la configura como un host Docker y ejecuta un contenedor NGINX. |
| [Creación de una máquina virtual y ejecución de un script de configuración](./../scripts/virtual-machines-linux-powershell-sample-create-vm-nginx.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) | Crea una máquina virtual y usa la extensión de script personalizado de Azure para instalar NGINX. |
| [Creación de una máquina virtual con WordPress instalado](./../scripts/virtual-machines-linux-powershell-sample-create-vm-wordpress.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) | Crea una máquina virtual y usa la extensión de script personalizado de Azure para instalar WordPress. |
| [Creación de una máquina virtual desde un disco administrado](./../scripts/virtual-machines-powershell-sample-create-vm-from-managed-os-disks.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) | Crea una máquina virtual conectando un disco administrado como disco del SO. |
| [Creación de una VM a partir de una instantánea](./../scripts/virtual-machines-linux-powershell-sample-create-vm-from-snapshot.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) | Crea una máquina virtual desde una instantánea creando primero un disco administrado de la instantánea y conectando luego el nuevo disco administrado como disco del sistema operativo. |
|**Administrar el almacenamiento**||
| [Creación de un disco administrado desde un VHD en la misma suscripción o en otra](../scripts/virtual-machines-powershell-sample-create-managed-disk-from-vhd.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) | Crea un disco administrado a partir de un VHD especializado como un disco del sistema operativo, o a partir de un VHD de datos como un disco de datos en la misma suscripción o en otra.  |
| [Crear un disco administrado a partir de una instantánea](../scripts/virtual-machines-powershell-sample-create-managed-disk-from-snapshot.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) | Crea un disco administrado a partir de una instantánea. |
| [Exportación de una instantánea como VHD a una cuenta de almacenamiento](../scripts/virtual-machines-powershell-sample-copy-snapshot-to-storage-account.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) | Exporta una instantánea administrada como VHD a una cuenta de almacenamiento en otra región. |
| [Exportar el disco duro virtual de un disco administrado a una cuenta de almacenamiento](../scripts/virtual-machines-powershell-sample-copy-managed-disks-vhd.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) | Exporta el VHD subyacente de un disco administrado a una cuenta de almacenamiento en otra región. |
| [Creación de una instantánea a partir de un disco duro virtual](../scripts/virtual-machines-powershell-sample-create-snapshot-from-vhd.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) | Crea una instantánea a partir de un VHD y luego usa esa instantánea para crear varios discos administrados idénticos rápidamente.  |
| [Copia de una instantánea en la misma suscripción o en otra](../scripts/virtual-machines-powershell-sample-copy-snapshot-to-same-or-different-subscription.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) | Copia la instantánea en la misma suscripción o en otra, que está en la misma región que la instantánea primaria. |
|**Supervisión de máquinas virtuales**||
| [Supervisión de una máquina virtual con registros de Azure Monitor](./../scripts/virtual-machines-linux-powershell-sample-create-vm-oms.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) | Crea una máquina virtual, instala el agente de Log Analytics e inscribe la máquina virtual en un área de trabajo de Log Analytics.  |
| [Copia de un disco administrado en la misma suscripción o en otra](../scripts/virtual-machines-linux-powershell-sample-copy-managed-disks-to-same-or-different-subscription.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) | Copia un disco administrado en la misma suscripción o en otra, que está en la misma región que el disco administrado primario.
| [Recopilación de detalles sobre todas las máquinas virtuales de una suscripción con PowerShell](../scripts/virtual-machines-powershell-sample-collect-vm-details.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) | Crea un csv que contiene el nombre de la máquina virtual, el nombre del grupo de recursos, la región, la red virtual, la subred, la dirección IP privada, el tipo de sistema operativo y la dirección IP pública de las máquinas virtuales de la suscripción proporcionada.
| | |
