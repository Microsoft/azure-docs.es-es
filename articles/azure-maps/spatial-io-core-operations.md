---
title: Operaciones de E/S principales | Microsoft Azure Maps
description: Obtenga información sobre cómo leer y escribir datos XML y delimitados de forma eficaz mediante las bibliotecas principales del módulo de E/S espacial.
author: anastasia-ms
ms.author: v-stharr
ms.date: 03/03/2020
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: philmea
ms.custom: devx-track-javascript
ms.openlocfilehash: 021680a81cc5f97bb92c3a9e14036e1a88dae278
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87279515"
---
# <a name="core-io-operations"></a>Operaciones de E/S principales

Además de proporcionar herramientas para leer archivos de datos espaciales, el módulo de E/S espacial expone las bibliotecas subyacentes principales para leer y escribir los datos XML y delimitados de forma rápida y eficaz.

El espacio de nombres `atlas.io.core` contiene dos clases de bajo nivel que pueden leer y escribir rápidamente datos CSV y XML. Estas clases base potencian los lectores y escritores de datos espaciales en el módulo de E/S espacial. No dude en usarlas para agregar compatibilidad adicional de lectura y escritura para los archivos CSV o XML.
 
## <a name="read-delimited-files"></a>Lectura de archivos delimitados

La clase `atlas.io.core.CsvReader` lee las cadenas que contienen conjuntos de datos delimitados. Esta clase proporciona dos métodos para leer los datos:

- La función `read` leerá el conjunto de datos completo y devolverá una matriz bidimensional de cadenas que representa todas las celdas del conjunto de datos delimitado.
- La función `getNextRow` lee cada línea de texto en un conjunto de datos delimitado y devuelve una matriz de cadena que representa todas las celdas de esa línea del conjunto de datos. El usuario puede procesar la fila y desechar cualquier memoria innecesaria de esa fila antes de procesar la siguiente. Por lo tanto, la función es más eficaz con la memoria.

De forma predeterminada, el lector usará el carácter de coma como delimitador. Sin embargo, el delimitador se puede cambiar por cualquier carácter individual o establecerse en `'auto'`. Cuando se establezca en `'auto'`, el lector analizará la primera línea de texto de la cadena. A continuación, seleccionará el carácter más común de la tabla siguiente para usarlo como delimitador.

| Delimitador | Carácter |
| :-- | :-- |
| Coma | `,` |
| Pestaña | `\t` |
| Pipe | `|` |

Este lector también admite los calificadores de texto que se usan para controlar las celdas que contienen el carácter delimitador. El carácter de comillas (`'"'`) es el calificador de texto predeterminado, pero se puede cambiar por cualquier carácter individual.

## <a name="write-delimited-files"></a>Escritura de archivos delimitados

El `atlas.io.core.CsvWriter` escribe una matriz de objetos como una cadena delimitada. Cualquier carácter individual se puede usar como delimitador o como calificador de texto. El delimitador predeterminado es una coma (`','`) y el calificador de texto predeterminado es el carácter de comillas (`'"'`).

Para usar esta clase, siga estos pasos:

- Cree una instancia de la clase y, si lo desea, establezca un delimitador personalizado o un calificador de texto.
- Escriba los datos en la clase mediante la función `write` o la función `writeRow`. En el caso de la función `write`, pase una matriz bidimensional de objetos que representen varias filas y celdas. Para usar la función `writeRow`, pase una matriz de objetos que represente una fila de datos con varias columnas.
- Llame a la función `toString` para recuperar la cadena delimitada. 
- Puede llamar al método `clear` para que el escritor sea reutilizable y reducir su asignación de recursos, o llamar al método `delete` para desechar la instancia del escritor.

> [!Note]
> El número de columnas escritas se restringirá al número de celdas de la primera fila de los datos pasados al escritor.

## <a name="read-xml-files"></a>Lectura de archivos XML

