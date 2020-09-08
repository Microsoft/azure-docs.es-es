---
title: Recopilación de datos de registro
titleSuffix: Azure Cognitive Search
description: Recopile y analice los datos de registro. Para ello, habilite una configuración de diagnóstico y, a continuación, use el lenguaje de consulta Kusto para explorar los resultados.
manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 06/30/2020
ms.openlocfilehash: 52230d6b13c4210e0ff8e85d0a3efe39af55f6e2
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88935065"
---
# <a name="collect-and-analyze-log-data-for-azure-cognitive-search"></a>Recopilación y análisis de datos de registro para Azure Cognitive Search

Los registros operativos o de diagnóstico proporcionan una visión general de las operaciones detalladas de Azure Cognitive Search y son útiles para supervisar los procesos de servicio y de carga de trabajo. Internamente, parte de la información del sistema se encuentra en el back-end durante un breve período de tiempo, suficiente para investigación y análisis si se archiva una incidencia de soporte técnico. Sin embargo, si desea autodireccionar datos operativos, debe establecer una configuración de diagnóstico para especificar dónde se recopila la información de registro.

El registro de diagnóstico se habilita a través de la integración con [Azure Monitor](../azure-monitor/index.yml). 

Al configurar el registro de diagnóstico, se le pedirá que especifique un mecanismo de almacenamiento. En la tabla siguiente se enumeran las opciones para recopilar y almacenar datos.

| Recurso | Se usa para |
|----------|----------|
| [Envío al área de trabajo de Log Analytics](../azure-monitor/learn/tutorial-resource-logs.md) | Los eventos y las métricas se envían a un área de trabajo de Log Analytics que se puede consultar en el portal para devolver información detallada. Para ver una introducción, consulte [Introducción a los registros de Azure Monitor](../azure-monitor/log-query/get-started-portal.md). |
| [Archivado con Blob Storage](../storage/blobs/storage-blobs-overview.md) | Los eventos y las métricas se archivan en un contenedor de blobs y se almacenan en archivos JSON. Los registros pueden ser bastante detallados (por hora o minuto), útiles para investigar un incidente específico, pero no para una investigación abierta. Use un editor de JSON para ver un archivo de registro sin formato o Power BI para agregar y visualizar datos de registro.|
| [Transmisión a Event Hubs](../event-hubs/index.yml) | Los eventos y las métricas se transmiten al servicio Azure Event Hubs. Elija esta opción como un servicio alternativo de recopilación de datos para registros muy grandes. |

## <a name="prerequisites"></a>Requisitos previos

Cree recursos de antemano para que pueda seleccionar uno o más al configurar el registro de diagnóstico.

+ [Cree un área de trabajo de Log Analytics](../azure-monitor/learn/quick-create-workspace.md)

+ [Cree una cuenta de almacenamiento](../storage/common/storage-account-create.md)

+ [Creación de un Centro de eventos](../event-hubs/event-hubs-create.md)

## <a name="enable-data-collection"></a>Habilitación de recolección de datos

La configuración de diagnóstico especifica cómo se recopilan los eventos y las métricas registrados.

1. En **Supervisión**, seleccione **Configuración de diagnóstico**.

   ![Configuración de diagnóstico](./media/search-monitor-usage/diagnostic-settings.png "Configuración de diagnóstico")

1. Seleccione **+ Agregar configuración de diagnóstico**.

1. Active **Log Analytics**, elija el área de trabajo y seleccione **OperationLogs** y **AllMetrics**.

   ![Configuración de la recopilación de datos](./media/search-monitor-usage/configure-storage.png "Configuración de la recopilación de datos")

1. Guarde la configuración.

1. Una vez habilitado el registro, use el servicio de búsqueda para empezar a generar registros y métricas. Tardará un tiempo antes de que los eventos registrados y las métricas estén disponibles.

En el caso de Log Analytics, puede tardar varios minutos antes de que los datos estén disponibles, después de lo cual se pueden ejecutar consultas de Kusto que devuelvan datos. Para más información, consulte [Supervisión de solicitudes de consulta](search-monitor-logs.md).

En el caso de Blob Storage, transcurre una hora antes de que los contenedores aparezcan en Blob Storage. Hay un blob, por hora y por contenedor. Los contenedores solo se crean cuando hay una actividad para registrar o medir. Cuando los datos se copian en una cuenta de almacenamiento, adoptan el formato JSON y se colocan en dos contenedores:

+ insights-logs-operationlogs: para los registros del tráfico de búsqueda
+ insights-metrics-pt1m: para las métricas

## <a name="query-log-information"></a>Consulta de la información de registro

Dos tablas contienen registros y métricas de Azure Cognitive Search: **AzureDiagnostics** y **AzureMetrics**.

1. En **Supervisión**, seleccione **Registros**.

