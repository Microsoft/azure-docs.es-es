---
title: Agrupación en clústeres de datos de punto en un mapa| Microsoft Azure Maps
description: Información sobre cómo agrupar datos de puntos en los mapas Vea cómo usar el SDK web de Azure Maps para agrupar datos en clústeres, reaccionar a eventos de mouse de clúster y mostrar agregados de clústeres.
author: rbrundritt
ms.author: richbrun
ms.date: 07/29/2019
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: cpendle
ms.custom: codepen, devx-track-javascript
ms.openlocfilehash: 207106d8daab4d5d2a6d8d681aad5a20923a5a48
ms.sourcegitcommit: 98854e3bd1ab04ce42816cae1892ed0caeedf461
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "88009059"
---
# <a name="clustering-point-data"></a>Agrupación en clústeres de datos de punto

Cuando se visualizan muchos puntos de datos en el mapa, los puntos pueden superponerse entre sí. Este solapamiento puede hacer que el mapa resulte ilegible y difícil de usar. La agrupación en clústeres de datos de punto es el proceso de combinar datos de punto que están cerca unos de otros y representarlos en el mapa como un único punto de datos agrupados en clúster. Cuando el usuario acerca el mapa, los clústeres se separan en sus puntos de datos individuales. Cuando trabaje con un gran número de puntos de datos, utilice procesos de agrupación en clústeres para mejorar la experiencia del usuario.

<br/>

<iframe src="https://channel9.msdn.com/Shows/Internet-of-Things-Show/Clustering-point-data-in-Azure-Maps/player" width="960" height="540" allowFullScreen frameBorder="0"></iframe>

## <a name="enabling-clustering-on-a-data-source"></a>Habilitar la agrupación en clústeres en un origen de datos

Habilite la agrupación en clústeres en la clase `DataSource` estableciendo la opción `cluster` en true. Establezca `ClusterRadius` para seleccionar puntos cercanos y combinarlos en un clúster. El valor de `ClusterRadius` está expresado en píxeles. Utilice `clusterMaxZoom` para especificar el nivel de zoom en el que se va a deshabilitar la lógica de agrupación en clústeres. Este es un ejemplo de cómo habilitar la agrupación en clústeres en un origen de datos.

```javascript
//Create a data source and enable clustering.
var datasource = new atlas.source.DataSource(null, {
    //Tell the data source to cluster point data.
    cluster: true,

    //The radius in pixels to cluster points together.
    clusterRadius: 45,

    //The maximum zoom level in which clustering occurs.
    //If you zoom in more than this, all points are rendered as symbols.
    clusterMaxZoom: 15
});
```

> [!TIP]
> Si dos puntos de datos están muy próximos en la superficie, es posible que nunca se separen, con independencia de lo mucho que el usuario acerque el mapa. Para solucionarlo, puede establecer la opción `clusterMaxZoom` para que la lógica de agrupación en clústeres se deshabilite y se muestren todos los elementos.

Estos son algunos otros métodos que la clase `DataSource` proporciona para la agrupación en clústeres:

| Método | Tipo de valor devuelto | Descripción |
|--------|-------------|-------------|
| getClusterChildren(clusterId: number) | Promesa&lt;Matriz&lt;Característica&lt;Geometría, cualquiera&gt;\| Forma&gt;&gt; | Recupera los elementos secundarios del clúster especificado en el siguiente nivel de zoom. Estos elementos secundarios pueden ser una combinación de formas y subclústeres. Los subclústeres serán características con propiedades que coincidan con ClusteredProperties. |
| getClusterExpansionZoom(clusterId: number) | Promesa&lt;número&gt; | Calcula un nivel de zoom en el que el clúster empezará a expandirse o separarse. |
| getClusterLeaves(clusterId: number, limit: number, offset: number) | Promesa&lt;Matriz&lt;Característica&lt;Geometría, cualquiera&gt;\| Forma&gt;&gt; | Recupera todos los puntos de un clúster. Establezca `limit` para que devuelva un subconjunto de los puntos y use `offset` para paginar a través de los puntos. |

## <a name="display-clusters-using-a-bubble-layer"></a>Mostrar clústeres con una capa de burbujas

Una capa de burbujas es un mecanismo ideal para representar los puntos de datos agrupados en clústeres. Utilice expresiones para escalar el radio y cambiar el color en función del número de puntos del clúster. Si los clústeres se muestran utilizando una capa de burbujas, deberá usar también otra capa diferente para representar los puntos de datos no agrupados.

