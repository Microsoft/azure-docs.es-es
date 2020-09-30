---
title: Información sobre la carga de archivos de Azure IoT Hub | Microsoft Docs
description: 'Guía del desarrollador: uso de la característica de carga de archivos de IoT Hub para administrar la carga de archivos desde un dispositivo a un contenedor de blobs de Azure Storage.'
author: robinsh
manager: philmea
ms.author: robinsh
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 11/07/2018
ms.custom:
- mqtt
- 'Role: Cloud Development'
- 'Role: IoT Device'
ms.openlocfilehash: 3286b464051b8fea88d2797d4f82b20fe432b4b8
ms.sourcegitcommit: 43558caf1f3917f0c535ae0bf7ce7fe4723391f9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/11/2020
ms.locfileid: "90019536"
---
# <a name="upload-files-with-iot-hub"></a>Carga de archivos con IoT Hub

Tal como se detalla en el artículo [Puntos de conexión de IoT Hub](iot-hub-devguide-endpoints.md), un dispositivo puede iniciar cargas de archivos mediante el envío de una notificación a través de un punto de conexión accesible desde el dispositivo ( **/devices/{deviceId}/files**). Cuando un dispositivo notifica a IoT Hub que se ha completado una carga, este envía un mensaje de notificación de carga a través del punto de conexión accesible desde el servicio ( **/messages/servicebound/filenotifications**).

En lugar de servir de intermediario de los mensajes, IoT Hub actúa como distribuidor a una cuenta de Azure Storage asociada. Un dispositivo solicita un token de almacenamiento de IoT Hub que es específico del archivo que el dispositivo quiere cargar. El dispositivo usa el URI de SAS para cargar el archivo en el almacenamiento y, cuando la carga ha finalizado, envía una notificación de finalización a IoT Hub. IoT Hub comprueba que la carga del archivo está completa y agrega después un mensaje de notificación de carga de archivo al nuevo punto de conexión de notificaciones de archivos accesible desde el servicio.

