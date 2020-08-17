---
title: Elemento de interfaz de usuario MultiStorageAccountCombo
description: Describe el elemento de la interfaz de usuario Microsoft.Storage.MultiStorageAccountCombo para Azure Portal.
author: tfitzmac
ms.topic: conceptual
ms.date: 06/28/2018
ms.author: tomfitz
ms.openlocfilehash: a8172b63039d2d247f30fca4099254cb8fca068e
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87073403"
---
# <a name="microsoftstoragemultistorageaccountcombo-ui-element"></a>Elemento de la interfaz de usuario Microsoft.Storage.MultiStorageAccountCombo

Un grupo de controles para crear varias cuentas de almacenamiento con nombres que comienzan con un prefijo común.

## <a name="ui-sample"></a>Ejemplo de interfaz de usuario

![Microsoft.Storage.MultiStorageAccountCombo](./media/managed-application-elements/microsoft-storage-multistorageaccountcombo.png)

## <a name="schema"></a>Schema

```json
{
  "name": "element1",
  "type": "Microsoft.Storage.MultiStorageAccountCombo",
  "label": {
    "prefix": "Storage account prefix",
    "type": "Storage account type"
  },
  "toolTip": {
    "prefix": "",
    "type": ""
  },
  "defaultValue": {
    "prefix": "sa",
    "type": "Premium_LRS"
  },
  "constraints": {
    "allowedTypes": [],
    "excludedTypes": []
  },
  "count": 2,
  "visible": true
}
```

## <a name="sample-output"></a>Salida de ejemplo

```json
{
  "prefix": "sa",
  "count": 2,
  "resourceGroup": "rg01",
  "type": "Premium_LRS"
}
```

## <a name="remarks"></a>Observaciones

- El valor de `defaultValue.prefix` se concatena con uno o varios números enteros para generar la secuencia de nombres de cuenta de almacenamiento. Por ejemplo, si `defaultValue.prefix` es **sa** y `count` es **2**, se generan los nombres de cuenta de almacenamiento **sa1** y **sa2**. La unicidad de los nombres de cuenta de almacenamiento generados se valida automáticamente.
- Los nombres de cuenta de almacenamiento se generan lexicográficamente en función de `count`. Por ejemplo, si `count` es 10, los nombres de cuenta de almacenamiento terminan en enteros de dos dígitos (01, 02, 03).
- El valor predeterminado de `defaultValue.prefix` es **null**, mientras que el de `defaultValue.type` es **Premium_LRS**.
- Los tipos no especificados en `constraints.allowedTypes` está oculto, mientras que los tipos no especificado en `constraints.excludedTypes` se muestran. Tanto `constraints.allowedTypes` como `constraints.excludedTypes` son opcionales, pero no se pueden usar simultáneamente.
- Además de para generar nombres de cuenta de almacenamiento, `count` se usa para establecer el multiplicador apropiado para el elemento. Admite un valor estático, como **2**, o un valor dinámico de otro elemento, como `[steps('step1').storageAccountCount]`. El valor predeterminado es **1**.

## <a name="next-steps"></a>Pasos siguientes

* Para ver una introducción sobre la creación de definiciones de interfaz de usuario, consulte [Introducción a CreateUiDefinition](create-uidefinition-overview.md).
* Para ver una descripción de las propiedades comunes de los elementos de interfaz de usuario, consulte [Elementos CreateUiDefinition](create-uidefinition-elements.md).
