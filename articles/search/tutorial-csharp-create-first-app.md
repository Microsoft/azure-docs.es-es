---
title: Tutorial de C# para crear la primera aplicación
titleSuffix: Azure Cognitive Search
description: Aprenda a crear su primera aplicación de búsqueda con C# paso a paso. En el tutorial se proporcionan un vínculo a una aplicación en funcionamiento en GitHub y el proceso completo para compilar la aplicación desde cero. Conozca los componentes esenciales de Azure Cognitive Search.
manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.service: cognitive-search
ms.topic: tutorial
ms.date: 06/20/2020
ms.custom: devx-track-csharp
ms.openlocfilehash: adcd6aa2c232bf87dc82284acbe2815484660ca7
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "88998567"
---
# <a name="tutorial-create-your-first-search-app-using-the-net-sdk"></a>Tutorial: Creación de la primera aplicación de búsqueda con el SDK de .NET

Aprenda a crear una interfaz web para consultar y presentar los resultados de búsqueda desde un índice mediante Azure Cognitive Search. Este tutorial comienza con un índice existente hospedado para que pueda centrarse en la creación de una página de búsqueda. El índice contiene datos ficticios de hotel. Cuando tenga una página básica, puede mejorarla en lecciones posteriores para incluir paginación, facetas y escritura automática.

En este tutorial, aprenderá a:
> [!div class="checklist"]
> * Configurar entorno de desarrollo
> * Modelar estructuras de datos
> * Crear una aplicación web
> * Definición de métodos
> * Prueba de la aplicación

También verá lo sencillo que es una llamada de búsqueda. Las instrucciones clave en el código que desarrollará están encapsuladas en las siguientes líneas.

```cs
var parameters = new SearchParameters
{
    // Enter Hotel property names into this list, so only these values will be returned.
    Select = new[] { "HotelName", "Description" }
};

DocumentSearchResult<Hotel> results  = await _indexClient.Documents.SearchAsync<Hotel>("search text", parameters);
```

Esta llamada única inicia una búsqueda de datos de Azure y devuelve los resultados.

![Buscar "pool"](./media/tutorial-csharp-create-first-app/azure-search-pool.png)


## <a name="prerequisites"></a>Prerrequisitos

Para completar este tutorial, necesita:

