---
title: Arquitectura de IoT Plug and Play | Microsoft Docs
description: Como creador de soluciones, comprenda los elementos clave arquitectónicos de IoT Plug and Play.
author: ridomin
ms.author: rmpablos
ms.date: 07/06/2020
ms.topic: conceptual
ms.custom: mvc
ms.service: iot-pnp
services: iot-pnp
manager: philmea
ms.openlocfilehash: f656de0bb2e5244e137ae21a6d7af88f3430b12c
ms.sourcegitcommit: 5f7b75e32222fe20ac68a053d141a0adbd16b347
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87475692"
---
# <a name="iot-plug-and-play-preview-architecture"></a>Arquitectura de la versión preliminar de IoT Plug and Play

La versión preliminar de IoT Plug and Play permite a los creadores de soluciones integrar dispositivos inteligentes en sus soluciones sin necesidad de configuración manual. IoT Plug and Play se basa en un _modelo_ de dispositivo que describe las funcionalidades del dispositivo para una aplicación compatible con IoT Plug and Play. Este modelo se estructura como un conjunto de interfaces que definen:

- _Propiedades_ que representan el estado de solo lectura y grabable de un dispositivo o de otra entidad. Por ejemplo, el número de serie de un dispositivo puede ser una propiedad de solo lectura, y la temperatura objetivo de un termostato puede ser una propiedad grabable.
- _Datos de telemetría_, que son los datos que emite un dispositivo, independientemente de que sean una secuencia normal de lecturas de un sensor, un error ocasional o un mensaje informativo.
- _Comandos_ que describen una función u operación que se puede realizar en un dispositivo. Por ejemplo, un comando puede reiniciar una puerta de enlace o tomar una imagen mediante una cámara remota.

Cada modelo e interfaz tiene un id. único.

El diagrama siguiente muestra los elementos clave de una solución IoT Plug and Play:

:::image type="content" source="media/concepts-architecture/pnp-architecture.png" alt-text="Arquitectura de IoT Plug and Play":::

## <a name="model-repository"></a>Repositorio de modelos

El [repositorio de modelos](./concepts-model-repository.md) es un almacén de definiciones de interfaz y modelo. Los modelos e interfaces se definen con el [Lenguaje de definición de Digital Twins (DTDL)](https://github.com/Azure/opendigitaltwins-dtdl).

La interfaz de usuario web le permite administrar los modelos y las interfaces.

El repositorio de modelos usa RBAC para permitirle limitar el acceso a las definiciones de interfaz.

## <a name="devices"></a>Dispositivos

Un creador de dispositivos implementa el código para ejecutarse en una instancia de Smart Device de IoT mediante uno de los [SDK de dispositivo IoT de Azure](./libraries-sdks.md). Los SDK de dispositivo ayudan al creador de dispositivos a:

- Conectarse de forma segura a una instancia de IoT Hub.
- Registrar el dispositivo en su centro de IoT y anunciar el id. de modelo que identifica la colección de interfaces que el dispositivo implementa.
- Actualizar las propiedades definidas en las interfaces de DTDL que el dispositivo implementa. Estas propiedades se implementan mediante gemelos digitales que administran la sincronización con su centro de IoT.
- Agregar controladores de comandos para los comandos definidos en las interfaces de DTDL que el dispositivo implementa.
- Enviar telemetría al centro de IoT.

## <a name="iot-hub"></a>IoT Hub

[IoT Hub](../iot-hub/about-iot-hub.md) es un servicio hospedado en la nube que actúa como centro de mensajes para comunicaciones bidireccionales entre la solución IoT y los dispositivos que administra.

Un centro de IoT:

- Hace que el id. de modelo implementado por un dispositivo esté disponible para una solución de back-end.
- Mantiene el gemelo digital asociado a cada dispositivo Plug and Play conectado al centro.
- Reenvía transmisiones de telemetría a otros servicios para su procesamiento o almacenamiento.
- Enruta eventos de cambio de gemelo digital a otros servicios para permitir la supervisión de dispositivos.

## <a name="backend-solution"></a>Solución de back-end

Una solución de back-end supervisa y controla los dispositivos conectados mediante la interacción con gemelos digitales en el centro de IoT. Use uno de los SDK de servicio para implementar la solución de back-end. Para comprender las capacidades de un dispositivo conectado, el back-end de la solución:

1. Recupera el id. de modelo que el dispositivo registró en el centro de IoT.
1. Usa el id. de modelo para recuperar las definiciones de interfaz de cualquier repositorio del modelo.
1. Utiliza el analizador de modelos para extraer información de las definiciones de interfaz.

La solución de back-end puede usar la información de las definiciones de interfaz para:

- Leer los valores de propiedad informados por los dispositivos.
- Actualizar las propiedades grabables en un dispositivo.
- Llamar a comandos implementados por un dispositivo.
- Comprender el formato de la telemetría enviada por un dispositivo.

## <a name="next-steps"></a>Pasos siguientes

Ahora que tiene una visión general de la arquitectura de una solución IoT Plug and Play, los pasos siguientes sirven para obtener más información acerca de:

- [Repositorio de modelos](./concepts-model-repository.md)
- [Integración de modelos de gemelos digitales](./concepts-model-discovery.md)
- [Desarrollo para IoT Plug and Play](./concepts-developer-guide.md)
