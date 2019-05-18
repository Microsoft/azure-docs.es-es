---
title: Referencia de esquema de lenguaje de definición de flujo de trabajo - Azure Logic Apps
description: Guía de referencia de esquema del lenguaje de definición de flujo de trabajo en Azure Logic Apps
services: logic-apps
ms.service: logic-apps
ms.suite: integration
author: ecfan
ms.author: estfan
ms.reviewer: klam, LADocs
ms.topic: reference
ms.date: 05/13/2019
ms.openlocfilehash: 3b0ad33ea6348f24079b3c88f972437244c0bc93
ms.sourcegitcommit: 1fbc75b822d7fe8d766329f443506b830e101a5e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/14/2019
ms.locfileid: "65596765"
---
# <a name="schema-reference-for-workflow-definition-language-in-azure-logic-apps"></a>Referencia de esquema del lenguaje de definición de flujo de trabajo en Azure Logic Apps

Cuando se crea una aplicación lógica en [Azure Logic Apps](../logic-apps/logic-apps-overview.md), la aplicación lógica tiene una definición de flujo de trabajo subyacente que describe la lógica real que se ejecuta en la aplicación lógica. Esa definición de flujo de trabajo usa [JSON](https://www.json.org/) y sigue una estructura que se valida el esquema del lenguaje de definición de flujo de trabajo. Esta referencia proporciona información general acerca de esta estructura y cómo el esquema define los atributos en la definición de flujo de trabajo.

## <a name="workflow-definition-structure"></a>Estructura de definición de flujo de trabajo

Una definición de flujo de trabajo siempre incluye un desencadenador para crear instancias de la aplicación lógica, además de una o varias acciones que se ejecutan tras el desencadenador se activa.

Esta es la estructura de alto nivel de una definición de flujo de trabajo:

```json
"definition": {
  "$schema": "<workflow-definition-language-schema-version>",
  "actions": { "<workflow-action-definitions>" },
  "contentVersion": "<workflow-definition-version-number>",
  "outputs": { "<workflow-output-definitions>" },
  "parameters": { "<workflow-parameter-definitions>" },
  "staticResults": { "<static-results-definitions>" },
  "triggers": { "<workflow-trigger-definitions>" }
}
```

| Atributo | Obligatorio | DESCRIPCIÓN |
|-----------|----------|-------------|
| `definition` | Sí | El elemento inicial de la definición de flujo de trabajo. |
| `$schema` | Solo cuando se hace referencia externa a una definición de flujo de trabajo. | La ubicación del archivo de esquema JSON que describe la versión del lenguaje de definición de flujo de trabajo, que puede encontrar aquí: <p>`https://schema.management.azure.com/providers/Microsoft.Logic/schemas/2016-06-01/workflowdefinition.json`</p> |
| `actions` | No | Las definiciones de una o varias acciones que se ejecutan en tiempo de ejecución de flujo de trabajo. Para obtener más información, consulte [desencadenadores y acciones](#triggers-actions). <p><p>Máximo de acciones: 250 |
| `contentVersion` | No | El número de versión de la definición de flujo de trabajo, que es "1.0.0.0" de forma predeterminada. Para ayudar a identificar y confirmar la definición correcta al implementar un flujo de trabajo, especifique el valor que usará. |
| `outputs` | No | Las definiciones de las salidas que se devuelven desde un flujo de trabajo que se ejecute. Para obtener más información, consulte [salidas](#outputs). <p><p>Máximo de salidas: 10 |
| `parameters` | No | Las definiciones de uno o varios parámetros que pasan los datos en el flujo de trabajo. Para obtener más información, consulte [parámetros](#parameters). <p><p>Máximo de parámetros: 50 |
| `staticResults` | No | Las definiciones de uno o varios resultados estáticos devueltos por las acciones como salidas ficticias cuando resultados estáticos están habilitados en esas acciones. En cada definición de la acción, el `runtimeConfiguration.staticResult.name` atributo hace referencia a la definición correspondiente dentro de `staticResults`. Para obtener más información, consulte [resultados estáticos](#static-results). |
| `triggers` | No | Las definiciones de uno o varios desencadenadores que crean una instancia del flujo de trabajo. Puede definir más de un desencadenador, pero solo con el lenguaje de definición de flujo de trabajo, no visualmente mediante el Diseñador de aplicaciones lógicas. Para obtener más información, consulte [desencadenadores y acciones](#triggers-actions). <p><p>Máximo de desencadenadores: 10 |
||||

<a name="triggers-actions"></a>

## <a name="triggers-and-actions"></a>Desencadenadores y acciones

En una definición de flujo de trabajo, las secciones `triggers` y `actions` definen las llamadas que se producen durante la ejecución del flujo de trabajo. Para más información sobre estas secciones y la sintaxis, consulte [Desencadenadores y acciones para flujos de trabajo](../logic-apps/logic-apps-workflow-actions-triggers.md).

<a name="outputs"></a>

## <a name="outputs"></a>Salidas

En la sección `outputs`, defina los datos que puede devolver el flujo de trabajo cuando termine de ejecutarse. Por ejemplo, para realizar el seguimiento de un estado o valor específico de cada ejecución, especifique que la salida del flujo de trabajo devuelva esos datos.

> [!NOTE]
> Al responder a las solicitudes entrantes de la API REST de un servicio, no use `outputs`. En su lugar, use el tipo de acción `Response`. Para más información, consulte [Desencadenadores y acciones para flujos de trabajo](../logic-apps/logic-apps-workflow-actions-triggers.md).

Esta es la estructura general de una definición de salida:

```json
"outputs": {
  "<key-name>": {
    "type": "<key-type>",
    "value": "<key-value>"
  }
}
```

| Atributo | Obligatorio | Type | DESCRIPCIÓN |
|-----------|----------|------|-------------|
| <*key-name*> | Sí | String | El nombre de clave del valor devuelto de salida. |
| <*key-type*> | Sí | int, float, string, securestring, bool, array, JSON object | El tipo del valor devuelto de salida. |
| <*valor-de-la-clave*> | Sí | Igual que <*tipo de clave*> | El valor devuelto de salida. |
|||||

Para obtener el resultado de ejecutar un flujo de trabajo, revise el historial de ejecución y los detalles en el portal de Azure de la aplicación lógica o usar el [API de REST de flujo de trabajo](https://docs.microsoft.com/rest/api/logic/workflows). También puede pasar la salida a sistemas externos, por ejemplo, a Power BI, para crear paneles.

<a name="parameters"></a>

## <a name="parameters"></a>Parámetros

En la `parameters` sección, defina todos los parámetros de flujo de trabajo que usa la definición de flujo de trabajo en la implementación para aceptar entradas. Tanto las declaraciones de parámetros como los valores de parámetros son necesarios en la implementación. Antes de poder usar estos parámetros en otras secciones del flujo de trabajo, asegúrese de declarar todos los parámetros en las siguientes secciones. 

Esta es la estructura general de una definición de parámetro:

```json
"parameters": {
  "<parameter-name>": {
    "type": "<parameter-type>",
    "defaultValue": "<default-parameter-value>",
    "allowedValues": [ <array-with-permitted-parameter-values> ],
    "metadata": {
      "key": {
        "name": "<key-value>"
      }
    }
  }
},
```

| Atributo | Obligatorio | Type | DESCRIPCIÓN |
|-----------|----------|------|-------------|
| <*parameter-type*> | Sí | int, float, string, securestring, bool, array, JSON object, secureobject <p><p>**Nota**: Con todas las contraseñas, claves y secretos, use los tipos `securestring` y `secureobject` porque la operación `GET` no los devuelve. Para obtener más información acerca de cómo proteger los parámetros, vea [proteger la aplicación lógica](../logic-apps/logic-apps-securing-a-logic-app.md#secure-action-parameters) | El tipo del parámetro. |
| <*default-parameter-values*> | Sí | Igual que `type`. | El valor de parámetro predeterminado cuando no se especifica ningún valor al crear una instancia del flujo de trabajo. |
| <*array-with-permitted-parameter-values*> | No | Matriz | Una matriz con valores que puede aceptar el parámetro. |
| `metadata` | No | Objeto JSON | Cualquier otro detalle del parámetro, por ejemplo, el nombre o una descripción legible de la aplicación lógica o flujo o los datos de tiempo de diseño usados por Visual Studio u otras herramientas |
||||

<a name="static-results"></a>

## <a name="static-results"></a>Resultados estáticos

En el `staticResults` atributo, defina la simulación de una acción `outputs` y `status` que la acción que se devuelve cuando la acción resultado estático está activada. En la definición de la acción, el `runtimeConfiguration.staticResult.name` atributo hace referencia al nombre de la definición de resultado estático dentro de `staticResults`. Aprenda cómo puede [probar las aplicaciones lógicas con datos simulados mediante la configuración de resultados estáticos](../logic-apps/test-logic-apps-mock-data-static-results.md).

```json
"definition": {
   "$schema": "<...>",
   "actions": { "<...>" },
   "contentVersion": "<...>",
   "outputs": { "<...>" },
   "parameters": { "<...>" },
   "staticResults": {
      "<static-result-definition-name>": {
         "outputs": {
            <output-attributes-and-values-returned>,
            "headers": { <header-values> },
            "statusCode": "<status-code-returned>"
         },
         "status": "<action-status>"
      }
   },
   "triggers": { "<...>" }
}
```

| Atributo | Obligatorio | Type | DESCRIPCIÓN |
|-----------|----------|------|-------------|
| <*definición de nombres estático de resultado*> | Sí | string | El nombre de una definición estática de resultado que puede hacer referencia a una definición de la acción a través de un `runtimeConfiguration.staticResult` objeto. Para más información, consulte [Opciones de configuración del entorno en tiempo de ejecución](../logic-apps/logic-apps-workflow-actions-triggers.md#runtime-config-options). <p>Puede usar cualquier nombre único que desee. De forma predeterminada, este nombre único se anexa con un número, que se incrementa según sea necesario. |
| <*output-attributes-and-values-returned*> | Sí | Varía | Los requisitos para estos atributos varían en función de diferentes condiciones. Por ejemplo, cuando el `status` es `Succeeded`, el `outputs` atributo incluye los atributos y valores devueltos como simulacro salidas por la acción. Si el `status` es `Failed`, el `outputs` atributo incluye los `errors` atributo, que es una matriz con uno o varios errores `message` objetos que tienen información de error. |
| <*header-values*> | No | JSON | Los valores de encabezado devueltos por la acción |
| <*status-code-returned*> | Sí | String | El código de estado devuelto por la acción |
| <*estado de la acción*> | Sí | String | Estado de la acción, por ejemplo, `Succeeded` o `Failed` |
|||||

Por ejemplo, en esta definición de la acción HTTP, el `runtimeConfiguration.staticResult.name` las referencias de atributo `HTTP0` dentro de la `staticResults` atributo donde se definen las salidas ficticias para la acción. El `runtimeConfiguration.staticResult.staticResultOptions` atributo especifica que la configuración de resultado estático es `Enabled` en la acción HTTP.

```json
"actions": {
   "HTTP": {
      "inputs": {
         "method": "GET",
         "uri": "https://www.microsoft.com"
      },
      "runAfter": {},
      "runtimeConfiguration": {
         "staticResult": {
            "name": "HTTP0",
            "staticResultOptions": "Enabled"
         }
      },
      "type": "Http"
   }
},
```

La acción HTTP devuelve los resultados en el `HTTP0` definición dentro `staticResults`. En este ejemplo, para el código de estado, la salida de simulacro es `OK`. Para valores de encabezado, la salida de simulacro es `"Content-Type": "application/JSON"`. Para el estado de la acción, la salida de simulacro es `Succeeded`.

```json
"definition": {
   "$schema": "<...>",
   "actions": { "<...>" },
   "contentVersion": "<...>",
   "outputs": { "<...>" },
   "parameters": { "<...>" },
   "staticResults": {
      "HTTP0": {
         "outputs": {
            "headers": {
               "Content-Type": "application/JSON"
            },
            "statusCode": "OK"
         },
         "status": "Succeeded"
      }
   },
   "triggers": { "<...>" }
},
```

<a name="expressions"></a>

## <a name="expressions"></a>Expresiones

Con JSON, puede tener valores literales que existen en tiempo de diseño, por ejemplo:

```json
"customerName": "Sophia Owen",
"rainbowColors": ["red", "orange", "yellow", "green", "blue", "indigo", "violet"],
"rainbowColorsCount": 7
```

También puede tener valores que no existen hasta el momento de ejecución. Para representar estos valores, puede usar *expresiones*, que se evalúan en tiempo de ejecución. Una expresión es una secuencia que puede contener una o varias [funciones](#functions), [operadores](#operators), variables, valores explícitos o constantes. En la definición de flujo de trabajo, se puede usar una expresión en cualquier lugar de un valor de cadena JSON agregando delante de la expresión el prefijo de signo de arroba (\@). Al evaluar una expresión que representa un valor JSON, se extrae el cuerpo de la expresión quitando el carácter \@ y siempre da como resultado otro valor JSON.

Por ejemplo, en el caso de la propiedad `customerName` anteriormente definida, puede obtener su valor mediante la función [parameters()](../logic-apps/workflow-definition-language-functions-reference.md#parameters) en una expresión y asignar ese valor a la propiedad `accountName`:

```json
"customerName": "Sophia Owen",
"accountName": "@parameters('customerName')"
```

La *interpolación de cadena* también le permite usar varias expresiones dentro de cadenas que están encapsuladas mediante el carácter \@ y llaves ({}). Esta es la sintaxis:

```json
@{ "<expression1>", "<expression2>" }
```

El resultado siempre es una cadena, de forma que esta funcionalidad es similar a la función `concat()`, por ejemplo: 

```json
"customerName": "First name: @{parameters('firstName')} Last name: @{parameters('lastName')}"
```

Si tiene un literal de cadena que comienza por el carácter \@, coloque delante de carácter\@ otro carácter \@ como carácter de escape: \@\@

En estos ejemplos se muestra cómo se evalúan las expresiones:

| Valor JSON | Resultado |
|------------|--------|
| "Sophia Owen" | Devuelve estos caracteres: "Sophia Owen" |
| "array[1]" | Devuelve estos caracteres: "array [1]" |
| "\@\@" | Devuelve estos caracteres como una cadena de un carácter: "\@" |
| " \@" | Devuelve estos caracteres como una cadena de dos caracteres: "\@" |
|||

Para estos ejemplos, suponga que tiene que definir "myBirthMonth" es igual a "January" y "myAge" es igual al número 42:

```json
"myBirthMonth": "January",
"myAge": 42
```

Estos ejemplos muestran cómo se evalúan las expresiones siguientes:

| Expresión JSON | Resultado |
|-----------------|--------|
| "\@parameters('myBirthMonth')" | Devuelve esta cadena: "January" |
| "\@{parameters('myBirthMonth')}" | Devuelve esta cadena: "January" |
| "\@parameters('myAge')" | Devuelve este número: 42 |
| "\@{parameters('myAge')}" | Devuelve este número como una cadena: "42" |
| "My age is \@{parameters('myAge')}" | Devuelve esta cadena: "My age is 42" |
| "\@concat('My age is ', string(parameters('myAge')))" | Devuelve esta cadena: "My age is 42" |
| "My age is \@\@{parameters('myAge')}" | Devuelve esta cadena, que incluye la expresión: "My age is \@{parameters('myAge')}` |
|||

Al trabajar visualmente en el Diseñador de aplicaciones lógicas, puede crear expresiones mediante el generador de expresiones, por ejemplo:

![Diseñador de aplicaciones lógicas > Generador de expresiones](./media/logic-apps-workflow-definition-language/expression-builder.png)

Cuando haya terminado, la expresión aparece en la propiedad correspondiente en la definición de flujo de trabajo, por ejemplo, la propiedad `searchQuery` aquí:

```json
"Search_tweets": {
  "inputs": {
    "host": {
      "connection": {
        "name": "@parameters('$connections')['twitter']['connectionId']"
      }
    }
  },
  "method": "get",
  "path": "/searchtweets",
  "queries": {
    "maxResults": 20,
    "searchQuery": "Azure @{concat('firstName','', 'LastName')}"
  }
},
```

<a name="operators"></a>

## <a name="operators"></a>Operadores

En [expresiones](#expressions) y [funciones](#functions), los operadores realizan tareas específicas, como hacer referencia a una propiedad o un valor en una matriz.

| Operador | Tarea |
|----------|------|
| ' | Para usar un literal de cadena como entrada o en expresiones y funciones, encapsule la cadena solo con comillas sencillas, por ejemplo, `'<myString>'`. No use comillas dobles (""), ya que entran en conflicto con el formato JSON que rodea una expresión entera. Por ejemplo: <p>**Sí**: length('Hello') </br>**No**: length("Hello") <p>Al pasar matrices o números, no es necesario encapsular los signos de puntuación. Por ejemplo: <p>**Sí**: length([1, 2, 3]) </br>**No**: length("[1, 2, 3]") |
| [] | Para hacer referencia a un valor en una posición concreta (índice) de una matriz, use corchetes. Por ejemplo, para obtener el segundo elemento de una matriz: <p>`myArray[1]` |
| . | Para hacer referencia a una propiedad de un objeto, use el operador punto. Por ejemplo, para obtener la propiedad `name` de un objeto JSON `customer`: <p>`"@parameters('customer').name"` |
| ? | Para hacer referencia a propiedades nulas de un objeto sin un error de tiempo de ejecución, use el operador de signo de interrogación. Por ejemplo, para administrar salidas nulas desde un desencadenador, puede usar esta expresión: <p>`@coalesce(trigger().outputs?.body?.<someProperty>, '<property-default-value>')` |
|||

<a name="functions"></a>

## <a name="functions"></a>Funciones

Algunas expresiones obtienen sus valores de acciones en tiempo de ejecución que no podrían existir aún cuando la definición de flujo de trabajo empieza a ejecutarse. Para trabajar con estos valores o hacer referencia a ellos mediante expresiones, puede usar [*funciones*](../logic-apps/workflow-definition-language-functions-reference.md) proporcionadas por el lenguaje de definición de flujo de trabajo.

## <a name="next-steps"></a>Pasos siguientes

* Más información sobre [Desencadenadores y acciones para flujos de trabajo](../logic-apps/logic-apps-workflow-actions-triggers.md).
* Aprenda sobre la creación y administración de aplicaciones lógicas mediante programación con la [API REST de flujo de trabajo](https://docs.microsoft.com/rest/api/logic/workflows).
