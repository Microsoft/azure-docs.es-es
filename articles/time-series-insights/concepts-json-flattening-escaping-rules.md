---
title: 'Reglas de acoplamiento y escape de JSON: Azure Time Series Insights Gen2 | Microsoft Docs'
description: Aprenda sobre el acoplamiento, el escape y el control de matrices de JSON en Azure Time Series Insights Gen2.
author: lyrana
ms.author: lyhughes
manager: deepakpalled
ms.workload: big-data
ms.service: time-series-insights
services: time-series-insights
ms.topic: conceptual
ms.date: 07/07/2020
ms.openlocfilehash: 0cf0ef97cc1e06906a529c577e9c2578e5091ef4
ms.sourcegitcommit: 8a7b82de18d8cba5c2cec078bc921da783a4710e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89050733"
---
# <a name="ingestion-rules"></a>Reglas de ingesta
### <a name="json-flattening-escaping-and-array-handling"></a>Acoplamiento de JSON, escape y control de matrices

El entorno de Azure Time Series Insights Gen2 crea dinámicamente las columnas de los almacenes intermedios y en reposo conforme a un conjunto determinado de convenciones de nomenclatura. Cuando se ingiere un evento, se aplica un conjunto de reglas a la carga y los nombres de propiedad de JSON. Entre ellas se incluyen el escape de determinados caracteres especiales y el acoplamiento de objetos JSON anidados. Es importante conocer estas reglas para que comprenda cómo influirá la forma de JSON en cómo se almacenan y consultan los eventos. Consulte la tabla siguiente para ver la lista completa de reglas. En los ejemplos A y B también se muestra cómo se pueden procesar por lotes de forma eficaz varias series temporales en una matriz.

