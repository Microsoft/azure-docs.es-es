---
title: Conceptos de chat en Azure Communication Services
titleSuffix: An Azure Communication Services concept document
description: Obtenga información sobre los conceptos de chat de Communication Services.
author: mikben
manager: jken
services: azure-communication-services
ms.author: mikben
ms.date: 03/10/2020
ms.topic: overview
ms.service: azure-communication-services
ms.openlocfilehash: af07894fcbfae386849d32492be9d2718a3adcc3
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90945483"
---
# <a name="chat-concepts"></a>Conceptos de chat

[!INCLUDE [Public Preview Notice](../../includes/public-preview-include.md)]

Las bibliotecas cliente de chat de Azure Communication Services se pueden usar para agregar chat de texto en tiempo real a las aplicaciones. En esta página se resumen las funcionalidades y conceptos clave del chat.

Consulte la [Información general sobre la biblioteca cliente de chat de Communication Services](./sdk-features.md) para obtener más información sobre los lenguajes y funcionalidades específicos de la biblioteca cliente.

## <a name="chat-overview"></a>Introducción al chat 

Las conversaciones de chat se producen dentro de los hilos o conversaciones de chat. Una conversación de chat puede contener muchos mensajes y muchos usuarios. Cada mensaje pertenece a una conversación única, y un usuario puede formar parte de una o varias conversaciones. 

A cada usuario de la conversación de chat se le denomina miembro. Una conversación de chat puede tener hasta 250 miembros. Solo los miembros de una conversación pueden enviar y recibir mensajes, o agregar o quitar miembros de esta. El tamaño máximo de mensaje permitido es de aproximadamente 28 KB. Communication Services almacena el historial de chat hasta que se ejecuta una operación de eliminación en la conversación de chat. Puede recuperar todos los mensajes de una conversación de chat mediante la operación `List/Get Messages`.

En el caso de las conversaciones de chat con más de 20 miembros, las características de confirmación de lectura e indicador de escritura están deshabilitadas. 

## <a name="chat-architecture"></a>Arquitectura del chat

Hay dos componentes principales en la arquitectura de chat: El 1) servicio de confianza y la 2) aplicación cliente.

:::image type="content" source="../../media/chat-architecture.png" alt-text="Diagrama que muestra la arquitectura de chat de Communication Services.":::

 - **Servicio de confianza:** para administrar correctamente una sesión de chat, necesita un servicio que le ayude a conectarse a Communication Services mediante la cadena de conexión del recurso. Este servicio es responsable de la creación de las conversaciones de chat, la administración de las pertenencias a dichas conversaciones y el suministro de tokens de acceso a los usuarios. Puede encontrar más información sobre los tokens de acceso en la guía de inicio rápido sobre [tokens de acceso](../../quickstarts/access-tokens.md).

 - **Aplicación cliente:**  la aplicación cliente se conecta al servicio de confianza y recibe los tokens de acceso que se usan para conectarse directamente a Communication Services. Una vez realizada la conexión, la aplicación cliente puede enviar y recibir mensajes.
    
## <a name="message-types"></a>Tipos de mensaje

El chat de Communication Services comparte los mensajes generados por el usuario, así como los mensajes generados por el sistema, denominados **actividades de conversación**. Las actividades de conversación se generan cuando se actualiza una conversación de chat. Cuando se llama a `List Messages` o `Get Messages` en una conversación de chat, el resultado contiene los mensajes de texto generados por el usuario, así como los mensajes del sistema en orden cronológico. De este modo, puede identificar cuándo se ha agregado o quitado un miembro, o cuándo se ha actualizado el tema de conversación. Los siguientes son los tipos de mensajes admitidos:  

 - `Text`: mensaje real creado y enviado por el usuario como parte de la conversación de chat. 
 - `ThreadActivity/AddMember`: mensaje del sistema que indica que uno o más miembros se han agregado a la conversación de chat. Por ejemplo:

```xml

<addmember>
    <eventtime>1598478187549</eventtime>
    <initiator>8:acs:57b9bac9-df6c-4d39-a73b-26e944adf6ea_0e59221d-0c1d-46ae-9544-c963ce56c10b</initiator>
    <detailedinitiatorinfo>
        <friendlyName>User 1</friendlyName>
    </detailedinitiatorinfo>
    <rosterVersion>1598478184564</rosterVersion>
    <target>8:acs:57b9bac9-df6c-4d39-a73b-26e944adf6ea_0e59221d-0c1d-46ae-9544-c963ce56c10b</target>
    <detailedtargetinfo>
        <id>8:acs:57b9bac9-df6c-4d39-a73b-26e944adf6ea_0e59221d-0c1d-46ae-9544-c963ce56c10b</id>
        <friendlyName>User 1</friendlyName>
    </detailedtargetinfo>
    <target>8:acs:57b9bac9-df6c-4d39-a73b-26e944adf6ea_8540c0de-899f-5cce-acb5-3ec493af3800</target>
    <detailedtargetinfo>
        <id>8:acs:57b9bac9-df6c-4d39-a73b-26e944adf6ea_8540c0de-899f-5cce-acb5-3ec493af3800</id>
        <friendlyName>User 2</friendlyName>
    </detailedtargetinfo>
</addmember>

```  

