---
title: Información de Digital Twins de IoT Plug and Play
description: Información de cómo la versión preliminar de IoT Plug and Play usa Digital Twins
author: prashmo
ms.author: prashmo
ms.date: 07/17/2020
ms.topic: conceptual
ms.service: iot-pnp
services: iot-pnp
ms.openlocfilehash: 1908abfb3d0ea20c69a68344d54076c6760e9e63
ms.sourcegitcommit: 46f8457ccb224eb000799ec81ed5b3ea93a6f06f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87352045"
---
# <a name="understand-iot-plug-and-play-digital-twins"></a>Información de Digital Twins de IoT Plug and Play

Un dispositivo IoT Plug and Play implementa un modelo descrito por el esquema de [Lenguaje de definición de Digital Twins (DTDL)](https://github.com/Azure/opendigitaltwins-dtdl). Un modelo describe el conjunto de componentes, las propiedades, los comandos y los mensajes de telemetría que puede tener un dispositivo determinado. Un dispositivo gemelo y un gemelo digital se inicializan la primera vez que un dispositivo IoT Plug and Play se conecta a un centro de IoT.

IoT Plug and Play usa DTDL versión 2. Para más información sobre esta versión, consulte la especificación [Lenguaje de definición de Digital Twins (DTDL): versión 2](https://github.com/Azure/opendigitaltwins-dtdl/blob/master/DTDL/v2/dtdlv2.md) en GitHub.

DTDL no es exclusivo de IoT Plug and Play. Otros servicios de IoT, como [Azure Digital Twins](../digital-twins/overview.md), lo usan para representar entornos completos, como edificios y redes energéticas. Para más información, consulte [Descripción de los modelos gemelos de Azure Digital Twins](../digital-twins/concepts-models.md).

En este artículo se describe cómo se representan los componentes y las propiedades en las secciones *desired* y *reported* de un dispositivo gemelo. También se describe cómo se asignan estos conceptos al gemelo digital correspondiente.

El dispositivo IoT Plug and Play de este artículo que implementa el [modelo de controlador de temperatura](https://github.com/Azure/opendigitaltwins-dtdl/blob/master/DTDL/v2/samples/TemperatureController.json) con un componente de [termostato](https://github.com/Azure/opendigitaltwins-dtdl/blob/master/DTDL/v2/samples/Thermostat.json).

## <a name="device-twins-and-digital-twins"></a>Dispositivo gemelos y gemelos digitales

Los dispositivos gemelos son documentos JSON que almacenan información acerca del estado del dispositivo, incluidos metadatos, configuraciones y condiciones. Para más información, consulte [Información y uso de dispositivos gemelos en IoT Hub](../iot-hub/iot-hub-devguide-device-twins.md). Los creadores de dispositivos y soluciones pueden seguir usando el mismo conjunto de API y SDK de dispositivo gemelo para implementar dispositivos y soluciones con las convenciones de IoT Plug and Play.

Las API de gemelo digital funcionan en construcciones de nivel superior, en lenguaje de definición de Digital Twins (DTDL), como componentes, propiedades y comandos. Las API de gemelo digital facilitan la creación de soluciones de IoT Plug and Play.

En un dispositivo gemelo, el estado de una propiedad grabable se divide entre las secciones desired y reported. Todas las propiedades de solo lectura están disponibles en la sección reported.

En un gemelo digital, hay una vista unificada del estado actual y deseado de la propiedad. El estado de sincronización de una propiedad determinada se almacena en la sección `$metadata` correspondiente del componente o de nivel de raíz.

### <a name="digital-twin-json-format"></a>Formato JSON de gemelo digital

Si se representa como un objeto JSON, un gemelo digital incluye los campos siguientes:

| Nombre del campo | Descripción |
| --- | --- |
| `$dtId` | Cadena proporcionada por el usuario que representa el id. del gemelo digital. |
| `{propertyName}` | Valor de una propiedad en JSON. |
| `$metadata.$model` | [Opcional] Identificador de la interfaz del modelo que caracteriza al gemelo digital. |
| `$metadata.{propertyName}.desiredValue` | [Solo para propiedades grabables] Valor deseado de la propiedad especificada. |
| `$metadata.{propertyName}.desiredVersion` | [Solo para propiedades grabables] Versión del valor deseado que IoT Hub mantiene.|
| `$metadata.{propertyName}.ackVersion` | [Obligatorio, solo para propiedades grabables] Versión confirmada por el dispositivo que implementa el gemelo digital, debe ser mayor o igual que la versión deseada. |
| `$metadata.{propertyName}.ackCode` | [Obligatorio, solo para propiedades grabables] Código `ack` devuelto por la aplicación del dispositivo que implementa el gemelo digital. |
| `$metadata.{propertyName}.ackDescription` | [Opcional, solo para propiedades grabables] Descripción `ack` devuelta por la aplicación del dispositivo que implementa el gemelo digital. |
| `$metadata.{propertyName}.lastUpdateTime` | IoT Hub conserva la marca de tiempo de la última actualización de la propiedad por parte del dispositivo. Las marcas de tiempo están en formato UTC y se codifican con el formato ISO8601 AAAA-MM-DDTHH:MM:SS.mmmZ. |
| `{componentName}` | Objeto JSON que contiene los valores de propiedad y los metadatos del componente, similar a un objeto raíz. |
| `{componentName}.{propertyName}` | Valor de la propiedad del componente en formato JSON. |
| `{componentName}.$metadata` | Información de metadatos del componente, similar al nivel de raíz `$metadata`. |

#### <a name="device-twin-sample"></a>Ejemplo de dispositivo gemelo

En el fragmento de código siguiente se muestra un dispositivo gemelo IoT Plug and Play con formato de objeto JSON:

```json
{
    "deviceId": "sample-device",
    "modelId": "dtmi:com:example:TemperatureController;1",
    "version": 15,
    "properties": {
        "desired": {
            "thermostat1": {
                "__t": "c",
                "targetTemperature": 21.8
            },
            "$metadata": {...},
            "$version": 4
        },
        "reported": {
            "serialNumber": "alwinexlepaho8329",
            "thermostat1": {
                "maxTempSinceLastReboot": 25.3,
                "__t": "c",
                "targetTemperature": {
                    "value": 21.8,
                    "ac": 200,
                    "ad": "Successfully executed patch",
                }
            },
            "$metadata": {...},
            "$version": 11
        }
    }
}
```

#### <a name="digital-twin-sample"></a>Ejemplo de gemelo digital

En el fragmento de código siguiente se muestra un gemelo digital con formato de objeto JSON:

```json
{
    "$dtId": "sample-device",
    "serialNumber": "alwinexlepaho8329",
    "thermostat1": {
        "maxTempSinceLastReboot": 25.3,
        "targetTemperature": 21.8,
        "$metadata": {
            "targetTemperature": {
                "desiredValue": 21.8,
                "desiredVersion": 4,
                "ackVersion": 4,
                "ackCode": 200,
                "ackDescription": "Successfully executed patch",
                "lastUpdateTime": "2020-07-17T06:11:04.9309159Z"
            },
            "maxTempSinceLastReboot": {
                "lastUpdateTime": "2020-07-17T06:10:31.9609233Z"
            }
        }
    },
    "$metadata": {
        "$model": "dtmi:com:example:TemperatureController;1",
        "serialNumber": {
            "lastUpdateTime": "2020-07-17T06:10:31.9609233Z"
        }
    }
}
```

### <a name="properties"></a>Propiedades

Las propiedades son campos de datos que representan el estado de una entidad (como las propiedades de muchos lenguajes de programación orientados a objetos).

#### <a name="read-only-property"></a>Propiedad de solo lectura

Esquema:

```json
{
    "@type": "Property",
    "name": "serialNumber",
    "displayName": "Serial Number",
    "description": "Serial number of the device.",
    "schema": "string"
}
```

En este ejemplo, `alwinexlepaho8329` es el valor actual de la propiedad de solo lectura `serialNumber` que el dispositivo indica.
En los fragmentos de código siguientes se muestra la representación JSON en paralelo de la propiedad `serialNumber`:

:::row:::
   :::column span="":::
      **Dispositivo gemelo**

```json
"properties": {
    "reported": {
        "serialNumber": "alwinexlepaho8329"
    }
}
```

   :::column-end:::
   :::column span="":::
      **Gemelo digital**

```json
"serialNumber": "alwinexlepaho8329"
```

   :::column-end:::
:::row-end:::

#### <a name="writable-property"></a>Propiedad grabable

Supongamos que el dispositivo también tenía la siguiente propiedad grabable en el nivel raíz:

```json
{
    "@type": "Property",
    "name": "fanSpeed",
    "displayName": "Fan Speed",
    "writable": true,
    "schema": "double"
}
```

:::row:::
   :::column span="":::
      **Dispositivo gemelo**

```json
{
    "properties": {
        "desired": {
            "fanSpeed": 2.0,
        },
        "reported": {
            "fanSpeed": {
                "value": 3.0,
                "ac": 200,
                "av": 1,
                "ad": "Successfully executed patch version 1"
            }
        }
    },
}
```

   :::column-end:::
   :::column span="":::
      **Gemelo digital**

```json
{
    "fanSpeed": 3.0,
    "$metadata": {
        "fanSpeed": {
            "desiredValue": 2.0,
            "desiredVersion": 2,
            "ackVersion": 1,
            "ackCode": 200,
            "ackDescription": "Successfully executed patch version 1",
            "lastUpdateTime": "2020-07-17T06:10:31.9609233Z"
        }
    }
}
```

   :::column-end:::
:::row-end:::

En este ejemplo, `3.0` es el valor actual de la propiedad `fanSpeed` que el dispositivo indica. `2.0` es el valor deseado establecido por la solución. El valor deseado y el estado de sincronización de una propiedad de nivel raíz se establecen dentro del elemento `$metadata` de nivel de raíz para un gemelo digital. Cuando el dispositivo está en línea, puede aplicar esta actualización e informar del valor actualizado.

### <a name="components"></a>Componentes

Los componentes permiten compilar una interfaz del modelo como un ensamblado de otras interfaces.
Considere la interfaz [Thermostat](https://github.com/Azure/opendigitaltwins-dtdl/blob/master/DTDL/v2/samples/Thermostat.json), que está definida como un modelo.
Ahora, esta interfaz se puede incorporar como un componente thermostat1 (y otro componente thermostat2) al definir el [modelo de controlador de temperatura](https://github.com/Azure/opendigitaltwins-dtdl/blob/master/DTDL/v2/samples/TemperatureController.json).

En un dispositivo gemelo, un componente se identifica mediante el marcador `{ "__t": "c"}`. En un gemelo digital, la presencia de `$metadata` marca un componente.

En este ejemplo, `thermostat1` es un componente con dos propiedades:

- `maxTempSinceLastReboot` es una propiedad de solo lectura.
- `targetTemperature` es una propiedad grabable que el dispositivo ha sincronizado correctamente. El valor deseado y el estado de sincronización de estas propiedades se encuentran en el elemento `$metadata` del componente.

En los fragmentos de código siguientes se muestra la representación JSON en paralelo del componente `thermostat1`:

:::row:::
   :::column span="":::
      **Dispositivo gemelo**

```json
"properties": {
    "desired": {
        "thermostat1": {
            "__t": "c",
            "targetTemperature": 21.8
        },
        "$metadata": {
        },
        "$version": 4
    },
    "reported": {
        "thermostat1": {
            "maxTempSinceLastReboot": 25.3,
            "__t": "c",
            "targetTemperature": {
                "value": 21.8,
                "ac": 200,
                "ad": "Successfully executed patch",
                "av": 4
            }
        },
        "$metadata": {
        },
        "$version": 11
    }
}
```

   :::column-end:::
   :::column span="":::
      **Gemelo digital**

```json
"thermostat1": {
    "maxTempSinceLastReboot": 25.3,
    "targetTemperature": 21.8,
    "$metadata": {
        "targetTemperature": {
            "desiredValue": 21.8,
            "desiredVersion": 4,
            "ackVersion": 4,
            "ackCode": 200,
            "ackDescription": "Successfully executed patch",
            "lastUpdateTime": "2020-07-17T06:11:04.9309159Z"
        },
        "maxTempSinceLastReboot": {
            "lastUpdateTime": "2020-07-17T06:10:31.9609233Z"
        }
    }
}
```

   :::column-end:::
:::row-end:::

## <a name="digital-twin-apis"></a>API de gemelo digital

Azure Digital Twins viene equipado con **Get Digital Twin** (Obtener gemelo digital), **Update Digital Twin** (Actualizar gemelo digital), **Invoke Component Command** (Invocar comando de componente) e **Invoke Command** (Invocar comando) para administrar el dispositivo de gemelo digital. Puede usar las [API de REST](https://docs.microsoft.com/rest/api/iothub/service/digitaltwin) directamente o a través de un [SDK de servicio](../iot-pnp/libraries-sdks.md).

## <a name="digital-twin-change-events"></a>Eventos de cambio de gemelo digital

Cuando se habilitan los eventos de cambio de gemelo digital, se desencadena un evento cada vez que cambia el valor actual o el deseado del componente o propiedad. Los eventos de cambio de gemelo digital se generan en formato de [revisión de JSON](http://jsonpatch.com/). Los eventos correspondientes se generan en el formato del dispositivo gemelo si se habilitan los eventos de cambio de gemelo.

Para información sobre cómo habilitar el enrutamiento para eventos de dispositivo digital y de gemelo digital, consulte [Uso del enrutamiento de mensajes de IoT Hub para enviar mensajes del dispositivo a la nube a distintos puntos de conexión](../iot-hub/iot-hub-devguide-messages-d2c.md#non-telemetry-events). Para comprender el formato de los mensajes, consulte [Creación y lectura de mensajes de IoT Hub](../iot-hub/iot-hub-devguide-messages-construct.md).

Por ejemplo, se desencadena el evento de cambio de gemelo digital siguiente cuando `targetTemperature` se establece en la solución:

```json
iothub-connection-device-id:sample-device
iothub-enqueuedtime:7/17/2020 6:11:04 AM
iothub-message-source:digitalTwinChangeEvents
correlation-id:275d463fa034
content-type:application/json-patch+json
content-encoding:utf-8
[
  {
    "op": "add",
    "path": "/thermostat1/$metadata/targetTemperature",
    "value": {
      "desiredValue": 21.8,
      "desiredVersion": 4
    }
  }
]
```

Se desencadena el evento de cambio de gemelo digital siguiente cuando el dispositivo informa de que se ha aplicado el cambio deseado anterior:

```json
iothub-connection-device-id:sample-device
iothub-enqueuedtime:7/17/2020 6:11:05 AM
iothub-message-source:digitalTwinChangeEvents
correlation-id:275d464a2c80
content-type:application/json-patch+json
content-encoding:utf-8
[
  {
    "op": "add",
    "path": "/thermostat1/$metadata/targetTemperature/ackCode",
    "value": 200
  },
  {
    "op": "add",
    "path": "/thermostat1/$metadata/targetTemperature/ackDescription",
    "value": "Successfully executed patch"
  },
  {
    "op": "add",
    "path": "/thermostat1/$metadata/targetTemperature/ackVersion",
    "value": 4
  },
  {
    "op": "add",
    "path": "/thermostat1/$metadata/targetTemperature/lastUpdateTime",
    "value": "2020-07-17T06:11:04.9309159Z"
  },
  {
    "op": "add",
    "path": "/thermostat1/targetTemperature",
    "value": 21.8
  }
]
```

## <a name="next-steps"></a>Pasos siguientes

Ahora que ha aprendido acerca de los gemelos digitales, estos son algunos recursos adicionales:

- [Uso de las API de gemelo digital de IoT Plug and Play](howto-manage-digital-twin.md)
- [Interacción con un dispositivo desde su solución](quickstart-service-node.md)
- [API de REST de gemelo digital de IoT](https://docs.microsoft.com/rest/api/iothub/service/digitaltwin)
- [Azure IoT Explorer](howto-use-iot-explorer.md)
