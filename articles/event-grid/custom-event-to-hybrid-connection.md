---
title: 'Tutorial: Envío de eventos personalizados a conexiones híbridas: Event Grid'
description: 'Tutorial: Use Azure Event Grid y la CLI de Azure para publicar un tema y suscribirse a ese evento. Una conexión híbrida se usa para el punto de conexión.'
ms.date: 07/07/2020
ms.topic: tutorial
ms.custom: devx-track-azurecli
ms.openlocfilehash: 77633a717478ff143c3b084adc0b69019abb203f
ms.sourcegitcommit: 2ff0d073607bc746ffc638a84bb026d1705e543e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87832134"
---
# <a name="tutorial-route-custom-events-to-azure-relay-hybrid-connections-with-azure-cli-and-event-grid"></a>Tutorial: Enrutar eventos personalizados a Conexiones híbridas de Azure Relay con la CLI de Azure y Event Grid

Azure Event Grid es un servicio de eventos para la nube. La solución Conexiones híbridas de Azure Relay es uno de los controladores de eventos compatibles. Las conexiones híbridas se usan como el controlador de eventos cuando es necesario procesar los eventos de las aplicaciones que no tienen un punto de conexión público. Estas aplicaciones pueden estar dentro de la red empresarial corporativa. En este artículo, se usará la CLI de Azure para crear un tema personalizado, suscribirse a él y desencadenar el evento para ver el resultado. Los eventos se envían a la conexión híbrida.

## <a name="prerequisites"></a>Prerrequisitos

En este artículo, se presupone que ya tiene una conexión híbrida y una aplicación de escucha. Para empezar a trabajar con las conexiones híbridas, consulte [Introducción a Conexiones híbridas de Relay: .NET](../azure-relay/relay-hybrid-connections-dotnet-get-started.md) o [Introducción a Conexiones híbridas de Relay: nodo](../azure-relay/relay-hybrid-connections-node-get-started.md).

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

> [!NOTE]
> Si usa la CLI de Azure en la máquina local, utilice la versión 2.0.56 o superior. Para instrucciones sobre cómo instalar la versión más reciente de la CLI de Azure, consulte [Instalación de la CLI de Azure](/cli/azure/install-azure-cli).

## <a name="create-a-resource-group"></a>Crear un grupo de recursos

Los temas de Event Grid son recursos de Azure y se deben colocar en un grupo de recursos de Azure. El grupo de recursos de Azure es una colección lógica en la que se implementan y administran los recursos de Azure.

