---
title: Inicio y detención de nodos de clúster
description: Obtenga información acerca de cómo iniciar o detener nodos de clúster para usar la inserción de errores con el fin de probar una aplicación de Service Fabric.
author: LMWF
ms.topic: conceptual
ms.date: 6/12/2017
ms.author: lemai
ms.custom: devx-track-csharp
ms.openlocfilehash: 9c31040ec13084f9e4b08bbc9a347e4ad44975bf
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89021262"
---
# <a name="replacing-the-start-node-and-stop-node-apis-with-the-node-transition-api"></a>Sustitución de las API Start Node y Stop node con la API Node Transition

## <a name="what-do-the-stop-node-and-start-node-apis-do"></a>¿Qué hacen las API Stop Node y Start Node?

Stop Node API (administrada: [StopNodeAsync()][stopnode], PowerShell: [Stop-ServiceFabricNode][stopnodeps]) detiene un nodo de Service Fabric.  Un nodo de Service Fabric es un proceso, no un equipo ni una máquina virtual: el equipo o la máquina virtual seguirán en ejecución.  Para el resto del documento, "nodo" hace referencia a un nodo de Service Fabric.  Al detener un nodo, este adquiere el estado *detenido*, en el que no es un miembro del clúster y no puede hospedar servicios, por lo que simula un nodo *apagado*.  Esto es útil para insertar errores en el sistema a fin de probar la aplicación.  Start Node API (administrada: [StartNodeAsync()][startnode], PowerShell: [Start-ServiceFabricNode][startnodeps]]) invierte la Stop Node API, lo que hace que el nodo vuelva a un estado normal.

## <a name="why-are-we-replacing-these"></a>¿Por qué se realiza la sustitución?

Como se ha descrito anteriormente, un nodo de Service Fabric *detenido* es un nodo intencionalmente dirigido con la API Stop Node.  Un nodo *apagado* es un nodo que está apagado por cualquier otra razón (por ejemplo, porque el equipo o la máquina virtual están apagados).  Con la API Stop Node, el sistema no expone información para diferenciar entre los nodos *detenidos* y los nodos *apagados*.

Además, algunos errores devueltos por estas API no son tan descriptivos como debieran.  Por ejemplo, al invocar la API Stop Node en un nodo que ya está *detenido*, se devolverá el error *InvalidAddress*.  Se puede mejorar esta experiencia.

Además, el tiempo durante el cual un nodo está detenido es "infinito" hasta que se invoca la API Start Node.  Se ha detectado que esto puede ocasionar problemas y ser propenso a errores.  Por ejemplo, se han observado problemas en los que un usuario ha invocado la API Stop Node en un nodo y luego se ha olvidado de ello.  Luego no está claro si un nodo está *apagado* o *detenido*.


## <a name="introducing-the-node-transition-apis"></a>Introducción a las API Node Transition

Se han solucionado estos problemas anteriores en un nuevo conjunto de API.  La nueva Node Transition API (administrada: [StartNodeTransitionAsync()][snt]) puede utilizarse para realizar la transición de un nodo de Service Fabric a un estado *detenido*, o bien para realizar su transición de un estado *detenido* a un estado normal.  Tenga en cuenta que la palabra "Start" en el nombre de la API no hace referencia a iniciar un nodo.  Se refiere a comenzar una operación asincrónica que el sistema ejecutará para realizar la transición del nodo a un estado *detenido* o iniciado.

**Uso**

Si la API Node Transition no lanza una excepción al invocarla, significa que el sistema ha aceptado la operación asincrónica y que la ejecutará.  Una llamada correcta no implica que la operación haya finalizado aún.  Para obtener información acerca del estado actual de la operación, llame a la Node Transition Progress API (administrada: [GetNodeTransitionProgressAsync()][gntp]) con el GUID utilizado al invocar Node Transition API para realizar esta operación.  La API Node Transition Progress devuelve un objeto NodeTransitionProgress.  La propiedad State de este objeto especifica el estado actual de la operación.  Si el estado es “Running”, significa que la operación está en ejecución.  Si el estado es Completed, significa que la operación ha finalizado sin errores.  Si el estado es Faulted, significa que se ha producido un problema al ejecutar la operación.  La propiedad Exception de la propiedad Result indica de qué problema se trata.  Consulte https://docs.microsoft.com/dotnet/api/system.fabric.testcommandprogressstate para obtener más información sobre la propiedad Estado y la sección "Ejemplo de uso" más adelante para ver los ejemplos de código.


