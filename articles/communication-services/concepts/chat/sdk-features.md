---
title: Información general de la biblioteca cliente de chat para Azure Communication Services
titleSuffix: An Azure Communication Services concept document
description: Obtenga información sobre la biblioteca cliente de chat de Azure Communication Services.
author: mikben
manager: jken
services: azure-communication-services
ms.author: mikben
ms.date: 03/10/2020
ms.topic: overview
ms.service: azure-communication-services
ms.openlocfilehash: fd62a7fed0c6dec818bf4e2558945fb24c9d5261
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90945482"
---
# <a name="chat-client-library-overview"></a>Información general de la biblioteca cliente de chat

[!INCLUDE [Public Preview Notice](../../includes/public-preview-include.md)]

Las bibliotecas cliente de chat de Azure Communication Services se pueden usar para agregar chats enriquecidos en tiempo real a las aplicaciones.

## <a name="chat-client-library-capabilities"></a>Funcionalidades de la biblioteca cliente de chat

En la lista siguiente se presenta el conjunto de características que están disponibles actualmente en las bibliotecas cliente de chat de Communication Services.

| Grupo de características | Capacidad                                                                                                          | JS  | Java | .NET | Python |
| ----------------- | ------------------------------------------------------------------------------------------------------------------- | --- | ----- | ---- | -----  |
| Funcionalidades principales | Crear una conversación de chat entre 2 o más usuarios (hasta 250 usuarios)                                                       | ✔️   | ✔️  | ✔️    | ✔️   |
|                   | Actualizar el tema de una conversación de chat                                                                              | ✔️   | ✔️ | ✔️    | ✔️   |
|                   | Agrear o quitar miembros de una conversación de chat                                                                           | ✔️   | ✔️  | ✔️    | ✔️  |
|                   | Optar por compartir el historial de mensajes del chat con miembros recién agregados (*todos/ninguno/hasta cierta fecha*) | ✔️   | ✔️   | ✔️    | ✔️  |
|                   | Obtener una lista de todos los miembros de la conversación de chat                                                                          | ✔️   | ✔️  | ✔️ | ✔️ |
|                   | Eliminar una conversación de chat                                                                                              | ✔️   | ✔️  | ✔️    | ✔️  |
|                   | Obtener una lista de las conversaciones de chat a las que pertenece un usuario                                                                  | ✔️   | ✔️  | ✔️    | ✔️  |
|                   | Obtener información sobre una conversación de chat determinada                                                                              | ✔️   | ✔️  | ✔️ | ✔️ |
|                   | Enviar y recibir mensajes en una conversación de chat                                                                            | ✔️   | ✔️   | ✔️    | ✔️  |
|                   | Editar el contenido de un mensaje después de enviarlo                                                                   | ✔️   | ✔️  | ✔️ | ✔️ |
|                   | Eliminar un mensaje                                                                                                       | ✔️   | ✔️  | ✔️ | ✔️ |
|                   | Etiquetar un mensaje con prioridad normal o alta en el momento del envío                                               | ✔️   | ✔️  | ✔️    | ✔️   |
|                   | Enviar y recibir confirmaciones de lectura para los mensajes que han leído los miembros <br/> *No está disponible cuando hay más de 20 miembros en una conversación de chat*    | ✔️   | ✔️  | ✔️    | ✔️   |
|                   | Enviar y recibir notificaciones de escritura cuando un miembro está escribiendo un mensaje en una conversación de chat <br/> *No está disponible cuando hay más de 20 miembros en una conversación de chat*      | ✔️   | ✔️   | ✔️    | ✔️    |
|                   | Obtener todos los mensajes de una conversación de chat <br/> *Se admiten los emojis Unicode*                                                  | ✔️   | ✔️  | ✔️    | ✔️  |
|                   | Enviar emojis como parte del contenido del mensaje                                                                              | ✔️   | ✔️  | ✔️    | ✔️  |
|Señalización en tiempo real (habilitada por un paquete de señalización propietario)| Recibir notificaciones cuando el usuario recibe un mensaje nuevo en una conversación de chat a la que pertenece                                     | ✔️   | ❌    | ❌  | ❌  |
|                    | Recibir notificaciones cuando otro usuario de una conversación de chat a la que pertenece editó un mensaje                | ✔️   | ❌    | ❌    | ❌  |
|                    | Recibir notificaciones cuando otro usuario de una conversación de chat a la que pertenece eliminó un mensaje                | ✔️   | ❌    | ❌    | ❌  |
|                    | Recibir una notificación cuando otro miembro de la conversación de chat esté escribiendo                                                             | ✔️   | ❌    | ❌    | ❌  |
|                    | Recibir una notificación cuando otro miembro ha leído un mensaje (confirmación de lectura) en la conversación de chat                               | ✔️   | ❌    | ❌    | ❌  |
| Events             | Usar Event Grid para suscribirse a la actividad de usuario que se produce en las conversaciones de chat e integrar servicios de notificación personalizados o lógica de negocios     | ✔️   | ✔️  | ✔️    | ✔️  |
| Supervisión        | Supervisar el uso en términos de los mensajes enviados                                                                               | ✔️   | ✔️  | ✔️    | ✔️  |
|                    | Supervisar la calidad y el estado de las solicitudes de API realizadas por la aplicación y configurar las alertas a través del portal                                                          | ✔️   | ✔️  | ✔️    | ✔️  |
|Características adicionales | Usar [Cognitive Services APIs](https://docs.microsoft.com/azure/cognitive-services/) junto con la biblioteca cliente de chat para habilitar las características inteligentes de *traducción de idioma y análisis de opinión de los mensajes entrantes en un cliente, conversión de voz en texto para redactar un mensaje mientras un miembro habla, etc.*                                                                                         | ✔️   | ✔️  | ✔️    | ✔️  |

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Introducción al chat](../../quickstarts/chat/get-started.md)

Puede que los siguientes documentos le resulten interesantes:

- Familiarícese con los [conceptos del chat](../chat/concepts.md)
