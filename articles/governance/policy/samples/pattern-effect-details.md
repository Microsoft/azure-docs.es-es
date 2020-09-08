---
title: 'Patrón: efectos de una definición de directiva'
description: Este patrón de Azure Policy proporciona un ejemplo de cómo usar los distintos efectos de una definición de directiva.
ms.date: 06/29/2020
ms.topic: sample
ms.openlocfilehash: 9c7e7689aa837a2dba0992112df3cd36ac9affe6
ms.sourcegitcommit: 62717591c3ab871365a783b7221851758f4ec9a4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/22/2020
ms.locfileid: "85565741"
---
# <a name="azure-policy-pattern-effects"></a>Patrón de Azure Policy: efectos

Azure Policy tiene varios [efectos](../concepts/effects.md) que determinan cómo reacciona el servicio ante los recursos no compatibles. Algunos efectos son simples y no requieren ninguna propiedad adicional en la definición de directiva, mientras que otros requieren varias propiedades.

## <a name="sample-1-simple-effect"></a>Muestra 1: Efecto simple

Esta definición de directiva comprueba si la etiqueta definida en el parámetro **tagName** existe en el recurso evaluado. Si la etiqueta aún no existe, se desencadena el efecto [modify](../concepts/effects.md#modify) para agregar la etiqueta con el valor en el parámetro **tagValue**.

:::code language="json" source="~/policy-templates/patterns/pattern-effect-details-1.json":::

### <a name="sample-1-explanation"></a>Muestra 1: Explicación

:::code language="json" source="~/policy-templates/patterns/pattern-effect-details-1.json" range="40-50":::

El efecto **modify** requiere el bloque **policyRule.then.details** que define **roleDefinitionIds** y **operations**. Estos parámetros informan a Azure Policy qué roles son necesarios para agregar la etiqueta y corregir el recurso y qué operación **modify** se va a realizar. En este ejemplo, el valor de **operation** es _add_ y los parámetros se usan para establecer la etiqueta y su valor.

## <a name="sample-2-complex-effect"></a>Ejemplo 2: Efecto complejo

Esta definición de directiva audita cada máquina virtual para comprobar si una extensión, definida en los parámetros **publisher** y **type**, no existe. Usa [auditIfNotExists](../concepts/effects.md#auditifnotexists) para comprobar un recurso relacionado con la máquina virtual para ver si existe una instancia que coincida con los parámetros definidos. En este ejemplo se comprueba el tipo **extensions**.

:::code language="json" source="~/policy-templates/patterns/pattern-effect-details-2.json":::

### <a name="sample-2-explanation"></a>Ejemplo 2: Explicación

:::code language="json" source="~/policy-templates/patterns/pattern-effect-details-2.json" range="45-58":::

El efecto **auditIfNotExists** requiere el bloque **policyRule.then.details** para definir los valores de **type** y **existenceCondition** que se van a buscar. **existenceCondition** utiliza elementos de lenguaje de directivas, como los [operadores lógicos](../concepts/definition-structure.md#logical-operators), para determinar si existe un recurso relacionado coincidente. En este ejemplo, los valores comprobados para cada [alias](../concepts/definition-structure.md#aliases) se definen en los parámetros.

## <a name="next-steps"></a>Pasos siguientes

- Consulte otros [patrones y definiciones integradas](./index.md).
- Revise la [estructura de definición de Azure Policy](../concepts/definition-structure.md).
- Vea la [Descripción de los efectos de directivas](../concepts/effects.md).