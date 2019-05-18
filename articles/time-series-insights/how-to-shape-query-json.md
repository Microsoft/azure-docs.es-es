---
title: Procedimientos recomendados para dar forma a JSON en las consultas de Azure Time Series Insights | Microsoft Docs
description: Aprenda a mejorar la eficacia de consulta de Azure Time Series Insights.
services: time-series-insights
author: ashannon7
manager: cshankar
ms.service: time-series-insights
ms.topic: article
ms.date: 05/09/2019
ms.author: anshan
ms.custom: seodec18
ms.openlocfilehash: 535fe9a7920db89e434b4cc1b66aa38bf72a7829
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65790081"
---
# <a name="how-to-shape-json-to-maximize-query-performance"></a>Cómo dar forma a JSON para maximizar el rendimiento de las consultas 

Este artículo proporcionan instrucciones para dar forma a JSON, para maximizar la eficacia de las consultas de Azure Time Series Insights.

## <a name="video"></a>Vídeo

### <a name="learn-best-practices-for-shaping-json-to-meet-your-storage-needsbr"></a>Obtenga información sobre los procedimientos recomendados para dar forma a JSON para satisfacer las necesidades de almacenamiento.</br>

> [!VIDEO https://www.youtube.com/embed/b2BD5hwbg5I]

## <a name="best-practices"></a>Procedimientos recomendados

Es importante que piense en cómo envía eventos a Time Series Insights. Es decir, siempre debería:

1. Enviar los datos a través de la red de la forma más eficaz posible.
1. Asegúrese de que los datos se almacenan de forma que le permite realizar agregaciones adecuadas para su escenario.
1. Asegúrese de no alcancen los límites de la propiedad maximum de Time Series Insights de:
   - 600 propiedades (columnas) para entornos de S1,
   - 800 propiedades (columnas) para entornos de S2.

Las siguientes instrucciones le ayudan a garantizar el máximo rendimiento posible de consultas:

1. No use las propiedades dinámicas, como un identificador de etiqueta como nombre de propiedad, ya que esto contribuye a llegar al límite máximo de propiedades.
1. No envíe propiedades innecesarias. Si una propiedad de consulta no es necesaria, es mejor que no la envíe y evite las limitaciones de almacenamiento.
1. Use [datos de referencia](time-series-insights-add-reference-data-set.md) para evitar el envío de datos estáticos a través de la red.
1. Comparta propiedades de dimensión entre varios eventos para enviar datos a través de la red de forma más eficaz.
1. No utilice el anidamiento profundo de matriz. Time Series Insights admite hasta dos niveles de matrices anidadas que contienen objetos. Time Series Insights reduce las matrices en los mensajes, en varios eventos con pares de valor de propiedad.
1. Si solo existen algunas de las medidas para todos o la mayoría de los eventos, es mejor que envíe estas medidas como propiedades independientes dentro del mismo objeto. Al enviarlas por separado se reduce el número de eventos y podría hacer que las consultas sean más eficientes ya que se deben procesar menos eventos. Cuando existen varias medidas, al enviarlas como valores en una sola propiedad se minimiza la posibilidad de llegar al límite máximo de propiedades.

## <a name="example-overview"></a>Información general de ejemplo

Los dos ejemplos siguientes muestran los eventos de envío para resaltar las recomendaciones anteriores. Después de cada ejemplo puede ver cómo se han aplicado las recomendaciones.

Los ejemplos se basan en un escenario donde varios dispositivos envían señales o medidas. Las medidas o señales podrían ser Caudal, Presión de aceite del motor, Temperatura y Humedad. En el primer ejemplo, hay algunas medidas en todos los dispositivos. En el segundo ejemplo, hay muchos dispositivos y cada uno envía muchas medidas únicas.

## <a name="scenario-one-only-a-few-measurements-exist"></a>Escenario uno: existen sólo algunas medidas

> [!TIP]
> **Recomendación**: enviar cada señal de medida como una propiedad o columna independiente.

En el ejemplo siguiente, hay un único mensaje de IoT Hub, donde la matriz externa contiene una sección compartida de valores de dimensión comunes. La matriz externa usa datos de referencia para aumentar la eficacia del mensaje. Los datos de referencia contienen metadatos del dispositivo, que no cambian con cada evento, pero proporcionan propiedades útiles para el análisis de datos. Tanto los valores de dimensión común de procesamiento por lotes y el uso de datos de referencia, ahorran bytes enviados a través de la conexión, lo que hace que el mensaje sea más eficaz.

Carga JSON de ejemplo:

```json
[
    {
        "deviceId": "FXXX",
        "timestamp": "2018-01-17T01:17:00Z",
        "series": [
            {
                "Flow Rate ft3/s": 1.0172575712203979,
                "Engine Oil Pressure psi ": 34.7
            },
            {
                "Flow Rate ft3/s": 2.445906400680542,
                "Engine Oil Pressure psi ": 49.2
            }
        ]
    },
    {
        "deviceId": "FYYY",
        "timestamp": "2018-01-17T01:18:00Z",
        "series": [
            {
                "Flow Rate ft3/s": 0.58015072345733643,
                "Engine Oil Pressure psi ": 22.2
            }
        ]
    }
]
```

* Tabla de datos de referencia (propiedad de clave es **deviceId**):

   | deviceId | messageId | deviceLocation |
   | --- | --- | --- |
   | FXXX | LINE\_DATA | EU |
   | FYYY | LINE\_DATA | US |

* Tabla de eventos de Time Series Insights (después del acoplamiento):

   | deviceId | messageId | deviceLocation |  timestamp | series.Flow Rate ft3/s | series.Engine Oil Pressure psi |
   | --- | --- | --- | --- | --- | --- |
   | FXXX | LINE\_DATA | EU | 2018-01-17T01:17:00Z | 1.0172575712203979 | 34.7 |
   | FXXX | LINE\_DATA | EU | 2018-01-17T01:17:00Z | 2.445906400680542 | 49.2 |
   | FYYY | LINE\_DATA | US | 2018-01-17T01:18:00Z | 0.58015072345733643 | 22.2 |

encima:

- La columna **deviceId** actúa como el encabezado de columna para los distintos dispositivos en una línea. Si se intentase establecer el valor de deviceId como su propio nombre de propiedad, se habría limitado el número total de dispositivos a 595 (entornos S1) o 795 (entornos S2), con las otras cinco columnas.

- Se evitan las propiedades innecesarias, por ejemplo, información de marca y modelo, etc. Puesto que no se consultarán en el futuro, al eliminarlas se permite que la eficacia de red y almacenamiento sea mejor.

- Los datos de referencia se utilizan para reducir el número de bytes transferidos a través de la red. Los dos atributos, **messageId** y **deviceLocation**, se combinan mediante la propiedad de clave **deviceId**. Estos datos se combina con los datos de telemetría en tiempo de entrada y quedan almacenados en Time Series Insights para realizar consultas.

- Se utilizan dos niveles de anidamiento, que es la cantidad máxima de anidamiento compatible con Time Series Insights. Es fundamental para evitar matrices profundamente anidadas.

- Las medidas se envían como propiedades independientes en el mismo objeto, dado que hay varias medidas. En este caso, **series.Flow Rate psi** y **series.Engine Oil Pressure ft3/s** son columnas únicas.

## <a name="scenario-two-several-measures-exist"></a>Escenario dos: existen varias medidas

> [!TIP]
> **Recomendación:** Envíe las mediciones como tuplas de "tipo", "unidad", "valor".

Carga JSON de ejemplo:

```json
[
    {
        "deviceId": "FXXX",
        "timestamp": "2018-01-17T01:17:00Z",
        "series": [
            {
                "tagId": "pumpRate",
                "value": 1.0172575712203979
            },
            {
                "tagId": "oilPressure",
                "value": 49.2
            },
            {
                "tagId": "pumpRate",
                "value": 2.445906400680542
            },
            {
                "tagId": "oilPressure",
                "value": 34.7
            }
        ]
    },
    {
        "deviceId": "FYYY",
        "timestamp": "2018-01-17T01:18:00Z",
        "series": [
            {
                "tagId": "pumpRate",
                "value": 0.58015072345733643
            },
            {
                "tagId": "oilPressure",
                "value": 22.2
            }
        ]
    }
]
```

* Datos de referencia (las propiedades claves son **deviceId** y **series.tagId**):

   | deviceId | series.tagId | messageId | deviceLocation | Tipo | unidad |
   | --- | --- | --- | --- | --- | --- |
   | FXXX | pumpRate | LINE\_DATA | EU | Caudal | ft3/s |
   | FXXX | oilPressure | LINE\_DATA | EU | Presión de aceite del motor | psi |
   | FYYY | pumpRate | LINE\_DATA | US | Caudal | ft3/s |
   | FYYY | oilPressure | LINE\_DATA | US | Presión de aceite del motor | psi |

* Tabla de eventos de Time Series Insights (después del acoplamiento):

   | deviceId | series.tagId | messageId | deviceLocation | Tipo | unidad |  timestamp | series.value |
   | --- | --- | --- | --- | --- | --- | --- | --- |
   | FXXX | pumpRate | LINE\_DATA | EU | Caudal | ft3/s | 2018-01-17T01:17:00Z | 1.0172575712203979 | 
   | FXXX | oilPressure | LINE\_DATA | EU | Presión de aceite del motor | psi | 2018-01-17T01:17:00Z | 34.7 |
   | FXXX | pumpRate | LINE\_DATA | EU | Caudal | ft3/s | 2018-01-17T01:17:00Z | 2.445906400680542 | 
   | FXXX | oilPressure | LINE\_DATA | EU | Presión de aceite del motor | Psi | 2018-01-17T01:17:00Z | 49.2 |
   | FYYY | pumpRate | LINE\_DATA | US | Caudal | ft3/s | 2018-01-17T01:18:00Z | 0.58015072345733643 |
   | FYYY | oilPressure | LINE\_DATA | US | Presión de aceite del motor | psi | 2018-01-17T01:18:00Z | 22.2 |

encima:

- Las columnas **deviceId** y **series.tagId** actuar como los encabezados de columna para los distintos dispositivos y las etiquetas de una flota. Al usar cada una como su propio atributo, se limitaría la consulta a 594 (entornos S1) o 794 (entornos S2) dispositivos en total, con las otras seis columnas.

- se han evitado propiedades innecesarias, por la razón citada en el primer ejemplo.

- datos de referencia se usan para reducir el número de bytes transferidos a través de la red mediante la introducción de **deviceId**, para un único par de **messageId** y **deviceLocation**. Se utiliza una clave compuesta, **series.tagId**, para el par único de **tipo** y **unidad**. La clave compuesta permite usar el par **deviceId** y **series.tagId** para hacer referencia a cuatro valores: **messageId, deviceLocation, tipo** y **unidad**. Estos datos se combina con los datos de telemetría en tiempo de entrada y quedan almacenados en Time Series Insights para realizar consultas.

- se usan dos niveles de anidamiento, por la razón citada en el primer ejemplo.

### <a name="for-both-scenarios"></a>Para ambos escenarios

Si tiene una propiedad con un gran número de valores posibles, es mejor que enviarlos como valores distintos dentro de una sola columna, en lugar de crear una nueva columna para cada valor. De los dos ejemplos anteriores:

  - En el primer ejemplo, hay algunas propiedades que tienen varios valores, por lo que es adecuado hacer que cada uno sea una propiedad independiente.
  - Pero, en el segundo ejemplo, puede ver que las medidas no se especifican como propiedades individuales, sino como una matriz de valores y medidas en una propiedad de la serie común. Se envía una nueva clave, **tagId**, que crea una nueva columna, **series.tagId** en la tabla acoplada. Se crean nuevas propiedades, **tipo** y **unidad**, con los datos de referencia, lo que impide que se alcance el límite de propiedades.

## <a name="next-steps"></a>Pasos siguientes

- Lectura [sintaxis de consulta de Azure Time Series Insights](/rest/api/time-series-insights/ga-query-syntax) para obtener más información sobre la sintaxis de consulta para la API de REST de acceso a los datos de Time Series Insights.

- Obtenga información sobre [cómo los eventos de forma](./time-series-insights-send-events.md).
