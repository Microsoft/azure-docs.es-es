---
title: Detalles de los formatos de datos admitidos | Microsoft Azure Maps
description: Obtenga información sobre cómo se analizan los datos espaciales delimitados en el módulo de E/S espacial.
author: anastasia-ms
ms.author: v-stharr
ms.date: 03/03/2020
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: philmea
ms.openlocfilehash: 4b7c82e4650c7680709e809d9f563d79f068601f
ms.sourcegitcommit: 0e8a4671aa3f5a9a54231fea48bcfb432a1e528c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/24/2020
ms.locfileid: "87127934"
---
# <a name="supported-data-format-details"></a>Detalles de formatos de datos admitidos

En este artículo se proporcionan detalles sobre la compatibilidad de lectura y escritura para todas las etiquetas XML y los tipos de geometría de Well-Known Text. También detalla cómo se analizan los datos espaciales delimitados en el módulo de E/S espacial.

## <a name="supported-xml-namespaces"></a>Espacios de nombres XML admitidos

El módulo de E/S espacial admite etiquetas XML de los siguientes espacios de nombres.

| Prefijo de espacio de nombres | URI de espacio de nombres   | Notas                                                                    |
|:------------------|:-----------------|:----------------------------------------|
| `atom`           | `http://www.w3.org/2005/Atom`   |                                         |
| `geo`            | `http://www.w3.org/2003/01/geo/wgs84_pos#`  | Compatibilidad con solo lectura en archivos GeoRSS.           |
| `georss`         | `http://www.georss.org/georss`  |                                                |
| `geourl`         | `http://geourl.org/rss/module/` | Compatibilidad con solo lectura en archivos GeoRSS.                       |
| `gml`            | `http://www.opengis.net/gml`    |                                                        |
| `gpx`            | `http://www.topografix.com/GPX/1/1` |                                                   |
| `gpxx`           | `http://www.garmin.com/xmlschemas/GpxExtensions/v3` | Compatibilidad con solo lectura en archivos GPX. Analiza y usa DisplayColor. Todas las demás propiedades agregadas a los metadatos de la forma. |
| `gpx_style`      | `http://www.topografix.com/GPX/gpx_style/0/2`      | Se admite en archivos GPX. Usa color de línea. |
| `gx`             | `http://www.google.com/kml/ext/2.2` |                                                      |
| `kml`            | `http://www.opengis.net/kml/2.2`    |                                                      |
| `rss`            |                                 | Solo lectura. GeoRSS escribe con el formato Atom.              |

## <a name="supported-xml-elements"></a>Elementos XML admitidos

El módulo de E/S espacial admite los siguientes elementos XML. Las etiquetas XML que no se admitan se convertirán en un objeto JSON. A continuación, cada etiqueta se agregará como una propiedad en el campo `properties` de la capa o la forma primaria.

### <a name="kml-elements"></a>Elementos KML

El módulo de E/S espacial admite los siguientes elementos KML.

