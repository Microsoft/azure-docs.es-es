---
title: Ahorre en SAP HANA (instancias grandes) con una reserva de Azure
description: Comprenda las cosas que debe saber antes de comprar una reserva de instancia grande de HANA y cómo realizar la compra.
author: yashesvi
ms.reviewer: yashar
ms.service: cost-management-billing
ms.subservice: reservations
ms.topic: how-to
ms.date: 07/24/2020
ms.author: banders
ms.openlocfilehash: 44f7ce657ea9341779e15f6e4817e8fae1515e47
ms.sourcegitcommit: 56cbd6d97cb52e61ceb6d3894abe1977713354d9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88685976"
---
# <a name="save-on-sap-hana-large-instances-with-an-azure-reservation"></a>Ahorre en SAP HANA (instancias grandes) con una reserva de Azure

Puede ahorrar en los costos de SAP HANA (instancias grandes) (HLI) cuando compre previamente reservas de Azure durante uno o tres años. El descuento de reserva se aplica a la SKU de HLI aprovisionada que coincide con la instancia reservada adquirida. Este artículo le ayuda a comprender las cosas que debe saber antes de comprar una reserva y cómo realizar la compra.

Al adquirir una reserva, confirma usar la HLI durante uno o tres años. La compra de capacidad reservada de HLI cubre el proceso y almacenamiento NFS incluido en la SKU. La reserva no incluye los costos de licencias de software, como el sistema operativo, SAP ni costos de almacenamiento adicionales. El descuento por reserva se aplica automáticamente a la SAP HLI aprovisionada. Cuando finalice el período de reserva, se aplicarán las tarifas de pago por uso al recurso aprovisionado.

## <a name="purchase-considerations"></a>Consideraciones de compra

Se debe aprovisionar una SKU de HLI antes de pasar por la compra de capacidad reservada. La reserva se paga por adelantado o de manera mensual. Las siguientes restricciones se aplican a la capacidad reservada de HLI:

- Los descuentos de reserva se aplican solo a Contrato Enterprise y a las suscripciones de Contrato de cliente de Microsoft. No se admiten otras suscripciones.
- No se admite la flexibilidad de tamaño de instancia para la capacidad reservada de HLI. Una reserva solo se aplica a la SKU y a la región para la que la compra.
- No se admiten la cancelación ni el intercambio de autoservicio.
- El ámbito de la capacidad reservada es único, por lo que se aplica a una única suscripción y grupo de recursos. La capacidad adquirida no se puede actualizar para que la use otra suscripción.
- No puede tener un ámbito de reserva compartido para la capacidad reservada de HANA. No se puede dividir, combinar ni actualizar el ámbito de la reserva.
- Puede adquirir una única HLI a la vez mediante las llamadas API de capacidad reservada. Haga llamadas API adicionales para comprar cantidades adicionales.

