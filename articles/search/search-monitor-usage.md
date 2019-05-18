---
title: Supervisión del uso de recursos y las métricas de consulta para un servicio de búsqueda en Azure Search
description: Habilite el registro y obtenga métricas de actividades de consulta, uso de recursos y otros datos del sistema de un servicio de Azure Search.
author: HeidiSteen
manager: cgronlun
tags: azure-portal
services: search
ms.service: search
ms.devlang: na
ms.topic: conceptual
ms.date: 05/16/2019
ms.author: heidist
ms.custom: seodec2018
ms.openlocfilehash: 3fa463cb7178fa5cc2108383047a7ca94ffb48a3
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65797374"
---
# <a name="monitor-resource-consumption-and-query-activity-in-azure-search"></a>Supervisión del consumo de recursos y la actividad de consulta en Azure Search

En la página de información general del servicio Azure Search, puede ver los datos del sistema sobre el uso de recursos, las métricas de consulta y la cantidad de cuota disponible para crear más índices, indexadores y orígenes de datos. También puede usar el portal para configurar el análisis de registros u otro recurso usado para la recopilación de datos persistentes. 

La configuración de registros es útil para el autodiagnóstico y la conservación del historial de operaciones. Internamente, los registros existen en el back-end durante un breve período de tiempo, suficiente para su investigación y análisis si presenta una incidencia de soporte técnico. Si quiere tener control sobre la información de registro y acceder a ella, debe configurar una de las soluciones que se describen en este artículo.

En este artículo, aprenderá sobre las opciones de supervisión, cómo habilitar el registro y el almacenamiento de registros y cómo ver el contenido del registro.

## <a name="metrics-at-a-glance"></a>Métricas de un vistazo

Las secciones **Usage** (Uso) y **Monitoring** (Supervisión) integradas en la página de información general informan sobre las métricas de consumo de recursos y ejecución de consultas. Esta información está disponible en cuanto comienza a usar el servicio, sin necesidad de ninguna configuración. Esta página se actualiza cada pocos minutos. Si va a concretar las decisiones sobre [qué nivel usar para las cargas de trabajo de producción](search-sku-tier.md) o si [ajustar el número de réplicas activas y particiones](search-capacity-planning.md), estas métricas pueden ayudarle con esas decisiones al mostrarle la velocidad a la que se consumen los recursos y el modo en que la configuración actual administra la carga existente.

La pestaña **Usage** (Uso) muestra la disponibilidad de recursos relativa a los [límites](search-limits-quotas-capacity.md) actuales. La siguiente ilustración es del servicio gratuito, que está limitado a 3 objetos de cada tipo y 50 MB de almacenamiento. Un servicio básico o estándar tiene límites más altos y, si aumentan los recuentos de particiones, el almacenamiento máximo aumenta proporcionalmente.

![Estado de uso relativo a los límites efectivos](./media/search-monitor-usage/usage-tab.png
 "Usage status relative to effective limits")

## <a name="queries-per-second-qps-and-other-metrics"></a>Consultas por segundo (QPS) y otras métricas

La pestaña **Monitoring** (Supervisión) muestra las medias acumuladas de métricas como *Search queries per second* (QPS) (Buscar consultas por segundo), agregadas por minuto. 
*Search latency* (Latencia de búsqueda): la cantidad de tiempo que ha necesitado el servicio de búsqueda para procesar las consultas de búsqueda, agregadas por minuto. *Throttled search queries percentage* (Porcentaje de consultas de búsqueda limitadas) (no se muestra): porcentaje de consultas de búsqueda que se han limitado, también agregadas por minuto.

Estos números son aproximados y están pensados para proporcionarle una idea general de la capacidad del sistema para atender las solicitudes. El QPS real puede ser mayor o menor que el número notificado en el portal.

![Actividad de consultas por segunda](./media/search-monitor-usage/monitoring-tab.png "Queries per second activity")

## <a name="activity-logs"></a>Registros de actividad

