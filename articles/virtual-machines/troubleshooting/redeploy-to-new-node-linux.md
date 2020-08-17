---
title: Nueva implementación de máquinas virtuales Linux en Azure | Microsoft Docs
description: Cómo volver a implementar máquinas virtuales Linux en Azure para solucionar problemas de conexión SSH.
services: virtual-machines-linux
documentationcenter: virtual-machines
author: genlin
manager: dcscontentpm
tags: azure-resource-manager,top-support-issue
ms.assetid: e9530dd6-f5b0-4160-b36b-d75151d99eb7
ms.service: virtual-machines-linux
ms.topic: troubleshooting
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 10/31/2018
ms.author: genli
ms.openlocfilehash: 6b6abaf10f74b29685309ed5a24a5e6b9f261014
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87074442"
---
# <a name="redeploy-linux-virtual-machine-to-new-azure-node"></a>Nueva implementación de una máquina virtual Linux en un nuevo nodo de Azure
Si encuentra dificultades para solucionar los problemas de acceso de SSH o de las aplicaciones a una máquina virtual (VM) Linux en Azure, puede que lo ayude volverla a implementar. Cuando se vuelve a implementar una máquina virtual, se mueve a otro nodo dentro de la infraestructura de Azure y después se vuelve a conectar. Se conservan todas las opciones de configuración y los recursos asociados. En este artículo se muestra cómo volver a implementar una máquina virtual con la CLI de Azure o el Portal de Azure.

> [!NOTE]
> Después de volver a implementar una máquina virtual, se perderá el disco temporal y se actualizarán las direcciones IP dinámicas asociadas con la interfaz de red virtual. 


## <a name="use-the-azure-cli"></a>Uso de la CLI de Azure
Instale la versión más reciente de la [CLI de Azure](/cli/azure/install-az-cli2) e inicie sesión en su cuenta de Azure con [az login](/cli/azure/reference-index).

Vuelva a implementar la máquina virtual con [az vm redeploy](/cli/azure/vm). En el ejemplo siguiente se reimplementa la máquina virtual llamada *myVM* en el grupo de recursos denominado *myResourceGroup*:

```azurecli
az vm redeploy --resource-group myResourceGroup --name myVM 
```

## <a name="use-the-azure-classic-cli"></a>Uso de la CLI de Azure clásica

[!INCLUDE [classic-vm-deprecation](../../../includes/classic-vm-deprecation.md)]


Instale la versión más reciente de la [CLI de Azure clásica](/cli/azure/install-classic-cli) e inicie sesión en su cuenta de Azure. Asegúrese de que está en modo de Resource Manager (`azure config mode arm`).

En el ejemplo siguiente se reimplementa la máquina virtual llamada *myVM* en el grupo de recursos denominado *myResourceGroup*:

```console
azure vm redeploy --resource-group myResourceGroup --vm-name myVM 
```

[!INCLUDE [virtual-machines-common-redeploy-to-new-node](../../../includes/virtual-machines-common-redeploy-to-new-node.md)]

## <a name="next-steps"></a>Pasos siguientes
Puede encontrar ayuda específica sobre la [solución de problemas de las conexiones SSH](troubleshoot-ssh-connection.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) o [pasos detallados para solucionar problemas de SSH](detailed-troubleshoot-ssh-connection.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) si tiene problemas para conectarse a la máquina virtual. También puede leer sobre la [solución de problemas de aplicaciones](troubleshoot-app-connection.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) si no puede acceder a una aplicación que se ejecuta en la máquina virtual.
