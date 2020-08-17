---
title: 'Creación de una puerta de enlace de NAT: CLI de Azure'
titlesuffix: Azure Virtual Network NAT
description: En este inicio rápido se muestra cómo crear una puerta de enlace de NAT mediante la CLI de Azure
services: virtual-network
documentationcenter: na
author: asudbring
manager: KumudD
Customer intent: I want to create a NAT gateway for outbound connectivity for my virtual network.
ms.service: virtual-network
ms.subservice: nat
ms.devlang: na
ms.topic: how-to
ms.workload: infrastructure-services
ms.date: 02/18/2020
ms.author: allensu
ms.custom: devx-track-azurecli
ms.openlocfilehash: a44a3af7be214aa2ed000eb824b63c0bf7a95aee
ms.sourcegitcommit: 269da970ef8d6fab1e0a5c1a781e4e550ffd2c55
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/10/2020
ms.locfileid: "88054033"
---
# <a name="create-a-nat-gateway-using-azure-cli"></a>Creación de una puerta de enlace de NAT con la CLI de Azure

En este tutorial se muestra cómo usar el servicio Azure Virtual Network NAT. Creará una puerta de enlace de NAT para proporcionar conectividad saliente para una máquina virtual en Azure. 


[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

Puede realizar este tutorial con Azure Cloud Shell o ejecutar los comandos respectivos de forma local.  Si nunca ha usado Azure Cloud Shell, [inicie sesión ahora](https://shell.azure.com) para recorrer la configuración inicial.
Si elige ejecutar estos comandos de forma local, debe instalar la CLI.  En este tutorial es necesario ejecutar la versión 2.0.71 de la CLI de Azure o cualquier versión posterior. Para encontrar la versión, ejecute `az --version`. Si necesita instalarla o actualizarla, vea [Instalación de la CLI de Azure]( /cli/azure/install-azure-cli).


## <a name="create-a-resource-group"></a>Crear un grupo de recursos

Cree un grupo de recursos con [az group create](https://docs.microsoft.com/cli/azure/group). Un grupo de recursos de Azure es un contenedor lógico en el que se implementan y se administran los recursos de Azure.

En el ejemplo siguiente se crea un grupo de recursos llamado **myResourceGroupNAT** en la ubicación **eastus2**:

```azurecli-interactive
  az group create \
    --name myResourceGroupNAT \
    --location eastus2
```

## <a name="create-the-nat-gateway"></a>Creación de la puerta de enlace de NAT

### <a name="create-a-public-ip-address"></a>Crear una dirección IP pública

Para acceder a la red pública de Internet, necesita una o varias direcciones IP públicas para la puerta de enlace de NAT. Use [az network public-ip create](https://docs.microsoft.com/cli/azure/network/public-ip) para crear un recurso de dirección IP pública llamado **myPublicIP** en **myResourceGroupNAT**.

```azurecli-interactive
  az network public-ip create \
    --resource-group myResourceGroupNAT \
    --name myPublicIP \
    --sku standard
```

### <a name="create-a-public-ip-prefix"></a>Creación de un prefijo de dirección IP pública

Puede usar uno o varios recursos de dirección IP pública o prefijos de dirección IP pública (o ambos) con la puerta de enlace de NAT. Para demostrarlo, vamos a agregar un recurso de prefijo de dirección IP pública a este escenario.   Use [az network public-ip prefix create](https://docs.microsoft.com/cli/azure/network/public-ip/prefix#az-network-public-ip-prefix-create) para crear un recurso de prefijo de dirección IP pública llamado **myPublicIPprefix** en **myResourceGroupNAT**.

```azurecli-interactive
  az network public-ip prefix create \
    --resource-group myResourceGroupNAT \
    --name myPublicIPprefix \
    --length 31
```

### <a name="create-a-nat-gateway-resource"></a>Creación de un recurso de puerta de enlace de NAT

En esta sección se detalla cómo crear y configurar los componentes siguientes del servicio NAT mediante el recurso de puerta de enlace de NAT:
  - Un grupo de direcciones IP públicas y un prefijo de dirección IP pública que se usarán en los flujos salientes traducidos por el recurso de puerta de enlace de NAT.
  - Cambie el tiempo de espera de inactividad de 4 (el valor predeterminado) a 10 minutos.

Cree una puerta de enlace de NAT de Azure global con [az network nat gateway create](https://docs.microsoft.com/cli/azure/network/nat?view=azure-cli-latest) llamada **myNATgateway**. El comando usa la dirección IP pública **myPublicIP** y el prefijo de dirección IP pública **myPublicIPprefix**. El comando cambia el tiempo de espera de inactividad a **10** minutos.

```azurecli-interactive
  az network nat gateway create \
    --resource-group myResourceGroupNAT \
    --name myNATgateway \
    --public-ip-addresses myPublicIP \
    --public-ip-prefixes myPublicIPprefix \
    --idle-timeout 10       
  ```

En este momento, la puerta de enlace de NAT es funcional y todo lo que queda es configurar qué subredes de una red virtual deben usarla.

## <a name="configure-virtual-network"></a>Configurar la red virtual

Antes de implementar una máquina virtual y poder usar la puerta de enlace de NAT, es necesario crear la red virtual.

Cree una red virtual llamada **myVnet** con una subred llamada **mySubnet** en **myResourceGroupNAT** con el comando [az network vnet create](https://docs.microsoft.com/cli/azure/network/vnet).  El espacio de direcciones IP de la red virtual es **192.168.0.0/16**. La subred de la red virtual es **192.168.0.0/24**.

```azurecli-interactive
  az network vnet create \
    --resource-group myResourceGroupNAT \
    --location eastus2 \
    --name myVnet \
    --address-prefix 192.168.0.0/16 \
    --subnet-name mySubnet \
    --subnet-prefix 192.168.0.0/24
```

### <a name="configure-nat-service-for-source-subnet"></a>Configuración del servicio NAT para la subred de origen

Se va a configurar la subred de origen **mySubnet** de la red virtual **myVnet** para usar el recurso de puerta de enlace de NAT concreto **myNATgateway** con [az network vnet subnet update](https://docs.microsoft.com/cli/azure/network/vnet/subnet).  Este comando activará el servicio NAT en la subred especificada.

```azurecli-interactive
  az network vnet subnet update \
    --resource-group myResourceGroupNAT \
    --vnet-name myVnet \
    --name mySubnet \
    --nat-gateway myNATgateway
```

Todo el tráfico saliente a destinos de Internet usa ahora la puerta de enlace de NAT.  No es necesario configurar una UDR.

## <a name="create-a-vm-to-use-the-nat-service"></a>Creación de una máquina virtual para usar el servicio NAT

Ahora se va a crear una máquina virtual para usar el servicio NAT.  Esta máquina virtual tiene una dirección IP pública en el nivel de instancia que le permite el acceso a la máquina virtual.  El servicio NAT tiene en cuenta la dirección del flujo y reemplazará el destino predeterminado de Internet en la subred. La dirección IP pública de la máquina virtual no se usará para las conexiones salientes.

### <a name="create-public-ip-for-source-vm"></a>Creación de una dirección IP pública para la máquina virtual de origen

Se va a crear una dirección IP pública que se usará para acceder a la máquina virtual.  Use [az network public-ip create](https://docs.microsoft.com/cli/azure/network/public-ip) para crear un recurso de dirección IP pública llamado **myPublicIPVM** en **myResourceGroupNAT**.

```azurecli-interactive
  az network public-ip create \
    --resource-group myResourceGroupNAT \
    --name myPublicIPVM \
    --sku standard
```

### <a name="create-an-nsg-for-vm"></a>Creación de un grupo de seguridad de red para la máquina virtual

Dado que las direcciones IP públicas estándar son "seguras de forma predeterminada", es necesario crear un grupo de seguridad de red para permitir el acceso de entrada mediante SSH. Use [az network nsg create](https://docs.microsoft.com/cli/azure/network/nsg?view=azure-cli-latest#az-network-nsg-create) para crear un recurso de grupo de seguridad de red llamado **myNSG** en **myResourceGroupNAT**.

```azurecli-interactive
  az network nsg create \
    --resource-group myResourceGroupNAT \
    --name myNSG 
```

### <a name="expose-ssh-endpoint-on-source-vm"></a>Exposición del punto de conexión SSH en la máquina virtual de origen

Se va a crear una regla en el grupo de seguridad de red para el acceso mediante SSH a la máquina virtual de origen. Use [az network nsg rule create](https://docs.microsoft.com/cli/azure/network/nsg/rule?view=azure-cli-latest#az-network-nsg-rule-create) para crear una regla de grupo de seguridad de red llamada **ssh** en el grupo de seguridad de red llamado **myNSG** en **myResourceGroupNAT**.

```azurecli-interactive
  az network nsg rule create \
    --resource-group myResourceGroupNAT \
    --nsg-name myNSG \
    --priority 100 \
    --name ssh \
    --description "SSH access" \
    --access allow \
    --protocol tcp \
    --direction inbound \
    --destination-port-ranges 22
```

### <a name="create-nic-for-vm"></a>Creación de una NIC para una máquina virtual

Cree una interfaz de red con el comando [az network nic create](/cli/azure/network/nic#az-network-nic-create) y asóciela a la dirección IP pública y al grupo de seguridad de red. 

```azurecli-interactive
  az network nic create \
    --resource-group myResourceGroupNAT \
    --name myNic \
    --vnet-name myVnet \
    --subnet mySubnet \
    --public-ip-address myPublicIPVM \
    --network-security-group myNSG
```

### <a name="create-vm"></a>Creación de una máquina virtual

Cree la máquina virtual con [az vm create](/cli/azure/vm#az-vm-create).  Se van a generar claves SSH para esta máquina virtual y se va a almacenar la clave privada para usarla más adelante.

 ```azurecli-interactive
  az vm create \
    --resource-group myResourceGroupNAT \
    --name myVM \
    --nics myNic \
    --image UbuntuLTS \
    --generate-ssh-keys
```

Espere a que la máquina virtual se implemente y continúe con el resto de los pasos.

## <a name="discover-the-ip-address-of-the-vm"></a>Detección de la dirección IP de la máquina virtual

En primer lugar, es necesario detectar la dirección IP de la máquina virtual que ha creado. Para recuperar la dirección IP pública de la máquina virtual, use [az network public-ip show](/cli/azure/network/public-ip#az-network-public-ip-show). 

```azurecli-interactive
  az network public-ip show \
    --resource-group myResourceGroupNAT \
    --name myPublicIPVM \
    --query [ipAddress] \
    --output tsv
``` 

>[!IMPORTANT]
>Copie la dirección IP pública y, luego, péguela en un bloc de notas para que pueda usarla para acceder a la máquina virtual.

### <a name="sign-in-to-vm"></a>Inicio de sesión en la máquina virtual

Las credenciales de SSH deben almacenarse en la instancia de Cloud Shell de la operación anterior.  Abra [Azure Cloud Shell](https://shell.azure.com) en el explorador. Use la dirección IP recuperada en el paso anterior para conectarse a la máquina virtual mediante SSH.

```bash
ssh <ip-address-destination>
```

Ahora está listo para usar el servicio NAT.

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no lo necesite, puede usar el comando [az group delete](/cli/azure/group#az-group-delete) para quitar el grupo de recursos y todos los recursos que contiene.

```azurecli-interactive 
  az group delete \
    --name myResourceGroupNAT
```

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha creado una puerta de enlace de NAT y una máquina virtual para usarla. 

Revise las métricas de Azure Monitor para ver el funcionamiento del servicio NAT. Diagnostique problemas como el agotamiento de recursos de los puertos SNAT disponibles.  El agotamiento de recursos de los puertos SNAT se soluciona agregando recursos de dirección IP pública o recursos de prefijo de dirección IP pública adicionales (o ambos).


- Obtenga más información sobre [Azure Virtual Network NAT](./nat-overview.md).
- Más información sobre [recursos de puerta de enlace de NAT](./nat-gateway-resource.md).
- Inicio rápido para la implementación de [recursos de puerta de enlace de NAT con la CLI de Azure](./quickstart-create-nat-gateway-cli.md).
- Inicio rápido para la implementación de [recursos de puerta de enlace de NAT con PowerShell](./quickstart-create-nat-gateway-powershell.md).
- Inicio rápido para la implementación de [recursos de puerta de enlace de NAT con Azure Portal](./quickstart-create-nat-gateway-portal.md).
> [!div class="nextstepaction"]

