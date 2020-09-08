---
title: Escalado mediante programación de Azure Service Fabric
description: Escalar o reducir horizontalmente mediante programación un clúster de Azure Service Fabric, de acuerdo a desencadenadores personalizados
author: mjrousos
ms.topic: conceptual
ms.date: 01/23/2018
ms.author: mikerou
ms.custom: devx-track-csharp
ms.openlocfilehash: bd47e5e39684bd4b684cd1e12dd9a3d420640ee2
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89005843"
---
# <a name="scale-a-service-fabric-cluster-programmatically"></a>Escalado mediante programación de un clúster de Service Fabric 

Los clústeres de Service Fabric que se ejecutan en Azure se basan en conjuntos de escalado de máquinas virtuales.  En el artículo de [escalado de clústeres](./service-fabric-cluster-scale-in-out.md) se describe la manera de escalar los clústeres de Service Fabric, ya sea manualmente o con reglas de escalado automático. En este artículo se describe cómo administrar las credenciales y reducir o escalar horizontalmente un clúster en mediante el SDK de proceso fluido de Azure, que es un escenario más avanzado. Para una introducción, lea la sección de [métodos de programación para coordinar las operaciones de escalado de Azure](service-fabric-cluster-scaling.md#programmatic-scaling). 


[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="manage-credentials"></a>Administración de credenciales
Una dificultad que se presenta al escribir un servicio para controlar el escalado es que el servicio tiene que tener acceso a recursos de conjunto de escalado de máquinas virtuales sin un inicio de sesión interactivo. El acceso al clúster de Service Fabric es fácil si el servicio de escalado está modificando su propia aplicación de Service Fabric, pero se requieren credenciales para tener acceso al conjunto de escalado. Para iniciar sesión, puede usar una [entidad de servicio](/cli/azure/create-an-azure-service-principal-azure-cli) creada con la [CLI de Azure](https://github.com/azure/azure-cli).

Una entidad de servicio se puede crear con los pasos siguientes:

1. Inicie sesión en la CLI de Azure (`az login`) como un usuario con acceso al conjunto de escalado de máquinas virtuales.
2. Cree la entidad de servicio con `az ad sp create-for-rbac`
    1. Anote el appId (denominado "ID de cliente" en otros lugares), el nombre, la contraseña y el inquilino para su uso posterior.
    2. También necesitará el identificador de suscripción, que se puede ver con `az account list`

La biblioteca de proceso fluida puede iniciar sesión mediante estas credenciales de la siguiente manera (tenga en cuenta que los tipos fluidos básicos de Azure como `IAzure` están en el paquete [Microsoft.Azure.Management.Fluent](https://www.nuget.org/packages/Microsoft.Azure.Management.Fluent/)):

```csharp
var credentials = new AzureCredentials(new ServicePrincipalLoginInformation {
                ClientId = AzureClientId,
                ClientSecret = 
                AzureClientKey }, AzureTenantId, AzureEnvironment.AzureGlobalCloud);
IAzure AzureClient = Azure.Authenticate(credentials).WithSubscription(AzureSubscriptionId);

if (AzureClient?.SubscriptionId == AzureSubscriptionId)
{
    ServiceEventSource.Current.ServiceMessage(Context, "Successfully logged into Azure");
}
else
{
    ServiceEventSource.Current.ServiceMessage(Context, "ERROR: Failed to login to Azure");
}
```

Una vez iniciada la sesión, el recuento de instancias de conjunto de escalado se puede consultar a través de `AzureClient.VirtualMachineScaleSets.GetById(ScaleSetId).Capacity`.

## <a name="scaling-out"></a>Escalado horizontal
Con SDK de proceso fluido de Azure, se pueden agregar instancias al conjunto de escalado de máquinas virtuales con unas pocas llamadas.

```csharp
var scaleSet = AzureClient.VirtualMachineScaleSets.GetById(ScaleSetId);
var newCapacity = (int)Math.Min(MaximumNodeCount, scaleSet.Capacity + 1);
scaleSet.Update().WithCapacity(newCapacity).Apply(); 
``` 

Como alternativa, el tamaño del conjunto de escalado de máquinas virtuales también puede administrarse con cmdlets de PowerShell. [`Get-AzVmss`](/powershell/module/az.compute/get-azvmss) puede recuperar el objeto del conjunto de escalado de máquinas virtuales. La capacidad actual está disponible a través de la propiedad `.sku.capacity`. Después de cambiarla capacidad al valor deseado, el conjunto de escalado de máquinas virtuales en Azure puede actualizarse con el comando [`Update-AzVmss`](/powershell/module/az.compute/update-azvmss).

Tal y como cuando se agrega manualmente un nodo, agregar una instancia de conjunto de escalado debe ser todo lo que se necesita para iniciar un nuevo nodo de Service Fabric, ya que la plantilla de conjunto de escalado incluye extensiones para unir automáticamente nuevas instancias al clúster de Service Fabric. 

## <a name="scaling-in"></a>Reducir horizontalmente

Reducir horizontalmente es similar a escalar horizontalmente. Los cambios reales del conjunto de escalado de máquinas virtuales son prácticamente los mismos. Pero, tal y como se ha descrito anteriormente, Service Fabric solo limpia automáticamente los nodos eliminados con una durabilidad Gold o Silver. Por lo tanto, en el caso de reducción horizontal de durabilidad Bronze, es necesario interactuar con el clúster de Service Fabric para apagar el nodo que desea eliminar y a continuación eliminar su estado.

Preparar el nodo para el apagado implica buscar el nodo que se va a eliminar (la instancia del conjunto de escalado de máquinas virtuales más reciente) y desactivarlo. Las instancias del conjunto de escalado de máquinas virtuales se numeran en el orden en que se agregan, por lo que los nodos más recientes pueden encontrarse comparando el número de sufijo en los nombres de los nodos (que coinciden con los nombres de las instancias del conjunto de escalado de máquinas virtuales subyacente). 

```csharp
using (var client = new FabricClient())
{
    var mostRecentLiveNode = (await client.QueryManager.GetNodeListAsync())
        .Where(n => n.NodeType.Equals(NodeTypeToScale, StringComparison.OrdinalIgnoreCase))
        .Where(n => n.NodeStatus == System.Fabric.Query.NodeStatus.Up)
        .OrderByDescending(n =>
        {
            var instanceIdIndex = n.NodeName.LastIndexOf("_");
            var instanceIdString = n.NodeName.Substring(instanceIdIndex + 1);
            return int.Parse(instanceIdString);
        })
        .FirstOrDefault();
```

Cuando se encuentre el nodo que se va a quitar, puede desactivarse y eliminarse utilizando la misma instancia `FabricClient` y la `IAzure` que se usó anteriormente.

```csharp
var scaleSet = AzureClient.VirtualMachineScaleSets.GetById(ScaleSetId);

// Remove the node from the Service Fabric cluster
ServiceEventSource.Current.ServiceMessage(Context, $"Disabling node {mostRecentLiveNode.NodeName}");
await client.ClusterManager.DeactivateNodeAsync(mostRecentLiveNode.NodeName, NodeDeactivationIntent.RemoveNode);

// Wait (up to a timeout) for the node to gracefully shutdown
var timeout = TimeSpan.FromMinutes(5);
var waitStart = DateTime.Now;
while ((mostRecentLiveNode.NodeStatus == System.Fabric.Query.NodeStatus.Up || mostRecentLiveNode.NodeStatus == System.Fabric.Query.NodeStatus.Disabling) &&
        DateTime.Now - waitStart < timeout)
{
    mostRecentLiveNode = (await client.QueryManager.GetNodeListAsync()).FirstOrDefault(n => n.NodeName == mostRecentLiveNode.NodeName);
    await Task.Delay(10 * 1000);
}

// Decrement VMSS capacity
var newCapacity = (int)Math.Max(MinimumNodeCount, scaleSet.Capacity - 1); // Check min count 

scaleSet.Update().WithCapacity(newCapacity).Apply(); 
```

Como con el escalado, los cmdlets de PowerShell para la capacidad de modificación del conjunto de escalado de máquinas virtuales pueden usarse aquí si es preferible un enfoque de scripting. Una vez que se ha eliminado la instancia de máquina virtual, se puede quitar el estado del nodo de Service Fabric.

```csharp
await client.ClusterManager.RemoveNodeStateAsync(mostRecentLiveNode.NodeName);
```

## <a name="next-steps"></a>Pasos siguientes

Para empezar a implementar su propia lógica de escalado automático, familiarícese con los siguientes conceptos y API útiles:

- [Escalado manual o con reglas de escalado automático](./service-fabric-cluster-scale-in-out.md)
- [Bibliotecas fluidas de administración de Azure para .NET](https://github.com/Azure/azure-sdk-for-net/tree/Fluent) (resulta útil para interactuar con los conjuntos de escalado de máquinas virtuales subyacentes de un clúster de Service Fabric)
- [System.Fabric.FabricClient](/dotnet/api/system.fabric.fabricclient) (resulta útil para interactuar con un clúster de Service Fabric y sus nodos)