Antes de cargar un archivo en IoT Hub desde un dispositivo, tiene que configurar su centro mediante la [asociación de una cuenta de Azure Storage](iot-hub-devguide-file-upload.md#associate-an-azure-storage-account-with-iot-hub) al mismo.

El dispositivo puede entonces [inicializar una carga](iot-hub-devguide-file-upload.md#initialize-a-file-upload) y, a continuación, [notificar a IoT Hub](iot-hub-devguide-file-upload.md#notify-iot-hub-of-a-completed-file-upload) cuando se complete la carga. Opcionalmente, cuando un dispositivo notifica a IoT Hub que la carga está completa, el servicio puede generar un [mensaje de notificación](iot-hub-devguide-file-upload.md#file-upload-notifications).

[!INCLUDE [iot-hub-include-x509-ca-signed-file-upload-support-note](../../includes/iot-hub-include-x509-ca-signed-file-upload-support-note.md)]

### <a name="when-to-use"></a>Cuándo se usa

Cargas de archivos, para archivos multimedia y grandes lotes de telemetría cargados por dispositivos conectados de manera intermitente o comprimidos para ahorrar ancho de banda.

Si duda entre el uso de propiedades notificadas, mensajes de dispositivo a nube o carga de archivos, consulte [Guía de comunicación de dispositivo a nube](iot-hub-devguide-d2c-guidance.md).

## <a name="associate-an-azure-storage-account-with-iot-hub"></a>Asociar una cuenta de Azure Storage con IoT Hub

Para utilizar la funcionalidad de carga de archivos, primero debe vincular una cuenta de Azure Storage a IoT Hub. Puede completar esta tarea a través de Azure Portal o mediante programación, con las [API REST del proveedor de recursos de IoT Hub](/rest/api/iothub/iothubresource). Una vez asociada una cuenta de Azure Storage a IoT Hub, el servicio devuelve un URI de SAS a un dispositivo cuando este inicia una solicitud de carga de archivos.

Las guías paso a paso de [Carga de archivos de un dispositivo a la nube con IoT Hub](iot-hub-csharp-csharp-file-upload.md) proporcionan un tutorial completo sobre el proceso de carga de archivos. Estas guías paso a paso muestran cómo usar Azure Portal para asociar una cuenta de almacenamiento a un centro de IoT.

> [!NOTE]
> Los [SDK de IoT de Azure](iot-hub-devguide-sdks.md) administran automáticamente la recuperación del URI de SAS: cargan el archivo y notifican a IoT Hub que la carga se ha completado.

## <a name="initialize-a-file-upload"></a>Inicialización de una carga de archivos
IoT Hub tiene un punto de conexión concreto para dispositivos para solicitar un URI de SAS para el almacenamiento para cargar un archivo. Para iniciar el proceso de carga de archivos, el dispositivo envía una solicitud POST a `{iot hub}.azure-devices.net/devices/{deviceId}/files` con el cuerpo JSON siguiente:

```json
{
    "blobName": "{name of the file for which a SAS URI will be generated}"
}
```

IoT Hub devuelve los datos siguientes, que usa el dispositivo para cargar el archivo:

```json
{
    "correlationId": "somecorrelationid",
    "hostName": "yourstorageaccount.blob.core.windows.net",
    "containerName": "testcontainer",
    "blobName": "test-device1/image.jpg",
    "sasToken": "1234asdfSAStoken"
}
```

### <a name="deprecated-initialize-a-file-upload-with-a-get"></a>En desuso: inicializar una carga de archivo con un comando GET

> [!NOTE]
> Esta sección describe la funcionalidad en desuso para la recepción de un URI de SAS de IoT Hub. Use el método POST que se ha descrito anteriormente.

IoT Hub tiene dos puntos de conexión de REST que permiten la carga de archivos: uno para obtener el URI de SAS para el almacenamiento y el otro para notificar a IoT Hub que la carga se ha completado. El dispositivo inicia el proceso de carga de archivos mediante el envío de una operación GET al centro de IoT en `{iot hub}.azure-devices.net/devices/{deviceId}/files/{filename}`. IoT Hub devuelve:

* Un URI de SAS específico del archivo que se va a cargar.

* Un identificador de correlación que se usará cuando se haya completado la carga.

## <a name="notify-iot-hub-of-a-completed-file-upload"></a>Notificación a IoT Hub de una carga de archivos completada

El dispositivo carga el archivo en el almacenamiento mediante los SDK de Azure Storage. Cuando la carga se haya completado, el dispositivo envía una solicitud POST a `{iot hub}.azure-devices.net/devices/{deviceId}/files/notifications` con el siguiente cuerpo JSON:

```json
{
    "correlationId": "{correlation ID received from the initial request}",
    "isSuccess": bool,
    "statusCode": XXX,
    "statusDescription": "Description of status"
}
```

El valor de `isSuccess` es un valor booleano que indica si se cargó correctamente el archivo. El código de estado para `statusCode` es el estado de la carga del archivo al almacenamiento y `statusDescription` corresponde a `statusCode`.

## <a name="reference-topics"></a>Temas de referencia:

Los siguientes temas de referencia proporcionan más información sobre la carga de archivos desde un dispositivo.

## <a name="file-upload-notifications"></a>Notificaciones de carga de archivos

Opcionalmente, cuando un dispositivo notifica a IoT Hub que la carga está completa, IoT Hub genera un mensaje de notificación. Este mensaje contiene el nombre y ubicación de almacenamiento del archivo.

Como se ha explicado en [Puntos de conexión](iot-hub-devguide-endpoints.md), IoT Hub entrega notificaciones de carga de archivos a través de un punto de conexión accesible desde el servicio ( **/messages/servicebound/fileuploadnotifications**) en forma de mensajes. La semántica de recepción de las notificaciones de carga de archivos es la misma que para los mensajes de nube a dispositivo y tiene el mismo [ciclo de vida del mensaje](iot-hub-devguide-messages-c2d.md#the-cloud-to-device-message-life-cycle). Cada mensaje recuperado del punto de conexión de notificación de carga de archivos es un registro JSON con las siguientes propiedades:

| Propiedad | Descripción |
| --- | --- |
| EnqueuedTimeUtc |Marca de tiempo que indica cuándo se creó la notificación. |
| deviceId |**DeviceId** del dispositivo que ha cargado el archivo. |
| BlobUri |URI del archivo cargado. |
| BlobName |Nombre del archivo cargado. |
| LastUpdatedTime |Marca de tiempo que indica cuándo se actualizó por última vez el archivo. |
| BlobSizeInBytes |Tamaño del archivo cargado. |

**Ejemplo**. Este ejemplo muestra el cuerpo de ejemplo de un mensaje de notificación de carga de archivos.

```json
{
    "deviceId":"mydevice",
    "blobUri":"https://{storage account}.blob.core.windows.net/{container name}/mydevice/myfile.jpg",
    "blobName":"mydevice/myfile.jpg",
    "lastUpdatedTime":"2016-06-01T21:22:41+00:00",
    "blobSizeInBytes":1234,
    "enqueuedTimeUtc":"2016-06-01T21:22:43.7996883Z"
}
```

## <a name="file-upload-notification-configuration-options"></a>Opciones de configuración de notificaciones de carga de archivos

Cada centro de IoT tiene las siguientes opciones de configuración para las notificaciones de carga de archivos:

| Propiedad | Descripción | Intervalo y valor predeterminado |
| --- | --- | --- |
| **enableFileUploadNotifications** |Controla si las notificaciones de carga de archivos se escriben en el punto de conexión de notificaciones de archivo. |Bool. Valor predeterminado: True. |
| **fileNotifications.ttlAsIso8601** |TTL predeterminado para las notificaciones de carga de archivos. |Intervalo ISO_8601 hasta 48H (1 minuto como mínimo). Valor predeterminado: 1 hora. |
| **fileNotifications.lockDuration** |Duración del bloqueo de la cola de notificaciones de carga de archivos. |De 5 a 300 segundos (5 segundos como mínimo). Valor predeterminado: 60 segundos. |
| **fileNotifications.maxDeliveryCount** |Número máximo de entregas en la cola de notificaciones de carga de archivos. |De 1 a 100. Valor predeterminado: 100. |

Puede establecer estas propiedades en la instancia de IoT Hub mediante Azure Portal, la CLI de Azure o PowerShell. Para más información sobre cómo hacerlo, consulte los temas que se encuentran en [Configuración de la carga de archivos](iot-hub-configure-file-upload.md).

## <a name="additional-reference-material"></a>Material de referencia adicional

Otros temas de referencia en la guía del desarrollador de IoT Hub son los siguientes:

* En [Puntos de conexión de IoT Hub](iot-hub-devguide-endpoints.md) se describen los diferentes puntos de conexión del centro de IoT para las operaciones en tiempo de ejecución y de administración.

* En [Cuotas y limitación](iot-hub-devguide-quotas-throttling.md) se describen las cuotas y el comportamiento de limitación que se aplican al servicio IoT Hub.

* En [SDK de dispositivo y servicio IoT de Azure](iot-hub-devguide-sdks.md) se muestran los diversos SDK de lenguaje que puede usar para desarrollar aplicaciones para dispositivo y de servicio que interactúen con IoT Hub.

* En [Lenguaje de consulta de IoT Hub](iot-hub-devguide-query-language.md) se describe el lenguaje de consulta que se puede usar para recuperar información de IoT Hub sobre los trabajos y dispositivos gemelos.

* En [Compatibilidad con MQTT de IoT Hub](iot-hub-mqtt-support.md) se proporciona más información sobre la compatibilidad de IoT Hub con el protocolo MQTT.

## <a name="next-steps"></a>Pasos siguientes

Ahora que ha aprendido a cargar los archivos desde dispositivos con IoT Hub, pueden interesarle los siguientes temas de la guía del desarrollador de IoT Hub:

* [Administración de identidades del dispositivo en IoT Hub](iot-hub-devguide-identity-registry.md)

* [Control del acceso a IoT Hub](iot-hub-devguide-security.md)

* [Uso de dispositivos gemelos para sincronizar el estado y las configuraciones](iot-hub-devguide-device-twins.md)

* [Invocación de un método directo en un dispositivo](iot-hub-devguide-direct-methods.md)

* [Programación de trabajos en varios dispositivos](iot-hub-devguide-jobs.md)

Para probar algunos de los conceptos descritos en este artículo, vea el siguiente tutorial de IoT Hub:

* [Carga de archivos desde dispositivos a la nube con IoT Hub](iot-hub-csharp-csharp-file-upload.md)