---
title: Descripción del funcionamiento de los efectos
description: Las definiciones de Azure Policy tienen diversos efectos que determinan cómo se administra y notifica el cumplimiento.
ms.date: 08/17/2020
ms.topic: conceptual
ms.openlocfilehash: 0cfa8215d828de6d5426c3883ca1968e7a7cb542
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88544730"
---
# <a name="understand-azure-policy-effects"></a>Comprender los efectos de Azure Policy

Cada definición de directiva en Azure Policy tiene un único efecto. Dicho efecto determina lo que ocurre cuando la regla de directivas se evalúa con la coincidencia. Los efectos se comportan de manera diferente si son para un nuevo recurso, un recurso actualizado o un recurso existente.

Actualmente, se admiten estos efectos en una definición de directiva:

- [Append](#append)
- [Auditoría](#audit)
- [AuditIfNotExists](#auditifnotexists)
- [Deny](#deny)
- [DeployIfNotExists](#deployifnotexists)
- [Deshabilitada](#disabled)
- [Modify](#modify)

Los siguientes efectos se están _en desuso_:

- [EnforceOPAConstraint](#enforceopaconstraint)
- [EnforceRegoPolicy](#enforceregopolicy)

> [!IMPORTANT]
> En lugar de los efectos **EnforceOPAConstraint** o **EnforceRegoPolicy**, use _audit_ y _deny_ con el modo de Proveedor de recursos `Microsoft.Kubernetes.Data`. Se han actualizado las definiciones de directiva integradas. Cuando se modifican las asignaciones de directivas existentes de estas definiciones de directiva integradas, el parámetro _effect_ tiene que cambiarse a un valor en la lista _allowedValues_ actualizada.

## <a name="order-of-evaluation"></a>Orden de evaluación

En primer lugar Azure Policy evalúa las solicitudes para crear o actualizar un recurso. Azure Policy crea una lista de todas las asignaciones que se aplican al recurso y, a continuación, evalúa el recurso de acuerdo con cada definición. Para un [modo de Administrador de recursos](./definition-structure.md#resource-manager-modes) Azure Policy procesa algunos de los efectos antes de entregar la solicitud al proveedor de recursos adecuado. Al seguirse este orden se evita que un proveedor de recursos realice un procesamiento innecesario cuando un recurso no cumple con los controles de gobernanza diseñados de Azure Policy. Con un [modo de Proveedor de recursos](./definition-structure.md#resource-provider-modes), el proveedor de recursos administra la evaluación y el resultado, e informa a Azure Policy de los resultados.

- Primero se selecciona **Deshabilitado** para determinar si se debe evaluar la regla de directivas.
- Después se evalúan **Append** y **Modify**. Ambos efectos podrían alterar la solicitud; así, un cambio realizado por ellos podría impedir la activación de un efecto audit o deny. Estos efectos solo están disponibles con un modo de Administrador de recursos.
- Luego se evalúa **deny**. La evaluación de deny antes de audit impide el doble registro de un recurso no deseado.
- **Audit** se evalúa en último lugar.

Después de que el proveedor de recursos devuelva un código correcto en una solicitud de modo de Resource Manager, **AuditIfNotExists** y **DeployIfNotExists** evalúan para determinar si es necesario realizar una acción o un registro de cumplimiento adicionales.

## <a name="append"></a>Append

Append se utiliza para agregar campos adicionales al recurso solicitado durante la creación o actualización. Un ejemplo habitual es la especificación de direcciones IP permitidas para un recurso de almacenamiento.

> [!IMPORTANT]
> Append está pensado para su uso con propiedades que no son de etiqueta. Aunque Append puede agregar etiquetas a un recurso durante una solicitud de creación o actualización, se recomienda usar en su lugar el efecto [Modify](#modify) para las etiquetas.

### <a name="append-evaluation"></a>Evaluación de append

Append realiza la evaluación antes de que un proveedor de recursos procese la solicitud durante la creación o actualización de un recurso. Append agrega campos al recurso cuando se cumple la condición **if** de la regla de directiva. En caso de que el efecto append reemplace un valor de la solicitud original por otro, actúa como un efecto deny y rechaza la solicitud. Para anexar un nuevo valor a una matriz existente, use la versión **\[\*\]** del alias.

Cuando una definición de directiva que utiliza el efecto append se ejecuta como parte de un ciclo de evaluación, no realiza cambios en los recursos que ya existen. En su lugar, marca cualquier recurso que cumple la condición **if** como no conforme.

### <a name="append-properties"></a>Propiedades de append

Un efecto append solo tiene una matriz **details**, que es necesaria. Como **details** es una matriz, puede tomar un único par **campo-valor** o varios. Consulte [estructura de definición](definition-structure.md#fields) para obtener la lista de campos aceptables.

### <a name="append-examples"></a>Ejemplos de append

Ejemplo 1: Un único par **campo-valor** que usa un valor distinto de **\[\*\]** como [alias](definition-structure.md#aliases) con un **valor** de matriz para establecer reglas en una cuenta de almacenamiento. Cuando el alias distinto de **\[\*\]** es una matriz, el efecto anexa el **valor** como toda la matriz. Si la matriz ya existe, el conflicto ocasiona un evento de rechazo.

```json
"then": {
    "effect": "append",
    "details": [{
        "field": "Microsoft.Storage/storageAccounts/networkAcls.ipRules",
        "value": [{
            "action": "Allow",
            "value": "134.5.0.0/21"
        }]
    }]
}
```

Ejemplo 2: Un único par **campo-valor** que usa un valor distinto de **\[\*\]** como [alias](definition-structure.md#aliases) con un **valor** de matriz para establecer reglas IP en una cuenta de almacenamiento. Mediante el alias **\[\*\]** , el efecto anexa el **valor** a una matriz que es posible que ya exista. Si la matriz todavía no existe, se crea.

```json
"then": {
    "effect": "append",
    "details": [{
        "field": "Microsoft.Storage/storageAccounts/networkAcls.ipRules[*]",
        "value": {
            "value": "40.40.40.40",
            "action": "Allow"
        }
    }]
}
```

## <a name="audit"></a>Auditoría

Audit se utiliza para crear un evento de advertencia en el registro de actividad cuando se evalúa un recurso no compatible, pero no se detiene la solicitud.

### <a name="audit-evaluation"></a>Evaluación de audit

Audit es el último efecto que Azure Policy comprueba durante la creación o actualización de un recurso. Para un modo de Administrador de recursos, Azure Policy envía el recurso al proveedor de recursos. Audit funciona igual para una solicitud de recurso y un ciclo de evaluación. Azure Policy agrega una operación `Microsoft.Authorization/policies/audit/action` al registro de actividad y marca el recurso como no compatible.

### <a name="audit-properties"></a>Propiedades de audit

Para un modo de Administrador de recursos, el efecto audit no tiene propiedades adicionales para su uso en la condición **then** de la definición de directiva.

En el modo de Proveedor de recursos de `Microsoft.Kubernetes.Data`, el efecto audit tiene las siguientes subpropiedades adicionales de **details**.

- **constraintTemplate** (obligatorio)
  - La plantilla de restricción CustomResourceDefinition (CRD) que define nuevas restricciones. La plantilla define la lógica de Rego, el esquema de restricción y los parámetros de restricción que se pasan a través de **valores** desde Azure Policy.
- **constraint** (obligatorio)
  - La implementación de CRD de la plantilla de restricción. Usa los parámetros que se han pasado a través de **valores** como `{{ .Values.<valuename> }}`. En el ejemplo 2 siguiente, estos valores son `{{ .Values.excludedNamespaces }}` y `{{ .Values.allowedContainerImagesRegex }}`.
- **values** (opcional)
  - Define cualquier parámetro y valor para pasar a la restricción. Cada valor debe existir en la CRD de la plantilla de restricción.

### <a name="audit-example"></a>Ejemplo de audit

Ejemplo 1: Uso del efecto audit para los modos de Administrador de recursos.

```json
"then": {
    "effect": "audit"
}
```

Ejemplo 2: Uso del efecto audit para un modo de Proveedor de recursos de `Microsoft.Kubernetes.Data`. La información adicional en **details** define la plantilla de restricción y el CRD que se va a usar en Kubernetes para limitar las imágenes de contenedor permitidas.

```json
"then": {
    "effect": "audit",
    "details": {
        "constraintTemplate": "https://raw.githubusercontent.com/Azure/azure-policy/master/built-in-references/Kubernetes/container-allowed-images/template.yaml",
        "constraint": "https://raw.githubusercontent.com/Azure/azure-policy/master/built-in-references/Kubernetes/container-allowed-images/constraint.yaml",
        "values": {
            "allowedContainerImagesRegex": "[parameters('allowedContainerImagesRegex')]",
            "excludedNamespaces": "[parameters('excludedNamespaces')]"
        }
    }
}
```

## <a name="auditifnotexists"></a>AuditIfNotExists

AuditIfNotExists habilita la auditoría de recursos _relacionados_ con el recurso que coincide con la condición **if**, pero no tiene las propiedades especificadas en **details** de la condición **then**.

### <a name="auditifnotexists-evaluation"></a>Evaluación de AuditIfNotExists

AuditIfNotExists se ejecuta después de que un proveedor de recursos haya operado con una solicitud de creación o actualización de recursos y haya devuelto un código de estado correcto. La auditoría se produce si no hay recursos relacionados o si los recursos definidos por **ExistenceCondition** no se evalúan como true. Azure Policy agrega una operación `Microsoft.Authorization/policies/audit/action` al registro de actividad del mismo modo que el efecto Audit. Cuando se desencadena, el recurso que cumple la condición **if** es el recurso que está marcado como no conforme.

### <a name="auditifnotexists-properties"></a>Propiedades de AuditIfNotExists

La propiedad **details** de los efectos de AuditIfNotExists tiene todas las subpropiedades que definen los recursos relacionados para coincidir.

- **Type** (obligatorio)
  - Especifica el tipo del recurso relacionado para coincidir.
  - Si **details.type** es un tipo de recurso debajo del recurso de condición **if**, la directiva consultará los recursos de este **tipo** que estén dentro del alcance del recurso evaluado. De lo contrario, la directiva realizará consultas dentro del mismo grupo de recursos que el recurso evaluado.
- **Nombre** (opcional)
  - Especifica el nombre exacto del recurso para coincidir y hace que la directiva recupere un recurso específico en lugar de todos los recursos del tipo especificado.
  - Cuando los valores de condición para **if.field.type** y **then.details.type** coinciden, entonces el valor **Name** es _necesario_ y debe ser `[field('name')]`. Sin embargo, un efecto [Audit](#audit) debe tenerse en cuenta en su lugar.
- **ResourceGroupName** (opcional)
  - Permite que la coincidencia del recurso relacionado provenga de un grupo de recursos diferente.
  - No se aplica si **type** es un recurso que estaría debajo del recurso de condición **if**.
  - El valor predeterminado es el grupo de recursos del recurso de la condición **if**.
- **ExistenceScope** (opcional)
  - Los valores permitidos son _Subscription_ y _ResourceGroup_.
  - Establece el ámbito de donde obtener el recurso relacionado para que coincida.
  - No se aplica si **type** es un recurso que estaría debajo del recurso de condición **if**.
  - Para _ResourceGroup_, se limitaría al grupo de recursos del recurso de la condición **if** o al grupo de recursos especificado en **ResourceGroupName**.
  - Para _Subscription_, se consulta toda la suscripción para el recurso relacionado.
  - El valor predeterminado es _ResourceGroup_.
- **ExistenceCondition** (opcional)
  - Si no se especifica, todo recurso relacionado de **type** cumple con el efecto y no desencadena la auditoría.
  - Utiliza el mismo lenguaje que la regla de directiva para la condición **if**, pero se evalúa individualmente en relación con cada recurso relacionado.
  - Si algún recurso relacionado coincidente se evalúa como true, se cumple la condición del efecto y no se desencadena la auditoría.
  - Puede utilizar [field()] para comprobar la equivalencia con los valores en la condición **if**.
  - Por ejemplo, podría usarse para validar que el recurso primario (en la condición **if**) está en la misma ubicación de recursos que el recurso relacionado coincidente.

### <a name="auditifnotexists-example"></a>Ejemplo de AuditIfNotExists

Ejemplo: evalúa Virtual Machines para determinar si existe la extensión Antimalware y luego audita cuando faltan.

```json
{
    "if": {
        "field": "type",
        "equals": "Microsoft.Compute/virtualMachines"
    },
    "then": {
        "effect": "auditIfNotExists",
        "details": {
            "type": "Microsoft.Compute/virtualMachines/extensions",
            "existenceCondition": {
                "allOf": [{
                        "field": "Microsoft.Compute/virtualMachines/extensions/publisher",
                        "equals": "Microsoft.Azure.Security"
                    },
                    {
                        "field": "Microsoft.Compute/virtualMachines/extensions/type",
                        "equals": "IaaSAntimalware"
                    }
                ]
            }
        }
    }
}
```

## <a name="deny"></a>Denegar

Deny se utiliza para evitar una solicitud de recursos que no coincida con los estándares definidos a través de una definición de directiva, así como para impedir que se produzca un error en la solicitud.

### <a name="deny-evaluation"></a>Evaluación de deny

Al crear o actualizar un recurso coincidente en un modo de Administrador de recursos, deny evita la solicitud antes de que se envíe al proveedor de recursos. La solicitud se devuelve como un código `403 (Forbidden)`. En el portal, este estado de prohibido puede verse como un estado en la implementación que evitó la asignación de directiva. Para el modo de Proveedor de recursos, el proveedor de recursos administra la evaluación del recurso.

Durante la evaluación de los recursos existentes, los recursos que coinciden con una definición de directiva deny se marcan como no compatibles.

### <a name="deny-properties"></a>Propiedades de deny

Para un modo de Administrador de recursos, el efecto deny no tiene propiedades adicionales para su uso en la condición **then** de la definición de directiva.

En el modo de Proveedor de recursos de `Microsoft.Kubernetes.Data`, el efecto deny tiene las siguientes subpropiedades adicionales de **details**.

- **constraintTemplate** (obligatorio)
  - La plantilla de restricción CustomResourceDefinition (CRD) que define nuevas restricciones. La plantilla define la lógica de Rego, el esquema de restricción y los parámetros de restricción que se pasan a través de **valores** desde Azure Policy.
- **constraint** (obligatorio)
  - La implementación de CRD de la plantilla de restricción. Usa los parámetros que se han pasado a través de **valores** como `{{ .Values.<valuename> }}`. En el ejemplo 2 siguiente, estos valores son `{{ .Values.excludedNamespaces }}` y `{{ .Values.allowedContainerImagesRegex }}`.
- **values** (opcional)
  - Define cualquier parámetro y valor para pasar a la restricción. Cada valor debe existir en la CRD de la plantilla de restricción.

### <a name="deny-example"></a>Ejemplo de deny

Ejemplo 1: Uso del efecto deny para los modos de Administrador de recursos.

```json
"then": {
    "effect": "deny"
}
```

Ejemplo 2: Uso del efecto deny para un modo de Proveedor de recursos de `Microsoft.Kubernetes.Data`. La información adicional en **details** define la plantilla de restricción y el CRD que se va a usar en Kubernetes para limitar las imágenes de contenedor permitidas.

```json
"then": {
    "effect": "deny",
    "details": {
        "constraintTemplate": "https://raw.githubusercontent.com/Azure/azure-policy/master/built-in-references/Kubernetes/container-allowed-images/template.yaml",
        "constraint": "https://raw.githubusercontent.com/Azure/azure-policy/master/built-in-references/Kubernetes/container-allowed-images/constraint.yaml",
        "values": {
            "allowedContainerImagesRegex": "[parameters('allowedContainerImagesRegex')]",
            "excludedNamespaces": "[parameters('excludedNamespaces')]"
        }
    }
}
```

## <a name="deployifnotexists"></a>DeployIfNotExists

Similar a AuditIfNotExists, una definición de directiva DeployIfNotExists ejecuta una implementación de plantilla cuando se cumple la condición.

> [!NOTE]
> Las [plantillas anidadas](../../../azure-resource-manager/templates/linked-templates.md#nested-template) son compatibles con **deployIfNotExists**, pero las [plantillas vinculadas](../../../azure-resource-manager/templates/linked-templates.md#linked-template) no son compatibles actualmente.

### <a name="deployifnotexists-evaluation"></a>Evaluación de DeployIfNotExists

DeployIfNotExists se ejecuta unos 15 minutos después de que un proveedor de recursos haya controlado una solicitud de creación o actualización de recursos y haya devuelto un código de estado correcto. La implementación de una plantilla se produce si no hay recursos relacionados o si los recursos definidos por **ExistenceCondition** no se evalúan como true.
La duración de la implementación depende de la complejidad de los recursos incluidos en la plantilla.

Durante un ciclo de evaluación, las definiciones de directiva con un efecto DeployIfNotExists que coinciden con los recursos se marcan como no compatibles, pero no se realiza ninguna acción en dicho recurso. Los recursos no conformes existentes se pueden solucionar con una [tarea de corrección](../how-to/remediate-resources.md).

### <a name="deployifnotexists-properties"></a>Propiedades de DeployIfNotExists

La propiedad **details** del efecto DeployIfNotExists tiene todas las subpropiedades que definen los recursos relacionados con los que se establece la coincidencia y la implementación de plantilla que se ejecuta.

- **Type** (obligatorio)
  - Especifica el tipo del recurso relacionado para coincidir.
  - Comienza tratando de recuperar un recurso debajo del recurso de condición **if** , luego consulta dentro del mismo grupo de recursos que el recurso de condición **if**.
- **Nombre** (opcional)
  - Especifica el nombre exacto del recurso para coincidir y hace que la directiva recupere un recurso específico en lugar de todos los recursos del tipo especificado.
  - Cuando los valores de condición para **if.field.type** y **then.details.type** coinciden, entonces el valor **Name** es _necesario_ y debe ser `[field('name')]`.
- **ResourceGroupName** (opcional)
  - Permite que la coincidencia del recurso relacionado provenga de un grupo de recursos diferente.
  - No se aplica si **type** es un recurso que estaría debajo del recurso de condición **if**.
  - El valor predeterminado es el grupo de recursos del recurso de la condición **if**.
  - Si se ejecuta una implementación de plantilla, se implementa en el grupo de recursos de este valor.
- **ExistenceScope** (opcional)
  - Los valores permitidos son _Subscription_ y _ResourceGroup_.
  - Establece el ámbito de donde obtener el recurso relacionado para que coincida.
  - No se aplica si **type** es un recurso que estaría debajo del recurso de condición **if**.
  - Para _ResourceGroup_, se limitaría al grupo de recursos del recurso de la condición **if** o al grupo de recursos especificado en **ResourceGroupName**.
  - Para _Subscription_, se consulta toda la suscripción para el recurso relacionado.
  - El valor predeterminado es _ResourceGroup_.
- **ExistenceCondition** (opcional)
  - Si no se especifica, todo recurso relacionado de **type** cumple con el efecto y no desencadena la implementación.
  - Utiliza el mismo lenguaje que la regla de directiva para la condición **if**, pero se evalúa individualmente en relación con cada recurso relacionado.
  - Si algún recurso relacionado coincidente se evalúa como true, se cumple la condición del efecto y no se desencadena la implementación.
  - Puede utilizar [field()] para comprobar la equivalencia con los valores en la condición **if**.
  - Por ejemplo, podría usarse para validar que el recurso primario (en la condición **if**) está en la misma ubicación de recursos que el recurso relacionado coincidente.
- **roleDefinitionIds** (obligatorio)
  - Esta propiedad debe incluir una matriz de cadenas que coinciden con el identificador de rol de control de acceso basado en rol accesible por la suscripción. Para obtener más información, vea [remediation - configure policy definition](../how-to/remediate-resources.md#configure-policy-definition) (corrección: configurar la definición de directiva).
- **DeploymentScope** (opcional)
  - Los valores permitidos son _Subscription_ y _ResourceGroup_.
  - Establece el tipo de implementación que se desencadena. La _Suscripción_ indica una [implementación a nivel de suscripción](../../../azure-resource-manager/templates/deploy-to-subscription.md), _ResourceGroup_ indica una implementación en un grupo de recursos.
  - En la _Implementación_ debe especificarse una propiedad _location_ al usar las implementaciones de nivel de suscripción.
  - El valor predeterminado es _ResourceGroup_.
- **Deployment** (obligatorio)
  - Esta propiedad debe incluir la implementación de plantilla completa, ya que luego se pasaría a la API PUT `Microsoft.Resources/deployments`. Para más información, consulte [Deployments REST API](/rest/api/resources/deployments) (API REST de implementaciones).

  > [!NOTE]
  > Todas las funciones dentro de la propiedad **Deployment** se evalúan como componentes de la plantilla, no la directiva. La excepción es la propiedad **parameters** que pasa los valores de la directiva a la plantilla. El **valor** en esta sección bajo el nombre de un parámetro de plantilla se usa para realizar este paso de valor (vea _fullDbName_ en el ejemplo DeployIfNotExists).

### <a name="deployifnotexists-example"></a>Ejemplo de DeployIfNotExists

Ejemplo: evalúa las bases de datos de SQL Server para determinar si está habilitado transparentDataEncryption. De lo contrario, se ejecuta una implementación para habilitarlo.

```json
"if": {
    "field": "type",
    "equals": "Microsoft.Sql/servers/databases"
},
"then": {
    "effect": "DeployIfNotExists",
    "details": {
        "type": "Microsoft.Sql/servers/databases/transparentDataEncryption",
        "name": "current",
        "roleDefinitionIds": [
            "/subscriptions/{subscriptionId}/providers/Microsoft.Authorization/roleDefinitions/{roleGUID}",
            "/providers/Microsoft.Authorization/roleDefinitions/{builtinroleGUID}"
        ],
        "existenceCondition": {
            "field": "Microsoft.Sql/transparentDataEncryption.status",
            "equals": "Enabled"
        },
        "deployment": {
            "properties": {
                "mode": "incremental",
                "template": {
                    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
                    "contentVersion": "1.0.0.0",
                    "parameters": {
                        "fullDbName": {
                            "type": "string"
                        }
                    },
                    "resources": [{
                        "name": "[concat(parameters('fullDbName'), '/current')]",
                        "type": "Microsoft.Sql/servers/databases/transparentDataEncryption",
                        "apiVersion": "2014-04-01",
                        "properties": {
                            "status": "Enabled"
                        }
                    }]
                },
                "parameters": {
                    "fullDbName": {
                        "value": "[field('fullName')]"
                    }
                }
            }
        }
    }
}
```

## <a name="disabled"></a>Disabled

Este efecto es útil para probar situaciones o cuando la definición de directiva ha parametrizado el efecto. Esta flexibilidad permite deshabilitar una única asignación en lugar de deshabilitar todas las asignaciones de la directiva.

Una alternativa al efecto Disabled es **enforcementMode**, que se establece en la asignación de directiva.
Cuando **enforcementMode**  es _Disabled_, los recursos se siguen evaluando. El registro, como los registros de actividad, y el efecto de la directiva no se producen. Para más información, consulte [Asignación de directivas: modo de cumplimiento](./assignment-structure.md#enforcement-mode).

## <a name="enforceopaconstraint"></a>EnforceOPAConstraint

Este efecto se usa con un _modo_ de definición de directiva de `Microsoft.Kubernetes.Data`. Se usa para pasar reglas de control de admisión de Gatekeeper v3 definidas con [OPA Constraint Framework](https://github.com/open-policy-agent/frameworks/tree/master/constraint#opa-constraint-framework) en [Open Policy Agent](https://www.openpolicyagent.org/) (OPA) para clústeres de Kubernetes en Azure.

> [!NOTE]
> [Azure Policy para Kubernetes](./policy-for-kubernetes.md) está en versión preliminar y solo admite grupos de nodos de Linux y definiciones de directivas integradas. Las definiciones de directivas integradas se encuentran en la categoría **Kubernetes**. Las definiciones de directivas de versión preliminar limitadas con efecto **EnforceOPAConstraint** y la categoría **Kubernetes Service** relacionada están _en desuso_. En su lugar, use los efectos _audit_ y _deny_ con el modo de Proveedor de recursos `Microsoft.Kubernetes.Data`.

### <a name="enforceopaconstraint-evaluation"></a>Evaluación de EnforceOPAConstraint

El controlador de admisión de Open Policy Agent evalúa cualquier solicitud nueva del clúster en tiempo real.
Cada 15 minutos, se completa un análisis completo del clúster y los resultados se envían a Azure Policy.

### <a name="enforceopaconstraint-properties"></a>Propiedades de EnforceOPAConstraint

La propiedad de **detalles** del efecto EnforceOPAConstraint tiene las subpropiedades que describen la regla de control de admisión de Gatekeeper v3.

- **constraintTemplate** (obligatorio)
  - La plantilla de restricción CustomResourceDefinition (CRD) que define nuevas restricciones. La plantilla define la lógica de Rego, el esquema de restricción y los parámetros de restricción que se pasan a través de **valores** desde Azure Policy.
- **constraint** (obligatorio)
  - La implementación de CRD de la plantilla de restricción. Usa los parámetros que se han pasado a través de **valores** como `{{ .Values.<valuename> }}`. En el ejemplo siguiente, estos valores son `{{ .Values.cpuLimit }}` y `{{ .Values.memoryLimit }}`.
- **values** (opcional)
  - Define cualquier parámetro y valor para pasar a la restricción. Cada valor debe existir en la CRD de la plantilla de restricción.

### <a name="enforceopaconstraint-example"></a>Ejemplo de EnforceOPAConstraint

Ejemplo: regla de control de admisión de Gatekeeper v3 para establecer los límites de recursos de memoria y CPU de contenedor en Kubernetes.

```json
"if": {
    "allOf": [
        {
            "field": "type",
            "in": [
                "Microsoft.ContainerService/managedClusters",
                "AKS Engine"
            ]
        },
        {
            "field": "location",
            "equals": "westus2"
        }
    ]
},
"then": {
    "effect": "enforceOPAConstraint",
    "details": {
        "constraintTemplate": "https://raw.githubusercontent.com/Azure/azure-policy/master/built-in-references/Kubernetes/container-resource-limits/template.yaml",
        "constraint": "https://raw.githubusercontent.com/Azure/azure-policy/master/built-in-references/Kubernetes/container-resource-limits/constraint.yaml",
        "values": {
            "cpuLimit": "[parameters('cpuLimit')]",
            "memoryLimit": "[parameters('memoryLimit')]"
        }
    }
}
```

## <a name="enforceregopolicy"></a>EnforceRegoPolicy

Este efecto se usa con un _modo_ de definición de directiva de `Microsoft.ContainerService.Data`. Se usa para pasar las reglas de control de admisión de Gatekeeper v2 definidas con [Rego](https://www.openpolicyagent.org/docs/latest/policy-language/#what-is-rego) a [Open Policy Agent](https://www.openpolicyagent.org/) (OPA) en [Azure Kubernetes Service](../../../aks/intro-kubernetes.md).

> [!NOTE]
> [Azure Policy para Kubernetes](./policy-for-kubernetes.md) está en versión preliminar y solo admite grupos de nodos de Linux y definiciones de directivas integradas. Las definiciones de directivas integradas se encuentran en la categoría **Kubernetes**. Las definiciones de directivas de versión preliminar limitadas con efecto **EnforceRegoPolicy** y la categoría **Kubernetes Service** relacionada están _en desuso_. En su lugar, use los efectos _audit_ y _deny_ con el modo de Proveedor de recursos `Microsoft.Kubernetes.Data`.

### <a name="enforceregopolicy-evaluation"></a>Evaluación de EnforceRegoPolicy

El controlador de admisión de Open Policy Agent evalúa cualquier solicitud nueva del clúster en tiempo real.
Cada 15 minutos, se completa un análisis completo del clúster y los resultados se envían a Azure Policy.

### <a name="enforceregopolicy-properties"></a>Propiedades de EnforceRegoPolicy

La propiedad de **detalles** del efecto EnforceRegoPolicy tiene las subpropiedades que describen la regla de control de admisión de Gatekeeper v2.

- **policyId** (obligatorio)
  - Un nombre único que pasa como parámetro a la regla de control de admisión de Rego.
- **policy** (obligatorio)
  - Especifica el URI de la regla de control de admisión de Rego.
- **policyParameters** (opcional)
  - Define cualquier parámetro y valor para pasar a la directiva de Rego.

### <a name="enforceregopolicy-example"></a>Ejemplo de EnforceRegoPolicy

Ejemplo: Regla de control de admisión de Gatekeeper v2 para permitir solo las imágenes de contenedor especificadas en AKS.

```json
"if": {
    "allOf": [
        {
            "field": "type",
            "equals": "Microsoft.ContainerService/managedClusters"
        },
        {
            "field": "location",
            "equals": "westus2"
        }
    ]
},
"then": {
    "effect": "EnforceRegoPolicy",
    "details": {
        "policyId": "ContainerAllowedImages",
        "policy": "https://raw.githubusercontent.com/Azure/azure-policy/master/built-in-references/KubernetesService/container-allowed-images/limited-preview/gatekeeperpolicy.rego",
        "policyParameters": {
            "allowedContainerImagesRegex": "[parameters('allowedContainerImagesRegex')]"
        }
    }
}
```

## <a name="modify"></a>Modificar

Modify se usa para agregar, actualizar o quitar etiquetas de un recurso durante la creación o actualización. Un ejemplo habitual es la actualización de etiquetas en recursos como costCenter. Una directiva de modificación siempre debe tener `mode` establecido en _Indexed_ a menos que el recurso de destino sea un grupo de recursos. Los recursos no conformes existentes se pueden solucionar con una [tarea de corrección](../how-to/remediate-resources.md). Una sola regla de Modify puede tener cualquier número de operaciones.

> [!IMPORTANT]
> Actualmente, Modify solo se usa con etiquetas. Si está administrando etiquetas, se recomienda usar Modify en lugar de Append, ya que Modify proporciona tipos de operación adicionales y la capacidad de corregir los recursos existentes. Sin embargo, se recomienda Append si no se puede crear una identidad administrada.

### <a name="modify-evaluation"></a>Evaluación de Modify

Modify se evalúa antes de que un proveedor de recursos procese la solicitud durante la creación o actualización de un recurso. Modify agrega o actualiza etiquetas en un recurso cuando se cumple la condición **if** de la regla de directiva.

Cuando una definición de directiva que utiliza el efecto Modify se ejecuta como parte de un ciclo de evaluación, no realiza cambios en los recursos que ya existen. En su lugar, marca cualquier recurso que cumple la condición **if** como no conforme.

### <a name="modify-properties"></a>Propiedades de Modify

La propiedad **details** del efecto Modify tiene todas las subpropiedades que definen los permisos necesarios para la corrección y las propiedades **operations** que se usan para agregar, actualizar o quitar valores de etiqueta.

- **roleDefinitionIds** (obligatorio)
  - Esta propiedad debe incluir una matriz de cadenas que coinciden con el identificador de rol de control de acceso basado en rol accesible por la suscripción. Para obtener más información, vea [remediation - configure policy definition](../how-to/remediate-resources.md#configure-policy-definition) (corrección: configurar la definición de directiva).
  - El rol definido debe incluir todas las operaciones concedidas al rol [Colaborador](../../../role-based-access-control/built-in-roles.md#contributor).
- **conflictEffect** (opcional)
  - Determina qué definición de directiva "gana" en caso de que haya más de una definición de directiva que modifique la misma propiedad.
    - En el caso de los recursos nuevos o actualizados, la definición de la directiva con _deny_ tiene prioridad. Las definiciones de directivas con _Audit_ omiten todas las **operaciones**. Si más de una definición de directiva tiene _deny_, la solicitud se deniega como conflicto. Si todas las definiciones de directiva tienen _audit_, no se procesa ninguna de las **operaciones** de las definiciones de directiva en conflicto.
    - En el caso de los recursos existentes, si hay más de una definición de directiva tenga _deny_, el estado de cumplimiento es _Conflict_. Si una o varias definiciones de directivas tienen _deny_, cada asignación devuelve un estado de cumplimiento de _Non-compliant_.
  - Valores disponibles: _audit_, _deny_, _disabled_.
  - El valor predeterminado es _deny_.
- **operations** (obligatorio)
  - Una matriz de todas las operaciones de etiqueta que se van a llevar a cabo en los recursos coincidentes.
  - Propiedades:
    - **operation** (obligatorio)
      - Define qué acción se va a realizar en un recurso coincidente. Las opciones son: _addOrReplace_, _Add_, _Remove_. _Add_ se comporta de forma similar al efecto [Append](#append).
    - **field** (obligatorio)
      - La etiqueta que se va a agregar, reemplazar o quitar. Los nombres de etiqueta deben seguir la misma convención de nomenclatura que otros [campos](./definition-structure.md#fields).
    - **value** (opcional)
      - Valor en el que se va a establecer la etiqueta.
      - Esta propiedad es necesaria si **operation** es _addOrReplace_ o _Add_.

### <a name="modify-operations"></a>Operaciones de Modify

La matriz de propiedades **operations** permite modificar varias etiquetas de maneras diferentes a partir de una única definición de directiva. Cada operación se compone de las propiedades **operation**, **field** y **value**. Operation determina qué hace la tarea de corrección en las etiquetas, field determina qué etiqueta se modifica y value define el nuevo valor de la etiqueta. En el ejemplo siguiente se realizan los siguientes cambios en la etiqueta:

- Se establece la etiqueta `environment` en "Test", incluso si ya existe con un valor diferente.
- Se quita la etiqueta `TempResource`.
- Se establece la etiqueta `Dept` en el parámetro de directiva _DeptName_ configurado en la asignación de directiva.

```json
"details": {
    ...
    "operations": [
        {
            "operation": "addOrReplace",
            "field": "tags['environment']",
            "value": "Test"
        },
        {
            "operation": "Remove",
            "field": "tags['TempResource']",
        },
        {
            "operation": "addOrReplace",
            "field": "tags['Dept']",
            "value": "[parameters('DeptName')]"
        }
    ]
}
```

La propiedad **operation** tiene las siguientes opciones:

|Operación |Descripción |
|-|-|
|addOrReplace |Agrega la etiqueta y el valor definidos al recurso, incluso si la etiqueta ya existe con un valor diferente. |
|Sumar |Agrega la etiqueta y el valor definidos al recurso. |
|Remove |Quita la etiqueta definida del recurso. |

### <a name="modify-examples"></a>Ejemplos de Modify

Ejemplo 1: se agrega la etiqueta `environment` y se reemplazan las etiquetas `environment` existentes por "Test":

```json
"then": {
    "effect": "modify",
    "details": {
        "roleDefinitionIds": [
            "/providers/Microsoft.Authorization/roleDefinitions/b24988ac-6180-42a0-ab88-20f7382dd24c"
        ],
        "operations": [
            {
                "operation": "addOrReplace",
                "field": "tags['environment']",
                "value": "Test"
            }
        ]
    }
}
```

Ejemplo 2: se quita la etiqueta `env` y se agrega la etiqueta `environment` o se reemplazan las etiquetas `environment` existentes por un valor parametrizado:

```json
"then": {
    "effect": "modify",
    "details": {
        "roleDefinitionIds": [
            "/providers/Microsoft.Authorization/roleDefinitions/b24988ac-6180-42a0-ab88-20f7382dd24c"
        ],
        "conflictEffect": "deny",
        "operations": [
            {
                "operation": "Remove",
                "field": "tags['env']"
            },
            {
                "operation": "addOrReplace",
                "field": "tags['environment']",
                "value": "[parameters('tagValue')]"
            }
        ]
    }
}
```

## <a name="layering-policy-definitions"></a>Definiciones de directivas en capas

Un recurso puede verse afectado por varias asignaciones. Estas asignaciones pueden estar en el mismo o diferentes ámbitos. Cada una de estas asignaciones también es probable que tenga un efecto diferente al definido. La condición y el efecto de cada directiva se evalúan por separado. Por ejemplo:

- Directiva 1
  - Restringe la ubicación de recursos a "westus".
  - Se asigna a la suscripción A.
  - Efecto deny.
- Directiva 2
  - Restringe la ubicación de recursos a "eastus".
  - Se asigna al grupo de recursos B de la suscripción A.
  - Efecto audit.
  
Esta configuración produciría el resultado siguiente:

- Cualquier recurso que ya esté en el grupo de recursos B en "eastus" es compatible con la directiva 2 y no compatible con la directiva 1.
- Cualquier recurso que ya esté en el grupo de recursos B y que no esté en "eastus" no es compatible con la directiva 2 ni con la directiva 1 si no está en "westus".
- La directiva 1 deniega cualquier recurso nuevo en la suscripción A que no esté en "westus".
- Cualquier recurso nuevo en la suscripción A y en el grupo de recursos B en "westus"se crea y no es compatible con la directiva 2.

Si tanto la directiva 1 como la directiva 2 tienen efecto deny, la situación cambia a:

- Cualquier recurso que ya esté en el grupo de recursos B y no en "eastus" no es compatible con la directiva 2.
- Cualquier recurso que ya esté en el grupo de recursos B y no en "westus" no es compatible con la directiva 1.
- La directiva 1 deniega cualquier recurso nuevo en la suscripción A que no esté en "westus".
- Cualquier recurso nuevo que esté en el grupo de recursos B de la suscripción A se deniega.

Cada asignación se evalúa individualmente. Por lo tanto, no hay ninguna oportunidad de que un recurso se escape por diferencias en el ámbito. El resultado neto de las definiciones de directivas en capas se considera **acumulativo en su mayoría restrictivo**. Por ejemplo, si tanto la directiva 1 como la 2 tuviesen un efecto Deny, las definiciones de directivas en conflicto y superpuestas bloquearían un recurso. Si aún necesita que el recurso se cree en el ámbito de destino, revise las exclusiones en cada asignación para validar si las asignaciones de directivas adecuadas están incidiendo en los ámbitos adecuados.

## <a name="next-steps"></a>Pasos siguientes

- Puede consultar ejemplos en [Ejemplos de Azure Policy](../samples/index.md).
- Revise la [estructura de definición de Azure Policy](definition-structure.md).
- Obtenga información acerca de cómo se pueden [crear directivas mediante programación](../how-to/programmatically-create.md).
- Obtenga información sobre cómo [obtener datos de cumplimiento](../how-to/get-compliance-data.md).
- Obtenga información sobre cómo [corregir recursos no compatibles](../how-to/remediate-resources.md).
- En [Organización de los recursos con grupos de administración de Azure](../../management-groups/overview.md), obtendrá información sobre lo que es un grupo de administración.
