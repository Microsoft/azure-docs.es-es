---
title: 'Conceptos: interconectividad de red'
description: Conozca los aspectos clave y los casos de uso de redes e interconectividad en la solución de VMware en Azure (AVS)
ms.topic: conceptual
ms.date: 07/23/2020
ms.openlocfilehash: c0416da9c745ccf92970ff39f623a782d5784983
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87062845"
---
# <a name="azure-vmware-solution-avs-preview-networking-and-interconnectivity-concepts"></a>Conceptos de interconectividad y redes de la versión preliminar de la solución de VMware en Azure (AVS)

La interconectividad de red entre las nubes privadas de Azure VMware Solution (AVS) y los entornos locales o las redes virtuales de Azure permite acceder a la nube privada y usarla. En este artículo se tratan algunos conceptos clave que establecen la base de las redes y la interconectividad.

Una perspectiva útil de la interconectividad es considerar los dos tipos de implementaciones de nube privada de AVS:

1. La [**interconectividad básica solo de Azure**](#azure-virtual-network-interconnectivity) permite administrar y usar la nube privada con una sola red virtual en Azure. Esta implementación es más adecuada para las evaluaciones o implementaciones de AVS que no requieren acceso desde entornos locales.

1. La [**interconectividad completa local a nube privada**](#on-premises-interconnectivity) extiende la implementación básica solo de Azure para incluir la interconectividad entre las nubes privadas locales y de AVS.
 
Puede encontrar más información sobre los requisitos y los dos tipos de implementaciones de interconectividad de nube privada de AVS descritos en las secciones siguientes.

## <a name="avs-private-cloud-use-cases"></a>Casos de uso de nube privada de AVS

Los casos de uso de nubes privadas de AVS incluyen:
- nuevas cargas de trabajo de máquinas virtuales de VMware en la nube
- expansión de carga de trabajo de máquina virtual a la nube (solo local a AVS)
- migración de carga de trabajo de máquina virtual a la nube (solo local a AVS)
- recuperación ante desastres (AVS a AVS o local a AVS)
- consumo de servicios de Azure

 Todos los casos de uso del servicio AVS se habilitan con conectividad del entorno local a la nube privada. 

## <a name="virtual-network-and-expressroute-circuit-requirements"></a>Requisitos de red virtual y circuito ExpressRoute
 
Al crear una conexión desde una red virtual de la suscripción, el circuito ExpressRoute se establece por medio del emparejamiento, usa una clave de autorización y un identificador de emparejamiento que se solicita en Azure Portal. El emparejamiento es una conexión privada de uno a uno entre la nube privada y la red virtual.

> [!NOTE] 
> El circuito ExpressRoute no forma parte de una implementación de nube privada. El circuito de ExpressRoute local está fuera del ámbito de este documento. Si necesita una conectividad desde el entorno local a su nube privada, puede utilizar uno de los circuitos de ExpressRoute existentes o comprar uno en Azure Portal.

Al implementar una nube privada, se reciben direcciones IP de vCenter y NSX-T Manager. Para acceder a esas interfaces de administración, es necesario crear recursos adicionales en una red virtual de la suscripción. En los tutoriales puede encontrar los procedimientos para crear esos recursos y establecer el emparejamiento privado de ExpressRoute.

Las redes lógicas de nube privada incluyen NSX-T previamente aprovisionado. Se aprovisionan previamente de forma automática una puerta de enlace de nivel 0 y una de nivel 1. Puede crear un segmento y asociarlo a la puerta de enlace de nivel 1 existente o hacerlo a una nueva puerta de enlace de nivel 1 que puede definir. Los componentes de redes lógicas NSX-T proporcionan conectividad horizontal de derecha a izquierda entre cargas de trabajo y también proporcionan conectividad vertical de arriba abajo a Internet y a los servicios de Azure. 

## <a name="routing-and-subnet-requirements"></a>Requisitos de enrutamiento y subred

El enrutamiento se basa en el Protocolo de puerta de enlace de borde (BGP), que se aprovisiona y habilita automáticamente de forma predeterminada para cada implementación de nube privada. En las nubes privadas de AVS, es necesario planear los espacios de direcciones de red de la nube privada con un mínimo de bloques de direcciones de red CIDR de longitud de prefijo /22 para subredes, que se muestran en la tabla siguiente. El bloque de direcciones no debe superponerse a los bloques de direcciones usados en otras redes virtuales ubicadas en la suscripción y las redes locales. Dentro de este bloque de direcciones, la administración, el aprovisionamiento y las redes de vMotion se aprovisionan automáticamente.

Un ejemplo de bloque de direcciones de red CIDR `/22` es `10.10.0.0/22`

Las subredes:

| Uso de red             | Subnet | Ejemplo        |
| ------------------------- | ------ | -------------- |
| Administración de la nube privada  | `/24`  | `10.10.0.0/24` |
| Red vMotion           | `/24`  | `10.10.1.0/24` |
| Cargas de trabajo de máquinas virtuales              | `/24`  | `10.10.2.0/24` |
| Emparejamiento de ExpressRoute      | `/24`  | `10.10.3.8/30` |


## <a name="azure-virtual-network-interconnectivity"></a>Conectividad de la red virtual de Azure

En la implementación de red virtual a nube privada, puede administrar la nube privada de AVS, usar cargas de trabajo de la nube privada y acceder a servicios de Azure mediante la conexión de ExpressRoute. 

En el diagrama siguiente se muestra la interconectividad de red básica establecida al implementar una nube privada. Muestra las redes lógicas basadas en ExpressRoute entre una red virtual de Azure y una nube privada. La interconectividad cumple tres de los casos de uso principales:
* Acceso de entrada a vCenter Server y a NSX-T Manager que es accesible desde las máquinas virtuales de la suscripción de Azure y no desde los sistemas locales. 
* Acceso de salida desde las máquinas virtuales a los servicios de Azure. 
* Acceso de entrada y consumo de cargas de trabajo que ejecutan una nube privada.

:::image type="content" source="media/concepts/adjacency-overview-drawing-single.png" alt-text="Conectividad de red virtual básica a nube privada" border="false":::

## <a name="on-premises-interconnectivity"></a>Interconectividad local

En la implementación de red virtual y local a nube privada completa, puede acceder a las nubes privadas de AVS desde entornos locales. Esta implementación es una extensión de la implementación básica descrita en la sección anterior. Al igual que la implementación básica, se requiere un circuito ExpressRoute pero, con esta implementación, se usa para conectarse desde entornos locales a la nube privada en Azure. 

En el diagrama siguiente se muestra la interconectividad local a nube privada, que permite los siguientes casos de uso:
* Cross-vCenter vMotion frecuente/poco frecuente
* Acceso de administración desde el entorno local a la nube privada de AVS

:::image type="content" source="media/concepts/adjacency-overview-drawing-double.png" alt-text="Red virtual y conectividad de nube privada local completa" border="false":::

Para una interconectividad total a la nube privada, habilite ExpressRoute Global Reach y solicite una clave de autorización y un identificador de emparejamiento privado de Global Reach en Azure Portal. La clave de autorización y el identificador de emparejamiento se usan para establecer Global Reach entre un circuito ExpressRoute de la suscripción y el circuito ExpressRoute de la nueva nube privada. Una vez vinculados, los dos circuitos ExpressRoute enrutan el tráfico de red entre los entornos locales a la nube privada.  Vea el [tutorial para crear un emparejamiento de ExpressRoute Global Reach con una nube privada](tutorial-expressroute-global-reach-private-cloud.md) para conocer los procedimientos necesarios para solicitar y usar la clave de autorización y el identificador de emparejamiento.


## <a name="next-steps"></a>Pasos siguientes 

El siguiente paso es obtener información sobre los [conceptos de almacenamiento de nubes privadas](concepts-storage.md).

<!-- LINKS - external -->
[enable Global Reach]: ../expressroute/expressroute-howto-set-global-reach.md

<!-- LINKS - internal -->
