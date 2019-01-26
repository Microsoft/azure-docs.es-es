---
title: Crear variables para guardar valores - Azure Logic Apps | Microsoft Docs
description: Cómo guardar y administrar valores mediante la creación de variables en Azure Logic Apps
services: logic-apps
author: ecfan
manager: jeconnoc
ms.author: estfan
ms.topic: article
ms.date: 05/30/2018
ms.service: logic-apps
ms.reviewer: klam, LADocs
ms.suite: integration
ms.openlocfilehash: bb84c7d5e483b0a2abc3b7d1a37de8760513d203
ms.sourcegitcommit: 3ab534773c4decd755c1e433b89a15f7634e088a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 01/07/2019
ms.locfileid: "54063223"
---
# <a name="create-variables-for-saving-and-managing-values-in-azure-logic-apps"></a>Crear variables para guardar y administrar valores en Azure Logic Apps

En este artículo se muestra la manera de almacenar y trabajar con valores de la aplicación lógica mediante la creación de variables. Por ejemplo, las variables pueden ayudarle a contar el número de veces que se ejecuta un bucle. Al iterar una matriz o al comprobar una matriz para buscar un elemento específico, puede usar una variable para hacer referencia al número de índice de cada elemento de la matriz. 

Puede crear variables para tipos de datos como, por ejemplo, "entero", "float", "booleano", "cadena", "matriz" y "objeto". Después de crear una variable, puede realizar otras tareas; por ejemplo:

* Obtener o hacer referencia al valor de la variable.
* Aumentar o disminuir la variable por un valor constante, también conocido como *incremento* y *decremento*.
* Asignar un valor diferente a la variable.
* Insertar o *anexar* el valor de la variable como la última hora en una cadena o una matriz.

Las variables solo existen y son globales en la instancia de la aplicación lógica que las crea. Además, persisten en todas las iteraciones de bucle de una instancia de esa aplicación lógica. Cuando se hace referencia a una variable, debe usar el nombre de la variable como token, y no el nombre de la acción; este último se usa de forma habitual para hacer referencia a los resultados de la acción. 

