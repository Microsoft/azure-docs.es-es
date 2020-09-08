---
title: 'Tutorial: Uso del enriquecimiento de mensajes de Azure IoT Hub'
description: Tutorial que muestra cómo usar el enriquecimiento de mensajes para los mensajes de IoT Hub de Azure.
author: robinsh
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 12/20/2019
ms.author: robinsh
ms.custom: mqtt, devx-track-azurecli, devx-track-csharp
ms.openlocfilehash: 1e04ed8fe9087cb06b1533887da3c0fed61259ef
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89019732"
---
# <a name="tutorial-use-azure-iot-hub-message-enrichments"></a>Tutorial: Uso del enriquecimiento de mensajes de Azure IoT Hub

El *enriquecimiento de mensajes* describe la capacidad de Azure IoT Hub para *marcar* los mensajes con información adicional antes de que los mensajes se envíen al punto de conexión designado. Una razón para utilizar el enriquecimiento de mensajes es incluir datos que se pueden usar para simplificar el procesamiento descendente. Por ejemplo, el enriquecimiento de mensajes de telemetría de dispositivos con una etiqueta de dispositivo gemelo puede reducir la carga de los clientes para realizar llamadas API de dispositivos gemelos para obtener esta información. Para más información, consulte la [introducción al enriquecimiento de mensajes](iot-hub-message-enrichments-overview.md).

En este tutorial, verá dos maneras de crear y configurar los recursos necesarios para probar el enriquecimiento de mensajes para un centro de IoT. Los recursos incluyen una cuenta de almacenamiento con dos contenedores de almacenamiento. Un contenedor contiene los mensajes enriquecidos, y otro contenedor contiene los mensajes originales. También se incluye un centro de IoT para recibir los mensajes y enrutarlos al contenedor de almacenamiento adecuado en función de si están enriquecidos o no.

