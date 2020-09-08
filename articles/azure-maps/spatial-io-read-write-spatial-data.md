---
title: Lectura y escritura de datos espaciales | Microsoft Azure Maps
description: Obtenga información sobre cómo leer y escribir datos mediante el módulo de E/S espacial, proporcionado por el SDK web de Azure Maps.
author: anastasia-ms
ms.author: v-stharr
ms.date: 03/01/2020
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: philmea
ms.custom: devx-track-javascript
ms.openlocfilehash: b094f63c075bdb8af225ff366343c60bc6818224
ms.sourcegitcommit: d39f2cd3e0b917b351046112ef1b8dc240a47a4f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88816769"
---
# <a name="read-and-write-spatial-data"></a>Lectura y escritura de datos espaciales

En la tabla siguiente se enumeran los formatos de archivo espacial que se admiten para las operaciones de lectura y escritura con el módulo de E/S espacial.

| Formato de datos       | Lectura | Escritura |
|-------------------|------|-------|
| GeoJSON           | ✓  |  ✓  |
| GeoRSS            | ✓  |  ✓  |
| GML               | ✓  |  ✓  |
| GPX               | ✓  |  ✓  |
| KML               | ✓  |  ✓  |
| KMZ               | ✓  |  ✓  |
| CSV espacial       | ✓  |  ✓  |
| Well-Known Text (texto conocido)   | ✓  |  ✓  |

En estas secciones se describen todas las distintas herramientas para leer y escribir datos espaciales mediante el módulo de E/S espacial.

## <a name="read-spatial-data"></a>Lectura de datos espaciales

`atlas.io.read` es la función principal que se usa para leer formatos de datos espaciales comunes, como KML, GPX, GeoRSS, GeoJSON y archivos CSV con datos espaciales. Esta función también puede leer versiones comprimidas de estos formatos, como un archivo ZIP o un archivo KMZ. El formato de archivo KMZ es una versión comprimida de KML que también puede incluir recursos como imágenes. Como alternativa, la función de lectura puede tomar una dirección URL que apunte a un archivo en cualquiera de estos formatos. Las direcciones URL deben estar hospedadas en un punto de conexión habilitado para CORS o se debe proporcionar un servicio de proxy en las opciones de lectura. El servicio de proxy se usa para cargar recursos en dominios que no están habilitados para CORS. La función de lectura devuelve una promesa para agregar los iconos de imagen al mapa y procesa los datos de forma asincrónica para minimizar el impacto en el subproceso de la interfaz de usuario.

Al leer un archivo comprimido, ya sea como un archivo ZIP o KMZ, se descomprime y se examina para si contiene el primer archivo válido. Por ejemplo, doc.kml o un archivo con otra extensión válida, como .kml, .xml, geojson, .json, .csv, .tsv o .txt. A continuación, se cargan previamente las imágenes a las que se hace referencia en los archivos KML y GeoRSS para asegurarse de que están accesibles. Los datos de imagen inaccesibles pueden cargar una imagen de reserva alternativa o se quitarán de los estilos. Las imágenes extraídas de archivos KMZ se convertirán en URI de datos.

El resultado de la función de lectura es un objeto `SpatialDataSet`. Este objeto extiende la clase FeatureCollection de GeoJSON. Se puede pasar fácilmente a un `DataSource` tal cual para representar sus características en un mapa. El `SpatialDataSet` no solo contiene información de características, sino que también puede incluir superposiciones de suelo KML, métricas de procesamiento y otros detalles, como se describe en la tabla siguiente.

| Nombre de propiedad | Tipo | Descripción | 
|---------------|------|-------------|
| `bbox` | `BoundingBox` | Cuadro de límite de todos los datos del conjunto de datos. |
| `features` | `Feature[]` | Características de GeoJSON en el conjunto de datos. |
| `groundOverlays` | `(atlas.layer.ImageLayer | atlas.layers.OgcMapLayer)[]` | Una matriz de KML GroundOverlays. |
| `icons` | Registro&lt;cadena, cadena&gt; | Un conjunto de direcciones URL de icono. Clave = nombre del icono, Valor = dirección URL. |
| properties | cualquiera | Información de propiedad proporcionada en el nivel de documento de un conjunto de datos espaciales. |
| `stats` | `SpatialDataSetStats` | Estadísticas sobre el contenido y el tiempo de procesamiento de un conjunto de datos espaciales. |
| `type` | `'FeatureCollection'` | Valor de tipo GeoJSON de solo lectura. |

