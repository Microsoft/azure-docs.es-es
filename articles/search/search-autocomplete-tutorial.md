---
title: Agregar sugerencias y Autocompletar en un cuadro de búsqueda - Azure Search
description: Habilitar acciones de consulta de escritura anticipada en Azure Search mediante la creación de proveedores de sugerencias y formular las solicitudes que rellene un cuadro de búsqueda con completado términos o frases.
manager: pablocas
author: mrcarter8
services: search
ms.service: search
ms.devlang: NA
ms.topic: conceptual
ms.date: 05/02/2019
ms.author: mcarter
ms.custom: seodec2018
ms.openlocfilehash: 9385ed5127b8cc1aaf84c887b652fd8970883ba6
ms.sourcegitcommit: bb85a238f7dbe1ef2b1acf1b6d368d2abdc89f10
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/10/2019
ms.locfileid: "65524026"
---
# <a name="add-suggestions-or-autocomplete-to-your-azure-search-application"></a>Agregar sugerencias o Autocompletar a la aplicación de Azure Search

En este artículo, aprenderá a usar [sugerencias](https://docs.microsoft.com/rest/api/searchservice/suggestions) y [Autocompletar](https://docs.microsoft.com/rest/api/searchservice/autocomplete) para crear un cuadro de búsqueda eficaz que admite los comportamientos de búsqueda como-al escribir.

+ *Sugerencias* sugerido resultados generados mientras escribe, donde cada sugerencia es un resultado único desde el índice que coincide con lo que ha escrito hasta ahora. 

+ *Autocompletar* "finalice" de la palabra o frase que un usuario está escribiendo actualmente. En lugar de devolver los resultados, se completa una consulta, que, a continuación, puede ejecutar para devolver resultados. Al igual que con sugerencias, una palabra completa o frase en una consulta se basa en una coincidencia en el índice. El servicio no ofrece las consultas que devuelven cero da como resultado en el índice.

Puede descargar y ejecutar el código de ejemplo **DotNetHowToAutocomplete** para evaluar estas características. El código de ejemplo tiene como destino un índice creado previamente rellenado con [NYCJobs demostración datos](https://github.com/Azure-Samples/search-dotnet-asp-net-mvc-jobs). Contiene el índice NYCJobs un [construcción del proveedor de sugerencias](index-add-suggesters.md), que es un requisito para el uso de sugerencias o Autocompletar. Puede usar el índice preparado hospedado en un servicio de espacio aislado o [llenar su propio índice](#configure-app) mediante un cargador de datos en la solución de ejemplo NYCJobs. 

El **DotNetHowToAutocomplete** muestra sugerencias y Autocompletar, tanto en C# y las versiones de lenguaje de JavaScript. C#pueden ir a los desarrolladores a través de una aplicación basada en ASP.NET MVC que usa el [SDK para .NET de Azure Search](https://aka.ms/search-sdk). La lógica para realizar llamadas de consulta sugeridos y Autocompletar puede encontrarse en el archivo HomeController.cs. Los desarrolladores de JavaScript se pueden buscar la lógica de consulta equivalente en IndexJavaScript.cshtml, que incluye llamadas directas a la [API de REST de Azure Search](https://docs.microsoft.com/rest/api/searchservice/). 

Para ambas versiones de idioma, la experiencia del usuario de front-end se basa en el [jQuery UI](https://jqueryui.com/autocomplete/) y [XDSoft](https://xdsoft.net/jqplugins/autocomplete/) bibliotecas. Estas bibliotecas se usa para crear el cuadro de búsqueda que admiten las sugerencias y Autocompletar. Las entradas que se recopilan en el cuadro de búsqueda se emparejan con sugerencias y las acciones de Autocompletar, como los que tal como se define en HomeController.cs o IndexJavaScript.cshtml.

Este ejercicio le guiará a través de las siguientes tareas:

> [!div class="checklist"]
> * Implementar un cuadro de entrada de búsqueda en JavaScript y emitir solicitudes para coincidencias sugeridas o términos de autocompletado
> * En C#, definir sugerencias y las acciones de Autocompletar en HomeController.cs
> * En JavaScript, llamar a las API de REST directamente para proporcionar la misma funcionalidad

## <a name="prerequisites"></a>Requisitos previos

Un servicio Azure Search es opcional para este ejercicio porque la solución utiliza un servicio de espacio aislado en vivo que hospeda un índice de la demostración NYCJobs preparado. Si desea ejecutar este ejemplo en su propio servicio de búsqueda, vea [índice configurar trabajos de Nueva York](#configure-app) para obtener instrucciones.

* [Visual Studio 2017](https://visualstudio.microsoft.com/downloads/), cualquier edición. Se han probado código de ejemplo e instrucciones en la edición Community Edition gratuita.

* Descargue el [DotNetHowToAutoComplete ejemplo](https://github.com/Azure-Samples/search-dotnet-getting-started/tree/master/DotNetHowToAutocomplete).

El ejemplo está completo, cobertura sugerencias, Autocompletar, la navegación por facetas y almacenamiento en caché del lado cliente. Debe revisar el archivo Léame y comentarios para obtener una descripción completa de lo que ofrece el ejemplo.

## <a name="run-the-sample"></a>Ejecución del ejemplo

1. Abra **AutocompleteTutorial.sln** en Visual Studio. La solución contiene un proyecto de MVC de ASP.NET con una conexión con el índice de la demostración de trabajos de Nueva York.

2. Presione F5 para ejecutar el proyecto y cargar la página en el explorador que prefiera.

En la parte superior, verá una opción para seleccionar C# o JavaScript. La C# opción llama a HomeController desde el explorador y usa el SDK de .NET de Azure Search para recuperar los resultados. 

La opción de JavaScript llama a la API REST de Azure Search directamente desde el explorador. Por lo general, esta opción tendrá un rendimiento considerablemente mejor porque quita el controlador del flujo. Puede elegir la opción que mejor se adapte a sus necesidades y a sus preferencias de lenguaje. En la página aparecen varios ejemplos de la función Autocompletar con ayuda para cada uno de ellos. En cada ejemplo hay texto de ejemplo recomendado que puede probar.  

Para probar, empiece a escribir algunas letras en cada cuadro de búsqueda para ver lo que sucede.

## <a name="search-box"></a>Cuadro de búsqueda

Para ambos C# y las versiones de JavaScript, la implementación del cuadro de búsqueda es exactamente el mismo. 

Abra el **Index.cshtml** archivo bajo la carpeta \Views\Home para ver el código:

```html
<input class="searchBox" type="text" id="example1a" placeholder="search">
```

Se trata de un cuadro de texto de entrada simple con una clase para establecer el estilo, un identificador para hacer referencia a JavaScript y el texto de marcador de posición.  La magia está en el código JavaScript incrustado.

El C# ejemplo de lenguaje usa JavaScript en Index.cshtml para aprovechar la [biblioteca de jQuery UI Autocompletar](https://jqueryui.com/autocomplete/). Esta biblioteca agrega la experiencia de Autocompletar en el cuadro de búsqueda mediante la realización de llamadas asincrónicas al controlador de MVC para recuperar sugerencias. La versión del lenguaje JavaScript está en IndexJavaScript.cshtml. Incluye el siguiente script para la barra de búsqueda, así como llamadas de API de REST a Azure Search.

Veamos el código de JavaScript para el primer ejemplo, que llama a la función de Autocompletar de la interfaz de usuario, pasando en una solicitud para obtener sugerencias de jQuery:

```javascript
$(function () {
    $("#example1a").autocomplete({
        source: "/home/suggest?highlights=false&fuzzy=false&",
        minLength: 3,
        position: {
            my: "left top",
            at: "left-23 bottom+10"
        }
    });
});
```

El código anterior se ejecuta en el explorador carga la página Configuración de Autocompletar de jQuery UI para el cuadro de entrada "example1a".  `minLength: 3` garantiza que la recomendación solo se mostrará cuando haya al menos tres caracteres en el cuadro de búsqueda.  El valor de origen es importante:

```javascript
source: "/home/suggest?highlights=false&fuzzy=false&",
```

La línea anterior indica dónde obtener la lista de elementos que se va a mostrar en el cuadro de búsqueda de la función de Autocompletar de la interfaz de usuario de jQuery. Puesto que se trata de un proyecto de MVC, llama a la función sugerir en HomeController.cs que contiene la lógica para devolver las sugerencias de consulta (más información acerca de sugerir en la sección siguiente). Esta función también pasa parámetros algunos aspectos destacados de control, la coincidencia aproximada y término. La API de JavaScript de la función Autocompletar agrega el parámetro de término.

### <a name="extending-the-sample-to-support-fuzzy-matching"></a>Ampliación del ejemplo para admitir la coincidencia aproximada

La búsqueda aproximada permite obtener resultados basados en coincidencias cercanas incluso si el usuario escribe incorrectamente una palabra en el cuadro de búsqueda. Aunque no es necesario, mejora considerablemente la solidez de una experiencia de escritura anticipada. Para probar esto, cambiemos la línea de asunto para habilitar la coincidencia aproximada.

Cambie la línea siguiente de esto:

```javascript
source: "/home/suggest?highlights=false&fuzzy=false&",
```

a esto:

```javascript
source: "/home/suggest?highlights=false&fuzzy=true&",
```

Presione F5 para iniciar la aplicación.

Pruebe escribir algo como "ejecativo" y observe cómo aparecen resultados para "ejecutivo", incluso si no son coincidencia exacta de las letras que escribió.

### <a name="jquery-autocomplete--backed-by-azure-search-autocomplete"></a>Autocompletar de jQuery respaldada por Azure Search Autocompletar

Hasta ahora, la búsqueda de código de la experiencia del usuario se ha centrado en las sugerencias. El bloque de código siguiente muestra la función de Autocompletar de la interfaz de usuario de jQuery (línea de index.cshtml en 91,), pasando una solicitud de Autocompletar de Azure Search:

```javascript
$(function () {
    // using modified jQuery Autocomplete plugin v1.2.6 https://xdsoft.net/jqplugins/autocomplete/
    // $.autocomplete -> $.autocompleteInline
    $("#example2").autocompleteInline({
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
    $("#example2").keydown(function (evt) {
        if (evt.keyCode === 9 /* TAB */ && currentSuggestion2) {
            $("#example2").val(currentSuggestion2);
            return false;
        } else if (evt.keyCode === 27 /* ESC */) {
            currentSuggestion2 = "";
            $("#example2").val("");
        }
    });
});
```

## <a name="c-example"></a>Ejemplo de C#

Ahora que hemos revisado el código JavaScript de la página web, echemos un vistazo a la C# código del controlador de servidor que realmente recupera las coincidencias sugeridas mediante el SDK de .NET de Azure Search.

Abra el **HomeController.cs** archivo bajo el directorio de controladores. 

Lo primero que notará es un método en la parte superior de la clase denominada `InitSearch`. Esto crea un cliente del índice HTTP autenticado en el servicio Azure Search. Para obtener más información, consulte [cómo usar Azure Search desde una aplicación .NET](https://docs.microsoft.com/azure/search/search-howto-dotnet-sdk).

En la línea 41, tenga en cuenta la función sugerir. Se basa en el [DocumentsOperationsExtensions.Suggest método](/dotnet/api/microsoft.azure.search.documentsoperationsextensions.suggest?view=azure-dotnet).

```csharp
public ActionResult Suggest(bool highlights, bool fuzzy, string term)
{
    InitSearch();

    // Call suggest API and return results
    SuggestParameters sp = new SuggestParameters()
    {
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

La función Sugerir toma dos parámetros que determinar si se devuelven resaltados de aciertos o si se usa la coincidencia aproximada además de la entrada del término de búsqueda. El método crea un [SuggestParameters objeto](https://docs.microsoft.com/dotnet/api/microsoft.azure.search.models.suggestparameters?view=azure-dotnet), que, a continuación, se pasa a la API de sugerir. Luego el resultado se convierte en JSON para poder mostrarlo en el cliente.

En la línea 69, tenga en cuenta la función de Autocompletar. Se basa en el [DocumentsOperationsExtensions.Autocomplete método](https://docs.microsoft.com/dotnet/api/microsoft.azure.search.documentsoperationsextensions.autocomplete?view=azure-dotnet).

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

    // Conver the Suggest results to a list that can be displayed in the client.
    List<string> autocomplete = autocompleteResult.Results.Select(x => x.Text).ToList();
    return new JsonResult
    {
        JsonRequestBehavior = JsonRequestBehavior.AllowGet,
        Data = autocomplete
    };
}
```

La función de Autocompletar toma la entrada del término de búsqueda. El método crea un [AutoCompleteParameters objeto](https://docs.microsoft.com/rest/api/searchservice/autocomplete). Luego el resultado se convierte en JSON para poder mostrarlo en el cliente.

(Opcional) Agregue un punto de interrupción al inicio de la función Sugerir y recorra el código. Tenga en cuenta la respuesta devuelta por el SDK y cómo se convierte en el resultado devuelto del método.

Los otros ejemplos en la página siguen el mismo patrón para agregar resaltado de referencias y las facetas para admitir el almacenamiento en caché del lado cliente de los resultados de Autocompletar. Revise cada uno de estos elementos para saber cómo funcionan y cómo puede aprovecharlos en la experiencia de búsqueda.

## <a name="javascript-example"></a>Ejemplo de JavaScript

Una implementación de Javascript de Autocompletar y sugerencias, llama a la API de REST, mediante un identificador URI como el origen para especificar el índice y la operación. 

Para revisar la implementación de JavaScript, abra **IndexJavaScript.cshtml**. Tenga en cuenta que también se usa la función de Autocompletar de la interfaz de usuario de jQuery para el cuadro Buscar, recopilar entradas de término de búsqueda y realización de llamadas asincrónicas a Azure Search para recuperar sugiere coincidencias o completado términos. 

Veamos el código de JavaScript del primer ejemplo:

```javascript
$(function () {
    $("#example1a").autocomplete({
        source: function (request, response) {
        $.ajax({
            type: "POST",
            url: suggestUri,
            dataType: "json",
            headers: {
                "api-key": searchServiceApiKey,
                "Content-Type": "application/json"
            },
            data: JSON.stringify({
                top: 5,
                fuzzy: false,
                suggesterName: "sg",
                search: request.term
            }),
                success: function (data) {
                    if (data.value && data.value.length > 0) {
                        response(data.value.map(x => x["@@search.text"]));
                    }
                }
            });
        },
        minLength: 3,
        position: {
            my: "left top",
            at: "left-23 bottom+10"
        }
    });
});
```

Si lo compara con el ejemplo anterior donde se llama al controlador de inicio, verá varias similitudes.  La configuración de la función Autocompletar para `minLength` y `position` es exactamente igual. 

Aquí el cambio importante es el origen. En lugar de llamar al método sugerir en el controlador principal, una solicitud REST se crea en una función de JavaScript y ejecuta mediante Ajax. Luego la respuesta se procesa en estado "correcto" y se usa como el origen.

Llamadas de REST utilizan identificadores URI para especificar si una [Autocompletar](https://docs.microsoft.com/rest/api/searchservice/autocomplete) o [sugerencias](https://docs.microsoft.com/rest/api/searchservice/suggestions) se realiza la llamada de API. Los siguientes URI son en las líneas 9 y 10, respectivamente.

```javascript
var suggestUri = "https://" + searchServiceName + ".search.windows.net/indexes/" + indexName + "/docs/suggest?api-version=" + apiVersion;
var autocompleteUri = "https://" + searchServiceName + ".search.windows.net/indexes/" + indexName + "/docs/autocomplete?api-version=" + apiVersion;
```

En la línea 148, encontrará un script que llame a la `autocompleteUri`. La primera llamada a `suggestUri` está en línea 39.

> [!Note]
> Realizar llamadas REST al servicio en JavaScript se ofrece aquí como una demostración práctica de la API de REST, pero no debe interpretarse como un procedimiento recomendado o esta recomendación. La inclusión de una clave de API y el punto de conexión en una secuencia de comandos abre el servicio hasta la denegación de servicio a cualquier persona que puede leer esos valores fuera de la secuencia de comandos. Mientras su seguro usar JavaScript con fines de aprendizaje, quizás en los índices que se hospeda en el servicio gratuito, se recomienda usar Java o C# para las operaciones de indexación y consulta en código de producción. 

<a name="configure-app"></a>

## <a name="configure-nycjobs-to-run-on-your-service"></a>Configurar NYCJobs para ejecutarse en el servicio

Hasta ahora, ha estado utilizando el índice de la demostración NYCJobs hospedado. Si desea que la visibilidad completa de todo el código, incluido el índice, siga estas instrucciones para crear y cargar el índice en su propio servicio de búsqueda.

1. [Cree un servicio Azure Search](search-create-service-portal.md) o [busque un servicio existente](https://ms.portal.azure.com/#blade/HubsExtension/BrowseResourceBlade/resourceType/Microsoft.Search%2FsearchServices) en su suscripción actual. Puede usar un servicio gratuito para este ejemplo. 

   > [!Note]
   > Si va a usar el servicio gratuito Azure Search, está limitado a tres índices. El cargador de datos de NYCJobs crea dos índices. Asegúrese de que haya espacio en el servicio para aceptar los nuevos índices.

1. Descargar [NYCJobs](https://github.com/Azure-Samples/search-dotnet-asp-net-mvc-jobs) código de ejemplo.

1. En la carpeta DataLoader del código de ejemplo NYCJobs, abra **DataLoader.sln** en Visual Studio.

1. Agregue la información de conexión para el servicio Azure Search. Abra el archivo App.config dentro del proyecto DataLoader y cambie los valores de appSettings TargetSearchServiceName y TargetSearchServiceApiKey para reflejar el servicio Azure Search y la clave de API de Azure Search Service. Puede encontrarlos en Azure Portal.

1. Presione F5 para iniciar la aplicación, crear dos índices e importar los datos de ejemplo NYCJob.

1. Abra **AutocompleteTutorial.sln** y edite el archivo Web.config en el **AutocompleteTutorial** proyecto. Cambie los valores SearchServiceName y SearchServiceApiKey a valores que son válidos para el servicio de búsqueda.

1. Presione F5 para ejecutar la aplicación. Se abre la aplicación web de ejemplo en el explorador predeterminado. La experiencia es idéntica a la versión de espacio aislado, el índice y los datos se hospedan en su servicio.

## <a name="next-steps"></a>Pasos siguientes

En este ejemplo muestra los pasos básicos para la creación de un cuadro de búsqueda que es compatible con Autocompletar y sugerencias. Se ha explicado cómo puede compilar una aplicación ASP.NET MVC y usar el SDK de .NET de Azure Search o la API de REST para recuperar sugerencias.

Como paso siguiente, intentar integrar las sugerencias y Autocompletar en su experiencia de búsqueda. Deben ayudar los siguientes artículos de referencia.

> [!div class="nextstepaction"]
> [Autocomplete REST API (API REST de la función Autocompletar)](https://docs.microsoft.com/rest/api/searchservice/autocomplete)
> [Suggestions REST API (API REST de Sugerencias)](https://docs.microsoft.com/rest/api/searchservice/suggestions)
> [Facets index attribute on a Create Index REST API (Atributo de índice de facetas en una API REST de creación de índice)](https://docs.microsoft.com/rest/api/searchservice/create-index)

