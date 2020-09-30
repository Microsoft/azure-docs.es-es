---
title: Azure Monitor for Networks (versión preliminar)
description: Información general rápida de Azure Monitor for Networks que ofrece una vista completa del estado y las métricas de todos los recursos de red implementados sin ninguna configuración.
ms.subservice: ''
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 09/22/2020
ms.openlocfilehash: 88e69ac4156bb2bdfb217a4fc83ac2d798d08211
ms.sourcegitcommit: bdd5c76457b0f0504f4f679a316b959dcfabf1ef
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90987886"
---
# <a name="azure-monitor-for-networks-preview"></a>Azure Monitor for Networks (versión preliminar)
Azure Monitor para redes ofrece una vista completa del [estado](https://docs.microsoft.com/azure/service-health/resource-health-checks-resource-types) y las [métricas](../platform/metrics-supported.md) de todos los recursos de red implementados sin ninguna configuración.  También proporciona acceso a todas las funcionalidades de supervisión de la red, como [Connection Monitor](../../network-watcher/connection-monitor-preview.md), [registro de flujo de los grupos de seguridad de red (NSG)](../../network-watcher/network-watcher-nsg-flow-logging-overview.md), [Análisis de tráfico](../../network-watcher/traffic-analytics.md) y otras características de [diagnóstico](../../network-watcher/network-watcher-monitoring-overview.md#diagnostics) de red.

Azure Monitor para redes está estructurado en torno a los siguientes componentes clave de supervisión:
- [Estado y métricas de red](#networkhealth)
- [Conectividad](#connectivity)
- [Tráfico](#traffic)
- [Kit de herramientas de diagnóstico](#diagnostictoolkit)

## <a name="network-health-and-metrics"></a><a name="networkhealth"></a>Estado y métricas de red

La página de **información general** de Azure Monitor for Networks ofrece una manera fácil de visualizar el inventario de los recursos de red junto con las alertas y el estado de los recursos. Se divide en tres áreas funcionales principales: búsqueda y filtrado, Resource Health y vista de métricas, alertas y dependencias.

![Página de información general](media/network-insights-overview/overview.png)

### <a name="search-and-filtering"></a>Búsqueda y filtrado
La vista de Resource Health y alertas se puede personalizar con filtros como **Suscripción**, **Grupo de recursos** y **Tipo de recurso**. El cuadro de búsqueda proporciona la capacidad de buscar en las propiedades de los recursos.

El cuadro de búsqueda se puede usar para buscar recursos y recursos asociados. Por ejemplo, una dirección IP pública está asociada a una instancia de Application Gateway. La búsqueda del nombre DNS de las direcciones IP públicas identificará tanto la dirección IP pública como la instancia de Application Gateway asociada.

![Azure Monitor para redes: búsqueda](media/network-insights-overview/search.png)


### <a name="resource-health-and-metric"></a>Resource Health y métrica
Cada icono representa un tipo de recurso, con el número de instancias implementadas en todas las suscripciones seleccionadas junto con el estado de Resource Health. En el ejemplo siguiente, hay 45 conexiones de ER y VPN implementadas, 44 están en buen estado y 1 no está disponible.

![Azure Monitor para redes: Resource Health](media/network-insights-overview/resource-health.png)

Al hacer clic en las dos conexiones de ER y VPN no disponibles, se inicia una vista de las métricas. 

![Azure Monitor para redes: vista de métricas](media/network-insights-overview/metric-view.png)

Puede hacer clic en cada elemento de la vista de cuadrícula. Haga clic en el icono de estado para redirigir a Resource Health para esa conexión. Haga clic en Alertas para redirigir a la página de alertas y métricas, respectivamente, para esa conexión. 

### <a name="alerts"></a>Alertas
La cuadrícula **Alertas** de la derecha ofrece una vista de todas las alertas generadas para los recursos seleccionados en todas las suscripciones. Haga clic en el número de alertas para ir a la página de alertas detalladas.

### <a name="dependency-view"></a>Vista de dependencias
La vista **Dependencia** ayuda a visualizar cómo se configura el recurso. Actualmente se admite la vista Dependencias para Application Gateway, Virtual WAN y Load Balancer. Por ejemplo, en el caso de Application Gateway, se puede acceder a la vista Dependencias al hacer clic en el nombre del recurso de Application Gateway desde la vista de cuadrícula de métricas. Esto también se aplica a Virtual WAN y Load Balancer.

![Azure Monitor para redes: vista de Application Gateway](media/network-insights-overview/application-gateway.png)

La vista **Dependencia** de Application Gateway ofrece una vista simplificada de cómo se conectan las direcciones IP de front-end a los agentes de escucha, las reglas y el grupo de back-end. Los bordes de conexión están codificados por colores y ofrecen detalles adicionales según el estado del grupo de back-end. La vista también ofrece información detallada de las métricas de Application Gateway y las métricas de todos los grupos de back-end relacionados, como las instancias de conjunto de escalado de máquinas virtuales y de máquina virtual.

![Azure Monitor para redes: vista de dependencias](media/network-insights-overview/dependency-view.png)

El gráfico de dependencias facilita la navegación a las opciones de configuración. Haga clic con el botón derecho en un grupo de back-end para acceder a otras funciones. Por ejemplo, si el grupo de back-end es una máquina virtual, puede acceder directamente a la información detallada de la máquina virtual y a la solución de problemas de conexión de Network Watcher para identificar problemas de conectividad.

![Azure Monitor para redes: menú de la vista de dependencias](media/network-insights-overview/dependency-view-menu.png)

La barra de búsqueda y de filtro de la vista de dependencias ofrece una forma sencilla de buscar en el gráfico. Por ejemplo, la búsqueda de *AppGWTestRule* en el ejemplo siguiente limitará la vista gráfica a todos los nodos conectados a través de *AppGWTestRule*.

![Azure Monitor para redes: ejemplo de búsqueda](media/network-insights-overview/search-example.png)

Los diferentes filtros proporcionan ayuda para reducir la búsqueda a una ruta y un estado específicos. Por ejemplo, seleccione solo *Incorrecto* en el menú desplegable **Estado de mantenimiento** para mostrar todos los bordes en los que el estado es *Incorrecto*.

Haga clic en la **vista de métricas detalladas** para iniciar un libro preconfigurado con métricas detalladas de Application Gateway, todos los recursos del grupo de back-end y las direcciones IP de front-end. 

## <a name="connectivity"></a><a name="connectivity"></a>Conectividad

La pestaña **Conectividad** proporciona una forma sencilla de visualizar todas las pruebas configuradas mediante Connection Monitor y [Connection Monitor (versión preliminar)](../../network-watcher/connection-monitor-preview.md) para el conjunto de suscripciones seleccionado.

![Pestaña Conectividad en Azure Monitor para redes](media/network-insights-overview/azure-monitor-for-networks-connectivity-tab.png)

Las pruebas se agrupan por iconos de orígenes y destinos y muestran el estado de disponibilidad de cada una. La configuración accesible proporciona un acceso sencillo para configurar los criterios de disponibilidad en función de las comprobaciones con errores (%) y RTT (ms). Una vez que se establecen los valores, el estado de cada prueba se actualiza en función de los criterios de selección.

![Pruebas de conectividad en Azure Monitor para redes](media/network-insights-overview/azure-monitor-for-networks-connectivity-tests.png)

Al hacer clic en cualquier icono de origen o destino, se inicia una vista de las métricas.

![Métricas de conectividad en Azure Monitor para redes](media/network-insights-overview/azure-monitor-for-networks-connectivity-metrics.png)


Puede hacer clic en cada elemento de la vista de cuadrícula. Haga clic en **Accesibilidad** para redirigir a la página del portal de **Connection Monitor** y ver la topología de salto a salto y los problemas identificados que afectan a la conectividad. Haga clic en **Alertas** para redirigir a las alertas y en **Checks Failed Percent/Round-Trip Time** (Porcentaje de comprobaciones con error/tiempo de ida y vuelta) para redirigir a la página de métricas del monitor de conexión seleccionado.

La cuadrícula  **Alertas**  de la derecha proporciona una vista de todas las alertas generadas para las pruebas de conectividad configuradas en todas las suscripciones. Haga clic en el número de alertas para ir a la página de alertas detalladas.

## <a name="traffic"></a><a name="traffic"></a>Tráfico
La pestaña Tráfico proporciona acceso a todos los grupos de seguridad de red configurados para [Registros de flujo de NSG](../../network-watcher/network-watcher-nsg-flow-logging-overview.md) y [Análisis de tráfico](../../network-watcher/traffic-analytics.md) para el conjunto seleccionado de suscripciones y agrupados por ubicaciones. La funcionalidad de búsqueda proporcionada en esta pestaña permite identificar los grupos de seguridad de red configurados para la dirección IP buscada. Puede buscar cualquier dirección IP de su entorno, y en la vista regional en mosaico se mostrarán todos los grupos de seguridad de red junto con los registros de flujos de NSG y el estado de configuración de Análisis de tráfico.

![Vista del tráfico en Azure Monitor para redes](media/network-insights-overview/azure-monitor-for-networks-traffic-view.png)

Al hacer clic en cualquier icono de región, se abre una vista de cuadrícula que facilita la visualización y configuración de registros de flujos de NSG y Análisis de tráfico.  

![Vista de la región del tráfico en Azure Monitor para redes](media/network-insights-overview/azure-monitor-for-networks-traffic-region-view.png)

Puede hacer clic en cada elemento de la vista de cuadrícula. Haga clic en el estado de configuración para editar el registro de flujos de NSG y la configuración de Análisis de tráfico. Haga clic en Alertas para redirigir a las alertas de tráfico configuradas para el grupo de seguridad de red seleccionado. Del mismo modo, puede ir a la vista de Análisis de tráfico haciendo clic en el área de trabajo.  

La cuadrícula  **Alertas**  de la derecha proporciona una vista de todas las alertas basadas en el área de trabajo de Análisis de tráfico de todas las suscripciones. Haga clic en el número de alertas para ir a la página de alertas detalladas.

## <a name="diagnostic-toolkit"></a><a name="diagnostictoolkit"></a> Kit de herramientas de diagnóstico
El kit de herramientas de diagnóstico proporciona acceso a todas las características de diagnóstico disponibles para solucionar problemas de la red. En esta lista desplegable se obtiene acceso a características como [Captura de paquetes](../../network-watcher/network-watcher-packet-capture-overview.md), [Solución de problemas de VPN](../../network-watcher/network-watcher-troubleshoot-overview.md), [Solución de problemas de conexión](../../network-watcher/network-watcher-connectivity-overview.md), [Próximo salto](../../network-watcher/network-watcher-next-hop-overview.md) y [Comprobación del flujo de IP](../../network-watcher/network-watcher-ip-flow-verify-overview.md).

![Pestaña del kit de herramientas de diagnóstico](media/network-insights-overview/azure-monitor-for-networks-diagnostic-toolkit.png)

## <a name="next-steps"></a>Pasos siguientes

- Más información sobre la supervisión de red en [¿Qué es Azure Network Watcher?](../../network-watcher/network-watcher-monitoring-overview.md)
