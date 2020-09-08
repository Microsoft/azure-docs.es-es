---
title: Crear un traductor con Azure Functions, Cognitive Services, IoT DevKit
description: Use el micrófono de IoT DevKit para recibir un mensaje de voz y luego use Azure Cognitive Services para procesarlo en texto traducido al inglés.
author: liydu
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.tgt_pltfrm: arduino
ms.date: 12/19/2018
ms.author: liydu
ms.custom: devx-track-csharp
ms.openlocfilehash: 412a3e78006d263858ff0e28af52cf11bf44c7bb
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89004347"
---
# <a name="use-iot-devkit-az3166-with-azure-functions-and-cognitive-services-to-make-a-language-translator"></a>Uso de IoT DevKit AZ3166 con Azure Functions y Cognitive Services para crear un traductor de idiomas

En este artículo, aprenderá a convertir IoT DevKit en un traductor de idiomas mediante [Azure Cognitive Services](https://azure.microsoft.com/services/cognitive-services/). Registra la voz y la traduce a texto en inglés que se muestra en la pantalla de DevKit.

[MXChip IoT DevKit](https://aka.ms/iot-devkit) es una placa integral compatible de Arduino con periféricos y sensores varios. Puede desarrollar con [Azure IoT Device Workbench](https://aka.ms/iot-workbench) o el paquete de extensión [Azure IoT Tools](https://aka.ms/azure-iot-tools) en Visual Studio Code. El [catálogo de proyectos](https://microsoft.github.io/azure-iot-developer-kit/docs/projects/) contiene aplicaciones de ejemplo para ayudarlo a crear prototipos de las soluciones de IoT.

## <a name="before-you-begin"></a>Antes de empezar

Para completar los pasos de este tutorial, deberá realizar las siguientes tareas:

* Prepare su DevKit siguiendo los pasos descritos en [Conexión de IoT DevKit AZ3166 a Azure IoT Hub en la nube](/azure/iot-hub/iot-hub-arduino-iot-devkit-az3166-get-started).

## <a name="create-azure-cognitive-service"></a>Creación de Azure Cognitive Service

1. En Azure Portal, haga clic en **Crear un recurso** y busque **Voz**. Rellene el formulario para crear el servicio de voz.
  ![Servicio Voz](media/iot-hub-arduino-iot-devkit-az3166-translator/speech-service.png)

1. Vaya al servicio de voz que acaba de crear, haga clic en la sección **Claves** para copiar y anotar el valor de **Key1** para que DevKit pueda obtener acceso.
  ![Copia de claves](media/iot-hub-arduino-iot-devkit-az3166-translator/copy-keys.png)

## <a name="open-sample-project"></a>Abra el proyecto de ejemplo

1. Asegúrese de que IoT DevKit **no está conectado** al equipo. Primero, inicie VS Code y luego conecte DevKit al equipo.

1. Haga clic en `F1` para abrir la paleta de comandos, escriba y seleccione **Azure IoT Device Workbench: Open Examples...** (Abrir ejemplos...). A continuación, seleccione **IoT DevKit** como placa.

1. En la página de ejemplos de IoT Workbench, busque **DevKit Translator** (Traductor de DevKit) y haga clic en **Open Sample** (Abrir ejemplo). A continuación, seleccione la ruta de acceso predeterminada para descargar el código de ejemplo.
  ![Abrir ejemplo](media/iot-hub-arduino-iot-devkit-az3166-translator/open-sample.png)

## <a name="use-speech-service-with-azure-functions"></a>Usar el servicio de voz con Azure Functions

1. En VS Code, haga clic en `F1`, escriba y seleccione **Azure IoT Device Workbench: Provision Azure Services...** (Aprovisionar servicios de Azure). ![Aprovisionamiento de servicios de Azure](media/iot-hub-arduino-iot-devkit-az3166-translator/provision.png)

1. Siga los pasos para finalizar el aprovisionamiento de Azure IoT Hub y Azure Functions.
   ![Pasos de aprovisionamiento](media/iot-hub-arduino-iot-devkit-az3166-translator/provision-steps.png)

   Anote el nombre de dispositivo de Azure IoT Hub que creó.

1. Abra `Functions\DevKitTranslatorFunction.cs` y actualice las siguientes líneas de código con el nombre de dispositivo y la clave del servicio de Voz que anotó.
   ```csharp
   // Subscription Key of Speech Service
   const string speechSubscriptionKey = "";

   // Region of the speech service, see https://docs.microsoft.com/azure/cognitive-services/speech-service/regions for more details.
   const string speechServiceRegion = "";

   // Device ID
   const string deviceName = "";
   ```

1. Presione `F1`, escriba y seleccione **Azure IoT Device Workbench: Deploy to Azure...** (Implementar en Azure...) Si VS Code pide confirmación para la reimplementación, haga clic en **Sí**.
   ![Advertencia de implementación](media/iot-hub-arduino-iot-devkit-az3166-translator/deploy-warning.png)

1. Asegúrese de que la implementación se ha realizado correctamente.
   ![Implementación correcta](media/iot-hub-arduino-iot-devkit-az3166-translator/deploy-success.png)

1. En Azure Portal, vaya a la sección de **Functions Apps**, busque la instancia de Azure Function App que acaba de crear. Haga clic en `devkit_translator` y luego en **</> Obtener la dirección URL de la función** para copiar la dirección URL.
   ![Copiar la dirección URL de la función](media/iot-hub-arduino-iot-devkit-az3166-translator/get-function-url.png)

1. Pegue la dirección URL en el archivo `azure_config.h`.
   ![Configuración de Azure](media/iot-hub-arduino-iot-devkit-az3166-translator/azure-config.png)

   > [!NOTE]
   > Si Function App no funciona correctamente, consulte la sección de [Preguntas frecuentes](https://microsoft.github.io/azure-iot-developer-kit/docs/faq#compilation-error-for-azure-function) para resolver el problema.

## <a name="build-and-upload-device-code"></a>Creación y carga del código de dispositivo

1. Cambie DevKit al **modo de configuración**:
   * Mantenga presionado el botón **A**.
   * Presione y suelte el botón **Restablecer**.

   La pantalla mostrará la ID de DevKit y **Configuración**.

   ![Modo de configuración de DevKit](media/iot-hub-arduino-iot-devkit-az3166-translator/devkit-configuration-mode.png)

1. Presione `F1`, escriba y seleccione **Azure IoT Device Workbench: Configurar opciones de dispositivo... > Config Device Connection String (Configurar cadena de conexión de dispositivo)** . Seleccione **Select IoT Hub Device Connection String** (Seleccionar cadena de conexión de dispositivo de IoT Hub) para configurarla en DevKit.
   ![Configurar cadena de conexión](media/iot-hub-arduino-iot-devkit-az3166-translator/configure-connection-string.png)

1. Verá la notificación cuando se realice correctamente.
   ![Configuración correcta de cadena de conexión](media/iot-hub-arduino-iot-devkit-az3166-translator/configure-connection-string-success.png)

1. Haga clic en `F1` de nuevo, escriba y seleccione **Azure IoT Device Workbench: Upload Device Code** (Carga del código de dispositivo). Se inicia la compilación y la carga del código en DevKit.
   ![Carga del dispositivo](media/iot-hub-arduino-iot-devkit-az3166-translator/device-upload.png)

## <a name="test-the-project"></a>Prueba del proyecto

Después de la inicialización de la aplicación, siga las instrucciones en la pantalla de DevKit. El idioma de origen predeterminado es el chino.

Para seleccionar otro idioma para traducir:

1. Presione el botón A para acceder al modo de configuración.

2. Presione el botón B para desplazarse por todos los idiomas origen admitidos.

3. Presione el botón A para confirmar la elección de idioma de origen.

4. Mantenga pulsado el botón B mientras habla y suéltelo para iniciar la traducción.

5. El texto traducido al inglés se mostrará en la pantalla.

![Selección de idioma](media/iot-hub-arduino-iot-devkit-az3166-translator/select-language.jpg)

![Resultado de la traducción](media/iot-hub-arduino-iot-devkit-az3166-translator/translation-result.jpg)

En la pantalla de resultados de traducción, puede:

- Presionar los botones A y B para desplazarse y seleccionar el idioma de origen.

- Presionar el botón B para hablar. Para enviar la voz y obtener el texto traducido, suelte el botón B.

## <a name="how-it-works"></a>Funcionamiento

![mini-solution-voice-to-tweet-diagram](media/iot-hub-arduino-iot-devkit-az3166-translator/diagram.png)

IoT DevKit graba su voz y, a continuación, envía una solicitud HTTP para activar Azure Functions. Azure Functions llama a la API del traductor de voz de la instancia de Cognitive Services para realizar la traducción. Cuando Azure Functions recibe el texto de traducción, envía un mensaje C2D al dispositivo. A continuación, la traducción se mostrará en la pantalla.

## <a name="problems-and-feedback"></a>Problemas y comentarios

Si tiene problemas, consulte las [preguntas frecuentes de IoT DevKit](https://microsoft.github.io/azure-iot-developer-kit/docs/faq/) o póngase en contacto con nosotros mediante los siguientes canales:

* [Gitter.im](https://gitter.im/Microsoft/azure-iot-developer-kit)
* [Stack Overflow](https://stackoverflow.com/questions/tagged/iot-devkit)

## <a name="next-steps"></a>Pasos siguientes

Ha aprendido a usar IoT DevKit como traductor mediante Azure Functions y Cognitive Services. En este tema de procedimientos, ha aprendido lo siguiente:

> [!div class="checklist"]
> * Uso de una tarea de Visual Studio Code para automatizar aprovisionamientos en la nube
> * Configuración de la cadena de conexión del dispositivo IoT de Azure
> * Implementación de la función de Azure
> * Prueba de la traducción de mensajes de voz

Consulte los otros tutoriales para obtener información sobre:

> [!div class="nextstepaction"]
> [Conexión de IoT DevKit AZ3166 al acelerador de la solución de supervisión remota de Azure IoT](https://docs.microsoft.com/azure/iot-hub/iot-hub-arduino-iot-devkit-az3166-devkit-remote-monitoring)
