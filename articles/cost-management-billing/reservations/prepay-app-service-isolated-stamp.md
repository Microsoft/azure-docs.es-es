---
title: Ahorros para Azure App Service con capacidad reservada
description: Aprenda a ahorrar costos en el impuesto sobre el timbre de Azure App Service aislado con capacidad reservada.
author: yashesvi
ms.reviewer: yashar
ms.service: cost-management-billing
ms.topic: how-to
ms.date: 07/24/2020
ms.author: banders
ms.openlocfilehash: c90824ed0119378c2939c4b5edf9dd5b8c31b8c0
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87287613"
---
# <a name="save-costs-for-azure-app-service-isolated-stamp-fee-with-reserved-capacity"></a>Ahorro de costos en el impuesto sobre el timbre de Azure App Service aislado con capacidad reservada

Para ahorrar dinero en los impuestos sobre el timbre de Azure App Service aislado, puede confirmar una reserva para el uso que hará del timbre durante un plazo de tres años. Para comprar la capacidad reservada del impuesto sobre el timbre en entorno aislado, debe elegir la región de Azure en la que se implementará el timbre y el número de timbres que quiere comprar.

Cuando compra una reserva, el uso del impuesto sobre el timbre en entorno aislado que coincide con los atributos de reserva deja de pagarse según las tarifas de pago por uso. La reserva se aplica automáticamente al número de timbres en entornos aislados que coinciden con el ámbito y la región de la capacidad reservada. No es necesario que asigne una reserva a un timbre aislado. La reserva no se aplica a los trabajos, por lo que los demás recursos asociados con el timbre se cobran por separado.

Cuando expira la capacidad reservada, los timbres en entornos aislados siguen ejecutándose, pero se facturan a la tarifa de pago por uso. Las reservas no se renuevan automáticamente.

## <a name="determine-the-right-reservation-to-purchase"></a>Determinación de la reserva de compra adecuada

Al comprar una reserva, confirma el uso que hará de las cantidades reservadas durante los tres años siguientes. Compruebe los datos de uso para determinar cuántos timbres de App Service aislado usa de forma coherente y es posible que use en un futuro.

Además, asegúrese de comprender cómo el timbre en entorno aislado emite el medidor de Linux o de Windows.

- De manera predeterminada, un timbre en entorno aislado vacío emite el medidor de timbres de Windows. Por ejemplo, sin trabajos implementados. Si los trabajos de Windows se implementan en el timbre, seguirá emitiendo este medidor.
- Si implementa un trabajo de Linux, el medidor cambia al medidor de timbres de Linux.
- El timbre emite el medidor de Windows en los casos en que se implementan trabajos de Windows y Linux.

Por este motivo, el medidor de timbres puede cambiar entre Windows y Linux durante la vigencia del timbre.

Compre reservas de timbres de Windows si tiene uno o más trabajos de Windows en el timbre. El único caso en el que debe comprar una reserva de timbres de Linux es si planea tener _solo_ trabajos de Linux en el timbre.

## <a name="buy-isolated-stamp-reserved-capacity"></a>Compra de la capacidad reservada de timbres en entornos aislados

