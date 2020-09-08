---
title: 'Conexión de redes locales a una red virtual: VPN de sitio a sitio: CLI'
description: Cree una conexión de puerta de enlace VPN de sitio a sitio de IPsec desde la red local hasta una red virtual de Azure a través de la red pública de Internet mediante la CLI.
titleSuffix: Azure VPN Gateway
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: how-to
ms.date: 09/02/2020
ms.author: cherylmc
ms.openlocfilehash: fb7afa9afb72ce16213fd15953b6c82d0fddfeb5
ms.sourcegitcommit: 5a3b9f35d47355d026ee39d398c614ca4dae51c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89401255"
---
# <a name="create-a-virtual-network-with-a-site-to-site-vpn-connection-using-cli"></a>Creación de una red virtual con una conexión VPN de sitio a sitio mediante la CLI

Este artículo muestra cómo utilizar la CLI de Azure para crear una conexión de puerta de enlace VPN de sitio a sitio desde la red local a la red virtual. Los pasos descritos en este artículo se aplican al modelo de implementación de Resource Manager. También se puede crear esta configuración con una herramienta o modelo de implementación distintos, mediante la selección de una opción diferente en la lista siguiente:<br>

> [!div class="op_single_selector"]
> * [Azure Portal](vpn-gateway-howto-site-to-site-resource-manager-portal.md)
> * [PowerShell](vpn-gateway-create-site-to-site-rm-powershell.md)
> * [CLI](vpn-gateway-howto-site-to-site-resource-manager-cli.md)
> * [Portal de Azure clásico](vpn-gateway-howto-site-to-site-classic-portal.md)
> 
>


![Diagrama de la conexión entre locales de VPN Gateway de sitio a sitio](./media/vpn-gateway-howto-site-to-site-resource-manager-cli/site-to-site-diagram.png)

Se utiliza una conexión de puerta de enlace VPN de sitio a sitio para conectar su red local a una red virtual de Azure a través de un túnel VPN de IPsec/IKE (IKEv1 o IKEv2). Este tipo de conexión requiere un dispositivo VPN local que tenga una dirección IP pública asignada. Para más información acerca de las puertas de enlace VPN, consulte [Acerca de VPN Gateway](vpn-gateway-about-vpngateways.md).

## <a name="before-you-begin"></a>Antes de empezar

Antes de comenzar con la configuración, compruebe que se cumplen los criterios siguientes:

