---
title: Creación de un modelo de dispositivo simulado avanzado en Azure | Microsoft Docs
description: En esta guía de procedimientos, aprenderá a crear un modelo de dispositivo avanzado para usarlo con el acelerador de soluciones Simulación de dispositivo.
author: troyhopwood
manager: timlt
ms.service: iot-accelerators
services: iot-accelerators
ms.topic: conceptual
ms.date: 03/18/2019
ms.author: troyhop
ms.custom:
- mvc
- amqp
- mqtt
- devx-track-javascript
ms.openlocfilehash: b424ea01d8283963c49d899a97120c9132d862f8
ms.sourcegitcommit: e71da24cc108efc2c194007f976f74dd596ab013
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87422545"
---
# <a name="create-an-advanced-device-model"></a>Creación de un modelo de dispositivo avanzado

Esta guía de procedimientos describe los archivos JSON y JavaScript que definen un modelo de dispositivo personalizado. El artículo incluye algunos archivos de definición de modelos de dispositivo de ejemplo y muestra cómo cargarlos en la instancia de Simulación de dispositivo. Puede crear modelos de dispositivo avanzado para simular comportamientos de dispositivo más realistas para las pruebas.

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="prerequisites"></a>Requisitos previos

Para seguir los pasos de esta guía de procedimientos, necesitará una instancia implementada de Simulación de dispositivo en la suscripción de Azure.

Si aún no la ha implementado, debe completar primero la [Guía de inicio rápido: Implementación y ejecución de una simulación de dispositivo IoT en Azure](quickstart-device-simulation-deploy.md).

### <a name="open-device-simulation"></a>Abrir Simulación de dispositivo