1. Escriba **AzureMetrics** en la ventana de consulta. Ejecute esta sencilla consulta para familiarizarse con los datos recopilados en esta tabla. Desplácese por la tabla para ver las métricas y los valores. Observe el número de registros en la parte superior y, si el servicio ha estado recopilando métricas durante un tiempo, es posible que desee ajustar el intervalo de tiempo para obtener un conjunto de datos más manejable.

   ![Tabla AzureMetrics](./media/search-monitor-usage/azuremetrics-table.png "Tabla AzureMetrics")

1. Escriba la siguiente consulta para que devuelva un conjunto de resultados tabular.

   ```
   AzureMetrics
    | project MetricName, Total, Count, Maximum, Minimum, Average
   ```

1. Repita los pasos anteriores, empezando por **AzureDiagnostics** para que devuelva todas las columnas con fines informativos, seguido de una consulta más selectiva que extraiga información más interesante.

   ```
   AzureDiagnostics
   | project OperationName, resultSignature_d, DurationMs, Query_s, Documents_d, IndexName_s
   | where OperationName == "Query.Search" 
   ```

   ![Tabla AzureDiagnostics](./media/search-monitor-usage/azurediagnostics-table.png "Tabla AzureDiagnostics")

## <a name="kusto-query-examples"></a>Ejemplos de consultas de Kusto

Si habilitó el registro de diagnóstico, puede consultar **AzureDiagnostics** para obtener una lista de las operaciones que se ejecutaron en el servicio y cuándo. También puede poner en correlación una actividad para investigar los cambios en el rendimiento.

#### <a name="example-list-operations"></a>Ejemplo: Mostrar operaciones 

Devuelve una lista de operaciones y un recuento de cada una de ellas.

```
AzureDiagnostics
| summarize count() by OperationName
```

#### <a name="example-correlate-operations"></a>Ejemplo: Poner en correlación operaciones

Pone en correlación una solicitud de consulta con operaciones de indexación y representar los puntos de datos en un gráfico de tiempo.

```
AzureDiagnostics
| summarize OperationName, Count=count()
| where OperationName in ('Query.Search', 'Indexing.Index')
| summarize Count=count(), AvgLatency=avg(DurationMs) by bin(TimeGenerated, 1h), OperationName
| render timechart
```

## <a name="logged-operations"></a>Operaciones registradas

Los eventos registrados capturados por Azure Monitor incluyen aquellos relacionados con la indexación y las consultas. La tabla **AzureDiagnostics** de Log Analytics recopila datos operativos relacionados con consultas e indexación.

| OperationName | Descripción |
|---------------|-------------|
| ServiceStats | Esta operación es una llamada de una rutina a [Get Service Statistics](/rest/api/searchservice/get-service-statistics), que se realiza de forma directa o implícita para rellenar una página de información general del portal cuando se carga o se actualiza. |
| Query.Search |  Consulte [Supervisión de consultas](search-monitor-queries.md) para ver información acerca de las consultas registradas.|
| Indexing.Index  | Esta operación es una llamada a [Agregación, actualización o eliminación de documentos](/rest/api/searchservice/addupdate-or-delete-documents). |
| indexes.Prototype | Este es un índice que crea el Asistente para importar datos. |
| Indexers.Create | Cree un índice de forma explícita o implícita mediante el Asistente para importar datos. |
| Indexers.Get | Devuelve el nombre de un indexador siempre que este se ejecuta. |
| Indexers.Status | Devuelve el estado de un indexador siempre que este se ejecuta. |
| DataSources.Get | Devuelve el nombre del origen de datos siempre que se ejecuta un indexador.|
| Indexes.Get | Devuelve el nombre de un índice siempre que se ejecuta un indexador. |

## <a name="log-schema"></a>Esquema de registro

Si va a compilar informes personalizados, las estructuras de datos que contienen datos de registro de Azure Cognitive Search se ajustan al esquema siguiente. En el caso de Blob Storage, cada blob tiene un objeto raíz llamado **registros** que contiene una matriz de objetos de registro. Cada blob contiene registros de todas las operaciones que tuvieron lugar durante la misma hora.

La siguiente tabla es una lista parcial de campos comunes al registro de recursos.

| Nombre | Tipo | Ejemplo | Notas |
| --- | --- | --- | --- |
| timeGenerated |datetime |"2018-12-07T00:00:43.6872559Z" |Marca de tiempo de la operación |
| resourceId |string |"/SUBSCRIPTIONS/11111111-1111-1111-1111-111111111111/<br/>RESOURCEGROUPS/DEFAULT/PROVIDERS/<br/> MICROSOFT.SEARCH/SEARCHSERVICES/SEARCHSERVICE" |Su ResourceId |
| operationName |string |"Query.Search" |El nombre de la operación |
| operationVersion |string |"30-06-2020" |La versión de la API usada |
| category |string |"OperationLogs" |constant |
| resultType |string |"Success" |Valores posibles: Success o Failure |
| resultSignature |int |200 |Código de resultado HTTP |
| durationMS |int |50 |Duración de la operación en milisegundos |
| properties |object |consulte la tabla siguiente |Objeto que contiene datos específicos de la operación |

