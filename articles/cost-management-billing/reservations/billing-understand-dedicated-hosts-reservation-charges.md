---
title: Información sobre el descuento de instancias reservadas de Azure Dedicated Host
description: Obtenga información sobre cómo se aplica el descuento de instancias reservadas de máquina virtual de Azure a las instancias de Azure Dedicated Host.
author: yashesvi
ms.service: cost-management-billing
ms.subservice: reservations
ms.topic: conceptual
ms.date: 02/28/2020
ms.author: banders
ms.openlocfilehash: f9080d13c4358a03b3f13ffcf346af42e49c47c6
ms.sourcegitcommit: 56cbd6d97cb52e61ceb6d3894abe1977713354d9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88689463"
---
# <a name="how-the-azure-reservation-discount-is-applied-to-azure-dedicated-hosts"></a>Aplicación del descuento de la reserva de Azure a instancias de Azure Dedicated Host

Después de comprar una instancia reservada de Azure Dedicated Host, el descuento de reserva se aplica automáticamente a los hosts dedicados que coincidan con los atributos y la cantidad de la reserva. Una reserva cubre los costos de proceso de los hosts dedicados.

## <a name="how-reservation-discount-is-applied"></a>Aplicación del descuento por reserva

Un descuento de reserva significa "*usarlo o perderlo*". Por lo tanto, si no tiene recursos coincidentes para ninguna hora, perderá una cantidad de reserva para esa hora. No se pueden arrastrar las horas reservadas no utilizadas.

Al eliminar un host dedicado, el descuento por reserva se aplica automáticamente a otro que coincida con el ámbito especificado. Si no se encuentran recursos coincidentes en el ámbito especificado, las horas reservadas se *pierden*.

## <a name="reservation-discount-for-dedicated-hosts"></a>Descuento de reserva para host dedicados

La instancia reservada de Azure Dedicated Host proporciona un descuento en el costo de la infraestructura de proceso que se usa con los hosts dedicados. El descuento se aplica a los hosts dedicados, independientemente de si las máquinas virtuales lo usan, o no. La reserva no cubre costos adicionales como la concesión de licencias, el uso de la red o el consumo de almacenamiento por parte de la máquina virtual implementada en el host dedicado.

## <a name="need-help-contact-us"></a>¿Necesita ayuda? Ponerse en contacto con nosotros

Si tiene alguna pregunta o necesita ayuda,  [cree una solicitud de soporte técnico](https://go.microsoft.com/fwlink/?linkid=2083458).

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información acerca de Azure Reservations, consulte los siguientes artículos:

- [¿Qué son las reservas para Azure?](https://docs.microsoft.com/azure/billing/billing-save-compute-costs-reservations)

- [Uso de instancias de Azure Dedicated Host](https://docs.microsoft.com/azure/virtual-machines/windows/dedicated-hosts)

- [Precios de hosts dedicados](https://azure.microsoft.com/pricing/details/virtual-machines/dedicated-host/)

- [Administración de reservas para Azure](https://docs.microsoft.com/azure/billing/billing-manage-reserved-vm-instance)

- [Información sobre el uso de reservas para suscripciones de pago por uso](https://docs.microsoft.com/azure/billing/billing-understand-reserved-instance-usage)

- [Información sobre el uso de reservas para la inscripción Enterprise](https://docs.microsoft.com/azure/billing/billing-understand-reserved-instance-usage-ea)

- [Información sobre el uso de reservas para suscripciones de CSP](https://docs.microsoft.com/partner-center/azure-reservations)

