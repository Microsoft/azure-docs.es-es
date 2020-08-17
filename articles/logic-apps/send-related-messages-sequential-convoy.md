---
title: Envío de mensajes correlacionados en orden mediante un convoy secuencial
description: Envíe mensajes relacionados en orden mediante el patrón de convoy secuencial en Azure Logic Apps con Azure Service Bus
services: logic-apps
ms.suite: integration
ms.reviewer: apseth, divswa, logicappspm
ms.topic: conceptual
ms.date: 05/29/2020
ms.openlocfilehash: 8c00d2e4f622bcfad7b2468013336f0d936e318c
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87048657"
---
# <a name="send-related-messages-in-order-by-using-a-sequential-convoy-in-azure-logic-apps-with-azure-service-bus"></a>Envío de mensajes relacionados en orden mediante un convoy secuencial en Azure Logic Apps con Azure Service Bus

Cuando necesite enviar mensajes correlacionados en un orden específico, puede usar el [patrón *convoy secuencial*](/azure/architecture/patterns/sequential-convoy) al usar [Azure Logic Apps](../logic-apps/logic-apps-overview.md) mediante el [conector de Azure Service Bus](../connectors/connectors-create-api-servicebus.md). Los mensajes correlacionados tienen una propiedad que define la relación entre esos mensajes, como el identificador de la [sesión](../service-bus-messaging/message-sessions.md) en Service Bus.

Por ejemplo, supongamos que tiene 10 mensajes para una sesión denominada "Session 1" y tiene 5 mensajes para una sesión denominada "Session 2" que se envían todos a la misma [cola de Service Bus](../service-bus-messaging/service-bus-queues-topics-subscriptions.md). Puede crear una aplicación lógica que procese los mensajes de la cola de modo que una sola ejecución del desencadenar se encargue de todos los mensajes de "Session 1" y la siguiente ejecución del desencadenador se encargue de todos los mensajes de "Session 2".

![Patrón de convoy secuencial general](./media/send-related-messages-sequential-convoy/sequential-convoy-pattern-general.png)

En este artículo se muestra cómo crear una aplicación lógica que implementa este patrón mediante la plantilla **Entrega ordenada y correlacionada mediante sesiones de Service Bus**. Esta plantilla define un flujo de trabajo de aplicación lógica que se inicia con el desencadenador **Cuando se recibe un mensaje en una cola (bloque de inspección)** del conector de Service Bus, que recibe mensajes de una [cola de Service Bus](../service-bus-messaging/service-bus-queues-topics-subscriptions.md). Estos son los pasos generales que realiza esta aplicación lógica:

* Inicializar una sesión basada en un mensaje que lee el desencadenador de la cola de Service Bus.

* Lea y procese todos los mensajes de la misma sesión en la cola durante la ejecución del flujo de trabajo actual.

