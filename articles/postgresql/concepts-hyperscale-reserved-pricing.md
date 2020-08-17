---
title: Precios de procesos reservados de Hiperescala (Citus) para Azure Database for PostgreSQL
description: Pago por adelantado de recursos de proceso de Hiperescala (Citus) para Azure Database for PostgreSQL con capacidad reservada.
author: jonels-msft
ms.author: jonels
ms.service: postgresql
ms.subservice: hyperscale-citus
ms.topic: conceptual
ms.date: 06/15/2020
ms.openlocfilehash: a5ce99927ce4cd2b04b5dd5cb865299b4be84ecb
ms.sourcegitcommit: 3543d3b4f6c6f496d22ea5f97d8cd2700ac9a481
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/20/2020
ms.locfileid: "86519803"
---
# <a name="prepay-for-azure-database-for-postgresql---hyperscale-citus-compute-resources-with-reserved-capacity"></a>Pago por adelantado de recursos de proceso de Hiperescala (Citus) para Azure Database for PostgreSQL con capacidad reservada

Ahora Hiperescala (Citus) para Azure Database for PostgreSQL le ayuda a ahorrar mediante el pago por adelantado de los recursos de proceso en comparación con los precios de pago por uso. Con la capacidad reservada de Hiperescala (Citus), se realiza un compromiso inicial en el grupo de servidores de Hiperescala (Citus) durante un periodo de uno o tres años con el fin de obtener un descuento importante en los costos de proceso. Para comprar capacidad reservada de Hiperescala (Citus), debe especificar la región de Azure, el plazo de reserva y la frecuencia de facturación.

> [!IMPORTANT]
> Este artículo trata sobre la capacidad reservada para Hiperescala (Citus) de Azure Database for PostgreSQL. Para obtener información sobre la capacidad reservada para Un solo servidor de Azure Database for PostgreSQL, consulte [Pago por adelantado de recursos de proceso de Un solo servidor para Azure Database for PostgreSQL con capacidad reservada](/azure/postgresql/concept-reserved-pricing).

No es necesario asignar la reserva a grupos de servidores específicos de Hiperescala (Citus). Un grupo de servidores de Hiperescala (Citus) que ya está en ejecución, o los que se han implementado recientemente, obtendrán de forma automática la ventaja de los precios reservados. Al comprar una reserva, se adelanta el pago de los costos de proceso durante un período de uno a tres años. Tan pronto como se compra una reserva, los costos de proceso de Hiperescala (Citus) que coinciden con los atributos de la reserva dejan de pagarse según las tarifas de pago por uso. 

