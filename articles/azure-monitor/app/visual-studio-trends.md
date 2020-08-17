---
title: Análisis de tendencias en Visual Studio | Microsoft Docs
description: Analizar, mostrar y explorar las tendencias de la telemetría de Application Insights en Visual Studio.
ms.topic: conceptual
author: NumberByColors
ms.author: daviste
ms.date: 03/17/2017
ms.reviewer: mbullwin
ms.custom: vs-azure
ms.openlocfilehash: 07848d0b1a2677595eba0da53402993103289d4f
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87318171"
---
# <a name="analyzing-trends-in-visual-studio"></a>Análisis de tendencias en Visual Studio
La herramienta Tendencias de Application Insights muestra cómo los eventos de telemetría importantes de la aplicación web cambian a lo largo del tiempo, lo que ayuda a identificar rápidamente los problemas y las anomalías. Al vincularle a información de diagnóstico más detallada, Tendencias puede ayudarle a mejorar el rendimiento de la aplicación, realizar un seguimiento de las causas de las excepciones y revelar información de los eventos personalizados.

![Ejemplo de la ventana de tendencias](./media/visual-studio-trends/app-insights-trends-hero-750.png)

## <a name="configure-your-web-app-for-application-insights"></a>Configuración de la aplicación web para Application Insights

Si aún no lo ha hecho, [configure la aplicación web para Application Insights](./app-insights-overview.md). Esto le permite enviar datos de telemetría al portal de Application Insights. La herramienta Tendencias lee ahí la telemetría.

Tendencias de Application Insights está disponible en Visual Studio 2015 Update 3 y versiones posteriores.

## <a name="open-application-insights-trends"></a>Tendencias de Application Insights
Para abrir la ventana de Tendencias de Application Insights:

* En el botón de la barra de herramientas de Application Insights, elija **Explorar tendencias de telemetría**, o bien
* En el menú contextual del proyecto, elija **Application Insights > Explorar tendencias de telemetría**, o bien
* En la barra de menús de Visual Studio, elija **Vista > Otras ventanas > Tendencias de Application Insights**.

Verá un aviso para seleccionar un recurso. Haga clic en **Seleccionar un recurso**, inicie sesión con una suscripción de Azure y elija el recurso de Application Insights de la lista para el que desea analizar las tendencias de telemetría.

## <a name="choose-a-trend-analysis"></a>Elección de un análisis de tendencias
![Menú de tipos comunes de análisis de tendencias](./media/visual-studio-trends/app-insights-trends-1-750.png)

Para empezar, elija entre uno de los cinco análisis de tendencias comunes, cada de los cuales analiza los datos de las últimas 24 horas:

* **Investigue los problemas de rendimiento con las solicitudes del servidor** : solicitudes realizadas al servicio, agrupadas por tiempos de respuesta.
* **Analice los errores en las solicitudes del servidor** : solicitudes realizadas al servicio, agrupadas por código de respuesta HTTP.
* **Examine las excepciones de la aplicación** -excepciones del servicio, agrupadas por tipo de excepción.
* **Compruebe el rendimiento de las dependencias de la aplicación** : servicios llamados por su servicio, agrupados por tiempos de respuesta.
* **Inspeccione sus eventos personalizados** : eventos personalizados que se ha configurado para el servicio, agrupados por tipo de evento.

Estos análisis pregenerados están disponibles más adelante, en el botón **Ver tipos comunes de análisis de telemetría** , situado en la esquina superior izquierda de la ventana Tendencias.

## <a name="visualize-trends-in-your-application"></a>Visualización de tendencias en la aplicación
Tendencias de Application Insights crea una visualización de la serie temporal o de la telemetría de su aplicación. Cada visualización de la serie temporal muestra un tipo de telemetría, agrupados por una propiedad de esa telemetría, a lo largo de algún intervalo de tiempo. Por ejemplo, puede que quiera ver las solicitudes del servidor, agrupadas por el país o región en el que se han originado, a lo largo de las últimas 24 horas. En este ejemplo, cada burbuja de la visualización representaría un recuento de las solicitudes del servidor para algún país o región durante una hora.

Utilice los controles de la parte superior de la ventana para ajustar los tipos de telemetría que ve. En primer lugar, elija los tipos de telemetría en los que está interesado:

* **Tipo de telemetría**: solicitudes del servidor, excepciones, dependencias o eventos personalizados.
* **Intervalo de tiempo** : en cualquier momento entre los últimos 30 minutos y los últimos tres días.
* **Agrupar por** : tipo excepción, identificador del problema, país o región, etc.

A continuación, haga clic en **Analizar telemetría** para ejecutar la consulta.

Para desplazarse entre las burbujas de la visualización:

