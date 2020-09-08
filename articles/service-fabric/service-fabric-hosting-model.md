---
title: Modelo de hospedaje de Service Fabric
description: Describe la relación entre las réplicas (o instancias) de un servicio implementado de Service Fabric y el proceso de host de servicio.
author: harahma
ms.topic: conceptual
ms.date: 04/15/2017
ms.author: harahma
ms.custom: devx-track-csharp
ms.openlocfilehash: 2e14995b92e99e1a9695f81fb71bcab6dd62303a
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89011674"
---
# <a name="azure-service-fabric-hosting-model"></a>Modelo de hospedaje de Service Fabric
En este artículo se ofrece información general sobre los modelos de hospedaje de aplicaciones que Azure Service Fabric proporciona y, además, se describen las diferencias entre los modelos de **proceso compartido** y **proceso exclusivo**. Se describe el aspecto de una aplicación implementada en un nodo de Service Fabric y la relación entre las réplicas (o instancias) del servicio y el proceso de host de servicio.

Antes de continuar, asegúrese de comprender los distintos conceptos y relaciones que se explican en [Modelar una aplicación en Service Fabric][a1]. 

> [!NOTE]
> En este artículo, a menos que se mencione explícitamente que significa algo distinto:
>
> - La palabra *réplica* hace referencia a una réplica de un servicio con estado y a una instancia de un servicio sin estado.
> - *CodePackage* recibe un tratamiento equivalente al proceso de *ServiceHost* que registra una instancia de *ServiceType* y hospeda réplicas de servicios de dicha instancia de *ServiceType*.
>

Para entender el modelo de hospedaje, analizaremos un ejemplo. Supongamos que el valor de *ApplicationType* es "MyAppType", cuyo valor de *ServiceType* es "MyServiceType". "MyServiceType" lo proporciona *ServicePackage* '"MyServicePackage", cuyo valor de *CodePackage*  es "MyCodePackage". "MyCodePackage" registra *ServiceType*  con "MyServiceType" cuando se ejecuta.

Por ejemplo, hay un clúster de tres nodos y se crea una *aplicación* **fabric:/App1** de tipo "MyAppType". Dentro de la aplicación **fabric:/App1**, se crea un servicio **fabric:/App1/ServiceA** de tipo "MyServiceType". Este servicio tiene dos particiones (por ejemplo, **P1** y **P2**) y tres réplicas por partición. En el siguiente diagrama se muestra la vista de esta aplicación tal y como se termina implementando en un nodo.


![Diagrama de la vista del nodo de la aplicación implementada][node-view-one]


Service Fabric activó "MyServicePackage", que inició "MyCodePackage", donde se hospedan las réplicas de ambas particiones. Todos los nodos del clúster tienen la misma vista, porque se eligió un número de réplicas por partición igual al número de nodos del clúster. Crearemos otro servicio, **fabric:/App1/ServiceB**, en la aplicación **fabric:/App1**. Este servicio tiene una partición (por ejemplo, **P3**) y tres réplicas por partición. En el diagrama siguiente se muestra la nueva vista del nodo:


![Diagrama de la vista del nodo de la aplicación implementada][node-view-two]


Service Fabric colocó la nueva réplica de la partición **P3** del servicio **fabric:/App1/ServiceB** en la activación existente de "MyServicePackage". Ahora, crearemos otra aplicación **fabric:/App2** del tipo "MyAppType". Dentro de **fabric:/App2**, cree un servicio **fabric:/App2/ServiceA**. Este servicio tiene dos particiones (**P4** y **P5**) y tres réplicas por partición. En el diagrama siguiente se muestra la nueva vista del nodo:


![Diagrama de la vista del nodo de la aplicación implementada][node-view-three]


Service Fabric activa una nueva copia de "MyServicePackage", que inicia una nueva copia de "MyCodePackage". Las réplicas de ambas particiones del servicio **fabric:/App2/ServiceA** (**P4** y **P5**) se colocan en esta nueva copia "MyCodePackage".

## <a name="shared-process-model"></a>Modelo de proceso compartido
En la sección anterior se describe el modelo de hospedaje predeterminado que proporciona Service Fabric, que se conoce como el modelo de proceso compartido. En este modelo, para una aplicación determinada, solo hay una copia de un *ServicePackage* determinado activada en un nodo (que inicia todos los *CodePackages* que contiene). Todas las réplicas de todos los servicios de un *ServiceType* determinado se colocan en el *CodePackage* que registra ese *ServiceType*. En otras palabras, todas las réplicas de todos los servicios en un nodo de un determinado *ServiceType* comparten el mismo proceso.

