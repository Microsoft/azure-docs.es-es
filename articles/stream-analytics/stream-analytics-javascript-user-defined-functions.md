---
title: Funciones definidas por el usuario en JavaScript para Azure Stream Analytics
description: Este artículo es una introducción a las funciones definidas por el usuario de JavaScript en Stream Analytics.
author: rodrigoaatmicrosoft
ms.author: rodrigoa
ms.service: stream-analytics
ms.topic: tutorial
ms.reviewer: mamccrea
ms.custom: mvc, devx-track-javascript
ms.date: 06/16/2020
ms.openlocfilehash: 6540b35925a92ebd6a8bcced427b5457785603db
ms.sourcegitcommit: 269da970ef8d6fab1e0a5c1a781e4e550ffd2c55
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/10/2020
ms.locfileid: "88056914"
---
# <a name="javascript-user-defined-functions-in-azure-stream-analytics"></a>Funciones definidas por el usuario de JavaScript en Azure Stream Analytics
 
Azure Stream Analytics admite funciones definidas por el usuario escritas en JavaScript. Con el conjunto completo de métodos **String**, **RegExp**, **Math**, **Array** y **Date** que proporciona JavaScript, las transformaciones de datos complejas con trabajos de Stream Analytics son más fáciles de crear.

## <a name="overview"></a>Información general

Las funciones definidas por el usuario en JavaScript admiten funciones escalares de solo cálculo y sin estado, que no requieren conectividad externa. El valor devuelto de una función solo puede ser un valor escalar (único). Después de agregar una función definida por el usuario en JavaScript a un trabajo, puede usar la función en cualquier parte de la consulta, como una función escalar integrada.

Estos son algunos escenarios en los que le podrían resultar útiles las funciones definidas por el usuario en JavaScript:
* Análisis y manipulación de cadenas que incluyen funciones de expresiones regulares, por ejemplo, **Regexp_Replace()** y **Regexp_Extract()**
* Descodificación y codificación de datos, por ejemplo, conversión de binario a hexadecimal
* Realización de cálculos matemáticos con funciones **matemáticas** de JavaScript
* Realización de operaciones de matriz como ordenar, combinar, buscar y rellenar

Aquí se indican algunas operaciones que no se pueden realizar con una función definida por el usuario de JavaScript en Stream Analytics:
* Llamar a puntos de conexión de REST externos, por ejemplo, mediante la realización de una búsqueda inversa de direcciones IP o una extracción de datos de referencia de un origen externo
* Realizar la serialización o deserialización de formatos de eventos personalizados en entradas y salidas
* Crear agregados personalizados

Aunque haya funciones como **Date.GetDate()** o **Math.random()** que no están bloqueadas en la definición de funciones, debe evitar su uso. Estas funciones **no** devuelven el mismo resultado cada vez que las llama y el servicio Azure Stream Analytics no mantiene un diario de las llamadas a funciones y los resultados devueltos. Si una función devuelve resultados diferentes en los mismos eventos, la repetibilidad no se garantizará cuando el usuario o el servicio Stream Analytics reinicie un trabajo.

## <a name="add-a-javascript-user-defined-function-to-your-job"></a>Adición de una función definida por el usuario en JavaScript al trabajo

