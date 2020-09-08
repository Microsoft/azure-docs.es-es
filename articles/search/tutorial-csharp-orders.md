---
title: Tutorial de C# sobre la ordenación de los resultados
titleSuffix: Azure Cognitive Search
description: En este tutorial de C# se muestra cómo ordenar los resultados de la búsqueda. Se basa en un proyecto de hoteles anterior, que se ordena por propiedad principal y propiedad secundaria, y que incluye un perfil de puntuación para agregar criterios de mejora.
manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.service: cognitive-search
ms.topic: tutorial
ms.date: 06/20/2020
ms.custom: devx-track-javascript, devx-track-csharp
ms.openlocfilehash: 9819615039a6dd6aceec796a3ec1c7338f6f3968
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "88998533"
---
# <a name="tutorial-order-search-results-using-the-net-sdk"></a>Tutorial: Ordenación de los resultados de búsqueda mediante el SDK de .NET

Hasta este punto en nuestra serie de tutoriales, los resultados se devuelven y se muestran en un orden predeterminado. Esto puede ser el orden en que se encuentran los datos, o posiblemente se ha definido un _perfil de puntuación_ predeterminado, que se usará cuando no se especifique ningún parámetro de ordenamiento. En este tutorial, describiremos cómo ordenar los resultados según una propiedad principal y luego, para los resultados con la misma propiedad principal, cómo ordenar esa selección según una propiedad secundaria. Como alternativa al ordenamiento basado en valores numéricos, en el último ejemplo se muestra cómo ordenar según un perfil de puntuación personalizado. También describiremos un poco más la visualización de _tipos complejos_.

