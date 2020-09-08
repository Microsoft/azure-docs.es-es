---
title: Eliminación de un tipo de nodo en Azure Service Fabric | Microsoft Docs
description: Aprenda a quitar un tipo de nodo de un clúster de Service Fabric que se ejecute en Azure.
author: inputoutputcode
manager: sridmad
ms.topic: conceptual
ms.date: 08/11/2020
ms.author: chrpap
ms.openlocfilehash: ede999bee9ce1a4a9dd10652a2c52a840d5b24be
ms.sourcegitcommit: c28fc1ec7d90f7e8b2e8775f5a250dd14a1622a6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/13/2020
ms.locfileid: "88163584"
---
# <a name="how-to-remove-a-service-fabric-node-type"></a>Eliminación de un tipo de nodo de Service Fabric
En este artículo, se explica cómo escalar un clúster de Azure Service Fabric quitando un tipo de nodo existente de un clúster. Un clúster de Service Fabric es un conjunto de máquinas físicas o virtuales conectadas a la red, en las que se implementan y administran los microservicios. Un equipo o máquina virtual que forma parte de un clúster se denomina nodo. Los conjuntos de escalado de máquinas virtuales son un recurso de proceso de Azure que se puede usar para implementar y administrar una colección de máquinas virtuales de forma conjunta. Cada tipo de nodo que se define en un clúster de Azure está [configurado como un conjunto de escalado independiente](service-fabric-cluster-nodetypes.md). Cada tipo de nodo, a continuación, se puede administrar por separado. Después de crear un clúster de Service Fabric, puede escalarlo horizontalmente quitando un tipo de nodo (conjunto de escalado de máquinas virtuales) junto con todos sus nodos.  Puede escalar el clúster en cualquier momento, incluso con cargas de trabajo en ejecución en el clúster.  Según se escala el clúster, las aplicaciones se escalan automáticamente.

> [!WARNING]
> No se recomienda usar este enfoque muy a menudo para quitar un tipo de nodo de un clúster de producción. Se trata de un comando muy peligroso, ya que elimina el recurso del conjunto de escalado de máquinas virtuales que hay detrás del tipo de nodo. 

