---
title: Cargas de telemetría, propiedades y comandos en Azure IoT Central | Microsoft Docs
description: Las plantillas de dispositivos de Azure IoT Central permiten especificar la telemetría, las propiedades y los comandos que un dispositivo debe implementar. Conozca el formato de los datos que un dispositivo puede intercambiar con IoT Central.
author: dominicbetts
ms.author: dobett
ms.date: 06/12/2020
ms.topic: conceptual
ms.service: iot-central
services: iot-central
ms.openlocfilehash: 554079ddec3332ced2817d18ea55ce1260d68817
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87291610"
---
# <a name="telemetry-property-and-command-payloads"></a>Cargas de telemetría, propiedades y comandos

_Este artículo se aplica a los desarrolladores de dispositivos._

Una plantilla de dispositivo de Azure IoT Central es un plano técnico que define:

* La telemetría que un dispositivo envía a IoT Central.
* Las propiedades que un dispositivo sincroniza con IoT Central.
* Los comandos a los que IoT Central llama en un dispositivo.

En este artículo, destinado a los desarrolladores de dispositivos, se describen las cargas JSON que los dispositivos envían y reciben en relación con la telemetría, las propiedades y los comandos definidos en una plantilla de dispositivo.

En el artículo no se describen todos los tipos posibles de cargas de telemetría, propiedades y comandos, pero los ejemplos muestran todos los tipos clave.

En cada ejemplo se muestra un fragmento de código del modelo de funcionalidad del dispositivo (DCM) que define el tipo y las cargas JSON de ejemplo para mostrar cómo el dispositivo debe interactuar con la aplicación de IoT Central.

> [!NOTE]
> IoT Central acepta cualquier carga JSON válida, pero solo se puede usar para visualizaciones si coincide con una definición de DCM. Puede exportar datos que no coincidan con una definición; para ello, consulte [Exportación de datos de IoT a destinos en Azure](howto-export-data.md).

