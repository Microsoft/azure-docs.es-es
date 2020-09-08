---
title: 'Creación de un centro de eventos con la captura habilitada: Azure Event Hubs | Microsoft Docs'
description: Creación de un espacio de nombres de Azure Event Hubs con un centro de eventos y habilitación de la funcionalidad de captura mediante una plantilla de Azure Resource Manager
ms.topic: conceptual
ms.date: 06/23/2020
ms.custom: devx-track-azurecli, devx-track-azurepowershell
ms.openlocfilehash: 1ba98bb47d53e3f95a1f212c5dc9459f983a30c2
ms.sourcegitcommit: 656c0c38cf550327a9ee10cc936029378bc7b5a2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89079768"
---
# <a name="create-a-namespace-with-event-hub-and-enable-capture-using-a-template"></a>Creación de un espacio de nombres con Event Hubs y habilitación de la característica Capture mediante una plantilla

En este artículo se muestra cómo usar una plantilla de Azure Resource Manager que crea un espacio de nombres de [Event Hubs](./event-hubs-about.md), con una instancia de centro de eventos, y también habilita la [característica Capture](event-hubs-capture-overview.md) en el centro de eventos. En este artículo se describe cómo definir los recursos que se implementan y los parámetros que se especifican cuando se ejecuta la implementación. Puede usar esta plantilla para sus propias implementaciones o personalizarla para satisfacer sus necesidades.

En este artículo también se muestra cómo especificar los eventos que se capturan en instancias de Azure Storage Blob o de Azure Data Lake Store, basándose en el destino que elija.

Para más información sobre la creación de plantillas, consulte [Creación de plantillas de Azure Resource Manager][Authoring Azure Resource Manager templates]. Para la sintaxis y las propiedades de JSON que se usan en una plantilla, consulte [Tipos de recursos de Microsoft.EventHub](/azure/templates/microsoft.eventhub/allversions).

Para obtener más información sobre patrones y prácticas de convenciones de nomenclatura de recursos de Azure, consulte las [convenciones de nomenclatura de los recursos de Azure][Azure Resources naming conventions].

Para ver las plantillas completas, haga clic en los siguientes vínculos de GitHub:

- [Centro de eventos y habilitación de Capture en una plantilla de Storage][Event Hub and enable Capture to Storage template] 
- [Centro de eventos y habilitación de Capture en una plantilla de Azure Data Lake Store][Event Hub and enable Capture to Azure Data Lake Store template]

> [!NOTE]
> Para buscar las últimas plantillas, visite la galería de [Plantillas de inicio rápido de Azure][Azure Quickstart Templates] y busque Event Hubs.
> 
> 

## <a name="what-will-you-deploy"></a>¿Qué va a implementar?

Con esta plantilla, implementará un espacio de nombres de Event Hubs con un centro de eventos y habilitará la [captura de Event Hubs](event-hubs-capture-overview.md). Event Hubs Capture le permite entregar automáticamente los datos de streaming de sus instancias de Event Hubs a una instancia de Azure Blob Storage o Azure Data Lake Store, en el tiempo especificado o el intervalo de tamaño que prefiera. Haga clic en el botón siguiente para habilitar Event Hubs Capture en Azure Storage:

[![Implementación en Azure](./media/event-hubs-resource-manager-namespace-event-hub/deploybutton.png)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F201-eventhubs-create-namespace-and-enable-capture%2Fazuredeploy.json)

Haga clic en el botón siguiente para habilitar Event Hubs Capture en Azure Data Lake Store:

[![Implementación en Azure](./media/event-hubs-resource-manager-namespace-event-hub/deploybutton.png)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F201-eventhubs-create-namespace-and-enable-capture-for-adls%2Fazuredeploy.json)

## <a name="parameters"></a>Parámetros

Con el Administrador de recursos de Azure, se definen los parámetros de los valores que desea especificar al implementar la plantilla. La plantilla incluye una sección denominada `Parameters` que contiene todos los valores de los parámetros. Debe definir un parámetro para esos valores que variarán según el proyecto que vaya a implementar o según el entorno en el que vaya a realizar la implementación. No defina parámetros para valores que siempre permanezcan igual. Cada valor de parámetro se usa en la plantilla para definir los recursos que se implementan.

La plantilla define los parámetros siguientes.

### <a name="eventhubnamespacename"></a>eventHubNamespaceName

El nombre del espacio de nombres de Event Hubs que se creará.

