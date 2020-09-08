---
title: Banca abierta (PSD2) y autenticación segura de clientes (SCA) para los clientes de Azure
description: En este artículo se explica por qué se necesita la autenticación multifactor en algunas compras de Azure y cómo realizar la autenticación.
author: bandersmsft
ms.reviewer: judupont
tags: billing
ms.service: cost-management-billing
ms.subservice: billing
ms.topic: conceptual
ms.date: 08/20/2020
ms.author: banders
ms.openlocfilehash: b4c2db76405e7bad7653c1801eb2bcc5db82fe9b
ms.sourcegitcommit: 56cbd6d97cb52e61ceb6d3894abe1977713354d9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88684803"
---
# <a name="open-banking-psd2-and-strong-customer-authentication-sca-for-azure-customers"></a>Banca abierta (PSD2) y autenticación segura de clientes (SCA) para los clientes de Azure

A partir del 14 de septiembre de 2019, los bancos de 31 países o regiones del [Espacio Económico Europeo](https://en.wikipedia.org/wiki/European_Economic_Area) están obligados a comprobar la identidad de todas las personas que realizan compras en línea antes de procesar el pago. Esta comprobación requiere la autenticación multifactor para ayudar a garantizar que las compras en línea son seguras y están protegidas. La fecha de este requisito de comprobación se retrasará en algunos países y regiones. Para más información, consulte las [P+F de Microsoft sobre PSD2](https://support.microsoft.com/en-us/help/4517854?preview).

## <a name="what-psd2-means-for-azure-customers"></a>Qué significa PSD2 para los clientes de Azure

Si paga por Azure con una tarjeta de crédito que ha emitido un banco del [espacio económico Europao](https://en.wikipedia.org/wiki/European_Economic_Area), podría estar obligado a realizar la autenticación multifactor para el método de pago de su cuenta. Puede que se le pida que lleve a cabo el desafío de autenticación multifactor al registrar o actualizar su cuenta de Azure, incluso si no realiza una compra en ese momento. También es posible que se le pida que proporcione autenticación multifactor al cambiar el método de pago de su cuenta de Azure, quitar el límite de gasto o realizar un pago inmediato desde Azure Portal, como la liquidación de saldos pendientes o la compra de créditos de Azure.

Si su banco rechaza los cargos mensuales de Azure, recibirá un correo electrónico de vencimiento de Azure con instrucciones para solucionarlo. Puede realizar el desafío de autenticación multifactor y liquidar los cargos pendientes en Azure Portal.

## <a name="complete-multi-factor-authentication-in-the-azure-portal"></a>Realización de la autenticación multifactor en Azure Portal

En las secciones siguientes se describe cómo realizar la autenticación multifactor en Azure Portal. Use la información que le sea aplicable.

### <a name="change-the-active-payment-method-of-your-azure-account"></a>Cambio del método de pago activo de la cuenta de Azure

Puede cambiar el método de pago activo de la cuenta de Azure mediante estos pasos:

1. Inicie sesión en [Azure Portal](https://portal.azure.com) como administrador de la cuenta y vaya a **Administración de costos + facturación**.
2. En la página **Información general**, seleccione la suscripción correspondiente en la cuadrícula **Mis suscripciones**.
3. En "Facturación", seleccione **Métodos de pago**. Puede agregar una nueva tarjeta de crédito o establecer una tarjeta existente como método de pago activo de la suscripción. Si su banco exige la autenticación multifactor, se le pedirá que realice un desafío de autenticación durante el proceso.

Para más información, consulte [Agregar, actualizar o quitar una tarjeta de crédito para Azure](change-credit-card.md).

### <a name="settle-outstanding-charges-for-azure-services"></a>Liquidación de los cargos pendientes de los servicios de Azure

Si el banco rechaza los cargos, el estado de la cuenta de Azure cambiará a **Vencida** en Azure Portal. Puede comprobar el estado de su cuenta mediante estos pasos:

1. Inicie sesión en [Azure Portal](https://portal.azure.com/) como administrador de la cuenta.
2. Busque en **Administración de costos + facturación**.
3. En la página **Información general** de **Administración de costos + facturación**, revise la columna de estado de la cuadrícula **Mis suscripciones**.
4. Si en la suscripción pone **Vencida**, haga clic en **Liquidar saldo**. Durante el proceso, se le pedirá que realice la autenticación multifactor.

### <a name="settle-outstanding-charges-for-marketplace-and-reservation-purchases"></a>Liquidación de cargos pendientes de compras en Marketplace y de reservas

Las compras en Marketplace y de reservas se facturan de forma independiente de los servicios de Azure. Si el banco rechaza los cargos de Marketplace o de reservas, la factura aparecerá como vencida y verá la opción de **Pagar ahora** en Azure Portal. Puede pagar las facturas vencidas de Marketplace y de reservas mediante estos pasos:

1. Inicie sesión en [Azure Portal](https://portal.azure.com/) como administrador de la cuenta.
2. Busque en **Administración de costos + facturación**.
3. En "Facturación", seleccione **Facturas**.
5. En el filtro desplegable de suscripción, seleccione la suscripción asociada a la compra de Marketplace o de reserva.
6. En la cuadrícula de facturas, revise la columna de tipo. Si el tipo es **Azure Marketplace y Reservations**, verá un vínculo **Pagar ahora** si la factura está vencida. Si no ve **Pagar ahora**, significa que ya se ha pagado la factura. Durante el proceso Pagar ahora, se le pedirá que realice la autenticación multifactor.

## <a name="next-steps"></a>Pasos siguientes
- Consulte [Resolución del saldo de la suscripción de Azure vencido](resolve-past-due-balance.md) si necesita pagar una factura de Azure.