## <a name="examples-of-reading-spatial-data"></a>Ejemplos de lectura de datos espaciales

En el código siguiente se muestra cómo leer un conjunto de datos espaciales y representarlo en el mapa mediante la clase `SimpleDataLayer`. El código usa un archivo GPX al que una dirección URL apunta.

<br/>

<iframe height='500' scrolling='no' title='Carga de datos espaciales simples' src='//codepen.io/azuremaps/embed/yLNXrZx/?height=500&theme-id=0&default-tab=js,result&embed-version=2&editable=true' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>Vea el lápiz <a href='https://codepen.io/azuremaps/pen/yLNXrZx/'>Carga de datos espaciales sencillos</a> de Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) en <a href='https://codepen.io'>CodePen</a>.
</iframe>

En la demostración de código siguiente se muestra cómo leer y cargar KML o KMZ en el mapa. KML puede contener superposiciones de suelo, que tendrán el formato de `ImageLyaer` o `OgcMapLayer`. Estas superposiciones deben agregarse en el mapa por separado de las características. Además, si el conjunto de datos tiene iconos personalizados, dichos iconos deben cargarse en los recursos de mapas antes de que se carguen las características.

<br/>

<iframe height='500' scrolling='no' title='Carga de KML en el mapa' src='//codepen.io/azuremaps/embed/XWbgwxX/?height=500&theme-id=0&default-tab=js,result&embed-version=2&editable=true' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>Vea el lápiz de <a href='https://codepen.io/azuremaps/pen/XWbgwxX/'>carga de KML en el mapa</a> de Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) en <a href='https://codepen.io'>CodePen</a>.
</iframe>

Si quiere, puede proporcionar un servicio de proxy para acceder a los recursos entre dominios que pueden no tener CORS habilitado. La función de lectura intentará obtener acceso a los archivos de otro dominio primero con CORS. La primera vez que no pueda acceder a ningún recurso de otro dominio con CORS, solo solicitará archivos adicionales en el caso de que se haya proporcionado un servicio de proxy. La función read anexa la dirección URL del archivo al final de la dirección URL del proxy proporcionada. En este fragmento de código se muestra cómo pasar un servicio de proxy a la función de lectura:

```javascript
//Read a file from a URL or pass in a raw data as a string.
atlas.io.read('https://nonCorsDomain.example.com/mySuperCoolData.xml', {
    //Provide a proxy service
    proxyService: window.location.origin + '/YourCorsEnabledProxyService.ashx?url='
}).then(async r => {
    if (r) {
        // Some code goes here . . .
    }
});

```

En la siguiente demostración se muestra cómo leer un archivo delimitado y representarlo en el mapa. En este caso, el código usa un archivo CSV que tiene columnas de datos espaciales.

<br/>

<iframe height='500' scrolling='no' title='Incorporación de un archivo delimitado' src='//codepen.io/azuremaps/embed/ExjXBEb/?height=500&theme-id=0&default-tab=js,result&embed-version=2&editable=true' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>Vea el lápiz para <a href='https://codepen.io/azuremaps/pen/ExjXBEb/'>agregar un archivo delimitado</a> de Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) en <a href='https://codepen.io'>CodePen</a>.
</iframe>

## <a name="write-spatial-data"></a>Escritura de datos espaciales

Hay dos funciones de escritura principales en el módulo de E/S espacial. La función `atlas.io.write` genera una cadena, mientras que la función `atlas.io.writeCompressed` genera un archivo ZIP comprimido. El archivo ZIP comprimido contendría un archivo basado en texto con los datos espaciales. Ambas funciones devuelven una promesa para agregar los datos al archivo. Además, ambos pueden escribir cualquiera de los siguientes datos: `SpatialDataSet`, `DataSource`, `ImageLayer`, `OgcMapLayer`, colección de características, característica, geometría o una matriz de cualquier combinación de estos tipos de datos. Al escribir con cualquiera de las funciones, puede especificar el formato de archivo deseado. Si no se especifica el formato de archivo, los datos se escribirán como KML.

