---
title: Visualización del mantenimiento agregado de entidades de Azure Service Fabric
description: Se describe cómo consultar, ver y evaluar el mantenimiento agregado de entidades de Azure Service Fabric, a través de consultas de mantenimiento y consultas generales.
author: georgewallace
ms.topic: conceptual
ms.date: 2/28/2018
ms.author: gwallace
ms.custom: devx-track-csharp
ms.openlocfilehash: eeb2fd94e6b98bc9d89be22501406db9a8ba7773
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89013170"
---
# <a name="view-service-fabric-health-reports"></a>Vista de los informes de estado de Service Fabric
Azure Service Fabric presenta un [modelo de mantenimiento](service-fabric-health-introduction.md) con entidades de estado en las que componentes y guardianes del sistema pueden notificar las condiciones locales que están supervisando. El [almacén de estado](service-fabric-health-introduction.md#health-store) agrega todos los datos de mantenimiento para determinar si las entidades son correctas.

El clúster se rellena automáticamente con informes de estado enviados por los componentes del sistema. Lea más en [Uso de informes de mantenimiento del sistema para solucionar problemas](service-fabric-understand-and-troubleshoot-with-system-health-reports.md).

Service Fabric proporciona varias maneras de obtener el mantenimiento agregado de las entidades:

* [Explorador de Service Fabric](service-fabric-visualizing-your-cluster.md) y otras herramientas de visualización
* Consultas de mantenimiento (a través de PowerShell, API o REST)
* Consultas generales que devuelven una lista de entidades con el estado como una de las propiedades (a través de PowerShell, API o REST)

Para demostrar estas opciones, vamos a usar un clúster local con los nodos y la [aplicación fabric:/WordCount](https://github.com/Azure-Samples/service-fabric-wordcount/raw/master/WordCountV1.sfpkg). La aplicación **fabric:/WordCount** contiene dos servicios predeterminados, uno con estado de tipo `WordCountServiceType` y otro sin estado de tipo `WordCountWebServiceType`. He cambiado el archivo `ApplicationManifest.xml` para requerir siete réplicas de destino para el servicio con estado y una partición. Dado que hay solo cinco nodos en el clúster, los componentes del sistema notifican una advertencia en la partición del servicio porque está por debajo del número de destino.

```xml
<Service Name="WordCountService">
  <StatefulService ServiceTypeName="WordCountServiceType" TargetReplicaSetSize="7" MinReplicaSetSize="2">
    <UniformInt64Partition PartitionCount="[WordCountService_PartitionCount]" LowKey="1" HighKey="26" />
  </StatefulService>
</Service>
```

## <a name="health-in-service-fabric-explorer"></a>Mantenimiento del Explorador de Service Fabric
El Explorador de Service Fabric proporciona una vista visual del clúster. En la imagen siguiente, puede ver que:

* La aplicación **fabric:/WordCount** está en rojo (en error) porque tiene un evento de error notificado por **MyWatchdog** para la propiedad **Availability**.
* Uno de sus servicios, **fabric:/WordCount/WordCountService** está en amarillo (en advertencia). El servicio está configurado con siete réplicas y el clúster tiene cinco nodos, por lo que no se pueden colocar dos réplicas. Aunque no se muestra aquí, la partición de servicio está en amarillo debido al informe del sistema de `System.FM` que indica que `Partition is below target replica or instance count` (la partición está por debajo del número de instancias o réplicas de destino). La partición en amarillo desencadena el servicio en amarillo.
* El clúster está en rojo debido a la aplicación en rojo.

La evaluación usa las directivas predeterminadas del manifiesto de clúster y el manifiesto de aplicación. Son directivas estrictas y no toleran ningún error.

Vista del clúster con el Explorador de Service Fabric.

![Vista del clúster con el Explorador de Service Fabric.][1]

[1]: ./media/service-fabric-view-entities-aggregated-health/servicefabric-explorer-cluster-health.png


> [!NOTE]
> Obtenga más información sobre el [Explorador de Service Fabric](service-fabric-visualizing-your-cluster.md).
>
>

## <a name="health-queries"></a>Consultas de mantenimiento
Service Fabric expone las consultas de mantenimiento para cada uno de los [tipos de entidad](service-fabric-health-introduction.md#health-entities-and-hierarchy)admitidos. Se puede acceder a ellas mediante la API, con los métodos de [FabricClient.HealthManager](/dotnet/api/system.fabric.fabricclient.healthmanager?view=azure-dotnet), los cmdlets de PowerShell y REST. Estas consultas devuelven información completa de mantenimiento sobre la entidad: el estado de mantenimiento agregado, los eventos de mantenimiento de la entidad, los estados de mantenimiento de los elementos secundarios (si procede), las evaluaciones de estado incorrecto y las estadísticas de mantenimiento de los elementos secundarios (cuando corresponde).

> [!NOTE]
> Se devuelve una entidad de mantenimiento cuando se rellena completamente en el Almacén de estado. La entidad debe estar activa (no eliminada) y tener un informe de sistema. Sus entidades primarias en la cadena de jerarquía deben tener también informes del sistema. Si no se cumple alguna de estas condiciones, las consultas de mantenimiento devuelven una excepción [FabricException](/dotnet/api/system.fabric.fabricexception) con el código [FabricErrorCode](/dotnet/api/system.fabric.fabricerrorcode) `FabricHealthEntityNotFound` que muestra por qué no se devuelve la entidad.
>
>

Las consultas de mantenimiento requieren pasar el identificador de entidad, que depende del tipo de entidad. Las consultas aceptan parámetros de directivas de mantenimiento opcionales. Si no se especifican directivas de mantenimiento, se usan para la evaluación las [directivas de mantenimiento](service-fabric-health-introduction.md#health-policies) del manifiesto de aplicación o de clúster. Si los manifiestos no contienen una definición de las directivas de mantenimiento, para la evaluación se usan las predeterminadas. Las directivas de mantenimiento predeterminadas no toleran errores. También aceptan filtros para devolver solo los elementos secundarios o eventos parciales, los que respetan los filtros especificados. Otro filtro permite excluir las estadísticas de los elementos secundarios.

> [!NOTE]
> Los filtros de salida se aplican en el servidor, por lo que se reduce el tamaño de la respuesta del mensaje. Recomendamos usar los filtros de salida para limitar los datos devueltos en lugar de aplicar filtros en el cliente.
>
>

Contiene el mantenimiento de la entidad:

* El estado de mantenimiento agregado de la entidad. El Almacén de estado lo calcula en función de los informes de mantenimiento de la entidad, los estados de mantenimiento de los elementos secundarios (si procede) y las directivas de mantenimiento. Lea más sobre la [evaluación del mantenimiento de entidades](service-fabric-health-introduction.md#health-evaluation).  
* Eventos de mantenimiento de la entidad.
* La colección de los estados de mantenimiento de todos los elementos secundarios de las entidades que pueden tener elementos secundarios. Los estados de mantenimiento contienen identificadores de entidad y el estado de mantenimiento agregado. Para obtener el mantenimiento completo de un elemento secundario, llame al mantenimiento de consultas del tipo de entidad secundaria y pase el identificador de elementos secundarios.
* Si la entidad no es correcta, las evaluaciones de mantenimiento incorrecto apuntan al informe que desencadenó el estado de la entidad. Las evaluaciones son recursivas, contienen las evaluaciones de mantenimiento de los elementos secundarios que desencadenaron el estado de mantenimiento actual. Por ejemplo, un guardián notificó un error en una réplica. El estado de la aplicación muestra una evaluación de estado incorrecto debido a un servicio con un estado incorrecto; este estado del servicio se debe a que una partición tiene un error; el estado de la partición es incorrecto debido a que una réplica tiene otro error; el estado de la réplica es incorrecto debido al informe de estado de error del guardián.
* Las estadísticas de estado de todos los tipos de elementos secundarios de las entidades que los tienen. Por ejemplo, el estado del clúster muestra el número total de aplicaciones, servicios, particiones, réplicas y entidades implementadas en el clúster. El estado del servicio muestra el número total de particiones y réplicas en el servicio especificado.

## <a name="get-cluster-health"></a>Obtención del mantenimiento de clúster
Devuelve el mantenimiento de la entidad del clúster y contiene los estados de mantenimiento de aplicaciones y nodos (elementos secundarios del clúster). Entrada:

* [Opcional] La directiva de mantenimiento del clúster utilizada para evaluar los nodos y los eventos del clúster.
* [Opcional] La asignación de directivas de mantenimiento de aplicaciones, con las directivas de mantenimiento usadas para invalidar las directivas de manifiesto de aplicación.
* [Opcional] Filtros para eventos, nodos y aplicaciones que especifican las entradas que son de interés y se deben devolver en el resultado (por ejemplo, únicamente los errores o advertencias y errores). Todos los eventos, nodos y aplicaciones se utilizan para evaluar el mantenimiento agregado de la entidad, independientemente del filtro.
* [Opcional] Filtro para excluir las estadísticas de estado.
* [Opcional] Filtro para incluir las estadísticas de estado fabric:/System. Solo se pude aplicar cuando no se excluyen las estadísticas de estado. De forma predeterminada, las estadísticas de estado incluyen solo las estadísticas de las aplicaciones de usuario y no la aplicación del sistema.

### <a name="api"></a>API
Para obtener el mantenimiento del clúster, cree una instancia de `FabricClient` y llame al método [GetClusterHealthAsync](/dotnet/api/system.fabric.fabricclient.healthclient.getclusterhealthasync) en su instancia de **HealthManager**.

La siguiente llamada obtiene el mantenimiento del clúster:

```csharp
ClusterHealth clusterHealth = await fabricClient.HealthManager.GetClusterHealthAsync();
```

El código siguiente obtiene el mantenimiento del clúster mediante una directiva de mantenimiento personalizada del clúster y filtros para nodos y aplicaciones. Especifica que las estadísticas de estado incluyen las de tipo fabric:/System. Crea la clase [ClusterHealthQueryDescription](/dotnet/api/system.fabric.description.clusterhealthquerydescription), que contiene toda la información de entrada.

```csharp
var policy = new ClusterHealthPolicy()
{
    MaxPercentUnhealthyNodes = 20
};
var nodesFilter = new NodeHealthStatesFilter()
{
    HealthStateFilterValue = HealthStateFilter.Error | HealthStateFilter.Warning
};
var applicationsFilter = new ApplicationHealthStatesFilter()
{
    HealthStateFilterValue = HealthStateFilter.Error
};
var healthStatisticsFilter = new ClusterHealthStatisticsFilter()
{
    ExcludeHealthStatistics = false,
    IncludeSystemApplicationHealthStatistics = true
};
var queryDescription = new ClusterHealthQueryDescription()
{
    HealthPolicy = policy,
    ApplicationsFilter = applicationsFilter,
    NodesFilter = nodesFilter,
    HealthStatisticsFilter = healthStatisticsFilter
};

ClusterHealth clusterHealth = await fabricClient.HealthManager.GetClusterHealthAsync(queryDescription);
```

### <a name="powershell"></a>PowerShell
El cmdlet para obtener el mantenimiento del clúster es [Get-ServiceFabricClusterHealth](/powershell/module/servicefabric/get-servicefabricclusterhealth). Conéctese primero al clúster mediante el cmdlet [Connect-ServiceFabricCluster](/powershell/module/servicefabric/connect-servicefabriccluster?view=azureservicefabricps) .

El estado del clúster es igual a cinco nodos, la aplicación del sistema y fabric:/WordCount se configuran tal como se ha descrito.

El siguiente cmdlet obtiene el mantenimiento del clúster con directivas de mantenimiento predeterminadas. El estado de mantenimiento agregado está en advertencia, porque también lo está la aplicación fabric:/WordCount. Observe cómo las evaluaciones de mantenimiento incorrecto muestran detalles de las condiciones que desencadenaron el mantenimiento agregado.

```xml
PS D:\ServiceFabric> Get-ServiceFabricClusterHealth


AggregatedHealthState   : Warning
UnhealthyEvaluations    : 
                          Unhealthy applications: 100% (1/1), MaxPercentUnhealthyApplications=0%.

                          Unhealthy application: ApplicationName='fabric:/WordCount', AggregatedHealthState='Warning'.

                            Unhealthy services: 100% (1/1), ServiceType='WordCountServiceType', MaxPercentUnhealthyServices=0%.

                            Unhealthy service: ServiceName='fabric:/WordCount/WordCountService', AggregatedHealthState='Warning'.

                                Unhealthy partitions: 100% (1/1), MaxPercentUnhealthyPartitionsPerService=0%.

                                Unhealthy partition: PartitionId='af2e3e44-a8f8-45ac-9f31-4093eb897600', AggregatedHealthState='Warning'.

                                    Unhealthy event: SourceId='System.FM', Property='State', HealthState='Warning', ConsiderWarningAsError=false.


NodeHealthStates        : 
                          NodeName              : _Node_4
                          AggregatedHealthState : Ok

                          NodeName              : _Node_3
                          AggregatedHealthState : Ok

                          NodeName              : _Node_2
                          AggregatedHealthState : Ok

                          NodeName              : _Node_1
                          AggregatedHealthState : Ok

                          NodeName              : _Node_0
                          AggregatedHealthState : Ok

ApplicationHealthStates : 
                          ApplicationName       : fabric:/System
                          AggregatedHealthState : Ok

                          ApplicationName       : fabric:/WordCount
                          AggregatedHealthState : Warning

HealthEvents            : None
HealthStatistics        : 
                          Node                  : 5 Ok, 0 Warning, 0 Error
                          Replica               : 6 Ok, 0 Warning, 0 Error
                          Partition             : 1 Ok, 1 Warning, 0 Error
                          Service               : 1 Ok, 1 Warning, 0 Error
                          DeployedServicePackage : 6 Ok, 0 Warning, 0 Error
                          DeployedApplication   : 5 Ok, 0 Warning, 0 Error
                          Application           : 0 Ok, 1 Warning, 0 Error
```

El siguiente cmdlet de PowerShell obtiene el mantenimiento del clúster mediante una directiva de aplicación personalizada. Filtra los resultados para obtener solo los nodos y las aplicaciones con error o advertencia. Como resultado, no se devuelve ningún nodo, ya que todos son correctos. Solo la aplicación fabric:/WordCount respeta el filtro de aplicaciones. Puesto que la directiva personalizada especifica que hay que considerar que las advertencias son errores en la aplicación fabric:/WordCount, la aplicación se evalúa como en error y lo mismo el clúster.

```powershell
PS D:\ServiceFabric> $appHealthPolicy = New-Object -TypeName System.Fabric.Health.ApplicationHealthPolicy
$appHealthPolicy.ConsiderWarningAsError = $true
$appHealthPolicyMap = New-Object -TypeName System.Fabric.Health.ApplicationHealthPolicyMap
$appUri1 = New-Object -TypeName System.Uri -ArgumentList "fabric:/WordCount"
$appHealthPolicyMap.Add($appUri1, $appHealthPolicy)
Get-ServiceFabricClusterHealth -ApplicationHealthPolicyMap $appHealthPolicyMap -ApplicationsFilter "Warning,Error" -NodesFilter "Warning,Error" -ExcludeHealthStatistics


AggregatedHealthState   : Error
UnhealthyEvaluations    : 
                          Unhealthy applications: 100% (1/1), MaxPercentUnhealthyApplications=0%.

                          Unhealthy application: ApplicationName='fabric:/WordCount', AggregatedHealthState='Error'.

                            Unhealthy services: 100% (1/1), ServiceType='WordCountServiceType', MaxPercentUnhealthyServices=0%.

                            Unhealthy service: ServiceName='fabric:/WordCount/WordCountService', AggregatedHealthState='Error'.

                                Unhealthy partitions: 100% (1/1), MaxPercentUnhealthyPartitionsPerService=0%.

                                Unhealthy partition: PartitionId='af2e3e44-a8f8-45ac-9f31-4093eb897600', AggregatedHealthState='Error'.

                                    Unhealthy event: SourceId='System.FM', Property='State', HealthState='Warning', ConsiderWarningAsError=true.


NodeHealthStates        : None
ApplicationHealthStates : 
                          ApplicationName       : fabric:/WordCount
                          AggregatedHealthState : Error

HealthEvents            : None
```

### <a name="rest"></a>REST
Puede obtener el mantenimiento del clúster con una [solicitud GET](/rest/api/servicefabric/get-the-health-of-a-cluster) o una [solicitud POST](/rest/api/servicefabric/get-the-health-of-a-cluster-by-using-a-health-policy) que incluye las directivas de mantenimiento descritas en el cuerpo.

## <a name="get-node-health"></a>Obtención del mantenimiento de nodo
Devuelve el mantenimiento de una entidad del nodo y contiene los eventos de mantenimiento notificados en el nodo. Entrada:

* [Obligatorio] El nombre de nodo que identifica al nodo.
* [Opcional] La configuración de la directiva de mantenimiento del clúster usada para evaluar el mantenimiento.
* [Opcional] Filtros para eventos que especifican las entradas que son de interés y se deben devolver en el resultado (por ejemplo, únicamente errores o advertencias y errores). Todos los eventos se utilizan para evaluar el mantenimiento agregado de la entidad, independientemente del filtro.

### <a name="api"></a>API
Para obtener el mantenimiento del nodo mediante la API, cree una instancia de `FabricClient` y llame al método [GetNodeHealthAsync](/dotnet/api/system.fabric.fabricclient.healthclient.getnodehealthasync) en su instancia de HealthManager.

El código siguiente obtiene el mantenimiento del nodo para el nombre de nodo especificado:

```csharp
NodeHealth nodeHealth = await fabricClient.HealthManager.GetNodeHealthAsync(nodeName);
```

El código siguiente obtiene el mantenimiento del nodo para el nombre de nodo especificado, y pasa un filtro de eventos y la directiva personalizada mediante [NodeHealthQueryDescription](/dotnet/api/system.fabric.description.nodehealthquerydescription):

```csharp
var queryDescription = new NodeHealthQueryDescription(nodeName)
{
    HealthPolicy = new ClusterHealthPolicy() {  ConsiderWarningAsError = true },
    EventsFilter = new HealthEventsFilter() { HealthStateFilterValue = HealthStateFilter.Warning },
};

NodeHealth nodeHealth = await fabricClient.HealthManager.GetNodeHealthAsync(queryDescription);
```

### <a name="powershell"></a>PowerShell
El cmdlet para obtener el mantenimiento del nodo es [Get-ServiceFabricNodeHealth](/powershell/module/servicefabric/get-servicefabricnodehealth). Conéctese primero al clúster mediante el cmdlet [Connect-ServiceFabricCluster](/powershell/module/servicefabric/connect-servicefabriccluster?view=azureservicefabricps) .
El siguiente cmdlet obtiene el mantenimiento del nodo mediante directivas de mantenimiento predeterminadas:

```powershell
PS D:\ServiceFabric> Get-ServiceFabricNodeHealth _Node_1


NodeName              : _Node_1
AggregatedHealthState : Ok
HealthEvents          : 
                        SourceId              : System.FM
                        Property              : State
                        HealthState           : Ok
                        SequenceNumber        : 3
                        SentAt                : 7/13/2017 4:39:23 PM
                        ReceivedAt            : 7/13/2017 4:40:47 PM
                        TTL                   : Infinite
                        Description           : Fabric node is up.
                        RemoveWhenExpired     : False
                        IsExpired             : False
                        Transitions           : Error->Ok = 7/13/2017 4:40:47 PM, LastWarning = 1/1/0001 12:00:00 AM
```

El siguiente cmdlet obtiene el mantenimiento de todos los nodos del clúster:

```powershell
PS D:\ServiceFabric> Get-ServiceFabricNode | Get-ServiceFabricNodeHealth | select NodeName, AggregatedHealthState | ft -AutoSize

NodeName AggregatedHealthState
-------- ---------------------
_Node_4                     Ok
_Node_3                     Ok
_Node_2                     Ok
_Node_1                     Ok
_Node_0                     Ok
```

### <a name="rest"></a>REST
Puede obtener el mantenimiento del nodo con una [solicitud GET](/rest/api/servicefabric/get-the-health-of-a-node) o una [solicitud POST](/rest/api/servicefabric/get-the-health-of-a-node-by-using-a-health-policy) que incluye las directivas de mantenimiento descritas en el cuerpo.

## <a name="get-application-health"></a>Obtención del mantenimiento de la aplicación
Devuelve el mantenimiento de una entidad de aplicación. Contiene los estados de mantenimiento de la aplicación implementada y de los elementos secundarios del servicio. Entrada:

* [Obligatorio] El nombre de la aplicación (URI) que identifica la aplicación.
* [Opcional] La directiva de mantenimiento de aplicación usada para invalidar las directivas de manifiesto de aplicación.
* [Opcional] Filtros para eventos, servicios y aplicaciones implementadas que especifican las entradas que son de interés y se deben devolver en el resultado (por ejemplo, únicamente errores o advertencias y errores). Todos los eventos, servicios y aplicaciones implementadas se utilizan para evaluar el mantenimiento agregado de la entidad, independientemente del filtro.
* [Opcional] Filtro para excluir las estadísticas de estado. Si no se especifica, las estadísticas de estado incluyen los estados correcto, de advertencia y recuento de errores para todos los elementos secundarios de la aplicación: servicios, particiones, réplicas, aplicaciones implementadas y paquetes de servicio implementados.

### <a name="api"></a>API
Para obtener el mantenimiento de la aplicación, cree una instancia de `FabricClient` y llame al método [GetApplicationHealthAsync](/dotnet/api/system.fabric.fabricclient.healthclient.getapplicationhealthasync) en su instancia de HealthManager.

El código siguiente obtiene el mantenimiento de la aplicación para el nombre de aplicación especificado (URI):

```csharp
ApplicationHealth applicationHealth = await fabricClient.HealthManager.GetApplicationHealthAsync(applicationName);
```

El código siguiente obtiene el mantenimiento de la aplicación para el nombre de aplicación especificado (URI), con filtros y directivas personalizadas especificados mediante [ApplicationHealthQueryDescription](/dotnet/api/system.fabric.description.applicationhealthquerydescription).

```csharp
HealthStateFilter warningAndErrors = HealthStateFilter.Error | HealthStateFilter.Warning;
var serviceTypePolicy = new ServiceTypeHealthPolicy()
{
    MaxPercentUnhealthyPartitionsPerService = 0,
    MaxPercentUnhealthyReplicasPerPartition = 5,
    MaxPercentUnhealthyServices = 0,
};
var policy = new ApplicationHealthPolicy()
{
    ConsiderWarningAsError = false,
    DefaultServiceTypeHealthPolicy = serviceTypePolicy,
    MaxPercentUnhealthyDeployedApplications = 0,
};

var queryDescription = new ApplicationHealthQueryDescription(applicationName)
{
    HealthPolicy = policy,
    EventsFilter = new HealthEventsFilter() { HealthStateFilterValue = warningAndErrors },
    ServicesFilter = new ServiceHealthStatesFilter() { HealthStateFilterValue = warningAndErrors },
    DeployedApplicationsFilter = new DeployedApplicationHealthStatesFilter() { HealthStateFilterValue = warningAndErrors },
};

ApplicationHealth applicationHealth = await fabricClient.HealthManager.GetApplicationHealthAsync(queryDescription);
```

### <a name="powershell"></a>PowerShell
El cmdlet para obtener el mantenimiento de la aplicación es [Get-ServiceFabricApplicationHealth](/powershell/module/servicefabric/get-servicefabricapplicationhealth?view=azureservicefabricps). Conéctese primero al clúster mediante el cmdlet [Connect-ServiceFabricCluster](/powershell/module/servicefabric/connect-servicefabriccluster?view=azureservicefabricps) .

El siguiente cmdlet devuelve el mantenimiento de la aplicación **fabric:/WordCount** :

```powershell
PS D:\ServiceFabric> Get-ServiceFabricApplicationHealth fabric:/WordCount


ApplicationName                 : fabric:/WordCount
AggregatedHealthState           : Warning
UnhealthyEvaluations            : 
                                  Unhealthy services: 100% (1/1), ServiceType='WordCountServiceType', MaxPercentUnhealthyServices=0%.

                                  Unhealthy service: ServiceName='fabric:/WordCount/WordCountService', AggregatedHealthState='Warning'.

                                    Unhealthy partitions: 100% (1/1), MaxPercentUnhealthyPartitionsPerService=0%.

                                    Unhealthy partition: PartitionId='af2e3e44-a8f8-45ac-9f31-4093eb897600', AggregatedHealthState='Warning'.

                                        Unhealthy event: SourceId='System.FM', Property='State', HealthState='Warning', ConsiderWarningAsError=false.

ServiceHealthStates             : 
                                  ServiceName           : fabric:/WordCount/WordCountWebService
                                  AggregatedHealthState : Ok

                                  ServiceName           : fabric:/WordCount/WordCountService
                                  AggregatedHealthState : Warning

DeployedApplicationHealthStates : 
                                  ApplicationName       : fabric:/WordCount
                                  NodeName              : _Node_4
                                  AggregatedHealthState : Ok

                                  ApplicationName       : fabric:/WordCount
                                  NodeName              : _Node_3
                                  AggregatedHealthState : Ok

                                  ApplicationName       : fabric:/WordCount
                                  NodeName              : _Node_0
                                  AggregatedHealthState : Ok

                                  ApplicationName       : fabric:/WordCount
                                  NodeName              : _Node_2
                                  AggregatedHealthState : Ok

                                  ApplicationName       : fabric:/WordCount
                                  NodeName              : _Node_1
                                  AggregatedHealthState : Ok

HealthEvents                    : 
                                  SourceId              : System.CM
                                  Property              : State
                                  HealthState           : Ok
                                  SequenceNumber        : 282
                                  SentAt                : 7/13/2017 5:57:05 PM
                                  ReceivedAt            : 7/13/2017 5:57:05 PM
                                  TTL                   : Infinite
                                  Description           : Application has been created.
                                  RemoveWhenExpired     : False
                                  IsExpired             : False
                                  Transitions           : Error->Ok = 7/13/2017 5:57:05 PM, LastWarning = 1/1/0001 12:00:00 AM

HealthStatistics                : 
                                  Replica               : 6 Ok, 0 Warning, 0 Error
                                  Partition             : 1 Ok, 1 Warning, 0 Error
                                  Service               : 1 Ok, 1 Warning, 0 Error
                                  DeployedServicePackage : 6 Ok, 0 Warning, 0 Error
                                  DeployedApplication   : 5 Ok, 0 Warning, 0 Error
```

El siguiente cmdlet de PowerShell pasa directivas personalizadas. También filtra los elementos secundarios y los eventos.

```powershell
PS D:\ServiceFabric> Get-ServiceFabricApplicationHealth -ApplicationName fabric:/WordCount -ConsiderWarningAsError $true -ServicesFilter Error -EventsFilter Error -DeployedApplicationsFilter Error -ExcludeHealthStatistics


ApplicationName                 : fabric:/WordCount
AggregatedHealthState           : Error
UnhealthyEvaluations            : 
                                  Unhealthy services: 100% (1/1), ServiceType='WordCountServiceType', MaxPercentUnhealthyServices=0%.

                                  Unhealthy service: ServiceName='fabric:/WordCount/WordCountService', AggregatedHealthState='Error'.

                                    Unhealthy partitions: 100% (1/1), MaxPercentUnhealthyPartitionsPerService=0%.

                                    Unhealthy partition: PartitionId='af2e3e44-a8f8-45ac-9f31-4093eb897600', AggregatedHealthState='Error'.

                                        Unhealthy event: SourceId='System.FM', Property='State', HealthState='Warning', ConsiderWarningAsError=true.

ServiceHealthStates             : 
                                  ServiceName           : fabric:/WordCount/WordCountService
                                  AggregatedHealthState : Error

DeployedApplicationHealthStates : None
HealthEvents                    : None
```

### <a name="rest"></a>REST
Puede obtener el mantenimiento de la aplicación con una [solicitud GET](/rest/api/servicefabric/get-the-health-of-an-application) o una [solicitud POST](/rest/api/servicefabric/get-the-health-of-an-application-by-using-an-application-health-policy) que incluye las directivas de mantenimiento descritas en el cuerpo.

## <a name="get-service-health"></a>Obtención del mantenimiento del servicio
Devuelve el mantenimiento de una entidad del servicio. Contiene los estados de mantenimiento de la partición. Entrada:

* [Obligatorio] El nombre del servicio (URI) que identifica el servicio.
* [Opcional] La directiva de mantenimiento de aplicación usada para invalidar la directiva de manifiesto de aplicación.
* [Opcional] Filtros para eventos y particiones que especifican las entradas que son de interés y se deben devolver en el resultado (por ejemplo, únicamente errores o advertencias y errores). Todos los eventos y particiones se utilizan para evaluar el mantenimiento agregado de la entidad, independientemente del filtro.
* [Opcional] Filtro para excluir las estadísticas de estado. Si no se especifica, las estadísticas de estado muestran el estado correcto, de advertencia y recuento de errores para todas las particiones y réplicas del servicio.

### <a name="api"></a>API
Para obtener el mantenimiento del servicio mediante la API, cree una instancia de `FabricClient` y llame al método [GetServiceHealthAsync](/dotnet/api/system.fabric.fabricclient.healthclient.getservicehealthasync) en su instancia de HealthManager.

En el ejemplo siguiente se obtiene el mantenimiento de un servicio con el nombre de servicio especificado (URI):

```csharp
ServiceHealth serviceHealth = await fabricClient.HealthManager.GetServiceHealthAsync(serviceName);
```

El código siguiente obtiene el mantenimiento del servicio para el nombre de servicio especificado (URI), mediante la especificación de filtros y directivas personalizadas con la clase [ServiceHealthQueryDescription](/dotnet/api/system.fabric.description.servicehealthquerydescription):

```csharp
var queryDescription = new ServiceHealthQueryDescription(serviceName)
{
    EventsFilter = new HealthEventsFilter() { HealthStateFilterValue = HealthStateFilter.All },
    PartitionsFilter = new PartitionHealthStatesFilter() { HealthStateFilterValue = HealthStateFilter.Error },
};

ServiceHealth serviceHealth = await fabricClient.HealthManager.GetServiceHealthAsync(queryDescription);
```

### <a name="powershell"></a>PowerShell
El cmdlet para obtener el mantenimiento del servicio es [Get-ServiceFabricServiceHealth](/powershell/module/servicefabric/get-servicefabricservicehealth). Conéctese primero al clúster mediante el cmdlet [Connect-ServiceFabricCluster](/powershell/module/servicefabric/connect-servicefabriccluster?view=azureservicefabricps) .

El siguiente cmdlet obtiene el mantenimiento del servicio con directivas de mantenimiento predeterminadas.

```powershell
PS D:\ServiceFabric> Get-ServiceFabricServiceHealth -ServiceName fabric:/WordCount/WordCountService


ServiceName           : fabric:/WordCount/WordCountService
AggregatedHealthState : Warning
UnhealthyEvaluations  : 
                        Unhealthy partitions: 100% (1/1), MaxPercentUnhealthyPartitionsPerService=0%.

                        Unhealthy partition: PartitionId='af2e3e44-a8f8-45ac-9f31-4093eb897600', AggregatedHealthState='Warning'.

                            Unhealthy event: SourceId='System.FM', Property='State', HealthState='Warning', ConsiderWarningAsError=false.

PartitionHealthStates : 
                        PartitionId           : af2e3e44-a8f8-45ac-9f31-4093eb897600
                        AggregatedHealthState : Warning

HealthEvents          : 
                        SourceId              : System.FM
                        Property              : State
                        HealthState           : Ok
                        SequenceNumber        : 15
                        SentAt                : 7/13/2017 5:57:05 PM
                        ReceivedAt            : 7/13/2017 5:57:18 PM
                        TTL                   : Infinite
                        Description           : Service has been created.
                        RemoveWhenExpired     : False
                        IsExpired             : False
                        Transitions           : Error->Ok = 7/13/2017 5:57:18 PM, LastWarning = 1/1/0001 12:00:00 AM

HealthStatistics      : 
                        Replica               : 5 Ok, 0 Warning, 0 Error
                        Partition             : 0 Ok, 1 Warning, 0 Error
```

### <a name="rest"></a>REST
Puede obtener el mantenimiento del servicio con una [solicitud GET](/rest/api/servicefabric/get-the-health-of-a-service) o una [solicitud POST](/rest/api/servicefabric/get-the-health-of-a-service-by-using-a-health-policy) que incluye las directivas de mantenimiento descritas en el cuerpo.

## <a name="get-partition-health"></a>Obtención del mantenimiento de partición
Devuelve el mantenimiento de una entidad de partición. Contiene los estados de mantenimiento de la réplica. Entrada:

* [Obligatorio] El identificador de partición (GUID) que identifica la partición.
* [Opcional] La directiva de mantenimiento de aplicación usada para invalidar la directiva de manifiesto de aplicación.
* [Opcional] Filtros de eventos y réplicas que especifican las entradas que son de interés y se deben devolver en el resultado (por ejemplo, únicamente errores o advertencias y errores). Todos los eventos y réplicas se utilizan para evaluar el mantenimiento agregado de la entidad, independientemente del filtro.
* [Opcional] Filtro para excluir las estadísticas de estado. Si no se especifica, las estadísticas de estado muestran cuántas réplicas están en estado correcto, de advertencia y de error.

### <a name="api"></a>API
Para obtener el mantenimiento de la partición mediante la API, cree una instancia de `FabricClient` y llame al método [GetPartitionHealthAsync](/dotnet/api/system.fabric.fabricclient.healthclient.getpartitionhealthasync) en su instancia de HealthManager. Para especificar parámetros opcionales, cree [PartitionHealthQueryDescription](/dotnet/api/system.fabric.description.partitionhealthquerydescription).

```csharp
PartitionHealth partitionHealth = await fabricClient.HealthManager.GetPartitionHealthAsync(partitionId);
```

### <a name="powershell"></a>PowerShell
El cmdlet para obtener el mantenimiento de la partición es [Get-ServiceFabricPartitionHealth](/powershell/module/servicefabric/get-servicefabricpartitionhealth). Conéctese primero al clúster mediante el cmdlet [Connect-ServiceFabricCluster](/powershell/module/servicefabric/connect-servicefabriccluster?view=azureservicefabricps) .

El siguiente cmdlet obtiene el estado de todas las particiones del servicio **fabric:/WordCount/WordCountService** y filtra los estados de mantenimiento de las réplicas:

```powershell
PS D:\ServiceFabric> Get-ServiceFabricPartition fabric:/WordCount/WordCountService | Get-ServiceFabricPartitionHealth -ReplicasFilter None

PartitionId           : af2e3e44-a8f8-45ac-9f31-4093eb897600
AggregatedHealthState : Warning
UnhealthyEvaluations  : 
                        Unhealthy event: SourceId='System.FM', Property='State', HealthState='Warning', ConsiderWarningAsError=false.

ReplicaHealthStates   : None
HealthEvents          : 
                        SourceId              : System.FM
                        Property              : State
                        HealthState           : Warning
                        SequenceNumber        : 72
                        SentAt                : 7/13/2017 5:57:29 PM
                        ReceivedAt            : 7/13/2017 5:57:48 PM
                        TTL                   : Infinite
                        Description           : Partition is below target replica or instance count.
                        fabric:/WordCount/WordCountService 7 2 af2e3e44-a8f8-45ac-9f31-4093eb897600
                          N/P RD _Node_2 Up 131444422260002646
                          N/S RD _Node_4 Up 131444422293113678
                          N/S RD _Node_3 Up 131444422293113679
                          N/S RD _Node_1 Up 131444422293118720
                          N/S RD _Node_0 Up 131444422293118721
                          (Showing 5 out of 5 replicas. Total available replicas: 5.)

                        RemoveWhenExpired     : False
                        IsExpired             : False
                        Transitions           : Ok->Warning = 7/13/2017 5:57:48 PM, LastError = 1/1/0001 12:00:00 AM

                        SourceId              : System.PLB
                        Property              : ServiceReplicaUnplacedHealth_Secondary_af2e3e44-a8f8-45ac-9f31-4093eb897600
                        HealthState           : Warning
                        SequenceNumber        : 131444445174851664
                        SentAt                : 7/13/2017 6:35:17 PM
                        ReceivedAt            : 7/13/2017 6:35:18 PM
                        TTL                   : 00:01:05
                        Description           : The Load Balancer was unable to find a placement for one or more of the Service's Replicas:
                        Secondary replica could not be placed due to the following constraints and properties:  
                        TargetReplicaSetSize: 7
                        Placement Constraint: N/A
                        Parent Service: N/A

                        Constraint Elimination Sequence:
                        Existing Secondary Replicas eliminated 4 possible node(s) for placement -- 1/5 node(s) remain.
                        Existing Primary Replica eliminated 1 possible node(s) for placement -- 0/5 node(s) remain.

                        Nodes Eliminated By Constraints:

                        Existing Secondary Replicas -- Nodes with Partition's Existing Secondary Replicas/Instances:
                        --
                        FaultDomain:fd:/4 NodeName:_Node_4 NodeType:NodeType4 UpgradeDomain:4 UpgradeDomain: ud:/4 Deactivation Intent/Status: None/None
                        FaultDomain:fd:/3 NodeName:_Node_3 NodeType:NodeType3 UpgradeDomain:3 UpgradeDomain: ud:/3 Deactivation Intent/Status: None/None
                        FaultDomain:fd:/1 NodeName:_Node_1 NodeType:NodeType1 UpgradeDomain:1 UpgradeDomain: ud:/1 Deactivation Intent/Status: None/None
                        FaultDomain:fd:/0 NodeName:_Node_0 NodeType:NodeType0 UpgradeDomain:0 UpgradeDomain: ud:/0 Deactivation Intent/Status: None/None

                        Existing Primary Replica -- Nodes with Partition's Existing Primary Replica or Secondary Replicas:
                        --
                        FaultDomain:fd:/2 NodeName:_Node_2 NodeType:NodeType2 UpgradeDomain:2 UpgradeDomain: ud:/2 Deactivation Intent/Status: None/None


                        RemoveWhenExpired     : True
                        IsExpired             : False
                        Transitions           : Error->Warning = 7/13/2017 5:57:48 PM, LastOk = 1/1/0001 12:00:00 AM

HealthStatistics      : 
                        Replica               : 5 Ok, 0 Warning, 0 Error
```

### <a name="rest"></a>REST
Puede obtener el mantenimiento de la partición con una [solicitud GET](/rest/api/servicefabric/sfclient-api-getpartitionhealth) o una [solicitud POST](/rest/api/servicefabric/get-the-health-of-a-partition-by-using-a-health-policy) que incluye las directivas de mantenimiento descritas en el cuerpo.

## <a name="get-replica-health"></a>Obtención del mantenimiento de réplica
Devuelve el mantenimiento de una réplica de servicio con estado o una instancia de servicio sin estado. Entrada:

* [Obligatorio] El identificador de partición (GUID) y el identificador de réplica que identifica a la réplica.
* [Opcional] Los parámetros de la directiva de mantenimiento de aplicación usados para invalidar las directivas de manifiesto de aplicación.
* [Opcional] Filtros para eventos que especifican las entradas que son de interés y se deben devolver en el resultado (por ejemplo, únicamente errores o advertencias y errores). Todos los eventos se utilizan para evaluar el mantenimiento agregado de la entidad, independientemente del filtro.

### <a name="api"></a>API
Para obtener el mantenimiento de la réplica mediante la API, cree una instancia de `FabricClient` y llame al método [GetReplicaHealthAsync](/dotnet/api/system.fabric.fabricclient.healthclient.getreplicahealthasync) en su instancia de HealthManager. Para especificar parámetros avanzados, utilice [ReplicaHealthQueryDescription](/dotnet/api/system.fabric.description.replicahealthquerydescription).

```csharp
ReplicaHealth replicaHealth = await fabricClient.HealthManager.GetReplicaHealthAsync(partitionId, replicaId);
```

### <a name="powershell"></a>PowerShell
El cmdlet para obtener el mantenimiento de la réplica es [Get-ServiceFabricReplicaHealth](/powershell/module/servicefabric/get-servicefabricreplicahealth). Conéctese primero al clúster mediante el cmdlet [Connect-ServiceFabricCluster](/powershell/module/servicefabric/connect-servicefabriccluster?view=azureservicefabricps) .

El cmdlet siguiente obtiene el mantenimiento de la réplica principal para todas las particiones del servicio:

```powershell
PS D:\ServiceFabric> Get-ServiceFabricPartition fabric:/WordCount/WordCountService | Get-ServiceFabricReplica | where {$_.ReplicaRole -eq "Primary"} | Get-ServiceFabricReplicaHealth


PartitionId           : af2e3e44-a8f8-45ac-9f31-4093eb897600
ReplicaId             : 131444422260002646
AggregatedHealthState : Ok
HealthEvents          : 
                        SourceId              : System.RA
                        Property              : State
                        HealthState           : Ok
                        SequenceNumber        : 131444422263668344
                        SentAt                : 7/13/2017 5:57:06 PM
                        ReceivedAt            : 7/13/2017 5:57:18 PM
                        TTL                   : Infinite
                        Description           : Replica has been created._Node_2
                        RemoveWhenExpired     : False
                        IsExpired             : False
                        Transitions           : Error->Ok = 7/13/2017 5:57:18 PM, LastWarning = 1/1/0001 12:00:00 AM
```

### <a name="rest"></a>REST
Puede obtener el mantenimiento de la réplica con una [solicitud GET](/rest/api/servicefabric/get-the-health-of-a-replica) o una[solicitud POST](/rest/api/servicefabric/get-the-health-of-a-replica-by-using-a-health-policy) que incluye las directivas de mantenimiento descritas en el cuerpo.

## <a name="get-deployed-application-health"></a>Obtención del mantenimiento de la aplicación implementada
Devuelve el mantenimiento de una aplicación implementada en una actividad del nodo. Contiene los estados de mantenimiento del paquete de servicio implementado. Entrada:

* [Obligatorio] El nombre de la aplicación (URI) y el nombre del nodo (cadena) que identifican a la aplicación implementada
* [Opcional] La directiva de mantenimiento de aplicación usada para invalidar las directivas de manifiesto de aplicación.
* [Opcional] Filtros de eventos y paquetes de servicio implementados que especifican las entradas que son de interés y se deben devolver en el resultado (por ejemplo, únicamente errores o advertencias y errores). Todos los eventos y paquetes de servicio implementados se utilizan para evaluar el mantenimiento agregado de la entidad, independientemente del filtro.
* [Opcional] Filtro para excluir las estadísticas de estado. Si no se especifica, las estadísticas de estado muestran el número de paquetes de servicio implementados en los estados de mantenimiento correcto, de advertencia y de error.

### <a name="api"></a>API
Para obtener el mantenimiento de una aplicación implementada en un nodo mediante la API, cree una instancia de `FabricClient` y llame al método [GetDeployedApplicationHealthAsync](/dotnet/api/system.fabric.fabricclient.healthclient.getdeployedapplicationhealthasync) en su instancia de HealthManager. Para especificar parámetros opcionales, utilice [DeployedApplicationHealthQueryDescription](/dotnet/api/system.fabric.description.deployedapplicationhealthquerydescription).

```csharp
DeployedApplicationHealth health = await fabricClient.HealthManager.GetDeployedApplicationHealthAsync(
    new DeployedApplicationHealthQueryDescription(applicationName, nodeName));
```

### <a name="powershell"></a>PowerShell
El cmdlet para obtener el mantenimiento de la aplicación implementada es [Get-ServiceFabricDeployedApplicationHealth](/powershell/module/servicefabric/get-servicefabricdeployedapplicationhealth?view=azureservicefabricps). Conéctese primero al clúster mediante el cmdlet [Connect-ServiceFabricCluster](/powershell/module/servicefabric/connect-servicefabriccluster?view=azureservicefabricps) . Para averiguar dónde está implementada una aplicación, ejecute [Get-ServiceFabricApplicationHealth](/powershell/module/servicefabric/get-servicefabricapplicationhealth?view=azureservicefabricps) y observe los elementos secundarios de la aplicación implementada.

El siguiente cmdlet obtiene el mantenimiento de la aplicación **fabric:/WordCount** implementada en **_Node_2**.

```powershell
PS D:\ServiceFabric> Get-ServiceFabricDeployedApplicationHealth -ApplicationName fabric:/WordCount -NodeName _Node_0


ApplicationName                    : fabric:/WordCount
NodeName                           : _Node_0
AggregatedHealthState              : Ok
DeployedServicePackageHealthStates : 
                                     ServiceManifestName   : WordCountServicePkg
                                     ServicePackageActivationId : 
                                     NodeName              : _Node_0
                                     AggregatedHealthState : Ok

                                     ServiceManifestName   : WordCountWebServicePkg
                                     ServicePackageActivationId : 
                                     NodeName              : _Node_0
                                     AggregatedHealthState : Ok

HealthEvents                       : 
                                     SourceId              : System.Hosting
                                     Property              : Activation
                                     HealthState           : Ok
                                     SequenceNumber        : 131444422261848308
                                     SentAt                : 7/13/2017 5:57:06 PM
                                     ReceivedAt            : 7/13/2017 5:57:17 PM
                                     TTL                   : Infinite
                                     Description           : The application was activated successfully.
                                     RemoveWhenExpired     : False
                                     IsExpired             : False
                                     Transitions           : Error->Ok = 7/13/2017 5:57:17 PM, LastWarning = 1/1/0001 12:00:00 AM

HealthStatistics                   : 
                                     DeployedServicePackage : 2 Ok, 0 Warning, 0 Error
```

### <a name="rest"></a>REST
Puede obtener el mantenimiento de la aplicación implementada con una [solicitud GET](/rest/api/servicefabric/get-the-health-of-a-deployed-application) o una [solicitud POST](/rest/api/servicefabric/get-the-health-of-a-deployed-application-by-using-a-health-policy) que incluye las directivas de mantenimiento descritas en el cuerpo.

## <a name="get-deployed-service-package-health"></a>Obtención del mantenimiento del paquete de servicio implementado
Devuelve el mantenimiento de una entidad de paquete de servicio implementado. Entrada:

* [Obligatorio] El nombre de la aplicación (URI), el nombre del nodo (cadena) y el nombre del manifiesto de servicio (cadena) que identifican al paquete de servicio implementado.
* [Opcional] La directiva de mantenimiento de aplicación usada para invalidar la directiva de manifiesto de aplicación.
* [Opcional] Filtros para eventos que especifican las entradas que son de interés y se deben devolver en el resultado (por ejemplo, únicamente errores o advertencias y errores). Todos los eventos se utilizan para evaluar el mantenimiento agregado de la entidad, independientemente del filtro.

### <a name="api"></a>API
Para obtener el mantenimiento de un paquete de servicio implementado mediante la API, cree una instancia de `FabricClient` y llame al método [GetDeployedServicePackageHealthAsync](/dotnet/api/system.fabric.fabricclient.healthclient.getdeployedservicepackagehealthasync) en su instancia de HealthManager. Para especificar parámetros opcionales, utilice [DeployedServicePackageHealthQueryDescription](/dotnet/api/system.fabric.description.deployedservicepackagehealthquerydescription).

```csharp
DeployedServicePackageHealth health = await fabricClient.HealthManager.GetDeployedServicePackageHealthAsync(
    new DeployedServicePackageHealthQueryDescription(applicationName, nodeName, serviceManifestName));
```

### <a name="powershell"></a>PowerShell
El cmdlet para obtener el mantenimiento del paquete de servicio implementado es [Get-ServiceFabricDeployedServicePackageHealth](/powershell/module/servicefabric/get-servicefabricdeployedservicepackagehealth). Conéctese primero al clúster mediante el cmdlet [Connect-ServiceFabricCluster](/powershell/module/servicefabric/connect-servicefabriccluster?view=azureservicefabricps) . Para averiguar dónde está implementada una aplicación, ejecute [Get-ServiceFabricApplicationHealth](/powershell/module/servicefabric/get-servicefabricapplicationhealth?view=azureservicefabricps) y examine las aplicaciones implementadas. Para ver qué paquetes de servicio están en una aplicación, examine los elementos secundarios del paquete de servicio implementado en la salida de [Get-ServiceFabricDeployedApplicationHealth](/powershell/module/servicefabric/get-servicefabricdeployedapplicationhealth?view=azureservicefabricps) .

El siguiente cmdlet obtiene el mantenimiento del paquete de servicio **WordCountServicePkg** de la aplicación **fabric:/WordCount** implementada en **_Node_2**. La entidad tiene informes **System.Hosting** para la activación correcta del paquete de servicio y del punto de entrada, y para el registro correcto del tipo de servicio.

```powershell
PS D:\ServiceFabric> Get-ServiceFabricDeployedApplication -ApplicationName fabric:/WordCount -NodeName _Node_2 | Get-ServiceFabricDeployedServicePackageHealth -ServiceManifestName WordCountServicePkg


ApplicationName            : fabric:/WordCount
ServiceManifestName        : WordCountServicePkg
ServicePackageActivationId : 
NodeName                   : _Node_2
AggregatedHealthState      : Ok
HealthEvents               : 
                             SourceId              : System.Hosting
                             Property              : Activation
                             HealthState           : Ok
                             SequenceNumber        : 131444422267693359
                             SentAt                : 7/13/2017 5:57:06 PM
                             ReceivedAt            : 7/13/2017 5:57:18 PM
                             TTL                   : Infinite
                             Description           : The ServicePackage was activated successfully.
                             RemoveWhenExpired     : False
                             IsExpired             : False
                             Transitions           : Error->Ok = 7/13/2017 5:57:18 PM, LastWarning = 1/1/0001 12:00:00 AM

                             SourceId              : System.Hosting
                             Property              : CodePackageActivation:Code:EntryPoint
                             HealthState           : Ok
                             SequenceNumber        : 131444422267903345
                             SentAt                : 7/13/2017 5:57:06 PM
                             ReceivedAt            : 7/13/2017 5:57:18 PM
                             TTL                   : Infinite
                             Description           : The CodePackage was activated successfully.
                             RemoveWhenExpired     : False
                             IsExpired             : False
                             Transitions           : Error->Ok = 7/13/2017 5:57:18 PM, LastWarning = 1/1/0001 12:00:00 AM

                             SourceId              : System.Hosting
                             Property              : ServiceTypeRegistration:WordCountServiceType
                             HealthState           : Ok
                             SequenceNumber        : 131444422272458374
                             SentAt                : 7/13/2017 5:57:07 PM
                             ReceivedAt            : 7/13/2017 5:57:18 PM
                             TTL                   : Infinite
                             Description           : The ServiceType was registered successfully.
                             RemoveWhenExpired     : False
                             IsExpired             : False
                             Transitions           : Error->Ok = 7/13/2017 5:57:18 PM, LastWarning = 1/1/0001 12:00:00 AM
```

### <a name="rest"></a>REST
Puede obtener el mantenimiento del paquete de servicio implementado con una [solicitud GET](/rest/api/servicefabric/get-the-health-of-a-service-package) o una [solicitud POST](/rest/api/servicefabric/get-the-health-of-a-service-package-by-using-a-health-policy) que incluye las directivas de mantenimiento descritas en el cuerpo.

## <a name="health-chunk-queries"></a>Consultas de fragmentos de mantenimiento
Las consultas de fragmentos de mantenimiento pueden devolver elementos secundarios de clúster de varios niveles (recursivamente), según los filtros de entrada. Admite filtros avanzados que permiten una gran flexibilidad al elegir los elementos secundarios que se devuelven. Los filtros pueden especificar elementos secundarios por el identificador único o por otros identificadores de grupo o estados de mantenimiento. De forma predeterminada, no se incluye ningún elemento secundario, a diferencia de los comandos de mantenimiento que siempre incluyen elementos secundarios de primer nivel.

Las [consultas de mantenimiento](service-fabric-view-entities-aggregated-health.md#health-queries) solo devuelven elementos secundarios de primer nivel de la entidad especificada, según los filtros necesarios. Para obtener los elementos secundarios de los elementos secundarios, debe llamar a las API de mantenimiento adicionales para cada entidad que sea de interés. De igual forma, para obtener el mantenimiento de entidades específicas, debe llamar a una API de mantenimiento para cada entidad deseada. El filtrado avanzado de consultas de fragmentos le permite solicitar varios elementos de interés en una consulta, lo que reduce el tamaño del mensaje y el número de mensajes.

El valor de la consulta de fragmentos es que puede obtener el estado de mantenimiento para más entidades del clúster (posiblemente todas las entidades del clúster que comienzan en la raíz requerida) en una llamada. Puede expresar consultas de mantenimiento complejas, por ejemplo:

* Devolver solo las aplicaciones con error y, para esas aplicaciones, incluir todos los servicios con advertencias o errores. Para los servicios devueltos, incluir todas las particiones.
* Devolver solo el mantenimiento de cuatro aplicaciones, especificadas por sus nombres.
* Devolver solo el mantenimiento de aplicaciones de un tipo de aplicación deseado.
* Devolver todas las entidades implementadas en un nodo. Devuelve todas las aplicaciones, todas las aplicaciones implementadas en el nodo especificado y todos los paquetes de servicio implementados en ese nodo.
* Devolver todas las réplicas con error. Devuelve todas las aplicaciones, los servicios, las particiones y solo las réplicas con error.
* Devolver todas las aplicaciones. Para un servicio especificado, incluir todas las particiones.

Actualmente, la consulta de fragmentos de mantenimiento se expone solo para la entidad del clúster. Devuelve un fragmento de estado del clúster, que contiene:

* El estado de mantenimiento agregado del clúster.
* La lista de fragmentos de estado de mantenimiento de nodos que respetan los filtros de entrada.
* La lista de fragmentos de estado de mantenimiento de aplicaciones que respetan los filtros de entrada. Cada fragmento de estado de mantenimiento de aplicación contiene una lista de fragmentos con todos los servicios que respetan los filtros de entrada y una lista de fragmentos con todas las aplicaciones implementadas que respetan los filtros. Igual para los elementos secundarios de servicios y aplicaciones implementadas. De este modo, todas las entidades del clúster pueden devolverse potencialmente si se solicitan, de una manera jerárquica.

### <a name="cluster-health-chunk-query"></a>Consulta de fragmentos de mantenimiento del clúster
Devuelve el mantenimiento de la entidad del clúster y contiene los fragmentos del estado de mantenimiento jerárquico de los elementos secundarios necesarios. Entrada:

* [Opcional] La directiva de mantenimiento del clúster utilizada para evaluar los nodos y los eventos del clúster.
* [Opcional] La asignación de directivas de mantenimiento de aplicaciones, con las directivas de mantenimiento usadas para invalidar las directivas de manifiesto de aplicación.
* [Opcional] Filtros para nodos y aplicaciones que especifican las entradas que son de interés y se deben devolver en el resultado. Los filtros son específicos de una entidad o grupo de entidades o son aplicables a todas las entidades de ese nivel. La lista de filtros puede contener un filtro general o filtros para identificadores específicos a fin de precisar más las entidades devueltas por la consulta. Si está vacía, no se devuelven los elementos secundarios de forma predeterminada.
  Más información sobre los filtros en [NodeHealthStateFilter](/dotnet/api/system.fabric.health.nodehealthstatefilter) y [ApplicationHealthStateFilter](/dotnet/api/system.fabric.health.applicationhealthstatefilter). Los filtros de aplicación pueden especificar de forma recursiva filtros avanzados para elementos secundarios.

El resultado del fragmento incluye los elementos secundarios que respetan los filtros.

Actualmente, la consulta de fragmentos no devuelve evaluaciones de mantenimiento incorrecto o eventos de entidad. Esta información adicional puede obtenerse mediante la consulta de mantenimiento del clúster existente.

### <a name="api"></a>API
Para obtener fragmentos de mantenimiento del clúster, cree una instancia de `FabricClient` y llame al método [GetClusterHealthChunkAsync](/dotnet/api/system.fabric.fabricclient.healthclient.getclusterhealthchunkasync) en su instancia de **HealthManager**. Puede pasar [ClusterHealthQueryDescription](/dotnet/api/system.fabric.description.clusterhealthchunkquerydescription) para describir directivas de mantenimiento y filtros avanzados.

El código siguiente obtiene fragmentos de mantenimiento del clúster con filtros avanzados.

```csharp
var queryDescription = new ClusterHealthChunkQueryDescription();
queryDescription.ApplicationFilters.Add(new ApplicationHealthStateFilter()
    {
        // Return applications only if they are in error
        HealthStateFilter = HealthStateFilter.Error
    });

// Return all replicas
var wordCountServiceReplicaFilter = new ReplicaHealthStateFilter()
    {
        HealthStateFilter = HealthStateFilter.All
    };

// Return all replicas and all partitions
var wordCountServicePartitionFilter = new PartitionHealthStateFilter()
    {
        HealthStateFilter = HealthStateFilter.All
    };
wordCountServicePartitionFilter.ReplicaFilters.Add(wordCountServiceReplicaFilter);

// For specific service, return all partitions and all replicas
var wordCountServiceFilter = new ServiceHealthStateFilter()
{
    ServiceNameFilter = new Uri("fabric:/WordCount/WordCountService"),
};
wordCountServiceFilter.PartitionFilters.Add(wordCountServicePartitionFilter);

// Application filter: for specific application, return no services except the ones of interest
var wordCountApplicationFilter = new ApplicationHealthStateFilter()
    {
        // Always return fabric:/WordCount application
        ApplicationNameFilter = new Uri("fabric:/WordCount"),
    };
wordCountApplicationFilter.ServiceFilters.Add(wordCountServiceFilter);

queryDescription.ApplicationFilters.Add(wordCountApplicationFilter);

var result = await fabricClient.HealthManager.GetClusterHealthChunkAsync(queryDescription);
```

### <a name="powershell"></a>PowerShell
El cmdlet para obtener el mantenimiento del clúster es [Get-ServiceFabricClusterChunkHealth](/powershell/module/servicefabric/get-servicefabricclusterhealthchunk). Conéctese primero al clúster mediante el cmdlet [Connect-ServiceFabricCluster](/powershell/module/servicefabric/connect-servicefabriccluster?view=azureservicefabricps) .

El código siguiente obtiene los nodos solo si se encuentran en estado de error, a excepción de un nodo específico, que se debe devolver siempre.

```xml
PS D:\ServiceFabric> $errorFilter = [System.Fabric.Health.HealthStateFilter]::Error;
$allFilter = [System.Fabric.Health.HealthStateFilter]::All;

$nodeFilter1 = New-Object System.Fabric.Health.NodeHealthStateFilter -Property @{HealthStateFilter=$errorFilter}
$nodeFilter2 = New-Object System.Fabric.Health.NodeHealthStateFilter -Property @{NodeNameFilter="_Node_1";HealthStateFilter=$allFilter}
# Create node filter list that will be passed in the cmdlet
$nodeFilters = New-Object System.Collections.Generic.List[System.Fabric.Health.NodeHealthStateFilter]
$nodeFilters.Add($nodeFilter1)
$nodeFilters.Add($nodeFilter2)

Get-ServiceFabricClusterHealthChunk -NodeFilters $nodeFilters


HealthState                  : Warning
NodeHealthStateChunks        : 
                               TotalCount            : 1

                               NodeName              : _Node_1
                               HealthState           : Ok

ApplicationHealthStateChunks : None
```

El siguiente cmdlet obtiene fragmentos del clúster con filtros de aplicación.

```xml
PS D:\ServiceFabric> $errorFilter = [System.Fabric.Health.HealthStateFilter]::Error;
$allFilter = [System.Fabric.Health.HealthStateFilter]::All;

# All replicas
$replicaFilter = New-Object System.Fabric.Health.ReplicaHealthStateFilter -Property @{HealthStateFilter=$allFilter}

# All partitions
$partitionFilter = New-Object System.Fabric.Health.PartitionHealthStateFilter -Property @{HealthStateFilter=$allFilter}
$partitionFilter.ReplicaFilters.Add($replicaFilter)

# For WordCountService, return all partitions and all replicas
$svcFilter1 = New-Object System.Fabric.Health.ServiceHealthStateFilter -Property @{ServiceNameFilter="fabric:/WordCount/WordCountService"}
$svcFilter1.PartitionFilters.Add($partitionFilter)

$svcFilter2 = New-Object System.Fabric.Health.ServiceHealthStateFilter -Property @{HealthStateFilter=$errorFilter}

$appFilter = New-Object System.Fabric.Health.ApplicationHealthStateFilter -Property @{ApplicationNameFilter="fabric:/WordCount"}
$appFilter.ServiceFilters.Add($svcFilter1)
$appFilter.ServiceFilters.Add($svcFilter2)

$appFilters = New-Object System.Collections.Generic.List[System.Fabric.Health.ApplicationHealthStateFilter]
$appFilters.Add($appFilter)

Get-ServiceFabricClusterHealthChunk -ApplicationFilters $appFilters


HealthState                  : Error
NodeHealthStateChunks        : None
ApplicationHealthStateChunks : 
                               TotalCount            : 1

                               ApplicationName       : fabric:/WordCount
                               ApplicationTypeName   : WordCount
                               HealthState           : Error
                               ServiceHealthStateChunks : 
                                TotalCount            : 1

                                ServiceName           : fabric:/WordCount/WordCountService
                                HealthState           : Error
                                PartitionHealthStateChunks : 
                                    TotalCount            : 1

                                    PartitionId           : af2e3e44-a8f8-45ac-9f31-4093eb897600
                                    HealthState           : Error
                                    ReplicaHealthStateChunks : 
                                        TotalCount            : 5

                                        ReplicaOrInstanceId   : 131444422293118720
                                        HealthState           : Ok

                                        ReplicaOrInstanceId   : 131444422293118721
                                        HealthState           : Ok

                                        ReplicaOrInstanceId   : 131444422293113678
                                        HealthState           : Ok

                                        ReplicaOrInstanceId   : 131444422293113679
                                        HealthState           : Ok

                                        ReplicaOrInstanceId   : 131444422260002646
                                        HealthState           : Error
```

El siguiente cmdlet devuelve todas las entidades implementadas en un nodo.

```xml
PS D:\ServiceFabric> $errorFilter = [System.Fabric.Health.HealthStateFilter]::Error;
$allFilter = [System.Fabric.Health.HealthStateFilter]::All;

$dspFilter = New-Object System.Fabric.Health.DeployedServicePackageHealthStateFilter -Property @{HealthStateFilter=$allFilter}
$daFilter =  New-Object System.Fabric.Health.DeployedApplicationHealthStateFilter -Property @{HealthStateFilter=$allFilter;NodeNameFilter="_Node_2"}
$daFilter.DeployedServicePackageFilters.Add($dspFilter)

$appFilter = New-Object System.Fabric.Health.ApplicationHealthStateFilter -Property @{HealthStateFilter=$allFilter}
$appFilter.DeployedApplicationFilters.Add($daFilter)

$appFilters = New-Object System.Collections.Generic.List[System.Fabric.Health.ApplicationHealthStateFilter]
$appFilters.Add($appFilter)
Get-ServiceFabricClusterHealthChunk -ApplicationFilters $appFilters


HealthState                  : Error
NodeHealthStateChunks        : None
ApplicationHealthStateChunks : 
                               TotalCount            : 2

                               ApplicationName       : fabric:/System
                               HealthState           : Ok
                               DeployedApplicationHealthStateChunks : 
                                TotalCount            : 1

                                NodeName              : _Node_2
                                HealthState           : Ok
                                DeployedServicePackageHealthStateChunks :
                                    TotalCount            : 1

                                    ServiceManifestName   : FAS
                                    ServicePackageActivationId : 
                                    HealthState           : Ok



                               ApplicationName       : fabric:/WordCount
                               ApplicationTypeName   : WordCount
                               HealthState           : Error
                               DeployedApplicationHealthStateChunks : 
                                TotalCount            : 1

                                NodeName              : _Node_2
                                HealthState           : Ok
                                DeployedServicePackageHealthStateChunks :
                                    TotalCount            : 1

                                    ServiceManifestName   : WordCountServicePkg
                                    ServicePackageActivationId : 
                                    HealthState           : Ok
```

### <a name="rest"></a>REST
Puede obtener fragmentos de mantenimiento del clúster con una [solicitud GET](/rest/api/servicefabric/get-the-health-of-a-cluster-using-health-chunks) o una [solicitud POST](/rest/api/servicefabric/health-of-cluster) que incluye las directivas de mantenimiento y filtros avanzados descritos en el cuerpo.

## <a name="general-queries"></a>Consultas generales
Las consultas generales devuelven una lista de entidades de Service Fabric de un tipo especificado. Se exponen mediante la API (por medio de los métodos de **FabricClient.QueryManager**), los cmdlets de PowerShell y REST. Estas consultas agregan subconsultas de varios componentes. Uno de ellos es el [Almacén de estado](service-fabric-health-introduction.md#health-store), que rellena el estado de mantenimiento agregado para cada resultado de consulta.  

> [!NOTE]
> Las consultas generales devuelven el estado de mantenimiento agregado de la entidad y no contienen los datos completos de mantenimiento. Si el mantenimiento de una entidad no es correcto, puede seguir con las consultas de mantenimiento para obtener toda su información de mantenimiento, como eventos, estados de mantenimiento de los elementos secundarios y evaluaciones de mantenimiento incorrecto.
>
>

Si las consultas generales devuelven un estado de mantenimiento desconocido para una entidad, es posible que el almacén de estado no tenga datos completos sobre la entidad. También es posible que una subconsulta al almacén de estado no fuera correcta (por ejemplo, se produjo un error de comunicación o se limitó el almacén de estado). Realice un seguimiento con una consulta de mantenimiento para la entidad. Si la subconsulta se encontró con errores transitorios, como problemas de red, esta consulta de seguimiento puede ser de ayuda. También le puede proporcionar más detalles del almacén de estado sobre por qué no se expone la entidad.

Las consultas que contienen **HealthState** para las entidades son las siguientes:

* Lista de nodos: devuelve la lista de nodos del clúster (paginada).
  * API: [FabricClient.QueryClient.GetNodeListAsync](/dotnet/api/system.fabric.fabricclient.queryclient.getnodelistasync)
  * PowerShell: Get-ServiceFabricNode
* Lista de aplicaciones: devuelve la lista de aplicaciones del clúster (paginada).
  * API: [FabricClient.QueryClient.GetApplicationListAsync](/dotnet/api/system.fabric.fabricclient.queryclient.getapplicationlistasync)
  * PowerShell: Get-ServiceFabricApplication
* Lista de servicios: devuelve la lista de servicios de una aplicación (paginada).
  * API: [FabricClient.QueryClient.GetServiceListAsync](/dotnet/api/system.fabric.fabricclient.queryclient.getservicelistasync)
  * PowerShell: Get-ServiceFabricService
* Lista de particiones: devuelve la lista de particiones de un servicio (paginada).
  * API: [FabricClient.QueryClient.GetPartitionListAsync](/dotnet/api/system.fabric.fabricclient.queryclient.getpartitionlistasync)
  * PowerShell: Get-ServiceFabricPartition
* Lista de réplicas: devuelve la lista de réplicas de una partición (paginada).
  * API: [FabricClient.QueryClient.GetReplicaListAsync](/dotnet/api/system.fabric.fabricclient.queryclient.getreplicalistasync)
  * PowerShell: Get-ServiceFabricReplica
* Lista de aplicaciones implementadas: devuelve la lista de aplicaciones implementadas en un nodo.
  * API: [FabricClient.QueryClient.GetDeployedApplicationListAsync](/dotnet/api/system.fabric.fabricclient.queryclient.getdeployedapplicationlistasync)
  * PowerShell: Get-ServiceFabricDeployedApplication
* Lista de paquetes de servicio implementados: devuelve la lista de paquetes de servicio de una aplicación implementada.
  * API: [FabricClient.QueryClient.GetDeployedServicePackageListAsync](/dotnet/api/system.fabric.fabricclient.queryclient.getdeployedservicepackagelistasync)
  * PowerShell: Get-ServiceFabricDeployedApplication

> [!NOTE]
> Algunas de las consultas devuelven resultados paginados. La devolución de estas consultas es una lista que se deriva de [PagedList\<T>](/dotnet/api/system.fabric.query.pagedlist-1). Si los resultados no caben en un mensaje, solo se devuelve una página y ContinuationToken, que realiza el seguimiento de dónde se detuvo la enumeración. Continúe llamando a la misma consulta y pase el token de continuación de la consulta anterior para obtener los siguientes resultados.

### <a name="examples"></a>Ejemplos
El código siguiente obtiene las aplicaciones incorrectas en el clúster:

```csharp
var applications = fabricClient.QueryManager.GetApplicationListAsync().Result.Where(
  app => app.HealthState == HealthState.Error);
```

El siguiente cmdlet obtiene los detalles de la aplicación fabric:/WordCount. Observe que el estado de mantenimiento está en advertencia.

```powershell
PS C:\> Get-ServiceFabricApplication -ApplicationName fabric:/WordCount

ApplicationName        : fabric:/WordCount
ApplicationTypeName    : WordCount
ApplicationTypeVersion : 1.0.0
ApplicationStatus      : Ready
HealthState            : Warning
ApplicationParameters  : { "WordCountWebService_InstanceCount" = "1";
                         "_WFDebugParams_" = "[{"ServiceManifestName":"WordCountWebServicePkg","CodePackageName":"Code","EntryPointType":"Main","Debug
                         ExePath":"C:\\Program Files (x86)\\Microsoft Visual Studio
                         14.0\\Common7\\Packages\\Debugger\\VsDebugLaunchNotify.exe","DebugArguments":" {74f7e5d5-71a9-47e2-a8cd-1878ec4734f1} -p
                         [ProcessId] -tid [ThreadId]","EnvironmentBlock":"_NO_DEBUG_HEAP=1\u0000"},{"ServiceManifestName":"WordCountServicePkg","CodeP
                         ackageName":"Code","EntryPointType":"Main","DebugExePath":"C:\\Program Files (x86)\\Microsoft Visual Studio
                         14.0\\Common7\\Packages\\Debugger\\VsDebugLaunchNotify.exe","DebugArguments":" {2ab462e6-e0d1-4fda-a844-972f561fe751} -p
                         [ProcessId] -tid [ThreadId]","EnvironmentBlock":"_NO_DEBUG_HEAP=1\u0000"}]" }
```

El siguiente cmdlet obtiene los servicios con un estado de mantenimiento de error:

```powershell
PS D:\ServiceFabric> Get-ServiceFabricApplication | Get-ServiceFabricService | where {$_.HealthState -eq "Error"}


ServiceName            : fabric:/WordCount/WordCountService
ServiceKind            : Stateful
ServiceTypeName        : WordCountServiceType
IsServiceGroup         : False
ServiceManifestVersion : 1.0.0
HasPersistedState      : True
ServiceStatus          : Active
HealthState            : Error
```

## <a name="cluster-and-application-upgrades"></a>Actualización de clústeres y aplicaciones
Durante una actualización supervisada del clúster y la aplicación, Service Fabric comprueba el mantenimiento para asegurarse de que todo está correcto. Si una entidad es incorrecta según se ha evaluado mediante las directivas de mantenimiento configuradas, la actualización aplica directivas específicas de actualización para determinar la próxima acción. La actualización se puede poner en pausa para permitir la interacción de los usuarios (por ejemplo, corregir las condiciones de error o cambiar las directivas), o se puede revertir automáticamente a la versión buena anterior.

Durante la actualización de un *clúster* , puede obtener su estado de actualización. El estado de actualización incluye las evaluaciones de mantenimiento incorrecto, que señalan lo que está mal en el clúster. Si se revierte la actualización debido a problemas de mantenimiento, el estado de actualización recuerda las razones del último mantenimiento incorrecto. Esta información puede ayudar a los administradores a investigar qué salió mal después de que se revirtiera o detuviera la actualización.

De igual forma, durante la actualización de una *aplicación* , las evaluaciones de mantenimiento incorrecto están contenidas en el estado de actualización de la aplicación.

A continuación se muestra el estado de actualización de la aplicación para una aplicación fabric/WordCount modificada. Un guardián ha informado de un error en una de sus réplicas. La actualización se revierte porque no se respetan las comprobaciones de mantenimiento.

```powershell
PS C:\> Get-ServiceFabricApplicationUpgrade fabric:/WordCount

ApplicationName               : fabric:/WordCount
ApplicationTypeName           : WordCount
TargetApplicationTypeVersion  : 1.0.0.0
ApplicationParameters         : {}
StartTimestampUtc             : 4/21/2017 5:23:26 PM
FailureTimestampUtc           : 4/21/2017 5:23:37 PM
FailureReason                 : HealthCheck
UpgradeState                  : RollingBackInProgress
UpgradeDuration               : 00:00:23
CurrentUpgradeDomainDuration  : 00:00:00
CurrentUpgradeDomainProgress  : UD1

                                NodeName            : _Node_1
                                UpgradePhase        : Upgrading

                                NodeName            : _Node_2
                                UpgradePhase        : Upgrading

                                NodeName            : _Node_3
                                UpgradePhase        : PreUpgradeSafetyCheck
                                PendingSafetyChecks :
                                EnsurePartitionQuorum - PartitionId: 30db5be6-4e20-4698-8185-4bd7ca744020
NextUpgradeDomain             : UD2
UpgradeDomainsStatus          : { "UD1" = "Completed";
                                "UD2" = "Pending";
                                "UD3" = "Pending";
                                "UD4" = "Pending" }
UnhealthyEvaluations          :
                                Unhealthy services: 100% (1/1), ServiceType='WordCountServiceType', MaxPercentUnhealthyServices=0%.

                                  Unhealthy service: ServiceName='fabric:/WordCount/WordCountService', AggregatedHealthState='Error'.

                                      Unhealthy partitions: 100% (1/1), MaxPercentUnhealthyPartitionsPerService=0%.

                                      Unhealthy partition: PartitionId='a1f83a35-d6bf-4d39-b90d-28d15f39599b', AggregatedHealthState='Error'.

                                          Unhealthy replicas: 20% (1/5), MaxPercentUnhealthyReplicasPerPartition=0%.

                                          Unhealthy replica: PartitionId='a1f83a35-d6bf-4d39-b90d-28d15f39599b',
                                  ReplicaOrInstanceId='131031502346844058', AggregatedHealthState='Error'.

                                              Error event: SourceId='DiskWatcher', Property='Disk'.

UpgradeKind                   : Rolling
RollingUpgradeMode            : UnmonitoredAuto
ForceRestart                  : False
UpgradeReplicaSetCheckTimeout : 00:15:00
```

Más información sobre la [actualización de aplicaciones de Service Fabric](service-fabric-application-upgrade.md).

## <a name="use-health-evaluations-to-troubleshoot"></a>Uso de las evaluaciones de mantenimiento para solucionar problemas
Siempre que haya un problema con el clúster o una aplicación, consulte el mantenimiento del clúster o de la aplicación para analizar lo que pasa. Las evaluaciones de mantenimiento incorrecto proporcionan detalles sobre lo que activó el estado incorrecto actual. Si lo necesita, puede explorar en profundidad las entidades secundarias incorrectas para identificar la causa principal.

Por ejemplo, considere una aplicación con un estado incorrecto porque hay un informe de errores en una de sus réplicas. El siguiente cmdlet de Powershell muestra las evaluaciones de estado incorrecto:

```powershell
PS D:\ServiceFabric> Get-ServiceFabricApplicationHealth fabric:/WordCount -EventsFilter None -ServicesFilter None -DeployedApplicationsFilter None -ExcludeHealthStatistics


ApplicationName                 : fabric:/WordCount
AggregatedHealthState           : Error
UnhealthyEvaluations            : 
                                  Unhealthy services: 100% (1/1), ServiceType='WordCountServiceType', MaxPercentUnhealthyServices=0%.

                                  Unhealthy service: ServiceName='fabric:/WordCount/WordCountService', AggregatedHealthState='Error'.

                                    Unhealthy partitions: 100% (1/1), MaxPercentUnhealthyPartitionsPerService=0%.

                                    Unhealthy partition: PartitionId='af2e3e44-a8f8-45ac-9f31-4093eb897600', AggregatedHealthState='Error'.

                                        Unhealthy replicas: 20% (1/5), MaxPercentUnhealthyReplicasPerPartition=0%.

                                        Unhealthy replica: PartitionId='af2e3e44-a8f8-45ac-9f31-4093eb897600', ReplicaOrInstanceId='131444422260002646', AggregatedHealthState='Error'.

                                            Error event: SourceId='MyWatchdog', Property='Memory'.

ServiceHealthStates             : None
DeployedApplicationHealthStates : None
HealthEvents                    : None
```

Puede mirar la réplica para obtener más información:

```powershell
PS D:\ServiceFabric> Get-ServiceFabricReplicaHealth -ReplicaOrInstanceId 131444422260002646 -PartitionId af2e3e44-a8f8-45ac-9f31-4093eb897600


PartitionId           : af2e3e44-a8f8-45ac-9f31-4093eb897600
ReplicaId             : 131444422260002646
AggregatedHealthState : Error
UnhealthyEvaluations  : 
                        Error event: SourceId='MyWatchdog', Property='Memory'.

HealthEvents          : 
                        SourceId              : System.RA
                        Property              : State
                        HealthState           : Ok
                        SequenceNumber        : 131444422263668344
                        SentAt                : 7/13/2017 5:57:06 PM
                        ReceivedAt            : 7/13/2017 5:57:18 PM
                        TTL                   : Infinite
                        Description           : Replica has been created._Node_2
                        RemoveWhenExpired     : False
                        IsExpired             : False
                        Transitions           : Error->Ok = 7/13/2017 5:57:18 PM, LastWarning = 1/1/0001 12:00:00 AM

                        SourceId              : MyWatchdog
                        Property              : Memory
                        HealthState           : Error
                        SequenceNumber        : 131444451657749403
                        SentAt                : 7/13/2017 6:46:05 PM
                        ReceivedAt            : 7/13/2017 6:46:05 PM
                        TTL                   : Infinite
                        Description           : 
                        RemoveWhenExpired     : False
                        IsExpired             : False
                        Transitions           : Warning->Error = 7/13/2017 6:46:05 PM, LastOk = 1/1/0001 12:00:00 AM
```

> [!NOTE]
> Las evaluaciones de mantenimiento incorrecto muestran la razón principal de que la entidad se evaluara con el estado de mantenimiento actual. Puede haber muchos otros eventos que desencadenen este estado pero no se reflejan en las evaluaciones. Para más información, explore en profundidad las entidades de mantenimiento para averiguar todos los informes de mantenimiento incorrecto en el clúster.
>
>

## <a name="next-steps"></a>Pasos siguientes
[Utilización de informes de mantenimiento del sistema para solucionar problemas](service-fabric-understand-and-troubleshoot-with-system-health-reports.md)

[Incorporación de informes de mantenimiento de Service Fabric personalizados](service-fabric-report-health.md)

[Notificación y comprobación del estado del servicio](service-fabric-diagnostics-how-to-report-and-check-service-health.md)

[Supervisión y diagnóstico de los servicios localmente](service-fabric-diagnostics-how-to-monitor-and-diagnose-services-locally.md)

[Actualización de la aplicación de Service Fabric](service-fabric-application-upgrade.md)
