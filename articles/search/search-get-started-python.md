---
title: 'Inicio rápido: Creación de un índice de búsqueda en Python mediante las API REST'
titleSuffix: Azure Cognitive Search
description: Se explica cómo crear un índice, cargar datos y ejecutar consultas mediante Python, Jupyter Notebook y la API REST de Azure Cognitive Search.
author: HeidiSteen
manager: nitinme
ms.author: heidist
ms.service: cognitive-search
ms.topic: quickstart
ms.devlang: rest-api
ms.date: 04/01/2020
ms.custom: devx-track-python
ms.openlocfilehash: 8e8f10b07d47f7aebd8482b8dae995bd50050162
ms.sourcegitcommit: 7fe8df79526a0067be4651ce6fa96fa9d4f21355
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87852215"
---
# <a name="quickstart-create-an-azure-cognitive-search-index-in-python-using-jupyter-notebooks"></a>Inicio rápido: Creación de un índice de Azure Cognitive Search en Python mediante Jupyter Notebook

> [!div class="op_single_selector"]
> * [Python (REST)](search-get-started-python.md)
> * [PowerShell (REST)](search-create-index-rest-api.md)
> * [C#](search-create-index-dotnet.md)
> * [Postman (REST)](search-get-started-postman.md)
> * [Portal](search-get-started-portal.md)
> 

Compile un cuaderno de Jupyter Notebook que cree, cargue y consulte un índice de Azure Cognitive Search mediante Python y las [API REST de Azure Cognitive Search](https://docs.microsoft.com/rest/api/searchservice/). En este artículo se explica cómo crear un cuaderno paso a paso. Como alternativa, puede [descargar y ejecutar un cuaderno de Jupyter Python finalizado](https://github.com/Azure-Samples/azure-search-python-samples).

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="prerequisites"></a>Prerrequisitos

Para este inicio rápido, se requieren los siguientes servicios y herramientas. 

+ [Anaconda 3.x](https://www.anaconda.com/distribution/#download-section), que proporciona Python 3.x e instancias de Jupyter Notebook.

+ [Cree un servicio Azure Cognitive Search](search-create-service-portal.md) o [busque uno existente](https://ms.portal.azure.com/#blade/HubsExtension/BrowseResourceBlade/resourceType/Microsoft.Search%2FsearchServices) en su suscripción actual. Puede usar el nivel Gratis para este inicio rápido. 

## <a name="get-a-key-and-url"></a>Obtención de una clave y una dirección URL

Las llamadas de REST requieren la dirección URL del servicio y una clave de acceso en cada solicitud. Con ambos se crea un servicio de búsqueda, por lo que, si ha agregado Azure Cognitive Search a su suscripción, siga estos pasos para obtener la información necesaria:

1. [Inicie sesión en Azure Portal](https://portal.azure.com/) y en la página **Introducción** del servicio de búsqueda, obtenga la dirección URL. Un punto de conexión de ejemplo podría ser similar a `https://mydemo.search.windows.net`.

1. En **Configuración** > **Claves**, obtenga una clave de administrador para tener derechos completos en el servicio. Se proporcionan dos claves de administrador intercambiables para lograr la continuidad empresarial, por si necesitara sustituir una de ellas. Puede usar la clave principal o secundaria en las solicitudes para agregar, modificar y eliminar objetos.

![Obtención de una clave de acceso y un punto de conexión HTTP](media/search-get-started-postman/get-url-key.png "Obtención de una clave de acceso y un punto de conexión HTTP")

Todas las solicitudes requieren una clave de API en cada solicitud enviada al servicio. Tener una clave válida genera la confianza, solicitud a solicitud, entre la aplicación que envía la solicitud y el servicio que se encarga de ella.

## <a name="connect-to-azure-cognitive-search"></a>Conexión a Azure Cognitive Search

En esta tarea, inicie un cuaderno de Jupyter Notebook y compruebe que puede conectarse a Azure Cognitive Search. Para ello, solicite una lista de índices de su servicio. En Windows con Anaconda3, puede utilizar el navegador de Anaconda para iniciar un cuaderno.

1. Cree un cuaderno de Python3.

1. En la primera celda, cargue las bibliotecas que usa para trabajar con JSON y formular las solicitudes HTTP.

   ```python
   import json
   import requests
   from pprint import pprint
   ```

1. En la segunda celda, introduzca los elementos de solicitud que serán constantes en cada solicitud. Reemplace el nombre del servicio de búsqueda (YOUR-SEARCH-SERVICE-NAME) y la clave de API de administración (YOUR-ADMIN-API-KEY) por los valores apropiados. 

   ```python
   endpoint = 'https://<YOUR-SEARCH-SERVICE-NAME>.search.windows.net/'
   api_version = '?api-version=2020-06-30'
   headers = {'Content-Type': 'application/json',
           'api-key': '<YOUR-ADMIN-API-KEY>' }
   ```

   Si obtiene ConnectionError `"Failed to establish a new connection"`, compruebe que la clave de API sea una clave de administrador principal o secundaria, y que todos los caracteres iniciales y finales (`?` y `/`) estén en su lugar.

1. En la tercera celda, formule la solicitud. Esta solicitud GET tiene como destino la colección de índices de su servicio de búsqueda y selecciona la propiedad de nombre de los índices existentes.

   ```python
   url = endpoint + "indexes" + api_version + "&$select=name"
   response  = requests.get(url, headers=headers)
   index_list = response.json()
   pprint(index_list)
   ```

1. Ejecute cada paso. Si existen índices, la respuesta contiene una lista de nombres de índice. En la siguiente captura de pantalla, el servicio ya tiene un azureblob-index y un índice realestate-us-sample.

   ![Script de Python en Jupyter Notebook con solicitudes HTTP a Azure Cognitive Search](media/search-get-started-python/connect-azure-search.png "Script de Python en Jupyter Notebook con solicitudes HTTP a Azure Cognitive Search")

   En cambio, una colección de índices vacía devuelve esta respuesta: `{'@odata.context': 'https://mydemo.search.windows.net/$metadata#indexes(name)', 'value': []}`

## <a name="1---create-an-index"></a>1 - Creación de un índice

A menos que use el portal, debe existir un índice en el servicio antes de poder cargar los datos. Este paso se usa la [API de REST de Crear índice](https://docs.microsoft.com/rest/api/searchservice/create-index) para insertar un esquema de índice en el servicio.

Los elementos necesarios de un índice incluyen un nombre, una colección de campos y una clave. La colección de campos define la estructura de un *documento*. Cada campo tiene un nombre, un tipo y unos atributos que determinan cómo se usa el campo (por ejemplo, si es texto completo que se puede buscar, filtrar o recuperar en los resultados de búsqueda). Dentro de un índice, uno de los campos de tipo `Edm.String` se debe designar como la *clave* para la identidad del documento.

Este índice se denomina "hotels-quickstart" y tiene las definiciones de campo que aparecen a continuación. Es un subconjunto de un [índice de hoteles](https://github.com/Azure-Samples/azure-search-sample-data/blob/master/hotels/Hotels_IndexDefinition.JSON) mayor usado en otros tutoriales. Lo hemos acortado en este inicio rápido para mayor brevedad.

1. En la siguiente celda, pegue el siguiente ejemplo en una celda para proporcionar el esquema. 

    ```python
    index_schema = {
       "name": "hotels-quickstart",  
       "fields": [
         {"name": "HotelId", "type": "Edm.String", "key": "true", "filterable": "true"},
         {"name": "HotelName", "type": "Edm.String", "searchable": "true", "filterable": "false", "sortable": "true", "facetable": "false"},
         {"name": "Description", "type": "Edm.String", "searchable": "true", "filterable": "false", "sortable": "false", "facetable": "false", "analyzer": "en.lucene"},
         {"name": "Description_fr", "type": "Edm.String", "searchable": "true", "filterable": "false", "sortable": "false", "facetable": "false", "analyzer": "fr.lucene"},
         {"name": "Category", "type": "Edm.String", "searchable": "true", "filterable": "true", "sortable": "true", "facetable": "true"},
         {"name": "Tags", "type": "Collection(Edm.String)", "searchable": "true", "filterable": "true", "sortable": "false", "facetable": "true"},
         {"name": "ParkingIncluded", "type": "Edm.Boolean", "filterable": "true", "sortable": "true", "facetable": "true"},
         {"name": "LastRenovationDate", "type": "Edm.DateTimeOffset", "filterable": "true", "sortable": "true", "facetable": "true"},
         {"name": "Rating", "type": "Edm.Double", "filterable": "true", "sortable": "true", "facetable": "true"},
         {"name": "Address", "type": "Edm.ComplexType", 
         "fields": [
         {"name": "StreetAddress", "type": "Edm.String", "filterable": "false", "sortable": "false", "facetable": "false", "searchable": "true"},
         {"name": "City", "type": "Edm.String", "searchable": "true", "filterable": "true", "sortable": "true", "facetable": "true"},
         {"name": "StateProvince", "type": "Edm.String", "searchable": "true", "filterable": "true", "sortable": "true", "facetable": "true"},
         {"name": "PostalCode", "type": "Edm.String", "searchable": "true", "filterable": "true", "sortable": "true", "facetable": "true"},
         {"name": "Country", "type": "Edm.String", "searchable": "true", "filterable": "true", "sortable": "true", "facetable": "true"}
        ]
       }
      ]
    }
    ```

2. En otra celda, formule la solicitud. Esta solicitud POST tiene como destino la colección de índices de su servicio de búsqueda y crea un índice en función del esquema de índice proporcionado en la celda anterior.

   ```python
   url = endpoint + "indexes" + api_version
   response  = requests.post(url, headers=headers, json=index_schema)
   index = response.json()
   pprint(index)
   ```

3. Ejecute cada paso.

   La respuesta incluye la representación JSON del esquema. La siguiente captura de pantalla muestra solo una parte de la respuesta.

    ![Solicitud de creación de un índice](media/search-get-started-python/create-index.png "Solicitud de creación de un índice")

> [!Tip]
> Otra forma de comprobar la creación de índices es comprobar la lista de índices en el portal.

<a name="load-documents"></a>

## <a name="2---load-documents"></a>2 - Carga de documentos

Para insertar documentos, use una solicitud HTTP POST al punto de conexión de la dirección URL del índice. La API de REST es de [Agregar, Actualizar o Eliminar documentos](https://docs.microsoft.com/rest/api/searchservice/addupdate-or-delete-documents). Los documentos se originan en [HotelsData](https://github.com/Azure-Samples/azure-search-sample-data/blob/master/hotels/HotelsData_toAzureSearch.JSON) en GitHub.

1. En una celda nueva, proporcione cuatro documentos que se ajusten al esquema de índice. Especifique una acción de carga para cada documento.

    ```python
    documents = {
        "value": [
        {
        "@search.action": "upload",
        "HotelId": "1",
        "HotelName": "Secret Point Motel",
        "Description": "The hotel is ideally located on the main commercial artery of the city in the heart of New York. A few minutes away is Time's Square and the historic centre of the city, as well as other places of interest that make New York one of America's most attractive and cosmopolitan cities.",
        "Description_fr": "L'hôtel est idéalement situé sur la principale artère commerciale de la ville en plein cœur de New York. A quelques minutes se trouve la place du temps et le centre historique de la ville, ainsi que d'autres lieux d'intérêt qui font de New York l'une des villes les plus attractives et cosmopolites de l'Amérique.",
        "Category": "Boutique",
        "Tags": [ "pool", "air conditioning", "concierge" ],
        "ParkingIncluded": "false",
        "LastRenovationDate": "1970-01-18T00:00:00Z",
        "Rating": 3.60,
        "Address": {
            "StreetAddress": "677 5th Ave",
            "City": "New York",
            "StateProvince": "NY",
            "PostalCode": "10022",
            "Country": "USA"
            }
        },
        {
        "@search.action": "upload",
        "HotelId": "2",
        "HotelName": "Twin Dome Motel",
        "Description": "The hotel is situated in a  nineteenth century plaza, which has been expanded and renovated to the highest architectural standards to create a modern, functional and first-class hotel in which art and unique historical elements coexist with the most modern comforts.",
        "Description_fr": "L'hôtel est situé dans une place du XIXe siècle, qui a été agrandie et rénovée aux plus hautes normes architecturales pour créer un hôtel moderne, fonctionnel et de première classe dans lequel l'art et les éléments historiques uniques coexistent avec le confort le plus moderne.",
        "Category": "Boutique",
        "Tags": [ "pool", "free wifi", "concierge" ],
        "ParkingIncluded": "false",
        "LastRenovationDate": "1979-02-18T00:00:00Z",
        "Rating": 3.60,
        "Address": {
            "StreetAddress": "140 University Town Center Dr",
            "City": "Sarasota",
            "StateProvince": "FL",
            "PostalCode": "34243",
            "Country": "USA"
            }
        },
        {
        "@search.action": "upload",
        "HotelId": "3",
        "HotelName": "Triple Landscape Hotel",
        "Description": "The Hotel stands out for its gastronomic excellence under the management of William Dough, who advises on and oversees all of the Hotel's restaurant services.",
        "Description_fr": "L'hôtel est situé dans une place du XIXe siècle, qui a été agrandie et rénovée aux plus hautes normes architecturales pour créer un hôtel moderne, fonctionnel et de première classe dans lequel l'art et les éléments historiques uniques coexistent avec le confort le plus moderne.",
        "Category": "Resort and Spa",
        "Tags": [ "air conditioning", "bar", "continental breakfast" ],
        "ParkingIncluded": "true",
        "LastRenovationDate": "2015-09-20T00:00:00Z",
        "Rating": 4.80,
        "Address": {
            "StreetAddress": "3393 Peachtree Rd",
            "City": "Atlanta",
            "StateProvince": "GA",
            "PostalCode": "30326",
            "Country": "USA"
            }
        },
        {
        "@search.action": "upload",
        "HotelId": "4",
        "HotelName": "Sublime Cliff Hotel",
        "Description": "Sublime Cliff Hotel is located in the heart of the historic center of Sublime in an extremely vibrant and lively area within short walking distance to the sites and landmarks of the city and is surrounded by the extraordinary beauty of churches, buildings, shops and monuments. Sublime Cliff is part of a lovingly restored 1800 palace.",
        "Description_fr": "Le sublime Cliff Hotel est situé au coeur du centre historique de sublime dans un quartier extrêmement animé et vivant, à courte distance de marche des sites et monuments de la ville et est entouré par l'extraordinaire beauté des églises, des bâtiments, des commerces et Monuments. Sublime Cliff fait partie d'un Palace 1800 restauré avec amour.",
        "Category": "Boutique",
        "Tags": [ "concierge", "view", "24-hour front desk service" ],
        "ParkingIncluded": "true",
        "LastRenovationDate": "1960-02-06T00:00:00Z",
        "Rating": 4.60,
        "Address": {
            "StreetAddress": "7400 San Pedro Ave",
            "City": "San Antonio",
            "StateProvince": "TX",
            "PostalCode": "78216",
            "Country": "USA"
            }
        }
    ]
    }
    ```   

2. En otra celda, formule la solicitud. Esta solicitud POST tiene como destino la colección de documentos del índice hotels-quickstart y envía los documentos proporcionados en el paso anterior.

   ```python
   url = endpoint + "indexes/hotels-quickstart/docs/index" + api_version
   response  = requests.post(url, headers=headers, json=documents)
   index_content = response.json()
   pprint(index_content)
   ```

3. Ejecute cada paso para insertar los documentos en un índice en el servicio de búsqueda. Los resultados deben tener un aspecto similar al siguiente ejemplo. 

    ![Envío de documentos a un índice](media/search-get-started-python/load-index.png "Envío de documentos a un índice")

## <a name="3---search-an-index"></a>3 - Búsqueda en un índice

Este paso muestra cómo realizar consultas en un índice con la [API de REST de Buscar documentos](https://docs.microsoft.com/rest/api/searchservice/search-documents).

1. En una celda, proporcione una expresión de consulta que ejecute una búsqueda vacía (búsqueda=*),la cual devuelve una lista no clasificada (puntuación de búsqueda=1,0) de documentos arbitrarios. De forma predeterminada, Azure Cognitive Search devuelve 50 resultados cada vez. Al ser estructurada, esta consulta devuelve la estructura y los valores del documento entero. Agregue $count=true para obtener un recuento de todos los documentos de los resultados.

   ```python
   searchstring = '&search=*&$count=true'

   url = endpoint + "indexes/hotels-quickstart/docs" + api_version + searchstring
   response  = requests.get(url, headers=headers, json=searchstring)
   query = response.json()
   pprint(query)
   ```

1. En una celda nueva, proporcione el ejemplo siguiente para buscar los términos "hoteles" y "wifi". Agregue $select para especificar qué campos quiere incluir en los resultados de búsqueda.

   ```python
   searchstring = '&search=hotels wifi&$count=true&$select=HotelId,HotelName'

   url = endpoint + "indexes/hotels-quickstart/docs" + api_version + searchstring
   response  = requests.get(url, headers=headers, json=searchstring)
   query = response.json()
   pprint(query)   
   ```

   Los resultados deben tener un aspecto similar a la siguiente salida. 

    ![Búsqueda de un índice](media/search-get-started-python/search-index.png "Búsqueda de un índice")

1. A continuación, aplique una expresión $filter que seleccione solo aquellos hoteles con una clasificación superior a 4. 

   ```python
   searchstring = '&search=*&$filter=Rating gt 4&$select=HotelId,HotelName,Description,Rating'

   url = endpoint + "indexes/hotels-quickstart/docs" + api_version + searchstring
   response  = requests.get(url, headers=headers, json=searchstring)
   query = response.json()
   pprint(query)     
   ```

1. De forma predeterminada, el motor de búsqueda devuelve los 50 documentos más importantes, pero puede usar las subcláusulas TOP y SKIP para agregar paginación y elegir el número de documentos en cada resultado. Esta consulta devuelve dos documentos en cada conjunto de resultados.

   ```python
   searchstring = '&search=boutique&$top=2&$select=HotelId,HotelName,Description'

   url = endpoint + "indexes/hotels-quickstart/docs" + api_version + searchstring
   response  = requests.get(url, headers=headers, json=searchstring)
   query = response.json()
   pprint(query)
   ```

1. En este último ejemplo, use $orderby para ordenar los resultados por ciudad. En este ejemplo se incluyen los campos de la colección de direcciones.

   ```python
   searchstring = '&search=pool&$orderby=Address/City&$select=HotelId, HotelName, Address/City, Address/StateProvince'

   url = endpoint + "indexes/hotels-quickstart/docs" + api_version + searchstring
   response  = requests.get(url, headers=headers, json=searchstring)
   query = response.json()
   pprint(query)
   ```

## <a name="clean-up"></a>Limpieza

Cuando trabaje con su propia suscripción, es una buena idea al final de un proyecto identificar si todavía se necesitan los recursos que ha creado. Los recursos que se dejan en ejecución pueden costarle mucho dinero. Puede eliminar los recursos de forma individual o eliminar el grupo de recursos para eliminar todo el conjunto de recursos.

Puede encontrar y administrar recursos en el portal, mediante el vínculo **Todos los recursos** o **Grupos de recursos** en el panel de navegación izquierdo.

Si está usando un servicio gratuito, recuerde que está limitado a tres índices, indexadores y orígenes de datos. Puede eliminar elementos individuales en el portal para mantenerse por debajo del límite. 

## <a name="next-steps"></a>Pasos siguientes

Como simplificación, este tutorial de rápido usa una versión abreviada del índice de hoteles. Puede crear la versión completa para realizar más consultas interesantes. Para obtener la versión completa y los 50 documentos, ejecute el asistente **Importar datos** y seleccione *hotels-sample* en los orígenes de datos de ejemplo integrados.

> [!div class="nextstepaction"]
> [Inicio rápido: Creación de un índice de Azure Search en Azure Portal](search-get-started-portal.md)
