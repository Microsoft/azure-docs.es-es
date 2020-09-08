---
title: Soporte técnico para el tipo de datos Long en Azure Time Series Insights Gen2 | Microsoft Docs
description: Soporte técnico para el tipo de datos Long en Azure Time Series Insights Gen2.
ms.service: time-series-insights
services: time-series-insights
author: deepakpalled
ms.author: dpalled
manager: diviso
ms.workload: big-data
ms.topic: conceptual
ms.date: 08/25/2020
ms.custom: dpalled
ms.openlocfilehash: 8d9b166d4ac9290a0389eac23f9d6882f667e1a0
ms.sourcegitcommit: b33c9ad17598d7e4d66fe11d511daa78b4b8b330
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88856103"
---
# <a name="adding-support-for-long-data-type-in-azure-time-series-insights-gen2"></a>Adición de soporte técnico para el tipo de datos Long en Azure Time Series Insights Gen2

La adición del soporte técnico para el tipo de datos Long afecta al modo en que se almacenan e indexan los datos numéricos solo en entornos de Azure Time Series Insights Gen2. Si tiene un entorno de tipo Gen1, puede omitir estos cambios.

A partir del 29 de junio o del 30 de junio de 2020, en función de su región, los datos se indexarán como **Long** y **Double**.  Si tiene alguna pregunta o duda sobre este cambio, envíe una incidencia de soporte técnico a través de Azure Portal y mencione esta publicación.

Si se ve afectado por cualquiera de los siguientes casos, realice los cambios recomendados:

