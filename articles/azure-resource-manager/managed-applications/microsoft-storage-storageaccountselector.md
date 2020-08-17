---
title: Elemento de la interfaz de usuario de StorageAccountSelector
description: Describe el elemento de la interfaz de usuario Microsoft.Storage.StorageAccountSelector para Azure Portal.
author: tfitzmac
ms.topic: conceptual
ms.date: 06/28/2018
ms.author: tomfitz
ms.openlocfilehash: fe70c42387e9dedea8943b5dcce04a1f9ded5190
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87033310"
---
# <a name="microsoftstoragestorageaccountselector-ui-element"></a>Elemento de interfaz de usuario Microsoft.Storage.StorageAccountSelector

Un control para seleccionar una cuenta de almacenamiento nueva o existente.

## <a name="ui-sample"></a>Ejemplo de interfaz de usuario

El control muestra el valor predeterminado.

![Microsoft.Storage.StorageAccountSelector](./media/managed-application-elements/microsoft-storage-storageaccountselector.png)

El control permite al usuario crear una cuenta de almacenamiento o seleccionar una cuenta de almacenamiento existente.

![Microsoft.Storage.StorageAccountSelector nueva](./media/managed-application-elements/microsoft-storage-storageaccountselector-new.png)

## <a name="schema"></a>Schema

```json
{
  "name": "element1",
  "type": "Microsoft.Storage.StorageAccountSelector",
  "label": "Storage account",
  "toolTip": "",
  "defaultValue": {
    "name": "storageaccount01",
    "type": "Premium_LRS"
  },
  "constraints": {
    "allowedTypes": [],
    "excludedTypes": []
  },
  "options": {
    "hideExisting": false
  },
  "visible": true
}
```

## <a name="sample-output"></a>Salida de ejemplo

```json
{
  "name": "storageaccount01",
  "resourceGroup": "rg01",
  "type": "Premium_LRS",
  "newOrExisting": "new"
}
```

## <a name="remarks"></a>Observaciones

- Si se especifica, se valida automáticamente la unicidad de `defaultValue.name`. Si el nombre de la cuenta de almacenamiento no es único, el usuario debe especificar otro nombre o elegir una cuenta de almacenamiento existente.
- El valor predeterminado de `defaultValue.type` es **Premium_LRS**.
- Los tipos no especificados en `constraints.allowedTypes` está oculto, mientras que los tipos no especificado en `constraints.excludedTypes` se muestran. Tanto `constraints.allowedTypes` como `constraints.excludedTypes` son opcionales, pero no se pueden usar simultáneamente.
- Si el valor de `options.hideExisting` es **true**, el usuario no puede elegir una cuenta de almacenamiento existente. El valor predeterminado es **false**.

## <a name="next-steps"></a>Pasos siguientes
* Para ver una introducción sobre la creación de definiciones de interfaz de usuario, consulte [Introducción a CreateUiDefinition](create-uidefinition-overview.md).
* Para ver una descripción de las propiedades comunes de los elementos de interfaz de usuario, consulte [Elementos CreateUiDefinition](create-uidefinition-elements.md).
