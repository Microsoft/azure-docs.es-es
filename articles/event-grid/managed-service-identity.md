---
title: Entrega de evento con Managed Service Identity
description: En este artículo se describe cómo habilitar Managed Service Identity para un tema de Azure Event Grid. Úselo para reenviar eventos a los destinos admitidos.
ms.topic: how-to
ms.date: 07/07/2020
ms.openlocfilehash: 7eaa3ddd43cc68a99ad7c2bab66630f30d4960c9
ms.sourcegitcommit: 3d56d25d9cf9d3d42600db3e9364a5730e80fa4a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/03/2020
ms.locfileid: "87534250"
---
# <a name="event-delivery-with-a-managed-identity"></a>Entrega de evento con una identidad administrada
En este artículo se describe cómo habilitar una [identidad de servicio administrada](../active-directory/managed-identities-azure-resources/overview.md) de temas o dominios de Azure Event Grid. Úselo para reenviar eventos a destinos compatibles, como colas y temas de Service Bus, centros de eventos y cuentas de almacenamiento.

Estos son los pasos que se describen en detalle en este artículo:
1. Cree un tema o un dominio con una identidad asignada por el sistema, o bien actualice un tema o dominio existente para habilitar la identidad. 
1. Agregue la identidad a un rol adecuado (por ejemplo, Remitente de los datos de Service Bus) en el destino (por ejemplo, una cola de Service Bus).
1. Al crear suscripciones de eventos, habilite el uso de la identidad para enviar eventos al destino. 

## <a name="create-a-topic-or-domain-with-an-identity"></a>Creación de un tema o un dominio con una identidad
En primer lugar, echemos un vistazo a cómo crear un tema o un dominio con una identidad administrada por el sistema.

### <a name="use-the-azure-portal"></a>Uso de Azure Portal
Puede habilitar la identidad de un tema o dominio asignada por el sistema mientras la crea en Azure Portal. La siguiente imagen muestra cómo habilitar una identidad administrada por el sistema de un tema. Básicamente, se selecciona la opción **Enable system assigned identity** (Habilitar identidad asignada por el sistema) en la página de **opciones avanzadas** del Asistente para crear temas. También verá esta opción en la página de **opciones avanzadas** del Asistente para crear dominios. 

![Habilitación de la identidad al crear un tema](./media/managed-service-identity/create-topic-identity.png)

### <a name="use-the-azure-cli"></a>Uso de la CLI de Azure
También puede usar la CLI de Azure para crear un tema o un dominio con una identidad asignada por el sistema. Use el comando `az eventgrid topic create` con el parámetro `--identity` establecido en `systemassigned`. Si no especifica un valor para este parámetro, se utiliza el valor predeterminado `noidentity`. 

```azurecli-interactive
# create a topic with a system-assigned identity
az eventgrid topic create -g <RESOURCE GROUP NAME> --name <TOPIC NAME> -l <LOCATION>  --identity systemassigned
```

Del mismo modo, puede usar el comando `az eventgrid domain create` para crear un dominio con una identidad administrada por el sistema.

## <a name="enable-an-identity-for-an-existing-topic-or-domain"></a>Habilitación de una identidad de un tema o dominio existente
En la última sección, ha aprendido a habilitar la identidad administrada por el sistema durante la creación de un tema o un dominio. En esta sección, aprenderá a habilitar una identidad administrada por el sistema de un tema o dominio existente. 

### <a name="use-the-azure-portal"></a>Uso de Azure Portal
El siguiente procedimiento muestra cómo habilitar la identidad administrada por el sistema para un tema. Los pasos para habilitar una identidad para un dominio son similares. 

