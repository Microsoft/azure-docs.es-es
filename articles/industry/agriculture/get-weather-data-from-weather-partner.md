---
title: Obtener datos meteorológicos de asociados meteorológicos
description: En este artículo se describe cómo obtener datos meteorológicos de los asociados.
author: sunasing
ms.topic: article
ms.date: 03/31/2020
ms.author: sunasing
ms.openlocfilehash: 35acf4e9bd338a0e67b046a59d8884df0626e516
ms.sourcegitcommit: 0b8320ae0d3455344ec8855b5c2d0ab3faa974a3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/30/2020
ms.locfileid: "87429260"
---
# <a name="get-weather-data-from-weather-partners"></a>Obtener datos meteorológicos de asociados meteorológicos

Azure FarmBeats le ayuda a obtener datos meteorológicos de los proveedores de datos correspondientes mediante un marco de conectores basado en Docker. Mediante este marco, los proveedores de datos meteorológicos implementan una instancia de Docker que se puede integrar en FarmBeats. Actualmente, se admiten los siguientes proveedores de datos meteorológicos.

  ![Asociados de FarmBeats](./media/get-sensor-data-from-sensor-partner/dtn-logo.png)
  
   [DTN](https://www.dtn.com/dtn-content-integration/)

Los datos meteorológicos se pueden usar para generar información procesable y compilar modelos de inteligencia artificial o Machine Learning en FarmBeats.

## <a name="before-you-start"></a>Antes de comenzar

Para obtener datos meteorológicos, asegúrese de que ha [instalado FarmBeats](https://aka.ms/farmbeatsinstalldocumentation). La integración de datos meteorológicos se admite en la versión 1.2.11 y en versiones posteriores. 

## <a name="enable-weather-integration-with-farmbeats"></a>Habilitación de la integración meteorológica con FarmBeats

Para empezar a obtener datos meteorológicos en el centro de datos de FarmBeats:

1. Vaya a Swagger en el centro de datos de FarmBeats `https://farmbeatswebsite-api.azurewebsites.net/swagger`.

2. Vaya a /Partner API y, después, realice una solicitud POST. Use la siguiente carga de entrada:

   ```json
   {  
 
     "dockerDetails": {  
       "credentials": { 
         "username": "<credentials to access private docker - not required for public docker>", 
         "password": "<credentials to access private docker – not required for public docker>"  
       },  
       "imageName" : "<docker image name",
       "imageTag" : "<docker image tag, default:latest>",
       "azureBatchVMDetails": {  
         "batchVMSKU" : "<VM SKU. Default is standard_d2_v2>",  
         "dedicatedComputerNodes" : 1,
         "nodeAgentSKUID": "<Node SKU. Default is batch.node.ubuntu 18.04>"  
       }
     },
     "partnerCredentials": {  
       "key1": "value1",  
       "key2": "value2"  
     },  
     "partnerType": "Weather",  
     "name": "<Name of the partner>",  
     "description": "<Description>",
     "properties": {  }  
   }  
   ```

   Por ejemplo, para obtener datos meteorológicos de DTN, use la carga siguiente. Puede modificar el nombre y la descripción según sus preferencias.

   > [!NOTE]
   > El paso siguiente requiere una clave de API. Para obtener una clave para una suscripción a DTN, póngase en contacto con DTN.

   ```json
   {
 
     "dockerDetails": {
       "imageName": "dtnweather/dtn-farm-beats",
       "imageTag": "latest",
       "azureBatchVMDetails": {
         "batchVMSKU": "standard_d2_v2",
         "dedicatedComputerNodes": 1,
         "nodeAgentSKUID": "batch.node.ubuntu 18.04"
       }
     },
     "partnerCredentials": {
      "apikey": "<API key from DTN>"
      },
     "partnerType": "Weather",
     "name": "dtn-weather",
     "description": "DTN registered as a Weather Partner in FarmBeats"
   }  
   ```

   > [!NOTE]
   > Para más información sobre el objeto partner, consulte el [apéndice](get-weather-data-from-weather-partner.md#appendix) de este artículo.

   En el paso anterior, se aprovisionarán los recursos para que Docker se pueda ejecutar en el entorno de FarmBeats del cliente.  

   El aprovisionamiento de los recursos tarda entre 10 y 15 minutos.

3. Compruebe el estado del objeto /Partner que creó en el paso anterior. Para ello, compruebe el estado, realice una solicitud GET en /Partner API y compruebe el estado del objeto partner. Una vez que FarmBeats aprovisiona el asociado correctamente, el estado se establece en **activo**.

4. En /JobType API, realice una solicitud GET. Busque los trabajos meteorológicos que creó anteriormente, en el proceso de adición de asociados. En los trabajos meteorológicos, el campo **pipelineName** tiene el siguiente formato: **partner-name_partner-type_job-name**.

      Ahora, la instancia de FarmBeats tiene un asociado de datos meteorológicos activo. Puede ejecutar trabajos para solicitar datos meteorológicos de una ubicación (latitud y longitud) y un intervalo de fechas determinados. Los tipos de trabajo tendrán detalles sobre cuáles son los parámetros necesarios para ejecutar trabajos meteorológicos.

      Por ejemplo, en el caso de DTN, se crearán los siguientes tipos de trabajo:
   
      - **get_dtn_daily_observations**: obtiene las observaciones diarias de una ubicación y un período de tiempo.
      - **get_dtn_daily_forecasts**: obtiene las previsiones diarias de una ubicación y un período de tiempo.
      - **get_dtn_hourly_observations**: obtiene las observaciones horarias de una ubicación y un período de tiempo.
      - **get_dtn_hourly_forecasts**: obtiene las previsiones horarias de una ubicación y un período de tiempo.

6. Anote el identificador y los parámetros de los tipos de trabajo.

7. Vaya a /Jobs API y realice una solicitud POST en /Jobs. Use la siguiente carga de entrada:

   ```json
    {
         "typeId": "<id of the JobType>",
         "arguments": {
           "additionalProp1": {},
           "additionalProp2": {},
           "additionalProp3": {}
         },
         "name": "<name of the job>",
         "description": "<description>",
         "properties": {}
       }
   ```

   Por ejemplo, para ejecutar **get_dtn_daily_observations**, use la siguiente carga:

   ```json
   { 
         "typeId": "<id of the JobType>",
         "arguments": {
               "latitude": 47.620422,
               "longitude": -122.349358,
               "days": 5
         },
         "name": "<name of the job>",
         "description": "<description>",
         "properties": {}
   }
   ```

8. En el paso anterior se ejecutan los trabajos meteorológicos tal y como se definieron en la instancia de Docker del asociado y se ingieren los datos meteorológicos en FarmBeats. Para comprobar el estado del trabajo, realice una solicitud GET en /Jobs. En la respuesta, busque **currentState**. Cuando termine, **currentState** se establecerá en **Succeeded** (Correcto).

## <a name="query-ingested-weather-data"></a>Consulta de los datos meteorológicos ingeridos

Después de que se completen los trabajos meteorológicos, puede consultar los datos meteorológicos ingeridos para crear modelos o información procesable mediante las API REST del centro de datos de FarmBeats.

Para consultar los datos meteorológicos mediante una API REST de FarmBeats:

1. En [Swagger](https://yourdatahub.azurewebsites.net/swagger), en el centro de datos de FarmBeats, vaya a /WeatherDataLocation API y realice una solicitud GET. La respuesta incluye objetos /WeatherDataLocation creados para la ubicación (latitud y longitud) que la ejecución del trabajo ha especificado. Anote el **identificador** y la propiedad **weatherDataModelId** de los objetos.

2. Realice una solicitud GET/{id} en /WeatherDataModel API para la propiedad **weatherDataModelId**, como ya hizo anteriormente. El modelo de datos meteorológicos muestra todos los metadatos y los detalles sobre los datos meteorológicos ingeridos. Por ejemplo, en el objeto del modelo de datos meteorológicos, la medición meteorológica contiene detalles sobre qué información meteorológica se admite y en qué tipos y unidades se admite. Por ejemplo:

   ```json
   {
       "name": "Temperature <name of the weather measure - this is what we will receive as part of the queried weather data>",
       "dataType": "Double <Data Type - eg. Double, Enum>",
       "type": "AmbientTemperature <Type of measure eg. AmbientTemperature, Wind speed etc.>",
       "unit": "Celsius <Unit of measure eg. Celsius, Percentage etc.>",
       "aggregationType": "None <either of None, Average, Maximum, Minimum, StandardDeviation, Sum, Total>",
       "description": "<Description of the measure>"
   }
   ```

   Anote la respuesta de la llamada GET/{id} del modelo de datos meteorológicos.

3. Vaya a Telemetry API y realice una solicitud POST. Use la siguiente carga de entrada:

   ```json
   {
       "weatherDataLocationId": "<id from step 1 above>",
     "searchSpan": {
       "from": "2020-XX-XXT07:30:00Z",
       "to": "2020-XX-XXT07:45:00Z"
     }
   }
   ```

    La respuesta muestra los datos meteorológicos que están disponibles para el intervalo de tiempo especificado:

   ```json
   {
     "timestamps": [
       "2020-XX-XXT07:30:00Z",
       "2020-XX-XXT07:45:00Z"
     ],
     "properties": [
       {
         "values": [
           "<id of the weatherDataLocation>",
           "<id of the weatherDataLocation>"
         ],
         "name": "Id",
         "type": "String"
       },
       {
         "values": [
           29.1,
           30.2
         ],
         "name": "Temperature <name of the WeatherMeasure as defined in the WeatherDataModel object>",
         "type": "Double <Data Type of the value - eg. Double>"
       }
     ]
   }
   ```

En el ejemplo anterior, la respuesta muestra los datos de dos marcas de tiempo. También muestra el nombre de la medida (temperature) y los valores de los datos meteorológicos indicados en las dos marcas de tiempo. Consulte el modelo de datos meteorológicos asociado para interpretar el tipo y la unidad de los valores notificados.

## <a name="troubleshoot-job-failures"></a>Solución de problemas de errores de trabajos

Para solucionar los errores de los trabajos, [consulte los registros de trabajo](troubleshoot-azure-farmbeats.md#weather-data-job-failures).


## <a name="appendix"></a>Apéndice

|        Asociado   |  Detalles   |
| ------- | -------             |
|     DockerDetails - imageName         |          Nombre de la imagen de Docker. Por ejemplo, docker.io/mydockerimage (imagen en hub.docker.com) o myazureacr.azurecr.io/mydockerimage (imagen en Azure Container Registry), etc. Si no se proporciona ningún registro, el valor predeterminado es hub.docker.com.      |
|          DockerDetails - imageTag             |         Nombre de etiqueta de la imagen de Docker. El valor predeterminado es "latest".     |
|  DockerDetails - credentials      |  Credenciales para acceder a la instancia de Docker privada. El asociado proporciona las credenciales.   |
|  DockerDetails - azureBatchVMDetails - batchVMSKU     |    SKU de la máquina virtual de Azure Batch. Para más información, consulte [todas las máquinas virtuales Linux disponibles](https://docs.microsoft.com/azure/virtual-machines/linux/sizes?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json). <BR> <BR> Los valores válidos son: "Small", "ExtraLarge", "Large", "A8", "A9", "Medium", "A5", "A6", "A7", "STANDARD_D1", "STANDARD_D2", "STANDARD_D3", "STANDARD_D4", "STANDARD_D11", "STANDARD_D12", "STANDARD_D13", "STANDARD_D14", "A10", "A11", "STANDARD_D1_V2", "STANDARD_D2_V2", "STANDARD_D3_V2", "STANDARD_D4_V2", "STANDARD_D11_V2", "STANDARD_D12_V2", "STANDARD_D13_V2", "STANDARD_D14_V2", "STANDARD_G1", "STANDARD_G2", "STANDARD_G3", "STANDARD_G4", "STANDARD_G5", "STANDARD_D5_V2", "BASIC_A1", "BASIC_A2", "BASIC_A3", "BASIC_A4", "STANDARD_A1", "STANDARD_A2", "STANDARD_A3", "STANDARD_A4", "STANDARD_A5", "STANDARD_A6", "STANDARD_A7", "STANDARD_A8", "STANDARD_A9", "STANDARD_A10", "STANDARD_A11", "STANDARD_D15_V2", "STANDARD_F1", "STANDARD_F2", "STANDARD_F4", "STANDARD_F8", "STANDARD_F16", "STANDARD_NV6", "STANDARD_NV12", "STANDARD_NV24", "STANDARD_NC6", "STANDARD_NC12", "STANDARD_NC24", "STANDARD_NC24r", "STANDARD_H8", "STANDARD_H8m", "STANDARD_H16", "STANDARD_H16m", "STANDARD_H16mr", "STANDARD_H16r", "STANDARD_A1_V2", "STANDARD_A2_V2", "STANDARD_A4_V2", "STANDARD_A8_V2", "STANDARD_A2m_V2", "STANDARD_A4m_V2", "STANDARD_A8m_V2", "STANDARD_M64ms", "STANDARD_M128s", "STANDARD_D2_V3". *El valor predeterminado es "STANDARD_D2_V2"* .  |
|    DockerDetails - azureBatchVMDetails - dedicatedComputerNodes   |  Número de nodos de equipos dedicados por grupo de Batch. El valor predeterminado es 1. |
|    DockerDetails - azureBatchVMDetails - nodeAgentSKUID          |    Identificador de SKU del agente del nodo de Azure Batch. Actualmente, solo se admite el agente del nodo de Batch "batch.node.ubuntu 18.04".    |
| DockerDetails - partnerCredentials | Credenciales para llamar a la API de asociado en Docker. El asociado proporciona esta información en función del mecanismo de autorización admitido; por ejemplo, nombre de usuario y contraseña, o claves de API. |
| partnerType | "Weather". En FarmBeats, otros tipos de asociados son "Sensor" e "Imagery".  |
|  name   |   Nombre deseado del asociado en el sistema de FarmBeats.   |
|  description |  Descripción.   |

## <a name="next-steps"></a>Pasos siguientes

Ahora que ha consultado los datos de los sensores desde la instancia de Azure FarmBeats, aprenda a [generar mapas](generate-maps-in-azure-farmbeats.md#generate-maps) para las granjas.
