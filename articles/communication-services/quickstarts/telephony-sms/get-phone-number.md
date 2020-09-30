---
title: 'Inicio rápido: obtención de un número de teléfono de Azure Communication Services'
description: Obtenga información sobre cómo comprar un número de teléfono de Communication Services mediante Azure Portal.
author: ddematheu2
manager: nimag
services: azure-communication-services
ms.author: dademath
ms.date: 07/09/2020
ms.topic: quickstart
ms.service: azure-communication-services
ms.custom: references_regions
ms.openlocfilehash: 2743dc0164f604c9c5e033aacc3e58fae42a1fd2
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90946051"
---
# <a name="quickstart-get-a-phone-number-using-the-azure-portal"></a>Inicio rápido: Obtención de un número de teléfono mediante Azure Portal

[!INCLUDE [Private Preview Notice](../../includes/private-preview-include.md)]

Comience a usar Azure Communication Services mediante Azure Portal para comprar un número de teléfono.

## <a name="prerequisites"></a>Requisitos previos

- Una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
- [Recurso activo de Communication Services.](../create-communication-resource.md)

## <a name="get-a-phone-number"></a>Obtención de un número de teléfono

Para empezar a aprovisionar números, vaya al recurso de Communication Services en [Azure Portal](https://portal.azure.com).

:::image type="content" source="../media/manage-phone-azure-portal-start.png" alt-text="Captura de pantalla que muestra la página principal de un recurso de Communication Services.":::

### <a name="getting-new-phone-numbers"></a>Obtención de nuevos números de teléfono

Vaya a la hoja números de teléfono en el menú de recursos.

:::image type="content" source="../media/manage-phone-azure-portal-phone-page.png" alt-text="Captura de pantalla que muestra la página de teléfonos de un recurso de Communication Services.":::

Presione el botón `Get` para iniciar el asistente. El asistente de la hoja de `Phone numbers` le guiará a través de una serie de preguntas que le ayudarán a elegir el número de teléfono que mejor se adapte a su escenario. 

En primer lugar, deberá elegir el `Country/region` en el que desea aprovisionar el número de teléfono. Después de seleccionar el país o la región, deberá seleccionar el `phone plan` que mejor se adapte a sus necesidades. 

:::image type="content" source="../media/manage-phone-azure-portal-get-numbers.png" alt-text="Captura de pantalla que muestra la vista para obtener números de teléfono.":::

### <a name="select-a-phone-plan"></a>Selección de un plan de teléfono

La selección de un plan de teléfono se divide en dos pasos: 

1. La selección del tipo del [número de teléfono](../../concepts/telephony-sms/plan-solution.md#phone-number-types-in-azure-communication-services)
2. La selección del [plan](../../concepts/telephony-sms/plan-solution.md#plans)

Actualmente ofrecemos dos tipos de números: `Geographic` y `Toll-free`. Después de seleccionar un tipo de número, se le ofrecerán varios planes entre los que puede elegir.

> [!NOTE]
> Actualmente solo se admite la selección de números de teléfono con llamadas entrantes o salientes. No obstante, puede comprar un número de teléfono con una llamada entrante habilitada y, a continuación, configurar el id. del autor de llamada saliente para que coincida con el número habilitado para la llamada de entrada (lo que verán los usuarios al llamarlos desde la aplicación de Communication Services).
> Esto solo se aplica a las llamadas bidireccionales. SMS bidireccional se admite de forma nativa.

En nuestro ejemplo, hemos elegido un tipo de número `Toll-free` con el plan `Outbound calling`.

:::image type="content" source="../media/manage-phone-azure-portal-select-plans.png" alt-text="Captura de pantalla que muestra la vista para seleccionar planes.":::

### <a name="declare-purpose"></a>Declaración de propósito

A continuación, el asistente le preguntará el propósito de uso del número. Recopilamos esta información para aplicar las regulaciones correctas de los impuestos y las llamadas de emergencia.

:::image type="content" source="../media/quickstart-search-and-acquire-bot-or-human.png" alt-text="Captura de pantalla que muestra la selección de un bot o un usuario en el proceso de adquisición del número de teléfono.":::

Desde aquí, haga clic en el botón `Next: Numbers` en la parte inferior de la página para personalizar los números de teléfono que le gustaría aprovisionar.

### <a name="customizing-phone-numbers"></a>Personalización de los números de teléfono

En la página `Numbers`, personalizará los números de teléfono que le gustaría aprovisionar.

:::image type="content" source="../media/manage-phone-azure-portal-select-numbers-start.png" alt-text="Captura de pantalla que muestra la página para la selección de números.":::

> [!NOTE]
> En esta guía de inicio rápido se muestra el flujo de personalización del tipo de número `Toll-free`. La experiencia puede ser ligeramente diferente si ha elegido el tipo de número `Geographic`, pero el resultado final será el mismo.

Elija el `Area code` en la lista de códigos de área disponibles e introduzca la cantidad que desea aprovisionar y, a continuación, haga clic en `Search` para buscar números que cumplan los requisitos seleccionados. Los números de teléfono que satisfagan sus necesidades se mostrarán junto con el costo mensual.

:::image type="content" source="../media/manage-phone-azure-portal-found-numbers.png" alt-text="Captura de pantalla que muestra la página para la selección de números con números reservados.":::

> [!NOTE]
> La disponibilidad depende del tipo de número, la ubicación y el plan que haya seleccionado.
> Los números se reservan durante un breve período de tiempo antes de que expire la transacción. Si la transacción expira, tendrá que volver a seleccionar los números.

Para ver el resumen de la compra y realizar el pedido, haga clic en el botón `Next: Summary` en la parte inferior de la página.

### <a name="place-order"></a>Hacer pedido

En la página de resumen se revisará el tipo de número, las características, los números de teléfono y el costo mensual total para aprovisionar los números de teléfono.

> [!NOTE]
> Los precios que se muestran son los **cargos periódicos mensuales** que cubren el costo por conceder el número de teléfono seleccionado. **Los costos de pago por uso**, que se incurren al realizar o recibir llamadas, no se incluyen en esta vista. Los costos de lista están [disponibles aquí](../../concepts/pricing.md). Estos costos dependen del tipo de número y de la ubicación del número a llamar. Por ejemplo, precio por minuto para una llamada de un número regional de Seattle a un número regional de Nueva York y una llamada del mismo número a un número de teléfono móvil del Reino Unido puede ser diferente.

Finalmente, haga clic en `Place order` situado en la parte inferior de la página para confirmar.

:::image type="content" source="../media/manage-phone-azure-portal-get-numbers-summary.png" alt-text="Captura de pantalla que muestra la página de resumen con el tipo de número, las características, los números de teléfono y el costo mensual total.":::

## <a name="find-your-phone-numbers-on-the-azure-portal"></a>Busque los números de teléfono en Azure Portal

En [Azure Portal](https://portal.azure.com), navegue hasta el recurso de comunicación de Azure:

:::image type="content" source="../media/manage-phone-azure-portal-start.png" alt-text="Captura de pantalla que muestra la página principal de un recurso de Communication Services.":::

Seleccione la pestaña de Números de teléfono en el menú para administrarlos.

:::image type="content" source="../media/manage-phone-azure-portal-phones.png" alt-text="Captura de pantalla que muestra la página de números de teléfono de un recurso de Communication Services.":::

> [!NOTE]
> Los números aprovisionados pueden tardar unos minutos en mostrarse en esta página.

## <a name="troubleshooting"></a>Solución de problemas

Preguntas y problemas comunes:

- En este momento, solo Estados Unidos y Canadá admiten la compra de números de teléfono. Esto se basa en la dirección de facturación de la suscripción a la que está asociado el recurso. En este momento, no se puede trasladar el recurso a otra suscripción.

- Si se elimina un número de teléfono, este no se liberará ni podrá volver a comprarse hasta el final del período de facturación.

- Si se elimina un recurso de Communication Services, los números de teléfono asociados a ese recurso se liberarán automáticamente al mismo tiempo.

## <a name="next-steps"></a>Pasos siguientes

En esta guía de inicio rápido, ha aprendido a hacer lo siguiente:

> [!div class="checklist"]
> * Comprar un número de teléfono
> * Administrar el número de teléfono
> * Liberar un número de teléfono

> [!div class="nextstepaction"]
> [Enviar un SMS](../telephony-sms/send.md)
> [Introducción a las llamadas](../voice-video-calling/getting-started-with-calling.md)