Para ejecutar Simulación de dispositivo en el explorador, primero vaya a [Aceleradores de soluciones de Microsoft Azure IoT](https://www.azureiotsolutions.com).

Se le pedirá que inicie sesión con sus credenciales de suscripción de Azure.

A continuación, haga clic en **Iniciar** en el icono de Simulación de dispositivo que ha implementado en la [Guía de inicio rápido: Implementación y ejecución de una simulación de dispositivo IoT en Azure](quickstart-device-simulation-deploy.md).

## <a name="device-models"></a>Modelos de dispositivos

Cada dispositivo simulado pertenece a un modelo de dispositivo específico que define el comportamiento de simulación. Este comportamiento incluye la frecuencia de envío de telemetría, qué tipo de mensajes enviar y los métodos admitidos.

Un modelo de dispositivo se define con un archivo de definición de dispositivo JSON y un conjunto de archivos JavaScript. Estos archivos JavaScript determinan el comportamiento de simulación, como los datos de telemetría aleatorios y la lógica del método.

Un modelo de dispositivo típico tiene:

* Un archivo JSON para cada modelo de dispositivo (por ejemplo, elevator.json).
* Un archivo de script de comportamiento de JavaScript para cada modelo de dispositivo (por ejemplo, elevator-state.js)
* Un archivo de script de método de JavaScript para cada método de dispositivo (por ejemplo, elevator-go-down.js)

> [!NOTE]
> No todos los modelos de dispositivo definen métodos. Por lo tanto, un modelo de dispositivo puede tener o no scripts de método. Sin embargo, todos los modelos de dispositivo deben tener un script de comportamiento.

## <a name="device-definition-file"></a>Archivo de definición de dispositivo

Cada archivo de definición de dispositivo contiene detalles de un modelo de dispositivo simulado, incluida la información siguiente:

* Nombre del modelo de dispositivo: cadena.
* Protocolo: AMQP | MQTT | HTTP.
* Estado inicial del dispositivo.
* Frecuencia con que se actualiza el estado del dispositivo.
* Qué archivo JavaScript se debe usar para actualizar el estado del dispositivo.
* Una lista de mensajes de telemetría para enviar, cada uno con una frecuencia determinada.
* El esquema de los mensajes de telemetría, usado por la aplicación back-end para analizar la telemetría recibida.
* Una lista de los métodos admitidos y el archivo JavaScript que se utiliza para simular cada uno.

### <a name="file-schema"></a>Esquema de archivos

La versión del esquema es siempre "1.0.0" y es específica del formato de este archivo:

```json
"SchemaVersion": "1.0.0"
```

### <a name="device-model-description"></a>Descripción del modelo de dispositivo

Las siguientes propiedades describen el modelo del dispositivo. Cada tipo tiene un identificador único, una versión semántica, un nombre y una descripción:

```json
"Id": "chiller-01",
"Version": "0.0.1",
"Name": "Chiller",
"Description": "Chiller with external temperature and humidity sensors."
```

### <a name="iot-protocol"></a>Protocolo IoT

Los dispositivos IoT pueden conectarse mediante protocolos diferentes. La simulación le permite usar **AMQP**, **MQTT** o **HTTP**:

```json
"Protocol": "AMQP"
```

### <a name="simulated-device-state"></a>Estado del dispositivo simulado

Cada dispositivo simulado tiene un estado interno, que debe definirse. El estado también define las propiedades que se pueden notificar en los datos de telemetría. Por ejemplo, un refrigerador podría tener un estado inicial como:

```json
"InitialState": {
    "temperature": 50,
    "humidity": 40
},
```

Un dispositivo móvil con varios sensores podría tener más propiedades, por ejemplo:

```json
"InitialState": {
    "latitude": 47.445301,
    "longitude": -122.296307,
    "speed": 30.0,
    "speed_unit": "mph",
    "cargotemperature": 38.0,
    "cargotemperature_unit": "F"
}
```

El estado del dispositivo se mantiene en memoria mediante el servicio de simulación y se proporciona como entrada a la función JavaScript. La función JavaScript podría decidir:

* Omitir el estado y generar algunos datos aleatorios.
* Actualizar el estado del dispositivo de alguna manera realista para un escenario determinado.

La función que genera el estado también recibe como entrada:

* El identificador del dispositivo.
* El modelo del dispositivo.
* La hora actual. Este valor hace posible generar datos diferentes según el dispositivo y la hora.

### <a name="generating-telemetry-messages"></a>Generación de mensajes de telemetría

El servicio de simulación puede enviar varios tipos de datos de telemetría para cada dispositivo. Normalmente, la telemetría incluye datos del estado del dispositivo. Por ejemplo, una sala simulada puede enviar información acerca de la temperatura y la humedad cada diez segundos. Tenga en cuenta los marcadores de posición en el siguiente fragmento de código, que se reemplazan automáticamente con los valores de estado del dispositivo:

```json
"Telemetry": [
    {
        "Interval": "00:00:10",
        "MessageTemplate":
            "{\"temperature\":${temperature},\"temperature_unit\":\"${temperature_unit}\",\"humidity\":\"${humidity}\"}",
        "MessageSchema": {
            "Name": "RoomComfort;v1",
            "Format": "JSON",
            "Fields": {
                "temperature": "double",
                "temperature_unit": "text",
                "humidity": "integer"
            }
        }
    }
],
```

Los marcadores de posición utilizan una sintaxis especial **${NAME}** donde **NAME** es una clave del objeto de estado de dispositivo devuelto por la función JavaScript **principal**. Las cadenas deben ir entrecomilladas, pero no los números.

#### <a name="message-schema"></a>Esquema de los mensajes

Cada tipo de mensaje debe tener un esquema bien definido. El esquema del mensaje también se publica en IoT Hub de modo que las aplicaciones back-end puedan reutilizar la información para interpretar la telemetría entrante.

El esquema es compatible con el formato JSON, lo que facilita el análisis y la transformación entre varios sistemas y servicios.

Los campos enumerados en el esquema pueden ser de los siguientes tipos:

* Objeto: serializado mediante JSON
* Binario: serializado mediante base64
* Texto
* Boolean
* Entero
* Double
* DateTime

### <a name="supported-methods"></a>Métodos admitidos

Los dispositivos simulados también pueden reaccionar a las llamadas a los métodos, en cuyo caso ejecutan alguna lógica y proporcionan una respuesta determinada. De forma similar a la simulación, la lógica del método se almacena en un archivo JavaScript y puede interactuar con el estado del dispositivo. Por ejemplo:

```json
"CloudToDeviceMethods": {
    "Start": {
        "Type": "JavaScript",
        "Path": "release-pressure.js"
    }
}
```

## <a name="create-a-device-definition-file"></a>Creación de un archivo de definición de dispositivo

En esta guía de procedimientos, verá cómo crear un modelo de dispositivo para un dron. El dron volará aleatoriamente en un conjunto inicial de coordenadas en las que se cambia la ubicación y la altitud.

Copie el siguiente JSON en un editor de texto y guárdelo como **drone.json**.

### <a name="device-definition-json-example"></a>Ejemplo de JSON de definición de dispositivo

```json
{
  "SchemaVersion": "1.0.0",
  "Id": "drone",
  "Version": "0.0.1",
  "Name": "Drone",
  "Description": "Simple drone.",
  "Protocol": "AMQP",
  "Simulation": {
    "InitialState": {
      "velocity": 0.0,
      "velocity_unit": "mm/sec",
      "acceleration": 0.0,
      "acceleration_unit": "mm/sec^2",
      "latitude": 47.476075,
      "longitude": -122.192026,
      "altitude": 0.0
    },
    "Interval": "00:00:05",
    "Scripts": [{
      "Type": "JavaScript",
      "Path": "drone-state.js"
    }]
  },
  "Properties": {
    "Type": "Drone",
    "Firmware": "1.0",
    "Model": "P-96"
  },
  "Tags": {
    "Owner": "Contoso"
  },
  "Telemetry": [{
      "Interval": "00:00:05",
      "MessageTemplate": "{\"velocity\":\"${velocity}\",\"acceleration\":\"${acceleration}\",\"position\":\"${latitude}|${longitude}|${altitude}\"}",
      "MessageSchema": {
        "Name": "drone-event-sensor;v1",
        "Format": "JSON",
        "Fields": {
          "velocity": "double",
          "velocity_unit": "text",
          "acceleration": "double",
          "acceleration_unit": "text",
          "latitude": "double",
          "longitude": "double",
          "altitude": "double"
        }
      }
    }
  ],
    "CloudToDeviceMethods": {
        "RecallDrone": {
            "Type": "JavaScript",
            "Path": "droneRecall-method.js"
        }
    }
}
```

## <a name="behavior-script-files"></a>Archivos de script de comportamiento

El código del archivo de script de comportamiento mueve el dron. Para modificar la elevación y la ubicación del dron, el script manipula el estado de la memoria del dispositivo.

Los archivos JavaScript deben tener una función **principal**, que acepta dos parámetros:

* Un objeto **context** que contiene tres propiedades:
    * **currentTime** como una cadena con el formato **aaaa-MM-dd'T'HH:mm:sszzz**.
    * **deviceId**. Por ejemplo, **Simulated.Elevator.123**.
    * **deviceModel**. Por ejemplo, **Elevator**.
* Un objeto **state** que es el valor devuelto por la función en la llamada anterior. El servicio de simulación mantiene este estado del dispositivo, que se usa para generar mensajes de telemetría.

La función **main** devuelve el nuevo estado del dispositivo. Por ejemplo:

```JavaScript
function main(context, state) {

    // Use context if the simulation depends on
    // time or device details.
    // Execute some logic, updating 'state'

    return state;
}
```

## <a name="create-a-behavior-script-file"></a>Creación de un archivo de script de comportamiento

Copie el siguiente código JavaScript en un editor de texto y guárdelo como **drone-state.json**.

### <a name="device-model-javascript-simulation-example"></a>Ejemplo de simulación de JavaScript de modelo de dispositivo

```JavaScript
"use strict";

// Position control
const DefaultLatitude = 47.476075;
const DefaultLongitude = -122.192026;
const DistanceVariation = 10;

// Altitude control
const DefaultAltitude = 0.0;
const AverageAltitude = 499.99;
const AltitudeVariation = 5;

// Velocity control
const AverageVelocity = 60.00;
const MinVelocity = 20.00;
const MaxVelocity = 120.00;
const VelocityVariation = 5;

// Acceleration control
const AverageAcceleration = 2.50;
const MinAcceleration = 0.01;
const MaxAcceleration = 9.99;
const AccelerationVariation = 1;

// Display control for position and other attributes
const GeoSpatialPrecision = 6;
const DecimalPrecision = 2;

// Default state
var state = {
    velocity: 0.0,
    velocity_unit: "mm/sec",
    acceleration: 0.0,
    acceleration_unit: "mm/sec^2",
    latitude: DefaultLatitude,
    longitude: DefaultLongitude,
    altitude: DefaultAltitude
};

// Default device properties
var properties = {};

/**
 * Restore the global state using data from the previous iteration.
 *
 * @param previousState device state from the previous iteration
 * @param previousProperties device properties from the previous iteration
 */
function restoreSimulation(previousState, previousProperties) {
    // If the previous state is null, force a default state
    if (previousState) {
        state = previousState;
    } else {
        log("Using default state");
    }

    if (previousProperties) {
        properties = previousProperties;
    } else {
        log("Using default properties");
    }
}

/**
 * Simple formula generating a random value around the average
 * in between min and max
 */
function vary(avg, percentage, min, max) {
    var value = avg * (1 + ((percentage / 100) * (2 * Math.random() - 1)));
    value = Math.max(value, min);
    value = Math.min(value, max);
    return value;
}

/**
 * Entry point function called by the simulation engine.
 * Returns updated simulation state.
 * Device property updates must call updateProperties() to persist.
 *
 * @param context             The context contains current time, device model and id
 * @param previousState       The device state since the last iteration
 * @param previousProperties  The device properties since the last iteration
 */
/*jslint unparam: true*/
function main(context, previousState, previousProperties) {

    // Restore the global state before generating the new telemetry, so that
    // the telemetry can apply changes using the previous function state.
    restoreSimulation(previousState, previousProperties);

    state.acceleration = vary(AverageAcceleration, AccelerationVariation, MinAcceleration, MaxAcceleration).toFixed(DecimalPrecision);
    state.velocity = vary(AverageVelocity, VelocityVariation, MinVelocity, MaxVelocity).toFixed(DecimalPrecision);

    // Use the last coordinates to calculate the next set with a given variation
    var coords = varylocation(Number(state.latitude), Number(state.longitude), DistanceVariation);
    state.latitude = Number(coords.latitude).toFixed(GeoSpatialPrecision);
    state.longitude = Number(coords.longitude).toFixed(GeoSpatialPrecision);

    // Fluctuate altitude between given variation constant by more or less
    state.altitude = vary(AverageAltitude, AltitudeVariation, AverageAltitude - AltitudeVariation, AverageAltitude + AltitudeVariation).toFixed(DecimalPrecision);

    return state;
}

/**
 * Generate a random geolocation at some distance (in miles)
 * from a given location
 */
function varylocation(latitude, longitude, distance) {
    // Convert to meters, use Earth radius, convert to radians
    var radians = (distance * 1609.344 / 6378137) * (180 / Math.PI);
    return {
        latitude: latitude + radians,
        longitude: longitude + radians / Math.cos(latitude * Math.PI / 180)
    };
}
```

## <a name="create-a-method-script-file"></a>Creación de un archivo de script de método

Los scripts de método son similares a los de comportamiento. Definen el comportamiento del dispositivo cuando se llama a un método concreto de nube para el dispositivo.

El script de recuperación del dron establece las coordenadas del dron en un punto fijo para simular que vuelve a su lugar de partida.

Copie el siguiente código JavaScript en un editor de texto y guárdelo como **droneRecall-method.json**.

### <a name="device-model-javascript-simulation-example"></a>Ejemplo de simulación de JavaScript de modelo de dispositivo

```JavaScript
"use strict";

// Default state
var state = {
    velocity: 0.0,
    velocity_unit: "mm/sec",
    acceleration: 0.0,
    acceleration_unit: "mm/sec^2",
    latitude: 0.0,
    longitude: 0.0,
    altitude: 0.0
};

// Default device properties
var properties = {};

/**
 * Restore the global state using data from the previous iteration.
 *
 * @param previousState device state from the previous iteration
 * @param previousProperties device properties from the previous iteration
 */
function restoreSimulation(previousState, previousProperties) {
    // If the previous state is null, force a default state
    if (previousState) {
        state = previousState;
    } else {
        log("Using default state");
    }

    if (previousProperties) {
        properties = previousProperties;
    } else {
        log("Using default properties");
    }
}

/**
 * Entry point function called by the simulation engine.
 *
 * @param context        The context contains current time, device model and id, not used
 * @param previousState  The device state since the last iteration, not used
 * @param previousProperties  The device properties since the last iteration
 */
/*jslint unparam: true*/
function main(context, previousState, previousProperties) {

    // Restore the global device properties and the global state before
    // generating the new telemetry, so that the telemetry can apply changes
    // using the previous function state.
    restoreSimulation(previousState, previousProperties);

    //simulate the behavior of a drone when recalled
  state.latitude = 47.476075;
  state.longitude = -122.192026;
  return state;
}
```

## <a name="debugging-script-files"></a>Depuración de archivos de script

Aunque no puede adjuntar un depurador a un archivo de comportamiento en ejecución, es posible escribir información en el registro de servicio con la función **log**. Con los errores de sintaxis, el intérprete no puede seguir y escribe información sobre la excepción en el registro.

Ejemplo de registro:

```JavaScript
function main(context, state) {

    log("This message will appear in the service logs.");

    log(context.deviceId);

    if (typeof(state) !== "undefined" && state !== null) {
        log("Previous value: " + state.temperature);
    }

    // ...

    return updateState;
}
```

## <a name="deploy-an-advanced-device-model"></a>Implementación de un modelo de dispositivo avanzado

Para implementar el modelo de dispositivo avanzado, cargue los archivos de la instancia de Simulación de dispositivo:

Seleccione **Modelos de dispositivo** en la barra de menús. La página **Modelos de dispositivo** muestra todos los modelos de dispositivo disponibles en esta instancia de Simulación de dispositivo:

![Modelos de dispositivos](media/iot-accelerators-device-simulation-advanced-device/devicemodelnav.png)

Haga clic en **+ Agregar modelos de dispositivo** en la esquina superior derecha de la página:

![Incorporación del modelo de dispositivo](media/iot-accelerators-device-simulation-advanced-device/devicemodels.png)

Haga clic en **Opciones avanzadas** para abrir la pestaña del modelo de dispositivo avanzado:

![Pestaña Avanzadas](media/iot-accelerators-device-simulation-advanced-device/advancedtab.png)

Haga clic en **Examinar** y seleccione los archivos JSON y JavaScript que creó. Asegúrese de seleccionar los tres archivos. Si falta algún archivo, se produce un error de validación:

![Examen de los archivos](media/iot-accelerators-device-simulation-advanced-device/browse.png)

Si los archivos pasan la validación, haga clic en **Guardar** y su modelo de dispositivo estará preparado para usarse en una simulación. En caso contrario, corrija los errores y vuelva a cargar los archivos:

![Save](media/iot-accelerators-device-simulation-advanced-device/validated.png)

## <a name="next-steps"></a>Pasos siguientes

En esta guía de procedimientos, aprendió sobre los archivos de modelo de dispositivo que se usan en Simulación de dispositivo y el modo de crear un modelo de dispositivo avanzado. A continuación, puede ser conveniente que explore cómo [usar Time Series Insights para visualizar los datos de telemetría que se envíen desde el acelerador de soluciones Simulación de dispositivo](https://docs.microsoft.com/azure/iot-accelerators/iot-accelerators-device-simulation-time-series-insights).