- **Caso 1**: actualmente usa variables del modelo de Time Series y envía solo tipos de datos enteros en los datos de telemetría.
- **Caso 2**: actualmente usa variables del modelo de Time Series y envía tipos de datos enteros y no enteros en los datos de telemetría.
- **Caso 3**: usa variables de categorías para asignar valores enteros a las categorías.
- **Caso 4**: usa el SDK de JavaScript para crear una aplicación de front-end personalizada.
- **Caso 5**: está llegando al límite del nombre de la propiedad 1000 en el almacenamiento intermedio y envía datos enteros y no enteros. El recuento de propiedades se puede ver como una métrica en [Azure Portal](https://portal.azure.com/).

Si alguno de estos casos se aplica a su situación, realice cambios en el modelo. Actualice la expresión de Time Series (TSX) en la definición de la variable con los cambios recomendados. Actualice ambos:

- Explorador de Azure Time Series Insights TSI
- Cualquier cliente personalizado que use nuestras API

En función de las restricciones y la solución de IoT, es posible que no pueda ver los datos que se envían a su entorno de Azure Time Series Insights Gen2. Si no está seguro de si los datos son solo enteros, o enteros y no enteros, tiene algunas opciones:

- Puede esperar a que se libere la característica. A continuación, explore los eventos sin procesar en la interfaz de usuario del explorador para comprender qué propiedades se guardan en dos columnas independientes.
- Puede realizar de forma preferente los cambios recomendados para todas las etiquetas numéricas.
- Puede enrutar temporalmente un subconjunto de eventos en el almacenamiento para comprender mejor el esquema y poder explorarlo.

Para almacenar los eventos, active la opción de [captura de eventos](https://docs.microsoft.com/azure/event-hubs/event-hubs-capture-overview) de Azure Event Hubs o [enrútelos](https://docs.microsoft.com/azure/iot-hub/iot-hub-devguide-messages-d2c#azure-storage) desde la instancia de IoT Hub a Azure Blob Storage.

Los datos también se pueden observar a través del [explorador del centro de eventos](https://marketplace.visualstudio.com/items?itemName=Summer.azure-event-hub-explorer) o mediante el [host del procesador de eventos](https://docs.microsoft.com/azure/event-hubs/event-hubs-dotnet-standard-getstarted-send#receive-events).

Si usa IoT Hub, vaya a [Leer los mensajes del dispositivo a la nube desde el punto de conexión integrado](https://docs.microsoft.com/azure/iot-hub/iot-hub-devguide-messages-read-builtin) para saber cómo obtener acceso al punto de conexión integrado.

> [!NOTE]
> Es posible que se produzca una interrupción si no realiza los cambios recomendados. Por ejemplo, las variables de Time Series Insights afectadas a las que se tiene acceso a través de las API de consulta o el explorador de Time Series Insights devolverán **NULL** (es decir, no se muestra ningún dato en el explorador).

## <a name="recommended-changes"></a>Cambios recomendados

### <a name="case-1-using-time-series-model-variables-and-sending-only-integral-data-types-in-telemetry-data"></a>Caso 1: usar variables de modelo de Time Series y enviar solo tipos de datos enteros en datos de telemetría

Los cambios recomendados para el caso 1 son los mismos que para el caso 2. Siga las instrucciones de la sección correspondiente al caso 2.

### <a name="case-2-using-time-series-model-variables-and-sending-both-integral-and-nonintegral-types-in-telemetry-data"></a>Caso 2: usar variables de modelo de Time Series y enviar tipos de datos enteros y no enteros en datos de telemetría

Si actualmente envía datos de telemetría de enteros, los datos se dividirán en dos columnas:

- **propertyValue_double**
- **propertyValue_long**

Los datos enteros se escriben en **propertyValue_long**. Los datos numéricos incorporados previamente (y los que se incorporarán en el futuro) en **propertyValue_double** no se copian.

Si quiere consultar los datos de estas dos columnas para la propiedad **propertyValue**, tendrá que usar la función escalar **coalesce()** en TSX. La función acepta argumentos del mismo **DataType** y devuelve el primer valor distinto de NULL en la lista de argumentos. Para obtener más información, consulte [Conceptos de acceso a los datos de Azure Time Series Insights Gen2](https://docs.microsoft.com/rest/api/time-series-insights/reference-time-series-expression-syntax#other-functions).

#### <a name="variable-definition-in-tsx---numeric"></a>Definición de variable en TSX: numérica

*Definición de variable anterior:*

[![Definición de variable anterior](media/time-series-insights-long-data-type/var-def-previous.png)](media/time-series-insights-long-data-type/var-def-previous.png#lightbox)

*Nueva definición de variable:*

[![Nueva definición de variable](media/time-series-insights-long-data-type/var-def.png)](media/time-series-insights-long-data-type/var-def.png#lightbox)

También puede usar **coalesce($event.propertyValue.Double, toDouble($event.propertyValue.Long))** como [expresión de Time Series](https://docs.microsoft.com/rest/api/time-series-insights/reference-time-series-expression-syntax) personalizada.

#### <a name="inline-variable-definition-using-tsx-query-apis---numeric"></a>Definición de variables insertadas mediante las API de consulta de TSX: numérica

*Definición de variable anterior:*

```JSON
"PropertyValueVariable": {

    "kind": "numeric",

    "value": {

        "tsx": "$event.propertyValue.Double"

    },

    "filter": null,

    "aggregation": {

        "tsx": "avg($value)"
    }
}
```

*Nueva definición de variable:*

```JSON
"PropertyValueVariable ": {

    "kind": "numeric",

    "value": {

        "tsx": "coalesce($event.propertyValue.Long, toLong($event.propertyValue.Double))"

    },

    "filter": null,

    "aggregation": {

        "tsx": "avg($value)"
    }
}
```

También puede usar **coalesce($event.propertyValue.Double, toDouble($event.propertyValue.Long))** como [expresión de Time Series](https://docs.microsoft.com/rest/api/time-series-insights/reference-time-series-expression-syntax) personalizada.

> [!NOTE]
> Se recomienda actualizar estas variables en todos los lugares en los que se puedan usar. Estos lugares incluyen el modelo de Time Series, las consultas guardadas y las consultas del conector de Power BI.

### <a name="case-3-using-categorical-variables-to-map-integer-values-to-categories"></a>Caso 3: uso de variables de categorías para asignar valores enteros a las categorías

Si actualmente usa variables de categorías que asignan valores enteros a las categorías, probablemente está usando la función **toLong** para convertir los datos de tipo **Double** al tipo **Long**. Al igual que en los casos 1 y 2, deberá fusionar las columnas **Double**, **Long** y **DataType** .

#### <a name="variable-definition-in-time-series-explorer---categorical"></a>Definición de variables en el explorador de Time Series: categorías

*Definición de variable anterior:*

[![Definición de variable anterior](media/time-series-insights-long-data-type/var-def-cat-previous.png)](media/time-series-insights-long-data-type/var-def-cat-previous.png#lightbox)

*Nueva definición de variable:*

[![Nueva definición de variable](media/time-series-insights-long-data-type/var-def-cat.png)](media/time-series-insights-long-data-type/var-def-cat.png#lightbox)

También puede usar **coalesce($event.propertyValue.Double, toDouble($event.propertyValue.Long))** como [expresión de Time Series](https://docs.microsoft.com/rest/api/time-series-insights/preview#time-series-expression-and-syntax) personalizada.

Las variables de categoría aún requieren que el valor sea de tipo entero. El valor **DataType** de todos los argumentos en la función **coalesce()** debe ser de tipo **Long** en la [expresión de Time Series](https://docs.microsoft.com/rest/api/time-series-insights/reference-time-series-expression-syntax) personalizada.

#### <a name="inline-variable-definition-using-tsx-query-apis---categorical"></a>Definición de variables insertadas mediante las API de consulta de TSX: categorías

*Definición de variable anterior:*

```JSON
"PropertyValueVariable_Long": {

    "kind": "categorical",

    "value": {

        "tsx": "tolong($event.propertyValue.Double)"

    },

    "categories": [

    {
        "label": "Good",

        "values": [0, 1, 2 ]

    },

    {

        "label": "Bad",

        "values": [ 3, 4 ]

    } ],

    "defaultCategory": {

        "label": "Unknown"

    }
}
```

*Nueva definición de variable:*

```JSON
"PropertyValueVariable_Long": {

    "kind": "categorical",

    "value": {

        "tsx": "coalesce($event.propertyValue.Long, tolong($event.propertyValue.Double))"

    },

    "categories": [

    {
        "label": "Good",

        "values": [0, 1, 2 ]

    },

    {

        "label": "Bad",

        "values": [ 3, 4 ]

    } ],

    "defaultCategory": {

        "label": "Unknown"

    }
}
```

Las variables de categoría aún requieren que el valor sea de tipo entero. El valor **DataType** de todos los argumentos en la función **coalesce()** debe ser de tipo **Long** en la [expresión de Time Series](https://docs.microsoft.com/rest/api/time-series-insights/reference-time-series-expression-syntax) personalizada.

> [!NOTE]
> Se recomienda actualizar estas variables en todos los lugares en los que se puedan usar. Estos lugares incluyen el modelo de Time Series, las consultas guardadas y las consultas del conector de Power BI.

### <a name="case-4-using-the-javascript-sdk-to-build-a-custom-front-end-application"></a>Caso 4: uso del SDK de JavaScript para crear una aplicación de front-end personalizada

Si se ve afectado por los casos 1 a 3 y crea aplicaciones personalizadas, debe actualizar las consultas para usar la función **coalesce()** , tal como se muestra en los ejemplos anteriores.

### <a name="case-5-nearing-warm-store-1000-property-limit"></a>Caso 5: cercanía al límite de 1000 propiedades en el almacenamiento intermedio

Si es un usuario del almacenamiento intermedio, tiene un gran número de propiedades y cree que este cambio hará que su entorno supere el límite de 1000 nombres de propiedad del almacenamiento intermedio, envíe una incidencia de soporte técnico a través de Azure Portal y mencione esta publicación.

## <a name="next-steps"></a>Pasos siguientes

- Ver la lista completa de los [tipos de datos admitidos](concepts-supported-data-types.md).
