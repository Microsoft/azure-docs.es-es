---
title: Ejecución de tareas en paralelo para optimizar los recursos de proceso
description: Aumente la eficiencia y reduzca los costos usando menos nodos de proceso y ejecutando tareas simultáneas en cada nodo de un grupo de Azure Batch
ms.topic: how-to
ms.date: 04/17/2019
ms.custom: H1Hack27Feb2017, devx-track-csharp
ms.openlocfilehash: e4c98244755cae7a606ebe26cbadef53ca5fd922
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88926293"
---
# <a name="run-tasks-concurrently-to-maximize-usage-of-batch-compute-nodes"></a>Ejecución simultánea de tareas para maximizar el uso de los nodos de proceso de Batch 

A través de la ejecución simultánea de más de una tarea en cada nodo de proceso dentro del grupo de Azure Batch, puede maximizar el uso de recursos en un menor número de nodos en el grupo. Para algunas cargas de trabajo, esto puede reducir los costos y el tiempo dedicado al trabajo.

Aunque en algunos casos puede resultar beneficioso que todos los recursos de un nodo estén dedicados a una sola tarea, en otras situaciones será conveniente permitir que varias tareas compartan esos recursos:

* **Minimizar la transferencia de datos** cuando las tareas son capaces de compartir datos. En este escenario, puede reducir considerablemente los gastos de transferencia de datos copiando datos compartidos en un número menor de nodos y ejecutando tareas en paralelo en cada nodo. Esto es válido especialmente si los datos que se copian en cada nodo deben transferirse entre regiones geográficas.
* **Maximizar el uso de memoria** cuando las tareas requieren una gran cantidad de memoria, pero solo durante períodos breves y en momentos variables durante la ejecución. Puede emplear menos nodos de ejecución, pero de mayor tamaño y con más memoria, para controlar de forma eficaz dichos aumentos. Estos nodos tendrían varias tareas ejecutándose en paralelo en cada nodo, pero cada tarea aprovecharía la abundante memoria de los nodos en distintos momentos.
* **Mitigar los límites de número de nodos** cuando se requiere la comunicación entre nodos dentro de un grupo. Actualmente, los grupos configurados para la comunicación entre nodos están limitados a 50 nodos de ejecución. Si cada uno de los nodos de este tipo de grupo es capaz de ejecutar tareas en paralelo, el número de tareas que se podrán ejecutar simultáneamente será mayor.
* **Replicar en un clúster de proceso local**, como cuando mueve por primera vez un entorno de procesos a Azure. Si la solución local existente ejecuta varias tareas en cada nodo de proceso, puede aumentar el número máximo de tareas de nodo para que refleje con mayor fidelidad esa configuración.

## <a name="example-scenario"></a>Escenario de ejemplo
Para ilustrar las ventajas de la ejecución de tareas en paralelo, imaginemos que la aplicación de la tarea tiene unos requisitos de CPU y memoria que hacen que el tamaño de nodo [Standard\_StandardD1](../cloud-services/cloud-services-sizes-specs.md) sea suficiente. Pero, para ejecutar el trabajo en el tiempo requerido, se necesitan 1.000 nodos de ese tipo.

En lugar de utilizar nodos Standard\_D1 con un núcleo de CPU, podría utilizar nodos [Standard\_D14](../cloud-services/cloud-services-sizes-specs.md) con 16 núcleos en cada nodo y habilitar la ejecución de tareas en paralelo. En este caso, se podría usar un *número de nodos 16 veces menor* ; es decir, en lugar de 1000 nodos, solo serían necesarios 63. Además, si para cada nodo son necesarios datos de referencia o archivos de aplicación de gran tamaño, la eficiencia y la duración del trabajo también se mejoran, ya que los datos se copian en solo 63 nodos.

## <a name="enable-parallel-task-execution"></a>Habilitación de la ejecución en paralelo de tareas
Los nodos de proceso para la ejecución en paralelo de tareas se configuran a nivel de grupo. Con la biblioteca de Batch para .NET, establezca la propiedad [CloudPool.MaxTasksPerComputeNode][maxtasks_net] al crear un grupo. Si usa la API REST de Batch, establezca el elemento [maxTasksPerNode][rest_addpool] en el cuerpo de la solicitud durante la creación del grupo.

