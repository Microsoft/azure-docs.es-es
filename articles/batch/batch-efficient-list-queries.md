---
title: Diseño de consultas de lista eficaces
description: Mejore el rendimiento filtrando las consultas cuando solicite información en recursos de Batch, como grupos, trabajos, tareas y nodos de proceso.
ms.topic: how-to
ms.date: 06/18/2020
ms.custom: seodec18, devx-track-csharp
ms.openlocfilehash: 3a767cc8ae3c8c48e1e40e0735c33fa807ba0015
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88933521"
---
# <a name="create-queries-to-list-batch-resources-efficiently"></a>Creación de consultas para enumerar los recursos de Batch con eficacia

Casi todas las aplicaciones que usan Batch realizan algún tipo de supervisión u otra operación que consulta el servicio Batch, a menudo a intervalos regulares. Por ejemplo, para determinar si quedan tareas en cola en un trabajo, debe obtener información de cada una de las tareas del trabajo. Para determinar el estado de los nodos de un grupo, tiene que obtener los datos de cada nodo del grupo. En este artículo, se explica cómo ejecutar este tipo de consultas de la manera más eficaz.

Puede mejorar el rendimiento de la aplicación de Azure Batch si reduce la cantidad de datos que devuelve el servicio al consultar trabajos, tareas, nodos de ejecución y otros recursos mediante la biblioteca de [.NET de Batch](/dotnet/api/microsoft.azure.batch).

> [!NOTE]
> El servicio Batch proporciona compatibilidad con la API para los escenarios comunes de recuento de tareas en un trabajo y recuento de nodos de ejecución en el grupo de Batch. En lugar de usar una consulta de lista para ello, puede llamar a las operaciones [Get Task Counts](/rest/api/batchservice/job/gettaskcounts) y [List Pool Node Counts](/rest/api/batchservice/account/listpoolnodecounts). Estas operaciones son más eficaces que una consulta de lista, pero devuelven información más limitada que quizás no esté siempre actualizada. Para más información, consulte el tema de [recuento de las tareas y los nodos de ejecución por estado](batch-get-resource-counts.md).

## <a name="specify-a-detail-level"></a>Especificación de un nivel de detalle

En una aplicación de Batch de producción, algunas entidades como los trabajos, las tareas y los nodos de ejecución pueden contarse por millares. Cuando solicita información sobre estos recursos, es posible que cada consulta envíe un gran volumen de información desde el servicio de Batch hasta la aplicación. Si limita el número de elementos y el tipo de información que va a devolver una consulta, podrá acelerar las consultas y, por tanto, mejorar el rendimiento de la aplicación.

Este fragmento de código de la API de [.NET de Batch](/dotnet/api/microsoft.azure.batch) enumera *todas* las tareas asociadas a un trabajo, junto con *todas* las propiedades de cada tarea:

```csharp
// Get a collection of all of the tasks and all of their properties for job-001
IPagedEnumerable<CloudTask> allTasks =
    batchClient.JobOperations.ListTasks("job-001");
```

Sin embargo, puede realizar una consulta de lista mucho más eficiente, aplicando un "nivel de detalle" a la consulta. Para ello, puede suministrar un objeto [ODATADetailLevel](/dotnet/api/microsoft.azure.batch.odatadetaillevel) al método [JobOperations.ListTasks](/dotnet/api/microsoft.azure.batch.joboperations). Este fragmento de código devuelve solo las propiedades de identificador, línea de comandos e información de nodo de proceso para las tareas completadas:

```csharp
// Configure an ODATADetailLevel specifying a subset of tasks and
// their properties to return
ODATADetailLevel detailLevel = new ODATADetailLevel();
detailLevel.FilterClause = "state eq 'completed'";
detailLevel.SelectClause = "id,commandLine,nodeInfo";

// Supply the ODATADetailLevel to the ListTasks method
IPagedEnumerable<CloudTask> completedTasks =
    batchClient.JobOperations.ListTasks("job-001", detailLevel);
```

