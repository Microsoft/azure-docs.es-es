---
title: 'Cluster Resource Manager de Service Fabric: directivas de colocación'
description: Información general sobre las reglas y directivas de colocación adicionales para los servicios de Service Fabric
author: masnider
ms.topic: conceptual
ms.date: 08/18/2017
ms.author: masnider
ms.custom: devx-track-csharp
ms.openlocfilehash: e27c6661c34ab6d177feec11f8e9ec891987ab48
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89005758"
---
# <a name="placement-policies-for-service-fabric-services"></a>Directivas de colocación de servicios de Service Fabric
Las directivas de colocación son reglas adicionales que pueden usarse para controlar la colocación del servicio en algunos escenarios específicos menos comunes. Algunos ejemplos de esos escenarios son:

- El clúster de Service Fabric abarca distancias geográficas, como varios centros de datos locales o regiones de Azure
- El entorno abarca diversas áreas de control geopolítico o legal, o cualquier otro caso en el que haya límites de directiva que deba exigir.
- Existen aspectos sobre el rendimiento o la latencia de las comunicaciones que se deben tener en cuenta debido a grandes distancias o al uso de vínculos de red más lentos o menos confiables
- Hay que hacer lo posible por mantener colocadas determinadas cargas de trabajo, bien con otras cargas de trabajo o cerca de los clientes

La mayoría de estos requisitos guardan relación con el diseño físico del clúster, representado como los dominios de error del clúster. 

Las directivas de colocación avanzadas que ayudan a abordar estos escenarios son:

1. Dominios no válidos
2. Dominios requeridos
3. Dominios de preferencia
4. No permitir el empaquetado de réplicas

La mayoría de los controles siguientes pueden configurarse mediante propiedades de nodo y restricciones de colocación, pero otros son más complicados. Para simplificar las cosas, Cluster Resource Manager de Service Fabric proporciona estas directivas de colocación adicionales. Las directivas de colocación se configuran por instancia de servicio con nombre previo. También se pueden actualizar dinámicamente.

## <a name="specifying-invalid-domains"></a>Especificación de dominios no válidos
La directiva de colocación **InvalidDomain** le permite especificar que un determinado dominio de error no es válido para un servicio específico. Resulta útil para garantizar que un servicio específico no se ejecute nunca en un área determinada, por ejemplo por motivos geopolíticos o relacionados con la directiva corporativa. Pueden especificarse varios dominios no válidos mediante distintas directivas.

<center>

![Ejemplo de dominio no válido][Image1]
</center>

Código:

```csharp
ServicePlacementInvalidDomainPolicyDescription invalidDomain = new ServicePlacementInvalidDomainPolicyDescription();
invalidDomain.DomainName = "fd:/DCEast"; //regulations prohibit this workload here
serviceDescription.PlacementPolicies.Add(invalidDomain);
```

PowerShell:

```posh
New-ServiceFabricService -ApplicationName $applicationName -ServiceName $serviceName -ServiceTypeName $serviceTypeName –Stateful -MinReplicaSetSize 3 -TargetReplicaSetSize 3 -PartitionSchemeSingleton -PlacementPolicy @("InvalidDomain,fd:/DCEast”)
```
## <a name="specifying-required-domains"></a>Especificación de dominios requeridos
La directiva de colocación del dominio requerido exige que el servicio esté presente solo en el dominio especificado. Pueden especificarse varios dominios requeridos mediante directivas diferentes.

<center>

![Ejemplo de dominio requerido][Image2]
</center>

Código:

```csharp
ServicePlacementRequiredDomainPolicyDescription requiredDomain = new ServicePlacementRequiredDomainPolicyDescription();
requiredDomain.DomainName = "fd:/DC01/RK03/BL2";
serviceDescription.PlacementPolicies.Add(requiredDomain);
```

PowerShell:

```posh
New-ServiceFabricService -ApplicationName $applicationName -ServiceName $serviceName -ServiceTypeName $serviceTypeName –Stateful -MinReplicaSetSize 3 -TargetReplicaSetSize 3 -PartitionSchemeSingleton -PlacementPolicy @("RequiredDomain,fd:/DC01/RK03/BL2")
```

## <a name="specifying-a-preferred-domain-for-the-primary-replicas-of-a-stateful-service"></a>Especificación de un dominio preferido para las réplicas principales de un servicio sin estado
El dominio principal preferido especifica el dominio de error en el que colocar la réplica principal. El dominio principal acaba en este dominio cuando todo está correcto. Si el dominio o la réplica principal dan error o se apaga, la réplica principal se mueve a alguna otra ubicación, preferiblemente en el mismo dominio. Si esta ubicación nueva no está en el dominio preferido, Cluster Resource Manager la devolverá al dominio preferido lo antes posible. Evidentemente, esta configuración solo tiene sentido para los servicios con estado. Esta directiva resulta útil principalmente en clústeres que están distribuidos entre regiones de Azure o entre varios centros de datos, pero tienen servicios que prefieren la colocación en una determinada ubicación. Mantener las réplicas principales cerca de sus usuarios u otros servicios ayuda a reducir la latencia, sobre todo en las lecturas, que controlan de forma predeterminada las réplicas principales.

<center>

![Dominios principales preferidos y conmutación por error][Image3]
</center>

```csharp
ServicePlacementPreferPrimaryDomainPolicyDescription primaryDomain = new ServicePlacementPreferPrimaryDomainPolicyDescription();
primaryDomain.DomainName = "fd:/EastUS/";
serviceDescription.PlacementPolicies.Add(primaryDomain);
```

