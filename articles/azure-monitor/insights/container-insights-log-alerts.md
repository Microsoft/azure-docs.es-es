---
title: Alertas de registro de Azure Monitor para contenedores | Microsoft Docs
description: En este artículo se describe cómo crear alertas personalizadas basadas en consultas de registro para el uso de memoria y de CPU desde Azure Monitor para contenedores.
ms.topic: conceptual
ms.date: 01/07/2020
ms.openlocfilehash: c023471ae041fa524fc4a2164c633ca80bcfdd88
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87095218"
---
# <a name="how-to-create-log-alerts-from-azure-monitor-for-containers"></a>Creación de alertas de registro a partir de Azure Monitor para contenedores

Azure Monitor para contenedores supervisa el rendimiento de las cargas de trabajo de contenedor implementadas en clústeres de Kubernetes administrados o autoadministrados. Para alertar sobre lo que importa, en este artículo se describe cómo crear alertas basadas en registro para las siguientes situaciones con los clústeres de AKS:

- Cuando el uso de CPU o memoria en los nodos del clúster supera un umbral
- Cuando el uso de CPU o memoria en cualquier contenedor dentro de un controlador supera un umbral en comparación con un límite que se establece en el recurso correspondiente
- Recuentos del nodo de estado *NotReady*
- Recuentos de fase de pod *Erróneo*, *Pendiente*, *Desconocido*, *Ejecución* o *Correcto*
- Cuando el espacio libre en disco de los nodos del clúster supera un umbral

Para generar una alerta sobre el uso elevado de CPU o de memoria o sobre poco espacio libre en disco en los nodos del clúster, use las consultas que se proporcionan para crear una alerta de métrica o una alerta de medida de métrica. Mientras que las alertas de métricas tienen una latencia menor que las alertas de registro, estas proporcionan consultas avanzadas y una mayor sofisticación. Las consultas de alertas de registro comparan una fecha y hora con la actual mediante el operador *now* y retroceden una hora. (Azure Monitor para contenedores almacena todas las fechas en formato de Hora universal coordinada [UTC]).

Si no está familiarizado con las alertas en Azure Monitor, consulte [Información general sobre las alertas en Microsoft Azure](../platform/alerts-overview.md) antes de empezar. Para más información acerca de las alertas que usan consultas de registro, consulte [Alertas de registro en Azure Monitor](../platform/alerts-unified-log.md). Para más información acerca de las alertas de métrica consulte [Comprender cómo funcionan las alertas de métricas en Azure Monitor](../platform/alerts-metric-overview.md).

## <a name="resource-utilization-log-search-queries"></a>Consultas de búsqueda de registro de uso de recursos

