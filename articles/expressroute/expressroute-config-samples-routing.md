---
title: 'Azure ExpressRoute: Ejemplos de configuración de enrutadores'
description: Use estos ejemplos de configuración de enrutamiento e interfaces para enrutadores Cisco serie IOS-XE y Juniper serie MX para trabajar con Azure ExpressRoute.
services: expressroute
author: duongau
ms.service: expressroute
ms.topic: article
ms.date: 03/26/2020
ms.author: duau
ms.openlocfilehash: 3bc850f02884ae0547c2ecf56a46a57a4e66a752
ms.sourcegitcommit: 5a3b9f35d47355d026ee39d398c614ca4dae51c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89397413"
---
# <a name="router-configuration-samples-to-set-up-and-manage-routing"></a>Ejemplos de configuración de enrutadores para configurar y administrar enrutamiento
Esta página ofrece ejemplos de configuración de enrutamiento e interfaces para enrutadores Cisco serie IOS-XE y Juniper serie MX para trabajar con Azure ExpressRoute.

> [!IMPORTANT]
> Los ejemplos de esta página sirven meramente de guía. Para elaborar las configuraciones adecuadas que satisfagan sus necesidades tiene que trabajar con los equipos técnico y de ventas de su proveedor y con su equipo de red. Microsoft no dará soporte técnico para problemas relacionados con las configuraciones que aparecen en esta página. Debe ponerse en contacto con el fabricante del dispositivo si tiene problemas de soporte técnico.
> 
> 

## <a name="mtu-and-tcp-mss-settings-on-router-interfaces"></a>Configuración de MSS de MTU y TCP en las interfaces de enrutador
La unidad de transmisión máxima (MTU) de la interfaz de ExpressRoute es 1500, que es la MTU predeterminada típica de una interfaz Ethernet en un enrutador. A menos que el enrutador tenga una MTU diferente de forma predeterminada, no es necesario especificar un valor en la interfaz del enrutador.

A diferencia de una puerta de enlace de red privada virtual de Azure, no es necesario especificar el tamaño de segmento máximo (MSS) de TCP para un circuito de ExpressRoute.

Los ejemplos de configuración del enrutador de este artículo son aplicables a todos los emparejamientos. Si desea más información, vea [Emparejamientos de ExpressRoute](expressroute-circuit-peerings.md) y [Requisitos de NAT de ExpressRoute](expressroute-routing.md).


## <a name="cisco-ios-xe-based-routers"></a>Enrutadores basados en Cisco IOS-XE
Los ejemplos en esta sección son aplicables a cualquier enrutador que ejecute la familia del sistema operativo IOS-XE.

### <a name="configure-interfaces-and-subinterfaces"></a>Configuración de interfaces y subinterfaces
En cada enrutador que se conecte a Microsoft necesitará una subinterfaz por cada emparejamiento. Una subinterfaz puede identificarse con un identificador de VLAN o un par apilado de identificadores de VLAN y una dirección IP.

**Definición de interfaz Dot1Q**

Este ejemplo ofrece la definición de subinterfaz de una subinterfaz con un solo identificador de VLAN. El identificador de VLAN es único para cada emparejamiento. El último octeto de la dirección IPv4 siempre será un número impar.

```console
interface GigabitEthernet<Interface_Number>.<Number>
 encapsulation dot1Q <VLAN_ID>
 ip address <IPv4_Address><Subnet_Mask>
```

**Definición de interfaz QinQ**

Este ejemplo ofrece la definición de subinterfaz de una subinterfaz con dos identificadores de VLAN. El identificador de VLAN externo (s-tag), si se usa, es el mismo en todos los emparejamientos. El identificador de VLAN interno (c-tag) es único para cada emparejamiento. El último octeto de la dirección IPv4 siempre será un número impar.

```console
interface GigabitEthernet<Interface_Number>.<Number>
 encapsulation dot1Q <s-tag> seconddot1Q <c-tag>
 ip address <IPv4_Address><Subnet_Mask>
```

