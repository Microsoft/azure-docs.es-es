---
title: ¿Qué son las plantillas de dispositivos en Azure IoT Central | Microsoft Docs
description: Las plantillas de dispositivos de Azure IoT Central permiten especificar el comportamiento de los dispositivos conectados a la aplicación.
author: dominicbetts
ms.author: dobett
ms.date: 05/21/2020
ms.topic: conceptual
ms.service: iot-central
services: iot-central
ms.openlocfilehash: 1eab1022e9b0f03597c108c3c927909aa9bb2712
ms.sourcegitcommit: 46f8457ccb224eb000799ec81ed5b3ea93a6f06f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87337116"
---
# <a name="what-are-device-templates"></a>¿Qué son las plantillas de dispositivo?

_Este artículo se aplica a desarrolladores de dispositivos y generadores de soluciones._

Una plantilla de dispositivo en Azure IoT Central es un plano técnico que define las características y los comportamientos de un tipo de dispositivo que se conecta a la aplicación. Por ejemplo, la plantilla de dispositivo define la telemetría que un dispositivo envía para que IoT Central pueda crear visualizaciones que usen las unidades y los tipos de datos correctos.

Un generador de soluciones agrega plantillas de dispositivos a una aplicación de IoT Central. Un desarrollador de dispositivos escribe el código del dispositivo que implementa los comportamientos definidos en la plantilla de dispositivo.

En una plantilla de dispositivo se incluyen las secciones siguientes:

- _Un modelo de funcionalidad del dispositivo (DCM)_ . Esta parte de la plantilla de dispositivo define cómo interactúa el dispositivo con la aplicación. Un desarrollador de dispositivos implementa los comportamientos definidos en el DCM.
- _Propiedades de la nube_. Esta parte de la plantilla de dispositivo permite al desarrollador de soluciones especificar los metadatos del dispositivo que se van a almacenar. Las propiedades de la nube nunca se sincronizan con los dispositivos y solo existen en la aplicación. Las propiedades de la nube no afectan al código que escribe el desarrollador de dispositivos para implementar el DCM.
- _Personalizaciones_. Esta parte de la plantilla de dispositivo permite que el desarrollador de soluciones reemplace algunas de las definiciones del DCM. Las personalizaciones son útiles si el desarrollador de soluciones desea restringir el modo en que la aplicación controla un valor, como cambiar el nombre para mostrar de una propiedad o el color usado para mostrar un valor de telemetría. Las personalizaciones no afectan al código que escribe el desarrollador de dispositivos para implementar el DCM.
- _Vistas_. Esta parte de la plantilla de dispositivo permite que el desarrollador de soluciones defina visualizaciones para ver los datos del dispositivo y los formularios para administrar y controlar un dispositivo. Las vistas usan el DCM, las propiedades de la nube y las personalizaciones. Las vistas no afectan al código que escribe el desarrollador de dispositivos para implementar el DCM.

> [!NOTE]
> La [versión de actualización de IoT Plug and Play Public (versión preliminar)](../../iot-pnp/overview-iot-plug-and-play.md) está dirigida a los desarrolladores de dispositivos y OEM para que empiecen a crear dispositivos que puedan certificarse en IoT Plug and Play con anterioridad al lanzamiento de disponibilidad general.

## <a name="device-capability-models"></a>Modelos de funcionalidad del dispositivo

Un DCM define cómo interactúa un dispositivo con la aplicación de IoT Central. El desarrollador de dispositivos debe asegurarse de que el dispositivo implementa los comportamientos definidos en el DCM para que IoT Central pueda supervisar y administrar el dispositivo. Un DCM se compone de una o varias _interfaces_ y cada interfaz puede definir una colección de tipos de _telemetría_, _propiedades del dispositivo_ y _comandos_. Un desarrollador de soluciones puede importar un archivo JSON, que defina el DCM, a una plantilla de dispositivo, o usar la interfaz de usuario web en IoT Central para crear o editar un DCM. Los cambios en un DCM realizados mediante la interfaz de usuario web requieren la [cambio de versión de la plantilla de dispositivo](./howto-version-device-template.md).

