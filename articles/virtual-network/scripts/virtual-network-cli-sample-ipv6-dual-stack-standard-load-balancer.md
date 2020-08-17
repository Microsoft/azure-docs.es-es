---
title: 'Ejemplo de script de configuración del front-end IPv6 con la CLI de Azure: Standard Load Balancer'
titlesuffix: Azure Virtual Network
description: Obtenga información sobre cómo configurar puntos de conexión IPv6 en un ejemplo de script de red virtual con Standard Load Balancer.
services: virtual-network
documentationcenter: na
author: KumudD
manager: twooley
ms.service: virtual-network
ms.devlang: NA
ms.topic: article
ms.workload: infrastructure-services
ms.date: 07/15/2019
ms.author: kumud
ms.custom: devx-track-azurecli
ms.openlocfilehash: 12233fd44a74d127e9b7de71971b9831cf80f7b4
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87492945"
---
# <a name="configure-ipv6-endpoints-in-virtual-network-script-sample-using-standard-load-balancerpreview"></a>Configuración de puntos de conexión IPv6 en el de script de red virtual de ejemplo con Standard Load Balancer (versión preliminar)

En este artículo, se explica cómo se implementa en Azure una aplicación de pila doble (IPv4 + IPv6) que contiene una red virtual de pila doble y una subred de pila doble, una instancia de Standard Load Balancer con configuraciones de front-end duales (IPv4 + IPv6), VM con NIC que tienen una configuración de IP dual, reglas de grupo de seguridad de red dual e IP públicas duales.

