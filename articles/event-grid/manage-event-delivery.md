---
title: 'Mensajes fallidos y directivas de reintento: Azure Event Grid'
description: Describe la personalización de las opciones de entrega de eventos para Event Grid. Defina un destino de mensajes fallidos y especifique el tiempo para reintentar la entrega.
ms.topic: conceptual
ms.date: 07/20/2020
ms.openlocfilehash: 2ff1d05899fb74583489649154ffa062e857cb95
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87074884"
---
# <a name="dead-letter-and-retry-policies"></a>Mensajes fallidos y directivas de reintento

Cuando crea una suscripción a eventos, puede personalizar la configuración de entrega de estos. Este artículo muestra cómo configurar una ubicación de la cola de mensajes fallidos y personalizar la configuración de reintentos. Para obtener información acerca de estas características, consulte [Entrega y reintento de entrega de mensajes de Event Grid](delivery-and-retry.md).

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

> [!NOTE]
> Para obtener información sobre la entrega de mensajes, los reintentos y la cola de mensajes fallidos, consulte el artículo conceptual: [Entrega y reintento de entrega de mensajes de Event Grid]()

## <a name="set-dead-letter-location"></a>Establecimiento de una ubicación para los eventos fallidos

Para establecer una ubicación de la cola de mensajes fallidos, se necesita una cuenta de almacenamiento para mantener los eventos que no se pueden entregar a un punto de conexión. Los ejemplos obtienen el identificador de recurso de una cuenta de almacenamiento existente. Crean una suscripción de eventos que usa un contenedor en esa cuenta de almacenamiento para el punto de conexión de mensajes con problemas de entrega.

> [!NOTE]
> - Cree una cuenta de almacenamiento y un contenedor de blobs en el almacenamiento antes de ejecutar los comandos de este artículo.
> - El servicio Event Grid crea blobs en este contenedor. Los nombres de los blobs tendrán el nombre de la suscripción de Event Grid con todas las letras en mayúsculas. Por ejemplo, si el nombre de la suscripción es My-Blob-Subscription, los nombres de los blobs de mensajes con problemas de entrega tendrán MY-BLOB-SUBSCRIPTION (myblobcontainer/MY-BLOB-SUBSCRIPTION/2019/8/8/5/111111111-1111-1111-1111-111111111111.json). Este comportamiento sirve para proteger contra las diferencias en el control de las mayúsculas y minúsculas entre servicios de Azure.


### <a name="azure-cli"></a>Azure CLI

```azurecli-interactive
containername=testcontainer

topicid=$(az eventgrid topic show --name demoTopic -g gridResourceGroup --query id --output tsv)
storageid=$(az storage account show --name demoStorage --resource-group gridResourceGroup --query id --output tsv)

az eventgrid event-subscription create \
  --source-resource-id $topicid \
  --name <event_subscription_name> \
  --endpoint <endpoint_URL> \
  --deadletter-endpoint $storageid/blobServices/default/containers/$containername
```

Para desactivar las colas de mensajes fallidos, vuelva a ejecutar el comando para crear la suscripción de eventos, pero no proporcione un valor para `deadletter-endpoint`. No es necesario eliminar la suscripción de eventos.

> [!NOTE]
> Si usa la CLI de Azure en la máquina local, utilice la versión 2.0.56 o superior. Para instrucciones sobre cómo instalar la versión más reciente de la CLI de Azure, consulte [Instalación de la CLI de Azure](/cli/azure/install-azure-cli).

### <a name="powershell"></a>PowerShell

```azurepowershell-interactive
$containername = "testcontainer"

$topicid = (Get-AzEventGridTopic -ResourceGroupName gridResourceGroup -Name demoTopic).Id
$storageid = (Get-AzStorageAccount -ResourceGroupName gridResourceGroup -Name demostorage).Id

New-AzEventGridSubscription `
  -ResourceId $topicid `
  -EventSubscriptionName <event_subscription_name> `
  -Endpoint <endpoint_URL> `
  -DeadLetterEndpoint "$storageid/blobServices/default/containers/$containername"
```

Para desactivar las colas de mensajes fallidos, vuelva a ejecutar el comando para crear la suscripción de eventos, pero no proporcione un valor para `DeadLetterEndpoint`. No es necesario eliminar la suscripción de eventos.

