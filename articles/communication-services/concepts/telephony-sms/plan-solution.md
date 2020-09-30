---
title: Planeación de la solución de telefonía y SMS de Azure Communication Services
titleSuffix: An Azure Communication Services concept document
description: Obtenga información sobre cómo planear de forma eficaz el uso de números de teléfono y telefonía.
author: stkozak
manager: rampras
services: azure-communication-services
ms.author: stkozak
ms.date: 06/23/2020
ms.topic: overview
ms.service: azure-communication-services
ms.openlocfilehash: 39f88ab8b735438f60d8e20513ea5cbda43d41ee
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90945314"
---
# <a name="plan-your-telephony-and-sms-solution"></a>Planeación de la solución de telefonía y SMS

[!INCLUDE [Private Preview Notice](../../includes/private-preview-include.md)]

En este documento se describen los distintos planes de telefonía y tipos de número que ofrece Azure Communication Services. Le guiaremos a través de los flujos de decisión para ayudarle a seleccionar al proveedor de servicios de voz, los tipos de números de teléfono, los planes y las funcionalidades disponibles a través de Communication Services.

## <a name="about-phone-numbers-in-azure-communications-services"></a>Acerca de los números de teléfono en Azure Communication Services

Azure Communication Services le permite usar números de teléfono para realizar y recibir llamadas de telefonía y mensajes SMS. Estos números de teléfono se pueden usar para configurar el id. de llamada en las llamadas salientes que realiza su servicio.
  
Si no tiene un número de teléfono actual para importarlo en la solución de Communication Services, la manera más sencilla de empezar es obtener un nuevo número de teléfono de Azure Communication Services en cuestión de minutos.

Si ya tiene un número de teléfono que desea seguir usando en la solución (por ejemplo, 1-800-EMPRESA), puede cambiar el número de teléfono del proveedor actual a Communication Services.

En el siguiente diagrama se explica cómo navegar por las opciones disponibles:

:::image type="content" source="../media/plan-solution/decision-tree-basic.png" alt-text="Diagrama que muestra cómo tomar una decisión relacionada con los números de teléfono.":::

Ahora, vamos a revisar los tipos de números de teléfono y las funcionalidades disponibles a través de Communication Services. 

## <a name="microsoft-direct-offer-of-phone-numbers-and-capabilities"></a>Oferta directa de Microsoft de números de teléfono y funcionalidades

Azure Communication Services proporciona una flexibilidad excelente para los desarrolladores. En la mayoría de los números de teléfono, puede configurar un conjunto de planes "a la carta". Algunos desarrolladores solo necesitan un plan de llamadas entrantes, mientras otros optan por los planes de llamadas entrantes y SMS salientes. Estos planes se pueden seleccionar a medida que alquile o cambie sus números de teléfono en Communication Services.

Los planes disponibles dependen del país y el tipo de número de teléfono con el que esté trabajando. El siguiente diagrama representa el flujo de decisiones:    Los planes disponibles dependen del país y el tipo de número de teléfono con el que esté trabajando.

<!-- Tami/team have rejected this multiple times despite updates, says it needs to be higher res - need to work with her to get approval for this image. Commenting out to move our staging forward. :::image type="content" source="../../media/example-decision-flow.png" alt-text="Example for the decision flow"::: -->

Antes de seleccionar un tipo de número de teléfono, revisemos el plan internacional de numeración de teléfonos.

### <a name="optional-reading-international-public-telecommunication-numbering-plan-e164"></a>Lectura opcional. Plan internacional de numeración de telecomunicaciones públicas (E.164)

> [!NOTE]
> Se recomienda revisar esta información incluso si está familiarizado con el plan de numeración de teléfonos E.164 para comprender mejor los tipos de número y las funcionalidades que ofrece la oferta directa de Azure Communication Services.

El plan internacional de numeración de telecomunicaciones públicas está definido en la recomendación E.164 de la Unión Internacional de Telecomunicaciones (ITU). Los números que se ajustan a la norma se limitan a un máximo de 15 dígitos.

El número de teléfono consta de lo siguiente

