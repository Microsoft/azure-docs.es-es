---
title: Creación de un origen de datos para un mapa | Microsoft Azure Maps
description: 'Obtenga información sobre cómo crear un origen de datos para un mapa. Aprenda sobre los orígenes de datos que usa el SDK Web de Azure Maps: Orígenes GeoJSON e iconos vectoriales.'
author: rbrundritt
ms.author: richbrun
ms.date: 08/08/2019
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: cpendle
ms.custom: codepen, devx-track-javascript
ms.openlocfilehash: a6ee36e3aee515b589f1e460626df91a38f79f25
ms.sourcegitcommit: 07166a1ff8bd23f5e1c49d4fd12badbca5ebd19c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/15/2020
ms.locfileid: "90089384"
---
# <a name="create-a-data-source"></a>Creación de un origen de datos

El SDK web de Azure Maps almacena los datos en orígenes de datos. El uso de orígenes de datos optimiza las operaciones de datos en las consultas y representaciones. Actualmente hay dos tipos de orígenes de datos:

- **Origen GeoJSON**: administra localmente los datos de ubicación sin procesar en formato GeoJSON. Adecuado para conjuntos de datos de tamaño pequeño a medio (con cientos de miles de formas).
- **Origen de mosaico vectorial**: carga los datos con formato de mosaico vectorial para la vista del mapa actual, en función del sistema de mosaico de mapas. Ideal para conjuntos de datos grandes o masivos (millones o miles de millones de formas).

## <a name="geojson-data-source"></a>Origen de datos de GeoJSON

Un origen de datos basado en GeoJSON puede cargar y almacenar datos localmente mediante la clase `DataSource`. Los datos de GeoJSON se pueden crear manualmente o mediante las clases auxiliares del espacio de nombres de [atlas.data](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.data). La clase `DataSource` proporciona funciones para importar archivos GeoJSON locales o remotos. Los archivos GeoJSON remotos deben hospedarse en un punto de conexión habilitado para CORs. La clase `DataSource` proporciona funcionalidad para agrupar en clústeres los datos de punto. Además, los datos se pueden agregar, quitar y actualizar fácilmente con la clase `DataSource`. En el siguiente código se muestra cómo crear datos GeoJSON en Azure Maps.

```javascript
//Create raw GeoJSON object.
var rawGeoJson = {
     "type": "Feature",
     "geometry": {
         "type": "Point",
         "coordinates": [-100, 45]
     },
     "properties": {
         "custom-property": "value"
     }
};

//Create GeoJSON using helper classes (less error prone and less typing).
var geoJsonClass = new atlas.data.Feature(new atlas.data.Point([-100, 45]), {
    "custom-property": "value"
}); 
```

Una vez creados, los orígenes de datos se pueden agregar a la asignación mediante la propiedad `map.sources`, que es un objeto [SourceManager](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.sourcemanager). El código siguiente muestra cómo crear un `DataSource` y agregarlo al mapa.

```javascript
//Create a data source and add it to the map.
var dataSource = new atlas.source.DataSource();
map.sources.add(dataSource);
```

En el siguiente código se muestran las distintas formas de agregar datos de GeoJSON a `DataSource`.

```javascript
//GeoJsonData in the following code can be a single or array of GeoJSON features or geometries, a GeoJSON feature colleciton, or a single or array of atlas.Shape objects.

//Add geoJSON object to data source. 
dataSource.add(geoJsonData);

//Load geoJSON data from URL. URL should be on a CORs enabled endpoint.
dataSource.importDataFromUrl(geoJsonUrl);

//Overwrite all data in data source.
dataSource.setShapes(geoJsonData);
```

> [!TIP]
> Supongamos que quiere sobrescribir todos los datos de un elemento `DataSource`. Si realiza llamadas a las funciones `clear` y `add`, es posible que el mapa se vuelva a representar dos veces, lo que puede provocar cierto retraso. Use en su lugar la función `setShapes`, que quitará y reemplazará todos los datos del origen de datos y solo desencadenará una nueva representación del mapa.

