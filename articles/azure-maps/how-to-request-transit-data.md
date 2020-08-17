---
title: Solicitud de información sobre el transporte mediante el servicio Mobility de Microsoft Azure Maps
description: En este artículo, obtendrá información sobre cómo solicitar datos de tránsito público mediante el servicio Mobility de Microsoft Azure Maps.
author: anastasia-ms
ms.author: v-stharr
ms.date: 07/22/2020
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: philmea
ms.custom: mvc
ms.openlocfilehash: 38fc6f213cbd58fc829a6605bdbed7d25e99bb8d
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87319021"
---
# <a name="request-public-transit-data-using-the-azure-maps-mobility-service"></a>Solicitud de información sobre el transporte público mediante el servicio Mobility de Azure Maps

En este artículo se muestra cómo usar el [servicio Mobility](https://aka.ms/AzureMapsMobilityService) de Azure Maps para solicitar datos de transporte público. Los datos de transporte incluyen las paradas, la información de ruta y las estimaciones de tiempo de viaje.

En este artículo, aprenderá a:

* Obtener un identificador de área metropolitana mediante [Get Metro Area API](https://aka.ms/AzureMapsMobilityMetro).
* Solicitar las paradas próximas mediante el servicio [Get Nearby Transit](https://aka.ms/AzureMapsMobilityNearbyTransit).
* Consulte [Get Transit Routes API](https://aka.ms/AzureMapsMobilityTransitRoute) para planear una ruta en transporte público.
* Solicitar el trayecto de la ruta y la programación detallada con [Get Transit Itinerary API](https://aka.ms/https://azure.microsoft.com/services/azure-maps/).

## <a name="prerequisites"></a>Prerrequisitos

1. [Cree una cuenta de Azure Maps](quick-demo-map-app.md#create-an-azure-maps-account).
2. [Obtenga una clave de suscripción principal](quick-demo-map-app.md#get-the-primary-key-for-your-account), también conocida como clave principal o clave de suscripción. Para más información sobre la autenticación en Azure Maps, consulte [Administración de la autenticación en Azure Maps](./how-to-manage-authentication.md).

En este tutorial se usa la aplicación [Postman](https://www.postman.com/), pero puede elegir un entorno de desarrollo de API diferente.

## <a name="get-a-metro-area-id"></a>Obtención de un identificador de área metropolitana

Para solicitar información detallada sobre las empresas de transporte y los tipos de transporte admitidos para un área metropolitana determinada, necesitará el valor `metroId` de esa área. La [API Get Metro Area](https://aka.ms/AzureMapsMobilityMetro) permite solicitar áreas metropolitanas en las que el servicio Mobility de Azure Maps está disponible. La respuesta incluye detalles como `metroId`, `metroName` y una representación de las rutas del área metropolitana en formato GeoJSON.

Vamos a hacer una solicitud para obtener el área metropolitana del identificador de área metropolitana de Seattle-Tacoma. Para solicitar el identificador de un área metropolitana, siga estos pasos:

1. Abra la aplicación Postman y cree una colección para almacenar las solicitudes. Cerca de la parte superior de la aplicación Postman, seleccione**New** (Nuevo). En la ventana **Create New** (Crear nuevo), seleccione **Collection** (Colección).  Asigne un nombre a la colección y seleccione el botón **Create** (Crear).

2. Para crear la solicitud, seleccione **New** (Nuevo) otra vez. En la ventana **Create New** (Crear nuevo), seleccione **Request** (Solicitud). Escriba un valor de **Request name** (Nombre de solicitud) para la solicitud. Seleccione la colección que creó en el paso anterior como la ubicación en la que se va a guardar la solicitud. Después, seleccione **Guardar**.
  
    ![Creación de una solicitud en Postman](./media/how-to-request-transit-data/postman-new.png)

3. Seleccione el método HTTP **GET** en la pestaña del generador y escriba la siguiente dirección URL para realizar una solicitud GET. Reemplace `{subscription-key}` por la clave principal de Azure Maps.

    ```HTTP
    https://atlas.microsoft.com/mobility/metroArea/id/json?subscription-key={subscription-key}&api-version=1.0&query=47.63096,-122.126
    ```

4. Si la solicitud es correcta, recibirá la respuesta siguiente:

    ```JSON
    {
        "results": [
            {
                "metroId": 522,
                "metroName": "Seattle–Tacoma–Bellevue, WA",
                "viewport": {
                    "topLeftPoint": {
                        "latitude": 48.5853,
                        "longitude": -124.80934
                    },
                    "btmRightPoint": {
                        "latitude": 46.90534,
                        "longitude": -121.55032
                    }
                },
                "geometry": {
                    "type": "Polygon",
                    "coordinates": [
                        [
                            [
                                -121.99604,
                                47.16147
                            ],
                            [
                                -121.97051,
                                47.17222
                            ],
                            [
                                -121.96308,
                                47.17671
                            ],
                            ...,
                            ...,
                            ...,
                            [
                                -122.01525,
                                47.16008
                            ],
                            [
                                -122.00553,
                                47.15919
                            ],
                            [
                                -121.99604,
                                47.16147
                            ]
                        ]
                    ]
                }
            }
        ]
    }
    ```

## <a name="request-nearby-transit-stops"></a>Solicitud de paradas cercanas

El servicio [Get Nearby Transit](https://aka.ms/AzureMapsMobilityNearbyTransit) de Azure Maps le permite buscar objetos de transporte. La API devuelve los detalles del objeto de transporte, como paradas de transporte público y bicicletas compartidas en torno a una ubicación determinada. A continuación vamos a solicitar que el servicio busque paradas de transporte público cercanas, en un radio de 300 metros a partir de una ubicación dada.

Para realizar una solicitud a [Get Nearby Transit](https://aka.ms/AzureMapsMobilityNearbyTransit), siga estos pasos:

1. En Postman, haga clic en **New Request** (Nueva solicitud)  | **GET request** (Solicitud GET) y asígnele el nombre **Get Nearby stops** (Obtener paradas cercanas).

2. En la pestaña Builder (Generador), seleccione el método HTTP **GET**, escriba la siguiente dirección URL de solicitud para el punto de conexión de API y haga clic en **Send** (Enviar).

    ```HTTP
    https://atlas.microsoft.com/mobility/transit/nearby/json?subscription-key={subscription-key}&api-version=1.0&query=47.63096,-122.126&radius=300&objectType=stop
    ```

3. Si la solicitud es correcta, la estructura de la respuesta se parecerá a la siguiente:

    ```JSON
    {
        "results": [
            {
                "id": "522---2060603",
                "type": "stop",
                "objectDetails": {
                    "stopKey": "71300",
                    "stopName": "NE 24th St & 162nd Ave NE",
                    "stopCode": "71300",
                    "mainTransitType": "Bus",
                    "mainAgencyId": "522---5872",
                    "mainAgencyName": "Metro Transit"
                },
                "position": {
                    "latitude": 47.631504,
                    "longitude": -122.125275
                },
                "viewport": {
                    "topLeftPoint": {
                        "latitude": 47.632413,
                        "longitude": -122.12659
                    },
                    "btmRightPoint": {
                        "latitude": 47.630594,
                        "longitude": -122.123959
                    }
                }
            },
            {
                "id": "522---2061020",
                "type": "stop",
                "objectDetails": {
                    "stopKey": "68372",
                    "stopName": "NE 24th St & 160th Ave NE",
                    "stopCode": "68372",
                    "mainTransitType": "Bus",
                    "mainAgencyId": "522---5872",
                    "mainAgencyName": "Metro Transit"
                },
                "position": {
                    "latitude": 47.631409,
                    "longitude": -122.127136
                },
                "viewport": {
                    "topLeftPoint": {
                        "latitude": 47.632318,
                        "longitude": -122.128451
                    },
                    "btmRightPoint": {
                        "latitude": 47.630499,
                        "longitude": -122.12582
                    }
                }
            },
            {
                "id": "522---2060604",
                "type": "stop",
                "objectDetails": {
                    "stopKey": "71310",
                    "stopName": "NE 24th St & 160th Ave NE",
                    "stopCode": "71310",
                    "mainTransitType": "Bus",
                    "mainAgencyId": "522---5872",
                    "mainAgencyName": "Metro Transit"
                },
                "position": {
                    "latitude": 47.631565,
                    "longitude": -122.127808
                },
                "viewport": {
                    "topLeftPoint": {
                        "latitude": 47.632474,
                        "longitude": -122.129124
                    },
                    "btmRightPoint": {
                        "latitude": 47.630655,
                        "longitude": -122.126492
                    }
                }
            }
        ]
    } 
    ```

Si observa detenidamente la estructura de respuesta, verá que contiene los parámetros de cada objeto de transporte. Cada objeto de transporte tiene parámetros como `id`, `type`, `stopName`, `mainTransitType`, `mainAgencyName`, y la posición, en coordenadas, del objeto.

Con el fin de aprender, usaremos un `id` de una parada de autobús como origen para nuestra ruta en la sección siguiente.  

## <a name="request-a-transit-route"></a>Solicitud de una ruta de transporte

La [API Get Transit Routes](https://aka.ms/AzureMapsMobilityTransitRoute) de Azure Maps permite planear el viaje. Devuelve las mejores opciones de ruta posibles de un origen a un destino. El servicio proporciona distintos modos de transporte (por ejemplo, a pie, en bicicleta y en transporte público). A continuación buscaremos una ruta desde la parada de autobús más cercana a la torre Space Needle (Seattle).

### <a name="get-location-coordinates-for-destination"></a>Obtención de las coordenadas de posición de destino

Para obtener las coordenadas de posición de la torre Space Needle, usaremos el [servicio Fuzzy Search](https://docs.microsoft.com/rest/api/maps/search/getsearchfuzzy) de Azure Maps.

Para realizar una solicitud a Get Search Fuzzy, siga estos pasos:

1. En Postman, haga clic en **New Request** (Nueva solicitud)  | **GET request** (Solicitud GET) y asígnele el nombre **Get location coordinates** (Obtener coordenadas de la ubicación).

2. En la pestaña Builder (Generador), seleccione el método HTTP **GET**, escriba la siguiente dirección URL de solicitud y haga clic en **Send** (Enviar).

    ```HTTP
    https://atlas.microsoft.com/search/fuzzy/json?subscription-key={subscription-key}&api-version=1.0&query=space needle
    ```

3. Si observa detenidamente la respuesta, contiene varias ubicaciones en los resultados de la búsqueda de Space Needle. Cada resultado contiene las coordenadas de la ubicación en **position**. Copie `lat` y `lon` de **position** del primer resultado.

   ```JSON
   {
        "summary": {
            "query": "space needle",
            "queryType": "NON_NEAR",
            "queryTime": 35,
            "numResults": 8,
            "offset": 0,
            "totalResults": 11,
            "fuzzyLevel": 1
        },
        "results": [
            {
                "type": "POI",
                "id": "US/POI/p0/6993440",
                "score": 4.67369,
                "info": "search:ta:840539001406144-US",
                "poi": {
                    "name": "Space Needle",
                    "phone": "+(1)-(206)-9052100",
                    "categorySet": [
                        {
                            "id": 7376009
                        }
                    ],
                    "url": "www.spaceneedle.com",
                    "categories": [
                        "important tourist attraction",
                        "tower"
                    ],
                    "classifications": [
                        {
                            "code": "IMPORTANT_TOURIST_ATTRACTION",
                            "names": [
                                {
                                    "nameLocale": "en-US",
                                    "name": "important tourist attraction"
                                },
                                {
                                    "nameLocale": "en-US",
                                    "name": "tower"
                                }
                            ]
                        }
                    ]
                },
                "address": {
                    "streetNumber": "400",
                    "streetName": "Broad St",
                    "municipalitySubdivision": "South Lake Union, Seattle, Lower Queen Anne",
                    "municipality": "Seattle",
                    "countrySecondarySubdivision": "King",
                    "countryTertiarySubdivision": "Seattle",
                    "countrySubdivision": "WA",
                    "postalCode": "98109",
                    "countryCode": "US",
                    "country": "United States",
                    "countryCodeISO3": "USA",
                    "freeformAddress": "400 Broad St, Seattle, WA 98109",
                    "localName": "Seattle",
                    "countrySubdivisionName": "Washington"
                },
                "position": {
                    "lat": 47.62039,
                    "lon": -122.34928
                },
                "viewport": {
                    "topLeftPoint": {
                        "lat": 47.62129,
                        "lon": -122.35061
                    },
                    "btmRightPoint": {
                        "lat": 47.61949,
                        "lon": -122.34795
                    }
                },
                "entryPoints": [
                    {
                        "type": "main",
                        "position": {
                            "lat": 47.61982,
                            "lon": -122.34886
                        }
                    }
                ]
            },
            ...,
            ...,
            ...
        ]
    }
    ```

### <a name="request-route"></a>Solicitud de ruta

Para realizar una solicitud de ruta, siga estos pasos:

1. En Postman, haga clic en **New Request** (Nueva solicitud)  | **GET request** (Solicitud GET) y asígnele el nombre **Get Route info** (Obtener información de ruta).

2. En la pestaña Builder (Generador), seleccione el método HTTP **GET**, escriba la siguiente dirección URL de solicitud para el punto de conexión de API y haga clic en **Send** (Enviar).

    Solicitaremos las rutas de transporte público en autobús; para ello, especificaremos los parámetros `modeType` y `transitType`. La solicitud de dirección URL contiene las ubicaciones recuperadas en las secciones anteriores. En `originType`, ahora tenemos **stopId**. Y para `destionationType`, tenemos la **position**.

    Consulte la [lista de parámetros de URI](https://aka.ms/AzureMapsMobilityTransitRoute#uri-parameters) que puede usar en la solicitud a [Get Transit Routes API](https://aka.ms/AzureMapsMobilityTransitRoute).
  
    ```HTTP
    https://atlas.microsoft.com/mobility/transit/route/json?subscription-key={subscription-key}&api-version=1.0&originType=stopId&origin=522---2060603&destionationType=position&destination=47.62039,-122.34928&modeType=publicTransit&transitType=bus
    ```

3. Si la solicitud es correcta, la estructura de la respuesta se parecerá a la siguiente:

    ```JSON
    {
        "results": [
            {
                "itineraryId": "cb6b6b6f-5cda-451e-b68d-2e97971dd60c---20190906BBBEC4D2219A436A9D794224978C9BBF:0---522",
                "departureTime": "2019-09-07T01:01:50Z",
                "arrivalTime": "2019-09-07T02:16:33Z",
                "travelTimeInSeconds": 4483,
                "numberOfLegs": 8,
                "legs": [
                    {
                        "legType": "Wait",
                        "legStartTime": "2019-09-07T01:01:50Z",
                        "legEndTime": "2019-09-07T01:01:50Z",
                        "caption": "249"
                    },
                    {
                        "legType": "Bus",
                        "legStartTime": "2019-09-07T01:01:50Z",
                        "legEndTime": "2019-09-07T01:26:00Z",
                        "caption": "249",
                        "lengthInMeters": 9139
                    },
                    {
                        "legType": "Wait",
                        "legStartTime": "2019-09-07T01:26:00Z",
                        "legEndTime": "2019-09-07T01:28:00Z",
                        "caption": "255"
                    },
                    {
                        "legType": "Bus",
                        "legStartTime": "2019-09-07T01:28:00Z",
                        "legEndTime": "2019-09-07T01:57:21Z",
                        "caption": "255",
                        "lengthInMeters": 13136
                    },
                    {
                        "legType": "Walk",
                        "legStartTime": "2019-09-07T01:57:22Z",
                        "legEndTime": "2019-09-07T02:01:27Z",
                        "caption": "Denny Way",
                        "lengthInMeters": 308
                    },
                    {
                        "legType": "Wait",
                        "legStartTime": "2019-09-07T02:01:27Z",
                        "legEndTime": "2019-09-07T02:06:33Z",
                        "caption": "8"
                    },
                    {
                        "legType": "Bus",
                        "legStartTime": "2019-09-07T02:06:33Z",
                        "legEndTime": "2019-09-07T02:12:41Z",
                        "caption": "8",
                        "lengthInMeters": 1060
                    },
                    {
                        "legType": "Walk",
                        "legStartTime": "2019-09-07T02:12:42Z",
                        "legEndTime": "2019-09-07T02:16:33Z",
                        "lengthInMeters": 251
                    }
                ],
                "itineraryFare": {
                    "price": {
                        "amount": 550,
                        "currencyCode": "USD"
                    },
                    "tickets": [
                        {
                            "amount": 275,
                            "currencyCode": "USD"
                        },
                        {
                            "amount": 275,
                            "currencyCode": "USD"
                        }
                    ]
                }
            },
            ...,
            {
                "itineraryId": "cb6b6b6f-5cda-451e-b68d-2e97971dd60c---20190906BBBEC4D2219A436A9D794224978C9BBF:2---522",
                "departureTime": "2019-09-07T00:49:32Z",
                "arrivalTime": "2019-09-07T02:20:06Z",
                "travelTimeInSeconds": 5434,
                "numberOfLegs": 10,
                "legs": [
                    {
                        "legType": "Wait",
                        "legStartTime": "2019-09-07T00:49:32Z",
                        "legEndTime": "2019-09-07T00:49:32Z",
                        "caption": "226"
                    },
                    {
                        "legType": "Bus",
                        "legStartTime": "2019-09-07T00:49:32Z",
                        "legEndTime": "2019-09-07T01:15:00Z",
                        "caption": "226",
                        "lengthInMeters": 6792
                    },
                    {
                        "legType": "Wait",
                        "legStartTime": "2019-09-07T01:15:00Z",
                        "legEndTime": "2019-09-07T01:20:00Z",
                        "caption": "241"
                    },
                    {
                        "legType": "Bus",
                        "legStartTime": "2019-09-07T01:20:00Z",
                        "legEndTime": "2019-09-07T01:28:00Z",
                        "caption": "241",
                        "lengthInMeters": 3397
                    },
                    {
                        "legType": "Wait",
                        "legStartTime": "2019-09-07T01:28:00Z",
                        "legEndTime": "2019-09-07T01:33:00Z",
                        "caption": "550"
                    },
                    {
                        "legType": "Bus",
                        "legStartTime": "2019-09-07T01:33:00Z",
                        "legEndTime": "2019-09-07T01:58:00Z",
                        "caption": "550",
                        "lengthInMeters": 12899
                    },
                    {
                        "legType": "Walk",
                        "legStartTime": "2019-09-07T01:58:01Z",
                        "legEndTime": "2019-09-07T01:59:21Z",
                        "caption": "4th Avenue South",
                        "lengthInMeters": 99
                    },
                    {
                        "legType": "Wait",
                        "legStartTime": "2019-09-07T01:59:21Z",
                        "legEndTime": "2019-09-07T02:01:00Z",
                        "caption": "33"
                    },
                    {
                        "legType": "Bus",
                        "legStartTime": "2019-09-07T02:01:00Z",
                        "legEndTime": "2019-09-07T02:13:29Z",
                        "caption": "33,24",
                        "lengthInMeters": 2447
                    },
                    {
                        "legType": "Walk",
                        "legStartTime": "2019-09-07T02:13:30Z",
                        "legEndTime": "2019-09-07T02:20:06Z",
                        "lengthInMeters": 457
                    }
                ],
                "itineraryFare": {
                    "price": {
                        "amount": 550,
                        "currencyCode": "USD"
                    },
                    "tickets": [
                        {
                            "amount": 275,
                            "currencyCode": "USD"
                        },
                        {
                            "amount": 275,
                            "currencyCode": "USD"
                        }
                    ]
                }
            }
        ]
    }
    ```

4. Si observa detenidamente, hay varias rutas de **Bus** en la respuesta. Cada ruta tiene un **identificador de itinerario** único, un resumen que describe cada uno de los tramos de la ruta y un valor `itineraryFare` que proporciona el precio total y por unidades de los billetes de bus. Un trayecto de ruta es la parte de la ruta entre dos puntos de referencia de parada. A continuación, solicitaremos detalles de la ruta más rápida mediante el uso de `itineraryId` en la respuesta.

## <a name="request-fastest-route-itinerary"></a>Solicitud del itinerario más rápido

El servicio [Get Transit Itinerary](https://aka.ms/AzureMapsMobilityTransitItinerary) de Azure Maps permite solicitar datos de una ruta concreta con el uso del **ID de itinerario** que devuelve [Get Transit Routes API](https://aka.ms/AzureMapsMobilityTransitRoute). Para realizar una solicitud, siga estos pasos:

1. En Postman, haga clic en **New Request** (Nueva solicitud)  | **GET request** (Solicitud GET) y asígnele el nombre **Get Transit info** (Obtener información de transporte).

2. En la pestaña del generador, seleccione el método HTTP **GET**. Escriba la siguiente dirección URL de solicitud para el punto de conexión de la API y haga clic en **Enviar**.

    Estableceremos el parámetro `detailType` en **geometry** para que la respuesta contenga la información de las paradas de transporte público y navegación detallada para las opciones de ruta a pie y en bicicleta.

    ```HTTP
    https://atlas.microsoft.com/mobility/transit/itinerary/json?api-version=1.0&subscription-key={subscription-key}&query={itineraryId}&detailType=geometry
    ```

3. Si la solicitud es correcta, la estructura de la respuesta se parecerá a la siguiente.  Si observa la respuesta JSON, observará que cada tramo de bus contiene un elemento `legfare`. El elemento `legfare` contiene el costo, en céntimos, de cada ruta de bus que se adquiere por separado. Al final de la respuesta, verá un elemento `itineraryFare` que contiene el costo, en céntimos, de toda la ruta. En este ejemplo, hay cuatro rutas de bus por `$2.75` cada una. Sin embargo, si compra un solo billete para toda la ruta, el costo es `$5.50`.

    ```JSON
   {
        "departureTime": "2020-07-22T19:54:47Z",
        "arrivalTime": "2020-07-22T21:12:21Z",
        "legs": [
            {
                "legType": "Wait",
                "legStartTime": "2020-07-22T19:54:47Z",
                "legEndTime": "2020-07-22T19:54:47Z",
                "lineGroup": {
                    "lineGroupId": "522---666063",
                    "agencyId": "522---5872",
                    "agencyName": "Metro Transit",
                    "lineNumber": "226",
                    "caption1": "Eastgate P&R-Crossroads-Overlake-Bellevue TC",
                    "caption2": "226 Eastgate P&R-Crossroads-Overlake-Bellevue TC",
                    "color": "347E5D",
                    "transitType": "Bus"
                },
                "line": {
                    "lineId": "522---2756599",
                    "lineGroupId": "522---666063",
                    "direction": "forward",
                    "agencyId": "522---5872",
                    "lineNumber": "226",
                    "lineDestination": "Bellevue Transit Center Crossroads"
                },
                "stops": [
                    {
                        "stopId": "522---2060603",
                        "stopKey": "71300",
                        "stopName": "NE 24th St & 162nd Ave NE",
                        "stopCode": "71300",
                        "position": {
                            "latitude": 47.631504,
                            "longitude": -122.125275
                        },
                        "mainTransitType": "Bus",
                        "mainAgencyId": "522---5872",
                        "mainAgencyName": "Metro Transit"
                    },
                    {
                        "stopId": "522---2062263",
                        "stopKey": "85630",
                        "stopName": "Bellevue Tc",
                        "stopCode": "85630",
                        "position": {
                            "latitude": 47.615591,
                            "longitude": -122.196491
                        },
                        "mainTransitType": "Bus",
                        "mainAgencyId": "522---5872",
                        "mainAgencyName": "Metro Transit"
                    }
                ],
                "waitOnVehicle": false
            },
            {
                "legType": "Bus",
                "legStartTime": "2020-07-22T19:54:47Z",
                "legEndTime": "2020-07-22T20:15:00Z",
                "lineGroup": {
                    "lineGroupId": "522---666063",
                    "agencyId": "522---5872",
                    "agencyName": "Metro Transit",
                    "lineNumber": "226",
                    "caption1": "Eastgate P&R-Crossroads-Overlake-Bellevue TC",
                    "caption2": "226 Eastgate P&R-Crossroads-Overlake-Bellevue TC",
                    "color": "347E5D",
                    "transitType": "Bus"
                },
                "line": {
                    "lineId": "522---2756599",
                    "lineGroupId": "522---666063",
                    "direction": "forward",
                    "agencyId": "522---5872",
                    "lineNumber": "226",
                    "lineDestination": "Bellevue Transit Center Crossroads"
                },
                "stops": [
                   ...
                ],
                "geometry": {
                    "type": "LineString",
                    "coordinates": [
                       ...
                    ]
                },
                "legFare": {
                    "fares": [
                        {
                            "price": {
                                "amount": 275,
                                "currencyCode": "USD"
                            },
                            "usage": "pay"
                        }
                    ]
                }
            },
            ...,
            ...,
            {
                "legType": "Bus",
                "legStartTime": "2020-07-22T20:20:00Z",
                "legEndTime": "2020-07-22T20:28:00Z",
                "lineGroup": {
                    "lineGroupId": "522---666071",
                    "agencyId": "522---5872",
                    "agencyName": "Metro Transit",
                    "lineNumber": "241",
                    "caption1": "Eastgate P&R - Bellevue Transit Center",
                    "caption2": "241 Eastgate P&R - Bellevue Transit Center",
                    "color": "347E5D",
                    "transitType": "Bus"
                },
                "line": {
                    "lineId": "522---2756619",
                    "lineGroupId": "522---666071",
                    "direction": "backward",
                    "agencyId": "522---5872",
                    "lineNumber": "241",
                    "lineDestination": "Eastgate P&R Factoria"
                },
                "stops": [
                ...
                ],
                "geometry": {
                    "type": "LineString",
                    "coordinates": [
                   ...
                    ]
                },
                "legFare": {
                    "fares": [
                        {
                            "price": {
                                "amount": 275,
                                "currencyCode": "USD"
                            },
                            "usage": "transfer"
                        }
                    ]
                }
            },
            ...,
            {
                "legType": "Bus",
                "legStartTime": "2020-07-22T20:31:00Z",
                "legEndTime": "2020-07-22T20:54:13Z",
                "lineGroup": {
                    "lineGroupId": "522---312636",
                    "agencyId": "522---854535",
                    "agencyName": "Sound Transit",
                    "lineNumber": "550",
                    "caption1": "Bellevue - Seattle",
                    "caption2": "550 Bellevue - Seattle",
                    "color": "00008B",
                    "transitType": "Bus"
                },
                "line": {
                    "lineId": "522---962201",
                    "lineGroupId": "522---312636",
                    "direction": "backward",
                    "agencyId": "522---854535",
                    "lineNumber": "550",
                    "lineDestination": "Seattle"
                },
                "stops": [
                   ...
                ],
                "geometry": {
                    "type": "LineString",
                    "coordinates": [
                 ...
                    ]
                },
                "legFare": {
                    "fares": [
                        {
                            "price": {
                                "amount": 275,
                                "currencyCode": "USD"
                            },
                            "usage": "pay"
                        }
                    ]
                }
            },
            ...,
            ...,
            {
                "legType": "Bus",
                "legStartTime": "2020-07-22T20:57:00Z",
                "legEndTime": "2020-07-22T21:06:00Z",
                "lineGroup": {
                    "lineGroupId": "522---480518",
                    "agencyId": "522---5872",
                    "agencyName": "Metro Transit",
                    "lineNumber": "13",
                    "caption1": "Seattle Pacific - Downtown Seattle",
                    "caption2": "13 Seattle Pacific - Downtown Seattle",
                    "color": "347E5D",
                    "transitType": "Bus"
                },
                "line": {
                    "lineId": "522---1744932",
                    "lineGroupId": "522---480518",
                    "direction": "forward",
                    "agencyId": "522---5872",
                    "lineNumber": "13",
                    "lineDestination": "Seattle Pacific University Seattle Center W"
                },
                "stops": [
                   ...
                ],
                "geometry": {
                    "type": "LineString",
                    "coordinates": [
                      ...
                    ]
                },
                "legFare": {
                    "fares": [
                        {
                            "price": {
                                "amount": 275,
                                "currencyCode": "USD"
                            },
                            "usage": "transfer"
                        }
                    ]
                }
            },
            ...,
        ],
        "itineraryFare": {
            "price": {
                "amount": 550,
                "currencyCode": "USD"
            },
            "tickets": [
                {
                    "amount": 275,
                    "currencyCode": "USD"
                },
                {
                    "amount": 275,
                    "currencyCode": "USD"
                }
            ]
        }
    }

    ```

## <a name="next-steps"></a>Pasos siguientes

Aprenda a solicitar datos en tiempo real mediante el servicio Mobility:

> [!div class="nextstepaction"]
> [Cómo solicitar datos en tiempo real](how-to-request-real-time-data.md)

Explorar la documentación de la API del servicio Mobility de Azure Maps

> [!div class="nextstepaction"]
> [Documentación del servicio Mobility](https://aka.ms/AzureMapsMobilityService)
