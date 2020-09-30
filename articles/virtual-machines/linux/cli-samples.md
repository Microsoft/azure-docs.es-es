---
title: Ejemplos de la CLI de Azure
description: Ejemplos de la CLI para máquinas virtuales de Azure
author: cynthn
ms.service: virtual-machines-linux
ms.topic: how-to
ms.date: 03/01/2019
ms.author: cynthn
ms.custom: devx-track-azurecli
ms.openlocfilehash: 63c420a7ae082cbf38b529e71d8324550a84b7bd
ms.sourcegitcommit: 5ed504a9ddfbd69d4f2d256ec431e634eb38813e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89319175"
---
# <a name="azure-cli-samples-for-linux-virtual-machines"></a>Ejemplos de la CLI de Azure para máquinas virtuales Linux

En la tabla siguiente se incluyen vínculos a scripts de Bash creados con la CLI de Azure.

| Script | Descripción |
|---|---|
|**Creación de máquinas virtuales**||
| [Creación de una máquina virtual](./../scripts/virtual-machines-linux-cli-sample-create-vm-quick-create.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) | Crea una máquina virtual Linux con una configuración mínima. |
| [Creación una máquina virtual completamente configurada](./../scripts/virtual-machines-linux-cli-sample-create-vm.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) | Crea un grupo de recursos, una máquina virtual y todos los recursos relacionados.|
| [Creación de máquinas virtuales de alta disponibilidad](./../scripts/virtual-machines-linux-cli-sample-nlb.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) | Crea varias máquinas virtuales de alta disponibilidad y una configuración de equilibrio de carga. |
| [Creación de una máquina virtual y ejecución de un script de configuración](./../scripts/virtual-machines-linux-cli-sample-create-vm-nginx.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) | Crea una máquina virtual y usa la extensión de script personalizado de Azure para instalar NGINX. |
| [Creación de una máquina virtual con WordPress instalado](./../scripts/virtual-machines-linux-cli-sample-create-vm-wordpress.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) | Crea una máquina virtual y usa la extensión de script personalizado de Azure para instalar WordPress. |
| [Creación de una máquina virtual desde un disco administrado](./../scripts/virtual-machines-linux-cli-sample-create-vm-from-managed-os-disks.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) | Crea una máquina virtual conectando un disco administrado como disco del SO. |
| [Creación de una VM a partir de una instantánea](./../scripts/virtual-machines-linux-cli-sample-create-vm-from-snapshot.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) | Crea una máquina virtual desde una instantánea creando primero un disco administrado de instantánea y asociando después el nuevo disco administrado como disco del sistema operativo. |
|**Administrar el almacenamiento**||
| [Crear disco administrado desde un disco duro virtual](../scripts/virtual-machines-cli-sample-create-managed-disk-from-vhd.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) | Crea un disco administrado desde un disco duro virtual especializado como un disco del sistema operativo, o desde un disco duro virtual de datos como un disco de datos.  |
| [Crear un disco administrado a partir de una instantánea](../scripts/virtual-machines-cli-sample-create-managed-disk-from-snapshot.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) | Crea un disco administrado a partir de una instantánea. |
| [Copiar el disco administrado en la misma suscripción o en otra](../scripts/virtual-machines-cli-sample-copy-managed-disks-to-same-or-different-subscription.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) | Copia el disco administrado en la misma u otra suscripción pero en la misma región que el disco administrado primario. 
| [Exportar una instantánea como un disco duro virtual a una cuenta de almacenamiento](../scripts/virtual-machines-cli-sample-copy-snapshot-to-storage-account.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) | Exporta una instantánea administrada como un disco duro virtual a una cuenta de almacenamiento en una región diferente. |
| [Exportar el disco duro virtual de un disco administrado a una cuenta de almacenamiento](../scripts/virtual-machines-cli-sample-copy-managed-disks-vhd.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) | Exporta el disco duro virtual subyacente de un disco administrado a una cuenta de almacenamiento de otra región. |
| [Copiar una instantánea en la misma suscripción o en otra](../scripts/virtual-machines-cli-sample-copy-snapshot-to-same-or-different-subscription.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) | Copia la instantánea en la misma u otra suscripción, pero en la misma región que la instantánea primaria. |
|**Máquinas virtuales de red**||
| [Protección del tráfico de red entre máquinas virtuales](./../scripts/virtual-machines-linux-cli-sample-create-vm-nsg.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) | Crea dos máquinas virtuales, todos los recursos relacionados y grupos de seguridad de red internos y externos (NSG). |
|**Protección de las máquinas virtuales**||
| [Cifrado de una máquina virtual y discos de datos](./../scripts/virtual-machines-linux-cli-sample-encrypt-vm.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) | Crea una instancia de Azure Key Vault, una clave de cifrado y una entidad de servicio, y luego cifra una máquina virtual. |
|**Supervisión de máquinas virtuales**||
| [Supervisión de una máquina virtual con registros de Azure Monitor](./../scripts/virtual-machines-linux-cli-sample-create-vm-oms.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) | Crea una máquina virtual, instala el agente de Log Analytics e inscribe la máquina virtual en un área de trabajo de Log Analytics.  |
|**Solución de problemas de máquinas virtuales**||
| [Solución de problemas de un disco de sistema operativo de máquina virtual](./../scripts/virtual-machines-linux-cli-sample-mount-os-disk.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) | Permite montar el disco del sistema operativo a partir de una máquina virtual como un disco de datos en una segunda máquina virtual. |
| | |
