---
title: 'Azure ExpressRoute: Ejemplos de configuración de enrutadores: NAT'
description: Esta página ofrece ejemplos de configuración de enrutamiento para enrutadores Cisco y Juniper.
services: expressroute
author: duongau
ms.service: expressroute
ms.topic: article
ms.date: 12/06/2018
ms.author: duau
ms.openlocfilehash: 4b57dec32a3a008cadf68c19ea52353a23fb2cd3
ms.sourcegitcommit: 5a3b9f35d47355d026ee39d398c614ca4dae51c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89397501"
---
# <a name="router-configuration-samples-to-set-up-and-manage-nat"></a>Ejemplos de configuración de enrutadores para configurar y administrar NAT

En esta página se proporcionan ejemplos de configuración de NAT para enrutadores Cisco serie ASA y Juniper serie SRX al trabajar con ExpressRoute. Solo pretenden ser ejemplos de carácter informativo y no se deben usar tal cual. Puede trabajar con el proveedor para elaborar las configuraciones adecuadas para la red.

> [!IMPORTANT]
> Los ejemplos de esta página pretenden tener un carácter meramente informativo. Debe trabajar con los equipos técnico y de ventas del proveedor y su equipo de red para elaborar las configuraciones adecuadas que satisfagan sus necesidades. Microsoft no dará soporte técnico en problemas relacionados con las configuraciones que aparecen en esta página. Debe ponerse en contacto con el fabricante del dispositivo para problemas de soporte técnico.
> 
> 

* Los ejemplos de configuración de enrutadores siguientes se aplican a emparejamientos públicos de Azure y Microsoft. No debe configurar NAT para emparejamiento privado de Azure. Si desea más información, consulte [Emparejamientos de ExpressRoute](expressroute-circuit-peerings.md) y [Requisitos de NAT de ExpressRoute](expressroute-nat.md).

* Tiene que usar grupos de direcciones IP de NAT independientes para la conectividad a internet y ExpressRoute. Si usa el mismo grupo de direcciones IP de NAT a través de Internet y ExpressRoute se traducirá en enrutamiento asimétrico y pérdida de conectividad.


## <a name="cisco-asa-firewalls"></a>Firewalls de Cisco ASA
### <a name="pat-configuration-for-traffic-from-customer-network-to-microsoft"></a>Configuración de PAT para el tráfico de red de cliente a Microsoft

```console
object network MSFT-PAT
  range <SNAT-START-IP> <SNAT-END-IP>


object-group network MSFT-Range
  network-object <IP> <Subnet_Mask>

object-group network on-prem-range-1
  network-object <IP> <Subnet-Mask>

object-group network on-prem-range-2
  network-object <IP> <Subnet-Mask>

object-group network on-prem
  network-object object on-prem-range-1
  network-object object on-prem-range-2

nat (outside,inside) source dynamic on-prem pat-pool MSFT-PAT destination static MSFT-Range MSFT-Range
```

### <a name="pat-configuration-for-traffic-from-microsoft-to-customer-network"></a>Configuración de PAT para el tráfico de Microsoft a la red de cliente

**Interfaces y dirección:**

Interfaz de origen (donde el tráfico entra en el ASA): dentro Interfaz de destino (donde el tráfico sale del ASA): fuera

**Configuración:**

Grupo NAT:

```console
object network outbound-PAT
    host <NAT-IP>
```

Servidor de destino:

```console
object network Customer-Network
    network-object <IP> <Subnet-Mask>
```

Grupo de objetos para direcciones IP de cliente:

```console
object-group network MSFT-Network-1
    network-object <MSFT-IP> <Subnet-Mask>

object-group network MSFT-PAT-Networks
    network-object object MSFT-Network-1
```

Comandos NAT:

```console
nat (inside,outside) source dynamic MSFT-PAT-Networks pat-pool outbound-PAT destination static Customer-Network Customer-Network
```


## <a name="juniper-srx-series-routers"></a>Enrutadores Juniper serie SRX
### <a name="1-create-redundant-ethernet-interfaces-for-the-cluster"></a>1. Creación de interfaces Ethernet redundantes para el clúster

```console
    interfaces {
        reth0 {
            description "To Internal Network";
            vlan-tagging;
            redundant-ether-options {
                redundancy-group 1;
            }
            unit 100 {
                vlan-id 100;
                family inet {
                    address <IP-Address/Subnet-mask>;
                }
            }
        }
        reth1 {
            description "To Microsoft via Edge Router";
            vlan-tagging;
            redundant-ether-options {
                redundancy-group 2;
            }
            unit 100 {
                description "To Microsoft via Edge Router";
                vlan-id 100;
                family inet {
                    address <IP-Address/Subnet-mask>;
                }
            }
        }
    }
```

### <a name="2-create-two-security-zones"></a>2. Creación de dos zonas de seguridad
* Zona de confianza para la red interna y Zona de no confianza para la red externa que soporta enrutadores de borde
* Asignación de interfaces adecuadas a las zonas
* Autorización de servicios en las interfaces