El archivo JSON que define el DCM usa el [Lenguaje de definición de gemelos digitales (DTDL) V1](https://github.com/Azure/opendigitaltwins-dtdl/blob/master/DTDL/v1-preview/dtdlv1.md). Esta especificación incluye la definición del formato de la propiedad `@id`.

Para consultar código de dispositivo de ejemplo que muestra algunas de estas cargas en uso, vea los tutoriales [Creación y conexión de un aplicación cliente a una aplicación de Azure IoT Central (Node.js)](tutorial-connect-device-nodejs.md) y [Creación y conexión de un aplicación cliente a una aplicación de Azure IoT Central (Python)](tutorial-connect-device-python.md).

## <a name="view-raw-data"></a>Visualización de datos sin procesar

IoT Central permite ver los datos sin procesar que un dispositivo envía a una aplicación. Esta vista es útil para solucionar problemas con la carga enviada desde un dispositivo. Para ver los datos sin procesar que envía un dispositivo:

1. Vaya al dispositivo en la página **Dispositivos**.

1. Seleccione la pestaña **Datos sin procesar**:

    :::image type="content" source="media/concepts-telemetry-properties-commands/raw-data.png" alt-text="Vista Datos sin procesar":::

    En esta vista, puede seleccionar las columnas que se van a mostrar y establecer el intervalo de tiempo que desea ver. La columna **Datos no modelados** muestra datos del dispositivo que no coinciden con ninguna de las definiciones de telemetría o propiedades de la plantilla de dispositivo.

## <a name="telemetry"></a>Telemetría

### <a name="primitive-types"></a>Tipos primitivos

En esta sección se muestran ejemplos de tipos de telemetría primitivos que un dispositivo transmite a una aplicación de IoT Central.

El siguiente fragmento de código de un DCM muestra la definición de un tipo de telemetría `boolean`:

```json
{
  "@id": "<element id>",
  "@type": "Telemetry",
  "displayName": {
    "en": "BooleanTelemetry"
  },
  "name": "BooleanTelemetry",
  "schema": "boolean"
}
```

Un cliente de dispositivo debe enviar la telemetría como JSON con un aspecto similar al del ejemplo siguiente:

```json
{ "BooleanTelemetry": true }
```

El siguiente fragmento de código de un DCM muestra la definición de un tipo de telemetría `string`:

```json
{
  "@id": "<element id>",
  "@type": "Telemetry",
  "displayName": {
    "en": "StringTelemetry"
  },
  "name": "StringTelemetry",
  "schema": "string"
}
```

Un cliente de dispositivo debe enviar la telemetría como JSON con un aspecto similar al del ejemplo siguiente:

```json
{ "StringTelemetry": "A string value - could be a URL" }
```

El siguiente fragmento de código de un DCM muestra la definición de un tipo de telemetría `integer`:

```json
{
  "@id": "<element id>",
  "@type": "Telemetry",
  "displayName": {
    "en": "IntegerTelemetry"
  },
  "name": "IntegerTelemetry",
  "schema": "integer"
}

```

Un cliente de dispositivo debe enviar la telemetría como JSON con un aspecto similar al del ejemplo siguiente:

```json
{ "IntegerTelemetry": 23 }
```

El siguiente fragmento de código de un DCM muestra la definición de un tipo de telemetría `double`:

```json
{
  "@id": "<element id>",
  "@type": "Telemetry",
  "displayName": {
    "en": "DoubleTelemetry"
  },
  "name": "DoubleTelemetry",
  "schema": "double"
}
```

Un cliente de dispositivo debe enviar la telemetría como JSON con un aspecto similar al del ejemplo siguiente:

```json
{ "DoubleTelemetry": 56.78 }
```

El siguiente fragmento de código de un DCM muestra la definición de un tipo de telemetría `dateTime`:

```json
{
  "@id": "<element id>",
  "@type": "Telemetry",
  "displayName": {
    "en": "DateTimeTelemetry"
  },
  "name": "DateTimeTelemetry",
  "schema": "dateTime"
}
```

Un cliente de dispositivo debe enviar la telemetría como JSON con un aspecto similar al del ejemplo siguiente: los tipos `DateTime` deben cumplir con la norma ISO 8061:

```json
{ "DateTimeTelemetry": "2020-08-30T19:16:13.853Z" }
```

El siguiente fragmento de código de un DCM muestra la definición de un tipo de telemetría `duration`:

```json
{
  "@id": "<element id>",
  "@type": "Telemetry",
  "displayName": {
    "en": "DurationTelemetry"
  },
  "name": "DurationTelemetry",
  "schema": "duration"
}
```

Un cliente de dispositivo debe enviar la telemetría como JSON con un aspecto similar al del ejemplo siguiente: las duraciones deben cumplir con la duración de la norma ISO 8601:

```json
{ "DurationTelemetry": "PT10H24M6.169083011336625S" }
```

### <a name="complex-types"></a>Tipos complejos

En esta sección se muestran ejemplos de tipos de telemetría complejos que un dispositivo transmite a una aplicación de IoT Central.

El siguiente fragmento de código de un DCM muestra la definición de un tipo de telemetría `geopoint`:

```json
{
  "@id": "<element id>",
  "@type": "Telemetry",
  "displayName": {
    "en": "GeopointTelemetry"
  },
  "name": "GeopointTelemetry",
  "schema": "geopoint"
}
```

Un cliente de dispositivo debe enviar la telemetría como JSON con un aspecto similar al del ejemplo siguiente. IoT Central muestra el valor como una chincheta en un mapa:

```json
{
  "GeopointTelemetry": {
    "lat": 47.64263,
    "lon": -122.13035,
    "alt": 0
  }
}
```

El siguiente fragmento de código de un DCM muestra la definición de un tipo de telemetría `Enum`:

```json
{
  "@id": "<element id>",
  "@type": "Telemetry",
  "displayName": {
    "en": "EnumTelemetry"
  },
  "name": "EnumTelemetry",
  "schema": {
    "@id": "<element id>",
    "@type": "Enum",
    "displayName": {
      "en": "Enum"
    },
    "valueSchema": "integer",
    "enumValues": [
      {
        "@id": "<element id>",
        "@type": "EnumValue",
        "displayName": {
          "en": "Item1"
        },
        "enumValue": 0,
        "name": "Item1"
      },
      {
        "@id": "<element id>",
        "@type": "EnumValue",
        "displayName": {
          "en": "Item2"
        },
        "enumValue": 1,
        "name": "Item2"
      },
      {
        "@id": "<element id>",
        "@type": "EnumValue",
        "displayName": {
          "en": "Item3"
        },
        "enumValue": 2,
        "name": "Item3"
      }
    ]
  }
}
```

Un cliente de dispositivo debe enviar la telemetría como JSON con un aspecto similar al del ejemplo siguiente. Los valores posibles son `0`, `1`y `2`, que se muestran en IoT Central como `Item1`, `Item2` y `Item3`:

```json
{ "EnumTelemetry": 1 }
```

El siguiente fragmento de código de un DCM muestra la definición de un tipo de telemetría `Object`. Este objeto tiene tres campos con tipos `dateTime`, `integer` y `Enum`:

```json
{
  "@id": "<element id>",
  "@type": "Telemetry",
  "displayName": {
    "en": "ObjectTelemetry"
  },
  "name": "ObjectTelemetry",
  "schema": {
    "@id": "<element id>",
    "@type": "Object",
    "displayName": {
      "en": "Object"
    },
    "fields": [
      {
        "@id": "<element id>",
        "@type": "SchemaField",
        "displayName": {
          "en": "Property1"
        },
        "name": "Property1",
        "schema": "dateTime"
      },
      {
        "@id": "<element id>",
        "@type": "SchemaField",
        "displayName": {
          "en": "Property2"
        },
        "name": "Property2",
        "schema": "integer"
      },
      {
        "@id": "<element id>",
        "@type": "SchemaField",
        "displayName": {
          "en": "Property3"
        },
        "name": "Property3",
        "schema": {
          "@id": "<element id>",
          "@type": "Enum",
          "displayName": {
            "en": "Enum"
          },
          "valueSchema": "integer",
          "enumValues": [
            {
              "@id": "<element id>",
              "@type": "EnumValue",
              "displayName": {
                "en": "Item1"
              },
              "enumValue": 0,
              "name": "Item1"
            },
            {
              "@id": "<element id>",
              "@type": "EnumValue",
              "displayName": {
                "en": "Item2"
              },
              "enumValue": 1,
              "name": "Item2"
            },
            {
              "@id": "<element id>",
              "@type": "EnumValue",
              "displayName": {
                "en": "Item3"
              },
              "enumValue": 2,
              "name": "Item3"
            }
          ]
        }
      }
    ]
  }
}
```

Un cliente de dispositivo debe enviar la telemetría como JSON con un aspecto similar al del ejemplo siguiente. Los tipos `DateTime` deben cumplir con la norma ISO 8061. Los valores posibles para `Property3` son `0` y `1`, que se muestran en IoT Central como `Item1`, `Item2` y `Item3`:

```json
{
  "ObjectTelemetry": {
      "Property1": "2020-09-09T03:36:46.195Z",
      "Property2": 37,
      "Property3": 2
  }
}
```

El siguiente fragmento de código de un DCM muestra la definición de un tipo de telemetría `vector`:

```json
{
  "@id": "<element id>",
  "@type": "Telemetry",
  "displayName": {
    "en": "VectorTelemetry"
  },
  "name": "VectorTelemetry",
  "schema": "vector"
}
```

Un cliente de dispositivo debe enviar la telemetría como JSON con un aspecto similar al del ejemplo siguiente:

```json
{
  "VectorTelemetry": {
    "x": 74.72395045538597,
    "y": 74.72395045538597,
    "z": 74.72395045538597
  }
}
```

### <a name="event-and-state-types"></a>Tipos de eventos y estados

En esta sección se muestran ejemplos de eventos y estados de telemetría que un dispositivo envía a una aplicación de IoT Central.

El siguiente fragmento de código de un DCM muestra la definición de un tipo de evento `integer`:

```json
{
  "@id": "<element id>",
  "@type": [
    "Telemetry",
    "SemanticType/Event"
  ],
  "displayName": {
    "en": "IntegerEvent"
  },
  "name": "IntegerEvent",
  "schema": "integer"
}
```

Un cliente de dispositivo debe enviar los datos del evento como JSON con un aspecto similar al del ejemplo siguiente:

```json
{ "IntegerEvent": 74 }
```

El siguiente fragmento de código de un DCM muestra la definición de un tipo de estado `integer`:

```json
{
  "@id": "<element id>",
  "@type": [
    "Telemetry",
    "SemanticType/State"
  ],
  "displayName": {
    "en": "IntegerState"
  },
  "name": "IntegerState",
  "schema": {
    "@id": "<element id>",
    "@type": "Enum",
    "valueSchema": "integer",
    "enumValues": [
      {
        "@id": "<element id>",
        "@type": "EnumValue",
        "displayName": {
          "en": "Level1"
        },
        "enumValue": 1,
        "name": "Level1"
      },
      {
        "@id": "<element id>",
        "@type": "EnumValue",
        "displayName": {
          "en": "Level2"
        },
        "enumValue": 2,
        "name": "Level2"
      },
      {
        "@id": "<element id>",
        "@type": "EnumValue",
        "displayName": {
          "en": "Level3"
        },
        "enumValue": 3,
        "name": "Level3"
      }
    ]
  }
}
```

Un cliente de dispositivo debe enviar el estado como JSON con un aspecto similar al del ejemplo siguiente. Los valores de estado entero posibles son `1`, `2` o `3`:

```json
{ "IntegerState": 2 }
```

## <a name="properties"></a>Propiedades

> [!NOTE]
> Los formatos de carga de las propiedades se aplican a las aplicaciones creadas el 14/07/2020 o después.

### <a name="primitive-types"></a>Tipos primitivos

En esta sección se muestran ejemplos de tipos de propiedad primitivos que un dispositivo envía a una aplicación de IoT Central.

El siguiente fragmento de código de un DCM muestra la definición de un tipo de propiedad `boolean`:

```json
{
  "@id": "<element id>",
  "@type": "Property",
  "displayName": {
    "en": "BooleanProperty"
  },
  "name": "BooleanProperty",
  "schema": "boolean"
}
```

Un cliente de dispositivo debe enviar una carga JSON similar a la del ejemplo siguiente como una propiedad notificada en el dispositivo gemelo:

```json
{ "BooleanProperty": false }
```

El siguiente fragmento de código de un DCM muestra la definición de un tipo de propiedad `boolean`:

```json
{
  "@id": "<element id>",
  "@type": "Property",
  "displayName": {
    "en": "LongProperty"
  },
  "name": "LongProperty",
  "schema": "long"
}
```

Un cliente de dispositivo debe enviar una carga JSON similar a la del ejemplo siguiente como una propiedad notificada en el dispositivo gemelo:

```json
{ "LongProperty": 439 }
```

El siguiente fragmento de código de un DCM muestra la definición de un tipo de propiedad `date`:

```json
{
  "@id": "<element id>",
  "@type": "Property",
  "displayName": {
    "en": "DateProperty"
  },
  "name": "DateProperty",
  "schema": "date"
}
```

Un cliente de dispositivo debe enviar una carga JSON similar a la del ejemplo siguiente como una propiedad notificada en el dispositivo gemelo. Los tipos `Date` deben cumplir con la norma ISO 8061:

```json
{ "DateProperty": "2020-05-17" }
```

El siguiente fragmento de código de un DCM muestra la definición de un tipo de propiedad `duration`:

```json
{
  "@id": "<element id>",
  "@type": "Property",
  "displayName": {
    "en": "DurationProperty"
  },
  "name": "DurationProperty",
  "schema": "duration"
}
```

Un cliente de dispositivo debe enviar una carga JSON similar a la del ejemplo siguiente como una propiedad notificada en el dispositivo gemelo; las duraciones deben cumplir con la duración de la norma ISO 8601:

```json
{ "DurationProperty": "PT10H24M6.169083011336625S" }
```

El siguiente fragmento de código de un DCM muestra la definición de un tipo de propiedad `float`:

```json
{
  "@id": "<element id>",
  "@type": "Property",
  "displayName": {
    "en": "FloatProperty"
  },
  "name": "FloatProperty",
  "schema": "float"
}
```

Un cliente de dispositivo debe enviar una carga JSON similar a la del ejemplo siguiente como una propiedad notificada en el dispositivo gemelo:

```json
{ "FloatProperty": 1.9 }
```

El siguiente fragmento de código de un DCM muestra la definición de un tipo de propiedad `string`:

```json
{
  "@id": "<element id>",
  "@type": "Property",
  "displayName": {
    "en": "StringProperty"
  },
  "name": "StringProperty",
  "schema": "string"
}
```

Un cliente de dispositivo debe enviar una carga JSON similar a la del ejemplo siguiente como una propiedad notificada en el dispositivo gemelo:

```json
{ "StringProperty": "A string value - could be a URL" }
```

### <a name="complex-types"></a>Tipos complejos

En esta sección se muestran ejemplos de tipos de propiedad complejos que un dispositivo envía a una aplicación de IoT Central.

El siguiente fragmento de código de un DCM muestra la definición de un tipo de propiedad `geopoint`:

```json
{
  "@id": "<element id>",
  "@type": "Property",
  "displayName": {
    "en": "GeopointProperty"
  },
  "name": "GeopointProperty",
  "schema": "geopoint"
}
```

Un cliente de dispositivo debe enviar una carga JSON similar a la del ejemplo siguiente como una propiedad notificada en el dispositivo gemelo:

```json
{
  "GeopointProperty": {
    "lat": 47.64263,
    "lon": -122.13035,
    "alt": 0
  }
}
```

El siguiente fragmento de código de un DCM muestra la definición de un tipo de propiedad `Enum`:

```json
{
  "@id": "<element id>",
  "@type": "Property",
  "displayName": {
    "en": "EnumProperty"
  },
  "name": "EnumProperty",
  "schema": {
    "@id": "<element id>",
    "@type": "Enum",
    "displayName": {
      "en": "Enum"
    },
    "valueSchema": "integer",
    "enumValues": [
      {
        "@id": "<element id>",
        "@type": "EnumValue",
        "displayName": {
          "en": "Item1"
        },
        "enumValue": 0,
        "name": "Item1"
      },
      {
        "@id": "<element id>",
        "@type": "EnumValue",
        "displayName": {
          "en": "Item2"
        },
        "enumValue": 1,
        "name": "Item2"
      },
      {
        "@id": "<element id>",
        "@type": "EnumValue",
        "displayName": {
          "en": "Item3"
        },
        "enumValue": 2,
        "name": "Item3"
      }
    ]
  }
}
```

Un cliente de dispositivo debe enviar una carga JSON similar a la del ejemplo siguiente como una propiedad notificada en el dispositivo gemelo. Los valores posibles son `0` y `1`, que se muestran en IoT Central como `Item1`, `Item2` y `Item3`:

```json
{ "EnumProperty": 1 }
```

El siguiente fragmento de código de un DCM muestra la definición de un tipo de propiedad `Object`. Este objeto tiene dos campos con tipos `string` y `integer`:

```json
{
  "@id": "<element id>",
  "@type": "Property",
  "displayName": {
    "en": "ObjectProperty"
  },
  "name": "ObjectProperty",
  "schema": {
    "@id": "<element id>",
    "@type": "Object",
    "displayName": {
      "en": "Object"
    },
    "fields": [
      {
        "@id": "<element id>",
        "@type": "SchemaField",
        "displayName": {
          "en": "Field1"
        },
        "name": "Field1",
        "schema": "integer"
      },
      {
        "@id": "<element id>",
        "@type": "SchemaField",
        "displayName": {
          "en": "Field2"
        },
        "name": "Field2",
        "schema": "string"
      }
    ]
  }
}
```

Un cliente de dispositivo debe enviar una carga JSON similar a la del ejemplo siguiente como una propiedad notificada en el dispositivo gemelo:

```json
{
  "ObjectProperty": {
    "Field1": 37,
    "Field2": "A string value"
  }
}
```

El siguiente fragmento de código de un DCM muestra la definición de un tipo de propiedad `vector`:

```json
{
  "@id": "<element id>",
  "@type": "Property",
  "displayName": {
    "en": "VectorProperty"
  },
  "name": "VectorProperty",
  "schema": "vector"
}
```

Un cliente de dispositivo debe enviar una carga JSON similar a la del ejemplo siguiente como una propiedad notificada en el dispositivo gemelo:

```json
{
  "VectorProperty": {
    "x": 74.72395045538597,
    "y": 74.72395045538597,
    "z": 74.72395045538597
  }
}
```

### <a name="writeable-property-types"></a>Tipos de propiedad grabables

En esta sección se muestran ejemplos de tipos de propiedad grabables que un dispositivo recibe de una aplicación de IoT Central.

IoT Central espera una respuesta del dispositivo a las actualizaciones de las propiedades grabables. El mensaje de respuesta debe incluir los campos `ac` y `av`. El campo `ad` es opcional. Consulte los fragmentos de código siguientes para ver ejemplos.

`ac` es un campo numérico que usa los valores de la tabla siguiente:

| Value | Etiqueta | Descripción |
| ----- | ----- | ----------- |
| `'ac': 200` | Completed | La operación de cambio de propiedad se ha completado correctamente. |
| `'ac': 202` o `'ac': 201` | Pending | La operación de cambio de propiedad está pendiente o en curso. |
| `'ac': 4xx` | Error | El cambio de propiedad solicitado no era válido o tenía un error. |
| `'ac': 5xx` | Error | El dispositivo experimentó un error inesperado al procesar el cambio solicitado. |

`av` es el número de versión enviado al dispositivo.

`ad` es una descripción de cadena de opción.

El siguiente fragmento de código de un DCM muestra la definición de un tipo de propiedad `string` grabable:

```json
{
  "@id": "<element id>",
  "@type": "Property",
  "displayName": {
    "en": "StringPropertyWritable"
  },
  "name": "StringPropertyWritable",
  "writable": true,
  "schema": "string"
}
```

El dispositivo recibe la siguiente carga de IoT Central:

```json
{  
  "StringPropertyWritable": "A string from IoT Central", "$version": 7
}
```

El dispositivo debe enviar la siguiente carga JSON a IoT Central después de procesar la actualización. Este mensaje incluye el número de versión de la actualización original recibida de IoT Central. Cuando IoT Central recibe este mensaje, marca la propiedad como **sincronizada** en la interfaz de usuario:

```json
{
  "StringPropertyWritable": {
    "value": "A string from IoT Central",
    "ac": 200,
    "ad": "completed",
    "av": 7
  }
}
```

El siguiente fragmento de código de un DCM muestra la definición de un tipo de propiedad `Enum` grabable:

```json
{
  "@id": "<element id>",
  "@type": "Property",
  "displayName": {
    "en": "EnumPropertyWritable"
  },
  "name": "EnumPropertyWritable",
  "writable": true,
  "schema": {
    "@id": "<element id>",
    "@type": "Enum",
    "displayName": {
      "en": "Enum"
    },
    "valueSchema": "integer",
    "enumValues": [
      {
        "@id": "<element id>",
        "@type": "EnumValue",
        "displayName": {
          "en": "Item1"
        },
        "enumValue": 0,
        "name": "Item1"
      },
      {
        "@id": "<element id>",
        "@type": "EnumValue",
        "displayName": {
          "en": "Item2"
        },
        "enumValue": 1,
        "name": "Item2"
      },
      {
        "@id": "<element id>",
        "@type": "EnumValue",
        "displayName": {
          "en": "Item3"
        },
        "enumValue": 2,
        "name": "Item3"
      }
    ]
  }
}
```

El dispositivo recibe la siguiente carga de IoT Central:

```json
{  
  "EnumPropertyWritable":  1 , "$version": 10
}
```

El dispositivo debe enviar la siguiente carga JSON a IoT Central después de procesar la actualización. Este mensaje incluye el número de versión de la actualización original recibida de IoT Central. Cuando IoT Central recibe este mensaje, marca la propiedad como **sincronizada** en la interfaz de usuario:

```json
{
  "EnumPropertyWritable": {
    "value": 1,
    "ac": 200,
    "ad": "completed",
    "av": 10
  }
}
```

## <a name="commands"></a>Comandos:

### <a name="synchronous-command-types"></a>Tipos de comandos sincrónicos

El siguiente fragmento de código de un DCM muestra la definición de un comando sincrónico que no tiene ningún parámetro y que no espera que el dispositivo devuelva nada:

```json
{
  "@id": "<element id>",
  "@type": "Command",
  "commandType": "synchronous",
  "durable": false,
  "displayName": {
    "en": "SynchronousCommandBasic"
  },
  "name": "SynchronousCommandBasic"
}
```

El dispositivo recibe una carga vacía en la solicitud y debe devolver una carga vacía en la respuesta con un código de respuesta HTTP `200` para indicar que la operación se ha realizado correctamente.

El siguiente fragmento de código de un DCM muestra la definición de un comando sincrónico que tiene un parámetro entero y que espera que el dispositivo devuelva un valor entero:

```json
{
  "@id": "<element id>",
  "@type": "Command",
  "commandType": "synchronous",
  "durable": false,
  "request": {
    "@id": "<element id>",
    "@type": "SchemaField",
    "displayName": {
      "en": "RequestParam"
    },
    "name": "RequestParam",
    "schema": "integer"
  },
  "response": {
    "@id": "<element id>",
    "@type": "SchemaField",
    "displayName": {
      "en": "ResponseParam"
    },
    "name": "ResponseParam",
    "schema": "integer"
  },
  "displayName": {
    "en": "SynchronousCommandSimple"
  },
  "name": "SynchronousCommandSimple"
}
```

El dispositivo recibe un valor entero como la carga de la solicitud. El dispositivo debe devolver un valor entero como la carga de respuesta con un código de respuesta HTTP `200` para indicar que la operación se ha realizado correctamente.

El siguiente fragmento de código de un DCM muestra la definición de un comando sincrónico que tiene un parámetro de objeto y que espera que el dispositivo devuelva un objeto. En este ejemplo, ambos objetos tienen campos de cadena y enteros:

```json
{
  "@id": "<element id>",
  "@type": "Command",
  "commandType": "synchronous",
  "durable": false,
  "request": {
    "@id": "<element id>",
    "@type": "SchemaField",
    "displayName": {
      "en": "RequestParam"
    },
    "name": "RequestParam",
    "schema": {
      "@id": "<element id>",
      "@type": "Object",
      "displayName": {
        "en": "Object"
      },
      "fields": [
        {
          "@id": "<element id>",
          "@type": "SchemaField",
          "displayName": {
            "en": "Field1"
          },
          "name": "Field1",
          "schema": "integer"
        },
        {
          "@id": "<element id>",
          "@type": "SchemaField",
          "displayName": {
            "en": "Field2"
          },
          "name": "Field2",
          "schema": "string"
        }
      ]
    }
  },
  "response": {
    "@id": "<element id>",
    "@type": "SchemaField",
    "displayName": {
      "en": "ResponseParam"
    },
    "name": "ResponseParam",
    "schema": {
      "@id": "<element id>",
      "@type": "Object",
      "displayName": {
        "en": "Object"
      },
      "fields": [
        {
          "@id": "<element id>",
          "@type": "SchemaField",
          "displayName": {
            "en": "Field1"
          },
          "name": "Field1",
          "schema": "integer"
        },
        {
          "@id": "<element id>",
          "@type": "SchemaField",
          "displayName": {
            "en": "Field2"
          },
          "name": "Field2",
          "schema": "string"
        }
      ]
    }
  },
  "displayName": {
    "en": "SynchronousCommandComplex"
  },
  "name": "SynchronousCommandComplex"
}
```

El siguiente fragmento de código muestra una carga de solicitud de ejemplo enviada al dispositivo:

```json
{ "Field1": 56, "Field2": "A string value" }
```

El siguiente fragmento de código muestra una carga de respuesta de ejemplo enviada desde el dispositivo. Use un código de respuesta HTTP `200` para indicar que la operación se ha realizado correctamente:

```json
{ "Field1": 87, "Field2": "Another string value" }
```

### <a name="asynchronous-command-types"></a>Tipos de comandos asincrónicos

El siguiente fragmento de código de un DCM muestra la definición de un comando asincrónico. El comando tiene un parámetro de entero y espera que el dispositivo devuelva un valor entero:

```json
{
  "@id": "<element id>",
  "@type": "Command",
  "commandType": "asynchronous",
  "durable": false,
  "request": {
    "@id": "<element id>",
    "@type": "SchemaField",
    "displayName": {
      "en": "RequestParam"
    },
    "name": "RequestParam",
    "schema": "integer"
  },
  "response": {
    "@id": "<element id>",
    "@type": "SchemaField",
    "displayName": {
      "en": "ResponseParam"
    },
    "name": "ResponseParam",
    "schema": "integer"
  },
  "displayName": {
    "en": "AsynchronousCommandSimple"
  },
  "name": "AsynchronousCommandSimple"
}
```

El dispositivo recibe un valor entero como la carga de la solicitud. El dispositivo debe devolver una carga de respuesta vacía con un código de respuesta HTTP `202` para indicar que el dispositivo ha aceptado la solicitud de procesamiento asincrónico.

Cuando el dispositivo termina de procesar la solicitud, debe enviar una propiedad a IoT Central similar a la del ejemplo siguiente. El nombre de la propiedad debe ser el mismo que el nombre del comando:

```json
{
  "AsynchronousCommandSimple": {
    "value": 87
  }
}
```

## <a name="next-steps"></a>Pasos siguientes

Como desarrollador de dispositivos, ahora conoce las plantillas de dispositivos, el siguiente paso que se sugiere es leer [Conexión a Azure IoT Central](./concepts-get-connected.md) para más información sobre cómo registrar dispositivos con IoT Central y cómo IoT Central protege las conexiones de dispositivos.
