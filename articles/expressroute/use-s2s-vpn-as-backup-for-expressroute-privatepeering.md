---
title: Uso de VPN S2S como copia de seguridad para el emparejamiento privado de Azure ExpressRoute | Microsoft Docs
description: En esta página se proporcionan recomendaciones sobre la arquitectura para el emparejamiento privado de Azure ExpressRoute con VPN S2S.
services: networking
author: duongau
ms.service: expressroute
ms.topic: how-to
ms.date: 02/05/2020
ms.author: duau
ms.openlocfilehash: 0ab74a14c16b7ea1d587cfcc82eea689e2f98c83
ms.sourcegitcommit: 5a3b9f35d47355d026ee39d398c614ca4dae51c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89393027"
---
# <a name="using-s2s-vpn-as-a-backup-for-expressroute-private-peering"></a>Uso de VPN S2S como copia de seguridad para el emparejamiento privado de ExpressRoute

En el artículo titulado [Diseño para la recuperación ante desastres con el emparejamiento privado de ExpressRoute][DR-PP], se describe la necesidad de una solución de conectividad de copia de seguridad para una conectividad de emparejamiento privado de ExpressRoute y cómo usar los circuitos ExpressRoute con redundancia geográfica para ello. En este artículo, vamos a considerar cómo aprovechar y mantener la VPN de sitio a sitio (S2S) como copia de seguridad para el emparejamiento privado de ExpressRoute. 

A diferencia de los circuitos ExpressRoute con redundancia geográfica, solo puede usar la combinación de recuperación ante desastres de ExpressRoute-VPN en modo activo-pasivo. Un desafío importante del uso de cualquier conectividad de red de copia de seguridad en el modo pasivo es que la conexión pasiva a menudo falla junto con la conexión principal. El motivo común de los errores de la conexión pasiva es la falta de mantenimiento activo. Por lo tanto, en este artículo nos centraremos en cómo comprobar y mantener activamente la conectividad VPN S2S que realiza la copia de seguridad de un emparejamiento privado de ExpressRoute.

>[!NOTE] 
>Cuando se anuncia una ruta determinada a través de ExpressRoute y VPN, Azure prefiere el enrutamiento a través de ExpressRoute.  
>

En este artículo, veremos cómo comprobar la conectividad tanto desde la perspectiva de Azure como desde la del perímetro de la red del lado cliente. La capacidad de validar desde cualquier extremo le ayudará, independientemente de si administra o no los dispositivos de red del lado cliente que se encuentran en el mismo nivel que las entidades de red de Microsoft. 

## <a name="example-topology"></a>Topología de ejemplo

En nuestra configuración, tenemos una red local conectada a una red virtual del concentrador de Azure a través de un circuito ExpressRoute y una conexión VPN S2S. La red virtual del concentrador de Azure está, a su vez, en el mismo nivel que una red virtual radial, tal como se muestra en el diagrama siguiente:

![1][1]

En la instalación, el circuito ExpressRoute finaliza en un par de enrutadores de tipo "Perimetral de cliente" (CE) en el entorno local. La LAN local se conecta a los enrutadores de CE a través de un par de firewalls que funcionan en modo líder-seguidor. La VPN S2S finaliza directamente en los firewalls.

En la tabla siguiente se enumeran los prefijos IP clave de la topología:

| **Entidad** | **Prefijo** |
| --- | --- |
| LAN local | 10.1.11.0/25 |
| Red virtual del concentrador de Azure | 10.17.11.0/25 |
| Red virtual radial de Azure | 10.17.11.128/26 |
| Servidor de prueba local | 10.1.11.10 |
| Máquina virtual de prueba de red virtual radial | 10.17.11.132 |
| Subred P2P de conexión principal de ExpressRoute | 192.168.11.16/30 |
| Subred P2P de conexión secundaria de ExpressRoute | 192.168.11.20/30 |
| IP del par BGP principal de VPN Gateway | 10.17.11.76 |
| IP del par BGP secundario de VPN Gateway | 10.17.11.77 |
| IP del par BGP de VPN del firewall local | 192.168.11.88 |
| I/f del enrutador de CE principal hacia la IP del firewall | 192.168.11.0/31 |
| I/f del firewall hacia la IP del enrutador de CE principal | 192.168.11.1/31 |
| I/f del enrutador de CE secundario hacia la IP del firewall | 192.168.11.2/31 |
| I/f del firewall hacia la IP del enrutador de CE secundario | 192.168.11.3/31 |


