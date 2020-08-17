---
title: Pago por adelantado de instancias de Azure Dedicated Host para ahorrar dinero
description: Aprenda a comprar instancias reservadas de Azure Dedicated Host para ahorrar en costos de proceso.
services: virtual-machines
author: yashar
ms.service: virtual-machines
ms.topic: conceptual
ms.workload: infrastructure-services
ms.date: 02/28/2020
ms.author: banders
ms.openlocfilehash: 919ff7474dd64cfe3005975beb4cab6f77c177ce
ms.sourcegitcommit: 2ff0d073607bc746ffc638a84bb026d1705e543e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87825266"
---
# <a name="save-costs-with-azure-dedicated-host-reservations"></a>Ahorro de costos con las reservas de Azure Dedicated Host

Al confirmar una instancia reservada de Azure Dedicated Host, puede ahorrar dinero. El descuento de la reserva se aplica automáticamente el número de hosts dedicados en ejecución que coinciden con el ámbito y los atributos de la reserva. No es necesario asignar una reserva a un host dedicado para obtener los descuentos. La compra de una instancia reservada cubre solo la parte de proceso de su uso e incluye los costos de licencia de software. Consulte [Información general de Azure Dedicated Host para máquinas virtuales](./windows/dedicated-hosts.md).

## <a name="determine-the-right-dedicated-host-sku-before-you-buy"></a>Determinación de la SKU de host dedicada adecuada antes de comprar


Antes de adquirir una reserva, debe determinar el host dedicado que necesita. Se define una SKU para un host dedicado que representa el tipo y la serie de máquina virtual. 

Para empezar, vaya a los tamaños admitidos para una [máquina virtual Windows](./sizes.md) o [Linux](./sizes.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) para identificar la serie de máquinas virtuales.

