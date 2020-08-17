---
title: Errores de plantilla no válida
description: Describe cómo resolver errores de plantilla no válida al implementar plantillas de Azure Resource Manager.
ms.topic: troubleshooting
ms.date: 05/22/2020
ms.openlocfilehash: 3e68d4634d0092d05222348ca0db0116422faec3
ms.sourcegitcommit: f353fe5acd9698aa31631f38dd32790d889b4dbb
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87373817"
---
# <a name="resolve-errors-for-invalid-template"></a>Resolución de errores de plantilla no válida

Ese artículo describe cómo resolver errores de plantilla no válida.

## <a name="symptom"></a>Síntoma

Al implementar una plantilla, recibe un error que indica:

```
Code=InvalidTemplate
Message=<varies>
```

El mensaje de error depende del tipo de error.

## <a name="cause"></a>Causa

Este error puede tener distintos orígenes. Normalmente implican un error estructural o de sintaxis en la plantilla.

<a id="syntax-error"></a>

## <a name="solution-1---syntax-error"></a>Solución 1: error de sintaxis

Si recibe un mensaje de error que indica que la plantilla no superó la validación, puede que esta tenga un problema de sintaxis.

```
Code=InvalidTemplate
Message=Deployment template validation failed
```

Es fácil cometer este error porque las expresiones de plantilla pueden ser complejas. Por ejemplo, la siguiente asignación de nombre de una cuenta de almacenamiento contiene un conjunto de corchetes, tres funciones, tres conjuntos de paréntesis, un conjunto de comillas simples y una propiedad:

```json
"name": "[concat('storage', uniqueString(resourceGroup().id))]",
```

Si no proporciona la sintaxis coincidente, la plantilla genera un valor que es diferente del que se espera.

Si recibe este tipo de error, revise cuidadosamente la sintaxis de las expresiones. Considere la posibilidad de utilizar un editor JSON como [Visual Studio](create-visual-studio-deployment-project.md) o [Visual Studio Code](quickstart-create-templates-use-visual-studio-code.md) que puede avisarlo de los errores de sintaxis.

<a id="incorrect-segment-lengths"></a>

## <a name="solution-2---incorrect-segment-lengths"></a>Solución 2: longitudes de segmentos incorrectas

Cuando el nombre del recurso no tiene el formato correcto, se produce otro error de plantilla no válida.

```
Code=InvalidTemplate
Message=Deployment template validation failed: 'The template resource {resource-name}'
for type {resource-type} has incorrect segment lengths.
```

Un recurso de nivel de raíz debe tener un segmento menos en el nombre que en el tipo de recurso. Cada segmento se distingue por una barra diagonal. En el ejemplo siguiente, el tipo tiene dos segmentos y el nombre tiene uno, por lo que es un **nombre válido**.

```json
{
  "type": "Microsoft.Web/serverfarms",
  "name": "myHostingPlanName",
  ...
}
```

Pero el ejemplo siguiente **no es un nombre válido** porque el nombre tiene el mismo número de segmentos que el tipo.

```json
{
  "type": "Microsoft.Web/serverfarms",
  "name": "appPlan/myHostingPlanName",
  ...
}
```

Para los recursos secundarios, el tipo y el nombre deben tener el mismo número de segmentos. Este número de segmentos tiene sentido, ya que el tipo y el nombre completo del elemento secundario incluyen el tipo y el nombre del elemento primario. Por lo tanto, el nombre completo sigue teniendo un segmento menos que el tipo completo.

```json
"resources": [
  {
    "type": "Microsoft.KeyVault/vaults",
    "name": "contosokeyvault",
    ...
    "resources": [
      {
        "type": "secrets",
        "name": "appPassword",
        ...
      }
    ]
  }
]
```

Obtener los segmentos correctos puede resultar complicado con los tipos de Resource Manager que se aplican en los proveedores de recursos. Por ejemplo, para aplicar un bloqueo de recurso a un sitio web, el tipo debe tener 4 segmentos. Por lo tanto, el nombre tiene 3 segmentos:

