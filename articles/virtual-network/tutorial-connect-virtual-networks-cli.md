---
title: 'Conexión de redes virtuales con emparejamiento de VNet: CLI de Azure'
description: En este artículo, obtendrá información sobre cómo conectar las redes virtuales con emparejamiento de redes virtuales usando la CLI de Azure.
services: virtual-network
documentationcenter: virtual-network
author: KumudD
tags: azure-resource-manager
Customer intent: I want to connect two virtual networks so that virtual machines in one virtual network can communicate with virtual machines in the other virtual network.
ms.assetid: ''
ms.service: virtual-network
ms.devlang: azurecli
ms.topic: how-to
ms.tgt_pltfrm: virtual-network
ms.workload: infrastructure
ms.date: 03/13/2018
ms.author: kumud
ms.custom: devx-track-azurecli
ms.openlocfilehash: 855ea936ff91d4c22b0670cd989f91c692c567c8
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87502603"
---
# <a name="connect-virtual-networks-with-virtual-network-peering-using-the-azure-cli"></a>Conexión de redes virtuales con emparejamiento de redes virtuales con la CLI de Azure

Puede conectar redes virtuales entre sí con el emparejamiento de redes virtuales. Una vez que las redes virtuales están emparejadas, los recursos de ambas se pueden comunicar entre sí con el mismo ancho de banda y la misma latencia que si estuvieran en la misma red virtual. En este artículo aprenderá a:

* Crear dos redes virtuales
* Conectar dos redes virtuales con el emparejamiento de redes virtuales
* Implementar una máquina virtual (VM) en cada red virtual
* Comunicarse entre máquinas virtuales

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

Si decide instalar y usar la CLI localmente, para este artículo es preciso que ejecute la versión 2.0.28 o posterior de la CLI de Azure. Para encontrar la versión, ejecute `az --version`. Si necesita instalarla o actualizarla, vea [Instalación de la CLI de Azure](/cli/azure/install-azure-cli). 

## <a name="create-virtual-networks"></a>Creación de redes virtuales

Antes de crear una red virtual, cree un grupo de recursos para ella y los demás recursos que se crearon en este artículo. Cree un grupo de recursos con [az group create](/cli/azure/group). En el ejemplo siguiente, se crea un grupo de recursos denominado *myResourceGroup* en la ubicación *eastus*.

```azurecli-interactive 
az group create --name myResourceGroup --location eastus
```

Cree la red virtual con el comando [az network vnet create](/cli/azure/network/vnet). En el ejemplo siguiente se crea una red virtual denominada *myVirtualNetwork1* con el prefijo de dirección *10.0.0.0/16*.

```azurecli-interactive 
az network vnet create \
  --name myVirtualNetwork1 \
  --resource-group myResourceGroup \
  --address-prefixes 10.0.0.0/16 \
  --subnet-name Subnet1 \
  --subnet-prefix 10.0.0.0/24
```

Cree una red virtual denominada *myVirtualNetwork2* con el prefijo de dirección *10.1.0.0/16*:

```azurecli-interactive 
az network vnet create \
  --name myVirtualNetwork2 \
  --resource-group myResourceGroup \
  --address-prefixes 10.1.0.0/16 \
  --subnet-name Subnet1 \
  --subnet-prefix 10.1.0.0/24
```

## <a name="peer-virtual-networks"></a>Emparejamiento de redes virtuales

Los emparejamientos se establecen entre los identificadores de red virtual, por lo que primero debe obtener el identificador de cada red virtual con [az network vnet show](/cli/azure/network/vnet) y almacenarlo en una variable.

```azurecli-interactive
# Get the id for myVirtualNetwork1.
vNet1Id=$(az network vnet show \
  --resource-group myResourceGroup \
  --name myVirtualNetwork1 \
  --query id --out tsv)

# Get the id for myVirtualNetwork2.
vNet2Id=$(az network vnet show \
  --resource-group myResourceGroup \
  --name myVirtualNetwork2 \
  --query id \
  --out tsv)
```

Cree un emparejamiento de *myVirtualNetwork1* con *myVirtualNetwork2* con [az network vnet peering create](/cli/azure/network/vnet/peering). Si el parámetro `--allow-vnet-access` no se especifica, se establece un emparejamiento, pero no fluye la comunicación.

```azurecli-interactive
az network vnet peering create \
  --name myVirtualNetwork1-myVirtualNetwork2 \
  --resource-group myResourceGroup \
  --vnet-name myVirtualNetwork1 \
  --remote-vnet $vNet2Id \
  --allow-vnet-access
```

En la salida que se devuelve al ejecutarse el comando anterior, verá que **peeringState** está en estado *iniciado*. El emparejamiento permanece en estado *Iniciado* hasta que cree el emparejamiento de *myVirtualNetwork2* con *myVirtualNetwork1*. Cree un emparejamiento de *myVirtualNetwork2* con *myVirtualNetwork1*. 