Un desarrollador de soluciones también puede exportar un archivo JSON que contenga el DCM. Un desarrollador de dispositivos puede usar este documento JSON para comprender cómo debe comunicarse el dispositivo con la aplicación de IoT Central.

El archivo JSON que define el DCM usa el [Lenguaje de definición de gemelos digitales (DTDL) V1](https://github.com/Azure/IoTPlugandPlay/tree/master/DTDL). IoT Central espera que el archivo JSON contenga el DCM con las interfaces definidas insertadas, en lugar de en archivos independientes.

Habitualmente, los dispositivos IoT constan de:

- Elementos personalizados, que son las cosas que hacen que el dispositivo sea único.
- Elementos estándar, que son cosas comunes a todos los dispositivos.

Estos elementos se denominan _interfaces_  en un DCM. Las interfaces definen los detalles de cada uno de los elementos que implementa el dispositivo. Las interfaces se pueden reutilizar entre los DCM.

En el ejemplo siguiente se muestra el esquema de un modelo de funcionalidad del dispositivo para un sensor medioambiental con dos interfaces:

```json
{
  "@id": "urn:contoso:sensor_device:1",
  "@type": "CapabilityModel",
  "displayName": "Environment Sensor Capability Model",
  "implements": [
    {
      "@type": "InterfaceInstance",
      "name": "deviceinfo",
      "schema": {
        "@id": "urn:azureiot:DeviceManagement:DeviceInformation:1",
        "@type": "Interface",
        "displayName": "Device Information",
        "@context": "http://azureiot.com/v1/contexts/IoTModel.json",
        "contents": [
          ...
        ]
      }
    },
    {
      "@type": "InterfaceInstance",
      "name": "sensor",
      "schema": {
        "@id": "urn:contoso:EnvironmentalSensor:1",
        "@type": "Interface",
        "displayName": "Environmental Sensor",
        "@context": "http://azureiot.com/v1/contexts/IoTModel.json",
        "contents": [
          ...
        ]
      }
    }
  ],
  "@context": "http://azureiot.com/v1/contexts/IoTModel.json"
}
```

Un modelo de funcionalidad tiene varios campos obligatorios:

- `@id`: un identificador único en forma de nombre de recursos uniforme simple.
- `@type`: declara que este objeto es un modelo de funcionalidad.
- `@context`: especifica la versión de DTDL que se usa para el modelo de funcionalidad.
- `implements`: enumera las interfaces que implementa el dispositivo.

Todas las entradas de la lista de interfaces de la sección de implementaciones tienen:

- `name`: el nombre de programación de la interfaz.
- `schema`: la interfaz que implementa el modelo de funcionalidad.

Una interfaz tiene varios campos obligatorios:

- `@id`: un identificador único en forma de nombre de recursos uniforme simple.
- `@type`: declara que este objeto es una interfaz.
- `@context`: especifica la versión de DTDL que se usa para la interfaz.
- `contents`: enumera las propiedades, la telemetría y los comandos que componen el dispositivo.

Hay algunos campos opcionales que puede usar para agregar más detalles al modelo de funcionalidad, como el de nombre para mostrar y el de descripción.

### <a name="interface"></a>Interfaz

DTDL permite describir las funcionalidades del dispositivo. Las funcionalidades relacionadas se agrupan en interfaces. Las interfaces describen las propiedades, la telemetría y los comandos que implementa un elemento del dispositivo:

- `Properties`. Las propiedades son campos de datos que representan el estado de un dispositivo. Se usan para representar el estado duradero del dispositivo, como el estado de encendido-apagado de una bomba de líquido refrigerante. Las propiedades también pueden representar propiedades básicas del dispositivo, como la versión del firmware del dispositivo. Las propiedades se pueden declarar como de solo lectura o de escritura.
- `Telemetry`. Los campos de telemetría representan las medidas de los sensores. Cada vez que un dispositivo toma una medida de un sensor, debe enviar un evento de telemetría que contenga los datos del sensor.
- `Commands`. Los comandos representan métodos que los usuarios del dispositivo pueden ejecutar en el dispositivo. Por ejemplo, un comando de restablecimiento o un comando para encender o apagar un ventilador.

En el ejemplo siguiente se muestra la definición de la interfaz del sensor ambiental:

```json
{
  "@type": "Property",
  "displayName": "Device State",
  "description": "The state of the device. Two states online/offline are available.",
  "name": "state",
  "schema": "boolean"
},
{
  "@type": "Property",
  "displayName": "Customer Name",
  "description": "The name of the customer currently operating the device.",
  "name": "name",
  "schema": "string",
  "writable": true
},
{
  "@type": [
    "Telemetry",
    "SemanticType/Temperature"
  ],
  "description": "Current temperature on the device",
  "displayName": "Temperature",
  "name": "temp",
  "schema": "double",
  "unit": "Units/Temperature/fahrenheit"
},
{
  "@type": "Command",
  "name": "turnon",
  "comment": "This Commands will turn-on the LED light on the device.",
  "commandType": "synchronous"
},
{
  "@type": "Command",
  "name": "turnoff",
  "comment": "This Commands will turn-off the LED light on the device.",
  "commandType": "synchronous"
}
```

En este ejemplo se muestran dos propiedades, un tipo de telemetría y dos comandos. Una descripción de campo mínima tiene:

- `@type` para especificar el tipo de funcionalidad: `Telemetry`, `Property`o `Command`.  En algunos casos, el tipo incluye un tipo semántico para permitir que IoT Central realice algunas suposiciones sobre cómo controlar el valor.
- `name` para el valor de telemetría.
- `schema` para especificar el tipo de datos para la telemetría o la propiedad. Este valor puede ser un tipo primitivo, como doble, entero, booleano o cadena. También se admiten tipos de objetos complejos, matrices y mapas.
- `commandType` para especificar cómo debe controlarse el comando.

Los campos opcionales, como el nombre para mostrar y la descripción, permiten agregar más detalles a la interfaz y las funcionalidades.

### <a name="properties"></a>Propiedades

De forma predeterminada, las propiedades son de solo lectura. Las propiedades de solo lectura significan que el dispositivo informa de las actualizaciones de valores de propiedad a la aplicación de IoT Central. La aplicación de IoT Central no puede establecer el valor de una propiedad de solo lectura.

Una propiedad también se puede marcar como editable en una interfaz. Un dispositivo puede recibir una actualización de una propiedad editable desde la aplicación de IoT Central, así como notificar las actualizaciones de los valores de propiedad a la aplicación.

No es preciso que los dispositivos estén conectados para establecer los valores de propiedad. Los valores actualizados se transfieren la siguiente vez que el dispositivo se conecta a la aplicación. Este comportamiento se aplica a tanto a las propiedades de solo lectura como a las de escritura.

No use las propiedades para enviar datos de telemetría desde un dispositivo. Por ejemplo, una propiedad de solo lectura como m`temperatureSetting=80` debe significar que la temperatura del dispositivo se ha establecido en 80 y que el dispositivo está intentando llegar a esta temperatura o quedarse en ella.

En el caso de las propiedades de escritura, la aplicación de dispositivo devuelve un código de estado deseado, una versión y una descripción para indicar si ha recibido y aplicado el valor de propiedad.

### <a name="telemetry"></a>Telemetría

IoT Central le permite ver la telemetría en paneles y gráficos, y usar reglas para desencadenar acciones cuando se alcanzan los umbrales. IoT Central usa la información del DCM, como los tipos de datos, las unidades y los nombres para mostrar, para determinar cómo se muestran los valores de telemetría.

Puede usar la característica de exportación de datos de IoT Central para transmitir la telemetría a otros destinos, como Storage o Event Hubs.

### <a name="commands"></a>Comandos:

Los comandos son sincrónicos o asincrónicos. Un comando sincrónico debe ejecutarse en un plazo máximo de 30 segundos de forma predeterminada y el dispositivo debe estar conectado cuando llegue el comando. Si el dispositivo no responde a tiempo o no está conectado, se produce un error en el comando.

Use comandos asincrónicos para las operaciones de ejecución prolongada. El dispositivo envía la información del progreso mediante mensajes de telemetría. Estos mensajes tienen las siguientes propiedades del encabezado:

- `iothub-command-name`: el nombre del comando, por ejemplo `UpdateFirmware`.
- `iothub-command-request-id`: el identificador de solicitud generado en el servidor y enviado al dispositivo en la llamada inicial.
- `iothub-interface-id`:  el identificador de la interfaz en la que está definido este comando, por ejemplo `urn:example:AssetTracker:1`.
 `iothub-interface-name`: el nombre de instancia de esta interfaz, por ejemplo `myAssetTracker`.
- `iothub-command-statuscode`: el código de estado que devuelve el dispositivo, por ejemplo `202`.

## <a name="cloud-properties"></a>Propiedades de la nube

Las propiedades de la nube forman parte de la plantilla de dispositivo, pero no forman parte del DCM. Las propiedades de la nube permiten al desarrollador de soluciones especificar los metadatos de los dispositivos que se van a almacenar en la aplicación de IoT Central. Las propiedades de la nube no afectan al código que escribe el desarrollador de dispositivos para implementar el DCM.

Un desarrollador de soluciones puede agregar propiedades de la nube a los paneles y las vistas, junto a las propiedades del dispositivo, para permitir que un operador administre los dispositivos conectados a la aplicación. Un desarrollador de soluciones también puede usar las propiedades de la nube como parte de una definición de regla para que un operador pueda editar un valor de umbral.

## <a name="customizations"></a>Personalizaciones

Las personalizaciones forman parte de la plantilla de dispositivo, pero no forman parte del DCM. Las personalizaciones permiten al desarrollador de soluciones reemplazar algunas de las definiciones del DCM. Por ejemplo, un desarrollador de soluciones puede cambiar el nombre para mostrar de un tipo de telemetría o propiedad. Un desarrollador de soluciones también puede usar personalizaciones para agregar una validación, como la longitud mínima o máxima de una propiedad de dispositivo de cadena.

Las personalizaciones pueden afectan al código que escribe el desarrollador de dispositivos para implementar el DCM. Por ejemplo, una personalización podría establecer longitudes de cadena mínima y máxima, o valores numéricos mínimo y máximo para la telemetría.

## <a name="views"></a>Vistas

Un desarrollador de soluciones crea vistas que permiten a los operadores supervisar y administrar los dispositivos conectados. Las vistas forman parte de la plantilla de dispositivo, por lo que una vista se asocia a un tipo de dispositivo específico. Una vista puede incluir:

- Gráficos para trazar la telemetría.
- Iconos para mostrar las propiedades de un dispositivo de solo lectura.
- Iconos para permitir que el operador edite las propiedades grabables del dispositivo.
- Iconos para permitir que el operador edite las propiedades de la nube.
- Iconos para permitir que el operador llame a comandos, incluidos los comandos que esperan una carga útil.
- Iconos para mostrar etiquetas, imágenes o texto de Markdown.

La telemetría, las propiedades y los comandos que puede agregar a una vista vienen determinados por el DCM, las propiedades de la nube y las personalizaciones de la plantilla de dispositivo.

## <a name="next-steps"></a>Pasos siguientes

Como desarrollador de dispositivos, ahora que conoce las plantillas de dispositivo, los siguientes pasos sugeridos consisten en leer [Cargas de telemetría, propiedades y comandos](./concepts-telemetry-properties-commands.md) para obtener más información sobre los datos que un dispositivo intercambia con IoT Central.

Como desarrollador de soluciones, el siguiente paso sugerido es leer [Definición de un nuevo tipo de dispositivo IoT en la aplicación de Azure IoT Central](./howto-set-up-template.md) para obtener más información sobre cómo crear una plantilla de dispositivo.
