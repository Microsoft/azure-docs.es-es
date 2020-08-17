---
title: Consulta sobre el enrutamiento de mensajes de Azure IoT Hub | Microsoft Docs
description: Obtenga información sobre el lenguaje de consulta de enrutamiento de mensajes de IoT Hub que puede usar para aplicar consultas enriquecidas a mensajes para recibir los datos que le interesan.
author: ash2017
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 08/13/2018
ms.author: asrastog
ms.custom:
- 'Role: Cloud Development'
- 'Role: Data Analytics'
ms.openlocfilehash: 4ff61687f2fa045b51dfcb69488d1fbd87b65f75
ms.sourcegitcommit: 46f8457ccb224eb000799ec81ed5b3ea93a6f06f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87336504"
---
# <a name="iot-hub-message-routing-query-syntax"></a>Sintaxis de las consultas de enrutamiento de mensajes de IoT Hub

El enrutamiento de mensajes permite a los usuarios enrutar diferentes tipos de datos, es decir, mensajes de telemetría de dispositivos, eventos de ciclo de vida de dispositivos y eventos de cambio de dispositivos gemelos a varios puntos de conexión. También puede aplicar consultas enriquecidas a estos datos antes de enrutarlos para recibir los datos que le interesan. En este artículo se describe el lenguaje de consulta de enrutamiento de mensajes de IoT Hub, y se proporcionan algunos patrones de consulta comunes.

[!INCLUDE [iot-hub-basic](../../includes/iot-hub-basic-partial.md)]

El enrutamiento de mensajes le permite realizar consultas sobre las propiedades de los mensajes y el cuerpo del mensaje, así como las etiquetas del dispositivo gemelo y las propiedades del dispositivo gemelo. Si el cuerpo del mensaje no está en formato JSON, el enrutamiento de mensajes aún puede enrutar el mensaje, pero las consultas no se pueden aplicar al cuerpo del mensaje.  Las consultas se describen como expresiones booleanas en las que un valor booleano true hace que la consulta se ejecute correctamente, lo que enruta todos los datos de entrada, y un valor booleano false produce un error en la consulta y no se enruta ningún dato. Si la expresión se evalúa como nula o indefinida, se trata como falsa y se generará un error en los registros de diagnóstico en caso de error. La sintaxis de consulta debe ser correcta para que la ruta se guarde y evalúe.  

## <a name="message-routing-query-based-on-message-properties"></a>Consulta de enrutamiento de mensajes basada en las propiedades del mensaje 

IoT Hub define un [formato común](iot-hub-devguide-messages-construct.md) para todos los mensajes del dispositivo a la nube a fin de generar interoperabilidad entre protocolos. El mensaje de IoT Hub da por supuesto la siguiente representación JSON del mensaje. Las propiedades del sistema se agregan para todos los usuarios e identifican el contenido del mensaje. Los usuarios pueden agregar propiedades de aplicación al mensaje de forma selectiva. Se recomienda utilizar nombres de propiedades únicos, ya que la mensajería del dispositivo a la nube de IoT Hub no distingue mayúsculas de minúsculas. Por ejemplo, si tiene varias propiedades con el mismo nombre, IoT Hub solo enviará una de las propiedades.  

```json
{ 
  "message": { 
    "systemProperties": { 
      "contentType": "application/json", 
      "contentEncoding": "UTF-8", 
      "iothub-message-source": "deviceMessages", 
      "iothub-enqueuedtime": "2017-05-08T18:55:31.8514657Z" 
    }, 
    "appProperties": { 
      "processingPath": "{cold | warm | hot}", 
      "verbose": "{true, false}", 
      "severity": 1-5, 
      "testDevice": "{true | false}" 
    }, 
    "body": "{\"Weather\":{\"Temperature\":50}}" 
  } 
} 
```

### <a name="system-properties"></a>Propiedades del sistema

Las propiedades del sistema ayudan a identificar el contenido y el origen de los mensajes. 

