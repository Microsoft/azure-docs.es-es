---
title: 'Conexión de redes virtuales clásicas a redes virtuales de Azure Resource Manager: PowerShell'
description: Cree una conexión VPN entre redes virtuales clásicas y redes de Resource Manager mediante VPN Gateway y PowerShell.
services: vpn-gateway
titleSuffix: Azure VPN Gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: how-to
ms.date: 10/17/2018
ms.author: cherylmc
ms.openlocfilehash: 2c9b8a769dec1a2aa461a34203c98a228cf71d16
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87082059"
---
# <a name="connect-virtual-networks-from-different-deployment-models-using-powershell"></a>Conexión de redes virtuales a partir de diferentes modelos de implementación con PowerShell

Este artículo muestra cómo conectar redes virtuales clásicas a redes virtuales de Resource Manager para permitir que los recursos que se encuentran en modelos de implementación independientes se comuniquen entre sí. En los pasos de este artículo se usa fundamentalmente PowerShell, pero también se puede crear esta configuración con Azure Portal si se selecciona el artículo en esta lista.

> [!div class="op_single_selector"]
> * [Portal](vpn-gateway-connect-different-deployment-models-portal.md)
> * [PowerShell](vpn-gateway-connect-different-deployment-models-powershell.md)
> 
> 

