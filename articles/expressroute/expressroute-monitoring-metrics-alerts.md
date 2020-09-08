---
title: 'Azure ExpressRoute: Supervisión, métricas y alertas'
description: Obtenga información sobre la supervisión, las métricas y las alertas de Azure ExpressRoute con Azure Monitor, la ventanilla única para todas las métricas, alertas y registros de diagnóstico en Azure.
services: expressroute
author: duongau
ms.service: expressroute
ms.topic: how-to
ms.date: 08/25/2020
ms.author: duau
ms.openlocfilehash: fc83e5e8d14250ed163a56830311533144bbe344
ms.sourcegitcommit: 5a3b9f35d47355d026ee39d398c614ca4dae51c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89395441"
---
# <a name="expressroute-monitoring-metrics-and-alerts"></a>Supervisión, métricas y alertas de ExpressRoute

Este artículo le servirá para comprender la supervisión, las métricas y las alertas de ExpressRoute con Azure Monitor. Azure Monitor es un centro único para todas las métricas, alertas y registros de diagnóstico en todo Azure.
 
>[!NOTE]
>No se recomienda el uso de **métricas clásicas**.
>

## <a name="expressroute-metrics"></a>Métricas de ExpressRoute

Para ver **Métricas**, vaya a la página *Azure Monitor* y haga clic en *Métricas*. Para ver las métricas de **ExpressRoute**, filtre por el tipo de recurso *Circuitos de ExpressRoute*. Para ver las métricas de **Global Reach**, filtre por el tipo de recurso *Circuitos de ExpressRoute* y seleccione un recurso de circuito de ExpressRoute que tenga Global Reach habilitado. Para ver las métricas **ExpressRoute Direct** , filtre el Tipo de recurso por *Puertos ExpressRoute*. 

Una vez seleccionada una métrica, se aplicará la agregación predeterminada. Opcionalmente, puede aplicar la división, que mostrará la métrica con diferentes dimensiones.

### <a name="available-metrics"></a>Métricas disponibles

|**Métrica**|**Categoría**|**Dimensiones**|**Características**|
| --- | --- | --- | --- |
|Disponibilidad de ARP|Disponibilidad|<ui><li>Par (enrutador de ExpressRoute principal y secundario)</ui></li><ui><li> Tipo de emparejamiento (privado, público o Microsoft)</ui></li>|ExpressRoute|
|Disponibilidad de BGP|Disponibilidad|<ui><li> Par (enrutador de ExpressRoute principal y secundario)</ui></li><ui><li> Tipo de emparejamiento</ui></li>|ExpressRoute|
|BitsInPerSecond|Tráfico|<ui><li> Tipo de emparejamiento (ExpressRoute)</ui></li><ui><li>Vínculo (ExpressRoute Direct)</ui></li>|<li>ExpressRoute</li><li>ExpressRoute Direct|
|BitsOutPerSecond|Tráfico| <ui><li>Tipo de emparejamiento (ExpressRoute)</ui></li><ui><li> Vínculo (ExpressRoute Direct) |<ui><li>ExpressRoute<ui><li>ExpressRoute Direct</ui></li> |
|Uso de CPU|Rendimiento| <ui><li>Instancia</ui></li>|Puerta de enlace de red virtual de ExpressRoute|
|Paquetes por segundo|Rendimiento| <ui><li>Instancia</ui></li>|Puerta de enlace de red virtual de ExpressRoute.|
|GlobalReachBitsInPerSecond|Tráfico|<ui><li>Skey de circuito emparejado (clave de servicio)</ui></li>|Global Reach|
|GlobalReachBitsOutPerSecond|Tráfico|<ui><li>Skey de circuito emparejado (clave de servicio)</ui></li>|Global Reach|
|AdminState|Conectividad física|Vínculo|ExpressRoute Direct|
|LineProtocol|Conectividad física|Vínculo|ExpressRoute Direct|
|RxLightLevel|Conectividad física|<ui><li>Vínculo</ui></li><ui><li>Carril</ui></li>|ExpressRoute Direct|
|TxLightLevel|Conectividad física|<ui><li>Vínculo</ui></li><ui><li>Carril</ui></li>|ExpressRoute Direct|
>[!NOTE]
>El uso de *GlobalGlobalReachBitsInPerSecond* y *GlobalGlobalReachBitsOutPerSecond* solo será visible si se ha establecido al menos una conexión Global Reach.
>

