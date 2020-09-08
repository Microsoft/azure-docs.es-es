---
title: Simulación de errores en aplicaciones de Azure Service Fabric
description: Protección de los servicios de Azure Service Fabric contra errores correctos/incorrectos.
author: anmolah
ms.topic: conceptual
ms.date: 06/15/2017
ms.author: anmola
ms.custom: devx-track-csharp
ms.openlocfilehash: c193ef00d56b2d2fc7ad806cc795dcbe360db548
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89022112"
---
# <a name="simulate-failures-during-service-workloads"></a>Simulación de errores durante las cargas de trabajo del servicio
Los escenarios de capacidad de prueba en Service Fabric de Azure permiten a los desarrolladores dejar de preocuparse por tratar los errores individuales. Sin embargo, hay escenarios donde se necesita una intercalación explícita de la carga de trabajo de cliente y de los errores. La intercalación de la carga de trabajo de cliente y de los errores garantiza que el servicio realmente realiza alguna acción cuando se produce el error. Dado el nivel de control que ofrece la capacidad de prueba, podrían encontrarse en puntos precisos de la ejecución de la carga de trabajo. Esta inducción de errores en los distintos estados de la aplicación puede buscar errores y mejorar la calidad.

## <a name="sample-custom-scenario"></a>Escenario de ejemplo personalizado
Esta prueba muestra un escenario que intercala la carga de trabajo de negocios con [errores correctos e incorrectos](service-fabric-testability-actions.md#graceful-vs-ungraceful-fault-actions). Los errores deben inducirse en el centro de operaciones de servicio o de proceso para obtener mejores resultados.

Veamos un ejemplo de un servicio que expone cuatro cargas de trabajo: A, B, C y D. Cada una corresponde a un conjunto de flujos de trabajo y podría ser de proceso, almacenamiento o una combinación. Por simplicidad, se aislarán las cargas de trabajo en nuestro ejemplo. Los diferentes errores ejecutados en este ejemplo son:

* RestartNode: error al simular un reinicio de la máquina.
* RestartDeployedCodePackage: error al simular el bloqueo de los procesos del host de servicio.
* RemoveReplica: error al simular la eliminación de réplicas.
* MovePrimary: error al simular los movimientos de réplica desencadenados por el equilibrador de carga de Service Fabric.

```csharp
// Add a reference to System.Fabric.Testability.dll and System.Fabric.dll.

using System;
using System.Fabric;
using System.Fabric.Testability.Scenario;
using System.Threading;
using System.Threading.Tasks;

class Test
{
    public static int Main(string[] args)
    {
        // Replace these strings with the actual version for your cluster and application.
        string clusterConnection = "localhost:19000";
        Uri applicationName = new Uri("fabric:/samples/PersistentToDoListApp");
        Uri serviceName = new Uri("fabric:/samples/PersistentToDoListApp/PersistentToDoListService");

        Console.WriteLine("Starting Workload Test...");
        try
        {
            RunTestAsync(clusterConnection, applicationName, serviceName).Wait();
        }
        catch (AggregateException ae)
        {
            Console.WriteLine("Workload Test failed: ");
            foreach (Exception ex in ae.InnerExceptions)
            {
                if (ex is FabricException)
                {
                    Console.WriteLine("HResult: {0} Message: {1}", ex.HResult, ex.Message);
                }
            }
            return -1;
        }

        Console.WriteLine("Workload Test completed successfully.");
        return 0;
    }

    public enum ServiceWorkloads
    {
        A,
        B,
        C,
        D
    }

    public enum ServiceFabricFaults
    {
        RestartNode,
        RestartCodePackage,
        RemoveReplica,
        MovePrimary,
    }

    public static async Task RunTestAsync(string clusterConnection, Uri applicationName, Uri serviceName)
    {
        // Create FabricClient with connection and security information here.
        FabricClient fabricClient = new FabricClient(clusterConnection);
        // Maximum time to wait for a service to stabilize.
        TimeSpan maxServiceStabilizationTime = TimeSpan.FromSeconds(120);

        // How many loops of faults you want to execute.
        uint testLoopCount = 20;
        Random random = new Random();

        for (var i = 0; i < testLoopCount; ++i)
        {
            var workload = SelectRandomValue<ServiceWorkloads>(random);
            // Start the workload.
            var workloadTask = RunWorkloadAsync(workload);

            // While the task is running, induce faults into the service. They can be ungraceful faults like
            // RestartNode and RestartDeployedCodePackage or graceful faults like RemoveReplica or MovePrimary.
            var fault = SelectRandomValue<ServiceFabricFaults>(random);

            // Create a replica selector, which will select a primary replica from the given service to test.
            var replicaSelector = ReplicaSelector.PrimaryOf(PartitionSelector.RandomOf(serviceName));
            // Run the selected random fault.
            await RunFaultAsync(applicationName, fault, replicaSelector, fabricClient);
            // Validate the health and stability of the service.
            await fabricClient.TestManager.ValidateServiceAsync(serviceName, maxServiceStabilizationTime);

            // Wait for the workload to finish successfully.
            await workloadTask;
        }
    }

    private static async Task RunFaultAsync(Uri applicationName, ServiceFabricFaults fault, ReplicaSelector selector, FabricClient client)
    {
        switch (fault)
        {
            case ServiceFabricFaults.RestartNode:
                await client.FaultManager.RestartNodeAsync(selector, CompletionMode.Verify);
                break;
            case ServiceFabricFaults.RestartCodePackage:
                await client.FaultManager.RestartDeployedCodePackageAsync(applicationName, selector, CompletionMode.Verify);
                break;
            case ServiceFabricFaults.RemoveReplica:
                await client.FaultManager.RemoveReplicaAsync(selector, CompletionMode.Verify, false);
                break;
            case ServiceFabricFaults.MovePrimary:
                await client.FaultManager.MovePrimaryAsync(selector.PartitionSelector);
                break;
        }
    }

    private static Task RunWorkloadAsync(ServiceWorkloads workload)
    {
        throw new NotImplementedException();
        // This is where you trigger and complete your service workload.
        // Note that the faults induced while your service workload is running will
        // fault the primary service. Hence, you will need to reconnect to complete or check
        // the status of the workload.
    }

    private static T SelectRandomValue<T>(Random random)
    {
        Array values = Enum.GetValues(typeof(T));
        T workload = (T)values.GetValue(random.Next(values.Length));
        return workload;
    }
}
```
