---
title: 'Patrón: uso de etiquetas en una definición de directiva'
description: En este patrón de Azure Policy se proporcionan ejemplos de cómo agregar etiquetas con parámetros o heredar etiquetas de un grupo de recursos en una definición de directiva.
ms.date: 08/17/2020
ms.topic: sample
ms.openlocfilehash: 3016fc7889f68fd13e993c67ca645a4af055c651
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88545563"
---
# <a name="azure-policy-pattern-tags"></a>Patrón de Azure Policy: etiquetas

Las [etiquetas](../../..//azure-resource-manager/management/tag-resources.md) son una parte importante de la administración, organización y control de los recursos de Azure. Azure Policy permite configurar etiquetas en los recursos nuevos y existentes a escala con el efecto [modificar](../concepts/effects.md#modify) y las [tareas de corrección](../how-to/remediate-resources.md).

## <a name="sample-1-parameterize-tags"></a>Muestra 1: Parametrizar

Esta definición de directiva usa dos parámetros, **tagName** y **tagValue**, para establecer lo que busca la asignación de directiva en los grupos de recursos. Este formato permite usar la definición de directiva para cualquier número de combinaciones de nombre de etiqueta y valor de etiqueta, pero solo se mantiene una única definición de directiva.

> [!NOTE]
> Aunque este patrón de definición de directiva es similar al de [Patrón: Parámetros: ejemplo n.º 1](./pattern-parameters.md#sample-1-string-parameters), en este ejemplo se usa el **modo** _Todos_ y se destina a los grupos de recursos.

:::code language="json" source="~/policy-templates/patterns/pattern-tags-1.json":::

### <a name="sample-1-explanation"></a>Muestra 1: Explicación

:::code language="json" source="~/policy-templates/patterns/pattern-tags-1.json" range="2-8" highlight="3":::

En este ejemplo, el **modo** se establece en _Todos_, ya que tiene como destino un grupo de recursos. En la mayoría de los casos, el **modo** debe establecerse en _Indizado_ al trabajar con etiquetas. Para más información, consulte los [modos](../concepts/definition-structure.md#resource-manager-modes).

:::code language="json" source="~/policy-templates/patterns/pattern-tags-1.json" range="26-36" highlight="7-8":::

En esta parte de la definición de la directiva, `concat` combina el parámetro **tagName** con parámetros y el formato `tags['name']` para indicar al **campo** que evalúe la etiqueta del parámetro **tagValue**.
Como se usa **notEquals**, si la **etiqueta\[tagName\]** no es igual que **tagValue**, se desencadena el efecto **modificar**.

:::code language="json" source="~/policy-templates/patterns/pattern-tags-1.json" range="43-47" highlight="3-4":::

En este caso, la operación **addOrReplace** utiliza el mismo formato para usar los valores de etiqueta con parámetros a fin de crear o actualizar la etiqueta con el valor deseado en el grupo de recursos evaluado.

## <a name="sample-2-inherit-tag-value-from-resource-group"></a>Ejemplo 2: Herencia de valor de una etiqueta del grupo de recursos

Esta definición de directiva usa el parámetro **tagName** para determinar el valor de la etiqueta que se va a heredar del grupo de recursos primario.

:::code language="json" source="~/policy-templates/patterns/pattern-tags-2.json":::

### <a name="sample-2-explanation"></a>Ejemplo 2: Explicación

:::code language="json" source="~/policy-templates/patterns/pattern-tags-2.json" range="2-8" highlight="3":::

En este ejemplo, el **modo** se establece en _Indizado_, ya que no tiene como destino un grupo de recursos ni una suscripción a pesar de que obtiene el valor de un grupo de recursos. Para más información, consulte los [modos](../concepts/definition-structure.md#resource-manager-modes).

:::code language="json" source="~/policy-templates/patterns/pattern-tags-2.json" range="19-29" highlight="3-4,7-8":::

**policyRule.if** usa `concat` como [Ejemplo n.º 1](#sample-1-parameterize-tags) para evaluar el valor de **tagName**, pero utiliza la función `resourceGroup()` para compararlo con el valor de la misma etiqueta en el grupo de recursos primario. La segunda cláusula comprueba que la etiqueta del grupo de recursos tenga un valor y no sea NULL.

:::code language="json" source="~/policy-templates/patterns/pattern-tags-2.json" range="36-40" highlight="3-4":::

Aquí, el valor que se asigna a la etiqueta **tagName** en el recurso también utiliza la función `resourceGroup()` para obtener el valor del grupo de recursos primario. De esta manera, puede heredar etiquetas de grupos de recursos primarios. Si ya ha creado el recurso pero no ha agregado la etiqueta, esta misma definición de directiva y una [tarea de corrección](../how-to/remediate-resources.md) pueden actualizar los recursos existentes.

## <a name="next-steps"></a>Pasos siguientes

- Consulte otros [patrones y definiciones integradas](./index.md).
- Revise la [estructura de definición de Azure Policy](../concepts/definition-structure.md).
- Vea la [Descripción de los efectos de directivas](../concepts/effects.md).