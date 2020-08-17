---
title: 'Procedimientos recomendados para dar forma a JSON: consultas de Azure Time Series Insights | Microsoft Docs'
description: Aprenda a mejorar la eficacia de las consultas de Azure Time Series Insights dando forma a JSON.
services: time-series-insights
author: deepakpalled
ms.author: dpalled
manager: diviso
ms.service: time-series-insights
ms.topic: article
ms.date: 08/12/2020
ms.custom: seodec18
ms.openlocfilehash: 1a7a88e0db38f399dc47c030f3b97f6b26f4da07
ms.sourcegitcommit: c28fc1ec7d90f7e8b2e8775f5a250dd14a1622a6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/13/2020
ms.locfileid: "88168242"
---
# <a name="shape-json-to-maximize-query-performance-in-your-gen1-environment"></a>Dar forma a JSON para maximizar el rendimiento de las consultas en un entorno de Gen1

En este artículo se proporciona orientación sobre cómo dar forma a JSON para maximizar la eficacia de las consultas de Azure Time Series Insights.

## <a name="video"></a>Vídeo

### <a name="learn-best-practices-for-shaping-json-to-meet-your-storage-needsbr"></a>Obtenga procedimientos recomendados para dar forma a JSON y, de este modo, satisfacer sus necesidades de almacenamiento.</br>

