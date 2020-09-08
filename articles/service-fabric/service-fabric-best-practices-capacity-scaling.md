---
title: Escalado y planeamiento de capacidad de Azure Service Fabric
description: Procedimientos recomendados para el planeamiento y escalado de aplicaciones y clústeres de Service Fabric.
author: peterpogorski
ms.topic: conceptual
ms.date: 04/25/2019
ms.author: pepogors
ms.custom: devx-track-csharp
ms.openlocfilehash: e6b6cebfd146ffe23bdc21751f86c71d14ea875e
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89002256"
---
# <a name="capacity-planning-and-scaling-for-azure-service-fabric"></a>Escalado y planeamiento de capacidad de Azure Service Fabric

Es importante planear la capacidad antes de crear cualquier clúster de Azure Service Fabric o de escalar los recursos de proceso que hospedan al clúster. Para obtener más información sobre el planeamiento de la capacidad, consulte [planeamiento de capacidad del clúster de Service Fabric](./service-fabric-cluster-capacity.md). Para obtener instrucciones adicionales recomendadas para la escalabilidad del clúster, consulte [consideraciones sobre escalabilidad de Service Fabric](/azure/architecture/reference-architectures/microservices/service-fabric#scalability-considerations).

Además de tener en cuenta las características de clúster y el tipo de nodo, debería esperar que las operaciones de escalado tarden más de una hora en completarse en un entorno de producción. Esta consideración es cierta independientemente del número de máquinas virtuales que se vayan a agregar.

## <a name="autoscaling"></a>Escalado automático
Debe realizar las operaciones de escalado con plantillas de Azure Resource Manager, porque es el procedimiento recomendado para tratar [configuraciones de recursos como código](./service-fabric-best-practices-infrastructure-as-code.md). 

El uso del escalado automático a través de conjuntos de escalado de máquinas virtuales hará que la plantilla de Resource Manager con control de versiones defina sus recuentos de instancias para conjuntos de escalado de máquinas virtuales de manera imprecisa. Una definición imprecisa aumenta el riesgo de que las futuras implementaciones provoquen operaciones de escalado no intencionadas. En general, debe usar el escalado automático si:

* Si implementar las plantillas de Resource Manager con la capacidad adecuada declarada no es compatible con su caso de uso.
     
   Además del escalado manual, puede configurar una [canalización de entrega e integración continua en Azure DevOps Services mediante el uso de proyectos de implementación de grupo de recursos de Azure](../azure-resource-manager/templates/add-template-to-azure-pipelines.md). Esta canalización la desencadena habitualmente una aplicación lógica que usa las métricas de rendimiento de máquinas virtuales consultadas desde la [API REST de Azure Monitor](../azure-monitor/platform/rest-api-walkthrough.md). La canalización se escala automáticamente de forma eficiente basándose en las métricas que quiere mientras se optimiza para las plantillas de Resource Manager.
* Necesita escalar horizontalmente un nodo del conjunto de escalado de máquinas virtuales a la vez.
   
   Para escalar horizontalmente mediante tres o más nodos a la vez, debe [escalar horizontalmente un clúster de Service Fabric mediante la adición de un conjunto de escalado de máquinas virtuales](virtual-machine-scale-set-scale-node-type-scale-out.md). Resulta más seguro reducir horizontalmente y escalar horizontalmente un nodo de los conjuntos de escalado de máquinas virtuales a la vez.
* Cuenta con un nivel de fiabilidad Silver o superior en su clúster de Service Fabric y un nivel de durabilidad Silver o superior en cualquier escalado en el que configure reglas de escalado automático.
  
   La capacidad mínima de reglas de escalado automático debe ser igual o mayor que cinco instancias de máquina virtual. También debe ser igual o mayor que el mínimo nivel de confianza para el tipo de nodo principal.

> [!NOTE]
> El servicio con estado de Service Fabric fabric:/System/InfastructureService/<NOMBRE_TIPO_NODO> se ejecuta en cada tipo de nodo que tenga durabilidad Silver o superior. Es el único servicio de sistema que es compatible con la ejecución en Azure en cualquiera de los tipos de nodo de clústeres.

## <a name="vertical-scaling-considerations"></a>Consideraciones sobre escalado vertical

Para [escalar verticalmente](./virtual-machine-scale-set-scale-node-type-scale-out.md) un tipo de nodo en Azure Service Fabric, se necesitan una serie de pasos y consideraciones. Por ejemplo:

* El clúster debe estar en buen estado antes de escalarlo. En caso contrario, se desestabilizará aún más el clúster.
* Se requiere un nivel de durabilidad Silver o superior en todos los tipos de nodo de clúster de Service Fabric que hospedan servicios con estado.

> [!NOTE]
> El tipo de nodo principal que hospeda los servicios del sistema con estado de Service Fabric deben tener un nivel de durabilidad Silver o superior. Tras habilitar la durabilidad nivel Silver, las operaciones del clúster tales como las actualizaciones o la adición o eliminación de nodos serán más lentas, ya que el sistema realiza optimizaciones para la seguridad de los datos y no para la velocidad de las operaciones.

La realización del escalado vertical de un conjunto de escalado de máquinas virtuales es una operación destructiva. En su lugar, escale horizontalmente el clúster mediante la adición de un nuevo conjunto de escalado con la SKU que quiera. Luego, migre los servicios a la SKU que quiera para completar una operación de escalado vertical segura. El cambio de la SKU del recurso de conjunto de escalado de máquinas virtuales es una operación destructiva ya que restablece la imagen inicial de los hosts, lo que elimina todos los estados que persistan de forma local.

El clúster usa las [restricciones de ubicación y propiedades de nodo](./service-fabric-cluster-resource-manager-cluster-description.md#node-properties-and-placement-constraints) de Service Fabric para determinar en qué ubicación hospedará los servicios de las aplicaciones. Cuando se escale verticalmente el tipo de nodo principal, declare valores de propiedad idénticos para `"nodeTypeRef"`. Puede encontrar estos valores en la extensión de Service Fabric para conjuntos de escalado de máquinas virtuales. 

El siguiente fragmento de una plantilla de Resource Manager muestra las propiedades que se van a declarar. Tiene el mismo valor para los conjuntos de escalado recién aprovisionados a los que se está escalando y solo se admite como un servicio con estado temporal del clúster.

```json
"settings": {
   "nodeTypeRef": ["[parameters('primaryNodetypeName')]"]
}
```

> [!NOTE]
> No deje el clúster ejecutándose con varios conjuntos de escalado que usen los mismos valores de la propiedad `nodeTypeRef` durante más tiempo del necesario para que la operación de escalado vertical se realice correctamente.
>
> Valide siempre las operaciones en entornos de prueba antes de intentar realizar cambios en el entorno de producción. De manera predeterminada, los servicios de sistema del clúster de Service Fabric tienen una restricción de ubicación solo para el tipo de nodo principal de destino.

Una vez declaradas las restricciones de ubicación y las propiedades de nodo, realice los siguientes pasos en una instancia de máquina virtual a la vez. De este modo, los servicios del sistema (y los servicios con estado) se apagarán correctamente en la instancia de máquina virtual que se va a quitar a la vez que se crean nuevas réplicas en otros nodos.

1. En PowerShell, ejecute `Disable-ServiceFabricNode` con la intención `RemoveNode` para deshabilitar el nodo que se va a quitar. Quite el tipo de nodo que tiene el número más alto. Por ejemplo, si tiene un clúster de seis nodos, quite la instancia de máquina virtual "MyNodeType_5".
2. Ejecute `Get-ServiceFabricNode` para asegurarse de que el nodo ya está como deshabilitado. Si no es así, espere hasta que se deshabilite. Esta operación puede tardar un par de horas por cada nodo. No continúe hasta que el nodo esté como deshabilitado.
3. Disminuya en uno el número de máquinas virtuales de ese tipo de nodo. Se quitará la instancia de máquina virtual más alta.
4. Repita los pasos del 1 al 3 según sea necesario, pero nunca reduzca horizontalmente las instancias de los tipos de nodo principal a un número inferior al que garantiza el nivel de confiabilidad. Consulte el [planeamiento de la capacidad del clúster de Service Fabric](./service-fabric-cluster-capacity.md) para obtener una lista de instancias recomendadas.
5. Una vez que todas las máquinas virtuales hayan desaparecido (representadas como "Fuera de servicio"), fabric:/System/InfrastructureService/[nombre de nodo] mostrará un estado de Error. Luego, puede actualizar el recurso de clúster para quitar el tipo de nodo. Puede usar la implementación de plantilla de Resource Manager o editar el recurso de clúster a través de [Azure Resource Manager](https://resources.azure.com). Esta acción iniciará una actualización de clúster que quitará al servicio fabric:/System/InfrastructureService/[tipo de nodo] que se encuentra en estado de error.
 6. Después de eliminar, si quiere, VMScaleSet, seguirá viendo los nodos como "Fuera de servicio" en la vista de Service Fabric Explorer. El último paso sería limpiarlos con el comando `Remove-ServiceFabricNodeState`.

## <a name="horizontal-scaling"></a>Escalado horizontal

Puede realizar el escalado horizontal [manualmente](./service-fabric-cluster-scale-in-out.md) o [mediante programación](./service-fabric-cluster-programmatic-scaling.md).

> [!NOTE]
> Si va a escalar un tipo de nodo con una durabilidad de nivel Silver o Gold, el escalado será lento.

### <a name="scaling-out"></a>Escalado horizontal

Puede escalar horizontalmente un clúster de Service Fabric al aumentar el número de instancias para un determinado conjunto de escalado de máquinas virtuales. Puede escalar horizontalmente mediante programación si utiliza `AzureClient` y el identificador del conjunto de escalado para aumentar su capacidad.

```csharp
var scaleSet = AzureClient.VirtualMachineScaleSets.GetById(ScaleSetId);
var newCapacity = (int)Math.Min(MaximumNodeCount, scaleSet.Capacity + 1);
scaleSet.Update().WithCapacity(newCapacity).Apply(); 
```

Para escalar horizontalmente de forma manual, actualice la capacidad de la propiedad SKU del recurso de [conjunto de escalado de máquinas virtuales](/rest/api/compute/virtualmachinescalesets/createorupdate#virtualmachinescalesetosprofile) de su elección.

```json
"sku": {
    "name": "[parameters('vmNodeType0Size')]",
    "capacity": "[parameters('nt0InstanceCount')]",
    "tier": "Standard"
}
```

### <a name="scaling-in"></a>Reducir horizontalmente

La reducción horizontal requiere más atención que el escalado horizontal. Por ejemplo:

* Los servicios del sistema de Service Fabric se ejecutan en el tipo de nodo principal del clúster. Nunca apague las instancias de esos tipos de nodo ni las reduzca horizontalmente a un número inferior al que garantiza el nivel de confiabilidad. 
* Para un servicio con estado, necesita que un determinado número de nodos siempre esté activo para mantener la disponibilidad y preservar el estado del servicio. Como mínimo, necesita que el número de nodos sea igual al recuento de conjuntos de réplicas de destino de la partición o el servicio.

Para reducir horizontalmente de forma manual, siga estos pasos:

1. En PowerShell, ejecute `Disable-ServiceFabricNode` con la intención `RemoveNode` para deshabilitar el nodo que se va a quitar. Quite el tipo de nodo que tiene el número más alto. Por ejemplo, si tiene un clúster de seis nodos, quite la instancia de máquina virtual "MyNodeType_5".
2. Ejecute `Get-ServiceFabricNode` para asegurarse de que el nodo ya está como deshabilitado. Si no es así, espere hasta que se deshabilite. Esta operación puede tardar un par de horas por cada nodo. No continúe hasta que el nodo esté como deshabilitado.
3. Disminuya en uno el número de máquinas virtuales de ese tipo de nodo. Se quitará la instancia de máquina virtual más alta.
4. Repita los pasos del 1 al 3, según sea necesario, hasta que aprovisione la capacidad que quiera. Nunca reduzca horizontalmente las instancias de los tipos de nodo principal a un número inferior al que garantiza el nivel de confiabilidad. Consulte el [planeamiento de la capacidad del clúster de Service Fabric](./service-fabric-cluster-capacity.md) para obtener una lista de instancias recomendadas.

Para reducir horizontalmente de forma manual, actualice la capacidad de la propiedad SKU del recurso de [conjunto de escalado de máquinas virtuales](/rest/api/compute/virtualmachinescalesets/createorupdate#virtualmachinescalesetosprofile) de su elección.

```json
"sku": {
    "name": "[parameters('vmNodeType0Size')]",
    "capacity": "[parameters('nt0InstanceCount')]",
    "tier": "Standard"
}
```

Debe preparar el nodo para que se apague con el fin de realizar una reducción horizontal mediante programación. Busque el nodo que se va a quitar (el nodo de instancia más alta). Por ejemplo:

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

Desactive y quite el nodo con la misma instancia de `FabricClient` (en este caso, `client`) y la misma instancia de nodo (en este caso, `instanceIdString`) que ha usado en el código anterior:

```csharp
var scaleSet = AzureClient.VirtualMachineScaleSets.GetById(ScaleSetId);

// Remove the node from the Service Fabric cluster
ServiceEventSource.Current.ServiceMessage(Context, $"Disabling node {mostRecentLiveNode.NodeName}");
await client.ClusterManager.DeactivateNodeAsync(mostRecentLiveNode.NodeName, NodeDeactivationIntent.RemoveNode);

// Wait (up to a timeout) for the node to gracefully shut down
var timeout = TimeSpan.FromMinutes(5);
var waitStart = DateTime.Now;
while ((mostRecentLiveNode.NodeStatus == System.Fabric.Query.NodeStatus.Up || mostRecentLiveNode.NodeStatus == System.Fabric.Query.NodeStatus.Disabling) &&
        DateTime.Now - waitStart < timeout)
{
    mostRecentLiveNode = (await client.QueryManager.GetNodeListAsync()).FirstOrDefault(n => n.NodeName == mostRecentLiveNode.NodeName);
    await Task.Delay(10 * 1000);
}

// Decrement virtual machine scale set capacity
var newCapacity = (int)Math.Max(MinimumNodeCount, scaleSet.Capacity - 1); // Check min count 

scaleSet.Update().WithCapacity(newCapacity).Apply();
```

> [!NOTE]
> Al reducir horizontalmente un clúster, verá la instancia quitada del nodo o la máquina virtual mostrada en un estado incorrecto en Service Fabric Explorer. Para obtener una explicación de este comportamiento, consulte [Comportamientos que se pueden observar en Service Fabric Explorer](./service-fabric-cluster-scale-in-out.md#behaviors-you-may-observe-in-service-fabric-explorer). Puede:
> * Llamar al [comando Remove-ServiceFabricNodeState](/powershell/module/servicefabric/remove-servicefabricnodestate?view=azureservicefabricps) con el nombre de nodo adecuado.
> * Implementar la [aplicación auxiliar de escalabilidad automática de Service Fabric](https://github.com/Azure/service-fabric-autoscale-helper/) en el clúster. Esta aplicación garantiza que los nodos que se han reducido verticalmente se borran de Service Fabric Explorer.

## <a name="reliability-levels"></a>Niveles de confiabilidad

El [nivel de fiabilidad](./service-fabric-cluster-capacity.md) es una propiedad de los recursos de clúster de Service Fabric. No se puede configurar de forma distinta para los tipos de nodo individuales. Controla el factor de replicación de los servicios del sistema para el clúster y es una configuración a nivel de recursos de clúster. 

El nivel de confiabilidad determinará el número mínimo de nodos que debe tener el tipo de nodo principal. El nivel de confiabilidad puede adoptar los siguientes valores:

* Platinum: ejecuta los servicios del sistema con un objetivo de recuento de conjunto de réplicas de siete y nueve nodos de inicialización.
* Gold: ejecuta los servicios del sistema con un objetivo de recuento de conjunto de réplicas de siete y siete nodos de inicialización.
* Silver: ejecuta los servicios del sistema con un objetivo de recuento de conjunto de réplicas de cinco y cinco nodos de inicialización.
* Bronze: ejecuta los servicios del sistema con un objetivo de recuento de conjunto de réplicas de tres y tres nodos raíz.

El nivel de confiabilidad mínima recomendada es Silver.

El nivel de confiabilidad se establece en la sección de propiedades del [recurso Microsoft.servicefabric/clusters](/azure/templates/microsoft.servicefabric/2018-02-01/clusters) de la siguiente forma:

```json
"properties":{
    "reliabilityLevel": "Silver"
}
```

## <a name="durability-levels"></a>Niveles de durabilidad

> [!WARNING]
> Los tipos de nodos que se ejecutan con la durabilidad Bronze _no obtienen privilegios_. Los trabajos de infraestructura que afectan a las cargas de trabajo sin estado no se detendrán ni retrasarán, lo que podría afectar a las cargas de trabajo. 
>
> Utilice el nivel de durabilidad Bronze en los tipos de nodos que ejecutan cargas de trabajo sin estado. Para las cargas de trabajo de producción, ejecute el nivel Silver o uno superior con el fin de garantizar la coherencia del estado. Elija la confiabilidad adecuada según las instrucciones de la [documentación de planeamiento de capacidad](./service-fabric-cluster-capacity.md).

El nivel de durabilidad se debe establecer en los dos recursos. Uno es el perfil de extensión del [recurso del conjunto de escalado de máquinas virtuales](/rest/api/compute/virtualmachinescalesets/createorupdate#virtualmachinescalesetosprofile):

```json
"extensionProfile": {
    "extensions":          {
        "name": "[concat('ServiceFabricNodeVmExt','_vmNodeType0Name')]",
        "properties": {
            "settings": {
                "durabilityLevel": "Bronze"
            }
        }
    }
}
```

El otro recurso está en `nodeTypes` del [recurso Microsoft.ServiceFabric/clusters](/azure/templates/microsoft.servicefabric/2018-02-01/clusters): 

```json
"nodeTypes": [
    {
        "name": "[variables('vmNodeType0Name')]",
        "durabilityLevel": "Bronze"
    }
]
```

## <a name="next-steps"></a>Pasos siguientes

* Creación de un clúster en máquinas virtuales o equipos que ejecutan Windows Server: [Creación de un clúster de Service Fabric para Windows Server](service-fabric-cluster-creation-for-windows-server.md).
* Creación de un clúster en máquinas virtuales o equipos que ejecutan Linux: [Creación de un clúster de Linux](service-fabric-cluster-creation-via-portal.md).
* Más información sobre las [opciones de soporte técnico de Service Fabric](service-fabric-support.md).

[Image1]: ./media/service-fabric-best-practices/generate-common-name-cert-portal.png