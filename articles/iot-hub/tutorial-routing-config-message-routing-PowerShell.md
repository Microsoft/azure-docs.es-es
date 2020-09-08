---
title: Configuración del enrutamiento de mensajes para Azure IoT Hub con Azure PowerShell
description: Configure el enrutamiento de mensajes para Azure IoT Hub mediante Azure PowerShell. En función de las propiedades del mensaje, se enruta a una cuenta de almacenamiento o a una cola de Service Bus.
author: robinsh
manager: philmea
ms.service: iot-hub
services: iot-hub
ms.topic: tutorial
ms.date: 03/25/2019
ms.author: robinsh
ms.custom: mvc, devx-track-azurepowershell
ms.openlocfilehash: 92c30287e836ccadda915dab70a4d40e9c4e851a
ms.sourcegitcommit: 656c0c38cf550327a9ee10cc936029378bc7b5a2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89073410"
---
# <a name="tutorial-use-azure-powershell-to-configure-iot-hub-message-routing"></a>Tutorial: Uso de Azure PowerShell para configurar el enrutamiento de mensajes de IoT Hub

[!INCLUDE [iot-hub-include-routing-intro](../../includes/iot-hub-include-routing-intro.md)]

[!INCLUDE [iot-hub-include-routing-create-resources](../../includes/iot-hub-include-routing-create-resources.md)]

## <a name="download-the-script-optional"></a>Descarga del script (opcional)

En la segunda parte de este tutorial, descargará y ejecutará una aplicación de Visual Studio para enviar mensajes a IoT Hub. En esa descarga hay una carpeta que contiene la plantilla de Azure Resource Manager y el archivo de parámetros, así como los scripts de la CLI de Azure y PowerShell. 

