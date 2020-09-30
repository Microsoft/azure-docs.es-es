---
title: Escenarios de precios para llamadas (voz o vídeo) y chat
titleSuffix: An Azure Communication Services concept document
description: Obtenga información sobre el modelo de precios de Communication Services.
author: mikben
manager: jken
services: azure-communication-services
ms.author: mikben
ms.date: 03/10/2020
ms.topic: overview
ms.service: azure-communication-services
ms.openlocfilehash: 94a34937af92eefa6146c7a043b6d5d68cf6b852
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90945339"
---
# <a name="pricing-scenarios"></a>Escenarios de precios

[!INCLUDE [Public Preview Notice](../includes/public-preview-include.md)]

<!--
> [!WARNING]
> This document is under construction and needs the following items to be addressed:
> - Looks like other resources point to a /pricing page that is managed by Commerce or Marketing? https://azure.microsoft.com/pricing/details/functions/ Should we? FOLLOWING UP WITH KRISTIN TO FIND THE RIGHT ACS PAGE
-->

Los precios de Azure Communication Services se basan en un modelo de pago por uso sin tarifas iniciales. Solo se le facturará por su consumo y uso de los servicios.

## <a name="voicevideo-calling-and-screen-sharing"></a>Llamadas de voz o vídeo y uso compartido de pantalla

Azure Communication Services permite agregar llamadas de voz o vídeo a las aplicaciones, así como uso compartido de pantalla. Puede insertar la experiencia en sus aplicaciones mediante las bibliotecas cliente de JavaScript, Objective-C (Apple), Java (Android) o .NET. Consulte nuestra [lista completa de bibliotecas cliente disponibles](./sdk-options.md).

### <a name="pricing"></a>Precios

Los servicios de llamadas y uso compartido de pantalla se cobran por minuto a cada participante con un costo de 0,004 USD por participante por minuto para las llamadas de grupo. Para más información sobre los distintos flujos de llamada posibles, consulte [esta página](./call-flows.md).

Cada participante de la llamada se contará en la facturación por cada minuto que esté conectado a la llamada. Este principio se aplica independientemente de si el usuario está realizando llamadas por vídeo o voz, o si está compartiendo su pantalla.

### <a name="pricing-example-group-audiovideo-call-using-js-and-ios-client-libraries"></a>Ejemplo de precios: Llamada grupal de audio o vídeo mediante las bibliotecas cliente de JS y iOS

Alice realizó una llamada grupal con sus compañeros de trabajo, Bob y Charlie. Alice y Bob usaron las bibliotecas cliente de JS, y Charlie las bibliotecas cliente de iOS.

- La llamada duró un total de 60 minutos.
- Alice y Bob participaron durante la llamada completa. Alicia activó su vídeo durante cinco minutos y compartió su pantalla durante 23 minutos. Bob mantuvo encendido el vídeo durante la llamada completa (60 minutos) y compartió su pantalla durante 12 minutos.
- Charlie abandonó la llamada después de 43 minutos. Charlie usó audio y vídeo durante el tiempo que participó (43 minutos).

**Cálculos de costos**

- 2 participantes x 60 minutos x 0,004 USD por participante por minuto = 0,48 USD [el vídeo y el audio se cobran a la misma tarifa]
- 1 participante x 43 minutos x 0,004 USD por participante por minuto = 0,172 USD [el vídeo y el audio se cobran a la misma tarifa]

**Costo total de la llamada grupal**: 0,48 USD + 0,172 USD = 0,652 USD

## <a name="chat"></a>Chat

Puede usar Communication Services para mejorar su aplicación con la capacidad de enviar y recibir mensajes de chat entre 2 o más usuarios. Hay bibliotecas cliente de chat disponibles para JavaScript, .NET, Python y Java. Consulte [esta página para obtener información acerca de las bibliotecas cliente](./sdk-options.md)

### <a name="price"></a>Price

- Se le cobrarán 0,0008 USD por cada mensaje de chat enviado.

### <a name="pricing-example-chat-between-two-users"></a>Ejemplo de precios: Chat entre dos usuarios 

Geeta inicia una conversación de chat con Emily para compartir una actualización y le envía cinco mensajes. La conversación dura 10 minutos, en los que Geeta y Emily envían otros 15 mensajes cada una.

**Cálculos de costos** 
- Número de mensajes enviados (5 + 15 + 15) x 0,0008 USD = 0,028 USD

### <a name="pricing-example-group-chat-with-multiple-users"></a>Ejemplo de precios: Chat grupal con varios usuarios 

Charlie inicia una conversación de chat con sus amigos Casey y Jasmine para planear unas vacaciones. Conversan por unos momentos, en los que Charlie, Casey y Jasmine envían 20, 30 y 18 mensajes, respectivamente. Saben que quizá a su amiga Rose también le interese unirse al viaje, de modo que la agregan a la conversación de chat y comparten todo el historial de mensajes con ella. 

Rose ve los mensajes y comienza a responder. Mientras tanto, Casey recibe una llamada y decide ponerse al día con la conversación más adelante. Charlie, Jasmine y Rose deciden las fechas del viaje y envían otros 30, 25 y 35 mensajes, respectivamente.

**Cálculos de costos** 

- Número de mensajes enviados (20 + 30 + 18 + 30 + 25 + 35) x 0,0008 USD = 0,1264 USD
