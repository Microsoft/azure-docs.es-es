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
ms.openlocfilehash: f60236163cd88822c384bd356a1770d616639174
ms.sourcegitcommit: 1fbc75b822d7fe8d766329f443506b830e101a5e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/14/2019
ms.locfileid: "65596626"
---
# <a name="upload-files-from-your-device-to-the-cloud-with-iot-hub"></a>Carga de archivos de un dispositivo a la nube con IoT Hub

[!INCLUDE [iot-hub-file-upload-language-selector](../../includes/iot-hub-file-upload-language-selector.md)]

Este tutorial se basa en el código del tutorial sobre el [envío de mensajes de la nube a un dispositivo con IoT Hub](iot-hub-node-node-c2d.md) para mostrar cómo se usan las [funcionalidades de carga de archivos de IoT Hub](iot-hub-devguide-file-upload.md) para cargar un archivo en [Azure Blob Storage](../storage/index.yml). En este tutorial se muestra cómo realizar las siguientes acciones:

- Proporcionar un dispositivo de forma segura con un identificador URI de blob de Azure para cargar un archivo.
- 
- Usar las notificaciones de carga de archivo de IoT Hub para desencadenar el procesamiento del archivo en el back-end de aplicación.

El tutorial [Introducción al IoT Hub](quickstart-send-telemetry-node.md) muestra la funcionalidad de mensajería básica del dispositivo a la nube de la instancia de IoT Hub. Sin embargo, en algunos casos no se pueden asignar fácilmente los datos de que los dispositivos envían en los mensajes de dispositivo a nube con un tamaño relativamente reducido que acepta Azure IoT Hub. Por ejemplo:

*  Archivos grandes con imágenes
*  Vídeos
*  Datos de vibración muestreados con alta frecuencia
*  Algún tipo de datos preprocesados.

Dichos archivos se suelen procesar por lotes en la nube mediante herramientas como [Azure Data Factory](../data-factory/introduction.md) o la pila [Hadoop](../hdinsight/index.yml). Cuando necesite cargar archivos desde un dispositivo, todavía puede usar la seguridad y confiabilidad de IoT Hub.

Al final de este tutorial, ejecutará dos aplicaciones de consola de Node.js:

* **SimulatedDevice.js**, que carga un archivo en el almacenamiento mediante un identificador URI de SAS proporcionado por IoT Hub.

* **ReadFileUploadNotification.js**, que recibe notificaciones de carga de archivos de IoT Hub.

> [!NOTE]
> IoT Hub admite muchas plataformas de dispositivos y lenguajes (incluido C, .NET, JavaScript, Python y Java), mediante los SDK de dispositivo IoT de Azure. Para obtener instrucciones detalladas sobre cómo conectar el dispositivo a Azure IoT Hub, consulte el [centro IoT de Azure para desarrolladores].

Para completar este tutorial, necesitará lo siguiente:

* Node.js versión 10.0 o posterior.

* Una cuenta de Azure activa. (En caso de no tener ninguna, puede crear una [cuenta gratuita](https://azure.microsoft.com/pricing/free-trial/) en tan solo unos minutos).

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

5. Agregue una variable `iothubconnectionstring` y utilícela para crear una instancia de **cliente**.  Sustituya `{iothubconnectionstring}` por la cadena de conexión a IoT Hub que creó en la sección _Creación de un IoT Hub_:

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

*  [Creación de un centro de IoT mediante programación](iot-hub-rm-template-powershell.md)
*  [Introducción a SDK para C](iot-hub-device-sdk-c-intro.md)
*  [SDK de IoT de Azure](iot-hub-devguide-sdks.md)