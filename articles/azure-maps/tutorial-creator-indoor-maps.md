---
title: Uso de Creator para crear planos interiores
description: Use Azure Maps Creator para crear planos interiores.
author: anastasia-ms
ms.author: v-stharr
ms.date: 06/17/2020
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: philmea
ms.openlocfilehash: 7ea1995b6d1232b3e4c6371313e5b3d45bdbb756
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87075403"
---
# <a name="use-creator-to-create-indoor-maps"></a>Uso de Creator para crear planos interiores

En este tutorial verá cómo crear planos interiores. En este tutorial, aprenderá a usar la API para realizar lo siguiente:

> [!div class="checklist"]
> * Cargar el paquete de dibujo del plano interior.
> * Convertir el paquete de dibujo en datos del plano.
> * Crear un conjunto de datos a partir de los datos del plano.
> * Crear un conjunto de mosaicos a partir de los datos del conjunto de datos.
> * Consultar la API del servicio de características web (WFS) de Azure Maps para obtener información acerca de las características del plano.
> * Crear un conjunto de estados de características mediante las características del plano y los datos del conjunto de datos.
> * Actualizar el conjunto de estados de características.

## <a name="prerequisites"></a>Requisitos previos

Para crear planos interiores:

1. [Cree una cuenta de Azure Maps](quick-demo-map-app.md#create-an-azure-maps-account).
2. [Obtenga una clave de suscripción principal](quick-demo-map-app.md#get-the-primary-key-for-your-account), también conocida como clave principal o clave de suscripción.
3. [Cree un recurso de Creator](how-to-manage-creator.md)
4. Descargue el [paquete de dibujo de ejemplo](https://github.com/Azure-Samples/am-creator-indoor-data-examples).

En este tutorial se usa la aplicación [Postman](https://www.postman.com/), pero puede elegir un entorno de desarrollo de API diferente.

>[!IMPORTANT]
> Es posible que las direcciones URL de la API de este documento tengan que ajustarse según la ubicación del recurso del Creador. Para más información, consulte [Acceso a Servicios del creador](how-to-manage-creator.md#access-to-creator-services).

## <a name="upload-a-drawing-package"></a>Carga de un paquete de dibujo

Use [Data Upload API](https://docs.microsoft.com/rest/api/maps/data/uploadpreview) para cargar un paquete de dibujo a los recursos de Azure Maps.

Data Upload API es una transacción de larga duración que implementa el patrón que se define aquí. Una vez completada la operación, usaremos el valor `udid` para obtener acceso al paquete cargado y poder convertirlo. Para ello, siga estos pasos para obtener el valor `udid`.

1. Abra la aplicación Postman. Cerca de la parte superior de la aplicación Postman, seleccione**New** (Nuevo). En la ventana **Create New** (Crear nuevo), seleccione **Collection** (Colección).  Asigne un nombre a la colección y seleccione el botón **Create** (Crear).

2. Para crear la solicitud, seleccione **New** (Nuevo) otra vez. En la ventana **Create New** (Crear nuevo), seleccione **Request** (Solicitud). Escriba un valor de **Request name** (Nombre de solicitud) para la solicitud. Seleccione la colección que creó en el paso anterior y haga clic en **Save** (Guardar).

3. Seleccione el método HTTP **POST** en la pestaña del generador y escriba la siguiente dirección URL para cargar el paquete de dibujo en el servicio Azure Maps. Para esta solicitud y otras solicitudes mencionadas en este artículo, reemplace `{Azure-Maps-Primary-Subscription-key}` por su clave de suscripción principal.

    ```http
    https://atlas.microsoft.com/mapData/upload?api-version=1.0&dataFormat=zip&subscription-key={Azure-Maps-Primary-Subscription-key}
    ```

4. En la pestaña de **encabezados**, especifique un valor para la clave `Content-Type`. El paquete de dibujo es una carpeta comprimida, por lo que debe usar el valor `application/octet-stream`. En la pestaña **Body** (Cuerpo), seleccione **binary** (binario). Haga clic en **Select File** (Seleccionar archivo) y elija un paquete de dibujo.

     ![administración de datos](./media/tutorial-creator-indoor-maps/enter-content-type-dialog.png)

5. Haga clic en el botón azul **Send** (Enviar) y espere a que se procese la solicitud. Una vez finalizada la solicitud, vaya a la pestaña **Headers** (Encabezados) de la respuesta. Copie el valor de la clave **Location** (Ubicación), que es `status URL`.

6. Para comprobar el estado de la llamada de API, cree una solicitud HTTP **GET** en el elemento `status URL`. Tendrá que anexar la clave de suscripción principal a la dirección URL para realizar la autenticación. La solicitud **GET** debe tener un aspecto similar a la siguiente dirección URL:

    ```http
    https://atlas.microsoft.com/mapData/operations/<operationId>?api-version=1.0&subscription-key={Azure-Maps-Primary-Subscription-key}
    ```

7. Cuando a solicitud HTTP **GET** se completa correctamente, devolverá una `resourceLocation`. El `resourceLocation` contiene el `udid` único para el contenido cargado. Opcionalmente, puede usar la dirección URL del `resourceLocation` para recuperar los metadatos de este recurso en el siguiente paso.

    ```json
    {
        "status": "Succeeded",
        "resourceLocation": "https://atlas.microsoft.com/mapData/metadata/{udid}?api-version=1.0"
    }
    ```

8. Para recuperar metadatos de contenido, cree una solicitud HTTP **GET** en la dirección URL `resourceLocation` que se recuperó en el paso 7. Asegúrese de anexar la clave de suscripción principal a la dirección URL para la autenticación. La solicitud **GET** debe ser como la siguiente dirección URL:

    ```http
   https://atlas.microsoft.com/mapData/metadata/{udid}?api-version=1.0&subscription-key={Azure-Maps-Primary-Subscription-key}
    ```

9. Cuando la solicitud HTTP **GET** se completa correctamente, el cuerpo de la respuesta contendrá el `udid` especificado en el `resourceLocation` del paso 7, la ubicación para obtener acceso o descargar el contenido en el futuro y otros metadatos sobre el contenido, como la fecha de creación o actualización, el tamaño, etc. Un ejemplo de la respuesta general es:

    ```json
    {
        "udid": "{udid}",
        "location": "https://atlas.microsoft.com/mapData/{udid}?api-version=1.0",
        "created": "2020-02-03T02:32:25.0509366+00:00",
        "updated": "2020-02-11T06:12:13.0309351+00:00",
        "sizeInBytes": 766,
        "uploadStatus": "Completed"
    }
    ```

## <a name="convert-a-drawing-package"></a>Conversión de un paquete de dibujo

 Cuando se haya cargado el paquete de dibujo, usaremos `udid` en el paquete cargado con el fin de convertirlo en datos de plano. Conversion API usa una transacción de larga duración que implementa el patrón que se define [aquí](creator-long-running-operation.md). Una vez completada la operación, usaremos el valor `conversionId` para obtener acceso los datos convertidos. Siga estos pasos para obtener el valor `conversionId`.

1. Seleccione **Nuevo**. En la ventana **Create New** (Crear nuevo), seleccione **Request** (Solicitud). Escriba un **nombre de solicitud** y seleccione una colección. Haga clic en **Save**(Guardar).

2. Seleccione el método HTTP **POST** en la pestaña del generador y escriba la siguiente dirección URL para convertir el paquete de dibujo cargado en los datos del plano. Use el valor `udid` en el paquete cargado.

    ```http
    https://atlas.microsoft.com/conversion/convert?subscription-key={Azure-Maps-Primary-Subscription-key}&api-version=1.0&udid={udid}&inputType=DWG
    ```
    >[!IMPORTANT]
    > Es posible que las direcciones URL de la API de este documento tengan que ajustarse según la ubicación del recurso del Creador. Para más información, consulte [Acceso a Servicios del creador](how-to-manage-creator.md#access-to-creator-services).

3. Haga clic en el botón **Send** (Enviar) y espere a que se procese la solicitud. Una vez finalizada la solicitud, vaya a la pestaña **Headers** (Encabezados) de la respuesta y busque la clave **Location** (Ubicación). Copie el valor de la clave **Location** (Ubicación), que es `status URL` en la solicitud de conversión.

4. Inicie un nuevo método HTTP **GET**  en la pestaña del generador. Anexe la clave de suscripción principal de Azure Maps a `status URL`. Realice una solicitud **GET** en el elemento `status URL` del paso anterior. Si aún no se ha completado el proceso de conversión, puede ver algo parecido a la siguiente respuesta JSON:

    ```json
    {
        "operationId": "77dc9262-d3b8-4e32-b65d-74d785b53504",
        "created": "2020-04-22T19:39:54.9518496+00:00",
        "status": "Running"
    }
    ```

5. Una vez que la solicitud se complete correctamente, verá un mensaje de estado correcto en el cuerpo de la respuesta.  Copie el valor `conversionId` de la dirección URL `resourceLocation` del paquete convertido. Otra API usa el valor `conversionId` para obtener acceso a los datos de plano convertidos.

    ```json
   {
        "operationId": "77dc9262-d3b8-4e32-b65d-74d785b53504",
        "created": "2020-04-22T19:39:54.9518496+00:00",
        "status": "Succeeded",
        "resourceLocation": "https://atlas.microsoft.com/conversion/{conversionId}?api-version=1.0",
        "properties": {}
    }
    ```

>[!NOTE]
>La aplicación Postman no admite de forma nativa solicitudes de ejecución de larga duración de HTTP. Como resultado, es posible que observe un retraso largo mientras realiza una solicitud **GET** en la dirección URL del estado.  Espere unos treinta segundos y vuelva a hacer clic en el botón **Send** (Enviar) hasta que la respuesta se muestre correctamente o con errores.

Tenga en cuenta que el paquete de dibujo de ejemplo debe convertirse sin errores ni advertencias. Sin embargo, si recibe errores o advertencias de su propio paquete de dibujo, la respuesta JSON le proporcionará un vínculo al [visualizador de errores de dibujo](drawing-error-visualizer.md). El visualizador de errores de dibujo le permitirá inspeccionar los detalles de los errores y las advertencias. Para recibir recomendaciones sobre cómo resolver errores y advertencias de conversión, consulte [Errores y advertencias en la conversión de dibujos](drawing-conversion-error-codes.md).

```json
{
    "operationId": "77dc9262-d3b8-4e32-b65d-74d785b53504",
    "created": "2020-04-22T19:39:54.9518496+00:00",
    "status": "Failed",
    "resourceLocation": "https://atlas.microsoft.com/conversion/{conversionId}?api-version=1.0",
    "properties": {
        "diagnosticPackageLocation": "https://atlas.microsoft.com/mapData/ce61c3c1-faa8-75b7-349f-d863f6523748?api-version=1.0"
    }
}
```

## <a name="create-a-dataset"></a>Crear un conjunto de datos

Un conjunto de datos es una colección de características de plano, como edificios, niveles y salones. Para crear un conjunto de datos, use [Dataset Create API](https://docs.microsoft.com/rest/api/maps/dataset/createpreview). Dataset Create API toma el valor `conversionId` del paquete de dibujo convertido y devuelve un valor `datasetId` del conjunto de datos creado. En los pasos siguientes se muestra cómo crear un conjunto de datos.

1. En la aplicación Postman, seleccione**New** (Nuevo). En la ventana **Create New** (Crear nuevo), seleccione **Request** (Solicitud). Escriba un **nombre de solicitud** y seleccione una colección. Haga clic en **Guardar**

2. Realice una solicitud **POST** a [Dataset Create API](https://docs.microsoft.com/rest/api/maps/dataset/createpreview) para crear un nuevo conjunto de datos. Antes de enviar la solicitud, anexe la clave de suscripción y el valor `conversionId` con el valor `conversionId` obtenidos durante el proceso de conversión en el paso 5.  La solicitud debe tener un aspecto similar a la siguiente dirección URL:

    ```http
    https://atlas.microsoft.com/dataset/create?api-version=1.0&conversionID={conversionId}&type=facility&subscription-key={Azure-Maps-Primary-Subscription-key}
    ```

3. Obtenga el valor `statusURL` en la clave **Location** (Ubicación) de los **encabezados** de respuesta.

4. Realice una solicitud **GET** en `statusURL` para obtener el valor de `datasetId`. Anexe la clave de suscripción principal de Azure Maps para realizar la autenticación. La solicitud debe tener un aspecto similar a la siguiente dirección URL:

    ```http
    https://atlas.microsoft.com/dataset/operations/<operationId>?api-version=1.0&subscription-key={Azure-Maps-Primary-Subscription-key}
    ```

5. Cuando la solicitud HTTP **GET** se completa correctamente, el encabezado de respuesta contendrá el valor `datasetId` del conjunto de datos creado. Copie el valor de `datasetId`. Deberá usar `datasetId` para crear un conjunto de mosaicos.

    ```json
    {
        "operationId": "a93570cb-3e4f-4e45-a2b1-360df174180a",
        "created": "2020-04-22T19:52:38.9352189+00:00",
        "status": "Succeeded",
        "resourceLocation": "https://azure.microsoft.com/dataset/{datasetiId}?api-version=1.0"
     }
    ```

## <a name="create-a-tileset"></a>Creación de un conjunto de mosaicos

Un conjunto de mosaicos es un conjunto de mosaicos vectoriales que se representan en el plano. Los conjuntos de mosaicos se crean a partir de conjuntos de datos existentes. Sin embargo, un conjunto de mosaicos es independiente del conjunto de datos del que se creó. Si se elimina el conjunto de datos, el conjunto de mosaicos seguirá existiendo. Siga estos pasos para crear un conjunto de mosaicos:

1. En la aplicación Postman, seleccione**New** (Nuevo). En la ventana **Create New** (Crear nuevo), seleccione **Request** (Solicitud). Escriba un **nombre de solicitud** y seleccione una colección. Haga clic en **Guardar**

2. Realice una solicitud **POST** en la pestaña del generador. La dirección URL de la solicitud debe tener un aspecto similar a la siguiente dirección URL:

    ```http
    https://atlas.microsoft.com/tileset/create/vector?api-version=1.0&datasetID={datasetId}&subscription-key={Azure-Maps-Primary-Subscription-key}
    ```

3. Realice una solicitud **GET** en `statusURL` para obtener el conjunto de mosaicos. Anexe la clave de suscripción principal de Azure Maps para realizar la autenticación. La solicitud debe tener un aspecto similar a la siguiente dirección URL:

   ```http
    https://atlas.microsoft.com/tileset/operations/<operationId>?api-version=1.0&subscription-key=<Azure-Maps-Primary-Subscription-key>
    ```

4. Cuando la solicitud HTTP **GET** se complete correctamente, el encabezado de respuesta contendrá el valor `tilesetId` del conjunto de mosaicos creado. Copie el valor de `tilesetId`.

    ```json
    {
        "operationId": "a93570cb-3e4f-4e45-a2b1-360df174180a",
        "createdDateTime": "3/11/2020 8:45:13 PM +00:00",
        "status": "Succeeded",
        "resourceLocation": "https://atlas.microsoft.com/tileset/{tilesetId}?api-version=1.0"
    }
    ```

## <a name="query-datasets-with-wfs-api"></a>Consulta de conjuntos de datos con la API de WFS

 Los conjuntos de datos se pueden consultar mediante la [API de WFS](https://docs.microsoft.com/rest/api/maps/wfs). Con la API de WFS puede consultar colecciones de características, una colección específica o una característica específica con un **id.** de característica. El **id.** de característica identifica de forma única la característica en el conjunto de datos. Se usa, por ejemplo, para identificar qué estado de la característica debe actualizarse en un conjunto de estados determinado.

1. En la aplicación Postman, seleccione**New** (Nuevo). En la ventana **Create New** (Crear nuevo), seleccione **Request** (Solicitud). Escriba un **nombre de solicitud** y seleccione una colección. Haga clic en **Guardar**

2. Cree una solicitud **GET** para ver una lista de las colecciones del conjunto de datos. Reemplace `<dataset-id>` con `datasetId`. Use la clave principal de Azure Maps en lugar del marcador de posición. La solicitud debe tener un aspecto similar a la siguiente dirección URL:

    ```http
    https://atlas.microsoft.com/wfs/datasets/{datasetId}/collections?subscription-key={Azure-Maps-Primary-Subscription-key}&api-version=1.0
    ```

3. El cuerpo de la respuesta se entregará en formato GeoJSON y contendrá todas las colecciones del conjunto de datos. Para simplificar, en este ejemplo solo se muestra la colección de `unit`. Para ver un ejemplo que contenga todas las colecciones, consulte [Describe Collections API de WFS](https://docs.microsoft.com/rest/api/maps/wfs/collectiondescriptionpreview). Para obtener más información sobre cualquier colección, puede hacer clic en cualquiera de las direcciones URL del elemento `link`.

    ```json
    {
    "collections": [
        {
            "name": "unit",
            "description": "A physical and non-overlapping area which might be occupied and traversed by a navigating agent. Can be a hallway, a room, a courtyard, etc. It is surrounded by physical obstruction (wall), unless the is_open_area attribute is equal to true, and one must add openings where the obstruction shouldn't be there. If is_open_area attribute is equal to true, all the sides are assumed open to the surroundings and walls are to be added where needed. Walls for open areas are represented as a line_element or area_element with is_obstruction equal to true.",
            "links": [
                {
                    "href": "https://atlas.microsoft.com/wfs/datasets/{datasetId}/collections/unit/definition?api-version=1.0",
                    "rel": "describedBy",
                    "title": "Metadata catalogue for unit"
                },
                {
                    "href": "https://atlas.microsoft.com/wfs/datasets/{datasetId}/collections/unit/items?api-version=1.0",
                    "rel": "data",
                    "title": "unit"
                }
                {
                    "href": "https://atlas.microsoft.com/wfs/datasets/{datasetId}/collections/unit?api-version=1.0",
                    "rel": "self",
                    "title": "Metadata catalogue for unit"
                }
            ]
        },
    ```

4. Cree una solicitud **GET** para las colecciones de características `unit`.  Reemplace `{datasetId}` con su `datasetId`. Use la clave principal de Azure Maps en lugar del marcador de posición. El cuerpo de la respuesta contendrá todas las características de la colección `unit`. La solicitud debe tener un aspecto similar a la siguiente dirección URL:

    ```http
    https://atlas.microsoft.com/wfs/datasets/{datasetId}/collections/unit/items?subscription-key={Azure-Maps-Primary-Subscription-key}&api-version=1.0
    ```

5. Copie el `id` de característica correspondiente a una característica de unidad que tenga propiedades de estilo que se puedan modificar dinámicamente.  Dado que el estado de ocupación y la temperatura de la unidad se pueden actualizar dinámicamente, usaremos este `id` de característica en la sección siguiente. En el ejemplo siguiente, el `id` de la característica es "UNIT26". Haremos referencia a las propiedades de estilo de esta característica a modo de estados y usaremos la característica para crear un conjunto de estados.

     ```json
    {
        "type": "FeatureCollection",
        "features": [
            {
                "type": "Feature",
                "geometry": {
                    "type": "Polygon",
                    "coordinates": ["..."]
                },
                "properties": {
                    "original_id": "b7410920-8cb0-490b-ab23-b489fd35aed0",
                    "category_id": "CTG8",
                    "is_open_area": true,
                    "navigable_by": [
                        "pedestrian"
                    ],
                    "route_through_behavior": "allowed",
                    "level_id": "LVL14",
                    "occupants": [],
                    "address_id": "DIR1",
                    "name": "157"
                },
                "id": "UNIT26",
                "featureType": ""
            }, {"..."}
        ]
    }
    ```

## <a name="create-a-feature-stateset"></a>Creación de un conjunto de estados de características

1. En la aplicación Postman, seleccione**New** (Nuevo). En la ventana **Create New** (Crear nuevo), seleccione **Request** (Solicitud). Escriba un **nombre de solicitud** y seleccione una colección. Haga clic en **Guardar**

2. Realice una solicitud **POST** en [Create Stateset API](https://docs.microsoft.com/rest/api/maps/featurestate/createstatesetpreview). Use el `datasetId` del conjunto de datos que contiene el estado que quiere modificar. La solicitud debe tener un aspecto similar a la siguiente dirección URL:

    ```http
    https://atlas.microsoft.com/featureState/stateset?api-version=1.0&datasetId={datasetId}&subscription-key={Azure-Maps-Primary-Subscription-key}
    ```

3. En el **encabezado** de la solicitud **POST**, establezca `Content-Type` en `application/json`. En el **cuerpo**, proporcione los siguientes estilos para reflejar los cambios en los *estados* `occupied` y `temperature`. Cuando haya terminado, haga clic en **Send** (Enviar).

    ```json
    {
       "styles":[
          {
             "keyname":"occupied",
             "type":"boolean",
             "rules":[
                {
                   "true":"#FF0000",
                   "false":"#00FF00"
                }
             ]
          },
          {
             "keyname":"temperature",
             "type":"number",
             "rules":[
                {
                   "range":{
                      "exclusiveMaximum":66
                   },
                   "color":"#00204e"
                },
                {
                   "range":{
                      "minimum":66,
                      "exclusiveMaximum":70
                   },
                   "color":"#0278da"
                },
                {
                   "range":{
                      "minimum":70,
                      "exclusiveMaximum":74
                   },
                   "color":"#187d1d"
                },
                {
                   "range":{
                      "minimum":74,
                      "exclusiveMaximum":78
                   },
                   "color":"#fef200"
                },
                {
                   "range":{
                      "minimum":78,
                      "exclusiveMaximum":82
                   },
                   "color":"#fe8c01"
                },
                {
                   "range":{
                      "minimum":82
                   },
                   "color":"#e71123"
                }
             ]
          }
       ]
    }
    ```

4. Copie el valor de `statesetId` del cuerpo de la respuesta.

5. Cree una solicitud **POST** para actualizar el estado: Pase el valor de statesetId y la característica `ID` con la clave de suscripción de Azure Maps. La solicitud debe tener un aspecto similar a la siguiente dirección URL:

    ```http
    https://atlas.microsoft.com/featureState/state?api-version=1.0&statesetID={statesetId}&featureID={featureId}&subscription-key={Azure-Maps-Primary-Subscription-key}
    ```

6. En el **encabezado** de la solicitud **POST**, establezca `Content-Type` en `application/json`. En el **CUERPO** de la solicitud **POST**, copie y pegue el archivo JSON del ejemplo siguiente.

    ```json
    {
        "states": [
            {
                "keyName": "occupied",
                "value": true,
                "eventTimestamp": "2019-11-14T17:10:20"
            }
        ]
    }
    ```

    >[!NOTE]
    > La actualización solo se guardará si la marca de tiempo registrada es posterior a la marca de tiempo de la solicitud anterior. Podemos pasar cualquier nombre de clave que haya configurado anteriormente durante la creación.

7. Tras una actualización correcta, recibirá un código de estado HTTP `200 OK`. Si tiene un [estilo dinámico implementado](indoor-map-dynamic-styling.md) en un plano interior, la actualización se mostrará en el plano representado en la marca de tiempo especificada.

[Feature Get States API](https://docs.microsoft.com/rest/api/maps/featurestate/getstatespreview) permite recuperar el estado de una característica mediante el `ID` de la característica. También puede eliminar el conjunto de estados y sus recursos mediante [Feature State Delete API](https://docs.microsoft.com/rest/api/maps/featurestate/deletestatesetpreview).

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha aprendido a:

> [!div class="checklist"]
> * Cargar el paquete de dibujo del plano interior.
> * Convertir el paquete de dibujo en datos del plano.
> * Crear un conjunto de datos a partir de los datos del plano.
> * Crear un conjunto de mosaicos a partir de los datos del conjunto de datos.
> * Consultar el servicio WFS de Azure Maps para obtener información acerca de las características del plano.
> * Crear un conjunto de estados de características mediante las características del plano y los datos del conjunto de datos.
> * Actualizar el conjunto de estados de características.

Ya cuenta con los conocimientos que necesita para pasar a las siguientes guías:

> [!div class="nextstepaction"]
> [Uso del módulo de planos interiores](how-to-use-indoor-module.md)

> [!div class="nextstepaction"]
> [Implementación de un estilo dinámico para planos interiores](indoor-map-dynamic-styling.md)

Obtenga más información sobre los diferentes servicios de Azure Maps que se describen en este artículo:

> [!div class="nextstepaction"]
> [Data Upload](creator-indoor-maps.md#upload-a-drawing-package)

> [!div class="nextstepaction"]
> [Conversión de datos](creator-indoor-maps.md#convert-a-drawing-package)

> [!div class="nextstepaction"]
> [Dataset](creator-indoor-maps.md#datasets)

> [!div class="nextstepaction"]
> [Conjunto de mosaicos](creator-indoor-maps.md#tilesets)

> [!div class="nextstepaction"]
> [Conjunto de estados de características](creator-indoor-maps.md#feature-statesets)

> [!div class="nextstepaction"]
> [Servicio WFS](creator-indoor-maps.md#web-feature-service-api)
