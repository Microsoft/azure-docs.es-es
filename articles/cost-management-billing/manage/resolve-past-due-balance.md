---
title: Correo electrónico de saldo vencido de Azure
description: Describe cómo realizar el pago si la suscripción de Azure tiene un saldo pendiente
author: genlin
ms.reviewer: dcscontentpm
tags: billing
ms.service: cost-management-billing
ms.subservice: billing
ms.topic: how-to
ms.date: 08/20/2020
ms.author: banders
ms.openlocfilehash: 8c792ae1b6ac43ead15fd4f27d2468c517153354
ms.sourcegitcommit: 56cbd6d97cb52e61ceb6d3894abe1977713354d9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88684820"
---
# <a name="resolve-past-due-balance-for-your-azure-subscription"></a>Saldo de la suscripción de Azure vencido

Este artículo se aplica a los clientes que se hayan suscrito a Azure Online con una tarjeta de crédito y que tengan una cuenta de facturación de Microsoft Online Services Program. Aprenda a [comprobar el tipo de cuenta de facturación](#check-the-type-of-your-account). Si tiene una cuenta de facturación de Contrato de cliente de Microsoft, consulte [Cómo pagar la factura de Microsoft Azure](../understand/pay-bill.md) en su lugar.

Si no se recibe su pago o no puede procesarse, es posible que reciba un correo electrónico o vea una alerta en Azure Portal que indica que la suscripción está vencida. Si el método de pago predeterminado es una tarjeta de crédito, el [administrador de cuenta](billing-subscription-transfer.md#whoisaa) puede liquidar los cargos pendientes en Azure Portal. Si paga mediante factura (cheque o transferencia bancaria), envíe el pago a la ubicación indicada en la parte inferior de la factura.

> [!IMPORTANT]
> * Si tiene varias suscripciones que utilizan la misma tarjeta de crédito y todas están vencidas, deberá pagar todo el saldo pendiente a la vez.
> * La tarjeta de crédito que utilice para liquidar los cargos pendientes se convertirá en el nuevo método de pago predeterminado para todas las suscripciones que estaban usando el método de pago que produjo los errores.

## <a name="resolve-past-due-balance-in-the-azure-portal"></a>Resolución del saldo pendiente pasado en Azure Portal

1. Inicie sesión en [Azure Portal](https://portal.azure.com) como administrador de la cuenta.
1. Busque **Administración de costos + facturación**.
1. Seleccione la suscripción vencida en la página **Información general**.
1. En la página de **información general de la suscripción**, haga clic en el banner rojo del vencimiento para liquidar el saldo.
    > [!NOTE]
    > Si no es el administrador de la cuenta, no podrá liquidar el saldo.
1. En la nueva página **Liquidar saldo**, haga clic en **Seleccionar método de pago**.
    ![Captura de pantalla que muestra el enlace Seleccionar método de pago](./media/resolve-past-due-balance/settle-balance-screen.png)

1. En la nueva hoja de la derecha, seleccione una tarjeta en la lista desplegable o agregue una nueva haciendo clic en el vínculo azul **Add new payment method** (Agregar nuevo método de pago). Esta tarjeta de crédito se convertirá en el método de pago activo para todas las suscripciones que actualmente usan el método de pago con errores.
     > [!NOTE]
     > * El saldo total pendiente refleja los cargos pendientes de todos los servicios de Microsoft que utilizaban el método de pago que produjo los errores.
     > * Si el método de pago seleccionado también tiene cargos pendientes de servicios de Microsoft, esto se reflejará en el saldo pendiente total. Deberá pagar esos cargos pendientes también.
1. Haga clic en **Pagar**.

## <a name="troubleshoot-declined-credit-card"></a>Solución del problema de tarjeta de crédito rechazada

Si su entidad financiera rechaza el cargo en la tarjeta de crédito, póngase en contacto con ella para resolver la incidencia. Cuando lo haga compruebe que:
- Las transacciones internacionales están habilitadas en la tarjeta.
- La tarjeta tiene suficiente límite de crédito o fondos para liquidar el saldo.
- Están habilitados los pagos periódicos en la tarjeta.

## <a name="not-getting-billing-email-notifications"></a>¿No se reciben notificaciones de correo electrónico de facturación?

Si es el administrador de la cuenta, [compruebe qué dirección de correo electrónico se utiliza para las notificaciones](change-azure-account-profile.md). Se recomienda que use una dirección de correo electrónico que consulte periódicamente. Si el correo electrónico es el adecuado, compruebe la carpeta de correo basura.

## <a name="if-i-forget-to-pay-what-happens"></a>¿Qué ocurre si se olvida de pagar?

Se cancelará el servicio y los recursos ya no estarán disponibles. Los datos de Azure se eliminarán a los 90 días de terminarse el servicio. Para más información, consulte el [Microsoft Trust Center: How we manage your data](https://go.microsoft.com/fwLink/p/?LinkID=822930&clcid=0x409) (Centro de confianza de Microsoft: Administración de los datos).

Si tiene la certeza de que se ha procesado el pago pero la suscripción sigue deshabilitada, póngase en contacto con el [Soporte técnico de Azure](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade).

## <a name="check-the-type-of-your-account"></a>Comprobación del tipo de la cuenta
[!INCLUDE [billing-check-mca](../../../includes/billing-check-account-type.md)]

## <a name="need-help-contact-us"></a>¿Necesita ayuda? Póngase en contacto con nosotros.

Si tiene alguna pregunta o necesita ayuda, [cree una solicitud de soporte técnico](https://go.microsoft.com/fwlink/?linkid=2083458).