En la tabla siguiente se enumeran los ASN de la topología:

| **Sistema autónomo** | **ASN** |
| --- | --- |
| Local | 65020 |
| Microsoft Enterprise Edge | 12076 |
| Virtual Network GW (ExR) | 65515 |
| Virtual Network GW (VPN) | 65515 |

## <a name="high-availability-without-asymmetricity"></a>Alta disponibilidad sin asimetría

### <a name="configuring-for-high-availability"></a>Configuración para lograr alta disponibilidad

En [Configuración de conexiones ExpressRoute y de sitio a sitio coexistentes con PowerShell][Conf-CoExist] se explica cómo configurar las conexiones coexistentes del circuito ExpressRoute y VPN S2S. Como se explicó en [Diseño de alta disponibilidad con ExpressRoute][HA], para mejorar la alta disponibilidad de ExpressRoute, nuestra configuración mantiene la redundancia de red (evita un único punto de error) hasta los puntos de conexión. Además, las conexiones principal y secundaria de los circuitos ExpressRoute están configuradas para funcionar en modo activo-activo anunciando los prefijos locales de la misma manera a través de ambas conexiones. 

A continuación, se muestra el anuncio de ruta local del enrutador de CE principal a través de la conexión principal del circuito ExpressRoute (comandos de Juno):

```console
user@SEA-MX03-01> show route advertising-protocol bgp 192.168.11.18 

Cust11.inet.0: 8 destinations, 8 routes (7 active, 0 holddown, 1 hidden)
  Prefix                  Nexthop              MED     Lclpref    AS path
* 10.1.11.0/25            Self                                    I
```

A continuación, se muestra el anuncio de ruta local del enrutador de CE secundario a través de la conexión secundaria del circuito ExpressRoute (comandos de Juno):

```console
user@SEA-MX03-02> show route advertising-protocol bgp 192.168.11.22 

Cust11.inet.0: 8 destinations, 8 routes (7 active, 0 holddown, 1 hidden)
  Prefix                  Nexthop              MED     Lclpref    AS path
* 10.1.11.0/25            Self                                    I
```

Para mejorar la alta disponibilidad de la conexión de copia de seguridad, VPN S2S también se configura en el modo activo-activo. La configuración de Azure VPN Gateway se muestra a continuación. Observe que, como parte de la configuración de VPN, también se enumeran las direcciones IP del par BGP de la puerta de enlace: 10.17.11.76 y 10.17.11.77.

![2][2]

Los firewalls anuncian la ruta local a los pares BGP principal y secundario de VPN Gateway. A continuación, se muestran los anuncios de ruta (Juno):

```console
user@SEA-SRX42-01> show route advertising-protocol bgp 10.17.11.76 

Cust11.inet.0: 14 destinations, 21 routes (14 active, 0 holddown, 0 hidden)
  Prefix                  Nexthop              MED     Lclpref    AS path
* 10.1.11.0/25            Self                                    I

{primary:node0}
user@SEA-SRX42-01> show route advertising-protocol bgp 10.17.11.77    

Cust11.inet.0: 14 destinations, 21 routes (14 active, 0 holddown, 0 hidden)
  Prefix                  Nexthop              MED     Lclpref    AS path
* 10.1.11.0/25            Self                                    I
```

>[!NOTE] 
>La configuración de VPN S2S en modo activo-activo no solo proporciona alta disponibilidad a la conectividad de red de copia de seguridad de recuperación ante desastres, sino que también proporciona un mayor rendimiento a la conectividad de copia de seguridad. Es decir, se recomienda la configuración de VPN S2S en modo activo-activo porque fuerza la creación de varios túneles subyacentes.
>

### <a name="configuring-for-symmetric-traffic-flow"></a>Configuración del flujo de tráfico simétrico

Hemos observado que, cuando una determinada ruta local se anuncia a través de ExpressRoute y VPN S2S, Azure prefiere la ruta de acceso de ExpressRoute. Para forzar que Azure prefiera una ruta de acceso de VPN S2S en lugar de la de ExpressRoute coexistente, debe anunciar rutas más específicas (prefijo más largo con máscara de subred mayor) a través de la conexión VPN. Nuestro objetivo es usar las conexiones VPN como copia de seguridad solamente. Por lo tanto, el comportamiento de la selección de la ruta de acceso predeterminada de Azure está en línea con nuestro objetivo. 