## <a name="circuits-metrics"></a>Métricas de circuito

### <a name="bits-in-and-out---metrics-across-all-peerings"></a>Bits dentro y fuera: métricas de todos los emparejamientos

Puede ver las métricas de todos los emparejamientos de un circuito ExpressRoute determinado.

:::image type="content" source="./media/expressroute-monitoring-metrics-alerts/ermetricspeering.jpg" alt-text="métricas de circuito":::

### <a name="bits-in-and-out---metrics-per-peering"></a>Bits dentro y fuera: métricas por emparejamiento

Puede ver las métricas de emparejamiento público, privado y de Microsoft en bits por segundo.

:::image type="content" source="./media/expressroute-monitoring-metrics-alerts/erpeeringmetrics.jpg" alt-text="métricas por emparejamiento":::

### <a name="bgp-availability---split-by-peer"></a>Disponibilidad de BGP: división por par  

Puede ver la disponibilidad casi en tiempo real de BGP entre emparejamientos y pares (enrutadores de ExpressRoute principales y secundarios). En este panel se muestra la sesión de BGP principal arriba para el emparejamiento privado y la segunda sesión BGP abajo para el emparejamiento privado. 

:::image type="content" source="./media/expressroute-monitoring-metrics-alerts/erBgpAvailabilityMetrics.jpg" alt-text="Disponibilidad de BGP por par":::

### <a name="arp-availability---split-by-peering"></a>Disponibilidad de ARP: división por emparejamiento  