En este caso de ejemplo, si hay miles de tareas en el trabajo, los resultados de la segunda consulta normalmente se devolverán más rápido que en la primera consulta. [A continuación](#efficient-querying-in-batch-net)encontrará más información sobre el uso de ODATADetailLevel al enumerar elementos con la API de Batch de .NET.

> [!IMPORTANT]
> Es muy recomendable suministrar siempre un objeto ODATADetailLevel a las llamadas de la lista de la API de .NET para garantizar una eficacia y un rendimiento máximos de la aplicación. La especificación de un nivel de detalle puede ayudarle a reducir los tiempos de respuesta del servicio de Batch, a mejorar la utilización de la red y a minimizar el uso de la memoria por parte de las aplicaciones cliente.

## <a name="filter-select-and-expand"></a>Filtrado, selección y expansión

Las API de [Batch para .NET](/dotnet/api/microsoft.azure.batch) y de [REST de Batch](/rest/api/batchservice/) ofrecen la posibilidad de reducir tanto el número de elementos que se devuelven en una lista como la cantidad de información que se devuelve para cada uno de ellos. Para ello, debe especificar **filter**, **select** y **expand strings** cuando realice consultas de lista.

### <a name="filter"></a>Filter

La cadena filter es una expresión que reduce el número de elementos que se devuelven. Por ejemplo, puede mostrar solo las tareas en ejecución de un trabajo o solo los nodos de ejecución que estén listos para ejecutar tareas.

La cadena filter se compone de una o varias expresiones, una de la cuales consta de un nombre de propiedad, un operador y un valor. Las propiedades que se pueden indicar son específicas de cada tipo de entidad que consulta, al igual que los operadores compatibles con cada propiedad.  Puede combinar varias expresiones con los operadores lógicos `and` y `or`.

Esta cadena filter de ejemplo solo muestra las tareas "render" en ejecución: `(state eq 'running') and startswith(id, 'renderTask')`.

### <a name="select"></a>Seleccionar

La cadena select limita los valores de propiedad que se devuelven para cada elemento. Especifique una lista de nombres de propiedad separados por comas y solo se devolverán los valores de propiedad de los elementos de los resultados de la consulta. Puede especificar cualquiera de las propiedades para el tipo de entidad que está consultando.

Esta cadena select de ejemplo especifica que solo se deben devolver tres valores de propiedades para cada tarea: `id, state, stateTransitionTime`.

### <a name="expand"></a>Expanda

La cadena expand reduce el número de llamadas API necesarias para obtener determinada información. Cuando se utiliza una cadena expand, se puede obtener más información sobre cada elemento con una sola llamada API. En lugar de obtener primero la lista de entidades para después solicitar información de cada elemento de la lista, use una cadena expand para obtener la misma información en una sola llamada API, lo que le ayudará a mejorar el rendimiento al reducir el número de llamadas API.

De forma similar a la cadena select, la cadena expand controla si se incluyen determinados datos en los resultados de la consulta de lista. Cuando se requieren todas las propiedades y no se especifica ninguna cadena select, se *debe* usar la cadena expand para obtener información estadística. Si se usa una cadena select para obtener un subconjunto de propiedades, se puede especificar `stats` en la cadena select y no es preciso especificar la cadena expand.

La cadena expand solo se admite cuando se usa para enumerar trabajo, programaciones de trabajos, tareas y grupos. En la actualidad, solo admite información estadística.

Esta cadena expand de ejemplo especifica que se debe devolver información estadística de cada elemento de la lista: `stats`.

> [!NOTE]
> Al construir cualquiera de los tres tipos de cadena de consulta (filter, select y expand), debe asegurarse de que los nombres de las propiedades y el uso de mayúsculas y minúsculas coinciden con los elementos homólogos de la API de REST. Por ejemplo, si trabaja con la clase .NET [CloudTask](/dotnet/api/microsoft.azure.batch.cloudtask), debe especificar **state** en lugar de **State**, aunque la propiedad de .NET sea [CloudTask.State](/dotnet/api/microsoft.azure.batch.cloudtask.state#Microsoft_Azure_Batch_CloudTask_State). Consulte en las tablas siguientes las asignaciones de propiedad entre la API de REST y la de .NET.

### <a name="rules-for-filter-select-and-expand-strings"></a>Reglas de las cadenas filter, select y expand

- Los nombres de propiedades de las cadenas filter, select y expand deben aparecer exactamente igual que en la API de [REST de Batch](/rest/api/batchservice/), aunque se use [Batch para .NET](/dotnet/api/microsoft.azure.batch) o uno de los SDK de Batch.
- Todos los nombres de propiedades distinguen mayúsculas y minúsculas, pero los valores de propiedad no lo hacen.
- Las cadenas de fecha y hora pueden tener uno de los dos formatos y deben ir precedidas por `DateTime`.
  
  - Ejemplo de formato W3C-DTF: `creationTime gt DateTime'2011-05-08T08:49:37Z'`
  - Ejemplo de formato RFC 1123: `creationTime gt DateTime'Sun, 08 May 2011 08:49:37 GMT'`
- Las cadenas booleanas son `true` o `false`.
- Si se especifica una propiedad o un operador no válidos, aparecerá el error `400 (Bad Request)` .

## <a name="efficient-querying-in-batch-net"></a>Consultas eficaces en .NET de Batch

En la API de [.NET de Batch](/dotnet/api/microsoft.azure.batch), se usa la clase [ODATADetailLevel](/dotnet/api/microsoft.azure.batch.odatadetaillevel) para suministrar cadenas filter, select y expand a las operaciones de lista. La clase ODataDetailLevel tiene tres propiedades de cadena públicas que se pueden especificar en el constructor, o bien establecerse directamente en el objeto. Luego el objeto ODataDetailLevel pasa como parámetro a las distintas operaciones de lista como [ListPools](/dotnet/api/microsoft.azure.batch.pooloperations), [ListJobs](/dotnet/api/microsoft.azure.batch.joboperations) y [ListTasks](/dotnet/api/microsoft.azure.batch.joboperations).

- [ODATADetailLevel.FilterClause](/dotnet/api/microsoft.azure.batch.odatadetaillevel.filterclause): limita el número de elementos que se devuelven.
- [ODATADetailLevel.SelectClause](/dotnet/api/microsoft.azure.batch.odatadetaillevel.selectclause): especifica los valores de propiedad devueltos con cada elemento.
- [ODATADetailLevel.ExpandClause](/dotnet/api/microsoft.azure.batch.odatadetaillevel.expandclause): recupera datos para todos los elementos en una sola llamada a la API en lugar de en llamadas independientes para cada elemento.

El siguiente fragmento de código usa la API de .NET de Batch para consultar de forma eficaz las estadísticas de un conjunto específico de grupos en el servicio Batch. En este escenario, el usuario de Batch tiene grupos de prueba y de producción. Los identificadores del grupo de prueba tienen el prefijo "test", mientras que los del grupo de producción tienen el prefijo "prod". En el fragmento de código, *myBatchClient* es una instancia de la clase [BatchClient](/dotnet/api/microsoft.azure.batch.batchclient) inicializada correctamente.

```csharp
// First we need an ODATADetailLevel instance on which to set the filter, select,
// and expand clause strings
ODATADetailLevel detailLevel = new ODATADetailLevel();

// We want to pull only the "test" pools, so we limit the number of items returned
// by using a FilterClause and specifying that the pool IDs must start with "test"
detailLevel.FilterClause = "startswith(id, 'test')";

// To further limit the data that crosses the wire, configure the SelectClause to
// limit the properties that are returned on each CloudPool object to only
// CloudPool.Id and CloudPool.Statistics
detailLevel.SelectClause = "id, stats";

// Specify the ExpandClause so that the .NET API pulls the statistics for the
// CloudPools in a single underlying REST API call. Note that we use the pool's
// REST API element name "stats" here as opposed to "Statistics" as it appears in
// the .NET API (CloudPool.Statistics)
detailLevel.ExpandClause = "stats";

// Now get our collection of pools, minimizing the amount of data that is returned
// by specifying the detail level that we configured above
List<CloudPool> testPools =
    await myBatchClient.PoolOperations.ListPools(detailLevel).ToListAsync();
```

> [!TIP]
> Para limitar la cantidad de datos devueltos, también se puede pasar una instancia de [ODATADetailLevel](/dotnet/api/microsoft.azure.batch.odatadetaillevel) configurada con las cláusulas Select y Expand a los métodos Get apropiados, como [PoolOperations.GetPool](/dotnet/api/microsoft.azure.batch.pooloperations.getpool#Microsoft_Azure_Batch_PoolOperations_GetPool_System_String_Microsoft_Azure_Batch_DetailLevel_System_Collections_Generic_IEnumerable_Microsoft_Azure_Batch_BatchClientBehavior__).

## <a name="batch-rest-to-net-api-mappings"></a>Asignaciones de la API de Batch de REST y la de .NET

Los nombres de propiedades en las cadenas filter, select y expand deben reflejar sus homólogos en la API de REST, tanto en el nombre en sí como en el uso de mayúsculas y minúsculas. En las tablas siguientes, se proporcionan asignaciones entre los homólogos de la API de .NET y la de REST.

### <a name="mappings-for-filter-strings"></a>Asignaciones de las cadenas filter

- **Métodos de lista de .NET**: todos los métodos de la API de .NET de esta columna aceptan un objeto [ODATADetailLevel](/dotnet/api/microsoft.azure.batch.odatadetaillevel) como parámetro.
- **Solicitudes de lista de REST**: todas las páginas de la API REST de esta columna contienen una tabla que especifica las propiedades y operaciones permitidas en las cadenas *filter*. Estos nombres de propiedad y estas operaciones se usan al construir una cadena [ODATADetailLevel.FilterClause](/dotnet/api/microsoft.azure.batch.odatadetaillevel.filterclause).

| Métodos de lista de .NET | Solicitudes de lista de REST |
| --- | --- |
| [CertificateOperations.ListCertificates](/dotnet/api/microsoft.azure.batch.certificateoperations) |[Enumeración de los certificados de una cuenta](/rest/api/batchservice/certificate/list) |
| [CloudTask.ListNodeFiles](/dotnet/api/microsoft.azure.batch.cloudtask) |[Enumeración de los archivos asociados a una tarea](/rest/api/batchservice/file/listfromtask) |
| [JobOperations.ListJobPreparationAndReleaseTaskStatus](/dotnet/api/microsoft.azure.batch.joboperations) |[Enumeración del estado de la preparación de trabajo y las tareas de la versión de trabajo de un trabajo](/rest/api/batchservice/job/listpreparationandreleasetaskstatus) |
| [JobOperations.ListJobs](/dotnet/api/microsoft.azure.batch.joboperations) |[Enumeración de los trabajos de una cuenta](/rest/api/batchservice/job/list) |
| [JobOperations.ListNodeFiles](/dotnet/api/microsoft.azure.batch.joboperations) |[Enumeración de los archivos de un nodo](/rest/api/batchservice/file/listfromcomputenode) |
| [JobOperations.ListTasks](/dotnet/api/microsoft.azure.batch.joboperations) |[Enumeración de las tareas asociadas a un trabajo](/rest/api/batchservice/task/list) |
| [JobScheduleOperations.ListJobSchedules](/dotnet/api/microsoft.azure.batch.jobscheduleoperations) |[Enumeración de las programaciones de trabajos de una cuenta](/rest/api/batchservice/jobschedule/list) |
| [JobScheduleOperations.ListJobs](/dotnet/api/microsoft.azure.batch.jobscheduleoperations) |[Enumeración de los trabajos asociados a una programación de trabajo](/rest/api/batchservice/job/listfromjobschedule) |
| [PoolOperations.ListComputeNodes](/dotnet/api/microsoft.azure.batch.pooloperations) |[Enumeración de los nodos de proceso de un grupo](/rest/api/batchservice/computenode/list) |
| [PoolOperations.ListPools](/dotnet/api/microsoft.azure.batch.pooloperations) |[Enumeración de los grupos de una cuenta](/rest/api/batchservice/pool/list) |

### <a name="mappings-for-select-strings"></a>Asignaciones de las cadenas select

- **Tipos de .NET de Batch**: Tipos de API de .NET de Batch.
- **Entidades de la API REST**: todas las páginas de esta columna contienen una o varias tablas que enumeran los nombres de propiedades de la API REST para el tipo. Estos nombres de propiedades se usan al construir cadenas *select* . Estos mismos nombres se usan al construir una cadena [ODATADetailLevel.SelectClause](/dotnet/api/microsoft.azure.batch.odatadetaillevel.selectclause).

| Tipos de .NET de Batch | Entidades de la API de REST |
| --- | --- |
| [Certificate](/dotnet/api/microsoft.azure.batch.certificate) |[Obtención de información sobre un certificado](/rest/api/batchservice/certificate/get) |
| [CloudJob](/dotnet/api/microsoft.azure.batch.cloudjob) |[Obtención de información sobre un trabajo](/rest/api/batchservice/job/get) |
| [CloudJobSchedule](/dotnet/api/microsoft.azure.batch.cloudjobschedule) |[Obtención de información sobre una programación de trabajo](/rest/api/batchservice/jobschedule/get) |
| [ComputeNode](/dotnet/api/microsoft.azure.batch.computenode) |[Obtención de información sobre un nodo](/rest/api/batchservice/computenode/get) |
| [CloudPool](/dotnet/api/microsoft.azure.batch.cloudpool) |[Obtención de información sobre un grupo](/rest/api/batchservice/pool/get) |
| [CloudTask](/dotnet/api/microsoft.azure.batch.cloudtask) |[Obtención de información sobre una tarea](/rest/api/batchservice/task/get) |

## <a name="example-construct-a-filter-string"></a>Ejemplo: construcción de una cadena filter

Al construir una cadena filter para [ODATADetailLevel.FilterClause](/dotnet/api/microsoft.azure.batch.odatadetaillevel.filterclause), consulte la tabla en la sección "Asignaciones de las cadenas filter" para buscar la página de documentación de la API de REST correspondiente a la operación de lista que desea realizar. Encontrará las propiedades filtrables y sus operadores admitidos en la primera tabla de varias filas de dicha página. Por ejemplo, si desea recuperar todas las tareas cuyo código de salida era distinto de cero, en [Lista de las tareas asociadas a un trabajo](/rest/api/batchservice/task/list) esta fila especifica la cadena de propiedad aplicable y los operadores permitidos:

| Propiedad | Operaciones permitidas | Tipo |
|:--- |:--- |:--- |
| `executionInfo/exitCode` |`eq, ge, gt, le , lt` |`Int` |

Por lo tanto, la cadena filter para enumerar todas las tareas con un código de salida distinto de cero sería:

`(executionInfo/exitCode lt 0) or (executionInfo/exitCode gt 0)`

## <a name="example-construct-a-select-string"></a>Ejemplo: construcción de una cadena select

Para construir [ODATADetailLevel.SelectClause](/dotnet/api/microsoft.azure.batch.odatadetaillevel.selectclause), consulte la tabla de "Asignaciones de las cadenas select" y navegue a la página de la API de REST correspondiente al tipo de entidad que vaya a enumerar. Encontrará las propiedades seleccionables y sus operadores admitidos en la primera tabla de varias filas de dicha página. Por ejemplo, si desea recuperar solo el identificador y la línea de comandos para cada tarea de una lista, encontrará estas filas en la tabla correspondiente en la página sobre cómo [obtener información acerca de una tarea](/rest/api/batchservice/task/get):

| Propiedad | Tipo | Notas |
|:--- |:--- |:--- |
| `id` |`String` |`The ID of the task.` |
| `commandLine` |`String` |`The command line of the task.` |

La cadena select para incluir solo el identificador y la línea de comandos con cada tarea de la lista sería:

`id, commandLine`

## <a name="code-samples"></a>Ejemplos de código

### <a name="efficient-list-queries-code-sample"></a>Código de ejemplo de consultas de lista eficaces

Consulte el proyecto de ejemplo [EfficientListQueries](https://github.com/Azure-Samples/azure-batch-samples/tree/master/CSharp/ArticleProjects/EfficientListQueries) en GitHub para ver cómo una consulta de lista eficaz puede afectar al rendimiento de una aplicación. Esta aplicación de consola de C# crea y agrega un gran número de tareas a un trabajo. Luego realiza varias llamadas al método [JobOperations.ListTasks](/dotnet/api/microsoft.azure.batch.joboperations) y pasa objetos [ODATADetailLevel](/dotnet/api/microsoft.azure.batch.odatadetaillevel) que se configuran con distintos valores de propiedad para variar la cantidad de datos que se van a devolver. Genera una salida similar a la siguiente:

```
Adding 5000 tasks to job jobEffQuery...
5000 tasks added in 00:00:47.3467587, hit ENTER to query tasks...

4943 tasks retrieved in 00:00:04.3408081 (ExpandClause:  | FilterClause: state eq 'active' | SelectClause: id,state)
0 tasks retrieved in 00:00:00.2662920 (ExpandClause:  | FilterClause: state eq 'running' | SelectClause: id,state)
59 tasks retrieved in 00:00:00.3337760 (ExpandClause:  | FilterClause: state eq 'completed' | SelectClause: id,state)
5000 tasks retrieved in 00:00:04.1429881 (ExpandClause:  | FilterClause:  | SelectClause: id,state)
5000 tasks retrieved in 00:00:15.1016127 (ExpandClause:  | FilterClause:  | SelectClause: id,state,environmentSettings)
5000 tasks retrieved in 00:00:17.0548145 (ExpandClause: stats | FilterClause:  | SelectClause: )

Sample complete, hit ENTER to continue...
```

Como se muestra en los tiempos transcurridos, se pueden reducir considerablemente los tiempos de respuesta, para lo que es preciso limitar las propiedades y el número de elementos que se devuelven. Puede encontrar este y otros proyectos de ejemplo en el repositorio [azure-batch-samples](https://github.com/Azure-Samples/azure-batch-samples) de GitHub.

### <a name="batchmetrics-library-and-code-sample"></a>Biblioteca y código de ejemplo de BatchMetrics

Además del ejemplo de código EfficientListQueries anterior, el proyecto de ejemplo [BatchMetrics](https://github.com/Azure-Samples/azure-batch-samples/tree/master/CSharp/BatchMetrics) demuestra cómo supervisar de forma eficiente el progreso del trabajo de Azure Batch mediante la API de Batch.

El ejemplo [BatchMetrics](https://github.com/Azure-Samples/azure-batch-samples/tree/master/CSharp/BatchMetrics) incluye un proyecto de biblioteca de clases de .NET que puede incorporar en sus propios proyectos y un programa de línea de comandos simple para probar y demostrar el uso de la biblioteca.

La aplicación de ejemplo del proyecto muestra las siguientes operaciones:

1. Selección de atributos específicos para descargar solo las propiedades que necesita
2. Filtrado en tiempo de transición de estado para descargar solo los cambios desde la última consulta

Por ejemplo, el siguiente método aparece en la biblioteca BatchMetrics. Devuelve un objeto ODATADetailLevel que establece que solo deben obtenerse las propiedades `id` y `state` en las consultas realizadas. También especifica que solo las entidades cuyo estado ha cambiado desde que se ha especificado el parámetro `DateTime` se deben devolver.

```csharp
internal static ODATADetailLevel OnlyChangedAfter(DateTime time)
{
    return new ODATADetailLevel(
        selectClause: "id, state",
        filterClause: string.Format("stateTransitionTime gt DateTime'{0:o}'", time)
    );
}
```

## <a name="next-steps"></a>Pasos siguientes

- Obtenga información sobre cómo [Maximizar el uso de recursos de proceso de Azure Batch con tareas simultáneas de nodo](batch-parallel-node-tasks.md). Algunos tipos de cargas de trabajo pueden beneficiarse de la ejecución de tareas paralelas en una menor cantidad de nodos de ejecución que, sin embargo, sean de mayor tamaño. Consulte la sección [Escenario de ejemplo](batch-parallel-node-tasks.md#example-scenario) en el artículo para obtener detalles sobre dicho escenario.
- Obtenga más información sobre la [Supervisión de las soluciones de Batch realizando un recuento de las tareas y los nodos por estado](batch-get-resource-counts.md).


[api_net]: /dotnet/api/microsoft.azure.batch
[api_net_listjobs]: /dotnet/api/microsoft.azure.batch.joboperations
[api_rest]: /rest/api/batchservice/
[batch_metrics]: https://github.com/Azure/azure-batch-samples/tree/master/CSharp/BatchMetrics
[efficient_query_sample]: https://github.com/Azure/azure-batch-samples/tree/master/CSharp/ArticleProjects/EfficientListQueries
[github_samples]: https://github.com/Azure/azure-batch-samples
[odata]: /dotnet/api/microsoft.azure.batch.odatadetaillevel
[odata_ctor]: /dotnet/api/microsoft.azure.batch.odatadetaillevel
[odata_expand]: /dotnet/api/microsoft.azure.batch.odatadetaillevel
[odata_filter]: /dotnet/api/microsoft.azure.batch.odatadetaillevel
[odata_select]: /dotnet/api/microsoft.azure.batch.odatadetaillevel

[net_list_certs]: /dotnet/api/microsoft.azure.batch.certificateoperations
[net_list_compute_nodes]: /dotnet/api/microsoft.azure.batch.pooloperations
[net_list_job_schedules]: /dotnet/api/microsoft.azure.batch.jobscheduleoperations
[net_list_jobprep_status]: /dotnet/api/microsoft.azure.batch.joboperations
[net_list_jobs]: /dotnet/api/microsoft.azure.batch.joboperations
[net_list_nodefiles]: /dotnet/api/microsoft.azure.batch.joboperations
[net_list_pools]: /dotnet/api/microsoft.azure.batch.pooloperations
[net_list_schedule_jobs]: /dotnet/api/microsoft.azure.batch.jobscheduleoperations
[net_list_task_files]: /dotnet/api/microsoft.azure.batch.cloudtask
[net_list_tasks]: /dotnet/api/microsoft.azure.batch.joboperations

[rest_list_certs]: /rest/api/batchservice/certificate/list
[rest_list_compute_nodes]: /rest/api/batchservice/computenode/list
[rest_list_job_schedules]: /rest/api/batchservice/jobschedule/list
[rest_list_jobprep_status]: /rest/api/batchservice/job/listpreparationandreleasetaskstatus
[rest_list_jobs]: /rest/api/batchservice/job/list
[rest_list_nodefiles]: /rest/api/batchservice/file/listfromcomputenode
[rest_list_pools]: /rest/api/batchservice/pool/list
[rest_list_schedule_jobs]: /rest/api/batchservice/job/listfromjobschedule
[rest_list_task_files]: /rest/api/batchservice/file/listfromtask
[rest_list_tasks]: /rest/api/batchservice/task/list

[rest_get_cert]: /rest/api/batchservice/certificate/get
[rest_get_job]: /rest/api/batchservice/job/get
[rest_get_node]: /rest/api/batchservice/computenode/get
[rest_get_pool]: /rest/api/batchservice/pool/get
[rest_get_schedule]: /rest/api/batchservice/jobschedule/get
[rest_get_task]: /rest/api/batchservice/task/get

[net_cert]: /dotnet/api/microsoft.azure.batch.certificate
[net_job]: /dotnet/api/microsoft.azure.batch.cloudjob
[net_node]: /dotnet/api/microsoft.azure.batch.computenode
[net_pool]: /dotnet/api/microsoft.azure.batch.cloudpool
[net_schedule]: /dotnet/api/microsoft.azure.batch.cloudjobschedule
[net_task]: /dotnet/api/microsoft.azure.batch.cloudtask

[rest_get_task_counts]: /rest/api/batchservice/get-the-task-counts-for-a-job
[rest_get_node_counts]: /rest/api/batchservice/account/listpoolnodecounts
