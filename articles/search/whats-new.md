---
title: Novedades de Azure Cognitive Search
description: Anuncios de características nuevas y mejoradas, como el cambio de nombre del servicio Azure Search por Azure Cognitive Search.
manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.service: cognitive-search
ms.topic: overview
ms.date: 08/01/2020
ms.custom: references_regions
ms.openlocfilehash: 60efc645b1b5824306c1d4aa3e763eb9d2bc745d
ms.sourcegitcommit: 1b2d1755b2bf85f97b27e8fbec2ffc2fcd345120
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/04/2020
ms.locfileid: "87553940"
---
# <a name="whats-new-in-azure-cognitive-search"></a>Novedades de Azure Cognitive Search

Conozca las novedades del servicio. Marque esta página para mantenerse actualizado con el servicio.

## <a name="feature-announcements-in-2020"></a>Anuncios de características en 2020

### <a name="august-2020"></a>Agosto de 2020

|Característica&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;  | Category | Descripción | Disponibilidad  |
|---------|------------------|-------------|---------------|
| [doble cifrado](search-security-overview.md#encryption) | Seguridad | Habilite el cifrado doble en la capa de almacenamiento. Para ello, debe configurar el cifrado de clave administrada por el cliente (CMK) en los nuevos servicios de búsqueda. Cree un servicio, [configure las claves administradas por el cliente y aplíquelas ](search-security-manage-encryption-keys.md) a índices o mapas de sinónimos, y benefíciese del cifrado doble en ese contenido. | Disponible con carácter general en todos los servicios de búsqueda creados después del 1 de agosto de 2020 en estas regiones: Oeste de EE. UU. 2, este de EE. UU., centro-sur de EE. UU., US Gov Virginia y US Gov Arizona. Use el portal, las API REST de administración o los SDK para crear el servicio. |

### <a name="july-2020"></a>Julio de 2020

|Característica&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;  | Category | Descripción | Disponibilidad  |
|---------|------------------|-------------|---------------|
| [Biblioteca cliente Azure.Search.Documents](https://docs.microsoft.com/dotnet/api/overview/azure/search.documents-readme?view=azure-dotnet) | SDK de Azure para .NET | Una biblioteca cliente de .NET publicada por el equipo de Azure SDK y diseñada para mantener la coherencia con otras bibliotecas cliente de .NET. <br/><br/>La versión 11 va dirigida a Search REST api-version=2020-06-30, pero aún no admite el almacén de conocimiento, filtros geográficos o [FieldBuilder](https://docs.microsoft.com/dotnet/api/microsoft.azure.search.fieldbuilder?view=azure-dotnet). | Disponible con carácter general. </br> Instale el [paquete Azure.Search.Documents](https://www.nuget.org/packages/Azure.Search.Documents/) desde NuGet. |
| [Biblioteca cliente azure.search.documents](https://docs.microsoft.com/python/api/overview/azure/search-documents-readme?view=azure-python)  | SDK de Azure para Python| Una biblioteca cliente de Python publicada por el equipo de Azure SDK y diseñada para mantener la coherencia con otras bibliotecas cliente de Python. <br/><br/>La versión 11 va dirigida a Search REST api-version=2020-06-30. | Disponible con carácter general. </br> Instale el [paquete azure-search-documents](https://pypi.org/project/azure-search-documents/) desde PyPI. |
| [Biblioteca cliente de@azure/search-documents](https://docs.microsoft.com/javascript/api/overview/azure/search-documents-readme?view=azure-node-latest)  | SDK de Azure para JavaScript | Biblioteca cliente de JavaScript publicada por el equipo de Azure SDK y diseñada para mantener la coherencia con otras bibliotecas cliente de JavaScript. <br/><br/>La versión 11 va dirigida a Search REST api-version=2020-06-30. | Disponible con carácter general. </br> Instale el [paquete @azure/search-documents](https://www.npmjs.com/package/@azure/search-documents) desde npm. |

### <a name="june-2020"></a>Junio de 2020

|Característica&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;  | Category | Descripción | Disponibilidad  |
|---------|------------------|-------------|---------------|
[**Almacén de conocimiento**](knowledge-store-concept-intro.md) | Enriquecimiento con inteligencia artificial | La salida de un indexador enriquecido con IA que almacena el contenido en Azure Storage para usarlo en otras aplicaciones y procesos. | Disponible con carácter general. </br> Use la [API REST de búsqueda 2020-06-30](https://docs.microsoft.com/rest/api/searchservice/) o posterior, o el portal. |
| [**API REST de búsqueda 2020-06-30**](https://docs.microsoft.com/rest/api/searchservice/) | REST | Una nueva versión estable de las API REST. Además del almacén de conocimiento, esta versión incluye mejoras en la relevancia y puntuación de las búsquedas. | Disponible con carácter general. |
| [**Algoritmo de relevancia Okapi BM25**](https://en.wikipedia.org/wiki/Okapi_BM25) | Consultar | Nuevo algoritmo de clasificación de relevancia que se usa automáticamente con todos los nuevos servicios de búsqueda creados después del 15 de julio. En el caso de los servicios creados con anterioridad, para optar a este algoritmo, establezca la propiedad `similarity` en los campos de índice. | Disponible con carácter general. </br> Use la [API REST de búsqueda 2020-06-30](https://docs.microsoft.com/rest/api/searchservice/) o posterior, o la API REST 2019-05-06. |
| **executionEnvironment** | Seguridad (indexadores) | Establezca explícitamente esta propiedad de configuración del indexador en `private` para forzar todas las conexiones a orígenes de datos externos a través de un punto de conexión privado. Solo se aplica a los servicios de búsqueda que aprovechan Azure Private Link. | Disponible con carácter general. </br> Use la [API REST de búsqueda 2020-06-30](https://docs.microsoft.com/rest/api/searchservice/) para establecer este parámetro de configuración general. |

### <a name="may-2020-microsoft-build"></a>Mayo de 2020 (Microsoft Build)

|Característica&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;  | Category | Descripción | Disponibilidad  |
|---------|------------------|-------------|---------------|
| [**Sesiones de depuración**](cognitive-search-debug-session.md) | Enriquecimiento con inteligencia artificial | Las sesiones de depuración proporcionan una interfaz basada en portal para investigar y resolver problemas con un conjunto de aptitudes existente. Las correcciones creadas en la sesión de depuración se pueden guardar en conjuntos de aptitudes de producción. Para empezar, realice [este tutorial](cognitive-search-tutorial-debug-sessions.md). | Versión preliminar pública, en el portal. |
| [**Reglas de IP para la compatibilidad con el firewall de entrada**](service-configure-firewall.md) | Seguridad | Limite el acceso de un punto de conexión de servicio de búsqueda a direcciones IP concretas. | Disponible con carácter general. </br> Use la [API REST de administración 2020-03-13](https://docs.microsoft.com/rest/api/searchmanagement/) o posterior, o el portal. |
| [**Azure Private Link para un punto de conexión de búsqueda privado**](service-create-private-endpoint.md) | Seguridad| Proteja los servicios de búsqueda de la red pública de Internet ejecutándolos como un recurso de vínculo privado, accesibles solo para aplicaciones cliente y otros servicios de Azure de la misma red virtual. | Disponible con carácter general. </br> Use la [API REST de administración 2020-03-13](https://docs.microsoft.com/rest/api/searchmanagement/) o posterior, o el portal. |
| [**identidad administrada por el sistema (versión preliminar)** ](search-howto-managed-identities-data-sources.md) | Seguridad (indexadores) | Registre los servicios de búsqueda como servicios de confianza con Azure Active Directory para configurar conexiones a un origen de datos de Azure admitido para la indexación. Se aplica a los [indexadores](search-indexer-overview.md) que ingieren contenido de orígenes de datos de Azure como Azure SQL Database, Azure Cosmos DB y Azure Storage. | Versión preliminar pública. </br> Use el portal para registrar el servicio de búsqueda. |
| [**parámetro de consulta sessionId**](index-similarity-and-scoring.md), [parámetro scoringStatistics=global](index-similarity-and-scoring.md#scoring-statistics) | Consulta (relevancia) | Agregue sessionID a las consultas a fin de establecer una sesión para el cálculo de puntuaciones de búsqueda, con scoringStatistics=global para recopilar puntuaciones de todas las particiones, con el objeto de aumentar la coherencia de dichos cálculos. | Disponible con carácter general. </br> Use la [API REST de búsqueda 2020-06-30](https://docs.microsoft.com/rest/api/searchservice/) o posterior, o la API REST 2019-05-06. |
| [**featuresMode (versión preliminar)** ](index-similarity-and-scoring.md#featuresMode-param) | Consultar | Agregue este parámetro de consulta para expandir una puntuación de relevancia y mostrar más detalles: la puntuación de similitud por campo, la frecuencia de los términos por campo y el número de tokens únicos coincidentes por campo. Puede consumir estos puntos de datos en algoritmos de puntuación personalizados. Para obtener un ejemplo que muestra esta funcionalidad, consulte [Adición de aprendizaje automático (LearnToRank) a la relevancia de búsqueda](https://github.com/Azure-Samples/search-ranking-tutorial). | Versión preliminar pública. </br> Use la [API REST de búsqueda 2020-06-30-Preview](https://docs.microsoft.com/rest/api/searchservice/index-preview) o la API REST 2019-05-06-Preview. |

### <a name="march-2020"></a>Marzo de 2020

|Característica&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;  | Category | Descripción | Disponibilidad  |
|---------|------------------|-------------|---------------|
| [**Eliminación temporal de blobs nativos (versión preliminar)** ](search-howto-indexing-azure-blob-storage.md#incremental-indexing-and-deletion-detection) | Indexadores | Un indexador de Azure Blob Storage de Azure Cognitive Search reconocerá los blobs que se encuentran en estado de eliminación temporal y quitará el documento de búsqueda correspondiente durante la indexación. | Versión preliminar pública. </br> Use la [API REST de búsqueda 2020-06-30-Preview](https://docs.microsoft.com/rest/api/searchservice/index-preview) y la API REST 2019-05-06-Preview, con la opción Ejecutar indexador en un origen de datos de Azure Blob que tenga habilitada la "eliminación temporal". |
| [**API REST de administración (2020-03-13)** ](https://docs.microsoft.com/rest/api/searchmanagement/management-api-versions) | REST | Nueva API REST estable para crear y administrar los servicios de búsqueda. Agrega compatibilidad con el firewall de IP y con Private Link. | Disponible con carácter general. |

### <a name="february-2020"></a>Febrero de 2020

|Característica&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;  | Category | Descripción | Disponibilidad  |
|---------|------------------|-------------|---------------|
| [**Detección de PII (versión preliminar)** ](cognitive-search-skill-pii-detection.md) | Enriquecimiento con inteligencia artificial | Nueva aptitud cognitiva utilizada durante la indexación que extrae información de identificación personal de un texto de entrada y le ofrece la opción de enmascararla a partir de ese texto de varias maneras. | Versión preliminar pública. </br> Use el portal o la [API REST de búsqueda 2020-06-30-Preview](https://docs.microsoft.com/rest/api/searchservice/index-preview) o la API REST 2019-05-06-Preview. |
| [**Búsqueda de entidades personalizadas (versión preliminar)** ](cognitive-search-skill-custom-entity-lookup.md )| Enriquecimiento con inteligencia artificial | Nueva aptitud cognitiva que busca texto de una lista de palabras y frases personalizada definida por el usuario. Con esta lista, se etiquetan todos los documentos con entidades coincidentes. La aptitud también admite un grado de coincidencia aproximada que se puede aplicar para buscar coincidencias similares, pero no exactas. | Versión preliminar pública. </br> Use el portal o la [API REST de búsqueda 2020-06-30-Preview](https://docs.microsoft.com/rest/api/searchservice/index-preview) o la API REST 2019-05-06-Preview. |

### <a name="january-2020"></a>Enero de 2020

|Característica&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;  | Category | Descripción | Disponibilidad  |
|---------|------------------|-------------|---------------|
| [**Claves de cifrado administradas por el cliente**](search-security-manage-encryption-keys.md) |Seguridad | Agrega una capa adicional de cifrado además del cifrado integrado de la plataforma. Con una clave de cifrado que se crea y administra, se puede cifrar el contenido del índice y los mapas de sinónimos antes de que la carga llegue a un servicio de búsqueda. | Disponible con carácter general. </br> Use la API REST de búsqueda 2019-05-06 o posterior. Para el código administrado, el paquete correcto sigue siendo el [SDK de .NET, versión 8.0-preview](search-dotnet-sdk-migration-version-9.md) aunque la característica no se incluya en la versión preliminar. |
| [**Reglas de IP para la compatibilidad con el firewall de entrada (versión preliminar)** ](service-configure-firewall.md) | Seguridad | Limite el acceso de un punto de conexión de servicio de búsqueda a direcciones IP concretas. La API de vista previa tiene las propiedades nuevas **IpRule** y **NetworkRuleSet** en la [API CreateOrUpdate](https://docs.microsoft.com/rest/api/searchmanagement/2019-10-01-preview/createorupdate-service). Esta característica en vista previa está disponible en regiones seleccionadas. |  Versión preliminar pública mediante api-version=2019-10-01-Preview.  |
| [**Azure Private Link para un punto de conexión de búsqueda privado (versión preliminar)** ](service-create-private-endpoint.md) | Seguridad| Proteja los servicios de búsqueda de la red pública de Internet ejecutándolos como un recurso de vínculo privado, accesibles solo para aplicaciones cliente y otros servicios de Azure de la misma red virtual. | Versión preliminar pública mediante api-version=2019-10-01-Preview.  |

## <a name="feature-announcements-in-2019"></a>Anuncios de características en 2019

### <a name="december-2019"></a>Diciembre de 2019

+ [Create Demo App (versión preliminar)](search-create-app-portal.md) (Crear aplicación de demostración [versión preliminar]) es un nuevo asistente del portal que genera un archivo HTML descargable con acceso de consulta (solo lectura) a un índice. El archivo incluye un script insertado que representa una aplicación web de estilo "localhost" operativa, enlazada a un índice en el servicio de búsqueda. Las páginas se pueden configurar en el asistente y pueden contener una barra de búsqueda, un área de resultados, una barra de navegación lateral y compatibilidad con consultas de escritura anticipada. Puede modificar el código HTML sin conexión para extender o personalizar el flujo de trabajo o la apariencia. Una aplicación de demostración no se extiende fácilmente para incluir las capas de seguridad y hospedaje que normalmente son necesarias en escenarios de producción. Debería considerarla una herramienta de validación y prueba, en lugar de un acceso directo a una aplicación cliente completa.

+ En [Creación de un punto de conexión privado para conexiones seguras (versión preliminar)](service-create-private-endpoint.md) se explica cómo configurar un vínculo privado para las conexiones seguras al servicio de búsqueda. Esta característica en vista previa está disponible a petición y usa [Azure Private Link](../private-link/private-link-overview.md) y [Azure Virtual Network](../virtual-network/virtual-networks-overview.md) como parte de la solución.

### <a name="november-2019---ignite-conference"></a>Noviembre de 2019: Conferencia Ignite

+ [Enriquecimiento incremental (versión preliminar)](cognitive-search-incremental-indexing-conceptual.md) agrega almacenamiento en caché y disponibilidad de estados a una canalización de enriquecimiento para que pueda trabajar en pasos o fases específicos sin perder el contenido que ya se ha procesado. Anteriormente, cualquier cambio en una canalización de enriquecimiento requería una recompilación completa. Con el enriquecimiento incremental, se conserva la salida del análisis costoso, especialmente el de las imágenes.

<!-- 
+ Custom Entity Lookup is a cognitive skill used during indexing that allows you to provide a list of custom entities (such as part numbers, diseases, or names of locations you care about) that should be found within the text. It supports fuzzy matching, case-insensitive matching, and entity synonyms. -->

+ La [extracción de documentos (versión preliminar)](cognitive-search-skill-document-extraction.md) es una aptitud cognitiva que se usa durante la indexación y que permite extraer el contenido de un archivo de un conjunto de aptitudes. Antes, el descifrado de documentos solo tenía lugar antes de la ejecución del conjunto de aptitudes. Con la incorporación de esta aptitud, también puede realizar esta operación dentro de la ejecución del conjunto de aptitudes.

+ La [traducción de texto](cognitive-search-skill-text-translation.md) es una aptitud cognitiva usada durante la indexación que evalúa el texto y devuelve para cada registro el texto traducido al idioma de destino especificado.

+ Las [plantillas de Power BI](https://github.com/Azure-Samples/cognitive-search-templates/blob/master/README.md) pueden impulsar las visualizaciones y el análisis de contenido enriquecido en un almacén de conocimiento en Power BI Desktop. Esta plantilla está diseñada para las proyecciones de tablas de Azure creadas mediante el [Asistente para importar datos](knowledge-store-create-portal.md).

+ [Azure Data Lake Storage Gen2 (versión preliminar)](search-howto-index-azure-data-lake-storage.md), [Gremlin API de Cosmos DB (versión preliminar)](search-howto-index-cosmosdb.md) y [Cassandra API de Cosmos DB (versión preliminar)](search-howto-index-cosmosdb.md) ahora se admiten en los indexadores. Para registrarse, utilice [este formulario](https://aka.ms/azure-cognitive-search/indexer-preview). Recibirá un correo de confirmación cuando se le haya aceptado en el programa de versión preliminar.

### <a name="july-2019"></a>Julio de 2019

+ Está disponible generalmente en la [nube de Azure Government](../azure-government/documentation-government-services-webandmobile.md#azure-cognitive-search).

<a name="new-service-name"></a>

## <a name="new-service-name"></a>Nuevo nombre de servicio

Azure Search se llama ahora **Azure Cognitive Search** para reflejar el uso ampliado (aunque opcional) de las aptitudes cognitivas y el procesamiento de inteligencia artificial en las operaciones principales. Las versiones de API, los paquetes NuGet, los espacios de nombres y los puntos de conexión permanecen sin cambios. Las soluciones de búsqueda nuevas y las existentes no se ven afectadas por el cambio de nombre del servicio.

## <a name="service-updates"></a>Actualizaciones del servicio

Los [anuncios de actualización de servicios](https://azure.microsoft.com/updates/?product=search&status=all) de Azure Cognitive Search se pueden encontrar en el sitio web de Azure.