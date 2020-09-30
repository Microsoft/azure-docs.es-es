---
title: Ejemplo de configuración para conectar dispositivos Cisco ASA a puertas de enlace de VPN de Azure
description: En este artículo se proporciona un ejemplo de configuración para conectar dispositivos Cisco ASA a puertas de enlace de VPN de Azure.
services: vpn-gateway
author: yushwang
ms.service: vpn-gateway
ms.topic: how-to
ms.date: 09/03/2020
ms.author: yushwang
ms.openlocfilehash: 82e00e0b519040c57fdfd798d8992086c61e5481
ms.sourcegitcommit: ac5cbef0706d9910a76e4c0841fdac3ef8ed2e82
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/03/2020
ms.locfileid: "89426186"
---
# <a name="sample-configuration-cisco-asa-device-ikev2no-bgp"></a>Ejemplo de configuración: dispositivo Cisco ASA (IKEv2/no BGP)
En este artículo se proporciona un ejemplo de configuración para conectar dispositivos Cisco Adaptive Security Appliance (ASA) a puertas de enlace de VPN de Azure. El ejemplo se aplica a dispositivos Cisco ASA que ejecutan IKEv2 sin Border Gateway Protocol (BGP). 

## <a name="device-at-a-glance"></a>Detalles del dispositivo

* Proveedor del dispositivo: **Cisco**
* Modelo del dispositivo: **ASA**           
* Versión de destino: **8.4 y posteriores**
* Modelo probado: **ASA 5505**
* Versión probada: **9.2**             
* Versión de IKE: **IKEv2**                  
* BGP: **No**      
* Tipo de Azure VPN Gateway **VPN Gateway basada en rutas**

> [!NOTE]
> La configuración de ejemplo conecta un dispositivo Cisco ASA a una puerta de enlace de VPN **basada en rutas** de Azure. La conexión usa una directiva de IPsec/IKE personalizada con la opción **UsePolicyBasedTrafficSelectors**, como se describe en [este artículo](vpn-gateway-connect-multiple-policybased-rm-ps.md).
>
> El ejemplo requiere que los dispositivos ASA usen la directiva **IKEv2** con configuraciones basadas en listas de acceso, no basadas en VTI. Consulte las especificaciones del proveedor de dispositivos VPN para comprobar que los dispositivos VPN locales admiten la directiva IKEv2.


## <a name="vpn-device-requirements"></a>Requisitos del dispositivo VPN
Las puertas de enlace de VPN de Azure usan los conjuntos de protocolos IPsec o IKE estándar para establecer túneles VPN de sitio a sitio (S2S). Consulte [Acerca de los dispositivos VPN](vpn-gateway-about-vpn-devices.md) para obtener los parámetros detallados del protocolo IPsec o IKE y los algoritmos criptográficos predeterminados para las puertas de enlace de VPN de Azure.

> [!NOTE]
> También puede especificar una combinación exacta de algoritmos criptográficos y niveles de clave para una conexión específica, como se describe en [Acerca de los requisitos criptográficos](vpn-gateway-about-compliance-crypto.md). Si especifica una combinación específica de algoritmos y niveles de clave, asegúrese de que usa las especificaciones correspondientes en los dispositivos VPN.

## <a name="single-vpn-tunnel"></a>Un solo túnel VPN
Esta configuración consta de un solo túnel VPN S2S entre una puerta de enlace de VPN de Azure y el dispositivo VPN local. Opcionalmente puede configurar BGP a través del túnel VPN.

![Túnel VPN S2S único](./media/vpn-gateway-3rdparty-device-config-cisco-asa/singletunnel.png)

