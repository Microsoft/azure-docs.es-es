---
title: 'Patrón: propiedades field en una definición de directiva'
description: Este patrón de Azure Policy proporciona un ejemplo de cómo usar propiedades field en una definición de directiva.
ms.date: 06/29/2020
ms.topic: sample
ms.openlocfilehash: 87c550e5fb3ef0efbf219efc738a69071d2e89ba
ms.sourcegitcommit: 62717591c3ab871365a783b7221851758f4ec9a4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/22/2020
ms.locfileid: "85565722"
---
# <a name="azure-policy-pattern-field-properties"></a>Patrón de Azure Policy: propiedades field

El operador [field](../concepts/definition-structure.md#fields) evalúa la propiedad especificada o [alias](../concepts/definition-structure.md#aliases) con un valor proporcionado para una [condición](../concepts/definition-structure.md#conditions) determinada.

## <a name="sample-policy-definition"></a>Definición de directiva de ejemplo

Esta definición de directiva le permite definir regiones permitidas que cumplan los requisitos de ubicación geográfica de la organización. Los recursos permitidos se definen en el parámetro **listOfAllowedLocations** (_matriz_). Los recursos que coinciden con la definición se [deniegan](../concepts/effects.md#deny).

:::code language="json" source="~/policy-templates/patterns/pattern-fields.json":::

### <a name="explanation"></a>Explicación

:::code language="json" source="~/policy-templates/patterns/pattern-fields.json" range="18-36" highlight="3,7,11":::

El operador **field** se utiliza tres veces dentro del [operador lógico](../concepts/definition-structure.md#logical-operators) **allOf**.

- El primer uso evalúa la propiedad `location` con la condición **notIn** con el parámetro **listOfAllowedLocations**. **notIn** funciona como si esperase una _matriz_ y el parámetro es una _matriz_. Si el valor de `location` del recurso creado o actualizado no está en la lista de aprobados, este elemento se evalúa como verdadero.
- El segundo uso también evalúa la propiedad `location`, pero utiliza la condición **notEquals** para ver si el recurso es _global_. Si el valor de `location` del recurso creado o actualizado no es _global_, este elemento se evalúa como verdadero.
- El último uso evalúa la propiedad `type` y usa la condición **notEquals** para validar que el tipo de recurso no es _Microsoft.AzureActiveDirectory/b2cDirectories_. Si no lo es, este elemento se evalúa como verdadero.

Si las tres instrucciones de condición del operador lógico **allOf** se evalúan como verdaderas, Azure Policy bloquea la creación o actualización del recurso.

## <a name="next-steps"></a>Pasos siguientes

- Consulte otros [patrones y definiciones integradas](./index.md).
- Revise la [estructura de definición de Azure Policy](../concepts/definition-structure.md).
- Vea la [Descripción de los efectos de directivas](../concepts/effects.md).