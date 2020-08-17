---
title: Información de los SDK de IoT de Azure | Microsoft Docs
description: 'Guía del desarrollador: Información y vínculos a los diversos SDK de dispositivos y servicios IoT de Azure que puede usar para compilar aplicaciones de back-end y de aplicaciones de dispositivo.'
author: wesmc7777
manager: philmea
ms.author: wesmc
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 01/14/2020
ms.custom:
- mqtt
- 'Role: IoT Device'
- 'Role: Cloud Development'
ms.openlocfilehash: 1933a53b6e69a6179faaa0bd6b86619478159b3b
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87322931"
---
# <a name="understand-and-use-azure-iot-hub-sdks"></a>Información y uso de los SDK de Azure IoT Hub

Hay dos categorías de kits de desarrollo de software (SDK) para trabajar con IoT Hub:

* Los **SDK de dispositivo de IoT Hub** permiten compilar aplicaciones que se ejecuten en los dispositivos IoT mediante clientes de dispositivo o módulo. Estas aplicaciones envían datos de telemetría a IoT Hub y, de manera opcional, reciben mensajes, trabajos, métodos o actualizaciones gemelas de dicho servicio.  También puede usar el cliente de módulo para crear [módulos](../iot-edge/iot-edge-modules.md) para el [entorno de ejecución de Azure IoT Edge](../iot-edge/about-iot-edge.md).

* Los **SDK de servicio de IoT Hub** permiten compilar aplicaciones de back-end para administrar el centro de IoT y, opcionalmente, enviar mensajes, programar trabajos, invocar métodos directos o enviar las actualizaciones de propiedades deseadas a los dispositivos o módulos IoT.

Además, también proporcionamos un conjunto de SDK para trabajar con el [servicio Device Provisioning](../iot-dps/about-iot-dps.md).

* Los **SDK de dispositivo de aprovisionamiento** permiten compilar aplicaciones que se ejecuten en los dispositivos IoT para la comunicación con el servicio Device Provisioning.

* Los **SDK del servicio de aprovisionamiento** permiten compilar aplicaciones de back-end para administrar las inscripciones en el servicio Device Provisioning.

