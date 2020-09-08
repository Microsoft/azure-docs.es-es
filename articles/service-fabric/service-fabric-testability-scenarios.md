---
title: Creación de pruebas de conmutación por error y caos para Azure Service Fabric
description: Uso de los escenarios de prueba de conmutación por error y pruebas de caos de Service Fabric para inducir acciones de error y comprobar la confiabilidad de los servicios.
author: motanv
ms.topic: conceptual
ms.date: 10/1/2019
ms.author: motanv
ms.custom: devx-track-csharp
ms.openlocfilehash: 17f0af8c041042ddceccc1f8701e44ab8522840f
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89022129"
---
# <a name="testability-scenarios"></a>Escenarios de Testability
Los grandes sistemas distribuidos, como infraestructuras de nube, son inherentemente poco confiables. Azure Service Fabric ofrece a los desarrolladores la capacidad de escribir servicios para ejecutarse sobre infraestructuras poco confiables. Para poder escribir servicios de alta calidad, los desarrolladores deben poder inducir tal infraestructura confiable para probar la estabilidad de sus servicios.

El servicio de análisis de errores proporciona a los desarrolladores la capacidad de inducir acciones de error para probar los servicios en casos de mal funcionamiento. Sin embargo, hasta ahora se obtendrán solo errores simulados dirigidos. Para realizar más pruebas, puede usar los escenarios de prueba en Service Fabric: una prueba de caos y una prueba de conmutación por error. Estos escenarios simulan errores continuos intercalados, tanto correctos como incorrectos, en todo el clúster durante períodos prolongados de tiempo. Una vez configurada una prueba con la tasa y el tipo de errores, se puede iniciar mediante las API de C# o de PowerShell para generar errores en el clúster y en el servicio.

> [!WARNING]
> ChaosTestScenario va a reemplazarse por una versión de Caos más resistente y basado en servicios. Consulte el artículo nuevo sobre [inducción de errores controlados con Caos](service-fabric-controlled-chaos.md) para obtener más detalles.
> 
> 

## <a name="chaos-test"></a>Prueba de caos
El escenario de caos genera errores en todo el clúster de Service Fabric. El escenario comprime los errores que se ven por lo general durante meses o años en unas pocas horas. Esta combinación de errores intercalados con una elevada tasa de errores encuentra casos excepcionales que de otra manera pasan desapercibidos. Esto conduce a una mejora considerable en la calidad del código del servicio.

### <a name="faults-simulated-in-the-chaos-test"></a>Errores simulados en la prueba de caos
* Reinicio de un nodo
* Reinicio de un paquete de código implementado
* Eliminación de una réplica
* Reinicio de una réplica
* Desplazamiento de una réplica principal (opcional)
* Desplazamiento de una réplica secundaria (opcional)

La prueba de caos ejecuta varias iteraciones de errores y las validaciones de clúster para el período de tiempo especificado. También se puede configurar el tiempo empleado por el clúster para que la estabilización y la validación sean correctas. Se produce un error en el escenario cuando se encuentra un error en la validación del clúster.

Por ejemplo, considere un conjunto de pruebas que se va a ejecutar durante una hora con un máximo de tres errores simultáneos. La prueba inducirá tres errores y después validará el mantenimiento del clúster. La prueba recorrerá en iteración el paso anterior hasta que el clúster pase a ser incorrecto o transcurra una hora. Si el clúster pasa a ser incorrecto en cualquier iteración, es decir, no se estabiliza dentro de un tiempo configurado, la prueba producirá un error con una excepción. Esta excepción indica que algo salió mal y que se necesita más investigación.

En su forma actual, el motor de generación de errores de la prueba de caos induce solo errores seguros. Esto significa que en ausencia de errores externos nunca se producirá una pérdida de quórum o de datos.

### <a name="important-configuration-options"></a>Opciones de configuración importantes
* **TimeToRun**: tiempo total en el que se ejecutará la prueba antes de finalizarse con éxito. La prueba puede finalizarse antes en lugar de un error de validación.
* **MaxClusterStabilizationTimeout**: cantidad máxima de tiempo de espera para que el mantenimiento del clúster sea correcto antes de cancelar la prueba. Las comprobaciones realizadas son si el mantenimiento del clúster es correcto, el mantenimiento del servicio es correcto, se consigue el tamaño del conjunto de réplicas de destino para la partición de servicio y si no hay réplicas InBuild.
* **MaxConcurrentFaults**: número máximo de errores simultáneos inducidos en cada iteración. Cuanto mayor sea el número, más agresiva será la prueba. Por lo tanto, dará como resultado combinaciones de conmutaciones por error y de transición más complejas. La prueba garantiza que en ausencia de errores externos no habrá pérdida de quórum o de datos, con independencia de lo elevado del número de esta configuración.
* **EnableMoveReplicaFaults**: habilita o deshabilita los errores provocando el movimiento de las réplicas principales o secundarias. Estos errores están deshabilitados de forma predeterminada.
* **WaitTimeBetweenIterations**: cantidad de tiempo de espera entre iteraciones, es decir, después de una ronda de errores y de su validación correspondiente.

