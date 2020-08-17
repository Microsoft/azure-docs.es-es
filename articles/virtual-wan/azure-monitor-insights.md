---
title: Supervisión de Azure Virtual WAN con Azure Monitor Insights
description: Más información sobre la supervisión de Virtual WAN con Azure Monitor Insights
services: virtual-wan
author: cherylmc
ms.service: virtual-wan
ms.topic: conceptual
ms.date: 07/22/2020
ms.author: cherylmc
ms.openlocfilehash: 8553a809173d955a21e6730de35c70de5b69e81b
ms.sourcegitcommit: 0e8a4671aa3f5a9a54231fea48bcfb432a1e528c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/24/2020
ms.locfileid: "87136026"
---
# <a name="azure-monitor-insights-for-virtual-wan-preview"></a>Azure Monitor Insights para Virtual WAN (versión preliminar)

[Azure Monitor Insights](../azure-monitor/insights/network-insights-overview.md) para Virtual WAN ofrece a los usuarios y operadores la posibilidad de ver el estado de Virtual WAN, presentado a través de un mapa topológico de detección automática. El estado de los recursos se superpone en el mapa para proporcionar una vista de instantánea del estado general de Virtual WAN. La navegación de recursos está habilitada en el mapa mediante el acceso con un clic a las páginas de configuración de recursos del portal de Virtual WAN.

Las métricas de nivel de recurso de Virtual WAN se recopilan y presentan mediante un libro de métricas de Virtual WAN previamente empaquetado que muestra las métricas en los niveles de una WAN virtual, centro, puerta de enlace y conexión. Este artículo le guiará por los pasos para usar Azure Monitor Insights para Virtual WAN, a fin de ver la topología de Virtual WAN y las métricas en un solo lugar.

> [!NOTE]
> El menú Insights está en proceso de implementación para esta versión preliminar.
>

## <a name="before-you-begin"></a>Antes de empezar

En los pasos de este artículo se da por hecho que ya ha implementado una WAN virtual con uno o más centros. Para crear una nueva WAN virtual y un nuevo centro, siga los pasos que se describen en los siguientes artículos:

* [Creación de una instancia de Virtual WAN](virtual-wan-site-to-site-portal.md#openvwan)
* [Creación de un centro de conectividad](virtual-wan-site-to-site-portal.md#hub)

## <a name="view-vwan-topology"></a><a name="topology"></a>Visualización de la topología de VWAN

En **Azure Portal ->Virtual WAN**, en el menú **Monitor** de la izquierda, seleccione **Insights (versión preliminar)** . Se abre la **vista Insights**, que muestra el mapa de dependencias de Virtual WAN y el minilibro de métricas de alto nivel.

**Figura 1: menú Insights de Monitor**

:::image type="content" source="./media/azure-monitor-insights/monitor-menu.png" alt-text="figura" lightbox="./media/azure-monitor-insights/monitor-menu.png":::

En la vista **Insights**, puede ver los recursos de Virtual WAN de detección automática, como centros, puertas de enlace, firewalls, conexiones y redes virtuales radiales, NVA de terceros y ramas de una instancia de Virtual WAN de un extremo a otro, tal como se muestra en la **Figura 2**.

El **estado de los recursos** y el **estado** general están codificados por colores y superpuestos en los iconos de los recursos del mapa. Las métricas de alto nivel de Virtual WAN, como las capacidades de los centros y el uso de la puerta de enlace, se muestran a la derecha a través de un minilibro.

**Figura 2: vista Insights**

:::image type="content" source="./media/azure-monitor-insights/insights-view.png" alt-text="figura" lightbox="./media/azure-monitor-insights/insights-view.png":::

## <a name="dependency-view"></a><a name="dependency"></a>Vista de dependencias

La vista de **Dependencias** de Virtual WAN ayuda a visualizar la vista interconectada de todos los recursos de Virtual WAN organizados principalmente en una arquitectura tipo hub-and-spoke.

**Figura 3: vista de dependencias de VWAN**

:::image type="content" source="./media/azure-monitor-insights/dependency-map.png" alt-text="Mapa de dependencias" lightbox="./media/azure-monitor-insights/dependency-map.png":::

En el mapa de la vista de Dependencias se muestran los siguientes recursos como un gráfico conectado:

* Centros de Virtual WAN en varias regiones de Azure.
* Redes virtuales radiales que están conectadas directamente al centro.
* Sitios de ramas de VPN y ExpressRoute y usuarios de P2S que están conectados a cada centro a través de sus respectivas conexiones de ExpressRoute, S2S y P2S y puertas de enlace de red virtual.
* Azure Firewall (incluidos los proxies de firewall de terceros) implementado en un centro (centro protegido).
* NVA de terceros (aplicaciones virtuales de red) que se implementan en redes virtuales radiales.

El mapa de Dependencias también muestra redes virtuales conectadas indirectamente (VNet que están emparejadas con redes virtuales radiales de Virtual WAN).

El mapa de Dependencias facilita la navegación a las opciones de configuración de cada recurso. Por ejemplo, puede mantener el mouse sobre el recurso del centro para ver la configuración básica de los recursos, como la región y el prefijo del centro. Haga clic con el botón derecho para acceder a la página de Azure Portal del recurso del centro.

**Ilustración 4: Navegar a la información específica del recurso**

:::image type="content" source="./media/azure-monitor-insights/resource-information.png" alt-text="Información de recursos":::

La barra de búsqueda y de filtro de la vista de Dependencias ofrece una forma sencilla de buscar en el gráfico. Los diferentes filtros proporcionan ayuda para restringir la búsqueda a una ruta y un estado específicos.

**Ilustración 5: Búsqueda y filtrado**

:::image type="content" source="./media/azure-monitor-insights/search-filter.png" alt-text="barra de búsqueda y filtros" lightbox="./media/azure-monitor-insights/search-filter.png":::

## <a name="detailed-metrics"></a><a name="detailed"></a>Métricas detalladas

Puede seleccionar **Ver métricas detalladas** para acceder a la página de **Métricas** detalladas. La página de métricas es un panel que está preconfigurado con pestañas independientes que proporcionan información útil sobre la capacidad, el rendimiento y la utilización de los recursos de Virtual WAN en el nivel de WAN virtual, el nivel de centros y las conexiones individuales.

**Ilustración 6: Panel de métricas detalladas**

:::image type="content" source="./media/azure-monitor-insights/detailed-metrics.png" alt-text="métricas detalladas" lightbox="./media/azure-monitor-insights/detailed-metrics.png":::

## <a name="next-steps"></a>Pasos siguientes

* Para más información sobre las métricas en Azure Monitor, consulte [Métricas en Azure monitor](../azure-monitor/platform/data-platform-metrics.md).
* Para obtener una descripción completa de todas las métricas de Virtual WAN, vea [Registros y métricas de Azure Virtual WAN](logs-metrics.md).