Puede adquirir capacidad reservada en Azure Portal o mediante la [API de REST](https://docs.microsoft.com/rest/api/reserved-vm-instances/reservationorder/purchase).

## <a name="buy-a-hana-large-instance-reservation"></a>Compra de una reserva de instancia grande de HANA

Use la siguiente información para comprar una reserva de HLI con las [API de REST de Pedido de reserva](https://docs.microsoft.com/rest/api/reserved-vm-instances/reservationorder/purchase).

### <a name="get-the-reservation-order-and-price"></a>Obtención del pedido y el precio de reserva

En primer lugar, obtenga el pedido y el precio de reserva de la SKU de la instancia grande de HANA aprovisionada mediante la API [Calculate Price](https://docs.microsoft.com/rest/api/reserved-vm-instances/reservationorder/calculate).

En el ejemplo siguiente se usa [armclient](https://github.com/projectkudu/ARMClient) para hacer llamadas de la API de REST con PowerShell. Aquí se muestran a qué deberían parecerse el pedido de reserva, la solicitud de la API Calculate Price y el cuerpo de la solicitud :

```azurepowershell-interactive
armclient post /providers/Microsoft.Capacity/calculatePrice?api-version=2018-06-01 "{
    'sku': {
        'name': 'SAP_HANA_On_Azure_S224om'
    },
    'location': 'eastus',
    'properties': {
        'reservedResourceType': 'SapHana',
        'billingScopeId': '/subscriptions/11111111-1111-1111-111111111111',
        'term': 'P1Y',
        'quantity': '1',
        'displayName': 'testreservation_S224om',
        'appliedScopes': ['/subscriptions/11111111-1111-1111-111111111111'],
        'appliedScopeType': 'Single',
       'instanceFlexibility': 'NotSupported'
    }
}"
```

Para obtener más información sobre los campos de datos y sus descripciones, consulte [Campos de reserva de HLI](#hli-reservation-fields).

La siguiente respuesta de ejemplo es similar a lo que se devuelve. Anote el valor devuelto para `quoteId`.

```
{
  "properties": {
    "currencyCode": "USD",
    "netTotal": 313219.0,
    "taxTotal": 0.0,
    "isTaxIncluded": false,
    "grandTotal": 313219.0,
    "purchaseRequest": {
      "sku": {
        "name": "SAP_HANA_On_Azure_S224om"
      },
      "location": "eastus",
      "properties": {
        "billingScopeId": "/subscriptions/11111111-1111-1111-111111111111",
        "term": "P1Y",
        "billingPlan": "Upfront",
        "quantity": 1,
        "displayName": "testreservation_S224om",
        "appliedScopes": [
          "/subscriptions/11111111-1111-1111-111111111111"
        ],
        "appliedScopeType": "Single",
        "reservedResourceType": "SapHana",
        "instanceFlexibility": "NotSupported"
      }
    },
    "quoteId": "d0fd3a890795",
    "isBillingPartnerManaged": true,
    "reservationOrderId": "22222222-2222-2222-2222-222222222222",
    "skuTitle": "SAP HANA on Azure Large Instances - S224om - US East",
    "skuDescription": "SAP HANA on Azure Large Instances, S224om",
    "pricingCurrencyTotal": {
      "currencyCode": "USD",
      "amount": 313219.0
    }
  }
}
```

### <a name="make-your-purchase"></a>Realización de la compra

Realice la compra con el `quoteId` devuelto y el `reservationOrderId` que obtuvo de la sección [Obtención del pedido y el precio de reserva](#get-the-reservation-order-and-price) anterior.

Este es un ejemplo de solicitud:

```azurepowershell-interactive
armclient put /providers/Microsoft.Capacity/reservationOrders/22222222-2222-2222-2222-222222222222?api-version=2018-06-01 "{
    'sku': {
        'name': 'SAP_HANA_On_Azure_S224om'
    },
    'location': 'eastus',
    'properties': {
        'reservedResourceType': 'SapHana',
        'billingScopeId': '/subscriptions/11111111-1111-1111-111111111111',
        'term': 'P1Y',
        'quantity': '1',
        'displayName': ' testreservation_S224om',
        'appliedScopes': ['/subscriptions/11111111-1111-1111-111111111111/resourcegroups/123'],
        'appliedScopeType': 'Single',
       'instanceFlexibility': 'NotSupported',
       'renew': true,
       'quoteId': 'd0fd3a890795'
    }
}"
```

Este es un ejemplo de respuesta. Si el pedido se realiza correctamente, `provisioningState` debe ser `creating`.

```
{
  "id": "/providers/microsoft.capacity/reservationOrders/22222222-2222-2222-2222-222222222222",
  "type": "Microsoft.Capacity/reservationOrders",
  "name": "22222222-2222-2222-2222-222222222222",
  "etag": 1,
  "properties": {
    "displayName": "testreservation_S224om",
    "requestDateTime": "2020-07-14T05:42:34.3528353Z",
    "term": "P1Y",
    "provisioningState": "Creating",
    "reservations": [
      {
        "sku": {
          "name": "SAP_HANA_On_Azure_S224om"
        },
        "id": "/providers/microsoft.capacity/reservationOrders22222222-2222-2222-2222-222222222222/reservations/33333333-3333-3333-3333-3333333333333",
        "type": "Microsoft.Capacity/reservationOrders/reservations",
        "name": "22222222-2222-2222-2222-222222222222/33333333-3333-3333-3333-3333333333333",
        "etag": 1,
        "location": "eastus”
        "properties": {
          "appliedScopes": [
            "/subscriptions/11111111-1111-1111-111111111111/resourcegroups/123"
          ],
          "appliedScopeType": "Single",
          "quantity": 1,
          "provisioningState": "Creating",
          "displayName": " testreservation_S224om",
          "effectiveDateTime": "2020-07-14T05:42:34.3528353Z",
          "lastUpdatedDateTime": "2020-07-14T05:42:34.3528353Z",
          "reservedResourceType": "SapHana",
          "instanceFlexibility": "NotSupported",
          "skuDescription": "SAP HANA on Azure Large Instances – S224om - US East",
          "renew": true
        }
      }
    ],
    "originalQuantity": 1,
    "billingPlan": "Upfront"
  }
}
```

### <a name="verify-purchase-status-success"></a>Verificación del estado correcto de la compra

Ejecute la solicitud GET del pedido de reserva para ver el estado del pedido de compra. `provisioningState` debe ser `Succeeded`.

```azurepowershell-interactive
armclient get /providers/microsoft.capacity/reservationOrders/22222222-2222-2222-2222-222222222222?api-version=2018-06-01
```

La respuesta debe tener un aspecto similar al ejemplo siguiente.

```
{
  "id": "/providers/microsoft.capacity/reservationOrders/44444444-4444-4444-4444-444444444444",
  "type": "Microsoft.Capacity/reservationOrders",
  "name": "22222222-2222-2222-2222-222222222222 ",
  "etag": 8,
  "properties": {
    "displayName": "testreservation_S224om",
    "requestDateTime": "2020-07-14T05:42:34.3528353Z",
    "createdDateTime": "2020-07-14T05:44:47.157579Z",
    "expiryDate": "2021-07-14",
    "term": "P1Y",
    "provisioningState": "Succeeded",
    "reservations": [
      {
        "id": "/providers/microsoft.capacity/reservationOrders/22222222-2222-2222-2222-222222222222/reservations/33333333-3333-3333-3333-3333333333333"
      }
    ],
    "originalQuantity": 1,
    "billingPlan": "Upfront"
  }
}
```

## <a name="hli-reservation-fields"></a>Campos de reserva de HLI

En la siguiente información se explica el significado de los distintos campos de reserva.

  **SKU** Nombre de la SKU de HLI. Su aspecto es similar a `SAP_HANA_On_Azure_<SKUname>`.

  **Location** Regiones de HLI disponibles. Consulte [SKU para SAP HANA en Azure (instancias grandes)](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/hana-available-skus) para conocer las regiones disponibles. Para obtener el formato de cadena de ubicación, use la [llamada API de obtención de ubicaciones](https://docs.microsoft.com/rest/api/resources/subscriptions/listlocations#locationlistresult).

  **Reserved Resource type** `SapHana`

  **Subscription** Suscripción que se usa para pagar la reserva. Los costos de la reserva se cobran en el método de pago de la suscripción. El tipo de suscripción debe ser Contrato Enterprise (números de oferta: MS-AZR-0017P o MS-AZR-0148P) o Contrato de cliente de Microsoft. Los cargos se deducen del saldo de compromiso monetario, si está disponible, o se cobran como uso por encima del límite.

  **Scope** El ámbito de la reserva debe ser único.

  **Term** Uno o tres años. Su aspecto es similar a `P1Y` o `P3Y`.

  **Quantity** Número de instancias que se compran para la reserva. La cantidad que se va a comprar es una única HLI a la vez. Para las reservas adicionales, repita la llamada API con los campos correspondientes.

## <a name="troubleshoot-errors"></a>Solución de errores

Podría recibir un error como el del ejemplo siguiente al hacer una compra de reserva. La causa posible es que la HLI no esté aprovisionada para la compra. Si es así, póngase en contacto con su equipo de cuentas de Microsoft para obtener una HLI aprovisionada antes de intentar realizar una compra de reserva.

```
{
  "error": {
    "code": "BadRequest",
    "message": "Capacity check or quota check failed. Please select a different subscription or 
location. You can also go to https://aka.ms/corequotaincrease to learn about quota increase."
  }
} 
```

## <a name="next-steps"></a>Pasos siguientes

- Obtenga información sobre el [procedimiento para llamar a las API de REST de Azure con Postman y cURL](https://docs.microsoft.com/rest/api/azure/#how-to-call-azure-rest-apis-with-postman).
- Consulte [SKU para SAP HANA en Azure (instancias grandes)](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/hana-available-skus) para conocer la lista de SKU y las regiones disponibles.