---
title: Sinónimos para la expansión de consultas a través de un índice de búsqueda
titleSuffix: Azure Cognitive Search
description: Creación de un mapa de sinónimos para expandir el ámbito de una consulta de búsqueda en un índice de Azure Cognitive Search. El ámbito se ha ampliado para que incluya los términos equivalentes que proporcione en una lista.
manager: nitinme
author: brjohnstmsft
ms.author: brjohnst
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 08/26/2020
ms.openlocfilehash: aad953483749d676844221f7e519f50c50b63ad4
ms.sourcegitcommit: e69bb334ea7e81d49530ebd6c2d3a3a8fa9775c9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "88948647"
---
# <a name="synonyms-in-azure-cognitive-search"></a>Sinónimos de Azure Cognitive Search

Los sinónimos de los motores de búsqueda asocian términos equivalentes que expanden implícitamente el ámbito de una consulta, sin que el usuario tenga que proporcionar realmente el término. Por ejemplo, con el término "perro" y las asociaciones de sinónimos de "canino" y "cachorro", los documentos que contengan los términos "perro", "canino" o "cachorro" estarán dentro del ámbito de la consulta.

En Azure Cognitive Search, la expansión de sinónimos se realiza en el momento de la consulta. Puede agregar asignaciones de sinónimos a un servicio sin que se interrumpan las operaciones existentes. Puede agregar una propiedad **synonymMaps** a una definición de campo sin tener que volver a crear un índice.

## <a name="create-synonyms"></a>Creación de sinónimos

No hay ningún soporte técnico del portal para crear sinónimos, pero puede usar la API REST o el SDK de .NET. Para empezar a trabajar con REST, se recomienda hacerlo [mediante Postman](search-get-started-postman.md) y la formulación de solicitudes que usan esta API: [Creación de asignaciones de sinónimos](/rest/api/searchservice/create-synonym-map). En el caso de los desarrolladores de C#, puede empezar a trabajar con la opción [Add Synonyms in Azure Cognitive Searching using C#](search-synonyms-tutorial-sdk.md) (Adición de sinónimos en Azure Cognitive Search con C#).

Opcionalmente, si usa [claves administradas por el cliente](search-security-manage-encryption-keys.md) para el cifrado en reposo del servicio, puede aplicar dicha protección al contenido de la asignación de sinónimos.

## <a name="use-synonyms"></a>Usar sinónimos

En Azure Cognitive Search, la compatibilidad de los sinónimos se basa en las asignaciones de sinónimos que defina y cargue en el servicio. Estas asignaciones constituyen un recurso independiente (como índices u orígenes de datos), y cualquier campo buscable puede usarlas en cualquier índice en el servicio de búsqueda.

Los índices y asignaciones de sinónimos se mantienen de forma independiente. Una vez que defina una asignación de sinónimos y la cargue en el servicio, podrá habilitar la característica Sinónimos en un campo agregando una nueva propiedad denominada **synonymMaps** en la definición del campo. La creación, carga y eliminación de una asignación de sinónimos es siempre una operación de documento completo, lo que significa que no puede crear, actualizar o eliminar partes de la asignación de sinónimos de forma incremental. La actualización de incluso una entrada única requiere que se vuelva a cargar.

La incorporación de sinónimos en la aplicación de búsqueda es un proceso de dos pasos:

1.  Agregar una asignación de sinónimos al servicio de búsqueda a través de las API siguientes.  

2.  Configurar un campo buscable para usar la asignación de sinónimos en la definición del índice.

Puede crear varias asignaciones para la aplicación de búsqueda (por ejemplo, mediante el idioma si la aplicación es compatible con una base de cliente de varios idiomas). Actualmente, un campo solo puede usar una de ellas. Puede actualizar una propiedad synonymMaps del campo en cualquier momento.

### <a name="synonymmaps-resource-apis"></a>API de recursos de SynonymMaps

#### <a name="add-or-update-a-synonym-map-under-your-service-using-post-or-put"></a>Adición o actualización de una asignación de sinónimos en su servicio, con POST o PUT

Las asignaciones de sinónimos se cargan en el servicio a través de POST o PUT. Cada regla debe estar delimitada por el nuevo carácter de línea ('\n'). Puede definir hasta 5000 reglas por mapa de sinónimos en un servicio gratuito y 20 000 reglas por mapa en las demás SKU. Cada regla puede tener hasta 20 expansiones.