La conexión de una red virtual clásica a una red virtual de Resource Manager es similar a la conexión de una red virtual a una ubicación de sitio local. Ambos tipos de conectividad usan una puerta de enlace de VPN para proporcionar un túnel seguro con IPsec/IKE. Puede crear una conexión entre redes virtuales que estén en diferentes suscripciones y en diferentes regiones. También puede conectar redes virtuales que tengan ya conexiones a redes locales, siempre que la puerta de enlace con la que se hayan configurado sea dinámica o basada en ruta. Para más información acerca de las conexiones de red virtual a red virtual, consulte [P+F sobre conexiones de red virtual a red virtual](#faq) al final de este artículo. 

Si aún no tiene una puerta de enlace de red virtual y no desea crear una, considere la posibilidad de conectar sus redes virtuales mediante Emparejamiento de VNET. El emparejamiento de VNET no usa VPN Gateway. Para más información, consulte [Emparejamiento de VNET](../virtual-network/virtual-network-peering-overview.md).

## <a name="before-you-begin"></a><a name="before"></a>Antes de empezar

Los siguientes pasos le guiarán a través de los valores necesarios para configurar una puerta de enlace dinámica o basada en ruta para cada red virtual y crear una conexión VPN entre las puertas de enlace. Esta configuración no admite puertas de enlace estáticas o basadas en directivas.

### <a name="prerequisites"></a><a name="pre"></a>Requisitos previos

* Se han creado ambas redes virtuales. Si tiene que crear una red virtual del administrador de recursos, consulte [Creación de un grupo de recursos y una red virtual](../virtual-network/quick-create-powershell.md#create-a-resource-group-and-a-virtual-network). Para crear una red virtual clásica, consulte [Create a classic VNet](https://docs.microsoft.com/azure/virtual-network/create-virtual-network-classic) (Creación de una red virtual clásica).
* Los intervalos de direcciones de las redes virtuales no se superponen entre sí ni con alguno de los intervalos de otras conexiones con las que puedan estar conectadas las puertas de enlace.
* Tiene instalados los últimos cmdlets de PowerShell. Para obtener más información, vea [Instalación y configuración de Azure PowerShell](/powershell/azure/) . Asegúrese de instalar los cmdlets tanto de Service Management (SM) como de Resource Manager (RM). 

### <a name="example-settings"></a><a name="exampleref"></a>Configuración de ejemplo

Puede usar estos valores para crear un entorno de prueba o hacer referencia a ellos para comprender mejor los ejemplos de este artículo.

**Configuración de redes virtuales clásicas**

Nombre de la red virtual = RedVClásica <br>
Ubicación = Oeste de EE. UU. <br>
Espacios de direcciones de red virtual = 10.0.0.0/24 <br>
Subred-1 = 10.0.0.0/27 <br>
Subred de la puerta de enlace= 10.0.0.32/29 <br>
Nombre de la red local = RedLocalRMV <br>
Tipo de puerta de enlace = EnrutamientoDinámico

**Configuración de redes virtuales de Resource Manager**

Nombre de red virtual = RedRMV <br>
Grupo de recursos = GR1 <br>
Espacios de direcciones IP de red virtual = 192.168.0.0/16 <br>
Subred-1 = 192.168.1.0/24 <br>
Subred de la puerta de enlace = 192.168.0.0/26 <br>
Ubicación = Este de EE. UU. <br>
Nombre IP público de puerta de enlace = pepip <br>
Puerta de enlace de red local = LocalRedVClásica <br>
Nombre de la puerta de enlace de red virtual = PuertaDeEnlaceRM <br>
Configuración de direccionamiento IP de puerta de enlace = configpeip

## <a name="section-1---configure-the-classic-vnet"></a><a name="createsmgw"></a>Sección 1: configuración de la red virtual clásica
### <a name="1-download-your-network-configuration-file"></a>1. Descarga del archivo de configuración de red
1. Inicie sesión en su cuenta de Azure en la consola de PowerShell con derechos elevados. El siguiente cmdlet pide las credenciales de inicio de sesión de la cuenta de Azure. Después de iniciar la sesión, se descarga la configuración de la cuenta a fin de ponerla a disposición para Azure PowerShell. Los cmdlets clásicos de Azure PowerShell para Service Management (SM) clásicos se usan en esta sección.

   ```azurepowershell
   Add-AzureAccount
   ```

   Obtenga la suscripción a Azure.

   ```azurepowershell
   Get-AzureSubscription
   ```

   Si tiene varias suscripciones, seleccione la que quiera usar.

   ```azurepowershell
   Select-AzureSubscription -SubscriptionName "Name of subscription"
   ```
2. Exporte el archivo de configuración de red de Azure mediante la ejecución del comando siguiente. Puede cambiar la ubicación del archivo que se va a exportar a una ubicación diferente si es necesario.

   ```azurepowershell
   Get-AzureVNetConfig -ExportToFile C:\AzureNet\NetworkConfig.xml
   ```
3. Abra el archivo .xml que ha descargado para editarlo. Para obtener un ejemplo del archivo de configuración de red, consulte [Esquema de configuración de red virtual](https://msdn.microsoft.com/library/jj157100.aspx).

### <a name="2-verify-the-gateway-subnet"></a>2. Comprobación de la subred de puerta de enlace
En el elemento **VirtualNetworkSites** , agregue una subred de puerta de enlace a la red virtual si no hay ya una creada. Al trabajar con el archivo de configuración de red, la puerta de enlace DEBE tener el nombre "GatewaySubnet" o Azure no podrá reconocerla y utilizarla como subred de puerta de enlace.

[!INCLUDE [vpn-gateway-no-nsg-include](../../includes/vpn-gateway-no-nsg-include.md)]

**Ejemplo**:

```xml
<VirtualNetworkSites>
  <VirtualNetworkSite name="ClassicVNet" Location="West US">
    <AddressSpace>
      <AddressPrefix>10.0.0.0/24</AddressPrefix>
    </AddressSpace>
    <Subnets>
      <Subnet name="Subnet-1">
        <AddressPrefix>10.0.0.0/27</AddressPrefix>
      </Subnet>
      <Subnet name="GatewaySubnet">
        <AddressPrefix>10.0.0.32/29</AddressPrefix>
      </Subnet>
    </Subnets>
  </VirtualNetworkSite>
</VirtualNetworkSites>
```

### <a name="3-add-the-local-network-site"></a>3. Incorporación del sitio de red local
El sitio de red local que agregue representa la red virtual de RM a la que desea conectarse. Agregue un elemento **LocalNetworkSites** al archivo si aún no existe. En este punto de la configuración, el valor de VPNGatewayAddress puede ser cualquier dirección IP pública válida ya que todavía no se ha creado la puerta de enlace para la red virtual de Resource Manager. Una vez creada la puerta de enlace, esta dirección IP de marcador de posición se reemplaza por la dirección IP pública correcta que se ha asignado a la puerta de enlace de RM.

```xml
<LocalNetworkSites>
  <LocalNetworkSite name="RMVNetLocal">
    <AddressSpace>
      <AddressPrefix>192.168.0.0/16</AddressPrefix>
    </AddressSpace>
    <VPNGatewayAddress>13.68.210.16</VPNGatewayAddress>
  </LocalNetworkSite>
</LocalNetworkSites>
```

### <a name="4-associate-the-vnet-with-the-local-network-site"></a>4. Asociación de la red virtual con el sitio de red local
En esta sección, se especifica el sitio de red local con la que se desea conectar la red virtual. En este caso, es la red virtual de Resource Manager a la que se ha hecho referencia anteriormente. Asegúrese de que los nombres coincidan. En este paso no se crea una puerta de enlace. Se especifica la red local con la que se conectará la puerta de enlace.

```xml
<Gateway>
  <ConnectionsToLocalNetwork>
    <LocalNetworkSiteRef name="RMVNetLocal">
      <Connection type="IPsec" />
    </LocalNetworkSiteRef>
  </ConnectionsToLocalNetwork>
</Gateway>
```

### <a name="5-save-the-file-and-upload"></a>5. Guardado del archivo y carga
Ejecute el comando siguiente para guardar el archivo e importarlo a Azure. Asegúrese de cambiar la ruta de acceso según sea necesario para su entorno.

```azurepowershell
Set-AzureVNetConfig -ConfigurationPath C:\AzureNet\NetworkConfig.xml
```

Verá un resultado similar que muestra que la importación se realizó correctamente.

```output
OperationDescription        OperationId                      OperationStatus                                                
--------------------        -----------                      ---------------                                                
Set-AzureVNetConfig        e0ee6e66-9167-cfa7-a746-7casb9    Succeeded 
```

### <a name="6-create-the-gateway"></a>6. Creación de la puerta de enlace

Antes de ejecutar este ejemplo, consulte el archivo de configuración de red que ha descargado para consultar los nombres exactos que Azure espera ver. El archivo de configuración de red contiene los valores para las redes virtuales clásicas. A veces se cambian los nombres de los sitios de las redes virtuales clásicas en el archivo de configuración de red al crear la configuración de la red virtual clásica en Azure Portal debido a las diferencias en los modelos de implementación. Por ejemplo, si ha utilizado Azure Portal para crear una red virtual clásica llamada 'Classic VNet' y la ha creado en un grupo de recursos llamado 'ClassicRG', el nombre que se encuentra en el archivo de configuración de red se convierte en 'Group ClassicRG Classic VNet'. Al especificar el nombre de una red virtual que contiene espacios, utilice comillas alrededor del valor.


Para crear una puerta de enlace de enrutamiento dinámico, utilice el siguiente ejemplo:

```azurepowershell
New-AzureVNetGateway -VNetName ClassicVNet -GatewayType DynamicRouting
```

Puede comprobar el estado de la puerta de enlace usando el cmdlet **Get-AzureVNetGateway**.

## <a name="section-2---configure-the-rm-vnet-gateway"></a><a name="creatermgw"></a>Sección 2: Configuración de la puerta de enlace de la red virtual de RM



Para los requisitos previos, se supone que ya ha creado una red virtual de RM. En este paso, creará una puerta de enlace de VPN para la red virtual de RM. No empiece con los pasos hasta que haya recuperado la dirección IP pública de la puerta de enlace de la red virtual clásica. 

1. Inicie sesión en su cuenta de Azure en la consola de PowerShell. El siguiente cmdlet pide las credenciales de inicio de sesión de la cuenta de Azure. Después de iniciar la sesión, se descarga la configuración de la cuenta para que esté disponible para Azure PowerShell. Asimismo, tiene la opción de usar la característica "Pruébelo" para iniciar Azure Cloud Shell en el explorador.

   Si usa Azure Cloud Shell, omita el siguiente cmdlet:

   ```azurepowershell
   Connect-AzAccount
   ``` 
   Para comprobar que está usando la suscripción correcta, ejecute el siguiente cmdlet:  

   ```azurepowershell-interactive
   Get-AzSubscription
   ```
   
   Si tiene varias suscripciones, seleccione la que quera usar.

   ```azurepowershell-interactive
   Select-AzSubscription -SubscriptionName "Name of subscription"
   ```
2. Cree una puerta de enlace de red local. En una red virtual, la puerta de enlace de red local suele hacer referencia a la ubicación local. En este caso, la puerta de enlace de red local hace referencia a la red virtual clásica. Asígnele un nombre que sirva de referencia a Azure y especifique también el prefijo de espacio de direcciones. Azure usa el prefijo de dirección IP que especifique para identificar qué tráfico enviar a la ubicación local. Si necesita ajustar la información aquí posteriormente, antes de crear la puerta de enlace, puede modificar los valores y volver a ejecutar el ejemplo.
   
   **-Name** es el nombre de referencia que quiere asignar a la puerta de enlace de la red local.<br>
   **-AddressPrefix** es el espacio de direcciones para la red virtual clásica.<br>
   **-GatewayIpAddress** es la dirección IP pública de la puerta de enlace de la red virtual clásica. Asegúrese de cambiar el texto de ejemplo "n.n.n.n" para que refleje la dirección IP correcta.<br>

   ```azurepowershell-interactive
   New-AzLocalNetworkGateway -Name ClassicVNetLocal `
   -Location "West US" -AddressPrefix "10.0.0.0/24" `
   -GatewayIpAddress "n.n.n.n" -ResourceGroupName RG1
   ```
3. Solicite que se asigne una dirección IP pública a la puerta de enlace de la red virtual para Azure Resource Manager. No puede especificar la dirección IP que desea usar. La dirección IP se asigna dinámicamente a la puerta de enlace de red virtual. Sin embargo, esto no significa que la dirección IP vaya a cambiar. La única vez que cambia la dirección IP de la puerta de enlace virtual es cuando se elimina y se vuelve a crear la puerta de enlace. No cambiará al modificar el tamaño, restablecer o realizar otro tipo de mantenimiento interno o actualizaciones de la puerta de enlace.

   En este paso, también se establece una variable que se utilizará en un paso posterior.

   ```azurepowershell-interactive
   $ipaddress = New-AzPublicIpAddress -Name gwpip `
   -ResourceGroupName RG1 -Location 'EastUS' `
   -AllocationMethod Dynamic
   ```

4. Compruebe que la red virtual tenga una subred de puerta de enlace. Si no existe ninguna subred de puerta de enlace, agregue una. Asegúrese de que la subred de puerta de enlace tenga el nombre *GatewaySubnet*.
5. Recupere la subred usada para la puerta de enlace mediante la ejecución del comando siguiente. En este paso, también se establece una variable que se utilizará en el paso siguiente.
   
   **-Name** es el nombre de la red virtual de Resource Manager.<br>
   **-ResourceGroupName** es el grupo de recursos con el que está asociado la red virtual. La subred de puerta de enlace debe existir para esta red virtual y su nombre tiene que ser *GatewaySubnet* para que funcione correctamente.<br>

   ```azurepowershell-interactive
   $subnet = Get-AzVirtualNetworkSubnetConfig -Name GatewaySubnet `
   -VirtualNetwork (Get-AzVirtualNetwork -Name RMVNet -ResourceGroupName RG1)
   ``` 

6. Cree la configuración de direccionamiento IP de la puerta de enlace. La configuración de puerta de enlace define la subred y la dirección IP pública. Use el ejemplo siguiente para crear la configuración de la puerta de enlace.

   En este paso, debe pasarse la propiedad de Id de la subred y los objetos de la dirección IP respectivamente a los parámetros **-SubnetId** y **-PublicIpAddressId**. No puede usar una cadena sencilla. Estas variables se establecen en el paso para solicitar una dirección IP pública y el paso para recuperar la subred.

   ```azurepowershell-interactive
   $gwipconfig = New-AzVirtualNetworkGatewayIpConfig `
   -Name gwipconfig -SubnetId $subnet.id `
   -PublicIpAddressId $ipaddress.id
   ```
7. Cree la puerta de enlace de la red virtual de Resource Manager mediante la ejecución del comando siguiente. `-VpnType` debe ser *RouteBased*. La puerta de enlace puede tardar hasta 45 o más minutos en crearse.

   ```azurepowershell-interactive
   New-AzVirtualNetworkGateway -Name RMGateway -ResourceGroupName RG1 `
   -Location "EastUS" -GatewaySKU Standard -GatewayType Vpn `
   -IpConfigurations $gwipconfig `
   -EnableBgp $false -VpnType RouteBased
   ```
8. Copie la dirección IP pública una vez creada la instancia de VPN Gateway. Utilícela cuando configure la red local para la red virtual implementada con el modelo clásico. Puede utilizar el siguiente cmdlet para recuperar la dirección IP pública. La dirección IP pública aparece devuelta como *IpAddress*.

   ```azurepowershell-interactive
   Get-AzPublicIpAddress -Name gwpip -ResourceGroupName RG1
   ```

## <a name="section-3---modify-the-classic-vnet-local-site-settings"></a><a name="localsite"></a>Sección 3: modificación de la configuración del sitio local de la red virtual clásica

En esta sección trabajará con la red virtual clásica. Reemplace la dirección IP de marcador de posición que ha utilizado al especificar la configuración del sitio local que se usará para conectarse a la puerta de enlace de la red virtual de Resource Manager. Puesto que está trabajando con la red virtual clásica, use la instancia de PowerShell que tiene instalada de forma local en el equipo y no la opción Pruébelo de Azure Cloud Shell.

1. Exporte un archivo de configuración de red.

   ```azurepowershell
   Get-AzureVNetConfig -ExportToFile C:\AzureNet\NetworkConfig.xml
   ```
2. Con un editor de texto, modifique el valor de VPNGatewayAddress. Reemplace la dirección IP de marcador de posición por la dirección IP pública de la puerta de enlace de la red virtual de Resource Manager.

   ```
   <VPNGatewayAddress>13.68.210.16</VPNGatewayAddress>
   ```
3. Importe el archivo de configuración de red modificado a Azure.

   ```azurepowershell
   Set-AzureVNetConfig -ConfigurationPath C:\AzureNet\NetworkConfig.xml
   ```

## <a name="section-4---create-a-connection-between-the-gateways"></a><a name="connect"></a>Sección 4: Creación de una conexión entre las puertas de enlace
La creación de una conexión entre las puertas de enlace requiere PowerShell. Debe agregar su cuenta de Azure para usar la versión clásica de los cmdlets de PowerShell. Para ello, use **Add-AzureAccount**.

1. En la consola de PowerShell, establezca la clave compartida. Antes de ejecutar los cmdlets, consulte el archivo de configuración de red que ha descargado para consultar los nombres exactos que Azure espera ver. Al especificar el nombre de una red virtual que contiene espacios, utilice comillas simples alrededor del valor.<br><br>En el ejemplo siguiente, **-VNetName** es el nombre de la red virtual clásica y **-LocalNetworkSiteName** es el nombre especificado para el sitio de la red local. **-SharedKey** es un valor que se puede generar y especificar. En este ejemplo, hemos utilizado 'abc123' pero puede generar y usar algo más complejo. Lo importante es que el valor que especifique aquí debe ser el mismo que el que va a especificar en el paso siguiente, cuando cree la conexión. El valor devuelto para este ejemplo muestra **Estado: Correcto**.

   ```azurepowershell
   Set-AzureVNetGatewayKey -VNetName ClassicVNet `
   -LocalNetworkSiteName RMVNetLocal -SharedKey abc123
   ```
2. Ejecute los comandos siguientes para crear la conexión de VPN:
   
   Establezca las variables.

   ```azurepowershell-interactive
   $vnet01gateway = Get-AzLocalNetworkGateway -Name ClassicVNetLocal -ResourceGroupName RG1
   $vnet02gateway = Get-AzVirtualNetworkGateway -Name RMGateway -ResourceGroupName RG1
   ```
   
   Cree la conexión. Tenga en cuenta que **-ConnectionType** es IPsec y no Vnet2Vnet.

   ```azurepowershell-interactive
   New-AzVirtualNetworkGatewayConnection -Name RM-Classic -ResourceGroupName RG1 `
   -Location "East US" -VirtualNetworkGateway1 `
   $vnet02gateway -LocalNetworkGateway2 `
   $vnet01gateway -ConnectionType IPsec -RoutingWeight 10 -SharedKey 'abc123'
   ```

## <a name="section-5---verify-your-connections"></a><a name="verify"></a>Sección 5: Comprobación de las conexiones

### <a name="to-verify-the-connection-from-your-classic-vnet-to-your-resource-manager-vnet"></a>Comprobación de la conexión de la red virtual clásica a la red virtual de Resource Manager

#### <a name="powershell"></a>PowerShell

[!INCLUDE [vpn-gateway-verify-connection-ps-classic](../../includes/vpn-gateway-verify-connection-ps-classic-include.md)]

#### <a name="azure-portal"></a>Portal de Azure

[!INCLUDE [vpn-gateway-verify-connection-azureportal-classic](../../includes/vpn-gateway-verify-connection-azureportal-classic-include.md)]


### <a name="to-verify-the-connection-from-your-resource-manager-vnet-to-your-classic-vnet"></a>Comprobación de la conexión de la red virtual de Resource Manager a la red virtual clásica

#### <a name="powershell"></a>PowerShell

[!INCLUDE [vpn-gateway-verify-ps-rm](../../includes/vpn-gateway-verify-connection-ps-rm-include.md)]

#### <a name="azure-portal"></a>Portal de Azure

[!INCLUDE [vpn-gateway-verify-connection-portal-rm](../../includes/vpn-gateway-verify-connection-portal-rm-include.md)]

## <a name="vnet-to-vnet-faq"></a><a name="faq"></a>P+F sobre conexiones de red virtual a red virtual

[!INCLUDE [vpn-gateway-vnet-vnet-faq](../../includes/vpn-gateway-faq-vnet-vnet-include.md)]
