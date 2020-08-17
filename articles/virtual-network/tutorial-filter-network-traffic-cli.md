---
title: Filtrado del tráfico de red en la CLI de Azure | Microsoft Docs
description: En este artículo, aprenderá a filtrar el tráfico de red a una subred, con un grupo de seguridad de red, mediante la CLI de Azure.
services: virtual-network
documentationcenter: virtual-network
author: KumudD
manager: twooley
editor: ''
tags: azure-resource-manager
Customer intent: I want to filter network traffic to virtual machines that perform similar functions, such as web servers.
ms.assetid: ''
ms.service: virtual-network
ms.devlang: azurecli
ms.topic: how-to
ms.tgt_pltfrm: virtual-network
ms.workload: infrastructure
ms.date: 03/30/2018
ms.author: kumud
ms.custom: devx-track-azurecli
ms.openlocfilehash: 9af6311165f50023cfca8f9253f77b4c84c25dd5
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87500936"
---
# <a name="filter-network-traffic-with-a-network-security-group-using-the-azure-cli"></a>Filtrado del tráfico de red con un grupo de seguridad de red mediante la CLI de Azure

Puede filtrar el tráfico de red entrante y saliente de una subred de una red virtual con un grupo de seguridad de red. Los grupos de seguridad de red contienen reglas de seguridad que filtran el tráfico de red por dirección IP, puerto y protocolo. Las reglas de seguridad se aplican a los recursos implementados en una subred. En este artículo aprenderá a:

* Crear un grupo de seguridad de red y reglas de seguridad
* Crear una red virtual y asociar un grupo de seguridad de red a una subred
* Implementar máquinas virtuales (VM) en una subred
* Probar los filtros de tráfico

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

Si decide instalar y usar la CLI localmente, para este artículo es preciso que ejecute la versión 2.0.28 o posterior de la CLI de Azure. Para encontrar la versión, ejecute `az --version`. Si necesita instalarla o actualizarla, vea [Instalación de la CLI de Azure](/cli/azure/install-azure-cli). 


## <a name="create-a-network-security-group"></a>Crear un grupo de seguridad de red

Un grupo de seguridad de red contiene reglas de seguridad. Las reglas de seguridad especifican un origen y destino. Los orígenes y destinos pueden ser grupos de seguridad de aplicaciones.

### <a name="create-application-security-groups"></a>Creación de grupos de seguridad de aplicaciones

En primer lugar, debe crear un grupo de recursos para todos los recursos creados en este artículo con [az group create](/cli/azure/group). En el ejemplo siguiente se crea un grupo de recursos en la ubicación *eastus*: 

```azurecli-interactive
az group create \
  --name myResourceGroup \
  --location eastus
```

Cree un grupo de seguridad de aplicaciones con [az network asg create](/cli/azure/network/asg). Un grupo de seguridad de aplicaciones permite agrupar servidores con requisitos de filtrado de puertos similar. El ejemplo siguiente crea dos grupos de seguridad de aplicaciones.

```azurecli-interactive
az network asg create \
  --resource-group myResourceGroup \
  --name myAsgWebServers \
  --location eastus

az network asg create \
  --resource-group myResourceGroup \
  --name myAsgMgmtServers \
  --location eastus
```

### <a name="create-a-network-security-group"></a>Crear un grupo de seguridad de red

Cree un grupo de seguridad de red con [az network nsg create](/cli/azure/network/nsg). En el ejemplo siguiente se crea un grupo de seguridad de red llamado *myNsg*: 

```azurecli-interactive 
# Create a network security group
az network nsg create \
  --resource-group myResourceGroup \
  --name myNsg
```

### <a name="create-security-rules"></a>Creación de reglas de seguridad

Cree una regla de seguridad con [az network nsg rule create](/cli/azure/network/nsg/rule). En el ejemplo siguiente se crea una regla que permite el tráfico entrante desde Internet al grupo de seguridad de aplicaciones *myWebServers* a través de los puertos 80 y 443:

```azurecli-interactive
az network nsg rule create \
  --resource-group myResourceGroup \
  --nsg-name myNsg \
  --name Allow-Web-All \
  --access Allow \
  --protocol Tcp \
  --direction Inbound \
  --priority 100 \
  --source-address-prefix Internet \
  --source-port-range "*" \
  --destination-asgs "myAsgWebServers" \
  --destination-port-range 80 443
```

