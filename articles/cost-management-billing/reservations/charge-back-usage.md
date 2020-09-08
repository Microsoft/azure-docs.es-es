---
title: Contracargo de costos de Reserva de Azure
description: Aprenda a ver los costos de Reserva de Azure para el contracargo.
author: yashesvi
ms.service: cost-management-billing
ms.subservice: reservations
ms.topic: how-to
ms.date: 07/24/2020
ms.author: banders
ms.openlocfilehash: 4387a1aa5b22c9808f36364b71f89a71736958b8
ms.sourcegitcommit: 56cbd6d97cb52e61ceb6d3894abe1977713354d9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88689480"
---
# <a name="charge-back-azure-reservation-costs"></a>Contracargo de costos de Reserva de Azure

Los lectores de facturación de Contrato Enterprise y Contrato de cliente de Microsoft pueden ver los datos de los costos amortizados para las reservas. Pueden utilizar los datos de costo para realizar un contracargo del valor monetario de una suscripción, un grupo de recursos, un recurso o una etiqueta a sus asociados. En los datos amortizados, el precio efectivo es el costo de la reserva por hora prorrateada. El costo es el costo total de uso de la reserva por parte del recurso en ese día.

Los usuarios con una suscripción individual pueden obtener los datos de costos amortizados en su archivo de uso. Cuando un recurso obtiene un descuento por reserva, la sección *AdditionalInfo* del archivo de uso contiene los detalles de la reserva. Para más información, consulte [Descarga de uso en Azure Portal](https://docs.microsoft.com/azure/cost-management-billing/understand/download-azure-daily-usage#download-usage-from-the-azure-portal-csv).

## <a name="get-reservation-charge-back-data-for-chargeback"></a>Obtención de datos de la reserva para realizar el contracargo

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. Vaya a **Administración de costos + facturación**.
1. En **Costo real**, seleccione la métrica **Costo amortizado**.
1. Para ver qué recursos ha usado una reserva, aplique un filtro para **Reserva** y, después, seleccione reservas.
1. Establezca la **Granularidad** en **Mensual** o **Diaria**.
1. Establezca el tipo de gráfico en **Tablas**.
1. En la opción **Agrupar por**, seleccione **Recurso**.

[![Ejemplo que muestra los costos de recursos de reserva que se pueden usar para el contracargo](./media/charge-back-usage/amortized-reservation-costs.png)](./media/charge-back-usage/amortized-reservation-costs.png#lightbox)

En este vídeo se muestra cómo ver los costos de uso de la reserva en Azure Portal.

 > [!VIDEO https://www.microsoft.com/videoplayer/embed/RE4sQOw] 

## <a name="need-help-contact-us"></a>¿Necesita ayuda? Póngase en contacto con nosotros.

Si tiene alguna pregunta o necesita ayuda, [cree una solicitud de soporte técnico](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest).

## <a name="next-steps"></a>Pasos siguientes

- Para más información sobre cómo administrar una reserva, consulte [Administración de Azure Reservations](manage-reserved-vm-instance.md).
- Para obtener más información acerca de Azure Reservations, consulte los siguientes artículos:
  - [¿Qué es Azure Reservations?](save-compute-costs-reservations.md)
  - [Administración de reservas en Azure](manage-reserved-vm-instance.md)
