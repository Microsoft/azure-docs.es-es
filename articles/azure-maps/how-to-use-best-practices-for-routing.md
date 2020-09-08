---
title: Procedimientos recomendados para el servicio Route de Azure Maps en Microsoft Azure Maps
description: Conozca cómo enrutar vehículos mediante el servicio Route de Microsoft Azure Maps.
author: anastasia-ms
ms.author: v-stharr
ms.date: 09/02/2020
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: philmea
ms.openlocfilehash: b957453758b9b8e34989877516a9083f06a85ed8
ms.sourcegitcommit: 5a3b9f35d47355d026ee39d398c614ca4dae51c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89400794"
---
# <a name="best-practices-for-azure-maps-route-service"></a>Procedimientos recomendados para el servicio Route de Azure Maps

Las API Route Directions y Route Matrix del [servicio Route](https://docs.microsoft.com/rest/api/maps/route) de Azure Maps se pueden usar para calcular las horas de llegada estimadas (ETA) de cada ruta solicitada. Las API de Route tienen en cuenta factores como la información del tráfico en tiempo real y los datos de tráfico históricos, como las velocidades de conducción típicas en el día de la semana y la hora del día solicitados. Las API devuelven las rutas más cortas o más rápidas disponibles para varios destinos a la vez de forma secuencial o en orden optimizado, en función de la hora o la distancia. Los usuarios también pueden solicitar rutas y detalles especializados para transeúntes, ciclistas y vehículos comerciales como camiones. En este artículo, se compartirán los procedimientos recomendados para llamar al [servicio Route](https://docs.microsoft.com/rest/api/maps/route) de Azure Maps y aprenderá a:

> [!div class="checklist"]
> * Elegir entre las API Route Directions y la API Matrix Routing
> * Solicitar tiempos de viaje históricos y previstos, en función de datos de tráfico históricos y en tiempo real
> * Solicitar detalles de la ruta, como el tiempo y la distancia, para toda la ruta y cada segmento de la ruta
> * Solicitar la ruta para un vehículo comercial, como un camión
> * Solicitar información de tráfico a lo largo de una ruta, como atascos e información de peajes
> * Solicitar una ruta que conste de una o varias paradas (puntos de referencia)
> * Optimizar una ruta de una o varias paradas para obtener el mejor orden para visitar cada parada (punto de referencia)
> * Optimizar rutas alternativas mediante puntos complementarios; por ejemplo, ofrecer rutas alternativas que pasen por una estación de recarga de vehículos eléctricos
> * Usar el [servicio Route](https://docs.microsoft.com/rest/api/maps/route) con el SDK web de Azure Maps

## <a name="prerequisites"></a>Requisitos previos

1. [Cree una cuenta de Azure Maps](quick-demo-map-app.md#create-an-azure-maps-account).
2. [Obtenga una clave de suscripción principal](quick-demo-map-app.md#get-the-primary-key-for-your-account), también conocida como clave principal o clave de suscripción.

Para obtener más información sobre la cobertura del servicio Route, vea [Cobertura de enrutamiento](routing-coverage.md).

En este artículo se usa la [aplicación Postman](https://www.postman.com/downloads/) para compilar llamadas REST, pero puede elegir cualquier entorno de desarrollo de API.

## <a name="choose-between-route-directions-and-matrix-routing"></a>Elección entre Route Directions y Matrix Routing

Las API Route Directions devuelven las instrucciones de un trayecto, incluidos el tiempo de desplazamiento y las coordenadas. La API Route Matrix permite calcular el tiempo de desplazamiento y las distancias para un conjunto de rutas definidas por las ubicaciones de origen y de destino. Para cada origen determinado, la API Matrix calcula el costo (tiempo y distancia de desplazamiento) de la ruta desde ese origen a cada destino determinado. Todas estas API permiten especificar parámetros como la hora de salida deseada, las horas de llegada y el tipo de vehículo, como un coche o un camión. En todas se usan datos de tráfico predictivos o en tiempo real para devolver las rutas óptimas.

Considere la posibilidad de llamar a las API Route Directions si el escenario es para:

* Solicitar la ruta de conducción más corta o más rápida entre dos o más ubicaciones conocidas, a fin de obtener horas de llegada precisas para los vehículos de reparto.
* Solicitar instrucciones de ruta detalladas, incluida la geometría de la ruta, para visualizar las rutas en el mapa.
* Dada una lista de ubicaciones de clientes, calcular la ruta más corta posible para visitar cada ubicación y volver al origen. Este escenario se conoce comúnmente como "problema del viajante". En una solicitud, puede pasar hasta 150 puntos de referencia (paradas).
* Envíe lotes de consultas a la API Route Directions Batch mediante una sola llamada API.

Considere la posibilidad de llamar a la API Matrix Routing si el escenario es para:

* Calcular el tiempo y la distancia de viaje entre un conjunto de orígenes y destinos. Por ejemplo, tiene 12 conductores y necesita encontrar el conductor disponible más cercano para recoger la entrega de comida del restaurante.
* Ordenar las posibles rutas por su distancia o tiempo de viaje reales. Matrix API solo devuelve los tiempos y las distancias de desplazamiento para cada combinación de origen y destino.
* Agrupar datos en función del tiempo o las distancias de viaje. Por ejemplo, si la empresa tiene 50 empleados, busque todos los que vivan a 20 minutos en coche de la oficina.

Esta es una comparación para mostrar algunas de las funcionalidades de las API Route Directions y Matrix:

| API de Azure Maps | Número máximo de consultas en la solicitud | Áreas que evitar | Rutas para camiones y vehículos eléctricos | Optimización de puntos de referencia y viajante | Puntos complementarios |
| :--------------: |  :--------------: |  :--------------: | :--------------: | :--------------: | :--------------: |
| Get Route Directions | 1 | | X | X | |
| Post Route Directions | 1 | X | X | X | X |
| Post Route Directions Batch | 700 | | X | X | |
| Post Route Matrix | 700 | | X | | |

Para obtener más información sobre las funcionalidades de ruta para vehículos eléctricos, vea el tutorial sobre [rutas para vehículos eléctricos mediante Azure Notebooks (Python)](tutorial-ev-routing.md).

## <a name="request-historic-and-real-time-data"></a>Solicitud de datos históricos y en tiempo real

De forma predeterminada, el servicio Route supone que el modo de viaje es un coche y la hora de salida es ahora. Devuelve la ruta según las condiciones del tráfico en tiempo real, a menos que una solicitud de cálculo de ruta especifique lo contrario. El motor de enrutamiento captura y tiene en cuenta restricciones de tráfico fijas dependientes del tiempo, como el hecho de que no se permitan giros a la izquierda entre las 16:00 y las 18:00. También se tienen en cuenta los cierres de carreteras, como las que están en obras, a menos que solicite específicamente una ruta que omita el tráfico real actual. Para omitir el tráfico actual, establezca `traffic` en `false` en la solicitud de API.

El valor **travelTimeInSeconds** del cálculo de la ruta incluye el retraso debido al tráfico. Para generarlo, se aprovechan los datos actuales y históricos del tiempo de viaje, cuando la hora de salida está establecida en ahora. Si el tiempo de salida se establece en el futuro, las API devuelven los tiempos de desplazamiento previstos en función de los datos históricos.

Si incluye el parámetro **computeTravelTimeFor=all** en la solicitud, el elemento summary de la respuesta tendrá los siguientes campos adicionales, incluidas las condiciones del tráfico histórico:

| Elemento | Descripción|
| :--- | :--- |
| noTrafficTravelTimeInSeconds | Tiempo de desplazamiento estimado calculado como si no hubiera retrasos en la ruta debido a las condiciones del tráfico, por ejemplo, a causa de un atasco. |
| historicTrafficTravelTimeInSeconds | Tiempo de desplazamiento estimado calculado con los datos de tráfico históricos dependientes del tiempo. |
| liveTrafficIncidentsTravelTimeInSeconds | Tiempo de desplazamiento estimado calculado con datos de velocidad en tiempo real. |

En las secciones siguientes se muestra cómo realizar llamadas a las API de Route mediante los parámetros descritos.

### <a name="sample-query"></a>Consulta de ejemplo

En el primero de los ejemplos siguientes, la hora de salida se establece en el futuro, en el momento de redactar este texto.

```http
https://atlas.microsoft.com/route/directions/json?subscription-key=<Your-Azure-Maps-Primary-Subscription-Key>&api-version=1.0&query=51.368752,-0.118332:51.385426,-0.128929&travelMode=car&traffic=true&departAt=2025-03-29T08:00:20&computeTravelTimeFor=all
```

La respuesta contiene un elemento summary (resumen), como el siguiente. Como la hora de salida se establece en el futuro, el valor de **trafficDelayInSeconds** es cero. El valor **travelTimeInSeconds** se calcula con los datos de tráfico histórico dependientes del tiempo. Por tanto, en este caso el valor **travelTimeInSeconds** es igual al valor **historicTrafficTravelTimeInSeconds**.

```json
"summary": {
    "lengthInMeters": 2131,
    "travelTimeInSeconds": 248,
    "trafficDelayInSeconds": 0,
    "departureTime": "2025-03-29T08:00:20Z",
    "arrivalTime": "2025-03-29T08:04:28Z",
    "noTrafficTravelTimeInSeconds": 225,
    "historicTrafficTravelTimeInSeconds": 248,
    "liveTrafficIncidentsTravelTimeInSeconds": 248
},
```

### <a name="sample-query"></a>Consulta de ejemplo

En el segundo ejemplo siguiente, hay una solicitud de enrutamiento en tiempo real, donde el tiempo de salida es ahora. No se especifica explícitamente en la dirección URL porque es el valor predeterminado.

```http
https://atlas.microsoft.com/route/directions/json?subscription-key=<Your-Azure-Maps-Primary-Subscription-Key>&api-version=1.0&query=47.6422356,-122.1389797:47.6641142,-122.3011268&travelMode=car&traffic=true&computeTravelTimeFor=all
```

La respuesta contiene un resumen, como se muestra a continuación. Debido a los atascos, el valor **trafficDelaysInSeconds** es mayor que cero. También es mayor que **historicTrafficTravelTimeInSeconds**.

```json
"summary": {
    "lengthInMeters": 16637, 
    "travelTimeInSeconds": 2905, 
    "trafficDelayInSeconds": 1604, 
    "departureTime": "2020-02-28T01:00:20+00:00",
    "arrivalTime": "2020-02-28T01:48:45+00:00", 
    "noTrafficTravelTimeInSeconds": 872, 
    "historicTrafficTravelTimeInSeconds": 1976, 
    "liveTrafficIncidentsTravelTimeInSeconds": 2905 
},
```

## <a name="request-route-and-leg-details"></a>Solicitud de detalles sobre una ruta y un tramo

De forma predeterminada, el servicio Route devolverá una matriz de coordenadas. La respuesta contendrá las coordenadas que componen el trayecto en una lista denominada `points`. La respuesta de Route también incluye la distancia desde el inicio de la ruta y el tiempo transcurrido estimado. Estos valores se pueden usar para calcular la velocidad media de toda la ruta.

En la imagen siguiente se muestra el elemento `points`.

![Elemento points](media/how-to-use-best-practices-for-routing/points-list-is-hidden-img.png)

Expanda el elemento `point` para ver la lista de coordenadas para el trayecto:

![Elemento points expandido](media/how-to-use-best-practices-for-routing/points-list-img.png)

Las API Route Directions admiten diferentes formatos de instrucciones que se pueden usar mediante la especificación del parámetro **instructionsType**. Para dar formato a las instrucciones y facilitar el procesamiento del equipo, use **instructionsType=coded**. Use **instructionsType=tagged** para mostrar las instrucciones como texto para el usuario. Además, a las instrucciones se les puede aplicar formato de texto, donde se marcan algunos elementos de las instrucciones y la indicación se presenta con un formato especial. Para obtener más información, vea la [lista de tipos de instrucciones compatibles](https://docs.microsoft.com/rest/api/maps/route/postroutedirections#routeinstructionstype).

Cuando se solicitan instrucciones, la respuesta devuelve un nuevo elemento denominado `guidance`. El elemento `guidance` contiene dos fragmentos de información: indicaciones paso a paso e instrucciones resumidas.

![Tipo de instrucciones](media/how-to-use-best-practices-for-routing/instructions-type-img.png)

El elemento `instructions` contiene indicaciones paso a paso para el recorrido, mientras que `instructionGroups` presenta instrucciones resumidas. Cada resumen de instrucciones abarca un segmento del viaje que podría abarcar varias carreteras. Las API pueden devolver detalles de las secciones de una ruta, por ejemplo, el intervalo de coordenadas de un atasco o la velocidad actual del tráfico.

![Instrucciones paso a paso](media/how-to-use-best-practices-for-routing/instructions-turn-by-turn-img.png)

![Instrucciones resumidas](media/how-to-use-best-practices-for-routing/instructions-summary-img.png)

## <a name="request-a-route-for-a-commercial-vehicle"></a>Solicitud de una ruta para un vehículo comercial

Las API de enrutamiento de Azure Maps admiten el enrutamiento de vehículos comerciales, que incluye el de los camiones comerciales. Las API tienen en cuenta límites concretos. Por ejemplo, el alto y el peso del vehículo, y si transporta mercancías peligrosas. Por ejemplo, si un vehículo transporta productos inflamables, el motor de enrutamiento evitará determinados túneles que estén cerca de zonas residenciales.

### <a name="sample-query"></a>Consulta de ejemplo

La solicitud de ejemplo siguiente consulta una ruta para un camión comercial. El camión transporta residuos peligrosos de clase 1.

```http
https://atlas.microsoft.com/route/directions/json?subscription-key=<Your-Azure-Maps-Primary-Subscription-Key>&api-version=1.0&vehicleWidth=2&vehicleHeight=2&vehicleCommercial=true&vehicleLoadType=USHazmatClass1&travelMode=truck&instructionsType=text&query=51.368752,-0.118332:41.385426,-0.128929
```

La API de Route devuelve indicaciones que acomodan las dimensiones del camión y los residuos peligrosos. Puede leer las instrucciones de ruta expandiendo el elemento `guidance`.

![Camión con residuos peligrosos de clase 1](media/how-to-use-best-practices-for-routing/truck-with-hazwaste-img.png)

### <a name="sample-query"></a>Consulta de ejemplo

Si se cambia el valor US Hazmat Class de la consulta anterior, el resultado será una ruta diferente para acomodar este cambio.

```http
https://atlas.microsoft.com/route/directions/json?subscription-key=<Your-Azure-Maps-Primary-Subscription-Key>&api-version=1.0&vehicleWidth=2&vehicleHeight=2&vehicleCommercial=true&vehicleLoadType=USHazmatClass9&travelMode=truck&instructionsType=text&query=51.368752,-0.118332:41.385426,-0.128929
```

La respuesta siguiente es para un camión que transporta materiales peligrosos de clase 9, que son menos peligrosos que los de clase 1. Al expandir el elemento `guidance` para leer las indicaciones, observará que no son las mismas. Hay más instrucciones de enrutamiento para el camión que transporta materiales peligrosos de clase 1.



![Camión con residuos peligrosos de clase 9](media/how-to-use-best-practices-for-routing/truck-with-hazwaste9-img.png)



## <a name="request-traffic-information-along-a-route"></a>Solicitud de información de tráfico a lo largo de una ruta

Con las API Route Direction de Azure Maps, los desarrolladores pueden solicitar detalles de cada tipo de sección si se incluye el parámetro `sectionType` en la solicitud. Por ejemplo, puede solicitar la información de velocidad de cada tramo de un atasco. Consulte la [lista de valores de la clave sectionType](https://docs.microsoft.com/rest/api/maps/route/getroutedirections#sectiontype) para obtener información sobre los distintos detalles que puede solicitar.

### <a name="sample-query"></a>Consulta de ejemplo

En la consulta siguiente se establece `sectionType` en `traffic`. Solicita las secciones que contienen información de tráfico de Seattle a San Diego.

```http
https://atlas.microsoft.com/route/directions/json?subscription-key=<Your-Azure-Maps-Primary-Subscription-Key>&api-version=1.0&sectionType=traffic&query=47.6062,-122.3321:32.7157,-117.1611
```

La respuesta contiene las secciones que son adecuadas para el tráfico en las coordenadas especificadas.

![Secciones de tráfico](media/how-to-use-best-practices-for-routing/traffic-section-type-img.png)

Esta opción se puede usar para colorear las secciones al representar el mapa, como en la imagen siguiente: 

![Secciones coloreadas representadas en el mapa](media/how-to-use-best-practices-for-routing/show-traffic-sections-img.png)

## <a name="calculate-and-optimize-a-multi-stop-route"></a>Cálculo y optimización de una ruta con varias paradas

En la actualidad, Azure Maps proporciona dos formas para optimizar una ruta:

* Optimizaciones basadas en el tipo de ruta solicitado sin cambiar el orden de los puntos de referencia. Puede encontrar los [tipos de ruta admitidos aquí](https://docs.microsoft.com/rest/api/maps/route/postroutedirections#routetype).

* Optimización del viajante, que cambia el orden de los puntos de referencia para obtener el mejor orden para visitar cada parada.

Para el enrutamiento de varias paradas, se pueden especificar hasta 150 puntos de referencia en una única solicitud de ruta. Las ubicaciones de coordenadas inicial y final pueden ser las mismas, como en el caso de un viaje de ida y vuelta. Sin embargo, tendrá que proporcionar por lo menos un punto de referencia adicional para realizar el cálculo de la ruta. Los puntos de referencia se pueden agregar a la consulta entre las coordenadas de origen y de destino.

Si quiere optimizar el mejor orden para visitar los puntos de referencia concretos, debe especificar **computeBestOrder=true**. Este escenario también se conoce como "problema de optimización del viajante".

### <a name="sample-query"></a>Consulta de ejemplo

En la siguiente consulta se solicitan seis puntos de referencia a la ruta, con el parámetro `computeBestOrder` establecido en `false`. También es el valor predeterminado del parámetro `computeBestOrder`.

```http
https://atlas.microsoft.com/route/directions/json?api-version=1.0&subscription-key=<Your-Azure-Maps-Primary-Subscription-Key>&computeBestOrder=false&query=47.606544,-122.336502:47.759892,-122.204821:47.670682,-122.120415:47.480133,-122.213369:47.615556,-122.193689:47.676508,-122.206054:47.495472,-122.360861
```

La respuesta describe que la longitud del trayecto es de 140 851 metros y que se tardarían 9991 segundos en recorrerlo.

![Respuesta no optimizada](media/how-to-use-best-practices-for-routing/non-optimized-response-img.png)

En la imagen siguiente se muestra el trayecto resultante de esta consulta. Este trayecto es uno de los posibles. No es el trayecto óptimo, según la hora o la distancia.

![Imagen no optimizada](media/how-to-use-best-practices-for-routing/non-optimized-image-img.png)



El orden de los puntos de referencia de este trayecto es el siguiente: 0, 1, 2, 3, 4, 5 y 6.

### <a name="sample-query"></a>Consulta de ejemplo

En la consulta se solicitan los mismos seis puntos de referencia del trayecto, como en el ejemplo anterior. En esta ocasión, el parámetro `computeBestOrder` se establece en `true` (la optimización del viajante).

```http
https://atlas.microsoft.com/route/directions/json?api-version=1.0&subscription-key=<Your-Azure-Maps-Primary-Subscription-Key>&computeBestOrder=true&query=47.606544,-122.336502:47.759892,-122.204821:47.670682,-122.120415:47.480133,-122.213369:47.615556,-122.193689:47.676508,-122.206054:47.495472,-122.360861
```

La respuesta describe que la longitud del trayecto es de 91 814 metros y que se tardarían 7797 segundos en recorrerlo. Aquí, los valores de distancia y tiempo de desplazamiento son bajos porque la API ha devuelto la ruta optimizada.

![Respuesta optimizada](media/how-to-use-best-practices-for-routing/optimized-response-img.png)

En la imagen siguiente se muestra el trayecto resultante de esta consulta.

![Imagen optimizada](media/how-to-use-best-practices-for-routing/optimized-image-img.png)

En la ruta óptima, el orden de los puntos de referencia es el siguiente: 0, 5, 1, 2, 4, 3 y 6.

>[!TIP]
> La información del orden optimizado de los puntos de referencia del servicio de enrutamiento proporciona un conjunto de índices. Se excluyen los índices de origen y de destino. Tendrá que incrementar estos valores en 1 para tener en cuenta el origen. Después, agregue el destino al final para obtener la lista completa de puntos de referencia ordenados.

## <a name="calculate-and-bias-alternative-routes-using-supporting-points"></a>Cálculo y sesgo de rutas alternativas mediante puntos complementarios

Es posible que haya situaciones en las que quiera reconstruir una ruta para calcular cero o más rutas alternativas para una ruta de referencia. Por ejemplo, es posible que quiera mostrar a los clientes rutas alternativas que pasen por su tienda. En este caso, tendrá que sesgar una ubicación mediante puntos de referencia. Estos son los pasos para sesgar una ubicación:

1. Calcule una ruta tal cual y obtenga el trayecto de la respuesta de la ruta.
2. Use el trayecto de la ruta para buscar las ubicaciones deseadas a lo largo del trayecto o cercanas a este. Por ejemplo, puede usar la [API Point of Interest](https://docs.microsoft.com/rest/api/maps/search/getsearchpoi) de Azure Maps o consultar datos propios en la base de datos.  
3. Ordene las ubicaciones en función de la distancia con respecto al inicio de la ruta.
4. Agregue estas ubicaciones como puntos complementarios en una nueva solicitud de ruta a la [API Post Route Directions](https://docs.microsoft.com/rest/api/maps/route/postroutedirections). Para obtener más información sobre los puntos complementarios, vea la [documentación de la API Post Route Directions](https://docs.microsoft.com/rest/api/maps/route/postroutedirections#supportingpoints). 

Al llamar a la [API Post Route Directions](https://docs.microsoft.com/rest/api/maps/route/postroutedirections), puede establecer el tiempo de desviación mínimo o las restricciones de distancia, junto con los puntos complementarios. Use estos parámetros si quiere ofrecer rutas alternativas, pero también limitar el tiempo de desplazamiento. Cuando se usan estas restricciones, las rutas alternativas seguirán la ruta de referencia desde el punto de origen durante el tiempo o la distancia especificados. En otras palabras, las demás rutas difieren de la ruta de referencia según las restricciones especificadas.

La imagen siguiente es un ejemplo de representación de rutas alternativas con límites de desviación especificados para el tiempo y la distancia.

![Rutas alternativas](media/how-to-use-best-practices-for-routing/alternative-routes-img.png)

## <a name="use-the-routing-service-in-a-web-app"></a>Uso del servicio de enrutamiento en una aplicación web

El SDK web de Azure Maps proporciona un [módulo de servicios](https://docs.microsoft.com/javascript/api/azure-maps-rest/?view=azure-maps-typescript-latest). Este módulo es una biblioteca auxiliar que facilita el uso de las API REST de Azure Maps en la web o de aplicaciones de Node.js mediante JavaScript o TypeScript. El módulo de servicios se puede usar para representar las rutas devueltas en el mapa. El módulo determina de forma automática qué API se va a usar con las solicitudes GET y POST.

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información, vea:

> [!div class="nextstepaction"]
> [Servicio Route de Azure Maps](https://docs.microsoft.com/rest/api/maps/route)

> [!div class="nextstepaction"]
> [Procedimientos para usar el módulo de servicios](https://docs.microsoft.com/azure/azure-maps/how-to-use-services-module)

> [!div class="nextstepaction"]
> [Representación de la ruta en el mapa](https://docs.microsoft.com/azure/azure-maps/map-route)

> [!div class="nextstepaction"]
> [Paquete de NPM de Azure Maps](https://www.npmjs.com/package/azure-maps-rest  )