Es nuestra responsabilidad asegurarnos de que el tráfico destinado a Azure desde el entorno local también prefiera la ruta de acceso de ExpressRoute en vez de VPN S2S. La preferencia local predeterminada de los enrutadores de CE y firewalls de la instalación local es 100. Por lo tanto, al configurar la preferencia local de las rutas recibidas a través de los emparejamientos privados de ExpressRoute mayores que 100 (por ejemplo, 150), podemos hacer que el tráfico destinado a Azure prefiera el circuito ExpressRoute en el estado estable.

A continuación, se muestra la configuración de BGP del enrutador de CE principal que finaliza la conexión principal del circuito ExpressRoute. Tenga en cuenta que el valor de la preferencia local de las rutas anunciadas en la sesión de iBGP está configurado para que sea 150. Del mismo modo, debemos asegurarnos de que la preferencia local del enrutador de CE secundario que finaliza la conexión secundaria del circuito ExpressRoute también está configurada para que sea 150.

```console
user@SEA-MX03-01> show configuration routing-instances Cust11
description "Customer 11 VRF";
instance-type virtual-router;
interface xe-0/0/0:0.110;
interface ae0.11;
protocols {
  bgp {
    group ibgp {
        type internal;
        local-preference 150;
        neighbor 192.168.11.1;
    }
    group ebgp {
        peer-as 12076;
        bfd-liveness-detection {
            minimum-interval 300;
            multiplier 3;
        }
        neighbor 192.168.11.18;
    }
  }
}
```

La tabla de enrutamiento de los firewalls locales confirma (se muestra a continuación) que, para el tráfico local destinado a Azure, la ruta de acceso preferida es a través de ExpressRoute en el estado estable.

```console
user@SEA-SRX42-01> show route table Cust11.inet.0 10.17.11.0/24

Cust11.inet.0: 14 destinations, 21 routes (14 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

10.17.11.0/25      *[BGP/170] 2d 00:34:04, localpref 150
                      AS path: 12076 I, validation-state: unverified
                    > to 192.168.11.0 via reth1.11
                      to 192.168.11.2 via reth2.11
                    [BGP/170] 2d 00:34:01, localpref 150
                      AS path: 12076 I, validation-state: unverified
                     > to 192.168.11.2 via reth2.11
                    [BGP/170] 2d 21:12:13, localpref 100, from 10.17.11.76
                       AS path: 65515 I, validation-state: unverified
                    > via st0.118
                    [BGP/170] 2d 00:41:51, localpref 100, from 10.17.11.77
                       AS path: 65515 I, validation-state: unverified
                     > via st0.119
10.17.11.76/32     *[Static/5] 2d 21:12:16
                     > via st0.118
10.17.11.77/32     *[Static/5] 2d 00:41:56
                    > via st0.119
10.17.11.128/26    *[BGP/170] 2d 00:34:04, localpref 150
                       AS path: 12076 I, validation-state: unverified
                     > to 192.168.11.0 via reth1.11
                       to 192.168.11.2 via reth2.11
                    [BGP/170] 2d 00:34:01, localpref 150
                      AS path: 12076 I, validation-state: unverified
                     > to 192.168.11.2 via reth2.11
                    [BGP/170] 2d 21:12:13, localpref 100, from 10.17.11.76
                       AS path: 65515 I, validation-state: unverified
                    > via st0.118
                     [BGP/170] 2d 00:41:51, localpref 100, from 10.17.11.77
                       AS path: 65515 I, validation-state: unverified
                     > via st0.119
```

En la tabla de rutas anterior, para las rutas de red virtual de concentrador y radio (10.17.11.0/25 y 10.17.11.128/26), vemos que se prefiere el circuito ExpressRoute a través de las conexiones VPN. 192.168.11.0 y 192.168.11.2 son direcciones IP de la interfaz de firewall hacia los enrutadores de CE.

## <a name="validation-of-route-exchange-over-s2s-vpn"></a>Validación de intercambio de ruta a través de VPN S2S

Anteriormente en este artículo, comprobamos el anuncio de la ruta local de los firewalls a los pares BGP principal y secundario de VPN Gateway. Además, vamos a confirmar las rutas de Azure recibidas por los firewalls desde los pares BGP principal y secundario de VPN Gateway.