A continuación, compruebe si es compatible con Azure Dedicated Host. La página de [precios de Azure Dedicated Host](https://aka.ms/ADHPricing) tiene la lista completa de SKU de hosts dedicados, su información de CPU y diversas opciones de precios (incluidas las instancias reservadas).

Es posible que encuentre varias SKU que son compatibles con la serie de máquinas virtuales (con tipos diferentes). Identifique la mejor SKU comparando la capacidad del host (número de vCPU). Tenga en cuenta que podrá aplicar la reserva a varias SKU de hosts dedicados que admitan la misma serie de máquinas virtuales (por ejemplo DSv3_Type1 y DSv3_Type2), pero no en distintas series de máquinas virtuales (como DSv3 y ESv3).



## <a name="purchase-restriction-considerations"></a>Consideraciones sobre restricciones de compra

Las instancias reservadas están disponibles para la mayoría de los tamaños de host dedicados, con algunas excepciones.

El descuento de la reserva no se aplica para lo siguiente:

- **Nubes** : las reservas no están disponibles para la compra en las regiones de Alemania o China.

- **Cuota insuficiente** : una reserva cuyo ámbito sea de una sola suscripción debe tener cuota de vCPU disponible en la suscripción para la nueva instancia reservada. Por ejemplo, si la suscripción de destino tiene un límite de cuota de 10 vCPU para la serie DSv3, no podrá comprar hosts dedicados de reserva que admitan esta serie. La comprobación de cuota para las reservas incluye las máquinas virtuales y los host dedicados ya implementados en la suscripción. Puede  [cursar una solicitud de aumento de la cuota](../azure-portal/supportability/resource-manager-core-quotas-request.md) para resolver este problema.

- **Restricciones de capacidad** : en algunas circunstancias poco frecuentes, Azure limita la compra de nuevas reservas a un subconjunto de SKU de hosts dedicados debido a que la capacidad en una región es baja.

## <a name="buy-a-reservation"></a>Adquisición de una reserva

Puede comprar una instancia reservada de una instancia de Azure Dedicated Host en  [Azure Portal](https://portal.azure.com/#blade/Microsoft_Azure_Reservations/CreateBlade/referrer/documentation/filters/%7B%22reservedResourceType%22%3A%22VirtualMachines%22%7D).

Pague la reserva [ por adelantado o de manera mensual](../cost-management-billing/reservations/prepare-buy-reservation.md). Estos requisitos se aplican a la compra de una instancia reservada de host dedicado:

- Debe tener un rol de propietario en al menos una suscripción de EA o en una suscripción con una tarifa de pago por uso.

- En el caso de las suscripciones de EA, la opción  **Agregar instancias reservadas** debe estar habilitada en el  [portal de EA](https://ea.azure.com/). O bien, si esa opción está deshabilitada, debe ser un administrador de EA en la suscripción.

- En el caso del programa Proveedor de soluciones en la nube (CSP), solo los agentes de administración o de ventas pueden adquirir reservas.

Para comprar una instancia:

1. Inicie sesión en  [Azure Portal](https://portal.azure.com/).

2. Seleccione  **Todos los servicios** \> **Reservas**.

3. Seleccione **Agregar** para comprar una nueva reserva y, luego, haga clic en **Dedicated Hosts**.

4. Rellene los campos obligatorios. Las instancias de Dedicated Host en ejecución que coinciden con los atributos seleccionados cumplen los requisitos para obtener el descuento de la reserva. El número real de instancias de Dedicated Host que obtienen el descuento depende el ámbito y la cantidad seleccionados.

Si tiene un Contrato Enterprise, puede usar la opción  **Agregar más** para agregar rápidamente instancias adicionales. La opción no está disponible para otros tipos de suscripciones.

| **Campo**           | **Descripción**                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
|---------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Subscription        | Suscripción que se usa para pagar la reserva. Los costos de la reserva se cobran en el método de pago de la suscripción. El tipo de suscripción debe ser Contrato Enterprise (números de oferta: MS-AZR-0017P o MS-AZR-0148P) o Contrato de cliente de Microsoft o una suscripción individual con tarifas de pago por uso (números de oferta: MS-AZR-0003P o MS-AZR-0023P). Los cargos se deducen del saldo de compromiso monetario, si está disponible, o se cobran como uso por encima del límite. En una suscripción con tarifas de pago por uso, los cargos se cobran con el método de pago de factura o la tarjeta de crédito de la suscripción. |
| Ámbito               | El ámbito de la reserva puede cubrir una o varias suscripciones (ámbito compartido). Si selecciona:                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| Region              | Región de Azure que está cubierta por la reserva.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| Tamaño de Dedicated Host | Tamaño de las instancias de Dedicated Host.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| Término                | Un año o tres años.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| Cantidad            | Número de instancias que se compran dentro de la reserva. La cantidad es el número de instancias de Dedicated Host en ejecución a las que se aplica el descuento de facturación.                                                                                                                                                                                                                                                                                                                                                                                                                                                      |

- **Ámbito de grupo de recursos único** : aplica el descuento por reserva a los recursos coincidentes solo en el grupo de recursos seleccionado.

- **Ámbito de suscripción única** : aplica el descuento por reserva a los recursos coincidentes de la suscripción seleccionada.

- **Ámbito compartido** : aplica el descuento por reserva a los recursos coincidentes en suscripciones aptas que están en el contexto de facturación. Para los clientes de EA, el contexto de facturación es la inscripción. En el caso de suscripciones individuales con tarifas de pago por uso, el ámbito de facturación son todas las suscripciones aptas creadas por el administrador de la cuenta.

## <a name="usage-data-and-reservation-utilization"></a>Datos de uso y utilización de la reserva

Los datos de uso que obtienen un descuento de reserva tienen un precio efectivo de cero. Puede ver qué instancia de máquina virtual recibió el descuento de reserva para cada reserva.

Para obtener más información sobre cómo aparecen los descuentos de reserva en los datos de uso, consulte  [Obtención del uso y los costos de reservas de Contrato Enterprise](../cost-management-billing/reservations/understand-reserved-instance-usage-ea.md) si es un cliente de EA. Si tiene una suscripción individual, consulte  [Descripción del uso de reservas para su suscripción individual de pago por uso](../cost-management-billing/reservations/understand-reserved-instance-usage.md).

## <a name="change-a-reservation-after-purchase"></a>Cambiar una reserva después de la compra

Puede realizar los siguientes tipos de cambios en una reserva después de haberla comprado:

- Actualización del ámbito de la reserva

- Flexibilidad del tamaño de instancia (si procede)

- Propiedad

También puede dividir una reserva en fragmentos más pequeños y combinar reservas ya divididas. Ninguno de estos cambios conlleva una nueva transacción comercial ni un cambio en la fecha de finalización de la reserva.

No se pueden realizar los siguientes tipos de cambios tras la compra:

- La región de una reserva existente

- SKU

- Cantidad

- Duration

Pero sí se puede  *intercambiar* una reserva si quiere realizar cambios.

## <a name="cancel-exchange-or-refund-reservations"></a>Cancelación, intercambio o reembolso de reservas

Puede cancelar, intercambiar o reembolsar reservas con ciertas limitaciones. Para más información, consulte  [Autoservicio de intercambios y reembolsos de reservas de Azure](../cost-management-billing/reservations/exchange-and-refund-azure-reservations.md).

## <a name="need-help-contact-us"></a>¿Necesita ayuda? Póngase en contacto con nosotros.

Si tiene alguna pregunta o necesita ayuda,  [cree una solicitud de soporte técnico](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest).

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre cómo administrar una reserva, consulte  [Administración de Azure Reservations](../cost-management-billing/reservations/manage-reserved-vm-instance.md).

Para obtener más información acerca de Azure Reservations, consulte los siguientes artículos:

- [¿Qué es Azure Reservations?](../cost-management-billing/reservations/save-compute-costs-reservations.md)

- [Uso de instancias de Azure Dedicated Host](./windows/dedicated-hosts.md)

- [Precios de Azure Dedicated Host](https://azure.microsoft.com/pricing/details/virtual-machines/dedicated-host/)

- [Administración de reservas en Azure](../cost-management-billing/reservations/manage-reserved-vm-instance.md)

- [Información sobre cómo se aplica el descuento por la reserva](../cost-management-billing/manage/understand-vm-reservation-charges.md)

- [Información sobre el uso de reservas de Azure para suscripciones de pago por uso](../cost-management-billing/reservations/understand-reserved-instance-usage.md)

- [Información sobre el uso de reservas para la inscripción Enterprise](../cost-management-billing/reservations/understand-reserved-instance-usage-ea.md)

- [Costos de software de Windows no incluidos con reservas](../cost-management-billing/reservations/reserved-instance-windows-software-costs.md)

- [Azure Reservations en el programa del Proveedor de soluciones en la nube (CSP) del Centro de partners](/partner-center/azure-reservations)