---
title: Intercambio de mensajes con Azure Service Bus
description: Creación de tareas y flujos de trabajo automatizados que envíen y reciban mensajes mediante Azure Service Bus en Azure Logic Apps
services: logic-apps
ms.suite: integration
ms.reviewer: logicappspm
ms.topic: conceptual
ms.date: 07/31/2020
tags: connectors
ms.openlocfilehash: 768186d4b1cf9ac62d4ffdb0af8fdb3df04e9b19
ms.sourcegitcommit: f988fc0f13266cea6e86ce618f2b511ce69bbb96
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87461641"
---
# <a name="exchange-messages-in-the-cloud-by-using-azure-logic-apps-and-azure-service-bus"></a>Intercambio de mensajes en la nube con Azure Logic Apps y Azure Service Bus

Con [Azure Logic Apps](../logic-apps/logic-apps-overview.md) y el conector de [Azure Service Bus](../service-bus-messaging/service-bus-messaging-overview.md) puede crear tareas y flujos de trabajo automatizados que transfieren datos, por ejemplo, ventas y pedidos de compra, diarios y movimientos del inventario, en todas las aplicaciones de la organización. El conector no solo supervisa, envía y administra los mensajes, sino que además realiza acciones con las colas, las sesiones, los temas, las suscripciones, etc., por ejemplo:

* Supervisa los mensajes cuando llegan (autocompletar) o se reciben (bloqueo de información) en las colas, los temas y las suscripciones a los temas.
* Envía mensajes.
* Crea y elimina suscripciones a temas.
* Administra los mensajes de las colas y las suscripciones a temas, por ejemplo, realiza acciones como obtener, obtener los aplazados, completar, aplazar, abandonar y administrar los mensajes fallidos.
* Renueva los bloqueos de mensajes y sesiones en las colas y suscripciones a temas.
* Cierra sesiones en las colas y los temas.

Puede usar desencadenadores que obtengan respuestas de Service Bus y hagan que la salida esté disponible para otras acciones en las aplicaciones lógicas. También puede hacer que otras acciones usen la salida de las de Service Bus. Si no está familiarizado con Service Bus y Logic Apps, revise [Qué es Azure Service Bus](../service-bus-messaging/service-bus-messaging-overview.md) y [Qué es Azure Logic Apps](../logic-apps/logic-apps-overview.md)

[!INCLUDE [Warning about creating infinite loops](../../includes/connectors-infinite-loops.md)]

## <a name="prerequisites"></a>Requisitos previos