| Nombre del elemento         | Lectura    | Escritura   | Notas                                                                                                                      |
|----------------------|---------|---------|----------------------------------------------------------------------------------------------------------------------------|
| `address`            | parcialmente | sí     | El objeto se analiza, pero no se usa para la forma de posicionamiento.                                                                    |
| `AddressDetails`     | parcialmente | no      | El objeto se analiza, pero no se usa para la forma de posicionamiento.                                                                    |
| `atom:author`        | sí     | sí     |                                                                                                                            |
| `atom:link`          | sí     | sí     |                                                                                                                            |
| `atom:name`          | sí     | sí     |                                                                                                                            |
| `BalloonStyle`       | parcialmente | parcialmente | `displayMode` no se admite. Se convierte a una propiedad `PopupTemplate`. Para escribir, agregue una propiedad `popupTemplate` como una propiedad de la característica para la que desee escribirla. |
| `begin`              | sí     | sí     |                                                                                                                            |
| `color`              | sí     | sí     | Incluye `#AABBGGRR` y `#BBGGRR`. Analizado en una cadena de color CSS                                                           |
| `colorMode`          | sí     | no      |                                                                                                                            |
| `coordinates`        | sí     | sí     |                                                                                                                            |
| `Data`               | sí     | sí     |                                                                                                                            |
| `description`        | sí     | sí     |                                                                                                                            |
| `displayName`        | sí     | sí     |                                                                                                                            |
| `Document`           | sí     | sí     |                                                                                                                            |
| `drawOrder`          | parcialmente | no      | Se lee para las superposiciones de suelo y se usa para ordenarlas. 
| `east`               | sí     | sí     |                                                                                                                            |
| `end`                | sí     | sí     |                                                                                                                            |
| `ExtendedData`       | sí     | sí     | Admite `Data`, `SimpleData` o `Schema` sin tipo y los reemplazos de entidad del formulario `$[dataName]`.                      |
| `extrude`            | parcialmente | parcialmente | Solo se admite para los polígonos. La multigeometría que tenga polígonos de diferentes altos se dividirá en características individuales. No se admiten los estilos de línea. Los polígonos con una altitud de 0 se representarán como un polígono plano. Al leer, la altitud de la primera coordenada del anillo exterior se agregará como una propiedad de alto del polígono. A continuación, se usará la altitud de la primera coordenada para representar el polígono en el mapa. |
| `fill`               | sí     | sí     |                                                                                                                            |
| `Folder`             | sí     | sí     |                                                                                                                            |
| `GroundOverlay`      | sí     | sí     | `color` no se admite.                                                                                                   |
| `heading`            | parcialmente | no      | Analizado pero no representado por `SimpleDataLayer`. Solo escribe si los datos se almacenan en la propiedad de la forma.                 |
| `hotSpot`            | sí     | parcialmente | Solo escribe si los datos se almacenan en la propiedad de la forma. Las unidades se configuran solo como "píxeles".                         |
| `href`               | sí     | sí     |                                                                                                                            |
| `Icon`               | parcialmente | parcialmente | Analizado pero no representado por `SimpleDataLayer`. Solo escribe la propiedad icon de la forma si contiene un URI de datos. Solo se admite `href`. |
| `IconStyle`          | parcialmente | parcialmente | Se analizan los valores `icon`, `heading`, `colorMode` y `hotspots`, pero no se representan en `SimpleDataLayer`         |
| `innerBoundaryIs`    | sí     | sí     |                                                                                                                            |
| `kml`                | sí     | sí     |                                                                                                                            |
| `LabelStyle`         | no      | no      |                                                                                                                            |
| `LatLonBox`          | sí     | sí     |                                                                                                                            |
| `gx:LatLonQuad`      | sí     | sí     |                                                                                                                            |
| `LinearRing`         | sí     | sí     |                                                                                                                            |
| `LineString`         | sí     | sí     |                                                                                                                            |
| `LineStyle`          | sí     | sí     | `colorMode` no se admite.                                                                                         |
| `Link`               | sí     | no      | Solo se admite la propiedad `href` para los vínculos de red.                                                                   |
| `MultiGeometry`      | parcialmente | parcialmente | Se puede desglosar en características individuales cuando se lee.                                                                     |
| `name`               | sí     | sí     |                                                                                                                            |
| `NetworkLink`        | sí     | no      | Los vínculos deben estar en el mismo dominio que el documento.                                                                  |
| `NetworkLinkControl` | no      | no      |                                                                                                                            |
| `north`              | sí     | sí     |                                                                                                                            |
| `open`               | sí     | sí     |                                                                                                                            |
| `outerBoundaryIs`    | sí     | sí     |                                                                                                                            |
| `outline`            | sí     | sí     |                                                                                                                            |
| `overlayXY`          | no      | no      |                                                                                                                            |
| `Pair`               | parcialmente | no      | Solo se admite el estilo `normal` en un `StyleMap`. `highlight` no se admite.                                   |
| `phoneNumber`        | sí     | sí     |                                                                                                                            |
| `PhotoOverlay`       | no      | no      |                                                                                                                            |
| `Placemark`          | sí     | sí     |                                                                                                                            |
| `Point`              | sí     | sí     |                                                                                                                            |
| `Polygon`            | sí     | sí     |                                                                                                                            |
| `PolyStyle`          | sí     | sí     |                                                                                                                            |
| `Region`             | parcialmente | parcialmente | `LatLongBox` se admite en el nivel de documento.                                                                      |
| `rotation`           | no      | no      |                                                                                                                            |
| `rotationXY`         | no      | no      |                                                                                                                            |
| `scale`              | no      | no      |                                                                                                                            |
| `Schema`             | sí     | sí     |                                                                                                                            |
| `SchemaData`         | sí     | sí     |                                                                                                                            |
| `schemaUrl`          | parcialmente | sí     | No admite la carga de estilos de documentos externos que no están incluidos en una KMZ.                             |
| `ScreenOverlay`      | no      | no      |                                                                                                                            |
| `screenXY`           | no      | no      |                                                                                                                            |
| `SimpleData`         | sí     | sí     |                                                                                                                            |
| `SimpleField`        | sí     | sí     |                                                                                                                            |
| `size`               | no      | no      |                                                                                                                            |
| `Snippet`            | parcialmente | parcialmente | El atributo `maxLines` se omite.                                                                                  |
| `south`              | sí     | sí     |                                                                                                                            |
| `Style`              | sí     | sí     |                                                                                                                            |
| `StyleMap`           | parcialmente | no      | Solo se admite el estilo normal en un `StyleMap`.                                                                        |
| `styleUrl`           | parcialmente | sí     | No se admiten direcciones URL de estilo externo.                                                                         |
| `text`               | sí     | sí     | No se admite la sustitución de `$[geDirections]`                                                                          |
| `textColor`          | sí     | sí     |                                                                                                                            |
| `TimeSpan`           | sí     | sí     |                                                                                                                            |
| `TimeStamp`          | sí     | sí     |                                                                                                                            |
| `value`              | sí     | sí     |                                                                                                                            |
| `viewRefreshMode`    | parcialmente | no      |  Si apunta a un servicio WMS, solo se admite `onStop` para las superposiciones de suelo. Anexará `BBOX=[bboxWest],[bboxSouth],[bboxEast],[bboxNorth]` a la dirección URL y se actualizará a medida que se mueva la asignación.  |
| `visibility`         | sí     | sí     |                                                                                                                            |
| `west`               | sí     | sí     |                                                                                                                            |
| `when`               | sí     | sí     |                                                                                                                            |
| `width`              | sí     | sí     |                                                                                                                            |

