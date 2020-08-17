---
title: 'Tutorial: Combinación de datos de sensores con datos de previsión meteorológica mediante Azure Notebooks (Python) | Microsoft Azure Maps'
description: En este tutorial se muestra cómo combinar datos de sensores con datos de previsión meteorológica del servicio meteorológico de Microsoft Azure Maps mediante Azure Notebooks (Python).
author: anastasia-ms
ms.author: v-stharr
ms.date: 01/29/2020
ms.topic: tutorial
ms.service: azure-maps
services: azure-maps
manager: philmea
ms.custom: mvc, devx-track-python
ms.openlocfilehash: 4ec51bc84041008c0b843a8549e299747c3134a3
ms.sourcegitcommit: 7fe8df79526a0067be4651ce6fa96fa9d4f21355
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87851652"
---
# <a name="tutorial-join-sensor-data-with-weather-forecast-data-by-using-azure-notebooks-python"></a>Tutorial: Combinación de datos de sensores con datos de previsión meteorológica mediante Azure Notebooks (Python)

La energía eólica es una fuente de energía alternativa a los combustibles fósiles para combatir el cambio climático. Dado que el viento no es coherente por naturaleza, los operadores de energía eólica necesitan crear modelos de aprendizaje automático (ML) para predecir la capacidad de la energía del viento. Esta predicción es necesaria para satisfacer la demanda de electricidad y garantizar la estabilidad de la red. En este tutorial, veremos cómo se combinan los datos de previsión meteorológica de Azure Maps con los datos de demostración de las lecturas meteorológicas. Los datos de previsión meteorológica se solicitan mediante una llamada al servicio meteorológico de Azure Maps.

En este tutorial, aprenderá lo siguiente:

