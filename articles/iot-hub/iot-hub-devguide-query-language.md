---
title: Información sobre el lenguaje de consulta de IoT Hub de Azure | Microsoft Docs
description: 'Guía del desarrollador: descripción del lenguaje de consulta de IoT Hub de tipo SQL que se usa para recuperar información sobre dispositivos o módulos gemelos y trabajos desde IoT Hub.'
author: robinsh
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 10/29/2018
ms.author: robinsh
ms.custom: devx-track-csharp
ms.openlocfilehash: 77becbf4777d0668991adcd74b722cd28ac36f03
ms.sourcegitcommit: 3fc3457b5a6d5773323237f6a06ccfb6955bfb2d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/11/2020
ms.locfileid: "90031184"
---
# <a name="iot-hub-query-language-for-device-and-module-twins-jobs-and-message-routing"></a>Lenguaje de consulta de IoT Hub para dispositivos y módulos gemelos, trabajos y enrutamiento de mensajes

IoT Hub proporciona un lenguaje eficaz de tipo SQL para recuperar información con respecto a los [dispositivos gemelos](iot-hub-devguide-device-twins.md), [módulos gemelos](iot-hub-devguide-module-twins.md), [trabajos](iot-hub-devguide-jobs.md) y [enrutamiento de mensajes](iot-hub-devguide-messages-d2c.md). Este artículo presenta:

* una introducción a las características principales del lenguaje de consulta de IoT Hub y
* una descripción más detallada del lenguaje. Para más información acerca del lenguaje de consulta del enrutamiento de mensajes, consulte [Consultas en el enrutamiento de mensajes](../iot-hub/iot-hub-devguide-routing-query-syntax.md).

[!INCLUDE [iot-hub-basic](../../includes/iot-hub-basic-partial.md)]

## <a name="device-and-module-twin-queries"></a>Consultas de dispositivos y módulos gemelos

Los [dispositivos gemelos](iot-hub-devguide-device-twins.md) y los [módulos gemelos](iot-hub-devguide-module-twins.md) pueden contener objetos JSON arbitrarios en forma de etiquetas y propiedades. IoT Hub permite consultar dispositivos gemelos y módulos gemelos como un solo documento JSON que contiene toda la información sobre ellos.

Por ejemplo, suponga que los dispositivos gemelos de su centro de IoT tienen la siguiente estructura (lo mismo se aplica a los módulos gemelos, excepto que tienen un valor de moduleId adicional):

```json
{
    "deviceId": "myDeviceId",
    "etag": "AAAAAAAAAAc=",
    "status": "enabled",
    "statusUpdateTime": "0001-01-01T00:00:00",
    "connectionState": "Disconnected",
    "lastActivityTime": "0001-01-01T00:00:00",
    "cloudToDeviceMessageCount": 0,
    "authenticationType": "sas",
    "x509Thumbprint": {
        "primaryThumbprint": null,
        "secondaryThumbprint": null
    },
    "version": 2,
    "tags": {
        "location": {
            "region": "US",
            "plant": "Redmond43"
        }
    },
    "properties": {
        "desired": {
            "telemetryConfig": {
                "configId": "db00ebf5-eeeb-42be-86a1-458cccb69e57",
                "sendFrequencyInSecs": 300
            },
            "$metadata": {
            ...
            },
            "$version": 4
        },
        "reported": {
            "connectivity": {
                "type": "cellular"
            },
            "telemetryConfig": {
                "configId": "db00ebf5-eeeb-42be-86a1-458cccb69e57",
                "sendFrequencyInSecs": 300,
                "status": "Success"
            },
            "$metadata": {
            ...
            },
            "$version": 7
        }
    }
}
```

### <a name="device-twin-queries"></a>Consultas de dispositivos gemelos

IoT Hub expone los dispositivos gemelos como una colección de documentos denominada **devices**. Por ejemplo, la consulta siguiente recupera el conjunto completo de dispositivos gemelos:

```sql
SELECT * FROM devices
```

> [!NOTE]
> Los [SDK IoT de Azure](iot-hub-devguide-sdks.md) admiten la paginación de resultados de gran tamaño.

