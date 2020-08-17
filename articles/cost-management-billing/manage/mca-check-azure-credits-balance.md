---
title: Seguimiento del saldo del crédito de Azure para el contrato de cliente de Microsoft
description: Aprenda a comprobar el saldo del crédito de Azure para un contrato de cliente de Microsoft.
author: bandersmsft
ms.reviewer: amberb
tags: billing
ms.service: cost-management-billing
ms.topic: how-to
ms.date: 07/24/2019
ms.author: banders
ms.openlocfilehash: 5d6e0e72335b1f792ca6201d5c810c7660ea06f6
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87281759"
---
# <a name="track-microsoft-customer-agreement-azure-credit-balance"></a>Seguimiento del saldo del crédito de Azure para el contrato de cliente de Microsoft

Puede consultar el saldo del crédito de Azure para la cuenta de facturación de un Contrato de cliente de Microsoft en Azure Portal o mediante las API REST.

En la cuenta de facturación de un contrato de cliente de Microsoft, los créditos se asignan a un perfil de facturación. Cada perfil de facturación tiene sus propios créditos que se aplican automáticamente a los cargos de su factura. Debe tener un rol de propietario, colaborador, lector o administrador de facturación en el perfil de facturación o el rol de propietario, colaborador o lector en la cuenta de facturación para ver el saldo del crédito de Azure para un perfil de facturación. Para más información sobre los roles, consulte [Descripción de los roles administrativos del contrato de cliente de Microsoft en Azure](understand-mca-roles.md).

> [!NOTE]
> El nuevo crédito puede tardar hasta 24 horas en aparecer en Azure Portal. Si obtiene un nuevo crédito y no lo ve en el portal, tenga en cuenta que existe la posibilidad de que tarde 24 horas en aparecer.

