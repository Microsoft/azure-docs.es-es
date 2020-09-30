---
title: Uso de los modelos IoT Plug and Play en una solución | Microsoft Docs
description: Como generador de soluciones, obtenga información sobre cómo puede usar los modelos IoT Plug and Play en la solución de IoT.
author: arunmannengal
ms.author: arunmann
ms.date: 07/23/2020
ms.topic: conceptual
ms.service: iot-pnp
services: iot-pnp
ms.openlocfilehash: 9752589c8863cc911369225d268035d9f61c0273
ms.sourcegitcommit: 3fc3457b5a6d5773323237f6a06ccfb6955bfb2d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/11/2020
ms.locfileid: "90032034"
---
# <a name="use-iot-plug-and-play-models-in-an-iot-solution"></a>Uso de los modelos IoT Plug and Play en una solución de IoT

En este artículo se describe cómo, puede identificar el id. de modelo de un dispositivo IoT Plug and Play y recuperar su definición de modelo en una solución de IoT.

Existen dos grandes categorías para una solución de IoT:

- Una *solución compilada específicamente* funciona con un conjunto conocido de modelos para los dispositivos IoT Plug and Play que se conectarán a la solución. Estos modelos se usan al desarrollar la solución.

- Las soluciones *basadas en modelos* pueden funcionar con el modelo de cualquier dispositivo IoT Plug and Play. La creación de una solución basada en modelos es más compleja, pero la ventaja es que la solución funcionará con cualquier dispositivo que agregue en el futuro. Una solución de IoT basada en modelos puede recuperar un modelo y usarlo para determinar la telemetría, las propiedades y los comandos que implementa el dispositivo.

Para usar un modelo IoT Plug and Play, una solución de IoT realiza lo siguiente:

1. Identifica el id. de modelo del modelo que implementó el dispositivo IoT Plug and Play conectado a la solución.

1. Usa el id. del modelo para recuperar la definición del modelo del dispositivo conectado desde un repositorio de modelos o un almacén personalizado.

## <a name="identify-model-id"></a>Identificación del id. de modelo

Cuando un dispositivo IoT Plug and Play se conecta a IoT Hub, registra el id. de modelo del modelo que se implementa con IoT Hub.

IoT Hub proporciona a la solución el id. del modelo de dispositivo como parte del flujo de conexión del dispositivo.

Una solución puede obtener el id. de modelo del dispositivo IoT Plug and Play mediante uno de los tres métodos siguientes:

### <a name="get-device-twin-api"></a>Get Device Twin API