En el ejemplo siguiente se crea una regla que permite el tráfico entrante desde Internet al grupo de seguridad de aplicaciones *myMgmtServers* a través del puerto 22:

```azurecli-interactive
az network nsg rule create \
  --resource-group myResourceGroup \
  --nsg-name myNsg \
  --name Allow-SSH-All \
  --access Allow \
  --protocol Tcp \
  --direction Inbound \
  --priority 110 \
  --source-address-prefix Internet \
  --source-port-range "*" \
  --destination-asgs "myAsgMgmtServers" \
  --destination-port-range 22
```

En este artículo, se expone SSH (puerto 22) a Internet para la máquina virtual *myAsgMgmtServers*. Para entornos de producción, en lugar de exponer el puerto 22 a Internet, se recomienda conectarse a los recursos de Azure que desea administrar mediante una [VPN](../vpn-gateway/vpn-gateway-about-vpngateways.md?toc=%2fazure%2fvirtual-network%2ftoc.json) o una conexión de red [privada](../expressroute/expressroute-introduction.md?toc=%2fazure%2fvirtual-network%2ftoc.json).

## <a name="create-a-virtual-network"></a>Creación de una red virtual

Cree la red virtual con el comando [az network vnet create](/cli/azure/network/vnet). En el ejemplo siguiente se crea una red virtual llamada *myVirtualNetwork*:

```azurecli-interactive 
az network vnet create \
  --name myVirtualNetwork \
  --resource-group myResourceGroup \
  --address-prefixes 10.0.0.0/16
```

Agregue una subred a una red virtual con [az network vnet subnet create](/cli/azure/network/vnet/subnet). En el ejemplo siguiente se agrega una subred llamada *mySubnet* a la red virtual y se asocia el grupo de seguridad de red *myNsg* a dicha subred:

```azurecli-interactive
az network vnet subnet create \
  --vnet-name myVirtualNetwork \
  --resource-group myResourceGroup \
  --name mySubnet \
  --address-prefix 10.0.0.0/24 \
  --network-security-group myNsg
```

## <a name="create-virtual-machines"></a>Creación de máquinas virtuales

Cree dos máquinas virtuales en la red virtual para que pueda validar el filtrado de tráfico en un paso posterior. 

Cree la máquina virtual con [az vm create](/cli/azure/vm). En el ejemplo siguiente se crea una máquina virtual que actúa como un servidor web. La opción `--asgs myAsgWebServers` hace que Azure incluya la interfaz de red que se crea para la máquina virtual como un miembro del grupo de seguridad de aplicaciones *myAsgWebServers*.

La opción `--nsg ""` se especifica para evitar que Azure cree un grupo de seguridad de red predeterminado para la interfaz de red que Azure crea al generar la máquina virtual. Para simplificar los pasos de este artículo, se usa una contraseña. Las claves se utilizan habitualmente en las implementaciones de producción. Si utiliza claves, también debe configurar el agente SSH de reenvío para los pasos restantes. Para más información, consulte la documentación del cliente SSH. Reemplace `<replace-with-your-password>` en el siguiente comando por una contraseña de su elección.

```azurecli-interactive
adminPassword="<replace-with-your-password>"

az vm create \
  --resource-group myResourceGroup \
  --name myVmWeb \
  --image UbuntuLTS \
  --vnet-name myVirtualNetwork \
  --subnet mySubnet \
  --nsg "" \
  --asgs myAsgWebServers \
  --admin-username azureuser \
  --admin-password $adminPassword
```

La máquina virtual tarda en crearse unos minutos. Una vez creada la máquina virtual, se devuelve una salida similar a la del siguiente ejemplo: 

```output
{
  "fqdns": "",
  "id": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVmWeb",
  "location": "eastus",
  "macAddress": "00-0D-3A-23-9A-49",
  "powerState": "VM running",
  "privateIpAddress": "10.0.0.4",
  "publicIpAddress": "13.90.242.231",
  "resourceGroup": "myResourceGroup"
}
```