> [!NOTE]
> Si usa Azure PowerShell en la máquina local, use Azure PowerShell versión 1.1.0 o superior. Descargue e instale la última versión de Azure PowerShell desde las [descargas de Azure](https://azure.microsoft.com/downloads/).

## <a name="set-retry-policy"></a>Establecimiento de la directiva de reintentos

Al crear una suscripción a Event Grid, puede establecer durante cuanto tiempo debe intentar Event Grid entregar el evento. De manera predeterminada, Event Grid lo intenta durante 24 horas (1440 minutos), o 30 veces. Puede establecer cualquiera de estos valores para la suscripción a Event Grid. El valor del período de vida del evento debe ser un entero entre 1 y 1440. El valor de reintentos máximos debe ser un entero entre 1 y 30.

No se puede configurar la [programación de reintentos](delivery-and-retry.md#retry-schedule-and-duration).

### <a name="azure-cli"></a>Azure CLI

Para establecer el período de vida del evento en otro valor distinto de 1440 minutos, use:

```azurecli-interactive
az eventgrid event-subscription create \
  -g gridResourceGroup \
  --topic-name <topic_name> \
  --name <event_subscription_name> \
  --endpoint <endpoint_URL> \
  --event-ttl 720
```

Para establecer el número máximo de reintentos en otro valor distinto a 30, use:

```azurecli-interactive
az eventgrid event-subscription create \
  -g gridResourceGroup \
  --topic-name <topic_name> \
  --name <event_subscription_name> \
  --endpoint <endpoint_URL> \
  --max-delivery-attempts 18
```

> [!NOTE]
> Si establece `event-ttl` y `max-deliver-attempts`, Event Grid usa la primera opción más próxima a la expiración para determinar cuándo se debe detener la entrega de eventos. Por ejemplo, si establece 30 minutos como período de vida (TTL) y 10 intentos de entrega máximos. Cuando un evento no se entrega después de 30 minutos (o) no se entrega después de 10 intentos, lo que ocurra primero, el evento se pone en la cola de mensajes fallidos.  

### <a name="powershell"></a>PowerShell

Para establecer el período de vida del evento en otro valor distinto de 1440 minutos, use:

```azurepowershell-interactive
$topicid = (Get-AzEventGridTopic -ResourceGroupName gridResourceGroup -Name demoTopic).Id

New-AzEventGridSubscription `
  -ResourceId $topicid `
  -EventSubscriptionName <event_subscription_name> `
  -Endpoint <endpoint_URL> `
  -EventTtl 720
```

Para establecer el número máximo de reintentos en otro valor distinto a 30, use:

```azurepowershell-interactive
$topicid = (Get-AzEventGridTopic -ResourceGroupName gridResourceGroup -Name demoTopic).Id

New-AzEventGridSubscription `
  -ResourceId $topicid `
  -EventSubscriptionName <event_subscription_name> `
  -Endpoint <endpoint_URL> `
  -MaxDeliveryAttempt 18
```

> [!NOTE]
> Si establece `event-ttl` y `max-deliver-attempts`, Event Grid usa la primera opción más próxima a la expiración para determinar cuándo se debe detener la entrega de eventos. Por ejemplo, si establece 30 minutos como período de vida (TTL) y 10 intentos de entrega máximos. Cuando un evento no se entrega después de 30 minutos (o) no se entrega después de 10 intentos, lo que ocurra primero, el evento se pone en la cola de mensajes fallidos.  

## <a name="next-steps"></a>Pasos siguientes

* Para una aplicación de ejemplo que usa una aplicación de función de Azure para procesar los eventos de mensajes fallidos, consulte [Ejemplos de mensajes fallidos de Azure Event Grid para .NET](https://azure.microsoft.com/resources/samples/event-grid-dotnet-handle-deadlettered-events/).
* Para obtener información acerca de los reintentos y las entregas de eventos, consulte [Entrega y reintento de entrega de mensajes de Event Grid](delivery-and-retry.md).
* Para obtener una introducción a Event Grid, vea [Acerca de Event Grid](overview.md).
* Para comenzar a usar rápidamente Event Grid, vea [Creación y enrutamiento de eventos personalizados con Azure Event Grid](custom-event-quickstart.md).
