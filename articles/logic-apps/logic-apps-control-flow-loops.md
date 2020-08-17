---
title: Incorporación de bucles para repetir acciones
description: Crear bucles que repiten las acciones del flujo de trabajo o procesan matrices en Azure Logic Apps
services: logic-apps
ms.suite: integration
ms.reviewer: klam, logicappspm
ms.topic: article
ms.date: 01/05/2019
ms.openlocfilehash: 8a72dff055f2733a07b6da705b66da939ad29bae
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87495614"
---
# <a name="create-loops-that-repeat-workflow-actions-or-process-arrays-in-azure-logic-apps"></a>Crear bucles que repiten las acciones del flujo de trabajo o procesan matrices en Azure Logic Apps

Para procesar una matriz en la aplicación lógica, puede crear un [bucle "Foreach"](#foreach-loop). Este bucle repite una o varias acciones en cada elemento de la matriz. Para conocer los límites del número de elementos de matriz que los bucles "Foreach" pueden procesar, consulte [Límites y configuración](../logic-apps/logic-apps-limits-and-config.md). 

Para repetir las acciones hasta que se cumpla una condición o cambie un estado, puede crear un [bucle "Until"](#until-loop). La aplicación lógica ejecuta primero todas las acciones dentro del bucle y luego comprueba la condición o el estado. Si se cumple la condición, se detiene el bucle. En caso contrario, se repite el bucle. Para conocer el número de bucles "Until" en una ejecución de aplicación lógica, consulte [Límites y configuración](../logic-apps/logic-apps-limits-and-config.md). 

> [!TIP]
> Si tiene un desencadenador que recibe una matriz y desea ejecutar un flujo de trabajo para cada elemento de matriz, puede *desagrupar* esa matriz con la propiedad de desencadenador [**SplitOn**](../logic-apps/logic-apps-workflow-actions-triggers.md#split-on-debatch). 

## <a name="prerequisites"></a>Requisitos previos

* Una cuenta y una suscripción de Azure. Si aún no tiene una, [regístrese para obtener una cuenta de Azure gratuita](https://azure.microsoft.com/free/). 

* Conocimientos básicos acerca de [cómo crear aplicaciones lógicas](../logic-apps/quickstart-create-first-logic-app-workflow.md)

<a name="foreach-loop"></a>

## <a name="foreach-loop"></a>Bucle "Foreach"

Un bucle "Foreach" repite una o varias acciones en cada elemento de la matriz y solo funciona en matrices. Estas son algunas consideraciones que debe tener en cuenta al usar bucles "Foreach":

* De forma predeterminada, las iteraciones de un bucle "Para cada uno" se ejecutan al mismo tiempo o en paralelo. Este comportamiento difiere del del [bucle **Aplicar a cada uno** de Power Automate](/power-automate/apply-to-each), donde las iteraciones ejecutan uno a la vez o secuencialmente. Sin embargo, puede [configurar iteraciones de bucles "Para cada uno" secuenciales](#sequential-foreach-loop). Por ejemplo, si desea pausar la siguiente iteración en un bucle "Para cada uno" mediante la [acción de retraso](../connectors/connectors-native-delay.md), debe establecer el bucle para que se ejecute secuencialmente.

  La excepción al comportamiento predeterminado son bucles anidados en los cuales las iteraciones siempre se ejecutan secuencialmente, no en paralelo. Para ejecutar operaciones en paralelo en elementos de un bucle anidado, cree y [llame a una aplicación lógica secundaria](../logic-apps/logic-apps-http-endpoint.md).

* Para obtener resultados predecibles de operaciones en variables durante cada iteración del bucle, ejecute esos bucles secuencialmente. Por ejemplo, cuando un bucle de ejecución simultánea finaliza, el incremento, el decremento y la anexión a las operaciones con variables devuelven resultados predecibles. Sin embargo, durante cada iteración del bucle de ejecución simultánea, es posible que estas operaciones devuelvan resultados imprevisibles. 

* Las acciones incluidas en un bucle "Foreach" utilizan la expresión [`@item()`](../logic-apps/workflow-definition-language-functions-reference.md#item) 
para hacer referencia a cada elemento de la matriz y procesarlo. Si especifica datos que no están en una matriz, se producirá un error en el flujo de trabajo de la aplicación lógica. 

Esta aplicación lógica de ejemplo envía un resumen diario de una fuente RSS de sitio web. La aplicación utiliza un bucle "Foreach" que envía un correo electrónico por cada nuevo elemento.

1. [Cree una aplicación lógica de ejemplo](../logic-apps/quickstart-create-first-logic-app-workflow.md) con una cuenta de Outlook.com o de Office 365 Outlook.

2. Entre el desencadenador RSS y la acción para enviar un correo electrónico, agregue un bucle "Foreach". 

   1. Para agregar un bucle entre un paso y otro, mueva el puntero sobre la flecha entre ellos. 
   Elija el **signo más** ( **+** ) que aparece y seleccione **Agregar una acción**.

      ![Seleccione "Add an action" (Agregar una acción).](media/logic-apps-control-flow-loops/add-for-each-loop.png)

   1. En el cuadro de búsqueda, elija **Todas**. En el cuadro de búsqueda, escriba "para cada uno" como filtro. En la lista de acciones, seleccione esta acción: **Para cada uno: control**

      ![Incorporación de un bucle "para cada uno"](media/logic-apps-control-flow-loops/select-for-each.png)

3. Ahora, compile el bucle. En **Select an output from previous steps** (Seleccionar una salida de los pasos anteriores) después de que aparezca la lista **Agregar contenido dinámico**, seleccione la matriz **Vínculos de fuente**, que es la salida del desencadenador RSS. 

   ![Seleccionar de la lista de contenido dinámico](media/logic-apps-control-flow-loops/for-each-loop-dynamic-content-list.png)

   > [!NOTE] 
   > *Solo* puede seleccionar salidas de matriz del paso anterior.

   La matriz seleccionada ahora aparece aquí:

   ![Seleccionar matriz](media/logic-apps-control-flow-loops/for-each-loop-select-array.png)

4. Para ejecutar una acción en cada elemento de matriz, arrastre la acción **Enviar un correo electrónico** al bucle. 

   La aplicación lógica podría ser similar a la de este ejemplo:

   ![Agregar pasos a un bucle "Foreach"](media/logic-apps-control-flow-loops/for-each-loop-with-step.png)

5. Guarde la aplicación lógica. Para probar manualmente la aplicación lógica, en la barra de herramientas del diseñador, elija **Ejecutar**.

<a name="for-each-json"></a>

## <a name="foreach-loop-definition-json"></a>Definición de bucle "Foreach" (JSON)

Si está trabajando en la vista de código de la aplicación lógica, puede definir el bucle `Foreach` en la definición JSON de la aplicación lógica en su lugar, por ejemplo:

``` json
"actions": {
   "myForEachLoopName": {
      "type": "Foreach",
      "actions": {
         "Send_an_email": {
            "type": "ApiConnection",
            "inputs": {
               "body": {
                  "Body": "@{item()}",
                  "Subject": "New CNN post @{triggerBody()?['publishDate']}",
                  "To": "me@contoso.com"
               },
               "host": {
                  "api": {
                     "runtimeUrl": "https://logic-apis-westus.azure-apim.net/apim/office365"
                  },
                  "connection": {
                     "name": "@parameters('$connections')['office365']['connectionId']"
                  }
               },
               "method": "post",
               "path": "/Mail"
            },
            "runAfter": {}
         }
      },
      "foreach": "@triggerBody()?['links']",
      "runAfter": {}
   }
}
```

<a name="sequential-foreach-loop"></a>

## <a name="foreach-loop-sequential"></a>Bucle "Foreach": secuencial

De forma predeterminada, los ciclos de un bucle "Foreach" se ejecutan en paralelo. Para ejecutar cada ciclo secuencialmente, establezca la opción **Secuencial** del bucle. Los bucles "Foreach" deben ejecutarse secuencialmente cuando se han anidado bucles o variables dentro de bucles en los que se esperan resultados predecibles. 

1. En la esquina superior derecha del bucle, elija **puntos suspensivos** ( **...** ) > **Configuración**.

   ![En bucle "Foreach", elija "..." > "Configuración"](media/logic-apps-control-flow-loops/for-each-loop-settings.png)

1. En **Control de simultaneidad**, establezca la opción **Control de simultaneidad** en **Activar**. Arrastre el control deslizante **Grado de paralelismo** a **1** y elija **Listo**.

   ![Activación del control de simultaneidad](media/logic-apps-control-flow-loops/for-each-loop-sequential-setting.png)

Si está trabajando con la definición de JSON de la aplicación lógica, puede usar la opción `Sequential` agregando el parámetro `operationOptions`, por ejemplo:

``` json
"actions": {
   "myForEachLoopName": {
      "type": "Foreach",
      "actions": {
         "Send_an_email": { }
      },
      "foreach": "@triggerBody()?['links']",
      "runAfter": {},
      "operationOptions": "Sequential"
   }
}
```

<a name="until-loop"></a>

## <a name="until-loop"></a>Bucle "Until"
  
Para ejecutar y repetir las acciones hasta que se cumpla una condición o cambie un estado, coloque esas acciones en un bucle "Until". La aplicación lógica ejecuta primero todas las acciones dentro del bucle y luego comprueba la condición o el estado. Si se cumple la condición, se detiene el bucle. En caso contrario, se repite el bucle.

Estos son algunos escenarios comunes en los que puede utilizar un bucle "Until":

* Llamada a un punto de conexión hasta obtener la respuesta que quiere.

* Creación de un registro en una base de datos. Espere hasta que un campo específico de ese registro sea aprobado. Continúe el procesamiento. 

A partir de las 8:00 a. m. cada día, esta aplicación lógica de ejemplo incrementa una variable hasta que su valor sea igual a 10. La aplicación lógica envía después un correo electrónico que confirma el valor actual. 

> [!NOTE]
> En estos pasos se usa Office 365 Outlook, pero puede usarse cualquier proveedor de correo electrónico que Logic Apps admita. 
> [Compruebe aquí la lista de conectores](/connectors/). Si utiliza otra cuenta de correo electrónico, los pasos generales siguen siendo los mismos pero la interfaz de usuario podría ser ligeramente distinta. 

1. Crear una aplicación lógica en blanco. En el cuadro de búsqueda del Diseñador de aplicación lógica, elija **Todas**. Busque "periodicidad". 
   En la lista de desencadenadores, seleccione este desencadenador: **Periodicidad: Programación**

   ![Incorporación del desencadenador "Periodicidad: Programación"](./media/logic-apps-control-flow-loops/do-until-loop-add-trigger.png)

1. Especifique cuando se debe activar el desencadenador estableciendo el intervalo, la frecuencia y la hora del día. Para establecer la hora, seleccione **Mostrar opciones avanzadas**.

   ![Configuración de la programación de periodicidad](./media/logic-apps-control-flow-loops/do-until-loop-set-trigger-properties.png)

   | Propiedad | Value |
   | -------- | ----- |
   | **Intervalo** | 1 | 
   | **Frecuencia** | Día |
   | **A estas horas** | 8 |
   ||| 

1. En el desencadenador, elija **New step** (Nuevo paso). 
   Busque "variables" y seleccione esta acción: **Inicializar variable: variables**

   ![Incorporación de la acción "Inicializar variable: variables"](./media/logic-apps-control-flow-loops/do-until-loop-add-variable.png)

1. Configure la variable con estos valores:

   ![Establecer propiedades de las variables](./media/logic-apps-control-flow-loops/do-until-loop-set-variable-properties.png)

   | Propiedad | Value | Descripción |
   | -------- | ----- | ----------- |
   | **Nombre** | Límite | El nombre de la variable | 
   | **Tipo** | Entero | El tipo de datos de la variable | 
   | **Valor** | 0 | El valor de inicio de la variable | 
   |||| 

1. En la acción **Inicializar variable**, elija **Nuevo paso**. 

1. En el cuadro de búsqueda, elija **Todas**. Busque "hasta" y seleccione esta acción: **Hasta: control**

   ![Incorporación del bucle "Until"](./media/logic-apps-control-flow-loops/do-until-loop-add-until-loop.png)

1. Cree la condición de salida del bucle seleccionando la variable **Limit** y el operador **is equal** (es igual a). 
   Escriba **10** como valor de comparación.

   ![Crear condición de salida para detener el bucle](./media/logic-apps-control-flow-loops/do-until-loop-settings.png)

1. Dentro del bucle, elija **Agregar una acción**. 

1. En el cuadro de búsqueda, elija **Todas**. Busque "variables" y seleccione esta acción: **Incrementar variable: variables**

   ![Agregar acción para incrementar la variable](./media/logic-apps-control-flow-loops/do-until-loop-increment-variable.png)

1. En **Nombre**, seleccione la variable **Limit**. En **Valor**, escriba "1". 

     ![Aumentar "Límite" en 1](./media/logic-apps-control-flow-loops/do-until-loop-increment-variable-settings.png)

1. Fuera, en el bucle, elija **Nuevo paso**. 

1. En el cuadro de búsqueda, elija **Todas**. 
     Incorporación de una acción que envía notificaciones por correo electrónico, por ejemplo: 

     ![Agregar una acción que envía correo electrónico](media/logic-apps-control-flow-loops/do-until-loop-send-email.png)

1. Si se le pide, inicie sesión en la cuenta de correo electrónico.

1. Establezca las propiedades de la acción de correo electrónico. Agregue la variable **Limit** al asunto. De este modo, se puede confirmar que el valor actual de la variable cumple la condición especificada, por ejemplo:

      ![Configurar las propiedades del correo electrónico](./media/logic-apps-control-flow-loops/do-until-loop-send-email-settings.png)

      | Propiedad | Value | Descripción |
      | -------- | ----- | ----------- | 
      | **To** | *\<email-address\@domain>* | La dirección de correo electrónico del destinatario. Para las pruebas, use su propia dirección de correo electrónico. | 
      | **Subject** | El valor actual de "Límite" es **Limit** | Especifique el asunto del correo electrónico. En este ejemplo, asegúrese de que incluye la variable **Limit**. | 
      | **Cuerpo** | <*email-content*> | Especifique el contenido del mensaje de correo electrónico que desea enviar. En este ejemplo, escriba cualquier texto que desee. | 
      |||| 

1. Guarde la aplicación lógica. Para probar manualmente la aplicación lógica, en la barra de herramientas del diseñador, elija **Ejecutar**.

      Después de que la lógica empiece a ejecutarse, recibirá un correo electrónico con el contenido que haya especificado:

      ![Correo electrónico recibido](./media/logic-apps-control-flow-loops/do-until-loop-sent-email.png)

## <a name="prevent-endless-loops"></a>Impedir bucles sin fin

Un bucle "Until" tiene límites predeterminados que detienen la ejecución si se produce alguna de estas condiciones:

| Propiedad | Valor predeterminado | Descripción | 
| -------- | ------------- | ----------- | 
| **Recuento** | 60 | Número máximo de bucles que se ejecutan antes de salir del bucle. El valor predeterminado es de 60 ciclos. | 
| **Tiempo de espera** | PT1H | Cantidad máxima de tiempo que se ejecuta un bucle antes de salir del bucle. El valor predeterminado es una hora y se especifica en formato ISO 8601. <p>El valor de tiempo de espera se evalúa para cada ciclo del bucle. Si cualquier acción en el bucle tarda más que el límite de tiempo de expiración, el ciclo actual no se detiene. Pero el siguiente ciclo no se inicia porque no se cumple la condición del límite. | 
|||| 

Para cambiar estos límites predeterminados, elija **Mostrar opciones avanzadas** en la forma de acción del bucle.

<a name="until-json"></a>

## <a name="until-definition-json"></a>Definición "Until" (JSON)

Si está trabajando en la vista de código de la aplicación lógica, puede definir un bucle `Until` en la definición JSON de la aplicación lógica en su lugar, por ejemplo:

``` json
"actions": {
   "Initialize_variable": {
      // Definition for initialize variable action
   },
   "Send_an_email": {
      // Definition for send email action
   },
   "Until": {
      "type": "Until",
      "actions": {
         "Increment_variable": {
            "type": "IncrementVariable",
            "inputs": {
               "name": "Limit",
               "value": 1
            },
            "runAfter": {}
         }
      },
      "expression": "@equals(variables('Limit'), 10)",
      // To prevent endless loops, an "Until" loop 
      // includes these default limits that stop the loop. 
      "limit": { 
         "count": 60,
         "timeout": "PT1H"
      },
      "runAfter": {
         "Initialize_variable": [
            "Succeeded"
         ]
      }
   }
}
```

El bucle "Until" de este ejemplo llama a un punto de conexión HTTP, que crea un recurso. El bucle se detiene cuando se devuelve el cuerpo de la respuesta HTTP con el estado `Completed`. Para impedir bucles sin fin, el bucle también se detendrá si se produce alguna de estas condiciones:

* El bucle se ha ejecutado 10 veces según lo especificado por el atributo `count`. El valor predeterminado es 60 veces. 

* El bucle se ejecutó durante dos horas según lo especificado por el atributo `timeout` en formato ISO 8601. El valor predeterminado es de una hora.
  
``` json
"actions": {
   "myUntilLoopName": {
      "type": "Until",
      "actions": {
         "Create_new_resource": {
            "type": "Http",
            "inputs": {
               "body": {
                  "resourceId": "@triggerBody()"
               },
               "url": "https://domain.com/provisionResource/create-resource",
               "body": {
                  "resourceId": "@triggerBody()"
               }
            },
            "runAfter": {},
            "type": "ApiConnection"
         }
      },
      "expression": "@equals(triggerBody(), 'Completed')",
      "limit": {
         "count": 10,
         "timeout": "PT2H"
      },
      "runAfter": {}
   }
}
```

## <a name="get-support"></a>Obtención de soporte técnico

* Si tiene alguna duda, visite la [página de preguntas y respuestas de Microsoft sobre Azure Logic Apps](/answers/topics/azure-logic-apps.html).
* Para enviar características y sugerencias o votar las que ya se han enviado, visite el [sitio web de comentarios de los usuarios de Azure Logic Apps](https://aka.ms/logicapps-wish).

## <a name="next-steps"></a>Pasos siguientes

* [Ejecución de pasos en función de una condición (instrucciones condicionales)](../logic-apps/logic-apps-control-flow-conditional-statement.md)
* [Realización de pasos en función de los diferentes valores (instrucciones switch)](../logic-apps/logic-apps-control-flow-switch-statement.md)
* [Ejecución o combinación de pasos en paralelo (ramas)](../logic-apps/logic-apps-control-flow-branches.md)
* [Ejecución de pasos en función del estado de las acciones agrupadas (ámbitos)](../logic-apps/logic-apps-control-flow-run-steps-group-scopes.md)
