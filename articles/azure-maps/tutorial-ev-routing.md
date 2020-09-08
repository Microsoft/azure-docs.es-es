---
title: 'Tutorial: Enrutamiento de vehículos eléctricos mediante Azure Notebooks (Python) | Microsoft Azure Maps'
description: Enrutamiento de vehículos eléctricos mediante las API de enrutamiento de Microsoft Azure Maps y Azure Notebooks.
author: anastasia-ms
ms.author: v-stharr
ms.date: 11/12/2019
ms.topic: tutorial
ms.service: azure-maps
services: azure-maps
manager: philmea
ms.custom: mvc, devx-track-python
ms.openlocfilehash: 506429f51ac442b73adea98058a833f52a728c72
ms.sourcegitcommit: 628be49d29421a638c8a479452d78ba1c9f7c8e4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88639756"
---
# <a name="tutorial-route-electric-vehicles-by-using-azure-notebooks-python"></a>Tutorial: Enrutamiento de vehículos eléctricos mediante Azure Notebooks (Python)

Azure Maps es una cartera de API de servicios geoespaciales que se integran de forma nativa en Azure. Estas API permiten a los desarrolladores, las empresas y los ISV pueden desarrollar aplicaciones con reconocimiento de ubicación y soluciones de seguimiento de recursos, IoT, movilidad y logística. 

