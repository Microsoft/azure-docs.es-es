---
title: 'Conceptos: interconectividad de red'
description: Conozca los aspectos clave y los casos de uso de redes e interconectividad en Azure VMware Solution.
ms.topic: conceptual
ms.date: 07/23/2020
ms.openlocfilehash: 3420f6aa61ced7632175f3e12edda9de72639517
ms.sourcegitcommit: 62717591c3ab871365a783b7221851758f4ec9a4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/22/2020
ms.locfileid: "88750565"
---
# <a name="azure-vmware-solution-preview-networking-and-interconnectivity-concepts"></a>Conceptos de interconectividad y redes en la versión preliminar de Azure VMware Solution

Azure VMware Solution (AVS) ofrece un entorno de nube privada de VMware accesible para usuarios y aplicaciones desde recursos o entornos locales y basados en Azure. Servicios como Azure ExpressRoute y las conexiones VPN proporcionan la conectividad. Estos servicios requieren intervalos de direcciones de red y puertos de firewall específicos para habilitar los servicios.  

Cuando se implementa una nube privada, se crean redes privadas para la administración, el aprovisionamiento y vMotion. Se usan para acceder a vCenter y NSX-T Manager, así como para la implementación o vMotion de máquinas virtuales. Se puede acceder a todas las redes privadas desde una red virtual de Azure o desde entornos locales. ExpressRoute Global Reach se usa para conectar nubes privadas a entornos locales, y esta conexión requiere una red virtual con un circuito ExpressRoute en su suscripción.

Además, al implementar una nube privada, el acceso a Internet y los servicios de Azure se aprovisionan y proporcionan para que las máquinas virtuales de las redes de producción puedan consumirlos.  El acceso a Internet está deshabilitado de forma predeterminada para las nuevas nubes privadas, pero puede habilitarse o deshabilitarse en cualquier momento.

Una perspectiva útil de la interconectividad es considerar los dos tipos de implementaciones de nube privada de AVS:

1. La [**interconectividad básica solo de Azure**](#azure-virtual-network-interconnectivity) permite administrar y usar la nube privada con una sola red virtual en Azure. Esta implementación es más adecuada para las evaluaciones o implementaciones de AVS que no requieren acceso desde entornos locales.

1. La [**interconectividad completa local a nube privada**](#on-premises-interconnectivity) extiende la implementación básica solo de Azure para incluir la interconectividad entre las nubes privadas locales y de AVS.
 
En este artículo se tratan algunos conceptos clave que establecen las redes y la interconectividad, incluidos los requisitos y las limitaciones. También trataremos más información sobre los dos tipos de implementaciones de la interconectividad de la nube privada de AVS. En este artículo se proporciona la información necesaria para configurar correctamente las redes a fin de que funcionen con AVS.

## <a name="avs-private-cloud-use-cases"></a>Casos de uso de nube privada de AVS

Los casos de uso de nubes privadas de AVS incluyen:
- nuevas cargas de trabajo de máquinas virtuales de VMware en la nube
- expansión de carga de trabajo de máquina virtual a la nube (solo local a AVS)
- migración de carga de trabajo de máquina virtual a la nube (solo local a AVS)
- recuperación ante desastres (AVS a AVS o local a AVS)
- Consumo de servicios de Azure

> [!TIP]
> Todos los casos de uso del servicio AVS se habilitan con conectividad del entorno local a la nube privada.

## <a name="azure-virtual-network-interconnectivity"></a>Conectividad de la red virtual de Azure

En la implementación de red virtual a nube privada, puede administrar la nube privada de Azure VMware Solution, usar cargas de trabajo de la nube privada y acceder a servicios de Azure mediante la conexión de ExpressRoute. 

En el diagrama siguiente se muestra la interconectividad de red básica establecida al implementar una nube privada. Muestra las redes lógicas basadas en ExpressRoute entre una red virtual de Azure y una nube privada. La interconectividad cumple tres de los casos de uso principales:
* Acceso de entrada a vCenter Server y a NSX-T Manager que es accesible desde las máquinas virtuales de la suscripción de Azure y no desde los sistemas locales. 
* Acceso de salida desde las máquinas virtuales a los servicios de Azure. 
* Acceso de entrada y consumo de cargas de trabajo que ejecutan una nube privada.

:::image type="content" source="media/concepts/adjacency-overview-drawing-single.png" alt-text="Conectividad de red virtual básica a nube privada" border="false":::

## <a name="on-premises-interconnectivity"></a>Interconectividad local

En la implementación de red virtual y local a nube privada completa, puede acceder a las nubes privadas de Azure VMware Solution desde entornos locales. Esta implementación es una extensión de la implementación básica descrita en la sección anterior. Al igual que la implementación básica, se requiere un circuito ExpressRoute pero, con esta implementación, se usa para conectarse desde entornos locales a la nube privada en Azure. 

En el diagrama siguiente se muestra la interconectividad local a nube privada, que permite los siguientes casos de uso:
* Cross-vCenter vMotion frecuente/poco frecuente
* Acceso de administración desde los entornos locales a la nube privada de Azure VMware Solution

:::image type="content" source="media/concepts/adjacency-overview-drawing-double.png" alt-text="Red virtual y conectividad de nube privada local completa" border="false":::

Para una interconectividad total a la nube privada, habilite ExpressRoute Global Reach y solicite una clave de autorización y un identificador de emparejamiento privado de Global Reach en Azure Portal. La clave de autorización y el identificador de emparejamiento se usan para establecer Global Reach entre un circuito ExpressRoute de la suscripción y el circuito ExpressRoute de la nueva nube privada. Una vez vinculados, los dos circuitos ExpressRoute enrutan el tráfico de red entre los entornos locales a la nube privada.  Vea el [tutorial para crear un emparejamiento de ExpressRoute Global Reach con una nube privada](tutorial-expressroute-global-reach-private-cloud.md) para conocer los procedimientos necesarios para solicitar y usar la clave de autorización y el identificador de emparejamiento.

## <a name="next-steps"></a>Pasos siguientes 

- Más información sobre las [consideraciones y los requisitos de la conectividad de red](tutorial-network-checklist.md) 
- Información sobre los [conceptos de almacenamiento en la nube privada](concepts-storage.md)


<!-- LINKS - external -->
[enable Global Reach]: ../expressroute/expressroute-howto-set-global-reach.md

<!-- LINKS - internal -->