```json
"eventHubNamespaceName":{  
     "type":"string",
     "metadata":{  
         "description":"Name of the EventHub namespace"
      }
}
```

### <a name="eventhubname"></a>eventHubName

El nombre del centro de eventos creado en el espacio de nombres de Event Hubs.

```json
"eventHubName":{  
    "type":"string",
    "metadata":{  
        "description":"Name of the event hub"
    }
}
```

### <a name="messageretentionindays"></a>messageRetentionInDays

El número de días que se deben conservar los mensajes en el centro de eventos. 

```json
"messageRetentionInDays":{
    "type":"int",
    "defaultValue": 1,
    "minValue":"1",
    "maxValue":"7",
    "metadata":{
       "description":"How long to retain the data in event hub"
     }
 }
```

### <a name="partitioncount"></a>partitionCount

El número de particiones que se van a crear en el centro de eventos.

```json
"partitionCount":{
    "type":"int",
    "defaultValue":2,
    "minValue":2,
    "maxValue":32,
    "metadata":{
        "description":"Number of partitions chosen"
    }
 }
```

### <a name="captureenabled"></a>captureEnabled

Habilita la funcionalidad de captura en el centro de eventos.

```json
"captureEnabled":{
    "type":"string",
    "defaultValue":"true",
    "allowedValues": [
    "false",
    "true"],
    "metadata":{
        "description":"Enable or disable the Capture for your event hub"
    }
 }
```
### <a name="captureencodingformat"></a>captureEncodingFormat

El formato de codificación que especifica para serializar los datos de eventos.

```json
"captureEncodingFormat":{
    "type":"string",
    "defaultValue":"Avro",
    "allowedValues":[
    "Avro"],
    "metadata":{
        "description":"The encoding format in which Capture serializes the EventData"
    }
}
```

### <a name="capturetime"></a>captureTime

El intervalo de tiempo en el que Event Hubs Capture comienza a capturar los datos.

```json
"captureTime":{
    "type":"int",
    "defaultValue":300,
    "minValue":60,
    "maxValue":900,
    "metadata":{
         "description":"The time window in seconds for the capture"
    }
}
```

### <a name="capturesize"></a>captureSize
El intervalo de tamaño en el que Capture comienza a capturar los datos.

```json
"captureSize":{
    "type":"int",
    "defaultValue":314572800,
    "minValue":10485760,
    "maxValue":524288000,
    "metadata":{
        "description":"The size window in bytes for capture"
    }
}
```

### <a name="capturenameformat"></a>captureNameFormat

El formato de nombre que usa Event Hubs Capture para escribir archivos Avro. Tenga en cuenta que un formato de nombre de Capture debe contener los campos `{Namespace}`, `{EventHub}`, `{PartitionId}`, `{Year}`, `{Month}`, `{Day}`, `{Hour}`, `{Minute}` y `{Second}`. Estos se pueden organizar en cualquier orden, con o sin delimitadores.
 
```json
"captureNameFormat": {
      "type": "string",
      "defaultValue": "{Namespace}/{EventHub}/{PartitionId}/{Year}/{Month}/{Day}/{Hour}/{Minute}/{Second}",
      "metadata": {
        "description": "A Capture Name Format must contain {Namespace}, {EventHub}, {PartitionId}, {Year}, {Month}, {Day}, {Hour}, {Minute} and {Second} fields. These can be arranged in any order with or without delimeters. E.g.  Prod_{EventHub}/{Namespace}\\{PartitionId}_{Year}_{Month}/{Day}/{Hour}/{Minute}/{Second}"
      }
    }
  
```

### <a name="apiversion"></a>apiVersion

La versión de API de la plantilla.

```json
 "apiVersion":{  
    "type":"string",
    "defaultValue":"2017-04-01",
    "metadata":{  
        "description":"ApiVersion used by the template"
    }
 }
```

Si elige como destino Azure Storage, use los parámetros siguientes.

### <a name="destinationstorageaccountresourceid"></a>destinationStorageAccountResourceId

La funcionalidad de captura requiere un identificador de recurso de una cuenta de Azure Storage para habilitar esta funcionalidad en la cuenta deseada de Storage.

```json
 "destinationStorageAccountResourceId":{
    "type":"string",
    "metadata":{
        "description":"Your existing Storage account resource ID where you want the blobs be captured"
    }
 }
```

### <a name="blobcontainername"></a>blobContainerName