Se puede llamar a las API REST de Azure Maps desde lenguajes como Python y R para habilitar el análisis de datos geoespaciales y los escenarios de aprendizaje automático. Azure Maps ofrece un sólido conjunto de [API de enrutamiento](https://docs.microsoft.com/rest/api/maps/route) que permiten a los usuarios calcular rutas entre varios puntos de datos. Los cálculos se basan en diversas condiciones, como el tipo de vehículo o el área de alcance. 

En este tutorial, ayudará a un conductor cuyo vehículo eléctrico tiene poca batería. El conductor necesita encontrar la estación de carga más cercana desde la ubicación del vehículo.

En este tutorial, aprenderá lo siguiente:

> [!div class="checklist"]
> * Creación y ejecución de un archivo de Jupyter Notebook en [Azure Notebooks](https://docs.microsoft.com/azure/notebooks) en la nube.
> * Llamada a las API REST de Azure Maps en Python.
> * Búsqueda de un intervalo accesible en función del modelo de consumo del vehículo eléctrico
> * Búsqueda de estaciones de carga de vehículos eléctricos en el intervalo de alcance (o isócrono).
> * Representación del límite del intervalo accesible y de las estaciones de carga en un mapa
> * Búsqueda y visualización de la ruta a la estación de carga de vehículos eléctricos más cercana en función del tiempo de conducción.


## <a name="prerequisites"></a>Prerrequisitos 

Para realizar este tutorial, primero debe crear una cuenta de Azure Maps y obtener la clave principal (clave de suscripción). 

Para crear una suscripción a una cuenta de Azure Maps, siga las instrucciones de [Crear una cuenta](quick-demo-map-app.md#create-an-azure-maps-account). Necesita una suscripción a una cuenta de Azure Maps con el nivel de precios S1. 

Para obtener la clave de suscripción principal de la cuenta, siga las instrucciones de [Obtención de la clave principal](quick-demo-map-app.md#get-the-primary-key-for-your-account).

Para más información sobre la autenticación en Azure Maps, consulte [Administración de la autenticación en Azure Maps](./how-to-manage-authentication.md).

## <a name="create-an-azure-notebooks-project"></a>Creación de un proyecto de Azure Notebooks

Para seguir este tutorial, tendrá que crear un proyecto de Azure Notebooks y descargar y ejecutar el archivo de Jupyter Notebook. El archivo de Jupyter Notebook contiene código Python que implementa el escenario de este tutorial. Siga los pasos que se indican a continuación para crear un proyecto de Azure Notebooks y cargar el documento de Jupyter Notebook en él:

1. Vaya a [Azure Notebooks](https://notebooks.azure.com) e inicie sesión. Para más información, consulte [Inicio rápido: Inicio de sesión y establecimiento de un identificador de usuario](https://docs.microsoft.com/azure/notebooks/quickstart-sign-in-azure-notebooks).
1. En parte superior de la página del perfil público, seleccione **My Projects** (Mis proyectos).

    ![Botón My Projects (Mis proyectos)](./media/tutorial-ev-routing/myproject.png)

1. En la página **My Projects** (Mis proyectos), seleccione **New Project** (Nuevo proyecto).
 
   ![Botón New Project (Nuevo proyecto)](./media/tutorial-ev-routing/create-project.png)

1. En el panel **Create New Project** (Crear nuevo proyecto), escriba un nombre de proyecto y un identificador de proyecto.
 
    ![Panel Create New Project (Crear nuevo proyecto)](./media/tutorial-ev-routing/create-project-window.png)

1. Seleccione **Crear**.

1. Una vez creado el proyecto, descargue este [archivo del documento de Jupyter Notebook](https://github.com/Azure-Samples/Azure-Maps-Jupyter-Notebook/blob/master/AzureMapsJupyterSamples/Tutorials/EV%20Routing%20and%20Reachable%20Range/EVrouting.ipynb) del [repositorio de Jupyter Notebook para Azure Maps](https://github.com/Azure-Samples/Azure-Maps-Jupyter-Notebook).

1. En la lista de proyectos de la página **My Projects** (Mis proyectos), seleccione el proyecto y seleccione **Upload** (Cargar) para cargar el archivo del documento de Jupyter Notebook. 

    ![Carga de archivos de Jupyter Notebook](./media/tutorial-ev-routing/upload-notebook.png)

1. Cargue el archivo desde el equipo y, a continuación, seleccione **Done** (Listo).

1. Una vez finalizada la carga correctamente, el archivo se muestra en la página del proyecto. Haga doble clic en el archivo para abrirlo como un cuaderno de Jupyter Notebook.

Intente comprender la funcionalidad que se implementa en el archivo de Jupyter Notebook. Ejecute el código en el archivo de Jupyter Notebook, de celda en celda. Para ejecutar el código de cada celda, seleccione el botón **Run** (Ejecutar) situado en la parte superior de la aplicación Jupyter Notebook.

  ![Botón Run (Ejecutar)](./media/tutorial-ev-routing/run.png)

## <a name="install-project-level-packages"></a>Instalación de los paquetes de nivel de proyecto

Para ejecutar el código de Jupyter Notebook, instale los paquetes en el nivel de proyecto; para ello, siga estos pasos:

1. Descargue el archivo [*requirements.txt*](https://github.com/Azure-Samples/Azure-Maps-Jupyter-Notebook/blob/master/AzureMapsJupyterSamples/Tutorials/EV%20Routing%20and%20Reachable%20Range/requirements.txt) del [repositorio de Jupyter Notebook para Azure Maps](https://github.com/Azure-Samples/Azure-Maps-Jupyter-Notebook) y cárguelo en el proyecto.
1. En el panel del proyecto, seleccione **Project Settings** (Configuración del proyecto). 
1. En el panel **Project Settings** (Configuración del proyecto), seleccione la pestaña **Environment** (Entorno) y, después, seleccione **Add** (Agregar).
1. En **Environment Setup Steps** (Pasos para la configuración del entorno), haga lo siguiente:   
    a. En la primera lista desplegable, seleccione **Requirements.txt**.  
    b. En la segunda lista desplegable, seleccione el archivo *requirements.txt*.  
    c. En la tercera lista desplegable, seleccione la versión **Python Version 3.6**.
1. Seleccione **Guardar**.

    ![Instalar paquetes](./media/tutorial-ev-routing/install-packages.png)

## <a name="load-the-required-modules-and-frameworks"></a>Carga de los módulos y marcos necesarios

Ejecute el siguiente script para cargar todos los módulos y marcos necesarios.

```Python
import time
import aiohttp
import urllib.parse
from IPython.display import Image, display
```

## <a name="request-the-reachable-range-boundary"></a>Solicitud del límite del intervalo de alcance

Una empresa de entrega de paquetes tiene algunos vehículos eléctricos en su flota. Durante el día, los vehículos eléctricos deben recargarse sin tener que volver al almacén. Cada vez que la carga restante cae a menos de una hora, se busca un conjunto de estaciones de carga que estén dentro de un alcance accesible. Básicamente, se busca una estación de carga cuando la batería tiene poca carga, y se obtiene la información sobre el límite del alcance de las estaciones de carga. 

Dado que la compañía prefiere el uso de rutas equilibradas por economía y velocidad, el valor de routeType solicitado es *eco*. El siguiente script llama a [Get Route Range API](https://docs.microsoft.com/rest/api/maps/route/getrouterange) del servicio de enrutamiento de Azure Maps. Usa parámetros para el modelo de consumo del vehículo. Después, el script analiza la respuesta para crear un objeto de tipo polígono con el formato GeoJSON, que representa el intervalo máximo de alcance del automóvil.

Para determinar los límites del intervalo de alcance del vehículo eléctrico, ejecute el script de la siguiente celda:

```python
subscriptionKey = "Your Azure Maps key"
currentLocation = [34.028115,-118.5184279]
session = aiohttp.ClientSession()

# Parameters for the vehicle consumption model 
travelMode = "car"
vehicleEngineType = "electric"
currentChargeInkWh=45
maxChargeInkWh=80
timeBudgetInSec=550
routeType="eco"
constantSpeedConsumptionInkWhPerHundredkm="50,8.2:130,21.3"


# Get boundaries for the electric vehicle's reachable range.
routeRangeResponse = await (await session.get("https://atlas.microsoft.com/route/range/json?subscription-key={}&api-version=1.0&query={}&travelMode={}&vehicleEngineType={}&currentChargeInkWh={}&maxChargeInkWh={}&timeBudgetInSec={}&routeType={}&constantSpeedConsumptionInkWhPerHundredkm={}"
                                              .format(subscriptionKey,str(currentLocation[0])+","+str(currentLocation[1]),travelMode, vehicleEngineType, currentChargeInkWh, maxChargeInkWh, timeBudgetInSec, routeType, constantSpeedConsumptionInkWhPerHundredkm))).json()

polyBounds = routeRangeResponse["reachableRange"]["boundary"]

for i in range(len(polyBounds)):
    coordList = list(polyBounds[i].values())
    coordList[0], coordList[1] = coordList[1], coordList[0]
    polyBounds[i] = coordList

polyBounds.pop()
polyBounds.append(polyBounds[0])

boundsData = {
               "geometry": {
                 "type": "Polygon",
                 "coordinates": 
                   [
                      polyBounds
                   ]
                }
             }
```

## <a name="search-for-electric-vehicle-charging-stations-within-the-reachable-range"></a>Búsqueda de estaciones de carga de vehículos eléctricos en el intervalo de alcance

Una vez que haya determinado el intervalo de alcance (isócrono) del vehículo eléctrico, puede buscar las estaciones de carga en ese intervalo. 

El siguiente script llama a [Post Search Inside Geometry API](https://docs.microsoft.com/rest/api/maps/search/postsearchinsidegeometry) de Azure Maps. Busca estaciones de carga para el vehículo eléctrico, dentro de los límites del intervalo máximo accesible del vehículo. A continuación, el script analiza la respuesta en una matriz de ubicaciones cubiertas.

Ejecute el siguiente script para buscar estaciones de carga de vehículos eléctricos dentro del intervalo de alcance:

```python
# Search for electric vehicle stations within reachable range.
searchPolyResponse = await (await session.post(url = "https://atlas.microsoft.com/search/geometry/json?subscription-key={}&api-version=1.0&query=electric vehicle station&idxSet=POI&limit=50".format(subscriptionKey), json = boundsData)).json() 

reachableLocations = []
for loc in range(len(searchPolyResponse["results"])):
                location = list(searchPolyResponse["results"][loc]["position"].values())
                location[0], location[1] = location[1], location[0]
                reachableLocations.append(location)
```

## <a name="upload-the-reachable-range-and-charging-points-to-azure-maps-data-service"></a>Carga del intervalo de alcance y de los puntos de carga en el servicio de datos de Azure Maps

Querrá visualizar en un mapa las estaciones de carga y el límite del alcance máximo al que puede llegar el vehículo eléctrico. Para ello, cargue los datos de límites y los datos de las estaciones de carga como objetos GeoJSON en el servicio de datos de Azure Maps. Use [Data Upload API](https://docs.microsoft.com/rest/api/maps/data/uploadpreview). 

Ejecute las dos celdas siguientes para cargar los datos del límite y los datos de puntos de carga en el servicio de datos de Azure Maps:

```python
rangeData = {
  "type": "FeatureCollection",
  "features": [
    {
      "type": "Feature",
      "properties": {},
      "geometry": {
        "type": "Polygon",
        "coordinates": [
          polyBounds
        ]
      }
    }
  ]
}

# Upload the range data to Azure Maps Data Service.
uploadRangeResponse = await session.post("https://atlas.microsoft.com/mapData/upload?subscription-key={}&api-version=1.0&dataFormat=geojson".format(subscriptionKey), json = rangeData)

rangeUdidRequest = uploadRangeResponse.headers["Location"]+"&subscription-key={}".format(subscriptionKey)

while True:
    getRangeUdid = await (await session.get(rangeUdidRequest)).json()
    if 'udid' in getRangeUdid:
        break
    else:
        time.sleep(0.2)
rangeUdid = getRangeUdid["udid"]
```

```python
poiData = {
    "type": "FeatureCollection",
    "features": [
      {
        "type": "Feature",
        "properties": {},
        "geometry": {
            "type": "MultiPoint",
            "coordinates": reachableLocations
        }
    }
  ]
}

# Upload the electric vehicle charging station data to Azure Maps Data Service.
uploadPOIsResponse = await session.post("https://atlas.microsoft.com/mapData/upload?subscription-key={}&api-version=1.0&dataFormat=geojson".format(subscriptionKey), json = poiData)

poiUdidRequest = uploadPOIsResponse.headers["Location"]+"&subscription-key={}".format(subscriptionKey)

while True:
    getPoiUdid = await (await session.get(poiUdidRequest)).json()
    if 'udid' in getPoiUdid:
        break
    else:
        time.sleep(0.2)
poiUdid = getPoiUdid["udid"]
```

## <a name="render-the-charging-stations-and-reachable-range-on-a-map"></a>Representación de las estaciones de carga y el intervalo de alcance en un mapa

Una vez que haya cargado los datos en el servicio de datos, llame al servicio de Azure Maps [Get Map Image Service](https://docs.microsoft.com/rest/api/maps/render/getmapimage). Este servicio ejecuta el siguiente script para representar los puntos de carga y el alcance máximo en la imagen de mapa estática:

```python
# Get boundaries for the bounding box.
def getBounds(polyBounds):
    maxLon = max(map(lambda x: x[0], polyBounds))
    minLon = min(map(lambda x: x[0], polyBounds))

    maxLat = max(map(lambda x: x[1], polyBounds))
    minLat = min(map(lambda x: x[1], polyBounds))
    
    # Buffer the bounding box by 10 percent to account for the pixel size of pins at the ends of the route.
    lonBuffer = (maxLon-minLon)*0.1
    minLon -= lonBuffer
    maxLon += lonBuffer

    latBuffer = (maxLat-minLat)*0.1
    minLat -= latBuffer
    maxLat += latBuffer
    
    return [minLon, maxLon, minLat, maxLat]

minLon, maxLon, minLat, maxLat = getBounds(polyBounds)

path = "lcff3333|lw3|la0.80|fa0.35||udid-{}".format(rangeUdid)
pins = "custom|an15 53||udid-{}||https://raw.githubusercontent.com/Azure-Samples/AzureMapsCodeSamples/master/AzureMapsCodeSamples/Common/images/icons/ev_pin.png".format(poiUdid)

encodedPins = urllib.parse.quote(pins, safe='')

# Render the range and electric vehicle charging points on the map.
staticMapResponse =  await session.get("https://atlas.microsoft.com/map/static/png?api-version=1.0&subscription-key={}&pins={}&path={}&bbox={}&zoom=12".format(subscriptionKey,encodedPins,path,str(minLon)+", "+str(minLat)+", "+str(maxLon)+", "+str(maxLat)))

poiRangeMap = await staticMapResponse.content.read()

display(Image(poiRangeMap))
```

![Mapa que muestra el intervalo de ubicación](./media/tutorial-ev-routing/location-range.png)


## <a name="find-the-optimal-charging-station"></a>Búsqueda de la estación de carga óptima

En primer lugar, quiere determinar todas las posibles estaciones de carga dentro del alcance accesible. A continuación, querrá saber cuál de ellas se puede alcanzar en el menor tiempo posible. 

El siguiente script llama a [Matrix Routing API](https://docs.microsoft.com/rest/api/maps/route/postroutematrix) de Azure Maps. Devuelve la ubicación del vehículo especificado, el tiempo de desplazamiento y la distancia hasta cada estación de carga. El script de la siguiente celda analiza la respuesta para obtener la ubicación de la estación de carga más cercana dentro del alcance en función del tiempo.

Ejecute el script de la siguiente celda para encontrar la estación de carga más cercana dentro del alcance en la mínima cantidad de tiempo:

```python
locationData = {
            "origins": {
              "type": "MultiPoint",
              "coordinates": [[currentLocation[1],currentLocation[0]]]
            },
            "destinations": {
              "type": "MultiPoint",
              "coordinates": reachableLocations
            }
         }

# Get the travel time and distance to each specified charging station.
searchPolyRes = await (await session.post(url = "https://atlas.microsoft.com/route/matrix/json?subscription-key={}&api-version=1.0&routeType=shortest&waitForResults=true".format(subscriptionKey), json = locationData)).json()

distances = []
for dist in range(len(reachableLocations)):
    distances.append(searchPolyRes["matrix"][0][dist]["response"]["routeSummary"]["travelTimeInSeconds"])

minDistLoc = []
minDistIndex = distances.index(min(distances))
minDistLoc.extend([reachableLocations[minDistIndex][1], reachableLocations[minDistIndex][0]])
closestChargeLoc = ",".join(str(i) for i in minDistLoc)
```

## <a name="calculate-the-route-to-the-closest-charging-station"></a>Cálculo de la ruta a la estación de carga más cercana

Ahora que ha encontrado la estación de carga más cercana, puede llamar a [Get Route Directions API](https://docs.microsoft.com/rest/api/maps/route/getroutedirections) para solicitar la ruta detallada desde la ubicación actual del vehículo eléctrico a la estación de carga.

Ejecute el script de la siguiente celda para obtener la ruta a la estación de carga y analizar la respuesta para crear un objeto GeoJSON que representa esa ruta:

```python
# Get the route from the electric vehicle's current location to the closest charging station. 
routeResponse = await (await session.get("https://atlas.microsoft.com/route/directions/json?subscription-key={}&api-version=1.0&query={}:{}".format(subscriptionKey, str(currentLocation[0])+","+str(currentLocation[1]), closestChargeLoc))).json()

route = []
for loc in range(len(routeResponse["routes"][0]["legs"][0]["points"])):
                location = list(routeResponse["routes"][0]["legs"][0]["points"][loc].values())
                location[0], location[1] = location[1], location[0]
                route.append(location)

routeData = {
         "type": "LineString",
         "coordinates": route
     }
```

## <a name="visualize-the-route"></a>Visualización de la ruta

Para ayudar a visualizar la ruta, primero debe cargar los datos de la ruta como un objeto GeoJSON en el servicio de datos de Azure Maps. Para ello, use [Data Upload API](https://docs.microsoft.com/rest/api/maps/data/uploadpreview) de Azure Maps. A continuación, llame al servicio de representación, [Get Map Image API](https://docs.microsoft.com/rest/api/maps/render/getmapimage), para representar la ruta en el mapa y visualizarla.

Ejecute el siguiente script para obtener una imagen de la ruta representada en el mapa:

```python
# Upload the route data to Azure Maps Data Service.
routeUploadRequest = await session.post("https://atlas.microsoft.com/mapData/upload?subscription-key={}&api-version=1.0&dataFormat=geojson".format(subscriptionKey), json = routeData)

udidRequestURI = routeUploadRequest.headers["Location"]+"&subscription-key={}".format(subscriptionKey)

while True:
    udidRequest = await (await session.get(udidRequestURI)).json()
    if 'udid' in udidRequest:
        break
    else:
        time.sleep(0.2)

udid = udidRequest["udid"]

destination = route[-1]

destination[1], destination[0] = destination[0], destination[1]

path = "lc0f6dd9|lw6||udid-{}".format(udid)
pins = "default|codb1818||{} {}|{} {}".format(str(currentLocation[1]),str(currentLocation[0]),destination[1],destination[0])


# Get boundaries for the bounding box.
minLat, maxLat = (float(destination[0]),currentLocation[0]) if float(destination[0])<currentLocation[0] else (currentLocation[0], float(destination[0]))
minLon, maxLon = (float(destination[1]),currentLocation[1]) if float(destination[1])<currentLocation[1] else (currentLocation[1], float(destination[1]))

# Buffer the bounding box by 10 percent to account for the pixel size of pins at the ends of the route.
lonBuffer = (maxLon-minLon)*0.1
minLon -= lonBuffer
maxLon += lonBuffer

latBuffer = (maxLat-minLat)*0.1
minLat -= latBuffer
maxLat += latBuffer

# Render the route on the map.
staticMapResponse = await session.get("https://atlas.microsoft.com/map/static/png?api-version=1.0&subscription-key={}&&path={}&pins={}&bbox={}&zoom=16".format(subscriptionKey,path,pins,str(minLon)+", "+str(minLat)+", "+str(maxLon)+", "+str(maxLat)))

staticMapImage = await staticMapResponse.content.read()

await session.close()
display(Image(staticMapImage))
```

![Mapa que muestra la ruta](./media/tutorial-ev-routing/route.png)

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha aprendido a llamar directamente a las API REST de Azure Maps y a visualizar datos de Azure Maps con Python.

Para explorar las API de Azure Maps que se usan en este tutorial, consulte:

* [Get Route Range](https://docs.microsoft.com/rest/api/maps/route/getrouterange)
* [Post Search Inside Geometry](https://docs.microsoft.com/rest/api/maps/search/postsearchinsidegeometry)
* [Data Upload](https://docs.microsoft.com/rest/api/maps/data/uploadpreview)
* [Render - Get Map Image](https://docs.microsoft.com/rest/api/maps/render/getmapimage)
* [Post Route Matrix](https://docs.microsoft.com/rest/api/maps/route/postroutematrix)
* [Get Route Directions](https://docs.microsoft.com/rest/api/maps/route/getroutedirections)

Para obtener una lista completa de las API REST de Azure Maps, consulte [API REST de Azure Maps](https://docs.microsoft.com/azure/azure-maps/consumption-model).

Para más información sobre Azure Notebooks, consulte [Azure Notebooks](https://docs.microsoft.com/azure/notebooks).