1. Vaya a [Azure Portal](https://portal.azure.com).
2. Busque **temas de Event Grid** en la barra de búsqueda de la parte superior.
3. Seleccione el **tema** para el que desea habilitar la identidad administrada. 
4. Cambie a la pestaña **Identidad**. 
5. **Active** el conmutador para habilitar la identidad. 
1. Seleccione **Guardar** en la barra de herramientas para guardar la configuración. 

    :::image type="content" source="./media/managed-service-identity/identity-existing-topic.png" alt-text="Página de identidad de un tema"::: 

Puede usar pasos similares para habilitar una identidad de un dominio de Event Grid.

### <a name="use-the-azure-cli"></a>Uso de la CLI de Azure
Use el comando `az eventgrid topic update` con `--identity` establecido en `systemassigned` para habilitar la identidad asignada por el sistema para un tema existente. Si desea deshabilitar la identidad, especifique `noidentity` como valor. 

```azurecli-interactive
# Update the topic to assign a system-assigned identity. 
az eventgrid topic update -g $rg --name $topicname --identity systemassigned --sku basic 
```

El comando para actualizar un dominio existente es similar (`az eventgrid domain update`).

## <a name="supported-destinations-and-azure-roles"></a>Destinos admitidos y roles de Azure
Después de habilitar la identidad del tema o dominio de Event Grid, Azure crea automáticamente una identidad en Azure Active Directory. Agregue esta identidad a los roles de Azure adecuados para que el tema o el dominio pueda reenviar eventos a los destinos admitidos. Por ejemplo, agregue la identidad al rol **Remitente de los datos de Azure Event Hubs** en un espacio de nombres de Azure Event Hubs de modo que el tema de Event Grid pueda reenviar eventos a los centros de eventos en dicho espacio de nombres. 

Actualmente, Azure Event Grid admite temas o dominios configurados con una identidad administrada asignada por el sistema para reenviar eventos a los siguientes destinos. En esta tabla también se proporcionan los roles en los que debe estar la identidad para que el tema pueda reenviar los eventos.

| Destination | Rol de Azure | 
| ----------- | --------- | 
| Colas y temas de Service Bus | [Emisor de datos de Azure Service Bus](../service-bus-messaging/authenticate-application.md#azure-built-in-roles-for-azure-service-bus) |
| Azure Event Hubs | [Emisor de datos de Azure Event Hubs](../event-hubs/authorize-access-azure-active-directory.md#azure-built-in-roles-for-azure-event-hubs) | 
| Azure Blob Storage | [Colaborador de datos de blobs de almacenamiento](../storage/common/storage-auth-aad-rbac-portal.md#azure-roles-for-blobs-and-queues) |
| Azure Queue Storage |[Emisor de mensajes de datos de la cola de Storage](../storage/common/storage-auth-aad-rbac-portal.md#azure-roles-for-blobs-and-queues) | 

## <a name="add-an-identity-to-azure-roles-on-destinations"></a>Incorporación de una identidad a los roles de Azure en los destinos
En esta sección se describe cómo agregar la identidad del tema o dominio a un rol de Azure. 

### <a name="use-the-azure-portal"></a>Uso de Azure Portal
Puede usar Azure Portal para asignar la identidad del tema o del dominio a un rol adecuado, de modo que el tema o el dominio puedan reenviar eventos al destino. 

En el siguiente ejemplo se agrega una identidad administrada de un tema de Event Grid denominado **msitesttopic** al rol **Remitente de los datos de Azure Event Hubs** de un espacio de nombres de Service Bus que contiene un recurso de cola o tema. Al agregar al rol en el nivel de espacio de nombres, el tema puede reenviar eventos a todas las entidades en el espacio de nombres. 

1. Vaya al **espacio de nombres de Service Bus** en [Azure Portal](https://portal.azure.com). 
1. Seleccione **Control de acceso** en el panel izquierdo. 
1. Seleccione **Agregar** en la sección **Agregar una asignación de roles**. 
1. En la página **Agregar una asignación de roles**, siga estos pasos:
    1. Seleccione el rol. En este caso, es **Emisor de datos de Azure Event Hubs** 
    1. Seleccione la **identidad** para su tema o dominio. 
    1. Para guardar la configuración, seleccione **Guardar**.

Los pasos son similares para agregar una identidad a otros roles que se mencionan en la tabla. 

### <a name="use-the-azure-cli"></a>Uso de la CLI de Azure
En el ejemplo de esta sección se muestra cómo usar la CLI de Azure para agregar una identidad a un rol de Azure. Los comandos de ejemplo son para los temas de Event Grid. Los comandos de los dominios de Event Grid son similares. 

#### <a name="get-the-principal-id-for-the-topics-system-identity"></a>Obtención del identificador de entidad de seguridad de la identidad del sistema del tema 
En primer lugar, obtenga el identificador de entidad de seguridad de la identidad administrada por el sistema del tema y asigne la identidad a los roles adecuados.

```azurecli-interactive
topic_pid=$(az ad sp list --display-name "$<TOPIC NAME>" --query [].objectId -o tsv)
```

#### <a name="create-a-role-assignment-for-event-hubs-at-various-scopes"></a>Creación de una asignación de roles para centros de eventos en varios ámbitos 
En el siguiente ejemplo de la CLI se muestra cómo agregar la identidad de un tema al rol **Emisor de datos de Azure Event Hubs** en el nivel de espacio de nombres o en el nivel de centro de eventos. Si crea la asignación de roles en el nivel de espacio de nombres, el tema puede reenviar eventos a todos los centros de eventos de ese espacio de nombres. Si la crea en el nivel de centro de eventos, el tema solo puede reenviar eventos a ese centro de eventos específico. 


```azurecli-interactive
role="Azure Event Hubs Data Sender" 
namespaceresourceid=$(az eventhubs namespace show -n $<EVENT HUBS NAMESPACE NAME> -g <RESOURCE GROUP of EVENT HUB> --query "{I:id}" -o tsv) 
eventhubresourceid=$(az eventhubs eventhub show -n <EVENT HUB NAME> --namespace-name <EVENT HUBS NAMESPACE NAME> -g <RESOURCE GROUP of EVENT HUB> --query "{I:id}" -o tsv) 

# create role assignment for the whole namespace 
az role assignment create --role "$role" --assignee "$topic_pid" --scope "$namespaceresourceid" 

# create role assignment scoped to just one event hub inside the namespace 
az role assignment create --role "$role" --assignee "$topic_pid" --scope "$eventhubresourceid" 
```

#### <a name="create-a-role-assignment-for-a-service-bus-topic-at-various-scopes"></a>Creación de una asignación de roles de un tema de Service Bus en varios ámbitos 
En el siguiente ejemplo de CLI se muestra cómo agregar la identidad de un tema al rol **Emisor de datos de Azure Event Hubs** en el nivel de espacio de nombres o en el nivel de tema de Service Bus. Si crea la asignación de roles en el nivel de espacio de nombres, el tema de Event Grid puede reenviar eventos a todas las entidades (colas o temas de Service Bus ) dentro de ese espacio de nombres. Si la crea en el nivel de cola o tema de Service Bus, el tema de Event Grid solo puede reenviar eventos a esa cola o tema de Service Bus específica. 

```azurecli-interactive
role="Azure Service Bus Data Sender" 
namespaceresourceid=$(az servicebus namespace show -n $RG\SB -g "$RG" --query "{I:id}" -o tsv 
sbustopicresourceid=$(az servicebus topic show -n topic1 --namespace-name $RG\SB -g "$RG" --query "{I:id}" -o tsv) 

# create role assignment for the whole namespace 
az role assignment create --role "$role" --assignee "$topic_pid" --scope "$namespaceresourceid" 

# create role assignment scoped to just one hub inside the namespace 
az role assignment create --role "$role" --assignee "$topic_pid" --scope "$sbustopicresourceid" 
```

## <a name="create-event-subscriptions-that-use-an-identity"></a>Creación de suscripciones de eventos que usan una identidad
Una vez que tenga un tema o un dominio con una identidad administrada por el sistema y haya agregado la identidad al rol adecuado en el destino, estará listo para crear suscripciones que usen la identidad. 

### <a name="use-the-azure-portal"></a>Uso de Azure Portal
Al crear una suscripción de eventos, se muestra una opción para habilitar el uso de una identidad asignada por el sistema para un punto de conexión en la sección **DETALLES DE PUNTO DE CONEXIÓN**. 

![Habilitación de una identidad al crear una suscripción de eventos de una cola de Service Bus](./media/managed-service-identity/service-bus-queue-subscription-identity.png)

También puede habilitar el uso de una identidad asignada por el sistema para que se use en la cola de mensajes fallidos en la pestaña **Características adicionales**. 

![Habilitación de la identidad asignada por el sistema para la cola de mensajes fallidos](./media/managed-service-identity/enable-deadletter-identity.png)

### <a name="use-the-azure-cli---service-bus-queue"></a>Uso de la CLI de Azure: cola de Service Bus 
En esta sección, aprenderá a usar la CLI de Azure para habilitar el uso de una identidad asignada por el sistema para enviar eventos a una cola de Service Bus. La identidad debe ser miembro del rol **Emisor de datos de Azure Service Bus**. También debe ser miembro del rol **Colaborador de datos de blobs de almacenamiento** en la cuenta de almacenamiento que se usa para la cola de mensajes fallidos. 

#### <a name="define-variables"></a>Definición de variables
En primer lugar, especifique los valores de las siguientes variables que se usarán en el comando de la CLI. 

```azurecli-interactive
subid="<AZURE SUBSCRIPTION ID>"
rg = "<RESOURCE GROUP of EVENT GRID TOPIC>"
topicname = "<EVENT GRID TOPIC NAME>"

# get the service bus queue resource id
queueid=$(az servicebus queue show --namespace-name <SERVICE BUS NAMESPACE NAME> --name <QUEUE NAME> --resource-group <RESOURCE GROUP NAME> --query id --output tsv)
sb_esname = "<Specify a name for the event subscription>" 
```

#### <a name="create-an-event-subscription-by-using-a-managed-identity-for-delivery"></a>Creación de una suscripción de eventos usando una identidad administrada para la entrega 
Este comando de ejemplo crea una suscripción de eventos de un tema de Event Grid con un tipo de punto de conexión establecido en **Cola de Service Bus**. 

```azurecli-interactive
az eventgrid event-subscription create  
    --source-resource-id /subscriptions/$subid/resourceGroups/$rg/providers/Microsoft.EventGrid/topics/$topicname
    --delivery-identity-endpoint-type servicebusqueue  
    --delivery-identity systemassigned 
    --delivery-identity-endpoint $queueid
    -n $sb_esname 
```

#### <a name="create-an-event-subscription-by-using-a-managed-identity-for-delivery-and-dead-lettering"></a>Creación de una suscripción de eventos usando una identidad administrada para la entrega y cola de mensajes fallidos
Este comando de ejemplo crea una suscripción de eventos de un tema de Event Grid con un tipo de punto de conexión establecido en **Cola de Service Bus**. También especifica que la identidad administrada por el sistema se va a usar para la cola de mensajes fallidos. 

```azurecli-interactive
storageid=$(az storage account show --name demoStorage --resource-group gridResourceGroup --query id --output tsv)
deadletterendpoint="$storageid/blobServices/default/containers/<BLOB CONTAINER NAME>"

az eventgrid event-subscription create  
    --source-resource-id /subscriptions/$subid/resourceGroups/$rg/providers/Microsoft.EventGrid/topics/$topicname 
    --delivery-identity-endpoint-type servicebusqueue
    --delivery-identity systemassigned 
    --delivery-identity-endpoint $queueid
    --deadletter-identity-endpoint $deadletterendpoint 
    --deadletter-identity systemassigned 
    -n $sb_esnameq 
```

### <a name="use-the-azure-cli---event-hubs"></a>Uso de la CLI de Azure: Event Hubs 
En esta sección, aprenderá a usar la CLI de Azure para habilitar el uso de una identidad asignada por el sistema para enviar eventos a un centro de eventos. La identidad debe ser miembro del rol **Emisor de datos de Azure Event Hubs**. También debe ser miembro del rol **Colaborador de datos de blobs de almacenamiento** en la cuenta de almacenamiento que se usa para la cola de mensajes fallidos. 

#### <a name="define-variables"></a>Definición de variables
```azurecli-interactive
subid="<AZURE SUBSCRIPTION ID>"
rg = "<RESOURCE GROUP of EVENT GRID TOPIC>"
topicname = "<EVENT GRID TOPIC NAME>"

hubid=$(az eventhubs eventhub show --name <EVENT HUB NAME> --namespace-name <NAMESPACE NAME> --resource-group <RESOURCE GROUP NAME> --query id --output tsv)
eh_esname = "<SPECIFY EVENT SUBSCRIPTION NAME>" 
```

#### <a name="create-an-event-subscription-by-using-a-managed-identity-for-delivery"></a>Creación de una suscripción de eventos usando una identidad administrada para la entrega 
Este comando de ejemplo crea una suscripción de eventos de un tema de Event Grid con un tipo de punto de conexión establecido en **Event Hubs**. 

```azurecli-interactive
az eventgrid event-subscription create  
    --source-resource-id /subscriptions/$subid/resourceGroups/$rg/providers/Microsoft.EventGrid/topics/$topicname 
    --delivery-identity-endpoint-type eventhub 
    --delivery-identity systemassigned 
    --delivery-identity-endpoint $hubid
    -n $sbq_esname 
```

#### <a name="create-an-event-subscription-by-using-a-managed-identity-for-delivery--deadletter"></a>Creación de una suscripción de eventos usando una identidad administrada para la entrega y cola de mensajes fallidos 
Este comando de ejemplo crea una suscripción de eventos de un tema de Event Grid con un tipo de punto de conexión establecido en **Event Hubs**. También especifica que la identidad administrada por el sistema se va a usar para la cola de mensajes fallidos. 

```azurecli-interactive
storageid=$(az storage account show --name demoStorage --resource-group gridResourceGroup --query id --output tsv)
deadletterendpoint="$storageid/blobServices/default/containers/<BLOB CONTAINER NAME>"

az eventgrid event-subscription create
    --source-resource-id /subscriptions/$subid/resourceGroups/$rg/providers/Microsoft.EventGrid/topics/$topicname 
    --delivery-identity-endpoint-type servicebusqueue  
    --delivery-identity systemassigned 
    --delivery-identity-endpoint $hubid
    --deadletter-identity-endpoint $eh_deadletterendpoint
    --deadletter-identity systemassigned 
    -n $eh_esname 
```

### <a name="use-the-azure-cli---azure-storage-queue"></a>Uso de la CLI de Azure: cola de Azure Storage 
En esta sección, aprenderá a usar la CLI de Azure para habilitar el uso de una identidad asignada por el sistema para enviar eventos a una cola de Azure Storage. La identidad debe ser miembro del rol **Colaborador de datos de blobs de almacenamiento** de la cuenta de almacenamiento.

#### <a name="define-variables"></a>Definición de variables  

```azurecli-interactive
subid="<AZURE SUBSCRIPTION ID>"
rg = "<RESOURCE GROUP of EVENT GRID TOPIC>"
topicname = "<EVENT GRID TOPIC NAME>"

# get the storage account resource id
storageid=$(az storage account show --name <STORAGE ACCOUNT NAME> --resource-group <RESOURCE GROUP NAME> --query id --output tsv)

# build the resource id for the queue
queueid="$storageid/queueservices/default/queues/<QUEUE NAME>" 

sa_esname = "<SPECIFY EVENT SUBSCRIPTION NAME>" 
```

#### <a name="create-an-event-subscription-by-using-a-managed-identity-for-delivery"></a>Creación de una suscripción de eventos usando una identidad administrada para la entrega 

```azurecli-interactive
az eventgrid event-subscription create 
    --source-resource-id /subscriptions/$subid/resourceGroups/$rg/providers/Microsoft.EventGrid/topics/$topicname 
    --delivery-identity-endpoint-type storagequeue  
    --delivery-identity systemassigned 
    --delivery-identity-endpoint $queueid
    -n $sa_esname 
```

#### <a name="create-an-event-subscription-by-using-a-managed-identity-for-delivery--deadletter"></a>Creación de una suscripción de eventos usando una identidad administrada para la entrega y cola de mensajes fallidos 

```azurecli-interactive
storageid=$(az storage account show --name demoStorage --resource-group gridResourceGroup --query id --output tsv)
deadletterendpoint="$storageid/blobServices/default/containers/<BLOB CONTAINER NAME>"

az eventgrid event-subscription create  
    --source-resource-id /subscriptions/$subid/resourceGroups/$rg/providers/Microsoft.EventGrid/topics/$topicname 
    --delivery-identity-endpoint-type storagequeue  
    --delivery-identity systemassigned 
    --delivery-identity-endpoint $queueid
    --deadletter-identity-endpoint $deadletterendpoint 
    --deadletter-identity systemassigned 
    -n $sa_esname 
```



## <a name="next-steps"></a>Pasos siguientes
Para más información sobre las identidades de servicio administradas, consulte el artículo sobre [Qué son las identidades administradas para recursos de Azure](../active-directory/managed-identities-azure-resources/overview.md). 
