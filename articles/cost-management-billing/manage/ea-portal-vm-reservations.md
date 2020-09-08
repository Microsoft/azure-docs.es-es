---
title: Instancias reservadas de máquinas virtuales de Azure EA
description: En este artículo se resume la forma en que las reservas de Azure para las instancias reservadas de máquinas virtuales pueden ayudarle a ahorrar dinero en la inscripción de su empresa.
author: bandersmsft
ms.author: banders
ms.date: 08/20/2020
ms.topic: conceptual
ms.service: cost-management-billing
ms.subservice: enterprise
ms.reviewer: boalcsva
ms.openlocfilehash: aa95cf8b649558dc3164fe12e4268f210301f755
ms.sourcegitcommit: 56cbd6d97cb52e61ceb6d3894abe1977713354d9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88687047"
---
# <a name="azure-ea-vm-reserved-instances"></a>Instancias reservadas de máquinas virtuales de Azure EA

En este artículo se resume la forma en que las reservas de Azure para las instancias reservadas de máquinas virtuales pueden ayudarle a ahorrar dinero en la inscripción de su empresa. Para más información sobre las reservas, consulte [¿Qué es Azure Reservations?](../reservations/save-compute-costs-reservations.md).

## <a name="reservation-exchanges-and-refunds"></a>Cambios de reserva y reembolsos

Puede intercambiar una reserva por otra del mismo tipo. También puede reembolsar una reserva, hasta 50 000 USD al año, si ya no la necesita. Puede usar Azure Portal para intercambiar o reembolsar una reserva. Para más información, consulte [Autoservicio de intercambios y reembolsos de reservas de Azure](../reservations/exchange-and-refund-azure-reservations.md).

## <a name="reservation-costs-and-usage"></a>Costos y uso de las reservas

Los clientes que tengan Contrato Enterprise pueden ver los datos de uso y costos tanto en Azure Portal como en las API REST. Para conocer los costos y el uso de las reservas, puede:

- Obtener los datos de compra de la reserva.
- Saber qué suscripción, grupo de recursos o recurso usó una reserva.
- Anular el uso de la reserva.
- Calcular el ahorro de la reserva.
- Obtener los datos de infrautilización de la reserva.
- Amortizar los costos de la reserva.

Para obtener más información sobre los costos de reserva y el uso, vea [Obtención del uso y los costos de reservas de Contrato Enterprise](../reservations/understand-reserved-instance-usage-ea.md).

