---
title: Incorporación de informes de mantenimiento de Service Fabric personalizados
description: Describe cómo enviar informes de estado personalizados a entidades de estado de Azure Service Fabric. Proporciona recomendaciones para diseñar e implementar informes de estado de calidad.
author: georgewallace
ms.topic: conceptual
ms.date: 2/28/2018
ms.author: gwallace
ms.custom: devx-track-csharp
ms.openlocfilehash: 6df434610a8f595ecca7f16e31f8a302373b02f9
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89012660"
---
# <a name="add-custom-service-fabric-health-reports"></a>Incorporación de informes de mantenimiento de Service Fabric personalizados
Azure Service Fabric presenta un [modelo de estado](service-fabric-health-introduction.md) diseñado para marcar las condiciones poco favorables del clúster o de la aplicación en entidades específicas. El modelo de mantenimiento utiliza **informadores de estado** (componentes del sistema y guardianes). El objetivo es obtener un diagnóstico y reparación sencillo y rápido. Los escritores de servicio deben pensar por adelantado sobre el estado. Cualquier condición que pueda afectar al estado debe notificarse, sobre todo si puede ayudar a marcar la causa raíz de los problemas. La información de estado puede ahorrar tiempo y esfuerzo en la depuración y la investigación. La utilidad resulta especialmente clara una vez que el servicio está en funcionamiento a escala en la nube (privada o Azure).

