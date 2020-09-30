---
title: Flujos de llamadas en Azure Communication Services
titleSuffix: An Azure Communication Services concept document
description: Obtenga información sobre los flujos de llamadas en Azure Communication Services.
author: mikben
manager: jken
services: azure-communication-services
ms.author: mikben
ms.date: 03/10/2020
ms.topic: overview
ms.service: azure-communication-services
ms.openlocfilehash: 7172e3319e60603d46dc2af87f3818a5c3664285
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90945891"
---
# <a name="call-flows"></a>Flujos de llamadas

[!INCLUDE [Public Preview Notice](../includes/public-preview-include.md)]

En la sección siguiente se ofrece información general sobre los flujos de llamadas en Azure Communication Services. Los flujos de señalización y multimedia dependen de los tipos de llamadas que realizan los usuarios. Algunos ejemplos de tipos de llamadas son VoIP de uno a uno, RTC de uno a uno y llamadas de grupo que contienen una combinación de participantes de VoIP y conectados a RTC. Revise [Tipos de llamada](./voice-video-calling/about-call-types.md).

## <a name="about-signaling-and-media-protocols"></a>Acerca de los protocolos de señalización y multimedia

Cuando se establece una llamada de punto a punto o de grupo, se usan dos protocolos en segundo plano: HTTP (REST) para señalización y el SRTP para los flujos multimedia. 

La señalización entre bibliotecas cliente o entre bibliotecas cliente y controladores de señalización de Communication Services se controla con HTTP REST (TLS). Para el tráfico multimedia en tiempo real (RTP), se prefiere el Protocolo de datagramas de usuario (UDP). Si el firewall impide el uso de UDP, la biblioteca cliente usará el protocolo de control de transmisión (TCP) para los flujos multimedia. 

Vamos a revisar los protocolos de señalización y multimedia en varios escenarios. 

## <a name="call-flow-cases"></a>Casos de flujo de llamadas

### <a name="case-1-voip-where-a-direct-connection-between-two-devices-is-possible"></a>Caso 1: VoIP, donde se puede establecer una conexión directa entre dos dispositivos

En las videollamadas o las llamadas VoIP de uno a uno, el tráfico prefiere la ruta de acceso más directa. "Ruta directa" significa que si dos bibliotecas cliente pueden conectarse entre sí directamente, se establecerá una conexión directa. Normalmente, esto es posible cuando dos bibliotecas cliente están en la misma subred (por ejemplo, en una subred 192.168.1.0/24) o cuando los dispositivos están activos en subredes que se pueden ver entre sí (las bibliotecas cliente de las subredes 10.10.0.0/16 y 192.168.1.0/24 pueden comunicarse entre sí).

:::image type="content" source="./media/call-flows/about-voice-case-1.png" alt-text="Diagrama que muestra una llamada VoIP directa entre usuarios y Communication Services.":::

### <a name="case-2-voip-where-a-direct-connection-between-devices-is-not-possible-but-where-connection-between-nat-devices-is-possible"></a>Caso 2: VoIP donde no es posible una conexión directa entre dispositivos, pero sí una conexión entre dispositivos NAT

Si hay dos dispositivos ubicados en subredes que no se pueden comunicar entre sí (por ejemplo, Alice trabaja desde una cafetería y Roberto trabaja desde su oficina doméstica), pero la conexión entre dispositivos NAT es posible, las bibliotecas cliente del cliente establecerán la conectividad a través de los dispositivos NAT. 

En el caso de Alice, será el dispositivo NAT de la cafetería y, para Bob, será el de la oficina doméstica. El dispositivo de Alice enviará la dirección externa de su dispositivo NAT y Bob hará lo mismo. Las bibliotecas cliente aprenden las direcciones externas de un servicio STUN (Session Traversal Utilities for NAT) que Azure Communication Services proporciona de forma gratuita. La lógica que controla el protocolo de enlace entre Alice y Bob se inserta en las bibliotecas cliente que proporciona Azure Communication Services. (No se requiere ninguna configuración adicional)

