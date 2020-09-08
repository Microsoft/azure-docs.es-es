---
title: Visualización de reservas de recursos de Azure | Microsoft Docs
description: Aprenda a ver las reservas de Azure Reservations en Azure Portal. Consulte la información sobre reservas y uso con las API, PowerShell, la CLI y Power BI.
author: yashesvi
ms.reviewer: yashar
ms.service: cost-management-billing
ms.subservice: reservations
ms.topic: how-to
ms.date: 07/24/2020
ms.author: banders
ms.openlocfilehash: ec91d0997717d536c3b47ce7b276f75b21d1baa1
ms.sourcegitcommit: 56cbd6d97cb52e61ceb6d3894abe1977713354d9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88681658"
---
# <a name="view-azure-reservations"></a>Visualización de reservas de Azure

Puede ver y administrar la reserva comprada en Azure Portal.

## <a name="permissions-to-view-a-reservation"></a>Permisos para ver una reserva

Para ver o administrar una reserva, debe tener permiso de lectura o de propietario en ella. De forma predeterminada, al comprar una reserva, el usuario y el administrador de cuenta obtienen automáticamente el rol de propietario en el pedido de la reserva y en la reserva. Para permitir que otras personas vean la reserva, debe agregarlos a ella como **propietario** o **lector** en el pedido de la reserva o reserva. Al agregar a un usuario a la suscripción que se proporciona para la facturación de la reserva, no se le agrega automáticamente a la reserva. 

Para más información, consulte [Agregar o cambiar los usuarios que pueden administrar una reserva](manage-reserved-vm-instance.md#add-or-change-users-who-can-manage-a-reservation).

## <a name="view-reservation-and-utilization-in-the-azure-portal"></a>Visualización de la reserva y el uso en Azure Portal

Para ver una reserva como propietario o lector

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
2. Vaya a [Reservas](https://portal.azure.com/#blade/Microsoft_Azure_Reservations/ReservationsBrowseBlade).
3. La lista mostrará todas las reservas donde tenga el rol de propietario o lector. Cada reserva muestra el último porcentaje de utilización conocido.
4. Haga clic en el porcentaje de uso para ver el historial y los detalles de uso. Consulte los detalles en el vídeo siguiente.
   > [!VIDEO https://www.microsoft.com/en-us/videoplayer/embed/RE4sYwk] 

## <a name="get-reservations-and-utilization-using-apis-powershell-cli"></a>Obtención de reservas y uso con API, PowerShell y la CLI

Obtenga una lista de todas las reservas con los siguientes recursos:
- [API: Pedido de reserva: lista](/rest/api/reserved-vm-instances/reservationorder/list)
- [PowerShell: Pedido de reserva: lista](/powershell/module/azurerm.reservations/get-azurermreservationorder)
- [CLI: Pedido de reserva: lista](/cli/azure/reservations/reservation-order#az-reservations-reservation-order-list)

También puede obtener el [uso de la reserva](/rest/api/billing/enterprise/billing-enterprise-api-reserved-instance-usage) mediante la API de uso de la instancia reservada. 

## <a name="see-reservations-and-utilization-in-power-bi"></a>Visualización de reservas y uso en Power BI

Los usuarios de Power BI tienen dos opciones:
- Paquete de contenido: en el [paquete de contenido de Power BI de Consumption Insights](/power-bi/desktop-connect-azure-cost-management) están disponibles los datos de uso y compras de reservas. Cree los informes que desee con este paquete de contenido. 
- Aplicación Cost Management: use la [aplicación Cost Management](https://appsource.microsoft.com/product/power-bi/costmanagement.azurecostmanagementapp) para obtener informes creados previamente que puede personalizar.

## <a name="next-steps"></a>Pasos siguientes

- [Administración de reservas de Azure](manage-reserved-vm-instance.md)
- [Información sobre el uso de reservas para suscripciones de pago por uso](understand-reserved-instance-usage.md)
- [Información sobre el uso de reservas para la inscripción Enterprise](understand-reserved-instance-usage-ea.md)
- [Información sobre el uso de reservas para suscripciones de CSP](https://docs.microsoft.com/partner-center/azure-reservations)

## <a name="need-help-contact-us"></a>¿Necesita ayuda? Ponerse en contacto con nosotros

Si tiene alguna pregunta o necesita ayuda, [cree una solicitud de soporte técnico](https://go.microsoft.com/fwlink/?linkid=2083458).
