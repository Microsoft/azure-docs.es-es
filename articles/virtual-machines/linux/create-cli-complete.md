---
title: Creación de un entorno de Linux con la CLI de Azure
description: Cree almacenamiento, una máquina virtual Linux, una red virtual y subred, un equilibrador de carga, una NIC, una dirección IP pública, un grupo de seguridad de red, todo desde el principio mediante la CLI de Azure.
author: cynthn
ms.service: virtual-machines-linux
ms.topic: how-to
ms.date: 12/14/2017
ms.author: cynthn
ms.openlocfilehash: 4348d3d71259b5bdf63b1c52af53bff59c650086
ms.sourcegitcommit: 2ff0d073607bc746ffc638a84bb026d1705e543e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87829023"
---
# <a name="create-a-complete-linux-virtual-machine-with-the-azure-cli"></a>Creación de una máquina virtual completa de Linux con la CLI de Azure
Para crear rápidamente una máquina virtual en Azure, puede usar un solo comando de la CLI de Azure que use valores predeterminados para crear los recursos de apoyo necesarios. Los recursos como una red virtual, una dirección IP pública y reglas de grupo de seguridad de red se crean automáticamente. Para tener más control del entorno en uso de producción, puede crear estos recursos antes de tiempo y luego agregarles las máquinas virtuales. Este artículo lo guía a lo largo del proceso de creación de una máquina virtual y de cada uno de los recursos de apoyo.

Asegúrese de que ha instalado la versión más reciente de la [CLI de Azure](/cli/azure/install-az-cli2) e iniciado sesión en una cuenta de Azure con [az login](/cli/azure/reference-index).

En los ejemplos siguientes, reemplace los nombres de parámetros de ejemplo por los suyos propios. Los nombres de parámetro de ejemplo incluyen *myResourceGroup*, *myVnet* y *myVM*.

## <a name="create-resource-group"></a>Creación de un grupo de recursos
Un grupo de recursos de Azure es un contenedor lógico en el que se implementan y se administran los recursos de Azure. Se debe crear un grupo de recursos antes que una máquina virtual y los recursos de red virtual de apoyo. Cree el grupo de recursos con [az group create](/cli/azure/group). En el ejemplo siguiente, se crea un grupo de recursos denominado *myResourceGroup* en la ubicación *eastus*:

```azurecli
az group create --name myResourceGroup --location eastus
```

De forma predeterminada, la salida de los comandos de la CLI de Azure está en JSON (notación de objetos JavaScript). Para cambiar la salida predeterminada a una lista o tabla, por ejemplo, use [az configure --output](/cli/azure/reference-index). También puede agregar `--output` a cualquier comando para un cambio específico del formato de salida. En el ejemplo siguiente se muestra la salida JSON desde el comando `az group create`:

```json                       
{
  "id": "/subscriptions/guid/resourceGroups/myResourceGroup",
  "location": "eastus",
  "name": "myResourceGroup",
  "properties": {
    "provisioningState": "Succeeded"
  },
  "tags": null
}
```

## <a name="create-a-virtual-network-and-subnet"></a>Creación de una red virtual y una subred
Después el usuario crea una red virtual en Azure y una subred en la que pueda crear las máquinas virtuales. Use [az network vnet create](/cli/azure/network/vnet) para crear una red virtual denominada *myVnet* con el prefijo de dirección *192.168.0.0/16*. Agregue también una subred denominada *mySubnet* con el prefijo de dirección *192.168.1.0/24*:

```azurecli
az network vnet create \
    --resource-group myResourceGroup \
    --name myVnet \
    --address-prefix 192.168.0.0/16 \
    --subnet-name mySubnet \
    --subnet-prefix 192.168.1.0/24
```

La salida muestra que la subred está creada lógicamente dentro de la red virtual:

