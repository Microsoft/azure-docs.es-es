---
title: Concesión de acceso para crear suscripciones de Azure Enterprise | Microsoft Docs
description: Obtenga información sobre cómo conceder a un usuario o a una entidad de servicio la capacidad de crear mediante programación las suscripciones de Azure Enterprise.
services: azure-resource-manager
author: jureid
manager: jureid
editor: ''
ms.assetid: ''
ms.service: azure-resource-manager
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 04/09/2019
ms.author: jureid
ms.openlocfilehash: 742658e36da956c46bd932b59903e68786c65b93
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65794576"
---
# <a name="grant-access-to-create-azure-enterprise-subscriptions-preview"></a>Concesión de acceso para crear suscripciones de Azure Enterprise (versión preliminar)

Como cliente de Azure del [Contrato Enterprise (EA)](https://azure.microsoft.com/pricing/enterprise-agreement/), puede conceder a otro usuario u otra entidad de servicio permisos para crear suscripciones facturadas en su cuenta. En este artículo, aprenderá a usar el [control de acceso basado en rol (RBAC)](../active-directory/role-based-access-control-configure.md) para compartir la capacidad de crear suscripciones y cómo auditar creaciones de suscripciones. Debe tener el rol de propietario en la cuenta que desea compartir.

Para crear una suscripción, vea [Creación de suscripciones de Azure Enterprise mediante programación (versión preliminar)](programmatically-create-subscription.md).

## <a name="grant-subscription-creation-access-to-a-user-or-group"></a>Conceder acceso a la creación de suscripción a un usuario o grupo

Para crear suscripciones con una cuenta de inscripción, los usuarios deben tener la [rol de propietario de RBAC](../role-based-access-control/built-in-roles.md#owner) en esa cuenta. Puede conceder un usuario o un grupo de usuarios del rol de propietario de RBAC en una cuenta de inscripción, siga estos pasos:

### <a name="1-get-the-object-id-of-the-enrollment-account-you-want-to-grant-access-to"></a>1. Obtener el identificador de objeto del que desea conceder acceso a la cuenta de inscripción

Para conceder a otros usuarios el rol de propietario de RBAC en una cuenta de inscripción, debe ser propietario de la cuenta o de la cuenta de propietario de RBAC.

### <a name="resttabrest"></a>[REST](#tab/rest)

Para enumerar todas las cuentas de inscripción a que tiene acceso la solicitud:

```json
GET https://management.azure.com/providers/Microsoft.Billing/enrollmentAccounts?api-version=2018-03-01-preview
```

Azure responde con una lista de todas las cuentas de inscripción a las que tiene acceso:

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

Use el `principalName` propiedad para identificar la cuenta que desea conceder acceso de propietario de RBAC a. Copia el `name` de esa cuenta. Por ejemplo, si desea conceder acceso de propietario de RBAC a la SignUpEngineering@contoso.com cuenta de inscripción, copiaría ```747ddfe5-xxxx-xxxx-xxxx-xxxxxxxxxxxx```. Este es el identificador de objeto de la cuenta de inscripción. Pegue este valor en algún lugar para que se puede usar en el paso siguiente como `enrollmentAccountObjectId`.

### <a name="powershelltabazure-powershell"></a>[PowerShell](#tab/azure-powershell)

Abra [Azure Cloud Shell](https://shell.azure.com/) y seleccione PowerShell.

Use el cmdlet [Get-AzEnrollmentAccount](/powershell/module/az.billing/get-azenrollmentaccount) para mostrar todas las cuentas de inscripción a las que tiene acceso.

```azurepowershell-interactive
Get-AzEnrollmentAccount
```

Azure responde con una lista de cuentas de inscripción que tiene acceso:

```azurepowershell
ObjectId                               | PrincipalName
747ddfe5-xxxx-xxxx-xxxx-xxxxxxxxxxxx   | SignUpEngineering@contoso.com
4cd2fcf6-xxxx-xxxx-xxxx-xxxxxxxxxxxx   | BillingPlatformTeam@contoso.com
```

Use el `principalName` propiedad para identificar la cuenta que desea conceder acceso de propietario de RBAC a. Copia el `ObjectId` de esa cuenta. Por ejemplo, si desea conceder acceso de propietario de RBAC a la SignUpEngineering@contoso.com cuenta de inscripción, copiaría ```747ddfe5-xxxx-xxxx-xxxx-xxxxxxxxxxxx```. Pegue este Id. de objeto en alguna parte para que se puede usar en el paso siguiente según la `enrollmentAccountObjectId`.

### <a name="azure-clitabazure-cli"></a>[CLI de Azure](#tab/azure-cli)

Use el comando [az billing enrollment-account list](https://aka.ms/EASubCreationPublicPreviewCLI) para mostrar todas las cuentas de inscripción a las que tiene acceso.

```azurecli-interactive 
az billing enrollment-account list
```

Azure responde con una lista de cuentas de inscripción que tiene acceso:

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

Use el `principalName` propiedad para identificar la cuenta que desea conceder acceso de propietario de RBAC a. Copia el `name` de esa cuenta. Por ejemplo, si desea conceder acceso de propietario de RBAC a la SignUpEngineering@contoso.com cuenta de inscripción, copiaría ```747ddfe5-xxxx-xxxx-xxxx-xxxxxxxxxxxx```. Este es el identificador de objeto de la cuenta de inscripción. Pegue este valor en algún lugar para que se puede usar en el paso siguiente como `enrollmentAccountObjectId`.

<a id="userObjectId"></a>

### <a name="2-get-object-id-of-the-user-or-group-you-want-to-give-the-rbac-owner-role-to"></a>2. Obtener Id. de objeto del usuario o grupo que desea asignar al rol de propietario de RBAC para

1. En Azure portal, busque **Azure Active Directory**.
1. Si desea conceder acceso a un usuario, haga clic en **usuarios** en el menú de la izquierda. Si desea conceder acceso a un grupo, haga clic en **grupos**.
1. Seleccione el usuario o grupo que desea asignar al rol de propietario de RBAC a.
1. Si ha seleccionado un usuario, encontrará el identificador de objeto en la página de perfil. Si ha seleccionado un grupo, será el identificador de objeto en la página información general. Copia el **ObjectID** haciendo clic en el icono a la derecha del cuadro de texto. Pegue este en algún lugar para que se puede usar en el paso siguiente como `userObjectId`.

### <a name="3-grant-the-user-or-group-the-rbac-owner-role-on-the-enrollment-account"></a>3. Conceda al usuario o grupo el rol de propietario de RBAC en la cuenta de inscripción

Con los valores que recopiló en los dos primeros pasos, conceda al usuario o grupo el rol de propietario de RBAC en la cuenta de inscripción.

### <a name="resttabrest-2"></a>[REST](#tab/rest-2)

Ejecute el siguiente comando, reemplazando ```<enrollmentAccountObjectId>``` con el `name` que copió en el primer paso (```747ddfe5-xxxx-xxxx-xxxx-xxxxxxxxxxxx```). Reemplace ```<userObjectId>``` con el identificador de objeto que ha copiado en el segundo paso.

```json
PUT  https://management.azure.com/providers/Microsoft.Billing/enrollmentAccounts/<enrollmentAccountObjectId>/providers/Microsoft.Authorization/roleAssignments/<roleAssignmentGuid>?api-version=2015-07-01

{
  "properties": {
    "roleDefinitionId": "/providers/Microsoft.Billing/enrollmentAccounts/providers/Microsoft.Authorization/roleDefinitions/<ownerRoleDefinitionId>",
    "principalId": "<userObjectId>"
  }
}
```

Cuando el rol de propietario se asigna correctamente en el ámbito de la cuenta de inscripción, Azure responde con información de la asignación de roles:

```json
{
  "properties": {
    "roleDefinitionId": "/providers/Microsoft.Billing/enrollmentAccounts/providers/Microsoft.Authorization/roleDefinitions/<ownerRoleDefinitionId>",
    "principalId": "<userObjectId>",
    "scope": "/providers/Microsoft.Billing/enrollmentAccounts/747ddfe5-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
    "createdOn": "2018-03-05T08:36:26.4014813Z",
    "updatedOn": "2018-03-05T08:36:26.4014813Z",
    "createdBy": "<assignerObjectId>",
    "updatedBy": "<assignerObjectId>"
  },
  "id": "/providers/Microsoft.Billing/enrollmentAccounts/providers/Microsoft.Authorization/roleDefinitions/<ownerRoleDefinitionId>",
  "type": "Microsoft.Authorization/roleAssignments",
  "name": "<roleAssignmentGuid>"
}
```

### <a name="powershelltabazure-powershell-2"></a>[PowerShell](#tab/azure-powershell-2)

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Ejecute el siguiente [New AzRoleAssignment](../active-directory/role-based-access-control-manage-access-powershell.md) comando, reemplazando ```<enrollmentAccountObjectId>``` con el `ObjectId` recopilados en el primer paso (```747ddfe5-xxxx-xxxx-xxxx-xxxxxxxxxxxx```). Reemplace ```<userObjectId>``` con el objeto identificador recopilados en el segundo paso.

```azurepowershell-interactive
New-AzRoleAssignment -RoleDefinitionName Owner -ObjectId <userObjectId> -Scope /providers/Microsoft.Billing/enrollmentAccounts/<enrollmentAccountObjectId>
```

### <a name="azure-clitabazure-cli-2"></a>[CLI de Azure](#tab/azure-cli-2)

Ejecute el siguiente [crear asignación de roles az](../active-directory/role-based-access-control-manage-access-azure-cli.md) comando, reemplazando ```<enrollmentAccountObjectId>``` con el `name` que copió en el primer paso (```747ddfe5-xxxx-xxxx-xxxx-xxxxxxxxxxxx```). Reemplace ```<userObjectId>``` con el objeto identificador recopilados en el segundo paso.

```azurecli-interactive
az role assignment create --role Owner --assignee-object-id <userObjectId> --scope /providers/Microsoft.Billing/enrollmentAccounts/<enrollmentAccountObjectId>
```

----

Una vez que un usuario se convierte en propietario de RBAC para la cuenta de inscripción, pueden [crear suscripciones mediante programación](programmatically-create-subscription.md) debajo de él. Una suscripción creada por un usuario delegado aún tiene el propietario de la cuenta original como administrador de servicios, pero también tiene el usuario delegado como propietario de RBAC de forma predeterminada.

## <a name="audit-who-created-subscriptions-using-activity-logs"></a>Auditoría que ha creado suscripciones con registros de actividad

Para realizar el seguimiento de las suscripciones creadas a través de esta API, use la [API de registro de actividad de inquilinos](/rest/api/monitor/tenantactivitylogs). Actualmente no es posible usar PowerShell, la CLI ni Azure Portal para realizar el seguimiento de la creación de suscripciones.

1. Como administrador de inquilinos del inquilino de Azure AD, [eleve los privilegios de acceso](../active-directory/role-based-access-control-tenant-admin-access.md) y asigne un rol de lector al usuario de auditoría con el ámbito `/providers/microsoft.insights/eventtypes/management`.
1. Como usuario de auditoría, llame a la [API de registro de actividad de inquilinos](/rest/api/monitor/tenantactivitylogs) para ver las actividades de creación de suscripciones. Ejemplo:

```
GET "/providers/Microsoft.Insights/eventtypes/management/values?api-version=2015-04-01&$filter=eventTimestamp ge '{greaterThanTimeStamp}' and eventTimestamp le '{lessThanTimestamp}' and eventChannels eq 'Operation' and resourceProvider eq 'Microsoft.Subscription'" 
```

> Para llamar cómodamente a esta API desde la línea de comandos, pruebe [ARMClient](https://github.com/projectkudu/ARMClient).

## <a name="next-steps"></a>Pasos siguientes

* Ahora que el usuario o la entidad de servicio tienen permiso para crear una suscripción, puede usar dicha identidad para [crear suscripciones de Azure Enterprise mediante programación](programmatically-create-subscription.md).
* Para obtener un ejemplo sobre cómo crear suscripciones con. NET, vea [ejemplo de código en GitHub](https://github.com/Azure-Samples/create-azure-subscription-dotnet-core).
* Para obtener más información sobre Azure Resource Manager y sus API, vea [Información general de Azure Resource Manager](resource-group-overview.md).
* Para obtener más información sobre cómo administrar grandes cantidades de suscripciones mediante grupos de administración, consulte [Organización de los recursos con grupos de administración de Azure](management-groups-overview.md).
* Para consultar una guía completa de procedimientos recomendados para grandes organizaciones sobre la gobernanza de suscripciones, vea [Scaffolding empresarial de Azure: gobernanza de suscripciones preceptiva](/azure/architecture/cloud-adoption-guide/subscription-governance)
