---
title: 'Equilibrio de carga entre varios sitios web con la CLI de Azure: Azure Load Balancer'
description: En este ejemplo de script de la CLI de Azure se muestra cómo equilibrar la carga entre varios sitios web con la misma máquina virtual
documentationcenter: load-balancer
author: asudbring
ms.service: load-balancer
ms.devlang: azurecli
ms.topic: sample
ms.workload: infrastructure
ms.date: 04/20/2018
ms.author: allensu
ms.custom: devx-track-azurecli
ms.openlocfilehash: 8be0dd6e68d6d50ef80c75ffa87974a63d2e0b36
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87501770"
---
# <a name="azure-cli-script-example-load-balance-multiple-websites"></a>Ejemplo de script de la CLI de Azure: Equilibrio de carga entre varios sitios web

En este ejemplo de script de la CLI de Azure se crea una red virtual con dos máquinas virtuales que son miembros de un conjunto de disponibilidad. Un equilibrador de carga dirige el tráfico de dos direcciones IP independientes a las dos máquinas virtuales. Después de ejecutar el script, podría implementar software de servidor web en las máquinas virtuales y hospedar varios sitios web, cada uno con su propia dirección IP.

[!INCLUDE [sample-cli-install](../../../includes/sample-cli-install.md)]

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="sample-script"></a>Script de ejemplo


[!code-azurecli-interactive[main](../../../cli_scripts/load-balancer/load-balance-multiple-web-sites-vm/load-balance-multiple-web-sites-vm.sh  "Load balance multiple web sites")]

## <a name="clean-up-deployment"></a>Limpieza de la implementación 

Ejecute el siguiente comando para quitar el grupo de recursos, la máquina virtual y todos los recursos relacionados.

```azurecli
az group delete --name myResourceGroup --yes
```

## <a name="script-explanation"></a>Explicación del script

Este script usa los siguientes comandos para crear un grupo de recursos, una red virtual, un equilibrador de carga y todos los recursos relacionados. Cada comando de la tabla crea un vínculo a documentación específica del comando.

| Get-Help | Notas |
|---|---|
| [az group create](https://docs.microsoft.com/cli/azure/group#az-group-create) | Crea un grupo de recursos en el que se almacenan todos los recursos. |
| [az network vnet create](https://docs.microsoft.com/cli/azure/network/vnet#az-network-vnet-create) | Crea una red virtual y una subred de Azure. |
| [az network public-ip create](https://docs.microsoft.com/cli/azure/network/public-ip#az-network-public-ip-create) | Crea una dirección IP pública con una dirección IP estática y un nombre DNS asociado. |
| [az network lb create](https://docs.microsoft.com/cli/azure/network/lb#az-network-lb-create) | Crea una instancia de Azure Load Balancer. |
| [az network lb probe create](https://docs.microsoft.com/cli/azure/network/lb/probe#az-network-lb-probe-create) | Crea un sondeo de equilibrador de carga. Un sondeo de equilibrador de carga se usa para supervisar cada máquina virtual del conjunto de equilibrador de carga. Si alguna máquina virtual deja de estar accesible, el tráfico no se enruta a la máquina virtual. |
| [az network lb rule create](https://docs.microsoft.com/cli/azure/network/lb/rule#az-network-lb-rule-create) | Crea una regla de equilibrador de carga. En este ejemplo, se crea una regla para el puerto 80. Según va llegando el tráfico HTTP al equilibrador de carga, se enruta al puerto 80 de una de las máquinas virtuales del conjunto de equilibrador de carga. |
| [az network lb frontend-ip create](https://docs.microsoft.com/cli/azure/network/lb/frontend-ip#az-network-lb-frontend-ip-create) | Crea una dirección IP de front-end para el equilibrador de carga. |
| [az network lb address-pool create](https://docs.microsoft.com/cli/azure/network/lb/address-pool#az-network-lb-address-pool-create) | Crea un grupo de direcciones de back-end. |
| [az network nic create](https://docs.microsoft.com/cli/azure/network/nic#az-network-nic-create) | Crea una tarjeta de red virtual y la conecta con la red virtual y la subred. |
| [az vm availability-set create](https://docs.microsoft.com/cli/azure/network/lb/rule#az-network-lb-rule-create) | Crea un conjunto de disponibilidad. Los conjuntos de disponibilidad garantizan la disponibilidad de las aplicaciones al repartir las máquinas virtuales entre los recursos físicos, de forma que si se produce un error, el conjunto no se verá afectado en su totalidad. |
| [az network nic ip-config create](https://docs.microsoft.com/cli/azure/network/nic/ip-config#az-network-nic-ip-config-create) | Crea una configuración IP. Debe tener la característica de Microsoft.Network/AllowMultipleIpConfigurationsPerNic habilitada en su suscripción. Solo se puede designar una configuración como configuración IP primaria por adaptador de red, mediante la marca --make-primary. |
| [az vm create](https://docs.microsoft.com/cli/azure/vm/availability-set#az-vm-availability-set-create) | Crea la máquina virtual y la conecta con la tarjeta de red, la red virtual, la subred y el NSG. Este comando también especifica la imagen de máquina virtual que se usará, y las credenciales administrativas.  |
| [az group delete](https://docs.microsoft.com/cli/azure/vm/extension#az-vm-extension-set) | Elimina un grupo de recursos, incluidos todos los recursos anidados. |

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre la CLI de Azure, consulte la [documentación de la CLI de Azure](https://docs.microsoft.com/cli/azure).

En la [documentación de la información general de redes de Azure](../cli-samples.md?toc=%2fazure%2fnetworking%2ftoc.json) puede encontrar ejemplos adicionales de script de la CLI de redes.
