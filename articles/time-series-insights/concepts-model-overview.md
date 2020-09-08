---
title: 'Modelo de serie temporal: Azure Time Series Insights Gen2 | Microsoft Docs'
description: Información sobre el modelo de serie temporal en Azure Time Series Insights Gen2.
author: deepakpalled
ms.author: dpalled
manager: diviso
ms.workload: big-data
ms.service: time-series-insights
services: time-series-insights
ms.topic: conceptual
ms.date: 08/25/2020
ms.custom: seodec18
ms.openlocfilehash: 18212bf92304e75c702c51ff12628cd670755bb0
ms.sourcegitcommit: b33c9ad17598d7e4d66fe11d511daa78b4b8b330
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88855202"
---
# <a name="time-series-model-in-azure-time-series-insights-gen2"></a>Modelo de serie temporal en Azure Time Series Insights Gen2

En este artículo se describen el modelo de serie temporal, las funciones y cómo empezar a crear y actualizar modelos propios en el entorno de Azure Time Series Insights Gen2.

> [!TIP]
>
> * Vaya al entorno de la  [demostración Contoso Wind Farm](https://insights.timeseries.azure.com/preview/samples) para obtener un ejemplo activo de modelo de serie temporal.
> * Aprenda a [trabajar con un modelo de serie temporal](/azure/time-series-insights/how-to-edit-your-model) mediante el explorador de Azure Time Series Insights TSI.

## <a name="summary"></a>Resumen

Tradicionalmente, los datos que se recopilan de los dispositivos de IoT carecen de información contextual, lo que dificulta encontrar y analizar los sensores rápidamente. El principal objetivo del modelo de serie temporal es simplificar la búsqueda y el análisis de datos de IoT o serie temporal. Este objetivo se logra al permitir la protección, el mantenimiento y el enriquecimiento de datos de serie temporal para ayudarle a preparar conjuntos de datos listos para el consumidor para el análisis.

## <a name="scenario-contosos-new-smart-oven"></a>Escenario: nuevo horno inteligente de Contoso

**Imagine el escenario ficticio de un horno inteligente de Contoso.** En este escenario, imagine que cada horno inteligente de Contoso tiene cinco sensores de temperatura, uno para cada uno de los cuatro quemadores superiores y otro para el horno en sí. Hasta hace poco, cada sensor de temperatura de Contoso enviaba, almacenaba y visualizaba sus datos individualmente. Para la supervisión de su dispositivo de cocina, Contoso se basaba en gráficos básicos, uno para cada sensor individual.

Aunque Contoso estaba satisfecho con su solución inicial de datos y visualización, se detectaron varias limitaciones:

* Los clientes querían saber qué temperatura alcanzaría el horno con la mayoría de los quemadores superiores encendidos. Contoso tenía más dificultades para analizar y presentar una respuesta unificada sobre las condiciones del horno.
* Los ingenieros de Contoso querían comprobar que el funcionamiento simultáneo de los quemadores superiores no se traduciría en una potencia ineficaz. Había dificultades a la hora de establecer las referencias cruzadas de qué sensores de temperatura y voltaje estaban asociados entre sí y cómo localizarlos en el almacén.
* El equipo de control de calidad de Contoso quería auditar y comparar el historial entre dos versiones de sensor. Había dificultad para determinar qué datos pertenecían a cada versión del sensor.

Sin la capacidad de estructurar, organizar y definir el modelo de serie temporal del horno inteligente, cada sensor de temperatura mantenía puntos de datos deslocalizados, aislados y menos informativos. La conversión de estos puntos de datos en información procesable era más difícil, ya que cada conjunto de datos era independiente de los demás.

Estas limitaciones revelaron la importancia de las herramientas de visualización y agregación de datos inteligentes como acompañamiento del nuevo horno de Contoso:

* La visualización de datos resulta útil cuando se pueden asociar y combinar datos en una vista práctica. Un ejemplo muestra los sensores de voltaje junto con los sensores de temperatura.
* La administración de datos multidimensionales de varias entidades junto con las funcionalidades de comparación, zoom e intervalo de tiempo puede ser difícil de lograr.

**El modelo de serie temporal proporciona una solución práctica** para muchos de los escenarios de este ejemplo ficticio:

[![Ejemplo de gráfico de hornos inteligentes del modelo de serie temporal](media/v2-update-tsm/time-series-model-smart-oven.png)](media/v2-update-tsm/time-series-model-smart-oven.png#lightbox)

* El modelo de serie temporal desempeña un papel fundamental en las consultas y en la navegación porque contextualiza datos al permitir comparaciones entre intervalos de tiempo y entre tipos de sensor y dispositivo. (**A**)
* Los datos se contextualizan aún más, ya que los datos guardados en el modelo de serie temporal conservan los cálculos de consulta de serie temporal como variables y los reutilizan en el momento de la consulta.
* El modelo de serie temporal organiza y agrega datos para mejorar las funciones de visualización y administración. (**B**)

### <a name="key-capabilities"></a>Principales capacidades

Con el objetivo de simplificar la gestión de la contextualización de serie temporal, el modelo de serie temporal habilita las siguientes capacidades en Azure Time Series Insights Gen2. Esto le permitirá realizar lo siguiente:

* Crear y administrar cálculos o fórmulas mediante el aprovechamiento de funciones escalares, operaciones de agregación, etc.
* Definir relaciones de primario a secundario para permitir la navegación, la búsqueda y la referencia.
* Definir propiedades asociadas a las instancias, definidas como *campos de instancia*, y usarlas para crear jerarquías.

### <a name="components"></a>Componentes

El modelo de serie temporal tiene tres componentes principales:

* [Instancias del modelo de serie temporal](#time-series-model-instances)
* [Jerarquías del modelo de serie temporal](#time-series-model-hierarchies)
* [Tipos de modelo de serie temporal](#time-series-model-types)

Estos componentes se combinan para especificar un modelo de serie temporal y organizar los datos.

[![Gráfico de información general del modelo de serie temporal](media/v2-update-tsm/time-series-model-overview.png)](media/v2-update-tsm/time-series-model-overview.png#lightbox)

Un modelo de serie temporal se puede crear y administrar mediante el [explorador de Azure Time Series Insights TSI](/azure/time-series-insights/concepts-model-overview). La configuración del modelo de serie temporal se puede administrar mediante la [API de configuración del modelo](https://docs.microsoft.com/rest/api/time-series-insights/reference-model-apis).

## <a name="time-series-model-instances"></a>Instancias del modelo de serie temporal

Las *instancias* del modelo de serie temporal son representaciones virtuales de las propias series temporales.

En la mayoría de los casos, las instancias se identifican de forma única mediante **deviceId** o **assetId**, que se guardan como identificadores de serie temporal.

Las instancias tienen información descriptiva asociada denominada *propiedades de instancia*, como un identificador de serie temporal, un tipo, un nombre, una descripción, jerarquías y campos de instancia. Como mínimo, las propiedades de la instancia incluyen información de la jerarquía.

Los *campos de instancia* son una colección de información descriptiva que puede incluir valores para los niveles de jerarquía, así como el fabricante, el operador, etc.

Después de configurar un origen de evento para el entorno de Azure Time Series Insights Gen2, las instancias se detectan y crean automáticamente en un modelo de serie temporal. Las instancias se pueden crear o actualizar mediante el explorador de Azure Time Series Insights TSI con consultas de modelo de serie temporal.

En la [demostración Contoso Wind Farm](https://insights.timeseries.azure.com/preview/samples) se proporcionan varios ejemplos de instancias activos.

[![Ejemplo de instancia del modelo de serie temporal](media/v2-update-tsm/time-series-model-instance.png)](media/v2-update-tsm/time-series-model-instance.png#lightbox)

### <a name="instance-properties"></a>Propiedades de instancia

Las instancias se definen mediante **timeSeriesId**, **typeId**, **name**, **description**, **hierarchyIds** e **instanceFields**. Cada instancia se asigna a solo un *tipo* y a una o más *jerarquías*.

| Propiedad | Descripción |
| --- | ---|
| timeSeriesId | El identificador único de la serie temporal al que está asociado la instancia. En la mayoría de los casos, las instancias se identifican de forma única mediante una propiedad como deviceId o assetId. En algunos casos, se puede usar un identificador compuesto más específico que combine hasta 3 propiedades. |
| typeId | Identificador de cadena único que distingue entre mayúsculas y minúsculas de tipo Modelo de serie temporal al que está asociado la instancia. De forma predeterminada, todas las instancias nuevas detectadas se asocian a un tipo predeterminado.
| name | La propiedad **name** es opcional y distingue mayúsculas de minúsculas. Si **name** no está disponible, el valor predeterminado es **timeSeriesId**. Si se proporciona un nombre, **timeSeriesId** sigue disponible en el [cuadro](time-series-insights-update-explorer.md#4-time-series-well). |
| description | Descripción de texto de la instancia. |
| hierarchyIds | Define a qué jerarquías pertenece la instancia. |
| instanceFields | Las propiedades de una instancia y cualquier dato estático que define una instancia. Definen valores de jerarquía o propiedades no jerárquicas a la vez que admiten la indexación para realizar operaciones de búsqueda. |

> [!NOTE]
> Las jerarquías se crean mediante campos de instancia. Se pueden agregar elementos **instanceFields** adicionales para más definiciones de propiedades de instancia.

Las instancias tienen la siguiente representación JSON:

```JSON
{
  "timeSeriesId": ["PU2"],
  "typeId": "545314a5-7166-4b90-abb9-fd93966fa39b",
  "hierarchyIds": ["95f0a8d1-a3ef-4549-b4b3-f138856b3a12"],
  "description": "Pump #2",
  "instanceFields": {
    "Location": "Redmond",
    "Fleet": "Fleet 5",
    "Unit": "Pump Unit 3",
    "Manufacturer": "Contoso",
    "ScalePres": "0.54",
    "scaleTemp": "0.54"
  }
}
```

> [!TIP]
> Para obtener información sobre la compatibilidad con crear, leer, actualizar y eliminar (CRUD) de Instance API, lea el artículo [Consulta de datos](time-series-insights-update-tsq.md#time-series-model-query-tsm-q-apis) y la [documentación de Instance API REST](https://docs.microsoft.com/rest/api/time-series-insights/reference-model-apis#instances-api).

## <a name="time-series-model-hierarchies"></a>Jerarquías del modelo de serie temporal

Las *jerarquías* del modelo de serie temporal organizan instancias al especificar los nombres de las propiedades y sus relaciones.

En un entorno de Azure Time Series Insights Gen2 dado se pueden configurar varias jerarquías. Una instancia del modelo de serie temporal puede asignarse a una sola o a varias jerarquías (relación varios a varios).

En la [demostración Contoso Wind Farm](https://insights.timeseries.azure.com/preview/samples) se muestra una instancia estándar y una jerarquía de tipos.

[![Ejemplo de jerarquía del modelo de serie temporal](media/v2-update-tsm/time-series-model-hierarchies.png)](media/v2-update-tsm/time-series-model-hierarchies.png#lightbox)

### <a name="hierarchy-definition"></a>Definición de jerarquía

Las jerarquías se definen mediante **id.** , **name** y **source** de jerarquía.

| Propiedad | Descripción |
| ---| ---|
| id | Identificador único de la jerarquía que se usa, por ejemplo, al definir una instancia. |
| name | Cadena que se usa para proporcionar un nombre para la jerarquía. |
| source | Especifica la jerarquía o la ruta de acceso organizativa, que es un orden descendente de primario a secundario de la jerarquía que los usuarios quieren crear. Las propiedades de primario a secundario asignan campos de instancia. |

Las jerarquías se representan en JSON como:

```JSON
{
  "hierarchies": [
    {
      "id": "6e292e54-9a26-4be1-9034-607d71492707",
      "name": "Location",
      "source": {
        "instanceFieldNames": [
          "state",
          "city"
        ]
      }
    },
    {
      "id": "a28fd14c-6b98-4ab5-9301-3840f142d30e",
      "name": "ManufactureDate",
      "source": {
        "instanceFieldNames": [
          "year",
          "month"
        ]
      }
    }
  ]
}
```

En el ejemplo JSON anterior:

* `Location` define una jerarquía con `states` primario y `cities`secundario. Cada `location` puede tener varios `states` que, a su vez, pueden tener varios `cities`.
* `ManufactureDate` define una jerarquía con `year` primario y `month`secundario. Cada `ManufactureDate` puede tener varios `years` que, a su vez, pueden tener varios `months`.

> [!TIP]
> Para obtener información sobre la compatibilidad con crear, leer, actualizar y eliminar (CRUD) de Hierarchy API, lea el artículo [Consulta de datos](concepts-query-overview.md#time-series-model-query-tsm-q-apis) y la [documentación de Hierarchy API REST](https://docs.microsoft.com/rest/api/time-series-insights/reference-model-apis#hierarchies-api).

### <a name="hierarchy-example"></a>Ejemplo de jerarquía

Imagine un ejemplo en el que la jerarquía **H1** tiene `building`, `floor` y `room` como parte de su definición **instanceFieldNames**:

```JSON
{
  "id": "aaaaaa-bbbbb-ccccc-ddddd-111111",
  "name": "H1",
  "source": {
    "instanceFieldNames": [
      "building",
      "floor",
      "room"
    ]
  }
}
```

Con los campos de instancia usados en la definición anterior y varias series temporales, los atributos y valores de la jerarquía aparecen como se muestra en la tabla siguiente:

| Identificador de serie temporal | Campos de instancia |
| --- | --- |
| ID1 | "building" = "1000", "floor" = "10", "room" = "55"  |
| ID2 | "building" = "1000", "room" = "55" |
| ID3 | "floor" = "10" |
| ID4 | "building" = "1000", "floor" = "10"  |
| ID5 | No hay ningún valor establecido en "building", "floor" o "room". |

La series temporales **ID1** e **ID4** se muestran como parte de la jerarquía **H1** en el [explorador de Azure Time Series Insights TSI](time-series-insights-update-explorer.md) porque tienen parámetros *building*, *floor* y *room* totalmente definidos y correctamente ordenados.

Las demás se clasifican como *instancias no primarias* porque no se ajustan a la jerarquía de datos especificada.

## <a name="time-series-model-types"></a>Tipos de modelo de serie temporal

Los *tipos* de modelo de serie temporal le permiten definir variables o fórmulas para realizar cálculos. Los tipos se asocian con una instancia concreta.

Un tipo puede tener una o más variables. Por ejemplo, una instancia de modelo de serie temporal podría ser de tipo *Sensor de temperatura*, que consta de las variables *temperatura media*, *temperatura mínima* y *temperatura máxima*.

En la [demostración Contoso Wind Farm](https://insights.timeseries.azure.com/preview/samples) se visualizan varios tipos de modelo de serie temporal asociados a sus respectivas instancias.

[![Ejemplo de tipo del modelo de serie temporal](media/v2-update-tsm/time-series-model-types.png)](media/v2-update-tsm/time-series-model-types.png#lightbox)

> [!TIP]
> Para obtener información sobre la compatibilidad con crear, leer, actualizar y eliminar (CRUD) de Types API, lea el artículo [Consulta de datos](concepts-query-overview.md#time-series-model-query-tsm-q-apis) y la [documentación de Type API REST](https://docs.microsoft.com/rest/api/time-series-insights/reference-model-apis#types-api).

### <a name="type-properties"></a>Propiedades de tipo

Los tipos de modelo de serie temporal se definen mediante **id**, **name**, **description** y **variables**.

| Propiedad | Descripción |
| ---| ---|
| id | Identificador de cadena único que distingue entre mayúsculas y minúsculas para el tipo. |
| name | Cadena que se usa para proporcionar un nombre para el tipo. |
| description | Descripción de cadena para el tipo. |
| variables | Especifican variables asociadas al tipo. |

Los tipos se ajustan al ejemplo JSON siguiente:

```JSON
{
  "types": [
    {
      "id": "1be09af9-f089-4d6b-9f0b-48018b5f7393",
      "name": "DefaultType",
      "description": "Default type",
      "variables": {
        "EventCount": {
          "kind": "aggregate",
          "value": null,
          "filter": null,
          "aggregation": {
            "tsx": "count()"
          }
        },
        "Interpolated Speed": {
          "kind": "numeric",
          "value": {
              "tsx": "$event['Speed-Sensor'].Double"
          },
          "filter": null,
          "interpolation": {
              "kind": "step",
              "boundary": {
                  "span": "P1D"
              }
          },
          "aggregation": {
              "tsx": "right($value)"
          }
        }
      }
    }
  ]
}
```

Los tipos del modelo de serie temporal pueden tener muchas variables que especifiquen la fórmula y las reglas de cálculo en los eventos. Más información sobre [cómo definir variables del modelo de serie temporal](./concepts-variables.md)

## <a name="next-steps"></a>Pasos siguientes

* Lea la documentación de referencia del [modelo de la serie temporal](https://docs.microsoft.com/rest/api/time-series-insights/reference-model-apis) para más información sobre cómo editarlo a través de las API.

* Explore las fórmulas y los cálculos que puede crear con las [variables del modelo de serie temporal](./concepts-variables.md)

* Aprenda a [consultar datos](concepts-query-overview.md) en Azure Time Series Insights Gen2
