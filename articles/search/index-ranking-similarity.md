---
title: Algoritmo de similitud de clasificación
titleSuffix: Azure Cognitive Search
description: Cómo establecer el algoritmo de similitud para probar el nuevo algoritmo de similitud para la clasificación
manager: nitinme
author: luiscabrer
ms.author: luisca
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 03/13/2020
ms.openlocfilehash: ee788125a5710e5a8b9861c28c4af8e6d83d5d80
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88924544"
---
# <a name="ranking-algorithm-in-azure-cognitive-search"></a>Algoritmo de clasificación de Azure Cognitive Search

> [!IMPORTANT]
> A partir del 15 de julio de 2020, los servicios de búsqueda recién creados utilizarán la función de categoría BM25 automáticamente, que, en la mayoría de los casos, ha demostrado proporcionar clasificaciones de búsqueda que se ajustan mejor a las expectativas de los usuarios que la clasificación predeterminada actual. Más allá de la clasificación superior, BM25 también habilita las opciones de configuración para optimizar los resultados en función de factores como el tamaño del documento.  
>
> Con este cambio, lo más probable es que vea pequeños cambios en el orden de los resultados de la búsqueda. Para aquellos que desean probar el impacto de este cambio, el algoritmo BM25 está disponible en la versión de API 2019-05-06-Preview y en 2020-06-30.  

En este artículo se describe cómo puede usar el nuevo algoritmo de clasificación BM25 en los servicios de búsqueda existentes para los nuevos índices creados y consultados mediante la API de versión preliminar.

Azure Cognitive Search está en proceso de adoptar la implementación oficial de Lucene del algoritmo BM25 de Okapi, *BM25Similarity*, que reemplazará a la implementación de *ClassicSimilarity* usada anteriormente. Al igual que el algoritmo ClassicSimilarity anterior, BM25Similarity es una función de recuperación de tipo TF-IDF que usa la frecuencia de términos (TF) y la frecuencia de documentos inversos (IDF) como variables para calcular las puntuaciones de relevancia de cada par documento-consulta, que posteriormente se usa para la clasificación. 

A pesar de que conceptualmente es similar al algoritmo Classic Similarity más antiguo, BM25 toma su raíz de la recuperación de información probabilística para mejorarla. BM25 también ofrece opciones de personalización avanzadas, como permitir que el usuario decida cómo escala la puntuación de relevancia con la frecuencia de los términos coincidentes.

## <a name="how-to-test-bm25-today"></a>Cómo probar BM25 hoy

Al crear un índice, puede establecer una propiedad **similarity** para especificar el algoritmo. Puede usar `api-version=2019-05-06-Preview`, como se muestra a continuación, o `api-version=2020-06-30`

```
PUT https://[search service name].search.windows.net/indexes/[index name]?api-version=2019-05-06-Preview
```

```json  
{
    "name": "indexName",
    "fields": [
        {
            "name": "id",
            "type": "Edm.String",
            "key": true
        },
        {
            "name": "name",
            "type": "Edm.String",
            "searchable": true,
            "analyzer": "en.lucene"
        },
        ...
    ],
    "similarity": {
        "@odata.type": "#Microsoft.Azure.Search.BM25Similarity"
    }
}
```

La propiedad **similarity** resulta útil durante este período provisional cuando ambos algoritmos están disponibles, solo en los servicios existentes. 

| Propiedad | Descripción |
|----------|-------------|
| similarity | Opcional. Entre los valores válidos se incluyen *"#Microsoft.Azure.Search.ClassicSimilarity"* o *"#Microsoft.Azure.Search.BM25Similarity"* . <br/> Requiere `api-version=2019-05-06-Preview` o posterior en un servicio de búsqueda creado antes del 15 de julio de 2020. |

En el caso de los servicios creados después del 15 de julio de 2020, BM25 se usa automáticamente y es el único algoritmo similarity. Si intenta establecer **similarity** en `ClassicSimilarity` en un nuevo servicio, se devolverá un error 400 porque ese algoritmo no se admite en un nuevo servicio.