PowerShell:

```posh
New-ServiceFabricService -ApplicationName $applicationName -ServiceName $serviceName -ServiceTypeName $serviceTypeName –Stateful -MinReplicaSetSize 3 -TargetReplicaSetSize 3 -PartitionSchemeSingleton -PlacementPolicy @("PreferredPrimaryDomain,fd:/EastUS")
```

## <a name="requiring-replica-distribution-and-disallowing-packing"></a>Necesidad de la distribución de réplicas y prohibición del empaquetado
Las réplicas se distribuyen _normalmente_ entre dominios de error y de actualización cuando el clúster tiene un estado correcto. Sin embargo, hay casos donde más de una réplica para una partición determinada puede acabar empaquetada temporalmente en un solo dominio. Por ejemplo, supongamos que el clúster tiene nueve nodos en tres dominios de error: fd:/0, fd:/1 y fd:/2. Supongamos también que el servicio tiene tres réplicas. Supongamos que los nodos que se usaban para esas réplicas en fd:/1 y fd:/2 quedaron inactivos. Lo habitual es que Cluster Resource Manager prefiera otros nodos en esos mismos dominios de error. En este caso, supongamos que, debido a problemas de capacidad, ninguno de los demás nodos de esos dominios eran válidos. Si Cluster Resource Manager crea reemplazos para esas réplicas, debería tener que escoger los nodos de fd:/0. Sin embargo, con _esa acción_ se crea una situación en la que se infringe la restricción de dominio de error. El empaquetado de las réplicas aumenta la oportunidad de que el conjunto entero de réplicas quede inactivo o se pierda. 

> [!NOTE]
> Para más información sobre las restricciones y las prioridades de restricción en general, consulte [este tema](service-fabric-cluster-resource-manager-management-integration.md#constraint-priorities).
>

Si alguna vez vio un mensaje de estado como "`The Load Balancer has detected a Constraint Violation for this Replica:fabric:/<some service name> Secondary Partition <some partition ID> is violating the Constraint: FaultDomain`", es porque se topó con esta condición o algo parecido. Normalmente solo una o dos réplicas se empaquetan juntas temporalmente. Siempre que haya menos de un cuórum de réplicas en un dominio dado, está seguro. Aunque el empaquetado es raro, puede suceder y, usualmente, estas situaciones son transitorias porque los nodos vuelven a estar activos. Si los nodos permanecen inactivos y Cluster Resource Manager debe crear reemplazos, habitualmente hay otros nodos disponibles en los dominios de error ideales.

Algunas cargas de trabajo preferirían tener siempre el número objetivo de réplicas, incluso si se empaquetan en menos dominios. Estas cargas de trabajo apuesta contra el número total de errores permanentes del dominio simultáneos y normalmente pueden recuperar el estado local. Otras cargas de trabajo preferirían asumir el tiempo de inactividad antes que arriesgar la exactitud o la pérdida de los datos. La mayoría de las cargas de trabajo de producción se ejecutan con más de tres réplicas, más de tres dominios de error y muchos nodos válidos por dominio de error. Por este motivo, el comportamiento predeterminado es permitir siempre el empaquetado de dominios. El comportamiento predeterminado permite el equilibrio y la conmutación por error normales para controlar estos casos extremos, incluso si eso significa el empaquetado temporal de dominios.

Si quiere deshabilitar dicho empaquetado para una carga de trabajo dada, puede especificar la directiva `RequireDomainDistribution` en el servicio. Cuando se establece esta directiva, Cluster Resource Manager garantiza que nunca se ejecutarán dos réplicas de la misma partición en el mismo dominio de error o de actualización.

Código:

```csharp
ServicePlacementRequireDomainDistributionPolicyDescription distributeDomain = new ServicePlacementRequireDomainDistributionPolicyDescription();
serviceDescription.PlacementPolicies.Add(distributeDomain);
```

PowerShell:

```posh
New-ServiceFabricService -ApplicationName $applicationName -ServiceName $serviceName -ServiceTypeName $serviceTypeName –Stateful -MinReplicaSetSize 3 -TargetReplicaSetSize 3 -PartitionSchemeSingleton -PlacementPolicy @("RequiredDomainDistribution")
```

Se preguntará si sería posible usar estas configuraciones para los servicios de un clúster que no esté distribuido geográficamente. Aunque podría, no hay razones de peso para hacerlo. Las configuraciones de dominio no válidas, obligatorias y preferidas se deben evitar a menos que así lo exijan los escenarios. No tiene sentido intentar forzar que una carga de trabajo determinada se ejecute en un solo bastidor o dar preferencia a algún segmento del clúster local sobre otro. Se deben distribuir diferentes configuraciones de hardware entre dominios de error y se deben controlar mediante propiedades de nodo y restricciones de colocación normales.

## <a name="next-steps"></a>Pasos siguientes
- Para más información sobre la configuración de servicios, vaya a [este vínculo](service-fabric-cluster-resource-manager-configure-services.md).

[Image1]:./media/service-fabric-cluster-resource-manager-advanced-placement-rules-placement-policies/cluster-invalid-placement-domain.png
[Image2]:./media/service-fabric-cluster-resource-manager-advanced-placement-rules-placement-policies/cluster-required-placement-domain.png
[Image3]:./media/service-fabric-cluster-resource-manager-advanced-placement-rules-placement-policies/cluster-preferred-primary-domain.png