| Propiedad | Tipo | Descripción |
| -------- | ---- | ----------- |
| contentType | string | El usuario especifica el tipo de contenido del mensaje. Para permitir la consulta en el cuerpo del mensaje, este valor debe establecerse en application/JSON. |
| contentEncoding | string | El usuario especifica el tipo de codificación del mensaje. Los valores permitidos son UTF-8, UTF-16, UTF-32, si contentType se establece en application/JSON. |
| iothub-connection-device-id | string | Este valor se establece mediante IoT Hub e identifica el identificador del dispositivo. Para realizar la consulta, use `$connectionDeviceId`. |
| iothub-enqueuedtime | string | Este valor lo establece IoT Hub y representa la hora real en UTC de espera en cola del mensaje. Para realizar la consulta, use `enqueuedTime`. |
| dt-dataschema | string |  El centro de IoT establece este valor en mensajes de dispositivo a la nube. Contiene el id. de modelo del dispositivo establecido en la conexión del dispositivo. Esta característica está disponible como parte de la [versión preliminar pública de IoT Plug and Play](../iot-pnp/overview-iot-plug-and-play.md). Para realizar la consulta, use `$dt-dataschema`. |
| dt-subject | string | Nombre del componente que envía los mensajes del dispositivo a la nube. Esta característica está disponible como parte de la [versión preliminar pública de IoT Plug and Play](../iot-pnp/overview-iot-plug-and-play.md). Para realizar la consulta, use `$dt-subject`. |

Como se describe en los [mensajes de IoT Hub](iot-hub-devguide-messages-construct.md), hay propiedades adicionales del sistema en un mensaje. Además de las propiedades anteriores de la tabla anterior, también puede consultar **connectionDeviceId**, **connectionModuleId**.

### <a name="application-properties"></a>Propiedades de la aplicación

Las propiedades de la aplicación son cadenas definidas por el usuario que se pueden agregar al mensaje. Estos campos son opcionales.  

### <a name="query-expressions"></a>Expresiones de consulta

Una consulta sobre las propiedades del sistema de mensajes tienen como prefijo el símbolo `$`. Las consultas a las propiedades de aplicación se realizan con el nombre y no deben prefijarse con el símbolo`$`. Si el nombre de una propiedad de la aplicación comienza por `$`, IoT Hub la buscará en las propiedades del sistema y, si no se encuentra, la buscará en las propiedades de la aplicación. Por ejemplo: 

Para consultar en la propiedad del sistema contentEncoding 

```sql
$contentEncoding = 'UTF-8'
```

Para consultar la aplicación de la propiedad processingPath:

```sql
processingPath = 'hot'
```

Para combinar estas consultas, se pueden usar funciones y expresiones booleanas:

```sql
$contentEncoding = 'UTF-8' AND processingPath = 'hot'
```

