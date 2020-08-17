---
title: Uso de temas y suscripciones de Azure Service Bus con Node.js
description: 'Inicio rápido: Aprenda a usar los temas y las suscripciones de Service Bus en Azure desde una aplicación Node.js.'
author: spelluru
ms.devlang: nodejs
ms.topic: quickstart
ms.date: 06/23/2020
ms.author: spelluru
ms.custom: devx-track-javascript
ms.openlocfilehash: fafdf18a3593f7ec444e9970d99807cbfe0cb13d
ms.sourcegitcommit: 2ffa5bae1545c660d6f3b62f31c4efa69c1e957f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/11/2020
ms.locfileid: "88080633"
---
# <a name="quickstart-how-to-use-service-bus-topics-and-subscriptions-with-nodejs-and-the-azureservice-bus-package"></a>Inicio rápido: Uso de los temas y las suscripciones de Service Bus con Node.js y el paquete azure/services-bus.
En este tutorial, obtendrá información sobre cómo crear un programa de Node.js para enviar mensajes a un tema de Service Bus y recibir mensajes de una suscripción de Service Bus con el paquete [@azure/service-bus](https://www.npmjs.com/package/@azure/service-bus) nuevo. En este paquete se usa el [protocolo AMQP 1.0](service-bus-amqp-overview.md) (más rápido), mientras que en el paquete [azure-sb](https://www.npmjs.com/package/azure-sb) anterior se utilizaban las [API de tiempo de ejecución REST de Service Bus](/rest/api/servicebus/service-bus-runtime-rest). Los ejemplos están escritos en JavaScript.

## <a name="prerequisites"></a>Prerrequisitos
- Suscripción a Azure. Para completar este tutorial, deberá tener una cuenta de Azure. Puede activar sus [ventajas de suscriptor a MSDN](https://azure.microsoft.com/pricing/member-offers/credit-for-visual-studio-subscribers/?WT.mc_id=A85619ABF) o registrarse para obtener una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A85619ABF).
- Si no tiene un tema ni una suscripción con los cuales trabajar, siga los pasos que aparecen en el artículo [Use Azure Portal para crear un tema de Service Bus y suscripciones a dicho tema](service-bus-quickstart-topics-subscriptions-portal.md) para crearlos. Anote la cadena de conexión de la instancia de Service Bus y el nombre del tema y de la suscripción que creó. Utilizaremos estos valores en los ejemplos.

> [!NOTE]
> - En este tutorial se trabaja con ejemplos que puede copiar y ejecutar con [Nodejs](https://nodejs.org/). Para instrucciones sobre cómo crear una aplicación Node.js, consulte [Creación e implementación de una aplicación Node.js en un sitio web de Azure](../app-service/quickstart-nodejs.md) o [Servicio en la nube Node.js (con Windows PowerShell)](../cloud-services/cloud-services-nodejs-develop-deploy-app.md).
> - El paquete [@azure/service-bus](https://www.npmjs.com/package/@azure/service-bus) nuevo todavía no permite crear temas ni suscripciones. Use el paquete [@azure/arm-servicebus](https://www.npmjs.com/package/@azure/arm-servicebus) si quiere crearlos mediante programación.

### <a name="use-node-package-manager-npm-to-install-the-package"></a>Uso del Administrador de paquetes para Node (NPM) para instalar el paquete
Para instalar el paquete de npm para Service Bus, abra un símbolo del sistema que tenga `npm` en la ruta de acceso, cambie el directorio a la carpeta en la que quiere colocar los ejemplos y ejecute este comando.

```bash
npm install @azure/service-bus
```

## <a name="send-messages-to-a-topic"></a>Envío de mensajes a un tema
La interactuación con un tema de Service Bus empieza por la creación de una instancia de la clase [ServiceBusClient](/javascript/api/@azure/service-bus/servicebusclient) y su uso para crear una instancia de la clase [TopicClient](/javascript/api/@azure/service-bus/topicclient). Una vez que tenga el cliente de tema, puede crear un emisor y usar los métodos [send](/javascript/api/@azure/service-bus/sender#send-sendablemessageinfo-) o [sendBatch](/javascript/api/@azure/service-bus/sender#sendbatch-sendablemessageinfo---) en él para enviar mensajes.

1. Abra el editor que prefiera, como [Visual Studio Code](https://code.visualstudio.com/)
2. Cree un archivo denominado `send.js` y pegue en él el código siguiente. Este código enviará 10 mensajes al tema.

    ```javascript
    const { ServiceBusClient } = require("@azure/service-bus"); 
    
    // Define connection string and related Service Bus entity names here
    const connectionString = "";
    const topicName = ""; 
    
    async function main(){
      const sbClient = ServiceBusClient.createFromConnectionString(connectionString); 
      const topicClient = sbClient.createTopicClient(topicName);
      const sender = topicClient.createSender();
      
        try {
            for (let i = 0; i < 10; i++) {
              const message= {
                body: `Hello world! ${i}`,
                label: `test`,
                userProperties: {
                    myCustomPropertyName: `my custom property value ${i}`
                }
              };
              console.log(`Sending message: ${message.body}`);
              await sender.send(message);
            }

            await topicClient.close();
          } finally {
            await sbClient.close();
          }
    }
    
    main().catch((err) => {
      console.log("Error occurred: ", err);
    });
    ```
3. Escriba la cadena de conexión y el nombre del tema en el código anterior.
4. Después, ejecute el comando `node send.js` en un símbolo del sistema para ejecutar este archivo. 

Felicidades. Acaba de enviar mensajes a una cola de Service Bus.

Los mensajes tienen algunas propiedades estándar, como `label` y `messageId`, que se pueden establecer cuando se envían. Si desea establecer alguna propiedad personalizada, utilice `userProperties`, que es un objeto JSON que puede contener pares de clave y valor de los datos personalizados.

El tamaño máximo de mensaje que admiten los temas de Service Bus es de 256 KB en el [nivel Estándar](service-bus-premium-messaging.md) y de 1 MB en el [nivel Premium](service-bus-premium-messaging.md). No hay límite para el número de mensajes que puede contener un tema, pero sí lo hay para el tamaño total de los mensajes de un tema. El tamaño de los temas se define en el momento de la creación (el límite máximo es de 5 GB). Para obtener más información sobre las cuotas, consulte [Cuotas de Service Bus](service-bus-quotas.md).

## <a name="receive-messages-from-a-subscription"></a>Recepción de mensajes de una suscripción
La interactuación con una suscripción de Service Bus empieza por la creación de una instancia de la clase [ServiceBusClient](/javascript/api/@azure/service-bus/servicebusclient) y su uso para crear una instancia de la clase [SubscriptionClient](/javascript/api/@azure/service-bus/subscriptionclient). Una vez que tenga el cliente de suscripción, puede crear un receptor y usar los métodos [receiveMessages](/javascript/api/@azure/service-bus/receiver#receivemessages-number--undefined---number-) o [registerMessageHandler](/javascript/api/@azure/service-bus/receiver#registermessagehandler-onmessage--onerror--messagehandleroptions-) en él para recibir mensajes.

1. Abra el editor que prefiera, como [Visual Studio Code](https://code.visualstudio.com/)
2. Cree un archivo denominado `recieve.js` y pegue en él el código siguiente. Este código intentará recibir 10 mensajes de la suscripción. El recuento real que recibe depende del número de mensajes en la suscripción y la latencia de red.

    ```javascript
    const { ServiceBusClient, ReceiveMode } = require("@azure/service-bus"); 
    
    // Define connection string and related Service Bus entity names here
    const connectionString = "";
    const topicName = ""; 
    const subscriptionName = ""; 
    
    async function main(){
      const sbClient = ServiceBusClient.createFromConnectionString(connectionString); 
      const subscriptionClient = sbClient.createSubscriptionClient(topicName, subscriptionName);
      const receiver = subscriptionClient.createReceiver(ReceiveMode.receiveAndDelete);
      
      try {
        const messages = await receiver.receiveMessages(10);
        console.log("Received messages:");
        console.log(messages.map(message => message.body));
        
        await subscriptionClient.close();
      } finally {
        await sbClient.close();
      }
    }
    
    main().catch((err) => {
      console.log("Error occurred: ", err);
    });
    ```
3. Escriba la cadena de conexión y los nombres del tema y la suscripción del código anterior.
4. Después, ejecute el comando `node receiveMessages.js` en un símbolo del sistema para ejecutar este archivo.

Felicidades. Acaba de recibir mensajes de una suscripción de Service Bus.

El método [createReceiver](/javascript/api/@azure/service-bus/subscriptionclient#createreceiver-receivemode-) toma un elemento `ReceiveMode` que es una enumeración con los valores [ReceiveAndDelete](message-transfers-locks-settlement.md#settling-receive-operations) y [PeekLock](message-transfers-locks-settlement.md#settling-receive-operations). No olvide [resolver los mensajes](message-transfers-locks-settlement.md#settling-receive-operations) si usa el modo `PeekLock` mediante el uso de cualquiera de los métodos `complete()`, `abandon()`, `defer()` o `deadletter()` en el mensaje.

## <a name="subscription-filters-and-actions"></a>Acciones y filtros de suscripción
Service Bus admite [filtros y acciones en las suscripciones](topic-filters.md), lo que le permite filtrar los mensajes entrantes a una suscripción y editar sus propiedades.

Una vez que tenga una instancia de `SubscriptionClient`, puede usar los métodos siguientes para obtener, agregar y quitar reglas en la suscripción a fin de controlar los filtros y las acciones.

- getRules
- addRule
- removeRule

Cada suscripción tiene una regla predeterminada que usa el filtro true para permitir todos los mensajes entrantes. Cuando agregue una regla nueva, recuerde quitar el filtro predeterminado para que funcione el filtro en la regla nueva. Si una suscripción no tiene ninguna regla, no recibirá ningún mensaje.

> [!NOTE]
> Puede administrar los recursos de Service Bus con el [Explorador de Service Bus](https://github.com/paolosalvatori/ServiceBusExplorer/). El Explorador de Service Bus permite a los usuarios conectarse a un espacio de nombres de Service Bus y administrar las entidades de mensajería de una forma sencilla. La herramienta dispone de características avanzadas, como la funcionalidad de importación y exportación o la capacidad de probar el tema, las colas, las suscripciones, los servicios de retransmisión, los centros de notificaciones y los centros de eventos. 

## <a name="next-steps"></a>Pasos siguientes
Para más información, consulte estos recursos.

- [Colas, temas y suscripciones](service-bus-queues-topics-subscriptions.md)
- Consulte otros [ejemplos de Node.js para Service Bus en GitHub](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/servicebus/service-bus/samples/javascript)
- [Centro para desarrolladores de Node.js](https://azure.microsoft.com/develop/nodejs/)