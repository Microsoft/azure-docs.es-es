---
title: Contenedores y servicios de escalado automático de Azure Service Fabric
description: Azure Service Fabric permite establecer directivas de escalado automático para servicios y contenedores.
author: radicmilos
ms.topic: conceptual
ms.date: 04/17/2018
ms.author: miradic
ms.custom: devx-track-csharp
ms.openlocfilehash: f57e5079543a3112b4fa59f26ba0ae27c24b79a2
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89005520"
---
# <a name="introduction-to-auto-scaling"></a>Introducción al escalado automático
El escalado automático es una funcionalidad adicional de Service Fabric para escalar servicios de manera dinámica en función de la carga que notifican los servicios o del uso de recursos. El escalado automático proporciona una gran elasticidad y permite el aprovisionamiento de instancias o particiones adicionales del servicio a petición. Todo el proceso de escalado es automático y transparente. Además, una vez configuradas las directivas en un servicio, no es necesario realizar ninguna operación de escalado manual a nivel de servicio. El escalado automático se puede activar al crear el servicio o al actualizarlo en cualquier momento.

El escalado automático es útil en escenarios en los que la carga de un servicio concreto varía con el tiempo. Por ejemplo, un servicio (como, una puerta de enlace) puede escalar en función de la cantidad de recursos necesarios para gestionar las solicitudes entrantes. Echemos un vistazo a un ejemplo del aspecto que tendrían estas reglas de escalado:
* Si todas las instancias de una puerta de enlace usan más de dos núcleos de media, agregue una instancia más para escalar el servicio de puerta de enlace horizontalmente. Hágalo cada hora, pero no tenga nunca más de siete instancias en total.
* Si todas las instancias de la puerta de enlace usan menos de 0,5 núcleos de media, quite una instancia para reducir el servicio horizontalmente. Hágalo cada hora, pero no tenga nunca menos de tres instancias en total.

Se admite el escalado automático para contenedores y servicios normales de Service Fabric. Para poder usar el escalado automático, debe ejecutar la versión 6.2 o posterior del entorno de ejecución de Service Fabric. 

En lo que queda de este artículo, se describen las directivas de escalado, cómo habilitar o deshabilitar el escalado automático y se proporcionan ejemplos sobre cómo usar esta característica.

## <a name="describing-auto-scaling"></a>Descripción del escalado automático
Las directivas de escalado automático pueden definirse para cada servicio de un clúster de Service Fabric. Cada directiva de escalado consta de dos partes:
* El **desencadenador de escalado** describe cuándo se escalará el servicio. Las condiciones que se definen en el desencadenador se comprueban periódicamente para determinar si se debe escalar un servicio.

* El **mecanismo de escalado** describe cómo se escalará cuando se desencadene. El mecanismo solo se aplica cuando se cumplen las condiciones del desencadenador.

Todos los desencadenadores que son compatibles actualmente funcionan con [métricas de carga lógica](service-fabric-cluster-resource-manager-metrics.md) o con métricas físicas, como uso de memoria o CPU. En cualquier caso, Service Fabric supervisará la carga notificada para la métrica y evaluará el desencadenador periódicamente para determinar si es necesario el escalado.

Hay dos mecanismos compatibles actualmente con el escalado automático. El primero está pensado para servicios sin estado o para contenedores en los que el escalado automático se realiza agregando o quitando [instancias](service-fabric-concepts-replica-lifecycle.md). Para servicios con y sin estado, también se puede realizar el escalado automático agregando o quitando [particiones](service-fabric-concepts-partitioning.md) con nombre del servicio.

> [!NOTE]
> Actualmente se admite una única directiva de escalado por servicio y solo un desencadenador de escalado por directiva de escalado.

## <a name="average-partition-load-trigger-with-instance-based-scaling"></a>Desencadenador de carga de partición media con escalado basado en instancias
El primer tipo de desencadenador se basa en la carga de instancias de una partición de servicio sin estado. En primer lugar, se suavizan las cargas de métricas para obtener la carga de cada instancia de una partición y, a continuación, se realiza la media de estos valores entre todas las instancias de la partición. Hay tres factores que determinan cuándo se escalará el servicio:

