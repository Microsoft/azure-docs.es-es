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
ms.openlocfilehash: 3c05596e16edc5243b8a97002a5cc5990c69ec43
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90945626"
---
## <a name="prerequisites"></a>Prerrequisitos
Antes de comenzar, compruebe lo siguiente:

- Cree de una cuenta de Azure con una suscripción activa. Para más información, consulte [Creación de una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F). 
- Instalación de [Python](https://www.python.org/downloads/)
- Cree un recurso de Azure Communication Services. Para obtener más información, consulte [Creación de un recurso de Azure Communication Services](../../create-communication-resource.md). Deberá registrar el **punto de conexión** del recurso para esta guía de inicio rápido.
- Un [Token de acceso de usuario](../../access-tokens.md). Asegúrese de establecer el ámbito en "chat" y anote la cadena del token, así como la cadena userId.

## <a name="setting-up"></a>Instalación

### <a name="create-a-new-python-application"></a>Creación de una nueva aplicación de Python

Abra la ventana de comandos o de terminal, cree un nuevo directorio para la aplicación y navegue hasta este.

```console
mkdir chat-quickstart && cd chat-quickstart
```

Use un editor de texto para crear un archivo denominado **start-chat.py** en el directorio raíz del proyecto y agregue la estructura del programa, incluido el control de excepciones básico. En las secciones siguientes agregará todo el código fuente de esta guía de inicio rápido a este archivo.

```python
import os
# Add required client library components from quickstart here

try:
    # Quickstart code goes here
except Exception as ex:
    print('Exception:')
    print(ex)
```

### <a name="install-client-library"></a>Instalación de la biblioteca cliente

```console

pip install azure-communication-chat

```

## <a name="object-model"></a>Modelo de objetos

Las siguientes clases e interfaces controlan algunas de las características principales de la biblioteca cliente de chat de Azure Communication Services para Python.

| Nombre                                  | Descripción                                                  |
| ------------------------------------- | ------------------------------------------------------------ |
| ChatClient | Esta clase es necesaria para la funcionalidad de chat. Cree una instancia de esta con la información de suscripción y úsela para crear, obtener y eliminar subprocesos. |
| ChatThreadClient | Esta clase es necesaria para la funcionalidad de subproceso de chat. Obtiene una instancia de a través de la clase ChatClient y la usa para enviar, recibir, actualizar o eliminar mensajes, agregar, quitar u obtener usuarios, enviar notificaciones de escritura y confirmaciones de lectura. |

## <a name="create-a-chat-client"></a>Creación de un cliente de chat

Para crear un cliente de chat, usará el punto de conexión de Communication Services y el `Access Token` que se generó como parte de los pasos de requisitos previos. Obtenga más información sobre los [tokens de acceso de usuario](../../access-tokens.md).

```console
pip install azure-communication-administration
```

```python
from azure.communication.chat import ChatClient, CommunicationUserCredential

endpoint = "https://<RESOURCE_NAME>.communication.azure.com"
chat_client = ChatClient(endpoint, CommunicationUserCredential(<Access Token>))
```

## <a name="start-a-chat-thread"></a>Inicio de un subproceso de chat

Use el método `create_chat_thread` para crear un subproceso de chat.

- Use `topic` para proporcionar un tema de subproceso; el tema puede actualizarse después de crear el subproceso de chat mediante la función `update_thread`.
- Use `members` para enumerar el `ChatThreadMember` que se va a agregar al subproceso de chat, el elemento `ChatThreadMember` toma el tipo `CommunicationUser` como `user`, que es el valor que obtuvo después de crearlo con [Creación de un usuario](../../access-tokens.md#create-a-user).

La respuesta `chat_thread_client` se usa para realizar operaciones en el subproceso de chat recién creado, como agregar miembros al subproceso de chat, enviar o eliminar un mensaje, etc. Contiene una propiedad `thread_id` que es el id. único del subproceso de chat.

```python
from datetime import datetime
from azure.communication.chat import ChatThreadMember

topic="test topic"
thread_members=[ChatThreadMember(
    user=user,
    display_name='name',
    share_history_time=datetime.utcnow()
)]
chat_thread_client = chat_client.create_chat_thread(topic, thread_members)
```

## <a name="get-a-chat-thread-client"></a>Obtención de un cliente de subproceso de chat
El método get_chat_thread_client devuelve un cliente de subproceso para un subproceso que ya existe. Se puede usar para realizar operaciones en el subproceso creado: agregar miembros, enviar un mensaje, etc. thread_id es el id. único del subproceso de chat existente.

```python
thread_id = 'id'
chat_thread_client = chat_client.get_chat_thread_client(thread_id)
```

## <a name="send-a-message-to-a-chat-thread"></a>Envío de un mensaje a un subproceso de chat

Utilice el método `send_message` para enviar un mensaje a un subproceso de chat recién creado, identificado mediante threadId.

- Utilice `content` para proporcionar el contenido del mensaje de chat.
- Use `priority` para especificar el nivel de prioridad del mensaje, como "Normal" o "Alta"; esta propiedad se puede usar para que un indicador de la interfaz de usuario haga que el destinatario de la aplicación se fije en el mensaje o para ejecutar la lógica de negocios personalizada.
- Utilice `senderDisplayName` para especificar el nombre para mostrar del remitente.

La respuesta `SendChatMessageResult` contiene un "id.", que es el identificador único del mensaje.

```python
from azure.communication.chat import ChatMessagePriority

content='hello world'
priority=ChatMessagePriority.NORMAL
sender_display_name='sender name'

send_message_result = chat_thread_client.send_message(content, priority=priority, sender_display_name=sender_display_name)
```

## <a name="receive-chat-messages-from-a-chat-thread"></a>Recepción de mensajes de chat de un subproceso de chat

Puede recuperar mensajes de chat mediante el sondeo del método `list_messages` a intervalos especificados.

```python
chat_messages = chat_thread_client.list_messages()
```
`list_messages` devuelve la versión más reciente del mensaje, incluidas las modificaciones o eliminaciones que se han producido en este mediante `update_message` y `delete_message`. En el caso de los mensajes eliminados, `ChatMessage.deleted_on` devuelve un valor de fecha y hora que indica cuándo se eliminó el mensaje. En el caso de los mensajes editados, `ChatMessage.edited_on` devuelve un valor de fecha y hora que indica cuándo se editó el mensaje. Es posible acceder a la hora original de creación del mensaje mediante `ChatMessage.created_on`, que se puede usar para ordenar los mensajes.

`list_messages` devuelve distintos tipos de mensajes que se pueden identificar mediante `ChatMessage.type`. Estos tipos son:

- `Text`: mensaje de chat normal enviado por un miembro del subproceso.

- `ThreadActivity/TopicUpdate`: mensaje del sistema que indica que el tema se ha actualizado.

- `ThreadActivity/AddMember`: mensaje del sistema que indica que uno o más miembros se han agregado al subproceso de chat.

- `ThreadActivity/DeleteMember`: mensaje del sistema que indica que un miembro se han quitado del subproceso de chat.

Para obtener más información, consulte [Tipos de mensajes](../../../concepts/chat/concepts.md#message-types).

## <a name="add-a-user-as-member-to-the-chat-thread"></a>Adición de un usuario como miembro al subproceso de chat

Una vez que se crea un subproceso de chat, puede agregar y quitar usuarios de este. Al agregar usuarios, les concede acceso para poder enviar mensajes al subproceso de chat, y agregar o quitar otros miembros. Antes de llamar al método `add_members`, asegúrese de que ha adquirido un nuevo token de acceso y una identidad para ese usuario. El usuario necesitará ese token de acceso para poder inicializar su cliente de chat.

Utilice el método `add_members` para agregar miembros de subproceso al subproceso identificado por threadId.

- Use `members` para enumerar los miembros que se van a agregar al subproceso de chat.
- El elemento obligatorio `user` es el `CommunicationUser` que creó mediante `CommunicationIdentityClient` en el paso [Creación de un usuario](../../access-tokens.md#create-a-user).
- El elemento opcional `display_name` es el nombre para mostrar del miembro del subproceso.
- El elemento opcional `share_history_time` es la hora a partir de la cual el historial de chat se compartió con el miembro. Para compartir el historial desde el inicio del subproceso de chat, establezca esta propiedad en cualquier fecha igual o anterior a la hora de creación del subproceso. Para no compartir ningún historial anterior a la hora en que se agregó el miembro, establézcala en la fecha actual. Para compartir el historial parcialmente, establézcala en una fecha intermedia.

```python
new_user = identity_client.create_user()

from azure.communication.chat import ChatThreadMember
from datetime import datetime
member = ChatThreadMember(
    user=new_user,
    display_name='name',
    share_history_time=datetime.utcnow())
thread_members = [member]
chat_thread_client.add_members(thread_members)
```

## <a name="remove-user-from-a-chat-thread"></a>Eliminación de un usuario de un subproceso de chat

De forma similar a la adición de un miembro, también puede quitar miembros de un subproceso. Para quitarlos, deberá realizar un seguimiento de los identificadores de los miembros que ha agregado.

Utilice el método `remove_member` para quitar miembros de subproceso del subproceso identificado por threadId.
- `user` es el elemento CommunicationUser que se va a quitar del subproceso.

```python
chat_thread_client.remove_member(user)
```

## <a name="run-the-code"></a>Ejecución del código

Ejecute la aplicación desde el directorio de la aplicación con el comando `python`.

```console
python start-chat.py
```
