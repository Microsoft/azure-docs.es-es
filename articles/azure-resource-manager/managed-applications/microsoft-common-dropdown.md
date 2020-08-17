---
title: Elemento DropDown de la interfaz de usuario
description: Describe el elemento de la interfaz de usuario Microsoft.Common.DropDown para Azure Portal. Se usa para seleccionar entre las opciones disponibles al implementar una aplicación administrada.
author: tfitzmac
ms.topic: conceptual
ms.date: 07/14/2020
ms.author: tomfitz
ms.openlocfilehash: cc50e49d4bc59235a147d114d86ecdff95dca797
ms.sourcegitcommit: 5f7b75e32222fe20ac68a053d141a0adbd16b347
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87474349"
---
# <a name="microsoftcommondropdown-ui-element"></a>Elemento de interfaz de usuario Microsoft.Common.DropDown

Control de selección con una lista desplegable. Puede permitir la selección de un solo elemento o de varios. De forma opcional, también puede incluir una descripción con los elementos.

## <a name="ui-sample"></a>Ejemplo de interfaz de usuario

El elemento DropDown tiene distintas opciones que determinan su apariencia en el portal.

Cuando solo se permite la selección de un elemento, el control aparece como:

:::image type="content" source="./media/managed-application-elements/microsoft-common-dropdown-1.png" alt-text="Selección única de Microsoft.Common.DropDown":::

Cuando se incluyen descripciones, el control aparece como:

:::image type="content" source="./media/managed-application-elements/microsoft-common-dropdown-2.png" alt-text="Selección única de Microsoft.Common.DropDown con descripciones":::

Cuando la selección múltiple está habilitada, el control agrega una opción **Seleccionar todo** y casillas para activar más de un elemento:

:::image type="content" source="./media/managed-application-elements/microsoft-common-dropdown-3.png" alt-text="Selección múltiple de Microsoft.Common.DropDown":::

Se pueden incluir descripciones con la selección múltiple habilitada.

:::image type="content" source="./media/managed-application-elements/microsoft-common-dropdown-4.png" alt-text="Selección múltiple de Microsoft.Common.DropDown con descripciones":::

Cuando el filtrado está habilitado, el control incluye un cuadro de texto para agregar el valor de filtrado.

:::image type="content" source="./media/managed-application-elements/microsoft-common-dropdown-5.png" alt-text="Selección múltiple de Microsoft.Common.DropDown con descripciones":::

## <a name="schema"></a>Schema

```json
{
    "name": "element1",
    "type": "Microsoft.Common.DropDown",
    "label": "Example drop down",
    "placeholder": "",
    "defaultValue": "Value two",
    "toolTip": "",
    "multiselect": true,  
    "selectAll": true,  
    "filter": true,  
    "filterPlaceholder": "Filter items ...",  
    "multiLine": true,  
    "defaultDescription": "A value for selection",  
    "constraints": {
        "allowedValues": [
            {
                "label": "Value one",
                "description": "The value to select for option 1.",
                "value": "one"
            },
            {
                "label": "Value two",
                "description": "The value to select for option 2.",
                "value": "two"
            }
        ],
        "required": true
    },
    "visible": true
}
```

## <a name="sample-output"></a>Salida de ejemplo

```json
"two"
```

## <a name="remarks"></a>Observaciones

- Use `multiselect` para especificar si los usuarios pueden seleccionar más de un elemento.
- De forma predeterminada, `selectAll` es `true` cuando la selección múltiple está habilitada.
- La propiedad `filter` permite a los usuarios buscar en una larga lista de opciones.
- La etiqueta de `constraints.allowedValues` es el texto para mostrar de un elemento, y su valor es el valor de salida del elemento cuando se selecciona.
- Si se especifica, el valor predeterminado debe ser una etiqueta presente en `constraints.allowedValues`. Si no se especifica, se selecciona el primer elemento de `constraints.allowedValues`. El valor predeterminado es **null**.
- `constraints.allowedValues` debe tener al menos un elemento.
- Para emular un valor que no es obligatorio, agregue un elemento con una etiqueta y el valor `""` (cadena vacía) a `constraints.allowedValues`.
- La propiedad `defaultDescription` se usa para los elementos que no tienen una descripción.
- La propiedad `placeholder` es el texto de ayuda que desaparece cuando el usuario comienza a editar. Si se definen tanto `placeholder` como `defaultValue`, `defaultValue` tiene prioridad y se muestra.

## <a name="next-steps"></a>Pasos siguientes

* Para ver una introducción sobre la creación de definiciones de interfaz de usuario, consulte [Introducción a CreateUiDefinition](create-uidefinition-overview.md).
* Para ver una descripción de las propiedades comunes de los elementos de interfaz de usuario, consulte [Elementos CreateUiDefinition](create-uidefinition-elements.md).
