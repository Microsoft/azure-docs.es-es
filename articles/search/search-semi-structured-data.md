---
title: 'Tutorial: Indexación de datos semiestructurados en blobs JSON'
titleSuffix: Azure Cognitive Search
description: Aprenda a indexar y realizar búsquedas en blobs JSON de Azure semiestructurados mediante las API REST de Azure Cognitive Search y Postman.
manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.service: cognitive-search
ms.topic: tutorial
ms.date: 06/23/2020
ms.openlocfilehash: 8a615dc02b78993a18a86def9d8f496ba0bba922
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88929710"
---
# <a name="tutorial-index-json-blobs-from-azure-storage-using-rest"></a>Tutorial: Indexación de blobs JSON de Azure Storage con REST

Azure Cognitive Search puede indexar documentos y matrices JSON en Azure Blob Storage mediante un [indexador](search-indexer-overview.md) que sabe cómo leer datos semiestructurados. Los datos semiestructurados contienen etiquetas o marcas que separan el contenido dentro de los datos. Dividen la diferencia entre datos no estructurados, que se deben indexar completamente, y datos estructurados formalmente que se ajustan a un modelo de datos, como un esquema de base de datos relacional, que se pueden indexar por campo.

En este tutorial se usa Postman y las [API REST de Search](/rest/api/searchservice/) para realizar las siguientes tareas:

> [!div class="checklist"]
> * Configurar un origen de datos de Azure Cognitive Search para un contenedor de blobs de Azure
> * Crear un índice de Azure Cognitive Search que contenga el contenido que permite búsquedas
> * Configurar y ejecutar un indexador para leer el contenedor y extraer contenido que permite búsquedas de Azure Blob Storage
> * Buscar el índice que acaba de crear.

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="prerequisites"></a>Prerrequisitos

