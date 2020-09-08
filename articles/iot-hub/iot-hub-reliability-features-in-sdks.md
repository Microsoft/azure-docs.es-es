---
title: Administración de la conectividad de IoT Hub y la mensajería confiable con los SDK de dispositivo
description: Obtenga información sobre cómo mejorar la conectividad de dispositivos y la mensajería al usar los SDK de dispositivo de Azure IoT Hub
services: iot-hub
author: robinsh
ms.author: robinsh
ms.date: 07/07/2018
ms.topic: article
ms.service: iot-hub
ms.custom:
- amqp
- mqtt
- devx-track-csharp
ms.openlocfilehash: 8f1b1131ab634cdc5418773b8a2e18c2195ffc18
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89014564"
---
# <a name="manage-connectivity-and-reliable-messaging-by-using-azure-iot-hub-device-sdks"></a>Administrar la conectividad y la mensajería confiable mediante los SDK de dispositivo de Azure IoT Hub

En este artículo se proporciona orientación de alto nivel para ayudarle a diseñar aplicaciones más resistentes para sus dispositivos. Igualmente muestra cómo aprovechar las ventajas de la conectividad y las características de mensajería confiable en el SDK de dispositivo IoT de Azure. El objetivo de esta guía es ayudarle a administrar los siguientes escenarios:

* Corregir una conexión de red descartada.

* Cambiar entre distintas conexiones de red.

* Volver a conectar debido a errores de conexión transitorios del servicio.

Los detalles de implementación pueden variar según el lenguaje. Para obtener más información, consulte la documentación de la API o el SDK específico:

