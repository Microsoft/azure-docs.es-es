---
title: Elemento de la interfaz de usuario UserNameTextBox
description: Describe el elemento de la interfaz de usuario Microsoft.Common.UserNameTextBox para Azure Portal. Permite a los usuarios proporcionar nombres de usuario de Windows o Linux.
author: tfitzmac
ms.topic: conceptual
ms.date: 06/27/2018
ms.author: tomfitz
ms.openlocfilehash: 82478f322e1df22bde50769b90f0424140920e9a
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87063574"
---
# <a name="microsoftcomputeusernametextbox-ui-element"></a>Elemento de interfaz de usuario Microsoft.Compute.UserNameTextBox

Control de cuadro de texto con validación integrada para nombres de usuario de Windows y Linux.

## <a name="ui-sample"></a>Ejemplo de interfaz de usuario

![Microsoft.Compute.UserNameTextBox](./media/managed-application-elements/microsoft-compute-usernametextbox.png)

## <a name="schema"></a>Schema

```json
{
  "name": "element1",
  "type": "Microsoft.Compute.UserNameTextBox",
  "label": "User name",
  "defaultValue": "",
  "toolTip": "",
  "constraints": {
    "required": true,
    "regex": "^[a-z0-9A-Z]{1,30}$",
    "validationMessage": "Only alphanumeric characters are allowed, and the value must be 1-30 characters long."
  },
  "osPlatform": "Windows",
  "visible": true
}
```

## <a name="sample-output"></a>Salida de ejemplo

```json
"Example name"
```

## <a name="remarks"></a>Observaciones

- Si `constraints.required` está establecido en **true**, el cuadro de texto debe contener un valor para que la validación sea correcta. El valor predeterminado es **true**.
- `osPlatform` debe especificarse y puede ser **Windows** o **Linux**.
- `constraints.regex` es un patrón de expresión regular de JavaScript. Si se especifica, el valor del cuadro de texto debe coincidir con el patrón para que la validación sea correcta. El valor predeterminado es **null**.
- `constraints.validationMessage` es una cadena que se muestra cuando el valor del cuadro de texto produce un error en la validación especificada por `constraints.regex`. Si no se especifica, se utilizan los mensajes de validación integrados del cuadro de texto. El valor predeterminado es **null**.
- Este elemento tiene validación integrada que se basa en el valor especificado para `osPlatform`. La validación integrada puede usarse junto con una expresión regular personalizada. Si se especifica un valor para `constraints.regex`, se activan las validaciones integradas y personalizadas.

## <a name="next-steps"></a>Pasos siguientes

* Para ver una introducción sobre la creación de definiciones de interfaz de usuario, consulte [Introducción a CreateUiDefinition](create-uidefinition-overview.md).
* Para ver una descripción de las propiedades comunes de los elementos de interfaz de usuario, consulte [Elementos CreateUiDefinition](create-uidefinition-elements.md).