Para crear un grupo de recursos, use el comando [az group create](/cli/azure/group#az-group-create). 

En el ejemplo siguiente, se crea un grupo de recursos denominado *gridResourceGroup* en la ubicación *westus2*.

```azurecli-interactive
az group create --name gridResourceGroup --location westus2
```

## <a name="create-a-custom-topic"></a>Creación de un tema personalizado

Un tema de cuadrícula de eventos proporciona un punto de conexión definido por el usuario en el que se registran los eventos. En el ejemplo siguiente se crea el tema personalizado en el grupo de recursos. Reemplace `<topic_name>` por un nombre único para el tema personalizado. El nombre del tema de cuadrícula de eventos debe ser único porque se representa mediante una entrada DNS.

```azurecli-interactive
az eventgrid topic create --name <topic_name> -l westus2 -g gridResourceGroup
```

## <a name="subscribe-to-a-custom-topic"></a>Suscripción a un tema personalizado

Suscríbase a un tema de cuadrícula de eventos para indicar a Event Grid los eventos cuyo seguimiento desea realizar. En el ejemplo siguiente se suscribirá al tema personalizado que creó y pasará el identificador de recurso de la conexión híbrida al punto de conexión. El identificador de conexión híbrida tiene el formato siguiente:

`/subscriptions/<subscription-id>/resourceGroups/<resource-group-name>/providers/Microsoft.Relay/namespaces/<relay-namespace>/hybridConnections/<hybrid-connection-name>`

En el siguiente script, el identificador de recurso se obtiene del espacio de nombres de Relay. Se genera el identificador de la conexión híbrida y se suscribe a un tema de la cuadrícula de eventos. El script establece el tipo de punto de conexión en `hybridconnection` y se usa el identificador de conexión híbrida para el punto de conexión.

```azurecli-interactive
relayname=<namespace-name>
relayrg=<resource-group-for-relay>
hybridname=<hybrid-name>

relayid=$(az resource show --name $relayname --resource-group $relayrg --resource-type Microsoft.Relay/namespaces --query id --output tsv)
hybridid="$relayid/hybridConnections/$hybridname"
topicid=$(az eventgrid topic show --name <topic_name> -g gridResourceGroup --query id --output tsv)

az eventgrid event-subscription create \
  --source-resource-id $topicid \
  --name <event_subscription_name> \
  --endpoint-type hybridconnection \
  --endpoint $hybridid \
  --expiration-date "<yyyy-mm-dd>"
```

Tenga en cuenta que se ha establecido una [fecha de expiración](concepts.md#event-subscription-expiration) para la suscripción.

## <a name="create-application-to-process-events"></a>Creación de una aplicación para procesar eventos

Necesita una aplicación que puede recuperar eventos desde la conexión híbrida. El [ejemplo de consumidor de conexión híbrida de Microsoft Azure Event Grid para C#](https://github.com/Azure-Samples/event-grid-dotnet-hybridconnection-destination) ejecuta esa operación. Ya completó los pasos de requisitos previos.

1. Asegúrese de que tiene Visual Studio 2019 o posterior.

1. Clone el repositorio en la máquina local.

1. Cargue el proyecto HybridConnectionConsumer en Visual Studio.

1. En Program.cs, reemplace `<relayConnectionString>` y `<hybridConnectionName>` por la cadena de conexión de Relay y el nombre de la conexión híbrida que creó.

1. Compile y ejecute la aplicación desde Visual Studio.

## <a name="send-an-event-to-your-topic"></a>Envío de un evento al tema

Vamos a desencadenar un evento para ver cómo Event Grid distribuye el mensaje al punto de conexión. En este artículo se muestra cómo usar la CLI de Azure para desencadenar el evento. De manera alternativa, puede usar la [aplicación de publicador de Event Grid](https://github.com/Azure-Samples/event-grid-dotnet-publish-consume-events/tree/master/EventGridPublisher).

En primer lugar, vamos a obtener la dirección URL y la clave del tema personalizado. De nuevo, use su nombre de tema personalizado para `<topic_name>`.

```azurecli-interactive
endpoint=$(az eventgrid topic show --name <topic_name> -g gridResourceGroup --query "endpoint" --output tsv)
key=$(az eventgrid topic key list --name <topic_name> -g gridResourceGroup --query "key1" --output tsv)
```

Para simplificar este artículo, usará datos de evento de ejemplo para enviar al tema personalizado. Normalmente, una aplicación o servicio de Azure enviaría los datos del evento. CURL es una utilidad que envía solicitudes HTTP. En este artículo, usará CURL para enviar el evento al tema personalizado.

```azurecli-interactive
event='[ {"id": "'"$RANDOM"'", "eventType": "recordInserted", "subject": "myapp/vehicles/motorcycles", "eventTime": "'`date +%Y-%m-%dT%H:%M:%S%z`'", "data":{ "make": "Ducati", "model": "Monster"},"dataVersion": "1.0"} ]'
curl -X POST -H "aeg-sas-key: $key" -d "$event" $endpoint
```

La aplicación de agente de escucha debe recibir el mensaje de evento.

## <a name="clean-up-resources"></a>Limpieza de recursos
Si piensa seguir trabajando con este evento, no limpie los recursos creados en este artículo. En caso contrario, use el siguiente comando para eliminar los recursos creados en este artículo.

```azurecli-interactive
az group delete --name gridResourceGroup
```

## <a name="next-steps"></a>Pasos siguientes

Ahora que sabe cómo crear suscripciones a temas y eventos, aprenda más sobre cómo Event Grid puede ayudarle:

- [Una introducción a Azure Event Grid](overview.md)
- [Enrutamiento de eventos de Blob Storage a un punto de conexión web personalizado](../storage/blobs/storage-blob-event-quickstart.md?toc=%2fazure%2fevent-grid%2ftoc.json)
- [Supervisión de los cambios en máquinas virtuales con Azure Event Grid y Logic Apps](monitor-virtual-machine-changes-event-grid-logic-app.md)
- [Transmisión de macrodatos a un almacén de datos](event-grid-event-hubs-integration.md)