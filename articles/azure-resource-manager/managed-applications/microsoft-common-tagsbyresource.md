---
title: Elemento de la interfaz de usuario de TagsByResource
description: Describe el elemento de la interfaz de usuario Microsoft.Common.TagsByResource para Azure Portal. Use para aplicar etiquetas a un recurso durante la implementación.
author: tfitzmac
ms.topic: conceptual
ms.date: 11/11/2019
ms.author: tomfitz
ms.openlocfilehash: e730201812005a9b469a964e4acd081ebe86b100
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87063949"
---
# <a name="microsoftcommontagsbyresource-ui-element"></a>Elemento de la interfaz de usuario Microsoft.Common.TagsByResource

Un control para asociar [etiquetas](../management/tag-resources.md) a los recursos de una implementación.

## <a name="ui-sample"></a>Ejemplo de interfaz de usuario

![Microsoft.Common.DropDown](./media/managed-application-elements/microsoft-common-tagsbyresource.png)

## <a name="schema"></a>Schema

```json
{
  "name": "element1",
  "type": "Microsoft.Common.TagsByResource",
  "resources": [
    "Microsoft.Storage/storageAccounts",
    "Microsoft.Compute/virtualMachines"
  ]
}
```

## <a name="sample-output"></a>Salida de ejemplo

```json
{
  "Microsoft.Storage/storageAccounts": {
    "Dept": "Finance",
    "Environment": "Production"
  },
  "Microsoft.Compute/virtualMachines": {
    "Dept": "Finance"
  }
}
```

## <a name="remarks"></a>Observaciones

- Como mínimo, se debe especificar un elemento de la matriz `resources`.
- Cada elemento de `resources` debe ser un tipo de recurso completo. Estos elementos aparecen en la lista desplegable **Recurso** y el usuario los puede etiquetar.
- A la salida del control se le ha dado un formato para facilitar la asignación de valores de etiqueta en una plantilla de Azure Resource Manager. Para recibir la salida del control en una plantilla, incluya un parámetro en ella tal como se muestra en el ejemplo siguiente:

  ```json
  "parameters": {
    "tagsByResource": { "type": "object", "defaultValue": {} }
  }
  ```

  Para cada recurso que se pueda etiquetar, asigne la propiedad "tags" al valor del parámetro para ese tipo de recurso:

  ```json
  {
    "name": "saName1",
    "type": "Microsoft.Storage/storageAccounts",
    "tags": "[ if(contains(parameters('tagsByResource'), 'Microsoft.Storage/storageAccounts'), parameters('tagsByResource')['Microsoft.Storage/storageAccounts'], json('{}')) ]",
    ...
  ```

- Utilice la función [if](../templates/template-functions-logical.md#if) cuando acceda al parámetro tagsByResource. Le permite asignar un objeto vacío cuando no hay ninguna etiqueta asignada al tipo de recurso especificado.

## <a name="next-steps"></a>Pasos siguientes

- Para ver una introducción sobre la creación de definiciones de interfaz de usuario, consulte [Introducción a CreateUiDefinition](create-uidefinition-overview.md).
- Para ver una descripción de las propiedades comunes de los elementos de interfaz de usuario, consulte [Elementos CreateUiDefinition](create-uidefinition-elements.md).