```azurecli-interactive
az network vnet peering create \
  --name myVirtualNetwork2-myVirtualNetwork1 \
  --resource-group myResourceGroup \
  --vnet-name myVirtualNetwork2 \
  --remote-vnet $vNet1Id \
  --allow-vnet-access
```

En la salida que se devuelve al ejecutarse el comando anterior, verá que **peeringState** está en estado *conectado*. Azure también cambia el estado del emparejamiento *myVirtualNetwork1-myVirtualNetwork2* a *Conectado*. Confirme que el estado de del emparejamiento *myVirtualNetwork1-myVirtualNetwork2* cambia a *conectado* con [az network vnet peering show](/cli/azure/network/vnet/peering).

```azurecli-interactive
az network vnet peering show \
  --name myVirtualNetwork1-myVirtualNetwork2 \
  --resource-group myResourceGroup \
  --vnet-name myVirtualNetwork1 \
  --query peeringState
```

Los recursos de una red virtual no se comunican con los de la otra hasta que el estado **peeringState** de los emparejamientos de ambas redes virtuales es *conectado*. 

## <a name="create-virtual-machines"></a>Creación de máquinas virtuales

Cree una máquina virtual en cada red virtual para que puedan comunicarse entre ellas en un paso posterior.

### <a name="create-the-first-vm"></a>Creación de la primera máquina virtual

Cree la máquina virtual con [az vm create](/cli/azure/vm). En el ejemplo siguiente se crea una máquina virtual llamada *myVm1* en la red virtual *myVirtualNetwork1*. Si las claves SSH no existen en la ubicación de claves predeterminada, el comando las crea. Para utilizar un conjunto específico de claves, utilice la opción `--ssh-key-value`. La opción `--no-wait` crea la máquina virtual en segundo plano, así que puede continuar con el siguiente paso.

```azurecli-interactive
az vm create \
  --resource-group myResourceGroup \
  --name myVm1 \
  --image UbuntuLTS \
  --vnet-name myVirtualNetwork1 \
  --subnet Subnet1 \
  --generate-ssh-keys \
  --no-wait
```

### <a name="create-the-second-vm"></a>Creación de la segunda máquina virtual

Cree una red virtual en la red virtual *myVirtualNetwork2*.

```azurecli-interactive
az vm create \
  --resource-group myResourceGroup \
  --name myVm2 \
  --image UbuntuLTS \
  --vnet-name myVirtualNetwork2 \
  --subnet Subnet1 \
  --generate-ssh-keys
```

La máquina virtual tarda en crearse unos minutos. Después de crear la máquina virtual, la CLI de Azure muestra información similar a la del ejemplo siguiente: 

```output
{
  "fqdns": "",
  "id": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVm2",
  "location": "eastus",
  "macAddress": "00-0D-3A-23-9A-49",
  "powerState": "VM running",
  "privateIpAddress": "10.1.0.4",
  "publicIpAddress": "13.90.242.231",
  "resourceGroup": "myResourceGroup"
}
```

Anote el valor de **publicIpAddress**. En un paso posterior, usaremos esta dirección para obtener acceso a la máquina virtual desde Internet.

## <a name="communicate-between-vms"></a>Comunicarse entre máquinas virtuales

Use el siguiente comando para crear una sesión de SSH con la máquina virtual *myVm2*. Reemplace `<publicIpAddress>` con la dirección IP pública de la máquina virtual. En el ejemplo anterior, la dirección IP pública es *13.90.242.231*.

```bash
ssh <publicIpAddress>
```

Haga ping a la máquina virtual en *myVirtualNetwork1*.

```bash
ping 10.0.0.4 -c 4
```

Recibirá cuatro respuestas. 

Cierre la sesión de SSH en la máquina virtual *myVm2*. 

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no se necesiten, use [az group delete](/cli/azure/group) para quitar el grupo de recursos y todos los recursos que contenga.

```azurecli-interactive
az group delete --name myResourceGroup --yes
```

## <a name="next-steps"></a>Pasos siguientes

En este artículo, ha aprendido a conectar dos redes, de la misma región de Azure, con el emparejamiento de redes virtuales. También puede emparejar redes virtuales de distintas [regiones compatibles](virtual-network-manage-peering.md#cross-region) y en [distintas suscripciones de Azure](create-peering-different-subscriptions.md#cli), además de crear [diseños de red de concentrador y radio](/azure/architecture/reference-architectures/hybrid-networking/hub-spoke#virtual-network-peering) con emparejamiento. Para más información sobre el emparejamiento de redes virtuales, consulte los artículos sobre el [emparejamiento de redes virtuales](virtual-network-peering-overview.md) y la [administración de emparejamientos de redes virtuales](virtual-network-manage-peering.md).

Puede [conectar su equipo a una red virtual](../vpn-gateway/vpn-gateway-howto-point-to-site-resource-manager-portal.md?toc=%2fazure%2fvirtual-network%2ftoc.json) mediante VPN e interactuar con recursos en una red virtual, o en redes virtuales emparejadas. En el caso de los scripts reutilizables para completar muchas de las tareas descritas en los artículos sobre las redes virtuales, consulte los [ejemplos de script](cli-samples.md).
