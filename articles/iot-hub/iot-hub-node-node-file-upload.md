---
title: Carga de archivos desde dispositivos a Azure IoT Hub con Node | Microsoft Docs
description: Cómo cargar archivos de un dispositivo a la nube mediante el SDK de dispositivo IoT de Azure para Node.js. Los archivos cargados se almacenan en un contenedor de blobs de Azure Storage.
author: wesmc7777
manager: philmea
ms.author: wesmc
ms.service: iot-hub
services: iot-hub
ms.devlang: nodejs
ms.topic: conceptual
ms.date: 06/28/2017
ms.custom: mqtt, devx-track-javascript
ms.openlocfilehash: 65155d9f78cc82eb797c4b655adeeeabb24a8e33
ms.sourcegitcommit: 43558caf1f3917f0c535ae0bf7ce7fe4723391f9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/11/2020
ms.locfileid: "90019519"
---
# <a name="upload-files-from-your-device-to-the-cloud-with-iot-hub-nodejs"></a>Carga de archivos de un dispositivo a la nube con IoT Hub (Node.js)

[!INCLUDE [iot-hub-file-upload-language-selector](../../includes/iot-hub-file-upload-language-selector.md)]

Este tutorial se basa en el código del tutorial sobre el [envío de mensajes de la nube al dispositivo con IoT Hub](iot-hub-node-node-c2d.md) para mostrar cómo se usan las [funcionalidades de carga de archivos de IoT Hub](iot-hub-devguide-file-upload.md) para cargar un archivo en [Azure Blob Storage](../storage/index.yml). En este tutorial se muestra cómo realizar las siguientes acciones:

* Proporcionar un dispositivo de forma segura con un identificador URI de blob de Azure para cargar un archivo.

* Usar las notificaciones de carga de archivo de IoT Hub para desencadenar el procesamiento del archivo en el back-end de aplicación.

En el inicio rápido sobre el [Envío de telemetría desde un dispositivo a un centro de IoT](quickstart-send-telemetry-node.md) se explica la funcionalidad básica de mensajería del dispositivo a la nube de IoT Hub. Sin embargo, en algunos casos no se pueden asignar fácilmente los datos de que los dispositivos envían en los mensajes de dispositivo a nube con un tamaño relativamente reducido que acepta Azure IoT Hub. Por ejemplo:

* Archivos grandes con imágenes
* Vídeos
* Datos de vibración muestreados con alta frecuencia
* Alguna forma de datos procesados previamente.

Dichos archivos se suelen procesar por lotes en la nube mediante herramientas como [Azure Data Factory](../data-factory/introduction.md) o la pila [Hadoop](../hdinsight/index.yml). Cuando necesite cargar archivos desde un dispositivo, todavía puede usar la seguridad y confiabilidad de IoT Hub.

Al final de este tutorial, ejecutará dos aplicaciones de consola de Node.js:

* **SimulatedDevice.js**, que carga un archivo en el almacenamiento mediante un identificador URI de SAS proporcionado por IoT Hub.

* **ReadFileUploadNotification.js**, que recibe notificaciones de carga de archivos de IoT Hub.

> [!NOTE]
> IoT Hub admite muchas plataformas de dispositivos y lenguajes (incluido C, .NET, JavaScript, Python y Java), mediante los SDK de dispositivo IoT de Azure. Vea el [Centro para desarrolladores de IoT de Azure] para obtener instrucciones paso a paso sobre cómo conectar el dispositivo a Azure IoT Hub.

[!INCLUDE [iot-hub-include-x509-ca-signed-file-upload-support-note](../../includes/iot-hub-include-x509-ca-signed-file-upload-support-note.md)]

## <a name="prerequisites"></a>Requisitos previos

