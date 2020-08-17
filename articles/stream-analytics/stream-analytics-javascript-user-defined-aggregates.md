---
title: Agregados definidos por el usuario de JavaScript en Azure Stream Analytics
description: En este artículo se describe cómo realizar mecanismos de consulta avanzados con agregados definidos por el usuario de JavaScript en Azure Stream Analytics.
author: rodrigoaatmicrosoft
ms.author: rodrigoa
ms.reviewer: mamccrea
ms.service: stream-analytics
ms.topic: conceptual
ms.date: 10/28/2017
ms.custom: devx-track-javascript
ms.openlocfilehash: 433b9ad0f86b84538163b142d762d4e2fb7e8159
ms.sourcegitcommit: 0b8320ae0d3455344ec8855b5c2d0ab3faa974a3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/30/2020
ms.locfileid: "87430829"
---
# <a name="azure-stream-analytics-javascript-user-defined-aggregates"></a>Agregados definidos por el usuario en JavaScript para Azure Stream Analytics
 
Azure Stream Analytics admite agregados definidos por el usuario (UDA) escritos en JavaScript, que le permiten implementar lógica empresarial con estado compleja. Dentro de UDA tiene control completo sobre la estructura de los datos de estado, la acumulación y desacumulación de estado y el cálculo de resultados de agregados. El artículo presenta las dos interfaces de UDA en JavaScript diferentes, los pasos para crear un UDA y cómo usar UDA con operaciones basadas en ventanas en consultas de Stream Analytics.

## <a name="javascript-user-defined-aggregates"></a>Agregados definidos por el usuario en JavaScript

Los agregados definidos por el usuario se usan sobre una especificación de ventana de tiempo para agregarlos sobre los eventos de esa ventana y generar un valor con un único resultado. Actualmente hay dos tipos de interfaces de UDA que admite Stream Analytics: AccumulateOnly y AccumulateDeaccumulate. Ambos tipos de UDA se pueden usar con ventanas de saltos de tamaño constante, ventanas de salto, ventanas deslizantes y ventanas de sesión. UDA AccumulateDeaccumulate funciona mejor que UDA AccumulateOnly cuando se usa en combinación con ventanas de salto, ventanas deslizantes y ventanas de sesión. El tipo que elija dependerá del algoritmo que utilice.

### <a name="accumulateonly-aggregates"></a>Agregados de AccumulateOnly

Los agregados de AccumulateOnly solo pueden acumular nuevos eventos a su estado, el algoritmo no permite la desacumulación de valores. Elija este tipo de agregado cuando sea imposible la desacumulación de la información de un evento de su valor de estado. A continuación se muestra la plantilla de JavaScript para los agregados de AccumulatOnly:

```JavaScript
// Sample UDA which state can only be accumulated.
function main() {
    this.init = function () {
        this.state = 0;
    }

    this.accumulate = function (value, timestamp) {
        this.state += value;
    }

    this.computeResult = function () {
        return this.state;
    }
}
```

### <a name="accumulatedeaccumulate-aggregates"></a>Agregados de AccumulateDeaccumulate

Los agregados de AccumulateDeaccumulate permiten la desacumulación de un valor acumulado anterior del estado; por ejemplo, quitar un par clave-valor de una lista de valores de evento o restar un valor de un estado de agregado de suma. A continuación se muestra la plantilla de JavaScript para los agregados de AccumulateDeaccumulate:

```JavaScript
// Sample UDA which state can be accumulated and deaccumulated.
function main() {
    this.init = function () {
        this.state = 0;
    }

    this.accumulate = function (value, timestamp) {
        this.state += value;
    }

    this.deaccumulate = function (value, timestamp) {
        this.state -= value;
    }

    this.deaccumulateState = function (otherState){
        this.state -= otherState.state;
    }

    this.computeResult = function () {
        return this.state;
    }
}
```

## <a name="uda---javascript-function-declaration"></a>UDA: declaración de función de JavaScript

Cada UDA de JavaScript se define mediante una declaración de objeto de función. Estos son los elementos principales de una definición de UDA.

### <a name="function-alias"></a>Alias de función

El alias de función es el identificador de UDA. Cuando se llame en consultas de Stream Analytics, use siempre UDA en combinación con "uda". Prefix.

### <a name="function-type"></a>Tipo de función

En UDA, el tipo de función debe ser **UDA de Javascript**.

### <a name="output-type"></a>Tipo de salida

