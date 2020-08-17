---
title: 'Tutorial: Actualización del surtido de inventario comercial mediante canales de publicación y suscripción, y filtros de tema con la CLI de Azure'
description: 'Tutorial: En este tutorial, aprenderá a enviar y recibir mensajes de un tema y una suscripción, y a agregar y usar reglas de filtro mediante la CLI de Azure'
ms.date: 06/23/2020
ms.topic: tutorial
author: spelluru
ms.author: spelluru
ms.custom: devx-track-azurecli
ms.openlocfilehash: 2526559a8b88309c098e59e8cc6d0ffd2793984f
ms.sourcegitcommit: d8b8768d62672e9c287a04f2578383d0eb857950
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/11/2020
ms.locfileid: "88067620"
---
# <a name="tutorial-update-inventory-using-cli-and-topicssubscriptions"></a>Tutorial: Actualización del inventario mediante la CLI y temas o suscripciones

Microsoft Azure Service Bus es un servicio de mensajería multiinquilino en la nube que envía información entre aplicaciones y servicios. Las operaciones asincrónicas le ofrecen una mensajería flexible y asincrónica, además de funcionalidades de mensajería estructurada de tipo FIFO (primero en entrar, primero en salir) y de publicación y suscripción. En este tutorial se muestra cómo usar los temas y las suscripciones de Service Bus en un escenario de inventario comercial con canales de publicación y suscripción mediante la CLI de Azure y Java.

En este tutorial, aprenderá a:
> [!div class="checklist"]
> * Crear un tema de Service Bus y una o varias suscripciones a ese tema mediante la CLI de Azure
> * Agregar filtros de tema mediante la CLI de Azure
> * Crear dos mensajes con contenido diferente
> * Enviar los mensajes y comprobar que llegaron a las suscripciones esperadas
> * Recibir mensajes de las suscripciones

Un ejemplo de este escenario es una actualización del surtido de inventario de varias tiendas. En este escenario, cada tienda o conjunto de tiendas, recibe mensajes dirigidos a que actualicen sus inventarios. En este tutorial se muestra cómo implementar este escenario mediante suscripciones y filtros. En primer lugar puede crear un tema con 3 suscripciones, agregar algunas reglas y filtros y, a continuación, enviar y recibir mensajes de los temas y las suscripciones.

![topic](./media/service-bus-tutorial-topics-subscriptions-cli/about-service-bus-topic.png)

Si no tiene una suscripción a Azure, puede crear una [cuenta gratuita][] antes de empezar.

## <a name="prerequisites"></a>Prerrequisitos

Para desarrollar una aplicación de Service Bus con Java, debe tener instalado lo siguiente:

- [Kit de desarrollo de Java](https://aka.ms/azure-jdks), la versión más reciente.
- [CLI de Azure](/cli/azure)
- [Apache Maven](https://maven.apache.org), versión 3.0 o posterior.

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

Si decide instalar y usar la CLI localmente, para este tutorial es preciso que ejecute la CLI de Azure versión 2.0.4 o posterior. Ejecute `az --version` para encontrar la versión. Si necesita instalarla o actualizarla, consulte [Instalación de la CLI de Azure]( /cli/azure/install-azure-cli).

## <a name="service-bus-topics-and-subscriptions"></a>Temas y suscripciones de Service Bus

Cada [suscripción a un tema](service-bus-messaging-overview.md#topics) puede recibir una copia de cada mensaje. Los temas son completamente compatibles desde el punto de vista semántico y del protocolo con las colas de Service Bus. Los temas de Service Bus admiten una amplia variedad de reglas de selección con condiciones de filtro, con acciones opcionales que establecerán o modificarán las propiedades del mensaje. Cada vez que coincide una regla, esta genera un mensaje. Para más información acerca de reglas, filtros y acciones, siga este [vínculo](topic-filters.md).

## <a name="sign-in-to-azure"></a>Inicio de sesión en Azure

Una vez instalada la CLI, abra un símbolo del sistema y use los siguientes comandos para iniciar sesión en Azure. Estos pasos no son necesarios si usa Cloud Shell:

1. Si usa la CLI de Azure localmente, ejecute el siguiente comando para iniciar sesión en Azure. Este paso de inicio de sesión no es necesario si ejecuta estos comandos en Cloud Shell:

   ```azurecli-interactive
   az login
   ```

2. Establezca el contexto actual de suscripción en la suscripción de Azure que desea usar:

   ```azurecli-interactive
   az account set --subscription Azure_subscription_name
   ```

## <a name="use-cli-to-provision-resources"></a>Uso de la CLI para aprovisionar los recursos

Emita los siguientes comandos para aprovisionar recursos de Service Bus. Asegúrese de reemplazar todos los marcadores de posición por los valores apropiados:

```azurecli-interactive
# Create a resource group
az group create --name myResourcegroup --location eastus

# Create a Service Bus messaging namespace with a unique name
namespaceName=myNameSpace$RANDOM
az servicebus namespace create \
   --resource-group myResourceGroup \
   --name $namespaceName \
   --location eastus

# Create a Service Bus topic
az servicebus topic create --resource-group myResourceGroup \
   --namespace-name $namespaceName \
   --name myTopic

# Create subscription 1 to the topic
az servicebus subscription create --resource-group myResourceGroup --namespace-name $namespaceName --topic-name myTopic --name S1

# Create filter 1 - use custom properties
az servicebus rule create --resource-group myResourceGroup --namespace-name $namespaceName --topic-name myTopic --subscription-name S1 --name MyFilter --filter-sql-expression "StoreId IN ('Store1','Store2','Store3')"

# Create filter 2 - use custom properties
az servicebus rule create --resource-group myResourceGroup --namespace-name $namespaceName --topic-name myTopic --subscription-name S1 --name MySecondFilter --filter-sql-expression "StoreId = 'Store4'"

# Create subscription 2
az servicebus subscription create --resource-group myResourceGroup --namespace-name $namespaceName --topic-name myTopic --name S2

# Create filter 3 - use message header properties via IN list and 
# combine with custom properties.
az servicebus rule create --resource-group myResourceGroup --namespace-name $namespaceName --topic-name myTopic --subscription-name S2 --name MyFilter --filter-sql-expression "sys.To IN ('Store5','Store6','Store7') OR StoreId = 'Store8'"

# Create subscription 3
az servicebus subscription create --resource-group myResourceGroup --namespace-name $namespaceName --topic-name myTopic --name S3

# Create filter 4 - Get everything except messages for subscription 1 and 2. 
# Also modify and add an action; in this case set the label to a specified value. 
# Assume those stores might not be part of your main store, so you only add 
# specific items to them. For that, you flag them specifically.
az servicebus rule create --resource-group DemoGroup --namespace-name DemoNamespaceSB --topic-name tutorialtest1
 --subscription-name S3 --name MyFilter --filter-sql-expression "sys.To NOT IN ('Store1','Store2','Store3','Store4','Sto
re5','Store6','Store7','Store8') OR StoreId NOT IN ('Store1','Store2','Store3','Store4','Store5','Store6','Store7','Stor
e8')" --action-sql-expression "SET sys.Label = 'SalesEvent'"

# Get the connection string
connectionString=$(az servicebus namespace authorization-rule keys list \
   --resource-group myResourceGroup \
   --namespace-name  $namespaceName \
   --name RootManageSharedAccessKey \
   --query primaryConnectionString --output tsv)
```

Después de que se ejecute el último comando, copie y pegue la cadena de conexión y el nombre de cola que ha seleccionado en una ubicación temporal como, por ejemplo, el Bloc de notas. Esta información la necesitará en el siguiente paso.

## <a name="create-filter-rules-on-subscriptions"></a>Creación de reglas de filtro en suscripciones

Después de aprovisionar el espacio de nombres y los temas y suscripciones, y de tener las credenciales necesarias, está listo para crear reglas de filtro en las suscripciones y, posteriormente, enviar y recibir mensajes. Puede examinar el código en [esta carpeta de ejemplos de GitHub](https://github.com/Azure/azure-service-bus/tree/master/samples/Java/azure-servicebus/TopicFilters).

## <a name="send-and-receive-messages"></a>Envío y recepción de mensajes

1. Asegúrese de que Cloud Shell está abierto y muestra el símbolo del sistema de Bash.

2. Clone el [repositorio de GitHub de Service Bus](https://github.com/Azure/azure-service-bus/) mediante la emisión del comando siguiente:

   ```shell
   git clone https://github.com/Azure/azure-service-bus.git
   ```

2. Vaya a la carpeta de ejemplos `azure-service-bus/samples/Java/quickstarts-and-tutorials/quickstart-java/tutorial-topics-subscriptions-filters-java`. Tenga en cuenta que en el shell de Bash, los comandos distinguen mayúsculas de minúsculas y que los separadores de ruta de acceso deben ser barras diagonales.

3. Emita el siguiente comando para compilar la aplicación:
   
   ```shell
   mvn clean package -DskipTests
   ```
4. Para ejecutar el programa, emita el siguiente comando. Asegúrese de reemplazar los marcadores de posición por el nombre del tema y la cadena de conexión que obtuvo en el paso anterior:

   ```shell
   java -jar .\target\tutorial-topics-subscriptions-filters-1.0.0-jar-with-dependencies.jar -c "myConnectionString" -t "myTopicName"
   ```

   Observe que se envían 10 mensajes al tema y que, posteriormente, se reciben de las suscripciones individuales:

   ![salida del programa](./media/service-bus-tutorial-topics-subscriptions-cli/service-bus-tutorial-topics-subscriptions-cli.png)

## <a name="clean-up-resources"></a>Limpieza de recursos

Ejecute el siguiente comando para quitar el grupo de recursos, el espacio de nombres y todos los recursos relacionados:

```azurecli-interactive
az group delete --resource-group my-resourcegroup
```

## <a name="understand-the-sample-code"></a>Descripción del código de ejemplo

Esta sección contiene información más detallada acerca de lo que hace el código de ejemplo.

### <a name="get-connection-string-and-queue"></a>Obtención de la cadena de conexión y la cola

En primer lugar, el código declara un conjunto de variables que dirige la ejecución del resto del programa:

```java
    public String ConnectionString = null;
    public String TopicName = null;
    static final String[] Subscriptions = {"S1","S2","S3"};
    static final String[] Store = {"Store1","Store2","Store3","Store4","Store5","Store6","Store7","Store8","Store9","Store10"};
    static final String SysField = "sys.To";
    static final String CustomField = "StoreId";
    int NrOfMessagesPerStore = 1; // Send at least 1.
```

La cadena de conexión y el nombre del tema son los únicos valores agregados mediante parámetros de línea de comandos, y pasados a `main()`. La ejecución del código real se desencadena en el método `run()`, que envía y luego recibe mensajes del tema:

```java
public static void main(String[] args) {
    TutorialTopicsSubscriptionsFilters app = new TutorialTopicsSubscriptionsFilters();
        try {
            app.runApp(args);
            app.run();
        } catch (Exception e) {
            System.out.printf("%s", e.toString());
        }
        System.exit(0);
    }
}

public void run() throws Exception {
    // Send sample messages.
    this.sendMessagesToTopic();

    // Receive messages from subscriptions.
    this.receiveAllMessages();
}
```

### <a name="create-topic-client-to-send-messages"></a>Creación del cliente de temas para enviar mensajes

Para enviar y recibir mensajes, el método `sendMessagesToTopic()` crea una instancia de cliente de temas, que usa la cadena de conexión y el nombre del tema y luego llama a otro método que envía los mensajes:

```java
public void sendMessagesToTopic() throws Exception, ServiceBusException {
    // Create client for the topic.
    TopicClient topicClient = new TopicClient(new ConnectionStringBuilder(ConnectionString, TopicName));

    // Create a message sender from the topic client.

    System.out.printf("\nSending orders to topic.\n");

    // Now we can start sending orders.
    CompletableFuture.allOf(
            SendOrders(topicClient,Store[0]),
            SendOrders(topicClient,Store[1]),
            SendOrders(topicClient,Store[2]),
            SendOrders(topicClient,Store[3]),
            SendOrders(topicClient,Store[4]),
            SendOrders(topicClient,Store[5]),
            SendOrders(topicClient,Store[6]),
            SendOrders(topicClient,Store[7]),
            SendOrders(topicClient,Store[8]),
            SendOrders(topicClient,Store[9])
    ).join();

    System.out.printf("\nAll messages sent.\n");
}

    public CompletableFuture<Void> SendOrders(TopicClient topicClient, String store) throws Exception {

        for(int i = 0;i<NrOfMessagesPerStore;i++) {
            Random r = new Random();
            final Item item = new Item(r.nextInt(5),r.nextInt(5),r.nextInt(5));
            IMessage message = new Message(GSON.toJson(item,Item.class).getBytes(UTF_8));
            // We always set the Sent to field
            message.setTo(store);
            final String StoreId = store;
            Double priceToString = item.getPrice();
            final String priceForPut = priceToString.toString();
            message.setProperties(new HashMap<String, String>() {{
                // Additionally we add a customer store field. In reality you would use sys.To or a customer property but not both.
                // This is just for demo purposes.
                put("StoreId", StoreId);
                // Adding more potential filter / rule and action able fields
                put("Price", priceForPut);
                put("Color", item.getColor());
                put("Category", item.getItemCategory());
            }});

            System.out.printf("Sent order to Store %s. Price=%f, Color=%s, Category=%s\n", StoreId, item.getPrice(), item.getColor(), item.getItemCategory());
            topicClient.sendAsync(message);
        }

        return new CompletableFuture().completedFuture(null);
    }
```

### <a name="receive-messages-from-the-individual-subscriptions"></a>Recepción de mensajes de las suscripciones individuales

El método `receiveAllMessages()` llama al método `receiveAllMessageFromSubscription()`, que crea entonces un cliente de suscripción por llamada y recibe mensajes de las suscripciones individuales:

```java
public void receiveAllMessages() throws Exception {
    System.out.printf("\nStart Receiving Messages.\n");

    CompletableFuture.allOf(
            receiveAllMessageFromSubscription(Subscriptions[0]),
            receiveAllMessageFromSubscription(Subscriptions[1]),
            receiveAllMessageFromSubscription(Subscriptions[2])
            ).join();
}

public CompletableFuture<Void> receiveAllMessageFromSubscription(String subscription) throws Exception {

    int receivedMessages = 0;

    // Create subscription client.
    IMessageReceiver subscriptionClient = ClientFactory.createMessageReceiverFromConnectionStringBuilder(new ConnectionStringBuilder(ConnectionString, TopicName+"/subscriptions/"+ subscription), ReceiveMode.PEEKLOCK);

    // Create a receiver from the subscription client and receive all messages.
    System.out.printf("\nReceiving messages from subscription %s.\n\n", subscription);

    while (true)
    {
        // This will make the connection wait for N seconds if new messages are available.
        // If no additional messages come we close the connection. This can also be used to realize long polling.
        // In case of long polling you would obviously set it more to e.g. 60 seconds.
        IMessage receivedMessage = subscriptionClient.receive(Duration.ofSeconds(1));
        if (receivedMessage != null)
        {
            if ( receivedMessage.getProperties() != null ) {
                System.out.printf("StoreId=%s\n", receivedMessage.getProperties().get("StoreId"));
                
                // Show the label modified by the rule action
                if(receivedMessage.getLabel() != null)
                    System.out.printf("Label=%s\n", receivedMessage.getLabel());
            }
            
            byte[] body = receivedMessage.getBody();
            Item theItem = GSON.fromJson(new String(body, UTF_8), Item.class);
            System.out.printf("Item data. Price=%f, Color=%s, Category=%s\n", theItem.getPrice(), theItem.getColor(), theItem.getItemCategory());
            
            subscriptionClient.complete(receivedMessage.getLockToken());
            receivedMessages++;
        }
        else
        {
            // No more messages to receive.
            subscriptionClient.close();
            break;
        }
    }
    System.out.printf("\nReceived %s messages from subscription %s.\n", receivedMessages, subscription);
    
    return new CompletableFuture().completedFuture(null);
}
```

> [!NOTE]
> Puede administrar los recursos de Service Bus con el [Explorador de Service Bus](https://github.com/paolosalvatori/ServiceBusExplorer/). El Explorador de Service Bus permite a los usuarios conectarse a un espacio de nombres de Service Bus y administrar las entidades de mensajería de una forma sencilla. La herramienta dispone de características avanzadas, como la funcionalidad de importación y exportación o la capacidad de probar el tema, las colas, las suscripciones, los servicios de retransmisión, los centros de notificaciones y los centros de eventos. 

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, aprovisionó recursos mediante la CLI de Azure, y luego envió y recibió mensajes desde un tema de Service Bus y sus suscripciones. Ha aprendido a:

> [!div class="checklist"]
> * Crear un tema de Service Bus y una o varias suscripciones a ese tema mediante Azure Portal
> * Agregar filtros de tema mediante código .NET
> * Crear dos mensajes con contenido diferente
> * Enviar los mensajes y comprobar que llegaron a las suscripciones esperadas
> * Recibir mensajes de las suscripciones

Para más ejemplos de envío y recepción de mensajes, empiece a trabajar con los [ejemplos de Service Bus en GitHub](https://github.com/Azure/azure-service-bus/tree/master/samples/DotNet/GettingStarted).

Avance hasta el siguiente tutorial para aprender más acerca del uso de las funcionalidades de publicación y suscripción de Service Bus.

> [!div class="nextstepaction"]
> [Actualización del inventario mediante PowerShell y temas/suscripciones](service-bus-tutorial-topics-subscriptions-portal.md)

[cuenta gratuita]: https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio
[fully qualified domain name]: https://wikipedia.org/wiki/Fully_qualified_domain_name
[Install the Azure CLI]: /cli/azure/install-azure-cli
[az group create]: /cli/azure/group#az_group_create