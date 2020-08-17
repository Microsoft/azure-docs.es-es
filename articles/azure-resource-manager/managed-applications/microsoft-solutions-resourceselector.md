---
title: Elemento de la interfaz de usuario de ResourceSelector
description: Describe el elemento de la interfaz de usuario Microsoft.Solutions.ArmApiControl para Azure Portal. Se usa para obtener una lista de los recursos existentes.
author: tfitzmac
ms.topic: conceptual
ms.date: 07/13/2020
ms.author: tomfitz
ms.openlocfilehash: d4b21f092733f6ebb3c2bf5aa06c1d2782323730
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87095360"
---
# <a name="microsoftsolutionsresourceselector-ui-element"></a>Elemento de la interfaz de usuario Microsoft.Solutions.ResourceSelector

ResourceSelector permite a los usuarios seleccionar un recurso existente de una suscripción.

## <a name="ui-sample"></a>Ejemplo de interfaz de usuario

![Microsoft.Solutions.ResourceSelector](./media/managed-application-elements/microsoft-solutions-resourceselector.png)

## <a name="schema"></a>Schema

```json
{
    "name": "storageSelector",
    "type": "Microsoft.Solutions.ResourceSelector",
    "label": "Select storage accounts",
    "resourceType": "Microsoft.Storage/storageAccounts",
    "options": {
        "filter": {
            "subscription": "onBasics",
            "location": "onBasics"
        }
    }
}
```

## <a name="sample-output"></a>Salida de ejemplo

```json
"name": "{resource-name}",
"id": "/subscriptions/{subscription-id}/resourceGroups/{resource-group}/providers/{resource-provider-namespace}/{resource-type}/{resource-name}",
"location": "{deployed-location}"
```

## <a name="remarks"></a>Observaciones

En la propiedad `resourceType`, proporcione el espacio de nombres del proveedor de recursos y el nombre del tipo de recurso que quiere mostrar en la lista.

La propiedad `filter` restringe las opciones disponibles para los recursos. Puede restringir los resultados por ubicación o suscripción. Para mostrar solo los recursos que coinciden con la selección en los aspectos básicos, use `onBasics`. Para mostrar todos los recursos, use `all`. El valor predeterminado es `all`.

## <a name="next-steps"></a>Pasos siguientes

* Para ver una introducción sobre la creación de definiciones de interfaz de usuario, consulte [Introducción a CreateUiDefinition](create-uidefinition-overview.md).
* Para ver una descripción de las propiedades comunes de los elementos de interfaz de usuario, consulte [Elementos CreateUiDefinition](create-uidefinition-elements.md).
