---
title: Novedades Actualización de la versión preliminar de IoT Plug and Play | Microsoft Docs
description: Obtenga información sobre las novedades de la versión de actualización de la versión preliminar de IoT Plug and Play.
author: rido-min
ms.author: rmpablos
ms.date: 07/06/2020
ms.topic: overview
ms.custom: mvc
ms.service: iot-pnp
services: iot-pnp
manager: eliotgra
ms.openlocfilehash: 60ad7f5df4f13d626d7a2c24990c7f48db28d01d
ms.sourcegitcommit: 5f7b75e32222fe20ac68a053d141a0adbd16b347
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87475233"
---
# <a name="iot-plug-and-play-preview-refresh"></a>Actualización de la versión preliminar de IoT Plug and Play

En este artículo se describen los principales cambios de los SDK, las bibliotecas, las herramientas y los servicios de la versión de actualización de la versión preliminar de IoT Plug and Play de julio de 2020. La versión preliminar anterior de IoT Plug and Play era de agosto de 2019.

## <a name="digital-twins-definition-language-dtdl"></a>Lenguaje de definición de Digital Twins (DTDL)

Esta versión agrega compatibilidad con [DTDL v2](https://github.com/Azure/opendigitaltwins-dtdl) y deja en desuso [DTDL v1](https://github.com/Azure/opendigitaltwins-dtdl/tree/master/DTDL/v1-preview).

En la lista siguiente se muestran las diferencias principales entre DTDL v1 y DTDL v2. En DTDL v2:

- Los identificadores de modelo tienen el prefijo `dtmi` en lugar de `urn`. Los identificadores de modelo de gemelos digitales (DTMI) reemplazan a los identificadores de gemelos digitales con el prefijo `urn` usados en DTDL v1. Ahora, la versión está precedida de un `;`. Por ejemplo, donde antes usaba `urn:CompanyName:Model:1`, ahora usa `dtmi:CompanyName:Model;1`.
- Establezca `@context` en `dtmi:dtdl:context;2` en lugar de `http://azureiot.com/v1/contexts/IoTModel.json`.
- Use el tipo **Interface** en lugar de los tipos **CapabilityModel** para modelar un dispositivo.
- Los **componentes** reemplazan las instancias de **Interface**. Puede definir **Relationships** y **Components** como parte del contenido de un tipo **Interface**.
- Un tipo **Interface** puede heredarse de otro tipo **Interface**.
- Puede extender DTDL mediante _tipos semánticos extensibles_. Este sistema de tipos extensibles ofrece mayor flexibilidad que los tipos semánticos codificados de forma rígida, como la temperatura y la humedad en DTDL v1.
- La propiedad **displayUnit** se ha quitado.
- No se pueden usar caracteres de subrayado al principio ni al final de un nombre. Los caracteres de subrayado al principio de un nombre se reservan que los use el sistema.

Para trabajar con DTDL v1, debe usar la versión anterior del SDK y Azure IoT Explorer 0.10.x. Para trabajar con DTDL v2, necesita la versión más reciente del SDK y Azure IoT Explorer 0.11.x.

## <a name="no-component-and-multiple-component-implementations"></a>Sin componentes ni implementaciones de varios componentes

Los dispositivos simples que usan algunos datos de telemetría, comandos o propiedades se pueden describir en una sola interfaz sin usar componentes. Cualquier dispositivo existente se puede convertir en un dispositivo IoT Plug and Play mediante el anuncio del **Id. de modelo** sin realizar ningún cambio en la implementación del dispositivo existente.

Los dispositivos más complejos pueden agrupar datos de telemetría, comandos y propiedades en diferentes interfaces para administrar la complejidad y permitir la reutilización en todos los dispositivos. Estos dispositivos deben actualizarse para seguir un conjunto de reglas sencillas definidas en las [convenciones de mensajes de la versión preliminar de IoT Plug and Play](concepts-convention.md).

## <a name="registration-and-discovery"></a>Registro y detección

En esta versión, los dispositivos anuncian su **Id. de modelo** con IoT Hub en cada conexión. IoT Hub almacena en caché el **Id. del modelo** mediante la habilitación de las soluciones de back-end para recuperar el **Id. de modelo** mediante la propiedad `modelId` del dispositivo gemelo. También se puede recuperar el **Id. de modelo** de `$metadata.$model` en el gemelo digital.

## <a name="microsoft-defined-interfaces"></a>Interfaces definidas por Microsoft

Las siguientes interfaces definidas por Microsoft están en desuso y no se publican en el nuevo repositorio de modelos:

- **urn:azureiot:ModelDiscovery:DigitalTwin:1**
- **urn:azureiot:ModelDiscovery:ModelInformation:1**

La siguiente interfaz se publica en el nuevo repositorio de modelos: `dtmi:azure:DeviceManagement:DeviceInformation;1` disponible en la dirección URL [https://repo.azureiotrepository.com/Models/dtmi:azure:DeviceManagement:DeviceInformation;1?api-version=2020-05-01-preview](https://repo.azureiotrepository.com/Models/dtmi:azure:DeviceManagement:DeviceInformation;1?api-version=2020-05-01-preview).

## <a name="digitaltwinchangeevents"></a>DigitalTwinChangeEvents

La estructura de eventos del [origen del evento](../iot-hub/iot-hub-devguide-messages-d2c.md#non-telemetry-events) **DigitalTwinChangeEvents** ha cambiado para usar el formato **JSON-Patch**. Este es un cambio importante que no ofrece compatibilidad con versiones anteriores.

## <a name="message-routing"></a>Enrutamiento de mensajes

Los mensajes de telemetría presentan los siguientes cambios en la colección [SystemProperties](../iot-hub/iot-hub-devguide-messages-construct.md).

Ahora incluye una propiedad **dt-dataschema**, que es el **Id. de modelo** registrado por el dispositivo.

La propiedad **dt-subject** representa el componente que envía el mensaje de telemetría.

La propiedad **iothub-interface-name** está en desuso.

## <a name="device-and-service-sdks"></a>SDK de dispositivos y servicios

No existe compatibilidad con las versiones preliminares anteriores de los SDK. Tendrá que modificar el código si cambia a la versión preliminar más reciente de un SDK.

Con el enfoque basado en convenciones, no se requieren SDK de cliente de dispositivo independientes. A partir de esta versión preliminar, las bibliotecas **DigitalTwinClient** existentes estarán en desuso en todos los idiomas. En su lugar, los SDK de cliente de dispositivo IoT Hub se han actualizado para incluir una opción para anunciar el **Id. de modelo**.

Los dispositivos que no usan componentes requieren cambios mínimos en el código; solo se anuncia el **Id. de modelo**. Los dispositivos más complejos que usan varios componentes pueden requerir algunas funciones reutilizables para implementar las [convenciones](concepts-convention.md). Los ejemplos de dispositivos incluyen un conjunto de funciones que puede volver a usar en la implementación del dispositivo.

### <a name="service-sdks"></a>SDK de servicio

El SDK de servicio está disponible en [Node.js](https://github.com/Azure/azure-iot-sdk-node/blob/digitaltwins-preview/digitaltwins/service/readme.md) y [Python](https://github.com/Azure/azure-iot-sdk-python/blob/digitaltwins-preview/azure-iot-hub/README.md).

## <a name="vs-code-extension"></a>Extensión de VS Code

La extensión [Azure IoT Device Workbench](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.vscode-iot-workbench) proporciona compatibilidad con la creación de DTDL v1, integración con la versión anterior del repositorio de modelos y generación de código.

Si necesita compatibilidad con la creación de DTDL v2 en VS Code, instale la nueva [extensión DTDL](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.vscode-dtdl) en VS Code. La extensión no proporciona la integración con el repositorio de modelos ni la generación de código. En el repositorio, la administración de modelos ahora se realiza mediante una [interfaz de usuario web](https://aka.ms/iotmodelrepo) o la [interfaz de la línea de comandos](https://docs.microsoft.com/cli/azure/ext/azure-iot/iot/pnp?view=azure-cli-latest).

## <a name="digital-twin-service-side-rest-apis"></a>API de REST de servicio de gemelos digitales

Las cargas y las [API de REST del servicio de gemelos digitales](https://docs.microsoft.com/rest/api/iothub/service/digitaltwin) han cambiado. Las API admitidas son:

- Recuperación de un gemelo digital.
- Llamada a un comando.
- Actualización de un gemelo digital mediante la carga **JSON-Patch**.

Las API de REST existentes continúan siendo compatibles en esta versión.

## <a name="model-repository"></a>Repositorio de modelos

Ahora hay un repositorio de modelos único que contiene modelos públicos publicados y modelos de empresa privados protegidos por RBAC. Todos los modelos tienen un identificador único y son inmutables una vez creados.

Los repositorios de modelos existentes de la empresa de la versión anterior no se admiten en esta versión. Puede seguir usando el sitio web de [Azure Certified for IoT](https://preview.catalog.azureiotsolutions.com/products) para administrar los antiguos modelos de DTDL v1. Sin embargo, ya no puede utilizar este sitio web para registrar, probar ni certificar dispositivos.

## <a name="azure-iot-central"></a>Azure IoT Central

Azure IoT Central se está actualizando para admitir la versión de actualización de la versión preliminar de IoT Plug and Play.
