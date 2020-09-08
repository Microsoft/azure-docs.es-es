---
title: Autoservicio de cambios y reembolsos de reservas de Azure
description: Obtenga información sobre cómo puede obtener cambios o reembolsos para las reservas de Azure. Para cambiar o recibir el reembolso de las reservas, debe tener acceso de propietario para el pedido de reserva.
author: yashesvi
ms.service: cost-management-billing
ms.subservice: reservations
ms.topic: how-to
ms.date: 07/24/2020
ms.author: banders
ms.openlocfilehash: 553652832f3f4ce84a0a141455551e4977215184
ms.sourcegitcommit: 56cbd6d97cb52e61ceb6d3894abe1977713354d9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88684429"
---
# <a name="self-service-exchanges-and-refunds-for-azure-reservations"></a>Autoservicio de cambios y reembolsos de reservas de Azure

Las reservas de Azure proporcionan flexibilidad para ayudar a satisfacer sus necesidades en constante cambio. Puede intercambiar una reserva por otra del mismo tipo. Por ejemplo, puede intercambiar una reserva de máquina virtual para adquirir otra reserva para cualquier otro tamaño de VM o región. Del mismo modo, se puede cambiar una reserva de base de datos PaaS de SQL para adquirir otra reserva de cualquier tipo o región de base de datos PaaS de SQL. También puede reembolsar las reservas, pero la suma total de todos los compromisos de reserva cancelados en el ámbito de facturación (por ejemplo, Contrato Enterprise, Contrato de cliente de Microsoft y Contrato de partner de Microsoft) no puede superar 50 000 USD en un período acumulado de 12 meses. La capacidad reservada de Azure Databricks, la solución de VMware de Azure por reserva de CloudSimple, la reserva de Azure Red Hat Open Shift, los planes de Red Hat y los planes de SUSE Linux no son aptos para devoluciones.

El autoservicio de intercambio y la funcionalidad de cancelación no están disponible para los clientes de Contrato Enterprise de US Government. Otros tipos de suscripción de US Government, como Pago por uso y Proveedor de soluciones en la nube, sí se admiten.

