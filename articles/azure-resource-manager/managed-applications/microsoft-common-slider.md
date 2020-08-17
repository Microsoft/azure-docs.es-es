---
title: Elemento de la interfaz de usuario Slider
description: Describe el elemento de la interfaz de usuario Microsoft.Common.Slider para Azure Portal. Permite a los usuarios establecer un valor a partir de un intervalo de opciones.
author: tfitzmac
ms.topic: conceptual
ms.date: 07/10/2020
ms.author: tomfitz
ms.openlocfilehash: bb168a4bff6d2570f539307530232b5063bb535c
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87095273"
---
# <a name="microsoftcommonslider-ui-element"></a>Elemento de la interfaz de usuario Microsoft.Common.Slider

El control Slider permite a los usuarios seleccionar en un intervalo de valores permitidos.

## <a name="ui-sample"></a>Ejemplo de interfaz de usuario

:::image type="content" source="./media/managed-application-elements/microsoft-common-slider.png" alt-text="Microsoft.Common.Slider":::

## <a name="schema"></a>Schema

```json
{
    "name": "memorySize",
    "type": "Microsoft.Common.Slider",
    "min": 1,
    "max": 64,
    "label": "Memory",
    "subLabel": "MB",
    "defaultValue": 24,
    "showStepMarkers": false,
    "toolTip": "Pick the size in MB",
    "constraints": {
        "required": false
    },
    "visible": true
}
```

## <a name="sample-output"></a>Salida de ejemplo

```json
26
```

## <a name="remarks"></a>Observaciones

- Los valores `min` y `max` son obligatorios. Establecen los puntos inicial y final para el control deslizante.
- El valor predeterminado de la propiedad `showStepMarkers` es true. Los marcadores de paso solo se muestran cuando el intervalo de mínimo a máximo es 100 o menos.


## <a name="next-steps"></a>Pasos siguientes

* Para ver una introducción sobre la creación de definiciones de interfaz de usuario, consulte [Introducción a CreateUiDefinition](create-uidefinition-overview.md).
* Para ver una descripción de las propiedades comunes de los elementos de interfaz de usuario, consulte [Elementos CreateUiDefinition](create-uidefinition-elements.md).
