---
title: Optimizar los costos de Azure Disk Storage con reservas
description: Obtenga información sobre la compra de reservas de Azure Disk Storage para ahorrar costos en discos administrados SSD Premium.
author: roygara
ms.author: rogarana
ms.date: 01/30/2020
ms.topic: how-to
ms.service: virtual-machines
ms.subservice: disks
ms.openlocfilehash: 9a6a56491a327d5d4eafc2c05dc2b54b137096b8
ms.sourcegitcommit: 62717591c3ab871365a783b7221851758f4ec9a4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/22/2020
ms.locfileid: "88752062"
---
# <a name="reduce-costs-with-azure-disks-reservation"></a>Reducción de costos con la reserva de discos de Azure

Ahorre en el uso de Azure Disk Storage con capacidad reservada. Las reservas de Azure Disk Storage combinadas con Azure Reserved Virtual Machine Instances permiten reducir el costo total de la máquina virtual. El descuento de reserva se aplica automáticamente a los discos coincidentes en el ámbito de reserva seleccionado. Debido a esta aplicación automática, no es necesario asignar una reserva a un disco administrado para obtener los descuentos.

Los descuentos se aplican cada hora según el uso del disco. La capacidad reservada no utilizada no se transfiere. El descuento de la reserva de Azure Disk Storage no se aplica a los discos no administrados, discos ultra o consumo de blobs en páginas.

## <a name="determine-your-storage-needs"></a>Determinación de las necesidades de almacenamiento

Antes de adquirir una reserva, debe determinar cuáles son sus necesidades de almacenamiento. Actualmente, las reservas de Azure Disk Storage solo están disponibles para las SKU de SSD Premium de Azure. La SKU de un disco SSD Premium determina el tamaño y el rendimiento del disco.

A la hora de determinar las necesidades de almacenamiento, no piense en discos basados solo en la capacidad. Por ejemplo, no puede tener una reserva para un disco P40 y usarla para pagar dos discos P30 más pequeños. Al adquirir una reserva, solo se compra una reserva para el número total de discos por SKU.

Se realiza una reserva de disco por SKU de disco. Como resultado, el consumo de reserva se basa en la unidad de las SKU de disco en lugar del tamaño proporcionado.

Por ejemplo, supongamos que reserva un disco P40 con 2 TiB de capacidad de almacenamiento aprovisionada. Supongamos también que asigna solo dos discos P30. La reserva P40 en ese caso no tiene en cuenta el consumo de P30 y usted paga la tarifa de pago por uso en los discos P30.
<br/>
<br/>
[!INCLUDE [disk-storage-premium-ssd-sizes](../../includes/disk-storage-premium-ssd-sizes.md)]

## <a name="purchase-considerations"></a>Consideraciones de compra

Se recomiendan los siguientes procedimientos al considerar la compra de la reserva de discos:

- Analice su información de uso para averiguar qué reservas debe adquirir. Asegúrese de que realiza el seguimiento del uso en las SKU de disco en lugar de la capacidad de disco aprovisionada o usada.
- Examine la reserva de discos junto con la reserva de la máquina virtual. Se recomienda encarecidamente que se realice la reserva para el uso de disco y de la máquina virtual para un ahorro máximo. Puede empezar con la determinación de la reserva de la máquina virtual correcta y, después, evaluar la reserva de discos. Por lo general, tendrá una configuración estándar para cada una de las cargas de trabajo. Por ejemplo, un servidor de SQL Server puede tener dos discos de datos P40 y un disco de sistema operativo P30.
  
  Este tipo de patrón puede ayudarle a determinar la cantidad reservada que puede comprar. Este enfoque puede simplificar el proceso de evaluación y asegurar también de que dispone de un plan alineado para máquinas virtuales y discos. El plan contiene consideraciones como suscripciones o regiones.

## <a name="purchase-restrictions"></a>Restricciones de compra

Los descuentos de reserva no están disponibles actualmente para lo siguiente:

- Blobs en páginas o discos no administrados
- SSD estándar o unidades de disco duro estándar (HDD).
- SKU de discos SSD Premium menores que P30: SKU de SSD P1, P2, P3, P4, P6, P10, P15 y P20.
- Discos en las regiones Azure Government, Azure Alemania o Azure China.

En algunas circunstancias poco frecuentes, Azure limita la compra de nuevas reservas a un subconjunto de SKU de discos debido a que la capacidad en una región es baja.

## <a name="buy-a-disk-reservation"></a>Compra de una reserva de discos

