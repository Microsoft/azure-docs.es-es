---
title: Conexión a Azure Event Hubs
description: Cree tareas y flujos de trabajo automatizados que supervisen y administren eventos mediante Azure Event Hubs y Azure Logic Apps
services: logic-apps
ms.suite: integration
ms.reviewer: logicappspm
ms.topic: conceptual
ms.date: 04/23/2019
tags: connectors
ms.openlocfilehash: 198a5da63ed90937c53f7f12f3559f15100e8f19
ms.sourcegitcommit: bfeae16fa5db56c1ec1fe75e0597d8194522b396
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/10/2020
ms.locfileid: "88031426"
---
# <a name="monitor-receive-and-send-events-with-azure-event-hubs-and-azure-logic-apps"></a>Supervisión, recepción y envío de eventos con Azure Event Hubs y Azure Logic Apps

En este artículo se muestra cómo puede supervisar y administrar eventos enviados a [Azure Event Hubs](../event-hubs/event-hubs-about.md) desde una aplicación lógica con el conector de Azure Event Hubs. De esta forma, puede crear aplicaciones lógicas que automatizan las tareas y los flujos de trabajo para comprobar, enviar y recibir eventos desde su centro de eventos. Para obtener información técnica específica del conector, consulte la [referencia sobre el conector de Azure Event Hubs](/connectors/eventhubs/)</a>.

## <a name="prerequisites"></a>Prerrequisitos

