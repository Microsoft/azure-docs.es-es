---
title: Concesión de acceso para crear suscripciones de Azure Enterprise
description: Obtenga información sobre cómo conceder a un usuario o a una entidad de servicio la capacidad de crear mediante programación las suscripciones de Azure Enterprise.
author: bandersmsft
ms.service: cost-management-billing
ms.subservice: billing
ms.reviewer: amberb
ms.topic: conceptual
ms.date: 08/26/2020
ms.author: banders
ms.openlocfilehash: b154d723e82d02ea864459ef65eb5c05c14ae336
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88943188"
---
# <a name="grant-access-to-create-azure-enterprise-subscriptions-preview"></a>Concesión de acceso para crear suscripciones de Azure Enterprise (versión preliminar)

Como cliente de Azure del [Contrato Enterprise (EA)](https://azure.microsoft.com/pricing/enterprise-agreement/), puede conceder a otro usuario u otra entidad de servicio permisos para crear suscripciones facturadas en su cuenta. En este artículo, aprenderá a usar el [control de acceso basado en rol (RBAC)](../../role-based-access-control/role-assignments-portal.md) para compartir la capacidad de crear suscripciones y cómo auditar creaciones de suscripciones. Debe tener el rol de propietario en la cuenta que desea compartir.

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

## <a name="grant-access"></a>Conceder acceso

Para [crear suscripciones en una cuenta de inscripción](programmatically-create-subscription.md), los usuarios deben contar con el [rol de propietario de RBAC](../../role-based-access-control/built-in-roles.md#owner) en esa cuenta. Puede conceder a un usuario o a un grupo de usuarios el rol de propietario de RBAC en una cuenta de inscripción si sigue estos pasos:

1. Obtenga el id. de objeto de la cuenta de inscripción a la que quiere conceder acceso

    Para conceder a otros usuarios el rol de propietario de RBAC en una cuenta de inscripción, debe ser propietario de la cuenta o un propietario de RBAC de la cuenta.

    # <a name="rest"></a>[REST](#tab/rest)

    Solicite mostrar todas las cuentas de inscripción a las que tiene acceso:

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

    Use la propiedad `principalName` para identificar la cuenta a la que quiere conceder acceso al propietario de RBAC. Copie el elemento `name` de esa cuenta. Por ejemplo, si quisiera conceder acceso al propietario de RBAC a la cuenta de inscripción SignUpEngineering@contoso.com, copiaría ```747ddfe5-xxxx-xxxx-xxxx-xxxxxxxxxxxx```. Es el identificador de objeto de la cuenta de inscripción. Pegue este valor en algún lugar para poder usarlo en el paso siguiente como `enrollmentAccountObjectId`.

    # <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

    Use el cmdlet [Get-AzEnrollmentAccount](/powershell/module/az.billing/get-azenrollmentaccount) para mostrar todas las cuentas de inscripción a las que tiene acceso. Seleccione **Probar ahora** para abrir [Azure Cloud Shell](https://shell.azure.com/). Para pegar el código, haga clic con el botón derecho en la ventana del shell y seleccione **Pegar**.

    ```azurepowershell-interactive
    Get-AzEnrollmentAccount
    ```

    Azure responde con una lista de las cuentas de inscripción a las que tiene acceso:

    ```azurepowershell
    ObjectId                               | PrincipalName
    747ddfe5-xxxx-xxxx-xxxx-xxxxxxxxxxxx   | SignUpEngineering@contoso.com
    4cd2fcf6-xxxx-xxxx-xxxx-xxxxxxxxxxxx   | BillingPlatformTeam@contoso.com
    ```

    Use la propiedad `principalName` para identificar la cuenta a la que quiere conceder acceso al propietario de RBAC. Copie el elemento `ObjectId` de esa cuenta. Por ejemplo, si quisiera conceder acceso al propietario de RBAC a la cuenta de inscripción SignUpEngineering@contoso.com, copiaría ```747ddfe5-xxxx-xxxx-xxxx-xxxxxxxxxxxx```. Pegue este identificador de objeto en alguna parte para poder usarlo en el paso siguiente como `enrollmentAccountObjectId`.

    # <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

    Use el comando [az billing enrollment-account list](https://aka.ms/EASubCreationPublicPreviewCLI) para mostrar todas las cuentas de inscripción a las que tiene acceso. Seleccione **Probar ahora** para abrir [Azure Cloud Shell](https://shell.azure.com/). Para pegar el código, haga clic con el botón derecho en la ventana del shell y seleccione **Pegar**.

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

    ---

    Use la propiedad `principalName` para identificar la cuenta a la que quiere conceder acceso al propietario de RBAC. Copie el elemento `name` de esa cuenta. Por ejemplo, si quisiera conceder acceso al propietario de RBAC a la cuenta de inscripción SignUpEngineering@contoso.com, copiaría ```747ddfe5-xxxx-xxxx-xxxx-xxxxxxxxxxxx```. Es el identificador de objeto de la cuenta de inscripción. Pegue este valor en algún lugar para poder usarlo en el paso siguiente como `enrollmentAccountObjectId`.

1. <a id="userObjectId"></a>Obtenga el id. del usuario o grupo al que quiere conceder el rol de propietario de RBAC

    1. En Azure Portal, busque **Azure Active Directory**.
    1. Si desea conceder acceso a un usuario, seleccione **Usuarios** en el menú de la izquierda. Para conceder acceso a un grupo, seleccione **Grupos**.
    1. Seleccione el usuario o grupo al que quiere asignar el rol de propietario de RBAC.
    1. Si ha seleccionado un usuario, encontrará el id. de objeto en la página de perfil. Si ha seleccionado un grupo, el identificador de objeto estará en la página introducción. Copie el valor de **ObjectID** seleccionando el icono de la derecha del cuadro de texto. Péguelo en algún lugar para poder usarlo en el paso siguiente como `userObjectId`.

1. Conceda al usuario o grupo el rol de propietario de RBAC en la cuenta de inscripción

    Con los valores que recopiló en los dos primeros pasos, conceda al usuario o grupo el rol de propietario de RBAC en la cuenta de inscripción.

    # <a name="rest"></a>[REST](#tab/rest-2)

    Ejecute el comando siguiente, reemplazando ```<enrollmentAccountObjectId>``` por el elemento `name` copiado en el primer paso (```747ddfe5-xxxx-xxxx-xxxx-xxxxxxxxxxxx```). Reemplace ```<userObjectId>``` con el identificador de objeto que ha copiado en el segundo paso.

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

    # <a name="powershell"></a>[PowerShell](#tab/azure-powershell-2)

    Ejecute el comando [New-AzRoleAssignment](../../role-based-access-control/role-assignments-powershell.md) siguiente, reemplazando ```<enrollmentAccountObjectId>``` con el valor `ObjectId` recopilado en el primer paso (```747ddfe5-xxxx-xxxx-xxxx-xxxxxxxxxxxx```). Reemplace ```<userObjectId>``` con el identificador de objeto recopilado en el segundo paso.

    ```azurepowershell-interactive
    New-AzRoleAssignment -RoleDefinitionName Owner -ObjectId <userObjectId> -Scope /providers/Microsoft.Billing/enrollmentAccounts/<enrollmentAccountObjectId>
    ```

    # <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli-2)

    Ejecute el comando [az role assignment create](../../role-based-access-control/role-assignments-cli.md) siguiente, reemplazando ```<enrollmentAccountObjectId>``` con el valor `name` recopilado en el primer paso (```747ddfe5-xxxx-xxxx-xxxx-xxxxxxxxxxxx```). Reemplace ```<userObjectId>``` con el identificador de objeto recopilado en el segundo paso.

    ```azurecli-interactive
    az role assignment create --role Owner --assignee-object-id <userObjectId> --scope /providers/Microsoft.Billing/enrollmentAccounts/<enrollmentAccountObjectId>
    ```

    Cuando un usuario se convierte en propietario de RBAC de su cuenta de inscripción, puede [crear suscripciones en ella mediante programación](programmatically-create-subscription.md). Una suscripción creada por un usuario delegado aún tiene el propietario de cuenta original como administrador de servicios, pero también tiene el usuario delegado como propietario de RBAC de forma predeterminada.

    ---

## <a name="audit-who-created-subscriptions-using-activity-logs"></a>Auditoría que ha creado suscripciones con registros de actividad

Para realizar el seguimiento de las suscripciones creadas a través de esta API, use la [API de registro de actividad de inquilinos](/rest/api/monitor/tenantactivitylogs). Actualmente no es posible usar PowerShell, la CLI ni Azure Portal para realizar el seguimiento de la creación de suscripciones.

1. Como administrador de inquilinos del inquilino de Azure AD, [eleve los privilegios de acceso](../../role-based-access-control/elevate-access-global-admin.md) y asigne un rol de lector al usuario de auditoría con el ámbito `/providers/microsoft.insights/eventtypes/management`. Este acceso está disponible en el rol [Lector](../../role-based-access-control/built-in-roles.md#reader), el rol [Colaborador de supervisión](../../role-based-access-control/built-in-roles.md#monitoring-contributor) o un [rol personalizado](../../role-based-access-control/custom-roles.md).
1. Como usuario de auditoría, llame a la [API de registro de actividad de inquilinos](/rest/api/monitor/tenantactivitylogs) para ver las actividades de creación de suscripciones. Ejemplo:

    ```
    GET "/providers/Microsoft.Insights/eventtypes/management/values?api-version=2015-04-01&$filter=eventTimestamp ge '{greaterThanTimeStamp}' and eventTimestamp le '{lessThanTimestamp}' and eventChannels eq 'Operation' and resourceProvider eq 'Microsoft.Subscription'"
    ```

Para llamar cómodamente a esta API desde la línea de comandos, pruebe [ARMClient](https://github.com/projectkudu/ARMClient).

## <a name="next-steps"></a>Pasos siguientes

* Ahora que el usuario o la entidad de servicio tienen permiso para crear una suscripción, puede usar dicha identidad para [crear suscripciones de Azure Enterprise mediante programación](programmatically-create-subscription.md).
* Para obtener un ejemplo sobre cómo crear suscripciones con. NET, vea [ejemplo de código en GitHub](https://github.com/Azure-Samples/create-azure-subscription-dotnet-core).
* Para obtener más información sobre Azure Resource Manager y sus API, vea [Información general de Azure Resource Manager](../../azure-resource-manager/management/overview.md).
* Para obtener más información sobre cómo administrar grandes cantidades de suscripciones mediante grupos de administración, consulte [Organización de los recursos con grupos de administración de Azure](../../governance/management-groups/overview.md).
* Para consultar una guía completa de procedimientos recomendados para grandes organizaciones sobre la gobernanza de suscripciones, vea [Scaffolding empresarial de Azure: gobernanza de suscripciones preceptiva](/azure/architecture/cloud-adoption-guide/subscription-governance)
