---
title: Agregar y ejecutar fragmentos de código - Azure Logic Apps
description: Agregar y ejecutar fragmentos de código con código en línea en Azure Logic Apps
services: logic-apps
ms.service: logic-apps
ms.suite: integration
author: ecfan
ms.author: estfan
ms.reviewer: derek1ee, LADocs
ms.topic: article
ms.date: 05/14/2019
ms.openlocfilehash: 0bfa98396ee3afb80b486a5a17959664dfbe603c
ms.sourcegitcommit: 6ea7f0a6e9add35547c77eef26f34d2504796565
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/14/2019
ms.locfileid: "65602121"
---
# <a name="add-and-run-code-snippets-by-using-inline-code-in-azure-logic-apps"></a>Agregar y ejecutar fragmentos de código mediante el uso de código en línea en Azure Logic Apps

Cuando desea ejecutar un fragmento de código dentro de la aplicación lógica, puede agregar la integrada **código alineado** acción como un paso en el flujo de trabajo de la aplicación lógica. Esta acción funciona mejor cuando desea ejecutar código que se adapte a este escenario:

* Se ejecuta en JavaScript. Lenguajes más próximamente.
* Termina de ejecutarse en cinco segundos o menos.
* Controla los datos de hasta 50 MB de tamaño.
* Usa Node.js versión 8.11.1. Para obtener más información, consulte [objetos integrados estándares](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects). 

  > [!NOTE]
  > La función require() no es compatible con la **código alineado** acción para la ejecución de JavaScript.

Esta acción ejecuta el fragmento de código y devuelve el resultado de ese fragmento de código como un token denominado **resultado**, que puede usar en las acciones posteriores en la aplicación lógica. Para otros escenarios donde desea crear una función para el código, intente [creando y llamando a una función de Azure](../logic-apps/logic-apps-azure-functions.md) en la aplicación lógica.

En este artículo, los desencadenadores de aplicación de lógica de ejemplo, cuando un nuevo correo electrónico llega en una cuenta de Office 365 Outlook. El fragmento de código extrae y devuelve todas las direcciones de correo electrónico que aparecen en el cuerpo del correo electrónico.

![Información general de ejemplo](./media/logic-apps-add-run-inline-code/inline-code-example-overview.png)

## <a name="prerequisites"></a>Requisitos previos

