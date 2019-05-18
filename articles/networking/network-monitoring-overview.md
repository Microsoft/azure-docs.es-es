---
title: Acerca de la supervisión de red en Azure Monitor registra | Microsoft Docs
description: Introducción a las soluciones de supervisión de redes, como es NPM, para administrar redes en entornos en la nube, locales e híbridos.
services: monitoring-and-diagnostics
documentationcenter: na
author: agummadi
manager: ''
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: monitoring-and-diagnostics
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 02/22/2018
ms.author: ajaycode
ms.openlocfilehash: b112771e8c0f08e6b9a02c2fc5b7e1887b3f2588
ms.sourcegitcommit: 3675daec6c6efa3f2d2bf65279e36ca06ecefb41
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/14/2019
ms.locfileid: "65620415"
---
# <a name="network-monitoring-solutions"></a>Soluciones de supervisión de redes 

Azure ofrece una gran cantidad de soluciones para supervisar los activos de una red. Azure ofrece soluciones y utilidades para supervisar la conectividad de red y el estado de los circuitos ExpressRoute, y para analizar el tráfico de red en la nube.

## <a name="network-performance-monitor-npm"></a>Network Performance Monitor (NPM) (monitor de rendimiento de red)

Network Performance Monitor (NPM) es un conjunto de funcionalidades, cada una orientada específicamente a la supervisión del estado de una red y a la conectividad de red para las aplicaciones, además de proporcionar información detallada sobre el rendimiento de una red. NPM está basado en la nube y proporciona una solución híbrida para la supervisión de la red que controla la conectividad entre:
 
* Las implementaciones en la nube y las ubicaciones locales
* Varios centros de datos y sucursales
* Aplicaciones y microservicios esenciales de niveles múltiples
* Aplicaciones basadas en web (HTTP/HTTPs) y ubicaciones de usuario 

Monitor de rendimiento, Supervisión de ExpressRoute y Monitor de conectividad de servicio son funcionalidades de supervisión dentro de NPM y se describen a continuación.

## <a name="performance-monitor"></a>Monitor de rendimiento

Performance Monitor es parte de NPM y supervisa la red en entornos en la nube, locales e híbridos. Puede supervisar la conectividad de red entre oficinas locales y sucursales remotas, tiendas, centros de datos y nubes. Puede detectar problemas de red antes de que los usuarios se quejen. Las principales ventajas son:

* Supervisar la pérdida y la latencia en varias subredes y establecer alertas
* Supervisar todas las rutas de acceso (incluidas las rutas de acceso redundantes) en la red
* Solucionar problemas de red transitorios y puntuales que resultan difíciles de replicar
* Determinar un segmento específico de la red que es responsable de la degradación del rendimiento
* Supervisar el estado de la red, sin necesidad de SNMP

![Mapa de topología NPM](./media/network-monitoring-overview/npm-topology-map.png) 

Para más información, consulte los siguientes artículos:

* [Configurar una solución Network Performance Monitor en registros de Azure Monitor](../azure-monitor/insights/network-performance-monitor.md) 
* [Casos de uso](https://blogs.technet.microsoft.com/msoms/2016/08/30/monitor-on-premises-cloud-iaas-and-hybrid-networks-using-oms-network-performance-monitor/)
* Actualizaciones del producto:
  * [Febrero de 2017](https://blogs.technet.microsoft.com/msoms/2017/02/27/oms-network-performance-monitor-is-now-generally-available/)
  * [Agosto de 2017](https://blogs.technet.microsoft.com/msoms/2017/08/14/improvements-to-oms-network-performance-monitor/)

## <a name="expressroute-monitor"></a>Supervisión de ExpressRoute

NPM para ExpressRoute ofrece una supervisión completa de ExpressRoute para conexiones de emparejamiento de Microsoft y conexiones de emparejamiento privadas de Azure. Puede supervisar la conectividad E2E y el rendimiento entre las sucursales y Azure a través de ExpressRoute. Las funcionalidades principales son:

* Detección automática de circuitos ER asociados a una suscripción
* Detección de la topología de red del entorno local a las aplicaciones en la nube
* Planeamiento de la capacidad, análisis de uso de ancho de banda
* Supervisión y alertas relacionadas con las rutas de acceso principales y secundarias
* Supervisión de la conectividad a los servicios de Azure, como Office 365, Dynamics 365, etc., mediante ExpressRoute
* Detección del deterioro de la conectividad con redes virtuales

![Mapa geográfico que muestra el tráfico entre regiones](./media/network-monitoring-overview/expressroute-topology-map.png) 

Para más información, consulte los siguientes artículos.

* [Configuración de Network Performance Monitor para ExpressRoute](../expressroute/how-to-npm.md)
* [Entradas del blog](https://aka.ms/NPMExRmonitorGA)

## <a name="service-connectivity-monitor"></a>Monitor de conectividad de servicio

Con el Monitor de conectividad de servicio, ahora puede probar la accesibilidad de las aplicaciones y detectar cuellos de botella en el rendimiento de las redes locales, redes de operadores y centros de datos en la nube y privados.

* Supervisar de un extremo a otro la conectividad de red con las aplicaciones
* Poner en correlación la entrega de las aplicaciones con el rendimiento de la red, detectar la ubicación exacta de la degradación a lo largo de la ruta entre el usuario y la aplicación
* Probar la accesibilidad de las aplicaciones desde varias ubicaciones de usuario en todo el mundo
* Determinar la pérdida de paquetes y la latencia de red para la línea de negocio y las aplicaciones SaaS
* Determinar las zonas activas en la red, que puedan estar causando el rendimiento insuficiente de las aplicaciones
* Supervisar la disponibilidad para las aplicaciones de Office 365, con pruebas integradas de Microsoft Office 365, Dynamics 365, Skype Empresarial y otros servicios de Microsoft

Para más información, consulte los siguientes artículos.

* [Configurar Network Performance Monitor para la supervisión de puntos de conexión de servicio](../azure-monitor/insights/network-performance-monitor-service-connectivity.md#configuration)
* [Entradas del blog](https://aka.ms/svcendptmonitor)

## <a name="traffic-analytics"></a>Análisis de tráfico
Análisis de tráfico es una solución basada en la nube que proporciona visibilidad sobre la actividad de usuarios y aplicaciones en las redes en la nube. Los registros del flujo de NSG se analizan para proporcionar información sobre:

* El flujo de tráfico a través de las redes entre Azure e Internet, las regiones de la nube pública, las redes virtuales y las subredes.
* Las aplicaciones y los protocolos de la red, sin necesidad de rastreadores o aparatos de recopilación de flujo dedicados.
* Top Talkers, aplicaciones comunicativas, conversaciones de máquinas virtuales en la nube, zonas activas de tráfico.
* Orígenes y destinos del tráfico entre redes virtuales, interrelaciones entre servicios y aplicaciones críticos para el negocio.
* Seguridad: tráfico malintencionado, puertos abiertos a Internet, aplicaciones o máquinas virtuales que intentan acceder a Internet…
* Utilización de la capacidad: le ayuda a eliminar los problemas de aprovisionamiento en exceso o infrautilización al supervisarse las tendencias de uso de las puertas de enlace de VPN y otros servicios.

Análisis de tráfico le proporciona información práctica que le ayuda a auditar la actividad de red de la organización, a proteger las aplicaciones y los datos, a optimizar el rendimiento de la carga de trabajo y a satisfacer los requisitos de cumplimiento.

![Mapa geográfico que muestra el tráfico entre regiones](../network-watcher/media/traffic-analytics/geo-map-view-showcasing-traffic-distribution-to-countries-and-continents.png) 

Vínculos relacionados:
* [Entrada de blog](https://aka.ms/trafficanalytics), [Documentación](https://aka.ms/trafficanalyticsdocs), [Preguntas frecuentes](https://docs.microsoft.com/azure/network-watcher/traffic-analytics-faq)

## <a name="dns-analytics"></a>DNS Analytics
Creada para los administradores de DNS, esta solución recopila, analiza y correlaciona registros DNS para proporcionar información detallada relacionada con la seguridad, las operaciones y el rendimiento.  Algunas de las funcionalidades son:

* Identificación de los clientes que intentan resolver dominios malintencionados.
* Identificación de registros de recursos obsoletos.
* Visibilidad sobre los nombres de dominio consultados con más frecuencia y los clientes DNS participativos.
* Visibilidad sobre la carga de solicitudes en servidores DNS.
* Supervisión de errores de registro DNS dinámicos.

![Panel de DNS Analytics](./media/network-monitoring-overview/dns-analytics-overview.png) 

Vínculos relacionados:
* [Entrada de blog](https://blogs.technet.microsoft.com/msoms/2017/04/19/introducing-oms-dns-analytics/), [Documentación](https://docs.microsoft.com/azure/log-analytics/log-analytics-dns)

## <a name="miscellaneous"></a>Miscelánea

* [Nuevos precios](https://docs.microsoft.com/azure/log-analytics/log-analytics-network-performance-monitor-pricing-faq)