## <a name="vector-tile-source"></a>Origen de mosaico vectorial

Un origen de mosaico vectorial describe cómo acceder a una capa de mosaico vectorial. Use la clase [VectorTileSource](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.source.vectortilesource) para crear una instancia de un origen de mosaico vectorial. Las capas de mosaico vectorial son similares a las de mosaico, pero no son iguales. Una capa de mosaico es una imagen de trama. Las capas de mosaico vectorial son un archivo comprimido, en formato **PBF**. Este archivo comprimido contiene datos de mapas vectoriales y una o varias capas. El archivo se puede representar en el cliente, y aplicarle un estilo, en función del estilo de cada capa. Los datos de un mosaico vectorial contienen características geográficas en forma de puntos, líneas y polígonos. El uso de capas de mosaico vectorial en lugar de capas de mosaico de trama presenta varias ventajas:

 - Un tamaño de archivo de un mosaico vectorial suele ser mucho menor que el de un mosaico de trama equivalente. Como tal, se usa menos ancho de banda. Esto significa una menor latencia, un mapa más rápido y una mejor experiencia de usuario.
 - Como los mosaicos vectoriales se representan en el cliente, pueden adaptarse a la resolución del dispositivo en el que se muestran. Como resultado, los mapas representados aparecen mejor definidos, con etiquetas nítidas.
 - Para cambiar el estilo de los datos en los mapas vectoriales no es necesario descargar los datos de nuevo, ya que el nuevo estilo se puede aplicar en el cliente. Al contrario, para cambiar el estilo de una capa de mosaico de trama es necesario normalmente cargar los mosaicos desde el servidor y luego aplicar el nuevo estilo.
 - Dado que los datos se entregan en forma de vector, se requiere menos procesamiento del lado servidor para preparar los datos. Como resultado, los datos más recientes pueden estar disponibles más rápido.