Un tipo específico que admite el trabajo de Stream Analytics o "cualquiera" si quiere gestionar el tipo de su consulta.

### <a name="function-name"></a>Nombre de función

El nombre de este objeto de función. El nombre de la función debe coincidir con el alias de UDA.

### <a name="method---init"></a>Método init()

El método init() inicializa el estado del agregado. Este método se llama cuando se inicia la ventana.

### <a name="method--accumulate"></a>Método accumulate()

El método accumulate() calcula el estado de UDA según el estado anterior y los valores de evento actuales. Este método se llama cuando un evento entra en una ventana de tiempo (TUMBLINGWINDOW, HOPPINGWINDOW, SLIDINGWINDOW o SESSIONWINDOW).

### <a name="method--deaccumulate"></a>Método deaccumulate()

El método deaccumulate() vuelve a calcular el estado según el estado anterior y los valores de evento actuales. Este método se llama cuando un evento sale de una ventana SLIDINGWINDOW o SESSIONWINDOW.

### <a name="method--deaccumulatestate"></a>Método deaccumulateState()

El método deaccumulateState() vuelve a calcular el estado según el estado anterior y el estado de un salto. Este método se llama cuando un conjunto de eventos sale de una ventana HOPPINGWINDOW.

### <a name="method--computeresult"></a>Método computeResult()

El método computeResult() devuelve resultados de agregados basados en el estado actual. Este método se llama al final de una ventana de tiempo (TUMBLINGWINDOW, HOPPINGWINDOW, SLIDINGWINDOW o SESSIONWINDOW).

## <a name="javascript-uda-supported-input-and-output-data-types"></a>Tipos de datos de entrada y salida admitidos por UDA de JavaScript
En el caso de tipos de datos de UDA de JavaScript, consulte la sección **Conversión de tipos de Stream Analytics y JavaScript** de [Integración de las UDF de JavaScript](stream-analytics-javascript-user-defined-functions.md).

## <a name="adding-a-javascript-uda-from-the-azure-portal"></a>Adición de un UDA de JavaScript desde Azure Portal

Este artículo le guía por el proceso de creación de un UDA desde el portal. En el ejemplo que se usa aquí se calculan las medias ponderadas de tiempo.

Ahora vamos a crear un UDA de JavaScript en un trabajo de ASA mediante estos pasos.

1. Inicie sesión en Azure Portal y busque el trabajo de Stream Analytics existente.
1. A continuación, haga clic en el vínculo de funciones en **TOPOLOGÍA DE TRABAJO**.
1. Haga clic en el icono **Agregar** para agregar una nueva función.
1. En la vista Nueva función, seleccione **UDA de JavaScript** como el tipo de función y verá que aparece una plantilla de UDA predeterminada en el editor.
1. Como alias de UDA use "TWA" y cambie la implementación de función como se indica a continuación:

    ```JavaScript
    // Sample UDA which calculate Time-Weighted Average of incoming values.
    function main() {
        this.init = function () {
            this.totalValue = 0.0;
            this.totalWeight = 0.0;
        }

        this.accumulate = function (value, timestamp) {
            this.totalValue += value.level * value.weight;
            this.totalWeight += value.weight;

        }

        // Uncomment below for AccumulateDeaccumulate implementation
        /*
        this.deaccumulate = function (value, timestamp) {
            this.totalValue -= value.level * value.weight;
            this.totalWeight -= value.weight;
        }

        this.deaccumulateState = function (otherState){
            this.state -= otherState.state;
            this.totalValue -= otherState.totalValue;
            this.totalWeight -= otherState.totalWeight;
        }
        */

        this.computeResult = function () {
            if(this.totalValue == 0) {
                result = 0;
            }
            else {
                result = this.totalValue/this.totalWeight;
            }
            return result;
        }
    }
    ```

1. Tras hacer clic en el botón "Guardar", el UDA aparece en la lista de funciones.

1. Haga clic en la nueva función "TWA" para comprobar la definición de función.

## <a name="calling-javascript-uda-in-asa-query"></a>Llamada de un UDA de JavaScript en una consulta de ASA

En Azure Portal, abra el trabajo, edite la consulta y llame a la función TWA() con el prefijo obligatorio "uda.". Por ejemplo:

```SQL
WITH value AS
(
    SELECT
    NoiseLevelDB as level,
    DurationSecond as weight
FROM
    [YourInputAlias] TIMESTAMP BY EntryTime
)
SELECT
    System.Timestamp as ts,
    uda.TWA(value) as NoseDoseTWA
FROM value
GROUP BY TumblingWindow(minute, 5)
```

