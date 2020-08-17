---
title: Adición de particiones de forma dinámica a un centro de eventos en Azure Event Hubs
description: En este artículo se muestra cómo agregar particiones de forma dinámica a un centro de eventos en Azure Event Hubs.
ms.topic: how-to
ms.date: 06/23/2020
ms.openlocfilehash: 4a729147eaa11497c66f82a9764dfee9492786b9
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87002546"
---
# <a name="dynamically-add-partitions-to-an-event-hub-apache-kafka-topic-in-azure-event-hubs"></a>Agregar particiones de forma dinámica a un centro de eventos (tema Apache Kafka) en Azure Event Hubs
Event Hubs proporciona streaming de mensajes mediante un patrón de consumidor con particiones en el que cada consumidor lee solo un subconjunto específico o una partición del flujo de mensajes. Este patrón permite un escalado horizontal para el procesamiento de eventos y ofrece otras características centradas en los flujos que no están disponibles en las colas y los temas. Una partición es una secuencia ordenada de eventos que se mantiene en un centro de eventos. A medida que llegan eventos más recientes, se agregan al final de esta secuencia. Para más información sobre las particiones en general, consulte [Particiones](event-hubs-scalability.md#partitions).

Puede especificar el número de particiones en el momento de crear un centro de eventos. En algunos escenarios, puede que tenga que agregar particiones después de crear el centro de eventos. En este artículo se describe cómo agregar particiones de forma dinámica a un centro de eventos existente. 

> [!IMPORTANT]
> La adición dinámica de particiones solo está disponible en clústeres de Event Hubs **dedicados**.

> [!NOTE]
> En el caso de los clientes de Apache Kafka, un **centro de eventos** se asigna a un **tema de Kafka**. Para obtener más asignaciones entre Azure Event Hubs y Apache Kafka, consulte [Asignación conceptual de Kafka y Event Hubs](event-hubs-for-kafka-ecosystem-overview.md#kafka-and-event-hub-conceptual-mapping).


## <a name="update-the-partition-count"></a>Actualización del recuento de particiones
En esta sección se muestra cómo actualizar el recuento de particiones de un centro de eventos de maneras diferentes (PowerShell, CLI, etc.).

### <a name="powershell"></a>PowerShell
Use el comando [Set-AzureRmEventHub](/powershell/module/azurerm.eventhub/Set-AzureRmEventHub?view=azurermps-6.13.0) de PowerShell para actualizar las particiones en un centro de eventos. 

```azurepowershell-interactive
Set-AzureRmEventHub -ResourceGroupName MyResourceGroupName -Namespace MyNamespaceName -Name MyEventHubName -partitionCount 12
```

### <a name="cli"></a>CLI
Use el comando [`az eventhubs eventhub update`](/cli/azure/eventhubs/eventhub?view=azure-cli-latest#az-eventhubs-eventhub-update) de la CLI para actualizar particiones en un centro de eventos. 

```azurecli-interactive
az eventhubs eventhub update --resource-group MyResourceGroupName --namespace-name MyNamespaceName --name MyEventHubName --partition-count 12
```

### <a name="resource-manager-template"></a>Plantilla de Resource Manager
Actualice el valor de la propiedad `partitionCount` en la plantilla de Resource Manager y vuelva a implementar la plantilla para actualizar el recurso. 

```json
    {
        "apiVersion": "2017-04-01",
        "type": "Microsoft.EventHub/namespaces/eventhubs",
        "name": "[concat(parameters('namespaceName'), '/', parameters('eventHubName'))]",
        "location": "[parameters('location')]",
        "dependsOn": [
            "[resourceId('Microsoft.EventHub/namespaces', parameters('namespaceName'))]"
        ],
        "properties": {
            "messageRetentionInDays": 7,
            "partitionCount": 12
        }
    }
```

### <a name="apache-kafka"></a>Apache Kafka
Use la API de `AlterTopics` (por ejemplo, a través de la herramienta de **temas de Kafka** de la CLI) para aumentar el número de particiones. Para obtener más información, consulte [Modificación de temas de Kafka](http://kafka.apache.org/documentation/#basic_ops_modify_topic). 

## <a name="event-hubs-clients"></a>Clientes de Event Hubs
Echemos un vistazo al comportamiento de los clientes de Event Hubs cuando se actualiza el recuento de particiones en un centro de eventos. 

Cuando se agrega una partición a un centro de eventos existente, el cliente del centro de eventos recibe `MessagingException` del servicio que informa a los clientes de que los metadatos de la entidad (la entidad es el centro de eventos y los metadatos son la información de la partición) se han modificado. Los clientes volverán a abrir automáticamente los vínculos de AMQP, que luego recopilarán la información de metadatos modificada. Los clientes seguirán funcionando con normalidad.

### <a name="senderproducer-clients"></a>Clientes de remitente/productor
Event Hubs proporciona tres opciones de remitente:

- **Remitente de la partición**: en este escenario, los clientes envían eventos directamente a una partición. Aunque las particiones son identificables y se pueden enviar eventos directamente a estas, no se recomienda este patrón. La adición de particiones no afecta a este escenario. Se recomienda reiniciar las aplicaciones para que puedan detectar las particiones recién agregadas. 
- **Remitente de la clave de partición**: en este escenario, los clientes envían los eventos con una clave para que todos los eventos que pertenecen a esa clave terminen en la misma partición. En este caso, el servicio aplica un algoritmo hash a la clave y la enruta a la partición correspondiente. La actualización del recuento de particiones puede provocar problemas de desorden debido a un cambio de hash. Por lo tanto, si le preocupa el orden, asegúrese de que la aplicación consume todos los eventos de las particiones existentes antes de aumentar el número de particiones.
- **Remitente de round robin (predeterminado)** : en este escenario, el servicio Event Hubs aplica el método round robin a los eventos en las particiones. El servicio de Event Hubs es consciente de los cambios en el número de particiones y se enviará a nuevas particiones en cuestión de segundos después de modificar el número de particiones.

### <a name="receiverconsumer-clients"></a>Clientes del receptor/consumidor
Event Hubs proporciona receptores directos y una biblioteca de consumidor sencilla denominada [host del procesador de eventos (SDK antiguo)](event-hubs-event-processor-host.md) o [procesador de eventos (SDK nuevo)](event-processor-balance-partition-load.md).

- **Receptores directos**: los receptores directos escuchan particiones específicas. Su comportamiento en tiempo de ejecución no se ve afectado cuando las particiones se escalan horizontalmente para un centro de eventos. La aplicación que utiliza receptores directos debe encargarse de recoger las nuevas particiones y asignar los receptores en consecuencia.
- **Host del procesador de eventos**: este cliente no actualiza automáticamente los metadatos de la entidad. Por lo tanto, no se dará cuenta del aumento del número de particiones. Al volver a crear una instancia del procesador de eventos, se generará una captura de metadatos de entidad que, a su vez, creará nuevos blobs para las particiones recién agregadas. Los blobs preexistentes no se verán afectados. Se recomienda reiniciar todas las instancias del procesador de eventos para garantizar que todas las instancias sean conscientes las particiones recién agregadas y que el equilibrio de carga se controle correctamente entre los consumidores.

    Si usa la versión anterior del SDK de .NET ([WindowsAzure.ServiceBus](https://www.nuget.org/packages/WindowsAzure.ServiceBus/)), el host del procesador de eventos quita un punto de control existente al reiniciarse si el recuento de particiones del punto de control no coincide con el recuento de particiones capturado del servicio. Este comportamiento puede afectar a la aplicación. 

## <a name="apache-kafka-clients"></a>Clientes de Apache Kafka
En esta sección se describe el comportamiento de los clientes de Apache Kafka que usan el punto de conexión de Kafka de Azure Event Hubs cuando se actualiza el recuento de particiones de un centro de eventos. 

Los clientes de Kafka que usan Event Hubs con el protocolo de Apache Kafka se comportan de manera diferente a los clientes del centro de eventos que usan el protocolo AMQP. Los clientes de Kafka actualizan sus metadatos una vez cada `metadata.max.age.ms` milisegundos. Este valor se especifica en las configuraciones de cliente. Las bibliotecas de `librdkafka` también usan la misma configuración. Las actualizaciones de metadatos informan a los clientes de los cambios en el servicio, incluidos los aumentos del número de particiones. Para obtener una lista de configuraciones, consulte [Configuraciones de Apache Kafka para Event Hubs](apache-kafka-configurations.md).

### <a name="senderproducer-clients"></a>Clientes de remitente/productor
Los productores siempre dictan que las solicitudes de envío contienen el destino de la partición de cada conjunto de registros producidos. Por lo tanto, la creación de particiones se realiza en el lado cliente con la vista de metadatos del agente del productor. Una vez que las nuevas particiones se agreguen a la vista de metadatos del productor, estarán disponibles para las solicitudes del productor.

### <a name="consumerreceiver-clients"></a>Clientes de consumidor/receptor
Cuando un miembro del grupo de consumidores realiza una actualización de metadatos y recoge las particiones recién creadas, ese miembro inicia un reequilibrio de grupo. Posteriormente, los metadatos del consumidor se actualizarán para todos los miembros del grupo y el líder de reequilibrio asignado realizará la asignación de las nuevas particiones.

## <a name="recommendations"></a>Recomendaciones

- Si usa la clave de partición con las aplicaciones de productor y depende del hash de clave para garantizar el orden en una partición, no se recomienda agregar particiones de forma dinámica. 

    > [!IMPORTANT]
    > Aunque los datos existentes conservan el orden, el hash de partición se interrumpirá para los mensajes con hash aplicados cuando el número de particiones cambie debido a la adición de particiones.
- Se recomienda agregar una partición a un tema existente o a una instancia del centro de eventos en los siguientes casos:
    - Cuando se usa el método round robin (predeterminado) para enviar eventos.
     - Con estrategias de creación de particiones predeterminadas de Kafka, como Sticky Assignor.


## <a name="next-steps"></a>Pasos siguientes
Para más información sobre las particiones, consulte [Particiones](event-hubs-scalability.md#partitions).

