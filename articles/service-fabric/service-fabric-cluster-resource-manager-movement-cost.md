---
title: 'Resource Manager para el clúster de Service Fabric: Costo del movimiento'
description: Obtenga información sobre el costo del movimiento de los servicios de Service Fabric y cómo puede especificarse para ajustarse a las necesidades arquitectónicas, incluida la configuración dinámica.
author: masnider
ms.topic: conceptual
ms.date: 08/18/2017
ms.author: masnider
ms.custom: devx-track-csharp
ms.openlocfilehash: 0fdcfb02851d56ed996ae4bf32671ab545782733
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89005350"
---
# <a name="service-movement-cost"></a>Costo del movimiento del servicio
Un factor que Cluster Resource Manager de Service Fabric toma en cuenta al intentar determinar qué cambios se deben realizar en un clúster es el costo de esos cambios. La noción de "costo" se valora frente a la cantidad de mejora que se puede introducir en el clúster. El costo se tiene en cuenta al mover los servicios por cuestiones de equilibrio, desfragmentación y otros requisitos. El objetivo es cumplir los requisitos de la forma menos perjudicial o costosa.

Mover servicios supone unos costos mínimos de tiempo de CPU y ancho de banda de red. Para los servicios con estado, requiere copiar el estado de dichos servicios, consumiendo memoria y espacio en disco adicionales. Minimizar el costo de las soluciones que Cluster Resource Manager de Azure Service Fabric proporciona ayuda a garantizar que los recursos del clúster no se utilizan de forma innecesaria. De todas formas, tampoco deben pasar por alto soluciones que mejorarían significativamente la asignación de recursos en el clúster.

Cluster Resource Manager cuenta con dos formas de calcular los costos y limitarlos, mientras administra el clúster. El primer mecanismo consiste simplemente en contar cada movimiento que realizaría. Si se generan dos soluciones con prácticamente el mismo equilibrio (puntuación), Cluster Resource Manager elige la que tiene el menor costo (número total de movimientos).

Esta estrategia da buenos resultados. Pero, al igual que con cargas predeterminadas o estáticas, es poco probable que en cualquier sistema complejo todos los movimientos sean iguales. Probablemente algunos sean mucho más costosos.

## <a name="setting-move-costs"></a>Establecimiento de los costos de movimiento 
Puede especificar el costo de movimiento predeterminado para un servicio cuando este se crea:

PowerShell:

```posh
New-ServiceFabricService -ApplicationName $applicationName -ServiceName $serviceName -ServiceTypeName $serviceTypeName –Stateful -MinReplicaSetSize 3 -TargetReplicaSetSize 3 -PartitionSchemeSingleton -DefaultMoveCost Medium
```

C#: 

```csharp
FabricClient fabricClient = new FabricClient();
StatefulServiceDescription serviceDescription = new StatefulServiceDescription();
//set up the rest of the ServiceDescription
serviceDescription.DefaultMoveCost = MoveCost.Medium;
await fabricClient.ServiceManager.CreateServiceAsync(serviceDescription);
```

También puede especificar o actualizar MoveCost de forma dinámica para un servicio después de que este se haya creado: 

PowerShell: 

```posh
Update-ServiceFabricService -Stateful -ServiceName "fabric:/AppName/ServiceName" -DefaultMoveCost High
```

C#:

```csharp
StatefulServiceUpdateDescription updateDescription = new StatefulServiceUpdateDescription();
updateDescription.DefaultMoveCost = MoveCost.High;
await fabricClient.ServiceManager.UpdateServiceAsync(new Uri("fabric:/AppName/ServiceName"), updateDescription);
```

## <a name="dynamically-specifying-move-cost-on-a-per-replica-basis"></a>Especificación de forma dinámica del costo de movimiento por réplica

Los fragmentos de código anteriores son todos para especificar MoveCost para un servicio completo de una sola vez desde fuera del propio servicio. Sin embargo, el costo de movimiento es más útil cuando el costo de movimiento de un objeto de servicio específico cambia a través de su tiempo medio de vida. Dado que los propios servicios tienen probablemente la idea más precisa del coste de su movimiento en un momento dado, hay una API para que los servicios informen sobre el costo de su propios movimientos en tiempo de ejecución. 

