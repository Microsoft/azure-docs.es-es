---
title: Tutorial de C# sobre la paginación de los resultados de la búsqueda
titleSuffix: Azure Cognitive Search
description: Agregue botones de navegación y paginación a los resultados de la búsqueda, y realice la compilación sobre un proyecto de hoteles existente para agregar los botones primero, siguiente, anterior, último y numerados. Un segundo sistema de paginación usa el desplazamiento infinito, desencadenado al mover una barra de desplazamiento vertical en el límite inferior.
manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.service: cognitive-search
ms.topic: tutorial
ms.date: 06/20/2020
ms.custom: devx-track-javascript, devx-track-csharp
ms.openlocfilehash: 3223bf3e2b413940fd99404453b7f958655e6fff
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "88998482"
---
# <a name="tutorial-add-paging-to-search-results-using-the-net-sdk"></a>Tutorial: Incorporación de la paginación a los resultados de búsqueda mediante el SDK de .NET

Aprenda a implementar dos sistemas de paginación distintos, el primero basado en los números de página y el segundo en el desplazamiento infinito. Ambos sistemas de paginación se utilizan ampliamente y seleccionar el correcto depende de la experiencia de usuario que desee con los resultados. Este tutorial se basa en los sistemas de paginación del proyecto que se creó en el [Tutorial de C#: Creación de la primera aplicación: Azure Cognitive Search](tutorial-csharp-create-first-app.md).

En este tutorial, aprenderá a:
> [!div class="checklist"]
> * Ampliar la aplicación con la paginación numerada
> * Ampliar la aplicación con el desplazamiento infinito

## <a name="prerequisites"></a>Prerrequisitos

Para completar este tutorial, necesita:

Tener listo y en ejecución el proyecto [Tutorial de C#: Creación de la primera aplicación: Azure Cognitive Search](tutorial-csharp-create-first-app.md). Este proyecto puede ser su propia versión o puede instalarlo desde GitHub: [Create first app](https://github.com/Azure-Samples/azure-search-dotnet-samples) (Crear la primera aplicación).

## <a name="extend-your-app-with-numbered-paging"></a>Ampliar la aplicación con la paginación numerada

La paginación numerada es el sistema de paginación favoritos de los principales motores de búsqueda de Internet y de la mayoría otros sitios web de búsqueda. La paginación numerado normalmente incluye una opción "siguiente" y "anterior", además de un intervalo de números de página real. También pueden estar disponibles una opción de "última página" y de "primera página". Estas opciones le dan al usuario el control sobre el desplazamiento en los resultados de la página.

Agregaremos un sistema que incluya las opciones de primera página, anterior, siguiente y última, junto con números de página que no se inicien en 1, sino usar la página actual en la que esté el usuario (por lo tanto, por ejemplo, si el usuario está viendo la página 10, podrían mostrarse los números de página 8, 9, 10, 11 y 12).

El sistema será lo suficientemente flexible como para permitir que la cantidad de números de página visibles se establezca en una variable global.

El sistema considerará los botones de número de página más a la izquierda y más a la derecha como especiales, lo que significa que desencadenarán el cambio del intervalo de números de página que se muestra. Por ejemplo, si se muestran los números de página 8, 9, 10, 11 y 12, y el usuario hace clic en 8, el intervalo de números de página muestra los cambios a 6, 7, 8, 9 y 10. Y hay un cambio similar a la derecha si se selecciona 12.

### <a name="add-paging-fields-to-the-model"></a>Incorporación de campos de paginación al modelo

Tiene abierta la solución de la página de búsqueda básica.

1. Abra el archivo de modelo SearchData.cs.

2. En primer lugar, agregue algunas variables globales. En MVC, las variables globales se declaran en su propia clase estática. **ResultsPerPage** establece el número de resultados por página. **MaxPageRange** determina la cantidad de números de página visibles en la vista. **PageRangeDelta** determina cuántas páginas a la izquierda o la derecha se debe desplazar el intervalo de páginas, cuando se selecciona el número de página más a la izquierda o a la derecha. Normalmente, este último número es en torno a la mitad de **MaxPageRange**. Agregue el código siguiente en el espacio de nombres.

    ```cs
    public static class GlobalVariables
    {
        public static int ResultsPerPage
        {
            get
            {
                return 3;
            }
        }
        public static int MaxPageRange
        {
            get
            {
                return 5;
            }
        }

        public static int PageRangeDelta
        {
            get
            {
                return 2;
            }
        }
    }
    ```

    >[!Tip]
    >Si ejecuta este proyecto en un dispositivo con una pantalla más pequeña, como un equipo portátil, considere la posibilidad de cambiar **ResultsPerPage** a 2.

3. Agregue las propiedades de paginación a la clase **SearchData**, por ejemplo, después de la propiedad **searchText**.

    ```cs
        // The current page being displayed.
        public int currentPage { get; set; }

        // The total number of pages of results.
        public int pageCount { get; set; }

        // The left-most page number to display.
        public int leftMostPage { get; set; }

        // The number of page numbers to display - which can be less than MaxPageRange towards the end of the results.
        public int pageRange { get; set; }

        // Used when page numbers, or next or prev buttons, have been selected.
        public string paging { get; set; }
    ```

### <a name="add-a-table-of-paging-options-to-the-view"></a>Incorporación de una tabla de opciones de paginación a la vista

1. Abra el archivo index.cshtml y agregue el código siguiente justo antes de la etiqueta &lt;/body&gt; de cierre. Este nuevo código presenta una tabla de opciones de paginación: primero, anterior, 1, 2, 3, 4, 5, siguiente, última.

    ```cs
    @if (Model != null && Model.pageCount > 1)
    {
    // If there is more than one page of results, show the paging buttons.
    <table>
        <tr>
            <td>
                @if (Model.currentPage > 0)
                {
                    <p class="pageButton">
                        @Html.ActionLink("|<", "Page", "Home", new { paging = "0" }, null)
                    </p>
                }
                else
                {
                    <p class="pageButtonDisabled">|&lt;</p>
                }
            </td>

            <td>
                @if (Model.currentPage > 0)
                {
                    <p class="pageButton">
                        @Html.ActionLink("<", "Page", "Home", new { paging = "prev" }, null)
                    </p>
                }
                else
                {
                    <p class="pageButtonDisabled">&lt;</p>
                }
            </td>

            @for (var pn = Model.leftMostPage; pn < Model.leftMostPage + Model.pageRange; pn++)
            {
                <td>
                    @if (Model.currentPage == pn)
                    {
                        // Convert displayed page numbers to 1-based and not 0-based.
                        <p class="pageSelected">@(pn + 1)</p>
                    }
                    else
                    {
                        <p class="pageButton">
                            @Html.ActionLink((pn + 1).ToString(), "Page", "Home", new { paging = @pn }, null)
                        </p>
                    }
                </td>
            }

            <td>
                @if (Model.currentPage < Model.pageCount - 1)
                {
                    <p class="pageButton">
                        @Html.ActionLink(">", "Page", "Home", new { paging = "next" }, null)
                    </p>
                }
                else
                {
                    <p class="pageButtonDisabled">&gt;</p>
                }
            </td>

            <td>
                @if (Model.currentPage < Model.pageCount - 1)
                {
                    <p class="pageButton">
                        @Html.ActionLink(">|", "Page", "Home", new { paging = Model.pageCount - 1 }, null)
                    </p>
                }
                else
                {
                    <p class="pageButtonDisabled">&gt;|</p>
                }
            </td>
        </tr>
    </table>
    }
    ```

    Se usa una tabla HTML para alinear los elementos ordenadamente. Aunque todas las acciones proceden de las instrucciones @Html.ActionLink, cada una invoca al controlador con un modelo **nuevo** creado con entradas diferentes para la propiedad **paging** agregada anteriormente.

    Las opciones de primera y última página no envían cadenas como "first" y "last", sin que, en su lugar, envían los números de página correctos.

2. Agregue algunas clases de paginación a la lista de estilos HTML en el archivo hotels.css. La clase **pageSelected** está ahí para identificar la página que el usuario está viendo actualmente (y se vuelve el número en negrita) en la lista de números de página.

    ```html
        .pageButton {
            border: none;
            color: darkblue;
            font-weight: normal;
            width: 50px;
        }

        .pageSelected {
            border: none;
            color: black;
            font-weight: bold;
            width: 50px;
        }

        .pageButtonDisabled {
            border: none;
            color: lightgray;
            font-weight: bold;
            width: 50px;
        }
    ```

### <a name="add-a-page-action-to-the-controller"></a>Incorporación de una acción Page al controlador

1. Abra el archivo HomeController.cs y agregue la acción **Page**. Esta acción responde a cualquiera de las opciones de página seleccionadas.

    ```cs
        public async Task<ActionResult> Page(SearchData model)
        {
            try
            {
                int page;

                switch (model.paging)
                {
                    case "prev":
                        page = (int)TempData["page"] - 1;
                        break;

                    case "next":
                        page = (int)TempData["page"] + 1;
                        break;

                    default:
                        page = int.Parse(model.paging);
                        break;
                }

                // Recover the leftMostPage.
                int leftMostPage = (int)TempData["leftMostPage"];

                // Recover the search text and search for the data for the new page.
                model.searchText = TempData["searchfor"].ToString();

                await RunQueryAsync(model, page, leftMostPage);

                // Ensure Temp data is stored for next call, as TempData only stored for one call.
                TempData["page"] = (object)page;
                TempData["searchfor"] = model.searchText;
                TempData["leftMostPage"] = model.leftMostPage;
            }

            catch
            {
                return View("Error", new ErrorViewModel { RequestId = "2" });
            }
            return View("Index", model);
        }
    ```

    El método **RunQueryAsync** mostrará ahora un error de sintaxis, debido al tercer parámetro, sobre el que volveremos en un momento.

    > [!Note]
    > Las llamadas a **TempData** almacenan un valor (un **objeto**) en el almacenamiento temporal, aunque este almacenamiento persiste _solo_ una llamada. Si se almacena algo en los datos temporales, estará disponible para la siguiente llamada a una acción de controlador, pero desaparecerá definitivamente tras la llamada después. Debido a esta corta duración, almacenamos el texto de búsqueda y las propiedades de paginación de nuevo en cada almacenamiento temporal y en cada llamada a **Page**.

2. La acción **Index(model)** tiene que actualizarse para almacenar las variables temporales y agregar el parámetro de página más a la izquierda para la llamada a **RunQueryAsync**.

    ```cs
        public async Task<ActionResult> Index(SearchData model)
        {
            try
            {
                // Ensure the search string is valid.
                if (model.searchText == null)
                {
                    model.searchText = "";
                }

                // Make the search call for the first page.
                await RunQueryAsync(model, 0, 0);

                // Ensure temporary data is stored for the next call.
                TempData["page"] = 0;
                TempData["leftMostPage"] = 0;
                TempData["searchfor"] = model.searchText;
            }

            catch
            {
                return View("Error", new ErrorViewModel { RequestId = "1" });
            }
            return View(model);
        }
    ```

3. El método **RunQueryAsync** necesita ser actualizado a menudo. Usamos los campos **Skip**, **Top** e **IncludeTotalResultCount** de la clase **SearchParameters** para solicitar solo una página de resultados, a partir de la opción **Skip**. También es necesario calcular las variables de paginación para nuestra vista. Reemplace el método entero por el código siguiente.

    ```cs
        private async Task<ActionResult> RunQueryAsync(SearchData model, int page, int leftMostPage)
        {
            InitSearch();

            var parameters = new SearchParameters
            {
                   // Enter Hotel property names into this list so only these values will be returned.
                   // If Select is empty, all values will be returned, which can be inefficient.
                   Select = new[] { "HotelName", "Description" },
                   SearchMode = SearchMode.All,

                   // Skip past results that have already been returned.
                   Skip = page * GlobalVariables.ResultsPerPage,

                   // Take only the next page worth of results.
                   Top = GlobalVariables.ResultsPerPage,

                   // Include the total number of results.
                   IncludeTotalResultCount = true,
               };

            // For efficiency, the search call should be asynchronous, so use SearchAsync rather than Search.
            model.resultList = await _indexClient.Documents.SearchAsync<Hotel>(model.searchText, parameters);

            // This variable communicates the total number of pages to the view.
            model.pageCount = ( (int)model.resultList.Count + GlobalVariables.ResultsPerPage - 1) / GlobalVariables.ResultsPerPage;

            // This variable communicates the page number being displayed to the view.
            model.currentPage = page;

            // Calculate the range of page numbers to display.
            if (page == 0)
            {
                leftMostPage = 0;
            }
            else
               if (page <= leftMostPage)
            {
                // Trigger a switch to a lower page range.
                leftMostPage = Math.Max(page - GlobalVariables.PageRangeDelta, 0);
            }
            else
            if (page >= leftMostPage + GlobalVariables.MaxPageRange - 1)
            {
                // Trigger a switch to a higher page range.
                leftMostPage = Math.Min(page - GlobalVariables.PageRangeDelta, model.pageCount - GlobalVariables.MaxPageRange);
            }
            model.leftMostPage = leftMostPage;

            // Calculate the number of page numbers to display.
            model.pageRange = Math.Min(model.pageCount - leftMostPage, GlobalVariables.MaxPageRange);

            return View("Index", model);
        }
    ```

4. Por último, tenemos que realizar un pequeño cambio en la vista. La variable **resultsList.Results.Count** ahora contendrá el número de resultados devueltos en una sola página (tres, en nuestro ejemplo), no el número total. Dado que establecemos la **IncludeTotalResultCount** en true, la variable **resultsList.Count** ahora contiene el número total de resultados. Por tanto, busque dónde se muestra el número de resultados en la vista y cámbielo al código siguiente.

    ```cs
            // Show the result count.
            <p class="sampleText">
                @Html.DisplayFor(m => m.resultList.Count) Results
            </p>
    ```

    > [!Note]
    > Hay un aumento del valor, aunque normalmente no superior a uno, al establecer **IncludeTotalResultCount** como true, ya que es necesario que Azure Cognitive Search calcule este total. Con conjuntos de datos complejos, se advierte de que el valor devuelto es una _aproximación_. En los datos del hotel, será preciso.

### <a name="compile-and-run-the-app"></a>Compilación y ejecución de la aplicación

Ahora, seleccione **Iniciar sin depurar** (o presione la tecla F5).

1. Busque algún texto que ofrezca una gran cantidad de resultados (por ejemplo, "wifi"). ¿Puede recorrer las páginas correctamente en los resultados?

    ![Paginación numerada en los resultados del "grupo"](./media/tutorial-csharp-create-first-app/azure-search-numbered-paging.png)

2. Pruebe a hacer clic en los números de página de la izquierda, de la derecha y última. ¿Los números de página se ajustan adecuadamente al centro de la página en la que se encuentra?

3. ¿Son útiles las opciones "primera" y "última"? No todas las búsquedas en web populares usan estas opciones.

4. Vaya a la última página de resultados. La última página es la única que puede contener menos de **ResultsPerPage** resultados.

    ![Examen de la última página de "wifi"](./media/tutorial-csharp-create-first-app/azure-search-pool-last-page.png)

5. Escriba "town" y haga clic en Buscar. Si hay menos de una página de resultados, no se muestran las opciones de paginación.

    ![Búsqueda de "town"](./media/tutorial-csharp-create-first-app/azure-search-town.png)

Ahora guarde este proyecto y vamos a probar una alternativa a esta forma de paginación.

## <a name="extend-your-app-with-infinite-scrolling"></a>Ampliar la aplicación con el desplazamiento infinito

El desplazamiento infinito se desencadena cuando un usuario se desplaza hasta el último de los resultados que se muestra con una barra de desplazamiento vertical. En este caso, se realiza una llamada al servidor para obtener la siguiente página de resultados. Si no hay ningún resultado más, no se devuelve nada y la barra de desplazamiento vertical no cambia. Si hay más resultados, se anexan a la página actual y la barra de desplazamiento cambia para mostrar que hay más resultados disponibles.

Lo importante aquí es que la página que se muestra no se reemplaza, sino que se anexa con los nuevos resultados. Un usuario siempre puede desplazarse hasta los primeros resultados de la búsqueda.

Para implementar el desplazamiento infinito, comencemos con el proyecto antes de que se agregara cualquiera de los elementos de desplazamiento del número de página. Por lo tanto, si es necesario, haga otra copia de la página de búsqueda básica desde GitHub: [Create first app](https://github.com/Azure-Samples/azure-search-dotnet-samples) (Crear la primera aplicación).

### <a name="add-paging-fields-to-the-model"></a>Incorporación de campos de paginación al modelo

1. En primer lugar, agregue una propiedad **paging** a la clase **SearchData** (en el archivo de modelo SearchData.cs).

    ```cs
        // Record if the next page is requested.
        public string paging { get; set; }
    ```

    Esta variable es una cadena que contiene "next" (siguiente) si la página siguiente de resultados se debe enviar o es null para la primera página de una búsqueda.

2. En el mismo archivo y dentro del espacio de nombres, agregue una clase de variable global con una propiedad. En MVC, las variables globales se declaran en su propia clase estática. **ResultsPerPage** establece el número de resultados por página. 

    ```cs
    public static class GlobalVariables
    {
        public static int ResultsPerPage
        {
            get
            {
                return 3;
            }
        }
    }
    ```

### <a name="add-a-vertical-scroll-bar-to-the-view"></a>Incorporación de una barra de desplazamiento vertical a la vista

1. Busque la sección del archivo index.cshtml que muestra los resultados (comienza con **@if (Model !=null)** ).

2. Reemplace la sección por el código siguiente. La nueva sección **&lt;div&gt;** está alrededor del área que debe ser desplazable y agrega tanto un atributo **overflow-y** como una llamada a una función **onscroll** denominada "scrolled()".

    ```cs
        @if (Model != null)
        {
            // Show the result count.
            <p class="sampleText">
                @Html.DisplayFor(m => m.resultList.Count) Results
            </p>

            <div id="myDiv" style="width: 800px; height: 450px; overflow-y: scroll;" onscroll="scrolled()">

                <!-- Show the hotel data. -->
                @for (var i = 0; i < Model.resultList.Results.Count; i++)
                {
                    // Display the hotel name and description.
                    @Html.TextAreaFor(m => Model.resultList.Results[i].Document.HotelName, new { @class = "box1" })
                    @Html.TextArea($"desc{i}", Model.resultList.Results[i].Document.Description, new { @class = "box2" })
                }
            </div>
        }
    ```

3. Directamente debajo del bucle, después de la etiqueta &lt;/div&gt;, agregue la función **scrolled**.

    ```javascript
        <script>
                function scrolled() {
                    if (myDiv.offsetHeight + myDiv.scrollTop >= myDiv.scrollHeight) {
                        $.getJSON("/Home/Next", function (data) {
                            var div = document.getElementById('myDiv');

                            // Append the returned data to the current list of hotels.
                            for (var i = 0; i < data.length; i += 2) {
                                div.innerHTML += '\n<textarea class="box1">' + data[i] + '</textarea>';
                                div.innerHTML += '\n<textarea class="box2">' + data[i + 1] + '</textarea>';
                            }
                        });
                    }
                }
        </script>
    ```

    La instrucción **if** en el script anterior comprueba si el usuario se ha desplazado a la parte inferior de la barra de desplazamiento vertical. Si se han desplazado, se realiza una llamada al controlador **Home** para una acción denominada **Next**. El controlador no necesita ninguna otra información, devolverá la siguiente página de datos. A continuación, se da formato a estos datos con estilos HTML idénticos al de la página original. Si no se devuelve ningún resultado, no se anexa nada y todo permanece como esté.

### <a name="handle-the-next-action"></a>Control de acción Next

Solo hay tres acciones que deban enviarse al controlador: la primera ejecución de la aplicación, que llama a **Index()** , la primera búsqueda del usuario, que llama a **Index(model)** y, a continuación, las siguientes llamadas para obtener más resultados mediante **Next(model)** .

1. Abra el archivo del controlador de home y elimine el método **RunQueryAsync** del tutorial original.

2. Reemplace la acción **Index(model)** con el código siguiente. Ahora controla el campo **paging** cuando es null o se establece en "next" y controla la llamada a Azure Cognitive Search.

    ```cs
        public async Task<ActionResult> Index(SearchData model)
        {
            try
            {
                InitSearch();

                int page;

                if (model.paging != null && model.paging == "next")
                {
                    // Increment the page.
                    page = (int)TempData["page"] + 1;

                    // Recover the search text.
                    model.searchText = TempData["searchfor"].ToString();
                }
                else
                {
                    // First call. Check for valid text input.
                    if (model.searchText == null)
                    {
                        model.searchText = "";
                    }
                    page = 0;
                }

                // Setup the search parameters.
                var parameters = new SearchParameters
                {
                    // Enter Hotel property names into this list so only these values will be returned.
                    // If Select is empty, all values will be returned, which can be inefficient.
                    Select = new[] { "HotelName", "Description" },
                    SearchMode = SearchMode.All,

                    // Skip past results that have already been returned.
                    Skip = page * GlobalVariables.ResultsPerPage,

                    // Take only the next page worth of results.
                    Top = GlobalVariables.ResultsPerPage,

                    // Include the total number of results.
                    IncludeTotalResultCount = true,
                };

                // For efficiency, the search call should be asynchronous, so use SearchAsync rather than Search.
                model.resultList = await _indexClient.Documents.SearchAsync<Hotel>(model.searchText, parameters);

                // Ensure TempData is stored for the next call.
                TempData["page"] = page;
                TempData["searchfor"] = model.searchText;
            }
            catch
            {
                return View("Error", new ErrorViewModel { RequestId = "1" });
            }
            return View("Index", model);
        }
    ```

    Al igual que el método de paginación numerada, usamos las opciones de búsqueda **Skip** y **Top** para solicitar que se devuelvan solo los datos que necesitamos.

3. Agregue la acción **Next** al controlador de inicio. Tenga en cuenta cómo devuelve una lista, cada hotel agrega dos elementos a la lista: un nombre y una descripción de hotel. Este formato se establece para que coincida con el uso de la función **scrolled** de los datos devueltos en la vista.

    ```cs
        public async Task<ActionResult> Next(SearchData model)
        {
            // Set the next page setting, and call the Index(model) action.
            model.paging = "next";
            await Index(model);

            // Create an empty list.
            var nextHotels = new List<string>();

            // Add a hotel name, then description, to the list.
            for (int n = 0; n < model.resultList.Results.Count; n++)
            {
                nextHotels.Add(model.resultList.Results[n].Document.HotelName);
                nextHotels.Add(model.resultList.Results[n].Document.Description);
            }

            // Rather than return a view, return the list of data.
            return new JsonResult(nextHotels);
        }
    ```

4. Si recibe un error de sintaxis en la cadena **List&lt;&gt;** , agregue la siguiente directiva **using** al encabezado del archivo del controlador.

    ```cs
    using System.Collections.Generic;
    ```

### <a name="compile-and-run-your-project"></a>Compilación y ejecución del proyecto

Ahora, seleccione **Iniciar sin depurar** (o presione la tecla F5).

1. Escriba un término que proporcione una gran cantidad de resultados (por ejemplo, "pool") y, a continuación, pruebe la barra de desplazamiento vertical. ¿Desencadena una nueva página de resultados?

    ![El desplazamiento infinito por los resultados de "pool"](./media/tutorial-csharp-create-first-app/azure-search-infinite-scroll.png)

    > [!Tip]
    > Para asegurarse de que aparece una barra de desplazamiento en la primera página, la primera página de resultados debe superar ligeramente el alto del área en que se muestran. En nuestro ejemplo **.box1** tiene un alto de 30 píxeles, **.box2** tiene un alto de 100 píxeles _y_ un margen inferior de 24 píxeles. Por lo tanto, cada entrada usa 154 píxeles. Tres entradas pueden ocupar hasta 3 x 154 = 462 píxeles. Para asegurarse de que aparece una barra de desplazamiento vertical, se debe establecer un alto para el área de presentación menor que 462 píxeles, incluso 461 funciona. Este problema se produce únicamente en la primera página, después de que se tenga la seguridad de que la barra de desplazamiento aparecerá. La línea para actualizar es: **&lt;div id="myDiv" style="width: 800px; height: 450px; overflow-y: scroll;" onscroll="scrolled()"&gt;** .

2. Desplácese hacia abajo hasta la parte inferior de los resultados. Tenga en cuenta cómo toda la información ya está en la página de una vista. Puede desplazarse remontándose a la parte superior sin desencadenar ninguna llamada al servidor.

Los sistemas más sofisticados de desplazamiento infinito podrían usar la rueda del mouse u otro mecanismo similar para desencadenar la carga de una nueva página de resultados. No adoptaremos el desplazamiento infinito más en estos tutoriales, pero son útiles, ya que evitan clics adicionales del mouse y puede resultar interesante investigar otras opciones.

## <a name="takeaways"></a>Puntos clave

Tenga en cuenta las siguientes conclusiones de este proyecto:

* La paginación numerada es adecuada para las búsquedas en las que el orden de los resultados sea bastante arbitrario, lo que significa que puede haber algo interés para los usuarios en las páginas posteriores.
* El desplazamiento infinito es adecuado cuando el orden de los resultados es especialmente importante. Por ejemplo, si se ordenan según la distancia desde el centro de una ciudad de destino.
* La paginación numerada permite una mejor navegación. Por ejemplo, un usuario puede recordar que un resultado interesante estaba en la página 6, mientras que tal referencia fácil no existe en el desplazamiento infinito.
* El desplazamiento infinito tiene una apariencia sencilla y el desplazamiento se produce arriba y abajo sin que haya que hacer clic en números de página irrelevantes.
* Una característica clave del desplazamiento infinito es que los resultados se anexan a una página existente, no reemplazan esa página, lo que resulta eficaz.
* El almacenamiento temporal se mantiene solo una llamada y debe reiniciarse para sobrevivir a las siguientes.


## <a name="next-steps"></a>Pasos siguientes

La paginación es fundamental para las búsquedas de internet. Si la paginación se ha resuelto adecuadamente, el paso siguiente es mejorar aún más la experiencia del usuario incorporando búsquedas de escritura anticipada.

> [!div class="nextstepaction"]
> [Tutorial de C#: Incorporación de la función Autocompletar y las sugerencias en Azure Cognitive Search](tutorial-csharp-type-ahead-and-suggestions.md)
