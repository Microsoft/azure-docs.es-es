---
title: Enviar y recibir eventos mediante Node.js - Azure Event Hubs | Microsoft Docs
description: En este artículo se ofrece un tutorial para crear una aplicación de Node.js que envía eventos de Azure Event Hubs.
services: event-hubs
author: spelluru
manager: kamalb
ms.service: event-hubs
ms.workload: core
ms.topic: article
ms.custom: seodec18
ms.date: 04/15/2019
ms.author: spelluru
ms.openlocfilehash: e67be59e0ed78b2080986acb73a33fc87599c9d3
ms.sourcegitcommit: f6c85922b9e70bb83879e52c2aec6307c99a0cac
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2019
ms.locfileid: "65539342"
---
# <a name="send-events-to-or-receive-events-from-azure-event-hubs-using-nodejs"></a>Enviar eventos a o recibir eventos desde Azure Event Hubs con Node.js

Azure Event Hubs es una plataforma de streaming de macrodatos y servicio de ingesta de eventos capaz de recibir y procesar millones de eventos por segundo. Event Hubs puede procesar y almacenar eventos, datos o telemetría generados por dispositivos y software distribuido. Los datos enviados a un centro de eventos se pueden transformar y almacenar con cualquier proveedor de análisis en tiempo real o adaptadores de procesamiento por lotes y almacenamiento. Para más información sobre Event Hubs, consulte [Introducción a Event Hubs](event-hubs-about.md) y [Características de Event Hubs](event-hubs-features.md).

Este tutorial describe cómo crear aplicaciones de Node.js para enviar eventos a o recibir eventos desde un centro de eventos.