**Diferencias entre un nodo detenido y un nodo inactivo** Si un nodo se *detiene* mediante la Node Transition API, el resultado de una consulta de nodo (administrada: [GetNodeListAsync()][nodequery], PowerShell: [Get-ServiceFabricNode][nodequeryps]) mostrará que el valor de la propiedad *IsStopped* de este nodo es true.  Tenga en cuenta que este valor es diferente del valor de la propiedad *NodeStatus*, que será *Down* (Apagado).  Si la propiedad *NodeStatus* tiene el valor *Down*, pero *IsStopped* es false, entonces significa que el nodo no se ha detenido con la API Node Transition y que está *apagado* por alguna otra razón.  Si la propiedad *IsStopped* es true y la propiedad *NodeStatus* es *Down*, entonces significa que se ha detenido con la API Node Transition.

Al iniciar un nodo *detenido* con la API Node Transition, el nodo volverá a funcionar como un miembro normal del clúster.  El resultado de la API de consulta de nodo mostrará *IsStopped* como false, y *NodeStatus* con un estado distinto a Down (apagado), como por ejemplo, Up (encendido).


**Duración limitada** Al usar la API Node Transition para detener un nodo, uno de los parámetros necesarios *stopNodeDurationInSeconds*, representa la duración en segundos durante la cual el nodo debe estar *detenido*.  Este valor debe estar dentro del intervalo permitido, que tiene un mínimo de 600, y un máximo de 14400.  Después de que expire este tiempo, el propio nodo se reiniciará automáticamente con el estado Up.  Consulte el Ejemplo 1 a continuación para ver un ejemplo de uso.

> [!WARNING]
> Evite mezclar las API Node Transition y las API Stop Node y Start Node.  Se recomienda usar solo la API Node Transition.  Si un nodo ya está detenido con la API Stop Node, debe iniciarse con la API Start Node primero antes de usar las API Node Transition.

> [!WARNING]
> No se pueden realizar en paralelo varias llamadas a las API Node Transition en el mismo nodo.  En esta situación, la API Node Transition lanzará una excepción FabricException, donde el valor de la propiedad ErrorCode será NodeTransitionInProgress.  Una vez iniciada una transición de nodo de un nodo específico, debe esperar hasta que la operación llega a un estado terminal (Completed, Faulted o ForceCancelled) antes de iniciar una transición nueva en el mismo nodo.  Sí se permiten llamadas paralelas a las API Node Transition en diferentes nodos.


#### <a name="sample-usage"></a>Ejemplo de uso


**Ejemplo 1** - El ejemplo siguiente utiliza la API Node Transition para detener un nodo.

