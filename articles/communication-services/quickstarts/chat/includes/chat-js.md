---
title: archivo de inclusión
description: archivo de inclusión
services: azure-communication-services
author: mikben
manager: mikben
ms.service: azure-communication-services
ms.subservice: azure-communication-services
ms.date: 9/1/2020
ms.topic: include
ms.custom: include file
ms.author: mikben
ms.openlocfilehash: a50f09dd52ccf4d35fd3803967898a4a25e3299f
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90945627"
---
## <a name="prerequisites"></a>Prerrequisitos
Antes de comenzar, compruebe lo siguiente:

- Cree de una cuenta de Azure con una suscripción activa. Para más información, consulte [Creación de una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F). 
- Instale las versiones de [Node.js](https://nodejs.org/en/download/), Active LTS y Maintenance LTS (se recomiendan 8.11.1 y 10.14.1).
- Cree un recurso de Azure Communication Services. Para obtener más información, consulte [Creación de un recurso de Azure Communication Services](../../create-communication-resource.md). Deberá registrar el **punto de conexión** del recurso para esta guía de inicio rápido.
- Un [Token de acceso de usuario](../../access-tokens.md). Asegúrese de establecer el ámbito en "chat" y anote la cadena del token, así como la cadena userId.

## <a name="setting-up"></a>Instalación

### <a name="create-a-new-web-application"></a>Creación de una nueva aplicación web

En primer lugar, abra la ventana de comandos o de terminal, cree un nuevo directorio para la aplicación y navegue hasta este.

```console
mkdir chat-quickstart && cd chat-quickstart
```
   
Ejecute `npm init -y` para crear un archivo **package.json** con la configuración predeterminada.

```console
npm init -y
```

Use un editor de texto para crear un archivo denominado **start-chat.js** en el directorio raíz del proyecto. En las secciones siguientes agregará todo el código fuente de esta guía de inicio rápido a este archivo.

### <a name="install-the-packages"></a>Instalación de los paquetes

Use el comando `npm install` para instalar las siguientes bibliotecas de cliente de Communication Services para JavaScript.

```console
npm install @azure/communication-common --save

npm install @azure/communication-administration --save

npm install @azure/communication-signaling --save

npm install @azure/communication-chat --save

```

La opción `--save` muestra la biblioteca como una dependencia en el archivo **package.json**.

### <a name="set-up-the-app-framework"></a>Instalación del marco de la aplicación

Esta guía de inicio rápido usa webpack para agrupar los recursos de la aplicación. Ejecute el siguiente comando para instalar los paquetes de npm webpack, webpack-cli y webpack-dev-server, y mostrarlos como dependencias de desarrollo en el archivo **package.json**:

```console
npm install webpack webpack-cli webpack-dev-server --save-dev
```

Cree un archivo **index.html** en el directorio raíz del proyecto. Usaremos este archivo como plantilla para agregar la funcionalidad de chat mediante la biblioteca de cliente de chat de comunicación de Azure para JavaScript.

Este es el código:

```html
<!DOCTYPE html>
<html>
  <head>
    <title>Communication Client - Chat Sample</title>
  </head>
  <body>
    <h4>Azure Communication Services</h4>
    <h1>Chat Quickstart</h1>
    <script src="./bundle.js"></script>
  </body>
</html>
```
Cree un archivo en el directorio raíz del proyecto denominado **client.js** que contendrá la lógica de la aplicación para esta guía de inicio rápido. 

### <a name="create-a-chat-client"></a>Creación de un cliente de chat

Para crear un cliente de chat, usará el punto de conexión de Communication Services y el token de acceso que se generó como parte de los pasos de requisitos previos. Los tokens de acceso de usuario permiten compilar aplicaciones cliente que se autentiquen directamente en Azure Communication Services. Después de generar estos tokens en el servidor, vuelva a pasarlos a un dispositivo cliente. Debe usar la clase `AzureCommunicationUserCredential` del `Common client library` para pasar el token al cliente de chat.

```JavaScript

import { ChatClient } from '@azure/communication-chat';
import { AzureCommunicationUserCredential } from '@azure/communication-common';

// Your unique Azure Communication service endpoint
let endpointUrl = 'https://<RESOURCE_NAME>.communication.azure.com';
let userAccessToken = '<USER_ACCESS_TOKEN>';

let chatClient = new ChatClient(endpointUrl, new AzureCommunicationUserCredential(userAccessToken));
console.log('Azure Communication Chat client created!');
```
Reemplace **ENDPOINT** por el punto de conexión creado anteriormente basado en la documentación de [Creación de un recurso de Azure Communication Services](../../create-communication-resource.md).
Reemplace **USER_ACCESS_TOKEN** por un token emitido basado en la documentación de [Token de acceso de usuario](../../access-tokens.md).
Agregue este código al archivo **client.js**.


### <a name="run-the-code"></a>Ejecución del código
Utilice `webpack-dev-server` para compilar y ejecutar su aplicación. Ejecute el siguiente comando para agrupar el host de aplicación en un servidor web local:
```console
npx webpack-dev-server --entry ./client.js --output bundle.js --debug --devtool inline-source-map
```
Abra el explorador web y vaya a http://localhost:8080/.
En la consola de herramientas de desarrollo en el explorador, debe ver lo siguiente:

```console
Azure Communication Chat client created!
```

## <a name="object-model"></a>Modelo de objetos 
Las siguientes clases e interfaces controlan algunas de las características principales de la biblioteca cliente de chat de Azure Communication Services para JavaScript.

| Nombre                                   | Descripción                                                                                                                                                                           |
| -------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| ChatClient | Esta clase es necesaria para la funcionalidad de chat. Cree una instancia de esta con la información de suscripción y úsela para crear, obtener y eliminar subprocesos. |
| ChatThreadClient | Esta clase es necesaria para la funcionalidad de subproceso de chat. Obtiene una instancia de a través de la clase ChatClient y la usa para enviar, recibir, actualizar o eliminar mensajes, agregar, quitar u obtener usuarios, enviar notificaciones de escritura y confirmaciones de lectura, y suscribirse a eventos de chat. |


## <a name="start-a-chat-thread"></a>Inicio de un subproceso de chat

Use el método `createThread` para crear un subproceso de chat.

`createThreadRequest` se usa para describir la solicitud de subproceso:

- Use `topic` para proporcionar un tema a este chat; el tema puede actualizarse después de crear el subproceso de chat mediante la función `UpdateThread`. 
- Use `members` para enumerar los miembros que se van a agregar al subproceso de chat.

Una vez resuelto, el método `createChatThread` devuelve `threadId`, que se usa para realizar operaciones en el subproceso de chat recién creado, como agregar miembros al subproceso de chat, enviar mensajes, eliminar un mensaje, etc.

```Javascript
async function createChatThread() {
   let createThreadRequest = {
       topic: 'Preparation for London conference',
       members: [{
                   user: { communicationUserId: '<USER_ID_FOR_JACK>' },
                   displayName: 'Jack'
               }, {
                   user: { communicationUserId: '<USER_ID_FOR_GEETA>' },
                   displayName: 'Geeta'
               }]
   };
   let chatThreadClient= await chatClient.createChatThread(createThreadRequest);
   let threadId = chatThreadClient.threadId;
   return threadId;
}

createChatThread().then(async threadId => {
   console.log(`Thread created:${threadId}`);
   // PLACEHOLDERS
   // <CREATE CHAT THREAD CLIENT>
   // <RECEIVE A CHAT MESSAGE FROM A CHAT THREAD>
   // <SEND MESSAGE TO A CHAT THREAD>
   // <LIST MESSAGES IN A CHAT THREAD>
   // <ADD NEW MEMBER TO THREAD>
   // <LIST MEMBERS IN A THREAD>
   // <REMOVE MEMBER FROM THREAD>
});
```

Reemplace **USER_ID_FOR_JACK** y **USER_ID_FOR_GEETA** por los id. de usuario obtenidos en el paso anterior (creación de usuarios y emisión de [tokens de acceso de usuario](../../access-tokens.md))

Al actualizar la pestaña del explorador, debería ver lo siguiente en la consola.
```console
Thread created: <threadId>
```

## <a name="get-a-chat-thread-client"></a>Obtención de un cliente de subproceso de chat

El método `getChatThreadClient` devuelve un `chatThreadClient` para un subproceso que ya existe. Se puede usar para realizar operaciones en el subproceso creado: agregar miembros, enviar un mensaje, etc. thread_id es el id. único del subproceso de chat existente.

```JavaScript

let chatThreadClient = await chatClient.getChatThreadClient(threadId);
console.log(`Chat Thread client for threadId:${chatThreadClient.threadId}`);

```
Agregue este código en lugar del comentario `<CREATE CHAT THREAD CLIENT>` en **client.js**, actualice la pestaña del explorador y compruebe la consola. Debería ver lo siguiente:
```console
Chat Thread client for threadId: <threadId>
```

## <a name="send-a-message-to-a-chat-thread"></a>Envío de un mensaje a un subproceso de chat

Utilice el método `sendMessage` para enviar un mensaje de chat al subproceso recién creado, identificado mediante threadId.

`sendMessageRequest` describe los campos obligatorios de la solicitud del mensaje de chat:

- Utilice `content` para proporcionar el contenido del mensaje de chat.

`sendMessageOptions` describe los campos opcionales de la solicitud del mensaje de chat:

- Use `priority` para especificar el nivel de prioridad del mensaje de chat, como "Normal" o "Alta"; esta propiedad se puede usar para que un indicador de la interfaz de usuario haga que el destinatario de la aplicación se fije en el mensaje o para ejecutar la lógica de negocios personalizada.   
- Utilice `senderDisplayName` para especificar el nombre para mostrar del remitente.

La respuesta `sendChatMessageResult` contiene un "id.", que es el identificador único del mensaje.

```JavaScript

let sendMessageRequest =
{
    content: 'Hello Geeta! Can you share the deck for the conference?'
};
let sendMessageOptions =
{
    priority: 'Normal',
    senderDisplayName : 'Jack'
};
let sendChatMessageResult = await chatThreadClient.sendMessage(sendMessageRequest, sendMessageOptions);
let messageId = sendChatMessageResult.id;
console.log(`Message sent!, message id:${messageId}`);

```
Agregue este código en lugar del comentario `<SEND MESSAGE TO A CHAT THREAD>` en el archivo **client.js**, actualice la pestaña del explorador y compruebe la consola.
```console
Message sent!, message id:<number>
```

## <a name="receive-chat-messages-from-a-chat-thread"></a>Recepción de mensajes de chat de un subproceso de chat

Con la señalización en tiempo real, puede suscribirse para escuchar nuevos mensajes entrantes y actualizar los mensajes actuales en la memoria en consecuencia. Azure Communication Services admite una [lista de eventos a los que se puede suscribir](../../../concepts/chat/concepts.md#real-time-signaling).

```JavaScript
// open notifications channel
await chatClient.startRealtimeNotifications();
// subscribe to new notification
chatClient.on("chatMessageReceived", (e) => {
    console.log("Notification chatMessageReceived!");
    // your code here
});

```
Agregue este código en lugar del comentario `<RECEIVE A CHAT MESSAGE FROM A CHAT THREAD>` en **client.js**.
Actualice la pestaña del explorador; debería ver en la consola un mensaje `Notification chatMessageReceived`.

También puede recuperar mensajes de chat mediante el sondeo del método `listMessages` a intervalos especificados. 

```JavaScript

let pagedAsyncIterableIterator = await chatThreadClient.listMessages();
let nextMessage = await pagedAsyncIterableIterator.next();
 while (!nextMessage.done) {
     let chatMessage = nextMessage.value;
     console.log(`Message :${chatMessage.content}`);
     // your code here
     nextMessage = await pagedAsyncIterableIterator.next();
 }

```
Agregue este código en lugar del comentario `<LIST MESSAGES IN A CHAT THREAD>` en **client.js**.
Actualice la pestaña; en la consola debería encontrar una lista de los mensajes enviados en este subproceso de chat.


`listMessages` devuelve la versión más reciente del mensaje, incluidas las modificaciones o eliminaciones que se han producido en este mediante `updateMessage` y `deleteMessage`.
En el caso de los mensajes eliminados, `chatMessage.deletedOn` devuelve un valor de fecha y hora que indica cuándo se eliminó el mensaje. En el caso de los mensajes editados, `chatMessage.editedOn` devuelve un valor de fecha y hora que indica cuándo se editó el mensaje. Es posible acceder a la hora original de creación del mensaje mediante `chatMessage.createdOn`, que se puede usar para ordenar los mensajes.

`listMessages` devuelve distintos tipos de mensajes que se pueden identificar mediante `chatMessage.type`. Estos tipos son:

- `Text`: mensaje de chat normal enviado por un miembro del subproceso.

- `ThreadActivity/TopicUpdate`: mensaje del sistema que indica que el tema se ha actualizado.

- `ThreadActivity/AddMember`: mensaje del sistema que indica que uno o más miembros se han agregado al subproceso de chat.

- `ThreadActivity/RemoveMember`: mensaje del sistema que indica que un miembro se han quitado del subproceso de chat.

Para obtener más información, consulte [Tipos de mensajes](../../../concepts/chat/concepts.md#message-types).

## <a name="add-a-user-as-member-to-the-chat-thread"></a>Adición de un usuario como miembro al subproceso de chat

Una vez que se crea un subproceso de chat, puede agregar y quitar usuarios de este. Al agregar usuarios, les concede acceso para enviar mensajes al subproceso de chat, y agregar o quitar otros miembros. Antes de llamar al método `addMembers`, asegúrese de que ha adquirido un nuevo token de acceso y una identidad para ese usuario. El usuario necesitará ese token de acceso para poder inicializar su cliente de chat.

`addMembersRequest` describe el objeto de solicitud, donde `members` enumera los miembros que se van a agregar al subproceso de chat.
- El elemento `user` obligatorio es el usuario de comunicación que se va a agregar al subproceso de chat.
- El elemento opcional `displayName` es el nombre para mostrar del miembro de subproceso.
- El elemento opcional `shareHistoryTime` es la hora a partir de la cual el historial de chat se compartió con el miembro. Para compartir el historial desde el inicio del subproceso de chat, establezca esta propiedad en cualquier fecha igual o anterior a la hora de creación del subproceso. Para no compartir ningún historial anterior a la hora en que se agregó el miembro, establézcala en la fecha actual. Para compartir el historial parcialmente, establezca la opción en la fecha de su elección.

```JavaScript

let addMembersRequest =
{
    members: [
        {
            user: { communicationUserId: '<NEW_MEMBER_USER_ID>' },
            displayName: 'Jane'
        }
    ]
};

await chatThreadClient.addMembers(addMembersRequest);

```
Reemplace **NEW_MEMBER_USER_ID** por un [Id. de nuevo usuario](../../access-tokens.md). Agregue este código en lugar del comentario `<ADD NEW MEMBER TO THREAD>` en **client.js**.

## <a name="list-users-in-a-chat-thread"></a>Enumeración de usuarios en un subproceso de chat
```JavaScript
async function listThreadMembers() {
   let pagedAsyncIterableIterator = await chatThreadClient.listMembers();
   let next = await pagedAsyncIterableIterator.next();
   while (!next.done) {
      let user = next.value;
      console.log(`User :${user.displayName}`);
      next = await pagedAsyncIterableIterator.next();
   }
}
await listThreadMembers();
```
Agregue este código en lugar del comentario `<LIST MEMBERS IN A THREAD>` en **client.js**, actualice la pestaña del explorador y compruebe la consola; debería ver información acerca de los usuarios de un subproceso.

## <a name="remove-user-from-a-chat-thread"></a>Eliminación de un usuario de un subproceso de chat

De forma similar a la adición de un miembro, puede quitar miembros de un subproceso de chat. Para quitarlos, deberá realizar un seguimiento de los identificadores de los miembros que ha agregado.

Utilice el método `removeMember`, donde `member` es el usuario de la comunicación que se va a quitar del subproceso.

```JavaScript

await chatThreadClient.removeMember({ communicationUserId: <MEMBER_ID> });
await listThreadMembers();
```
Reemplace **MEMBER_ID** por un id. de usuario utilizado en el paso anterior (<NEW_MEMBER_USER_ID>).
Agregue este código en lugar del comentario `<REMOVE MEMBER FROM THREAD>` en **client.js**.