Para obtener información sobre los precios, consulte [Precios de máquinas virtuales Linux](https://azure.microsoft.com/pricing/details/virtual-machines/linux/) o [Precios de máquinas virtuales Windows](https://azure.microsoft.com/pricing/details/virtual-machines/windows/).

## <a name="reserved-instances-api-support"></a>Soporte técnico de la API de instancias reservadas

Use las API de Azure para obtener información mediante programación para su organización sobre el servicio de Azure o las reservas de software. Por ejemplo, use las API para:

- Buscar las reservas que desee comprar
- Adquisición de una reserva
- Visualización de las reservas adquiridas
- Ver y administrar el acceso a las reservas
- Dividir o combinar reservas
- Cambiar el ámbito de las reservas

Para más información, consulte [APIs for Azure reservation automation](../reservations/reservation-apis.md) (API de automatización de reservas de Azure).

## <a name="azure-reserved-virtual-machine-instances"></a>Instancias reservadas de máquinas virtuales de Azure

Las instancias reservadas pueden reducir los costos de las máquinas virtuales hasta un 72 % respecto a los precios de Pago por uso en todas las máquinas virtuales. O bien, hasta el 82 % cuando se combinan con la Ventaja híbrida de Azure. Las instancias reservadas le ayudan a administrar mejor las cargas de trabajo, el presupuesto y la previsión, gracias al pago por adelantado en períodos de un año o de tres años. También se pueden intercambiar o cancelar las reservas a medida que cambien las necesidades del negocio.

### <a name="how-to-buy-reserved-virtual-machine-instances"></a>Compra de instancias reservadas de máquinas virtuales

Para comprar una instancia reservada de máquina virtual de Azure, un administrador de inscripciones de Azure Enterprise debe habilitar la opción de compra _Reserve Instance_ (Reservar instancia). La opción se encuentra en la sección _Enrollment Detail_ (Detalles de la inscripción) de la pestaña _Enrollment_ (Inscripción) de [Azure Enterprise Portal](https://ea.azure.com/).

Una vez que la inscripción de Contrato Enterprise está habilitada para agregar instancias reservadas, cualquier propietario de la cuenta con una suscripción activa asociada a esta inscripción puede comprar una instancia reservada de máquina virtual en [Azure Portal](https://aka.ms/reservations). Para obtener más información, consulte [Pago por adelantado de máquinas virtuales para ahorrar dinero con instancias reservadas de máquina virtual](https://go.microsoft.com/fwlink/?linkid=861721).

### <a name="how-to-view-reserved-instance-purchase-details"></a>Visualización de los detalles de la compra de instancias reservadas

Puede ver los detalles de la compra de instancias reservadas a través del menú _Reservas_ en el lado izquierdo de [Azure Portal](https://aka.ms/reservations) o desde [Azure EA Portal](https://ea.azure.com/). Seleccione **Reports** (Informes) en el menú de la izquierda y desplácese hacia abajo, hasta la sección _Charges by Services_ (Cargos por servicios) de la pestaña _Usage Summary_ (Resumen de uso). Desplácese hasta la parte inferior de la sección; las compras y el uso de las instancia reservadas se mostrarán al final, indicadas con la designación de `1 year` o `3 years` junto al nombre del servicio; por ejemplo, `Standard_DS1_v2 eastus 1 year` o `Standard_D2s_v3 eastus2 3 years`.

### <a name="how-can-i-change-the-subscription-associated-with-reserved-instance-or-transfer-my-reserved-instance-benefits-to-a-subscription-under-the-same-account"></a>¿Cómo puedo cambiar la suscripción asociada a la instancia reservada o transferir las ventajas de la instancia reservada a una suscripción incluida en la misma cuenta?

Puede cambiar la suscripción que recibe las ventajas de las instancias reservadas del siguiente modo:

- Inicie sesión en [Azure Portal](https://aka.ms/reservations).
- Actualice el ámbito de la suscripción aplicada; para ello, asocie una suscripción diferente con la misma cuenta.

Para más información sobre cómo cambiar el ámbito de una reserva, consulte [Cambio del ámbito de reserva](../reservations/manage-reserved-vm-instance.md#change-the-reservation-scope).

### <a name="how-to-view-reserved-instance-usage-details"></a>Visualización de los detalles de uso de las instancias reservadas

Puede ver los detalles de uso de las instancias reservadas en [Azure Portal](https://aka.ms/reservations) o en [Azure EA Portal](https://ea.azure.com/) (para los clientes de Contrato Enterprise que tienen acceso para ver la información de facturación) en _Reports_ > _Usage Summary_ > _Charges by Services_ (Informes > Resumen de uso > Cargos por servicios). Sus instancias reservadas se pueden identificar como nombres de servicio que contienen el término "Reservation"; por ejemplo, `Reservation-Base VM or Virtual Machines Reservation-Windows Svr (1 Core)`.

El archivo .csv de descarga del informe avanzado y de detalles de uso tiene información adicional sobre el uso de las instancias reservadas. El campo _Additional Info_ (Información adicional) le ayuda a identificar el uso de instancias reservadas.

Si no usó la ventaja híbrida de Azure para comprar instancias reservadas de máquinas virtuales de Azure, las instancias reservadas emitirán dos medidores (hardware y software). Cuando use la ventaja híbrida de Azure para comprar instancias reservadas, no verá el medidor de software en los detalles de uso de las instancias reservadas.

### <a name="reserved-instance-billing"></a>Facturación de instancias reservadas

En el caso de los clientes de Enterprise, se usa el prepago de Azure para comprar instancias reservadas de máquinas virtuales de Azure. Si la inscripción tiene un saldo de prepago de Azure suficiente para cubrir la compra de las instancias reservadas, el importe se deducirá del saldo del prepago de Azure y no recibirá ninguna factura por la compra.

En los casos en los que los clientes de Azure EA hayan usado todo su prepago de Azure, todavía podrán comprar instancias reservadas, cuyo importe se incluirá en la próxima factura de uso por encima del límite. El uso por encima del límite de las instancias reservadas, si se produce, formará parte de la factura normal de uso por encima del límite.

### <a name="reserved-instance-expiration"></a>Expiración de las instancias reservadas

Recibirá notificaciones por correo electrónico 30 días antes de la reserva y en la expiración. Una vez que expire la reserva, las máquinas virtuales implementadas seguirán ejecutándose y se facturarán mediante una tasa de pago por uso. Para más información, consulte las [ofertas de instancias reservadas de máquinas virtuales](https://azure.microsoft.com/pricing/reserved-vm-instances/).

## <a name="next-steps"></a>Pasos siguientes

- Para más información sobre las reservas de Azure, consulte [¿Qué es Azure Reservations?](../reservations/save-compute-costs-reservations.md).
- Para más información sobre los costos y el uso de las reservas, consulte [Obtención del uso y los costos de reservas de Contrato Enterprise](../reservations/understand-reserved-instance-usage-ea.md).
- Para obtener información sobre los precios, consulte [Precios de máquinas virtuales Linux](https://azure.microsoft.com/pricing/details/virtual-machines/linux/) o [Precios de máquinas virtuales Windows](https://azure.microsoft.com/pricing/details/virtual-machines/windows/).