-   Un prefijo "+"
-   Un prefijo de marcado internacional o un código de país o región (uno, dos o tres dígitos) 
-   *(Opcional)* Un plan de numeración o indicativo nacional de destino, normalmente denominado código de área. La longitud de este código depende del país. En Estados Unidos, es de tres dígitos. En Australia y Nueva Zelanda, es un dígito. En Alemania, Japón, México y otros países los códigos de área tienen longitudes variables. Por ejemplo, en Alemania el código de área puede tener entre dos y cinco dígitos, mientras que en Japón puede tener entre uno y cinco dígitos.
-   Un número de abonado

> [!NOTE]
> La clasificación anterior no se ajusta totalmente al estándar ITU E.164 y está pensado como una descripción simplificada. Por ejemplo, el número de abonado está subdividido en el estándar. Si le interesa obtener más información sobre el plan internacional de numeración, el [estándar ITU E.164](https://www.itu.int/rec/T-REC-E.164) es un lugar excelente para comenzar.  

Estos son algunos ejemplos que le ayudarán a comprender mejor el plan de numeración:

Un número de teléfono regional en Estados Unidos:

:::image type="content" source="../media/plan-solution/regional-us.png" alt-text="Ejemplo de un número de teléfono regional en Estados Unidos":::

Un número de teléfono regional en Canadá:

:::image type="content" source="../media/plan-solution/regional-canada.png" alt-text="Ejemplo de un número de teléfono regional en Canadá":::

Un número gratuito en la región de Norteamérica:

:::image type="content" source="../media/plan-solution/tollfree-us.png" alt-text="Ejemplo de un número gratuito en Norteamérica":::

Un número de teléfono móvil en Reino Unido:

:::image type="content" source="../media/plan-solution/mobile-uk.png" alt-text="Ejemplo de un número de teléfono móvil en Reino Unido":::

A continuación, revisaremos los tipos de números de teléfono específicos disponibles en Azure Communication Services.

## <a name="phone-number-types-in-azure-communication-services"></a>Tipos de número de teléfono en Azure Communication Services

Microsoft ofrece planes regionales y gratuitos para los códigos cortos y números móviles en los países donde corresponde.

En la tabla siguiente se resumen estos tipos de números de teléfono: 

| Tipo de número de teléfono | Ejemplo                              | Disponibilidad de país    | Caso de uso común                                                                                                     |
| ----------------- | ------------------------------------ | ----------------------- | ------------------------------------------------------------------------------------------------------------------- |
| Regional          | \+ 1 (código de área geográfica) XXX XX XX  | Estados Unidos, Canadá Puerto Rico | Asignar números de teléfono a los usuarios de las aplicaciones, o asignarlos a sistemas o bots de respuesta interactiva de voz (IVR). |
| Gratuito         | \+ 1 (*código* de área gratuito) XXX XX XX | Estados Unidos, Canadá Puerto Rico | Asignar números a sistemas o bots de respuesta interactiva de voz (IVR), aplicaciones de SMS.                                        |

## <a name="plans"></a>Planes 

Echemos un vistazo a las funcionalidades que puede habilitar para sus números de teléfono. Estas funcionalidades varían según el país debido a los requisitos normativos. Azure Communication Services ofrece las siguientes funcionalidades:

- **SMS saliente unidireccional**, que resulta útil para escenarios de autenticación en dos fases y de notificación.
- **SMS saliente y entrante bidireccional**, un paquete predefinido que permite enviar y recibir SMS como parte de un plan.
- **Llamada RTC**, puede seleccionar una llamada entrante y usar el id. de llamada para realizar llamadas salientes.

## <a name="next-steps"></a>Pasos siguientes

### <a name="quickstarts"></a>Guías de inicio rápido

- [Obtención de un número de teléfono](../../quickstarts/telephony-sms/get-phone-number.md)
- [Realización de una llamada](../../quickstarts/voice-video-calling/calling-client-samples.md)
- [Enviar un mensaje de texto.](../../quickstarts/telephony-sms/send.md)

### <a name="conceptual-documentation"></a>Documentación conceptual

- [Conceptos de voz y vídeo](../voice-video-calling/about-call-types.md)
- [Flujos de llamadas y SMS](../call-flows.md)
- [Precios](../pricing.md)
