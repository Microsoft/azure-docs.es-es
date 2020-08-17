---
title: Azure Monitor for Networks (versión preliminar)
description: Información general rápida de Azure Monitor for Networks que ofrece una vista completa del estado y las métricas de todos los recursos de red implementados sin ninguna configuración.
ms.subservice: ''
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 11/07/2019
ms.openlocfilehash: ccc5578944dc6eea9a62360045272896d78e2fac
ms.sourcegitcommit: 0e8a4671aa3f5a9a54231fea48bcfb432a1e528c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/24/2020
ms.locfileid: "87128682"
---
# <a name="azure-monitor-for-networks-preview"></a>Azure Monitor for Networks (versión preliminar)
Azure Monitor for Networks ofrece una vista completa del estado y las métricas de todos los recursos de red implementados sin ninguna configuración. La capacidad de búsqueda avanzada ayuda a identificar dependencias de recursos, lo que habilita escenarios tales como la identificación de recursos que hospedan el sitio web con una simple búsqueda del nombre del sitio web hospedado.

La página de **información general** de Azure Monitor for Networks ofrece una manera fácil de visualizar el inventario de los recursos de red junto con las alertas y el estado de los recursos. Se divide en cuatro áreas funcionales clave:

- Búsqueda y filtrado
- Resource Health y métricas
- Alertas 
- Vista de dependencias

![Página de información general](media/network-insights-overview/overview.png)

## <a name="search-and-filtering"></a>Búsqueda y filtrado
La vista de Resource Health y alertas se puede personalizar con filtros como **Suscripción**, **Grupo de recursos** y **Tipo de recurso**. El cuadro de búsqueda proporciona la capacidad de buscar en las propiedades de los recursos.

El cuadro de búsqueda se puede usar para buscar recursos y recursos asociados. Por ejemplo, una dirección IP pública está asociada a una instancia de Application Gateway. La búsqueda del nombre DNS de las direcciones IP públicas identificará tanto la dirección IP pública como la instancia de Application Gateway asociada.

![Search](media/network-insights-overview/search.png)


## <a name="resource-health-and-metric"></a>Resource Health y métrica
Cada icono representa un tipo de recurso, con el número de instancias implementadas en todas las suscripciones seleccionadas junto con el estado de Resource Health. En el ejemplo siguiente, hay 105 conexiones de ER y VPN implementadas, 103 son correctas y 2 no están disponibles.

![Estado de los recursos](media/network-insights-overview/resource-health.png)

Al hacer clic en las dos conexiones de ER y VPN no disponibles, se inicia una vista de métrica. 

![Vista de métrica](media/network-insights-overview/metric-view.png)

Puede hacer clic en cada elemento de la vista de cuadrícula. Haga clic en el icono de estado para redirigir a Resource Health para esa conexión. Haga clic en Alertas para redirigir a la página de alertas y métricas, respectivamente, para esa conexión. 

## <a name="alerts"></a>Alertas
La cuadrícula **Alertas** de la derecha ofrece una vista de todas las alertas generadas para los recursos seleccionados en todas las suscripciones. Haga clic en el número de alertas para ir a la página de alertas detalladas.

## <a name="dependency-view"></a>Vista de dependencias
La vista **Dependencia** ayuda a visualizar cómo se configura el recurso. Actualmente se admite la vista Dependencias para Application Gateway, Virtual WAN y Load Balancer. Por ejemplo, en el caso de Application Gateway, se puede acceder a la vista Dependencias al hacer clic en el nombre del recurso de Application Gateway desde la vista de cuadrícula de métricas. Esto también se aplica a Virtual WAN y Load Balancer. 

![Vista Application Gateway](media/network-insights-overview/application-gateway.png)

La vista **Dependencia** de Application Gateway ofrece una vista simplificada de cómo se conectan las direcciones IP de front-end a los agentes de escucha, las reglas y el grupo de back-end. Los bordes de conexión están codificados por colores y ofrecen detalles adicionales según el estado del grupo de back-end. La vista también ofrece una vista detallada de las métricas de Application Gateway y las métricas de todos los grupos de back-end relacionados, como las instancias de VMSS y VM.

![Vista de dependencias](media/network-insights-overview/dependency-view.png)

El gráfico de dependencias facilita la navegación a las opciones de configuración. Haga clic con el botón derecho en un grupo de back-end para acceder a otras funciones. Por ejemplo, si el grupo de back-end es una máquina virtual, puede acceder directamente a la información de la máquina virtual y a la solución de problemas de conexión de Network Watcher para identificar problemas de conectividad.

![Menú de la vista de dependencias](media/network-insights-overview/dependency-view-menu.png)

La barra de búsqueda y de filtro de la vista de dependencias ofrece una forma sencilla de buscar en el gráfico. Por ejemplo, la búsqueda de *AppGWTestRule* en el ejemplo siguiente limitará la vista gráfica a todos los nodos conectados a través de *AppGWTestRule*. 

![Ejemplo de búsqueda](media/network-insights-overview/search-example.png)

Los diferentes filtros proporcionan ayuda para reducir la búsqueda a una ruta y un estado específicos. Por ejemplo, seleccione solo *Incorrecto* en el menú desplegable **Estado de mantenimiento** para mostrar todos los bordes en los que el estado es *Incorrecto*.

Haga clic en **vista de métricas detalladas** para iniciar un libro preconfigurado con métricas detalladas para la puerta de enlace de aplicaciones, todos los recursos del grupo de back-end y las direcciones IP de front-end. 

## <a name="next-steps"></a>Pasos siguientes 

- Obtenga más información sobre la supervisión de red en [¿Qué es Azure Network Watcher?](../../network-watcher/network-watcher-monitoring-overview.md)