```json
{
  "addressSpace": {
    "addressPrefixes": [
      "192.168.0.0/16"
    ]
  },
  "dhcpOptions": {
    "dnsServers": []
  },
  "etag": "W/\"e95496fc-f417-426e-a4d8-c9e4d27fc2ee\"",
  "id": "/subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Network/virtualNetworks/myVnet",
  "location": "eastus",
  "name": "myVnet",
  "provisioningState": "Succeeded",
  "resourceGroup": "myResourceGroup",
  "resourceGuid": "ed62fd03-e9de-430b-84df-8a3b87cacdbb",
  "subnets": [
    {
      "addressPrefix": "192.168.1.0/24",
      "etag": "W/\"e95496fc-f417-426e-a4d8-c9e4d27fc2ee\"",
      "id": "/subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Network/virtualNetworks/myVnet/subnets/mySubnet",
      "ipConfigurations": null,
      "name": "mySubnet",
      "networkSecurityGroup": null,
      "provisioningState": "Succeeded",
      "resourceGroup": "myResourceGroup",
      "resourceNavigationLinks": null,
      "routeTable": null
    }
  ],
  "tags": {},
  "type": "Microsoft.Network/virtualNetworks",
  "virtualNetworkPeerings": null
}
```


## <a name="create-a-public-ip-address"></a>Crear una dirección IP pública
Ahora cree una dirección IP pública con [az network public-ip create](/cli/azure/network/public-ip). Esta dirección IP pública le permite conectarse a las máquinas virtuales desde Internet. Dado que la dirección predeterminada es dinámica, cree una entrada DNS con nombre con el parámetro `--domain-name-label`. En el ejemplo siguiente se crea una IP pública denominada "*myPublicIP*" con el nombre DNS *mypublicdns*. Como el nombre DNS debe ser único, especifique su propio nombre DNS único:

```azurecli
az network public-ip create \
    --resource-group myResourceGroup \
    --name myPublicIP \
    --dns-name mypublicdns
```

Salida:

```json
{
  "publicIp": {
    "dnsSettings": {
      "domainNameLabel": "mypublicdns",
      "fqdn": "mypublicdns.eastus.cloudapp.azure.com",
      "reverseFqdn": null
    },
    "etag": "W/\"2632aa72-3d2d-4529-b38e-b622b4202925\"",
    "id": "/subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Network/publicIPAddresses/myPublicIP",
    "idleTimeoutInMinutes": 4,
    "ipAddress": null,
    "ipConfiguration": null,
    "location": "eastus",
    "name": "myPublicIP",
    "provisioningState": "Succeeded",
    "publicIpAddressVersion": "IPv4",
    "publicIpAllocationMethod": "Dynamic",
    "resourceGroup": "myResourceGroup",
    "resourceGuid": "4c65de38-71f5-4684-be10-75e605b3e41f",
    "tags": null,
    "type": "Microsoft.Network/publicIPAddresses"
  }
}
```


## <a name="create-a-network-security-group"></a>Crear un grupo de seguridad de red
Para controlar el flujo de tráfico de entrada y salida de las máquinas virtuales, aplique un grupo de seguridad de red a una NIC virtual o subred. En el ejemplo siguiente se usa [az network nsg create](/cli/azure/network/nsg) para crear un grupo de seguridad de red denominado *myNetworkSecurityGroup*:

```azurecli
az network nsg create \
    --resource-group myResourceGroup \
    --name myNetworkSecurityGroup
```

Defina las reglas que permiten o deniegan el tráfico específico. Para permitir conexiones entrantes en el puerto 22 (para permitir el acceso SSH), cree una regla de entrada con [az network nsg rule create](/cli/azure/network/nsg/rule). En el ejemplo siguiente se crea una regla denominada *myNetworkSecurityGroupRuleSSH*:

```azurecli
az network nsg rule create \
    --resource-group myResourceGroup \
    --nsg-name myNetworkSecurityGroup \
    --name myNetworkSecurityGroupRuleSSH \
    --protocol tcp \
    --priority 1000 \
    --destination-port-range 22 \
    --access allow
```

Para permitir las conexiones entrantes en el puerto 80 (para el tráfico web), agregue otra regla del grupo de seguridad de red. En el ejemplo siguiente se crea una regla denominada *myNetworkSecurityGroupRuleHTTP*:

```azurecli
az network nsg rule create \
    --resource-group myResourceGroup \
    --nsg-name myNetworkSecurityGroup \
    --name myNetworkSecurityGroupRuleWeb \
    --protocol tcp \
    --priority 1001 \
    --destination-port-range 80 \
    --access allow
```

Examine el grupo de seguridad de red y las reglas con [az network nsg create](/cli/azure/network/nsg):

```azurecli
az network nsg show --resource-group myResourceGroup --name myNetworkSecurityGroup
```

Salida:

