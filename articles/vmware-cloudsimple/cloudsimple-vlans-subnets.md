---
title: VLAN y subredes en VMware Solution by CloudSimple
description: Obtenga información sobre las VLAN y las subredes en Nube privada de CloudSimple y sobre la red que CloudSimple proporciona para implementar el servicio CloudSimple.
author: sharaths-cs
ms.author: dikamath
ms.date: 08/15/2019
ms.topic: article
ms.service: azure-vmware-cloudsimple
ms.reviewer: cynthn
manager: dikamath
ms.openlocfilehash: e3235b394f739e8af846f30dfe51705d0926c843
ms.sourcegitcommit: a2a7746c858eec0f7e93b50a1758a6278504977e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/12/2020
ms.locfileid: "88140673"
---
# <a name="vlans-and-subnets-overview"></a>Introducción a las VLAN y subredes

CloudSimple proporciona una red por región donde se implementa el servicio CloudSimple.  La red es un único espacio de direcciones TCP de nivel 3 con el enrutamiento habilitado de forma predeterminada.  Todas las nubes privadas y las subredes creadas en esta región se pueden comunicar entre sí sin ninguna configuración adicional.  En vCenter, puede crear grupos de puertos distribuidos mediante VLAN.

![Topología de red de CloudSimple](media/cloudsimple-network-topology.png)

## <a name="vlans"></a>VLAN

Se crea una VLAN (red de nivel 2) para cada nube privada.  El tráfico de la red de nivel 2 permanece dentro del límite de una nube privada, lo que le permite aislar el tráfico local dentro de la nube privada.  Una VLAN creada en la nube privada puede usarse para crear grupos de puertos distribuidos solo en esa nube privada.  Una VLAN creada en una nube privada se configura automáticamente en todos los conmutadores conectados a los hosts de una nube privada.

## <a name="subnets"></a>Subredes

Puede crear una subred al crear una VLAN, definiendo el espacio de direcciones de la subred. Se asigna una dirección IP del espacio de direcciones como puerta de enlace de la subred. Se asigna un único espacio de direcciones privadas de nivel 3 por región y cliente. Puede configurar cualquier espacio de direcciones no superpuesto de RFC 1918 con su red local o red virtual de Azure en su región de red.

Todas las subredes pueden comunicarse entre sí de forma predeterminada, lo que reduce la sobrecarga de configuración para el enrutamiento entre nubes privadas. El tráfico de datos este-oeste entre equipos de la misma región permanece en la misma red de nivel 3 y se transfiere a través de la infraestructura de red local dentro de la región. No se requiere ninguna salida para la comunicación entre las nubes privadas de una región. Este enfoque elimina cualquier pérdida de rendimiento de WAN/salida en la implementación de diferentes cargas de trabajo en distintas nubes privadas.

## <a name="vspherevsan-subnets-cidr-range"></a>Intervalo de CIDR de subredes vSphere/vSAN

Una nube privada se crea como un entorno de pila VMware aislada (hosts ESXi, vCenter, vSAN y NSX) administrado por un servidor vCenter.  Los componentes de administración se implementan en la red seleccionada para CIDR de subredes vSphere/vSAN.  El intervalo de CIDR de red se divide en subredes diferentes durante la implementación.

* Prefijo de intervalo de CIDR de subredes vSphere/vSAN mínimo: **/24**
* Prefijo de intervalo de CIDR de subredes vSphere/vSAN máximo: **/21**

> [!CAUTION]
> Las direcciones IP en el intervalo CIDR de vSphere/vSAN están reservadas para que las use la infraestructura de nube privada.  No use la dirección IP de este intervalo en ninguna máquina virtual.

### <a name="vspherevsan-subnets-cidr-range-limits"></a>Límites del intervalo de CIDR de subredes vSphere/vSAN

La selección del tamaño del intervalo de CIDR de subredes vSphere/vSAN afecta al tamaño de la nube privada.  En la tabla siguiente se muestra el número máximo de nodos que puede tener en función del tamaño del intervalo de CIDR de subredes vSphere/vSAN.

| Longitud del prefijo de CIDR de subredes vSphere/vSAN especificado | Número máximo de nodos |
|---------------------------------------------------|-------------------------|
| /24 | 26 |
| /23 | 58 |
| /22 | 118 |
| /21 | 220 |

### <a name="management-subnets-created-on-a-private-cloud"></a>Subredes de administración creadas en una nube privada

Las siguientes subredes de administración se generan al crear una nube privada.

* **Administración del sistema**. VLAN y subred para la red de administración de hosts ESXi, servidor DNS y servidor vCenter.
* **VMotion**. VLAN y subred para la red vMotion de hosts ESXi.
* **VSAN**. VLAN y subred para la red vSAN de hosts ESXi.
* **NsxtEdgeUplink1**. VLAN y subred para vínculos superiores de VLAN a una red externa.
* **NsxtEdgeUplink2**. VLAN y subred para vínculos superiores de VLAN a una red externa.
* **NsxtEdgeTransport**. VLAN y subred para el control de zonas de transporte al alcance de redes de nivel 2 en NSX-T.
* **NsxtHostTransport**. VLAN y subred para la zona de transporte de host.

### <a name="management-network-cidr-range-breakdown"></a>Desglose del intervalo de CIDR de la red de administración

El intervalo de CIDR de subredes vSphere/vSAN especificado se divide en varias subredes.  En la tabla siguiente se muestra un ejemplo del desglose de los prefijos permitidos.  En el ejemplo se utiliza 192.168.0.0 como intervalo de CIDR.

Ejemplo:

| Prefijo/CIDR de subredes vSphere/vSAN especificado | 192.168.0.0/21 | 192.168.0.0/22 | 192.168.0.0/23 | 192.168.0.0/24 |
|---------------------------------|----------------|----------------|----------------|----------------|
| Administración del sistema | 192.168.0.0/24 | 192.168.0.0/24 | 192.168.0.0/25 | 192.168.0.0/26 |
| vMotion | 192.168.1.0/24 | 192.168.1.0/25 | 192.168.0.128/26 | 192.168.0.64/27 |
| vSAN | 192.168.2.0/24 | 192.168.1.128/25 | 192.168.0.192/26 | 192.168.0.96/27 |
| Transporte de host de NSX-T | 192.168.4.0/23 | 192.168.2.0/24 | 192.168.1.0/25 | 192.168.0.128/26 |
| Transporte perimetral de NSX-T | 192.168.7.208/28 | 192.168.3.208/28 | 192.168.1.208/28 | 192.168.0.208/28 |
| Vínculo superior de NSX-T 1 | 192.168.7.224/28 | 192.168.3.224/28 | 192.168.1.224/28 | 192.168.0.224/28 |
| Vínculo superior de NSX-T 2 | 192.168.7.240/28 | 192.168.3.240/28 | 192.168.1.240/28 | 192.168.0.240/28 |

## <a name="next-steps"></a>Pasos siguientes

* [Creación y administración de VLAN y subredes](create-vlan-subnet.md)
