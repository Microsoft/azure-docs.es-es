---
title: Creación de suscripciones de Azure mediante programación
description: Aprenda a crear suscripciones de Azure adicionales mediante programación.
author: anuragdalmia
ms.topic: conceptual
ms.date: 07/09/2020
ms.reviewer: andalmia
ms.author: banders
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 1b4c16dd276f9f564963fdefe8d16dbc92c1303d
ms.sourcegitcommit: 85eb6e79599a78573db2082fe6f3beee497ad316
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/05/2020
ms.locfileid: "87810452"
---
# <a name="programmatically-create-azure-subscriptions-preview"></a>Creación de suscripciones de Azure mediante programación (versión preliminar)

Los clientes de Azure que tienen una cuenta de facturación de un [Contrato Enterprise (EA)](https://azure.microsoft.com/pricing/enterprise-agreement/), un [Contrato de cliente de Microsoft (MCA)](https://azure.microsoft.com/pricing/purchase-options/microsoft-customer-agreement/) o un contrato [Microsoft Partner Agreement (MPA)](https://www.microsoft.com/licensing/news/introducing-microsoft-partner-agreement) pueden crear suscripciones mediante programación. En este artículo, se ofrece información sobre cómo crear suscripciones mediante programación con Azure Resource Manager.

Al crear una suscripción a Azure mediante programación, dicha suscripción se rige por el contrato en cuyo marco ha obtenido los servicios de Azure de Microsoft o de un distribuidor autorizado. Para obtener más información, vea [Información legal de Microsoft Azure](https://azure.microsoft.com/support/legal/).

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]


## <a name="create-subscriptions-for-an-ea-billing-account"></a>Creación de suscripciones para una cuenta de facturación de EA

Use la información de las secciones siguientes para aprender a crea suscripciones a EA.

### <a name="prerequisites"></a>Requisitos previos

Debe tener un rol Propietario en una cuenta de inscripción para crear una suscripción. Existen dos formas de obtener el rol:

* El administrador de empresa de la inscripción puede [convertirle en propietario de una cuenta](https://ea.azure.com/helpdocs/addNewAccount) (inicio de sesión requerido), lo que le hace propietario de la cuenta de inscripción.

* Un propietario existente de la cuenta de inscripción puede [concederle acceso](grant-access-to-create-subscription.md). De forma similar, si desea usar una entidad de servicio para crear una suscripción a Contrato Enterprise, debe [conceder a dicha entidad de servicio la capacidad de crear suscripciones](grant-access-to-create-subscription.md).

### <a name="find-accounts-you-have-access-to"></a>Búsqueda de cuentas a las que tiene acceso

Cuando haya sido agregado a una cuenta de inscripción asociada a un propietario de cuenta, Azure usa la relación cuenta-inscripción para determinar dónde se cobra la suscripción. Todas las suscripciones creadas en la cuenta se facturan a la inscripción de EA en la que se encuentra la cuenta. Para crear suscripciones, debe pasar valores sobre la cuenta de inscripción y las entidades de seguridad de usuario al propietario de la suscripción.

Para ejecutar los comandos siguientes, debe iniciar sesión en el *directorio particular* del propietario de cuenta, que es el directorio en el que las suscripciones se crean de manera predeterminada.

### <a name="rest"></a>[REST](#tab/rest)

Solicite mostrar todas las cuentas de inscripción a las que tiene acceso:

```json
GET https://management.azure.com/providers/Microsoft.Billing/enrollmentAccounts?api-version=2018-03-01-preview
```

La respuesta de la API muestra todas las cuentas de inscripción a las que tiene acceso:

```json
{
  "value": [
    {
      "id": "/providers/Microsoft.Billing/enrollmentAccounts/747ddfe5-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
      "name": "747ddfe5-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
      "type": "Microsoft.Billing/enrollmentAccounts",
      "properties": {
        "principalName": "SignUpEngineering@contoso.com"
      }
    },
    {
      "id": "/providers/Microsoft.Billing/enrollmentAccounts/4cd2fcf6-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
      "name": "4cd2fcf6-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
      "type": "Microsoft.Billing/enrollmentAccounts",
      "properties": {
        "principalName": "BillingPlatformTeam@contoso.com"
      }
    }
  ]
}
```

Use la propiedad `principalName` para identificar la cuenta a la que quiere que se facturen las suscripciones. Copie el elemento `name` de esa cuenta. Por ejemplo, si quisiera crear suscripciones en la cuenta de inscripción SignUpEngineering@contoso.com, copiaría ```747ddfe5-xxxx-xxxx-xxxx-xxxxxxxxxxxx```. Este es el identificador de objeto de la cuenta de inscripción. Pegue este valor en algún lugar para poder usarlo en el paso siguiente como `enrollmentAccountObjectId`.

### <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Abra [Azure Cloud Shell](https://shell.azure.com/) y seleccione PowerShell.

Use el cmdlet [Get-AzEnrollmentAccount](/powershell/module/az.billing/get-azenrollmentaccount) para mostrar todas las cuentas de inscripción a las que tiene acceso.

```azurepowershell-interactive
Get-AzEnrollmentAccount
```

Azure responde con una lista de las cuentas de inscripción a las que tiene acceso:

```azurepowershell
ObjectId                               | PrincipalName
747ddfe5-xxxx-xxxx-xxxx-xxxxxxxxxxxx   | SignUpEngineering@contoso.com
4cd2fcf6-xxxx-xxxx-xxxx-xxxxxxxxxxxx   | BillingPlatformTeam@contoso.com
```
Use la propiedad `principalName` para identificar la cuenta a la que quiere que se facturen las suscripciones. Copie el elemento `ObjectId` de esa cuenta. Por ejemplo, si quisiera crear suscripciones en la cuenta de inscripción SignUpEngineering@contoso.com, copiaría ```747ddfe5-xxxx-xxxx-xxxx-xxxxxxxxxxxx```. Pegue este identificador de objeto en alguna parte para poder usarlo en el paso siguiente como `enrollmentAccountObjectId`.

### <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

Use el comando [az billing enrollment-account list](https://aka.ms/EASubCreationPublicPreviewCLI) para mostrar todas las cuentas de inscripción a las que tiene acceso.

```azurecli-interactive
az billing enrollment-account list
```

Azure responde con una lista de las cuentas de inscripción a las que tiene acceso:

```json
[
  {
    "id": "/providers/Microsoft.Billing/enrollmentAccounts/747ddfe5-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
    "name": "747ddfe5-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
    "principalName": "SignUpEngineering@contoso.com",
    "type": "Microsoft.Billing/enrollmentAccounts",
  },
  {
    "id": "/providers/Microsoft.Billing/enrollmentAccounts/747ddfe5-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
    "name": "4cd2fcf6-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
    "principalName": "BillingPlatformTeam@contoso.com",
    "type": "Microsoft.Billing/enrollmentAccounts",
  }
]
```

Use la propiedad `principalName` para identificar la cuenta a la que quiere que se facturen las suscripciones. Copie el elemento `name` de esa cuenta. Por ejemplo, si quisiera crear suscripciones en la cuenta de inscripción SignUpEngineering@contoso.com, copiaría ```747ddfe5-xxxx-xxxx-xxxx-xxxxxxxxxxxx```. Este es el identificador de objeto de la cuenta de inscripción. Pegue este valor en algún lugar para poder usarlo en el paso siguiente como `enrollmentAccountObjectId`.

---

### <a name="create-subscriptions-under-a-specific-enrollment-account"></a>Creación de suscripciones con una cuenta de inscripción concreta

En el ejemplo siguiente se crea una suscripción denominada *Dev Team Subscription* en la cuenta de inscripción seleccionada en el paso anterior. La oferta de suscripción es *MS-AZR-0017P* (Contrato Enterprise de Microsoft normal). Opcionalmente, también agrega dos usuarios como propietarios de RBAC a la suscripción.

### <a name="rest"></a>[REST](#tab/rest)

Realice la siguiente solicitud; para ello, reemplace `<enrollmentAccountObjectId>` por el elemento `name` copiado en el primer paso (```747ddfe5-xxxx-xxxx-xxxx-xxxxxxxxxxxx```). Si quiere especificar propietarios, obtenga información sobre [cómo obtener identificadores de objeto de usuario](grant-access-to-create-subscription.md#userObjectId).

```json
POST https://management.azure.com/providers/Microsoft.Billing/enrollmentAccounts/<enrollmentAccountObjectId>/providers/Microsoft.Subscription/createSubscription?api-version=2018-03-01-preview

{
  "displayName": "Dev Team Subscription",
  "offerType": "MS-AZR-0017P",
  "owners": [
    {
      "objectId": "<userObjectId>"
    },
    {
      "objectId": "<servicePrincipalObjectId>"
    }
  ]
}
```

| Nombre del elemento  | Obligatorio | Tipo   | Descripción                                                                                               |
|---------------|----------|--------|-----------------------------------------------------------------------------------------------------------|
| `displayName` | No      | String | Nombre para mostrar de la suscripción. Si no se especifica, se establece en el nombre de la oferta, por ejemplo, "Microsoft Azure Enterprise".                                 |
| `offerType`   | Sí      | String | Oferta de la suscripción. Las dos opciones para EA son [MS-AZR-0017P](https://azure.microsoft.com/pricing/enterprise-agreement/) (uso en producción) y [MS-AZR-0148P](https://azure.microsoft.com/offers/ms-azr-0148p/) (desarrollo y pruebas, tiene que [activarse mediante el portal de EA](https://ea.azure.com/helpdocs/DevOrTestOffer)).                |
| `owners`      | No       | String | Identificador de objeto de cualquier usuario que quiera agregarse como propietario de RBAC en la suscripción al crearla.  |

En la respuesta, como parte del encabezado `Location`, obtiene una dirección URL en la que puede consultar el estado de la operación de creación de la suscripción. Cuando haya finalizado la creación de la suscripción, una solicitud GET en la URL `Location` devolvería un objeto `subscriptionLink`, que tiene el identificador de suscripción. Para obtener más información, consulte la [documentación de la API de suscripción](/rest/api/subscription/).

### <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Para instalar la versión más reciente del módulo que contiene el cmdlet `New-AzSubscription`, ejecute `Install-Module Az.Subscription`. Para instalar una versión reciente de PowerShellGet, consulte [Obtención del módulo PowerShellGet](/powershell/scripting/gallery/installing-psget).

Ejecute el comando [New-AzSubscription](/powershell/module/az.subscription) siguiente y reemplace `<enrollmentAccountObjectId>` por el elemento `ObjectId` recopilado en el primer paso (```747ddfe5-xxxx-xxxx-xxxx-xxxxxxxxxxxx```). Si quiere especificar propietarios, obtenga información sobre [cómo obtener identificadores de objeto de usuario](grant-access-to-create-subscription.md#userObjectId).

```azurepowershell-interactive
New-AzSubscription -OfferType MS-AZR-0017P -Name "Dev Team Subscription" -EnrollmentAccountObjectId <enrollmentAccountObjectId> -OwnerObjectId <userObjectId1>,<servicePrincipalObjectId>
```

| Nombre del elemento  | Obligatorio | Tipo   | Descripción                                                                                               |
|---------------|----------|--------|-----------------------------------------------------------------------------------------------------------|
| `Name` | No      | String | Nombre para mostrar de la suscripción. Si no se especifica, se establece en el nombre de la oferta, por ejemplo, "Microsoft Azure Enterprise".                                 |
| `OfferType`   | Sí      | String | Oferta de la suscripción. Las dos opciones para EA son [MS-AZR-0017P](https://azure.microsoft.com/pricing/enterprise-agreement/) (uso en producción) y [MS-AZR-0148P](https://azure.microsoft.com/offers/ms-azr-0148p/) (desarrollo y pruebas, tiene que [activarse mediante el portal de EA](https://ea.azure.com/helpdocs/DevOrTestOffer)).                |
| `EnrollmentAccountObjectId`      | Sí       | String | El identificador de objeto de la cuenta de inscripción con la que se crea la suscripción y contra la que se realizan los cargos. Este valor es un GUID que se obtiene de `Get-AzEnrollmentAccount`. |
| `OwnerObjectId`      | No       | String | Identificador de objeto de cualquier usuario que quiera agregarse como propietario de RBAC en la suscripción al crearla.  |
| `OwnerSignInName`    | No       | String | Dirección de correo electrónico de cualquier usuario que quiera agregarse como propietario de RBAC en la suscripción al crearla. Puede utilizar este parámetro en lugar de `OwnerObjectId`.|
| `OwnerApplicationId` | No       | String | Identificador de aplicación de cualquier entidad de servicio que quiera agregarse como propietario de RBAC en la suscripción al crearla. Puede utilizar este parámetro en lugar de `OwnerObjectId`. Al usar este parámetro, la entidad de servicio debe tener [acceso de lectura al directorio](/powershell/azure/active-directory/signing-in-service-principal?view=azureadps-2.0#give-the-service-principal-reader-access-to-the-current-tenant-get-azureaddirectoryrole).|

Para obtener una lista completa de todos los parámetros, consulte [New-AzSubscription](/powershell/module/az.subscription/New-AzSubscription).


### <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

En primer lugar, instale esta extensión de versión preliminar al ejecutar `az extension add --name subscription`.

Ejecute el comando [az account create](/cli/azure/ext/subscription/account?view=azure-cli-latest#-ext-subscription-az-account-create) siguiente y reemplace `<enrollmentAccountObjectId>` por el elemento `name` copiado en el primer paso (```747ddfe5-xxxx-xxxx-xxxx-xxxxxxxxxxxx```). Si quiere especificar propietarios, obtenga información sobre [cómo obtener identificadores de objeto de usuario](grant-access-to-create-subscription.md#userObjectId).

```azurecli-interactive
az account create --offer-type "MS-AZR-0017P" --display-name "Dev Team Subscription" --enrollment-account-object-id "<enrollmentAccountObjectId>" --owner-object-id "<userObjectId>","<servicePrincipalObjectId>"
```

| Nombre del elemento  | Obligatorio | Tipo   | Descripción                                                                                               |
|---------------|----------|--------|-----------------------------------------------------------------------------------------------------------|
| `display-name` | No      | String | Nombre para mostrar de la suscripción. Si no se especifica, se establece en el nombre de la oferta, por ejemplo, "Microsoft Azure Enterprise".                                 |
| `offer-type`   | Sí      | String | Oferta de la suscripción. Las dos opciones para EA son [MS-AZR-0017P](https://azure.microsoft.com/pricing/enterprise-agreement/) (uso en producción) y [MS-AZR-0148P](https://azure.microsoft.com/offers/ms-azr-0148p/) (desarrollo y pruebas, tiene que [activarse mediante el portal de EA](https://ea.azure.com/helpdocs/DevOrTestOffer)).                |
| `enrollment-account-object-id`      | Sí       | String | El identificador de objeto de la cuenta de inscripción con la que se crea la suscripción y contra la que se realizan los cargos. Este valor es un GUID que se obtiene de `az billing enrollment-account list`. |
| `owner-object-id`      | No       | String | Identificador de objeto de cualquier usuario que quiera agregarse como propietario de RBAC en la suscripción al crearla.  |
| `owner-upn`    | No       | String | Dirección de correo electrónico de cualquier usuario que quiera agregarse como propietario de RBAC en la suscripción al crearla. Puede utilizar este parámetro en lugar de `owner-object-id`.|
| `owner-spn` | No       | String | Identificador de aplicación de cualquier entidad de servicio que quiera agregarse como propietario de RBAC en la suscripción al crearla. Puede utilizar este parámetro en lugar de `owner-object-id`. Al usar este parámetro, la entidad de servicio debe tener [acceso de lectura al directorio](/powershell/azure/active-directory/signing-in-service-principal?view=azureadps-2.0#give-the-service-principal-reader-access-to-the-current-tenant-get-azureaddirectoryrole).|

Para obtener una lista completa de todos los parámetros, vea [az account create](/cli/azure/ext/subscription/account?view=azure-cli-latest#-ext-subscription-az-account-create).

---

### <a name="limitations-of-azure-enterprise-subscription-creation-api"></a>Limitaciones de la API de creación de suscripciones de Azure Enterprise

- Con esta API solo pueden crearse suscripciones de Azure Enterprise.
- Hay un límite de 2000 suscripciones por cuenta de inscripción. Después, solo se pueden crear más suscripciones para la cuenta en Azure Portal. Si quiere crear más suscripciones a través de la API, cree otra cuenta de inscripción.
- Los usuarios que no son propietarios de cuenta, pero se han agregado a una cuenta de inscripción a través de RBAC, no pueden crear suscripciones en Azure Portal.
- No puede seleccionar el inquilino para la suscripción en que se va a crear. La suscripción siempre se crea en el inquilino principal del propietario de cuenta. Para mover la suscripción a otro inquilino, vea cómo [cambiar un inquilino de la suscripción](../../active-directory/fundamentals/active-directory-how-subscriptions-associated-directory.md).


## <a name="create-subscriptions-for-an-mca-account"></a>Creación de suscripciones para una cuenta de MCA

### <a name="prerequisites"></a>Requisitos previos

Debe tener un rol de propietario, colaborador o creador de la suscripción de Azure en una sección de factura, o un rol de propietario o colaborador en un perfil de facturación o una cuenta de facturación para crear suscripciones. Para más información, consulte [Tareas y roles de la facturación de suscripción](../../cost-management-billing/manage/understand-mca-roles.md#subscription-billing-roles-and-tasks).

El ejemplo que se muestra a continuación usa API REST. Actualmente, no están admitidos PowerShell ni la CLI de Azure.

### <a name="find-billing-accounts-that-you-have-access-to"></a>Búsqueda de cuentas de facturación a las que tiene acceso

Realice la solicitud siguiente para mostrar todas las cuentas de facturación.

```json
GET https://management.azure.com/providers/Microsoft.Billing/billingAccounts?api-version=2019-10-01-preview
```
La respuesta de la API muestra las cuentas de facturación a las que tiene acceso.

```json
{
  "value": [
    {
      "id": "/providers/Microsoft.Billing/billingAccounts/5e98e158-xxxx-xxxx-xxxx-xxxxxxxxxxxx:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx_xxxx-xx-xx",
      "name": "5e98e158-xxxx-xxxx-xxxx-xxxxxxxxxxxx:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx_xxxx-xx-xx",
      "properties": {
        "accountId": "5e98e158-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
        "accountStatus": "Active",
        "accountType": "Enterprise",
        "agreementType": "MicrosoftCustomerAgreement",
        "displayName": "Contoso",
        "hasReadAccess": true,
        "organizationId": "41b29574-xxxx-xxxx-xxxx-xxxxxxxxxxxxx_xxxx-xx-xx"
      },
      "type": "Microsoft.Billing/billingAccounts"
    },
    {
      "id": "/providers/Microsoft.Billing/billingAccounts/4f89e155-xxxx-xxxx-xxxx-xxxxxxxxxxxx:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx_xxxx-xx-xx",
      "name": "4f89e155-xxxx-xxxx-xxxx-xxxxxxxxxxxx:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx_xxxx-xx-xx",
      "properties": {
        "accountId": "4f89e155-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
        "accountStatus": "Active",
        "accountType": "Enterprise",
        "agreementType": "MicrosoftCustomerAgreement",
        "displayName": "Fabrikam",
        "hasReadAccess": true,
        "organizationId": "41b29574-xxxx-xxxx-xxxx-xxxxxxxxxxxxx_xxxx-xx-xx"
      },
      "type": "Microsoft.Billing/billingAccounts"
    }
  ]
}

```
Use la propiedad `displayName` para identificar la cuenta de facturación para la que desea crear suscripciones. Asegúrese de que el valor agreeementType de la cuenta sea *MicrosoftCustomerAgreement*. Copie el valor de `name` de la cuenta.  Por ejemplo, si desea crear una suscripción para la cuenta de facturación `Contoso`, debe copiar `5e98e158-xxxx-xxxx-xxxx-xxxxxxxxxxxx:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx_xxxx-xx-xx`. Pegue este valor en algún lugar para poder usarlo en el paso siguiente.

### <a name="find-invoice-sections-to-create-subscriptions"></a>Búsqueda de secciones de factura para crear suscripciones

Los cargos de la suscripción aparecerán en una sección de una factura del perfil de facturación. Use la siguiente API para obtener la lista de secciones de factura y perfiles de facturación en los que tiene permiso para crear suscripciones de Azure.

Realice la siguiente solicitud; para ello, reemplace `<billingAccountName>` por el elemento `name` copiado en el primer paso (```5e98e158-xxxx-xxxx-xxxx-xxxxxxxxxxxx:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx_xxxx-xx-xx```).

```json
POST https://management.azure.com/providers/Microsoft.Billing/billingAccounts/<billingAccountName>/listInvoiceSectionsWithCreateSubscriptionPermission?api-version=2019-10-01-preview
```
La respuesta de la API muestra todas las secciones de factura y sus perfiles de facturación en los que tiene acceso para crear suscripciones:

```json
{
    "value": [{
        "billingProfileDisplayName": "Contoso finance",
        "billingProfileId": "/providers/Microsoft.Billing/billingAccounts/5e98e158-xxxx-xxxx-xxxx-xxxxxxxxxxxx:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx_xxxx-xx-xx/billingProfiles/PBFV-xxxx-xxx-xxx",
        "enabledAzurePlans": [{
            "productId": "DZH318Z0BPS6",
            "skuId": "0001",
            "skuDescription": "Microsoft Azure Plan"
        }, {
            "productId": "DZH318Z0BPS6",
            "skuId": "0002",
            "skuDescription": "Microsoft Azure Plan for DevTest"
        }],
        "invoiceSectionDisplayName": "Development",
        "invoiceSectionId": "/providers/Microsoft.Billing/billingAccounts/5e98e158-xxxx-xxxx-xxxx-xxxxxxxxxxxx:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx_xxxx-xx-xx/billingProfiles/PBFV-xxxx-xxx-xxx/invoiceSections/GJ77-xxxx-xxx-xxx"
    }, {
        "billingProfileDisplayName": "Contoso finance",
        "billingProfileId": "/providers/Microsoft.Billing/billingAccounts/5e98e158-xxxx-xxxx-xxxx-xxxxxxxxxxxx:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx_xxxx-xx-xx/billingProfiles/PBFV-xxxx-xxx-xxx",
        "enabledAzurePlans": [{
            "productId": "DZH318Z0BPS6",
            "skuId": "0001",
            "skuDescription": "Microsoft Azure Plan"
        }, {
            "productId": "DZH318Z0BPS6",
            "skuId": "0002",
            "skuDescription": "Microsoft Azure Plan for DevTest"
        }],
        "invoiceSectionDisplayName": "Testing",
        "invoiceSectionId": "/providers/Microsoft.Billing/billingAccounts/5e98e158-xxxx-xxxx-xxxx-xxxxxxxxxxxx:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx_xxxx-xx-xx/billingProfiles/PBFV-XXXX-XXX-XXX/invoiceSections/GJGR-XXXX-XXX-XXX"
  }]
}

```

Use la propiedad `invoiceSectionDisplayName` para identificar la sección de factura para la que desea crear suscripciones. Copie los valores de `invoiceSectionId` y `billingProfileId` y uno de los valores de `skuId` de la sección de factura. Por ejemplo, si desea crear una suscripción de tipo `Microsoft Azure plan` para la sección de factura `Development`, copie `/providers/Microsoft.Billing/billingAccounts/5e98e158-xxxx-xxxx-xxxx-xxxxxxxxxxxx:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx_2019-05-31/billingProfiles/PBFV-XXXX-XXX-XXX/invoiceSections/GJGR-XXXX-XXX-XXX`, `/providers/Microsoft.Billing/billingAccounts/5e98e158-xxxx-xxxx-xxxx-xxxxxxxxxxxx:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx_2019-05-31/billingProfiles/PBFV-xxxx-xxx-xxx` y `0001`. Pegue estos valores en algún lugar para poder usarlos en el paso siguiente.

### <a name="create-a-subscription-for-an-invoice-section"></a>Creación de una suscripción para una sección de factura

En el ejemplo siguiente se crea una suscripción llamada *Dev Team subscription* del tipo *Microsoft Azure Plan* para la sección de factura *Development*. La suscripción se facturará al perfil de facturación de *Contoso finance* y aparecerá en la sección *Development* de su factura.

Realice la siguiente solicitud; para ello, reemplace `<invoiceSectionId>` por el elemento `invoiceSectionId` copiado en el segundo paso (```/providers/Microsoft.Billing/billingAccounts/5e98e158-xxxx-xxxx-xxxx-xxxxxxxxxxxx:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx_2019-05-31/billingProfiles/PBFV-XXXX-XXX-XXX/invoiceSections/GJGR-XXXX-XXX-XXX```). Debe pasar los valores `billingProfileId` y `skuId` copiados en el segundo paso en los parámetros de solicitud de la API. Si quiere especificar propietarios, obtenga información sobre [cómo obtener identificadores de objeto de usuario](grant-access-to-create-subscription.md#userObjectId).

```json
POST https://management.azure.com<invoiceSectionId>/providers/Microsoft.Subscription/createSubscription?api-version=2018-11-01-preview
```

```json
'{"displayName": "Dev Team subscription",
  "billingProfileId": "<billingProfileId>",
  "skuId": "<skuId>",
  "owners": [
      {
        "objectId": "<userObjectId>"
      },
      {
        "objectId": "<servicePrincipalObjectId>"
      }
    ],
  "costCenter": "35683",
  "managementGroupId": "/providers/Microsoft.Management/managementGroups/xxxxxxx",",
}'

```

| Nombre del elemento  | Obligatorio | Tipo   | Descripción                                                                                               |
|---------------|----------|--------|-----------------------------------------------------------------------------------------------------------|
| `displayName` | Sí      | String | Nombre para mostrar de la suscripción.|
| `billingProfileId`   | Sí      | String | Identificador del perfil de facturación que se facturará por los cargos de la suscripción.  |
| `skuId` | Sí      | String | Identificador de SKU que determina el tipo de plan de Azure. |
| `owners`      | No       | String | Identificador de objeto de cualquier usuario o entidad de servicio que quiera agregarse como propietario de RBAC en la suscripción al crearla.  |
| `costCenter` | No      | String | Centro de coste asociado a la suscripción. Se muestra en el archivo CSV de uso. |
| `managementGroupId` | No      | String | Identificador del grupo de administración al que se agregará la suscripción. Para obtener la lista de grupos de administración, consulte [Grupos de administración: API de lista](/rest/api/resources/managementgroups/list). Use el identificador de un grupo de administración de la API. |

En la respuesta, se recupera un objeto `subscriptionCreationResult` para la supervisión. Cuando haya finalizado la creación de la suscripción, el objeto `subscriptionCreationResult` devolvería un objeto `subscriptionLink`, que tiene el identificador de suscripción.

## <a name="create-subscriptions-for-an-mpa-billing-account"></a>Creación de suscripciones para una cuenta de facturación de MPA

### <a name="prerequisites"></a>Requisitos previos

Debe tener un rol de administrador global o de agente de administración en la cuenta del proveedor de soluciones en la nube de su organización para crear una suscripción para la cuenta de facturación. Para más información, consulte [Centro de partners: Asignar roles y permisos de usuarios](/partner-center/permissions-overview).

El ejemplo que se muestra a continuación usa API REST. Actualmente, no están admitidos PowerShell ni la CLI de Azure.

### <a name="find-the-billing-accounts-that-you-have-access-to"></a>Búsqueda de cuentas de facturación a las que tiene acceso

Realice la solicitud siguiente para mostrar todas las cuentas de facturación a las que tiene acceso.

```json
GET https://management.azure.com/providers/Microsoft.Billing/billingAccounts?api-version=2019-10-01-preview
```
La respuesta de la API muestra las cuentas de facturación.

```json
{
  "value": [
    {
      "id": "/providers/Microsoft.Billing/billingAccounts/99a13315-xxxx-xxxx-xxxx-xxxxxxxxxxxx:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx_xxxx-xx-xx",
      "name": "99a13315-xxxx-xxxx-xxxx-xxxxxxxxxxxx:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx_xxxx-xx-xx",
      "properties": {
        "accountId": "5e98e158-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
        "accountStatus": "Active",
        "accountType": "Enterprise",
        "agreementType": "MicrosoftPartnerAgreement",
        "displayName": "Contoso",
        "hasReadAccess": true,
        "organizationId": "1d100e69-xxxx-xxxx-xxxx-xxxxxxxxxxxxx_xxxx-xx-xx"
      },
      "type": "Microsoft.Billing/billingAccounts"
    },
    {
      "id": "/providers/Microsoft.Billing/billingAccounts/4f89e155-xxxx-xxxx-xxxx-xxxxxxxxxxxx:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx_xxxx-xx-xx",
      "name": "4f89e155-xxxx-xxxx-xxxx-xxxxxxxxxxxx:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx_xxxx-xx-xx",
      "properties": {
        "accountId": "4f89e155-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
        "accountStatus": "Active",
        "accountType": "Enterprise",
        "agreementType": "MicrosoftCustomerAgreement",
        "displayName": "Fabrikam",
        "hasReadAccess": true,
        "organizationId": "1d100e69-xxxx-xxxx-xxxx-xxxxxxxxxxxxx_xxxx-xx-xx"
      },
      "type": "Microsoft.Billing/billingAccounts"
    }
  ]
}

```
Use la propiedad `displayName` para identificar la cuenta de facturación para la que desea crear suscripciones. Asegúrese de que el valor agreeementType de la cuenta sea *MicrosoftPartnerAgreement*. Copie el valor `name` de la cuenta. Por ejemplo, si desea crear una suscripción para la cuenta de facturación `Contoso`, debe copiar `99a13315-xxxx-xxxx-xxxx-xxxxxxxxxxxx:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx_xxxx-xx-xx`. Pegue este valor en algún lugar para poder usarlo en el paso siguiente.

### <a name="find-customers-that-have-azure-plans"></a>Búsqueda de clientes que tienen planes de Azure

Realice la solicitud siguiente, reemplazando `<billingAccountName>` por el valor `name` copiado en el primer paso (```5e98e158-xxxx-xxxx-xxxx-xxxxxxxxxxxx:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx_xxxx-xx-xx```) para enumerar todos los clientes de la cuenta de facturación para los que puede crear suscripciones de Azure.

```json
GET https://management.azure.com/providers/Microsoft.Billing/billingAccounts/<billingAccountName>/customers?api-version=2019-10-01-preview
```
La respuesta de la API muestra los clientes de la cuenta de facturación con planes de Azure. Puede crear suscripciones para estos clientes.

```json
{
  "value": [
    {
      "id": "/providers/Microsoft.Billing/billingAccounts/99a13315-xxxx-xxxx-xxxx-xxxxxxxxxxxx:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx_xxxx-xx-xx/customers/2281f543-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
      "name": "2281f543-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
      "properties": {
        "billingProfileDisplayName": "Contoso USD",
        "billingProfileId": "/providers/Microsoft.Billing/billingAccounts/99a13315-xxxx-xxxx-xxxx-xxxxxxxxxxxx:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx_xxxx-xx-xx/billingProfiles/JUT6-xxxx-xxxx-xxxx",
        "displayName": "Fabrikam toys"
      },
      "type": "Microsoft.Billing/billingAccounts/customers"
    },
    {
      "id": "/providers/Microsoft.Billing/billingAccounts/99a13315-xxxx-xxxx-xxxx-xxxxxxxxxxxx:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx_xxxx-xx-xx/customers/97c3fac4-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
      "name": "97c3fac4-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
      "properties": {
        "billingProfileDisplayName": "Fabrikam sports",
        "billingProfileId": "/providers/Microsoft.Billing/billingAccounts/99a13315-xxxx-xxxx-xxxx-xxxxxxxxxxxx:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx_xxxx-xx-xx/billingProfiles/JUT6-xxxx-xxxx-xxxx",
        "displayName": "Fabrikam bakery"
      },
      "type": "Microsoft.Billing/billingAccounts/customers"
    }]
}

```

Use la propiedad `displayName` para identificar al cliente para el que desea crear suscripciones. Copie el valor `id` del cliente. Por ejemplo, si desea crear una suscripción para `Fabrikam toys`, debe copiar `/providers/Microsoft.Billing/billingAccounts/99a13315-xxxx-xxxx-xxxx-xxxxxxxxxxxx:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx_xxxx-xx-xx/customers/2281f543-xxxx-xxxx-xxxx-xxxxxxxxxxxx`. Pegue este valor en algún lugar para usarlo en los pasos siguientes.

### <a name="optional-for-indirect-providers-get-the-resellers-for-a-customer"></a>Opcional para proveedores indirectos: Obtención de los revendedores de un cliente

Como proveedor indirecto del modelo de dos niveles de CSP, puede especificar un revendedor mientras crea suscripciones para los clientes.

Realice la solicitud siguiente, reemplazando `<customerId>` por el valor `id` copiado en el segundo paso (```/providers/Microsoft.Billing/billingAccounts/99a13315-xxxx-xxxx-xxxx-xxxxxxxxxxxx:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx_xxxx-xx-xx/customers/2281f543-xxxx-xxxx-xxxx-xxxxxxxxxxxx```) para enumerar todos los revendedores que están disponibles para un cliente.

```json
GET https://management.azure.com<customerId>?$expand=resellers&api-version=2019-10-01-preview
```
La respuesta de la API muestra los revendedores del cliente:

```json
{
  "value": [{
  "id": "/providers/Microsoft.Billing/billingAccounts/99a13315-xxxx-xxxx-xxxx-xxxxxxxxxxxx:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx_xxxx-xx-xx/customers/2ed2c490-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "name": "2ed2c490-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "type": "Microsoft.Billing/billingAccounts/customers",
  "properties": {
    "displayName": "Fabrikam toys",
    "resellers": [
      {
        "resellerId": "3xxxxx",
        "description": "Wingtip"
      }
    ]
  }
},
{
  "id": "/providers/Microsoft.Billing/billingAccounts/99a13315-xxxx-xxxx-xxxx-xxxxxxxxxxxx:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx_xxxx-xx-xx/customers/4ed2c793-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "name": "4ed2c793-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "type": "Microsoft.Billing/billingAccounts/customers",
  "properties": {
    "displayName": "Fabrikam toys",
    "resellers": [
      {
        "resellerId": "5xxxxx",
        "description": "Tailspin"
      }
    ]
  }
}]
}
```
Use la propiedad `description` para identificar al revendedor que se va a asociar a la suscripción. Copie el valor `resellerId` del revendedor. Por ejemplo, si desea asociar `Wingtip`, debe copiar `3xxxxx`. Pegue este valor en algún lugar para poder usarlo en el paso siguiente.

### <a name="create-a-subscription-for-a-customer"></a>Creación de una suscripción para un cliente

En el ejemplo siguiente, se crea una suscripción denominada *Dev Team subscription* para *Fabrikam toys* y se asocia el revendedor *Wingtip* a la suscripción. T

Realice la siguiente solicitud; para ello, reemplace `<customerId>` por el elemento `id` copiado en el segundo paso (```/providers/Microsoft.Billing/billingAccounts/99a13315-xxxx-xxxx-xxxx-xxxxxxxxxxxx:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx_xxxx-xx-xx/customers/2281f543-xxxx-xxxx-xxxx-xxxxxxxxxxxx```). Debe pasar el valor *resellerId* copiado en el segundo paso en los parámetros de solicitud de la API.

```json
POST https://management.azure.com<customerId>/providers/Microsoft.Subscription/createSubscription?api-version=2018-11-01-preview
```

```json
'{"displayName": "Dev Team subscription",
  "skuId": "0001",
  "resellerId": "<resellerId>",
}'
```

| Nombre del elemento  | Obligatorio | Tipo   | Descripción                                                                                               |
|---------------|----------|--------|-----------------------------------------------------------------------------------------------------------|
| `displayName` | Sí      | String | Nombre para mostrar de la suscripción.|
| `skuId` | Sí      | String | El identificador de SKU del plan de Azure. Use *0001* para las suscripciones de tipo Microsoft Azure Plan. |
| `resellerId`      | No       | String | Use el identificador de MPN del revendedor que se va a asociar a la suscripción.  |

En la respuesta, se recupera un objeto `subscriptionCreationResult` para la supervisión. Cuando haya finalizado la creación de la suscripción, el objeto `subscriptionCreationResult` devolvería un objeto `subscriptionLink`, que tiene el identificador de suscripción.

## <a name="next-steps"></a>Pasos siguientes

* Para obtener un ejemplo sobre cómo crear suscripciones de Contrato Enterprise (EA) con. NET, consulte el [ejemplo de código en GitHub](https://github.com/Azure-Samples/create-azure-subscription-dotnet-core).
* Ahora que ha creado una suscripción, puede conceder dicha capacidad a otros usuarios y entidades de servicio. Para más información, vea [Concesión de acceso para crear suscripciones de EA (versión preliminar)](grant-access-to-create-subscription.md).
* Para obtener más información sobre cómo administrar grandes cantidades de suscripciones mediante grupos de administración, consulte [Organización de los recursos con grupos de administración de Azure](../../governance/management-groups/overview.md).