[Instale Visual Studio](https://visualstudio.microsoft.com/) para usarlo como IDE.

### <a name="install-and-run-the-project-from-github"></a>Instalar y ejecutar el proyecto de GitHub

1. Busque el ejemplo en GitHub: [Create first app](https://github.com/Azure-Samples/azure-search-dotnet-samples) (Crear la primera aplicación).
1. Seleccione **Clone or download** (Clonar o descargar) y cree su copia privada local del proyecto.
1. Con Visual Studio, vaya a la solución y ábrala para la página de búsqueda básica y seleccione **Iniciar sin depurar** (o presione F5).
1. Escriba algunas palabras (por ejemplo "wifi", "view", "bar", "parking") y examine los resultados.

    ![Buscar "wifi"](./media/tutorial-csharp-create-first-app/azure-search-wifi.png)

Con suerte, este proyecto se ejecutará sin problemas y la aplicación de Azure está en ejecución. En esta aplicación única se incluyen muchos de los componentes esenciales para búsquedas más sofisticadas, por lo que es una buena idea repasarlo y volver a crearlo paso a paso.

Para crear este proyecto desde cero y, por lo tanto, ayudar a reforzar los conocimientos sobre los componentes de Azure Cognitive Search, siga estos pasos.

## <a name="set-up-a-development-environment"></a>Configurar entorno de desarrollo

1. En Visual Studio 2017 o versiones posteriores, seleccione **Nuevo/Proyecto** y luego **Aplicación web ASP.NET Core**. Asigne un nombre al proyecto, como "FirstAzureSearchApp".

    ![Crear un proyecto en la nube](./media/tutorial-csharp-create-first-app/azure-search-project1.png)

2. Después de hacer clic en **Aceptar** para este tipo de proyecto, se le ofrecerá un segundo conjunto de opciones que se aplican a este proyecto. Seleccione **Aplicación web (Modelo-Vista-Controlador)** .

    ![Crear un proyecto Modelo-Vista-Controlador](./media/tutorial-csharp-create-first-app/azure-search-project2.png)

3. Después, en el menú **Herramientas**, seleccione **Administrador de paquetes NuGet** y luego **Administrar paquetes NuGet para la solución...** Hay un paquete que debemos instalar. Seleccione la pestaña **Examinar** y, luego, escriba "Azure Cognitive Search". Instale **Microsoft.Azure.Search** cuando aparezca en la lista (versión 9.0.1 o posterior). Tendrá que hacer clic en algunos cuadros de diálogo adicionales para completar la instalación.

    ![Uso de NuGet para agregar bibliotecas de Azure](./media/tutorial-csharp-create-first-app/azure-search-nuget-azure.png)

### <a name="initialize-azure-cognitive-search"></a>Inicialización de Azure Cognitive Search

Para este ejemplo, usamos datos de hotel disponibles públicamente. Estos datos son una colección arbitraria de 50 nombres de hotel y descripciones ficticios, creados exclusivamente para proporcionar datos de demostración. Para acceder a estos datos, debe especificar un nombre y la clave correspondiente.

1. Abra el archivo appsettings.json en el nuevo proyecto y reemplace las líneas predeterminadas por el siguiente nombre y clave. La clave de API que se muestra aquí no es un ejemplo de una clave, es _exactamente_ la clave que necesita para acceder a los datos de hotel. El archivo appsettings.json ahora debería tener este aspecto.

    ```cs
    {
        "SearchServiceName": "azs-playground",
        "SearchServiceQueryApiKey": "EA4510A6219E14888741FCFC19BFBB82"
    }
    ```

2. Todavía no hemos terminado con este archivo. Seleccione las propiedades para este archivo y cambie el valor **Copiar en el directorio de salida** en **Copiar si es más reciente**.

    ![Copiar la configuración de la aplicación a la salida](./media/tutorial-csharp-create-first-app/azure-search-copy-if-newer.png)

## <a name="model-data-structures"></a>Modelar estructuras de datos

Los modelos (clases de C#) se usan para comunicar datos entre el cliente (vista), el servidor (controlador) y también la nube de Azure mediante la arquitectura MVC (modelo, vista, controlador). Normalmente, estos modelos reflejan la estructura de los datos a los que se accede. Además, necesitamos un modelo para controlar las comunicaciones de controlador y vista.

1. Con el Explorador de soluciones, abra la carpeta **Modelos** del proyecto y verá un modelo predeterminado: **ErrorViewModel.cs**.

2. Haga clic con el botón derecho en la carpeta **Modelos**, seleccione **Agregar** y luego **Nuevo elemento**. Después, en el cuadro de diálogo que aparece, seleccione **ASP.NET Core** y luego la primera opción **Clase**. Cambie el nombre del archivo .cs a Hotel.cs y haga clic en **Agregar**. Reemplace todo el contenido de Hotel.cs por el código siguiente. Observe los miembros **Address** y **Room** de la clase; estos campos son clases en sí, por lo que necesitamos modelos para ellos también.

    ```cs
    using System;
    using Microsoft.Azure.Search;
    using Microsoft.Azure.Search.Models;
    using Microsoft.Spatial;
    using Newtonsoft.Json;

    namespace FirstAzureSearchApp.Models
    {
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
    }
    ```

3. Siga el mismo proceso de creación de un modelo para la clase **Address**, excepto el paso de asignar un nombre al archivo Address.cs. Reemplace el contenido por el siguiente.

    ```cs
    using Microsoft.Azure.Search;

    namespace FirstAzureSearchApp.Models
    {
        public partial class Address
        {
            [IsSearchable]
            public string StreetAddress { get; set; }

            [IsSearchable, IsFilterable, IsSortable, IsFacetable]
            public string City { get; set; }

            [IsSearchable, IsFilterable, IsSortable, IsFacetable]
            public string StateProvince { get; set; }

            [IsSearchable, IsFilterable, IsSortable, IsFacetable]
            public string PostalCode { get; set; }

            [IsSearchable, IsFilterable, IsSortable, IsFacetable]
            public string Country { get; set; }
        }
    }
    ```

4. Nuevamente, siga el mismo proceso para crear la clase **Room**, asignando el nombre Room.cs al archivo. Otra vez más, reemplace el contenido por el siguiente.

    ```cs
    using Microsoft.Azure.Search;
    using Microsoft.Azure.Search.Models;
    using Newtonsoft.Json;

    namespace FirstAzureSearchApp.Models
    {
        public partial class Room
        {
            [IsSearchable]
            [Analyzer(AnalyzerName.AsString.EnMicrosoft)]

            public string Description { get; set; }

            [IsSearchable]
            [Analyzer(AnalyzerName.AsString.FrMicrosoft)]
            [JsonProperty("Description_fr")]
            public string DescriptionFr { get; set; }

            [IsSearchable, IsFilterable, IsFacetable]
            public string Type { get; set; }

            [IsFilterable, IsFacetable]
            public double? BaseRate { get; set; }

            [IsSearchable, IsFilterable, IsFacetable]
            public string BedOptions { get; set; }

            [IsFilterable, IsFacetable]

            public int SleepsCount { get; set; }

            [IsFilterable, IsFacetable]
            public bool? SmokingAllowed { get; set; }

            [IsSearchable, IsFilterable, IsFacetable]
            public string[] Tags { get; set; }
        }
    }
    ```

5. El conjunto de clases **Hotel**, **Address** y **Room** es lo que se conoce en Azure como [_tipos complejos_](search-howto-complex-data-types.md), una característica importante de Azure Cognitive Search. Los tipos complejos pueden tener una profundidad de varios niveles de clases y subclases, y permiten la representación de estructuras de datos mucho más complejas que con el uso de _tipos simples_ (una clase que contiene solo miembros primitivos). Necesitamos un modelo más, por lo que pase otra vez por el proceso de crear una nueva clase de modelo, pero, esta vez asígnele el nombre SearchData.cs y reemplace el código predeterminado por lo siguiente.

    ```cs
    using Microsoft.Azure.Search.Models;

    namespace FirstAzureSearchApp.Models
    {
        public class SearchData
        {
            // The text to search for.
            public string searchText { get; set; }

            // The list of results.
            public DocumentSearchResult<Hotel> resultList;
        }
    }
    ```

    Esta clase contiene la entrada del usuario (**searchText**) y de salida de la búsqueda (**resultList**). El tipo de la salida es crítico, **DocumentSearchResult&lt;Hotel&gt;** , ya que este tipo coincide exactamente con los resultados de la búsqueda y necesitamos pasar esta referencia por la vista.



## <a name="create-a-web-page"></a>Crear una aplicación web

De manera predeterminada, el proyecto que creó creará un número de vistas de cliente. Las vistas exactas dependen de la versión de .NET Core que use (en este ejemplo, usamos 2.1). Todas se encuentran en la carpeta **Vistas** del proyecto. Solo tendrá que modificar el archivo Index.cshtml (en la carpeta **Vistas/Inicio**).

Elimine el contenido de Index.cshtml en su totalidad y vuelva a generar el archivo con los pasos siguientes.

1. Usamos dos imágenes pequeñas en la vista. Puede usar una imagen propia o copiar entre las imágenes del proyecto de GitHub: azure-logo.png y search.png. Estas dos imágenes deben colocarse en la carpeta **wwwroot/images**.

2. La primera línea de Index.cshtml debe hacer referencia al modelo que se va a usar para comunicar datos entre el cliente (vista) y el servidor (controlador), que es el modelo **SearchData** que creamos. Agregue esta línea al archivo Index.cshtml.

    ```cs
    @model FirstAzureSearchApp.Models.SearchData
    ```

3. La práctica habitual es escribir un título para la vista, por lo que las líneas siguientes deberían ser:

    ```cs
    @{
        ViewData["Title"] = "Home Page";
    }
    ```

4. Después del título, escriba una referencia a una hoja de estilos HTML que crearemos en breve.

    ```cs
    <head>
        <link rel="stylesheet" href="~/css/hotels.css" />
    </head>
    ```

5. Ahora pasemos al núcleo de la vista. Un elemento clave que debe recordar es que la vista tiene que controlar dos situaciones. En primer lugar, debe controlar la visualización cuando la aplicación se inicia por primera vez, y el usuario aún no ha especificado ningún texto de búsqueda. En segundo lugar, debe controlar la visualización de los resultados, además del cuadro de texto de búsqueda, para su uso repetido por el usuario. Para controlar estas dos situaciones, es necesario comprobar si el modelo proporcionado a la vista es null o no. Un modelo nulo indica que estamos en la primera de las dos situaciones (la ejecución inicial de la aplicación). Agregue lo siguiente al archivo Index.cshtml y lea los comentarios.

    ```cs
    <body>
    <h1 class="sampleTitle">
        <img src="~/images/azure-logo.png" width="80" />
        Hotels Search
    </h1>

    @using (Html.BeginForm("Index", "Home", FormMethod.Post))
    {
        // Display the search text box, with the search icon to the right of it.
        <div class="searchBoxForm">
            @Html.TextBoxFor(m => m.searchText, new { @class = "searchBox" }) <input class="searchBoxSubmit" type="submit" value="">
        </div>

        @if (Model != null)
        {
            // Show the result count.
            <p class="sampleText">
                @Html.DisplayFor(m => m.resultList.Results.Count) Results
            </p>

            @for (var i = 0; i < Model.resultList.Results.Count; i++)
            {
                // Display the hotel name and description.
                @Html.TextAreaFor(m => Model.resultList.Results[i].Document.HotelName, new { @class = "box1" })
                @Html.TextArea($"desc{i}", Model.resultList.Results[i].Document.Description, new { @class = "box2" })
            }
        }
    }
    </body>
    ```

6. Por último, agregamos la hoja de estilos. En Visual Studio, en el menú **Archivo**, seleccione **Nuevo/Archivo** y luego **Hoja de estilos** (con la opción **General** resaltada). Reemplace el código predeterminado por el siguiente. No entraremos en más detalles en este archivo; los estilos son estilos HTML estándar.

    ```html
    textarea.box1 {
        width: 648px;
        height: 30px;
        border: none;
        background-color: azure;
        font-size: 14pt;
        color: blue;
        padding-left: 5px;
    }

    textarea.box2 {
        width: 648px;
        height: 100px;
        border: none;
        background-color: azure;
        font-size: 12pt;
        padding-left: 5px;
        margin-bottom: 24px;
    }

    .sampleTitle {
        font: 32px/normal 'Segoe UI Light',Arial,Helvetica,Sans-Serif;
        margin: 20px 0;
        font-size: 32px;
        text-align: left;
    }

    .sampleText {
        font: 16px/bold 'Segoe UI Light',Arial,Helvetica,Sans-Serif;
        margin: 20px 0;
        font-size: 14px;
        text-align: left;
        height: 30px;
    }

    .searchBoxForm {
        width: 648px;
        box-shadow: 0 0 0 1px rgba(0,0,0,.1), 0 2px 4px 0 rgba(0,0,0,.16);
        background-color: #fff;
        display: inline-block;
        border-collapse: collapse;
        border-spacing: 0;
        list-style: none;
        color: #666;
    }

    .searchBox {
        width: 568px;
        font-size: 16px;
        margin: 5px 0 1px 20px;
        padding: 0 10px 0 0;
        border: 0;
        max-height: 30px;
        outline: none;
        box-sizing: content-box;
        height: 35px;
        vertical-align: top;
    }

    .searchBoxSubmit {
        background-color: #fff;
        border-color: #fff;
        background-image: url(/images/search.png);
        background-repeat: no-repeat;
        height: 20px;
        width: 20px;
        text-indent: -99em;
        border-width: 0;
        border-style: solid;
        margin: 10px;
        outline: 0;
    }
    ```

7. Guarde el archivo de hoja de estilos como hotels.css, en la carpeta wwwroot/css, junto con el archivo site.css predeterminado.

Esto completa nuestra vista. Estamos por el buen camino. Los modelos y vistas se completaron, solo falta el controlador vincular todo.

## <a name="define-methods"></a>Definición de métodos

Es necesario modificar el contenido del controlador único (controlador **Home**), que se crea de forma predeterminada.

1. Abra el archivo HomeController.cs y reemplace las instrucciones **using** por lo siguiente.

    ```cs
    using System;
    using System.Diagnostics;
    using System.Threading.Tasks;
    using Microsoft.AspNetCore.Mvc;
    using FirstAzureSearchApp.Models;
    using Microsoft.Extensions.Configuration;
    using Microsoft.Azure.Search;
    using Microsoft.Azure.Search.Models;
    ```

### <a name="add-index-methods"></a>Agregar métodos de índice

Necesitamos dos métodos **Index**, uno que no toma ningún parámetro (para la situación en la que la aplicación abre por primera vez) y otro toma un modelo como parámetro (para cuando el usuario ha especificado texto de búsqueda). El primero de estos métodos se crea de manera predeterminada. 

1. Agregue el método siguiente, después del método **Index()** predeterminado.

    ```cs
        [HttpPost]
        public async Task<ActionResult> Index(SearchData model)
        {
            try
            {
                // Ensure the search string is valid.
                if (model.searchText == null)
                {
                    model.searchText = "";
                }

                // Make the Azure Cognitive Search call.
                await RunQueryAsync(model);
            }

            catch
            {
                return View("Error", new ErrorViewModel { RequestId = "1" });
            }
            return View(model);
        }
    ```

    Observe la declaración **async** del método y la llamada **await** a **RunQueryAsync**. Estas palabras clave se encargan de realizar las llamadas asincrónicas y así evitar el bloqueo de subprocesos en el servidor.

    El bloque **catch** usa el modelo de error que se creó automáticamente de manera predeterminada.

### <a name="note-the-error-handling-and-other-default-views-and-methods"></a>Observar el control de errores y otras vistas y métodos predeterminados

Según la versión de .NET Core que usa, de manera predeterminada se crea un conjunto diferente de vistas predeterminadas. Para .NET Core 2.1, las vistas predeterminadas son el Índice, Acerca de, Contacto, Privacidad y Error. Para .NET Core 2.2, por ejemplo, las vistas predeterminadas son Índice, Privacidad y Error. En cualquiera de los casos, puede ver estas páginas predeterminadas al ejecutar la aplicación y examinar cómo se controlan en el controlador.

Vamos a probar la vista Error más adelante en este tutorial.

En el ejemplo de GitHub, eliminamos las vistas sin usar y sus acciones asociadas.

### <a name="add-the-runqueryasync-method"></a>Agregar el método RunQueryAsync

La llamada a Azure Cognitive Search se encapsula en nuestro método **RunQueryAsync**.

1. En primer lugar, agregue algunas variables estáticas para configurar el servicio de Azure y una llamada para iniciarlas.

    ```cs
        private static SearchServiceClient _serviceClient;
        private static ISearchIndexClient _indexClient;
        private static IConfigurationBuilder _builder;
        private static IConfigurationRoot _configuration;

        private void InitSearch()
        {
            // Create a configuration using the appsettings file.
            _builder = new ConfigurationBuilder().AddJsonFile("appsettings.json");
            _configuration = _builder.Build();

            // Pull the values from the appsettings.json file.
            string searchServiceName = _configuration["SearchServiceName"];
            string queryApiKey = _configuration["SearchServiceQueryApiKey"];

            // Create a service and index client.
            _serviceClient = new SearchServiceClient(searchServiceName, new SearchCredentials(queryApiKey));
            _indexClient = _serviceClient.Indexes.GetClient("hotels");
        }
    ```

2. Ahora, agregue el método **RunQueryAsync** en sí.

    ```cs
        private async Task<ActionResult> RunQueryAsync(SearchData model)
        {
            InitSearch();

            var parameters = new SearchParameters
            {
                // Enter Hotel property names into this list so only these values will be returned.
                // If Select is empty, all values will be returned, which can be inefficient.
                Select = new[] { "HotelName", "Description" }
            };

            // For efficiency, the search call should be asynchronous, so use SearchAsync rather than Search.
            model.resultList = await _indexClient.Documents.SearchAsync<Hotel>(model.searchText, parameters);

            // Display the results.
            return View("Index", model);
        }
    ```

    En este método, primero nos aseguramos de que nuestra configuración de Azure se haya iniciado y luego configuramos algunos parámetros de búsqueda. Los nombres de los campos del parámetro **Select** coinciden exactamente con los nombres de propiedad en la clase **hotel**. Es posible omitir el parámetro **Select**, en cuyo caso, se devuelven todas las propiedades. Sin embargo, no establecer parámetros **Select** es poco eficiente si solo nos interesa un subconjunto de los datos. Al especificar las propiedades que nos interesan, se devuelven solo estas propiedades.

    La llamada asincrónica para buscar (**model.resultList = await _indexClient.Documents.SearchAsync&lt;Hotel&gt;(model.searchText, parámetros);** ) es el propósito de este tutorial y la aplicación. La clase **DocumentSearchResult** es un elemento interesante. Es recomendable (cuando la aplicación esté en ejecución) establecer un punto de interrupción aquí y usar un depurador para examinar el contenido de **model.resultList**. Verá que este proceso es intuitivo, ya que le proporciona los datos que se piden y no mucho más.

Ahora llegó el momento de la verdad.

### <a name="test-the-app"></a>Prueba de la aplicación

Comprobemos que la aplicación se ejecuta correctamente.

1. Seleccione **Depurar/Iniciar sin depurar** o presione la tecla F5. Si todo se ha codificado correctamente, obtendrá la vista Índice inicial.

     ![Abrir la aplicación](./media/tutorial-csharp-create-first-app/azure-search-index.png)

2. Escriba texto, como "beach" (o cualquier texto que venga a la mente) y haga clic en el icono de búsqueda. Debería obtener algunos resultados.

     ![Buscar "beach"](./media/tutorial-csharp-create-first-app/azure-search-beach.png)

3. Pruebe a escribir "five star". Observe que no se devuelven resultados. Una búsqueda más sofisticada sería tratar "five star" como sinónimo de "luxury" y hacer que se devuelvan esos resultados. El uso de sinónimos está disponible en Azure Cognitive Search, aunque no se tratará en los tutoriales iniciales.
 
4. Pruebe a escribir "hot" como texto de búsqueda. _No_ devuelve entradas con la palabra "hotel" en ellas. Nuestra búsqueda solo busca palabras completas, aunque se devuelven algunos resultados.

5. Pruebe otras palabras: "pool", "sunshine", "view" y lo que sea. Verá que Azure Cognitive Search funciona en su nivel más sencillo, pero no por ello menos convincente.

## <a name="test-edge-conditions-and-errors"></a>Probar las condiciones de borde y los errores

Es importante comprobar que las características de control de errores funcionan del modo esperado, incluso cuando todo funciona perfectamente. 

1. En el método **Index**, después de la llamada **try {** , escriba la línea **Throw new Exception()** . Esta excepción fuerza un error cuando se realizan búsquedas de texto.

2. Ejecute la aplicación, escriba "bar" como texto de búsqueda y haga clic en el icono de búsqueda. La excepción debería provocar una vista de error.

     ![Forzar un error](./media/tutorial-csharp-create-first-app/azure-search-error.png)

    > [!Important]
    > Se considera un riesgo de seguridad devolver números de error interno en las páginas de error. Si la aplicación está pensada para uso general, investigue la seguridad y los procedimientos recomendados sobre lo que se debe devolver cuando se produce un error.

3. Cuando esté satisfecho del funcionamiento previsto del control de errores, quite **Throw new Exception()** .

## <a name="takeaways"></a>Puntos clave

Tenga en cuenta las siguientes conclusiones de este proyecto:

* Una llamada a Azure Cognitive Search es concisa, y los resultados son fáciles de interpretar.
* Las llamadas asincrónicas agregan una pequeña cantidad de complejidad al controlador, pero son el procedimiento recomendado si va a desarrollar aplicaciones de calidad.
* Esta aplicación realizó una búsqueda de texto sencillo, definida por lo que se configura en **searchParameters**. Sin embargo, esta clase única se puede rellenar con muchos miembros que agregan sofisticación a una búsqueda. No hace falta mucho trabajo adicional para hacer que esta aplicación sea mucho más potente.

## <a name="next-steps"></a>Pasos siguientes

Para ofrecer la mejor experiencia de usuario con Azure Cognitive Search, es necesario agregar más características, en particular, la paginación (ya sea mediante números de página o desplazamiento infinito), así como opciones de autocompletar y sugerencias. También deberíamos considerar parámetros de búsqueda más sofisticados (por ejemplo, búsquedas geográficas de hoteles dentro de un radio especificado de un punto dado y el ordenamiento de los resultados de búsqueda).

Estos pasos siguientes se tratan en una serie de tutoriales. Comencemos por la paginación.

> [!div class="nextstepaction"]
> [Tutorial de C#: Paginación de los resultados de Azure Cognitive Search](tutorial-csharp-paging.md)


