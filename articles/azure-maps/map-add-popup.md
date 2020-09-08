---
title: Adición de un elemento emergente a un punto de un mapa | Microsoft Azure Maps
description: Obtenga información sobre los elementos emergentes, las plantillas emergentes y los eventos emergentes en Azure Maps. Vea cómo agregar un elemento emergente a un punto en un mapa y cómo reutilizar y personalizar los elementos emergentes.
author: anastasia-ms
ms.author: v-stharr
ms.date: 02/27/2020
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: ''
ms.custom: codepen, devx-track-javascript
ms.openlocfilehash: 5235b273ab45a2ece011f38bf363b33230101396
ms.sourcegitcommit: bfeae16fa5db56c1ec1fe75e0597d8194522b396
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/10/2020
ms.locfileid: "88036511"
---
# <a name="add-a-popup-to-the-map"></a>Adición de un elemento emergente al mapa

En este artículo se muestra cómo agregar un elemento emergente a un punto de un mapa.

## <a name="understand-the-code"></a>Comprendiendo el código

El código siguiente agrega una característica de punto, que tiene las propiedades `name` y `description`, al mapa mediante una capa de símbolo. Se crea una instancia de la [clase popup](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.popup) pero no se muestra. Los eventos del mouse se agregan a la capa de símbolos para desencadenar la apertura y el cierre del elemento emergente. Cuando el mouse se mantiene sobre el símbolo del marcador, la propiedad `position` del elemento emergente se actualiza con la posición del marcador y la opción `content` se actualiza con el código HTML que contiene las propiedades `name` y `description` de la característica de puntos que se está activando. Después, se muestra el elemento emergente en el mapa mediante su función `open`.

```javascript
//Define an HTML template for a custom popup content laypout.
var popupTemplate = '<div class="customInfobox"><div class="name">{name}</div>{description}</div>';

//Create a data source and add it to the map.
var dataSource = new atlas.source.DataSource();
map.sources.add(dataSource);

dataSource.add(new atlas.data.Feature(new atlas.data.Point([-122.1333, 47.63]), {
  name: 'Microsoft Building 41', 
  description: '15571 NE 31st St, Redmond, WA 98052'
}));

//Create a layer to render point data.
var symbolLayer = new atlas.layer.SymbolLayer(dataSource);

//Add the polygon and line the symbol layer to the map.
map.layers.add(symbolLayer);

//Create a popup but leave it closed so we can update it and display it later.
popup = new atlas.Popup({
  pixelOffset: [0, -18],
  closeButton: false
});

//Add a hover event to the symbol layer.
map.events.add('mouseover', symbolLayer, function (e) {
  //Make sure that the point exists.
  if (e.shapes && e.shapes.length > 0) {
    var content, coordinate;
    var properties = e.shapes[0].getProperties();
    content = popupTemplate.replace(/{name}/g, properties.name).replace(/{description}/g, properties.description);
    coordinate = e.shapes[0].getCoordinates();

    popup.setOptions({
      //Update the content of the popup.
      content: content,

      //Update the popup's position with the symbol's coordinate.
      position: coordinate

    });
    //Open the popup.
    popup.open(map);
  }
});

map.events.add('mouseleave', symbolLayer, function (){
  popup.close();
});
```

A continuación se muestra el código de ejemplo de ejecución completo de la funcionalidad anterior.

<br/>

<iframe height='500' scrolling='no' title='Adición de un elemento emergente con Azure Maps' src='//codepen.io/azuremaps/embed/MPRPvz/?height=500&theme-id=0&default-tab=result&embed-version=2&editable=true' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>Consulte el Pen <a href='https://codepen.io/azuremaps/pen/MPRPvz/'>Adición de un elemento emergente con Azure Maps</a> de Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) en <a href='https://codepen.io'>CodePen</a>.
</iframe>

## <a name="reusing-a-popup-with-multiple-points"></a>Reutilizar un elemento emergente con varios puntos

Hay casos en los que el mejor enfoque es crear un elemento emergente y reutilizarlo. Por ejemplo, puede tener una gran cantidad de puntos y desear mostrar solo un elemento emergente cada vez. Al reutilizar el elemento emergente, el número de elementos DOM creados por la aplicación se reduce tanto que puede proporcionar un mejor rendimiento. En el ejemplo siguiente se crean tres características de punto. Si hace clic en cualquiera de ellas, se mostrará un menú emergente con el contenido de esa característica de punto.