```json
{
  "defaultSecurityRules": [
    {
      "access": "Allow",
      "description": "Allow inbound traffic from all VMs in VNET",
      "destinationAddressPrefix": "VirtualNetwork",
      "destinationPortRange": "*",
      "direction": "Inbound",
      "etag": "W/\"3371b313-ea9f-4687-a336-a8ebdfd80523\"",
      "id": "/subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Network/networkSecurityGroups/myNetworkSecurityGroup/defaultSecurityRules/AllowVnetInBound",
      "name": "AllowVnetInBound",
      "priority": 65000,
      "protocol": "*",
      "provisioningState": "Succeeded",
      "resourceGroup": "myResourceGroup",
      "sourceAddressPrefix": "VirtualNetwork",
      "sourcePortRange": "*"
    },
    {
      "access": "Allow",
      "description": "Allow inbound traffic from azure load balancer",
      "destinationAddressPrefix": "*",
      "destinationPortRange": "*",
      "direction": "Inbound",
      "etag": "W/\"3371b313-ea9f-4687-a336-a8ebdfd80523\"",
      "id": "/subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Network/networkSecurityGroups/myNetworkSecurityGroup/defaultSecurityRules/AllowAzureLoadBalancerInBou",
      "name": "AllowAzureLoadBalancerInBound",
      "priority": 65001,
      "protocol": "*",
      "provisioningState": "Succeeded",
      "resourceGroup": "myResourceGroup",
      "sourceAddressPrefix": "AzureLoadBalancer",
      "sourcePortRange": "*"
    },
    {
      "access": "Deny",
      "description": "Deny all inbound traffic",
      "destinationAddressPrefix": "*",
      "destinationPortRange": "*",
      "direction": "Inbound",
      "etag": "W/\"3371b313-ea9f-4687-a336-a8ebdfd80523\"",
      "id": "/subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Network/networkSecurityGroups/myNetworkSecurityGroup/defaultSecurityRules/DenyAllInBound",
      "name": "DenyAllInBound",
      "priority": 65500,
      "protocol": "*",
      "provisioningState": "Succeeded",
      "resourceGroup": "myResourceGroup",
      "sourceAddressPrefix": "*",
      "sourcePortRange": "*"
    },
    {
      "access": "Allow",
      "description": "Allow outbound traffic from all VMs to all VMs in VNET",
      "destinationAddressPrefix": "VirtualNetwork",
      "destinationPortRange": "*",
      "direction": "Outbound",
      "etag": "W/\"3371b313-ea9f-4687-a336-a8ebdfd80523\"",
      "id": "/subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Network/networkSecurityGroups/myNetworkSecurityGroup/defaultSecurityRules/AllowVnetOutBound",
      "name": "AllowVnetOutBound",
      "priority": 65000,
      "protocol": "*",
      "provisioningState": "Succeeded",
      "resourceGroup": "myResourceGroup",
      "sourceAddressPrefix": "VirtualNetwork",
      "sourcePortRange": "*"
    },
    {
      "access": "Allow",
      "description": "Allow outbound traffic from all VMs to Internet",
      "destinationAddressPrefix": "Internet",
      "destinationPortRange": "*",
      "direction": "Outbound",
      "etag": "W/\"3371b313-ea9f-4687-a336-a8ebdfd80523\"",
      "id": "/subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Network/networkSecurityGroups/myNetworkSecurityGroup/defaultSecurityRules/AllowInternetOutBound",
      "name": "AllowInternetOutBound",
      "priority": 65001,
      "protocol": "*",
      "provisioningState": "Succeeded",
      "resourceGroup": "myResourceGroup",
      "sourceAddressPrefix": "*",
      "sourcePortRange": "*"
    },
    {
      "access": "Deny",
      "description": "Deny all outbound traffic",
      "destinationAddressPrefix": "*",
      "destinationPortRange": "*",
      "direction": "Outbound",
      "etag": "W/\"3371b313-ea9f-4687-a336-a8ebdfd80523\"",
      "id": "/subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Network/networkSecurityGroups/myNetworkSecurityGroup/defaultSecurityRules/DenyAllOutBound",
      "name": "DenyAllOutBound",
      "priority": 65500,
      "protocol": "*",
      "provisioningState": "Succeeded",
      "resourceGroup": "myResourceGroup",
      "sourceAddressPrefix": "*",
      "sourcePortRange": "*"
    }
  ],
  "etag": "W/\"3371b313-ea9f-4687-a336-a8ebdfd80523\"",
  "id": "/subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Network/networkSecurityGroups/myNetworkSecurityGroup",
  "location": "eastus",
  "name": "myNetworkSecurityGroup",
  "networkInterfaces": null,
  "provisioningState": "Succeeded",
  "resourceGroup": "myResourceGroup",
  "resourceGuid": "47a9964e-23a3-438a-a726-8d60ebbb1c3c",
  "securityRules": [
    {
      "access": "Allow",
      "description": null,
      "destinationAddressPrefix": "*",
      "destinationPortRange": "22",
      "direction": "Inbound",
      "etag": "W/\"9e344b60-0daa-40a6-84f9-0ebbe4a4b640\"",
      "id": "/subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Network/networkSecurityGroups/myNetworkSecurityGroup/securityRules/myNetworkSecurityGroupRuleSSH",
      "name": "myNetworkSecurityGroupRuleSSH",
      "priority": 1000,
      "protocol": "Tcp",
      "provisioningState": "Succeeded",
      "resourceGroup": "myResourceGroup",
      "sourceAddressPrefix": "*",
      "sourcePortRange": "*"
    },
    {
      "access": "Allow",
      "description": null,
      "destinationAddressPrefix": "*",
      "destinationPortRange": "80",
      "direction": "Inbound",
      "etag": "W/\"9e344b60-0daa-40a6-84f9-0ebbe4a4b640\"",
      "id": "/subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Network/networkSecurityGroups/myNetworkSecurityGroup/securityRules/myNetworkSecurityGroupRuleWeb",
      "name": "myNetworkSecurityGroupRuleWeb",
      "priority": 1001,
      "protocol": "Tcp",
      "provisioningState": "Succeeded",
      "resourceGroup": "myResourceGroup",
      "sourceAddressPrefix": "*",
      "sourcePortRange": "*"
    }
  ],
  "subnets": null,
  "tags": null,
  "type": "Microsoft.Network/networkSecurityGroups"
}
```