La solución puede usar la API [Get Device Twin](https://docs.microsoft.com/java/api/com.microsoft.azure.sdk.iot.device.deviceclient.getdevicetwin?view=azure-java-stable) para recuperar el id. de modelo del dispositivo IoT Plug and Play.

En el siguiente fragmento de código de respuesta del dispositivo gemelo, `modelId` contiene el id. de modelo de un dispositivo IoT Plug and Play:

```json
{
    "deviceId": "sample-device",
    "etag": "AAAAAAAAAAc=",
    "deviceEtag": "NTk0ODUyODgx",
    "status": "enabled",
    "statusUpdateTime": "0001-01-01T00:00:00Z",
    "connectionState": "Disconnected",
    "lastActivityTime": "2020-07-17T06:12:26.8402249Z",
    "cloudToDeviceMessageCount": 0,
    "authenticationType": "sas",
    "x509Thumbprint": {
        "primaryThumbprint": null,
        "secondaryThumbprint": null
    },
    "modelId": "dtmi:com:example:TemperatureController;1",
    "version": 15,
    "properties": {...}
    }
}
```

### <a name="get-digital-twin-api"></a>Get Digital Twin API

La solución puede usar la API [Get Digital Twin](https://docs.microsoft.com/rest/api/iothub/service/digitaltwin/getdigitaltwin) para recuperar el id. de modelo del modelo que implementó el dispositivo IoT Plug and Play.

En el siguiente fragmento de código de respuesta del gemelo digital, `$metadata.$model` contiene el id. de modelo de un dispositivo IoT Plug and Play:

```json
{
    "$dtId": "sample-device",
    "$metadata": {
        "$model": "dtmi:com:example:TemperatureController;1",
        "serialNumber": {
            "lastUpdateTime": "2020-07-17T06:10:31.9609233Z"
        }
    }
}
```

### <a name="digital-twin-change-event-notification"></a>Notificación de eventos de cambio del gemelo digital

Una conexión del dispositivo da como resultado una notificación del [evento de cambio de gemelo digital](concepts-digital-twin.md#digital-twin-change-events). Una solución debe suscribirse a esta notificación de eventos. Para información sobre cómo habilitar el enrutamiento para eventos de gemelo digital, consulte [Uso del enrutamiento de mensajes de IoT Hub para enviar mensajes del dispositivo a la nube a distintos puntos de conexión](../iot-hub/iot-hub-devguide-messages-d2c.md#non-telemetry-events).

La solución puede usar el evento que se muestra en el siguiente fragmento de código para obtener información sobre el dispositivo IoT Plug and Play que se conecta y obtiene su id. de modelo:

```json
iothub-connection-device-id:sample-device
iothub-enqueuedtime:7/22/2020 8:02:27 PM
iothub-message-source:digitalTwinChangeEvents
correlation-id:100f322dc2c5
content-type:application/json-patch+json
content-encoding:utf-8
[
  {
    "op": "replace",
    "path": "/$metadata/$model",
    "value": "dtmi:com:example:TemperatureController;1"
  }
]
```

## <a name="retrieve-a-model-definition"></a>Recuperación de la definición del modelo

Una solución usa el id. de modelo identificado anteriormente para recuperar la definición de modelo correspondiente.

Una solución puede obtener la definición del modelo mediante una de las siguientes opciones:

### <a name="model-repository"></a>Repositorio de modelos

Las soluciones pueden usar el [repositorio de modelos](concepts-model-repository.md) para recuperar modelos. Los generadores de dispositivos o de soluciones deben cargar sus modelos en el repositorio de antemano, para que la solución pueda recuperarlos.

Una vez haya identificado el id. de modelo de una nueva conexión de dispositivo, siga estos pasos:

1. Recupere la definición del modelo mediante el id. de modelo del repositorio de modelos. Para obtener más información, consulte [Obtención de modelos](https://docs.microsoft.com/rest/api/iothub/digitaltwinmodelrepositoryservice/getmodelasync/getmodelasync).

1. Con la definición de modelo del dispositivo conectado, puede enumerar las capacidades del dispositivo.

1. Con las funcionalidades enumeradas del dispositivo, puede permitir a los usuarios [interactuar con el dispositivo](quickstart-service-node.md).

### <a name="custom-store"></a>Almacén personalizado

Las soluciones pueden almacenar estas definiciones de modelo en un sistema de archivos local, en un almacén de archivos público o usar una implementación personalizada.

Una vez haya identificado el id. de modelo de una nueva conexión de dispositivo, siga estos pasos:

1. Recupere la definición del modelo mediante el id. de modelo del almacén personalizado.

1. Con la definición de modelo del dispositivo conectado, puede enumerar las capacidades del dispositivo. 

1. Con las funcionalidades enumeradas del dispositivo, puede permitir a los usuarios [interactuar con el dispositivo](quickstart-service-node.md).  

## <a name="next-steps"></a>Pasos siguientes

Ahora que ha aprendido a integrar los modelos de IoT Plug and Play en una solución de IoT, le sugerimos realizar estos pasos:

- [Interacción con un dispositivo desde su solución](quickstart-service-node.md)
- [API de REST de gemelo digital de IoT](https://docs.microsoft.com/rest/api/iothub/service/digitaltwin)
- [Azure IoT Explorer](howto-use-iot-explorer.md)