Puede ver la disponibilidad casi en tiempo real de [ARP](https://docs.microsoft.com/azure/expressroute/expressroute-troubleshooting-arp-resource-manager) entre emparejamientos y pares (enrutadores de ExpressRoute principales y secundarios). En este panel se muestra la sesión de ARP de emparejamiento privado arriba entre ambos pares, y completa abajo para el emparejamientos de Microsoft entre emparejamientos. La agregación predeterminada (promedio) se usó en ambos pares.  

:::image type="content" source="./media/expressroute-monitoring-metrics-alerts/erArpAvailabilityMetrics.jpg" alt-text="Disponibilidad de ARP por par":::

## <a name="expressroute-direct-metrics"></a>Métricas directas de ExpressRoute

### <a name="admin-state---split-by-link"></a>Estado de administración: dividido por vínculo

Puede ver el estado del administrador para cada enlace del par de puertos directos de ExpressRoute.

:::image type="content" source="./media/expressroute-monitoring-metrics-alerts/adminstate-per-link.jpg" alt-text="Estado de administrador ER directo":::

### <a name="bits-in-per-second---split-by-link"></a>Bits por segundo: divido por vínculo

Puede ver los bits por segundo en ambos enlaces del par de puertos directos de ExpressRoute.

:::image type="content" source="./media/expressroute-monitoring-metrics-alerts/bits-in-per-second-per-link.jpg" alt-text="ER bits directos por segundo":::

### <a name="bits-out-per-second---split-by-link"></a>Bits de salida por segundo: divido por vínculo

También puede ver los bits por segundo en ambos enlaces del par de puertos directos de ExpressRoute.

:::image type="content" source="./media/expressroute-monitoring-metrics-alerts/bits-out-per-second-per-link.jpg" alt-text="ER bits directos por segundo":::

### <a name="line-protocol---split-by-link"></a>Protocolo de línea: división por vínculo

Puede ver el protocolo de línea en cada enlace del par de puertos directos de ExpressRoute.

:::image type="content" source="./media/expressroute-monitoring-metrics-alerts/line-protocol-per-link.jpg" alt-text="Protocolo ER de línea directa":::

### <a name="rx-light-level---split-by-link"></a>Nivel de luz de RX: división por vínculo

Puede ver el nivel de luz de Rx (el nivel de luz que el puerto directo de ExpressRoute está **recibiendo**) para cada puerto. Los niveles de luz Rx saludables generalmente caen dentro de un rango de -10 a 0 dBm

:::image type="content" source="./media/expressroute-monitoring-metrics-alerts/rxlight-level-per-link.jpg" alt-text="Nivel de luz de recepción de línea directa de ER":::

### <a name="tx-light-level---split-by-link"></a>Nivel de luz de RX: división por vínculo

Puede ver el nivel de luz Tx (el nivel de luz que el puerto directo de ExpressRoute **transmite**) para cada puerto. Los niveles de luz Tx saludables generalmente caen dentro de un rango de -10 a 0 dBm

:::image type="content" source="./media/expressroute-monitoring-metrics-alerts/txlight-level-per-link.jpg" alt-text="Nivel de luz de recepción de línea directa de ER":::

## <a name="expressroute-virtual-network-gateway-metrics"></a>Métricas de puerta de enlace de red virtual de ExpressRoute

### <a name="cpu-utilization---split-instance"></a>Uso de CPU: división de instancia

Puede ver el uso de CPU de las instancias de puerta de enlace.

:::image type="content" source="./media/expressroute-monitoring-metrics-alerts/cpu-split.jpg" alt-text="División de CPU":::

### <a name="packets-per-second---split-by-instance"></a>Paquetes por segundo: división por instancia

Puede ver los paquetes por segundo que atraviesan la puerta de enlace.

:::image type="content" source="./media/expressroute-monitoring-metrics-alerts/pps-split.jpg" alt-text="Paquetes por segundo: división":::

## <a name="expressroute-gateway-connections-in-bitsseconds"></a>Conexiones de puerta de enlace de ExpressRoute en bits por segundo

:::image type="content" source="./media/expressroute-monitoring-metrics-alerts/erconnections.jpg" alt-text="conexiones de puerta de enlace":::

## <a name="alerts-for-expressroute-gateway-connections"></a>Alertas para las conexiones de puerta de enlace de ExpressRoute

1. Para configurar las alertas, vaya a **Azure Monitor** y, a continuación, seleccione **Alertas**.

   :::image type="content" source="./media/expressroute-monitoring-metrics-alerts/eralertshowto.jpg" alt-text="alerts":::
2. Haga clic en **+Seleccionar destino** y seleccione el recurso de conexión de puerta de enlace de ExpressRoute.

   :::image type="content" source="./media/expressroute-monitoring-metrics-alerts/alerthowto2.jpg" alt-text="Destino":::
3. Defina los detalles de la alerta.

   :::image type="content" source="./media/expressroute-monitoring-metrics-alerts/alerthowto3.jpg" alt-text="grupo de acciones":::
4. Defina y agregue el grupo de acciones.

   :::image type="content" source="./media/expressroute-monitoring-metrics-alerts/actiongroup.png" alt-text="adición de grupo de acciones":::

## <a name="alerts-based-on-each-peering"></a>Alertas basadas en cada emparejamiento

:::image type="content" source="./media/expressroute-monitoring-metrics-alerts/basedpeering.jpg" alt-text="cada emparejamiento":::

## <a name="configure-alerts-for-activity-logs-on-circuits"></a>Configuración de alertas para los registros de actividad en circuitos

En **Criterios de alerta**, puede seleccionar **Registro de actividad** para el tipo de señal y seleccionar la señal.

:::image type="content" source="./media/expressroute-monitoring-metrics-alerts/alertshowto6activitylog.jpg" alt-text="registros de actividad":::
  
## <a name="next-steps"></a>Pasos siguientes

Configure su conexión ExpressRoute.
  
* [Creación y modificación de un circuito](expressroute-howto-circuit-arm.md)
* [Creación y modificación de la configuración de emparejamiento](expressroute-howto-routing-arm.md)
* [Vinculación de redes virtuales a circuitos ExpressRoute](expressroute-howto-linkvnet-arm.md)