Para mostrar el tamaño del clúster encima de la burbuja, utilice una capa de símbolos y no utilice iconos.

<br/>

<iframe height="500" style="width: 100%;" scrolling="no" title="Capa básica de burbujas de agrupación en clústeres" src="//codepen.io/azuremaps/embed/qvzRZY/?height=500&theme-id=0&default-tab=js,result&editable=true" frameborder="no" allowtransparency="true" allowfullscreen="true">
Consulte el Pen <a href='https://codepen.io/azuremaps/pen/qvzRZY/'>Basic bubble layer clustering</a> (Agrupación en clústeres básica de capa de burbujas) de Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) en <a href='https://codepen.io'>CodePen</a>.
</iframe>

## <a name="display-clusters-using-a-symbol-layer"></a>Mostrar clústeres con una capa de símbolos

Cuando se visualizan puntos de datos, la capa de símbolos oculta automáticamente los símbolos que se solapan entre sí para que la interfaz de usuario se vea más limpia. Este comportamiento predeterminado podría no resultar conveniente si desea mostrar la densidad de puntos de datos en el mapa. Sin embargo, esta configuración se puede cambiar. Para mostrar todos los símbolos, establezca la opción `allowOverlap` de la propiedad `iconOptions` de la capa de símbolos en `true`. 

Utilice la agrupación en clústeres para mostrar la densidad de puntos de datos y, al mismo tiempo, que la interfaz de usuario se vea limpia. En el ejemplo siguiente, se muestra cómo se agregan símbolos personalizados y cómo se representan clústeres y puntos de datos individuales utilizando la capa de símbolos.

<br/>

<iframe height="500" style="width: 100%;" scrolling="no" title="Capa de símbolos agrupados en clúster" src="//codepen.io/azuremaps/embed/Wmqpzz/?height=500&theme-id=0&default-tab=js,result&editable=true" frameborder="no" allowtransparency="true" allowfullscreen="true">
Consulte el Pen <a href='https://codepen.io/azuremaps/pen/Wmqpzz/'>Clustered Symbol layer</a> (Capa de símbolos agrupados en clústeres) de Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) en <a href='https://codepen.io'>CodePen</a>.
</iframe>

## <a name="clustering-and-the-heat-maps-layer"></a>Agrupación en clústeres y capa de mapas térmicos

Los mapas térmicos son una excelente manera de mostrar la densidad de los datos en el mapa. Este método de visualización puede administrar un gran número de puntos de datos por sí solo. Si los puntos de datos están agrupados en clústeres y el tamaño del clúster se utiliza como ponderación del mapa térmico, este mapa podrá administrar aún más datos. Para ello, establezca la opción `weight` de la capa del mapa térmico en `['get', 'point_count']`. Si el radio del clúster es pequeño, el mapa térmico será casi idéntico a un mapa térmico que use puntos de datos no agrupados en clústeres, pero funcionará mucho mejor. Sin embargo, cuanto menor sea el radio del clúster, más preciso será el mapa térmico, aunque también menos beneficioso para el rendimiento.

<br/>

<iframe height="500" style="width: 100%;" scrolling="no" title="Mapa térmico ponderado del clúster" src="//codepen.io/azuremaps/embed/VRJrgO/?height=500&theme-id=0&default-tab=js,result&editable=true" frameborder="no" allowtransparency="true" allowfullscreen="true">
Consulte el Pen <a href='https://codepen.io/azuremaps/pen/VRJrgO/'>Cluster weighted Heat Map</a> (Clúster ponderado de mapa térmico) de Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) en <a href='https://codepen.io'>CodePen</a>.
</iframe>

## <a name="mouse-events-on-clustered-data-points"></a>Eventos del mouse en los puntos de datos agrupados en clúster

Si se producen eventos del mouse en una capa que contiene puntos de datos agrupados en clústeres, se devolverán los puntos de datos agrupados para el evento como un objeto de característica de punto de GeoJSON. La característica de punto tendrá las siguientes propiedades:

| Nombre de propiedad             | Tipo    | Descripción   |
|---------------------------|---------|---------------|
| `cluster`                 | boolean | Indica si la característica representa un clúster. |
| `cluster_id`              | string  | Un id. exclusivo para el clúster que se puede usar con los métodos `getClusterExpansionZoom`, `getClusterChildren` y `getClusterLeaves` de DataSource. |
| `point_count`             | number  | El número de puntos que contiene el clúster.  |
| `point_count_abbreviated` | string  | Cadena que abrevia el valor de `point_count`, si es muy largo. (por ejemplo, 4000 se convierte en 4 K).  |

