---
title: Creación de un proveedor de sugerencias
titleSuffix: Azure Cognitive Search
description: Para habilitar acciones de escritura automática en Azure Cognitive Search, puede crear proveedores de sugerencias y formular las solicitudes que invocan los términos de consulta de sugerencias automáticas o de autocompletar.
manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 04/21/2020
ms.custom: devx-track-csharp
ms.openlocfilehash: 4889e73e851e285c84d5d4429298e9a7cdacc140
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89014394"
---
# <a name="create-a-suggester-to-enable-autocomplete-and-suggested-results-in-a-query"></a>Creación de un proveedor de sugerencias para habilitar la función autocompletar y los resultados sugeridos en una consulta

En Azure Cognitive Search, la función de "búsqueda al escribir" se habilita mediante una construcción de **proveedor de sugerencias** que se agrega a un [índice de búsqueda](search-what-is-an-index.md). Un proveedor de sugerencias admite dos experiencias: *autocompletar*, que completa una entrada parcial para una consulta de un término completo, y *sugerencias*, que invita a hacer clic en una coincidencia concretar. Autocompletar genera una consulta. Las sugerencias producen un documento coincidente.

En la siguiente captura de pantalla del artículo [Creación de la primera aplicación en C#](tutorial-csharp-type-ahead-and-suggestions.md), se muestran ambas experiencias. Autocompletar anticipa un posible término, agregando a "met" la terminación "ro" por ejemplo. Las sugerencias son resultados de búsquedas en miniatura, donde un campo como el nombre de un hotel representa un documento de búsqueda de hoteles coincidentes del índice. Para obtener sugerencias, puede mostrar cualquier campo que proporcione información descriptiva.

![Comparación visual de Autocompletar y consultas sugeridas](./media/index-add-suggesters/hotel-app-suggestions-autocomplete.png "Comparación visual de Autocompletar y consultas sugeridas")

Puede utilizar estas características de forma conjunta o por separado. Para implementar estos comportamientos en Azure Cognitive Search, hay un componente de consulta e índice. 

+ En el índice, agregue un proveedor de sugerencias a un índice. Puede usar el portal, la [API REST](/rest/api/searchservice/create-index) o el [SDK de .NET](/dotnet/api/microsoft.azure.search.models.suggester?view=azure-dotnet). El resto de este artículo se centra en la creación de un proveedor de sugerencias.

+ En la solicitud de consulta, llame a una de las [API enumeradas a continuación](#how-to-use-a-suggester).

La compatibilidad con la búsqueda al escribir está habilitada por cada campo para los campos de cadena. Puede implementar ambos comportamientos de escritura automática dentro de la misma solución de búsqueda si desea una experiencia similar a la indicada en la captura de pantalla. El destino de ambas solicitudes son la colección de *documentos* de índice específico y las respuestas se devuelven después de que un usuario haya proporcionado al menos una cadena de entrada de tres caracteres.

## <a name="what-is-a-suggester"></a>¿Qué es un proveedor de sugerencias?

Un proveedor de sugerencias es una estructura de datos interna que admite comportamientos de búsqueda mientras se escribe mediante el almacenamiento de prefijos para buscar coincidencias en consultas parciales. Igual que sucede con los términos tokenizados, los prefijos se almacenan en índices invertidos, uno para cada campo especificado en una colección de campos del proveedor de sugerencias.

## <a name="define-a-suggester"></a>Definición de un proveedor de sugerencias

Para crear un proveedor de sugerencias, agregue uno a un [esquema de índice](/rest/api/searchservice/create-index) y [establezca las propiedades](#property-reference). El mejor momento para crear un proveedor de sugerencias es durante la definición del campo que va a usarlo.

+ Usar solo campos de cadena

+ Usar el analizador Lucene estándar predeterminado (`"analyzer": null`) o un [analizador de idioma](index-add-language-analyzers.md) (por ejemplo, `"analyzer": "en.Microsoft"`) en el campo

### <a name="choose-fields"></a>Elegir los campos

Aunque los proveedores de sugerencias tienen varias propiedades, son principalmente una colección de campos de cadena para los que va a habilitar la experiencia de búsqueda mientras se escribe. Hay un proveedor de sugerencias para cada índice, por lo que la lista de proveedores de sugerencias debe incluir todos los campos que contribuyen al contenido tanto de las sugerencias como de autocompletar.

La ventaja de autocompletar es que tiene un grupo de campos mayor, ya que el contenido adicional tiene más potencial de finalización de términos.

Por su parte, las sugerencias generan mejores resultados cuando la elección de campo es selectiva. Recuerde que la sugerencia es un proxy para un documento de búsqueda, por lo que deseará los campos que mejor se representen como un resultado individual. Los nombres, títulos u otros campos únicos que distinguen entre varias coincidencias son los que mejor funcionan. Si los campos se componen de valores repetitivos, las sugerencias consistirán en resultados idénticos y el usuario no sabrá en cuál hacer clic.

Para satisfacer las experiencias de buscar mientras se escribe, agregue todos los campos que necesita para autocompletar y, después, use **$select**, **$top**, **$filter** y **searchFields** para controlar los resultados de las sugerencias.

### <a name="choose-analyzers"></a>Elección de analizadores

La elección de un analizador determina el modo en que los campos se tokenizan y, posteriormente, se prefijan. Por ejemplo, para una cadena con guiones como "anti-virus", el uso de un analizador de idioma producirá estas combinaciones de tokens: "anti", "virus", "anti-virus". Si usa el analizador de Lucene estándar, la cadena con guiones no existirá. 

Al evaluar analizadores, considere la posibilidad de usar [Analyze Text API](/rest/api/searchservice/test-analyzer) para obtener información sobre cómo se tokenizan los términos y después se prefijan. Una vez creado un índice, puede probar varios analizadores en una cadena para ver la salida de los tokens.

Los campos que utilizan [analizadores personalizados](index-add-custom-analyzers.md) o [predefinidos](index-add-custom-analyzers.md#predefined-analyzers-reference) (con la excepción del estándar de Lucene) no se permiten para evitar resultados deficientes.

> [!NOTE]
> Si necesita solucionar la restricción del analizador, por ejemplo si necesita una palabra clave o un analizador de n-gramas para ciertos escenarios de consulta, debería usar dos campos independientes para el mismo contenido. De esta forma, uno de los campos tendrá un proveedor de sugerencias y el otro se puede configurar con una configuración de analizador personalizada.

### <a name="when-to-create-a-suggester"></a>Cuándo crear un proveedor de sugerencias

El mejor momento para crear un proveedor de sugerencias es durante la creación de la propia definición de campo.

Si intenta crearlo con campos ya existentes, la API no lo permitirá. Los prefijos se generan durante la indexación, cuando se tokenizan los términos parciales de dos o más combinaciones de caracteres junto con los términos completos. Dado que los campos existentes ya están acortados, tendrá que volver a generar el índice si quiere agregarlos a un proveedor de sugerencias. Para más información, consulte [Cómo recompilar un índice de Azure Cognitive Search](search-howto-reindex.md).

## <a name="create-using-rest"></a>Creación mediante REST

En la API REST, puede agregar proveedores de sugerencias con [Create Index](/rest/api/searchservice/create-index) o [Update Index](/rest/api/searchservice/update-index). 

  ```json
  {
    "name": "hotels-sample-index",
    "fields": [
      . . .
          {
              "name": "HotelName",
              "type": "Edm.String",
              "facetable": false,
              "filterable": false,
              "key": false,
              "retrievable": true,
              "searchable": true,
              "sortable": false,
              "analyzer": "en.microsoft",
              "indexAnalyzer": null,
              "searchAnalyzer": null,
              "synonymMaps": [],
              "fields": []
          },
    ],
    "suggesters": [
      {
        "name": "sg",
        "searchMode": "analyzingInfixMatching",
        "sourceFields": ["HotelName"]
      }
    ],
    "scoringProfiles": [
      . . .
    ]
  }
  ```

## <a name="create-using-net"></a>Creación mediante .NET

En C#, defina un [objeto Suggester](/dotnet/api/microsoft.azure.search.models.suggester?view=azure-dotnet). `Suggesters` es una colección, pero puede tomar solamente un elemento. 

```csharp
private static void CreateHotelsIndex(SearchServiceClient serviceClient)
{
    var definition = new Index()
    {
        Name = "hotels-sample-index",
        Fields = FieldBuilder.BuildForType<Hotel>(),
        Suggesters = new List<Suggester>() {new Suggester()
            {
                Name = "sg",
                SourceFields = new string[] { "HotelName", "Category" }
            }}
    };

    serviceClient.Indexes.Create(definition);

}
```

## <a name="property-reference"></a>Referencia de propiedades

|Propiedad      |Descripción      |
|--------------|-----------------|
|`name`        |El nombre del proveedor de sugerencias.|
|`searchMode`  |La estrategia que se usa para buscar las frases candidatas. El único modo que se admite actualmente es `analyzingInfixMatching`, que actualmente coincide en el principio de un término.|
|`sourceFields`|Una lista de uno o más campos que son el origen del contenido para obtener sugerencias. Los campos deben ser de tipo `Edm.String` y `Collection(Edm.String)`. Si se especifica un analizador en el campo, debe ser un analizador con nombre de [esta lista](/dotnet/api/microsoft.azure.search.models.analyzername?view=azure-dotnet) (no un analizador personalizado).<p/> Como procedimiento recomendado, especifique solo los campos que se prestan a una respuesta adecuada y esperada, ya sea una cadena completa en una barra de búsqueda o una lista desplegable.<p/>Un nombre de hotel es buen candidato porque tiene precisión. Los campos detallados, como las descripciones y los comentarios, son demasiado densos. De forma similar, los campos repetitivos, como las categorías y las etiquetas, son menos eficaces. En los ejemplos, se incluye "categoría" de todos modos para demostrar que puede incluir varios campos. |

<a name="how-to-use-a-suggester"></a>

## <a name="use-a-suggester"></a>Uso de un proveedor de sugerencias

El proveedor de sugerencias se utiliza en una consulta. Después de que se crea un proveedor de sugerencias, llame a una de los siguientes API para una experiencia de buscar mientras se escribe:

+ [API REST de sugerencias](/rest/api/searchservice/suggestions) 
+ [API REST de autocompletar](/rest/api/searchservice/autocomplete) 
+ [Método SuggestWithHttpMessagesAsync](/dotnet/api/microsoft.azure.search.idocumentsoperations.suggestwithhttpmessagesasync?view=azure-dotnet)
+ [Método AutocompleteWithHttpMessagesAsync](/dotnet/api/microsoft.azure.search.idocumentsoperations.autocompletewithhttpmessagesasync?view=azure-dotnet&viewFallbackFrom=azure-dotnet)

En una aplicación de búsqueda, el código de cliente debería aprovechar una biblioteca como [jQuery UI Autocomplete](https://jqueryui.com/autocomplete/) para recopilar la consulta parcial y ofrecer la coincidencia. Para más información acerca de esta tarea, consulte [Incorporación de sugerencias o de la función de autocompletar al código de cliente](search-autocomplete-tutorial.md).

El uso de la API se ilustra en la siguiente llamada a la API REST de autocompletar. De este ejemplo se pueden extraer dos conclusiones: En primer lugar, al igual que en todas las consultas, la operación se realiza con una colección de documentos de un índice y la consulta incluye un parámetro **search**, que en este caso proporciona la consulta parcial. En segundo lugar, debe agregar **suggesterName** a la solicitud. Si no se define un proveedor de sugerencias en el índice, se producirá un error en la llamada a autocompletar o sugerencias.

```http
POST /indexes/myxboxgames/docs/autocomplete?search&api-version=2020-06-30
{
  "search": "minecraf",
  "suggesterName": "sg"
}
```

## <a name="sample-code"></a>Código de ejemplo

+ En el ejemplo de [Creación de la primera aplicación en C# (lección 3: Incorporación de buscar mientras se escribe)](tutorial-csharp-type-ahead-and-suggestions.md) se muestra una construcción de proveedor de sugerencias, consultas sugeridas, autocompletar y navegación por facetas. Este ejemplo de código se ejecuta en un servicio de espacio aislado de Azure Cognitive Search y usa un índice de hoteles cargado previamente, por lo que todo lo que tiene que hacer es presionar F5 para ejecutar la aplicación. No se necesita ninguna suscripción o inicio de sesión.

+ [DotNetHowToAutocomplete](https://github.com/Azure-Samples/search-dotnet-getting-started/tree/master/DotNetHowToAutocomplete) es otro ejemplo más antiguo que contiene tanto código de C# como de Java. También se muestra una construcción de proveedor de sugerencias, consultas sugeridas, autocompletar y navegación por facetas. Este ejemplo de código usa los datos de ejemplo [NYCJobs](https://github.com/Azure-Samples/search-dotnet-asp-net-mvc-jobs) hospedados. 

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre cómo se solicita la formulación, se recomienda el siguiente artículo.

> [!div class="nextstepaction"]
> [Incorporación de sugerencias y de la función de autocompletar al código de cliente](search-autocomplete-tutorial.md)