La clase `atlas.io.core.SimpleXmlReader` es más rápida en el análisis de archivos XML que `DOMParser`. Sin embargo, la clase `atlas.io.core.SimpleXmlReader` requiere que los archivos XML estén bien formateados. Los archivos XML que no tienen el formato correcto, por ejemplo si faltan etiquetas de cierre, probablemente producirán un error.

En el código siguiente se muestra cómo usar la clase `SimpleXmlReader` para analizar una cadena XML en un objeto JSON y serializarla en un formato deseado.

```javascript
//Create an instance of the SimpleXmlReader and parse an XML string into a JSON object.
var xmlDoc = new atlas.io.core.SimpleXmlReader().parse(xmlStringToParse);

//Verify that the root XML tag name of the document is the file type your code is designed to parse.
if (xmlDoc && xmlDoc.root && xmlDoc.root.tagName && xmlDoc.root.tagName === '<Your desired root XML tag name>') {

    var node = xmlDoc.root;

    //Loop through the child node tree to navigate through the parsed XML object.
    for (var i = 0, len = node.childNodes.length; i < len; i++) {
        childNode = node.childNodes[i];

        switch (childNode.tagName) {
            //Look for tag names, parse and serialized as desired.
        }
    }
}
```

## <a name="write-xml-files"></a>Escritura de archivos XML

La clase `atlas.io.core.SimpleXmlWriter` escribe XML con formato correcto de un modo eficaz con la memoria.

En el código siguiente se muestra cómo usar la clase `SimpleXmlWriter` para generar una cadena XML con formato correcto.

```javascript
//Create an instance of the SimpleXmlWriter class.
var writer = new atlas.io.core.SimpleXmlWriter();

//Start writing the document. All write functions return a reference to the writer, making it easy to chain the function calls to reduce the code size.
writer.writeStartDocument(true)
    //Specify the root XML tag name, in this case 'root'
    .writeStartElement('root', {
        //Attributes to add to the root XML tag.
        'version': '1.0',
        'xmlns': 'http://www.example.com',
         //Example of a namespace.
        'xmlns:abc': 'http://www.example.com/abc'
    });

//Start writing an element that has the namespace abc and add other XML elements as children.
writer.writeStartElement('abc:parent');

//Write a simple XML element like <title>Azure Maps is awesome!</title>
writer.writeElement('title', 'Azure Maps is awesome!');

//Close the element that we have been writing children to.
writer.writeEndElement();

//Finish writing the document by closing the root tag and the document.
writer.writeEndElement().writeEndDocument();

//Get the generated XML string from the writer.
var xmlString = writer.toString();
```

El XML generado con el código anterior tendría una apariencia similar a la siguiente.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<root version="1.0" xmlns="http://www.example.com" xmlns:abc="http://www.example.com/abc">
    <abc:parent>
        <title>Azure Maps is awesome!</title>
    </abc:parent>
</root>
```

## <a name="next-steps"></a>Pasos siguientes

Más información sobre las clases y los métodos utilizados en este artículo:

> [!div class="nextstepaction"]
> [CsvReader](https://docs.microsoft.com/javascript/api/azure-maps-spatial-io/atlas.io.core.csvreader)

> [!div class="nextstepaction"]
> [CsvWriter](https://docs.microsoft.com/javascript/api/azure-maps-spatial-io/atlas.io.core.csvwriter)

> [!div class="nextstepaction"]
> [SimpleXmlReader](https://docs.microsoft.com/javascript/api/azure-maps-spatial-io/atlas.io.core.simplexmlreader)

> [!div class="nextstepaction"]
> [SimpleXmlWriter](https://docs.microsoft.com/javascript/api/azure-maps-spatial-io/atlas.io.core.simplexmlwriter)

Para obtener más ejemplos de código para agregar a los mapas:

> [!div class="nextstepaction"]
> [Detalles de formatos de datos admitidos](spatial-io-supported-data-format-details.md)