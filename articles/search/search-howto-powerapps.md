---
title: 'Tutorial: Consulta desde Power Apps'
titleSuffix: Azure Cognitive Search
description: Instrucciones paso a paso sobre cómo compilar una aplicación de Power Apps que se conecta a un índice de Azure Cognitive Search, envía consultas y representa los resultados.
author: luiscabrer
manager: eladz
ms.author: luisca
ms.service: cognitive-search
ms.devlang: rest-api
ms.topic: tutorial
ms.date: 08/21/2020
ms.openlocfilehash: fd74bfca73323209012dfd1fda61bbaada84092f
ms.sourcegitcommit: 03662d76a816e98cfc85462cbe9705f6890ed638
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/15/2020
ms.locfileid: "90530699"
---
# <a name="tutorial-query-a-cognitive-search-index-from-power-apps"></a>Tutorial: Consulta de un índice de Cognitive Search desde Power Apps

Aproveche el entorno de desarrollo rápido de aplicaciones de Power Apps para crear una aplicación personalizada para el contenido que se puede buscar en Azure Cognitive Search.

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Conexión a Azure Cognitive Search
> * Configurar una solicitud de consulta
> * Visualizar los resultados en una aplicación de lienzo

Si no tiene una suscripción a Azure, abra una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="prerequisites"></a>Prerrequisitos