* Una suscripción de Azure. Si no tiene una suscripción de Azure, [regístrese para obtener una cuenta gratuita de Azure](https://azure.microsoft.com/free/).

* La aplicación lógica donde desea agregar el fragmento de código, incluido un desencadenador. Si no tiene una aplicación lógica, consulte [inicio rápido: Creación de la primera aplicación lógica](../logic-apps/quickstart-create-first-logic-app-workflow.md).

   El ejemplo de aplicación lógica en este tema usa este desencadenador de Office 365 Outlook: **Cuando llega un nuevo correo electrónico**

* Un [cuenta de integración](../logic-apps/logic-apps-enterprise-integration-create-integration-account.md) que esté vinculado a la aplicación lógica

## <a name="add-inline-code"></a>Agregue el código en línea

1. Si no lo ha hecho ya, en el [portal Azure](https://portal.azure.com), abra la aplicación lógica en el Diseñador de aplicaciones lógicas.

1. En el diseñador, agregue el **código alineado** acción en la ubicación que desee en el flujo de trabajo de la aplicación lógica.

   * Para agregar la acción al final del flujo de trabajo, elija **nuevo paso**.

   * Para agregar la acción entre pasos existentes, mueva el puntero del mouse sobre la flecha que conecta estos pasos. Seleccione el signo más (**+**) y seleccione **agregar una acción**.

   Este ejemplo se agrega el **código alineado** acción en el desencadenador de Office 365 Outlook.

   ![Agregar nuevo paso](./media/logic-apps-add-run-inline-code/add-new-step.png)

1. En **elegir una acción**, en el cuadro de búsqueda, escriba "en línea código" como filtro. En la lista de acciones, seleccione esta acción: **Ejecutar el código de JavaScript**

   ![Seleccione "Ejecutar código de JavaScript"](./media/logic-apps-add-run-inline-code/select-inline-code-action.png)

   La acción aparece en el diseñador y contiene el código de ejemplo de forma predeterminada, incluyendo una instrucción return.

   ![Acción de código en línea con el código de ejemplo predeterminado](./media/logic-apps-add-run-inline-code/inline-code-action-default.png)

1. En el **código** cuadro, elimine el código de ejemplo y escriba el código que desea ejecutar. Escribir código que se colocaría dentro de un método, pero sin definir la firma del método. 

   Cuando se escribe una palabra clave conocida, aparece la lista de Autocompletar para que puedan seleccionar desde las palabras clave disponibles, por ejemplo:

   ![Lista de Autocompletar de la palabra clave](./media/logic-apps-add-run-inline-code/auto-complete.png)

   Este fragmento de código de ejemplo crea primero una variable que almacena un *expresión regular*, que especifica un patrón de coincidencia en el texto de entrada. El código, a continuación, crea una variable que almacena los datos de cuerpo del correo electrónico desde el desencadenador.

   ![Creación de variables](./media/logic-apps-add-run-inline-code/save-email-body-variable.png)

   Para facilitar los resultados de los desencadenadores y acciones anteriores a la referencia, aparece la lista de contenido dinámico mientras el cursor se encuentra dentro de la **código** cuadro. En este ejemplo, la lista muestra los resultados disponibles desde el desencadenador, incluido el **cuerpo** token, que ahora puede seleccionar.

   Después de seleccionar el **cuerpo** token, la acción de código en línea resuelve el token para un `workflowContext` objeto que hace referencia el correo de electrónico `Body` valor de propiedad:

   ![Resultado de SELECT](./media/logic-apps-add-run-inline-code/inline-code-example-select-outputs.png)

   En el **código** , cuadro de solo lectura, puede usar el fragmento de código `workflowContext` objeto como entrada. Este objeto tiene subpropiedades que dan al código acceso a los resultados de los desencadenadores y acciones anteriores en el flujo de trabajo.
   Para obtener más información, vea esta sección más adelante en este tema: [Hacer referencia a los resultados de desencadenadores y las acciones en el código](#workflowcontext).

   > [!NOTE]
   >
   > Si el fragmento de código hace referencia a los nombres de acción que utiliza el operador punto (.), debe agregar esos nombres de acción para el [ **acciones** parámetro](#add-parameters). Esas referencias también deben incluir los nombres de acción con corchetes ([]) y las comillas, por ejemplo:
   >
   > `// Correct`</br> 
   > `workflowContext.actions["my.action.name"].body`</br>
   >
   > `// Incorrect`</br>
   > `workflowContext.actions.my.action.name.body`

   La acción de código en línea no requiere un `return` instrucción, pero los resultados de una `return` están disponibles para referencia en las acciones posteriores a través de la instrucción el **resultado** token. 
   Por ejemplo, el fragmento de código devuelve el resultado mediante una llamada a la `match()` función, que busca coincidencias en el cuerpo del correo electrónico con la expresión regular. El **Compose** acción utiliza el **resultado** tokens para hacer referencia a los resultados en la línea acción de código y crea un único resultado.

   ![Aplicación lógica terminada](./media/logic-apps-add-run-inline-code/inline-code-complete-example.png)

1. Cuando haya terminado, guarde la aplicación lógica.

<a name="workflowcontext"></a>

### <a name="reference-trigger-and-action-results-in-your-code"></a>Resultados de desencadenadores y las acciones de referencia en el código

El `workflowContext` objeto tiene esta estructura, que incluye el `actions`, `trigger`, y `workflow` subpropiedades:

```json
{
   "workflowContext": {
      "actions": {
         "<action-name-1>": @actions('<action-name-1>'),
         "<action-name-2>": @actions('<action-name-2>')
      },
      "trigger": {
         @trigger()
      },
      "workflow": {
         @workflow()
      }
   }
}
```

Esta tabla contiene más información acerca de estas subpropiedades:

| Propiedad | Escriba | DESCRIPCIÓN |
|----------|------|-------|
| `actions` | Colección de objetos | Objetos de resultado de las acciones que se ejecutan antes de que el fragmento de código se ejecuta. Cada objeto tiene un *pares clave-valor* par donde la clave es el nombre de una acción y el valor es equivalente a llamar a la [actions() función](../logic-apps/workflow-definition-language-functions-reference.md#actions) con `@actions('<action-name>')`. Nombre de acción utiliza el mismo nombre de acción que se usa en la definición de flujo de trabajo subyacente, que reemplaza los espacios ("") en el nombre de acción con caracteres de subrayado (_). Este objeto proporciona acceso a los valores de propiedad de acción de ejecutar la instancia de flujo de trabajo actual. |
| `trigger` | Object | Objeto de resultado del desencadenador y equivalente a llamar a la [trigger() función](../logic-apps/workflow-definition-language-functions-reference.md#trigger). Este objeto proporciona acceso a los valores de propiedad de desencadenador de ejecutar la instancia de flujo de trabajo actual. |
| `workflow` | Object | El objeto de flujo de trabajo y el equivalente a llamar a la [workflow() función](../logic-apps/workflow-definition-language-functions-reference.md#workflow). Este objeto proporciona acceso a los valores de propiedad de flujo de trabajo, como el nombre de flujo de trabajo, identificador de ejecución y así sucesivamente, desde ejecutar la instancia de flujo de trabajo actual. |
|||

En el ejemplo de este tema, la `workflowContext` objeto tiene estas propiedades que puede tener acceso su código:

```json
{
   "workflowContext": {
      "trigger": {
         "name": "When_a_new_email_arrives",
         "inputs": {
            "host": {
               "connection": {
                  "name": "/subscriptions/<Azure-subscription-ID>/resourceGroups/<Azure-resource-group-name>/providers/Microsoft.Web/connections/office365"
               }
            },
            "method": "get",
            "path": "/Mail/OnNewEmail",
            "queries": {
               "includeAttachments": "False"
            }
         },
         "outputs": {
            "headers": {
               "Pragma": "no-cache",
               "Content-Type": "application/json; charset=utf-8",
               "Expires": "-1",
               "Content-Length": "962095"
            },
            "body": {
               "Id": "AAMkADY0NGZhNjdhLTRmZTQtNGFhOC1iYjFlLTk0MjZlZjczMWRhNgBGAAAAAABmZwxUQtCGTqSPpjjMQeD",
               "DateTimeReceived": "2019-03-28T19:42:16+00:00",
               "HasAttachment": false,
               "Subject": "Hello World",
               "BodyPreview": "Hello World",
               "Importance": 1,
               "ConversationId": "AAQkADY0NGZhNjdhLTRmZTQtNGFhOC1iYjFlLTk0MjZlZjczMWRhNgAQ",
               "IsRead": false,
               "IsHtml": true,
               "Body": "Hello World",
               "From": "<sender>@<domain>.com",
               "To": "<recipient-2>@<domain>.com;<recipient-2>@<domain>.com",
               "Cc": null,
               "Bcc": null,
               "Attachments": []
            }
         },
         "startTime": "2019-05-03T14:30:45.971564Z",
         "endTime": "2019-05-03T14:30:50.1746874Z",
         "scheduledTime": "2019-05-03T14:30:45.8778117Z",
         "trackingId": "1cd5ffbd-f989-4df5-a96a-6e9ce31d03c5",
         "clientTrackingId": "08586447130394969981639729333CU06",
         "originHistoryName": "08586447130394969981639729333CU06",
         "code": "OK",
         "status": "Succeeded"
      },
      "workflow": {
         "id": "/subscriptions/<Azure-subscription-ID>/resourceGroups/<Azure-resource-group-name>/providers/Microsoft.Logic/workflows/<logic-app-workflow-name>",
         "name": "<logic-app-workflow-name>",
         "type": "Microsoft.Logic/workflows",
         "location": "<Azure-region>",
         "run": {
            "id": "/subscriptions/<Azure-subscription-ID>/resourceGroups/<Azure-resource-group-name>/providers/Microsoft.Logic/workflows/<logic-app-workflow-name>/runs/08586453954668694173655267965CU00",
            "name": "08586453954668694173655267965CU00",
            "type": "Microsoft.Logic/workflows/runs"
         }
      }
   }
}
```

<a name="add-parameters"></a>

## <a name="add-parameters"></a>Agregar parámetros

En algunos casos, es posible que deba que necesita de forma explícita el **código alineado** acción incluye los resultados del desencadenador o acciones específicas que el código hace referencia como dependencias agregando el **desencadenador** o **Acciones** parámetros. Esta opción es útil para escenarios donde no se encuentran los resultados que se hace referencia en tiempo de ejecución.

> [!TIP]
> Si piensa reutilizar el código, agregue referencias a las propiedades mediante el **código** cuadro para que su código incluye las referencias del token resueltas, en lugar de agregar el desencadenador o acciones como dependencias explícitas.

Por ejemplo, suponga que tiene código que hace referencia a la **SelectedOption** derivarse el **enviar correo electrónico de aprobación** acción para el conector de Office 365 Outlook. En tiempo de creación, el motor de Logic Apps analiza el código para determinar si ha hace referencia a cualquier desencadenador o acción da como resultado e incluye los resultados automáticamente. En tiempo de ejecución debe obtendrá un error que el resultado de acción o desencadenador que se hace referencia no está disponible en la instancia especificada `workflowContext` objeto, puede agregar el desencadenador o acción como una dependencia explícita. En este ejemplo, se agrega el **acciones** parámetro y especificar que el **código alineado** acción incluir explícitamente el resultado de la **enviar correo electrónico de aprobación** acción.

Para agregar estos parámetros, abra el **Agregar nuevo parámetro** enumerar y seleccionar los parámetros deseados:

   ![Agregar parámetros](./media/logic-apps-add-run-inline-code/inline-code-action-add-parameters.png)

   | Parámetro | DESCRIPCIÓN |
   |-----------|-------------|
   | **Acciones** | Incluir resultados de las acciones anteriores. Consulte [incluir resultados de acción](#action-results). |
   | **Desencadenador** | Incluir resultados desde el desencadenador. Consulte [incluir resultados de desencadenador](#trigger-results). |
   |||

<a name="trigger-results"></a>

### <a name="include-trigger-results"></a>Incluir resultados de desencadenador

Si selecciona **desencadenadores**, se le pedirá si incluir desencadenador da como resultado.

* Desde el **desencadenador** lista, seleccione **Sí**.

<a name="action-results"></a>

### <a name="include-action-results"></a>Incluir resultados de acción

Si selecciona **acciones**, se le pedirá para las acciones que desea agregar. Sin embargo, antes de empezar a agregar acciones, necesitará la versión del nombre de acción que aparece en la definición de flujo de trabajo subyacente de la aplicación lógica.

* Esta funcionalidad no es compatible con variables, bucles y los índices de la iteración.

* Los nombres de definición de flujo de trabajo de la aplicación lógica use un carácter de subrayado (_), no un espacio.

* Para los nombres de acción que utiliza el operador punto (.), incluya esos operadores, por ejemplo:

  `My.Action.Name`

1. En la barra de herramientas del diseñador, elija **vista código**y buscar dentro de la `actions` atributo para el nombre de acción.

   Por ejemplo, `Send_approval_email_` es el nombre JSON para el **enviar correo electrónico de aprobación** acción.

   ![Buscar nombre de acción en JSON](./media/logic-apps-add-run-inline-code/find-action-name-json.png)

1. Para volver a la vista del diseñador, en la barra de herramientas de la vista de código, elija **diseñador**.

1. Para agregar la primera acción, en el **acciones elemento - 1** , escriba un nombre de la acción de JSON.

   ![Escriba la primera acción](./media/logic-apps-add-run-inline-code/add-action-parameter.png)

1. Para agregar otra acción, elija **Agregar nuevo elemento**.

## <a name="reference"></a>Referencia

Para obtener más información sobre la **ejecutar código de JavaScript** la acción de la estructura y sintaxis de definición de flujo de trabajo subyacente de la aplicación lógica mediante el lenguaje de definición de flujo de trabajo, consulte esta acción [sección referencia ](../logic-apps/logic-apps-workflow-actions-triggers.md#run-javascript-code).

## <a name="next-steps"></a>Pasos siguientes

Obtenga más información sobre [conectores para Azure Logic Apps](../connectors/apis-list.md)
