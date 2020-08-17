---
title: Simulación de errores en microservicios de Azure
description: En este artículo se habla sobre las acciones de capacidad de prueba que se encuentra en el servicio de Microsoft Azure Fabric.
author: motanv
ms.topic: conceptual
ms.date: 06/07/2017
ms.author: motanv
ms.openlocfilehash: 14b2b5bd2577a10ad77a715fb5d20e10da84cf1d
ms.sourcegitcommit: 3543d3b4f6c6f496d22ea5f97d8cd2700ac9a481
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/20/2020
ms.locfileid: "86518986"
---
# <a name="testability-actions"></a>Acciones de Testability
Para simular una infraestructura no confiable, Azure Service Fabric proporciona a los desarrolladores distintas formas de simular varios errores y transiciones de estados que se producen en escenarios reales. Dichas formas se exponen como acciones de Testability. Las acciones son las API de bajo nivel que provocan una inserción de errores específicos, una transición de estado o una validación. Mediante la combinación de estas acciones, puede escribir escenarios de prueba completos para los servicios.

Service Fabric proporciona varios escenarios de prueba comunes que constan de estas acciones. Recomendamos encarecidamente usar estos escenarios integrados, que se seleccionan meticulosamente, para probar las transiciones de estado comunes y los casos de error. Sin embargo, las acciones se pueden utilizar para crear escenarios de prueba personalizados cuando desee agregar cobertura a aquellos escenarios que no aún estén cubiertos por los escenarios integrados o adaptados de manera personalizada a su aplicación.

Las implementaciones en C# de las acciones se encuentran en el ensamblado System.Fabric.dll. El módulo System Fabric PowerShell se encuentra en el ensamblado Microsoft.ServiceFabric.Powershell.dll. Como parte de la instalación en tiempo de ejecución, se instala el módulo ServiceFabric PowerShell para facilitar su uso.

## <a name="graceful-vs-ungraceful-fault-actions"></a>Acciones de errores estables frente a inestables
Las acciones de Testability se clasifican en dos cubos principales:

* Errores inestables: simulan errores como reinicios de equipos y bloqueos de procesos. En tales casos, el contexto de ejecución del proceso se detiene abruptamente, lo que significa que no se puede ejecutar ninguna limpieza de estado antes de que se vuelva a iniciar la aplicación.
* Errores estables: simulan acciones estables como movimientos y eliminaciones de réplicas desencadenados por el equilibrio de carga. En tales casos, el servicio recibe una notificación de cierre y puede limpiar el estado antes de salir.

Para mejorar la calidad de la validación, ejecute la carga de trabajo del servicio y del negocio mientras provoca varios errores estables e inestables. Los errores inestables crean escenarios en que el proceso se cierra abruptamente en medio de algún flujo de trabajo. Así se prueba la ruta de recuperación una vez que Service Fabric restaura la réplica del servicio. Esto facilitará la comprobación de la coherencia de los datos y de si el estado del servicio se mantiene correctamente después de los errores. El otro conjunto de errores (los errores estables) prueban que el servicio reacciona correctamente al hecho de que Service Fabric mueva las réplicas. Así se prueba el control de la cancelación en el método RunAsync. El servicio debe comprobar si se ha establecido el token de cancelación, guardar correctamente su estado y salir del método RunAsync.