## <a name="exclusive-process-model"></a>Modelo de proceso exclusivo
El otro modelo de hospedaje proporcionado por Service Fabric es el modelo de proceso exclusivo. En este modelo, en un nodo determinado, cada réplica reside en su propio proceso dedicado. Service Fabric activa una copia nueva de *ServicePackage* (que inicia todos los *CodePackages* que contiene). Las réplicas se colocan en el *CodePackage* que registró el *ServiceType* del servicio al que pertenece la réplica. 

Si usa la versión 5.6 o una versión posterior de Service Fabric, puede elegir el modelo de proceso exclusivo en el momento de crear un servicio (mediante [PowerShell][p1], [REST][r1] o [FabricClient][c1]). Especifique **ServicePackageActivationMode** como "ExclusiveProcess".

```powershell
PS C:\>New-ServiceFabricService -ApplicationName "fabric:/App1" -ServiceName "fabric:/App1/ServiceA" -ServiceTypeName "MyServiceType" -Stateless -PartitionSchemeSingleton -InstanceCount -1 -ServicePackageActivationMode "ExclusiveProcess"
```

```csharp
var serviceDescription = new StatelessServiceDescription
{
    ApplicationName = new Uri("fabric:/App1"),
    ServiceName = new Uri("fabric:/App1/ServiceA"),
    ServiceTypeName = "MyServiceType",
    PartitionSchemeDescription = new SingletonPartitionSchemeDescription(),
    InstanceCount = -1,
    ServicePackageActivationMode = ServicePackageActivationMode.ExclusiveProcess
};

var fabricClient = new FabricClient(clusterEndpoints);
await fabricClient.ServiceManager.CreateServiceAsync(serviceDescription);
```

Si tiene un servicio predeterminado en el manifiesto de aplicación, puede elegir el modelo de proceso exclusivo si especifica el atributo **ServicePackageActivationMode**:

```xml
<DefaultServices>
  <Service Name="MyService" ServicePackageActivationMode="ExclusiveProcess">
    <StatelessService ServiceTypeName="MyServiceType" InstanceCount="1">
      <SingletonPartition/>
    </StatelessService>
  </Service>
</DefaultServices>
```
Crearemos otro servicio, **fabric:/App1/ServiceC**, en la aplicación **fabric:/App1**. Este servicio tiene dos particiones (por ejemplo, **P6** y **P7**) y tres réplicas por partición. Establezca **ServicePackageActivationMode** en "ExclusiveProcess". En el diagrama siguiente se muestra la nueva vista del nodo:


![Diagrama de la vista del nodo de la aplicación implementada][node-view-four]


Como se puede observar, Service Fabric activó dos copias nuevas de "MyServicePackage" (una para cada réplica de la partición **P6** y **P7**). Service Fabric colocó cada réplica en su copia dedicada de *CodePackage*. Cuando se usa el modelo de proceso exclusivo, en una aplicación determinada, puede haber varias copias de un *ServicePackage* determinado activas en un nodo. En el ejemplo anterior, hay tres copias de "MyServicePackage" activas para **fabric:/App1**. Cada una de estas copias activas de "MyServicePackage" tiene asociado un **ServicePackageActivationId**. Este id. identifica esa copia dentro de la aplicación **fabric:/App1**.

Cuando solo se usa el modelo de proceso compartido para una aplicación, solo hay una copia activa de *ServicePackage* en un nodo. El **ServicePackageActivationId** correspondiente a esta activación de *ServicePackage* es una cadena vacía. Por ejemplo, esto ocurre con **fabric:/App2**.

> [!NOTE]
>- La correspondencia del modelo de hospedaje de proceso compartido para **ServicePackageActivationMode** es igual a **SharedProcess**. Se trata del modelo de hospedaje predeterminado y no es necesario especificar **ServicePackageActivationMode** en el momento en que se crea el servicio.
>
>- La correspondencia del modelo de hospedaje de proceso exclusivo para **ServicePackageActivationMode** es igual a **ExclusiveProcess**. Para usar esta configuración, debe especificarla explícitamente en el momento de crear el servicio. 
>
>- Para ver el modelo de hospedaje de un servicio, consulte la [descripción del servicio][p2] y observe el valor de **ServicePackageActivationMode**.
>
>