## <a name="create-a-virtual-nic"></a>Crear un adaptador de red virtual
Los adaptadores de red (NIC) virtuales están disponibles mediante programación porque se pueden aplicar reglas a su uso. En función del [tamaño de máquina virtual](../sizes.md), puede asociar varias NIC virtuales a una máquina virtual. En el siguiente comando [az network nic create](/cli/azure/network/nic), cree un NIC llamado *myNic* y asócielo al grupo de seguridad de red. La dirección IP pública *myPublicIP* también se asocia al NIC virtual.

```azurecli
az network nic create \
    --resource-group myResourceGroup \
    --name myNic \
    --vnet-name myVnet \
    --subnet mySubnet \
    --public-ip-address myPublicIP \
    --network-security-group myNetworkSecurityGroup
```

Salida:

```json
{
  "NewNIC": {
    "dnsSettings": {
      "appliedDnsServers": [],
      "dnsServers": [],
      "internalDnsNameLabel": null,
      "internalDomainNameSuffix": "brqlt10lvoxedgkeuomc4pm5tb.bx.internal.cloudapp.net",
      "internalFqdn": null
    },
    "enableAcceleratedNetworking": false,
    "enableIpForwarding": false,
    "etag": "W/\"04b5ab44-d8f4-422a-9541-e5ae7de8466d\"",
    "id": "/subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Network/networkInterfaces/myNic",
    "ipConfigurations": [
      {
        "applicationGatewayBackendAddressPools": null,
        "etag": "W/\"04b5ab44-d8f4-422a-9541-e5ae7de8466d\"",
        "id": "/subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Network/networkInterfaces/myNic/ipConfigurations/ipconfig1",
        "loadBalancerBackendAddressPools": null,
        "loadBalancerInboundNatRules": null,
        "name": "ipconfig1",
        "primary": true,
        "privateIpAddress": "192.168.1.4",
        "privateIpAddressVersion": "IPv4",
        "privateIpAllocationMethod": "Dynamic",
        "provisioningState": "Succeeded",
        "publicIpAddress": {
          "dnsSettings": null,
          "etag": null,
          "id": "/subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Network/publicIPAddresses/myPublicIP",
          "idleTimeoutInMinutes": null,
          "ipAddress": null,
          "ipConfiguration": null,
          "location": null,
          "name": null,
          "provisioningState": null,
          "publicIpAddressVersion": null,
          "publicIpAllocationMethod": null,
          "resourceGroup": "myResourceGroup",
          "resourceGuid": null,
          "tags": null,
          "type": null
        },
        "resourceGroup": "myResourceGroup",
        "subnet": {
          "addressPrefix": null,
          "etag": null,
          "id": "/subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Network/virtualNetworks/myVnet/subnets/mySubnet",
          "ipConfigurations": null,
          "name": null,
          "networkSecurityGroup": null,
          "provisioningState": null,
          "resourceGroup": "myResourceGroup",
          "resourceNavigationLinks": null,
          "routeTable": null
        }
      }
    ],
    "location": "eastus",
    "macAddress": null,
    "name": "myNic",
    "networkSecurityGroup": {
      "defaultSecurityRules": null,
      "etag": null,
      "id": "/subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Network/networkSecurityGroups/myNetworkSecurityGroup",
      "location": null,
      "name": null,
      "networkInterfaces": null,
      "provisioningState": null,
      "resourceGroup": "myResourceGroup",
      "resourceGuid": null,
      "securityRules": null,
      "subnets": null,
      "tags": null,
      "type": null
    },
    "primary": null,
    "provisioningState": "Succeeded",
    "resourceGroup": "myResourceGroup",
    "resourceGuid": "b3dbaa0e-2cf2-43be-a814-5cc49fea3304",
    "tags": null,
    "type": "Microsoft.Network/networkInterfaces",
    "virtualMachine": null
  }
}
```


