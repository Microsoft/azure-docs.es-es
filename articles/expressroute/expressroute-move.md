---
title: 'ExpressRoute: Traslado de circuitos del modelo clásico a Azure Resource Manager'
description: Obtenga información sobre lo que significa mover un circuito Azure ExpressRoute del modelo de implementación clásica al de Azure Resource Manager.
services: expressroute
author: duongau
ms.service: expressroute
ms.topic: how-to
ms.date: 12/07/2018
ms.author: duau
ms.openlocfilehash: 12ace8dff58d3d747bf17260950d2b91e31b398a
ms.sourcegitcommit: 5a3b9f35d47355d026ee39d398c614ca4dae51c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89395169"
---
# <a name="moving-expressroute-circuits-from-the-classic-to-the-resource-manager-deployment-model"></a>Transición de los circuitos ExpressRoute del modelo de implementación clásica al modelo de implementación de Resource Manager
En este artículo se ofrece información general de lo que significa mover un circuito Azure ExpressRoute del modelo de implementación clásica al de Azure Resource Manager.

Se puede usar un solo circuito ExpressRoute para conectarse a redes virtuales implementadas tanto en el modelo de implementación clásica como en el modelo de Resource Manager. Con independencia de cómo se cree, ahora se puede vincular un circuito ExpressRoute a redes virtuales en ambos modelos de implementación.

![Un circuito ExpressRoute que se vincula a redes virtuales a través de ambos modelos de implementación](./media/expressroute-move/expressroute-move-1.png)

## <a name="expressroute-circuits-that-are-created-in-the-classic-deployment-model"></a>Circuitos ExpressRoute creados en el modelo de implementación clásica
Los circuitos ExpressRoute creados en el modelo de implementación clásica deben trasladarse primero al modelo de implementación de Resource Manager para permitir la conectividad con los modelos de implementación clásica y de Resource Manager. Mientras se traslada una conexión, no se producen pérdida de conectividad ni interrupciones. Todos los vínculos entre el circuito y la red virtual en el modelo de implementación clásica (dentro de la misma suscripción y entre suscripciones) se conservan.

Finalizada la transición, el circuito ExpressRoute funciona exactamente igual que un circuito ExpressRoute creado en el modelo de implementación de Resource Manager. Ahora podrá crear conexiones a redes virtuales en el modelo de implementación de Resource Manager.

Cuando se haya trasladado el circuito ExpressRoute al modelo de implementación de Resource Manager, solo podrá administrar el ciclo de vida del circuito ExpressRoute con el modelo de implementación de Resource Manager. Esto significa que operaciones tales como agregar, actualizar y eliminar emparejamientos; actualizar propiedades del circuito, como el ancho de banda, el SKU y el tipo de facturación; y eliminar circuitos, solo se pueden realizar en el modelo de implementación de Resource Manager. Consulte la sección siguiente sobre los circuitos creados en el modelo de implementación de Resource Manager para más información sobre cómo administrar el acceso a ambos modelos de implementación.

No es necesario que su proveedor de conectividad intervenga en la transición.

## <a name="expressroute-circuits-that-are-created-in-the-resource-manager-deployment-model"></a>Circuitos ExpressRoute creados en el modelo de implementación de Resource Manager
Puede habilitar los circuitos ExpressRoute creados en el modelo de implementación de Resource Manager para que sean accesibles desde ambos modelos de implementación. Cualquier circuito ExpressRoute de su suscripción se puede habilitar para el acceso desde ambos modelos de implementación.

* Los circuitos ExpressRoute que se crearon en el modelo de implementación de Resource Manager no tienen acceso al modelo de implementación clásica de forma predeterminada.
* Los circuitos ExpressRoute que se han trasladado del modelo de implementación clásica al modelo de Resource Manager son accesibles desde ambos modelos de implementación de forma predeterminada.
* Un circuito ExpressRoute siempre tendrá acceso al modelo de implementación de Resource Manager, independientemente de si se creó en el modelo de implementación de Resource Manager o en el modelo de implementación clásica. Esto significa que puede crear conexiones a redes virtuales creadas en el modelo de implementación de Resource Manager siguiendo las instrucciones sobre [cómo se vinculan redes virtuales](expressroute-howto-linkvnet-arm.md).
* El acceso al modelo de implementación clásica se controla mediante el parámetro **allowClassicOperations** del circuito ExpressRoute.

> [!IMPORTANT]
> Se aplican todas las cuotas documentadas en la página de [límites de servicio](../azure-resource-manager/management/azure-subscription-service-limits.md) . Por ejemplo, un circuito estándar puede tener como máximo 10 vínculos de red virtual/conexiones entre los modelos de implementación clásica y de Resource Manager.
> 
> 

## <a name="controlling-access-to-the-classic-deployment-model"></a>Control del acceso al modelo de implementación clásica
Puede permitir que un solo circuito ExpressRoute se vincule a redes virtuales en ambos modelos de implementación; para ello, configure el parámetro **allowClassicOperations** del circuito ExpressRoute.

Al definir **allowClassicOperations** en TRUE podrá vincular redes virtuales de ambos modelos de implementación al circuito ExpressRoute. Para vincular redes virtuales en el modelo de implementación clásica, siga las instrucciones sobre [cómo se vinculas redes virtuales en el modelo de implementación clásica](expressroute-howto-linkvnet-classic.md). Para vincular redes virtuales en el modelo de implementación clásica, siga las instrucciones sobre [cómo se vinculan redes virtuales en el modelo de implementación de Resource Manager](expressroute-howto-linkvnet-arm.md).

