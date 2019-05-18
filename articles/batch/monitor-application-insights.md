---
title: Supervisión de Batch con Azure Application Insights | Microsoft Docs
description: Aprenda a instrumentar una aplicación .NET de Azure Batch con la biblioteca de Azure Application Insights.
services: batch
author: laurenhughes
manager: jeconnoc
ms.assetid: ''
ms.service: batch
ms.devlang: .NET
ms.topic: article
ms.workload: na
ms.date: 04/05/2018
ms.author: lahugh
ms.openlocfilehash: c527b0b10a2b9a351b242d0858fdbe64687970a7
ms.sourcegitcommit: 1fbc75b822d7fe8d766329f443506b830e101a5e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/14/2019
ms.locfileid: "65595290"
---
# <a name="monitor-and-debug-an-azure-batch-net-application-with-application-insights"></a>Supervisión y depuración de una aplicación .NET de Azure Batch con Application Insights

[Application Insights](../azure-monitor/app/app-insights-overview.md) proporciona una forma elegante y eficaz para que los desarrolladores supervisen y depuren las aplicaciones implementadas en los servicios de Azure. Utilice Application Insights para supervisar los contadores de rendimiento y las excepciones, además de para instrumentar el código con métricas y seguimiento personalizados. La integración de Application Insights en la aplicación de Azure Batch permite obtener información detallada sobre los distintos comportamientos e investigar los problemas casi en tiempo real.

Este artículo muestra cómo agregar la biblioteca de Application Insights a la solución .NET de Azure Batch, configurarla e instrumentar el código de aplicación. También muestra los métodos para supervisar la aplicación con Azure Portal y crear paneles personalizados. Para la compatibilidad de Application Insights con otros lenguajes, consulte la [documentación de lenguajes, plataformas e integraciones](../azure-monitor/app/platforms.md).

Hay una solución de C# de ejemplo con código que acompaña este artículo disponible en [GitHub](https://github.com/Azure/azure-batch-samples/tree/master/CSharp/ArticleProjects/ApplicationInsights). Este ejemplo agrega el código de instrumentación de Application Insights al ejemplo [TopNWords](https://github.com/Azure/azure-batch-samples/tree/master/CSharp/TopNWords). Si no está familiarizado con ese ejemplo, pruebe primero a compilar y ejecutar TopNWords. Esto le ayudará a comprender un flujo de trabajo básico de procesamiento de un conjunto de blobs de entrada en paralelo en varios nodos de ejecución con Batch. 

