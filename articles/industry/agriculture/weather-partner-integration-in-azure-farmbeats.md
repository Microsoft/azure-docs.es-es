---
title: Integración de asociados de datos meteorológicos
description: En este artículo se describe el modo de integrar un proveedor de datos meteorológicos con FarmBeats.
author: sunasing
ms.topic: article
ms.date: 07/09/2020
ms.author: sunasing
ms.openlocfilehash: a2677b5343b2d65a39e7c9f6d5006db599c1ac73
ms.sourcegitcommit: 3543d3b4f6c6f496d22ea5f97d8cd2700ac9a481
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/20/2020
ms.locfileid: "86497002"
---
# <a name="weather-partner-integration"></a>Integración de asociados de datos meteorológicos

En este artículo se proporciona información sobre el componente de Docker **Connector** de FarmBeats que los proveedores de datos pueden desarrollar para integrarse con FarmBeats aprovechando sus API y enviar datos meteorológicos a FarmBeats. Una vez que los datos están disponibles en FarmBeats, se pueden usar para la fusión de datos y para crear modelos de inteligencia artificial y aprendizaje automático.

 > [!NOTE]
 > Para los fines de esta documentación, usaremos una implementación de referencia que se ha creado con NOAA desde Azure Open Datasets y está disponible en [https://github.com/azurefarmbeats/noaa_docker](https://github.com/azurefarmbeats/noaa_docker).
 > La imagen de Docker correspondiente está disponible en [https://hub.docker.com/r/azurefarmbeats/farmbeats-noaa](https://hub.docker.com/r/azurefarmbeats/farmbeats-noaa).

Un asociado meteorológico deberá proporcionar una imagen o un programa de Docker (con las especificaciones que se mencionan a continuación) y hospedar la imagen de Docker en un registro de contenedor al que puedan acceder los clientes. El asociado meteorológico deberá proporcionar la siguiente información a sus clientes:

- Dirección URL de la imagen de Docker.
- Etiqueta de la imagen de Docker.
- Claves/credenciales para tener acceso a la imagen de Docker.
- Claves/credenciales de API específicas del cliente para acceder a los datos del sistema del asociado meteorológico.
- Detalles de SKU de máquina virtual (los asociados pueden proporcionar esta información en caso de que Docker tenga requisitos de máquina virtual específicos; de lo contrario, los clientes pueden elegir las SKU de máquina virtual admitidas en Azure).

Con la información de Docker anterior, el cliente registrará un asociado meteorológico en la instancia de FarmBeats. Para obtener más información sobre la forma en que los clientes pueden usar Docker para ingerir datos meteorológicos en FarmBeats, consulte la guía para [obtener datos meteorológicos](https://docs.microsoft.com/azure/industry/agriculture/get-weather-data-from-weather-partner).

## <a name="connector-docker-development"></a>Desarrollo del Docker de Connector

**Integración basada en la API REST**

Las API de FarmBeats contienen documentación técnica de Swagger. Para obtener información sobre las API y sus solicitudes o respuestas correspondientes, vea la instancia de [Swagger de FarmBeats](https://aka.ms/farmbeatsswagger). 

Si ha instalado FarmBeats, puede acceder a su instancia de Swagger de FarmBeats en `https://yourfarmbeatswebsitename-api.azurewebsites.net/swagger`.

Tenga en cuenta que se anexa "-api" al nombre del sitio web de FarmBeats.
El punto de conexión de la API será: `https://yourfarmbeatswebsitename-api.azurewebsites.net`

### <a name="datahub-lib"></a>Biblioteca del centro de datos

FarmBeats proporcionará una biblioteca que el asociado meteorológico puede usar. La biblioteca está disponible actualmente como parte de la implementación de referencia [aquí](https://github.com/azurefarmbeats/noaa_docker/tree/master/datahub_lib). En el futuro, estará disponible como un SDK para varios lenguajes.

### <a name="authentication"></a>Authentication

**Autenticación con las API de FarmBeats**

FarmBeats usa la autenticación de portador, y se puede tener acceso a las API proporcionando un token de acceso en la sección de encabezado de la solicitud como se indica a continuación:

```
headers = *{"Authorization": "Bearer " + access_token, …}*
```

El token de acceso se puede solicitar desde una función de Azure que se ejecuta en la instancia de FarmBeats del cliente. La dirección URL de la función de Azure se proporcionará al programa de Docker como argumento, y el token de acceso se puede obtener mediante una solicitud GET en la dirección URL. La respuesta de la dirección URL contendrá el token de acceso. La biblioteca del centro de datos proporciona funciones auxiliares que permiten a los asociados obtener el token de acceso. Más detalles [aquí](https://github.com/azurefarmbeats/noaa_docker/blob/master/datahub_lib/auth/partner_auth_helper.py).

El token de acceso solo es válido durante unas horas, y debe volver a solicitarse cuando haya expirado.

**Autenticación con las API del lado del asociado**

Para permitir que los clientes se autentiquen con las API del lado del asociado durante la ejecución de Docker, los clientes deben proporcionar las credenciales durante el registro del asociado de la siguiente manera:

```json
{
 "partnerCredentials": {
   "key1": "value1",
   "key2": "value2"
   }
}
```
El servicio de API Serializa este diccionario y lo almacena en una instancia de [KeyVault](https://docs.microsoft.com/azure/key-vault/basic-concepts).

[Azure Data Factory](https://docs.microsoft.com/azure/data-factory/introduction) se usa para organizar los trabajos meteorológicos y pone en marcha los recursos para ejecutar el código de Docker. También proporciona un mecanismo para insertar datos de forma segura en la máquina virtual donde se ejecuta el trabajo de Docker. Las credenciales de API, que ahora están almacenadas de forma segura en el almacén KeyVault, se leen como cadenas seguras de KeyVault y se ponen a disposición del usuario como propiedades extendidas en el directorio de trabajo del contenedor de Docker como activity.json (la ruta de acceso al archivo es "/mnt/working_dir/activity.json"). El código de Docker puede leer las credenciales de este archivo durante el tiempo de ejecución para obtener acceso a las API del lado del asociado en nombre del cliente. Las credenciales estarán disponibles en el archivo de la siguiente manera:

```json
{ 
   "typeProperties" : {
      "extendedProperties" : { 
         "partnerCredentials": "" } 
   } 
}
```
Tenga en cuenta que "partnerCredentials" estará disponible del modo exacto proporcionado por el cliente durante el registro del asociado.

La biblioteca de FarmBeats proporciona funciones auxiliares que permiten a los asociados leer las credenciales de las propiedades de la actividad. Más detalles [aquí](https://github.com/azurefarmbeats/noaa_docker/blob/master/datahub_lib/auth/partner_adf_helper.py).

La duración del archivo se extiende solo durante la ejecución del código de Docker, y se eliminará después de que finalice la ejecución de Docker.

Para obtener más información sobre cómo funcionan las canalizaciones y actividades de ADF, vea [https://docs.microsoft.com/azure/data-factory/copy-activity-schema-and-type-mapping](https://docs.microsoft.com/azure/data-factory/copy-activity-schema-and-type-mapping).

**Encabezados de solicitud HTTP**

Estos son los encabezados de solicitud más comunes que deben especificarse al realizar una llamada API a FarmBeats.

**Encabezado** | **Descripción y ejemplo**
--- | ---
Content-Type | El formato de la solicitud (Content-Type: application/<format>). En el caso de las API de centro de datos de FarmBeats, el formato es JSON. Content-Type: application/json
Authorization | Especifica el token de acceso necesario para realizar una llamada API. Autorización: Bearer <Access-Token>
Accept | El formato de respuesta. En el caso de las API de centro de datos de FarmBeats, el formato es JSON. Accept: application/json

## <a name="data-format"></a>Formato de datos

JSON es un formato de datos común independiente del lenguaje que proporciona una representación de texto simple de estructuras de datos arbitrarias. Para obtener más información, visite [json.org](http://json.org).

## <a name="docker-specifications"></a>Especificaciones de Docker

El programa Docker debe tener dos componentes: **Bootstrap** (arranque) y **Jobs** (trabajos). Puede haber más de un trabajo.

### <a name="bootstrap"></a>Bootstrap

Este componente debe ejecutarse cuando el cliente inicia el registro de Docker en FarmBeats. Los argumentos (arg1, arg2) que se pasarán a este programa son:

- Punto de conexión de API de FarmBeats: Punto de conexión de API de FarmBeats para solicitudes de API: Este es el punto de conexión para realizar llamadas de API a la implementación de FarmBeats.
- Dirección URL de la función de Azure: Este es su propio punto de conexión personal que le proporcionará el token de acceso para las API de FarmBeats. Basta con llamar a GET en esta dirección URL para que recuperar el token de acceso en su respuesta.

La responsabilidad del componente de arranque es crear los metadatos necesarios para que los usuarios puedan ejecutar los trabajos para obtener datos meteorológicos. Consulte la implementación de referencia [aquí](https://github.com/azurefarmbeats/noaa_docker). Puede actualizar el archivo bootstrap_manifest.json según sus necesidades y el programa de arranque de referencia creará los metadatos necesarios automáticamente.

Los metadatos siguientes se crean como parte de este proceso. 

 > [!NOTE]
 > **Tenga en cuenta** que si actualiza el bootstrap_manifest.json tal como se mencionó en la [implementación de referencia](https://github.com/azurefarmbeats/noaa_docker), no es necesario crear los metadatos siguientes, ya que el arranque los creará en función del archivo de manifiesto.

- /**WeatherDataModel**:  WeatherDataModel es un modelo que representa los datos meteorológicos y corresponde a los diferentes conjuntos de datos proporcionados por el origen. Por ejemplo, un modelo DailyForecastSimpleModel puede proporcionar la información media de la temperatura, la humedad y las precipitaciones una vez al día, mientras que un modelo DailyForecastAdvancedModel puede proporcionar mucha más información detallada a la hora. Puede crear cualquier número de modelos WeatherDataModels.
- /**JobType**: FarmBeats tiene un sistema de administración de trabajos extensible. Como proveedor de datos meteorológicos, tendrá diferentes conjuntos de datos o API (por ejemplo, GetDailyForecasts), que puede habilitar en FarmBeats como JobType. Una vez que se crea un JobType, un cliente puede desencadenar trabajos de ese tipo para obtener datos meteorológicos para su ubicación o granja de interés (consulte JobType y API de trabajo en el [Swagger de FarmBeats](https://aka.ms/farmbeatsswagger)).

### <a name="jobs"></a>Trabajos

Este componente se invocará cada vez que un usuario de FarmBeats ejecute un trabajo de su /JobType creado como parte del proceso de arranque. El comando de ejecución de Docker para el trabajo se define como parte de **/JobType** que creó.
- La responsabilidad del trabajo será recuperar datos del origen e insertarlos en FarmBeats. Los parámetros necesarios para obtener los datos deben definirse como parte de /JobType en el proceso de arranque.
- Como parte del trabajo, el programa tendrá que crear un elemento **/WeatherDataLocation** basado en el modelo /WeatherDataModel que se creó como parte del proceso de arranque. **/WeatherDataLocation** se corresponde con una ubicación (lat/long) que proporciona el usuario como un parámetro para el trabajo.

### <a name="details-of-the-objects"></a>Detalles de los objetos

  WeatherDataModel | Descripción |
  --- | ---
  Nombre  | Nombre del modelo de datos meteorológicos |
  Descripción  | Proporciona una descripción significativa del modelo. |
  Propiedades  | Propiedades adicionales definidas por el proveedor de datos. |
  weatherMeasures > Name  | Nombre de la medida meteorológica. Por ejemplo, humidity_max. |
  weatherMeasures > DataType  | Double o Enum. Si es Enum, measureEnumDefinition es obligatorio. |
  weatherMeasures > measureEnumDefinition  | Solo es necesario si DataType es Enum. Por ejemplo, { "NoRain": 0, "Snow": 1, "Drizzle": 2, "Rain": 3 } |
  weatherMeasures > Type  | Tipo de datos de telemetría meteorológicos. Por ejemplo "RelativeHumidity". A continuación se indican los tipos definidos por el sistema: AmbientTemperature, NoUnit, CO2, Depth, ElectricalConductivity, LeafWetness, Length, LiquidLevel, Nitrate, O2, PH, Phosphate, PointInTime, Potassium, Pressure, RainGauge, RelativeHumidity, Salinity, SoilMoisture, SoilTemperature, SolarRadiation, State, TimeDuration, UVRadiation, UVIndex, Volume, WindDirection, WindRun, WindSpeed, Evapotranspiration, PAR. Para agregar más, consulte la API /ExtendedType o la sección [Agregar tipos y unidades](weather-partner-integration-in-azure-farmbeats.md#add-extendedtype) más adelante.
  weatherMeasures > Unit | Unidad de datos de telemetría meteorológicos. A continuación se indican las unidades definidas por el sistema: NoUnit, Celsius, Fahrenheit, Kelvin, Rankine, Pascal, Mercury, PSI, MilliMeter, CentiMeter, Meter, Inch, Feet, Mile, KiloMeter, MilesPerHour, MilesPerSecond, KMPerHour, KMPerSecond, MetersPerHour, MetersPerSecond, Degree, WattsPerSquareMeter, KiloWattsPerSquareMeter, MilliWattsPerSquareCentiMeter, MilliJoulesPerSquareCentiMeter, VolumetricWaterContent, Percentage, PartsPerMillion, MicroMol, MicroMolesPerLiter, SiemensPerSquareMeterPerMole, MilliSiemensPerCentiMeter, Centibar, DeciSiemensPerMeter, KiloPascal, VolumetricIonContent, Liter, MilliLiter, Seconds, UnixTimestamp, MicroMolPerMeterSquaredPerSecond y InchesPerHour. Para agregar más, consulte la API /ExtendedType o la sección [Agregar tipos y unidades](weather-partner-integration-in-azure-farmbeats.md#add-extendedtype) más adelante.
  weatherMeasures > AggregationType  | Puede ser None, Average, Maximum, Minimum, StandardDeviation, Sum, Total.
  weatherMeasures > Depth  | La profundidad del sensor en centímetros. Por ejemplo, la medida de la humedad 10 cm bajo el suelo.
  weatherMeasures > Description  | Proporciona una descripción significativa de la medida. |
  **JobType** | **Descripción** |
  Nombre  | Nombre del trabajo, por ejemplo Get_Daily_Forecast; el trabajo que el cliente ejecutará para obtener los datos meteorológicos.|
  pipelineDetails > parameters > name  | Nombre del parámetro. |
  pipelineDetails > parameters > type | String, Int, Float, Bool, Array |
  pipelineDetails > parameters > isRequired | boolean; true si es el parámetro requerido, false si no lo es; el valor predeterminado es true. |
  pipelineDetails > parameters > defaultValue | Valor predeterminado del parámetro. |
  pipelineDetails > parameters > description | Descripción del parámetro |
  Propiedades  | Propiedades adicionales del fabricante.
  Properties > **programRunCommand** | Comando de ejecución de Docker: este comando se ejecutará cuando el cliente ejecute el trabajo meteorológico. |
  **WeatherDataLocation** | **Descripción** |
  weatherDataModelId  | Identificador del modelo WeatherDataModel correspondiente que se creó durante el arranque.|
  ubicación  | Representa la latitud, la longitud y la elevación. |
  Nombre | Nombre del objeto. |
  Descripción | Descripción |
  farmId | **opcional** identificador de la granja, proporcionada por el cliente como parte del parámetro del trabajo. |
  Propiedades  | Propiedades adicionales del fabricante.

 Para obtener información sobre cada uno de los objetos y sus propiedades, consulte [Swagger](https://aka.ms/FarmBeatsSwagger).

 > [!NOTE]
 > Las API devuelven identificadores únicos para cada instancia creada. El componente traductor debe conservar este identificador para la sincronización de los metadatos y la administración de dispositivos.

**Sincronización de metadatos**

El Docker de Connector debería tener la capacidad de enviar actualizaciones en los metadatos. Algunos ejemplos de escenarios de actualización son la adición de nuevos parámetros meteorológicos en el conjunto de datos del proveedor meteorológico, o la adición de funcionalidad (por ejemplo, adición de previsión de 30 días).

> [!NOTE]
> No se admite la eliminación para los metadatos, por ejemplo, el modelo de datos meteorológicos.
>
> Para actualizar los metadatos, es obligatorio llamar a /Get/{ID} en el modelo de datos meteorológicos, actualizar las propiedades cambiadas y, a continuación, realizar una llamada /Put/{ID} para que no se pierdan las propiedades establecidas por el usuario.

## <a name="weather-data-telemetry-specifications"></a>Especificaciones de datos meteorológicos (telemetría)

Los datos meteorológicos se asignan a un mensaje canónico publicado en Azure Event Hub para su procesamiento. Azure EventHub es un servicio que permite la ingesta de datos en tiempo real (telemetría) de aplicaciones y dispositivos conectados. Para enviar datos meteorológicos a FarmBeats, deberá crear un cliente que envíe mensajes a un centro de eventos en FarmBeats. Para obtener más información sobre el envío de telemetría, consulte [Envío de telemetría a un centro de eventos.](https://docs.microsoft.com/azure/event-hubs/event-hubs-dotnet-standard-getstarted-send)

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
   "weatherstations": [
   {
   "id": "id of the WeatherDataLocation",
   "weatherdata": [
   {
     "timestamp": "timestamp of the data. For historical, this is the time for which the observations are sent. For forecast this is the time for which data is forecasted. Format is ISO 8601. Default time-zone is UTC",
     "predictiontimestamp": "timestamp on which the forecast data is predicted i.e time of prediction. Required only for forecast data. Format is ISO 8601. Default timezone is UTC ",
     "weathermeasurename1": <value>,
     "weathermeasurename2": <value>
     }
     ]
    }
   ]
}
```

A continuación se incluye un mensaje de telemetría de ejemplo:

```json
{
   "weatherstations": [
   {
     "id": "5e4b34e7-bf9e-4f39-xxxx-f3c06d0366ea",
     "weatherdata": [
     {
      "timestamp": "2019-07-10T00:00:00Z",
      "predictiontimestamp": "2019-07-05T00:00:00Z",
      "PrecipitationTotalLiquidEquivalent": 0,
      "AvgPressure": 0,
      "AvgRelativeHumidity": 72
     }
    ] 
  }
 ]
}

```

## <a name="troubleshooting-and-error-management"></a>Solución de problemas y administración de errores

**Registro de errores**

Dado que el trabajo del asociado se ejecutará en el marco existente, los errores se registran de la misma manera que los errores de otros trabajos ya existentes en FarmBeats (como GetFarmData, SensorPlacement etc.). La actividad de ADF que se ejecuta dentro de la canalización de ADF registra STDERR y STDOUT. Ambos archivos están disponibles en la cuenta de almacenamiento "datahublogs-xxx" dentro del grupo de recursos FarmBeats.

La biblioteca del centro de datos proporciona funciones auxiliares para habilitar el registro como parte de los registros del centro de datos generales. Más detalles [aquí](https://github.com/azurefarmbeats/noaa_docker/blob/master/datahub_lib/framework/logger.py).

**Mecanismo para solución de problemas o soporte técnico**

En caso de que el cliente no pueda recibir datos meteorológicos en la instancia de FarmBeats especificada, el asociado meteorológico debe proporcionar soporte técnico y un mecanismo para solucionar el problema.

## <a name="add-extendedtype"></a>Adición de ExtendedType

FarmBeats permite agregar nuevas unidades y tipos de medidas de sensor. Tenga en cuenta que un asociado meteorológico puede agregar nuevas unidades o tipos actualizando el archivo bootstrap_manifest.json en la implementación de referencia [aquí](https://github.com/azurefarmbeats/noaa_docker).

Para agregar un nuevo tipo de WeatherMeasure, por ejemplo, "PrecipitationDepth", siga los pasos que se indican a continuación.

1. Realice una solicitud GET en /ExtendedType con el filtro de consulta- key = WeatherMeasureType.
2. Anote el identificador del objeto devuelto.
3. Agregue el nuevo tipo a la lista del objeto devuelto y realice una solicitud PUT en /ExtendedType{ID} con la siguiente lista nueva. La carga de entrada debe ser la misma que la respuesta recibida anteriormente y la nueva unidad anexada al final de la lista de valores.

Para obtener más información sobre la API /ExtendedType, consulte [Swagger](https://aka.ms/FarmBeatsSwagger).

## <a name="next-steps"></a>Pasos siguientes

Ahora tiene un Docker de Connector que se integra con FarmBeats. A continuación, puede ver cómo obtener datos meteorológicos usando Docker en FarmBeats. Consulte [Obtención de datos meteorológicos](get-weather-data-from-weather-partner.md).