* Haga clic para seleccionar una burbuja; esto actualiza los filtros situados en la parte inferior de la ventana y resume solo los eventos que se produjeron durante un período específico.
* Haga doble clic en una burbuja para ir a la herramienta de búsqueda y ver todos los eventos de telemetría individuales que se produjeron durante ese período.
* Haga Ctrl-clic en una burbuja para anular su selección en la visualización.

> [!TIP]
> Las herramientas Tendencias y Buscar funcionan conjuntamente para ayudar a identificar las causas de los problemas en el servicio entre miles de eventos de telemetría. Por ejemplo, si una tarde sus clientes notan que la aplicación tiene menos capacidad de respuesta, empiece con Tendencias. Analice las solicitudes al servicio realizadas durante las últimas horas, agrupadas por tiempo de respuesta. Vea si hay un clúster de solicitudes lentas inusualmente grande. A continuación, haga doble clic en esa burbuja para ir a la herramienta Buscar, con el filtro esos eventos de solicitud filtrados. En Buscar puede explorar el contenido de esas solicitudes e ir al código correspondiente para resolver el problema.
> 
> 

## <a name="filter"></a>Filter
Detecte tendencias más específicas con los controles de filtro en la parte inferior de la ventana. Para aplicar un filtro, haga clic en su nombre. Puede cambiar rápidamente entre distintos filtros para detectar tendencias que pueden estar ocultas en una determinada dimensión de la telemetría. Si aplica un filtro en una dimensión, como Tipo de excepción, sigue siendo posible hacer clic en los filtros de otras dimensione, aunque aparecen atenuados. Para dejar de aplicar un filtro, vuelva a hacer clic en él. Haga ctrl-clic para seleccionar varios filtros de la misma dimensión.

![Filtros de tendencia](./media/visual-studio-trends/TrendsFiltering-750.png)

¿Qué ocurre si desea aplicar varios filtros? 

1. Aplique el primer filtro. 
2. Haga clic en el botón **Apply selected filters and query again** (Aplicar filtros seleccionados y volver a consultar) para usar el nombre de la dimensión de su primer filtro. Esto volverá a consultar la telemetría solo para los eventos que coinciden con el primer filtro. 
3. Aplique un segundo filtro. 
4. Repita el proceso para buscar tendencias en subconjuntos específicos de la telemetría. Por ejemplo, las solicitudes de servidor denominadas "GET Home/Index" *y* que procedían de Alemania *y* que recibieron un código de respuesta 500. 

Para dejar de aplicar uno de estos filtros, haga clic en el botón **Remove selected filters and query again** (Quitar filtros seleccionados y volver a consultar) de la dimensión.

![Varios filtros](./media/visual-studio-trends/TrendsFiltering2-750.png)

## <a name="find-anomalies"></a>Búsqueda de anomalías
La herramienta Tendencias puede resaltar las burbujas de eventos anómalos en comparación con otras burbujas de la misma serie temporal. En la lista desplegable Tipo de vista, elija **Recuentos en el ciclo (resaltar anomalías)** o **Porcentajes en el ciclo (resaltar anomalías)** . Las burbujas rojas son anómalas. Las anomalías se definen como burbujas con recuentos o porcentajes que superan 2,1 veces la desviación estándar de los recuentos o porcentajes que se produjeron en los últimos dos períodos (48 horas si está viendo las últimas 24 horas, etc.).

![Los puntos de color indican anomalías](./media/visual-studio-trends/TrendsAnomalies-750.png)

> [!TIP]
> Resaltar las anomalías es especialmente útil para buscar valores atípicos en series temporales de pequeñas burbujas que, de lo contrario, pueden parecer de tamaño similar.  
> 
> 

## <a name="next-steps"></a><a name="next"></a>Pasos siguientes
|  |  |
| --- | --- |
| **[Trabajo con Application Insights en Visual Studio](./visual-studio.md)**<br/>Busque la telemetría, consulte los datos de CodeLens y configure Application Insights. Todos desde Visual Studio. |![Haga clic con el botón derecho en el proyecto y seleccione Application Insights, Búsqueda.](./media/visual-studio-trends/34.png) |
| **[Incorporación de datos adicionales](./asp-net-more.md)**<br/>Supervise el uso, la disponibilidad, las dependencias y las excepciones. Integrar seguimientos de marcos de registro. Escribir telemetría personalizada. |![Visual Studio](./media/visual-studio-trends/64.png) |
| **[Trabajo con el portal de Application Insights](./overview-dashboard.md)**<br/>Paneles, eficaces herramientas de diagnóstico y análisis, alertas, un mapa activo de dependencias de la aplicación y exportación de la telemetría. |![Visual Studio](./media/visual-studio-trends/62.png) |

