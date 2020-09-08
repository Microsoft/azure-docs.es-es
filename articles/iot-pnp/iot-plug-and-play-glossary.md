---
title: Glosario de términos - Versión preliminar de IoT Plug and Play | Microsoft Docs
description: 'Conceptos: glosario de términos comunes relacionados con la versión preliminar de IoT Plug and Play.'
author: dominicbetts
ms.author: dobett
ms.date: 07/22/2020
ms.topic: conceptual
ms.service: iot-pnp
services: iot-pnp
ms.openlocfilehash: 2320bed07f574c096be1883a9d82da7311e92fa7
ms.sourcegitcommit: b33c9ad17598d7e4d66fe11d511daa78b4b8b330
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88854214"
---
# <a name="glossary-of-terms-for-iot-plug-and-play-preview"></a>Glosario de términos de la versión preliminar de IoT Plug and Play

Definiciones de los términos comunes que se usan en los artículos de IoT Plug and Play.

## <a name="azure-iot-explorer-tool"></a>Herramienta Azure IoT Explorer

Azure IoT Explorer es una herramienta gráfica que puede usar para interactuar con [dispositivos IoT Plug and Play](#iot-plug-and-play-device) y probarlos. Después de instalar la herramienta en la máquina local, puede usarla para:

- Ver los dispositivos conectados a su [centro de IoT](#azure-iot-hub).
- Conectar con un dispositivo IoT Plug and Play.
- Ver los [componentes](#component) del dispositivo.
- Ver la [telemetría](#telemetry) que envía el dispositivo.
- Trabajar con las [propiedades](#properties) del dispositivo.
- Llamar a [comandos](#commands) del dispositivo.

## <a name="azure-iot-hub"></a>Azure IoT Hub

IoT Hub es un servicio administrado, hospedado en la nube, que actúa como centro de mensajes para comunicaciones bidireccionales entre la aplicación de IoT y los dispositivos que administra. Los [dispositivos IoT Plug and Play](#iot-plug-and-play-device) pueden conectarse a un centro de IoT. Una solución de IoT usa un centro de IoT para habilitar:

- Dispositivos que envían telemetría a una solución basada en la nube.
- Una solución basada en la nube para administrar los dispositivos conectados.

## <a name="azure-iot-device-sdk"></a>SDK de dispositivos IoT de Azure

Hay SDK de dispositivos para varios lenguajes que puede usar para crear aplicaciones cliente de dispositivos IoT Plug and Play.

## <a name="commands"></a>Comandos:

Los comandos definidos en una [interfaz](#interface) representan los métodos que se pueden ejecutar en el [gemelo digital](#digital-twin). Por ejemplo, un comando para restablecer un dispositivo.

## <a name="component"></a>Componente

Los componentes permiten compilar una [interfaz](#interface) del modelo como un ensamblado de otras interfaces. Un [modelo de dispositivo](#device-model) puede combinar varias interfaces como componentes. Por ejemplo, un modelo podría incluir un componente interruptor y un componente termostato. Varios componentes de un modelo pueden usar también el mismo tipo de interfaz. Por ejemplo, un modelo puede incluir dos componentes termostato.

## <a name="connection-string"></a>Cadena de conexión

Una cadena de conexión encapsula la información necesaria para conectarse a un punto de conexión. Normalmente, una cadena de conexión incluye la dirección de la información del punto de conexión y de seguridad, pero los formatos de la cadena de conexión varían en función de los servicios. Hay dos tipos de cadena de conexión asociadas con el servicio de IoT Hub:

- Las cadenas de conexión de dispositivo permiten que los [dispositivos IoT Plug and Play](#iot-plug-and-play-device) se conecten a puntos de conexión orientados a dispositivos de un centro de IoT. El código de cliente en un dispositivo usa la cadena de conexión para establecer una conexión segura con un centro de IoT.
- Las cadenas de conexión de IoT Hub permiten que las aplicaciones y herramientas de back-end se conecten de forma segura a puntos de conexión orientados a servicios de un centro de IoT. Estas soluciones y herramientas administran el centro de IoT y los dispositivos conectados a él.

## <a name="device-certification"></a>Certificación de dispositivos

El programa de certificación de dispositivos IoT Plug and Play comprueba que un dispositivo cumple los requisitos de certificación de IoT Plug and Play. Puede agregar un dispositivo certificado al [catálogo de dispositivos certificados para Azure IoT](https://aka.ms/devicecatalog).

## <a name="device-model"></a>Modelo de dispositivo

Un modelo de dispositivo describe un [dispositivo IoT Plug and Play](#iot-plug-and-play-device) y define los [componentes](#component) que lo forman. Un modelo de dispositivo simple no tiene componentes independientes y contiene una definición para una sola interfaz de nivel raíz. Un modelo de dispositivo más complejo incluye varios componentes. Un modelo de dispositivo se corresponde normalmente con un dispositivo físico, un producto o una SKU. Para definir un modelo de dispositivo, se usa el [Lenguaje de definición de Digital Twins, versión 2](#digital-twins-definition-language).

## <a name="device-builder"></a>Creador de dispositivos

Un creador de dispositivos usa un [modelo de dispositivo](#device-model) e [interfaces](#interface) al implementar código para que se ejecute en un [dispositivo IoT Plug and Play](#iot-plug-and-play-device). Los creadores de dispositivos usan normalmente uno de los [SDK de dispositivo IoT de Azure](#azure-iot-device-sdk) para implementar el cliente del dispositivo, pero no es necesario.

## <a name="device-modeling"></a>Modelado de dispositivos

Un [creador de dispositivos](#device-builder) usa el [Lenguaje de definición de Digital Twins](#digital-twins-definition-language) para modelar las funcionalidades de un [dispositivo IoT Plug and Play](#iot-plug-and-play-device). Un [creador de soluciones](#solution-builder) puede configurar una solución de IOT a partir del modelo.

## <a name="digital-twin"></a>Gemelo digital

Un gemelo digital es un modelo de [dispositivo IoT Plug and Play](#iot-plug-and-play-device). Un gemelo digital se modela mediante el [Lenguaje de definición de Digital Twins (DTDL)](#digital-twins-definition-language). Puede usar los [SDK de dispositivo IoT de Azure](#azure-iot-device-sdk) para interactuar con los gemelos digitales en tiempo de ejecución. Por ejemplo, puede establecer un valor de propiedad en un gemelo digital en un dispositivo y el SDK comunica este cambio a su solución de IoT en la nube.

## <a name="digital-twin-change-events"></a>Eventos de cambio de gemelo digital

Cuando un [dispositivo IoT Plug and Play](#iot-plug-and-play-device) se conecta a un [centro de IoT](#azure-iot-hub), el centro puede usar su capacidad de enrutamiento para enviar notificaciones de cambios de gemelos digitales. Por ejemplo, cada vez que un valor de [propiedad](#properties) cambia en un dispositivo, IoT Hub puede enviar una notificación a un punto de conexión, como un centro de eventos.

## <a name="digital-twins-definition-language"></a>Lenguaje de definición de Digital Twins

Lenguaje que sirve para describir los modelos y las interfaces de los [dispositivos IoT Plug and Play](#iot-plug-and-play-device). Use el [Lenguaje de definición de Digital Twins, versión 2](https://github.com/Azure/opendigitaltwins-dtdl) para describir las funcionalidades de un [gemelo digital](#digital-twin) y para habilitar la plataforma IoT y las soluciones IoT con el fin de aprovechar la semántica de la entidad.

## <a name="digital-twin-route"></a>Enrutamiento de gemelo digital

Una ruta configurada en un [centro de IoT](#azure-iot-hub) para ofrecer [eventos de cambio de gemelo digital](#digital-twin-change-events) a un punto de conexión, como un centro de eventos.

## <a name="interface"></a>Interfaz

Una interfaz describe las funcionalidades relacionadas que un [dispositivo IoT Plug and Play](#iot-plug-and-play-device) o un [gemelo digital](#digital-twin) implementa. Puede reutilizar las interfaces en distintos [modelos de dispositivo](#device-model). Cuando se usa una interfaz en un modelo de dispositivo, define un [componente](#component) del dispositivo.

## <a name="iot-hub-query-language"></a>Lenguaje de consulta de IoT Hub

El lenguaje de consulta de IoT Hub se usa para varios propósitos. Por ejemplo, puede usar el lenguaje para buscar dispositivos registrados en su centro de IoT o para mejorar el comportamiento de [enrutamiento del gemelo digital](#digital-twin-route).

## <a name="iot-plug-and-play-device"></a>Dispositivo IoT Plug and Play

Un dispositivo IoT Plug and Play suele ser un dispositivo informático independiente y a pequeña escala que recopila datos o controla otros dispositivos, y que ejecuta software o firmware que implementa un [modelo de dispositivo](#device-model).  Por ejemplo, un dispositivo IoT Plug and Play podría ser un dispositivo de supervisión ambiental o un controlador de un sistema de riego de agricultura inteligente. Puede escribir una solución de IoT hospedada en la nube para imponer, controlar y recibir datos de dispositivos IoT Plug and Play.

## <a name="iot-plug-and-play-conventions"></a>Convenciones de IoT Plug and Play

Se espera que los [dispositivos](#iot-plug-and-play-device) IoT Plug and Play sigan un conjunto de [convenciones](concepts-convention.md) al intercambiar datos con una solución.

## <a name="model-id"></a>Id. de modelo

Cuando un dispositivo IoT Plug and Play se conecta a IoT Hub, envía el **Id. de modelo** del modelo de [DTDL](#digital-twins-definition-language) que implementa. Esto permite que la solución encuentre el modelo de dispositivo.

## <a name="model-repository"></a>Repositorio de modelos

En el [repositorio de modelos](concepts-model-repository.md) se almacenan los [modelos de dispositivo](#device-model) y las [interfaces](#interface).

## <a name="model-repository-rest-api"></a>API REST del repositorio de modelos

Una API para administrar e interactuar con el repositorio de modelos. Por ejemplo, puede usar la API para agregar y buscar [modelos de dispositivo](#device-model).

## <a name="properties"></a>Propiedades

Las propiedades son campos de datos definidos en una [interfaz](#interface) que representan un estado de un gemelo digital. Las propiedades se pueden declarar como de solo lectura o de escritura. Las propiedades de solo lectura, como el número de serie, se establecen mediante código que se ejecuta en el propio [dispositivo IoT Plug and Play](#iot-plug-and-play-device).  Las propiedades que se pueden escribir, como un umbral de alarma, se establecen normalmente a partir de la solución de IoT basada en la nube.

## <a name="shared-access-signature"></a>Firma de acceso compartido

Las firmas de acceso compartido son un mecanismo de autenticación basado en hash seguros SHA-256 o URI. La autenticación de firma de acceso compartido tiene dos componentes: una directiva de acceso compartido y una firma de acceso compartido (a menudo denominada token). Un [dispositivo IoT Plug and Play](#iot-plug-and-play-device) usa una firma de acceso compartido para autenticarse con un [centro de IoT](#azure-iot-hub).

## <a name="solution-builder"></a>Creador de soluciones

Un creador de soluciones crea el back-end de la solución. Normalmente, un creador de soluciones trabaja con recursos de Azure, como [IoT Hub](#azure-iot-hub) y [repositorios de modelos](#model-repository).

## <a name="telemetry"></a>Telemetría

Los campos de telemetría definidos en una [interfaz](#interface) representan medidas. Estas medidas suelen ser valores como lecturas de sensores que envía el [dispositivo IoT Plug and Play](#iot-plug-and-play-device) como un flujo de datos.