## <a name="work-with-a-deployed-service-package"></a>Trabajo con un paquete de servicio implementado
Se hace referencia a una copia activa de *ServicePackage* en un nodo como [paquete de servicio implementado][p3]. Cuando usa el modelo de proceso exclusivo para crear servicios, en una aplicación determinada, podrían existir varios paquetes de servicio implementados para el mismo *ServicePackage*. Si realiza operaciones específicas para un paquete de servicio implementado, debe proporcionar **ServicePackageActivationId** para identificar un paquete de servicio implementado específico. Por ejemplo, proporcione el identificador si [informa sobre el estado de un paquete de servicio implementado][p4] o [reinicia el paquete de código de un paquete de servicio implementado][p5].

Puede averiguar el valor de **ServicePackageActivationId** de un paquete de servicio implementado si consulta la lista de [paquetes de servicio implementados][p3] en un nodo. Cuando consulta los [tipos de servicios implementados][p6], las [réplicas implementadas][p7] y los [paquetes de código implementados][p8] de un nodo, el resultado de consulta también contiene el valor de **ServicePackageActivationId** del paquete de servicio implementado primario.

> [!NOTE]
>- En el modelo de hospedaje de proceso compartido de un nodo concreto para una aplicación determinada, solo se activa una copia de *ServicePackage*. Aquí, **ServicePackageActivationId** es igual al valor de *cadena vacía*, y no debe especificarse al realizar las operaciones relacionadas con el paquete de servicio implementado. 
>
> - En el modelo de hospedaje de proceso exclusivo de un nodo concreto para una aplicación determinada, pueden activarse una o varias copias de *ServicePackage*. Cada activación tiene un valor *no vacío* para **ServicePackageActivationId**, que se especifica al realizar operaciones relacionadas con el paquete de servicio implementado. 
>
> - Si **ServicePackageActivationId** se omite, su valor predeterminado es *cadena vacía*. Si hay un paquete de servicio implementado activado en el modelo de proceso compartido, la operación se realizará en él. De lo contrario, la operación presentará un error.
>
> - No debe consultar el **ServicePackageActivationId** una vez y almacenarlo en caché. El identificador se genera de manera dinámica y puede cambiar por diversos motivos. Antes de realizar una operación que necesita **ServicePackageActivationId**, primero debe consultar la lista de [paquetes de servicio implementados][p3] en un nodo. Luego, use el **ServicePackageActivationId** del resultado de la consulta para realizar la operación original.
>
>

## <a name="guest-executable-and-container-applications"></a>Aplicaciones de contenedor y de archivo ejecutable invitado
Service Fabric trata las aplicaciones de [archivo ejecutable invitado][a2] y de [contenedor][a3] como servicios sin estado, que son independientes. No hay runtime de Service Fabric en *ServiceHost* (un proceso o contenedor). Como se trata de servicios independientes, el número de réplicas por *ServiceHost* no es aplicable para estos servicios. La configuración más común que se usa con estos servicios es la partición única con [InstanceCount][c2] igual a -1 (una copia del código de servicio que se ejecuta en cada nodo del clúster). 

El valor predeterminado de **ServicePackageActivationMode** para estos servicios es **SharedProcess**, en cuyo caso, Service Fabric solo activa una copia de *ServicePackage* en un nodo para una aplicación determinada.  Esto significa que solo una copia del código de servicio ejecutará un nodo. Si desea que varias copias del código de servicio se ejecute en un nodo, especifique **ServicePackageActivationMode** como **ExclusiveProcess** en el momento de crear el servicio. Por ejemplo, puede hacerlo cuando crea varios servicio (*Service1* a *ServiceN*) de *ServiceType* (especificado en *ServiceManifest*) o cuando el servicio se divide en varias particiones. 

## <a name="change-the-hosting-model-of-an-existing-service"></a>Cambio del modelo de hospedaje de un servicio existente
En este momento, no puede cambiar el modelo de hospedaje de un servicio existente desde proceso compartido a proceso exclusivo (o viceversa).

