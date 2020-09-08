---
title: Búsqueda de ubicaciones con los servicios Search de Azure Maps
description: Obtenga información sobre el servicio Search de Azure Maps. Vea cómo usar este conjunto de API para la geocodificación, la geocodificación inversa, las búsquedas parciales y las búsquedas de calles transversales.
author: anastasia-ms
ms.author: v-stharr
ms.date: 07/21/2020
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: philmea
ms.openlocfilehash: 48dd0168f878a16e2eabe47151d0b09993d9f5f9
ms.sourcegitcommit: bfeae16fa5db56c1ec1fe75e0597d8194522b396
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/10/2020
ms.locfileid: "88037786"
---
# <a name="search-for-a-location-using-azure-maps-search-services"></a>Búsqueda de ubicaciones con los servicios Search de Azure Maps

El [servicio Search de Azure Maps](https://docs.microsoft.com/rest/api/maps/search) es un conjunto de API RESTful diseñadas para que los desarrolladores busquen direcciones, lugares y listados de empresas por nombre y categoría u otra información geográfica. Además de permitir la geocodificación tradicional, los servicios pueden realizar también la geocodificación inversa de direcciones y calles transversales en función de la longitud y la latitud. Los valores de latitud y longitud devueltos por la búsqueda se pueden usar como parámetros en otros servicios de Azure Maps, como los servicios de [ruta](https://docs.microsoft.com/rest/api/maps/route) y [tiempo](https://docs.microsoft.com/rest/api/maps/weather).

En este artículo, aprenderá a:

* Solicitar las coordenadas de latitud y longitud de una dirección (geocodificación de la ubicación de la dirección) mediante [Search Address API]( https://docs.microsoft.com/rest/api/maps/search/getsearchaddress).
* Buscar una dirección o punto de interés (POI) mediante [Fuzzy Search API](https://docs.microsoft.com/rest/api/maps/search/getsearchfuzzy).
* Realizar una [búsqueda de dirección inversa](https://docs.microsoft.com/rest/api/maps/search/getsearchaddressreverse) para traducir la ubicación de coordenadas a la dirección postal.
* Traducir la ubicación de coordenadas a una calle transversal que pueda entender cualquier persona mediante [Search Address Reverse Cross Street API](https://docs.microsoft.com/rest/api/maps/search/getsearchaddressreversecrossstreet).  Por lo general, esto es necesario en las aplicaciones de seguimiento que reciben una fuente GPS de un dispositivo o recurso y desea saber en qué dirección se encuentra en la coordenada.

## <a name="prerequisites"></a>Requisitos previos

1. [Cree una cuenta de Azure Maps](quick-demo-map-app.md#create-an-azure-maps-account).
2. [Obtenga una clave de suscripción principal](quick-demo-map-app.md#get-the-primary-key-for-your-account), también conocida como clave principal o clave de suscripción.

En este tutorial se usa la aplicación [Postman](https://www.postman.com/), pero puede elegir un entorno de desarrollo de API diferente.

## <a name="request-latitude-and-longitude-for-an-address-geocoding"></a>Solicitud de la latitud y longitud de una dirección (geocodificación)

En este ejemplo, usaremos [Get Search Address API](https://docs.microsoft.com/rest/api/maps/search/getsearchaddress) de Azure Maps para convertir una dirección en coordenadas de latitud y longitud. Este proceso también se denomina *geocodificación*. Además de devolver las coordenadas, la respuesta también devolverá propiedades de dirección detalladas como la calle, el código postal, el municipio y la información de país o región.

>[!TIP]
>Si tiene un conjunto de direcciones para la geocodificación, puede usar [Post Search Address Batch API](https://docs.microsoft.com/rest/api/maps/search/postsearchaddressbatch) para enviar un lote de consultas en una única llamada API.

1. Abra la aplicación Postman. Cerca de la parte superior de la aplicación Postman, seleccione**New** (Nuevo). En la ventana **Create New** (Crear nuevo), seleccione **Collection** (Colección).  Asigne un nombre a la colección y seleccione el botón **Create** (Crear). Usará esta colección para el resto de los ejemplos de este documento.

2. Para crear la solicitud, seleccione **New** (Nuevo) otra vez. En la ventana **Create New** (Crear nuevo), seleccione **Request** (Solicitud). Escriba un valor de **Request name** (Nombre de solicitud) para la solicitud. Seleccione la colección que creó en el paso anterior y haga clic en **Save** (Guardar).

3. Seleccione el método **GET** HTTP en la pestaña del generador y escriba la dirección URL siguiente. En esta solicitud, estamos buscando una dirección específica: `400 Braod St, Seattle, WA 98109`.

    Para esta solicitud y otras solicitudes mencionadas en este artículo, reemplace `{Azure-Maps-Primary-Subscription-key}` por su clave de suscripción principal. La solicitud debe tener un aspecto similar a la siguiente dirección URL:

    ```http
    https://atlas.microsoft.com/search/address/json?&subscription-key={Azure-Maps-Primary-Subscription-key}&api-version=1.0&language=en-US&query=400 Broad St, Seattle, WA 98109
    ```

4. Haga clic en el botón azul **Enviar**. El cuerpo de la respuesta contendrá datos para una sola ubicación.

5. Ahora buscaremos una dirección que tenga más de una posible ubicación. En la sección **Params** (Parámetros), cambie la clave `query` a `400 Broad, Seattle`. Haga clic en el botón azul **Enviar**.

    :::image type="content" source="./media/how-to-search-for-address/search-address.png" alt-text="Búsqueda de dirección":::

6. Luego, intente establecer la clave `query` en `400 Broa`.

7. Haga clic en el botón **Enviar**.  Ahora puede ver que la respuesta incluye respuestas de varios países. Para sesgar geográficamente los resultados al área pertinente para el usuario, agregue siempre a la solicitud tantos detalles como sea posible.

## <a name="using-fuzzy-search-api"></a>Uso de Fuzzy Search API

[Fuzzy Search API](https://docs.microsoft.com/rest/api/maps/search/getsearchfuzzy) de Azure Maps admiten búsquedas de forma libre y de línea única estándar. Se recomienda usar Azure Maps Search Fuzzy API de Azure Maps cuando no conozca el tipo de entrada del usuario de una solicitud de búsqueda.  La entrada de la consulta puede ser una dirección completa o parcial. También puede ser un token de punto de interés (POI), como un nombre de POI, una categoría de POI o el nombre de una marca. Además, para mejorar la pertinencia de los resultados de la búsqueda, los resultados de la consulta se pueden restringir según un radio y una ubicación de coordenadas o mediante la definición de un rectángulo delimitador.

>[!TIP]
>La mayoría de las consultas de búsqueda tienen como valor predeterminado maxFuzzyLevel=1 para aumentar el rendimiento y reducir los resultados inusuales. Puede ajustar los niveles de aproximación con los parámetros `maxFuzzyLevel` o `minFuzzyLevel`. Para más información sobre `maxFuzzyLevel` y una lista completa de todos los parámetros adicionales, consulte la sección sobre los [parámetros de identificador URI de la búsqueda aproximada](https://docs.microsoft.com/rest/api/maps/search/getsearchfuzzy#uri-parameters).

### <a name="search-for-an-address-using-fuzzy-search"></a>Buscar una dirección mediante la búsqueda aproximada

En este ejemplo, usaremos la búsqueda aproximada para buscar `pizza` en todo el mundo.  A continuación, le mostraremos cómo buscar en el ámbito de un país específico. Por último, le mostraremos cómo usar un radio y una ubicación de coordenadas para limitar el ámbito de una búsqueda en un área específica y limitar el número de resultados devueltos.

>[!IMPORTANT]
>Para sesgar geográficamente los resultados al área pertinente para los usuarios, agregue siempre tantos detalles de ubicación como sea posible. Para más información, consulte [Procedimientos recomendados de búsqueda](how-to-use-best-practices-for-search.md#geobiased-search-results).

1. Abra la aplicación Postman, haga clic en **New** (Nuevo) y seleccione **Request** (Solicitud). Escriba un valor de **Request name** (Nombre de solicitud) para la solicitud. Seleccione la colección que creó en la sección anterior o cree una nueva y, a continuación, seleccione **Save** (Guardar).

2. Seleccione el método **GET** HTTP en la pestaña del generador y escriba la dirección URL siguiente. Para esta solicitud y otras solicitudes mencionadas en este artículo, reemplace `{Azure-Maps-Primary-Subscription-key}` por su clave de suscripción principal. La solicitud debe tener un aspecto similar a la siguiente dirección URL:

    ```http
   https://atlas.microsoft.com/search/fuzzy/json?&api-version=1.0&subscription-key={Azure-Maps-Primary-Subscription-key}&language=en-US&query=pizza
    ```

    >[!NOTE]
    >El atributo _json_ de la ruta de acceso URL determina el formato de respuesta. En este artículo se usa JSON para facilitar el uso y la legibilidad. Para buscar otros formatos de respuesta admitidos, consulte la definición del parámetro `format` en la [documentación de referencia Parámetros de identificador URI](https://docs.microsoft.com/rest/api/maps/search/getsearchfuzzy#uri-parameters).

3. Haga clic en **Enviar** y revise el cuerpo de la respuesta.

    La cadena de consulta ambigua para "pizza" devolvió 10 [resultados de punto de interés](https://docs.microsoft.com/rest/api/maps/search/getsearchpoi#searchpoiresponse) (POI) en las categorías "pizza" y "restaurante". Cada resultado incluye detalles como una dirección, valores de latitud y longitud, ventanilla y puntos de entrada para la ubicación. Los resultados ahora son variados para esta consulta y no están asociados a ninguna ubicación de referencia.
  
    En el paso siguiente, usaremos el parámetro `countrySet` para especificar solo los países o regiones para los que la aplicación necesita cobertura. Para una lista completa de los países y regiones admitidos, consulte [Cobertura de búsqueda](https://docs.microsoft.com/azure/azure-maps/geocoding-coverage).

4. El comportamiento predeterminado es buscar en todo el mundo, lo que puede devolver resultados innecesarios. A continuación, buscaremos pizza solo en los Estados Unidos. Agregue la clave `countrySet` a la sección **Params** (Parámetros) y establezca su valor en `US`. Al establecer la clave `countrySet` en `US`, los resultados se enlazarán a los Estados Unidos.

    :::image type="content" source="./media/how-to-search-for-address/search-fuzzy-country.png" alt-text="Búsqueda de pizza en los Estados Unidos":::

    Ahora los resultados están limitados por el código de país y la consulta devuelve pizzerías de los Estados Unidos.

5. Para obtener una búsqueda aún más dirigida, puede buscar en el ámbito de un par de coordenadas de latitud/longitud. En este ejemplo, usaremos las coordenadas de latitud y longitud de la Space Needle de Seattle. Como solo queremos que se devuelvan resultados dentro de un radio de 400 metros, agregaremos el parámetro `radius`. Además, agregaremos el parámetro `limit` para limitar los resultados a las cinco pizzerías más cercanas.

    En la sección **Params** (Parámetros), agregue los pares clave-valor siguientes:

     | Clave | Value |
    |-----|------------|
    | lat | 47,620525 |
    | lon | -122,349274 |
    | radio | 400 |
    | limit | 5|

6. Haga clic en **Enviar**. La respuesta incluye los resultados de restaurantes de pizza cerca de la Space Needle de Seattle.

## <a name="search-for-a-street-address-using-reverse-address-search"></a>Buscar una dirección mediante la búsqueda de direcciones inversa

[Get Search Address Reverse API]( https://docs.microsoft.com/rest/api/maps/search/getsearchaddressreverse) de Azure Maps traduce las coordenadas en direcciones legibles. Esta API se usa a menudo para las aplicaciones que consumen fuentes GPS y desean detectar direcciones en puntos de coordenadas específicos.

>[!IMPORTANT]
>Para sesgar geográficamente los resultados al área pertinente para los usuarios, agregue siempre tantos detalles de ubicación como sea posible. Para más información, consulte [Procedimientos recomendados de búsqueda](how-to-use-best-practices-for-search.md#geobiased-search-results).

>[!TIP]
>Si tiene un conjunto de direcciones en forma de coordenadas para realizar la geocodificación inversa, puede usar [Post Search Address Batch API](https://docs.microsoft.com/rest/api/maps/search/postsearchaddressreversebatch) para enviar un lote de consultas en una única llamada de API.

En este ejemplo vamos a realizar búsquedas inversas con algunos de los parámetros opcionales que están disponibles. Para una lista completa de los parámetros opcionales, consulte la sección sobre [parámetros de búsqueda inversa](https://docs.microsoft.com/rest/api/maps/search/getsearchaddressreverse#uri-parameters).

1. En la aplicación Postman, haga clic en **New** (Nuevo) y seleccione **Request** (Solicitud). Escriba un valor de **Request name** (Nombre de solicitud) para la solicitud. Seleccione la colección que creó en la primera sección o cree una nueva y, a continuación, seleccione **Save** (Guardar).

2. Seleccione el método **GET** HTTP en la pestaña del generador y escriba la dirección URL siguiente. Para esta solicitud y otras solicitudes mencionadas en este artículo, reemplace `{Azure-Maps-Primary-Subscription-key}` por su clave de suscripción principal. La solicitud debe tener un aspecto similar a la siguiente dirección URL:

    ```http
    https://atlas.microsoft.com/search/address/reverse/json?api-version=1.0&subscription-key={Azure-Maps-Primary-Subscription-key}&language=en-US&query=47.591180,-122.332700&number=1
    ```

3. Haga clic en **Enviar** y revise el cuerpo de la respuesta. Debería ver un resultado de la consulta. La respuesta incluye información de dirección clave sobre Safeco Field.
  
4. Ahora agregaremos los pares clave-valor siguientes en la sección **Params** (Parámetros):

    | Clave | Value | Devuelve
    |-----|------------|------|
    | number | 1 |La respuesta puede incluir el lado de la calle (izquierda/derecha) y también una posición de desplazamiento para el número.|
    | returnSpeedLimit | true | Devuelve el límite de velocidad en la dirección.|
    | returnRoadUse | true | Devuelve los tipos de uso de carretera en la dirección. Para ver todos los tipos de uso de carretera posibles, consulte la sección [Tipos de uso de carretera](https://docs.microsoft.com/rest/api/maps/search/getsearchaddressreverse#uri-parameters).|
    | returnMatchType | true| Devuelve el tipo de coincidencia. Para ver todos los valores posibles, consulte la sección [Resultados de la búsqueda de direcciones inversa](https://docs.microsoft.com/rest/api/maps/search/getsearchaddressreverse#searchaddressreverseresult).

   :::image type="content" source="./media/how-to-search-for-address/search-reverse.png" alt-text="Búsqueda inversa.":::

5. Haga clic en **Enviar** y revise el cuerpo de la respuesta.

6. A continuación, agregaremos la clave `entityType` y estableceremos su valor en `Municipality`. La clave `entityType` reemplazará la clave `returnMatchType` del paso anterior. También tendremos que quitar `returnSpeedLimit` y `returnRoadUse`, puesto que estamos solicitando información sobre el municipio.  Para ver todos los tipos de entidad posibles, consulte [Tipos de entidad](https://docs.microsoft.com/rest/api/maps/search/getsearchaddressreverse#entitytype).

    :::image type="content" source="./media/how-to-search-for-address/search-reverse-entity-type.png" alt-text="Tipo de entidad de búsqueda inversa.":::

7. Haga clic en **Enviar**. Compare los resultados con los resultados devueltos en el paso 5.  Como el tipo de entidad solicitado ahora es `municipality`, la respuesta no incluye información de dirección postal. Además, el valor de `geometryId` devuelto se puede usar para solicitar un límite poligonal a través de Get [Search Polygon API](https://docs.microsoft.com/rest/api/maps/search/getsearchpolygon) de Azure Maps.

>[!TIP]
>Para más información sobre estos parámetros, así como aprender sobre otros, consulte la sección [Parámetros de búsqueda inversa](https://docs.microsoft.com/rest/api/maps/search/getsearchaddressreverse#uri-parameters).

## <a name="search-for-cross-street-using-reverse-address-cross-street-search"></a>Búsqueda de la calle transversal mediante la búsqueda de calles transversales de direcciones inversa

En este ejemplo, buscaremos una calle transversal en función de las coordenadas de una dirección.

1. En la aplicación Postman, haga clic en **New** (Nuevo) y seleccione **Request** (Solicitud). Escriba un valor de **Request name** (Nombre de solicitud) para la solicitud. Seleccione la colección que creó en la primera sección o cree una nueva y, a continuación, seleccione **Save** (Guardar).

2. Seleccione el método **GET** HTTP en la pestaña del generador y escriba la dirección URL siguiente. Para esta solicitud y otras solicitudes mencionadas en este artículo, reemplace `{Azure-Maps-Primary-Subscription-key}` por su clave de suscripción principal. La solicitud debe tener un aspecto similar a la siguiente dirección URL:
  
    ```http
   https://atlas.microsoft.com/search/address/reverse/crossstreet/json?&api-version=1.0&subscription-key={Azure-Maps-Primary-Subscription-key}&language=en-US&query=47.591180,-122.332700
    ```

    :::image type="content" source="./media/how-to-search-for-address/search-address-cross.png" alt-text="Búsqueda de calle transversal.":::
  
3. Haga clic en **Enviar** y revise el cuerpo de la respuesta. Verá que la respuesta contiene un valor `crossStreet` de `Occidental Avenue South`.

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [API REST del servicio Search de Azure Maps](https://docs.microsoft.com/rest/api/maps/search)

> [!div class="nextstepaction"]
> [Procedimientos recomendados del servicio Search de Azure Maps](how-to-use-best-practices-for-search.md)
