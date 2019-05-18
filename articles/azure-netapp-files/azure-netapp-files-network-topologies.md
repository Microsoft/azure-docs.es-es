---
title: Directrices para Azure Files de NetApp planeación de la red | Microsoft Docs
description: Describe las instrucciones que pueden ayudarle a diseñar una arquitectura de red eficaz mediante el uso de Azure Files de NetApp.
services: azure-netapp-files
documentationcenter: ''
author: b-juche
manager: ''
editor: ''
ms.assetid: ''
ms.service: azure-netapp-files
ms.workload: storage
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 05/08/2019
ms.author: b-juche
ms.openlocfilehash: fa2de14ada5d24531dfecc7f2f709a87f39ea6cb
ms.sourcegitcommit: be9fcaace62709cea55beb49a5bebf4f9701f7c6
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/17/2019
ms.locfileid: "65826437"
---
# <a name="guidelines-for-azure-netapp-files-network-planning"></a>Instrucciones para el planeamiento de red de Azure NetApp Files

Planeación de la arquitectura de red es un elemento clave del diseño de cualquier infraestructura de aplicación. Este artículo le ayudará a diseñar una arquitectura de red eficaz para las cargas de trabajo para beneficiarse de las completas funcionalidades de Azure Files de NetApp.

Volúmenes de Azure Files de NetApp están diseñados para incluirse en una subred de propósito especial llamada [delegada subred](https://docs.microsoft.com/azure/virtual-network/virtual-network-manage-subnet) dentro de la red Virtual de Azure. Por lo tanto, se pueden acceder a los volúmenes directamente desde la red virtual, desde redes virtuales emparejadas en la misma región o desde un entorno local a través de una Virtual Network Gateway (puerta de enlace de VPN o ExpressRoute) según sea necesario. La subred está dedicada a Azure Files de NetApp y no hay ninguna conectividad a internet o de otros servicios de Azure.

## <a name="considerations"></a>Consideraciones  

Debe entender algunas consideraciones al planear de red de Azure Files de NetApp.

### <a name="constraints"></a>Restricciones

Las siguientes funciones no se admiten actualmente para los archivos de NetApp de Azure: 

* Grupos de seguridad de red (NSG) en la subred
* Rutas definidas por el usuario (Udr) con próximo salto que la subred de los archivos de NetApp de Azure
* Directivas de Azure (por ejemplo, directivas de nomenclatura personalizadas) en la interfaz de Azure Files de NetApp
* Equilibradores de carga para el tráfico de Azure Files de NetApp

Las siguientes restricciones de red se aplican a los archivos de NetApp de Azure:

* El número de máquinas virtuales que puede conectarse a un volumen (con una red virtual o entre redes virtuales emparejadas) no puede superar los 1000.
* En cada red virtual de Azure (VNet), solo puede delegarse una subred a Azure NetApp Files.


### <a name="supported-network-topologies"></a>Topologías de red admitidas

En la tabla siguiente se describe las topologías de red compatibles con Azure Files de NetApp.  También se describen las soluciones alternativas para las topologías no admitidas. 

|    Topologías    |    se admite    |     Solución alternativa    |
|-------------------------------------------------------------------------------------------------------------------------------|--------------------|-----------------------------------------------------------------------------|
|    Conectividad con el volumen en una red virtual local    |    Sí    |         |
|    Conectividad al volumen en una red virtual emparejada (misma región)    |    Sí    |         |
|    Conectividad al volumen en una red virtual emparejada (tránsito de emparejamiento global o región)    |    No    |    None    |
|    Conectividad a un volumen a través de puerta de enlace de ExpressRoute    |    Sí    |         |
|    Conectividad de local a un volumen en un radio de la red virtual a través de emparejamiento con tránsito de puerta de enlace de red virtual y puerta de enlace de ExpressRoute    |    No    |    Crear una subred delegada en el concentrador de red virtual (red virtual de Azure con la puerta de enlace)    |
|    Conectividad de local a un volumen en un radio de la red virtual a través de puerta de enlace de VPN    |    Sí    |         |
|    Conectividad de local a un volumen en un radio de la red virtual a través de emparejamiento con tránsito de puerta de enlace de red virtual y puerta de enlace VPN    |    Sí    |         |


## <a name="virtual-network-for-azure-netapp-files-volumes"></a>Red virtual para volúmenes de Azure Files de NetApp

En esta sección se explica los conceptos que le ayudarán a planear la red virtual.

### <a name="azure-virtual-networks"></a>Redes virtuales de Azure

Antes de aprovisionar un volumen de Azure Files de NetApp, deberá crear una red virtual de Azure (VNet) o utilizar uno que ya existe en su suscripción. La red virtual define los límites de red del volumen.  Para obtener más información sobre cómo crear redes virtuales, consulte el [documentación de Azure Virtual Network](https://docs.microsoft.com/azure/virtual-network/virtual-networks-overview).

### <a name="subnets"></a>Subredes

Subredes segmentación la red virtual en los espacios de direcciones independientes que se pueden usar los recursos de Azure en ellos.  Volúmenes de Azure Files de NetApp se encuentran en una subred con una finalidad especial llamada [delegada subred](https://docs.microsoft.com/azure/virtual-network/virtual-network-manage-subnet). 

Delegación de la subred proporciona permisos explícitos para el servicio Azure Files de NetApp para crear los recursos específicos del servicio en la subred.  Usa un identificador único en la implementación del servicio. En este caso, se crea una interfaz de red para habilitar la conectividad a Azure Files de NetApp.

Si usa una red virtual nueva, puede crear una subred y delegar la subred a los archivos de NetApp Azure siguiendo las instrucciones de [delegar una subred para Azure Files de NetApp](azure-netapp-files-delegate-subnet.md). También puede delegar una subred vacía existente que ya no se delega a otros servicios.

Si la red virtual se empareja con otra red virtual, no se puede expandir el espacio de direcciones de red virtual. Por ese motivo, la nueva subred delegada debe crearse en el espacio de direcciones de red virtual. Si necesita ampliar el espacio de direcciones, debe eliminar el emparejamiento antes de expandir el espacio de direcciones de redes virtuales.

### <a name="udrs-and-nsgs"></a>Udr y NSG

Grupos de seguridad de red (NSG) con el próximo salto no se puede usar como subredes de delegado para Azure Files de NetApp. De forma similar, también no se admiten rutas definidas por el usuario (Udr). 

Como alternativa, puede aplicar los NSG a otras subredes que permiten o deniegan el tráfico hacia y desde la subred de Azure Files de NetApp delegada.  

## <a name="azure-native-environments"></a>Entornos nativos de Azure

El siguiente diagrama muestra un entorno nativo de Azure:

![Entorno de red de Azure nativas](../media/azure-netapp-files/azure-netapp-files-network-azure-native-environment.png)

### <a name="local-vnet"></a>Red virtual local

Un escenario básico consiste en crear ni conectarse a un volumen de Azure Files de NetApp desde una máquina virtual (VM) en la misma red virtual. Para la red virtual 2 en el diagrama anterior, volumen 1 se crea en una subred delegada y se pueden montar en la máquina virtual 1 en la subred predeterminada.

### <a name="vnet-peering"></a>Emparejamiento de VNET

Si tiene redes virtuales adicionales en la misma región que necesitan tener acceso a recursos de los demás, las redes virtuales se pueden conectar mediante [emparejamiento de VNet](https://docs.microsoft.com/azure/virtual-network/virtual-network-peering-overview) para habilitar la conectividad segura a través de la infraestructura de Azure. 

Considere la posibilidad de red virtual 2 y 3 de la red virtual en el diagrama anterior. Si necesita conectarse a máquina virtual 2 y 2 de volumen 1 de la máquina virtual, o si necesita conectarse a la máquina virtual 1 o 1 volumen 2 de la máquina virtual, debe habilitar el emparejamiento de VNet entre la red virtual 2 y 3 de la red virtual. 

Además, considere un escenario donde 1 red virtual está emparejada con la red virtual 2 y VNet 2 está emparejada con 3 de la red virtual en la misma región. Los recursos de red virtual 1 pueden conectarse a los recursos de red virtual 2, pero no se puede conectar a los recursos de red virtual 3, a menos que la red virtual 1 y 3 de la red virtual están emparejadas. 

En el diagrama anterior, aunque el 3 de la máquina virtual puede conectarse al volumen 1, 4 de la máquina virtual no se puede conectar al volumen 2.  La razón es que no se emparejan redes virtuales de radios, y _no se admite el enrutamiento de tránsito a través del emparejamiento de red virtual_.

## <a name="hybrid-environments"></a>Entornos híbridos

El siguiente diagrama muestra un entorno híbrido: 

![Entorno de red híbrida](../media/azure-netapp-files/azure-netapp-files-networ-hybrid-environment.png)

En el escenario híbrido, las aplicaciones de centros de datos locales necesitan acceso a los recursos de Azure.  Este es el caso si desea ampliar su centro de datos en Azure, o desea usar servicios nativos de Azure o para la recuperación ante desastres. Consulte [planear las opciones de puerta de enlace de VPN](https://docs.microsoft.com/azure/vpn-gateway/vpn-gateway-about-vpngateways?toc=%2fazure%2fvirtual-network%2ftoc.json#planningtable) sobre cómo conectar varios recursos locales a los recursos de Azure a través de una VPN de sitio a sitio o ExpressRoute.

En una topología de concentrador y radio híbrido, el centro de la red virtual de Azure actúa como un punto central de conectividad a la red local. Los radios se emparejan redes virtuales con el centro y se puede usar para aislar las cargas de trabajo.

Según la configuración. Puede conectar los recursos locales a los recursos en el concentrador y los radios.

En la topología que se ilustra arriba, la red local está conectada a un concentrador de red virtual en Azure, y hay radio 2 redes virtuales emparejadas con la red virtual del concentrador.  En este escenario, las opciones de conectividad que se admite para los volúmenes de Azure Files de NetApp son los siguientes:

* En los recursos locales VM 1 y 2 de la máquina virtual pueden conectarse a 1 volumen en el concentrador a través de una VPN de sitio a sitio o ExpressRoute. 
* En los recursos locales VM 1 y 2 de la máquina virtual pueden conectarse a volumen 2 o 3 de volumen.
* 3 de la máquina virtual en el concentrador de red virtual puede conectarse a volumen 2 del radio 1 red virtual y el volumen 3 en radio 2 de la red virtual.
* 4 de la máquina virtual de radio 1 red virtual y 5 de la máquina virtual de radio 2 de la red virtual pueden conectarse a 1 volumen en la red virtual del concentrador.

4 de la máquina virtual de radio 1 red virtual no se puede conectar al volumen 3 en 2 de la red virtual de radio. Además, 5 de la máquina virtual de radio VNet2 no se puede conectar al volumen 2 del radio 1 red virtual. Esto sucede porque no se emparejan redes virtuales de radios y _no se admite el enrutamiento de tránsito a través del emparejamiento de red virtual_.

## <a name="next-steps"></a>Pasos siguientes

[Delegación de una subred en Azure NetApp Files](azure-netapp-files-delegate-subnet.md)