:::image type="content" source="./media/call-flows/about-voice-case-2.png" alt-text="Diagrama que muestra una llamada VoIP que emplea una conexión STUN.":::

### <a name="case-3-voip-where-neither-a-direct-nor-nat-connection-is-possible"></a>Caso 3: VoIP sin posibilidad de una conexión directa ni NAT

Si uno o ambos dispositivos cliente están detrás de un dispositivo NAT simétrico, se requiere un servicio en la nube independiente para retransmitir contenido multimedia entre las dos bibliotecas cliente. Este servicio se denomina TURN (Traversal Use Relays around NAT) y también lo proporciona Communication Services. La biblioteca cliente de llamadas de Communication Services usa automáticamente los servicios TURN en función de las condiciones de red detectadas. El uso del servicio TURN de Microsoft se cobra por separado.

:::image type="content" source="./media/call-flows/about-voice-case-3.png" alt-text="Diagrama que muestra una llamada VoIP que emplea una conexión TURN.":::
 
### <a name="case-4-group-calls-with-pstn"></a>Caso 4: Llamadas de grupo con RTC

Tanto la señalización como los flujos multimedia para llamadas RTC usan el recurso de telefonía de Azure Communication Services. Este recurso está interconectado con otros operadores.

El tráfico multimedia de RTC fluye a través de un componente denominado procesador de multimedia.

:::image type="content" source="./media/call-flows/about-voice-pstn.png" alt-text="Diagrama que muestra una llamada de grupo RTC con Communication Services.":::

> [!NOTE]
> Para aquellos que estén familiarizados con el procesamiento de multimedia, nuestro procesador de multimedia también es un agente de usuario adosado, tal como se define en [RFC 3261 SIP: Protocolo de inicio de sesión](https://tools.ietf.org/html/rfc3261), lo que significa que puede traducir códecs al controlar las llamadas entre las redes de Microsoft y de operador. El controlador de señalización de Azure Communication Services es la implementación de Microsoft de un proxy SIP por la misma RFC.

En el caso de las llamadas de grupo, los flujos de señalización y multimedia siempre son a través del back-end de Azure Communication Services. El audio o el vídeo de todos los participantes se mezcla en el componente procesador de multimedia. Todos los miembros de una llamada de grupo envían sus secuencias de audio o vídeo al procesador de multimedia, que devuelve transmisiones multimedia combinadas.

El Protocolo en tiempo real (RTP) predeterminado para las llamadas de grupo es el Protocolo de datagramas de usuario (UDP).

> [!NOTE]
> El procesador de multimedia puede actuar como una unidad de control multipunto (MCU) o una unidad de reenvío selectivo (SFU).

:::image type="content" source="./media/call-flows/about-voice-group-calls.png" alt-text="Diagrama que muestra el flujo del proceso multimedia UDP en Communication Services.":::

Si la biblioteca cliente no puede usar UDP para los flujos multimedia debido a restricciones del firewall, se intentará usar el Protocolo de control de transmisión (TCP). Tenga en cuenta que el componente procesador de multimedia requiere UDP, de modo que, si se da este caso, el servicio TURN de Communication Services se agregará a la llamada de grupo para trasladar TCP a UDP. En este caso, se incurrirá en cargos de TURN, menos que se deshabiliten manualmente las capacidades de dicho servicio.

:::image type="content" source="./media/call-flows/about-voice-group-calls-2.png" alt-text="Diagrama que muestra el flujo del proceso de multimedia TCP en Communication Services.":::

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Introducción a las llamadas](../quickstarts/voice-video-calling/getting-started-with-calling.md)

Puede que los siguientes documentos le resulten interesantes:

- Más información sobre los [tipos de llamada](../concepts/voice-video-calling/about-call-types.md)
- Información sobre la [arquitectura de cliente y servidor](./client-and-server-architecture.md)