* El primer método consiste en usar la CLI de Azure para crear los recursos y configurar el enrutamiento de mensajes. A continuación, defina el enriquecimiento de forma manual mediante [Azure Portal](https://portal.azure.com).

* El segundo método consiste en usar una plantilla de Azure Resource Manager para crear los recursos *y* las configuraciones para el enrutamiento y las características enriquecidas de los mensajes.

Una vez finalizadas las configuraciones para el enrutamiento de mensajes y el enriquecimiento de mensajes, use una aplicación para enviar mensajes al centro de IoT. A continuación, el centro los enruta a ambos contenedores de almacenamiento. Solo están enriquecidos los mensajes enviados al punto de conexión para el contenedor de almacenamiento **enriquecido**.

Estas son las tareas que tendrá que llevar a cabo para completar este tutorial:

**Uso del enriquecimiento de mensajes de IoT Hub**
> [!div class="checklist"]
> * Primer método: Cree recursos y configure el enrutamiento de mensajes mediante la CLI de Azure. Configure el enriquecimiento de mensajes de forma manual mediante [Azure Portal](https://portal.azure.com).
> * Segundo método: Cree recursos y configure el enrutamiento de mensajes y el enriquecimiento de mensajes mediante una plantilla de Resource Manager. 
> * Ejecute una aplicación que simule un dispositivo IoT que envía mensajes al centro.
> * Vea los resultados y verifique que el enriquecimiento de mensajes funcione según lo previsto.

## <a name="prerequisites"></a>Prerrequisitos

* Debe tener una suscripción de Azure. Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.
* Instale [Visual Studio](https://www.visualstudio.com/).

* Asegúrese de que el puerto 8883 está abierto en el firewall. En el dispositivos de ejemplo de este tutorial se usa el protocolo MQTT, que se comunica mediante el puerto 8883. Este puerto puede estar bloqueado en algunos entornos de red corporativos y educativos. Para más información y saber cómo solucionar este problema, consulte el artículo sobre la [conexión a IoT Hub (MQTT)](iot-hub-mqtt-support.md#connecting-to-iot-hub).


[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

## <a name="retrieve-the-iot-c-samples-repository"></a>Recuperación del repositorio de muestras en C# de IoT

Descargue las [muestras en C# de IoT](https://github.com/Azure-Samples/azure-iot-samples-csharp/archive/master.zip) de GitHub y descomprímalas. Este repositorio tiene varias aplicaciones, scripts y plantillas de Resource Manager. Los que se van a usar para este tutorial son los siguientes:

* Para el método manual, hay un script de la CLI que se usa para crear los recursos. El script se encuentra en /azure-iot-samples-csharp/iot-hub/Tutorials/Routing/SimulatedDevice/resources/iothub_msgenrichment_cli.azcli. Este script crea los recursos y configura el enrutamiento de mensajes. Después de ejecutar este script, cree el enriquecimiento de mensajes de forma manual mediante [Azure Portal](https://portal.azure.com).
* Para el método automatizado, hay una plantilla de Azure Resource Manager. La plantilla se encuentra en /azure-iot-samples-csharp/iot-hub/Tutorials/Routing/SimulatedDevice/resources/template_msgenrichments.json. Esta plantilla crea los recursos, configura el enrutamiento de mensajes y, a continuación, configura el enriquecimiento de mensajes.
* La tercera aplicación que usa es la aplicación Simulación de dispositivo, que se usa para enviar mensajes a IoT Hub y probar las características enriquecidas del mensaje.

## <a name="manually-set-up-and-configure-by-using-the-azure-cli"></a>Instalación y configuración manuales mediante la CLI de Azure

Además de crear los recursos necesarios, el script de la CLI de Azure también configura las dos rutas a los puntos de conexión que son contenedores de almacenamiento independientes. Para obtener más información sobre la configuración del enrutamiento de mensajes, consulte el [tutorial de enrutamiento](tutorial-routing.md). Una vez configurados los recursos, use [Azure Portal](https://portal.azure.com) para configurar el enriquecimiento de mensajes para cada punto de conexión. Luego, continúe con el paso de prueba.

> [!NOTE]
> Todos los mensajes se enrutan a ambos puntos de conexión, pero solo se enriquecerán los mensajes que pasen al punto de conexión con el enriquecimiento de mensajes configurado.
>

Puede usar el siguiente script o abrir el script en la carpeta /resources del repositorio descargado. El script sigue los pasos a continuación:

* Cree un Centro de IoT.
* Cree una cuenta de almacenamiento.
* Creación de dos contenedores en la cuenta de almacenamiento. Un contenedor es para los mensajes enriquecidos, y otro para los mensajes que no están enriquecidos.
* Configuración del enrutamiento para las dos cuentas de almacenamiento diferentes:
    * Creación de un punto de conexión para cada contenedor de la cuenta de almacenamiento.
    * Creación de una ruta a cada uno de los puntos de conexión del contenedor de la cuenta de almacenamiento.

Hay varios nombres de recurso que deben ser únicos globalmente, como el nombre del centro de IoT y el nombre de la cuenta de almacenamiento. Para facilitar la ejecución del script, se anexan los nombres de los recursos con un valor alfanumérico aleatorio denominado *randomValue*. El valor aleatorio se genera una vez en la parte superior del script. Se anexa a los nombres de recursos según sea necesario a lo largo del script. Si no desea que el valor sea aleatorio, puede establecerlo en una cadena vacía o en un valor específico.

Si todavía no lo ha hecho, abra una [ventana de Azure Cloud Shell](https://shell.azure.com) y asegúrese de que esté establecida en Bash. Abra el script en el repositorio sin comprimir, use CTRL+A para seleccionar todo y, a continuación, CTRL+C para copiarlo. También puede copiar el siguiente script de la CLI o abrirlo directamente en Cloud Shell. Para pegar el script en la ventana de Cloud Shell, haga clic con el botón derecho en la línea de comandos y seleccione **Pegar**. Las instrucciones del script se ejecutan de una en una. Cuando el script deje de ejecutarse, seleccione **Intro** para asegurarse de que ejecuta el último comando. El siguiente bloque de código muestra el script que se utiliza, con los comentarios que explican lo que hace.

A continuación se muestran los recursos creados por el script. *Enriched* significa que el recurso es para los mensajes con enriquecimiento. *Original* significa que el recurso es para los mensajes no enriquecidos.

| Nombre | Value |
|-----|-----|
| resourceGroup | ContosoResourcesMsgEn |
| container name | original  |
| container name | enriched  |
| IoT device name | Contoso-Test-Device |
| Nombre de IoT Hub | ContosoTestHubMsgEn |
| storage Account Name | contosostorage |
| endpoint Name 1 | ContosoStorageEndpointOriginal |
| endpoint Name 2 | ContosoStorageEndpointEnriched|
| route Name 1 | ContosoStorageRouteOriginal |
| route Name 2 | ContosoStorageRouteEnriched |

```azurecli-interactive
# This command retrieves the subscription id of the current Azure account.
# This field is used when setting up the routing queries.
subscriptionID=$(az account show --query id -o tsv)

# Concatenate this number onto the resources that have to be globally unique.
# You can set this to "" or to a specific value if you don't want it to be random.
# This retrieves a random value.
randomValue=$RANDOM

# This command installs the IOT Extension for Azure CLI.
# You only need to install this the first time.
# You need it to create the device identity.
az extension add --name azure-iot

# Set the values for the resource names that
#   don't have to be globally unique.
location=westus2
resourceGroup=ContosoResourcesMsgEn
containerName1=original
containerName2=enriched
iotDeviceName=Contoso-Test-Device

# Create the resource group to be used
#   for all the resources for this tutorial.
az group create --name $resourceGroup \
    --location $location

# The IoT hub name must be globally unique,
#   so add a random value to the end.
iotHubName=ContosoTestHubMsgEn$randomValue
echo "IoT hub name = " $iotHubName

# Create the IoT hub.
az iot hub create --name $iotHubName \
    --resource-group $resourceGroup \
    --sku S1 --location $location

# You need a storage account that will have two containers
#   -- one for the original messages and
#   one for the enriched messages.
# The storage account name must be globally unique,
#   so add a random value to the end.
storageAccountName=contosostorage$randomValue
echo "Storage account name = " $storageAccountName

# Create the storage account to be used as a routing destination.
az storage account create --name $storageAccountName \
    --resource-group $resourceGroup \
    --location $location \
    --sku Standard_LRS

# Get the primary storage account key.
#    You need this to create the containers.
storageAccountKey=$(az storage account keys list \
    --resource-group $resourceGroup \
    --account-name $storageAccountName \
    --query "[0].value" | tr -d '"')

# See the value of the storage account key.
echo "storage account key = " $storageAccountKey

# Create the containers in the storage account.
az storage container create --name $containerName1 \
    --account-name $storageAccountName \
    --account-key $storageAccountKey \
    --public-access off

az storage container create --name $containerName2 \
    --account-name $storageAccountName \
    --account-key $storageAccountKey \
    --public-access off

# Create the IoT device identity to be used for testing.
az iot hub device-identity create --device-id $iotDeviceName \
    --hub-name $iotHubName

# Retrieve the information about the device identity, then copy the primary key to
#   Notepad. You need this to run the device simulation during the testing phase.
# If you are using Cloud Shell, you can scroll the window back up to retrieve this value.
az iot hub device-identity show --device-id $iotDeviceName \
    --hub-name $iotHubName

##### ROUTING FOR STORAGE #####

# You're going to have two routes and two endpoints.
# One route points to the first container ("original") in the storage account
#   and includes the original messages.
# The other points to the second container ("enriched") in the same storage account
#   and includes the enriched versions of the messages.

endpointType="azurestoragecontainer"
endpointName1="ContosoStorageEndpointOriginal"
endpointName2="ContosoStorageEndpointEnriched"
routeName1="ContosoStorageRouteOriginal"
routeName2="ContosoStorageRouteEnriched"

# for both endpoints, retrieve the messages going to storage
condition='level="storage"'

# Get the connection string for the storage account.
# Adding the "-o tsv" makes it be returned without the default double quotes around it.
storageConnectionString=$(az storage account show-connection-string \
  --name $storageAccountName --query connectionString -o tsv)

# Create the routing endpoints and routes.
# Set the encoding format to either avro or json.

# This is the endpoint for the first container, for endpoint messages that are not enriched.
az iot hub routing-endpoint create \
  --connection-string $storageConnectionString \
  --endpoint-name $endpointName1 \
  --endpoint-resource-group $resourceGroup \
  --endpoint-subscription-id $subscriptionID \
  --endpoint-type $endpointType \
  --hub-name $iotHubName \
  --container $containerName1 \
  --resource-group $resourceGroup \
  --encoding json

# This is the endpoint for the second container, for endpoint messages that are enriched.
az iot hub routing-endpoint create \
  --connection-string $storageConnectionString \
  --endpoint-name $endpointName2 \
  --endpoint-resource-group $resourceGroup \
  --endpoint-subscription-id $subscriptionID \
  --endpoint-type $endpointType \
  --hub-name $iotHubName \
  --container $containerName2 \
  --resource-group $resourceGroup \
  --encoding json

# This is the route for messages that are not enriched.
# Create the route for the first storage endpoint.
az iot hub route create \
  --name $routeName1 \
  --hub-name $iotHubName \
  --source devicemessages \
  --resource-group $resourceGroup \
  --endpoint-name $endpointName1 \
  --enabled \
  --condition $condition

# This is the route for messages that are enriched.
# Create the route for the second storage endpoint.
az iot hub route create \
  --name $routeName2 \
  --hub-name $iotHubName \
  --source devicemessages \
  --resource-group $resourceGroup \
  --endpoint-name $endpointName2 \
  --enabled \
  --condition $condition
```

En este momento, se establecen todos los recursos y se configura el enrutamiento de mensajes. Puede ver la configuración de enrutamiento de mensajes en el portal y configurar el enriquecimiento de mensajes para los mensajes que pasan al contenedor de almacenamiento **enriquecido**.

### <a name="manually-configure-the-message-enrichments-by-using-the-azure-portal"></a>Configuración manual del enriquecimiento de mensajes mediante Azure Portal

1. Para ir a su centro de IoT, seleccione **Grupos de recursos**. A continuación, seleccione el grupo de recursos configurado para este tutorial (**ContosoResourcesMsgEn**). Busque el centro de IoT Hub en la lista y selecciónelo. Seleccione **Enrutamiento de mensajes** para el centro de IoT.

   ![Selección del enrutamiento de mensajes](./media/tutorial-message-enrichments/select-iot-hub.png)

   El panel de enrutamiento de mensajes tiene tres pestañas: **Rutas**, **Puntos de conexión personalizados** y **Enriquecer mensajes**. Examine las dos primeras pestañas para ver la configuración establecida por el script. Utilice la tercera pestaña para agregar el enriquecimiento de mensajes. A continuación, vamos a enriquecer mensajes que pasan al punto de conexión para el contenedor de almacenamiento denominado **enriched**. Complete el nombre y el valor y, a continuación, seleccione el punto de conexión **ContosoStorageEndpointEnriched** de la lista desplegable. Aquí se muestra un ejemplo de cómo configurar un enriquecimiento que agrega el nombre del centro de IoT al mensaje:

   ![Agregar primer enriquecimiento](./media/tutorial-message-enrichments/add-message-enrichments.png)

2. Agregue estos valores a la lista para el punto de conexión ContosoStorageEndpointEnriched.

   | Clave | Value | Punto de conexión (lista desplegable) |
   | ---- | ----- | -------------------------|
   | myIotHub | $iothubname | AzureStorageContainers > ContosoStorageEndpointEnriched |
   | DeviceLocation | $twin.tags.location | AzureStorageContainers > ContosoStorageEndpointEnriched |
   |customerID | 6ce345b8-1e4a-411e-9398-d34587459a3a | AzureStorageContainers > ContosoStorageEndpointEnriched |

   > [!NOTE]
   > Si el dispositivo no tiene un gemelo, el valor que especifique aquí se marcará como cadena para el valor en el enriquecimiento de mensajes. Para ver la información del dispositivo gemelo, vaya al centro en el portal y seleccione **Dispositivos IoT**. Seleccione el dispositivo y, a continuación, **Dispositivo gemelo** en la parte superior de la página.
   >
   > Puede editar la información sobre el dispositivo gemelo para agregar etiquetas (como la ubicación) y establecerlo en un valor específico. Para más información, consulte [Información y uso de dispositivos gemelos en IoT Hub](iot-hub-devguide-device-twins.md).

3. Cuando haya terminado, el panel debe ser similar al siguiente:

   ![Tabla con todos los enriquecimientos agregados](./media/tutorial-message-enrichments/all-message-enrichments.png)

4. Seleccione **Aplicar** para guardar los cambios. Vaya a la sección [Prueba del enriquecimiento de mensajes](#test-message-enrichments).

## <a name="create-and-configure-by-using-a-resource-manager-template"></a>Creación y configuración mediante una plantilla de Resource Manager
Puede usar una plantilla de Resource Manager para crear y configurar los recursos, el enrutamiento de mensajes y el enriquecimiento de mensajes

1. Inicie sesión en Azure Portal. Seleccione **+ Crear un recurso** para que aparezca un cuadro de búsqueda. Escriba *implementación de plantillas* y búsquela. En el panel de resultados, seleccione **Template deployment (deploy using custom templates)** (Implementación de plantillas [implementar mediante plantillas personalizadas]).

   ![Implementación de plantillas en Azure Portal](./media/tutorial-message-enrichments/template-select-deployment.png)

1. Seleccione **Crear** en el panel **Implementación de plantillas**.

1. En el panel **Implementación personalizada**, seleccione **Cree su propia plantilla en el editor**.

1. En el panel **Editar plantilla**, seleccione **Cargar archivo**. Aparece el Explorador de Windows. Busque el archivo **template_messageenrichments.json** en el archivo de repositorio descomprimido en **/iot-hub/Tutorials/Routing/SimulatedDevice/resources**. 

   ![Seleccionar plantilla desde la máquina local](./media/tutorial-message-enrichments/template-select.png)

1. Seleccione **Abrir** para cargar el archivo de plantilla desde la máquina local. Se carga y aparece en el panel de edición.

   Esta plantilla está configurada para usar un nombre de centro de IoT único global y un nombre de cuenta de almacenamiento agregando un valor aleatorio al final de los nombres predeterminados, por lo que puede usar la plantilla sin hacerle ningún cambio.

   Estos son los recursos que se crean al cargar la plantilla. **Enriched** significa que el recurso es para los mensajes con enriquecimiento. **Original** significa que el recurso es para los mensajes no enriquecidos. Estos son los mismos valores que se usan en el script de la CLI de Azure.

   | Nombre | Value |
   |-----|-----|
   | resourceGroup | ContosoResourcesMsgEn |
   | container name | original  |
   | container name | enriched  |
   | IoT device name | Contoso-Test-Device |
   | Nombre de IoT Hub | ContosoTestHubMsgEn |
   | storage Account Name | contosostorage |
   | endpoint Name 1 | ContosoStorageEndpointOriginal |
   | endpoint Name 2 | ContosoStorageEndpointEnriched|
   | route Name 1 | ContosoStorageRouteOriginal |
   | route Name 2 | ContosoStorageRouteEnriched |

1. Seleccione **Guardar**. Aparece el panel **Implementación personalizada** muestra todos los parámetros que usa la plantilla. El único campo que debe establecer es **Grupo de recursos**. Cree uno o seleccione uno de la lista desplegable.

   Esta es la mitad superior del panel **Implementación personalizada**. Puede ver dónde rellena el grupo de recursos.

   ![Mitad superior del panel Implementación personalizada](./media/tutorial-message-enrichments/template-deployment-top.png)

1. Esta es la mitad inferior del panel **Implementación personalizada**. Puede ver el resto de los parámetros, así como los términos y condiciones. 

   ![Mitad inferior del panel Implementación personalizada](./media/tutorial-message-enrichments/template-deployment-bottom.png)

1. Seleccione la casilla para aceptar los términos y condiciones. A continuación, seleccione **Compra** para continuar con la implementación de la plantilla.

1. Espere a que la plantilla se implemente por completo. Seleccione el icono de campana de la parte superior de la pantalla para consultar el progreso. Cuando haya finalizado, continúe con la sección [Prueba del enriquecimiento de mensajes](#test-message-enrichments).

## <a name="test-message-enrichments"></a>Prueba del enriquecimiento de mensajes

Para ver el enriquecimiento de mensajes, seleccione **Grupos de recursos**. A continuación, seleccione el grupo de recursos que utiliza para este tutorial. Seleccione el centro de IoT de la lista de recursos y vaya a **Mensajería**. Aparecen la configuración del enrutamiento de mensajes y el enriquecimiento configurado.

Ahora que el enriquecimiento de mensajes está configurado para el punto de conexión, ejecute la aplicación de dispositivo simulado para enviar mensajes al centro de IoT. El centro se configuró con opciones que permiten cumplir las tareas siguientes:

* Los mensajes enrutados al punto de conexión de almacenamiento ContosoStorageEndpointOriginal no se enriquecerán y almacenarán en el contenedor de almacenamiento `original`.

* Los mensajes enrutados al punto de conexión de almacenamiento ContosoStorageEndpointEnriched se enriquecerán y almacenarán en el contenedor de almacenamiento `enriched`.

La aplicación de dispositivo simulado es una de las aplicaciones en la descarga sin comprimir. La aplicación envía mensajes para cada uno de los métodos de enrutamiento de mensajes diferentes en el [Tutorial de enrutamiento](tutorial-routing.md), lo que incluye Azure Storage.

Haga doble clic en el archivo de la solución **IoT_SimulatedDevice.sln** para abrir el código en Visual Studio y abra **Program.cs**. Sustituya el nombre del centro de IoT por el marcador `{your hub name}`. El formato del nombre de host de IoT Hub es **{su nombre del centro}.azure-devices.net**. Para este tutorial, el nombre de host del centro es ContosoTestHubMsgEn.azure-devices.net. A continuación, sustituya la clave de dispositivo que guardó anteriormente cuando ejecutó el script para crear los recursos del marcador `{your device key}`.

Si no dispone de la clave de dispositivo, puede recuperarla desde el portal. Después de iniciar sesión, vaya a **Grupos de recursos**, seleccione el grupo de recursos y, a continuación, seleccione su centro de IoT. Busque su dispositivo de prueba en **Dispositivos IoT** y selecciónelo. Seleccione el icono de copia junto a la **clave principal** para copiarla en el portapapeles.

   ```csharp
        private readonly static string s_myDeviceId = "Contoso-Test-Device";
        private readonly static string s_iotHubUri = "ContosoTestHubMsgEn.azure-devices.net";
        // This is the primary key for the device. This is in the portal.
        // Find your IoT hub in the portal > IoT devices > select your device > copy the key.
        private readonly static string s_deviceKey = "{your device key}";
   ```

### <a name="run-and-test"></a>Ejecución y prueba

Ejecute la aplicación de consola durante unos minutos. Los mensajes que se envían aparecen en la pantalla de la consola de la aplicación.

La aplicación envía un nuevo mensaje de dispositivo a nube a IoT Hub cada segundo. El mensaje contiene un objeto serializado mediante JSON serializado con el identificador de dispositivo, la temperatura, la humedad y el nivel de mensaje, cuyo valor predeterminado es `normal`. Asigna de forma aleatoria un nivel `critical` o `storage`, lo que hace que el mensaje se enrute a la cuenta de almacenamiento o al punto de conexión predeterminado. Se enriquecerán los mensajes enviados al contenedor **enriched** en la cuenta de almacenamiento.

Una vez que se envíen varios mensajes de almacenamiento, vea los datos.

1. Seleccione **Grupos de recursos**. Busque su grupo de recursos, **ContosoResourcesMsgEn**, y selecciónelo.

2. Seleccione su cuenta de almacenamiento, que es **contosostorage**. A continuación, seleccione **Explorador de Storage (versión preliminar)** en el panel izquierdo.

   ![Selección del Explorador de Storage](./media/tutorial-message-enrichments/select-storage-explorer.png)

   Seleccione **CONTENEDORES DE BLOB** para ver los dos contenedores que pueden utilizarse.

   ![Ver los contenedores en la cuenta de Storage](./media/tutorial-message-enrichments/show-blob-containers.png)

Los mensajes del contenedor denominados **enriched** cuentan con enriquecimiento de mensajes incluido en los mensajes. Los mensajes en el contenedor denominado **original** contienen los mensajes sin procesar y sin enriquecimiento. Explore en profundidad uno de los contenedores hasta que llegue al final, luego abra el archivo de mensaje más reciente. A continuación, haga lo mismo con el otro contenedor para verificar que no se hayan agregado características enriquecidas a los mensajes de ese contenedor.

Cuando examine los mensajes enriquecidos, debería ver "my IoT Hub" con el nombre del centro, así como la ubicación y el identificador de cliente, de esta manera:

```json
{"EnqueuedTimeUtc":"2019-05-10T06:06:32.7220000Z","Properties":{"level":"storage","my IoT Hub":"contosotesthubmsgen3276","devicelocation":"$twin.tags.location","customerID":"6ce345b8-1e4a-411e-9398-d34587459a3a"},"SystemProperties":{"connectionDeviceId":"Contoso-Test-Device","connectionAuthMethod":"{\"scope\":\"device\",\"type\":\"sas\",\"issuer\":\"iothub\",\"acceptingIpFilterRule\":null}","connectionDeviceGenerationId":"636930642531278483","enqueuedTime":"2019-05-10T06:06:32.7220000Z"},"Body":"eyJkZXZpY2VJZCI6IkNvbnRvc28tVGVzdC1EZXZpY2UiLCJ0ZW1wZXJhdHVyZSI6MjkuMjMyMDE2ODQ4MDQyNjE1LCJodW1pZGl0eSI6NjQuMzA1MzQ5NjkyODQ0NDg3LCJwb2ludEluZm8iOiJUaGlzIGlzIGEgc3RvcmFnZSBtZXNzYWdlLiJ9"}
```

Este es un mensaje sin enriquecimiento. Tenga en cuenta que "my IoT Hub", "devicelocation" y "customerID" no se muestran aquí porque el enriquecimiento agrega estos campos. Este punto de conexión no tiene características enriquecidas.

```json
{"EnqueuedTimeUtc":"2019-05-10T06:06:32.7220000Z","Properties":{"level":"storage"},"SystemProperties":{"connectionDeviceId":"Contoso-Test-Device","connectionAuthMethod":"{\"scope\":\"device\",\"type\":\"sas\",\"issuer\":\"iothub\",\"acceptingIpFilterRule\":null}","connectionDeviceGenerationId":"636930642531278483","enqueuedTime":"2019-05-10T06:06:32.7220000Z"},"Body":"eyJkZXZpY2VJZCI6IkNvbnRvc28tVGVzdC1EZXZpY2UiLCJ0ZW1wZXJhdHVyZSI6MjkuMjMyMDE2ODQ4MDQyNjE1LCJodW1pZGl0eSI6NjQuMzA1MzQ5NjkyODQ0NDg3LCJwb2ludEluZm8iOiJUaGlzIGlzIGEgc3RvcmFnZSBtZXNzYWdlLiJ9"}
```

## <a name="clean-up-resources"></a>Limpieza de recursos

Para quitar todos los recursos que creó en este tutorial, elimine el grupo de recursos. Esta acción elimina también todos los recursos del grupo. En este caso, se quita el centro de IoT, la cuenta de almacenamiento y el propio grupo de recursos.

### <a name="use-the-azure-cli-to-clean-up-resources"></a>Uso de la CLI de Azure para quitar recursos

Para quitar el grupo de recursos, use el comando [az group delete](https://docs.microsoft.com/cli/azure/group?view=azure-cli-latest#az-group-delete). Recuerde que `$resourceGroup` se estableció en **ContosoResourcesMsgEn** al principio de este tutorial.

```azurecli-interactive
az group delete --name $resourceGroup
```

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha configurado y probado la adición del enriquecimiento de mensajes para mensajes de IoT Hub mediante los pasos siguientes:

**Uso del enriquecimiento de mensajes de IoT Hub**
> [!div class="checklist"]
> * Primer método: Cree recursos y configure el enrutamiento de mensajes mediante la CLI de Azure. Configure el enriquecimiento de mensajes de forma manual mediante [Azure Portal](https://portal.azure.com).
> * Segundo método: Cree recursos y configure el enrutamiento de mensajes y el enriquecimiento de mensajes mediante una plantilla de Azure Resource Manager.
> * Ejecute una aplicación que simule un dispositivo IoT que envía mensajes al centro.
> * Vea los resultados y verifique que el enriquecimiento de mensajes funcione según lo previsto.

Para obtener más información sobre el enriquecimiento de mensajes, consulte la [Introducción al enriquecimiento de mensajes](iot-hub-message-enrichments-overview.md).

Para obtener más información sobre el enrutamiento de mensajes, consulte estos artículos:

* [Uso del enrutamiento de mensajes de IoT Hub para enviar mensajes del dispositivo a la nube a distintos puntos de conexión](iot-hub-devguide-messages-d2c.md)
* [Tutorial: Enrutamiento de IoT Hub](tutorial-routing.md)