### <a name="georss-elements"></a>Elementos GeoRSS

El módulo de E/S espacial admite los siguientes elementos GeoRSS.

| Nombre del elemento             | Lectura    | Escritura | Notas                                                                                          |
|--------------------------|---------|-------|------------------------------------------------------------------------------------------------|
| `atom:author`            | sí     | sí   |                                                                                                |
| `atom:category`          | sí     | sí   |                                                                                                |
| `atom:content`           | sí     | sí   |                                                                                                |
| `atom:contributor`       | sí     | sí   |                                                                                                |
| `atom:email`             | sí     | sí   |                                                                                                |
| `atom:entry`             | sí     | sí   |                                                                                                |
| `atom:feed`              | sí     | sí   |                                                                                                |
| `atom:icon`              | sí     | sí   |                                                                                                |
| `atom:id`                | sí     | sí   |                                                                                                |
| `atom:link`              | sí     | sí   |                                                                                                |
| `atom:logo`              | sí     | sí   |                                                                                                |
| `atom:name`              | sí     | sí   |                                                                                                |
| `atom:published`         | sí     | sí   |                                                                                                |
| `atom:rights`            | sí     | sí   |                                                                                                |
| `atom:source`            | sí     | sí   |                                                                                                |
| `atom:subtitle`          | sí     | sí   |                                                                                                |
| `atom:summary`           | sí     | sí   |                                                                                                |
| `atom:title`             | sí     | sí   |                                                                                                |
| `atom:updated`           | sí     | sí   |                                                                                                |
| `atom:uri`               | sí     | sí   |                                                                                                |
| `geo:lat`                | sí     | no    | Se escribe como un `georss:point`.                                                                   |
| `geo:lon`                | sí     | no    | Se escribe como un `georss:point`.                                                                   |
| `geo:long`               | sí     | no    | Se escribe como un `georss:point`.                                                                   |
| `georss:box`             | sí     | no    | Se lee como un polígono y se le asigna una propiedad `subType` de "Rectangle".                                |
| `georss:circle`          | sí     | sí   |                                                                                                |
| `georss:elev`            | sí     | sí   |                                                                                                |
| `georss:featurename`     | sí     | sí   |                                                                                                |
| `georss:featuretypetag`  | sí     | sí   |                                                                                                |
| `georss:floor`           | sí     | sí   |                                                                                                |
| `georss:line`            | sí     | sí   |                                                                                                |
| `georss:point`           | sí     | sí   |                                                                                                |
| `georss:polygon`         | sí     | sí   |                                                                                                |
| `georss:radius`          | sí     | sí   |                                                                                                |
| `georss:relationshiptag` | sí     | sí   |                                                                                                |
| `georss:where`           | sí     | sí   |                                                                                                |
| `geourl:latitude`        | sí     | no    | Se escribe como un `georss:point`.                                                                   |
| `geourl:longitude`       | sí     | no    | Se escribe como un `georss:point`.                                                                   |
| `position`               | sí     | no    | Algunas fuentes XML ajustan GML con una etiqueta de posición en lugar de encapsularla con una etiqueta `georss:where`. Leerá esta etiqueta, pero la escribirá con una etiqueta `georss:where`. |
| `rss`                    | sí     | no    | GeoRSS escrito en formato ATOM.                                                                 |
| `rss:author`             | sí     | parcialmente | Se escribe como un `atom:author`.                                                                 |
| `rss:category`           | sí     | parcialmente | Se escribe como un `atom:category`.                                                               |
| `rss:channel`            | sí     | no    |                                                                                                |
| `rss:cloud`              | sí     | no    |                                                                                                |
| `rss:comments`           | sí     | no    |                                                                                                |
| `rss:copyright`          | sí     | parcialmente | Se escribe como un `atom:rights` si la forma no tiene ya una propiedad `properties` `rights`.       |
| `rss:description`        | sí     | parcialmente | Se escribe como un `atom:content` si la forma no tiene ya una propiedad `properties` `content`.      |
| `rss:docs`               | sí     | no    |                                                                                                |
| `rss:enclosure`          | sí     | no    |                                                                                                |
| `rss:generator`          | sí     | no    |                                                                                                |
| `rss:guid`               | sí     | parcialmente | Se escribe como un `atom:id` si la forma no tiene ya una propiedad `properties` `id`.         |
| `rss:image`              | sí     | parcialmente | Se escribe como un `atom:logo` si la forma no tiene ya una propiedad `properties` `logo`.      |
| `rss:item`               | sí     | parcialmente | Se escribe como un `atom:entry`.                                                                  |
| `rss:language`           | sí     | no    |                                                                                                |
| `rss:lastBuildDate`      | sí     | parcialmente | Se escribe como un `atom:updated` si la forma no tiene ya una propiedad `properties` `updated`.     |
| `rss:link`               | sí     | parcialmente | Se escribe como un `atom:link`.                                                                   |
| `rss:managingEditor`     | sí     | parcialmente | Se escribe como un `atom:contributor`.                                                            |
| `rss:pubDate`            | sí     | parcialmente | Se escribe como un `atom:published` si la forma no tiene ya una propiedad `properties` `published`.  |
| `rss:rating`             | sí     | no    |                                                                                                |
| `rss:skipDays`           | sí     | no    |                                                                                                |
| `rss:skipHours`          | sí     | no    |                                                                                                |
| `rss:source`             | sí     | parcialmente | Se escribe como un `atom:source` que contiene un `atom:link`.                                       |
| `rss:textInput`          | sí     | no    |                                                                                                |
| `rss:title`              | sí     | parcialmente | Se escribe como un `atom:title`.                                                                  |
| `rss:ttl`                | sí     | no    |                                                                                                |
| `rss:webMaster`          | sí     | no    |                                                                                                |