Los generadores de informes de Service Fabric identificaron las condiciones de interés. Informan sobre esas condiciones en función de su vista local. El [almacén de estado](service-fabric-health-introduction.md#health-store) agrega datos de mantenimiento que todos los informadores envían para determinar si el estado de las entidades es bueno en general. La intención es que el modelo sea completo, flexible y fácil de usar. La calidad de los informes de mantenimiento determina la precisión de la vista de estado del clúster. Los falsos positivos que muestran erróneamente problemas pueden afectar negativamente a las actualizaciones u otros servicios que usan datos de mantenimiento. Ejemplos de estos servicios son los servicios de reparación y los mecanismos de alerta. Por lo tanto, se necesita cierto grado de reflexión para proporcionar informes que capturen condiciones que resulten interesantes de la mejor manera posible.

Para diseñar e implementar informes de mantenimiento, los guardianes y componentes del sistema deben:

* Definir la condición que le interese, la forma en que se supervisa y la repercusión sobre la funcionalidad del clúster o de la aplicación. Según esta información, decidir sobre la propiedad de informe de mantenimiento y el estado de mantenimiento.
* Determinar la [entidad](service-fabric-health-introduction.md#health-entities-and-hierarchy) a la que se aplica el informe.
* Determinar dónde se generan los informes, desde el servicio o desde un guardián interno o externo.
* Definir un origen utilizado para identificar el generador de informes.
* Elegir una estrategia de generación de informes, periódicamente o en las transiciones. La manera recomendada es periódicamente, ya que el código necesario es más simple y menos propenso a errores.
* Determinar cuánto tiempo debe permanecer en el almacén de estado y cómo debe borrarse el informe de condiciones incorrectas. Con esta información, decidir el tiempo de duración del informe y el comportamiento de eliminación cuando caduca.

Tal y como se ha mencionado, la creación de informes puede realizarse desde:

* La réplica de Service Fabric supervisado.
* Guardianes internos implementados como un servicio de Service Fabric (por ejemplo, un servicio sin estado de Service Fabric que supervisa las condiciones y los informes de problemas). Los guardianes pueden implementarse un todos los nodos o pueden afinarse en el servicio supervisado.
* Guardianes internos que se ejecutan en los nodos de Service Fabric pero que *no* se implementan como servicios de Service Fabric.
* Guardianes externos que sondean el recurso desde *fuera* del clúster de Service Fabric (por ejemplo, un servicio de supervisión de tipo Gomez).

> [!NOTE]
> De fábrica, el clúster está rellenado con informes de estado enviados por los componentes del sistema. Lea más en [Uso de informes de mantenimiento del sistema para solucionar problemas](service-fabric-understand-and-troubleshoot-with-system-health-reports.md). Los informes de usuario deben enviarse en [entidades de estado](service-fabric-health-introduction.md#health-entities-and-hierarchy) ya creadas por el sistema.
> 
> 

Una vez que el diseño de informes de mantenimiento está vacío, los informes de mantenimiento se pueden enviar de forma fácil. Puede usar [FabricClient](/dotnet/api/system.fabric.fabricclient) para informar del mantenimiento si el clúster no es [seguro](service-fabric-cluster-security.md) o si el cliente de Fabric tiene privilegios de administrador. La notificación se puede hacer a través de la API mediante [FabricClient.HealthManager.ReportHealth](/dotnet/api/system.fabric.fabricclient.healthclient.reporthealth), PowerShell o REST. Los botones de configuración procesan los informes por lotes para un mejor rendimiento.

> [!NOTE]
> El informe de mantenimiento es sincrónico y solo representa el trabajo de validación en el cliente. El hecho de que el cliente de mantenimiento o los objetos `Partition` o `CodePackageActivationContext` acepten el informe no significa que se aplique en el almacén. Se envía de forma asincrónica y posiblemente por lotes con otros informes. El procesamiento en el servidor todavía puede seguir dando error: el número de secuencia puede estar obsoleto, la entidad en la que se debe aplicar el informe ha sido eliminada, etc.
> 
> 

## <a name="health-client"></a>Cliente de mantenimiento
Los informes de mantenimiento se envían al administrador de mantenimiento por medio de un cliente de mantenimiento, que reside en el cliente de Fabric. El administrador de mantenimiento guarda los informes en el almacén de estado. El cliente de mantenimiento puede configurarse con las opciones siguientes:

* **HealthReportSendInterval**: el retraso entre el momento en que el informe se agrega al cliente y el momento en que se envía al administrador de mantenimiento. Se usa para procesar por lotes los informes en un único mensaje, en lugar de enviar un mensaje para cada informe. El procesamiento por lotes mejora el rendimiento. Valor predeterminado: 30 segundos.
* **HealthReportRetrySendInterval**: el intervalo en el que el cliente de mantenimiento reenvía los informes de mantenimiento acumulados al administrador de mantenimiento. Valor predeterminado: 30 segundos, mínimo: 1 segundo.
* **HealthOperationTimeout**: el período de tiempo de expiración de un mensaje de informe enviado al administrador de mantenimiento. Si un mensaje supera el tiempo de espera, el cliente de mantenimiento lo sigue intentando hasta que el administrador de mantenimiento confirme que el informe se ha procesado. Valor predeterminado: dos minutos.

> [!NOTE]
> Cuando los informes se procesan por lotes, el cliente de Fabric se debe mantener activo durante al menos el valor de HealthReportSendInterval para tener la seguridad de que se envían. Si el mensaje se pierde o el administrador de mantenimiento no es capaz de aplicarlos debido a errores transitorios, el cliente de Fabric debe mantenerse activo más tiempo para darle una oportunidad de volver a intentarlo.
> 
> 

El almacenamiento en búfer en el cliente toma en consideración el carácter único de los informes. Por ejemplo, si un informador incorrecto determinado notifica 100 informes por segundo en la misma propiedad de la misma entidad, los informes se reemplazan por la versión más reciente. A lo sumo, existirá un informe de este tipo en la cola de cliente. Si se configura el procesamiento por lotes, el número de informes que se envían al administrador de mantenimiento es simplemente uno por intervalo de envío. Este informe es el último informe agregado, que refleja el estado más reciente de la entidad.
Especifique los parámetros de configuración al crear `FabricClient`, pasando [FabricClientSettings](/dotnet/api/system.fabric.fabricclientsettings) con los valores deseados para las entradas relacionadas con el mantenimiento.

En el siguiente ejemplo se crea un cliente de Fabric y se especifica que se deben enviar los informes cuando se agregan. En tiempos de espera y errores que se pueden reintentar, los reintentos se producen cada 40 segundos.

```csharp
var clientSettings = new FabricClientSettings()
{
    HealthOperationTimeout = TimeSpan.FromSeconds(120),
    HealthReportSendInterval = TimeSpan.FromSeconds(0),
    HealthReportRetrySendInterval = TimeSpan.FromSeconds(40),
};
var fabricClient = new FabricClient(clientSettings);
```

Se recomienda mantener la configuración predeterminada del cliente de Fabric, que establece `HealthReportSendInterval` en 30 segundos. Esta configuración garantiza un rendimiento óptimo debido al procesamiento por lotes. Para los informes importantes que se deban enviar tan pronto como sea posible, utilice `HealthReportSendOptions` con la marca Immediate establecida en `true` en la API [FabricClient.HealthClient.ReportHealth](/dotnet/api/system.fabric.fabricclient.healthclient.reporthealth). Los informes inmediatos omiten el intervalo de procesamiento por lotes. Utilice esta marca con cuidado; queremos aprovechar el procesamiento por lotes del cliente de mantenimiento siempre que sea posible. El envío inmediato también es útil cuando se está cerrando el cliente Fabric (por ejemplo, el proceso ha determinado que el estado no es válido y debe cerrarse para evitar efectos secundarios). Garantiza el envío más eficaz de los informes acumulados. Cuando se agrega un informe con la marca Immediate, el cliente de mantenimiento envía por lotes todos los informes acumulados desde el último envío.

Se pueden especificar los mismos parámetros cuando se crea una conexión a un clúster mediante Powershell. En el ejemplo siguiente se inicia una conexión con un clúster local:

```powershell
PS C:\> Connect-ServiceFabricCluster -HealthOperationTimeoutInSec 120 -HealthReportSendIntervalInSec 0 -HealthReportRetrySendIntervalInSec 40
True

ConnectionEndpoint   :
FabricClientSettings : {
                       ClientFriendlyName                   : PowerShell-1944858a-4c6d-465f-89c7-9021c12ac0bb
                       PartitionLocationCacheLimit          : 100000
                       PartitionLocationCacheBucketCount    : 1024
                       ServiceChangePollInterval            : 00:02:00
                       ConnectionInitializationTimeout      : 00:00:02
                       KeepAliveInterval                    : 00:00:20
                       HealthOperationTimeout               : 00:02:00
                       HealthReportSendInterval             : 00:00:00
                       HealthReportRetrySendInterval        : 00:00:40
                       NotificationGatewayConnectionTimeout : 00:00:00
                       NotificationCacheUpdateTimeout       : 00:00:00
                       }
GatewayInformation   : {
                       NodeAddress                          : localhost:19000
                       NodeId                               : 1880ec88a3187766a6da323399721f53
                       NodeInstanceId                       : 130729063464981219
                       NodeName                             : Node.1
                       }
```

De forma similar a la API, los informes pueden enviarse con la marca `-Immediate` para ser enviados inmediatamente, con independencia del valor `HealthReportSendInterval`.

Para REST, los informes se envían a la puerta de enlace de Service Fabric, que tiene un cliente Fabric interno. De forma predeterminada, este cliente se configura para enviar informes por lotes cada 30 segundos. Puede cambiar el intervalo de envío de lotes con la opción de configuración de clúster `HttpGatewayHealthReportSendInterval` en `HttpGateway`. Como se mencionó antes, una mejor opción consiste en enviar los informes con `Immediate` establecido en true. 

> [!NOTE]
> Para asegurarse de que los servicios no autorizados no puedan notificar el estado en las entidades del clúster, configure el servidor para que acepte solamente solicitudes de clientes protegidos. El objeto `FabricClient` utilizado para la creación de informes debe tener habilitada la seguridad para poder comunicarse con el clúster (por ejemplo, con la autenticación Kerberos o de certificados). Obtenga más información sobre la [seguridad del clúster](service-fabric-cluster-security.md).
> 
> 

## <a name="report-from-within-low-privilege-services"></a>Informes desde servicios con pocos privilegios
Si los servicios de Service Fabric no tienen acceso de administrador al clúster, puede informar sobre el estado de las entidades desde el contexto actual mediante `Partition` o `CodePackageActivationContext`.

* Para los servicios sin estado, use [IStatelessServicePartition.ReportInstanceHealth](/dotnet/api/system.fabric.istatelessservicepartition.reportinstancehealth) para informar sobre la instancia de servicio actual.
* Para los servicios con estado, use [IStatefulServicePartition.ReportReplicaHealth](/dotnet/api/system.fabric.istatefulservicepartition.reportreplicahealth) para informar sobre la réplica actual.
* Use [IServicePartition.ReportPartitionHealth](/dotnet/api/system.fabric.iservicepartition.reportpartitionhealth) para informar sobre la entidad de partición actual.
* Use [CodePackageActivationContext.ReportApplicationHealth](/dotnet/api/system.fabric.codepackageactivationcontext.reportapplicationhealth) para informar sobre la aplicación actual.
* Use [CodePackageActivationContext.ReportDeployedApplicationHealth](/dotnet/api/system.fabric.codepackageactivationcontext.reportdeployedapplicationhealth) para informar sobre la aplicación actual implementada en el nodo actual.
* Use [CodePackageActivationContext.ReportDeployedServicePackageHealth](/dotnet/api/system.fabric.codepackageactivationcontext.reportdeployedservicepackagehealth) para informar sobre un paquete de servicio para la aplicación implementada en el nodo actual.

> [!NOTE]
> Internamente, `Partition` y `CodePackageActivationContext` contienen un cliente de mantenimiento configurado con los valores predeterminados. Como se explica para el [cliente de mantenimiento](service-fabric-report-health.md#health-client), los informes se envían por lotes según un temporizador. Los objetos deben estar activos para que tener ocasión de enviar el informe.
> 
> 

Puede especificar `HealthReportSendOptions` al enviar informes a través de las API de mantenimiento `Partition` y `CodePackageActivationContext`. Si tiene informes importantes que deben enviarse tan pronto como sea posible, use `HealthReportSendOptions` con la marca Inmediate establecida en `true`. Los informes inmediatos omiten el intervalo de procesamiento por lotes del cliente de mantenimiento interno. Como se mencionó anteriormente, use esta marca con cuidado: queremos aprovechar el procesamiento por lotes del cliente de mantenimiento siempre que sea posible.

## <a name="design-health-reporting"></a>Informes de estado del diseño
El primer paso en la generación de informes de alta calidad es identificar las condiciones que pueden afectar al estado del servicio. Cualquier condición que puede ayudar a indicar los problemas en el servicio o el clúster cuando se inicia, o mejor aún, antes de que ocurran, puede suponer un ahorro potencial de miles de millones de dólares. Las ventajas incluyen menos tiempo de inactividad, menos noches dedicadas a investigar y reparar problemas y una mayor satisfacción del cliente.

Una vez identificadas las condiciones, los escritores de guardianes deben averiguar cuál es la mejor forma de supervisarlas para obtener un mejor equilibrio entre sobrecarga y utilidad. Por ejemplo, considere un servicio que realiza cálculos complejos mediante el uso de algunos archivos temporales en un recurso compartido. Un guardián podría supervisar el recurso compartido para asegurarse de que hay suficiente espacio disponible. Podría escuchar las notificaciones de cambios en un archivo o un directorio. Podría notificar una advertencia si se alcanza un umbral anticipadamente, y un error si el recurso compartido está lleno. Con una advertencia, un sistema de reparación podría iniciar la limpieza de los archivos antiguos en el recurso compartido. En caso de error, un sistema de reparación podría mover la réplica de servicio a otro nodo. Observe cómo los estados de la condición se describen en lo relativo al mantenimiento: el estado de la condición que se puede considerar correcto (válido) o incorrecto (advertencia o error).

Una vez que se establecen los detalles de supervisión, el escritor del guardián debe determinar cómo implementarlo. Si las condiciones se pueden determinar desde dentro del servicio, el guardián puede formar parte del propio servicio supervisado. Por ejemplo, el código del servicio puede comprobar el uso del recurso compartido y luego notificarlo cada vez que intente escribir un archivo. La ventaja de este enfoque es que la generación de informes es sencilla. Debe tener cuidado para evitar que los errores de vigilancia afecten a la funcionalidad del servicio.

La generación de informes desde dentro del servicio supervisado no es siempre una opción. Puede que un guardián dentro del servicio no detecte las condiciones. Puede que no tenga la lógica o los datos para realizar la determinación. La sobrecarga de las condiciones de supervisión puede ser alta. Además, las condiciones pueden no ser específicas de un servicio, sino que afecten a las interacciones entre servicios. Otra opción es tener los guardianes del clúster como procesos independientes. Los guardianes supervisan las condiciones y el informe, sin que afecte a los servicios principales de ninguna manera. Por ejemplo, estos guardianes se podrían implementar como servicios sin estado en la misma aplicación, en todos los nodos o en los mismos nodos que el servicio.

En ocasiones, un guardián que se ejecuta en el clúster tampoco es una opción. Si la condición supervisada es la disponibilidad o la funcionalidad del servicio que ven los usuarios, es mejor tener los guardianes en el mismo lugar que los clientes de usuario. Allí, pueden probar las operaciones tal y como las llaman los usuarios. Por ejemplo, puede tener un guardián que resida fuera del clúster, emita las solicitudes al servicio y compruebe la latencia y si el resultado es correcto. (Para un servicio de calculadora, por ejemplo, ¿2 + 2 devuelve 4 en un período de tiempo razonable?)

Cuando los detalles del guardián hayan finalizado, debe determinar un identificador de origen que lo identifique de forma única. Si en el clúster residen varios guardianes del mismo tipo, deben informar en entidades diferentes o, si lo hacen en la misma entidad, usar un identificador de origen o propiedad distintos. De este modo, los informes podrán coexistir. La propiedad del informe de estado debe capturar la condición supervisada. (En el ejemplo anterior, la propiedad puede ser **ShareSize**). Si se aplican varios informes a la misma condición, la propiedad debe contener cierta información dinámica para permitir que los informes coexistan. Por ejemplo, si hay varios recursos compartidos que deben supervisarse, el nombre de propiedad puede ser **ShareSize-sharename**.

> [!NOTE]
> *No* use el almacén de estado para mantener la información de estado. Solo la información relacionada con el mantenimiento se debe notificar como estado, dado que esta información afecta a la evaluación del mantenimiento de una entidad. El almacén de estado no se diseñó como almacén de uso general. Usa una lógica de evaluación de estado para agregar todos los datos al estado de mantenimiento. El envío de información no relacionada con el estado (por ejemplo, el informe del estado con un estado de mantenimiento de correcto) no afecta al estado de mantenimiento agregado pero puede afectar negativamente al rendimiento del almacén de estado.
> 
> 

El siguiente punto de decisión trata de en qué entidad informar. La mayor parte de las veces la condición identifica claramente la entidad. Elija la entidad con la mejor granularidad posible. Si una condición afecta a todas las réplicas de una partición, haga un informe sobre la partición, no sobre el servicio. Hay casos excepcionales en los que es necesario dedicar más esfuerzo. Si la condición afecta a una entidad (por ejemplo, a una réplica) pero lo que se quiere es marcar la condición durante un período superior al de la duración de la réplica, se debe notificar en la partición. De lo contrario, cuando se elimina la réplica, el almacén de estado limpia todos sus informes. Los escritores guardianes deben tener en cuenta la duración de la entidad y del informe. Debe quedar claro cuándo se debe limpiar un informe de un almacén (por ejemplo, cuándo deja de aplicarse un error notificado en una entidad).

Veamos un ejemplo que reúne los puntos descritos. Considere una aplicación de Service Fabric compuesta de un servicio persistente con estado maestro y varios servicios sin estado secundario implementados en todos los nodos (un tipo de servicio secundario para cada tipo de tarea). El servicio maestro tiene una cola de procesamiento con comandos que ejecutarán los servicios secundarios. Estos últimos ejecutan las solicitudes entrantes y devuelven señales de confirmación. Una condición que puede supervisarse es la longitud de la cola de procesamiento maestra. Si la longitud de la cola maestra alcanza un umbral, se devuelve una advertencia La advertencia que indica que los servicios secundarios no pueden controlar la carga. Si la cola alcanza la longitud máxima y se quitan los comandos, se notifica un error, ya que el servicio no se puede recuperar. Los informes pueden estar en la propiedad **QueueStatus**. El guardián reside dentro del servicio y se envía periódicamente en la réplica principal maestra. El período de vida es de dos minutos y se envía periódicamente cada 30 segundos. Si el principal deja de funcionar, el informe se limpia automáticamente del almacén. Si la réplica de servicio está activa, pero se ha interbloqueado o tiene otros problemas, el informe expira en el Almacén de estado. En este caso, la entidad se evalúa con error.

Otra condición que se pueden supervisar es el tiempo de ejecución de las tareas. El servicio maestro distribuye las tareas a los servicios secundarios según su tipo. Dependiendo del diseño, el maestro podría sondear los servicios secundarios para averiguar el estado de la tarea. También podría esperar a que los servicios secundarios envíen señales de confirmación cuando hayan terminado. En el segundo caso, se debe tener cuidado de detectar situaciones en las que los servicios secundarios mueren o se pierden los mensajes. Una opción es que el servicio maestro envíe una solicitud de ping al mismo servicio secundario, que devuelve su estado. Si no se recibe ningún estado, el maestro lo considera un error y reprograma la tarea. Este comportamiento asume que las tareas son idempotentes.

Se puede traducir la condición supervisada como Advertencia si la tarea no se realiza en un tiempo determinado (**t1**, por ejemplo, 10 minutos). Si la tarea no se completó a tiempo (**t2**, por ejemplo, 20 minutos), se puede traducir la condición supervisada como Error. Estos informes pueden realizarse de varias maneras:

* La réplica principal maestra informa periódicamente sobre sí misma. Puede tener una propiedad para todas las tareas pendientes en la cola. Si al menos una de las tareas tarda más, el estado del informe en la propiedad **PendingTasks** es una advertencia o un error, según corresponda. Si no hay ninguna tarea pendiente o todas acaban de empezar su ejecución, el estado del informe es correcto. Las tareas son persistentes. Si la principal deja de funcionar, la nueva tarea principal promocionada puede seguir informando correctamente.
* Otro proceso guardián (en la nube o externo) comprueba las tareas (desde fuera, en función del resultado deseado de la tarea) para ver si se han completado. Si no respetan los umbrales, se envía un informe en el servicio principal. También se envía un informe en cada tarea que incluye el identificador de la tarea, como **PendingTask+taskId**. Los informes solo se deben enviar con estados incorrectos. Establezca el período de vida en unos minutos y marque los informes para ser eliminados cuando caduquen para garantizar la limpieza.
* El servicio secundario que ejecuta una tarea informa cuando tarda más tiempo del esperado en ejecutarse. Informa en la instancia de servicio en la propiedad **PendingTasks**. Este informe indica la instancia de servicio que tiene problemas pero no captura la situación en la que se bloquea la instancia. Los informes se limpian en ese momento. Podría informar sobre el servicio secundario. Si el servicio secundario completa la tarea, la instancia secundaria borra el informe del almacén. El informe no captura la situación en la que se pierde el mensaje de confirmación y la tarea no se considera terminada desde el punto de vista del maestro.

Sin embargo, los informes se crean en los casos que se han descrito anteriormente y se capturan en el mantenimiento de la aplicación cuando se evalúe el mantenimiento.

## <a name="report-periodically-vs-on-transition"></a>Informar periódicamente frente a en transiciones
Con el modelo de informes de mantenimiento, los guardianes pueden enviar informes periódicamente o en las transiciones. Se recomienda que los informes guardián sean periódicos, ya que el código es mucho más sencillo y menos propenso a errores. Los guardianes deben esforzarse por ser los más simples posible para evitar errores que desencadenen informes incorrectos. Los informes de *mal estado* incorrectos afectarán a los escenarios y las evaluaciones de mantenimiento en función del estado, como las actualizaciones. Los informes de *buen estado* incorrectos ocultan problemas en el clúster, lo cual no es deseable.

Para los informes periódicos, el guardián puede implementarse con un temporizador. En la devolución de llamada del temporizador, el guardián puede comprobar el estado y enviar un informe según el estado actual. No es necesario comprobar qué informe se ha enviado anteriormente ni realizar ninguna optimización en términos de mensajería. El cliente de mantenimiento tiene una lógica de procesamiento por lotes para ayudarle con el rendimiento. Mientras el cliente de mantenimiento se mantenga activo, vuelve a intentarlo internamente hasta que el informe sea confirmado por parte del Almacén de estado o que el guardián genere un informe más reciente con la misma entidad, propiedad y origen.

Los informes en las transiciones requieren un tratamiento especial del estado. El guardián supervisa algunas condiciones y solo informa cuando cambian las condiciones. La parte positiva de este enfoque es que se necesitan menos informes. La parte negativa es que la lógica del guardián es compleja. El guardián debe mantener las condiciones o los informes, de modo que se puedan inspeccionar para determinar los cambios de estado. En la conmutación por error, tenga cuidado con los informes agregados que aún no se hayan enviado al almacén de estado. El número de secuencia debe ser creciente. Si no es así, los informes se rechazan como obsoletos. En los casos poco frecuentes en los que se produce la pérdida de datos, puede que sea necesario realizar la sincronización entre el estado del informador y el estado del almacén de estado.

La creación de informes sobre las transiciones tiene sentido para los servicios que crean informes sobre sí mismos, mediante `Partition` o `CodePackageActivationContext`. Cuando se quita el objeto local (paquete de servicio de réplica o implementado / aplicación implementada), también se quitan todos sus informes. Esta limpieza automática reduce la necesidad de sincronización entre el informador y el Almacén de estado. Si el informe es para la partición o la aplicación primaria, debe tener cuidado en la conmutación por error para evitar informes obsoletos en el almacén de estado. Debe agregarse lógica para mantener el estado correcto y borrar el informe del almacén cuando ya no sea necesario.

## <a name="implement-health-reporting"></a>Implementación de la generación de informes de estado
Una vez que los detalles de entidades e informes están claros, el envío de informes de estado puede realizarse a través de API, Powershell o REST.

### <a name="api"></a>API
Para informar a través de API, debe crear un informe de mantenimiento específico del tipo de entidad sobre la que desea informar. Entregue el informe a un cliente de mantenimiento. Como alternativa, cree información de mantenimiento y pásela para corregir los métodos de creación de informes en `Partition` o `CodePackageActivationContext` a fin de informar sobre entidades actuales.

El ejemplo siguiente muestra informes periódicos de un guardián dentro del clúster. El guardián comprueba si se puede acceder a un recurso externo desde dentro de un nodo. El recurso es necesario para un manifiesto de servicio de dentro de la aplicación. Si el recurso no está disponible, los demás servicios de dentro de la aplicación pueden seguir funcionando correctamente. Por lo tanto, el informe se envía en la entidad del paquete de servicio implementado cada 30 segundos.

```csharp
private static Uri ApplicationName = new Uri("fabric:/WordCount");
private static string ServiceManifestName = "WordCount.Service";
private static string NodeName = FabricRuntime.GetNodeContext().NodeName;
private static Timer ReportTimer = new Timer(new TimerCallback(SendReport), null, 30 * 1000, 30 * 1000);
private static FabricClient Client = new FabricClient(new FabricClientSettings() { HealthReportSendInterval = TimeSpan.FromSeconds(0) });

public static void SendReport(object obj)
{
    // Test whether the resource can be accessed from the node
    HealthState healthState = this.TestConnectivityToExternalResource();

    // Send report on deployed service package, as the connectivity is needed by the specific service manifest
    // and can be different on different nodes
    var deployedServicePackageHealthReport = new DeployedServicePackageHealthReport(
        ApplicationName,
        ServiceManifestName,
        NodeName,
        new HealthInformation("ExternalSourceWatcher", "Connectivity", healthState));

    // TODO: handle exception. Code omitted for snippet brevity.
    // Possible exceptions: FabricException with error codes
    // FabricHealthStaleReport (non-retryable, the report is already queued on the health client),
    // FabricHealthMaxReportsReached (retryable; user should retry with exponential delay until the report is accepted).
    Client.HealthManager.ReportHealth(deployedServicePackageHealthReport);
}
```

### <a name="powershell"></a>PowerShell
Envíe informes de mantenimiento con **Send-ServiceFabric *EntityType* HealthReport**.

El ejemplo siguiente muestra informes periódicos sobre valores de la CPU en un nodo. Los informes se deben enviar cada 30 segundos, y tienen un período de vida de dos minutos. Si caducan, significa que el informador tiene problemas, por lo que el nodo se ha evaluado con error. Cuando la CPU está por encima de un umbral, el informe tiene un estado de advertencia. Cuando la CPU se mantiene por encima de un umbral un tiempo superior al configurado, se notifica como error. De lo contrario, el informador envía un estado correcto.

```powershell
PS C:\> Send-ServiceFabricNodeHealthReport -NodeName Node.1 -HealthState Warning -SourceId PowershellWatcher -HealthProperty CPU -Description "CPU is above 80% threshold" -TimeToLiveSec 120

PS C:\> Get-ServiceFabricNodeHealth -NodeName Node.1
NodeName              : Node.1
AggregatedHealthState : Warning
UnhealthyEvaluations  :
                        Unhealthy event: SourceId='PowershellWatcher', Property='CPU', HealthState='Warning', ConsiderWarningAsError=false.

HealthEvents          :
                        SourceId              : System.FM
                        Property              : State
                        HealthState           : Ok
                        SequenceNumber        : 5
                        SentAt                : 4/21/2015 8:01:17 AM
                        ReceivedAt            : 4/21/2015 8:02:12 AM
                        TTL                   : Infinite
                        Description           : Fabric node is up.
                        RemoveWhenExpired     : False
                        IsExpired             : False
                        Transitions           : ->Ok = 4/21/2015 8:02:12 AM

                        SourceId              : PowershellWatcher
                        Property              : CPU
                        HealthState           : Warning
                        SequenceNumber        : 130741236814913394
                        SentAt                : 4/21/2015 9:01:21 PM
                        ReceivedAt            : 4/21/2015 9:01:21 PM
                        TTL                   : 00:02:00
                        Description           : CPU is above 80% threshold
                        RemoveWhenExpired     : False
                        IsExpired             : False
                        Transitions           : ->Warning = 4/21/2015 9:01:21 PM
```

En el ejemplo siguiente se notifica una advertencia transitoria en una réplica. Primero se obtiene el identificador de partición y luego el identificador de réplica del servicio en el que está interesado. A continuación, se envía un informe de **PowershellWatcher** en la propiedad **ResourceDependency**. El informe es de interés durante solo dos minutos, y se quita del almacén de manera automática.

```powershell
PS C:\> $partitionId = (Get-ServiceFabricPartition -ServiceName fabric:/WordCount/WordCount.Service).PartitionId

PS C:\> $replicaId = (Get-ServiceFabricReplica -PartitionId $partitionId | where {$_.ReplicaRole -eq "Primary"}).ReplicaId

PS C:\> Send-ServiceFabricReplicaHealthReport -PartitionId $partitionId -ReplicaId $replicaId -HealthState Warning -SourceId PowershellWatcher -HealthProperty ResourceDependency -Description "The external resource that the primary is using has been rebooted at 4/21/2015 9:01:21 PM. Expect processing delays for a few minutes." -TimeToLiveSec 120 -RemoveWhenExpired

PS C:\> Get-ServiceFabricReplicaHealth  -PartitionId $partitionId -ReplicaOrInstanceId $replicaId


PartitionId           : 8f82daff-eb68-4fd9-b631-7a37629e08c0
ReplicaId             : 130740415594605869
AggregatedHealthState : Warning
UnhealthyEvaluations  :
                        Unhealthy event: SourceId='PowershellWatcher', Property='ResourceDependency', HealthState='Warning', ConsiderWarningAsError=false.

HealthEvents          :
                        SourceId              : System.RA
                        Property              : State
                        HealthState           : Ok
                        SequenceNumber        : 130740768777734943
                        SentAt                : 4/21/2015 8:01:17 AM
                        ReceivedAt            : 4/21/2015 8:02:12 AM
                        TTL                   : Infinite
                        Description           : Replica has been created.
                        RemoveWhenExpired     : False
                        IsExpired             : False
                        Transitions           : ->Ok = 4/21/2015 8:02:12 AM

                        SourceId              : PowershellWatcher
                        Property              : ResourceDependency
                        HealthState           : Warning
                        SequenceNumber        : 130741243777723555
                        SentAt                : 4/21/2015 9:12:57 PM
                        ReceivedAt            : 4/21/2015 9:12:57 PM
                        TTL                   : 00:02:00
                        Description           : The external resource that the primary is using has been rebooted at 4/21/2015 9:01:21 PM. Expect processing delays for a few minutes.
                        RemoveWhenExpired     : True
                        IsExpired             : False
                        Transitions           : ->Warning = 4/21/2015 9:12:32 PM
```

### <a name="rest"></a>REST
Envíe informes de mantenimiento mediante REST con solicitudes POST que vayan a la entidad deseada y tengan en el cuerpo la descripción del informe de mantenimiento. Por ejemplo, consulte cómo enviar [informes de mantenimiento de clúster](/rest/api/servicefabric/report-the-health-of-a-cluster) o [informes de mantenimiento de servicio](/rest/api/servicefabric/report-the-health-of-a-service) de REST. Se admiten todas las entidades.

## <a name="next-steps"></a>Pasos siguientes
Según los datos del estado, los escritores del servicio y los administradores de clúster/aplicación pueden pensar en maneras de utilizar la información. Por ejemplo, pueden configurar alertas basadas en el estado de mantenimiento para detectar problemas graves antes de provocar interrupciones. Los administradores también pueden configurar sistemas de reparación para solucionar problemas de forma automática.

[Introducción a la supervisión de mantenimiento de Service Fabric](service-fabric-health-introduction.md)

[Vista de los informes de estado de Service Fabric](service-fabric-view-entities-aggregated-health.md)

[Notificación y comprobación del estado del servicio](service-fabric-diagnostics-how-to-report-and-check-service-health.md)

[Uso de informes de mantenimiento del sistema para solucionar problemas](service-fabric-understand-and-troubleshoot-with-system-health-reports.md)

[Supervisión y diagnóstico de los servicios localmente](service-fabric-diagnostics-how-to-monitor-and-diagnose-services-locally.md)

[Actualización de la aplicación de Service Fabric](service-fabric-application-upgrade.md)
