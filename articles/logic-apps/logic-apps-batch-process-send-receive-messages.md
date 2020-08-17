---
title: Procesamiento por lotes de mensajes como grupo
description: Enviar y recibir mensajes en grupos entre los flujos de trabajo mediante el procesamiento por lotes en Azure Logic Apps
services: logic-apps
ms.suite: integration
author: divyaswarnkar
ms.author: divswa
ms.reviewer: estfan, jonfan, logicappspm
ms.topic: article
ms.date: 07/31/2020
ms.openlocfilehash: 0985afe3ddfd0d9de3c36ad6b030b6f259708c88
ms.sourcegitcommit: f988fc0f13266cea6e86ce618f2b511ce69bbb96
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87458356"
---
# <a name="send-receive-and-batch-process-messages-in-azure-logic-apps"></a>Envío, recepción y procesamiento por lotes de mensajes en Azure Logic Apps

Para enviar y procesar mensajes juntos de una manera específica como grupos, puede crear una solución de procesamiento por lotes. Esta solución recopila mensajes en un *lote* y espera a que se cumplan los criterios especificados antes de liberar y procesar los mensajes en lote. El procesamiento por lotes puede reducir la frecuencia con la que aplicación lógica procesa los mensajes.

En este artículo se muestra cómo compilar una solución de procesamiento por lotes creando dos aplicaciones lógicas en la misma suscripción y región de Azure, y en este orden:

1. La aplicación lógica ["receptora de lotes"](#batch-receiver), que acepta y recopila los mensajes en un lote hasta que se cumplan los criterios especificados para liberar y procesar esos mensajes. Asegúrese de crear primero este receptor de lotes para que más tarde pueda seleccionar el destino de los lotes cuando cree el remitente de lotes.

1. Una o varias aplicaciones lógicas ["remitente de lotes"](#batch-sender), que envíen los mensajes a la receptora de lotes creada anteriormente.

   También puede especificar una clave única, como un número de cliente, que *particione* o divida el lote de destino en subconjuntos lógicos basándose en esa clave. De este modo, la aplicación receptora puede recopilar todos los elementos con la misma clave y procesarlos juntos.

Asegúrese de que el receptor y el remitente de lotes compartan la misma suscripción *y* región de Azure. Si no es así, no puede seleccionar la receptora de lotes al crear la remitente de lotes, ya que no se verán entre ellas.

## <a name="prerequisites"></a>Prerrequisitos

* Una cuenta y una suscripción de Azure. Si no tiene una suscripción, puede [comenzar con una cuenta de Azure gratuita](https://azure.microsoft.com/free/). También puede [registrarse para obtener una suscripción de pago por uso](https://azure.microsoft.com/pricing/purchase-options/).

* Una cuenta de correo electrónico con cualquier [proveedor de correo electrónico compatible con Azure Logic Apps](../connectors/apis-list.md)

  > [!IMPORTANT]
  > Si quiere usar el conector de Gmail, solo las cuentas empresariales de G-Suite pueden usar este conector sin restricciones en las aplicaciones lógicas. Si tiene una cuenta de consumidor de Gmail, puede usar este conector solo con servicios específicos aprobados por Google o puede [crear una aplicación cliente de Google para usarla en la autenticación con el conector de Gmail](/connectors/gmail/#authentication-and-bring-your-own-application). Para más información, consulte [Directivas de privacidad y seguridad de datos de los conectores de Google en Azure Logic Apps](../connectors/connectors-google-data-security-privacy-policy.md).

* Conocimientos básicos acerca de [cómo crear aplicaciones lógicas](../logic-apps/quickstart-create-first-logic-app-workflow.md)

* Para usar Visual Studio en lugar de Azure Portal, asegúrese de [configurar Visual Studio para trabajar con Logic Apps](../logic-apps/quickstart-create-logic-apps-with-visual-studio.md).

<a name="batch-receiver"></a>

## <a name="create-batch-receiver"></a>Creación de la receptora de lotes

Antes de poder enviar mensajes a un lote, ese lote debe existir como el destino adonde enviar esos mensajes. Primero, debe crear la aplicación lógica "batch receiver", que comienza con el desencadenador **Batch**. De este modo, al crear la aplicación lógica "batch sender", podrá seleccionar la aplicación lógica batch receiver. La receptora de lotes continúa recopilando los mensajes hasta que se cumplan los criterios especificados para liberar y procesar esos mensajes. Si bien las receptoras de lotes no necesitan saber nada sobre las remitentes de lotes, estas últimas sí necesitan conocer el destino adonde enviar los mensajes.

1. En [Azure Portal](https://portal.azure.com) o Visual Studio, cree una aplicación lógica con este nombre: `BatchReceiver`.

1. En el Diseñador de aplicación lógica, agregue el desencadenador **Batch**, que inicia el flujo de trabajo de la aplicación lógica. En el cuadro de búsqueda, escriba `batch` y seleccione este desencadenador: **Mensajes de lote**

   ![Adición del desencadenador "Mensajes de lote"](./media/logic-apps-batch-process-send-receive-messages/add-batch-receiver-trigger.png)

1. Establezca las siguientes propiedades para la receptora de lotes:

   | Propiedad | Descripción |
   |----------|-------------|
   | **Batch Mode** | - **Inline**: para definir los criterios de versión en el desencadenador de lotes <br>- **Cuenta de integración**: para definir varias configuraciones de criterios de lanzamiento a través de una [cuenta de integración](../logic-apps/logic-apps-enterprise-integration-create-integration-account.md). Con una cuenta de integración, puede mantener todas estas configuraciones en un mismo lugar, en lugar de en aplicaciones lógicas independientes. |
   | **Batch Name** | El nombre del lote, que es "TestBatch" en este ejemplo, y solo se aplica al modo por lotes **Inline** |
   | **Release Criteria** | Solo se aplica al modo por lotes **Inline** y selecciona los criterios que deben cumplirse antes de procesar cada lote: <p>- **Basado en el número de mensajes**: se publica el lote en función del número de mensajes recopilados por el lote. <br>- **Basado en el tamaño**: se publica el lote en función del tamaño total en bytes de todos los mensajes recopilados por el lote. <br>- **Programación**: se publica el lote según la periodicidad de la programación, que especifica un intervalo y una frecuencia. En las opciones avanzadas, también puede seleccionar una zona horaria y proporcionar una fecha y hora de inicio. <br>- **Seleccionar todo**: se usan todos los criterios especificados. |
   | **Message Count** | Número de mensajes que se recopilan en el lote, por ejemplo, 10 mensajes. El límite de un lote es de 8000 mensajes. |
   | **Tamaño de lote** | Tamaño total en bytes que se recolectará en el lote, por ejemplo, 10 MB. El límite de tamaño de lote es de 80 MB. |
   | **Programación** | Intervalo y frecuencia entre las publicaciones de lotes, por ejemplo, 10 minutos. La periodicidad mínima es de 60 segundos o 1 minuto. Los minutos fraccionarios se redondean eficazmente hasta 1 minuto. Para especificar una zona horaria o una fecha y hora de inicio, abra la lista **Add new parameter** (Agregar nuevo parámetro) y seleccione las propiedades correspondientes. |
   |||

   > [!NOTE]
   >
   > Si cambia los criterios de publicación mientras el desencadenador aún tiene mensajes procesados por lote sin enviar, el desencadenador utiliza los criterios de publicación actualizados para administrar los mensajes no enviados.

   En el siguiente ejemplo se muestran todos los criterios, pero para realizar sus pruebas, use solo uno:

   ![Proporcionar detalles del desencadenador de lotes](./media/logic-apps-batch-process-send-receive-messages/batch-receiver-criteria.png)

1. Ahora, agregue una o varias acciones que procesen cada lote.

   Para este ejemplo, agregue una acción que envíe un correo electrónico cuando se active el desencadenador de lotes. El desencadenador se ejecuta y envía un correo electrónico cuando el lote tiene 10 mensajes, llega a 10 MB o pasan 10 minutos.

   1. En el desencadenador de lotes, seleccione **New step** (Nuevo paso).

   1. En el cuadro de búsqueda, escriba `send email` como filtro. En función de su proveedor de correo electrónico, seleccione un conector de correo electrónico.

      Por ejemplo, si tiene una cuenta profesional o educativa, como @fabrikam.com o @fabrikam.onmicrosoft.com, seleccione el conector de **Microsoft 365 Outlook**. Si tiene una cuenta personal, como @outlook.com o @hotmail.com, seleccione el conector de **Outlook.com**. En este ejemplo, se usa el conector de Microsoft 365 Outlook.

   1. Seleccione la acción "enviar un correo electrónico" para el proveedor, por ejemplo:

      ![Seleccione la acción "Enviar un correo electrónico" para el proveedor de correo electrónico](./media/logic-apps-batch-process-send-receive-messages/batch-receiver-send-email-action.png)

1. Si se le pide, inicie sesión en la cuenta de correo electrónico.

1. Establezca las propiedades de la acción que agregó.

   * En el cuadro **Para**, escriba la dirección de correo electrónico del destinatario. Para realizar pruebas, puede usar su propia dirección de correo electrónico.

   * En el cuadro **Asunto**, cuando aparezca la lista de contenido dinámico, seleccione el campo **Nombre de la partición**.

     ![En la lista de contenido dinámico, seleccione "Nombre de la partición"](./media/logic-apps-batch-process-send-receive-messages/send-email-action-details.png)

     Más adelante, en el remitente de lotes, puede especificar una clave de partición única que divida el lote de destino en subconjuntos lógicos en los que puede enviar mensajes. Cada conjunto tiene un número único generado por la aplicación lógica remitente de lotes. Esta funcionalidad le permite usar un único lote con varios subconjuntos y definir cada subconjunto con el nombre que proporcione.

     > [!IMPORTANT]
     > Una partición tiene un límite de 5000 mensajes o 80 MB. Si se cumple alguna de estas condiciones, Logic Apps puede liberar el lote, incluso cuando no se cumpla la condición de liberación definida.

   * En el cuadro **Cuerpo**, cuando aparezca la lista de contenido dinámico, seleccione el campo **Id. del mensaje**.

     El Diseñador de aplicación lógica agrega automáticamente un bucle **For each** en torno a la acción para enviar correos electrónicos, debido a que esa acción trata la salida de la acción anterior como una colección, y no como un lote.

     ![En "Cuerpo", seleccione "Id. del mensaje"](./media/logic-apps-batch-process-send-receive-messages/send-email-action-details-for-each.png)

1. Guarde la aplicación lógica. Ahora ha creado una receptora de lotes.

    ![Guardado de la aplicación lógica](./media/logic-apps-batch-process-send-receive-messages/save-batch-receiver-logic-app.png)

   > [!IMPORTANT]
   > Si usa Visual Studio, antes de continuar con la siguiente sección, asegúrese primero de [*implementar* la aplicación lógica de recepción de lotes en Azure](../logic-apps/quickstart-create-logic-apps-with-visual-studio.md#deploy-logic-app-to-azure). En caso contrario, no podrá seleccionar la receptora de lotes cuando cree la remitente de lotes.

<a name="batch-sender"></a>

## <a name="create-batch-sender"></a>Creación de la remitente de lotes

Ahora cree una o más aplicaciones lógicas remitentes de lotes que envíen mensajes a la aplicación lógica receptora de lotes. En cada remitente de lotes, especifique el nombre de la receptora de lotes y del lote, el contenido del mensaje, y el resto de las opciones. Opcionalmente, también puede proporcionar una clave de partición única para dividir el lote en subconjuntos lógicos para recopilar mensajes con esa clave.

* Asegúrese de que ya ha [creado e implementado el receptor de lotes](#batch-receiver), de modo que, cuando cree el remitente de lotes, pueda seleccionar el receptor de lotes existente como lote de destino. Si bien las receptoras de lotes no necesitan saber nada sobre las remitentes de lotes, estas últimas deben saber adonde enviar los mensajes.

* Asegúrese de que el receptor y el remitente de lotes compartan la misma región *y* suscripción de Azure. Si no es así, no puede seleccionar la receptora de lotes al crear la remitente de lotes, ya que no se verán entre ellas.

1. Cree otra aplicación lógica con este nombre: `BatchSender`

   1. En el cuadro de búsqueda, escriba `recurrence` como filtro. En la lista de desencadenadores, seleccione este desencadenador: **Periodicidad**

      ![Adición del desencadenador Periodicidad](./media/logic-apps-batch-process-send-receive-messages/add-schedule-trigger-batch-sender.png)

   1. Establezca el intervalo y la frecuencia para ejecutar la aplicación lógica remitente cada minuto.

      ![Establecer la frecuencia y el intervalo del desencadenador Recurrence](./media/logic-apps-batch-process-send-receive-messages/recurrence-trigger-batch-sender-details.png)

1. Agregue una nueva acción para enviar mensajes a un lote.

   1. En el desencadenador **Recurrence**, seleccione **New step** (Nuevo paso).

   1. En el cuadro de búsqueda, escriba `batch` como filtro y seleccione esta acción: **Choose a Logic Apps workflow with batch trigger**

      ![Seleccione "Choose a Logic Apps workflow with batch trigger"](./media/logic-apps-batch-process-send-receive-messages/send-messages-batch-action.png)

      Aparece una lista que muestra solo las aplicaciones lógicas que tienen desencadenadores Batch y que existen en la misma región *y* suscripción de Azure que la aplicación lógica de envío de lotes.

   1. En la lista de aplicaciones lógicas, seleccione la aplicación lógica receptora de lotes que creó anteriormente.

      ![Seleccionar la aplicación lógica receptora de lotes](./media/logic-apps-batch-process-send-receive-messages/batch-sender-select-batch-receiver.png)

      > [!IMPORTANT]
      > Si usa Visual Studio y no ve ningún receptor de lotes para seleccionar, compruebe que ha creado e implementado previamente el receptor de lotes en Azure. Si no lo ha hecho, aprenda a [implementar la aplicación lógica receptora de lotes en Azure](../logic-apps/quickstart-create-logic-apps-with-visual-studio.md#deploy-logic-app-to-azure).

   1. En la lista de acciones, seleccione esta acción: **Batch_messages - <*su nombre de aplicación lógica*>**

      ![Seleccione esta acción: "Batch_messages - <su-aplicación-lógica>"](./media/logic-apps-batch-process-send-receive-messages/batch-sender-select-batch.png)

1. Establezca las propiedades de la remitente de lotes:

   | Propiedad | Descripción |
   |----------|-------------|
   | **Batch Name** | El nombre de lote definido por la aplicación lógica receptora, `TestBatch` en este ejemplo. <p>**Importante**: el nombre del lote se valida en tiempo de ejecución y debe coincidir con el nombre especificado por la aplicación lógica receptora. Si cambia el nombre del lote, provocará un error en la remitente de lotes. |
   | **Message Content** | El contenido del mensaje que desea enviar |
   |||

   > [!NOTE]
   > Los valores de propiedad **Trigger Name** (Nombre de desencadenador) y **Workflow** (Flujo de trabajo) se rellenan automáticamente a partir de la aplicación lógica seleccionada.

   En este ejemplo, agregue esta expresión, que inserta la fecha y hora actuales en el contenido del mensaje que envía al lote:

   1. Haga clic en el cuadro **Contenido del mensaje**.

   1. Cuando aparezca la lista de contenido dinámico, seleccione **Expression** (Expresión).

   1. Escriba la expresión `utcnow()` y seleccione **OK** (Aceptar).

      ![En "Message Content"(Contenido del mensaje), seleccione "Expression" (Expresión), escriba "utcnow()" y elija "OK" (Aceptar).](./media/logic-apps-batch-process-send-receive-messages/batch-sender-details.png)

1. Ahora debe configurar una partición para el lote. En la acción `BatchReceiver`, abra la lista **Add new parameter** (Agregar nuevo parámetro) y seleccione estas propiedades:

   | Propiedad | Descripción |
   |----------|-------------|
   | **Nombre de la partición** | Una clave de partición única opcional utilizada para dividir el lote de destino en subconjuntos lógicos y recopilar mensajes basándose en esa clave |
   | **Id. de mensaje** | Un identificador de mensaje opcional que es un identificador único global (GUID) generado cuando está vacío |
   |||

   En este ejemplo, en el cuadro **Nombre de la partición** agregue una expresión que genera un número aleatorio entre uno y cinco. Deje vacío el cuadro **Id. de mensaje**.

   1. Haga clic en el cuadro **Nombre de la partición** para que aparezca la lista de contenido dinámico.

   1. En la lista de contenido dinámico, seleccione **Expresión**.

   1. Escriba la expresión `rand(1,6)` y elija **OK** (Aceptar).

      ![Establecer una partición para el lote de destino](./media/logic-apps-batch-process-send-receive-messages/batch-sender-partition-advanced-options.png)

      Esta función **rand** genera un número comprendido entre uno y cinco. Por tanto, va a dividir este lote en cinco particiones numeradas, que esta expresión establece dinámicamente.

1. Guarde la aplicación lógica. La aplicación lógica remitente tiene ahora un aspecto similar al de este ejemplo:

   ![Guardar la aplicación lógica remitente](./media/logic-apps-batch-process-send-receive-messages/batch-sender-finished.png)

## <a name="test-your-logic-apps"></a>Comprobación de las aplicaciones lógicas

Para probar la solución de procesamiento por lotes, deje las aplicaciones lógicas en ejecución durante unos minutos. Pronto, empezará a recibir mensajes de correo electrónico en grupos de cinco, todos con la misma clave de partición.

La aplicación lógica remitente de lotes se ejecuta cada minuto, genera un número aleatorio entre uno y cinco, y utiliza este número generado como clave de partición para el lote de destino donde se envían los mensajes. Cada vez que el lote tiene cinco elementos con la misma clave de partición, la aplicación lógica receptora de lotes se activa y envía los correos electrónicos para cada mensaje.

> [!IMPORTANT]
> Cuando haya terminado las pruebas, asegúrese de que deshabilita la aplicación lógica `BatchSender` para detener el envío de mensajes y evitar la sobrecarga de la bandeja de entrada.

## <a name="next-steps"></a>Pasos siguientes

* [Procesamiento por lotes y envío de mensajes EDI](../logic-apps/logic-apps-scenario-edi-send-batch-messages.md)
* [Generar definiciones de aplicación lógica mediante el uso de JSON](../logic-apps/logic-apps-author-definitions.md)
* [Cree una aplicación sin servidor en Visual Studio con Azure Logic Apps y Functions](../logic-apps/logic-apps-serverless-get-started-vs.md)
* [Control de excepciones y registro de errores para aplicaciones lógicas](../logic-apps/logic-apps-scenario-error-and-exception-handling.md)
