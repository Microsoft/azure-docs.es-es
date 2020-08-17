---
title: Configuración del enrutamiento de mensajes para Azure IoT Hub mediante la CLI de Azure
description: Configure el enrutamiento de mensajes para Azure IoT Hub mediante la CLI de Azure. En función de las propiedades del mensaje, se enruta a una cuenta de almacenamiento o a una cola de Service Bus.
author: robinsh
manager: philmea
ms.service: iot-hub
services: iot-hub
ms.topic: tutorial
ms.date: 03/25/2019
ms.author: robinsh
ms.custom: mvc, devx-track-azurecli
ms.openlocfilehash: 239d8f2bcc1422a1098fb8f6cb3fba6706d671f2
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87500204"
---
# <a name="tutorial-use-the-azure-cli-to-configure-iot-hub-message-routing"></a>Tutorial: Uso de la CLI de Azure para configurar el enrutamiento de mensajes de IoT Hub

[!INCLUDE [iot-hub-include-routing-intro](../../includes/iot-hub-include-routing-intro.md)]

[!INCLUDE [iot-hub-include-routing-create-resources](../../includes/iot-hub-include-routing-create-resources.md)]

## <a name="download-the-script-optional"></a>Descarga del script (opcional)

En la segunda parte de este tutorial, descargará y ejecutará una aplicación de Visual Studio para enviar mensajes a IoT Hub. En esa descarga hay una carpeta que contiene la plantilla de Azure Resource Manager y el archivo de parámetros, así como los scripts de la CLI de Azure y PowerShell.