* El _umbral inferior de carga_ es un valor que determina si el servicio se **reducirá horizontalmente**. Si la carga media de todas las instancias de las particiones es inferior a este valor, el servicio se reducirá horizontalmente.
* El _umbral superior de carga_ es un valor que determina si el servicio se **escalará horizontalmente**. Si la carga media de todas las instancias de la partición es inferior a este valor, el servicio se escalará horizontalmente.
* El _intervalo de escalado_ determina la frecuencia con la que se comprobará el desencadenador. Una vez comprobado el desencadenador, se aplicará el mecanismo si es necesario escalar. Si no es necesario escalar, no se realizará ninguna acción. En ambos casos, no se volverá a comprobar el desencadenador antes de que vuelva a expirar el intervalo de escalado.

Este desencadenador solo se puede usar con servicios sin estado (contenedores sin estado o servicios de Service Fabric). Si un servicio tiene varias particiones, el desencadenador se evalúa para cada partición por separado y se aplicará el mecanismo especificado a cada partición de manera independiente. Por lo tanto, en este caso, es posible que algunas particiones del servicio se escalen horizontalmente; otras se reduzcan horizontalmente y otras no se escalen a la vez, en función de su carga.

El único mecanismo que se puede usar con este desencadenador es PartitionInstanceCountScaleMechanism. Hay tres factores que determinan cómo se aplica este mecanismo:
* El _incremento de escala_ determina cuántas instancias se agregarán o se quitarán cuando se desencadene el mecanismo.
* El _recuento máximo de instancias_ define el límite superior para el escalado. Si el número de instancias de la partición alcanza este límite, el servicio no se escalará horizontalmente, independientemente de la carga. Es posible omitir este límite si se especifica un valor de -1. En ese caso, el servicio se escalará horizontalmente tanto como sea posible (el límite es el número de nodos disponibles en el clúster).
* El _recuento mínimo de instancias_ define el límite inferior para el escalado. Si el número de instancias de la partición alcanza este límite, el servicio no se reducirá horizontalmente, independientemente de la carga.

## <a name="setting-auto-scaling-policy"></a>Configuración de la directiva de escalado automático

### <a name="using-application-manifest"></a>Uso del manifiesto de aplicación
``` xml
<LoadMetrics>
<LoadMetric Name="MetricB" Weight="High"/>
</LoadMetrics>
<ServiceScalingPolicies>
<ScalingPolicy>
    <AveragePartitionLoadScalingTrigger MetricName="MetricB" LowerLoadThreshold="1" UpperLoadThreshold="2" ScaleIntervalInSeconds="100"/>
    <InstanceCountScalingMechanism MinInstanceCount="3" MaxInstanceCount="4" ScaleIncrement="1"/>
</ScalingPolicy>
</ServiceScalingPolicies>
```
### <a name="using-c-apis"></a>Uso de las API de C#
```csharp
FabricClient fabricClient = new FabricClient();
StatelessServiceDescription serviceDescription = new StatelessServiceDescription();
//set up the rest of the ServiceDescription
AveragePartitionLoadScalingTrigger trigger = new AveragePartitionLoadScalingTrigger();
PartitionInstanceCountScaleMechanism mechanism = new PartitionInstanceCountScaleMechanism();
mechanism.MaxInstanceCount = 3;
mechanism.MinInstanceCount = 1;
mechanism.ScaleIncrement = 1;
trigger.MetricName = "servicefabric:/_CpuCores";
trigger.ScaleInterval = TimeSpan.FromMinutes(20);
trigger.LowerLoadThreshold = 1.0;
trigger.UpperLoadThreshold = 2.0;
ScalingPolicyDescription policy = new ScalingPolicyDescription(mechanism, trigger);
serviceDescription.ScalingPolicies.Add(policy);
//as we are using scaling on a resource this must be exclusive service
//also resource monitor service needs to be enabled
serviceDescription.ServicePackageActivationMode = ServicePackageActivationMode.ExclusiveProcess
await fabricClient.ServiceManager.CreateServiceAsync(serviceDescription);
```
### <a name="using-powershell"></a>Uso de PowerShell
```posh
$mechanism = New-Object -TypeName System.Fabric.Description.PartitionInstanceCountScaleMechanism
$mechanism.MinInstanceCount = 1
$mechanism.MaxInstanceCount = 6
$mechanism.ScaleIncrement = 2
$trigger = New-Object -TypeName System.Fabric.Description.AveragePartitionLoadScalingTrigger
$trigger.MetricName = "servicefabric:/_CpuCores"
$trigger.LowerLoadThreshold = 0.3
$trigger.UpperLoadThreshold = 0.8
$trigger.ScaleInterval = New-TimeSpan -Minutes 10
$scalingpolicy = New-Object -TypeName System.Fabric.Description.ScalingPolicyDescription
$scalingpolicy.ScalingMechanism = $mechanism
$scalingpolicy.ScalingTrigger = $trigger
$scalingpolicies = New-Object 'System.Collections.Generic.List[System.Fabric.Description.ScalingPolicyDescription]'
$scalingpolicies.Add($scalingpolicy)
#as we are using scaling on a resource this must be exclusive service
#also resource monitor service needs to be enabled
Update-ServiceFabricService -Stateless -ServiceName "fabric:/AppName/ServiceName" -ScalingPolicies $scalingpolicies
```

