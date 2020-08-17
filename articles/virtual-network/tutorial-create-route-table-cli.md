---
title: Enrutamiento del tráfico de red en la CLI de Azure | Microsoft Docs
description: En este artículo, aprenderá a enrutar el tráfico de red con una tabla de rutas mediante la CLI de Azure.
services: virtual-network
documentationcenter: virtual-network
author: KumudD
manager: mtillman
editor: ''
tags: azure-resource-manager
Customer intent: I want to route traffic from one subnet, to a different subnet, through a network virtual appliance.
ms.assetid: ''
ms.service: virtual-network
ms.devlang: azurecli
ms.topic: how-to
ms.tgt_pltfrm: virtual-network
ms.workload: infrastructure
ms.date: 03/13/2018
ms.author: kumud
ms.custom: devx-track-azurecli
ms.openlocfilehash: 027165f797977311fd77f3cd3e626b126c26e47b
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87494679"
---
# <a name="route-network-traffic-with-a-route-table-using-the-azure-cli"></a>Enrutamiento del tráfico de red con una tabla de rutas mediante la CLI de Azure

De forma predeterminada, Azure enruta automáticamente el tráfico entre todas las subredes de una red virtual. Sin embargo, puede crear sus propias rutas para invalidar las predeterminadas de Azure. La posibilidad de crear rutas personalizadas resulta de utilidad si, por ejemplo, quiere enrutar el tráfico entre subredes por medio de una aplicación virtual de red (NVA). En este artículo aprenderá a:

* Creación de una tabla de rutas
* Creación de una ruta
* Creación de una red virtual con varias subredes
* Asociación de una tabla de rutas a una subred
* Creación de una aplicación virtual de red para enrutar el tráfico
* Implementación de máquinas virtuales (VM) en subredes diferentes
* Enrutamiento del tráfico desde una subred a otra a través de una aplicación virtual de red

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

Si decide instalar y usar la CLI en un entorno local, para esta guía de inicio rápido es preciso que ejecute la versión 2.0.28 de la CLI de Azure o una versión posterior. Para encontrar la versión, ejecute `az --version`. Si necesita instalarla o actualizarla, vea [Instalación de la CLI de Azure](/cli/azure/install-azure-cli). 

## <a name="create-a-route-table"></a>Creación de una tabla de rutas

Antes de poder crear una tabla de rutas, debe crear un grupo de recursos con [az group create](/cli/azure/group) para todos los recursos creados en este artículo. 

```azurecli-interactive
# Create a resource group.
az group create \
  --name myResourceGroup \
  --location eastus
```

