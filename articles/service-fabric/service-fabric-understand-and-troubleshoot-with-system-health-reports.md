---
title: Solución de problemas con los informes de mantenimiento del sistema | Microsoft Docs
description: Describe los informes de mantenimiento enviados que envían componentes de Azure Service Fabric y su uso para solucionar problemas de los clústeres o las aplicaciones.
services: service-fabric
documentationcenter: .net
author: oanapl
manager: chackdan
editor: ''
ms.assetid: 52574ea7-eb37-47e0-a20a-101539177625
ms.service: service-fabric
ms.devlang: dotnet
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 2/28/2018
ms.author: oanapl
ms.openlocfilehash: d5cfe91cfcc124ef3073cfb6bbeda683505ff8e1
ms.sourcegitcommit: 179918af242d52664d3274370c6fdaec6c783eb6
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/13/2019
ms.locfileid: "65561380"
---
# <a name="use-system-health-reports-to-troubleshoot"></a>Utilización de informes de mantenimiento del sistema para solucionar problemas
Los componentes de Azure Service Fabric proporcionan informes de mantenimiento del sistema inmediatos sobre todas las entidades del clúster. El [almacén de estado](service-fabric-health-introduction.md#health-store) crea y elimina entidades basándose en los informes del sistema. También las organiza en una jerarquía que captura las interacciones de la entidad.

> [!NOTE]
> Para comprender los conceptos relacionados con el mantenimiento, obtenga más información sobre el [modelo de mantenimiento de Service Fabric](service-fabric-health-introduction.md).
> 
> 

Los informes de mantenimiento del sistema proporcionan visibilidad sobre la funcionalidad de los clústeres y las aplicaciones y señalan los problemas. Para aplicaciones y servicios, los informes de mantenimiento del sistema comprueban que las entidades se implementan y que se comportan correctamente desde la perspectiva de Service Fabric. Los informes no proporcionan ninguna supervisión del mantenimiento de la lógica de negocios del servicio o la detección de los procesos que no responden. Los servicios de usuario pueden enriquecer los datos de mantenimiento con información específica de su lógica.

> [!NOTE]
> Los informes de mantenimiento enviados por los guardianes de usuarios solo son visibles *después* de que los componentes del sistema hayan creado una entidad. Cuando se elimina una entidad, el almacén de estado elimina automáticamente todos los informes de mantenimiento asociados a ella. Lo mismo ocurre cuando se crea una nueva instancia de la entidad. Un ejemplo es cuando se crea una nueva instancia de réplica de servicio persistente con estado. Todos los informes asociados a la instancia anterior se eliminan y se limpian del almacén.
> 
> 

Los informes de los componentes del sistema se identifican mediante el origen, que comienza por el prefijo "**System.**" . Los guardianes no pueden utilizar el mismo prefijo para sus orígenes, ya que los informes con parámetros no válidos se rechazan.

Veamos algunos informes del sistema para entender qué los desencadena y cómo corregir los posibles problemas que representan.

> [!NOTE]
> Service Fabric sigue agregando informes de condiciones de interés que mejoran la visibilidad de lo que sucede en el clúster y la aplicación. También se pueden mejorar los informes existentes con más detalles para ayudar a solucionar el problema con mayor rapidez.
> 
> 

## <a name="cluster-system-health-reports"></a>Informes de mantenimiento del sistema de clúster
La entidad de mantenimiento del clúster se crea automáticamente en el Almacén de estado. Si todo funciona correctamente, no tiene un informe del sistema.

### <a name="neighborhood-loss"></a>Pérdida de entorno
**System.Federation** notifica un error cuando detecta una pérdida de entorno. El informe procede de los nodos individuales y el identificador de nodo está incluido en el nombre de la propiedad. Si hay una pérdida de entorno en todo el anillo de Service Fabric, normalmente puede esperar dos eventos que representan ambos lados del informe de carencias. Si se pierden más entornos, se producen más eventos.

El informe especifica el tiempo de expiración de concesión global como período de vida (TTL). El informe se vuelve a enviar una vez transcurrida la mitad de la duración del período de vida siempre y cuando la condición permanezca activa. El evento se quita automáticamente cuando haya expirado. El comportamiento de eliminación cuando caduca garantiza que el informe se limpie del almacén de estado correctamente, incluso si el nodo de informes está inactivo.

* **SourceId**: System.Federation
* **Propiedad**: Se inicia con **vecindario** e incluye información sobre el nodo.
* **Pasos siguientes**: Investigue por qué se pierde el entorno. Por ejemplo, compruebe la comunicación entre los nodos de clúster.

### <a name="rebuild"></a>Recompilación

El servicio Administrador de conmutación por error (FM) administra información acerca de los nodos de clúster. Cuando FM pierde los datos y entra en estado de pérdida de datos, no puede garantizar que la información que tiene sobre los nodos de clúster sea la más actualizada. En este caso, el sistema se somete a una recompilación y System.FM reúne datos de todos los nodos del clúster para recompilar su estado. En ocasiones, debido a problemas de la red o del nodo, la recompilación puede bloquearse o detenerse. Lo mismo puede suceder con el servicio Maestro administrador de conmutación por error (FMM). FMM es un servicio de sistema sin estado que realiza un seguimiento del lugar en el que se encuentran todos los FM en el clúster. El nodo principal del FMM siempre es el nodo con el identificador más cercano a 0. Si ese nodo se quita, se desencadena una recompilación.
Cuando se produce una de las condiciones anteriores, **System.FM** o **System.FMM** la marcan mediante un informe de errores. La recompilación podría bloquearse en una de estas dos fases:

* **Esperando la difusión**: FM/FMM espera la respuesta del mensaje de difusión de los demás nodos.

  * **Pasos siguientes**: Investigue si hay un problema de conexión de red entre los nodos.
* **Esperando los nodos**: FM/FMM ya recibió una respuesta de difusión de los demás nodos y está esperando una respuesta de nodos específicos. En el informe de mantenimiento se enumeran los nodos de los que FM/FMM está esperando una respuesta.
   * **Pasos siguientes**: Investigue la conexión de red entre FM/FMM y los nodos enumerados. Investigue cada nodo enumerado en busca de otros posibles problemas.

* **SourceID**: System.FM o System.fmm.
* **Propiedad**: Vuelva a generar.
* **Pasos siguientes**: Investigue la conexión de red entre los nodos, así como el estado de los nodos específicos que se muestran en la descripción del informe de mantenimiento.

## <a name="node-system-health-reports"></a>Informes de mantenimiento del sistema de nodos
System.FM, que representa el servicio Administrador de conmutación por error, es la autoridad que administra la información acerca de los nodos de clúster. Todos los nodos deben tener un informe de System.FM que muestre el estado. Las entidades de nodo se quitan cuando se quita el estado del nodo. Para más información, consulte [RemoveNodeStateAsync](https://docs.microsoft.com/dotnet/api/system.fabric.fabricclient.clustermanagementclient.removenodestateasync).

### <a name="node-updown"></a>Nodo activo o inactivo
System.FM notifica que está todo correcto cuando el nodo se une al anillo (está en funcionamiento). Notifica un error cuando el nodo sale del anillo (no funciona, ya sea porque se está actualizando o simplemente porque no pudo). La jerarquía de mantenimiento generada por el almacén de estado actúa sobre las entidades implementadas en correlación con los informes de nodo de System.FM. Considera el nodo como un elemento primario virtual de todas las entidades implementadas. Las entidades implementadas en ese nodo se exponen a través de las consultas si System.FM notifica que el nodo está activo, con la misma instancia que la instancia asociada a las entidades. Cuando System.FM informa que el nodo está inactivo o que se ha reiniciado como una nueva instancia, el almacén de estado limpia automáticamente las entidades implementadas que solo pueden existir en el nodo inactivo o en la instancia anterior del nodo.

* **SourceId**: System.FM
* **Propiedad**: Estado.
* **Pasos siguientes**: Si el nodo está inactivo durante una actualización, debería volver a una vez que se haya actualizado. En este caso, se debe cambiar el estado de mantenimiento a Correcto. Si el nodo no recupera ese estado o se produce un error, deberá investigar más el problema.

El ejemplo siguiente muestra el evento System.FM con el estado de mantenimiento Correcto para el nodo activo:

```powershell
PS C:\> Get-ServiceFabricNodeHealth  _Node_0

NodeName              : _Node_0
AggregatedHealthState : Ok
HealthEvents          : 
                        SourceId              : System.FM
                        Property              : State
                        HealthState           : Ok
                        SequenceNumber        : 8
                        SentAt                : 7/14/2017 4:54:51 PM
                        ReceivedAt            : 7/14/2017 4:55:14 PM
                        TTL                   : Infinite
                        Description           : Fabric node is up.
                        RemoveWhenExpired     : False
                        IsExpired             : False
                        Transitions           : Error->Ok = 7/14/2017 4:55:14 PM, LastWarning = 1/1/0001 12:00:00 AM
```


### <a name="certificate-expiration"></a>Caducidad del certificado
**System.FabricNode** notifica una advertencia cuando los certificados usados por el nodo están a punto de expirar. Hay tres certificados por nodo: **Certificate_cluster**, **Certificate_server**, y **Certificate_default_client**. Si faltan más de dos semanas para expirar, el estado de mantenimiento del informe es Correcto. Si faltan menos de dos semanas, el tipo de informe es una advertencia. El TTL de estos eventos es infinito y se quitan cuando un nodo deja el clúster.

* **SourceId**: System.FabricNode
* **Propiedad**: Se inicia con **certificado** y contiene más información sobre el tipo de certificado.
* **Pasos siguientes**: Actualice los certificados si están a punto de expirar.

### <a name="load-capacity-violation"></a>Infracción de la capacidad de carga
El equilibrador de carga de Service Fabric notifica una advertencia cuando detecta una infracción de la capacidad del nodo.

* **SourceId**: System.PLB
* **Propiedad**: Se inicia con **capacidad**.
* **Pasos siguientes**: Compruebe las métricas proporcionadas y vea la capacidad actual en el nodo.

### <a name="node-capacity-mismatch-for-resource-governance-metrics"></a>Error de coincidencia de la capacidad de nodo de las métricas de gobernanza de recursos
Si las capacidades de nodo definidas en el manifiesto de clúster son mayores que las capacidades de nodo reales de las métricas de gobernanza de recursos (núcleos de memoria y CPU), System.Hosting se encarga de proporcionar un aviso. Aparece un informe de mantenimiento cuando el primer paquete de servicio que usa la [gobernanza de recursos](service-fabric-resource-governance.md) se registra en un nodo específico.

* **SourceId**: System.Hosting
* **Propiedad**: **ResourceGovernance**.
* **Pasos siguientes**: Este problema puede ser un problema porque rige los paquetes de servicio no se aplican según lo previsto y [regulación de recursos](service-fabric-resource-governance.md) no funciona correctamente. Puede actualizar el manifiesto de clúster con las capacidades de nodo correctas para estas métricas, o puede no especificarlas y permitir que Service Fabric detecte automáticamente los recursos disponibles.

## <a name="application-system-health-reports"></a>Informes de mantenimiento del sistema de la aplicación
System.CM, que representa el servicio Administrador de clústeres, es la autoridad que administra la información acerca de una aplicación.

### <a name="state"></a>Estado
System.CM notifica un estado Correcto cuando se ha creado o actualizado la aplicación. Informa al almacén de estado de cuándo se elimina la aplicación, por lo que puede quitarse del almacén.

* **SourceId**: System.CM
* **Propiedad**: Estado.
* **Pasos siguientes**: Si la aplicación se ha creado o actualizado, debe incluir el informe de mantenimiento del Administrador de clústeres. En caso contrario, compruebe el estado de la aplicación mediante la emisión de una consulta. Por ejemplo, use el cmdlet de PowerShell **Get-ServiceFabricApplication -ApplicationName** *applicationName*.

El ejemplo siguiente muestra el evento de estado en la aplicación **fabric:/WordCount** :

```powershell
PS C:\> Get-ServiceFabricApplicationHealth fabric:/WordCount -ServicesFilter None -DeployedApplicationsFilter None -ExcludeHealthStatistics

ApplicationName                 : fabric:/WordCount
AggregatedHealthState           : Ok
ServiceHealthStates             : None
DeployedApplicationHealthStates : None
HealthEvents                    : 
                                  SourceId              : System.CM
                                  Property              : State
                                  HealthState           : Ok
                                  SequenceNumber        : 282
                                  SentAt                : 7/13/2017 5:57:05 PM
                                  ReceivedAt            : 7/14/2017 4:55:10 PM
                                  TTL                   : Infinite
                                  Description           : Application has been created.
                                  RemoveWhenExpired     : False
                                  IsExpired             : False
                                  Transitions           : Error->Ok = 7/13/2017 5:57:05 PM, LastWarning = 1/1/0001 12:00:00 AM
```

## <a name="service-system-health-reports"></a>Informes de mantenimiento del sistema de servicio
System.FM, que representa el servicio Administrador de conmutación por error, es la autoridad que administra la información acerca de los servicios.

### <a name="state"></a>Estado
System.FM notifica un estado Correcto cuando se ha creado el servicio. Elimina la entidad del almacén de estado cuando se elimina el servicio.

* **SourceId**: System.FM
* **Propiedad**: Estado.

El ejemplo siguiente muestra el evento de estado en el servicio **fabric:/WordCount/WordCountService**:

```powershell
PS C:\> Get-ServiceFabricServiceHealth fabric:/WordCount/WordCountWebService -ExcludeHealthStatistics


ServiceName           : fabric:/WordCount/WordCountWebService
AggregatedHealthState : Ok
PartitionHealthStates : 
                        PartitionId           : 8bbcd03a-3a53-47ec-a5f1-9b77f73c53b2
                        AggregatedHealthState : Ok
                        
HealthEvents          : 
                        SourceId              : System.FM
                        Property              : State
                        HealthState           : Ok
                        SequenceNumber        : 14
                        SentAt                : 7/13/2017 5:57:05 PM
                        ReceivedAt            : 7/14/2017 4:55:10 PM
                        TTL                   : Infinite
                        Description           : Service has been created.
                        RemoveWhenExpired     : False
                        IsExpired             : False
                        Transitions           : Error->Ok = 7/13/2017 5:57:18 PM, LastWarning = 1/1/0001 12:00:00 AM
```

### <a name="service-correlation-error"></a>Error de correlación de servicio
**System.PLB** notifica un error cuando detecta que la actualización de un servicio que se va a correlacionar con otro crea una cadena de afinidad. El informe se borra cuando se produce una actualización correcta.

* **SourceId**: System.PLB
* **Propiedad**: **ServiceDescription**.
* **Pasos siguientes**: Compruebe las descripciones de servicios correlacionados.

## <a name="partition-system-health-reports"></a>Informes de mantenimiento del sistema de partición
System.FM, que representa el servicio Administrador de conmutación por error, es la autoridad que administra la información acerca de las particiones del servicio.

### <a name="state"></a>Estado
System.FM notifica un estado Correcto cuando la partición se ha creado y es correcta. Elimina la entidad del almacén de estado cuando se ha eliminado la partición.

Si la partición es inferior al recuento mínimo de réplicas, notifica un error. Si la partición no es inferior al recuento mínimo de réplicas, pero es inferior al recuento objetivo de réplicas, notifica una advertencia. Si la partición está en situación de pérdida de cuórum, System.FM notifica un error.

Otros eventos importantes incluyen una advertencia cuando la reconfiguración tarda más tiempo de lo esperado y cuando la compilación tarda más de lo previsto. Los tiempos de compilación o reconfiguración previstos se pueden configurar en función de los escenarios de servicio. Por ejemplo, si un servicio tiene un terabyte de estado como, por ejemplo, Azure SQL Database, la compilación tarda más tiempo del que tardaría para un servicio con una pequeña cantidad de estado.

* **SourceId**: System.FM
* **Propiedad**: Estado.
* **Pasos siguientes**: Si el estado de mantenimiento no es correcto, es posible que algunas réplicas no han se ha creado, abiertos o promover a principal o secundaria correctamente. 

Si la descripción detalla la pérdida de cuórum, examinar el informe de mantenimiento detallado de las réplicas que están inactivas y luego activarlas ayuda a volver a poner en línea la partición.

Si la descripción indica una interrupción de la partición en la [reconfiguración](service-fabric-concepts-reconfiguration.md), el informe de mantenimiento de la réplica principal proporciona información adicional.

En el caso de otros informes de mantenimiento de System.FM, habrá informes sobre las réplicas, o la partición o servicio de otros componentes del sistema. 

En los ejemplos siguientes se describen algunos de estos informes. 

El ejemplo siguiente muestra una partición correcta:

```powershell
PS C:\> Get-ServiceFabricPartition fabric:/WordCount/WordCountWebService | Get-ServiceFabricPartitionHealth -ExcludeHealthStatistics -ReplicasFilter None

PartitionId           : 8bbcd03a-3a53-47ec-a5f1-9b77f73c53b2
AggregatedHealthState : Ok
ReplicaHealthStates   : None
HealthEvents          : 
                        SourceId              : System.FM
                        Property              : State
                        HealthState           : Ok
                        SequenceNumber        : 70
                        SentAt                : 7/13/2017 5:57:05 PM
                        ReceivedAt            : 7/14/2017 4:55:10 PM
                        TTL                   : Infinite
                        Description           : Partition is healthy.
                        RemoveWhenExpired     : False
                        IsExpired             : False
                        Transitions           : Error->Ok = 7/13/2017 5:57:18 PM, LastWarning = 1/1/0001 12:00:00 AM
```

En el ejemplo siguiente se muestra el mantenimiento de una partición que es inferior al recuento objetivo de réplicas. El siguiente paso es obtener la descripción de la partición, que muestra cómo se ha configurado: **MinReplicaSetSize** es tres y **TargetReplicaSetSize** es siete. A continuación, obtenga el número de nodos del clúster, que en este caso es cinco. En este caso, por lo tanto, no se pueden colocar dos réplicas porque el número de réplicas objetivo es mayor que el número de nodos disponibles.

```powershell
PS C:\> Get-ServiceFabricPartition fabric:/WordCount/WordCountService | Get-ServiceFabricPartitionHealth -ReplicasFilter None -ExcludeHealthStatistics


PartitionId           : af2e3e44-a8f8-45ac-9f31-4093eb897600
AggregatedHealthState : Warning
UnhealthyEvaluations  : 
                        Unhealthy event: SourceId='System.FM', Property='State', HealthState='Warning', ConsiderWarningAsError=false.
                        
ReplicaHealthStates   : None
HealthEvents          : 
                        SourceId              : System.FM
                        Property              : State
                        HealthState           : Warning
                        SequenceNumber        : 123
                        SentAt                : 7/14/2017 4:55:39 PM
                        ReceivedAt            : 7/14/2017 4:55:44 PM
                        TTL                   : Infinite
                        Description           : Partition is below target replica or instance count.
                        fabric:/WordCount/WordCountService 7 2 af2e3e44-a8f8-45ac-9f31-4093eb897600
                          N/S Ready _Node_2 131444422260002646
                          N/S Ready _Node_4 131444422293113678
                          N/S Ready _Node_3 131444422293113679
                          N/S Ready _Node_1 131444422293118720
                          N/P Ready _Node_0 131444422293118721
                          (Showing 5 out of 5 replicas. Total available replicas: 5)
                        
                        RemoveWhenExpired     : False
                        IsExpired             : False
                        Transitions           : Error->Warning = 7/14/2017 4:55:44 PM, LastOk = 1/1/0001 12:00:00 AM
                        
                        SourceId              : System.PLB
                        Property              : ServiceReplicaUnplacedHealth_Secondary_af2e3e44-a8f8-45ac-9f31-4093eb897600
                        HealthState           : Warning
                        SequenceNumber        : 131445250939703027
                        SentAt                : 7/14/2017 4:58:13 PM
                        ReceivedAt            : 7/14/2017 4:58:14 PM
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
                        FaultDomain:fd:/2 NodeName:_Node_2 NodeType:NodeType2 UpgradeDomain:2 UpgradeDomain: ud:/2 Deactivation Intent/Status: None/None
                        FaultDomain:fd:/1 NodeName:_Node_1 NodeType:NodeType1 UpgradeDomain:1 UpgradeDomain: ud:/1 Deactivation Intent/Status: None/None
                        
                        Existing Primary Replica -- Nodes with Partition's Existing Primary Replica or Secondary Replicas:
                        --
                        FaultDomain:fd:/0 NodeName:_Node_0 NodeType:NodeType0 UpgradeDomain:0 UpgradeDomain: ud:/0 Deactivation Intent/Status: None/None
                        
                        
                        RemoveWhenExpired     : True
                        IsExpired             : False
                        Transitions           : Error->Warning = 7/14/2017 4:56:14 PM, LastOk = 1/1/0001 12:00:00 AM

PS C:\> Get-ServiceFabricPartition fabric:/WordCount/WordCountService | select MinReplicaSetSize,TargetReplicaSetSize

MinReplicaSetSize TargetReplicaSetSize
----------------- --------------------
                2                    7                        

PS C:\> @(Get-ServiceFabricNode).Count
5
```

En el ejemplo siguiente se muestra el mantenimiento de una partición bloqueada en la reconfiguración debido a que el usuario no respeta el token de cancelación del método **RunAsync**. Investigar el informe de mantenimiento de cualquier réplica marcada como principal (P) puede ayudar a profundizar más en el problema.

```powershell
PS C:\utilities\ServiceFabricExplorer\ClientPackage\lib> Get-ServiceFabricPartitionHealth 0e40fd81-284d-4be4-a665-13bc5a6607ec -ExcludeHealthStatistics 


PartitionId           : 0e40fd81-284d-4be4-a665-13bc5a6607ec
AggregatedHealthState : Warning
UnhealthyEvaluations  : 
                        Unhealthy event: SourceId='System.FM', Property='State', HealthState='Warning', 
                        ConsiderWarningAsError=false.
                                               
HealthEvents          : 
                        SourceId              : System.FM
                        Property              : State
                        HealthState           : Warning
                        SequenceNumber        : 7
                        SentAt                : 8/27/2017 3:43:09 AM
                        ReceivedAt            : 8/27/2017 3:43:32 AM
                        TTL                   : Infinite
                        Description           : Partition reconfiguration is taking longer than expected.
                        fabric:/app/test1 3 1 0e40fd81-284d-4be4-a665-13bc5a6607ec
                          P/S Ready Node1 131482789658160654
                          S/P Ready Node2 131482789688598467
                          S/S Ready Node3 131482789688598468
                          (Showing 3 out of 3 replicas. Total available replicas: 3)                        
                        
                        For more information see: https://aka.ms/sfhealth
                        RemoveWhenExpired     : False
                        IsExpired             : False
                        Transitions           : Ok->Warning = 8/27/2017 3:43:32 AM, LastError = 1/1/0001 12:00:00 AM
```
Este informe muestra el estado de las réplicas de la partición en las que se está llevando a cabo la reconfiguración: 

```
  P/S Ready Node1 131482789658160654
  S/P Ready Node2 131482789688598467
  S/S Ready Node3 131482789688598468
```

Para cada réplica, el informe de mantenimiento contiene:
- Rol de configuración anterior
- Rol de configuración actual
- [Estado de réplica](service-fabric-concepts-replica-lifecycle.md)
- Nodo en el que se ejecuta la réplica
- Id. de réplica

En un caso similar al del ejemplo, es necesario seguir investigando. Investigue el mantenimiento de cada réplica individual comenzando con las réplicas marcadas como `Primary` y `Secondary` (131482789658160654 y 131482789688598467) en el ejemplo anterior.

### <a name="replica-constraint-violation"></a>Infracción de restricción de réplica
**System.PLB** notifica una advertencia si detecta una infracción de restricción de réplica y no se pueden colocar todas las réplicas de la partición. Los detalles del informe muestran qué restricciones y propiedades evitan la ubicación de las réplicas.

* **SourceId**: System.PLB
* **Propiedad**: Se inicia con **ReplicaConstraintViolation**.

## <a name="replica-system-health-reports"></a>Informes de mantenimiento del sistema de replica
**System.RA**, que representa el componente del agente de reconfiguración, es la autoridad para el estado de la réplica.

### <a name="state"></a>Estado
System.RA notifica Correcto cuando se ha creado la réplica.

* **SourceId**: System.RA
* **Propiedad**: Estado.

El ejemplo siguiente muestra una réplica correcta:

```powershell
PS C:\> Get-ServiceFabricPartition fabric:/WordCount/WordCountService | Get-ServiceFabricReplica | where {$_.ReplicaRole -eq "Primary"} | Get-ServiceFabricReplicaHealth

PartitionId           : af2e3e44-a8f8-45ac-9f31-4093eb897600
ReplicaId             : 131444422293118721
AggregatedHealthState : Ok
HealthEvents          : 
                        SourceId              : System.RA
                        Property              : State
                        HealthState           : Ok
                        SequenceNumber        : 131445248920273536
                        SentAt                : 7/14/2017 4:54:52 PM
                        ReceivedAt            : 7/14/2017 4:55:13 PM
                        TTL                   : Infinite
                        Description           : Replica has been created._Node_0
                        RemoveWhenExpired     : False
                        IsExpired             : False
                        Transitions           : Error->Ok = 7/14/2017 4:55:13 PM, LastWarning = 1/1/0001 12:00:00 AM
```

### <a name="replicaopenstatus-replicaclosestatus-replicachangerolestatus"></a>ReplicaOpenStatus, ReplicaCloseStatus, ReplicaChangeRoleStatus
Esta propiedad se usa para indicar advertencias o errores al intentar abrir una réplica, cerrar una réplica o realizar la transición de una réplica de un rol a otro. Para más información, consulte el [ciclo de vida de las réplicas](service-fabric-concepts-replica-lifecycle.md). Los errores podrían ser excepciones devueltas por las llamadas API o bloqueos del proceso de host de servicios durante este tiempo. Si hubiera errores debido a llamadas API del código C#, Service Fabric agregaría la excepción y el seguimiento de la pila al informe de mantenimiento.

Estas advertencias de estado se producen después de volver a intentar la acción localmente cierto número de veces (según la directiva). Service Fabric reintenta la acción hasta un umbral máximo. Cuando se alcanza el umbral máximo, puede intentar actuar para corregir la situación. Este intento puede provocar que estas advertencias se borren a medida que abandona la acción en este nodo. Por ejemplo, si una réplica no se puede abrir en un nodo, Service Fabric emite una advertencia de mantenimiento. Si la réplica continúa sin abrirse, Service Fabric actúa para repararla automáticamente. Esta acción podría implicar probar la misma operación en otro nodo. Este intento hace que se borre la advertencia generada para esta réplica. 

* **SourceId**: System.RA
* **Propiedad**: **ReplicaOpenStatus**, **ReplicaCloseStatus**, y **ReplicaChangeRoleStatus**.
* **Pasos siguientes**: Investigue el código del servicio o los volcados de memoria para determinar por qué se producen errores en la operación.

El siguiente ejemplo muestra el mantenimiento de una réplica que lanza `TargetInvocationException` desde su método abierto. La descripción contiene el punto de error (**IStatefulServiceReplica.Open**, el tipo de excepción **TargetInvocationException** y el seguimiento de la pila.

```powershell
PS C:\> Get-ServiceFabricReplicaHealth -PartitionId 337cf1df-6cab-4825-99a9-7595090c0b1b -ReplicaOrInstanceId 131483509874784794


PartitionId           : 337cf1df-6cab-4825-99a9-7595090c0b1b
ReplicaId             : 131483509874784794
AggregatedHealthState : Warning
UnhealthyEvaluations  : 
                        Unhealthy event: SourceId='System.RA', Property='ReplicaOpenStatus', HealthState='Warning', 
                        ConsiderWarningAsError=false.
                        
HealthEvents          : 
                        SourceId              : System.RA
                        Property              : ReplicaOpenStatus
                        HealthState           : Warning
                        SequenceNumber        : 131483510001453159
                        SentAt                : 8/27/2017 11:43:20 PM
                        ReceivedAt            : 8/27/2017 11:43:21 PM
                        TTL                   : Infinite
                        Description           : Replica had multiple failures during open on _Node_0 API call: IStatefulServiceReplica.Open(); Error = System.Reflection.TargetInvocationException (-2146232828)
Exception has been thrown by the target of an invocation.
   at Microsoft.ServiceFabric.Replicator.RecoveryManager.d__31.MoveNext()
--- End of stack trace from previous location where exception was thrown ---
   at System.Runtime.CompilerServices.TaskAwaiter.ThrowForNonSuccess(Task task)
   at System.Runtime.CompilerServices.TaskAwaiter.HandleNonSuccessAndDebuggerNotification(Task task)
   at Microsoft.ServiceFabric.Replicator.LoggingReplicator.d__137.MoveNext()
--- End of stack trace from previous location where exception was thrown ---
   at System.Runtime.CompilerServices.TaskAwaiter.ThrowForNonSuccess(Task task)
   at System.Runtime.CompilerServices.TaskAwaiter.HandleNonSuccessAndDebuggerNotification(Task task)
   at Microsoft.ServiceFabric.Replicator.DynamicStateManager.d__109.MoveNext()
--- End of stack trace from previous location where exception was thrown ---
   at System.Runtime.CompilerServices.TaskAwaiter.ThrowForNonSuccess(Task task)
   at System.Runtime.CompilerServices.TaskAwaiter.HandleNonSuccessAndDebuggerNotification(Task task)
   at Microsoft.ServiceFabric.Replicator.TransactionalReplicator.d__79.MoveNext()
--- End of stack trace from previous location where exception was thrown ---
   at System.Runtime.CompilerServices.TaskAwaiter.ThrowForNonSuccess(Task task)
   at System.Runtime.CompilerServices.TaskAwaiter.HandleNonSuccessAndDebuggerNotification(Task task)
   at Microsoft.ServiceFabric.Replicator.StatefulServiceReplica.d__21.MoveNext()
--- End of stack trace from previous location where exception was thrown ---
   at System.Runtime.CompilerServices.TaskAwaiter.ThrowForNonSuccess(Task task)
   at System.Runtime.CompilerServices.TaskAwaiter.HandleNonSuccessAndDebuggerNotification(Task task)
   at Microsoft.ServiceFabric.Services.Runtime.StatefulServiceReplicaAdapter.d__0.MoveNext()

    For more information see: https://aka.ms/sfhealth
                        RemoveWhenExpired     : False
                        IsExpired             : False
                        Transitions           : Error->Warning = 8/27/2017 11:43:21 PM, LastOk = 1/1/0001 12:00:00 AM                        
```

En el ejemplo siguiente se muestra una réplica que se bloquea constantemente durante el cierre:

```powershell
C:>Get-ServiceFabricReplicaHealth -PartitionId dcafb6b7-9446-425c-8b90-b3fdf3859e64 -ReplicaOrInstanceId 131483565548493142


PartitionId           : dcafb6b7-9446-425c-8b90-b3fdf3859e64
ReplicaId             : 131483565548493142
AggregatedHealthState : Warning
UnhealthyEvaluations  : 
                        Unhealthy event: SourceId='System.RA', Property='ReplicaCloseStatus', HealthState='Warning', 
                        ConsiderWarningAsError=false.
                        
HealthEvents          : 
                        SourceId              : System.RA
                        Property              : ReplicaCloseStatus
                        HealthState           : Warning
                        SequenceNumber        : 131483565611258984
                        SentAt                : 8/28/2017 1:16:01 AM
                        ReceivedAt            : 8/28/2017 1:16:03 AM
                        TTL                   : Infinite
                        Description           : Replica had multiple failures during close on _Node_1. The application 
                        host has crashed.
                        
                        For more information see: https://aka.ms/sfhealth
                        RemoveWhenExpired     : False
                        IsExpired             : False
                        Transitions           : Error->Warning = 8/28/2017 1:16:03 AM, LastOk = 1/1/0001 12:00:00 AM
```

### <a name="reconfiguration"></a>Reconfiguración
Esta propiedad se usa para indicar si una réplica que realiza una [reconfiguración](service-fabric-concepts-reconfiguration.md) detecta que la reconfiguración está detenida o bloqueada. Este informe de mantenimiento podría estar en la réplica que, actualmente, tiene el rol principal excepto en los casos de una reconfiguración principal de intercambio, donde puede estar en la réplica que va a disminuir de principal a secundaria activa.

La reconfiguración puede bloquearse por una de las razones siguientes:

- No se está completando una acción en la réplica local, la misma réplica que la que realiza la reconfiguración. En este caso, la investigación de los informes de mantenimiento sobre esta réplica de otros componentes, System.RAP o System.RE, puede proporcionar información adicional.

- No se está completando una acción en una réplica remota. Las réplicas cuyas acciones están pendientes se enumeran en el informe de mantenimiento. Es necesario investigar más los informes de estado para las réplicas remotas. También podría haber problemas de comunicación entre este nodo y el nodo remoto.

En raras ocasiones, la reconfiguración puede bloquearse debido a la comunicación u otros problemas entre este nodo y el servicio Administrador de conmutación por error.

* **SourceId**: System.RA
* **Propiedad**: Reconfiguración.
* **Pasos siguientes**: Investigar las réplicas locales o remotas según la descripción del informe de mantenimiento.

En el ejemplo siguiente se muestra un informe de mantenimiento donde una reconfiguración está bloqueada en la réplica local. En este ejemplo, se debe a un servicio que no respeta el token de cancelación.

```powershell
PS C:\> Get-ServiceFabricReplicaHealth -PartitionId 9a0cedee-464c-4603-abbc-1cf57c4454f3 -ReplicaOrInstanceId 131483600074836703


PartitionId           : 9a0cedee-464c-4603-abbc-1cf57c4454f3
ReplicaId             : 131483600074836703
AggregatedHealthState : Warning
UnhealthyEvaluations  : 
                        Unhealthy event: SourceId='System.RA', Property='Reconfiguration', HealthState='Warning', 
                        ConsiderWarningAsError=false.
                        
HealthEvents          : 
                        SourceId              : System.RA
                        Property              : Reconfiguration
                        HealthState           : Warning
                        SequenceNumber        : 131483600309264482
                        SentAt                : 8/28/2017 2:13:50 AM
                        ReceivedAt            : 8/28/2017 2:13:57 AM
                        TTL                   : Infinite
                        Description           : Reconfiguration is stuck. Waiting for response from the local replica
                        
                        For more information see: https://aka.ms/sfhealth
                        RemoveWhenExpired     : False
                        IsExpired             : False
                        Transitions           : Error->Warning = 8/28/2017 2:13:57 AM, LastOk = 1/1/0001 12:00:00 AM
```

El siguiente ejemplo muestra un informe de mantenimiento donde una reconfiguración está bloqueada a la espera de una respuesta de dos réplicas remotas. En este ejemplo, hay tres réplicas en la partición, incluida la principal actual. 

```Powershell
PS C:\> Get-ServiceFabricReplicaHealth -PartitionId  579d50c6-d670-4d25-af70-d706e4bc19a2 -ReplicaOrInstanceId 131483956274977415


PartitionId           : 579d50c6-d670-4d25-af70-d706e4bc19a2
ReplicaId             : 131483956274977415
AggregatedHealthState : Warning
UnhealthyEvaluations  : 
                        Unhealthy event: SourceId='System.RA', Property='Reconfiguration', HealthState='Warning', ConsiderWarningAsError=false.
                        
HealthEvents          : 
                        SourceId              : System.RA
                        Property              : Reconfiguration
                        HealthState           : Warning
                        SequenceNumber        : 131483960376212469
                        SentAt                : 8/28/2017 12:13:57 PM
                        ReceivedAt            : 8/28/2017 12:14:07 PM
                        TTL                   : Infinite
                        Description           : Reconfiguration is stuck. Waiting for response from 2 replicas
                        
                        Pending Replicas: 
                        P/I Down 40 131483956244554282
                        S/S Down 20 131483956274972403
                        
                        For more information see: https://aka.ms/sfhealth
                        RemoveWhenExpired     : False
                        IsExpired             : False
                        Transitions           : Error->Warning = 8/28/2017 12:07:37 PM, LastOk = 1/1/0001 12:00:00 AM
```

Este informe de mantenimiento muestra que la reconfiguración está bloqueada a la espera de respuesta de dos réplicas: 

```
    P/I Down 40 131483956244554282
    S/S Down 20 131483956274972403
```

Para cada réplica, se proporciona la siguiente información:
- Rol de configuración anterior
- Rol de configuración actual
- [Estado de réplica](service-fabric-concepts-replica-lifecycle.md)
- Id. de nodo
- Id. de réplica

Para desbloquear la reconfiguración:
- Es necesario activar las réplicas **inactivas**. 
- Las réplicas **inbuild** deben completar la compilación y la transición a preparadas.

### <a name="slow-service-api-call"></a>Llamada a la API de servicio lenta
**System.RAP** y **System.Replicator** notifican una advertencia si una llamada al código de servicio de usuario tarda más tiempo del configurado. La advertencia se borra cuando finaliza la llamada.

* **SourceId**: System.RAP o System.Replicator
* **Propiedad**: El nombre de la API lenta. La descripción proporciona más detalles sobre el tiempo que la API ha estado pendiente.
* **Pasos siguientes**: Investigue por qué la llamada tarda más de lo esperado.

El ejemplo siguiente muestra el evento de mantenimiento de System.RAP para un servicio confiable que no respeta el token de cancelación de **RunAsync**:

```powershell
PS C:\> Get-ServiceFabricReplicaHealth -PartitionId 5f6060fb-096f-45e4-8c3d-c26444d8dd10 -ReplicaOrInstanceId 131483966141404693


PartitionId           : 5f6060fb-096f-45e4-8c3d-c26444d8dd10
ReplicaId             : 131483966141404693
AggregatedHealthState : Warning
UnhealthyEvaluations  : 
                        Unhealthy event: SourceId='System.RA', Property='Reconfiguration', HealthState='Warning', ConsiderWarningAsError=false.
                        
HealthEvents          :                         
                        SourceId              : System.RAP
                        Property              : IStatefulServiceReplica.ChangeRole(S)Duration
                        HealthState           : Warning
                        SequenceNumber        : 131483966663476570
                        SentAt                : 8/28/2017 12:24:26 PM
                        ReceivedAt            : 8/28/2017 12:24:56 PM
                        TTL                   : Infinite
                        Description           : The api IStatefulServiceReplica.ChangeRole(S) on _Node_1 is stuck. Start Time (UTC): 2017-08-28 12:23:56.347.
                        RemoveWhenExpired     : False
                        IsExpired             : False
                        Transitions           : Error->Warning = 8/28/2017 12:24:56 PM, LastOk = 1/1/0001 12:00:00 AM
                        
```

La propiedad y el texto indican qué API se ha bloqueado. Los pasos siguientes que se deben realizar son diferentes para las diferentes API bloqueadas. Cualquier API de *IStatefulServiceReplica* o *IStatelessServiceInstance* suele ser un error en el código del servicio. En la siguiente sección se describe cómo se trasladan estas API al [modelo Reliable Services](service-fabric-reliable-services-lifecycle.md):

- **IStatefulServiceReplica.Open**: Esta advertencia indica que una llamada a `CreateServiceInstanceListeners`, `ICommunicationListener.OpenAsync`, o si se reemplaza, `OnOpenAsync` está bloqueada.

- **IStatefulServiceReplica.Close** y **IStatefulServiceReplica.Abort**: El caso más común es un servicio que no respeta el token de cancelación pasado a `RunAsync`. También podría ser que `ICommunicationListener.CloseAsync` o, si se invalida, `OnCloseAsync` sufra un bloqueo.

- **IStatefulServiceReplica.ChangeRole (S)** y **istatefulservicereplica.changerole (n)**: El caso más común es un servicio que no respeta el token de cancelación pasado a `RunAsync`. En este escenario, la mejor solución es reiniciar la réplica.

- **IStatefulServiceReplica.ChangeRole(P)**: El caso más común es que el servicio no ha devuelto una tarea de `RunAsync`.

Otras llamadas API que se pueden bloquear están en la interfaz **IReplicator**. Por ejemplo: 

- **IReplicator.CatchupReplicaSet**: Esta advertencia indica una de estas dos cosas. No hay suficientes réplicas activas. Para ver si este es el caso, examine el estado de réplica de las réplicas de la partición, o el informe de mantenimiento de System.FM para ver si encuentra una reconfiguración bloqueada. También es posible que las réplicas no estén confirmando las operaciones. El cmdlet de PowerShell `Get-ServiceFabricDeployedReplicaDetail` se puede utilizar para determinar el progreso de todas las réplicas. El problema radica en las réplicas cuyo valor de `LastAppliedReplicationSequenceNumber` está detrás del valor de `CommittedSequenceNumber` de la principal.

- **IReplicator.BuildReplica (\<ReplicaId remoto >)**: Esta advertencia indica un problema en el proceso de compilación. Para más información, consulte el [ciclo de vida de las réplicas](service-fabric-concepts-replica-lifecycle.md). Puede ser debido a una configuración incorrecta de la dirección del replicador. Para más información, consulte [Configurar Reliable Services con estado](service-fabric-reliable-services-configuration.md) y [Especificar recursos en un manifiesto de servicio](service-fabric-service-manifest-resources.md). También podría ser un problema en el nodo remoto.

### <a name="replicator-system-health-reports"></a>Informes de mantenimiento del sistema del replicador
**Cola de replicación completa:**
**System.Replicator** notifica una advertencia cuando la cola de replicación está llena. En la réplica principal, la cola de replicación suele llenarse porque una o varias réplicas secundarias son lentas a la hora de confirmar las operaciones. En la secundaria, esto suele ocurrir cuando el servicio es lento en aplicar las operaciones. La advertencia se borra cuando la cola ya no está llena.

* **SourceId**: System.Replicator
* **Propiedad**: **PrimaryReplicationQueueStatus** o **SecondaryReplicationQueueStatus**, según el rol de réplica.
* **Pasos siguientes**: Si el informe está en el servidor principal, compruebe la conexión entre los nodos del clúster. Si todas las conexiones son correctas, puede haber al menos una réplica secundaria lenta con una elevada latencia de disco para aplicar las operaciones. Si el informe se encuentra en la réplica secundaria, compruebe primero el uso de disco y el rendimiento en el nodo. A continuación, compruebe la conexión saliente desde el nodo lento a la réplica principal.

**RemoteReplicatorConnectionStatus:**
**System.Replicator** en la réplica principal notifica una advertencia cuando la conexión a un replicador secundario (remoto) no es correcta. La dirección del replicador remoto se muestra en el mensaje del informe, lo que permite detectar más fácilmente si se pasó una configuración incorrecta o si existen problemas de red entre los replicadores.

* **SourceId**: System.Replicator
* **Propiedad**: **RemoteReplicatorConnectionStatus**.
* **Pasos siguientes**: Compruebe el mensaje de error y asegúrese de que la dirección del replicador remoto está configurada correctamente. Por ejemplo, si se abre el replicador remoto con la dirección de escucha "localhost", no es accesible desde el exterior. Si la dirección parece correcta, compruebe la conexión entre el nodo principal y la dirección remota para ver si hay posibles problemas de red.

### <a name="replication-queue-full"></a>Cola de replicación completa
**System.Replicator** notifica una advertencia cuando la cola de replicación está llena. En la réplica principal, la cola de replicación suele llenarse porque una o varias réplicas secundarias son lentas a la hora de confirmar las operaciones. En la secundaria, esto suele ocurrir cuando el servicio es lento en aplicar las operaciones. La advertencia se borra cuando la cola ya no está llena.

* **SourceId**: System.Replicator
* **Propiedad**: **PrimaryReplicationQueueStatus** o **SecondaryReplicationQueueStatus**, según el rol de réplica.

### <a name="slow-naming-operations"></a>Operaciones de nomenclatura lentas
**System.NamingService** informa del mantenimiento en su réplica principal cuando una operación de nomenclatura tarda más de lo aceptable. [CreateServiceAsync](https://docs.microsoft.com/dotnet/api/system.fabric.fabricclient.servicemanagementclient.createserviceasync) o [DeleteServiceAsync](https://docs.microsoft.com/dotnet/api/system.fabric.fabricclient.servicemanagementclient.deleteserviceasync) son ejemplos de operaciones de nomenclatura. En FabricClient pueden encontrarse más métodos. Por ejemplo, pueden encontrarse en [métodos de administración de servicios](https://docs.microsoft.com/dotnet/api/system.fabric.fabricclient.servicemanagementclient) o [métodos de administración de propiedades](https://docs.microsoft.com/dotnet/api/system.fabric.fabricclient.propertymanagementclient).

> [!NOTE]
> El servicio de nomenclatura resuelve los nombres de servicio a una ubicación en el clúster. Los usuarios pueden usarlo para administrar propiedades y nombres de servicio. Se trata de un servicio persistente con particiones de Service Fabric. Una de las particiones representa a *Authority Owner*, que contiene metadatos sobre todos los nombres y servicios de Service Fabric. Los nombres de Service Fabric se asignan a particiones diferentes, denominadas particiones *Name Owner*, por lo que el servicio se puede ampliar. Aprenda más sobre el [servicio de nomenclatura](service-fabric-architecture.md).
> 
> 

Cuando una operación de nomenclatura tarda más de lo esperado, la operación se marca con un informe de advertencia en la réplica principal de la partición del servicio de nomenclatura que se usa para la operación. Si la operación se completa correctamente, la advertencia se elimina. Si la operación se completa con un error, el informe de estado incluye detalles sobre el error.

* **SourceId**: System.NamingService
* **Propiedad**: Comienza con el prefijo "**Duration_**" e identifica la operación lenta y el nombre de Service Fabric que se aplica la operación. Por ejemplo, si el servicio de creación en el nombre **fabric:/MyApp/MyService** tarda demasiado, la propiedad es **Duration_AOCreateService.fabric:/MyApp/MyService**. "AO" apunta al rol de la partición de nomenclatura para este nombre y operación.
* **Pasos siguientes**: Compruebe por qué se produce un error en la operación de nomenclatura. Cada operación puede tener diferentes causas. Por ejemplo, puede que esté bloqueado el servicio de eliminación. El servicio de eliminación puede estar bloqueado debido a que el host de la aplicación se bloquee continuamente en un nodo por un error de usuario en el código del servicio.

El ejemplo siguiente muestra una operación de servicio de creación. La operación tardó más de la duración configurada. "AO" vuelve a intentarlo y envía trabajo a "NO." "NO" completó la última operación con TIMEOUT. En este caso, la misma réplica es la principal tanto para el rol de "AO" como para el de "NO".

```powershell
PartitionId           : 00000000-0000-0000-0000-000000001000
ReplicaId             : 131064359253133577
AggregatedHealthState : Warning
UnhealthyEvaluations  :
                        Unhealthy event: SourceId='System.NamingService', Property='Duration_AOCreateService.fabric:/MyApp/MyService', HealthState='Warning', ConsiderWarningAsError=false.

HealthEvents          :
                        SourceId              : System.RA
                        Property              : State
                        HealthState           : Ok
                        SequenceNumber        : 131064359308715535
                        SentAt                : 4/29/2016 8:38:50 PM
                        ReceivedAt            : 4/29/2016 8:39:08 PM
                        TTL                   : Infinite
                        Description           : Replica has been created.
                        RemoveWhenExpired     : False
                        IsExpired             : False
                        Transitions           : Error->Ok = 4/29/2016 8:39:08 PM, LastWarning = 1/1/0001 12:00:00 AM

                        SourceId              : System.NamingService
                        Property              : Duration_AOCreateService.fabric:/MyApp/MyService
                        HealthState           : Warning
                        SequenceNumber        : 131064359526778775
                        SentAt                : 4/29/2016 8:39:12 PM
                        ReceivedAt            : 4/29/2016 8:39:38 PM
                        TTL                   : 00:05:00
                        Description           : The AOCreateService started at 2016-04-29 20:39:08.677 is taking longer than 30.000.
                        RemoveWhenExpired     : True
                        IsExpired             : False
                        Transitions           : Error->Warning = 4/29/2016 8:39:38 PM, LastOk = 1/1/0001 12:00:00 AM

                        SourceId              : System.NamingService
                        Property              : Duration_NOCreateService.fabric:/MyApp/MyService
                        HealthState           : Warning
                        SequenceNumber        : 131064360657607311
                        SentAt                : 4/29/2016 8:41:05 PM
                        ReceivedAt            : 4/29/2016 8:41:08 PM
                        TTL                   : 00:00:15
                        Description           : The NOCreateService started at 2016-04-29 20:39:08.689 completed with FABRIC_E_TIMEOUT in more than 30.000.
                        RemoveWhenExpired     : True
                        IsExpired             : False
                        Transitions           : Error->Warning = 4/29/2016 8:39:38 PM, LastOk = 1/1/0001 12:00:00 AM
```

## <a name="deployedapplication-system-health-reports"></a>Informes de mantenimiento del sistema DeployedApplication
**System.Hosting** es la autoridad en las entidades implementadas.

### <a name="activation"></a>Activación
System.Hosting notifica un estado Correcto cuando una aplicación se ha activado correctamente en el nodo. De lo contrario, notifica un error.

* **SourceId**: System.Hosting
* **Propiedad**: **Activación**, incluida la versión de lanzamiento.
* **Pasos siguientes**: Si la aplicación está en mal estada, investigue el motivo del error de la activación.

El ejemplo siguiente muestra una activación correcta:

```powershell
PS C:\> Get-ServiceFabricDeployedApplicationHealth -NodeName _Node_1 -ApplicationName fabric:/WordCount -ExcludeHealthStatistics

ApplicationName                    : fabric:/WordCount
NodeName                           : _Node_1
AggregatedHealthState              : Ok
DeployedServicePackageHealthStates : 
                                     ServiceManifestName   : WordCountServicePkg
                                     ServicePackageActivationId : 
                                     NodeName              : _Node_1
                                     AggregatedHealthState : Ok
                                     
HealthEvents                       : 
                                     SourceId              : System.Hosting
                                     Property              : Activation
                                     HealthState           : Ok
                                     SequenceNumber        : 131445249083836329
                                     SentAt                : 7/14/2017 4:55:08 PM
                                     ReceivedAt            : 7/14/2017 4:55:14 PM
                                     TTL                   : Infinite
                                     Description           : The application was activated successfully.
                                     RemoveWhenExpired     : False
                                     IsExpired             : False
                                     Transitions           : Error->Ok = 7/14/2017 4:55:14 PM, LastWarning = 1/1/0001 12:00:00 AM
```

### <a name="download"></a>Descargar
System.Hosting notifica un error si la descarga del paquete de aplicación no se pudo realizar.

* **SourceId**: System.Hosting
* **Propiedad**: **Descargar**, incluida la versión de lanzamiento.
* **Pasos siguientes**: Investigue el motivo del error en el nodo de la descarga.

## <a name="deployedservicepackage-system-health-reports"></a>Informes de mantenimiento del sistema DeployedServicePackage
**System.Hosting** es la autoridad en las entidades implementadas.

### <a name="service-package-activation"></a>Activación de paquete de servicio
System.Hosting notifica un estado Correcto si la activación del paquete de servicio en el nodo se ha realizado correctamente. De lo contrario, notifica un error.

* **SourceId**: System.Hosting
* **Propiedad**: Activación.
* **Pasos siguientes**: Investigue el motivo del error de la activación.

### <a name="code-package-activation"></a>Activación del paquete de código
System.Hosting notifica un estado Correcto para cada paquete de código si la activación se ha realizado correctamente. Si se produce un error en la activación, notifica una advertencia tal y como está configurado. Si **CodePackage** no se puede activar o finaliza con un error mayor que el configurado **CodePackageHealthErrorThreshold**, el hospedaje notifica un error. Si hay varios paquetes de código en un paquete de servicios, se genera un informe de activación para cada uno.

* **SourceId**: System.Hosting
* **Propiedad**: Usa el prefijo **CodePackageActivation** y contiene el nombre del paquete de código y el punto de entrada como *CodePackageActivation:CodePackageName:SetupEntryPoint / EntryPoint*. Por ejemplo, **CodePackageActivation:Code:SetupEntryPoint**.

### <a name="service-type-registration"></a>Registro del tipo de servicio
System.Hosting notifica un estado Correcto si el tipo de servicio se ha registrado correctamente. Notifica un error si el registro no se ha realizado a tiempo tal y como se ha configurado mediante **ServiceTypeRegistrationTimeout**. Si se cierra el tiempo de ejecución, el tipo de servicio no está registrado en el nodo y Hosting notifica una advertencia.

* **SourceId**: System.Hosting
* **Propiedad**: Usa el prefijo **ServiceTypeRegistration** y contiene el nombre de tipo de servicio. Por ejemplo, **ServiceTypeRegistration:FileStoreServiceType**.

El ejemplo siguiente muestra un paquete de servicio implementado con mantenimiento correcto:

```powershell
PS C:\> Get-ServiceFabricDeployedServicePackageHealth -NodeName _Node_1 -ApplicationName fabric:/WordCount -ServiceManifestName WordCountServicePkg


ApplicationName            : fabric:/WordCount
ServiceManifestName        : WordCountServicePkg
ServicePackageActivationId : 
NodeName                   : _Node_1
AggregatedHealthState      : Ok
HealthEvents               : 
                             SourceId              : System.Hosting
                             Property              : Activation
                             HealthState           : Ok
                             SequenceNumber        : 131445249084026346
                             SentAt                : 7/14/2017 4:55:08 PM
                             ReceivedAt            : 7/14/2017 4:55:14 PM
                             TTL                   : Infinite
                             Description           : The ServicePackage was activated successfully.
                             RemoveWhenExpired     : False
                             IsExpired             : False
                             Transitions           : Error->Ok = 7/14/2017 4:55:14 PM, LastWarning = 1/1/0001 12:00:00 AM
                             
                             SourceId              : System.Hosting
                             Property              : CodePackageActivation:Code:EntryPoint
                             HealthState           : Ok
                             SequenceNumber        : 131445249084306362
                             SentAt                : 7/14/2017 4:55:08 PM
                             ReceivedAt            : 7/14/2017 4:55:14 PM
                             TTL                   : Infinite
                             Description           : The CodePackage was activated successfully.
                             RemoveWhenExpired     : False
                             IsExpired             : False
                             Transitions           : Error->Ok = 7/14/2017 4:55:14 PM, LastWarning = 1/1/0001 12:00:00 AM
                             
                             SourceId              : System.Hosting
                             Property              : ServiceTypeRegistration:WordCountServiceType
                             HealthState           : Ok
                             SequenceNumber        : 131445249088096842
                             SentAt                : 7/14/2017 4:55:08 PM
                             ReceivedAt            : 7/14/2017 4:55:14 PM
                             TTL                   : Infinite
                             Description           : The ServiceType was registered successfully.
                             RemoveWhenExpired     : False
                             IsExpired             : False
                             Transitions           : Error->Ok = 7/14/2017 4:55:14 PM, LastWarning = 1/1/0001 12:00:00 AM
```

### <a name="download"></a>Descargar
System.Hosting notifica un error si la descarga del paquete de servicio no se pudo realizar.

* **SourceId**: System.Hosting
* **Propiedad**: **Descargar**, incluida la versión de lanzamiento.
* **Pasos siguientes**: Investigue el motivo del error en el nodo de la descarga.

### <a name="upgrade-validation"></a>Validación de actualización
System.Hosting notifica un error si se produce un error de validación durante la actualización o si se produce un error de actualización en el nodo.

* **SourceId**: System.Hosting
* **Propiedad**: Usa el prefijo **FabricUpgradeValidation** y contiene la versión de actualización.
* **Descripción**: Señala el error encontrado.

### <a name="undefined-node-capacity-for-resource-governance-metrics"></a>Capacidad de nodo sin definir de las métricas de gobernanza de recursos
Si las capacidades de nodo no se definen en el manifiesto de clúster y se desactiva la configuración de la detección automática, System.Hosting se encarga de mostrar un aviso. Asimismo, Service Fabric mostrará un aviso de mantenimiento cada vez que el paquete de servicio que usa la [gobernanza de recursos](service-fabric-resource-governance.md) se registre en un nodo específico.

* **SourceId**: System.Hosting
* **Propiedad**: **ResourceGovernance**.
* **Pasos siguientes**: La mejor manera de solucionar este problema es cambiar el manifiesto de clúster para habilitar la detección automática de los recursos disponibles. Otra manera de solucionar esto es actualizar el manifiesto de clúster con las capacidades de nodo de estas métricas especificadas correctamente.

## <a name="next-steps"></a>Pasos siguientes
* [Vista de los informes de estado de Service Fabric](service-fabric-view-entities-aggregated-health.md)

* [Notificación y comprobación del estado del servicio](service-fabric-diagnostics-how-to-report-and-check-service-health.md)

* [Supervisión y diagnóstico de los servicios localmente](service-fabric-diagnostics-how-to-monitor-and-diagnose-services-locally.md)

* [Actualización de la aplicación de Service Fabric](service-fabric-application-upgrade.md)