## <a name="create-an-availability-set"></a>Crear un conjunto de disponibilidad
Los conjuntos de disponibilidad ayudan a propagar las máquinas virtuales a los dominios de error y de actualización. Aunque ahora solo cree una máquina virtual, es recomendable usar conjuntos de disponibilidad para facilitar la propagación en el futuro. 

Los dominios de error definen un grupo de máquinas virtuales que comparten una fuente de alimentación común y un conmutador de red. De manera predeterminada, las máquinas virtuales configuradas dentro de su conjunto de disponibilidad se separan en hasta tres dominios de error. Un problema de hardware en uno de estos dominios de error no afecta a todas las máquinas virtuales que ejecutan la aplicación.

Los dominios de actualización indican grupos de máquinas virtuales y hardware físico subyacente que se pueden reiniciar al mismo tiempo. Durante el mantenimiento planeado, es posible que el orden de reinicio de los dominios de actualización no sea secuencial, sino que solo se reinicie un dominio de actualización cada vez.

Azure distribuye automáticamente las máquinas virtuales en los dominios de error y actualización al colocarlas en un conjunto de disponibilidad. Para más información, vea [Administración de la disponibilidad de las máquinas virtuales](manage-availability.md).

Cree un conjunto de disponibilidad para las máquinas virtuales con [az vm availability-set create](/cli/azure/vm/availability-set). En el ejemplo siguiente se crea un conjunto de disponibilidad denominado *myAvailabilitySet*:

```azurecli
az vm availability-set create \
    --resource-group myResourceGroup \
    --name myAvailabilitySet
```

La salida anota los dominios de error y de actualización:

```json
{
  "id": "/subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Compute/availabilitySets/myAvailabilitySet",
  "location": "eastus",
  "managed": null,
  "name": "myAvailabilitySet",
  "platformFaultDomainCount": 2,
  "platformUpdateDomainCount": 5,
  "resourceGroup": "myResourceGroup",
  "sku": {
    "capacity": null,
    "managed": true,
    "tier": null
  },
  "statuses": null,
  "tags": {},
  "type": "Microsoft.Compute/availabilitySets",
  "virtualMachines": []
}
```


## <a name="create-a-vm"></a>Crear una VM
Ha creado los recursos de red para dar soporte a VM con acceso a Internet. Ahora cree una máquina virtual y protéjala con una clave SSH. En este ejemplo, vamos a crear una máquina virtual con Ubuntu basada en la LTS más reciente. Puede encontrar imágenes adicionales con [az vm image list](/cli/azure/vm/image), como se explica en [Finding Azure VM images (Búsqueda de imágenes de máquina virtual de Azure)](cli-ps-findimage.md).

Especifique una clave SSH para usarla para la autenticación. Si no tiene un par de claves públicas de SSH, puede [crearlas](mac-create-ssh-keys.md) o usar el parámetro `--generate-ssh-keys` para crearlas automáticamente. Si ya tiene un par de claves, este parámetro usa las claves existentes en `~/.ssh`.