```console
    security {
        zones {
            security-zone Trust {
                host-inbound-traffic {
                    system-services {
                        ping;
                    }
                    protocols {
                        bgp;
                    }
                }
                interfaces {
                    reth0.100;
                }
            }
            security-zone Untrust {
                host-inbound-traffic {
                    system-services {
                        ping;
                    }
                    protocols {
                        bgp;
                    }
                }
                interfaces {
                    reth1.100;
                }
            }
        }
    }
```


### <a name="3-create-security-policies-between-zones"></a>3. Creación de directivas de seguridad entre zonas

```console
    security {
        policies {
            from-zone Trust to-zone Untrust {
                policy allow-any {
                    match {
                        source-address any;
                        destination-address any;
                        application any;
                    }
                    then {
                        permit;
                    }
                }
            }
            from-zone Untrust to-zone Trust {
                policy allow-any {
                    match {
                        source-address any;
                        destination-address any;
                        application any;
                    }
                    then {
                        permit;
                    }
                }
            }
        }
    }
```

### <a name="4-configure-nat-policies"></a>4. Configuración de directivas de NAT
* Creación de dos grupos de NAT. Uno se utilizará para el tráfico saliente de NAT a Microsoft y otro de Microsoft al cliente.
* Creación de reglas para NAT del tráfico correspondiente

```console
       security {
           nat {
               source {
                   pool SNAT-To-ExpressRoute {
                       routing-instance {
                           External-ExpressRoute;
                       }
                       address {
                           <NAT-IP-address/Subnet-mask>;
                       }
                   }
                   pool SNAT-From-ExpressRoute {
                       routing-instance {
                           Internal;
                       }
                       address {
                           <NAT-IP-address/Subnet-mask>;
                       }
                   }
                   rule-set Outbound_NAT {
                       from routing-instance Internal;
                       to routing-instance External-ExpressRoute;
                       rule SNAT-Out {
                           match {
                               source-address 0.0.0.0/0;
                           }
                           then {
                               source-nat {
                                   pool {
                                       SNAT-To-ExpressRoute;
                                   }
                               }
                           }
                       }
                   }
                   rule-set Inbound-NAT {
                       from routing-instance External-ExpressRoute;
                       to routing-instance Internal;
                       rule SNAT-In {
                           match {
                               source-address 0.0.0.0/0;
                           }
                           then {
                               source-nat {
                                   pool {
                                       SNAT-From-ExpressRoute;
                                   }
                               }
                           }
                       }
                   }
               }
           }
       }
```

### <a name="5-configure-bgp-to-advertise-selective-prefixes-in-each-direction"></a>5. Configuración de BGP para anunciar prefijos selectivos en cada dirección
Consulte ejemplos en la página [Ejemplos de configuración de enrutamiento ](expressroute-config-samples-routing.md).

### <a name="6-create-policies"></a>6. Creación de directivas

```console
    routing-options {
                  autonomous-system <Customer-ASN>;
    }
    policy-options {
        prefix-list Microsoft-Prefixes {
            <IP-Address/Subnet-Mask;
            <IP-Address/Subnet-Mask;
        }
        prefix-list private-ranges {
            10.0.0.0/8;
            172.16.0.0/12;
            192.168.0.0/16;
            100.64.0.0/10;
        }
        policy-statement Advertise-NAT-Pools {
            from {
                protocol static;
                route-filter <NAT-Pool-Address/Subnet-mask> prefix-length-range /32-/32;
            }
            then accept;
        }
        policy-statement Accept-from-Microsoft {
            term 1 {
                from {
                    instance External-ExpressRoute;
                    prefix-list-filter Microsoft-Prefixes orlonger;
                }
                then accept;
            }
            term deny {
                then reject;
            }
        }
        policy-statement Accept-from-Internal {
            term no-private {
                from {
                    instance Internal;
                    prefix-list-filter private-ranges orlonger;
                }
                then reject;
            }
            term bgp {
                from {
                    instance Internal;
                    protocol bgp;
                }
                then accept;
            }
            term deny {
                then reject;
            }
        }
    }
    routing-instances {
        Internal {
            instance-type virtual-router;
            interface reth0.100;
            routing-options {
                static {
                    route <NAT-Pool-IP-Address/Subnet-mask> discard;
                }
                instance-import Accept-from-Microsoft;
            }
            protocols {
                bgp {
                    group customer {
                        export <Advertise-NAT-Pools>;
                        peer-as <Customer-ASN-1>;
                        neighbor <BGP-Neighbor-IP-Address>;
                    }
                }
            }
        }
        External-ExpressRoute {
            instance-type virtual-router;
            interface reth1.100;
            routing-options {
                static {
                    route <NAT-Pool-IP-Address/Subnet-mask> discard;
                }
                instance-import Accept-from-Internal;
            }
            protocols {
                bgp {
                    group edge-router {
                        export <Advertise-NAT-Pools>;
                        peer-as <Customer-Public-ASN>;
                        neighbor <BGP-Neighbor-IP-Address>;
                    }
                }
            }
        }
    }
```

## <a name="next-steps"></a>Pasos siguientes
Consulte [P+F de ExpressRoute](expressroute-faqs.md) para obtener más detalles.

