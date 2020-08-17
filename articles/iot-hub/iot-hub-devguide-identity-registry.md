---
title: Información del registro de identidades de Azure IoT Hub | Microsoft Docs
description: 'Guía del desarrollador: descripción del registro de identidades de IoT Hub y cómo usarlo para administrar los dispositivos Incluye información sobre la importación y exportación de identidades de dispositivos de forma masiva.'
author: wesmc7777
manager: philmea
ms.author: wesmc
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 08/29/2018
ms.custom:
- amqp
- mqtt
- 'Role: Cloud Development'
- 'Role: IoT Device'
ms.openlocfilehash: 675f25107a2e4f0ddedf468dc06afea36ffc6151
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87327759"
---
# <a name="understand-the-identity-registry-in-your-iot-hub"></a>Descripción del registro de identidades de un centro de IoT

Cada centro de IoT tiene un registro de identidades que almacena información acerca de los dispositivos y módulos que pueden conectarse a IoT Hub. Para que un dispositivo o un módulo puedan conectarse a un centro de IoT, debe haber una entrada para ese dispositivo o módulo en el registro de identidades del centro de IoT. También se debe autenticar un dispositivo o un módulo en el centro de IoT basándose en las credenciales almacenadas en el registro de identidades.

El identificador de dispositivo o módulo que se almacena en el registro de identidades distingue mayúsculas de minúsculas.

En un nivel superior, el registro de identidades es un conjunto de recursos de identidad de dispositivos o módulos compatible con REST. Cuando agrega una entrada al registro de identidades, IoT Hub crea un conjunto de recursos por dispositivo, como una cola que contiene mensajes de nube al dispositivo en curso.

Utilice el registro de identidad cuando necesite:

* Aprovisionar dispositivos o módulos que se conectan a la instancia de IoT Hub
* Controlar el acceso de cada dispositivo o módulo a los puntos de conexión accesibles desde el dispositivo o módulo del centro.

> [!NOTE]
> * El registro de identidad no contiene metadatos específicos de la aplicación.
> * La identidad de módulo y el módulo gemelo se encuentran en versión preliminar pública. La siguiente característica se admitirá en el módulo de identidad cuanto esté disponible con carácter general.
>

## <a name="identity-registry-operations"></a>Operaciones de registro de identidad

El registro de identidades de IoT Hub muestra las operaciones siguientes:

* Crear la identidad de dispositivo o módulo
* Actualizar la identidad del dispositivo o módulo
* Recuperar la identidad del dispositivo o módulo por identificador
* Eliminar la identidad del dispositivo o módulo
* Enumerar hasta 1.000 identidades
* Exportar todas las identidades de dispositivo a Azure Blob Storage
* Importar las identidades de dispositivo desde Azure Blob Storage

