---
title: Formato de datos GeoJSON para geovalla | Microsoft Azure Maps
description: En este artículo, obtendrá información sobre cómo preparar los datos de geovalla para que se usen en las API GET Geofence y POST Geofence de Microsoft Azure Maps.
author: anastasia-ms
ms.author: v-stharr
ms.date: 02/14/2019
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: ''
ms.openlocfilehash: 924c23f0fb0156ff585872dded72932a1574a12d
ms.sourcegitcommit: 0e8a4671aa3f5a9a54231fea48bcfb432a1e528c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/24/2020
ms.locfileid: "87131312"
---
# <a name="geofencing-geojson-data"></a>Datos GeoJSON de geovallas

Las API [GET Geofence](/rest/api/maps/spatial/getgeofence) y [POST Geofence](/rest/api/maps/spatial/postgeofence) de Azure Maps le permiten recuperar la proximidad de una coordenada en relación a una geovalla o conjunto de vallas proporcionados. En este artículo se describe cómo preparar los datos de la geovalla para que se usen en las API GET y POST de Azure Maps.

Los datos de la geovalla o conjunto de estas viene dado por el objeto `Feature` y el objeto `FeatureCollection` en formato `GeoJSON`, que se define en [rfc7946](https://tools.ietf.org/html/rfc7946). Además de esto:

* El tipo de objeto GeoJSON puede ser un objeto `Feature` o un objeto `FeatureCollection`.
* El tipo de objeto Geometry puede ser `Point`, `MultiPoint`, `LineString`, `MultiLineString`, `Polygon`, `MultiPolygon`, y `GeometryCollection`.
* Todas las propiedades de la característica deben contener un identificador `geometryId`, que se usa para identificar la geovalla.
* Las características con `Point`, `MultiPoint`, `LineString`, `MultiLineString` deben contener `radius` en las propiedades. `radius`El valor de `radius` se mide en metros y oscila entre 1 y 10 000.
* La característica con los tipos de geometría `polygon` y `multipolygon` no tiene una propiedad radius.
* `validityTime` es una propiedad opcional que permite al usuario establecer el tiempo de expiración y el período de tiempo de validez de los datos de la geovalla. Si no se especifica, los datos nunca expirarán y serán siempre válidos.
* `expiredTime` es la fecha y hora de expiración de los datos de la geovalla. Si el valor de `userTime` en la solicitud es posterior a este valor, los datos de la geovalla correspondiente se consideran como datos expirados y no se realizarán consultas en ellos. El identificador geometryId de estos datos de geovalla se incluirá en la matriz `expiredGeofenceGeometryId` dentro de la respuesta de la geovalla.
* `validityPeriod` es una lista de los períodos de tiempo de validez de la geovalla. Si el valor de `userTime` en la solicitud se encuentra fuera del período de validez, los datos correspondientes de la geovalla se considerarán como no válidos y no se realizarán consultas en ellos. El identificador geometryId de estos datos de geovalla se incluye en la matriz `invalidPeriodGeofenceGeometryId` dentro de la respuesta de la geovalla. En la tabla siguiente se muestran las propiedades del elemento validityPeriod.

| Nombre | Tipo | Obligatorio  | Descripción |
| :------------ |:------------: |:---------------:| :-----|
| startTime | Datetime  | true | Fecha y hora de inicio del período de tiempo de validez. |
| endTime   | Datetime  | true |  Fecha y hora de finalización del período de tiempo de validez. |
| recurrenceType | string | false |   El tipo de periodicidad del período. El valor puede ser `Daily`, `Weekly`, `Monthly` o `Yearly`. El valor predeterminado es `Daily`.|
| businessDayOnly | Boolean | false |  Indica si los datos solo son válidos durante los días laborables. El valor predeterminado es `false`.|


* Todos los valores de las coordenadas se representan como [longitud, latitud] que se define en `WGS84`.
* Para cada característica, que contiene `MultiPoint`, `MultiLineString`, `MultiPolygon` o `GeometryCollection`, las propiedades se aplican a todos los elementos. Por ejemplo: Todos los puntos de `MultiPoint` usarán el mismo radio para formar una geovalla de varios círculos.
* En un escenario de punto-círculo, se puede representar una geometría de círculo mediante un objeto de geometría `Point` con las propiedades que se elaboran en [Extender las geometrías GeoJSON](https://docs.microsoft.com/azure/azure-maps/extend-geojson).      

El siguiente es un ejemplo de cuerpo de la solicitud de una geovalla que se representa como una geometría de geovalla circular en `GeoJSON` mediante un punto central y un radio. El período de validez de los datos de la geovalla se inicia el 22-10-2018, de 9 a.m. a 5 p.m., y se repite todos los días excepto los fines de semana. `expiredTime` indica que se considerará que los datos de esta geovalla han expirado si `userTime` en la solicitud es posterior a `2019-01-01`.  

```json
{
    "type": "Feature",
    "geometry": {
        "type": "Point",
        "coordinates": [-122.126986, 47.639754]
    },
    "properties": {
        "geometryId" : "1",
        "subType": "Circle",
        "radius": 500,
        "validityTime": 
        {
            "expiredTime": "2019-01-01T00:00:00",
            "validityPeriod": [
                {
                    "startTime": "2018-10-22T09:00:00",
                    "endTime": "2018-10-22T17:00:00",
                    "recurrenceType": "Daily",
                    "recurrenceFrequency": 1,
                    "businessDayOnly": true
                }
            ]
        }
    }
}
```
