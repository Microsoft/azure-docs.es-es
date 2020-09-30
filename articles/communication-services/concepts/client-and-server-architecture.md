---
title: Arquitectura de cliente y servidor
titleSuffix: An Azure Communication Services concept document
description: Obtenga información sobre la arquitectura de Communication Services.
author: mikben
manager: mikben
services: azure-communication-services
ms.author: mikben
ms.date: 03/10/2020
ms.topic: conceptual
ms.service: azure-communication-services
ms.openlocfilehash: 38d6b131c972b4fd890af53624fdd6a36b3ecb6c
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90932039"
---
# <a name="client-and-server-architecture"></a>Arquitectura de cliente y servidor

[!INCLUDE [Public Preview Notice](../includes/public-preview-include.md)]

<!--
> [!WARNING]
> This document is under construction and needs the following items to be addressed: 
> - Need to add security best practices for token management here
> - Reference docs:
> - https://docs.microsoft.com/windows/security/threat-protection/security-policy-settings/create-a-token-object
> - https://docs.microsoft.com/azure/aks/operator-best-practices-identity
> - https://docs.microsoft.com/cloud-app-security/api-tokens?view=gestures-1.0-->

Cada aplicación de Azure Communication Services tendrá **aplicaciones cliente** que usan **servicios** para facilitar la conectividad de persona a persona. En esta página se muestran los elementos de la arquitectura comunes en diversos escenarios.

## <a name="user-access-management"></a>Administración de acceso de usuario

Las bibliotecas cliente de Azure Communication Services requieren `user access tokens` para acceder a los recursos de Communication Services de forma segura. `User access tokens` debe generarse y administrarse mediante un servicio de confianza debido a la naturaleza confidencial del token y la cadena de conexión necesaria para generarlos. Si no se administran correctamente los tokens de acceso, pueden producirse cargos adicionales debido a un uso incorrecto de los recursos. Se recomienda encarecidamente que use un servicio de confianza para la administración de usuarios. El servicio de confianza generará los tokens y los devolverá al cliente mediante el cifrado adecuado. A continuación se muestra un flujo de arquitectura de ejemplo:

:::image type="content" source="../media/scenarios/archdiagram-access.png" alt-text="Diagrama que muestra la arquitectura de token de acceso de usuario.":::

Para consultar información adicional, revise los [procedimientos recomendados de administración de identidades](https://docs.microsoft.com/azure/security/fundamentals/identity-management-best-practices)

## <a name="browser-communication"></a>Comunicación del explorador

Las bibliotecas cliente de JavaScript de Azure Communication Services pueden habilitar aplicaciones web con interacción de texto enriquecido, voz y vídeo. La aplicación interactúa directamente con Azure Communication Services a través de la biblioteca cliente para acceder al plano de datos y ofrecer comunicación de texto, voz y vídeo en tiempo real. A continuación se muestra un flujo de arquitectura de ejemplo:

:::image type="content" source="../media/scenarios/archdiagram-browser.png" alt-text="Diagrama que muestra la arquitectura de explorador a explorador para Communication Services.":::

## <a name="native-app-communication"></a>Comunicación de aplicaciones nativas

Muchos escenarios se atienden mejor con las aplicaciones nativas. Azure Communication Services admite tanto la comunicación de explorador a aplicación como de aplicación a aplicación.  Al crear una experiencia de aplicación nativa, tener notificaciones de inserción permitirá a los usuarios recibir llamadas incluso cuando la aplicación no se esté ejecutando. Azure Communication Services facilita esta tarea gracias a las notificaciones de inserción integradas en Google Firebase, Apple Push Notification Service y notificaciones de inserción de Windows. A continuación se muestra un flujo de arquitectura de ejemplo:

:::image type="content" source="../media/scenarios/archdiagram-app.png" alt-text="Diagrama que muestra la arquitectura de Communication Services para la comunicación de aplicaciones nativas.":::

## <a name="voice-and-sms-over-the-public-switched-telephony-network-pstn"></a>Voz y SMS a través de la red de telefonía pública conmutada (RTC)

La comunicación a través del sistema telefónico puede aumentar drásticamente el alcance de la aplicación. Para admitir escenarios de voz y SMS de RTC, Azure Communication Services le ayuda a [adquirir números de teléfono](../quickstarts/telephony-sms/get-phone-number.md) directamente desde Azure Portal o mediante las API REST y las bibliotecas cliente. Una vez que se adquieren los números de teléfono, se pueden usar para llegar a los clientes mediante llamadas RTC y SMS tanto en escenarios de entrada como de salida. A continuación se muestra un flujo de arquitectura de ejemplo:

> [!Note]
> Durante la versión preliminar pública, el aprovisionamiento de los números de teléfono de EE. UU. está disponible para los clientes con direcciones de facturación ubicadas en Estados Unidos y Canadá. 

:::image type="content" source="../media/scenarios/archdiagram-pstn.png" alt-text="Diagrama que muestra la arquitectura RTC de Communication Services.":::

Para obtener más información sobre las soluciones de RTC y SMS, consulte [Planeación de la solución de RTC y SMS](../concepts/telephony-sms/plan-solution.md).

## <a name="humans-communicating-with-bots-and-other-services"></a>Personas que se comunican con bots y otros servicios

Azure Communication Services admite la comunicación de usuario a sistema a través de canales de texto y voz, con servicios que acceden directamente al plano de datos de Azure Communication Services. Por ejemplo, puede hacer que un bot conteste las llamadas telefónicas entrantes o participe en un chat web. Azure Communication Services proporciona bibliotecas cliente que habilitan estos escenarios para llamar y chatear. A continuación se muestra un flujo de arquitectura de ejemplo:

:::image type="content" source="../media/scenarios/archdiagram-bot.png" alt-text="Diagrama que muestra la arquitectura de bot de Communication Services.":::

## <a name="networking"></a>Redes

Es posible que desee intercambiar datos arbitrarios entre usuarios, por ejemplo, para sincronizar una realidad mixta compartida o una experiencia de juegos. El plano de datos en tiempo real que se usa para la comunicación de texto, voz y vídeo está disponible directamente de dos maneras:

- **Biblioteca cliente de llamada**: los dispositivos de una llamada tienen acceso a las API para enviar y recibir datos a través del canal de llamada. Esta es la manera más fácil de agregar la comunicación de datos a una interacción existente.
- **STUN/TURN**: Azure Communication Services pone servicios STUN y TURN que cumplen con las normativas a su disposición. Esto le permite crear una capa de transporte muy personalizada sobre estas primitivas normalizadas. Puede crear su propio cliente compatible con las normas o usar bibliotecas de código abierto, como [WinRTC](https://github.com/microsoft/winrtc).

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Creación de tokens de acceso de usuario](../quickstarts/access-tokens.md)

Para más información, consulte los siguientes artículos.

- Más información sobre la [autenticación](../concepts/authentication.md).
- Más información sobre las [soluciones de RTC y SMS](../concepts/telephony-sms/plan-solution.md).

- [Incorporación de chat a una aplicación](../quickstarts/chat/get-started.md)
- [Adición de la llamada de voz a una aplicación](../quickstarts/voice-video-calling/getting-started-with-calling.md)
