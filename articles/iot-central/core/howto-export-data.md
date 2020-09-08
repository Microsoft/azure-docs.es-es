---
title: Exportación de los datos de Azure IoT Central | Microsoft Docs
description: Cómo exportar datos desde la aplicación de Azure IoT Central a Azure Event Hubs, Azure Service Bus y Azure Blob Storage
services: iot-central
author: viv-liu
ms.author: viviali
ms.date: 06/25/2020
ms.topic: how-to
ms.service: iot-central
manager: corywink
ms.openlocfilehash: 1428df124272816927c6bbbc4a242170c7f46c00
ms.sourcegitcommit: 98854e3bd1ab04ce42816cae1892ed0caeedf461
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "88008532"
---
# <a name="export-iot-data-to-destinations-in-azure-using-data-export-legacy"></a>Exportación de datos de IoT a destinos en Azure (método heredado)

> [!Note]
> Hay una nueva forma de exportar datos a IoT Central. Puede usar la nueva exportación de datos para filtrar y enriquecer los datos exportados y exportarlos a nuevos destinos, como los puntos de conexión de webhook. Más información sobre la nueva exportación de datos [aquí](./howto-use-data-export.md). Para información sobre las diferencias entre la nueva exportación de datos y la exportación de datos heredada, consulte la [tabla de comparación](./howto-use-data-export.md#comparison-of-legacy-data-export-and-new-data-export).

En este artículo se describe cómo usar la característica de exportación de datos en Azure IoT Central. Esta característica permite exportar los datos de forma continua a **Azure Event Hubs**, **Azure Service Bus** o **Azure Blob Storage**. La exportación de datos usa el formato JSON y pueden incluir telemetría, información del dispositivo e información de la plantilla de dispositivo. Use los datos exportados para:

- Información y análisis de la ruta de acceso activa. Esta opción incluye el desencadenamiento de reglas personalizadas en Azure Stream Analytics, el desencadenamiento de flujos de trabajo personalizados en Azure Logic Apps o su paso a través de Azure Functions para su transformación.
- Análisis en frío de rutas de acceso, como modelos de entrenamiento en Azure Machine Learning o análisis de tendencias a largo plazo en Microsoft Power BI.

> [!Note]
> Cuando se activa la exportación de datos, solo se obtienen los datos a partir de ese momento. Actualmente, no se pueden recuperar los datos del tiempo durante el que la exportación de datos estaba desactivada. Para conservar más datos históricos, active la exportación de datos al principio.

## <a name="prerequisites"></a>Prerrequisitos

Ser administrador en la aplicación de IoT Central o contar con permisos de exportación de datos.

## <a name="set-up-export-destination"></a>Configuración del destino de exportación

El destino de exportación debe existir antes de configurar la exportación de datos.

### <a name="create-event-hubs-namespace"></a>Creación de un espacio de nombres de Event Hubs

Si no tiene un espacio de nombres existente de Event Hubs al que exportar, siga estos pasos:

1. Cree un [espacio de nombres de Event Hubs en Azure Portal](https://ms.portal.azure.com/#create/Microsoft.EventHub). Puede encontrar más información en los [documentos de Azure Event Hubs](../../event-hubs/event-hubs-create.md).

2. Elija una suscripción. Puede exportar datos a otras suscripciones que no estén en la misma suscripción que la aplicación de IoT Central. En este caso se conecta mediante una cadena de conexión.

3. Cree un centro de eventos en el espacio de nombres de Event Hubs Vaya al espacio de nombres y seleccione **+ Centro de eventos** en la parte superior para crear una instancia de centro de eventos.

### <a name="create-service-bus-namespace"></a>Creación de un espacio de nombres de Service Bus

Si no tiene un espacio de nombres existente de Service Bus al que exportar, siga estos pasos:

1. Cree un [nuevo espacio de nombres de Service Bus en Azure Portal](https://ms.portal.azure.com/#create/Microsoft.ServiceBus.1.0.5). Puede encontrar más información en los [documentos de Azure Service Bus](../../service-bus-messaging/service-bus-create-namespace-portal.md).
2. Elija una suscripción. Puede exportar datos a otras suscripciones que no estén en la misma suscripción que la aplicación de IoT Central. En este caso se conecta mediante una cadena de conexión.

3. Para crear una cola o tema como destino de exportación, vaya al espacio de nombres de Service Bus y seleccione **+ Cola** o **+ Tema**.

Al elegir Service Bus como destino de exportación, las colas y los temas no deben tener habilitada la opción de detección de sesiones o de duplicados. Si cualquiera de estas opciones está habilitada, algunos mensajes no llegarán a la cola o tema.

### <a name="create-storage-account"></a>Crear cuenta de almacenamiento

Si no tiene una cuenta existente de Azure Storage a la que exportar, siga estos pasos:

1. Cree una [cuenta de almacenamiento en Azure Portal](https://ms.portal.azure.com/#create/Microsoft.StorageAccount-ARM). Puede obtener más información sobre cómo crear nuevas [cuentas de almacenamiento de Azure Blob ](https://aka.ms/blobdocscreatestorageaccount) o [cuentas de almacenamiento de Azure Data Lake Storage v2](../../storage/blobs/data-lake-storage-quickstart-create-account.md). La exportación de datos solo puede escribir datos en cuentas de almacenamiento que admiten blobs en bloques. En la lista siguiente se muestran los tipos de cuenta de almacenamiento compatibles conocidos:

    |Nivel de rendimiento|Tipo de cuenta|
    |-|-|
    |Estándar|Uso general V2|
    |Estándar|Uso general V1|
    |Estándar|Blob Storage|
    |Premium|Almacenamiento de blob en bloques|

2. Cree un contenedor en la cuenta de almacenamiento. Vaya a la cuenta de almacenamiento. En **Blob Service** seleccione **Examinar blobs**. Seleccione **+ Contenedor** en la parte superior para crear un contenedor.

## <a name="set-up-data-export"></a>Configuración de la exportación de datos

Ahora que tiene un destino al que exportar los datos, siga estos pasos para configurar la exportación de datos.

1. Inicie sesión en su aplicación de IoT Central.

2. En el panel izquierdo, seleccione **Exportación de datos**.

    > [!Tip]
    > Si no ve **Exportación de datos** en el panel izquierdo, no tiene permisos para configurar la exportación de datos en la aplicación. Hable con un administrador para configurar la exportación de datos.

3. Seleccione el botón **+ Nuevo**. Elija entre **Azure Blob Storage**, **Azure Event Hubs**, **Azure Service Bus Queue** o **Azure Service Bus Topic** como destino de la exportación. El número máximo de exportaciones por aplicación es cinco.

4. Especifique un nombre para la exportación. En el cuadro de lista desplegable, seleccione el **espacio de nombres** o **Escriba una cadena de conexión**.

    - Solo verá las cuentas de almacenamiento, los espacios de nombres de Event Hubs y los espacios de nombres de Service Bus de la misma suscripción que la de la aplicación de IoT Central. Si quiere exportar a un destino fuera de esta suscripción, elija **Escriba una cadena de conexión** y vea el paso 6.
    - Para las aplicaciones creadas con el plan de tarifas gratis, la única manera de configurar la exportación de datos es mediante una cadena de conexión. Las aplicaciones en el plan de tarifas gratis no tienen una suscripción a Azure asociada.

    ![Creación de un centro de eventos](media/howto-export-data/export-event-hub.png)

5. Elija un centro de eventos, una cola, un tema o un contenedor del cuadro de lista desplegable.

6. (Opcional) Si eligió **Escriba una cadena de conexión**, aparece un nuevo cuadro para que pegue la cadena de conexión. Para obtener la cadena de conexión para su:

    - Event Hubs o Service Bus, vaya al espacio de nombres en Azure Portal:
        - Para usar una cadena de conexión para todo el espacio de nombres:
            1. En **Configuración**, seleccione **Directivas de acceso compartido**.
            2. Cree una nueva clave o elija una clave existente que tenga permisos de **envío**.
            3. Copie la cadena de conexión principal o la secundaria.
        - Para usar una cadena de conexión para una instancia específica del centro de eventos o cola o tema de Service Bus, vaya a **Entidades > Event Hubs** o **Entidades > Colas** o **Entidades > Temas**. Elija una instancia específica y siga los mismos pasos anteriores para obtener una cadena de conexión.
    - Cuenta de almacenamiento, vaya a la cuenta de almacenamiento en Azure Portal:
        - Solo se admiten las cadenas de conexión para toda la cuenta de almacenamiento. No se admiten las cadenas de conexión limitadas a un único contenedor.
          1. En **Configuración**, seleccione **Claves de acceso**.
          2. Copie la cadena de conexión key1 o la cadena de conexión key2.

    Péguela en la cadena de conexión. Escriba la instancia o el **nombre del contenedor**, teniendo en cuenta que distingue mayúsculas de minúsculas.

7. En **Datos para exportar**, elija los tipos de datos para exportar y establezca el tipo en **Activado**.

8. Para activar la exportación de datos, asegúrese de que **Habilitado** esté **Activado**. Seleccione **Guardar**.

9. Transcurridos unos minutos, los datos aparecen en el destino elegido.

## <a name="export-contents-and-format"></a>Exportación del contenido y formato

Los datos de telemetría exportados contienen la totalidad del mensaje que los dispositivos enviaron a IoT Central, no solo los valores de los propios datos de telemetría. Los datos de dispositivo exportados contienen los cambios en las propiedades y los metadatos de todos los dispositivos; las plantillas de dispositivo exportadas contienen los cambios en todas las plantillas de dispositivo.

Para Event Hubs y Service Bus, los datos se exportan casi en tiempo real. Los datos se encuentran en la propiedad `body` y están en formato JSON. Vea ejemplos a continuación.

Para Blob Storage, los datos se exportan una vez por minuto y cada archivo contiene el lote de cambios desde el último archivo exportado. Los datos exportados se colocan en tres carpetas con formato JSON. Las rutas de acceso predeterminadas de la cuenta de almacenamiento son:

- Telemetría: _{contenedor}/{id. de aplicación}/telemetry/{AAAA}/{MM}/{dd}/{hh}/{mm}/{nombre de archivo}_
- Dispositivos: _{contenedor}/{id. de aplicación}/devices/{AAAA}/{MM}/{dd}/{hh}/{mm}/{nombre de archivo}_
- Plantillas de dispositivo: _{contenedor}/{id. de aplicación}/deviceTemplates/{AAAA}/{MM}/{dd}/{hh}/{mm}/{nombre de archivo}_

Para examinar los archivos exportados en Azure Portal, navegue hasta al archivo y seleccione la pestaña **Editar blob**.

## <a name="telemetry"></a>Telemetría

En el caso de Event Hubs y Service Bus, IoT Central exporta un mensaje nuevo rápidamente después de recibir el mensaje de un dispositivo. Cada mensaje exportado contiene el mensaje completo que el dispositivo ha enviado en la propiedad body en formato JSON.

Para Blob Storage, los mensajes se procesan por lotes y se exportan una vez por minuto. Los archivos exportados usan el mismo formato que los archivos de mensajes exportados por el [enrutamiento de mensajes de IoT Hub](../../iot-hub/tutorial-routing.md) a Blob Storage.

> [!NOTE]
> Para Blob Storage, asegúrese de que los dispositivos envíen mensajes que tengan `contentType: application/JSON` y `contentEncoding:utf-8` (o `utf-16`, `utf-32`). Consulte la [documentación de IoT Hub](../../iot-hub/iot-hub-devguide-routing-query-syntax.md#message-routing-query-based-on-message-body) para ver un ejemplo.

El dispositivo que envió la telemetría se representa mediante el identificador del dispositivo (consulte las secciones siguientes). Para obtener los nombres de los dispositivos, exporte los datos del dispositivo y ponga en correlación cada mensaje mediante el objeto **connectionDeviceId** que coincide con el objeto **deviceId** del mensaje del dispositivo.

En el ejemplo siguiente se muestra un mensaje recibido de un centro de eventos o una cola o tema de Service Bus:

```json
{
  "temp":81.129693132351775,
  "humid":59.488071477541247,
  "EventProcessedUtcTime":"2020-04-07T09:41:15.2877981Z",
  "PartitionId":0,
  "EventEnqueuedUtcTime":"2020-04-07T09:38:32.7380000Z"
}
```

Este mensaje no incluye el id. de dispositivo del dispositivo emisor.

Para recuperar el id. de dispositivo de los datos del mensaje en una consulta de Azure Stream Analytics, utilice la función [GetMetadataPropertyValue](https://docs.microsoft.com/stream-analytics-query/getmetadatapropertyvalue). Para un ejemplo, vea la consulta de [Extensión de Azure IoT Central con reglas personalizadas mediante Stream Analytics, Azure Functions y SendGrid](./howto-create-custom-rules.md).

Para recuperar el id. de dispositivo en un área de trabajo de Azure Databricks o Apache Spark, use [systemProperties](https://github.com/Azure/azure-event-hubs-spark/blob/master/docs/structured-streaming-eventhubs-integration.md). Para un ejemplo, vea el área de trabajo de Databricks en [Ampliación de Azure IoT Central con análisis personalizados mediante Azure Databricks](./howto-create-custom-analytics.md).

En el ejemplo siguiente se muestra un registro exportado en Blob Storage:

```json
{
  "EnqueuedTimeUtc":"2019-09-26T17:46:09.8870000Z",
  "Properties":{

  },
  "SystemProperties":{
    "connectionDeviceId":"<deviceid>",
    "connectionAuthMethod":"{\"scope\":\"device\",\"type\":\"sas\",\"issuer\":\"iothub\",\"acceptingIpFilterRule\":null}",
    "connectionDeviceGenerationId":"637051167384630591",
    "contentType":"application/json",
    "contentEncoding":"utf-8",
    "enqueuedTime":"2019-09-26T17:46:09.8870000Z"
  },
  "Body":{
    "temp":49.91322758395974,
    "humid":49.61214852573155,
    "pm25":25.87332214661367
  }
}
```

## <a name="devices"></a>Dispositivos

Cada mensaje o registro de una instantánea representa uno o más cambios que se han producido en un dispositivo, y en las propiedades del dispositivo y la nube desde la exportación del último mensaje. El mensaje incluye:

- `id` del dispositivo en IoT Central
- `displayName` del dispositivo
- Id. de la plantilla de dispositivo en `instanceOf`
- Marca `simulated`, se cumple si se trata de un dispositivo simulado
- Marca `provisioned`, se cumple si el dispositivo se ha aprovisionado
- Marca `approved`, se cumple si el dispositivo tiene aprobado el envío de datos
- Valores de propiedad
- `properties`, incluidos los valores de propiedades de la nube y del dispositivo

Los dispositivos eliminados no se exportan. Actualmente, no hay ningún indicador en los mensajes exportados para los dispositivos eliminados.

En Event Hubs y Service Bus, IoT Central envía los mensajes que contienen datos del dispositivo al centro de eventos o a la cola o tema de Service Bus casi en tiempo real.

Para Blob Storage, una nueva instantánea que contiene todos los cambios desde la última vez escrita se exporta una vez por minuto.

En el siguiente mensaje de ejemplo se muestra información sobre los datos de dispositivos y propiedades en un centro de eventos o una cola o tema de Service Bus:

```json
{
  "body":{
    "id": "<device Id>",
    "etag": "<etag>",
    "displayName": "Sensor 1",
    "instanceOf": "<device template Id>",
    "simulated": false,
    "provisioned": true,
    "approved": true,
    "properties": {
        "sensorComponent": {
            "setTemp": "30",
            "fwVersion": "2.0.1",
            "status": { "first": "first", "second": "second" },
            "$metadata": {
                "setTemp": {
                    "desiredValue": "30",
                    "desiredVersion": 3,
                    "desiredTimestamp": "2020-02-01T17:15:08.9284049Z",
                    "ackVersion": 3
                },
                "fwVersion": { "ackVersion": 3 },
                "status": {
                    "desiredValue": {
                        "first": "first",
                        "second": "second"
                    },
                    "desiredVersion": 2,
                    "desiredTimestamp": "2020-02-01T17:15:08.9284049Z",
                    "ackVersion": 2
                }
            },
            
        }
    },
    "installDate": { "installDate": "2020-02-01" }
},
  "annotations":{
    "iotcentral-message-source":"devices",
    "x-opt-partition-key":"<partitionKey>",
    "x-opt-sequence-number":39740,
    "x-opt-offset":"<offset>",
    "x-opt-enqueued-time":1539274959654
  },
  "partitionKey":"<partitionKey>",
  "sequenceNumber":39740,
  "enqueuedTimeUtc":"2020-02-01T18:14:49.3820326Z",
  "offset":"<offset>"
}
```

Esta instantánea es un mensaje de ejemplo que muestra los datos de los dispositivos y las propiedades en Blob Storage. Los archivos exportados contienen una sola línea por registro.

```json
{
  "id": "<device Id>",
  "etag": "<etag>",
  "displayName": "Sensor 1",
  "instanceOf": "<device template Id>",
  "simulated": false,
  "provisioned": true,
  "approved": true,
  "properties": {
      "sensorComponent": {
          "setTemp": "30",
          "fwVersion": "2.0.1",
          "status": { "first": "first", "second": "second" },
          "$metadata": {
              "setTemp": {
                  "desiredValue": "30",
                  "desiredVersion": 3,
                  "desiredTimestamp": "2020-02-01T17:15:08.9284049Z",
                  "ackVersion": 3
              },
              "fwVersion": { "ackVersion": 3 },
              "status": {
                  "desiredValue": {
                      "first": "first",
                      "second": "second"
                  },
                  "desiredVersion": 2,
                  "desiredTimestamp": "2020-02-01T17:15:08.9284049Z",
                  "ackVersion": 2
              }
          },
          
      }
  },
  "installDate": { "installDate": "2020-02-01" }
}
```

## <a name="device-templates"></a>Plantillas de dispositivo

Cada mensaje o registro de instantáneas representa uno o más cambios que se han producido en una plantilla de dispositivo publicada desde la exportación del último mensaje. Entre la información enviada en cada mensaje o registro se incluye lo siguiente:

- `id` de la plantilla de dispositivo que coincide con el objeto `instanceOf` del flujo de dispositivos anterior
- `displayName` de la plantilla de dispositivo
- El dispositivo `capabilityModel`, incluido su `interfaces`, y los datos de telemetría, las propiedades y las definiciones de comandos
- Definiciones de `cloudProperties`
- Reemplazos y valores iniciales, insertados con el `capabilityModel`

Las plantillas de dispositivo eliminadas no se exportan. Actualmente, no hay ningún indicador en los mensajes exportados para las plantillas de dispositivo eliminadas.

En Event Hubs y Service Bus, IoT Central envía los mensajes que contienen datos de plantilla de dispositivo al centro de eventos o a la cola o tema de Service Bus casi en tiempo real.

Para Blob Storage, una nueva instantánea que contiene todos los cambios desde la última que se escribió se exporta una vez por minuto.

En el ejemplo siguiente se muestra un mensaje sobre los datos de las plantillas de dispositivo en el centro de eventos o en la cola o tema de Service Bus:

```json
{
  "body":{
      "id": "<device template id>",
      "etag": "<etag>",
      "types": ["DeviceModel"],
      "displayName": "Sensor template",
      "capabilityModel": {
          "@id": "<capability model id>",
          "@type": ["CapabilityModel"],
          "contents": [],
          "implements": [
              {
                  "@id": "<component Id>",
                  "@type": ["InterfaceInstance"],
                  "name": "sensorComponent",
                  "schema": {
                      "@id": "<interface Id>",
                      "@type": ["Interface"],
                      "displayName": "Sensor interface",
                      "contents": [
                          {
                              "@id": "<id>",
                              "@type": ["Telemetry"],
                              "displayName": "Humidity",
                              "name": "humidity",
                              "schema": "double"
                          },
                          {
                              "@id": "<id>",
                              "@type": ["Telemetry", "SemanticType/Event"],
                              "displayName": "Error event",
                              "name": "error",
                              "schema": "integer"
                          },
                          {
                              "@id": "<id>",
                              "@type": ["Property"],
                              "displayName": "Set temperature",
                              "name": "setTemp",
                              "writable": true,
                              "schema": "integer",
                              "unit": "Units/Temperature/fahrenheit",
                              "initialValue": "30"
                          },
                          {
                              "@id": "<id>",
                              "@type": ["Property"],
                              "displayName": "Firmware version read only",
                              "name": "fwversion",
                              "schema": "string"
                          },
                          {
                              "@id": "<id>",
                              "@type": ["Property"],
                              "displayName": "Display status",
                              "name": "status",
                              "writable": true,
                              "schema": {
                                  "@id": "urn:testInterface:status:obj:ka8iw8wka:1",
                                  "@type": ["Object"]
                              }
                          },
                          {
                              "@id": "<id>",
                              "@type": ["Command"],
                              "commandType": "synchronous",
                              "request": {
                                  "@id": "<id>",
                                  "@type": ["SchemaField"],
                                  "displayName": "Configuration",
                                  "name": "config",
                                  "schema": "string"
                              },
                              "response": {
                                  "@id": "<id>",
                                  "@type": ["SchemaField"],
                                  "displayName": "Response",
                                  "name": "response",
                                  "schema": "string"
                              },
                              "displayName": "Configure sensor",
                              "name": "sensorConfig"
                          }
                      ]
                  }
              }
          ],
          "displayName": "Sensor capability model"
      },
      "solutionModel": {
          "@id": "<id>",
          "@type": ["SolutionModel"],
          "cloudProperties": [
              {
                  "@id": "<id>",
                  "@type": ["CloudProperty"],
                  "displayName": "Install date",
                  "name": "installDate",
                  "schema": "dateTime",
                  "valueDetail": {
                      "@id": "<id>",
                      "@type": ["ValueDetail/DateTimeValueDetail"]
                  }
              }
          ]
      }
  },
    "annotations":{
      "iotcentral-message-source":"deviceTemplates",
      "x-opt-partition-key":"<partitionKey>",
      "x-opt-sequence-number":25315,
      "x-opt-offset":"<offset>",
      "x-opt-enqueued-time":1539274985085
    },
    "partitionKey":"<partitionKey>",
    "sequenceNumber":25315,
    "enqueuedTimeUtc":"2019-10-02T16:23:05.085Z",
    "offset":"<offset>"
  }
}
```

En esta instantánea de ejemplo se muestra un mensaje que contiene los datos de dispositivos y propiedades en Blob Storage. Los archivos exportados contienen una sola línea por registro.

```json
{
      "id": "<device template id>",
      "etag": "<etag>",
      "types": ["DeviceModel"],
      "displayName": "Sensor template",
      "capabilityModel": {
          "@id": "<capability model id>",
          "@type": ["CapabilityModel"],
          "contents": [],
          "implements": [
              {
                  "@id": "<component Id>",
                  "@type": ["InterfaceInstance"],
                  "name": "Sensor component",
                  "schema": {
                      "@id": "<interface Id>",
                      "@type": ["Interface"],
                      "displayName": "Sensor interface",
                      "contents": [
                          {
                              "@id": "<id>",
                              "@type": ["Telemetry"],
                              "displayName": "Humidity",
                              "name": "humidity",
                              "schema": "double"
                          },
                          {
                              "@id": "<id>",
                              "@type": ["Telemetry", "SemanticType/Event"],
                              "displayName": "Error event",
                              "name": "error",
                              "schema": "integer"
                          },
                          {
                              "@id": "<id>",
                              "@type": ["Property"],
                              "displayName": "Set temperature",
                              "name": "setTemp",
                              "writable": true,
                              "schema": "integer",
                              "unit": "Units/Temperature/fahrenheit",
                              "initialValue": "30"
                          },
                          {
                              "@id": "<id>",
                              "@type": ["Property"],
                              "displayName": "Firmware version read only",
                              "name": "fwversion",
                              "schema": "string"
                          },
                          {
                              "@id": "<id>",
                              "@type": ["Property"],
                              "displayName": "Display status",
                              "name": "status",
                              "writable": true,
                              "schema": {
                                  "@id": "urn:testInterface:status:obj:ka8iw8wka:1",
                                  "@type": ["Object"]
                              }
                          },
                          {
                              "@id": "<id>",
                              "@type": ["Command"],
                              "commandType": "synchronous",
                              "request": {
                                  "@id": "<id>",
                                  "@type": ["SchemaField"],
                                  "displayName": "Configuration",
                                  "name": "config",
                                  "schema": "string"
                              },
                              "response": {
                                  "@id": "<id>",
                                  "@type": ["SchemaField"],
                                  "displayName": "Response",
                                  "name": "response",
                                  "schema": "string"
                              },
                              "displayName": "Configure sensor",
                              "name": "sensorconfig"
                          }
                      ]
                  }
              }
          ],
          "displayName": "Sensor capability model"
      },
      "solutionModel": {
          "@id": "<id>",
          "@type": ["SolutionModel"],
          "cloudProperties": [
              {
                  "@id": "<id>",
                  "@type": ["CloudProperty"],
                  "displayName": "Install date",
                  "name": "installDate",
                  "schema": "dateTime",
                  "valueDetail": {
                      "@id": "<id>",
                      "@type": ["ValueDetail/DateTimeValueDetail"]
                  }
              }
          ]
      }
  }
```

## <a name="data-format-change-notice"></a>Aviso de cambio de formato de datos

> [!Note]
> El formato de los datos del flujo de telemetría no se ve afectado por este cambio. Solo se ven afectados los flujos de datos de dispositivos y de plantillas de dispositivos.

Si tiene una exportación de datos existente en la aplicación de versión preliminar con los flujos *Dispositivos* y *Plantillas de dispositivo* activados, actualice la exportación antes del **30 de junio de 2020**. Este requisito se aplica a las exportaciones a Azure Blob Storage, Azure Event Hubs y Azure Service Bus.

A partir del 3 de febrero de 2020, todas las nuevas exportaciones en aplicaciones con Dispositivos y Plantillas de dispositivo habilitados tendrán el formato de datos descrito anteriormente. Todas las exportaciones creadas antes de esta fecha permanecen en el formato de datos antiguo hasta el 30 de junio de 2020, en esa fecha estas exportaciones se migrarán automáticamente al nuevo formato de datos. El nuevo formato de datos coincide con los objetos [dispositivo](https://docs.microsoft.com/rest/api/iotcentral/devices/get), [propiedad del dispositivo](https://docs.microsoft.com/rest/api/iotcentral/devices/getproperties), [propiedad de la nube del dispositivo](https://docs.microsoft.com/rest/api/iotcentral/devices/getcloudproperties) y [plantilla de dispositivo](https://docs.microsoft.com/rest/api/iotcentral/devicetemplates/get) en la API pública de IoT Central.

En el caso de **Dispositivos**, las diferencias importantes entre el formato de datos anterior y el nuevo formato de datos incluyen:
- Se quitó `@id` para el dispositivo, se cambió el nombre de `deviceId` a `id` 
- Se agregó la marca `provisioned` para describir el estado de aprovisionamiento del dispositivo
- Se agregó la marca `approved` para describir el estado de aprobación del dispositivo
- `properties`, incluidas las propiedades de dispositivo y de nube, coincide con las entidades de la API pública

En el caso de **Plantillas de dispositivo**, las diferencias importantes entre el formato de datos anterior y el nuevo formato de datos incluyen:

- `@id` para la plantilla de dispositivo cambió de nombre a `id`
- `@type` para la plantilla de dispositivo cambió de nombre a `types`y ahora es una matriz

### <a name="devices-format-deprecated-as-of-3-february-2020"></a>Dispositivos (formato en desuso a partir del 3 de febrero de 2020)

```json
{
  "@id":"<id-value>",
  "@type":"Device",
  "displayName":"Airbox",
  "data":{
    "$cloudProperties":{
        "Color":"blue"
    },
    "EnvironmentalSensor":{
      "thsensormodel":{
        "reported":{
          "value":"Neque quia et voluptatem veritatis assumenda consequuntur quod.",
          "$lastUpdatedTimestamp":"2019-09-30T20:35:43.8478978Z"
        }
      },
      "pm25sensormodel":{
        "reported":{
          "value":"Aut alias odio.",
          "$lastUpdatedTimestamp":"2019-09-30T20:35:43.8478978Z"
        }
      }
    },
    "urn_azureiot_DeviceManagement_DeviceInformation":{
      "totalStorage":{
        "reported":{
          "value":27900.9730905171,
          "$lastUpdatedTimestamp":"2019-09-30T20:35:43.8478978Z"
        }
      },
      "totalMemory":{
        "reported":{
          "value":4667.82916715811,
          "$lastUpdatedTimestamp":"2019-09-30T20:35:43.8478978Z"
        }
      }
    }
  },
  "instanceOf":"<template-id>",
  "deviceId":"<device-id>",
  "simulated":true
}
```

### <a name="device-templates-format-deprecated-as-of-3-february-2020"></a>Plantillas de dispositivo (formato en desuso a partir del 3 de febrero de 2020)

```json
{
  "@id":"<template-id>",
  "@type":"DeviceModelDefinition",
  "displayName":"Airbox",
  "capabilityModel":{
    "@id":"<id>",
    "@type":"CapabilityModel",
    "implements":[
      {
        "@id":"<id>",
        "@type":"InterfaceInstance",
        "name":"EnvironmentalSensor",
        "schema":{
          "@id":"<id>",
          "@type":"Interface",
          "comment":"Requires temperature and humidity sensors.",
          "description":"Provides functionality to report temperature, humidity. Provides telemetry, commands and read-write properties",
          "displayName":"Environmental Sensor",
          "contents":[
            {
              "@id":"<id>",
              "@type":"Telemetry",
              "description":"Current temperature on the device",
              "displayName":"Temperature",
              "name":"temp",
              "schema":"double",
              "unit":"Units/Temperature/celsius",
              "valueDetail":{
                "@id":"<id>",
                "@type":"ValueDetail/NumberValueDetail",
                "minValue":{
                  "@value":"50"
                }
              },
              "visualizationDetail":{
                "@id":"<id>",
                "@type":"VisualizationDetail"
              }
            },
            {
              "@id":"<id>",
              "@type":"Telemetry",
              "description":"Current humidity on the device",
              "displayName":"Humidity",
              "name":"humid",
              "schema":"integer"
            },
            {
              "@id":"<id>",
              "@type":"Telemetry",
              "description":"Current PM2.5 on the device",
              "displayName":"PM2.5",
              "name":"pm25",
              "schema":"integer"
            },
            {
              "@id":"<id>",
              "@type":"Property",
              "description":"T&H Sensor Model Name",
              "displayName":"T&H Sensor Model",
              "name":"thsensormodel",
              "schema":"string"
            },
            {
              "@id":"<id>",
              "@type":"Property",
              "description":"PM2.5 Sensor Model Name",
              "displayName":"PM2.5 Sensor Model",
              "name":"pm25sensormodel",
              "schema":"string"
            }
          ]
        }
      },
      {
        "@id":"<id>",
        "@type":"InterfaceInstance",
        "name":"urn_azureiot_DeviceManagement_DeviceInformation",
        "schema":{
          "@id":"<id>",
          "@type":"Interface",
          "displayName":"Device information",
          "contents":[
            {
              "@id":"<id>",
              "@type":"Property",
              "comment":"Total available storage on the device in kilobytes. Ex. 20480000 kilobytes.",
              "displayName":"Total storage",
              "name":"totalStorage",
              "displayUnit":"kilobytes",
              "schema":"long"
            },
            {
              "@id":"<id>",
              "@type":"Property",
              "comment":"Total available memory on the device in kilobytes. Ex. 256000 kilobytes.",
              "displayName":"Total memory",
              "name":"totalMemory",
              "displayUnit":"kilobytes",
              "schema":"long"
            }
          ]
        }
      }
    ],
    "displayName":"AAEONAirbox52"
  },
  "solutionModel":{
    "@id":"<id>",
    "@type":"SolutionModel",
    "cloudProperties":[
      {
        "@id":"<id>",
        "@type":"CloudProperty",
        "displayName":"Color",
        "name":"Color",
        "schema":"string",
        "valueDetail":{
          "@id":"<id>",
          "@type":"ValueDetail/StringValueDetail"
        },
        "visualizationDetail":{
          "@id":"<id>",
          "@type":"VisualizationDetail"
        }
      }
    ]
  }
}
```

## <a name="next-steps"></a>Pasos siguientes

Ahora que sabe cómo exportar los datos a Azure Event Hubs, Azure Service Bus y Azure Blob Storage, vaya al paso siguiente:

> [!div class="nextstepaction"]
> [Cómo ejecutar un análisis personalizado con Databricks](./howto-create-custom-analytics.md)