Al definir **allowClassicOperations** en FALSE se bloquea el acceso al circuito desde el modelo de implementación clásica. Sin embargo, se conservan todos los vínculos a redes virtuales del modelo de implementación clásica. En este caso, el circuito ExpressRoute no será visible desde el modelo de implementación clásica.

## <a name="supported-operations-in-the-classic-deployment-model"></a>Operaciones admitidas en el modelo de implementación clásica
Cuando se define en TRUE el parámetro **allowClassicOperations** , en un circuito ExpressRoute se admiten las siguientes operaciones clásicas:

* Obtener información del circuito ExpressRoute
* Crear, actualizar, obtener y eliminar vínculos a redes virtuales clásicas
* Crear, actualizar, obtener y eliminar autorizaciones de vínculo de red virtual para la conectividad entre suscripciones

Sin embargo, cuando **allowClassicOperations** se define en TRUE, no se pueden realizar las siguientes operaciones clásicas:

* Crear, actualizar, obtener y eliminar emparejamientos de Border Gateway Protocol (BGP) para emparejamientos públicos y privados de Azure y emparejamientos de Microsoft
* Eliminar circuitos ExpressRoute

## <a name="communication-between-the-classic-and-the-resource-manager-deployment-models"></a>Comunicación entre los modelos de implementación clásica y de Resource Manager
El circuito ExpressRoute actúa como puente entre los modelos de implementación clásica y de Resource Manager. El tráfico entre las máquinas virtuales implementadas en las redes virtuales según el modelo clásico y las implementadas en las redes virtuales según el de Resource Manager fluirá a través de ExpressRoute si ambas redes virtuales están vinculadas al mismo circuito ExpressRoute.

El rendimiento agregado se ve limitado por la capacidad de procesamiento de la puerta de enlace de red virtual. En tales casos, el tráfico no pasa al proveedor de conectividad ni a las redes. El flujo del tráfico entre las redes virtuales está completamente contenido en la red de Microsoft.

## <a name="access-to-azure-public-and-microsoft-peering-resources"></a>Acceso a recursos de emparejamiento público de Azure y de Microsoft
Puede seguir teniendo acceso a los recursos a los que normalmente se accede mediante el emparejamiento público de Azure y el emparejamiento de Microsoft sin interrupciones.  

## <a name="whats-supported"></a>Lo que se admite
En esta sección se describe lo que se admite para los circuitos ExpressRoute:

* Para acceder a redes virtuales implementadas tanto en el modelo clásico como en el de Resource Manager se puede usar un solo circuito ExpressRoute.
* Un circuito ExpressRoute se puede trasladar del modelo de implementación clásica al modelo de implementación de Resource Manager. Una vez realizada la transición, el circuito ExpressRoute funciona exactamente igual que cualquier otro circuito ExpressRoute creado en el modelo de implementación de Resource Manager.
* Solo se puede trasladar el circuito ExpressRoute. Los vínculos del circuito, las redes virtuales y las puertas de enlace de VPN no se trasladan mediante esta operación.
* Cuando se haya trasladado el circuito ExpressRoute al modelo de implementación de Resource Manager, solo podrá administrar el ciclo de vida del circuito ExpressRoute con el modelo de implementación de Resource Manager. Esto significa que operaciones tales como agregar, actualizar y eliminar emparejamientos; actualizar propiedades del circuito, como el ancho de banda, el SKU y el tipo de facturación; y eliminar circuitos, solo se pueden realizar en el modelo de implementación de Resource Manager.
* El circuito ExpressRoute actúa como puente entre los modelos de implementación clásica y de Resource Manager. El tráfico entre las máquinas virtuales implementadas en las redes virtuales según el modelo clásico y las implementadas en las redes virtuales según el de Resource Manager fluirá a través de ExpressRoute si ambas redes virtuales están vinculadas al mismo circuito ExpressRoute.
* Los modelos de implementación clásica y de Resource Manager admiten ambos la conectividad entre suscripciones.
* Después de mover un circuito ExpressRoute del modelo clásico al de Azure Resource Manager, puede [migrar las redes virtuales vinculadas al circuito ExpressRoute](expressroute-migration-classic-resource-manager.md).

## <a name="whats-not-supported"></a>Lo que no se admite
En esta sección se describe lo que no se admite para los circuitos ExpressRoute:

* Administrar el ciclo de vida de un circuito ExpressRoute desde el modelo de implementación clásica.
* Compatibilidad de control de acceso basado en rol (RBAC) para el modelo de implementación clásica. No puede realizar RBAC para un circuito del modelo de implementación clásica. Cualquier administrador o coadministrador de la suscripción puede vincular o desvincular redes virtuales al/del circuito.

## <a name="configuration"></a>Configuración
Revise las instrucciones que se ofrecen en [Transición de los circuitos ExpressRoute desde el modelo de implementación clásica al modelo de implementación de Resource Manager](expressroute-howto-move-arm.md).

## <a name="next-steps"></a>Pasos siguientes
* [Migración de las redes virtuales vinculadas al circuito ExpressRoute del modelo clásico al de Azure Resource Manager](expressroute-migration-classic-resource-manager.md)
* Para obtener información sobre los flujos de trabajo, consulte [Flujos de trabajo de aprovisionamiento de circuitos y estados de circuito de ExpressRoute](expressroute-workflows.md).
* Configure su conexión ExpressRoute:
  
  * [Creación de un circuito ExpressRoute](expressroute-howto-circuit-arm.md)
  * [Configuración del enrutamiento](expressroute-howto-routing-arm.md)
  * [Vinculación de una red virtual a un circuito ExpressRoute](expressroute-howto-linkvnet-arm.md)