+ [Almacenamiento de Azure](../storage/common/storage-account-create.md)
+ [Aplicación de escritorio Postman](https://www.getpostman.com/)
+ [Creación](search-create-service-portal.md) o [búsqueda de un servicio de búsqueda existente](https://ms.portal.azure.com/#blade/HubsExtension/BrowseResourceBlade/resourceType/Microsoft.Search%2FsearchServices) 

> [!Note]
> Puede usar un servicio gratuito para este tutorial. Un servicio de búsqueda gratuito tiene una limitación de tres índices, tres indexadores y tres orígenes de datos. En este tutorial se crea uno de cada uno. Antes de empezar, asegúrese de que haya espacio en el servicio para aceptar los nuevos recursos.

## <a name="download-files"></a>Descarga de archivos

[Clínical-trials-json.zip](https://github.com/Azure-Samples/storage-blob-integration-with-cdn-search-hdi/raw/master/clinical-trials-json.zip) contiene los datos que se usan en este tutorial. Descargue y descomprima este archivo en su propia carpeta. Los datos proceden de [clinicaltrials.gov](https://clinicaltrials.gov/ct2/results), convertido a JSON para los fines de este tutorial.

## <a name="1---create-services"></a>1: Creación de servicios

En este tutorial se usa Azure Cognitive Search en la indexación y las consultas y Azure Blob Storage en la provisión de datos. 

Si es posible, cree los dos en la misma región y grupo de recursos para la proximidad y la capacidad de administración. En la práctica, la cuenta de Azure Storage puede estar en cualquier región.

### <a name="start-with-azure-storage"></a>Comienzo con Azure Storage

1. [Inicie sesión en Azure Portal](https://portal.azure.com/) y haga clic en **+ Crear un recurso**.

1. Busque *cuenta de almacenamiento* y seleccione la oferta de Cuenta de almacenamiento de Microsoft.

   ![Creación de una cuenta de almacenamiento](media/cognitive-search-tutorial-blob/storage-account.png "Crear cuenta de almacenamiento")

1. En la pestaña Datos básicos, se necesitan los siguientes elementos. Acepte los valores predeterminados para todo lo demás.

   + **Grupo de recursos**. Seleccione un grupo existente o cree uno nuevo, pero use el mismo grupo para todos los servicios, con el fin de que pueda administrarlos colectivamente.

   + **Nombre de cuenta de almacenamiento**. Si cree que puede tener varios recursos del mismo tipo, use el nombre para diferenciarlos por tipo y región, por ejemplo *blobstoragewestus*. 

   + **Ubicación**. Si es posible, elija la misma ubicación que se usa para Azure Cognitive Search y Cognitive Services. Una ubicación única anula los cargos de ancho de banda.

   + **Tipo de cuenta**. Elija el valor predeterminado, *StorageV2 (uso general v2)* .

1. Haga clic en **Revisar y crear** para crear el servicio.

1. Una vez creado, haga clic en **Go to the resource** (Ir al recurso) para abrir la página de información general.

1. Haga clic en el servicio **Blobs**.

1. [Cree un contenedor de blobs](../storage/blobs/storage-quickstart-blobs-portal.md) que contenga los datos de ejemplo. Puede establecer el nivel de acceso público a cualquiera de sus valores válidos.

1. Una vez creado el contenedor, ábralo y seleccione **Cargar** en la barra de comandos.

   ![Carga en la barra de comandos](media/search-semi-structured-data/upload-command-bar.png "Carga en la barra de comandos")

1. Vaya a la carpeta que contiene los archivos de ejemplo. Selecciónelos todos ellos y haga clic en **Cargar**.

   ![Carga de archivos](media/search-semi-structured-data/clinicalupload.png "Carga de archivos")

Una vez finalizada la carga, los archivos deberían aparecer en su propia subcarpeta dentro del contenedor de datos.

### <a name="azure-cognitive-search"></a>Azure Cognitive Search

El siguiente recurso es Azure Cognitive Search, que se puede [crear en el portal](search-create-service-portal.md). Puede usar el nivel Gratis para completar este tutorial. 

Al igual que con Azure Blob Storage dedique un momento a recopilar la clave de acceso. Además, cuando empiece a estructurar las solicitudes, deberá proporcionar el punto de conexión y la clave de la API de administración que se usan para autenticar cada solicitud.

### <a name="get-a-key-and-url"></a>Obtención de una clave y una dirección URL

Las llamadas de REST requieren la dirección URL del servicio y una clave de acceso en cada solicitud. Con ambos se crea un servicio de búsqueda, por lo que, si ha agregado Azure Cognitive Search a su suscripción, siga estos pasos para obtener la información necesaria:

1. [Inicie sesión en Azure Portal](https://portal.azure.com/) y en la página **Introducción** del servicio de búsqueda, obtenga la dirección URL. Un punto de conexión de ejemplo podría ser similar a `https://mydemo.search.windows.net`.

1. En **Configuración** > **Claves**, obtenga una clave de administrador para tener derechos completos en el servicio. Se proporcionan dos claves de administrador intercambiables para lograr la continuidad empresarial, por si necesitara sustituir una de ellas. Puede usar la clave principal o secundaria en las solicitudes para agregar, modificar y eliminar objetos.

![Obtención de una clave de acceso y un punto de conexión HTTP](media/search-get-started-postman/get-url-key.png "Obtención de una clave de acceso y un punto de conexión HTTP")

Todas las solicitudes requieren una clave de API en cada solicitud enviada al servicio. Tener una clave válida genera la confianza, solicitud a solicitud, entre la aplicación que envía la solicitud y el servicio que se encarga de ella.

## <a name="2---set-up-postman"></a>2: Configuración de Postman

Inicie Postman y configure una solicitud HTTP. Si no está familiarizado con esta herramienta, consulte [Exploración de las API REST de Azure Cognitive Search mediante Postman](search-get-started-postman.md).

Los métodos de solicitud para cada llamada de este tutorial son **POST** y **GET**. Realizará tres llamadas API al servicio de búsqueda con el fin de crear un origen de datos, un índice y un indexador. El origen de datos incluye un puntero a la cuenta de almacenamiento y a los datos JSON. El servicio de búsqueda realiza la conexión al cargar los datos.

En Headers (Encabezados), establezca "Content-Type" en `application/json` y establezca `api-key` en la clave de API de administración de su servicio Azure Cognitive Search. Una vez que establezca los encabezados, puede usarlos para todas las solicitudes de este ejercicio.

  ![Encabezado y dirección URL de solicitud de Postman](media/search-get-started-postman/postman-url.png "Encabezado y dirección URL de solicitud de Postman")

Los identificadores URI deben especificar un elemento api-version. Además, cada llamada debe devolver el mensaje **201 - Creado**. El elemento api-version disponible con carácter general para el uso de matrices JSON es `2020-06-30`.

## <a name="3---create-a-data-source"></a>3: Creación de un origen de datos

[Create Data Source API](/rest/api/searchservice/create-data-source) crea un objeto de Azure Cognitive Search que especifica qué datos se indexan.

1. Establezca el punto de conexión de esta llamada en `https://[service name].search.windows.net/datasources?api-version=2020-06-30`. Reemplace `[service name]` por el nombre del servicio de búsqueda. 

1. Copie el siguiente archivo JSON en el cuerpo de la solicitud.

    ```json
    {
        "name" : "clinical-trials-json-ds",
        "type" : "azureblob",
        "credentials" : { "connectionString" : "DefaultEndpointsProtocol=https;AccountName=[storage account name];AccountKey=[storage account key];" },
        "container" : { "name" : "[blob container name]"}
    }
    ```

1. Reemplace la cadena de conexión por una cadena válida para su cuenta.

1. Reemplace "[blob container name]" por el contenedor que creó para los datos de ejemplo. 

1. Envíe la solicitud. La respuesta debería tener este aspecto:

    ```json
    {
        "@odata.context": "https://exampleurl.search.windows.net/$metadata#datasources/$entity",
        "@odata.etag": "\"0x8D505FBC3856C9E\"",
        "name": "clinical-trials-json-ds",
        "description": null,
        "type": "azureblob",
        "subtype": null,
        "credentials": {
            "connectionString": "DefaultEndpointsProtocol=https;AccountName=[mystorageaccounthere];AccountKey=[[myaccountkeyhere]]];"
        },
        "container": {
            "name": "[mycontainernamehere]",
            "query": null
        },
        "dataChangeDetectionPolicy": null,
        "dataDeletionDetectionPolicy": null
    }
    ```

## <a name="4---create-an-index"></a>4: Creación de un índice
    
La segunda llamada es a la [API Create Index](/rest/api/searchservice/create-index), que crea un índice de Azure Cognitive Search que almacena todos los datos en los que se pueden realizar búsquedas. Un índice especifica todos los parámetros y sus atributos.

1. Establezca el punto de conexión de esta llamada en `https://[service name].search.windows.net/indexes?api-version=2020-06-30`. Reemplace `[service name]` por el nombre del servicio de búsqueda.

1. Copie el siguiente archivo JSON en el cuerpo de la solicitud.

    ```json
    {
      "name": "clinical-trials-json-index",  
      "fields": [
      {"name": "FileName", "type": "Edm.String", "searchable": false, "retrievable": true, "facetable": false, "filterable": false, "sortable": true},
      {"name": "Description", "type": "Edm.String", "searchable": true, "retrievable": false, "facetable": false, "filterable": false, "sortable": false},
      {"name": "MinimumAge", "type": "Edm.Int32", "searchable": false, "retrievable": true, "facetable": true, "filterable": true, "sortable": true},
      {"name": "Title", "type": "Edm.String", "searchable": true, "retrievable": true, "facetable": false, "filterable": true, "sortable": true},
      {"name": "URL", "type": "Edm.String", "searchable": false, "retrievable": false, "facetable": false, "filterable": false, "sortable": false},
      {"name": "MyURL", "type": "Edm.String", "searchable": false, "retrievable": true, "facetable": false, "filterable": false, "sortable": false},
      {"name": "Gender", "type": "Edm.String", "searchable": false, "retrievable": true, "facetable": true, "filterable": true, "sortable": false},
      {"name": "MaximumAge", "type": "Edm.Int32", "searchable": false, "retrievable": true, "facetable": true, "filterable": true, "sortable": true},
      {"name": "Summary", "type": "Edm.String", "searchable": true, "retrievable": true, "facetable": false, "filterable": false, "sortable": false},
      {"name": "NCTID", "type": "Edm.String", "key": true, "searchable": true, "retrievable": true, "facetable": false, "filterable": true, "sortable": true},
      {"name": "Phase", "type": "Edm.String", "searchable": false, "retrievable": true, "facetable": true, "filterable": true, "sortable": false},
      {"name": "Date", "type": "Edm.String", "searchable": false, "retrievable": true, "facetable": false, "filterable": false, "sortable": true},
      {"name": "OverallStatus", "type": "Edm.String", "searchable": false, "retrievable": true, "facetable": true, "filterable": true, "sortable": false},
      {"name": "OrgStudyId", "type": "Edm.String", "searchable": true, "retrievable": true, "facetable": false, "filterable": true, "sortable": false},
      {"name": "HealthyVolunteers", "type": "Edm.String", "searchable": false, "retrievable": true, "facetable": true, "filterable": true, "sortable": false},
      {"name": "Keywords", "type": "Collection(Edm.String)", "searchable": true, "retrievable": true, "facetable": true, "filterable": false, "sortable": false},
      {"name": "metadata_storage_last_modified", "type":"Edm.DateTimeOffset", "searchable": false, "retrievable": true, "filterable": true, "sortable": false},
      {"name": "metadata_storage_size", "type":"Edm.String", "searchable": false, "retrievable": true, "filterable": true, "sortable": false},
      {"name": "metadata_content_type", "type":"Edm.String", "searchable": true, "retrievable": true, "filterable": true, "sortable": false}
      ]
    }
   ```

1. Envíe la solicitud. La respuesta debería tener este aspecto:

    ```json
    {
        "@odata.context": "https://exampleurl.search.windows.net/$metadata#indexes/$entity",
        "@odata.etag": "\"0x8D505FC00EDD5FA\"",
        "name": "clinical-trials-json-index",
        "fields": [
            {
                "name": "FileName",
                "type": "Edm.String",
                "searchable": false,
                "filterable": false,
                "retrievable": true,
                "sortable": true,
                "facetable": false,
                "key": false,
                "indexAnalyzer": null,
                "searchAnalyzer": null,
                "analyzer": null,
                "synonymMaps": []
            },
            {
                "name": "Description",
                "type": "Edm.String",
                "searchable": true,
                "filterable": false,
                "retrievable": false,
                "sortable": false,
                "facetable": false,
                "key": false,
                "indexAnalyzer": null,
                "searchAnalyzer": null,
                "analyzer": null,
                "synonymMaps": []
            },
            ...
          }
    ```

## <a name="5---create-and-run-an-indexer"></a>5: Creación y ejecución de un indexador

Un indexador se conecta al origen de datos, importa los datos en un índice de búsqueda de destino y proporciona opcionalmente una programación para automatizar la actualización de datos. La API REST es [Create Indexer](/rest/api/searchservice/create-indexer).

1. Establezca el URI para esta llamada en `https://[service name].search.windows.net/indexers?api-version=2020-06-30`. Reemplace `[service name]` por el nombre del servicio de búsqueda.

1. Copie el siguiente archivo JSON en el cuerpo de la solicitud.

    ```json
    {
      "name" : "clinical-trials-json-indexer",
      "dataSourceName" : "clinical-trials-json-ds",
      "targetIndexName" : "clinical-trials-json-index",
      "parameters" : { "configuration" : { "parsingMode" : "jsonArray" } }
    }
    ```

1. Envíe la solicitud. La solicitud se procesa inmediatamente. Cuando vuelva la respuesta, tendrá un índice de búsqueda de texto completo. La respuesta debería tener este aspecto:

    ```json
    {
        "@odata.context": "https://exampleurl.search.windows.net/$metadata#indexers/$entity",
        "@odata.etag": "\"0x8D505FDE143D164\"",
        "name": "clinical-trials-json-indexer",
        "description": null,
        "dataSourceName": "clinical-trials-json-ds",
        "targetIndexName": "clinical-trials-json-index",
        "schedule": null,
        "parameters": {
            "batchSize": null,
            "maxFailedItems": null,
            "maxFailedItemsPerBatch": null,
            "base64EncodeKeys": null,
            "configuration": {
                "parsingMode": "jsonArray"
            }
        },
        "fieldMappings": [],
        "enrichers": [],
        "disabled": null
    }
    ```

## <a name="6---search-your-json-files"></a>6: Búsqueda de los archivos JSON

Puede empezar a realizar búsquedas en cuanto se cargue el primer documento.

1. Cambie el verbo a **GET**.

1. Establezca el URI para esta llamada en `https://[service name].search.windows.net/indexes/clinical-trials-json-index/docs?search=*&api-version=2020-06-30&$count=true`. Reemplace `[service name]` por el nombre del servicio de búsqueda.

1. Envíe la solicitud. Se trata de una consulta de búsqueda de texto completo no especificada que devuelve todos los campos marcados como recuperables en el índice, junto con un recuento de documentos. La respuesta debería tener este aspecto:

    ```json
    {
        "@odata.context": "https://exampleurl.search.windows.net/indexes('clinical-trials-json-index')/$metadata#docs(*)",
        "@odata.count": 100,
        "value": [
            {
                "@search.score": 1.0,
                "FileName": "NCT00000102.txt",
                "MinimumAge": 14,
                "Title": "Congenital Adrenal Hyperplasia: Calcium Channels as Therapeutic Targets",
                "MyURL": "https://azure.storagedemos.com/clinical-trials/NCT00000102.txt",
                "Gender": "Both",
                "MaximumAge": 35,
                "Summary": "This study will test the ability of extended release nifedipine (Procardia XL), a blood pressure medication, to permit a decrease in the dose of glucocorticoid medication children take to treat congenital adrenal hyperplasia (CAH).",
                "NCTID": "NCT00000102",
                "Phase": "Phase 1/Phase 2",
                "Date": "ClinicalTrials.gov processed this data on October 25, 2016",
                "OverallStatus": "Completed",
                "OrgStudyId": "NCRR-M01RR01070-0506",
                "HealthyVolunteers": "No",
                "Keywords": [],
                "metadata_storage_last_modified": "2019-04-09T18:16:24Z",
                "metadata_storage_size": "33060",
                "metadata_content_type": null
            },
            . . . 
    ```

1. Agregue el parámetro de consulta `$select` para limitar los resultados a menos campos: `https://[service name].search.windows.net/indexes/clinical-trials-json-index/docs?search=*&$select=Gender,metadata_storage_size&api-version=2020-06-30&$count=true`.  En esta consulta, 100 documentos coinciden, pero de forma predeterminada, Azure Cognitive Search solo devuelve 50 en los resultados.

   ![Consulta con parámetros](media/search-semi-structured-data/lastquery.png "Consulta con parámetros")

1. Un ejemplo de consulta más compleja incluiría `$filter=MinimumAge ge 30 and MaximumAge lt 75`, que devuelve solo resultados en los que el parámetro MinimumAge es mayor o igual a 30 y el parámetro MaximumAge es menor que 75. Reemplace la expresión `$select` por la expresión `$filter`.

   ![Búsqueda de datos semiestructurados](media/search-semi-structured-data/metadatashort.png)

También puede usar operadores lógicos (and, or, not) y operadores de comparación (eq, ne, gt, lt, ge, le). La comparación de cadenas distingue mayúsculas de minúsculas. Para obtener más información y ejemplos, vea [crear una consulta simple](search-query-simple-examples.md).

> [!NOTE]
> El parámetro `$filter` solo funciona con los metadatos que se marcaron como filtrables al crear el índice.

## <a name="reset-and-rerun"></a>Restablecer y volver a ejecutar

En las primeras etapas experimentales de desarrollo, el enfoque más práctico para las iteraciones del diseño es eliminar los objetos de Azure Cognitive Search y permitir que el código vuelva a generarlos. Los nombres de los recursos son únicos. La eliminación de un objeto permite volver a crearlo con el mismo nombre.

Puede usar el portal para eliminar los índices, indexadores y orígenes de datos. También puede usar **DELETE** y proporcionar direcciones URL a cada objeto. El siguiente comando elimina un indexador.

```http
DELETE https://[YOUR-SERVICE-NAME].search.windows.net/indexers/clinical-trials-json-indexer?api-version=2020-06-30
```

Se devuelve el código de estado 204 si la eliminación se realiza correctamente.

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando trabaje con su propia suscripción, al final de un proyecto, es recomendable eliminar los recursos que ya no necesite. Los recursos que se dejan en ejecución pueden costarle mucho dinero. Puede eliminar los recursos de forma individual o eliminar el grupo de recursos para eliminar todo el conjunto de recursos.

Puede buscar y administrar los recursos en el portal, mediante el vínculo Todos los recursos o Grupos de recursos en el panel de navegación izquierdo.

## <a name="next-steps"></a>Pasos siguientes

Ahora que está familiarizado con los aspectos básicos de la indexación de blobs de Azure, podemos abordar en detalle la configuración del indexador para blobs JSON en Azure Storage.

> [!div class="nextstepaction"]
> [Configuración de la indexación de blobs JSON](search-howto-index-json-blobs.md)