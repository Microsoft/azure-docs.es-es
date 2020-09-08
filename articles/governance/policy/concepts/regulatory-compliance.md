---
title: Cumplimiento normativo en las definiciones de iniciativa
description: Describe cómo usar una definición de iniciativa para agrupar directivas por dominio normativo, como Access Control, Administración de configuración, etc.
ms.date: 08/17/2020
ms.topic: conceptual
ms.openlocfilehash: 296febc1aad62d06c8e9dfcf4f285b16cacf0a40
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88544628"
---
# <a name="regulatory-compliance-in-azure-policy"></a>Cumplimiento normativo de Azure Policy

Cumplimiento normativo de Azure Policy proporciona definiciones de iniciativas integradas para ver una lista de los **controles** y los **dominios de cumplimiento** en función de la responsabilidad (_Cliente_, _Microsoft_, _Compartido_).
En el caso de los controles responsabilidad de Microsoft, se proporcionan detalles adicionales de los resultados de auditoría basados en la atestación de terceros y en nuestros detalles de implementación para lograr ese cumplimiento.
La propiedad `type` de los controles responsabilidad de Microsoft es [estático](./definition-structure.md#type).

> [!NOTE]
> Cumplimiento normativo es una característica en versión preliminar. En el caso de los elementos integrados actualizados, los controles de iniciativas se asignan al estándar de cumplimiento correspondiente. Las iniciativas estándar de cumplimiento existentes están en proceso de actualización para admitir la característica Cumplimiento normativo.

## <a name="regulatory-compliance-defined"></a>Definición de Cumplimiento normativo

Cumplimiento normativo se basa en la parte [agrupación](./initiative-definition-structure.md#policy-definition-groups) de una definición de iniciativa. En los elementos integrados, cada agrupación de la definición de iniciativa define un nombre (**control**) y una categoría (**dominio de cumplimiento**), y proporciona una referencia al objeto [policyMetadata](./initiative-definition-structure.md#metadata-objects) que contiene información sobre ese **control**. Una definición de iniciativa de Cumplimiento normativo debe tener la propiedad `category` establecida en **Cumplimiento normativo**. Como otras definiciones de iniciativa estándar, las iniciativas de Cumplimiento normativo admiten [parámetros](./initiative-definition-structure.md#parameters) para crear asignaciones dinámicas.

Los clientes pueden crear sus propias iniciativas de Cumplimiento normativo. Estas definiciones pueden ser originales o copiadas de definiciones integradas existentes. Si usa una definición de iniciativa de Cumplimiento normativo integrada como referencia, se recomienda supervisar el origen de las definiciones de Cumplimiento normativo en el [repositorio de GitHub de Azure Policy](https://github.com/Azure/azure-policy/tree/master/built-in-policies/policySetDefinitions/Regulatory%20Compliance).

Para vincular una iniciativa de Cumplimiento normativo personalizada a su panel de Azure Security Center, consulte [Azure Security Center: uso de directivas de seguridad personalizadas](../../../security-center/custom-security-policies.md).

## <a name="regulatory-compliance-in-portal"></a>Cumplimiento normativo en el portal

Cuando se ha creado una definición de iniciativa con [grupos](./initiative-definition-structure.md#policy-definition-groups), la página de detalles **Cumplimiento** del portal para esa iniciativa presenta información adicional. 

Una nueva pestaña, **Controles**, se ha agregado a la página. El filtrado está disponible a través del **dominio de cumplimiento** y las definiciones de directiva se agrupan por el campo `title` del objeto **policyMetadata**. Cada fila representa un **control** que muestra su estado de cumplimiento, el **dominio de cumplimiento** del que forma parte, la información de responsabilidad, y el número de definiciones de directivas conformes y no conformes que componen ese **control**.

:::image type="content" source="../media/regulatory-compliance/regulatory-compliance-overview.png" alt-text="Ejemplo de información general de Cumplimiento normativo para la definición integrada de NIST SP 800-53 R4.":::

Al seleccionar un **control** se abre una página de detalles sobre ese control. **Información general** contiene la información de `description` y `requirements`. En la pestaña **Directivas** se encuentran todas las definiciones de directivas individuales de la iniciativa que contribuyen a este **control**. La pestaña **Cumplimiento normativo** proporciona una vista detallada de cada recurso evaluado por una directiva de miembro del **control** que se muestra actualmente.

> [!NOTE]
> Un tipo de evaluación **Administrado por Microsoft** está destinado a la propiedad `type` de la definición de directiva establecida en [estático](./definition-structure.md#type) .

:::image type="content" source="../media/regulatory-compliance/regulatory-compliance-policies.png" alt-text="Ejemplo de las definiciones de directiva de Cumplimiento normativo en el control de protección de límites del dominio de protección del sistema y las comunicaciones de la definición integrada de NIST SP 800-53 R4.":::

En la misma página de **control**, al cambiar a la pestaña **Compatibilidad de recursos** se muestran todos los recursos que se incluyen en las definiciones de directiva de este **control**. Los filtros están disponibles para el nombre o el identificador, el estado de cumplimiento, el tipo de recurso y la ubicación.

:::image type="content" source="../media/regulatory-compliance/regulatory-compliance-resources.png" alt-text="Ejemplo de los recursos de Cumplimiento normativo que se ven afectados por las definiciones de directiva en el control de protección de límites del dominio de protección del sistema y las comunicaciones de la definición integrada de NIST SP 800-53 R4.":::

## <a name="regulatory-compliance-in-sdk"></a>Cumplimiento normativo en el SDK

Si se habilita el Cumplimiento normativo en una definición de iniciativa, la evaluación examina que los SDK de registros, eventos y estados de directiva devuelvan propiedades adicionales. Estas propiedades adicionales se agrupan por estado de cumplimiento y proporcionan información sobre el número de grupos en cada estado.

El siguiente código es un ejemplo de resultados agregados desde una llamada a `summarize`:

```json
"policyGroupDetails": [{
        "complianceState": "noncompliant",
        "count": 4
    },
    {
        "complianceState": "compliant",
        "count": 76
    }
]
```

## <a name="next-steps"></a>Pasos siguientes

- Consulte la [estructura de definición de la iniciativa](./initiative-definition-structure.md).
- Puede consultar ejemplos en [Ejemplos de Azure Policy](../samples/index.md).
- Vea la [Descripción de los efectos de directivas](./effects.md).
- Obtenga información sobre cómo [corregir recursos no compatibles](../how-to/remediate-resources.md).
