---
title: Opciones de dispositivo a la nube de IoT Hub de Azure | Microsoft Docs
description: 'Guía del desarrollador: una guía sobre cuándo usar mensajes de dispositivo a nube, propiedades notificadas o carga de archivos para comunicaciones de nube a dispositivo.'
author: wesmc7777
manager: philmea
ms.author: wesmc
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 01/29/2018
ms.custom:
- amqp
- mqtt
- 'Role: Cloud Development'
- 'Role: IoT Device'
ms.openlocfilehash: dd4f4ad7f84ebc1f5e254843e7afa4aa0f3f224f
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87319140"
---
# <a name="device-to-cloud-communications-guidance"></a>Guía de comunicación de dispositivo a nube

Al enviar información desde la aplicación del dispositivo al back-end de solución, IoT Hub presenta tres opciones:

* [Mensajes de dispositivo a nube](iot-hub-devguide-messages-d2c.md), para telemetría y alertas de series temporales.

* [Propiedades notificadas del dispositivo gemelo](iot-hub-devguide-device-twins.md), para notificar información de estado del dispositivo, como funcionalidades disponibles, condiciones o estado de los flujos de trabajo de ejecución prolongada. Por ejemplo, configuración y actualizaciones de software.

* [Cargas de archivos](iot-hub-devguide-file-upload.md), para archivos multimedia y grandes lotes de telemetría cargados por dispositivos conectados de manera intermitente o comprimidos para ahorrar ancho de banda.

[!INCLUDE [iot-hub-basic](../../includes/iot-hub-basic-partial.md)]

Esta es una comparación detallada de las distintas opciones de comunicación de dispositivo a nube.

| Factor | Mensajes de dispositivo a nube | Propiedades notificadas del dispositivo gemelo | Cargas de archivos |
| ---- | ------- | ---------- | ---- |
| Escenario | Serie temporal de telemetría y alertas. Por ejemplo, lotes de datos del sensor de 256 KB enviados cada cinco minutos. | Funcionalidades disponibles y condiciones. Por ejemplo, el modo actual de conectividad del dispositivo, como móvil o WiFi. Sincronización de flujos de trabajo de ejecución prolongada, como configuración y actualizaciones de software. | Archivos multimedia. Lotes de telemetría (generalmente comprimidos) de gran tamaño. |
| Almacenamiento y recuperación | Almacenados temporalmente por IoT Hub, hasta 7 días. Solo lectura secuencial. | Almacenados por IoT Hub en el dispositivo gemelo. Recuperables mediante el [lenguaje de consulta de IoT Hub](iot-hub-devguide-query-language.md). | Almacenadas en la cuenta de Azure Storage proporcionada por el usuario. |
| Size | Mensajes de hasta 256 KB. | El tamaño máximo de las propiedades notificadas es 32 KB. | Tamaño máximo de archivo admitido por Azure Blob Storage. |
| Frecuencia | Alta. Para más información, consulte los [Límites de IoT Hub](iot-hub-devguide-quotas-throttling.md). | Mediana. Para más información, consulte los [Límites de IoT Hub](iot-hub-devguide-quotas-throttling.md). | Baja. Para más información, consulte los [Límites de IoT Hub](iot-hub-devguide-quotas-throttling.md). |
| Protocolo | Disponible en todos los protocolos. | Disponible con MQTT o AMQP. | Disponible cuando se usa cualquier protocolo, pero hace falta HTTPS en el dispositivo. |

Es posible que una aplicación necesite enviar información como alertas o series temporales de telemetría y permitir que esté disponible en el dispositivo gemelo. En este escenario, puede elegir una de las siguientes opciones:

* La aplicación para dispositivo envía un mensaje de dispositivo a nube y notifica un cambio de propiedad.
* El back-end de la solución puede almacenar la información en las etiquetas del dispositivo gemelo cuando recibe el mensaje.

Dado que los mensajes de dispositivo a nube permiten una capacidad de proceso mucho mayor que las actualizaciones de dispositivos gemelos, a veces es aconsejable evitar actualizar el dispositivo gemelo con cada mensaje de dispositivo a nube.
