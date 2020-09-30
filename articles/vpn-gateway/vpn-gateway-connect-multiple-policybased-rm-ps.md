---
title: 'Azure VPN Gateway: Conexión de puertas de enlace a varios dispositivos VPN locales basados en directivas'
description: Configure una puerta de enlace VPN basada en rutas de Azure para varios dispositivos VPN basados en directivas con Azure Resource Manager y PowerShell.
services: vpn-gateway
author: yushwang
ms.service: vpn-gateway
ms.topic: how-to
ms.date: 09/02/2020
ms.author: yushwang
ms.openlocfilehash: 2a85204fef026940394a19934bef1c631a8e2d21
ms.sourcegitcommit: 9c262672c388440810464bb7f8bcc9a5c48fa326
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/03/2020
ms.locfileid: "89418890"
---
# <a name="connect-azure-vpn-gateways-to-multiple-on-premises-policy-based-vpn-devices-using-powershell"></a>Conexión de puertas de enlace Azure VPN Gateway a varios dispositivos VPN locales basados en directivas con PowerShell

Este artículo le ayuda a configurar una puerta de enlace de VPN basada en rutas de Azure para conectarse a varios dispositivos VPN locales basados en directivas al aprovechar las directivas IPsec/IKE personalizadas en las conexiones VPN de sitio a sitio.

## <a name="about-policy-based-and-route-based-vpn-gateways"></a><a name="about"></a>Acerca de las puertas de enlace de VPN basadas en directivas y en rutas

Los dispositivos VPN basados en directivas se diferencian *frente* a los basados en rutas en la forma en la que se establecen los selectores de tráfico IPsec en una conexión:

* Los dispositivos VPN **basados en directivas** usan las combinaciones de prefijos de ambas redes para definir la forma en la que se cifra/descifra el tráfico a través de túneles IPsec. Normalmente se basa en dispositivos de firewall que realizan el filtrado de paquetes. El cifrado y descifrado de túneles IPsec se agregan al motor de procesamiento y al filtrado de paquetes.
* Los dispositivos VPN **basados en rutas** usan selectores de tráfico universales (caracteres comodín) y permiten a las tablas de reenvío/enrutamiento dirigir el tráfico a distintos túneles IPsec. Normalmente se basa en plataformas de enrutador donde cada túnel IPsec se modela como una interfaz de red o VTI (interfaz de túnel virtual).

Los diagramas siguientes resaltan los dos modelos:

### <a name="policy-based-vpn-example"></a>Ejemplo de VPN basada en directivas
![basada en directivas](./media/vpn-gateway-connect-multiple-policybased-rm-ps/policybasedmultisite.png)

### <a name="route-based-vpn-example"></a>Ejemplo de VPN basada en rutas
![basada en rutas](./media/vpn-gateway-connect-multiple-policybased-rm-ps/routebasedmultisite.png)

### <a name="azure-support-for-policy-based-vpn"></a>Compatibilidad de Azure con VPN basada en directivas
Actualmente, Azure admite los dos modos de puertas de enlace de VPN: puertas de enlace de VPN basadas en directivas y puertas de enlace de VPN basadas en rutas. Se crean en distintas plataformas internas, lo que da lugar a diferentes especificaciones:

| Category | VPN Gateway PolicyBased | VPN Gateway RouteBased | VPN Gateway RouteBased |
| -------- | ----------------------- | ---------------------- | ---------------------- |---                                                 |
| **SKU de puerta de enlace de Azure**    | Básica                       | Básico                            | VpnGw1, VpnGw2, VpnGw3, VpnGw4, VpnGw5  |
| **Versión de IKE**          | IKEv1                       | IKEv2                            | IKEv1 e IKEv2                         |
| **Máx. de conexiones de sitio a sitio** | **1**                       | 10                               | 30                     |
|                          |                             |                                  |                                                    |

Con la directiva IPsec/IKE personalizada, ahora puede configurar puertas de enlace Azure VPN Gateway basadas en rutas para usar selectores de tráfico basados en prefijo con la opción "**PolicyBasedTrafficSelectors**" para conectarse a dispositivos VPN locales basados en directivas. Esta capacidad le permite conectarse desde una red virtual de Azure y una puerta de enlace Azure VPN Gateway a varios dispositivos VPN/firewall locales basados en directivas y quitar el límite de conexión único de las actuales puertas de enlace Azure VPN Gateway basadas en directivas.

