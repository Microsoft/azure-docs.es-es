---
title: 'Inicio rápido: PowerShell y API de REST - Azure Search'
description: Crear, cargar y consultar un índice mediante PowerShell Invoke-RestMethod y la API de REST de Azure Search.
ms.date: 05/16/2019
author: heidisteen
manager: cgronlun
ms.author: heidist
services: search
ms.service: search
ms.devlang: rest-api
ms.topic: conceptual
ms.custom: seodec2018
ms.openlocfilehash: a82ee51a168a018a4df537c05d987974e775b6cc
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65795933"
---
# <a name="quickstart-create-an-azure-search-index-using-powershell"></a>Inicio rápido: Crear un índice de Azure Search con PowerShell
> [!div class="op_single_selector"]
> * [PowerShell (REST)](search-create-index-rest-api.md)
> * [C#](search-create-index-dotnet.md)
> * [Postman (REST)](search-fiddler.md)
> * [Python](search-get-started-python.md)
> * [Portal](search-create-index-portal.md)
> 

Este artículo le guiará a través del proceso de crear, cargar y consultar una búsqueda de Azure [índice](search-what-is-an-index.md) mediante PowerShell y la [las API de REST de Azure Search Service](https://docs.microsoft.com/rest/api/searchservice/). La definición del índice y el contenido de búsqueda se proporcionan en el cuerpo de solicitud como contenido JSON con formato correcto.

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar y luego [regístrese en Azure Search](search-create-service-portal.md).

## <a name="prerequisites"></a>Requisitos previos

En este inicio rápido se usan los siguientes servicios y herramientas. 

+ [Cree un servicio Azure Search](search-create-service-portal.md) o [busque un servicio existente](https://ms.portal.azure.com/#blade/HubsExtension/BrowseResourceBlade/resourceType/Microsoft.Search%2FsearchServices) en su suscripción actual. Puede usar un servicio gratuito para este inicio rápido. 

+ [PowerShell 5.1 o una versión posterior](https://github.com/PowerShell/PowerShell)con [Invoke-RestMethod](https://docs.microsoft.com/powershell/module/Microsoft.PowerShell.Utility/Invoke-RestMethod) para conocer los pasos secuenciales e interactivos.

## <a name="get-a-key-and-url"></a>Obtención de una clave y una dirección URL

Las llamadas de REST requieren la dirección URL del servicio y una clave de acceso en cada solicitud. Con ambos se crea un servicio de búsqueda, por lo que si ha agregado Azure Search a su suscripción, siga estos pasos para obtener la información necesaria:

1. [Inicie sesión en Azure Portal](https://portal.azure.com/) y en la página **Introducción** del servicio de búsqueda, obtenga la dirección URL. Un punto de conexión de ejemplo podría ser similar a `https://mydemo.search.windows.net`.

2. En **Configuración** > **Claves**, obtenga una clave de administrador para tener derechos completos en el servicio. Se proporcionan dos claves de administrador intercambiables para lograr la continuidad empresarial, por si necesitara sustituir una de ellas. Puede usar la clave principal o secundaria en las solicitudes para agregar, modificar y eliminar objetos.

![Obtención de una clave de acceso y un punto de conexión HTTP](media/search-fiddler/get-url-key.png "Get an HTTP endpoint and access key")

Todas las solicitudes requieren una clave de API en cada solicitud enviada al servicio. Tener una clave válida genera la confianza, solicitud a solicitud, entre la aplicación que envía la solicitud y el servicio que se encarga de ella.

## <a name="connect-to-azure-search"></a>Conexión con Azure Search

1. En PowerShell, cree un **$headers** objeto para almacenar el tipo de contenido y la clave de API. Reemplace la clave de API de administración (YOUR-ADMIN-API-KEY) con una clave que es válida para el servicio de búsqueda. Solo tiene que establecer este encabezado de una vez por la duración de la sesión, pero se agregará a cada solicitud. 

    ```powershell
    $headers = @{
    'api-key' = '<YOUR-ADMIN-API-KEY>'
    'Content-Type' = 'application/json' 
    'Accept' = 'application/json' }
    ```

2. Crear un **$url** índices de objeto que especifica el servicio de la colección. Reemplace el nombre del servicio (YOUR-SEARCH-SERVICE-NAME) por un servicio de búsqueda válida.

    ```powershell
    $url = "https://<YOUR-SEARCH-SERVICE-NAME>.search.windows.net/indexes?api-version=2019-05-06"
    ```

3. Ejecute **Invoke-RestMethod** para enviar una solicitud GET al servicio y comprobar la conexión. Agregar **ConvertTo-Json** para que pueda ver las respuestas enviadas desde el servicio.

    ```powershell
    Invoke-RestMethod -Uri $url -Headers $headers | ConvertTo-Json
    ```

   Si el servicio está vacío y no tiene índices, los resultados son similares al ejemplo siguiente. En caso contrario, verá una representación JSON de definiciones de índice.

    ```
    {
        "@odata.context":  "https://mydemo.search.windows.net/$metadata#indexes",
        "value":  [

                ]
    }
    ```

## <a name="1---create-an-index"></a>1 - Creación de un índice

A menos que se está usando el portal, debe existir un índice en el servicio antes de poder cargar los datos. Este paso define el índice y lo inserta en el servicio. El [API de REST de Create Index](https://docs.microsoft.com/rest/api/searchservice/create-index) se usa para este paso.

Elementos necesarios de un índice incluyen un nombre y una colección de campos. La colección de campos define la estructura de un *documento*. Cada campo tiene un nombre, tipo y los atributos que determinan cómo se utiliza (por ejemplo, si es texto que se pueden buscar, filtrar o recuperar resultados de búsqueda). Dentro de un índice, uno de los campos de tipo `Edm.String` debe designarse como el *clave* para la identidad del documento.

Este índice se denomina "powershell de hoteles" y tiene las definiciones de campo que aparece a continuación. Es un subconjunto de una mayor [índice de hoteles](https://github.com/Azure-Samples/azure-search-sample-data/blob/master/hotels/Hotels_IndexDefinition.JSON) utilizado en otros tutoriales. Lo hemos acortado en este inicio rápido para mayor brevedad.

1. Pegue este ejemplo en PowerShell para crear un **$body** objeto que contiene el esquema de índice.

    ```powershell
    $body = @"
    {
        "name": "hotels-powershell",  
        "fields": [
            {"name": "hotelId", "type": "Edm.String", "key": true, "searchable": false, "sortable": false, "facetable": false},
            {"name": "baseRate", "type": "Edm.Double"},
            {"name": "description", "type": "Edm.String", "filterable": false, "sortable": false, "facetable": false},
            {"name": "description_fr", "type": "Edm.String", "filterable": false, "sortable": false, "facetable": false, "analyzer": "fr.lucene"},
            {"name": "hotelName", "type": "Edm.String", "facetable": false},
            {"name": "category", "type": "Edm.String"},
            {"name": "tags", "type": "Collection(Edm.String)"},
            {"name": "parkingIncluded", "type": "Edm.Boolean", "sortable": false},
            {"name": "smokingAllowed", "type": "Edm.Boolean", "sortable": false},
            {"name": "lastRenovationDate", "type": "Edm.DateTimeOffset"},
            {"name": "rating", "type": "Edm.Int32"},
            {"name": "location", "type": "Edm.GeographyPoint"}
        ]
    }
    "@
    ```

2. Establecer el URI en la colección de índices en el servicio y el *hoteles powershell* índice.

    ```powershell
    $url = "https://<YOUR-SEARCH-SERVICE>.search.windows.net/indexes/hotels-powershell?api-version=2019-05-06"
    ```

3. Ejecute el comando con **$url**, **$headers**, y **$body** para crear el índice en el servicio. 

    ```powershell
    Invoke-RestMethod -Uri $url -Headers $headers -Method Put -Body $body | ConvertTo-Json
    ```

    Resultado debería ser similar al siguiente (truncada para los dos primeros campos por razones de brevedad):

    ```
    {
        "@odata.context":  "https://mydemo.search.windows.net/$metadata#indexes/$entity",
        "@odata.etag":  "\"0x8D6A99E2DED96B0\"",
        "name":  "hotels-powershell",
        "defaultScoringProfile":  null,
        "fields":  [
                    {
                        "name":  "hotelId",
                        "type":  "Edm.String",
                        "searchable":  false,
                        "filterable":  true,
                        "retrievable":  true,
                        "sortable":  false,
                        "facetable":  false,
                        "key":  true,
                        "indexAnalyzer":  null,
                        "searchAnalyzer":  null,
                        "analyzer":  null,
                        "synonymMaps":  ""
                    },
                    {
                        "name":  "baseRate",
                        "type":  "Edm.Double",
                        "searchable":  false,
                        "filterable":  true,
                        "retrievable":  true,
                        "sortable":  true,
                        "facetable":  true,
                        "key":  false,
                        "indexAnalyzer":  null,
                        "searchAnalyzer":  null,
                        "analyzer":  null,
                        "synonymMaps":  ""
                    },
    . . .
    ```

> [!Tip]
> Para la comprobación, podría también comprobar la lista de índices en el portal o vuelva a ejecutar el comando que se usa para comprobar la conexión de servicio para ver el *hoteles powershell* índice aparece en la colección de índices.

<a name="load-documents"></a>

## <a name="2---load-documents"></a>2 - Carga de documentos

Para insertar los documentos, use una solicitud HTTP POST al punto de conexión de dirección URL del índice. La API de REST para esta tarea es [agregar, actualizar o eliminar documentos](https://docs.microsoft.com/rest/api/searchservice/addupdate-or-delete-documents).

1. Pegue este ejemplo en PowerShell para crear un **$body** objeto que contiene los documentos que desea cargar. 

    Esta solicitud incluye dos completo y un registro parcial. El registro parcial muestra que puede cargar documentos incompletos. El `@search.action` parámetro especifica cómo se realiza la indexación. Los valores válidos incluyen cargar, combinar, mergeOrUpload y delete. El comportamiento de mergeOrUpload crea un nuevo documento para hotelId = 3 o actualiza el contenido si ya existe.

    ```powershell
    $body = @"
    {
        "value": [
            {
                "@search.action": "upload",
                "hotelId": "1",
                "baseRate": 199.0,
                "description": "Best hotel in town",
                "hotelName": "Fancy Stay",
                "category": "Luxury",
                "tags": ["pool", "view", "wifi", "concierge"],
                "parkingIncluded": false,
                "smokingAllowed": false,
                "lastRenovationDate": "2010-06-27T00:00:00Z",
                "rating": 5,
                "location": { "type": "Point", "coordinates": [-122.131577, 47.678581] }
            },
            {
                "@search.action": "upload",
                "hotelId": "2",
                "baseRate": 79.99,
                "description": "Cheapest hotel in town",
                "hotelName": "Roach Motel",
                "category": "Budget",
                "tags": ["motel", "budget"],
                "parkingIncluded": true,
                "smokingAllowed": true,
                "lastRenovationDate": "1982-04-28T00:00:00Z",
                "rating": 1,
                "location": { "type": "Point", "coordinates": [-122.131577, 49.678581] }
            },
            {
                "@search.action": "mergeOrUpload",
                "hotelId": "3",
                "baseRate": 129.99,
                "description": "Close to town hall and the river"
            }
        ]
    }
    "@
    ```

1. Establece el punto de conexión en el *hoteles powershell* colección documentos e incluir la operación de índice (índices/hoteles-powershell/docs/index).

    ```powershell
    $url = "https://<YOUR-SEARCH-SERVICE>.search.windows.net/indexes/hotels-powershell/docs/index?api-version=2019-05-06"
    ```

1. Ejecute el comando con **$url**, **$headers**, y **$body** para cargar documentos en el índice de hoteles powershell.

    ```powershell
    Invoke-RestMethod -Uri $url -Headers $headers -Method Post -Body $body | ConvertTo-Json
    ```
    Resultados deben ser similares al ejemplo siguiente. Debería ver un código de estado de 201. Para obtener una descripción de todos los códigos de estado, vea [códigos de estado HTTP (Azure Search)](https://docs.microsoft.com/rest/api/searchservice/HTTP-status-codes).

    ```
    {
        "@odata.context":  "https://mydemo.search.windows.net/indexes/hotels-powershell/$metadata#Collection(Microsoft.Azure.Search.V2017_11_11.IndexResult)",
        "value":  [
                    {
                        "key":  "1",
                        "status":  true,
                        "errorMessage":  null,
                        "statusCode":  201
                    },
                    {
                        "key":  "2",
                        "status":  true,
                        "errorMessage":  null,
                        "statusCode":  201
                    },
                    {
                        "key":  "3",
                        "status":  true,
                        "errorMessage":  null,
                        "statusCode":  201
                    }
                ]
    }
    ```

## <a name="3---search-an-index"></a>3 - Búsqueda en un índice

Este paso muestra cómo consultar un índice mediante el [API de búsqueda de documentos](https://docs.microsoft.com/rest/api/searchservice/search-documents).

1. Establece el punto de conexión en el *hoteles powershell* colección documentos y agregue un **búsqueda** parámetro para incluir las cadenas de consulta. Esta cadena es una búsqueda vacía y devuelve una lista de todos los documentos unranked.

    ```powershell
    $url = 'https://<YOUR-SEARCH-SERVICE>.search.windows.net/indexes/hotels-powershell/docs?api-version=2019-05-06&search=*'
    ```

1. Ejecute el comando para enviar el **$url** al servicio.

    ```powershell
    Invoke-RestMethod -Uri $url -Headers $headers | ConvertTo-Json
    ```

    Resultados deben ser similares al siguiente resultado.

    ```
    {
        "@odata.context":  "https://mydemo.search.windows.net/indexes/hotels-powershell/$metadata#docs(*)",
        "value":  [
                    {
                        "@search.score":  1.0,
                        "hotelId":  "1",
                        "baseRate":  199.0,
                        "description":  "Best hotel in town",
                        "description_fr":  null,
                        "hotelName":  "Fancy Stay",
                        "category":  "Luxury",
                        "tags":  "pool view wifi concierge",
                        "parkingIncluded":  false,
                        "smokingAllowed":  false,
                        "lastRenovationDate":  "2010-06-27T00:00:00Z",
                        "rating":  5,
                        "location":  "@{type=Point; coordinates=System.Object[]; crs=}"
                    },
                    {
                        "@search.score":  1.0,
                        "hotelId":  "2",
                        "baseRate":  79.99,
                        "description":  "Cheapest hotel in town",
                        "description_fr":  null,
                        "hotelName":  "Roach Motel",
                        "category":  "Budget",
                        "tags":  "motel budget",
                        "parkingIncluded":  true,
                        "smokingAllowed":  true,
                        "lastRenovationDate":  "1982-04-28T00:00:00Z",
                        "rating":  1,
                        "location":  "@{type=Point; coordinates=System.Object[]; crs=}"
                    },
                    {
                        "@search.score":  1.0,
                        "hotelId":  "3",
                        "baseRate":  129.99,
                        "description":  "Close to town hall and the river",
                        "description_fr":  null,
                        "hotelName":  null,
                        "category":  null,
                        "tags":  "",
                        "parkingIncluded":  null,
                        "smokingAllowed":  null,
                        "lastRenovationDate":  null,
                        "rating":  null,
                        "location":  null
                    }
                ]
    }
    ```

Pruebe algunos otros ejemplos de consultas para hacerse una idea de la sintaxis. Puede realizar una búsqueda de cadena, las consultas $filter textual, limitar el conjunto de resultados, el ámbito de la búsqueda a campos específicos y mucho más.

```powershell
# Query example 1
# Search the entire index for the term 'budget'
# Return only the `hotelName` field, "Roach hotel"
$url = 'https://<YOUR-SEARCH-SERVICE>.search.windows.net/indexes/hotels-powershell/docs?api-version=2019-05-06&search=budget&$select=hotelName'

# Query example 2 
# Apply a filter to the index to find hotels cheaper than $150 per night
# Returns the `hotelId` and `description`. Two documents match.
$url = 'https://<YOUR-SEARCH-SERVICE>.search.windows.net/indexes/hotels-powershell/docs?api-version=2019-05-06&search=*&$filter=baseRate lt 150&$select=hotelId,description'

# Query example 3
# Search the entire index, order by a specific field (`lastRenovationDate`) in descending order
# Take the top two results, and show only `hotelName` and `lastRenovationDate`
$url = 'https://<YOUR-SEARCH-SERVICE>.search.windows.net/indexes/hotels-powershell/docs?api-version=2019-05-06&search=*&$top=2&$orderby=lastRenovationDate desc&$select=hotelName,lastRenovationDate'
```
## <a name="clean-up"></a>Limpieza 

Debe eliminar el índice si ya no lo necesita. Un servicio gratuito está limitado a tres índices. Es posible que desee eliminar cualquier índice que no está utilizando activamente para que se puede ejecutar paso a paso a través de otros tutoriales.

```powershell
# Set the URI to the hotel index
$url = 'https://mydemo.search.windows.net/indexes/hotels-powershell?api-version=2019-05-06'

# Delete the index
Invoke-RestMethod -Uri $url -Headers $headers -Method Delete
```

## <a name="next-steps"></a>Pasos siguientes

Intente agregar descripciones de francés para el índice. El ejemplo siguiente incluye cadenas en francés y muestra las acciones de búsqueda adicionales. Use mergeOrUpload para crear o agregar campos existentes. Las siguientes cadenas deben estar codificado con UTF-8.

```json
{
    "value": [
        {
            "@search.action": "mergeOrUpload",
            "hotelId": "1",
            "description_fr": "Meilleur hôtel en ville"
        },
        {
            "@search.action": "merge",
            "hotelId": "2",
            "description_fr": "Hôtel le moins cher en ville"
        }
    ]
}
```