Este artículo se aplica a una cuenta de facturación para un contrato de cliente de Microsoft. [Compruebe si tiene acceso a un contrato de cliente de Microsoft](#check-access-to-a-microsoft-customer-agreement).

## <a name="check-your-credit-balance"></a>Comprobación del saldo del crédito

### <a name="azure-portal"></a>[Azure Portal](#tab/portal)

1. Inicie sesión en [Azure Portal](https://portal.azure.com).

2. Busque **Administración de costos + facturación**.

    ![Captura de pantalla que muestra la búsqueda en el portal para la administración de cosos y facturación](./media/mca-check-azure-credits-balance/billing-search-cost-management-billing.png)

3. En la página de ámbitos de facturación, seleccione la cuenta de facturación para la que desea realizar el seguimiento del saldo de crédito. La cuenta de facturación será del tipo **Contrato de cliente de Microsoft**.

    ![Captura de pantalla que muestra la búsqueda en el portal para la administración de cosos y facturación](./media/mca-check-azure-credits-balance/list-of-scopes.png)

    > [!NOTE]
    >
    > Azure Portal recuerda el último ámbito de facturación al que tiene acceso y muestra el ámbito la próxima vez que llega a la página Administración de costos + facturación. No verá la página de ámbitos de facturación si ha visitado Administración de costos y facturación anteriormente. Si es así, compruebe que se encuentra en el [ámbito correcto](#check-access-to-a-microsoft-customer-agreement). En caso contrario, [cambie el ámbito](view-all-accounts.md#switch-billing-scope-in-the-azure-portal) para seleccionar la cuenta de facturación de un Contrato de cliente de Microsoft.

3. Seleccione **Métodos de pago** en el lado izquierdo y, a continuación, seleccione **Créditos de Azure**.

   ![Captura de pantalla del saldo de crédito para un perfil de facturación](./media/mca-check-azure-credits-balance/mca-payment-methods.png)

4. La página Créditos de Azure incluye las siguientes secciones:

   #### <a name="balance"></a>Saldo

   La sección de saldo muestra el resumen del saldo de crédito de Azure.

   ![Captura de pantalla del saldo de crédito para un perfil de facturación](./media/mca-check-azure-credits-balance/mca-credit-balance.png)

   | Término               | Definición                           |
   |--------------------|--------------------------------------------------------|
   | Saldo estimado  | Es la cantidad estimada de crédito del que dispone teniendo en cuenta todas las transacciones pendientes y no liquidadas. |
   | Saldo actual    | Es la cantidad de créditos desde la última factura. No incluye las transacciones pendientes. |

   Cuando su saldo estimado desciende a 0, se le cobrará por todo su uso, incluidos los productos que son aptos para los créditos.

   #### <a name="credits-list"></a>Lista de créditos

   La sección Lista de créditos muestra la lista de créditos de Azure.

   ![Captura de pantalla de las listas de créditos para un perfil de facturación](./media/mca-check-azure-credits-balance/mca-credits-list.png)

   | Término | Definición |
   |---|---|
   | Source | El origen de la adquisición del crédito |
   | Fecha de inicio | La fecha en la que se adquirió el crédito |
   | Fecha de expiración | Cuando expira el crédito |
   | Saldo actual | El saldo desde la última factura |
   | Importe original | El importe original del crédito |
   | Estado | El estado actual del crédito. El estado puede ser activo, usado, expirado o en proceso de expiración |

   #### <a name="transactions"></a>Transacciones

   En la sección de transacciones se muestran todas las transacciones que afectan al saldo de créditos.

   ![Captura de pantalla de transacciones de crédito para un perfil de facturación](./media/mca-check-azure-credits-balance/mca-credits-transactions.png)

   | Término | Definición |
   |---|---|
   | Fecha de transacción | Fecha en la que ha ocurrido la transacción |
   | Descripción | Descripción de la transacción |
   | Importe| El importe de la transacción |
   | Saldo | Saldo después de la transacción |

    > [!NOTE]
    >
    > Si no ve los créditos de Azure en la página Métodos de pago, puede que no tenga créditos o que no haya seleccionado el ámbito adecuado. Seleccione la cuenta de facturación que tenga créditos o uno de sus perfiles de facturación. Para obtener información sobre cómo cambiar los ámbitos, consulte [Cambio de ámbitos de facturación en Azure Portal](view-all-accounts.md#switch-billing-scope-in-the-azure-portal).

5. Si está viendo créditos de Azure en el ámbito de la cuenta de facturación y esta cuenta tiene más de un perfil de facturación, la página Créditos de Azure mostrará una tabla con un resumen de los créditos de Azure para cada perfil de facturación. Seleccione un perfil de facturación de la lista, seleccione los métodos de pago y, a continuación, los créditos de Azure para ver los detalles de un perfil de facturación.

    ![Captura de pantalla de la lista de crédito para una cuenta de facturación](./media/mca-check-azure-credits-balance/mca-account-credit-list.png)

### <a name="rest-api"></a>[REST API](#tab/rest)

Puede usar las API de [facturación de Azure](https://docs.microsoft.com/rest/api/billing/) y de [consumo](https://docs.microsoft.com/rest/api/consumption/) para obtener mediante programación el saldo del crédito de la cuenta de facturación.

Los ejemplos que se muestran a continuación usan las API de REST. Actualmente, no están admitidos PowerShell ni la CLI de Azure.

### <a name="find-billing-profiles-you-have-access-to"></a>Búsqueda de perfiles de facturación a los que se tiene acceso

```json
GET https://management.azure.com/providers/Microsoft.Billing/billingAccounts?$expand=billingProfiles&api-version=2019-10-01-preview
```
La respuesta de la API devuelve una lista de cuentas de facturación y sus perfiles de facturación.

```json
{
  "value": [
    {
      "id": "/providers/Microsoft.Billing/billingAccounts/5e98e158-xxxx-xxxx-xxxx-xxxxxxxxxxxx:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxx_xxxx-xx-xx",
      "name": "5e98e158-xxxx-xxxx-xxxx-xxxxxxxxxxxx:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxx_xxxx-xx-xx",
      "properties": {
        "accountId": "5e98e158-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
        "accountStatus": "Active",
        "accountType": "Enterprise",
        "agreementType": "MicrosoftCustomerAgreement",
        "billingProfiles": [
          {
            "id": "/providers/Microsoft.Billing/billingAccounts/5e98e158-xxxx-xxxx-xxxx-xxxxxxxxxxxx:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxx_xxxx-xx-xx/billingProfiles/PBFV-xxxx-xxx-xxx",
            "name": "PBFV-xxxx-xxx-xxx",
            "properties": {
              "address": {
                "addressLine1": "AddressLine1",
                "city": "City",
                "companyName": "CompanyName",
                "country": "Country",
                "postalCode": "xxxxx",
                "region": "Region"
              },
              "currency": "USD",
              "displayName": "Development",
              "hasReadAccess": true,
              "invoiceDay": 5,
              "invoiceEmailOptIn": true
            },
            "type": "Microsoft.Billing/billingAccounts/billingProfiles"
          }
        ],
        "displayName": "Contoso",
        "hasReadAccess": true,
      },
      "type": "Microsoft.Billing/billingAccounts"
    }
  ]
}
```

Use la propiedad `displayName` del perfil de facturación para identificar el perfil de facturación para el que desea comprobar el saldo del crédito. Copie `id` del perfil de facturación. Por ejemplo, si desea comprobar el saldo del crédito del perfil de facturación **Development**, debe copiar ```/providers/Microsoft.Billing/billingAccounts/5e98e158-xxxx-xxxx-xxxx-xxxxxxxxxxxx:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxx_xxxx-xx-xx/billingProfiles/PBFV-xxxx-xxx-xxx```. Pegue este valor en algún lugar para poder usarlo en el paso siguiente.

### <a name="get-azure-credit-balance"></a>Obtención del saldo del crédito de Azure

Realice la siguiente solicitud; para ello, reemplace `<billingProfileId>` por el elemento `id` copiado en el primer paso (```/providers/Microsoft.Billing/billingAccounts/5e98e158-xxxx-xxxx-xxxx-xxxxxxxxxxxx:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxx_xxxx-xx-xx/billingProfiles/PBFV-xxxx-xxx-xxx```).

```json
GET https://management.azure.com<billingProfileId>/providers/Microsoft.Consumption/credits/balanceSummary?api-version=2019-10-01
```

La respuesta de la API devuelve el saldo estimado y actual para el perfil de facturación.

```json
{
  "id": "/providers/Microsoft.Billing/billingAccounts/5e98e158-xxxx-xxxx-xxxx-xxxxxxxxxxxx:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxx_xxxx-xx-xx/billingProfiles/PBFV-xxxx-xxx-xxx/providers/Microsoft.Consumption/credits/balanceSummary/57c2e8df-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "name": "57c2e8df-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "type": "Microsoft.Consumption/credits/balanceSummary",
  "eTag": null,
  "properties": {
    "balanceSummary": {
      "estimatedBalance": {
        "currency": "USD",
        "value": 996.13
      },
      "currentBalance": {
        "currency": "USD",
        "value": 997.87
      }
    },
    "pendingCreditAdjustments": {
      "currency": "USD",
      "value": 0.0
    },
    "expiredCredit": {
      "currency": "USD",
      "value": 0.0
    },
    "pendingEligibleCharges": {
      "currency": "USD",
      "value": -1.74
    }
  }
}
```

| Nombre del elemento  | Descripción                                                                           |
|---------------|---------------------------------------------------------------------------------------|
| `estimatedBalance` | La cantidad estimada de créditos de la que dispone teniendo en cuenta todas las transacciones facturadas y pendientes. |
| `currentBalance`   | La cantidad de créditos desde la última factura. No incluye las transacciones pendientes.    |
| `pendingCreditAdjustments`      | Los ajustes, como reembolsos, que aún no se han facturado.  |
| `expiredCredit`      |  El crédito que expiró desde la última factura.  |
| `pendingEligibleCharges`  | Los cargos aptos para el crédito que aún no se han facturado.   |

### <a name="get-list-of-credits"></a>Obtención de listas de créditos

Realice la siguiente solicitud; para ello, reemplace `<billingProfileId>` por el elemento `id` copiado en el primer paso (```/providers/Microsoft.Billing/billingAccounts/5e98e158-xxxx-xxxx-xxxx-xxxxxxxxxxxx:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxx_xxxx-xx-xx/billingProfiles/PBFV-xxxx-xxx-xxx```).

```json
GET https://management.azure.com<billingProfileId>/providers/Microsoft.Consumption/lots?api-version=2019-10-01
```
La respuesta de la API devuelve listas de créditos de Azure para un perfil de facturación.

```json
{
  "value": [
    {
      "id": "/providers/Microsoft.Billing/billingAccounts/5e98e158-xxxx-xxxx-xxxx-xxxxxxxxxxxx:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxx_xxxx-xx-xx/billingProfiles/PBFV-xxxx-xxx-xxx/providers/Microsoft.Consumption/lots/f2ecfd94-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
      "name": "f2ecfd94-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
      "type": "Microsoft.Consumption/lots",
      "eTag": null,
      "properties": {
        "originalAmount": {
          "currency": "USD",
          "value": 500.0
        },
        "closedBalance": {
          "currency": "USD",
          "value": 500.0
        },
        "source": "Azure Promotional Credit",
        "startDate": "09/18/2019 21:47:31",
        "expirationDate": "09/18/2020 21:47:30",
        "poNumber": ""
      }
    },
    {
      "id": "/providers/Microsoft.Billing/billingAccounts/5e98e158-xxxx-xxxx-xxxx-xxxxxxxxxxxx:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxx_xxxx-xx-xx/billingProfiles/xxxx-xxxx-xxx-xxx/providers/Microsoft.Consumption/lots/4ea40eb5-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
      "name": "4ea40eb5-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
      "type": "Microsoft.Consumption/lots",
      "eTag": null,
      "properties": {
        "originalAmount": {
          "currency": "USD",
          "value": 500.0
        },
        "closedBalance": {
          "currency": "USD",
          "value": 497.87
        },
        "source": "Azure Promotional Credit",
        "startDate": "09/18/2019 21:47:31",
        "expirationDate": "09/18/2020 21:47:30",
        "poNumber": ""
      }
    }
  ]
}
```
| Nombre del elemento  | Descripción                                                                                               |
|---------------|-----------------------------------------------------------------------------------------------------------|
| `originalAmount` | El importe original del crédito. |
| `closedBalance`   | El saldo desde la última factura.    |
| `source`      | El origen que define quién ha adquirido el crédito. |
| `startDate`      |  La fecha en la que se ha activado el crédito.  |
| `expirationDate`  | La fecha en la que expira el crédito.   |
| `poNumber`  | El número de pedido de compra de la factura en la que se ha facturado el crédito.   |

### <a name="get-transactions-that-affected-credit-balance"></a>Obtención de transacciones que afectan al saldo del crédito

Realice la siguiente solicitud; para ello, reemplace `<billingProfileId>` por el elemento `id` copiado en el primer paso (```providers/Microsoft.Billing/billingAccounts/5e98e158-xxxx-xxxx-xxxx-xxxxxxxxxxxx:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxx_xxxx-xx-xx/billingProfiles/PBFV-xxxx-xxx-xxx```). Deberá pasar **startDate** y **endDate** a fin de obtener las transacciones para la duración que necesita.

```json
GET https://management.azure.com<billingProfileId>/providers/Microsoft.Consumption/events?api-version=2019-10-01&startDate=2018-10-01T00:00:00.000Z&endDate=2019-10-11T12:00:00.000Z?api-version=2019-10-01
```
La respuesta de la API devuelve todas las transacciones que afectan al saldo del crédito de su perfil de facturación.

```json
{
  "value": [
    {
      "id": "/providers/Microsoft.Billing/billingAccounts/5e98e158-xxxx-xxxx-xxxx-xxxxxxxxxxxx:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxx_xxxx-xx-xx/billingProfiles/PBFV-xxxx-xxx-xxx`/providers/Microsoft.Consumption/events/e2032eb5-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
      "name": "e2032eb5-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
      "type": "Microsoft.Consumption/events",
      "eTag": null,
      "properties": {
        "transactionDate": "10/11/2019",
        "description": "Credit eligible charges as of 10/11/2019",
        "newCredit": {
          "currency": "USD",
          "value": 0.0
        },
        "adjustments": {
          "currency": "USD",
          "value": 0.0
        },
        "creditExpired": {
          "currency": "USD",
          "value": 0.0
        },
        "charges": {
          "currency": "USD",
          "value": -1.74
        },
        "closedBalance": {
          "currency": "USD",
          "value": 998.26
        },
        "eventType": "PendingCharges",
        "invoiceNumber": ""
      }
    },
    {
      "id": "/providers/Microsoft.Billing/billingAccounts/5e98e158-xxxx-xxxx-xxxx-xxxxxxxxxxxx:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxx_xxxx-xx-xx/billingProfiles/PBFV-xxxx-xxx-xxx/providers/Microsoft.Consumption/events/381efd80-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
      "name": "381efd80-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
      "type": "Microsoft.Consumption/events",
      "eTag": null,
      "properties": {
        "transactionDate": "09/18/2019",
        "description": "New credit added on 09/18/2019",
        "newCredit": {
          "currency": "USD",
          "value": 500.0
        },
        "adjustments": {
          "currency": "USD",
          "value": 0.0
        },
        "creditExpired": {
          "currency": "USD",
          "value": 0.0
        },
        "charges": {
          "currency": "USD",
          "value": 0.0
        },
        "closedBalance": {
          "currency": "USD",
          "value": 1000.0
        },
        "eventType": "PendingNewCredit",
        "invoiceNumber": ""
      }
    }
  ]
}
```
| Nombre del elemento  | Descripción                                                                                               |
|---------------|-----------------------------------------------------------------------------------------------------------|
| `transactionDate` | La fecha en la que ha tenido lugar la transacción. |
| `description` | La descripción de la transacción. |
| `adjustments`   | Los ajustes de crédito de la transacción.    |
| `creditExpired`      | El importe del crédito que ha expirado. |
| `charges`      |  Los cargos de la transacción.  |
| `closedBalance`  | El saldo después de la transacción.   |
| `eventType`  | El tipo de transacción.   |
| `invoiceNumber`  | El número de la factura en que se factura la transacción. Estará vacío para las transacciones pendientes.   |

---

## <a name="how-credits-are-used"></a>Uso de los créditos

En una cuenta de facturación para un contrato de cliente de Microsoft, utilizará perfiles de facturación que le permiten administrar las facturas y los métodos de pago. Se crea una factura mensual para cada perfil de facturación y utilizará los métodos de pago para pagar la factura.

Asigna créditos que adquiere para un perfil de facturación. Cuando se genera una factura para el perfil de facturación, los créditos se aplican automáticamente a los cargos totales para calcular la cantidad que tiene que pagar. Paga el importe restante con sus métodos de pago, como cheque, transferencia bancaria o tarjeta de crédito.

## <a name="products-that-arent-covered-by-azure-credits"></a>Productos que no están cubiertos por los créditos de Azure

 Los siguientes productos no están cubiertos por los créditos de Azure. Se le cobrará por usar estos productos, con independencia del saldo del crédito:

- Canonical
- Citrix XenApp Essentials
- Citrix XenDesktop
- Usuario registrado
- Openlogic
- Usuario registrado de Remote Access Rights XenApp Essentials
- Ubuntu Advantage
- Visual Studio Enterprise (mensual)
- Visual Studio Enterprise (anual)
- Visual Studio Professional (mensual)
- Visual Studio Professional (anual)
- Productos de Azure Marketplace
- Planes de soporte técnico de Azure

## <a name="check-access-to-a-microsoft-customer-agreement"></a>Comprobación del acceso a un contrato de cliente de Microsoft
[!INCLUDE [billing-check-mca](../../../includes/billing-check-mca.md)]

## <a name="need-help-contact-support"></a>¿Necesita ayuda? Póngase en contacto con el servicio de soporte técnico.

Si necesita ayuda, [póngase en contacto con soporte técnico](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade) para resolver el problema rápidamente.

## <a name="next-steps"></a>Pasos siguientes

- [Descripción de la cuenta de facturación para el contrato de cliente de Microsoft](../understand/mca-overview.md)
- [Descripción de los términos en la factura del contrato de cliente de Microsoft](../understand/mca-understand-your-invoice.md)