* Una cuenta y una suscripción de Azure. Si no tiene una suscripción de Azure, [regístrese para obtener una cuenta gratuita de Azure](https://azure.microsoft.com/free/). 

* Un [espacio de nombres de Azure Event Hubs y un centro de eventos](../event-hubs/event-hubs-create.md)

* La aplicación lógica donde quiere acceder al centro de eventos. Para iniciar la aplicación lógica con un desencadenador de Azure Event Hubs, necesita una [aplicación lógica en blanco](../logic-apps/quickstart-create-first-logic-app-workflow.md).
Si no está familiarizado con las aplicaciones lógicas, consulte [¿Qué es Azure Logic Apps?](../logic-apps/logic-apps-overview.md) e [Inicio rápido: Creación de la primera aplicación lógica](../logic-apps/quickstart-create-first-logic-app-workflow.md).

<a name="permissions-connection-string"></a>

## <a name="check-permissions-and-get-connection-string"></a>Comprobación de los permisos y obtención de la cadena de conexión

Para asegurarse de que la aplicación lógica accede a la instancia de Event Hubs, debe tener permisos de administrador y la cadena de conexión para el espacio de nombres de Event Hubs.

1. Inicie sesión en [Azure Portal](https://portal.azure.com).

1. Vaya al *espacio de nombres* de sus instancias de Event Hubs, no a un centro de eventos específico. 

1. En el menú del espacio de nombres, en **Configuración**, seleccione **Directivas de acceso compartido**. En **Notificaciones**, compruebe que tenga permisos de **Administrador** para ese espacio de nombres.

   ![Administrar los permisos del espacio de nombres del Event Hub](./media/connectors-create-api-azure-event-hubs/event-hubs-namespace.png)

1. Si más adelante desea escribir manualmente la información de conexión, obtenga la cadena de conexión para el espacio de nombres de sus instancias de Event Hubs.

   1. En **Directiva**, elija **RootManageSharedAccessKey**.

   1. Busque la cadena de conexión de clave principal. Elija el botón Copiar y guarde la cadena de conexión para su uso posterior.

      ![Copie la cadena de conexión del espacio de nombres de los Event Hubs](media/connectors-create-api-azure-event-hubs/find-event-hub-namespace-connection-string.png)

      > [!TIP]
      > Para confirmar si la cadena de conexión está asociada al espacio de nombres de Event Hubs o a un centro de eventos específico, asegúrese de que la cadena de conexión no tiene el parámetro `EntityPath` . Si encuentra este parámetro, la cadena de conexión es para la "entidad" de una instancia de Event Hubs específica y no es la cadena correcta para utilizar con la aplicación lógica.

1. Ahora, continúe con [Adición de un desencadenador de Event Hubs](#add-trigger) o [Adición de una acción de Event Hubs](#add-action).

<a name="add-trigger"></a>

## <a name="add-event-hubs-trigger"></a>Adición de un desencadenador de Event Hubs

En Azure Logic Apps, cada aplicación lógica debe comenzar con un [desencadenador](../logic-apps/logic-apps-overview.md#logic-app-concepts), que se activa cuando sucede un evento específico o cuando se cumple una condición determinada. Cada vez que el desencadenador se activa, el motor de Logic Apps crea una instancia de aplicación lógica y empieza a ejecutar el flujo de trabajo de la aplicación.

En este ejemplo se muestra cómo puede iniciar un flujo de trabajo de aplicaciones lógicas cuando se envían nuevos eventos al centro de eventos. 

> [!NOTE]
> Todos los desencadenadores de Event Hub son de *sondeo largo*, lo que significa que el desencadenador procesa todos los eventos y espera 30 segundos por partición a que aparezcan más eventos en la instancia de Event Hubs. Por lo tanto, si el desencadenador está configurado con cuatro particiones, este retraso podría ser de hasta dos minutos antes de que el desencadenador termine de sondear todas las particiones. Si no se recibe ningún evento dentro de este retraso, se omite la ejecución del desencadenador. De lo caso contrario, el desencadenador sigue leyendo eventos hasta que su instancia de Event Hubs está vacía. El siguiente sondeo del desencadenador ocurre en función del intervalo de periodicidad especificado en las propiedades del desencadenador.

1. En Azure Portal o Visual Studio, cree una aplicación lógica en blanco, con lo que se abre el diseñador de Logic Apps. En este ejemplo se usa Azure Portal.

1. En el cuadro de búsqueda, escriba "event hubs" como filtro. En la lista de desencadenadores, seleccione este desencadenador: **When events are available in Event Hub - Event Hubs** (Cuando los eventos estén disponibles en el centro de eventos - Event Hubs)

   ![Seleccionar un desencadenador](./media/connectors-create-api-azure-event-hubs/find-event-hubs-trigger.png)

1. Si se le pide indicar los detalles de la conexión, [cree ahora mismo su conexión de Event Hubs](#create-connection). 

1. En el desencadenador, proporcione información sobre el centro de eventos que quiere supervisar. Para ver más propiedades, abra la lista **Agregar nuevo parámetro**. Si selecciona un parámetro, se agrega esa propiedad en la tarjeta de desencadenador.

   ![Propiedades de desencadenador](./media/connectors-create-api-azure-event-hubs/event-hubs-trigger.png)

   | Propiedad | Obligatorio | Descripción |
   |----------|----------|-------------|
   | **Nombre del centro de eventos** | Sí | Nombre del centro de eventos que quiere supervisar. |
   | **Tipo de contenido** | No | Tipo de contenido del evento. El valor predeterminado es `application/octet-stream`. |
   | **Nombre del grupo de consumidores** | No | [Nombre del grupo de consumidores del centro de eventos](../event-hubs/event-hubs-features.md#consumer-groups) para leer los eventos. Si no se especifica, se utiliza el grupo de consumidores predeterminado. |
   | **Número máximo de eventos**. | No | Número máximo de eventos. El desencadenador devuelve entre uno y el número de eventos que especifica esta propiedad. |
   | **Intervalo** | Sí | Entero positivo que describe la frecuencia con la que se ejecuta el flujo de trabajo. |
   | **Frecuencia** | Sí | Unidad de tiempo que se usa para la periodicidad. |
   ||||

   **Propiedades adicionales**

   | Propiedad | Obligatorio | Descripción |
   |----------|----------|-------------|
   | **Esquema de contenido** | No | Esquema de contenido JSON para los eventos que se leerán desde el centro de eventos. Por ejemplo, si especifica el esquema de contenido, puede desencadenar la aplicación lógica solo para los eventos que coinciden con el esquema. |
   | **Clave de partición mínima** | No | Escriba el identificador de la [partición](../event-hubs/event-hubs-features.md#partitions) mínima que va a leer. De forma predeterminada, se leen todas las particiones. |
   | **Clave de partición máxima** | No | Escriba el identificador de la [partición](../event-hubs/event-hubs-features.md#partitions) máxima que va a leer. De forma predeterminada, se leen todas las particiones. |
   | **Zona horaria** | No | Solo se aplica cuando se especifica una hora de inicio porque este desencadenador no acepta diferencia horaria con UTC. Seleccione la zona horaria que desea aplicar. <p>Para obtener más información, consulte [Creación y ejecución de tareas y flujos de trabajo repetitivos con Azure Logic Apps](../connectors/connectors-native-recurrence.md). |
   | **Hora de inicio** | No | Proporcione una hora de inicio con este formato: <p>AAAA-MM-DDThh:mm:ss si selecciona una zona horaria<p>O bien<p>AAAA-MM-DDThh:mm:ssZ si no selecciona una zona horaria<p>Para obtener más información, consulte [Creación y ejecución de tareas y flujos de trabajo repetitivos con Azure Logic Apps](../connectors/connectors-native-recurrence.md). |
   ||||

1. Cuando esté listo, elija **Guardar** en la barra de herramientas del diseñador.

1. Ahora, agregue a la aplicación lógica una o varias acciones, en función de las tareas que desea realizar con los resultados del desencadenador. 

   Por ejemplo, para filtrar eventos basándose en un valor específico, (como una categoría), puede agregar una condición para que la acción **Send event** (Enviar evento) envíe solo los eventos que cumplan la condición. 

<a name="add-action"></a>

## <a name="add-event-hubs-action"></a>Adición de una acción de Event Hubs

En Azure Logic Apps, una [acción](../logic-apps/logic-apps-overview.md#logic-app-concepts) es un paso del flujo de trabajo que sigue a un desencadenador u otra acción. Para este ejemplo, la aplicación lógica comienza con un desencadenador de Event Hubs que comprueba si hay nuevos eventos en su centro de eventos.

1. En Azure Portal o Visual Studio, abra la aplicación lógica el Diseñador de Logic Apps. En este ejemplo se usa Azure Portal.

1. En el desencadenador o acción, elija **Nuevo paso**.

   Para agregar una acción entre los pasos existentes, mueva el mouse sobre la flecha de conexión. 
   Elija el signo más ( **+** ) que aparece y seleccione **Agregar una acción**.

1. En el cuadro de búsqueda, escriba "event hubs" como filtro.
En la lista de acciones, seleccione esta acción: **Send event - Event Hubs** (Enviar evento - Event Hubs)

   ![Seleccione la acción "Enviar evento"](./media/connectors-create-api-azure-event-hubs/find-event-hubs-action.png)

1. Si se le pide indicar los detalles de la conexión, [cree ahora mismo su conexión de Event Hubs](#create-connection). 

1. En la acción, proporcione información sobre los eventos que quiere enviar. Para ver más propiedades, abra la lista **Agregar nuevo parámetro**. Si selecciona un parámetro, se agrega esa propiedad en la tarjeta de acción.

   ![Seleccione el nombre del centro de eventos y proporcione su contenido](./media/connectors-create-api-azure-event-hubs/event-hubs-send-event-action.png)

   | Propiedad | Obligatorio | Descripción |
   |----------|----------|-------------|
   | **Nombre del centro de eventos** | Sí | Centro de eventos donde quiere enviar el evento. |
   | **Contenido** | No | El contenido del evento que quiere enviar |
   | **Propiedades** | No | Las propiedades de la aplicación y los valores para enviar |
   | **Clave de partición** | No | Identificador de la [partición](../event-hubs/event-hubs-features.md#partitions) a la que se va a enviar el evento. |
   ||||

   Por ejemplo, puede enviar la salida desde el desencadenador de Event Hubs a otro centro de eventos:

   ![Ejemplo de evento de envío](./media/connectors-create-api-azure-event-hubs/event-hubs-send-event-action-example.png)

1. Cuando esté listo, elija **Guardar** en la barra de herramientas del diseñador.

<a name="create-connection"></a>

## <a name="connect-to-your-event-hub"></a>Conexión al centro de eventos

[!INCLUDE [Create connection general intro](../../includes/connectors-create-connection-general-intro.md)] 

1. Cuando se le solicite la información de conexión, proporcione estos detalles:

   | Propiedad | Obligatorio | Value | Descripción |
   |----------|----------|-------|-------------|
   | **Nombre de la conexión** | Sí | <*connection-name*> | El nombre que se va a crear para su conexión |
   | **Espacio de nombres de Event Hubs** | Sí | <*event-hubs-namespace*> | Seleccione el espacio de nombres de Event Hubs que desea usar. |
   |||||  

   Por ejemplo:

   ![Creación de una conexión de Event Hubs](./media/connectors-create-api-azure-event-hubs/create-event-hubs-connection-1.png)

   Para escribir de forma manual la cadena de conexión, seleccione **Especificar la información de conexión manualmente**. 
   Obtenga información acerca de [cómo buscar la cadena de conexión](#permissions-connection-string).

2. Seleccione la directiva de Event Hubs que desea usar, si no está seleccionada. Seleccione **Create**.

   ![Creación de la cadena de conexión de Event Hubs, parte 2](./media/connectors-create-api-azure-event-hubs/create-event-hubs-connection-2.png)

3. Después de crear la conexión, continúe con [Adición de un desencadenador de Event Hubs](#add-trigger) o [Adición de una acción de Event Hubs](#add-action).

## <a name="connector-reference"></a>Referencia de conectores

Para obtener datos técnica, como los desencadenadores, las acciones y los límites, tal como lo describe el archivo Swagger del conector, consulte la [página de referencia del conector](/connectors/eventhubs/).

> [!NOTE]
> En el caso de las aplicaciones lógicas de un [entorno de servicio de integración (ISE)](../logic-apps/connect-virtual-network-vnet-isolated-environment-overview.md), la versión con la etiqueta ISE de este conector usa en su lugar los [límites de mensajes de ISE](../logic-apps/logic-apps-limits-and-config.md#message-size-limits).

## <a name="next-steps"></a>Pasos siguientes

* Obtenga más información sobre otros [conectores de Logic Apps](../connectors/apis-list.md)