### <a name="gml-elements"></a>Elementos de GML

El módulo de E/S espacial admite los siguientes elementos de GML. 

| Nombre del elemento            | Lectura | Escritura | Notas                                                                                  |
|-------------------------|------|-------|----------------------------------------------------------------------------------------|
| `gml:coordinates`       | sí  | no    | Se escribe como `gml:posList`.                                                              |
| `gml:curveMember`       | sí  | no    |                                                                                        |
| `gml:curveMembers`      | sí  | no    |                                                                                        |
| `gml:Box`               | sí  | no    | Se escribe como `gml:Envelope`.                                                             |
| `gml:description`       | sí  | sí   |                                                                                        |
| `gml:Envelope`          | sí  | sí   |                                                                                        |
| `gml:exterior`          | sí  | sí   |                                                                                        |
| `gml:Feature`           | sí  | no    | Se escribe como una forma.                                                                    |
| `gml:FeatureCollection` | sí  | no    | Se escribe como una colección Geometry.                                                      |
| `gml:featureMember`     | sí  | no    | Se escribe como una colección Geometry.                                                      |
| `gml:geometry`          | sí  | no    | Se escribe como una forma.                                                                    |
| `gml:geometryMember`    | sí  | sí   |                                                                                        |
| `gml:geometryMembers`   | sí  | sí   |                                                                                        |
| `gml:identifier`        | sí  | sí   |                                                                                        |
| `gml:innerBoundaryIs`   | sí  | no    | Se escribe con `gml.interior`.                                                          |
| `gml:interior`          | sí  | sí   |                                                                                        |
| `gml:LinearRing`        | sí  | sí   |                                                                                        |
| `gml:LineString`        | sí  | sí   |                                                                                        |
| `gml:lineStringMember`  | sí  | sí   |                                                                                        |
| `gml:lineStringMembers` | sí  | no    |                                                                                        |
| `gml:MultiCurve`        | sí  | no    | Solo lee miembros `gml:LineString`. Se escribe como `gml.MultiLineString`                  |
| `gml:MultiGeometry`     | parcialmente  | parcialmente   | Solo se lee como FeatureCollection.                                              |
| `gml:MultiLineString`   | sí  | sí   |                                                                                        |
| `gml:MultiPoint`        | sí  | sí   |                                                                                        |
| `gml:MultiPolygon`      | sí  | sí   |                                                                                        |
| `gml:MultiSurface`      | sí  | no    | Solo lee miembros `gml:Polygon`. Se escribe como `gml.MultiPolygon`                        |
| `gml:name`              | sí  | sí   |                                                                                        |
| `gml:outerBoundaryIs`   | sí  | no    | Se escribe con `gml.exterior`.                                                          |
| `gml:Point`             | sí  | sí   |                                                                                        |
| `gml:pointMember`       | sí  | sí   |                                                                                        |
| `gml:pointMembers`      | sí  | no    |                                                                                        |
| `gml:Polygon`           | sí  | sí   |                                                                                        |
| `gml:polygonMember`     | sí  | sí   |                                                                                        |
| `gml:polygonMembers`    | sí  | no    |                                                                                        |
| `gml:pos`               | sí  | sí   |                                                                                        |
| `gml:posList`           | sí  | sí   |                                                                                        |
| `gml:surfaceMember`     | sí  | sí   |                                                                                        |

