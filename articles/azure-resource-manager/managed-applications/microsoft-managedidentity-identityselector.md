---
title: Elemento de la interfaz de usuario IdentitySelector
description: Describe el elemento de la interfaz de usuario Microsoft.ManagedIdentity.IdentitySelector para Azure Portal. Úselo para asignar identidades administradas a un recurso.
author: tfitzmac
ms.topic: conceptual
ms.date: 02/06/2020
ms.author: tomfitz
ms.openlocfilehash: 91e3cd0b99825fd72eb342ce7a8555b046455538
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87063407"
---
# <a name="microsoftmanagedidentityidentityselector-ui-element"></a>Elemento de la interfaz de usuario Microsoft.ManagedIdentity.IdentitySelector

Control para asignar [identidades administradas](../../active-directory/managed-identities-azure-resources/overview.md) para un recurso en una implementación.

## <a name="ui-sample"></a>Ejemplo de interfaz de usuario

El control está formado por los siguientes elementos:

![Primer paso de Microsoft.ManagedIdentity.IdentitySelector](./media/managed-application-elements/microsoft-managedidentity-identityselector-1.png)

Cuando el usuario selecciona **Agregar**, se abre el formulario siguiente. El usuario puede seleccionar una o varias identidades asignadas por el usuario para el recurso.

![Segundo paso de Microsoft.ManagedIdentity.IdentitySelector](./media/managed-application-elements/microsoft-managedidentity-identityselector-2.png)

Las identidades seleccionadas se muestran en la tabla. El usuario puede agregar o eliminar elementos de esta tabla.

![Tercer paso de Microsoft.ManagedIdentity.IdentitySelector](./media/managed-application-elements/microsoft-managedidentity-identityselector-3.png)

## <a name="schema"></a>Schema

```json
{
  "name": "identity",
  "type": "Microsoft.ManagedIdentity.IdentitySelector",
  "label": "Managed Identity Configuration",
  "toolTip": {
    "systemAssignedIdentity": "Enable system assigned identity to grant the resource access to other existing resources.",
    "userAssignedIdentity": "Add user assigned identities to grant the resource access to other existing resources."
  },
  "defaultValue": {
    "systemAssignedIdentity": "Off"
  },
  "options": {
    "hideSystemAssignedIdentity": false,
    "hideUserAssignedIdentity": false
  },
  "visible": true
}
```

## <a name="sample-output"></a>Salida de ejemplo

```json
{
  "identity": {
    "value": {
      "type": "UserAssigned",
      "userAssignedIdentities": {
        "/subscriptions/xxxx/resourceGroups/TestResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/TestUserIdentity1": {}
      }
    }
  }
}
```

## <a name="remarks"></a>Observaciones

- Use **defaultValue.systemAssignedIdentity** para establecer un valor inicial para el control de opciones de la identidad asignada por el sistema. El valor predeterminado es **Off**. Se admiten los valores siguientes:
  - **On**: se asigna una identidad asignada por el sistema al recurso.
  - **Off**: no se asigna una identidad asignada por el sistema al recurso.
  - **OnOnly**: se asigna una identidad asignada por el sistema al recurso. Los usuarios no pueden editar este valor durante la implementación.
  - **OffOnly**: no se asigna una identidad asignada por el sistema al recurso. Los usuarios no pueden editar este valor durante la implementación.

- Si **options.hideSystemAssignedIdentity** está establecido en **true**, no se muestra la interfaz de usuario para configurar la identidad asignada por el sistema. El valor predeterminado de esta opción es **false**.
- Si **options.hideUserAssignedIdentity** está establecido en **true**, no se muestra la interfaz de usuario para configurar la identidad asignada por el usuario. No se asigna una identidad asignada por el usuario al recurso. El valor predeterminado de esta opción es **false**.

## <a name="next-steps"></a>Pasos siguientes

- Para ver una introducción sobre la creación de definiciones de interfaz de usuario, consulte [Introducción a CreateUiDefinition](create-uidefinition-overview.md).
- Para ver una descripción de las propiedades comunes de los elementos de interfaz de usuario, consulte [Elementos CreateUiDefinition](create-uidefinition-elements.md).