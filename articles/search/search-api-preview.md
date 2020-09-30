---
title: Lista de características en versión preliminar
titleSuffix: Azure Cognitive Search
description: Se publican características en versión preliminar para que los clientes puedan proporcionar comentarios sobre el diseño y la utilidad. En este artículo se incluye una lista completa de todas las características de que actualmente se encuentran en versión preliminar.
manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 09/22/2020
ms.openlocfilehash: ac0b04db783571b6ef31161adaf18b4220244c6a
ms.sourcegitcommit: bdd5c76457b0f0504f4f679a316b959dcfabf1ef
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90969811"
---
# <a name="preview-features-in-azure-cognitive-search"></a>Características en versión preliminar de Azure Cognitive Search

En este artículo se incluye una lista completa de todas las características que se encuentran en versión preliminar pública. La funcionalidad de versión preliminar se ofrece sin un Acuerdo de Nivel de Servicio y no es aconsejable usarla para cargas de trabajo de producción. Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

Se han quitado de la lista las características en versión preliminar que pasan a disponibilidad general. Si a continuación no aparece una característica, puede suponer que está disponible con carácter general. Para ver los anuncios relativos a la disponibilidad general, consulte [Actualizaciones del servicio](https://azure.microsoft.com/updates/?product=search) o [Novedades](whats-new.md).

|Característica&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;  | Category | Descripción | Disponibilidad  |
|---------|------------------|-------------|---------------|
| [**Aptitud de Azure Machine Learning (AML)** ](cognitive-search-aml-skill.md) | Enriquecimiento con inteligencia artificial| Nuevo tipo de aptitud para integrar un punto de conexión de inferencia desde Azure Machine Learning. Para empezar, realice [este tutorial](cognitive-search-tutorial-aml-custom-skill.md). | Use la [API REST de búsqueda 2020-06-30-Preview](https://docs.microsoft.com/rest/api/searchservice/) o 2019-05-06-Preview. También está disponible en el portal, en el diseño de aptitudes, suponiendo que Cognitive Search y los servicios Azure Machine Learning estén implementados en la misma suscripción. |
| [**Parámetro featuresMode**](https://docs.microsoft.com/rest/api/searchservice/search-documents#featuresmode) | Relevancia (puntuación) | Expansión de puntuación de relevancia para incluir detalles: puntuación de similitud por campo, frecuencia de términos por campo y número de tokens únicos coincidentes por campo. Puede consumir estos puntos de datos en [soluciones de puntuación personalizadas](https://github.com/Azure-Samples/search-ranking-tutorial). | Agregue este parámetro de consulta mediante [Documentos de búsqueda (REST)](https://docs.microsoft.com/rest/api/searchservice/search-documents) con api-version=2020-06-30-Preview o 2019-05-06-Preview. |
| [**Sesiones de depuración**](cognitive-search-debug-session.md) | Portal, enriquecimiento con IA (conjunto de aptitudes) | Editor de conjunto de aptitudes en sesión que se usa para investigar y resolver problemas en un conjunto de aptitudes. Las correcciones que se aplican durante una sesión de depuración se pueden guardar en un conjunto de aptitudes en el servicio. | Solo en el portal, mediante los vínculos de página intermedia en la página de información general para abrir una sesión de depuración. |
| [**Eliminación temporal de blobs nativos**](search-howto-indexing-azure-blob-storage.md#incremental-indexing-and-deletion-detection) | Indexadores, blobs de Azure| El indexador de Azure Blob Storage en Azure Cognitive Search reconocerá los blobs que se encuentran en un estado de eliminación temporal y quitará el documento de búsqueda correspondiente durante la indexación. | Agregue esta opción de configuración mediante [Creación de indexador (REST)](/rest/api/searchservice/create-indexer) con api-version=2020-06-30-Preview o api-version=2019-05-06-Preview. |
| [**Aptitud de búsqueda de entidades personalizadas**](cognitive-search-skill-custom-entity-lookup.md ) | Enriquecimiento con IA (conjunto de aptitudes) | Aptitud cognitiva que busca texto de una lista de palabras y frases personalizada definida por el usuario. Con esta lista, se etiquetan todos los documentos con entidades coincidentes. La aptitud también admite un grado de coincidencia aproximada que se puede aplicar para buscar coincidencias similares pero no exactas. | Haga referencia a esta aptitud en versión preliminar con el editor de conjuntos de aptitudes en el portal o [Creación de conjuntos de aptitudes (REST)](/rest/api/searchservice/create-skillset) con api-version=2020-06-30-Preview o api-version=2019-05-06-Preview. |
| [**Aptitud de detección de PII**](cognitive-search-skill-pii-detection.md) | Enriquecimiento con IA (conjunto de aptitudes) | Aptitud cognitiva utilizada durante la indexación que extrae información de identificación personal de un texto de entrada y le ofrece la opción de enmascararla a partir de ese texto de varias maneras. | Haga referencia a esta aptitud en versión preliminar con el editor de conjuntos de aptitudes en el portal o [Creación de conjuntos de aptitudes (REST)](/rest/api/searchservice/create-skillset) con api-version=2020-06-30-Preview o api-version=2019-05-06-Preview. |
| [**Enriquecimiento incremental**](cognitive-search-incremental-indexing-conceptual.md) | Configuración del indexador| Agrega almacenamiento en caché a una canalización de enriquecimiento, lo que le permite reutilizar los resultados existentes si una modificación de destino, como una actualización de conjunto de aptitudes u otro objeto, no cambia el contenido. El almacenamiento en caché solo se aplica a los documentos enriquecidos generados por un conjunto de aptitudes.| Agregue esta opción de configuración mediante [Creación de indexador (REST)](/rest/api/searchservice/create-indexer) con api-version=2020-06-30-Preview o api-version=2019-05-06-Preview. |
| [**Indexador de Cosmos DB: API de MongoDB, API de Gremlin, API de Cassandra**](search-howto-index-cosmosdb.md) | Origen de datos del indexador | Para Cosmos DB, la API de SQL está disponible con carácter general, pero las API de MongoDB, Gremlin y Cassandra se encuentran en versión preliminar. | Solo para Gremlin y Cassandra, [regístrese primero](https://aka.ms/azure-cognitive-search/indexer-preview) para poder habilitar el soporte técnico para su suscripción en el back-end. Los orígenes de datos de MongoDB se pueden configurar en el portal. De lo contrario, se admite la configuración de orígenes de datos para las tres API mediante [Creación de orígenes de datos (REST)](/rest/api/searchservice/create-data-source) con api-version=2020-06-30-Preview o api-version=2019-05-06-Preview. |
|  [**Indexador de Azure Data Lake Storage Gen2**](search-howto-index-azure-data-lake-storage.md) | Origen de datos del indexador | Indiza contenido y metadatos de Azure Data Lake Storage Gen2.| Es necesario [registrarse](https://aka.ms/azure-cognitive-search/indexer-preview) para que se pueda habilitar el soporte técnico para su suscripción en el back-end. Acceda a este origen de datos mediante [Creación de orígenes de datos (REST)](/rest/api/searchservice/create-data-source) con api-version=2020-06-30-Preview o api-version=2019-05-06-Preview. |
| [**moreLikeThis**](search-more-like-this.md) | Consultar | Busca documentos que sean pertinentes para un documento específico. Esta característica ha aparecido en versiones anteriores. | Agregue este parámetro de consulta mediante [Documentos de búsqueda (REST)](/rest/api/searchservice/search-documents) con api-version=2020-06-30-Preview, 2019-05-06-Preview, 2016-09-01-Preview o 2017-11-11-Preview. |

## <a name="how-to-call-a-preview-rest-api"></a>Llamada a una API REST en versión preliminar

Azure Cognitive Search siempre publica previamente características experimentales primero a través de la API REST y, después, a través de las versiones preliminares del SDK de .NET.

Las características en versión preliminar están disponibles para prueba y experimentación con el fin de recabar comentarios sobre la implementación y el diseño de características. Por este motivo, las características en versión preliminar pueden cambiar con el tiempo, posiblemente de maneras que interrumpan la compatibilidad con versiones anteriores. Esto difiere de características en versión de disponibilidad general, que son estables y es poco probable que cambien, con la excepción de las mejoras y correcciones compatibles con versiones anteriores. Además, las características en versión preliminar no siempre terminando pasando a la fase de disponibilidad general.

Aunque algunas características en versión preliminar pueden estar disponibles en el portal y en el SDK de .NET, la API REST siempre tiene características en versión preliminar.

+ En el caso de las operaciones de búsqueda, la versión preliminar actual es [ **`2020-06-30-Preview`** ](/rest/api/searchservice/index-preview).

+ En el caso de las operaciones de administración, la versión preliminar actual es [ **`2019-10-01-Preview`** ](/rest/api/searchmanagement/index-2019-10-01-preview).

Las vistas previas anteriores siguen funcionando, pero quedarán obsoletas con el paso del tiempo. Si el código llama a `api-version=2019-05-06-Preview` o `api-version=2016-09-01-Preview` o `api-version=2017-11-11-Preview`, dichas llamadas siguen siendo válidas. Sin embargo, solo la versión preliminar más reciente se actualiza con mejoras.

En la sintaxis de ejemplo siguiente se ilustra una llamada a la versión de API de versión preliminar.

```HTTP
POST https://[service name].search.windows.net/indexes/hotels-idx/docs/search?api-version=2020-06-30-Preview  
  Content-Type: application/json  
  api-key: [admin key]
```

El servicio Azure Cognitive Search está disponible en varias versiones. Para obtener más información, consulte [Versiones de API](search-api-versions.md).

## <a name="next-steps"></a>Pasos siguientes

Revise la documentación de referencia de la API de REST en versión preliminar del servicio Search. Si tiene algún problema, puede solicitar ayuda en [Stack Overflow](https://stackoverflow.com/) o [ponerse en contacto con el servicio de soporte técnico](https://azure.microsoft.com/support/community/?product=search).

> [!div class="nextstepaction"]
> [Referencia de la API de REST del servicio Search (versión preliminar)](/rest/api/searchservice/index-preview)