## <a name="testability-actions-list"></a>Lista de acciones de Testability
| Acción | Descripción | API administrada | Cmdlet de PowerShell | Errores estables o no estables |
| --- | --- | --- | --- | --- |
| CleanTestState |Quita todo el estado de prueba del clúster en caso de un cierre incorrecto del controlador de prueba. |CleanTestStateAsync |Remove-ServiceFabricTestState |No aplicable |
| InvokeDataLoss |Provoca la pérdida de datos en una partición del servicio. |InvokeDataLossAsync |Invoke-ServiceFabricPartitionDataLoss |Estable |
| InvokeQuorumLoss |Coloca una partición determinada del servicio con estado en pérdida de quórum. |InvokeQuorumLossAsync |Invoke-ServiceFabricQuorumLoss |Estable |
| MovePrimary |Mueve la réplica principal especificada del servicio con estado al nodo de clúster especificado. |MovePrimaryAsync |Move-ServiceFabricPrimaryReplica |Estable |
| MoveSecondary |Mueve la réplica secundaria actual de un servicio con estado a otro nodo de clúster. |MoveSecondaryAsync |Move-ServiceFabricSecondaryReplica |Estable |
| RemoveReplica |Simula un error de réplica mediante la eliminación de una réplica de un clúster. Esto cerrará la réplica y realizará su transición al rol 'None', con lo que quitará todo su estado del clúster. |RemoveReplicaAsync |Remove-ServiceFabricReplica |Estable |
| RestartDeployedCodePackage |Simula un error de proceso del paquete de código mediante el reinicio de un paquete de código implementado en un nodo de un clúster. Esto anula el proceso del paquete de código que reiniciará todas las réplicas del servicio de usuario hospedadas en dicho proceso. |RestartDeployedCodePackageAsync |Restart-ServiceFabricDeployedCodePackage |Inestable |
| RestartNode |Simula un error de nodo de clúster de Service Fabric mediante el reinicio de un nodo. |RestartNodeAsync |Restart-ServiceFabricNode |Inestable |
| RestartPartition |Simula un escenario de falta de disponibilidad del centro de datos o una falta de disponibilidad del clúster mediante el reinicio de algunas o todas las réplicas de una partición. |RestartPartitionAsync |Restart-ServiceFabricPartition |Estable |
| RestartReplica |Simula un error de réplica mediante el reinicio de una réplica persistente en un clúster, para lo que cierra la réplica y vuelva a abrirla. |RestartReplicaAsync |Restart-ServiceFabricReplica |Estable |
| StartNode |Inicia un nodo en un clúster que se ha detenido. |StartNodeAsync |Start-ServiceFabricNode |No aplicable |
| StopNode |Simula un error de nodo mediante la detención de un nodo en un clúster. El nodo permanecerá inactivo hasta que se llame a StartNode. |StopNodeAsync |Stop-ServiceFabricNode |Inestable |
| ValidateApplication |Valida la disponibilidad y mantenimiento de todos los servicios de Service Fabric de una aplicación, normalmente después de provocar algunos errores en el sistema. |ValidateApplicationAsync |Test-ServiceFabricApplication |No aplicable |
| ValidateService |Valida la disponibilidad y mantenimiento de todos un servicios de Service Fabric, normalmente después de provocar algunos errores en el sistema. |ValidateServiceAsync |Test-ServiceFabricService |No aplicable |

## <a name="running-a-testability-action-using-powershell"></a>Ejecución de una acción de Testability con PowerShell
Este tutorial muestra cómo ejecutar una acción de Testability con PowerShell. Aprenderá a ejecutar una acción de Testability en un clúster local (one-box) o un clúster de Azure. Microsoft.Fabric.Powershell.dll, el módulo Service Fabric PowerShell, se instala automáticamente al instalar el MSI de Microsoft Service Fabric. El módulo se carga automáticamente al abrir un aviso de PowerShell.

Secciones del tutorial:

* [Ejecución de una acción en un clúster one-box](#run-an-action-against-a-one-box-cluster)
* [Ejecución de una acción en un clúster de Azure](#run-an-action-against-an-azure-cluster)

### <a name="run-an-action-against-a-one-box-cluster"></a>Ejecución de una acción en un clúster one-box
Para ejecutar una acción de Testability en un clúster local, primero es preciso conectarse al clúster y, a continuación, abrir el aviso de PowerShell en modo de administrador. Examinemos la acción **Restart-ServiceFabricNode** .

```powershell
Restart-ServiceFabricNode -NodeName Node1 -CompletionMode DoNotVerify
```

Aquí, la acción **Restart-ServiceFabricNode** se ejecuta en un nodo denominado "Node1". El modo de finalización especifica que no debe comprobar si la acción restart-node se realizó correctamente. La especificación del modo de finalización como comprobación provocará que se compruebe si la acción de reinicio se realizó correctamente. En lugar de especificar directamente el nodo por su nombre, puede especificarlo a través de una clave de partición y el tipo de réplica, tal como se muestra a continuación:

```powershell
Restart-ServiceFabricNode -ReplicaKindPrimary  -PartitionKindNamed -PartitionKey Partition3 -CompletionMode Verify
```


```powershell
$connection = "localhost:19000"
$nodeName = "Node1"

Connect-ServiceFabricCluster $connection
Restart-ServiceFabricNode -NodeName $nodeName -CompletionMode DoNotVerify
```

**Restart-ServiceFabricNode** debe utilizarse para reiniciar un nodo de Service Fabric en un clúster. Esto detendrá el proceso Fabric.exe, que reiniciará todas las réplicas de los servicios de sistema y de los servicios de usuario hospedados en dicho nodo. Si usa esta API para probar el servicio, le ayudará a revelar los errores a lo largo de las rutas de recuperación de conmutación por error. Le ayuda a simular errores en los nodos del clúster.

La siguiente captura de pantalla muestra el comando **Restart-ServiceFabricNode** de Testability en acción.

![Captura de pantalla de la ejecución del comando Restart-ServiceFabricNode en PowerShell.](media/service-fabric-testability-actions/Restart-ServiceFabricNode.png)

El resultado del primer **Get ServiceFabricNode** (un cmdlet del módulo de PowerShell de Service Fabric) muestra que el clúster local tiene cinco nodos: de Node.1 a Node.5. Una vez que la acción de Testability (cmdlet) **Restart-ServiceFabricNode** se ejecute en el nodo, denominado Node.4, veremos que se ha restablecido el tiempo de actividad del nodo.

### <a name="run-an-action-against-an-azure-cluster"></a>Ejecución de una acción en un clúster de Azure
La ejecución de una acción de Testability (mediante el uso de PowerShell) en un clúster de Azure es similar a la ejecución de la acción en un clúster local. La única diferencia es que, para poder ejecutar la acción, en lugar de conectarse al clúster local, debe conectarse primero al clúster de Azure.

## <a name="running-a-testability-action-using-c35"></a>Ejecución de una acción de Testability con C&#35;
Para ejecutar una acción de Testability con C#, primero es preciso conectarse al clúster mediante FabricClient. A continuación, hay que obtener los parámetros necesarios para ejecutar la acción. Se pueden usar distintos parámetros para ejecutar la misma acción.
Examinando la acción RestartServiceFabricNode, una forma de ejecutarla es usar la información (nombre del nodo e Id. de instancia de nodo) del nodo en el clúster.

```csharp
RestartNodeAsync(nodeName, nodeInstanceId, completeMode, operationTimeout, CancellationToken.None)
```

Explicación de parámetros:

* **CompleteMode** especifica que el modo no debe comprobar si la acción de reinicio se realizó correctamente. La especificación del modo de finalización como comprobación provocará que se compruebe si la acción de reinicio se realizó correctamente.  
* **OperationTimeout** : establece la cantidad de tiempo que falta para que la operación finalice antes de que se inicie una excepción TimeoutException.
* **CancellationToken** : permite cancelar una llamada pendiente.

En lugar de especificar directamente el nodo por su nombre, se puede especificar a través de una clave de partición y el tipo de réplica.

Para más información, consulte PartitionSelector y ReplicaSelector.

```csharp
// Add a reference to System.Fabric.Testability.dll and System.Fabric.dll
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using System.Fabric.Testability;
using System.Fabric;
using System.Threading;
using System.Numerics;

class Test
{
    public static int Main(string[] args)
    {
        string clusterConnection = "localhost:19000";
        Uri serviceName = new Uri("fabric:/samples/PersistentToDoListApp/PersistentToDoListService");
        string nodeName = "N0040";
        BigInteger nodeInstanceId = 130743013389060139;

        Console.WriteLine("Starting RestartNode test");
        try
        {
            //Restart the node by using ReplicaSelector
            RestartNodeAsync(clusterConnection, serviceName).Wait();

            //Another way to restart node is by using nodeName and nodeInstanceId
            RestartNodeAsync(clusterConnection, nodeName, nodeInstanceId).Wait();
        }
        catch (AggregateException exAgg)
        {
            Console.WriteLine("RestartNode did not complete: ");
            foreach (Exception ex in exAgg.InnerExceptions)
            {
                if (ex is FabricException)
                {
                    Console.WriteLine("HResult: {0} Message: {1}", ex.HResult, ex.Message);
                }
            }
            return -1;
        }

        Console.WriteLine("RestartNode completed.");
        return 0;
    }

    static async Task RestartNodeAsync(string clusterConnection, Uri serviceName)
    {
        PartitionSelector randomPartitionSelector = PartitionSelector.RandomOf(serviceName);
        ReplicaSelector primaryofReplicaSelector = ReplicaSelector.PrimaryOf(randomPartitionSelector);

        // Create FabricClient with connection and security information here
        FabricClient fabricclient = new FabricClient(clusterConnection);
        await fabricclient.FaultManager.RestartNodeAsync(primaryofReplicaSelector, CompletionMode.Verify);
    }

    static async Task RestartNodeAsync(string clusterConnection, string nodeName, BigInteger nodeInstanceId)
    {
        // Create FabricClient with connection and security information here
        FabricClient fabricclient = new FabricClient(clusterConnection);
        await fabricclient.FaultManager.RestartNodeAsync(nodeName, nodeInstanceId, CompletionMode.Verify);
    }
}
```

## <a name="partitionselector-and-replicaselector"></a>PartitionSelector y ReplicaSelector
### <a name="partitionselector"></a>PartitionSelector
PartitionSelector es un asistente que se expone en Testability y que se utiliza para seleccionar una partición concreta en la que se va a realizar cualquiera de las acciones de Testability. Se puede usar para seleccionar una partición concreta si se conoce de antemano el Id. de la partición. O bien, se puede proporcionar la clave de partición y la operación resolverá internamente el Id. de la partición. También existe la opción de seleccionar una partición aleatoria.

Para usar este asistente, cree el objeto PartitionSelector y seleccione la partición mediante uno de los métodos Select*. A continuación, pase el objeto PartitionSelector a la API que lo requiera. Si no se selecciona ninguna opción, el valor predeterminado es una partición aleatoria.

```csharp
Uri serviceName = new Uri("fabric:/samples/InMemoryToDoListApp/InMemoryToDoListService");
Guid partitionIdGuid = new Guid("8fb7ebcc-56ee-4862-9cc0-7c6421e68829");
string partitionName = "Partition1";
Int64 partitionKeyUniformInt64 = 1;

// Select a random partition
PartitionSelector randomPartitionSelector = PartitionSelector.RandomOf(serviceName);

// Select a partition based on ID
PartitionSelector partitionSelectorById = PartitionSelector.PartitionIdOf(serviceName, partitionIdGuid);

// Select a partition based on name
PartitionSelector namedPartitionSelector = PartitionSelector.PartitionKeyOf(serviceName, partitionName);

// Select a partition based on partition key
PartitionSelector uniformIntPartitionSelector = PartitionSelector.PartitionKeyOf(serviceName, partitionKeyUniformInt64);
```

### <a name="replicaselector"></a>ReplicaSelector
ReplicaSelector es un asistente que se expone en Testability y que se utiliza para ayudar a seleccionar una réplica en la que se va a realizar cualquiera de las acciones de Testability. Se puede usar para seleccionar una réplica concreta si se conoce de antemano el identificador de la réplica. Además, existe la opción de seleccionar una réplica principal o secundaria aleatoria. ReplicaSelector se deriva de PartitionSelector, por lo que es preciso seleccionar tanto la réplica como la partición en la que se desea realizar la operación de la Testability.

Para esta aplicación auxiliar, cree un objeto ReplicaSelector y establezca la forma en que desea seleccionar la réplica y la partición. A continuación, puede pasarlo a la API que lo requiera. Si no se selecciona ninguna opción, el valor predeterminado es una réplica aleatoria y una partición aleatoria.

```csharp
Guid partitionIdGuid = new Guid("8fb7ebcc-56ee-4862-9cc0-7c6421e68829");
PartitionSelector partitionSelector = PartitionSelector.PartitionIdOf(serviceName, partitionIdGuid);
long replicaId = 130559876481875498;

// Select a random replica
ReplicaSelector randomReplicaSelector = ReplicaSelector.RandomOf(partitionSelector);

// Select the primary replica
ReplicaSelector primaryReplicaSelector = ReplicaSelector.PrimaryOf(partitionSelector);

// Select the replica by ID
ReplicaSelector replicaByIdSelector = ReplicaSelector.ReplicaIdOf(partitionSelector, replicaId);

// Select a random secondary replica
ReplicaSelector secondaryReplicaSelector = ReplicaSelector.RandomSecondaryOf(partitionSelector);
```

## <a name="next-steps"></a>Pasos siguientes
* [Escenarios de Testability](service-fabric-testability-scenarios.md)
* Procedimientos para probar un servicio
  * [Simulación de errores durante las cargas de trabajo del servicio](service-fabric-testability-workload-tests.md)
  * [Errores de comunicación entre servicios](service-fabric-testability-scenarios-service-communication.md)

