---
title: Solución de problemas de pago de Azure
description: Resolución de un problema al actualizar la cuenta de información de pago en el Centro de cuentas o en Microsoft Azure Portal.
author: v-miegge
ms.reviewerr: dcscontentpm
tags: billing
ms.service: cost-management-billing
ms.subservice: billing
ms.topic: troubleshooting
ms.date: 08/20/2020
ms.author: jaserano
ms.openlocfilehash: adc3874ba211de270589d8ea36a04b9431e9e12a
ms.sourcegitcommit: 56cbd6d97cb52e61ceb6d3894abe1977713354d9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88684922"
---
# <a name="troubleshoot-azure-payment-issues"></a>Solución de problemas de pago de Azure

Puede experimentar problemas o errores al intentar actualizar la cuenta de información de pago en Microsoft Azure Portal o en el centro de cuentas de Azure.

Para resolver el problema, seleccione a continuación el tema que más se asemeje al error.

## <a name="my-credit-card-was-declined-when-i-tried-to-sign-up-for-azure"></a>Se ha rechazado mi tarjeta de crédito al intentar suscribirme a Azure

Para solucionar problemas relacionados con el rechazo de la tarjeta de crédito, consulte [Solución de problemas de una tarjeta rechazada en el registro de Azure](troubleshoot-declined-card.md).

## <a name="unable-to-see-subscriptions-under-my-account-to-update-the-payment-method"></a>No se pueden ver las suscripciones de mi cuenta para actualizar el método de pago

Es posible que esté usando un identificador de correo electrónico diferente del que usa en las suscripciones.

Para solucionar este problema, vea [Error de inicio de sesión No se encontraron suscripciones en Azure Portal o en el Centro de cuentas de Azure](no-subscriptions-found.md).

## <a name="unable-to-use-a-virtual-or-prepaid-credit-or-debit-card-as-a-payment-method"></a>No se pueden usar tarjetas de crédito o débito virtuales o de prepago como método de pago.

*   Las tarjetas de crédito virtuales o de prepago no se aceptan como opción de pago para suscripciones de Azure.
*   Las tarjetas de débito no se aceptan como opción de pago para suscripciones de Azure.

Para más información, consulte [Solución de problemas de una tarjeta rechazada en el registro de Azure](troubleshoot-declined-card.md).

## <a name="unable-to-remove-a-credit-card-from-a-saved-billing-payment-method"></a>No se puede eliminar una tarjeta de crédito del método de pago de facturación guardado

Por diseño, una tarjeta de crédito no se puede eliminar de la suscripción activa.

Si una tarjeta existente tiene que eliminarse, debe agregar una nueva tarjeta a la suscripción (para que el instrumento de pago anterior se pueda eliminar correctamente), o bien cancelar la suscripción. Esto hace que la suscripción se elimine de forma permanente y se quite la tarjeta.

## <a name="unable-to-delete-an-old-payment-method-after-adding-a-new-payment-method"></a>No se puede eliminar un método de pago anterior después de agregar el nuevo método de pago

El nuevo instrumento de pago podría no estar asociado con la suscripción. Si necesita ayuda para asociar el instrumento de pago a la suscripción, consulte [Agregar, actualizar o quitar una tarjeta de crédito para Azure](change-credit-card.md).

## <a name="unable-to-delete-a-payment-method-because-of-cannot-delete-payment-method-error"></a>No se puede eliminar un método de pago debido al mensaje de error *No se puede eliminar método de pago*

Esto sucede cuando hay un saldo pendiente. Liquide los saldos pendientes antes de eliminar el método de pago.

## <a name="unable-to-make-payment-for-a-subscription"></a>No se puede realizar el pago de una suscripción

Si recibe los mensajes de error *El pago está vencido. Hay un problema con el método de pago* o *Lo sentimos, no se puede guardar la información. Cierre el explorador y vuelva a intentarlo*, podría deberse a que hay un pago pendiente en la tarjeta porque su entidad financiera denegó la tarjeta.

Confirme que la tarjeta de crédito tiene saldo suficiente para satisfacer un pago. Si no es así, use otra tarjeta para realizar el pago o póngase en contacto con la institución financiera para resolver el problema.

Acuda a su entidad bancaria para comprobar lo siguiente:

- Las transacciones internacionales no están habilitadas.
- La tarjeta tiene un límite de crédito y el saldo debe liquidarse.
- Hay un pago periódico habilitado en la tarjeta.

## <a name="unable-to-change-payment-method-because-of-browser-issues-browser-does-not-respond-does-not-load-and-so-on"></a>El método de pago no se puede cambiar debido a problemas del explorador (no responde, no se carga, etc.)

Cierre todas las sesiones activas de Azure y siga los pasos descritos en el artículo [Explorar con InPrivate en Microsoft Edge](https://support.microsoft.com/help/4026200/microsoft-edge-browse-inprivate) para iniciar una sesión de InPrivate en Microsoft Edge o Internet Explorer.

En la sesión privada, siga lo indicado en [Cambio de la tarjeta de crédito](change-credit-card.md) para actualizar o cambiar la información de la tarjeta de crédito.

También puede probar a hacer lo siguiente:

- Actualizar el explorador
- Usar otro explorador
- Eliminar las cookies almacenadas en caché

## <a name="my-subscription-is-still-disabled-after-updating-the-payment-method"></a>La suscripción sigue deshabilitada después de actualizar el método de pago

Este problema sucede cuando hay un saldo pendiente. Liquide los saldos pendientes antes de eliminar el método de pago.

## <a name="unable-to-change-payment-method-because-of-an-xml-error-response-page"></a>No se puede cambiar el método de pago debido a una página de respuesta de error de XML

Este mensaje aparece si utiliza [Azure Portal](https://portal.azure.com/) para agregar una nueva tarjeta de crédito.

Para agregar los detalles de una tarjeta, inicie sesión en el portal de cuentas de Azure con la dirección de correo electrónico del administrador de la cuenta.

## <a name="additional-help-resources"></a>Más recursos de ayuda

Otros artículos de solución de problemas relativos a la facturación y las suscripciones de Azure

- [Tarjeta rechazada](troubleshoot-declined-card.md)
- [Problemas de inicio de sesión de suscripción](troubleshoot-sign-in-issue.md)
- [No se han encontrado suscripciones](no-subscriptions-found.md)
- [Se ha deshabilitado la vista de costos empresariales](enterprise-mgmt-grp-troubleshoot-cost-view.md)

## <a name="contact-us-for-help"></a>Póngase en contacto con nosotros para obtener ayuda

Si tiene alguna pregunta o necesita ayuda, [cree una solicitud de soporte técnico](https://ms.portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest).

## <a name="next-steps"></a>Pasos siguientes

- [Documentación de Facturación de Azure](../../billing/index.md)
