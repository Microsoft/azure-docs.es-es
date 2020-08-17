---
title: 'Supervisión de aplicaciones en Azure Kubernetes Service (AKS) con Application Insights: Azure Monitor | Microsoft Docs'
description: Azure Monitor se integra sin problemas con las aplicaciones que se ejecutan en Kubernetes y permite detectar problemas con las aplicaciones al instante.
ms.topic: conceptual
author: MS-jgol
ms.author: jgol
ms.date: 05/13/2020
ms.openlocfilehash: 52bd6d2a98e5126ff2463de1ef99da03ca555567
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87075312"
---
# <a name="zero-instrumentation-application-monitoring-for-kubernetes---azure-monitor-application-insights"></a>Supervisión de aplicaciones sin instrumentación para Kubernetes: Application Insights de Azure Monitor

> [!IMPORTANT]
>  Actualmente, puede habilitar la supervisión de las aplicaciones Java que se ejecutan en Kubernetes sin instrumentar el código: use el [agente independiente de Java](./java-in-process-agent.md). Aunque la solución para habilitar sin problemas la supervisión de aplicaciones está en desarrollo para otros lenguajes, puede usar los SDK para supervisar las aplicaciones que se ejecutan en AKS: [ASP.NET Core](./asp-net-core.md), [ASP.NET](./asp-net.md), [Node.js](./nodejs.md), [JavaScript](./javascript.md) y [Python](./opencensus-python.md).

## <a name="application-monitoring-without-instrumenting-the-code"></a>Supervisión de aplicaciones sin instrumentar el código
Actualmente, solo Java permite habilitar la supervisión de aplicaciones sin instrumentar el código. Para supervisar aplicaciones en otros lenguajes, use los SDK. 

## <a name="java"></a>Java
Una vez habilitado, el agente de Java recopilará automáticamente una gran cantidad de solicitudes, dependencias, registros y métricas de las bibliotecas y los marcos de trabajo más usados.

Siga [las instrucciones detalladas](./java-in-process-agent.md) para supervisar las aplicaciones Java que se ejecutan en aplicaciones de Kubernetes, así como otros entornos. 

## <a name="other-languages"></a>Otros idiomas

En el caso de aplicaciones en otros lenguajes, actualmente se recomienda usar los SDK:
* [ASP.NET Core](./asp-net-core.md)
* [ASP.NET](./asp-net.md)
* [Node.js](./nodejs.md) 
* [JavaScript](./javascript.md)
* [Python](./opencensus-python.md)

## <a name="next-steps"></a>Pasos siguientes

* Más información sobre [Azure Monitor](../overview.md) y [Application Insights](./app-insights-overview.md).
* Consulte una introducción al [seguimiento distribuido](./distributed-tracing.md) y descubra lo que el [Mapa de aplicación](./app-map.md?tabs=net) puede hacer por su empresa.
