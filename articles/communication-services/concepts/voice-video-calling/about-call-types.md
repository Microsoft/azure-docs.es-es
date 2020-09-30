---
title: Conceptos de voz y vídeo en Azure Communication Services
titleSuffix: An Azure Communication Services concept document
description: Obtenga información sobre los tipos de llamada de Communication Services.
author: mikben
manager: jken
services: azure-communication-services
ms.author: mikben
ms.date: 03/10/2020
ms.topic: overview
ms.service: azure-communication-services
ms.openlocfilehash: 932b9f84e9f17555f364ae504967678ee1d7ce97
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90945298"
---
# <a name="voice-and-video-concepts"></a>Conceptos de voz y vídeo

[!INCLUDE [Public Preview Notice](../../includes/public-preview-include.md)]

Puede usar Azure Communication Services para hacer y recibir llamadas de voz y vídeo, analizar metadatos de las llamadas, registrar llamadas e, incluso, crear árboles de llamadas. Las llamadas se pueden hacer a otros dispositivos conectados a Internet y en teléfonos convencionales. Puede usar las bibliotecas cliente JavaScript, Android o iOS de Communication Services para compilar aplicaciones que permitan a los usuarios hablar entre sí en conversaciones privadas o en discusiones grupales. Azure Communication Services admite hacer y recibir llamadas desde servicios o bots.

## <a name="call-types-in-azure-communication-services"></a>Tipos de llamada en Azure Communication Services

Hay varios tipos de llamadas que puede hacer en Azure Communication Services. El tipo de llamadas que haga determinará el esquema de señales, los flujos de tráfico multimedia y el modelo de precios.

### <a name="voice-over-ip-voip"></a>Llamada de voz sobre IP (VoIP) 

Cuando un usuario de la aplicación llama a otro usuario de la aplicación a través de una conexión de datos o Internet, la llamada se hace a través de voz sobre IP (VoIP). En este caso, las señales y los elementos multimedia fluyen a través de Internet.

### <a name="public-switched-telephone-network-pstn"></a>Red telefónica conmutada (RTC)

Cada vez que los usuarios interactúan con un número de teléfono tradicional, las llamadas se facilitan mediante llamadas de voz de RTC (red telefónica conmutada). Para hacer y recibir llamadas RTC, tiene que agregar funcionalidades de telefonía a su recurso de Azure Communication Services. En este caso, las señales y los elementos multimedia usan una combinación de tecnologías basadas en IP y en RTC para conectar a los usuarios.

### <a name="one-to-one-call"></a>Llamadas de uno a uno

Una llamada uno a uno en Azure Communication Services se produce cuando uno de los usuarios se conecta con otro usuario a través de una de nuestras bibliotecas cliente. La llamada puede ser VoIP o RTC.

### <a name="group-call"></a>Llamada grupal

Una llamada grupal en Azure Communication Services se produce cuando tres o más participantes se conectan entre sí. Cualquier combinación de usuarios con conexión VoIP o RTC puede estar presente en una llamada grupal. Una llamada de uno a uno se puede convertir en una llamada grupal si se agregan más participantes a la llamada. Uno de estos participantes puede ser un bot.

### <a name="supported-video-standards"></a>Estándares de vídeo compatibles
Se admite H.264 (MPEG-4) 

### <a name="video-quality"></a>Calidad de vídeo
En los SDK nativos (iOS y Android), se admite hasta Full HD 1080p. Para el SDK web (JS), se admite el Standard HD 720p. La calidad depende del ancho de banda disponible.  

### <a name="rooms-concept"></a>Concepto de salas
Las salas son un conjunto de API y SDK que permiten agregar fácilmente interacciones de audio, vídeo, uso compartido de pantalla, RTC y SMS a su sitio web o aplicación nativa.
Durante la vista previa, puede usar el identificador de grupo para unirse a la misma conversación. Puede crear tantos id. de grupo como necesite y separar los usuarios por "salas". Al avanzar, se introducirán más controles relativos a las "salas". 

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Introducción a las llamadas](../../quickstarts/voice-video-calling/getting-started-with-calling.md)

Para más información, consulte los siguientes artículos.
- Familiarización con los [flujos de llamada](../call-flows.md) generales
- [Planeación de una solución RTC](../telephony-sms/plan-solution.md)
- Más información sobre las [funcionalidades de las bibliotecas cliente de llamadas](../voice-video-calling/calling-sdk-features.md)
