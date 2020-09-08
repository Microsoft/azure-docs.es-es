---
title: Tutorial de C# sobre la función Autocompletar y las sugerencias
titleSuffix: Azure Cognitive Search
description: Agregue la función Autocompletar y sugerencias para recopilar entradas de términos de búsqueda de los usuarios mediante la lista desplegable. Este tutorial se basa en un proyecto de hoteles existente.
manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.service: cognitive-search
ms.topic: tutorial
ms.date: 07/15/2020
ms.custom: devx-track-javascript, devx-track-csharp
ms.openlocfilehash: 1afeca4f627236c5172dd07a44751015c16d2f58
ms.sourcegitcommit: 4a7a4af09f881f38fcb4875d89881e4b808b369b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/04/2020
ms.locfileid: "89462028"
---
# <a name="tutorial-add-autocomplete-and-suggestions-using-the-net-sdk"></a>Tutorial: Incorporación de la función Autocompletar y sugerencias mediante el SDK de .NET

Obtenga información sobre cómo implementar la función Autocompletar (escritura anticipada de consultas y documentos sugeridos) cuando un usuario empieza a escribir en el cuadro de búsqueda. En este tutorial, se mostrarán las consultas autocompletadas y los resultados de sugerencias por separado y luego juntos. Un usuario puede que solo tenga que escribir dos o tres caracteres para buscar todos los resultados disponibles.

En este tutorial, aprenderá a:
> [!div class="checklist"]
> * Adición de sugerencias
> * Agregar resaltado para las sugerencias
> * Adición de la función Autocompletar
> * Combinar la función Autocompletar y las sugerencias

## <a name="prerequisites"></a>Prerrequisitos