#### <a name="additional-notes"></a>Notas adicionales

- Se buscará en los elementos de miembro una geometría que pueda estar oculta en los elementos secundarios. Esta operación de búsqueda es necesaria, ya que muchos formatos XML que se extienden desde GML pueden no colocar una geometría como elemento secundario directo de un elemento de miembro.
- `srsName` es parcialmente compatible con las coordenadas WGS84 y los códigos siguientes: [EPSG: 4326](https://epsg.io/4326) y web Mercator [EPSG: 3857](https://epsg.io/3857), o uno de sus códigos alternativos. Cualquier otro sistema de coordenadas se analizará como WGS84 tal cual.
- A menos que se especifique al leer una fuente XML, el orden del eje se determina en función de las sugerencias de la fuente XML. Se proporciona una preferencia para el orden de los ejes "latitud, longitud".
- A menos que se especifique un espacio de nombres GML personalizado para las propiedades al escribir en un archivo GML, no se agregará información de propiedad adicional.

### <a name="gpx-elements"></a>Elementos GPX

El módulo de E/S espacial admite los siguientes elementos GPX.

| Nombre del elemento             | Lectura    | Escritura   | Notas                                                                                       |
|--------------------------|---------|---------|---------------------------------------------------------------------------------------------|
| `gpx:ageofdgpsdata`      | sí     | sí     |                                                                                             |
| `gpx:author`             | sí     | sí     |                                                                                             |
| `gpx:bounds`             | sí     | sí     | Se convierte en un LocationRect cuando se lee.                                                    |
| `gpx:cmt`                | sí     | sí     |                                                                                             |
| `gpx:copyright`          | sí     | sí     |                                                                                             |
| `gpx:desc`               | sí     | sí     | Se copia en una propiedad description cuando la lectura se alinea con otros formatos XML.               |
| `gpx:dgpsid`             | sí     | sí     |                                                                                             |
| `gpx:ele`                | sí     | sí     |                                                                                             |
| `gpx:extensions`         | parcialmente | parcialmente | Cuando se lee, se extrae la información de estilo. Todas las demás extensiones se acoplarán en un objeto JSON simple. Solo se escribe la información de estilo de la forma. |
| `gpx:geoidheight`        | sí     | sí     |                                                                                             |
| `gpx:gpx`                | sí     | sí     |                                                                                             |
| `gpx:hdop`               | sí     | sí     |                                                                                             |
| `gpx:link`               | sí     | sí     |                                                                                             |
| `gpx:magvar`             | sí     | sí     |                                                                                             |
| `gpx:metadata`           | sí     | sí     |                                                                                             |
| `gpx:name`               | sí     | sí     |                                                                                             |
| `gpx:pdop`               | sí     | sí     |                                                                                             |
| `gpx:rte`                | sí     | sí     |                                                                                             |
| `gpx:rtept`              | sí     | sí     |                                                                                             |
| `gpx:sat`                | sí     | sí     |                                                                                             |
| `gpx:src`                | sí     | sí     |                                                                                             |
| `gpx:sym`                | sí     | sí     | Se captura el valor, pero no se usa para modificar el icono de alfiler.                               |
| `gpx:text`               | sí     | sí     |                                                                                             |
| `gpx:time`               | sí     | sí     |                                                                                             |
| `gpx:trk`                | sí     | sí     |                                                                                             |
| `gpx:trkpt`              | sí     | sí     |                                                                                             |
| `gpx:trkseg`             | sí     | sí     |                                                                                             |
| `gpx:type`               | sí     | sí     |                                                                                             |
| `gpx:vdop`               | sí     | sí     |                                                                                             |
| `gpx:wpt`                | sí     | sí     |                                                                                             |
| `gpx_style:color`        | sí     | sí     |                                                                                             |
| `gpx_style:line`         | parcialmente | parcialmente | Se admiten `color`, `opacity`, `width` y `lineCap`.                                           |
| `gpx_style:opacity`      | sí     | sí     |                                                                                             |
| `gpx_style:width`        | sí     | sí     |                                                                                             |
| `gpxx:DisplayColor`      | sí     | no      | Se utiliza para especificar el color de una forma. Al escribir, en su lugar se utilizará el color `gpx_style:line`.  |
| `gpxx:RouteExtension`    | parcialmente | no      | Todas las propiedades se leen en `properties`. Solo se utiliza `DisplayColor`.                     |
| `gpxx:TrackExtension`    | parcialmente | no      | Todas las propiedades se leen en `properties`. Solo se utiliza `DisplayColor`.                     |
| `gpxx:WaypointExtension` | parcialmente | no      | Todas las propiedades se leen en `properties`. Solo se utiliza `DisplayColor`.                     |
| `gpx:keywords`           | sí     | sí     |                                                                                             |
| `gpx:fix`                | sí     | sí     |                                                                                             |

#### <a name="additional-notes"></a>Notas adicionales

Al escribir;

- Los puntos de Multipoint se dividirán en puntos de trayecto individuales.
- Polygons y MultiPolygons se escribirán como pistas. 
  
## <a name="supported-well-known-text-geometry-types"></a>Tipos de geometría de texto Well-Known Text

| Tipo Geometry | Lectura | Escritura |
|--------------|:----:|:-----:|
| POINT | x | x |
| POINT Z | x | x | 
| POINT M | x | x<sup>[2]</sup> |
| POINT ZM | x<sup>[1]</sup><sup>[2]</sup> | | 
| LINESTRING | x | x |
| LINESTRING Z | x | x | 
| LINESTRING M | x | x<sup>[2]</sup> |
| LINESTRING ZM | x<sup>[1]</sup><sup>[2]</sup> | | 
| POLYGON | x | x |
| POLYGON Z | x | x |
| POLYGON M | x | x<sup>[2]</sup> |
| POLYGON ZM | x<sup>[1]</sup><sup>[2]</sup> | | 
| MULTIPOINT | x | x |
| MULTIPOINT Z | x | x | 
| MULTIPOINT M | x | x<sup>[2]</sup> |
| POMULTIPOINTINT ZM | x<sup>[1]</sup><sup>[2]</sup> | | 
| MULTILINESTRING | x | x |
| MULTILINESTRING Z | x | x | 
| MULTILINESTRING M | x | x<sup>[2]</sup> |
| MULTILINESTRING ZM | x<sup>[1]</sup><sup>[2]</sup> | | 
| MULTIPOLYGON | x | x |
| MULTIPOLYGON Z | x | x | 
| MULTIPOLYGON M | x | x<sup>[2]</sup> |
| MULTIPOLYGON ZM | x<sup>[1]</sup><sup>[2]</sup> | | 
| GEOMETRYCOLLECTION | x | x |
| GEOMETRYCOLLECTION Z | x | x | 
| GEOMETRYCOLLECTION M | x | x<sup>[2]</sup> | 
| GEOMETRYCOLLECTION ZM | x<sup>[1]</sup><sup>[2]</sup> | x | 

\[1\] Solo se captura el parámetro Z y se agrega como tercer valor en el valor de Position.

\[2\] El parámetro M no se captura.

## <a name="delimited-spatial-data-support"></a>Compatibilidad con datos espaciales delimitados

Los datos espaciales delimitados, como los archivos de valores separados por comas (CSV), suelen tener columnas que contienen datos espaciales. Por ejemplo, puede haber columnas que contengan información de latitud y longitud. En el formato Well-Known Text (WKT) puede haber una columna que contenga datos de geometría espacial.

### <a name="spatial-data-column-detection"></a>Detección de columnas de datos espaciales

Al leer un archivo delimitado que contenga datos espaciales, se analizará el encabezado para determinar qué columnas contienen campos de ubicación. Si el encabezado contiene información de tipos, se usará para convertir los valores de celda al tipo adecuado. Si no se especifica ningún encabezado, se analizará la primera fila y se utilizará para generar un encabezado. Al analizar la primera fila, se ejecuta una comprobación para buscar coincidencias de los nombres de columna con los nombres siguientes, sin distinción entre mayúsculas y minúsculas. El orden de los nombres es la prioridad, en caso de que existan dos o más nombres en un archivo.

#### <a name="latitude"></a>Latitud

- `latitude`
- `lat`
- `latdd`
- `lat_dd`
- `latitude83`
- `latdecdeg`
- `y`
- `ycenter`
- `point-y`

#### <a name="longitude"></a>Longitud

- `longitude`
- `lon`
- `lng`
- `long`
- `longdd`
- `long_dd`
- `longitude83`
- `longdecdeg`
- `x`
- `xcenter`
- `point-x`

#### <a name="elevation"></a>Elevation

- `elevation`
- `elv`
- `altitude`
- `alt`
- `z`

#### <a name="geography"></a>Geography

La primera fila de datos se examinará en busca de cadenas con un formato WKT. 

### <a name="delimited-data-column-types"></a>Tipos de columnas de datos delimitados

Al examinar la fila de encabezado, toda la información de tipo que se encuentre en el nombre de columna se extrae y se utiliza para convertir las celdas de esa columna. A continuación se muestra un ejemplo de un nombre de columna que tiene un valor de tipo: "ColumnName (typeName)". Se admiten los siguientes nombres de tipo que no distinguen mayúsculas de minúsculas:

#### <a name="numbers"></a>Números

- edm.int64
- int
- long
- edm.double
- FLOAT
- double
- number

#### <a name="booleans"></a>Valores booleanos

- edm.boolean
- bool
- boolean

#### <a name="dates"></a>Fechas

- edm.datetime
- date
- datetime

#### <a name="geography"></a>Geography

- edm.geography
- geography

#### <a name="strings"></a>Cadenas

- edm.string
- varchar
- text
- string

Si no se puede extraer información de tipo del encabezado y la opción de escritura dinámica está habilitada al leer, cada celda se analizará individualmente para determinar qué tipo de datos es el más adecuado para convertirse.

## <a name="next-steps"></a>Pasos siguientes

Para obtener más ejemplos de código para agregar a los mapas:

> [!div class="nextstepaction"]
> [Lectura y escritura de datos espaciales](spatial-io-read-write-spatial-data.md)
