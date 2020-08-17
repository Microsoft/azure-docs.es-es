---
title: 'Pago por adelantado de recursos de proceso con capacidad reservada: Azure Cache for Redis'
description: Pago por adelantado de recursos de proceso de Azure Cache for Redis con capacidad reservada
author: yegu-ms
ms.author: yegu
ms.service: cache
ms.topic: conceptual
ms.date: 02/20/2020
ms.openlocfilehash: c4ea8db464c7cae57b80837ced9f8886a8b3cf4a
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87093024"
---
# <a name="prepay-for-azure-cache-for-redis-compute-resources-with-reserved-capacity"></a>Pago por adelantado de recursos de proceso de Azure Cache for Redis con capacidad reservada

Ahora Azure Cache for Redis le ayuda a ahorrar mediante el pago por adelantado de los recursos de proceso en comparación con los precios de pago por uso. Con la capacidad reservada de Azure Cache for Redis, se realiza un compromiso inicial en la caché durante un periodo de uno o tres años con el fin de obtener un descuento importante en los costos de proceso. Para comprar capacidad reservada de Azure Cache for Redis, debe especificar la región de Azure, el nivel de servicio y el período.

No es necesario asignar la reserva a instancias específicas de Azure Cache for Redis. Una instancia de Azure Cache for Redis que ya está en ejecución o las que se han implementado recientemente obtendrán de forma automática la ventaja de los precios reservados, hasta el tamaño de caché reservado. Al comprar una reserva, se adelanta el pago de los costos de proceso durante un período de uno a tres años. En cuanto se compra una reserva, los cargos de proceso de Azure Cache for Redis que coincidan con los atributos de reserva dejan de pagarse según las tarifas de pago por uso. Una reserva no cubre los cargos de red ni de almacenamiento asociados con la caché. Al final del período de reserva, la ventaja en la facturación expira y la instancia de Azure Cache for Redis se factura según el precio de pago por uso. Las reservas no se renuevan automáticamente. Para obtener información sobre precios, vea [Oferta de capacidad reservada de Azure Cache for Redis](https://azure.microsoft.com/pricing/details/cache).

Puede comprar capacidad reservada de Azure Cache for Redis en [Azure Portal](https://portal.azure.com/). Para comprar capacidad reservada:

* Debe tener el rol de propietario al menos en una suscripción Enterprise o individual con tarifas de pago por uso.
* En el caso de las suscripciones Enterprise, la opción **Agregar instancias reservadas** debe estar habilitada en el [portal de EA](https://ea.azure.com/). O bien, si esa opción está deshabilitada, debe ser un administrador de EA en la suscripción.
* En el caso del programa del Proveedor de soluciones en la nube (CSP), los únicos que pueden comprar la capacidad reservada de Azure Cache for Redis son los agentes de administración o de ventas.

Para más información sobre cómo se les cobran a los clientes de empresa y a los de pago por uso las compras de reservas, consulte [Información sobre el uso de reservas de Azure para la inscripción Enterprise](https://docs.microsoft.com/azure/billing/billing-understand-reserved-instance-usage-ea) e [Información sobre el uso de reservas de Azure para suscripciones de pago por uso](https://docs.microsoft.com/azure/billing/billing-understand-reserved-instance-usage).


## <a name="determine-the-right-cache-size-before-purchase"></a>Determinación del tamaño de caché adecuado antes de la compra

El tamaño de la reserva se debe basar en la cantidad total de memoria que usa la caché existente o que se va a implementar pronto en una región específica y con el mismo nivel de servicio.

Por ejemplo, supongamos que está ejecutando dos memorias caché (una en 13 GB y la otra en 26 GB). Necesitará ambas durante al menos un año. Además, supongamos que planea escalar las cachés existentes de 13 GB a 26 GB durante un mes para satisfacer la demanda estacional y, después, volver a escalar. En este caso, puede adquirir una caché P2 y una caché P3, o bien tres cachés P2 en una reserva de un año para maximizar el ahorro. Recibirá un descuento en la cantidad total de memoria caché que reserva, independientemente de cómo se asigne esa cantidad a través de las cachés.


## <a name="buy-azure-cache-for-redis-reserved-capacity"></a>Compra de capacidad reservada de Azure Cache for Redis

1. Inicie sesión en [Azure Portal](https://portal.azure.com/).
2. Seleccione **Todos los servicios** > **Reservations**.
3. Seleccione **Agregar** y, en el panel Comprar reservas, seleccione **Azure Cache for Redis** para comprar una nueva reserva para las cachés.
4. Rellene todos los campos obligatorios. Las bases de datos existentes o nuevas que coincidan con los atributos seleccionados serán aptas para el descuento en la capacidad reservada. El número real de instancias de Azure Cache for Redis que obtienen el descuento depende del ámbito y la cantidad que se seleccionen.


![Información general sobre los precios reservados](media/cache-reserved-pricing/cache-reserved-price.png)


En la siguiente tabla se describen los campos obligatorios.

| Campo | Descripción |
| :------------ | :------- |
| Suscripción   | La suscripción usada para pagar la reserva de capacidad reservada de Azure Cache for Redis. Los costos anticipados por la reserva de capacidad reservada de Azure Cache for Redis se cobran mediante el método de pago de la suscripción. El tipo de suscripción debe ser Contrato Enterprise (números de oferta: MS-AZR-0017P o MS-AZR-0148P) o un contrato individual con precios de pago por uso (números de oferta: MS-AZR-0003P o MS-AZR-0023P). Para una suscripción Enterprise, los cargos se deducen del saldo de compromiso monetario de la inscripción o se cobran como uso por encima del límite. Para una suscripción individual con precios de pago por uso, los cargos se cobran en el método de pago de la factura o la tarjeta de crédito de la suscripción.
| Ámbito | El ámbito de la reserva puede cubrir una o varias suscripciones (ámbito compartido). Si selecciona: </br></br> **Compartido**: el descuento de la reserva se aplica a las instancias de Azure Cache for Redis que se ejecutan en cualquier suscripción en el contexto de facturación. Para los clientes Enterprise, el ámbito compartido es la inscripción e incluye todas las suscripciones que esta contiene. Para los clientes de Pago por uso, el ámbito compartido incluye todas las suscripciones de Pago por uso creadas por el administrador de la cuenta.</br></br> **Suscripción única**: el descuento de reserva se aplica a las instancias de Azure Cache for Redis de esta suscripción. </br></br> **Grupo de recursos único**: el descuento de reserva se aplica a las instancias de Azure Cache for Redis de la suscripción seleccionada y al grupo de recursos seleccionado de esa suscripción.
| Region | Región de Azure que abarca la reserva de capacidad reservada de Azure Cache for Redis.
| Plan de tarifa | Nivel de servicio de los servidores de Azure Cache for Redis.
| Término | Uno o tres años.
| Cantidad | Número de recursos de proceso que se compran dentro de la reserva de capacidad reservada de Azure Cache for Redis. La cantidad es un número de cachés de la región de Azure y el nivel de servicio seleccionados que se están reservando y obtendrán el descuento de facturación. Por ejemplo, si ejecuta o planea ejecutar servidores de Azure Cache for Redis con la capacidad total de caché de 26 GB en la región Este de EE. UU., deberá especificar la cantidad que le otorgue el equivalente a 26 GB para maximizar las ventajas de todas las cachés. Podrían ser una caché P3 o dos P2.

## <a name="cancel-exchange-or-refund-reservations"></a>Cancelación, intercambio o reembolso de reservas

Puede cancelar, intercambiar o reembolsar reservas con ciertas limitaciones. Para más información, consulte [Autoservicio de intercambios y reembolsos de reservas de Azure](https://docs.microsoft.com/azure/billing/billing-azure-reservations-self-service-exchange-and-refund).

## <a name="cache-size-flexibility"></a>Flexibilidad de tamaño de la caché

La flexibilidad de tamaño de la caché le ayuda a escalar o reducir verticalmente dentro de un nivel de servicio y una región, sin perder los beneficios de la capacidad reservada.

## <a name="need-help-contact-us"></a>¿Necesita ayuda? Ponerse en contacto con nosotros

Si tiene alguna pregunta o necesita ayuda, [cree una solicitud de soporte técnico](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest).

## <a name="next-steps"></a>Pasos siguientes

El descuento de la reserva se aplica automáticamente a las instancias de Azure Cache for Redis que coinciden con el ámbito y los atributos de la reserva. Puede actualizar el ámbito de la reserva mediante Azure Portal, PowerShell, la CLI de Azure o la API.

*  Para saber cómo se aplican los descuentos de capacidad reservada a Azure Cache for Redis, consulte [Información sobre el descuento de Azure Reservations](../cost-management-billing/reservations/understand-azure-cache-for-redis-reservation-charges.md).

* Para obtener más información acerca de Azure Reservations, consulte los siguientes artículos:

    * [¿Qué es Azure Reservations?](https://docs.microsoft.com/azure/billing/billing-save-compute-costs-reservations)
    * [Administración de Azure Reservations](https://docs.microsoft.com/azure/billing/billing-manage-reserved-vm-instance)
    * [Información sobre el descuento de Azure Reservations](https://docs.microsoft.com/azure/billing/billing-understand-reservation-charges)
    * [Información sobre el uso de reservas para suscripciones de pago por uso](https://docs.microsoft.com/azure/billing/billing-understand-reservation-charges-mysql)
    * [Información sobre el uso de reservas para la inscripción Enterprise](https://docs.microsoft.com/azure/billing/billing-understand-reserved-instance-usage-ea)
    * [Azure Reservations en el programa del Proveedor de soluciones en la nube (CSP) del Centro de partners](https://docs.microsoft.com/partner-center/azure-reservations)