Para comparar fácilmente los resultados devueltos, este proyecto se basa en el proyecto de desplazamiento infinito que creó en el [Tutorial de C#: Paginación de los resultados de búsqueda: Azure Cognitive Search](tutorial-csharp-paging.md).

En este tutorial, aprenderá a:
> [!div class="checklist"]
> * Ordenar los resultados según una propiedad
> * Ordenar los resultados de orden según varias propiedades
> * Filtrar los resultados según una distancia desde un punto geográfico
> * Ordenar los resultados según un perfil de puntuación

## <a name="prerequisites"></a>Prerrequisitos

Para completar este tutorial, necesita:

Haber abierto y tener en ejecución el proyecto [Tutorial de C#: El proyecto Paginación de los resultados de búsqueda: Azure Cognitive Search](tutorial-csharp-paging.md) en funcionamiento. Este proyecto puede ser su propia versión o puede instalarlo desde GitHub: [Create first app](https://github.com/Azure-Samples/azure-search-dotnet-samples) (Crear la primera aplicación).

## <a name="order-results-based-on-one-property"></a>Ordenar los resultados según una propiedad

Cuando se orden los resultados según una propiedad, por ejemplo, clasificación de hotel, no solo queremos ordenar los resultados, también queremos confirmación de que el orden es correcto. Es decir, si ordenamos según la clasificación, deberíamos mostrar la clasificación en la vista.

En este tutorial, también agregaremos un poco más a la visualización de los resultados (la tarifa más baja de habitación y la tarifa más alta de habitación) para cada hotel. A medida que nos adentremos en el ordenamiento, también agregaremos valores para asegurarnos de que lo que estamos ordenando también se muestre en la vista.

No hace falta modificar ninguno de los modelos para habilitar el ordenamiento. Es necesario actualizar la vista y el controlador. Para comenzar, abra el controlador principal.

### <a name="add-the-orderby-property-to-the-search-parameters"></a>Agregar la propiedad OrderBy a los parámetros de búsqueda

1. Lo único que hace falta hacer para ordenar los resultados según una única propiedad numérica es establecer el parámetro **OrderBy** en el nombre de la propiedad. En el siguiente método **Index(SearchData model)** , agregue la siguiente línea a los parámetros de búsqueda.

    ```cs
        OrderBy = new[] { "Rating desc" },
    ```

    >[!Note]
    > El orden predeterminado es ascendente, aunque puede agregar **asc** a la propiedad para aclarar este punto. El orden descendente se especifica mediante la adición de **desc**.

2. Ahora ejecute la aplicación y escriba un término de búsqueda común. Los resultados pueden o no estar en el orden correcto, ya que ni el desarrollador ni el usuario tienen una forma sencilla de comprobar los resultados.

3. Confirmemos que los resultados se ordenan según la clasificación. En primer lugar, reemplace las clases **box1** y **box2** del archivo hotels.css por las siguientes clases (estas clases son todas las nuevas que necesitamos para este tutorial).

    ```html
    textarea.box1A {
        width: 324px;
        height: 32px;
        border: none;
        background-color: azure;
        font-size: 14pt;
        color: blue;
        padding-left: 5px;
        text-align: left;
    }

    textarea.box1B {
        width: 324px;
        height: 32px;
        border: none;
        background-color: azure;
        font-size: 14pt;
        color: blue;
        text-align: right;
        padding-right: 5px;
    }

    textarea.box2A {
        width: 324px;
        height: 32px;
        border: none;
        background-color: azure;
        font-size: 12pt;
        color: blue;
        padding-left: 5px;
        text-align: left;
    }

    textarea.box2B {
        width: 324px;
        height: 32px;
        border: none;
        background-color: azure;
        font-size: 12pt;
        color: blue;
        text-align: right;
        padding-right: 5px;
    }

    textarea.box3 {
        width: 648px;
        height: 100px;
        border: none;
        background-color: azure;
        font-size: 12pt;
        padding-left: 5px;
        margin-bottom: 24px;
    }
    ```

    >[!Tip]
    >Los exploradores suelen almacenar archivos css en caché, lo que puede resultar en el uso de un archivo css antiguo y la omisión de las modificaciones. Una buena manera de evitar esto es agregar al vínculo una cadena de consulta con un parámetro de versión. Por ejemplo:
    >
    >```html
    >   <link rel="stylesheet" href="~/css/hotels.css?v1.1" />
    >```
    >
    >Actualice el número de versión solamente si cree que el explorador está usando un archivo css antiguo.

4. Agregue la propiedad **Rating** al parámetro **Select**, en el método **Index(SearchData model)** .

    ```cs
    Select = new[] { "HotelName", "Description", "Rating"},
    ```

5. Abra la vista (index.cshtml) y reemplace el bucle de representación ( **&lt;!-- Show the hotel data. --&gt;** ) por el código siguiente.

    ```cs
                <!-- Show the hotel data. -->
                @for (var i = 0; i < Model.resultList.Results.Count; i++)
                {
                    var ratingText = $"Rating: {Model.resultList.Results[i].Document.Rating}";

                    // Display the hotel details.
                    @Html.TextArea($"name{i}", Model.resultList.Results[i].Document.HotelName, new { @class = "box1A" })
                    @Html.TextArea($"rating{i}", ratingText, new { @class = "box1B" })
                    @Html.TextArea($"desc{i}", Model.resultList.Results[i].Document.Description, new { @class = "box3" })
                }
    ```

6. La clasificación debe estar disponible tanto la primera página que se muestra como en las páginas subsiguientes a las que se llaman mediante el desplazamiento infinito. En la segunda de estas dos situaciones, es necesario actualizar tanto la acción **Next** del controlador como la función **scrolled** de la vista. Comenzando por el controlador, cambie el método **Next** al código siguiente. Este código crea y comunica el texto de la clasificación.

    ```cs
        public async Task<ActionResult> Next(SearchData model)
        {
            // Set the next page setting, and call the Index(model) action.
            model.paging = "next";
            await Index(model);

            // Create an empty list.
            var nextHotels = new List<string>();

            // Add a hotel details to the list.
            for (int n = 0; n < model.resultList.Results.Count; n++)
            {
                var ratingText = $"Rating: {model.resultList.Results[n].Document.Rating}";

                // Add three strings to the list.
                nextHotels.Add(model.resultList.Results[n].Document.HotelName);
                nextHotels.Add(ratingText);
                nextHotels.Add(model.resultList.Results[n].Document.Description);
            }

            // Rather than return a view, return the list of data.
            return new JsonResult(nextHotels);
        }
    ```

7. Ahora, actualice la función **scrolled** en la vista para mostrar el texto de la clasificación.

    ```javascript
            <script>
                function scrolled() {
                    if (myDiv.offsetHeight + myDiv.scrollTop >= myDiv.scrollHeight) {
                        $.getJSON("/Home/Next", function (data) {
                            var div = document.getElementById('myDiv');

                            // Append the returned data to the current list of hotels.
                            for (var i = 0; i < data.length; i += 3) {
                                div.innerHTML += '\n<textarea class="box1A">' + data[i] + '</textarea>';
                                div.innerHTML += '\n<textarea class="box1B">' + data[i + 1] + '</textarea>';
                                div.innerHTML += '\n<textarea class="box3">' + data[i + 2] + '</textarea>';
                            }
                        });
                    }
                }
            </script>

    ```

8. Ahora, vuelva a ejecutar la aplicación. Busque cualquier término común, como "wifi", y compruebe que los resultados se ordenan en orden descendente de clasificación de hotel.

    ![Ordenamiento basado en la clasificación](./media/tutorial-csharp-create-first-app/azure-search-orders-rating.png)

    Observará que varios hoteles tienen una clasificación idéntica, por lo que su visualización en pantalla es de nuevo en el orden en el que se encuentran los datos, que es arbitrario.

    Antes de agregar un segundo nivel de ordenamiento, agreguemos algo de código para mostrar el rango de tarifas de habitación. Agregamos este código a para mostrar los datos de extracción de un _tipo complejo_ y también para que podamos analizar los resultados de ordenamiento en función del precio (por ejemplo, el precio más bajo primero).

### <a name="add-the-range-of-room-rates-to-the-view"></a>Agregar un rango de tarifas de habitación a la vista

1. Agregue las propiedades que contengan la tarifa de habitación más económica y la más cara al modeloHotel.cs.

    ```cs
        // Room rate range
        public double cheapest { get; set; }
        public double expensive { get; set; }
    ```

2. Calcule las tarifas de habitación al final de la acción **Index(SearchData model)** , en el controlador principal. Agregue los cálculos después de almacenar los datos temporales.

    ```cs
                // Ensure TempData is stored for the next call.
                TempData["page"] = page;
                TempData["searchfor"] = model.searchText;

                // Calculate the room rate ranges.
                for (int n = 0; n < model.resultList.Results.Count; n++)
                {
                    // Calculate room rates.
                    var cheapest = 0d;
                    var expensive = 0d;

                    for (var r = 0; r < model.resultList.Results[n].Document.Rooms.Length; r++)
                    {
                        var rate = model.resultList.Results[n].Document.Rooms[r].BaseRate;
                        if (rate < cheapest || cheapest == 0)
                        {
                            cheapest = (double)rate;
                        }
                        if (rate > expensive)
                        {
                            expensive = (double)rate;
                        }
                    }
                    model.resultList.Results[n].Document.cheapest = cheapest;
                    model.resultList.Results[n].Document.expensive = expensive;
                }
    ```

3. Agregue la propiedad **Rooms** al parámetro **Select** en el método de acción **Index(SearchData model)** del controlador.

    ```cs
     Select = new[] { "HotelName", "Description", "Rating", "Rooms" },
    ```

4. Cambie el bucle de representación en la vista para mostrar el rango de tarifas para la primera página de resultados.

    ```cs
                <!-- Show the hotel data. -->
                @for (var i = 0; i < Model.resultList.Results.Count; i++)
                {
                    var rateText = $"Rates from ${Model.resultList.Results[i].Document.cheapest} to ${Model.resultList.Results[i].Document.expensive}";
                    var ratingText = $"Rating: {Model.resultList.Results[i].Document.Rating}";

                    // Display the hotel details.
                    @Html.TextArea($"name{i}", Model.resultList.Results[i].Document.HotelName, new { @class = "box1A" })
                    @Html.TextArea($"rating{i}", ratingText, new { @class = "box1B" })
                    @Html.TextArea($"rates{i}" , rateText, new { @class = "box2A" })
                    @Html.TextArea($"desc{i}", Model.resultList.Results[i].Document.Description, new { @class = "box3" })
                }
    ```

5. Cambie el método **Next** en el controlador principal para comunicar el rango de tarifas para las páginas de resultados subsiguientes.

    ```cs
        public async Task<ActionResult> Next(SearchData model)
        {
            // Set the next page setting, and call the Index(model) action.
            model.paging = "next";
            await Index(model);

            // Create an empty list.
            var nextHotels = new List<string>();

            // Add a hotel details to the list.
            for (int n = 0; n < model.resultList.Results.Count; n++)
            {
                var ratingText = $"Rating: {model.resultList.Results[n].Document.Rating}";
                var rateText = $"Rates from ${model.resultList.Results[n].Document.cheapest} to ${model.resultList.Results[n].Document.expensive}";

                // Add strings to the list.
                nextHotels.Add(model.resultList.Results[n].Document.HotelName);
                nextHotels.Add(ratingText);
                nextHotels.Add(rateText);
                nextHotels.Add(model.resultList.Results[n].Document.Description);
            }

            // Rather than return a view, return the list of data.
            return new JsonResult(nextHotels);
        }
    ```

6. Actualice la función **scrolled** en la vista para controlar el texto de tarifas de habitación.

    ```javascript
            <script>
                function scrolled() {
                    if (myDiv.offsetHeight + myDiv.scrollTop >= myDiv.scrollHeight) {
                        $.getJSON("/Home/Next", function (data) {
                            var div = document.getElementById('myDiv');

                            // Append the returned data to the current list of hotels.
                            for (var i = 0; i < data.length; i += 4) {
                                div.innerHTML += '\n<textarea class="box1A">' + data[i] + '</textarea>';
                                div.innerHTML += '\n<textarea class="box1B">' + data[i + 1] + '</textarea>';
                                div.innerHTML += '\n<textarea class="box2A">' + data[i + 2] + '</textarea>';
                                div.innerHTML += '\n<textarea class="box3">' + data[i + 4] + '</textarea>';
                            }
                        });
                    }
                }
            </script>
    ```

7. Ejecute la aplicación y compruebe que se muestran los rangos de precios de habitación.

    ![Mostrar rangos de precios de habitación](./media/tutorial-csharp-create-first-app/azure-search-orders-rooms.png)

La propiedad **OrderBy** de los parámetros de búsqueda no aceptará una entrada como, por ejemplo, **Rooms.BaseRate**, para proporcionar la tarifa de habitación más económica, incluso si las habitaciones ya estaban ordenadas según la tarifa. En este caso, las habitaciones no se ordenan según la tarifa. Para mostrar hoteles en el conjunto de datos de ejemplo, ordenados según la tarifa de habitación, tendría que ordenar los resultados en el controlador principal y enviar estos a la vista en el orden deseado.

## <a name="order-results-based-on-multiple-values"></a>Ordenar los resultados según varios valores

Ahora la pregunta es cómo diferenciar entre los hoteles con la misma clasificación. Una buena manera sería ordenar según la última vez que se ha reformado el hotel. Es decir, cuanto más recientes sean las reformas de un hotel, más alto aparecerá el hotel en los resultados.

1. Para agregar un segundo nivel de ordenamiento, cambie las propiedades **OrderBy** y **Select** en el método **Index(SearchData model)** de modo que se incluya la propiedad  **LastRenovationDate**.

    ```cs
    OrderBy = new[] { "Rating desc", "LastRenovationDate desc" },
    Select = new[] { "HotelName", "Description", "Rating", "Rooms", "LastRenovationDate" },
    ```

    >[!Tip]
    >Puede especificar cualquier número de propiedades en la lista **OrderBy**. Si los hoteles tuvieran la misma clasificación y fecha de reforma, se puede especificar una tercera propiedad para diferenciar entre ellos.

2. Nuevamente, es necesario ver la fecha de reforma en la vista, simplemente para estar seguro de que el ordenamiento es correcto. Para alguno como una reforma, es probable que solo se necesite el año. Cambie el bucle de representación en la vista por el código siguiente.

    ```cs
                <!-- Show the hotel data. -->
                @for (var i = 0; i < Model.resultList.Results.Count; i++)
                {
                    var rateText = $"Rates from ${Model.resultList.Results[i].Document.cheapest} to ${Model.resultList.Results[i].Document.expensive}";
                    var lastRenovatedText = $"Last renovated: { Model.resultList.Results[i].Document.LastRenovationDate.Value.Year}";
                    var ratingText = $"Rating: {Model.resultList.Results[i].Document.Rating}";

                    // Display the hotel details.
                    @Html.TextArea($"name{i}", Model.resultList.Results[i].Document.HotelName, new { @class = "box1A" })
                    @Html.TextArea($"rating{i}", ratingText, new { @class = "box1B" })
                    @Html.TextArea($"rates{i}" , rateText, new { @class = "box2A" })
                    @Html.TextArea($"renovation{i}", lastRenovatedText, new { @class = "box2B" })
                    @Html.TextArea($"desc{i}", Model.resultList.Results[i].Document.Description, new { @class = "box3" })
                }
    ```

3. Cambie el método **Next** en el controlador principal para reenviar el componente de año de la última fecha de reforma.

    ```cs
        public async Task<ActionResult> Next(SearchData model)
        {
            // Set the next page setting, and call the Index(model) action.
            model.paging = "next";
            await Index(model);

            // Create an empty list.
            var nextHotels = new List<string>();

            // Add a hotel details to the list.
            for (int n = 0; n < model.resultList.Results.Count; n++)
            {
                var ratingText = $"Rating: {model.resultList.Results[n].Document.Rating}";
                var rateText = $"Rates from ${model.resultList.Results[n].Document.cheapest} to ${model.resultList.Results[n].Document.expensive}";
                var lastRenovatedText = $"Last renovated: {model.resultList.Results[n].Document.LastRenovationDate.Value.Year}";

                // Add strings to the list.
                nextHotels.Add(model.resultList.Results[n].Document.HotelName);
                nextHotels.Add(ratingText);
                nextHotels.Add(rateText);
                nextHotels.Add(lastRenovatedText);
                nextHotels.Add(model.resultList.Results[n].Document.Description);
            }

            // Rather than return a view, return the list of data.
            return new JsonResult(nextHotels);
        }
    ```

4. Cambie la función **scrolled** en la vista para mostrar el texto de reforma.

    ```javascript
            <script>
                function scrolled() {
                    if (myDiv.offsetHeight + myDiv.scrollTop >= myDiv.scrollHeight) {
                        $.getJSON("/Home/Next", function (data) {
                            var div = document.getElementById('myDiv');

                            // Append the returned data to the current list of hotels.
                            for (var i = 0; i < data.length; i += 5) {
                                div.innerHTML += '\n<textarea class="box1A">' + data[i] + '</textarea>';
                                div.innerHTML += '\n<textarea class="box1B">' + data[i + 1] + '</textarea>';
                                div.innerHTML += '\n<textarea class="box2A">' + data[i + 2] + '</textarea>';
                                div.innerHTML += '\n<textarea class="box2B">' + data[i + 3] + '</textarea>';
                                div.innerHTML += '\n<textarea class="box3">' + data[i + 4] + '</textarea>';
                            }
                        });
                    }
                }
            </script>
    ```

5. Ejecute la aplicación. Busque un término común, por ejemplo "pool" o "view", y compruebe que los hoteles con la misma clasificación ahora se muestran en orden descendente según la fecha de reforma.

    ![Ordenar según la fecha de reforma](./media/tutorial-csharp-create-first-app/azure-search-orders-renovation.png)

## <a name="filter-results-based-on-a-distance-from-a-geographical-point"></a>Filtrar los resultados según una distancia desde un punto geográfico

La clasificación y la fecha de reforma son ejemplos de propiedades que mejor se muestran en orden descendente. Una lista alfabética sería un ejemplo de un buen uso de orden ascendente (por ejemplo, si hubiera solo una propiedad **OrderBy** y esta se estableciera en **HotelName**, se mostraría un orden alfabético ). Sin embargo, para los datos de ejemplo, la distancia desde un punto geográfico sería más adecuada.

Para mostrar los resultados según la distancia geográfica, se requieren varios pasos.

1. Para filtrar todos los hoteles que se encuentran fuera de un radio especificado desde un punto dado, especifique un filtro con parámetros de longitud, latitud y radio. La longitud se asigna primero a la función POINT. El radio es en kilómetros.

    ```cs
        // "Location" must match the field name in the Hotel class.
        // Distance (the radius) is in kilometers.
        // Point order is Longitude then Latitude.
        Filter = $"geo.distance(Location, geography'POINT({model.lon} {model.lat})') le {model.radius}",
    ```

2. El filtro anterior _no_ ordena los resultados según la distancia, simplemente quita los valores atípicos. Para ordenar los resultados, escriba un valor de **OrderBy** que especifique el método geoDistance.

    ```cs
    OrderBy = new[] { $"geo.distance(Location, geography'POINT({model.lon} {model.lat})') asc" },
    ```

3. Aunque Azure Cognitive Search devolvió los resultados mediante un filtro de distancia, la distancia calculada entre los datos y el punto especificado _no_ se devuelve. Si quiere mostrarla en los resultados, vuelva a calcular este valor en la vista o el controlador.

    El código siguiente calcula la distancia entre dos puntos de latitud y longitud.

    ```cs
        const double EarthRadius = 6371;

        public static double Degrees2Radians(double deg)
        {
            return deg * Math.PI / 180;
        }

        public static double DistanceInKm( double lat1,  double lon1, double lat2, double lon2)
        {
            double dlon = Degrees2Radians(lon2 - lon1);
            double dlat = Degrees2Radians(lat2 - lat1);

            double a = (Math.Sin(dlat / 2) * Math.Sin(dlat / 2)) + Math.Cos(Degrees2Radians(lat1)) * Math.Cos(Degrees2Radians(lat2)) * (Math.Sin(dlon / 2) * Math.Sin(dlon / 2));
            double angle = 2 * Math.Atan2(Math.Sqrt(a), Math.Sqrt(1 - a));
            return angle * EarthRadius;
        }
    ```

4. Ahora, tiene que enlazar estos conceptos. Sin embargo, en este tutorial solo llega a estos fragmentos de código, la compilación de una aplicación basada en mapas es un ejercicio para el lector. Para ampliar este ejemplo, considere la posibilidad de especificar un nombre de ciudad con un radio o localizar un punto en un mapa y seleccionar un radio. Para examinar estas opciones con más detalle, consulte los siguientes recursos:

* [Documentación de Azure Maps](../azure-maps/index.yml)
* [Búsqueda de una dirección mediante el servicio de búsqueda de Azure Maps](../azure-maps/how-to-search-for-address.md)

## <a name="order-results-based-on-a-scoring-profile"></a>Ordenar los resultados según un perfil de puntuación

En los ejemplos que se proporcionan en el tutorial hasta ahora se muestra el ordenamiento según valores numéricos (clasificación, fecha de reforma, distancia geográfica), que proporciona un proceso _exacto_ de ordenamiento. Sin embargo, algunas búsquedas y datos no se prestan a una comparación tan fácil entre dos elementos de datos. Azure Cognitive Search incluye el concepto de _puntuación_. Los _perfiles de puntuación_ se pueden especificar para un conjunto de datos que se puede usar para proporcionar comparaciones más complejas y cualitativas, que deben ser especialmente útil cuando, por ejemplo, se comparan datos basado en texto para decidir cuáles deberían mostrarse primero.

Los perfiles de puntuación no los definen los usuarios, sino que normalmente lo hacen los administradores de un conjunto de datos. Se han configurado varios perfiles de puntuación en los datos de hoteles. Echemos un vistazo a cómo se define un perfil de puntuación y luego intentemos escribir código para buscar en ellos.

### <a name="how-scoring-profiles-are-defined"></a>Definición de perfiles de puntuación

Veamos tres ejemplos de perfiles de puntuación y consideremos cómo cada uno _debería_ afecta al orden de los resultados. Como desarrollador de aplicaciones, no escribe los perfiles; estos los escribe el administrador de datos. Sin embargo, resulta útil analizar la sintaxis.

1. Este es el perfil de puntuación predeterminado para el conjunto de datos de hoteles, que se usa cuando no se especifica ningún parámetro **OrderBy** o **ScoringProfile**. Este perfil aumenta la _puntuación_ para un hotel si el texto de búsqueda está presente en el nombre del hotel, la descripción o la lista de etiquetas (servicios). Observe cómo las ponderaciones de la puntuación favorecen determinados campos. Si el texto de búsqueda aparece en otro campo, que no se muestra abajo, tendrá una ponderación de 1. Obviamente, cuanto más alta sea la puntuación, más arriba en la vista aparecerá un resultado.

     ```cs
    {
            "name": "boostByField",
            "text": {
                "weights": {
                    "HotelName": 2,
                    "Description": 1.5,
                    "Description_fr": 1.5,
                    "Tags": 3
                }
            }
        }

    ```

2. El siguiente perfil de puntuación aumenta la puntuación significativamente, si un parámetro proporcionado incluye uno o varios de la lista de etiquetas (que llamaremos "servicios"). El punto clave de este perfil es que _debe_ proporcionarse un parámetro que contenga texto. Si el parámetro está vacío o no se proporciona, se lanzará un error.
 
    ```cs
            {
            "name": "boostAmenities",
            "functions": [
                {
                    "type": "tag",
                    "fieldName": "Tags",
                    "boost": 5,
                    "tag": {
                        "tagsParameter": "amenities"
                    }
                }
            ]
        }
    ```

3. En este tercer ejemplo, la clasificación proporciona un aumento significativo a la puntuación. La fecha de la última reforma también aumentará la puntuación, pero solo si esos datos se encuentran dentro de los 730 días (2 años) desde la fecha actual.

    ```cs
            {
            "name": "renovatedAndHighlyRated",
            "functions": [
                {
                    "type": "magnitude",
                    "fieldName": "Rating",
                    "boost": 20,
                    "interpolation": "linear",
                    "magnitude": {
                        "boostingRangeStart": 0,
                        "boostingRangeEnd": 5,
                        "constantBoostBeyondRange": false
                    }
                },
                {
                    "type": "freshness",
                    "fieldName": "LastRenovationDate",
                    "boost": 10,
                    "interpolation": "quadratic",
                    "freshness": {
                        "boostingDuration": "P730D"
                    }
                }
            ]
        }

    ```

    Ahora, veamos si estos perfiles funcionan según teníamos previsto.

### <a name="add-code-to-the-view-to-compare-profiles"></a>Agregar código a la vista para comparar perfiles

1. Abra el archivo index.cshtml y reemplace la sección &lt;body&gt; por el código siguiente.

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
                        Hotels Search - Order Results
                    </h1>
                </td>
            </tr>
            <tr>
                <td></td>
                <td>
                    <!-- Display the search text box, with the search icon to the right of it. -->
                    <div class="searchBoxForm">
                        @Html.TextBoxFor(m => m.searchText, new { @class = "searchBox" }) <input class="searchBoxSubmit" type="submit" value="">
                    </div>

                    <div class="searchBoxForm">
                        <b>&nbsp;Order:&nbsp;</b>
                        @Html.RadioButtonFor(m => m.scoring, "Default") Default&nbsp;&nbsp;
                        @Html.RadioButtonFor(m => m.scoring, "RatingRenovation") By numerical Rating&nbsp;&nbsp;
                        @Html.RadioButtonFor(m => m.scoring, "boostAmenities") By Amenities&nbsp;&nbsp;
                        @Html.RadioButtonFor(m => m.scoring, "renovatedAndHighlyRated") By Renovated date/Rating profile&nbsp;&nbsp;
                    </div>
                </td>
            </tr>

            <tr>
                <td valign="top">
                    <div id="facetplace" class="facetchecks">

                        @if (Model != null && Model.facetText != null)
                        {
                            <h5 class="facetheader">Amenities:</h5>
                            <ul class="facetlist">
                                @for (var c = 0; c < Model.facetText.Length; c++)
                                {
                                    <li> @Html.CheckBoxFor(m => m.facetOn[c], new { @id = "check" + c.ToString() }) @Model.facetText[c] </li>
                                }

                            </ul>
                        }
                    </div>
                </td>
                <td>
                    @if (Model != null && Model.resultList != null)
                    {
                        // Show the total result count.
                        <p class="sampleText">
                            @Html.DisplayFor(m => m.resultList.Count) Results <br />
                        </p>

                        <div id="myDiv" style="width: 800px; height: 450px; overflow-y: scroll;" onscroll="scrolled()">

                            <!-- Show the hotel data. -->
                            @for (var i = 0; i < Model.resultList.Results.Count; i++)
                            {
                                var rateText = $"Rates from ${Model.resultList.Results[i].Document.cheapest} to ${Model.resultList.Results[i].Document.expensive}";
                                var lastRenovatedText = $"Last renovated: { Model.resultList.Results[i].Document.LastRenovationDate.Value.Year}";
                                var ratingText = $"Rating: {Model.resultList.Results[i].Document.Rating}";

                                string amenities = string.Join(", ", Model.resultList.Results[i].Document.Tags);
                                string fullDescription = Model.resultList.Results[i].Document.Description;
                                fullDescription += $"\nAmenities: {amenities}";

                                // Display the hotel details.
                                @Html.TextArea($"name{i}", Model.resultList.Results[i].Document.HotelName, new { @class = "box1A" })
                                @Html.TextArea($"rating{i}", ratingText, new { @class = "box1B" })
                                @Html.TextArea($"rates{i}", rateText, new { @class = "box2A" })
                                @Html.TextArea($"renovation{i}", lastRenovatedText, new { @class = "box2B" })
                                @Html.TextArea($"desc{i}", fullDescription, new { @class = "box3" })
                            }
                        </div>

                        <script>
                            function scrolled() {
                                if (myDiv.offsetHeight + myDiv.scrollTop >= myDiv.scrollHeight) {
                                    $.getJSON("/Home/Next", function (data) {
                                        var div = document.getElementById('myDiv');

                                        // Append the returned data to the current list of hotels.
                                        for (var i = 0; i < data.length; i += 5) {
                                            div.innerHTML += '\n<textarea class="box1A">' + data[i] + '</textarea>';
                                            div.innerHTML += '<textarea class="box1B">' + data[i + 1] + '</textarea>';
                                            div.innerHTML += '\n<textarea class="box2A">' + data[i + 2] + '</textarea>';
                                            div.innerHTML += '<textarea class="box2B">' + data[i + 3] + '</textarea>';
                                            div.innerHTML += '\n<textarea class="box3">' + data[i + 4] + '</textarea>';
                                        }
                                    });
                                }
                            }
                        </script>
                    }
                </td>
            </tr>
        </table>
    }
    </body>
    ```

2. Abra el archivo SearchData.cs y reemplace la clase **SearchData** por el código siguiente.

    ```cs
    public class SearchData
    {
        public SearchData()
        {
        }

        // Constructor to initialize the list of facets sent from the controller.
        public SearchData(List<string> facets)
        {
            facetText = new string[facets.Count];

            for (int i = 0; i < facets.Count; i++)
            {
                facetText[i] = facets[i];
            }
        }

        // Array to hold the text for each amenity.
        public string[] facetText { get; set; }

        // Array to hold the setting for each amenitity.
        public bool[] facetOn { get; set; }

        // The text to search for.
        public string searchText { get; set; }

        // Record if the next page is requested.
        public string paging { get; set; }

        // The list of results.
        public DocumentSearchResult<Hotel> resultList;

        public string scoring { get; set; }       
    }
    ```

3. Abra el archivo hotels.css y agregue las siguientes clases HTML.

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

### <a name="add-code-to-the-controller-to-specify-a-scoring-profile"></a>Agregar código al controlador para especificar un perfil de puntuación

1. Abra el archivo de controlador principal. Agregue la siguiente instrucción **using** (para ayudar con la creación de listas).

    ```cs
    using System.Linq;
    ```

2.  En este ejemplo, necesitamos que la llamada inicial a **Index** haga algo más que simplemente devolver la vista inicial. El método ahora busca hasta 20 servicios que se mostrarán en la vista.

    ```cs
        public async Task<ActionResult> Index()
        {
            InitSearch();

            // Set up the facets call in the search parameters.
            SearchParameters sp = new SearchParameters()
            {
                // Search for up to 20 amenities.
                Facets = new List<string> { "Tags,count:20" },
            };

            DocumentSearchResult<Hotel> searchResult = await _indexClient.Documents.SearchAsync<Hotel>("*", sp);

            // Convert the results to a list that can be displayed in the client.
            List<string> facets = searchResult.Facets["Tags"].Select(x => x.Value.ToString()).ToList();

            // Initiate a model with a list of facets for the first view.
            SearchData model = new SearchData(facets);

            // Save the facet text for the next view.
            SaveFacets(model, false);

            // Render the view including the facets.
            return View(model);
        }
    ```

3. Necesitamos dos métodos privados para guardar las facetas en el almacenamiento temporal y para recuperarlas del almacenamiento temporal y rellenar un modelo.

    ```cs
        // Save the facet text to temporary storage, optionally saving the state of the check boxes.
        private void SaveFacets(SearchData model, bool saveChecks = false)
        {
            for (int i = 0; i < model.facetText.Length; i++)
            {
                TempData["facet" + i.ToString()] = model.facetText[i];
                if (saveChecks)
                {
                    TempData["faceton" + i.ToString()] = model.facetOn[i];
                }
            }
            TempData["facetcount"] = model.facetText.Length;
        }

        // Recover the facet text to a model, optionally recoving the state of the check boxes.
        private void RecoverFacets(SearchData model, bool recoverChecks = false)
        {
            // Create arrays of the appropriate length.
            model.facetText = new string[(int)TempData["facetcount"]];
            if (recoverChecks)
            {
                model.facetOn = new bool[(int)TempData["facetcount"]];
            }

            for (int i = 0; i < (int)TempData["facetcount"]; i++)
            {
                model.facetText[i] = TempData["facet" + i.ToString()].ToString();
                if (recoverChecks)
                {
                    model.facetOn[i] = (bool)TempData["faceton" + i.ToString()];
                }
            }
        }
    ```

4. Es necesario establecer los parámetros **OrderBy** y **ScoringProfile** según corresponda. Reemplace el método **Index(SearchData model)** existente por lo siguiente.

    ```cs
        public async Task<ActionResult> Index(SearchData model)
        {
            try
            {
                InitSearch();

                int page;

                if (model.paging != null && model.paging == "next")
                {
                    // Recover the facet text, and the facet check box settings.
                    RecoverFacets(model, true);

                    // Increment the page.
                    page = (int)TempData["page"] + 1;

                    // Recover the search text.
                    model.searchText = TempData["searchfor"].ToString();
                }
                else
                {
                    // First search with text. 
                    // Recover the facet text, but ignore the check box settings, and use the current model settings.
                    RecoverFacets(model,false);

                    // First call. Check for valid text input, and valid scoring profile.
                    if (model.searchText == null)
                    {
                        model.searchText = "";
                    }
                    if (model.scoring == null)
                    {
                        model.scoring = "Default";
                    }
                    page = 0;
                }

                // Set empty defaults for ordering and scoring parameters.
                var orderby = new List<string>();
                string profile = "";
                var scoringParams = new List<ScoringParameter>();

                // Set the ordering based on the user's radio button selection.
                switch (model.scoring)
                {
                    case "RatingRenovation":
                        orderby.Add("Rating desc");
                        orderby.Add("LastRenovationDate desc");
                        break;

                    case "boostAmenities":
                        {
                            profile = model.scoring;
                            var setAmenities = new List<string>();

                            // Create a string list of amenities that have been clicked.
                            for (int a = 0; a < model.facetOn.Length; a++)
                            {
                                if (model.facetOn[a])
                                {
                                    setAmenities.Add(model.facetText[a]);
                                }
                            }
                            if (setAmenities.Count > 0)
                            {
                                // Only set scoring parameters if there are any.
                                var sp = new ScoringParameter("amenities", setAmenities);
                                scoringParams.Add(sp);
                            }
                            else
                            {
                                // No amenities selected, so set profile back to default.
                                profile = "";
                            }
                        }
                        break;

                    case "renovatedAndHighlyRated":
                        profile = model.scoring;
                        break;

                    default:
                        break;
                }

                // Setup the search parameters.
                var parameters = new SearchParameters
                {
                    // Set the ordering/scoring parameters.
                    OrderBy = orderby,
                    ScoringProfile = profile,
                    ScoringParameters = scoringParams,

                    // Select the data properties to be returned.
                    Select = new[] { "HotelName", "Description", "Tags", "Rooms", "Rating", "LastRenovationDate" },
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
                TempData["scoring"] = model.scoring;
                SaveFacets(model,true);

                // Calculate the room rate ranges.
                for (int n = 0; n < model.resultList.Results.Count; n++)
                {
                    var cheapest = 0d;
                    var expensive = 0d;

                    for (var r = 0; r < model.resultList.Results[n].Document.Rooms.Length; r++)
                    {
                        var rate = model.resultList.Results[n].Document.Rooms[r].BaseRate;
                        if (rate < cheapest || cheapest == 0)
                        {
                            cheapest = (double)rate;
                        }
                        if (rate > expensive)
                        {
                            expensive = (double)rate;
                        }
                    }
                    model.resultList.Results[n].Document.cheapest = cheapest;
                    model.resultList.Results[n].Document.expensive = expensive;
                }
            }
            catch
            {
                return View("Error", new ErrorViewModel { RequestId = "1" });
            }
            return View("Index", model);
        }
    ```

    Lea los comentarios para cada una de las selecciones **switch**.

5. Si completó el código adicional para la sección anterior sobre el ordenamiento basado en varias propiedades, no es necesario hacer cambios en la acción **Next**.

### <a name="run-and-test-the-app"></a>Ejecutar y probar la aplicación

1. Ejecute la aplicación. Debería ver un conjunto completo de servicios en la vista.

2. Para el ordenamiento, si selecciona "By numerical Rating", obtendrá el ordenamiento que ya se implementó en este tutorial, con la fecha de reforma que decide entre los hoteles de clasificación igual.

![Ordenar "beach" según la clasificación](./media/tutorial-csharp-create-first-app/azure-search-orders-beach.png)

3. Ahora pruebe el perfil "By amenities". Haga varias selecciones de servicios y compruebe que en la lista de resultados se ascienden los hoteles con dichos servicios.

![Ordenar "beach" según el perfil](./media/tutorial-csharp-create-first-app/azure-search-orders-beach-profile.png)

4. Pruebe el perfil "By Renovated date/Rating" para ver si obtiene los resultados esperados. Solo los hoteles reformados recientemente deberían obtener un aumento tipo _freshness_.

### <a name="resources"></a>Recursos

Para más información, consulte el siguiente tema [Incorporación de perfiles de puntuación a un índice de Azure Cognitive Search](/azure/search/index-add-scoring-profiles).

## <a name="takeaways"></a>Puntos clave

Tenga en cuenta las siguientes conclusiones de este proyecto:

* Los usuarios esperarán que los resultados de búsqueda estén ordenados, con los más relevantes en primer lugar.
* Los datos deben estar estructurados para facilitar el ordenamiento. No pudimos ordenar fácilmente primero por "más económico", ya que los datos no están estructurados para habilitar el ordenamiento sin código adicional.
* Puede haber varios niveles en ordenamiento para diferenciar entre los resultados que tienen el mismo valor en un nivel superior de ordenamiento.
* Es natural que algunos resultados se ordenen en orden ascendente (por ejemplo, la distancia desde un punto determinado) y algunos en orden descendente (por ejemplo, la clasificación de huésped).
* Los perfiles de puntuación se pueden definir cuando no haya comparaciones numéricas disponibles, o estas no sean suficientemente inteligentes, para un conjunto de datos. La puntuación de cada resultado ayudará a ordenar y mostrar los resultados de forma inteligente.

## <a name="next-steps"></a>Pasos siguientes

Ha completado esta serie de tutoriales de C#, y ahora dispone de conocimientos valiosos sobre las API de Azure Cognitive Search.

Si necesita otras referencias y tutoriales, puede explorar [Microsoft Learn](/learn/browse/?products=azure) o los demás tutoriales en la [documentación de Azure Cognitive Search](./index.yml).