---
title: Convenciones de IoT Plug and Play | Microsoft Docs
description: Descripción de las convenciones que IoT Plug and Play espera que los dispositivos usen cuando envían telemetría y propiedades y controlan los comandos y las actualizaciones de propiedades.
author: rido-min
ms.author: rmpablos
ms.date: 07/10/2020
ms.topic: conceptual
ms.service: iot-pnp
services: iot-pnp
ms.openlocfilehash: ef29be53e776c4c185ac8430b3340c53ca85d855
ms.sourcegitcommit: b33c9ad17598d7e4d66fe11d511daa78b4b8b330
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88856059"
---
# <a name="iot-plug-and-play-conventions"></a>Convenciones de IoT Plug and Play

Los dispositivos de versión preliminar de IoT Plug and Play deben seguir un conjunto de convenciones cuando intercambien mensajes con un centro de IoT. Los dispositivos de la versión preliminar de IoT Plug and Play usan el protocolo MQTT para comunicarse con IoT Hub.

Se describen la telemetría, las propiedades y los comandos que implementa un dispositivo IoT Plug and Play con un _modelo_ de [lenguaje de definición de Digital Twins v2 (DTDL)](https://github.com/Azure/opendigitaltwins-dtdl). En este artículo se hace referencia a dos tipos de modelo:

- **Sin componentes**: un modelo sin componentes. El modelo declara la telemetría, las propiedades y los comandos como propiedades de nivel superior en la sección de contenido de la interfaz principal.
- **Varios componentes**: modelo compuesto por dos o más interfaces. Una interfaz principal con telemetría, propiedades y comandos. Una o más interfaces declaradas como componentes con telemetría, propiedades y comandos adicionales.

Para más información, consulte [Componentes de los modelos de IoT Plug and Play](concepts-components.md).

## <a name="identify-the-model"></a>Identificación del modelo

Para anunciar el modelo que implementa, un dispositivo IoT Plug and Play incluye el identificador del modelo en el paquete de conexión MQTT agregando `model-id` al campo `USERNAME`.

Para identificar el modelo que implementa un dispositivo, un servicio puede obtener el identificador de modelo en:

- El campo `modelId` del dispositivo gemelo.
- El campo `$metadata.$model` del gemelo digital.
- Una notificación de cambios de gemelo digital.

## <a name="telemetry"></a>Telemetría

La telemetría enviada desde un dispositivo sin componentes no requiere metadatos adicionales. El sistema agrega la propiedad `dt-dataschema`.

La telemetría enviada desde un dispositivo con varios componentes debe agregar `$.sub` como una propiedad de mensaje. El sistema agrega las propiedades `dt-subject` y `dt-dataschema`.

## <a name="read-only-properties"></a>Propiedades de solo lectura

### <a name="sample-no-component-read-only-property"></a>Propiedad de solo lectura sin componentes de ejemplo

Un dispositivo puede enviar cualquier JSON válido que sigue las reglas de DTDL v2.

:::row:::
   :::column span="":::
      **DTDL**

      ```json
      "@context": "dtmi:dtdl:context;2",
      "@id": "dtmi:example: Thermostat;1",
      "@type": "Interface",
      "contents": [
        {
          "@type": "Property",
          "name": "temperature",
          "schema": "double"
        }
      ]
      ```
   :::column-end:::
   :::column span="":::
      **Carga de ejemplo**

      ```json
      "reported" :
      {
        "temperature" : 21.3
      }
      ```
   :::column-end:::
:::row-end:::

### <a name="sample-multiple-components-read-only-property"></a>Propiedad de solo lectura de varios componentes de ejemplo

El dispositivo debe agregar el marcador `{"__t": "c"}` para indicar que el elemento hace referencia a un componente.

:::row:::
   :::column span="":::
      **DTDL**

      ```json
      "@context": "dtmi:dtdl:context;2",
      "@id": "dtmi:com:example:TemperatureController;1",
      "@type": "Interface",
      "displayName": "Temperature Controller",
      "contents": [
        {
          "@type" : "Component",
          "schema": "dtmi:com:example:Thermostat;1",
          "name": "thermostat1"
        }
      ]

      "@context": "dtmi:dtdl:context;2",
      "@id": "dtmi:com:example:Thermostat;1",
      "@type": "Interface",
      "contents": [
        {
          "@type": "Property",
          "name": "temperature",
          "schema": "double"
        }
      ]
      ```
   :::column-end:::
   :::column span="":::
      **Propiedad notificada**

      ```json
      "reported": {
        "thermostat1": {
          "__t": "c",
          "temperature": 21.3
        }
      }
      ```
   :::column-end:::
:::row-end:::

## <a name="writable-properties"></a>Propiedades editables

El dispositivo debe confirmar que recibió la propiedad mediante el envío de una propiedad notificada. La propiedad notificada debe incluir:

- `value`: el valor real de la propiedad (normalmente el valor recibido, pero el dispositivo puede decidir informar de un valor diferente).
- `ac`: un código de confirmación que usa un código de estado HTTP.
- `av`: una versión de confirmación que hace referencia a la `$version` de la propiedad deseada.
- `ad` : una descripción de confirmación opcional.

### <a name="sample-no-component-writable-property"></a>Propiedad editable sin componentes de ejemplo

Un dispositivo puede enviar cualquier JSON válido que sigue las reglas de DTDL v2:

:::row:::
   :::column span="":::
      **DTDL**

      ```json
      "@context": "dtmi:dtdl:context;2",
      "@id": "dtmi:example: Thermostat;1",
      "@type": "Interface",
      "contents": [
        {
          "@type": "Property",
          "name": "targetTemperature",
          "schema": "double",
          "writable": true
        }
      ]
      ```
   :::column-end:::
   :::column span="":::
      **Propiedad deseada**

      ```json
      "desired" :
      {
        "targetTemperature" : 21.3
      },
      "$version" : 3
      ```
   :::column-end:::
   :::column span="":::
      **Propiedad notificada**

      ```json
      "reported": {
        "targetTemperature": {
          "value": 21.3,
          "ac": 200,
          "av": 3,
          "ad": "complete"
       }
     }
      ```
   :::column-end:::
:::row-end:::

### <a name="sample-multiple-components-writable-property"></a>Propiedad editable de varios componentes de ejemplo

El dispositivo debe agregar el marcador `{"__t": "c"}` para indicar que el elemento hace referencia a un componente.

El marcador se envía solo para actualizaciones de nivel de componente, por lo que los dispositivos no deben comprobar si existe esta marca.

El dispositivo debe confirmar que recibió la propiedad mediante el envío de una propiedad notificada:

:::row:::
   :::column span="":::
      **DTDL**

      ```json
      "@context": "dtmi:dtdl:context;2",
      "@id": "dtmi:com:example:TemperatureController;1",
      "@type": "Interface",
      "displayName": "Temperature Controller",
      "contents": [
        {
          "@type" : "Component",
          "schema": "dtmi:com:example:Thermostat;1",
          "name": "thermostat1"
        }
      ]

      "@context": "dtmi:dtdl:context;2",
      "@id": "dtmi:com:example:Thermostat;1",
      "@type": "Interface",
      "contents": [
        {
          "@type": "Property",
          "name": "targetTemperature",
          "schema": "double",
          "writable": true
        }
      ]
      ```
   :::column-end:::
   :::column span="":::
      **Propiedad deseada**

      ```json
      "desired": {
        "thermostat1": {
          "__t": "c",
          "targetTemperature": 21.3
        }
      },
      "$version" : 3
      ```
   :::column-end:::
   :::column span="":::
      **Propiedad notificada**

      ```json
      "reported": {
        "thermostat1": {
          "__t": "c",
          "targetTemperature": {
            "value": 23,
            "ac": 200,
            "av": 3,
            "ad": "complete"
          }
        }
      }
      ```
   :::column-end:::
:::row-end:::

## <a name="commands"></a>Comandos:

Las interfaces sin componentes usan el nombre del comando sin prefijo.

En un dispositivo, las interfaces con varios componentes usan nombres de comando con el formato siguiente: `componentName*commandName`.

## <a name="next-steps"></a>Pasos siguientes

Ahora que aprendió sobre las convenciones de IoT Plug and Play, estos son algunos recursos adicionales:

- [Lenguaje de definición de Digital Twins (DTDL)](https://github.com/Azure/opendigitaltwins-dtdl)
- [SDK para dispositivos C](https://docs.microsoft.com/azure/iot-hub/iot-c-sdk-ref/)
- [API REST de IoT](https://docs.microsoft.com/rest/api/iothub/device)
- [Componentes del modelo](./concepts-components.md)
