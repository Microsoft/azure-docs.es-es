---
title: 'Patrón: operadores lógicos en una definición de directiva'
description: Este patrón de Azure Policy proporciona un ejemplo de cómo usar los operadores lógicos en una definición de directiva.
ms.date: 08/17/2020
ms.topic: sample
ms.openlocfilehash: 3f644cdbfc45b06d1ad5db8e7727c0fa69742f00
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88545597"
---
# <a name="azure-policy-pattern-logical-operators"></a>Patrón de Azure Policy: operadores lógicos

Una definición de directiva puede contener varias instrucciones condicionales. Es posible que necesite que cada instrucción sea verdadera o que solo necesite que algunas de ellas sean verdaderas. Para satisfacer estas necesidades, el lenguaje tiene [operadores lógicos](../concepts/definition-structure.md#logical-operators) para **not**, **allOf** y **anyOf**. Son opcionales y se pueden anidar para crear escenarios complejos.

## <a name="sample-1-one-logical-operator"></a>Muestra 1: un operador lógico

Esta definición de directiva evalúa las cuentas de CosmosDB para ver si están configuradas las conmutaciones por error automáticas y varias ubicaciones de escritura. Cuando no lo están, se desencadena el efecto [audit](../concepts/effects.md#audit) y crea una entrada de registro cuando se crea o actualiza el recurso no compatible.

:::code language="json" source="~/policy-templates/patterns/pattern-logical-operators-1.json":::

### <a name="sample-1-explanation"></a>Muestra 1: Explicación

:::code language="json" source="~/policy-templates/patterns/pattern-logical-operators-1.json" range="6-22" highlight="3":::

El bloque **policyRule.if** usa un solo operador **allOf** para asegurarse de que las tres condiciones son verdaderas.
Solo cuando todas estas condiciones se evalúan como verdaderas, se desencadena el efecto **audit**.

## <a name="sample-2-multiple-logical-operators"></a>Ejemplo 2: varios operadores lógicos

Esta definición de directiva evalúa los recursos según un patrón de nomenclatura. Si un recurso no coincide, es [denegado](../concepts/effects.md#deny).

:::code language="json" source="~/policy-templates/patterns/pattern-logical-operators-2.json":::

### <a name="sample-2-explanation"></a>Ejemplo 2: Explicación

:::code language="json" source="~/policy-templates/patterns/pattern-logical-operators-2.json" range="7-21" highlight="2,3,9":::

Este bloque **policyRule.if** también incluye un único operador **allOf**, pero cada condición se encapsula con el operador lógico **not**. El condicional dentro del operador lógico **not** se evalúa primero y, a continuación, se evalúa el operador **not** para determinar si la cláusula completa es verdadera o falsa. Si ambos operadores lógicos **not** se evalúan como verdaderos, se desencadena el efecto de la directiva.

## <a name="sample-3-combining-logical-operators"></a>Ejemplo 3: Combinación de operadores lógicos

Esta definición de directiva evalúa las cuentas de Java Spring para ver si el seguimiento no está habilitado o si el seguimiento no se encuentra en un estado correcto.

:::code language="json" source="~/policy-templates/patterns/pattern-logical-operators-3.json":::

### <a name="sample-3-explanation"></a>Ejemplo 3: Explicación

:::code language="json" source="~/policy-templates/patterns/pattern-logical-operators-3.json" range="6-28" highlight="3,8":::

Este bloque **policyRule.if** incluye los operadores lógicos **allOf** y **anyOf**. El operador lógico **anyOf** se evalúa como True siempre que se cumpla una condición incluida. Dado que _type_ está en el núcleo de **allOf**, siempre se evalúa como True. Si _type_ y una de las condiciones de **anyOf** son True, se desencadena el efecto de la directiva.

## <a name="next-steps"></a>Pasos siguientes

- Consulte otros [patrones y definiciones integradas](./index.md).
- Revise la [estructura de definición de Azure Policy](../concepts/definition-structure.md).
- Vea la [Descripción de los efectos de directivas](../concepts/effects.md).