Este tutorial forma parte de una serie y se basa en el proyecto de paginación creado en [Tutorial de C#: Paginación de los resultados de la búsqueda: Azure Cognitive Search](tutorial-csharp-paging.md).

Como alternativa, puede descargar y ejecutar la solución para este tutorial específico: [3-add-typeahead](https://github.com/Azure-Samples/azure-search-dotnet-samples/tree/master/create-first-app/v10/3-add-typeahead).

## <a name="add-suggestions"></a>Adición de sugerencias

Comencemos con el caso más simple de la oferta de alternativas para el usuario: una lista desplegable de sugerencias.

1. En el archivo index.cshtml, cambie `@id` de la instrucción **TextBoxFor** a **azureautosuggest**.

    ```cs
     @Html.TextBoxFor(m => m.searchText, new { @class = "searchBox", @id = "azureautosuggest" }) <input value="" class="searchBoxSubmit" type="submit">
    ```

2. Siguiendo esta aclaración, tras el **&lt;/div&gt;** del cierre, escriba este script. Este script aprovecha el [widget Autocomplete](https://api.jqueryui.com/autocomplete/) de la biblioteca de la interfaz de usuario de jQuery de código abierto para presentar la lista desplegable de resultados sugeridos. 

    ```javascript
    <script>
        $("#azureautosuggest").autocomplete({
            source: "/Home/Suggest?highlights=false&fuzzy=false",
            minLength: 2,
            position: {
                my: "left top",
                at: "left-23 bottom+10"
            }
        });
    </script>
    ```

    El identificador "azureautosuggest" conecta el script anterior al cuadro de búsqueda. La opción de origen del widget está establecida en un método Suggest que llama a la API Suggest con dos parámetros de consulta: **highlights** y **fuzzy**, ambos establecidos en false en este caso. Además, se necesita un mínimo de dos caracteres para desencadenar la búsqueda.

### <a name="add-references-to-jquery-scripts-to-the-view"></a>Incorporación de referencias a scripts de jQuery para la vista

1. Para acceder a la biblioteca de jQuery, cambie la sección &lt;head&gt; del archivo de vista por el código siguiente:

    ```cs
    <head>
        <meta charset="utf-8">
        <title>Typeahead</title>
        <link href="https://code.jquery.com/ui/1.12.1/themes/start/jquery-ui.css"
              rel="stylesheet">
        <script src="https://code.jquery.com/jquery-1.10.2.js"></script>
        <script src="https://code.jquery.com/ui/1.12.1/jquery-ui.js"></script>

        <link rel="stylesheet" href="~/css/hotels.css" />
    </head>
    ```

2. Dado que estamos introduciendo una nueva referencia de jQuery, también es necesario quitar o comentar la referencia de jQuery predeterminada en el archivo _Layout.cshtml (en la carpeta **Views/Shared**). Busque las líneas siguientes y marque como comentario la primera línea del script como se muestra. Este cambio evita conflictos de referencias a jQuery.

    ```html
    <environment include="Development">
        <!-- <script src="~/lib/jquery/dist/jquery.js"></script> -->
        <script src="~/lib/bootstrap/dist/js/bootstrap.js"></script>
        <script src="~/js/site.js" asp-append-version="true"></script>
    </environment>
    ```

    Ahora se pueden usar las funciones jQuery de Autocompletar predefinidas.

### <a name="add-the-suggest-action-to-the-controller"></a>Incorporación de la acción Suggest al controlador

1. En el controlador principal, agregue la acción **Suggest** (por ejemplo, después de la acción **Page**).

    ```cs
        public async Task<ActionResult> Suggest(bool highlights, bool fuzzy, string term)
        {
            InitSearch();

            // Setup the suggest parameters.
            var parameters = new SuggestParameters()
            {
                UseFuzzyMatching = fuzzy,
                Top = 8,
            };

            if (highlights)
            {
                parameters.HighlightPreTag = "<b>";
                parameters.HighlightPostTag = "</b>";
            }

            // Only one suggester can be specified per index. It is defined in the index schema.
            // The name of the suggester is set when the suggester is specified by other API calls.
            // The suggester for the hotel database is called "sg", and simply searches the hotel name.
            DocumentSuggestResult<Hotel> suggestResult = await _indexClient.Documents.SuggestAsync<Hotel>(term, "sg", parameters);

            // Convert the suggest query results to a list that can be displayed in the client.
            List<string> suggestions = suggestResult.Results.Select(x => x.Text).ToList();

            // Return the list of suggestions.
            return new JsonResult(suggestions);
        }
    ```

    El parámetro **Top** especifica cuántos resultados deben devolverse (si no se especifica, el valor predeterminado es 5). Un _proveedor de sugerencias_ se especifica en el índice de Azure, que se realiza cuando los datos se configuran y no por una aplicación cliente como en este tutorial. En este caso, el proveedor de sugerencias se denomina "sg" y busca el campo **HotelName** y ningún otro. 

    La coincidencia aproximada permite que se incluyan las anomalías en la salida, hasta una edición de distancia. Si el parámetro **highlights** se establece en true, las etiquetas HTML en negrita se agregan a la salida. Se establecerá estos dos parámetros en true en la sección siguiente.

2. Es posible que obtenga algunos errores de sintaxis. Si es así, agregue las siguientes dos instrucciones **using** en la parte superior del archivo.

    ```cs
    using System.Collections.Generic;
    using System.Linq;
    ```

3. Ejecute la aplicación. ¿Se obtienen diversas de opciones al escribir "po", por ejemplo? Ahora pruebe con "pa".

    ![Al escribir "po", se revelan dos sugerencias](./media/tutorial-csharp-create-first-app/azure-search-suggest-po.png)

    Tenga en cuenta que las letras que escriba _deben_ comenzar con una palabra y simplemente incluirse dentro de la palabra.

4. En el script de la vista, establezca **&fuzzy** en true y vuelva a ejecutar la aplicación. Ahora, escriba "pedido". Observe que en la búsqueda se da por supuesto que obtuvo una letra incorrecta.
 
    ![Escriba "pa" con el parámetro fuzzy establecido en true](./media/tutorial-csharp-create-first-app/azure-search-suggest-fuzzy.png)

    Si está interesado, la [sintaxis de consulta de Lucene en Azure Cognitive Search](./query-lucene-syntax.md) describe la lógica utilizada en las búsquedas aproximadas en detalle.

## <a name="add-highlighting-to-the-suggestions"></a>Agregar resaltado para las sugerencias

Se puede mejorar la apariencia de las sugerencias para el usuario si se establece el parámetro **highlights** en true. Sin embargo, primero debemos agregar código a la vista para mostrar el texto en negrita.

1. En la vista (index.cshtml), agregue el siguiente script después del script **azureautosuggest** que especificó antes.

    ```javascript
    <script>
        var updateTextbox = function (event, ui) {
            var result = ui.item.value.replace(/<\/?[^>]+(>|$)/g, "");
            $("#azuresuggesthighlights").val(result);
            return false;
        };

        $("#azuresuggesthighlights").autocomplete({
            html: true,
            source: "/home/suggest?highlights=true&fuzzy=false&",
            minLength: 2,
            position: {
                my: "left top",
                at: "left-23 bottom+10"
            },
            select: updateTextbox,
            focus: updateTextbox
        }).data("ui-autocomplete")._renderItem = function (ul, item) {
            return $("<li></li>")
                .data("item.autocomplete", item)
                .append("<a>" + item.label + "</a>")
                .appendTo(ul);
        };
    </script>
    ```

2. Ahora cambie el identificador del cuadro de texto para que quede como sigue.

    ```cs
    @Html.TextBoxFor(m => m.searchText, new { @class = "searchBox", @id = "azuresuggesthighlights" }) <input value="" class="searchBoxSubmit" type="submit">
    ```

3. Vuelva a ejecutar la aplicación y debería ver el texto que especificó en negrita en las sugerencias. Por ejemplo, pruebe a escribir "pa".
 
    ![Escritura de "pa" con resaltado](./media/tutorial-csharp-create-first-app/azure-search-suggest-highlight.png)

4. La lógica utilizada en el script de resaltado anterior no es infalible. Si escribe un término que aparece dos veces en el mismo nombre, los resultados en negrita no son lo que usted desearía. Pruebe a escribir "mo".

    Una de las preguntas a las que un desarrollador debe responder es, cuando es un script funciona "lo suficientemente bien" y cuándo deben solucionarse también sus peculiaridades. No seguiremos más con el resaltado en este tutorial, pero es importante considerar la búsqueda de un algoritmo preciso si el resaltado no resulta efectivo para sus datos. Para más información, consulte [Resaltado de referencias](search-pagination-page-layout.md#hit-highlighting).

## <a name="add-autocomplete"></a>Adición de la función Autocompletar

Otra variación, que es ligeramente diferente de las sugerencias, es la función Autocompletar (a veces denominada "escritura anticipada") que completa un término de consulta. Nuevamente, comenzaremos con la implementación más sencilla antes de mejorar la experiencia del usuario.

1. Escriba el siguiente script en la vista, después de los scripts anteriores.

    ```javascript
    <script>
        $("#azureautocompletebasic").autocomplete({
            source: "/Home/Autocomplete",
            minLength: 2,
            position: {
                my: "left top",
                at: "left-23 bottom+10"
            }
        });
    </script>
    ```

2. Ahora cambie el identificador del cuadro de texto para que quede como sigue.

    ```cs
    @Html.TextBoxFor(m => m.searchText, new { @class = "searchBox", @id = "azureautocompletebasic" }) <input value="" class="searchBoxSubmit" type="submit">
    ```

3. En el controlador principal es necesario que escriba la acción **Autocomplete**, por ejemplo, a continuación de la acción **Suggest**.

    ```cs
        public async Task<ActionResult> AutoComplete(string term)
        {
            InitSearch();

            // Setup the autocomplete parameters.
            var ap = new AutocompleteParameters()
            {
                AutocompleteMode = AutocompleteMode.OneTermWithContext,
                Top = 6
            };
            AutocompleteResult autocompleteResult = await _indexClient.Documents.AutocompleteAsync(term, "sg", ap);

            // Convert the results to a list that can be displayed in the client.
            List<string> autocomplete = autocompleteResult.Results.Select(x => x.Text).ToList();

            // Return the list.
            return new JsonResult(autocomplete);
        }
    ```

    Tenga en cuenta que estamos usando la misma función de *proveedor de sugerencias*, denominada "sg", en la búsqueda de autocompletar que cuando obtuvimos la sugerencias (por lo que intentamos solo para autocompletar los nombres de hotel).

    Hay diversas opciones de **AutocompleteMode** y estamos usando **OneTermWithContext**. Consulte [Autocomplete API](/rest/api/searchservice/autocomplete) para obtener una descripción de las opciones adicionales.

4. Ejecute la aplicación. Observe cómo la variedad de opciones que se muestra en la lista desplegable son las palabras individuales. Pruebe a escribir palabras que empiezan por "re". Tenga en cuenta cómo se reduce el número de opciones a medida que se escriben más letras.

    ![Escritura con la función básica de autocompletar](./media/tutorial-csharp-create-first-app/azure-search-suggest-autocompletebasic.png)

    Tal como está, el script de sugerencias que ejecutó anteriormente es probablemente más útil que este script de la función Autocompletar. Para facilitar fácil, es mejor agregarlo a la búsqueda de sugerencias.

## <a name="combine-autocompletion-and-suggestions"></a>Combinar la función Autocompletar y las sugerencias

La combinación de la función Autocompletar y de las sugerencias es la más compleja de nuestras opciones y, probablemente, proporcione la mejor experiencia de usuario. Lo que queremos es mostrar, junto con el texto que se escribe, la primera opción de Azure Cognitive Search para autocompletar el texto. Además, queremos un intervalo de sugerencias como una lista desplegable.

Hay bibliotecas que ofrecen esta funcionalidad: a menudo denominada "Autocompletar insertada" o un nombre similar. Sin embargo, vamos a implementar de forma nativa esta característica, para que pueda ver lo que está ocurriendo. Vamos a empezar a trabajar en el controlador en primer lugar en este ejemplo.

1. Tenemos que agregar una acción al controlador que devuelva un solo resultado de Autocompletar, junto con un número especificado de sugerencias. Llamaremos a esta acción **AutocompleteAndSuggest**. En el controlador principal, agregue la siguiente acción, después de las otras acciones nuevas.

    ```cs
        public async Task<ActionResult> AutocompleteAndSuggest(string term)
        {
            InitSearch();

            // Setup the type-ahead search parameters.
            var ap = new AutocompleteParameters()
            {
                AutocompleteMode = AutocompleteMode.OneTermWithContext,
                Top = 1,
            };
            AutocompleteResult autocompleteResult = await _indexClient.Documents.AutocompleteAsync(term, "sg", ap);

            // Setup the suggest search parameters.
            var sp = new SuggestParameters()
            {
                Top = 8,
            };

            // Only one suggester can be specified per index. The name of the suggester is set when the suggester is specified by other API calls.
            // The suggester for the hotel database is called "sg", and it searches only the hotel name.
            DocumentSuggestResult<Hotel> suggestResult = await _indexClient.Documents.SuggestAsync<Hotel>(term, "sg", sp);

            // Create an empty list.
            var results = new List<string>();

            if (autocompleteResult.Results.Count > 0)
            {
                // Add the top result for type-ahead.
                results.Add(autocompleteResult.Results[0].Text);
            }
            else
            {
                // There were no type-ahead suggestions, so add an empty string.
                results.Add("");
            }
            for (int n = 0; n < suggestResult.Results.Count; n++)
            {
                // Now add the suggestions.
                results.Add(suggestResult.Results[n].Text);
            }

            // Return the list.
            return new JsonResult(results);
        }
    ```

    Se devuelve una opción de Autocompletar en la parte superior de la lista de **resultados**, seguida de todas las sugerencias.

2. En la vista, se implementa en primer lugar un truco para que una palabra de Autocompletar de color gris claro se represente correctamente debajo del texto más oscuro que especifique el usuario. HTML incluye la colocación relativa para este propósito. Cambie la instrucción **TextBoxFor** (y sus instrucciones &lt;div&gt; que la rodean) por lo siguiente, teniendo en cuenta que un segundo cuadro de búsqueda que se identifica como **debajo** está justo debajo de nuestro cuadro de búsqueda normal, al insertar este cuadro de búsqueda 39 píxeles fuera de su ubicación predeterminada.

    ```cs
    <div id="underneath" class="searchBox" style="position: relative; left: 0; top: 0">
    </div>

    <div id="searchinput" class="searchBoxForm" style="position: relative; left: 0; top: -39px">
        @Html.TextBoxFor(m => m.searchText, new { @class = "searchBox", @id = "azureautocomplete" }) <input value="" class="searchBoxSubmit" type="submit">
    </div>
    ```

    Tenga en cuenta que estamos cambiando el identificador de nuevo, por **azureautocomplete** en este caso.

3. También en la vista, escriba el siguiente script, después de todo los scripts que ha escrito hasta ahora. Hay bastante que hacer.

    ```javascript
    <script>
        $('#azureautocomplete').autocomplete({
            delay: 500,
            minLength: 2,
            position: {
                my: "left top",
                at: "left-23 bottom+10"
            },

            // Use Ajax to set up a "success" function.
            source: function (request, response) {
                var controllerUrl = "/Home/AutoCompleteAndSuggest?term=" + $("#azureautocomplete").val();
                $.ajax({
                    url: controllerUrl,
                    dataType: "json",
                    success: function (data) {
                        if (data && data.length > 0) {

                            // Show the autocomplete suggestion.
                            document.getElementById("underneath").innerHTML = data[0];

                            // Remove the top suggestion as it is used for inline autocomplete.
                            var array = new Array();
                            for (var n = 1; n < data.length; n++) {
                                array[n - 1] = data[n];
                            }

                            // Show the drop-down list of suggestions.
                            response(array);
                        } else {

                            // Nothing is returned, so clear the autocomplete suggestion.
                            document.getElementById("underneath").innerHTML = "";
                        }
                    }
                });
            }
        });

        // Complete on TAB.
        // Clear on ESC.
        // Clear if backspace to less than 2 characters.
        // Clear if any arrow key hit as user is navigating the suggestions.
        $("#azureautocomplete").keydown(function (evt) {

            var suggestedText = document.getElementById("underneath").innerHTML;
            if (evt.keyCode === 9 /* TAB */ && suggestedText.length > 0) {
                $("#azureautocomplete").val(suggestedText);
                return false;
            } else if (evt.keyCode === 27 /* ESC */) {
                document.getElementById("underneath").innerHTML = "";
                $("#azureautocomplete").val("");
            } else if (evt.keyCode === 8 /* Backspace */) {
                if ($("#azureautocomplete").val().length < 2) {
                    document.getElementById("underneath").innerHTML = "";
                }
            } else if (evt.keyCode >= 37 && evt.keyCode <= 40 /* Any arrow key */) {
                document.getElementById("underneath").innerHTML = "";
            }
        });

        // Character replace function.
        function setCharAt(str, index, chr) {
            if (index > str.length - 1) return str;
            return str.substr(0, index) + chr + str.substr(index + 1);
        }

        // This function is needed to clear the "underneath" text when the user clicks on a suggestion, and to
        // correct the case of the autocomplete option when it does not match the case of the user input.
        // The interval function is activated with the input, blur, change, or focus events.
        $("#azureautocomplete").on("input blur change focus", function (e) {

            // Set a 2 second interval duration.
            var intervalDuration = 2000, 
                interval = setInterval(function () {

                    // Compare the autocorrect suggestion with the actual typed string.
                    var inputText = document.getElementById("azureautocomplete").value;
                    var autoText = document.getElementById("underneath").innerHTML;

                    // If the typed string is longer than the suggestion, then clear the suggestion.
                    if (inputText.length > autoText.length) {
                        document.getElementById("underneath").innerHTML = "";
                    } else {

                        // If the strings match, change the case of the suggestion to match the case of the typed input.
                        if (autoText.toLowerCase().startsWith(inputText.toLowerCase())) {
                            for (var n = 0; n < inputText.length; n++) {
                                autoText = setCharAt(autoText, n, inputText[n]);
                            }
                            document.getElementById("underneath").innerHTML = autoText;

                        } else {
                            // The strings do not match, so clear the suggestion.
                            document.getElementById("underneath").innerHTML = "";
                        }
                    }

                    // If the element loses focus, stop the interval checking.
                    if (!$input.is(':focus')) clearInterval(interval);

                }, intervalDuration);
        });
    </script>
    ```

    Tenga en cuenta el uso inteligente de la función **interval** para borrar tanto el texto subyacente cuando ya no coincida con lo que escriba el usuario, y también para establecer las mismas mayúsculas y minúsculas que el usuario esté escribiendo (ya que "pa" coincide con "PA", "pA", "Pa" en las búsquedas), para que el texto superpuesto esté ordenado.

    Lea los comentarios en el script para obtener una descripción más completa.

4. Por último, necesitamos realizar un ajuste sencillo a dos clases HTML para que sean transparentes. Agregue la siguiente línea a las clases **searchBoxForm** y **searchBox**, en el archivo hotels.css.

    ```html
        background: rgba(0,0,0,0);
    ```

5. Después, ejecute la aplicación. En el cuadro de búsqueda, escriba "pa". ¿Obtiene "palacio" como la sugerencia de Autocompletar, junto con dos hoteles que contienen "pa"?

    ![Escritura con sugerencias y Autocompletar insertada](./media/tutorial-csharp-create-first-app/azure-search-suggest-autocomplete.png)

6. Pruebe la tabulación para aceptar la sugerencia de Autocompletar y pruebe a seleccionar sugerencias con las teclas de dirección y la tecla tab, e inténtelo de nuevo con el mouse y un solo clic. Compruebe que el script controla perfectamente todas estas situaciones.

    Podría decidir que resulta más sencillo cargar una biblioteca que ofrezca esta característica por usted, pero ahora ya conoce al menos una forma de conseguir que Autocompletar insertada funcione.

## <a name="takeaways"></a>Puntos clave

Tenga en cuenta las siguientes conclusiones de este proyecto:

* La función Autocompletar (también conocido como "escritura anticipada") y las sugerencias pueden facilitar que el usuario escriba solo algunas teclas para encontrar exactamente lo desean.
* El uso de Autocompletar y las sugerencias conjuntamente puede proporcionar una mejor experiencia al usuario.
* Pruebe siempre las funciones de Autocompletar con todos los formularios de entrada.
* El uso de la función **setInterval** puede ser útil para comprobar y corregir los elementos de la interfaz de usuario.

## <a name="next-steps"></a>Pasos siguientes

En el siguiente tutorial, echaremos un vistazo a otra manera de mejorar la experiencia del usuario, usando facetas para restringir las búsquedas con un solo clic.

> [!div class="nextstepaction"]
> [Tutorial de C#: Uso de facetas para facilitar la navegación en Azure Cognitive Search](tutorial-csharp-facets.md)