> [!TIP]
> Como alternativa, configure la solución de Batch para mostrar datos de Application Insights, como los contadores de rendimiento de máquina virtual en Batch Explorer. [Batch Explorer](https://github.com/Azure/BatchExplorer) es una herramienta de cliente independiente, completa y gratuita que puede ayudarle a crear, depurar y supervisar aplicaciones de Azure Batch. Descargue un [paquete de instalación](https://azure.github.io/BatchExplorer/) para Mac, Linux o Windows. Consulte el [repositorio batch-insights](https://github.com/Azure/batch-insights) para ver los pasos rápidos para habilitar los datos de Application Insights en Batch Explorer. 
>

## <a name="prerequisites"></a>Requisitos previos
* [Visual Studio 2017 o posterior](https://www.visualstudio.com/vs)

* [Una cuenta de Batch y una cuenta de Storage vinculada](batch-account-create-portal.md)

* [Recurso de Application Insights](../azure-monitor/app/create-new-resource.md )
  
   * Cree un nuevo *recurso* de Application Insights mediante Azure Portal. Seleccione el **Tipo de aplicación** *General*.

   * Copie la [clave de instrumentación](../azure-monitor/app/create-new-resource.md #copy-the-instrumentation-key) del portal. Se necesita más adelante en este artículo.
  
  > [!NOTE]
  > Es posible que se le [cobre](https://azure.microsoft.com/pricing/details/application-insights/) por los datos almacenados en Application Insights. Esto incluye el diagnóstico y la supervisión de los datos descritos en este artículo.
  > 

## <a name="add-application-insights-to-your-project"></a>Agregar Application Insights a un proyecto

El paquete NuGet **Microsoft.ApplicationInsights.WindowsServer** y sus dependencias son necesarios para el proyecto. Agréguelos al proyecto de la aplicación o restáurelos en él. Para instalar el paquete, use el comando `Install-Package` o el Administrador de paquetes NuGet.

```powershell
Install-Package Microsoft.ApplicationInsights.WindowsServer
```
Cree una referencia a Application Insights en la aplicación .NET mediante el espacio de nombres **Microsoft.ApplicationInsights**.

## <a name="instrument-your-code"></a>Instrumentación del código

Para instrumentar el código, la solución debe crear [TelemetryClient](/dotnet/api/microsoft.applicationinsights.telemetryclient) en Application Insights. En el ejemplo, TelemetryClient carga su configuración del archivo [ApplicationInsights.config](../azure-monitor/app/configuration-with-applicationinsights-config.md). No olvide actualizar ApplicationInsights.config en los siguientes proyectos con la clave de instrumentación de Application Insights: Microsoft.Azure.Batch.Samples.TelemetryStartTask y TopNWordsSample.

```xml
<InstrumentationKey>YOUR-IKEY-GOES-HERE</InstrumentationKey>
```
Agregue también la clave de instrumentación al archivo TopNWords.cs.

En el ejemplo de TopNWords.cs se usan las siguientes [llamadas de instrumentación](../azure-monitor/app/api-custom-events-metrics.md) desde la API de Application Insights:
* `TrackMetric()`: realiza un seguimiento de la media de tiempo que un nodo de proceso tarda en descargar el archivo de texto necesario.
* `TrackTrace()`: agrega llamadas de depuración al código.
* `TrackEvent()`: realiza un seguimiento de los eventos interesantes de capturar.

En este ejemplo se omite el control de excepciones de manera deliberada. En su lugar, Application Insights notifica automáticamente las excepciones no controladas, lo que mejora considerablemente la experiencia de depuración. 

El siguiente fragmento de código muestra cómo usar estos métodos.

```csharp
public void CountWords(string blobName, int numTopN, string storageAccountName, string storageAccountKey)
{
    // simulate exception for some set of tasks
    Random rand = new Random();
    if (rand.Next(0, 10) % 10 == 0)
    {
        blobName += ".badUrl";
    }

    // log the url we are downloading the file from
    insightsClient.TrackTrace(new TraceTelemetry(string.Format("Task {0}: Download file from: {1}", this.taskId, blobName), SeverityLevel.Verbose));

    // open the cloud blob that contains the book
    var storageCred = new StorageCredentials(storageAccountName, storageAccountKey);
    CloudBlockBlob blob = new CloudBlockBlob(new Uri(blobName), storageCred);
    using (Stream memoryStream = new MemoryStream())
    {
        // calculate blob download time
        DateTime start = DateTime.Now;
        blob.DownloadToStream(memoryStream);
        TimeSpan downloadTime = DateTime.Now.Subtract(start);

        // track how long the blob takes to download on this node
        // this will help debug timing issues or identify poorly performing nodes
        insightsClient.TrackMetric("Blob download in seconds", downloadTime.TotalSeconds, this.CommonProperties);

        memoryStream.Position = 0; //Reset the stream
        var sr = new StreamReader(memoryStream);
        var myStr = sr.ReadToEnd();
        string[] words = myStr.Split(' ');

        // log how many words were found in the text file
        insightsClient.TrackTrace(new TraceTelemetry(string.Format("Task {0}: Found {1} words", this.taskId, words.Length), SeverityLevel.Verbose));
        var topNWords =
            words.
                Where(word => word.Length > 0).
                GroupBy(word => word, (key, group) => new KeyValuePair<String, long>(key, group.LongCount())).
                OrderByDescending(x => x.Value).
                Take(numTopN).
                ToList();
        foreach (var pair in topNWords)
        {
            Console.WriteLine("{0} {1}", pair.Key, pair.Value);
        }

        // emit an event to track the completion of the task
        insightsClient.TrackEvent("Done counting words");
    }
}
```

### <a name="azure-batch-telemetry-initializer-helper"></a>Asistente del inicializador de telemetría de Azure Batch
Al informar de telemetría de un servidor e instancia determinados, Application Insights utiliza el rol y el nombre de la máquina virtual de Azure como valores predeterminados. En el contexto de Azure Batch, en el ejemplo se muestra cómo usar el nombre del grupo y el nombre del nodo de proceso en su lugar. Use un [inicializador de telemetría](../azure-monitor/app/api-filtering-sampling.md#add-properties) para invalidar los valores predeterminados. 

```csharp
using Microsoft.ApplicationInsights.Channel;
using Microsoft.ApplicationInsights.Extensibility;
using System;
using System.Threading;

namespace Microsoft.Azure.Batch.Samples.TelemetryInitializer
{
    public class AzureBatchNodeTelemetryInitializer : ITelemetryInitializer
    {
        // Azure Batch environment variables
        private const string PoolIdEnvironmentVariable = "AZ_BATCH_POOL_ID";
        private const string NodeIdEnvironmentVariable = "AZ_BATCH_NODE_ID";

        private string roleInstanceName;
        private string roleName;

        public void Initialize(ITelemetry telemetry)
        {
            if (string.IsNullOrEmpty(telemetry.Context.Cloud.RoleName))
            {
                // override the role name with the Azure Batch Pool name
                string name = LazyInitializer.EnsureInitialized(ref this.roleName, this.GetPoolName);
                telemetry.Context.Cloud.RoleName = name;
            }

            if (string.IsNullOrEmpty(telemetry.Context.Cloud.RoleInstance))
            {
                // override the role instance with the Azure Batch Compute Node name
                string name = LazyInitializer.EnsureInitialized(ref this.roleInstanceName, this.GetNodeName);
                telemetry.Context.Cloud.RoleInstance = name;
            }
        }

        private string GetPoolName()
        {
            return Environment.GetEnvironmentVariable(PoolIdEnvironmentVariable) ?? string.Empty;
        }

        private string GetNodeName()
        {
            return Environment.GetEnvironmentVariable(NodeIdEnvironmentVariable) ?? string.Empty;
        }
    }
}
```

Para habilitar el inicializador de telemetría, el archivo ApplicationInsights.config del proyecto TopNWordsSample incluye lo siguiente:

```xml
<TelemetryInitializers>
    <Add Type="Microsoft.Azure.Batch.Samples.TelemetryInitializer.AzureBatchNodeTelemetryInitializer, Microsoft.Azure.Batch.Samples.TelemetryInitializer"/>
</TelemetryInitializers>
``` 

## <a name="update-the-job-and-tasks-to-include-application-insights-binaries"></a>Actualización del trabajo y las tareas para incluir los archivos binarios de Application Insights

Para que Application Insights para se ejecute correctamente en los nodos de proceso, asegúrese de que los archivos binarios se ubican correctamente. Agregue los archivos binarios necesarios a la colección de archivos de recursos de la tarea para que se descarguen cuando se ejecute la tarea. Los siguientes fragmentos de código son similares al código de Job.cs.

En primer lugar, cree una lista estática de archivos de Application Insights para cargar.

```csharp
private static readonly List<string> AIFilesToUpload = new List<string>()
{
    // Application Insights config and assemblies
    "ApplicationInsights.config",
    "Microsoft.ApplicationInsights.dll",
    "Microsoft.AI.Agent.Intercept.dll",
    "Microsoft.AI.DependencyCollector.dll",
    "Microsoft.AI.PerfCounterCollector.dll",
    "Microsoft.AI.ServerTelemetryChannel.dll",
    "Microsoft.AI.WindowsServer.dll",
    
    // custom telemetry initializer assemblies
    "Microsoft.Azure.Batch.Samples.TelemetryInitializer.dll",
 };
...
```

A continuación, cree los archivos de almacenamiento provisional que se usa la tarea.
```csharp
...
// create file staging objects that represent the executable and its dependent assembly to run as the task.
// These files are copied to every node before the corresponding task is scheduled to run on that node.
FileToStage topNWordExe = new FileToStage(TopNWordsExeName, stagingStorageAccount);
FileToStage storageDll = new FileToStage(StorageClientDllName, stagingStorageAccount);

// Upload Application Insights assemblies
List<FileToStage> aiStagedFiles = new List<FileToStage>();
foreach (string aiFile in AIFilesToUpload)
{
    aiStagedFiles.Add(new FileToStage(aiFile, stagingStorageAccount));
}
...
```

El método `FileToStage` es una función auxiliar del código de ejemplo que permite cargar fácilmente un archivo del disco local a un blob de Azure Storage. Cada archivo se descargará después en un nodo de proceso y se le hará referencia con una tarea.

Por último, agregue las tareas al trabajo e incluya los archivos binarios de Application Insights necesarios.
```csharp
...
// initialize a collection to hold the tasks that will be submitted in their entirety
List<CloudTask> tasksToRun = new List<CloudTask>(topNWordsConfiguration.NumberOfTasks);
for (int i = 1; i <= topNWordsConfiguration.NumberOfTasks; i++)
{
    CloudTask task = new CloudTask("task_no_" + i, String.Format("{0} --Task {1} {2} {3} {4}",
        TopNWordsExeName,
        string.Format("https://{0}.blob.core.windows.net/{1}",
            accountSettings.StorageAccountName,
            documents[i]),
        topNWordsConfiguration.TopWordCount,
        accountSettings.StorageAccountName,
        accountSettings.StorageAccountKey));

    //This is the list of files to stage to a container -- for each job, one container is created and 
    //files all resolve to Azure Blobs by their name (so two tasks with the same named file will create just 1 blob in
    //the container).
    task.FilesToStage = new List<IFileStagingProvider>
                        {
                            // required application binaries
                            topNWordExe,
                            storageDll,
                        };
    foreach (FileToStage stagedFile in aiStagedFiles)
   {
        task.FilesToStage.Add(stagedFile);
   }    
    task.RunElevated = false;
    tasksToRun.Add(task);
}
```

## <a name="view-data-in-the-azure-portal"></a>Visualización de datos en Azure Portal

Ahora que ha configurado el trabajo y las tareas para usar Application Insights, ejecute el trabajo de ejemplo en el grupo. Vaya a Azure Portal y abra el recurso de Application Insights que ha aprovisionado. Después de aprovisionar el grupo, deberá empezar a ver el flujo de datos y su registro. El resto de este artículo afecta solo a algunas características de Application Insights, pero no dude en explorarlas todas.

### <a name="view-live-stream-data"></a>Visualización del flujo de datos en directo

Para ver los registros de seguimiento en el recurso de Application Insights, haga clic en **Live Stream**. En la captura de pantalla siguiente se muestra cómo ver los datos en directo procedentes de los nodos de proceso del grupo, por ejemplo, el uso de CPU por nodo de proceso.

![Flujo en directo de datos del nodo de proceso](./media/monitor-application-insights/applicationinsightslivestream.png)

### <a name="view-trace-logs"></a>Visualización de los registros de seguimiento

Para ver los registros de seguimiento en el recurso de Application Insights, haga clic en **Buscar**. Esta vista muestra una lista de datos de diagnóstico capturados por Application Insights, incluidas las excepciones, los eventos y los seguimientos. 

En la captura de pantalla siguiente se muestra cómo el simple seguimiento de una tarea se registra y consulta más adelante para fines de depuración.

![Imagen de registros de seguimiento](./media/monitor-application-insights/tracelogsfortask.png)

### <a name="view-unhandled-exceptions"></a>Visualización de excepciones no controladas

En las capturas de pantalla siguientes se muestra cómo Application Insights registra las excepciones que produce la aplicación. En este caso, unos segundos después de que la aplicación que inicie la excepción concreta, puede profundizar en ella y diagnosticar el problema.

![Excepciones no controladas](./media/monitor-application-insights/exception.png)

### <a name="measure-blob-download-time"></a>Medición del tiempo de descarga de blobs

Las métricas personalizadas también son una valiosa herramienta del portal. Por ejemplo, puede mostrar el tiempo medio que tardó cada nodo de proceso en descargar el archivo de texto necesario que estaba procesando.

Creación de un gráfico de ejemplo:
1. En el recurso de Application Insights, haga clic en **Explorador de métricas** > **Agregar un gráfico**.
2. Haga clic en **Editar** en el gráfico que se ha agregado.
2. Actualice los datos del gráfico de la manera siguiente:
   * Establezca el **Tipo de gráfico** en **Cuadrícula**.
   * Establezca **Agregación** en **Media**.
   * Establezca **Agrupar por** en **NodeId**.
   * En **Métricas**, seleccione **Personalizado** > **Blob download in seconds** (Descarga de blobs en segundos).
   * Ajuste la visualización de la **paleta de colores** como prefiera. 

![Tiempo de descarga de blobs por nodo](./media/monitor-application-insights/blobdownloadtime.png)


## <a name="monitor-compute-nodes-continuously"></a>Supervisión continua de los nodos de proceso

Quizá haya observado que todas las métricas, incluidos los contadores de rendimiento, solo se registran con las tareas en ejecución. Este comportamiento es útil porque limita la cantidad de datos que registra Application Insights. Sin embargo, hay casos en los que querrá supervisar nodos de proceso de manera permanente. Por ejemplo, cuando ejecuten trabajo en segundo plano que no esté programado a través del servicio Batch. En este caso, configure un proceso de supervisión para ejecutarlo durante toda la vida del nodo de proceso. 

Una manera de conseguir este comportamiento es generar un proceso que cargue la biblioteca de Application Insights y se ejecute en segundo plano. En el ejemplo, la tarea de inicio carga los archivos binarios en la máquina y mantiene un proceso en ejecución de manera indefinida. Configure el archivo de configuración de Application Insights para que este proceso emita los datos adicionales que le interesen, como los contadores de rendimiento.

```csharp
...
 // Batch start task telemetry runner
private const string BatchStartTaskFolderName = "StartTask";
private const string BatchStartTaskTelemetryRunnerName = "Microsoft.Azure.Batch.Samples.TelemetryStartTask.exe";
private const string BatchStartTaskTelemetryRunnerAIConfig = "ApplicationInsights.config";
...
CloudPool pool = client.PoolOperations.CreatePool(
    topNWordsConfiguration.PoolId,
    targetDedicated: topNWordsConfiguration.PoolNodeCount,
    virtualMachineSize: "standard_d1_v2",
    cloudServiceConfiguration: new CloudServiceConfiguration(osFamily: "5"));
...

// Create a start task which will run a dummy exe in background that simply emits performance
// counter data as defined in the relevant ApplicationInsights.config.
// Note that the waitForSuccess on the start task was not set so the Compute Node will be
// available immediately after this command is run.
pool.StartTask = new StartTask()
{
    CommandLine = string.Format("cmd /c {0}", BatchStartTaskTelemetryRunnerName),
    ResourceFiles = resourceFiles
};
...
```

> [!TIP]
> Para aumentar la capacidad de administración de la solución, puede agrupar el ensamblado en un [paquete de aplicación](./batch-application-packages.md). A continuación, para implementar el paquete de aplicación automáticamente en los grupos, agregue una referencia de paquete de aplicación a la configuración del grupo.
>

## <a name="throttle-and-sample-data"></a>Limitaciones y datos de ejemplo 

Debido a la naturaleza a gran escala de las aplicaciones de Azure Batch que se ejecutan en producción, puede limitar la cantidad de datos que Application Insights recopila para administrar los costos. Consulte [Muestreo en Application Insights](../azure-monitor/app/sampling.md) para conocer algunos mecanismos para lograrlo.


## <a name="next-steps"></a>Pasos siguientes
* Más información sobre [Application Insights](../azure-monitor/app/app-insights-overview.md).

* Para la compatibilidad de Application Insights con otros lenguajes, consulte la [documentación de lenguajes, plataformas e integraciones](../azure-monitor/app/platforms.md).


