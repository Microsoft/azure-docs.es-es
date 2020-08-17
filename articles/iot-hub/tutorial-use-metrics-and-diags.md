---
title: Configuración y uso de métricas y registros de diagnóstico con una instancia de Azure IoT Hub
description: Aprenda a configurar y usar métricas y registros de diagnóstico con una instancia de Azure IoT Hub. Esto le proporcionará datos para su análisis que le ayudarán a diagnosticar los problemas que pueda tener el centro de conectividad.
author: robinsh
ms.service: iot-hub
services: iot-hub
ms.topic: tutorial
ms.date: 3/13/2019
ms.author: robinsh
ms.custom:
- mvc
- mqtt
- devx-track-azurecli
ms.openlocfilehash: b31fc9df5451665b79a41172286a0cc471b681fd
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87501004"
---
# <a name="tutorial-set-up-and-use-metrics-and-diagnostic-logs-with-an-iot-hub"></a>Tutorial: Configuración y uso de métricas y registros de diagnóstico con una instancia de IoT Hub

Si tiene una solución de IoT Hub que se ejecuta en producción, querrá configurar algunas métricas y habilitar los registros de diagnóstico. Luego, si se produce un problema, tiene datos para examinar que le ayudarán a diagnosticar el problema y corregirlo de manera más rápida. En este artículo, verá cómo habilitar los registros de diagnóstico y comprobarlos para ver si hay errores. También configurará algunas métricas para inspeccionar y alertas que se activarán cuando la métrica alcance un límite determinado. Por ejemplo, se le podría enviar un correo electrónico cuando el número de mensajes de telemetría enviados supere un límite específico, o cuando el número de mensajes usados se acerque a la cuota de mensajes permitidos al día para el centro de IoT. 

Un caso de uso de ejemplo es una gasolinera donde las bombas son dispositivos IoT que se comunican con un centro de IoT. Las tarjetas de crédito se validan y la transacción final se escribe en un almacén de datos. Si los dispositivos IoT dejan de conectarse al centro y de enviar mensajes, es mucho más difícil resolver esta situación si no dispone de ninguna visibilidad sobre lo que pasa.

En este tutorial se usa el ejemplo de Azure del [enrutamiento de IoT Hub](tutorial-routing.md) para enviar mensajes al centro de IoT.

En este tutorial se realizan las siguientes tareas:

> [!div class="checklist"]
> * Mediante la CLI de Azure, crear un centro de IoT, un dispositivo simulado y una cuenta de almacenamiento.  
> * Habilitar los registros de diagnóstico.
> * Habilitar las métricas.
> * Configurar alertas para esas métricas. 
> * Descargar y ejecutar una aplicación que simule un dispositivo IoT que envía mensajes al centro. 
> * Ejecutar la aplicación hasta que empiecen a activarse las alertas. 
> * Ver los resultados de las métricas y comprobar los registros de diagnóstico. 

## <a name="prerequisites"></a>Prerrequisitos

- Suscripción a Azure. Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

