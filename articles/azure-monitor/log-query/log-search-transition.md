---
title: Visualización y análisis de datos en Azure Log Analytics | Microsoft Docs
description: Asistencia para las consultas de registro de Azure Monitor para los usuarios de búsqueda de registros de Log Analytics.
ms.subservice: logs
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 01/16/2019
ms.openlocfilehash: 8abd9d7f33a07141418ad67cc2128af40ad0bd51
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87324767"
---
# <a name="transition-from-log-analytics-log-search-to-azure-monitor-logs"></a>Transición desde la búsqueda de registros de Log Analytics a los registros de Azure Monitor
La búsqueda de registros en Log Analytics se ha reemplazado recientemente por una nueva experiencia para analizar los registros de Azure Monitor. La página de búsqueda de registros sigue siendo accesible con el elemento de menú **Registros (clásicos)** de la página **Áreas de trabajo de Log Analytics** en Azure Portal, pero se retirará el 15 de febrero de 2019. En este artículo se describen las diferencias entre las dos experiencias para ayudar a realizar la transición desde la búsqueda de registros. 

## <a name="filter-results-of-a-query"></a>Filtrado de los resultados de una consulta
En la búsqueda de registros, se muestra una lista de filtros cuando se entregan los resultados de búsqueda. Seleccione un filtro y haga clic en **Aplicar** para ejecutar la consulta con el filtro seleccionado.

![Filtro de búsqueda de registros](media/log-search-transition/filter-log-search.png)

En los registros de Azure Monitor, seleccione *Filter (preview)* [Filtro (versión preliminar)] para mostrar los filtros. Haga clic en el icono de filtro para mostrar los filtros de adición. Seleccione un filtro y haga clic en **Apply & Run** (Aplicar y ejecutar) para ejecutar la consulta con el filtro seleccionado.

![Filtro de registros](media/log-search-transition/filter-logs.png)

## <a name="extract-custom-fields"></a>Extracción de campos personalizados 
En la búsqueda de registros, los [campos personalizados](../platform/custom-fields.md) se extraen en la vista de lista, donde el menú de un campo incluye la acción _Extraer campos de tabla_.

![Extraer campos en la búsqueda de registros](media/log-search-transition/extract-fields-log-search.png)

En los registros de Azure Monitor, los campos personalizados se extraen en la vista de tabla. Haga clic en la flecha situada a la izquierda de un registro para expandirlo y, después, haga clic en los puntos suspensivos para acceder a la acción _Extract fields_ (Extraer campos).

![Extracción de campos de registro](media/log-search-transition/extract-fields-logs.png)

## <a name="functions-and-computer-groups"></a>Funciones y grupos de equipos
Para guardar una búsqueda en la búsqueda de registros, seleccione **Búsquedas guardadas** y **Agregar** para proporcionar el nombre, la categoría y el texto de la consulta. Agregue un alias de función para crear un [grupo de equipos](../platform/computer-groups.md).

![Guardado de la búsqueda de registros](media/log-search-transition/save-search-log-search.png)

Para guardar la consulta actual en los registros de Azure Monitor, seleccione **Save** (Guardar). Cambie **Save as** (Guardar como) a _Function_ (Función) y proporcione un alias de función en **Function Alias** para crear una [función](functions.md). Seleccione _Guardar esta consulta como un grupo de equipos_ para utilizar el alias de función para un [grupo de equipos](../platform/computer-groups.md).

![Guardado de la consulta de registros](media/log-search-transition/save-query-logs.png)

## <a name="saved-queries"></a>Consultas guardadas
En la búsqueda de registros, las consultas guardadas están disponibles en el elemento de la barra de acciones **Búsquedas guardadas**. En los registros de Azure Monitor, las consultas guardadas están disponibles en el [Explorador de consultas](./get-started-portal.md#save-queries).

![Explorador de consultas](media/log-search-transition/query-explorer.png)

## <a name="drill-down-on-summarized-rows"></a>Exploración en profundidad de las filas resumidas
En la búsqueda de registros, puede hacer clic en una fila de una consulta resumida para iniciar otra consulta que muestre los registros detallados de esa fila.

![Exploración en profundidad de la búsqueda de registros](media/log-search-transition/drilldown-search.png)

En los registros de Azure Monitor, debe modificar la consulta para que devuelva estos registros. Expanda una de las filas en los resultados y haga clic en **+** junto al valor para agregarlo a la consulta. A continuación, marque como comentario el comando **summarize** y vuelva a ejecutar la consulta.

![Exploración en profundidad de los registros de Azure Monitor](media/log-search-transition/drilldown-logs.png)

## <a name="take-action"></a>Realizar acción
En la búsqueda de registros, puede seleccionar **Realizar acción** para [iniciar un runbook](../platform/action-groups.md).

![Realizar acción](media/log-search-transition/take-action-log-search.png)

En los registros de Azure Monitor, [cree una alerta para la consulta de registros](../platform/alerts-log.md). Configure un grupo de acciones con una o varias acciones que se ejecutarán en respuesta a la alerta.

![Grupo de acciones](media/log-search-transition/action-group.png)

## <a name="next-steps"></a>Pasos siguientes

- Más información sobre la nueva [experiencia de registros de Azure Monitor](get-started-portal.md).