El **registro de actividad** recopila información de Azure Resource Manager. Algunos ejemplos de la información encontrada en el registro de actividad incluyen la creación o eliminación de un servicio, la actualización de un grupo de recursos, la comprobación de disponibilidad de nombres o la obtención de una clave de acceso del servicio para administrar una solicitud. 

Puede acceder al **registro de actividad** desde el panel de navegación izquierdo, desde las notificaciones de la barra de comandos de la ventana superior o desde la página **Diagnose and solve problems** (Diagnosticar y resolver problemas).

Para las tareas en servicio, como crear un índice o eliminar un origen de datos, verá notificaciones genéricas, como "Obtener la clave de administrador" para cada solicitud, pero no la propia acción específica. En este nivel de información, debe habilitar una solución de supervisión complementaria.

## <a name="add-on-monitoring-solutions"></a>Soluciones de supervisión complementarias

Azure Search no almacena ningún dato aparte de los objetos que administra, lo que significa que los datos de registro se deben almacenar externamente. Puede configurar cualquiera de los recursos siguientes si desea conservar los datos de registro. 

En la tabla siguiente se comparan las opciones para almacenar registros, agregar supervisión exhaustiva de las operaciones de servicio y consultar las cargas de trabajo mediante Application Insights.

| Recurso | Usado para |
|----------|----------|
| [Application Insights](https://docs.microsoft.com/azure/azure-monitor/app/app-insights-overview) | Los eventos registrados y las métricas de consulta, según los esquemas siguientes, se correlacionan con los eventos de usuario en la aplicación. Esta es la única solución que tiene en cuenta las acciones o señales del usuario, y asigna eventos desde la búsqueda iniciada por el usuario, por oposición al filtrado de las solicitudes enviadas por el código de aplicación. Para usar este enfoque, copie y pegue el código de instrumentación en los archivos de código fuente para enrutar la información de solicitud a Application Insights. Para más información, consulte [Análisis de tráfico de búsqueda](search-traffic-analytics.md). |
| [Registros de Azure Monitor](https://docs.microsoft.com/azure/azure-monitor/log-query/log-query-overview) | Los eventos registrados y las métricas de consulta, según los esquemas siguientes. Los eventos se registran en un área de trabajo de Log Analytics. Puede ejecutar consultas en un área de trabajo para devolver información detallada del registro. Para obtener más información, consulte [empezar a trabajar con registros de Azure Monitor](https://docs.microsoft.com/azure/azure-monitor/learn/tutorial-viewdata) |
| [Blob Storage](https://docs.microsoft.com/azure/storage/blobs/storage-blobs-overview) | Los eventos registrados y las métricas de consulta, según los esquemas siguientes. Los eventos se registran en un contenedor de blobs y se almacenan en archivos JSON. Use un editor de JSON para ver el contenido del archivo.|
| [Event Hubs](https://docs.microsoft.com/azure/event-hubs/) | Eventos registrados y métricas de consulta, según los esquemas documentados en este artículo. Elija esta opción como un servicio alternativo de recopilación de datos para registros muy grandes. |

Los registros de Azure Monitor y Blob storage están disponibles como un servicio compartido gratuito, por lo que puede probarlo sin ningún costo para la duración de la suscripción de Azure. La suscripción a Application Insights y su uso son gratuitos siempre y cuando el tamaño de los datos de aplicación esté por debajo de determinados límites (consulte la [página de precios](https://azure.microsoft.com/pricing/details/monitor/) para más información).

La siguiente sección le guía por los pasos necesarios para habilitar y usar Azure Blob Storage para recopilar datos de registro creados por las operaciones de Azure Search, y acceder a ellos.

## <a name="enable-logging"></a>Habilitar registro

De forma predeterminada, el registro para indexación y consulta de cargas de trabajo está desactivado y la infraestructura de registro y el almacenamiento externo a largo plazo dependen de soluciones complementarias. Por sí solos, los únicos datos persistentes en Azure Search son los objetos que crea y administra, por lo que los registros se deben almacenar en otra parte.

En esta sección, aprenderá a usar Blob Storage para almacenar datos de métricas y eventos registrados.

1. Si aún no tiene una, [cree una cuenta de almacenamiento](https://docs.microsoft.com/azure/storage/common/storage-quickstart-create-account). Puede colocarla en el mismo grupo de recursos que Azure Search para simplificar la limpieza más adelante si quiere eliminar todos los recursos usados en este ejercicio.

   La cuenta de almacenamiento debe existir en la misma región que Azure Search.

2. Abra la página de información general del servicio de búsqueda. En el panel de navegación izquierdo, desplácese a **Monitoring** (Supervisión) y haga clic en **Enable Monitoring** (Habilitar supervisión).

   ![Habilitar supervisión](./media/search-monitor-usage/enable-monitoring.png "Enable monitoring")

3. Elija los datos que quiere exportar: registros, métricas o ambos. Puede copiarla en una cuenta de almacenamiento, enviarla a un centro de eventos o exportarlo a los registros de Azure Monitor.

   Para el archivado en Blob Storage, solo debe existir la cuenta de almacenamiento. Contenedores y blobs se creará según sea necesario cuando se exportan datos de registro.

   ![Configuración del archivado de Blob Storage](./media/search-monitor-usage/configure-blob-storage-archive.png "Configure blob storage archive")

4. Guarde el perfil.

5. Para probar el registro, cree o elimine objetos (se crean eventos de registro) y envíe consultas (se generan métricas). 

El registro se habilita después de guardar el perfil. Los contenedores solo se crean cuando hay una actividad para registrar o medir. Cuando los datos se copian en una cuenta de almacenamiento, adoptan el formato JSON y se colocan en dos contenedores:

* insights-logs-operationlogs: para los registros del tráfico de búsqueda
* insights-metrics-pt1m: para las métricas

**Se tarda una hora antes de que aparezcan los contenedores de Blob storage. Hay un blob, por hora y por contenedor.**

Puede usar [Visual Studio Code](#download-and-open-in-visual-studio-code) u otro editor de JSON para ver los archivos. 

### <a name="example-path"></a>Ruta de acceso de ejemplo

```
resourceId=/subscriptions/<subscriptionID>/resourcegroups/<resourceGroupName>/providers/microsoft.search/searchservices/<searchServiceName>/y=2018/m=12/d=25/h=01/m=00/name=PT1H.json
```

## <a name="log-schema"></a>Esquema de registro
Los blobs que contienen los registros de tráfico del servicio de búsqueda se estructuran como se describe en esta sección. Cada blob tiene un objeto raíz llamado **registros** que contiene una matriz de objetos de registro. Cada blob contiene registros de todas las operaciones que tuvieron lugar durante la misma hora.

| NOMBRE | Type | Ejemplo | Notas |
| --- | --- | --- | --- |
| time |datetime |"2018-12-07T00:00:43.6872559Z" |Marca de tiempo de la operación |
| resourceId |string |"/SUBSCRIPTIONS/11111111-1111-1111-1111-111111111111/<br/>RESOURCEGROUPS/DEFAULT/PROVIDERS/<br/>  MICROSOFT.SEARCH/SEARCHSERVICES/SEARCHSERVICE" |Su ResourceId |
| operationName |string |"Query.Search" |El nombre de la operación |
| operationVersion |string |"2019-05-06" |La versión de la API usada |
| category |string |"OperationLogs" |constant |
| resultType |string |"Success" |Valores posibles: Success o Failure |
| resultSignature |int |200 |Código de resultado HTTP |
| durationMS |int |50 |Duración de la operación en milisegundos |
| properties |objeto |consulte la tabla siguiente |Objeto que contiene datos específicos de la operación |

**Esquema de propiedades**

| NOMBRE | Type | Ejemplo | Notas |
| --- | --- | --- | --- |
| DESCRIPCIÓN |string |"GET /indexes('content')/docs" |Punto de conexión de la operación |
| Consultar |string |"?search=AzureSearch&$count=true&api-version=2019-05-06" |Los parámetros de consulta |
| Documentos |int |42 |Número de documentos procesados |
| IndexName |string |"testindex" |Nombre del índice asociado a la operación |

## <a name="metrics-schema"></a>Esquema de métricas

Se capturan las métricas de solicitudes de consulta.

| NOMBRE | Type | Ejemplo | Notas |
| --- | --- | --- | --- |
| resourceId |string |"/SUBSCRIPTIONS/11111111-1111-1111-1111-111111111111/<br/>RESOURCEGROUPS/DEFAULT/PROVIDERS/<br/> MICROSOFT.SEARCH/SEARCHSERVICES/SEARCHSERVICE" |el identificador de recurso |
| metricName |string |"Latency" |el nombre de la métrica |
| Twitter en tiempo |datetime |"2018-12-07T00:00:43.6872559Z" |la marca de tiempo de la operación |
| media |int |64 |El valor de media de las muestras sin procesar en el intervalo de tiempo de la métrica |
| mínimo |int |37 |El valor mínimo de las muestras sin procesar en el intervalo de tiempo de la métrica |
| máximo |int |78 |El valor máximo de las muestras sin procesar en el intervalo de tiempo de la métrica |
| total |int |258 |El valor total de las muestras sin procesar en el intervalo de tiempo de la métrica |
| número |int |4 |El número de muestras sin procesar usadas para generar la métrica |
| timegrain |string |"PT1M" |El intervalo de agregación de la métrica en ISO 8601 |

Todas las métricas se notifican en intervalos de un minuto. Cada métrica expone los valores mínimo, máximo y promedio por minuto.

En el caso de la métrica SearchQueriesPerSecond, el mínimo es el valor más bajo de las consultas de búsqueda por segundo que se registró en ese minuto. Lo mismo sucede con el valor máximo. El promedio es el agregado en todo el minuto.
Piense en este escenario durante un minuto: un segundo de carga elevada que es el máximo en SearchQueriesPerSecond, seguido de 58 segundos de carga media y, por último, 1 segundo con solo una consulta (que es el mínimo).

En el caso de ThrottledSearchQueriesPercentage, el mínimo, máximo, promedio y total tendrán el mismo valor: el porcentaje de consultas de búsqueda con limitaciones, del número total de consultas de búsqueda durante un minuto.

## <a name="download-and-open-in-visual-studio-code"></a>Descarga y apertura en Visual Studio Code

Puede usar cualquier editor de JSON para ver el archivo de registro. Si no tiene uno, se recomienda [Visual Studio Code](https://code.visualstudio.com/download).

1. En Azure Portal, abra su cuenta de almacenamiento. 

2. En el panel de navegación izquierdo, haga clic en **Blobs**. Verá **insights-logs-operationlogs** e **insights-metrics-pt1m**. Estos contenedores se crean mediante Azure Search cuando se exportan los datos de registro a Blob Storage.

3. Haga clic en la jerarquía de carpetas hasta llegar al archivo .json.  Use el menú contextual para descargar el archivo.

Una vez descargado el archivo, ábralo en un editor de JSON para ver el contenido.

## <a name="use-system-apis"></a>Uso de las API del sistema
La API REST de Azure Search y .NET SDK proporcionan acceso mediante programación a información de métricas de servicio, índice e indexadores y recuentos de documentos.

* [Get Services Statistics](/rest/api/searchservice/get-service-statistics) (Obtención de estadísticas de servicios)
* [Obtención de estadísticas de índice](/rest/api/searchservice/get-index-statistics)
* [Recuento de documentos](/rest/api/searchservice/count-documents)
* [Obtención del estado del indizador](/rest/api/searchservice/get-indexer-status)

Para habilitar el uso de PowerShell o la CLI de Azure, consulte la documentación [aquí](https://docs.microsoft.com/azure/monitoring-and-diagnostics/monitoring-overview-of-diagnostic-logs#how-to-enable-collection-of-diagnostic-logs).

## <a name="next-steps"></a>Pasos siguientes

Consulte [Administración de servicios de Azure Search en Microsoft Azure](search-manage.md) para más información sobre la administración de servicios, o [Rendimiento y optimización](search-performance-optimization.md) para que le sirva de guía de optimización.