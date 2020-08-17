---
title: Representación de datos personalizados en un mapa de trama | Microsoft Azure Maps
description: En este artículo, aprenderá a representar datos personalizados en un mapa de trama mediante el servicio de imagen estática de Microsoft Azure Maps.
author: anastasia-ms
ms.author: v-stharr
ms.date: 01/23/2020
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: timlt
ms.custom: mvc
ms.openlocfilehash: 6eae060b2ed2b0f7d6de5ac54d92677c95fc8a89
ms.sourcegitcommit: 0e8a4671aa3f5a9a54231fea48bcfb432a1e528c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/24/2020
ms.locfileid: "87126880"
---
# <a name="render-custom-data-on-a-raster-map"></a>Representación de datos personalizados en un mapa de trama

En este artículo se explica cómo usar el [servicio de imagen estática](https://docs.microsoft.com/rest/api/maps/render/getmapimage) con la funcionalidad de composición de imágenes para permitir las superposiciones encima de un mapa de trama. La composición de imágenes incluye la capacidad de obtener un icono de mapa de trama con datos adicionales, como marcadores, etiquetas y superposiciones geométricas personalizadas.

Para representar marcadores, etiquetas y superposiciones geométricas personalizadas, se puede usar la aplicación Postman. Puede usar las [API de Data Service](https://docs.microsoft.com/rest/api/maps/data) de Azure Maps para almacenar y representar superposiciones.

> [!Tip]
> A menudo es mucho más rentable usar el SDK web de Azure Maps para mostrar un mapa sencillo en una página web que usar el servicio de imágenes estáticas. El SDK web usa mosaicos de mapa y, a menos que el usuario desplace lateralmente el mapa y lo acerque, con frecuencia solo se generará una fracción de una transacción por carga del mapa. Observe que el SDK web de Azure Maps tiene opciones para deshabilitar las funciones de desplazar lateralmente y acercar. Además, ofrece un conjunto mucho más rico de opciones de visualización de datos que las de un servicio web de mapas estáticos.  

## <a name="prerequisites"></a>Prerrequisitos

### <a name="create-an-azure-maps-account"></a>Crear una cuenta de Azure Maps

Para completar los procedimientos descritos en este artículo, primero debe crear una cuenta de Azure Maps y obtener la clave de cuenta de los mapas. Siga las instrucciones que se indican en [Creación de una cuenta](quick-demo-map-app.md#create-an-azure-maps-account) para crear una suscripción a una cuenta de Azure Maps y siga los pasos para [obtener la clave principal](quick-demo-map-app.md#get-the-primary-key-for-your-account) de la cuenta. Para más información sobre la autenticación en Azure Maps, consulte [Administración de la autenticación en Azure Maps](./how-to-manage-authentication.md).


## <a name="render-pushpins-with-labels-and-a-custom-image"></a>Representación de marcadores con etiquetas e imágenes personalizadas

> [!Note]
> El procedimiento descrito en esta sección requiere una cuenta de Azure Maps en el plan de tarifa S0 o S1.

El nivel S0 de la cuenta de Azure Maps solo admite una única instancia del parámetro `pins`. Permite representar hasta cinco marcadores, especificados en la solicitud de dirección URL, con una imagen personalizada.

Para representar los marcadores con etiquetas y una imagen personalizada, siga estos pasos:

1. Cree una colección en la que vaya a almacenar las solicitudes. En la aplicación Postman, seleccione**New** (Nuevo). En la ventana **Create New** (Crear nuevo), seleccione **Collection** (Colección). Asigne un nombre a la colección y seleccione el botón **Create** (Crear). 

2. Para crear la solicitud, seleccione **New** (Nuevo) otra vez. En la ventana **Create New** (Crear nuevo), seleccione **Request** (Solicitud). Escriba un valor de **Request name** (Nombre de solicitud) para los marcadores. Seleccione la colección que creó en el paso anterior como ubicación en la que se va a guardar la solicitud. Después, seleccione **Guardar**.
    
    ![Creación de una solicitud en Postman](./media/how-to-render-custom-data/postman-new.png)

3. Seleccione el método GET HTTP en la pestaña del generador y escriba la siguiente dirección URL para realizar una solicitud GET.

    ```HTTP
    https://atlas.microsoft.com/map/static/png?subscription-key={subscription-key}&api-version=1.0&layer=basic&style=main&zoom=12&center=-73.98,%2040.77&pins=custom%7Cla15+50%7Cls12%7Clc003b61%7C%7C%27CentralPark%27-73.9657974+40.781971%7C%7Chttps%3A%2F%2Fraw.githubusercontent.com%2FAzure-Samples%2FAzureMapsCodeSamples%2Fmaster%2FAzureMapsCodeSamples%2FCommon%2Fimages%2Ficons%2Fylw-pushpin.png
    ```
    Esta es la imagen resultante:

    ![Un marcador personalizado con una etiqueta](./media/how-to-render-custom-data/render-pins.png)


## <a name="get-data-from-azure-maps-data-storage"></a>Obtener datos del almacenamiento de datos de Azure Maps

> [!Note]
> El procedimiento descrito en esta sección requiere una cuenta de Azure Maps en el plan de tarifa S1.

También puede obtener la información de ubicación de ruta de acceso y de pin mediante la [API de carga de datos](https://docs.microsoft.com/rest/api/maps/data/uploadpreview). Siga los pasos descritos a continuación para cargar los datos de la ruta de acceso y de los marcadores.

1. En la aplicación Postman, abra una nueva pestaña en la colección que creó en la sección anterior. Seleccione el método POST HTTP en la pestaña del generador y escriba la siguiente dirección URL para realizar una solicitud POST:

    ```HTTP
    https://atlas.microsoft.com/mapData/upload?subscription-key={subscription-key}&api-version=1.0&dataFormat=geojson
    ```

2. En la pestaña **Params** (Parámetros), escriba los siguientes pares de clave-valor que se usarán para la dirección URL de la solicitud POST. Reemplace el valor de `subscription-key` por su clave de suscripción de Azure Maps.
    
    ![Parámetros de clave-valor en Postman](./media/how-to-render-custom-data/postman-key-vals.png)

3. En **Body** (Cuerpo), seleccione el formato de entrada sin procesar y elija JSON como formato de entrada en la lista desplegable. Proporcione el código JSON como datos que se van a cargar:
    
    ```JSON
    {
      "type": "FeatureCollection",
      "features": [
        {
          "type": "Feature",
          "properties": {},
          "geometry": {
            "type": "Polygon",
            "coordinates": [
              [
                [
                  -73.98235,
                  40.76799
                ],
                [
                  -73.95785,
                  40.80044
                ],
                [
                  -73.94928,
                  40.7968
                ],
                [
                  -73.97317,
                  40.76437
                ],
                [
                  -73.98235,
                  40.76799
                ]
              ]
            ]
          }
        },
        {
          "type": "Feature",
          "properties": {},
          "geometry": {
            "type": "LineString",
            "coordinates": [
              [
                -73.97624731063843,
                40.76560773817073
              ],
              [
                -73.97914409637451,
                40.766826609362575
              ],
              [
                -73.98513078689575,
                40.7585866048861
              ]
            ]
          }
        }
      ]
    }
    ```

4. Haga clic en **Send** (Enviar) y revise el encabezado de la respuesta. Tras una solicitud correcta, el encabezado de ubicación contendrá el URI de estado para comprobar el estado actual de la solicitud de carga. El URI de estado tendría el formato siguiente.  

   ```HTTP
   https://atlas.microsoft.com/mapData/{uploadStatusId}/status?api-version=1.0
   ```

5. Copie el identificador URI de estado y anéxele un parámetro subscription-key, cuyo valor sea la clave de suscripción de la cuenta de Azure Maps. Use la misma clave de suscripción de la cuenta que usó para cargar los datos. El formato del URI de estado debe ser similar al siguiente:

   ```HTTP
   https://atlas.microsoft.com/mapData/{uploadStatusId}/status?api-version=1.0&subscription-key={Subscription-key}
   ```

6. Para obtener el UDID, abra una nueva pestaña en la aplicación de Postman. Seleccione el método GET HTTP en la pestaña del generador. Realice una solicitud GET en el identificador URI de estado. Si la carga de datos se realizó correctamente, recibirá un UDID en el cuerpo de respuesta. Copie el UDID.

   ```JSON
   {
      "udid" : "{udId}"
   }
   ```

7. Use el valor de `udId` recibido de la API de carga de datos para representar las características en el mapa. Para ello, abra una nueva pestaña en la colección que creó en la sección anterior. Seleccione el método GET HTTP en la pestaña Builder, reemplace {subscription-Key} y {udId} por sus valores y escriba esta dirección URL para realizar una solicitud GET:

    ```HTTP
    https://atlas.microsoft.com/map/static/png?subscription-key={subscription-key}&api-version=1.0&layer=basic&style=main&zoom=12&center=-73.96682739257812%2C40.78119135317995&pins=default|la-35+50|ls12|lc003C62|co9B2F15||'Times Square'-73.98516297340393 40.758781646381024|'Central Park'-73.96682739257812 40.78119135317995&path=lc0000FF|fc0000FF|lw3|la0.80|fa0.30||udid-{udId}
    ```

    Esta es la imagen de respuesta:

    ![Obtener datos del almacenamiento de datos de Azure Maps](./media/how-to-render-custom-data/uploaded-path.png)

## <a name="render-a-polygon-with-color-and-opacity"></a>Representación de un polígono con color y opacidad

> [!Note]
> El procedimiento descrito en esta sección requiere una cuenta de Azure Maps en el plan de tarifa S1.


Puede modificar la apariencia de un polígono mediante el uso de modificadores de estilo con el [parámetro path](https://docs.microsoft.com/rest/api/maps/render/getmapimage#uri-parameters).

1. En la aplicación Postman, abra una nueva pestaña en la colección que creó antes. Seleccione el método GET HTTP en la pestaña del generador y escriba la siguiente dirección URL para realizar una solicitud GET a fin de representar un polígono con color y opacidad:
    
    ```HTTP
    https://atlas.microsoft.com/map/static/png?api-version=1.0&style=main&layer=basic&sku=S1&zoom=14&height=500&Width=500&center=-74.040701, 40.698666&path=lc0000FF|fc0000FF|lw3|la0.80|fa0.50||-74.03995513916016 40.70090237454063|-74.04082417488098 40.70028420372218|-74.04113531112671 40.70049568385827|-74.04298067092896 40.69899904076542|-74.04271245002747 40.69879568992435|-74.04367804527283 40.6980961582905|-74.04364585876465 40.698055487620714|-74.04368877410889 40.698022951066996|-74.04168248176573 40.696444909137|-74.03901100158691 40.69837271818651|-74.03824925422668 40.69837271818651|-74.03809905052185 40.69903971085914|-74.03771281242369 40.699340668780984|-74.03940796852112 40.70058515602143|-74.03948307037354 40.70052821920425|-74.03995513916016 40.70090237454063
    &subscription-key={subscription-key}
    ```

    Esta es la imagen de respuesta:

    ![Representación de un polígono opaco](./media/how-to-render-custom-data/opaque-polygon.png)


## <a name="render-a-circle-and-pushpins-with-custom-labels"></a>Representación de un círculo y marcadores con etiquetas personalizadas

> [!Note]
> El procedimiento descrito en esta sección requiere una cuenta de Azure Maps en el plan de tarifa S1.


Puede aplicar modificadores de estilo para modificar la apariencia de los marcadores. Por ejemplo, para hacer que los marcadores y sus etiquetas sean mayores o menores, use el modificador "estilo de escala" `sc`. Este modificador toma un valor que es mayor que cero. Un valor de 1 es la escala estándar. Los valores mayores que 1 aumentarán el tamaño de los marcadores, mientras que los valores menores que 1 reducirán su tamaño. Para más información sobre los modificadores de estilo, consulte los [parámetros path del servicio de imagen estática](https://docs.microsoft.com/rest/api/maps/render/getmapimage#uri-parameters).


Siga estos pasos para representar un círculo y marcadores con etiquetas personalizadas:

1. En la aplicación Postman, abra una nueva pestaña en la colección que creó antes. Seleccione el método GET HTTP en la pestaña del generador y escriba esta dirección URL para realizar una solicitud GET:

    ```HTTP
    https://atlas.microsoft.com/map/static/png?api-version=1.0&style=main&layer=basic&zoom=14&height=700&Width=700&center=-122.13230609893799,47.64599069048016&path=lcFF0000|lw2|la0.60|ra1000||-122.13230609893799 47.64599069048016&pins=default|la15+50|al0.66|lc003C62|co002D62||'Microsoft Corporate Headquarters'-122.14131832122801  47.64690503939462|'Microsoft Visitor Center'-122.136828 47.642224|'Microsoft Conference Center'-122.12552547454833 47.642940335653996|'Microsoft The Commons'-122.13687658309935  47.64452336193245&subscription-key={subscription-key}
    ```

    Esta es la imagen de respuesta:

    ![Representación de un círculo con marcadores personalizados](./media/how-to-render-custom-data/circle-custom-pins.png)

2. Para cambiar el color de los marcadores del último paso, cambie el modificador de estilo "co". Observe `pins=default|la15+50|al0.66|lc003C62|co002D62|`; el color actual se especificaría como #002D62 en CSS. Supongamos que desea cambiarlo a #41d42a. Escriba el nuevo valor de color después del especificador "co", así: `pins=default|la15+50|al0.66|lc003C62|co41D42A|`. Realice una nueva solicitud GET:

    ```HTTP
    https://atlas.microsoft.com/map/static/png?api-version=1.0&style=main&layer=basic&zoom=14&height=700&Width=700&center=-122.13230609893799,47.64599069048016&path=lcFF0000|lw2|la0.60|ra1000||-122.13230609893799 47.64599069048016&pins=default|la15+50|al0.66|lc003C62|co41D42A||'Microsoft Corporate Headquarters'-122.14131832122801  47.64690503939462|'Microsoft Visitor Center'-122.136828 47.642224|'Microsoft Conference Center'-122.12552547454833 47.642940335653996|'Microsoft The Commons'-122.13687658309935  47.64452336193245&subscription-key={subscription-key}
    ```

    Esta es la imagen de respuesta después de cambiar los colores de los marcadores:

    ![Representación de un círculo con marcadores actualizados](./media/how-to-render-custom-data/circle-updated-pins.png)

Del mismo modo, puede cambiar, agregar y quitar otros modificadores de estilo.

## <a name="next-steps"></a>Pasos siguientes


* Explore la documentación sobre [Get Map Image API de Azure Maps Get](https://docs.microsoft.com/rest/api/maps/render/getmapimage).
* Para más información sobre las funcionalidades Azure Maps Data Service, consulte la [documentación sobre el servicio](https://docs.microsoft.com/rest/api/maps/data).