Anote el valor de **publicIpAddress**. En un paso posterior, usaremos esta dirección para obtener acceso a la máquina virtual desde Internet.  Cree una máquina virtual para que actúe como un servidor de administración:

```azurecli-interactive
az vm create \
  --resource-group myResourceGroup \
  --name myVmMgmt \
  --image UbuntuLTS \
  --vnet-name myVirtualNetwork \
  --subnet mySubnet \
  --nsg "" \
  --asgs myAsgMgmtServers \
  --admin-username azureuser \
  --admin-password $adminPassword
```

La máquina virtual tarda en crearse unos minutos. Una vez creada la máquina virtual, tome nota de **publicIpAddress** en la salida devuelta. Esta dirección se utiliza para acceder a la máquina virtual en el siguiente paso. No continúe con el paso siguiente hasta que Azure finalice la creación de la máquina virtual.

## <a name="test-traffic-filters"></a>Probar los filtros de tráfico

Use el siguiente comando para crear una sesión SSH con la máquina virtual *myVmMgmt*. Reemplace *\<publicIpAddress>* con la dirección IP pública de la máquina virtual. En el ejemplo anterior, la dirección IP era *13.90.242.231*.

```bash 
ssh azureuser@<publicIpAddress>
```

Cuando se le solicite una contraseña, escriba la que escribió en el artículo de [creación de máquinas virtuales](#create-virtual-machines).

La conexión se realiza correctamente porque el puerto 22 tiene permitida la entrada desde Internet al grupo de seguridad de aplicaciones *myAsgMgmtServers* en el que se encuentra la interfaz de red conectada a la máquina virtual *myVmMgmt*.

Utilice el siguiente comando para conectarse mediante SSH a la máquina virtual *myVmWeb* desde la máquina virtual *myVmMgmt*:

```bash 
ssh azureuser@myVmWeb
```

La conexión se realiza correctamente porque una regla de seguridad predeterminada dentro de cada grupo de seguridad de red permite el tráfico a través de todos los puertos entre todas las direcciones IP de una red virtual. No es posible conectarse mediante SSH a la máquina virtual *myVmWeb* desde Internet porque la regla de seguridad para *myAsgWebServers* no permite el puerto 22 de entrada desde Internet.

Use los siguientes comandos para instalar el servidor web nginx en la máquina virtual *myVmWeb*:

```bash 
# Update package source
sudo apt-get -y update

# Install NGINX
sudo apt-get -y install nginx
```

La máquina virtual *myVmWeb* tiene permiso de salida a Internet para recuperar nginx porque una regla de seguridad predeterminada permite todo el tráfico de salida a Internet. Cierre la sesión SSH con *myVmWeb*, que le deja en el símbolo del sistema `username@myVmMgmt:~$` de la máquina virtual *myVmMgmt*. Escriba el siguiente comando para recuperar la pantalla de bienvenida de nginx desde la máquina virtual *myVmWeb*:

```bash
curl myVmWeb
```

Cierre la sesión de la máquina virtual *myVmMgmt*. Para confirmar que puede acceder al servidor web *myVmWeb* desde fuera de Azure, escriba `curl <publicIpAddress>` desde su propio equipo. La conexión se realiza correctamente porque el puerto 80 tiene permitida la entrada desde Internet al grupo de seguridad de aplicaciones *myAsgWebServers* en el que se encuentra la interfaz de red conectada a la máquina virtual *myVmWeb*.

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no se necesiten, use [az group delete](/cli/azure/group) para quitar el grupo de recursos y todos los recursos que contenga.

```azurecli-interactive
az group delete --name myResourceGroup --yes
```

## <a name="next-steps"></a>Pasos siguientes

En este artículo, ha creado un grupo de seguridad de red y lo ha asociado a una subred de una red virtual. Para más información acerca de los grupos de seguridad de red, consulte [Introducción a los grupos de seguridad de red](security-overview.md) y [Administración de un grupo de seguridad de red](manage-network-security-group.md).

De forma predeterminada, Azure enruta el tráfico entre subredes. En su lugar, puede elegir enrutar el tráfico entre subredes a través de una máquina virtual, que actúa como un firewall, por ejemplo. Para obtener información sobre cómo hacerlo, consulte [Creación de una tabla de rutas](tutorial-create-route-table-cli.md).
