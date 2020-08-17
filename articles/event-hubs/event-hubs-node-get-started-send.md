---
title: Envío o recepción de eventos desde Azure Event Hubs mediante JavaScript (heredado)
description: Este artículo es un tutorial para crear una aplicación de JavaScript que envíe eventos a Azure Event Hubs, o los reciba de él, mediante el antiguo paquete azure/event-hubs, versión 2.
ms.topic: quickstart
ms.date: 06/23/2020
ms.custom: devx-track-javascript
ms.openlocfilehash: ffa6b821d617731bd6ee5cbb9bec8bd6d856a7a6
ms.sourcegitcommit: e71da24cc108efc2c194007f976f74dd596ab013
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87414129"
---
# <a name="quickstart-send-events-to-or-receive-events-from-azure-event-hubs-using-javascript-azureevent-hubs-version-2"></a>Inicio rápido: Envío de eventos a Azure Event Hubs o recepción de eventos de él mediante JavaScript (@azure/event-hubsversión 2)
En este inicio rápido, se muestra cómo crear aplicaciones de JavaScript para enviar y recibir eventos desde un centro de eventos mediante el paquete de JavaScript azure/event-hubs, versión 2. 

> [!WARNING]
> En este inicio rápido se usa el paquete azure/event-hubs versión 2 anterior. Para ver un inicio rápido que use la **versión 5** más reciente del paquete, consulte [Envío y recepción de eventos mediante la versión 5 de azure/eventhubs](get-started-node-send-v2.md). Para que la aplicación pase de usar el paquete antiguo a uno nuevo, consulte la [guía para migrar de la versión 1 a la versión 5 de azure/eventhubs](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/eventhub/event-hubs/migrationguide.md). 


## <a name="prerequisites"></a>Prerrequisitos

Si es la primera vez que usa Azure Event Hubs, consulte la [información general de Event Hubs](event-hubs-about.md) antes de continuar con este inicio rápido. 

Para completar este tutorial de inicio rápido, debe cumplir los siguientes requisitos previos:

- Una **suscripción a Microsoft Azure**. Para usar los servicios de Azure, entre los que se incluye Azure Event Hubs, se necesita una suscripción.  Si no se dispone de una cuenta de Azure, es posible registrarse para obtener una [evaluación gratuita](https://azure.microsoft.com/free/), o bien usar las ventajas que disfrutan los suscriptores MSDN al [crear una cuenta](https://azure.microsoft.com).
- Node.js versión 8.x y posteriores. Descargue la versión LTS más reciente en [https://nodejs.org](https://nodejs.org).
- Visual Studio Code (recomendado) o cualquier otro IDE
- **Creación de un espacio de nombres de Event Hubs y un centro de eventos**. El primer paso consiste en usar [Azure Portal](https://portal.azure.com) para crear un espacio de nombres de tipo Event Hubs y obtener las credenciales de administración que la aplicación necesita para comunicarse con el centro de eventos. Para crear un espacio de nombres y un centro de eventos, siga el procedimiento de [este artículo](event-hubs-create.md) y después continúe con los pasos siguientes de este tutorial. Después, obtenga la cadena de conexión para el espacio de nombres del centro de eventos. Para ello, siga las instrucciones del artículo [Obtenga la cadena de conexión](event-hubs-get-connection-string.md#get-connection-string-from-the-portal). Utilizará la cadena de conexión más adelante en el tutorial.


### <a name="install-npm-package"></a>Instalación del paquete de npm
Para instalar el [paquete de npm para Event Hubs](https://www.npmjs.com/package/@azure/event-hubs/v/2.1.0), abra un símbolo del sistema que tenga `npm` en la ruta de acceso, cambie el directorio a la carpeta en la que quiere tener los ejemplos y ejecute este comando.

```shell
npm install @azure/event-hubs@2
```

Para instalar el [paquete de npm para el Host de procesador de eventos](https://www.npmjs.com/package/@azure/event-processor-host), ejecute el comando siguiente.

```shell
npm install @azure/event-processor-host
```

## <a name="send-events"></a>Envío de eventos

En esta sección se muestra cómo crear una aplicación de JavaScript que envíe eventos a un centro de eventos. 

> [!NOTE]
> Puede descargar esta guía de inicio rápido como un ejemplo desde [GitHub](https://github.com/Azure/azure-event-hubs-node/tree/master/client), reemplazar las cadenas `EventHubConnectionString` y `EventHubName` por los valores del centro de eventos, y ejecutarlo. También puede seguir los pasos de este tutorial para crear el suyo propio.

1. Abra el editor que prefiera, como [Visual Studio Code](https://code.visualstudio.com). 
2. Cree un archivo denominado `send.js` y pegue en él el código siguiente. Obtenga la cadena de conexión para el espacio de nombres del centro de eventos siguiendo las instrucciones del artículo: [Obtenga la cadena de conexión](event-hubs-get-connection-string.md#get-connection-string-from-the-portal). 

    ```javascript
    const { EventHubClient } = require("@azure/event-hubs@2");

    // Connection string - primary key of the Event Hubs namespace. 
    // For example: Endpoint=sb://myeventhubns.servicebus.windows.net/;SharedAccessKeyName=RootManageSharedAccessKey;SharedAccessKey=XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
    const connectionString = "Endpoint=sb://<EVENT HUBS NAMESPACE NAME>.servicebus.windows.net/;SharedAccessKeyName=RootManageSharedAccessKey;SharedAccessKey=<SHARED ACCESS KEY>";

    // Name of the event hub. For example: myeventhub
    const eventHubsName = "<EVENT HUB NAME>";

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
3. Escriba la cadena de conexión y el nombre del centro de eventos en el código anterior.
4. Después, ejecute el comando `node send.js` en un símbolo del sistema para ejecutar este archivo. De este modo, se enviarán 100 eventos a su centro de eventos.

Felicidades. Ha enviado eventos a un centro de eventos.


## <a name="receive-events"></a>Recepción de eventos

En esta sección se muestra cómo crear una aplicación de JavaScript que recibe eventos de una única partición del grupo de consumidores predeterminado de un centro de eventos. 

1. Abra el editor que prefiera, como [Visual Studio Code](https://code.visualstudio.com). 
2. Cree un archivo denominado `receive.js` y pegue en él el código siguiente.
    ```javascript
    const { EventHubClient, delay } = require("@azure/event-hubs@2");

    // Connection string - primary key of the Event Hubs namespace. 
    // For example: Endpoint=sb://myeventhubns.servicebus.windows.net/;SharedAccessKeyName=RootManageSharedAccessKey;SharedAccessKey=XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
    const connectionString = "Endpoint=sb://<EVENT HUBS NAMESPACE NAME>.servicebus.windows.net/;SharedAccessKeyName=RootManageSharedAccessKey;SharedAccessKey=<SHARED ACCESS KEY>";

    // Name of the event hub. For example: myeventhub
    const eventHubsName = "<EVENT HUB NAME>";

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
4. Después, ejecute el comando `node receive.js` en un símbolo del sistema para ejecutar este archivo. Recibirá los eventos de una de las particiones del grupo de consumidores predeterminado en su centro de eventos.

Felicidades. Ha recibido eventos procedentes del centro de eventos.

## <a name="receive-events-using-event-processor-host"></a>Recepción de eventos mediante el Host de procesador de eventos

En esta sección se muestra cómo recibir eventos de un centro de eventos mediante [EventProcessorHost](event-hubs-event-processor-host.md) de Azure en una aplicación de JavaScript. EventProcessorHost (EPH) ayuda a recibir de manera eficaz eventos de un centro de eventos mediante la creación de receptores en todas las particiones del grupo de consumidores de un centro de eventos. Establece un punto de comprobación de los metadatos de los mensajes recibidos a intervalos regulares en una instancia de Azure Storage Blob. Este enfoque le permite seguir recibiendo más adelante mensajes desde donde lo dejó.

1. Abra el editor que prefiera, como [Visual Studio Code](https://code.visualstudio.com). 
2. Cree un archivo denominado `receiveAll.js` y pegue en él el código siguiente.
    ```javascript
    const { EventProcessorHost, delay } = require("@azure/event-processor-host");

    // Connection string - primary key of the Event Hubs namespace. 
    // For example: Endpoint=sb://myeventhubns.servicebus.windows.net/;SharedAccessKeyName=RootManageSharedAccessKey;SharedAccessKey=XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
    const eventHubConnectionString = "Endpoint=sb://<EVENT HUBS NAMESPACE NAME>.servicebus.windows.net/;SharedAccessKeyName=RootManageSharedAccessKey;SharedAccessKey=<SHARED ACCESS KEY>";

    // Name of the event hub. For example: myeventhub
    const eventHubName = "<EVENT HUB NAME>";

    // Azure Storage connection string
    const storageConnectionString = "DefaultEndpointsProtocol=https;AccountName=<STORAGE ACCOUNT NAME>;AccountKey=<STORAGE ACCOUNT KEY>;EndpointSuffix=core.windows.net";

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
3. Escriba la cadena de conexión y el nombre del centro de eventos en el código anterior, junto con la cadena de conexión de Azure Blob Storage.
4. Después, ejecute el comando `node receiveAll.js` en un símbolo del sistema para ejecutar este archivo.

Felicidades. Ha recibido eventos de un centro de eventos por medio del Host de procesador de eventos. Recibirá los eventos de todas las particiones del grupo de consumidores predeterminado en su centro de eventos.

## <a name="next-steps"></a>Pasos siguientes
Lea los siguientes artículos:

- [EventProcessorHost](event-hubs-event-processor-host.md)
- [Características y terminología de Azure Event Hubs](event-hubs-features.md)
- [Preguntas más frecuentes sobre Event Hubs](event-hubs-faq.md)
- Consulte otros ejemplos de JavaScript para [Event Hubs](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/eventhub/event-hubs/samples) y [Host de procesador de eventos](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/eventhub/event-processor-host/samples) en GitHub
