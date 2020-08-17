---
title: Elemento de la interfaz de usuario de Sección
description: Describe el elemento de la interfaz de usuario Microsoft.Common.Section para Azure Portal. Se usa para agrupar elementos en el portal para implementar aplicaciones administradas.
author: tfitzmac
ms.topic: conceptual
ms.date: 06/27/2018
ms.author: tomfitz
ms.openlocfilehash: 924aff8f2ba3d796b65f52494845f3b10018065c
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87063967"
---
# <a name="microsoftcommonsection-ui-element"></a>Elemento de interfaz de usuario Microsoft.Common.Section

Control que agrupa uno o varios elementos en un encabezado.

## <a name="ui-sample"></a>Ejemplo de interfaz de usuario

![Microsoft.Common.Section](./media/managed-application-elements/microsoft-common-section.png)

## <a name="schema"></a>Schema

```json
{
  "name": "section1",
  "type": "Microsoft.Common.Section",
  "label": "Example section",
  "elements": [
    {
      "name": "text1",
      "type": "Microsoft.Common.TextBox",
      "label": "Example text box 1"
    },
    {
      "name": "text2",
      "type": "Microsoft.Common.TextBox",
      "label": "Example text box 2"
    }
  ],
  "visible": true
}
```

## <a name="remarks"></a>Observaciones

- `elements` debe tener al menos un elemento y puede tener todos los tipos de elementos, excepto `Microsoft.Common.Section`.
- Este elemento no admite la propiedad `toolTip`.

## <a name="sample-output"></a>Salida de ejemplo
Para obtener acceso a los valores de salida de los elementos en `elements`, use las funciones [basics()](create-ui-definition-referencing-functions.md#basics) o [steps()](create-ui-definition-referencing-functions.md#steps) y la notación de puntos:

```json
steps('configuration').section1.text1
```

Los elementos del tipo `Microsoft.Common.Section` no tienen valores de salida por sí mismos.

## <a name="next-steps"></a>Pasos siguientes

* Para ver una introducción sobre la creación de definiciones de interfaz de usuario, consulte [Introducción a CreateUiDefinition](create-uidefinition-overview.md).
* Para ver una descripción de las propiedades comunes de los elementos de interfaz de usuario, consulte [Elementos CreateUiDefinition](create-uidefinition-elements.md).
