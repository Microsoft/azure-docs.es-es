---
title: Orígenes de datos de libros de Azure Monitor | Microsoft docs
description: Simplifique los informes complejos con libros de Azure Monitor parametrizados predefinidos y personalizados creados a partir de varios orígenes de datos.
services: azure-monitor
documentationcenter: ''
author: mrbullwinkle
manager: carmonm
ms.workload: tbd
ms.tgt_pltfrm: ibiza
ms.topic: conceptual
ms.date: 06/29/2020
ms.author: mbullwin
ms.openlocfilehash: f27771291d95770a693fa56041f7dce3de459d13
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87081430"
---
# <a name="azure-monitor-workbooks-data-sources"></a>Orígenes de datos de libros de Azure Monitor

Los libros son compatibles con un gran número de orígenes de datos. Este artículo le guía por los orígenes de datos que están disponibles actualmente para libros de Azure Monitor.

## <a name="logs"></a>Registros

Los libros permiten consultar registros desde los orígenes siguientes:

* Registros de Azure Monitor (recursos de Application Insights y áreas de trabajo de Log Analytics)
* Datos orientados a recursos (registros de actividad)

Los autores de libros pueden usar consultas de KQL que transforman los datos de recursos subyacentes para seleccionar un conjunto de resultados que se puede visualizar como texto, gráficos o cuadrículas.

![Captura de pantalla de la interfaz de informes de registros de libros](./media/workbooks-overview/logs.png)

Los autores de libros pueden realizar consultas fácilmente en varios recursos y crear una experiencia de creación de informes realmente unificada.

## <a name="metrics"></a>Métricas

Los recursos de Azure emiten [métricas](data-platform-metrics.md) a las que se puede acceder mediante los libros. El acceso a las métricas de los libros se puede realizar mediante un control especializado que permite especificar los recursos de destino, las métricas deseadas y su agregación. Estos datos se pueden trazar en gráficos o cuadrículas.

![Captura de pantalla de gráficos de métricas de libro de uso de CPU](./media/workbooks-overview/metrics-graph.png)

![Captura de pantalla de la interfaz de métricas de libro](./media/workbooks-overview/metrics.png)

## <a name="azure-resource-graph"></a>Azure Resource Graph

Los libros admiten la consulta de recursos y sus metadatos mediante Azure Resource Graph (ARG). Esta funcionalidad se usa principalmente para crear ámbitos de consulta personalizados para los informes. El ámbito de los recursos se expresa mediante un subconjunto KQL que es compatible con ARG, que suele ser suficiente en la mayoría de los casos.

Para que un control de consulta use este origen de datos, utilice la lista desplegable de tipo de consulta para elegir Azure Resource Graph y seleccione las suscripciones de destino. Use el control de consulta para agregar el subconjunto KQL para ARG que selecciona un subconjunto de recursos interesante.

![Captura de pantalla de la consulta KQL de Azure Resource Graph](./media/workbooks-overview/azure-resource-graph.png)

## <a name="azure-resource-manager"></a>Azure Resource Manager

El libro admite operaciones REST de Azure Resource Manager. Esto permite consultar el punto de conexión de management.azure.com sin necesidad de proporcionar su propio token de encabezado de autorización.

Para que un control de consulta use este origen de datos, utilice la lista desplegable Origen de datos para elegir Azure Resource Manager. Proporcione los parámetros adecuados como el método HTTP, la ruta de dirección URL, los encabezados, los parámetros de URL o el cuerpo.

> [!NOTE]
> Actualmente solo se admiten las operaciones `GET`, `POST`y `HEAD`.

## <a name="azure-data-explorer"></a>Explorador de datos de Azure

Los libros admiten ahora la consulta desde clústeres de [Azure Data Explorer](/azure/data-explorer/) con el poderoso lenguaje de consulta de [Kusto](/azure/kusto/query/index).   

![Captura de pantalla de la ventana de consulta de Kusto](./media/workbooks-overview/data-explorer.png)

## <a name="workload-health"></a>Mantenimiento de la carga de trabajo

Azure Monitor presenta funcionalidad que supervisa de forma proactiva la disponibilidad y el rendimiento de los sistemas operativos invitados Windows o Linux. Azure Monitor modela componentes clave y sus relaciones, criterios para medir el estado de dichos componentes y qué componentes generan una alerta cuando se detecta una condición incorrecta. Los libros permiten a los usuarios usar esta información para crear informes interactivos enriquecidos.

