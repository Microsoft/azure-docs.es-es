---
title: Límites y cuotas de IoT Plug and Play (versión preliminar) | Microsoft Docs
description: Comprenda los límites, las cuotas y las limitaciones que se aplican al usar IoT Plug and Play (versión preliminar).
author: prashmo
ms.author: prashmo
ms.date: 07/21/2020
ms.topic: conceptual
ms.service: iot-pnp
services: iot-pnp
ms.openlocfilehash: 5c4377120f61792b580225a22b9f5ff51b5e1b64
ms.sourcegitcommit: 46f8457ccb224eb000799ec81ed5b3ea93a6f06f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87337405"
---
# <a name="iot-plug-and-play-preview-limits-quotas-and-throttles"></a>Límites, cuotas y limitaciones de IoT Plug and Play (versión preliminar)

En este artículo se explican los límites, las cuotas y las limitaciones específicos de IoT Plug and Play que se aplican en la versión preliminar pública. También se aplican [cuotas y limitaciones de IoT Hub](../iot-hub/iot-hub-devguide-quotas-throttling.md) existentes.

## <a name="iot-hub"></a>IoT Hub

En la versión preliminar pública, los límites y las cuotas siguientes se aplican a un centro de IoT:

| Límites, restricciones y limitaciones | Value | Notas |
|-----|-----|-----|
| Número de interfaces que se pueden registrar por centro | 1\.500 ||
| Tamaño máximo de un nombre de componente | 1-64 caracteres | Caracteres permitidos: a-z, A-Z, 0-9 (no como primer carácter) y guión bajo (no como primer ni último carácter). |
| Tamaño máximo de un nombre de propiedad | 1-64 caracteres | Caracteres permitidos: a-z, A-Z, 0-9 (no como primer carácter) y guión bajo (no como primer ni último carácter). |
| Tamaño máximo de un valor de propiedad | Igual que la [propiedad](https://github.com/Azure/opendigitaltwins-dtdl/blob/master/DTDL/v2/dtdlv2.md#property) del lenguaje de definición de Digital Twins | 5 niveles en profundidad y no pueden ser una matriz ni un esquema complejo que contenga una matriz |
| Tamaño máximo de un nombre de comando | 1-64 caracteres | Caracteres permitidos: a-z, A-Z, 0-9 (no como primer carácter) y guión bajo (no como primer ni último carácter).|
| Tamaño del dispositivo gemelo | Igual que [Límites de IoT Hub](../iot-hub/iot-hub-devguide-device-twins.md#device-twin-size) ||

## <a name="parser-library"></a>Biblioteca de analizador

La biblioteca del analizador sigue los límites que se aplican al [lenguaje de definición de Digital Twins](https://github.com/Azure/opendigitaltwins-dtdl).

## <a name="next-steps"></a>Pasos siguientes

El paso siguiente sugerido es revisar la [arquitectura de IoT Plug and Play](concepts-architecture.md).
