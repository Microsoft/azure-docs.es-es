---
title: Información sobre el descuento por reservas para Azure Disk Storage
description: Aprenda cómo se aplica un descuento por discos reservados de Azure a los discos administrados SSD premium de Azure.
author: roygara
ms.author: rogarana
ms.service: cost-management-billing
ms.subservice: reservations
ms.topic: conceptual
ms.date: 01/30/2020
ms.openlocfilehash: cd89c3ddc3c58de02f1104109ce7f243c4d1e6fd
ms.sourcegitcommit: 56cbd6d97cb52e61ceb6d3894abe1977713354d9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88682457"
---
# <a name="understand-how-your-reservation-discount-is-applied-to-azure-disk-storage"></a>Aplicación de un descuento por reserva a Azure Disk Storage

Después de comprar capacidad reservada de discos de Azure, se aplica automáticamente un descuento por reserva a los recursos de disco que coincidan con los términos de la reserva. Esta reserva solo se aplica a los SKU de disco. Las instantáneas de discos se cobran según las tarifas de pago por uso.

Para más información sobre la reserva de discos de Azure, consulte [Reducción de costos con la reserva de discos de Azure](../../virtual-machines/disks-reserved-capacity.md). Para más información sobre los precios de la reserva de discos de Azure, consulte los [precios de discos administrados de Azure](https://azure.microsoft.com/pricing/details/managed-disks/).

## <a name="how-the-reservation-discount-is-applied"></a>Aplicación del descuento por reserva

El descuento por la reserva de discos es un descuento para "usarlo o perderlo". Se aplica a los recursos de disco administrado cada hora. Durante una hora determinada, si no tiene ningún recurso de disco administrado que cumpla los términos de reserva, perderá una cantidad de reserva en esa hora. Las horas no utilizadas no se acumulan.

Al eliminar un recurso, el descuento por reserva se aplica automáticamente a otro que coincida con el ámbito especificado. Si no se encuentra ningún recurso coincidente, se perderán las horas reservadas.

## <a name="discount-examples"></a>Ejemplos de descuento

En los ejemplos siguientes se muestra cómo se aplica el descuento por reserva de discos de Azure en función de la implementación.

Supongamos que compra y reserva 100 discos P30 en la región Oeste de EE. UU. 2 durante un período de un año. Cada disco tiene aproximadamente 1 TiB de almacenamiento. Supongamos que el costo de esta reserva de ejemplo es de 140 100 USD. Puede optar por pagar la cantidad completa por adelantado o pagar cuotas mensuales fijas de 11 675 USD durante los próximos 12 meses.

En los siguientes escenarios se describe lo que sucede si el uso excede la capacidad reservada, no llega a esta o se usa por niveles. En estos ejemplos, se supone que se ha registrado en un plan de pago de reserva mensual.

### <a name="underusing-your-capacity"></a>Uso por debajo de la capacidad

Supongamos que solo implementa 99 de los 100 discos P30 de la unidad de estado sólido (SSD) premium de Azure durante una hora dentro del período de reserva. El disco P30 restante no se aplica durante esa hora, y tampoco se acumula.

### <a name="overusing-your-capacity"></a>Uso por encima de la capacidad

Supongamos que en una hora dentro del período de reserva, usó 101 discos P30 SSD premium. El descuento por reserva solo se aplica a 100 discos P30. El disco P30 restante se cobra según las tarifas de pago por uso durante esa hora. En la siguiente hora, si el uso disminuye a 100 discos P30, todo el uso queda cubierto por la reserva.

### <a name="tiering-your-capacity"></a>Uso de la capacidad por niveles

Supongamos que en una hora dada dentro del período de reserva quiere usar un total de 200 discos SSD premium P30. Supongamos también que usa solo 100 durante los primeros 30 minutos. Durante este período, el uso está totalmente cubierto, ya que hizo una reserva de 100 discos P30. Si después deja de usar los primeros 100 (de modo que usa cero) y, a continuación, comienza a usar los otros 100 durante los 30 minutos restantes, ese uso también está cubierto por la reserva.

![Ejemplo de uso por debajo de la capacidad, por encima de la capacidad y de la capacidad por niveles](media/understand-disk-reservations/reserved-disks-example-scenarios.png)

## <a name="need-help-contact-us"></a>¿Necesita ayuda? Ponerse en contacto con nosotros

Si tiene alguna pregunta o necesita ayuda, [cree una solicitud de soporte técnico](https://go.microsoft.com/fwlink/?linkid=2083458).

## <a name="next-steps"></a>Pasos siguientes

- [Reducción de costos con la reserva de discos de Azure](../../virtual-machines/disks-reserved-capacity.md)
- [¿Qué es Azure Reservations?](save-compute-costs-reservations.md)
