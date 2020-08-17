---
title: Dispositivos gemelos de Azure IoT Hub | Microsoft Docs
description: 'Guía para desarrolladores: uso de dispositivos gemelos para sincronizar los datos de estado y configuración entre IoT Hub y sus dispositivos'
author: ash2017
manager: philmea
ms.author: asrastog
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 02/01/2020
ms.custom:
- mqtt
- 'Role: Cloud Development'
ms.openlocfilehash: 4887315ddef3f15ee3f6ef5ad80cf8df8b1dcd34
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87327776"
---
# <a name="understand-and-use-device-twins-in-iot-hub"></a>Dispositivos gemelos en IoT Hub

Los *dispositivos gemelos* son documentos JSON que almacenan información acerca del estado del dispositivo, incluidos metadatos, configuraciones y condiciones. Azure IoT Hub mantiene un dispositivo gemelo para cada dispositivo que se conecta a IoT Hub. 

[!INCLUDE [iot-hub-basic](../../includes/iot-hub-basic-whole.md)]

En este artículo se describe:

* La estructura del dispositivo gemelo: *etiquetas*, las propiedades *deseadas* y las *notificadas*.
* Las operaciones que las aplicaciones de dispositivo y los back-ends pueden realizar en los dispositivos gemelos.

Use los dispositivos gemelos para:

* Almacenar metadatos específicos de dispositivo en la nube. Por ejemplo, la ubicación de implementación de una máquina expendedora.

* Notificar la información sobre el estado actual, como las funcionalidades y las condiciones disponibles de la aplicación del dispositivo. Por ejemplo, un dispositivo está conectado con su instancia de IoT Hub a través de un móvil o de WiFi.

* Sincronizar el estado de flujos de trabajo de larga duración entre la aplicación del dispositivo y el back-end. Por ejemplo, cuando el back-end de la solución especifica la nueva versión de firmware que se va a instalar y la aplicación de dispositivo informa de las distintas fases del proceso de actualización.

* Consultar los metadatos, la configuración o el estado del dispositivo.

Para obtener instrucciones sobre el uso de propiedades notificadas, mensajes de dispositivo a nube o carga de archivos, consulte [Guía de comunicación de dispositivo a nube](iot-hub-devguide-d2c-guidance.md).

Para obtener instrucciones sobre el uso de propiedades deseadas, los métodos directos o los mensajes de nube a dispositivo, consulte [Guía de comunicación de nube a dispositivo](iot-hub-devguide-c2d-guidance.md).

## <a name="device-twins"></a>Dispositivos gemelos

Los dispositivos gemelos almacenan información relacionada con el dispositivo que:

* El dispositivo y el back end pueden usar para sincronizar la configuración y las condiciones del dispositivo.

* El back-end de la solución se puede usar para consultar e identificar operaciones de larga duración.

El ciclo de vida de un dispositivo gemelo está vinculado a la [identidad del dispositivo](iot-hub-devguide-identity-registry.md) correspondiente. Los dispositivos gemelos se crean y se eliminan implícitamente cuando se crea o se elimina una identidad de dispositivo en IoT Hub.

Un dispositivo gemelo es un documento JSON que incluye:

* **Etiquetas**. Una sección del documento JSON en la que el back-end de solución puede leer y escribir. Las etiquetas no son visibles para las aplicaciones de dispositivo.

* **Propiedades deseadas**. Se usan en conjunción con las propiedades notificadas para sincronizar la configuración o la condición del dispositivo. El back-end de solución puede establecer propiedades deseadas, y la aplicación de dispositivo puede leerlas. La aplicación de dispositivo también puede recibir notificaciones de los cambios en las propiedades deseadas.

* **Propiedades notificadas**. Se usan en conjunción con las propiedades deseadas para sincronizar la configuración o la condición del dispositivo. La aplicación de dispositivo puede establecer propiedades notificadas, y el back-end de solución puede leerlas y consultarlas.

