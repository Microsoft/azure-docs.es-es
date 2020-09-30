---
title: Detalles de la estructura de exención de directiva
description: Aquí se describe la definición de exención de directiva usada por Azure Policy para eximir recursos de la evaluación de iniciativas o definiciones.
ms.date: 09/22/2020
ms.topic: conceptual
ms.openlocfilehash: 1fd14d31824dc86dcd3788607030f28f978f5801
ms.sourcegitcommit: bdd5c76457b0f0504f4f679a316b959dcfabf1ef
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90968050"
---
# <a name="azure-policy-exemption-structure"></a>Estructura de exención de Azure Policy

La característica de exenciones de Azure Policy (versión preliminar) se usa para _eximir_ una jerarquía de recursos o un recurso individual de la evaluación de iniciativas o definiciones. Los recursos que están _exentos_ cuentan para el cumplimiento general, pero no se pueden evaluar ni tienen una renuncia temporal. Para obtener más información, vea [Descripción del ámbito de Azure Policy](./scope.md). Las exenciones de Azure Policy solo funcionan con los [modos de Resource Manager](./definition-structure.md#resource-manager-modes) y no con los **modos del proveedor de recursos**.

> [!IMPORTANT]
> Esta característica es gratuita durante la **versión preliminar**. Para obtener información detallada sobre los precios, vea [Precios de Azure Policy](https://azure.microsoft.com/pricing/details/azure-policy/). Para más información sobre las versiones preliminares, consulte [Términos de uso complementarios de las versiones preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

Use JSON para crear una exención de directiva. La exención de directiva contiene elementos para:

- nombre para mostrar
- description
- metadata
- asignación de directiva
- definiciones de directivas en una iniciativa
- categoría de exención
- expiration

> [!NOTE]
> Una exención de directiva se crea como un objeto secundario en la jerarquía de recursos o el recurso individual a los que se ha concedido la exención, por lo que el destino no está incluido en la definición de la exención.

Por ejemplo, en el siguiente JSON se puede ver una exención de directiva de la categoría de **renuncia** de un recurso en una asignación de iniciativa denominada `resourceShouldBeCompliantInit`. El recurso está _exento_ solo de dos de las definiciones de directiva de la iniciativa, la definición de directiva personalizada `customOrgPolicy` (referencia `requiredTags`) y la definición de directiva integrada "Allowed locations" (id.: `e56962a6-4747-49cd-b67b-bf8b01975c4c`, referencia `allowedLocations`):

```json
{
    "id": "/subscriptions/{subId}/resourceGroups/ExemptRG/providers/Microsoft.Authorization/policyExemptions/resourceIsNotApplicable",
    "name": "resourceIsNotApplicable",
    "type": "Microsoft.Authorization/policyExemptions",
    "properties": {
        "displayName": "This resource is scheduled for deletion",
        "description": "This resources is planned to be deleted by end of quarter and has been granted a waiver to the policy.",
        "metadata": {
            "requestedBy": "Storage team",
            "approvedBy": "IA",
            "approvedOn": "2020-07-26T08:02:32.0000000Z",
            "ticketRef": "4baf214c-8d54-4646-be3f-eb6ec7b9bc4f"
        },
        "policyAssignmentId": "/subscriptions/{mySubscriptionID}/providers/Microsoft.Authorization/policyAssignments/resourceShouldBeCompliantInit",
        "policyDefinitionReferenceIds": [
            "requiredTags",
            "allowedLocations"
        ],
        "exemptionCategory": "waiver",
        "expiresOn": "2020-12-31T23:59:00.0000000Z"
    }
}
```

Fragmento de código de la iniciativa relacionada con el elemento `policyDefinitionReferenceIds` coincidente usado por la exención de directiva:

```json
"policyDefinitions": [
    {
        "policyDefinitionId": "/subscriptions/{mySubscriptionID}/providers/Microsoft.Authorization/policyDefinitions/customOrgPolicy",
        "policyDefinitionReferenceId": "requiredTags",
        "parameters": {
            "reqTags": {
                "value": "[parameters('init_reqTags')]"
            }
        }
    },
    {
        "policyDefinitionId": "/providers/Microsoft.Authorization/policyDefinitions/e56962a6-4747-49cd-b67b-bf8b01975c4c",
        "policyDefinitionReferenceId": "allowedLocations",
        "parameters": {
            "listOfAllowedLocations": {
                "value": "[parameters('init_listOfAllowedLocations')]"
            }
        }
    }
]
```

## <a name="display-name-and-description"></a>Nombre para mostrar y descripción

Use **displayName** y **description** para identificar la exención de directiva y proporcionar contexto para su uso con el recurso específico. **displayName** tiene una longitud máxima de _128_ caracteres y **description** tiene una longitud máxima de _512_ caracteres.

## <a name="metadata"></a>Metadatos

La propiedad **metadata** permite crear cualquier propiedad secundaria necesaria para almacenar información relevante. En el ejemplo anterior, las propiedades **requestedBy**, **approvedBy**, **approvedOn** y **ticketRef** contienen valores del cliente para proporcionar información sobre quién ha solicitado la exención, quién y cuándo la ha aprobado, y un vale de seguimiento interno de la solicitud. Estas propiedades **metadata** son ejemplos, pero no son necesarias, y **metadata** no se limita a estas propiedades secundarias.

## <a name="policy-assignment-id"></a>Id. de asignación de directiva

Este campo debe ser el nombre de la ruta de acceso completa de una asignación de directiva o una asignación de iniciativa.
`policyAssignmentId` es una cadena y no una matriz. Esta propiedad define de qué asignación está _exenta_ la jerarquía de recursos primaria o el recurso individual.

## <a name="policy-definition-ids"></a>Identificadores de definición de directiva

Si `policyAssignmentId` corresponde a una asignación de iniciativa, se puede usar la propiedad `policyDefinitionReferenceIds` para especificar las definiciones de directiva de la iniciativa en las que el recurso del asunto tiene una exención. Como el recurso puede estar exento de una o varias definiciones de directivas incluidas, esta propiedad es una _matriz_. Los valores deben coincidir con los valores de la definición de iniciativa de los campos `policyDefinitions.policyDefinitionReferenceId`.

## <a name="exemption-category"></a>Categoría de exención

Existen dos categorías de exención y se usan para agrupar las exenciones:

- **Mitigada**: Se concede la exención porque la intención de la directiva se alcanza a través de otro método.
- **Renuncia**: Se concede la exención porque el estado de no cumplimiento del recurso se acepta temporalmente. Otra razón para usar esta categoría es para una jerarquía de recursos o recursos que se debe excluir de una o más definiciones en una iniciativa, pero no debe excluirse de toda la iniciativa.

## <a name="expiration"></a>Expiration

Para establecer cuándo una jerarquía de recursos o un recurso individual ya no están _exentos_ en una asignación, establezca la propiedad `expiresOn`. Esta propiedad opcional debe tener el formato DateTime universal ISO 8601 `yyyy-MM-ddTHH:mm:ss.fffffffZ`.

> [!NOTE]
> Las exenciones de directiva no se eliminan cuando se alcanza la fecha `expiresOn`. El objeto se conserva para el mantenimiento de registros, pero ya no se respeta la exención.

## <a name="required-permissions"></a>Permisos necesarios

Los permisos RBAC de Azure necesarios para administrar objetos de exención de directiva se encuentran en el grupo de operaciones `Microsoft.Authorization/policyExemptions`. Los roles integrados [Colaborador de la directiva de recursos](../../../role-based-access-control/built-in-roles.md#resource-policy-contributor) y [Administrador de seguridad](../../../role-based-access-control/built-in-roles.md#security-admin) tienen los permisos `read` y `write`, y [Escritor de datos de Policy Insights (versión preliminar)](../../../role-based-access-control/built-in-roles.md#policy-insights-data-writer-preview) tiene el permiso `read`.

Las exenciones tienen medidas de seguridad adicionales debido al impacto de la concesión de una exención. Además de exigir la operación `Microsoft.Authorization/policyExemptions/write` en la jerarquía de recursos o el recurso individual, el creador de una exención debe tener el verbo `exempt/Action` en la asignación de destino.

## <a name="next-steps"></a>Pasos siguientes

- Más información sobre la [estructura de la definición de directiva](./definition-structure.md).
- Obtenga información acerca de cómo se pueden [crear directivas mediante programación](../how-to/programmatically-create.md).
- Obtenga información sobre cómo [obtener datos de cumplimiento](../how-to/get-compliance-data.md).
- Obtenga información sobre cómo [corregir recursos no compatibles](../how-to/remediate-resources.md).
- En [Organización de los recursos con grupos de administración de Azure](../../management-groups/overview.md), obtendrá información sobre lo que es un grupo de administración.