<br/>

<iframe height='500' scrolling='no' title='Reutilizar un elemento emergente con varias marcas' src='//codepen.io/azuremaps/embed/rQbjvK/?height=500&theme-id=0&default-tab=result&embed-version=2&editable=true' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>Consulte el Pen sobre <a href='https://codepen.io/azuremaps/pen/rQbjvK/'>reutilización de elementos emergentes con varias marcas</a> de Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) en <a href='https://codepen.io'>CodePen</a>.
</iframe>

## <a name="customizing-a-popup"></a>Personalización de un elemento emergente

De forma predeterminada, el elemento emergente tiene un fondo blanco, una flecha de puntero en la parte inferior y un botón de cierre en la esquina superior derecha. En el ejemplo siguiente se cambia el color de fondo a negro mediante la opción `fillColor` del elemento emergente. El botón de cierre se quita al establecer la opción `CloseButton` en false. El contenido HTML del elemento emergente utiliza un relleno de 10 píxeles desde los bordes del elemento emergente. El texto aparece en color blanco para que pueda verse claramente sobre el fondo negro.  

<br/>

<iframe height="500" style="width: 100%;" scrolling="no" title="Elemento emergente personalizado" src="//codepen.io/azuremaps/embed/ymKgdg/?height=500&theme-id=0&default-tab=result" frameborder="no" allowtransparency="true" allowfullscreen="true">
Consulte el fragmento de código (pen) <a href='https://codepen.io/azuremaps/pen/ymKgdg/'>elemento emergente personalizado</a> de Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) en <a href='https://codepen.io'>CodePen</a>.
</iframe>

## <a name="add-popup-templates-to-the-map"></a>Incorporación de plantillas emergentes al mapa

Las plantillas emergentes facilitan la creación de diseños basados en datos para los elementos emergentes. En las secciones siguientes se muestra el uso de varias plantillas emergentes para generar contenido con formato mediante las propiedades de las características.

> [!NOTE]
> De forma predeterminada, todo el contenido representado en la plantilla emergente estará en un espacio aislado dentro de un iframe como característica de seguridad, Sin embargo, existen limitaciones:
>
> - Todos los scripts, los formularios y las funcionalidades de bloqueo de puntero y de navegación superior están deshabilitados. Al hacer clic en los vínculos, se pueden abrir en una pestaña nueva. 
> - Los exploradores más antiguos que no admitan el parámetro `srcdoc` en iframes se limitarán a representar una pequeña cantidad de contenido.
> 
> Si confía en los datos que se van a cargar en los elementos emergentes y, potencialmente, quiere que estos scripts cargados en los elementos emergentes puedan acceder a su aplicación, puede deshabilitarlo estableciendo la opción `sandboxContent` de las plantillas emergentes en false. 

### <a name="string-template"></a>Plantilla de cadena

La plantilla de cadena reemplaza los marcadores de posición por los valores de las propiedades de la característica. No es necesario asignar un valor de tipo String a las propiedades de la característica. Por ejemplo, `value1` contiene un entero. A continuación, estos valores se pasan a la propiedad Content de `popupTemplate`. 

La opción `numberFormat` especifica el formato del número que se va a mostrar. Si no se especifica el `numberFormat`, el código utilizará el formato de fecha de las plantillas emergentes. La opción `numberFormat` da formato a los números mediante la función [Number.toLocaleString](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Number/toLocaleString). Para dar formato a números grandes, considere la posibilidad de usar la opción `numberFormat` con funciones de [NumberFormat.Format](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/NumberFormat/format). Por ejemplo, el siguiente fragmento de código usa `maximumFractionDigits` para limitar el número de dígitos de fracción a dos.

> [!Note]
> Solo hay una manera en la que la plantilla de cadena puede representar imágenes. En primer lugar, la plantilla de cadena debe tener una etiqueta de imagen. El valor que se pasa a la etiqueta de imagen debe ser una dirección URL a una imagen. A continuación, la plantilla de cadena debe tener `isImage` establecida en true en `HyperLinkFormatOptions`. La opción `isImage` especifica que el hipervínculo es para una imagen y el hipervínculo se cargará en una etiqueta de imagen. Cuando se haga clic en el hipervínculo, se abrirá la imagen.