Si desea ver el script terminado, descargue los [ejemplos de C# de Azure IoT](https://github.com/Azure-Samples/azure-iot-samples-csharp/archive/master.zip). Descomprima el archivo master.zip. El script de la CLI de Azure se encuentra en /iot-hub/Tutorials/Routing/SimulatedDevice/resources/ como **iothub_routing_psh.ps1**.

## <a name="create-your-resources"></a>Creación de los recursos

Empiece por crear los recursos con PowerShell.

### <a name="use-powershell-to-create-your-base-resources"></a>Uso de PowerShell para crear los recursos de base

Copie y pegue el siguiente script en Cloud Shell y presione Entrar. El script se ejecuta de línea en línea. Con esta primera sección del script creará los recursos de base para este tutorial, incluida la cuenta de almacenamiento, la instancia de IoT Hub, el espacio de nombres de Service Bus y la cola de Service Bus. Conforme avance en el tutorial, copie cada bloque de script y péguelo en Cloud Shell para ejecutarlo.

Hay varios nombres de recurso que deben ser únicos globalmente, como el nombre de IoT Hub y el nombre de la cuenta de almacenamiento. Para facilitar esta tarea, se anexan los nombres de los recursos con un valor alfanumérico aleatorio denominado *randomValue*. El valor RandomValue se genera una vez en la parte superior del script y se anexa a los nombres de los recursos según sea necesario en el script. Si no desea que sea aleatorio, puede establecerlo en una cadena vacía o en un valor específico. 

> [!IMPORTANT]
> Las variables que se establecen en el script inicial también las usa el script de enrutamiento, por lo que debe ejecutar todo el script en la misma sesión de Cloud Shell. Si abre una nueva sesión para ejecutar el script para configurar el enrutamiento, a varias de las variables les faltarán valores. 
>

```azurepowershell-interactive
# This command retrieves the subscription id of the current Azure account.
# This field is used when setting up the routing queries.
$subscriptionID = (Get-AzContext).Subscription.Id

# Concatenate this number onto the resources that have to be globally unique.
# You can set this to "" or to a specific value if you don't want it to be random.
# This retrieves the first 6 digits of a random value.
$randomValue = "$(Get-Random)".Substring(0,6)

# Set the values for the resource names that don't have to be globally unique.
$location = "West US"
$resourceGroup = "ContosoResources"
$iotHubConsumerGroup = "ContosoConsumers"
$containerName = "contosoresults"

# Create the resource group to be used 
#   for all resources for this tutorial.
New-AzResourceGroup -Name $resourceGroup -Location $location

# The IoT hub name must be globally unique, 
#   so add a random value to the end.
$iotHubName = "ContosoTestHub" + $randomValue
Write-Host "IoT hub name is " $iotHubName

# Create the IoT hub.
New-AzIotHub -ResourceGroupName $resourceGroup `
    -Name $iotHubName `
    -SkuName "S1" `
    -Location $location `
    -Units 1 

# Add a consumer group to the IoT hub for the 'events' endpoint.
Add-AzIotHubEventHubConsumerGroup -ResourceGroupName $resourceGroup `
  -Name $iotHubName `
  -EventHubConsumerGroupName $iotHubConsumerGroup `
  -EventHubEndpointName "events"

# The storage account name must be globally unique, so add a random value to the end.
$storageAccountName = "contosostorage" + $randomValue
Write-Host "storage account name is " $storageAccountName

# Create the storage account to be used as a routing destination.
# Save the context for the storage account 
#   to be used when creating a container.
$storageAccount = New-AzStorageAccount -ResourceGroupName $resourceGroup `
    -Name $storageAccountName `
    -Location $location `
    -SkuName Standard_LRS `
    -Kind Storage
# Retrieve the connection string from the context. 
$storageConnectionString = $storageAccount.Context.ConnectionString
Write-Host "storage connection string = " $storageConnectionString 

# Create the container in the storage account.
New-AzStorageContainer -Name $containerName `
    -Context $storageAccount.Context

# The Service Bus namespace must be globally unique,
#   so add a random value to the end.
$serviceBusNamespace = "ContosoSBNamespace" + $randomValue
Write-Host "Service Bus namespace is " $serviceBusNamespace

# Create the Service Bus namespace.
New-AzServiceBusNamespace -ResourceGroupName $resourceGroup `
    -Location $location `
    -Name $serviceBusNamespace 

# The Service Bus queue name must be globally unique,
#  so add a random value to the end.
$serviceBusQueueName  = "ContosoSBQueue" + $randomValue
Write-Host "Service Bus queue name is " $serviceBusQueueName 

# Create the Service Bus queue to be used as a routing destination.
New-AzServiceBusQueue -ResourceGroupName $resourceGroup `
    -Namespace $serviceBusNamespace `
    -Name $serviceBusQueueName  `
    -EnablePartitioning $False 
```

### <a name="create-a-simulated-device"></a>Cree un dispositivo simulado:

[!INCLUDE [iot-hub-include-create-simulated-device-portal](../../includes/iot-hub-include-create-simulated-device-portal.md)]

Ahora que los recursos de base están configurados, puede continuar por el enrutamiento de mensajes.

## <a name="set-up-message-routing"></a>Configuración del enrutamiento de mensajes

[!INCLUDE [iot-hub-include-create-routing-description](../../includes/iot-hub-include-create-routing-description.md)]

Para crear un punto de conexión de enrutamiento, use [Add-AzIotHubRoutingEndpoint](/powershell/module/az.iothub/Add-AzIotHubRoutingEndpoint). Para crear la ruta de mensajería para el punto de conexión, use [Add-AzIotHubRoute](/powershell/module/az.iothub/Add-AzIoTHubRoute).

### <a name="route-to-a-storage-account"></a>Enrutamiento a una cuenta de almacenamiento 

En primer lugar, configure el punto de conexión para la cuenta de almacenamiento y, después, cree la ruta de mensajes.

[!INCLUDE [iot-hub-include-blob-storage-format](../../includes/iot-hub-include-blob-storage-format.md)]

Estas son las variables utilizadas por el script que se debe establecer en la sesión de Cloud Shell:

**resourceGroup**: este campo aparece dos veces, configure ambos para el grupo de recursos.

**name**: este campo es el nombre de la instancia de IoT Hub a la cual se aplicará el enrutamiento.

**endpointName**: este campo es el nombre que identifica el punto de conexión. 

**endpointType**: este campo es el tipo de punto de conexión. Este valor debe establecerse en `azurestoragecontainer`, `eventhub`, `servicebusqueue` o `servicebustopic`. Para los fines de este tutorial, establézcalo en `azurestoragecontainer`.

**subscriptionID**: este campo se establece con el identificador de suscripción de la cuenta de Azure.

**storageConnectionString**: Este valor se recupera de la cuenta de almacenamiento configurada en el script anterior. El enrutamiento lo utiliza para acceder a la cuenta de almacenamiento.

**containerName**: este campo es el nombre del contenedor de la cuenta de almacenamiento en la que se escribirán los datos.

**Encoding**: establezca este campo en `AVRO` o `JSON`. Esta opción designa el formato de los datos almacenados. El valor predeterminado es AVRO.

**routeName**: este campo es el nombre de la ruta que se va a configurar. 

**condition**: este campo es la consulta utilizada para filtrar los mensajes enviados a este punto de conexión. La condición de consulta para que los mensajes se enruten al almacenamiento es `level="storage"`.

**enabled**: el valor predeterminado de este campo es `true`, lo cual indica que la ruta de mensajes debe estar habilitada tras la creación.

Copie este script y péguelo en la ventana de Cloud Shell.

```powershell
##### ROUTING FOR STORAGE #####

$endpointName = "ContosoStorageEndpoint"
$endpointType = "azurestoragecontainer"
$routeName = "ContosoStorageRoute"
$condition = 'level="storage"'
```

El siguiente paso es crear el punto de conexión de enrutamiento para la cuenta de almacenamiento. También se especificará el contenedor donde se almacenarán los resultados. El contenedor se creó cuando se creó la cuenta de almacenamiento.

```powershell
# Create the routing endpoint for storage.
# Specify 'AVRO' or 'JSON' for the encoding of the data.
Add-AzIotHubRoutingEndpoint `
  -ResourceGroupName $resourceGroup `
  -Name $iotHubName `
  -EndpointName $endpointName `
  -EndpointType $endpointType `
  -EndpointResourceGroup $resourceGroup `
  -EndpointSubscriptionId $subscriptionId `
  -ConnectionString $storageConnectionString  `
  -ContainerName $containerName `
  -Encoding AVRO
```

A continuación se crea la ruta de mensajes para el punto de conexión de almacenamiento. La ruta de mensajes designa dónde enviar los mensajes que cumplan la especificación de consulta.

```powershell
# Create the route for the storage endpoint.
Add-AzIotHubRoute `
   -ResourceGroupName $resourceGroup `
   -Name $iotHubName `
   -RouteName $routeName `
   -Source DeviceMessages `
   -EndpointName $endpointName `
   -Condition $condition `
   -Enabled 
```

### <a name="route-to-a-service-bus-queue"></a>Ruta a una cola de Service Bus

Ahora, configure el enrutamiento de la cola de Service Bus. Para recuperar la cadena de conexión para la cola de Service Bus, debe crear una regla de autorización con los derechos correctos definidos. El siguiente script crea una regla de autorización para la cola de Service Bus llamada `sbauthrule` y establece los derechos en `Listen Manage Send`. Una vez configurada esta regla de autorización, podrá usarla para recuperar la cadena de conexión para la cola.

```powershell
##### ROUTING FOR SERVICE BUS QUEUE #####

# Create the authorization rule for the Service Bus queue.
New-AzServiceBusAuthorizationRule `
  -ResourceGroupName $resourceGroup `
  -NamespaceName $serviceBusNamespace `
  -Queue $serviceBusQueueName `
  -Name "sbauthrule" `
  -Rights @("Manage","Listen","Send")
```

Ahora, use la regla de autorización para recuperar la clave de la cola de Service Bus. Esta regla de autorización se usará para recuperar la cadena de conexión más adelante en el script.

```powershell
$sbqkey = Get-AzServiceBusKey `
    -ResourceGroupName $resourceGroup `
    -NamespaceName $serviceBusNamespace `
    -Queue $servicebusQueueName `
    -Name "sbauthrule"
```

Ahora, configure el punto de conexión de enrutamiento y la ruta de mensajes para la cola de Service Bus. Estas son las variables utilizadas por el script que se debe establecer en la sesión de Cloud Shell:

**endpointName**: este campo es el nombre que identifica el punto de conexión. 

**endpointType**: este campo es el tipo de punto de conexión. Este valor debe establecerse en `azurestoragecontainer`, `eventhub`, `servicebusqueue` o `servicebustopic`. Para los fines de este tutorial, establézcalo en `servicebusqueue`.

**routeName**: este campo es el nombre de la ruta que se va a configurar. 

**condition**: este campo es la consulta utilizada para filtrar los mensajes enviados a este punto de conexión. La condición de consulta para que los mensajes se enruten a la cola de Service Bus es `level="critical"`.

Puede realizar el enrutamiento de mensajes para la cola de Service Bus con Azure PowerShell.

```powershell
$endpointName = "ContosoSBQueueEndpoint"
$endpointType = "servicebusqueue"
$routeName = "ContosoSBQueueRoute"
$condition = 'level="critical"'

# Add the routing endpoint, using the connection string property from the key.
Add-AzIotHubRoutingEndpoint `
  -ResourceGroupName $resourceGroup `
  -Name $iotHubName `
  -EndpointName $endpointName `
  -EndpointType $endpointType `
  -EndpointResourceGroup $resourceGroup `
  -EndpointSubscriptionId $subscriptionId `
  -ConnectionString $sbqkey.PrimaryConnectionString

# Set up the message route for the Service Bus queue endpoint.
Add-AzIotHubRoute `
   -ResourceGroupName $resourceGroup `
   -Name $iotHubName `
   -RouteName $routeName `
   -Source DeviceMessages `
   -EndpointName $endpointName `
   -Condition $condition `
   -Enabled 
```

### <a name="view-message-routing-in-the-portal"></a>Visualización del enrutamiento de mensajes en el portal

[!INCLUDE [iot-hub-include-view-routing-in-portal](../../includes/iot-hub-include-view-routing-in-portal.md)]

## <a name="next-steps"></a>Pasos siguientes

Ahora que tiene las rutas de mensajes y los recursos configurados, pase al siguiente tutorial para aprender a enviar mensajes a IoT Hub y verlos enrutados a los distintos destinos. 

> [!div class="nextstepaction"]
> [Parte 2: Visualización de los resultados del enrutamiento de mensajes](tutorial-routing-view-message-routing-results.md)
