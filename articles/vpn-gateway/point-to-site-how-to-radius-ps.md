---
title: 'Conexión de punto a sitio de un equipo a una red virtual con autenticación RADIUS: PowerShell | Azure'
description: Conecte clientes Windows y Mac OS X de forma segura a una red virtual con autenticación RADIUS y P2S.
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: how-to
ms.date: 09/02/2020
ms.author: cherylmc
ms.openlocfilehash: e45afed3332d26006cf0b4296986edb6f6588962
ms.sourcegitcommit: 9c262672c388440810464bb7f8bcc9a5c48fa326
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/03/2020
ms.locfileid: "89421737"
---
# <a name="configure-a-point-to-site-connection-to-a-vnet-using-radius-authentication-powershell"></a>Configuración de una conexión de punto a sitio a una red virtual con autenticación RADIUS: PowerShell

Este artículo muestra cómo crear una red virtual con conexión de punto a sitio que usa la autenticación RADIUS. Esta configuración solo está disponible para el modelo de implementación de Resource Manager.

Una puerta de enlace de VPN de punto a sitio (P2S) permite crear una conexión segura a la red virtual desde un equipo cliente individual. Las conexiones VPN de punto a sitio resultan útiles cuando quiere conectarse a la red virtual desde una ubicación remota; por ejemplo, cuando realiza teletrabajo desde casa o desde una conferencia. Una VPN P2S también es una solución útil en lugar de una VPN de sitio a sitio cuando hay solo unos pocos clientes que necesitan conectarse a una red virtual.

Una conexión VPN de punto a sitio se inicia desde dispositivos Windows y Mac. Al conectarse, los clientes pueden usar los siguientes métodos de autenticación: 

* Servidor RADIUS
* Autenticación mediante certificados nativos de puerta de enlace de VPN

Este artículo le ayuda a establecer una configuración de punto a sitio con autenticación mediante el servidor RADIUS. Si desea una autenticación mediante certificados generados y una autenticación mediante certificados nativos de puerta de enlace de VPN en su lugar, consulte el artículo de [configuración de una conexión de punto a sitio a una red virtual con autenticación mediante certificados nativos de puerta de enlace de VPN](vpn-gateway-howto-point-to-site-rm-ps.md).

![Diagrama de conexión: RADIUS](./media/point-to-site-how-to-radius-ps/p2sradius.png)

Las conexiones de punto a sitio no requieren un dispositivo VPN ni una dirección IP de acceso público. P2S crea la conexión VPN sobre SSTP (Protocolo de túnel de sockets seguros), OpenVPN o IKEv2.

* SSTP es un túnel VPN basado en TLS que solo se admite en plataformas de cliente Windows. Puede traspasar firewalls, por lo que resulta ideal para conectarse a Azure desde cualquier lugar. En el lado servidor, se admiten las versiones 1.0, 1.1 y 1.2 de SSTP. El cliente decide qué versión va a usar. Para Windows 8.1 y versiones posteriores, SSTP usa 1.2 de forma predeterminada.

* Protocolo OpenVPN®, un protocolo VPN basado en SSL/TLS. Una solución de VPN basada en TLS puede penetrar firewalls, puesto que la mayoría de ellos abre el puerto TCP 443 saliente, que usa TLS. OpenVPN puede utilizarse para la conexión desde dispositivos Android, iOS (11.0 y versiones posteriores), Windows, Linux y Mac (OSX 10.13 y versiones posteriores).

* La conexión VPN IKEv2, una solución de VPN con protocolo de seguridad de Internet basada en estándares. La conexión VPN IKEv2 puede utilizarse para la conexión desde dispositivos Mac (versión de OSX 10.11 y versiones posteriores).

Las conexiones P2S requieren lo siguiente:

* Una puerta de enlace de VPN RouteBased. 
* Un servidor RADIUS para controlar la autenticación de los usuarios. El servidor RADIUS puede implementarse de forma local o en la red virtual de Azure.
* Un paquete de configuración de cliente VPN para los dispositivos Windows que se conectarán a la red virtual. Un paquete de configuración de cliente VPN proporciona la configuración necesaria para que un cliente de VPN realice una conexión de punto a sitio.

## <a name="about-active-directory-ad-domain-authentication-for-p2s-vpns"></a><a name="aboutad"></a>Acerca de la autenticación de dominio de Active Directory (AD) para VPN de punto a sitio