### <a name="how-to-run-the-chaos-test"></a>Ejecución de una prueba de caos
Ejemplo de C#

```csharp
using System;
using System.Fabric;
using System.Fabric.Testability.Scenario;
using System.Threading;
using System.Threading.Tasks;

class Test
{
    public static int Main(string[] args)
    {
        string clusterConnection = "localhost:19000";

        Console.WriteLine("Starting Chaos Test Scenario...");
        try
        {
            RunChaosTestScenarioAsync(clusterConnection).Wait();
        }
        catch (AggregateException ae)
        {
            Console.WriteLine("Chaos Test Scenario did not complete: ");
            foreach (Exception ex in ae.InnerExceptions)
            {
                if (ex is FabricException)
                {
                    Console.WriteLine("HResult: {0} Message: {1}", ex.HResult, ex.Message);
                }
            }
            return -1;
        }

        Console.WriteLine("Chaos Test Scenario completed.");
        return 0;
    }

    static async Task RunChaosTestScenarioAsync(string clusterConnection)
    {
        TimeSpan maxClusterStabilizationTimeout = TimeSpan.FromSeconds(180);
        uint maxConcurrentFaults = 3;
        bool enableMoveReplicaFaults = true;

        // Create FabricClient with connection and security information here.
        FabricClient fabricClient = new FabricClient(clusterConnection);

        // The chaos test scenario should run at least 60 minutes or until it fails.
        TimeSpan timeToRun = TimeSpan.FromMinutes(60);
        ChaosTestScenarioParameters scenarioParameters = new ChaosTestScenarioParameters(
          maxClusterStabilizationTimeout,
          maxConcurrentFaults,
          enableMoveReplicaFaults,
          timeToRun);

        // Other related parameters:
        // Pause between two iterations for a random duration bound by this value.
        // scenarioParameters.WaitTimeBetweenIterations = TimeSpan.FromSeconds(30);
        // Pause between concurrent actions for a random duration bound by this value.
        // scenarioParameters.WaitTimeBetweenFaults = TimeSpan.FromSeconds(10);

        // Create the scenario class and execute it asynchronously.
        ChaosTestScenario chaosScenario = new ChaosTestScenario(fabricClient, scenarioParameters);

        try
        {
            await chaosScenario.ExecuteAsync(CancellationToken.None);
        }
        catch (AggregateException ae)
        {
            throw ae.InnerException;
        }
    }
}
```

PowerShell

El módulo de PowerShell de Service Fabric incluye dos maneras de iniciar un escenario de caos. `Invoke-ServiceFabricChaosTestScenario` está basado en el cliente y, si la máquina cliente se apaga a lo largo de la prueba, no se introducirán más errores. Como alternativa, hay un conjunto de comandos pensado para mantener la prueba en ejecución en el caso de que se cierre la máquina. `Start-ServiceFabricChaos` utiliza un servicio de sistema con estado y confiable denominado FaultAnalysisService, que garantiza que los errores se seguirán introduciendo hasta que TimeToRun esté activo. `Stop-ServiceFabricChaos` se puede usar para detener manualmente el escenario y `Get-ServiceFabricChaosReport` obtendrá un informe. Para más información, consulte la [referencia de PowerShell para Azure Service Fabric](/powershell/module/servicefabric/?view=azureservicefabricps) y [Inducing controlled chaos in Service Fabric clusters](service-fabric-controlled-chaos.md) (Inducción del caos controlado en clústeres de Service Fabric).

```powershell
$connection = "localhost:19000"
$timeToRun = 60
$maxStabilizationTimeSecs = 180
$concurrentFaults = 3
$waitTimeBetweenIterationsSec = 60

Connect-ServiceFabricCluster $connection

Invoke-ServiceFabricChaosTestScenario -TimeToRunMinute $timeToRun -MaxClusterStabilizationTimeoutSec $maxStabilizationTimeSecs -MaxConcurrentFaults $concurrentFaults -EnableMoveReplicaFaults -WaitTimeBetweenIterationsSec $waitTimeBetweenIterationsSec
```


## <a name="failover-test"></a>Prueba de conmutación por error
El escenario de prueba de conmutación por error es una versión del escenario de prueba de caos dirigida a una partición de servicio específica. Comprueba el efecto de la conmutación por error en una partición de servicio específica al mismo tiempo que deja sin afectar los otros servicios. Una vez configurada con la información de la partición de destino y otros parámetros, se ejecuta como una herramienta de cliente mediante las API de C# o PowerShell para generar errores para una partición de servicio. El escenario se itera por una secuencia de errores simulados y una validación de servicio mientras la lógica de negocios se ejecuta en paralelo para proporcionar una carga de trabajo. Un error en la validación de servicio indica un problema que necesita más investigación.