- `ThreadActivity/DeleteMember`: mensaje del sistema que indica que un miembro se han quitado de la conversación de chat. Por ejemplo:

```xml

<deletemember>
    <eventtime>1598478187642</eventtime>
    <initiator>8:acs:57b9bac9-df6c-4d39-a73b-26e944adf6ea_0e59221d-0c1d-46ae-9544-c963ce56c10b</initiator>
    <detailedinitiatorinfo>
        <friendlyName>User 1</friendlyName>
    </detailedinitiatorinfo>
    <rosterVersion>1598478184564</rosterVersion>
    <target>8:acs:57b9bac9-df6c-4d39-a73b-26e944adf6ea_8540c0de-899f-5cce-acb5-3ec493af3800</target>
    <detailedtargetinfo>
        <id>8:acs:57b9bac9-df6c-4d39-a73b-26e944adf6ea_8540c0de-899f-5cce-acb5-3ec493af3800</id>
        <friendlyName>User 2</friendlyName>
    </detailedtargetinfo>
</deletemember>

```

- `ThreadActivity/TopicUpdate`: mensaje del sistema que indica que el tema se ha actualizado. Por ejemplo:

```xml

<topicupdate>
    <eventtime>1598477591811</eventtime>
    <initiator>8:acs:57b9bac9-df6c-4d39-a73b-26e944adf6ea_0e59221d-0c1d-46ae-9544-c963ce56c10b</initiator>
    <value>New topic</value>
</topicupdate>

```

## <a name="real-time-signaling"></a>Señalización en tiempo real 

La biblioteca cliente de chat para JavaScript incluye señalización en tiempo real. De este modo, los clientes pueden escuchar actualizaciones en tiempo real y mensajes entrantes en una conversación de chat sin tener que sondear las API. Entre los eventos disponibles se incluye:

 - `ChatMessageReceived`: cuando se envía un mensaje nuevo a una conversación de chat a la que pertenece el usuario. Este evento no se envía para los mensajes del sistema generados automáticamente, como se describió en el tema anterior.  
 - `ChatMessageEdited`: cuando se edita un mensaje en una conversación de chat a la que pertenece el usuario. 
 - `ChatMessageDeleted`: cuando se elimina un mensaje en una conversación de chat a la que pertenece el usuario. 
 - `TypingIndicatorReceived`: cuando otro miembro está escribiendo un mensaje en una conversación de chat a la que pertenece el usuario. 
 - `ReadReceiptReceived` : cuando otro miembro ha leído el mensaje que el usuario envió en una conversación de chat. 

## <a name="chat-events"></a>Eventos de chat 

La señalización en tiempo real permite que los usuarios conversen en tiempo real. Los servicios pueden usar Azure Event Grid para suscribirse a los eventos relacionados con el chat. Para obtener más información, consulte los [conceptos del control de eventos](../event-handling.md).

## <a name="using-cognitive-services-with-chat-client-library-to-enable-intelligent-features"></a>Uso de Cognitive Services con la biblioteca cliente de chat para habilitar características inteligentes

Puede usar [Azure Cognitive APIs](https://docs.microsoft.com/azure/cognitive-services/) con la biblioteca cliente de chat para agregar características inteligentes a sus aplicaciones. Por ejemplo, puede:

- Permitir a los usuarios conversar entre sí en distintos idiomas. 
- Ayudar a un agente de soporte técnico a priorizar los vales mediante la detección de una opinión negativa en un nuevo problema de un cliente.
- Analizar los mensajes entrantes para detectar claves reconocer entidades, así como solicitar información pertinente al usuario en la aplicación en función del contenido del mensaje.

Una manera de lograrlo es hacer que su servicio de confianza actúe como miembro de una conversación de chat. Supongamos que desea quiere la traducción de idiomas. Este servicio será responsable de escuchar los mensajes que intercambian otros miembros [1], llamar a Cognitive Services APIs para traducir el contenido al idioma deseado [2, 3] y enviar el resultado traducido como mensaje en la conversación de chat [4]. 

De este modo, el historial de mensajes contendrá los mensajes originales y los traducidos. En la aplicación cliente, puede agregar lógica para mostrar el mensaje original o el traducido. Consulte [esta guía de inicio rápido](https://docs.microsoft.com/azure/cognitive-services/translator/quickstart-translate) para más información sobre cómo usar Cognitive Services APIs para traducir texto a diferentes idiomas. 

:::image type="content" source="../media/chat/cognitive-services.png" alt-text="Diagrama que muestra a Cognitive Services interactuando con Communication Services.":::

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Introducción al chat](../../quickstarts/chat/get-started.md)

Puede que los siguientes documentos le resulten interesantes:

- Familiarícese con nuestra [biblioteca cliente de chat](sdk-features.md)