Para revisar el archivo JSON de esta plantilla, consulte [GitHub: service-bus-sessions.json](https://github.com/Azure/logicapps/blob/master/templates/service-bus-sessions.json).

Para obtener más información, consulte [Patrón de convoy secuencial: patrones de diseño en la nube de arquitectura de Azure](/azure/architecture/patterns/sequential-convoy).

## <a name="prerequisites"></a>Requisitos previos

* Suscripción a Azure. Si aún no tiene una, [regístrese para obtener una cuenta de Azure gratuita](https://azure.microsoft.com/free/).

* Espacio de nombres de Service Bus y una [cola de Service Bus](../service-bus-messaging/service-bus-queues-topics-subscriptions.md), que es una entidad de mensajería que se usará en la aplicación lógica. Estos elementos y la aplicación lógica deben usar la misma suscripción de Azure. Asegúrese de seleccionar **Habilitar sesiones** al crear la cola. Si no tiene estos elementos, aprenda a [crear el espacio de nombres de Service Bus y una cola](../service-bus-messaging/service-bus-create-namespace-portal.md).

  [!INCLUDE [Warning about creating infinite loops](../../includes/connectors-infinite-loops.md)]

* Conocimientos básicos acerca de cómo crear aplicaciones lógicas. Si no está familiarizado con Azure Logic Apps, pruebe la guía de inicio rápido [Creación del primer flujo de trabajo automatizado](../logic-apps/quickstart-create-first-logic-app-workflow.md).

<a name="permissions-connection-string"></a>

## <a name="check-access-to-service-bus-namespace"></a>Comprobación del acceso al espacio de nombres de Service Bus

Si no está seguro de si la aplicación lógica tiene permisos de acceso a su espacio de nombres de Service Bus, confirme dichos permisos.

1. Inicie sesión en [Azure Portal](https://portal.azure.com). Busque y seleccione el *espacio de nombres* de Service Bus.

1. En el menú del espacio de nombres, en **Configuración**, seleccione **Directivas de acceso compartido**. En **Notificaciones**, compruebe que tenga permisos de **Administrador** para ese espacio de nombres.

   ![Administración de permisos para el espacio de nombres de Service Bus](./media/send-related-messages-sequential-convoy/check-service-bus-permissions.png)

1. Ahora obtenga la cadena de conexión para el espacio de nombres de Service Bus. Puede usar esta cadena posteriormente cuando cree una conexión al espacio de nombres desde la aplicación lógica.

   1. En el panel **Directivas de acceso compartido**, en **Directiva**, seleccione **RootManageSharedAccessKey**.
   
   1. Al lado de la cadena de conexión principal, elija el botón de copia. Guarde la cadena de conexión para usarla más adelante.

      ![Copia de la cadena de conexión del espacio de nombres de Service Bus](./media/send-related-messages-sequential-convoy/copy-service-bus-connection-string.png)

   > [!TIP]
   > Para confirmar si la cadena de conexión está asociada al espacio de nombres de Service Bus o a una entidad de mensajería, como una cola, compruebe la cadena de conexión del parámetro `EntityPath` . Si encuentra este parámetro, la cadena de conexión es para una entidad específica y no es la correcta para la aplicación lógica.

## <a name="create-logic-app"></a>Creación de la aplicación lógica

En esta sección, creará una aplicación lógica mediante la plantilla **Entrega ordenada y correlacionada mediante sesiones de Service Bus**, que incluye el desencadenador y las acciones para implementar este modelo de flujo de trabajo. También creará una conexión con el espacio de nombres de Service Bus y especificará el nombre de la cola de Service Bus que quiere usar.

1. En [Azure Portal](https://portal.azure.com), cree una aplicación lógica en blanco. En la página principal de Azure, seleccione **Crear un recurso** > **Integración** > **Aplicación lógica**.

1. Después de que aparezca la galería de plantillas, desplácese más allá de las secciones de vídeo y desencadenadores comunes. En la sección **Plantillas** , seleccione la plantilla **Entrega ordenada y correlacionada mediante sesiones de Service Bus**.

   ![Selección de la plantilla "Entrega ordenada y correlacionada mediante sesiones de Service Bus"](./media/send-related-messages-sequential-convoy/select-correlated-in-order-delivery-template.png)

1. Cuando aparezca el cuadro de confirmación, seleccione **Usar esta plantilla**.

1. En el Diseñador de aplicación lógica, en la forma **Service Bus**, seleccione **continuar** y, a continuación, seleccione el signo más ( **+** ) que aparece en la forma.

   ![Seleccione "Continuar" para conectarse a Azure Service Bus](./media/send-related-messages-sequential-convoy/connect-to-service-bus.png)

1. Ahora elija cualquiera de las dos opciones siguientes para crear una conexión de Service Bus:

   * Para usar la cadena de conexión que copió anteriormente del espacio de nombres de Service Bus, siga estos pasos:

     1. Seleccione **Especificar la información de conexión manualmente**.

     1. En **Nombre de la conexión**, especifique un nombre para la conexión. En **Cadena de conexión**, pegue la cadena de conexión del espacio de nombres y seleccione **Crear**, por ejemplo:

        ![Especificación del nombre de conexión y la cadena de conexión de Service Bus](./media/send-related-messages-sequential-convoy/provide-service-bus-connection-string.png)

        > [!TIP]
        > Si no tiene esta cadena de conexión, obtenga información sobre cómo [buscar y copiar la cadena de conexión del espacio de nombres de Service Bus](#permissions-connection-string).

   * Para seleccionar un espacio de nombres de Service Bus de la suscripción actual a Azure, siga estos pasos:

     1. En **Nombre de la conexión**, especifique un nombre para la conexión. En **Espacio de nombres de Service Bus**, seleccione el espacio de nombres de Service Bus, por ejemplo:

        ![Especificación del nombre de conexión y selección del espacio de nombres de Service Bus](./media/send-related-messages-sequential-convoy/create-service-bus-connection.png)

     1. Cuando aparezca el siguiente panel, seleccione la directiva de Service Bus y, luego, **Crear**.

        ![Selección de la directiva de Service Bus y, a continuación, "Crear"](./media/send-related-messages-sequential-convoy/create-service-bus-connection-2.png)

1. Seleccione **Continuar** cuando haya terminado.

   El Diseñador de aplicaciones lógicas ahora muestra la plantilla **Entrega ordenada y correlacionada mediante sesiones de Service Bus**, que contiene un flujo de trabajo previamente rellenado con un desencadenador y acciones, incluidos dos ámbitos que implementan control de errores que sigue el patrón `Try-Catch`.

Ahora puede obtener más información sobre el desencadenador y las acciones de la plantilla, o ir directamente a [proporcionar los valores para la plantilla de aplicación lógica](#complete-template).

<a name="template-summary"></a>

## <a name="template-summary"></a>Resumen de plantilla

Este es el flujo de trabajo general en la plantilla **Entrega ordenada y correlacionada mediante sesiones de Service Bus** cuando se contraen los detalles:

![Flujo de trabajo general de la plantilla](./media/send-related-messages-sequential-convoy/template-top-level-flow.png)

| Nombre | Descripción |
|------|-------------|
| **`When a message is received in a queue (peek-lock)`** | En función de la periodicidad especificada, este desencadenador de Service Bus comprueba si hay mensajes en la cola de Service Bus especificada. Si existe algún mensaje en la cola, el desencadenador se activa para crear y ejecutar una instancia de flujo de trabajo. <p><p>El término *bloque de inspección* significa que el desencadenador envía una solicitud para recuperar un mensaje de la cola. Si existe un mensaje, el desencadenador lo recupera y bloquea para que no se procese nuevamente el mensaje hasta que expire el período de bloqueo. Para obtener más información, [inicialice la sesión](#initialize-session). |
| **`Init isDone`** | Esta [acción **Inicializar variable**](../logic-apps/logic-apps-create-variables-store-values.md#initialize-variable) crea una variable booleana que se establece en `false` e indica cuando se cumplen las condiciones siguientes: <p><p>- No hay más mensajes disponibles para lectura en la sesión. <br>- Ya no es necesario renovar el bloqueo de sesión, por lo que la instancia de flujo de trabajo actual puede finalizar. <p><p>Para obtener más información, consulte [Inicialización de la sesión](#initialize-session). |
| **`Try`** | Esta [acción de **Ámbito**](../logic-apps/logic-apps-control-flow-run-steps-group-scopes.md) contiene las acciones que se ejecutan para procesar un mensaje. Si se produce un problema en el ámbito `Try`, la siguiente acción de **ámbito** `Catch` controla ese problema. Para obtener más información, consulte [Ámbito "try"](#try-scope). |
| **`Catch`**| Esta [acción de **Ámbito**](../logic-apps/logic-apps-control-flow-run-steps-group-scopes.md) contiene las acciones que se ejecutan si se produce un problema en el ámbito `Try` anterior. Para obtener más información, consulte [Ámbito "catch"](#catch-scope). |
|||

<a name="try-scope"></a>

### <a name="try-scope"></a>Ámbito "Try"

Este es el flujo general de la [acción de ámbito](../logic-apps/logic-apps-control-flow-run-steps-group-scopes.md) `Try` cuando se contraen los detalles:

![Flujo de trabajo de la acción de ámbito "Try"](./media/send-related-messages-sequential-convoy/try-scope-action.png)

| Nombre | Descripción |
|------|-------------|
| **`Send initial message to topic`** | Puede reemplazar esta acción por cualquier otra que quiera para controlar el primer mensaje de la sesión en la cola. El id. de sesión especifica la sesión. <p><p>En el caso de esta plantilla, una acción de Service Bus envía el primer mensaje a un tema de Service Bus. Para obtener más información, consulte [Control del mensaje inicial](#handle-initial-message). |
| (rama paralela) | Esta [acción de rama paralela](../logic-apps/logic-apps-control-flow-branches.md) crea dos rutas de acceso: <p><p>- Rama n.º 1: continúa el procesamiento del mensaje. Para obtener más información, consulte [Rama n.º 1: Completar el mensaje inicial de la cola](#complete-initial-message). <p><p>- Rama n.º 2: abandona el mensaje si se produce algún error y lo libera para que lo use otra ejecución de desencadenador. Para obtener más información, consulte[Rama n.º 2: Abandonar el mensaje inicial de la cola](#abandon-initial-message). <p><p>Ambas rutas se unen más adelante en la acción **Close session in a queue and succeed** (Cerrar la sesión de una cola y finalizar correctamente), que se describe en la siguiente fila. |
| **`Close a session in a queue and succeed`** | Esta acción de Service Bus combina las ramas descritas anteriormente y cierra la sesión de la cola después de que se produzca cualquiera de los siguientes eventos: <p><p>- El flujo de trabajo termina de procesar los mensajes disponibles en la cola. <br>- El flujo de trabajo abandona el mensaje inicial porque se produjo un error. <p><p>Para obtener más información, consulte [Cerrar la sesión de una cola y finalizar correctamente](#close-session-succeed). |
|||

<a name="complete-initial-message"></a>

#### <a name="branch-1-complete-initial-message-in-queue"></a>Rama n.º 1: Completar el mensaje inicial de la cola

| Nombre | Descripción |
|------|-------------|
| `Complete initial message in queue` | Esta acción de Service Bus marca un mensaje recuperado correctamente como completo y quita el mensaje de la cola para evitar que vuelva a procesarse. Para obtener más información, consulte [Control del mensaje inicial](#handle-initial-message). |
| `While there are more messages for the session in the queue` | Este [bucle **Until**](../logic-apps/logic-apps-control-flow-loops.md#until-loop) sigue obteniendo mensajes mientras exista alguno o hasta que pase una hora. Para obtener más información sobre las acciones de este bucle, consulte [Mientras haya más mensajes para la sesión en la cola](#while-more-messages-for-session). |
| **`Set isDone = true`** | Cuando no hay más mensajes, esta [acción **Establecer variable**](../logic-apps/logic-apps-create-variables-store-values.md#set-variable) establece `isDone` en `true`. |
| **`Renew session lock until cancelled`** | Este [bucle **Until**](../logic-apps/logic-apps-control-flow-loops.md#until-loop) se asegura de que la aplicación lógica mantiene el bloqueo de sesión mientras haya mensajes o hasta que pase una hora. Para obtener más información sobre las acciones de este bucle, consulte [Renovación del bloqueo de sesión hasta que se cancele](#renew-session-while-messages-exist). |
|||

<a name="abandon-initial-message"></a>

#### <a name="branch-2-abandon-initial-message-from-the-queue"></a>Rama n.º 2: Abandonar el mensaje inicial de la cola

Si se produce un error en la acción que controla el primer mensaje, la acción de Service Bus **Abandon initial message from the queue** (Abandonar el mensaje inicial de la cola) libera el mensaje para que otra ejecución de instancia del flujo de trabajo la procese. Para obtener más información, consulte [Control del mensaje inicial](#handle-initial-message).

<a name="catch-scope"></a>

### <a name="catch-scope"></a>Ámbito "Catch"

Si se produce un error en las acciones del ámbito `Try`, la aplicación lógica todavía debe cerrar la sesión. La [acción de ámbito](../logic-apps/logic-apps-control-flow-run-steps-group-scopes.md) `Catch` se ejecuta cuando la acción del ámbito `Try` produce el estado `Failed`, `Skipped` o `TimedOut`. El ámbito devuelve un mensaje de error que incluye el id. de sesión en el que se produjo el problema y finaliza la aplicación lógica.

Este es el flujo general de la acción de ámbito `Catch` cuando se contraen los detalles:

![Flujo de trabajo de la acción de ámbito "Catch"](./media/send-related-messages-sequential-convoy/catch-scope-action.png)

| Nombre | Descripción |
|------|-------------|
| **`Close a session in a queue and fail`** | Esta acción de Service Bus cierra la sesión en la cola para que el bloqueo de sesión no se mantenga abierta. Para obtener más información, consulte [Cerrar la sesión de una cola y finalizar con errores](#close-session-fail). |
| **`Find failure msg from 'Try' block`** | Esta [acción **Filtrar matriz**](../logic-apps/logic-apps-perform-data-operations.md#filter-array-action) crea una matriz a partir de las entradas y salidas de todas las acciones dentro del ámbito `Try` en función de los criterios especificados. En este caso, esta acción devuelve los resultados de las acciones que dieron como resultado el estado `Failed`. Para obtener más información, consulte [Búsqueda del mensaje de error en el bloque "Try"](#find-failure-message). |
| **`Select error details`** | Esta [acción **Seleccionar**](../logic-apps/logic-apps-perform-data-operations.md#select-action) crea una matriz que contiene objetos JSON basados en los criterios especificados. Estos objetos JSON se crean a partir de los valores de la matriz que creó la acción anterior, `Find failure msg from 'Try' block`. En este caso, esta acción devuelve una matriz que contiene un objeto JSON que se creó a partir de los detalles de error devueltos por la acción anterior. Para obtener más información, consulte [Seleccionar detalles del error](#select-error-details). |
| **`Terminate`** | Esta [acción **Finalizar**](../logic-apps/logic-apps-workflow-actions-triggers.md#terminate-action) detiene la ejecución del flujo de trabajo, cancela cualquier acción en curso, omite las acciones restantes y devuelve el estado especificado, id. de sesión y el resultado de error de la acción `Select error details`. Para obtener más información, consulte [Finalización de la aplicación lógica](#terminate-logic-app). |
|||

<a name="complete-template"></a>

## <a name="complete-the-template"></a>Relleno de la plantilla

Para proporcionar los valores para el desencadenador y las acciones en la plantilla **Entrega ordenada y correlacionada mediante sesiones de Service Bus**, siga estos pasos. Debe proporcionar todos los valores necesarios, que están marcados con un asterisco ( **\*** ), para poder guardar la aplicación lógica.

<a name="initialize-session"></a>

### <a name="initialize-the-session"></a>Inicialización de la sesión

* En el caso del desencadenador **Cuando se recibe un mensaje en una cola (bloque de inspección)** , proporcione esta información para que la plantilla pueda inicializar una sesión mediante la propiedad **id. de sesión**, por ejemplo:

  ![Detalles del desencadenador de Service Bus "Cuando se recibe un mensaje en una cola (bloque de inspección)"](./media/send-related-messages-sequential-convoy/service-bus-check-message-peek-lock-trigger.png)

  > [!NOTE]
  > En principio, el intervalo de sondeo se establece en tres minutos para que la aplicación lógica no se ejecute con más frecuencia de la esperado y ocasione cargos de facturación imprevistos. Lo ideal es que establezca el intervalo y la frecuencia en 30 segundos para que la aplicación lógica se desencadene inmediatamente cuando llegue un mensaje.

  | Propiedad | Obligatorio para este escenario | Value | Descripción |
  |----------|----------------------------|-------|-------------|
  | **Nombre de la cola** | Sí | <*queue-name*> | Nombre de la cola de Service Bus creada anteriormente. En este ejemplo se usa "Fabrikam-Service-Bus-Queue". |
  | **Tipo de cola** | Sí | **Primario** | Cola de Service Bus principal. |
  | **Id. de sesión** | Sí | **Próximo disponible** | Esta opción obtiene una sesión para cada ejecución del desencadenador, en función del id. de sesión del mensaje en la cola de Service Bus. La sesión también de bloquea para que ninguna otra aplicación lógica u otro cliente pueda procesar los mensajes relacionados con esta sesión. Las siguientes acciones del flujo de trabajo procesan todos los mensajes que están asociados a esa sesión, como se describe más adelante en este artículo. <p><p>A continuación se presenta más información sobre las demás opciones de **id. de sesión**: <p>- **Ninguno**: La opción predeterminada, que no da como resultado ninguna sesión y no se puede usar para implementar el patrón de convoy secuencial. <p>- **Especificar un valor personalizado**: Use esta opción si conoce el id. de sesión que quiere usar y ejecutará el desencadenador de ese id. de sesión. <p>**Nota**: El conector de Service Bus puede guardar un número limitado de sesiones únicas a la vez de Azure Service Bus en la memoria caché del conector. Si el número de sesiones supera este límite, las sesiones antiguas se quitan de la caché. Para obtener más información, consulte [Intercambio de mensajes en la nube con Azure Logic Apps y Azure Service Bus](../connectors/connectors-create-api-servicebus.md#connector-reference). |
  | **Intervalo** | Sí | <*number-of-intervals*> | Número de unidades de tiempo entre repeticiones antes de comprobar la existencia de mensajes. |
  | **Frecuencia** | Sí | **Segundo**, **Minuto**, **Hora**, **Día**, **Semana** o **Mes** | Unidad de tiempo de la periodicidad que se va a usar al comprobar la existencia de mensajes. <p>**Sugerencia**: Para agregar una **Zona horaria** u **Hora de inicio**, seleccione estas propiedades en la lista **Agregar nuevo parámetro**. |
  |||||

  Para más información sobre el desencadenador, consulte [Service Bus: Cuando se recibe un mensaje en una cola (bloque de inspección)](/connectors/servicebus/#when-a-message-is-received-in-a-queue-(peek-lock)). El desencadenador genera un [ServiceBusMessage](/connectors/servicebus/#servicebusmessage).

Después de inicializar la sesión, el flujo de trabajo usa la acción **Inicializar variable** para crear una variable booleana que al principio se establece en `false` e indica cuando se cumplen las condiciones siguientes: 

* No hay más mensajes disponibles para lectura en la sesión.

* Ya no es necesario renovar el bloqueo de sesión, por lo que la instancia de flujo de trabajo actual puede finalizar.

![Detalles de la acción "Inicializar variable" para "Init isDone"](./media/send-related-messages-sequential-convoy/init-is-done-variable.png)

Después, en el bloque **Try**, el flujo de trabajo realiza acciones en el primer mensaje que se lea.

<a name="handle-initial-message"></a>

### <a name="handle-the-initial-message"></a>Control del mensaje inicial

La primera acción es un marcador de posición de acción de Service Bus, **Send initial message to topic** (Enviar el mensaje inicial a un tema), que puede reemplazar por cualquier otra acción que quiera usar para administrar el primer mensaje de la sesión en la cola. El id. de sesión especifica la sesión de origen del mensaje.

El marcador de posición de acción de Service Bus envía el primer mensaje al tema de Service Bus que especifica la propiedad **id. de sesión**. De este modo, todos los mensajes que están asociados a una sesión específica se envían al mismo tema. Todas las propiedades **id. de sesión** de las siguientes acciones en esta plantilla usan el mismo valor de id. de sesión.

![Detalles de la acción de Service Bus "Send initial message to topic" (Enviar el mensaje inicial a un tema)](./media/send-related-messages-sequential-convoy/send-initial-message-to-topic-action.png)

1. En la acción de Service Bus **Complete initial message in queue** (Completar el mensaje inicial de la cola) proporcione el nombre de la cola de Service Bus y deje todos los demás valores de propiedad predeterminados de la acción.

   ![Detalles de la acción de Service Bus "Complete initial message in queue" (Completar el mensaje inicial de la cola)](./media/send-related-messages-sequential-convoy/complete-initial-message-queue.png)

1. En la acción de Service Bus **Abandon initial message from the queue** (Abandonar el mensaje inicial de la cola) proporcione el nombre de la cola de Service Bus y deje todos los demás valores de propiedad predeterminados de la acción.

   ![Detalles de la acción de Service Bus "Abandon initial message from the queue" (Abandonar el mensaje inicial de la cola)](./media/send-related-messages-sequential-convoy/abandon-initial-message-from-queue.png)

A continuación, proporcione la información necesaria para las acciones que se realizan después de **Complete initial message in queue** (Completar el mensaje inicial de la cola). Comience por las acciones del bucle **While there are more messages for the session in the queue** (Mientras haya más mensajes para la sesión en la cola).

<a name="while-more-messages-for-session"></a>

### <a name="while-there-are-more-messages-for-the-session-in-the-queue"></a>Mientras haya más mensajes para la sesión en la cola

Este [bucle **Until**](../logic-apps/logic-apps-control-flow-loops.md#until-loop) ejecuta las acciones mientras existan mensajes en la cola o hasta que pase una hora. Para cambiar el límite de tiempo del bucle, edite el valor de la propiedad **Tiempo de expiración** del bucle.

* Obtenga los mensajes adicionales de la cola mientras haya mensajes.

* Compruebe el número de mensajes restantes. Si todavía hay mensajes, siga procesándolos. Si no hay ningún mensaje, el flujo de trabajo establece la variable `isDone` en `true` y sale del bucle.

![Bucle Until: procesamiento de mensajes mientras están en la cola](./media/send-related-messages-sequential-convoy/while-more-messages-for-session-in-queue.png)

1. Proporcione el nombre de la cola de Service Bus en la acción de Service Bus **Get additional messages from session** (Obtener mensajes adicionales de la sesión). De lo contrario, mantenga todos los demás valores predeterminados de propiedad de la acción.

   > [!NOTE]
   > De forma predeterminada, el número máximo de mensajes se establece en `175`, pero las propiedades de tamaño de mensaje y tamaño máximo de mensaje de Service Bus afectan este límite. Para más información, consulte [Tamaño de mensaje para una cola](../service-bus-messaging/service-bus-quotas.md).

   ![Acción de Service Bus: "Get additional messages from session" (Obtener mensajes adicionales de la sesión)](./media/send-related-messages-sequential-convoy/get-additional-messages-from-session.png)

   Después, el flujo de trabajo se divide en las siguientes ramas paralelas:

   * Si se produce un error durante la comprobación de mensajes adicionales, establezca la variable `isDone` en `true`.

   * La condición **Process messages if we got any** (Procesar los mensajes, si hay alguno) comprueba si el número de mensajes restantes es cero. Si es false y quedan más mensajes, continúa con el procesamiento. Si es true y no hay ningún mensaje, el flujo de trabajo establece la variable `isDone` en `true`.

   ![Condición: Procesar los mensajes, si los hay](./media/send-related-messages-sequential-convoy/process-messages-if-any.png)

   En la sección **Si es falso**, un bucle **For each** procesa cada mensaje en el orden primero en entrar, primero en salir (FIFO). En la **Configuración** del bucle, el valor **Control de simultaneidad** se establece en `1`, por lo que solo se procesa un mensaje cada vez.

   ![Bucle "for each": procesar todos los mensajes de uno en uno](./media/send-related-messages-sequential-convoy/for-each-additional-message.png)

1. Proporcione el nombre de la cola de Service Bus para las acciones de Service Bus **Completar el mensaje en una cola** y **Abandonar el mensaje en una cola**.

   ![Acciones de Service Bus: "Completar el mensaje en una cola" y "Abandonar el mensaje en una cola"](./media/send-related-messages-sequential-convoy/abandon-or-complete-message-in-queue.png)

   Cuando haya finalizado el bucle **While there are more messages for the session in the queue** (Mientras haya más mensajes para la sesión en la cola), el flujo de trabajo establecerá la variable `isDone` en `true`.

A continuación, proporcione la información necesaria para las acciones del bucle **Renew session lock until cancelled** (Renovar bloqueo de sesión hasta que se cancele).

<a name="renew-session-while-messages-exist"></a>

### <a name="renew-session-lock-until-cancelled"></a>Renovar bloqueo de sesión hasta que se cancele

Este [bucle **Until**](../logic-apps/logic-apps-control-flow-loops.md#until-loop) se asegura de que la aplicación lógica mantenga el bloqueo de sesión mientras haya mensajes o hasta que pase una hora al ejecutar estas acciones. Para cambiar el límite de tiempo del bucle, edite el valor de la propiedad **Tiempo de expiración** del bucle.

* Use un retraso de 25 segundos o una cantidad de tiempo menor que la duración del tiempo de expiración de bloqueo para la cola que se está procesando. La duración mínima del bloqueo es de 30 segundos, por lo que el valor predeterminado resulta suficiente. Sin embargo, puede optimizar el número de veces que el bucle se ejecuta si lo ajusta según corresponda.

* Compruebe si la variable `isDone` está establecida en `true`.

  * Si `isDone` no está establecida en `true`, el flujo de trabajo sigue procesando mensajes, por lo que se renovará el bloqueo de la sesión en la cola y se comprobará de nuevo la condición del bucle.

    Debe proporcionar el nombre de la cola de Service Bus en la acción de Service Bus [**Renovar bloqueo de la sesión de una cola**](#renew-lock-on-session).

  * Si `isDone` está establecida en `true`, el flujo de trabajo no renovará el bloqueo de la sesión de la cola y saldrá del bucle.

  ![Bucle Until: "Renew session lock until cancelled" (Renovar bloqueo de sesión hasta que se cancele)](./media/send-related-messages-sequential-convoy/renew-lock-until-session-cancelled.png)

<a name="renew-lock-on-session"></a>

#### <a name="renew-lock-on-the-session-in-a-queue"></a>Renovar bloqueo de la sesión de una cola

Esta acción de Service Bus renueva el bloqueo en la sesión de la cola mientras el flujo de trabajo siga procesando mensajes.

* Proporcione el nombre de la cola de Service Bus en la acción de Service Bus **Renovar bloqueo de la sesión de una cola**.

  ![Acción de Service Bus: "Renovar bloqueo de la sesión de una cola"](./media/send-related-messages-sequential-convoy/renew-lock-on-session-in-queue.png)

A continuación, proporcione la información necesaria para la acción de Service Bus **Close session in a queue and succeed** (Cerrar la sesión de una cola y finalizar correctamente).

<a name="close-session-succeed"></a>

### <a name="close-a-session-in-a-queue-and-succeed"></a>Cerrar la sesión de una cola y finalizar correctamente

Esta acción de Service Bus cierra la sesión en la cola después de que el flujo de trabajo termina de procesar todos los mensajes disponibles en la cola, o cuando el flujo de trabajo abandona el mensaje inicial.

* Proporcione el nombre de la cola de Service Bus en la acción de Service Bus **Close session in a queue and succeed** (Cerrar la sesión de una cola y finalizar correctamente).

  ![Acción de Service Bus: "Close session in a queue and succeed" (Cerrar la sesión de una cola y finalizar correctamente)](./media/send-related-messages-sequential-convoy/close-session-in-queue-succeed.png)

En las secciones siguientes se describen las acciones de la sección `Catch`, que controlan los errores y las excepciones que se producen en el flujo de trabajo.

<a name="close-session-fail"></a>

### <a name="close-a-session-in-a-queue-and-fail"></a>Cerrar la sesión de una cola y finalizar con errores

Esta acción de Service Bus siempre se ejecuta como la primera acción del ámbito `Catch` y cierra la sesión de la cola.

* Proporcione el nombre de la cola de Service Bus en la acción de Service Bus **Close session in a queue and fail** (Cerrar la sesión de una cola y finalizar con errores).

  ![Acción de Service Bus: "Close session in a queue and fail" (Cerrar la sesión de una cola y finalizar con errores)](./media/send-related-messages-sequential-convoy/close-session-in-queue-fail.png)

Después, el flujo de trabajo crea una matriz que tiene las entradas y salidas de todas las acciones del ámbito `Try` para que la aplicación lógica pueda acceder a la información sobre el error que se produjo.

<a name="find-failure-message"></a>

### <a name="find-failure-msg-from-try-block"></a>Búsqueda del mensaje de error en el bloque "Try"

Esta [acción **Filtrar matriz**](../logic-apps/logic-apps-perform-data-operations.md#filter-array-action) crea una matriz que contiene las entradas y salidas de todas las acciones dentro del ámbito `Try` en función de los criterios especificados mediante la [función `result()`](../logic-apps/workflow-definition-language-functions-reference.md#result). En este caso, esta acción devuelve los resultados de las acciones que tienen el estado `Failed` mediante la [función `equals()`](../logic-apps/workflow-definition-language-functions-reference.md#equals) y la [función `item()`](../logic-apps/workflow-definition-language-functions-reference.md#item).

![Acción de filtrado de matriz: Find failure msg from "Try" block (Búsqueda del mensaje de error en el bloque "Try")](./media/send-related-messages-sequential-convoy/find-failure-message.png)

Esta es la definición JSON para esta acción:

```json
"Find_failure_msg_from_'Try'_block": {
   "inputs": {
      "from": "@Result('Try')",
      "where": "@equals(item()['status'], 'Failed')"
   },
   "runAfter": {
      "Close_the_session_in_the_queue_and_fail": [
         "Succeeded"
      ]
   },
   "type": "Query"
},
```

Después, el flujo de trabajo crea una matriz con un objeto JSON que contiene la información de errores en la matriz que devolvió la acción `Find failure msg from 'Try' block`.

<a name="select-error-details"></a>

### <a name="select-error-details"></a>Seleccionar detalles del error

Esta [acción **Seleccionar**](../logic-apps/logic-apps-perform-data-operations.md#select-action) crea una matriz que contiene objetos JSON basados en la matriz de entradas que fue el resultado de la acción anterior, `Find failure msg from 'Try' block`. En concreto, esta acción devuelve una matriz que solo tiene las propiedades especificadas para cada objeto de la matriz. En este caso, la matriz contiene las propiedades de nombre de acción y resultado del error.

![Acción Seleccionar: "Seleccionar detalles del error"](./media/send-related-messages-sequential-convoy/select-error-details.png)

Esta es la definición JSON para esta acción:

```json
"Select_error_details": {
   "inputs": {
      "from": "@body('Find_failure_msg_from_''Try''_block')[0]['outputs']",
      "select": {
         "action": "@item()['name']",
         "errorResult": "@item()"
      }
   },
   "runAfter": {
      "Find_failure_msg_from_'Try'_block": [
         "Succeeded"
      ]
   },
   "type": "Select"
},
```

Después, el flujo de trabajo detiene la ejecución de la aplicación lógica y devuelve el estado de ejecución junto con más información sobre el error que se produjo.

<a name="terminate-logic-app"></a>

### <a name="terminate-logic-app-run"></a>Terminar la ejecución de la aplicación lógica

Esta [acción **Terminar**](../logic-apps/logic-apps-workflow-actions-triggers.md#terminate-action) detiene la ejecución de la aplicación lógica y devuelve `Failed` como estado de ejecución de la aplicación lógica, junto con el id. de sesión y el resultado de errores de la acción `Select error details`.

![Acción Terminar para detener la ejecución de la aplicación lógica](./media/send-related-messages-sequential-convoy/terminate-logic-app-run.png)

Esta es la definición JSON para esta acción:

```json
"Terminate": {
   "description": "This Failure Termination only runs if the Close Session upon Failure action runs - otherwise the LA will be terminated as Success",
   "inputs": {
      "runError": {
         "code": "",
         "message": "There was an error processing messages for Session ID @{triggerBody()?['SessionId']}. The following error(s) occurred: @{body('Select_error_details')['errorResult']}"
         },
         "runStatus": "Failed"
      },
      "runAfter": {
         "Select_error_details": [
            "Succeeded"
         ]
      },
      "type": "Terminate"
   }
},
```

## <a name="save-and-run-logic-app"></a>Guardado y ejecución de la aplicación lógica

Después de completar la plantilla, ya puede guardar la aplicación lógica. En la barra de herramientas del diseñador, seleccione **Save** (Guardar).

Para probar la aplicación lógica, envíe mensajes a la cola de Service Bus. 

## <a name="next-steps"></a>Pasos siguientes

* Más información acerca de las [acciones y los desencadenadores del conector de Service Bus](/connectors/servicebus/).
