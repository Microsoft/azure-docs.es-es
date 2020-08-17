---
title: 'Ejemplo de script de la CLI de Azure: creación de dos máquinas virtuales con un grupo de seguridad de red interno y externo'
description: 'Ejemplo de script de CLI de Azure: creación de dos máquinas virtuales con un NSG interno y externo'
services: virtual-machines-windows
documentationcenter: virtual-machines
author: rickstercdn
manager: gwallace
tags: ''
ms.assetid: ''
ms.service: virtual-machines-windows
ms.devlang: azurecli
ms.topic: sample
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure
ms.date: 02/23/2017
ms.author: rclaus
ms.custom: mvc, devx-track-azurecli
ms.openlocfilehash: 0d0e3dd53c592c991e342240b9c973e1677969d6
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87498385"
---
# <a name="secure-network-traffic-between-virtual-machines"></a>Protección del tráfico de red entre máquinas virtuales

Este script crea dos máquinas virtuales y protege el tráfico entrante en ambas. Una máquina virtual es accesible en Internet y tiene un grupo de seguridad de red (NSG) configurado para permitir el tráfico en el puerto 80 y el puerto 3389. La segunda máquina virtual no es accesible en Internet, y tiene un NSG configurado para permitir el tráfico únicamente a la primera máquina virtual. 

[!INCLUDE [sample-cli-install](../../../includes/sample-cli-install.md)]

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="sample-script"></a>Script de ejemplo

[!code-azurecli-interactive[main](../../../cli_scripts/virtual-machine/create-vm-nsg/create-windows-vm-nsg.sh "Create VM with NSG")]

## <a name="clean-up-deployment"></a>Limpieza de la implementación 

Ejecute el siguiente comando para quitar el grupo de recursos, la máquina virtual y todos los recursos relacionados.

```azurecli-interactive 
az group delete --name myResourceGroup --yes
```

## <a name="script-explanation"></a>Explicación del script

Este script usa los siguientes comandos para crear un grupo de recursos, una máquina virtual y todos los recursos relacionados. Cada comando de la tabla crea un vínculo a documentación específica del comando.

| Get-Help | Notas |
|---|---|
| [az group create](/cli/azure/group) | Crea un grupo de recursos en el que se almacenan todos los recursos. |
| [az network vnet create](/cli/azure/network/vnet) | Crea una red virtual y una subred de Azure. |
| [az network vnet subnet create](/cli/azure/network/vnet/subnet) | Crea una subred. |
| [az vm create](/cli/azure/vm) | Crea la máquina virtual y la conecta con la tarjeta de red, la red virtual, la subred y el NSG. Este comando también especifica la imagen de máquina virtual que se usará, y las credenciales administrativas.  |
| [az network nsg rule update](/cli/azure/network/nsg/rule) | Actualiza una regla de NSG. En este ejemplo, la regla de back-end se actualiza para que solo pase el tráfico de la subred de front-end. |
| [az network nsg rule list](/cli/azure/network/nsg/rule) | Devuelve información acerca de una regla de grupo de seguridad de red. En este ejemplo, el nombre de la regla se almacena en una variable para usarlo más adelante en el script. |
| [az group delete](/cli/azure/vm/extension) | Elimina un grupo de recursos, incluidos todos los recursos anidados. |

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre la CLI de Azure, consulte la [documentación de la CLI de Azure](/cli/azure).

Encontrará más ejemplos de scripts de la CLI de máquina virtual en la [documentación sobre máquinas virtuales Windows de Azure](../windows/cli-samples.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json).
