---
title: Recuperación de un mensaje de Twitter con Azure Functions | Microsoft Docs
description: Uso del sensor de movimiento para detectar el gesto de agitar y uso de Azure Functions para buscar un tweet aleatorio con un hashtag determinado
author: liydu
manager: jeffya
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.tgt_pltfrm: arduino
ms.date: 03/07/2018
ms.author: liydu
ms.custom: devx-track-csharp
ms.openlocfilehash: f0aa71b34818cf373d1bb58531ee5c68c8d3d5ec
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89004330"
---
# <a name="shake-shake-for-a-tweet----retrieve-a-twitter-message-with-azure-functions"></a>Agite para un Tweet: recuperar un mensaje de Twitter con Azure Functions

En este proyecto, aprenderá a usar el sensor de movimiento para desencadenar un evento mediante Azure Functions. La aplicación recupera un tweet aleatorio con el #hashtag que configure en el boceto de Arduino. El tweet se muestra en la pantalla del DevKit.

## <a name="what-you-need"></a>Lo que necesita

Siga la [Guía de introducción](https://docs.microsoft.com/azure/iot-hub/iot-hub-arduino-iot-devkit-az3166-get-started) para:

* Conecte el DevKit a la red Wi-Fi.
* Prepare el entorno de desarrollo.

Una suscripción de Azure activa. Si no tiene una, puede registrarse a través de uno de estos métodos:

* Active una [cuenta de Microsoft Azure de prueba de 30 días gratis](https://azure.microsoft.com/free/)
* Puede reclamar su [crédito de Azure](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details/) si tiene una suscripción a MSDN o Visual Studio.

## <a name="open-the-project-folder"></a>Apertura de la carpeta del proyecto

Primero, abra la carpeta del proyecto. 

### <a name="start-vs-code"></a>Inicio de VS Code

* Asegúrese de que el DevKit está conectado al equipo.

* Inicie VS Code.

* Conecte el DevKit al equipo.

   > [!NOTE]
   > Al iniciar VS Code, podría recibir un mensaje de error indicando que el IDE de Arduino o el paquete de placa relacionado no se encuentra. Si se produce este error, cierre VS Code y vuelva a iniciar el IDE de Arduino. Ahora VS Code debe encontrar la ruta de acceso del IDE de Arduino correctamente.

### <a name="open-the-arduino-examples-folder"></a>Apertura de la carpeta de ejemplos de Arduino

Expanda la sección de la izquierda **EJEMPLOS DE ARDUINO**, vaya a **Ejemplos para MXCHIP AZ3166 > AzureIoT** y seleccione **ShakeShake**. Se abre una nueva ventana de VS Code que muestra la carpeta del proyecto. Si no ve la sección MXCHIP AZ3166, asegúrese de que el dispositivo está conectado correctamente y reinicie Visual Studio Code.  
![mini-solution-examples](media/iot-hub-arduino-iot-devkit-az3166-retrieve-twitter-message/vscode_examples.png)

También puede abrir el proyecto de ejemplo desde la paleta de comandos. Haga clic en `Ctrl+Shift+P` (macOS: `Cmd+Shift+P`) para abrir la paleta de comandos, escriba **Arduino** y, después, busque y seleccione **Arduino: Ejemplos**.

## <a name="provision-azure-services"></a>Aprovisionamiento de los servicios de Azure

En la ventana de la solución, ejecute la tarea mediante `Ctrl+P` (macOS: `Cmd+P`) escribiendo `task cloud-provision`.

En el terminal de VS Code, una línea de comandos interactiva le guía durante el aprovisionamiento de los servicios de Azure necesarios:

![Aprovisionamiento en la nube](media/iot-hub-arduino-iot-devkit-az3166-retrieve-twitter-message/cloud-provision.png)

> [!NOTE]
> Si la página se bloquea en el estado de carga al intentar iniciar sesión en Azure, consulte el [paso sobre el bloqueo de la página de inicio en las preguntas frecuentes del DevKit de IoT](https://microsoft.github.io/azure-iot-developer-kit/docs/faq/#page-hangs-when-log-in-azure).
 
## <a name="modify-the-hashtag"></a>Modificación del #hashtag

Abra `ShakeShake.ino` y busque esta línea de código:

```cpp
static const char* iot_event = "{\"topic\":\"iot\"}";
```

Reemplace la cadena `iot` dentro de las llaves por su hashtag preferido. Más adelante, el DevKit recuperará un tweet aleatorio que incluye el hashtag que se especifique en este paso.

## <a name="deploy-azure-functions"></a>Implementación de Azure Functions

Use `Ctrl+P` (macOS: `Cmd+P`) para ejecutar `task cloud-deploy` para comenzar la implementación del código de Azure Functions:

![implementación en la nube](media/iot-hub-arduino-iot-devkit-az3166-retrieve-twitter-message/cloud-deploy.png)

> [!NOTE]
> En ocasiones, la función de Azure podría no funcionar correctamente. Para resolver este problema cuando se produzca, consulte la sección de ["error de compilación" de las preguntas frecuentes del DevKit de IoT](https://microsoft.github.io/azure-iot-developer-kit/docs/faq/#compilation-error-for-azure-function).

## <a name="build-and-upload-the-device-code"></a>Creación y carga del código del dispositivo

A continuación, cree y cargue el código del dispositivo.

### <a name="windows"></a>Windows

1. Use `Ctrl+P` para ejecutar `task device-upload`.

2. El terminal le pide que especifique el modo de configuración. Para ello:

   * Mantenga presionado el botón A

   * Presione y suelte el botón de restablecimiento.

3. La pantalla muestra el identificador de DevKit y "Configuration" (Configuración).

### <a name="macos"></a>macOS

1. Ponga el DevKit en modo de configuración:

   Mantenga presionado el botón A, y presione y suelte el botón de restablecimiento. La pantalla muestra "Configuration" (Configuración).

2. Use `Cmd+P` para ejecutar `task device-upload` y establecer la cadena de conexión que se recuperó en el paso `task cloud-provision`.

### <a name="verify-upload-and-run"></a>Comprobación, carga y ejecución

Ahora que la cadena de conexión está establecida, se comprueba y se carga la aplicación y se ejecuta. 

1. VS Code inicia la comprobación y la carga del boceto de Arduino en el DevKit:

   ![carga-dispositivo](media/iot-hub-arduino-iot-devkit-az3166-retrieve-twitter-message/device-upload.png)

2. El DevKit se reinicia y comienza a ejecutar el código.

Si aparece el mensaje de error "Error: AZ3166: paquete desconocido". Este error se produce cuando el índice del paquete de la placa no se actualiza correctamente. Para resolver este problema, consulte el [error "paquete desconocido" en las preguntas frecuentes del DevKit de IoT](https://microsoft.github.io/azure-iot-developer-kit/docs/faq/#development).

## <a name="test-the-project"></a>Prueba del proyecto

Después de la inicialización de la aplicación, haga clic y suelte el botón A y, a continuación, agite la placa del DevKit. Esta acción recupera un tweet aleatorio, que contiene el hashtag especificado anteriormente. En unos segundos, se muestra un tweet en la pantalla del DevKit:

### <a name="arduino-application-initializing"></a>Arduino application initializing... (Aplicación de Arduino inicializando...)

![Aplicación de Arduino inicializando](media/iot-hub-arduino-iot-devkit-az3166-retrieve-twitter-message/result-1.png)

### <a name="press-a-to-shake"></a>Press A to shake... (Presione A para agitar...)

![Presione A para agitar](media/iot-hub-arduino-iot-devkit-az3166-retrieve-twitter-message/result-2.png)

### <a name="ready-to-shake"></a>Ready to shake... (Listo para agitar...)

![Listo para agitar](media/iot-hub-arduino-iot-devkit-az3166-retrieve-twitter-message/result-3.png)

### <a name="processing"></a>Processing... (Procesando...)

![Processing](media/iot-hub-arduino-iot-devkit-az3166-retrieve-twitter-message/result-4.png)

### <a name="press-b-to-read"></a>Press B to read... (Presione B para leer...)

![Presione B para leer](media/iot-hub-arduino-iot-devkit-az3166-retrieve-twitter-message/result-5.png)

### <a name="display-a-random-tweet"></a>Display a random tweet... (Mostrar un tweet aleatorio...)

![Mostrar un tweet aleatorio](media/iot-hub-arduino-iot-devkit-az3166-retrieve-twitter-message/result-6.png)

- Presione de nuevo el botón A y, a continuación, agite para obtener un nuevo tweet.
- Presione el botón B para desplazarse por el resto del tweet.

## <a name="how-it-works"></a>Funcionamiento

![diagrama](media/iot-hub-arduino-iot-devkit-az3166-retrieve-twitter-message/diagram.png)

El boceto de Arduino envía un evento a Azure IoT Hub. Este evento desencadena la aplicación de Azure Functions. La aplicación de Azure Functions contiene la lógica para conectar con la API de Twitter y recuperar un tweet. A continuación, encapsula el texto del tweet en un mensaje C2D (de la nube al dispositivo) y lo envía de vuelta al dispositivo.

## <a name="optional-use-your-own-twitter-bearer-token"></a>Opcional: use su propio token de portador de Twitter

Para realizar pruebas, este proyecto de ejemplo utiliza un token de portador de Twitter configurado previamente. Sin embargo, hay un [límite de velocidad](https://dev.twitter.com/rest/reference/get/search/tweets) para cada cuenta de Twitter. Si desea usar su propio token, siga estos pasos:

1. Vaya al [portal para desarrolladores de Twitter](https://dev.twitter.com/) para registrar una nueva aplicación de Twitter.

2. [Obtenga la clave del consumidor y el secreto del consumidor](https://support.yapsody.com/hc/en-us/articles/360003291573-How-do-I-get-a-Twitter-Consumer-Key-and-Consumer-Secret-key-) de la aplicación.

3. Use [alguna utilidad](https://gearside.com/nebula/utilities/twitter-bearer-token-generator/) para generar un token de portador de Twitter a partir de estas dos claves.

4. En [Azure Portal](https://portal.azure.com/){:target="_blank"}, vaya a **Grupo de recursos** y busque la función de Azure (tipo: App Service) del proyecto "Shake, Shake". El nombre siempre contiene la cadena "shake...".

   ![azure-function](media/iot-hub-arduino-iot-devkit-az3166-retrieve-twitter-message/azure-function.png)

5. Actualice el código para `run.csx` en **Funciones > shakeshake-cs** con su propio token:

   ```csharp
   string authHeader = "Bearer " + "[your own token]";
   ```
  
   ![token de twitter](media/iot-hub-arduino-iot-devkit-az3166-retrieve-twitter-message/twitter-token.png)

6. Guarde el archivo y haga clic en **Ejecutar**.

## <a name="problems-and-feedback"></a>Problemas y comentarios

Como solucionar problemas o proporcionar comentarios. 

### <a name="problems"></a>Problemas

Un problema que puede ver si la pantalla muestra "No hay tweets" aunque cada paso se ha ejecutado correctamente. Esta condición se produce normalmente la primera vez que se implementa y ejecuta el ejemplo, porque la aplicación de función requiere desde un par de segundos hasta un minuto para el arranque en frío de la aplicación. 

O bien, cuando se ejecuta el código, hay algunas señales que provocan un reinicio de la aplicación. Cuando se produce esta situación, la aplicación del dispositivo puede obtener un tiempo de expiración agotado para capturar el tweet. En este caso, puede intentar uno o ambos de estos métodos para resolver este problema:

1. Haga clic en el botón Restablecer en el DevKit para volver a ejecutar la aplicación del dispositivo.

2. En [Azure Portal](https://portal.azure.com/), busque la aplicación de Azure Functions que creó y reiníciela:

   ![azure-function-restart](media/iot-hub-arduino-iot-devkit-az3166-retrieve-twitter-message/azure-function-restart.png)

Si se encuentra otros problemas, puede consultar las [preguntas más frecuentes del DevKit de IoT](https://microsoft.github.io/azure-iot-developer-kit/docs/faq/) o también puede ponerse en contacto con nosotros mediante los siguientes canales:

* [Gitter.im](https://gitter.im/Microsoft/azure-iot-developer-kit)
* [Stack Overflow](https://stackoverflow.com/questions/tagged/iot-devkit)

## <a name="next-steps"></a>Pasos siguientes

Ahora que ha aprendido cómo conectar un dispositivo DevKit al acelerador de la solución de supervisión remota de Azure IoT y recuperar un tweet, estos son los siguientes pasos sugeridos:

* [Información general sobre el acelerador de la solución de supervisión remota de Azure IoT](https://docs.microsoft.com/azure/iot-suite/)