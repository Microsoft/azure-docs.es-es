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
ms.openlocfilehash: f32b81184b9a96760e43d0cd64c27459952c5953
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90945642"
---
## <a name="prerequisites"></a>Prerrequisitos
Antes de comenzar, compruebe lo siguiente:
- Cree de una cuenta de Azure con una suscripción activa. Para más información, consulte [Creación de una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F). 
- Instalar [Visual Studio](https://visualstudio.microsoft.com/downloads/) 
- Cree un recurso de Azure Communication Services. Para obtener más información, consulte [Creación de un recurso de Azure Communication Services](../../create-communication-resource.md). Deberá registrar el **punto de conexión** del recurso para esta guía de inicio rápido.
- Un [Token de acceso de usuario](../../access-tokens.md). Asegúrese de establecer el ámbito en "chat" y anote la cadena del token, así como la cadena userId.

## <a name="setting-up"></a>Instalación

### <a name="create-a-new-c-application"></a>Creación de una aplicación de C#

En una ventana de consola (por ejemplo, cmd, PowerShell o Bash), use el comando `dotnet new` para crear una nueva aplicación de consola con el nombre `ChatQuickstart`. Este comando crea un sencillo proyecto "Hola mundo" de C# con un solo archivo de origen: **Program.cs**.

```console
dotnet new console -o ChatQuickstart
```

Cambie el directorio a la carpeta de la aplicación recién creada y use el comando `dotnet build` para compilar la aplicación.

```console
cd ChatQuickstart
dotnet build
```

### <a name="install-the-package"></a>Instalar el paquete

Instale la biblioteca cliente de chat de comunicación de Azure para .NET

```PowerShell
dotnet add package Azure.Communication.Chat
``` 

## <a name="object-model"></a>Modelo de objetos

Las siguientes clases controlan algunas de las características principales de la biblioteca cliente de chat de Azure Communication Services para C#.

| Nombre                                  | Descripción                                                  |
| ------------------------------------- | ------------------------------------------------------------ |
| ChatClient | Esta clase es necesaria para la funcionalidad de chat. Cree una instancia de esta con la información de suscripción y úsela para crear, obtener y eliminar subprocesos. |
| ChatThreadClient | Esta clase es necesaria para la funcionalidad de subproceso de chat. Obtiene una instancia de a través de la clase ChatClient y la usa para enviar, recibir, actualizar o eliminar mensajes, agregar, quitar u obtener usuarios, enviar notificaciones de escritura y confirmaciones de lectura. |

## <a name="create-a-chat-client"></a>Creación de un cliente de chat

Para crear un cliente de chat, usará el punto de conexión de Communication Services y el token de acceso que se generó como parte de los pasos de requisitos previos. Debe usar la clase `CommunicationIdentityClient` de la biblioteca cliente de `Administration` para crear un usuario y emitir un token para pasarlo al cliente de chat. Obtenga más información sobre los [tokens de acceso de usuario](../../access-tokens.md).

```csharp
using Azure.Communication.Identity;
using Azure.Communication.Chat;

// Your unique Azure Communication service endpoint
Uri endpoint = new Uri("https://<RESOURCE_NAME>.communication.azure.com");

CommunicationUserCredential communicationUserCredential = new CommunicationUserCredential(<Access_Token>);
ChatClient chatClient = new ChatClient(endpoint, communicationUserCredential);
```

## <a name="start-a-chat-thread"></a>Inicio de un subproceso de chat

Use el método `createChatThread` para crear un subproceso de chat.
- Use `topic` para proporcionar un tema a este chat; el tema puede actualizarse después de crear el subproceso de chat mediante la función `UpdateThread`.
- Utilice la propiedad `members` para pasar una lista de objetos `ChatThreadMember` que se van a agregar al subproceso de chat. El objeto `ChatThreadMember` se inicializó con un objeto `CommunicationUser`. Para obtener un objeto `CommunicationUser`, deberá pasar un id. de acceso creado siguiendo las instrucciones de [Creación de un usuario](../../access-tokens.md#create-a-user).

La respuesta `chatThreadClient` se usa para realizar operaciones en el subproceso de chat creado: agregar miembros al subproceso de chat, enviar o eliminar un mensaje, etc. Contiene el atributo `Id`, que es el id. único del subproceso de chat. 

```csharp
var chatThreadMember = new ChatThreadMember(new CommunicationUser("<Access_ID>"))
{
    DisplayName = "UserDisplayName"
};
ChatThreadClient chatThreadClient = await chatClient.CreateChatThreadAsync(topic: "Chat Thread topic C# sdk", members: new[] { chatThreadMember });
string threadId = chatThreadClient.Id;
```

## <a name="get-a-chat-thread-client"></a>Obtención de un cliente de subproceso de chat
El método `GetChatThreadClient` devuelve un cliente de subproceso para un subproceso que ya existe. Se puede usar para realizar operaciones en el subproceso creado: agregar miembros, enviar un mensaje, etc. thread_id es el id. único del subproceso de chat existente.

```csharp
string threadId = "<THREAD_ID>";
ChatThreadClient chatThreadClient = await chatClient.GetChatThreadClient(threadId);
```

## <a name="send-a-message-to-a-chat-thread"></a>Envío de un mensaje a un subproceso de chat

Utilice el método `SendMessage` para enviar un mensaje a un subproceso identificado mediante threadId.

- Utilice `content` para proporcionar el contenido del mensaje de chat, que es obligatorio.
- Use `priority` para especificar el nivel de prioridad del mensaje, como "Normal" o "Alta". Si no se especifica, se usará "Normal".
- Use `senderDisplayName` para especificar el nombre para mostrar del remitente. Si no se especifica, se usará un nombre vacío.

`SendChatMessageResult` es la respuesta que se devuelve al enviar un mensaje. Contiene un id. que es el identificador único del mensaje.

```csharp
var content = "hello world";
var priority = ChatMessagePriority.Normal;
var senderDisplayName = "sender name";

SendChatMessageResult sendChatMessageResult = await chatThreadClient.SendMessageAsync(content, priority, senderDisplayName);
string messageId = sendChatMessageResult.Id;
```

## <a name="receive-chat-messages-from-a-chat-thread"></a>Recepción de mensajes de chat de un subproceso de chat

También puede recuperar mensajes de chat mediante el sondeo del método `GetMessages` en el cliente de subproceso de chat a intervalos especificados.

```csharp
AsyncPageable<ChatMessage> allMessages = chatThreadClient.GetMessagesAsync();
await foreach (ChatMessage message in allMessages)
{
    Console.WriteLine($"{message.Id}:{message.Sender.Id}:{message.Content}");
}
```

`GetMessages` toma un parámetro `DateTimeOffset` opcional. Si se especifica ese desplazamiento, recibirá los mensajes recibidos, actualizados o eliminados posteriormente. Tenga en cuenta que los mensajes recibidos antes de la hora de desplazamiento, pero editados o eliminados posteriormente, también se devolverán.

`GetMessages` devuelve la versión más reciente del mensaje, incluidas las modificaciones o eliminaciones que se han producido en este mediante `UpdateMessage` y `DeleteMessage`. En el caso de los mensajes eliminados, `chatMessage.DeletedOn` devuelve un valor de fecha y hora que indica cuándo se eliminó el mensaje. En el caso de los mensajes editados, `chatMessage.EditedOn` devuelve un valor de fecha y hora que indica cuándo se editó el mensaje. Es posible acceder a la hora original de creación del mensaje mediante `chatMessage.CreatedOn`, que se puede usar para ordenar los mensajes.

`GetMessages` devuelve distintos tipos de mensajes que se pueden identificar mediante `chatMessage.Type`. Estos tipos son:

- `Text`: mensaje de chat normal enviado por un miembro del subproceso.

- `ThreadActivity/TopicUpdate`: mensaje del sistema que indica que el tema se ha actualizado.

- `ThreadActivity/AddMember`: mensaje del sistema que indica que uno o más miembros se han agregado al subproceso de chat.

- `ThreadActivity/DeleteMember`: mensaje del sistema que indica que un miembro se han quitado del subproceso de chat.

Para obtener más información, consulte [Tipos de mensajes](../../../concepts/chat/concepts.md#message-types).

## <a name="update-a-message"></a>Actualización de un mensaje

Para actualizar un mensaje que ya se envío, puede invocar `UpdateMessage` en `ChatThreadClient`.

```csharp
string id = "id-of-message-to-edit";
string content = "updated content";
await chatThreadClient.UpdateMessageAsync(id, content);
```

## <a name="deleting-a-message"></a>Eliminación de un mensaje

Para eliminar un mensaje, puede invocar `DeleteMessage` en `ChatThreadClient`.

```csharp
string id = "id-of-message-to-delete";
await chatThreadClient.DeleteMessageAsync(id);
```

## <a name="add-a-user-as-member-to-the-chat-thread"></a>Adición de un usuario como miembro al subproceso de chat

Una vez que se crea un subproceso, puede agregar y quitar usuarios de este. Al agregar usuarios, les concede acceso para poder enviar mensajes al subproceso, y agregar o quitar otros miembros. Antes de llamar a `AddMembers`, asegúrese de que ha adquirido un token de acceso y una identidad nuevos para ese usuario. El usuario necesitará ese token de acceso para poder inicializar su cliente de chat.

Utilice el método `AddMembers` para agregar miembros de subproceso al subproceso identificado por threadId.

 - Use `members` para enumerar los miembros que se van a agregar al subproceso de chat.
 - El elemento `User` obligatorio es la identidad obtenida para el nuevo usuario.
 - El elemento opcional `DisplayName` es el nombre para mostrar del miembro del subproceso.
 - El elemento opcional `ShareHistoryTime` es la hora a partir de la cual el historial de chat se compartió con el miembro. Para compartir el historial desde el principio del subproceso de chat, establezca esta opción en DateTime.MinValue. Para no compartir ningún historial anterior a la hora en que se agregó el miembro, establézcala en la hora actual. Para compartir el historial parcial, establezca la opción en un momento entre la creación del subproceso y la hora actual.

```csharp
ChatThreadMember member = new ChatThreadMember(communicationUser);
member.DisplayName = "display name member 1";
member.ShareHistoryTime = DateTime.MinValue; // share all history
await chatThreadClient.AddMembersAsync(members: new[] {member});
```
## <a name="remove-user-from-a-chat-thread"></a>Eliminación de un usuario de un subproceso de chat

De forma similar a la adición de un usuario a un subproceso, puede quitar usuarios de un subproceso de chat. Para ello, debe realizar un seguimiento de la identidad (CommunicationUser) de los miembros que ha agregado.

```csharp
await chatThreadClient.RemoveMemberAsync(communicationUser);
```

## <a name="run-the-code"></a>Ejecución del código

Ejecute la aplicación desde el directorio de la aplicación con el comando `dotnet run`.

```console
dotnet run
```