El contenedor de blobs en el que se van a capturar los datos del evento.

```json
 "blobContainerName":{
    "type":"string",
    "metadata":{
        "description":"Your existing storage container in which you want the blobs captured"
    }
}
```

Use los parámetros siguientes si elige Azure Data Lake Storage Gen1 como destino. Debe establecer permisos en la ruta de acceso de Data Lake Store, en el que desea capturar el evento. Para establecer permisos, vea [Captura de datos para Azure Data Lake Storage Gen1](event-hubs-capture-enable-through-portal.md#capture-data-to-azure-data-lake-storage-gen-1).

### <a name="subscriptionid"></a>subscriptionId

El identificador de suscripción para el espacio de nombres de Event Hubs y Azure Data Lake Store. Ambos recursos deben estar en el mismo identificador de suscripción.

```json
"subscriptionId": {
    "type": "string",
    "metadata": {
        "description": "Subscription ID of both Azure Data Lake Store and Event Hubs namespace"
     }
 }
```

### <a name="datalakeaccountname"></a>dataLakeAccountName

El nombre de Azure Data Lake Store para los eventos capturados.

```json
"dataLakeAccountName": {
    "type": "string",
    "metadata": {
        "description": "Azure Data Lake Store name"
    }
}
```

### <a name="datalakefolderpath"></a>dataLakeFolderPath

La ruta de acceso de la carpeta de destino para los eventos capturados. Es la carpeta de Data Lake Store en la que se insertarán los eventos durante la operación de captura. Para establecer los permisos de esta carpeta, consulte [Uso de Azure Data Lake Store para capturar datos de Event Hubs](../data-lake-store/data-lake-store-archive-eventhub-capture.md).

```json
"dataLakeFolderPath": {
    "type": "string",
    "metadata": {
        "description": "Destination capture folder path"
    }
}
```

## <a name="resources-to-deploy-for-azure-storage-as-destination-to-captured-events"></a>Recursos que deben implementarse para Azure Storage como destino para los eventos capturados

Crea un espacio de nombres del tipo **EventHub**, con un centro de eventos, y también habilita la característica Capture en Azure Blob Storage.

```json
"resources":[  
      {  
         "apiVersion":"[variables('ehVersion')]",
         "name":"[parameters('eventHubNamespaceName')]",
         "type":"Microsoft.EventHub/Namespaces",
         "location":"[variables('location')]",
         "sku":{  
            "name":"Standard",
            "tier":"Standard"
         },
         "resources": [
    {
      "apiVersion": "2017-04-01",
      "name": "[parameters('eventHubNamespaceName')]",
      "type": "Microsoft.EventHub/Namespaces",
      "location": "[resourceGroup().location]",
      "sku": {
        "name": "Standard"
      },
      "properties": {
        "isAutoInflateEnabled": "true",
        "maximumThroughputUnits": "7"
      },
      "resources": [
        {
          "apiVersion": "2017-04-01",
          "name": "[parameters('eventHubName')]",
          "type": "EventHubs",
          "dependsOn": [
            "[concat('Microsoft.EventHub/namespaces/', parameters('eventHubNamespaceName'))]"
          ],
          "properties": {
            "messageRetentionInDays": "[parameters('messageRetentionInDays')]",
            "partitionCount": "[parameters('partitionCount')]",
            "captureDescription": {
              "enabled": "true",
              "skipEmptyArchives": false,
              "encoding": "[parameters('captureEncodingFormat')]",
              "intervalInSeconds": "[parameters('captureTime')]",
              "sizeLimitInBytes": "[parameters('captureSize')]",
              "destination": {
                "name": "EventHubArchive.AzureBlockBlob",
                "properties": {
                  "storageAccountResourceId": "[parameters('destinationStorageAccountResourceId')]",
                  "blobContainer": "[parameters('blobContainerName')]",
                  "archiveNameFormat": "[parameters('captureNameFormat')]"
                }
              }
            }
          }

        }
      ]
    }
  ]
```

## <a name="resources-to-deploy-for-azure-data-lake-store-as-destination"></a>Recursos que deben implementarse para Azure Data Lake Store como destino

Crea un espacio de nombres de tipo **EventHub**, con una instancia de Event Hubs, y también habilita la característica Capture en Azure Data Lake Store.

```json
 "resources": [
        {
            "apiVersion": "2017-04-01",
            "name": "[parameters('namespaceName')]",
            "type": "Microsoft.EventHub/Namespaces",
            "location": "[variables('location')]",
            "sku": {
                "name": "Standard",
                "tier": "Standard"
            },
            "resources": [
                {
                    "apiVersion": "2017-04-01",
                    "name": "[parameters('eventHubName')]",
                    "type": "EventHubs",
                    "dependsOn": [
                        "[concat('Microsoft.EventHub/namespaces/', parameters('namespaceName'))]"
                    ],
                    "properties": {
                        "path": "[parameters('eventHubName')]",
                        "captureDescription": {
                            "enabled": "true",
                            "skipEmptyArchives": false,
                            "encoding": "[parameters('archiveEncodingFormat')]",
                            "intervalInSeconds": "[parameters('captureTime')]",
                            "sizeLimitInBytes": "[parameters('captureSize')]",
                            "destination": {
                                "name": "EventHubArchive.AzureDataLake",
                                "properties": {
                                    "DataLakeSubscriptionId": "[parameters('subscriptionId')]",
                                    "DataLakeAccountName": "[parameters('dataLakeAccountName')]",
                                    "DataLakeFolderPath": "[parameters('dataLakeFolderPath')]",
                                    "ArchiveNameFormat": "[parameters('captureNameFormat')]"
                                }
                            }
                        }
                    }
                }
            ]
        }
    ]
```

> [!NOTE]
> Puede habilitar o deshabilitar el envío de archivos vacíos cuando no se producen eventos durante la ventura de Capture mediante la propiedad **skipEmptyArchives**. 

## <a name="commands-to-run-deployment"></a>Comandos para ejecutar la implementación

[!INCLUDE [app-service-deploy-commands](../../includes/app-service-deploy-commands.md)]

## <a name="powershell"></a>PowerShell

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Implemente la plantilla para habilitar Event Hubs Capture en Azure Storage:
 
```powershell
New-AzResourceGroupDeployment -ResourceGroupName \<resource-group-name\> -TemplateFile https://raw.githubusercontent.com/azure/azure-quickstart-templates/master/201-eventhubs-create-namespace-and-enable-capture/azuredeploy.json
```

Implemente la plantilla para habilitar Event Hubs Capture en Azure Data Lake Store:

```powershell
New-AzResourceGroupDeployment -ResourceGroupName \<resource-group-name\> -TemplateFile https://raw.githubusercontent.com/azure/azure-quickstart-templates/master/201-eventhubs-create-namespace-and-enable-capture-for-adls/azuredeploy.json
```

## <a name="azure-cli"></a>Azure CLI

Azure Blob Storage como destino:

```azurecli
az group deployment create \<my-resource-group\> \<my-deployment-name\> --template-uri [https://raw.githubusercontent.com/azure/azure-quickstart-templates/master/201-eventhubs-create-namespace-and-enable-capture/azuredeploy.json][]
```

Azure Data Lake Store como destino:

```azurecli
az group deployment create \<my-resource-group\> \<my-deployment-name\> --template-uri [https://raw.githubusercontent.com/azure/azure-quickstart-templates/master/201-eventhubs-create-namespace-and-enable-capture-for-adls/azuredeploy.json][]
```

## <a name="next-steps"></a>Pasos siguientes

También puede configurar la funcionalidad de captura de Event Hubs mediante [Azure Portal](https://portal.azure.com). Para más información, consulte [Habilitación de la funcionalidad de captura de Event Hubs mediante Azure Portal](event-hubs-capture-enable-through-portal.md).

Para más información acerca de Event Hubs, visite los vínculos siguientes:

* [Información general de Event Hubs](./event-hubs-about.md)
* [Creación de un centro de eventos](event-hubs-create.md)
* [Preguntas más frecuentes sobre Event Hubs](event-hubs-faq.md)

[Authoring Azure Resource Manager templates]: ../azure-resource-manager/templates/template-syntax.md
[Azure Quickstart Templates]:  https://azure.microsoft.com/documentation/templates/?term=event+hubs
[Azure Resources naming conventions]: /azure/cloud-adoption-framework/ready/azure-best-practices/naming-and-tagging
[Event hub and enable Capture to Storage template]: https://github.com/Azure/azure-quickstart-templates/tree/master/201-eventhubs-create-namespace-and-enable-capture
[Event hub and enable Capture to Azure Data Lake Store template]: https://github.com/Azure/azure-quickstart-templates/tree/master/201-eventhubs-create-namespace-and-enable-capture-for-adls