> [!IMPORTANT]
> 1. Para habilitar esta conectividad, los dispositivos VPN locales basados en directivas deben admitir **IKEv2** para conectarse a las puertas de enlace Azure VPN Gateway basadas en rutas. Compruebe las especificaciones del dispositivo VPN.
> 2. Las redes locales que se conectan a través de dispositivos VPN basados en directivas con este mecanismo solo pueden conectarse a la red virtual de Azure; **no se permite el tránsito a otras redes locales o redes virtuales a través de la misma puerta de enlace de VPN de Azure**.
> 3. La opción de configuración forma parte de la directiva de conexión IPsec/IKE personalizada. Si habilita la opción de selector de tráfico basado en directivas, debe especificar la directiva completa (vigencias de SA, puntos clave, algoritmos de integridad y cifrado IPsec/IKE).

El diagrama siguiente muestra por qué el enrutamiento del tránsito a través de la puerta de enlace de VPN de Azure no funciona con la opción basada en directivas:

![tránsito basado en directivas](./media/vpn-gateway-connect-multiple-policybased-rm-ps/policybasedtransit.png)

Como se muestra en el diagrama, la puerta de enlace de VPN de Azure tiene selectores de tráfico desde la red virtual a cada prefijo de red local, pero no a los prefijos de conexión cruzada. Por ejemplo, los sitios 2, 3 y 4 locales pueden comunicarse con VNet1, pero no se pueden conectar entre sí a través de la puerta de enlace de VPN de Azure. El diagrama muestra los selectores de tráfico de conexión cruzada que no están disponibles en la puerta de enlace Azure VPN Gateway en esta configuración.

## <a name="workflow"></a><a name="workflow"></a>Flujo de trabajo

Las instrucciones de este artículo siguen el mismo ejemplo de [Configurar una directiva de IPsec o IKE para conexiones VPN de sitio a sitio o de red virtual a red virtual](vpn-gateway-ipsecikepolicy-rm-powershell.md) para establecer una conexión VPN de sitio a sitio. Se muestra en el diagrama siguiente:

![s2s-policy](./media/vpn-gateway-connect-multiple-policybased-rm-ps/s2spolicypb.png)

El flujo de trabajo para habilitar esta conectividad:
1. Cree la red virtual, la puerta de enlace de red privada virtual y la puerta de enlace de red local para la conexión entre locales.
2. Cree una directiva IPsec o IKE.
3. Aplique la directiva cuando cree una conexión de sitio a sitio o de red virtual a red virtual, y **habilite los selectores de tráfico basados en directivas** en la conexión.
4. Si ya se ha creado la conexión, puede aplicar la directiva a una conexión existente o actualizarla.

## <a name="before-you-begin"></a>Antes de empezar

