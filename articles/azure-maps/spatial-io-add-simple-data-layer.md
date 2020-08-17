---
title: Adición de una capa de datos simple | Microsoft Azure Maps
description: Obtenga información sobre cómo agregar una capa de datos simple mediante el módulo de E/S espacial, proporcionado por el SDK web de Azure Maps.
author: anastasia-ms
ms.author: v-stharr
ms.date: 02/29/2020
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: philmea
ms.custom: devx-track-javascript
ms.openlocfilehash: 704cb43a31ff1dfcaf287f9af467f7d6e7d6c50d
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87282813"
---
# <a name="add-a-simple-data-layer"></a>Adición de una capa de datos simple

El módulo de E/S espacial proporciona una clase `SimpleDataLayer`. Esta clase facilita la representación de características con estilo en el mapa. Incluso puede representar conjuntos de datos que tengan propiedades de estilo y conjuntos de datos que contengan tipos de geometría mixtos. La capa de datos simple consigue esta funcionalidad mediante el encapsulado de varias capas de representación y el uso de expresiones de estilo. Las expresiones de estilo buscan propiedades de estilo comunes de las características dentro de estas capas encapsuladas. La función `atlas.io.read` y la función `atlas.io.write` usan estas propiedades para leer y escribir estilos en un formato de archivo compatible. Después de agregar las propiedades a un formato de archivo compatible, el archivo se puede usar para varios propósitos. Por ejemplo, el archivo se puede usar para mostrar las características con estilo en el mapa.

Además de las características de estilo, `SimpleDataLayer` proporciona una característica emergente integrada con una plantilla emergente. El elemento emergente se muestra cuando se hace clic en una característica. Si lo desea, se puede deshabilitar la característica emergente predeterminada. Esta capa también admite datos en clúster. Cuando se hace clic en un clúster, el mapa se acerca al clúster y lo expande en puntos y subclústeres individuales.

La clase `SimpleDataLayer` está pensada para su uso con conjuntos de datos de gran tamaño con muchos tipos de geometría y muchos estilos aplicados en las características. Cuando se utiliza, esta clase agrega una sobrecarga de seis capas que contienen expresiones de estilo. Por lo tanto, hay casos en los que es más eficaz usar las capas de representación básicas. Por ejemplo, se puede usar una capa básica para representar un par de tipos de geometría y algunos estilos en una característica.

## <a name="use-a-simple-data-layer"></a>Uso de una capa de datos simple

La clase `SimpleDataLayer` se usa como el resto de las capas de representación. En el código siguiente se muestra cómo usar una capa de datos simple en un mapa:

```javascript
//Create a data source and add it to the map.
var datasource = new atlas.source.DataSource();
map.sources.add(datasource);

//Add a simple data layer for rendering data.
var layer = new atlas.layer.SimpleDataLayer(datasource);
map.layers.add(layer);
```

Agregue características al origen de datos. A continuación, la capa de datos simple determinará la mejor manera de representar las características. Los estilos de las características individuales se pueden establecer como propiedades en la característica. En el código siguiente se muestra una característica de punto GeoJSON con la propiedad `color` establecida en `red`. 

```json
{
    "type": "Feature",
    "geometry": {
        "type": "Point",
        "coordinates": [0, 0]
    },
    "properties": {
        "color": "red"
    }
}
```

El código siguiente representa la característica de punto anterior mediante la capa de datos simple. 

<br/>

<iframe height="500" style="width: 100%;" scrolling="no" title="Uso de la capa de datos simple" src="//codepen.io/azuremaps/embed/zYGzpQV/?height=500&theme-id=0&default-tab=js,result&editable=true" frameborder="no" allowtransparency="true" allowfullscreen="true"> Consulte el fragmento de código <a href='https://codepen.io/azuremaps/pen/zYGzpQV/'>Uso de la capa de datos simple</a> de Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) en <a href='https://codepen.io'>CodePen</a>.
</iframe>

La eficacia real de la capa de datos simple se aprecia cuando:

- Hay varios tipos diferentes de características en un origen de datos o
- Las características del conjunto de datos tienen varias propiedades de estilo establecidas individualmente o
- No está seguro de lo que contiene exactamente el conjunto de datos.

Por ejemplo, al analizar fuentes de distribución de datos XML, es posible que no conozca los estilos exactos y los tipos de geometría de las características. En el ejemplo siguiente se muestra la eficacia de la capa de datos simple mediante la representación de las características de un archivo KML. También se muestran varias de las opciones que proporciona la clase de capa de datos simple.