## <a name="average-service-load-trigger-with-partition-based-scaling"></a>Desencadenador de carga de servicio media con escalado basado en particiones
El segundo desencadenador se basa en la carga de todas las particiones de un servicio. En primer lugar, se suavizan las cargas de métricas para obtener la carga de cada réplica o instancia de una partición. Para los servicios con estado, la carga de la partición es la carga de la réplica principal, mientras que para los servicios sin estado, la carga de la partición es la carga media de todas las instancias de la partición. Se realiza la media de estos valores entre todas las particiones del servicio y este valor se usa para desencadenar el escalado automático. Igual que en el mecanismo anterior, hay tres factores que determinan cuándo se escalará el servicio:

* El _umbral inferior de carga_ es un valor que determina si el servicio se **reducirá horizontalmente**. Si la carga media de todas las particiones del servicio es inferior a este valor, el servicio se reducirá horizontalmente.
* El _umbral superior de carga_ es un valor que determina si el servicio se **escalará horizontalmente**. Si la carga media de todas las particiones del servicio es superior a este valor, el servicio se escalará horizontalmente.
* El _intervalo de escalado_ determina la frecuencia con la que se comprobará el desencadenador. Una vez comprobado el desencadenador, se aplicará el mecanismo si es necesario escalar. Si no es necesario escalar, no se realizará ninguna acción. En ambos casos, no se volverá a comprobar el desencadenador antes de que vuelva a expirar el intervalo de escalado.

Este desencadenador se puede usar con servicios con y sin estado. El único mecanismo que se puede usar con este desencadenador es AddRemoveIncrementalNamedPartitionScalingMechanism. Cuando el servicio se escala horizontalmente, se agrega una partición nueva, mientras que, cuando el servicio se reduce horizontalmente, se quita una de las particiones existentes. Hay restricciones que se comprobarán al crear o actualizar el servicio y no se podrá crear o actualizar el servicio si no se cumplen estas condiciones:
* El esquema de partición con nombre se debe usar para el servicio.
* Los nombres de la partición deben ser números enteros consecutivos, como "0", "1"...
* El nombre de la primera partición debe ser "0".

Por ejemplo, si un servicio se crea inicialmente con tres particiones, la única posibilidad válida para los nombres de la partición es "0", "1" y "2".

La operación real de escalado automático que se realiza también respetará este esquema de nomenclatura:
* Si las particiones actuales del servicio se denominan "0", "1" y "2", la partición que se agregará para el escalado horizontal se denominará "3".
* Si las particiones actuales del servicio se denominan "0", "1" y "2", la partición que se quitará para la reducción horizontal es la partición "2".

Igual que el mecanismo que usa el escalado agregando o quitando instancias, hay tres parámetros que determinan cómo se aplica este mecanismo:
* El _incremento de escala_ determina cuántas particiones se agregarán o se quitarán cuando se desencadene el mecanismo.
* El _recuento máximo de particiones_ define el límite superior para el escalado. Si el número de particiones del servicio alcanza este límite, el servicio no se escalará horizontalmente, independientemente de la carga. Es posible omitir este límite si se especifica un valor de -1. En ese caso, el servicio se escalará horizontalmente tanto como sea posible (el límite es la capacidad real del clúster).
* El _recuento mínimo de instancias_ define el límite inferior para el escalado. Si el número de particiones del servicio alcanza este límite, el servicio no se reducirá horizontalmente, independientemente de la carga.

> [!WARNING] 
> Cuando se usa AddRemoveIncrementalNamedPartitionScalingMechanism con servicios con estado, Service Fabric agrega o quita las particiones **sin ninguna notificación o advertencia**. No se volverán a particionar los datos cuando se desencadene el mecanismo de escalado. En caso de una operación de escalado horizontal, las nuevas particiones estarán vacías y en el caso de una operación de reducción horizontal, la **partición se eliminará junto con todos los datos que contiene**.

## <a name="setting-auto-scaling-policy"></a>Configuración de la directiva de escalado automático

