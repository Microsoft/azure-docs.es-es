---
title: SDK y bibliotecas de IoT Plug and Play
description: Información sobre las bibliotecas de dispositivos y servicios disponibles para el desarrollo de soluciones compatibles con IoT Plug and Play.
author: rido-min
ms.author: rmpablos
ms.date: 07/22/2020
ms.topic: reference
ms.service: iot-pnp
services: iot-pnp
ms.custom: mvc
ms.openlocfilehash: 5638cd9973c6a4df809e0b200efe85b067aae026
ms.sourcegitcommit: 42107c62f721da8550621a4651b3ef6c68704cd3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87407802"
---
# <a name="microsoft-sdks-for-iot-plug-and-play"></a>SDK de Microsoft para IoT Plug and Play

Los SDK y las bibliotecas de IoT Plug and Play permiten a los desarrolladores crear soluciones de IoT con diversos lenguajes de programación en varias plataformas. En la tabla siguiente se incluyen vínculos a ejemplos y guías de inicio rápido para ayudarle a empezar:

## <a name="device-sdks-ga"></a>SDK de dispositivo (disponibilidad general)

| Idioma | Paquete | Repositorio de código | Ejemplos | Guía de inicio rápido | Referencia |
|---|---|---|---|---|---|
| C: dispositivo | [vcpkg 1.3.9](https://github.com/Azure/azure-iot-sdk-c/blob/master/doc/setting_up_vcpkg.md) | [GitHub](https://github.com/Azure/azure-iot-sdk-c/releases/tag/2020-07-19) | [Muestras](https://github.com/Azure/azure-iot-sdk-c/tree/2020-07-19/iothub_client/samples/pnp) | [Conexión a IoT Hub](quickstart-connect-device-c.md) | [Referencia](https://docs.microsoft.com/azure/iot-hub/iot-c-sdk-ref/) |
| .NET: dispositivo | [NuGet 1.27.0](https://www.nuget.org/packages/Microsoft.Azure.Devices.Client/1.27.0) | [GitHub](https://github.com/Azure/azure-iot-sdk-csharp/tree/master/) | [Muestras](https://github.com/Azure/azure-iot-sdk-csharp/tree/master/iothub/device/samples/PnpDeviceSamples) | [Conexión a IoT Hub](quickstart-connect-device-csharp.md) | [Referencia](https://docs.microsoft.com/dotnet/api/microsoft.azure.devices.client?view=azure-dotnet) |
| Java: dispositivo | [Maven 1.24.0](https://mvnrepository.com/artifact/com.microsoft.azure.sdk.iot/iot-device-client/1.24.0) | [GitHub](https://github.com/Azure/azure-iot-sdk-java/tree/master/) | [Muestras](https://github.com/Azure/azure-iot-sdk-java/tree/master/device/iot-device-samples/pnp-device-sample) | [Conexión a IoT Hub](quickstart-connect-device-java.md) | [Referencia](https://docs.microsoft.com/java/api/com.microsoft.azure.sdk.iot.device?view=azure-java-stable) |
| Python: dispositivo | [pip 2.1.4](https://pypi.org/project/azure-iot-device/) | [GitHub](https://github.com/Azure/azure-iot-sdk-python/tree/master/) | [Muestras](https://github.com/Azure/azure-iot-sdk-python/tree/master/azure-iot-device/samples/pnp) | [Conexión a IoT Hub](quickstart-connect-device-python.md) | [Referencia](https://docs.microsoft.com/python/api/azure-iot-device/azure.iot.device?view=azure-python) |
| Node: dispositivo | [npm 1.17.0](https://www.npmjs.com/package/azure-iot-device)  | [GitHub](https://github.com/Azure/azure-iot-sdk-node/tree/master/) | [Muestras](https://github.com/Azure/azure-iot-sdk-node/tree/master/device/samples/pnp) | [Conexión a IoT Hub](quickstart-connect-device-node.md) | [Referencia](https://docs.microsoft.com/javascript/api/azure-iot-device/?view=azure-node-latest) |

## <a name="device-sdks-preview"></a>SDK de dispositivo (versión preliminar)

| Idioma | Repositorio de código/ejemplos |
|---|---|
|SDK de Azure para Embedded| [GitHub](https://github.com/Azure/azure-sdk-for-c/#) |
|Middleware de IoT de Azure RTOS| [GitHub](https://github.com/azure-rtos/azure-iot-preview#) |
|Guías de introducción de Azure RTOS | [GitHub](https://github.com/azure-rtos/getting-started) |

## <a name="service-sdks-preview"></a>SDK de servicio (versión preliminar)

| Idioma | Paquete | Repositorio de código | Ejemplos | Guía de inicio rápido | Referencia |
|---|---|---|---|---|---|
| .NET: versión preliminar del servicio de IoT Hub | [NuGet 1.27.1-preview-002](https://www.nuget.org/packages/Microsoft.Azure.Devices/1.27.1-preview-002 ) | [GitHub](https://github.com/Azure/azure-iot-sdk-csharp/tree/pnp-preview-refresh) | [Muestras](https://github.com/Azure/azure-iot-sdk-csharp/tree/pnp-preview-refresh/iothub/service/samples/PnpServiceSamples) | No aplicable | N/D |
| Java: versión preliminar del servicio de IoT Hub | [Maven 1.1.0](https://mvnrepository.com/artifact/com.microsoft.azure.sdk.iot/iot-service-client-preview/1.1.0) | [GitHub](https://github.com/Azure/azure-iot-sdk-java/tree/pnp-preview-refresh) | [Muestras](https://github.com/Azure/azure-iot-sdk-java/tree/pnp-preview-refresh/service/iot-service-samples/pnp-service-sample) | No aplicable | N/D |
| Node: versión preliminar del servicio de IoT Hub | [npm 1.12.4-pnp-refresh.4](https://www.npmjs.com/package/azure-iothub/v/1.12.4-pnp-refresh.4) | [GitHub](https://github.com/Azure/azure-iot-sdk-node/tree/pnp-preview-refresh/) | [Muestras](https://github.com/Azure/azure-iot-sdk-node/tree/pnp-preview-refresh/service/samples) | No aplicable | N/D |
| Python: versión preliminar del servicio de IoT Hub o Digital Twins | [pip 2.2.1rc1](https://pypi.org/project/azure-iot-hub/2.2.1rc1/) | [GitHub](https://github.com/Azure/azure-iot-sdk-python/tree/pnp-preview-refresh) | [Muestras](https://github.com/Azure/azure-iot-sdk-python/tree/pnp-preview-refresh/azure-iot-hub/samples) | [Interacción con la API de Digital Twins de IoT Hub](quickstart-service-python.md) | N/D |
| Node: versión preliminar del servicio de Digital Twins | [npm 1.0.0-pnp-refresh.3](https://www.npmjs.com/package/azure-iot-digitaltwins-service/v/1.0.0-pnp-refresh.3) | [GitHub](https://github.com/Azure/azure-iot-sdk-node/tree/pnp-preview-refresh/) | [Muestras](https://github.com/Azure/azure-iot-sdk-node/tree/pnp-preview-refresh/digitaltwins/samples/service/javascript) | [Interacción con la API de Digital Twins de IoT Hub](quickstart-service-node.md) | N/D |

## <a name="next-steps"></a>Pasos siguientes

Para probar los SDK y las bibliotecas, consulte la [guía para desarrolladores](concepts-developer-guide.md) y los [inicios rápidos de dispositivos](quickstart-connect-device-c.md) e [inicios rápidos de servicios](quickstart-service-node.md).
