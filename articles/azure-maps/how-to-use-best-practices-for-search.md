---
title: Procedimientos recomendados del servicio Search de Azure Maps | Microsoft Azure Maps
description: Obtenga información sobre cómo poner en marcha algunos procedimientos recomendados relativos al servicio Search de Microsoft Azure Maps.
author: anastasia-ms
ms.author: v-stharr
ms.date: 01/23/2020
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: philmea
ms.openlocfilehash: 5e98763a3a1c8273cdeec5e945dd324ae43e773f
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87064265"
---
# <a name="best-practices-for-azure-maps-search-service"></a>Procedimientos recomendados del servicio Search de Azure Maps

El [servicio Search](https://docs.microsoft.com/rest/api/maps/search) incluye API que ofrecen diversas funcionalidades para ayudar a que los desarrolladores busquen direcciones, lugares o listados de empresas por nombre y categoría, y otra información geográfica. Por ejemplo,[Fuzzy Search API](https://docs.microsoft.com/rest/api/maps/search/getsearchfuzzy) permite a los usuarios buscar una dirección o un punto de interés (POI).

En este artículo se explica cómo aplicar buenas prácticas cuando se llama a datos desde el servicio Search de Azure Maps. Aprenderá a:

* Crear consultas que devuelvan coincidencias relevantes
* Limitar los resultados de la búsqueda
* Distinguir entre los tipos de resultados
* Leer la estructura de respuesta a la búsqueda de direcciones

## <a name="prerequisites"></a>Requisitos previos

Para realizar llamadas a las API de servicio de Azure Maps, necesita una cuenta de Azure Maps y una clave. Para más información, consulte [Creación de una cuenta](quick-demo-map-app.md#create-an-azure-maps-account) y [Obtención de una clave principal](quick-demo-map-app.md#get-the-primary-key-for-your-account). 

Para más información sobre la autenticación en Azure Maps, consulte [Administración de la autenticación en Azure Maps](./how-to-manage-authentication.md).

> [!TIP]
> Para consultar el servicio Search, use la [aplicación Postman](https://www.getpostman.com/apps) para generar llamadas API REST. Puede usar cualquier entorno de desarrollo de API que prefiera.

## <a name="best-practices-to-geocode-addresses"></a>Procedimientos recomendados para las direcciones de geocodificación

Al buscar una dirección completa o parcial mediante el servicio Search de Azure Maps, la API lee las palabras clave de la consulta de búsqueda. A continuación, devuelve las coordenadas de longitud y latitud de la dirección. Este proceso se denomina *geocodificación*. 

La capacidad de geocodificación de un país o región depende de la disponibilidad de datos de carreteras y de la precisión del servicio de geocodificación. Para más información sobre las funcionalidades de geocodificación de Azure Maps por país o región, consulte [Cobertura de geocodificación](https://docs.microsoft.com/azure/azure-maps/geocoding-coverage).

### <a name="limit-search-results"></a>Limitar los resultados de la búsqueda

 Las API de búsqueda de Azure Maps pueden ayudar a limitar los resultados de la búsqueda. Puede limitar los resultados para mostrar los datos pertinentes a los usuarios.

> [!NOTE]
> Las API de búsqueda admiten más parámetros que los que se describen en este artículo.

#### <a name="geobiased-search-results"></a>Resultados de búsqueda con sesgo geográfico

Para sesgar geográficamente los resultados al área pertinente para el usuario, agregue siempre tantos detalles como sea posible. Puede que quiera especificar algunos tipos de entrada para restringir los resultados de la búsqueda:

* Establezca el parámetro `countrySet`. Puede establecerlo en `US,FR`, por ejemplo. De forma predeterminada, la API busca en todo el mundo, por lo que podría devolver resultados innecesarios. Si la consulta no tiene ningún parámetro `countrySet`, es posible que la búsqueda devuelva resultados inexactos. Por ejemplo, la búsqueda de una ciudad llamada *Bellevue* devuelve resultados de Estados Unidos y Francia porque ambos países o regiones tienen una ciudad llamada *Bellevue*.

* Puede usar los parámetros `btmRight` y `topleft` para establecer el rectángulo delimitador. Estos parámetros restringen la búsqueda a un área específica del mapa.

* Para influir en el área de relevancia de los resultados, defina los parámetros de coordenadas `lat` y `lon`. Use el parámetro `radius` para establecer el radio del área de búsqueda.


#### <a name="fuzzy-search-parameters"></a>Parámetros de búsqueda aproximada

Se recomienda usar la [API de búsqueda aproximada](https://docs.microsoft.com/rest/api/maps/search/getsearchfuzzy) de Azure Maps cuando no conozca los datos de entrada de la consulta de búsqueda de un usuario. Por ejemplo, la entrada del usuario podría ser una dirección o el tipo de punto de interés, por ejemplo, *centro comercial*. La API combina la búsqueda de puntos de interés y geocodificación en una *búsqueda de una sola línea* canónica: 

* Los parámetros `minFuzzyLevel` y `maxFuzzyLevel` ayudan a devolver coincidencias relevantes aunque los parámetros de consulta no coincidan exactamente con la información que el usuario desea. Para obtener el máximo rendimiento y reducir los resultados inusuales, establezca las consultas de búsqueda en los valores predeterminados `minFuzzyLevel=1` y `maxFuzzyLevel=2`. 

    Por ejemplo, cuando el parámetro `maxFuzzyLevel` está establecido en 2, el término de búsqueda *restrant* coincide con *restaurante*. Puede invalidar los niveles predeterminados de búsqueda aproximada cuando sea necesario. 

* Use el parámetro `idxSet` para dar prioridad al conjunto exacto de tipos de resultados. Para dar prioridad a un conjunto exacto de resultados, puede enviar una lista de índices separada por comas. En la lista, el orden de los elementos no importa. Azure Maps admite los índices siguientes:

* `Addr` - **Intervalos de direcciones**: puntos de dirección que se interpolan desde el principio y el final de la calle. Esos puntos se representan como intervalos de direcciones.
* `Geo` - **Geografías**: divisiones administrativas del territorio. Una geografía puede ser un país o región, un estado o una ciudad, por ejemplo.
* `PAD` - **Direcciones de punto**: direcciones que incluyen el nombre y el número de la calle. Las direcciones de punto se pueden encontrar en un índice. Un ejemplo es *Soquel Dr 2501*. Una dirección de punto proporciona el nivel más alto de precisión disponible para las direcciones.  
* `POI` - **Puntos de interés**: puntos en un mapa que se considera que merece la pena prestar atención o que pueden ser interesantes. La [API de búsqueda de direcciones](https://docs.microsoft.com/rest/api/maps/search/getsearchaddress) no devuelve puntos de interés.  
* `Str` - **Calles**: calles en el mapa.
* `XStr` - **Cruces o intersecciones**: uniones o lugares donde dos calles forman una intersección.


#### <a name="usage-examples"></a>Ejemplos de uso

* `idxSet=POI`: buscar solo puntos de interés. 

* `idxSet=PAD,Addr`: buscar solo direcciones. `PAD` indica la dirección de punto y `Addr` indica el intervalo de direcciones.

### <a name="reverse-geocode-and-filter-for-a-geography-entity-type"></a>Geocodificación inversa y filtro por tipo de entidad geográfica

Cuando se realiza una búsqueda de geocodificación inversa en la [API de búsqueda de dirección inversa](https://docs.microsoft.com/rest/api/maps/search/getsearchaddressreverse), el servicio puede devolver los polígonos de las áreas administrativas. Por ejemplo, es posible que quiera obtener el área poligonal de una ciudad. Para restringir la búsqueda a unos tipos de entidad geográfica específicos, incluya el parámetro `entityType` en las solicitudes. 

La respuesta resultante contiene el identificador de geografía y el tipo de entidad que se buscó. Si se proporciona más de una entidad, el punto de conexión devuelve la *entidad más pequeña disponible*. Puede usar el identificador de geometría devuelto para obtener la geometría de la geografía mediante el [servicio de búsqueda de polígono](https://docs.microsoft.com/rest/api/maps/search/getsearchpolygon).

#### <a name="sample-request"></a>Solicitud de ejemplo

```HTTP
https://atlas.microsoft.com/search/address/reverse/json?api-version=1.0&subscription-key={subscription-key}&query=47.6394532,-122.1304551&language=en-US&entityType=Municipality
```

#### <a name="response"></a>Response

```JSON
{
    "summary": {
        "queryTime": 14,
        "numResults": 1
    },
    "addresses": [
        {
            "address": {
                "routeNumbers": [],
                "countryCode": "US",
                "countrySubdivision": "WA",
                "countrySecondarySubdivision": "King",
                "countryTertiarySubdivision": "Seattle East",
                "municipality": "Redmond",
                "country": "United States",
                "countryCodeISO3": "USA",
                "countrySubdivisionName": "Washington"
            },
            "position": "47.639454,-122.130455",
            "dataSources": {
                "geometry": {
                    "id": "00005557-4100-3c00-0000-0000596ae571"
                }
            },
            "entityType": "Municipality"
        }
    ]
}
```

### <a name="set-the-results-language"></a>Establecimiento del idioma de los resultados

Use el parámetro `language` para establecer el idioma de los resultados de búsqueda devueltos. Si la solicitud no establece el idioma, el servicio Search usa el idioma más común del país o la región de forma predeterminada. Cuando no hay datos disponibles en el idioma especificado, se usa el idioma predeterminado. 

Para más información, consulte [Idiomas admitidos por Azure Maps](https://docs.microsoft.com/azure/azure-maps/supported-languages).


### <a name="use-predictive-mode-automatic-suggestions"></a>Uso del modo predictivo (sugerencias automáticas)

Para buscar más coincidencias en consultas parciales, establezca el parámetro `typeahead` en `true`. Esta consulta se interpreta como una entrada parcial y la búsqueda inicia el modo predictivo. Si no establece el parámetro `typeahead` en `true`, el servicio asume que se ha pasado toda la información pertinente.

En la consulta del ejemplo siguiente, se consulta *Microso* al servicio de búsqueda de direcciones. Aquí, el parámetro `typeahead` está establecido en `true`. La respuesta muestra que el servicio de búsqueda interpretó la consulta como una consulta parcial. La respuesta contiene los resultados de una consulta sugerida automáticamente.

#### <a name="sample-query"></a>Consulta de ejemplo

```HTTP
https://atlas.microsoft.com/search/address/json?subscription-key={subscription-key}&api-version=1.0&typeahead=true&countrySet=US&lat=47.6370891183&lon=-122.123736172&query=Microsoft
```

#### <a name="response"></a>Response

```JSON
{
    "summary": {
        "query": "microsoft",
        "queryType": "NON_NEAR",
        "queryTime": 18,
        "numResults": 7,
        "offset": 0,
        "totalResults": 7,
        "fuzzyLevel": 1,
        "geoBias": {
            "lat": 47.6370891183,
            "lon": -122.123736172
        }
    },
    "results": [
        {
            "type": "Street",
            "id": "US/STR/p0/9438784",
            "score": 2.594099998474121,
            "dist": 314.0590106663596,
            "address": {
                "streetName": "Microsoft Way",
                "municipalitySubdivision": "Redmond",
                "municipality": "Redmond",
            },
            "position": {
                "lat": 47.63988,
                "lon": -122.12438
            },
            "viewport": {
                "topLeftPoint": {
                    "lat": 47.64223,
                    "lon": -122.1256,
                    "valid": true
                },
                "btmRightPoint": {
                    "lat": 47.63748,
                    "lon": -122.12309,
                    "valid": true
                }
            }
        },
        {
            "type": "Street",
            "id": "US/STR/p0/1756074",
            "score": 2.592679977416992,
            "dist": 876.0272035824189,
            "address": {
                "streetName": "Microsoft Road",
                "municipalitySubdivision": "Redmond",
                "municipality": "Redmond",
                "countrySecondarySubdivision": "King",
                "countryTertiarySubdivision": "Seattle East",
                "countrySubdivision": "WA",
                "countrySubdivisionName": "Washington",
                "postalCode": "98052",
                "countryCode": "US",
                "country": "United States",
                "countryCodeISO3": "USA",
                "freeformAddress": "Microsoft Road, Redmond, WA 98052"
            },
            "position": {
                "lat": 47.64032,
                "lon": -122.1344
            },
            "viewport": {
                "topLeftPoint": {
                    "lat": 47.64253,
                    "lon": -122.13535,
                    "valid": true
                },
                "btmRightPoint": {
                    "lat": 47.63816,
                    "lon": -122.13305,
                    "valid": true
                }
            }
        },
        {
            "type": "Street",
            "id": "US/STR/p0/1470668",
            "score": 2.5290400981903076,
            "dist": 2735.4883918101486,
            "address": {
                "streetName": "Microsoft West Campus Road",
                "municipalitySubdivision": "Redmond",
                "municipality": "Bellevue",
                "countrySecondarySubdivision": "King",
                "countryTertiarySubdivision": "Seattle East",
                "countrySubdivision": "WA",
                "countrySubdivisionName": "Washington",
                "postalCode": "98007",
                "countryCode": "US",
                "country": "United States",
                "countryCodeISO3": "USA",
                "freeformAddress": "Microsoft West Campus Road, Bellevue, WA 98007"
            },
            "position": {
                "lat": 47.65784,
                "lon": -122.14335
            },
            "viewport": {
                "topLeftPoint": {
                    "lat": 47.65785,
                    "lon": -122.14335,
                    "valid": true
                },
                "btmRightPoint": {
                    "lat": 47.65784,
                    "lon": -122.14325,
                    "valid": true
                }
            }
        },
        {
            "type": "Street",
            "id": "US/STR/p0/12812615",
            "score": 2.527509927749634,
            "dist": 2870.9579016916873,
            "address": {
                "streetName": "Microsoft West Campus Road",
                "municipalitySubdivision": "Redmond",
                "municipality": "Redmond",
                "countrySecondarySubdivision": "King",
                "countryTertiarySubdivision": "Seattle East",
                "countrySubdivision": "WA",
                "countrySubdivisionName": "Washington",
                "postalCode": "98052",
                "countryCode": "US",
                "country": "United States",
                "countryCodeISO3": "USA",
                "freeformAddress": "Microsoft West Campus Road, Redmond, WA 98052"
            },
            "position": {
                "lat": 47.66034,
                "lon": -122.1404
            },
            "viewport": {
                "topLeftPoint": {
                    "lat": 47.66039,
                    "lon": -122.14325,
                    "valid": true
                },
                "btmRightPoint": {
                    "lat": 47.65778,
                    "lon": -122.13749,
                    "valid": true
                }
            }
        },
        {
            "type": "Street",
            "id": "US/STR/p0/197588",
            "score": 2.4630401134490967,
            "dist": 878.1404663812472,
            "address": {
                "streetName": "157th Avenue Northeast",
                "municipalitySubdivision": "Redmond",
                "municipality": "Redmond",
                "countrySecondarySubdivision": "King",
                "countryTertiarySubdivision": "Seattle East",
                "countrySubdivision": "WA",
                "countrySubdivisionName": "Washington",
                "postalCode": "98052",
                "extendedPostalCode": "980525344, 980525398, 980525399",
                "countryCode": "US",
                "country": "United States",
                "countryCodeISO3": "USA",
                "freeformAddress": "157th Avenue Northeast, Redmond, WA 98052"
            },
            "position": {
                "lat": 47.64351,
                "lon": -122.13056
            },
            "viewport": {
                "topLeftPoint": {
                    "lat": 47.64473,
                    "lon": -122.13058,
                    "valid": true
                },
                "btmRightPoint": {
                    "lat": 47.6425,
                    "lon": -122.13016,
                    "valid": true
                }
            }
        },
        {
            "type": "Street",
            "id": "US/STR/p0/3033991",
            "score": 2.0754499435424805,
            "dist": 3655467.8844475765,
            "address": {
                "streetName": "Microsoft Way",
                "municipalitySubdivision": "Yorkmount, Charlotte",
            },
            "position": {
                "lat": 35.14267,
                "lon": -80.91824
            },
            "viewport": {
                "topLeftPoint": {
                    "lat": 35.14287,
                    "lon": -80.91839,
                    "valid": true
                },
                "btmRightPoint": {
                    "lat": 35.14267,
                    "lon": -80.91814,
                    "valid": true
                }
            }
        },
        {
            "type": "Street",
            "id": "US/STR/p0/8395877",
            "score": 2.0754499435424805,
            "dist": 3655437.0037482483,
            "address": {
                "streetName": "Microsoft Way",
                "municipalitySubdivision": "Charlotte",
                "municipality": "Charlotte",
                "countrySecondarySubdivision": "Mecklenburg",
                "countryTertiarySubdivision": "Township 1 Charlotte",
                "countrySubdivision": "NC",
                "countrySubdivisionName": "North Carolina",
                "postalCode": "28273",
                "extendedPostalCode": "282738105, 282738106, 282738108, 2827382, 282738200",
                "countryCode": "US",
                "country": "United States",
                "countryCodeISO3": "USA",
                "freeformAddress": "Microsoft Way, Charlotte, NC 28273"
            },
            "position": {
                "lat": 35.14134,
                "lon": -80.9198
            },
            "viewport": {
                "topLeftPoint": {
                    "lat": 35.14274,
                    "lon": -80.92159,
                    "valid": true
                },
                "btmRightPoint": {
                    "lat": 35.14002,
                    "lon": -80.91824,
                    "valid": true
                }
            }
        }
    ]
}
```


### <a name="encode-a-uri-to-handle-special-characters"></a>Codificación de un URI para controlar los caracteres especiales 

Para buscar direcciones compuestas, debe codificar el URI para controlar los caracteres especiales en la dirección. Considere este ejemplo de dirección: *1st Avenue & Union Street, Seattle*. Aquí, codifique el carácter de y comercial (`&`) antes de enviar la solicitud. 

Se recomienda codificar los datos de caracteres de un URI. En un URI, codifique todos los caracteres mediante un signo de porcentaje (`%`) y un valor hexadecimal de dos caracteres que se corresponda con el código UTF-8 de los caracteres.

#### <a name="usage-examples"></a>Ejemplos de uso

Comience con esta dirección:

```
query=1st Avenue & E 111th St, New York
```

Codifique la dirección:

```
query=1st%20Avenue%20%26%20E%20111th%20St%2C%20New%20York
```

Puede usar los métodos siguientes.

JavaScript o TypeScript:
```Javascript
encodeURIComponent(query)
```

C# o Visual Basic:
```csharp
Uri.EscapeDataString(query)
```

Java:
```Java
URLEncoder.encode(query, "UTF-8") 
```

Python:
```Python
import urllib.parse 
urllib.parse.quote(query)
```

C++:
```C++
#include <curl/curl.h>
curl_easy_escape(query)
```

PHP:
```PHP
urlencode(query)
```

Ruby:
```Ruby
CGI::escape(query) 
```

Swift:
```Swift
query.stringByAddingPercentEncodingWithAllowedCharacters(.URLHostAllowedCharacterSet()) 
```

Go:
```Go
import ("net/url") 
url.QueryEscape(query)
```


## <a name="best-practices-for-poi-searching"></a>Procedimientos recomendados para la búsqueda de puntos de interés

En una búsqueda de puntos de interés, puede solicitar los resultados por nombre. Por ejemplo, puede buscar una empresa por su nombre. 

Es muy recomendable que use el parámetro `countrySet` para especificar los países o regiones en los que la aplicación necesita cobertura. El comportamiento predeterminado es buscar en todo el mundo. Esta búsqueda tan amplia puede devolver resultados innecesarios y la búsqueda puede tardar mucho tiempo.

### <a name="brand-search"></a>Búsqueda de marcas

Para mejorar la pertinencia de los resultados y la información de la respuesta, la respuesta de una búsqueda de puntos de interés incluye información de marca. Puede usar esta información para analizar la respuesta.

En una solicitud, puede enviar una lista de nombres de marcas separada por comas. Use la lista para restringir los resultados a marcas específicas con el parámetro `brandSet`. En la lista, el orden de los elementos no importa. Cuando se proporcionan varias listas de marcas, los resultados que se devuelven deben pertenecer al menos a una de las listas.

Para explorar la búsqueda de marcas, vamos a crear una solicitud de [búsqueda de categorías de puntos de interés](https://docs.microsoft.com/rest/api/maps/search/getsearchpoicategory). En el ejemplo siguiente, buscamos estaciones de servicio próximas al campus de Microsoft en Redmond, Washington. La respuesta muestra la información de marca de cada punto de interés devuelto.

#### <a name="sample-query"></a>Consulta de ejemplo

```HTTP
https://atlas.microsoft.com/search/poi/json?subscription-key={subscription-key}&api-version=1.0&query=gas%20station&limit=3&lat=47.6413362&lon=-122.1327968
```

#### <a name="response"></a>Response

```JSON
{
    "summary": {
        "query": "gas station",
        "queryType": "NON_NEAR",
        "queryTime": 276,
        "numResults": 3,
        "offset": 0,
        "totalResults": 762680,
        "fuzzyLevel": 1,
        "geoBias": {
            "lat": 47.6413362,
            "lon": -122.1327968
        }
    },
    "results": [
        {
            "type": "POI",
            "id": "US/POI/p0/8831765",
            "score": 5.6631999015808105,
            "dist": 1037.0280221303253,
            "info": "search:ta:840531000004190-US",
            "poi": {
                "name": "Chevron",
                "phone": "+(1)-(425)-6532200",
                "brands": [
                    {
                        "name": "Chevron"
                    }
                ],
                "categorySet": [
                    {
                        "id": 7311
                    }
                ],
                "url": "www.chevron.com",
                "categories": [
                    "petrol station"
                ],
                "classifications": [
                    {
                        "code": "PETROL_STATION",
                        "names": [
                            {
                                "nameLocale": "en-US",
                                "name": "petrol station"
                            }
                        ]
                    }
                ]
            },
            "address": {
                "streetNumber": "2444",
                "streetName": "Bel Red Rd",
                "municipalitySubdivision": "Northeast Bellevue, Bellevue",
            },
            "position": {
                "lat": 47.63201,
                "lon": -122.13281
            },
            "viewport": {
                "topLeftPoint": {
                    "lat": 47.63291,
                    "lon": -122.13414,
                    "valid": true
                },
                "btmRightPoint": {
                    "lat": 47.63111,
                    "lon": -122.13148,
                    "valid": true
                }
            },
            "entryPoints": [
                {
                    "type": "main",
                    "position": {
                        "lat": 47.63222,
                        "lon": -122.13312,
                        "valid": true
                    }
                }
            ]
        },
        {
            "type": "POI",
            "id": "US/POI/p0/8831752",
            "score": 5.662710189819336,
            "dist": 1330.1278248163273,
            "info": "search:ta:840539001100326-US",
            "poi": {
                "name": "76",
                "phone": "+(1)-(425)-7472126",
                "brands": [
                    {
                        "name": "76"
                    }
                ],
                "categorySet": [
                    {
                        "id": 7311
                    }
                ],
                "url": "www.76.com",
                "categories": [
                    "petrol station"
                ],
                "classifications": [
                    {
                        "code": "PETROL_STATION",
                        "names": [
                            {
                                "nameLocale": "en-US",
                                "name": "petrol station"
                            }
                        ]
                    }
                ]
            },
            "address": {
                "streetNumber": "2421",
                "streetName": "148Th Ave Ne",
                "municipalitySubdivision": "Redmond, Bridle Trails, Bellevue",
                "municipality": "Redmond, Bellevue",
                "countrySecondarySubdivision": "King",
                "countryTertiarySubdivision": "Seattle East",
                "countrySubdivision": "WA",
                "countrySubdivisionName": "Washington",
                "postalCode": "98007",
                "countryCode": "US",
                "country": "United States",
                "countryCodeISO3": "USA",
                "freeformAddress": "2421 148Th Ave Ne, Bellevue, WA 98007",
                "localName": "Bellevue"
            },
            "position": {
                "lat": 47.63187,
                "lon": -122.14365
            },
            "viewport": {
                "topLeftPoint": {
                    "lat": 47.63277,
                    "lon": -122.14498,
                    "valid": true
                },
                "btmRightPoint": {
                    "lat": 47.63097,
                    "lon": -122.14232,
                    "valid": true
                }
            },
            "entryPoints": [
                {
                    "type": "minor",
                    "position": {
                        "lat": 47.63187,
                        "lon": -122.14374,
                        "valid": true
                    }
                },
                {
                    "type": "main",
                    "position": {
                        "lat": 47.63186,
                        "lon": -122.14313,
                        "valid": true
                    }
                }
            ]
        },
        {
            "type": "POI",
            "id": "US/POI/p0/8831764",
            "score": 5.662449836730957,
            "dist": 1458.645407416307,
            "info": "search:ta:840539000488527-US",
            "poi": {
                "name": "BROWN BEAR CAR WASH",
                "phone": "+(1)-(425)-6442868",
                "brands": [
                    {
                        "name": "Texaco"
                    }
                ],
                "categorySet": [
                    {
                        "id": 7311
                    }
                ],
                "url": "www.texaco.com/",
                "categories": [
                    "petrol station"
                ],
                "classifications": [
                    {
                        "code": "PETROL_STATION",
                        "names": [
                            {
                                "nameLocale": "en-US",
                                "name": "petrol station"
                            }
                        ]
                    }
                ]
            },
            "address": {
                "streetNumber": "15248",
                "streetName": "Bel Red Rd",
                "municipalitySubdivision": "Redmond",
            },
            "position": {
                "lat": 47.62843,
                "lon": -122.13628
            },
            "viewport": {
                "topLeftPoint": {
                    "lat": 47.62933,
                    "lon": -122.13761,
                    "valid": true
                },
                "btmRightPoint": {
                    "lat": 47.62753,
                    "lon": -122.13495,
                    "valid": true
                }
            },
            "entryPoints": [
                {
                    "type": "main",
                    "position": {
                        "lat": 47.62827,
                        "lon": -122.13628,
                        "valid": true
                    }
                }
            ]
        }
    ]
}
```


### <a name="airport-search"></a>Búsqueda de aeropuertos

La API de búsqueda de puntos de interés se puede usar para buscar aeropuertos usando su código oficial. Por ejemplo, puede usar *SEA* para buscar el aeropuerto Seattle-Tacoma International: 

```HTTP
https://atlas.microsoft.com/search/poi/json?subscription-key={subscription-key}&api-version=1.0&query=SEA 
```

### <a name="nearby-search"></a>Búsqueda de elementos cercanos

Para recuperar los resultados de puntos de interés en torno a una ubicación específica, puede probar la [API de búsqueda cercana](https://docs.microsoft.com/rest/api/maps/search/getsearchnearby). El punto de conexión devuelve solo resultados de puntos de interés. No toma un parámetro de consulta de búsqueda. 

Para limitar los resultados, se recomienda establecer el radio.

## <a name="understanding-the-responses"></a>Entender las respuestas

Vamos a buscar una dirección en Seattle enviando una solicitud de búsqueda de direcciones al servicio Search de Azure Maps. En la siguiente URL de solicitud, se establece el parámetro `countrySet` en `US` para buscar la dirección en Estados Unidos.

### <a name="sample-query"></a>Consulta de ejemplo

```HTTP
https://atlas.microsoft.com/search/address/json?subscription-key={subscription-key}&api-version=1&query=400%20Broad%20Street%2C%20Seattle%2C%20WA&countrySet=US
```

### <a name="supported-types-of-results"></a>Tipos de resultado admitidos

* **Dirección de punto**: puntos en un mapa que tienen una dirección específica, que incluye un nombre y número de calle. La dirección de punto proporciona el nivel más alto de precisión para las direcciones. 

* **Intervalo de direcciones**: intervalo de puntos de dirección que se interpolan desde el principio y el final de la calle.  

* **Geography**: áreas en un mapa que representan una división administrativa de un territorio, es decir, país o región, estado o ciudad. 

* **Punto de interés**: puntos de un mapa que merecen atención y pueden ser de interés.

* **Calle**: calles en el mapa. Las direcciones se resuelven en la coordenada de latitud y longitud de la calle que contiene la dirección. Es posible que no se pueda procesar el número de casa. 

* **Cruce**: intersecciones. Los cruces representan lugares en los que dos calles forman una intersección.

### <a name="response"></a>Response

Echemos un vistazo a la estructura de la respuesta. En la respuesta siguiente, los tipos de los objetos de resultado son diferentes. Si observa detenidamente, verá tres tipos de objetos de resultado:

* Dirección del punto
* Calle
* Cruce

Tenga en cuenta que la búsqueda de direcciones no devuelve puntos de interés.  

El parámetro `Score` de cada objeto de respuesta indica la relación entre la puntuación del resultado coincidente con las puntuaciones de otros objetos en la misma respuesta. Para más información sobre los parámetros de objeto de respuesta, consulte [Obtener dirección de búsqueda](https://docs.microsoft.com/rest/api/maps/search/getsearchaddress).

```JSON
{
    "summary": {
        "query": "400 broad street seattle wa",
        "queryType": "NON_NEAR",
        "queryTime": 146,
        "numResults": 6,
        "offset": 0,
        "totalResults": 7,
        "fuzzyLevel": 1
    },
    "results": [
        {
            "type": "Point Address",
            "id": "US/PAD/p0/28725082",
            "score": 9.893799781799316,
            "address": {
                "streetNumber": "400",
                "streetName": "Broad Street",
            },
            "position": {
                "lat": 47.62039,
                "lon": -122.34928
            },
            "viewport": {
                "topLeftPoint": {
                    "lat": 47.62129,
                    "lon": -122.35061,
                    "valid": true
                },
                "btmRightPoint": {
                    "lat": 47.61949,
                    "lon": -122.34795,
                    "valid": true
                }
            },
            "entryPoints": [
                {
                    "type": "main",
                    "position": {
                        "lat": 47.61982,
                        "lon": -122.34886,
                        "valid": true
                    }
                }
            ]
        },
        {
            "type": "Street",
            "id": "US/STR/p0/6700384",
            "score": 8.129190444946289,
            "address": {
                "streetName": "Broad Street",
            },
            "position": {
                "lat": 47.61724,
                "lon": -122.35207
            },
            "viewport": {
                "topLeftPoint": {
                    "lat": 47.61825,
                    "lon": -122.35336,
                    "valid": true
                },
                "btmRightPoint": {
                    "lat": 47.61626,
                    "lon": -122.35078,
                    "valid": true
                }
            }
        },
        {
            "type": "Street",
            "id": "US/STR/p0/9701953",
            "score": 8.129190444946289,
            "address": {
                "streetName": "Broad Street",
            },
            "position": {
                "lat": 47.61965,
                "lon": -122.349
            },
            "viewport": {
                "topLeftPoint": {
                    "lat": 47.62066,
                    "lon": -122.35041,
                    "valid": true
                },
                "btmRightPoint": {
                    "lat": 47.61857,
                    "lon": -122.34761,
                    "valid": true
                }
            }
        },
        {
            "type": "Street",
            "id": "US/STR/p0/11721297",
            "score": 8.129190444946289,
            "address": {
                "streetName": "Broad Street",
                "municipalitySubdivision": "Seattle, Downtown Seattle, Denny Regrade, Belltown",
                "municipality": "Seattle",
                "countrySecondarySubdivision": "King",
                "countryTertiarySubdivision": "Seattle",
                "countrySubdivision": "WA",
                "countrySubdivisionName": "Washington",
                "postalCode": "98121",
                "extendedPostalCode": "981211237",
                "countryCode": "US",
                "country": "United States",
                "countryCodeISO3": "USA",
                "freeformAddress": "Broad Street, Seattle, WA 98121"
            },
            "position": {
                "lat": 47.61825,
                "lon": -122.35078
            },
            "viewport": {
                "topLeftPoint": {
                    "lat": 47.61857,
                    "lon": -122.35078,
                    "valid": true
                },
                "btmRightPoint": {
                    "lat": 47.61825,
                    "lon": -122.35041,
                    "valid": true
                }
            }
        },
        {
            "type": "Cross Street",
            "id": "US/XSTR/p1/232144",
            "score": 6.754479885101318,
            "address": {
                "streetName": "Broad Street & Valley Street",
                "municipalitySubdivision": "South Lake Union, Seattle",
            },
            "position": {
                "lat": 47.62545,
                "lon": -122.33974
            },
            "viewport": {
                "topLeftPoint": {
                    "lat": 47.62635,
                    "lon": -122.34107,
                    "valid": true
                },
                "btmRightPoint": {
                    "lat": 47.62455,
                    "lon": -122.33841,
                    "valid": true
                }
            }
        }
    ]
}
```

### <a name="geometry"></a>Geometría

Un tipo de respuesta de *Geometría* puede incluir el identificador de geometría que se devuelve en el objeto `dataSources`, en `geometry` y `id`. Por ejemplo, puede usar el servicio [Buscar polígono](https://docs.microsoft.com/rest/api/maps/search/getsearchpolygon) para solicitar los datos de geometría en formato GeoJSON. Con este formato, puede obtener el contorno de una ciudad o aeropuerto para un conjunto de entidades. Después, puede usar estos datos de límite para [Configurar una geovalla](https://docs.microsoft.com/azure/azure-maps/tutorial-geofence) o [Buscar puntos de interés dentro de la geometría](https://docs.microsoft.com/rest/api/maps/search/postsearchinsidegeometry).


Las respuestas de la API de [búsqueda de direcciones](https://docs.microsoft.com/rest/api/maps/search/getsearchaddress) o la API de [búsqueda aproximada](https://docs.microsoft.com/rest/api/maps/search/getsearchfuzzy) pueden incluir el identificador de geometría que se devuelve en el objeto `dataSources`, en `geometry` y `id`:


```JSON 
"dataSources": { 
        "geometry": { 
            "id": "00005557-4100-3c00-0000-000059690938" // The geometry ID is returned in the dataSources object under "geometry" and "id".
        }
} 
```

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información, vea:

> [!div class="nextstepaction"]
> [Creación de solicitudes del servicio Search de Azure Maps](https://docs.microsoft.com/azure/azure-maps/how-to-search-for-address)

> [!div class="nextstepaction"]
> [Documentación de la API del servicio Search](https://docs.microsoft.com/rest/api/maps/search)
