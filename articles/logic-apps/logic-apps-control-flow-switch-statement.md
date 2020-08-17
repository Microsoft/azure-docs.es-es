---
title: Adición de instrucciones switch a flujos de trabajo
description: Cómo crear instrucciones switch que controlen las acciones de los flujos de trabajo en función de valores específicos en Azure Logic Apps
services: logic-apps
ms.suite: integration
ms.reviewer: klam, logicappspm
ms.topic: article
ms.date: 10/08/2018
ms.openlocfilehash: d2c8271f0bc33bc47e64c85846061a9b77bbe66f
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87066227"
---
# <a name="create-switch-statements-that-run-workflow-actions-based-on-specific-values-in-azure-logic-apps"></a>Crear instrucciones switch que controlen las acciones de los flujos de trabajo en función de valores específicos en Azure Logic Apps

Para ejecutar acciones específicas en función de los valores de objetos, expresiones o tokens, agregue una instrucción *switch*. En esta estructura se evalúa el objeto, la expresión o el token, se elige el caso que coincida con el resultado y se ejecutan acciones específicas para ese caso. Cuando se ejecuta la instrucción switch, solo un caso debe coincidir con el resultado.

Por ejemplo, imagine que desea una aplicación lógica que lleve a cabo pasos diferentes en función de una opción seleccionada en el correo electrónico. En este ejemplo, la aplicación lógica comprueba la fuente RSS de un sitio web para ver si hay nuevo contenido. Cuando aparece un elemento nuevo en la fuente RSS, la aplicación lógica envía un correo electrónico a un aprobador. Dependiendo de si el aprobador selecciona "Aprobar" o "Rechazar", la aplicación lógica sigue pasos diferentes.

> [!TIP]
> Al igual que todos los lenguajes de programación, la instrucción switch solo admite los operadores de igualdad. Utilice una [instrucción condicional](../logic-apps/logic-apps-control-flow-conditional-statement.md) si necesita otros operadores relacionales, como "mayor que".
> Para garantizar el comportamiento de ejecución determinístico, los casos tienen que contener un valor único y estático en lugar de expresiones o tokens dinámicos.

## <a name="prerequisites"></a>Prerrequisitos