### <a name="faults-simulated-in-the-failover-test"></a>Errores simulados en la prueba de conmutación por error
* Reinicio de un paquete de código implementado donde se hospeda la partición
* Eliminación de una instancia sin estado o de una réplica principal/secundaria
* Reinicio de una réplica principal/secundaria (si se conserva el servicio)
* Desplazamiento de una réplica principal
* Desplazamiento de una réplica secundaria
* Reinicio de la partición

La prueba de conmutación por error provoca un error seleccionado y después ejecuta la validación en el servicio para garantizar su estabilidad. La prueba de conmutación por error solo provoca un error a la ver en lugar de varios errores posibles en la prueba de caos. Si la partición de servicio no se estabiliza en el tiempo de espera configurado después del error, la prueba produce un error. La prueba provoca únicamente errores seguros. Esto significa que, en ausencia de errores externos, nunca se producirá una pérdida de quórum o de datos.

### <a name="important-configuration-options"></a>Opciones de configuración importantes
* **PartitionSelector**: objeto selector que especifica la partición a la que debe dirigirse.
* **TimeToRun**: tiempo total en el que se ejecutará la prueba antes de finalizarse.
* **MaxClusterStabilizationTimeout**: cantidad máxima de tiempo de espera para que el mantenimiento del clúster sea correcto antes que la prueba produzca un error. Las comprobaciones realizadas son si el mantenimiento del servicio es correcto, el tamaño del conjunto de réplicas de destino conseguido para todas las particiones y si no hay réplicas InBuild.
* **WaitTimeBetweenFaults**: cantidad de tiempo de espera entre cada ciclo de error y validación.

### <a name="how-to-run-the-failover-test"></a>Ejecución de la prueba de conmutación por error
**C#**

```csharp
using System;
using System.Fabric;
using System.Fabric.Testability.Scenario;
using System.Threading;
using System.Threading.Tasks;

class Test
{
    public static int Main(string[] args)
    {
        string clusterConnection = "localhost:19000";
        Uri serviceName = new Uri("fabric:/samples/PersistentToDoListApp/PersistentToDoListService");

        Console.WriteLine("Starting Chaos Test Scenario...");
        try
        {
            RunFailoverTestScenarioAsync(clusterConnection, serviceName).Wait();
        }
        catch (AggregateException ae)
        {
            Console.WriteLine("Chaos Test Scenario did not complete: ");
            foreach (Exception ex in ae.InnerExceptions)
            {
                if (ex is FabricException)
                {
                    Console.WriteLine("HResult: {0} Message: {1}", ex.HResult, ex.Message);
                }
            }
            return -1;
        }

        Console.WriteLine("Chaos Test Scenario completed.");
        return 0;
    }

    static async Task RunFailoverTestScenarioAsync(string clusterConnection, Uri serviceName)
    {
        TimeSpan maxServiceStabilizationTimeout = TimeSpan.FromSeconds(180);
        PartitionSelector randomPartitionSelector = PartitionSelector.RandomOf(serviceName);

        // Create FabricClient with connection and security information here.
        FabricClient fabricClient = new FabricClient(clusterConnection);

        // The chaos test scenario should run at least 60 minutes or until it fails.
        TimeSpan timeToRun = TimeSpan.FromMinutes(60);
        FailoverTestScenarioParameters scenarioParameters = new FailoverTestScenarioParameters(
          randomPartitionSelector,
          timeToRun,
          maxServiceStabilizationTimeout);

        // Other related parameters:
        // Pause between two iterations for a random duration bound by this value.
        // scenarioParameters.WaitTimeBetweenIterations = TimeSpan.FromSeconds(30);
        // Pause between concurrent actions for a random duration bound by this value.
        // scenarioParameters.WaitTimeBetweenFaults = TimeSpan.FromSeconds(10);

        // Create the scenario class and execute it asynchronously.
        FailoverTestScenario failoverScenario = new FailoverTestScenario(fabricClient, scenarioParameters);

        try
        {
            await failoverScenario.ExecuteAsync(CancellationToken.None);
        }
        catch (AggregateException ae)
        {
            throw ae.InnerException;
        }
    }
}
```


**PowerShell**

```powershell
$connection = "localhost:19000"
$timeToRun = 60
$maxStabilizationTimeSecs = 180
$waitTimeBetweenFaultsSec = 10
$serviceName = "fabric:/SampleApp/SampleService"

Connect-ServiceFabricCluster $connection

Invoke-ServiceFabricFailoverTestScenario -TimeToRunMinute $timeToRun -MaxServiceStabilizationTimeoutSec $maxStabilizationTimeSecs -WaitTimeBetweenFaultsSec $waitTimeBetweenFaultsSec -ServiceName $serviceName -PartitionKindSingleton
```