* Node.js versión 10.0.x o posteriores. En [Preparación del entorno de desarrollo](https://github.com/Azure/azure-iot-sdk-node/tree/master/doc/node-devbox-setup.md) se describe cómo instalar Node.js para este tutorial en Windows o Linux.

* Una cuenta de Azure activa. (En caso de no tener ninguna, puede crear una [cuenta gratuita](https://azure.microsoft.com/pricing/free-trial/) en tan solo unos minutos).

* Asegúrese de que el puerto 8883 está abierto en el firewall. En el ejemplo de dispositivo de este artículo se usa el protocolo MQTT, que se comunica mediante el puerto 8883. Este puerto puede estar bloqueado en algunos entornos de red corporativos y educativos. Para más información y saber cómo solucionar este problema, consulte [Conexión a IoT Hub (MQTT)](iot-hub-mqtt-support.md#connecting-to-iot-hub).

[!INCLUDE [iot-hub-associate-storage](../../includes/iot-hub-associate-storage.md)]

## <a name="upload-a-file-from-a-device-app"></a>Carga de un archivo desde una aplicación de dispositivo

En esta sección, creará la aplicación para dispositivos para cargar un archivo en IoT Hub.

1. Cree una carpeta vacía denominada ```simulateddevice```.  En la carpeta ```simulateddevice```, cree un archivo package.json con el siguiente comando en el símbolo del sistema.  Acepte todos los valores predeterminados:

    ```cmd/sh
    npm init
    ```

2. En el símbolo del sistema, en la carpeta ```simulateddevice```, ejecute el siguiente comando para instalar el paquete del SDK de dispositivo **azure-iot-device** y el paquete **azure-iot-device-mqtt**:

    ```cmd/sh
    npm install azure-iot-device azure-iot-device-mqtt --save
    ```

3. Con un procesador de texto, cree un archivo **SimulatedDevice.js** en la carpeta ```simulateddevice```.

4. Agregue las siguientes instrucciones ```require``` al principio del archivo **SimulatedDevice.js** :

    ```javascript
    'use strict';

    var fs = require('fs');
    var mqtt = require('azure-iot-device-mqtt').Mqtt;
    var clientFromConnectionString = require('azure-iot-device-mqtt').clientFromConnectionString;
    ```

5. Agregue una variable `deviceconnectionstring` y utilícela para crear una instancia de **cliente**.  Reemplace `{deviceconnectionstring}` por el nombre del dispositivo que creó en la sección *Creación de una instancia de IoT Hub*:

    ```javascript
    var connectionString = '{deviceconnectionstring}';
    var filename = 'myimage.png';
    ```

    > [!NOTE]
    > Por simplicidad, la cadena de conexión se incluye en el código: esto no es una práctica recomendada y, según su arquitectura y caso de uso, puede ser recomendable considerar alternativas más seguras para almacenar este secreto.

6. Agregue el siguiente código para conectar el cliente:

    ```javascript
    var client = clientFromConnectionString(connectionString);
    console.log('Client connected');
    ```

7. Cree una devolución de llamada y use la función **uploadToBlob** para cargar el archivo.

    ```javascript
    fs.stat(filename, function (err, stats) {
        const rr = fs.createReadStream(filename);

        client.uploadToBlob(filename, rr, stats.size, function (err) {
            if (err) {
                console.error('Error uploading file: ' + err.toString());
            } else {
                console.log('File uploaded');
            }
        });
    });
    ```

8. Guarde el archivo **SimulatedDevice.js** y ciérrelo.

9. Copie un archivo de imagen a la carpeta `simulateddevice` llámelo `myimage.png`.

## <a name="get-the-iot-hub-connection-string"></a>Obtención de la cadena de conexión de IoT Hub

En este artículo, creará un servicio de back-end para recibir mensajes de notificación de carga de archivos desde la instancia de IOT Hub que creó en [Enviar telemetría desde un dispositivo a IoT Hub](quickstart-send-telemetry-node.md). Para recibir mensajes de notificación de carga de archivos, el servicio necesita el permiso de **conexión de servicio**. De forma predeterminada, todas las instancias de IoT Hub se crean con una directiva de acceso compartido denominada **servicio** que concede este permiso.

[!INCLUDE [iot-hub-include-find-service-connection-string](../../includes/iot-hub-include-find-service-connection-string.md)]

## <a name="receive-a-file-upload-notification"></a>Recepción de una notificación de carga de archivos

En esta sección, se crea una aplicación de consola de Node.js que recibe mensajes de notificación de carga de archivos de IoT Hub.

Puede usar la cadena de conexión **iothubowner** de IoT Hub para completar esta sección. Puede encontrar la cadena de conexión en [Azure Portal](https://portal.azure.com/) o en la hoja **Directiva de acceso compartido**.

1. Cree una carpeta vacía denominada ```fileuploadnotification```.  En la carpeta ```fileuploadnotification```, cree un archivo package.json con el siguiente comando en el símbolo del sistema.  Acepte todos los valores predeterminados:

    ```cmd/sh
    npm init
    ```

2. En el símbolo del sistema, en la carpeta ```fileuploadnotification```, ejecute el siguiente comando para instalar el paquete del SDK **azure-iothub**:

    ```cmd/sh
    npm install azure-iothub --save
    ```

3. Con un editor de texto, cree un archivo **FileUploadNotification.js** en la carpeta `fileuploadnotification`.

4. Agregue las siguientes instrucciones `require` al principio del archivo **FileUploadNotification.js**:

    ```javascript
    'use strict';

    var Client = require('azure-iothub').Client;
    ```

5. Agregue una variable `iothubconnectionstring` y utilícela para crear una instancia de **cliente**.  Reemplace el valor de marcador de posición `{iothubconnectionstring}` por la cadena de conexión de IoT Hub que copió anteriormente en [Obtener la cadena de conexión de IoT Hub](#get-the-iot-hub-connection-string).

    ```javascript
    var connectionString = '{iothubconnectionstring}';
    ```

    > [!NOTE]
    > Por simplicidad, la cadena de conexión se incluye en el código: esto no es una práctica recomendada y, según su arquitectura y caso de uso, puede ser recomendable considerar alternativas más seguras para almacenar este secreto.

6. Agregue el siguiente código para conectar el cliente:

    ```javascript
    var serviceClient = Client.fromConnectionString(connectionString);
    ```

7. Abra el cliente y use la función **getFileNotificationReceiver** para recibir las actualizaciones de estado.

    ```javascript
    serviceClient.open(function (err) {
      if (err) {
        console.error('Could not connect: ' + err.message);
      } else {
        console.log('Service client connected');
        serviceClient.getFileNotificationReceiver(function receiveFileUploadNotification(err, receiver){
          if (err) {
            console.error('error getting the file notification receiver: ' + err.toString());
          } else {
            receiver.on('message', function (msg) {
              console.log('File upload from device:')
              console.log(msg.getData().toString('utf-8'));
            });
          }
        });
      }
    });
    ```

8. Guarde y cierre el archivo **FileUploadNotification.js**.

## <a name="run-the-applications"></a>Ejecución de las aplicaciones

Ahora está preparado para ejecutar las aplicaciones.

En la carpeta `fileuploadnotification` del símbolo del sistema, ejecute el siguiente comando:

```cmd/sh
node FileUploadNotification.js
```

En la carpeta `simulateddevice` del símbolo del sistema, ejecute el siguiente comando:

```cmd/sh
node SimulatedDevice.js
```

En la captura de pantalla siguiente se muestra la salida de la aplicación **SimulatedDevice**:

![Salida de la aplicación simulated-device](./media/iot-hub-node-node-file-upload/simulated-device.png)

En la captura de pantalla siguiente se muestra la salida de la aplicación **FileUploadNotification**:

![Salida de la aplicación read-file-upload-notification](./media/iot-hub-node-node-file-upload/read-file-upload-notification.png)

Puede usar el portal para ver el archivo cargado en el contenedor de almacenamiento que configuró:

![Archivo cargado](./media/iot-hub-node-node-file-upload/uploaded-file.png)

## <a name="next-steps"></a>Pasos siguientes

En este tutorial ha aprendido a usar la funcionalidad de carga de archivos de IoT Hub para simplificar la carga de archivos desde los dispositivos. Puede continuar explorando las características y los escenarios del centro de IoT con los siguientes artículos:

* [Creación de un centro de IoT mediante programación](iot-hub-rm-template-powershell.md)

* [Introducción a SDK para C](iot-hub-device-sdk-c-intro.md)

* [SDK de IoT de Azure](iot-hub-devguide-sdks.md)