```console
user@SEA-SRX42-01> show route receive-protocol bgp 10.17.11.76 table Cust11.inet.0 

Cust11.inet.0: 14 destinations, 21 routes (14 active, 0 holddown, 0 hidden)
  Prefix                  Nexthop              MED     Lclpref    AS path
  10.17.11.0/25           10.17.11.76                             65515 I
  10.17.11.128/26         10.17.11.76                             65515 I

{primary:node0}
user@SEA-SRX42-01> show route receive-protocol bgp 10.17.11.77 table Cust11.inet.0    

Cust11.inet.0: 14 destinations, 21 routes (14 active, 0 holddown, 0 hidden)
  Prefix                  Nexthop              MED     Lclpref    AS path
  10.17.11.0/25           10.17.11.77                             65515 I
  10.17.11.128/26         10.17.11.77                             65515 I
```

Del mismo modo, vamos a comprobar los prefijos de ruta de red locales recibidos por Azure VPN Gateway. 

```powershell
PS C:\Users\user> Get-AzVirtualNetworkGatewayLearnedRoute -ResourceGroupName SEA-Cust11 -VirtualNetworkGatewayName SEA-Cust11-VNet01-gw-vpn | where {$_.Network -eq "10.1.11.0/25"} | select Network, NextHop, AsPath, Weight

Network      NextHop       AsPath      Weight
-------      -------       ------      ------
10.1.11.0/25 192.168.11.88 65020        32768
10.1.11.0/25 10.17.11.76   65020        32768
10.1.11.0/25 10.17.11.69   12076-65020  32769
10.1.11.0/25 10.17.11.69   12076-65020  32769
10.1.11.0/25 192.168.11.88 65020        32768
10.1.11.0/25 10.17.11.77   65020        32768
10.1.11.0/25 10.17.11.69   12076-65020  32769
10.1.11.0/25 10.17.11.69   12076-65020  32769
```

Como se indicó antes, VPN Gateway tiene rutas que han recibido los pares BGP principal y secundario de VPN Gateway. También tiene visibilidad sobre las rutas recibidas a través de las conexiones ExpressRoute principal y secundaria (las que tienen AS-path con el prefijo 12076). Para confirmar las rutas recibidas a través de conexiones VPN, debemos conocer la IP de las conexiones del par BGP local. En nuestra configuración en cuestión, es 192.168.11.88 y vemos las rutas recibidas desde ella.

A continuación, vamos a comprobar las rutas anunciadas por Azure VPN Gateway en el par BGP del firewall (192.168.11.88).

```powershell
PS C:\Users\user> Get-AzVirtualNetworkGatewayAdvertisedRoute -Peer 192.168.11.88 -ResourceGroupName SEA-Cust11 -VirtualNetworkGatewayName SEA-Cust11-VNet01-gw-vpn |  select Network, NextHop, AsPath, Weight

Network         NextHop     AsPath Weight
-------         -------     ------ ------
10.17.11.0/25   10.17.11.76 65515       0
10.17.11.128/26 10.17.11.76 65515       0
10.17.11.0/25   10.17.11.77 65515       0
10.17.11.128/26 10.17.11.77 65515       0
```

Si no se pueden ver los intercambios de ruta significa que hay un error de conexión. Vea [Solución de problemas: una conexión VPN de sitio a sitio de Azure no puede conectarse y deja de funcionar][VPN Troubleshoot] para obtener ayuda en la solución de problemas con la conexión VPN.

## <a name="testing-failover"></a>Prueba de la conmutación por error

Ahora que hemos confirmado que los intercambios de ruta son correctos a través de la conexión VPN (plano de control), estamos listos para cambiar el tráfico (plano de datos) desde la conectividad de ExpressRoute a la conectividad de VPN. 

>[!NOTE] 
>En entornos de producción, la prueba de conmutación por error debe realizarse durante la ventana de trabajo de mantenimiento de la red programada, ya que puede interrumpir el servicio.
>

Antes de llevar a cabo el cambio de tráfico, vamos a realizar un seguimiento de la ruta de acceso actual en nuestra instalación, desde el servidor de prueba local hasta la máquina virtual de prueba en la red virtual de radio.

```console
C:\Users\PathLabUser>tracert 10.17.11.132

Tracing route to 10.17.11.132 over a maximum of 30 hops

  1    <1 ms    <1 ms    <1 ms  10.1.11.1
  2    <1 ms    <1 ms    11 ms  192.168.11.0
  3    <1 ms    <1 ms    <1 ms  192.168.11.18
  4     *        *        *     Request timed out.
  5     6 ms     6 ms     5 ms  10.17.11.132

Trace complete.
```