La herramienta siguiente muestra la mayoría de las opciones de escritura que se pueden usar con la función `atlas.io.write`.

<br/>

<iframe height='700' scrolling='no' title='Opciones de escritura de datos espaciales' src='//codepen.io/azuremaps/embed/YzXxXPG/?height=700&theme-id=0&default-tab=result&embed-version=2&editable=true' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>Vea el lápiz de <a href='https://codepen.io/azuremaps/pen/YzXxXPG/'>Opciones de escritura de datos espaciales</a> de Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) en <a href='https://codepen.io'>CodePen</a>.
</iframe>

## <a name="example-of-writing-spatial-data"></a>Ejemplo de escritura de datos espaciales

El ejemplo siguiente le permite arrastrar y colocar, y, a continuación, cargar los archivos espaciales en el mapa. Puede exportar los datos de GeoJSON desde el mapa y escribirlos en uno de los formatos de datos espaciales admitidos, por ejemplo, una cadena o un archivo comprimido.

<br/>

<iframe height='700' scrolling='no' title='Arrastrar y colocar archivos espaciales en un mapa' src='//codepen.io/azuremaps/embed/zYGdGoO/?height=700&theme-id=0&default-tab=result&embed-version=2&editable=true' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>Vea el lápiz <a href='https://codepen.io/azuremaps/pen/zYGdGoO/'>Arrastrar y colocar los archivos espaciales en el mapa</a> de Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) en <a href='https://codepen.io'>CodePen</a>.
</iframe>

Si quiere, puede proporcionar un servicio de proxy para acceder a los recursos entre dominios que pueden no tener CORS habilitado. Este fragmento de código muestra que puede incorporar un servicio de proxy:

```javascript
atlas.io.read(data, {
    //Provide a proxy service
    proxyService: window.location.origin + '/YourCorsEnabledProxyService.ashx?url='
}).then(
    //Success
    function(r) {
        //some code goes here ...
    }
);
```

## <a name="read-and-write-well-known-text-wkt"></a>Lectura y escritura de Well-Known Text (WKT)

[Well-Known Text](https://en.wikipedia.org/wiki/Well-known_text_representation_of_geometry) (WKT) es un estándar de Open Geospatial Consortium (OGC) para representar geometrías espaciales en forma de texto. Muchos sistemas geoespaciales admiten WKT, como Azure SQL y Azure PostgreSQL con el complemento PostGIS. Como la mayoría de los estándares de OGC, las coordenadas tienen el formato de "longitud latitud" para alinearse con la convención "x y". Por ejemplo, un punto de longitud -110 y latitud 45 se puede escribir como `POINT(-110 45)` en el formato WKT.

El texto WKT se puede leer con la función `atlas.io.ogc.WKT.read` y escribir con la función `atlas.io.ogc.WKT.write`.

## <a name="examples-of-reading-and-writing-well-known-text-wkt"></a>Ejemplos de lectura y escritura de Well-Known Text (WKT)

En el código siguiente se muestra cómo leer la cadena de texto conocido WKT `POINT(-122.34009 47.60995)` y representarla en el mapa mediante una capa de burbujas.

<br/>

<iframe height='500' scrolling='no' title='Lectura de Well-Known Text' src='//codepen.io/azuremaps/embed/XWbabLd/?height=500&theme-id=0&default-tab=result&embed-version=2&editable=true' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>Vea el lápiz <a href='https://codepen.io/azuremaps/pen/XWbabLd/'>Lectura de Well-Known Text</a> de Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) en <a href='https://codepen.io'>CodePen</a>.
</iframe>

En el código siguiente se muestra cómo leer y escribir texto conocido hacia atrás y hacia delante.

<br/>

