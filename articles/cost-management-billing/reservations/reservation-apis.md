---
title: API para la automatización de reservas de Azure | Microsoft Docs
description: Obtenga información sobre las API de Azure que puede usar para obtener información de reservas mediante programación.
author: yashesvi
ms.reviewer: yashesvi
tags: billing
ms.service: cost-management-billing
ms.subservice: reservations
ms.topic: conceptual
ms.date: 02/13/2020
ms.author: banders
ms.openlocfilehash: cffb753d515c4d066c8ce52e7dacebe5a8f6f8ca
ms.sourcegitcommit: 56cbd6d97cb52e61ceb6d3894abe1977713354d9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88682644"
---
# <a name="apis-for-azure-reservation-automation"></a>API para la automatización de reservas de Azure

Use las API de Azure para obtener información mediante programación para su organización sobre el servicio de Azure o las reservas de software.

## <a name="find-reservation-plans-to-buy"></a>Busque planes de reserva para comprar

Use la API de recomendación de reservas para obtener recomendaciones sobre qué plan de reservas puede comprar según el uso de su organización. Para obtener más información, consulte [Get reservation recommendations](/rest/api/billing/enterprise/billing-enterprise-api-reserved-instance-recommendation) (Obtener recomendaciones de reserva).

También puede analizar el uso de sus recursos mediante el detalle de uso de la API de consumo. Para obtener más información, consulte [Usage Details - List For Billing Period By Billing Account](/rest/api/consumption/usagedetails/list#billingaccountusagedetailslistforbillingperiod-legacy) (Detalles de uso: lista para el período de facturación en función de la cuenta de facturación). Los recursos de Azure que usa de forma sistemática suelen ser los mejores candidatos para realizar una reserva.

## <a name="buy-a-reservation"></a>Adquisición de una reserva

Puede comprar reservas de Azure y planes de software mediante programación con las API REST. Para obtener más información, consulte [API para pedido de reserva: compra](/rest/api/reserved-vm-instances/reservationorder/purchase).

A continuación se muestra una solicitud de ejemplo para comprar mediante una API REST:

```
PUT https://management.azure.com/providers/Microsoft.Capacity/reservationOrders/<GUID>?api-version=2019-04-01
```

Cuerpo de la solicitud:

```
{
 "sku": {
    "name": "standard_D1"
  },
 "location": "westus",
 "properties": {
    "reservedResourceType": "VirtualMachines",
    "billingScopeId": "/subscriptions/ed3a1871-612d-abcd-a849-c2542a68be83",
    "term": "P1Y",
    "quantity": "1",
    "displayName": "TestReservationOrder",
    "appliedScopes": null,
    "appliedScopeType": "Shared",
    "reservedResourceProperties": {
      "instanceFlexibility": "On"
    }
  }
}
```

También puede comprar una reserva en Azure Portal. Para más información, consulte los siguientes artículos.

Planes de servicio:
- [Máquina virtual](../../virtual-machines/windows/prepay-reserved-vm-instances.md?toc=/azure/billing/TOC.json)
-  [Cosmos DB](../../cosmos-db/cosmos-db-reserved-capacity.md?toc=/azure/billing/TOC.json)
- [SQL Database](../../azure-sql/database/reserved-capacity-overview.md?toc=/azure/billing/TOC.json)

Planes de software:
- [Software de SUSE Linux](../../virtual-machines/linux/prepay-suse-software-charges.md?toc=/azure/billing/TOC.json)

## <a name="get-reservations"></a>Obtener reservas

Si es un cliente de Azure con un Contrato Enterprise (cliente de EA), puede obtener las reservas que compró su organización mediante la [Get Reserved Instance transaction charges API](/rest/api/billing/enterprise/billing-enterprise-api-reserved-instance-charges). Para otras suscripciones, obtenga la lista de reservas que compró y los permisos para verlas mediante la API [Reservation Order - List](/rest/api/reserved-vm-instances/reservationorder/list). De forma predeterminada, el propietario de la cuenta o la persona que compró la reserva tiene permisos para verla.

## <a name="see-reservation-usage"></a>Ver el uso de las reservas

Si es un cliente con contrato Enterprise, puede ver mediante programación cómo se usan las reservas en su organización. Para obtener más información, consulte [Get Reserved Instance usage for enterprise customers](/rest/api/billing/enterprise/billing-enterprise-api-reserved-instance-usage) (Uso de Get Reserved Instance para clientes empresariales). Para otras suscripciones, use la API [Reservations Summaries - List By Reservation Order And Reservation](/rest/api/consumption/reservationssummaries/listbyreservationorderandreservation).

Si ve que las reservas de su organización están siendo usadas por debajo de su capacidad:

- Asegúrese de que las máquinas virtuales que crea su organización coincidan con el tamaño de la máquina virtual que se encuentra en la reserva.
- Asegúrese de que la flexibilidad de tamaño de instancia esté activada. Para obtener más información, consulte [Administrar reservas: cambiar la configuración de optimización para instancias reservadas de máquina virtual](manage-reserved-vm-instance.md#change-optimize-setting-for-reserved-vm-instances).
- Cambie el ámbito de la reserva para que sea compartido y así poder aplicarlo de manera más amplia. Para obtener más información, consulte [Administrar reservas: cambio del ámbito de una reserva](manage-reserved-vm-instance.md#change-the-reservation-scope).
- Cambie la cantidad que no haya usado. Para más información, consulte [Administración de reservas](manage-reserved-vm-instance.md).

## <a name="give-access-to-reservations"></a>Dar acceso a las reservas

Obtenga la lista de todas las reservas a las que un usuario tiene acceso mediante [Reservation - Operation - List API](/rest/api/reserved-vm-instances/reservationorder/list). Para dar acceso a una reserva mediante programación, consulte uno de los siguientes artículos:

- [Administración del acceso mediante RBAC y API REST](../../role-based-access-control/role-assignments-rest.md)
- [Administración de acceso mediante RBAC y Azure PowerShell](../../role-based-access-control/role-assignments-powershell.md)
- [Administración de acceso mediante RBAC y la CLI de Azure](../../role-based-access-control/role-assignments-cli.md)

## <a name="split-or-merge-reservation"></a>Dividir o combinar una reserva

Si se compran varias instancias de recursos en una reserva, se pueden asignar algunas de las instancias de la reserva a distintas suscripciones. Puede cambiar el ámbito de la reserva para que se aplique a todas las suscripciones dentro del mismo contexto de facturación. Pero si quiere usarla para fines de administración de costos o para presupuestos, le recomendamos que mantenga el ámbito como "suscripción única" y que asigne instancias de reserva a una suscripción específica.

Para dividir una reserva, use la API [Reservation - Split](/rest/api/reserved-vm-instances/reservation/split). También puede dividir una reserva mediante PowerShell. Para obtener más información, consulte [Administrar reservas: división de una reserva única en dos](manage-reserved-vm-instance.md#split-a-single-reservation-into-two-reservations).

Para combinar dos reservas en una reserva, use la API [Reservation - Merge](/rest/api/reserved-vm-instances/reservation/merge).

## <a name="change-scope-for-a-reservation"></a>Cambiar el ámbito de una reserva

El ámbito de una reserva puede ser una suscripción única, un grupo de recursos único o todas las suscripciones del contexto de facturación. Si establece que el ámbito es una suscripción única o un grupo de recursos único, la reserva se corresponderá con los recursos que se ejecuten en la suscripción seleccionada. Si elimina o mueve la suscripción o el grupo de recursos, no se usará la reserva.  Si establece que el ámbito es compartido, Azure asociará la reserva con los recursos que se ejecuten en todas las suscripciones dentro del contexto de facturación. El contexto de facturación depende de la suscripción que se usó para comprar la reserva. Puede seleccionar el ámbito en la compra o cambiarlo en cualquier momento después de la misma. Para obtener más información, consulte [Administrar reservas: cambio del ámbito](manage-reserved-vm-instance.md#change-the-reservation-scope).

Para cambiar el ámbito mediante programación, use la API [Reservation - Update](/rest/api/reserved-vm-instances/reservation/update).

## <a name="learn-more"></a>Más información

- [Qué son las reservas de Azure](save-compute-costs-reservations.md)
- [Información sobre cómo se aplica el descuento por la reserva de máquinas virtuales](../manage/understand-vm-reservation-charges.md)
- [Descubra cómo se aplica el descuento del plan de software SUSE Linux Enterprise](understand-suse-reservation-charges.md)
- [Descubra cómo se aplican otros descuentos por reservas](understand-reservation-charges.md)
- [Información sobre el uso de reservas para suscripciones de pago por uso](understand-reserved-instance-usage.md)
- [Información sobre el uso de reservas para la inscripción Enterprise](understand-reserved-instance-usage-ea.md)
- [Costos de software de Windows no incluidos con reservas](reserved-instance-windows-software-costs.md)
- [Azure Reservations en el programa del Proveedor de soluciones en la nube (CSP) del Centro de partners](https://docs.microsoft.com/partner-center/azure-reservations)
