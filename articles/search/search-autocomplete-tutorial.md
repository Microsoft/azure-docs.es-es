---
title: Incorporación de sugerencias y de la función de autocompletar en un cuadro de búsqueda
titleSuffix: Azure Cognitive Search
description: Para habilitar acciones de consulta de búsqueda al escribir en Azure Cognitive Search, puede crear proveedores de sugerencias y formular solicitudes que rellenen automáticamente un cuadro de búsqueda con los términos o las frases completadas. También puede devolver las coincidencias sugeridas.
manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 04/15/2020
ms.custom: devx-track-javascript
ms.openlocfilehash: 2de282da56a40c92eacde84ac913be0ceacf9e2b
ms.sourcegitcommit: e71da24cc108efc2c194007f976f74dd596ab013
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87413024"
---
# <a name="add-autocomplete-and-suggestions-to-client-apps"></a>Incorporación de sugerencias y de la función de autocompletar a las aplicaciones cliente

La búsqueda mientras se escribe es una técnica común para mejorar la productividad de las consultas iniciadas por el usuario. En Azure Cognitive Search, esta experiencia se admite a través de la función de *autocompletar*, que finaliza un término o frase según la entrada parcial (completa "micro" con "Microsoft"). Otra forma son las *sugerencias*: una breve lista de documentos coincidentes (se devuelven los títulos de libros con un id. que son vínculos a una página de detalles). Tanto la función de autocompletar como las sugerencias se declaran en una coincidencia en el índice. El servicio no ofrecerá consultas que devuelvan cero resultados.

Para implementar estas experiencias en Azure Cognitive Search, necesitará lo siguiente:

+ Un *proveedor de sugerencias* en el back-end.
+ Una *consulta* que especifique la API de [Autocompletar](https://docs.microsoft.com/rest/api/searchservice/autocomplete) o [Sugerencias](https://docs.microsoft.com/rest/api/searchservice/suggestions) en la solicitud.
+ Un *control de interfaz de usuario* para controlar las interacciones de la búsqueda mientras se escribe en la aplicación cliente. Se recomienda usar una biblioteca de JavaScript existente para este fin.

En Azure Cognitive Search, las consultas autocompletadas y los resultados sugeridos se recuperan desde el índice de búsqueda, a partir de los campos seleccionados que se han registrado con un proveedor de sugerencias. Un proveedor de sugerencias forma parte del índice y especifica los campos que proporcionarán contenido que complete una consulta, sugiera un resultado o ambos. Cuando se crea y carga el índice, se crea internamente una estructura de datos del proveedor de sugerencias para almacenar los prefijos usados para encontrar coincidencias en consultas parciales. Para obtener sugerencias, la elección de los campos adecuados que sean únicos o, al menos, no repetitivos, es esencial para la experiencia. Para obtener más información, consulte [Creación de un proveedor de sugerencias](index-add-suggesters.md).

El resto de este artículo se centra en las consultas y el código de cliente. Usa JavaScript y C# para ilustrar los puntos clave. Se usan ejemplos de API de REST para presentar cada operación de manera concisa. Para obtener vínculos a ejemplos de código completos, vea [Pasos siguientes](#next-steps).

## <a name="set-up-a-request"></a>Configuración de una solicitud

Los elementos de una solicitud incluyen una de las API de buscar mientras se escribe, una consulta parcial y un proveedor de sugerencias. En el siguiente script se ilustran los componentes de una solicitud, y se usa la API de REST de autocompletar como ejemplo.

```http
POST /indexes/myxboxgames/docs/autocomplete?search&api-version=2020-06-30
{
  "search": "minecraf",
  "suggesterName": "sg"
}
```

**suggesterName** proporciona los campos con reconocimiento del proveedor de sugerencias para completar los términos o las sugerencias. En el caso concreto de las sugerencias, la lista de campos debe estar compuesta por los que ofrecen opciones claras entre los resultados coincidentes. En un sitio que vende juegos de equipo, el campo puede ser el título del juego.

El parámetro **search** proporciona la consulta parcial, donde los caracteres se insertan en la solicitud de consulta a través del control de autocompletar de jQuery. En el ejemplo anterior, "minecraf" es una ilustración estática de lo que podría pasar el control.

Las API no imponen requisitos de longitud mínima en la consulta parcial; podría tener un solo carácter. Sin embargo, la función de autocompletar de jQuery proporciona una longitud mínima. Un mínimo de dos o tres caracteres es lo más habitual.

Las coincidencias son del principio de un término, en cualquier parte de la cadena de entrada. Dado "the quick brown fox", tanto la función de autocompletar como las sugerencias coincidirán en las versiones parciales de "the", "quick", "brown" o "fox", pero no en los términos parciales infijos como "rown" u "ox". Además, cada coincidencia establece el ámbito de las expansiones de bajada. Una consulta parcial de "quick br" coincidirá con "quick brown" o "quick bread", pero ni "brown" ni "bread" por sí mismos coincidirán, a menos que "quick" les preceda.

### <a name="apis-for-search-as-you-type"></a>API para buscar mientras se escribe

Siga estos vínculos para las páginas de referencia del SDK de REST y .NET:

+ [API REST de sugerencias](https://docs.microsoft.com/rest/api/searchservice/suggestions) 
+ [API REST de autocompletar](https://docs.microsoft.com/rest/api/searchservice/autocomplete) 
+ [Método SuggestWithHttpMessagesAsync](https://docs.microsoft.com/dotnet/api/microsoft.azure.search.idocumentsoperations.suggestwithhttpmessagesasync?view=azure-dotnet)
+ [Método AutocompleteWithHttpMessagesAsync](https://docs.microsoft.com/dotnet/api/microsoft.azure.search.idocumentsoperations.autocompletewithhttpmessagesasync?view=azure-dotnet&viewFallbackFrom=azure-dotnet)

## <a name="structure-a-response"></a>Estructuración de una respuesta

Las respuestas para la función de autocompletar y las sugerencias son lo que cabría esperar para el patrón: [Autocompletar](https://docs.microsoft.com/rest/api/searchservice/autocomplete#response) devuelve una lista de términos, [Sugerencias](https://docs.microsoft.com/rest/api/searchservice/suggestions#response) devuelve términos además de un id. de documento para que pueda capturar el documento (use la API [Buscar documento](https://docs.microsoft.com/rest/api/searchservice/lookup-document) para obtener el documento específico de una página de detalles).

Las respuestas están formadas por los parámetros de la solicitud. Para la función de autocompletar, establezca [**autocompleteMode**](https://docs.microsoft.com/rest/api/searchservice/autocomplete#autocomplete-modes) para determinar si la finalización del texto se produce en uno o dos términos. En el caso de las sugerencias, el campo que se elige determina el contenido de la respuesta.

Para obtener sugerencias, debe refinar aún más la respuesta para evitar duplicados o lo que parece ser resultados no relacionados. Para controlar los resultados, incluya más parámetros en la solicitud. Los parámetros siguientes se aplican tanto a las sugerencias como a la función de autocompletar, pero es posible que sean más necesarios para las sugerencias, especialmente cuando un proveedor de sugerencias incluya varios campos.

| Parámetro | Uso |
|-----------|-------|
| **$select** | Si tiene varios elementos **sourceFields** en un proveedor de sugerencias, use **$select** para elegir qué campo contribuye a los valores (`$select=GameTitle`). |
| **searchFields** | Restringe la consulta a campos específicos. |
| **$filter** | Aplica criterios de coincidencia en el conjunto de resultados (`$filter=Category eq 'ActionAdventure'`). |
| **$top** | Limita los resultados a un número específico (`$top=5`).|

## <a name="add-user-interaction-code"></a>Adición de código de interacción del usuario

El rellenado automático de un término de consulta o la eliminación de una lista de vínculos coincidentes requiere código de interacción del usuario, normalmente JavaScript, que puede consumir solicitudes de orígenes externos, como consultas con autocompletar o sugerencias en un índice de Azure Search Cognitive.

Aunque podría escribir este código de forma nativa, es mucho más fácil usar funciones de la biblioteca de JavaScript existente. En este artículo se muestran dos, uno para las sugerencias y otro para la función de autocompletar. 

+ El [widget de autocompletar (interfaz de usuario de jQuery)](https://jqueryui.com/autocomplete/) se usa en el ejemplo de sugerencias. Puede crear un cuadro de búsqueda y luego hacer referencia a él en una función de JavaScript que use el widget de autocompletar. Las propiedades del widget establecen la longitud mínima de los caracteres de entrada de origen (una función de autocompletar o sugerencias) para que se realice la acción y el posicionamiento.

+ El [complemento de autocompletar de XDSoft](https://xdsoft.net/jqplugins/autocomplete/) se usa en el ejemplo de función de autocompletar.

Usamos estas bibliotecas para crear el cuadro de búsqueda que admite tanto las sugerencias como la función de autocompletar. Las entradas recopiladas en el cuadro de búsqueda se emparejan con acciones de autocompletar y sugerencias.

## <a name="suggestions"></a>Sugerencias

En esta sección se guía a través de una implementación de resultados sugeridos, empezando por la definición del cuadro de búsqueda. También se muestra cómo y el script que invoca la primera biblioteca de autocompletar de JavaScript a la que se hace referencia en este artículo.

### <a name="create-a-search-box"></a>Creación de un cuadro de búsqueda

Suponiendo que dispone de la [biblioteca de autocompletar de la interfaz de usuario de jQuery](https://jqueryui.com/autocomplete/) y un proyecto de MVC en C#, podría definir el cuadro de búsqueda mediante JavaScript en el archivo **index.cshtml**. Esta biblioteca agrega la interacción de buscar mientras se escribe al cuadro de búsqueda; para ello realiza llamadas asincrónicas al controlador de MVC para recuperar sugerencias.

En **Index.cshtml** de la carpeta \Views\Home, una línea para crear un cuadro de búsqueda podría ser la siguiente:

```html
<input class="searchBox" type="text" id="searchbox1" placeholder="search">
```

Este ejemplo es un cuadro de texto de entrada simple con una clase para establecer el estilo y un identificador al que JavaScript hará referencia, además de un texto de marcador de posición.  

En el mismo archivo, inserte JavaScript que haga referencia al cuadro de búsqueda. La siguiente función llama a la API de sugerencias, que solicita los documentos coincidentes sugeridos en función de las entradas de términos parciales:

```javascript
$(function () {
    $("#searchbox1").autocomplete({
        source: "/home/suggest?highlights=false&fuzzy=false&",
        minLength: 3,
        position: {
            my: "left top",
            at: "left-23 bottom+10"
        }
    });
});
```

En `source` se indica a la función de autocompletar de la interfaz de usuario de jQuery dónde obtener la lista de elementos para mostrar debajo del cuadro de búsqueda. Puesto que este es un proyecto de MVC, llama a la función **Suggest** en **HomeController.cs**, que contiene la lógica para devolver las sugerencias de consulta. Esta función también pasa algunos parámetros para controlar aspectos destacados, una coincidencia aproximada o un término. La API de JavaScript de la función Autocompletar agrega el parámetro de término.

`minLength: 3` garantiza que la recomendación solo se mostrará cuando haya al menos tres caracteres en el cuadro de búsqueda.

### <a name="enable-fuzzy-matching"></a>Habilitación de la coincidencia aproximada

La búsqueda aproximada permite obtener resultados basados en coincidencias cercanas incluso si el usuario escribe incorrectamente una palabra en el cuadro de búsqueda. La distancia de edición es 1, lo que significa que puede haber una discrepancia máxima de un carácter entre la entrada del usuario y una coincidencia. 

```javascript
source: "/home/suggest?highlights=false&fuzzy=true&",
```

### <a name="enable-highlighting"></a>Habilitación del resaltado

El resaltado aplica el estilo de fuente a los caracteres del resultado que corresponden a la entrada. Por ejemplo, si la entrada parcial es "micro", el resultado aparecería como **micro**soft, **micro**ámbito, etc. El resaltado se basa en los parámetros HighlightPreTag y HighlightPostTag, definidos en línea con la función de sugerencias.

```javascript
source: "/home/suggest?highlights=true&fuzzy=true&",
```

### <a name="suggest-function"></a>Función de sugerencias

Si usa C# y una aplicación MVC, el archivo **HomeController.cs** del directorio Controllers es el lugar donde puede crear una clase para los resultados sugeridos. En .NET, una función de sugerencias se basa en el [método DocumentsOperationsExtensions.Suggest](/dotnet/api/microsoft.azure.search.documentsoperationsextensions.suggest?view=azure-dotnet).

El método `InitSearch` crea un cliente del índice HTTP autenticado en el servicio Azure Cognitive Search. Para más información sobre el SDK de .NET, consulte [Cómo usar Azure Cognitive Search desde una aplicación .NET](https://docs.microsoft.com/azure/search/search-howto-dotnet-sdk).

```csharp
public ActionResult Suggest(bool highlights, bool fuzzy, string term)
{
    InitSearch();

    // Call suggest API and return results
    SuggestParameters sp = new SuggestParameters()
    {
        Select = HotelName,
        SearchFields = HotelName,
        UseFuzzyMatching = fuzzy,
        Top = 5
    };

    if (highlights)
    {
        sp.HighlightPreTag = "<b>";
        sp.HighlightPostTag = "</b>";
    }

    DocumentSuggestResult resp = _indexClient.Documents.Suggest(term, "sg", sp);

    // Convert the suggest query results to a list that can be displayed in the client.
    List<string> suggestions = resp.Results.Select(x => x.Text).ToList();
    return new JsonResult
    {
        JsonRequestBehavior = JsonRequestBehavior.AllowGet,
        Data = suggestions
    };
}
```

La función Sugerir toma dos parámetros que determinar si se devuelven resaltados de aciertos o si se usa la coincidencia aproximada además de la entrada del término de búsqueda. El método crea un [objeto SuggestParameters](https://docs.microsoft.com/dotnet/api/microsoft.azure.search.models.suggestparameters?view=azure-dotnet) que luego se pasa a la API de Suggest. Luego el resultado se convierte en JSON para poder mostrarlo en el cliente.

## <a name="autocomplete"></a>Autocompletar

Hasta ahora, el código de experiencia de usuario de la búsqueda se ha centrado en las sugerencias. El bloque de código siguiente muestra la función de autocompletar de la interfaz de usuario de jQuery que pasa una solicitud de autocompletar de Azure Cognitive Search. Igual que con las sugerencias, en una aplicación C#, el código que admite la interacción del usuario se encuentra en **index.cshtml**.

```javascript
$(function () {
    // using modified jQuery Autocomplete plugin v1.2.6 https://xdsoft.net/jqplugins/autocomplete/
    // $.autocomplete -> $.autocompleteInline
    $("#searchbox1").autocompleteInline({
        appendMethod: "replace",
        source: [
            function (text, add) {
                if (!text) {
                    return;
                }

                $.getJSON("/home/autocomplete?term=" + text, function (data) {
                    if (data && data.length > 0) {
                        currentSuggestion2 = data[0];
                        add(data);
                    }
                });
            }
        ]
    });

    // complete on TAB and clear on ESC
    $("#searchbox1").keydown(function (evt) {
        if (evt.keyCode === 9 /* TAB */ && currentSuggestion2) {
            $("#searchbox1").val(currentSuggestion2);
            return false;
        } else if (evt.keyCode === 27 /* ESC */) {
            currentSuggestion2 = "";
            $("#searchbox1").val("");
        }
    });
});
```

### <a name="autocomplete-function"></a>Función de autocompletar

La función de autocompletar se basa en el [método DocumentsOperationsExtensions.Autocomplete](https://docs.microsoft.com/dotnet/api/microsoft.azure.search.documentsoperationsextensions.autocomplete?view=azure-dotnet). Igual que con las sugerencias, este bloque de código se incluye en el archivo **HomeController.cs**.

```csharp
public ActionResult AutoComplete(string term)
{
    InitSearch();
    //Call autocomplete API and return results
    AutocompleteParameters ap = new AutocompleteParameters()
    {
        AutocompleteMode = AutocompleteMode.OneTermWithContext,
        UseFuzzyMatching = false,
        Top = 5
    };
    AutocompleteResult autocompleteResult = _indexClient.Documents.Autocomplete(term, "sg", ap);

    // Convert the Suggest results to a list that can be displayed in the client.
    List<string> autocomplete = autocompleteResult.Results.Select(x => x.Text).ToList();
    return new JsonResult
    {
        JsonRequestBehavior = JsonRequestBehavior.AllowGet,
        Data = autocomplete
    };
}
```

La función Autocomplete toma la entrada del término de búsqueda. El método crea un [objeto AutoCompleteParameters](https://docs.microsoft.com/rest/api/searchservice/autocomplete). Luego el resultado se convierte en JSON para poder mostrarlo en el cliente.

## <a name="next-steps"></a>Pasos siguientes

Siga estos vínculos para obtener instrucciones completas o código que muestra las experiencias de buscar mientras se escribe. En ambos ejemplos de código se incluyen implementaciones híbridas de las sugerencias y la función de autocompletar.

+ [Tutorial: cree su primera aplicación en C# (lección 3)](tutorial-csharp-type-ahead-and-suggestions.md)
+ [Ejemplo de código C#: azure-search-dotnet-samples/create-first-app/3-add-typeahead/](https://github.com/Azure-Samples/azure-search-dotnet-samples/tree/master/create-first-app/3-add-typeahead)
+ [C# y JavaScript con código de ejemplo de REST en paralelo](https://github.com/Azure-Samples/search-dotnet-getting-started/tree/master/DotNetHowToAutocomplete)