### <a name="properties-schema"></a>Esquema de propiedades

Las propiedades siguientes son específicas de Azure Cognitive Search.

| Nombre | Tipo | Ejemplo | Notas |
| --- | --- | --- | --- |
| Description_s |string |"GET /indexes('content')/docs" |Punto de conexión de la operación |
| Documents_d |int |42 |Número de documentos procesados |
| IndexName_s |string |"test-index" |Nombre del índice asociado a la operación |
| Query_s |string |"?search=AzureSearch&$count=true&api-version=2020-06-30" |Los parámetros de consulta |

## <a name="metrics-schema"></a>Esquema de métricas

Las métricas se capturan para las solicitudes de consulta y se miden en intervalos de un minuto. Cada métrica expone los valores mínimo, máximo y promedio por minuto. Para más información, consulte [Supervisión de solicitudes de consulta](search-monitor-queries.md).

| Nombre | Tipo | Ejemplo | Notas |
| --- | --- | --- | --- |
| resourceId |string |"/SUBSCRIPTIONS/11111111-1111-1111-1111-111111111111/<br/>RESOURCEGROUPS/DEFAULT/PROVIDERS/<br/>MICROSOFT.SEARCH/SEARCHSERVICES/SEARCHSERVICE" |el identificador de recurso |
| metricName |string |"Latency" |el nombre de la métrica |
| time |datetime |"2018-12-07T00:00:43.6872559Z" |la marca de tiempo de la operación |
| average |int |64 |El valor medio de las muestras sin procesar en el intervalo de tiempo de la métrica, unidades en segundos o porcentaje, según la métrica. |
| minimum |int |37 |El valor mínimo de las muestras sin procesar en el intervalo de tiempo de la métrica, unidades en segundos. |
| maximum |int |78 |El valor máximo de las muestras sin procesar en el intervalo de tiempo de la métrica, unidades en segundos.  |
| total |int |258 |El valor total de las muestras sin procesar en el intervalo de tiempo de la métrica, unidades en segundos.  |
| count |int |4 |El número de métricas emitidas desde un nodo al registro en el intervalo de un minuto.  |
| timegrain |string |"PT1M" |El intervalo de agregación de la métrica en ISO 8601. |

Es habitual que las consultas se ejecuten en cuestión de milisegundos, por lo que solo las consultas que duran segundos aparecerán en las métricas como QPS.

En el caso de la métrica **Consultas de búsqueda por segundo**, el mínimo es el valor más bajo de las consultas de búsqueda por segundo que se registró en ese minuto. Lo mismo sucede con el valor máximo. El promedio es el agregado en todo el minuto. Por ejemplo, en el transcurso de un minuto, podría tener un patrón parecido a este: un segundo de carga elevada que es el máximo en SearchQueriesPerSecond, seguido de 58 segundos de carga media y, por último, 1 segundo con solo una consulta (que es el mínimo).

En el caso de **Porcentaje de consultas de búsqueda limitadas**, "minimum", "maximum", "average" y "total" tienen el mismo valor: el porcentaje de consultas de búsqueda que se limitaron en comparación con el número total de consultas de búsqueda durante un minuto.

## <a name="view-raw-log-files"></a>Visualización de archivos de registro sin formato

Blob Storage se usa para archivar archivos de registro. Puede usar cualquier editor de JSON para ver el archivo de registro. Si no tiene uno, se recomienda [Visual Studio Code](https://code.visualstudio.com/download).

1. En Azure Portal, abra su cuenta de almacenamiento. 

2. En el panel de navegación izquierdo, haga clic en **Blobs**. Verá **insights-logs-operationlogs** e **insights-metrics-pt1m**. Estos contenedores se crean mediante Azure Cognitive Search cuando se exportan los datos de registro a Blob Storage.

3. Haga clic en la jerarquía de carpetas hasta llegar al archivo .json.  Use el menú contextual para descargar el archivo.

Una vez descargado el archivo, ábralo en un editor de JSON para ver el contenido.

## <a name="next-steps"></a>Pasos siguientes

Si aún no lo ha hecho, revise los aspectos fundamentales de la supervisión en el servicio de búsqueda para más información sobre la gama completa de funcionalidades de supervisión.

> [!div class="nextstepaction"]
> [Supervisión de operaciones y actividades en Azure Cognitive Search](search-monitor-usage.md)