### <a name="set-up-ebgp-sessions"></a>Configuración de sesiones eBGP
Tiene que configurar una sesión BGP con Microsoft para cada emparejamiento. Configure una sesión de BGP utilizando el ejemplo a continuación. Si la dirección IPv4 que ha usado para la subinterfaz era a.b.c.d, entonces la dirección IP del vecino BGP (Microsoft) será a.b.c.d+1. El último octeto de la dirección IPv4 del vecino BGP siempre será un número par.

```console
router bgp <Customer_ASN>
 bgp log-neighbor-changes
 neighbor <IP#2_used_by_Azure> remote-as 12076
 !
 address-family ipv4
 neighbor <IP#2_used_by_Azure> activate
 exit-address-family
!
```

### <a name="set-up-prefixes-to-be-advertised-over-the-bgp-session"></a>Configuración de prefijos para anunciarse mediante la sesión BGP
Configure el enrutador para anunciar los prefijos seleccionados a Microsoft mediante el ejemplo siguiente.

```console
router bgp <Customer_ASN>
 bgp log-neighbor-changes
 neighbor <IP#2_used_by_Azure> remote-as 12076
 !
 address-family ipv4
  network <Prefix_to_be_advertised> mask <Subnet_mask>
  neighbor <IP#2_used_by_Azure> activate
 exit-address-family
!
```

### <a name="route-maps"></a>Asignaciones de ruta
Puede usar asignaciones de ruta y listas de prefijos para filtrar los prefijos propagados en la red. Consulte el ejemplo siguiente y asegúrese de que tiene configuradas las listas de prefijos adecuadas.

```console
router bgp <Customer_ASN>
 bgp log-neighbor-changes
 neighbor <IP#2_used_by_Azure> remote-as 12076
 !
 address-family ipv4
  network <Prefix_to_be_advertised> mask <Subnet_mask>
  neighbor <IP#2_used_by_Azure> activate
  neighbor <IP#2_used_by_Azure> route-map <MS_Prefixes_Inbound> in
 exit-address-family
!
route-map <MS_Prefixes_Inbound> permit 10
 match ip address prefix-list <MS_Prefixes>
!
```

### <a name="configure-bfd"></a>Configuración de BFD

Tendrá que configurar BFD en dos lugares: uno en el nivel de interfaz y otro en el nivel de BGP. El ejemplo siguiente es para la interfaz de QinQ. 

```console
interface GigabitEthernet<Interface_Number>.<Number>
 bfd interval 300 min_rx 300 multiplier 3
 encapsulation dot1Q <s-tag> seconddot1Q <c-tag>
 ip address <IPv4_Address><Subnet_Mask>

router bgp <Customer_ASN>
 bgp log-neighbor-changes
 neighbor <IP#2_used_by_Azure> remote-as 12076
 !
 address-family ipv4
  neighbor <IP#2_used_by_Azure> activate
  neighbor <IP#2_used_by_Azure> fall-over bfd
 exit-address-family
!
```


## <a name="juniper-mx-series-routers"></a>Enrutadores Juniper serie MX
Los ejemplos en esta sección se aplican a los enrutadores Juniper serie MX.

### <a name="configure-interfaces-and-subinterfaces"></a>Configuración de interfaces y subinterfaces

**Definición de interfaz Dot1Q**

Este ejemplo ofrece la definición de subinterfaz de una subinterfaz con un solo identificador de VLAN. El identificador de VLAN es único para cada emparejamiento. El último octeto de la dirección IPv4 siempre será un número impar.

```console
    interfaces {
        vlan-tagging;
        <Interface_Number> {
            unit <Number> {
                vlan-id <VLAN_ID>;
                family inet {
                    address <IPv4_Address/Subnet_Mask>;
                }
            }
        }
    }
```


**Definición de interfaz QinQ**

Este ejemplo ofrece la definición de subinterfaz de una subinterfaz con dos identificadores de VLAN. El identificador de VLAN externo (s-tag), si se usa, es el mismo en todos los emparejamientos. El identificador de VLAN interno (c-tag) es único para cada emparejamiento. El último octeto de la dirección IPv4 siempre será un número impar.

