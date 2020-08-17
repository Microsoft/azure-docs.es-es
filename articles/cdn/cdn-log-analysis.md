---
title: Análisis de patrones de uso de Azure CDN
description: En este artículo se describen los diferentes tipos de informes de análisis disponibles para los productos de Azure CDN.
services: cdn
author: asudbring
manager: KumudD
ms.service: azure-cdn
ms.workload: tbd
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 7/22/2020
ms.author: allensu
ms.openlocfilehash: 169889dbb87d00fdde44ff72c0d2004c331604ab
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87073037"
---
# <a name="analyze-azure-cdn-usage-patterns"></a>Análisis de patrones de uso de Azure CDN

Después de habilitar CDN para la aplicación, puede supervisar el uso de la red CDN, comprobar el mantenimiento de su entrega y solucionar posibles problemas. Azure CDN proporciona estas funcionalidades de las siguientes maneras: 

## <a name="raw-logs-for-azure-cdn-from-microsoft"></a>Registros sin procesar para Azure CDN de Microsoft
Con un perfil estándar de Microsoft, puede habilitar los registros sin procesar y seleccionar que se transmitan los registros a:

* Azure Storage
* Centros de eventos
* Azure Log Analytics

Con Azure Log Analytics, puede ver las métricas de supervisión y configurar alertas. 

Para más información, consulte [Registros sin procesar de HTTP de Azure CDN](enable-raw-logs.md).


## <a name="core-analytics-via-azure-diagnostic-logs"></a>Análisis básicos a través de los registros de diagnóstico de Azure

El análisis esencial está disponible para los puntos de conexión de CDN de todos los planes de tarifa. Los registros de diagnóstico de Azure permiten exportar el análisis esencial a Azure Storage, Event Hubs o registros de Azure Monitor. Los registros de Azure Monitor ofrecen una solución con gráficos que el usuario puede configurar y personalizar. Para más información sobre los registros de diagnóstico de Azure, consulte [Registros de diagnóstico de Azure](cdn-azure-diagnostic-logs.md).

## <a name="verizon-core-reports"></a>Informes principales de Verizon

Los perfiles de **Azure CDN Estándar de Verizon** o de **Azure CDN Premium de Verizon** proporcionan informes principales. Puede ver los informes principales en el portal complementario de Verizon. Se puede acceder a los informes principales de Verizon con la opción **Administrar** de Azure Portal, que ofrece una variedad de gráficos y vistas. Para más información, vea [Informes principales de Verizon](cdn-analyze-usage-patterns.md).

## <a name="verizon-custom-reports"></a>Informes personalizados de Verizon

Los perfiles de **Azure CDN Estándar de Verizon** o de **Azure CDN Premium de Verizon** proporcionan informes personalizados. Puede ver informes personalizados en el portal complementario de Verizon. Se puede acceder a los informes personalizados de Verizon con la opción **Administrar** de Azure Portal. 

Los informes personalizados muestran el número de resultados o de datos transferidos para cada CNAME perimetral. Los datos se agrupan por código de respuesta HTTP o estado de caché durante un período de tiempo. Para más información, consulte [Informes personalizados de Verizon](cdn-verizon-custom-reports.md).

## <a name="azure-cdn-premium-from-verizon-reports"></a>Informes de Azure CDN Premium de Verizon

Con **Azure CDN premium de Verizon**, también puede tener acceso a los informes siguientes:
   * [Informes de HTTP avanzados](cdn-advanced-http-reports.md)
   * [Estadísticas en tiempo real](cdn-real-time-stats.md)
   * [Rendimiento del nodo perimetral de Azure CDN](cdn-edge-performance.md)

## <a name="next-steps"></a>Pasos siguientes
En este artículo, ha aprendido las diferentes opciones de los informes de análisis de Azure CDN.

Para más información sobre Azure CDN y los otros servicios de Azure que se mencionan en este artículo, consulte:

* [¿Qué es Azure CDN?](cdn-overview.md)
* [Registros sin procesar de HTTP de Azure CDN](enable-raw-logs.md)


