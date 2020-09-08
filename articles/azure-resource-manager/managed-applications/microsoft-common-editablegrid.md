---
title: Elemento de la interfaz de usuario de EditableGrid
description: Describe el elemento de la interfaz de usuario Microsoft.Common.EditableGrid para Azure Portal. Permite a los usuarios recopilar datos tabulares.
author: tfitzmac
ms.topic: conceptual
ms.date: 08/24/2020
ms.author: tomfitz
ms.openlocfilehash: 04f86883a75110985d1cbe050fe3fd3e0582986a
ms.sourcegitcommit: c6b9a46404120ae44c9f3468df14403bcd6686c1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88893650"
---
# <a name="microsoftcommoneditablegrid-ui-element"></a>Elemento de la interfaz de usuario Microsoft.Common.EditableGrid

Se trata de un control para recopilar datos tabulares. Todos los campos de la cuadrícula son editables y el número de filas puede variar.

## <a name="ui-sample"></a>Ejemplo de interfaz de usuario

:::image type="content" source="./media/managed-application-elements/microsoft-common-editablegrid.png" alt-text="Microsoft.Common.EditableGrid":::

## <a name="schema"></a>Schema

```json
{
  "name": "people",
  "type": "Microsoft.Common.EditableGrid",
  "ariaLabel": "Enter information per person",
  "label": "People",
  "constraints": {
    "width": "Full",
    "rows": {
      "count": {
        "min": 1,
        "max": 10
      }
    },
    "columns": [
      {
        "id": "colName",
        "header": "Name",
        "width": "1fr",
        "element": {
          "type": "Microsoft.Common.TextBox",
          "placeholder": "Full name",
          "constraints": {
            "required": true,
            "validations": [
              {
                "isValid": "[startsWith(last(take(steps('grid').people, $rowIndex)).colName, 'contoso')]",
                "message": "Must start with 'contoso'."
              },
              {
                "regex": "^[a-z0-9A-Z]{1,30}$",
                "message": "Only alphanumeric characters are allowed, and the value must be 1-30 characters long."
              }
            ]
          }
        }
      },
      {
        "id": "colGender",
        "header": "Gender",
        "width": "1fr",
        "element": {
          "name": "dropDown1",
          "type": "Microsoft.Common.DropDown",
          "placeholder": "Select a gender...",
          "constraints": {
            "allowedValues": [
              {
                "label": "Female",
                "value": "female"
              },
              {
                "label": "Male",
                "value": "male"
              },
              {
                "label": "Other",
                "value": "other"
              }
            ],
            "required": true
          }
        }
      },
      {
        "id": "colContactPreference",
        "header": "Contact preference",
        "width": "1fr",
        "element": {
          "type": "Microsoft.Common.OptionsGroup",
          "constraints": {
            "allowedValues": [
              {
                "label": "Email",
                "value": "email"
              },
              {
                "label": "Text",
                "value": "text"
              }
            ],
            "required": true
          }
        }
      }
    ]
  }
}
```

## <a name="sample-output"></a>Salida de ejemplo

```json
{
  "colName": "contoso",
  "colGender": "female",
  "colContactPreference": "email"
}
```

## <a name="remarks"></a>Observaciones

- Los únicos controles válidos dentro de la matriz de columnas son [TextBox](microsoft-common-textbox.md), [OptionsGroup](microsoft-common-optionsgroup.md)y [DropDown](microsoft-common-dropdown.md).
- La variable `$rowIndex` solo es válida en expresiones incluidas en elementos secundarios de las columnas de la cuadrícula. Se trata de un entero que representa el índice de fila relativa del elemento; y el recuento comienza en uno y se incrementa en uno. Tal como se muestra en la sección `"columns":` del esquema, el valor `$rowIndex` se utiliza con fines de validación.
- Cuando las validaciones se realizan mediante la variable `$rowIndex`, es posible obtener el valor de la fila actual mediante la combinación de los comandos `last()` y `take()`.

  Por ejemplo:

  `last(take(<reference_to_grid>, $rowIndex))`

- La propiedad `label` no aparece como parte del control, pero se muestra en el resumen de la pestaña final.
- La propiedad `ariaLabel` es la etiqueta de accesibilidad de la cuadrícula. Especifique texto útil para los usuarios que utilizan lectores de pantalla.
- La propiedad `constraints.width` se utiliza para establecer el ancho global de la cuadrícula. Las opciones son _Full_ (Completa), _Medium_ (Media) y _Small_ (Pequeña). El valor predeterminado es _Full_ (Completa).
- La propiedad `width` de los elementos secundarios de las columnas determina el ancho de columna. Los anchos se especifican mediante unidades fraccionarias, como _3fr_; y el espacio total se asigna a las columnas de forma proporcional con respecto a las unidades. Si no se especifica un ancho de columna, el valor predeterminado es _1fr_.

## <a name="next-steps"></a>Pasos siguientes

- Para ver una introducción sobre la creación de definiciones de interfaz de usuario, consulte [Introducción a CreateUiDefinition](create-uidefinition-overview.md).
- Para ver una descripción de las propiedades comunes de los elementos de interfaz de usuario, consulte [Elementos CreateUiDefinition](create-uidefinition-elements.md).
