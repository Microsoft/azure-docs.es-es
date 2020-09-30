---
title: Descripción del ámbito en Azure Policy
description: Se describe el concepto de ámbito en Azure Resource Manager y cómo se aplica a Azure Policy para controlar qué recursos evalúa este servicio.
ms.date: 09/22/2020
ms.topic: conceptual
ms.openlocfilehash: 83eda330a35b007abfa37046a202b14728726849
ms.sourcegitcommit: bdd5c76457b0f0504f4f679a316b959dcfabf1ef
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90984431"
---
# <a name="understand-scope-in-azure-policy"></a>Descripción del ámbito en Azure Policy

Hay una serie de valores que determinan qué recursos se pueden evaluar y qué recursos se evalúan mediante Azure Policy. El concepto principal de estos controles es el _ámbito_.
Para obtener información general, consulte [Ámbito en Azure Resource Manager](../../../azure-resource-manager/management/overview.md#understand-scope).
En este artículo se explica el impacto del _ámbito_ en Azure Policy, así como sus objetos y propiedades relacionados.

## <a name="definition-location"></a>Ubicación de definición

Azure Policy usa el primer ámbito de instancia cuando se crea una definición de directiva. La definición se puede guardar en un grupo de administración o una suscripción. La ubicación determina el ámbito al que pueden asignarse la directiva o la iniciativa. Los recursos deben estar dentro de la jerarquía de recursos de la ubicación de la definición para que puedan ser objetivo de la asignación.

Si la ubicación de la definición es:

- **Suscripción**: la definición de la directiva solo se puede asignar a los recursos incluidos dentro de esa suscripción.
- **Grupo de administración**: la definición de la directiva solo se puede asignar a los recursos incluidos dentro de grupos de administración secundarios y suscripciones secundarias. Si planea aplicar la definición de directiva a varias suscripciones, la ubicación debe ser un grupo de administración que contenga esas suscripciones.

La ubicación debe ser el contenedor de recursos compartido por todos los recursos en los que desee usar la definición de directiva. Este contenedor de recursos suele ser un grupo de administración próximo al grupo de administración raíz.

## <a name="assignment-scopes"></a>Ámbitos de asignación

Una asignación tiene varias propiedades que establecen un ámbito. El uso de estas propiedades determina qué recursos de Azure Policy se evalúan y cuáles cuentan para el cumplimiento. Estas propiedades se corresponden con los conceptos siguientes:

- Inclusión: la definición debe evaluar el cumplimiento de una jerarquía de recursos o de un recurso individual. La propiedad `properties.scope` de un objeto de asignación determina qué se va a incluir y evaluar en términos de cumplimiento. Para obtener más información, consulte [Definición de asignación](./assignment-structure.md).

- Exclusión: la definición no debe evaluar el cumplimiento de una jerarquía de recursos o de un recurso individual. La propiedad `properties.notScopes` de la _matriz_ de un objeto de asignación determina qué se excluye. Los recursos incluidos en esos ámbitos no se evalúan ni se incluyen en el recuento de cumplimiento. Para obtener más información, consulte [Definición de asignación: ámbitos excluidos](./assignment-structure.md#excluded-scopes).

Además de las propiedades de la asignación de directiva, es el objeto de [exención de directiva](./exemption-structure.md). Las exenciones mejoran la historia del ámbito proporcionando un método para identificar una parte de una asignación que no se va a evaluar.

- Exención (**gratis en la característica en versión preliminar**): la definición debe evaluar el cumplimiento de una jerarquía de recursos o de un recurso individual, pero no lo hará si hay motivos como tener una exención o ser mitigado por otro método. Los recursos con este estado se muestran como **Exentos** en los informes de cumplimiento, para que se pueda realizar el seguimiento. El objeto de exención se crea en la jerarquía de recursos o en el recurso individual como un objeto secundario, que determina el ámbito de la exención. Una jerarquía de recursos o un recurso individual pueden estar exentos de varias asignaciones. La exención puede configurarse para que expire según una programación mediante la propiedad `expiresOn`. Para obtener más información, consulte [Definición de exención](./exemption-structure.md).

  > [!NOTE]
  > Debido al impacto que supone conceder una exención para una jerarquía de recursos o un recurso individual, las exenciones tienen medidas de seguridad adicionales. Además de exigir la operación `Microsoft.Authorization/policyExemptions/write` en la jerarquía de recursos o en el recurso individual, el creador de una exención debe tener el verbo `exempt/Action` en la asignación de destino.

## <a name="scope-comparison"></a>Comparación de ámbitos

En la tabla siguiente se presenta una comparación de las opciones de ámbito:

| | Inclusión | Exclusión (notScopes) | Exención |
|---|:---:|:---:|:---:|
|**Los recursos se evalúan** | &#10004; | - | - |
|**Objeto de Resource Manager** | - | - | &#10004; |
|**Requiere la modificación del objeto de asignación de directiva** | &#10004; | &#10004; | - |

## <a name="next-steps"></a>Pasos siguientes

- Más información sobre la [estructura de la definición de directiva](./definition-structure.md).
- Obtenga información acerca de cómo se pueden [crear directivas mediante programación](../how-to/programmatically-create.md).
- Obtenga información sobre cómo [obtener datos de cumplimiento](../how-to/get-compliance-data.md).
- Obtenga información sobre cómo [corregir recursos no compatibles](../how-to/remediate-resources.md).
- En [Organización de los recursos con grupos de administración de Azure](../../management-groups/overview.md), obtendrá información sobre lo que es un grupo de administración.