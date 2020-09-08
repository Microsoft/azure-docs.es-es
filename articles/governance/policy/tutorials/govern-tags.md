---
title: 'Tutorial: Administración de gobernanza de etiquetas'
description: En este tutorial, usará el efecto Modify de Azure Policy para crear y aplicar un modelo de gobernanza de etiquetas en los recursos nuevos y existentes.
ms.date: 08/17/2020
ms.topic: tutorial
ms.openlocfilehash: f49eedb00c98d3c362140fdca9b195a086903f10
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88545512"
---
# <a name="tutorial-manage-tag-governance-with-azure-policy"></a>Tutorial: Administración de la gobernanza de etiquetas con Azure Policy

Las [etiquetas](../../../azure-resource-manager/management/tag-resources.md) son una parte fundamental de la organización de los recursos de Azure en una taxonomía. Cuando se siguen [procedimientos recomendados para la administración de etiquetas](/azure/cloud-adoption-framework/ready/azure-best-practices/naming-and-tagging#naming-and-tagging-resources), las etiquetas pueden ser la base para aplicar las directivas empresariales con Azure Policy o para realizar el [seguimiento de los costos con Cost Management](../../../cost-management-billing/costs/cost-mgt-best-practices.md#tag-shared-resources).
Independientemente de cómo o por qué use las etiquetas, es importante que pueda agregar, cambiar y quitar rápidamente estas etiquetas en los recursos de Azure. Para ver si el recurso de Azure admite el etiquetado, consulte [Compatibilidad de etiqueta](../../../azure-resource-manager/management/tag-support.md).

El efecto [Modify](../concepts/effects.md#modify) de Azure Policy está diseñado para ayudar en la gobernanza de las etiquetas, sin importar la fase de gobernanza de los recursos en la que se encuentre. **Modify** sirve de ayuda en las siguientes circunstancias:

- No está familiarizado con la nube y no cuenta con gobernanza de etiquetas.
- Ya tiene miles de recursos sin gobernanza de etiquetas.
- Ya tiene una taxonomía existente que necesita cambiar.

En este tutorial, va a completar las siguientes tareas:

> [!div class="checklist"]
> - Identificar los requisitos empresariales
> - Asignar cada requisito a una definición de directiva
> - Agrupar las directivas de etiqueta en una iniciativa

## <a name="prerequisites"></a>Prerrequisitos

Para completar este tutorial, necesitará una suscripción de Azure. Si no tiene una, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="identify-requirements"></a>Identificación de los requisitos

Al igual que cualquier implementación adecuada de controles de gobernanza, los requisitos deben proceder de las necesidades empresariales y comprenderse bien antes de crear los controles técnicos. En el escenario de este tutorial, los siguientes elementos constituyen los requisitos empresariales:

- Hay dos etiquetas necesarias en todos los recursos: _CostCenter_ y _Env_.
- _CostCenter_ debe existir en todos los contenedores y en los recursos individuales.
  - Los recursos heredan del contenedor en el que se encuentran, pero pueden invalidarse de forma individual.
- _Env_ debe existir en todos los contenedores y en los recursos individuales.
  - Los recursos determinan el entorno por el esquema de nomenclatura del contenedor y no se pueden invalidar.
  - Todos los recursos de un contenedor forman parte del mismo entorno.

## <a name="configure-the-costcenter-tag"></a>Configuración de la etiqueta CostCenter

En lo que respecta específicamente a un entorno de Azure administrado por Azure Policy, los requisitos de la etiqueta _CostCenter_ exigen los siguientes resultados:

- La denegación de grupos de recursos en los que falte la etiqueta _CostCenter_.
- La modificación de los recursos para agregar la etiqueta _CostCenter_ del grupo de recursos primario cuando falte.

### <a name="deny-resource-groups-missing-the-costcenter-tag"></a>Denegación de los grupos de recursos en los que falte la etiqueta CostCenter

Como el valor de _CostCenter_ de un grupo de recursos no se puede determinar a partir del nombre de este grupo, debe tener la etiqueta definida en la solicitud para crear el grupo de recursos. La siguiente regla de directiva con el efecto [Deny](../concepts/effects.md#deny) impide la creación o actualización de grupos de recursos que no tengan la etiqueta _CostCenter_:

```json
"if": {
    "allOf": [{
            "field": "type",
            "equals": "Microsoft.Resources/subscriptions/resourceGroups"
        },
        {
            "field": "tags['CostCenter']",
            "exists": false
        }
    ]
},
"then": {
    "effect": "deny"
}
```

> [!NOTE]
> Dado que esta regla de directiva tiene como destino un grupo de recursos, el valor de _mode_ en la definición de la directiva debe ser "all" en lugar de "indexed".

### <a name="modify-resources-to-inherit-the-costcenter-tag-when-missing"></a>Modificación de los recursos para que hereden la etiqueta CostCenter cuando falte

La segunda necesidad de _CostCenter_ es que los recursos hereden la etiqueta del grupo de recursos primario cuando falte. Si la etiqueta ya está definida en el recurso, aunque sea diferente del grupo de recursos primario, debe dejarse tal cual. La siguiente regla de directiva usa [Modify](../concepts/effects.md#modify):

```json
"policyRule": {
    "if": {
        "field": "tags['CostCenter']",
        "exists": "false"
    },
    "then": {
        "effect": "modify",
        "details": {
            "roleDefinitionIds": [
                "/providers/microsoft.authorization/roleDefinitions/b24988ac-6180-42a0-ab88-20f7382dd24c"
            ],
            "operations": [{
                "operation": "add",
                "field": "tags['CostCenter']",
                "value": "[resourcegroup().tags['CostCenter']]"
            }]
        }
    }
}
```

Esta regla de directiva usa la operación **add** en lugar de **addOrReplace** ya que no se desea modificar el valor de la etiqueta si está presente al [corregir](../how-to/remediate-resources.md) recursos existentes. También utiliza la función de plantilla `[resourcegroup()]` para obtener el valor de la etiqueta del grupo de recursos primario.

> [!NOTE]
> Dado que esta regla de directiva está dirigida a recursos que admiten etiquetas, el valor de _mode_ en la definición de la directiva debe ser "indexed". Esta configuración también garantiza que esta directiva omite los grupos de recursos.

## <a name="configure-the-env-tag"></a>Configuración de la etiqueta Env

En lo que respecta específicamente a un entorno de Azure administrado por Azure Policy, los requisitos de la etiqueta _Env_ exigen los siguientes resultados:

- La modificación de la etiqueta _Env_ en el grupo de recursos en función del esquema de nomenclatura del grupo de recursos.
- La modificación de la etiqueta _Env_ en todos los recursos del grupo de recursos al mismo valor que el grupo de recursos primario.

### <a name="modify-resource-groups-env-tag-based-on-name"></a>La modificación de la etiqueta Env de los grupos de recursos según el nombre.

Se requiere una directiva [Modify](../concepts/effects.md#modify) para cada entorno que exista en el entorno de Azure. La directiva Modify de cada uno tiene un aspecto similar al de esta definición de directiva:

```json
"policyRule": {
    "if": {
        "allOf": [{
            "field": "type",
            "equals": "Microsoft.Resources/subscriptions/resourceGroups"
        },
        {
            "field": "name",
            "like": "prd-*"
        }
    ]
    },
    "then": {
        "effect": "modify",
        "details": {
            "roleDefinitionIds": [
                "/providers/microsoft.authorization/roleDefinitions/b24988ac-6180-42a0-ab88-20f7382dd24c"
            ],
            "operations": [{
                "operation": "addOrReplace",
                "field": "tags['Env']",
                "value": "Production"
            }]
        }
    }
}
```

> [!NOTE]
> Dado que esta regla de directiva tiene como destino un grupo de recursos, el valor de _mode_ en la definición de la directiva debe ser "all" en lugar de "indexed".

Esta directiva solo coincide con los grupos de recursos que cuentan con el esquema de nomenclatura de ejemplo que se usa para los recursos de producción de `prd-`. Se puede lograr un esquema de nomenclatura más complejo con varias condiciones de tipo **match** en lugar de la única condición **like** de este ejemplo.

### <a name="modify-resources-to-inherit-the-env-tag"></a>Modificación de los recursos para que hereden la etiqueta Env

El requisito empresarial exige que todos los recursos tengan la etiqueta _Env_ que tenga su grupo de recursos primario. Esta etiqueta no se puede invalidar, por lo que usaremos la operación **addOrReplace** con el efecto [Modify](../concepts/effects.md#modify). La directiva Modify de ejemplo es similar a la siguiente regla:

```json
"policyRule": {
    "if": {
        "anyOf": [{
            "field": "tags['Env']",
            "notEquals": "[resourcegroup().tags['Env']]"
        },
        {
            "field": "tags['Env']",
            "exists": false
        }
    ]
    },
    "then": {
        "effect": "modify",
        "details": {
            "roleDefinitionIds": [
                "/providers/microsoft.authorization/roleDefinitions/b24988ac-6180-42a0-ab88-20f7382dd24c"
            ],
            "operations": [{
                "operation": "addOrReplace",
                "field": "tags['Env']",
                "value": "[resourcegroup().tags['Env']]"
            }]
        }
    }
}
```

> [!NOTE]
> Dado que esta regla de directiva está dirigida a recursos que admiten etiquetas, el valor de _mode_ en la definición de la directiva debe ser "indexed". Esta configuración también garantiza que esta directiva omite los grupos de recursos.

Esta regla de directiva busca cualquier recurso que no tenga el valor de su grupo de recursos primario en la etiqueta _Env_ o en el que falte la etiqueta _Env_. En los recursos coincidentes, la etiqueta _Env_ se establece en el valor del grupo de recursos primario, incluso si la etiqueta ya existía en el recurso pero con un valor diferente.

## <a name="assign-the-initiative-and-remediate-resources"></a>Asignación de la iniciativa y corrección de los recursos

Una vez creadas las directivas de etiqueta anteriores, se unen en una única iniciativa para la gobernanza de las etiquetas y se asignan a un grupo de administración o una suscripción. Después, la iniciativa y las directivas incluidas evalúan el cumplimiento de los recursos existentes y modifican las solicitudes de los recursos nuevos o actualizados que coinciden con la propiedad **if** de la regla de la directiva. Sin embargo, la directiva no actualiza automáticamente los recursos no conformes existentes con los cambios de etiqueta definidos.

Al igual que las directivas [deployIfNotExists](../concepts/effects.md#deployifnotexists), la directiva **Modify** utiliza tareas de corrección para modificar los recursos no conformes existentes. Siga las instrucciones que se indican en [Corrección de los recursos](../how-to/remediate-resources.md) para identificar los recursos de **Modify** no conformes y corregir las etiquetas a la taxonomía definida.

## <a name="clean-up-resources"></a>Limpieza de recursos

Si terminó de trabajar con los recursos de este tutorial, use los pasos siguientes para eliminar todas las asignaciones y definiciones creadas anteriormente:

1. Seleccione **Definiciones** (o **Asignaciones** si trata de eliminar una asignación) en **Creación** en el lado izquierdo de la página de Azure Policy.

1. Busque la nueva definición de iniciativa o directiva (o asignación) que quiere quitar.

1. Haga clic con el botón derecho en la fila o seleccione los puntos suspensivos al final de la definición (o asignación) y elija **Eliminar definición** (o **Eliminar asignación** ).

## <a name="review"></a>Revisar

En este tutorial ha obtenido información sobre las siguientes tareas:

> [!div class="checklist"]
> - Ha identificado los requisitos empresariales
> - Ha asignado cada requisito a una definición de directiva
> - Ha agrupado las directivas de etiqueta en una iniciativa

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre las estructuras de las definiciones de directiva, consulte este artículo:

> [!div class="nextstepaction"]
> [Estructura de definición de Azure Policy](../concepts/definition-structure.md)