En el caso de los servicios existentes creados antes del 15 de julio de 2020, el algoritmo similarity clásico sigue siendo el predeterminado. Si se omite la propiedad **similarity** o se establece en NULL, el índice usará el algoritmo clásico. Si desea usar el nuevo algoritmo, tendrá que establecer **similarity** como se ha descrito anteriormente.

## <a name="bm25-similarity-parameters"></a>Parámetros de similitud de BM25

La similitud de BM25 agrega dos parámetros personalizables por el usuario para controlar la puntuación de relevancia calculada.

### <a name="k1"></a>k1

El parámetro *k1* controla la función de escalado entre la frecuencia de cada uno de los términos coincidentes y la puntuación de relevancia final de un par documento-consulta.

Un valor de cero representa un "modelo binario", en el que la contribución de un único término coincidente es la misma para todos los documentos coincidentes, independientemente del número de veces que ese término aparezca en el texto, mientras que un valor k1 mayor permite que la puntuación siga aumentando a medida que se encuentren más instancias del mismo término en el documento. De forma predeterminada, Azure Cognitive Search usa un valor de 1,2 para el parámetro k1. El uso de un valor de k1 mayor puede ser importante en los casos en los que se espera que varios términos formen parte de una consulta de búsqueda. En estos casos, podemos favorecer documentos que coincidan con muchos de los diferentes términos de consulta buscados frente a los documentos que solo coinciden con uno varias veces. Por ejemplo, al consultar el índice de los documentos que contienen los términos "Apollo Spaceflight", podemos reducir la puntuación de un artículo sobre la mitología griega que contiene el término "Apollo" algunas decenas de veces sin menciones a "Spaceflight", en comparación con otro artículo que menciona explícitamente "Apollo" y "Spaceflight" solo unas pocas veces. 
 
### <a name="b"></a>b

El parámetro *b* controla cómo afecta la longitud de un documento a la puntuación de relevancia.

Un valor de 0,0 significa que la longitud del documento no influye en la puntuación, mientras que un valor de 1,0 significa que el impacto de la frecuencia de los términos en la puntuación de relevancia se normalizará según la longitud del documento. El valor predeterminado que se usa en Azure Cognitive Search para el parámetro b es 0,75. La normalización de la frecuencia de los términos según la longitud del documento es útil en los casos en los que queremos penalizar los documentos más largos. En algunos casos, es más probable que los documentos más largos (como una novela completa) contengan muchos términos irrelevantes, en comparación con documentos mucho más cortos.

### <a name="setting-k1-and-b-parameters"></a>Establecimiento de los parámetros k1 y b

Para personalizar los valores de b o k1, simplemente agréguelos como propiedades al objeto de similitud cuando use BM25:

```json
    "similarity": {
        "@odata.type": "#Microsoft.Azure.Search.BM25Similarity",
        "b" : 0.5,
        "k1" : 1.3
    }
```

El algoritmo de similitud solo se puede establecer en el momento de crear el índice. Esto significa que no se puede cambiar el algoritmo de similitud que se usa para los índices existentes. Los parámetros *"b"* y *"k1"* se pueden modificar al actualizar una definición de índice existente que usa BM25. El cambio de estos valores en un índice existente dejará el índice sin conexión durante al menos unos segundos, lo que provocará errores en las solicitudes de indexación y consulta. Por eso, deberá establecer el parámetro "allowIndexDowntime=true" en la cadena de consulta de la solicitud de actualización:

```http
PUT https://[search service name].search.windows.net/indexes/[index name]?api-version=[api-version]&allowIndexDowntime=true
```

## <a name="see-also"></a>Consulte también  

+ [Referencia de API de REST](/rest/api/searchservice/)   
+ [Adición de perfiles de puntuación al índice](index-add-scoring-profiles.md)    
+ [Create Index API](/rest/api/searchservice/create-index)   
+ [SDK de .NET de Azure Cognitive Search](/dotnet/api/overview/azure/search?view=azure-dotnet)