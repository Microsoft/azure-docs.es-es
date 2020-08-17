---
title: 'Tutorial: Uso de eventos de IoT Hub para desencadenar Azure Logic Apps'
description: En este tutorial se indica cómo usar el servicio de enrutamiento de eventos de Azure Event Grid para crear procesos automatizados que lleven a cabo acciones de Azure Logic Apps basadas en eventos de IoT Hub.
services: iot-hub, event-grid
author: robinsh
ms.service: iot-hub
ms.topic: tutorial
ms.date: 07/07/2020
ms.author: robinsh
ms.custom: devx-track-azurecli
ms.openlocfilehash: 35359c63b79d9eea6f8f6ad688bd040428a39eb8
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87503453"
---
# <a name="tutorial-send-email-notifications-about-azure-iot-hub-events-using-event-grid-and-logic-apps"></a>Tutorial: Envío de notificaciones por correo electrónico sobre eventos de Azure IoT Hub mediante Event Grid y Logic Apps

Azure Event Grid permite reaccionar a los eventos en IoT Hub mediante el desencadenamiento de acciones en las aplicaciones empresariales de bajada.

En este artículo se ofrece orientación mediante una configuración de ejemplo que usa IoT Hub y Event Grid. Al final, tendrá una aplicación en Azure Logic Apps configurada para enviar una notificación por correo electrónico cada vez que se agregue un dispositivo a la instancia de IoT Hub. 

## <a name="prerequisites"></a>Requisitos previos

