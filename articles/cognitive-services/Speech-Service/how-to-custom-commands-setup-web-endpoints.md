---
title: Configuración de puntos de conexión web (versión preliminar)
titleSuffix: Azure Cognitive Services
description: configure puntos de conexión web para Comandos personalizados
services: cognitive-services
author: xiaojul
manager: yetian
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 06/18/2020
ms.author: xiaojul
ms.openlocfilehash: 0197bb81fdba8bab20742d95aebaa2028bb90c18
ms.sourcegitcommit: 0100d26b1cac3e55016724c30d59408ee052a9ab
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/07/2020
ms.locfileid: "86027688"
---
# <a name="set-up-web-endpoints"></a>Configuración de puntos de conexión web

En este artículo, aprenderá a configurar los puntos de conexión web en una aplicación de Comandos personalizados que le permitirá realizar solicitudes HTTP desde una aplicación cliente. Deberá completar las siguientes tareas:

- Configurar los puntos de conexión web en la aplicación de Comandos personalizados.
- Llamar a puntos de conexión web en la aplicación de Comandos personalizados.
- Recibir la respuesta de los puntos de conexión web. 
- Integrar la respuesta de los puntos de conexión web en una carga JSON personalizada, enviarla y visualizarla desde una aplicación cliente UWP en C# del SDK de Voz.