Azure Batch permite una configuración de tareas por nodo que cuadriplica el número de nodos de núcleo. Por ejemplo, si el grupo está configurado con nodos de tamaño "Grande" (cuatro núcleos), `maxTasksPerNode` se puede establecer en 16. Sin embargo, independientemente de cuántos núcleos tiene el nodo, no puede tener más de 256 tareas por nodo. Para más información sobre el número de núcleos de cada uno de los tamaños de nodo, consulte [Tamaños de Cloud Services](../cloud-services/cloud-services-sizes-specs.md). Para más información sobre los límites del servicio, consulte [Cuotas y límites del servicio Azure Batch](batch-quota-limit.md).

> [!TIP]
> Asegúrese de tener en cuenta el valor `maxTasksPerNode` al construir una [fórmula de escalado automático][enable_autoscaling] para el grupo. Por ejemplo, una fórmula que evalúe `$RunningTasks` podría verse afectada considerablemente por un aumento en las tareas por nodo. Consulte [Escalación automática de los nodos de ejecución en un grupo de Azure Batch](batch-automatic-scaling.md) para obtener más información.
>
>

## <a name="distribution-of-tasks"></a>Distribución de tareas
Cuando los nodos de proceso dentro de un grupo pueden ejecutar tareas de forma simultánea, es importante especificar cómo desea que se distribuyan las tareas entre los nodos del grupo.

Mediante la propiedad [CloudPool.TaskSchedulingPolicy][task_schedule], puede especificar que las tareas se deberían asignar de manera uniforme entre todos los nodos del grupo ("propagación"). O bien, puede especificar que se deberían asignar todas las tareas posibles a cada nodo antes de asignarlas a otro nodo del grupo ("empaquetado").

Como ejemplo de por qué esta característica es importante, considere el grupo de nodos [Standard\_D14](../cloud-services/cloud-services-sizes-specs.md) (en el ejemplo anterior) configurado con un valor [CloudPool.MaxTasksPerComputeNode][maxtasks_net] de 16. Si [CloudPool.TaskSchedulingPolicy][task_schedule] se configura con un [ComputeNodeFillType][fill_type] de *Pack*, se podría maximizar el uso de los 16 núcleos de cada nodo y permitir que un [grupo con escalado automático](batch-automatic-scaling.md) elimine los nodos sin usar del grupo (nodos sin tareas asignadas). Esto minimiza el uso de recursos y permite ahorrar dinero.

## <a name="batch-net-example"></a>Ejemplo de Batch .NET
En este fragmento de código de la API de [Batch para .NET][api_net], se muestra una solicitud para crear un grupo que contiene cuatro nodos con un máximo de cuatro tareas por nodo. Se especifica una directiva de programación de tareas que llenará cada nodo de tareas antes de asignarlas a otro nodo del grupo. Para más información sobre cómo agregar grupos mediante la API de .NET para Batch, consulte [BatchClient.PoolOperations.CreatePool][poolcreate_net].

```csharp
CloudPool pool =
    batchClient.PoolOperations.CreatePool(
        poolId: "mypool",
        targetDedicatedComputeNodes: 4
        virtualMachineSize: "standard_d1_v2",
        cloudServiceConfiguration: new CloudServiceConfiguration(osFamily: "5"));

pool.MaxTasksPerComputeNode = 4;
pool.TaskSchedulingPolicy = new TaskSchedulingPolicy(ComputeNodeFillType.Pack);
pool.Commit();
```

## <a name="batch-rest-example"></a>Ejemplo de REST Batch
En este fragmento de la API [REST de Batch][api_rest], se muestra una solicitud para crear un grupo que contiene dos nodos de gran tamaño con un máximo de cuatro tareas por nodo. Para más información sobre cómo agregar grupos mediante la API REST, consulte [Agregar un grupo a una cuenta][rest_addpool].

