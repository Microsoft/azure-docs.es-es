---
title: Integración de partners de sensores
description: En este artículo se describe la integración de asociados de sensores.
author: uhabiba04
ms.topic: article
ms.date: 11/04/2019
ms.author: v-umha
ms.openlocfilehash: 88a8dcb53ab2f845f52121b11c96c23ad0a3e791
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87078928"
---
# <a name="sensor-partner-integration"></a>Integración de partners de sensores

En este artículo se proporciona información sobre el componente **Translator** de FarmBeats de Azure, que permite la integración de asociados de sensor.

Con este componente, los asociados se pueden integrar con FarmBeats mediante las API de FarmBeats Datahub y enviar telemetría y datos de dispositivos de clientes a FarmBeats Datahub. Una vez que los datos están disponibles en FarmBeats, se visualizan con el acelerador de FarmBeats y se pueden usar para la fusión de datos y para crear modelos de inteligencia artificial y aprendizaje automático.

## <a name="before-you-start"></a>Antes de comenzar

Para desarrollar el componente Translator, necesitará las credenciales siguientes que permitirán el acceso a las API de FarmBeats.

- Punto de conexión de API
- Id. de inquilino
- Id. de cliente
- Secreto del cliente
- Cadena de conexión de EventHub

Vea esta sección para obtener las credenciales anteriores: [Habilitación de la integración de dispositivos](get-sensor-data-from-sensor-partner.md#enable-device-integration-with-farmbeats)

## <a name="translator-development"></a>Desarrollo del componente Traductor

**Integración basada en la API REST**

Las funcionalidades de integración de datos de sensores de FarmBeats se exponen a través de la API REST. Algunas de las funcionalidades son definición de metadatos, aprovisionamiento de dispositivos y sensores, y administración de dispositivos y sensores.

**Ingesta de telemetría**

Los datos de telemetría se asignan a un mensaje canónico publicado en Azure Event Hubs para su procesamiento. Azure Event Hubs es un servicio que permite la ingesta de datos en tiempo real (telemetría) de aplicaciones y dispositivos conectados.

**Desarrollo de API**

Las API contienen documentación técnica de Swagger. Para obtener más información sobre las API y sus solicitudes y respuestas correspondientes, consulte [Swagger](https://aka.ms/FarmBeatsSwagger).

**Autenticación**

FarmBeats usa autenticación de Microsoft Azure Active Directory. Azure App Service proporciona compatibilidad integrada con la autenticación y la autorización.

Para obtener más información, consulte [Azure Active Directory](https://docs.microsoft.com/azure/app-service/overview-authentication-authorization).

El centro de datos de FarmBeats usa la autenticación de portador, que necesita las credenciales siguientes:
   - Id. de cliente
   - Secreto del cliente
   - Id. de inquilino

Con estas credenciales, el autor de la llamada puede solicitar un token de acceso. El token se debe enviar en las solicitudes de API posteriores en la sección de encabezado, de la siguiente manera:

```
headers = {"Authorization": "Bearer " + access_token, …} 
```

El ejemplo de código de Python siguiente proporciona el token de acceso, que se puede usar para las posteriores llamadas API a FarmBeats.

```python
import requests
import json
import msal

# Your service principal App ID
CLIENT_ID = "<CLIENT_ID>"
# Your service principal password
CLIENT_SECRET = "<CLIENT_SECRET>"
# Tenant ID for your Azure subscription
TENANT_ID = "<TENANT_ID>"

AUTHORITY_HOST = 'https://login.microsoftonline.com'
AUTHORITY = AUTHORITY_HOST + '/' + TENANT_ID

ENDPOINT = "https://<yourfarmbeatswebsitename-api>.azurewebsites.net"
SCOPE = ENDPOINT + "/.default"

context = msal.ConfidentialClientApplication(CLIENT_ID, authority=AUTHORITY, client_credential=CLIENT_SECRET)
token_response = context.acquire_token_for_client(SCOPE)
# We should get an access token here
access_token = token_response.get('access_token')
```


**Encabezados de solicitud HTTP**

Estos son los encabezados de solicitud más comunes que deben especificarse al realizar una llamada API al centro de datos de FarmBeats.


**Encabezado** | **Descripción y ejemplo**
--- | ---
Content-Type | El formato de la solicitud (Content-Type: application/<format>). En el caso de las API de centro de datos de FarmBeats, el formato es JSON. Content-Type: application/json
Authorization | Especifica el token de acceso necesario para realizar una llamada API. Autorización: Bearer <Access-Token>
Accept | El formato de respuesta. En el caso de las API de centro de datos de FarmBeats, el formato es JSON. Accept: application/json

**Solicitudes de API**

Para realizar una solicitud de API REST, combine el método HTTP (GET, POST o PUT), la dirección URL al servicio de API y el URI (identificador uniforme de recursos) de un recurso para consultar, actualizar, eliminar o enviarle datos, y uno o varios encabezados de solicitud HTTP. La dirección URL del servicio de API es el punto de conexión de API que proporciona. Este es un ejemplo: https://\<yourdatahub-website-name>.azurewebsites.net

También se pueden incluir parámetros de consulta en las llamadas GET para filtrar y ordenar los datos en las respuestas y limitar su tamaño.

La siguiente solicitud de ejemplo se usa para obtener la lista de dispositivos.

```bash
curl -X GET "https://microsoft-farmbeats.azurewebsites.net/Device" -H "Content-Type: application/json" -H "Authorization: Bearer <Access-Token>"
```
La mayoría de llamadas GET, POST y PUT requieren un cuerpo de la solicitud JSON.

La siguiente solicitud de ejemplo se usa para crear un dispositivo. (Este ejemplo tiene un código JSON de entrada con el cuerpo de la solicitud).

```bash
curl -X POST "https://microsoft-farmbeats.azurewebsites.net/Device" -H  "accept: application/json" -H  "Content-Type: application/json" -H "Authorization: Bearer <Access-Token>" -d "{  \"deviceModelId\": \"ID123\",  \"hardwareId\": \"MHDN123\",  \"reportingInterval\": 900,  \"name\": \"Device123\",  \"description\": \"Test Device 123\",}"
```

## <a name="data-format"></a>Formato de datos

JSON es un formato de datos común independiente del lenguaje que proporciona una representación de texto simple de estructuras de datos arbitrarias. Para obtener más información, visite [json.org](http://json.org).

## <a name="metadata-specifications"></a>Especificaciones de metadatos

El centro de datos de FarmBeats tiene las siguientes API que permiten a los asociados de dispositivo crear y administrar los metadatos de dispositivos o sensores.

- /**DeviceModel**: DeviceModel se corresponde con los metadatos del dispositivo, como el fabricante o el tipo de dispositivo, que es puerta de enlace o nodo.
- /**Device**: Device se corresponde con un dispositivo físico presente en la granja.
- /**SensorModel**: SensorModel se corresponde con los metadatos del sensor, por ejemplo, el fabricante, el tipo de sensor (analógico o digital) o la medida de sensor (como temperatura ambiente y presión).
- /**Sensor**: El sensor corresponde a un sensor físico que registra valores. Un sensor normalmente se conecta a un dispositivo con un identificador de dispositivo.

  DeviceModel | Descripción |
  --- | ---
  Type (nodo, puerta de enlace)  | Tipo del dispositivo: nodo o puerta de enlace |
  Fabricante  | Nombre del fabricante |
  ProductCode  | Código de producto del dispositivo o nombre o número de modelo. Por ejemplo, EnviroMonitor#6800. |
  Puertos  | Nombre y tipo de puerto, que es digital o analógico.  |
  Nombre  | Nombre para identificar el recurso. Por ejemplo, nombre del modelo o nombre del producto. |
  Descripción  | Proporciona una descripción significativa del modelo. |
  Propiedades  | Propiedades adicionales del fabricante. |
  **Dispositivo** | **Descripción** |
  DeviceModelId  |Identificador del modelo de dispositivo asociado. |
  HardwareId   |Identificador único del dispositivo, como la dirección MAC.  |
  ReportingInterval |Intervalo de informes en segundos. |
  Location    |Latitud (de -90 a +90), longitud (de -180 a 180) y elevación (en metros) del dispositivo. |
  ParentDeviceId | Identificador del dispositivo primario al que está conectado este dispositivo. Por ejemplo, si un nodo está conectado a una puerta de enlace, el nodo tiene parentDeviceID como puerta de enlace. |
  Nombre  | Nombre para identificar el recurso. Los asociados de dispositivo deberán enviar un nombre que se corresponda con el del dispositivo en el lado del asociado de dispositivo. Si el nombre del dispositivo está definido por el usuario en el lado del partner de dispositivo, ese mismo nombre definido por el usuario se debe propagar a FarmBeats.  |
  Descripción  | Proporciona una descripción significativa.  |
  Propiedades  |Propiedades adicionales del fabricante.  |
  **SensorModel** | **Descripción** |
  Type (analog, digital)  |Indica un tipo analógico o digital.|
  Fabricante  | Nombre del fabricante. |
  ProductCode  | Código de producto o nombre o número de modelo. Por ejemplo, RS-CO2-N01.  |
  SensorMeasures > Name  | Nombre de la medida del sensor. Solo se admiten minúsculas. Para medidas desde diferentes profundidades, especifique la profundidad. Por ejemplo, soil_moisture_15cm. Este nombre debe ser coherente con los datos de telemetría. |
  SensorMeasures > DataType  | Tipo de datos de telemetría. Actualmente se admite doble. |
  SensorMeasures > Type  | Tipo de medida de los datos de telemetría del sensor. A continuación se indican los tipos definidos por el sistema: AmbientTemperature, CO2, Depth, ElectricalConductivity, LeafWetness, Length, LiquidLevel, Nitrate, O2, PH, Phosphate, PointInTime, Potassium, Pressure, RainGauge, RelativeHumidity, Salinity, SoilMoisture, SoilTemperature, SolarRadiation, State, TimeDuration, UVRadiation, UVIndex, Volume, WindDirection, WindRun, WindSpeed, Evapotranspiration y PAR. Para agregar más, consulte /ExtendedType API.
  SensorMeasures > Unit | Unidad de datos de telemetría del sensor. A continuación se indican las unidades definidas por el sistema: NoUnit, Celsius, Fahrenheit, Kelvin, Rankine, Pascal, Mercury, PSI, MilliMeter, CentiMeter, Meter, Inch, Feet, Mile, KiloMeter, MilesPerHour, MilesPerSecond, KMPerHour, KMPerSecond, MetersPerHour, MetersPerSecond, Degree, WattsPerSquareMeter, KiloWattsPerSquareMeter, MilliWattsPerSquareCentiMeter, MilliJoulesPerSquareCentiMeter, VolumetricWaterContent, Percentage, PartsPerMillion, MicroMol, MicroMolesPerLiter, SiemensPerSquareMeterPerMole, MilliSiemensPerCentiMeter, Centibar, DeciSiemensPerMeter, KiloPascal, VolumetricIonContent, Liter, MilliLiter, Seconds, UnixTimestamp, MicroMolPerMeterSquaredPerSecond y InchesPerHour. Para agregar más, consulte /ExtendedType API.
  SensorMeasures > AggregationType  | Puede ser none, average, maximum, minimum o StandardDeviation.
  SensorMeasures > Depth  | La profundidad del sensor en centímetros. Por ejemplo, la medida de la humedad 10 cm bajo el suelo.
  SensorMeasures > Description  | Proporciona una descripción significativa de la medida.
  Nombre  | Nombre para identificar el recurso. Por ejemplo, nombre del modelo o nombre del producto.
  Descripción  | Proporciona una descripción significativa del modelo.
  Propiedades  | Propiedades adicionales del fabricante.
  **Sensor**  | **Descripción** |
  HardwareId  | Identificador único del sensor establecido por el fabricante.
  SensorModelId  | Identificador del modelo de sensor asociado.
  Location  | Latitud (de -90 a +90), longitud (de -180 a 180) y elevación (en metros) del sensor.
  Port > Name  |Nombre y tipo de puerto al que está conectado el sensor en el dispositivo. Debe ser el mismo nombre que el definido en el modelo de dispositivo.
  deviceId  | Identificador del dispositivo al que está conectado el sensor.
  Nombre  | Nombre para identificar el recurso. Por ejemplo, el nombre del sensor o del producto, y el número de modelo o el código del producto.
  Descripción  | Proporciona una descripción significativa.
  Propiedades  | Propiedades adicionales del fabricante.

 Para obtener información sobre cada uno de los objetos y sus propiedades, consulte [Swagger](https://aka.ms/FarmBeatsDatahubSwagger).

 > [!NOTE]
 > Las API devuelven identificadores únicos para cada instancia creada. El componente traductor debe conservar este identificador para la sincronización de los metadatos y la administración de dispositivos.


**Sincronización de metadatos**

El componente Traductor debe enviar actualizaciones de los metadatos. Por ejemplo, los escenarios de actualización son el cambio de nombre del dispositivo o sensor, o bien el cambio de ubicación del dispositivo o sensor.

El componente traductor debe tener la capacidad de agregar nuevos dispositivos o sensores instalados por el usuario tras la vinculación de FarmBeats. Del mismo modo, si el usuario ha actualizado un dispositivo o sensor, debe actualizarse en FarmBeats para el dispositivo o sensor correspondiente. Los escenarios típicos que requieren la actualización de un dispositivo o un sensor son un cambio de la ubicación del dispositivo o la adición de sensores en un nodo.


> [!NOTE]
> No se pueden eliminar metadatos de dispositivo o sensor.
>
> Para actualizar los metadatos, es obligatorio llamar a /Get/{id} en el dispositivo o sensor, actualizar las propiedades cambiadas y, a continuación, realizar una llamada /Put/{id} para que no se pierdan las propiedades establecidas por el usuario.

### <a name="add-new-types-and-units"></a>Adición de nuevos tipos y unidades

FarmBeats permite agregar nuevas unidades y tipos de medidas de sensor. Para obtener más información sobre la API /ExtendedType, consulte [Swagger](https://aka.ms/FarmBeatsSwagger).

## <a name="telemetry-specifications"></a>Especificaciones de telemetría

Los datos de telemetría se asignan a un mensaje canónico publicado en Azure Event Hubs para su procesamiento. Azure Event Hubs es un servicio que permite la ingesta de datos en tiempo real (telemetría) de aplicaciones y dispositivos conectados.

## <a name="send-telemetry-data-to-farmbeats"></a>Envío de datos de telemetría a FarmBeats

Para enviar datos de telemetría a FarmBeats, cree un cliente que envíe mensajes a un centro de eventos en FarmBeats. Para más información sobre los datos de telemetría, consulte [Envío de telemetría a un centro de eventos](https://docs.microsoft.com/azure/event-hubs/event-hubs-dotnet-standard-getstarted-send).

Este es un ejemplo de código de Python que envía telemetría como un cliente a un centro de eventos especificado.

```python
import azure
from azure.eventhub import EventHubClient, Sender, EventData, Receiver, Offset
EVENTHUBCONNECTIONSTRING = "<EventHub Connection String provided by customer>"
EVENTHUBNAME = "<EventHub Name provided by customer>"

write_client = EventHubClient.from_connection_string(EVENTHUBCONNECTIONSTRING, eventhub=EVENTHUBNAME, debug=False)
sender = write_client.add_sender(partition="0")
write_client.run()
for i in range(5):
    telemetry = "<Canonical Telemetry message>"
    print("Sending telemetry: " + telemetry)
    sender.send(EventData(telemetry))
write_client.stop()

```

El formato de mensaje canónico es así:

```json
{
"deviceid": "<id of the Device created>",
"timestamp": "<timestamp in ISO 8601 format>",
"version" : "1",
"sensors": [
    {
      "id": "<id of the sensor created>",
      "sensordata": [
        {
          "timestamp": "< timestamp in ISO 8601 format >",
          "<sensor measure name (as defined in the Sensor Model)>": <value>
        },
        {
          "timestamp": "<timestamp in ISO 8601 format>",
          "<sensor measure name (as defined in the Sensor Model)>": <value>
        }
      ]
    }
 ]
}
```
Todos los nombres de clave del código JSON de telemetría deben estar en minúscula. Algunos ejemplos son deviceid y sensordata.

A continuación se incluye un mensaje de telemetría de ejemplo:


```json
{
  "deviceid": "7f9b4b92-ba45-4a1d-a6ae-c6eda3a5bd12",
  "timestamp": "2019-06-22T06:55:02.7279559Z",
  "version" : "1",
  "sensors": [
    {
      "id": "d8e7beb4-72de-4eed-9e00-45e09043a0b3",
      "sensordata": [
        {
          "timestamp": "2019-06-22T06:55:02.7279559Z",
          "hum_max": 15
        },
        {
          "timestamp": "2019-06-22T06:55:02.7279559Z",
          "hum_min": 42
        }
      ]
    },
    {
      "id": "d8e7beb4-72de-4eed-9e00-45e09043a0b3",
      "sensordata": [
        {
          "timestamp": "2019-06-22T06:55:02.7279559Z",
          "hum_max": 20
        },
        {
          "timestamp": "2019-06-22T06:55:02.7279559Z",
          "hum_min": 89
        }
      ]
    }
  ]
}

```

> [!NOTE]
> Las secciones siguientes están relacionadas con otros cambios (por ejemplo, interfaz de usuario, administración de errores, etc.) a la que el partner del sensor puede hacer referencia para desarrollar el componente Translator.


## <a name="link-a-farmbeats-account"></a>Vinculación de una cuenta de FarmBeats

Después de que los clientes hayan comprado e implementado dispositivos o sensores, pueden acceder a la telemetría y a los datos del dispositivo en el portal de software como servicio (SaaS) de los partners de dispositivo. Los partners de dispositivo pueden permitir a los clientes vincular su cuenta a su instancia de FarmBeats en Azure si proporcionan una manera de introducir las credenciales siguientes:

   - Nombre para mostrar (un campo opcional para que el usuario defina un nombre para esta integración)
   - Punto de conexión de API
   - Id. de inquilino
   - Id. de cliente
   - Secreto del cliente
   - Cadena de conexión de EventHub
   - Fecha de inicio

   > [!NOTE]
   > La fecha de inicio habilita la fuente de distribución de datos históricos, es decir, los datos a partir de la fecha especificada por el usuario.

## <a name="unlink-farmbeats"></a>Desvinculación de FarmBeats

Los partners de dispositivo pueden permitir a los clientes desvincular una integración de FarmBeats existente. La desvinculación de FarmBeats no debería eliminar los metadatos de los dispositivos o sensores que se han creado en FarmBeats Datahub. El proceso de desvinculación hace lo siguiente:

   - Detiene el flujo de telemetría.
   - Elimina y borra las credenciales de integración del asociado de dispositivo.

## <a name="edit-farmbeats-integration"></a>Edición de la integración de FarmBeats

Los partners de dispositivo pueden permitir a los clientes modificar la configuración de la integración de FarmBeats, si cambia el secreto de cliente o la cadena de conexión. En este caso, solo se pueden editar los campos siguientes:

   - Nombre para mostrar (si procede)
   - Secreto de cliente (debe mostrarse con el formato "2x8***********" o la característica Mostrar u ocultar, en lugar de con texto no cifrado)
   - Cadena de conexión (debe mostrarse con el formato "2x8***********" o la característica Mostrar u ocultar, en lugar de texto no cifrado)

## <a name="view-the-last-telemetry-sent"></a>Visualización de la última telemetría enviada

Los partners de dispositivo pueden permitir a los clientes ver la marca de tiempo de la última telemetría enviada, que se encuentra en **Telemetry Sent** (Telemetría enviada). Esta es la hora a la que se envió correctamente la telemetría más reciente a FarmBeats.

## <a name="troubleshooting-and-error-management"></a>Solución de problemas y administración de errores

**Mecanismo para solución de problemas o soporte técnico**

Si el cliente no puede recibir telemetría o datos de dispositivos en la instancia de FarmBeats especificada, el partner de dispositivo debe proporcionar soporte técnico y un mecanismo para solucionar el problema.

**Retención de datos de telemetría**

Los datos de telemetría también deben conservarse durante un período predefinido, de modo que este pueda ser útil para depurar o volver a enviar la telemetría en caso de error o pérdida de datos.

**Notificación y administración de errores**

Si un error afecta al flujo de datos de telemetría o a la integración de datos o metadatos de los dispositivos o sensores en el sistema del partner de dispositivo, el cliente debe recibir una notificación. También se debe diseñar e implementar un mecanismo para resolver los errores.

**Lista de comprobación de conexión**

Los asociados o fabricantes de dispositivos pueden usar la siguiente lista de comprobación para asegurarse de que las credenciales proporcionadas por el cliente son correctas:

   - Compruebe si se recibe un token de acceso con las credenciales proporcionadas.
   - Compruebe si una llamada API se realiza correctamente con el token de acceso recibido.
   - Compruebe si se ha establecido la conexión de cliente de EventHub.

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre la API REST, consulte [API REST](rest-api-in-azure-farmbeats.md).
