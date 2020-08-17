---
title: 'Inicio rápido: Envío de eventos personalizados al punto de conexión web (Event Grid y Azure Portal)'
description: 'Inicio rápido: Use Azure Event Grid y Azure Portal para publicar un tema personalizado y suscribirse a eventos de ese tema. Los eventos se controlan mediante una aplicación web.'
ms.date: 07/07/2020
ms.topic: quickstart
ms.openlocfilehash: 592e2d6b7393da8cb55a457b022d6c2358048cfe
ms.sourcegitcommit: e71da24cc108efc2c194007f976f74dd596ab013
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87421049"
---
# <a name="quickstart-route-custom-events-to-web-endpoint-with-the-azure-portal-and-event-grid"></a>Inicio rápido: Enrutamiento de eventos personalizados a puntos de conexión web con Azure Portal y Event Grid

Azure Event Grid es un servicio de eventos para la nube. En este artículo, se usa Azure Portal para crear un tema personalizado, suscribirse al tema y desencadenar el evento para ver el resultado. Por lo general, se envían eventos a un punto de conexión que procesa los datos del evento y realiza acciones. Sin embargo, para simplificar en este artículo, los eventos se envían a una aplicación web que recopila y muestra los mensajes.

## <a name="prerequisites"></a>Requisitos previos
[!INCLUDE [quickstarts-free-trial-note.md](../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [event-grid-register-provider-portal.md](../../includes/event-grid-register-provider-portal.md)]

## <a name="create-a-custom-topic"></a>Creación de un tema personalizado

Un tema de cuadrícula de eventos proporciona un punto de conexión definido por el usuario en el que se registran los eventos. 

1. Inicie sesión en el [portal de Azure](https://portal.azure.com/).
2. En la barra de búsqueda del tema, escriba **Temas de Event Grid** y, a continuación, seleccione **Temas de Event Grid** en la lista desplegable. 

    :::image type="content" source="./media/custom-event-quickstart-portal/select-event-grid-topics.png" alt-text="Búsqueda y selección de temas de Event Grid":::
3. En la página **Temas de Event Grid**, seleccione **+ Agregar** en la barra de herramientas. 

    :::image type="content" source="./media/custom-event-quickstart-portal/add-event-grid-topic-button.png" alt-text="Incorporación del botón Tema de Event Grid":::
4. En la página **Crear tema**, siga estos pasos:
    1. Selección la **suscripción**de Azure.
    2. Seleccione un grupo de recursos existente o **Crear nuevo** y escriba un **nombre** para el **grupo de recursos**.
    3. Escriba un **nombre** único para el tema personalizado. El nombre del tema debe ser único porque se representa mediante una entrada DNS. No use el nombre que se muestra en la imagen. En su lugar, cree su propio nombre: debe tener entre 3 y 50 caracteres y contener solo los valores a-z, A-Z, 0-9 y "-".
    4. Seleccione una **ubicación** para el tema de Event Grid.
    5. En la parte inferior de la página, seleccione **Revisar y crear**. 

        :::image type="content" source="./media/custom-event-quickstart-portal/create-custom-topic.png" alt-text="Página Crear tema":::
    6. En la pestaña **Revisar y crear** de la página **Crear tema**, seleccione **Crear**. 
    
        :::image type="content" source="./media/custom-event-quickstart-portal/review-create-page.png" alt-text="Revisión de la configuración y creación":::
5. Una vez finalizada la implementación, escriba **Temas de Event Grid** en la barra de búsqueda de nuevo y seleccione **Temas de Event Grid** en la lista desplegable como hizo anteriormente. 
6. Seleccione en la lista el tema que creó. 

    :::image type="content" source="./media/custom-event-quickstart-portal/select-event-grid-topic.png" alt-text="Selección del tema en la lista":::

7. Ahora verá la página **Tema de Event Grid** del tema. Mantenga esta página abierta. La usará más adelante en este inicio rápido. 

    :::image type="content" source="./media/custom-event-quickstart-portal/event-grid-topic-home-page.png" alt-text="Página principal Tema de Event Grid":::

## <a name="create-a-message-endpoint"></a>Creación de un punto de conexión de mensaje
Antes de crear una suscripción para el tema personalizado, cree un punto de conexión para el mensaje de evento. Normalmente, el punto de conexión realiza acciones en función de los datos del evento. Para simplificar esta guía de inicio rápido, se implementa una [aplicación web pregenerada](https://github.com/Azure-Samples/azure-event-grid-viewer) que muestra los mensajes de los eventos. La solución implementada incluye un plan de App Service, una aplicación web de App Service y el código fuente desde GitHub.

1. En la página del artículo, seleccione **Implementar en Azure** para implementar la solución en su suscripción. En Azure Portal, proporcione valores para los parámetros.

   <a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure-Samples%2Fazure-event-grid-viewer%2Fmaster%2Fazuredeploy.json" target="_blank"><img src="https://azuredeploy.net/deploybutton.png"  alt="Button to Deploy to Aquent." /></a>
1. La implementación puede tardar unos minutos en completarse. Después de que la implementación se haya realizado correctamente, puede ver la aplicación web para asegurarse de que se está ejecutando. En un explorador web, vaya a: `https://<your-site-name>.azurewebsites.net`

    Si se produce un error en la implementación, compruebe el mensaje de error. Puede deberse a que ya se ha utilizado el nombre del sitio web. Vuelva a implementar la plantilla y elija un nombre diferente para el sitio. 
1. Verá el sitio, pero aún no se ha publicado en él ningún evento.

   ![Visualización del nuevo sitio](./media/custom-event-quickstart-portal/view-site.png)

## <a name="subscribe-to-custom-topic"></a>Suscripción a un tema personalizado

Suscríbase a un tema de cuadrícula de eventos que indique a Event Grid los eventos cuyo seguimiento desea realizar y el lugar al que deben enviarse los eventos.

1. Ahora, en la página **Tema de Event Grid** de su tema personalizado, seleccione **+ Suscripción de eventos** en la barra de herramientas.

    :::image type="content" source="./media/custom-event-quickstart-portal/new-event-subscription.png" alt-text="Botón Agregar una suscripción a evento":::
2. En la página **Crear suscripción de eventos**, siga estos pasos:
    1. Escriba un **nombre** para la suscripción a eventos.
    3. Seleccione **Webhook** como **Tipo de punto de conexión**. 
    4. Elija **Seleccionar un punto de conexión**. 

        :::image type="content" source="./media/custom-event-quickstart-portal/provide-subscription-values.png" alt-text="Proporcionar valores de suscripción a eventos":::
    5. Para el punto de conexión de webhook, proporcione la dirección URL de la aplicación web y agregue `api/updates` a la dirección URL de la página principal. Seleccione **Confirm Selection** (Confirmar selección).

        :::image type="content" source="./media/custom-event-quickstart-portal/provide-endpoint.png" alt-text="Proporcionar la dirección URL del punto de conexión":::
    6. De nuevo en la página **Crear suscripción de eventos**, seleccione **Crear**.

3. Vuelva a la aplicación web y observe que se ha enviado un evento de validación de suscripción. Seleccione el icono del ojo para expandir los datos del evento. Event Grid envía el evento de validación para que el punto de conexión pueda verificar que desea recibir datos de eventos. La aplicación web incluye código para validar la suscripción.

    ![Visualización del evento de suscripción](./media/custom-event-quickstart-portal/view-subscription-event.png)

## <a name="send-an-event-to-your-topic"></a>Envío de un evento al tema

Ahora, vamos a desencadenar un evento para ver cómo Event Grid distribuye el mensaje al punto de conexión. Use la CLI de Azure o PowerShell para enviar un evento de prueba a su tema personalizado. Normalmente, una aplicación o servicio de Azure enviaría los datos del evento.

En el primer ejemplo se utiliza la CLI de Azure. Se obtiene la dirección URL y la clave del tema personalizado, y los datos de evento de ejemplo. Use su nombre de un tema personalizado para `<topic name>`. Se crean datos de evento de ejemplo. El elemento `data` del archivo JSON es la carga del evento. En este campo, puede usar cualquier archivo JSON bien formado. También puede usar el campo de asunto para realizar enrutamiento y filtrado avanzados. CURL es una utilidad que envía solicitudes HTTP.


### <a name="azure-cli"></a>Azure CLI
1. En Azure Portal, seleccione **Cloud Shell**. Cloud Shell se abre en el panel inferior del explorador web. 

    :::image type="content" source="./media/custom-event-quickstart-portal/select-cloud-shell.png" alt-text="Selección del icono de Cloud Shell":::
1. Seleccione **Bash** en la esquina superior izquierda de la ventana Cloud Shell. 

    ![Cloud Shell - Bash](./media/custom-event-quickstart-portal/cloud-shell-bash.png)
1. Ejecute el comando siguiente para obtener el **punto de conexión** para el tema: Después de copiar y pegar el comando, actualice el **nombre del tema** y el **nombre del grupo de recursos** antes de ejecutar el comando. Publicará eventos de ejemplo en este punto de conexión del tema. 

    ```azurecli
    endpoint=$(az eventgrid topic show --name <topic name> -g <resource group name> --query "endpoint" --output tsv)
    ```
2. Ejecute el siguiente comando para obtener la **clave** para el tema personalizado: Después de copiar y pegar el comando, actualice el **nombre del tema** y el **nombre del grupo de recursos** antes de ejecutar el comando. Esta es la clave principal del tema de Event Grid. Para obtener esta clave en Azure Portal, cambie a la pestaña **Claves de acceso** de la página **Tema de Event Grid**. Para poder publicar un evento en un tema personalizado, necesita la clave de acceso. 

    ```azurecli
    key=$(az eventgrid topic key list --name <topic name> -g <resource group name> --query "key1" --output tsv)
    ```
3. Copie la siguiente instrucción con la definición del evento y presione **ENTRAR**. 

    ```json
    event='[ {"id": "'"$RANDOM"'", "eventType": "recordInserted", "subject": "myapp/vehicles/motorcycles", "eventTime": "'`date +%Y-%m-%dT%H:%M:%S%z`'", "data":{ "make": "Ducati", "model": "Monster"},"dataVersion": "1.0"} ]'
    ```
4. Ejecute el siguiente comando de **CURL** para publicar el evento: En el comando, el encabezado `aeg-sas-key` se establece en la clave de acceso que obtuvo anteriormente. 

    ```
    curl -X POST -H "aeg-sas-key: $key" -d "$event" $endpoint
    ```

### <a name="azure-powershell"></a>Azure PowerShell
El segundo ejemplo usa PowerShell para realizar pasos similares.

1. En Azure Portal, seleccione **Cloud Shell** (o vaya a `https://shell.azure.com/`). Cloud Shell se abre en el panel inferior del explorador web. 

    :::image type="content" source="./media/custom-event-quickstart-portal/select-cloud-shell.png" alt-text="Selección del icono de Cloud Shell":::
1. En **Cloud Shell**, seleccione **PowerShell** en la esquina superior izquierda de la ventana Cloud Shell. Vea el la imagen de la ventana **Cloud Shell** del ejemplo en la sección de la CLI de Azure.
2. Establezca las siguientes variables. Después de copiar y pegar cada comando, actualice el **nombre del tema** y el **nombre del grupo de recursos** antes de ejecutar el comando:

    **Grupo de recursos**:
    ```powershell
    $resourceGroupName = "<resource group name>"
    ```

    **Nombre del tema de Event Grid**:    
    ```powershell
    $topicName = "<topic name>"
    ```
3. Ejecute los siguientes comandos para obtener el **punto de conexión** y las **claves** del tema:

    ```powershell
    $endpoint = (Get-AzEventGridTopic -ResourceGroupName $resourceGroupName -Name $topicName).Endpoint
    $keys = Get-AzEventGridTopicKey -ResourceGroupName $resourceGroupName -Name $topicName
    ```
4. Prepare el evento. Copie y ejecute las instrucciones en la ventana Cloud Shell. 

    ```powershell
    $eventID = Get-Random 99999

    #Date format should be SortableDateTimePattern (ISO 8601)
    $eventDate = Get-Date -Format s

    #Construct body using Hashtable
    $htbody = @{
        id= $eventID
        eventType="recordInserted"
        subject="myapp/vehicles/motorcycles"
        eventTime= $eventDate   
        data= @{
            make="Ducati"
            model="Monster"
        }
        dataVersion="1.0"
    }
    
    #Use ConvertTo-Json to convert event body from Hashtable to JSON Object
    #Append square brackets to the converted JSON payload since they are expected in the event's JSON payload syntax
    $body = "["+(ConvertTo-Json $htbody)+"]"
    ```
5. Use el cmdlet **Invoke-WebRequest** para enviar el evento. 

    ```powershell
    Invoke-WebRequest -Uri $endpoint -Method POST -Body $body -Headers @{"aeg-sas-key" = $keys.Key1}
    ```

### <a name="verify-in-the-event-grid-viewer"></a>Comprobación en Visor de Event Grid
Ha desencadenado el evento y Event Grid ha enviado el mensaje al punto de conexión que configuró al realizar la suscripción. Vaya a la aplicación web para ver el evento que acaba de enviar.

:::image type="content" source="./media/custom-event-quickstart-portal/event-grid-viewer-end.png" alt-text="Visor de Event Grid":::

## <a name="clean-up-resources"></a>Limpieza de recursos
Si piensa seguir trabajando con este evento, no limpie los recursos creados en este artículo. De lo contrario, elimine los recursos que ha creado en este artículo.

1. Seleccione **Grupos de recursos** en el menú de la izquierda. Si no lo ve en el menú izquierdo, seleccione **Todos los servicios** en el menú de la izquierda y, después, seleccione **Grupos de recursos**. 

    ![Grupos de recursos](./media/custom-event-quickstart-portal/delete-resource-groups.png)
1. Seleccione el grupo de recursos para iniciar la página **Grupo de recursos**. 
1. Seleccione **Eliminar grupo de recursos** en la barra de herramientas. 
1. Confirme la eliminación; para ello, escriba el nombre del grupo de recursos y seleccione **Eliminar**. 

    El otro grupo de recursos que se ve en la imagen ha sido creado y utilizado por la ventana Cloud Shell. Elimínelo si no va a usar la ventana Cloud Shell más adelante. 

## <a name="next-steps"></a>Pasos siguientes

Ahora que sabe cómo crear suscripciones a temas personalizados y eventos, aprenda más acerca de cómo puede ayudarle Event Grid:

- [Una introducción a Azure Event Grid](overview.md)
- [Enrutamiento de eventos de Blob Storage a un punto de conexión web personalizado](../storage/blobs/storage-blob-event-quickstart.md?toc=%2fazure%2fevent-grid%2ftoc.json)
- [Supervisión de los cambios en máquinas virtuales con Azure Event Grid y Logic Apps](monitor-virtual-machine-changes-event-grid-logic-app.md)
- [Transmisión de macrodatos a un almacén de datos](event-grid-event-hubs-integration.md)