Las consultas de esta sección admiten todos los escenarios de alertas. Se usan en el paso 7 de la sección [crear alerta](#create-an-alert-rule) de este artículo.

En la siguiente consulta se calcula el uso medio de CPU como la media de uso de CPU de los nodos miembros, cada minuto.  

```kusto
let endDateTime = now();
let startDateTime = ago(1h);
let trendBinSize = 1m;
let capacityCounterName = 'cpuCapacityNanoCores';
let usageCounterName = 'cpuUsageNanoCores';
KubeNodeInventory
| where TimeGenerated < endDateTime
| where TimeGenerated >= startDateTime
// cluster filter would go here if multiple clusters are reporting to the same Log Analytics workspace
| distinct ClusterName, Computer
| join hint.strategy=shuffle (
  Perf
  | where TimeGenerated < endDateTime
  | where TimeGenerated >= startDateTime
  | where ObjectName == 'K8SNode'
  | where CounterName == capacityCounterName
  | summarize LimitValue = max(CounterValue) by Computer, CounterName, bin(TimeGenerated, trendBinSize)
  | project Computer, CapacityStartTime = TimeGenerated, CapacityEndTime = TimeGenerated + trendBinSize, LimitValue
) on Computer
| join kind=inner hint.strategy=shuffle (
  Perf
  | where TimeGenerated < endDateTime + trendBinSize
  | where TimeGenerated >= startDateTime - trendBinSize
  | where ObjectName == 'K8SNode'
  | where CounterName == usageCounterName
  | project Computer, UsageValue = CounterValue, TimeGenerated
) on Computer
| where TimeGenerated >= CapacityStartTime and TimeGenerated < CapacityEndTime
| project ClusterName, Computer, TimeGenerated, UsagePercent = UsageValue * 100.0 / LimitValue
| summarize AggregatedValue = avg(UsagePercent) by bin(TimeGenerated, trendBinSize), ClusterName
```

En la siguiente consulta se calcula el uso medio de memoria como la media de uso de memoria de los nodos miembros, cada minuto.

```kusto
let endDateTime = now();
let startDateTime = ago(1h);
let trendBinSize = 1m;
let capacityCounterName = 'memoryCapacityBytes';
let usageCounterName = 'memoryRssBytes';
KubeNodeInventory
| where TimeGenerated < endDateTime
| where TimeGenerated >= startDateTime
// cluster filter would go here if multiple clusters are reporting to the same Log Analytics workspace
| distinct ClusterName, Computer
| join hint.strategy=shuffle (
  Perf
  | where TimeGenerated < endDateTime
  | where TimeGenerated >= startDateTime
  | where ObjectName == 'K8SNode'
  | where CounterName == capacityCounterName
  | summarize LimitValue = max(CounterValue) by Computer, CounterName, bin(TimeGenerated, trendBinSize)
  | project Computer, CapacityStartTime = TimeGenerated, CapacityEndTime = TimeGenerated + trendBinSize, LimitValue
) on Computer
| join kind=inner hint.strategy=shuffle (
  Perf
  | where TimeGenerated < endDateTime + trendBinSize
  | where TimeGenerated >= startDateTime - trendBinSize
  | where ObjectName == 'K8SNode'
  | where CounterName == usageCounterName
  | project Computer, UsageValue = CounterValue, TimeGenerated
) on Computer
| where TimeGenerated >= CapacityStartTime and TimeGenerated < CapacityEndTime
| project ClusterName, Computer, TimeGenerated, UsagePercent = UsageValue * 100.0 / LimitValue
| summarize AggregatedValue = avg(UsagePercent) by bin(TimeGenerated, trendBinSize), ClusterName
```
>[!IMPORTANT]
>En las consultas siguientes se usan los valores de marcador de posición \<your-cluster-name> y \<your-controller-name> para representar el clúster y el controlador. Reemplácelos por valores específicos del entorno al configurar las alertas.

En la siguiente consulta se calcula el uso medio de CPU de todos los contenedores en un controlador como la media de uso de CPU de cada instancia de contenedor en un controlador, cada minuto. La medida es un porcentaje del límite configurado para un contenedor.

```kusto
let endDateTime = now();
let startDateTime = ago(1h);
let trendBinSize = 1m;
let capacityCounterName = 'cpuLimitNanoCores';
let usageCounterName = 'cpuUsageNanoCores';
let clusterName = '<your-cluster-name>';
let controllerName = '<your-controller-name>';
KubePodInventory
| where TimeGenerated < endDateTime
| where TimeGenerated >= startDateTime
| where ClusterName == clusterName
| where ControllerName == controllerName
| extend InstanceName = strcat(ClusterId, '/', ContainerName),
         ContainerName = strcat(controllerName, '/', tostring(split(ContainerName, '/')[1]))
| distinct Computer, InstanceName, ContainerName
| join hint.strategy=shuffle (
    Perf
    | where TimeGenerated < endDateTime
    | where TimeGenerated >= startDateTime
    | where ObjectName == 'K8SContainer'
    | where CounterName == capacityCounterName
    | summarize LimitValue = max(CounterValue) by Computer, InstanceName, bin(TimeGenerated, trendBinSize)
    | project Computer, InstanceName, LimitStartTime = TimeGenerated, LimitEndTime = TimeGenerated + trendBinSize, LimitValue
) on Computer, InstanceName
| join kind=inner hint.strategy=shuffle (
    Perf
    | where TimeGenerated < endDateTime + trendBinSize
    | where TimeGenerated >= startDateTime - trendBinSize
    | where ObjectName == 'K8SContainer'
    | where CounterName == usageCounterName
    | project Computer, InstanceName, UsageValue = CounterValue, TimeGenerated
) on Computer, InstanceName
| where TimeGenerated >= LimitStartTime and TimeGenerated < LimitEndTime
| project Computer, ContainerName, TimeGenerated, UsagePercent = UsageValue * 100.0 / LimitValue
| summarize AggregatedValue = avg(UsagePercent) by bin(TimeGenerated, trendBinSize) , ContainerName
```

En la siguiente consulta se calcula el uso medio de memoria de todos los contenedores en un controlador como la media de uso de memoria de cada instancia de contenedor en un controlador, cada minuto. La medida es un porcentaje del límite configurado para un contenedor.

```kusto
let endDateTime = now();
let startDateTime = ago(1h);
let trendBinSize = 1m;
let capacityCounterName = 'memoryLimitBytes';
let usageCounterName = 'memoryRssBytes';
let clusterName = '<your-cluster-name>';
let controllerName = '<your-controller-name>';
KubePodInventory
| where TimeGenerated < endDateTime
| where TimeGenerated >= startDateTime
| where ClusterName == clusterName
| where ControllerName == controllerName
| extend InstanceName = strcat(ClusterId, '/', ContainerName),
         ContainerName = strcat(controllerName, '/', tostring(split(ContainerName, '/')[1]))
| distinct Computer, InstanceName, ContainerName
| join hint.strategy=shuffle (
    Perf
    | where TimeGenerated < endDateTime
    | where TimeGenerated >= startDateTime
    | where ObjectName == 'K8SContainer'
    | where CounterName == capacityCounterName
    | summarize LimitValue = max(CounterValue) by Computer, InstanceName, bin(TimeGenerated, trendBinSize)
    | project Computer, InstanceName, LimitStartTime = TimeGenerated, LimitEndTime = TimeGenerated + trendBinSize, LimitValue
) on Computer, InstanceName
| join kind=inner hint.strategy=shuffle (
    Perf
    | where TimeGenerated < endDateTime + trendBinSize
    | where TimeGenerated >= startDateTime - trendBinSize
    | where ObjectName == 'K8SContainer'
    | where CounterName == usageCounterName
    | project Computer, InstanceName, UsageValue = CounterValue, TimeGenerated
) on Computer, InstanceName
| where TimeGenerated >= LimitStartTime and TimeGenerated < LimitEndTime
| project Computer, ContainerName, TimeGenerated, UsagePercent = UsageValue * 100.0 / LimitValue
| summarize AggregatedValue = avg(UsagePercent) by bin(TimeGenerated, trendBinSize) , ContainerName
```

En la siguiente consulta se devuelven todos los nodos y recuentos que tienen un estado de *Ready* y *NotReady*.

```kusto
let endDateTime = now();
let startDateTime = ago(1h);
let trendBinSize = 1m;
let clusterName = '<your-cluster-name>';
KubeNodeInventory
| where TimeGenerated < endDateTime
| where TimeGenerated >= startDateTime
| distinct ClusterName, Computer, TimeGenerated
| summarize ClusterSnapshotCount = count() by bin(TimeGenerated, trendBinSize), ClusterName, Computer
| join hint.strategy=broadcast kind=inner (
    KubeNodeInventory
    | where TimeGenerated < endDateTime
    | where TimeGenerated >= startDateTime
    | summarize TotalCount = count(), ReadyCount = sumif(1, Status contains ('Ready'))
                by ClusterName, Computer,  bin(TimeGenerated, trendBinSize)
    | extend NotReadyCount = TotalCount - ReadyCount
) on ClusterName, Computer, TimeGenerated
| project   TimeGenerated,
            ClusterName,
            Computer,
            ReadyCount = todouble(ReadyCount) / ClusterSnapshotCount,
            NotReadyCount = todouble(NotReadyCount) / ClusterSnapshotCount
| order by ClusterName asc, Computer asc, TimeGenerated desc
```
En la siguiente consulta se devuelven recuentos de fase de pod basados en todas las fases: *Erróneo*, *Pendiente*, *Desconocido*, *Ejecución* o *Correcto*.  

```kusto
let endDateTime = now();
    let startDateTime = ago(1h);
    let trendBinSize = 1m;
    let clusterName = '<your-cluster-name>';
    KubePodInventory
    | where TimeGenerated < endDateTime
    | where TimeGenerated >= startDateTime
    | where ClusterName == clusterName
    | distinct ClusterName, TimeGenerated
    | summarize ClusterSnapshotCount = count() by bin(TimeGenerated, trendBinSize), ClusterName
    | join hint.strategy=broadcast (
        KubePodInventory
        | where TimeGenerated < endDateTime
        | where TimeGenerated >= startDateTime
        | distinct ClusterName, Computer, PodUid, TimeGenerated, PodStatus
        | summarize TotalCount = count(),
                    PendingCount = sumif(1, PodStatus =~ 'Pending'),
                    RunningCount = sumif(1, PodStatus =~ 'Running'),
                    SucceededCount = sumif(1, PodStatus =~ 'Succeeded'),
                    FailedCount = sumif(1, PodStatus =~ 'Failed')
                 by ClusterName, bin(TimeGenerated, trendBinSize)
    ) on ClusterName, TimeGenerated
    | extend UnknownCount = TotalCount - PendingCount - RunningCount - SucceededCount - FailedCount
    | project TimeGenerated,
              TotalCount = todouble(TotalCount) / ClusterSnapshotCount,
              PendingCount = todouble(PendingCount) / ClusterSnapshotCount,
              RunningCount = todouble(RunningCount) / ClusterSnapshotCount,
              SucceededCount = todouble(SucceededCount) / ClusterSnapshotCount,
              FailedCount = todouble(FailedCount) / ClusterSnapshotCount,
              UnknownCount = todouble(UnknownCount) / ClusterSnapshotCount
| summarize AggregatedValue = avg(PendingCount) by bin(TimeGenerated, trendBinSize)
```

>[!NOTE]
>Para generar una alerta en ciertas fases de pod, tales como *Pendiente*, *Erróneo* o *Desconocido*, modifique la última línea de la consulta. Por ejemplo, para generar alertas sobre *FailedCount*, use: <br/>`| summarize AggregatedValue = avg(FailedCount) by bin(TimeGenerated, trendBinSize)`

En la consulta siguiente se devuelven los discos de los nodos de clúster que superan el 90 % del espacio libre utilizado. Para obtener el id. de clúster, primero ejecute la siguiente consulta y copie el valor de la propiedad `ClusterId`:

```kusto
InsightsMetrics
| extend Tags = todynamic(Tags)            
| project ClusterId = Tags['container.azm.ms/clusterId']   
| distinct tostring(ClusterId)   
``` 

```kusto
let clusterId = '<cluster-id>';
let endDateTime = now();
let startDateTime = ago(1h);
let trendBinSize = 1m;
InsightsMetrics
| where TimeGenerated < endDateTime
| where TimeGenerated >= startDateTime
| where Origin == 'container.azm.ms/telegraf'            
| where Namespace == 'container.azm.ms/disk'            
| extend Tags = todynamic(Tags)            
| project TimeGenerated, ClusterId = Tags['container.azm.ms/clusterId'], Computer = tostring(Tags.hostName), Device = tostring(Tags.device), Path = tostring(Tags.path), DiskMetricName = Name, DiskMetricValue = Val   
| where ClusterId =~ clusterId       
| where DiskMetricName == 'used_percent'
| summarize AggregatedValue = max(DiskMetricValue) by bin(TimeGenerated, trendBinSize)
| where AggregatedValue >= 90
```

## <a name="create-an-alert-rule"></a>Crear una regla de alerta

Esta sección le guía a través de la creación de una regla de alerta de medición de métricas mediante datos de rendimiento de Azure Monitor para contenedores. Puede usar este proceso básico con diversas consultas de registro para alertar sobre diferentes contadores de rendimiento. Use una de las consultas de búsqueda de registros proporcionadas anteriormente para comenzar. Para crearla mediante una plantilla de ARM, consulte [Creación de una alerta de registro de ejemplo mediante la plantilla de Azure Resource Manager](../platform/alerts-log.md#sample-log-alert-creation-using-azure-resource-template).

>[!NOTE]
>En el siguiente procedimiento para crear una regla de alerta para el uso de recursos de contenedor, deberá cambiar a una nueva API de alertas de registro, tal como se describe en [Cambio de la preferencia de API para las alertas de registro](../platform/alerts-log-api-switch.md).
>

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
2. En Azure Portal, busque y seleccione **Áreas de trabajo de Log Analytics**.
3. En la lista de áreas de trabajo de Log Analytics, seleccione el área de trabajo que admite Azure Monitor para contenedores. 
4. En el panel de la izquierda, seleccione **Registros** para abrir la página de registros de Azure Monitor. Utilice esta página para escribir y ejecutar consultas de registro de Azure.
5. En la página **Registros**, pegue una de las [consultas](#resource-utilization-log-search-queries) proporcionadas anteriormente en el campo **Consulta de búsqueda** y luego seleccione **Ejecutar** para validar los resultados. Si no realiza este paso, la opción **Nueva alerta** no está disponible para la selección.
6. Seleccione **Nueva alerta** para crear una alerta de registro.
7. En la sección **Condición**, seleccione la condición de registro personalizada predefinida **Cada vez que la búsqueda de registros personalizada sea \<logic undefined>** . El tipo de señal de **búsqueda de registros personalizada** se selecciona automáticamente porque estamos creando una regla de alertas directamente desde la página de registros de Azure Monitor.  
8. Pegue cualquiera de las [consultas](#resource-utilization-log-search-queries) proporcionadas con anterioridad en el campo **Consulta de búsqueda**.
9. Configure la alerta de la manera siguiente:

    1. En la lista desplegable **Basado en**, seleccione **Unidades métricas**. Una medida de métrica crea una alerta por cada objeto de la consulta con un valor por encima del umbral especificado.
    1. En **Condición**, seleccione **Mayor que** y escriba **75** como un **umbral** de línea de base inicial para las alertas de uso de CPU y memoria. Para la alerta de poco espacio en disco, escriba **90**. O escriba un valor diferente que satisfaga sus criterios.
    1. En la sección **Desencadenar alerta según**, seleccione **Infracciones consecutivas**. En la lista desplegable, seleccione **Mayor que** y escriba **2**.
    1. Para configurar una alerta de uso de CPU o de memoria de contenedor, en **Agregado en**, seleccione **ContainerName**. Para configurar la alerta de poco espacio en disco de nodo de clúster, seleccione **ClusterId**.
    1. En la sección **Se evaluó basándose en**, establezca el valor de **Período** en **60 minutos**. La regla se ejecutará cada cinco minutos y devolverá los registros que se crearon dentro de la última hora a partir del momento actual. El establecimiento del período de tiempo en una ventana amplia considera la latencia de datos potencial. También garantiza que la consulta vaya a devolver datos para evitar un falso negativo en el que la alerta nunca se active.

10. Seleccione **Listo** para completar la regla de alertas.
11. Escriba un nombre en el campo **Nombre de la regla de alertas**. Especifique una **Descripción** que proporcione los detalles sobre la alerta. Y seleccione un nivel de gravedad adecuado en las opciones proporcionadas.
12. Para activar inmediatamente la regla de alertas, acepte el valor predeterminado de **Habilitar regla tras la creación**.
13. Seleccione un **Grupo de acciones** existente o cree uno nuevo. Este paso garantiza que se realizan las mismas acciones cada vez que se desencadena una alerta. Configure en función de cómo el equipo de operaciones de TI o DevOps administra los incidentes.
14. Seleccione **Crear regla de alertas** para finalizar la regla de alertas. Se iniciará la ejecución de inmediato.

## <a name="next-steps"></a>Pasos siguientes

- En los [ejemplos de consultas de registro](container-insights-log-search.md#search-logs-to-analyze-data) encontrará consultas predefinidas y ejemplos para evaluar o personalizar las alertas, la visualización o el análisis de los clústeres.

- Para más información sobre Azure Monitor y cómo supervisar otros aspectos del clúster de Kubernetes, consulte [Visualización del rendimiento del clúster de Kubernetes](container-insights-analyze.md) y [Visualización del estado del clúster de Kubernetes](container-insights-health.md).