* Una cuenta y una suscripción de Azure. Si no tiene una suscripción de Azure, [regístrese para obtener una cuenta gratuita de Azure](https://azure.microsoft.com/free/).

* Un espacio de nombres de Service Bus y una entidad de mensajería, como una cola. Estos elementos y la aplicación lógica deben usar la misma suscripción de Azure. Si no tiene estos elementos, aprenda a [crear el espacio de nombres de Service Bus y una cola](../service-bus-messaging/service-bus-create-namespace-portal.md).

* Conocimientos básicos acerca de [cómo crear aplicaciones lógicas](../logic-apps/quickstart-create-first-logic-app-workflow.md)

* La aplicación lógica donde se usa el espacio de nombres de Service Bus y la entidad de mensajería. La aplicación lógica y el bus de servicio deben usar la misma suscripción de Azure. Para comenzar el flujo de trabajo con un desencadenador de Service Bus, [cree una aplicación lógica en blanco](../logic-apps/quickstart-create-first-logic-app-workflow.md). Para usar una acción de Service Bus en el flujo de trabajo, inicie la aplicación lógica con otro desencadenador, por ejemplo, el de [periodicidad](../connectors/connectors-native-recurrence.md).

<a name="permissions-connection-string"></a>

## <a name="check-permissions"></a>Comprobación de los permisos

Confirme que la aplicación lógica tiene permiso para acceder al espacio de nombres de Service Bus.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con su cuenta de Azure.

1. Vaya al *espacio de nombres* de Service Bus. En la página del espacio de nombres, en **Configuración**, seleccione **Directivas de acceso compartido**. En **Notificaciones**, compruebe que tenga permisos de **Administrador** para ese espacio de nombres.

   ![Administración de permisos para el espacio de nombres de Service Bus](./media/connectors-create-api-azure-service-bus/azure-service-bus-namespace.png)

1. Obtenga la cadena de conexión para el espacio de nombres de Service Bus. La necesita al proporcionar la información de conexión en la aplicación lógica.

   1. En el panel **Directivas de acceso compartido**, seleccione **RootManageSharedAccessKey**.

   1. Al lado de la cadena de conexión principal, elija el botón de copia. Guarde la cadena de conexión para usarla más adelante.

      ![Copia de la cadena de conexión del espacio de nombres de Service Bus](./media/connectors-create-api-azure-service-bus/find-service-bus-connection-string.png)

   > [!TIP]
   > Para confirmar si la cadena de conexión está asociada al espacio de nombres de Service Bus o a una entidad de mensajería, como una cola, compruebe la cadena de conexión del parámetro `EntityPath` . Si encuentra este parámetro, la cadena de conexión es para una entidad específica y no es la correcta para la aplicación lógica.

## <a name="add-service-bus-trigger"></a>Adición del desencadenador de Service Bus

[!INCLUDE [Create connection general intro](../../includes/connectors-create-connection-general-intro.md)]

1. Inicie sesión en [Azure Portal](https://portal.azure.com) y abra la aplicación lógica en blanco en el Diseñador de aplicación lógica.

1. En el cuadro de búsqueda, escriba "Azure Service Bus" como filtro. En la lista de desencadenadores, seleccione el desencadenador que desee.

   Por ejemplo, para desencadenar la aplicación lógica cuando un nuevo elemento se envía a una cola de Service Bus, seleccione el desencadenador **Cuando se recibe un mensaje en una cola (autocompletar)** .

   ![Selección de un desencadenador de Service Bus](./media/connectors-create-api-azure-service-bus/select-service-bus-trigger.png)

   Todos los desencadenadores de Service Bus son desencadenadores de *sondeo prolongado*. Esto significa que, cuando se activa un desencadenador, este procesa todos los mensajes y espera 30 segundos a que aparezcan más en la suscripción del tema o la cola. Si no es el caso, se omite la ejecución del desencadenador. De lo contrario, el desencadenador sigue leyendo mensajes hasta que la suscripción del tema o de la cola se queda vacía. El siguiente sondeo del desencadenador se basa en el intervalo de periodicidad especificado en las propiedades del desencadenador.

   Algunos desencadenadores, como **Cuando llegan uno o más mensajes a una cola (autocompletar)** , pueden devolver uno o más mensajes. Cuando se activan estos desencadenadores, devuelven entre uno y el número de mensajes especificados por la propiedad **Recuento máximo de mensajes** del desencadenador.

1. Si el desencadenador se conecta a su espacio de nombres de Service Bus por primera vez, siga estos pasos cuando el Diseñador de aplicación lógica le pida información de conexión.

   1. Proporcione un nombre para la conexión y seleccione el espacio de nombres de Service Bus.

      ![Creación de la conexión de Service Bus, parte 1](./media/connectors-create-api-azure-service-bus/create-service-bus-connection-trigger-1.png)

      Para escribir manualmente la cadena de conexión, elija **Especificar la información de conexión manualmente**. Si no tiene la cadena de conexión, aprenda a [buscar la cadena de conexión](#permissions-connection-string).

   1. Seleccione la directiva de Service Bus y, luego, **Crear**.

      ![Creación de una conexión de Service Bus, parte 2](./media/connectors-create-api-azure-service-bus/create-service-bus-connection-trigger-2.png)

   1. Seleccione la entidad de mensajería que prefiera, como una cola o un tema. En este ejemplo, seleccione la cola de Service Bus.
   
      ![Selección de la cola de Service Bus](./media/connectors-create-api-azure-service-bus/service-bus-select-queue-trigger.png)

1. Proporcione la información necesaria para el desencadenador seleccionado. Para agregar otras propiedades disponibles a la acción, abra la lista **Agregar nuevo parámetro** y seleccione las propiedades que desee.

   Para el desencadenador de este ejemplo, seleccione el intervalo de sondeo y la frecuencia de comprobación de la cola.

   ![Configuración del intervalo de sondeo](./media/connectors-create-api-azure-service-bus/service-bus-trigger-details.png)

   Para más información sobre los desencadenadores y las propiedades disponibles, consulte la [página de referencia](/connectors/servicebus/) del conector.

1. Continúe con la creación de la aplicación lógica y agregue las acciones que quiera.

   Por ejemplo, puede agregar una acción que envíe un correo electrónico cuando llegue un mensaje nuevo. Cuando el desencadenador comprueba la cola y encuentra un nuevo mensaje, la aplicación lógica ejecuta las acciones seleccionadas para él.

## <a name="add-service-bus-action"></a>Adición de una acción de Service Bus

[!INCLUDE [Create connection general intro](../../includes/connectors-create-connection-general-intro.md)]

1. Inicie sesión en [Azure Portal](https://portal.azure.com) y abra la aplicación lógica en el Diseñador de aplicación lógica.

1. En el paso en el que quiera agregar una acción, seleccione **Nuevo paso**.

   También, para agregar una acción entre un paso y otro, mueva el puntero por encima de la flecha entre ellos. Seleccione el signo más ( **+** ) que aparece y elija **Agregar una acción**.

1. En **Elegir una acción**, en el cuadro de búsqueda, escriba "Azure Service Bus" como filtro. En la lista de acciones, seleccione la que desee. 

   En este ejemplo, seleccione la acción **Enviar mensaje**.

   ![Selección de una acción de Service Bus](./media/connectors-create-api-azure-service-bus/select-service-bus-send-message-action.png) 

1. Si la acción se conecta a su espacio de nombres de Service Bus por primera vez, siga estos pasos cuando el Diseñador de aplicación lógica le pida información de conexión.

   1. Proporcione un nombre para la conexión y seleccione el espacio de nombres de Service Bus.

      ![Creación de la conexión de Service Bus, parte 1](./media/connectors-create-api-azure-service-bus/create-service-bus-connection-action-1.png)

      Para escribir manualmente la cadena de conexión, elija **Especificar la información de conexión manualmente**. Si no tiene la cadena de conexión, aprenda a [buscar la cadena de conexión](#permissions-connection-string).

   1. Seleccione la directiva de Service Bus y, luego, **Crear**.

      ![Creación de una conexión de Service Bus, parte 2](./media/connectors-create-api-azure-service-bus/create-service-bus-connection-action-2.png)

   1. Seleccione la entidad de mensajería que prefiera, como una cola o un tema. En este ejemplo, seleccione la cola de Service Bus.

      ![Selección de la cola de Service Bus](./media/connectors-create-api-azure-service-bus/service-bus-select-queue-action.png)

1. Proporcione la información necesaria para la acción seleccionada. Para agregar otras propiedades disponibles a la acción, abra la lista **Agregar nuevo parámetro** y seleccione las propiedades que desee.

   Por ejemplo, seleccione las propiedades **Contenido** y **Tipo de contenido** para agregarlas a la acción. Luego, especifique el contenido del mensaje que quiere enviar.

   ![Indicación del contenido del mensaje y los detalles](./media/connectors-create-api-azure-service-bus/service-bus-send-message-details.png)

   Para más información sobre las acciones disponibles y sus propiedades, consulte la [página de referencia](/connectors/servicebus/) del conector.

1. Continúe con la creación de la aplicación lógica y agregue cualquier otra acción que desee.

   Por ejemplo, puede agregar una acción que envíe un correo electrónico para confirmar que se ha enviado el mensaje.

1. Guarde la aplicación lógica. En la barra de herramientas del diseñador, seleccione **Save** (Guardar).

<a name="sequential-convoy"></a>

## <a name="send-correlated-messages-in-order"></a>Envío de mensajes correlacionados en orden

Cuando necesite enviar mensajes relacionados en un orden específico, puede usar el patrón [*convoy secuencial*](/azure/architecture/patterns/sequential-convoy) mediante el [conector de Azure Service Bus](../connectors/connectors-create-api-servicebus.md). Los mensajes correlacionados tienen una propiedad que define la relación entre esos mensajes, como el identificador de la [sesión](../service-bus-messaging/message-sessions.md) en Service Bus.

Al crear una aplicación lógica, puede seleccionar la plantilla **Entrega por orden correlacionada mediante sesiones de Service Bus**, que implementa el patrón de convoy secuencial. Para más información, vea [Enviar mensajes relacionados en orden](../logic-apps/send-related-messages-sequential-convoy.md).

<a name="connector-reference"></a>

## <a name="connector-reference"></a>Referencia de conectores

El conector de Service Bus puede guardar hasta 1500 sesiones únicas a la vez de un bus de servicio en la memoria caché del conector. Si el número de sesiones supera este límite, las sesiones antiguas se quitan de la caché. Para más información, consulte [Sesiones de mensajes](../service-bus-messaging/message-sessions.md).

Para conocer otros detalles técnicos sobre los desencadenadores, las acciones y los límites, que se detallan en la descripción de Swagger del conector, vea la [página de referencia del conector](/connectors/servicebus/). Para más información sobre la mensajería de Azure Service Bus, consulte [Qué es Azure Service Bus](../service-bus-messaging/service-bus-messaging-overview.md).

## <a name="next-steps"></a>Pasos siguientes

* Obtenga más información sobre otros [conectores de Logic Apps](../connectors/apis-list.md)