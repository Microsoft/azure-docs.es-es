---
title: Elemento de la interfaz de usuario Checkbox
description: Describe el elemento de la interfaz de usuario Microsoft.Common.CheckBox para Azure Portal. Permite que los usuarios activen o desactiven una opción.
author: tfitzmac
ms.topic: conceptual
ms.date: 07/09/2020
ms.author: tomfitz
ms.openlocfilehash: 9f40f223cca34df58d2af7373d8f60fd7f383162
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87095275"
---
# <a name="microsoftcommoncheckbox-ui-element"></a>Elemento de interfaz de usuario Microsoft.Common.CheckBox

El control CheckBox permite que los usuarios activen o desactiven una opción. El control devuelve **true** cuando el control se activa o **false**, cuando no se activa.

## <a name="ui-sample"></a>Ejemplo de interfaz de usuario

:::image type="content" source="./media/managed-application-elements/microsoft-common-checkbox.png" alt-text="Microsoft.Common.CheckBox":::

## <a name="schema"></a>Schema

```json
{
    "name": "legalAccept",
    "type": "Microsoft.Common.CheckBox",
    "label": "I agree to the terms and conditions.",
    "constraints": {
        "required": true,
        "validationMessage": "Please acknowledge the legal conditions."
    }
}
```

## <a name="sample-output"></a>Salida de ejemplo

```json
true
```

## <a name="remarks"></a>Observaciones

Cuando establece **required** en **true**, el usuario debe activar la casilla. Si el usuario no activa la casilla, se muestra el mensaje de validación.

## <a name="next-steps"></a>Pasos siguientes

* Para ver una introducción sobre la creación de definiciones de interfaz de usuario, consulte [Introducción a CreateUiDefinition](create-uidefinition-overview.md).
* Para ver una descripción de las propiedades comunes de los elementos de interfaz de usuario, consulte [Elementos CreateUiDefinition](create-uidefinition-elements.md).