### <a name="using-application-manifest"></a>Uso del manifiesto de aplicación
``` xml
<NamedPartition>
    <Partition Name="0" />
</NamedPartition>
<ServiceScalingPolicies>
    <ScalingPolicy>
        <AverageServiceLoadScalingTrigger MetricName="servicefabric:/_MemoryInMB" LowerLoadThreshold="300" UpperLoadThreshold="500" ScaleIntervalInSeconds="600"/>
        <AddRemoveIncrementalNamedPartitionScalingMechanism MinPartitionCount="1" MaxPartitionCount="3" ScaleIncrement="1"/>
    </ScalingPolicy>
</ServiceScalingPolicies>
```
### <a name="using-c-apis"></a>Uso de las API de C#
```csharp
FabricClient fabricClient = new FabricClient();
StatefulServiceUpdateDescription serviceUpdate = new StatefulServiceUpdateDescription();
AveragePartitionLoadScalingTrigger trigger = new AverageServiceLoadScalingTrigger();
PartitionInstanceCountScaleMechanism mechanism = new AddRemoveIncrementalNamedPartitionScalingMechanism();
mechanism.MaxPartitionCount = 4;
mechanism.MinPartitionCount = 1;
mechanism.ScaleIncrement = 1;
//expecting that the service already has metric NumberOfConnections
trigger.MetricName = "NumberOfConnections";
trigger.ScaleInterval = TimeSpan.FromMinutes(15);
trigger.LowerLoadThreshold = 10000;
trigger.UpperLoadThreshold = 20000;
ScalingPolicyDescription policy = new ScalingPolicyDescription(mechanism, trigger);
serviceUpdate.ScalingPolicies = new List<ScalingPolicyDescription>;
serviceUpdate.ScalingPolicies.Add(policy);
await fabricClient.ServiceManager.UpdateServiceAsync(new Uri("fabric:/AppName/ServiceName"), serviceUpdate);
```
### <a name="using-powershell"></a>Uso de PowerShell
```posh
$mechanism = New-Object -TypeName System.Fabric.Description.AddRemoveIncrementalNamedPartitionScalingMechanism
$mechanism.MinPartitionCount = 1
$mechanism.MaxPartitionCount = 3
$mechanism.ScaleIncrement = 2
$trigger = New-Object -TypeName System.Fabric.Description.AverageServiceLoadScalingTrigger
$trigger.MetricName = "servicefabric:/_MemoryInMB"
$trigger.LowerLoadThreshold = 5000
$trigger.UpperLoadThreshold = 10000
$trigger.ScaleInterval = New-TimeSpan -Minutes 25
$scalingpolicy = New-Object -TypeName System.Fabric.Description.ScalingPolicyDescription
$scalingpolicy.ScalingMechanism = $mechanism
$scalingpolicy.ScalingTrigger = $trigger
$scalingpolicies = New-Object 'System.Collections.Generic.List[System.Fabric.Description.ScalingPolicyDescription]'
$scalingpolicies.Add($scalingpolicy)
#as we are using scaling on a resource this must be exclusive service
#also resource monitor service needs to be enabled
New-ServiceFabricService -ApplicationName $applicationName -ServiceName $serviceName -ServiceTypeName $serviceTypeName –Stateful -TargetReplicaSetSize 3 -MinReplicaSetSize 2 -HasPersistedState true -PartitionNames @("0","1") -ServicePackageActivationMode ExclusiveProcess -ScalingPolicies $scalingpolicies
```

## <a name="auto-scaling-based-on-resources"></a>Escalado automático basado en recursos

Para permitir que el servicio de Monitor de recursos escale en función de recursos reales:

``` json
"fabricSettings": [
...      
],
"addonFeatures": [
    "ResourceMonitorService"
],
```
Hay dos métricas que representan recursos físicos reales. Una es servicefabric:/_CpuCores, que representa el uso real de CPU (0,5 representa medio núcleo), y la otra es servicefabric:/_MemoryInMB, que representa el uso de memoria en MB.
ResourceMonitorService es responsable de realizar un seguimiento del uso de memoria y de CPU de los servicios del usuario. Este servicio aplicará una media móvil ponderada para responder a posibles picos de corta duración. Se admite la supervisión de recursos para las aplicaciones en contenedores y fuera de contenedores en Windows y para las aplicaciones en contenedores en Linux. Solo se habilita el escalado automático en recursos para servicios activados en el [modelo de proceso exclusivo](service-fabric-hosting-model.md#exclusive-process-model).

## <a name="next-steps"></a>Pasos siguientes
Obtenga más información sobre la [escalabilidad de aplicaciones](service-fabric-concepts-scalability.md).