> [!div class="checklist"]
> * Trabajar con archivos de datos en [Azure Notebooks](https://docs.microsoft.com/azure/notebooks) en la nube.
> * Cargar datos de demostración desde un archivo.
> * Llamada a las API REST de Azure Maps en Python.
> * Representar datos de ubicación en el mapa.
> * Enriquecer los datos de demostración con datos meteorológicos de la [previsión diaria](https://aka.ms/AzureMapsWeatherDailyForecast) de Azure Maps.
> * Trazar datos de previsión en gráficos.


## <a name="prerequisites"></a>Prerrequisitos

Para completar este tutorial, primero debe:

1. Crear una suscripción de cuenta de Azure Maps en el plan de tarifa S0; para ello, siga las instrucciones de [Crear una cuenta](quick-demo-map-app.md#create-an-azure-maps-account).
2. Obtener la clave de suscripción principal de la cuenta; para ello, siga las instrucciones que se indican en [Obtención de la clave principal](quick-demo-map-app.md#get-the-primary-key-for-your-account).


Para más información sobre la autenticación en Azure Maps, consulte [Administración de la autenticación en Azure Maps](./how-to-manage-authentication.md).

Para familiarizarse con Azure Notebooks y saber cómo comenzar, siga las instrucciones de [Creación de un cuaderno de Azure](https://docs.microsoft.com/azure/azure-maps/tutorial-ev-routing#create-an-azure-notebook).

> [!Note]
> El archivo de Jupyter Notebook de este proyecto se puede descargar desde el [repositorio de Jupyter Notebook de Azure Maps](https://github.com/Azure-Samples/Azure-Maps-Jupyter-Notebook/tree/master/AzureMapsJupyterSamples/Tutorials/Analyze%20Weather%20Data).

## <a name="load-the-required-modules-and-frameworks"></a>Carga de los módulos y marcos necesarios

Ejecute el siguiente script para cargar todos los módulos y marcos necesarios:

```python
import pandas as pd
import datetime
from IPython.display import Image, display
!pip install aiohttp
import aiohttp
```

## <a name="import-weather-data"></a>Importación de datos meteorológicos

En este tutorial, usaremos las lecturas de datos meteorológicos de los sensores instalados en cuatro turbinas eólicas diferentes. Los datos de ejemplo se componen de 30 días de lecturas meteorológicas. Estas lecturas se recopilan de centros de datos meteorológicos próximos a cada ubicación de las turbinas. Los datos de demostración contienen lecturas de datos para la temperatura, la velocidad del viento y la dirección. Puede descargar los datos de demostración desde [aquí](https://github.com/Azure-Samples/Azure-Maps-Jupyter-Notebook/tree/master/AzureMapsJupyterSamples/Tutorials/Analyze%20Weather%20Data/data). El script siguiente importa los datos de demostración en Azure Notebooks.

```python
df = pd.read_csv("./data/weather_dataset_demo.csv")
```

## <a name="request-daily-forecast-data"></a>Solicitud de los datos de previsión diarios

En nuestro escenario, nos gustaría solicitar la previsión diaria de cada ubicación de sensores. El siguiente script llama a [Daily Forecast API](https://aka.ms/AzureMapsWeatherDailyForecast) del servicio de climatología de Azure Maps. Esta API devuelve el pronóstico meteorológico de cada molino de viento para los próximos 15 días a partir de la fecha actual.


```python
subscription_key = "Your Azure Maps key"

# Get a lists of unique station IDs and their coordinates 
station_ids = pd.unique(df[['StationID']].values.ravel())
coords = pd.unique(df[['latitude','longitude']].values.ravel())

years,months,days = [],[],[]
dates_check=set()
wind_speeds, wind_direction = [], []

# Call azure maps weather service to get daily forecast data for 15 days from current date
session = aiohttp.ClientSession()
j=-1
for i in range(0, len(coords), 2):
    wind_speeds.append([])
    wind_direction.append([])
    
    query = str(coords[i])+', '+str(coords[i+1])
    forecast_response = await(await session.get("https://atlas.microsoft.com/weather/forecast/daily/json?query={}&api-version=1.0&subscription-key={}&duration=15".format(query, subscription_key))).json()
    j+=1
    for day in range(len(forecast_response['forecasts'])):
            date = forecast_response['forecasts'][day]['date'][:10]
            wind_speeds[j].append(forecast_response['forecasts'][day]['day']['wind']['speed']['value'])
            wind_direction[j].append(forecast_response['forecasts'][day]['day']['windGust']['direction']['degrees'])
            
            if date not in dates_check:
                year,month,day= date.split('-')
                years.append(year)
                months.append(month)
                days.append(day)
                dates_check.add(date)
            
await session.close()
```

El siguiente script representa las ubicaciones de las turbinas en el mapa mediante una llamada al [servicio Get Map Image](https://docs.microsoft.com/rest/api/maps/render/getmapimage) de Azure Maps.

```python
# Render the turbine locations on the map by calling the Azure Maps Get Map Image service
session = aiohttp.ClientSession()

pins="default|la-25+60|ls12|lc003C62|co9B2F15||'Location A'{} {}|'Location B'{} {}|'Location C'{} {}|'Location D'{} {}".format(coords[1],coords[0],coords[3],coords[2],coords[5],coords[4], coords[7],coords[6])

image_response = "https://atlas.microsoft.com/map/static/png?subscription-key={}&api-version=1.0&layer=basic&style=main&zoom=6&center={},{}&pins={}".format(subscription_key,coords[7],coords[6],pins)

static_map_response = await session.get(image_response)

poi_range_map = await static_map_response.content.read()

await session.close()

display(Image(poi_range_map))
```

![Ubicaciones de las turbinas](./media/weather-service-tutorial/location-map.png)


Vamos a agrupar los datos de previsión con los de demostración en función del identificador de la estación. El identificador de la estación es para el centro de datos meteorológicos. Esta agrupación aumenta los datos de demostración con los datos de previsión.

```python
# Group forecasted data for all locations
df = df.reset_index(drop=True)
forecast_data = pd.DataFrame(columns=['StationID','latitude','longitude','Year','Month','Day','DryBulbCelsius','WetBulbFarenheit','WetBulbCelsius','DewPointFarenheit','DewPointCelsius','RelativeHumidity','WindSpeed','WindDirection'])

for i in range(len(station_ids)):
    loc_forecast = pd.DataFrame({'StationID':station_ids[i], 'latitude':coords[0], 'longitude':coords[1], 'Year':years, 'Month':months, 'Day':days, 'WindSpeed':wind_speeds[i], 'WindDirection':wind_direction[i]})
    forecast_data = pd.concat([forecast_data,loc_forecast], axis=0, sort=False)
    
combined_weather_data = pd.concat([df,forecast_data])
grouped_weather_data = combined_weather_data.groupby(['StationID'])
```

En la tabla siguiente se muestran los datos históricos y de previsión combinados de una de las ubicaciones de las turbinas.

```python
# Display data for first location
grouped_weather_data.get_group(station_ids[0]).reset_index()
```

<center>

![Datos agrupados](./media/weather-service-tutorial/grouped-data.png)</center>

## <a name="plot-forecast-data"></a>Trazado de los datos de previsión

Vamos a trazar los valores pronosticados con respecto a los días para los que están previstos. Este trazado nos permite ver los cambios de velocidad y dirección del viento durante los próximos 15 días.

```python
# Plot wind speed
curr_date = datetime.datetime.now().date()
windsPlot_df = pd.DataFrame({ 'Location A': wind_speeds[0], 'Location B': wind_speeds[1], 'Location C': wind_speeds[2], 'Location D': wind_speeds[3]}, index=pd.date_range(curr_date,periods=15))
windsPlot = windsPlot_df.plot.line()
windsPlot.set_xlabel("Date")
windsPlot.set_ylabel("Wind speed")
```

```python
#Plot wind direction 
windsPlot_df = pd.DataFrame({ 'Location A': wind_direction[0], 'Location B': wind_direction[1], 'Location C': wind_direction[2], 'Location D': wind_direction[3]}, index=pd.date_range(curr_date,periods=15))
windsPlot = windsPlot_df.plot.line()
windsPlot.set_xlabel("Date")
windsPlot.set_ylabel("Wind direction")
```

En los gráficos siguientes se visualizan los datos de previsión. Para el cambio de velocidad del viento, consulte el gráfico de la izquierda. Para ver el cambio de la dirección del viento, consulte el gráfico de la derecha. Estos datos representan la predicción durante los próximos 15 días a partir del día en que se solicitan los datos.

<center>

![Trazado de la velocidad del viento](./media/weather-service-tutorial/speed-date-plot.png) ![Trazado de la dirección del viento](./media/weather-service-tutorial/direction-date-plot.png)</center>


## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha aprendido cómo llamar a las API REST de Azure Maps para obtener los datos de previsión meteorológica. También ha aprendido cómo visualizar los datos en gráficos.

Para más información sobre cómo llamar a las API REST de Azure Maps dentro de Azure Notebooks, consulte [Enrutamiento de vehículos eléctricos mediante Azure Notebooks](https://docs.microsoft.com/azure/azure-maps/tutorial-ev-routing).

Para explorar las API de Azure Maps que se usan en este tutorial, consulte:

* [Previsión diaria](https://aka.ms/AzureMapsWeatherDailyForecast)
* [Render - Get Map Image](https://docs.microsoft.com/rest/api/maps/render/getmapimage)

Para obtener una lista completa de las API REST de Azure Maps, consulte [API REST de Azure Maps](https://docs.microsoft.com/azure/azure-maps/consumption-model).

Para más información sobre Azure Notebooks, consulte [Azure Notebooks](https://docs.microsoft.com/azure/notebooks).
