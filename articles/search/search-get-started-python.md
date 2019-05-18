---
title: 'Inicio rápido: Python y las API de REST - Azure Search'
description: Crear, cargar y consultar un índice mediante Python, Jupyter Notebooks y la API de REST de Azure Search.
ms.date: 05/15/2019
author: heidisteen
manager: cgronlun
ms.author: heidist
services: search
ms.service: search
ms.devlang: rest-api
ms.topic: conceptual
ms.custom: seodec2018
ms.openlocfilehash: 1ab6bb069f60f4d2dbb4cfaecda54c3c2ef20adc
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65806437"
---
# <a name="quickstart-create-an-azure-search-index-using-jupyter-python-notebooks"></a>Inicio rápido: Crear un índice de Azure Search con cuadernos de Jupyter Python
> [!div class="op_single_selector"]
> * [Python (REST)](search-get-started-python.md)
> * [PowerShell (REST)](search-create-index-rest-api.md)
> * [C#](search-create-index-dotnet.md)
> * [Postman (REST)](search-fiddler.md)
> * [Portal](search-create-index-portal.md)
> 

Crear un cuaderno de Jupyter que crea, carga y consulta de una búsqueda de Azure [índice](search-what-is-an-index.md) con Python y la [las API de REST de Azure Search Service](https://docs.microsoft.com/rest/api/searchservice/). En este artículo se explica cómo crear su propio cuaderno paso a paso. Si lo desea, puede ejecutar un cuaderno finalizado. Para descargar una copia, vaya a [repositorio Azure-Search-python-samples](https://github.com/Azure-Samples/azure-search-python-samples).

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar y luego [regístrese en Azure Search](search-create-service-portal.md).

## <a name="prerequisites"></a>Requisitos previos

En este inicio rápido se usan los siguientes servicios y herramientas. 

+ [Cree un servicio Azure Search](search-create-service-portal.md) o [busque un servicio existente](https://ms.portal.azure.com/#blade/HubsExtension/BrowseResourceBlade/resourceType/Microsoft.Search%2FsearchServices) en su suscripción actual. Puede usar un servicio gratuito para este inicio rápido. 

+ [Anaconda 3.x](https://www.anaconda.com/distribution/#download-section), proporcionando Python 3.x y cuadernos de Jupyter.

## <a name="get-a-key-and-url"></a>Obtención de una clave y una dirección URL

Las llamadas de REST requieren la dirección URL del servicio y una clave de acceso en cada solicitud. Con ambos se crea un servicio de búsqueda, por lo que si ha agregado Azure Search a su suscripción, siga estos pasos para obtener la información necesaria:

1. [Inicie sesión en Azure Portal](https://portal.azure.com/) y en la página **Introducción** del servicio de búsqueda, obtenga la dirección URL. Un punto de conexión de ejemplo podría ser similar a `https://mydemo.search.windows.net`.

1. En **Configuración** > **Claves**, obtenga una clave de administrador para tener derechos completos en el servicio. Se proporcionan dos claves de administrador intercambiables para lograr la continuidad empresarial, por si necesitara sustituir una de ellas. Puede usar la clave principal o secundaria en las solicitudes para agregar, modificar y eliminar objetos.

![Obtención de una clave de acceso y un punto de conexión HTTP](media/search-fiddler/get-url-key.png "Get an HTTP endpoint and access key")

Todas las solicitudes requieren una clave de API en cada solicitud enviada al servicio. Tener una clave válida genera la confianza, solicitud a solicitud, entre la aplicación que envía la solicitud y el servicio que se encarga de ella.

## <a name="connect-to-azure-search"></a>Conexión con Azure Search

Abrir un cuaderno de Jupyter y compruebe la conexión desde su estación de trabajo local mediante la solicitud de una lista de índices en el servicio. En Windows con Anaconda3, puede utilizar el navegador de Anaconda para iniciar un bloc de notas.

1. Crear un nuevo cuaderno de Python3.

1. En la primera celda, cargue las bibliotecas que usa para trabajar con JSON y formular las solicitudes HTTP.

   ```python
   import json
   import requests
   from pprint import pprint
   ```

1. En la segunda celda, los elementos de solicitud que será constantes en cada solicitud de entrada. Reemplace el nombre de servicio de búsqueda (YOUR-SEARCH-SERVICE-NAME) y la clave de API de administración (YOUR-ADMIN-API-KEY) por los valores válidos. 

   ```python
    endpoint = 'https://<YOUR-SEARCH-SERVICE-NAME>.search.windows.net/'
    api_version = '?api-version=2019-05-06'
    headers = {'Content-Type': 'application/json',
           'api-key': '<YOUR-ADMIN-API-KEY>' }
   ```

1. En la tercera celda, formular la solicitud. Esta solicitud GET tiene como destino la colección de índices de su servicio de búsqueda y selecciona la propiedad name.

   ```python
   url = endpoint + "indexes" + api_version + "&$select=name"
   response  = requests.get(url, headers=headers)
   index_list = response.json()
   pprint(index_list)
   ```

1. Ejecutar cada paso. Si existen índices, la respuesta contiene una lista de índices. En la siguiente captura de pantalla, el servicio incluye un índice de Azure BLOB y un índice realestate-us-sample.

   ![Solicitudes de script de Python en el cuaderno de Jupyter notebook con HTTP a Azure Search](media/search-get-started-python/connect-azure-search.png "solicitudes de script de Python en el cuaderno de Jupyter notebook con HTTP a Azure Search")

   Una colección de índices vacía devuelve esta respuesta: `{'@odata.context': 'https://mydemo.search.windows.net/$metadata#indexes(name)', 'value': []}`

> [!Tip]
> En un servicio gratuito, estará limitado a tres índices, indexadores y orígenes de datos. Este tutorial rápido crea uno de cada uno. Asegúrese de que haya espacio suficiente para crear objetos nuevos antes de continuar.

## <a name="1---create-an-index"></a>1 - Creación de un índice

A menos que se está usando el portal, debe existir un índice en el servicio antes de poder cargar los datos. Este paso se usa el [API de REST de Create Index](https://docs.microsoft.com/rest/api/searchservice/create-index) para insertar un esquema de índice en el servicio

La colección de campos define la estructura de un *documento*. Elementos necesarios de un índice incluyen un nombre y una colección de campos. Cada campo tiene un nombre, tipo y los atributos que determinan cómo se utiliza (por ejemplo, si es texto que se pueden buscar, filtrar o recuperar resultados de búsqueda). Dentro de un índice, uno de los campos de tipo `Edm.String` debe designarse como el *clave* para la identidad del documento.

Este índice se denomina "hoteles-py" y tiene las definiciones de campo que aparece a continuación. Es un subconjunto de una mayor [índice de hoteles](https://github.com/Azure-Samples/azure-search-sample-data/blob/master/hotels/Hotels_IndexDefinition.JSON) utilizado en otros tutoriales. Lo hemos acortado en este inicio rápido para mayor brevedad.


1. En la siguiente celda, pegue el siguiente ejemplo en una celda para proporcionar el esquema. 

    ```python
    index_schema = {
       "name": "hotels-py",  
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

2. En otra celda, formular la solicitud. Esto coloca solicitud tiene como destino la colección de índices de su servicio de búsqueda y crea un índice basado en el esquema de índice proporcionado en el paso anterior.

   ```python
   url = endpoint + "indexes" + api_version
   response  = requests.post(url, headers=headers, json=index_schema)
   index = response.json()
   pprint(index)
   ```

3. Ejecutar cada paso.

   La respuesta incluye la representación JSON del esquema. Captura de pantalla siguiente recorta partes del esquema de índice para que puedan ver más de la respuesta.

    ![Solicitud para crear un índice](media/search-get-started-python/create-index.png "solicitud para crear un índice")

> [!Tip]
> Para la comprobación, podría también comprobar la lista de índices en el portal o vuelva a ejecutar la solicitud de conexión de servicio para ver el *hoteles py* índice aparece en la colección de índices.

<a name="load-documents"></a>

## <a name="2---load-documents"></a>2 - Carga de documentos

Para insertar los documentos, use una solicitud HTTP POST al punto de conexión de dirección URL del índice. La API de REST es [agregar, actualizar o eliminar documentos](https://docs.microsoft.com/rest/api/searchservice/addupdate-or-delete-documents). Documentos que se originan en [HotelsData](https://github.com/Azure-Samples/azure-search-sample-data/blob/master/hotels/HotelsData_toAzureSearch.JSON) en GitHub.

1. En una celda nueva, proporcionar tres documentos que se ajustan al esquema de índice. Especifique una acción de carga para cada documento.

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
        "Description": "The Hotel stands out for its gastronomic excellence under the management of William Dough, who advises on and oversees all of the Hotel’s restaurant services.",
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
      }
     ]
    }
    ```

2. En otra celda, formular la solicitud. Esta solicitud POST tiene como destino la colección de documentos del índice de hoteles py y envía los documentos proporcionados en el paso anterior.

   ```python
   url = endpoint + "indexes/hotels-py/docs/index" + api_version
   response  = requests.post(url, headers=headers, json=documents)
   index_content = response.json()
   pprint(index_content)
   ```

3. Ejecutar cada paso para insertar los documentos en un índice en el servicio de búsqueda. Resultados deben ser similares al ejemplo siguiente. 

   ```
   {'@odata.context': "https://mydemo.search.windows.net/indexes('hotels-py')/$metadata#Collection(Microsoft.Azure.Search.V2019_05_06.IndexResult)",
    'value': [{'errorMessage': None,
            'key': '1',
            'status': True,
            'statusCode': 201},
           {'errorMessage': None,
            'key': '2',
            'status': True,
            'statusCode': 201},
           {'errorMessage': None,
            'key': '3',
            'status': True,
            'statusCode': 201}]}
     ```


## <a name="3---search-an-index"></a>3 - Búsqueda en un índice

Este paso muestra cómo consultar un índice mediante el [API de REST de búsqueda de documentos](https://docs.microsoft.com/rest/api/searchservice/search-documents).


1. En una celda nueva, proporcionar una expresión de consulta. El siguiente ejemplo se busca en los términos "hoteles" y "wifi". También devuelve un *recuento* de documentos que coinciden, y *selecciona* qué campos desea incluir en los resultados de búsqueda.

   ```python
   searchstring = '&search=hotels wifi&$count=true&$select=HotelId,HotelName'
   ```

2. Formular una solicitud. Esta solicitud GET tiene como destino la colección de documentos del índice de hoteles py y adjunta la consulta que especificó en el paso anterior.

   ```python
   url = endpoint + "indexes/hotels-py/docs" + api_version + searchstring
   response  = requests.get(url, headers=headers, json=searchstring)
   query = response.json()
   pprint(query)
   ```

   Resultados deben ser similares al siguiente resultado.

   ```
   {'@odata.context': "https://mydemo.search.windows.net/indexes('hotels-py')/$metadata#docs(*)",
    '@odata.count': 3,
    'value': [{'@search.score': 1.0,
               'HotelId': '1',
               'HotelName': 'Secret Point Motel'},
              {'@search.score': 1.0,
               'HotelId': '2',
               'HotelName': 'Twin Dome Motel'},
              {'@search.score': 1.0,
               'HotelId': '3',
               'HotelName': 'Triple Landscape Hotel'}]}
   ```

3. Pruebe algunos otros ejemplos de consultas para hacerse una idea de la sintaxis. Puede aplicar un filtro, tomar los dos primeros resultados, ordenar por un campo específico, o 

   + `searchstring = '&search=*&$filter=Rating gt 4&$select=HotelId,HotelName,Description'`

   + `searchstring = '&search=hotel&$top=2&$select=HotelId,HotelName,Description'`

   + `searchstring = '&search=pool&$orderby=Address/City&$select=HotelId, HotelName, Address/City, Address/StateProvince'`

## <a name="clean-up"></a>Limpieza 

Debe eliminar el índice si ya no lo necesita. Un servicio gratuito está limitado a tres índices. Es posible que desee eliminar cualquier índice que no está usando activamente para dejar espacio para otros tutoriales.

   ```python
  url = endpoint + "indexes/hotels-py" + api_version
  response  = requests.delete(url, headers=headers)
   ```

Puede comprobar la eliminación de índices al devolver una lista de los índices existentes. Si desaparece py de hoteles, saber la solicitud se realizó correctamente.

```python
url = endpoint + "indexes" + api_version + "&$select=name"

response  = requests.get(url, headers=headers)
index_list = response.json()
pprint(index_list)
```

## <a name="next-steps"></a>Pasos siguientes

Más información sobre los escenarios y sintaxis de consulta.

> [!div class="nextstepaction"]
> [Crear una consulta básica](search-query-overview.md)