> [!NOTE]
> - **Para cambiar o recibir el reembolso de una reserva existente, debe tener acceso de propietario en el pedido de reserva**. Puede [agregar o cambiar los usuarios que pueden administrar una reserva](https://docs.microsoft.com/azure/cost-management-billing/reservations/manage-reserved-vm-instance#add-or-change-users-who-can-manage-a-reservation).
> - Actualmente, Microsoft no cobra ningún cargo por finalización prematura en las devoluciones de las reservas. En el futuro es posible que se cobre cuando se realicen devoluciones. En la actualidad, no hay una fecha establecida para habilitar este cargo.

## <a name="how-to-exchange-or-refund-an-existing-reservation"></a>Cambio o devolución de una reserva existente

Cualquier reserva se puede cambiar desde [Azure Portal](https://portal.azure.com/#blade/Microsoft_Azure_Reservations/ReservationsBrowseBlade).

1. Seleccione las reservas para las que quiera obtener un reembolso y seleccione **Intercambiar**.  
    [![Imagen de ejemplo que muestra las reservas para devolver](./media/exchange-and-refund-azure-reservations/exchange-refund-return.png)](./media/exchange-and-refund-azure-reservations/exchange-refund-return.png#lightbox)
1. Seleccione el producto de máquina virtual que quiere comprar y escriba la cantidad. Asegúrese de que el nuevo total de compra es mayor que el total de devolución. [Determine el tamaño adecuado antes de la compra](../../virtual-machines/windows/prepay-reserved-vm-instances.md#determine-the-right-vm-size-before-you-buy).  
    [![Imagen de ejemplo que muestra el producto máquina virtual para comprar con un cambio](./media/exchange-and-refund-azure-reservations/exchange-refund-select-purchase.png)](./media/exchange-and-refund-azure-reservations/exchange-refund-select-purchase.png#lightbox)
1. Revise y finalice la transacción.  
    [![Imagen de ejemplo que muestra el producto máquina virtual para comprar con un cambio, completando la devolución](./media/exchange-and-refund-azure-reservations/exchange-refund-confirm-exchange.png)](./media/exchange-and-refund-azure-reservations/exchange-refund-confirm-exchange.png#lightbox)

Para obtener el reembolso de una reserva, vaya a **Detalles de la reserva** y haga clic en **Reembolso**.

## <a name="exchange-non-premium-storage-for-premium-storage"></a>Intercambio de almacenamiento no premium por almacenamiento premium

Puede intercambiar una reserva comprada para un tamaño de máquina virtual que no es compatible con el almacenamiento premium por un tamaño de máquina virtual que sí lo es. Por ejemplo, un _F1_ por un _F1s_. Para hacer el intercambio, vaya a los detalles de la reserva y seleccione **Intercambiar**. El intercambio no restablece el plazo de la instancia reservada ni crea una nueva transacción. 

## <a name="how-transactions-are-processed"></a>Procesamiento de las transacciones

En primer lugar, Microsoft cancela la reserva existente y reembolsa la cantidad proporcional para esa reserva. Si hay un cambio, se procesa la compra nueva. Microsoft procesa los reembolsos mediante uno de los métodos siguientes, según el tipo de cuenta y el método de pago:

### <a name="enterprise-agreement-customers"></a>Clientes con contrato Enterprise

Se agrega el dinero al compromiso monetario para los cambios y devoluciones en caso de que se utilizara alguno en la compra original. Si el término de compromiso monetario que se compró con la reserva ya no está activo, se agregará crédito al término de compromiso monetario de su contrato Entreprise actual. El crédito es válido durante 90 días a partir de la fecha de reembolso. El crédito no utilizado expira una vez transcurridos los 90 días.

Si la compra original se realizó como uso por encima del límite, tanto la factura original en la que se compró la reserva como todas las facturas posteriores se vuelven a abrir y a reajustar. Microsoft emite un nota de crédito cuando se realiza una devolución.

### <a name="pay-as-you-go-invoice-payments-and-csp-program"></a>Pagos mediante factura de pago por uso y programa CSP

La factura de compra de la reserva original se cancela y luego se crea una nueva factura para el reembolso. En el caso de los cambios, la nueva factura muestra el reembolso y la nueva compra. El importe del reembolso se ajusta con la compra. Si solo reembolsó una reserva, Microsoft se queda con la cantidad prorrateada y se ajusta a una futura compra de reserva. Si adquirió una reserva con la tarifa de pago por uso y posteriormente cambia a la tarifa CSP, la reserva se puede devolver y volver a comprar sin penalización.

### <a name="pay-as-you-go-credit-card-customers"></a>Clientes con tarjeta de crédito de pago por uso

La factura original se cancela y se crea una nueva factura. El dinero se devuelve a la tarjeta de crédito que se utilizó para la compra original. Si ha cambiado la tarjeta, [póngase en contacto con el soporte técnico](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest).

## <a name="cancel-exchange-and-refund-policies"></a>Directivas de cancelación, intercambio y reembolso

Azure tiene estas directivas en caso de cancelaciones, intercambios y reembolsos.

**Directivas de intercambio**

- Puede devolver varias reservas existentes para comprar una sola reserva nueva del mismo tipo. No puede intercambiar las reservas de un tipo por otro. Por ejemplo, no puede devolver una reserva de máquina virtual para comprar una reserva de SQL. Puede cambiar una propiedad de la reserva, como la familia, la serie, la versión, la SKU, la región, la cantidad y el plazo, con un intercambio.
- Solo los propietarios de una reserva pueden procesar un intercambio. [Obtenga información sobre cómo agregar o cambiar los usuarios que pueden administrar una reserva](manage-reserved-vm-instance.md#add-or-change-users-who-can-manage-a-reservation).
- Un cambio se procesa como una devolución y una nueva compra: se crean diferentes transacciones para la cancelación y la nueva compra de la reserva. El importe prorrateado de la reserva se reembolsa por las reservas que se canjeen. Se le cobrará por completo la nueva compra. El importe de la reserva prorrateada es el valor residual prorrateado diario de la reserva que se devuelve.
- Puede intercambiar o reembolsar reservas incluso si el contrato Enterprise utilizado para comprar la reserva ha expirado y se ha renovado como un nuevo contrato.
- El compromiso de duración de la nueva reserva debe ser igual o mayor que el compromiso restante de la reserva devuelta. Ejemplo: en el caso de una reserva de tres años de 100 USD al mes que se haya cambiado después de haber realizado 18 pagos, el compromiso de duración de la nueva reserva debe ser, como mínimo, de 1800 USD (con pago mensual o por adelantado).
- La nueva reserva comprada en el marco del intercambio tiene un nuevo plazo a partir del momento del intercambio.
- No hay ninguna penalización ni límites anuales para los intercambios.

**Directivas de reembolso**

- Actualmente, no cobramos ningún cargo por finalización prematura, pero en el futuro podría haber una cargo del 12 % por este motivo en las cancelaciones.
- El compromiso total cancelado no puede superar los 50 000 USD en un período acumulado de 12 meses para un perfil de facturación o una inscripción única. Por ejemplo, en el caso de una reserva trienal de 100 USD al mes que se ha devuelto el decimoctavo mes, el compromiso cancelado es de 1800 USD. Después de la devolución, el nuevo límite disponible para reembolsos será 48 200 dólares. En 365 días a partir de la devolución, el límite de 48 200 USD aumentará en 1800 USD, hasta 50 000 USD. Cualquier otra cancelación de reserva para el perfil de facturación o la inscripción de Contrato Enterprise reducirá este importe, y se aplicará la misma lógica de reposición.
- Azure no procesará ningún reembolso que supere el límite de 50 000 USD en un período de 12 meses para un perfil de facturación o inscripción de Contrato Enterprise.
- Los reembolsos se calculan según el precio más bajo de su precio de compra o en el precio actual de la reserva.
- Solo los propietarios de un pedido de reserva pueden procesar un reembolso. [Obtenga información sobre cómo agregar o cambiar los usuarios que pueden administrar una reserva](manage-reserved-vm-instance.md#add-or-change-users-who-can-manage-a-reservation).

## <a name="need-help-contact-us"></a>¿Necesita ayuda? Póngase en contacto con nosotros.

Si tiene alguna pregunta o necesita ayuda, [cree una solicitud de soporte técnico](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest).

## <a name="next-steps"></a>Pasos siguientes

- Para más información sobre cómo administrar una reserva, consulte [Administración de Azure Reservations](manage-reserved-vm-instance.md).
- Para obtener más información acerca de Azure Reservations, consulte los siguientes artículos:
    - [¿Qué es Azure Reservations?](save-compute-costs-reservations.md)
    - [Administración de reservas en Azure](manage-reserved-vm-instance.md)
    - [Información sobre cómo se aplica el descuento por la reserva](../manage/understand-vm-reservation-charges.md)
    - [Información sobre el uso de reservas para suscripciones de pago por uso](understand-reserved-instance-usage.md)
    - [Información sobre el uso de reservas para la inscripción Enterprise](understand-reserved-instance-usage-ea.md)
    - [Costos de software de Windows no incluidos con reservas](reserved-instance-windows-software-costs.md)
    - [Azure Reservations en el programa CSP](/partner-center/azure-reservations)