```csharp
        // Helper function to get information about a node
        static Node GetNodeInfo(FabricClient fc, string node)
        {
            NodeList n = null;
            while (n == null)
            {
                n = fc.QueryManager.GetNodeListAsync(node).GetAwaiter().GetResult();
                Task.Delay(TimeSpan.FromSeconds(1)).GetAwaiter();
            };

            return n.FirstOrDefault();
        }

        static async Task WaitForStateAsync(FabricClient fc, Guid operationId, TestCommandProgressState targetState)
        {
            NodeTransitionProgress progress = null;

            do
            {
                bool exceptionObserved = false;
                try
                {
                    progress = await fc.TestManager.GetNodeTransitionProgressAsync(operationId, TimeSpan.FromMinutes(1), CancellationToken.None).ConfigureAwait(false);
                }
                catch (OperationCanceledException oce)
                {
                    Console.WriteLine("Caught exception '{0}'", oce);
                    exceptionObserved = true;
                }
                catch (FabricTransientException fte)
                {
                    Console.WriteLine("Caught exception '{0}'", fte);
                    exceptionObserved = true;
                }

                if (!exceptionObserved)
                {
                    Console.WriteLine("Current state of operation '{0}': {1}", operationId, progress.State);

                    if (progress.State == TestCommandProgressState.Faulted)
                    {
                        // Inspect the progress object's Result.Exception.HResult to get the error code.
                        Console.WriteLine("'{0}' failed with: {1}, HResult: {2}", operationId, progress.Result.Exception, progress.Result.Exception.HResult);

                        // ...additional logic as required
                    }

                    if (progress.State == targetState)
                    {
                        Console.WriteLine("Target state '{0}' has been reached", targetState);
                        break;
                    }
                }

                await Task.Delay(TimeSpan.FromSeconds(5)).ConfigureAwait(false);
            }
            while (true);
        }

        static async Task StopNodeAsync(FabricClient fc, string nodeName, int durationInSeconds)
        {
            // Uses the GetNodeListAsync() API to get information about the target node
            Node n = GetNodeInfo(fc, nodeName);

            // Create a Guid
            Guid guid = Guid.NewGuid();

            // Create a NodeStopDescription object, which will be used as a parameter into StartNodeTransition
            NodeStopDescription description = new NodeStopDescription(guid, n.NodeName, n.NodeInstanceId, durationInSeconds);

            bool wasSuccessful = false;

            do
            {
                try
                {
                    // Invoke StartNodeTransitionAsync with the NodeStopDescription from above, which will stop the target node.  Retry transient errors.
                    await fc.TestManager.StartNodeTransitionAsync(description, TimeSpan.FromMinutes(1), CancellationToken.None).ConfigureAwait(false);
                    wasSuccessful = true;
                }
                catch (OperationCanceledException oce)
                {
                    // This is retryable
                }
                catch (FabricTransientException fte)
                {
                    // This is retryable
                }

                // Backoff
                await Task.Delay(TimeSpan.FromSeconds(5)).ConfigureAwait(false);
            }
            while (!wasSuccessful);

            // Now call StartNodeTransitionProgressAsync() until the desired state is reached.
            await WaitForStateAsync(fc, guid, TestCommandProgressState.Completed).ConfigureAwait(false);
        }
```

**Ejemplo 2** - El ejemplo siguiente inicia un nodo *detenido*.  Usa algunos métodos auxiliares del primer ejemplo.

```csharp
        static async Task StartNodeAsync(FabricClient fc, string nodeName)
        {
            // Uses the GetNodeListAsync() API to get information about the target node
            Node n = GetNodeInfo(fc, nodeName);

            Guid guid = Guid.NewGuid();
            BigInteger nodeInstanceId = n.NodeInstanceId;

            // Create a NodeStartDescription object, which will be used as a parameter into StartNodeTransition
            NodeStartDescription description = new NodeStartDescription(guid, n.NodeName, nodeInstanceId);

            bool wasSuccessful = false;

            do
            {
                try
                {
                    // Invoke StartNodeTransitionAsync with the NodeStartDescription from above, which will start the target stopped node.  Retry transient errors.
                    await fc.TestManager.StartNodeTransitionAsync(description, TimeSpan.FromMinutes(1), CancellationToken.None).ConfigureAwait(false);
                    wasSuccessful = true;
                }
                catch (OperationCanceledException oce)
                {
                    Console.WriteLine("Caught exception '{0}'", oce);
                }
                catch (FabricTransientException fte)
                {
                    Console.WriteLine("Caught exception '{0}'", fte);
                }

                await Task.Delay(TimeSpan.FromSeconds(5)).ConfigureAwait(false);

            }
            while (!wasSuccessful);

            // Now call StartNodeTransitionProgressAsync() until the desired state is reached.
            await WaitForStateAsync(fc, guid, TestCommandProgressState.Completed).ConfigureAwait(false);
        }
```