## <a name="choose-between-the-hosting-models"></a>Elección entre los modelos de hospedaje
Debe evaluar el modelo de hospedaje que mejor se adapta a los requisitos. El modelo de proceso compartido usa mejor los recursos del sistema operativo, porque se generan menos procesos y varias réplicas del mismo proceso pueden compartir puertos. Sin embargo, si una de las réplicas presenta un error donde debe interrumpir el host de servicio, se afectarán todas las réplicas del mismo proceso.

 El modelo de proceso exclusivo proporciona un mejor aislamiento, en el que cada réplica está en su propio proceso. Si una de las réplicas presenta un error, no afecta a las demás réplicas. Este modelo resulta útil en casos donde el protocolo de comunicación no admite el uso compartido de puertos. Facilita la capacidad de aplicar la gobernanza de recursos a nivel de réplica. Sin embargo, el proceso exclusivo consume más recursos del sistema operativo, debido a que genera un proceso para cada réplica en el nodo.

## <a name="exclusive-process-model-and-application-model-considerations"></a>Consideraciones sobre el modelo de proceso exclusivo y el modelo de aplicación
En el caso de la mayoría de las aplicaciones, puede modelar la aplicación en Service Fabric si mantiene un *ServiceType* por *ServicePackage*. 

En determinados casos, Service Fabric también permite más de un *ServiceType* por *ServicePackage* (y un *CodePackage* puede registrar más de un  *ServiceType*). A continuación se presentan algunos escenarios en que esta configuración puede resultar útil:

- Desea optimizar el uso de recursos generando menos procesos y con una mayor densidad de réplica por proceso.
- Las réplicas de diferentes instancias de *ServiceTypes* deben compartir algunos datos comunes, lo que genera un alto costo de inicialización o memoria.
- Dispone de una oferta de servicio gratuita y desea establecer un límite en el uso de recursos mediante la colocación de todas las réplicas del servicio en el mismo proceso.

El modelo de hospedaje de proceso exclusivo no es coherente con el modelo de aplicación que tiene varias instancias de *ServiceTypes* por cada *ServicePackage*. Esto se debe a que la disponibilidad de varios *ServiceTypes* por *ServicePackage* está pensada para conseguir un mayor nivel de uso compartido de recursos entre réplicas y, además, ofrece mayor densidad de réplica por proceso. El modelo de proceso exclusivo está diseñado para alcanzar distintos resultados.

Pensemos en un caso en hay varios *ServiceTypes* por *ServicePackage* con distintos *CodePackage* que registran cada *ServiceType*. Por ejemplo, existe un *ServicePackage* "MultiTypeServicePackage" que tiene dos *CodePackages*:

- "MyCodePackageA", que registra el *ServiceType* "MyServiceTypeA".
- "MyCodePackageB", que registra el *ServiceType* "MyServiceTypeB".

Ahora, supongamos que creamos una aplicación, **fabric:/SpecialApp**. Dentro de **fabric:/SpecialApp**, creamos los dos servicios siguientes con el modelo de proceso exclusivo:

- El servicio **fabric:/SpecialApp/ServiceA** de tipo "MyServiceTypeA", con dos particiones (por ejemplo, **P1** y **P2**) y tres réplicas por partición.
- El servicio **fabric:/SpecialApp/ServiceB** de tipo "MyServiceTypeB", con dos particiones (**P3** y **P4**) y tres réplicas por partición.

En un nodo determinado, ambos servicios tienen dos réplicas cada uno. Como se usó el modelo de proceso exclusivo para crear los servicios, Service Fabric activa una copia nueva de "MyServicePackage" de cada réplica. Cada activación de "MultiTypeServicePackage" inicia una copia de "MyCodePackageA" y "MyCodePackageB". Sin embargo, solo "MyCodePackageA" o "MyCodePackageB" hospeda la réplica para la que se activó "MultiTypeServicePackage". En el diagrama siguiente se muestra la vista del nodo:


![Diagrama de la vista del nodo de la aplicación implementada][node-view-five]


