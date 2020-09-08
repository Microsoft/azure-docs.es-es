---
title: Directivas para etiquetar recursos
description: Describe las instancias de Azure Policy que puede asignar para garantizar el cumplimiento de la etiqueta.
ms.topic: conceptual
ms.date: 03/20/2020
ms.openlocfilehash: c6867bc01306ac3c08a9797ece0567a45e060af2
ms.sourcegitcommit: 58d3b3314df4ba3cabd4d4a6016b22fa5264f05a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89293749"
---
# <a name="assign-policies-for-tag-compliance"></a>Asignación de directivas para el cumplimiento de la etiqueta

Puede usar [Azure Policy](../../governance/policy/overview.md) para aplicar reglas de etiquetado y convenciones. Mediante la creación de una directiva, evitará el escenario de recursos que se vayan a implementar en su suscripción que no cuentan con las etiquetas esperadas para su organización. En lugar de aplicar manualmente las etiquetas o buscar recursos que no son compatibles, cree una directiva que aplica automáticamente las etiquetas necesarias durante la implementación. También se pueden aplicar etiquetas a los recursos existentes con el nuevo efecto de [modificación](../../governance/policy/concepts/effects.md#modify) y una [tarea de corrección](../../governance/policy/how-to/remediate-resources.md). La sección siguiente muestra las directivas de ejemplo para las etiquetas.

## <a name="policies"></a>Directivas

[!INCLUDE [Tag policies](../../../includes/policy/reference/bycat/policies-tags.md)]

## <a name="next-steps"></a>Pasos siguientes

* Para información sobre el etiquetado de recursos, consulte [Uso de etiquetas para organizar los recursos](tag-resources.md).
* No todos los tipos de recursos admiten etiquetas. Para determinar si se puede aplicar una etiqueta a un tipo de recurso determinado, consulte [Tag support for Azure resources](tag-support.md) (Compatibilidad con etiquetas para los recursos de Azure).
