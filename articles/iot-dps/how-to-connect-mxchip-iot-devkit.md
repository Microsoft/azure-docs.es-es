---
title: Registro del MXChip IoT DevKit con IoT Hub mediante el aprovisionamiento automático con el servicio Azure IoT Hub Device Provisioning | Microsoft Docs
description: Registro del MXChip IoT DevKit con IoT Hub mediante el aprovisionamiento automático con Azure IoT Hub Device Provisioning Service (DPS).
author: wesmc7777
ms.author: wesmc
ms.date: 06/25/2019
ms.topic: conceptual
ms.service: iot-dps
services: iot-dps
manager: eliotgra
ms.openlocfilehash: 2a030d9ca5422e12856dcb81b29f8327e684c97e
ms.sourcegitcommit: 03662d76a816e98cfc85462cbe9705f6890ed638
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/15/2020
ms.locfileid: "90528660"
---
# <a name="use-azure-iot-hub-device-provisioning-service-auto-provisioning-to-register-the-mxchip-iot-devkit-with-iot-hub"></a>Registro del MXChip IoT DevKit con IoT Hub mediante el aprovisionamiento automático con el servicio Azure IoT Hub Device Provisioning

En este artículo se describe el uso de Azure IoT Hub Device Provisioning Service para el [aprovisionamiento](about-iot-dps.md#provisioning-process) de MXChip IoT DevKit para una instancia de Azure IoT Hub. En este tutorial, aprenderá a:

* Configurar el punto de conexión global del servicio Device Provisioning en un dispositivo.
* Usar un secreto de dispositivo único (UDS) para generar un certificado X.509.
* Inscribir un dispositivo individual.
* Comprobar que el dispositivo está registrado.

[MXChip IoT DevKit](https://aka.ms/iot-devkit) es una placa integral compatible con Arduino que incluye numerosos periféricos y sensores. Puede desarrollar con [Azure IoT Device Workbench](https://aka.ms/iot-workbench) o el paquete de extensión [Azure IoT Tools](https://aka.ms/azure-iot-tools) en Visual Studio Code. DevKit incluye un [catálogo de proyectos](https://microsoft.github.io/azure-iot-developer-kit/docs/projects/) cada vez mayor que sirve de guía para crear prototipos de soluciones de Internet de las cosas (IoT) que aprovechen los servicios de Azure.

## <a name="before-you-begin"></a>Antes de empezar

Para completar los pasos de este tutorial, deberá realizar las siguientes tareas:

* Configure la red Wi-Fi de DevKit y prepare el entorno de desarrollo siguiendo los pasos en la sección "Preparación del entorno de desarrollo" de [Conexión de IoT DevKit AZ3166 a Azure IoT Hub en la nube](/azure/iot-hub/iot-hub-arduino-iot-devkit-az3166-get-started#prepare-the-development-environment).
* Actualice al firmware más reciente (1.3.0 o posterior) con el tutorial de [actualización del firmware de DevKit](https://microsoft.github.io/azure-iot-developer-kit/docs/firmware-upgrading/).
* Cree y vincule una instancia de IoT Hub con una instancia del servicio Device Provisioning mediante los pasos que se indican en [Configuración de Azure IoT Hub Device Provisioning Service con Azure Portal](/azure/iot-dps/quick-setup-auto-provision).

## <a name="open-sample-project"></a>Abra el proyecto de ejemplo

1. Asegúrese de que IoT DevKit **no está conectado** al equipo. Primero, inicie VS Code y luego conecte DevKit al equipo.

1. Haga clic en `F1` para abrir la paleta de comandos, escriba y seleccione **Azure IoT Device Workbench: Open Examples...** (Abrir ejemplos...). A continuación, seleccione **IoT DevKit** como placa.

1. En la página de ejemplos de IoT Workbench, busque **Device Registration with DPS** (Registro de dispositivos con DPS) y haga clic en **Open Sample** (Abrir ejemplo). A continuación, seleccione la ruta de acceso predeterminada para descargar el código de ejemplo.
    ![Abrir ejemplo](media/how-to-connect-mxchip-iot-devkit/open-sample.png)

## <a name="save-a-unique-device-secret-on-device-security-storage"></a>Guardar el secreto de dispositivo único en el almacenamiento de seguridad del dispositivo

Se puede configurar el aprovisionamiento automático en el dispositivo en función del [mecanismo de atestación](concepts-service.md#attestation-mechanism) de este. MXChip IoT DevKit usa la tecnología [Device Identity Composition Engine](https://trustedcomputinggroup.org/wp-content/uploads/Foundational-Trust-for-IOT-and-Resource-Constrained-Devices.pdf) de [Trusted Computing Group](https://trustedcomputinggroup.org). Se usa un **secreto de dispositivo único** (UDS) guardado en un chip de seguridad de STSAFE ([STSAFE-A100](https://microsoft.github.io/azure-iot-developer-kit/docs/understand-security-chip/)) en DevKit para generar el [certificado X.509](concepts-x509-attestation.md) único del dispositivo. El certificado se usa más adelante en el proceso de inscripción del servicio Device Provisioning y durante el registro en el entorno de ejecución.

Un UDS normal es una cadena de 64 caracteres, tal como se muestra en el ejemplo siguiente:

```
19e25a259d0c2be03a02d416c05c48ccd0cc7d1743458aae1cb488b074993eae
```

Para guardar un UDS en DevKit:

1. En VS Code, haga clic en la barra de estado para seleccionar el puerto COM para el DevKit.
  ![Seleccionar un puerto COM](media/how-to-connect-mxchip-iot-devkit/select-com.png)

1. En DevKit, mantenga presionado el **botón A**, presione y suelte el botón de **reinicio** y, a continuación, suelte el **botón A**. El DevKit entra en modo de configuración.

1. Haga clic en `F1` para abrir la paleta de comandos, escriba y seleccione **Azure IoT Device Workbench: Configurar opciones de dispositivo... > Config Unique Device String (Configurar cadena de dispositivo única)** .
  ![Configurar UDS](media/how-to-connect-mxchip-iot-devkit/config-uds.png)

1. Anote la cadena UDS generada. La necesitará para generar el certificado X.509. A continuación, presione `Enter`.
  ![Copiar UDS](media/how-to-connect-mxchip-iot-devkit/copy-uds.png)

1. Use la notificación para confirmar que la UDS se configuró correctamente en STSAFE.
  ![Se configuró correctamente el UDS](media/how-to-connect-mxchip-iot-devkit/config-uds-success.png)

> [!NOTE]
> También puede configurar el UDS a través del puerto serie mediante utilidades como Putty. Siga los pasos en [Use configuration mode](https://microsoft.github.io/azure-iot-developer-kit/docs/use-configuration-mode/) (Usar el modo de configuración) para hacerlo.

## <a name="update-the-global-device-endpoint-and-id-scope"></a>Actualizar el punto de conexión global del dispositivo y el ámbito de Id.

En el código de dispositivo, deberá especificar el [punto de conexión de aprovisionamiento de dispositivos](/azure/iot-dps/concepts-service#device-provisioning-endpoint) y el ámbito de Id. para garantizar el aislamiento del inquilino.

1. En Azure Portal, seleccione el panel **Información general** del servicio Device Provisioning y anote los valores de **Punto de conexión global del dispositivo** e **Ámbito de id**.
  ![Punto de conexión global y ámbito de id. del servicio Device Provisioning](media/how-to-connect-mxchip-iot-devkit/dps-global-endpoint.png)

1. Abra **DevKitDPS.ino**. Busque y reemplace `[Global Device Endpoint]` y `[ID Scope]` por los valores que acababa de anotar.
  ![Punto de conexión del servicio Device Provisioning](media/how-to-connect-mxchip-iot-devkit/endpoint.png)

1. Rellene la variable `registrationId` en el código. La variable solo puede contener caracteres alfanuméricos, minúsculas y guiones con un máximo de 128 caracteres permitido. Anote también el valor.
  ![Identificador de registro](media/how-to-connect-mxchip-iot-devkit/registration-id.png)

1. Presione `F1`, escriba y seleccione **Azure IoT Device Workbench: Upload Device Code** (Carga del código de dispositivo). El código comienza a compilarse y cargarse a DevKit.
  ![Carga del dispositivo](media/how-to-connect-mxchip-iot-devkit/device-upload.png)

## <a name="generate-x509-certificate"></a>Generación de certificado X.509

El certificado X.509 se usa como [mecanismo de atestación](/azure/iot-dps/concepts-device#attestation-mechanism) en este ejemplo. Deberá usar una utilidad para generarlo.

1. En VS Code, presione `F1`, escriba y seleccione **Abrir nuevo terminal** para abrir la ventana de terminal.

1. Ejecute `dps_cert_gen.exe` en la carpeta `tool`.

1. Especifique `..\.build\DevKitDPS` como la ubicación del archivo binario compilado. A continuación, pegue los valores de **UDS** y **registrationId** que acaba de anotar. 
  ![Generar X.509](media/how-to-connect-mxchip-iot-devkit/gen-x509.png)

1. Se genera un certificado X.509 `.pem` en la misma carpeta.
  ![Archivo X.509](media/how-to-connect-mxchip-iot-devkit/pem-file.png)

## <a name="create-a-device-enrollment-entry"></a>Creación de una entrada de inscripción de dispositivo

1. En Azure Portal, abra el servicio Device Provisioning, desplácese a la sección Administrar inscripciones y haga clic en **Agregar inscripción individual**.
  ![Agregar inscripción individual](media/how-to-connect-mxchip-iot-devkit/add-enrollment.png)

1. Haga clic en el icono de archivo junto al **archivo .pem o .cer del certificado principal** para cargar el archivo `.pem` generado.
  ![Cargar .pem](media/how-to-connect-mxchip-iot-devkit/upload-pem.png)

## <a name="verify-the-devkit-is-registered-with-azure-iot-hub"></a>Comprobar que DevKit está registrado en Azure IoT Hub

Presione el botón **Restablecer** en DevKit. Debería ver el mensaje **DPS Connected!** (DPS conectado) en la pantalla del DevKit. Una vez que arranca el dispositivo, tienen lugar las siguientes acciones:

1. El dispositivo envía una solicitud de registro al servicio Device Provisioning.
1. El servicio devuelve un desafío de registro al que responde el dispositivo.
1. Una vez que el registro se ha realizado correctamente, el servicio Device Provisioning envía el URI de IoT Hub, el identificador de dispositivo y la clave cifrada nuevamente al dispositivo.
1. A continuación, la aplicación cliente de IoT Hub del dispositivo se conecta al centro.
1. Después de una conexión correcta, el dispositivo aparece en la instancia de Device Explorer de IoT Hub.
  ![Dispositivo registrado](./media/how-to-connect-mxchip-iot-devkit/device-registered.png)

## <a name="problems-and-feedback"></a>Problemas y comentarios

Si tiene problemas, consulte las [preguntas frecuentes](https://microsoft.github.io/azure-iot-developer-kit/docs/faq/) de Iot DevKit o póngase en contacto con el soporte técnico en los siguientes canales:

* [Gitter.im](https://gitter.im/Microsoft/azure-iot-developer-kit)
* [Stack Overflow](https://stackoverflow.com/questions/tagged/iot-devkit)

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, aprendió a inscribir un dispositivo de forma segura en el servicio Device Provisioning mediante Device Identity Composition Engine, para que el dispositivo se pueda registrar automáticamente en Azure IoT Hub. 

En resumen, ha aprendido a:

> [!div class="checklist"]
> * Configurar el punto de conexión global del servicio Device Provisioning en un dispositivo.
> * Usar un secreto de dispositivo único para generar un certificado X.509.
> * Inscribir un dispositivo individual.
> * Comprobar que el dispositivo está registrado.

Aprenda cómo [crear y aprovisionar un dispositivo simulado](./quick-create-simulated-device.md).