En la activación de "MultiTypeServicePackage" para la réplica de la partición **P1** del servicio **fabric:/SpecialApp/ServiceA**, "MyCodePackageA" hospeda la réplica. "MyCodePackageB" está en ejecución. De manera similar, en la activación de "MultiTypeServicePackage" para la réplica de la partición **P3** del servicio **fabric:/SpecialApp/ServiceB**, "MyCodePackageB" hospeda la réplica. "MyCodePackageA" está en ejecución. Por tanto, mientras mayor sea el número de *CodePackages* (con el registro de diferentes *ServiceTypes*) por *ServicePackage*, mayor será el uso de recursos redundantes. 
 
 Sin embargo, si creamos los servicios **fabric:/SpecialApp/ServiceA** y **fabric:/SpecialApp/ServiceB** con el modelo de proceso compartido, Service Fabric activa solo una copia de "MultiTypeServicePackage" para la aplicación **fabric:/SpecialApp**. "MyCodePackageA" hospeda todas las réplicas del servicio **fabric:/SpecialApp/ServiceA**. "MyCodePackageB" hospeda todas las réplicas del servicio **fabric:/SpecialApp/ServiceB**. En el siguiente diagrama se muestra la vista del nodo con esta configuración: 


![Diagrama de la vista del nodo de la aplicación implementada][node-view-six]


Según el ejemplo anterior, es posible que se plantee que, si "MyCodePackageA" registra "MyServiceTypeA" y "MyServiceTypeB" y no hay ninguna instancia de "MyCodePackageB", entonces no se produce ninguna ejecución redundante de *CodePackage*. Si bien esto es correcto, este modelo de aplicación no es compatible con el modelo de hospedaje de proceso exclusivo. Si el objetivo consiste en colocar cada réplica en su propio proceso dedicado, no es necesario registrar ambos *ServiceTypes* del mismo *CodePackage*. En su lugar, simplemente se coloca cada *ServiceType* en su propio *ServicePackage*.

### <a name="reliable-services-and-actor-forking-subprocesses"></a>Subprocesos de bifurcación de Reliable Services y Actors

Service Fabric no admite Reliable Services y, por consiguiente, los subprocesos de bifurcación de Reliable Actors. Un ejemplo de por qué no se admite es que [CodePackageActivationContext](/dotnet/api/system.fabric.codepackageactivationcontext?view=azure-dotnet) no se puede usar para registrar un subproceso no admitido, y los tokens de cancelación solo se envían a procesos registrados; como consecuencia, se produce todo tipo de problemas, como errores de actualización, cuando los subprocesos no se cierran después de que el proceso principal ha recibido un token de cancelación.

## <a name="next-steps"></a>Pasos siguientes
[Empaquete una aplicación][a4] y prepárela para la implementación.

[Implementar y quitar aplicaciones][a5]. En este artículo se describe cómo usar PowerShell para administrar las instancias de aplicaciones.

<!--Image references-->
[node-view-one]: ./media/service-fabric-hosting-model/node-view-one.png
[node-view-two]: ./media/service-fabric-hosting-model/node-view-two.png
[node-view-three]: ./media/service-fabric-hosting-model/node-view-three.png
[node-view-four]: ./media/service-fabric-hosting-model/node-view-four.png
[node-view-five]: ./media/service-fabric-hosting-model/node-view-five.png
[node-view-six]: ./media/service-fabric-hosting-model/node-view-six.png

<!--Link references--In actual articles, you only need a single period before the slash-->
[a1]: service-fabric-application-model.md
[a2]: service-fabric-guest-executables-introduction.md
[a3]: service-fabric-containers-overview.md
[a4]: service-fabric-package-apps.md
[a5]: service-fabric-deploy-remove-applications.md

[r1]: /rest/api/servicefabric/sfclient-api-createservice

[c1]: /dotnet/api/system.fabric.fabricclient.servicemanagementclient.createserviceasync
[c2]: /dotnet/api/system.fabric.description.statelessservicedescription.instancecount

[p1]: /powershell/module/servicefabric/new-servicefabricservice
[p2]: /powershell/module/servicefabric/get-servicefabricservicedescription
[p3]: /powershell/module/servicefabric/get-servicefabricdeployedservicepackage
[p4]: /powershell/module/servicefabric/send-servicefabricdeployedservicepackagehealthreport
[p5]: /powershell/module/servicefabric/restart-servicefabricdeployedcodepackage
[p6]: /powershell/module/servicefabric/get-servicefabricdeployedservicetype
[p7]: /powershell/module/servicefabric/get-servicefabricdeployedreplica
[p8]: /powershell/module/servicefabric/get-servicefabricdeployedcodepackage
