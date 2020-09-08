---
title: Orden de la secuencia de implementación
description: Obtenga más detalles sobre el orden predeterminado en el que se implementan los artefactos de plano técnico durante una asignación de plano técnico y aprenda a personalizar el orden de implementación.
ms.date: 08/27/2020
ms.topic: conceptual
ms.openlocfilehash: 8305e5d44caef0f35e5b4beb4b70be9736272fa7
ms.sourcegitcommit: 8a7b82de18d8cba5c2cec078bc921da783a4710e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89051481"
---
# <a name="understand-the-deployment-sequence-in-azure-blueprints"></a>Información sobre la secuencia de implementación en Azure Blueprint

Azure Blueprints usa un **orden de secuenciación** para determinar el orden de creación de recursos al procesar la asignación de una definición de plano técnico. En este artículo se explican los conceptos siguientes:

- El orden de secuenciación predeterminado que se usa
- Cómo personalizar el orden
- Cómo se procesa el orden personalizado

Hay variables en los ejemplos JSON que se deben reemplazar por los suyos propios:

- `{YourMG}`: reemplácelo por el nombre del grupo de administración

## <a name="default-sequencing-order"></a>Orden de secuenciación predeterminado

Si la definición de plano técnico no contiene ninguna directiva para el orden en que se implementarán los artefactos o la directiva es null, se utiliza el orden siguiente:

- Artefactos de **asignación de roles** de nivel de suscripción ordenados por nombre de artefacto
- Artefactos de **asignación de directiva** de nivel de suscripción ordenados por nombre de artefacto
- Artefactos de **plantilla de Azure Resource Manager** (plantillas de ARM) de nivel de suscripción ordenados por nombre de artefacto
- Artefactos de **grupo de recursos** (incluidos los secundarios) ordenados por nombre de marcador de posición

Dentro de cada artefacto de **grupo de recursos**, se usa el orden de secuencia siguiente para crear los artefactos dentro de ese grupo de recursos:

- Artefactos secundarios de **asignación de roles** de grupo de recursos ordenados por nombre de artefacto
- Artefactos de **asignación de directiva** secundarios de grupo de recursos ordenados por nombre de artefacto
- Artefactos de **plantilla de Azure Resource Manager** (plantillas de ARM) secundarios de grupo de recursos ordenados por nombre de artefacto