* Suscripción a Azure. Si aún no tiene una, [regístrese para obtener una cuenta de Azure gratuita](https://azure.microsoft.com/free/).

* Para seguir el ejemplo de este artículo, [cree una aplicación lógica de ejemplo](../logic-apps/quickstart-create-first-logic-app-workflow.md) con una cuenta de Outlook.com o de Office 365 Outlook.

  1. Al agregar la acción para enviar correo electrónico, encuentre y seleccione esta acción en su lugar: **Enviar un correo electrónico de aprobación**

     ![Selección de "Enviar correo electrónico de aprobación"](./media/logic-apps-control-flow-switch-statement/send-approval-email-action.png)

  1. Proporcione los campos necesarios, como la dirección de correo electrónico de la persona que recibe el correo electrónico de aprobación. 
  En **Opciones de usuario**, escriba "Aprobar, Rechazar".

     ![Escriba los detalles del correo electrónico](./media/logic-apps-control-flow-switch-statement/send-approval-email-details.png)

## <a name="add-switch-statement"></a>Incorporación de la instrucción switch

1. En este ejemplo, agregue una instrucción switch al final del flujo de trabajo de ejemplo. Después del último paso, elija **Nuevo paso**.

   Si desea agregar una instrucción switch entre los pasos, mueva el puntero sobre la flecha donde desea agregar la instrucción switch. Elija el **signo más** ( **+** ) que aparece y, luego, elija **Agregar una acción**.

1. En el cuadro de búsqueda, escriba "switch" como filtro. Seleccione esta acción: **Switch - Control**

   ![Agregar switch](./media/logic-apps-control-flow-switch-statement/add-switch-statement.png)

   Aparece una instrucción switch con un caso y un caso predeterminado. 
   De forma predeterminada una instrucción switch requiere al menos un caso además del caso predeterminado. 

   ![Instrucción switch predeterminada vacía](./media/logic-apps-control-flow-switch-statement/empty-switch.png)

1. Haga clic en el cuadro **En** para que aparezca la lista de contenido dinámico. En la lista seleccione el campo **SelectedOption** cuyo resultado determina la acción que se va a realizar. 

   ![Seleccionar "SelectedOption"](./media/logic-apps-control-flow-switch-statement/select-selected-option.png)

1. Para controlar los casos donde el aprobador selecciona `Approve` o `Reject`, agregue otro caso entre **Caso** y **Predeterminado**. 

   ![Agregar otro caso](./media/logic-apps-control-flow-switch-statement/switch-plus.png)

1. Agregue estas acciones a los casos correspondientes:

   | Case nº | **SelectedOption** | Acción |
   |--------|--------------------|--------|
   | Caso 1 | **Aprobar** | Agregar la acción de Outlook **Enviar un correo electrónico** para enviar detalles sobre el elemento RSS únicamente cuando el aprobador ha seleccionado **Aprobar**. |
   | Caso 2 | **Rechazar** | Agregar la acción de Outlook **Enviar un correo electrónico** para notificar a otros aprobadores que el elemento RSS se ha rechazado. |
   | Valor predeterminado | None | No es necesaria ninguna acción. En este ejemplo, el caso **Predeterminado** está vacío porque **SelectedOption** solo tiene dos opciones. |
   |||

   ![Instrucción switch terminada](./media/logic-apps-control-flow-switch-statement/finished-switch.png)

1. Guarde la aplicación lógica. 

   Para probar manualmente este ejemplo, elija **Ejecutar** hasta que la aplicación lógica encuentra un nuevo elemento RSS y envía un correo electrónico de aprobación. 
   Seleccione **Aprobar** para observar los resultados.

## <a name="json-definition"></a>Definición JSON

Ahora que ha creado una aplicación lógica mediante una instrucción switch, echemos un vistazo a la definición de código de alto nivel detrás de la instrucción switch.

``` json
"Switch": {
   "type": "Switch",
   "expression": "@body('Send_approval_email')?['SelectedOption']",
   "cases": {
      "Case": {
         "actions": {
           "Send_an_email": {}
         },
         "case" : "Approve"
      },
      "Case_2": {
         "actions": {
           "Send_an_email_2": {}
         },
         "case": "Reject"
      }
   },
   "default": {
      "actions": {}
   },
   "runAfter": {
      "Send_approval_email": [
         "Succeeded"
      ]
   }
}
```

| Etiqueta | Descripción |
|-------|-------------|
| `"Switch"`         | El nombre de la instrucción switch, que puede cambiar para mejorar la legibilidad |
| `"type": "Switch"` | Especifica que la acción es una instrucción switch |
| `"expression"`     | En este ejemplo, especifica la opción seleccionada del aprobador que se evalúa con respecto a cada caso declarado más adelante en la definición |
| `"cases"` | Define cualquier número de casos. Para cada caso, `"Case_*"` es el nombre predeterminado del caso que puede cambiar para mejorar la legibilidad |
| `"case"` | Especifica el valor del caso, que tiene que ser un valor único y constante que la instrucción switch usa para comparar. Si no hay casos que coincidan con el resultado de la instrucción switch, se ejecutan las acciones de la sección `"default"`. | 
| | | 

## <a name="get-support"></a>Obtención de soporte técnico

* Si tiene alguna duda, visite la [página de preguntas y respuestas de Microsoft sobre Azure Logic Apps](/answers/topics/azure-logic-apps.html).
* Para enviar características o sugerencias o votar sobre ellas, visite el [sitio de comentarios de los usuarios de Azure Logic Apps](https://aka.ms/logicapps-wish).

## <a name="next-steps"></a>Pasos siguientes

* [Ejecución de pasos en función de una condición (instrucciones condicionales)](../logic-apps/logic-apps-control-flow-conditional-statement.md)
* [Ejecución y repetición de pasos (bucles)](../logic-apps/logic-apps-control-flow-loops.md)
* [Ejecución o combinación de pasos en paralelo (ramas)](../logic-apps/logic-apps-control-flow-branches.md)
* [Ejecución de pasos en función del estado de las acciones agrupadas (ámbitos)](../logic-apps/logic-apps-control-flow-run-steps-group-scopes.md)