Para que un control de consulta use este origen de datos, utilice la lista desplegable **Tipo de consulta** para elegir Mantenimiento de la carga de trabajo y seleccione la suscripción, el grupo de recursos o los recursos de máquina virtual de destino. Use las listas desplegables de filtros de mantenimiento para seleccionar un subconjunto interesante de incidentes de mantenimiento para sus necesidades de análisis.

![Captura de pantalla de consulta de alertas](./media/workbooks-overview/workload-health.png)

## <a name="azure-resource-health"></a>Azure Resource Health

Los libros permiten obtener el estado de los recursos de Azure y combinarlo con otros orígenes de datos para crear informes de mantenimiento completos e interactivos.

Para que un control de consulta use este origen de datos, utilice la lista desplegable **Tipo de consulta** para elegir Azure Health y seleccione los recursos de destino. Use las listas desplegables de filtros de mantenimiento para seleccionar un subconjunto interesante de problemas de recursos para sus necesidades de análisis.

![Captura de pantalla de consulta de alertas](./media/workbooks-overview/resource-health.png)

## <a name="json"></a>JSON

El proveedor JSON permite crear un resultado de la consulta a partir de contenido de JSON estático. Normalmente se usa en parámetros para crear parámetros de lista desplegable de valores estáticos. Los objetos o las matrices JSON simples se convertirán automáticamente en filas y columnas de la cuadrícula.  Para comportamientos más específicos, puede usar la pestaña Resultados y la configuración JSONPath para configurar columnas.

## <a name="alerts-preview"></a>Alertas (versión preliminar)

> [!NOTE]
> La forma recomendada para consultar la información de alertas de Azure es usar el origen de datos [Azure Resource Graph](#azure-resource-graph) consultando la tabla de `AlertsManagementResources`.
>
> Vea la [referencia de tabla de Azure Resource Graph](../../governance/resource-graph/reference/supported-tables-resources.md) o la [plantilla Alertas](https://github.com/microsoft/Application-Insights-Workbooks/blob/master/Workbooks/Azure%20Resources/Alerts/Alerts.workbook) para obtener ejemplos.
>
> El origen de datos Alertas permanecerá disponible durante un período de tiempo mientras los autores empiezan a usar ARG. No se recomienda el uso de este origen de datos en las plantillas. 

Los libros permiten a los usuarios visualizar las alertas activas relacionadas con sus recursos. Limitaciones: el origen de datos de alertas requiere acceso de lectura a la suscripción para consultar los recursos y es posible que no muestre nuevos tipos de alertas. 

Para que un control de consulta use este origen de datos, utilice la lista desplegable _Origen de datos_ para elegir _Alertas (versión preliminar)_ y seleccione las suscripciones, los grupos de recursos o los recursos de destino. Use las listas desplegables de filtros de alertas para seleccionar un subconjunto interesante de alertas para sus necesidades de análisis.

## <a name="custom-endpoint"></a>Punto de conexión personalizado

Los libros permiten obtener datos desde cualquier origen externo. Si los datos residen fuera de Azure, puede llevarlos a los libros mediante este tipo de origen de datos.

Para que un control de consulta use este origen de datos, utilice la lista desplegable _Origen de datos_ para elegir _Punto de conexión personalizado_. Proporcione los parámetros adecuados como `Http method`, `url`, `headers`, `url parameters` o `body`. Asegúrese de que el origen de datos admita [CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS); de lo contrario, se producirá un error en la solicitud.

Para evitar la realización automática de llamadas a hosts que no son de confianza cuando se usan plantillas, el usuario debe marcar los hosts utilizados como de confianza. Para ello, haga clic en el botón _Agregar como de confianza_ o agréguelo como un host de confianza en la configuración del libro. Esta configuración se guardará en los exploradores que admiten IndexDb con roles de trabajo. [Aquí](https://caniuse.com/#feat=indexeddb) hay más información.

> [!NOTE]
> No escriba ningún secreto en ninguno de los campos (`headers`, `parameters`, `body`, `url`), ya que estarán visibles para todos los usuarios del libro.

## <a name="next-steps"></a>Pasos siguientes

* [Comience](workbooks-visualizations.md) a aprender más sobre las muchas opciones de visualizaciones enriquecidas de los libros.
* [Controle](workbooks-access-control.md) y comparta el acceso a los recursos del libro.
* [Sugerencias para la optimización de consultas de análisis de registros](../log-query/query-optimization.md)
* 
