---
title: Visualización de métricas de Azure VPN Gateway
description: Pasos para la visualización de métricas de VPN Gateway
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: how-to
ms.date: 09/03/2020
ms.author: alzam
ms.openlocfilehash: 4b33af3e64726e124373f57920836bce145cd891
ms.sourcegitcommit: bf1340bb706cf31bb002128e272b8322f37d53dd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/03/2020
ms.locfileid: "89443184"
---
# <a name="view-vpn-gateway-metrics"></a>Visualización de métricas de VPN Gateway

Puede supervisar puertas de enlace de VPN de Azure mediante Azure Monitor. En este artículo se describen las métricas que están disponibles en el portal. Las métricas son ligeras y pueden admitir escenarios casi en tiempo real, lo que las hace útiles para alertas y detección rápida de problemas.


|**Métrica**   | **Unidad** | **Granularidad** | **Descripción** | 
|---       | ---        | ---       | ---            | ---       |
|**AverageBandwidth**| Bytes por segundo  | 5 minutos| Promedio de uso de ancho de banda combinado de todas las conexiones de sitio a sitio en la puerta de enlace.     |
|**P2SBandwidth**| Bytes por segundo  | 1 minuto.  | Promedio de uso de ancho de banda combinado de todas las conexiones de punto a sitio en la puerta de enlace.    |
|**P2SConnectionCount**| Count  | 1 minuto.  | Recuento de las conexiones de punto a sitio en la puerta de enlace.   |
|**TunnelAverageBandwidth** | Bytes por segundo    | 5 minutos  | Promedio de utilización del ancho de banda de los túneles creados en la puerta de enlace. |
|**TunnelEgressBytes** | Bytes | 5 minutos | Tráfico saliente en los túneles creados en la puerta de enlace.   |
|**TunnelEgressPackets** | Count | 5 minutos | Recuento de los paquetes salientes en los túneles creados en la puerta de enlace.   |
|**TunnelEgressPacketDropTSMismatch** | Count | 5 minutos | Recuento de los paquetes salientes eliminados de los túneles por un error de coincidencia del selector de tráfico. |
|**TunnelIngressBytes** | Bytes | 5 minutos | Tráfico entrante en los túneles creados en la puerta de enlace.   |
|**TunnelIngressPackets** | Count | 5 minutos | Recuento de los paquetes entrantes en los túneles creados en la puerta de enlace.   |
|**TunnelIngressPacketDropTSMismatch** | Count | 5 minutos | Recuento de los paquetes entrantes eliminados de los túneles por un error de coincidencia del selector de tráfico. |

## <a name="the-following-steps-help-you-locate-and-view-metrics"></a>Los siguientes pasos le ayudarán a ubicar y ver las métricas:

1. En el portal, vaya al recurso de puerta de enlace de red virtual.
2. Seleccione **Información general** para ver las métricas de total de entrada y salida del túnel.

   ![Selecciones para crear una regla de alertas](./media/vpn-gateway-howto-view-virtual-network-gateway-metrics/overview.png "Ver")

3. Para ver cualquiera de las otras métricas indicadas anteriormente: Haga clic en la sección **Métricas** en el recurso de puerta de enlace de red virtual y seleccione la métrica en la lista desplegable.

   ![El botón Seleccionar y la instancia de VPN Gateway en la lista de recursos](./media/vpn-gateway-howto-view-virtual-network-gateway-metrics/metrics.png "Seleccionar")

## <a name="next-steps"></a>Pasos siguientes

Para configurar alertas en las métricas de túnel, vea [Configuración de alertas en métricas de VPN Gateway](vpn-gateway-howto-setup-alerts-virtual-network-gateway-metric.md).

Para configurar alertas en los registros de recursos de túnel, consulte [Configuración de alertas en los registros de diagnóstico de VPN Gateway](vpn-gateway-howto-setup-alerts-virtual-network-gateway-log.md).