* Asegúrese de tener un dispositivo VPN compatible y alguien que pueda configurarlo. Para más información acerca de los dispositivos VPN compatibles y su configuración, consulte [Acerca de los dispositivos VPN](vpn-gateway-about-vpn-devices.md).
* Compruebe que tiene una dirección IPv4 pública externa para el dispositivo VPN.
* Si no está familiarizado con los intervalos de direcciones IP ubicados en la red local, necesita trabajar con alguien que pueda proporcionarle estos detalles. Al crear esta configuración, debe especificar los prefijos del intervalo de direcciones IP al que Azure enrutará la ubicación local. Ninguna de las subredes de la red local puede superponerse con las subredes de la red virtual a la que desea conectarse.
* Puede usar Azure Cloud Shell para ejecutar los comandos de la CLI (consulte las instrucciones a continuación). Sin embargo, si prefiere ejecutar los comandos de forma local, compruebe que haya instalado la última versión de los comandos de la CLI (versión 2.0 o posterior). Para más información acerca de la instalación de los comandos de la CLI, consulte [Instalación de la CLI de Azure](/cli/azure/install-azure-cli) y [Introducción a la CLI de Azure](/cli/azure/get-started-with-azure-cli). 
 
  [!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

### <a name="example-values"></a><a name="example"></a>Valores del ejemplo

Puede usar los siguientes valores para crear un entorno de prueba o hacer referencia a ellos para comprender mejor los ejemplos de este artículo:

```
#Example values

VnetName                = TestVNet1 
ResourceGroup           = TestRG1 
Location                = eastus 
AddressSpace            = 10.11.0.0/16 
SubnetName              = Subnet1 
Subnet                  = 10.11.0.0/24 
GatewaySubnet           = 10.11.255.0/27 
LocalNetworkGatewayName = Site2 
LNG Public IP           = <VPN device IP address>
LocalAddrPrefix1        = 10.0.0.0/24
LocalAddrPrefix2        = 20.0.0.0/24   
GatewayName             = VNet1GW 
PublicIP                = VNet1GWIP 
VPNType                 = RouteBased 
GatewayType             = Vpn 
ConnectionName          = VNet1toSite2
```

## <a name="1-connect-to-your-subscription"></a><a name="Login"></a>1. su suscripción

Si decide ejecutar la CLI de forma local, conéctese a su suscripción. Si está usando Azure Cloud Shell en el explorador, no necesita conectarse a su suscripción. Se conectará automáticamente en Azure Cloud Shell. Sin embargo, es posible que quiera comprobar si está usando la suscripción correcta después de conectarse.

[!INCLUDE [CLI login](../../includes/vpn-gateway-cli-login-include.md)]

## <a name="2-create-a-resource-group"></a><a name="rg"></a>2. Crear un grupo de recursos

En el ejemplo siguiente se crea un grupo de recursos con el nombre 'TestRG1' en la ubicación 'eastus'. Si ya tiene un grupo de recursos en la región que desea crear la red virtual, puede utilizarlo.

```azurecli-interactive
az group create --name TestRG1 --location eastus
```

## <a name="3-create-a-virtual-network"></a><a name="VNet"></a>3. Creación de una red virtual

Si aún no tiene una red virtual, créela con el comando [az network vnet create](/cli/azure/network/vnet). Al crear una red virtual, compruebe que los espacios de direcciones especificados no se superponen con los espacios de direcciones que existen en la red local.

>[!NOTE]
>Para que esta red virtual se conecte a una ubicación local, debe coordinarse con el administrador de la red local para delimitar un intervalo de direcciones IP que pueda usar específicamente para esta red virtual. Si existe un intervalo de direcciones duplicado en ambos lados de la conexión VPN, el tráfico no se enrutará como cabría esperar. Además, si desea conectar esta red virtual a otra red virtual, el espacio de direcciones no se puede superponer con otra red virtual. Por consiguiente, tenga cuidado al planear la configuración de red.
>
>

El siguiente ejemplo crea una red virtual denominada 'TestVNet1' y una subred llamada 'Subnet1'.

```azurecli-interactive
az network vnet create --name TestVNet1 --resource-group TestRG1 --address-prefix 10.11.0.0/16 --location eastus --subnet-name Subnet1 --subnet-prefix 10.11.0.0/24
```

## <a name="4-create-the-gateway-subnet"></a>4. <a name="gwsub"></a>Creación de la subred de la puerta de enlace


[!INCLUDE [About gateway subnets](../../includes/vpn-gateway-about-gwsubnet-include.md)]

Use el comando [az network vnet subnet create](/cli/azure/network/vnet/subnet) para la subred de la puerta de enlace.

```azurecli-interactive
az network vnet subnet create --address-prefix 10.11.255.0/27 --name GatewaySubnet --resource-group TestRG1 --vnet-name TestVNet1
```

[!INCLUDE [vpn-gateway-no-nsg](../../includes/vpn-gateway-no-nsg-include.md)]

## <a name="5-create-the-local-network-gateway"></a><a name="localnet"></a>5. Creación de la puerta de enlace de red local

La puerta de enlace de red local suele hacer referencia a la ubicación local. Asigne al sitio un nombre al que Azure pueda hacer referencia y, luego, especifique la dirección IP del dispositivo VPN local con la que creará una conexión. Especifique también los prefijos de dirección IP que se enrutarán a través de la puerta de enlace VPN al dispositivo VPN. Los prefijos de dirección que especifique son los prefijos que se encuentran en la red local. Puede actualizar fácilmente estos prefijos si cambia su red local.

Use los valores siguientes:

* *--gateway-ip-address* es la dirección IP del dispositivo VPN local.
* *--local-address-prefixes* son los espacios de direcciones locales.

Use el comando [az network local-gateway create](/cli/azure/network/local-gateway) para agregar una puerta de enlace de red local con varios prefijos de direcciones:

```azurecli-interactive
az network local-gateway create --gateway-ip-address 23.99.221.164 --name Site2 --resource-group TestRG1 --local-address-prefixes 10.0.0.0/24 20.0.0.0/24
```

## <a name="6-request-a-public-ip-address"></a><a name="PublicIP"></a>6. Solicite una dirección IP pública

Una puerta de enlace VPN debe tener una dirección IP pública. Primero se solicita el recurso de la dirección IP y, después, se hace referencia a él al crear la puerta de enlace de red virtual. La dirección IP se asigna dinámicamente al recurso cuando se crea la puerta de enlace VPN. Actualmente, VPN Gateway solo admite la asignación de direcciones IP públicas *dinámicas*. No se puede solicitar una asignación de direcciones IP públicas estáticas. Sin embargo, esto no significa que la dirección IP cambia después de que se ha asignado a una puerta de enlace VPN. La única vez que la dirección IP pública cambia es cuando la puerta de enlace se elimina y se vuelve a crear. No cambia cuando se cambia el tamaño, se restablece o se realizan actualizaciones u otras operaciones de mantenimiento interno de una puerta de enlace VPN.

Use el comando [az network public-ip create](/cli/azure/network/public-ip) para solicitar una dirección IP pública dinámica.

```azurecli-interactive
az network public-ip create --name VNet1GWIP --resource-group TestRG1 --allocation-method Dynamic
```

## <a name="7-create-the-vpn-gateway"></a><a name="CreateGateway"></a>7. Creación de la puerta de enlace VPN

Cree la puerta de enlace VPN de la red virtual. Crear una puerta de enlace VPN puede tardar 45 minutos o más en completarse.

Use los valores siguientes:

* El valor *---gateway-type* para una configuración de sitio a sitio es *Vpn*. El tipo de puerta de enlace siempre es específico de la configuración que se va a implementar. Para más información, consulte [Tipos de puerta de enlace](vpn-gateway-about-vpn-gateway-settings.md#gwtype).
* El valor de *--vpn-type* puede ser *RouteBased* (denominada puerta de enlace dinámica en algunos documentos) o *PolicyBased* (denominada puerta de enlace estática en algunos documentos). La configuración es específica según los requisitos del dispositivo al que se va a conectar. Para más información sobre los tipos de puertas de enlace de VPN, consulte [Acerca de la información de VPN Gateway](vpn-gateway-about-vpn-gateway-settings.md#vpntype).
* Seleccione la SKU de la puerta de enlace que desea utilizar. Hay limitaciones de configuración para determinadas SKU. Consulte [SKU de puertas de enlace](vpn-gateway-about-vpn-gateway-settings.md#gwsku) para más información.

Cree la puerta de enlace VPN con el comando [az network vnet-gateway create](/cli/azure/network/vnet-gateway). Si este comando se ejecuta con el parámetro '--no-wait', no se verán los comentarios o resultados. Este parámetro permite que la puerta de enlace se cree en segundo plano. Tarda aproximadamente 45 minutos en crear una puerta de enlace.

```azurecli-interactive
az network vnet-gateway create --name VNet1GW --public-ip-address VNet1GWIP --resource-group TestRG1 --vnet TestVNet1 --gateway-type Vpn --vpn-type RouteBased --sku VpnGw1 --no-wait 
```

## <a name="8-configure-your-vpn-device"></a><a name="VPNDevice"></a>8. Configurar el dispositivo VPN

Las conexiones de sitio a sitio a una red local requieren un dispositivo VPN. En este paso, se configura el dispositivo VPN. Al configurar el dispositivo VPN, necesita lo siguiente:

- Una clave compartida. Se trata de la misma clave compartida que se especifica al crear la conexión VPN de sitio a sitio. En estos ejemplos se utiliza una clave compartida básica. Se recomienda que genere y utilice una clave más compleja.
- La dirección IP pública de la puerta de enlace de red virtual. Puede ver la dirección IP pública mediante Azure Portal, PowerShell o la CLI.  Para buscar la dirección IP pública de la puerta de enlace de red virtual, use el comando [az network public-ip list](/cli/azure/network/public-ip). Para facilitar la lectura, la salida muestra la lista de direcciones IP públicas en formato de tabla.

  ```azurecli-interactive
  az network public-ip list --resource-group TestRG1 --output table
  ```


[!INCLUDE [Configure VPN device](../../includes/vpn-gateway-configure-vpn-device-rm-include.md)]


## <a name="9-create-the-vpn-connection"></a><a name="CreateConnection"></a>9. Creación de la conexión VPN

Creación de la conexión VPN de sitio a sitio entre la puerta de enlace de la red virtual y el dispositivo VPN local. Preste especial atención al valor de la clave compartida, que debe coincidir con el valor de la clave compartida configurada para el dispositivo VPN.

Cree la conexión mediante el comando [az network vpn-connection create](/cli/azure/network/vpn-connection).

```azurecli-interactive
az network vpn-connection create --name VNet1toSite2 --resource-group TestRG1 --vnet-gateway1 VNet1GW -l eastus --shared-key abc123 --local-gateway2 Site2
```

Pasado un momento, se establecerá la conexión.

## <a name="10-verify-the-vpn-connection"></a><a name="toverify"></a>10. Comprobación de la conexión VPN

[!INCLUDE [verify connection](../../includes/vpn-gateway-verify-connection-cli-rm-include.md)]

Si desea usar otro método para comprobar la conexión, consulte [Comprobación de una conexión de VPN Gateway](vpn-gateway-verify-connection-resource-manager.md).

## <a name="to-connect-to-a-virtual-machine"></a><a name="connectVM"></a>Conexión a una máquina virtual

[!INCLUDE [Connect to a VM](../../includes/vpn-gateway-connect-vm-s2s-include.md)]

## <a name="common-tasks"></a><a name="tasks"></a>Tareas comunes

Esta sección contiene comandos comunes que son útiles al trabajar con configuraciones de sitio a sitio. Para obtener la lista completa de los comandos de red de la CLI, consulte [Azure CLI - Networking](/cli/azure/network) (CLI de Azure - Redes).

[!INCLUDE [local network gateway common tasks](../../includes/vpn-gateway-common-tasks-cli-include.md)]

## <a name="next-steps"></a>Pasos siguientes

* Una vez completada la conexión, puede agregar máquinas virtuales a las redes virtuales. Consulte [Virtual Machines](https://docs.microsoft.com/azure/) para más información.
* Para más información acerca de BGP, consulte [Información general de BGP](vpn-gateway-bgp-overview.md) y [Configuración de BGP](vpn-gateway-bgp-resource-manager-ps.md).
* Para más información acerca de la tunelización forzada, consulte la [información acerca de la tunelización forzada](vpn-gateway-forced-tunneling-rm.md).
* Para obtener información acerca de las conexiones activo/activo de alta disponibilidad, consulte [Conectividad de alta disponibilidad entre locales y de red virtual a red virtual](vpn-gateway-highlyavailable.md).
* Para obtener una lista de comandos de red de la CLI de Azure, consulte [Networking - az network](https://docs.microsoft.com/cli/azure/network) (Redes: az network).
* Para obtener información acerca de cómo crear una conexión VPN de sitio a sitio mediante una plantilla de Azure Resource Manager, consulte [Create a Site-to-Site VPN Connection](https://azure.microsoft.com/resources/templates/101-site-to-site-vpn-create/) (Creación de una conexión VPN de sitio a sitio).
* Para obtener información acerca de cómo crear una conexión VPN entre redes virtuales mediante una plantilla de Azure Resource Manager, consulte [Deploy HBase geo replication](https://azure.microsoft.com/resources/templates/101-hdinsight-hbase-replication-geo/) (Implementación de replicación geográfica de HBase).