```javascript
var templateOptions = {
  content: 'This template uses a string template with placeholders.<br/><br/> - Value 1 = {value1}<br/> - Value 2 = {value2/subValue}<br/> - Array value [2] = {arrayValue/2}',
  numberFormat: {
    maximumFractionDigits: 2
  }
};

var feature = new atlas.data.Feature(new atlas.data.Point([0, 0]), {
    title: 'Template 1 - String template',
    value1: 1.2345678,
    value2: {
        subValue: 'Pizza'
    },
    arrayValue: [3, 4, 5, 6]
});

var popup = new atlas.Popup({
  content: atlas.PopupTemplate.applyTemplate(feature.properties, templateOptions),
  position: feature.geometry.coordinates
});
```

### <a name="propertyinfo-template"></a>Plantilla PropertyInfo

La plantilla PropertyInfo muestra las propiedades disponibles de la característica. La opción `label` especifica el texto que se va a mostrar al usuario. Si no se especifica `label`, el hipervínculo se mostrará. Y, si el hipervínculo es una imagen, se mostrará el valor asignado a la etiqueta "Alt". `dateFormat` especifica el formato de la fecha. Si no se especifica, la fecha se representará como una cadena. La opción `hyperlinkFormat` representa los vínculos en los que se pueden hacer clic; de forma similar, se puede usar la opción `email` para representar direcciones de correo electrónico en las que se puede hacer clic.

Antes de que la plantilla PropertyInfo muestre las propiedades al usuario final, comprueba de forma recursiva que las propiedades se definen realmente para esa característica. También se omite la visualización de las propiedades de estilo y título. Por ejemplo, no mostrará `color`, `size`, `anchor`, `strokeOpacity` y `visibility`. Por lo tanto, una vez completada la comprobación de la ruta de acceso de la propiedad en el back-end, la plantilla PropertyInfo muestra el contenido en un formato de tabla.

```javascript
var templateOptions = {
  content: [
    {
        propertyPath: 'createDate',
        label: 'Created Date'
    },
    {
        propertyPath: 'dateNumber',
        label: 'Formatted date from number',
        dateFormat: {
          weekday: 'long',
          year: 'numeric',
          month: 'long',
          day: 'numeric',
          timeZone: 'UTC',
          timeZoneName: 'short'
        }
    },
    {
        propertyPath: 'url',
        label: 'Code samples',
        hideLabel: true,
        hyperlinkFormat: {
          lable: 'Go to code samples!',
          target: '_blank'
        }
    },
    {
        propertyPath: 'email',
        label: 'Email us',
        hideLabel: true,
        hyperlinkFormat: {
          target: '_blank',
          scheme: 'mailto:'
        }
    }
  ]
};

var feature = new atlas.data.Feature(new atlas.data.Point([0, 0]), {
    title: 'Template 2 - PropertyInfo',
    createDate: new Date(),
    dateNumber: 1569880860542,
    url: 'https://aka.ms/AzureMapsSamples',
    email: 'info@microsoft.com'
}),

var popup = new atlas.Popup({
  content: atlas.PopupTemplate.applyTemplate(feature.properties, templateOptions),
  position: feature.geometry.coordinates
});
```

### <a name="multiple-content-templates"></a>Varias plantillas de contenido

Una característica también puede mostrar contenido mediante una combinación de la plantilla String y la plantilla PropertyInfo. En este caso, la plantilla String representa los valores de los marcadores de posición en un fondo blanco.  Y la plantilla PropertyInfo representa una imagen de ancho completo dentro de una tabla. Las propiedades de este ejemplo son similares a las que se explican en los ejemplos anteriores.

```javascript
var templateOptions = {
  content: [
    'This template has two pieces of content; a string template with placeholders and a array of property info which renders a full width image.<br/><br/> - Value 1 = {value1}<br/> - Value 2 = {value2/subValue}<br/> - Array value [2] = {arrayValue/2}',
    [{
      propertyPath: 'imageLink',
      label: 'Image',
      hideImageLabel: true,
      hyperlinkFormat: {
        isImage: true
      }
    }]
  ],
  numberFormat: {
    maximumFractionDigits: 2
  }
};

var feature = new atlas.data.Feature(new atlas.data.Point([0, 0]), {
    title: 'Template 3 - Multiple content template',
    value1: 1.2345678,
    value2: {
    subValue: 'Pizza'
    },
    arrayValue: [3, 4, 5, 6],
    imageLink: 'https://azuremapscodesamples.azurewebsites.net/common/images/Pike_Market.jpg'
});

var popup = new atlas.Popup({
  content: atlas.PopupTemplate.applyTemplate(feature.properties, templateOptions),
  position: feature.geometry.coordinates
});
```

