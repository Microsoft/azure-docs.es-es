---
title: 'Implementación de una aplicación de pila doble IPv6 con Standard Load Balancer: CLI'
titlesuffix: Azure Virtual Network
description: En este artículo, se explica cómo se implementa una aplicación de pila doble IPv6 en Azure Virtual Network mediante la CLI de Azure.
services: virtual-network
documentationcenter: na
author: KumudD
manager: mtillman
ms.service: virtual-network
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 03/31/2020
ms.author: kumud
ms.openlocfilehash: d709bd1c68608720e5fa354b23d2b457e99b1ade
ms.sourcegitcommit: 07166a1ff8bd23f5e1c49d4fd12badbca5ebd19c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/15/2020
ms.locfileid: "90087412"
---
# <a name="deploy-an-ipv6-dual-stack-application-in-azure-virtual-network---cli"></a>Implementación de una aplicación de pila doble IPv6 en Azure Virtual Network: CLI

En este artículo, se explica cómo se implementa en Azure una aplicación de pila doble (IPv4 + IPv6) con Standard Load Balancer que contiene una red virtual de pila doble y una subred de pila doble, una instancia de Standard Load Balancer con configuraciones de front-end duales (IPv4 + IPv6), VM con NIC que tienen una configuración de IP dual, reglas de grupo de seguridad de red dual e IP públicas duales.

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) ahora.

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

Si, en su lugar, decide instalar y usar la CLI de Azure en un entorno local, para esta guía de inicio rápido se necesita la versión 2.0.49 de la CLI de Azure o una versión posterior. Ejecute `az --version` para buscar la versión instalada. Consulte [Instalación de la CLI de Azure](/cli/azure/install-azure-cli) para obtener información sobre la instalación o actualización.

## <a name="create-a-resource-group"></a>Crear un grupo de recursos

Para poder generar la red virtual de doble pila, debe crear primero un grupo de recursos con [az group create](/cli/azure/group). En el ejemplo siguiente, se crea un grupo de recursos denominado *DsResourceGroup01* en la ubicación *eastus*:

```azurecli-interactive
az group create \
--name DsResourceGroup01 \
--location eastus
```

## <a name="create-ipv4-and-ipv6-public-ip-addresses-for-load-balancer"></a>Creación de direcciones IP públicas IPv4 e IPv6 para el equilibrador de carga
Para obtener acceso los puntos de conexión IPv4 e IPv6 en Internet, necesita direcciones IP públicas IPv4 e IPv6 para el equilibrador de carga. Cree una dirección IP pública con [az network public-ip create](/cli/azure/network/public-ip). En el ejemplo siguiente, se crean unas direcciones IP públicas IPv4 e IPv6 denominadas *dsPublicIP_v4* y *dsPublicIP_v6* en el grupo de recursos *DsResourceGroup01*:

```azurecli-interactive
# Create an IPV4 IP address
az network public-ip create \
--name dsPublicIP_v4  \
--resource-group DsResourceGroup01  \
--location eastus  \
--sku STANDARD  \
--allocation-method static  \
--version IPv4

# Create an IPV6 IP address
az network public-ip create \
--name dsPublicIP_v6  \
--resource-group DsResourceGroup01  \
--location eastus \
--sku STANDARD  \
--allocation-method static  \
--version IPv6

```

## <a name="create-public-ip-addresses-for-vms"></a>Creación de direcciones IP públicas para las VM

Para acceder de forma remota a las VM en Internet, necesita las direcciones IP públicas IPv4 de las VM. Cree una dirección IP pública con [az network public-ip create](/cli/azure/network/public-ip).

```azurecli-interactive
az network public-ip create \
--name dsVM0_remote_access  \
--resource-group DsResourceGroup01 \
--location eastus  \
--sku Standard  \
--allocation-method static  \
--version IPv4

az network public-ip create \
--name dsVM1_remote_access  \
--resource-group DsResourceGroup01  \
--location eastus  \
--sku Standard  \
--allocation-method static  \
--version IPv4
```

## <a name="create-standard-load-balancer"></a>Creación de Standard Load Balancer

En esta sección, configurará la dirección IP de front-end doble (IPv4 e IPv6) y el grupo de direcciones de back-end del equilibrador de carga, además de crear una instancia de Standard Load Balancer.

### <a name="create-load-balancer"></a>Creación de un equilibrador de carga