La reserva no cubre los cargos por software, redes o almacenamiento asociados a los grupos de servidor de Hiperescala (Citus). Al final del plazo de reserva, la ventaja en la facturación expira y los grupos de servidor de Hiperescala (Citus) se facturan según los precios de pago por uso. Las reservas no se renuevan automáticamente. Para obtener información sobre precios, consulte [Oferta de capacidad reservada de Hiperescala (Citus) para Azure Database for PostgreSQL](https://azure.microsoft.com/pricing/details/postgresql/hyperscale-citus/).

Puede comprar la capacidad reservada de Hiperescala (Citus) en [Azure Portal](https://portal.azure.com/). Pague la reserva [por adelantado o mensualmente](https://docs.microsoft.com/azure/cost-management-billing/reservations/monthly-payments-reservations). Para comprar capacidad reservada:

* Debe tener el rol Propietario al menos en una suscripción de contrato Enterprise (EA) o individual con tarifas de pago por uso.
* En el caso de las suscripciones de contrato Enterprise, la opción **Agregar instancias reservadas** debe estar habilitada en el [portal de EA](https://ea.azure.com/). O bien, si esa opción está deshabilitada, debe ser un administrador de contrato Enterprise en la suscripción.
* En el caso del programa del Proveedor de soluciones en la nube (CSP), los únicos que pueden comprar la capacidad reservada de Hiperescala (Citus) son los agentes de administración o de ventas.

Para obtener información sobre cómo se cargan los clientes con contrato Enterprise y los de Pago por uso por las compras de reserva, vea lo siguiente:
- [Información sobre el uso de reservas de Azure para la inscripción de contrato Enterprise](https://docs.microsoft.com/azure/billing/billing-understand-reserved-instance-usage-ea)
- [Información sobre el uso de reservas de Azure para suscripciones de Pago por uso](https://docs.microsoft.com/azure/billing/billing-understand-reserved-instance-usage)

## <a name="determine-the-right-server-group-size-before-purchase"></a>Determinación del tamaño de grupo de servidor adecuado antes de la compra

El tamaño de la reserva se basa en la cantidad total de proceso que van a usar los nodos de trabajo y coordinación existentes o que se van a implementar pronto en los grupos de servidores de Hiperescala (Citus) de una región específica.

Por ejemplo, supongamos que está ejecutando un grupo de servidores de Hiperescala (Citus) con un nodo de coordinación de 16 núcleos virtuales y tres nodos de trabajo de 8 núcleos virtuales. Además, supongamos que planea implementar en el siguiente mes un grupo de servidores de Hiperescala (Citus) adicional con un nodo de coordinación de 32 núcleos virtuales y dos nodos de trabajo de 4 núcleos virtuales. Vamos también a suponer que necesita estos recursos durante al menos un año.

En este caso, adquiera la reserva de un año para lo siguiente:

* Total de 16 núcleos virtuales + 32 núcleos virtuales = 48 núcleos virtuales para nodos de coordinación
* Total de tres nodos x 8 núcleos virtuales + dos nodos x 4 núcleos virtuales = 24 + 8 = 32 núcleos virtuales para nodos de trabajo

## <a name="buy-azure-database-for-postgresql-reserved-capacity"></a>Compra de capacidad reservada de Azure Database for PostgreSQL

1. Inicie sesión en [Azure Portal](https://portal.azure.com/).
1. Seleccione **Todos los servicios** > **Reservations**.
1. Seleccione **Agregar**. En el panel **Comprar reservas**, seleccione **Azure Database for PostgreSQL** para comprar una nueva reserva para las bases de datos de PostgreSQL.
1. Seleccione el tipo de **proceso de Hiperescala (Citus)** que se desea comprar y haga clic en **Seleccionar**.
1. Revise la cantidad del tipo de proceso seleccionado en la pestaña **Productos**.
1. Continúe con la pestaña **Comprar y revisar** para finalizar la compra.

En la siguiente tabla se describen los campos obligatorios.

| Campo        | Descripción                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
|--------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Suscripción | La suscripción usada para pagar la reserva de capacidad reservada de Azure Database for PostgreSQL. Los costos anticipados por la reserva de capacidad reservada de Azure Database for PostgreSQL se cobran mediante el método de pago de la suscripción. El tipo de suscripción debe ser Contrato Enterprise (números de oferta: MS-AZR-0017P o MS-AZR-0148P) o un contrato individual con precios de pago por uso (números de oferta: MS-AZR-0003P o MS-AZR-0023P). Para una suscripción de contrato Enterprise, los cargos se deducen del saldo de compromiso monetario de la inscripción o se cobran como uso por encima del límite. Para una suscripción individual con precios de pago por uso, los cargos se cobran en el método de pago de la factura o la tarjeta de crédito de la suscripción.                                                                                  |
| Ámbito        | El ámbito de la reserva de núcleos virtuales puede cubrir una suscripción o varias (ámbito compartido). Si selecciona **Compartido**, el descuento por la reserva de núcleos virtuales se aplica a los grupos de servidores de Hiperescala (Citus) en cualquiera de las suscripciones en el contexto de facturación. Para los clientes con contrato Enterprise, el ámbito compartido es la inscripción e incluye todas las suscripciones que esta contiene. Para los clientes de Pago por uso, el ámbito compartido incluye todas las suscripciones de Pago por uso creadas por el administrador de la cuenta. Si selecciona **Suscripción única**, el descuento por la reserva de núcleos virtuales se aplica a los grupos de servidores de Hiperescala (Citus) de esta suscripción. Si selecciona **Grupo de recursos único**, el descuento de reserva se aplica a los grupos de servidores de Hiperescala (Citus) de la suscripción seleccionada y al grupo de recursos seleccionado de esa suscripción. |
| Region       | La región de Azure que abarca la reserva de capacidad reservada de Hiperescala (Citus) para Azure Database for PostgreSQL.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| Término         | Un año o tres años.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| Cantidad     | Número de recursos de proceso que se compran dentro de la reserva de capacidad reservada de Hiperescala (Citus). En concreto, el número de núcleos virtuales de los nodos de trabajo o coordinación en la región de Azure seleccionada que se están reservando y obtendrán el descuento de facturación. Por ejemplo, si ejecuta (o planea ejecutar) los grupos de servidores de Hiperescala (Citus) con la capacidad de proceso total de 64 núcleos virtuales de nodo de coordinación y 32 núcleos virtuales de nodo de trabajo en la región Este de EE. UU., especifique la cantidad como 64 y 32 para los nodos de coordinación y de trabajo, respectivamente, para maximizar el beneficio de todos los servidores.                                                                                                                                                                                                                                                     |



## <a name="cancel-exchange-or-refund-reservations"></a>Cancelación, intercambio o reembolso de reservas

Puede cancelar, intercambiar o reembolsar reservas con ciertas limitaciones. Para más información, consulte [Autoservicio de intercambios y reembolsos de reservas de Azure](https://docs.microsoft.com/azure/billing/billing-azure-reservations-self-service-exchange-and-refund).

## <a name="vcore-size-flexibility"></a>Flexibilidad de tamaño del núcleo virtual

La flexibilidad de tamaño del núcleo virtual le ayuda a escalar o reducir verticalmente los nodos de trabajo y coordinación dentro de una región, sin perder los beneficios de la capacidad reservada.

## <a name="need-help-contact-us"></a>¿Necesita ayuda? Ponerse en contacto con nosotros

Si tiene alguna pregunta o necesita ayuda, [cree una solicitud de soporte técnico](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest).

## <a name="next-steps"></a>Pasos siguientes

El descuento por la reserva de núcleos virtuales se aplica automáticamente al número de grupos de servidores de Hiperescala (Citus) que coinciden con el ámbito y los atributos de la reserva de capacidad reservada de Azure Database for PostgreSQL. Se puede actualizar el ámbito de la reserva de capacidad reservada de Hiperescala (Citus) para Azure Database for PostgreSQL a través de Azure Portal, PowerShell, la CLI de Azure o la API.

Para más información acerca de las reservas de Azure, consulte los siguientes artículos:

* [¿Qué son las reservas de Azure?](https://docs.microsoft.com/azure/billing/billing-save-compute-costs-reservations)
* [Administración de reservas de Azure](https://docs.microsoft.com/azure/billing/billing-manage-reserved-vm-instance)
* [Información sobre el descuento de Azure Reservations](https://docs.microsoft.com/azure/billing/billing-understand-reservation-charges)
* [Información sobre el uso de reservas para suscripciones de pago por uso](https://docs.microsoft.com/azure/billing/billing-understand-reservation-charges-postgresql)
* [Información sobre el uso de reservas para la inscripción de contrato Enterprise](https://docs.microsoft.com/azure/billing/billing-understand-reserved-instance-usage-ea)
* [Azure Reservations en el programa del Proveedor de soluciones en la nube del Centro de partners](https://docs.microsoft.com/partner-center/azure-reservations)
