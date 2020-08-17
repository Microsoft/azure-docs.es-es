---
title: Uso de colas de azure/service-bus en Node.js
description: Aprenda a escribir un programa de Nodejs para enviar mensajes a una cola de Service Bus, y recibirlos de ella, con el nuevo paquete @azure/service-bus.
author: spelluru
ms.devlang: nodejs
ms.topic: quickstart
ms.date: 06/23/2020
ms.author: spelluru
ms.custom: devx-track-javascript
ms.openlocfilehash: c082eff4f70b0255b7701340ac6e8b377223c0b5
ms.sourcegitcommit: 2ffa5bae1545c660d6f3b62f31c4efa69c1e957f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/11/2020
ms.locfileid: "88076870"
---
# <a name="quickstart-how-to-use-service-bus-queues-with-nodejs-and-the-azureservice-bus-package"></a>Inicio rápido: Uso de colas de Service Bus con Node.js y el paquete azure/service-bus
En este tutorial, aprenderá a escribir un programa de Nodejs para enviar mensajes a una cola de Service Bus, y recibirlos de ella, con el nuevo paquete [@azure/service-bus](https://www.npmjs.com/package/@azure/service-bus). En este paquete se usa el [protocolo AMQP 1.0](service-bus-amqp-overview.md) (más rápido), mientras que en el paquete [azure-sb](https://www.npmjs.com/package/azure-sb) anterior se utilizaban las [API de tiempo de ejecución REST de Service Bus](/rest/api/servicebus/service-bus-runtime-rest). Los ejemplos están escritos en JavaScript.

## <a name="prerequisites"></a>Prerrequisitos
- Suscripción a Azure. Para completar este tutorial, deberá tener una cuenta de Azure. Puede activar sus [ventajas de suscriptor a MSDN](https://azure.microsoft.com/pricing/member-offers/credit-for-visual-studio-subscribers/?WT.mc_id=A85619ABF) o registrarse para obtener una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A85619ABF).
- Si no tiene una cola con la que trabajar, siga los pasos del artículo [Uso de Azure Portal para crear una cola de Service Bus](service-bus-quickstart-portal.md) para crear una. Anote la cadena de conexión para la instancia de Service Bus y el nombre de la cola que creó. Utilizaremos estos valores en los ejemplos.

> [!NOTE]
> - En este tutorial se trabaja con ejemplos que puede copiar y ejecutar con [Nodejs](https://nodejs.org/). Para obtener instrucciones acerca de cómo crear una aplicación de Node.js, consulte [Creación e implementación de una aplicación Node.js en un sitio web de Azure](../app-service/quickstart-nodejs.md) o [Servicio en la nube Node.js (con Windows PowerShell)](../cloud-services/cloud-services-nodejs-develop-deploy-app.md).
> - El nuevo paquete [@azure/service-bus](https://www.npmjs.com/package/@azure/service-bus) no admite aún la creación de colas. Use el paquete [@azure/arm-servicebus](https://www.npmjs.com/package/@azure/arm-servicebus) si quiere crearlos mediante programación.

### <a name="use-node-package-manager-npm-to-install-the-package"></a>Uso del Administrador de paquetes para Node (NPM) para instalar el paquete
Para instalar el paquete de npm para Service Bus, abra un símbolo del sistema que tenga `npm` en la ruta de acceso, cambie el directorio a la carpeta en la que quiere colocar los ejemplos y ejecute este comando.

```bash
npm install @azure/service-bus
```

## <a name="send-messages-to-a-queue"></a>mensajes a una cola
La interactuación con una cola de Service Bus empieza por la creación de una instancia de la clase [ServiceBusClient](/javascript/api/@azure/service-bus/servicebusclient) y su uso para crear una instancia de la clase [QueueClient](/javascript/api/@azure/service-bus/queueclient). Una vez que tenga el cliente de cola, puede crear un emisor y usar los métodos [send](/javascript/api/@azure/service-bus/sender#send-sendablemessageinfo-) o [sendBatch](/javascript/api/@azure/service-bus/sender#sendbatch-sendablemessageinfo---) en él para enviar mensajes.

1. Abra el editor que prefiera, como [Visual Studio Code](https://code.visualstudio.com/)
2. Cree un archivo denominado `send.js` y pegue en él el código siguiente. Este código enviará 10 mensajes a la cola.

    ```javascript
    const { ServiceBusClient } = require("@azure/service-bus"); 
    
    // Define connection string and related Service Bus entity names here
    const connectionString = "";
    const queueName = ""; 
    
    async function main(){
      const sbClient = ServiceBusClient.createFromConnectionString(connectionString); 
      const queueClient = sbClient.createQueueClient(queueName);
      const sender = queueClient.createSender();
      
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
        
        await queueClient.close();
      } finally {
        await sbClient.close();
      }
    }
    
    main().catch((err) => {
      console.log("Error occurred: ", err);
    });
    ```
3. Escriba la cadena de conexión y el nombre de la cola en el código anterior.
4. Después, ejecute el comando `node send.js` en un símbolo del sistema para ejecutar este archivo.

Felicidades. Acaba de enviar mensajes a una cola de Service Bus.

Los mensajes tienen algunas propiedades estándar, como `label` y `messageId`, que se pueden establecer cuando se envían. Si desea establecer alguna propiedad personalizada, utilice `userProperties`, que es un objeto JSON que puede contener pares de clave y valor de los datos personalizados.

El tamaño máximo de mensaje que admiten las colas de Service Bus es de 256 KB en el [nivel Estándar](service-bus-premium-messaging.md) y de 1 MB en el [nivel Premium](service-bus-premium-messaging.md). No hay límite para el número de mensajes que contiene una cola, pero hay un tope para el tamaño total de los mensajes contenidos en una cola. El tamaño de la cola se define en el momento de la creación, con un límite de 5 GB. Para obtener más información sobre las cuotas, consulte [Cuotas de Service Bus](service-bus-quotas.md).

## <a name="receive-messages-from-a-queue"></a>mensajes de una cola
La interactuación con una cola de Service Bus empieza por la creación de una instancia de la clase [ServiceBusClient](/javascript/api/@azure/service-bus/servicebusclient) y su uso para crear una instancia de la clase [QueueClient](/javascript/api/@azure/service-bus/queueclient). Una vez que tenga el cliente de cola, puede crear un receptor y usar los métodos [receiveMessages](/javascript/api/@azure/service-bus/receiver#receivemessages-number--undefined---number-) o [registerMessageHandler](/javascript/api/@azure/service-bus/receiver#registermessagehandler-onmessage--onerror--messagehandleroptions-) en él para recibir mensajes.

1. Abra el editor que prefiera, como [Visual Studio Code](https://code.visualstudio.com/)
2. Cree un archivo denominado `recieve.js` y pegue en él el código siguiente. Este código intentará recibir 10 mensajes de la cola. El recuento real que recibe depende del número de mensajes en la cola y la latencia de red.

    ```javascript
    const { ServiceBusClient, ReceiveMode } = require("@azure/service-bus"); 
    
    // Define connection string and related Service Bus entity names here
    const connectionString = "";
    const queueName = ""; 
    
    async function main(){
      const sbClient = ServiceBusClient.createFromConnectionString(connectionString); 
      const queueClient = sbClient.createQueueClient(queueName);
      const receiver = queueClient.createReceiver(ReceiveMode.receiveAndDelete);
      try {
        const messages = await receiver.receiveMessages(10)
        console.log("Received messages:");
        console.log(messages.map(message => message.body));
         
        await queueClient.close();
      } finally {
        await sbClient.close();
      }
    }
    
    main().catch((err) => {
      console.log("Error occurred: ", err);
    });
    ```
3. Escriba la cadena de conexión y el nombre de la cola en el código anterior.
4. Después, ejecute el comando `node receiveMessages.js` en un símbolo del sistema para ejecutar este archivo.

Felicidades. Acaba de recibir mensajes de una cola de Service Bus.

El método [createReceiver](/javascript/api/@azure/service-bus/queueclient#createreceiver-receivemode-) toma un elemento `ReceiveMode` que es una enumeración con los valores [ReceiveAndDelete](message-transfers-locks-settlement.md#settling-receive-operations) y [PeekLock](message-transfers-locks-settlement.md#settling-receive-operations). No olvide [resolver los mensajes](message-transfers-locks-settlement.md#settling-receive-operations) si usa el modo `PeekLock` mediante el uso de cualquiera de los métodos `complete()`, `abandon()`, `defer()` o `deadletter()` en el mensaje.

> [!NOTE]
> Puede administrar los recursos de Service Bus con el [Explorador de Service Bus](https://github.com/paolosalvatori/ServiceBusExplorer/). El Explorador de Service Bus permite a los usuarios conectarse a un espacio de nombres de Service Bus y administrar las entidades de mensajería de una forma sencilla. La herramienta dispone de características avanzadas, como la funcionalidad de importación y exportación o la capacidad de probar el tema, las colas, las suscripciones, los servicios de retransmisión, los centros de notificaciones y los centros de eventos. 

## <a name="next-steps"></a>Pasos siguientes
Para más información, consulte estos recursos.
- [Colas, temas y suscripciones](service-bus-queues-topics-subscriptions.md)
- Consulte otros [ejemplos de Node.js para Service Bus en GitHub](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/servicebus/service-bus/samples/javascript)
- [Centro para desarrolladores de Node.js](https://azure.microsoft.com/develop/nodejs/)