* Compruebe que tiene una suscripción a Azure. Si todavía no la tiene, puede activar sus [ventajas como suscriptor de MSDN](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details) o registrarse para obtener una [cuenta gratuita](https://azure.microsoft.com/pricing/free-trial).

* [!INCLUDE [powershell](../../includes/vpn-gateway-cloud-shell-powershell-about.md)]

## <a name="enable-policy-based-traffic-selectors"></a><a name="enablepolicybased"></a>Habilitación de selectores de tráfico basados en directivas

En esta sección se muestra cómo habilitar los selectores de tráfico basados en directivas en una conexión. Asegúrese de haber completado la [parte 3 del artículo de configuración de una directiva IPsec o IKE](vpn-gateway-ipsecikepolicy-rm-powershell.md). En los pasos de este artículo se usan los mismos parámetros.

### <a name="step-1---create-the-virtual-network-vpn-gateway-and-local-network-gateway"></a>Paso 1: crear la red virtual, la puerta de enlace de VPN y la puerta de enlace de red local

#### <a name="connect-to-your-subscription-and-declare-your-variables"></a>Conexión a la suscripción y declaración de variables

1. Si ejecuta PowerShell localmente en el equipo, inicie sesión con el cmdlet *Connect-AzAccount*. O bien, si lo prefiere, abra Azure Cloud Shell en el explorador.

2. Declare las variables. Para este ejercicio, usamos las siguientes variables:

   ```azurepowershell-interactive
   $Sub1          = "<YourSubscriptionName>"
   $RG1           = "TestPolicyRG1"
   $Location1     = "East US 2"
   $VNetName1     = "TestVNet1"
   $FESubName1    = "FrontEnd"
   $BESubName1    = "Backend"
   $GWSubName1    = "GatewaySubnet"
   $VNetPrefix11  = "10.11.0.0/16"
   $VNetPrefix12  = "10.12.0.0/16"
   $FESubPrefix1  = "10.11.0.0/24"
   $BESubPrefix1  = "10.12.0.0/24"
   $GWSubPrefix1  = "10.12.255.0/27"
   $DNS1          = "8.8.8.8"
   $GWName1       = "VNet1GW"
   $GW1IPName1    = "VNet1GWIP1"
   $GW1IPconf1    = "gw1ipconf1"
   $Connection16  = "VNet1toSite6"
   $LNGName6      = "Site6"
   $LNGPrefix61   = "10.61.0.0/16"
   $LNGPrefix62   = "10.62.0.0/16"
   $LNGIP6        = "131.107.72.22"
   ```

#### <a name="create-the-virtual-network-vpn-gateway-and-local-network-gateway"></a>Creación de la red virtual, la puerta de enlace de VPN y la puerta de enlace de red local

1. Cree un grupo de recursos.

   ```azurepowershell-interactive
   New-AzResourceGroup -Name $RG1 -Location $Location1
   ```
2. Use el ejemplo siguiente para crear la red virtual TestVNet1, con tres subredes y la puerta de enlace de VPN. Si desea reemplazar los valores, es importante que siempre asigne el nombre GatewaySubnet a la subred de la puerta de enlace. Si usa otro, se produce un error al crear la puerta de enlace.

    ```azurepowershell-interactive
    $fesub1 = New-AzVirtualNetworkSubnetConfig -Name $FESubName1 -AddressPrefix $FESubPrefix1
    $besub1 = New-AzVirtualNetworkSubnetConfig -Name $BESubName1 -AddressPrefix $BESubPrefix1
    $gwsub1 = New-AzVirtualNetworkSubnetConfig -Name $GWSubName1 -AddressPrefix $GWSubPrefix1
    
    New-AzVirtualNetwork -Name $VNetName1 -ResourceGroupName $RG1 -Location $Location1 -AddressPrefix $VNetPrefix11,$VNetPrefix12 -Subnet $fesub1,$besub1,$gwsub1
    
    $gw1pip1    = New-AzPublicIpAddress -Name $GW1IPName1 -ResourceGroupName $RG1 -Location $Location1 -AllocationMethod Dynamic
    $vnet1      = Get-AzVirtualNetwork -Name $VNetName1 -ResourceGroupName $RG1
    $subnet1    = Get-AzVirtualNetworkSubnetConfig -Name "GatewaySubnet" -VirtualNetwork $vnet1
    $gw1ipconf1 = New-AzVirtualNetworkGatewayIpConfig -Name $GW1IPconf1 -Subnet $subnet1 -PublicIpAddress $gw1pip1
    
    New-AzVirtualNetworkGateway -Name $GWName1 -ResourceGroupName $RG1 -Location $Location1 -IpConfigurations $gw1ipconf1 -GatewayType Vpn -VpnType RouteBased -GatewaySku HighPerformance
    
    New-AzLocalNetworkGateway -Name $LNGName6 -ResourceGroupName $RG1 -Location $Location1 -GatewayIpAddress $LNGIP6 -AddressPrefix $LNGPrefix61,$LNGPrefix62
    ```

### <a name="step-2---create-an-s2s-vpn-connection-with-an-ipsecike-policy"></a>Paso 2: Creación de una conexión VPN de sitio a sitio con una directiva IPsec o IKE

1. Cree una directiva IPsec o IKE.

   > [!IMPORTANT]
   > Debe crear una directiva IPsec/IKE para habilitar la opción "UsePolicyBasedTrafficSelectors" en la conexión.

   El ejemplo siguiente crea una directiva IPsec/IKE con estos algoritmos y parámetros:
    * IKEv2: AES256, SHA384 y DHGroup24
    * IPsec: AES256, SHA256, sin PFS, 14 400 segundos de vigencia de SA y 102 400 000 KB

   ```azurepowershell-interactive
   $ipsecpolicy6 = New-AzIpsecPolicy -IkeEncryption AES256 -IkeIntegrity SHA384 -DhGroup DHGroup24 -IpsecEncryption AES256 -IpsecIntegrity SHA256 -PfsGroup None -SALifeTimeSeconds 14400 -SADataSizeKilobytes 102400000
   ```
1. Cree la conexión VPN de sitio a sitio con los selectores de tráfico basados en directivas y la directiva IPsec o IKE, y aplique la directiva IPsec o IKE creada en el paso anterior. Tenga en cuenta el parámetro adicional "-UsePolicyBasedTrafficSelectors $True", que habilita los selectores de tráfico basados en directivas en la conexión.

   ```azurepowershell-interactive
   $vnet1gw = Get-AzVirtualNetworkGateway -Name $GWName1  -ResourceGroupName $RG1
   $lng6 = Get-AzLocalNetworkGateway  -Name $LNGName6 -ResourceGroupName $RG1

   New-AzVirtualNetworkGatewayConnection -Name $Connection16 -ResourceGroupName $RG1 -VirtualNetworkGateway1 $vnet1gw -LocalNetworkGateway2 $lng6 -Location $Location1 -ConnectionType IPsec -UsePolicyBasedTrafficSelectors $True -IpsecPolicies $ipsecpolicy6 -SharedKey 'AzureA1b2C3'
   ```
1. Después de completar los pasos, la conexión VPN de sitio a sitio usará la directiva IPsec/IKE definida y habilitará los selectores de tráfico basados en directivas en la conexión. Puede repetir los mismos pasos para agregar más conexiones a los dispositivos VPN locales adicionales basados en directivas desde la misma puerta de enlace Azure VPN Gateway.

## <a name="to-update-policy-based-traffic-selectors"></a><a name="update"></a>Para actualizar los selectores de tráfico basados en directivas
En esta sección se muestra cómo actualizar la opción de selectores de tráfico basados en directivas para una conexión VPN de sitio a sitio existente.

1. Obtenga el recurso de conexión.

   ```azurepowershell-interactive
   $RG1          = "TestPolicyRG1"
   $Connection16 = "VNet1toSite6"
   $connection6  = Get-AzVirtualNetworkGatewayConnection -Name $Connection16 -ResourceGroupName $RG1
   ```
1. Consulte la opción de selectores de tráfico basados en directivas.
La siguiente línea muestra si se usan los selectores de tráfico basados en directivas para la conexión:

   ```azurepowershell-interactive
   $connection6.UsePolicyBasedTrafficSelectors
   ```

   Si la línea devuelve "**True**", los selectores de tráfico basados en directivas están configurados en la conexión; en caso contrario, devuelve "**False**".
1. Una vez que obtenga el recurso de conexión, puede habilitar o deshabilitar los selectores de tráfico basados en directivas en una conexión.

   - Para habilitar

      En el ejemplo siguiente se habilita la opción de selectores de tráfico basados en directivas y se deja sin modificar la directiva IPsec/IKE:

      ```azurepowershell-interactive
      $RG1          = "TestPolicyRG1"
      $Connection16 = "VNet1toSite6"
      $connection6  = Get-AzVirtualNetworkGatewayConnection -Name $Connection16 -ResourceGroupName $RG1

      Set-AzVirtualNetworkGatewayConnection -VirtualNetworkGatewayConnection $connection6 -UsePolicyBasedTrafficSelectors $True
      ```

   - Para deshabilitar

      En el ejemplo siguiente se deshabilita la opción de selectores de tráfico basados en directivas, pero se deja sin modificar la directiva IPsec/IKE:

      ```azurepowershell-interactive
      $RG1          = "TestPolicyRG1"
      $Connection16 = "VNet1toSite6"
      $connection6  = Get-AzVirtualNetworkGatewayConnection -Name $Connection16 -ResourceGroupName $RG1

      Set-AzVirtualNetworkGatewayConnection -VirtualNetworkGatewayConnection $connection6 -UsePolicyBasedTrafficSelectors $False
      ```

## <a name="next-steps"></a>Pasos siguientes
Una vez completada la conexión, puede agregar máquinas virtuales a las redes virtuales. Consulte [Creación de una máquina virtual que ejecuta Windows en el Portal de Azure](../virtual-machines/windows/quick-create-portal.md) para ver los pasos.

Consulte también [Configure IPsec/IKE policy for S2S VPN or VNet-to-VNet connections](vpn-gateway-ipsecikepolicy-rm-powershell.md) (Configuración de la directiva IPsec/IKE para conexiones VPN de sitio a sitio o de red virtual a red virtual) para obtener más información sobre las directivas IPsec/IKE personalizadas.