Puede comprar la capacidad reservada de timbres en entornos aislados en [Azure Portal](https://portal.azure.com/#blade/Microsoft_Azure_Reservations/CreateBlade/referrer/documentation/filters/%7B%22reservedResourceType%22%3A%22AppService%22%7D). Pague la reserva [por adelantado o mensualmente](monthly-payments-reservations.md). Para comprar la capacidad reservada, debe tener el rol de propietario al menos en una suscripción Enterprise o individual con precios de pago por uso.

- En el caso de las suscripciones Enterprise, la opción **Agregar instancias reservadas** debe estar habilitada en el [portal de EA](https://ea.azure.com/). O bien, si esa opción está deshabilitada, debe ser un administrador del Contrato Enterprise.
- En el caso del programa del Proveedor de soluciones en la nube (CSP), los únicos que pueden comprar la capacidad reservada de Azure Synapse Analytics son los agentes de administración o de ventas.

**Para comprar:**

1. Vaya a [Azure Portal](https://portal.azure.com/#blade/Microsoft_Azure_Reservations/CreateBlade/referrer/documentation/filters/%7B%22reservedResourceType%22%3A%22AppService%22%7D).
1. Seleccione una suscripción. Utilice la lista **Suscripción** para elegir la que se va a usar para pagar la capacidad reservada. Los costos de la capacidad reservada se cobran en el método de pago de la suscripción. El tipo de suscripción debe ser Contrato Enterprise (números de oferta: MS-AZR-0017P o MS-AZR-0148P) o Pago por uso (números de oferta: MS-AZR-0003P o MS-AZR-0023P) o una suscripción a CSP.
    - Para una suscripción Enterprise, los cargos se deducen del saldo de compromiso monetario de la inscripción o se cobran como uso por encima del límite.
    - Para la suscripción Pago por uso, los cargos se cobran en el método de pago de tarjeta de crédito o factura de la suscripción.
1. Utilice un **ámbito** para elegir el ámbito de la suscripción.
    - **Single resource group scope** (Ámbito de grupo de recursos único): aplica el descuento por reserva a los recursos coincidentes solo en el grupo de recursos seleccionado.
    - **Single subscription scope** (Ámbito de suscripción única): aplica el descuento por reserva a los recursos coincidentes de la suscripción seleccionada.
    - **Ámbito compartido**: aplica el descuento por reserva a los recursos coincidentes en suscripciones aptas que están en el contexto de facturación. Para los clientes con Contrato Enterprise, el contexto de facturación es la inscripción. En el caso de suscripciones individuales con tarifas de pago por uso, el ámbito de facturación son todas las suscripciones aptas creadas por el administrador de la cuenta.
1. Seleccione una **región** para elegir una región de Azure que abarque la capacidad reservada y agregue la reserva al carro.
1. Seleccione un tipo de plan aislado y, luego, haga clic en **Seleccionar**.  
    ![Ejemplo ](./media/prepay-app-service-isolated-stamp/app-service-isolated-stamp-select.png)
1. Escriba la cantidad de timbres de App Service aislado que quiera reservar. Por ejemplo, una cantidad de tres le proporcionaría tres timbres reservados por región. Haga clic en **Siguiente: Review + Buy** (Revisar y comprar).
1. Revise la información y haga clic en **Comprar ahora**.

Una vez termine la compra, vaya a [Reservas](https://portal.azure.com/#blade/Microsoft_Azure_Reservations/ReservationsBrowseBlade) para ver el estado de la compra y supervisarlo en cualquier momento.

## <a name="cancel-exchange-or-refund-reservations"></a>Cancelación, intercambio o reembolso de reservas

Puede cancelar, intercambiar o reembolsar reservas con ciertas limitaciones. Para más información, consulte [Autoservicio de intercambios y reembolsos de reservas de Azure](exchange-and-refund-azure-reservations.md).

## <a name="discount-application-shown-in-usage-data"></a>Cómo se muestra la aplicación del descuento en los datos de uso

Los datos de uso que obtienen un descuento de reserva tienen un precio efectivo de cero. Los datos de uso muestran el descuento de reserva de cada instancia de timbre en cada reserva.

Para obtener más información sobre cómo se muestra el descuento de reserva en los datos de uso, consulte [Obtención del uso y los costos de reservas de Contrato Enterprise](understand-reserved-instance-usage-ea.md) si es un cliente del Contrato Enterprise (EA). De lo contrario, consulte el artículo [Descripción del uso de reservas de Azure para su suscripción individual con precios de pago por uso](understand-reserved-instance-usage.md).

## <a name="next-steps"></a>Pasos siguientes

- Para obtener más información acerca de Azure Reservations, consulte los siguientes artículos:
  - [¿Qué es Azure Reservations?](save-compute-costs-reservations.md)
  - [Understand how an Azure App Service Isolated Stamp reservation discount is applied](reservation-discount-app-service-isolated-stamp.md) (Información de cómo se aplica un descuento en la reserva del timbre de Azure App Service aislado).
  - [Información sobre el uso de reservas para la inscripción Enterprise](understand-reserved-instance-usage-ea.md)