## <a name="durability-characteristics"></a>Características de durabilidad
Al usar Remove-AzServiceFabricNodeType, se da prioridad a la seguridad frente a la velocidad. El tipo de nodo debe tener un [nivel de durabilidad](./service-fabric-cluster-capacity.md#durability-characteristics-of-the-cluster) Silver o Gold, ya que:
- Bronze no ofrece ningún tipo de garantía sobre la información de estado de guardado.
- Las durabilidades Silver y Gold identifican cualquier cambio tiene lugar en el conjunto de escalado.
- La durabilidad Gold también permite controlar las actualizaciones de Azure que tienen lugar bajo el conjunto de escalado.

Service Fabric “organiza” los cambios subyacentes y las actualizaciones para que no se pierdan los datos. Sin embargo, al quitar un tipo de nodo con una durabilidad Bronze, puede que se pierda la información de estado. Si quita un tipo de nodo principal y la aplicación no tiene ningún estado, es aceptable el uso de Bronze. Al ejecutar cargas de trabajo con estado en producción, la configuración mínima tiene que ser Silver. De forma similar, para escenarios de producción, el tipo de nodo principal siempre tiene que ser Silver o Gold.

### <a name="more-about-bronze-durability"></a>Más información sobre la durabilidad Bronze

Al quitar un tipo de nodo Bronze, todos los nodos del tipo de nodo dejarán de estar activos de inmediato. Service Fabric no obtiene las actualizaciones del conjunto de escalado de los nodos Bronze y, por lo tanto, todas las máquinas virtuales dejan de estar activas de inmediato. Si tenía algún elemento con estado en esos nodos, los datos se perderán. Ahora, aunque no tengan estado, todos los nodos de Service Fabric participarán en el anillo, lo que podría hacer que se perdiera un grupo de nodos entero y desestabilizar al propio clúster.

## <a name="remove-a-node-type"></a>Eliminación de un tipo de nodo

1. Compruebe estos requisitos previos antes de iniciar el proceso.

    - El clúster está en buen estado.
    - Todavía habrá suficiente capacidad después de quitar el tipo de nodo; por ejemplo, el número de nodos para colocar el recuento de réplicas necesario.

2. Saque del tipo de nodo todos los servicios que tengan restricciones de selección de ubicación para usarlo.

    - Modifique la aplicación o el manifiesto de servicio para que ya no hagan referencia al tipo de nodo.
    - Implemente el cambio.

    A continuación, asegúrese de que:
    - Todos los servicios modificados anteriormente ya no se ejecutan en el nodo que pertenece al tipo de nodo.
    - Todos los servicios están en buen estado.

3. Desmarque el tipo de nodo como no principal (omítalo para los tipos de nodos no principales).

    - Busque la plantilla de Azure Resource Manager usada para la implementación.
    - Busque la sección relacionada con el tipo de nodo en la sección Service Fabric.
    - Cambie la propiedad isPrimary a false. ** No quite la sección relacionada con el tipo de nodo de esta tarea.
    - Implemente la plantilla de Azure Resource Manager modificada. ** En función de la configuración del clúster, este paso puede tardar unos minutos.
    
    A continuación, asegúrese de que:
    - La sección Service Fabric del portal indica que el clúster está listo.
    - El clúster está en buen estado.
    - Ninguno de los nodos que pertenecen al tipo de nodo están marcados como nodo de inicialización.

4. Deshabilite todos los nodos del tipo de nodo.

    Conéctese al clúster mediante PowerShell y, luego, ejecute el siguiente paso.
    
    ```powershell
    $nodeType = "" # specify the name of node type
    $nodes = Get-ServiceFabricNode
    
    foreach($node in $nodes)
    {
      if ($node.NodeType -eq $nodeType)
      {
        $node.NodeName
     
        Disable-ServiceFabricNode -Intent RemoveNode -NodeName $node.NodeName -Force
      }
    }
    ```

    - Para la durabilidad Bronze, espere a que todos los nodos lleguen al estado Deshabilitado.
    - En el caso de la durabilidad Silver o Gold, algunos nodos pasarán a Deshabilitado y el resto estará en estado Deshabilitando. Compruebe la pestaña de detalles de los nodos en estado Deshabilitando. Si todos los nodos están detenidos para garantizar el cuórum de las particiones del servicio de infraestructura, es seguro continuar.

5. Detenga los datos para el tipo de nodo.

    Conéctese al clúster mediante PowerShell y, luego, ejecute el siguiente paso.
    
    ```powershell
    foreach($node in $nodes)
    {
      if ($node.NodeType -eq $nodeType)
      {
        $node.NodeName
     
        Start-ServiceFabricNodeTransition -Stop -OperationId (New-Guid) -NodeInstanceId $node.NodeInstanceId -NodeName $node.NodeName -StopDurationInSeconds 10000
      }
    }
    ```
    
    Espere a que todos los nodos del tipo de nodo estén marcados como inactivos.

6. Desasignación de nodos en el conjunto de escalado de máquinas virtuales original
    
    Inicie sesión en la suscripción de Azure en la que se implementó el conjunto de escalado y elimine el conjunto de escalado de máquinas virtuales. 

    ```powershell
    $scaleSetName="myscaleset"
    $scaleSetResourceType="Microsoft.Compute/virtualMachineScaleSets"
    
    Remove-AzResource -ResourceName $scaleSetName -ResourceType $scaleSetResourceType -ResourceGroupName $resourceGroupName -Force
    ```

    
7. Quite los datos para el tipo de nodo.

    Conéctese al clúster mediante PowerShell y, luego, ejecute el siguiente paso.
    
    ```powershell
    foreach($node in $nodes)
    {
      if ($node.NodeType -eq $nodeType)
      {
        $node.NodeName
     
        Remove-ServiceFabricNodeState -NodeName $node.NodeName -Force
      }
    }
    ```

    Espere hasta que se quiten todos los nodos del clúster. Los nodos no se deben mostrar en SFX.

8. Quite el tipo de nodo de la sección Service Fabric.

    - Busque la plantilla de Azure Resource Manager usada para la implementación.
    - Busque la sección relacionada con el tipo de nodo en la sección Service Fabric.
    - Quite la sección correspondiente al tipo de nodo.
    - Solo en el caso de los clústeres de durabilidad Silver y superior, actualice el recurso de clúster en la plantilla y configure directivas de mantenimiento para omitir el estado de la aplicación fabric:/System; para ello, agregue `applicationDeltaHealthPolicies` en el recurso de clúster `properties` como se indica a continuación. La directiva siguiente debe omitir los errores existentes, pero no permitir nuevos errores de estado. 
 
 
     ```json
    "upgradeDescription":  
    { 
      "forceRestart": false, 
      "upgradeReplicaSetCheckTimeout": "10675199.02:48:05.4775807", 
      "healthCheckWaitDuration": "00:05:00", 
      "healthCheckStableDuration": "00:05:00", 
      "healthCheckRetryTimeout": "00:45:00", 
      "upgradeTimeout": "12:00:00", 
      "upgradeDomainTimeout": "02:00:00", 
      "healthPolicy": { 
        "maxPercentUnhealthyNodes": 100, 
        "maxPercentUnhealthyApplications": 100 
      }, 
      "deltaHealthPolicy":  
      { 
        "maxPercentDeltaUnhealthyNodes": 0, 
        "maxPercentUpgradeDomainDeltaUnhealthyNodes": 0, 
        "maxPercentDeltaUnhealthyApplications": 0, 
        "applicationDeltaHealthPolicies":  
        { 
            "fabric:/System":  
            { 
                "defaultServiceTypeDeltaHealthPolicy":  
                { 
                        "maxPercentDeltaUnhealthyServices": 0 
                } 
            } 
        } 
      } 
    },
    ```

    - Implemente la plantilla de Azure Resource Manager modificada. ** Este paso tardará un rato, habitualmente puede llegar a las dos horas. Esta actualización cambiará la configuración a InfrastructureService, por lo que es necesario reiniciar el nodo. En este caso `forceRestart` se pasa por alto. 
    El parámetro `upgradeReplicaSetCheckTimeout` especifica el tiempo máximo que Service Fabric espera a que una partición pase a un estado seguro, si aún no lo está. Una vez que se superan las comprobaciones de seguridad de todas las particiones de un nodo, Service Fabric continúa con la actualización en ese nodo.
    El valor del parámetro `upgradeTimeout` puede reducirse a 6 horas, pero debe usarse la seguridad máxima de 12 horas.

    A continuación, asegúrese de que:
    - El recurso de Service Fabric en el portal muestra Listo.

9. Quite todas las referencias a los recursos relacionados con el tipo de nodo de la plantilla de Resource Manager.

    - Busque la plantilla de Azure Resource Manager usada para la implementación.
    - Elimine de la plantilla el conjunto de escalado de máquinas virtuales y otros recursos relacionados con el tipo de nodo.
    - Implemente los cambios.

    A continuación:
    - Espere a que la implementación finalice.
    
10. Quite los recursos relacionados con el tipo de nodo que ya no se usen. Recursos Equilibrador de carga e IP pública de ejemplo. 

    - Para quitar estos recursos, puede usar el mismo comando de PowerShell que se usa en el paso 6, pero debe especificar el tipo de recurso específico y la versión de la API. 

> [!Note]
> Este paso es opcional si se reutilizan los recursos Equilibrador de carga e IP entre los tipos de nodo.

## <a name="next-steps"></a>Pasos siguientes
- Obtenga más información sobre las [características de durabilidad](./service-fabric-cluster-capacity.md#durability-characteristics-of-the-cluster) del clúster.
- Obtenga más información sobre los [tipos de nodo y los conjuntos de escalado de máquinas virtuales](service-fabric-cluster-nodetypes.md).
- Obtenga más información sobre el [escalado de clústeres de Service Fabric](service-fabric-cluster-scaling.md).
