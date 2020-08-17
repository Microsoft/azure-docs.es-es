---
title: Continuidad empresarial y recuperación ante desastres
description: Diseñe la estrategia para proteger los datos, recuperarse rápidamente de eventos perjudiciales, restaurar los recursos necesarios para las funciones empresariales críticas y mantener la continuidad empresarial para Azure Logic Apps
services: logic-apps
ms.suite: integration
ms.reviewer: klam, logicappspm
ms.topic: conceptual
ms.date: 03/31/2020
ms.openlocfilehash: cc55b24c4852028eb1244e97b48415ba08420e20
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87066533"
---
# <a name="business-continuity-and-disaster-recovery-for-azure-logic-apps"></a>Continuidad empresarial y recuperación ante desastres para Azure Logic Apps

Para ayudar a reducir el impacto y los efectos que los eventos imprevisibles tienen en la empresa y los clientes, asegúrese de contar con una solución de [*recuperación ante desastres* (DR)](https://en.wikipedia.org/wiki/Disaster_recovery) para poder proteger los datos, restaurar rápidamente los recursos que complementan las funciones empresariales críticas y seguir ejecutando las operaciones para mantener la [*continuidad empresarial* (BC)](https://en.wikipedia.org/wiki/Business_continuity_planning). Por ejemplo, las interrupciones pueden incluir pérdidas en la infraestructura o los componentes subyacentes, como los recursos de almacenamiento, red o proceso, errores irrecuperables de aplicaciones o incluso una pérdida total del centro de información. Al tener una solución de continuidad empresarial y recuperación ante desastres (BCDR) lista, la empresa o la organización puede responder más rápidamente a las interrupciones (planeadas o no) y reducir el tiempo de inactividad para los clientes.

En este artículo se proporcionan instrucciones y estrategias de BCDR que puede aplicar al crear flujos de trabajo automatizados mediante [Azure Logic Apps](../logic-apps/logic-apps-overview.md). Los flujos de trabajo de aplicaciones lógicas le ayudan a integrar datos y organizarlos más fácilmente entre aplicaciones, servicios en la nube y sistemas locales mediante la reducción de la cantidad de código que se debe escribir. Al planificar la continuidad empresarial y recuperación ante desastres, asegúrese de tener en cuenta no solo las aplicaciones lógicas, sino también estos recursos de Azure que se usan con ellas:

* Las [conexiones](../connectors/apis-list.md) que se crean a partir de aplicaciones lógicas a otras aplicaciones, servicios y sistemas. Para obtener más información, vea [Conexiones a recursos](#resource-connections) más adelante en este tema.

* Las [puertas de enlace de datos locales](../logic-apps/logic-apps-gateway-connection.md), que son recursos de Azure que se crean y se usan en las aplicaciones lógicas para acceder a los datos de sistemas locales. Cada recurso de puerta de enlace representa una [instalación de puerta de enlace independiente](../logic-apps/logic-apps-gateway-install.md) en un equipo local. Para obtener más información, vea [Puertas de enlace de datos locales](#on-premises-data-gateways) más adelante en este tema.

* Las [cuentas de integración](../logic-apps/logic-apps-enterprise-integration-create-integration-account.md) donde se definen y almacenan los artefactos que las aplicaciones lógicas usan para escenarios de [integración empresarial de negocio a negocio (B2B)](../logic-apps/logic-apps-enterprise-integration-overview.md). Por ejemplo, puede [configurar la recuperación ante desastres entre regiones para cuentas de integración](../logic-apps/logic-apps-enterprise-integration-b2b-business-continuity.md).

* Los [entornos de servicio de integración (ISE)](../logic-apps/connect-virtual-network-vnet-isolated-environment-overview.md) en los que se crean aplicaciones lógicas que se ejecutan en una instancia aislada del entorno de ejecución de Logic Apps dentro de una red virtual de Azure. Después, estas aplicaciones lógicas pueden acceder a los recursos que están protegidos detrás de un firewall en esa red virtual.

<a name="primary-secondary-locations"></a>

## <a name="primary-and-secondary-locations"></a>Ubicaciones principales y secundarias

Cada aplicación lógica debe especificar la ubicación que quiera usar para la implementación. Esta ubicación es una región pública en una instancia de Azure multiinquilino global, como "Oeste de EE. UU.", o bien un entorno de servicio de integración (ISE) que ha creado e implementado antes en una red virtual de Azure. La ejecución de aplicaciones lógicas en un ISE es similar a la ejecución en una región global de Azure, lo que significa que la estrategia de recuperación ante desastres se puede aplicar a ambos escenarios. Sin embargo, los ISE tienen otras consideraciones, como la configuración del acceso a los recursos que solo están disponibles para ellos.

> [!NOTE]
> Si la aplicación lógica también funciona con artefactos B2B, como entidades, acuerdos entre socios comerciales, esquemas, mapas y certificados, que se almacenan en una cuenta de integración, tanto la cuenta de integración como las aplicaciones lógicas deben especificar la misma ubicación.

Esta estrategia de recuperación ante desastres se centra en la configuración de la aplicación lógica principal para que [*realice una conmutación por error*](https://en.wikipedia.org/wiki/Failover) en una aplicación lógica en espera o de copia de seguridad en una ubicación alternativa donde también esté disponible Azure Logic Apps. De este modo, si la aplicación lógica principal sufre pérdidas, interrupciones o errores, la secundaria puede continuar el trabajo. En esta estrategia es necesario que la aplicación lógica secundaria y los recursos dependientes ya estén implementados y listos en la ubicación alternativa.

Si sigue los procedimientos recomendados de DevOps, ya usa [plantillas de Azure Resource Manager](../azure-resource-manager/management/overview.md) para definir e implementar las aplicaciones lógicas y sus recursos dependientes. Las plantillas de Resource Manager le permiten usar una única definición de implementación y, después, utilizar archivos de parámetros para proporcionar los valores de configuración que se usarán en cada destino de implementación. Esta capacidad significa que puede implementar la misma aplicación lógica en entornos diferentes, por ejemplo, de desarrollo, prueba y producción. También puede implementar la misma aplicación lógica en diferentes regiones o ISE de Azure, lo que permite estrategias de recuperación ante desastres que usan [regiones emparejadas](../best-practices-availability-paired-regions.md).

En cuanto a la estrategia de conmutación por error, las aplicaciones lógicas y las ubicaciones deben cumplir estos requisitos:

* La instancia de aplicación lógica secundaria tiene acceso a las mismas aplicaciones, servicios y sistemas que la instancia de aplicación lógica principal.

* Las dos instancias de aplicación lógica tienen el mismo tipo de host. Por tanto, las dos instancias se implementan en regiones de Azure multiinquilino global, o bien en ISE, lo que permite a las aplicaciones lógicas acceder directamente a los recursos de una red virtual de Azure. Para obtener los procedimientos recomendados y más información sobre las regiones emparejadas para BCDR, vea [Continuidad empresarial y recuperación ante desastres (BCDR): regiones emparejadas de Azure](../best-practices-availability-paired-regions.md).

  Por ejemplo, las ubicaciones principal y secundaria deben ser ISE cuando la aplicación lógica principal se ejecuta en un ISE y usa [conectores con versión de ISE](../connectors/apis-list.md#ise-connectors), acciones HTTP para llamar a los recursos de la red virtual de Azure o ambos. En este escenario, la aplicación lógica secundaria también debe tener en la ubicación secundaria una configuración similar a la de la aplicación lógica principal.

  > [!NOTE]
  > En el caso de escenarios más avanzados, puede mezclar instancias multiinquilino de Azure y un ISE como ubicaciones. Pero asegúrese de tener en cuenta y comprender las [diferencias entre la ejecución de las aplicaciones lógicas en un ISE y en instancias multiinquilino de Azure](../logic-apps/connect-virtual-network-vnet-isolated-environment-overview.md#difference).

* Si usa ISE, [asegúrese de que se escalen horizontalmente o que tengan capacidad suficiente](../logic-apps/ise-manage-integration-service-environment.md#add-capacity) para controlar la carga.

#### <a name="example-multi-tenant-azure"></a>Ejemplo: instancias multiinquilino de Azure

En este ejemplo se muestran instancias de aplicaciones lógicas principales y secundarias, que se implementan en regiones independientes de la instancia multiinquilino global de Azure para este escenario. Las dos instancias de aplicación lógica y los recursos dependientes que necesitan se definen en una sola [plantilla de Resource Manager](../logic-apps/logic-apps-azure-resource-manager-templates-overview.md). Los valores de configuración que se usan para cada ubicación de implementación se especifican en archivos de parámetros independientes:

![Instancias de aplicaciones lógicas principales y secundarias en ubicaciones independientes](./media/business-continuity-disaster-recovery-guidance/primary-secondary-locations.png)

#### <a name="example-integration-service-environment"></a>Ejemplo: entorno de servicio de integración

En este ejemplo se muestran las instancias de aplicación lógica principal y secundaria anteriores, pero implementadas en ISE independientes. Las dos instancias de aplicación lógica, los recursos dependientes que necesitan y los ISE se definen en una sola plantilla de Resource Manager como las ubicaciones de implementación. Los valores de configuración que se usan para la implementación en cada ubicación se definen en archivos de parámetros independientes:

![Aplicaciones lógicas principales y secundarias en ubicaciones diferentes](./media/business-continuity-disaster-recovery-guidance/primary-secondary-locations-ise.png)

<a name="resource-connections"></a>

## <a name="connections-to-resources"></a>Conexiones a recursos

Azure Logic Apps proporciona [acciones y desencadenadores integrados además de cientos de conectores administrados](../connectors/apis-list.md) que la aplicación lógica puede usar para trabajar con otras aplicaciones, servicios, sistemas y recursos, como cuentas de Azure Storage, bases de datos de SQL Server, cuentas de correo electrónico de Office 365 Outlook, etc. Si la aplicación lógica necesita acceder a estos recursos, debe crear conexiones que autentiquen el acceso a estos recursos. Cada conexión es un recurso de Azure independiente que existe en una ubicación específica y que los recursos de otras ubicaciones no pueden usar.

En la estrategia de recuperación ante desastres, tenga en cuenta las ubicaciones en las que existen los recursos dependientes en relación con las instancias de la aplicación lógica:

* La instancia principal y los recursos dependientes existen en ubicaciones diferentes. En este caso, la instancia secundaria se puede conectar a los mismos recursos o puntos de conexión dependientes. Sin embargo, debe crear conexiones específicas para la instancia secundaria. De ese modo, si la ubicación principal deja de estar disponible, las conexiones de la secundaria no se verán afectadas.

  Por ejemplo, imagine que la aplicación lógica principal se conecta a un servicio externo, como Salesforce. Normalmente, la disponibilidad y la ubicación del servicio externo son independientes de la disponibilidad de la aplicación lógica. En este caso, la instancia secundaria se puede conectar al mismo servicio, pero debe tener su propia conexión.

* Tanto la instancia principal como los recursos dependientes existen en la misma ubicación. En este caso, los recursos dependientes deben tener copias de seguridad o versiones replicadas en otra ubicación, de modo que la instancia secundaria todavía pueda acceder a esos recursos.

  Por ejemplo, imagine que la aplicación lógica principal se conecta a un servicio que se encuentra en la misma ubicación o región, por ejemplo, Azure SQL Database. Si toda la región deja de estar disponible, es probable que el servicio Azure SQL Database de esa región tampoco esté disponible. En este caso, le interesará que la instancia secundaria use una base de datos replicada o de copia de seguridad junto con una conexión independiente a esa base de datos.

<a name="on-premises-data-gateways"></a>

## <a name="on-premises-data-gateways"></a>Puertas de enlace de datos locales

Si la aplicación lógica se ejecuta en la instancia multiinquilino de Azure y necesita acceso a recursos locales como bases de datos de SQL Server, tendrá que instalar la [puerta de enlace de datos local](../logic-apps/logic-apps-gateway-install.md) en un equipo local. Después, puede crear un recurso de puerta de enlace de datos en Azure Portal para que la aplicación lógica pueda usar la puerta de enlace al crear una conexión con el recurso.

El recurso de puerta de enlace de datos está asociado a una ubicación o región de Azure, al igual que el recurso de la aplicación lógica. En la estrategia de recuperación ante desastres, asegúrese de que la puerta de enlace de datos sigue disponible para que la use la aplicación lógica. Puede [habilitar la alta disponibilidad para la puerta de enlace](../logic-apps/logic-apps-gateway-install.md#high-availability) cuando tenga varias instalaciones de puerta de enlace.

> [!NOTE]
> Si la aplicación lógica se ejecuta en un entorno de servicio de integración (ISE) y solo usa conectores con versión de ISE para los orígenes de datos locales, no necesita la puerta de enlace de datos, ya que los conectores ISE proporcionan acceso directo al recurso local.
>
> Si no hay ningún conector de versión de ISE disponible para el recurso local que quiere, la aplicación lógica todavía puede crear la conexión mediante un conector que no sea ISE, que se ejecuta en la instancia multiinquilino global de Azure, no en el ISE. Sin embargo, esta conexión requiere la puerta de enlace de datos local.

<a name="roles"></a>

## <a name="active-active-versus-active-passive-roles"></a>Diferencias entre roles activos-activos y activos-pasivos

Puede configurar las ubicaciones principal y secundaria para que las instancias de la aplicación lógica en estas ubicaciones puedan desempeñar estos roles:

| Rol principal-secundario | Descripción |
|------------------------|-------------|
| *Activo-activo* | Las instancias principal y secundaria de la aplicación lógica en las dos ubicaciones controlan las solicitudes de forma activa mediante uno de estos patrones: <p><p>- *Equilibrio de carga*: puede hacer que las dos instancias escuchen en un punto de conexión y equilibrar la carga del tráfico en cada instancia según sea necesario. <p>- *Consumidores simultáneos*: puede hacer que las dos instancias actúen como consumidores simultáneos para que compitan por los mensajes de una cola. Si se produce un error en una instancia, la otra asume la carga de trabajo. |
| *Activo-pasivo* | La instancia de aplicación lógica principal controla de forma activa toda la carga de trabajo, mientras que la secundaria es pasiva (está deshabilitada o inactiva). La secundaria espera una señal de que la principal no está disponible o no funciona debido a una interrupción o un error, y asume la carga de trabajo como instancia activa. |
| Combinación | Algunas aplicaciones lógicas desempeñan un rol activo-activo, mientras que otras asumen un rol activo-pasivo. |
|||

<a name="active-active-examples"></a>

### <a name="active-active-examples"></a>Ejemplos de rol activo-activo

En estos ejemplos se muestra la configuración activo-activo donde las dos instancias de aplicación lógica controlan de forma activa las solicitudes o los mensajes. Otro sistema o servicio distribuye las solicitudes o los mensajes entre las instancias, por ejemplo, una de estas opciones:

* Un equilibrador de carga "físico", como un componente de hardware que enruta el tráfico

* Un equilibrador de carga "temporal", como [Azure Load Balancer](../load-balancer/load-balancer-overview.md) o [Azure API Management](../api-management/api-management-key-concepts.md). Con API Management, puede especificar directivas que determinen cómo equilibrar la carga del tráfico entrante. O bien, puede usar un servicio que admita el seguimiento de estado, por ejemplo, [Azure Service Bus](../service-bus-messaging/service-bus-messaging-overview.md).

  Aunque en este ejemplo se muestra principalmente Azure Load Balancer, puede usar la opción que mejor se adapte a las necesidades del escenario:

  ![Configuración "activo-activo" en la que se usa un equilibrador de carga o un servicio con estado](./media/business-continuity-disaster-recovery-guidance/active-active-setup-load-balancer.png)

* Cada instancia de aplicación lógica actúa como un consumidor y las dos instancias compiten por los mensajes de una cola:

  ![Configuración "activo-activo" con "consumidores simultáneos"](./media/business-continuity-disaster-recovery-guidance/active-active-competing-consumers-pattern.png)

<a name="active-passive-examples"></a>

### <a name="active-passive-examples"></a>Ejemplos de rol activo-pasivo

En este ejemplo se muestra la configuración activo-pasivo en la que la instancia de aplicación lógica principal está activa en una ubicación, mientras que la secundaria permanece inactiva en otra. Si la principal experimenta una interrupción o un error, puede hacer que un operador ejecute un script que active la secundaria para asumir la carga de trabajo.

![Configuración "activo-pasivo" con "consumidores simultáneos"](./media/business-continuity-disaster-recovery-guidance/active-passive-setup.png)

<a name="active-active-active-passive-examples"></a>

### <a name="combination-with-active-active-and-active-passive"></a>Combinación con activo-activo y activo-pasivo

En este ejemplo se muestra una configuración combinada en la que la ubicación principal tiene instancias de aplicación lógica activas, mientras que la secundaria tiene instancias de aplicación lógica activas-pasivas. Si la ubicación principal sufre una interrupción o un error, la aplicación lógica activa en la ubicación secundaria, que ya controla una carga de trabajo parcial, puede asumir toda la carga de trabajo.

* En la ubicación principal, una aplicación lógica activa escucha los mensajes en una cola de Azure Service Bus, mientras que otra aplicación lógica activa busca correos electrónicos mediante un desencadenador de sondeo de Office 365 Outlook.

* En la ubicación secundaria, una aplicación lógica activa trabaja con la de la ubicación principal escuchando y compitiendo por los mensajes de la misma cola de Service Bus. Mientras tanto, una aplicación lógica inactiva pasiva espera para comprobar los mensajes de correo electrónico cuando la ubicación principal deja de estar disponible, pero está *deshabilitada* para evitar que los mensajes se vuelvan a leer.

![Combinación "activa-pasiva" y "activa-pasiva" con desencadenadores de periodicidad](./media/business-continuity-disaster-recovery-guidance/combo-active-active-active-passive-setup.png)

<a name="state-history"></a>

## <a name="logic-app-state-and-history"></a>Estado e historial de la aplicación lógica

Cuando se desencadena la aplicación lógica y comienza a ejecutarse, su estado se almacena en la misma ubicación en la que se ha iniciado y no se transfiere a otra. Si se produce un error o una interrupción, se abandonan todas las instancias de flujo de trabajo en curso. Si ha configurado ubicaciones principales y secundarias, las nuevas instancias de flujo de trabajo comienzan a ejecutarse en la secundaria.

<a name="reduce-abandoned-in-progress"></a>

### <a name="reduce-abandoned-in-progress-instances"></a>Reducción de instancias en curso abandonadas

Para minimizar el número de instancias de flujo de trabajo en curso abandonadas, puede elegir entre varios patrones de mensaje para implementar, por ejemplo:

* [Patrón de lista de distribución fija](/biztalk/esb-toolkit/message-routing-patterns#routing-slip)

  Este patrón de mensajes empresariales divide un proceso empresarial en fases más pequeñas. En cada fase, se configura una aplicación lógica que controla la carga de trabajo de esa fase. Para comunicarse entre sí, las aplicaciones lógicas usan un protocolo de mensajería asincrónico como colas o temas de Azure Service Bus. Al dividir un proceso en fases más pequeñas, se reduce el número de procesos empresariales que podrían quedarse bloqueados en una instancia de aplicación lógica con errores. Para obtener más información general sobre este patrón, vea [Patrones de integración empresarial: lista de distribución](https://www.enterpriseintegrationpatterns.com/patterns/messaging/RoutingTable.html).

  En este ejemplo se muestra un patrón de lista de distribución en el que cada aplicación lógica representa una fase y usa una cola de Service Bus para comunicarse con la siguiente aplicación lógica del proceso.

  ![División de un proceso empresarial en fases representadas por aplicaciones lógicas, que se comunican entre sí mediante colas de Azure Service Bus](./media/business-continuity-disaster-recovery-guidance/fixed-routing-slip-pattern.png)

  Si las instancias principal y secundaria de la aplicación lógica siguen el mismo patrón de lista de distribución en sus ubicaciones, puede implementar el [patrón de consumidores simultáneos](/azure/architecture/patterns/competing-consumers) si configura [roles activos-activos](#roles) para esas instancias.

* [Patrón de administrador de procesos (agente)](https://www.enterpriseintegrationpatterns.com/patterns/messaging/ProcessManager.html)

* [Patrón de inspección y bloqueo sin tiempo de expiración](https://social.technet.microsoft.com/wiki/contents/articles/50022.azure-service-bus-how-to-peek-lock-a-message-from-queue-using-azure-logic-apps.aspx)

<a name="access-trigger-runs-history"></a>

### <a name="access-to-trigger-and-runs-history"></a>Acceso al historial de ejecuciones y desencadenadores

Para obtener más información sobre las ejecuciones de flujo de trabajo anteriores de la aplicación lógica, puede revisar su historial de desencadenadores y ejecuciones. El historial de ejecución de una aplicación lógica se almacena en la misma ubicación o región en la que se ha ejecutado, lo que significa que no se puede migrar a otra ubicación. Si la instancia principal realiza una conmutación por error a una instancia secundaria, solo se puede acceder al historial de ejecuciones y desencadenadores de cada instancia en las ubicaciones correspondientes en las que se hayan ejecutado. Sin embargo, puede obtener información independiente de la ubicación sobre el historial de la aplicación lógica si configura las aplicaciones lógicas para que envíen eventos de diagnóstico a un área de trabajo de Azure Log Analytics. Después, puede revisar el estado y el historial entre las aplicaciones lógicas que se ejecutan en varias ubicaciones.

<a name="trigger-types-guidance"></a>

## <a name="trigger-type-guidance"></a>Instrucciones sobre los tipos de desencadenador

El tipo de desencadenador que use en las aplicaciones lógicas determina las opciones de configuración de las aplicaciones lógicas en todas las ubicaciones de la estrategia de recuperación ante desastres. Estos son los tipos de desencadenador disponibles que puede usar en las aplicaciones lógicas:

* [Desencadenador de periodicidad](#recurrence-trigger)
* [Desencadenador de sondeo](#polling-trigger)
* [Desencadenador de solicitud](#request-trigger)
* [Desencadenador de webhook](#webhook-trigger)

<a name="recurrence-trigger"></a>

### <a name="recurrence-trigger"></a>Desencadenador de periodicidad

El desencadenador de **periodicidad** es independiente de cualquier servicio o punto de conexión específico, y solo se activa en función de una programación especificada y ningún otro criterio, por ejemplo:

* Una frecuencia y un intervalo fijos, por ejemplo, cada 10 minutos
* Una programación más avanzada, como el último lunes de cada mes a las 17:00.

Si la aplicación lógica se inicia con un desencadenador de periodicidad, debe configurar las instancias principal y secundaria de la aplicación lógica con los [roles activos-pasivos](#roles). Para reducir el *objetivo de tiempo de recuperación* (RTO), que hace referencia a la duración de destino para la restauración de un proceso empresarial después de una interrupción o un desastre, puede configurar las instancias de la aplicación lógica con una combinación de [roles activos-pasivos](#roles) y [roles pasivos-activos](#roles). En esta configuración, la programación se divide entre las ubicaciones.

Por ejemplo, imagine que tiene una aplicación lógica que se debe ejecutar cada 10 minutos. Puede configurar las ubicaciones y las aplicaciones lógicas de forma que si la ubicación principal deja de estar disponible, la secundaria pueda asumir el trabajo:

![Combinación "activa-pasiva" y "pasiva-activa" con desencadenadores de periodicidad](./media/business-continuity-disaster-recovery-guidance/combo-active-passive-passive-active-setup.png)

* En la ubicación principal, configure [roles activos-pasivos](#roles) para estas aplicaciones lógicas:

  * En el caso de la aplicación lógica habilitada *activa*, establezca el desencadenador de periodicidad para que se inicie al comenzar la hora y se repita cada 20 minutos, por ejemplo, 9:00, 9:20, etc.

  * En el caso de la aplicación lógica deshabilitada *pasiva*, establezca el desencadenador de periodicidad en la misma programación, pero para que se inicie a los 10 minutos de comenzar la hora y se repita cada 20 minutos, por ejemplo, 9:10, 9:30, etc.

* En la ubicación secundaria, configure estas aplicaciones lógicas como [pasiva-activa](#roles):

  * Para la aplicación lógica deshabilitada *pasiva*, establezca el desencadenador de periodicidad en la misma programación que la aplicación lógica activa de la ubicación principal, que empieza al principio de la hora y se repite cada 20 minutos, por ejemplo, 9:00, 9:10 y así sucesivamente.

  * Para la aplicación lógica habilitada *activa*, establezca el desencadenador de periodicidad en la misma programación que la aplicación lógica pasiva de la ubicación principal, que empieza a los 10 minutos de la hora y se repite cada 20 minutos, por ejemplo, 9:10, 9:20 y así sucesivamente.

Ahora, si se produce un evento de interrupción en la ubicación principal, se activa la aplicación lógica pasiva en la ubicación alternativa. De este modo, si para identificar el error se necesita tiempo, esta configuración limita el número de repeticiones que faltan durante ese retraso.

<a name="polling-trigger"></a>

### <a name="polling-trigger"></a>Desencadenador de sondeo

Para comprobar de forma periódica si hay nuevos datos disponibles para el procesamiento desde un servicio o punto de conexión concreto, la aplicación lógica puede usar un desencadenador *de sondeo* que llame repetidamente al servicio o punto de conexión en función de una programación de periodicidad fija. Los datos proporcionados por el servicio o el punto de conexión pueden tener cualquiera de estos tipos:

* Datos estáticos, que describen los datos que siempre están disponibles para la lectura
* Datos volátiles, que describen los datos que ya no están disponibles para la lectura

Para evitar leer repetidamente los mismos datos, la aplicación lógica debe recordar qué datos se han leído previamente mediante el mantenimiento del estado en el lado cliente o en el del servidor, servicio o sistema.

* Las aplicaciones lógicas que funcionan con el estado del lado cliente usan desencadenadores que pueden mantener el estado.

  Por ejemplo, un desencadenador que lee un mensaje nuevo de una bandeja de entrada de correo electrónico requiere que el desencadenador pueda recordar el mensaje leído más recientemente. De este modo, el desencadenador inicia la aplicación lógica solo cuando llega el siguiente mensaje no leído.

* Las aplicaciones lógicas que funcionan con el estado del servidor, el servicio o el sistema usan valores de propiedad u opciones que se encuentran en el lado del servidor, el servicio o el sistema.

  Por ejemplo, un desencadenador basado en consultas que lee una fila de una base de datos requiere que la fila tenga una columna `isRead` establecida en `FALSE`. Cada vez que el desencadenador lee una fila, la aplicación lógica actualiza esa fila y cambia la columna `isRead` de `FALSE` a `TRUE`.

  Este enfoque del lado servidor funciona de forma similar para colas o temas de Service Bus que tienen semántica de puesta en cola, donde un desencadenador puede leer y bloquear un mensaje mientras la aplicación lógica lo procesa. Cuando la aplicación lógica finaliza el procesamiento, el desencadenador elimina el mensaje de la cola o el tema.

Desde la perspectiva de la recuperación ante desastres, al configurar las instancias principal y secundaria de la aplicación lógica, asegúrese de que tiene en cuenta estos comportamientos en función de si la aplicación lógica realiza el seguimiento del estado en el lado cliente o en el lado servidor:

* En el caso de una aplicación lógica que funcione con el estado del lado cliente, asegúrese de que la aplicación lógica no lea el mismo mensaje más de una vez. Solo puede haber una instancia de aplicación lógica activa en un momento determinado en una ubicación. Asegúrese de que la instancia de aplicación lógica de la ubicación alternativa está inactiva o deshabilitada hasta que la instancia principal conmute por error a la ubicación alternativa.

  Por ejemplo, el desencadenador de Office 365 Outlook mantiene el estado del lado cliente y realiza el seguimiento de la marca de tiempo del último correo electrónico que se haya leído para evitar que se lea un duplicado.

* En el caso de una aplicación lógica que funciona con el estado del lado servidor, puede configurar las instancias para que desempeñen [roles activos-activos](#roles) en los que funcionan como consumidores simultáneos o [roles activos-pasivos](#roles) en los que la instancia alternativa espera hasta que la principal conmute por error a la ubicación alternativa.

  Por ejemplo, la lectura desde una cola de mensajes, como una cola de Azure Service Bus, usa el estado del lado servidor, ya que el servicio de puesta en cola mantiene bloqueos en los mensajes para evitar que otros clientes lean los mismos mensajes.

  > [!NOTE]
  > Si la aplicación lógica tiene que leer los mensajes en un orden concreto, por ejemplo, desde una cola de Service Bus, puede usar el patrón de consumidor simultáneo, pero solo cuando lo combina con sesiones de Service Bus, lo que también se conoce como [patrón de *convoy* secuencial](/azure/architecture/patterns/sequential-convoy). De lo contrario, tendrá que configurar las instancias de la aplicación lógica con los roles activos-pasivos.

<a name="request-trigger"></a>

### <a name="request-trigger"></a>Desencadenador de solicitud

El desencadenador de **solicitud** hace que se pueda llamar a la aplicación lógica desde otras aplicaciones, servicios y sistemas, y normalmente se usa para proporcionar estas funciones:

* API REST directa para la aplicación lógica a la que otros usuarios pueden llamar

  Por ejemplo, use el desencadenador de solicitud para iniciar la aplicación lógica para que otras aplicaciones lógicas puedan llamar al desencadenador mediante la acción **Flujo de trabajo de llamada: Logic Apps**.

* [Webhook](#webhook-trigger) o mecanismo de devolución de llamada para la aplicación lógica

* Una manera de poder ejecutar manualmente las operaciones de usuario o rutinas para llamar a la aplicación lógica, por ejemplo, mediante un script de PowerShell que realiza una tarea específica.

Desde una perspectiva de recuperación ante desastres, el desencadenador de solicitud es un receptor pasivo porque la aplicación lógica no realiza ningún trabajo y espera hasta que otro servicio o sistema llame de forma explícita al desencadenador. Como punto de conexión pasivo, puede configurar las instancias principal y secundaria de estas maneras:

* [Activo-activo](#roles): las dos instancias controlan las solicitudes o llamadas de forma activa. El autor de la llamada o el enrutador equilibra o distribuye el tráfico entre esas instancias.

* [Activo-pasivo](#roles): solo la instancia principal está activa y controla todo el trabajo, mientras que la instancia secundaria espera hasta que se produce una interrupción o un error en la principal. El autor de la llamada o el enrutador determina cuándo se debe llamar a la instancia secundaria.

Como arquitectura recomendada, puede utilizar Azure API Management como proxy para las aplicaciones lógicas que usan desencadenadores de solicitud. API Management proporciona [resistencia entre regiones integrada y la capacidad de enrutar el tráfico a través de varios puntos de conexión](../api-management/api-management-howto-deploy-multi-region.md).

<a name="webhook-trigger"></a>

### <a name="webhook-trigger"></a>Desencadenador de webhook

Un desencadenador de *webhook* proporciona la funcionalidad para que la aplicación lógica se suscriba a un servicio pasando una *dirección URL de devolución de llamada* a ese servicio. Después, la aplicación lógica puede escuchar y esperar a que se produzca un evento específico en ese punto de conexión de servicio. Cuando se produce el evento, el servicio llama al desencadenador de webhook mediante la dirección URL de devolución de llamada, que luego ejecuta la aplicación lógica. Cuando está habilitado, el desencadenador de webhook se suscribe al servicio. Cuando está deshabilitado, el desencadenador de webhook cancela la suscripción al servicio.

Desde una perspectiva de recuperación ante desastres, configure instancias principales y secundarias que usan desencadenadores de webhook para reproducir roles activos-pasivos, ya que solo una instancia debe recibir eventos o mensajes del punto de conexión suscrito.

## <a name="assess-primary-instance-health"></a>Evaluación del estado de la instancia principal

Para que la estrategia de recuperación ante desastres funcione, la solución necesita formas de realizar estas tareas:

* [Comprobar la disponibilidad de la instancia principal](#check-primary-availability)
* [Supervisar el estado de la instancia principal](#monitor-primary-health)
* [Activar la instancia secundaria](#activate-secondary)

En esta sección se describe una solución que puede usar de forma directa o como base para un diseño propio. A continuación se muestra una introducción visual general para esta solución:

![Creación de una aplicación lógica de guardián para supervisar una aplicación lógica de comprobación de estado en la ubicación principal](./media/business-continuity-disaster-recovery-guidance/check-location-health-watchdog.png)

<a name="check-primary-availability"></a>

### <a name="check-primary-instance-availability"></a>Comprobación de la disponibilidad de la instancia principal

Para determinar si la instancia principal está disponible, en ejecución y puede funcionar, puede crear una aplicación lógica de "comprobación de estado" que se encuentre en la misma ubicación que la instancia principal. Después, puede llamar a esta aplicación de comprobación de estado desde una ubicación alternativa. Si la aplicación de comprobación de estado responde de forma correcta, la infraestructura subyacente para el servicio Azure Logic Apps de esa región está disponible y en funcionamiento. Si la aplicación de comprobación de estado no responde, puede suponer que la ubicación ya no es correcta.

Para esta tarea, creará una aplicación lógica de comprobación de estado básica que realiza estas tareas:

1. Recibe una llamada de la aplicación de guardián mediante el desencadenador de solicitud.

1. Responda con un estado que indica si la aplicación lógica comprobada sigue funcionando mediante la acción de respuesta.

   > [!IMPORTANT]
   > La aplicación lógica de comprobación de estado debe usar una acción de respuesta para que la aplicación responda de forma sincrónica, no asincrónica.

1. Opcionalmente, para determinar si la ubicación principal es correcta, puede tener en cuenta el estado de otros servicios que interactúen con la aplicación lógica de destino en esta ubicación. Simplemente expanda la aplicación lógica de comprobación de estado para que también evalúe el estado de estos otros servicios.

<a name="monitor-primary-health"></a>

### <a name="create-a-watchdog-logic-app"></a>Creación de una aplicación lógica guardián

Para supervisar el estado de la instancia principal y llamar a la aplicación lógica de comprobación de estado, cree una aplicación lógica "guardián" en una *ubicación alternativa*. Por ejemplo, puede configurar la aplicación lógica guardián para que, si se produce un error al llamar a la lógica de comprobación de estado, el guardián pueda enviar una alerta al equipo de operaciones para que pueda investigar el error y el motivo de que la instancia principal no responda.

> [!IMPORTANT]
> Asegúrese de que la aplicación lógica guardián esté en una *ubicación distinta a la principal*. Si el servicio Logic Apps de la ubicación principal experimenta problemas, es posible que la aplicación lógica guardián no se ejecute.

Para esta tarea, cree una aplicación lógica guardián en la ubicación secundaria que realice estas tareas:

1. Se ejecute según una periodicidad fija o programada mediante el desencadenador de periodicidad.

   Puede establecer la periodicidad en un valor que esté por debajo del nivel de tolerancia para el objetivo de tiempo de recuperación (RTO).

1. Llame a la aplicación lógica de comprobación de estado en la ubicación principal mediante la acción HTTP, por ejemplo:

<a name="activate-secondary"></a>

### <a name="activate-your-secondary-instance"></a>Activación de la instancia secundaria

Para activar de forma automática la instancia secundaria, puede crear una aplicación lógica que llame a la API de administración, como el [conector de Azure Resource Manager](/connectors/arm/) para activar las aplicaciones lógicas adecuadas en la ubicación secundaria. Puede expandir la aplicación guardián para llamar a esta aplicación lógica de activación después de que se produzca un número específico de errores.

<a name="collect-diagnostic-data"></a>

## <a name="collect-diagnostic-data"></a>Recopilación de datos de diagnóstico

Puede configurar el registro de las ejecuciones de la aplicación lógica y enviar los datos de diagnóstico resultantes a servicios como Azure Storage, Azure Event Hubs y Azure Log Analytics para un mayor control y procesamiento.

* Si quiere usar estos datos con Azure Log Analytics, puede hacer que estén disponibles para las ubicaciones principal y secundaria si configura las **opciones de diagnóstico** de la aplicación lógica y envía los datos a varias áreas de trabajo de Log Analytics. Para más información, consulte [Configuración de registros de Azure Monitor y recopilación de datos de diagnóstico para Azure Logic Apps](../logic-apps/monitor-logic-apps-log-analytics.md).

* Si quiere enviar los datos a Azure Storage o Azure Event Hubs, puede hacer que estén disponibles para las ubicaciones principal y secundaria si configura la redundancia geográfica. Para obtener más información, consulte estos artículos:<p>

  * [Recuperación ante desastres y conmutación por error de Azure Blob Storage](../storage/common/storage-disaster-recovery-guidance.md)
  * [Recuperación ante desastres geográfica de Azure Event Hubs](../event-hubs/event-hubs-geo-dr.md)

## <a name="next-steps"></a>Pasos siguientes

* [Introducción a la resistencia para Azure](/azure/architecture/framework/resiliency/overview)
* [Lista de comprobación de resistencia para servicios de Azure específicos](/azure/architecture/checklist/resiliency-per-service)
* [Administración de datos para lograr la resistencia en Azure](/azure/architecture/framework/resiliency/data-management)
* [Copia de seguridad y recuperación ante desastres para aplicaciones de Azure](/azure/architecture/framework/resiliency/backup-and-recovery)
* [Recuperación después de una interrupción del servicio en toda la región](/azure/architecture/resiliency/recovery-loss-azure-region)
* [Contratos de nivel de servicio (SLA) de Microsoft para servicios de Azure](https://azure.microsoft.com/support/legal/sla/)