* **Propiedades de identidad del dispositivo**. La raíz del documento JSON del dispositivo gemelo contiene las propiedades de solo lectura de la identidad de dispositivo correspondiente, almacenadas en el [registro de identidad de dispositivo](iot-hub-devguide-identity-registry.md). No se incluirán las propiedades `connectionStateUpdatedTime` y `generationId`.

![Captura de pantalla de las propiedades de dispositivos gemelos](./media/iot-hub-devguide-device-twins/twin.png)

En el ejemplo siguiente se muestra un documento JSON del dispositivo gemelo:

```json
{
    "deviceId": "devA",
    "etag": "AAAAAAAAAAc=", 
    "status": "enabled",
    "statusReason": "provisioned",
    "statusUpdateTime": "0001-01-01T00:00:00",
    "connectionState": "connected",
    "lastActivityTime": "2015-02-30T16:24:48.789Z",
    "cloudToDeviceMessageCount": 0, 
    "authenticationType": "sas",
    "x509Thumbprint": {     
        "primaryThumbprint": null, 
        "secondaryThumbprint": null 
    }, 
    "version": 2, 
    "tags": {
        "$etag": "123",
        "deploymentLocation": {
            "building": "43",
            "floor": "1"
        }
    },
    "properties": {
        "desired": {
            "telemetryConfig": {
                "sendFrequency": "5m"
            },
            "$metadata" : {...},
            "$version": 1
        },
        "reported": {
            "telemetryConfig": {
                "sendFrequency": "5m",
                "status": "success"
            },
            "batteryLevel": 55,
            "$metadata" : {...},
            "$version": 4
        }
    }
}
```