> [!NOTE]
> El uso de [artifacts()](../reference/blueprint-functions.md#artifacts) crea una dependencia implícita en el artefacto al que se hace referencia.

## <a name="customizing-the-sequencing-order"></a>Personalización del orden de secuenciación

Cuando crea definiciones de planos técnicos de gran tamaño, puede ser necesario que los recursos se creen en un orden específico. El patrón de uso más común de este escenario se produce cuando una definición de plano técnico incluye varias plantillas de ARM. Para que Azure Blueprints controle este patrón, permite definir el orden de secuenciación.

La ordenación se logra definiendo una propiedad `dependsOn` en JSON. La definición de plano técnico (para grupos de recursos) y los objetos de artefacto admiten esta propiedad. `dependsOn` es una matriz de cadenas de nombres de artefacto que el artefacto en particular debe crear antes de su propia creación.

> [!NOTE]
> Al crear objetos de plano técnico, cada recurso de artefacto obtiene su nombre del nombre de archivo, si usa [PowerShell](/powershell/module/az.blueprint/new-azblueprintartifact), o del punto de conexión de la dirección URL, si usa la [API REST](/rest/api/blueprints/artifacts/createorupdate). Las referencias _resourceGroup_ de los artefactos deben coincidir con los definidos en la definición de plano técnico.

### <a name="example---ordered-resource-group"></a>Ejemplo: grupo de recursos ordenado

Esta definición de plano técnico de ejemplo tiene un grupo de recursos que ha definido un orden personalizado de secuenciación declarando un valor para `dependsOn`, junto con un grupo de recursos estándar. En este caso, el artefacto denominado **assignPolicyTags** se procesará antes que el grupo de recursos **ordered-rg**.
**standard-rg** se procesará en el orden de secuenciación predeterminado.

```json
{
    "properties": {
        "description": "Example blueprint with custom sequencing order",
        "resourceGroups": {
            "ordered-rg": {
                "dependsOn": [
                    "assignPolicyTags"
                ],
                "metadata": {
                    "description": "Resource Group that waits for 'assignPolicyTags' creation"
                }
            },
            "standard-rg": {
                "metadata": {
                    "description": "Resource Group that follows the standard sequence ordering"
                }
            }
        },
        "targetScope": "subscription"
    },
    "type": "Microsoft.Blueprint/blueprints"
}
```

### <a name="example---artifact-with-custom-order"></a>Ejemplo de artefacto con orden personalizado

Este ejemplo es un artefacto de directiva que depende de una plantilla de ARM. Con el orden predeterminado, se crearía un artefacto de directiva antes que la plantilla de ARM. Esta ordenación permite que el artefacto de directiva espere a que se cree la plantilla de ARM.

```json
{
    "properties": {
        "displayName": "Assigns an identifying tag",
        "policyDefinitionId": "/providers/Microsoft.Authorization/policyDefinitions/2a0e14a6-b0a6-4fab-991a-187a4f81c498",
        "resourceGroup": "standard-rg",
        "dependsOn": [
            "customTemplate"
        ]
    },
    "kind": "policyAssignment",
    "type": "Microsoft.Blueprint/artifacts"
}
```

### <a name="example---subscription-level-template-artifact-depending-on-a-resource-group"></a>Ejemplo: artefacto de plantilla en el nivel de la suscripción que depende de un grupo de recursos

Este ejemplo es para una plantilla de ARM implementada en el nivel de suscripción que depende de un grupo de recursos. En el orden predeterminado, los artefactos de nivel de suscripción se crean antes que los grupos de recursos y los artefactos secundarios de esos grupos de recursos. El grupo de recursos se define en la definición del plano técnico de la siguiente forma:

```json
"resourceGroups": {
    "wait-for-me": {
        "metadata": {
            "description": "Resource Group that is deployed prior to the subscription level template artifact"
        }
    }
}
```

El artefacto de plantilla de nivel de suscripción que depende del grupo de recursos **wait-for-me** se define de la siguiente forma:

```json
{
    "properties": {
        "template": {
            ...
        },
        "parameters": {
            ...
        },
        "dependsOn": ["wait-for-me"],
        "displayName": "SubLevelTemplate",
        "description": ""
    },
    "kind": "template",
    "type": "Microsoft.Blueprint/blueprints/artifacts"
}
```

## <a name="processing-the-customized-sequence"></a>Procesamiento de la secuencia personalizada

Durante el proceso de creación, se usa una ordenación topológica para crear el grafo de dependencias de los artefactos del plano técnico. La comprobación garantiza que se admita cada nivel de dependencia entre los grupos de recursos y los artefactos.

Si se declara una dependencia de artefacto que no alteraría el orden predeterminado, no se realiza ningún cambio.
Un ejemplo es un grupo de recursos que depende de una directiva de nivel de suscripción. Otro ejemplo es una asignación de una directiva secundaria "standard-rg" de un grupo de recursos que dependa de la asignación de roles secundarios de un grupo de recursos "standard-rg". En ambos casos, el `dependsOn` no habría alterado el orden de secuenciación predeterminado y no se habría realizado ningún cambio.

## <a name="next-steps"></a>Pasos siguientes

- Información acerca del [ciclo de vida del plano técnico](./lifecycle.md).
- Descubra cómo utilizar [parámetros estáticos y dinámicos](./parameters.md).
- Averigüe cómo usar el [bloqueo de recursos de planos técnicos](./resource-locking.md).
- Aprenda a [actualizar las asignaciones existentes](../how-to/update-existing-assignments.md).
- Puede consultar la información de [solución de problemas generales](../troubleshoot/general.md) para resolver los problemas durante la asignación de un plano técnico.