Cree la máquina virtual al recopilar toda la información y los recursos con el comando [az vm create](/cli/azure/vm). En el ejemplo siguiente se crea una máquina virtual denominada *myVM*:

```azurecli
az vm create \
    --resource-group myResourceGroup \
    --name myVM \
    --location eastus \
    --availability-set myAvailabilitySet \
    --nics myNic \
    --image UbuntuLTS \
    --admin-username azureuser \
    --generate-ssh-keys
```

Inicie sesión mediante SSH en la máquina virtual con la entrada DNS que proporcionó al crear la dirección IP pública. Este `fqdn` se muestra en la salida al crear la máquina virtual:

```json
{
  "fqdns": "mypublicdns.eastus.cloudapp.azure.com",
  "id": "/subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM",
  "location": "eastus",
  "macAddress": "00-0D-3A-13-71-C8",
  "powerState": "VM running",
  "privateIpAddress": "192.168.1.5",
  "publicIpAddress": "13.90.94.252",
  "resourceGroup": "myResourceGroup"
}
```

```bash
ssh azureuser@mypublicdns.eastus.cloudapp.azure.com
```

Salida:

```bash
The authenticity of host 'mypublicdns.eastus.cloudapp.azure.com (13.90.94.252)' can't be established.
ECDSA key fingerprint is SHA256:SylINP80Um6XRTvWiFaNz+H+1jcrKB1IiNgCDDJRj6A.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added 'mypublicdns.eastus.cloudapp.azure.com,13.90.94.252' (ECDSA) to the list of known hosts.
Welcome to Ubuntu 16.04.3 LTS (GNU/Linux 4.11.0-1016-azure x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  Get cloud support with Ubuntu Advantage Cloud Guest:
    https://www.ubuntu.com/business/services/cloud

0 packages can be updated.
0 updates are security updates.


The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

To run a command as administrator (user "root"), use "sudo <command>".
See "man sudo_root" for details.

azureuser@myVM:~$
```

Puede instalar NGINX y ver el flujo de tráfico a la máquina virtual. Instale NGINX como se indica a continuación:

```bash
sudo apt-get install -y nginx
```

Para ver el sitio NGINX predeterminado en acción, abra el explorador web y escriba el FQDN:

![Sitio NGINX predeterminado en la máquina virtual](media/create-cli-complete/nginx.png)

## <a name="export-as-a-template"></a>Exportar como plantilla
¿Y si ahora quiere crear un entorno de desarrollo adicional con los mismos parámetros o un entorno de producción correspondiente? Resource Manager usa plantillas JSON que definen todos los parámetros de su entorno. Puede crear entornos enteros haciendo referencia a esta plantilla JSON. Puede [compilar plantillas JSON manualmente](../../azure-resource-manager/templates/template-syntax.md?toc=/azure/virtual-machines/linux/toc.json) o exportar un entorno existente para que la plantilla JSON se cree automáticamente. Use [az group export](/cli/azure/group) para exportar su grupo de recursos de la siguiente manera:

```azurecli
az group export --name myResourceGroup > myResourceGroup.json
```

Este comando crea el archivo `myResourceGroup.json` en el directorio de trabajo actual. Al crear un entorno a partir de esta plantilla, se le piden todos los nombres de recursos. Puede rellenar estos nombres en el archivo de plantilla si agrega el parámetro `--include-parameter-default-value` al comando `az group export`. Edite su plantilla JSON para especificar los nombres de recursos o [cree un archivo parameters.json](../../azure-resource-manager/templates/template-syntax.md?toc=/azure/virtual-machines/linux/toc.json) que especifique los nombres de recursos.

Para crear un entorno desde su plantilla, use [az group deployment create](/cli/azure/group/deployment) de la siguiente manera:

```azurecli
az group deployment create \
    --resource-group myNewResourceGroup \
    --template-file myResourceGroup.json
```

Es posible que quiera leer [más sobre cómo realizar implementaciones desde plantillas](../../azure-resource-manager/templates/deploy-cli.md?toc=/azure/virtual-machines/linux/toc.json). Obtenga información sobre cómo actualizar los entornos de manera incremental, usar el archivo de parámetros y tener acceso a las plantillas desde una única ubicación de almacenamiento.

## <a name="next-steps"></a>Pasos siguientes
Ya está listo para empezar a trabajar con varios componentes de red y máquinas virtuales. Puede usar este entorno de ejemplo para crear la aplicación con los componentes principales aquí presentados.