> [!IMPORTANT]
>
> * Revise las reglas que se indican a continuación antes de seleccionar una [Propiedad de identificador de serie temporal](time-series-insights-update-how-to-id.md) y el origen del evento [Propiedades de marca de tiempo](concepts-streaming-ingestion-event-sources.md#event-source-timestamp). Si el identificador de TS o la marca de tiempo se encuentran en un objeto anidado o tiene uno o más de los caracteres especiales siguientes, es importante asegurarse de que el nombre de la propiedad que proporcione coincide con el nombre de la columna *después* de que se hayan aplicado las reglas de ingesta. Vea el ejemplo [B](concepts-json-flattening-escaping-rules.md#example-b) siguiente.

| Regla | Ejemplo de JSON | [Sintaxis de Time Series Expression](https://docs.microsoft.com/rest/api/time-series-insights/reference-time-series-expression-syntax) | Nombre de la columna de propiedad en Parquet
|---|---|---|---|
| El tipo de datos de Azure Time Series Insights Gen2 se anexa al final del nombre de la columna como "_\<dataType\>" | ```"type": "Accumulated Heat"``` | `$event.type.String` |`type_string` |
| La [propiedad timestamp](concepts-streaming-ingestion-event-sources.md#event-source-timestamp) del origen del evento se guarda en Azure Time Series Insights Gen2 como "timestamp" en el almacenamiento y el valor se almacena en UTC. Puede personalizar la propiedad marca de tiempo de orígenes de eventos para satisfacer las necesidades de la solución, pero el nombre de la columna en el almacenamiento intermedio y en frío es "marca de tiempo". Otras propiedades datetime JSON que no sean la marca de tiempo de origen del evento se guardarán con "_datetime" en el nombre de columna, como se mencionó en la regla anterior.  | ```"ts": "2020-03-19 14:40:38.318"``` |  `$event.$ts` | `timestamp` |
| Nombres de propiedad JSON que incluyen los caracteres especiales. [  \ y ' se convierten en caracteres de escape con [' y ']  |  ```"id.wasp": "6A3090FD337DE6B"``` |  `$event['id.wasp'].String` | `['id.wasp']_string` |
| Dentro de [' y '], hay un escape adicional de comillas simples y barras diagonales inversas. Una comilla simple se escribirá como \’ y una barra diagonal inversa, como \\\ | ```"Foo's Law Value": "17.139999389648"``` | `$event['Foo\'s Law Value'].Double` | `['Foo\'s Law Value']_double` |
| Los objetos JSON anidados se acoplan con un punto como separador. Se admite el anidamiento de hasta 10 niveles. |  ```"series": {"value" : 316 }``` | `$event.series.value.Long`, `$event['series']['value'].Long` o `$event.series['value'].Long` |  `series.value_long` |
| Las matrices de tipos primitivos se almacenan como el tipo dinámico |  ```"values": [154, 149, 147]``` | Los tipos dinámicos solo se pueden recuperar a través de la API de [GetEvents](https://docs.microsoft.com/rest/api/time-series-insights/dataaccessgen2/query/execute#getevents) | `values_dynamic` |
| Las matrices que contienen objetos tienen dos comportamientos en función del contenido del objeto: Si los identificadores de TS o las propiedades de marca de tiempo están dentro de los objetos de una matriz, la matriz se expandirá de forma que la carga útil inicial de JSON produzca múltiples eventos. Esto permite procesar por lotes varios eventos en una estructura JSON. Cualquier propiedad de nivel superior que esté en el mismo nivel que la matriz se guardará con cada objeto no expandido. Si los identificadores de TS y la marca de tiempo *no* están en la matriz, se guardarán como el tipo dinámico. | Vea los ejemplos [A](concepts-json-flattening-escaping-rules.md#example-a), [B](concepts-json-flattening-escaping-rules.md#example-b) y [C](concepts-json-flattening-escaping-rules.md#example-c) siguientes
| Las matrices que contienen elementos mixtos no están acopladas. |  ```"values": ["foo", {"bar" : 149}, 147]``` | Los tipos dinámicos solo se pueden recuperar a través de la API de [GetEvents](https://docs.microsoft.com/rest/api/time-series-insights/dataaccessgen2/query/execute#getevents) | `values_dynamic` |
| 512 caracteres es el límite del nombre de la propiedad JSON. Si el nombre supera los 512 caracteres, se truncará a 512 y se anexará un '_<'hashCode'>'. **Nota:** Esto también se aplica a los nombres de propiedad que se han concatenado del objeto acoplado, lo que denota una ruta de objeto anidada. |``"data.items.datapoints.values.telemetry<...continuing to over 512 chars>" : 12.3440495`` |`"$event.data.items.datapoints.values.telemetry<...continuing to include all chars>.Double"` | `data.items.datapoints.values.telemetry<...continuing to 512 chars>_912ec803b2ce49e4a541068d495ab570_double` |

## <a name="understanding-the-dual-behavior-for-arrays"></a>Descripción del comportamiento dual de las matrices

Las matrices de objetos se almacenan de forma completa o dividida en varios eventos, en función de cómo se hayan modelado los datos. Esto le permite usar una matriz para procesar los eventos y evitar repetir las propiedades de telemetría definidas en el nivel de objeto raíz. El procesamiento por lotes puede ser ventajoso, ya que se producen menos instancias de Event Hubs o mensajes de IoT Hub enviados. 

Sin embargo, en algunos casos, las matrices que contienen objetos solo son significativas en el contexto de otros valores. La creación de varios eventos representaría los datos sin significado. Para asegurarse de que una matriz de objetos se almacena tal cual como un tipo dinámico, siga la guía de modelado de datos que se indican a continuación y eche un vistazo al [ejemplo C](concepts-json-flattening-escaping-rules.md#example-c)

### <a name="how-do-i-know-if-my-array-of-objects-will-produce-multiple-events"></a>¿Cómo saber si la matriz de objetos producirá varios eventos?

Si una o más de las propiedades de identificador de serie temporal están anidadas dentro de los objetos de una matriz, *o* si la propiedad de marca de tiempo de origen del evento está anidada, el motor de ingesta la dividirá para crear múltiples eventos. Los nombres de propiedad que proporcionó para los identificadores de TS y la marca de tiempo deben seguir las reglas de acoplamiento anteriores y, por lo tanto, indicarán la forma del JSON. Vea los ejemplos siguientes y consulte la guía sobre cómo [Seleccionar una propiedad de identificador de serie temporal.](time-series-insights-update-how-to-id.md)

### <a name="example-a"></a>Ejemplo A:
Identificador de serie temporal en la raíz del objeto y marca de tiempo anidada<br/>
**Identificador de serie temporal del entorno:** `"id"`<br/>
**Marca de tiempo de origen del evento:** `"values.time"`<br/>
**Carga de JSON:**

```JSON
[
    {
        "id": "caaae533-1d6c-4f58-9b75-da102bcc2c8c",
        "values": [
            {
                "time": "2020-05-01T00:59:59.000Z",
                "value": 25.6073
            },
            {
                "time": "2020-05-01T01:00:29.000Z",
                "value": 43.9077
            }
        ]
    },
    {
        "id": "1ac87b74-0865-4a07-b512-56602a3a576f",
        "values": [
            {
                "time": "2020-05-01T00:59:59.000Z",
                "value": 0.337288
            },
            {
                "time": "2020-05-01T01:00:29.000Z",
                "value": 4.76562
            }
        ]
    }
]
```

**Resultado en el archivo de Parquet:**
<br/>
La configuración y la carga anterior producirán tres columnas y cuatro eventos

| timestamp  | id_string | values.value_double 
| ---- | ---- | ---- | 
| `2020-05-01T00:59:59.000Z` | `caaae533-1d6c-4f58-9b75-da102bcc2c8c`| ``25.6073`` | 
| `2020-05-01T01:00:29.000Z` |`caaae533-1d6c-4f58-9b75-da102bcc2c8c` | ``43.9077`` | 
| `2020-05-01T00:59:59.000Z` | `1ac87b74-0865-4a07-b512-56602a3a576f` | ``0.337288`` | 
| `2020-05-01T01:00:29.000Z` | `1ac87b74-0865-4a07-b512-56602a3a576f` | ``4.76562`` | 

### <a name="example-b"></a>Ejemplo B:
Combinación de un identificador de serie temporal con una propiedad anidada<br/> 
**Identificador de serie temporal del entorno:** `"plantId"` y `"telemetry.tagId"`<br/>
**Marca de tiempo de origen del evento:** `"timestamp"`<br/>
**Carga de JSON:**

```JSON
[
    {
        "plantId": "9336971",
        "timestamp": "2020-01-22T16:38:09Z",
        "telemetry": [
            {
                "tagId": "100231-A-A6",
                "tagValue": -31.149018
            },
            {
                "tagId": "100231-A-A1",
                "tagValue": 20.560796
            },
            {
                "tagId": "100231-A-A9",
                "tagValue": 177
            },
            {
                "tagId": "100231-A-A8",
                "tagValue": 420
            },
        ]
    },
    {
        "plantId": "9336971",
        "timestamp": "2020-01-22T16:42:14Z",
        "telemetry": [
            {
                "tagId": "103585-A-A7",
                "value": -30.9918
            },
            {
                "tagId": "103585-A-A4",
                "value": 19.960796
            }
        ]
    }
]
```

**Resultado en el archivo de Parquet:**
<br/>
La configuración y la carga anterior producirán cuatro columnas y seis eventos

| timestamp  | plantId_string | telemetry.tagId_string | telemetry.value_double 
| ---- | ---- | ---- | ---- |
| `2020-01-22T16:38:09Z` | `9336971`| ``100231-A-A6`` |  -31.149018 |
| `2020-01-22T16:38:09Z` |`9336971` | ``100231-A-A1`` | 20.560796 |
| `2020-01-22T16:38:09Z` | `9336971` | ``100231-A-A9`` | 177 |
| `2020-01-22T16:38:09Z` | `9336971` | ``100231-A-A8`` | 420 |
| `2020-01-22T16:42:14Z` | `9336972` | ``100231-A-A7`` | -30.9918 |  
| `2020-01-22T16:42:14Z` | `9336972` | ``100231-A-A4`` | 19.960796 | 

### <a name="example-c"></a>Ejemplo C:
El identificador de serie temporal y la marca de tiempo se encuentran en la raíz del objeto<br/> 
**Identificador de serie temporal del entorno:** `"id"`<br/>
**Marca de tiempo de origen del evento:** `"timestamp"`<br/>
**Carga de JSON:**

```JSON
{
    "id": "800500054755",
    "timestamp": "2020-11-01T10:00:00.000Z",
    "datapoints": [{
            "value": 120
        },
        {
            "value": 124
        }
    ]
}
```

**Resultado en el archivo de Parquet:**
<br/>
La configuración y la carga anterior producirán tres columnas y un evento

| timestamp  | id_string | datapoints_dynamic  
| ---- | ---- | ---- | 
| `2020-11-01T10:00:00.000Z` | `800500054755`| ``[{"value": 120},{"value":124}]`` |

## <a name="next-steps"></a>Pasos siguientes

* Comprender las [limitaciones de rendimiento](./concepts-streaming-ingress-throughput-limits.md) del entorno