## <a name="prerequisites"></a>Requisitos previos
> [!div class = "checklist"]
> * [Visual Studio 2019](https://visualstudio.microsoft.com/downloads/)
> * Una clave de suscripción de Azure para el servicio de voz: [Obtenga una gratis](get-started.md) o créela en [Azure Portal](https://portal.azure.com).
> * Una [aplicación de Comandos personalizados creada](quickstart-custom-commands-application.md) previamente.
> * Una aplicación cliente habilitada para el SDK de Voz: [Procedimiento: envío de actividad a una aplicación cliente](./how-to-custom-commands-setup-speech-sdk.md).

## <a name="setup-web-endpoints"></a>Configuración de puntos de conexión web

1. Abra la aplicación de Comandos personalizados que creó anteriormente. 
1. Vaya a "Puntos de conexión web" y haga clic en "New web endpoint" (nuevo punto de conexión web).

   > [!div class="mx-imgBorder"]
   > ![Nuevo punto de conexión web](media/custom-commands/setup-web-endpoint-new-endpoint.png)

   | Configuración | Valor sugerido | Descripción |
   | ------- | --------------- | ----------- |
   | Nombre | UpdateDeviceState | Nombre del punto de conexión web. |
   | URL | https://webendpointexample.azurewebsites.net/api/DeviceState | Dirección URL del punto de conexión con el que quiere que se comunique la aplicación de comandos personalizados. |
   | Método | POST | Interacciones permitidas (como GET, POST) con el punto de conexión.|
   | encabezados | Clave: app. Valor: tomar los primeros 8 dígitos de applicationId | Parámetros de encabezado que se incluirán en el encabezado de solicitud.|

    > [!NOTE]
    > - El punto de conexión web de ejemplo que se creó con [Azure Functions](https://docs.microsoft.com/azure/azure-functions/), que establece un enlace con la base de datos que guarda el estado del dispositivo tanto del televisor como del ventilador.
    > - El encabezado sugerido solo es necesario para el punto de conexión de ejemplo.
    > - Para asegurarnos de que el valor del encabezado es único en nuestro punto de conexión de ejemplo, tome los primeros 8 dígitos de applicationId.
    > - En realidad, el punto de conexión web puede ser el punto de conexión a la instancia de [IoT Hub](https://docs.microsoft.com/azure/iot-hub/about-iot-hub) que administra los dispositivos.

1. Haga clic en **Save**(Guardar).

## <a name="call-web-endpoints"></a>Llamada a los puntos de conexión web

1. Diríjase al comando **TurnOnOff**, seleccione **ConfirmationResponse** en regla de finalización y, a continuación, seleccione **Agregar una acción.**
1. En **Nuevo tipo de acción**, seleccione **Call web endpoint** (llamar a un punto de conexión web).
1. En **Editar acción: puntos de conexión**, seleccione **UpdateDeviceState**, que es el punto de conexión web que hemos creado.  
1. En **Configuración**, introduzca los siguientes valores: 
   > [!div class="mx-imgBorder"]
   > ![Parámetros de la acción para llamar a puntos de conexión web](media/custom-commands/setup-web-endpoint-edit-action-parameters.png)

   | Configuración | Valor sugerido | Descripción |
   | ------- | --------------- | ----------- |
   | Puntos de conexión | UpdateDeviceState | Punto de conexión web que quiere llamar en esta acción. |
   | Parámetros de consulta | item={SubjectDevice}&&value={OnOff} | Parámetros de consulta que se anexarán a la dirección URL del punto de conexión web.  |
   | Contenido del cuerpo | N/D | Contenido del cuerpo de la solicitud. |

    > [!NOTE]
    > - Los parámetros de consulta sugeridos solo son necesarios para el punto de conexión de ejemplo.

1. Desde **En caso de éxito: acción para ejecutar**, seleccione **Send speech response** (enviar respuesta de voz).
    
    En **Simple editor** (Editor sencillo), escriba `{SubjectDevice} is {OnOff}`.
   
   > [!div class="mx-imgBorder"]
   > ![En caso de error de la acción para llamar a puntos de conexión web](media/custom-commands/setup-web-endpoint-edit-action-on-success-send-response.png)

   | Configuración | Valor sugerido | Descripción |
   | ------- | --------------- | ----------- |
   | Acción para ejecutar | Enviar respuesta de voz | Acción que se va a ejecutar si la solicitud al punto de conexión web se realiza correctamente. |
   
   > [!NOTE]
   > - También puede acceder directamente a los campos de la respuesta HTTP mediante `{YourWebEndpointName.FieldName}`. Por ejemplo: `{UpdateDeviceState.TV}`

1. Desde **En caso de error: acción para ejecutar**, seleccione **Send speech response** (enviar respuesta de voz).

    En **Simple editor** (Editor sencillo), escriba `Sorry, {WebEndpointErrorMessage}`.

   > [!div class="mx-imgBorder"]
   > ![En caso de error de la acción para llamar a puntos de conexión web](media/custom-commands/setup-web-endpoint-edit-action-on-fail.png)

   | Configuración | Valor sugerido | Descripción |
   | ------- | --------------- | ----------- |
   | Acción para ejecutar | Enviar respuesta de voz | Acción que se va a ejecutar si hay un error en la solicitud al punto de conexión web. |

   > [!NOTE]
   > - `{WebEndpointErrorMessage}` es opcional. Puede quitarlo si no quiere exponer ningún mensaje de error.
   > - En nuestro punto de conexión de ejemplo, se devuelve una respuesta HTTP con mensajes de error detallados para los errores comunes, tales como los parámetros de encabezado faltantes. 

### <a name="try-it-out-in-test-portal"></a>Prueba en el portal de pruebas
- Respuesta en caso de éxito\
Guarde, entrene y realice pruebas
   > [!div class="mx-imgBorder"]
   > ![En caso de éxito de la acción para llamar a puntos de conexión web](media/custom-commands/setup-web-endpoint-on-success-response.png)
- Respuesta en caso de error\
Quite uno de los parámetros de consulta, guarde, vuelva a entrenar y realice pruebas
   > [!div class="mx-imgBorder"]
   > ![En caso de error de la acción para llamar a puntos de conexión web](media/custom-commands/setup-web-endpoint-on-fail-response.png)

## <a name="integrate-with-client-application"></a>Integración con la aplicación cliente

En [Procedimiento: Envío de actividad a la aplicación cliente (versión preliminar)](./how-to-custom-commands-send-activity-to-client.md), agregó una acción **Send activity to client** (enviar una actividad al cliente). La actividad se envía a la aplicación cliente, independientemente de si la acción **Call web endpoint** (llamar a un punto de conexión web) se realiza correctamente o no.
Sin embargo, en la mayoría de los casos solo querrá enviar la actividad a la aplicación cliente cuando la llamada al punto de conexión web se realice correctamente. En este ejemplo, es cuando el estado del dispositivo se actualizó correctamente.

1. Elimine la acción **Send activity to client** (enviar una actividad al cliente) que agregó anteriormente.
1. Edite la acción de llamada a un punto de conexión web: 
    1. En **Configuración**, asegúrese de que **Parámetros de consulta** sea `item={SubjectDevice}&&value={OnOff}`
    1. Desde **En caso de éxito**, cambie **Acción para ejecutar** por **Send activity to client** (enviar una actividad al cliente).
    1. Copie el código JSON siguiente en el campo **Activity content** (contenido de la actividad).
   ```json
   {
     "type": "event",
     "name": "UpdateDeviceState",
     "state": "{OnOff}",
     "device": "{SubjectDevice}"
   }
   ```
    > [!div class="mx-imgBorder"]
    > ![Envío de actividad en caso de éxito](media/custom-commands/setup-web-endpoint-edit-action-on-success-send-activity.png)
   
Ahora solo enviará la actividad al cliente cuando la solicitud al punto de conexión web se realice correctamente.

### <a name="create-visuals-for-syncing-device-state"></a>Creación de objetos visuales para sincronizar el estado del dispositivo
Agregue el siguiente código XML a `MainPage.xaml` antes del bloque de `"EnableMicrophoneButton"`.

```xml
<Button x:Name="SyncDeviceStateButton" Content="Sync Device State"
        Margin="0,10,10,0" Click="SyncDeviceState_ButtonClicked"
        Height="35"/>
<Button x:Name="EnableMicrophoneButton" ......
        .........../>
```

### <a name="sync-device-state"></a>Sincronización del estado de dispositivo 

En `MainPage.xaml.cs`, agregue la referencia `using Windows.Web.Http;`. Agregue el siguiente código a la clase `MainPage` . Este método enviará una solicitud GET al punto de conexión de ejemplo y extraerá el estado actual del dispositivo para la aplicación. Asegúrese de cambiar `<your_app_name>` por el valor que usó en el **encabezado** del punto de conexión web de comandos personalizados.

```C#
private async void SyncDeviceState_ButtonClicked(object sender, RoutedEventArgs e)
{
    //Create an HTTP client object
    var httpClient = new HttpClient();

    //Add a user-agent header to the GET request. 
    var your_app_name = "<your-app-name>";

    Uri endpoint = new Uri("https://webendpointexample.azurewebsites.net/api/DeviceState");
    var requestMessage = new HttpRequestMessage(HttpMethod.Get, endpoint);
    requestMessage.Headers.Add("app", $"{your_app_name}");

    try
    {
        //Send the GET request
        var httpResponse = await httpClient.SendRequestAsync(requestMessage);
        httpResponse.EnsureSuccessStatusCode();
        var httpResponseBody = await httpResponse.Content.ReadAsStringAsync();
        dynamic deviceState = JsonConvert.DeserializeObject(httpResponseBody);
        var TVState = deviceState.TV.ToString();
        var FanState = deviceState.Fan.ToString();
        await CoreApplication.MainView.CoreWindow.Dispatcher.RunAsync(
            CoreDispatcherPriority.Normal,
            () =>
            {
                State_TV.Text = TVState;
                State_Fan.Text = FanState;
            });
    }
    catch (Exception ex)
    {
        NotifyUser(
            $"Unable to sync device status: {ex.Message}");
    }
}
```

## <a name="try-it-out"></a>Prueba

1. Inicio de la aplicación
1. Haga clic en Sync Device State (sincronizar estado del dispositivo).\
Si probó la aplicación con `turn on tv` en la sección anterior, notará que el televisor aparece como "encendido".
    > [!div class="mx-imgBorder"]
    > ![Sincronización del estado de dispositivo](media/custom-commands/setup-web-endpoint-sync-device-state.png)
1. Seleccione Habilitar micrófono.
1. Seleccionar el botón Hablar.
1. Diga `turn on the fan`.
1. El estado visual del ventilador debe cambiar a "encendido"
    > [!div class="mx-imgBorder"]
    > ![Encender ventilador](media/custom-commands/setup-web-endpoint-turn-on-fan.png)

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Habilite un proceso de CI/CD para su aplicación de Comandos personalizados](./how-to-custom-commands-deploy-cicd.md)