Si desea ver el script terminado, descargue los [ejemplos de C# de Azure IoT](https://github.com/Azure-Samples/azure-iot-samples-csharp/archive/master.zip). Descomprima el archivo master.zip. El script de la CLI de Azure se encuentra en /iot-hub/Tutorials/Routing/SimulatedDevice/resources/ como **iothub_routing_cli.azcli**.

## <a name="use-the-azure-cli-to-create-your-resources"></a>Uso de la CLI de Azure para crear los recursos

Copie y pegue el siguiente script en Cloud Shell y presione Entrar. El script se ejecuta de línea en línea. Con esta primera sección del script creará los recursos de base para este tutorial, incluida la cuenta de almacenamiento, la instancia de IoT Hub, el espacio de nombres de Service Bus y la cola de Service Bus. Conforme avance en el resto del tutorial, copie cada bloque de script y péguelo en Cloud Shell para ejecutarlo.

> [!TIP]
> Una sugerencia de depuración: este script usa el símbolo de continuación (la barra diagonal inversa, `\`) para que el script se lea mejor. Si tiene problemas al ejecutar el script, asegúrese de que la sesión de Cloud Shell esté ejecutando `bash` y de que no hay ningún espacio después de cualquiera de las barras diagonales inversas.
> 

Hay varios nombres de recurso que deben ser únicos globalmente, como el nombre de IoT Hub y el nombre de la cuenta de almacenamiento. Para facilitar esta tarea, se anexan los nombres de los recursos con un valor alfanumérico aleatorio denominado *randomValue*. El valor RandomValue se genera una vez en la parte superior del script y se anexa a los nombres de los recursos según sea necesario en el script. Si no desea que sea aleatorio, puede establecerlo en una cadena vacía o en un valor específico. 

> [!IMPORTANT]
> Las variables que se establecen en el script inicial también las usa el script de enrutamiento, por lo que debe ejecutar todo el script en la misma sesión de Cloud Shell. Si abre una nueva sesión para ejecutar el script para configurar el enrutamiento, a varias de las variables les faltarán valores.
>

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
location=westus
resourceGroup=ContosoResources
iotHubConsumerGroup=ContosoConsumers
containerName=contosoresults
iotDeviceName=Contoso-Test-Device

# Create the resource group to be used
#   for all the resources for this tutorial.
az group create --name $resourceGroup \
    --location $location

# The IoT hub name must be globally unique, 
#   so add a random value to the end.
iotHubName=ContosoTestHub$randomValue 
echo "IoT hub name = " $iotHubName

# Create the IoT hub.
az iot hub create --name $iotHubName \
    --resource-group $resourceGroup \
    --sku S1 --location $location

# Add a consumer group to the IoT hub for the 'events' endpoint.
az iot hub consumer-group create --hub-name $iotHubName \
    --name $iotHubConsumerGroup

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
#    You need this to create the container.
storageAccountKey=$(az storage account keys list \
    --resource-group $resourceGroup \
    --account-name $storageAccountName \
    --query "[0].value" | tr -d '"') 

# See the value of the storage account key.
echo "storage account key = " $storageAccountKey

# Create the container in the storage account. 
az storage container create --name $containerName \
    --account-name $storageAccountName \
    --account-key $storageAccountKey \
    --public-access off

# The Service Bus namespace must be globally unique, 
#   so add a random value to the end.
sbNamespace=ContosoSBNamespace$randomValue
echo "Service Bus namespace = " $sbNamespace

# Create the Service Bus namespace.
az servicebus namespace create --resource-group $resourceGroup \
    --name $sbNamespace \
    --location $location

# The Service Bus queue name must be globally unique, 
#   so add a random value to the end.
sbQueueName=ContosoSBQueue$randomValue
echo "Service Bus queue name = " $sbQueueName

# Create the Service Bus queue to be used as a routing destination.
az servicebus queue create --name $sbQueueName \
    --namespace-name $sbNamespace \
    --resource-group $resourceGroup

# Create the IoT device identity to be used for testing.
az iot hub device-identity create --device-id $iotDeviceName \
    --hub-name $iotHubName

# Retrieve the information about the device identity, then copy the primary key to
#   Notepad. You need this to run the device simulation during the testing phase.
az iot hub device-identity show --device-id $iotDeviceName \
    --hub-name $iotHubName
```

Ahora que los recursos de base están configurados, puede continuar por el enrutamiento de mensajes.

## <a name="set-up-message-routing"></a>Configuración del enrutamiento de mensajes

[!INCLUDE [iot-hub-include-create-routing-description](../../includes/iot-hub-include-create-routing-description.md)]

Para crear un punto de conexión de enrutamiento, use [az iot hub routing-endpoint create](/cli/azure/iot/hub/routing-endpoint?view=azure-cli-latest#az-iot-hub-routing-endpoint-create). Para crear la ruta de mensajes para el punto de conexión, use [az iot hub route create](/cli/azure/iot/hub/route?view=azure-cli-latest#az-iot-hub-route-create).

### <a name="route-to-a-storage-account"></a>Enrutamiento a una cuenta de almacenamiento

[!INCLUDE [iot-hub-include-blob-storage-format](../../includes/iot-hub-include-blob-storage-format.md)]

En primer lugar, configure el punto de conexión para la cuenta de almacenamiento y, después, configure la ruta. 

Estas son las variables utilizadas por el script que se debe establecer en la sesión de Cloud Shell:

**storageConnectionString**: Este valor se recupera de la cuenta de almacenamiento configurada en el script anterior. El enrutamiento de mensajes lo utiliza para acceder a la cuenta de almacenamiento.

  **resourceGroup**: el grupo de recursos aparece dos veces, configure los datos de su grupo de recursos.

**endpoint subscriptionID**: este campo se establece con el identificador de suscripción de Azure del punto de conexión. 

**endpointType**: este campo es el tipo de punto de conexión. Este valor debe establecerse en `azurestoragecontainer`, `eventhub`, `servicebusqueue` o `servicebustopic`. Para los fines de este tutorial, establézcalo en `azurestoragecontainer`.

**iotHubName**: este campo es el nombre del centro que realizará el enrutamiento.

**containerName**: este campo es el nombre del contenedor de la cuenta de almacenamiento en la que se escribirán los datos.

**encoding**: este campo será `avro` o `json`. Esta opción denota el formato de los datos almacenados.

**routeName**: este campo es el nombre de la ruta que se va a configurar. 

**endpointName**: este campo es el nombre que identifica el punto de conexión. 

**enabled**: el valor predeterminado de este campo es `true`, lo cual indica que la ruta de mensajes debe estar habilitada tras la creación.

**condition**: este campo es la consulta utilizada para filtrar los mensajes enviados a este punto de conexión. La condición de consulta para que los mensajes se enruten al almacenamiento es `level="storage"`.

Copie este script, péguelo en la ventana de Cloud Shell y ejecútelo.

```azurecli
##### ROUTING FOR STORAGE ##### 

endpointName="ContosoStorageEndpoint"
endpointType="azurestoragecontainer"
routeName="ContosoStorageRoute"
condition='level="storage"'

# Get the connection string for the storage account.
# Adding the "-o tsv" makes it be returned without the default double quotes around it.
storageConnectionString=$(az storage account show-connection-string \
  --name $storageAccountName --query connectionString -o tsv)
```

El siguiente paso es crear el punto de conexión de enrutamiento para la cuenta de almacenamiento. También se especificará el contenedor donde se almacenarán los resultados. El contenedor se creó antes, cuando se creó la cuenta de almacenamiento.

```azurecli
# Create the routing endpoint for storage.
az iot hub routing-endpoint create \
  --connection-string $storageConnectionString \
  --endpoint-name $endpointName \
  --endpoint-resource-group $resourceGroup \
  --endpoint-subscription-id $subscriptionID \
  --endpoint-type $endpointType \
  --hub-name $iotHubName \
  --container $containerName \
  --resource-group $resourceGroup \
  --encoding avro
```

A continuación se crea la ruta para el punto de conexión de almacenamiento. La ruta de mensajes designa dónde enviar los mensajes que cumplan la especificación de consulta. 

```azurecli
# Create the route for the storage endpoint.
az iot hub route create \
  --name $routeName \
  --hub-name $iotHubName \
  --source devicemessages \
  --resource-group $resourceGroup \
  --endpoint-name $endpointName \
  --enabled \
  --condition $condition
```

### <a name="route-to-a-service-bus-queue"></a>Ruta a una cola de Service Bus

Ahora, configure el enrutamiento de la cola de Service Bus. Para recuperar la cadena de conexión para la cola de Service Bus, debe crear una regla de autorización con los derechos correctos definidos. El siguiente script crea una regla de autorización para la cola de Service Bus llamada `sbauthrule` y establece los derechos en `Listen Manage Send`. Una vez definida esta regla de autorización, podrá usarla para recuperar la cadena de conexión para la cola.

```azurecli
# Create the authorization rule for the Service Bus queue.
az servicebus queue authorization-rule create \
  --name "sbauthrule" \
  --namespace-name $sbNamespace \
  --queue-name $sbQueueName \
  --resource-group $resourceGroup \
  --rights Listen Manage Send \
  --subscription $subscriptionID
```

Ahora, use la regla de autorización para recuperar la cadena de conexión a la cola de Service Bus.

```azurecli
# Get the Service Bus queue connection string.
# The "-o tsv" ensures it is returned without the default double-quotes.
sbqConnectionString=$(az servicebus queue authorization-rule keys list \
  --name "sbauthrule" \
  --namespace-name $sbNamespace \
  --queue-name $sbQueueName \
  --resource-group $resourceGroup \
  --subscription $subscriptionID \
  --query primaryConnectionString -o tsv)

# Show the Service Bus queue connection string.
echo "service bus queue connection string = " $sbqConnectionString
```

Ahora, configure el punto de conexión de enrutamiento y la ruta de mensajes para la cola de Service Bus. Estas son las variables utilizadas por el script que se debe establecer en la sesión de Cloud Shell:

**endpointName**: este campo es el nombre que identifica el punto de conexión. 

**endpointType**: este campo es el tipo de punto de conexión. Este valor debe establecerse en `azurestoragecontainer`, `eventhub`, `servicebusqueue` o `servicebustopic`. Para los fines de este tutorial, establézcalo en `servicebusqueue`.

**routeName**: este campo es el nombre de la ruta que se va a configurar. 

**condition**: este campo es la consulta utilizada para filtrar los mensajes enviados a este punto de conexión. La condición de consulta para que los mensajes se enruten a la cola de Service Bus es `level="critical"`.

Esta es la CLI de Azure para el punto de conexión de enrutamiento y la ruta de mensajes para la cola de Service Bus.

```azurecli
endpointName="ContosoSBQueueEndpoint"
endpointType="ServiceBusQueue"
routeName="ContosoSBQueueRoute"
condition='level="critical"'

# Set up the routing endpoint for the Service Bus queue.
# This uses the Service Bus queue connection string.
az iot hub routing-endpoint create \
  --connection-string $sbqConnectionString \
  --endpoint-name $endpointName \
  --endpoint-resource-group $resourceGroup \
  --endpoint-subscription-id $subscriptionID \
  --endpoint-type $endpointType \
  --hub-name $iotHubName \
  --resource-group $resourceGroup 

# Set up the message route for the Service Bus queue endpoint.
az iot hub route create --name $routeName \
  --hub-name $iotHubName \
  --source-type devicemessages \
  --resource-group $resourceGroup \
  --endpoint-name $endpointName \
  --enabled \
  --condition $condition
  ```

### <a name="view-message-routing-in-the-portal"></a>Visualización del enrutamiento de mensajes en el portal

[!INCLUDE [iot-hub-include-view-routing-in-portal](../../includes/iot-hub-include-view-routing-in-portal.md)]

## <a name="next-steps"></a>Pasos siguientes

Ahora que tiene las rutas de mensajes y los recursos configurados, pase al siguiente tutorial para aprender a enviar mensajes a IoT Hub y verlos enrutados a los distintos destinos. 

> [!div class="nextstepaction"]
> [Parte 2: Visualización de los resultados del enrutamiento de mensajes](tutorial-routing-view-message-routing-results.md)