IoT Hub permite recuperar dispositivos gemelos filtrando por condiciones arbitrarias. Por ejemplo, para recibir los dispositivos gemelos allí donde la etiqueta **location.region** se estableció en **US**, use la siguiente consulta:

```sql
SELECT * FROM devices
WHERE tags.location.region = 'US'
```

También se admiten operadores booleanos y comparaciones aritméticas. Por ejemplo, para recuperar los dispositivos gemelos ubicados en la región denominada "US" y que hayan sido configurados para enviar datos de telemetría en intervalos inferiores a un minuto, use la siguiente consulta:

```sql
SELECT * FROM devices
  WHERE tags.location.region = 'US'
    AND properties.reported.telemetryConfig.sendFrequencyInSecs >= 60
```

Por comodidad, también es posible usar constantes de matriz con los operadores **IN** (En) y **NIN** (No en). Por ejemplo, para recuperar los dispositivos gemelos que notifiquen conectividad WiFi o con cable, use la siguiente consulta:

```sql
SELECT * FROM devices
  WHERE properties.reported.connectivity IN ['wired', 'wifi']
```

A menudo, es necesario identificar a todos los dispositivos gemelos que contienen una propiedad concreta. IoT Hub admite la función `is_defined()` para esta finalidad. Por ejemplo, para recuperar los dispositivos gemelos que definan la propiedad `connectivity`, use la siguiente consulta:

```SQL
SELECT * FROM devices
  WHERE is_defined(properties.reported.connectivity)
```