<br/>

<iframe height="700" style="width: 100%;" scrolling="no" title="Opciones de la capa de datos simple" src="//codepen.io/azuremaps/embed/gOpRXgy/?height=700&theme-id=0&default-tab=result" frameborder="no" allowtransparency="true" allowfullscreen="true"> Consulte el fragmento de código <a href='https://codepen.io/azuremaps/pen/gOpRXgy/'>Opciones de la cada de datos simple</a> de Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) en <a href='https://codepen.io'>CodePen</a>.
</iframe>


> [!NOTE]
> Este nivel de datos simple usa la clase [plantilla emergente](map-add-popup.md#add-popup-templates-to-the-map) para mostrar globos de KML o propiedades de características en forma de tabla. De forma predeterminada, todo el contenido representado en el elemento emergente estará en un espacio aislado dentro de un iframe como característica de seguridad. Sin embargo, existen limitaciones:
>
> - Todos los scripts, los formularios y las funcionalidades de bloqueo de puntero y de navegación superior están deshabilitados. Al hacer clic en los vínculos, se pueden abrir en una pestaña nueva. 
> - Los exploradores más antiguos que no admitan el parámetro `srcdoc` en iframes se limitarán a representar una pequeña cantidad de contenido.
> 
> Si confía en los datos que se van a cargar en los elementos emergentes y, potencialmente, quiere que estos scripts cargados en los elementos emergentes puedan acceder a su aplicación, puede deshabilitarlo estableciendo la opción `sandboxContent` de las plantillas emergentes en false. 

## <a name="default-supported-style-properties"></a>Propiedades de estilo predeterminadas admitidas

Como se mencionó anteriormente, la capa de datos simple encapsula varias de las capas de representación básicas: burbuja, símbolo, línea, polígono y polígono extruido. A continuación, utiliza expresiones para buscar propiedades de estilo válidas en características individuales.

Las propiedades de estilo de Azure Maps y GitHub son los dos conjuntos principales de nombres de propiedad admitidos. La mayoría de los nombres de propiedades de las diferentes opciones de capa de mapa de Azure se admiten como propiedades de estilo de las características de la capa de datos simple. Se han agregado expresiones a algunas opciones de capa para admitir nombres de propiedades de estilo utilizadas habitualmente en GitHub. Estos nombres de propiedades se definen en [Compatibilidad de mapas GeoJSON de GitHub](https://help.github.com/en/github/managing-files-in-a-repository/mapping-geojson-files-on-github) y se usan para aplicar estilo a los archivos GeoJSON que se almacenan y representan en la plataforma. Se admiten todas las propiedades de estilo de GitHub en la capa de datos simple, excepto en las propiedades de estilo `marker-symbol`.

Si el lector se encuentra con una propiedad de estilo menos habitual, la convertirá en la propiedad de estilo de Azure Maps más parecida. Además, las expresiones de estilo predeterminadas se pueden invalidar mediante el uso de la función `getLayers` de la capa de datos simple y la actualización de las opciones de cualquiera de las capas.

En las siguientes secciones se proporcionan detalles sobre las propiedades de estilo predeterminadas que admite la capa de datos simple. El orden del nombre de propiedad admitido también es la prioridad de la propiedad. Si se definen dos propiedades de estilo para la misma opción de capa, la primera de la lista tendrá mayor precedencia. Los colores pueden ser cualquier valor de color CSS3; como HEX, RGB, RGBA, HSL, HSLA o el valor de color con nombre.

### <a name="bubble-layer-style-properties"></a>Propiedades de estilo de la capa de burbujas

Si una característica es un elemento `Point` o un elemento `MultiPoint` y la característica no tiene una propiedad `image` que se pueda usar como icono personalizado para representar el punto como un símbolo, la característica se representará con un elemento `BubbleLayer`.

| Opción de capa | Nombres de propiedades admitidos | Valor predeterminado |
|--------------|----------------------------|---------------|
| `color` | `color`, `marker-color` | `'#1A73AA'` |
| `radius` | `size`<sup>1</sup>, `marker-size`<sup>2</sup>, `scale`<sup>1</sup> | `8` |
| `strokeColor` | `strokeColor`, `stroke` | `'#FFFFFF'` |

\[1\] Los valores de `size` y `scale` se consideran valores escalares y se multiplicarán por `8`.

\[2\] Si se especifica la opción `marker-size` de GitHub, se usarán los siguientes valores para el radio.

| Tamaño de marcador | Radio |
|-------------|--------|
| `small`     | `6`    |
| `medium`    | `8`    |
| `large`     | `12`   |

Los clústeres también se representan mediante la capa de burbujas. De forma predeterminada, el radio de un clúster se establece en `16`. El color del clúster varía en función del número de puntos del clúster, como se define a continuación:

| Número de puntos | Color    |
|-------------|----------|
| &gt;= 100   | `red`    |
| &gt;= 10    | `yellow` |
| &lt; 10     | `green`  |

### <a name="symbol-style-properties"></a>Propiedades de estilo de símbolo

Si una característica es un elemento `Point` o un elemento `MultiPoint` y la característica tiene una propiedad `image` que se pueda usar como icono personalizado para representar el punto como un símbolo, la característica se representará con un elemento `SymbolLayer`.

| Opción de capa | Nombres de propiedades admitidos | Valor predeterminado |
|--------------|----------------------------|---------------|
| `image` | `image` | ``none`` |
| `size` | `size`, `marker-size`<sup>1</sup> | `1` |
| `rotation` | `rotation` | `0` |
| `offset` | `offset` | `[0, 0]` |
| `anchor` | `anchor` | `'bottom'` |

\[1\] Si se especifica la opción `marker-size` de GitHub, se usarán los siguientes valores para la opción de tamaño de icono.

| Tamaño de marcador | Tamaño del símbolo |
|-------------|-------------|
| `small`     | `0.5`       |
| `medium`    | `1`         |
| `large`     | `2`         |

Si la característica de punto es un clúster, la propiedad `point_count_abbreviated` se representará como una etiqueta de texto. No se representará ninguna imagen.

### <a name="line-style-properties"></a>Propiedades de estilo de línea

Si la característica es un elemento `LineString`, `MultiLineString`, `Polygon` o `MultiPolygon`, la característica se representará con un elemento `LineLayer`.

| Opción de capa | Nombres de propiedades admitidos | Valor predeterminado |
|--------------|----------------------------|---------------|
| `strokeColor` | `strokeColor`, `stroke` | `'#1E90FF'` |
| `strokeWidth` | `strokeWidth`, `stroke-width`, `stroke-thickness` | `3` |
| `strokeOpacity` | `strokeOpacity`, `stroke-opacity` | `1` |

### <a name="polygon-style-properties"></a>Propiedades de estilo de polígono

Si la característica es un elemento `Polygon` o un elemento `MultiPolygon` y la característica no tiene una propiedad `height` o la propiedad `height` es cero, la característica se representará con un elemento `PolygonLayer`.

| Opción de capa | Nombres de propiedades admitidos | Valor predeterminado |
|--------------|----------------------------|---------------|
| `fillColor` | `fillColor`, `fill` | `'#1E90FF'` |
| `fillOpacity` | `fillOpacity`, `fill-opacity` | `0.5` |

### <a name="extruded-polygon-style-properties"></a>Propiedades de estilo de polígono extruido

Si la característica es un elemento `Polygon` o un elemento `MultiPolygon` y tiene una propiedad `height` con un valor mayor que 0, la característica se representará con un elemento `PolygonExtrusionLayer`.

| Opción de capa | Nombres de propiedades admitidos | Valor predeterminado |
|--------------|----------------------------|---------------|
| `base` | `base` | `0` |
| `fillColor` | `fillColor`, `fill` | `'#1E90FF'` |
| `height` | `height` | `0` |

## <a name="next-steps"></a>Pasos siguientes

Más información sobre las clases y los métodos utilizados en este artículo:

> [!div class="nextstepaction"]
> [SimpleDataLayer](https://docs.microsoft.com/javascript/api/azure-maps-spatial-io/atlas.layer.simpledatalayer)

> [!div class="nextstepaction"]
> [SimpleDataLayerOptions](https://docs.microsoft.com/javascript/api/azure-maps-spatial-io/atlas.simpledatalayeroptions)

Para obtener más ejemplos de código para agregar a los mapas:

> [!div class="nextstepaction"]
> [Lectura y escritura de datos espaciales](spatial-io-read-write-spatial-data.md)

> [!div class="nextstepaction"]
> [Adición de una capa de mapa de OGC](spatial-io-add-ogc-map-layer.md)

> [!div class="nextstepaction"]
> [Conexión a un servicio WFS](spatial-io-connect-wfs-service.md)

> [!div class="nextstepaction"]
> [Aprovechamiento de las operaciones básicas](spatial-io-core-operations.md)

> [!div class="nextstepaction"]
> [Detalles de formatos de datos admitidos](spatial-io-supported-data-format-details.md)