* [Cuenta de Power Apps](https://make.powerapps.com)

* [Índice hotels-sample](search-get-started-portal.md)

* [Clave de API de consulta](search-security-api-keys.md#find-existing-keys)

## <a name="1---create-a-custom-connector"></a>1\. Creación de un conector personalizado

Un conector de Power Apps es una conexión de origen de datos. En este paso, creará un conector personalizado para conectarse a un índice de búsqueda en la nube.

1. [Inicie sesión](https://make.powerapps.com) en Power Apps.

1. A la izquierda, expanda **Datos** > **Conectores personalizados**.
 
    :::image type="content" source="./media/search-howto-powerapps/1-2-custom-connector.png" alt-text="Menú con Conectores personalizados" border="true":::

1. Haga clic en **+ Nuevo conector personalizado** y, después, seleccione **Crear desde cero**.

    :::image type="content" source="./media/search-howto-powerapps/1-3-create-blank.png" alt-text="Menú Crear desde cero" border="true":::

1. Asigne un nombre al conector personalizado (por ejemplo, *AzureSearchQuery*) y, luego, haga clic en **Continuar**.

1. Escriba la información en la página General.

   * Color de fondo del icono (por ejemplo, #007ee5).
   * Descripción (por ejemplo, "Conector a Azure Cognitive Search").
   * En el host, tendrá que escribir la dirección URL del servicio de búsqueda (por ejemplo, `<yourservicename>.search.windows.net`).
   * Para la URL base, simplemente escriba "/".

    :::image type="content" source="./media/search-howto-powerapps/1-5-general-info.png" alt-text="Cuadro de diálogo Información general" border="true":::

1. En la página de seguridad, establezca *Clave de API* como **Tipo de autenticación** y use *api-key* como nombre y etiqueta del parámetro. En el campo **Ubicación de parámetro**, seleccione *Encabezado*, tal y como se muestra a continuación.

    :::image type="content" source="./media/search-howto-powerapps/1-6-authentication-type.png" alt-text="Opción de tipo de autenticación" border="true":::

1. En la página de definiciones, seleccione **+ Nueva acción** para crear una acción que consultará el índice. Escriba el valor "Consulta" en el campo de resumen y el nombre del identificador de operación. Escriba una descripción, como "*Consulta el índice de búsqueda*".

    :::image type="content" source="./media/search-howto-powerapps/1-7-new-action.png" alt-text="Opciones de la nueva acción" border="true":::

1. Desplácese hacia abajo. En Solicitudes, seleccione el botón **+ Importar desde ejemplo** para configurar una solicitud de consulta para el servicio de búsqueda:

   * Seleccione el verbo `GET`.

   * Como dirección URL, escriba una consulta de ejemplo para el índice de búsqueda (`search=*` devuelve todos los documentos, `$select=` le permite elegir campos). La versión de API es un dato obligatorio. Una dirección URL completamente especificada tendrá este aspecto: `https://mydemo.search.windows.net/indexes/hotels-sample-index/docs?search=*&$select=HotelName,Description,Address/City&api-version=2020-06-30`.

   * En Encabezados, escriba `Content-Type`. 

     **Power Apps** usará la sintaxis para extraer los parámetros de la consulta. Observe que hemos definido explícitamente el campo de búsqueda. 

       :::image type="content" source="./media/search-howto-powerapps/1-8-1-import-from-sample.png" alt-text="Importar desde ejemplo" border="true":::

1. Haga clic en **Importar** para rellenar automáticamente la solicitud. Para completar la configuración de los metadatos de parámetros, haga clic en el símbolo **...** , junto a cada uno de los parámetros. Haga clic en **Atrás** para volver a la página de solicitud después de la actualización de cada parámetro.

   :::image type="content" source="./media/search-howto-powerapps/1-8-2-import-from-sample.png" alt-text="Cuadro de diálogo Importar desde ejemplo" border="true":::

1. Parámetro *search*: Establezca `*` como **valor predeterminado**, establezca **Requerido** en *No* y establezca **Visibilidad** en *ninguna*. 

    :::image type="content" source="./media/search-howto-powerapps/1-10-1-parameter-metadata-search.png" alt-text="Metadatos del parámetro de búsqueda" border="true":::

1. En *Seleccionar*: Establezca `HotelName,Description,Address/City` como **valor predeterminado**, establezca **Requerido** en *No* y establezca **Visibilidad** en *ninguna*.  

    :::image type="content" source="./media/search-howto-powerapps/1-10-4-parameter-metadata-select.png" alt-text="Selección de metadatos del parámetro" border="true":::

1. Parámetro *api-version*: Establezca `2020-06-30` como **valor predeterminado**, establezca **Requerido** en *Sí* y establezca **Visibilidad** en *interna*.  

    :::image type="content" source="./media/search-howto-powerapps/1-10-2-parameter-metadata-version.png" alt-text="Metadatos del parámetro de versión" border="true":::

1. En *Content-Type*: Establézcalo en `application/json`.

1. Después de realizar estos cambios, cambie a la vista **Editor de Swagger**. En la sección de parámetros debería ver la siguiente configuración:

    ```JSON
    parameters:
      - {name: search, in: query, required: false, type: string, default: '*'}
      - {name: $select, in: query, required: false, type: string, default: 'HotelName,Description,Address/City'}
      - {name: api-version, in: query, required: true, type: string, default: '2020-06-30',
        x-ms-visibility: internal}
      - {name: Content-Type, in: header, required: false, type: string}
    ```

1. Vuelva al paso **3. Solicitud** y desplácese hacia abajo hasta la sección Respuesta. Elija **"Agregar respuesta predeterminada"** . Esta acción es fundamental porque ayudará a Power Apps a comprender el esquema de la respuesta. 

1. Pegue una respuesta de ejemplo. Una manera fácil de capturar una respuesta de ejemplo es a través del explorador de búsqueda en Azure Portal. En el explorador de búsqueda, debe escribir la misma consulta que hizo para la solicitud, pero agregue **$top=2** para restringir los resultados a solo dos documentos: `search=*&$select=HotelName,Description,Address/City&$top=2`. 

   Power Apps solo necesita algunos resultados para detectar el esquema.

    ```JSON
    {
        "@odata.context": "https://mydemo.search.windows.net/indexes('hotels-sample-index')/$metadata#docs(*)",
        "value": [
            {
                "@search.score": 1,
                "HotelName": "Arcadia Resort & Restaurant",
                "Description": "The largest year-round resort in the area offering more of everything for your vacation – at the best value!  What can you enjoy while at the resort, aside from the mile-long sandy beaches of the lake? Check out our activities sure to excite both young and young-at-heart guests. We have it all, including being named “Property of the Year” and a “Top Ten Resort” by top publications.",
                "Address": {
                    "City": "Seattle"
                }
            },
            {
                "@search.score": 1,
                "HotelName": "Travel Resort",
                "Description": "The Best Gaming Resort in the area.  With elegant rooms & suites, pool, cabanas, spa, brewery & world-class gaming.  This is the best place to play, stay & dine.",
                "Address": {
                    "City": "Albuquerque"
                }
            }
        ]
    }
    ```

    > [!TIP] 
    > Existe un límite de caracteres para la respuesta JSON que puede escribir, por lo que es conveniente que la simplifique antes de pegarla. El esquema y el formato de la respuesta son más importantes que los propios valores. Por ejemplo, el campo Descripción se puede simplificar a solo la primera oración.

1. Haga clic en **Crear conector** en la parte superior derecha.

## <a name="2---test-the-connection"></a>2\. Prueba de la conexión

La primera vez que se crea el conector, debe volver a abrirlo desde la lista Conectores personalizados para probarlo. Más adelante, si realiza más actualizaciones, puede probarlo desde el asistente.

Para esta tarea, necesitará una [clave de API de consulta](search-security-api-keys.md#find-existing-keys). Cada vez que se crea una conexión, ya sea para una ejecución de prueba o la inclusión en una aplicación, el conector necesita la clave de API de consulta que se usa para conectarse a Azure Cognitive Search.

1. En el extremo izquierdo, haga clic en **Conectores personalizados**.

1. Busque el conector por su nombre (en este tutorial, es "AzureSearchQuery").

1. Selecciónelo, expanda la lista de acciones y seleccione **Ver propiedades**.

    :::image type="content" source="./media/search-howto-powerapps/1-11-1-test-connector.png" alt-text="Ver propiedades" border="true":::

1. Seleccione **Editar** en la parte superior derecha.

1. Seleccione **4. Prueba** para abrir la página de prueba.

1. En Operación de prueba, haga clic en **+ Nueva conexión**.

1. Escriba una clave de API de consulta. Esta es una consulta de Azure Cognitive Search para el acceso de solo lectura a un índice. Puede [encontrar la clave](search-security-api-keys.md#find-existing-keys) en Azure Portal. 

1. En Operaciones, haga clic en el botón **Operación de prueba**. Si la operación se realiza correctamente, se devuelve un estado 200 y, en el cuerpo de la respuesta, verá un código JSON que describe los resultados de la búsqueda.

    :::image type="content" source="./media/search-howto-powerapps/1-11-2-test-connector.png" alt-text="Respuesta JSON" border="true":::

## <a name="3---visualize-results"></a>3\. Visualización de los resultados

En este paso, creará una aplicación de Power Apps con un cuadro de búsqueda, un botón de búsqueda y un área de presentación de los resultados. La aplicación de Power Apps se conectará al conector personalizado que se acaba de crear para obtener los datos de Azure Search.

1. A la izquierda, expanda **Aplicaciones** >  **+ Nueva aplicación** > **Lienzo**.

    :::image type="content" source="./media/search-howto-powerapps/2-1-create-canvas.png" alt-text="Creación de una aplicación de lienzo" border="true":::

1. Seleccione el tipo de aplicación. En este tutorial, seleccionará **Aplicación vacía** con **Diseño de teléfono**. Aparecerá la aplicación **Power Apps Studio**.

1. Una vez en Studio, seleccione la pestaña **Orígenes de datos** y haga clic en el nuevo conector que acaba de crear. En nuestro caso, este conector se denomina *AzureSearchQuery*. Haga clic en **Agregar una conexión**.

   Escriba la clave de la API de consulta.

    :::image type="content" source="./media/search-howto-powerapps/2-3-connect-connector.png" alt-text="Conexión del conector" border="true":::

    Ahora *AzureSearchQuery* es un origen de datos que está disponible para su uso desde la aplicación.

1. En la pestaña **Insertar**, agregue algunos controles al lienzo.

    :::image type="content" source="./media/search-howto-powerapps/2-4-add-controls.png" alt-text="Inserción de controles" border="true":::

1. Inserte los siguientes elementos:

   * Una etiqueta de texto con el valor "Consulta:".
   * Un elemento de entrada de texto (llámelo *txtQuery* y establezca su valor predeterminado en "*").
   * Un botón con el texto "Buscar". 
   * Una galería vertical (llámela *galleryResults*).

    El lienzo debe tener un aspecto similar al siguiente:

    :::image type="content" source="./media/search-howto-powerapps/2-5-controls-layout.png" alt-text="Diseño de los controles" border="true":::

1. Para que el **botón Buscar** emita una consulta, pegue la siguiente acción en **OnSelect**:

    ```
    If(!IsBlank(txtQuery.Text),
        ClearCollect(azResult, AzureSearchQuery.Query({search: txtQuery.Text}).value))
    ```

   En la captura de pantalla siguiente se muestra la barra de fórmulas de la acción **OnSelect**.

    :::image type="content" source="./media/search-howto-powerapps/2-6-search-button-event.png" alt-text="Botón OnSelect" border="true":::

   Esta acción hará que el botón actualice una nueva colección denominada *azResult* con el resultado de la consulta de búsqueda; para ello, usará el texto del cuadro de texto *txtQuery* como el término de la consulta.

   > [!NOTE]
   > Siga estos pasos si recibe un error de sintaxis de la fórmula que indica que la función "ClearCollect" tiene algunas funciones que no son válidas:
   > 
   > * En primer lugar, asegúrese de que la referencia del conector sea correcta. Borre el nombre del conector y comience a escribir el nombre del suyo. IntelliSense sugerirá el conector y el verbo correctos.
   > 
   > * Si el error persiste, elimine el conector y vuelva a crearlo. Si hay varias instancias de un conector, es posible que la aplicación esté usando la incorrecta.
   > 

1. Vincule el control Vertical Gallery (Galería vertical) a la colección *azResult* que se creó al completar el paso anterior. 

   Seleccione el control de galería y realice las siguientes acciones en el panel de propiedades.

   * Establezca **Origen de datos** en *azResult*.
   * Seleccione un **Diseño** adecuado en función del tipo de datos del índice. En este caso, hemos usado el diseño *Título, subtítulo y cuerpo*.
   * **Edite los campos** y seleccione aquellos que quiera visualizar.

    Como hemos proporcionado un resultado de ejemplo al definir el conector, la aplicación reconoce los campos disponibles en el índice.
    
    :::image type="content" source="./media/search-howto-powerapps/2-7-gallery-select-fields.png" alt-text="Campos de la galería" border="true":::   
 
1. Presione **F5** para obtener una versión preliminar de la aplicación.  

    :::image type="content" source="./media/search-howto-powerapps/2-8-3-final.png" alt-text="Aplicación final" border="true":::    

<!--     Remember that the fields can be set to calculated values.

    For the example, setting using the *"Image, Title and Subtitle"* layout and specifying the *Image* function as the concatenation of the root path for the data and the file name (for instance, `"https://mystore.blob.core.windows.net/multilang/" & ThisItem.metadata_storage_name`) will produce the result below.

    :::image type="content" source="./media/search-howto-powerapps/2-8-2-final.png" alt-text="Final app" border="true":::         -->

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando trabaje con su propia suscripción, es una buena idea al final de un proyecto identificar si todavía se necesitan los recursos que ha creado. Los recursos que se dejan en ejecución pueden costarle mucho dinero. Puede eliminar los recursos de forma individual o eliminar el grupo de recursos para eliminar todo el conjunto de recursos.

Puede encontrar y administrar recursos en el portal, mediante el vínculo **Todos los recursos** o **Grupos de recursos** en el panel de navegación izquierdo.

Si está usando un servicio gratuito, recuerde que está limitado a tres índices, indexadores y orígenes de datos. Puede eliminar elementos individuales en el portal para mantenerse por debajo del límite.

## <a name="next-steps"></a>Pasos siguientes

Power Apps permite el desarrollo rápido de aplicaciones personalizadas. Ahora que sabe cómo conectarse a un índice de búsqueda, aprenda más sobre cómo crear una experiencia de visualización enriquecida en una aplicación personalizada de Power Apps.

> [!div class="nextstepaction"]
> [Catálogo de aprendizaje de Power Apps](/powerapps/learning-catalog/get-started)