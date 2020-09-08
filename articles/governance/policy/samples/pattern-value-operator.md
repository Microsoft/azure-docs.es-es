---
title: 'Patrón: operador value en una definición de directiva'
description: Este patrón de Azure Policy proporciona un ejemplo de cómo usar el operador value en una definición de directiva.
ms.date: 06/29/2020
ms.topic: sample
ms.openlocfilehash: e246e3a5e2517fa80626081227070bcb2f967784
ms.sourcegitcommit: 62717591c3ab871365a783b7221851758f4ec9a4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/22/2020
ms.locfileid: "85565672"
---
# <a name="azure-policy-pattern-the-value-operator"></a>Patrón de Azure Policy: operador value

El operador [value](../concepts/definition-structure.md#value) evalúa [parámetros](../concepts/definition-structure.md#parameters), [funciones de plantilla admitidas](../concepts/definition-structure.md#policy-functions) o literales con un valor proporcionado para una [condición](../concepts/definition-structure.md#conditions) determinada.

> [!WARNING]
> Si el resultado de una _función de plantilla_ es un error, no se pude realizar la evaluación de directivas. Una evaluación con errores es una **denegación** implícita. Para más información, consulte cómo [evitar los errores de plantilla](../concepts/definition-structure.md#avoiding-template-failures).

## <a name="sample-policy-definition"></a>Definición de directiva de ejemplo

Esta definición de directiva agrega o reemplaza la etiqueta especificada en el parámetro **tagName** (_cadena_) en los recursos y hereda el valor de **tagName** del grupo de recursos en el que se encuentra el recurso. Esta evaluación se produce cuando se crea o actualiza el recurso. Como un efecto [modify](../concepts/effects.md#modify), la corrección se puede ejecutar en los recursos existentes mediante una [tarea de corrección](../how-to/remediate-resources.md).

:::code language="json" source="~/policy-templates/patterns/pattern-value-operator.json":::

### <a name="explanation"></a>Explicación

:::code language="json" source="~/policy-templates/patterns/pattern-value-operator.json" range="20-30" highlight="7,8":::

El operador **value** se usa dentro del bloque **policyRule.if** dentro de **properties**. En este ejemplo, el [operador lógico](../concepts/definition-structure.md#logical-operators) **allOf** se usa para comprobar que sean verdaderas ambas instrucciones condicionales para que el efecto **modify** tenga lugar.

**value** evalúa el resultado de la función de plantilla [resourceGroup()](../../../azure-resource-manager/templates/template-functions-resource.md#resourcegroup) con la condición **notEquals** de un valor en blanco. Si el nombre de etiqueta proporcionado en **tagName** existe en el grupo de recursos primario, la instrucción condicional se evalúa en verdadero.

## <a name="next-steps"></a>Pasos siguientes

- Consulte otros [patrones y definiciones integradas](./index.md).
- Revise la [estructura de definición de Azure Policy](../concepts/definition-structure.md).
- Vea la [Descripción de los efectos de directivas](../concepts/effects.md).