---
title: Ejemplos de gráficos de métricas de Azure Monitor
description: Obtenga información sobre cómo visualizar los datos de Azure Monitor.
author: vgorbenko
services: azure-monitor
ms.topic: conceptual
ms.date: 01/29/2019
ms.author: vitalyg
ms.subservice: metrics
ms.openlocfilehash: 9b2ab664f319de07fd70bd1a22b1ba6d64ac208f
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87320262"
---
# <a name="metric-chart-examples"></a>Ejemplos de gráficos de métricas 

La plataforma Azure ofrece [más de mil métricas](./metrics-supported.md), muchas de ellas con dimensiones. Mediante el uso de [filtros de dimensión](./metrics-charts.md), la aplicación de [divisiones](./metrics-charts.md), el control del tipo de gráfico y el ajuste de la configuración del gráfico puede crear paneles y vistas de diagnóstico eficaces que proporcionan información detallada sobre el estado de la infraestructura y las aplicaciones. En este artículo se muestran algunos ejemplos de los gráficos que se pueden compilar mediante el [Explorador de métricas](./metrics-charts.md) y se explican los pasos necesarios para configurar cada uno de estos gráficos.

¿Quiere compartir sus estupendos ejemplos de gráficos con todo el mundo? Contribuya con esta página en GitHub y comparta sus propios ejemplos de gráfico aquí.

## <a name="website-cpu-utilization-by-server-instances"></a>Uso de CPU del sitio web en instancias del servidor

Este gráfico muestra si la CPU de una instancia de App Service estaba dentro del rango aceptable y la desglosa por instancia para determinar si la carga se distribuyó correctamente. Puede ver en el gráfico que la aplicación se estaba ejecutando en una única instancia del servidor antes de las 6 a.m. y, a continuación, se escaló verticalmente mediante la adición de otra instancia.

![Porcentaje promedio de CPU del gráfico de líneas por instancia del servidor](./media/metric-chart-samples/cpu-by-instance.png)

### <a name="how-to-configure-this-chart"></a>Cómo configurar este gráfico

Seleccione el recurso de App Service y busque la métrica de **porcentaje de CPU**. A continuación, haga clic en **Aplicar división** y seleccione la dimensión **Instancia**.

## <a name="application-availability-by-region"></a>Disponibilidad de la aplicación por región

Consulte la disponibilidad de la aplicación por región para identificar qué ubicaciones geográficas tienen problemas. Este gráfico muestra la métrica de disponibilidad de Application Insights. Puede ver que la aplicación supervisada no tiene ningún problema con la disponibilidad desde el centro de datos del Este de EE. UU., pero está experimentando un problema de disponibilidad parcial desde el Oeste de EE. UU. y Este de Asia.

![Gráfico de disponibilidad media por ubicaciones](./media/metric-chart-samples/availability-run-location.png)

### <a name="how-to-configure-this-chart"></a>Cómo configurar este gráfico

Primero deberá activar la supervisión de la [disponibilidad de Application Insights](../app/monitor-web-app-availability.md) para su sitio web. Después, elija el recurso de Application Insights y seleccione la métrica de disponibilidad. Aplique la división en la dimensión **Ejecutar ubicación**.

## <a name="volume-of-storage-account-transactions-by-api-name"></a>Volumen de transacciones de la cuenta de almacenamiento por nombre de API

El recurso de la cuenta de almacenamiento está experimentando un volumen excesivo de transacciones. Puede usar la métrica de transacciones para identificar qué API es responsable de la carga excesiva. Tenga en cuenta que el gráfico siguiente se configura con la misma dimensión (nombre de API) en el filtrado y la división para restringir la vista únicamente a las llamadas API del interés:

![Gráfico de barras de transacciones de API](./media/metric-chart-samples/transactions-by-api.png)

### <a name="how-to-configure-this-chart"></a>Cómo configurar este gráfico

En el selector de métricas, seleccione la cuenta de almacenamiento y la métrica **Transacciones**. Cambie el tipo de gráfico a **Gráfico de barras**. Haga clic en **Aplicar división** y seleccione el **nombre de API** de la dimensión. A continuación, haga clic en **Agregar filtro** y vuelva a elegir la dimensión **nombre de API**. En el cuadro de diálogo Filtro, seleccione las API que quiere trazar en el gráfico.

## <a name="next-steps"></a>Pasos siguientes

* Más información acerca de Azure Monitor [Workbooks](./workbooks-overview.md).
* Más información acerca del [Explorador de métricas](metrics-charts.md).