> [!IMPORTANT]
> De forma predeterminada, los ciclos de un bucle "Foreach" se ejecutan en paralelo. Cuando se usan variables en bucles, ejecute el bucle [secuencialmente](../logic-apps/logic-apps-control-flow-loops.md#sequential-foreach-loop) para que las variables devuelvan resultados predecibles. 

Si aún no tiene ninguna suscripción de Azure, <a href="https://azure.microsoft.com/free/" target="_blank">regístrese para obtener una cuenta gratuita de Azure</a>. 

## <a name="prerequisites"></a>Requisitos previos

Para poder seguir los pasos de este artículo, estos son los elementos que necesita:

* La aplicación lógica en la que desea crear una variable. 

  Si no está familiarizado con las aplicaciones lógicas, consulte [¿Qué es Azure Logic Apps?](../logic-apps/logic-apps-overview.md) e [Inicio rápido: Creación de la primera aplicación lógica](../logic-apps/quickstart-create-first-logic-app-workflow.md).

* Un [desencadenador](../logic-apps/logic-apps-overview.md#logic-app-concepts) que será el primer paso de la aplicación lógica. 

  Antes de poder agregar acciones para crear y trabajar con variables, debe iniciar la aplicación lógica con un desencadenador.

<a name="create-variable"></a>

## <a name="initialize-variable"></a>Inicializar la variable

Puede crear una variable y declarar el tipo de datos y el valor inicial de la misma; todo mediante una sola acción de la aplicación lógica. Solo se pueden declarar variables en el nivel global, y no en los ámbitos, condiciones y bucles. 

1. En <a href="https://portal.azure.com" target="_blank">Azure Portal</a> o Visual Studio, abra la aplicación lógica en el diseñador de aplicaciones lógicas. 

   En este ejemplo se usa Azure Portal y una aplicación lógica con un desencadenador existente.

2. En la aplicación lógica, en el paso donde quiere agregar una variable, siga uno de estos pasos: 

   * Para agregar una acción en el último paso, elija **Nuevo paso** > **Agregar una acción**.

     ![Agregar una acción](./media/logic-apps-create-variables-store-values/add-action.png)

   * Para agregar una acción entre los pasos existentes, mueva el mouse sobre la flecha de conexión para que el signo más (+) aparezca. 
   Haga clic en el signo más y, a continuación, elija **Agregar una acción**.

3. En el cuadro de búsqueda, escriba "variables" como filtro. En la lista de acciones, seleccione **Variables: Inicializar variable**.

   ![Acción Select](./media/logic-apps-create-variables-store-values/select-initialize-variable-action.png)

4. Proporcione esta información para la variable:

   | Propiedad | Obligatorio | Valor |  DESCRIPCIÓN |
   |----------|----------|-------|--------------|
   | Name | SÍ | <*variable-name*> | El nombre de la variable que se va a incrementar | 
   | Type | SÍ | <*variable-type*> | El tipo de datos de la variable | 
   | Value | Sin  | <*start-value*> | Valor inicial de la variable <p><p>**Sugerencia**: aunque es opcional, establezca este valor como procedimiento recomendado para que sepa siempre el valor inicial de la variable. | 
   ||||| 

   ![Inicializar la variable](./media/logic-apps-create-variables-store-values/initialize-variable.png)

5. Ahora continúe agregando las acciones que quiera. Cuando esté listo, elija **Guardar** en la barra de herramientas del diseñador.

Si cambia del diseñador al editor de la vista de código, esta es la manera en que la acción **Inicializar variable** aparece en la definición de la aplicación lógica, que se encuentra en formato de notación de objetos JavaScript (JSON):

```json
"actions": {
   "Initialize_variable": {
      "type": "InitializeVariable",
      "inputs": {
         "variables": [ {
               "name": "Count",
               "type": "Integer",
               "value": 0
          } ]
      },
      "runAfter": {}
   }
},
```

Aquí tiene ejemplos de otros tipos de variable:

*Variable de cadena*

```json
"actions": {
   "Initialize_variable": {
      "type": "InitializeVariable",
      "inputs": {
         "variables": [ {
               "name": "myStringVariable",
               "type": "String",
               "value": "lorem ipsum"
          } ]
      },
      "runAfter": {}
   }
},
```

*Variable booleana*

```json
"actions": {
   "Initialize_variable": {
      "type": "InitializeVariable",
      "inputs": {
         "variables": [ {
               "name": "myBooleanVariable",
               "type": "Boolean",
               "value": false
          } ]
      },
      "runAfter": {}
   }
},
```

*Matriz con enteros*

```json
"actions": {
   "Initialize_variable": {
      "type": "InitializeVariable",
      "inputs": {
         "variables": [ {
               "name": "myArrayVariable",
               "type": "Array",
               "value": [1, 2, 3]
          } ]
      },
      "runAfter": {}
   }
},
```

*Matriz con cadenas*

```json
"actions": {
   "Initialize_variable": {
      "type": "InitializeVariable",
      "inputs": {
         "variables": [ {
               "name": "myArrayVariable",
               "type": "Array",
               "value": ["red", "orange", "yellow"]
          } ]
      },
      "runAfter": {}
   }
},
```

<a name="get-value"></a>

## <a name="get-the-variables-value"></a>Obtener el valor de la variable

Para recuperar o hacer referencia al contenido de una variable, también puede usar la [función variables()](../logic-apps/workflow-definition-language-functions-reference.md#variables) del Diseñador de la aplicación lógica y el editor de la vista de código.
Cuando se hace referencia a una variable, debe usar el nombre de la variable como token, y no el nombre de la acción; este último se usa de forma habitual para hacer referencia a los resultados de la acción. 

Por ejemplo, esta expresión obtiene los elementos de la variable de la matriz [que se creó anteriormente en este artículo](#append-value) mediante la función **variables()**. La función **string()** devuelve el contenido de la variable en formato de cadena: `"1, 2, 3, red"`

```json
@{string(variables('myArrayVariable'))}
```

<a name="increment-value"></a>

## <a name="increment-variable"></a>Incrementar variable 

Para aumentar o *incrementar* una variable con un valor constante, agregue la acción **Variables: incrementar variable** a la aplicación lógica. Esta acción solo funciona con las variables de enteros y flotantes.

1. En el Diseñador de la aplicación de lógica, en el paso donde quiera aumentar una variable existente, elija **Nuevo paso** > **Agregar una acción**. 

   Por ejemplo, esta aplicación lógica ya tiene un desencadenador y una acción que creó una variable. Por lo tanto, agregue una nueva acción según estos pasos:

   ![Agregar una acción](./media/logic-apps-create-variables-store-values/add-increment-variable-action.png)

   Para agregar una acción entre pasos existentes, mueva el mouse sobre la flecha de conexión para que el signo más (+) aparezca. Haga clic en el signo más y, a continuación, elija **Agregar una acción**.

2. En el cuadro de búsqueda, escriba "incrementar variable" como filtro. En la lista de acciones, seleccione **Variables: Incrementar variable**.

   ![Seleccione la acción "Incrementar variable".](./media/logic-apps-create-variables-store-values/select-increment-variable-action.png)

3. Proporcione esta información para la incrementar la variable:

   | Propiedad | Obligatorio | Valor |  DESCRIPCIÓN |
   |----------|----------|-------|--------------|
   | Name | SÍ | <*variable-name*> | El nombre de la variable que se va a incrementar | 
   | Value | Sin  | <*increment-value*> | El valor que se usa para incrementar la variable. El valor predeterminado es uno. <p><p>**Sugerencia**: aunque es opcional, establezca este valor como procedimiento recomendado para que sepa siempre el valor específico para incrementar la variable. | 
   |||| 

   Por ejemplo:  
   
   ![Ejemplo del valor incremental](./media/logic-apps-create-variables-store-values/increment-variable-action-information.png)

4. Cuando esté listo, elija **Guardar** en la barra de herramientas del diseñador. 

Si cambia del diseñador al editor de la vista de código, esta es la manera en que la acción **Incrementar variable** aparece en la definición de la aplicación lógica, que se encuentra en formato JSON:

```json
"actions": {
   "Increment_variable": {
      "type": "IncrementVariable",
      "inputs": {
         "name": "Count",
         "value": 1
      },
      "runAfter": {}
   }
},
```

## <a name="example-create-loop-counter"></a>Ejemplo: Creación de un contador de bucle

Las variables se usan habitualmente para contar el número de veces que se ejecuta un bucle. En este ejemplo se muestra cómo crear y usar variables para esta tarea, mediante la creación de un bucle que cuente los datos adjuntos de un correo electrónico.

1. En Azure Portal, cree una aplicación lógica en blanco. Agregue un desencadenador que compruebe si hay correos electrónicos y datos adjuntos nuevos. 

   En este ejemplo usaremos el desencadenador de Office 365 Outlook **cuando llegue un nuevo correo electrónico**. 
   Puede configurar este desencadenador para que solo active cuando el correo electrónico tenga datos adjuntos.
   Sin embargo, puede usar cualquier conector que compruebe si hay correos electrónicos con datos adjuntos nuevos; por ejemplo, el conector de Outlook.com.

2. En el desencadenador, elija **Mostrar opciones avanzadas**. Para que el desencadenador compruebe si hay datos adjuntos y los pase al flujo de trabajo de la aplicación lógica, seleccione **Sí** en estas propiedades:
   
   * **Tiene datos adjuntos** 
   * **Include Attachments** (Incluir datos adjuntos) 

   ![Buscar e incluir datos adjuntos](./media/logic-apps-create-variables-store-values/check-include-attachments.png)

3. Agregue la acción [**Inicializar variable**](#create-variable). Cree una variable de entero denominada **Recuento** con un valor de inicio de cero.

   ![Agregar la acción "Inicializar variable"](./media/logic-apps-create-variables-store-values/initialize-variable.png)

4. Para desplazarse a través de los datos adjuntos, agregue un bucle *Para cada uno*; para ello, elija **Nuevo paso** > **Más** > **Agregar Para cada uno**.

   ![Agregar un bucle "Para cada uno"](./media/logic-apps-create-variables-store-values/add-loop.png)

5. En el bucle, haga clic en la casilla de **Select an output from previous steps** (Seleccionar una salida de los pasos anteriores). Cuando aparezca la lista de contenido dinámico, elija **Datos adjuntos**. 

   ![Seleccionar "Attachments" (Datos adjuntos)](./media/logic-apps-create-variables-store-values/select-attachments.png)

   El campo **Datos adjuntos**  pasa una matriz que tiene los datos adjuntos del correo electrónico del resultado del desencadenador al bucle.

6. En el bucle "para cada uno", seleccione **Add an action** (Agregar una acción). 

   ![Seleccione "Add an action" (Agregar una acción).](./media/logic-apps-create-variables-store-values/add-action-2.png)

7. En el cuadro de búsqueda, escriba "incrementar variable" como filtro. En la lista de acciones, seleccione **Variables: Incrementar variable**.

   > [!NOTE]
   > Asegúrese de que la acción **Incrementar variable** aparezca dentro del bucle. Si la acción aparece fuera del bucle, arrástrela al mismo.

8. En la acción **Incrementar variable** de la lista **Nombre**, seleccione la variable **Recuento**. 

   ![Seleccione la variable "Recuento".](./media/logic-apps-create-variables-store-values/add-increment-variable-example.png)

9. En el bucle, agregue cualquier acción que le envíe la cantidad de datos adjuntos. En la acción, incluya el valor de la variable **Recuento** como, por ejemplo: 

   ![Agregar una acción que envíe resultados](./media/logic-apps-create-variables-store-values/send-email-results.png)

10. Guarde la aplicación lógica. En la barra de herramientas del diseñador, haga clic en **Guardar**. 

### <a name="test-your-logic-app"></a>Comprobación de la aplicación lógica

1. Si la aplicación lógica aún no está habilitada, en el menú de aplicación lógica, elija **Introducción**. En la barra de herramientas, elija **Habilitar**. 

2. En la barra de herramientas del Diseñador de aplicación lógica, elija **Ejecutar**. Con este paso, se inicia manualmente la aplicación lógica.

3. Envíe un correo electrónico con uno o varios datos adjuntos a la cuenta de correo electrónico que usó en este ejemplo.

   En este paso se activa el desencadenador de la aplicación lógica, que crea y ejecuta una instancia para el flujo de trabajo de la aplicación lógica.
   Como resultado, la aplicación lógica envía un mensaje o un correo electrónico que muestra el número de datos adjuntos del correo electrónico que envió.

Si cambia del diseñador al editor de la vista de código, esta es la manera en que el bucle "para cada uno" aparece con la acción **Incrementar variable** en la definición de la aplicación lógica, que se encuentra en formato JSON.

```json
"actions": {
   "For_each": {
      "type": "Foreach",
      "actions": {
         "Increment_variable": {
           "type": "IncrementVariable",
            "inputs": {
               "name": "Count",
               "value": 1
            },
            "runAfter": {}
         }
      },
      "foreach": "@triggerBody()?['Attachments']",
      "runAfter": {
         "Initialize_variable": [ "Succeeded" ]
      }
   }
},
```

<a name="decrement-value"></a>

## <a name="decrement-variable"></a>Reducir variable

Para disminuir o *reducir* una variable con un valor constante, siga los pasos para [aumentar una variable](#increment-value); en este caso encontrará y deberá seleccionar la acción **Variables: reducir variable**. Esta acción solo funciona con las variables de enteros y flotantes.

Estas son las propiedades para la acción **Reducir variable**:

| Propiedad | Obligatorio | Valor |  DESCRIPCIÓN |
|----------|----------|-------|--------------|
| Name | SÍ | <*variable-name*> | El nombre de la variable que se va a reducir | 
| Value | Sin  | <*increment-value*> | El valor para reducir la variable. El valor predeterminado es uno. <p><p>**Sugerencia**: aunque es opcional, establezca este valor como procedimiento recomendado para que sepa siempre el valor específico para reducir la variable. | 
||||| 

Si cambia del diseñador al editor de la vista de código, esta es la manera en que la acción **Reducir variable** aparece en la definición de la aplicación lógica, que se encuentra en formato JSON.

```json
"actions": {
   "Decrement_variable": {
      "type": "DecrementVariable",
      "inputs": {
         "name": "Count",
         "value": 1
      },
      "runAfter": {}
   }
},
```


<a name="assign-value"></a>

## <a name="set-variable"></a>Establecer la variable

Para asignar un valor diferente a una variable existente, siga los pasos para [aumentar una variable](#increment-value), pero tendrá que realizar lo siguiente: 

1. Buscar y seleccionar la acción **Variables: establecer variable**. 

2. Proporcionar el nombre de la variable y el valor que quiere asignar. Tanto el nuevo valor como la variable deben tener los mismos tipos de datos.
El valor es necesario porque esta acción no tiene un valor predeterminado. 

Estas son las propiedades para la acción **Establecer variable**:

| Propiedad | Obligatorio | Valor |  DESCRIPCIÓN | 
|----------|----------|-------|--------------| 
| Name | SÍ | <*variable-name*> | El nombre de la variable que se va a establecer | 
| Value | SÍ | <*new-value*> | El valor al que quiere asignar la variable. Ambos deben tener los mismos tipos de datos. | 
||||| 

> [!NOTE]
> A menos que esté incrementando o reduciendo variables, el cambio de variables dentro de los bucles *podría*  crear resultados inesperados, porque los bucles se ejecutan en paralelo o de forma simultánea, de manera predeterminada. Para estos casos, intente configurar el bucle para que se ejecute secuencialmente. Por ejemplo, cuando quiera hacer referencia al valor de la variable dentro del bucle y esperar el mismo valor al inicio y al final de esa instancia de bucle, siga estos pasos para cambiar la forma en que se ejecuta el bucle: 
>
> 1. En la esquina superior derecha del bucle, elija el botón de los puntos suspensivos .(...) y, a continuación, elija **Configuración**.
> 
> 2. En **Control de simultaneidad**, cambie la opción **Invalidar el valor predeterminado** en **Activado**.
>
> 3. Arrastre el control deslizante **Degree of Parallelism** (Grado de paralelismo) a **1**.

Si cambia del diseñador al editor de la vista de código, esta es la manera en que la acción **Establecer variable** aparece en la definición de la aplicación lógica, que se encuentra en formato JSON. En este ejemplo se cambia el valor actual de la variable "Recuento" a otro valor. 

```json
"actions": {
   "Initialize_variable": {
      "type": "InitializeVariable",
      "inputs": {
         "variables": [ {
               "name": "Count",
               "type": "Integer",
               "value": 0
          } ]
      },
      "runAfter": {}
   },
   "Set_variable": {
      "type": "SetVariable",
      "inputs": {
         "name": "Count",
         "value": 100
      },
      "runAfter": {
         "Initialize_variable": [ "Succeeded" ]
      }
   }
},
```

<a name="append-value"></a>

## <a name="append-to-variable"></a>Anexar a la variable

En cuanto a las variables que almacenan cadenas o matrices, puede insertar o *anexar* el valor de una variable como el último elemento de esas matrices o cadenas. Puede seguir los pasos para [aumenta una variable](#increment-value) salvo que en su lugar, deberá realizar lo siguiente: 

1. Busque y seleccione una de estas acciones en función de si la variable es una cadena o una matriz: 

  * **Variables: anexar a la variable de cadena**
  * **Variables: anexar a la variable de matriz** 

2. Proporcione el valor que se anexa como el último elemento de la cadena o matriz. Este valor es necesario. 

Estas son las propiedades para la acción **Anexar a...**:

| Propiedad | Obligatorio | Valor |  DESCRIPCIÓN | 
|----------|----------|-------|--------------| 
| Name | SÍ | <*variable-name*> | El nombre de la variable que se va a establecer | 
| Value | SÍ | <*append-value*> | El valor que quiere anexar, que puede ser de cualquier tipo. | 
|||||  

Si cambia del diseñador al editor de la vista de código, esta es la manera en que la acción **Anexar a la variable de la matriz** aparece en la definición de la aplicación lógica, que se encuentra en formato JSON.
En este ejemplo se crea una variable de matriz y se agrega otro valor como el último elemento de la matriz. El resultado es una variable actualizada que contiene esta matriz: `[1,2,3,"red"]`. 

```json
"actions": {
   "Initialize_variable": {
      "type": "InitializeVariable",
      "inputs": {
         "variables": [ {
            "name": "myArrayVariable",
            "type": "Array",
            "value": [1, 2, 3]
         } ]
      },
      "runAfter": {}
   },
   "Append_to_array_variable": {
      "type": "AppendToArrayVariable",
      "inputs": {
         "name": "myArrayVariable",
         "value": "red"
      },
      "runAfter": {
        "Initialize_variable": [ "Succeeded" ]
      }
   }
},
```

## <a name="get-support"></a>Obtención de soporte técnico

* Si tiene alguna duda, visite el [foro de Azure Logic Apps](https://social.msdn.microsoft.com/Forums/en-US/home?forum=azurelogicapps).
* Para enviar ideas sobre características o votar sobre ellas, visite el [sitio de comentarios de los usuarios de Logic Apps](https://aka.ms/logicapps-wish).

## <a name="next-steps"></a>Pasos siguientes

* Obtenga más información sobre los [conectores de Logic Apps](../connectors/apis-list.md)
