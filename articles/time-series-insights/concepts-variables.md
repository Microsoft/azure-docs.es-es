---
title: 'Variables de modelo: Azure Time Series Insights Gen2 | Microsoft Docs'
description: Variables de modelo
author: shreyasharmamsft
ms.author: shresha
ms.service: time-series-insights
ms.topic: conceptual
ms.date: 08/12/2020
ms.openlocfilehash: d0a5b48e93e839b0a0adaf185700d7f60fec7948
ms.sourcegitcommit: a2a7746c858eec0f7e93b50a1758a6278504977e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/12/2020
ms.locfileid: "88140690"
---
# <a name="time-series-model-variables"></a>Variables del modelo de serie temporal

En este artículo se describen las variables del modelo de serie temporal que especifican la fórmula y las reglas de cálculo en los eventos.

Cada variable puede ser de tres tipos: *numérico*, de *categoría* y de *agregado*.

* Los tipos **numéricos** funcionan con valores numéricos continuos.
* Los tipos de **categoría** funcionan con un conjunto definido de valores discretos.
* Los tipos de **agregado** combinan varias variables de un único tipo (todos numéricos o todos de categoría).

En la tabla siguiente se muestran las propiedades que son relevantes para cada tipo de variable.

[![Ejemplo de variable del modelo de serie temporal](media/v2-update-tsm/time-series-model-variable-table.png)](media/v2-update-tsm/time-series-model-variable-table.png#lightbox)

## <a name="numeric-variables"></a>Variables numéricas

| Propiedad de variable | Descripción |
| --- | ---|
| Filtro de variables | Los filtros son cláusulas opcionales condicionales para restringir el número de filas que se tienen en cuenta para el cálculo. |
| Valor variable | Valores de telemetría usados para el cálculo procedentes del dispositivo o los sensores, o transformados mediante expresiones de serie temporal. Las variables de tipo numérico deben ser del tipo *Double*.|
| Interpolación de variable | La interpolación especifica cómo reconstruir una señal mediante los datos existentes. Para las variables numéricas están disponibles las opciones de interpolación *Escalonada* y *Lineal*. |
| Agregación de la variable | Realice cálculos a través de las [funciones de agregación admitidas para los tipos de variables numéricas](https://docs.microsoft.com/rest/api/time-series-insights/reference-time-series-expression-syntax#numeric-variable-kind). |

Las variables se ajustan al ejemplo JSON siguiente:

```JSON
"Interpolated Speed": {
  "kind": "numeric",
  "value": {
    "tsx": "$event['Speed-Sensor'].Double"
  },
  "filter": null,
  "interpolation": {
    "kind": "step",
    "boundary": {
      "span": "P1D"
    }
  },
  "aggregation": {
    "tsx": "right($value)"
  }
}
```

## <a name="categorical-variables"></a>Variables de categoría

| Propiedad de variable | Descripción |
| --- | ---|
| Filtro de variables | Los filtros son cláusulas opcionales condicionales para restringir el número de filas que se tienen en cuenta para el cálculo. |
| Valor variable | Valores de telemetría que se usan para el cálculo procedentes del dispositivo o de los sensores. Las variables de tipo de categoría deben ser *Long* o *String*. |
| Interpolación de variable | La interpolación especifica cómo reconstruir una señal mediante los datos existentes. Para las variables de categoría está disponible la opción de interpolación *Escalonada*. |
| Categorías de variable | Las categorías crean una asignación entre los valores procedentes del dispositivo o los sensores a una etiqueta. |
| Categoría predeterminada de variable | La categoría predeterminada es para todos los valores que no se están asignando en la propiedad "categories". |

Las variables se ajustan al ejemplo JSON siguiente:

```JSON
"Status": {
  "kind": "categorical",
  "value": {
     "tsx": "$event.Status.Long"
},
  "interpolation": {
    "kind": "step",
    "boundary": {
      "span" : "PT1M"
    }
  },
  "categories": [
    {
      "values": [0, 1, 2, 3],
      "label": "Good"
    },
    {
      "values": [4],
      "label": "Bad"
    }
  ],
  "defaultCategory": {
    "label": "Not Applicable"
  }
}
```

## <a name="aggregate-variables"></a>Variables de agregado

| Propiedad de variable | Descripción |
| --- | ---|
| Filtro de variables | Los filtros son cláusulas opcionales condicionales para restringir el número de filas que se tienen en cuenta para el cálculo. |
| Agregación de la variable | Realice cálculos a través de las [funciones de agregación admitidas para los tipos de variables de agregado](https://docs.microsoft.com/rest/api/time-series-insights/reference-time-series-expression-syntax#aggregate-variable-kind). |

Las variables se ajustan al ejemplo JSON siguiente:

```JSON
"Speed Range": {
  "kind": "aggregate",
  "filter": null,
  "aggregation": {
    "tsx": "max($event.Speed.Double) - min($event.Speed.Double)"
  }
}
```

Las variables se almacenan en la definición de tipo de un modelo de serie temporal y se pueden proporcionar insertadas a través de las API para reemplazar o complementar la definición almacenada.

## <a name="next-steps"></a>Pasos siguientes

* Obtenga más información sobre el [modelo de serie temporal](./concepts-model-overview.md).

* Obtenga más información sobre cómo definir variables insertadas mediante las [API de consulta](./concepts-query-overview.md).
