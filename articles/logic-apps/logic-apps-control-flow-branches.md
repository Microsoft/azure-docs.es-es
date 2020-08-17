---
title: Crear o combinar ramas paralelas de las acciones en flujos de trabajo
description: Obtenga información sobre cómo crear o combinar bifurcaciones en ejecución en paralelo para acciones de flujo de trabajo independientes en Azure Logic Apps
services: logic-apps
ms.suite: integration
ms.reviewer: klam, logicappspm
ms.topic: article
ms.date: 10/10/2018
ms.openlocfilehash: 9dafb702117a7e171ff88b3c77d7b8b79503636f
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87066467"
---
# <a name="create-or-join-parallel-branches-for-workflow-actions-in-azure-logic-apps"></a>Crear o combinar ramas paralelas de las acciones de los flujos de trabajo en Azure Logic Apps

De forma predeterminada, las acciones en flujos de trabajo de aplicaciones lógicas se ejecutan simultáneamente. Para llevar a cabo acciones independientes al mismo tiempo, puede crear [ramas paralelas](#parallel-branches)y luego [unir esas ramas](#join-branches) más adelante en el flujo. 

> [!TIP] 
> Si tiene un desencadenador que recibe una matriz y desea ejecutar un flujo de trabajo para cada elemento de matriz, puede *desagrupar* esa matriz con la propiedad de desencadenador [**SplitOn**](../logic-apps/logic-apps-workflow-actions-triggers.md#split-on-debatch).

## <a name="prerequisites"></a>Prerrequisitos

* Suscripción a Azure. Si aún no tiene una, [regístrese para obtener una cuenta de Azure gratuita](https://azure.microsoft.com/free/). 

* Conocimientos básicos acerca de [cómo crear aplicaciones lógicas](../logic-apps/quickstart-create-first-logic-app-workflow.md)

<a name="parallel-branches"></a>

## <a name="add-parallel-branch"></a>Incorporación de una rama paralela

Para ejecutar pasos independientes al mismo tiempo, puede agregar ramas paralelas junto a un paso existente. 

![Ejecución de pasos en paralelo](media/logic-apps-control-flow-branches/parallel.png)

La aplicación lógica espera a que todas las ramas finalicen antes de continuar con el flujo de trabajo. Las ramas paralelas se ejecutan solo cuando sus valores de propiedad `runAfter` coinciden con estado del paso primario terminado. Por ejemplo, `branchAction1` y `branchAction2` están configuradas para ejecutarse solo cuando `parentAction` se completa con el estado `Succeeded`.

> [!NOTE]
> Antes de empezar, la aplicación lógica tiene que tener un paso al que se puedan agregar ramas paralelas.

1. En <a href="https://portal.azure.com" target="_blank">Azure Portal</a>, abra la aplicación lógica en Diseñador de aplicación lógica.

1. Mueva el puntero sobre la flecha situada encima del paso donde desea agregar ramas paralelas. Elija el signo **más** ( **+** ) que aparece y, luego, elija **Agregar una rama paralela**. 

   ![Incorporación de una rama paralela](media/logic-apps-control-flow-branches/add-parallel-branch.png)

1. En el cuadro de búsqueda, busque y seleccione la acción que desee.

   ![Búsqueda y selección de la acción deseada](media/logic-apps-control-flow-branches/find-select-parallel-action.png)

   La acción seleccionada ahora aparece en la rama paralela, por ejemplo:

   ![Búsqueda y selección de la acción deseada](media/logic-apps-control-flow-branches/added-parallel-branch.png)

1. Ahora, en cada rama paralela, agregue los pasos que desee. Para agregar otra acción a una rama, mueva el puntero bajo la acción en la que desea agregar una acción secuencial. Elija el signo **más** ( **+** ) que aparece y, a continuación, seleccione **Agregar una acción**.

   ![Incorporación de una acción secuencial a una rama paralela](media/logic-apps-control-flow-branches/add-sequential-action.png)

1. En el cuadro de búsqueda, busque y seleccione la acción que desee.

   ![Búsqueda y selección de la acción secuencial](media/logic-apps-control-flow-branches/find-select-sequential-action.png)

   La acción seleccionada ahora aparece en la rama actual, por ejemplo:

   ![Búsqueda y selección de la acción secuencial](media/logic-apps-control-flow-branches/added-sequential-action.png)

Para volver a combinar ramas [una las ramas paralelas](#join-branches). 

<a name="parallel-json"></a>

## <a name="parallel-branch-definition-json"></a>Definición de la rama paralela (JSON)

Si está trabajando en la vista de código, puede definir la estructura paralela en la definición JSON de la aplicación lógica en su lugar, por ejemplo:

``` json
{
  "triggers": {
    "myTrigger": {}
  },
  "actions": {
    "parentAction": {
      "type": "<action-type>",
      "inputs": {},
      "runAfter": {}
    },
    "branchAction1": {
      "type": "<action-type>",
      "inputs": {},
      "runAfter": {
        "parentAction": [
          "Succeeded"
        ]
      }
    },
    "branchAction2": {
      "type": "<action-type>",
      "inputs": {},
      "runAfter": {
        "parentAction": [
          "Succeeded"
        ]
      }
    }
  },
  "outputs": {}
}
```

<a name="join-branches"></a>

## <a name="join-parallel-branches"></a>Unión de ramas paralelas

Para combinar ramas paralelas, simplemente agregue un paso en la parte inferior debajo de todas las ramas. Este paso se ejecuta después de que todas las ramas paralelas terminen de ejecutarse.

![Unión de ramas paralelas](media/logic-apps-control-flow-branches/join.png)

1. En [Azure Portal](https://portal.azure.com), busque y abra la aplicación lógica en el Diseñador de aplicaciones lógicas. 

1. En las ramas paralelas que desea combinar, elija **Nuevo paso**. 

   ![Adición de un paso de combinación](media/logic-apps-control-flow-branches/add-join-step.png)

1. En el cuadro de búsqueda, busque y seleccione la acción que desea como el paso que combina las ramas.

   ![Búsqueda y selección de la acción que combina ramas paralelas](media/logic-apps-control-flow-branches/join-steps.png)

   Sus ramas paralelas están ahora combinadas.

   ![Ramas combinadas](media/logic-apps-control-flow-branches/joined-branches.png)

<a name="join-json"></a>

## <a name="join-definition-json"></a>Definición de la unión (JSON)

Si está trabajando en la vista de código, puede definir la estructura de la unión en la definición JSON de la aplicación lógica en su lugar, por ejemplo:

``` json
{
  "triggers": {
    "myTrigger": { }
  },
  "actions": {
    "parentAction": {
      "type": "<action-type>",
      "inputs": { },
      "runAfter": {}
    },
    "branchAction1": {
      "type": "<action-type>",
      "inputs": { },
      "runAfter": {
        "parentAction": [
          "Succeeded"
        ]
      }
    },
    "branchAction2": {
      "type": "<action-type>",
      "inputs": { },
      "runAfter": {
        "parentAction": [
          "Succeeded"
        ]
      }
    },
    "joinAction": {
      "type": "<action-type>",
      "inputs": { },
      "runAfter": {
        "branchAction1": [
          "Succeeded"
        ],
        "branchAction2": [
          "Succeeded"
        ]
      }
    }
  },
  "outputs": {}
}
```

## <a name="get-support"></a>Obtención de soporte técnico

* Si tiene alguna duda, visite la [página de preguntas y respuestas de Microsoft sobre Azure Logic Apps](/answers/topics/azure-logic-apps.html).
* Para enviar características y sugerencias o votar las que ya se han enviado, visite el [sitio web de comentarios de los usuarios de Azure Logic Apps](https://aka.ms/logicapps-wish).

## <a name="next-steps"></a>Pasos siguientes

* [Ejecución de pasos en función de una condición (instrucciones condicionales)](../logic-apps/logic-apps-control-flow-conditional-statement.md)
* [Realización de pasos en función de los diferentes valores (instrucciones switch)](../logic-apps/logic-apps-control-flow-switch-statement.md)
* [Ejecución y repetición de pasos (bucles)](../logic-apps/logic-apps-control-flow-loops.md)
* [Ejecución de pasos en función del estado de las acciones agrupadas (ámbitos)](../logic-apps/logic-apps-control-flow-run-steps-group-scopes.md)