```json
{
  "type": "Microsoft.Web/sites/providers/locks",
  "name": "[concat(variables('siteName'),'/Microsoft.Authorization/MySiteLock')]",
  ...
}
```

<a id="parameter-not-valid"></a>

## <a name="solution-3---parameter-is-not-valid"></a>Solución 3: el parámetro no es válido

Si proporciona un valor de parámetro que no es uno de los valores permitidos, recibe un mensaje de error similar al siguiente:

```
Code=InvalidTemplate;
Message=Deployment template validation failed: 'The provided value {parameter value}
for the template parameter {parameter name} is not valid. The parameter value is not
part of the allowed values
```

Compruebe los valores permitidos de la plantilla y proporcione uno durante la implementación. Para más información sobre los valores de parámetros permitidos, consulte [Sección de parámetros de plantillas de Azure Resource Manager](template-syntax.md#parameters).

<a id="too-many-resource-groups"></a>

## <a name="solution-4---too-many-target-resource-groups"></a>Solución 4: hay demasiados grupos de recursos de destino

Puede que vea este error en implementaciones anteriores porque estaba limitado a cinco grupos de recursos de destino en una sola implementación. En mayo de 2020 ese límite se ha aumentado a 800 grupos de recursos. Para más información, consulte [Implementación de recursos de Azure en varias suscripciones y grupos de recursos](cross-scope-deployment.md).

<a id="circular-dependency"></a>

## <a name="solution-5---circular-dependency-detected"></a>Solución 5: se detectó una dependencia circular

Este error se recibe cuando los recursos dependen unos de otros de una forma que impide que se inicie la implementación. Una combinación de interdependencias hace que dos o más recursos esperen otros recursos que también están esperando. Por ejemplo, resource1 depende de resource3, resource2 depende de resource1 y resource3 depende de resource2. Para resolver este problema, normalmente se eliminan las dependencias innecesarias.

Para resolver una dependencia circular:

1. En la plantilla, busque el recurso identificado en la dependencia circular.
2. Para ese recurso, examine la propiedad **dependsOn** y todos los usos de la función **reference** para ver de qué recursos depende.
3. Examine esos recursos para ver de qué recursos dependen. Siga las dependencias hasta que observe un recurso que dependa del recurso original.
5. En el caso de los recursos que intervienen en la dependencia circular, examine con cuidado todos los usos de la propiedad **dependsOn** para identificar las dependencias que no sean necesarias. Quite esas dependencias. Si no está seguro de si una dependencia es necesaria, pruebe a quitarla.
6. Vuelva a implementar la plantilla.

Al quitar los valores de la propiedad **dependsOn**, se pueden provocar errores al implementar la plantilla. Si recibe un error, vuelva a agregar la dependencia a la plantilla.

Si con ese enfoque no se soluciona la dependencia circular, considere la posibilidad de mover parte de la lógica de implementación a recursos secundarios (por ejemplo, extensiones o valores de configuración). Configure esos recursos secundarios para que se implementen después de los recursos que intervienen en la dependencia circular. Por ejemplo, suponga que va a implementar dos máquinas virtuales pero debe establecer propiedades en cada una que hagan referencia a la otra. Puede implementarlas en el orden siguiente:

1. vm1
2. vm2
3. La extensión en vm1 depende de vm1 y vm2. La extensión establece valores en vm1 que obtiene de vm2.
4. La extensión en vm2 depende de vm1 y vm2. La extensión establece valores en vm2 que obtiene de vm1.

El mismo enfoque funciona para las aplicaciones de App Service. Considere la posibilidad de mover los valores de configuración a un recurso secundario del recurso de aplicación. Puede implementar dos aplicaciones web en el orden siguiente:

1. webapp1
2. webapp2
3. La configuración de webapp1 depende de webapp1 y webapp2. Contiene la configuración de aplicación con valores de webapp2.
4. La configuración de webapp2 depende de webapp1 y webapp2. Contiene la configuración de aplicación con valores de webapp1.