> [!NOTE]
> Estos pasos funcionan en los trabajos de Stream Analytics configurados para ejecutarse en la nube. Si el trabajo de Stream Analytics está configurado para ejecutarse en Azure IoT Edge, use entonces Visual Studio y [escriba la función definida por el usuario mediante C# ](stream-analytics-edge-csharp-udf.md).

Para crear una función definida por el usuario de JavaScript en el trabajo de Stream Analytics, seleccione **Funciones** en **Topología de trabajo**. A continuación, seleccione **UDF de JavaScript** en el menú desplegable **+Agregar**. 

![Adición de UDF de JavaScript](./media/javascript/stream-analytics-jsudf-add.png)

A continuación, debe proporcionar las propiedades siguientes y seleccionar **Guardar**.

|Propiedad|Descripción|
|--------|-----------|
|Alias de función|Escriba un nombre para invocar la función en la consulta.|
|Tipo de salida|Tipo que devolverá la función definida por el usuario de JavaScript en la consulta Stream Analytics.|
|Definición de función|Implementación de la función de JavaScript que se ejecutará cada vez que se invoque la UDF desde la consulta.|

## <a name="test-and-troubleshoot-javascript-udfs"></a>Prueba y solución de problemas de UDF de JavaScript 

Puede probar y depurar la lógica de UDF de JavaScript en cualquier explorador. La depuración y prueba de la lógica de estas funciones definidas por el usuario no se admite actualmente en el portal de Stream Analytics. Una vez que la función funcione según lo previsto, puede agregarla al trabajo Stream Analytics tal y como se mencionó anteriormente y, a continuación, invocarla directamente desde la consulta. También puede probar la lógica de consulta con las UDF de JavaScript mediante las [herramientas de Stream Analytics para Visual Studio](https://docs.microsoft.com/azure/stream-analytics/stream-analytics-tools-for-visual-studio-install).

Los errores en tiempo de ejecución de JavaScript se consideran graves y se presentan en el registro de actividad. Para recuperar el registro, en Azure Portal, vaya al trabajo y seleccione **Registro de actividad**.

## <a name="call-a-javascript-user-defined-function-in-a-query"></a>Llamada a una función definida por el usuario de JavaScript en una consulta

Puede invocar fácilmente la función de JavaScript en la consulta mediante el alias de función con el prefijo **udf**. Este es un ejemplo de una UDF de JavaScript que convierte valores hexadecimales en un entero que se invoca en una consulta de Stream Analytics.

```SQL
    SELECT
        time,
        UDF.hex2Int(offset) AS IntOffset
    INTO
        output
    FROM
        InputStream
```

## <a name="supported-javascript-objects"></a>Objetos de JavaScript compatibles

Las funciones definidas por el usuario en JavaScript para Azure Stream Analytics admiten objetos de JavaScript estándar e integrados. Para obtener una lista de estos objetos, consulte [Objetos globales](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects).

### <a name="stream-analytics-and-javascript-type-conversion"></a>Conversión de tipos de Stream Analytics y JavaScript

Hay diferencias entre los tipos admitidos en el lenguaje de consultas de Stream Analytics y JavaScript. En esta tabla se muestran las asignaciones de conversión entre los dos:

Stream Analytics | JavaScript
--- | ---
bigint | Number (JavaScript solamente puede representar enteros hasta 2^53 con precisión)
DateTime | Date (JavaScript solo admite milisegundos)
double | Number
nvarchar(MAX) | String
Registro | Object
Array | Array
NULL | Null

Estas son las conversiones de JavaScript a Stream Analytics:

JavaScript | Stream Analytics
--- | ---
Number | Bigint (si el número es round y entre long.MinValue y long.MaxValue, double en caso contrario)
Date | DateTime
String | nvarchar(MAX)
Object | Registro
Array | Array
Null, sin definir | NULL
Cualquier otro tipo (por ejemplo, una función o un error) | No admitido (da lugar a un error en tiempo de ejecución)

El lenguaje JavaScript distingue mayúsculas de minúsculas y las mayúsculas y minúsculas de los campos de objetos del código JavaScript deben coincidir con las mayúsculas y minúsculas de los campos de los datos de entrada. Los trabajos con un nivel de compatibilidad de 1.0 convertirán los campos de la instrucción SELECT de SQL en minúsculas. En el nivel de compatibilidad 1.1 y superior, los campos de la instrucción SELECT tendrán la misma convención de mayúsculas y minúsculas que la especificada en la consulta SQL.

## <a name="other-javascript-user-defined-function-patterns"></a>Otros patrones de función definida por el usuario de JavaScript

### <a name="write-nested-json-to-output"></a>Escritura de JSON anidado en la salida

Si tiene un paso de procesamiento de seguimiento que usa la salida del trabajo de Stream Analytics como entrada y requiere un formato JSON, puede escribir una cadena JSON en la salida. El ejemplo siguiente llama a la función **JSON.stringify()** para empaquetar todos los pares nombre-valor de la entrada y escribirlos como un único valor de cadena en la salida.

**Definición de funciones definidas por el usuario en JavaScript:**

```javascript
function main(x) {
return JSON.stringify(x);
}
```

**Consulta de ejemplo:**
```SQL
SELECT
    DataString,
    DataValue,
    HexValue,
    UDF.jsonstringify(input) As InputEvent
INTO
    output
FROM
    input PARTITION BY PARTITIONID
```

### <a name="cast-string-to-json-object-to-process"></a>Conversión de una cadena en un objeto JSON para procesar

Si tiene un campo de cadena que sea JSON y desea convertirlo en un objeto JSON para procesarlo en una UDF de JavaScript, puede usar la función **JSON.parse()** a fin de crear un objeto JSON que se pueda usar.

**Definición de funciones definidas por el usuario en JavaScript:**

```javascript
function main(x) {
var person = JSON.parse(x);  
return person.name;
}
```

**Consulta de ejemplo:**
```SQL
SELECT
    UDF.getName(input) AS Name
INTO
    output
FROM
    input
```

### <a name="use-trycatch-for-error-handling"></a>Uso de try/catch para el control de errores

Los bloques try/catch pueden ayudarle a identificar problemas con los datos de entrada incorrectos que se pasan a una UDF de JavaScript.

**Definición de funciones definidas por el usuario en JavaScript:**

```javascript
function main(input, x) {
    var obj = null;

    try{
        obj = JSON.parse(x);
    }catch(error){
        throw input;
    }
    
    return obj.Value;
}
```

**Consulta de ejemplo: Pase el registro completo como primer parámetro para que se pueda devolver si se produce un error.**
```SQL
SELECT
    A.context.company AS Company,
    udf.getValue(A, A.context.value) as Value
INTO
    output
FROM
    input A
```

## <a name="next-steps"></a>Pasos siguientes

* [UDF de Machine Learning](https://docs.microsoft.com/azure/stream-analytics/machine-learning-udf)
* [UDF de C#](https://docs.microsoft.com/azure/stream-analytics/stream-analytics-edge-csharp-udf-methods)