Azure Maps se adhiere a la [especificación de mosaicos vectoriales de Mapbox](https://github.com/mapbox/vector-tile-spec), un estándar abierto. Azure Maps proporciona los siguientes servicios de mosaicos vectoriales como parte de la plataforma:

- [Documentación](https://docs.microsoft.com/rest/api/maps/renderv2/getmaptilepreview) | [Detalles del formato de datos](https://developer.tomtom.com/maps-api/maps-api-documentation-vector/tile) de mosaicos de carreteras
- [Documentación](https://docs.microsoft.com/rest/api/maps/traffic/gettrafficincidenttile) | [Detalles del formato de datos](https://developer.tomtom.com/traffic-api/traffic-api-documentation-traffic-incidents/vector-incident-tiles) de incidentes de tráfico
- [Documentación](https://docs.microsoft.com/rest/api/maps/traffic/gettrafficflowtile) | [Detalles del formato de datos](https://developer.tomtom.com/traffic-api/traffic-api-documentation-traffic-flow/vector-flow-tiles) de flujo de tráfico
- El Creador de Azure Maps también permite crear mosaicos vectoriales personalizados y acceder a ellos a través de [Render V2 de obtención de mosaicos](https://docs.microsoft.com/rest/api/maps/renderv2/getmaptilepreview).

> [!TIP]
> Al usar mosaicos de imagen vectoriales o de trama del servicio de representación de Azure Maps con el SDK web, puede reemplazar `atlas.microsoft.com` por el marcador de posición `{azMapsDomain}`. Este marcador de posición se reemplazará por el mismo dominio usado por el mapa y también anexará automáticamente los mismos detalles de autenticación. Esto simplifica considerablemente la autenticación con el servicio de representación al usar la autenticación de Azure Active Directory.

Para mostrar los datos de un origen de mosaico vectorial en el mapa, conecte el origen a una de las capas de representación de datos. Todas las capas que usan un origen vectorial deben especificar un valor `sourceLayer` en las opciones. El código siguiente carga el servicio de mosaico vectorial de flujo de tráfico de Azure Maps como un origen de mosaico vectorial y, a continuación, lo muestra en un mapa mediante una capa de línea. Este origen de mosaico vectorial tiene un único conjunto de datos en la capa de origen denominado "Traffic flow". Los datos de línea de este conjunto de datos tienen una propiedad denominada `traffic_level` que se usa en este código para seleccionar el color y escalar el tamaño de las líneas.

```javascript
//Create a vector tile source and add it to the map.
var datasource = new atlas.source.VectorTileSource(null, {
    tiles: ['https://{azMapsDomain}/traffic/flow/tile/pbf?api-version=1.0&style=relative&zoom={z}&x={x}&y={y}'],
    maxZoom: 22
});
map.sources.add(datasource);

//Create a layer for traffic flow lines.
var flowLayer = new atlas.layer.LineLayer(datasource, null, {
    //The name of the data layer within the data source to pass into this rendering layer.
    sourceLayer: 'Traffic flow',

    //Color the roads based on the traffic_level property. 
    strokeColor: [
        'interpolate',
        ['linear'],
        ['get', 'traffic_level'],
        0, 'red',
        0.33, 'orange',
        0.66, 'green'
    ],

    //Scale the width of roads based on the traffic_level property. 
    strokeWidth: [
        'interpolate',
        ['linear'],
        ['get', 'traffic_level'],
        0, 6,
        1, 1
    ]
});

//Add the traffic flow layer below the labels to make the map clearer.
map.layers.add(flowLayer, 'labels');
```

<br/>

<iframe height="500" style="width: 100%;" scrolling="no" title="Capa de línea de mosaico vectorial" src="https://codepen.io/azuremaps/embed/wvMXJYJ?height=500&theme-id=default&default-tab=js,result&editable=true" frameborder="no" allowtransparency="true" allowfullscreen="true">
Consulte el Pen <a href='https://codepen.io/azuremaps/pen/wvMXJYJ'>Capa de línea de mosaico vectorial</a> de Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) en <a href='https://codepen.io'>CodePen</a>.
</iframe>

<br/>

## <a name="connecting-a-data-source-to-a-layer"></a>Conexión de un origen de datos a una capa

Los datos se representan en el mapa mediante las capas de representación. Una o varias capas de representación pueden hacer referencia a un único origen de datos. Las siguientes capas de representación requieren un origen de datos:

- [Capa de burbuja](map-add-bubble-layer.md): representa los datos de puntos como círculos a escala en el mapa.
- [Capa de símbolo](map-add-pin.md): representa los datos de punto como iconos o texto.
- [Capa de mapa térmico](map-add-heat-map-layer.md): representa datos de puntos como un mapa térmico de densidad.
- [Capa de línea](map-add-shape.md): representa una línea o el contorno de los polígonos. 
- [Capa de polígono](map-add-shape.md): rellena el área de un polígono con un patrón de imagen o un color sólido.

En el código siguiente se muestra cómo crear un origen de datos, agregarlo al mapa y conectarlo a una capa de burbuja. Luego, importe los datos de punto de GeoJSON desde una ubicación remota hasta el origen de datos. 

```javascript
//Create a data source and add it to the map.
var datasource = new atlas.source.DataSource();
map.sources.add(datasource);

//Create a layer that defines how to render points in the data source and add it to the map.
map.layers.add(new atlas.layer.BubbleLayer(datasource));

//Load the earthquake data.
datasource.importDataFromUrl('https://earthquake.usgs.gov/earthquakes/feed/v1.0/summary/significant_month.geojson');
```

Hay capas de representación adicionales que no se conectan a estos orígenes de datos, sino que cargan los datos directamente para representarlos. 

- [Capa de imagen](map-add-image-layer.md): superpone una sola imagen en la parte superior del mapa y enlaza sus esquinas a un conjunto de coordenadas especificadas.
- [Capa de mosaico](map-add-tile-layer.md): superpone una capa de mosaico de trama en la parte superior del mapa.

## <a name="one-data-source-with-multiple-layers"></a>Un origen de datos con varias capas

Varias capas pueden estar conectadas a un solo origen de datos. Hay muchos escenarios diferentes en los que esta opción es útil. Por ejemplo, considere aquel en el que un usuario dibuja un polígono. Debemos representar y rellenar el área del polígono a medida que el usuario agrega puntos al mapa. Agregar una línea con estilo para delinear el polígono permite ver más fácilmente sus bordes, a medida que el usuario dibuja. Para editar cómodamente una posición individual del polígono, podemos agregar un identificador, como una chincheta o una marca, encima de cada posición.

![Mapa que muestra varias capas que representan datos de un único origen de datos](media/create-data-source-web-sdk/multiple-layers-one-datasource.png)

En la mayoría de las plataformas de mapas, necesitaría un objeto de polígono, un objeto de línea y una chincheta para cada posición del polígono. A medida que le polígono se modifica, tendría que actualizar manualmente las líneas y las chinchetas, que pueden volverse complejas rápidamente.

Con Azure Maps, lo único que necesita es un solo polígono en un origen de datos, como se muestra en el siguiente código.

```javascript
//Create a data source and add it to the map.
var dataSource = new atlas.source.DataSource();
map.sources.add(dataSource);

//Create a polygon and add it to the data source.
dataSource.add(new atlas.data.Polygon([[[/* Coordinates for polygon */]]]));

//Create a polygon layer to render the filled in area of the polygon.
var polygonLayer = new atlas.layer.PolygonLayer(dataSource, 'myPolygonLayer', {
     fillColor: 'rgba(255,165,0,0.2)'
});

//Create a line layer for greater control of rendering the outline of the polygon.
var lineLayer = new atlas.layer.LineLayer(dataSource, 'myLineLayer', {
     color: 'orange',
     width: 2
});

//Create a bubble layer to render the vertices of the polygon as scaled circles.
var bubbleLayer = new atlas.layer.BubbleLayer(dataSource, 'myBubbleLayer', {
     color: 'orange',
     radius: 5,
     outlineColor: 'white',
     outlineWidth: 2
});

//Add all layers to the map.
map.layers.add([polygonLayer, lineLayer, bubbleLayer]);
```

## <a name="next-steps"></a>Pasos siguientes

Más información sobre las clases y los métodos utilizados en este artículo:

> [!div class="nextstepaction"]
> [DataSource](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.source.datasource)

> [!div class="nextstepaction"]
> [DataSourceOptions](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.datasourceoptions)

> [!div class="nextstepaction"]
> [VectorTileSource](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.source.vectortilesource)

> [!div class="nextstepaction"]
> [VectorTileSourceOptions](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.vectortilesourceoptions)

Para obtener más ejemplos de código para agregar a los mapas:

> [!div class="nextstepaction"]
> [Adición de un elemento emergente](map-add-popup.md)

> [!div class="nextstepaction"]
> [Uso de expresiones de estilo controladas por datos](data-driven-style-expressions-web-sdk.md)

> [!div class="nextstepaction"]
> [Adición de una capa de símbolo](map-add-pin.md)

> [!div class="nextstepaction"]
> [Adición de una capa de burbuja](map-add-bubble-layer.md)

> [!div class="nextstepaction"]
> [Adición de una capa de línea](map-add-line-layer.md)

> [!div class="nextstepaction"]
> [Adición de una capa de polígono](map-add-shape.md)

> [!div class="nextstepaction"]
> [Adición de un mapa térmico](map-add-heat-map-layer.md)

> [!div class="nextstepaction"]
> [Ejemplos de código](https://docs.microsoft.com/samples/browse/?products=azure-maps)