Puede adquirir reservas de Azure Disk Storage mediante [Azure Portal](https://portal.azure.com/). Puede pagar la reserva por adelantado o de manera mensual. Para más información sobre cómo comprar con pagos mensuales, consulte [Compra de reservas con pagos mensuales](../cost-management-billing/reservations/prepare-buy-reservation.md#buy-reservations-with-monthly-payments).

Siga estos pasos para adquirir la capacidad reservada:

1. Vaya al panel [Comprar reservas](https://portal.azure.com/#blade/Microsoft_Azure_Reservations/CreateBlade/referrer/Browse_AddCommand) en Azure Portal.

1. Seleccione **Azure Managed Disks** para comprar una reserva.

    ![Panel para reservas de compras](media/disks-reserved-capacity/disks-reserved-purchase-reservation.png) 

1. Especifique los valores obligatorios tal como se describe en la tabla siguiente:

   |Elemento  |Descripción  |
   |---------|---------|
   |**Ámbito**   |  Cuántas suscripciones pueden usar la ventaja de facturación asociada con la reserva. Este valor también especifica cómo se aplica la reserva a suscripciones concretas. <br/><br/> Si selecciona **Compartido**, el descuento de la reserva se aplica a la capacidad de Azure Storage en todas las suscripciones en el contexto de facturación. El contexto de facturación se basa en cómo se haya suscrito a Azure. Para los clientes Enterprise, el ámbito compartido es la inscripción e incluye todas las suscripciones que esta contiene. Para los clientes de pago por uso, el ámbito compartido incluye todas las suscripciones con tarifas de pago por uso creadas por el administrador de la cuenta.  <br/><br/>  Si selecciona **Suscripción única**, el descuento de reserva se aplica a la capacidad de Azure Storage de la suscripción seleccionada. <br/><br/> Si selecciona **Grupo de recursos único**, el descuento de reserva se aplica a la capacidad de Azure Storage de la suscripción seleccionada y al grupo de recursos seleccionado de esa suscripción. <br/><br/> Puede cambiar el ámbito de reserva después de comprar la reserva.  |
   |**Suscripción**  | La suscripción que se usa para pagar la reserva de Azure Storage. El método de pago en la suscripción seleccionada se usa al cargar los costos. La suscripción debe ser uno de los tipos siguientes:<br/><ul><li> Contrato Enterprise (números de oferta MS-AZR-0017P y MS-AZR-0148P). Para una suscripción Enterprise, los cargos se deducen del saldo de compromiso monetario de la inscripción o se cobran como uso por encima del límite.</li><br/><li>Suscripción individual con las tarifas de pago por uso (números de oferta MS-AZR-0003P y MS-AZR-0023P). en una suscripción individual con tarifas de pago por uso, los cargos se cobran en el método de pago de tarjeta de crédito o factura de la suscripción.</li></ul>    |
   | **Discos** | La SKU que desea crear. |
   | **Región** | La región donde está en vigor la reserva. |
   | **Frecuencia de facturación** | Frecuencia con la que se factura la cuenta para la reserva. Entre las opciones se incluyen **Mensual** y **Por adelantado**. |

    ![Panel para seleccionar el producto que quiere comprar](media/disks-reserved-capacity/premium-ssd-reserved-purchase-selection.png)

1. Después de especificar los valores para la reserva, Azure Portal muestra el costo. El portal también muestra el porcentaje de descuento en la facturación de pago por uso. Seleccione **Siguiente** para continuar con el panel **Comprar reservas**.

1. En el panel **Comprar reservas**, puede asignar un nombre a la reserva y seleccionar la cantidad total de reservas que quiere crear. El número de reservas se asigna al número de discos. Por ejemplo, si quiere reservar cien discos, escriba en **Cantidad** un valor de **100**.

1. Revise el costo total de la reserva.

    ![Panel Comprar reservas](media/disks-reserved-capacity/premium-ssd-reserved-selecting-sku-total-purchase.png)

Después de comprar una reserva, se aplica automáticamente a cualquier recurso de Disk Storage que coincida con los términos de la reserva. Si aún no ha creado ningún recurso de Disk Storage, se aplicará la reserva siempre que cree un recurso que coincida con los términos de la reserva. En cualquier caso, el período de la reserva comienza inmediatamente después de una compra correcta.

## <a name="cancel-exchange-or-refund-reservations"></a>Cancelación, intercambio o reembolso de reservas

Puede cancelar, intercambiar o reembolsar reservas con ciertas limitaciones. Para más información, consulte [Autoservicio de intercambios y reembolsos de reservas de Azure](https://docs.microsoft.com/azure/cost-management-billing/reservations/exchange-and-refund-azure-reservations).

## <a name="expiration-of-a-reservation"></a>Expiración de una reserva

Cuando expira una reserva, cualquier capacidad de Azure Disk Storage que use en esa reserva se factura según la tarifa de pago por uso. Las reservas no se renuevan automáticamente.

Recibirá una notificación por correo electrónico 30 días antes de la expiración de la reserva y nuevamente en la fecha de expiración. Para seguir aprovechando el ahorro de costos que proporciona una reserva, debe renovarla a más tardar en la fecha de expiración.

## <a name="need-help-contact-us"></a>¿Necesita ayuda? Ponerse en contacto con nosotros

Si tiene alguna pregunta o necesita ayuda, [cree una solicitud de soporte técnico](https://go.microsoft.com/fwlink/?linkid=2083458).

## <a name="next-steps"></a>Pasos siguientes

- [¿Qué es Azure Reservations?](../cost-management-billing/reservations/save-compute-costs-reservations.md)
- [Aplicación del descuento por reserva a Azure Disk Storage](../cost-management-billing/reservations/understand-disk-reservations.md)
