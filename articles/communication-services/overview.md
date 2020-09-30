---
title: ¿Qué es Azure Communication Services?
description: Obtenga información sobre cómo Azure Communication Services le ayuda a desarrollar experiencias de usuario enriquecidas con comunicaciones en tiempo real.
author: mikben
manager: jken
services: azure-communication-services
ms.author: mikben
ms.date: 07/20/2020
ms.topic: overview
ms.service: azure-communication-services
ms.openlocfilehash: a006ece278e4ec750040bde5bd4b7a6144c9e720
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90946308"
---
# <a name="what-is-azure-communication-services"></a>¿Qué es Azure Communication Services?

[!INCLUDE [Public Preview Notice](./includes/public-preview-include.md)]

Azure Communication Services le permite agregar fácilmente características de comunicación multimedia de voz, vídeo y telefonía sobre IP en tiempo real a sus aplicaciones. Las bibliotecas de cliente de Communication Services también permiten agregar la funcionalidad de chat y SMS a las soluciones de comunicación.

Puede usar Communication Services para la comunicación de voz, vídeo, texto y datos en diversos escenarios:

- Comunicación de explorador a explorador, de explorador a aplicación y de aplicación a aplicación
- Usuarios que interactúan con bots u otros servicios
- Usuarios y bots que interactúan sobre la red de telefonía pública conmutada

Se admiten escenarios mixtos. Por ejemplo, una aplicación de Communication Services puede hacer que los usuarios hablen desde los exploradores y los dispositivos de telefonía tradicionales al mismo tiempo. Communication Services también se puede combinar con Azure Bot Service para compilar sistemas de respuesta interactiva de voz (IVR) basados en bot.

## <a name="common-scenarios"></a>Escenarios frecuentes

Los siguientes recursos son un buen punto de partida si no está familiarizado con Azure Communication Services:
<br>

| Recurso                               |Descripción                           |
|---                                    |---                                   |
|**[Creación de un recurso de Communication Services](./quickstarts/create-communication-resource.md)**|Puede empezar a usar Azure Communication Services mediante el Azure Portal o la biblioteca de cliente de administración de Communication Services para aprovisionar su primer recurso de Communication Services. Una vez que tenga la cadena de conexión del recurso de Communication Services, puede aprovisionar los primeros tokens de acceso de usuario.|
|**[Creación del primer token de acceso de usuario](./quickstarts/access-tokens.md)**|Los tokens de acceso de usuario se usan para autenticar los servicios en el recurso de Azure Communication Services. Estos tokens se aprovisionan y se remiten mediante la biblioteca de cliente de administración de Communication Services.|
|**[Obtención de un número de teléfono](./quickstarts/telephony-sms/get-phone-number.md)**|Puede usar Azure Communication Services para aprovisionar y liberar números de teléfono. Estos números de teléfono se pueden usar para iniciar llamadas salientes y compilar soluciones de comunicaciones en SMS.|
|**[Envío de un SMS desde la aplicación](./quickstarts/telephony-sms/send.md)**|La biblioteca de cliente de SMS de Azure Communication Services permite enviar y recibir mensajes SMS desde las aplicaciones .NET y JavaScript.|
|**[Introducción a las llamadas de voz y vídeo](./quickstarts/voice-video-calling/getting-started-with-calling.md)**| Azure Communication Services permite agregar llamadas de voz y vídeo a las aplicaciones mediante la biblioteca cliente de llamadas. Esta biblioteca se basa en WebRTC y le permite establecer comunicaciones de punto a punto, multimedia y en tiempo real dentro de sus aplicaciones.|
|**[Introducción al chat](./quickstarts/chat/get-started.md)**|Las bibliotecas cliente de chat de Azure Communication Services se pueden usar para integrar chats en tiempo real a las aplicaciones.|


## <a name="samples"></a>Ejemplos

En los siguientes ejemplos se muestra el uso de un extremo a otro de las bibliotecas de cliente de Azure Communication Services. No dude en usar estos ejemplos para arrancar sus propias soluciones de Communication Services.
<br>

| Nombre del ejemplo                               | Descripción                           |
|---                                    |---                                   |
|**[Muestra de elementos principales de una llamada grupal](./samples/calling-hero-sample.md)**|Aprenda cómo se pueden usar las bibliotecas de cliente de Communication Services para crear una experiencia de llamada grupal.|
|**[Muestra de elementos principales de un chat grupal](./samples/chat-hero-sample.md)**|Aprenda cómo se pueden usar las bibliotecas de cliente de Communication Services para crear una experiencia de chat grupal.|


## <a name="platforms-and-client-libraries"></a>Bibliotecas de cliente y de plataforma

Los siguientes recursos le ayudarán a obtener información acerca de las bibliotecas de cliente de Azure Communication Services:

| Recurso                               | Descripción                           |
|---                                    |---                                   |
|**[Bibliotecas de cliente y API de REST](./concepts/sdk-options.md)**|Las capacidades de Azure Communication Services están organizadas conceptualmente en seis áreas, cada una representada por una biblioteca cliente. Puede decidir qué bibliotecas de cliente usar en función de sus necesidades de comunicación en tiempo real.|
|**[Información general de la biblioteca cliente de llamadas](./concepts/voice-video-calling/calling-sdk-features.md)**|Revisión de la información general de la biblioteca cliente de llamadas de Communication Services.|
|**[Información general de la biblioteca de cliente de chat](./concepts/chat/sdk-features.md)**|Revisión de la información general de la biblioteca cliente de chat de Communication Services.|
|**[Información general de la biblioteca de cliente de SMS](./concepts/telephony-sms/sdk-features.md)**|Revisión de la información general de la biblioteca cliente de SMS de Communication Services.|

## <a name="compare-azure-communication-services"></a>Comparación de Azure Communication Services

Hay otros dos productos de comunicación de Microsoft que puede considerar aprovechar y que no pueden interactuar directamente con Communication Services en este momento:

 - [Las API de comunicación de la nube de Microsoft Graph](https://docs.microsoft.com/graph/cloud-communications-concept-overview) permiten a las organizaciones crear experiencias de comunicación asociadas a usuarios de Azure Active Directory con licencias de M365. Esto es idóneo para las aplicaciones asociadas a Azure Active Directory o en contextos de ampliación de la experiencia de productividad en Microsoft Teams. También hay API para compilar aplicaciones y personalización dentro de la [experiencia de equipos.](https://docs.microsoft.com/microsoftteams/platform/?view=msteams-client-js-latest&preserve-view=true)

 - [Azure PlayFab Party](https://docs.microsoft.com/gaming/playfab/features/multiplayer/networking/) simplifica la incorporación de comunicación de datos y chat de baja latencia a los juegos. Aunque puede potenciar el chat de juegos y los sistemas de red con Communication Services, PlayFab es una opción personalizada y gratuita de Xbox.


## <a name="next-steps"></a>Pasos siguientes

 - [Creación de un recurso de Communication Services](./quickstarts/create-communication-resource.md)
