---
title: Protección de los recursos de red en Azure Security Center
description: En este documento se explica cómo las recomendaciones de Azure Security Center ayudan a proteger los recursos de red y a cumplir las directivas de seguridad.
services: security-center
documentationcenter: na
author: memildin
manager: rkarlin
ms.assetid: 96c55a02-afd6-478b-9c1f-039528f3dea0
ms.service: security-center
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 04/05/2019
ms.author: memildin
ms.openlocfilehash: 868470292fbacd71e1eb2d39de7e3a9c5cf6900e
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90883891"
---
# <a name="protect-your-network-resources"></a>Protección de los recursos de red
Azure Security Center analiza continuamente el estado de seguridad de los recursos de Azure para los procedimientos recomendados de seguridad de red. Cuando Security Center identifica posibles vulnerabilidades de seguridad, crea recomendaciones que lo guiarán por el proceso de configuración de los controles necesarios para reforzar y proteger sus recursos.

Para ver la lista completa de las recomendaciones para redes, consulte [Recomendaciones de red](recommendations-reference.md#recs-network).

En este artículo se abordan las recomendaciones que se aplican a los recursos de Azure desde una perspectiva de seguridad de red. Las recomendaciones sobre redes se centran en los firewalls de próxima generación, los grupos de seguridad de red, el acceso a la VM JIT, las reglas de tráfico entrantes permisivas y mucho más. Para obtener una lista de posibles recomendaciones de red y acciones de corrección, consulte [Administración de recomendaciones de seguridad en Azure Security Center.](security-center-recommendations.md)

Las características de **red** de Security Center incluyen: 

- Mapa de red (requiere Azure Defender)
- [Protección de red adaptable](security-center-adaptive-network-hardening.md) (requiere Azure Defender)
- Recomendaciones de seguridad de red
 
## <a name="view-your-networking-resources-and-their-recommendations"></a>Ver los recursos de red y sus recomendaciones

En la [página del inventario de recursos](asset-inventory.md), use el filtro de tipo de recurso para seleccionar los recursos de red que desea investigar:

:::image type="content" source="./media/security-center-network-recommendations/network-filters-inventory.png" alt-text="Tipos de recursos de red del inventario de recursos" lightbox="./media/security-center-network-recommendations/network-filters-inventory.png":::


## <a name="network-map"></a>Mapa de red

El mapa de red interactivo proporciona una vista gráfica con superposiciones de seguridad que le ofrecen recomendaciones e información para proteger sus recursos de red. Con el mapa podrá ver la topología de red de sus cargas de trabajo de Azure, las conexiones entre sus máquinas virtuales y subredes, y la capacidad de explorar en profundidad el mapa para consultar recursos específicos y las recomendaciones para esos recursos.

Para abrir el mapa de red:

1. En el menú de Security Center, abra el panel de Azure Defender y seleccione **Mapa de red**.

    :::image type="content" source="./media/security-center-network-recommendations/opening-network-map.png" alt-text="Apertura del mapa de red desde el panel de Azure Defender" lightbox="./media/security-center-network-recommendations/opening-network-map.png":::

1. Seleccione el menú **Capas** y elija **Topología**.
 
La vista predeterminada del mapa topológico muestra:

- Suscripciones seleccionadas en Azure. El mapa admite varias suscripciones.
- Máquinas virtuales, subredes y redes virtuales del tipo de recurso de Resource Manager (no se admiten los recursos de Azure clásico)
- Redes virtuales emparejadas
- Solo los recursos que tienen [recomendaciones de red](security-center-recommendations.md) con una gravedad media o alta  
- Recursos accesibles desde Internet
- El mapa está optimizado para las suscripciones que seleccionó en Azure. Si modifica la selección, el mapa se vuelve a calcular y a reoptimizar en función de la nueva configuración.  

[![Mapa de topología de red](./media/security-center-network-recommendations/network-map-info.png)](./media/security-center-network-recommendations/network-map-info.png#lightbox)

## <a name="understanding-the-network-map"></a>Descripción del mapa de red

El mapa de red puede mostrar los recursos de Azure en una vista **Topología** y en una vista **Tráfico**. 

### <a name="the-topology-view"></a>Vista de topología

En la vista **Topología** del mapa de red, puede ver la información siguiente acerca de los recursos de red:

- En el círculo interior, puede ver todas las redes virtuales dentro de las suscripciones seleccionadas, el círculo siguiente son todas las subredes, el círculo exterior son todas las máquinas virtuales.
- Las líneas que conectan los recursos en el mapa le permiten saber qué recursos se asocian entre sí, y cómo está estructurada la red de Azure. 
- Use los indicadores de gravedad para obtener rápidamente información general acerca de qué recursos tienen recomendaciones abiertas desde Security Center.
- Puede hacer clic en cualquiera de los recursos para explorar en profundidad cada uno de ellos y ver los detalles de ese recurso y sus recomendaciones directamente en el contexto de la red.  
- Si se muestran demasiados recursos en el mapa, Azure Security Center usa su algoritmo propietario para agruparlos de manera inteligente y resaltar aquellos en estado más crítico y con recomendaciones de gravedad alta. 

Dado que el mapa es interactivo y dinámico, todos los nodos son seleccionables y la vista puede cambiarse en función de los filtros:

1. Puede modificar lo que ve en el mapa de red mediante el uso de los filtros situados en la parte superior. Puede centrarse en el mapa basándose en:

   -  **Estado de seguridad**: puede filtrar el mapa basándose en la gravedad (alta, media, baja) de los recursos de Azure.
   - **Recomendaciones**: puede seleccionar qué recursos se muestran según las recomendaciones activas en esos recursos. Por ejemplo, puede ver únicamente los recursos para los que Security Center recomienda habilitar los grupos de seguridad de red.
   - **Zonas de red**: de manera predeterminada, el mapa muestra solo recursos accesibles desde Internet, también puede seleccionar máquinas virtuales internas.
 
2. Puede hacer clic en **Restablecer** en la esquina superior izquierda en cualquier momento para devolver el mapa al estado predeterminado.

Para explorar en profundidad un recurso:

1. Al seleccionar un recurso específico en el mapa, el panel derecho se abre y le ofrece información general sobre el recurso, soluciones de seguridad conectadas (de existir) y recomendaciones relevantes para el recurso. Es el mismo tipo de comportamiento para cada tipo de recurso que seleccione. 
2. Cuando mantiene el mouse sobre un nodo en el mapa, puede ver información general sobre el recurso, incluida la suscripción, el tipo de recurso y el grupo de recursos.
3. Utilice el vínculo para acercar la información sobre herramientas y reorientar el mapa en ese nodo concreto. 
4. Para reorientar el mapa fuera de un determinado nodo debe alejar.

### <a name="the-traffic-view"></a>Vista de tráfico

La vista de **tráfico** proporciona un mapa con todo el tráfico posible entre los recursos. Esto proporciona un mapa visual de todas las reglas configuradas que define qué recursos pueden comunicarse entre sí. Esto permite ver la configuración existente de los grupos de seguridad de red, así como identificar rápidamente las posibles configuraciones de riesgo en las cargas de trabajo.

### <a name="uncover-unwanted-connections"></a>Descubrir conexiones no deseadas

La importancia de esta vista reside en su capacidad para mostrar estas conexiones permitidas junto con las vulnerabilidades que existen, por lo que puede usar esta sección transversal de datos para aplicar la protección necesaria a los recursos. 

Por ejemplo, podría detectar dos máquinas que no sabía que se podían comunicar, lo que le permitiría aislar mejor cargas de trabajo y subredes.

### <a name="investigate-resources"></a>Investigar los recursos

Para explorar en profundidad un recurso:

1. Al seleccionar un recurso específico en el mapa, el panel derecho se abre y le ofrece información general sobre el recurso, soluciones de seguridad conectadas (de existir) y recomendaciones relevantes para el recurso. Es el mismo tipo de comportamiento para cada tipo de recurso que seleccione. 
2. Haga clic en **Tráfico** para ver la lista de posible tráfico entrante y saliente en el recurso. Se trata de una lista completa de quién puede comunicarse con el recurso y viceversa, así como a través de qué protocolos y puertos. Por ejemplo, cuando se selecciona una máquina virtual, todas las máquinas virtuales con las que puede comunicarse se muestran y, cuando se selecciona una subred, se muestran todas las subredes con las que se puede comunicar.

**Estos datos se basan en el análisis de los Grupos de seguridad de red, así como en algoritmos con aprendizaje automático que analizan varias reglas para comprender las interacciones y cruces.** 

[![Mapa del tráfico de red](./media/security-center-network-recommendations/network-map-traffic.png)](./media/security-center-network-recommendations/network-map-traffic.png#lightbox)


## <a name="next-steps"></a>Pasos siguientes

Para obtener más información sobre las recomendaciones que se aplican a otros tipos de recursos de Azure, consulte los siguientes artículos:

- [Protección de las aplicaciones y las máquinas en Azure Security Center](security-center-virtual-machine-protection.md)