**Ejemplo 3** - El ejemplo siguiente muestra un uso incorrecto.  Este uso es incorrecto porque el valor *stopDurationInSeconds* que proporciona es mayor que el intervalo permitido.  Teniendo en cuenta que StartNodeTransitionAsync() provocará un error grave, la operación no se acepta y no se debe realizar ninguna llamada a la API de progreso.  Este ejemplo usa algunos métodos auxiliares del primer ejemplo.

```csharp
        static async Task StopNodeWithOutOfRangeDurationAsync(FabricClient fc, string nodeName)
        {
            Node n = GetNodeInfo(fc, nodeName);

            Guid guid = Guid.NewGuid();

            // Use an out of range value for stopDurationInSeconds to demonstrate error
            NodeStopDescription description = new NodeStopDescription(guid, n.NodeName, n.NodeInstanceId, 99999);

            try
            {
                await fc.TestManager.StartNodeTransitionAsync(description, TimeSpan.FromMinutes(1), CancellationToken.None).ConfigureAwait(false);
            }

            catch (FabricException e)
            {
                Console.WriteLine("Caught {0}", e);
                Console.WriteLine("ErrorCode {0}", e.ErrorCode);
                // Output:
                // Caught System.Fabric.FabricException: System.Runtime.InteropServices.COMException (-2147017629)
                // StopDurationInSeconds is out of range ---> System.Runtime.InteropServices.COMException: Exception from HRESULT: 0x80071C63
                // << Parts of exception omitted>>
                //
                // ErrorCode InvalidDuration
            }
        }
```

**Ejemplo 4** - El ejemplo siguiente muestra la información del error que la API Node Transition Progress devolverá cuando se acepte la operación iniciada por la API Node Transition, pero se produce un error más adelante durante la ejecución.  En este caso, el error se produce porque la API Node Transition trata de iniciar un nodo que no existe.  Este ejemplo usa algunos métodos auxiliares del primer ejemplo.

```csharp
        static async Task StartNodeWithNonexistentNodeAsync(FabricClient fc)
        {
            Guid guid = Guid.NewGuid();
            BigInteger nodeInstanceId = 12345;

            // Intentionally use a nonexistent node
            NodeStartDescription description = new NodeStartDescription(guid, "NonexistentNode", nodeInstanceId);

            bool wasSuccessful = false;

            do
            {
                try
                {
                    // Invoke StartNodeTransitionAsync with the NodeStartDescription from above, which will start the target stopped node.  Retry transient errors.
                    await fc.TestManager.StartNodeTransitionAsync(description, TimeSpan.FromMinutes(1), CancellationToken.None).ConfigureAwait(false);
                    wasSuccessful = true;
                }
                catch (OperationCanceledException oce)
                {
                    Console.WriteLine("Caught exception '{0}'", oce);
                }
                catch (FabricTransientException fte)
                {
                    Console.WriteLine("Caught exception '{0}'", fte);
                }

                await Task.Delay(TimeSpan.FromSeconds(5)).ConfigureAwait(false);

            }
            while (!wasSuccessful);

            // Now call StartNodeTransitionProgressAsync() until the desired state is reached.  In this case, it will end up in the Faulted state since the node does not exist.
            // When StartNodeTransitionProgressAsync()'s returned progress object has a State if Faulted, inspect the progress object's Result.Exception.HResult to get the error code.
            // In this case, it will be NodeNotFound.
            await WaitForStateAsync(fc, guid, TestCommandProgressState.Faulted).ConfigureAwait(false);
        }
```

[stopnode]: /dotnet/api/system.fabric.fabricclient.faultmanagementclient
[stopnodeps]: /previous-versions/azure/mt125982(v=azure.100)
[startnode]: /dotnet/api/system.fabric.fabricclient.faultmanagementclient
[startnodeps]: /previous-versions/azure/mt163520(v=azure.100)
[nodequery]: /dotnet/api/system.fabric.fabricclient.queryclient
[nodequeryps]: /powershell/module/servicefabric/get-servicefabricnode
[snt]: /dotnet/api/system.fabric.fabricclient.testmanagementclient
[gntp]: /dotnet/api/system.fabric.fabricclient.testmanagementclient