> [!VIDEO <https://www.youtube.com/embed/b2BD5hwbg5I>]

## <a name="best-practices"></a>Procedimientos recomendados

Piense en cómo envía eventos a Azure Time Series Insights. Concretamente, siempre hace lo siguiente:

1. Enviar los datos a través de la red de la forma más eficaz posible.
1. Asegurarse de que los datos se almacenan de forma que le permite realizar agregaciones adecuadas para su escenario.
1. Asegúrese de que no alcanza los límites máximos de propiedades de Azure Time Series Insights de:
   - 600 propiedades (columnas) para entornos de S1,
   - 800 propiedades (columnas) para entornos de S2.

Las siguientes instrucciones le ayudan a garantizar el máximo rendimiento posible de consultas:

1. No use propiedades dinámicas como, por ejemplo, un identificador de etiqueta, como nombre de propiedad. Este uso contribuye a alcanzar el límite máximo de propiedades.
1. No envíe propiedades innecesarias. Si una propiedad de consulta no es necesaria, es mejor que no la envíe. Así evita las limitaciones de almacenamiento.
1. Use [datos de referencia](time-series-insights-add-reference-data-set.md) para evitar el envío de datos estáticos a través de la red.
1. Comparta propiedades de dimensión entre varios eventos para enviar datos a través de la red de forma más eficaz.
1. No utilice el anidamiento profundo de matriz. Azure Time Series Insights admite hasta dos niveles de matrices anidadas que contienen objetos. Igualmente, Azure Time Series Insights acopla las matrices en mensajes de varios eventos con pares de valores de propiedad.
1. Si solo existen algunas de las medidas para todos o la mayoría de los eventos, es mejor que envíe estas medidas como propiedades independientes dentro del mismo objeto. Al enviarlas por separado se reduce el número de eventos y podría mejorar el rendimiento de consultas, ya que se deben procesar menos eventos. Cuando existen varias medidas, al enviarlas como valores en una sola propiedad se minimiza la posibilidad de alcanzar el límite máximo de propiedades.

## <a name="example-overview"></a>Información general de ejemplo

Los dos ejemplos siguientes muestran cómo enviar eventos para resaltar las recomendaciones anteriores. Después de cada ejemplo, puede revisar cómo se han aplicado las recomendaciones.

Los ejemplos se basan en un escenario donde varios dispositivos envían señales o medidas. Las medidas o señales podrían ser Caudal, Presión de aceite del motor, Temperatura y Humedad. En el primer ejemplo, hay algunas medidas en todos los dispositivos. El segundo ejemplo tiene muchos dispositivos y cada uno envía muchas medidas únicas.

## <a name="scenario-one-only-a-few-measurements-exist"></a>Escenario uno: solo existen algunas medidas

> [!TIP]
> Recomendamos que envíe cada medida o señal como propiedad o columna independiente.

En el ejemplo siguiente, hay un único mensaje de Azure IoT Hub donde la matriz externa contiene una sección compartida de valores de dimensión comunes. La matriz externa usa datos de referencia para aumentar la eficacia del mensaje. Los datos de referencia contienen metadatos del dispositivo que no cambian con cada evento, pero proporcionan propiedades útiles para el análisis de datos. Los valores de dimensión común de procesamiento por lotes y el uso de datos de referencia ahorran bytes enviados a través de la conexión, lo que hace que el mensaje sea más eficaz.

Considere la siguiente carga de JSON que se envía al entorno de disponibilidad general de Azure Time Series Insights mediante un [objeto de mensaje de dispositivo IoT](https://docs.microsoft.com/dotnet/api/microsoft.azure.devices.message?view=azure-dotnet) que se serializa en JSON cuando se envía a la nube de Azure:

```JSON
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

- Tabla de datos de referencia que tiene la propiedad de clave **deviceId**:

   | deviceId | messageId | deviceLocation |
   | --- | --- | --- |
   | FXXX | LINE\_DATA | EU |
   | FYYY | LINE\_DATA | US |

- Tabla de eventos de Azure Time Series Insights después del acoplamiento:

   | deviceId | messageId | deviceLocation | timestamp | series.Flow Rate ft3/s | series.Engine Oil Pressure psi |
   | --- | --- | --- | --- | --- | --- |
   | FXXX | LINE\_DATA | EU | 2018-01-17T01:17:00Z | 1.0172575712203979 | 34.7 |
   | FXXX | LINE\_DATA | EU | 2018-01-17T01:17:00Z | 2.445906400680542 | 49.2 |
   | FYYY | LINE\_DATA | US | 2018-01-17T01:18:00Z | 0.58015072345733643 | 22.2 |

> [!NOTE]

> - La columna **deviceId** actúa como el encabezado de columna para los distintos dispositivos en una línea. Al establecerse el valor de **deviceId** como su propio nombre de propiedad, se limita el número total de dispositivos a 595 (para entornos S1) o 795 (para entornos S2) con las otras cinco columnas.
> - Se evitan las propiedades innecesarias (por ejemplo, la información de marca y modelo). Puesto que las propiedades no se consultarán en el futuro, al eliminarlas se permite que la eficacia de red y almacenamiento sea mejor.
> - Los datos de referencia se utilizan para reducir el número de bytes transferidos a través de la red. Se combinan los dos atributos, **messageId** y **deviceLocation**, mediante la propiedad de clave **deviceId**. Estos datos se combinan con los de telemetría en el momento de entrada y posteriormente se almacenan en Azure Time Series Insights para las consultas.
> - Se utilizan dos niveles de anidamiento, que es la cantidad máxima de anidamiento que Azure Time Series Insights admite. Es fundamental para evitar matrices profundamente anidadas.
> - Las medidas se envían como propiedades independientes en el mismo objeto al haber pocas medidas. En este caso, **series.Flow Rate psi** y **series.Engine Oil Pressure ft3/s** son columnas únicas.

## <a name="scenario-two-several-measures-exist"></a>Escenario dos: existen varias medidas

> [!TIP]
> Recomendamos que envíe las mediciones como tuplas de "tipo", "unidad" y "valor".

Carga JSON de ejemplo:

```JSON
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

- Tabla de datos de referencia con las propiedades de clave **deviceId** y **series.tagId**:

   | deviceId | series.tagId | messageId | deviceLocation | type | unit |
   | --- | --- | --- | --- | --- | --- |
   | FXXX | pumpRate | LINE\_DATA | EU | Caudal | ft3/s |
   | FXXX | oilPressure | LINE\_DATA | EU | Presión de aceite del motor | psi |
   | FYYY | pumpRate | LINE\_DATA | US | Caudal | ft3/s |
   | FYYY | oilPressure | LINE\_DATA | US | Presión de aceite del motor | psi |

- Tabla de eventos de Azure Time Series Insights después del acoplamiento:

   | deviceId | series.tagId | messageId | deviceLocation | type | unit | timestamp | series.value |
   | --- | --- | --- | --- | --- | --- | --- | --- |
   | FXXX | pumpRate | LINE\_DATA | EU | Caudal | ft3/s | 2018-01-17T01:17:00Z | 1.0172575712203979 |
   | FXXX | oilPressure | LINE\_DATA | EU | Presión de aceite del motor | psi | 2018-01-17T01:17:00Z | 34.7 |
   | FXXX | pumpRate | LINE\_DATA | EU | Caudal | ft3/s | 2018-01-17T01:17:00Z | 2.445906400680542 |
   | FXXX | oilPressure | LINE\_DATA | EU | Presión de aceite del motor | psi | 2018-01-17T01:17:00Z | 49.2 |
   | FYYY | pumpRate | LINE\_DATA | US | Caudal | ft3/s | 2018-01-17T01:18:00Z | 0.58015072345733643 |
   | FYYY | oilPressure | LINE\_DATA | US | Presión de aceite del motor | psi | 2018-01-17T01:18:00Z | 22.2 |

> [!NOTE]

> - Las columnas **deviceId** y **series.tagId** actúan como encabezados de columna para los distintos dispositivos y etiquetas en una línea. Al usar cada una como su propio atributo, se limita la consulta a 594 (para entornos S1) o 794 (para entornos S2) dispositivos en total, con las otras seis columnas.
> - Se han evitado las propiedades innecesarias, por la razón indicada en el primer ejemplo.
> - Los datos de referencia se utilizan para reducir el número de bytes transferidos a través de la red mediante la introducción de **deviceId**, que se usa para el par único de **messageId** y **deviceLocation**. Se utiliza la clave compuesta **series.tagId** para el par único de **type** y **unit**. La clave compuesta permite usar el par **deviceId** y **series.tagId** para hacer referencia a cuatro valores: **messageId, deviceLocation, tipo** y **unidad**. Estos datos se unen con los datos de telemetría en el momento de la entrada. Luego, se almacenan en Azure Time Series Insights para realizar consultas.
> - Se utilizan dos niveles de anidamiento, por la razón indicada en el primer ejemplo.

### <a name="for-both-scenarios"></a>Para ambos escenarios

Para una propiedad con un gran número de valores posibles, es mejor enviarlos como valores distintos dentro de una sola columna en lugar de crear una nueva columna para cada valor. De los dos ejemplos anteriores:

- En el primer ejemplo, algunas propiedades tienen varios valores, por lo que es adecuado hacer que cada uno sea una propiedad independiente.
- En el segundo ejemplo, las medidas no se especifican como propiedades individuales. En su lugar, son una matriz de valores o medidas en una propiedad de serie común. Se envía la nueva clave **tagId**, que crea la nueva columna **series.tagId** en la tabla acoplada. Las nuevas propiedades **type** y **unit** se crean mediante datos de referencia para que no se alcance el límite de propiedades.

## <a name="next-steps"></a>Pasos siguientes

- Obtenga más información sobre el envío de [mensajes de dispositivo de IoT Hub a la nube](../iot-hub/iot-hub-devguide-messages-construct.md).

- Lea el artículo sobre la [sintaxis de las consultas de Azure Time Series Insights](https://docs.microsoft.com/rest/api/time-series-insights/gen1-query-syntax) para más información sobre la sintaxis de las consultas en la API REST de acceso a datos de Azure Time Series Insights.

- Aprenda a [dar forma a eventos](./time-series-insights-send-events.md).