En este ejemplo, se toma la capa de burbujas que representa los puntos del clúster y se agrega un evento de clic. Cuando el evento de clic se desencadena, el código realiza los cálculos y aplica el zoom en el mapa para llevarlo al siguiente nivel, donde el clúster se separa. Esta funcionalidad se implementa utilizando el método `getClusterExpansionZoom` de la clase `DataSource` y la propiedad `cluster_id` del punto de datos agrupado en el que hizo clic.

<br/>

<iframe height="500" style="width: 100%;" scrolling="no" title="Clúster getClusterExpansionZoom" src="//codepen.io/azuremaps/embed/moZWeV/?height=500&theme-id=0&default-tab=js,result&editable=true" frameborder="no" allowtransparency="true" allowfullscreen="true">
Consulte el Pen <a href='https://codepen.io/azuremaps/pen/moZWeV/'>Cluster getClusterExpansionZoom</a> (Clúster getClusterExpansionZoom) de Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) en <a href='https://codepen.io'>CodePen</a>.
</iframe>

## <a name="display-cluster-area"></a>Mostrar área de clúster 

Los datos de punto que un clúster representa se dispersan por un área. En este ejemplo, cuando el mouse se sitúa sobre un clúster, se producen dos comportamientos principales. En primer lugar, los puntos de datos individuales incluidos en el clúster se utilizan para calcular la envolvente convexa. Después, la envolvente convexa aparece en el mapa para mostrar un área.  Una envolvente convexa es un polígono que encapsula un conjunto de puntos como una banda elástica y puede calcularse mediante el método `atlas.math.getConvexHull`. Se pueden recuperar todos los puntos del origen de datos contenidos en un clúster mediante el método `getClusterLeaves`.

<br/>

 <iframe height="500" style="width: 100%;" scrolling="no" title="Envolvente convexa del área de clúster" src="//codepen.io/azuremaps/embed/QoXqWJ/?height=500&theme-id=0&default-tab=js,result&editable=true" frameborder="no" allowtransparency="true" allowfullscreen="true">
Consulte el Pen <a href='https://codepen.io/azuremaps/pen/QoXqWJ/'>Cluster area convex hull</a> (Envolvente convexa del área de clúster) de Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) en <a href='https://codepen.io'>CodePen</a>.
</iframe>

## <a name="aggregating-data-in-clusters"></a>Agregación de datos en clústeres

Los clústeres suelen representarse utilizando un símbolo con el número de puntos que hay dentro del clúster. Sin embargo, en ocasiones es preferible personalizar el estilo de los clústeres utilizando otras métricas. Con los agregados de clúster, se pueden crear y rellenar propiedades personalizadas utilizando un cálculo de [expresión de agregado](data-driven-style-expressions-web-sdk.md#aggregate-expression).  Los agregados de clúster se pueden en la opción `clusterProperties` de `DataSource`.

En los ejemplos siguientes, se utiliza una expresión de agregado. El código calcula un número en función de la propiedad de tipo de entidad de cada punto de datos del clúster. Cuando un usuario hace clic en un clúster, se abre un elemento emergente con información adicional sobre el clúster.

<iframe height="500" style="width: 100%;" scrolling="no" title="Agregados de clúster" src="//codepen.io/azuremaps/embed/jgYyRL/?height=500&theme-id=0&default-tab=js,result&editable=true" frameborder="no" allowtransparency="true" allowfullscreen="true">
Consulte los Pen <a href='https://codepen.io/azuremaps/pen/jgYyRL/'>Cluster aggregates</a> (Agregados de clúster) de Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) en <a href='https://codepen.io'>CodePen</a>.
</iframe>

## <a name="next-steps"></a>Pasos siguientes

Más información sobre las clases y los métodos utilizados en este artículo:

> [!div class="nextstepaction"]
> [Clase DataSource](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.source.datasource?view=azure-iot-typescript-latest)

> [!div class="nextstepaction"]
> [Objeto DataSourceOptions](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.datasourceoptions?view=azure-iot-typescript-latest)

> [!div class="nextstepaction"]
> [Espacio de nombres atlas.math](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.math?view=azure-iot-typescript-latest)

Vea los siguientes ejemplos de código para agregar funcionalidad a la aplicación:

> [!div class="nextstepaction"]
> [Adición de una capa de burbuja](map-add-bubble-layer.md)

> [!div class="nextstepaction"]
> [Adición de una capa de símbolo](map-add-pin.md)

> [!div class="nextstepaction"]
> [Adición de una capa de mapa térmico](map-add-heat-map-layer.md)
