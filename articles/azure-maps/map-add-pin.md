---
title: Adición de una capa de símbolo a un mapa | Microsoft Azure Maps
description: Aprenda a agregar símbolos personalizados, como texto o iconos, a mapas. Para este fin, consulte cómo usar orígenes de datos y capas de símbolos en el SDK web de Azure Maps.
author: rbrundritt
ms.author: richbrun
ms.date: 07/29/2019
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: ''
ms.custom: codepen, devx-track-javascript
ms.openlocfilehash: 72ce95373df0a670179424d8e7ea95254941db1a
ms.sourcegitcommit: 07166a1ff8bd23f5e1c49d4fd12badbca5ebd19c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/15/2020
ms.locfileid: "90085933"
---
# <a name="add-a-symbol-layer-to-a-map"></a>Adición de una capa de símbolo a un mapa

Conecte un símbolo a un origen de datos y úselo para representar un icono o un texto en un punto determinado. 

Las capas de símbolo se representan mediante WebGL. Use una capa de símbolos para representar grandes colecciones de puntos en el mapa. En comparación con los marcadores HTML, la capa de símbolos representa un gran número de datos de puntos en el mapa con un rendimiento mejorado. Sin embargo, la capa de símbolos no es compatible con los elementos CSS y HTML tradicionales para aplicar estilos.  

> [!TIP]
> De forma predeterminada, las capas de símbolo representarán las coordenadas de todos los objetos geométricos en un origen de datos. Para limitar la capa de forma que solo represente las características de la geometría de puntos, establezca la propiedad `filter` de la capa en `['==', ['geometry-type'], 'Point']` o `['any', ['==', ['geometry-type'], 'Point'], ['==', ['geometry-type'], 'MultiPoint']]`. Si lo desea, también puede incluir características de MultiPoint.

El administrador de sprites de imágenes de mapas carga imágenes personalizadas que se utilizan en la capa de símbolos. Los formatos de imagen compatibles son los siguientes:

- JPEG
- PNG
- SVG
- BMP
- GIF (sin animaciones)

## <a name="add-a-symbol-layer"></a>Adición de una capa de símbolo

Para poder agregar una capa de símbolos al mapa, debe realizar primero algunas operaciones. En primer lugar, cree un origen de datos y agréguelo al mapa. Cree una capa de símbolos. Pase el origen de datos a la capa de símbolos para recuperar los datos de este origen. Por último, agregue datos al origen de datos para que haya algo que representar. 

En el código siguiente se muestra lo que debe agregarse al mapa una vez que se haya cargado. En este ejemplo se representa un único punto en el mapa mediante una capa de símbolos. 

```javascript
//Create a data source and add it to the map.
var dataSource = new atlas.source.DataSource();
map.sources.add(dataSource);

//Create a symbol layer to render icons and/or text at points on the map.
var layer = new atlas.layer.SymbolLayer(dataSource);

//Add the layer to the map.
map.layers.add(layer);

//Create a point and add it to the data source.
dataSource.add(new atlas.data.Point([0, 0]));
```

Hay cuatro tipos diferentes de datos de puntos que se pueden agregar al mapa:

- Geometría Point de GeoJSON: Este objeto solo contiene una coordenada de un punto y nada más. La clase auxiliar `atlas.data.Point` se puede usar para crear fácilmente estos objetos.
- Geometría MultiPoint de GeoJSON: este objeto solamente contiene las coordenadas de varios puntos; no contiene nada más. La clase auxiliar `atlas.data.MultiPoint` se puede usar para crear fácilmente estos objetos.
- GeoJSON Feature: Este objeto se compone de cualquier geometría GeoJSON y un conjunto de propiedades que contienen metadatos asociados a la geometría. La clase auxiliar `atlas.data.Feature` se puede usar para crear fácilmente estos objetos.
- La clase `atlas.Shape` es similar a la característica GeoJSON. Las dos se componen de una geometría GeoJSON y de un conjunto de propiedades que contienen metadatos relacionados con la geometría. Si se agrega un objeto GeoJSON a un origen de datos, este podrá representarse fácilmente en una capa. Sin embargo, si se actualiza la propiedad de coordenadas de ese objeto GeoJSON, el origen de datos y el mapa no cambian. Esto se debe a que no hay ningún mecanismo en el objeto JSON que desencadene una actualización. La clase Shape dispone de funciones para actualizar los datos que contiene. Cuando se realiza un cambio, automáticamente se actualiza el origen de datos y el mapa, y se envían notificaciones. 

En el ejemplo de código siguiente se crea una geometría Point de GeoJSON y se pasa a la clase `atlas.Shape` para facilitar su actualización. En un principio, se usa el centro del mapa para representar un símbolo. Después, se agrega un evento de clic al mapa para que, cuando se active, las coordenadas del mouse se utilicen con la función `setCoordinates` de las formas. Las coordenadas del mouse se registran en el momento en que se produce el evento de clic. A continuación, `setCoordinates` actualiza la ubicación del símbolo en el mapa.