<iframe height='700' scrolling='no' title='Lectura y escritura de WKT' src='//codepen.io/azuremaps/embed/JjdyYav/?height=700&theme-id=0&default-tab=result&embed-version=2&editable=true' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>Vea el lápiz <a href='https://codepen.io/azuremaps/pen/JjdyYav/'>Lectura y escritura de Well-Known Text</a> de Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) en <a href='https://codepen.io'>CodePen</a>.
</iframe>

## <a name="read-and-write-gml"></a>Lectura y escritura de GML

GML es una especificación de archivo XML espacial que se suele usar como extensión de otras especificaciones XML. Los datos de GeoJSON se pueden escribir como XML con etiquetas GML mediante la función `atlas.io.core.GmlWriter.write`. El XML que contiene GML se puede leer mediante la función `atlas.io.core.GmlReader.read`. La función de lectura tiene dos opciones:

- La opción `isAxisOrderLonLat`: el orden de los ejes de las coordenadas "latitud, longitud" o "longitud, latitud" puede variar entre los conjuntos de datos y no siempre está bien definido. De forma predeterminada, el lector GML lee los datos de coordenadas como "latitud, longitud" pero, si se establece esta opción en true, se leerá como "longitud, latitud".
- La opción `propertyTypes`: esta opción es una tabla de búsqueda de valores clave donde la clave es el nombre de una propiedad del conjunto de datos. El valor es el tipo de objeto al que se va a convertir el valor al analizar. Los valores de tipo admitidos son: `string`, `number`, `boolean` y `date`. Si una propiedad no está en la tabla de búsqueda o el tipo no está definido, la propiedad se analizará como una cadena.

La función `atlas.io.read` tomará como valor predeterminado la función `atlas.io.core.GmlReader.read` cuando detecte que los datos de entrada son XML, pero los datos no sean uno de los demás formatos XML espaciales admitidos.

`GmlReader` analizará las coordenadas que tenga uno de los siguientes SRID:

- EPSG:4326 (preferido)
- EPSG:4269, EPSG:4283, EPSG:4258, EPSG:4308, EPSG:4230, EPSG:4272, EPSG:4271, EPSG:4267, EPSG:4608, EPSG:4674 posiblemente con un pequeño margen de error.
- EPSG:3857, EPSG:102100, EPSG:3785, EPSG:900913, EPSG:102113, EPSG:41001, EPSG:54004

## <a name="next-steps"></a>Pasos siguientes

Más información sobre las clases y los métodos utilizados en este artículo:

> [!div class="nextstepaction"]
> [Funciones estáticas atlas.io](https://docs.microsoft.com/javascript/api/azure-maps-spatial-io/atlas.io)

> [!div class="nextstepaction"]
> [SpatialDataSet](https://docs.microsoft.com/javascript/api/azure-maps-spatial-io/atlas.spatialdataset)

> [!div class="nextstepaction"]
> [SpatialDataSetStats](https://docs.microsoft.com/javascript/api/azure-maps-spatial-io/atlas.spatialdatasetstats)

> [!div class="nextstepaction"]
> [GmlReader](https://docs.microsoft.com/javascript/api/azure-maps-spatial-io/atlas.io.core.gmlreader?view=azure-maps-typescript-latest)

> [!div class="nextstepaction"]
> [GmlWriter](https://docs.microsoft.com/javascript/api/azure-maps-spatial-io/atlas.io.core.gmlwriter?view=azure-maps-typescript-latest)

> [!div class="nextstepaction"]
> [Funciones atlas.io.ogc.WKT](https://docs.microsoft.com/javascript/api/azure-maps-spatial-io/atlas.io.ogc.wkt)

Para obtener más ejemplos de código para agregar a los mapas:

> [!div class="nextstepaction"]
> [Adición de una capa de mapa de OGC](spatial-io-add-ogc-map-layer.md)

> [!div class="nextstepaction"]
> [Conexión a un servicio WFS](spatial-io-connect-wfs-service.md)

> [!div class="nextstepaction"]
> [Aprovechamiento de las operaciones básicas](spatial-io-core-operations.md)

> [!div class="nextstepaction"]
> [Detalles de formatos de datos admitidos](spatial-io-supported-data-format-details.md)