C#:

```csharp
this.Partition.ReportMoveCost(MoveCost.Medium);
```

## <a name="impact-of-move-cost"></a>Impacto del costo de movimiento
MoveCost tiene cinco niveles: Zero, Low, Medium, High y VeryHigh. Se aplican las reglas siguientes:

* Los valores de MoveCost están relacionados entre sí, excepto en el caso de Zero y VeryHigh. 
* Zero significa que mover una réplica es gratuito y que no se debe tener en cuenta en la puntuación de la solución.
* Establecer el costo de movimiento en High o VeryHigh *no* ofrece ninguna garantía con respecto a que la réplica no se moverá *nunca*.
* Las réplicas con el costo de movimiento VeryHigh solo se moverán si hay una infracción de restricción en el clúster que no se puede corregir de otra manera (aunque sea necesario mover muchas otras réplicas para corregir la infracción).



<center>

![Costo de movimiento como un factor en la selección de réplicas para el movimiento][Image1]
</center>

MoveCost ayuda a encontrar las soluciones que provocan la menor interrupción posible y que son más sencillas de conseguir a la vez que se llega a un equilibrio equivalente. La noción de costo de un servicio puede ser relativa a muchas cosas. Los factores más comunes para calcular el costo de movimiento son:

- La cantidad de estados o de datos que el servicio tiene que mover.
- El costo de desconexión de los clientes. El costo de mover una réplica principal es normalmente mayor que el costo de mover una réplica secundaria.
- El costo de interrumpir una operación en curso. Algunas operaciones en el nivel de almacén de datos o las operaciones realizadas en respuesta a una llamada del cliente son costosas. Llegadas a un cierto punto, no quiere detenerlas si no es necesario. Por lo tanto, mientras la operación está en ejecución, aumenta el costo de movimiento de este objeto de servicio para disminuir la probabilidad de que se mueva. Cuando finaliza la operación, devuelve el costo a la normalidad.

> [!IMPORTANT]
> Se debe considerar con cuidado el uso del costo de movimiento VeryHigh, ya que restringe significativamente la capacidad de Cluster Resource Manager para encontrar una solución de selección de ubicación globalmente óptima en el clúster. Las réplicas con el costo de movimiento VeryHigh solo se moverán si hay una infracción de restricción en el clúster que no se puede corregir de otra manera (aunque sea necesario mover muchas otras réplicas para corregir la infracción).

## <a name="enabling-move-cost-in-your-cluster"></a>Habilitación del costo de movimiento en el clúster
Para que sea posible tener en cuenta el MoveCost más pormenorizado, MoveCost tiene que estar habilitado en el clúster. Sin esta configuración, se utilizará el modo predeterminado de recuento de movimientos para calcular MoveCost y se omiten los informes de MoveCost.


ClusterManifest.xml:

``` xml
        <Section Name="PlacementAndLoadBalancing">
            <Parameter Name="UseMoveCostReports" Value="true" />
        </Section>
```

a través de ClusterConfig.json para las implementaciones independientes o Template.json para los clústeres hospedados en Azure:

```json
"fabricSettings": [
  {
    "name": "PlacementAndLoadBalancing",
    "parameters": [
      {
          "name": "UseMoveCostReports",
          "value": "true"
      }
    ]
  }
]
```

## <a name="next-steps"></a>Pasos siguientes
- Cluster Resource Manger de Service Fabric usa métricas para administrar el consumo y la capacidad en el clúster. Para más información sobre las métricas y cómo configurarlas, consulte [Administración de consumo y carga de recursos en Service Fabric con métricas](service-fabric-cluster-resource-manager-metrics.md).
- Para más información sobre la forma en que Cluster Resource Manager administra y equilibra la carga en el clúster, consulte [Equilibrio del clúster de Service Fabric](service-fabric-cluster-resource-manager-balancing.md).

[Image1]:./media/service-fabric-cluster-resource-manager-movement-cost/service-most-cost-example.png