Puede ejecutar el script desde Azure [Cloud Shell](https://shell.azure.com/bash) o desde una instalación de CLI de Azure local. Si usa la CLI localmente, este script requiere que ejecute la versión 2.0.28 o posterior. Ejecute `az --version` para ver cuál es la versión instalada. Si necesita instalarla o actualizarla, consulte [Instalación de la CLI de Azure](/cli/azure/install-azure-cli). Si ejecuta la CLI localmente, también debe ejecutar `az login` para crear una conexión con Azure.

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Prerequisites
Para usar IPv6 para la característica Azure Virtual Network, debe configurar la suscripción solo una vez como se indica a continuación:

```azurecli
az feature register --name AllowIPv6VirtualNetwork --namespace Microsoft.Network
az feature register --name AllowIPv6CAOnStandardLB --namespace Microsoft.Network
```

Se tarda hasta 30 minutos en completar el registro de características. Puede comprobar el estado del registro ejecutando el siguiente comando de la CLI de Azure:

```azurecli
az feature show --name AllowIPv6VirtualNetwork --namespace Microsoft.Network
az feature show --name AllowIPv6CAOnStandardLB --namespace Microsoft.Network
```

Una vez completado el registro, ejecute el siguiente comando:

```azurecli
az provider register --namespace Microsoft.Network
```

## <a name="sample-script"></a>Script de ejemplo


```azurecli
# Create a resource group
az group create \
--name DsResourceGroup01 \
--location eastus

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

# Create public IP addresses for remote access to VMs
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

# Create load balancer

az network lb create \
--name dsLB  \
--resource-group DsResourceGroup01 \
--sku Standard \
--location eastus \
--frontend-ip-name dsLbFrontEnd_v4  \
--public-ip-address dsPublicIP_v4  \
--backend-pool-name dsLbBackEndPool_v4

# Create IPv6 front-end
az network lb frontend-ip create \
--lb-name dsLB  \
--name dsLbFrontEnd_v6  \
--resource-group DsResourceGroup01  \
--public-ip-address dsPublicIP_v6

# Configure IPv6 back-end address pool
az network lb address-pool create \
--lb-name dsLB  \
--name dsLbBackEndPool_v6  \
--resource-group DsResourceGroup01

# Create a load balancer rule

az network lb rule create \
--lb-name dsLB  \
--name dsLBrule_v4  \
--resource-group DsResourceGroup01  \
--frontend-ip-name dsLbFrontEnd_v4  \
--protocol Tcp  \
--frontend-port 80  \
--backend-port 80  \
--backend-pool-name dsLbBackEndPool_v4


az network lb rule create \
--lb-name dsLB  \
--name dsLBrule_v6  \
--resource-group DsResourceGroup01 \
--frontend-ip-name dsLbFrontEnd_v6  \
--protocol Tcp  \
--frontend-port 80 \
--backend-port 80  \
--backend-pool-name dsLbBackEndPool_v6

# Create an availability set
az vm availability-set create \
--name dsAVset  \
--resource-group DsResourceGroup01  \
--location eastus \
--platform-fault-domain-count 2  \
--platform-update-domain-count 2  

# Create network security group

az network nsg create \
--name dsNSG1  \
--resource-group DsResourceGroup01  \
--location eastus

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

# Create virtual machine dsVM0

 az vm create \
--name dsVM0 \
--resource-group DsResourceGroup01 \
--nics dsNIC0 \
--size Standard_A2 \
--availability-set dsAVset \
--image MicrosoftWindowsServer:WindowsServer:2019-Datacenter:latest  

# Create virtual machine dsVM1

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

> [!NOTE]
> En esta versión preliminar, la dirección IPv6 de la red virtual de Azure está disponible en Azure Portal en modo de solo lectura.

## <a name="clean-up-deployment"></a>Limpieza de la implementación

Ejecute el siguiente comando para quitar el grupo de recursos, la VM y todos los recursos relacionados:

```azurecli
az group delete --name <resourcegroupname> --yes
```

## <a name="script-explanation"></a>Explicación del script

Este script usa los siguientes comandos para crear un grupo de recursos, una máquina virtual, un conjunto de disponibilidad, un equilibrador de carga y todos los recursos relacionados. Cada comando de la tabla crea un vínculo a documentación específica del comando.

| Get-Help | Notas |
|---|---|
| [az group create](https://docs.microsoft.com/cli/azure/group#az-group-create) | Crea un grupo de recursos en el que se almacenan todos los recursos. |
| [az network vnet create](https://docs.microsoft.com/cli/azure/network/vnet#az-network-vnet-create) | Crea una red virtual y una subred de Azure. |
| [az network public-ip create](https://docs.microsoft.com/cli/azure/network/public-ip#az-network-public-ip-create) | Crea una dirección IP pública con una dirección IP estática y un nombre DNS asociado. |
| [az network lb create](https://docs.microsoft.com/cli/azure/network/lb#az-network-lb-create) | Crea una instancia de Azure Load Balancer. |
| [az network lb probe create](https://docs.microsoft.com/cli/azure/network/lb/probe#az-network-lb-probe-create) | Crea un sondeo de equilibrador de carga. Un sondeo de equilibrador de carga se usa para supervisar cada máquina virtual del conjunto de equilibrador de carga. Si alguna máquina virtual deja de estar accesible, el tráfico no se enruta a la máquina virtual. |
| [az network lb rule create](https://docs.microsoft.com/cli/azure/network/lb/rule#az-network-lb-rule-create) | Crea una regla de equilibrador de carga. En este ejemplo, se crea una regla para el puerto 80. Según va llegando el tráfico HTTP a Load Balancer, se enruta al puerto 80 de una de las máquinas virtuales del conjunto de Load Balancer. |
| [az network lb inbound-nat-rule create](https://docs.microsoft.com/cli/azure/network/lb/inbound-nat-rule#az-network-lb-inbound-nat-rule-create) | Crea una regla de traducción de direcciones de red (NAT) de Load Balancer.  Las reglas de NAT asignan un puerto de Load Balancer a un puerto en una máquina virtual. En este ejemplo, se crea una regla NAT para el tráfico SSH para cada máquina virtual del conjunto de Load Balancer.  |
| [az network nsg create](https://docs.microsoft.com/cli/azure/network/nsg#az-network-nsg-create) | Crea un grupo de seguridad de red (NSG), que es un límite de seguridad entre Internet y la máquina virtual. |
| [az network nsg rule create](https://docs.microsoft.com/cli/azure/network/nsg/rule#az-network-nsg-rule-create) | Crea una regla de NSG para permitir el tráfico entrante. En este ejemplo, el puerto 22 está abierto al tráfico SSH. |
| [az network nic create](https://docs.microsoft.com/cli/azure/network/nic#az-network-nic-create) | Crea una tarjeta de máquina virtual y la conecta con la red virtual, la subred y el NSG. |
| [az vm availability-set create](https://docs.microsoft.com/cli/azure/network/lb/rule#az-network-lb-rule-create) | Crea un conjunto de disponibilidad. Los conjuntos de disponibilidad garantizan la disponibilidad de las aplicaciones al repartir las máquinas virtuales entre los recursos físicos, de forma que si se produce un error, el conjunto no se verá afectado en su totalidad. |
| [az vm create](/cli/azure/vm#az-vm-create) | Crea la máquina virtual y la conecta con la tarjeta de red, la red virtual, la subred y el NSG. Este comando también especifica la imagen de máquina virtual que se usará, y las credenciales administrativas.  |
| [az group delete](https://docs.microsoft.com/cli/azure/vm/extension#az-vm-extension-set) | Elimina un grupo de recursos, incluidos todos los recursos anidados. |

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre la CLI de Azure, consulte la [documentación de la CLI de Azure](https://docs.microsoft.com/cli/azure).

Encontrará más ejemplos de scripts de la CLI de Redes de Azure en la [documentación de Redis de Azure](../cli-samples.md).
