---
title: Elemento de la interfaz de usuario de CredentialsCombo
description: Describe el elemento de la interfaz de usuario Microsoft.Compute.CredentialsCombo para Azure Portal.
author: tfitzmac
ms.topic: conceptual
ms.date: 09/29/2018
ms.author: tomfitz
ms.openlocfilehash: 47c88e08e5d2eac09fbcd5b60a8ccd73b46c9616
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87063801"
---
# <a name="microsoftcomputecredentialscombo-ui-element"></a>Elemento de interfaz de usuario Microsoft.Compute.CredentialsCombo

Grupo de controles con validación integrada para las contraseñas de Windows y Linux, y las claves públicas de SSH.

## <a name="ui-sample"></a>Ejemplo de interfaz de usuario

Los usuarios de Windows ven:

![Microsoft.Compute.CredentialsCombo Windows](./media/managed-application-elements/microsoft-compute-credentialscombo-windows.png)

Los usuarios de Linux con contraseña seleccionada ven:

![Microsoft.Compute.CredentialsCombo contraseña Linux](./media/managed-application-elements/microsoft-compute-credentialscombo-linux-password.png)

Los usuarios de Linux con clave pública SSH seleccionada ven:

![Microsoft.Compute.CredentialsCombo clave Linux](./media/managed-application-elements/microsoft-compute-credentialscombo-linux-key.png)

## <a name="schema"></a>Schema

Para Windows, use el esquema siguiente:

```json
{
  "name": "element1",
  "type": "Microsoft.Compute.CredentialsCombo",
  "label": {
    "password": "Password",
    "confirmPassword": "Confirm password"
  },
  "toolTip": {
    "password": ""
  },
  "constraints": {
    "required": true,
    "customPasswordRegex": "^(?=.*[A-Za-z])(?=.*\\d)[A-Za-z\\d]{12,}$",
    "customValidationMessage": "The password must be alphanumeric, contain at least 12 characters, and have at least 1 letter and 1 number."
  },
  "options": {
    "hideConfirmation": false
  },
  "osPlatform": "Windows",
  "visible": true
}
```

Para **Linux**, use el esquema siguiente:

```json
{
  "name": "element1",
  "type": "Microsoft.Compute.CredentialsCombo",
  "label": {
    "authenticationType": "Authentication type",
    "password": "Password",
    "confirmPassword": "Confirm password",
    "sshPublicKey": "SSH public key"
  },
  "toolTip": {
    "authenticationType": "",
    "password": "",
    "sshPublicKey": ""
  },
  "constraints": {
    "required": true,
    "customPasswordRegex": "^(?=.*[A-Za-z])(?=.*\\d)[A-Za-z\\d]{12,}$",
    "customValidationMessage": "The password must be alphanumeric, contain at least 12 characters, and have at least 1 letter and 1 number."
  },
  "options": {
    "hideConfirmation": false,
    "hidePassword": false
  },
  "osPlatform": "Linux",
  "visible": true
}
```

## <a name="sample-output"></a>Salida de ejemplo

Si `osPlatform` es **Windows** o `osPlatform` es **Linux** y el usuario proporcionó una contraseña en lugar de una clave pública SSH, el control devuelve la siguiente salida:

```json
{
  "authenticationType": "password",
  "password": "p4ssw0rddem0",
}
```

Si `osPlatform` es **Linux** y el usuario proporcionó una clave pública SSH, el control devuelve la siguiente salida:

```json
{
  "authenticationType": "sshPublicKey",
  "sshPublicKey": "AAAAB3NzaC1yc2EAAAABIwAAAIEA1on8gxCGJJWSRT4uOrR13mUaUk0hRf4RzxSZ1zRbYYFw8pfGesIFoEuVth4HKyF8k1y4mRUnYHP1XNMNMJl1JcEArC2asV8sHf6zSPVffozZ5TT4SfsUu/iKy9lUcCfXzwre4WWZSXXcPff+EHtWshahu3WzBdnGxm5Xoi89zcE=",
}
```

## <a name="remarks"></a>Observaciones

- `osPlatform` debe especificarse y puede ser **Windows** o **Linux**.
- Si `constraints.required` está establecido en **true**, los cuadros de texto de clave pública SSH o contraseña deben contener valores para que la validación sea correcta. El valor predeterminado es **true**.
- Si `options.hideConfirmation` está establecido en **true**, se oculta el segundo cuadro de texto para confirmar la contraseña del usuario. El valor predeterminado es **false**.
- Si `options.hidePassword` está establecido en **true**, se oculta la opción para utilizar la autenticación de contraseña. Se puede utilizar solo cuando `osPlatform` es **Linux**. El valor predeterminado es **false**.
- Se pueden implementar restricciones adicionales en las contraseñas permitidas con la propiedad `customPasswordRegex`. La cadena de `customValidationMessage` se muestra cuando se produce un error de validación personalizada en una contraseña. El valor predeterminado para ambas propiedades es **null**.

## <a name="next-steps"></a>Pasos siguientes

* Para ver una introducción sobre la creación de definiciones de interfaz de usuario, consulte [Introducción a CreateUiDefinition](create-uidefinition-overview.md).
* Para ver una descripción de las propiedades comunes de los elementos de interfaz de usuario, consulte [Elementos CreateUiDefinition](create-uidefinition-elements.md).