- Instale [Visual Studio](https://www.visualstudio.com/). 

- Una cuenta de correo electrónico capaz de recibir correo electrónico.

- Asegúrese de que está abierto el puerto 8883 del firewall. En el dispositivos de ejemplo de este tutorial se usa el protocolo MQTT, que se comunica mediante el puerto 8883. Este puerto puede estar bloqueado en algunos entornos de red corporativos y educativos. Para más información y para saber cómo solucionar este problema, consulte el artículo sobre la [conexión a IoT Hub (MQTT)](iot-hub-mqtt-support.md#connecting-to-iot-hub).


[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

## <a name="set-up-resources"></a>Configuración de recursos

Para los objetivos de este tutorial, necesita un centro de IoT, una cuenta de almacenamiento y un dispositivo IoT simulado. Todos estos recursos se pueden crear con la CLI de Azure o Azure PowerShell. Use el mismo grupo de recursos y la misma ubicación para todos los recursos. Al final, puede quitar todos los elementos en un solo paso mediante la eliminación del grupo de recursos.

Estos son los pasos necesarios.

1. Cree un [grupo de recursos](../azure-resource-manager/management/overview.md). 

2. Cree un Centro de IoT.

3. Cree una cuenta de almacenamiento V1 estándar con replicación Standard_LRS.

4. Cree una identidad de dispositivo para el dispositivo simulado que envíe mensajes al centro. Guarde la clave para la fase de pruebas.

### <a name="set-up-resources-using-azure-cli"></a>Configuración de los recursos mediante la CLI de Azure

Copie y pegue este script en Cloud Shell. Suponiendo que ya haya iniciado sesión, el script se ejecutará línea a línea. Los nuevos recursos se crean en el grupo de recursos ContosoResources.

Las variables que deben ser globalmente únicas llevan concatenado `$RANDOM`. Cuando se ejecuta el script y se establecen las variables, se genera y se concatena una cadena numérica aleatoria al final de la cadena fija, lo que la convierte en única.

```azurecli-interactive

# This is the IOT Extension for Azure CLI.
# You only need to install this the first time.
# You need it to create the device identity. 
az extension add --name azure-iot

# Set the values for the resource names that don't have to be globally unique.
# The resources that have to have unique names are named in the script below
#   with a random number concatenated to the name so you can probably just
#   run this script, and it will work with no conflicts.
location=westus
resourceGroup=ContosoResources
iotDeviceName=Contoso-Test-Device 

# Create the resource group to be used
#   for all the resources for this tutorial.
az group create --name $resourceGroup \
    --location $location

# The IoT hub name must be globally unique, so add a random number to the end.
iotHubName=ContosoTestHub$RANDOM
echo "IoT hub name = " $iotHubName

# Create the IoT hub in the Free tier.
az iot hub create --name $iotHubName \
    --resource-group $resourceGroup \
    --sku F1 --location $location

# The storage account name must be globally unique, so add a random number to the end.
storageAccountName=contosostoragemon$RANDOM
echo "Storage account name = " $storageAccountName

# Create the storage account.
az storage account create --name $storageAccountName \
    --resource-group $resourceGroup \
    --location $location \
    --sku Standard_LRS

# Create the IoT device identity to be used for testing.
az iot hub device-identity create --device-id $iotDeviceName \
    --hub-name $iotHubName 

# Retrieve the information about the device identity, then copy the primary key to
#   Notepad. You need this to run the device simulation during the testing phase.
az iot hub device-identity show --device-id $iotDeviceName \
    --hub-name $iotHubName

```

>[!NOTE]
>Al crear la identidad del dispositivo, puede recibir el siguiente error: *No keys found for policy iothubowner of IoT Hub ContosoTestHub* (No se encontraron claves para la directiva iothubowner del centro de IoT ContosoTestHub). Para corregir este error, actualice la extensión IoT de la CLI de Azure y, luego, vuelva a ejecutar los dos últimos comandos en el script. 
>
>Este es el comando para actualizar la extensión. Ejecútelo en su instancia de Cloud Shell.
>
>```cli
>az extension update --name azure-iot
>```

## <a name="enable-the-diagnostic-logs"></a>Habilitación de los registros de diagnóstico 

Los [registros de diagnóstico](../azure-monitor/platform/platform-logs-overview.md) están deshabilitados de forma predeterminada cuando se crea un centro de IoT. En esta sección, habilitará los registros de diagnóstico para su centro.

1. En primer lugar, si no está aún en su centro en el portal, haga clic en **Grupos de recursos** y seleccione el grupo de recursos Contoso-Resources. Seleccione el centro de la lista de recursos que se muestran. 

2. Busque la sección **Supervisión** en la hoja IoT Hub. Haga clic en **Configuración de diagnóstico**. 

   ![Captura de pantalla que muestra la parte de configuración de diagnóstico de la hoja de IoT Hub.](./media/tutorial-use-metrics-and-diags/01-diagnostic-settings.png)


3. Asegúrese de que la suscripción y el grupo de recursos sean correctos. En **Tipo de recurso**, desactive **Seleccionar todo**, luego, busque y active **IoT Hub**. (Se vuelve a colocar la marca de verificación junto a *Seleccionar todo*; no haga caso). En **Recursos**, seleccione el nombre del centro. La pantalla debería mostrar esta imagen: 

   ![Captura de pantalla que muestra la parte de configuración de diagnóstico de la hoja de IoT Hub.](./media/tutorial-use-metrics-and-diags/02-diagnostic-settings-start.png)

4. Ahora, haga clic en **Activar diagnóstico**. Se muestra el panel de configuración de diagnósticos. Especifique el nombre de la configuración de los registros de diagnóstico como "diags-hub".

5. Seleccione **Archivar en una cuenta de almacenamiento**. 

   ![Captura de pantalla que muestra la configuración de los diagnósticos para archivar en una cuenta de almacenamiento.](./media/tutorial-use-metrics-and-diags/03-diagnostic-settings-storage.png)

    Haga clic en **Configurar** para ver la pantalla **Seleccione una cuenta de almacenamiento**, seleccione la de la derecha (*contosostoragemon*) y haga clic en **Aceptar** para volver al panel de configuración de diagnósticos. 

   ![Captura de pantalla que muestra la configuración de los registros de diagnóstico para archivar en una cuenta de almacenamiento.](./media/tutorial-use-metrics-and-diags/04-diagnostic-settings-after-storage.png)

6. En **LOG**, active **Conexiones** y **Telemetría de dispositivo** y establezca **Retención (días)** en 7 días para cada uno. La pantalla de configuración de diagnóstico se parecerá ahora a la de esta imagen:

   ![Captura de pantalla que muestra la configuración de registro de diagnóstico final.](./media/tutorial-use-metrics-and-diags/05-diagnostic-settings-done.png)

7. Haga clic en **Guardar** para guardar la configuración. Cierre el panel de configuración de diagnóstico.

Más adelante, cuando examine los registros de diagnóstico, podrá ver el registro de conexión y desconexión del dispositivo. 

## <a name="set-up-metrics"></a>Configuración de las métricas 

Ahora, configure algunas métricas para inspeccionar cuándo se envían los mensajes al centro. 

1. En el panel de configuración del centro de IoT, haga clic en la opción **Métricas** de la sección **Supervisión**.

2. En la parte superior de la pantalla, haga clic en **Last 24 hours (Automatic)** (Últimas 24 horas [Automático]). En la lista desplegable que aparece, seleccione **Últimas 4 horas** en **Intervalo de tiempo** y establezca **Granularidad de tiempo** en **1 minuto**, hora local. Haga clic en **Aplicar** para guardar la configuración. 

   ![Captura de pantalla que muestra la configuración de tiempo de las métricas.](./media/tutorial-use-metrics-and-diags/06-metrics-set-time-range.png)

3. Hay una entrada de métrica de forma predeterminada. Deje el grupo de recursos como predeterminado y el espacio de nombres de la métrica. En la lista desplegable **Métrica**, seleccione **Telemetry messages sent** (Mensajes de telemetría enviados). Establezca **Agregación** en **Suma**.

   ![Captura de pantalla que muestra cómo se agrega una métrica para los mensajes de telemetría enviados.](./media/tutorial-use-metrics-and-diags/07-metrics-telemetry-messages-sent.png)


4. Ahora haga clic en **Agregar métrica** para agregar otra métrica al gráfico. Seleccione el grupo de recursos (**ContosoTestHub**). En **Métrica**, seleccione **Total number of messages used** (Número total de mensajes usados). En **Agregación**, seleccione **Promedio**. 

   Ahora la pantalla muestra la métrica minimizada de *Telemetry messages sent* (Mensajes de telemetría enviados), más la nueva métrica de *Total number of messages used* (Número total de mensajes usados).

   ![Captura de pantalla que muestra cómo se agrega una métrica para los mensajes de telemetría enviados.](./media/tutorial-use-metrics-and-diags/07-metrics-num-messages-used.png)

   Haga clic en **Anclar al panel**. Se anclará al panel de Azure Portal, de modo que puede acceder a ella de nuevo. Si no la ancla al panel, no se conserva su configuración.

## <a name="set-up-alerts"></a>Configuración de alertas

Vaya al centro en el portal. Haga clic en **Grupos de recursos**, seleccione *ContosoResources* y, luego, seleccione el centro de IoT *ContosoTestHub*. 

IoT Hub no se ha migrado aún a las [métricas de Azure Monitor](/azure/azure-monitor/platform/data-collection#metrics); para ello, tiene que usar [alertas clásicas](/azure/azure-monitor/platform/alerts-classic.overview).

1. En **Supervisión**, haga clic en **Alertas**; se muestra la pantalla principal de alertas. 

   ![Captura de pantalla que muestra cómo buscar alertas clásicas.](./media/tutorial-use-metrics-and-diags/08-find-classic-alerts.png)

2. Para ir a las alertas clásicas desde aquí, haga clic en **Ver alertas clásicas**. 

    ![Captura de pantalla que muestra la pantalla de alertas clásicas.](./media/tutorial-use-metrics-and-diags/09-view-classic-alerts.png)

    Rellene los campos: 

    **Suscripción**: deje este campo establecido en su suscripción actual.

    **Origen**: establezca este campo en *Métricas*.

    **Grupo de recursos**: establezca este campo en su actual grupo de recursos, *ContosoResources*. 

    **Tipo de recurso**: establezca este campo en IoT Hub. 

    **Recursos**: seleccione su centro de IoT, *ContosoTestHub*.

3. Haga clic en **Agregar una alerta de métrica (clásica)** para configurar una nueva alerta.

    Rellene los campos:

    **Name**: proporcione un nombre para la regla de alerta, como *mensajes-telemetría*.

    **Descripción**: proporcione una descripción de la alerta, como *avisar cuando se hayan enviado 1000 mensajes de telemetría*. 

    **Origen**: establezca esta opción en *Métricas*.

    **Suscripción**, **Grupo de recursos** y **Recursos** se deben establecer en los valores que ha seleccionado en la pantalla **Ver alertas clásicas**. 

    Establezca **Métrica** en *Telemetry messages sent* (Mensajes de telemetría enviados).

    ![Captura de pantalla que muestra cómo configurar una alerta clásica para los mensajes de telemetría enviados.](./media/tutorial-use-metrics-and-diags/10-alerts-add-rule-telemetry-top.png)

4. Después del gráfico, establezca los campos siguientes:

   **Condición**: establézcalo en *Mayor que*.

   **Umbral**: establézcalo en 1000.

   **Período**: establézcalo en *En los últimos 5 minutos*.

   **Destinatarios del correo electrónico de notificación**: ponga aquí la dirección de correo electrónico. 

   ![Captura de pantalla que muestra la mitad inferior de la pantalla de alertas.](./media/tutorial-use-metrics-and-diags/11-alerts-add-rule-bottom.png)

   Haga clic en **Aceptar** para guardar la alerta. 

5. Ahora, configure otra alerta para *Total number of messages used* (Número total de mensajes usados). Esta métrica es útil si quiere enviar una alerta cuando el número de mensajes usado se aproxime a la cuota del centro de IoT; de este modo, sabrá que el centro comenzará pronto a rechazar mensajes.

   En la pantalla **Ver alertas clásicas**, haga clic en **Agregar alerta de métrica (clásica)** y, luego, rellene estos campos en el panel **Agregar regla**.

   **Name**: proporcione un nombre para la regla de alerta, como *número-de-mensajes-usados*.

   **Descripción**: proporcione una descripción de la alerta, como *avisar cuando se acerque a la cuota*.

   **Origen**: establezca este campo en *Métricas*.

    **Suscripción**, **Grupo de recursos** y **Recursos** se deben establecer en los valores que ha seleccionado en la pantalla **Ver alertas clásicas**. 

    Establezca **Métrica** en *Total number of messages used* (Número total de mensajes usados).

6. Debajo del gráfico, rellene los campos siguientes:

   **Condición**: establézcalo en *Mayor que*.

   **Umbral**: establézcalo en 1000.

   **Período**: establezca este campo en *En los últimos 5 minutos*. 

   **Destinatarios del correo electrónico de notificación**: ponga aquí la dirección de correo electrónico. 

   Haga clic en **Aceptar** para guardar la regla. 

5. Ahora debería ver dos alertas en el panel de alertas clásicas: 

   ![Captura de pantalla que muestra la pantalla de alertas clásicas con las nuevas reglas de alerta.](./media/tutorial-use-metrics-and-diags/12-alerts-done.png)

6. Cierre el panel de alertas. 
    
    Con esta configuración, recibirá una alerta cuando el número de mensajes enviados sea mayor que 400 y cuando el número total de mensajes usados supere el NÚMERO.

## <a name="run-simulated-device-app"></a>Ejecución de una aplicación de simulación de dispositivo

Anteriormente, en la sección de configuración del script, configuró un dispositivo de simulación, para lo que usó un dispositivo IoT. En esta sección se descarga una aplicación de consola de .NET que simula un dispositivo que envía mensajes de dispositivo a nube a una instancia de IoT Hub.  

Descargue la solución para la [simulación de un dispositivo IoT](https://github.com/Azure-Samples/azure-iot-samples-csharp/archive/master.zip). Este vínculo descarga un repositorio que contiene varias aplicaciones; la solución que busca está en iot-hub/Tutorials/Routing/.

Haga doble clic en el archivo de la solución (SimulatedDevice.sln) para abrir el código en Visual Studio y, después, abra Program.cs. Sustituya `{iot hub hostname}` por el nombre de host de IoT Hub. El formato del nombre de host de IoT Hub es **{nombre de iot hub} .azure-devices.net**. Para este tutorial, el nombre de host del centro es **ContosoTestHub.azure devices.net**. A continuación, sustituya `{device key}` por la clave de dispositivo que guardó anteriormente al configurar el dispositivo simulado. 

   ```csharp
        static string myDeviceId = "contoso-test-device";
        static string iotHubUri = "ContosoTestHub.azure-devices.net";
        // This is the primary key for the device. This is in the portal. 
        // Find your IoT hub in the portal > IoT devices > select your device > copy the key. 
        static string deviceKey = "{your device key here}";
   ```

## <a name="run-and-test"></a>Ejecución y prueba 

En Program.cs, cambie `Task.Delay` de 1000 a 10, lo que reduce la cantidad de tiempo entre envíos de mensajes de 1 segundo a 0,01 segundos. Al acortar este retraso aumenta el número de mensajes enviados.

```csharp
await Task.Delay(10);
```

Ejecución de la aplicación de consola. Espere unos minutos (10-15). Puede ver los mensajes enviados desde el dispositivo simulado al centro en la pantalla de la consola de la aplicación.

### <a name="see-the-metrics-in-the-portal"></a>Visualización de las métricas en el portal

Abra las métricas desde el panel. Cambie los valores de tiempo a *Últimos 30 minutos* con una granularidad de tiempo de *1 minuto*. Se muestran en el gráfico los mensajes de telemetría enviados y el número total de mensajes usados, con los números más recientes en la parte inferior.

   ![Captura de pantalla que muestra las métricas.](./media/tutorial-use-metrics-and-diags/13-metrics-populated.png)

### <a name="see-the-alerts"></a>Visualización de las alertas

Vuelva a las alertas. Haga clic en **Grupos de recursos**, seleccione *ContosoResources* y, luego, seleccione el centro *ContosoTestHub*. En la página de propiedades que se muestra para el centro, seleccione **Alertas** y, luego, **Ver alertas clásicas**. 

Cuando el número de mensajes enviados supere el límite, comenzará a recibir alertas por correo electrónico. Para ver si hay alguna alerta activa, vaya al centro y seleccione **Alertas**. Se muestran las alertas que están activas, y si hay alguna advertencia. 

   ![Captura de pantalla que muestra las alertas activadas.](./media/tutorial-use-metrics-and-diags/14-alerts-firing.png)

Haga clic en la alerta de los mensajes de telemetría. Se muestra el resultado de la métrica y un gráfico con los resultados. Además, el correo electrónico enviado para avisarle de la activación de la alerta se parece a este:

   ![Captura de pantalla del correo electrónico con las alertas activadas.](./media/tutorial-use-metrics-and-diags/15-alert-email.png)

### <a name="see-the-diagnostic-logs"></a>Visualización de los registros de diagnóstico

Configure los registros de diagnóstico que se exportarán a Blob Storage. Vaya al grupo de recursos y seleccione la cuenta de almacenamiento *contosostoragemon*. Seleccione Blobs y abra el contenedor *insights-logs-connections*. Explore en profundidad hasta llegar a la fecha actual y seleccione el archivo más reciente. 

   ![Captura de pantalla que muestra cómo profundizar en el contenedor de almacenamiento para ver los registros de diagnóstico.](./media/tutorial-use-metrics-and-diags/16-diagnostics-logs-list.png)

Haga clic en **Descargar** para descargarlo y abrirlo. Verá los registros del dispositivo que se conecta y se desconecta a medida que envía mensajes al centro. Este es un ejemplo:

``` json
{ 
  "time": "2018-12-17T18:11:25Z", 
  "resourceId": 
    "/SUBSCRIPTIONS/your-subscription-id/RESOURCEGROUPS/CONTOSORESOURCES/PROVIDERS/MICROSOFT.DEVICES/IOTHUBS/CONTOSOTESTHUB", 
  "operationName": "deviceConnect", 
  "category": "Connections", 
  "level": "Information", 
  "properties": 
      {"deviceId":"Contoso-Test-Device",
       "protocol":"Mqtt",
       "authType":null,
       "maskedIpAddress":"73.162.215.XXX",
       "statusCode":null
       }, 
  "location": "westus"
}
{ 
   "time": "2018-12-17T18:19:25Z", 
   "resourceId": 
     "/SUBSCRIPTIONS/your-subscription-id/RESOURCEGROUPS/CONTOSORESOURCES/PROVIDERS/MICROSOFT.DEVICES/IOTHUBS/CONTOSOTESTHUB", 
    "operationName": "deviceDisconnect", 
    "category": "Connections", 
    "level": "Error", 
    "resultType": "404104", 
    "resultDescription": "DeviceConnectionClosedRemotely", 
    "properties": 
        {"deviceId":"Contoso-Test-Device",
         "protocol":"Mqtt",
         "authType":null,
         "maskedIpAddress":"73.162.215.XXX",
         "statusCode":"404"
         }, 
    "location": "westus"
}
```

## <a name="clean-up-resources"></a>Limpieza de recursos 

Para quitar todos los recursos que ha creado en este tutorial, elimine el grupo de recursos. Esta acción elimina también todos los recursos del grupo. En este caso, se quita el centro de IoT, la cuenta de almacenamiento y el propio grupo de recursos. Si ha anclado métricas al panel, deberá quitarlas manualmente, para lo cual, tiene que hacer clic en los tres puntos de la esquina superior derecha de cada una y seleccionar **Quitar**.

Para quitar el grupo de recursos, use el comando [az group delete](https://docs.microsoft.com/cli/azure/group?view=azure-cli-latest#az-group-delete).

```azurecli-interactive
az group delete --name $resourceGroup
```

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, aprendió a usar métricas y registros de diagnóstico mediante la realización de las siguientes tareas:

> [!div class="checklist"]
> * Mediante la CLI de Azure, crear un centro de IoT, un dispositivo simulado y una cuenta de almacenamiento.  
> * Habilitar los registros de diagnóstico. 
> * Habilitar las métricas.
> * Configurar alertas para esas métricas. 
> * Descargar y ejecutar una aplicación que simule un dispositivo IoT que envía mensajes al centro. 
> * Ejecutar la aplicación hasta que empiecen a activarse las alertas. 
> * Ver los resultados de las métricas y comprobar los registros de diagnóstico. 

En el siguiente tutorial aprender a administrar el estado de un dispositivo IoT. 

> [!div class="nextstepaction"]
> [Configuración de dispositivos desde un servicio back-end](tutorial-device-twins.md)