Para obtener instrucciones paso a paso sobre cómo crear las configuraciones de Azure consulte [la configuración de túnel VPN único](vpn-gateway-3rdparty-device-config-overview.md#singletunnel).

### <a name="virtual-network-and-vpn-gateway-information"></a>Información de puerta de enlace de VPN y de red virtual
En esta sección se muestran los parámetros de este ejemplo.

| **Parámetro**                | **Valor**                    |
| ---                          | ---                          |
| Prefijos de direcciones de red virtual        | 10.11.0.0/16<br>10.12.0.0/16 |
| Dirección IP de la puerta de enlace de VPN de Azure         | Azure_Gateway_Public_IP      |
| Prefijos de direcciones locales | 10.51.0.0/16<br>10.52.0.0/16 |
| Dirección IP del dispositivo VPN local    | OnPrem_Device_Public_IP     |
| *Red Virtual BGP ASN                | 65010                        |
| *Dirección IP del par BGP de Azure           | 10.12.255.30                 |
| *ASN de BGP local         | 65050                        |
| *Dirección IP del par BGP local     | 10.52.255.254                |
|                              |                              |

\* Parámetro opcional solo para BGP.

### <a name="ipsecike-policy-and-parameters"></a>Parámetros y directivas de IPsec o IKE
En la tabla siguiente se enumeran los algoritmos y parámetros de IPsec o IKE que se usan en el ejemplo. Consulte las especificaciones del dispositivo VPN para asegurarse de que todos los algoritmos enumerados anteriormente son compatibles con los modelos de dispositivo VPN y las versiones de firmware.

| **IPsec o IKEv2**  | **Valor**                            |
| ---              | ---                                  |
| Cifrado IKEv2 | AES256                               |
| Integridad de IKEv2  | SHA384                               |
| Grupo DH         | DHGroup24                            |
| *Cifrado IPsec | AES256                               |
| *Integridad de IPsec  | SHA1                                 |
| Grupo PFS        | PFS24                                |
| Vigencia de SA QM   | 7\.200 segundos                         |
| Selector de tráfico | UsePolicyBasedTrafficSelectors $True |
| Clave previamente compartida   | PreSharedKey                         |
|                  |                                      |

\*En algunos dispositivos, la integridad de IPsec tiene que ser un valor null cuando el algoritmo de cifrado de IPsec es AES-GCM.

### <a name="asa-device-support"></a>Compatibilidad con dispositivos ASA

* La compatibilidad con IKEv2 requiere la versión 8.4 y versiones posteriores de ASA.

* La compatibilidad con grupos DH y PFS por encima del Grupo 5, requiere la versión ASA 9.x.

* La compatibilidad para cifrado IPsec con AES-GCM e integridad de IPsec con SHA-256, SHA-384 y SHA-512, requiere la versión ASA 9.x. Este requisito de compatibilidad se aplica a los dispositivos ASA más nuevos. En el modelo de la publicación, los modelos 5505, 5510, 5520, 5540, 5550 y 5580 de ASA no admiten estos algoritmos. Consulte las especificaciones del dispositivo VPN para asegurarse de que todos los algoritmos enumerados anteriormente son compatibles con los modelos de dispositivo VPN y las versiones de firmware.


### <a name="sample-device-configuration"></a>Ejemplo de configuración de dispositivo
El script proporciona un ejemplo que se basa en la configuración y los parámetros que se describen en las secciones anteriores. La configuración del túnel VPN de S2S consta de las siguientes partes:

1. Interfaces y rutas
2. Listas de acceso
3. Directiva y parámetros de IKE (fase 1 o modo principal)
4. Directiva y parámetros de IPsec (fase 2 o modo rápido)
5. Otros parámetros, tales como bloqueo MSS de TCP

> [!IMPORTANT]
> Realice los pasos siguientes antes de usar el script de ejemplo. Reemplace los valores de marcador de posición en el script con los ajustes de dispositivo para la configuración.

* Especifique la configuración de interfaz para interfaces internas y externas.
* Identifique las rutas para las redes internas o privadas, y públicas o externas.
* Asegúrese de que todos los nombres y números de directiva son únicos en el dispositivo.
* Asegúrese de que el dispositivo admite algoritmos criptográficos.
* Reemplace los siguiente **valores de marcador de posición** con valores reales para la configuración:
  - Nombre de la interfaz externa: **outside**
  - **Azure_Gateway_Public_IP**
  - **OnPrem_Device_Public_IP**
  - IKE: **Pre_Shared_Key**
  - Nombres de la red virtual y la puerta de enlace de red local: **VNetName** y **LNGName**
  - **Prefijos** de direcciones de red de la red virtual y local
  - **Máscaras de red** correctas

#### <a name="sample-script"></a>Script de ejemplo

```
! Sample ASA configuration for connecting to Azure VPN gateway
!
! Tested hardware: ASA 5505
! Tested version:  ASA version 9.2(4)
!
! Replace the following place holders with your actual values:
!   - Interface names - default are "outside" and "inside"
!   - <Azure_Gateway_Public_IP>
!   - <OnPrem_Device_Public_IP>
!   - <Pre_Shared_Key>
!   - <VNetName>*
!   - <LNGName>* ==> LocalNetworkGateway - the Azure resource that represents the
!     on-premises network, specifies network prefixes, device public IP, BGP info, etc.
!   - <PrivateIPAddress> ==> Replace it with a private IP address if applicable
!   - <Netmask> ==> Replace it with appropriate netmasks
!   - <Nexthop> ==> Replace it with the actual nexthop IP address
!
! (*) Must be unique names in the device configuration
!
! ==> Interface & route configurations
!
!     > <OnPrem_Device_Public_IP> address on the outside interface or vlan
!     > <PrivateIPAddress> on the inside interface or vlan; e.g., 10.51.0.1/24
!     > Route to connect to <Azure_Gateway_Public_IP> address
!
!     > Example:
!
!       interface Ethernet0/0
!        switchport access vlan 2
!       exit
!
!       interface vlan 1
!        nameif inside
!        security-level 100
!        ip address <PrivateIPAddress> <Netmask>
!       exit
!
!       interface vlan 2
!        nameif outside
!        security-level 0
!        ip address <OnPrem_Device_Public_IP> <Netmask>
!       exit
!
!       route outside 0.0.0.0 0.0.0.0 <NextHop IP> 1
!
! ==> Access lists
!
!     > Most firewall devices deny all traffic by default. Create access lists to
!       (1) Allow S2S VPN tunnels between the ASA and the Azure gateway public IP address
!       (2) Construct traffic selectors as part of IPsec policy or proposal
!
access-list outside_access_in extended permit ip host <Azure_Gateway_Public_IP> host <OnPrem_Device_Public_IP>
!
!     > Object group that consists of all VNet prefixes (e.g., 10.11.0.0/16 &
!       10.12.0.0/16)
!
object-group network Azure-<VNetName>
 description Azure virtual network <VNetName> prefixes
 network-object 10.11.0.0 255.255.0.0
 network-object 10.12.0.0 255.255.0.0
exit
!
!     > Object group that corresponding to the <LNGName> prefixes.
!       E.g., 10.51.0.0/16 and 10.52.0.0/16. Note that LNG = "local network gateway".
!       In Azure network resource, a local network gateway defines the on-premises
!       network properties (address prefixes, VPN device IP, BGP ASN, etc.)
!
object-group network <LNGName>
 description On-Premises network <LNGName> prefixes
 network-object 10.51.0.0 255.255.0.0
 network-object 10.52.0.0 255.255.0.0
exit
!
!     > Specify the access-list between the Azure VNet and your on-premises network.
!       This access list defines the IPsec SA traffic selectors.
!
access-list Azure-<VNetName>-acl extended permit ip object-group <LNGName> object-group Azure-<VNetName>
!
!     > No NAT required between the on-premises network and Azure VNet
!
nat (inside,outside) source static <LNGName> <LNGName> destination static Azure-<VNetName> Azure-<VNetName>
!
! ==> IKEv2 configuration
!
!     > General IKEv2 configuration - enable IKEv2 for VPN
!
group-policy DfltGrpPolicy attributes
 vpn-tunnel-protocol ikev1 ikev2
exit
!
crypto isakmp identity address
crypto ikev2 enable outside
!
!     > Define IKEv2 Phase 1/Main Mode policy
!       - Make sure the policy number is not used
!       - integrity and prf must be the same
!       - DH group 14 and above require ASA version 9.x.
!
crypto ikev2 policy 1
 encryption       aes-256
 integrity        sha384
 prf              sha384
 group            24
 lifetime seconds 86400
exit
!
!     > Set connection type and pre-shared key
!
tunnel-group <Azure_Gateway_Public_IP> type ipsec-l2l
tunnel-group <Azure_Gateway_Public_IP> ipsec-attributes
 ikev2 remote-authentication pre-shared-key <Pre_Shared_Key> 
 ikev2 local-authentication  pre-shared-key <Pre_Shared_Key> 
exit
!
! ==> IPsec configuration
!
!     > IKEv2 Phase 2/Quick Mode proposal
!       - AES-GCM and SHA-2 requires ASA version 9.x on newer ASA models. ASA
!         5505, 5510, 5520, 5540, 5550, 5580 are not supported.
!       - ESP integrity must be null if AES-GCM is configured as ESP encryption
!
crypto ipsec ikev2 ipsec-proposal AES-256
 protocol esp encryption aes-256
 protocol esp integrity  sha-1
exit
!
!     > Set access list & traffic selectors, PFS, IPsec proposal, SA lifetime
!       - This sample uses "Azure-<VNetName>-map" as the crypto map name
!       - ASA supports only one crypto map per interface, if you already have
!         an existing crypto map assigned to your outside interface, you must use
!         the same crypto map name, but with a different sequence number for
!         this policy
!       - "match address" policy uses the access-list "Azure-<VNetName>-acl" defined 
!         previously
!       - "ipsec-proposal" uses the proposal "AES-256" defined previously 
!       - PFS groups 14 and beyond requires ASA version 9.x.
!
crypto map Azure-<VNetName>-map 1 match address Azure-<VNetName>-acl
crypto map Azure-<VNetName>-map 1 set pfs group24
crypto map Azure-<VNetName>-map 1 set peer <Azure_Gateway_Public_IP>
crypto map Azure-<VNetName>-map 1 set ikev2 ipsec-proposal AES-256
crypto map Azure-<VNetName>-map 1 set security-association lifetime seconds 7200
crypto map Azure-<VNetName>-map interface outside
!
! ==> Set TCP MSS to 1350
!
sysopt connection tcpmss 1350
!
```

## <a name="simple-debugging-commands"></a>Comandos de depuración simples

Use los siguientes comandos de ASA para fines de depuración:

* Mostrar la asociación de seguridad (SA) de IPsec o IKE:
    ```
    show crypto ipsec sa
    show crypto ikev2 sa
    ```

* Entrar en modo de depuración:
    ```
    debug crypto ikev2 platform <level>
    debug crypto ikev2 protocol <level>
    ```
    Los comandos `debug` pueden generar resultados significativos en la consola.

* Mostrar las configuraciones actuales en el dispositivo:
    ```
    show run
    ```
    Usar subcomandos `show` para enumerar partes de la configuración del dispositivo, por ejemplo:
    ```
    show run crypto
    show run access-list
    show run tunnel-group
    ```

## <a name="next-steps"></a>Pasos siguientes
Para configurar activo-activo entre entornos y las conexiones de red virtual a red virtual, consulte [Configuración de conexiones VPN activo-activo con puertas de enlace VPN](vpn-gateway-activeactive-rm-powershell.md).