Las subredes de conexión punto a punto de ExpressRoute principal y secundaria de la instalación son, respectivamente, 192.168.11.16/30 y 192.168.11.20/30. En la ruta de seguimiento anterior, en el paso 3 vemos que estamos llegando a 192.168.11.18, que es la dirección IP de la interfaz de MSEE principal. La presencia de la interfaz de MSEE confirma que, como se esperaba, la ruta de acceso actual es a través de ExpressRoute.

Como se indica en [Restablecimiento de emparejamientos de circuitos ExpressRoute][RST], vamos a usar los siguientes comandos de PowerShell para deshabilitar el emparejamiento principal y secundario del circuito ExpressRoute.

```powershell
$ckt = Get-AzExpressRouteCircuit -Name "expressroute name" -ResourceGroupName "SEA-Cust11"
$ckt.Peerings[0].State = "Disabled"
Set-AzExpressRouteCircuit -ExpressRouteCircuit $ckt
```

El tiempo de cambio de la conmutación por error depende del tiempo de convergencia de BGP. En nuestra instalación, el cambio de conmutación por error tarda unos segundos (menos de 10). Después del cambio, la repetición de traceroute muestra la siguiente ruta de acceso:

```console
C:\Users\PathLabUser>tracert 10.17.11.132

Tracing route to 10.17.11.132 over a maximum of 30 hops

  1    <1 ms    <1 ms    <1 ms  10.1.11.1
  2     *        *        *     Request timed out.
  3     6 ms     7 ms     9 ms  10.17.11.132

Trace complete.
```

El resultado de traceroute confirma que la conexión de copia de seguridad a través de VPN S2S está activa y puede proporcionar continuidad de servicio si se produce un error en las conexiones de ExpressRoute principal y secundaria. Para completar las pruebas de conmutación por error, vuelva a habilitar las conexiones de ExpressRoute y normalice el flujo de tráfico con el siguiente conjunto de comandos.

```powershell
$ckt = Get-AzExpressRouteCircuit -Name "expressroute name" -ResourceGroupName "SEA-Cust11"
$ckt.Peerings[0].State = "Enabled"
Set-AzExpressRouteCircuit -ExpressRouteCircuit $ckt
```

Para confirmar que el tráfico se cambia de nuevo a ExpressRoute, repita traceroute y asegúrese de que pasa por el emparejamiento privado de ExpressRoute.

## <a name="next-steps"></a>Pasos siguientes

ExpressRoute está diseñado para ofrecer alta disponibilidad sin ningún punto único de error dentro de la red de Microsoft. Aun así, un circuito ExpressRoute se limita a una única región geográfica y a un proveedor de servicios. VPN S2S puede ser una buena solución de copia de seguridad pasiva de recuperación ante desastres para un circuito ExpressRoute. En el caso de una solución de conexión de copia de seguridad pasiva, es importante realizar un mantenimiento regular de la configuración pasiva y una validación periódica de la conexión. Es esencial no dejar que la configuración de VPN quede obsoleta y repetir de forma periódica (por ejemplo, cada trimestre) los pasos de prueba de validación y conmutación por error que se describen en este artículo durante la ventana de mantenimiento.

Para habilitar la supervisión y las alertas basadas en métricas de VPN Gateway, consulte [Configuración de alertas en métricas de VPN Gateway][VPN-alerts].

Para agilizar la convergencia de BGP después de un error de ExpressRoute, consulte [Configuración de BFD a través de ExpressRoute][BFD].

<!--Image References-->
[1]: ./media/use-s2s-vpn-as-backup-for-expressroute-privatepeering/topology.png "topología en cuestión"
[2]: ./media/use-s2s-vpn-as-backup-for-expressroute-privatepeering/vpn-gw-config.png "configuración de VPN GW"

<!--Link References-->
[DR-PP]: https://docs.microsoft.com/azure/expressroute/designing-for-disaster-recovery-with-expressroute-privatepeering
[Conf-CoExist]: https://docs.microsoft.com/azure/expressroute/expressroute-howto-coexist-resource-manager
[HA]: https://docs.microsoft.com/azure/expressroute/designing-for-high-availability-with-expressroute
[VPN Troubleshoot]: https://docs.microsoft.com/azure/vpn-gateway/vpn-gateway-troubleshoot-site-to-site-cannot-connect
[VPN-alerts]: https://docs.microsoft.com/azure/vpn-gateway/vpn-gateway-howto-setup-alerts-virtual-network-gateway-metric
[BFD]: https://docs.microsoft.com/azure/expressroute/expressroute-bfd
[RST]: https://docs.microsoft.com/azure/expressroute/expressroute-howto-reset-peering