Cree una tabla de rutas con [az network route-table create](/cli/azure/network/route-table#az-network-route-table-create). En el ejemplo siguiente se crea una tabla de rutas denominada *myRouteTablePublic*. 

```azurecli-interactive
# Create a route table
az network route-table create \
  --resource-group myResourceGroup \
  --name myRouteTablePublic
```

## <a name="create-a-route"></a>Creación de una ruta

Cree una ruta en la tabla de rutas con [az network route-table route create](/cli/azure/network/route-table/route#az-network-route-table-route-create). 

```azurecli-interactive
az network route-table route create \
  --name ToPrivateSubnet \
  --resource-group myResourceGroup \
  --route-table-name myRouteTablePublic \
  --address-prefix 10.0.1.0/24 \
  --next-hop-type VirtualAppliance \
  --next-hop-ip-address 10.0.2.4
```

## <a name="associate-a-route-table-to-a-subnet"></a>Asociación de una tabla de rutas a una subred

Para poder asociar una tabla de rutas a una subred, debe crear una red virtual y una subred. Cree una red virtual con una subred con [az network vnet create](/cli/azure/network/vnet).

```azurecli-interactive
az network vnet create \
  --name myVirtualNetwork \
  --resource-group myResourceGroup \
  --address-prefix 10.0.0.0/16 \
  --subnet-name Public \
  --subnet-prefix 10.0.0.0/24
```

Cree dos subredes adicionales con [az network vnet subnet create](/cli/azure/network/vnet/subnet).

```azurecli-interactive
# Create a private subnet.
az network vnet subnet create \
  --vnet-name myVirtualNetwork \
  --resource-group myResourceGroup \
  --name Private \
  --address-prefix 10.0.1.0/24

# Create a DMZ subnet.
az network vnet subnet create \
  --vnet-name myVirtualNetwork \
  --resource-group myResourceGroup \
  --name DMZ \
  --address-prefix 10.0.2.0/24
```

Asocie la tabla de rutas *myRouteTablePublic* a la subred *Pública* con [az network vnet subnet update](/cli/azure/network/vnet/subnet).

```azurecli-interactive
az network vnet subnet update \
  --vnet-name myVirtualNetwork \
  --name Public \
  --resource-group myResourceGroup \
  --route-table myRouteTablePublic
```

## <a name="create-an-nva"></a>Creación de una aplicación virtual de red

Una aplicación virtual de red es una máquina virtual que realiza una función de red, como el enrutamiento, el firewall o la optimización de la WAN.

Cree una aplicación virtual de red en la subred *DMZ* con [az vm create](/cli/azure/vm). Cuando se crea una máquina virtual, Azure crea una dirección IP pública y la asigna a la máquina virtual de forma predeterminada. El parámetro `--public-ip-address ""` indica a Azure que no cree ni asigne una dirección IP pública a la máquina virtual, dado que no es necesario que esta esté conectada a Internet. Si las claves SSH no existen en la ubicación de claves predeterminada, el comando las crea. Para utilizar un conjunto específico de claves, utilice la opción `--ssh-key-value`.

```azurecli-interactive
az vm create \
  --resource-group myResourceGroup \
  --name myVmNva \
  --image UbuntuLTS \
  --public-ip-address "" \
  --subnet DMZ \
  --vnet-name myVirtualNetwork \
  --generate-ssh-keys
```

La máquina virtual tarda en crearse unos minutos. No continúe con el paso siguiente hasta que Azure termine de crear la máquina virtual y devuelva información sobre ella. 

Para que una interfaz de red pueda reenviar el tráfico de red recibido, que no está destinado a su propia dirección IP, debe tener habilitado el reenvío IP. Habilite el reenvío IP para la interfaz de red con [az network nic update](/cli/azure/network/nic).

```azurecli-interactive
az network nic update \
  --name myVmNvaVMNic \
  --resource-group myResourceGroup \
  --ip-forwarding true
```

Dentro de la máquina virtual, también el sistema operativo o una aplicación en ejecución deben poder reenviar el tráfico de red. Habilite el reenvío IP en el sistema operativo de la máquina virtual con [az vm extension set](/cli/azure/vm/extension):

```azurecli-interactive
az vm extension set \
  --resource-group myResourceGroup \
  --vm-name myVmNva \
  --name customScript \
  --publisher Microsoft.Azure.Extensions \
  --settings '{"commandToExecute":"sudo sysctl -w net.ipv4.ip_forward=1"}'
```

El comando puede tardar un minuto en ejecutarse.

## <a name="create-virtual-machines"></a>Creación de máquinas virtuales

Cree dos máquinas virtuales en la red virtual para poder validar que el tráfico que procede de la subred *Public* se enruta a la subred *Private* mediante la aplicación virtual de red de un paso posterior. 

Cree una máquina virtual en la subred *Public* con [az vm create](/cli/azure/vm). El parámetro `--no-wait` permite a Azure ejecutar el comando en segundo plano para que pueda continuar con el siguiente comando. Para simplificar los pasos de este artículo, se usa una contraseña. Las claves se utilizan habitualmente en las implementaciones de producción. Si usa claves, también debe configurar el reenvío del agente SSH. Para más información, consulte la documentación del cliente SSH. Reemplace `<replace-with-your-password>` en el siguiente comando por una contraseña de su elección.

```azurecli-interactive
adminPassword="<replace-with-your-password>"

az vm create \
  --resource-group myResourceGroup \
  --name myVmPublic \
  --image UbuntuLTS \
  --vnet-name myVirtualNetwork \
  --subnet Public \
  --admin-username azureuser \
  --admin-password $adminPassword \
  --no-wait
```

Cree una máquina virtual en la subred *Private*.

```azurecli-interactive
az vm create \
  --resource-group myResourceGroup \
  --name myVmPrivate \
  --image UbuntuLTS \
  --vnet-name myVirtualNetwork \
  --subnet Private \
  --admin-username azureuser \
  --admin-password $adminPassword
```

La máquina virtual tarda en crearse unos minutos. Después de crear la máquina virtual, la CLI de Azure muestra información similar a la del ejemplo siguiente: 

```output
{
  "fqdns": "",
  "id": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVmPrivate",
  "location": "eastus",
  "macAddress": "00-0D-3A-23-9A-49",
  "powerState": "VM running",
  "privateIpAddress": "10.0.1.4",
  "publicIpAddress": "13.90.242.231",
  "resourceGroup": "myResourceGroup"
}
```

Anote el valor de **publicIpAddress**. En un paso posterior, usaremos esta dirección para obtener acceso a la máquina virtual desde Internet.

## <a name="route-traffic-through-an-nva"></a>Enrutamiento del tráfico a través de una aplicación virtual de red

Use el siguiente comando para crear una sesión de SSH con la máquina virtual *myVmPrivate*. Reemplace *\<publicIpAddress>* con la dirección IP pública de la máquina virtual. En el ejemplo anterior, la dirección IP era *13.90.242.231*.

```bash
ssh azureuser@<publicIpAddress>
```

Cuando se le solicite una contraseña, escriba la que seleccionó en [Creación de máquinas virtuales](#create-virtual-machines).

Use el siguiente comando para instalar traceroute en la máquina virtual *myVmPrivate*:

```bash
sudo apt-get install traceroute
```

Use el siguiente comando para probar el enrutamiento del tráfico de red a la máquina virtual *myVmPublic* desde la máquina virtual *myVmPrivate*.

```bash
traceroute myVmPublic
```

La respuesta es similar al siguiente ejemplo:

```output
traceroute to myVmPublic (10.0.0.4), 30 hops max, 60 byte packets
1  10.0.0.4 (10.0.0.4)  1.404 ms  1.403 ms  1.398 ms
```

Puede ver si el tráfico se enruta directamente de la máquina virtual *myVmPrivate* a la máquina virtual *myVmPublic*. Las rutas predeterminadas de Azure enrutan el tráfico entre las subredes. 

Use el siguiente comando para usar SSH en la máquina virtual *myVmPublic* desde la máquina virtual *myVmPrivate*:

```bash
ssh azureuser@myVmPublic
```

Use el siguiente comando para instalar traceroute en la máquina virtual *myVmPublic*:

```bash
sudo apt-get install traceroute
```

Use el siguiente comando para probar el enrutamiento del tráfico de red a la máquina virtual *myVmPrivate* desde la máquina virtual *myVmPublic*.

```bash
traceroute myVmPrivate
```

La respuesta es similar al siguiente ejemplo:

```output
traceroute to myVmPrivate (10.0.1.4), 30 hops max, 60 byte packets
1  10.0.2.4 (10.0.2.4)  0.781 ms  0.780 ms  0.775 ms
2  10.0.1.4 (10.0.0.4)  1.404 ms  1.403 ms  1.398 ms
```

Puede ver que el primer salto es 10.0.2.4, que es la dirección IP privada de la aplicación virtual de red. El segundo salto es 10.0.1.4, la dirección IP privada de la máquina virtual *myVmPrivate*. La ruta agregada a la tabla de rutas *myRouteTablePublic* y asociada a la subred *Pública* hizo que Azure enrutara el tráfico mediante la NVA, en lugar de a la subred *Privada* directamente.

Cierre las sesiones SSH tanto de la máquina virtual *myVmPublic* como de *myVmPrivate*.

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no se necesiten, use [az group delete](/cli/azure/group) para quitar el grupo de recursos y todos los recursos que contenga.

```azurecli-interactive
az group delete --name myResourceGroup --yes
```

## <a name="next-steps"></a>Pasos siguientes

En este artículo, creó una tabla de rutas y la asoció a una subred. Creó una aplicación virtual de red sencilla que enrutó el tráfico desde una subred pública hasta una subred privada. Puede implementar diversas aplicaciones virtuales de red preconfiguradas que realicen funciones de red, como son la optimización de la WAN y la de firewall, desde [Azure Marketplace](https://azuremarketplace.microsoft.com/marketplace/apps/category/networking). Para más información acerca del enrutamiento, consulte [Introducción al enrutamiento](virtual-networks-udr-overview.md) y [Administración de una tabla de rutas](manage-route-table.md).

Aunque puede implementar muchos recursos de Azure en una red virtual, no es el caso de los recursos de algunos servicios de PaaS de Azure. Pero puede restringir el acceso a los recursos de algunos servicios de PaaS de Azure solo al tráfico que procede de una subred de una red virtual. Para saber cómo hacerlo, consulte [Restricción del acceso de red a los recursos de PaaS](tutorial-restrict-network-access-to-resources-cli.md).
