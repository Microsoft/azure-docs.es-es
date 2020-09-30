---
title: 'Tutorial: Migración de servicios web desde Google Maps | Microsoft Azure Maps'
description: Migración de servicios web de Google Maps a Microsoft Azure Maps.
author: rbrundritt
ms.author: richbrun
ms.date: 12/17/2019
ms.topic: tutorial
ms.service: azure-maps
services: azure-maps
manager: cpendle
ms.custom: ''
ms.openlocfilehash: 8cc16e611002748dad2716a1c8dc914f297da9f1
ms.sourcegitcommit: 07166a1ff8bd23f5e1c49d4fd12badbca5ebd19c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/15/2020
ms.locfileid: "90090557"
---
# <a name="migrate-web-service-from-google-maps"></a>Migración de un servicio web desde Google Maps

Tanto Azure como Google Maps proporcionan acceso a API espaciales a través de servicios web REST. Las interfaces API de estas plataformas realizan funcionalidades similares. Sin embargo, cada una usa diferentes convenciones de nomenclatura y objetos de respuesta.

En la tabla se muestran las API de servicio de Azure Maps, que tienen una funcionalidad similar a las API de servicio de Google Maps enumeradas.

| API de servicio de Google Maps | API de servicio de Azure Maps                                                                      |
|-------------------------|---------------------------------------------------------------------------------------------|
| Direcciones              | [Route](https://docs.microsoft.com/rest/api/maps/route)                                     |
| Matriz de distancia         | [Matriz de ruta](https://docs.microsoft.com/rest/api/maps/route/postroutematrixpreview)       |
| Codificación geográfica               | [Búsqueda](https://docs.microsoft.com/rest/api/maps/search)                                   |
| Búsqueda de lugares           | [Búsqueda](https://docs.microsoft.com/rest/api/maps/search)                                   |
| Autocompletar para lugares      | [Búsqueda](https://docs.microsoft.com/rest/api/maps/search)                                   |
| Ajustar a la carretera            | Consulte la sección [Cálculo de rutas y direcciones](#calculate-routes-and-directions).            |
| Límites de velocidad            | Consulte la sección [Geocodificación inversa de una coordenada](#reverse-geocode-a-coordinate).                  |
| Mapa estático              | [Render](https://docs.microsoft.com/rest/api/maps/render/getmapimage)                       |
| Zona horaria               | [Zona horaria](https://docs.microsoft.com/rest/api/maps/timezone)                              |

Las siguientes API de servicio no están disponibles actualmente en Azure Maps:

- Elevation
- Geolocalización
- Detalles y fotos de lugares: los números de teléfono y la dirección URL del sitio web están disponibles en la API de búsqueda de Azure Maps.
- Direcciones URL de mapas
- Carreteras más cercanas: esto se puede conseguir mediante el SDK web, tal como se muestra [aquí](https://azuremapscodesamples.azurewebsites.net/index.html?sample=Basic%20snap%20to%20road%20logic
), pero no está disponible como servicio actualmente.
- Vista de calle estática

Azure Maps tiene varios servicios web REST extra que pueden ser de su interés:

- [Operaciones espaciales](https://docs.microsoft.com/rest/api/maps/spatial): descargue operaciones y cálculos espaciales complejos (como geovallas) en un servicio.
- [Tráfico](https://docs.microsoft.com/rest/api/maps/traffic): acceda a datos de incidencias y de flujo circulatorio en tiempo real.

## <a name="geocoding-addresses"></a>Geocodificación de direcciones

La geocodificación es el proceso de convertir una dirección en una coordenada. Por ejemplo, "1 Microsoft Way, Redmond, WA" se convierte en la longitud:-122.1298, latitud: 47.64005. Las coordenadas se pueden usar después con diferentes propósitos, como posicionar un marcador de centrado en un mapa.

Azure Maps proporciona varios métodos de geocodificación de direcciones:

- [**Geocodificación de direcciones de forma libre**](https://docs.microsoft.com/rest/api/maps/search/getsearchaddress): especifique una única cadena de dirección y procese la solicitud inmediatamente. "1 Microsoft Way, Redmond, WA" es un ejemplo de una única cadena de dirección. Esta es la API recomendada si hay que geocodificar direcciones individuales rápidamente.
- [**Geocodificación de direcciones estructurada**](https://docs.microsoft.com/rest/api/maps/search/getsearchaddressstructured): Especifique las partes de una dirección individual (por ejemplo, el nombre de la calle, la ciudad, el país o la región y el código postal) y procese la solicitud inmediatamente. Esta es la API recomendada si hay que geocodificar direcciones individuales rápidamente y los datos ya se han analizado en partes individuales.
- [**Geocodificación de direcciones por lotes**](https://docs.microsoft.com/rest/api/maps/search/postsearchaddressbatchpreview): cree una solicitud que contenga hasta 10 000 direcciones y procéselas durante un período de tiempo determinado. Todas las direcciones se geocodificarán en paralelo en el servidor y, cuando la operación se complete, puede descargar el conjunto de resultados completo. Este es el método recomendable para geocodificar grandes conjuntos de datos.
- [**Búsqueda aproximada**](https://docs.microsoft.com/rest/api/maps/search/getsearchfuzzy): esta API combina la geocodificación de direcciones con la búsqueda de puntos de interés. Esta API toma una cadena de forma libre. Esta cadena puede ser un dirección, un lugar, un punto de referencia, un punto de interés o una categoría de puntos de interés. Esta API procesa la solicitud casi en tiempo real. Se recomienda usar esta API en aplicaciones en las que los usuarios buscan direcciones o puntos de interés desde el mismo cuadro de texto.
- [**Búsqueda parcial por lotes**](https://docs.microsoft.com/rest/api/maps/search/postsearchfuzzybatchpreview): cree una solicitud que contenga hasta 10 000 direcciones, lugares, puntos de referencia o puntos de interés y procéselos durante un período de tiempo determinado. Todos los datos se procesarán en paralelo en el servidor y, cuando la operación se complete, puede descargar el conjunto de resultados completo.

En la siguiente tabla se establecen referencias cruzadas entre los parámetros de API de Google Maps y los parámetros de API similares correspondientes en Azure Maps.

| Parámetro de API de Google Maps | Parámetro de API de Azure Maps similar  |
|---------------------------|--------------------------------------|
| `address`                   | `query`                            |
| `bounds`                    | `topLeft` y `btmRight`           |
| `components`                | `streetNumber`<br/>`streetName`<br/>`crossStreet`<br/>`postalCode`<br/>`municipality` (ciudad/municipio)<br/>`municipalitySubdivision` (barrio, ciudades más grandes o más pequeñas)<br/>`countrySubdivision` (estado o provincia)<br/>`countrySecondarySubdivision` (condado)<br/>`countryTertiarySubdivision` (distrito)<br/>`countryCode` (código de dos letras de país o región) |
| `key`                       | `subscription-key` (vea también la documentación de [autenticación con Azure Maps](azure-maps-authentication.md)) |
| `language`                  | `language` (vea la documentación de [compatibilidad de idiomas](supported-languages.md))  |
| `region`                    | `countrySet`                       |

[Aquí](how-to-search-for-address.md) se documenta un ejemplo sobre cómo usar el servicio de búsqueda. Asegúrese de revisar los [procedimientos recomendados de búsqueda](how-to-use-best-practices-for-search.md).

> [!TIP]
> Las API de geocodificación de direcciones de forma libre y de búsqueda aproximada se pueden usar en el modo Autocompletar si se agrega `&typeahead=true` a la dirección URL de la solicitud. Esto indicará al servidor que el texto de entrada probablemente sea parcial y la búsqueda cambiará al modo predictivo.

## <a name="reverse-geocode-a-coordinate"></a>Geocodificación inversa de una coordenada

La geocodificación inversa es el proceso por el cual unas coordenadas geográficas se convierten en una dirección aproximada. Coordenadas con "longitud:-122.1298, latitud: 47.64005" se convierten en "1 Microsoft Way, Redmond, WA".

Azure Maps proporciona diversos métodos de geocodificación inversa:

- [**Geocodificador inverso de direcciones**](https://docs.microsoft.com/rest/api/maps/search/getsearchaddressreverse): especifique una única coordenada geográfica para obtener la dirección aproximada correspondiente. Procesa la solicitud casi en tiempo real.
- [**Geocodificador inverso de cruces de direcciones**](https://docs.microsoft.com/rest/api/maps/search/getsearchaddressreversecrossstreet): especifique una única coordenada geográfica para obtener información sobre un cruce de direcciones y procese la solicitud inmediatamente. Por ejemplo, puede recibir el cruce 1st Avenue y Main St.
- [**Geocodificador inverso de direcciones por lotes**](https://docs.microsoft.com/rest/api/maps/search/postsearchaddressreversebatchpreview): cree una solicitud que contenga hasta 10 000 coordenadas y procéselas durante un período de tiempo determinado. Todos los datos se procesarán en paralelo en el servidor. Cuando se complete la solicitud, podrá descargar el conjunto completo de resultados.

En esta tabla se establecen referencias cruzadas entre los parámetros de API de Google Maps y los parámetros de API similares correspondientes de Azure Maps.

| Parámetro de API de Google Maps   | Parámetro de API de Azure Maps similar   |
|-----------------------------|---------------------------------------|
| `key`                       | `subscription-key` (vea también la documentación de [autenticación con Azure Maps](azure-maps-authentication.md)) |
| `language`                  | `language` (vea la documentación de [compatibilidad de idiomas](supported-languages.md))  |
| `latlng`                    | `query`  |
| `location_type`             | *N/D*     |
| `result_type`               | `entityType`    |

Consulte [Procedimientos recomendados de búsqueda](how-to-use-best-practices-for-search.md).

La API de geocodificación inversa de Azure Maps tiene algunas características adicionales que no están disponibles en Google Maps. Estas características pueden ser útiles para realizar la integración con su aplicación cuando la migre:

- Obtención de datos de límite de velocidad.
- Obtención de información de uso de carreteras: comarcales, arterias, accesos limitados, rampas, etc.
- Obtención del lado de la calle en el que se encuentra una coordenada.

## <a name="search-for-points-of-interest"></a>Búsqueda de puntos de interés

Para realizar búsquedas de datos de puntos de interés en Google Maps se usa la API de búsqueda de lugares. Esta API ofrece tres formas diferentes de buscar puntos de interés:

- **Buscar lugares a partir de texto:** se busca un punto de interés en función de su nombre, dirección o número de teléfono.
- **Búsqueda de elementos cercanos:** se buscan los puntos de interés que están a una determinada distancia de una ubicación.
- **Búsqueda de texto:** se buscan lugares mediante un texto de forma libre que incluye información sobre el punto de interés y la ubicación (por ejemplo, "pizza en Nueva York" o "restaurantes cerca de la calle Central").

Azure Maps proporciona varias API de búsqueda de puntos de interés:

- [**Búsqueda de puntos de interés**](https://docs.microsoft.com/rest/api/maps/search/getsearchpoi): se buscan puntos de interés por su nombre (por ejemplo, "Starbucks").
- [**Búsqueda de categorías de puntos de interés**](https://docs.microsoft.com/rest/api/maps/search/getsearchpoicategory): se buscan puntos de interés por su categoría (por ejemplo, "restaurante").
- [**Búsqueda de elementos cercanos**](https://docs.microsoft.com/rest/api/maps/search/getsearchnearby): se buscan los puntos de interés que están a una determinada distancia de una ubicación.
- [**Búsqueda aproximada**](https://docs.microsoft.com/rest/api/maps/search/getsearchfuzzy): esta API combina la geocodificación de direcciones con la búsqueda de puntos de interés. La API toma una cadena de forma libre, que puede ser una dirección, un lugar, un punto de referencia, un punto de interés o una categoría de puntos de interés. Procesa la solicitud casi en tiempo real. Se recomienda usar esta API en aplicaciones en las que los usuarios buscan direcciones o puntos de interés desde el mismo cuadro de texto.
- [**Búsqueda en una geometría**](https://docs.microsoft.com/rest/api/maps/search/postsearchinsidegeometry): se buscan los puntos de interés que se encuentran dentro de una geometría específica. Por ejemplo, busque un punto de interés dentro de un polígono.
- [**Búsqueda a lo largo de una ruta**](https://docs.microsoft.com/rest/api/maps/search/postsearchalongroute): se buscan los puntos de interés que hay a lo largo del trayecto de una ruta específica.
- [**Búsqueda parcial por lotes**](https://docs.microsoft.com/rest/api/maps/search/postsearchfuzzybatchpreview): cree una solicitud que contenga hasta 10 000 direcciones, lugares, puntos de referencia o puntos de interés. Procese la solicitud durante un período de tiempo. Todos los datos se procesarán en paralelo en el servidor. Cuando la solicitud termine de procesarse, podrá descargar el conjunto completo de resultados.

Actualmente, Azure Maps no tiene una API comparable a la API de búsqueda de texto de Google Maps.

> [!TIP]
> Para usar las API de búsqueda aproximada, de búsqueda de puntos de interés y de búsqueda de categorías de puntos de interés en el modo Autocompletar, agregue `&typeahead=true` a la dirección URL de la solicitud. Esto indicará al servidor que el texto de entrada probablemente es parcial. La API realizará la búsqueda en modo predictivo.

Consulte la documentación sobre [procedimientos recomendados de búsqueda](how-to-use-best-practices-for-search.md).

### <a name="find-place-from-text"></a>Búsqueda de lugares a partir de texto

Para hallar puntos de interés por su nombre o dirección, use las API de Azure Maps de [búsqueda de puntos de interés](https://docs.microsoft.com/rest/api/maps/search/getsearchpoi) y [búsqueda aproximada](https://docs.microsoft.com/rest/api/maps/search/getsearchfuzzy).

En la tabla se establecen referencias cruzadas entre los parámetros de API de Google Maps y los parámetros de API similares correspondientes de Azure Maps.

| Parámetro de API de Google Maps | Parámetro de API de Azure Maps similar |
|---------------------------|-------------------------------------|
| `fields`                  | *N/D*                               |
| `input`                   | `query`                             |
| `inputtype`               | *N/D*                               |
| `key`                     | `subscription-key` (vea también la documentación de [autenticación con Azure Maps](azure-maps-authentication.md)) |
| `language`                | `language` (vea la documentación de [compatibilidad de idiomas](supported-languages.md))  |
| `locationbias`            | `lat`, `lon` y `radius`<br/>`topLeft` y `btmRight`<br/>`countrySet`  |

### <a name="nearby-search"></a>Búsqueda de elementos cercanos

Use la API de [búsqueda de elementos cercanos](https://docs.microsoft.com/rest/api/maps/search/getsearchnearby) para recuperar los puntos de interés cercanos en Azure Maps.

En esta tabla se muestran los parámetros de API de Google Maps y los parámetros de API similares correspondientes de Azure Maps.

| Parámetro de API de Google Maps | Parámetro de API de Azure Maps similar  |
|---------------------------|--------------------------------------|
| `key`                       | `subscription-key` (vea también la documentación de [autenticación con Azure Maps](azure-maps-authentication.md)) |
| `keyword`                   | `categorySet` y `brandSet`        |
| `language`                  | `language` (vea la documentación de [compatibilidad de idiomas](supported-languages.md))  |
| `location`                  | `lat` y `lon`                     |
| `maxprice`                  | *N/D*                               |
| `minprice`                  | *N/D*                               |
| `name`                      | `categorySet` y `brandSet`        |
| `opennow`                   | *N/D*                               |
| `pagetoken`                 | `ofs` y `limit`                   |
| `radius`                    | `radius`                            |
| `rankby`                    | *N/D*                               |
| `type`                      | `categorySet –` Vea la documentación de [categorías de búsqueda admitidas](supported-search-categories.md).   |

## <a name="calculate-routes-and-directions"></a>Cálculo de rutas y direcciones

Calcular rutas y direcciones con Azure Maps. Azure Maps tiene muchas funcionalidades similares a las del servicio de enrutamiento de Google Maps, como las siguientes:

- Horas de salida y de llegada.
- Rutas de tráfico predictivas y en tiempo real.
- Diferentes modos de transporte. Por ejemplo, vehículo, a pie o en bicicleta.

> [!NOTE]
> Azure Maps requiere que todos los puntos de referencia sean coordenadas Las direcciones deben estar geocodificadas primero.

El servicio de enrutamiento de Azure Maps proporciona las siguientes API para calcular rutas:

- [**Calcular ruta**](https://docs.microsoft.com/rest/api/maps/route/getroutedirections): calcula una ruta y la solicitud se procesa inmediatamente. Esta API admite solicitudes tanto GET como POST. Las solicitudes POST son recomendables cuando se especifica un gran número de puntos de referencia o se utilizan muchas opciones de ruta para garantizar que la solicitud de dirección URL no sea demasiado larga y provoque problemas. La dirección de la ruta POST de Azure Maps tiene una opción que puede tomar miles de [puntos de apoyo](https://docs.microsoft.com/rest/api/maps/route/postroutedirections#supportingpoints) y los usa para volver a crear una ruta lógica entre ellos (ajustar a la carretera). 
- [**Ruta por lotes**](https://docs.microsoft.com/rest/api/maps/route/postroutedirectionsbatchpreview): cree una solicitud que contenga hasta 1000 solicitudes de ruta y procéselas durante un período de tiempo determinado. Todos los datos se procesarán en paralelo en el servidor y, cuando la operación se complete, puede descargar el conjunto de resultados completo.
- [**Servicios de movilidad**](https://docs.microsoft.com/rest/api/maps/mobility): calcula rutas y direcciones según el transporte público.

En la tabla se establecen referencias cruzadas entre los parámetros de API de Google Maps y los parámetros de API similares correspondientes de Azure Maps.

| Parámetro de API de Google Maps    | Parámetro de API de Azure Maps similar  |
|------------------------------|--------------------------------------|
| `alternatives`                 | `maxAlternatives`                  |
| `arrival_time`                | `arriveAt`                          |
| `avoid`                        | `avoid`                            |
| `departure_time`              | `departAt`                          |
| `destination`                  | `query` (se coordina con el formato `"lat0,lon0:lat1,lon1…."`)  |
| `key`                          | `subscription-key` (vea también la documentación de [autenticación con Azure Maps](azure-maps-authentication.md)) |
| `language`                     | `language` (vea la documentación de [compatibilidad de idiomas](supported-languages.md))   |
| `mode`                         | `travelMode`                       |
| `optimize`                     | `computeBestOrder`                 |
| `origin`                       | `query`                            |
| `region`                       | *No procede*: esta característica está relacionada con la geocodificación. Use el parámetro *countrySet* al usar la API de geocodificación de Azure Maps)  |
| `traffic_model`               | *No procede* (solo puede especificar si los datos de tráfico deben usarse con el parámetro *traffic*) |
| `transit_mode`                | Vea la documentación de los [servicios de movilidad](https://docs.microsoft.com/rest/api/maps/mobility). |
| `transit_routing_preference` | Vea la documentación de los [servicios de movilidad](https://docs.microsoft.com/rest/api/maps/mobility). |
| `units`                        | *No procede* (Azure Maps usa exclusivamente el sistema métrico)  |
| `waypoints`                    | `query`                            |

> [!TIP]
> De forma predeterminada, la API de rutas de Azure Maps solo devuelve un resumen. Devuelve la distancia, el tiempo y las coordenadas de la trayectoria de la ruta. Use el parámetro `instructionsType` para obtener instrucciones paso a paso. Use también el parámetro `routeRepresentation` para filtrar el resumen y la trayectoria de la ruta.

La API de rutas de Azure Maps tiene características adicionales que no están disponibles en Google Maps. Al migrar la aplicación, considere la posibilidad de usar estas características, que podrían ser de utilidad.

- Compatibilidad con el tipo de ruta: la más corta, la más rápida y la de mayor ahorro de combustible.
- Compatibilidad con más modos de desplazamiento: autobús, motocicleta, taxi, camión y furgoneta.
- Compatibilidad con 150 puntos de referencia.
- Cálculo de varias duraciones de recorrido en una misma solicitud; tráfico histórico, tráfico en directo, sin tráfico.
- Posibilidad de descartar otros tipos de carreteras: transporte compartido, carreteras sin asfaltar, caminos ya usados.
- Especificación de las áreas personalizadas que deben evitarse.
- Limite la elevación a la que puede ascender la ruta.
- Ruta basada en especificaciones del motor. Cálculo de rutas para vehículos eléctricos o de combustión en función de la carga o el combustible que queda y de las especificaciones del motor.
- Admite parámetros de ruta para vehículos comerciales. Por ejemplo, dimensiones del vehículo, peso, número de ejes y tipo de carga.
- Especificación de la velocidad máxima del vehículo.

Además, el servicio de rutas de Azure Maps permite [calcular intervalos de rutas posibles](https://docs.microsoft.com/rest/api/maps/route/getrouterange). El cálculo de intervalos de rutas posibles también se conoce como isocronas. Para ello, se genera un polígono que cubre un área por la que se puede viajar en cualquier dirección desde un punto de origen. Todo ello en un período de tiempo o con una cantidad de combustible o de carga determinados.

Consulte la documentación sobre [procedimientos recomendados de cálculo de ruta](how-to-use-best-practices-for-routing.md).

## <a name="retrieve-a-map-image"></a>Recuperación de una imagen de mapa

Azure Maps proporciona una API para representar imágenes de mapa estático con datos superpuestos. La API de [representación de imágenes de mapa](https://docs.microsoft.com/rest/api/maps/render/getmapimagerytile) de Azure Maps es similar a la API de mapa estático de Google Maps.

> [!NOTE]
> Azure Maps requiere que el punto central, todos los marcadores y las ubicaciones de la ruta sean coordenadas con el formato "longitud,latitud" (Google Maps usa el formato "latitud,longitud") y que las direcciones estén geocodificadas.

En la tabla se establecen referencias cruzadas entre los parámetros de API de Google Maps y los parámetros de API similares correspondientes de Azure Maps.

| Parámetro de API de Google Maps | Parámetro de API de Azure Maps similar  |
|---------------------------|--------------------------------------|
| `center`                    | `center`                           |
| `format`                    | `format` (se especifica como parte de la ruta de dirección URL). Actualmente, solo se admite el formato PNG. |
| `key`                       | `subscription-key` (vea también la documentación de [autenticación con Azure Maps](azure-maps-authentication.md)) |
| `language`                  | `language` (vea la documentación de [compatibilidad de idiomas](supported-languages.md))  |
| `maptype`                   | `layer` y `style` (vea la documentación de [estilos de mapa compatibles](supported-map-styles.md)) |
| `markers`                   | `pins`                             |
| `path`                      | `path`                             |
| `region`                    | *No procede* (se trata de una característica relacionada con la geocodificación. Use el parámetro `countrySet` al usar la API de geocodificación de Azure Maps)  |
| `scale`                     | *N/D*                              |
| `size`                      | `width` y `height` (puede tener un tamaño máximo de 8192 x 8192) |
| `style`                     | *N/D*                              |
| `visible`                   | *N/D*                              |
| `zoom`                      | `zoom`                             |

> [!NOTE]
> En el sistema de mosaicos de Azure Maps, los mosaicos son el doble de grandes que los mosaicos de mapa que se usan en Google Maps. con lo cual el valor de nivel de zoom en Azure Maps será un nivel de zoom más cercano en Azure Maps que en Google Maps. Para compensar esta diferencia, disminuya el nivel de zoom en las solicitudes que va a migrar.

Para más información, vea la [Guía de procedimientos de la API de representación de imágenes de mapa](how-to-render-custom-data.md).

Además de poder generar una imagen de mapa estático, el servicio de representación de Azure Maps también ofrece la posibilidad de acceder directamente a los mosaicos de mapa en formato de trama (PNG) y vectorial:

- [**Mosaico de mapa**](https://docs.microsoft.com/rest/api/maps/render/getmaptile): obtenga mosaicos de trama (PNG) y vectoriales de los mapas base (carreteras, fronteras, entorno).
- [**Mosaico de imagen de mapa**](https://docs.microsoft.com/rest/api/maps/render/getmapimagerytile): se obtienen mosaicos de imágenes aéreas y por satélite.

> [!TIP]
> Hace unos años, muchas aplicaciones de Google Maps pasaron de experiencias de mapa interactivo a imágenes de mapa estático. Era un método para ahorrar costos. En Azure Maps, normalmente es más rentable usar el control de mapa interactivo del SDK web. El control de mapa interactivo cobra en función del número de cargas de mosaicos. Los mosaicos de mapa en Azure Maps son grandes. Normalmente, solo se necesitan algunos mosaicos para volver a crear la misma vista que un mapa estático. El explorador almacena automáticamente los mosaicos de mapa. Por lo tanto, el control de mapa interactivo suele generar solo una fracción de una transacción al reproducir una vista de mapa estático. Las operaciones de panorámica y zoom cargarán más mosaicos; sin embargo, existen opciones en el control de mapa para deshabilitar este comportamiento. El control de mapa interactivo también proporciona muchas más opciones de visualización que los servicios de mapa estático.

### <a name="marker-url-parameter-format-comparison"></a>Comparación de formatos de parámetro de dirección URL de marcador

**Antes: Google Maps**

Agregue marcadores mediante el parámetro `markers` en la dirección URL. El parámetro `markers` toma un estilo y una lista de ubicaciones que se van a representar en el mapa con ese estilo, como se indica aquí:

```
&markers=markerStyles|markerLocation1|markerLocation2|...
```

Para agregar estilos adicionales, use los parámetros `markers` en la dirección URL con diferentes estilos y conjuntos de ubicaciones.

Especifique las ubicaciones de los marcadores con el formato "latitud,longitud".

Agregue estilos de marcador con el formato `optionName:value`, separando los estilos con caracteres de barra vertical (\|), de esta manera: "optionName1:value1\|optionName2:value2". Fíjese en que los nombres de opción y los valores se separan con dos puntos (:). Use los siguientes nombres de opciones de estilo para aplicar un estilo a los marcadores de Google Maps:

- `color`: color del icono de marcador predeterminado. Puede ser un color hexadecimal de 24 bits (`0xrrggbb`) o uno de los valores siguientes: `black`, `brown`, `green`, `purple`, `yellow`, `blue`, `gray`, `orange`, `red``white`.
- `label`: un solo carácter alfanumérico en mayúsculas que se va a mostrar en la parte superior del icono.
- `size`: tamaño del marcador. Puede ser `tiny`, `mid` o `small`.

Use los siguientes nombres de opciones de estilo para personalizar los iconos en Google Maps:

- `anchor`: especifica la alineación de la imagen de icono con la coordenada. Puede ser un valor de píxel (x, y) o uno de los siguientes valores: `top`, `bottom`, `left`, `right`, `center`, `topleft`, `topright`, `bottomleft`, `bottomright`.
- `icon`: dirección URL que apunta a la imagen de icono.

Por ejemplo, vamos a agregar un marcador rojo de tamaño mediano al mapa en la longitud:-110, latitud: 45:

```
&markers=color:red|size:mid|45,-110
```

<center>

![Marcador de Google Maps](media/migrate-google-maps-web-services/google-maps-marker.png)</center>

**Después: Azure Maps**

Para agregar marcadores a una imagen de mapa estático, especifique el parámetro `pins` en la dirección URL. Al igual que en Google Maps, especifique un estilo y una lista de ubicaciones en el parámetro. El parámetro `pins` se puede especificar varias veces para aplicar distintos estilos a los marcadores.

```
&pins=iconType|pinStyles||pinLocation1|pinLocation2|...
```

Para usar estilos adicionales, agregue parámetros `pins` en la dirección URL con diferentes estilos y conjuntos de ubicaciones.

En Azure Maps, la ubicación de la chincheta debe tener el formato "longitud latitud". Google Maps usa el formato "latitud,longitud". En el formato de Azure Maps, la longitud y la latitud se separan con un espacio, no con una coma.

`iconType` especifica el tipo de chincheta que se va a crear. Puede presentar los siguientes valores:

- `default`: icono de anclaje predeterminado.
- `none`: no se muestra ningún icono, solo se representarán etiquetas.
- `custom`: especifica que se va a usar un icono personalizado. Se puede agregar una dirección URL que apunte a la imagen del icono al final del parámetro `pins`, después de la información de ubicación del anclaje.
- `{udid}`: identificador de datos único (UDID) de un icono almacenado en la plataforma de almacenamiento de datos de Azure Maps.

Agregue estilos de chincheta con el formato `optionNameValue`. Separe los distintos estilos con caracteres de barra vertical (\|). Por ejemplo: `iconType|optionName1Value1|optionName2Value2`. Los nombres y valores de las opciones no están separados. Use los siguientes nombres de opciones de estilo para aplicar estilo a los marcadores:

- `al`: especifica la opacidad (valor alfanumérico) del marcador. Elija un número entre 0 y 1.
- `an`: especifica el delimitador del anclaje. Especifique los valores de los píxeles X e Y con el formato "x y".
- `co`: color del anclaje. Especifique un color hexadecimal de 24 bits: de `000000` a `FFFFFF`.
- `la`: especifica el delimitador de la etiqueta. Especifique los valores de los píxeles X e Y con el formato "x y".
- `lc`: color de la etiqueta. Especifique un color hexadecimal de 24 bits: de `000000` a `FFFFFF`.
- `ls`: tamaño de la etiqueta en píxeles. Elija un número mayor que 0.
- `ro`: valor en grados para girar el icono. Elija un número entre -360 y 360.
- `sc`: valor de escala del icono de anclaje. Elija un número mayor que 0.

Especifique los valores de etiqueta para la ubicación de cada chincheta. Este enfoque es más eficaz que aplicar un único valor de etiqueta a todos los marcadores de la lista de ubicaciones. El valor de etiqueta puede ser una cadena de varios caracteres. Escriba la cadena entre comillas simples para que no se confunda con un valor de estilo o de ubicación.

Vamos a agregar un icono predeterminado rojo (`FF0000`), con la etiqueta "Space Needle" justo debajo (15 50). El icono está en la longitud:-122.349300, latitud: 47.620180:

```
&pins=default|coFF0000|la15 50||'Space Needle' -122.349300 47.620180
```

<center>

![Marcador de Azure Maps](media/migrate-google-maps-web-services/azure-maps-marker.png)</center>

Agregue tres chinchetas con los valores de etiqueta '1', '2' y '3':

```
&pins=default||'1'-122 45|'2'-119.5 43.2|'3'-121.67 47.12
```

<center>

![Varios marcadores en Azure Maps](media/migrate-google-maps-web-services/azure-maps-multiple-markers.png)</center>

### <a name="path-url-parameter-format-comparison"></a>Comparación de formatos de parámetro de dirección URL de ruta

**Antes: Google Maps**

Agregue líneas y polígonos a una imagen de mapa estático con el parámetro `path` en la dirección URL. El parámetro `path` toma un estilo y la lista de las ubicaciones que se van a representar en el mapa, como se muestra a continuación:

```
&path=pathStyles|pathLocation1|pathLocation2|...
```

Para usar estilos adicionales, agregue parámetros `path` en la dirección URL con diferentes estilos y conjuntos de ubicaciones.

Las ubicaciones de la ruta se especifican con el formato `latitude1,longitude1|latitude2,longitude2|…`. Las rutas se pueden codificar o contener direcciones de puntos.

Agregue estilos de ruta con el formato `optionName:value`; sepárelos con caracteres de barra vertical (\|). Separe los nombres y valores de las opciones con dos puntos (:). Por ejemplo: `optionName1:value1|optionName2:value2`. Se pueden usar los siguientes nombres de opción de estilo para aplicar estilo a las rutas de Google Maps:

- `color`: color de la ruta o del contorno del polígono. Puede ser un color hexadecimal de 24 bits (`0xrrggbb`), un color hexadecimal de 32 bits (`0xrrggbbbaa`) o uno de los valores siguientes: black, brown, green, purple, yellow, blue, gray, orange, red, white.
- `fillColor`: color con el que se va a rellenar el área de la ruta (polígono). Puede ser un color hexadecimal de 24 bits (`0xrrggbb`), un color hexadecimal de 32 bits (`0xrrggbbbaa`) o uno de los valores siguientes: black, brown, green, purple, yellow, blue, gray, orange, red, white.
- `geodesic`: indica si la ruta debe ser una línea que siga la curvatura de la tierra.
- `weight`: grosor de la línea de ruta en píxeles.

Agregue una opacidad de línea roja y un grosor de píxel al mapa entre las coordenadas, en el parámetro URL. En el ejemplo siguiente, la línea tiene una opacidad del 50 % y un grosor de cuatro píxeles. Las coordenadas son longitud:-110, latitud: 45 y longitud: -100, latitud: 50.

```
&path=color:0xFF000088|weight:4|45,-110|50,-100
```

<center>

![Polilínea de Google Maps](media/migrate-google-maps-web-services/google-maps-polyline.png)</center>

**Después: Azure Maps**

Para agregar líneas y polígonos a una imagen de mapa estático, especifique el parámetro `path` en la dirección URL. Al igual que en Google Maps, especifique un estilo y una lista de ubicaciones en este parámetro. Especifique el parámetro `path` varias veces para representar varios círculos, líneas y polígonos con distintos estilos.

```
&path=pathStyles||pathLocation1|pathLocation2|...
```

En cuanto a las ubicaciones de la ruta, Azure Maps requiere que las coordenadas estén en formato "longitud latitud". Google Maps usa el formato "latitud,longitud". En el formato de Azure Maps, la longitud y la latitud se separan con un espacio, no con una coma. Azure Maps no admite rutas de acceso codificadas ni direcciones de puntos. Se pueden cargar conjuntos de datos más grandes, como un archivo GeoJSON, en la API de almacenamiento de datos de Azure Maps, tal y como se documenta [aquí](how-to-render-custom-data.md#get-data-from-azure-maps-data-storage).

Agregue estilos de ruta con el formato `optionNameValue`. Separe los distintos estilos con caracteres de barra vertical (\|), de esta manera: `optionName1Value1|optionName2Value2`. Los nombres y valores de las opciones no están separados. Use los siguientes nombres de opción de estilo para aplicar estilo a las rutas de Azure Maps:

- `fa`: opacidad del color de relleno (alfanumérico) usada al representar polígonos. Elija un número entre 0 y 1.
- `fc`: color de relleno usado para representar el área de un polígono.
- `la`: opacidad del color de línea (alfanumérico) usada al representar líneas y el contorno de polígonos. Elija un número entre 0 y 1.
- `lc`: color de línea usado para representar líneas y el contorno de polígonos.
- `lw`: ancho de la línea en píxeles.
- `ra`: especifica el radio de un círculo en metros.

Agregue una opacidad de línea roja y un grosor de píxel entre las coordenadas, en el parámetro URL. En el ejemplo siguiente, la línea tiene una opacidad del 50 % y un grosor de cuatro píxeles. Las coordenadas tienen los siguientes valores: longitud:-110, latitud: 45 y longitud: -100, latitud: 50.

```
&path=lcFF0000|la.5|lw4||-110 45|-100 50
```

<center>

![Polilínea de Azure Maps](media/migrate-google-maps-web-services/azure-maps-polyline.png)</center>

## <a name="calculate-a-distance-matrix"></a>Cálculo de una matriz de distancia

Azure Maps proporciona la API de la matriz de distancias. Use esta API para calcular los tiempos de desplazamiento y las distancias entre un conjunto de ubicaciones, con una matriz de distancias. Es similar a la API de matriz de distancias de Google Maps.

- [**Matriz de ruta**](https://docs.microsoft.com/rest/api/maps/route/postroutematrixpreview): calcula asincrónicamente los tiempos y las distancias de desplazamiento de un conjunto de orígenes y destinos. Admite hasta 700 celdas por solicitud. Es el número de orígenes multiplicado por el número de destinos. Teniendo en cuenta esa restricción, estos son algunos ejemplos de posibles dimensiones de la matriz: 700 x 1, 50 x 10, 10 x 10, 28 x 25, 10 x 70.

> [!NOTE]
> Las solicitudes a la API de matriz de distancia solo se pueden realizar mediante una solicitud POST, con la información de origen y de destino en el cuerpo de la solicitud. Además, Azure Maps requiere que todos los orígenes y destinos sean coordenadas y que las direcciones estén geocodificadas.

En esta tabla se establecen referencias cruzadas entre los parámetros de API de Google Maps y los parámetros de API similares correspondientes de Azure Maps.

| Parámetro de API de Google Maps      | Parámetro de API de Azure Maps similar  |
|--------------------------------|--------------------------------------|
| `arrivial_time`                | `arriveAt`                           |
| `avoid`                        | `avoid`                              |
| `depature_time`                | `departAt`                           |
| `destinations`                 | `destination`: especifíquelo como GeoJSON en el cuerpo de la solicitud POST. |
| `key`                          | `subscription-key` (vea también la documentación de [autenticación con Azure Maps](azure-maps-authentication.md)) |
| `language`                     | `language` (vea la documentación de [compatibilidad de idiomas](supported-languages.md))  |
| `mode`                         | `travelMode`                         |
| `origins`                      | `origins`: especifíquelo como GeoJSON en el cuerpo de la solicitud POST.  |
| `region`                       | *No procede*: esta característica está relacionada con la geocodificación. Use el parámetro `countrySet` al usar la API de geocodificación de Azure Maps) |
| `traffic_model`                | *No procede* (solo puede especificar si los datos de tráfico deben usarse con el parámetro `traffic`) |
| `transit_mode`                 | *No procede*: actualmente no se admiten matrices de distancias basadas en el tránsito.  |
| `transit_routing_preference`   | *No procede*: actualmente no se admiten matrices de distancias basadas en el tránsito.  |
| `units`                        | *No procede* (Azure Maps usa exclusivamente el sistema métrico) |

> [!TIP]
> Todas las opciones avanzadas de enrutamiento disponibles en la API de enrutamiento de Azure Maps son compatibles con la API de matriz de distancias de Azure Maps. Entre las opciones de enrutamiento avanzadas se incluyen: enrutamiento de camiones, especificaciones del motor, etc.

Consulte la documentación sobre [procedimientos recomendados de cálculo de ruta](how-to-use-best-practices-for-routing.md).

## <a name="get-a-time-zone"></a>Obtención de una zona horaria

Azure Maps proporciona una API para recuperar la zona horaria de una coordenada. La API de zona horaria de Azure Maps es similar a la API de zona horaria de Google Maps:

- [**Zona horaria por coordenada**](https://docs.microsoft.com/rest/api/maps/timezone/gettimezonebycoordinates): Especifique una coordenada para recibir los detalles de la zona horaria de la coordenada.

En esta tabla se establecen referencias cruzadas entre los parámetros de API de Google Maps y los parámetros de API similares correspondientes de Azure Maps.

| Parámetro de API de Google Maps | Parámetro de API de Azure Maps similar   |
|---------------------------|---------------------------------------|
| `key`                       | `subscription-key` (vea también la documentación de [autenticación con Azure Maps](azure-maps-authentication.md))       |
| `language`                  | `language` (vea la documentación de [compatibilidad de idiomas](supported-languages.md))    |
| `location`                  | `query`             |
| `timestamp`                 | `timeStamp`         |

Además de esta API, Azure Maps proporciona varias API de zona horaria. Estas API convierten la hora en función de los nombres o los identificadores de la zona horaria:

- [**Zona horaria por identificador**](https://docs.microsoft.com/rest/api/maps/timezone/gettimezonebyid): devuelve la información de zona horaria actual, histórica y futura relativa al identificador de zona horaria de IANA especificado.
- [**Enumeración de zonas horarias, IANA**](https://docs.microsoft.com/rest/api/maps/timezone/gettimezoneenumiana): devuelve una lista completa de los identificadores de zona horaria de IANA. Las actualizaciones del servicio IANA se reflejan en el sistema en el plazo de un día.
- [**Enumeración de zonas horarias, Windows**](https://docs.microsoft.com/rest/api/maps/timezone/gettimezoneenumwindows): devuelve una lista completa de los identificadores de zona horaria de Windows.
- [**Versión de IANA de zona horaria**](https://docs.microsoft.com/rest/api/maps/timezone/gettimezoneianaversion): devuelve el número de versión de IANA actual utilizado por Azure Maps.
- [**Zona horaria de Windows a IANA**](https://docs.microsoft.com/rest/api/maps/timezone/gettimezonewindowstoiana): devuelve el identificador de IANA correspondiente a un identificador de zona horaria de Windows válido especificado. Se pueden obtener varios identificadores de IANA de un mismo identificador de Windows.

## <a name="client-libraries"></a>Bibliotecas de clientes

Azure Maps proporciona bibliotecas de cliente para los siguientes lenguajes de programación:

- JavaScript, TypeScript, Node.js ([documentación](how-to-use-services-module.md) \| [paquete de NPM](https://www.npmjs.com/package/azure-maps-rest))

Estas bibliotecas de cliente de código abierto son para otros lenguajes de programación:

- .NET Standard 2.0 ([proyecto de GitHub](https://github.com/perfahlen/AzureMapsRestServices) \| [paquete NuGet](https://www.nuget.org/packages/AzureMapsRestToolkit/)

## <a name="additional-resources"></a>Recursos adicionales

Estos son algunos recursos y documentación adicionales sobre los servicios REST de Azure Maps.

- [Procedimientos recomendados de búsqueda](how-to-use-best-practices-for-search.md)
- [Búsqueda de una dirección](how-to-search-for-address.md)
- [Procedimientos recomendados de cálculo de ruta](how-to-use-best-practices-for-routing.md)
- [Documentación de referencia de la API del servicio REST de Azure Maps](https://docs.microsoft.com/rest/api/maps/)

## <a name="next-steps"></a>Pasos siguientes

Obtenga más información sobre los servicios REST de Azure Maps.

> [!div class="nextstepaction"]
> [Procedimientos recomendados de uso del servicio de búsqueda](how-to-use-best-practices-for-search.md)

> [!div class="nextstepaction"]
> [Procedimientos recomendados de uso del servicio de cálculo de ruta](how-to-use-best-practices-for-search.md)

> [!div class="nextstepaction"]
> [Uso del módulo de servicios (SDK web)](how-to-use-best-practices-for-routing.md)

> [!div class="nextstepaction"]
> [Documentación de referencia de la API del servicio REST de Azure Maps](https://docs.microsoft.com/rest/api/maps/)

> [!div class="nextstepaction"]
> [Ejemplos de código](https://docs.microsoft.com/samples/browse/?products=azure-maps)