La autenticación de dominio de AD permite a los usuarios iniciar sesión en Azure con sus credenciales de dominio de la organización. Requiere un servidor RADIUS que se integre con el servidor de AD. Las organizaciones también pueden aprovechar las implementaciones existentes de RADIUS.
 
El servidor RADIUS puede ser local o encontrarse en la red virtual de Azure. Durante la autenticación, la puerta de enlace de VPN actúa como acceso directo y reenvía los mensajes de autenticación entre el servidor RADIUS y el dispositivo que se conecta. Es importante que la puerta de enlace de VPN se pueda comunicar con el servidor RADIUS. Si el servidor RADIUS es local, se necesita una conexión VPN de sitio a sitio de Azure al sitio local.

Además de con Active Directory, los servidores RADIUS también se integran con otros sistemas de identidad externos. Esto abre una gran cantidad de opciones de autenticación para VPN de punto a sitio, incluidas las opciones multifactor. Compruebe la documentación del proveedor del servidor RADIUS para obtener la lista de sistemas de identidad con los que se integra.

![Diagrama de conexión: RADIUS](./media/point-to-site-how-to-radius-ps/radiusimage.png)

> [!IMPORTANT]
>Para un servidor RADIUS local solo se puede usar una conexión VPN de sitio a sitio. No se puede usar una conexión ExpressRoute.
>
>

## <a name="before-beginning"></a><a name="before"></a>Antes de comenzar