Cree una instancia de Standard Load Balancer con [az network lb create](https://docs.microsoft.com/cli/azure/network/lb?view=azure-cli-latest) denominada **dsLB** que incluya un grupo de servidores front-end llamado **dsLbFrontEnd_v4** y un grupo de servidores back-end llamado **dsLbBackEndPool_v4** que esté asociado a la dirección IP pública IPv4 **dsPublicIP_v4** que creó en el paso anterior. 

```azurecli-interactive
az network lb create \
--name dsLB  \
--resource-group DsResourceGroup01 \
--sku Standard \
--location eastus \
--frontend-ip-name dsLbFrontEnd_v4  \
--public-ip-address dsPublicIP_v4  \
--backend-pool-name dsLbBackEndPool_v4
```

### <a name="create-ipv6-frontend"></a>Creación de un servidor front-end IPv6

Cree una dirección IP de front-end IPV6 con [az network lb frontend-ip create](https://docs.microsoft.com/cli/azure/network/lb/frontend-ip?view=azure-cli-latest#az-network-lb-frontend-ip-create). En el ejemplo siguiente, se crea una configuración de direcciones IP de front-end llamada *dsLbFrontEnd_v6* y se asocia a la dirección *dsPublicIP_v6*:

```azurecli-interactive
az network lb frontend-ip create \
--lb-name dsLB  \
--name dsLbFrontEnd_v6  \
--resource-group DsResourceGroup01  \
--public-ip-address dsPublicIP_v6

```

### <a name="configure-ipv6-back-end-address-pool"></a>Configuración del grupo de direcciones de back-end IPv6

Cree un grupo de direcciones de back-end IPv6 con [az network lb address-pool create](https://docs.microsoft.com/cli/azure/network/lb/address-pool?view=azure-cli-latest#az-network-lb-address-pool-create). En el ejemplo siguiente, se crean el grupo de direcciones de back-end denominado *dsLbBackEndPool_v6* que va a incluir VM con configuraciones de NIC IPv6:

```azurecli-interactive
az network lb address-pool create \
--lb-name dsLB  \
--name dsLbBackEndPool_v6  \
--resource-group DsResourceGroup01
```

### <a name="create-a-health-probe"></a>Creación de un sondeo de estado
Cree un sondeo de estado con [az network lb probe create](https://docs.microsoft.com/cli/azure/network/lb/probe?view=azure-cli-latest) para supervisar el estado de las máquinas virtuales. 

```azurecli-interactive
az network lb probe create -g DsResourceGroup01  --lb-name dsLB -n dsProbe --protocol tcp --port 3389
```

### <a name="create-a-load-balancer-rule"></a>Creación de una regla de equilibrador de carga

Las reglas de equilibrador de carga se utilizan para definir cómo se distribuye el tráfico a las máquinas virtuales. Defina la configuración de la IP de front-end para el tráfico entrante y el grupo de IP de back-end para el tráfico entrante, junto con los puertos de origen y destino requeridos. 

Cree una regla de equilibrador de carga con [az network lb rule create](https://docs.microsoft.com/cli/azure/network/lb/rule?view=azure-cli-latest#az-network-lb-rule-create). En el ejemplo siguiente, se crean reglas del equilibrador de carga llamadas *dsLBrule_v4* y *dsLBrule_v6*, y se equilibra el tráfico del puerto *TCP* *80* dirigido a las configuraciones de IP de front-end IPv4 e IPv6:

```azurecli-interactive
az network lb rule create \
--lb-name dsLB  \
--name dsLBrule_v4  \
--resource-group DsResourceGroup01  \
--frontend-ip-name dsLbFrontEnd_v4  \
--protocol Tcp  \
--frontend-port 80  \
--backend-port 80  \
--probe-name dsProbe \
--backend-pool-name dsLbBackEndPool_v4


az network lb rule create \
--lb-name dsLB  \
--name dsLBrule_v6  \
--resource-group DsResourceGroup01 \
--frontend-ip-name dsLbFrontEnd_v6  \
--protocol Tcp  \
--frontend-port 80 \
--backend-port 80  \
--probe-name dsProbe \
--backend-pool-name dsLbBackEndPool_v6

```

## <a name="create-network-resources"></a>Crear recursos de red
Para poder implementar algunas VM, debe crear recursos de red que lo permitan: un conjunto de disponibilidad, un grupo de seguridad de red, una red virtual y varias NIC virtuales. 
### <a name="create-an-availability-set"></a>Crear un conjunto de disponibilidad
Para mejorar la disponibilidad de la aplicación, coloque las VM en un conjunto de disponibilidad.

Cree el conjunto de disponibilidad con [az vm availability-set create](https://docs.microsoft.com/cli/azure/vm/availability-set?view=azure-cli-latest). En el ejemplo siguiente se crea un conjunto de disponibilidad denominado *dsAVset*:

```azurecli-interactive
az vm availability-set create \
--name dsAVset  \
--resource-group DsResourceGroup01  \
--location eastus \
--platform-fault-domain-count 2  \
--platform-update-domain-count 2  
```

### <a name="create-network-security-group"></a>Creación de un grupo de seguridad de red

Cree un grupo de seguridad de red para las reglas que van a controlar la comunicación entrante y saliente de la red virtual.

#### <a name="create-a-network-security-group"></a>Crear un grupo de seguridad de red

Cree un grupo de seguridad de red con [az network nsg create](https://docs.microsoft.com/cli/azure/network/nsg?view=azure-cli-latest#az-network-nsg-create).


```azurecli-interactive
az network nsg create \
--name dsNSG1  \
--resource-group DsResourceGroup01  \
--location eastus

```

#### <a name="create-a-network-security-group-rule-for-inbound-and-outbound-connections"></a>Creación de una regla de grupo de seguridad de red para las conexiones entrantes y salientes

Cree una regla de grupo de seguridad de red para permitir las conexiones RDP a través del puerto 3389, la conexión a Internet a través del puerto 80 y las conexiones salientes con [az network nsg rule create](https://docs.microsoft.com/cli/azure/network/nsg/rule?view=azure-cli-latest#az-network-nsg-rule-create).

```azurecli-interactive
# Create inbound rule for port 3389
az network nsg rule create \
--name allowRdpIn  \
--nsg-name dsNSG1  \
--resource-group DsResourceGroup01  \
--priority 100  \
--description "Allow Remote Desktop In"  \
--access Allow  \
--protocol "*"  \
--direction Inbound  \
--source-address-prefixes "*"  \
--source-port-ranges "*"  \
--destination-address-prefixes "*"  \
--destination-port-ranges 3389

# Create inbound rule for port 80
az network nsg rule create \
--name allowHTTPIn  \
--nsg-name dsNSG1  \
--resource-group DsResourceGroup01  \
--priority 200  \
--description "Allow HTTP In"  \
--access Allow  \
--protocol "*"  \
--direction Inbound  \
--source-address-prefixes "*"  \
--source-port-ranges 80  \
--destination-address-prefixes "*"  \
--destination-port-ranges 80

# Create outbound rule

az network nsg rule create \
--name allowAllOut  \
--nsg-name dsNSG1  \
--resource-group DsResourceGroup01  \
--priority 300  \
--description "Allow All Out"  \
--access Allow  \
--protocol "*"  \
--direction Outbound  \
--source-address-prefixes "*"  \
--source-port-ranges "*"  \
--destination-address-prefixes "*"  \
--destination-port-ranges "*"
```


### <a name="create-a-virtual-network"></a>Creación de una red virtual

Cree la red virtual con el comando [az network vnet create](https://docs.microsoft.com/cli/azure/network/vnet?view=azure-cli-latest#az-network-vnet-create). En el ejemplo siguiente se crea una red virtual denominada *dsVNET* con las subredes *dsSubNET_v4* y *dsSubNET_v6*:

```azurecli-interactive
# Create the virtual network
az network vnet create \
--name dsVNET \
--resource-group DsResourceGroup01 \
--location eastus  \
--address-prefixes "10.0.0.0/16" "ace:cab:deca::/48"

# Create a single dual stack subnet

az network vnet subnet create \
--name dsSubNET \
--resource-group DsResourceGroup01 \
--vnet-name dsVNET \
--address-prefixes "10.0.0.0/24" "ace:cab:deca:deed::/64" \
--network-security-group dsNSG1
```

### <a name="create-nics"></a>Creación de tarjetas NIC

Cree NIC virtuales para cada VM con [az network nic create](https://docs.microsoft.com/cli/azure/network/nic?view=azure-cli-latest#az-network-nic-create). El ejemplo siguiente crea una NIC virtual para cada VM. Cada NIC tiene dos configuraciones IP (una configuración IPv4 y una configuración IPv6). Debe crear la configuración IPV6 con [az network nic ip-config create](https://docs.microsoft.com/cli/azure/network/nic/ip-config?view=azure-cli-latest#az-network-nic-ip-config-create).
 
```azurecli-interactive
# Create NICs
az network nic create \
--name dsNIC0  \
--resource-group DsResourceGroup01 \
--network-security-group dsNSG1  \
--vnet-name dsVNET  \
--subnet dsSubNet  \
--private-ip-address-version IPv4 \
--lb-address-pools dsLbBackEndPool_v4  \
--lb-name dsLB  \
--public-ip-address dsVM0_remote_access

az network nic create \
--name dsNIC1 \
--resource-group DsResourceGroup01 \
--network-security-group dsNSG1 \
--vnet-name dsVNET \
--subnet dsSubNet \
--private-ip-address-version IPv4 \
--lb-address-pools dsLbBackEndPool_v4 \
--lb-name dsLB \
--public-ip-address dsVM1_remote_access

# Create IPV6 configurations for each NIC

az network nic ip-config create \
--name dsIp6Config_NIC0  \
--nic-name dsNIC0  \
--resource-group DsResourceGroup01 \
--vnet-name dsVNET \
--subnet dsSubNet \
--private-ip-address-version IPv6 \
--lb-address-pools dsLbBackEndPool_v6 \
--lb-name dsLB

az network nic ip-config create \
--name dsIp6Config_NIC1 \
--nic-name dsNIC1 \
--resource-group DsResourceGroup01 \
--vnet-name dsVNET \
--subnet dsSubNet \
--private-ip-address-version IPv6 \
--lb-address-pools dsLbBackEndPool_v6 \
--lb-name dsLB
```

### <a name="create-virtual-machines"></a>Creación de máquinas virtuales

Cree las máquinas virtuales con [az vm create](https://docs.microsoft.com/cli/azure/vm?view=azure-cli-latest#az-vm-create). En el ejemplo siguiente, se crean dos máquinas virtuales y los componentes de red virtual necesarios, si aún no existen. 

Cree la máquina virtual *dsVM0* de la siguiente manera:

```azurecli-interactive
 az vm create \
--name dsVM0 \
--resource-group DsResourceGroup01 \
--nics dsNIC0 \
--size Standard_A2 \
--availability-set dsAVset \
--image MicrosoftWindowsServer:WindowsServer:2019-Datacenter:latest  
```

Cree la máquina virtual *dsVM1* de la siguiente manera:

```azurecli-interactive
az vm create \
--name dsVM1 \
--resource-group DsResourceGroup01 \
--nics dsNIC1 \
--size Standard_A2 \
--availability-set dsAVset \
--image MicrosoftWindowsServer:WindowsServer:2019-Datacenter:latest 
```

## <a name="view-ipv6-dual-stack-virtual-network-in-azure-portal"></a>Visualización de la red virtual de doble pila IPv6 en Azure Portal
Para ver la red virtual de doble pila IPv6 en Azure Portal, siga estos pasos:
1. En la barra de búsqueda del portal, escriba *dsVnet*.
2. Cuando aparezca la opción **myVirtualNetwork** en los resultados de la búsqueda, selecciónela. De este modo, se abrirá la **página de información general** de la red virtual de doble pila llamada *dsVnet*. En la red virtual de doble pila, se muestran las dos NIC con las configuraciones IPv4 e IPv6 en una subred de pila doble denominada *dsSubnet*.

  ![Red virtual de doble pila IPv6 de Azure](./media/virtual-network-ipv4-ipv6-dual-stack-powershell/dual-stack-vnet.png)

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no se necesiten, puede usar el comando [az group delete](/cli/azure/group#az-group-delete) para quitar el grupo de recursos, la máquina virtual y todos los recursos relacionados.

```azurecli-interactive
 az group delete --name DsResourceGroup01
```

## <a name="next-steps"></a>Pasos siguientes

En este artículo, ha creado una instancia de Standard Load Balancer con una configuración IP de front-end doble (IPv4 e IPv6). También ha creado dos máquinas virtuales que contienen NIC con configuraciones de IP duales (IPV4 + IPv6) que se agregaron al grupo de back-end del equilibrador de carga. Para más información sobre la compatibilidad de IPv6 en las redes virtuales de Azure, consulte [¿Qué es IPv6 para Azure Virtual Network?](ipv6-overview.md)
