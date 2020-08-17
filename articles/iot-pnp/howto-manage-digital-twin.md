---
title: Administración de Digital Twins de IoT Plug and Play
description: Administración de dispositivos de la versión preliminar de IoT Plug and Play mediante API de gemelo digital
author: prashmo
ms.author: prashmo
ms.date: 07/20/2020
ms.topic: how-to
ms.service: iot-pnp
services: iot-pnp
ms.openlocfilehash: f86bf17c34d88fa48df4933e979a590fbc89820b
ms.sourcegitcommit: 46f8457ccb224eb000799ec81ed5b3ea93a6f06f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87352026"
---
# <a name="manage-iot-plug-and-play-digital-twins"></a>Administración de Digital Twins de IoT Plug and Play

IoT Plug and Play admite las operaciones **Get digital twin** (Obtener gemelo digital) y **Update digital twin** (Actualizar gemelo digital) para administrar Digital Twins. Puede usar las [API de REST](https://docs.microsoft.com/rest/api/iothub/service/digitaltwin) o uno de los [SDK de servicio](libraries-sdks.md).

En el momento de escribir este documento, la versión de API de gemelo digital para la versión preliminar pública es `2020-05-31-preview`.

## <a name="update-a-digital-twin"></a>Actualización de un gemelo digital

Un dispositivo IoT Plug and Play implementa un modelo descrito por [Lenguaje de definición de Digital Twins v2 (DTDL)](https://github.com/Azure/opendigitaltwins-dtdl). Los desarrolladores de soluciones pueden usar la **API de actualización de gemelo digital** para actualizar el estado del componente y las propiedades de gemelo digital.

El dispositivo IoT Plug and Play, que se usa como ejemplo en este artículo, implementa el [modelo de controlador de temperatura](https://github.com/Azure/opendigitaltwins-dtdl/blob/master/DTDL/v2/samples/TemperatureController.json) con un componentes de [termostato](https://github.com/Azure/opendigitaltwins-dtdl/blob/master/DTDL/v2/samples/Thermostat.json).

En el fragmento de código siguiente se muestra la respuesta a una solicitud **Get digital twin** (Obtener gemelo digital) con formato de objeto JSON. Para obtener más información sobre el formato de gemelo digital, vea [Información de Digital Twins de IoT Plug and Play](./concepts-digital-twin.md#digital-twin-json-format):

```json
{
    "$dtId": "sample-device",
    "serialNumber": "alwinexlepaho8329",
    "thermostat1": {
        "maxTempSinceLastReboot": 25.3,
        "targetTemperature": 20.4,
        "$metadata": {
            "targetTemperature": {
                "desiredValue": 20.4,
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

Digital Twins permite actualizar una propiedad o un componente completo mediante una [revisión de JSON](http://jsonpatch.com/).

Por ejemplo, puede actualizar la propiedad `targetTemperature` como se indica a continuación:

```json
[
    {
        "op": "add",
        "path": "/thermostat1/targetTemperature",
        "value": 21.4
    }
]
```

La actualización anterior establece el valor deseado de una propiedad en el elemento `$metadata` de nivel raíz o nivel de componente correspondiente, tal como se muestra en el siguiente fragmento de código. IoT Hub actualiza la versión deseada de la propiedad:

```json
"thermostat1": {
    "targetTemperature": 20.4,
    "$metadata": {
        "targetTemperature": {
            "desiredValue": 21.4,
            "desiredVersion": 5,
            "ackVersion": 4,
            "ackCode": 200,
            "ackDescription": "Successfully executed patch",
            "lastUpdateTime": "2020-07-17T06:11:04.9309159Z"
        }
    }
}
```

### <a name="add-replace-or-remove-a-component"></a>Adición, reemplazo o eliminación de un componente

Las operaciones de nivel de componente requieren un marcador `$metadata` de objeto vacío dentro del valor.

Una operación de adición o reemplazo de componente establece los valores deseados de todas las propiedades proporcionadas. También borra los valores deseados para las propiedades grabables que no se proporcionan con la actualización.

Al quitar un componente, se borran los valores deseados de todas las propiedades grabables presentes. Con el tiempo, un dispositivo sincroniza esta eliminación y deja de informar de las propiedades individuales. A continuación, el componente se quita del gemelo digital.

En el siguiente ejemplo de revisión de JSON se muestra cómo agregar, reemplazar o quitar un componente:

```json
[
    {
        "op": "add",
        "path": "/thermostat1",
        "value": {
            "targetTemperature": 21.4,
            "anotherWritableProperty": 42,
            "$metadata": {}
        }
    },
    {
        "op": "replace",
        "path": "/thermostat1",
        "value": {
            "targetTemperature": 21.4,
            "$metadata": {}
        }
    },
    {
        "op": "remove",
        "path": "/thermostat2"
    }
]
```

### <a name="add-replace-or-remove-a-property"></a>Adición, reemplazo o eliminación de una propiedad

Una operación de adición o reemplazo establece el valor deseado de una propiedad. El dispositivo puede sincronizar el estado y notificar una actualización del valor junto con un código `ack`, una versión y una descripción.

Al quitar una propiedad, se borra el valor deseado de la propiedad, si se ha establecido. Después, el dispositivo puede dejar de informar de esta propiedad y se quita del nivel raíz o del componente. Si esta propiedad es la última del componente, también se quita el componente.

En el siguiente ejemplo de revisión de JSON se muestra cómo agregar, reemplazar o quitar una propiedad dentro de un componente:

```json
[
    {
        "op": "add",
        "path": "/thermostat1/targetTemperature",
        "value": 21.4
    },
    {
        "op": "replace",
        "path": "/thermostat1/anotherWritableProperty",
        "value": 42
    },
    {
        "op": "remove",
        "path": "/thermostat2/targetTemperature",
    }
]
```

### <a name="rules-for-setting-the-desired-value-of-a-digital-twin-property"></a>Reglas para establecer el valor deseado de una propiedad de gemelo digital

**Nombre**

El nombre de un componente o propiedad debe ser un nombre válido en DTDL v2.

Los caracteres permitidos son a-z, A-Z, 0-9 (no como primer carácter) y guion bajo (no como primer ni último carácter).

Un nombre puede tener entre 1 y 64 caracteres.

**Valor de propiedad**

El valor debe ser una [propiedad de DTDL v2](https://github.com/Azure/opendigitaltwins-dtdl/blob/master/DTDL/v2/dtdlv2.md#property) válida.

Se admiten todos los tipos primitivos. Dentro de los tipos complejos, se admiten las enumeraciones, las asignaciones y los objetos. Para más información, consulte [Esquemas de DTDL v2](https://github.com/Azure/opendigitaltwins-dtdl/blob/master/DTDL/v2/dtdlv2.md#schemas).

Las propiedades no admiten matrices ni esquemas complejos con una matriz.

Se admite una profundidad máxima de cinco niveles para un objeto complejo.

Todos los nombres de campo del objeto complejo deben ser nombres válidos de DTDL v2.

Todas las claves de asignación deben ser nombres válidos de DTDL v2.

## <a name="troubleshoot-update-digital-twin-api-errors"></a>Solución de problemas de errores de API de gemelo digital

Durante la versión preliminar pública, la API de actualización de gemelo digital genera el siguiente mensaje de error genérico:

`ErrorCode:ArgumentInvalid;'{propertyName}' exists within the device twin and is not digital twin conformant property. Please refer to aka.ms/dtpatch to update this to be conformant.`

Asegúrese de que la revisión de actualización sigue las [reglas para establecer el valor deseado de una propiedad de gemelo digital](#rules-for-setting-the-desired-value-of-a-digital-twin-property).

Al actualizar un componente, asegúrese de que el [marcador $metadata de objeto vacío](#add-replace-or-remove-a-component) está establecido.

Se pueden producir errores en las actualizaciones si los valores informados de un dispositivo no se ajustan a las [convenciones de IoT plug and Play](./concepts-convention.md#writable-properties).

## <a name="next-steps"></a>Pasos siguientes

Ahora que ha aprendido acerca de los gemelos digitales, estos son algunos recursos adicionales:

- [Interacción con un dispositivo desde su solución](quickstart-service-node.md)
- [API de REST de gemelo digital de IoT](https://docs.microsoft.com/rest/api/iothub/service/digitaltwin)
- [Azure IoT Explorer](howto-use-iot-explorer.md)
