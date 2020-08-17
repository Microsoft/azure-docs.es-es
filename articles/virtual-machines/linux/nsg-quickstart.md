---
title: Apertura de puertos a una máquina virtual Linux con CLI de Azure
description: Más información sobre cómo abrir un puerto o crear un punto de conexión a la máquina virtual Linux mediante el modelo de implementación de Azure Resource Manager y la CLI de Azure
author: cynthn
manager: gwallace
ms.service: virtual-machines
ms.subservice: networking
ms.topic: how-to
ms.workload: infrastructure-services
ms.date: 12/13/2017
ms.author: cynthn
ms.custom: devx-track-azurecli
ms.openlocfilehash: 4c57ec2edf43bf0a710f3c15f30836f751c4d52b
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87500170"
---
# <a name="open-ports-and-endpoints-to-a-vm-with-the-azure-cli"></a>Apertura de puertos y puntos de conexión para una máquina virtual Linux en Azure con la CLI de Azure 2.0

En Azure, se abre un puerto o se crea un punto de conexión a una máquina virtual creando un filtro de red en una subred o una interfaz de red de máquina virtual. Estos filtros, que controlan el tráfico entrante y saliente, se colocan en un grupo de seguridad de red y se asocian al recurso que va a recibir dicho tráfico. Vamos a usar un ejemplo común de tráfico web en el puerto 80. En este artículo se muestra cómo abrir un puerto a una VM mediante la CLI de Azure. 


Para crear reglas y un grupo de seguridad de red, necesita la [CLI de Azure](/cli/azure/install-az-cli2) más reciente instalada y con la sesión iniciada en una cuenta de Azure mediante [az login](/cli/azure/reference-index).

En los ejemplos siguientes, reemplace los nombres de parámetros de ejemplo por los suyos propios. Los nombres de parámetro de ejemplo incluyen *myResourceGroup*, *myNetworkSecurityGroup* y *myVnet*.


## <a name="quickly-open-a-port-for-a-vm"></a>Abrir rápidamente un puerto para una máquina virtual
Si necesita abrir rápidamente un puerto para una máquina virtual en un escenario de desarrollo y prueba, puede usar el comando [az vm open-port](/cli/azure/vm). Este comando crea un grupo de seguridad de red, agrega una regla y la aplica a una máquina virtual o subred. En el ejemplo siguiente se abre el puerto *80* en la máquina virtual denominada "*myVM*" que se encuentra en el grupo de recursos *myResourceGroup*.

```azurecli
az vm open-port --resource-group myResourceGroup --name myVM --port 80
```

Para tener más control sobre reglas como, por ejemplo, para definir un intervalo de direcciones IP de origen, siga los pasos adicionales de este artículo.


## <a name="create-a-network-security-group-and-rules"></a>Crear un grupo de seguridad de red (NSG) y las reglas
Cree el grupo de seguridad de red con [az network nsg create](/cli/azure/network/nsg). En el ejemplo siguiente se crea un grupo de seguridad de red denominado *myNetworkSecurityGroup* en la ubicación *eastus*:

```azurecli
az network nsg create \
    --resource-group myResourceGroup \
    --location eastus \
    --name myNetworkSecurityGroup
```

Agregue una regla con [az network nsg rule create](/cli/azure/network/nsg/rule) que permita el tráfico HTTP hacia el servidor web (o adáptela a su propio escenario, por ejemplo, el acceso de SSH o la conectividad de base de datos). En el ejemplo siguiente, se crea una regla denominada *myNetworkSecurityGroupRule* para permitir el tráfico TCP en el puerto 80:

```azurecli
az network nsg rule create \
    --resource-group myResourceGroup \
    --nsg-name myNetworkSecurityGroup \
    --name myNetworkSecurityGroupRule \
    --protocol tcp \
    --priority 1000 \
    --destination-port-range 80
```


## <a name="apply-network-security-group-to-vm"></a>Aplicar el grupo de seguridad de red a la máquina virtual
Asocie el grupo de seguridad de red con la interfaz de red (NIC) de la máquina virtual con [az network nic update](/cli/azure/network/nic). En el ejemplo siguiente, se asocia una NIC existente denominada *myNic* con el grupo de seguridad de red llamado*myNetworkSecurityGroup*:

```azurecli
az network nic update \
    --resource-group myResourceGroup \
    --name myNic \
    --network-security-group myNetworkSecurityGroup
```

Como alternativa, puede asociar el grupo de seguridad de red a una subred de red virtual con [az network vnet subnet update](/cli/azure/network/vnet/subnet), en lugar de únicamente a la interfaz de red en una única máquina virtual. En el ejemplo siguiente, se asocia una subred existente denominado *mySubnet* de la red virtual *myVnet* con el grupo de seguridad de red llamado *myNetworkSecurityGroup*:

```azurecli
az network vnet subnet update \
    --resource-group myResourceGroup \
    --vnet-name myVnet \
    --name mySubnet \
    --network-security-group myNetworkSecurityGroup
```

## <a name="more-information-on-network-security-groups"></a>Más información sobre los grupos de seguridad de red
Los comandos rápidos que se describen aquí le permiten ponerse a trabajar con el flujo de tráfico a la máquina virtual. Los grupos de seguridad de red proporcionan muchas características excelentes y un gran nivel de detalle para controlar el acceso a sus recursos. Puede leer más sobre la [creación de un grupo de seguridad de red y las reglas de ACL aquí](tutorial-virtual-network.md#secure-network-traffic).

Para las aplicaciones web de alta disponibilidad, debe colocar las máquinas virtuales detrás de una instancia de Azure Load Balancer. El equilibrador de carga distribuye el tráfico a las máquinas virtuales, con un grupo de seguridad de red que proporciona el filtrado del tráfico. Para más información, consulte [Equilibrio de la carga de máquinas virtuales Linux en Azure para crear una aplicación de alta disponibilidad](tutorial-load-balancer.md).

## <a name="next-steps"></a>Pasos siguientes
En este ejemplo, se ha creado una regla sencilla para permitir tráfico HTTP. Puede encontrar información sobre la creación de entornos más detallados en los siguientes artículos:

* [Introducción a Azure Resource Manager](../../azure-resource-manager/management/overview.md)
* [¿Qué es un grupo de seguridad de red?](../../virtual-network/security-overview.md)