### <a name="points-without-a-defined-template"></a>Puntos sin una plantilla definida

Cuando la plantilla Popup no se define como una plantilla String, una plantilla PropertyInfo o una combinación de ambas, utiliza la configuración predeterminada. Cuando las propiedades `title` y `description` son las únicas asignadas, la plantilla emergente muestra un fondo blanco y un botón de cerrar en la esquina superior derecha. Y, en pantallas pequeñas y medianas, muestra una flecha en la parte inferior. La configuración predeterminada se muestra dentro de una tabla para todas las propiedades distintas de `title` y `description`. Incluso cuando se revierte a la configuración predeterminada, la plantilla emergente todavía se puede manipular mediante programación. Por ejemplo, los usuarios pueden desactivar la detección de hipervínculos y la configuración predeterminada se seguirá aplicando a otras propiedades.

Haga clic en los puntos del mapa en CodePen. Hay un punto en el mapa para cada una de las siguientes plantillas emergentes: Plantilla String, plantilla PropertyInfo y plantilla de contenido Multiple. También hay tres puntos para mostrar cómo se representan las plantillas mediante la configuración predeterminada.

<br/>

<iframe height='500' scrolling='no' title='PopupTemplates' src='//codepen.io/azuremaps/embed/dyovrzL/?height=500&theme-id=0&default-tab=result&embed-version=2&editable=true' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>Consulte el Pen <a href='https://codepen.io/azuremaps/pen/dyovrzL/'>PopupTemplates</a> de Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) en <a href='https://codepen.io'>CodePen</a>.
</iframe>

## <a name="reuse-popup-template"></a>Reutilización de la plantilla emergente

De forma similar a cómo se reutilizan las ventanas emergentes, puede volver a usar las plantillas emergentes. Este enfoque es útil cuando solo desea mostrar una plantilla emergente cada vez, para varios puntos. Al reutilizar la plantilla emergente, el número de elementos DOM que la aplicación crea se reduce, con lo que se mejora el rendimiento de la aplicación. En el ejemplo siguiente se usa la misma plantilla emergente para tres puntos. Si hace clic en cualquiera de ellas, se mostrará un menú emergente con el contenido de esa característica de punto.

<br/>

<iframe height='500' scrolling='no' title='ReusePopupTemplate' src='//codepen.io/azuremaps/embed/WNvjxGw/?height=500&theme-id=0&default-tab=result&embed-version=2&editable=true' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>Consulte el Pen <a href='https://codepen.io/azuremaps/pen/WNvjxGw/'>ReusePopupTemplate</a> de Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) en <a href='https://codepen.io'>CodePen</a>.
</iframe>

## <a name="popup-events"></a>Eventos emergentes

Los elementos emergentes se pueden abrir, cerrar y arrastrar. La clase del elemento emergente proporciona eventos que ayudan a los desarrolladores a reaccionar ante estos eventos. En el ejemplo siguiente, aparecen resaltados los eventos que se activan cuando un usuario abre, cierra o arrastra el elemento emergente. 

<br/>

<iframe height="500" style="width: 100%;" scrolling="no" title="Eventos emergentes" src="//codepen.io/azuremaps/embed/BXrpvB/?height=500&theme-id=0&default-tab=result" frameborder="no" allowtransparency="true" allowfullscreen="true">
Consulte el fragmento de código (pen) <a href='https://codepen.io/azuremaps/pen/BXrpvB/'>eventos de elemento emergente</a> de Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) en <a href='https://codepen.io'>CodePen</a>.
</iframe>

## <a name="next-steps"></a>Pasos siguientes

Más información sobre las clases y los métodos utilizados en este artículo:

> [!div class="nextstepaction"]
> [Popup](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.popup)

> [!div class="nextstepaction"]
> [PopupOptions](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.popupoptions)

> [!div class="nextstepaction"]
> [PopupTemplate](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.popuptemplate)

Consulte los siguientes artículos para obtener ejemplos de código completo:

> [!div class="nextstepaction"]
> [Adición de una capa de símbolo](./map-add-pin.md)

> [!div class="nextstepaction"]
> [Adición de un marcador HTML](./map-add-custom-html.md)

> [!div class="nextstepaction"]
> [Adición de una capa de línea](map-add-line-layer.md)

> [!div class="nextstepaction"]
> [Adición de una capa de polígono](map-add-shape.md)
