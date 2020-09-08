---
title: Optimización de los costes de Azure Databricks con una compra anticipada
description: Obtenga información sobre cómo puede pagar por adelantado los cargos de Azure Databricks con la capacidad reservada para ahorrar dinero.
author: yashesvi
ms.reviewer: yashar
ms.service: cost-management-billing
ms.subservice: reservations
ms.topic: how-to
ms.date: 07/24/2020
ms.author: banders
ms.openlocfilehash: 699b3d9c9799b9d3a4ec148cbf395b212715f536
ms.sourcegitcommit: 56cbd6d97cb52e61ceb6d3894abe1977713354d9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88686095"
---
# <a name="optimize-azure-databricks-costs-with-a-pre-purchase"></a>Optimización de los costes de Azure Databricks con una compra anticipada

Puede ahorrar en los costes de las unidades de Azure Databricks (DBU) si compra de manera anticipada unidades de confirmación de Azure Databricks (DBCU) para uno o tres años. Puede usar las DBCU que haya comprado de manera anticipada en cualquier momento durante el plazo de compra. A diferencia de las máquinas virtuales, las unidades compradas de manera anticipada no expiran cada hora y se pueden usar en cualquier momento durante el plazo de compra.

Cualquier uso de Azure Databricks se deduce automáticamente de las DBU compradas de manera anticipada. No es necesario volver a implementar o asignar un plan adquirido de forma anticipada a las áreas de trabajo de Azure Databricks para el uso de DBU con el fin de obtener los descuentos de compra anticipada.

El descuento de compra anticipada solo se aplica al uso de DBU. Otros gastos tales como proceso, almacenamiento y redes se cobran por separado.

## <a name="determine-the-right-size-to-buy"></a>Determinación del tamaño que se debe comprar

La compra anticipada de Databricks se aplica a todos los niveles y cargas de trabajo de Databricks. Puede considerar la compra anticipada como un grupo de unidades de confirmación de Databricks pagadas previamente. El uso se deduce del grupo, independientemente de la carga de trabajo o el nivel. El uso se deduce según la proporción siguiente:

| **Carga de trabajo** | **Proporción de aplicación de DBU: nivel Estándar** | **Proporción de aplicación de DBU: nivel Premium** |
| --- | --- | --- |
| Análisis de datos | 0,4 | 0,55 |
| Ingeniería de datos | 0,15 | 0,30 |
| Ingeniería de datos ligera | 0,07 | 0,22 |

Por ejemplo, cuando se consume una cantidad de análisis de datos de nivel Estándar, se deducen 0,4 unidades de las unidades de confirmación de Databricks compradas de manera anticipada.

Antes de comprar, calcule el total de DBU consumido para las diferentes cargas de trabajo y niveles. Use las proporciones anteriores para normalizar a DBCU y, luego, realice una proyección del uso total en el futuro durante uno o tres años.

## <a name="purchase-databricks-commit-units"></a>Compra de unidades de confirmación de Databricks

Puede comprar planes de Databricks en [Azure Portal](https://portal.azure.com/#blade/Microsoft_Azure_Reservations/CreateBlade/referrer/documentation/filters/%7B%22reservedResourceType%22%3A%22Databricks%22%7D). Para comprar capacidad reservada, debe tener el rol de propietario al menos en una suscripción Enterprise.

- Debe tener un rol de propietario al menos en una suscripción Enterprise (números de oferta: MS-AZR-0017P o MS-AZR-0148P) o Contrato de cliente de Microsoft o una suscripción individual con tarifas de pago por uso (números de oferta: MS-AZR-0003P o MS-AZR-0023P).
- En el caso de las suscripciones de EA, la opción Agregar instancias reservadas debe estar habilitada en el portal de EA. O bien, si esa opción está deshabilitada, debe ser un administrador de EA en la suscripción.
- En el caso de las suscripciones Enterprise, la opción **Agregar instancias reservadas** debe estar habilitada en el [portal de EA](https://ea.azure.com/). O bien, si esa opción está deshabilitada, debe ser un administrador de EA en la suscripción.

**Para comprar:**

1. Vaya a [Azure Portal](https://portal.azure.com/#blade/Microsoft_Azure_Reservations/CreateBlade/referrer/documentation/filters/%7B%22reservedResourceType%22%3A%22Databricks%22%7D).
1. Seleccione una suscripción. Use la lista **Suscripción** para seleccionar la que se va a usar para pagar la capacidad reservada. Los costos anticipados de la capacidad reservada se cobran en el método de pago de la suscripción. Los cargos se deducen del saldo de compromiso monetario de la inscripción o se cobran como uso por encima del límite.
1. Seleccione un ámbito. Use la lista **Ámbito** para seleccionar el ámbito de la suscripción:
    - **Single resource group scope** (Ámbito de grupo de recursos único): aplica el descuento por reserva a los recursos coincidentes solo en el grupo de recursos seleccionado.
    - **Single subscription scope** (Ámbito de suscripción única): aplica el descuento por reserva a los recursos coincidentes de la suscripción seleccionada.
    - **Ámbito compartido**: aplica el descuento por reserva a los recursos coincidentes en suscripciones aptas que están en el contexto de facturación. Para los clientes con Contrato Enterprise, el contexto de facturación es la inscripción.
1. Seleccione el número de unidades de confirmación de Azure Databricks que quiere comprar y complete la compra.


![Ejemplo en el que se muestra la compra de Azure Databricks en Azure Portal](./media/prepay-databricks-reserved-capacity/data-bricks-pre-purchase.png)

## <a name="change-scope-and-ownership"></a>Cambio del ámbito y de la propiedad

Puede realizar los siguientes tipos de cambios en una reserva después de haberla comprado:

- Actualización del ámbito de la reserva
- Acceso basado en rol

La compra anticipada de unidades de confirmación de Databricks no se puede dividir ni combinar. Para obtener más información sobre cómo administrar las reservas, consulte [Administración de reservas](manage-reserved-vm-instance.md).

## <a name="cancellations-and-exchanges"></a>Cancelaciones e intercambios

No se admite la cancelación ni el intercambio en los planes de compra anticipada de Databricks. Todas las compras son definitivas.

## <a name="need-help-contact-us"></a>¿Necesita ayuda? Póngase en contacto con nosotros.

Si tiene alguna pregunta o necesita ayuda, [cree una solicitud de soporte técnico](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest).

## <a name="next-steps"></a>Pasos siguientes

- Para obtener más información acerca de Azure Reservations, consulte los siguientes artículos:
  - [¿Qué es Azure Reservations?](save-compute-costs-reservations.md)
  - [Descripción de cómo se aplica un descuento de DBCU de compra anticipada de Azure Databricks](reservation-discount-databricks.md)
  - [Información sobre el uso de reservas para la inscripción Enterprise](understand-reserved-instance-usage-ea.md)
