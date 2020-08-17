---
title: Elemento de la interfaz de usuario de TextBox
description: Describe el elemento de la interfaz de usuario Microsoft.Common.TextBox para Azure Portal. Se usa para agregar texto sin formato.
author: tfitzmac
ms.topic: conceptual
ms.date: 06/27/2018
ms.author: tomfitz
ms.openlocfilehash: c89bc434d9d67144a95b5c2f23e7664078fe7825
ms.sourcegitcommit: 5f7b75e32222fe20ac68a053d141a0adbd16b347
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87474315"
---
# <a name="microsoftcommontextbox-ui-element"></a>Elemento de interfaz de usuario Microsoft.Common.TextBox

Control que puede usarse para editar texto sin formato.

## <a name="ui-sample"></a>Ejemplo de interfaz de usuario

![Microsoft.Common.TextBox](./media/managed-application-elements/microsoft-common-textbox.png)

## <a name="schema"></a>Schema

```json
{
    "name": "nameInstance",
    "type": "Microsoft.Common.TextBox",
    "label": "Name",
    "defaultValue": "contoso123",
    "toolTip": "Use only allowed characters",
    "placeholder": "",
    "constraints": {
        "required": true,
        "validations": [
            {
                "regex": "^[a-z0-9A-Z]{1,30}$",
                "message": "Only alphanumeric characters are allowed, and the value must be 1-30 characters long."
            },
            {
                "isValid": "[startsWith(steps('resourceConfig').nameInstance, 'contoso')]",
                "message": "Must start with 'contoso'."
            }
        ]
    },
    "visible": true
}
```

## <a name="sample-output"></a>Salida de ejemplo

```json
"contoso123"
```

## <a name="remarks"></a>Observaciones

- Si `constraints.required` está establecido en **true**, el cuadro de texto debe contener un valor para que la validación sea correcta. El valor predeterminado es **false**.
- La propiedad `validations` es una matriz en la que se agregan condiciones para comprobar el valor proporcionado en el cuadro de texto.
- La propiedad `regex` es un patrón de expresión regular de JavaScript. Si se especifica, el valor del cuadro de texto debe coincidir con el patrón para que la validación sea correcta. El valor predeterminado es **null**.
- La propiedad `isValid` contiene una expresión que se evalúa como verdadera o falsa. Dentro de la expresión, se define la condición que determina si el cuadro de texto es válido.
- La propiedad `message` es una cadena que se muestra cuando el valor del cuadro de texto no supera la validación.
- Es posible especificar un valor para `regex` cuando `required` está establecido en **false**. En este escenario, no se requiere un valor para que la validación del cuadro de texto sea correcta. Si se especifica uno, debe coincidir con el patrón de expresión regular.
- La propiedad `placeholder` es el texto de ayuda que desaparece cuando el usuario comienza a editar. Si se definen tanto `placeholder` como `defaultValue`, `defaultValue` tiene prioridad y se muestra.

## <a name="example"></a>Ejemplo

En el ejemplo siguiente se usa un cuadro de texto con el control [Microsoft.Solutions.ArmApiControl](microsoft-solutions-armapicontrol.md) para comprobar la disponibilidad de un nombre de recurso.

```json
"basics": [
    {
        "name": "nameApi",
        "type": "Microsoft.Solutions.ArmApiControl",
        "request": {
            "method": "POST",
            "path": "[concat(subscription().id, '/providers/Microsoft.Storage/checkNameAvailability?api-version=2019-06-01')]",
            "body": "[parse(concat('{\"name\": \"', basics('txtStorageName'), '\", \"type\": \"Microsoft.Storage/storageAccounts\"}'))]"
        }
    },
    {
        "name": "txtStorageName",
        "type": "Microsoft.Common.TextBox",
        "label": "Storage account name",
        "constraints": {
            "validations": [
                {
                    "isValid": "[not(equals(basics('nameApi').nameAvailable, false))]",
                    "message": "[concat('Name unavailable: ', basics('txtStorageName'))]"
                }
            ]
        }
    }
]
```

## <a name="next-steps"></a>Pasos siguientes

* Para ver una introducción sobre la creación de definiciones de interfaz de usuario, consulte [Introducción a CreateUiDefinition](create-uidefinition-overview.md).
* Para ver una descripción de las propiedades comunes de los elementos de interfaz de usuario, consulte [Elementos CreateUiDefinition](create-uidefinition-elements.md).
