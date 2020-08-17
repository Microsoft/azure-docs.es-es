---
title: Elemento de la interfaz de usuario InfoBox
description: Describe el elemento de la interfaz de usuario Microsoft.Common.InfoB para Azure Portal. Se usa para agregar texto o advertencias al implementar una aplicación administrada.
author: tfitzmac
ms.topic: conceptual
ms.date: 06/15/2018
ms.author: tomfitz
ms.openlocfilehash: 7580ac0650706d6aee49bbf0e8235e8c5dab33f4
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87033364"
---
# <a name="microsoftcommoninfobox-ui-element"></a>Elemento de interfaz de usuario Microsoft.Common.InfoBox

Un control que agrega un cuadro de información. El cuadro contiene texto importante o advertencias que explican a los usuarios los valores que se están proporcionando. También puede vincular a un URI para ampliar la información.

## <a name="ui-sample"></a>Ejemplo de interfaz de usuario

![Microsoft.Common.InfoBox](./media/managed-application-elements/microsoft-common-infobox.png)


## <a name="schema"></a>Schema

```json
{
  "name": "text1",
  "type": "Microsoft.Common.InfoBox",
  "visible": true,
  "options": {
    "icon": "None",
    "text": "Nullam eros mi, mollis in sollicitudin non, tincidunt sed enim. Sed et felis metus, rhoncus ornare nibh. Ut at magna leo.",
    "uri": "https://www.microsoft.com"
  }
}
```

## <a name="sample-output"></a>Salida de ejemplo

```json
"Nullam eros mi, mollis in sollicitudin non, tincidunt sed enim. Sed et felis metus, rhoncus ornare nibh. Ut at magna leo."
```

## <a name="remarks"></a>Observaciones

* Para `icon`, use **Ninguno**, **Información**, **Advertencia** o **Error**.
* La propiedad `uri` es opcional.

## <a name="next-steps"></a>Pasos siguientes

* Para ver una introducción sobre la creación de definiciones de interfaz de usuario, consulte [Introducción a CreateUiDefinition](create-uidefinition-overview.md).
* Para ver una descripción de las propiedades comunes de los elementos de interfaz de usuario, consulte [Elementos CreateUiDefinition](create-uidefinition-elements.md).