Compruebe que tiene una suscripción a Azure. Si todavía no la tiene, puede activar sus [ventajas como suscriptor de MSDN](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details) o registrarse para obtener una [cuenta gratuita](https://azure.microsoft.com/pricing/free-trial).

### <a name="working-with-azure-powershell"></a>Trabajo con Azure PowerShell

[!INCLUDE [powershell](../../includes/vpn-gateway-cloud-shell-powershell-about.md)]

### <a name="example-values"></a><a name="example"></a>Valores del ejemplo

Puede usar los valores del ejemplo para crear un entorno de prueba o hacer referencia a ellos para comprender mejor los ejemplos de este artículo. Puede utilizar los pasos como un tutorial y utilizar los valores sin cambiarlos o modificarlos para reflejar su entorno.

* **Nombre: VNet1**
* **Espacio de direcciones: 192.168.0.0/16** y **10.254.0.0/16**<br>En este ejemplo, se utiliza más de un espacio de direcciones para ilustrar que esta configuración funciona con varios. Sin embargo, para esta configuración no se necesitan varios espacios de direcciones.
* **Nombre de subred: FrontEnd**
  * **Intervalo de direcciones de subred: 192.168.1.0/24**
* **Nombre de subred: BackEnd**
  * **Intervalo de direcciones de subred: 10.254.1.0/24**
* **Nombre de subred: GatewaySubnet**<br>El nombre de subred *GatewaySubnet* es obligatorio para que VPN Gateway funcione.
  * **Intervalo de direcciones de GatewaySubnet: 192.168.200.0/24** 
* **Grupo de direcciones de clientes de VPN: 172.16.201.0/24**<br>Los clientes de VPN que se conectan a la red virtual mediante esta conexión de punto a sitio reciben una dirección IP del grupo de clientes de VPN.
* **Subscription** (Suscripción): si tiene más de una suscripción, compruebe que usa la correcta.
* **Grupo de recursos: TestRG**
* **Ubicación: Este de EE. UU.**
* **Servidor DNS: dirección IP** del servidor DNS que desea usar para la resolución de nombres en la red virtual. (opcional).
* **Nombre de GW: Vnet1GW**
* **Nombre de dirección IP pública: VNet1GWPIP**
* **VpnType: RouteBased**

## <a name="1-set-the-variables"></a><a name="signin"></a>1. Establecimiento de las variables

Declare las variables que desea utilizar. Use el ejemplo siguiente y sustituya los valores por los suyos propios cuando sea necesario. Si cierra la sesión de PowerShell/Cloud Shell en cualquier momento durante el ejercicio, copie y pegue los valores de nuevo para volver a declarar las variables.

  ```azurepowershell-interactive
  $VNetName  = "VNet1"
  $FESubName = "FrontEnd"
  $BESubName = "Backend"
  $GWSubName = "GatewaySubnet"
  $VNetPrefix1 = "192.168.0.0/16"
  $VNetPrefix2 = "10.254.0.0/16"
  $FESubPrefix = "192.168.1.0/24"
  $BESubPrefix = "10.254.1.0/24"
  $GWSubPrefix = "192.168.200.0/26"
  $VPNClientAddressPool = "172.16.201.0/24"
  $RG = "TestRG"
  $Location = "East US"
  $GWName = "VNet1GW"
  $GWIPName = "VNet1GWPIP"
  $GWIPconfName = "gwipconf"
  ```

## <a name="2-create-the-resource-group-vnet-and-public-ip-address"></a>2. <a name="vnet"></a>Creación del grupo de recursos, la red virtual y la dirección IP pública

Los pasos siguientes sirven para crear un grupo de recursos y una red virtual en el grupo de recursos con tres subredes. Al reemplazar valores, es importante que siempre asigne el nombre "GatewaySubnet" a la subred de la puerta de enlace. Si usa otro, se produce un error al crear la puerta de enlace.

1. Cree un grupo de recursos.

   ```azurepowershell-interactive
   New-AzResourceGroup -Name "TestRG" -Location "East US"
   ```
2. Cree las configuraciones de subred para la red virtual con los nombres *FrontEnd*, *BackEnd* y *GatewaySubnet*. Estos prefijos deben formar parte del espacio de direcciones de la red virtual que declaró anteriormente.

   ```azurepowershell-interactive
   $fesub = New-AzVirtualNetworkSubnetConfig -Name "FrontEnd" -AddressPrefix "192.168.1.0/24"  
   $besub = New-AzVirtualNetworkSubnetConfig -Name "Backend" -AddressPrefix "10.254.1.0/24"  
   $gwsub = New-AzVirtualNetworkSubnetConfig -Name "GatewaySubnet" -AddressPrefix "192.168.200.0/24"
   ```
3. Creación de la red virtual.

   En este ejemplo, el parámetro del servidor -DnsServer es opcional. La especificación de un valor no crea un servidor DNS nuevo. La dirección IP del servidor DNS que especifique debe ser un servidor DNS que pueda resolver los nombres de los recursos a los que se conecta desde la red virtual. En este ejemplo, se usa una dirección IP privada, pero es probable que no sea la dirección IP del servidor DNS. Asegúrese de utilizar sus propios valores. El valor especificado lo utilizan los recursos que se implementan en la red virtual, no la conexión de punto a sitio.

   ```azurepowershell-interactive
   New-AzVirtualNetwork -Name "VNet1" -ResourceGroupName "TestRG" -Location "East US" -AddressPrefix "192.168.0.0/16","10.254.0.0/16" -Subnet $fesub, $besub, $gwsub -DnsServer 10.2.1.3
   ```
4. Una puerta de enlace VPN debe tener una dirección IP pública. Primero se solicita el recurso de la dirección IP y, después, se hace referencia a él al crear la puerta de enlace de red virtual. La dirección IP se asigna dinámicamente al recurso cuando se crea la puerta de enlace VPN. Actualmente, VPN Gateway solo admite la asignación de direcciones IP públicas *dinámicas*. No se puede solicitar una asignación de direcciones IP públicas estáticas. Sin embargo, esto no significa que la dirección IP cambia después de que se ha asignado a una puerta de enlace VPN. La única vez que la dirección IP pública cambia es cuando la puerta de enlace se elimina y se vuelve a crear. No cambia cuando se cambia el tamaño, se restablece o se realizan actualizaciones u otras operaciones de mantenimiento interno de una puerta de enlace VPN.

   Especifique las variables para solicitar una dirección IP pública asignada de forma dinámica.

   ```azurepowershell-interactive
   $vnet = Get-AzVirtualNetwork -Name "VNet1" -ResourceGroupName "TestRG"  
   $subnet = Get-AzVirtualNetworkSubnetConfig -Name "GatewaySubnet" -VirtualNetwork $vnet 
   $pip = New-AzPublicIpAddress -Name "VNet1GWPIP" -ResourceGroupName "TestRG" -Location "East US" -AllocationMethod Dynamic 
   $ipconf = New-AzVirtualNetworkGatewayIpConfig -Name "gwipconf" -Subnet $subnet -PublicIpAddress $pip
   ```

## <a name="3-set-up-your-radius-server"></a>3. <a name="radius"></a>Configuración del servidor RADIUS

Antes de crear y configurar la puerta de enlace de red virtual, el servidor RADIUS debe configurarse correctamente para la autenticación.

1. Si no tiene un servidor RADIUS implementado, implemente uno. Para los pasos de implementación, consulte la guía de instalación que proporciona el proveedor de RADIUS.  
2. Configure la puerta de enlace de VPN como cliente RADIUS en RADIUS. Al agregar a este cliente RADIUS, especifique la red virtual GatewaySubnet que ha creado. 
3. Una vez configurado el servidor RADIUS, obtenga su dirección IP y el secreto compartido que deben usar los clientes RADIUS para comunicarse con él. Si el servidor RADIUS está en la red virtual de Azure, use la dirección IP de entidad de certificación de la máquina virtual del servidor RADIUS.

El artículo [Servidor de directivas de redes (NPS)](https://docs.microsoft.com/windows-server/networking/technologies/nps/nps-top) proporciona instrucciones acerca de cómo configurar un servidor Windows RADIUS (NPS) para la autenticación de dominio de AD.

## <a name="4-create-the-vpn-gateway"></a>4. <a name="creategw"></a>Creación de la puerta de enlace de VPN

Configure y cree la puerta de enlace de VPN para la red virtual.

* -GatewayType debe ser "Vpn" y -VpnType, "RouteBased".
* Una puerta de enlace de VPN puede tardar hasta 45 minutos en completarse, según la  [SKU de puerta de enlace](vpn-gateway-about-vpn-gateway-settings.md#gwsku)  que seleccione.

```azurepowershell-interactive
New-AzVirtualNetworkGateway -Name $GWName -ResourceGroupName $RG `
-Location $Location -IpConfigurations $ipconf -GatewayType Vpn `
-VpnType RouteBased -EnableBgp $false -GatewaySku VpnGw1
```

## <a name="5-add-the-radius-server-and-client-address-pool"></a>5. <a name="addradius"></a>Incorporación del servidor RADIUS y el grupo de direcciones de cliente
 
* El valor de -RadiusServer se puede especificar por nombre o por dirección IP. Si se especifica el nombre y el servidor es local, quizá la puerta de enlace de VPN no pueda resolver el nombre. Si es el caso, lo mejor es especificar la dirección IP del servidor. 
* El valor de -RadiusSecret debe coincidir con la configuración del servidor RADIUS.
* -VpnClientAddressPool es el intervalo del que reciben una dirección IP los clientes al conectarse a la VPN. Use un intervalo de direcciones IP privadas que no se superponga a la ubicación local desde la que se va a conectar ni a la red virtual a la que desea conectarse. Asegúrese de que tiene configurado un grupo de direcciones lo suficientemente grande.  

1. Cree una cadena segura para el secreto de RADIUS.

   ```azurepowershell-interactive
   $Secure_Secret=Read-Host -AsSecureString -Prompt "RadiusSecret"
   ```

2. Deberá escribir el secreto de RADIUS. Los caracteres que especifique no se mostrarán, sino que se reemplazarán por el carácter "*".

   ```azurepowershell-interactive
   RadiusSecret:***
   ```
3. Agregue el grupo de direcciones de cliente VPN y la información del servidor RADIUS.

   Para configuraciones SSTP:

    ```azurepowershell-interactive
    $Gateway = Get-AzVirtualNetworkGateway -ResourceGroupName $RG -Name $GWName
    Set-AzVirtualNetworkGateway -VirtualNetworkGateway $Gateway `
    -VpnClientAddressPool "172.16.201.0/24" -VpnClientProtocol "SSTP" `
    -RadiusServerAddress "10.51.0.15" -RadiusServerSecret $Secure_Secret
    ```

   Para configuraciones OpenVPN®:

    ```azurepowershell-interactive
    $Gateway = Get-AzVirtualNetworkGateway -ResourceGroupName $RG -Name $GWName
    Set-AzVirtualNetworkGateway -VirtualNetworkGateway $Gateway -VpnClientRootCertificates @()
    Set-AzVirtualNetworkGateway -VirtualNetworkGateway $Gateway `
    -VpnClientAddressPool "172.16.201.0/24" -VpnClientProtocol "OpenVPN" `
    -RadiusServerAddress "10.51.0.15" -RadiusServerSecret $Secure_Secret
    ```


   Para configuraciones IKEv2:

    ```azurepowershell-interactive
    $Gateway = Get-AzVirtualNetworkGateway -ResourceGroupName $RG -Name $GWName
    Set-AzVirtualNetworkGateway -VirtualNetworkGateway $Gateway `
    -VpnClientAddressPool "172.16.201.0/24" -VpnClientProtocol "IKEv2" `
    -RadiusServerAddress "10.51.0.15" -RadiusServerSecret $Secure_Secret
    ```

   Para SSTP + IKEv2

    ```azurepowershell-interactive
    $Gateway = Get-AzVirtualNetworkGateway -ResourceGroupName $RG -Name $GWName
    Set-AzVirtualNetworkGateway -VirtualNetworkGateway $Gateway `
    -VpnClientAddressPool "172.16.201.0/24" -VpnClientProtocol @( "SSTP", "IkeV2" ) `
    -RadiusServerAddress "10.51.0.15" -RadiusServerSecret $Secure_Secret
    ```

## <a name="6-download-the-vpn-client-configuration-package-and-set-up-the-vpn-client"></a>6. <a name="vpnclient"></a>Descarga del paquete de configuración de cliente VPN y configuración del cliente VPN

La configuración del cliente VPN permite que los dispositivos se conecten a una red virtual mediante una conexión de punto a sitio. Para generar un paquete de configuración de cliente VPN y configurar el cliente de VPN, consulte [Create a VPN Client Configuration for RADIUS authentication](point-to-site-vpn-client-configuration-radius.md) (Creación de la configuración de cliente de VPN para la autenticación RADIUS).

## <a name="7-connect-to-azure"></a><a name="connect"></a>7. Conexión con Azure

### <a name="to-connect-from-a-windows-vpn-client"></a>Para conectarse desde un cliente VPN en Windows

1. Para conectarse a su red virtual, en el equipo cliente, vaya a las conexiones VPN y ubique la que creó. Tiene el mismo nombre que su red virtual. Escriba sus credenciales de dominio y haga clic en "Conectar". Aparece un mensaje emergente que solicita derechos elevados. Haga clic en Aceptar y escriba las credenciales.

   ![El cliente VPN se conecta a Azure](./media/point-to-site-how-to-radius-ps/client.png)
2. Se ha establecido la conexión.

   ![Conexión establecida](./media/point-to-site-how-to-radius-ps/connected.png)

### <a name="connect-from-a-mac-vpn-client"></a>Conexión desde un cliente VPN en Mac

En el cuadro de diálogo Red, localice el perfil de cliente que desea utilizar y, después, haga clic en **Conectar**.

  ![Conexión de Mac](./media/vpn-gateway-howto-point-to-site-rm-ps/applyconnect.png)

## <a name="to-verify-your-connection"></a><a name="verify"></a>Para comprobar la conexión

1. Para comprobar que la conexión VPN está activa, abra un símbolo del sistema con privilegios elevados y ejecute *ipconfig/all*.
2. Vea los resultados. Observe que la dirección IP que recibió es una de las direcciones dentro del grupo de direcciones de cliente de VPN punto a sitio que especificó en la configuración. Los resultados son similares a los del ejemplo siguiente:

   ```
   PPP adapter VNet1:
      Connection-specific DNS Suffix .:
      Description.....................: VNet1
      Physical Address................:
      DHCP Enabled....................: No
      Autoconfiguration Enabled.......: Yes
      IPv4 Address....................: 172.16.201.3(Preferred)
      Subnet Mask.....................: 255.255.255.255
      Default Gateway.................:
      NetBIOS over Tcpip..............: Enabled
   ```

Para solucionar problemas de conexión de P2S, vea [Solución de problemas: conexión de punto a sitio de Azure](vpn-gateway-troubleshoot-vpn-point-to-site-connection-problems.md).

## <a name="to-connect-to-a-virtual-machine"></a><a name="connectVM"></a>Conexión a una máquina virtual

[!INCLUDE [Connect to a VM](../../includes/vpn-gateway-connect-vm-p2s-include.md)]

## <a name="faq"></a><a name="faq"></a>P+F

Estas preguntas más frecuentes se aplican a la conexión de punto a sitio con autenticación RADIUS

[!INCLUDE [Point-to-Site RADIUS FAQ](../../includes/vpn-gateway-faq-p2s-radius-include.md)]

## <a name="next-steps"></a>Pasos siguientes

Una vez completada la conexión, puede agregar máquinas virtuales a las redes virtuales. Consulte [Virtual Machines](https://docs.microsoft.com/azure/) para más información. Para más información acerca de las redes y las máquinas virtuales, consulte [Información general sobre las redes de máquina virtual con Linux y Azure](../virtual-machines/linux/azure-vm-network-overview.md).