Todas estas operaciones pueden usar la simultaneidad optimista, tal como se especifica en [RFC7232](https://tools.ietf.org/html/rfc7232).

> [!IMPORTANT]
> La única manera de recuperar todas las identidades en un registro de identidades de un centro de IoT es usar la funcionalidad [Exportar](iot-hub-devguide-identity-registry.md#import-and-export-device-identities).

Un registro de identidades de un centro de IoT:

* No contiene los metadatos de la aplicación.
* Se puede acceder a él como si fuera un diccionario, mediante **deviceId** o **moduleId** como clave.
* No admite consultas expresivas.

Una solución de IoT normalmente tiene un almacén independiente específico de la solución que contiene los metadatos específicos de la aplicación. Por ejemplo, el almacén específico de la solución en una solución de un edificio inteligente puede registrar la sala en la que se implementa un sensor de temperatura.

> [!IMPORTANT]
> Solo se debe utilizar el registro de identidad para las operaciones de aprovisionamiento y administración de dispositivos. Las operaciones de alto rendimiento en tiempo de ejecución no deben depender de la realización de operaciones en el registro de identidad. Por ejemplo, comprobar el estado de conexión de un dispositivo antes de enviar un comando no es un modelo compatible. Asegúrese de comprobar las [tasas de limitación](iot-hub-devguide-quotas-throttling.md) para el registro de identidad y el patrón de [latido de dispositivo](iot-hub-devguide-identity-registry.md#device-heartbeat).

## <a name="disable-devices"></a>Deshabilitar dispositivos

Puede deshabilitar los dispositivos actualizando la propiedad **status** de una identidad en el registro de identidad. Normalmente, esta propiedad se usa en dos escenarios:

* Durante el proceso de orquestación de aprovisionamiento. Para más información, consulte [Aprovisionamiento de dispositivos](iot-hub-devguide-identity-registry.md#device-provisioning).

* Si por cualquier motivo cree que un dispositivo está en peligro o no está autorizado.

Esta característica no está disponible para los módulos.

## <a name="import-and-export-device-identities"></a>Importar y exportar identidades de dispositivo

Use operaciones asincrónicas en el [punto de conexión del proveedor de recursos de IoT Hub](iot-hub-devguide-endpoints.md) para exportar identidades de dispositivo de forma masiva desde un registro de identidades de una instancia de IoT Hub. Las exportaciones son trabajos de ejecución prolongada que usan un contenedor de blobs proporcionado por el cliente para guardar los datos de identidad del dispositivo que se leen en el registro de identidad.

Use operaciones asincrónicas en el [punto de conexión del proveedor de recursos de IoT Hub](iot-hub-devguide-endpoints.md) para importar identidades de dispositivo de forma masiva en un registro de identidades de una instancia de IoT Hub. Las importaciones son trabajos de ejecución prolongada que usan los datos de un contenedor de blobs proporcionado por el cliente para escribir datos de identidad del dispositivo en el registro de identidad.

Para más información sobre las API de importación y exportación, consulte [API REST del proveedor de recursos de IoT Hub](/rest/api/iothub/iothubresource). Para más información sobre la ejecución de trabajos de importación y exportación, consulte [Administración de identidades de dispositivos de IoT Hub de forma masiva](iot-hub-bulk-identity-mgmt.md).

Las identidades de dispositivo también se pueden exportar e importar desde una instancia de IoT Hub a través de la API de servicio a través de la [API REST](/rest/api/iothub/service/jobclient/createimportexportjob) de o uno de los [SDK de servicio](/azure/iot-hub/iot-hub-devguide-sdks#azure-iot-hub-service-sdks) de IoT Hub.

## <a name="device-provisioning"></a>Aprovisionamiento de dispositivos

Los datos del dispositivo que almacena una solución de IoT determinada dependen de los requisitos específicos de la solución. Sin embargo, como mínimo, una solución debe almacenar identidades del dispositivo y claves de autenticación. El IoT Hub incluye un registro de identidades que puede almacenar valores para cada dispositivo como identificadores, claves de autenticación y códigos de estado. Una posible solución consiste en usar otros servicios de Azure como Table Storage, Blob Storage o Cosmos DB para almacenar datos de dispositivos adicionales.

*Aprovisionamiento de dispositivos* es el proceso de agregar los datos iniciales del dispositivo a los almacenes de la solución. Para habilitar un nuevo dispositivo y que se conecte al centro, debe agregar un nuevo identificador de dispositivo y claves al registro de identidades del centro de IoT. Como parte del proceso de aprovisionamiento, es posible que tenga que inicializar datos específicos del dispositivo en otros almacenes de la solución. También puede usar el servicio Azure IoT Hub Device Provisioning para habilitar el aprovisionamiento Just-In-Time sin intervención del usuario de uno o varios centros de IoT sin necesidad de ninguna intervención humana. Para más información, consulte la [documentación del servicio de aprovisionamiento](https://azure.microsoft.com/documentation/services/iot-dps).

## <a name="device-heartbeat"></a>Latido de dispositivo

El registro de identidades de IoT Hub contiene un campo llamado **connectionState**. Utilice únicamente el campo **connectionState** durante el desarrollo y la depuración. Las soluciones de IoT no deben consultar el campo en tiempo de ejecución. Por ejemplo, no hay que consultar el campo **connectionState** para comprobar si el dispositivo está conectado antes de enviar un mensaje de la nube al dispositivo o un SMS. Se recomienda suscribirse al [**evento dispositivo desconectado**](iot-hub-event-grid.md#event-types) en Event Grid para obtener alertas y supervisar el estado de conexión del dispositivo. Siga este [tutorial](iot-hub-how-to-order-connection-state-events.md) para aprender a integrar eventos de dispositivo conectado y dispositivo desconectado de IoT Hub en la solución de IoT.

Si la solución de IoT necesita saber si un dispositivo está conectado, debe implementar el *patrón de latido*.
En el patrón de latido, el dispositivo envía mensajes de dispositivo a la nube al menos una vez en un período de tiempo predeterminado (por ejemplo, al menos una vez cada hora). Por lo tanto, incluso si un dispositivo no tiene ningún dato que enviar, seguirá enviando un mensaje de dispositivo a la nube vacío (normalmente con una propiedad que lo identifica como un latido). En el lado del servicio, la solución mantiene un mapa con el último latido recibido para cada dispositivo. Si solución si no recibe del dispositivo un mensaje de latido en el tiempo esperado, da por supuesto que hay un problema con un dispositivo.

Una implementación más compleja podría incluir la información de [Azure Monitor](../azure-monitor/index.yml) y [Azure Resource Health](../service-health/resource-health-overview.md) para identificar los dispositivos que están intentando conectarse o comunicarse sin éxito. Consulte la guía [Monitor con diagnósticos](iot-hub-monitor-resource-health.md). Al implementar el patrón de latidos, asegúrese de echar un vistazo a [las cuotas y limitaciones de IoT Hub](iot-hub-devguide-quotas-throttling.md).

> [!NOTE]
> Si una solución de IoT utiliza el estado de conexión de los dispositivos únicamente para determinar si enviar mensajes de la nube a los dispositivos y los mensajes no se difunden a grandes conjuntos de dispositivos, considere usar el patrón más sencillo de un *tiempo de expiración breve*. Con este patrón se consigue el mismo resultado que con el mantenimiento de un registro del estado de la conexión de los dispositivos con el patrón de latido, a la vez que resulta más eficiente. Cuando solicita confirmaciones de mensajes, IoT Hub puede notificarle sobre qué dispositivos pueden recibir mensajes y cuáles no.

## <a name="device-and-module-lifecycle-notifications"></a>Notificaciones de ciclo de vida de dispositivo y módulo

IoT Hub puede notificar a la solución de IoT la creación o eliminación de la identidad de un dispositivo mediante el envío de notificaciones de ciclo de vida de dispositivo. Para ello, la solución de IoT debe crear una ruta y establecer el origen de datos igual a *DeviceLifecycleEvents* o *ModuleLifecycleEvents*. De forma predeterminada, no se envían notificaciones de ciclo de vida, es decir, no existen previamente tales rutas. El mensaje de notificaciones incluye propiedades y el cuerpo.

Propiedades: Las propiedades del sistema de mensajes tienen como prefijo el símbolo `$`.

Mensaje de notificación del dispositivo:

| Nombre | Value |
| --- | --- |
|$content-type | application/json |
|$iothub-enqueuedtime |  Hora de envío de la notificación |
|$iothub-message-source | deviceLifecycleEvents |
|$content-encoding | utf-8 |
|opType | **createDeviceIdentity** o **deleteDeviceIdentity** |
|hubName | Nombre de IoT Hub |
|deviceId | Id. del dispositivo |
|operationTimestamp | Marca de tiempo ISO8601 de operación |
|iothub-message-schema | deviceLifecycleNotification |

Cuerpo: Esta sección está en formato JSON y representa el dispositivo gemelo de la identidad del dispositivo creada. Por ejemplo,

```json
{
    "deviceId":"11576-ailn-test-0-67333793211",
    "etag":"AAAAAAAAAAE=",
    "properties": {
        "desired": {
            "$metadata": {
                "$lastUpdated": "2016-02-30T16:24:48.789Z"
            },
            "$version": 1
        },
        "reported": {
            "$metadata": {
                "$lastUpdated": "2016-02-30T16:24:48.789Z"
            },
            "$version": 1
        }
    }
}
```
Mensaje de notificación del módulo:

| Nombre | Value |
| --- | --- |
$content-type | application/json |
$iothub-enqueuedtime |  Hora de envío de la notificación |
$iothub-message-source | moduleLifecycleEvents |
$content-encoding | utf-8 |
opType | **createModuleIdentity** o **deleteModuleIdentity** |
hubName | Nombre de IoT Hub |
moduleId | Identificador del módulo. |
operationTimestamp | Marca de tiempo ISO8601 de operación |
iothub-message-schema | moduleLifecycleNotification |

Cuerpo: esta sección está en formato JSON y representa el gemelo de la identidad de módulo creada. Por ejemplo,

```json
{
    "deviceId":"11576-ailn-test-0-67333793211",
    "moduleId":"tempSensor",
    "etag":"AAAAAAAAAAE=",
    "properties": {
        "desired": {
            "$metadata": {
                "$lastUpdated": "2016-02-30T16:24:48.789Z"
            },
            "$version": 1
        },
        "reported": {
            "$metadata": {
                "$lastUpdated": "2016-02-30T16:24:48.789Z"
            },
            "$version": 1
        }
    }
}
```

## <a name="device-identity-properties"></a>Propiedades de identidad del dispositivo

Las identidades de dispositivos se representan como documentos JSON con las propiedades siguientes:

| Propiedad | Opciones | Descripción |
| --- | --- | --- |
| deviceId |necesarias, de solo lectura en actualizaciones |Una cadena que distingue entre mayúsculas y minúsculas (de hasta 128 caracteres) de caracteres alfanuméricos ASCII de 7 bits más determinados caracteres especiales: `- . + % _ # * ? ! ( ) , : = @ $ '`. |
| generationId |requerido, de solo lectura |Una cadena de hasta 128 caracteres que distingue mayúsculas y minúsculas generada por el centro de IoT. Este valor se usa para distinguir dispositivos con el mismo **deviceId**, cuando se han eliminado y vuelto a crear. |
| ETag |requerido, de solo lectura |Una cadena que representa un valor de ETag débil para la identidad del dispositivo, como [RFC7232](https://tools.ietf.org/html/rfc7232). |
| auth |opcional |Un objeto compuesto que contiene material de seguridad e información de autenticación. |
| auth.symkey |opcional |Un objeto compuesto que contiene una clave principal y una secundaria, almacenadas en formato base64. |
| status |requerido |Indicador de acceso Puede ser **Habilitado** o **Deshabilitado**. Si está **Habilitado**, el dispositivo se puede conectar. Si está **Dishabilitado**, este dispositivo no puede obtener acceso a ningún punto de conexión accesible desde el dispositivo. |
| statusReason |opcional |Una cadena de 128 caracteres que almacena el motivo del estado de identidad del dispositivo. Se permiten todos los caracteres UTF-8. |
| statusUpdateTime |solo lectura |Un indicador temporal, que muestra la fecha y hora de la última actualización de estado. |
| connectionState |solo lectura |Un campo que indica el estado de la conexión: **Conectado** o **Desconectado**. Este campo representa la vista de IoT Hub del estado de conexión del dispositivo. **Importante**: Este campo debe usarse solo para fines de desarrollo o depuración. El estado de conexión se actualiza solo para dispositivos que usen AMQP o MQTT. Además, se basa en pings de nivel de protocolo (pings MQTT o pings AMQP) y puede tener un retraso de 5 minutos como máximo. Por estos motivos es posible que haya falsos positivos, como dispositivos que se notifican como conectados pero que están desconectados. |
| connectionStateUpdatedTime |solo lectura |Un indicador temporal que muestra la fecha y hora de la última vez que se actualizó el estado de conexión. |
| lastActivityTime |solo lectura |Un indicador temporal que muestra la fecha y hora de la última vez que el dispositivo se conectó, recibió o envió un mensaje. |

> [!NOTE]
> El estado de la conexión solo puede representar la vista del IoT Hub del estado de la conexión. Las actualizaciones de este estado se pueden retrasar, dependiendo de las configuraciones y las condiciones de red.

> [!NOTE]
> Actualmente los SDK del dispositivo no admiten el uso de los caracteres `+` y `#` en **deviceId**.

## <a name="module-identity-properties"></a>Propiedades de la identidad de módulo

Las identidades de módulos se representan como documentos JSON con las propiedades siguientes:

| Propiedad | Opciones | Descripción |
| --- | --- | --- |
| deviceId |necesarias, de solo lectura en actualizaciones |Una cadena que distingue entre mayúsculas y minúsculas (de hasta 128 caracteres) de caracteres alfanuméricos ASCII de 7 bits más determinados caracteres especiales: `- . + % _ # * ? ! ( ) , : = @ $ '`. |
| moduleId |necesarias, de solo lectura en actualizaciones |Una cadena que distingue entre mayúsculas y minúsculas (de hasta 128 caracteres) de caracteres alfanuméricos ASCII de 7 bits más determinados caracteres especiales: `- . + % _ # * ? ! ( ) , : = @ $ '`. |
| generationId |requerido, de solo lectura |Una cadena de hasta 128 caracteres que distingue mayúsculas y minúsculas generada por el centro de IoT. Este valor se usa para distinguir dispositivos con el mismo **deviceId**, cuando se han eliminado y vuelto a crear. |
| ETag |requerido, de solo lectura |Una cadena que representa un valor de ETag débil para la identidad del dispositivo, como [RFC7232](https://tools.ietf.org/html/rfc7232). |
| auth |opcional |Un objeto compuesto que contiene material de seguridad e información de autenticación. |
| auth.symkey |opcional |Un objeto compuesto que contiene una clave principal y una secundaria, almacenadas en formato base64. |
| status |requerido |Indicador de acceso Puede ser **Habilitado** o **Deshabilitado**. Si está **Habilitado**, el dispositivo se puede conectar. Si está **Dishabilitado**, este dispositivo no puede obtener acceso a ningún punto de conexión accesible desde el dispositivo. |
| statusReason |opcional |Una cadena de 128 caracteres que almacena el motivo del estado de identidad del dispositivo. Se permiten todos los caracteres UTF-8. |
| statusUpdateTime |solo lectura |Un indicador temporal, que muestra la fecha y hora de la última actualización de estado. |
| connectionState |solo lectura |Un campo que indica el estado de la conexión: **Conectado** o **Desconectado**. Este campo representa la vista de IoT Hub del estado de conexión del dispositivo. **Importante**: Este campo debe usarse solo para fines de desarrollo o depuración. El estado de conexión se actualiza solo para dispositivos que usen AMQP o MQTT. Además, se basa en pings de nivel de protocolo (pings MQTT o pings AMQP) y puede tener un retraso de 5 minutos como máximo. Por estos motivos es posible que haya falsos positivos, como dispositivos que se notifican como conectados pero que están desconectados. |
| connectionStateUpdatedTime |solo lectura |Un indicador temporal que muestra la fecha y hora de la última vez que se actualizó el estado de conexión. |
| lastActivityTime |solo lectura |Un indicador temporal que muestra la fecha y hora de la última vez que el dispositivo se conectó, recibió o envió un mensaje. |

> [!NOTE]
> Actualmente los SDK del dispositivo no admiten el uso de los caracteres `+` y `#` en **deviceId** y **moduleId**.

## <a name="additional-reference-material"></a>Material de referencia adicional

Otros temas de referencia en la guía del desarrollador de IoT Hub son los siguientes:

* En [Puntos de conexión de IoT Hub](iot-hub-devguide-endpoints.md) se describen los diferentes puntos de conexión que expone cada centro de IoT Hub para las operaciones en tiempo de ejecución y de administración.

* En [Cuotas y limitación](iot-hub-devguide-quotas-throttling.md) se describen las cuotas y el comportamiento de limitación que se aplican al servicio IoT Hub.

* En [SDK de dispositivo y servicio IoT de Azure](iot-hub-devguide-sdks.md) se muestran los diversos SDK de lenguaje que puede usar para desarrollar aplicaciones para dispositivo y de servicio que interactúen con IoT Hub.

* En [Lenguaje de consulta de IoT Hub](iot-hub-devguide-query-language.md) se describe el lenguaje de consulta que se puede usar para recuperar información de IoT Hub sobre los trabajos y dispositivos gemelos.

* En [Compatibilidad con MQTT de IoT Hub](iot-hub-mqtt-support.md) se proporciona más información sobre la compatibilidad de IoT Hub con el protocolo MQTT.

## <a name="next-steps"></a>Pasos siguientes

Ahora que ha aprendido a usar el registro de identidad de IoT Hub, pueden interesarle los siguientes temas de la Guía para desarrolladores de IoT Hub:

* [Control del acceso a IoT Hub](iot-hub-devguide-security.md)

* [Uso de dispositivos gemelos para sincronizar el estado y las configuraciones](iot-hub-devguide-device-twins.md)

* [Invocación de un método directo en un dispositivo](iot-hub-devguide-direct-methods.md)

* [Programación de trabajos en varios dispositivos](iot-hub-devguide-jobs.md)

Para probar algunos de los conceptos descritos en este artículo, vea el siguiente tutorial de IoT Hub:

* [Introducción a Azure IoT Hub](quickstart-send-telemetry-dotnet.md)

Para explorar el uso del servicio IoT Hub Device Provisioning para habilitar el aprovisionamiento Just-In-Time sin intervención del usuario, vea: 

* [Servicio Azure IoT Hub Device Provisioning](https://azure.microsoft.com/documentation/services/iot-dps)
