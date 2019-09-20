---
title: 'Referencia de las funciones del Lenguaje de definición de flujo de trabajo: Azure Logic Apps y Microsoft Flow'
description: Guía de referencia de las funciones en las expresiones creadas con el Lenguaje de definición de flujo de trabajo para Azure Logic Apps y Microsoft Flow
services: logic-apps
ms.service: logic-apps
ms.suite: integration
author: ecfan
ms.author: estfan
ms.reviewer: klam, LADocs
ms.topic: reference
ms.date: 08/23/2019
ms.openlocfilehash: 484e2776d96d9beaca703f93b22c51299ccf63a7
ms.sourcegitcommit: 5f67772dac6a402bbaa8eb261f653a34b8672c3a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2019
ms.locfileid: "70208403"
---
# <a name="functions-reference-for-workflow-definition-language-in-azure-logic-apps-and-microsoft-flow"></a>Referencia de las funciones del Lenguaje de definición de flujo de trabajo en Azure Logic Apps y Microsoft Flow

En las definiciones de flujo de trabajo en [Azure Logic Apps](../logic-apps/logic-apps-overview.md) y [Microsoft Flow](https://docs.microsoft.com/flow/getting-started), algunas [expresiones](../logic-apps/logic-apps-workflow-definition-language.md#expressions) obtienen sus valores de las acciones del entorno de ejecución que puede que no existan aún cuando comienza a ejecutarse el flujo de trabajo. Para hacer referencia a estos valores o procesarlos en estas expresiones, puede usar las *funciones* que proporciona el [Lenguaje de definición de flujo de trabajo](../logic-apps/logic-apps-workflow-definition-language.md). 

> [!NOTE]
> Esta página de referencia se aplica a Azure Logic Apps y Microsoft Flow, pero aparece en la documentación de Azure Logic Apps. Aunque esta página se refiere específicamente a las aplicaciones lógicas, estas funciones son válidas tanto para flujos como para aplicaciones lógicas. Para más información acerca de las funciones y las expresiones en Microsoft Flow, consulte [Uso de expresiones en condiciones](https://docs.microsoft.com/flow/use-expressions-in-conditions).

Por ejemplo, puede calcular valores con funciones matemáticas, como la función [add()](../logic-apps/workflow-definition-language-functions-reference.md#add), cuando desee obtener la suma de números enteros y float. A continuación se muestran otras tareas de ejemplo que puede realizar con funciones:

| Tarea | Sintaxis de la función | Resultado |
| ---- | --------------- | ------ |
| Devuelve una cadena en formato de minúsculas. | toLower('<*text*>') <p>Por ejemplo: toLower('Hello') | "Hola" |
| Devuelve un identificador único global (GUID). | guid() |"c2ecc88d-88c8-4096-912c-d6f2e2b138ce" |
||||

Para buscar funciones [según su propósito general](#ordered-by-purpose), utilice las siguientes tablas. O bien, para obtener información detallada acerca de cada función, consulte la [lista alfabética](#alphabetical-list).

> [!NOTE]
> En la sintaxis de las definiciones de parámetros, el signo de interrogación (?) que aparece después de un parámetro significa que el parámetro es opcional.
> Por ejemplo, consulte [getFutureTime()](#getFutureTime).

## <a name="functions-in-expressions"></a>Funciones en expresiones

Para mostrar cómo usar una función en una expresión, en este ejemplo se muestra cómo obtener el valor del parámetro `customerName` y asignar dicho valor a la propiedad `accountName` mediante la función [parameters()](#parameters) en una expresión:

```json
"accountName": "@parameters('customerName')"
```

Estas son algunas otras formas generales en que puede usar funciones en expresiones:

| Tarea | Sintaxis de la función en una expresión |
| ---- | -------------------------------- |
| Realice el trabajo con un elemento y para ello pase ese elemento a una función. | "\@<*functionName*>(<*item*>)" |
| 1. Obtenga el valor de *parameterName* mediante la función anidada `parameters()`. </br>2. Realice el trabajo con el resultado y para ello pase ese valor a *functionName*. | "\@<*functionName*>(parameters('<*parameterName*>'))" |
| 1. Obtenga el resultado de la función interna anidada *functionName*. </br>2. Pase el resultado a la función externa *functionName2*. | "\@<*functionName2*>(<*functionName*>(<*item*>))" |
| 1. Obtenga el resultado de *functionName*. </br>2. Dado que el resultado es un objeto con la propiedad *propertyName*, obtenga el valor de la propiedad. | "\@<*functionName*>(<*item*>).<*propertyName*>" |
|||

Por ejemplo, la función `concat()` puede tomar dos o más valores de cadena como parámetros. Esta función combina esas cadenas en una cadena.
Puede pasar literales de cadena, por ejemplo, "Sophia" y "Owen" para obtener una cadena combinada, "SophiaOwen":

```json
"customerName": "@concat('Sophia', 'Owen')"
```

O bien, puede obtener valores de cadena de los parámetros. En este ejemplo se usa la función `parameters()` en cada parámetro `concat()` y en los parámetros `firstName` y `lastName`. Luego se pasan las cadenas resultantes a la función `concat()` para obtener una cadena combinada, por ejemplo, "SophiaOwen":

```json
"customerName": "@concat(parameters('firstName'), parameters('lastName'))"
```

En cualquier de los casos, ambos ejemplos asignan el resultado a la propiedad `customerName`.

Estas son las funciones disponibles ordenadas por su uso general, o bien puede ver las funciones [por orden alfabético](#alphabetical-list).

<a name="ordered-by-purpose"></a>
<a name="string-functions"></a>

## <a name="string-functions"></a>Funciones de cadena

Para trabajar con cadenas, puede usar estas funciones de cadena y también algunas [funciones de colección](#collection-functions).
Las funciones de cadena solo funcionan en cadenas.

| Función de cadena | Tarea |
| --------------- | ---- |
| [concat](../logic-apps/workflow-definition-language-functions-reference.md#concat) | Combina dos o más cadenas y devuelve la cadena combinada. |
| [endsWith](../logic-apps/workflow-definition-language-functions-reference.md#endswith) | Comprueba si una cadena termina con la subcadena especificada. |
| [guid](../logic-apps/workflow-definition-language-functions-reference.md#guid) | Genera un identificador único global (GUID) como una cadena. |
| [indexOf](../logic-apps/workflow-definition-language-functions-reference.md#indexof) | Devuelve la posición inicial de una subcadena. |
| [lastIndexOf](../logic-apps/workflow-definition-language-functions-reference.md#lastindexof) | Devuelve la posición inicial de la última repetición de una subcadena. |
| [replace](../logic-apps/workflow-definition-language-functions-reference.md#replace) | Reemplaza una subcadena por la cadena especificada y devuelve la cadena actualizada. |
| [split](../logic-apps/workflow-definition-language-functions-reference.md#split) | Devuelve una matriz que contiene subcadenas, separadas por comas, de una cadena mayor en función de un carácter delimitador especificado en la cadena original. |
| [startsWith](../logic-apps/workflow-definition-language-functions-reference.md#startswith) | Comprueba si una cadena comienza con una subcadena especificada. |
| [substring](../logic-apps/workflow-definition-language-functions-reference.md#substring) | Devuelve caracteres de una cadena, a partir de la posición especificada. |
| [toLower](../logic-apps/workflow-definition-language-functions-reference.md#toLower) | Devuelve una cadena en formato de minúsculas. |
| [toUpper](../logic-apps/workflow-definition-language-functions-reference.md#toUpper) | Devuelve una cadena en formato de mayúsculas. |
| [trim](../logic-apps/workflow-definition-language-functions-reference.md#trim) | Quita el espacio en blanco inicial y final de una cadena y devuelve la cadena actualizada. |
|||

<a name="collection-functions"></a>

## <a name="collection-functions"></a>Funciones de colección

Para trabajar con colecciones, por lo general matrices, cadenas y, en ocasiones, diccionarios, puede usar estas funciones de colección.

| Función de colección | Tarea |
| ------------------- | ---- |
| [contains](../logic-apps/workflow-definition-language-functions-reference.md#contains) | Comprueba si una colección contiene un elemento específico. |
| [empty](../logic-apps/workflow-definition-language-functions-reference.md#empty) | Comprueba si una colección está vacía. |
| [first](../logic-apps/workflow-definition-language-functions-reference.md#first) | Devuelve el primer elemento de una colección. |
| [intersection](../logic-apps/workflow-definition-language-functions-reference.md#intersection) | Devuelve una colección que tiene *solo* los elementos comunes en las colecciones especificadas. |
| [item](../logic-apps/workflow-definition-language-functions-reference.md#item) | Dentro de una acción de repetición sobre una matriz, devuelve el elemento actual de la matriz durante la iteración actual de la acción. |
| [join](../logic-apps/workflow-definition-language-functions-reference.md#join) | Devuelve una cadena que tiene *todos* los elementos de una matriz, separados por el carácter especificado. |
| [last](../logic-apps/workflow-definition-language-functions-reference.md#last) | Devuelve el último elemento de una colección. |
| [length](../logic-apps/workflow-definition-language-functions-reference.md#length) | Devuelve el número de elementos de una cadena o una matriz. |
| [skip](../logic-apps/workflow-definition-language-functions-reference.md#skip) | Quita elementos del principio de una colección y devuelve *todos los demás* elementos. |
| [take](../logic-apps/workflow-definition-language-functions-reference.md#take) | Devuelve elementos del principio de una colección. |
| [union](../logic-apps/workflow-definition-language-functions-reference.md#union) | Devuelve una colección que tiene *todos* los elementos de las colecciones especificadas. |
|||

<a name="comparison-functions"></a>

## <a name="logical-comparison-functions"></a>Funciones de comparación lógica

Para trabajar con condiciones, comparar valores y resultados de expresiones, o evaluar varios tipos de lógica, puede usar estas funciones de comparación lógica.
Para obtener la referencia completa sobre cada función, consulte la [lista en orden alfabético](../logic-apps/workflow-definition-language-functions-reference.md#alphabetical-list).

| Función de comparación lógica | Tarea |
| --------------------------- | ---- |
| [and](../logic-apps/workflow-definition-language-functions-reference.md#and) | Comprueba si todas las expresiones son true. |
| [equals](../logic-apps/workflow-definition-language-functions-reference.md#equals) | Comprueba si ambos valores son equivalentes. |
| [greater](../logic-apps/workflow-definition-language-functions-reference.md#greater) | Comprueba si el primer valor es mayor que el segundo. |
| [greaterOrEquals](../logic-apps/workflow-definition-language-functions-reference.md#greaterOrEquals) | Comprueba si el primer valor es mayor o igual que el segundo. |
| [if](../logic-apps/workflow-definition-language-functions-reference.md#if) | Comprueba si una expresión es true o false. En función del resultado, devuelve un valor especificado. |
| [less](../logic-apps/workflow-definition-language-functions-reference.md#less) | Comprueba si el primer valor es menor que el segundo. |
| [lessOrEquals](../logic-apps/workflow-definition-language-functions-reference.md#lessOrEquals) | Comprueba si el primer valor es menor o igual que el segundo. |
| [not](../logic-apps/workflow-definition-language-functions-reference.md#not) | Comprueba si una expresión es false. |
| [or](../logic-apps/workflow-definition-language-functions-reference.md#or) | Comprueba si al menos una expresión es true. |
|||

<a name="conversion-functions"></a>

## <a name="conversion-functions"></a>Funciones de conversión

Para cambiar el tipo o el formato de un valor, puede usar estas funciones de conversión.
Por ejemplo, puede cambiar un valor de booleano a entero.
Para más información sobre el modo en que Logic Apps administra los tipos de contenido durante la conversión, consulte [Administración de los tipos de contenido](../logic-apps/logic-apps-content-type.md).
Para obtener la referencia completa sobre cada función, consulte la [lista en orden alfabético](../logic-apps/workflow-definition-language-functions-reference.md#alphabetical-list).

| Función de conversión | Tarea |
| ------------------- | ---- |
| [array](../logic-apps/workflow-definition-language-functions-reference.md#array) | Devuelve una matriz a partir de una única entrada especificada. Para varias entradas, consulte [createArray](../logic-apps/workflow-definition-language-functions-reference.md#createArray). |
| [base64](../logic-apps/workflow-definition-language-functions-reference.md#base64) | Devuelve la versión de una cadena codificada en base64. |
| [base64ToBinary](../logic-apps/workflow-definition-language-functions-reference.md#base64ToBinary) | Devuelve la versión binaria de una cadena codificada en base64. |
| [base64ToString](../logic-apps/workflow-definition-language-functions-reference.md#base64ToString) | Devuelve la versión de cadena de una cadena codificada en base64. |
| [binary](../logic-apps/workflow-definition-language-functions-reference.md#binary) | Devuelve la versión binaria de un valor de entrada. |
| [bool](../logic-apps/workflow-definition-language-functions-reference.md#bool) | Devuelve la versión booleana de un valor de entrada. |
| [createArray](../logic-apps/workflow-definition-language-functions-reference.md#createArray) | Devuelve una matriz a partir de varias entradas. |
| [dataUri](../logic-apps/workflow-definition-language-functions-reference.md#dataUri) | Devuelve el URI de datos de un valor de entrada. |
| [dataUriToBinary](../logic-apps/workflow-definition-language-functions-reference.md#dataUriToBinary) | Devuelve la versión binaria de un URI de datos. |
| [dataUriToString](../logic-apps/workflow-definition-language-functions-reference.md#dataUriToString) | Devuelve la versión de cadena de un URI de datos. |
| [decodeBase64](../logic-apps/workflow-definition-language-functions-reference.md#decodeBase64) | Devuelve la versión de cadena de una cadena codificada en base64. |
| [decodeDataUri](../logic-apps/workflow-definition-language-functions-reference.md#decodeDataUri) | Devuelve la versión binaria de un URI de datos. |
| [decodeUriComponent](../logic-apps/workflow-definition-language-functions-reference.md#decodeUriComponent) | Devuelve una cadena que reemplaza los caracteres de escape por versiones descodificadas. |
| [encodeUriComponent](../logic-apps/workflow-definition-language-functions-reference.md#encodeUriComponent) | Devuelve una cadena que reemplaza los caracteres no seguros de la dirección URL por caracteres de escape. |
| [float](../logic-apps/workflow-definition-language-functions-reference.md#float) | Devuelve un número de punto flotante de un valor de entrada. |
| [int](../logic-apps/workflow-definition-language-functions-reference.md#int) | Devuelve la versión como número entero de una cadena. |
| [json](../logic-apps/workflow-definition-language-functions-reference.md#json) | Devuelve el valor o el objeto de tipo Notación de objetos JavaScript (JSON) de una cadena o XML. |
| [string](../logic-apps/workflow-definition-language-functions-reference.md#string) | Devuelve la versión de cadena de un valor de entrada. |
| [uriComponent](../logic-apps/workflow-definition-language-functions-reference.md#uriComponent) | Devuelve la versión codificada con el URI de un valor de entrada mediante la sustitución de los caracteres no seguros de la dirección URL por caracteres de escape. |
| [uriComponentToBinary](../logic-apps/workflow-definition-language-functions-reference.md#uriComponentToBinary) | Devuelve la versión binaria de una cadena codificada con el URI. |
| [uriComponentToString](../logic-apps/workflow-definition-language-functions-reference.md#uriComponentToString) | Devuelve la versión de cadena de una cadena codificada con el URI. |
| [xml](../logic-apps/workflow-definition-language-functions-reference.md#xml) | Devuelve la versión XML de una cadena. |
|||

<a name="math-functions"></a>

## <a name="math-functions"></a>Funciones matemáticas

Para trabajar con números enteros y flotantes, puede usar estas funciones matemáticas.
Para obtener la referencia completa sobre cada función, consulte la [lista en orden alfabético](../logic-apps/workflow-definition-language-functions-reference.md#alphabetical-list).

| Función matemática | Tarea |
| ------------- | ---- |
| [add](../logic-apps/workflow-definition-language-functions-reference.md#add) | Devuelve el resultado de sumar dos números. |
| [div](../logic-apps/workflow-definition-language-functions-reference.md#div) | Devuelve el resultado de dividir dos números. |
| [max](../logic-apps/workflow-definition-language-functions-reference.md#max) | Devuelve el valor más alto de un conjunto de números o una matriz. |
| [min](../logic-apps/workflow-definition-language-functions-reference.md#min) | Devuelve el valor más bajo de un conjunto de números o una matriz. |
| [mod](../logic-apps/workflow-definition-language-functions-reference.md#mod) | Devuelve el resto de dividir dos números. |
| [mul](../logic-apps/workflow-definition-language-functions-reference.md#mul) | Devuelve el producto de multiplicar dos números. |
| [rand](../logic-apps/workflow-definition-language-functions-reference.md#rand) | Devuelve un entero aleatorio desde un intervalo especificado. |
| [range](../logic-apps/workflow-definition-language-functions-reference.md#range) | Devuelve una matriz de enteros que comienza en un entero especificado. |
| [sub](../logic-apps/workflow-definition-language-functions-reference.md#sub) | Devuelve el resultado de restar el segundo número del primero. |
|||

<a name="date-time-functions"></a>

## <a name="date-and-time-functions"></a>Funciones de fecha y hora

Para trabajar con fechas y horas, puede usar estas funciones de fecha y hora.
Para obtener la referencia completa sobre cada función, consulte la [lista en orden alfabético](../logic-apps/workflow-definition-language-functions-reference.md#alphabetical-list).

| Función de fecha u hora | Tarea |
| --------------------- | ---- |
| [addDays](../logic-apps/workflow-definition-language-functions-reference.md#addDays) | Agrega un número de días a una marca de tiempo. |
| [addHours](../logic-apps/workflow-definition-language-functions-reference.md#addHours) | Agrega un número de horas a una marca de tiempo. |
| [addMinutes](../logic-apps/workflow-definition-language-functions-reference.md#addMinutes) | Agrega un número de minutos a una marca de tiempo. |
| [addSeconds](../logic-apps/workflow-definition-language-functions-reference.md#addSeconds) | Agrega un número de segundos a una marca de tiempo. |
| [addToTime](../logic-apps/workflow-definition-language-functions-reference.md#addToTime) | Agrega un número de unidades de tiempo a una marca de tiempo. Consulte también [getFutureTime](../logic-apps/workflow-definition-language-functions-reference.md#getFutureTime). |
| [convertFromUtc](../logic-apps/workflow-definition-language-functions-reference.md#convertFromUtc) | Convierte una marca de tiempo del formato Hora Universal Coordinada (UTC) a la zona horaria de destino. |
| [convertTimeZone](../logic-apps/workflow-definition-language-functions-reference.md#convertTimeZone) | Convierte una marca de tiempo de la zona horaria de origen a la zona horaria de destino. |
| [convertToUtc](../logic-apps/workflow-definition-language-functions-reference.md#convertToUtc) | Convierte una marca de tiempo de la zona horaria de origen al formato Hora Universal Coordinada (UTC). |
| [dayOfMonth](../logic-apps/workflow-definition-language-functions-reference.md#dayOfMonth) | Devuelve el día del componente de mes de una marca de tiempo. |
| [dayOfWeek](../logic-apps/workflow-definition-language-functions-reference.md#dayOfWeek) | Devuelve el día del componente de semana de una marca de tiempo. |
| [dayOfYear](../logic-apps/workflow-definition-language-functions-reference.md#dayOfYear) | Devuelve el día del componente de año de una marca de tiempo. |
| [formatDateTime](../logic-apps/workflow-definition-language-functions-reference.md#formatDateTime) | Devuelve la fecha de una marca de tiempo. |
| [getFutureTime](../logic-apps/workflow-definition-language-functions-reference.md#getFutureTime) | Devuelve la marca de tiempo actual más las unidades de tiempo especificadas. Consulte también [addToTime](../logic-apps/workflow-definition-language-functions-reference.md#addToTime). |
| [getPastTime](../logic-apps/workflow-definition-language-functions-reference.md#getPastTime) | Devuelve la marca de tiempo actual menos las unidades de tiempo especificadas. Consulte también [subtractFromTime](../logic-apps/workflow-definition-language-functions-reference.md#subtractFromTime). |
| [startOfDay](../logic-apps/workflow-definition-language-functions-reference.md#startOfDay) | Devuelve el inicio del día de una marca de tiempo. |
| [startOfHour](../logic-apps/workflow-definition-language-functions-reference.md#startOfHour) | Devuelve el inicio de la hora de una marca de tiempo. |
| [startOfMonth](../logic-apps/workflow-definition-language-functions-reference.md#startOfMonth) | Devuelve el inicio del mes de una marca de tiempo. |
| [subtractFromTime](../logic-apps/workflow-definition-language-functions-reference.md#subtractFromTime) | Resta un número de unidades de tiempo de una marca de tiempo. Consulte también [getPastTime](../logic-apps/workflow-definition-language-functions-reference.md#getPastTime). |
| [ticks](../logic-apps/workflow-definition-language-functions-reference.md#ticks) | Devuelve el valor de la propiedad `ticks` de una marca de tiempo especificada. |
| [utcNow](../logic-apps/workflow-definition-language-functions-reference.md#utcNow) | Devuelve la marca de tiempo actual como una cadena. |
|||

<a name="workflow-functions"></a>

## <a name="workflow-functions"></a>Funciones de flujo de trabajo

Estas funciones de flujo de trabajo pueden ayudarle a:

* Obtener detalles sobre una instancia de flujo de trabajo en tiempo de ejecución
* Trabaje con las entradas usadas para crear instancias de aplicaciones lógicas o flujos.
* Hacer referencia a las salidas de desencadenadores y acciones

Por ejemplo, puede hacer referencia a las salidas de una acción y usar esos datos en una acción posterior.
Para obtener la referencia completa sobre cada función, consulte la [lista en orden alfabético](../logic-apps/workflow-definition-language-functions-reference.md#alphabetical-list).

| Función de flujo de trabajo | Tarea |
| ----------------- | ---- |
| [action](../logic-apps/workflow-definition-language-functions-reference.md#action) | Devuelve la salida de la acción actual en tiempo de ejecución, o valores de otros pares de nombre y valor JSON. Consulte también [actions](../logic-apps/workflow-definition-language-functions-reference.md#actions). |
| [actionBody](../logic-apps/workflow-definition-language-functions-reference.md#actionBody) | Devuelve la salida en tiempo de ejecución del elemento `body` de una acción. Consulte también [body](../logic-apps/workflow-definition-language-functions-reference.md#body). |
| [actionOutputs](../logic-apps/workflow-definition-language-functions-reference.md#actionOutputs) | Devuelve la salida en tiempo de ejecución de una acción. Consulte [outputs](../logic-apps/workflow-definition-language-functions-reference.md#outputs) y [actions](../logic-apps/workflow-definition-language-functions-reference.md#actions). |
| [actions](../logic-apps/workflow-definition-language-functions-reference.md#actions) | Devuelve la salida de una acción en tiempo de ejecución, o valores de otros pares de nombre y valor JSON. Consulte también [action](../logic-apps/workflow-definition-language-functions-reference.md#action).  |
| [body](#body) | Devuelve la salida en tiempo de ejecución del elemento `body` de una acción. Consulte también [actionBody](../logic-apps/workflow-definition-language-functions-reference.md#actionBody). |
| [formDataMultiValues](../logic-apps/workflow-definition-language-functions-reference.md#formDataMultiValues) | Crea una matriz con los valores que coinciden con un nombre de clave en las salidas de acción *form-data* o *form-encoded*. |
| [formDataValue](../logic-apps/workflow-definition-language-functions-reference.md#formDataValue) | Devuelve un valor único que coincide con un nombre de clave en las salidas *form-data* o *form-encoded* de una acción. |
| [item](../logic-apps/workflow-definition-language-functions-reference.md#item) | Dentro de una acción de repetición sobre una matriz, devuelve el elemento actual de la matriz durante la iteración actual de la acción. |
| [items](../logic-apps/workflow-definition-language-functions-reference.md#items) | Cuando se utilizan dentro de un bucle Foreach o Until, devuelven el elemento actual del bucle especificado.|
| [iterationIndexes](../logic-apps/workflow-definition-language-functions-reference.md#iterationIndexes) | Dentro de un bucle Until, devuelven el valor de índice de la iteración actual. Puede usar esta función dentro de bucles Until anidados. |
| [listCallbackUrl](../logic-apps/workflow-definition-language-functions-reference.md#listCallbackUrl) | Devuelve la "dirección URL de devolución de llamada" que llama a un desencadenador o una acción. |
| [multipartBody](../logic-apps/workflow-definition-language-functions-reference.md#multipartBody) | Devuelve el elemento body de una parte específica de la salida de una acción que consta de varias partes. |
| [outputs](../logic-apps/workflow-definition-language-functions-reference.md#outputs) | Devuelve la salida en tiempo de ejecución de una acción. |
| [parameters](../logic-apps/workflow-definition-language-functions-reference.md#parameters) | Se devuelve el valor de un parámetro que se describe en la definición de flujo de trabajo. |
| [result](../logic-apps/workflow-definition-language-functions-reference.md#result) | Devuelve las entradas y salidas de todas las acciones dentro de la acción de ámbito especificada, como `For_each`, `Until` y `Scope`. |
| [trigger](../logic-apps/workflow-definition-language-functions-reference.md#trigger) | Devuelve la salida de un desencadenador en tiempo de ejecución, o desde otros pares de nombre y valor JSON. Consulte también [triggerOutputs](#triggerOutputs) y [triggerBody](../logic-apps/workflow-definition-language-functions-reference.md#triggerBody). |
| [triggerBody](../logic-apps/workflow-definition-language-functions-reference.md#triggerBody) | Devuelve la salida `body` de un desencadenador en tiempo de ejecución. Consulte [trigger](../logic-apps/workflow-definition-language-functions-reference.md#trigger). |
| [triggerFormDataValue](../logic-apps/workflow-definition-language-functions-reference.md#triggerFormDataValue) | Devuelve un valor único que coincide con un nombre de clave en las salidas del desencadenador *form-data* o *form-encoded*. |
| [triggerMultipartBody](../logic-apps/workflow-definition-language-functions-reference.md#triggerMultipartBody) | Devuelve el cuerpo de una parte específica de la salida de varias partes de un desencadenador. |
| [triggerFormDataMultiValues](../logic-apps/workflow-definition-language-functions-reference.md#triggerFormDataMultiValues) | Crea una matriz cuyos valores coinciden con un nombre de clave en las salidas del desencadenador *form-data* o *form-encoded*. |
| [triggerOutputs](../logic-apps/workflow-definition-language-functions-reference.md#triggerOutputs) | Devuelve la salida de un desencadenador en tiempo de ejecución o valores de otros pares de nombre y valor JSON. Consulte [trigger](../logic-apps/workflow-definition-language-functions-reference.md#trigger). |
| [variables](../logic-apps/workflow-definition-language-functions-reference.md#variables) | Devuelve el valor de una variable especificada. |
| [workflow](../logic-apps/workflow-definition-language-functions-reference.md#workflow) | Devuelve todos los detalles sobre el propio flujo de trabajo durante el tiempo de ejecución. |
|||

<a name="uri-parsing-functions"></a>

## <a name="uri-parsing-functions"></a>Funciones de análisis de URI

Para trabajar con identificadores uniformes de recursos (URI) y obtener distintos valores de propiedad para estos URI, puede usar estas funciones de análisis de URI.
Para obtener la referencia completa sobre cada función, consulte la [lista en orden alfabético](../logic-apps/workflow-definition-language-functions-reference.md#alphabetical-list).

| Función de análisis de URI | Tarea |
| -------------------- | ---- |
| [uriHost](../logic-apps/workflow-definition-language-functions-reference.md#uriHost) | Devuelve el valor `host` de un identificador uniforme de recursos (URI). |
| [uriPath](../logic-apps/workflow-definition-language-functions-reference.md#uriPath) | Devuelve el valor `path` de un identificador uniforme de recursos (URI). |
| [uriPathAndQuery](../logic-apps/workflow-definition-language-functions-reference.md#uriPathAndQuery) | Devuelve los valores `path` y `query` de un identificador uniforme de recursos (URI). |
| [uriPort](../logic-apps/workflow-definition-language-functions-reference.md#uriPort) | Devuelve el valor `port` de un identificador uniforme de recursos (URI). |
| [uriQuery](../logic-apps/workflow-definition-language-functions-reference.md#uriQuery) | Devuelve el valor `query` de un identificador uniforme de recursos (URI). |
| [uriScheme](../logic-apps/workflow-definition-language-functions-reference.md#uriScheme) | Devuelve el valor `scheme` de un identificador uniforme de recursos (URI). |
|||

<a name="manipulation-functions"></a>

## <a name="manipulation-functions-json--xml"></a>Funciones de manipulación: JSON Y XML

Para trabajar con objetos JSON y nodos XML, puede usar estas funciones de manipulación.
Para obtener la referencia completa sobre cada función, consulte la [lista en orden alfabético](../logic-apps/workflow-definition-language-functions-reference.md#alphabetical-list).

| Función de manipulación | Tarea |
| --------------------- | ---- |
| [addProperty](../logic-apps/workflow-definition-language-functions-reference.md#addProperty) | Agrega una propiedad y su valor o un par de nombre y valor a un objeto JSON y devuelve el objeto actualizado. |
| [coalesce](../logic-apps/workflow-definition-language-functions-reference.md#coalesce) | Devuelve el primer valor distinto de null de uno o más parámetros. |
| [removeProperty](../logic-apps/workflow-definition-language-functions-reference.md#removeProperty) | Quita una propiedad de un objeto JSON y devuelve el objeto actualizado. |
| [setProperty](../logic-apps/workflow-definition-language-functions-reference.md#setProperty) | Establece el valor de propiedad de un objeto JSON y devuelve el objeto actualizado. |
| [xpath](../logic-apps/workflow-definition-language-functions-reference.md#xpath) | Comprueba el código XML de los nodos o valores que coinciden con una expresión XPath (XML Path Language) y devuelve los nodos o valores coincidentes. |
|||

<a name="alphabetical-list"></a>

## <a name="all-functions---alphabetical-list"></a>Todas las funciones: lista alfabética

En esta sección se enumeran todas las funciones disponibles en orden alfabético.

<a name="action"></a>

### <a name="action"></a>action

Devuelve la salida en tiempo de ejecución de la acción *actual* o los valores de otros pares de nombre y valor JSON, que puede asignar a una expresión.
De forma predeterminada, esta función hace referencia al objeto de acción entero, pero también puede especificar una propiedad cuyo valor desee obtener.
Consulte también [actions()](../logic-apps/workflow-definition-language-functions-reference.md#actions).

Puede usar la función `action()` únicamente en estas ubicaciones:

* La propiedad `unsubscribe` de una acción de webhook de forma que pueda acceder al resultado de la solicitud original `subscribe`
* La propiedad `trackedProperties` de una acción
* La condición del bucle `do-until` de una acción

```
action()
action().outputs.body.<property>
```

| Parámetro | Obligatorio | type | DESCRIPCIÓN |
| --------- | -------- | ---- | ----------- |
| <*propiedad*> | Sin | String | Nombre de la propiedad del objeto de acción cuyo valor desee: **name**, **startTime**, **endTime**, **inputs**,  **outputs**, **status**, **code**, **trackingId** y **clientTrackingId**. En Azure Portal, revise los detalles de un historial de ejecución específico para encontrar estas propiedades. Para más información, consulte [API REST: acciones de ejecución de flujo de trabajo](https://docs.microsoft.com/rest/api/logic/workflowrunactions/get). |
|||||

| Valor devuelto | type | DESCRIPCIÓN |
| ------------ | -----| ----------- |
| <*salida-de-la-acción*> | String | Salida de la acción o propiedad actual |
||||

<a name="actionBody"></a>

### <a name="actionbody"></a>actionBody

Devuelve la salida en tiempo de ejecución del elemento `body` de una acción.
Abreviatura de `actions('<actionName>').outputs.body`.
Consulte [body()](#body) y [actions()](#actions).

```
actionBody('<actionName>')
```

| Parámetro | Obligatorio | type | DESCRIPCIÓN |
| --------- | -------- | ---- | ----------- |
| <*nombre_de_la_acción*> | Sí | String | Nombre de la acción cuya salida del elemento `body` desea |
|||||

| Valor devuelto | type | DESCRIPCIÓN |
| ------------ | -----| ----------- |
| <*salida-del-elemento-body-de-la-acción*> | String | Salida del elemento `body` de la acción especificada |
||||

*Ejemplo*

Este ejemplo obtiene la salida del elemento `body` de la acción de Twitter `Get user`:

```
actionBody('Get_user')
```

Y devuelve este resultado:

```json
"body": {
  "FullName": "Contoso Corporation",
  "Location": "Generic Town, USA",
  "Id": 283541717,
  "UserName": "ContosoInc",
  "FollowersCount": 172,
  "Description": "Leading the way in transforming the digital workplace.",
  "StatusesCount": 93,
  "FriendsCount": 126,
  "FavouritesCount": 46,
  "ProfileImageUrl": "https://pbs.twimg.com/profile_images/908820389907722240/gG9zaHcd_400x400.jpg"
}
```

<a name="actionOutputs"></a>

### <a name="actionoutputs"></a>actionOutputs

Devuelve la salida en tiempo de ejecución de una acción.  Y es la abreviatura de `actions('<actionName>').outputs`. Consulte [actions()](#actions). La función `actionOutputs()` se resuelve en `outputs()` en el diseñador de aplicaciones lógicas, por lo que piense en usar [outputs()](#outputs), en lugar de `actionOutputs()`. Aunque ambas funciones funcionan del mismo modo, `outputs()` es preferible.

```
actionOutputs('<actionName>')
```

| Parámetro | Obligatorio | type | DESCRIPCIÓN |
| --------- | -------- | ---- | ----------- |
| <*nombre_de_la_acción*> | Sí | String | Nombre de la acción cuya salida desea |
|||||

| Valor devuelto | type | DESCRIPCIÓN |
| ------------ | -----| ----------- |
| <*salida*> | String | Salida de la acción especificada |
||||

*Ejemplo*

Este ejemplo obtiene la salida de la acción de Twitter `Get user`:

```
actionOutputs('Get_user')
```

Y devuelve este resultado:

```json
{
  "statusCode": 200,
  "headers": {
    "Pragma": "no-cache",
    "Vary": "Accept-Encoding",
    "x-ms-request-id": "a916ec8f52211265d98159adde2efe0b",
    "X-Content-Type-Options": "nosniff",
    "Timing-Allow-Origin": "*",
    "Cache-Control": "no-cache",
    "Date": "Mon, 09 Apr 2018 18:47:12 GMT",
    "Set-Cookie": "ARRAffinity=b9400932367ab5e3b6802e3d6158afffb12fcde8666715f5a5fbd4142d0f0b7d;Path=/;HttpOnly;Domain=twitter-wus.azconn-wus.p.azurewebsites.net",
    "X-AspNet-Version": "4.0.30319",
    "X-Powered-By": "ASP.NET",
    "Content-Type": "application/json; charset=utf-8",
    "Expires": "-1",
    "Content-Length": "339"
  },
  "body": {
    "FullName": "Contoso Corporation",
    "Location": "Generic Town, USA",
    "Id": 283541717,
    "UserName": "ContosoInc",
    "FollowersCount": 172,
    "Description": "Leading the way in transforming the digital workplace.",
    "StatusesCount": 93,
    "FriendsCount": 126,
    "FavouritesCount": 46,
    "ProfileImageUrl": "https://pbs.twimg.com/profile_images/908820389907722240/gG9zaHcd_400x400.jpg"
  }
}
```

<a name="actions"></a>

### <a name="actions"></a>actions

Devuelve la salida en tiempo de ejecución de una acción o los valores de otros pares de nombre y valor JSON, que puede asignar a una expresión. De forma predeterminada, la función hace referencia al objeto de acción entero, pero también puede especificar una propiedad cuyo valor desee obtener.
Para las versiones en forma abreviada, consulte [actionBody()](#actionBody), [actionOutputs()](#actionOutputs) y [body()](#body).
Para la acción actual, consulte [action()](#action).

> [!NOTE]
> Anteriormente, podía usar la función `actions()` o el elemento `conditions` al especificar que una acción se ejecutó basándose en la salida de otra acción. Sin embargo, para declarar explícitamente las dependencias entre las acciones, ahora debe usar la propiedad `runAfter` de la acción dependiente.
> Para más información sobre la propiedad `runAfter`, consulte [Detección y control de errores con la propiedad runAfter](../logic-apps/logic-apps-workflow-definition-language.md).

```
actions('<actionName>')
actions('<actionName>').outputs.body.<property>
```

| Parámetro | Obligatorio | type | DESCRIPCIÓN |
| --------- | -------- | ---- | ----------- |
| <*nombre_de_la_acción*> | Sí | String | Nombre del objeto de acción cuya salida desea  |
| <*propiedad*> | Sin | String | Nombre de la propiedad del objeto de acción cuyo valor desee: **name**, **startTime**, **endTime**, **inputs**,  **outputs**, **status**, **code**, **trackingId** y **clientTrackingId**. En Azure Portal, revise los detalles de un historial de ejecución específico para encontrar estas propiedades. Para más información, consulte [API REST: acciones de ejecución de flujo de trabajo](https://docs.microsoft.com/rest/api/logic/workflowrunactions/get). |
|||||

| Valor devuelto | type | DESCRIPCIÓN |
| ------------ | -----| ----------- |
| <*salida-de-la-acción*> | String | Salida de la acción o propiedad especificada |
||||

*Ejemplo*

Este ejemplo obtiene el valor de la propiedad `status` de la acción de Twitter `Get user` en tiempo de ejecución:

```
actions('Get_user').outputs.body.status
```

Y devuelve este resultado: `"Succeeded"`

<a name="add"></a>

### <a name="add"></a>agregar

Devuelve el resultado de sumar dos números.

```
add(<summand_1>, <summand_2>)
```

| Parámetro | Obligatorio | type | DESCRIPCIÓN |
| --------- | -------- | ---- | ----------- |
| <*sumando_1*>, <*sumando_2*> | Sí | Integer, Float o mixto | Números que se van a sumar |
|||||

| Valor devuelto | type | DESCRIPCIÓN |
| ------------ | -----| ----------- |
| <*resultado-de-la-suma*> | Integer o Float | Resultado de sumar los números especificados |
||||

*Ejemplo*

Este ejemplo suma los números especificados:

```
add(1, 1.5)
```

Y devuelve este resultado: `2.5`

<a name="addDays"></a>

### <a name="adddays"></a>addDays

Agrega un número de días a una marca de tiempo.

```
addDays('<timestamp>', <days>, '<format>'?)
```

| Parámetro | Obligatorio | type | DESCRIPCIÓN |
| --------- | -------- | ---- | ----------- |
| <*marca_de_tiempo*> | Sí | String | Cadena que contiene la marca de tiempo |
| <*días*> | Sí | Integer | Número positivo o negativo de días que desea agregar |
| <*formato*> | Sin | String | Puede ser un [especificador de formato sencillo](https://docs.microsoft.com/dotnet/standard/base-types/standard-date-and-time-format-strings) o un [patrón de formato personalizado](https://docs.microsoft.com/dotnet/standard/base-types/custom-date-and-time-format-strings). El formato predeterminado de la marca de tiempo es ["o"](https://docs.microsoft.com/dotnet/standard/base-types/standard-date-and-time-format-strings) (aaaa-MM-ddTHH:mm:ss:fffffffK), que cumple con [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) y conserva la información de zona horaria. |
|||||

| Valor devuelto | type | DESCRIPCIÓN |
| ------------ | ---- | ----------- |
| <*marca-de-tiempo-actualizada*> | String | Marca de tiempo más el número de días especificado  |
||||

*Ejemplo 1*

Este ejemplo agrega 10 días a la marca de tiempo especificada:

```
addDays('2018-03-15T13:00:00Z', 10)
```

Y devuelve este resultado: `"2018-03-25T00:00:0000000Z"`

*Ejemplo 2*

Este ejemplo resta cinco días a la marca de tiempo especificada:

```
addDays('2018-03-15T00:00:00Z', -5)
```

Y devuelve este resultado: `"2018-03-10T00:00:0000000Z"`

<a name="addHours"></a>

### <a name="addhours"></a>addHours

Agrega un número de horas a una marca de tiempo.

```
addHours('<timestamp>', <hours>, '<format>'?)
```

| Parámetro | Obligatorio | type | DESCRIPCIÓN |
| --------- | -------- | ---- | ----------- |
| <*marca_de_tiempo*> | Sí | String | Cadena que contiene la marca de tiempo |
| <*horas*> | Sí | Integer | Número positivo o negativo de horas que desea agregar |
| <*formato*> | Sin | String | Puede ser un [especificador de formato sencillo](https://docs.microsoft.com/dotnet/standard/base-types/standard-date-and-time-format-strings) o un [patrón de formato personalizado](https://docs.microsoft.com/dotnet/standard/base-types/custom-date-and-time-format-strings). El formato predeterminado de la marca de tiempo es ["o"](https://docs.microsoft.com/dotnet/standard/base-types/standard-date-and-time-format-strings) (aaaa-MM-ddTHH:mm:ss:fffffffK), que cumple con [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) y conserva la información de zona horaria. |
|||||

| Valor devuelto | type | DESCRIPCIÓN |
| ------------ | ---- | ----------- |
| <*marca-de-tiempo-actualizada*> | String | Marca de tiempo más el número de horas especificado  |
||||

*Ejemplo 1*

Este ejemplo agrega 10 horas a la marca de tiempo especificada:

```
addHours('2018-03-15T00:00:00Z', 10)
```

Y devuelve este resultado: `"2018-03-15T10:00:0000000Z"`

*Ejemplo 2*

Este ejemplo resta cinco horas a la marca de tiempo especificada:

```
addHours('2018-03-15T15:00:00Z', -5)
```

Y devuelve este resultado: `"2018-03-15T10:00:0000000Z"`

<a name="addMinutes"></a>

### <a name="addminutes"></a>addMinutes

Agrega un número de minutos a una marca de tiempo.

```
addMinutes('<timestamp>', <minutes>, '<format>'?)
```

| Parámetro | Obligatorio | type | DESCRIPCIÓN |
| --------- | -------- | ---- | ----------- |
| <*marca_de_tiempo*> | Sí | String | Cadena que contiene la marca de tiempo |
| <*minutos*> | Sí | Integer | Número positivo o negativo de minutos que desea agregar |
| <*formato*> | Sin | String | Puede ser un [especificador de formato sencillo](https://docs.microsoft.com/dotnet/standard/base-types/standard-date-and-time-format-strings) o un [patrón de formato personalizado](https://docs.microsoft.com/dotnet/standard/base-types/custom-date-and-time-format-strings). El formato predeterminado de la marca de tiempo es ["o"](https://docs.microsoft.com/dotnet/standard/base-types/standard-date-and-time-format-strings) (aaaa-MM-ddTHH:mm:ss:fffffffK), que cumple con [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) y conserva la información de zona horaria. |
|||||

| Valor devuelto | type | DESCRIPCIÓN |
| ------------ | ---- | ----------- |
| <*marca-de-tiempo-actualizada*> | String | Marca de tiempo más el número de minutos especificado |
||||

*Ejemplo 1*

Este ejemplo agrega 10 minutos a la marca de tiempo especificada:

```
addMinutes('2018-03-15T00:10:00Z', 10)
```

Y devuelve este resultado: `"2018-03-15T00:20:00.0000000Z"`

*Ejemplo 2*

Este ejemplo resta cinco minutos a la marca de tiempo especificada:

```
addMinutes('2018-03-15T00:20:00Z', -5)
```

Y devuelve este resultado: `"2018-03-15T00:15:00.0000000Z"`

<a name="addProperty"></a>

### <a name="addproperty"></a>addProperty

Agrega una propiedad y su valor o un par de nombre y valor a un objeto JSON y devuelve el objeto actualizado. Si la propiedad ya existe en tiempo de ejecución, la función genera un error.

```
addProperty(<object>, '<property>', <value>)
```

| Parámetro | Obligatorio | type | DESCRIPCIÓN |
| --------- | -------- | ---- | ----------- |
| <*objeto*> | Sí | Object | Objeto JSON al que desea agregar una propiedad |
| <*propiedad*> | Sí | String | Nombre de la propiedad que se va a agregar |
| <*valor*> | Sí | Any | Valor de la propiedad |
|||||

| Valor devuelto | type | DESCRIPCIÓN |
| ------------ | ---- | ----------- |
| <*objeto-actualizado*> | Object | Objeto JSON actualizado con la propiedad especificada |
||||

Para agregar una propiedad secundaria a una propiedad existente, use esta sintaxis:

```
addProperty(<object>['<parent-property>'], '<child-property>', <value>)
```

| Parámetro | Obligatorio | type | DESCRIPCIÓN |
| --------- | -------- | ---- | ----------- |
| <*objeto*> | Sí | Object | Objeto JSON al que desea agregar una propiedad |
| <*parent-property*> | Sí | Cadena | Nombre de la propiedad primaria en la que desea agregar la propiedad secundaria |
| <*child-property*> | Sí | Cadena | Nombre de la propiedad secundaria que se va a agregar |
| <*valor*> | Sí | Any | Valor que se va a establecer para la propiedad especificada |
|||||

| Valor devuelto | type | DESCRIPCIÓN |
| ------------ | ---- | ----------- |
| <*objeto-actualizado*> | Object | Objeto JSON actualizado cuya propiedad se ha establecido |
||||

*Ejemplo 1*

En este ejemplo se agrega la propiedad `middleName` a un objeto JSON, que se convierte de una cadena a JSON mediante la función [JSON()](#json). El objeto ya incluye las propiedades `firstName` y `surName`. La función asigna el valor especificado a la nueva propiedad y devuelve el objeto actualizado:

```
addProperty(json('{ "firstName": "Sophia", "lastName": "Owen" }'), 'middleName', 'Anne')
```

Este es el objeto JSON actual:

```json
{
   "firstName": "Sophia",
   "surName": "Owen"
}
```

Este es el objeto JSON actualizado:

```json
{
   "firstName": "Sophia",
   "middleName": "Anne",
   "surName": "Owen"
}
```

*Ejemplo 2*

En este ejemplo se agrega la propiedad secundaria `middleName` a la propiedad existente `customerName` en un objeto JSON, que se convierte de una cadena a JSON mediante la función [JSON()](#json). La función asigna el valor especificado a la nueva propiedad y devuelve el objeto actualizado:

```
addProperty(json('{ "customerName": { "firstName": "Sophia", "surName": "Owen" } }')['customerName'], 'middleName', 'Anne')
```

Este es el objeto JSON actual:

```json
{
   "customerName": {
      "firstName": "Sophia",
      "surName": "Owen"
   }
}
```

Este es el objeto JSON actualizado:

```json
{
   "customerName": {
      "firstName": "Sophia",
      "middleName": "Anne",
      "surName": "Owen"
   }
}
```

<a name="addSeconds"></a>

### <a name="addseconds"></a>addSeconds

Agrega un número de segundos a una marca de tiempo.

```
addSeconds('<timestamp>', <seconds>, '<format>'?)
```

| Parámetro | Obligatorio | type | DESCRIPCIÓN |
| --------- | -------- | ---- | ----------- |
| <*marca_de_tiempo*> | Sí | String | Cadena que contiene la marca de tiempo |
| <*segundos*> | Sí | Integer | Número positivo o negativo de segundos que desea agregar |
| <*formato*> | Sin | String | Puede ser un [especificador de formato sencillo](https://docs.microsoft.com/dotnet/standard/base-types/standard-date-and-time-format-strings) o un [patrón de formato personalizado](https://docs.microsoft.com/dotnet/standard/base-types/custom-date-and-time-format-strings). El formato predeterminado de la marca de tiempo es ["o"](https://docs.microsoft.com/dotnet/standard/base-types/standard-date-and-time-format-strings) (aaaa-MM-ddTHH:mm:ss:fffffffK), que cumple con [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) y conserva la información de zona horaria. |
|||||

| Valor devuelto | type | DESCRIPCIÓN |
| ------------ | ---- | ----------- |
| <*marca-de-tiempo-actualizada*> | String | Marca de tiempo más el número de segundos especificado  |
||||

*Ejemplo 1*

Este ejemplo agrega 10 segundos a la marca de tiempo especificada:

```
addSeconds('2018-03-15T00:00:00Z', 10)
```

Y devuelve este resultado: `"2018-03-15T00:00:10.0000000Z"`

*Ejemplo 2*

Este ejemplo resta cinco segundos a la marca de tiempo especificada:

```
addSeconds('2018-03-15T00:00:30Z', -5)
```

Y devuelve este resultado: `"2018-03-15T00:00:25.0000000Z"`

<a name="addToTime"></a>

### <a name="addtotime"></a>addToTime

Agrega un número de unidades de tiempo a una marca de tiempo.
Consulte también [getFutureTime()](#getFutureTime).

```
addToTime('<timestamp>', <interval>, '<timeUnit>', '<format>'?)
```

| Parámetro | Obligatorio | type | DESCRIPCIÓN |
| --------- | -------- | ---- | ----------- |
| <*marca_de_tiempo*> | Sí | String | Cadena que contiene la marca de tiempo |
| <*intervalo*> | Sí | Integer | Número de unidades de tiempo especificadas que se va a agregar |
| <*unidad_de_tiempo*> | Sí | String | La unidad de tiempo que se usará con *intervalo*: "Segundo", "Minuto", "Hora", "Día", "Semana", "Mes", "Año" |
| <*formato*> | Sin | String | Puede ser un [especificador de formato sencillo](https://docs.microsoft.com/dotnet/standard/base-types/standard-date-and-time-format-strings) o un [patrón de formato personalizado](https://docs.microsoft.com/dotnet/standard/base-types/custom-date-and-time-format-strings). El formato predeterminado de la marca de tiempo es ["o"](https://docs.microsoft.com/dotnet/standard/base-types/standard-date-and-time-format-strings) (aaaa-MM-ddTHH:mm:ss:fffffffK), que cumple con [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) y conserva la información de zona horaria. |
|||||

| Valor devuelto | type | DESCRIPCIÓN |
| ------------ | ---- | ----------- |
| <*marca-de-tiempo-actualizada*> | String | Marca de tiempo más el número de unidades de tiempo especificado  |
||||

*Ejemplo 1*

Este ejemplo agrega un día a la marca de tiempo especificada:

```
addToTime('2018-01-01T00:00:00Z', 1, 'Day')
```

Y devuelve este resultado: `"2018-01-02T00:00:00.0000000Z"`

*Ejemplo 2*

Este ejemplo agrega un día a la marca de tiempo especificada:

```
addToTime('2018-01-01T00:00:00Z', 1, 'Day', 'D')
```

Y devuelve el resultado con el formato "D" opcional: `"Tuesday, January 2, 2018"`

<a name="and"></a>

### <a name="and"></a>y

Comprueba si todas las expresiones son true.
Devuelve true cuando todas las expresiones son verdaderas o devuelve false cuando al menos una expresión es falsa.

```
and(<expression1>, <expression2>, ...)
```

| Parámetro | Obligatorio | type | DESCRIPCIÓN |
| --------- | -------- | ---- | ----------- |
| <*expresión1*>, <*expresión2*>, ... | Sí | Boolean | Expresiones que se van a comprobar |
|||||

| Valor devuelto | type | DESCRIPCIÓN |
| ------------ | -----| ----------- |
| true o false | Boolean | Devuelve true si todas las expresiones son verdaderas. Devuelve false cuando al menos una expresión es falsa. |
||||

*Ejemplo 1*

Estos ejemplos comprueban si los valores booleanos especificados son todos verdaderos:

```
and(true, true)
and(false, true)
and(false, false)
```

Y devuelve estos resultados:

* Primer ejemplo: ambas expresiones son verdaderas, por lo que devuelve `true`.
* Segundo ejemplo: una expresión es falsa, por lo que devuelve `false`.
* Tercer ejemplo: ambas expresiones son falsas, por lo que devuelve `false`.

*Ejemplo 2*

Estos ejemplos comprueban si las expresiones especificadas son todas verdaderas:

```
and(equals(1, 1), equals(2, 2))
and(equals(1, 1), equals(1, 2))
and(equals(1, 2), equals(1, 3))
```

Y devuelve estos resultados:

* Primer ejemplo: ambas expresiones son verdaderas, por lo que devuelve `true`.
* Segundo ejemplo: una expresión es falsa, por lo que devuelve `false`.
* Tercer ejemplo: ambas expresiones son falsas, por lo que devuelve `false`.

<a name="array"></a>

### <a name="array"></a>array

Devuelve una matriz a partir de una única entrada especificada.
Para varias entradas, consulte [createArray()](#createArray).

```
array('<value>')
```

| Parámetro | Obligatorio | type | DESCRIPCIÓN |
| --------- | -------- | ---- | ----------- |
| <*valor*> | Sí | String | Cadena para la creación de una matriz |
|||||

| Valor devuelto | type | DESCRIPCIÓN |
| ------------ | ---- | ----------- |
| [<*valor*>] | Array | Matriz que contiene la única entrada especificada |
||||

*Ejemplo*

Este ejemplo crea una matriz a partir de la cadena "hello":

```
array('hello')
```

Y devuelve este resultado: `["hello"]`

<a name="base64"></a>

### <a name="base64"></a>base64

Devuelve la versión de una cadena codificada en base64.

```
base64('<value>')
```

| Parámetro | Obligatorio | type | DESCRIPCIÓN |
| --------- | -------- | ---- | ----------- |
| <*valor*> | Sí | String | Cadena de entrada |
|||||

| Valor devuelto | type | DESCRIPCIÓN |
| ------------ | ---- | ----------- |
| <*cadena-en-base64*> | String | Versión codificada en base64 de la cadena de entrada |
||||

*Ejemplo*

Este ejemplo convierte la cadena "hello" en una cadena codificada en base64:

```
base64('hello')
```

Y devuelve este resultado: `"aGVsbG8="`

<a name="base64ToBinary"></a>

### <a name="base64tobinary"></a>base64ToBinary

Devuelve la versión binaria de una cadena codificada en base64.

```
base64ToBinary('<value>')
```

| Parámetro | Obligatorio | type | DESCRIPCIÓN |
| --------- | -------- | ---- | ----------- |
| <*valor*> | Sí | String | Cadena con codificación base64 que se va a convertir |
|||||

| Valor devuelto | type | DESCRIPCIÓN |
| ------------ | ---- | ----------- |
| <*binario-de-cadena-en-base64*> | String | Versión binaria de la cadena con codificación base64 |
||||

*Ejemplo*

Este ejemplo convierte la cadena "aGVsbG8 =" codificada en base64 en una cadena binaria:

```
base64ToBinary('aGVsbG8=')
```

Y devuelve este resultado:

`"0110000101000111010101100111001101100010010001110011100000111101"`

<a name="base64ToString"></a>

### <a name="base64tostring"></a>base64ToString

Devuelve la versión de cadena de una cadena codificada en base64, decodificando dicha cadena en base64.
Use esta función en lugar de [decodeBase64()](#decodeBase64).
Aunque ambas funciones funcionan del mismo modo, `base64ToString()` es preferible.

```
base64ToString('<value>')
```

| Parámetro | Obligatorio | type | DESCRIPCIÓN |
| --------- | -------- | ---- | ----------- |
| <*valor*> | Sí | String | Cadena con codificación base64 que se va a decodificar |
|||||

| Valor devuelto | type | DESCRIPCIÓN |
| ------------ | ---- | ----------- |
| <*cadena-base64-decodificada*> | String | Versión de cadena de una cadena codificada en base64 |
||||

*Ejemplo*

Este ejemplo convierte la cadena "aGVsbG8 =" codificada en base64 en una cadena:

```
base64ToString('aGVsbG8=')
```

Y devuelve este resultado: `"hello"`

<a name="binary"></a>

### <a name="binary"></a>binary

Devuelve la versión binaria de una cadena.

```
binary('<value>')
```

| Parámetro | Obligatorio | type | DESCRIPCIÓN |
| --------- | -------- | ---- | ----------- |
| <*valor*> | Sí | String | Cadena que se va a convertir |
|||||

| Valor devuelto | type | DESCRIPCIÓN |
| ------------ | ---- | ----------- |
| <*binario-del-valor-de-entrada*> | String | Versión binaria de la cadena especificada |
||||

*Ejemplo*

Este ejemplo convierte la cadena "hello" en una cadena en binario:

```
binary('hello')
```

Y devuelve este resultado:

`"0110100001100101011011000110110001101111"`

<a name="body"></a>

### <a name="body"></a>body

Devuelve la salida en tiempo de ejecución del elemento `body` de una acción.
Abreviatura de `actions('<actionName>').outputs.body`.
Consulte [actionBody()](#actionBody) y [actions()](#actions).

```
body('<actionName>')
```

| Parámetro | Obligatorio | type | DESCRIPCIÓN |
| --------- | -------- | ---- | ----------- |
| <*nombre_de_la_acción*> | Sí | String | Nombre de la acción cuya salida del elemento `body` desea |
|||||

| Valor devuelto | type | DESCRIPCIÓN |
| ------------ | -----| ----------- |
| <*salida-del-elemento-body-de-la-acción*> | String | Salida del elemento `body` de la acción especificada |
||||

*Ejemplo*

Este ejemplo obtiene la salida del elemento `body` de la acción de Twitter `Get user`:

```
body('Get_user')
```

Y devuelve este resultado:

```json
"body": {
    "FullName": "Contoso Corporation",
    "Location": "Generic Town, USA",
    "Id": 283541717,
    "UserName": "ContosoInc",
    "FollowersCount": 172,
    "Description": "Leading the way in transforming the digital workplace.",
    "StatusesCount": 93,
    "FriendsCount": 126,
    "FavouritesCount": 46,
    "ProfileImageUrl": "https://pbs.twimg.com/profile_images/908820389907722240/gG9zaHcd_400x400.jpg"
}
```

<a name="bool"></a>

### <a name="bool"></a>booleano

Devuelve la versión booleana de un valor.

```
bool(<value>)
```

| Parámetro | Obligatorio | type | DESCRIPCIÓN |
| --------- | -------- | ---- | ----------- |
| <*valor*> | Sí | Any | Valor que se va a convertir |
|||||

| Valor devuelto | type | DESCRIPCIÓN |
| ------------ | ---- | ----------- |
| true o false | Boolean | Versión booleana del valor especificado |
||||

*Ejemplo*

Estos ejemplos convierten los valores especificados en valores booleanos:

```
bool(1)
bool(0)
```

Y devuelve estos resultados:

* Primer ejemplo: `true`
* Segundo ejemplo: `false`

<a name="coalesce"></a>

### <a name="coalesce"></a>coalesce

Devuelve el primer valor distinto de null de uno o más parámetros.
Las cadenas vacías, las matrices vacías y los objetos vacíos no son nulos.

```
coalesce(<object_1>, <object_2>, ...)
```

| Parámetro | Obligatorio | type | DESCRIPCIÓN |
| --------- | -------- | ---- | ----------- |
| <*objeto_1*>, <*objeto_2*>, ... | Sí | Cualquiera, se pueden mezclar tipos | Uno o más elementos para comprobar si hay valores NULL |
|||||

| Valor devuelto | type | DESCRIPCIÓN |
| ------------ | ---- | ----------- |
| <*primer-elemento-no-NULL*> | Any | Primer elemento o valor que no sea NULL. Si todos los parámetros son NULL, esta función devuelve NULL. |
||||

*Ejemplo*

Estos ejemplos devuelven el primer valor distinto de NULL de los valores especificados o NULL cuando todos los valores son NULL:

```
coalesce(null, true, false)
coalesce(null, 'hello', 'world')
coalesce(null, null, null)
```

Y devuelve estos resultados:

* Primer ejemplo: `true`
* Segundo ejemplo: `"hello"`
* Tercer ejemplo: `null`

<a name="concat"></a>

### <a name="concat"></a>concat

Combina dos o más cadenas y devuelve la cadena combinada.

```
concat('<text1>', '<text2>', ...)
```

| Parámetro | Obligatorio | type | DESCRIPCIÓN |
| --------- | -------- | ---- | ----------- |
| <*texto1*>, <*texto2*>, ... | Sí | String | Al menos dos cadenas para combinar |
|||||

| Valor devuelto | type | DESCRIPCIÓN |
| ------------ | ---- | ----------- |
| <*texto1texto2...* > | String | Cadena que se crea a partir de las cadenas de entrada combinadas |
||||

*Ejemplo*

Este ejemplo combina las cadenas "Hello" y "World":

```
concat('Hello', 'World')
```

Y devuelve este resultado: `"HelloWorld"`

<a name="contains"></a>

### <a name="contains"></a>contains

Comprueba si una colección contiene un elemento específico.
Devuelve true cuando se encuentra el elemento o devuelve false cuando no se encuentra.
Esta función distingue mayúsculas de minúsculas.

```
contains('<collection>', '<value>')
contains([<collection>], '<value>')
```

En concreto, esta función funciona en estos tipos de colección:

* Una *cadena* para buscar una *subcadena*
* Una *matriz* para buscar un *valor*
* Un *diccionario* para buscar una *clave*

| Parámetro | Obligatorio | type | DESCRIPCIÓN |
| --------- | -------- | ---- | ----------- |
| <*colección*> | Sí | Cadena, matriz o diccionario | Colección que se va a comprobar |
| <*valor*> | Sí | Cadena, matriz o diccionario, respectivamente | Elemento que se va a buscar |
|||||

| Valor devuelto | type | DESCRIPCIÓN |
| ------------ | ---- | ----------- |
| true o false | Boolean | Devuelve true cuando se encuentra el elemento. Devuelve false si no se encuentra. |
||||

*Ejemplo 1*

Este ejemplo comprueba la cadena "hello world" para buscar la subcadena "world" y devuelve true:

```
contains('hello world', 'world')
```

*Ejemplo 2*

Este ejemplo comprueba la cadena "hello world" para buscar la subcadena "universe" y devuelve false:

```
contains('hello world', 'universe')
```

<a name="convertFromUtc"></a>

### <a name="convertfromutc"></a>convertFromUtc

Convierte una marca de tiempo del formato Hora Universal Coordinada (UTC) a la zona horaria de destino.

```
convertFromUtc('<timestamp>', '<destinationTimeZone>', '<format>'?)
```

| Parámetro | Obligatorio | type | DESCRIPCIÓN |
| --------- | -------- | ---- | ----------- |
| <*marca_de_tiempo*> | Sí | String | Cadena que contiene la marca de tiempo |
| <*zona_horaria_de_destino*> | Sí | String | Nombre de la zona horaria de destino. Para más información acerca de los nombres de zona horaria, consulte [Valores de índice de zona horaria de Microsoft](https://support.microsoft.com/en-us/help/973627/microsoft-time-zone-index-values). |
| <*formato*> | Sin | String | Puede ser un [especificador de formato sencillo](https://docs.microsoft.com/dotnet/standard/base-types/standard-date-and-time-format-strings) o un [patrón de formato personalizado](https://docs.microsoft.com/dotnet/standard/base-types/custom-date-and-time-format-strings). El formato predeterminado de la marca de tiempo es ["o"](https://docs.microsoft.com/dotnet/standard/base-types/standard-date-and-time-format-strings) (aaaa-MM-ddTHH:mm:ss:fffffffK), que cumple con [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) y conserva la información de zona horaria. |
|||||

| Valor devuelto | type | DESCRIPCIÓN |
| ------------ | ---- | ----------- |
| <*marca-de-tiempo-convertida*> | String | Marca de tiempo convertida a la zona horaria de destino |
||||

*Ejemplo 1*

Este ejemplo convierte una marca de tiempo a la zona horaria especificada:

```
convertFromUtc('2018-01-01T08:00:00.0000000Z', 'Pacific Standard Time')
```

Y devuelve este resultado: `"2018-01-01T00:00:00.0000000"`

*Ejemplo 2*

Este ejemplo convierte una marca de tiempo a la zona horaria y el formato especificados:

```
convertFromUtc('2018-01-01T08:00:00.0000000Z', 'Pacific Standard Time', 'D')
```

Y devuelve este resultado: `"Monday, January 1, 2018"`

<a name="convertTimeZone"></a>

### <a name="converttimezone"></a>convertTimeZone

Convierte una marca de tiempo de la zona horaria de origen a la zona horaria de destino.

```
convertTimeZone('<timestamp>', '<sourceTimeZone>', '<destinationTimeZone>', '<format>'?)
```

| Parámetro | Obligatorio | type | DESCRIPCIÓN |
| --------- | -------- | ---- | ----------- |
| <*marca_de_tiempo*> | Sí | String | Cadena que contiene la marca de tiempo |
| <*zona_horaria_de_origen*> | Sí | String | Nombre de la zona horaria de origen. Para más información acerca de los nombres de zona horaria, consulte [Valores de índice de zona horaria de Microsoft](https://support.microsoft.com/en-us/help/973627/microsoft-time-zone-index-values). |
| <*zona_horaria_de_destino*> | Sí | String | Nombre de la zona horaria de destino. Para más información acerca de los nombres de zona horaria, consulte [Valores de índice de zona horaria de Microsoft](https://support.microsoft.com/en-us/help/973627/microsoft-time-zone-index-values). |
| <*formato*> | Sin | String | Puede ser un [especificador de formato sencillo](https://docs.microsoft.com/dotnet/standard/base-types/standard-date-and-time-format-strings) o un [patrón de formato personalizado](https://docs.microsoft.com/dotnet/standard/base-types/custom-date-and-time-format-strings). El formato predeterminado de la marca de tiempo es ["o"](https://docs.microsoft.com/dotnet/standard/base-types/standard-date-and-time-format-strings) (aaaa-MM-ddTHH:mm:ss:fffffffK), que cumple con [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) y conserva la información de zona horaria. |
|||||

| Valor devuelto | type | DESCRIPCIÓN |
| ------------ | ---- | ----------- |
| <*marca-de-tiempo-convertida*> | String | Marca de tiempo convertida a la zona horaria de destino |
||||

*Ejemplo 1*

Este ejemplo convierte la zona horaria de origen a la zona horaria de destino:

```
convertTimeZone('2018-01-01T08:00:00.0000000Z', 'UTC', 'Pacific Standard Time')
```

Y devuelve este resultado: `"2018-01-01T00:00:00.0000000"`

*Ejemplo 2*

Este ejemplo convierte una zona horaria a la zona horaria y el formato especificados:

```
convertTimeZone('2018-01-01T80:00:00.0000000Z', 'UTC', 'Pacific Standard Time', 'D')
```

Y devuelve este resultado: `"Monday, January 1, 2018"`

<a name="convertToUtc"></a>

### <a name="converttoutc"></a>convertToUtc

Convierte una marca de tiempo de la zona horaria de origen al formato Hora Universal Coordinada (UTC).

```
convertToUtc('<timestamp>', '<sourceTimeZone>', '<format>'?)
```

| Parámetro | Obligatorio | type | DESCRIPCIÓN |
| --------- | -------- | ---- | ----------- |
| <*marca_de_tiempo*> | Sí | String | Cadena que contiene la marca de tiempo |
| <*zona_horaria_de_origen*> | Sí | String | Nombre de la zona horaria de origen. Para más información acerca de los nombres de zona horaria, consulte [Valores de índice de zona horaria de Microsoft](https://support.microsoft.com/en-us/help/973627/microsoft-time-zone-index-values). |
| <*formato*> | Sin | String | Puede ser un [especificador de formato sencillo](https://docs.microsoft.com/dotnet/standard/base-types/standard-date-and-time-format-strings) o un [patrón de formato personalizado](https://docs.microsoft.com/dotnet/standard/base-types/custom-date-and-time-format-strings). El formato predeterminado de la marca de tiempo es ["o"](https://docs.microsoft.com/dotnet/standard/base-types/standard-date-and-time-format-strings) (aaaa-MM-ddTHH:mm:ss:fffffffK), que cumple con [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) y conserva la información de zona horaria. |
|||||

| Valor devuelto | type | DESCRIPCIÓN |
| ------------ | ---- | ----------- |
| <*marca-de-tiempo-convertida*> | String | Marca de tiempo convertida a formato UTC |
||||

*Ejemplo 1*

Este ejemplo convierte una marca de tiempo a formato UTC:

```
convertToUtc('01/01/2018 00:00:00', 'Pacific Standard Time')
```

Y devuelve este resultado: `"2018-01-01T08:00:00.0000000Z"`

*Ejemplo 2*

Este ejemplo convierte una marca de tiempo a formato UTC:

```
convertToUtc('01/01/2018 00:00:00', 'Pacific Standard Time', 'D')
```

Y devuelve este resultado: `"Monday, January 1, 2018"`

<a name="createArray"></a>

### <a name="createarray"></a>createArray

Devuelve una matriz a partir de varias entradas.
Para matrices de una sola entrada, consulte [array()](#array).

```
createArray('<object1>', '<object2>', ...)
```

| Parámetro | Obligatorio | type | DESCRIPCIÓN |
| --------- | -------- | ---- | ----------- |
| <*objeto1*>, <*objeto2*>,... | Sí | Cualquiera, pero no mixtos | Al menos dos elementos para crear la matriz |
|||||

| Valor devuelto | type | DESCRIPCIÓN |
| ------------ | ---- | ----------- |
| [<*objeto1*>, <*objeto2*>, ...] | Array | Matriz creada a partir de todos los elementos de entrada |
||||

*Ejemplo*

Este ejemplo crea una matriz a partir de estas entradas:

```
createArray('h', 'e', 'l', 'l', 'o')
```

Y devuelve este resultado: `["h", "e", "l", "l", "o"]`

<a name="dataUri"></a>

### <a name="datauri"></a>dataUri

Devuelve un identificador uniforme de recursos (URI) de datos de una cadena.

```
dataUri('<value>')
```

| Parámetro | Obligatorio | type | DESCRIPCIÓN |
| --------- | -------- | ---- | ----------- |
| <*valor*> | Sí | String | Cadena que se va a convertir |
|||||

| Valor devuelto | type | DESCRIPCIÓN |
| ------------ | ---- | ----------- |
| <*uri-de-datos*> | String | Identificador URI de datos de la cadena de entrada |
||||

*Ejemplo*

Este ejemplo crea un identificador URI de datos a partir de la cadena "hello":

```
dataUri('hello')
```

Y devuelve este resultado: `"data:text/plain;charset=utf-8;base64,aGVsbG8="`

<a name="dataUriToBinary"></a>

### <a name="datauritobinary"></a>dataUriToBinary

Devuelve la versión binaria de un identificador uniforme de recursos (URI) de datos.
Use esta función en lugar de [decodeDataUri()](#decodeDataUri).
Aunque ambas funciones funcionan del mismo modo, `dataUriBinary()` es preferible.

```
dataUriToBinary('<value>')
```

| Parámetro | Obligatorio | type | DESCRIPCIÓN |
| --------- | -------- | ---- | ----------- |
| <*valor*> | Sí | String | Identificador URI de datos que se va a convertir |
|||||

| Valor devuelto | type | DESCRIPCIÓN |
| ------------ | ---- | ----------- |
| <*binario-de-uri-de-datos*> | String | Versión binaria del identificador URI de datos |
||||

*Ejemplo*

Este ejemplo crea una versión binaria de este identificador URI de datos:

```
dataUriToBinary('data:text/plain;charset=utf-8;base64,aGVsbG8=')
```

Y devuelve este resultado:

`"01100100011000010111010001100001001110100111010001100101011110000111010000101111011100000
1101100011000010110100101101110001110110110001101101000011000010111001001110011011001010111
0100001111010111010101110100011001100010110100111000001110110110001001100001011100110110010
10011011000110100001011000110000101000111010101100111001101100010010001110011100000111101"`

<a name="dataUriToString"></a>

### <a name="datauritostring"></a>dataUriToString

Devuelve la versión de cadena de un identificador uniforme de recursos (URI) de datos.

```
dataUriToString('<value>')
```

| Parámetro | Obligatorio | type | DESCRIPCIÓN |
| --------- | -------- | ---- | ----------- |
| <*valor*> | Sí | String | Identificador URI de datos que se va a convertir |
|||||

| Valor devuelto | type | DESCRIPCIÓN |
| ------------ | ---- | ----------- |
| <*cadena-de-uri-de-datos*> | String | Versión de cadena del identificador URI de datos |
||||

*Ejemplo*

Este ejemplo crea una cadena a partir de este identificador URI de datos:

```
dataUriToString('data:text/plain;charset=utf-8;base64,aGVsbG8=')
```

Y devuelve este resultado: `"hello"`

<a name="dayOfMonth"></a>

### <a name="dayofmonth"></a>dayOfMonth

Devuelve el día del mes de una marca de tiempo.

```
dayOfMonth('<timestamp>')
```

| Parámetro | Obligatorio | type | DESCRIPCIÓN |
| --------- | -------- | ---- | ----------- |
| <*marca_de_tiempo*> | Sí | String | Cadena que contiene la marca de tiempo |
|||||

| Valor devuelto | type | DESCRIPCIÓN |
| ------------ | ---- | ----------- |
| <*día-del-mes*> | Integer | Día del mes de la marca de tiempo especificada |
||||

*Ejemplo*

Este ejemplo devuelve el número del día del mes de esta marca de tiempo:

```
dayOfMonth('2018-03-15T13:27:36Z')
```

Y devuelve este resultado: `15`

<a name="dayOfWeek"></a>

### <a name="dayofweek"></a>dayOfWeek

Devuelve el día de la semana de una marca de tiempo.

```
dayOfWeek('<timestamp>')
```

| Parámetro | Obligatorio | type | DESCRIPCIÓN |
| --------- | -------- | ---- | ----------- |
| <*marca_de_tiempo*> | Sí | String | Cadena que contiene la marca de tiempo |
|||||

| Valor devuelto | type | DESCRIPCIÓN |
| ------------ | ---- | ----------- |
| <*día-de-la-semana*> | Integer | Día de la semana de la marca de tiempo especificada, donde el domingo es 0, el lunes es 1, etc. |
||||

*Ejemplo*

Este ejemplo devuelve el número del día de la semana de esta marca de tiempo:

```
dayOfWeek('2018-03-15T13:27:36Z')
```

Y devuelve este resultado: `3`

<a name="dayOfYear"></a>

### <a name="dayofyear"></a>dayOfYear

Devuelve el día del año de una marca de tiempo.

```
dayOfYear('<timestamp>')
```

| Parámetro | Obligatorio | type | DESCRIPCIÓN |
| --------- | -------- | ---- | ----------- |
| <*marca_de_tiempo*> | Sí | String | Cadena que contiene la marca de tiempo |
|||||

| Valor devuelto | type | DESCRIPCIÓN |
| ------------ | ---- | ----------- |
| <*día-del-año*> | Integer | Día del año de la marca de tiempo especificada |
||||

*Ejemplo*

Este ejemplo devuelve el número del día del año de esta marca de tiempo:

```
dayOfYear('2018-03-15T13:27:36Z')
```

Y devuelve este resultado: `74`

<a name="decodeBase64"></a>

### <a name="decodebase64"></a>decodeBase64

Devuelve la versión de cadena de una cadena codificada en base64, decodificando dicha cadena en base64.
Considere el uso de [base64ToString()](#base64ToString) en lugar de `decodeBase64()`.
Aunque ambas funciones funcionan del mismo modo, `base64ToString()` es preferible.

```
decodeBase64('<value>')
```

| Parámetro | Obligatorio | type | DESCRIPCIÓN |
| --------- | -------- | ---- | ----------- |
| <*valor*> | Sí | String | Cadena con codificación base64 que se va a decodificar |
|||||

| Valor devuelto | type | DESCRIPCIÓN |
| ------------ | ---- | ----------- |
| <*cadena-base64-decodificada*> | String | Versión de cadena de una cadena codificada en base64 |
||||

*Ejemplo*

Este ejemplo crea una cadena a partir de una cadena codificada en base64:

```
decodeBase64('aGVsbG8=')
```

Y devuelve este resultado: `"hello"`

<a name="decodeDataUri"></a>

### <a name="decodedatauri"></a>decodeDataUri

Devuelve la versión binaria de un identificador uniforme de recursos (URI) de datos.
Considere el uso de [dataUriToBinary()](#dataUriToBinary) en lugar de `decodeDataUri()`.
Aunque ambas funciones funcionan del mismo modo, `dataUriToBinary()` es preferible.

```
decodeDataUri('<value>')
```

| Parámetro | Obligatorio | type | DESCRIPCIÓN |
| --------- | -------- | ---- | ----------- |
| <*valor*> | Sí | String | Cadena URI de datos que se va a decodificar |
|||||

| Valor devuelto | type | DESCRIPCIÓN |
| ------------ | ---- | ----------- |
| <*binario-de-uri-de-datos*> | String | Versión binaria de una cadena URI de datos |
||||

*Ejemplo*

Este ejemplo devuelve la versión binaria de este identificador URI de datos:

```
decodeDataUri('data:text/plain;charset=utf-8;base64,aGVsbG8=')
```

Y devuelve este resultado:

`"01100100011000010111010001100001001110100111010001100101011110000111010000101111011100000
1101100011000010110100101101110001110110110001101101000011000010111001001110011011001010111
0100001111010111010101110100011001100010110100111000001110110110001001100001011100110110010
10011011000110100001011000110000101000111010101100111001101100010010001110011100000111101"`

<a name="decodeUriComponent"></a>

### <a name="decodeuricomponent"></a>decodeUriComponent

Devuelve una cadena que reemplaza los caracteres de escape por versiones descodificadas.

```
decodeUriComponent('<value>')
```

| Parámetro | Obligatorio | type | DESCRIPCIÓN |
| --------- | -------- | ---- | ----------- |
| <*valor*> | Sí | String | Cadena con caracteres de escape que se va a decodificar |
|||||

| Valor devuelto | type | DESCRIPCIÓN |
| ------------ | ---- | ----------- |
| <*uri-decodificado*> | String | Cadena actualizada con los caracteres de escape decodificados |
||||

*Ejemplo*

Este ejemplo reemplaza los caracteres de escape de esta cadena por versiones decodificadas:

```
decodeUriComponent('http%3A%2F%2Fcontoso.com')
```

Y devuelve este resultado: `"https://contoso.com"`

<a name="div"></a>

### <a name="div"></a>div

Devuelve el resultado entero de dividir dos números.
Para obtener el resultado del resto, consulte [mod()](#mod).

```
div(<dividend>, <divisor>)
```

| Parámetro | Obligatorio | type | DESCRIPCIÓN |
| --------- | -------- | ---- | ----------- |
| <*dividendo*> | Sí | Integer o Float | Número que se va a dividir entre el *divisor*. |
| <*divisor*> | Sí | Integer o Float | Número que divide el *dividendo*, pero no puede ser 0 |
|||||

| Valor devuelto | type | DESCRIPCIÓN |
| ------------ | ---- | ----------- |
| <*resultado-cociente*> | Integer | Resultado entero de dividir el primer número entre el segundo número |
||||

*Ejemplo*

Ambos ejemplos dividen el primer número entre el segundo número:

```
div(10, 5)
div(11, 5)
```

Y devuelven este resultado: `2`

<a name="encodeUriComponent"></a>

### <a name="encodeuricomponent"></a>encodeUriComponent

Devuelve una versión codificada de un identificador uniforme de recursos (URI) de una cadena reemplazando los caracteres no seguros de la dirección URL por caracteres de escape.
Considere el uso de [uriComponent()](#uriComponent) en lugar de `encodeUriComponent()`.
Aunque ambas funciones funcionan del mismo modo, `uriComponent()` es preferible.

```
encodeUriComponent('<value>')
```

| Parámetro | Obligatorio | type | DESCRIPCIÓN |
| --------- | -------- | ---- | ----------- |
| <*valor*> | Sí | String | Cadena que se va a convertir en formato codificado de URI |
|||||

| Valor devuelto | type | DESCRIPCIÓN |
| ------------ | ---- | ----------- |
| <*uri-codificado*> | String | Cadena codificada en formato URI con caracteres de escape |
||||

*Ejemplo*

Este ejemplo crea una versión codificada en formato URI de esta cadena:

```
encodeUriComponent('https://contoso.com')
```

Y devuelve este resultado: `"http%3A%2F%2Fcontoso.com"`

<a name="empty"></a>

### <a name="empty"></a>empty

Comprueba si una colección está vacía.
Devuelve true cuando la colección está vacía o devuelve false cuando no está vacía.

```
empty('<collection>')
empty([<collection>])
```

| Parámetro | Obligatorio | type | DESCRIPCIÓN |
| --------- | -------- | ---- | ----------- |
| <*colección*> | Sí | Cadena, matriz u objeto | Colección que se va a comprobar |
|||||

| Valor devuelto | type | DESCRIPCIÓN |
| ------------ | ---- | ----------- |
| true o false | Boolean | Devuelve true cuando la colección está vacía. Devuelve false si no está vacía. |
||||

*Ejemplo*

Estos ejemplos comprueban si las colecciones especificadas están vacías:

```
empty('')
empty('abc')
```

Y devuelve estos resultados:

* Primer ejemplo: pasa una cadena vacía, por lo que la función devuelve `true`.
* Segundo ejemplo: pasa la cadena "abc", por lo que la función devuelve `false`.

<a name="endswith"></a>

### <a name="endswith"></a>endsWith

Comprueba si una cadena termina con una subcadena especificada.
Devuelve true cuando se encuentra la subcadena o devuelve false cuando no se encuentra.
Esta función no distingue mayúsculas de minúsculas.

```
endsWith('<text>', '<searchText>')
```

| Parámetro | Obligatorio | type | DESCRIPCIÓN |
| --------- | -------- | ---- | ----------- |
| <*texto*> | Sí | String | Cadena que se va a comprobar |
| <*texto_a_buscar*> | Sí | String | Subcadena final que se va a buscar |
|||||

| Valor devuelto | type | DESCRIPCIÓN |
| ------------ | ---- | ----------- |
| true o false  | Boolean | Devuelve true cuando se encuentra la subcadena final. Devuelve false si no se encuentra. |
||||

*Ejemplo 1*

Este ejemplo comprueba si la cadena "hello world" termina con la cadena "world":

```
endsWith('hello world', 'world')
```

Y devuelve este resultado: `true`

*Ejemplo 2*

Este ejemplo comprueba si la cadena "hello world" termina con la cadena "universe":

```
endsWith('hello world', 'universe')
```

Y devuelve este resultado: `false`

<a name="equals"></a>

### <a name="equals"></a>equals

Comprueba si los valores, expresiones u objetos son equivalentes.
Devuelve true cuando ambos son equivalentes o devuelve false cuando no son equivalentes.

```
equals('<object1>', '<object2>')
```

| Parámetro | Obligatorio | type | DESCRIPCIÓN |
| --------- | -------- | ---- | ----------- |
| <*objeto1*>, <*objeto2*> | Sí | Varios | Valores, expresiones u objetos que se van a comparar |
|||||

| Valor devuelto | type | DESCRIPCIÓN |
| ------------ | ---- | ----------- |
| true o false | Boolean | Devuelve true cuando ambos son equivalentes. Devuelve false cuando no son equivalentes. |
||||

*Ejemplo*

Estos ejemplos comprueban si las entradas especificadas son equivalentes.

```
equals(true, 1)
equals('abc', 'abcd')
```

Y devuelve estos resultados:

* Primer ejemplo: ambos valores son equivalentes, por lo que la función devuelve `true`.
* Segundo ejemplo: ambos valores no son equivalentes, por lo que la función devuelve `false`.

<a name="first"></a>

### <a name="first"></a>first

Devuelve el primer elemento de una cadena o una matriz.

```
first('<collection>')
first([<collection>])
```

| Parámetro | Obligatorio | type | DESCRIPCIÓN |
| --------- | -------- | ---- | ----------- |
| <*colección*> | Sí | Cadena o matriz | Colección en la que buscar el primer elemento |
|||||

| Valor devuelto | type | DESCRIPCIÓN |
| ------------ | ---- | ----------- |
| <*primer-elemento-de-la-colección*> | Any | Primer elemento de la colección |
||||

*Ejemplo*

Estos ejemplos buscan el primer elemento de estas colecciones:

```
first('hello')
first(createArray(0, 1, 2))
```

Y devuelven estos resultados:

* Primer ejemplo: `"h"`
* Segundo ejemplo: `0`

<a name="float"></a>

### <a name="float"></a>float

Convierte una versión de cadena de un número de punto flotante a un número de punto flotante real.
Solo puede usar esta función al pasar parámetros personalizados a una aplicación, por ejemplo, una aplicación lógica o un flujo.

```
float('<value>')
```

| Parámetro | Obligatorio | type | DESCRIPCIÓN |
| --------- | -------- | ---- | ----------- |
| <*valor*> | Sí | String | Cadena que contiene un número de punto flotante válido que se va a convertir |
|||||

| Valor devuelto | type | DESCRIPCIÓN |
| ------------ | ---- | ----------- |
| <*valor-de-tipo-float*> | Float | Número de punto flotante a partir de la cadena especificada |
||||

*Ejemplo*

Este ejemplo crea una versión de cadena para este número de punto flotante:

```
float('10.333')
```

Y devuelve este resultado: `10.333`

<a name="formatDateTime"></a>

### <a name="formatdatetime"></a>formatDateTime

Devuelve una marca de tiempo en el formato especificado.

```
formatDateTime('<timestamp>', '<format>'?)
```

| Parámetro | Obligatorio | type | DESCRIPCIÓN |
| --------- | -------- | ---- | ----------- |
| <*marca_de_tiempo*> | Sí | String | Cadena que contiene la marca de tiempo |
| <*formato*> | Sin | String | Puede ser un [especificador de formato sencillo](https://docs.microsoft.com/dotnet/standard/base-types/standard-date-and-time-format-strings) o un [patrón de formato personalizado](https://docs.microsoft.com/dotnet/standard/base-types/custom-date-and-time-format-strings). El formato predeterminado de la marca de tiempo es ["o"](https://docs.microsoft.com/dotnet/standard/base-types/standard-date-and-time-format-strings) (aaaa-MM-ddTHH:mm:ss:fffffffK), que cumple con [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) y conserva la información de zona horaria. |
|||||

| Valor devuelto | type | DESCRIPCIÓN |
| ------------ | ---- | ----------- |
| <*marca-de-tiempo-con-nuevo-formato*> | String | Marca de tiempo actualizada en el formato especificado |
||||

*Ejemplo*

Este ejemplo convierte una marca de tiempo al formato especificado:

```
formatDateTime('03/15/2018 12:00:00', 'yyyy-MM-ddTHH:mm:ss')
```

Y devuelve este resultado: `"2018-03-15T12:00:00"`

<a name="formDataMultiValues"></a>

### <a name="formdatamultivalues"></a>formDataMultiValues

Devuelve una matriz con valores que coinciden con un nombre de clave de la salidas *form-data* o *form-encoded* de una acción.

```
formDataMultiValues('<actionName>', '<key>')
```

| Parámetro | Obligatorio | type | DESCRIPCIÓN |
| --------- | -------- | ---- | ----------- |
| <*nombre_de_la_acción*> | Sí | String | Acción cuya salida tiene el valor de una clave que busca |
| <*clave*> | Sí | String | Nombre de la clave cuyo valor busca |
|||||

| Valor devuelto | type | DESCRIPCIÓN |
| ------------ | ---- | ----------- |
| [<*matriz-con-valores-de-clave*>] | Array | Matriz con todos los valores que coinciden con la clave especificada |
||||

*Ejemplo*

Este ejemplo crea una matriz a partir del valor de la clave "Subject" de la salidas form-data o form-encoded de la acción especificada:

```
formDataMultiValues('Send_an_email', 'Subject')
```

Y devuelve el texto del asunto en una matriz, por ejemplo: `["Hello world"]`

<a name="formDataValue"></a>

### <a name="formdatavalue"></a>formDataValue

Devuelve un valor único que coincide con un nombre de clave en las salidas *form-data* o *form-encoded* de una acción.
Si la función encuentra más de una coincidencia, la función produce un error.

```
formDataValue('<actionName>', '<key>')
```

| Parámetro | Obligatorio | type | DESCRIPCIÓN |
| --------- | -------- | ---- | ----------- |
| <*nombre_de_la_acción*> | Sí | String | Acción cuya salida tiene el valor de una clave que busca |
| <*clave*> | Sí | String | Nombre de la clave cuyo valor busca |
|||||

| Valor devuelto | type | DESCRIPCIÓN |
| ------------ | ---- | ----------- |
| <*key-value*> | String | Valor de la clave especificada  |
||||

*Ejemplo*

Este ejemplo crea una cadena a partir del valor de la clave "Subject" de la salidas form-data o form-encoded de la acción especificada:

```
formDataValue('Send_an_email', 'Subject')
```

Y devuelve el texto del asunto como una cadena, por ejemplo: `"Hello world"`

<a name="getFutureTime"></a>

### <a name="getfuturetime"></a>getFutureTime

Devuelve la marca de tiempo actual más las unidades de tiempo especificadas.

```
getFutureTime(<interval>, <timeUnit>, <format>?)
```

| Parámetro | Obligatorio | type | DESCRIPCIÓN |
| --------- | -------- | ---- | ----------- |
| <*intervalo*> | Sí | Integer | Número de unidades de tiempo especificadas que se va a sustraer |
| <*unidad_de_tiempo*> | Sí | String | La unidad de tiempo que se usará con *intervalo*: "Segundo", "Minuto", "Hora", "Día", "Semana", "Mes", "Año" |
| <*formato*> | Sin | String | Puede ser un [especificador de formato sencillo](https://docs.microsoft.com/dotnet/standard/base-types/standard-date-and-time-format-strings) o un [patrón de formato personalizado](https://docs.microsoft.com/dotnet/standard/base-types/custom-date-and-time-format-strings). El formato predeterminado de la marca de tiempo es ["o"](https://docs.microsoft.com/dotnet/standard/base-types/standard-date-and-time-format-strings) (aaaa-MM-ddTHH:mm:ss:fffffffK), que cumple con [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) y conserva la información de zona horaria. |
|||||

| Valor devuelto | type | DESCRIPCIÓN |
| ------------ | ---- | ----------- |
| <*marca-de-tiempo-actualizada*> | String | Marca de tiempo actual más el número de unidades de tiempo especificado |
||||

*Ejemplo 1*

Imagine que la marca de tiempo actual es "2018-03-01T00:00:00.0000000Z".
Este ejemplo agrega cinco días a esa marca de tiempo:

```
getFutureTime(5, 'Day')
```

Y devuelve este resultado: `"2018-03-06T00:00:00.0000000Z"`

*Ejemplo 2*

Imagine que la marca de tiempo actual es "2018-03-01T00:00:00.0000000Z".
Este ejemplo agrega cinco días y convierte el resultado al formato "D":

```
getFutureTime(5, 'Day', 'D')
```

Y devuelve este resultado: `"Tuesday, March 6, 2018"`

<a name="getPastTime"></a>

### <a name="getpasttime"></a>getPastTime

Devuelve la marca de tiempo actual menos las unidades de tiempo especificadas.

```
getPastTime(<interval>, <timeUnit>, <format>?)
```

| Parámetro | Obligatorio | type | DESCRIPCIÓN |
| --------- | -------- | ---- | ----------- |
| <*intervalo*> | Sí | Integer | Número de unidades de tiempo especificadas que se va a sustraer |
| <*unidad_de_tiempo*> | Sí | String | La unidad de tiempo que se usará con *intervalo*: "Segundo", "Minuto", "Hora", "Día", "Semana", "Mes", "Año" |
| <*formato*> | Sin | String | Puede ser un [especificador de formato sencillo](https://docs.microsoft.com/dotnet/standard/base-types/standard-date-and-time-format-strings) o un [patrón de formato personalizado](https://docs.microsoft.com/dotnet/standard/base-types/custom-date-and-time-format-strings). El formato predeterminado de la marca de tiempo es ["o"](https://docs.microsoft.com/dotnet/standard/base-types/standard-date-and-time-format-strings) (aaaa-MM-ddTHH:mm:ss:fffffffK), que cumple con [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) y conserva la información de zona horaria. |
|||||

| Valor devuelto | type | DESCRIPCIÓN |
| ------------ | ---- | ----------- |
| <*marca-de-tiempo-actualizada*> | String | Marca de tiempo actual menos el número de unidades de tiempo especificado |
||||

*Ejemplo 1*

Imagine que la marca de tiempo actual es "2018-02-01T00:00:00.0000000Z".
Este ejemplo resta cinco días a esa marca de tiempo:

```
getPastTime(5, 'Day')
```

Y devuelve este resultado: `"2018-01-27T00:00:00.0000000Z"`

*Ejemplo 2*

Imagine que la marca de tiempo actual es "2018-02-01T00:00:00.0000000Z".
Este ejemplo resta cinco días y convierte el resultado al formato "D":

```
getPastTime(5, 'Day', 'D')
```

Y devuelve este resultado: `"Saturday, January 27, 2018"`

<a name="greater"></a>

### <a name="greater"></a>greater

Comprueba si el primer valor es mayor que el segundo.
Devuelve true cuando el primer valor es mayor o devuelve false cuando es menor.

```
greater(<value>, <compareTo>)
greater('<value>', '<compareTo>')
```

| Parámetro | Obligatorio | type | DESCRIPCIÓN |
| --------- | -------- | ---- | ----------- |
| <*valor*> | Sí | Integer, Float o String | Primer valor para comprobar si es mayor que el segundo valor |
| <*comparar_con*> | Sí | Integer, Float o String, respectivamente | Valor de comparación |
|||||

| Valor devuelto | type | DESCRIPCIÓN |
| ------------ | ---- | ----------- |
| true o false | Boolean | Devuelve true si el primer valor es mayor que el segundo. Devuelve false si el primer valor es igual o menor que el segundo. |
||||

*Ejemplo*

Estos ejemplos comprueban si el primer valor es mayor que el segundo:

```
greater(10, 5)
greater('apple', 'banana')
```

Y devuelven estos resultados:

* Primer ejemplo: `true`
* Segundo ejemplo: `false`

<a name="greaterOrEquals"></a>

### <a name="greaterorequals"></a>greaterOrEquals

Comprueba si el primer valor es mayor o igual que el segundo.
Devuelve true cuando el primer valor es mayor o igual o devuelve false cuando el primer valor es menor.

```
greaterOrEquals(<value>, <compareTo>)
greaterOrEquals('<value>', '<compareTo>')
```

| Parámetro | Obligatorio | type | DESCRIPCIÓN |
| --------- | -------- | ---- | ----------- |
| <*valor*> | Sí | Integer, Float o String | Primer valor para comprobar si es mayor o igual que el segundo valor |
| <*comparar_con*> | Sí | Integer, Float o String, respectivamente | Valor de comparación |
|||||

| Valor devuelto | type | DESCRIPCIÓN |
| ------------ | ---- | ----------- |
| true o false | Boolean | Devuelve true si el primer valor es mayor o igual que el segundo. Devuelve false si el primer valor es menor que el segundo. |
||||

*Ejemplo*

Estos ejemplos comprueban si el primer valor es mayor o igual que el segundo:

```
greaterOrEquals(5, 5)
greaterOrEquals('apple', 'banana')
```

Y devuelven estos resultados:

* Primer ejemplo: `true`
* Segundo ejemplo: `false`

<a name="guid"></a>

### <a name="guid"></a>GUID

Genera un identificador único global (GUID) como una cadena, por ejemplo, "c2ecc88d-88c8-4096-912c-d6f2e2b138ce":

```
guid()
```

Además, es posible especificar un formato diferente del GUID que no sea el formato predeterminado, "D", que es 32 dígitos separados por guiones.

```
guid('<format>')
```

| Parámetro | Obligatorio | type | DESCRIPCIÓN |
| --------- | -------- | ---- | ----------- |
| <*formato*> | Sin | String | Un único [especificador de formato](https://msdn.microsoft.com/library/97af8hh4) para el GUID devuelto. De forma predeterminada, el formato es "D", pero puede usar "N", "D", "B", "P" o "X". |
|||||

| Valor devuelto | type | DESCRIPCIÓN |
| ------------ | ---- | ----------- |
| <*valor-de-GUID*> | String | GUID generado aleatoriamente |
||||

*Ejemplo*

Este ejemplo genera el mismo GUID, pero como 32 dígitos separados por guiones y entre paréntesis:

```
guid('P')
```

Y devuelve este resultado: `"(c2ecc88d-88c8-4096-912c-d6f2e2b138ce)"`

<a name="if"></a>

### <a name="if"></a>if

Comprueba si una expresión es true o false.
En función del resultado, devuelve un valor especificado.

```
if(<expression>, <valueIfTrue>, <valueIfFalse>)
```

| Parámetro | Obligatorio | type | DESCRIPCIÓN |
| --------- | -------- | ---- | ----------- |
| <*expresión*> | Sí | Boolean | Expresión que se va a evaluar |
| <*valor_si_es_true*> | Sí | Any | Valor que se devuelve cuando la expresión es verdadera |
| <*valor_si_es_false*> | Sí | Any | Valor que se devuelve cuando la expresión es falsa |
|||||

| Valor devuelto | type | DESCRIPCIÓN |
| ------------ | ---- | ----------- |
| <*valor-a-devolver-especificado*> | Any | Valor especificado que se devuelve en función de si la expresión es true o false |
||||

*Ejemplo*

Este ejemplo devuelve `"yes"` porque la expresión especificada devuelve true.
En caso contrario, el ejemplo devuelve `"no"`:

```
if(equals(1, 1), 'yes', 'no')
```

<a name="indexof"></a>

### <a name="indexof"></a>indexOf

Devuelve la posición de inicio o valor de índice de una subcadena.
Esta función no distingue entre mayúsculas y minúsculas y los índices comienzan por el número 0.

```
indexOf('<text>', '<searchText>')
```

| Parámetro | Obligatorio | type | DESCRIPCIÓN |
| --------- | -------- | ---- | ----------- |
| <*texto*> | Sí | String | Cadena que contiene la subcadena que se va a buscar |
| <*texto_a_buscar*> | Sí | String | Subcadena que se va a buscar |
|||||

| Valor devuelto | type | DESCRIPCIÓN |
| ------------ | ---- | ----------- |
| <*valor-de-índice*>| Integer | Posición de inicio o valor de índice de la subcadena especificada. <p>Si no se encuentra la cadena, devuelve el número -1. |
||||

*Ejemplo*

Este ejemplo busca el valor de índice inicial de la subcadena "world" en la cadena "hello world":

```
indexOf('hello world', 'world')
```

Y devuelve este resultado: `6`

<a name="int"></a>

### <a name="int"></a>int

Devuelve la versión como número entero de una cadena.

```
int('<value>')
```

| Parámetro | Obligatorio | type | DESCRIPCIÓN |
| --------- | -------- | ---- | ----------- |
| <*valor*> | Sí | String | Cadena que se va a convertir |
|||||

| Valor devuelto | type | DESCRIPCIÓN |
| ------------ | ---- | ----------- |
| <*resultado-de-número-entero*> | Integer | Versión como número entero de la cadena especificada. |
||||

*Ejemplo*

Este ejemplo crea una versión como número entero de la cadena "10":

```
int('10')
```

Y devuelve este resultado: `10`

<a name="item"></a>

### <a name="item"></a>item

Cuando se usa dentro de una acción de repetición sobre una matriz, devuelve el elemento actual de la matriz durante la iteración actual de la acción.
También puede obtener los valores de las propiedades de ese elemento.

```
item()
```

| Valor devuelto | type | DESCRIPCIÓN |
| ------------ | ---- | ----------- |
| <*elemento-de-la-matriz-actual*> | Any | Elemento actual de la matriz para la iteración actual de la acción |
||||

*Ejemplo*

Este ejemplo obtiene el elemento `body` del mensaje actual para la acción "Send_an_email" dentro de la iteración actual de un bucle for-each:

```
item().body
```

<a name="items"></a>

### <a name="items"></a>items

Devuelve el elemento actual de cada ciclo de un bucle for-each.
Utilice esta función dentro del bucle for-each.

```
items('<loopName>')
```

| Parámetro | Obligatorio | type | DESCRIPCIÓN |
| --------- | -------- | ---- | ----------- |
| <*nombre_del_bucle*> | Sí | String | Nombre del bucle for-each |
|||||

| Valor devuelto | type | DESCRIPCIÓN |
| ------------ | ---- | ----------- |
| <*elemento*> | Any | Elemento del ciclo actual del bucle for-each especificado |
||||

*Ejemplo*

Este ejemplo obtiene el elemento actual del bucle for-each especificado:

```
items('myForEachLoopName')
```

<a name="iterationIndexes"></a>

### <a name="iterationindexes"></a>iterationIndexes

Se devuelve el valor de índice de la iteración actual dentro de un bucle Until. Puede usar esta función dentro de bucles Until anidados. 

```
iterationIndexes('<loopName>')
```

| Parámetro | Obligatorio | type | DESCRIPCIÓN | 
| --------- | -------- | ---- | ----------- | 
| <*nombre_del_bucle*> | Sí | String | El nombre del bucle Until | 
||||| 

| Valor devuelto | type | DESCRIPCIÓN | 
| ------------ | ---- | ----------- | 
| <*índice*> | Integer | El valor de índice de la iteración actual dentro del bucle Until especificado | 
|||| 

*Ejemplo* 

Este ejemplo crea una variable de contador y la incrementa en uno durante cada iteración en un bucle Until hasta que el valor del contador llega a cinco. El ejemplo también crea una variable que realiza un seguimiento del índice actual para cada iteración. En el bucle Until, durante cada iteración, el ejemplo incrementa el contador y, a continuación, asigna su valor al valor de índice actual y, a continuación, incrementa el contador. En cualquier momento, puede determinar el número de iteración actual si recupera el valor de índice actual.

```
{
   "actions": {
      "Create_counter_variable": {
         "type": "InitializeVariable",
         "inputs": {
            "variables": [ 
               {
                  "name": "myCounter",
                  "type": "Integer",
                  "value": 0
               }
            ]
         },
         "runAfter": {}
      },
      "Create_current_index_variable": {
         "type": "InitializeVariable",
         "inputs": {
            "variables": [
               {
                  "name": "myCurrentLoopIndex",
                  "type": "Integer",
                  "value": 0
               }
            ]
         },
         "runAfter": {
            "Create_counter_variable": [ "Succeeded" ]
         }
      },
      "Until": {
         "type": "Until",
         "actions": {
            "Assign_current_index_to_counter": {
               "type": "SetVariable",
               "inputs": {
                  "name": "myCurrentLoopIndex",
                  "value": "@variables('myCounter')"
               },
               "runAfter": {
                  "Increment_variable": [ "Succeeded" ]
               }
            },
            "Increment_variable": {
               "type": "IncrementVariable",
               "inputs": {
                  "name": "myCounter",
                  "value": 1
               },
               "runAfter": {}
            }
         },
         "expression": "@equals(variables('myCounter'), 5),
         "limit": {
            "count": 60,
            "timeout": "PT1H"
         },
         "runAfter": {
            "Create_current_index_variable": [ "Succeeded" ]
         }
      }
   }
}
```

<a name="json"></a>

### <a name="json"></a>json

Devuelve el valor o el objeto de tipo Notación de objetos JavaScript (JSON) de una cadena o XML.

```
json('<value>')
```

| Parámetro | Obligatorio | type | DESCRIPCIÓN |
| --------- | -------- | ---- | ----------- |
| <*valor*> | Sí | String o XML | Cadena o XML que se va a convertir |
|||||

| Valor devuelto | type | DESCRIPCIÓN |
| ------------ | ---- | ----------- |
| <*resultado-JSON*> | Objeto o tipo nativo de JSON | Objeto o valor de tipo nativo de JSON de la cadena o XML especificados. Si la cadena es NULL, la función devuelve un objeto vacío. |
||||

*Ejemplo 1*

Este ejemplo convierte esta cadena en el valor JSON:

```
json('[1, 2, 3]')
```

Y devuelve este resultado: `[1, 2, 3]`

*Ejemplo 2*

Este ejemplo convierte esta cadena a JSON:

```
json('{"fullName": "Sophia Owen"}')
```

Y devuelve este resultado:

```
{
  "fullName": "Sophia Owen"
}
```

*Ejemplo 3*

Este ejemplo convierte este XML a JSON:

```
json(xml('<?xml version="1.0"?> <root> <person id='1'> <name>Sophia Owen</name> <occupation>Engineer</occupation> </person> </root>'))
```

Y devuelve este resultado:

```json
{
   "?xml": { "@version": "1.0" },
   "root": {
      "person": [ {
         "@id": "1",
         "name": "Sophia Owen",
         "occupation": "Engineer"
      } ]
   }
}
```

<a name="intersection"></a>

### <a name="intersection"></a>intersección

Devuelve una colección que tiene *solo* los elementos comunes en las colecciones especificadas.
Para que aparezca en el resultado, un elemento debe aparecer en todas las colecciones que se pasan a esta función.
Si uno o más elementos tienen el mismo nombre, el último elemento con ese nombre aparece en el resultado.

```
intersection([<collection1>], [<collection2>], ...)
intersection('<collection1>', '<collection2>', ...)
```

| Parámetro | Obligatorio | type | DESCRIPCIÓN |
| --------- | -------- | ---- | ----------- |
| <*colección1*>, <*colección2*>, ... | Sí | Matriz u objeto, pero no ambos | Colecciones de las que desea *solo* los elementos comunes |
|||||

| Valor devuelto | type | DESCRIPCIÓN |
| ------------ | ---- | ----------- |
| <*elementos-comunes*> | Matriz u objeto, respectivamente | Colección que tiene solo los elementos comunes en las colecciones especificadas |
||||

*Ejemplo*

Este ejemplo busca los elementos comunes en estas matrices:

```
intersection(createArray(1, 2, 3), createArray(101, 2, 1, 10), createArray(6, 8, 1, 2))
```

Y devuelve una matriz con *solo* estos elementos: `[1, 2]`

<a name="join"></a>

### <a name="join"></a>join

Devuelve una cadena que tiene todos los elementos de una matriz y tiene cada carácter separado por un *delimitador*.

```
join([<collection>], '<delimiter>')
```

| Parámetro | Obligatorio | type | DESCRIPCIÓN |
| --------- | -------- | ---- | ----------- |
| <*colección*> | Sí | Array | Matriz que tiene los elementos que se van a unir |
| <*delimitador*> | Sí | String | Separador que aparece entre cada carácter de la cadena resultante |
|||||

| Valor devuelto | type | DESCRIPCIÓN |
| ------------ | ---- | ----------- |
| <*carácter1*><*delimitador*><*carácter2*><*delimitador*>... | String | Cadena resultante creada a partir de todos los elementos de la matriz especificada |
||||

*Ejemplo*

Este ejemplo crea una cadena a partir de todos los elementos de esta matriz con el carácter especificado como delimitador:

```
join(createArray('a', 'b', 'c'), '.')
```

Y devuelve este resultado: `"a.b.c"`

<a name="last"></a>

### <a name="last"></a>last

Devuelve el último elemento de una colección.

```
last('<collection>')
last([<collection>])
```

| Parámetro | Obligatorio | type | DESCRIPCIÓN |
| --------- | -------- | ---- | ----------- |
| <*colección*> | Sí | Cadena o matriz | Colección en la que buscar el último elemento |
|||||

| Valor devuelto | type | DESCRIPCIÓN |
| ------------ | ---- | ----------- |
| <*último-elemento-de-la-colección*> | Cadena o matriz, respectivamente | Último elemento de la colección |
||||

*Ejemplo*

Estos ejemplos buscan el último elemento de estas colecciones:

```
last('abcd')
last(createArray(0, 1, 2, 3))
```

Y devuelve estos resultados:

* Primer ejemplo: `"d"`
* Segundo ejemplo: `3`

<a name="lastindexof"></a>

### <a name="lastindexof"></a>lastIndexOf

Devuelve la posición inicial o el valor de índice de la última repetición de una subcadena.
Esta función no distingue entre mayúsculas y minúsculas y los índices comienzan por el número 0.

```
lastIndexOf('<text>', '<searchText>')
```

| Parámetro | Obligatorio | type | DESCRIPCIÓN |
| --------- | -------- | ---- | ----------- |
| <*texto*> | Sí | String | Cadena que contiene la subcadena que se va a buscar |
| <*texto_a_buscar*> | Sí | String | Subcadena que se va a buscar |
|||||

| Valor devuelto | type | DESCRIPCIÓN |
| ------------ | ---- | ----------- |
| <*valor-de-índice-final*> | Integer | Posición inicial o valor de índice de la última repetición de la subcadena especificada. <p>Si no se encuentra la cadena, devuelve el número -1. |
||||

*Ejemplo*

Este ejemplo busca el valor de índice inicial de la última repetición de la subcadena "world" en la cadena "hello world":

```
lastIndexOf('hello world', 'world')
```

Y devuelve este resultado: `6`

<a name="length"></a>

### <a name="length"></a>length

Devuelve el número de elementos de una colección.

```
length('<collection>')
length([<collection>])
```

| Parámetro | Obligatorio | type | DESCRIPCIÓN |
| --------- | -------- | ---- | ----------- |
| <*colección*> | Sí | Cadena o matriz | Colección con los elementos que se van a contar |
|||||

| Valor devuelto | type | DESCRIPCIÓN |
| ------------ | ---- | ----------- |
| <*longitud-o-recuento*> | Integer | Número de elementos de la colección |
||||

*Ejemplo*

Estos ejemplos cuentan el número de elementos de estas colecciones:

```
length('abcd')
length(createArray(0, 1, 2, 3))
```

Y devuelven este resultado: `4`

<a name="less"></a>

### <a name="less"></a>less

Comprueba si el primer valor es menor que el segundo.
Devuelve true cuando el primer valor es menor o devuelve false cuando el primer valor es mayor.

```
less(<value>, <compareTo>)
less('<value>', '<compareTo>')
```

| Parámetro | Obligatorio | type | DESCRIPCIÓN |
| --------- | -------- | ---- | ----------- |
| <*valor*> | Sí | Integer, Float o String | Primer valor para comprobar si es menor que el segundo valor |
| <*comparar_con*> | Sí | Integer, Float o String, respectivamente | Elemento de comparación |
|||||

| Valor devuelto | type | DESCRIPCIÓN |
| ------------ | ---- | ----------- |
| true o false | Boolean | Devuelve true si el primer valor es menor que el segundo valor. Devuelve false si el primer valor es igual o mayor que el segundo. |
||||

*Ejemplo*

Estos ejemplos comprueban si el primer valor es menor que el segundo.

```
less(5, 10)
less('banana', 'apple')
```

Y devuelven estos resultados:

* Primer ejemplo: `true`
* Segundo ejemplo: `false`

<a name="lessOrEquals"></a>

### <a name="lessorequals"></a>lessOrEquals

Comprueba si el primer valor es menor o igual que el segundo.
Devuelve true cuando el primer valor es menor o igual o devuelve false cuando el primer valor es mayor.

```
lessOrEquals(<value>, <compareTo>)
lessOrEquals('<value>', '<compareTo>')
```

| Parámetro | Obligatorio | type | DESCRIPCIÓN |
| --------- | -------- | ---- | ----------- |
| <*valor*> | Sí | Integer, Float o String | Primer valor para comprobar si es menor o igual que el segundo valor |
| <*comparar_con*> | Sí | Integer, Float o String, respectivamente | Elemento de comparación |
|||||

| Valor devuelto | type | DESCRIPCIÓN |
| ------------ | ---- | ----------- |
| true o false  | Boolean | Devuelve true si el primer valor es menor o igual que el segundo. Devuelve false si el primer valor es mayor que el segundo. |
||||

*Ejemplo*

Estos ejemplos comprueban si el primer valor es menor o igual que el segundo.

```
lessOrEquals(10, 10)
lessOrEquals('apply', 'apple')
```

Y devuelven estos resultados:

* Primer ejemplo: `true`
* Segundo ejemplo: `false`

<a name="listCallbackUrl"></a>

### <a name="listcallbackurl"></a>listCallbackUrl

Devuelve la "dirección URL de devolución de llamada" que llama a un desencadenador o una acción.
Esta función solo funciona con desencadenadores y acciones para los tipos de conector **HttpWebhook** y **ApiConnectionWebhook**, pero no para los tipos **Manual**,  **Recurrence**, **HTTP** y **APIConnection**.

```
listCallbackUrl()
```

| Valor devuelto | type | DESCRIPCIÓN |
| ------------ | ---- | ----------- |
| <*Dirección-URL-de-devolución-de-llamada*> | String | Dirección URL de devolución de llamada para un desencadenador o una acción |
||||

*Ejemplo*

Este ejemplo muestra una dirección URL de devolución de llamada de ejemplo que podría devolver esta función:

`"https://prod-01.westus.logic.azure.com:443/workflows/<*workflow-ID*>/triggers/manual/run?api-version=2016-10-01&sp=%2Ftriggers%2Fmanual%2Frun&sv=1.0&sig=<*signature-ID*>"`

<a name="max"></a>

### <a name="max"></a>max

Devuelve el valor más alto de una lista o matriz con números, incluyendo ambos extremos.

```
max(<number1>, <number2>, ...)
max([<number1>, <number2>, ...])
```

| Parámetro | Obligatorio | type | DESCRIPCIÓN |
| --------- | -------- | ---- | ----------- |
| <*número1*>, <*número2*>,... | Sí | Integer, Float o ambos | Conjunto de números del que se desea obtener el valor más alto |
| [<*número2*>, <*número2*>,...] | Sí | Matriz: Integer, Float o ambos | Matriz de números de la que se desea obtener el valor más alto |
|||||

| Valor devuelto | type | DESCRIPCIÓN |
| ------------ | ---- | ----------- |
| <*valor-máximo*> | Integer o Float | Valor más alto en la matriz o conjunto de números especificados |
||||

*Ejemplo*

Estos ejemplos obtienen el valor más alto del conjunto de números y la matriz:

```
max(1, 2, 3)
max(createArray(1, 2, 3))
```

Y devuelven este resultado: `3`

<a name="min"></a>

### <a name="min"></a>Min

Devuelve el valor más bajo de un conjunto de números o una matriz.

```
min(<number1>, <number2>, ...)
min([<number1>, <number2>, ...])
```

| Parámetro | Obligatorio | type | DESCRIPCIÓN |
| --------- | -------- | ---- | ----------- |
| <*número1*>, <*número2*>,... | Sí | Integer, Float o ambos | Conjunto de números del que se desea obtener el valor más bajo |
| [<*número2*>, <*número2*>,...] | Sí | Matriz: Integer, Float o ambos | Matriz de números de la que se desea obtener el valor más bajo |
|||||

| Valor devuelto | type | DESCRIPCIÓN |
| ------------ | ---- | ----------- |
| <*valor-mínimo*> | Integer o Float | Valor más bajo de la matriz o conjunto de números especificados |
||||

*Ejemplo*

Estos ejemplos obtienen el valor más bajo del conjunto de números y la matriz:

```
min(1, 2, 3)
min(createArray(1, 2, 3))
```

Y devuelven este resultado: `1`

<a name="mod"></a>

### <a name="mod"></a>mod

Devuelve el resto de dividir dos números.
Para obtener el resultado de número entero, consulte [div()](#div).

```
mod(<dividend>, <divisor>)
```

| Parámetro | Obligatorio | type | DESCRIPCIÓN |
| --------- | -------- | ---- | ----------- |
| <*dividendo*> | Sí | Integer o Float | Número que se va a dividir entre el *divisor*. |
| <*divisor*> | Sí | Integer o Float | Número que divide el *dividendo*, pero no puede ser 0. |
|||||

| Valor devuelto | type | DESCRIPCIÓN |
| ------------ | ---- | ----------- |
| <*módulo-resultante*> | Integer o Float | Resto de dividir el primer número entre el segundo número |
||||

*Ejemplo*

Este ejemplo divide el primer número entre el segundo número:

```
mod(3, 2)
```

Y devuelven este resultado: `1`

<a name="mul"></a>

### <a name="mul"></a>mul

Devuelve el producto de multiplicar dos números.

```
mul(<multiplicand1>, <multiplicand2>)
```

| Parámetro | Obligatorio | type | DESCRIPCIÓN |
| --------- | -------- | ---- | ----------- |
| <*multiplicando1*> | Sí | Integer o Float | Número que se va a multiplicar por el *multiplicando2* |
| <*multiplicando2*> | Sí | Integer o Float | Número que multiplica al *multiplicando1* |
|||||

| Valor devuelto | type | DESCRIPCIÓN |
| ------------ | ---- | ----------- |
| <*producto-resultante*> | Integer o Float | Producto de multiplicar el primer número por el segundo número |
||||

*Ejemplo*

Estos ejemplos multiplican el primer número por el segundo número:

```
mul(1, 2)
mul(1.5, 2)
```

Y devuelven estos resultados:

* Primer ejemplo: `2`
* Segundo ejemplo: `3`

<a name="multipartBody"></a>

### <a name="multipartbody"></a>multipartBody

Devuelve el elemento body de una parte específica de la salida de una acción que consta de varias partes.

```
multipartBody('<actionName>', <index>)
```

| Parámetro | Obligatorio | type | DESCRIPCIÓN |
| --------- | -------- | ---- | ----------- |
| <*nombre_de_la_acción*> | Sí | String | Nombre de la acción cuya salida tiene varias partes |
| <*índice*> | Sí | Integer | Valor de índice de la parte que se busca |
|||||

| Valor devuelto | type | DESCRIPCIÓN |
| ------------ | ---- | ----------- |
| <*elemento_body*> | String | Elemento body de la parte especificada |
||||

<a name="not"></a>

### <a name="not"></a>not

Comprueba si una expresión es false.
Devuelve verdadero cuando la expresión es falsa o devuelve false cuando es verdadera.

```json
not(<expression>)
```

| Parámetro | Obligatorio | type | DESCRIPCIÓN |
| --------- | -------- | ---- | ----------- |
| <*expresión*> | Sí | Boolean | Expresión que se va a evaluar |
|||||

| Valor devuelto | type | DESCRIPCIÓN |
| ------------ | ---- | ----------- |
| true o false | Boolean | Devuelve true cuando la expresión es falsa. Devuelve false cuando la expresión es verdadera. |
||||

*Ejemplo 1*

Estos ejemplos comprueban si las expresiones especificadas son falsas:

```json
not(false)
not(true)
```

Y devuelven estos resultados:

* Primer ejemplo: la expresión es falsa, por lo que la función devuelve `true`.
* Segundo ejemplo: la expresión es verdadera, por lo que la función devuelve `false`.

*Ejemplo 2*

Estos ejemplos comprueban si las expresiones especificadas son falsas:

```json
not(equals(1, 2))
not(equals(1, 1))
```

Y devuelven estos resultados:

* Primer ejemplo: la expresión es falsa, por lo que la función devuelve `true`.
* Segundo ejemplo: la expresión es verdadera, por lo que la función devuelve `false`.

<a name="or"></a>

### <a name="or"></a>o

Comprueba si al menos una expresión es true.
Devuelve true cuando al menos una expresión es verdadera o devuelve false cuanto todas son falsas.

```
or(<expression1>, <expression2>, ...)
```

| Parámetro | Obligatorio | type | DESCRIPCIÓN |
| --------- | -------- | ---- | ----------- |
| <*expresión1*>, <*expresión2*>, ... | Sí | Boolean | Expresiones que se van a comprobar |
|||||

| Valor devuelto | type | DESCRIPCIÓN |
| ------------ | ---- | ----------- |
| true o false | Boolean | Devuelve true cuando al menos una expresión es verdadera. Devuelve false si todas las expresiones son falsas. |
||||

*Ejemplo 1*

Estos ejemplos comprueban si al menos una expresión es verdadera:

```json
or(true, false)
or(false, false)
```

Y devuelven estos resultados:

* Primer ejemplo: al menos una expresión es verdadera, por lo que la función devuelve `true`.
* Segundo ejemplo: ambas expresiones son falsas, por lo que la función devuelve `false`.

*Ejemplo 2*

Estos ejemplos comprueban si al menos una expresión es verdadera:

```json
or(equals(1, 1), equals(1, 2))
or(equals(1, 2), equals(1, 3))
```

Y devuelven estos resultados:

* Primer ejemplo: al menos una expresión es verdadera, por lo que la función devuelve `true`.
* Segundo ejemplo: ambas expresiones son falsas, por lo que la función devuelve `false`.

<a name="outputs"></a>

### <a name="outputs"></a>outputs

Devuelve las salidas de una acción en tiempo de ejecución. Use esta función, en lugar de `actionOutputs()`, que se resuelve en `outputs()` en el diseñador de aplicación lógica. Aunque ambas funciones funcionan del mismo modo, `outputs()` es preferible.

```
outputs('<actionName>')
```

| Parámetro | Obligatorio | type | DESCRIPCIÓN |
| --------- | -------- | ---- | ----------- |
| <*nombre_de_la_acción*> | Sí | String | Nombre de la acción cuya salida desea |
|||||

| Valor devuelto | type | DESCRIPCIÓN |
| ------------ | -----| ----------- |
| <*salida*> | String | Salida de la acción especificada |
||||

*Ejemplo*

Este ejemplo obtiene la salida de la acción de Twitter `Get user`:

```
outputs('Get_user')
```

Y devuelve este resultado:

```json
{
  "statusCode": 200,
  "headers": {
    "Pragma": "no-cache",
    "Vary": "Accept-Encoding",
    "x-ms-request-id": "a916ec8f52211265d98159adde2efe0b",
    "X-Content-Type-Options": "nosniff",
    "Timing-Allow-Origin": "*",
    "Cache-Control": "no-cache",
    "Date": "Mon, 09 Apr 2018 18:47:12 GMT",
    "Set-Cookie": "ARRAffinity=b9400932367ab5e3b6802e3d6158afffb12fcde8666715f5a5fbd4142d0f0b7d;Path=/;HttpOnly;Domain=twitter-wus.azconn-wus.p.azurewebsites.net",
    "X-AspNet-Version": "4.0.30319",
    "X-Powered-By": "ASP.NET",
    "Content-Type": "application/json; charset=utf-8",
    "Expires": "-1",
    "Content-Length": "339"
  },
  "body": {
    "FullName": "Contoso Corporation",
    "Location": "Generic Town, USA",
    "Id": 283541717,
    "UserName": "ContosoInc",
    "FollowersCount": 172,
    "Description": "Leading the way in transforming the digital workplace.",
    "StatusesCount": 93,
    "FriendsCount": 126,
    "FavouritesCount": 46,
    "ProfileImageUrl": "https://pbs.twimg.com/profile_images/908820389907722240/gG9zaHcd_400x400.jpg"
  }
}
```

<a name="parameters"></a>

### <a name="parameters"></a>parameters

Se devuelve el valor de un parámetro que se describe en la definición de flujo de trabajo.

```
parameters('<parameterName>')
```

| Parámetro | Obligatorio | type | DESCRIPCIÓN |
| --------- | -------- | ---- | ----------- |
| <*nombre_del_parámetro*> | Sí | String | Nombre del parámetro cuyo valor busca |
|||||

| Valor devuelto | type | DESCRIPCIÓN |
| ------------ | ---- | ----------- |
| <*valor-del-parámetro*> | Any | Valor del parámetro especificado |
||||

*Ejemplo*

Suponga que tiene este valor JSON:

```json
{
  "fullName": "Sophia Owen"
}
```

Este ejemplo obtiene el valor del parámetro especificado:

```
parameters('fullName')
```

Y devuelve este resultado: `"Sophia Owen"`

<a name="rand"></a>

### <a name="rand"></a>rand

Devuelve un entero aleatorio de un intervalo especificado, que incluye solo el extremo inicial.

```
rand(<minValue>, <maxValue>)
```

| Parámetro | Obligatorio | type | DESCRIPCIÓN |
| --------- | -------- | ---- | ----------- |
| <*valor_mínimo*> | Sí | Integer | Entero más bajo del intervalo |
| <*valor_maximo*> | Sí | Integer | Entero que sigue al entero más alto del intervalo que puede devolver la función |
|||||

| Valor devuelto | type | DESCRIPCIÓN |
| ------------ | ---- | ----------- |
| <*resultado-aleatorio*> | Integer | Entero aleatorio devuelto del intervalo especificado |
||||

*Ejemplo*

Este ejemplo obtiene un entero aleatorio del intervalo especificado, sin incluir el valor máximo:

```
rand(1, 5)
```

Y devuelve uno de estos números como resultado: `1`, `2`, `3` o `4`

<a name="range"></a>

### <a name="range"></a>range

Devuelve una matriz de enteros que comienza en un entero especificado.

```
range(<startIndex>, <count>)
```

| Parámetro | Obligatorio | type | DESCRIPCIÓN |
| --------- | -------- | ---- | ----------- |
| <*índice_inicial*> | Sí | Integer | Valor entero que inicia la matriz como el primer elemento |
| <*recuento*> | Sí | Integer | Número de enteros de la matriz |
|||||

| Valor devuelto | type | DESCRIPCIÓN |
| ------------ | ---- | ----------- |
| [<*intervalo-resultado*>] | Array | Matriz de enteros comenzando a partir del índice especificado |
||||

*Ejemplo*

Este ejemplo crea una matriz de enteros que comienza en el índice especificado y tiene el número especificado de enteros:

```
range(1, 4)
```

Y devuelve este resultado: `[1, 2, 3, 4]`

<a name="replace"></a>

### <a name="replace"></a>replace

Reemplaza una subcadena por la cadena especificada y devuelve la cadena resultante. Esta función distingue mayúsculas de minúsculas.

```
replace('<text>', '<oldText>', '<newText>')
```

| Parámetro | Obligatorio | type | DESCRIPCIÓN |
| --------- | -------- | ---- | ----------- |
| <*texto*> | Sí | String | Cadena que contiene la subcadena que se va a reemplazar |
| <*texto_anterior*> | Sí | String | Subcadena que se va a reemplazar |
| <*texto_nuevo*> | Sí | String | Cadena de sustitución |
|||||

| Valor devuelto | type | DESCRIPCIÓN |
| ------------ | ---- | ----------- |
| <*texto-actualizado*> | String | Cadena actualizada después de reemplazar la subcadena <p>Si no se encuentra la subcadena, devuelve la cadena original. |
||||

*Ejemplo*

Este ejemplo busca la subcadena "old" en "the old string" y reemplaza "old" por "new":

```
replace('the old string', 'old', 'new')
```

Y devuelve este resultado: `"the new string"`

<a name="removeProperty"></a>

### <a name="removeproperty"></a>removeProperty

Elimina una propiedad de un objeto y devuelve el objeto actualizado. Si la propiedad que intenta quitar no existe, la función devuelve el objeto original.

```
removeProperty(<object>, '<property>')
```

| Parámetro | Obligatorio | type | DESCRIPCIÓN |
| --------- | -------- | ---- | ----------- |
| <*objeto*> | Sí | Object | Objeto JSON del que desea eliminar una propiedad |
| <*propiedad*> | Sí | String | Nombre de la propiedad que se va a eliminar |
|||||

| Valor devuelto | type | DESCRIPCIÓN |
| ------------ | ---- | ----------- |
| <*objeto-actualizado*> | Object | Objeto JSON actualizado sin la propiedad especificada |
||||

Para quitar una propiedad secundaria de una propiedad existente, use esta sintaxis:

```
removeProperty(<object>['<parent-property>'], '<child-property>')
```

| Parámetro | Obligatorio | type | DESCRIPCIÓN |
| --------- | -------- | ---- | ----------- |
| <*objeto*> | Sí | Object | Objeto JSON cuya propiedad desea quitar |
| <*parent-property*> | Sí | Cadena | Nombre de la propiedad primaria con la propiedad secundaria que desea quitar |
| <*child-property*> | Sí | Cadena | Nombre de la propiedad secundaria que se va a quitar |
|||||

| Valor devuelto | type | DESCRIPCIÓN |
| ------------ | ---- | ----------- |
| <*objeto-actualizado*> | Object | Objeto JSON actualizado cuya propiedad secundaria ha quitado. |
||||

*Ejemplo 1*

En este ejemplo se quita la propiedad `middleName` de un objeto JSON, que se convierte de cadena a JSON con la función [JSON()](#json) y devuelve el objeto actualizado:

```
removeProperty(json('{ "firstName": "Sophia", "middleName": "Anne", "surName": "Owen" }'), 'middleName')
```

Este es el objeto JSON actual:

```json
{
   "firstName": "Sophia",
   "middleName": "Anne",
   "surName": "Owen"
}
```

Este es el objeto JSON actualizado:

```json
{
   "firstName": "Sophia",
   "surName": "Owen"
}
```

*Ejemplo 2*

En este ejemplo se quita la propiedad secundaria `middleName` de una propiedad primaria `customerName` en un objeto JSON, que se convierte de cadena a JSON mediante la función [JSON()](#json) y devuelve el objeto actualizado:

```
removeProperty(json('{ "customerName": { "firstName": "Sophia", "middleName": "Anne", "surName": "Owen" } }')['customerName'], 'middleName')
```

Este es el objeto JSON actual:

```json
{
   "customerName": {
      "firstName": "Sophia",
      "middleName": "Anne",
      "surName": "Owen"
   }
}
```

Este es el objeto JSON actualizado:

```json
{
   "customerName": {
      "firstName": "Sophia",
      "surName": "Owen"
   }
}
```

<a name="result"></a>

### <a name="result"></a>result

Devuelve las entradas y salidas de todas las acciones que se encuentran dentro de la acción de ámbito especificada, como una acción `For_each`, `Until` o `Scope`. Esta función resulta útil para devolver los resultados de una acción con errores para diagnosticar y controlar las excepciones. Para obtener más información, vea [Obtención del contexto y resultados de errores](../logic-apps/logic-apps-exception-handling.md#get-results-from-failures).

```
result('<scopedActionName>')
```

| Parámetro | Obligatorio | type | DESCRIPCIÓN |
| --------- | -------- | ---- | ----------- |
| <*scopedActionName*> | Sí | Cadena | Nombre de la acción con ámbito desde la que se van a devolver las entradas y salidas de todas las acciones internas |
||||

| Valor devuelto | type | DESCRIPCIÓN |
| ------------ | ---- | ----------- |
| <*array-object*> | Objeto de matriz | Matriz que contiene matrices de entradas y salidas de cada acción que aparece dentro de la acción con ámbito especificada |
||||

*Ejemplo*

En este ejemplo se devuelven las entradas y salidas de cada iteración de una acción HTTP que está dentro de un bucle `For_each` mediante la función `result()` en la acción `Compose`:

```json
{
   "actions": {
      "Compose": {
         "inputs": "@result('For_each')",
         "runAfter": {
            "For_each": [
               "Succeeded"
            ]
         },
         "type": "compose"
      },
      "For_each": {
         "actions": {
            "HTTP": {
               "inputs": {
                  "method": "GET",
                  "uri": "https://httpstat.us/200"
               },
               "runAfter": {},
               "type": "Http"
            }
         },
         "foreach": "@triggerBody()",
         "runAfter": {},
         "type": "Foreach"
      }
   }
}
```

Este es el aspecto que podría tener el ejemplo de matriz devuelta, donde el objeto `outputs` externo contiene las entradas y salidas de cada iteración de las acciones dentro de la acción `For_each`.

```json
[
   {
      "name": "HTTP",
      "outputs": [
         {
            "name": "HTTP",
            "inputs": {
               "uri": "https://httpstat.us/200",
               "method": "GET"
            },
            "outputs": {
               "statusCode": 200,
               "headers": {
                   "X-AspNetMvc-Version": "5.1",
                   "Access-Control-Allow-Origin": "*",
                   "Cache-Control": "private",
                   "Date": "Tue, 20 Aug 2019 22:15:37 GMT",
                   "Set-Cookie": "ARRAffinity=0285cfbea9f2ee7",
                   "Server": "Microsoft-IIS/10.0",
                   "X-AspNet-Version": "4.0.30319",
                   "X-Powered-By": "ASP.NET",
                   "Content-Length": "0"
               },
               "startTime": "2019-08-20T22:15:37.6919631Z",
               "endTime": "2019-08-20T22:15:37.95762Z",
               "trackingId": "6bad3015-0444-4ccd-a971-cbb0c99a7.....",
               "clientTrackingId": "085863526764.....",
               "code": "OK",
               "status": "Succeeded"
            }
         },
         {
            "name": "HTTP",
            "inputs": {
               "uri": "https://httpstat.us/200",
               "method": "GET"
            },
            "outputs": {
            "statusCode": 200,
               "headers": {
                   "X-AspNetMvc-Version": "5.1",
                   "Access-Control-Allow-Origin": "*",
                   "Cache-Control": "private",
                   "Date": "Tue, 20 Aug 2019 22:15:37 GMT",
                   "Set-Cookie": "ARRAffinity=0285cfbea9f2ee7",
                   "Server": "Microsoft-IIS/10.0",
                   "X-AspNet-Version": "4.0.30319",
                   "X-Powered-By": "ASP.NET",
                   "Content-Length": "0"
               },
               "startTime": "2019-08-20T22:15:37.6919631Z",
               "endTime": "2019-08-20T22:15:37.95762Z",
               "trackingId": "9987e889-981b-41c5-aa27-f3e0e59bf69.....",
               "clientTrackingId": "085863526764.....",
               "code": "OK",
               "status": "Succeeded"
            }
         }
      ]
   }
]
```

<a name="setProperty"></a>

### <a name="setproperty"></a>setProperty

Establece el valor de propiedad de un objeto JSON y devuelve el objeto actualizado. Si la propiedad que intenta establecer no existe, la propiedad se agrega al objeto. Para agregar una nueva propiedad, utilice la función [addProperty()](#addProperty).

```
setProperty(<object>, '<property>', <value>)
```

| Parámetro | Obligatorio | type | DESCRIPCIÓN |
| --------- | -------- | ---- | ----------- |
| <*objeto*> | Sí | Object | Objeto JSON cuya propiedad desea establecer |
| <*propiedad*> | Sí | String | Nombre de la propiedad existente o nueva que se va a establecer |
| <*valor*> | Sí | Any | Valor que se va a establecer para la propiedad especificada |
|||||

Para establecer la propiedad secundaria en un objeto secundario, use una llamada `setProperty()` anidada en su lugar. De lo contrario, la función devuelve solo el objeto secundario como salida.

```
setProperty(<object>['<parent-property>'], '<parent-property>', setProperty(<object>['parentProperty'], '<child-property>', <value>))
```

| Parámetro | Obligatorio | type | DESCRIPCIÓN |
| --------- | -------- | ---- | ----------- |
| <*objeto*> | Sí | Object | Objeto JSON cuya propiedad desea establecer |
| <*parent-property*> | Sí | Cadena | Nombre de la propiedad primaria con la propiedad secundaria que desea establecer |
| <*child-property*> | Sí | Cadena | Nombre de la propiedad secundaria que va a establecer |
| <*valor*> | Sí | Any | Valor que se va a establecer para la propiedad especificada |
|||||

| Valor devuelto | type | DESCRIPCIÓN |
| ------------ | ---- | ----------- |
| <*objeto-actualizado*> | Object | Objeto JSON actualizado cuya propiedad se ha establecido |
||||

*Ejemplo 1*

En este ejemplo se establece la propiedad `surName` en un objeto JSON, que se convierte de cadena a JSON mediante la función [JSON()](#json). La función asigna el valor especificado a la propiedad y devuelve el objeto actualizado:

```
setProperty(json('{ "firstName": "Sophia", "surName": "Owen" }'), 'surName', 'Hartnett')
```

Este es el objeto JSON actual:

```json
{
   "firstName": "Sophia",
   "surName": "Owen"
}
```

Este es el objeto JSON actualizado:

```json
{
   "firstName": "Sophia",
   "surName": "Hartnett"
}
```

*Ejemplo 2*

En este ejemplo se establece la propiedad secundaria `surName` para la propiedad primaria `customerName` en un objeto JSON, que se convierte de cadena a JSON mediante la función [JSON()](#json). La función asigna el valor especificado a la propiedad y devuelve el objeto actualizado:

```
setProperty(json('{ "customerName": { "firstName": "Sophia", "surName": "Owen" } }'), 'customerName', setProperty(json('{ "customerName": { "firstName": "Sophia", "surName": "Owen" } }')['customerName'], 'surName', 'Hartnett'))
```

Este es el objeto JSON actual:

```json
{
   "customerName": {
      "firstName": "Sophie",
      "surName": "Owen"
   }
}
```

Este es el objeto JSON actualizado:

```json
{
   "customerName": {
      "firstName": "Sophie",
      "surName": "Hartnett"
   }
}
```

<a name="skip"></a>

### <a name="skip"></a>skip

Quita elementos del principio de una colección y devuelve *todos los demás* elementos.

```
skip([<collection>], <count>)
```

| Parámetro | Obligatorio | type | DESCRIPCIÓN |
| --------- | -------- | ---- | ----------- |
| <*colección*> | Sí | Array | Colección cuyos elementos desea eliminar |
| <*recuento*> | Sí | Integer | Entero positivo para el número de elementos a eliminar al principio |
|||||

| Valor devuelto | type | DESCRIPCIÓN |
| ------------ | ---- | ----------- |
| [<*colección-actualizada*>] | Array | Colección actualizada después de eliminar los elementos especificados |
||||

*Ejemplo*

Este ejemplo elimina un elemento, el número 0, del principio de la matriz especificada:

```
skip(createArray(0, 1, 2, 3), 1)
```

Y devuelve esta matriz con los elementos restantes: `[1,2,3]`

<a name="split"></a>

### <a name="split"></a>split

Devuelve una matriz que contiene subcadenas, separadas por comas, en función de un carácter delimitador especificado en la cadena original.

```
split('<text>', '<delimiter>')
```

| Parámetro | Obligatorio | type | DESCRIPCIÓN |
| --------- | -------- | ---- | ----------- |
| <*texto*> | Sí | String | La cadena para separar en subcadenas según el delimitador especificado en la cadena original |
| <*delimitador*> | Sí | String | El carácter de la cadena original que se usará como delimitador |
|||||

| Valor devuelto | type | DESCRIPCIÓN |
| ------------ | ---- | ----------- |
| [<*subcadena1*>, <*subcadena2*>,...] | Array | Una matriz que contiene subcadenas de la cadena original, separadas por comas |
||||

*Ejemplo*

En este ejemplo se crea una matriz con las subcadenas de la cadena especificada según el carácter especificado como delimitador:

```
split('a_b_c', '_')
```

Y devuelve esta matriz como resultado: `["a","b","c"]`

<a name="startOfDay"></a>

### <a name="startofday"></a>startOfDay

Devuelve el inicio del día de una marca de tiempo.

```
startOfDay('<timestamp>', '<format>'?)
```

| Parámetro | Obligatorio | type | DESCRIPCIÓN |
| --------- | -------- | ---- | ----------- |
| <*marca_de_tiempo*> | Sí | String | Cadena que contiene la marca de tiempo |
| <*formato*> | Sin | String | Puede ser un [especificador de formato sencillo](https://docs.microsoft.com/dotnet/standard/base-types/standard-date-and-time-format-strings) o un [patrón de formato personalizado](https://docs.microsoft.com/dotnet/standard/base-types/custom-date-and-time-format-strings). El formato predeterminado de la marca de tiempo es ["o"](https://docs.microsoft.com/dotnet/standard/base-types/standard-date-and-time-format-strings) (aaaa-MM-ddTHH:mm:ss:fffffffK), que cumple con [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) y conserva la información de zona horaria. |
|||||

| Valor devuelto | type | DESCRIPCIÓN |
| ------------ | ---- | ----------- |
| <*marca-de-tiempo-actualizada*> | String | Marca de tiempo especificada, pero a partir de la marca de hora cero del día |
||||

*Ejemplo*

Este ejemplo busca el inicio del día de esta marca de tiempo:

```
startOfDay('2018-03-15T13:30:30Z')
```

Y devuelve este resultado: `"2018-03-15T00:00:00.0000000Z"`

<a name="startOfHour"></a>

### <a name="startofhour"></a>startOfHour

Devuelve el inicio de la hora de una marca de tiempo.

```
startOfHour('<timestamp>', '<format>'?)
```

| Parámetro | Obligatorio | type | DESCRIPCIÓN |
| --------- | -------- | ---- | ----------- |
| <*marca_de_tiempo*> | Sí | String | Cadena que contiene la marca de tiempo |
| <*formato*> | Sin | String | Puede ser un [especificador de formato sencillo](https://docs.microsoft.com/dotnet/standard/base-types/standard-date-and-time-format-strings) o un [patrón de formato personalizado](https://docs.microsoft.com/dotnet/standard/base-types/custom-date-and-time-format-strings). El formato predeterminado de la marca de tiempo es ["o"](https://docs.microsoft.com/dotnet/standard/base-types/standard-date-and-time-format-strings) (aaaa-MM-ddTHH:mm:ss:fffffffK), que cumple con [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) y conserva la información de zona horaria. |
|||||

| Valor devuelto | type | DESCRIPCIÓN |
| ------------ | ---- | ----------- |
| <*marca-de-tiempo-actualizada*> | String | Marca de tiempo especificada, pero a partir de la marca de minuto cero de la hora |
||||

*Ejemplo*

Este ejemplo busca el inicio de la hora de esta marca de tiempo:

```
startOfHour('2018-03-15T13:30:30Z')
```

Y devuelve este resultado: `"2018-03-15T13:00:00.0000000Z"`

<a name="startOfMonth"></a>

### <a name="startofmonth"></a>startOfMonth

Devuelve el inicio del mes de una marca de tiempo.

```
startOfMonth('<timestamp>', '<format>'?)
```

| Parámetro | Obligatorio | type | DESCRIPCIÓN |
| --------- | -------- | ---- | ----------- |
| <*marca_de_tiempo*> | Sí | String | Cadena que contiene la marca de tiempo |
| <*formato*> | Sin | String | Puede ser un [especificador de formato sencillo](https://docs.microsoft.com/dotnet/standard/base-types/standard-date-and-time-format-strings) o un [patrón de formato personalizado](https://docs.microsoft.com/dotnet/standard/base-types/custom-date-and-time-format-strings). El formato predeterminado de la marca de tiempo es ["o"](https://docs.microsoft.com/dotnet/standard/base-types/standard-date-and-time-format-strings) (aaaa-MM-ddTHH:mm:ss:fffffffK), que cumple con [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) y conserva la información de zona horaria. |
|||||

| Valor devuelto | type | DESCRIPCIÓN |
| ------------ | ---- | ----------- |
| <*marca-de-tiempo-actualizada*> | String | Marca de tiempo especificada, pero a partir del primer día del mes en la marca de hora cero |
||||

*Ejemplo*

Este ejemplo devuelve el inicio del mes de esta marca de tiempo:

```
startOfMonth('2018-03-15T13:30:30Z')
```

Y devuelve este resultado: `"2018-03-01T00:00:00.0000000Z"`

<a name="startswith"></a>

### <a name="startswith"></a>startsWith

Comprueba si una cadena comienza con una subcadena especificada.
Devuelve true cuando se encuentra la subcadena o devuelve false cuando no se encuentra.
Esta función no distingue mayúsculas de minúsculas.

```
startsWith('<text>', '<searchText>')
```

| Parámetro | Obligatorio | type | DESCRIPCIÓN |
| --------- | -------- | ---- | ----------- |
| <*texto*> | Sí | String | Cadena que se va a comprobar |
| <*texto_a_buscar*> | Sí | String | Cadena inicial que se va a buscar |
|||||

| Valor devuelto | type | DESCRIPCIÓN |
| ------------ | ---- | ----------- |
| true o false  | Boolean | Devuelve true cuando se encuentra la subcadena inicial. Devuelve false si no se encuentra. |
||||

*Ejemplo 1*

Este ejemplo comprueba si la cadena "hello world" comienza con la subcadena "hello":

```
startsWith('hello world', 'hello')
```

Y devuelve este resultado: `true`

*Ejemplo 2*

Este ejemplo comprueba si la cadena "hello world" comienza con la subcadena "greetings":

```
startsWith('hello world', 'greetings')
```

Y devuelve este resultado: `false`

<a name="string"></a>

### <a name="string"></a>string

Devuelve la versión de cadena de un valor.

```
string(<value>)
```

| Parámetro | Obligatorio | type | DESCRIPCIÓN |
| --------- | -------- | ---- | ----------- |
| <*valor*> | Sí | Any | Valor que se va a convertir |
|||||

| Valor devuelto | type | DESCRIPCIÓN |
| ------------ | ---- | ----------- |
| <*valor-de-cadena*> | String | Versión de cadena del valor especificado |
||||

*Ejemplo 1*

Este ejemplo crea la versión de cadena de este número:

```
string(10)
```

Y devuelve este resultado: `"10"`

*Ejemplo 2*

Este ejemplo crea una cadena a partir del objeto JSON especificado y usa el carácter de barra diagonal inversa (\\) como carácter de escape para la marca de comillas dobles (").

```
string( { "name": "Sophie Owen" } )
```

Y devuelve este resultado: `"{ \\"name\\": \\"Sophie Owen\\" }"`

<a name="sub"></a>

### <a name="sub"></a>sub

Devuelve el resultado de restar el segundo número del primero.

```
sub(<minuend>, <subtrahend>)
```

| Parámetro | Obligatorio | type | DESCRIPCIÓN |
| --------- | -------- | ---- | ----------- |
| <*minuendo*> | Sí | Integer o Float | Número del que se resta el *sustraendo* |
| <*sustraendo*> | Sí | Integer o Float | Número que se resta del *minuendo* |
|||||

| Valor devuelto | type | DESCRIPCIÓN |
| ------------ | ---- | ----------- |
| <*resultado*> | Integer o Float | Resultado de restar el segundo número del primero |
||||

*Ejemplo*

Este ejemplo resta el segundo número del primer número:

```
sub(10.3, .3)
```

Y devuelve este resultado: `10`

<a name="substring"></a>

### <a name="substring"></a>substring

Devuelve caracteres de una cadena a partir de la posición especificada o índice.
Los valores de índice comienzan desde el número 0.

```
substring('<text>', <startIndex>, <length>)
```

| Parámetro | Obligatorio | type | DESCRIPCIÓN |
| --------- | -------- | ---- | ----------- |
| <*texto*> | Sí | String | Cadena cuyos caracteres se toman |
| <*índice_inicial*> | Sí | Integer | Un número positivo igual o mayor que 0 que desea utilizar como el valor de índice o la posición inicial |
| <*longitud*> | Sí | Integer | Número positivo de caracteres que desea incluir en la subcadena |
|||||

| Valor devuelto | type | DESCRIPCIÓN |
| ------------ | ---- | ----------- |
| <*subcadena-resultante*> | String | Subcadena con el número especificado de caracteres, empezando en la posición de índice especificada de la cadena de origen |
||||

*Ejemplo*

Este ejemplo crea una subcadena de cinco caracteres a partir de la cadena especificada, empezando en el valor de índice 6:

```
substring('hello world', 6, 5)
```

Y devuelve este resultado: `"world"`

<a name="subtractFromTime"></a>

### <a name="subtractfromtime"></a>subtractFromTime

Resta un número de unidades de tiempo de una marca de tiempo.
Consulte también [getPastTime](#getPastTime).

```
subtractFromTime('<timestamp>', <interval>, '<timeUnit>', '<format>'?)
```

| Parámetro | Obligatorio | type | DESCRIPCIÓN |
| --------- | -------- | ---- | ----------- |
| <*marca_de_tiempo*> | Sí | String | Cadena que contiene la marca de tiempo |
| <*intervalo*> | Sí | Integer | Número de unidades de tiempo especificadas que se va a sustraer |
| <*unidad_de_tiempo*> | Sí | String | La unidad de tiempo que se usará con *intervalo*: "Segundo", "Minuto", "Hora", "Día", "Semana", "Mes", "Año" |
| <*formato*> | Sin | String | Puede ser un [especificador de formato sencillo](https://docs.microsoft.com/dotnet/standard/base-types/standard-date-and-time-format-strings) o un [patrón de formato personalizado](https://docs.microsoft.com/dotnet/standard/base-types/custom-date-and-time-format-strings). El formato predeterminado de la marca de tiempo es ["o"](https://docs.microsoft.com/dotnet/standard/base-types/standard-date-and-time-format-strings) (aaaa-MM-ddTHH:mm:ss:fffffffK), que cumple con [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) y conserva la información de zona horaria. |
|||||

| Valor devuelto | type | DESCRIPCIÓN |
| ------------ | ---- | ----------- |
| <*marca-de-tiempo-actualizada*> | String | Marca de tiempo menos el número de unidades de tiempo especificado |
||||

*Ejemplo 1*

Este ejemplo resta un día de esta marca de tiempo:

```
subtractFromTime('2018-01-02T00:00:00Z', 1, 'Day')
```

Y devuelve este resultado: `"2018-01-01T00:00:00:0000000Z"`

*Ejemplo 2*

Este ejemplo resta un día de esta marca de tiempo:

```
subtractFromTime('2018-01-02T00:00:00Z', 1, 'Day', 'D')
```

Y devuelve el resultado con el formato "D" opcional: `"Monday, January, 1, 2018"`

<a name="take"></a>

### <a name="take"></a>take

Devuelve elementos del principio de una colección.

```
take('<collection>', <count>)
take([<collection>], <count>)
```

| Parámetro | Obligatorio | type | DESCRIPCIÓN |
| --------- | -------- | ---- | ----------- |
| <*colección*> | Sí | Cadena o matriz | Colección cuyos elementos desea tomar |
| <*recuento*> | Sí | Integer | Entero positivo para el número de elementos a tomar desde el principio |
|||||

| Valor devuelto | type | DESCRIPCIÓN |
| ------------ | ---- | ----------- |
| <*subconjunto*> o [<*subconjunto*>] | Cadena o matriz, respectivamente | Cadena o matriz que tiene el número especificado de elementos tomados desde el principio de la colección original |
||||

*Ejemplo*

Estos ejemplos obtienen el número especificado de elementos desde el principio de estas colecciones:

```
take('abcde', 3)
take(createArray(0, 1, 2, 3, 4), 3)
```

Y devuelven estos resultados:

* Primer ejemplo: `"abc"`
* Segundo ejemplo: `[0, 1, 2]`

<a name="ticks"></a>

### <a name="ticks"></a>ticks

Devuelve el valor de la propiedad `ticks` de una marca de tiempo especificada.
Un *tick* es un intervalo de 100 nanosegundos.

```
ticks('<timestamp>')
```

| Parámetro | Obligatorio | type | DESCRIPCIÓN |
| --------- | -------- | ---- | ----------- |
| <*marca_de_tiempo*> | Sí | String | Cadena de una marca de tiempo |
|||||

| Valor devuelto | type | DESCRIPCIÓN |
| ------------ | ---- | ----------- |
| <*número-de-ticks*> | Integer | Número de ticks desde la marca de tiempo especificada |
||||

<a name="toLower"></a>

### <a name="tolower"></a>toLower

Devuelve una cadena en formato de minúsculas. Si un carácter de la cadena no tiene un equivalente en minúsculas, se incluye sin cambios en la cadena devuelta.

```
toLower('<text>')
```

| Parámetro | Obligatorio | type | DESCRIPCIÓN |
| --------- | -------- | ---- | ----------- |
| <*texto*> | Sí | String | Cadena que se devuelve en formato de minúsculas |
|||||

| Valor devuelto | type | DESCRIPCIÓN |
| ------------ | ---- | ----------- |
| <*texto-en-minúsculas*> | String | Cadena original en minúsculas |
||||

*Ejemplo*

Este ejemplo convierte esta cadena a minúsculas:

```
toLower('Hello World')
```

Y devuelve este resultado: `"hello world"`

<a name="toUpper"></a>

### <a name="toupper"></a>toUpper

Devuelve una cadena en formato de mayúsculas. Si un carácter de la cadena no tiene un equivalente en mayúsculas, se incluye sin cambios en la cadena devuelta.

```
toUpper('<text>')
```

| Parámetro | Obligatorio | type | DESCRIPCIÓN |
| --------- | -------- | ---- | ----------- |
| <*texto*> | Sí | String | Cadena que se devuelve en formato de mayúsculas |
|||||

| Valor devuelto | type | DESCRIPCIÓN |
| ------------ | ---- | ----------- |
| <*texto-en-mayúsculas*> | String | Cadena original en mayúsculas |
||||

*Ejemplo*

Este ejemplo convierte esta cadena a mayúsculas:

```
toUpper('Hello World')
```

Y devuelve este resultado: `"HELLO WORLD"`

<a name="trigger"></a>

### <a name="trigger"></a>trigger

Devuelve la salida en tiempo de ejecución de un desencadenador o los valores de otros pares de nombre y valor JSON, que puede asignar a una expresión.

* Dentro de las entradas del desencadenador, la función devuelve la salida de la ejecución anterior.

* Dentro de la condición del desencadenador, la función devuelve la salida de la ejecución actual.

De forma predeterminada, la función hace referencia al objeto del desencadenador entero, pero también puede especificar una propiedad cuyo valor desee obtener.
Además, esta función tiene versiones abreviadas disponibles, consulte [triggerOutputs()](#triggerOutputs) y [triggerBody()](#triggerBody).

```
trigger()
```

| Valor devuelto | type | DESCRIPCIÓN |
| ------------ | ---- | ----------- |
| <*salida-del-desencadenador*> | String | Salida de un desencadenador en tiempo de ejecución |
||||

<a name="triggerBody"></a>

### <a name="triggerbody"></a>triggerBody

Devuelve la salida `body` de un desencadenador en tiempo de ejecución.
Abreviatura de `trigger().outputs.body`.
Consulte [trigger()](#trigger).

```
triggerBody()
```

| Valor devuelto | type | DESCRIPCIÓN |
| ------------ | ---- | ----------- |
| <*salida-del-elemento-body-del-desencadenador*> | String | Salida del elemento `body` del desencadenador |
||||

<a name="triggerFormDataMultiValues"></a>

### <a name="triggerformdatamultivalues"></a>triggerFormDataMultiValues

Devuelve una matriz con valores que coinciden con un nombre de clave de la salidas *form-data* o *form-encoded* de un desencadenador.

```
triggerFormDataMultiValues('<key>')
```

| Parámetro | Obligatorio | type | DESCRIPCIÓN |
| --------- | -------- | ---- | ----------- |
| <*clave*> | Sí | String | Nombre de la clave cuyo valor busca |
|||||

| Valor devuelto | type | DESCRIPCIÓN |
| ------------ | ---- | ----------- |
| [<*matriz-con-valores-de-clave*>] | Array | Matriz con todos los valores que coinciden con la clave especificada |
||||

*Ejemplo*

Este ejemplo crea una matriz a partir del valor de la clave "feedUrl" de la salidas form-data o form-encoded de un desencadenador de RSS:

```
triggerFormDataMultiValues('feedUrl')
```

Y devuelve esta matriz como un resultado de ejemplo: `["http://feeds.reuters.com/reuters/topNews"]`

<a name="triggerFormDataValue"></a>

### <a name="triggerformdatavalue"></a>triggerFormDataValue

Devuelve una cadena con un valor único que coincide con un nombre de clave en las salidas *form-data* o *form-encoded* de un desencadenador.
Si la función encuentra más de una coincidencia, la función produce un error.

```
triggerFormDataValue('<key>')
```

| Parámetro | Obligatorio | type | DESCRIPCIÓN |
| --------- | -------- | ---- | ----------- |
| <*clave*> | Sí | String | Nombre de la clave cuyo valor busca |
|||||

| Valor devuelto | type | DESCRIPCIÓN |
| ------------ | ---- | ----------- |
| <*key-value*> | String | Valor de la clave especificada |
||||

*Ejemplo*

Este ejemplo crea una cadena a partir del valor de la clave "feedUrl" de la salidas form-data o form-encoded de un desencadenador de RSS:

```
triggerFormDataValue('feedUrl')
```

Y devuelve esta cadena como un resultado de ejemplo: `"http://feeds.reuters.com/reuters/topNews"`

<a name="triggerMultipartBody"></a>

### <a name="triggermultipartbody"></a>triggerMultipartBody

Devuelve el elemento body de una parte específica de la salida de un desencadenador que consta de varias partes.

```
triggerMultipartBody(<index>)
```

| Parámetro | Obligatorio | type | DESCRIPCIÓN |
| --------- | -------- | ---- | ----------- |
| <*índice*> | Sí | Integer | Valor de índice de la parte que se busca |
|||||

| Valor devuelto | type | DESCRIPCIÓN |
| ------------ | ---- | ----------- |
| <*elemento_body*> | String | Elemento body de una parte específica de la salida de varias partes de un desencadenador |
||||

<a name="triggerOutputs"></a>

### <a name="triggeroutputs"></a>triggerOutputs

Devuelve la salida de un desencadenador en tiempo de ejecución o valores de otros pares de nombre y valor JSON.
Abreviatura de `trigger().outputs`.
Consulte [trigger()](#trigger).

```
triggerOutputs()
```

| Valor devuelto | type | DESCRIPCIÓN |
| ------------ | ---- | ----------- |
| <*salida-del-desencadenador*> | String | Salida de un desencadenador en tiempo de ejecución  |
||||

<a name="trim"></a>

### <a name="trim"></a>trim

Quita el espacio en blanco inicial y final de una cadena y devuelve la cadena actualizada.

```
trim('<text>')
```

| Parámetro | Obligatorio | type | DESCRIPCIÓN |
| --------- | -------- | ---- | ----------- |
| <*texto*> | Sí | String | Cadena que contiene los espacios en blanco iniciales y finales que se van a eliminar |
|||||

| Valor devuelto | type | DESCRIPCIÓN |
| ------------ | ---- | ----------- |
| <*texto_actualizado*> | String | Versión actualizada de la cadena original sin espacios en blanco iniciales o finales |
||||

*Ejemplo*

Este ejemplo quita el espacio en blanco inicial y final de la cadena " Hello World ":

```
trim(' Hello World  ')
```

Y devuelve este resultado: `"Hello World"`

<a name="union"></a>

### <a name="union"></a>union

Devuelve una colección que tiene *todos* los elementos de las colecciones especificadas.
Para que aparezca en el resultado, un elemento debe aparecer en alguna de las colecciones que se pasan a esta función. Si uno o más elementos tienen el mismo nombre, el último elemento con ese nombre aparece en el resultado.

```
union('<collection1>', '<collection2>', ...)
union([<collection1>], [<collection2>], ...)
```

| Parámetro | Obligatorio | type | DESCRIPCIÓN |
| --------- | -------- | ---- | ----------- |
| <*colección1*>, <*colección2*>, ...  | Sí | Matriz u objeto, pero no ambos | Colecciones de las que desean *todos* los elementos |
|||||

| Valor devuelto | type | DESCRIPCIÓN |
| ------------ | ---- | ----------- |
| <*colección_actualizada*> | Matriz u objeto, respectivamente | Colección con todos los elementos de las colecciones especificadas, sin duplicados. |
||||

*Ejemplo*

Este ejemplo obtiene *todos* los elementos de estas colecciones:

```
union(createArray(1, 2, 3), createArray(1, 2, 10, 101))
```

Y devuelve este resultado: `[1, 2, 3, 10, 101]`

<a name="uriComponent"></a>

### <a name="uricomponent"></a>uriComponent

Devuelve una versión codificada de un identificador uniforme de recursos (URI) de una cadena reemplazando los caracteres no seguros de la dirección URL por caracteres de escape.
Use esta función en lugar de [encodeUriComponent()](#encodeUriComponent).
Aunque ambas funciones funcionan del mismo modo, `uriComponent()` es preferible.

```
uriComponent('<value>')
```

| Parámetro | Obligatorio | type | DESCRIPCIÓN |
| --------- | -------- | ---- | ----------- |
| <*valor*> | Sí | String | Cadena que se va a convertir en formato codificado de URI |
|||||

| Valor devuelto | type | DESCRIPCIÓN |
| ------------ | ---- | ----------- |
| <*uri-codificado*> | String | Cadena codificada en formato URI con caracteres de escape |
||||

*Ejemplo*

Este ejemplo crea una versión codificada en formato URI de esta cadena:

```
uriComponent('https://contoso.com')
```

Y devuelve este resultado: `"http%3A%2F%2Fcontoso.com"`

<a name="uriComponentToBinary"></a>

### <a name="uricomponenttobinary"></a>uriComponentToBinary

Devuelve la versión binaria de un componente de un identificador uniforme de recursos (URI).

```
uriComponentToBinary('<value>')
```

| Parámetro | Obligatorio | type | DESCRIPCIÓN |
| --------- | -------- | ---- | ----------- |
| <*valor*> | Sí | String | Cadena con codificación URI que se va a convertir |
|||||

| Valor devuelto | type | DESCRIPCIÓN |
| ------------ | ---- | ----------- |
| <*binario-de-codificado-uri*> | String | Versión binaria de la cadena con codificación URI El contenido binario está codificado en base64 y representado por `$content`. |
||||

*Ejemplo*

Este ejemplo crea la versión binaria de esta cadena con codificación URI:

```
uriComponentToBinary('http%3A%2F%2Fcontoso.com')
```

Y devuelve este resultado:

`"001000100110100001110100011101000111000000100101001100
11010000010010010100110010010001100010010100110010010001
10011000110110111101101110011101000110111101110011011011
110010111001100011011011110110110100100010"`

<a name="uriComponentToString"></a>

### <a name="uricomponenttostring"></a>uriComponentToString

Devuelve la versión de cadena de una cadena codificada en formato de identificador uniforme de recursos (URI), decodificando dicha cadena.

```
uriComponentToString('<value>')
```

| Parámetro | Obligatorio | type | DESCRIPCIÓN |
| --------- | -------- | ---- | ----------- |
| <*valor*> | Sí | String | Cadena con codificación URI que se va a decodificar |
|||||

| Valor devuelto | type | DESCRIPCIÓN |
| ------------ | ---- | ----------- |
| <*uri-decodificado*> | String | Versión decodificada de la cadena con codificación URI |
||||

*Ejemplo*

Este ejemplo crea la versión de cadena decodificada de esta cadena con codificación URI:

```
uriComponentToString('http%3A%2F%2Fcontoso.com')
```

Y devuelve este resultado: `"https://contoso.com"`

<a name="uriHost"></a>

### <a name="urihost"></a>uriHost

Devuelve el valor `host` de un identificador uniforme de recursos (URI).

```
uriHost('<uri>')
```

| Parámetro | Obligatorio | type | DESCRIPCIÓN |
| --------- | -------- | ---- | ----------- |
| <*uri*> | Sí | String | Identificador URI cuyo valor `host` desea |
|||||

| Valor devuelto | type | DESCRIPCIÓN |
| ------------ | ---- | ----------- |
| <*valor-host*> | String | Valor `host` del identificador URI especificado. |
||||

*Ejemplo*

Este ejemplo busca el valor `host` de este identificador URI:

```
uriHost('https://www.localhost.com:8080')
```

Y devuelve este resultado: `"www.localhost.com"`

<a name="uriPath"></a>

### <a name="uripath"></a>uriPath

Devuelve el valor `path` de un identificador uniforme de recursos (URI).

```
uriPath('<uri>')
```

| Parámetro | Obligatorio | type | DESCRIPCIÓN |
| --------- | -------- | ---- | ----------- |
| <*uri*> | Sí | String | Identificador URI cuyo valor `path` desea |
|||||

| Valor devuelto | type | DESCRIPCIÓN |
| ------------ | ---- | ----------- |
| <*valor-path*> | String | Valor `path` del identificador URI especificado. Si `path` no tiene un valor, se devuelve el carácter "/". |
||||

*Ejemplo*

Este ejemplo busca el valor `path` de este identificador URI:

```
uriPath('http://www.contoso.com/catalog/shownew.htm?date=today')
```

Y devuelve este resultado: `"/catalog/shownew.htm"`

<a name="uriPathAndQuery"></a>

### <a name="uripathandquery"></a>uriPathAndQuery

Devuelve los valores `path` y `query` de un identificador uniforme de recursos (URI).

```
uriPathAndQuery('<uri>')
```

| Parámetro | Obligatorio | type | DESCRIPCIÓN |
| --------- | -------- | ---- | ----------- |
| <*uri*> | Sí | String | Identificador URI cuyos valores `path` y `query` desea |
|||||

| Valor devuelto | type | DESCRIPCIÓN |
| ------------ | ---- | ----------- |
| <*valores-path-y-query*> | String | Valores `path` y `query` del identificador URI especificado. Si `path` no tiene un valor, se devuelve el carácter "/". |
||||

*Ejemplo*

Este ejemplo busca los valores `path` y `query` de este identificador URI:

```
uriPathAndQuery('http://www.contoso.com/catalog/shownew.htm?date=today')
```

Y devuelve este resultado: `"/catalog/shownew.htm?date=today"`

<a name="uriPort"></a>

### <a name="uriport"></a>uriPort

Devuelve el valor `port` de un identificador uniforme de recursos (URI).

```
uriPort('<uri>')
```

| Parámetro | Obligatorio | type | DESCRIPCIÓN |
| --------- | -------- | ---- | ----------- |
| <*uri*> | Sí | String | Identificador URI cuyo valor `port` desea |
|||||

| Valor devuelto | type | DESCRIPCIÓN |
| ------------ | ---- | ----------- |
| <*valor-port*> | Integer | Valor `port` del identificador URI especificado. Si `port` no especifica un valor, devuelve el puerto predeterminado del protocolo. |
||||

*Ejemplo*

Este ejemplo devuelve el valor `port` de este identificador URI:

```
uriPort('http://www.localhost:8080')
```

Y devuelve este resultado: `8080`

<a name="uriQuery"></a>

### <a name="uriquery"></a>uriQuery

Devuelve el valor `query` de un identificador uniforme de recursos (URI).

```
uriQuery('<uri>')
```

| Parámetro | Obligatorio | type | DESCRIPCIÓN |
| --------- | -------- | ---- | ----------- |
| <*uri*> | Sí | String | Identificador URI cuyo valor `query` desea |
|||||

| Valor devuelto | type | DESCRIPCIÓN |
| ------------ | ---- | ----------- |
| <*valor-query*> | String | Valor `query` del identificador URI especificado. |
||||

*Ejemplo*

Este ejemplo devuelve el valor `query` de este identificador URI:

```
uriQuery('http://www.contoso.com/catalog/shownew.htm?date=today')
```

Y devuelve este resultado: `"?date=today"`

<a name="uriScheme"></a>

### <a name="urischeme"></a>UriScheme

Devuelve el valor `scheme` de un identificador uniforme de recursos (URI).

```
uriScheme('<uri>')
```

| Parámetro | Obligatorio | type | DESCRIPCIÓN |
| --------- | -------- | ---- | ----------- |
| <*uri*> | Sí | String | Identificador URI cuyo valor `scheme` desea |
|||||

| Valor devuelto | type | DESCRIPCIÓN |
| ------------ | ---- | ----------- |
| <*valor-scheme*> | String | Valor `scheme` del identificador URI especificado. |
||||

*Ejemplo*

Este ejemplo devuelve el valor `scheme` de este identificador URI:

```
uriScheme('http://www.contoso.com/catalog/shownew.htm?date=today')
```

Y devuelve este resultado: `"http"`

<a name="utcNow"></a>

### <a name="utcnow"></a>utcNow

Devuelve la marca de tiempo actual.

```
utcNow('<format>')
```

Si lo desea, puede especificar un formato diferente con el parámetro <*format*>.


| Parámetro | Obligatorio | type | DESCRIPCIÓN |
| --------- | -------- | ---- | ----------- |
| <*formato*> | Sin | String | Puede ser un [especificador de formato sencillo](https://docs.microsoft.com/dotnet/standard/base-types/standard-date-and-time-format-strings) o un [patrón de formato personalizado](https://docs.microsoft.com/dotnet/standard/base-types/custom-date-and-time-format-strings). El formato predeterminado de la marca de tiempo es ["o"](https://docs.microsoft.com/dotnet/standard/base-types/standard-date-and-time-format-strings) (aaaa-MM-ddTHH:mm:ss:fffffffK), que cumple con [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) y conserva la información de zona horaria. |
|||||

| Valor devuelto | type | DESCRIPCIÓN |
| ------------ | ---- | ----------- |
| <*marca-de-tiempo-actual*> | String | Fecha y hora actuales |
||||

*Ejemplo 1*

Imagine que hoy es 15 de abril de 2018 a las 1:00:00 P.M.
Este ejemplo obtiene la marca de tiempo actual:

```
utcNow()
```

Y devuelve este resultado: `"2018-04-15T13:00:00.0000000Z"`

*Ejemplo 2*

Imagine que hoy es 15 de abril de 2018 a las 1:00:00 P.M.
Este ejemplo obtiene la marca de tiempo actual con el formato "D" opcional:

```
utcNow('D')
```

Y devuelve este resultado: `"Sunday, April 15, 2018"`

<a name="variables"></a>

### <a name="variables"></a>variables

Devuelve el valor de una variable especificada.

```
variables('<variableName>')
```

| Parámetro | Obligatorio | type | DESCRIPCIÓN |
| --------- | -------- | ---- | ----------- |
| <*nombre_de_variable*> | Sí | String | Nombre de la variable cuyo valor busca |
|||||

| Valor devuelto | type | DESCRIPCIÓN |
| ------------ | ---- | ----------- |
| <*valor-de-variable*> | Any | Valor de la variable especificada |
||||

*Ejemplo*

Suponga que el valor actual de la variable "numItems" es 20.
Este ejemplo obtiene el valor entero de esta variable:

```
variables('numItems')
```

Y devuelve este resultado: `20`

<a name="workflow"></a>

### <a name="workflow"></a>flujo de trabajo

Devuelve todos los detalles sobre el propio flujo de trabajo durante el tiempo de ejecución.

```
workflow().<property>
```

| Parámetro | Obligatorio | type | DESCRIPCIÓN |
| --------- | -------- | ---- | ----------- |
| <*propiedad*> | Sin | String | Nombre de la propiedad del flujo de trabajo cuyo valor desea <p>Un objeto de flujo de trabajo tiene estas propiedades: **name** (nombre), **type** (tipo), **id** (identificador), **location** (ubicación) y **run** (ejecución). El valor de la propiedad **run** también es un objeto que tiene estas propiedades: **name**, **type** e **id**. |
|||||

*Ejemplo*

Este ejemplo devuelve el nombre de la ejecución actual de un flujo de trabajo:

```
workflow().run.name
```

<a name="xml"></a>

### <a name="xml"></a>Xml

Devuelve la versión XML de una cadena que contiene un objeto JSON.

```
xml('<value>')
```

| Parámetro | Obligatorio | type | DESCRIPCIÓN |
| --------- | -------- | ---- | ----------- |
| <*valor*> | Sí | String | Cadena con el objeto JSON que se va a convertir <p>El objeto JSON debe tener solo una propiedad raíz, que no puede ser una matriz. <br>Use el carácter de barra diagonal inversa (\\) como carácter de escape para la marca de comillas dobles ("). |
|||||

| Valor devuelto | type | DESCRIPCIÓN |
| ------------ | ---- | ----------- |
| <*versión-xml*> | Object | XML codificado de la cadena u objeto JSON especificados |
||||

*Ejemplo 1*

Este ejemplo crea la versión XML de esta cadena, que contiene un objeto JSON:

`xml(json('{ \"name\": \"Sophia Owen\" }'))`

Y devuelve este código XML resultante:

```xml
<name>Sophia Owen</name>
```

*Ejemplo 2*

Suponga que tiene este objeto JSON:

```json
{
  "person": {
    "name": "Sophia Owen",
    "city": "Seattle"
  }
}
```

Este ejemplo crea código XML de una cadena que contiene este objeto JSON:

`xml(json('{\"person\": {\"name\": \"Sophia Owen\", \"city\": \"Seattle\"}}'))`

Y devuelve este código XML resultante:

```xml
<person>
  <name>Sophia Owen</name>
  <city>Seattle</city>
<person>
```

<a name="xpath"></a>

### <a name="xpath"></a>xpath

Comprueba el código XML de los nodos o valores que coinciden con una expresión XPath (XML Path Language) y devuelve los nodos o valores coincidentes. Una expresión XPath o simplemente "XPath", le ayuda a recorrer una estructura de documento XML para que pueda seleccionar nodos o valores de proceso del contenido XML.

```
xpath('<xml>', '<xpath>')
```

| Parámetro | Obligatorio | type | DESCRIPCIÓN |
| --------- | -------- | ---- | ----------- |
| <*xml*> | Sí | Any | Cadena XML en la que buscar nodos o valores que coincidan con un valor de la expresión XPath |
| <*xpath*> | Sí | Any | Expresión XPath utilizada para buscar nodos XML o valores coincidentes |
|||||

| Valor devuelto | type | DESCRIPCIÓN |
| ------------ | ---- | ----------- |
| <*nodo-xml*> | XML | Nodo XML si solo un nodo coincide con la expresión XPath especificada |
| <*valor*> | Any | Valor de un nodo XML si solo un valor coincide con la expresión XPath especificada |
| [<*xml-nodo1*>, <*xml-nodo2*>, ...] </br>O bien </br>[<*valor1*>, <*valor2*>, ...] | Array | Matriz con los nodos XML o valores que coinciden con la expresión XPath especificada |
||||

*Ejemplo 1*

Este ejemplo busca los nodos que coinciden con el nodo `<name></name>` de los argumentos especificados y devuelve una matriz con los valores de nodo:

`xpath(xml(parameters('items')), '/produce/item/name')`

Estos son los argumentos:

* La cadena "items", que contiene este código XML:

  `"<?xml version="1.0"?> <produce> <item> <name>Gala</name> <type>apple</type> <count>20</count> </item> <item> <name>Honeycrisp</name> <type>apple</type> <count>10</count> </item> </produce>"`

  El ejemplo utiliza la función [parameters()](#parameters) para obtener la cadena XML del argumento "items", pero debe también convertir la cadena a formato XML mediante la función [xml()](#xml).

* Esta expresión XPath, que se pasa como una cadena:

  `"/produce/item/name"`

Esta es la matriz de resultados con los nodos que coinciden `<name></name`:

`[ <name>Gala</name>, <name>Honeycrisp</name> ]`

*Ejemplo 2*

Siguiendo el ejemplo 1, este ejemplo busca nodos que coincidan con el nodo `<count></count>` y agrega los valores de nodo con la función `sum()`:

`xpath(xml(parameters('items')), 'sum(/produce/item/count)')`

Y devuelve este resultado: `30`

*Ejemplo 3*

En este ejemplo, ambas expresiones buscan nodos que coincidan con el nodo `<location></location>` de los argumentos especificados, que incluye código XML con un espacio de nombres. Las expresiones utilizan el carácter de barra diagonal inversa (\\) como carácter de escape para la marca de comillas dobles (").

* *Expresión 1*

  `xpath(xml(body('Http')), '/*[name()=\"file\"]/*[name()=\"location\"]')`

* *Expresión 2*

  `xpath(xml(body('Http')), '/*[local-name()=\"file\" and namespace-uri()=\"http://contoso.com\"]/*[local-name()=\"location\"]')`

Estos son los argumentos:

* Este código XML, que incluye el espacio de nombres del documento XML, `xmlns="http://contoso.com"`:

  ```xml
  <?xml version="1.0"?> <file xmlns="http://contoso.com"> <location>Paris</location> </file>
  ```

* Cualquier expresión XPath:

  * `/*[name()=\"file\"]/*[name()=\"location\"]`

  * `/*[local-name()=\"file\" and namespace-uri()=\"http://contoso.com\"]/*[local-name()=\"location\"]`

Este es el nodo de resultados que coincide con el nodo `<location></location>`:

```xml
<location xmlns="https://contoso.com">Paris</location>
```

*Ejemplo 4*

Siguiendo el ejemplo 3, este ejemplo busca el valor en el nodo `<location></location>`:

`xpath(xml(body('Http')), 'string(/*[name()=\"file\"]/*[name()=\"location\"])')`

Y devuelve este resultado: `"Paris"`

## <a name="next-steps"></a>Pasos siguientes

Más información sobre el [Lenguaje de definición de flujo de trabajo](../logic-apps/logic-apps-workflow-definition-language.md).