Las asignaciones de sinónimos deben estar en formato Apache Solr, que se explica a continuación. Si dispone de un diccionario de sinónimos existente en un formato distinto y desea usarlo directamente, háganoslo saber en [UserVoice](https://feedback.azure.com/forums/263029-azure-search).

Puede crear una nueva asignación de sinónimos con HTTP POST, como en el siguiente ejemplo:

```synonym-map
    POST https://[servicename].search.windows.net/synonymmaps?api-version=2020-06-30
    api-key: [admin key]

    {
       "name":"mysynonymmap",
       "format":"solr",
       "synonyms": "
          USA, United States, United States of America\n
          Washington, Wash., WA => WA\n"
    }
```

De forma alternativa, puede usar PUT y especificar el nombre de asignación del sinónimo en el identificador URI. Si la asignación de sinónimos no existe, se creará.

```synonym-map
    PUT https://[servicename].search.windows.net/synonymmaps/mysynonymmap?api-version=2020-06-30
    api-key: [admin key]

    {
       "format":"solr",
       "synonyms": "
          USA, United States, United States of America\n
          Washington, Wash., WA => WA\n"
    }
```

##### <a name="apache-solr-synonym-format"></a>Formato de sinónimos de Apache

El formato Solr es compatible con asignaciones de sinónimos equivalentes y explícitos. Las reglas de asignación se adhieren a la especificación del filtro de sinónimos de código abierto de Apache Solr descrita en este documento: [SynonymFilter](https://cwiki.apache.org/confluence/display/solr/Filter+Descriptions#FilterDescriptions-SynonymFilter). A continuación se muestra una regla de ejemplo para los sinónimos equivalentes.

```
USA, United States, United States of America
```

Con la regla anterior, la consulta de búsqueda "EE. UU." se ampliará a "EE. UU." O "Estados Unidos" O "Estados Unidos de América".

La asignación explícita se denota mediante una flecha "=>". Cuando se especifica, una secuencia de términos de una consulta de búsqueda que coincide con el lateral izquierdo de "=>" se sustituirá por las alternativas de la derecha. Según la regla siguiente, las consultas de búsqueda "Washington", "Wash." o "WA" se reescribirán a "WA". La asignación explícita solo se aplica en la dirección especificada y no reescribe la consulta "WA" a "Washington" en este caso.

```
Washington, Wash., WA => WA
```

Si necesita definir sinónimos que contengan comas, puede usar una barra diagonal inversa como carácter de escape, como en este ejemplo:

```
WA\, USA, WA, Washington
```

Dado que la barra diagonal inversa es en sí misma un carácter especial de otros lenguajes como JSON y C#, es probable que tenga que hacer doble escape. Por ejemplo, el JSON enviado la API de REST para la asignación de sinónimos anterior tendría el siguiente aspecto:

```json
    {
       "format":"solr",
       "synonyms": "WA\\, USA, WA, Washington"
    }
```

#### <a name="list-synonym-maps-under-your-service"></a>Enumeración de asignaciones de sinónimos en su servicio

```synonym-map
    GET https://[servicename].search.windows.net/synonymmaps?api-version=2020-06-30
    api-key: [admin key]
```

#### <a name="get-a-synonym-map-under-your-service"></a>Obtención de la asignación de sinónimos en su servicio

```synonym-map
    GET https://[servicename].search.windows.net/synonymmaps/mysynonymmap?api-version=2020-06-30
    api-key: [admin key]
```

#### <a name="delete-a-synonyms-map-under-your-service"></a>Eliminación de la asignación de sinónimos en su servicio

```synonym-map
    DELETE https://[servicename].search.windows.net/synonymmaps/mysynonymmap?api-version=2020-06-30
    api-key: [admin key]
```

### <a name="configure-a-searchable-field-to-use-the-synonym-map-in-the-index-definition"></a>Configurar un campo buscable para usar la asignación de sinónimos en la definición del índice.

Puede usarse una nueva propiedad de campo **synonymMaps** para especificar una asignación de sinónimos que usar para un campo buscable. Las asignaciones de sinónimos son recursos de nivel de servicio y puede hacerse referencia a ellas mediante cualquier campo del índice en el servicio.

```synonym-map
    POST https://[servicename].search.windows.net/indexes?api-version=2020-06-30
    api-key: [admin key]

    {
       "name":"myindex",
       "fields":[
          {
             "name":"id",
             "type":"Edm.String",
             "key":true
          },
          {
             "name":"name",
             "type":"Edm.String",
             "searchable":true,
             "analyzer":"en.lucene",
             "synonymMaps":[
                "mysynonymmap"
             ]
          },
          {
             "name":"name_jp",
             "type":"Edm.String",
             "searchable":true,
             "analyzer":"ja.microsoft",
             "synonymMaps":[
                "japanesesynonymmap"
             ]
          }
       ]
    }
```

**synonymMaps** puede especificarse para los campos buscables del tipo 'Edm.String' o 'Collection(Edm.String)'.

> [!NOTE]
> Solo puede tener una asignación de sinónimos por campo. Si desea usar varias asignaciones de sinónimos, indíquenoslo en [UserVoice](https://feedback.azure.com/forums/263029-azure-search).

## <a name="impact-of-synonyms-on-other-search-features"></a>Repercusión de Sinónimos en otras características de búsqueda

La característica Sinónimos reescribe la consulta original con sinónimos con el operador OR. Por este motivo, el resaltado de referencias y los perfiles de puntuación tratan el término original y los sinónimos como equivalentes.

La característica Sinónimos se aplica a consultas de búsquedas y no se aplica a filtros o facetas. De forma similar, las sugerencias se basan solo en el término original; las coincidencias de sinónimos no aparecen en la respuesta.

Las expansiones de sinónimos no se aplican a los términos de búsqueda de carácter comodín; los prefijos, las coincidencias parciales y las regex no se expanden.

Si tiene que realizar una consulta única que aplique la expansión de sinónimos y búsquedas aproximadas, de expresiones regulares y con comodines, puede combinar las consultas utilizando la sintaxis OR. Por ejemplo, para combinar sinónimos con caracteres comodín en la sintaxis de consulta única, el término sería `<query> | <query>*`.

Si dispone de un índice existente en un entorno de desarrollo (no producción), experimente con un diccionario pequeño para ver cómo la adición de sinónimos cambia la experiencia de búsqueda, incluida la repercusión en los perfiles de puntuación, el resaltado de referencias y las sugerencias.

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Creación de un mapa de sinónimos](/rest/api/searchservice/create-synonym-map)