En el objeto raíz están las propiedades de identidad del sistema y los objetos de contenedor para `tags` y las propiedades `reported` y `desired`. El contenedor `properties` incluye algunos elementos de solo lectura (`$metadata`, `$etag` y `$version`) descritos en las secciones [Metadatos de dispositivo gemelo](iot-hub-devguide-device-twins.md#device-twin-metadata) y [Simultaneidad optimista](iot-hub-devguide-device-twins.md#optimistic-concurrency).

### <a name="reported-property-example"></a>Ejemplo de propiedad notificada

En el ejemplo anterior, el dispositivo gemelo contiene una propiedad `batteryLevel` notificada por la aplicación de dispositivo. Esta propiedad permite consultar y operar en los dispositivos en función del último nivel de batería notificado. Otros ejemplos incluyen la aplicación de dispositivo notificando las funcionalidades del dispositivo o las opciones de conectividad.

> [!NOTE]
> Las propiedades notificadas simplifican los escenarios donde el back-end de la solución está interesado en el último valor conocido de una propiedad. Use [mensajes de dispositivo a nube](iot-hub-devguide-messages-d2c.md) si el back-end de la solución debe procesar la telemetría del dispositivo en forma de secuencias de eventos con marca de tiempo, por ejemplo, series temporales.

### <a name="desired-property-example"></a>Ejemplo de propiedad deseada

En el ejemplo anterior, el back-end de la solución y la aplicación de dispositivo usan las propiedades deseadas y notificadas del dispositivo gemelo `telemetryConfig` para sincronizar la configuración de telemetría de este dispositivo. Por ejemplo:

1. El back-end de la solución establece la propiedad deseada con el valor de configuración deseado. Esta es la parte del documento con el conjunto de propiedad deseada:

   ```json
   "desired": {
       "telemetryConfig": {
           "sendFrequency": "5m"
       },
       ...
   },
   ```

2. La aplicación de dispositivo recibe una notificación del cambio inmediatamente si está conectado, o en cuanto vuelva a conectarse. Después, la aplicación de dispositivo notifica la configuración actualizada (o una condición de error mediante la propiedad `status`. Esta es la parte de las propiedades notificadas:

   ```json
   "reported": {
       "telemetryConfig": {
           "sendFrequency": "5m",
           "status": "success"
       }
       ...
   }
   ```

3. El back-end de la solución puede realizar un seguimiento de los resultados de la operación de configuración en varios dispositivos; para ello, [consulta](iot-hub-devguide-query-language.md) los dispositivos gemelos.

> [!NOTE]
> Los fragmentos de código anteriores son ejemplos, optimizados para mejorar su legibilidad, de una manera de codificar una configuración de dispositivo y su estado. IoT Hub no impone un esquema específico para las propiedades deseadas y notificadas en los dispositivos gemelos.
> 

Se pueden usar los dispositivos gemelos para sincronizar operaciones de larga duración, como las actualizaciones de firmware. Para más información sobre cómo usar las propiedades para sincronizar y realizar un seguimiento de las operaciones de larga duración en todos los dispositivo consulte [Uso de las propiedades deseadas para configurar dispositivos](tutorial-device-twins.md).

## <a name="back-end-operations"></a>Operaciones de back-end

Para trabajar en el back-end de la solución, el dispositivo gemelo usa las siguientes operaciones atómicas expuestas mediante HTTPS:

* **Recuperación del dispositivo gemelo por el identificador**. Esta operación devuelve el documento del dispositivo gemelo, incluidas las etiquetas y las propiedades del sistema, deseadas y notificadas.

* **Actualización parcial de los dispositivos gemelos**. Esta operación permite que el back-end de la solución actualice parcialmente las etiquetas o las propiedades deseadas del dispositivo gemelo. La actualización parcial se expresa como un documento JSON que agrega o actualiza cualquier propiedad. Las propiedades establecidas en `null` se quitan. El ejemplo siguiente crea una nueva propiedad deseada con el valor `{"newProperty": "newValue"}`, sobrescribe el valor existente de `existingProperty` con `"otherNewValue"`, y quita `otherOldProperty`. No se realiza ningún cambio en otras etiquetas o propiedades deseadas existentes:

   ```json
   {
       "properties": {
           "desired": {
               "newProperty": {
                   "nestedProperty": "newValue"
               },
               "existingProperty": "otherNewValue",
               "otherOldProperty": null
           }
       }
   }
   ```

* **Reemplazar propiedades deseadas**. Esta operación permite que el back-end de la solución sobrescriba completamente todas las propiedades deseadas y sustituya un nuevo documento JSON para `properties/desired`.

* **Reemplazar etiquetas**. Esta operación permite que el back-end de la solución sobrescriba completamente todas las etiquetas y sustituya un nuevo documento JSON para `tags`.

* **Recibir notificaciones gemelas**. Esta operación permite que el back-end de la solución reciba una notificación cuando se modifique la gemela. Para ello, la solución de IoT debe crear una ruta y establecer el origen de datos igual a *twinChangeEvents*. De forma predeterminada, no existen previamente tales rutas, por tanto, no se envían notificaciones gemelas. Si la tasa de cambio es demasiado alta, o por otras razones, como errores internos, IoT Hub podría enviar una sola notificación que contiene todos los cambios. Por lo tanto, si la aplicación necesita registro y auditoría confiables de todos los estados intermedios, debe usar mensajes del dispositivo a la nube. El mensaje de notificaciones gemelas incluye propiedades y el cuerpo.

  - Propiedades

    | Nombre | Value |
    | --- | --- |
    $content-type | application/json |
    $iothub-enqueuedtime |  Hora de envío de la notificación |
    $iothub-message-source | twinChangeEvents |
    $content-encoding | utf-8 |
    deviceId | Id. del dispositivo |
    hubName | Nombre de IoT Hub |
    operationTimestamp | Marca de tiempo [ISO8601](https://en.wikipedia.org/wiki/ISO_8601) de operación |
    iothub-message-schema | twinChangeNotification |
    opType | "replaceTwin" o "updateTwin" |

    Las propiedades del sistema de mensajes tienen como prefijo el símbolo `$`.

  - Body
        
    Esta sección incluye todos los cambios gemelos en formato JSON. Se usa el mismo formato que una revisión, con la diferencia de que puede contener todas las secciones gemelas: etiquetas, propiedades notificadas, propiedades deseadas, y que contiene los elementos "$metadata". Por ejemplo,

    ```json
    {
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

Todas las operaciones anteriores admiten la [simultaneidad optimista](iot-hub-devguide-device-twins.md#optimistic-concurrency) y requieren el permiso **ServiceConnect**, tal y como se define en [Controlar el acceso a IoT Hub](iot-hub-devguide-security.md).

Además de estas operaciones, el back-end de soluciones puede hacer lo siguiente:

* Consultar los dispositivos gemelos mediante el [lenguaje de consultas de IoT Hub](iot-hub-devguide-query-language.md) del tipo SQL.

* Realizar operaciones en conjuntos grandes de dispositivos gemelos usando [trabajos](iot-hub-devguide-jobs.md).

## <a name="device-operations"></a>Operaciones de dispositivo

La aplicación de dispositivo aplica las siguientes operaciones atómicas en el dispositivo gemelo:

* **Recuperación del dispositivo gemelo**. Esta operación devuelve el documento del dispositivo gemelo (incluidas las propiedades del sistema deseadas y notificadas) para el dispositivo conectado actualmente. (Las etiquetas no son visibles para las aplicaciones de dispositivo).

* **Actualizar parcialmente propiedades notificadas**. Esta operación permite la actualización parcial de las propiedades notificadas del dispositivo conectado actualmente. Esta operación usa el mismo formato de actualización JSON que el back-end de la solución usa para una actualización parcial de propiedades deseadas.

* **Observar las propiedades deseadas**. El dispositivo conectado actualmente puede elegir recibir notificaciones de las actualizaciones de las propiedades deseadas cuando se produzcan. El dispositivo recibe la misma forma de actualización (sustitución parcial o completa) ejecutada por el back-end de la solución.

Todas las operaciones anteriores requieren el permiso **DeviceConnect**, tal y como se define en el artículo [Controlar el acceso a IoT Hub](iot-hub-devguide-security.md).

Los [SDK de dispositivos IoT de Azure](iot-hub-devguide-sdks.md) permiten usar fácilmente las operaciones anteriores desde numerosos lenguajes y plataformas. Para más información detallada sobre los tipos primitivos de IoT Hub para la sincronización de propiedades deseadas, consulte [Flujo de reconexión de dispositivos](iot-hub-devguide-device-twins.md#device-reconnection-flow).

## <a name="tags-and-properties-format"></a>Formato de etiquetas y propiedades

Las etiquetas y las propiedades deseadas y notificadas son objetos JSON con las siguientes restricciones:

* **Claves**: Todas las claves en objetos JSON tienen codificación UTF-8, con distinción de mayúsculas y minúsculas, y una longitud de hasta 1 KB. Entre los caracteres permitidos no se incluyen los caracteres de control UNICODE (segmentos C0 y C1), ni `.`, `$` y SP.

* **Valores**: Todos los valores en un objeto JSON pueden ser de los siguientes tipos JSON: booleano, número, cadena, objeto. No se permiten matrices.

    * Los enteros pueden tener un valor mínimo de -4503599627370496 y un máximo de 4503599627370495.

    * Los valores de la cadena son codificados UTF-8 y pueden tener una longitud máxima de 4 KB.

* **Profundidad**: La profundidad máxima de los objetos JSON en etiquetas y propiedades deseadas y notificadas es 10. Por ejemplo, el objeto siguiente es válido:

   ```json
   {
       ...
       "tags": {
           "one": {
               "two": {
                   "three": {
                       "four": {
                           "five": {
                               "six": {
                                   "seven": {
                                       "eight": {
                                           "nine": {
                                               "ten": {
                                                   "property": "value"
                                               }
                                           }
                                       }
                                   }
                               }
                           }
                       }
                   }
               }
           }
       },
       ...
   }
   ```

## <a name="device-twin-size"></a>Tamaño del dispositivo gemelo

IoT Hub aplica un límite de tamaño de 8 KB en el valor de `tags` y un límite de tamaño de 32 KB cada uno en el valor de `properties/desired` y `properties/reported`. Estos totales son exclusivos de los elementos de solo lectura, como `$etag`, `$version` y `$metadata/$lastUpdated`.

El tamaño gemelo se calcula como sigue:

* Para cada propiedad del documento JSON, IoT Hub calcula y agrega de forma acumulativa la longitud del valor y la clave de la propiedad.

* Las claves de la propiedad se consideran cadenas codificadas UTF-8.

* Los valores de propiedad simples se consideran cadenas codificadas UTF-8, valores numéricos (8 bytes) o valores booleanos (4 bytes).

* El tamaño de las cadenas codificadas UTF-8 se calcula contando todos los caracteres, excepto los caracteres de control UNICODE (segmentos C0 y C1).

* Los valores de propiedad complejos (objetos anidados) se calculan en función del tamaño agregado de las claves de la propiedad y los valores de propiedad que contienen.

IoT Hub rechaza con un error todas las operaciones que podrían aumentar el tamaño de los documentos `tags`, `properties/desired` o `properties/reported` por encima del límite.

## <a name="device-twin-metadata"></a>Metadatos de dispositivo gemelo

IoT Hub conserva la marca de tiempo de la última actualización para cada objeto JSON en las propiedades deseadas y notificadas del dispositivo gemelo. Las marcas de tiempo están en formato UTC y se codifican con el formato [ISO8601](https://en.wikipedia.org/wiki/ISO_8601)`YYYY-MM-DDTHH:MM:SS.mmmZ`.

Por ejemplo:

```json
{
    ...
    "properties": {
        "desired": {
            "telemetryConfig": {
                "sendFrequency": "5m"
            },
            "$metadata": {
                "telemetryConfig": {
                    "sendFrequency": {
                        "$lastUpdated": "2016-03-30T16:24:48.789Z"
                    },
                    "$lastUpdated": "2016-03-30T16:24:48.789Z"
                },
                "$lastUpdated": "2016-03-30T16:24:48.789Z"
            },
            "$version": 23
        },
        "reported": {
            "telemetryConfig": {
                "sendFrequency": "5m",
                "status": "success"
            },
            "batteryLevel": "55%",
            "$metadata": {
                "telemetryConfig": {
                    "sendFrequency": "5m",
                    "status": {
                        "$lastUpdated": "2016-03-31T16:35:48.789Z"
                    },
                    "$lastUpdated": "2016-03-31T16:35:48.789Z"
                },
                "batteryLevel": {
                    "$lastUpdated": "2016-04-01T16:35:48.789Z"
                },
                "$lastUpdated": "2016-04-01T16:24:48.789Z"
            },
            "$version": 123
        }
    }
    ...
}
```

Esta información se conserva en todo los niveles (no solo en las hojas de la estructura JSON) para conservar las actualizaciones que quitan las claves de objeto.

## <a name="optimistic-concurrency"></a>Simultaneidad optimista

Tanto las etiquetas como las propiedades deseadas y notificadas admiten la simultaneidad optimista.
Las etiquetas tienen una ETag, según la norma [RFC7232](https://tools.ietf.org/html/rfc7232), que es la representación JSON de la etiqueta. Puede usar ETags en operaciones de actualización condicional desde el back-end de la solución para garantizar la coherencia.

Las propiedades deseadas y notificadas no tienen ETags, pero tienen un valor `$version` que se garantiza que será incremental. De forma similar a una ETag, la parte que realiza la actualización puede usar la versión para garantizar la coherencia de las actualizaciones. Por ejemplo, una aplicación de dispositivo para una propiedad notificada o la solución de back-end para una propiedad deseada.

Las versiones también son útiles cuando un agente de observación (por ejemplo, la aplicación de dispositivo que observa las propiedades deseadas) tiene que conciliar las carreras entre el resultado de una operación de recuperación y una notificación de actualización. En la sección [Flujo de reconexión de dispositivos](iot-hub-devguide-device-twins.md#device-reconnection-flow) se proporciona más información.

## <a name="device-reconnection-flow"></a>Flujo de reconexión de dispositivos

IoT Hub no conserva las notificaciones de actualización de las propiedades deseadas para los dispositivos desconectados. Se supone que un dispositivo que se está conectando debe recuperar el documento con todas las propiedades deseadas, además de suscribirse a las notificaciones de actualización. Dada la posibilidad de carreras entre las notificaciones de actualización y la recuperación completa, se debe asegurar el flujo siguiente:

1. La aplicación de dispositivo se conecta a un centro de IoT.
2. La aplicación de dispositivo se suscribe a las notificaciones de actualización de las propiedades deseadas.
3. La aplicación de dispositivo recupera el documento completo de las propiedades deseadas.

La aplicación de dispositivo puede pasar por alto todas las notificaciones cuyo valor de `$version` sea anterior o igual a la versión del documento completo recuperado. Este enfoque es posible porque IoT Hub garantiza que las versiones siempre se incrementan.

> [!NOTE]
> Esta lógica ya se implementó en los [SDK de dispositivo IoT de Azure](iot-hub-devguide-sdks.md). Esta descripción es útil solo si la aplicación del dispositivo no puede usar ninguno de los SDK de dispositivos de IoT de Azure y debe programar directamente la interfaz MQTT.
> 

## <a name="additional-reference-material"></a>Material de referencia adicional

Otros temas de referencia en la guía del desarrollador de IoT Hub son los siguientes:

* En el artículo [Puntos de conexión de IoT Hub](iot-hub-devguide-endpoints.md) se describen los diferentes puntos de conexión que expone cada centro de IoT Hub para las operaciones en tiempo de ejecución y de administración.

* En el artículo [Cuotas y limitación](iot-hub-devguide-quotas-throttling.md), se describen las cuotas que se aplican al servicio IoT Hub y el comportamiento de limitación esperado al usar el servicio.

* En el artículo sobre [SDK de dispositivos y servicio de Azure IoT](iot-hub-devguide-sdks.md) se muestran los SDK de los diferentes lenguajes que puede usar para desarrollar aplicaciones para dispositivo y de servicio que interactúen con IoT Hub.

* En el artículo [Referencia: Lenguaje de consulta de IoT Hub para dispositivos gemelos, trabajos y enrutamiento de mensajes](iot-hub-devguide-query-language.md), se describe el lenguaje de consulta de IoT Hub que se puede usar para recuperar información de IoT Hub sobre los dispositivos gemelos y trabajos.

* En el artículo sobre la [compatibilidad con MQTT de IoT Hub](iot-hub-mqtt-support.md), se proporciona más información sobre la compatibilidad de IoT Hub con el protocolo MQTT.

## <a name="next-steps"></a>Pasos siguientes

Ahora que ya conoce algo más sobre los dispositivos gemelos, quizás le interesen los siguientes temas de la guía para desarrolladores de IoT Hub:

* [Descripción y uso de módulos gemelos en IoT Hub](iot-hub-devguide-module-twins.md)
* [Invocación de un método directo en un dispositivo](iot-hub-devguide-direct-methods.md)
* [Programación de trabajos en varios dispositivos](iot-hub-devguide-jobs.md)

Para probar algunos de los conceptos descritos en este artículo, vea los siguientes tutoriales de IoT Hub:

* [Uso de dispositivos gemelos](iot-hub-node-node-twin-getstarted.md)
* [Uso de propiedades de dispositivos gemelos](tutorial-device-twins.md)
* [Administración de dispositivos con Azure IoT Tools para VS Code](iot-hub-device-management-iot-toolkit.md)