<br/>

<iframe height='500' scrolling='no' title='Cambio del anclaje de ubicación' src='//codepen.io/azuremaps/embed/ZqJjRP/?height=500&theme-id=0&default-tab=js,result&embed-version=2&editable=true' frameborder='no' loading="lazy" allowtransparency='true' allowfullscreen='true' style='width: 100%;'>Consulte el Pen para <a href='https://codepen.io/azuremaps/pen/ZqJjRP/'>cambiar el anclaje de ubicación</a> de Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) en <a href='https://codepen.io'>CodePen</a>.
</iframe>

> [!TIP]
> De forma predeterminada, las capas de símbolos optimizan la representación de los símbolos, ya que ocultan los símbolos que se superponen. A medida que se acerca el zoom, los símbolos ocultos se hacen visibles. Para deshabilitar esta característica y representar todos los símbolos en todo momento, establezca la propiedad `allowOverlap` de las opciones `iconOptions` en `true`.

## <a name="add-a-custom-icon-to-a-symbol-layer"></a>Adición de un icono personalizado a una capa de símbolo

Las capas de símbolo se representan mediante WebGL. Por tanto, todos los recursos, como las imágenes de icono, se deben cargar en el contexto de WebGL. En este ejemplo, se muestra cómo se agrega un icono personalizado a los recursos del mapa. Este icono se usa después para representar los datos de punto en el mapa con un símbolo personalizado. La propiedad `textField` de la capa de símbolo requiere que se especifique una expresión. En este caso, queremos representar la propiedad "temperature". Como la temperatura es un número, debe convertirse en una cadena. Además, queremos anexarle "°F". Para ello, podemos utilizar la expresión `['concat', ['to-string', ['get', 'temperature']], '°F']`. 

<br/>

<iframe height='500' scrolling='no' title='Icono de imagen de símbolo personalizado' src='//codepen.io/azuremaps/embed/WYWRWZ/?height=500&theme-id=0&default-tab=js,result&embed-version=2&editable=true' frameborder='no' loading="lazy" allowtransparency='true' allowfullscreen='true' style='width: 100%;'>Consulte el fragmento de código <a href='https://codepen.io/azuremaps/pen/WYWRWZ/'>Custom Symbol Image Icon</a> de Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) en <a href='https://codepen.io'>CodePen</a>.
</iframe>

> [!TIP]
> El SDK de Azure Maps para web proporciona varias plantillas de imagen personalizables que puede usar con la capa de símbolo. Para más información, consulte el documento [Uso de plantillas de imagen](how-to-use-image-templates-web-sdk.md).

## <a name="customize-a-symbol-layer"></a>Personalización de una capa de símbolo 

La capa de símbolo tiene muchas opciones de estilo disponibles. Con esta herramienta puede probar las distintas opciones de estilo.

<br/>

<iframe height='700' scrolling='no' title='Opciones de capa de símbolo' src='//codepen.io/azuremaps/embed/PxVXje/?height=700&theme-id=0&default-tab=result' frameborder='no' loading="lazy" allowtransparency='true' allowfullscreen='true' style='width: 100%;'>Consulte el fragmento de código <a href='https://codepen.io/azuremaps/pen/PxVXje/'>Symbol Layer Options</a> de Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) en <a href='https://codepen.io'>CodePen</a>.
</iframe>

> [!TIP]
> Si solamente desea representar texto con una capa de símbolos, puede ocultar el icono estableciendo la propiedad `image` de las opciones del icono en `'none'`.

## <a name="next-steps"></a>Pasos siguientes

Más información sobre las clases y los métodos utilizados en este artículo:

> [!div class="nextstepaction"]
> [SymbolLayer](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.layer.symbollayer)

> [!div class="nextstepaction"]
> [SymbolLayerOptions](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.symbollayeroptions)

> [!div class="nextstepaction"]
> [IconOptions](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.iconoptions)

> [!div class="nextstepaction"]
> [TextOptions](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.textoptions)

Para obtener más ejemplos de código para agregar a los mapas:

> [!div class="nextstepaction"]
> [Creación de un origen de datos](create-data-source-web-sdk.md)

> [!div class="nextstepaction"]
> [Adición de un elemento emergente](map-add-popup.md)

> [!div class="nextstepaction"]
> [Uso de expresiones de estilo controladas por datos](data-driven-style-expressions-web-sdk.md)

> [!div class="nextstepaction"]
> [Uso de plantillas de imagen](how-to-use-image-templates-web-sdk.md)

> [!div class="nextstepaction"]
> [Adición de una capa de línea](map-add-line-layer.md)

> [!div class="nextstepaction"]
> [Adición de una capa de polígono](map-add-shape.md)

> [!div class="nextstepaction"]
> [Adición de una capa de burbuja](map-add-bubble-layer.md)

> [!div class="nextstepaction"]
> [Adición de marcadores HTML](map-add-bubble-layer.md)
