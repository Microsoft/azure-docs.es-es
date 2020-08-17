---
title: Creación de un índice de búsqueda
titleSuffix: Azure Cognitive Search
description: Incluye conceptos y herramientas de indexación en Azure Cognitive Search, entre otras las definiciones de esquema y la estructura física de los datos.
manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 07/15/2020
ms.openlocfilehash: 9e8d1c012ae07fc458a324315e2635f04c3dbd78
ms.sourcegitcommit: 3543d3b4f6c6f496d22ea5f97d8cd2700ac9a481
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/20/2020
ms.locfileid: "86496542"
---
# <a name="create-a-basic-search-index-in-azure-cognitive-search"></a>Creación de un índice básico en Azure Cognitive Search

Un *índice de búsqueda* en Azure Cognitive Search almacena contenido que permite búsquedas y se usa para las consultas filtradas y de texto completo. Un índice se define mediante un esquema y se guarda en el servicio y, a continuación, se realiza la importación como segundo paso. 

Los índices contienen *documentos*. Desde un punto de vista conceptual, un documento es una sola unidad de datos habilitada para búsquedas en el índice. Un minorista podría tener un documento para cada objeto que vende, una organización de noticias podría tener un documento para cada artículo, etc. Estos conceptos se pueden asignar a conceptos equivalentes de bases de datos más conocidos: un *índice de búsqueda* equivale a una *tabla* y los *documentos* son más o menos equivalentes a las *filas* de una tabla.

El esquema determina la estructura física de un índice, los campos marcados como "buscables" dan como resultado un índice invertido creado para ese campo. 

Puede crear un índice con las siguientes herramientas y API:

* En Azure Portal, use **Agregar índice** o el Asistente para la **importación de datos**.
* Mediante [Create Index (API REST)](https://docs.microsoft.com/rest/api/searchservice/create-index)
* Mediante el [SDK de .NET](search-create-index-dotnet.md)

Es más fácil aprender con una herramienta de portal. El portal aplica ciertos requisitos y reglas de esquema para tipos de datos específicos, como no permitir funciones de búsqueda de texto completo en campos numéricos. Una vez que tenga un índice funcional, puede realizar la transición al código mediante la recuperación de la definición de JSON desde el servicio con [Get index (API REST)](https://docs.microsoft.com/rest/api/searchservice/get-index) y la incorporación a la solución.

## <a name="recommended-workflow"></a>Flujo de trabajo recomendado

Llegar a un diseño de índice final es un proceso iterativo. Es habitual comenzar con el portal para crear el índice inicial y, a continuación, cambiar a código para colocar el índice bajo el control del código fuente.

1. Determine si puede usar [**Importar datos**](search-import-data-portal.md). Si los datos de origen proceden de un [tipo de origen de datos admitido en Azure](search-indexer-overview.md#supported-data-sources) el asistente realiza una indexación basada en indexador de todo en uno.

1. Si no puede usar **Importar datos**, comience con **Agregar índice** para definir el esquema.

   ![Incorporación de un comando de índice](media/search-what-is-an-index/add-index.png "Incorporación de un comando de índice")

1. Proporcione el nombre y la clave que se usan para identificar de forma exclusiva cada documento de búsqueda en el índice. La clave es obligatoria y tiene que ser de tipo Edm.String. Durante la importación, debe planear la asignación de un campo único de los datos de origen a este campo. 

   El portal proporciona un campo de `id` para la clave. Para invalidar el `id` predeterminado, cree un nuevo campo (por ejemplo, una nueva definición de campo denominada `HotelId`) y, a continuación, selecciónelo en **Clave**.

   ![Rellenar las propiedades necesarias](media/search-what-is-an-index//field-attributes.png "Rellenar las propiedades necesarias")

1. Agregue más campos. El portal muestra los [atributos de campo](#index-attributes) que están disponibles para los diferentes tipos de datos. Si no está familiarizado con el diseño de índices, esta información resulta útil.

   Si los datos de entrada son jerárquicos por naturaleza, asigne el [tipo complejo](search-howto-complex-data-types.md) de tipo de datos para representar las estructuras anidadas. El conjunto de datos de ejemplo integrado, Hotels, muestra tipos complejos mediante una dirección (contiene varios subcampos) que tiene una relación de uno a uno con cada hotel y una colección compleja de habitaciones, en la que hay varias habitaciones asociadas con cada hotel. 

1. Asigne los [Analizadores](#analyzers) a los campos de cadena antes de que se cree el índice. Haga lo mismo para los [proveedores de sugerencias](#suggesters) si desea habilitar Autocomplete en campos específicos.

1. Haga clic en **Crear** para compilar las estructuras físicas en el servicio de búsqueda.

1. Después de crear un índice, use comandos adicionales para revisar las definiciones o agregar más elementos.

   ![Página Agregar índice que muestra los atributos según el tipo de datos](media/search-what-is-an-index//field-definitions.png "Página Agregar índice que muestra los atributos según el tipo de datos")

1. Descargue el esquema de índices mediante [Get Index (API REST)](https://docs.microsoft.com/rest/api/searchservice/get-index) y una herramienta de prueba web como [Postman](search-get-started-postman.md). Ahora tiene una representación JSON del índice que puede adaptar para el código.

1. [Cargue el índice con datos](search-what-is-data-import.md). Azure Cognitive Search acepta documentos JSON. Para cargar los datos mediante programación, puede usar Postman con documentos JSON en la carga de solicitudes. Si los datos no se expresan con facilidad como JSON, este paso será más laborioso. 

    Una vez que se carga un índice con datos, la mayoría de los cambios que se realicen en los campos existentes requerirán que se anule el índice y se vuelva a generar otro.

1. Consulte el índice, examine los resultados y siga recorriendo en iteración el esquema de índice hasta que empiece a ver los resultados esperados. Puede usar el [**explorador de búsqueda**](search-explorer.md) o Postman para consultar el índice.

Durante el desarrollo, haga planes para realizar recompilaciones con frecuencia. Como se crean estructuras físicas en el servicio, la mayoría de las modificaciones realizadas en una definición de campo existente, requieren [quitar los índices y volverlos a crear](search-howto-reindex.md). Considere la posibilidad de trabajar con un subconjunto de los datos para asegurarse de que las recompilaciones van más rápido. 

> [!Tip]
> Para trabajar en el diseño de índices y la importación de datos simultáneamente, se recomienda el código en lugar del enfoque de portal. Como alternativa, herramientas como [Postman y la API REST](search-get-started-postman.md) son útiles con pruebas de concepto cuando los proyectos de desarrollo están aún en las fases iniciales. Puede realizar cambios incrementales en una definición de índice de un cuerpo de solicitud y, luego, enviar la solicitud al servicio para volver a crear un índice mediante un esquema actualizado.

## <a name="index-schema"></a>Esquema de índices

Un índice debe tener un nombre y un campo de clave designado (de Edm.string) en la colección de campos. La [*recopilación de campos*](#fields-collection) es normalmente el elemento más grande de un índice, en el que cada campo recibe un nombre, se escribe y se le asignan unos atributos con los comportamientos permitidos que determinan cómo se usa. 

Otros elementos incluyen [proveedores de sugerencias](#suggesters), [perfiles de puntuación](#scoringprofiles), [analizadores](#analyzers) que se utilizan para procesar cadenas en tokens de acuerdo con las reglas lingüísticas u otras características admitidas por el analizador y los valores de [scripting remoto entre orígenes (CORS)](#corsoptions).

```json
{
  "name": (optional on PUT; required on POST) "name_of_index",
  "fields": [
    {
      "name": "name_of_field",
      "type": "Edm.String | Collection(Edm.String) | Edm.Int32 | Edm.Int64 | Edm.Double | Edm.Boolean | Edm.DateTimeOffset | Edm.GeographyPoint",
      "searchable": true (default where applicable) | false (only Edm.String and Collection(Edm.String) fields can be searchable),
      "filterable": true (default) | false,
      "sortable": true (default where applicable) | false (Collection(Edm.String) fields cannot be sortable),
      "facetable": true (default where applicable) | false (Edm.GeographyPoint fields cannot be facetable),
      "key": true | false (default, only Edm.String fields can be keys),
      "retrievable": true (default) | false,
      "analyzer": "name_of_analyzer_for_search_and_indexing", (only if 'searchAnalyzer' and 'indexAnalyzer' are not set)
      "searchAnalyzer": "name_of_search_analyzer", (only if 'indexAnalyzer' is set and 'analyzer' is not set)
      "indexAnalyzer": "name_of_indexing_analyzer", (only if 'searchAnalyzer' is set and 'analyzer' is not set)
      "synonymMaps": [ "name_of_synonym_map" ] (optional, only one synonym map per field is currently supported)
    }
  ],
  "suggesters": [
    {
      "name": "name of suggester",
      "searchMode": "analyzingInfixMatching",
      "sourceFields": ["field1", "field2", ...]
    }
  ],
  "scoringProfiles": [
    {
      "name": "name of scoring profile",
      "text": (optional, only applies to searchable fields) {
        "weights": {
          "searchable_field_name": relative_weight_value (positive #'s),
          ...
        }
      },
      "functions": (optional) [
        {
          "type": "magnitude | freshness | distance | tag",
          "boost": # (positive number used as multiplier for raw score != 1),
          "fieldName": "...",
          "interpolation": "constant | linear (default) | quadratic | logarithmic",
          "magnitude": {
            "boostingRangeStart": #,
            "boostingRangeEnd": #,
            "constantBoostBeyondRange": true | false (default)
          },
          "freshness": {
            "boostingDuration": "..." (value representing timespan leading to now over which boosting occurs)
          },
          "distance": {
            "referencePointParameter": "...", (parameter to be passed in queries to use as reference location)
            "boostingDistance": # (the distance in kilometers from the reference location where the boosting range ends)
          },
          "tag": {
            "tagsParameter": "..." (parameter to be passed in queries to specify a list of tags to compare against target fields)
          }
        }
      ],
      "functionAggregation": (optional, applies only when functions are specified) 
        "sum (default) | average | minimum | maximum | firstMatching"
    }
  ],
  "analyzers":(optional)[ ... ],
  "charFilters":(optional)[ ... ],
  "tokenizers":(optional)[ ... ],
  "tokenFilters":(optional)[ ... ],
  "defaultScoringProfile": (optional) "...",
  "corsOptions": (optional) {
    "allowedOrigins": ["*"] | ["origin_1", "origin_2", ...],
    "maxAgeInSeconds": (optional) max_age_in_seconds (non-negative integer)
  },
  "encryptionKey":(optional){
    "keyVaultUri": "azure_key_vault_uri",
    "keyVaultKeyName": "name_of_azure_key_vault_key",
    "keyVaultKeyVersion": "version_of_azure_key_vault_key",
    "accessCredentials":(optional){
      "applicationId": "azure_active_directory_application_id",
      "applicationSecret": "azure_active_directory_application_authentication_key"
    }
  }
}
```

<a name="fields-collection"></a>

## <a name="fields-collection-and-field-attributes"></a>Recopilación de campos y atributos de campo

Los campos tienen un nombre, un tipo que clasifica los datos almacenados y atributos que especifican cómo se usa el campo.

### <a name="data-types"></a>Tipos de datos

| Tipo | Descripción |
|------|-------------|
| Edm.String |Texto que opcionalmente se puede acortar para búsquedas de texto completo (separación de palabras, lematización, etc.). |
| Collection(Edm.String) |Una lista de cadenas que opcionalmente se pueden acortar para búsquedas de texto completo. En teoría, no hay ningún límite superior para el número de elementos de una colección, pero el límite de 16 MB en el tamaño de la carga se aplica a las colecciones. |
| Edm.Boolean |Contiene valores true/false. |
| Edm.Int32 |Valores enteros de 32 bits. |
| Edm.Int64 |Valores enteros de 64 bits. |
| Edm.Double |Datos numéricos de precisión doble. |
| Edm.DateTimeOffset |Los valores de fecha y hora se representan con el formato OData V4 (por ejemplo, `yyyy-MM-ddTHH:mm:ss.fffZ` o `yyyy-MM-ddTHH:mm:ss.fff[+/-]HH:mm`). |
| Edm.GeographyPoint |Un punto que representa una ubicación geográfica en todo el mundo. |

Para más información, consulte los [tipos de datos admitidos](https://docs.microsoft.com/rest/api/searchservice/Supported-data-types).

<a name="index-attributes"></a>

### <a name="attributes"></a>Atributos

Los atributos de campo determinan cómo se usa un campo, por ejemplo, si se usa en la búsqueda de texto completo, la navegación por facetas, las operaciones de ordenación, etc. 

Los campos de cadena se suelen marcar como "buscables" y "recuperables". Los campos que se usan para restringir los resultados de búsqueda incluyen "ordenable", "filtrable" y "clasificable".

|Atributo|Descripción|  
|---------------|-----------------|  
|"buscable" |Permite realizar búsquedas de texto completo, sujetas a análisis léxico, como la separación de palabras durante la indexación. Si establece un campo buscable en un valor como "día soleado", internamente se dividirá en los tokens individuales "soleado" y "día". Para obtener detalles, vea [Búsqueda de texto completo](search-lucene-query-architecture.md).|  
|"filtrable" |Se hace referencia en consultas $filter. Los campos filtrables de tipo `Edm.String` o `Collection(Edm.String)` no sufren separación de palabras, por lo que las comparaciones son solo de coincidencias exactas. Por ejemplo, si establece un campo de este tipo en "día soleado", `$filter=f eq 'sunny'` no encontrará ninguna coincidencia, pero `$filter=f eq 'sunny day'` sí. |  
|"ordenable" |De forma predeterminada, el sistema ordena los resultados por puntuación, pero puede configurar la ordenación en función de los campos de los documentos. Los campos de tipo `Collection(Edm.String)` no pueden ser "ordenables". |  
|"clasificable" |Normalmente se usa en una presentación de resultados de búsqueda que incluye un recuento de visitas por categoría (por ejemplo, hoteles de una ciudad concreta). Esta opción no puede utilizarse con campos de tipo `Edm.GeographyPoint`. Los campos de tipo `Edm.String` que son "filtrables", "ordenables" o "clasificables" pueden tener como máximo 32 kilobytes de longitud. Para obtener detalles, vea [Creación de un índice de Búsqueda de Azure con la API de REST](https://docs.microsoft.com/rest/api/searchservice/create-index).|  
|"clave" |Identificador único de los documentos del índice. Es necesario elegir exactamente un campo como campo de clave, y debe ser de tipo `Edm.String`.|  
|"recuperable" |Determina si el campo se puede devolver en un resultado de búsqueda. Esto resulta útil cuando se quiere usar un campo (por ejemplo, *margen de beneficio*) como mecanismo de filtrado, ordenación o puntuación, pero no se quiere que sea visible para el usuario final. Este atributo debe ser `true` for `key` .|  

Aunque puede agregar nuevos campos en cualquier momento, las definiciones de campo existentes se bloquean durante la vigencia del índice. Por este motivo, los desarrolladores suelen usar el portal para crear índices sencillos, probar ideas o emplear las páginas del portal para buscar una configuración. La iteración frecuente de un diseño de índice es más eficaz si se sigue un enfoque basado en código para poder volver a crear el índice fácilmente.

> [!NOTE]
> Las API que usa para crear un índice tienen distintos comportamientos predeterminados. En el caso de las [API REST](https://docs.microsoft.com/rest/api/searchservice/Create-Index), la mayoría de los atributos están habilitados de forma predeterminada (por ejemplo, "buscable" y "recuperable" son verdaderos para los campos de cadena), y a menudo solo tiene que establecerlos si desea desactivarlos. Para el SDK de .NET, ocurre lo contrario. En cualquier propiedad que no establezca de forma explícita, el valor predeterminado es que el comportamiento de búsqueda correspondiente esté deshabilitado, a menos que lo habilite específicamente.

## `analyzers`

El elemento de analizadores establece el nombre del analizador de idioma que se utilizará para el campo. Para más información sobre el intervalo de analizadores disponibles, consulte [Adición de analizadores a un índice de Azure Cognitive Search](search-analyzers.md). Los analizadores solo pueden usarse con campos de que permiten búsquedas. Una vez que el analizador se asigna a un campo, no se puede cambiar a menos que se vuelva a generar el índice.

## `suggesters`

Un proveedor de sugerencias es una sección del esquema que define qué campos de un índice se utilizan para admitir consultas con la función Autocompletar o con escritura automática en las búsquedas. Normalmente, las cadenas de búsqueda parcial se envían a [Sugerencias (API REST)](https://docs.microsoft.com/rest/api/searchservice/suggestions) mientras el usuario escribe una consulta de búsqueda, y la API devuelve un conjunto de documentos o frases sugeridas. 

Los campos agregados a un proveedor de sugerencias se usan para compilar los términos de búsqueda de escritura anticipada. Todos los términos de búsqueda se crean durante la indexación y se almacenan por separado. Para más información sobre la creación de una estructura de proveedor de sugerencias, consulte [Incorporación de proveedores de sugerencias](index-add-suggesters.md).

## `corsOptions`

JavaScript del lado cliente no puede llamar a ninguna API de forma predeterminada debido a que el explorador impedirá todas las solicitudes entre orígenes. Para permitir consultas de origen cruzado en el índice, habilite CORS (uso compartido de recursos entre orígenes) estableciendo el atributo **corsOptions**. Por motivos de seguridad, solamente las API de consulta admiten CORS. 

Se pueden establecer las opciones siguientes para CORS:

+ **allowedOrigins** (obligatorio): se trata de una lista de orígenes a los que se le concederá acceso a su índice. Esto significa que cualquier código JavaScript que se suministre desde esos orígenes podrá consultar el índice (suponiendo que proporcione la clave de API correcta). Cada origen tiene normalmente el formato `protocol://<fully-qualified-domain-name>:<port>` aunque `<port>` se omite a menudo. Para más información, consulte [Uso compartido de recursos entre orígenes (wikipedia)](https://en.wikipedia.org/wiki/Cross-origin_resource_sharing).

  Si desea permitir el acceso a todos los orígenes, incluya `*` como elemento único en la matriz **allowedOrigins**. *Esto no es recomendable para los servicios de búsqueda de producción* pero a menudo resulta útil para el desarrollo y la depuración.

+ **maxAgeInSeconds** (opcional): los exploradores usan este valor para determinar la duración (en segundos) para almacenar en la memoria caché las respuestas preparatorias de CORS. Esto debe ser un entero no negativo. Cuanto mayor sea este valor es, mejor será el rendimiento, pero más tiempo tardarán en surtir efecto los cambios en la directiva CORS. Si no se establece, se usará una duración predeterminada de 5 minutos.

## `scoringProfiles`

Un [perfil de puntuación](index-add-scoring-profiles.md) es una sección del esquema que define comportamientos de puntuación personalizados que permiten influir en los elementos que aparecen más arriba en los resultados de la búsqueda. Los perfiles de puntuación se componen de ponderaciones de campos y de funciones. Para poder utilizarlos, especifique un perfil por nombre en la cadena de consulta.

El perfil de puntuación predeterminada funciona en segundo plano para calcular un resultado de búsqueda para todos los elementos de un conjunto de resultados. Puede usar un perfil de puntuación interno y sin nombre. Como alternativa, configure **defaultScoringProfile** para que use un perfil personalizado como valor predeterminado, al que se invoca cuando no se especifica un perfil personalizado en la cadena de consulta.

<a name="index-size"></a>

## <a name="attributes-and-index-size-storage-implications"></a>Atributos y tamaño de índice (implicaciones de almacenamiento)

El tamaño de un índice viene determinado por el tamaño de los documentos que se cargan, además de la configuración del índice, por ejemplo, si incluye los proveedores de sugerencias y cómo establece los atributos en los campos individuales. 

La siguiente captura de pantalla ilustra los patrones de almacenamiento de índices resultantes de diversas combinaciones de atributos. El índice se basa en el **índice de ejemplo de bienes inmuebles**, que puede crear fácilmente mediante el Asistente para la importación de datos. Aunque no se muestran los esquemas de índice, puede deducir los atributos según el nombre del índice. Por ejemplo, el índice *realestate-searchable* tiene seleccionado el atributo "buscable" y nada más, el índice *realestate-retrievable* tiene seleccionado el atributo "recuperable" y nada más y así sucesivamente.

![Tamaño del índice basado en la selección de atributos](./media/search-what-is-an-index/realestate-index-size.png "Tamaño del índice basado en la selección de atributos")

Aunque estas variantes de índice son artificiales, se puede hacer referencia a ellas en comparaciones más amplias de cómo los atributos afectan al almacenamiento. ¿El valor "recuperable" aumenta el tamaño del índice? No. ¿Agregar campos a un **proveedor de sugerencias** aumenta el tamaño del índice? Sí.

Los índices que admiten la ordenación y el filtrado son en proporción más grandes que los que solo admiten la búsqueda de texto completo. Esto se debe a que las operaciones de filtrado y ordenación detectan coincidencias exactas, lo que requiere la presencia de cadenas textuales de texto. En cambio, los campos que permiten búsquedas de texto completo usan índices invertidos, que se rellenan con términos acortados que consumen menos espacio que los documentos completos. 

> [!Note]
> La arquitectura de almacenamiento se considera un detalle de implementación de Azure Cognitive Search y podría cambiar sin previo aviso. No hay ninguna garantía de que el comportamiento actual se conserve en el futuro.

## <a name="next-steps"></a>Pasos siguientes

Con una descripción de la composición del índice, puede continuar en el portal para crear el primer índice. Se recomienda comenzar con el Asistente para la **importación de datos** y elegir los orígenes de datos hospedados *realestate-us-sample* o *hotels-sample*.

> [!div class="nextstepaction"]
> [Asistente para la importación de datos (portal)](search-get-started-portal.md)

En ambos conjuntos de datos, el asistente puede inferir un esquema de índice, importar los datos y generar un índice de búsqueda que se pueda consultar mediante el Explorador de búsqueda. Busque estos orígenes de datos en la página **Conectarse a los datos propios** del Asistente para la **importación de datos**.

   ![Creación de un índice de ejemplo](media/search-what-is-an-index//import-wizard-sample-data.png "Creación de un índice de ejemplo")