Consulte la sección [Cláusula WHERE](iot-hub-devguide-query-language.md#where-clause) para obtener la referencia completa de las funcionalidades de filtrado.

También se admiten la agrupación y las agregaciones. Por ejemplo, para buscar el número de dispositivos en cada estado de configuración de telemetría, use la siguiente consulta:

```sql
SELECT properties.reported.telemetryConfig.status AS status,
    COUNT() AS numberOfDevices
  FROM devices
  GROUP BY properties.reported.telemetryConfig.status
```

Esta consulta de agrupación devolverá un resultado similar al ejemplo siguiente:

```json
[
    {
        "numberOfDevices": 3,
        "status": "Success"
    },
    {
        "numberOfDevices": 2,
        "status": "Pending"
    },
    {
        "numberOfDevices": 1,
        "status": "Error"
    }
]
```

En este ejemplo, puede ver que tres dispositivos notificaron una configuración correcta, dos de ellos todavía están aplicándola y uno de ellos ha notificado un error.

Las consultas de proyección permiten a los desarrolladores devolver solo las propiedades que les interesen. Por ejemplo, para recuperar la hora de la última actividad de todos los dispositivos desconectados, use la consulta siguiente:

```sql
SELECT LastActivityTime FROM devices WHERE status = 'enabled'
```

### <a name="module-twin-queries"></a>Consultas de módulo gemelo

Las consultas en los módulos gemelos son parecidas a las consultas en los dispositivos gemelos, pero usan un espacio de nombres o colección diferentes porque, en lugar de realizar la consulta desde **devices**, se realiza desde **devices.modules**:

```sql
SELECT * FROM devices.modules
```

No se permite la unión entre las colecciones devices y devices.modules. Si quiere consultar módulos gemelos entre dispositivos, hágalo en función de etiquetas. Esta consulta devuelve todos los módulos gemelos entre todos los dispositivos con el estado de exploración:

```sql
SELECT * FROM devices.modules WHERE properties.reported.status = 'scanning'
```

Esta consulta devuelve todos los módulos gemelos con el estado de exploración, pero solo en el subconjunto de dispositivos especificado:

```sql
SELECT * FROM devices.modules
  WHERE properties.reported.status = 'scanning'
  AND deviceId IN ['device1', 'device2']
```

### <a name="c-example"></a>Ejemplo de C#

El [SDK del servicio C#](iot-hub-devguide-sdks.md) expone la funcionalidad de consulta en la clase **RegistryManager**.

Este ejemplo corresponde a una consulta simple:

```csharp
var query = registryManager.CreateQuery("SELECT * FROM devices", 100);
while (query.HasMoreResults)
{
    var page = await query.GetNextAsTwinAsync();
    foreach (var twin in page)
    {
        // do work on twin object
    }
}
```

Se crea una instancia del objeto **query** con un tamaño de páginas (hasta 100). Después, se recuperan diversas páginas con varias llamadas a los métodos **GetNextAsTwinAsync**.

El objeto query expone varios valores **Next**, en función de la opción de deserialización que requiera la consulta. Por ejemplo, objetos de trabajo o dispositivos gemelos, o JSON sin formato si se usan proyecciones.

### <a name="nodejs-example"></a>Ejemplo de Node.js

El [SDK de servicios IoT de Azure para Node.js](iot-hub-devguide-sdks.md) expone la funcionalidad de consulta en el objeto **Registry**.

Este ejemplo corresponde a una consulta simple:

```javascript
var query = registry.createQuery('SELECT * FROM devices', 100);
var onResults = function(err, results) {
    if (err) {
        console.error('Failed to fetch the results: ' + err.message);
    } else {
        // Do something with the results
        results.forEach(function(twin) {
            console.log(twin.deviceId);
        });

        if (query.hasMoreResults) {
            query.nextAsTwin(onResults);
        }
    }
};
query.nextAsTwin(onResults);
```

Se crea una instancia del objeto **query** con un tamaño de páginas (hasta 100). Después, se recuperan diversas páginas con varias llamadas al método **nextAsTwin**.

El objeto query expone varios valores **Next**, en función de la opción de deserialización que requiera la consulta. Por ejemplo, objetos de trabajo o dispositivos gemelos, o JSON sin formato si se usan proyecciones.

### <a name="limitations"></a>Limitaciones

> [!IMPORTANT]
> Los resultados de las consultas pueden demorarse unos minutos con respecto a los valores más recientes en los dispositivos gemelos. Si se consultan dispositivos gemelos individuales por su identificador, use la [API de REST Get Twin](https://docs.microsoft.com/java/api/com.microsoft.azure.sdk.iot.device.devicetwin?view=azure-java-stable). Esta API siempre devuelve los últimos valores y tiene límites restrictivos más altos. Puede emitir la API de REST directamente o usar la funcionalidad equivalente en uno de los [SDK del servicio Azure IoT Hub](iot-hub-devguide-sdks.md#azure-iot-hub-service-sdks).

Actualmente, las comparaciones solo se admiten entre tipos primitivos (no objetos), por ejemplo `... WHERE properties.desired.config = properties.reported.config` solo se admite si esas propiedades tienen valores primitivos.

## <a name="get-started-with-jobs-queries"></a>Introducción a las consultas de trabajos

Los [trabajos](iot-hub-devguide-jobs.md) proporcionan una forma de ejecutar operaciones en conjuntos de dispositivos. Cada dispositivo gemelo contiene la información de los trabajos de los que forma parte en una colección denominada **jobs**.

```json
{
    "deviceId": "myDeviceId",
    "etag": "AAAAAAAAAAc=",
    "tags": {
        ...
    },
    "properties": {
        ...
    },
    "jobs": [
        {
            "deviceId": "myDeviceId",
            "jobId": "myJobId",
            "jobType": "scheduleTwinUpdate",
            "status": "completed",
            "startTimeUtc": "2016-09-29T18:18:52.7418462",
            "endTimeUtc": "2016-09-29T18:20:52.7418462",
            "createdDateTimeUtc": "2016-09-29T18:18:56.7787107Z",
            "lastUpdatedDateTimeUtc": "2016-09-29T18:18:56.8894408Z",
            "outcome": {
                "deviceMethodResponse": null
            }
        },
        ...
    ]
}
```

Ahora, esta colección se puede consultar como **devices.jobs** en el lenguaje de consulta de IoT Hub.

> [!IMPORTANT]
> Actualmente, la propiedad jobs no se devuelve nunca cuando se consulta a dispositivos gemelos. Es decir, las consultas que contienen "FROM devices". Solo se puede acceder a la propiedad jobs directamente con consultas que usen `FROM devices.jobs`.
>
>

Por ejemplo, para obtener todos los trabajos (pasados y programados) que afecten a un único dispositivo, puede usar la siguiente consulta:

```sql
SELECT * FROM devices.jobs
  WHERE devices.jobs.deviceId = 'myDeviceId'
```

Observe cómo esta consulta proporciona el estado específico del dispositivo (y posiblemente la respuesta del método directo) para cada trabajo devuelto.

También es posible filtrar por condiciones booleanas arbitrarias en todas las propiedades de objeto de la colección **devices.jobs**.

Por ejemplo, para recuperar todos los trabajos de actualización de dispositivo gemelo que se crearon después de septiembre de 2016 para un dispositivo específico, use la siguiente consulta:

```sql
SELECT * FROM devices.jobs
  WHERE devices.jobs.deviceId = 'myDeviceId'
    AND devices.jobs.jobType = 'scheduleTwinUpdate'
    AND devices.jobs.status = 'completed'
    AND devices.jobs.createdTimeUtc > '2016-09-01'
```

Asimismo, también es posible recuperar los resultados por dispositivo de un único trabajo.

```sql
SELECT * FROM devices.jobs
  WHERE devices.jobs.jobId = 'myJobId'
```

### <a name="limitations"></a>Limitaciones

Actualmente, las consultas en **devices.jobs** no admiten:

* proyecciones, por lo tanto, solo `SELECT *` es posible.
* condiciones que hagan referencia al dispositivo gemelo, además de a las propiedades de trabajo (vea la sección anterior).
* realizar agregaciones, como count, avg, group by.

## <a name="basics-of-an-iot-hub-query"></a>Conceptos básicos de una consulta de IoT Hub

Cada consulta de IoT Hub consta de las cláusulas SELECT y FROM, además de las cláusulas opcionales WHERE y GROUP BY. Cada consulta se ejecuta en una colección de documentos JSON, por ejemplo, dispositivos gemelos. La cláusula FORM indica la colección de documentos en la que se va a iterar (**devices**, **devices.modules** o **devices.jobs**). Después se aplica el filtro en la cláusula WHERE. Con agregaciones, los resultados de este paso se agrupan según se especifique en la cláusula GROUP BY. Para cada grupo, se genera una fila de acuerdo con lo especificado en la cláusula SELECT.

```sql
SELECT <select_list>
  FROM <from_specification>
  [WHERE <filter_condition>]
  [GROUP BY <group_specification>]
```

## <a name="from-clause"></a>Cláusula FROM

La cláusula **FROM <from_specification>** solo puede asumir tres valores: **FROM devices** para consultar los dispositivos gemelos, **FROM devices.modules** para consultar desde los módulos gemelos o **FROM devices.jobs** para consultar los detalles de un trabajo por dispositivo.

## <a name="where-clause"></a>WHERE, cláusula

La cláusula **WHERE <filter_condition>** es opcional. Especifica una o varias condiciones que los documentos JSON en la colección FROM deben satisfacer para incluirse como parte del resultado. Cualquier documento JSON debe evaluar las condiciones especificadas como "true" para que se incluya en el resultado.

Las condiciones permitidas se describen en la sección [Expresiones y condiciones](iot-hub-devguide-query-language.md#expressions-and-conditions).

## <a name="select-clause"></a>Cláusula SELECT

La cláusula **SELECT <select_list>** es obligatoria y especifica qué valores se recuperan de la consulta. Especifica los valores JSON que se usarán para generar nuevos objetos JSON.
Para cada elemento del subconjunto filtrado (y, opcionalmente, agrupado) de la colección FROM, la fase de proyección genera un nuevo objeto JSON. Este objeto se construye con los valores especificados en la cláusula SELECT.

Esta es la gramática de la cláusula SELECT:

```
SELECT [TOP <max number>] <projection list>

<projection_list> ::=
    '*'
    | <projection_element> AS alias [, <projection_element> AS alias]+

<projection_element> :==
    attribute_name
    | <projection_element> '.' attribute_name
    | <aggregate>

<aggregate> :==
    count()
    | avg(<projection_element>)
    | sum(<projection_element>)
    | min(<projection_element>)
    | max(<projection_element>)
```

**Attribute_name** hace referencia a cualquier propiedad del documento JSON en la colección FROM. Puede ver algunos ejemplos de las cláusulas SELECT en la sección Introducción a las consultas de dispositivos gemelos.

Actualmente, las cláusulas de selección distintas a **SELECT** * solo se admiten en las consultas agregadas de dispositivos gemelos.

## <a name="group-by-clause"></a>Cláusula GROUP BY

La cláusula **GROUP BY <group_specification>** es un paso opcional que se ejecuta después del filtro especificado en la cláusula WHERE y antes de la proyección especificada en la cláusula SELECT. Agrupa los documentos según el valor de un atributo. Estos grupos se usan para generar valores agregados, como se especifica en la cláusula SELECT.

Un ejemplo de consulta con GROUP BY es:

```sql
SELECT properties.reported.telemetryConfig.status AS status,
    COUNT() AS numberOfDevices
FROM devices
GROUP BY properties.reported.telemetryConfig.status
```

La sintaxis formal de GROUP BY es:

```
GROUP BY <group_by_element>
<group_by_element> :==
    attribute_name
    | < group_by_element > '.' attribute_name
```

**Attribute_name** hace referencia a cualquier propiedad del documento JSON en la colección FROM.

Actualmente, solo se admite la cláusula GROUP BY al consultar dispositivos gemelos.

> [!IMPORTANT]
> En la actualidad, el término `group` se trata en las consultas como si fuera una palabra clave especial. Si utiliza `group` como un nombre de propiedad, considere la posibilidad de incluirlo entre corchetes dobles para evitar errores; por ejemplo, `SELECT * FROM devices WHERE tags.[[group]].name = 'some_value'`.
>

## <a name="expressions-and-conditions"></a>Expresiones y condiciones

Brevemente, una *expresión*:

* Se evalúa como una instancia de tipo JSON (como booleano, número, cadena, matriz u objeto).
* Se define manipulando datos procedentes del documento JSON del dispositivo y constantes mediante funciones y operadores integrados.

Las *condiciones* son expresiones que se evalúan como un valor booleano. Cualquier constante distinta al booleano **true** se considera como **false**. Esta regla incluye **null**, **undefined**, cualquier instancia de objeto o matriz, cualquier cadena y el valor booleano **false**.

La sintaxis de las expresiones es:

```
<expression> ::=
    <constant> |
    attribute_name |
    <function_call> |
    <expression> binary_operator <expression> |
    <create_array_expression> |
    '(' <expression> ')'

<function_call> ::=
    <function_name> '(' expression ')'

<constant> ::=
    <undefined_constant>
    | <null_constant>
    | <number_constant>
    | <string_constant>
    | <array_constant>

<undefined_constant> ::= undefined
<null_constant> ::= null
<number_constant> ::= decimal_literal | hexadecimal_literal
<string_constant> ::= string_literal
<array_constant> ::= '[' <constant> [, <constant>]+ ']'
```

Para comprender lo que significa cada símbolo en la sintaxis de expresiones, consulte la tabla siguiente:

| Símbolo | Definición |
| --- | --- |
| attribute_name | Cualquier propiedad del documento JSON en la colección **FROM**. |
| binary_operator | Cualquier operador binario que figure en la sección [Operadores](#operators). |
| function_name| Cualquier función que figure en la sección [Funciones](#functions). |
| decimal_literal |Un valor float expresado en notación decimal. |
| hexadecimal_literal |Un número expresado por la cadena "0x" seguido de una cadena de dígitos hexadecimales. |
| string_literal |Los literales de cadena son cadenas Unicode representadas por una secuencia de cero o varios caracteres Unicode o secuencias de escape. Los literales de cadena se cierran entre comillas simples o dobles. Caracteres de escape permitidos: `\'`, `\"`, `\\`, `\uXXXX` para los caracteres Unicode definidos con 4 dígitos hexadecimales. |

### <a name="operators"></a>Operadores

Se admiten los siguientes operadores:

| Familia | Operadores |
| --- | --- |
| Aritméticos |+, -, *, /, % |
| Lógicos |AND, OR, NOT |
| De comparación |=, !=, <, >, <=, >=, <> |

### <a name="functions"></a>Functions

Cuando se consultan gemelos y trabajos, la única función admitida es:

| Función | Descripción |
| -------- | ----------- |
| IS_DEFINED(property) | Devuelve un valor booleano que indica si se ha asignado un valor a la propiedad (incluido `null`). |

En condiciones de rutas, se admiten las siguientes funciones matemáticas:

| Función | Descripción |
| -------- | ----------- |
| ABS(x) | Devuelve el valor absoluto (positivo) de la expresión numérica especificada. |
| EXP(x) | Devuelve el valor exponencial de la expresión numérica especificada (e^x). |
| POWER(x,y) | Devuelve el valor de la expresión especificada a la potencia especificada (x^y).|
| SQUARE(x)    | Devuelve el cuadrado del valor numérico especificado. |
| CEILING(x) | Devuelve el valor entero más pequeño mayor o igual que la expresión numérica especificada. |
| FLOOR(x) | Devuelve el entero más grande que sea menor o igual que la expresión numérica especificada. |
| SIGN(x) | Devuelve el signo positivo (+1), cero (0) o negativo (-1) de la expresión numérica especificada.|
| SQRT(x) | Devuelve la raíz cuadrada del valor numérico especificado. |

En condiciones de rutas, se admiten las funciones de conversión y comprobación de tipos siguientes:

| Función | Descripción |
| -------- | ----------- |
| AS_NUMBER | Convierte la cadena de entrada en un número. `noop` si la entrada es un número; `Undefined` si la cadena no representa un número.|
| IS_ARRAY | Devuelve un valor booleano que indica si el tipo de la expresión especificada es una matriz. |
| IS_BOOL | Devuelve un valor booleano que indica si el tipo de la expresión especificada es un valor booleano. |
| IS_DEFINED | Devuelve un valor booleano que indica si se ha asignado un valor a la propiedad. Solo se admite cuando el valor es de tipo primitivo. Los tipos primitivos son cadena, booleano, numérico o `null`. No se admiten los valores DateTime, los tipos de objeto ni las matrices. |
| IS_NULL | Devuelve un valor booleano que indica si el tipo de la expresión especificada es nulo. |
| IS_NUMBER | Devuelve un valor booleano que indica si el tipo de la expresión especificada es un número. |
| IS_OBJECT | Devuelve un valor booleano que indica si el tipo de la expresión especificada es un objeto JSON. |
| IS_PRIMITIVE | Devuelve un valor booleano que indica si el tipo de la expresión especificada es un tipo primitivo (cadena, booleano, numérico o `null`). |
| IS_STRING | Devuelve un valor booleano que indica si el tipo de la expresión especificada es una cadena. |

En condiciones de rutas, se admiten las siguientes funciones de cadena:

| Función | Descripción |
| -------- | ----------- |
| CONCAT(x, y, …) | Devuelve una cadena que es el resultado de concatenar dos o más valores de cadena. |
| LENGTH(x) | Devuelve el número de caracteres de la expresión de cadena especificada.|
| LOWER(x) | Devuelve una expresión de cadena después de convertir los datos de caracteres en mayúsculas a minúsculas. |
| UPPER(x) | Devuelve una expresión de cadena después de convertir datos de caracteres en minúsculas a mayúsculas. |
| SUBSTRING(string, start [, length]) | Devuelve parte de una expresión de cadena a partir de la posición de base cero del carácter especificado y continúa hasta la longitud especificada, o hasta el final de la cadena. |
| INDEX_OF(string, fragment) | Devuelve la posición inicial de la primera aparición de la expresión de la segunda cadena dentro de la primera expresión de cadena especificada, o -1 si no se encuentra la cadena.|
| STARTS_WITH(x, y) | Devuelve un valor booleano que indica si la primera expresión de cadena empieza con la segunda. |
| ENDS_WITH(x, y) | Devuelve un valor booleano que indica si la primera expresión de cadena finaliza con la segunda. |
| CONTAINS(x,y) | Devuelve un valor booleano que indica si la primera expresión de cadena contiene la segunda. |

## <a name="next-steps"></a>Pasos siguientes

Aprenda a ejecutar consultas en sus aplicaciones mediante los [SDK IoT de Azure ](iot-hub-devguide-sdks.md).