```json
{
  "odata.metadata":"https://myaccount.myregion.batch.azure.com/$metadata#pools/@Element",
  "id":"mypool",
  "vmSize":"large",
  "cloudServiceConfiguration": {
    "osFamily":"4",
    "targetOSVersion":"*",
  }
  "targetDedicatedComputeNodes":2,
  "maxTasksPerNode":4,
  "enableInterNodeCommunication":true,
}
```

> [!NOTE]
> Solo puede establecer el elemento `maxTasksPerNode` y la propiedad [MaxTasksPerComputeNode][maxtasks_net] en el momento de crear el grupo. No se pueden modificar después de haberlos creado.
>
>

## <a name="code-sample"></a>Código de ejemplo
El proyecto [ParallelNodeTasks][parallel_tasks_sample] en GitHub muestra el uso de la propiedad [CloudPool.MaxTasksPerComputeNode][maxtasks_net].

Esta aplicación de consola de C# utiliza la biblioteca de [.NET de Batch][api_net] para crear un grupo con uno o más nodos de proceso. Ejecuta un número configurable de tareas en esos nodos para simular una carga variable. Los resultados de la aplicación especifican qué nodos han ejecutado cada tarea. La aplicación también proporciona un resumen de los parámetros de trabajo y la duración. Abajo se muestra la parte de resumen de los resultados de dos ejecuciones diferentes de la aplicación de ejemplo.

```
Nodes: 1
Node size: large
Max tasks per node: 1
Tasks: 32
Duration: 00:30:01.4638023
```

La primera ejecución de la aplicación de ejemplo muestra que, con un solo nodo en el grupo y la configuración predeterminada de una tarea por nodo, la duración del trabajo es superior a 30 minutos.

```
Nodes: 1
Node size: large
Max tasks per node: 4
Tasks: 32
Duration: 00:08:48.2423500
```

La segunda ejecución del ejemplo muestra una disminución notable en la duración del trabajo. Esto se debe a que el grupo se configuró con cuatro tareas por nodo, lo que permite la ejecución en paralelo de tareas de forma que el trabajo se completa en casi una cuarta parte del tiempo.

> [!NOTE]
> La duración del trabajo en los resúmenes anteriores no incluye el tiempo de creación del grupo. Cada uno de los trabajos anteriores se envió a grupos ya creados cuyos nodos de proceso se encontraban en el estado *inactivo* en el momento del envío.
>
>

## <a name="next-steps"></a>Pasos siguientes
### <a name="batch-explorer-heat-map"></a>Mapa térmico de Batch Explorer
[Batch Explorer][batch_labs] es una herramienta de cliente independiente, completa y gratuita que puede ayudarle a crear, depurar y supervisar aplicaciones de Azure Batch. Batch Explorer contiene una característica de *mapa térmico* que permite visualizar la ejecución de las tareas. Cuando ejecuta la aplicación de ejemplo [ParallelTasks][parallel_tasks_sample], puede usar la característica Mapa térmico para ver fácilmente la ejecución de tareas paralelas en cada nodo.


[api_net]: /dotnet/api/microsoft.azure.batch
[api_rest]: /rest/api/batchservice/
[batch_labs]: https://azure.github.io/BatchExplorer/
[cloudpool]: /dotnet/api/microsoft.azure.batch.cloudpool
[enable_autoscaling]: /rest/api/batchservice/pool/enableautoscale
[fill_type]: /dotnet/api/microsoft.azure.batch.common.computenodefilltype
[github_samples]: https://github.com/Azure/azure-batch-samples
[maxtasks_net]: /dotnet/api/microsoft.azure.batch.cloudpool
[rest_addpool]: /rest/api/batchservice/pool/add
[parallel_tasks_sample]: https://github.com/Azure/azure-batch-samples/tree/master/CSharp/ArticleProjects/ParallelTasks
[poolcreate_net]: /dotnet/api/microsoft.azure.batch.pooloperations
[task_schedule]: /dotnet/api/microsoft.azure.batch.cloudpool

