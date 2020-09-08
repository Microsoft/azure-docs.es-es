---
title: Tutorial de C# sobre el uso de facetas para facilitar la navegación
titleSuffix: Azure Cognitive Search
description: Continúe desde "Resultados de paginación" para agregar la navegación por facetas. Aprenda cómo usar facetas para restringir una búsqueda fácilmente.
manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.service: cognitive-search
ms.topic: tutorial
ms.date: 06/20/2020
ms.custom: devx-track-csharp
ms.openlocfilehash: 0b51960418b311ed6170d6d060f3674a9a825808
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "88998516"
---
# <a name="tutorial-add-faceted-navigation-using-the-net-sdk"></a>Tutorial: Incorporación de la navegación por facetas mediante el SDK de .NET

Las facetas se usan para facilitar la navegación al proporcionar al usuario un conjunto de vínculos se usan para centrar su búsqueda. Las facetas son atributos de los datos (por ejemplo, la categoría, o una característica específica, de un hotel en los datos de ejemplo).

Este tutorial se basa en el proyecto de paginación que se creó en el [Tutorial de C#: Paginación de los resultados de búsqueda: Azure Cognitive Search](tutorial-csharp-paging.md).

En este tutorial, aprenderá a:
> [!div class="checklist"]
> * Establecer las propiedades del modelo en _IsFacetable_
> * Agregar navegación por facetas a la aplicación

## <a name="prerequisites"></a>Prerrequisitos

Para completar este tutorial, necesita:

Tener listo y en ejecución el proyecto [Tutorial de C#: El proyecto Paginación de los resultados de búsqueda: Azure Cognitive Search](tutorial-csharp-paging.md) en funcionamiento. Este proyecto puede ser su propia versión o puede instalarlo desde GitHub: [Create first app](https://github.com/Azure-Samples/azure-search-dotnet-samples) (Crear la primera aplicación).

## <a name="set-model-properties-as-isfacetable"></a>Establecer las propiedades del modelo en IsFacetable

Para que una propiedad de modelo se encuentre en una búsqueda por facetas, deben estar etiquetada con **IsFacetable**.

1. Consulte la clase **Hotel**. Los objetos **Category** y **Tags**, por ejemplo, se etiquetan como **IsFacetable**, pero **HotelName** y **Description** no. 

    ```cs
    public partial class Hotel
    {
        [System.ComponentModel.DataAnnotations.Key]
        [IsFilterable]
        public string HotelId { get; set; }

        [IsSearchable, IsSortable]
        public string HotelName { get; set; }

        [IsSearchable]
        [Analyzer(AnalyzerName.AsString.EnLucene)]
        public string Description { get; set; }

        [IsSearchable]
        [Analyzer(AnalyzerName.AsString.FrLucene)]
        [JsonProperty("Description_fr")]
        public string DescriptionFr { get; set; }

        [IsSearchable, IsFilterable, IsSortable, IsFacetable]
        public string Category { get; set; }

        [IsSearchable, IsFilterable, IsFacetable]
        public string[] Tags { get; set; }

        [IsFilterable, IsSortable, IsFacetable]
        public bool? ParkingIncluded { get; set; }

        [IsFilterable, IsSortable, IsFacetable]
        public DateTimeOffset? LastRenovationDate { get; set; }

        [IsFilterable, IsSortable, IsFacetable]
        public double? Rating { get; set; }

        public Address Address { get; set; }

        [IsFilterable, IsSortable]
        public GeographyPoint Location { get; set; }

        public Room[] Rooms { get; set; }

    }
    ```

2. Como parte de este tutorial, no se cambiará ninguna etiquete, por lo que debe cerrar el archivo hotel.cs sin cambiar nada.

    > [!Note]
    > Una búsqueda por facetas producirá un error si un campo solicitado en la búsqueda no se etiqueta correctamente.


## <a name="add-facet-navigation-to-your-app"></a>Agregar navegación por facetas a la aplicación

En este ejemplo, vamos a permitir al usuario seleccionar una categoría de un hotel o un servicio de las listas de vínculos que se muestran a la izquierda de los resultados. Para comenzar, el usuario especifica texto de búsqueda y luego limita los resultados de la búsqueda al seleccionar una categoría. Luego, puede limitar aún más resultados al seleccionar un servicio, o bien puede seleccionar el servicio primero (el orden no es importante).

El controlador tiene que para pasar las listas de facetas a la vista. Es necesario mantener las selecciones del usuario a medida que avanza la búsqueda, y nuevamente, usamos el almacenamiento temporal como mecanismo para conservar los datos.

![Usar la navegación por facetas para limitar una búsqueda de "pool"](./media/tutorial-csharp-create-first-app/azure-search-facet-nav.png)

### <a name="add-filter-strings-to-the-searchdata-model"></a>Agregar cadenas de filtro al modelo SearchData

1. Abra el archivo SearchData.cs y agregue las propiedades de cadena a la clase **SearchData** para almacenar las cadenas de filtro de faceta.

    ```cs
        public string categoryFilter { get; set; }
        public string amenityFilter { get; set; }
    ```

### <a name="add-the-facet-action-method"></a>Agregar el método de acción de faceta

El controlador principal necesita una nueva acción, **Facet**, y actualizaciones a sus acciones existentes **Index** y **Page**, así como actualizaciones al método **RunQueryAsync** .

1. Abra el archivo del controlador principal y agregue la instrucción **using** para habilitar la construcción **List&lt;cadena&gt;** .

    ```cs
    using System.Collections.Generic;
    ```

2. Reemplace el método de acción **Index(SearchData model)** .

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
                await RunQueryAsync(model, 0, 0, "", "");
            }

            catch
            {
                return View("Error", new ErrorViewModel { RequestId = "1" });
            }
            return View(model);
        }
    ```

3. Reemplace el método de acción **Page(SearchData model)** .

    ```cs
        public async Task<ActionResult> Page(SearchData model)
        {
            try
            {
                int page;

                // Calculate the page that should be displayed.
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

                // Recover the filters.
                string catFilter = TempData["categoryFilter"].ToString();
                string ameFilter = TempData["amenityFilter"].ToString();

                // Recover the search text.
                model.searchText = TempData["searchfor"].ToString();

                // Search for the new page.
                await RunQueryAsync(model, page, leftMostPage, catFilter, ameFilter);
            }

            catch
            {
                return View("Error", new ErrorViewModel { RequestId = "2" });
            }
            return View("Index", model);
        }
    ```

4. Agregue un método de acción **Facet(SearchData model)** , que se activará cuando el usuario haga clic en un vínculo de faceta. El modelo incluirá un filtro de búsqueda de categorías o un filtro de búsqueda de servicios. Quizás agréguelo después de la acción **Page**.

    ```cs
        public async Task<ActionResult> Facet(SearchData model)
        {
            try
            {
                // Filters set by the model override those stored in temporary data.
                string catFilter;
                string ameFilter;
                if (model.categoryFilter != null)
                {
                    catFilter = model.categoryFilter;
                } else
                {
                    catFilter = TempData["categoryFilter"].ToString();
                }

                if (model.amenityFilter != null)
                {
                    ameFilter = model.amenityFilter;
                } else
                {
                    ameFilter = TempData["amenityFilter"].ToString();
                }

                // Recover the search text.
                model.searchText = TempData["searchfor"].ToString();

                // Initiate a new search.
                await RunQueryAsync(model, 0, 0, catFilter, ameFilter);
            }

            catch
            {
                return View("Error", new ErrorViewModel { RequestId = "2" });
            }
            return View("Index", model);
        }
    ```

### <a name="set-up-the-search-filter"></a>Configurar el filtro de búsqueda

Cuando un usuario selecciona una faceta determinada, por ejemplo, hace clic en la categoría **Resort and Spa**, en los resultados solo se deben devolver los hoteles que se especifican en esta categoría. Para limitar una búsqueda de este modo, se debe configurar un _filtro_.

1. Reemplace el método **RunQueryAsync** con el código siguiente. Principalmente, se toma una cadena de filtro de categoría y una cadena de filtro de servicios, y establece el parámetro **Filter** del objeto **SearchParameters**.

    ```cs
        private async Task<ActionResult> RunQueryAsync(SearchData model, int page, int leftMostPage, string catFilter, string ameFilter)
        {
            InitSearch();

            string facetFilter = "";

            if (catFilter.Length > 0 && ameFilter.Length > 0)
            {
                // Both facets apply.
                facetFilter = $"{catFilter} and {ameFilter}"; 
            } else
            {
                // One, or zero, facets apply.
                facetFilter = $"{catFilter}{ameFilter}";
            }

            var parameters = new SearchParameters
            {
                Filter = facetFilter,

                // Return information on the text, and number, of facets in the data.
                Facets = new List<string> { "Category,count:20", "Tags,count:20" },

                // Enter Hotel property names into this list, so only these values will be returned.
                Select = new[] { "HotelName", "Description", "Category", "Tags" },

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
            model.pageCount = ((int)model.resultList.Count + GlobalVariables.ResultsPerPage - 1) / GlobalVariables.ResultsPerPage;

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

            // Ensure Temp data is stored for the next call.
            TempData["page"] = page;
            TempData["leftMostPage"] = model.leftMostPage;
            TempData["searchfor"] = model.searchText;
            TempData["categoryFilter"] = catFilter;
            TempData["amenityFilter"] = ameFilter;

            // Return the new view.
            return View("Index", model);
        }
    ```

    Hemos agregado las propiedades **Category** y **Tags** a la lista de elementos del objeto **Select** que se devolverán. Esta adición no es un requisito para que la navegación por facetas funcione, pero usamos esta información para comprobar que se está filtrando correctamente.

### <a name="add-lists-of-facet-links-to-the-view"></a>Agregar listas de vínculos de faceta a la vista

La vista necesitará algunos cambios importantes. 

1. Comience por abrir el archivo hotels.css (en la carpeta wwwroot/css) y luego agregue las siguientes clases.

    ```html
    .facetlist {
        list-style: none;
    }

    .facetchecks {
        width: 250px;
        display: normal;
        color: #666;
        margin: 10px;
        padding: 5px;
    }

    .facetheader {
        font-size: 10pt;
        font-weight: bold;
        color: darkgreen;    
    }
    ```

2. Para la vista, la salida se organiza en una tabla para alinear de manera ordenada las listas de facetas a la izquierda y los resultados a la derecha. Abra el archivo index.cshtml. Reemplace todo el contenido de las etiquetas &lt;body&gt; del código HTML por el código siguiente.

    ```cs
    <body>

    @using (Html.BeginForm("Index", "Home", FormMethod.Post))
    {
        <table>
            <tr>
                <td></td>
                <td>
                    <h1 class="sampleTitle">
                        <img src="~/images/azure-logo.png" width="80" />
                        Hotels Search - Facet Navigation
                    </h1>
                </td>
            </tr>

            <tr>
                <td></td>
                <td>
                    <!-- Display the search text box, with the search icon to the right of it.-->
                    <div class="searchBoxForm">
                        @Html.TextBoxFor(m => m.searchText, new { @class = "searchBox" }) <input value="" class="searchBoxSubmit" type="submit">
                    </div>
                </td>
            </tr>

            <tr>
                <td valign="top">
                    <div id="facetplace" class="facetchecks">

                        @if (Model != null && Model.resultList != null)
                        {
                            List<string> categories = Model.resultList.Facets["Category"].Select(x => x.Value.ToString()).ToList();

                            if (categories.Count > 0)
                            {
                                <h5 class="facetheader">Category:</h5>
                                <ul class="facetlist">
                                    @for (var c = 0; c < categories.Count; c++)
                                    {
                                        var facetLink = $"{categories[c]} ({Model.resultList.Facets["Category"][c].Count})";
                                        <li>
                                            @Html.ActionLink(facetLink, "Facet", "Home", new { categoryFilter = $"Category eq '{categories[c]}'" }, null)
                                        </li>
                                    }
                                </ul>
                            }

                            List<string> tags = Model.resultList.Facets["Tags"].Select(x => x.Value.ToString()).ToList();

                            if (tags.Count > 0)
                            {
                                <h5 class="facetheader">Amenities:</h5>
                                <ul class="facetlist">
                                    @for (var c = 0; c < tags.Count; c++)
                                    {
                                        var facetLink = $"{tags[c]} ({Model.resultList.Facets["Tags"][c].Count})";
                                        <li>
                                            @Html.ActionLink(facetLink, "Facet", "Home", new { amenityFilter = $"Tags/any(t: t eq '{tags[c]}')" }, null)
                                        </li>
                                    }
                                </ul>
                            }
                        }
                    </div>
                </td>
                <td valign="top">
                    <div id="resultsplace">
                        @if (Model != null && Model.resultList != null)
                        {
                            // Show the result count.
                            <p class="sampleText">
                                @Html.DisplayFor(m => m.resultList.Count) Results
                            </p>

                            @for (var i = 0; i < Model.resultList.Results.Count; i++)
                            {
                                string amenities = string.Join(", ", Model.resultList.Results[i].Document.Tags);

                                string fullDescription = Model.resultList.Results[i].Document.Description;
                                fullDescription += $"\nCategory: {Model.resultList.Results[i].Document.Category}";
                                fullDescription += $"\nAmenities: {amenities}";

                                // Display the hotel name and description.
                                @Html.TextAreaFor(m => Model.resultList.Results[i].Document.HotelName, new { @class = "box1" })
                                @Html.TextArea($"desc{i}", fullDescription, new { @class = "box2" })
                            }
                        }
                    </div>
                </td>
            </tr>

            <tr>
                <td></td>
                <td valign="top">
                    @if (Model != null && Model.pageCount > 1)
                    {
                        // If there is more than one page of results, show the paging buttons.
                        <table>
                            <tr>
                                <td class="tdPage">
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

                                <td class="tdPage">
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
                                    <td class="tdPage">
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

                                <td class="tdPage">
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

                                <td class="tdPage">
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
                </td>
            </tr>
        </table>
    }
    </body>
    ```

    Observe el uso de la llamada a **Html.ActionLink**. Esta llamada comunica las cadenas de filtro válidas al controlador, cuando el usuario hace clic en un vínculo de faceta. 

### <a name="run-and-test-the-app"></a>Ejecutar y probar la aplicación

La ventaja de la navegación por facetas para el usuario es que este puede limitar las búsquedas con un solo clic, lo que podemos mostrar en la siguiente secuencia.

1. Ejecute la aplicación, escriba "airport" como el texto de búsqueda. Compruebe que la lista de facetas aparece de manera ordenada a la izquierda. Estas facetas son las que se aplican a los hoteles que tengan "airport" en sus datos de texto, con un recuento de la frecuencia con la que se producen.

    ![Usar la navegación por facetas para limitar una búsqueda de "airport"](./media/tutorial-csharp-create-first-app/azure-search-facet-airport.png)

2. Haga clic en la categoría **Resort and Spa**. Compruebe que todos los resultados están en esta categoría.

    ![Limitar la búsqueda a "Resort and Spa"](./media/tutorial-csharp-create-first-app/azure-search-facet-airport-ras.png)

3. Haga clic en el servicio **continental breakfast**. Compruebe que todos los resultados aún se encuentran en la categoría "Resort and Spa", con el servicio seleccionado.

    ![Limitar la búsqueda a "continental breakfast"](./media/tutorial-csharp-create-first-app/azure-search-facet-airport-ras-cb.png)

4. Pruebe a seleccionar otra categoría, luego un servicio, y consulte los resultados de limitación. Luego, pruebe lo inverso, un servicio y después una categoría.

    >[!Note]
    > Cuando se hace una selección en una lista de facetas (por ejemplo, una categoría), se invalidará toda selección anterior dentro de la lista de categorías.

## <a name="takeaways"></a>Puntos clave

Tenga en cuenta las siguientes conclusiones de este proyecto:

* Es imprescindible marcar cada propiedad como **IsFacetable**, si van a incluirse en la navegación por facetas.
* La navegación por facetas proporciona a un usuario una forma sencilla e intuitiva de limitar una búsqueda.
* Es recomendable que la navegación se divida en secciones (categorías de hotel, servicios de un hotel, rangos de precios, rangos de clasificación, etc.), con cada sección con un título apropiado.

## <a name="next-steps"></a>Pasos siguientes

En el siguiente tutorial, nos centramos en cómo ordenar los resultados. Llegado a este punto, los resultados se ordenan simplemente en el orden en que se encuentran en la base de datos.

> [!div class="nextstepaction"]
> [Tutorial de C#: Ordenación de los resultados: Azure Cognitive Search](tutorial-csharp-orders.md)
