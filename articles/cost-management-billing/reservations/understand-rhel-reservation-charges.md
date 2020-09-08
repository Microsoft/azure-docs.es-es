---
title: 'Descuentos del plan de reserva de Red Hat: Azure'
description: Obtenga información sobre cómo se aplican los descuentos del plan de Red Hat en el software de Red Hat en máquinas virtuales.
author: yashesvi
ms.service: cost-management-billing
ms.subservice: reservations
ms.topic: conceptual
ms.date: 06/11/2020
ms.author: banders
ms.openlocfilehash: 4314a52488473bcacd4aa62a0e8f44d88d2b75f1
ms.sourcegitcommit: 56cbd6d97cb52e61ceb6d3894abe1977713354d9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88681607"
---
# <a name="understand-how-the-red-hat-linux-enterprise-software-reservation-plan-discount-is-applied-for-azure"></a>Comprenda cómo se aplica para Azure el descuento del plan de reserva de software Red Hat Linux Enterprise).

Después de comprar un plan Red Hat Linux, el descuento se aplica automáticamente a las máquinas virtuales Red Hat implementadas que coinciden con la reserva. Un plan Red Hat Linux cubre el costo de ejecutar el software de Red Hat en una máquina virtual de Azure.

Para comprar el plan Red Hat Linux adecuado, debe saber qué máquinas virtuales Red Hat ejecuta y el número de vCPU de dichas máquinas virtuales. Use las secciones siguientes para ayudar a identificar, a partir del archivo CSV de uso, el plan que debe comprar.

## <a name="discount-applies-to-different-vm-sizes"></a>El descuento se aplica a los diferentes tamaños de máquinas virtuales

Del mismo modo que con las instancias reservadas de máquinas virtuales, las compras de planes Red Hat ofrecen flexibilidad de tamaño de instancia. Esto significa que el descuento se aplica incluso cuando se implementa una VM con un recuento de vCPU diferente. El descuento se aplica a los diferentes tamaños de VM del plan de software.

El importe de descuento depende de la relación que aparece en las tablas siguientes. La relación compara la superficie relativa de cada tamaño de máquina virtual del grupo. La relación depende de las vCPU de VM. Use el valor de relación para calcular cuántas instancias de máquinas virtuales obtienen el descuento del plan Red Hat Linux.

Por ejemplo, si compra un plan para Red Hat Linux Enterprise Server para una máquina virtual con entre 1 y 4 CPU virtuales, la relación de esa reserva es 1. El descuento cubre el costo de software de Red Hat para:

- 1 máquina virtual implementada con entre 1 y 4 CPU virtuales,
- o bien 0,46 o aproximadamente el 4 % de los costos de Red Hat Enterprise Linux de una máquina virtual con 5, o más, CPU virtuales.

### <a name="red-hat-enterprise-linux"></a>Red Hat Enterprise Linux

Nombres de marketplace de Azure Portal:
- Red Hat Enterprise Linux 6.7
- Red Hat Enterprise Linux 6.8
- Red Hat Enterprise Linux 6.9
- Red Hat Enterprise Linux 6.10
- Red Hat Enterprise Linux 7
- Red Hat Enterprise Linux 7.2
- Red Hat Enterprise Linux 7.3
- Red Hat Enterprise Linux 7.4
- Red Hat Enterprise Linux 7.5
- Red Hat Enterprise Linux 7.6
- Red Hat Enterprise Linux 8.2

[Compruebe los medidores de Red Hat Enterprise Linux a los que se aplica el plan](https://isfratio.blob.core.windows.net/isfratio/RHELRatios.csv)

## <a name="next-steps"></a>Pasos siguientes

Para más información acerca de las reservas, consulte los siguientes artículos:

- [Qué son las reservas de Azure](save-compute-costs-reservations.md)
- [Pago por adelantado para planes de software de Red Hat con reservas de Azure](../../virtual-machines/linux/prepay-rhel-software-charges.md)
- [Pago por adelantado de máquinas virtuales con Azure Reserved VM Instances](../../virtual-machines/windows/prepay-reserved-vm-instances.md)
- [Administración de reservas para Azure](manage-reserved-vm-instance.md)
- [Información sobre el uso de reservas para suscripciones de pago por uso](understand-reserved-instance-usage.md)
- [Información sobre el uso de reservas para la inscripción Enterprise](understand-reserved-instance-usage-ea.md)

## <a name="need-help-contact-us"></a>¿Necesita ayuda? Ponerse en contacto con nosotros

Si tiene alguna pregunta o necesita ayuda, [cree una solicitud de soporte técnico](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest).
