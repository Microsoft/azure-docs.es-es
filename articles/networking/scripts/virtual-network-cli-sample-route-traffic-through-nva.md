---
title: 'Ejemplo de script de la CLI de Azure: enrutamiento del tráfico mediante una aplicación virtual de red'
description: 'Ejemplo de script de la CLI de Azure: redirigir el tráfico a través de una aplicación virtual de red de firewall.'
services: virtual-network
documentationcenter: virtual-network
author: KumudD
manager: mtillman
ms.service: virtual-network
ms.devlang: azurecli
ms.topic: article
ms.tgt_pltfrm: ''
ms.workload: infrastructure
ms.date: 07/07/2017
ms.author: kumud
ms.custom: devx-track-azurecli
ms.openlocfilehash: a349f8160e8ab5b6459b2085e21e7368570c57db
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87503844"
---
# <a name="use-an-azure-cli-script-to-route-traffic-through-a-network-virtual-appliance"></a>Uso de un script de la CLI de Azure para enrutar el tráfico a través de una aplicación virtual de red

Este ejemplo de script crea una red virtual con subredes de front-end y back-end. También crea una VM con el reenvío IP habilitado para redirigir el tráfico entre las dos subredes. Después de ejecutar el script puede implementar software de red, como una aplicación de firewall, en la VM.

[!INCLUDE [sample-cli-install](../../../includes/sample-cli-install.md)]

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]


## <a name="sample-script"></a>Script de ejemplo


[!code-azurecli-interactive[main](../../../cli_scripts/virtual-network/route-traffic-through-nva/route-traffic-through-nva.sh "Route traffic through a network virtual appliance")]

## <a name="clean-up-deployment"></a>Limpieza de la implementación 

Ejecute el siguiente comando para quitar el grupo de recursos, la máquina virtual y todos los recursos relacionados.

```azurecli
az group delete --name MyResourceGroup --yes
```

## <a name="script-explanation"></a>Explicación del script

Este script usa los siguientes comandos para crear un grupo de recursos, una red virtual y grupos de seguridad de red. Cada comando de la tabla crea un vínculo a documentación específica del comando.

| Get-Help | Notas |
|---|---|
| [az group create](/cli/azure/group) | Crea un grupo de recursos en el que se almacenan todos los recursos. |
| [az network vnet create](/cli/azure/network/vnet) | Crea una subred de front-end y una red virtual de Azure. |
| [az network subnet create](/cli/azure/network/vnet/subnet) | Crea subredes de back-end y de red perimetral. |
| [az network public-ip create](/cli/azure/network/public-ip) | Crea una dirección IP pública para acceder a la VM desde Internet. |
| [az network nic create](/cli/azure/network/nic) | Crea una interfaz de red virtual y habilita el reenvío IP para ella. |
| [az network nsg create](/cli/azure/network/nsg) | Crea un grupo de seguridad de red (NSG). |
| [az network nsg rule create](/cli/azure/network/nsg/rule) | Crea reglas NSG que permiten puertos HTTP y HTTPS entrantes en la VM. |
| [az network vnet subnet update](/cli/azure/network/vnet/subnet)| Asocia los NSG y tablas de rutas a las subredes. |
| [az network route-table create](/cli/azure/network/route-table#az-network-route-table-create)| Crea una tabla de rutas para todas las rutas. |
| [az network route-table route create](/cli/azure/network/route-table/route#az-network-route-table-route-create)| Crea rutas para redirigir el tráfico entre subredes e Internet a través de la VM. |
| [az vm create](/cli/azure/vm) | Crea una máquina virtual y conecta la NIC a ella. Este comando también especifica la imagen de máquina virtual que se usará y las credenciales administrativas. |
| [az group delete](/cli/azure/group) | Elimina un grupo de recursos y todos los recursos que contiene. |

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre la CLI de Azure, consulte la [documentación de la CLI de Azure](/cli/azure).

En la [documentación de la información general de redes de Azure](../cli-samples.md) puede encontrar ejemplos adicionales de script de la CLI de redes.