Conozca las [ventajas de desarrollar con los SDK de Azure IoT](https://azure.microsoft.com/blog/benefits-of-using-the-azure-iot-sdks-in-your-azure-iot-solution/).

[!INCLUDE [iot-hub-basic](../../includes/iot-hub-basic-partial.md)]

## <a name="os-platform-and-hardware-compatibility"></a>Compatibilidad de hardware y de plataformas de sistema operativo

Para encontrar las plataformas admitidas para los SDK, consulte [Compatibilidad de plataformas de SDK de IoT de Azure](iot-hub-device-sdk-platform-support.md).

Para más información sobre la compatibilidad del SDK con dispositivos de hardware concretos, consulte el [catálogo de dispositivos Azure Certified for IoT](https://catalog.azureiotsolutions.com/) o el repositorio individual.

## <a name="azure-iot-hub-device-sdks"></a>SDK de dispositivos de Azure IoT Hub

Los SDK de dispositivos IoT de Microsoft Azure contienen código que facilita la compilación de aplicaciones que se conectan a servicios de Azure IoT Hub y que este administra.

SDK de dispositivos Azure IoT Hub para .NET: 

* Se descarga de [NuGet](https://www.nuget.org/packages/Microsoft.Azure.Devices.Client/).  El espacio de nombres es Microsoft.Azure.Devices.Clients, que contiene clientes de dispositivos de IoT Hub (DeviceClient, ModuleClient).
* [Código fuente](https://github.com/Azure/azure-iot-sdk-csharp)
* [Referencia de API](https://docs.microsoft.com/dotnet/api/microsoft.azure.devices?view=azure-dotnet)
* [Referencia para los módulos](https://docs.microsoft.com/dotnet/api/microsoft.azure.devices.client.moduleclient?view=azure-dotnet)

SDK de dispositivos de Azure IoT Hub para C (ANSI C - C99):

* Se instala desde [apt get, MBED, Arduino IDE o iOS](https://github.com/Azure/azure-iot-sdk-c/blob/master/readme.md#packages-and-libraries)
* [Código fuente](https://github.com/Azure/azure-iot-sdk-c)
* [Compilación del SDK de dispositivos para C](https://github.com/Azure/azure-iot-sdk-c/blob/master/iothub_client/readme.md#compiling-the-c-device-sdk)
* [Referencia de API](https://docs.microsoft.com/azure/iot-hub/iot-c-sdk-ref/)
* [Referencia para los módulos](https://docs.microsoft.com/azure/iot-hub/iot-c-sdk-ref/iothub-module-client-h)
* [Migración del SDK de C a otras plataformas](https://github.com/Azure/azure-c-shared-utility/blob/master/devdoc/porting_guide.md)
* [Documentación para desarrolladores](https://github.com/Azure/azure-iot-sdk-c/tree/master/doc): contiene información sobre la compilación cruzada y una introducción a distintas plataformas, entre otros temas.
* [Información de consumo de recursos de SDK de C de Azure IoT Hub](https://github.com/Azure/azure-iot-sdk-c/blob/master/doc/c_sdk_resource_information.md)

SDK de dispositivos Azure IoT Hub para Java:

* Se agrega al proyecto [Maven](https://github.com/Azure/azure-iot-sdk-java/blob/master/doc/java-devbox-setup.md#for-the-device-sdk)
* [Código fuente](https://github.com/Azure/azure-iot-sdk-java)
* [Referencia de API](https://docs.microsoft.com/java/api/com.microsoft.azure.sdk.iot.device)
* [Referencia para los módulos](https://docs.microsoft.com/java/api/com.microsoft.azure.sdk.iot.device.moduleclient?view=azure-java-stable)

SDK de dispositivos Azure IoT Hub para Node.js:

* Se instala desde [npm](https://www.npmjs.com/package/azure-iot-device)
* [Código fuente](https://github.com/Azure/azure-iot-sdk-node)
* [Referencia de API](https://docs.microsoft.com/javascript/api/azure-iot-device/?view=azure-iot-typescript-latest)
* [Referencia para los módulos](https://docs.microsoft.com/javascript/api/azure-iot-device/moduleclient?view=azure-node-latest)

SDK de dispositivos Azure IoT Hub para Python:

* Se instala desde [pip](https://pypi.org/project/azure-iot-device/)
* [Código fuente](https://github.com/Azure/azure-iot-sdk-python)
* [Referencia de API](https://docs.microsoft.com/python/api/azure-iot-device)

SDK de dispositivos Azure IoT Hub para iOS:

* Se instala desde [CocoaPod](https://cocoapods.org/pods/AzureIoTHubClient)
* [Muestras](https://github.com/Azure-Samples/azure-iot-samples-ios)
* Referencia de API: consulte [referencia de API de C](https://docs.microsoft.com/azure/iot-hub/iot-c-sdk-ref/)

## <a name="azure-iot-hub-service-sdks"></a>SDK de servicios de Azure IoT Hub

Los SDK de servicios IoT de Azure contienen código que facilitan la creación de aplicaciones que interactúan directamente con IoT Hub para administrar dispositivos y seguridad.

SDK de servicios Azure IoT Hub para .NET:

* Se descarga de [NuGet](https://www.nuget.org/packages/Microsoft.Azure.Devices/).  El espacio de nombres es Microsoft.Azure.Devices, que contiene clientes de servicios de IoT Hub (RegistryManager, ServiceClients).
* [Código fuente](https://github.com/Azure/azure-iot-sdk-csharp)
* [Referencia de API](https://docs.microsoft.com/dotnet/api/microsoft.azure.devices)

SDK de servicios Azure IoT Hub para Java:

* Se agrega al proyecto [Maven](https://github.com/Azure/azure-iot-sdk-java/blob/master/doc/java-devbox-setup.md#for-the-service-sdk)
* [Código fuente](https://github.com/Azure/azure-iot-sdk-java)
* [Referencia de API](https://docs.microsoft.com/java/api/com.microsoft.azure.sdk.iot.service)

SDK de servicios Azure IoT Hub para Node.js:

* Se descarga de [npm](https://www.npmjs.com/package/azure-iothub)
* [Código fuente](https://github.com/Azure/azure-iot-sdk-node)
* [Referencia de API](https://docs.microsoft.com/javascript/api/azure-iothub/?view=azure-iot-typescript-latest)

SDK de servicios Azure IoT Hub para Python:

* Se descarga de [pip](https://pypi.python.org/pypi/azure-iot-hub/)
* [Código fuente](https://github.com/Azure/azure-iot-sdk-python/tree/master)
* [Referencia de API](https://docs.microsoft.com/python/api/azure-iot-hub)

SDK de servicios Azure IoT Hub para C:

El SDK de servicios Azure IoT para C:¡ ya no está en desarrollo activo.
Seguiremos solucionando errores críticos, como bloqueos, datos dañados y vulnerabilidades de seguridad. Sin embargo, NO se agregará ninguna característica nueva ni se corregirán errores que no sean críticos.

La compatibilidad con el SDK de servicios Azure IoT está disponible en lenguajes de nivel superior ([C#](https://github.com/Azure/azure-iot-sdk-csharp), [Java](https://github.com/Azure/azure-iot-sdk-java), [Node](https://github.com/Azure/azure-iot-sdk-node), [Python](https://github.com/Azure/azure-iot-sdk-python)).

* Se descarga de [apt get, MBED, Arduino IDE o NuGet](https://github.com/Azure/azure-iot-sdk-c/blob/master/readme.md)
* [Código fuente](https://github.com/Azure/azure-iot-sdk-c)

SDK de servicios de Azure IoT Hub para iOS:

* Se instala desde [CocoaPod](https://cocoapods.org/pods/AzureIoTHubServiceClient)
* [Muestras](https://github.com/Azure-Samples/azure-iot-samples-ios)

> [!NOTE]
> Consulte los archivos Léame en los repositorios de GitHub para obtener información sobre el uso de administradores de paquetes específicos de la plataforma y el lenguaje para instalar los archivos binarios y dependencias en el equipo de desarrollo.

## <a name="microsoft-azure-provisioning-sdks"></a>SDK de aprovisionamiento de Microsoft Azure

Los **SDK de aprovisionamiento de Microsoft Azure** le permiten aprovisionar dispositivos en IoT Hub mediante el [servicio Device Provisioning](../iot-dps/about-iot-dps.md).

SDK de aprovisionamiento de dispositivos y servicios de Azure para C#:

* Se descargan desde el [SDK de dispositivo](https://www.nuget.org/packages/Microsoft.Azure.Devices.Provisioning.Client/) y el [SDK del servicio](https://www.nuget.org/packages/Microsoft.Azure.Devices.Provisioning.Service/) de NuGet.
* [Código fuente](https://github.com/Azure/azure-iot-sdk-csharp/)
* [Referencia de API](https://docs.microsoft.com/dotnet/api/microsoft.azure.devices.provisioning.client?view=azure-dotnet)

SDK de aprovisionamiento de dispositivos y servicios de Azure para C:

* Se instala desde [apt get, MBED, Arduino IDE o iOS](https://github.com/Azure/azure-iot-sdk-c/blob/master/readme.md#packages-and-libraries)
* [Código fuente](https://github.com/Azure/azure-iot-sdk-c/blob/master/provisioning_client)
* [Referencia de API](https://docs.microsoft.com/azure/iot-hub/iot-c-sdk-ref/)

SDK de aprovisionamiento de dispositivos y servicios de Azure para Java:

* Se agrega al proyecto [Maven](https://github.com/Azure/azure-iot-sdk-java/blob/master/doc/java-devbox-setup.md#for-the-service-sdk)
* [Código fuente](https://github.com/Azure/azure-iot-sdk-java/blob/master/provisioning)
* [Referencia de API](https://docs.microsoft.com/java/api/com.microsoft.azure.sdk.iot.provisioning.device?view=azure-java-stable)

SDK de aprovisionamiento de dispositivos y servicios de Azure para Node.js:

* [Código fuente](https://github.com/Azure/azure-iot-sdk-node/tree/master/provisioning)
* [Referencia de API](https://docs.microsoft.com/javascript/api/overview/azure/iothubdeviceprovisioning?view=azure-node-latest)
* Descargue el [SDK de dispositivo](https://badge.fury.io/js/azure-iot-provisioning-device) y el [SDK del servicio](https://badge.fury.io/js/azure-iot-provisioning-service) de npm

SDK de aprovisionamiento de dispositivos y servicios de Azure para Python:

* [Código fuente](https://github.com/Azure/azure-iot-sdk-python)
* Descargue el [SDK de dispositivo](https://pypi.org/project/azure-iot-device/) y el [SDK del servicio](https://pypi.org/project/azure-iothub-provisioningserviceclient/) de pip

## <a name="next-steps"></a>Pasos siguientes

Los SDK de Azure IoT también proporcionan un conjunto de herramientas para asistir en las tareas de desarrollo:

* [iothub-diagnostics](https://github.com/Azure/iothub-diagnostics): una herramienta de línea de comandos multiplataforma para ayudar a diagnosticar problemas relacionados con la conexión con IoT Hub.
* [azure-iot-explorer](https://github.com/Azure/azure-iot-explorer): aplicación de escritorio multiplataforma que se conecta a su instancia de IoT Hub y agrega, administra y se comunica con dispositivos IoT.

Documentos pertinentes relacionados con el desarrollo con los SDK de Azure IoT:

* Información acerca de la [administración de la conectividad y la mensajería confiable](iot-hub-reliability-features-in-sdks.md) mediante los SDK de IoT Hub.
* Aprenda cómo [desarrollar para plataformas móviles](iot-hub-how-to-develop-for-mobile-devices.md) como iOS y Android.
* [Compatibilidad con plataformas de SDK de Azure IoT](iot-hub-device-sdk-platform-support.md)

Otros temas de referencia en la Guía del desarrollador de IoT Hub son:

* [Puntos de conexión de IoT Hub](iot-hub-devguide-endpoints.md)
* [Lenguaje de consulta de IoT Hub para dispositivos gemelos, trabajos y enrutamiento de mensajes](iot-hub-devguide-query-language.md)
* [Cuotas y limitación](iot-hub-devguide-quotas-throttling.md)
* [Compatibilidad con MQTT de IoT Hub](iot-hub-mqtt-support.md)
* [Referencia de la API REST de IoT Hub](/rest/api/iothub/)