```console
    interfaces {
        <Interface_Number> {
            flexible-vlan-tagging;
            unit <Number> {
                vlan-tags outer <S-tag> inner <C-tag>;
                family inet {
                    address <IPv4_Address/Subnet_Mask>;
                }                           
            }                               
        }                                   
    }                           
```

### <a name="set-up-ebgp-sessions"></a>Configuración de sesiones eBGP
Tiene que configurar una sesión BGP con Microsoft para cada emparejamiento. Configure una sesión de BGP utilizando el ejemplo a continuación. Si la dirección IPv4 que ha usado para la subinterfaz era a.b.c.d, entonces la dirección IP del vecino BGP (Microsoft) será a.b.c.d+1. El último octeto de la dirección IPv4 del vecino BGP siempre será un número par.

```console
    routing-options {
        autonomous-system <Customer_ASN>;
    }
    }
    protocols {
        bgp { 
            group <Group_Name> { 
                peer-as 12076;              
                neighbor <IP#2_used_by_Azure>;
            }                               
        }                                   
    }
```

### <a name="set-up-prefixes-to-be-advertised-over-the-bgp-session"></a>Configuración de prefijos para anunciarse mediante la sesión BGP
Configure el enrutador para anunciar los prefijos seleccionados a Microsoft mediante el ejemplo siguiente.

```console
    policy-options {
        policy-statement <Policy_Name> {
            term 1 {
                from protocol OSPF;
                route-filter; 
                <Prefix_to_be_advertised/Subnet_Mask> exact;
                then {
                    accept;
                }
            }
        }
    }
    protocols {
        bgp { 
            group <Group_Name> { 
                export <Policy_Name>;
                peer-as 12076;              
                neighbor <IP#2_used_by_Azure>;
            }                               
        }                                   
    }
```

### <a name="route-policies"></a>Directivas de ruta
Puede usar asignaciones de ruta y listas de prefijos para filtrar los prefijos propagados en la red. Consulte el ejemplo siguiente y asegúrese de que tiene configuradas las listas de prefijos adecuadas.

```console
    policy-options {
        prefix-list MS_Prefixes {
            <IP_Prefix_1/Subnet_Mask>;
            <IP_Prefix_2/Subnet_Mask>;
        }
        policy-statement <MS_Prefixes_Inbound> {
            term 1 {
                from {
                    prefix-list MS_Prefixes;
                }
                then {
                    accept;
                }
            }
        }
    }
    protocols {
        bgp { 
            group <Group_Name> { 
                export <Policy_Name>;
                import <MS_Prefixes_Inbound>;
                peer-as 12076;              
                neighbor <IP#2_used_by_Azure>;
            }                               
        }                                   
    }
```

### <a name="configure-bfd"></a>Configuración de BFD
Configure BFD únicamente en la sección del protocolo BGP.

```console
    protocols {
        bgp { 
            group <Group_Name> { 
                peer-as 12076;              
                neighbor <IP#2_used_by_Azure>;
                bfd-liveness-detection {
                       minimum-interval 3000;
                       multiplier 3;
                }
            }                               
        }                                   
    }
```

### <a name="configure-macsec"></a>Configuración de MACSec
Para la configuración de MACSec, la clave de asociación de conectividad (CAK) y el nombre de clave de asociación de conectividad (CKN) deben coincidir con los valores configurados mediante comandos de PowerShell.

```console
    security {
        macsec {
            connectivity-association <Connectivity_Association_Name> {
                cipher-suite gcm-aes-xpn-128;
                security-mode static-cak;
                pre-shared-key {
                    ckn <Connectivity_Association_Key_Name>;
                    cak <Connectivity_Association_Key>; ## SECRET-DATA
                }
            }
            interfaces {
                <Interface_Number> {
                    connectivity-association <Connectivity_Association_Name>;
                }
            }
        }
    }
```

## <a name="next-steps"></a>Pasos siguientes
Consulte [P+F de ExpressRoute](expressroute-faqs.md) para obtener más detalles.



