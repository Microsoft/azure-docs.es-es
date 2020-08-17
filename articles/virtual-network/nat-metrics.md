---
title: Métricas y alertas de Azure Virtual Network NAT
titleSuffix: Azure Virtual Network
description: Conozca las métricas y alertas de Azure Monitor disponibles para Virtual Network NAT.
services: virtual-network
documentationcenter: na
author: asudbring
manager: KumudD
ms.service: virtual-network
ms.subservice: nat
Customer intent: As an IT administrator, I want to understand available Azure Monitor metrics and alerts for Virtual Network NAT.
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 03/04/2020
ms.author: allensu
ms.openlocfilehash: e3c47a60a6cda074eba7b5c3292577c29f50c2ab
ms.sourcegitcommit: e71da24cc108efc2c194007f976f74dd596ab013
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87424058"
---
# <a name="azure-virtual-network-nat-metrics"></a>Métricas de Azure Virtual Network NAT

Los recursos de puerta de enlace de Azure Virtual Network NAT proporcionan métricas multidimensionales. Estas métricas se pueden usar para observar la operación y para la [solución de problemas](troubleshoot-nat.md).  Se pueden configurar alertas para problemas críticos, como el agotamiento de SNAT.

<p align="center">
  <img src="media/nat-overview/flow-direction1.svg" alt="Figure depicts a NAT gateway resource that consumes all IP addresses for a public IP prefix and directs that traffic to and from two subnets of virtual machines and a virtual machine scale set." width="256" title="Virtual Network NAT para la salida a Internet">
</p>

*Ilustración: Virtual Network NAT para la salida a Internet*

## <a name="metrics"></a>Métricas

Los recursos de puerta de enlace de NAT proporcionan las siguientes métricas multidimensionales en Azure Monitor:

| Métrica | Descripción | Agregación recomendada | Dimensions |
|---|---|---|---|
| Bytes | Bytes procesados de entrada y de salida | Sum | Dirección (entrada; salida), protocolo (6 TCP; 17 UDP) |
| Paquetes | Paquetes procesados de entrada y de salida | Sum | Dirección (entrada; salida), protocolo (6 TCP; 17 UDP) |
| Paquetes descartados | Paquetes descartados por la puerta de enlace de NAT | Sum | / |
| Recuento de conexiones SNAT | Transiciones de estado por intervalo | Sum | Estado de conexión, protocolo (6 TCP; 17 UDP) |
| Recuento de conexiones SNAT totales | Conexiones SNAT activas actuales (número aproximado de puertos SNAT en uso) | Sum | Protocolo (6 TCP; 17 UDP) |


## <a name="alerts"></a>Alertas

En Azure Monitor se pueden configurar alertas para cada una de las [métricas](#metrics) anteriores.

## <a name="limitations"></a>Limitaciones

Resource Health no es compatible.

## <a name="next-steps"></a>Pasos siguientes

* Obtenga más información sobre [Virtual Network NAT](nat-overview.md).
* Obtenga información sobre los [recursos de puerta de enlace de NAT](nat-gateway-resource.md)
* Obtenga información acerca de [Azure Monitor](../azure-monitor/overview.md)
* Obtenga información sobre la [solución de los problemas de los recursos de la puerta de enlace de NAT](troubleshoot-nat.md).
* [Indíquenos qué crear a continuación para Virtual Network NAT en UserVoice](https://aka.ms/natuservoice).