## <a name="testing-query-with-uda"></a>Prueba de la consulta con UDA

Cree un archivo JSON local con el contenido siguiente, cárguelo en el trabajo de Stream Analytics y pruebe la consulta anterior.

```JSON
[
  {"EntryTime": "2017-06-10T05:01:00-07:00", "NoiseLevelDB": 80, "DurationSecond": 22.0},
  {"EntryTime": "2017-06-10T05:02:00-07:00", "NoiseLevelDB": 81, "DurationSecond": 37.8},
  {"EntryTime": "2017-06-10T05:02:00-07:00", "NoiseLevelDB": 85, "DurationSecond": 26.3},
  {"EntryTime": "2017-06-10T05:03:00-07:00", "NoiseLevelDB": 95, "DurationSecond": 13.7},
  {"EntryTime": "2017-06-10T05:03:00-07:00", "NoiseLevelDB": 88, "DurationSecond": 10.3},
  {"EntryTime": "2017-06-10T05:05:00-07:00", "NoiseLevelDB": 103, "DurationSecond": 5.5},
  {"EntryTime": "2017-06-10T05:06:00-07:00", "NoiseLevelDB": 99, "DurationSecond": 23.0},
  {"EntryTime": "2017-06-10T05:07:00-07:00", "NoiseLevelDB": 108, "DurationSecond": 1.76},
  {"EntryTime": "2017-06-10T05:07:00-07:00", "NoiseLevelDB": 79, "DurationSecond": 17.9},
  {"EntryTime": "2017-06-10T05:08:00-07:00", "NoiseLevelDB": 83, "DurationSecond": 27.1},
  {"EntryTime": "2017-06-10T05:09:00-07:00", "NoiseLevelDB": 91, "DurationSecond": 17.1},
  {"EntryTime": "2017-06-10T05:09:00-07:00", "NoiseLevelDB": 115, "DurationSecond": 7.9},
  {"EntryTime": "2017-06-10T05:09:00-07:00", "NoiseLevelDB": 80, "DurationSecond": 28.3},
  {"EntryTime": "2017-06-10T05:10:00-07:00", "NoiseLevelDB": 55, "DurationSecond": 18.2},
  {"EntryTime": "2017-06-10T05:10:00-07:00", "NoiseLevelDB": 93, "DurationSecond": 25.8},
  {"EntryTime": "2017-06-10T05:11:00-07:00", "NoiseLevelDB": 83, "DurationSecond": 11.4},
  {"EntryTime": "2017-06-10T05:12:00-07:00", "NoiseLevelDB": 89, "DurationSecond": 7.9},
  {"EntryTime": "2017-06-10T05:15:00-07:00", "NoiseLevelDB": 112, "DurationSecond": 3.7},
  {"EntryTime": "2017-06-10T05:15:00-07:00", "NoiseLevelDB": 93, "DurationSecond": 9.7},
  {"EntryTime": "2017-06-10T05:18:00-07:00", "NoiseLevelDB": 96, "DurationSecond": 3.7},
  {"EntryTime": "2017-06-10T05:20:00-07:00", "NoiseLevelDB": 108, "DurationSecond": 0.99},
  {"EntryTime": "2017-06-10T05:20:00-07:00", "NoiseLevelDB": 113, "DurationSecond": 25.1},
  {"EntryTime": "2017-06-10T05:22:00-07:00", "NoiseLevelDB": 110, "DurationSecond": 5.3}
]
```

## <a name="get-help"></a>Obtener ayuda

Para ayuda adicional, pruebe nuestra [Página de preguntas y respuestas de Microsoft sobre Azure Stream Analytics](https://docs.microsoft.com/answers/topics/azure-stream-analytics.html).

## <a name="next-steps"></a>Pasos siguientes

* [Introducción a Azure Stream Analytics](stream-analytics-introduction.md)
* [Introducción al uso de Azure Stream Analytics](stream-analytics-real-time-fraud-detection.md)
* [Escalación de trabajos de Azure Stream Analytics](stream-analytics-scale-jobs.md)
* [Referencia del lenguaje de consulta de Azure Stream Analytics](https://docs.microsoft.com/stream-analytics-query/stream-analytics-query-language-reference)
* [Referencia de API de REST de administración de Azure Stream Analytics](https://msdn.microsoft.com/library/azure/dn835031.aspx)