* Una suscripción de Azure activa. Si no tiene una suscripción, puede crear [una cuenta gratuita de Azure](https://azure.microsoft.com/pricing/free-trial/).

* Una cuenta de correo electrónico de cualquier proveedor de correo electrónico que sea compatible con Azure Logic Apps, como Office 365 Outlook, Outlook.com o Gmail. Esta cuenta de correo electrónico se usa para enviar las notificaciones de eventos. Puede encontrar una lista completa de los conectores de Logic Apps admitidos en [Introducción a los conectores](/connectors/).

  > [!IMPORTANT]
  > Antes de usar Gmail, compruebe que tenga una cuenta empresarial de G-Suite (dirección de correo electrónico con un dominio personalizado) o una cuenta de consumidor de Gmail (dirección de correo electrónico con @gmail.com o @googlemail.com). Solo las cuentas empresariales de G-Suite pueden usar el conector de Gmail con otros conectores sin restricciones en las aplicaciones lógicas. Si tiene una cuenta de consumidor de Gmail, puede usar el conector de Gmail solo con servicios específicos aprobados por Google, o puede [crear una aplicación cliente de Google para usarla en la autenticación](/connectors/gmail/#authentication-and-bring-your-own-application). Para más información, consulte [Directivas de privacidad y seguridad de datos de los conectores de Google en Azure Logic Apps](../connectors/connectors-google-data-security-privacy-policy.md).

* Una instancia de IoT Hub en Azure. Si aún no ha creado ninguna, vea un tutorial en [Introducción a Azure IoT Hub](../iot-hub/quickstart-send-telemetry-dotnet.md).

## <a name="create-a-logic-app"></a>Creación de una aplicación lógica

En primer lugar, cree una aplicación lógica y agregue un desencadenador de Event Grid que supervise el grupo de recursos de la máquina virtual. 

### <a name="create-a-logic-app-resource"></a>Creación de un recurso de aplicación lógica

1. En [Azure Portal](https://portal.azure.com), seleccione **Crear un recurso** y, a continuación, escriba "aplicación lógica" en el cuadro de búsqueda y presione Entrar. Seleccione **Aplicación lógica** en los resultados.

   ![Creación de la aplicación lógica](./media/publish-iot-hub-events-to-logic-apps/select-logic-app.png)

1. En la pantalla siguiente, seleccione **Crear**. 

1. Asigne un nombre a la aplicación lógica que sea único en su suscripción; a continuación, seleccione la misma suscripción, el grupo de recursos y la ubicación en que se encuentra la instancia de IoT Hub. 

   ![Campos para crear una aplicación lógica](./media/publish-iot-hub-events-to-logic-apps/create-logic-app-fields.png)

1. Seleccione **Crear**.

1. Una vez creado el recurso, vaya a la aplicación lógica. Para ello, seleccione **Grupos de recursos** y, luego, el grupo de recursos que creó para este tutorial. A continuación, busque la aplicación lógica en la lista de recursos y selecciónela. 

1. En el diseñador de aplicaciones lógicas, desplácese hacia abajo para ver **Plantillas**. Elija **Aplicación lógica en blanco** para que pueda crear la aplicación lógica desde el principio.

### <a name="select-a-trigger"></a>Selección de un desencadenador

Un desencadenador es un evento específico que inicia la aplicación lógica. Para este tutorial, el desencadenador que activa el flujo de trabajo recibe una solicitud a través de HTTP.  

1. En la barra de búsqueda de conectores y desencadenadores, escriba **HTTP**.

1. Seleccione **Solicitud: Cuando se recibe una solicitud HTTP** como desencadenador. 

   ![Selección del desencadenador de solicitud HTTP](./media/publish-iot-hub-events-to-logic-apps/http-request-trigger.png)

1. Seleccione **Usar una carga de ejemplo para generar el esquema**. 

   ![Selección del desencadenador de solicitud HTTP](./media/publish-iot-hub-events-to-logic-apps/sample-payload.png)

1. Pegue el siguiente código JSON de ejemplo en el cuadro de texto y después seleccione **Listo**:

   ```json
   [{
     "id": "56afc886-767b-d359-d59e-0da7877166b2",
     "topic": "/SUBSCRIPTIONS/<subscription ID>/RESOURCEGROUPS/<resource group name>/PROVIDERS/MICROSOFT.DEVICES/IOTHUBS/<hub name>",
     "subject": "devices/LogicAppTestDevice",
     "eventType": "Microsoft.Devices.DeviceCreated",
     "eventTime": "2018-01-02T19:17:44.4383997Z",
     "data": {
       "twin": {
         "deviceId": "LogicAppTestDevice",
         "etag": "AAAAAAAAAAE=",
         "deviceEtag": "null",
         "status": "enabled",
         "statusUpdateTime": "0001-01-01T00:00:00",
         "connectionState": "Disconnected",
         "lastActivityTime": "0001-01-01T00:00:00",
         "cloudToDeviceMessageCount": 0,
         "authenticationType": "sas",
         "x509Thumbprint": {
           "primaryThumbprint": null,
           "secondaryThumbprint": null
         },
         "version": 2,
         "properties": {
           "desired": {
             "$metadata": {
               "$lastUpdated": "2018-01-02T19:17:44.4383997Z"
             },
             "$version": 1
           },
           "reported": {
             "$metadata": {
               "$lastUpdated": "2018-01-02T19:17:44.4383997Z"
             },
             "$version": 1
           }
         }
       },
       "hubName": "egtesthub1",
       "deviceId": "LogicAppTestDevice"
     },
     "dataVersion": "1",
     "metadataVersion": "1"
   }]
   ```

1. Puede recibir una notificación emergente que dice: **Recuerde incluir un encabezado Content-Type establecido en application/json en la solicitud.** Puede ignorar esta sugerencia sin ningún problema y pasar a la sección siguiente. 

### <a name="create-an-action"></a>Creación de una acción

Las acciones son los pasos que se producen después de que el desencadenador inicia el flujo de trabajo de la aplicación lógica. En este tutorial, la acción consiste en enviar una notificación por correo electrónico desde su proveedor de correo electrónico. 

1. Seleccione **Nuevo paso**. Se abrirá una ventana para **elegir una acción**.

1. Busque **Correo electrónico**.

1. En función de su proveedor de correo electrónico, busque y seleccione el conector correspondiente. Este tutorial usa **Office 365 Outlook**. Los pasos para otros proveedores de correo electrónico son similares. 

   ![Selección del conector del proveedor de correo electrónico](./media/publish-iot-hub-events-to-logic-apps/o365-outlook.png)

1. Seleccione la acción **Enviar un correo electrónico**. 

1. Si se le pide, inicie sesión en la cuenta de correo electrónico. 

1. Cree la plantilla de correo electrónico. 

   * **Para**: escriba la dirección de correo electrónico en la que desea recibir los correos electrónicos de notificación. Para este tutorial, use una cuenta de correo electrónico a la que pueda acceder para las pruebas. 

   * **Firmante**: Rellene el texto del asunto. Al hacer clic en el cuadro de texto Asunto, puede seleccionar contenido dinámico para incluirlo. Por ejemplo, en este tutorial se usa `IoT Hub alert: {event Type}`. Si no puede ver el contenido dinámico, seleccione el hipervínculo **Agregar contenido dinámico**: con esto lo puede activar o desactivar.

   * **Cuerpo**: escriba el texto del correo electrónico. Seleccione las propiedades JSON de la herramienta del selector para incluir contenido dinámico basado en los datos de los eventos. Si no puede ver el contenido dinámico, seleccione el hipervínculo **Agregar contenido dinámico** situado en el cuadro de texto **Cuerpo**. Si no aparecen los campos que desea, haga clic en el signo *más* en la pantalla Contenido dinámico para que se incluyan los campos de la acción anterior.

   La plantilla de correo electrónico puede tener un aspecto similar al de este ejemplo:

   ![Rellenar la información del correo electrónico](./media/publish-iot-hub-events-to-logic-apps/email-content.png)

1. Guarde la aplicación lógica. 

### <a name="copy-the-http-url"></a>Copia de la dirección URL HTTP

Antes de cerrar el Diseñador de Logic Apps, copie la dirección URL en la que se realizan las escuchas de un desencadenador de las aplicaciones lógicas. Use esta dirección URL para configurar Event Grid. 

1. Haga clic en el cuadro de configuración del desencadenador **Cuando se recibe una solicitud HTTP** para expandirlo. 

1. Seleccione el botón de copia que se encuentra junto al valor **Dirección URL de HTTP POST** para copiarlo. 

   ![Copia de la dirección URL de HTTP POST](./media/publish-iot-hub-events-to-logic-apps/copy-url.png)

1. Guarde esta dirección URL para poder hacer referencia a ella en la siguiente sección. 

## <a name="configure-subscription-for-iot-hub-events"></a>Configurar la suscripción de eventos de IoT Hub

En esta sección, va a configurar IoT Hube para publicar eventos cuando se produzcan. 

1. En Azure Portal, vaya hasta el centro de IoT. Para ello, seleccione **Grupos de recursos**, a continuación, seleccione el grupo de recursos para este tutorial y, finalmente, seleccione el centro de IoT de la lista de recursos.

2. Seleccione **Eventos**.

   ![Abrir los detalles de Event Grid](./media/publish-iot-hub-events-to-logic-apps/event-grid.png)

3. Seleccione **Suscripción de eventos**. 

   ![Crear una suscripción de eventos](./media/publish-iot-hub-events-to-logic-apps/event-subscription.png)

4. Cree la suscripción de eventos con los siguientes valores: 

    1. En la sección **DETALLES DE SUSCRIPCIONES DE EVENTOS**, realice estas tareas:
        1. Especifique un **nombre** para la suscripción de eventos. 
        2. Seleccione **Esquema de Event Grid** en **Esquema de eventos**. 
   2. En la sección **DETALLES DEL TEMA**, realice las siguientes tareas:
       1. Confirme que en **Tipo de tema** está seleccionado **IoT Hub**. 
       2. Confirme que el nombre del centro de IoT se establece como valor del campo **Recurso de origen**. 
       3. Escriba el nombre del **tema del sistema** que se creará. Para obtener información acerca de los temas del sistema, consulte [Introducción a los temas del sistema](system-topics.md).
   3. En la sección **TIPOS DE EVENTO**, realice las siguientes tareas: 
        1. En **Filtro para tipos de evento**, desactive todas las opciones excepto **Dispositivo creado**.

           ![Tipos de eventos de suscripción](./media/publish-iot-hub-events-to-logic-apps/subscription-event-types.png)
   4. En la sección **DETALLES DE PUNTO DE CONEXIÓN**, realice las siguientes tareas: 
       1. En **Tipo de punto de conexión**, seleccione **Webhook**.
       2. Haga clic en **Seleccionar punto de conexión** y pegue la dirección URL que copió de la aplicación lógica y confirme la selección.

         ![seleccionar dirección URL de punto de conexión](./media/publish-iot-hub-events-to-logic-apps/endpoint-webhook.png)

         Una vez realizadas las acciones anteriores, el panel debe tener un aspecto similar al del ejemplo siguiente: 

        ![Formulario de ejemplo de suscripción de eventos](./media/publish-iot-hub-events-to-logic-apps/subscription-form.png)

5. Puede guardar la suscripción de eventos aquí y recibir notificaciones en todos los dispositivos creados en su instancia de IoT Hub. En este tutorial, sin embargo, se van a usar los campos opcionales para filtrar por dispositivos específicos. Seleccione **Filtros** en la parte superior del panel.

6. Seleccione **Agregar nuevo filtro**. Rellene los campos con estos valores:

   * **Clave**: Seleccione `Subject`.

   * **Operador**: Seleccione `String begins with`.

   * **Valor**:  escriba `devices/Building1_` para filtrar por los eventos de dispositivos del edificio 1.
  
   Agregue otro filtro con estos valores:

   * **Clave**: Seleccione `Subject`.

   * **Operador**: Seleccione `String ends with`.

   * **Valor**: escriba `_Temperature` para filtrar por los eventos de dispositivo relacionados con la temperatura.

   La pestaña **Filtros** de la suscripción al evento se debe parecer a esta imagen:

   ![Incorporación de filtros a suscripción al evento](./media/publish-iot-hub-events-to-logic-apps/event-subscription-filters.png)

7. Seleccione **Crear** para guardar la suscripción de eventos.

## <a name="create-a-new-device"></a>Creación de un dispositivo

Pruebe la aplicación lógica mediante la creación de un dispositivo para desencadenar un correo electrónico de notificación de eventos. 

1. En IoT Hub, seleccione **Dispositivos IoT**. 

2. Seleccione **Nuevo**.

3. En **Id. de dispositivo**, escriba `Building1_Floor1_Room1_Light`.

4. Seleccione **Guardar**. 

5. Puede agregar varios dispositivos con distintos identificadores de dispositivo para probar los filtros de suscripción de eventos. Pruebe estos ejemplos: 

   * Building1_Floor1_Room1_Light
   * Building1_Floor2_Room2_Temperature
   * Building2_Floor1_Room1_Temperature
   * Building2_Floor1_Room1_Light

   Si ha agregado los cuatro ejemplos, la lista de dispositivos IoT se parecerá a la siguiente imagen:

   ![Lista de dispositivos de IoT Hub](./media/publish-iot-hub-events-to-logic-apps/iot-hub-device-list.png)

6. Una vez que haya agregado unos cuantos dispositivos a su instancia de IoT Hub, consulte su correo electrónico para ver cuáles desencadenaron la aplicación lógica. 

## <a name="use-the-azure-cli"></a>Uso de la CLI de Azure

En lugar de usar Azure Portal, puede llevar a cabo los pasos de IoT Hub mediante la CLI de Azure. Para más información, consulte las páginas de la CLI de Azure para [crear una suscripción de eventos](/cli/azure/eventgrid/event-subscription) y [crear un dispositivo IoT](/cli/azure/ext/azure-iot/iot/hub/device-identity).

## <a name="clean-up-resources"></a>Limpieza de recursos

En este tutorial se usaron recursos que generan gastos en su suscripción de Azure. Cuando haya terminado de probar el tutorial y los resultados de las pruebas, deshabilite o elimine los recursos que no desea conservar. 

Para eliminar todos los recursos que ha creado en este tutorial, elimine el grupo de recursos. 

1. Seleccione **Grupos de recursos** y, luego, el grupo de recursos que creó para este tutorial.

2. En la página del grupo de recursos, seleccione **Eliminar grupo de recursos**. Se le pedirá que escriba el nombre del grupo de recursos y, a continuación, podrá eliminarlo. También se quitarán todos los recursos que contiene.

Si no desea quitar todos los recursos, puede administrarlos uno a uno. 

Si no desea perder el trabajo realizado en la aplicación lógica, deshabilítela en lugar de eliminarla. 

1. Vaya a la aplicación lógica.

2. En la hoja **Introducción**, seleccione **Eliminar** o **Deshabilitar**. 

Cada suscripción solo puede tener una instancia de IoT Hub gratuita. Si creó una instancia de IoT Hub gratis para este tutorial, no es necesario eliminarla para evitar cargos.

1. Vaya a su instancia de IoT Hub. 

2. En la página **Introducción**, seleccione **Eliminar**. 

Aunque conserve la instancia de IoT Hub, puede que desee eliminar la suscripción de eventos que creó. 

1. En la instancia de IoT Hub, seleccione **Event Grid**.

2. Seleccione la suscripción de eventos que desea eliminar. 

3. Seleccione **Eliminar**. 

## <a name="next-steps"></a>Pasos siguientes

* Obtenga más información sobre [reacción a eventos de IoT Hub mediante Event Grid para desencadenar acciones](../iot-hub/iot-hub-event-grid.md).
* [Aprenda a ordenar eventos conectados y desconectados de dispositivos](../iot-hub/iot-hub-how-to-order-connection-state-events.md)
* Obtenga información sobre qué más puede hacer con [Event Grid](overview.md).
