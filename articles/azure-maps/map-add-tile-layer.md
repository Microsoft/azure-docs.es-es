---
title: Adición de una capa de mosaico a un mapa | Microsoft Azure Maps
description: Aprenda a superponer imágenes en mapas. Vea un ejemplo en el que se usa el SDK web de Azure Maps para agregar una capa de mosaico que contenga una superposición de radar meteorológico a un mapa.
author: rbrundritt
ms.author: richbrun
ms.date: 07/29/2019
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: ''
ms.custom: codepen, devx-track-javascript
ms.openlocfilehash: c96916c029c9c49ad7d4c493fed8859354508ebe
ms.sourcegitcommit: 98854e3bd1ab04ce42816cae1892ed0caeedf461
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "88010181"
---
# <a name="add-a-tile-layer-to-a-map"></a>Adición de una capa de mosaico a un mapa

En este artículo se explica cómo puede superponer una capa de mosaico en el mapa. Las capas de mosaico permiten superponer imágenes encima de los mosaicos de mapa base de Azure Maps. Para más información sobre el sistema de mosaicos de Azure Maps, consulte [Niveles de zoom y cuadrícula de mosaico](zoom-levels-and-tile-grid.md).

Una capa de mosaico carga los mosaicos desde un servidor. Estas imágenes se pueden representar previamente o se pueden representar de forma dinámica. Las imágenes representadas previamente se almacenan como cualquier otra imagen en un servidor mediante una convención de nomenclatura que entienda la capa de mosaico. Las imágenes representadas de forma dinámica usan un servicio para cargar las imágenes casi en tiempo real. Hay tres convenciones diferentes de nomenclatura de servicio de mosaico compatibles con la clase [TileLayer](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.layer.tilelayer?view=azure-iot-typescript-latest) de Azure Maps: 

* X, Y, notación Zoom: X es la columna, Y es la posición de fila del mosaico en la cuadrícula de mosaico y la notación Zoom es un valor basado en el nivel de zoom.
* Notación Quadkey: combina la información de zoom, x e y en un valor de cadena único. Este valor de cadena se convierte en un identificador único para un solo mosaico.
* Cuadro de límite: especifique una imagen en el formato de coordenadas del cuadro de límite: `{west},{south},{east},{north}`. Este formato se usa normalmente en [Web Mapping Services (WMS)](https://www.opengeospatial.org/standards/wms).

> [!TIP]
> Un elemento [TileLayer](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.layer.tilelayer?view=azure-iot-typescript-latest) es una excelente manera de visualizar grandes conjuntos de datos en el mapa. No solo puede generarse una capa de mosaico a partir de una imagen, sino que también se pueden representar datos de vector como una capa de mosaico. Con la representación de datos de vectores como una capa de mosaico, el control de mapa solo necesita cargar los mosaicos que tienen un tamaño de archivo bastante más reducido que los datos de vector que representan. Esta técnica se usa normalmente para representar millones de filas de datos en el mapa.

La dirección URL del mosaico pasada a una capa de mosaico debe ser una dirección URL HTTP/HTTPS que apunte a un recurso TileJSON o a una plantilla de URL de mosaico que use los siguientes parámetros: 

* `{x}`: posición del mosaico en X. También necesita `{y}` y `{z}`.
* `{y}`: posición del mosaico en Y. También necesita `{x}` y `{z}`.
* `{z}`: nivel de zoom del mosaico. También necesita `{x}` y `{y}`.
* `{quadkey}`: identificador quadkey de mosaico basado en la convención de nomenclatura del sistema de mosaico de Bing Maps.
* `{bbox-epsg-3857}`: una cadena de un cuadro delimitador con el formato `{west},{south},{east},{north}` en el sistema de referencia espacial EPSG 3857.
* `{subdomain}`: si se especifica `subdomain`, se agregará un marcador de posición para los valores de subdominio.
* `{azMapsDomain}`: un marcador de posición para alinear el dominio y la autenticación de las solicitudes de mosaico con los mismos valores utilizados por el mapa.

## <a name="add-a-tile-layer"></a>Adición de una capa de icono

 En este ejemplo se muestra cómo crear una capa de mosaico que señale a un conjunto de mosaicos. En este ejemplo se usa el sistema de mosaicos de zoom, x e y. La fuente de esta capa de mosaico es una superposición de radar meteorológico de [Iowa Environmental Mesonet of Iowa State University](https://mesonet.agron.iastate.edu/ogc/). Al examinar los datos de radar, idealmente los usuarios verán claramente las etiquetas de las ciudades mientras se desplazan por el mapa. Este comportamiento se puede implementar mediante la inserción de una capa de mosaico debajo de la capa `labels`.

```javascript
//Create a tile layer and add it to the map below the label layer.
//Weather radar tiles from Iowa Environmental Mesonet of Iowa State University.
map.layers.add(new atlas.layer.TileLayer({
    tileUrl: 'https://mesonet.agron.iastate.edu/cache/tile.py/1.0.0/nexrad-n0q-900913/{z}/{x}/{y}.png',
    opacity: 0.8,
    tileSize: 256
}), 'labels');
```

A continuación se muestra el código de ejemplo de ejecución completo de la funcionalidad anterior.

<br/>

<iframe height='500' scrolling='no' title='Capa de mosaico que usa X, Y y Z' src='//codepen.io/azuremaps/embed/BGEQjG/?height=500&theme-id=0&default-tab=js,result&embed-version=2&editable=true' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>Consulte el Pen <a href='https://codepen.io/azuremaps/pen/BGEQjG/'>capa de mosaico que usa X, Y y Z</a> de Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) en <a href='https://codepen.io'>CodePen</a>.
</iframe>

## <a name="customize-a-tile-layer"></a>Personalizar una capa de mosaico

La clase de la capa de mosaico tiene muchas opciones de estilo. Esta es una herramienta para probarlas.

<br/>

<iframe height='700' scrolling='no' title='Opciones de capa de mosaico' src='//codepen.io/azuremaps/embed/xQeRWX/?height=700&theme-id=0&default-tab=result' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>Consulte el Pen <a href='https://codepen.io/azuremaps/pen/xQeRWX/'>opciones de capa de mosaico</a> de Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) en <a href='https://codepen.io'>CodePen</a>.
</iframe>

## <a name="next-steps"></a>Pasos siguientes

Más información sobre las clases y los métodos utilizados en este artículo:

> [!div class="nextstepaction"]
> [TileLayer](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.layer.tilelayer?view=azure-iot-typescript-latest)

> [!div class="nextstepaction"]
> [TileLayerOptions](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.tilelayeroptions?view=azure-iot-typescript-latest)

Para obtener más ejemplos de código para agregar a los mapas:

> [!div class="nextstepaction"]
> [Adición de una capa de imagen](./map-add-image-layer.md)
