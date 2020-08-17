---
title: Compatibilidad de plataformas de SDK de dispositivo IoT de Azure | Microsoft Docs
description: Los SDKs de dispositivos de código abierto están disponibles en GitHub en C,.NET (C#), Java, Node.js y Python, para conectar dispositivos a Azure IOT Hub y al servicio Device Provisioning (DPS).
author: robinsh
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 10/08/2019
ms.author: robinsh
ms.openlocfilehash: aef468d919e6f09722045f98c68383785d10b137
ms.sourcegitcommit: e71da24cc108efc2c194007f976f74dd596ab013
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87423089"
---
# <a name="azure-iot-device-sdks-platform-support"></a>Compatibilidad de plataformas de SDK de dispositivo IoT de Azure

Microsoft se esfuerza en ampliar continuamente el universo de los dispositivos compatibles con Azure IoT Hub. Microsoft publica los SDK de dispositivo de código abierto en GitHub para facilitar la conexión de los dispositivos a Azure IoT Hub y a Device Provisioning Service. Los SDK de dispositivo están disponibles para C, .NET (C#), Java, Node.js y Python. Microsoft prueba cada SDK para asegurarse de que funciona en las configuraciones admitidas que se detallan en la sección [Compatibilidad con plataformas de dispositivos y SDK de Microsoft](#microsoft-sdks-and-device-platform-support).

Además de los SDK de dispositivo, Microsoft ofrece otros varios caminos para permitir que los clientes y desarrolladores conecten sus dispositivos a IoT de Azure:

* Microsoft colabora con varias empresas asociadas para ayudarles a publicar kits de desarrollo, basados en el SDK de IoT de Azure para C, para sus plataformas de hardware.

* Microsoft trabaja con asociados de confianza para ofrecer un conjunto de dispositivos siempre en expansión que se han probado y certificado para IoT de Azure. Para obtener una lista actualizada de estos dispositivos, consulte el [catálogo de dispositivos Azure Certified for IoT](https://catalog.azureiotsolutions.com/).

* Microsoft proporciona una capa de abstracción de plataforma (PAL) en el SDK de dispositivos de Azure IoT Hub para C que ayuda a los desarrolladores a portar fácilmente el SDK a su plataforma. Para más información, consulte la [guía de portabilidad de SDK para C](https://github.com/Azure/azure-c-shared-utility/blob/master/devdoc/porting_guide.md).

En este tema se proporciona información sobre los SDK de Microsoft y las configuraciones de plataforma que admiten, así como las demás opciones indicadas anteriormente.

## <a name="microsoft-sdks-and-device-platform-support"></a>Compatibilidad con plataformas de dispositivos y SDK de Microsoft

Microsoft publica SDK de código abierto en GitHub para los siguientes lenguajes: C, .NET (C#), Node.js, Java y Python. En esta sección se enumeran los SDK y sus dependencias. Los SDK se admiten en cualquier plataforma de dispositivos que cumpla con estas dependencias.

Para cada uno de los SDK enumerados, Microsoft:

* Crea y ejecuta continuamente pruebas de un extremo a otro en la rama principal del SDK correspondiente en GitHub en varias plataformas conocidas.  Para proporcionar cobertura de pruebas en las distintas versiones del compilador, se suele realizar una prueba en comparación con la última versión de LTS y la versión más popular.

* Proporciona instrucciones de instalación o paquetes de instalación si corresponde.

* Proporciona soporte completo para los SDK en GitHub con código fuente abierto, una ruta de acceso para las contribuciones de los clientes y la participación del equipo de productos en los problemas de GitHub.

### <a name="c-sdk"></a>SDK DE C

El [SDK de dispositivo Azure IoT Hub para C](https://github.com/Azure/azure-iot-sdk-c) admite la siguiente configuración y se prueba con ella.

| SO                  | Biblioteca TLS                  | Requisitos adicionales                                                                     |
|---------------------|------------------------------|---------------------------------------------------------------------------------------------|
| Linux               | OpenSSL, WolfSSL o BearSSL | Berkeley sockets</br></br>Portable Operating System Interface (POSIX)                       |
| iOS 12.2            | OpenSSL                      | XCode emulado en OSX 10.13.4                                                               |
| Familia de productos de Windows 10   | SChannel                     |                                                                                             |
| Mbed OS 5.4         | Mbed TLS 2                   | [MXChip IoT DevKit](https://microsoft.github.io/azure-iot-developer-kit/)                  |
| SO de Azure Sphere     | WolfSSL                      | [Azure Sphere MT3620](https://azure.microsoft.com/services/azure-sphere/get-started/) |
| Arduino             | BearSSL                      | [ESP32 o ESP8266](https://github.com/Azure/azure-iot-arduino#simple-sample-instructions) 

### <a name="python-sdk"></a>SDK de Python

El [SDK de dispositivo Azure IoT Hub para Python](https://github.com/Azure/azure-iot-sdk-python) admite la siguiente configuración y se prueba con ella.

| SO                  | Compilador                          |
|---------------------|-----------------------------------|
| Linux               | Python 2.7.*, 3.5 o versiones posteriores |
| MacOS High Sierra   | Python 2.7.*, 3.5 o versiones posteriores |
| Familia de productos de Windows 10   | Python 2.7.*, 3.5 o versiones posteriores |

Solo Python versión 3.5.3 o posterior admite las API asincrónicas; se recomienda usar la versión 3.7 o posterior.

### <a name="net-sdk"></a>.NET SDK

El [SDK de dispositivo Azure IoT Hub para .NET (C#)](https://github.com/Azure/azure-iot-sdk-csharp) admite la siguiente configuración y se prueba con ella.

| SO                                   | Estándar                                                   |
|--------------------------------------|------------------------------------------------------------|
| Linux                                | .NET Core 2.1                                              |
| SKU de Windows 10 Desktop y Server   | .NET Core 2.1, .NET Framework 4.5.1 o .NET Framework 4.7 |

El SDK de .NET también se puede usar con Windows IoT Core con el [Azure Device Agent](https://github.com/ms-iot/azure-client-tools/blob/master/docs/device-agent/device-agent.md) o [un NTService personalizado que puede usar RPC para comunicarse con aplicaciones de UWP](https://docs.microsoft.com/samples/microsoft/windows-iotcore-samples/ntservice-rpc/).

### <a name="nodejs-sdk"></a>SDK de Node.js

El [SDK de dispositivo Azure IoT Hub para Node.js](https://github.com/Azure/azure-iot-sdk-node) admite la siguiente configuración y se prueba con ella.

| SO                  | Versión de Node    |
|---------------------|-----------------|
| Linux               | LTS y Current |
| Familia de productos de Windows 10   | LTS y Current |

### <a name="java-sdk"></a>SDK de Java

El [SDK de dispositivo Azure IoT Hub para Java](https://github.com/Azure/azure-iot-sdk-java) admite la siguiente configuración y se prueba con ella.

| SO                     | Versión de Java |
|------------------------|--------------|
| Android API 28         | Java 8       |
| Linux x64             | Java 8       |
| Familia de productos de Windows 10 x64  | Java 8       |

## <a name="partner-supported-development-kits"></a>Kits de desarrollo compatibles para asociados

Microsoft trabaja con diversos asociados para proporcionar kits de desarrollo para varias arquitecturas de microprocesadores. Estos asociados han migrado el SDK de IoT de Azure para C a su plataforma. Los asociados crean y mantienen la capa de abstracción de plataforma (PAL) del SDK. Microsoft trabaja con estos asociados para proporcionar soporte técnico ampliado.

| Asociado             | Dispositivos                            | Vínculo                     | Soporte técnico |
|---------------------|------------------------------------|--------------------------|---------|
| Espressif           | ESP32 <br/> ESP8266                              | [Esp-azure](https://github.com/espressif/esp-azure)                | [GitHub](https://github.com/espressif/esp-azure)  
| Qualcomm            | Qualcomm MDM9206 LTE IoT Modem     | [Qualcomm LTE for IoT SDK](https://developer.qualcomm.com/software/lte-iot-sdk) | [Foro](https://developer.qualcomm.com/forums/software/lte-iot-sdk)   |
| ST Microelectronics | STM32L4 Series <br/> STM32F4 Series <br/>  STM32F7 Series <br/>  STM32L4 Discovery Kit for IoT node    | [X-CUBE-AZURE](https://www.st.com/en/embedded-software/x-cube-azure.html) <br/>  <br/> [P-NUCLEO-AZURE](https://www.st.com/content/st_com/en/products/evaluation-tools/solution-evaluation-tools/communication-and-connectivity-solution-eval-boards/p-nucleo-azure1.html) <br/> [FP-CLD-AZURE](https://www.st.com/content/st_com/en/products/embedded-software/mcus-embedded-software/stm32-embedded-software/stm32-ode-function-pack-sw/fp-cld-azure1.html)            | [Soporte técnico](https://www.st.com/content/st_com/en/support/support-home.html)
| Texas Instruments   | CC3220SF LaunchPad </br> CC3220S LaunchPad </br> CC3235SF LaunchPad </br> CC3235S LaunchPad </br> MSP432E4 LaunchPad | [Complemento de Azure IoT para SimpleLink](https://github.com/TexasInstruments/azure-iot-pal-simplelink) | [TI E2E Forum](https://e2e.ti.com) <br/> [TI E2E Forum for CC3220](https://e2e.ti.com/support/wireless_connectivity/simplelink_wifi_cc31xx_cc32xx/) <br/> [TI E2E Forum for MSP432E4](https://e2e.ti.com/support/microcontrollers/msp430/) |

## <a name="porting-the-microsoft-azure-iot-c-sdk"></a>Portabilidad del SDK de IoT de Microsoft Azure para C

Si la plataforma del dispositivo no está incluida en una de las secciones anteriores, puede considerar la posibilidad de portar el SDK de IoT de Azure para C. La portabilidad del SDK de C conlleva principalmente la implementación de la capa de abstracción de plataforma (PAL) del SDK. La PAL define primitivas que proporcionan el nexo de unión entre el dispositivo y las funciones de nivel superior del SDK. Para más información, consulte la [guía de portabilidad](https://github.com/Azure/azure-c-shared-utility/blob/master/devdoc/porting_guide.md).

## <a name="microsoft-partners-and-certified-azure-iot-devices"></a>Asociados de Microsoft y dispositivos IoT de Azure certificados

Microsoft trabaja con varios asociados para ampliar continuamente el universo de IoT de Azure con dispositivos certificados y probados.

* Para examinar los dispositivos IoT de Azure certificados, consulte [Catálogo de dispositivos Microsoft Azure Certified for IoT](https://catalog.azureiotsolutions.com/).

* Para más información sobre el ecosistema Azure Certified for IoT, consulte [Unirse al ecosistema Certified for IoT](https://catalog.azureiotsolutions.com/register).

## <a name="connecting-to-iot-hub-without-an-sdk"></a>Conexión a IoT Hub sin un SDK

Si no puede usar alguno de los SDK de dispositivo IoT Hub, puede conectarse directamente a IoT Hub mediante las [API REST de IoT Hub](https://docs.microsoft.com/rest/api/iothub/) desde cualquier aplicación capaz de enviar y recibir solicitudes y respuestas HTTPS.

## <a name="support-and-other-resources"></a>Compatibilidad y otros recursos

Si tiene problemas al usar los SDK de dispositivo IoT de Azure, hay varias maneras de buscar soporte técnico que se resumen a continuación. Para obtener información completa sobre todas las opciones de soporte técnico, consulte [Opciones de ayuda y soporte técnico de IoT de Azure](https://aka.ms/iothelp). 

**Informes de errores**: se pueden notificar errores en los SDK de dispositivos en la página de problemas del proyecto de GitHub pertinente. Las correcciones pasan pronto del proyecto a las actualizaciones de producto.

* [Problemas del SDK de Azure IoT Hub para C](https://github.com/Azure/azure-iot-sdk-c/issues)

* [Problemas del SDK de Azure IoT Hub para .NET (C#)](https://github.com/Azure/azure-iot-sdk-csharp/issues)

* [Problemas del SDK de Azure IoT Hub para Java](https://github.com/Azure/azure-iot-sdk-java/issues)

* [Problemas del SDK de Azure IoT Hub para Node.js](https://github.com/Azure/azure-iot-sdk-node/issues)

* [Problemas del SDK de Azure IoT Hub para Python](https://github.com/Azure/azure-iot-sdk-python/issues)

**Preguntas técnicas**: puede hacer preguntas técnicas en [Microsoft Q&A](https://docs.microsoft.com/answers/topics/azure-iot-sdk.html) y [Stack Overflow](https://stackoverflow.com/questions/tagged/azure-iot-sdk) mediante la etiqueta *azure-iot-sdk*.

**Equipo de soporte técnico al cliente de Microsoft**: los usuarios que dispongan de un [plan de soporte técnico](https://azure.microsoft.com/support/plans/) pueden ponerse en contacto con este equipo creando una solicitud de soporte técnico directamente en [Azure Portal](https://ms.portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/overview).

**Solicitudes de características**: el seguimiento de las solicitudes de características de IoT de Azure se realiza a través de la [página UserVoice](https://feedback.azure.com/forums/321918-azure-iot) del producto.

## <a name="next-steps"></a>Pasos siguientes

* [SDK de dispositivos y servicios](iot-hub-devguide-sdks.md)
* [Porting Guidance](https://github.com/Azure/azure-c-shared-utility/blob/master/devdoc/porting_guide.md) (Guía de migración)