Una lista completa de los operadores y funciones admitidas se muestra en [Expresiones y condiciones](iot-hub-devguide-query-language.md#expressions-and-conditions).

## <a name="message-routing-query-based-on-message-body"></a>Consulta de enrutamiento de mensajes basada en el cuerpo del mensaje

Para habilitar la consulta en el cuerpo del mensaje, el mensaje debe estar en JSON codificado en UTF-8, UTF-16 o UTF-32. `contentType` debe establecerse en `application/JSON` y `contentEncoding` a una de las codificaciones UTF admitidas de la propiedad del sistema. Si no se especifican estas propiedades, IoT Hub no evaluará la expresión de consulta en el cuerpo del mensaje. 

En el ejemplo siguiente se muestra cómo crear un mensaje con un cuerpo JSON codificado y formado correctamente: 

```javascript
var messageBody = JSON.stringify(Object.assign({}, {
    "Weather": {
        "Temperature": 50,
        "Time": "2017-03-09T00:00:00.000Z",
        "PrevTemperatures": [
            20,
            30,
            40
        ],
        "IsEnabled": true,
        "Location": {
            "Street": "One Microsoft Way",
            "City": "Redmond",
            "State": "WA"
        },
        "HistoricalData": [
            {
                "Month": "Feb",
                "Temperature": 40
            },
            {
                "Month": "Jan",
                "Temperature": 30
            }
        ]
    }
}));

// Encode message body using UTF-8  
var messageBytes = Buffer.from(messageBody, "utf8");

var message = new Message(messageBytes);

// Set message body type and content encoding 
message.contentEncoding = "utf-8";
message.contentType = "application/json";

// Add other custom application properties   
message.properties.add("Status", "Active");

deviceClient.sendEvent(message, (err, res) => {
    if (err) console.log('error: ' + err.toString());
    if (res) console.log('status: ' + res.constructor.name);
});
```

> [!NOTE] 
> Se muestra cómo administrar la codificación del cuerpo en JavaScript. Si quiere ver un ejemplo en C#, descargue los [ejemplos de C# de Azure IoT](https://github.com/Azure-Samples/azure-iot-samples-csharp/archive/master.zip). Descomprima el archivo master.zip. El archivo Program.cs de la solución de Visual Studio *SimulatedDevice* muestra cómo codificar y enviar mensajes a una instancia de IoT Hub. Este es el mismo ejemplo que se usa para probar el enrutamiento de mensajes, como se explica en el [tutorial de enrutamiento de mensajes](tutorial-routing.md). En la parte inferior de Program.cs, también hay un método para leer en uno de los archivos codificados, descodificarlo y escribirlo de nuevo en ASCII para que pueda leerlo. 


### <a name="query-expressions"></a>Expresiones de consulta

Una consulta al cuerpo del mensaje debe tener el prefijo `$body`. Puede usar una referencia al cuerpo, una referencia a la matriz del cuerpo o varias referencias al cuerpo en la expresión de consulta. La expresión de consulta también puede combinar una referencia al cuerpo y a las referencias a las propiedades de la aplicación de mensajes. Por ejemplo, todas las expresiones siguientes son expresiones de consulta válidas: 

```sql
$body.Weather.HistoricalData[0].Month = 'Feb' 
```

```sql
$body.Weather.Temperature = 50 AND $body.Weather.IsEnabled 
```

```sql
length($body.Weather.Location.State) = 2 
```

```sql
$body.Weather.Temperature = 50 AND processingPath = 'hot'
```

## <a name="message-routing-query-based-on-device-twin"></a>Consulta de enrutamiento de mensajes basada en un dispositivo gemelo 

El enrutamiento de mensajes le permite consultar en etiquetas y propiedades de [dispositivos gemelos](iot-hub-devguide-device-twins.md), que son objetos JSON. También se admite la consulta en el módulo gemelo. A continuación se muestra un ejemplo de propiedades y etiquetas de dispositivo gemelo.

```JSON
{
    "tags": { 
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

### <a name="query-expressions"></a>Expresiones de consulta

Una consulta en el mensaje gemelo debe tener el prefijo `$twin`. La expresión de consulta también puede combinar referencias a propiedades o etiquetas de dispositivo gemelo con referencias al cuerpo del mensaje, a las propiedades del sistema de mensajes y a las propiedades de la aplicación de mensajes. Se recomienda usar nombres únicos en las etiquetas y propiedades ya que la consulta no distingue mayúsculas de minúsculas. Esto se aplica tanto a los dispositivos gemelos como a los módulo gemelos. También debe evitar utilizar `twin`, `$twin`, `body` o `$body` como nombres de propiedad. Por ejemplo, todas las expresiones siguientes son expresiones de consulta válidas: 

```sql
$twin.properties.desired.telemetryConfig.sendFrequency = '5m'
```

```sql
$body.Weather.Temperature = 50 AND $twin.properties.desired.telemetryConfig.sendFrequency = '5m'
```

```sql
$twin.tags.deploymentLocation.floor = 1 
```

No se admite la consulta de enrutamiento en el cuerpo o el dispositivo gemelo con un punto en el nombre de la propiedad o la carga.

## <a name="next-steps"></a>Pasos siguientes

* Más información sobre el [enrutamiento de mensajes](iot-hub-devguide-messages-d2c.md).
* Pruebe el [tutorial de enrutamiento de mensajes](tutorial-routing.md).