> [!NOTE]
> Puede descargar esta guía de inicio rápido como un ejemplo desde [GitHub](https://github.com/Azure/azure-event-hubs-node/tree/master/client), reemplazar las cadenas `EventHubConnectionString` y `EventHubName` por los valores del centro de eventos, y ejecutarlo. También puede seguir los pasos de este tutorial para crear el suyo propio.

## <a name="prerequisites"></a>Requisitos previos

Para completar este tutorial, debe cumplir los siguientes requisitos previos:

- Una cuenta de Azure activa. Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio) antes de empezar.
- Node.js versión 8.x y posteriores. Descargue la versión LTS más reciente en [https://nodejs.org](https://nodejs.org).
- Visual Studio Code (recomendado) o cualquier otro IDE
- **Crear un espacio de nombres de Event Hubs y un centro de eventos**. El primer paso consiste en usar [Azure Portal](https://portal.azure.com) para crear un espacio de nombres de tipo Event Hubs y obtener las credenciales de administración que la aplicación necesita para comunicarse con el centro de eventos. Para crear un espacio de nombres y un centro de eventos, siga el procedimiento de [este artículo](event-hubs-create.md) y después continúe con los pasos siguientes de este tutorial. A continuación, obtenga la cadena de conexión para el espacio de nombres del centro de eventos siguiendo las instrucciones del artículo: [Obtenga la cadena de conexión](event-hubs-get-connection-string.md#get-connection-string-from-the-portal). Utilizará la cadena de conexión más adelante en el tutorial.


### <a name="install-npm-package"></a>Instalar el paquete de npm
Para instalar el [paquete de npm para Event Hubs](https://www.npmjs.com/package/@azure/event-hubs), abra un símbolo del sistema que tiene `npm` en su ruta de acceso, cambie el directorio a la carpeta donde desea tener sus ejemplos y, a continuación, ejecute este comando

```shell
npm install @azure/event-hubs
```

Para instalar el [paquete de npm para Event Processor Host](https://www.npmjs.com/package/@azure/event-processor-host), ejecute el comando siguiente en su lugar

```shell
npm install @azure/event-processor-host
```

## <a name="send-events"></a>Envío de eventos

En esta sección se muestra cómo crear una aplicación de Node.js que envía eventos a un centro de eventos. 

1. Abra el editor que prefiera, como [Visual Studio Code](https://code.visualstudio.com). 
2. Cree un archivo denominado `send.js` y pegue el siguiente código en él.
    ```javascript
    const { EventHubClient } = require("@azure/event-hubs");

    // Define connection string and the name of the Event Hub
    const connectionString = "";
    const eventHubsName = "";

    async function main() {
      const client = EventHubClient.createFromConnectionString(connectionString, eventHubsName);

      for (let i = 0; i < 100; i++) {
        const eventData = {body: `Event ${i}`};
        console.log(`Sending message: ${eventData.body}`);
        await client.send(eventData);
      }

      await client.close();
    }

    main().catch(err => {
      console.log("Error occurred: ", err);
    });
    ```
3. Escriba la cadena de conexión y el nombre del centro de eventos en el código anterior
4. A continuación, ejecute el comando `node send.js` en un símbolo del sistema para ejecutar este archivo. Esto enviará 100 eventos al centro de eventos

Felicidades. Se han enviado eventos a un centro de eventos.


## <a name="receive-events"></a>Recepción de eventos

En esta sección se muestra cómo crear una aplicación de Node.js que recibe eventos de una única partición del grupo de consumidores predeterminado en un centro de eventos. 

1. Abra el editor que prefiera, como [Visual Studio Code](https://code.visualstudio.com). 
2. Cree un archivo denominado `receive.js` y pegue el siguiente código en él.
    ```javascript
    const { EventHubClient, delay } = require("@azure/event-hubs");

    // Define connection string and related Event Hubs entity name here
    const connectionString = "";
    const eventHubsName = "";

    async function main() {
      const client = EventHubClient.createFromConnectionString(connectionString, eventHubsName);
      const allPartitionIds = await client.getPartitionIds();
      const firstPartitionId = allPartitionIds[0];

      const receiveHandler = client.receive(firstPartitionId, eventData => {
        console.log(`Received message: ${eventData.body} from partition ${firstPartitionId}`);
      }, error => {
        console.log('Error when receiving message: ', error)
      });

      // Sleep for a while before stopping the receive operation.
      await delay(15000);
      await receiveHandler.stop();

      await client.close();
    }

    main().catch(err => {
      console.log("Error occurred: ", err);
    });
    ```
3. Escriba la cadena de conexión y el nombre del centro de eventos en el código anterior.
4. A continuación, ejecute el comando `node receive.js` en un símbolo del sistema para ejecutar este archivo. Recibirá los eventos de una de las particiones del grupo de consumidores predeterminado en el centro de eventos

Felicidades. Ahora ha recibido los eventos del centro de eventos.

## <a name="receive-events-using-event-processor-host"></a>Recepción de eventos mediante Event Processor Host

Esta sección muestra cómo recibir eventos desde un centro de eventos mediante el uso de Azure [EventProcessorHost](event-hubs-event-processor-host.md) en una aplicación Node.js. EventProcessorHost (EPH) ayuda a recibir de manera eficaz eventos de un centro de eventos mediante la creación de receptores en todas las particiones del grupo de consumidores de un centro de eventos. Establece un punto de comprobación de los metadatos de los mensajes recibidos a intervalos regulares en una instancia de Azure Storage Blob. Este enfoque le permite seguir recibiendo más adelante mensajes desde donde lo dejó.

1. Abra el editor que prefiera, como [Visual Studio Code](https://code.visualstudio.com). 
2. Cree un archivo denominado `receiveAll.js` y pegue el siguiente código en él.
    ```javascript
    const { EventProcessorHost, delay } = require("@azure/event-processor-host");

    // Define connection string and related Event Hubs entity name here
    const eventHubConnectionString = "";
    const eventHubName = "";
    const storageConnectionString = "";

    async function main() {
      const eph = EventProcessorHost.createFromConnectionString(
        "my-eph",
        storageConnectionString,
        "my-storage-container-name",
        eventHubConnectionString,
        {
          eventHubPath: eventHubName,
          onEphError: (error) => {
            console.log("[%s] Error: %O", error);
          }
        }
      );


      eph.start((context, eventData) => {
        console.log(`Received message: ${eventData.body} from partition ${context.partitionId}`);
      }, error => {
        console.log('Error when receiving message: ', error)
      });

      // Sleep for a while before stopping the receive operation.
      await delay(15000);
      await eph.stop();
    }

    main().catch(err => {
      console.log("Error occurred: ", err);
    });

    ```
3. Escriba la cadena de conexión y el nombre del centro de eventos en el código anterior junto con la cadena de conexión de Azure Blob Storage
4. A continuación, ejecute el comando `node receiveAll.js` en un símbolo del sistema para ejecutar este archivo.

Felicidades. Ahora ha recibido los eventos desde centro de eventos mediante Event Processor Host. Recibirá los eventos de todas las particiones del grupo de consumidores predeterminado en el centro de eventos

## <a name="next-steps"></a>Pasos siguientes
Lea los siguientes artículos:

- [EventProcessorHost](event-hubs-event-processor-host.md)
- [Características y la terminología de Azure Event Hubs](event-hubs-features.md)
- [Preguntas más frecuentes sobre Event Hubs](event-hubs-faq.md)
- Consulte otros ejemplos de Node.js para [Event Hubs](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/eventhub/event-hubs/samples) y [Event Processor Host](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/eventhub/event-processor-host/samples) en GitHub