* [SDK de C/iOS](https://github.com/azure/azure-iot-sdk-c)

* [SDK de .NET](https://github.com/Azure/azure-iot-sdk-csharp/blob/master/iothub/device/devdoc/retrypolicy.md)

* [SDK de Java](https://github.com/Azure/azure-iot-sdk-java/blob/master/device/iot-device-client/devdoc/requirement_docs/com/microsoft/azure/iothub/retryPolicy.md)

* [SDK de Node](https://github.com/Azure/azure-iot-sdk-node/wiki/Connectivity-and-Retries#types-of-errors-and-how-to-detect-them)

* [SDK de Python](https://github.com/Azure/azure-iot-sdk-python) (confiabilidad no implementada aún)

## <a name="designing-for-resiliency"></a>Diseño para lograr resistencia

A menudo, los dispositivos IoT se basan en conexiones de red no continuas o inestables, como GSM o las conexiones por satélite. Los errores suceden cuando los dispositivos interactúan con los servicios basados en la nube debido a la disponibilidad intermitente del servicio y los errores transitorios o de nivel de infraestructura. Una aplicación que se ejecuta en un dispositivo debe administrar los mecanismos de conexión y reconexión, así como la lógica de reintentos para enviar o recibir mensajes. Además, los requisitos de la estrategia de reintento dependen en gran medida del escenario, el contexto y las capacidades de IoT del dispositivo.

El objetivo de los SDK de los dispositivos Azure IoT Hub es simplificar la conexión y la comunicación de la nube a los dispositivos y de los dispositivos a la nube. Estos SDK proporcionan una manera sólida de conectarse a Azure IoT Hub y un conjunto completo de opciones para enviar y recibir mensajes. Los desarrolladores también pueden modificar la implementación existente para personalizar mejor la estrategia de reintento adecuada para un escenario determinado.

Las características pertinentes del SDK que admiten la conectividad y la mensajería de confianza se tratan en las secciones siguientes.

## <a name="connection-and-retry"></a>Conexión y reintento

En esta sección se proporciona información general sobre los patrones de reconexión y reintento disponibles al administrar conexiones. Además detalla la guía de implementación para usar una directiva de reintentos diferente en la aplicación del dispositivo y enumera las API relevantes de los SDK del dispositivo.

### <a name="error-patterns"></a>Patrones de error

Los errores de conexión pueden producirse en muchos niveles:

* Errores de red: un socket desconectado y errores de resolución de nombres.

* Errores a nivel de protocolo para el transporte de HTTP, AMQP y MQTT: vínculos desasociados o sesiones expiradas.

* Errores a nivel de aplicación que se generan a partir de errores locales: credenciales no válidas o un comportamiento de servicio, como exceder una cuota o una limitación.

Los SDK de dispositivo detectan errores en los tres niveles. Los errores relacionados con el sistema operativo y los errores de hardware no se detectan ni se controlan mediante los SDK de dispositivo. El SDK se basa en [la guía para controlar errores transitorios](/azure/architecture/best-practices/transient-faults#general-guidelines) del Centro de arquitectura de Azure.

### <a name="retry-patterns"></a>Patrones de reintentos

Los pasos siguientes describen el proceso de reintento cuando se detectan errores de conexión:

1. El SDK detecta el error y el error asociado en la red, el protocolo o la aplicación.

2. El SDK usa el filtro de error para determinar el tipo de error y decidir si es necesario un reintento.

3. Si el SDK identifica un **error irrecuperable**, operaciones como la conexión, el envío y la recepción se detienen. El SDK notifica al usuario. Un error de autenticación y un error de punto de conexión incorrecta son ejemplos de errores irrecuperables.

4. Si el SDK identifica un **error recuperable**, vuelve a intentar el proceso según la directiva de reintentos especificada hasta que transcurra el tiempo de espera definido.  Tenga en cuenta que el SDK usa de manera predeterminada la directiva de reintentos **Retroceso exponencial con vibración**.
5. Cuando el tiempo de espera definido expira, el SDK deja de tratar de conectarse o de enviar el contenido. Notifica al usuario.

6. El SDK permite que el usuario adjunte una devolución de llamada para recibir los cambios de estado de la conexión.

Los SDK proporcionan tres directivas de reintentos:

* **Retroceso exponencial con vibración**: esta directiva de reintentos predeterminada tiende a ser agresiva al principio y se ralentiza poco a poco hasta que se alcanza un retraso máximo. El diseño se basa en la [guía de reintentos de Azure Architecture Center](https://docs.microsoft.com/azure/architecture/best-practices/retry-service-specific). 

* **Reintento personalizado**: para algunos idiomas del SDK, puede diseñar una directiva de reintentos personalizada que sea más adecuada para su escenario y luego agregarla en RetryPolicy. El reintento personalizado no está disponible en el SDK de C y no se admite actualmente en el SDK de Python. El SDK de Python se vuelve a conectar según sea necesario.

* **Sin reintentos**: puede establecer la directiva de reintentos en "sin reintentos" para deshabilitar la lógica de reintentos. El SDK intenta conectarse una vez y envía un mensaje una vez, suponiendo que se establece la conexión. Esta directiva se usa normalmente en escenarios con problemas de ancho de banda o de costos. Si elige esta opción, tenga en cuenta que los mensajes que no se envíen se perderán y no se podrán recuperar.

### <a name="retry-policy-apis"></a>API de directiva de reintentos

   | SDK | Método SetRetryPolicy | Implementaciones de directivas | Guía de implementación |
   |-----|----------------------|--|--|
   |  C/iOS  | [IOTHUB_CLIENT_RESULT IoTHubClient_SetRetryPolicy](https://github.com/Azure/azure-iot-sdk-c/blob/2018-05-04/iothub_client/inc/iothub_client.h#L188)        | **Valor predeterminado**: [IOTHUB_CLIENT_RETRY_EXPONENTIAL_BACKOFF](https://github.com/Azure/azure-iot-sdk-c/blob/master/doc/connection_and_messaging_reliability.md#connection-retry-policies)<BR>**Personalizado:** use la [retryPolicy](https://github.com/Azure/azure-iot-sdk-c/blob/master/doc/connection_and_messaging_reliability.md#connection-retry-policies) disponible<BR>**Sin reintento**: [IOTHUB_CLIENT_RETRY_NONE](https://github.com/Azure/azure-iot-sdk-c/blob/master/doc/connection_and_messaging_reliability.md#connection-retry-policies)  | [Implementación de C/iOS](https://github.com/Azure/azure-iot-sdk-c/blob/master/doc/connection_and_messaging_reliability.md#)  |
   | Java| [SetRetryPolicy](https://docs.microsoft.com/java/api/com.microsoft.azure.sdk.iot.device.deviceclientconfig.setretrypolicy?view=azure-java-stable)        | **Valor predeterminado**: [ExponentialBackoffWithJitter class](https://github.com/Azure/azure-iot-sdk-java/blob/master/device/iot-device-client/src/main/java/com/microsoft/azure/sdk/iot/device/transport/NoRetry.java)<BR>**Personalizado:** implemente la [interfaz RetryPolicy](https://github.com/Azure/azure-iot-sdk-java/blob/master/device/iot-device-client/src/main/java/com/microsoft/azure/sdk/iot/device/transport/RetryPolicy.java)<BR>**Sin reintento**: [clase NoRetry](https://github.com/Azure/azure-iot-sdk-java/blob/master/device/iot-device-client/src/main/java/com/microsoft/azure/sdk/iot/device/transport/NoRetry.java)  | [Implementación de Java](https://github.com/Azure/azure-iot-sdk-java/blob/master/device/iot-device-client/devdoc/requirement_docs/com/microsoft/azure/iothub/retryPolicy.md) |
   | .NET| [DeviceClient.SetRetryPolicy](/dotnet/api/microsoft.azure.devices.client.deviceclient.setretrypolicy?view=azure-dotnet) | **Valor predeterminado**: [ExponentialBackoff class](/dotnet/api/microsoft.azure.devices.client.exponentialbackoff?view=azure-dotnet)<BR>**Personalizado:** implemente la [interfaz IRetryPolicy](https://docs.microsoft.com/dotnet/api/microsoft.azure.devices.client.iretrypolicy?view=azure-dotnet)<BR>**Sin reintento**: [clase NoRetry](/dotnet/api/microsoft.azure.devices.client.noretry?view=azure-dotnet) | [Implementación de C#](https://github.com/Azure/azure-iot-sdk-csharp) | |
   | Nodo| [setRetryPolicy](/javascript/api/azure-iot-device/client?view=azure-iot-typescript-latest) | **Valor predeterminado**: [ExponentialBackoffWithJitter class](https://github.com/Azure/azure-iot-sdk-java/blob/master/device/iot-device-client/src/main/java/com/microsoft/azure/sdk/iot/device/transport/NoRetry.java) | [Implementación de Node](https://github.com/Azure/azure-iot-sdk-node/wiki/Connectivity-and-Retries#types-of-errors-and-how-to-detect-them) |
   | Python| No se admite actualmente. | No se admite actualmente. | No se admite actualmente. |

Estos ejemplos de código ilustran este flujo:

#### <a name="net-implementation-guidance"></a>Guía de implementación de .NET

En el ejemplo de código siguiente se muestra cómo definir y establecer la directiva de reintentos predeterminada:

   ```csharp
   // define/set default retry policy
   IRetryPolicy retryPolicy = new ExponentialBackoff(int.MaxValue, TimeSpan.FromMilliseconds(100), TimeSpan.FromSeconds(10), TimeSpan.FromMilliseconds(100));
   SetRetryPolicy(retryPolicy);
   ```

Para evitar el uso elevado de la CPU, se limitan los reintentos si el código produce un error inmediatamente. Por ejemplo, cuando no hay ninguna red o ruta al destino. El tiempo mínimo para ejecutar el siguiente reintento es de 1 segundo.

Si el servicio responde con un error de limitación, es que la directiva de reintentos es diferente y no se puede cambiar a través de la API pública:

   ```csharp
   // throttled retry policy
   IRetryPolicy retryPolicy = new ExponentialBackoff(RetryCount, TimeSpan.FromSeconds(10), 
     TimeSpan.FromSeconds(60), TimeSpan.FromSeconds(5)); SetRetryPolicy(retryPolicy);
   ```

El mecanismo de reintentos se detendrá después de `DefaultOperationTimeoutInMilliseconds`, valor que actualmente está establecido en 4 minutos.

#### <a name="other-languages-implementation-guidance"></a>Guía de implementación de otros lenguajes

Para obtener ejemplos de código en otros lenguajes, revise los siguientes documentos para la implementación. El repositorio contiene ejemplos que muestran el uso de las API de la directiva de reintentos.

* [SDK de C/iOS](https://github.com/azure/azure-iot-sdk-c)

* [SDK de .NET](https://github.com/Azure/azure-iot-sdk-csharp/blob/master/iothub/device/devdoc/retrypolicy.md)

* [SDK de Java](https://github.com/Azure/azure-iot-sdk-java/blob/master/device/iot-device-client/devdoc/requirement_docs/com/microsoft/azure/iothub/retryPolicy.md)

* [SDK de Node](https://github.com/Azure/azure-iot-sdk-node/wiki/Connectivity-and-Retries#types-of-errors-and-how-to-detect-them)

* [SDK de Python](https://github.com/Azure/azure-iot-sdk-python) (confiabilidad no implementada aún)

## <a name="next-steps"></a>Pasos siguientes

* [Uso de los SDK de dispositivos y servicios](./iot-hub-devguide-sdks.md)

* [Uso del SDK de dispositivo IoT para C](./iot-hub-device-sdk-c-intro.md)

* [Desarrollo para dispositivos restringidos](./iot-hub-devguide-develop-for-constrained-devices.md)

* [Desarrollo para dispositivos móviles](./iot-hub-how-to-develop-for-mobile-devices.md)

* [Solución de problemas de desconexión de dispositivos](iot-hub-troubleshoot-connectivity.md)
