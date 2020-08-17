---
title: Versiones de API
titleSuffix: Azure Cognitive Search
description: Directiva de versión para las API REST de Azure Cognitive Search y la biblioteca de cliente en el SDK de .NET.
manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 07/20/2020
ms.openlocfilehash: 5be50453dff9acaf4a9876eec1d95b56abebf745
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87029848"
---
# <a name="api-versions-in-azure-cognitive-search"></a>Versiones de API en Azure Cognitive Search

Azure Cognitive Search implementa las actualizaciones de características de forma regular. A veces, aunque no siempre, estas actualizaciones requieren una nueva versión de la API para mantener la compatibilidad con versiones anteriores. La publicación de una nueva versión le permite controlar cuándo y cómo integrar en su código las actualizaciones del servicio de búsqueda.

Por lo general, el equipo de Azure Cognitive Search publica versiones nuevas solo cuando sea necesario, ya que el proceso de actualizar el código para que use una nueva versión de API puede suponer cierto esfuerzo. Solo se necesita una versión nueva si es necesario cambiar algún aspecto de la API de tal manera que se interrumpe la compatibilidad con versiones anteriores. Estos cambios pueden deberse a correcciones en las características existentes o a características nuevas que cambian el área expuesta de la API.

La misma regla se aplica a las actualizaciones del SDK. El SDK de Azure Cognitive Search sigue las reglas de [versionamiento semántico](https://semver.org/), lo que significa que la versión tiene tres partes: principal, secundaria y número de compilación (por ejemplo, 1.1.0). Solo se lanzará una versión principal nueva del SDK en caso de que haya cambios que impidan la compatibilidad con versiones anteriores. En caso de actualizaciones de características que no producen interrupciones, incrementaremos la versión secundaria y, en caso de correcciones de errores, solo aumentaremos la versión de compilación.

> [!Important]
> Los SDK de Azure para .NET, Java, Python y JavaScript están implementando nuevas bibliotecas de cliente para Azure Cognitive Search. Actualmente, ninguna de las bibliotecas de SDK de Azure admite totalmente las API REST de búsqueda más recientes (2020-06-30) o las API REST de administración (2020-03-13), pero esto cambiará en el futuro. Puede consultar periódicamente esta página o las [Novedades](whats-new.md) para estar al tanto de los anuncios sobre mejoras funcionales. 

## <a name="rest-apis"></a>API de REST

Una instancia del servicio Search de Azure Cognitive es compatible con varias versiones de API de REST, incluida la más reciente. Puede seguir usando una versión aunque no sea la más reciente, pero es recomendable que [migre el código](search-api-migration.md) para usar la versión más actualizada. Cuando se usa la API de REST, debe especificar la versión de API en cada solicitud realizada a través del parámetro api-version. Al usar el SDK de .NET, la versión del SDK que usa determina la versión correspondiente de la API de REST. Si usa un SDK anterior, puede seguir ejecutando ese código sin realizar ningún cambio, incluso si el servicio se ha actualizado para admitir una versión más reciente de la API.

En la siguiente tabla se proporciona el historial de versiones de las versiones actuales y publicadas anteriormente de la API REST del servicio Search. La documentación se publica solo para las versiones estables más recientes y en versión preliminar.

### <a name="search-service-apis"></a>API del servicio Search

Cree y administre contenido en un servicio de búsqueda.

| Versión&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;   | Estado | Problema de compatibilidad con versiones anteriores |
|-------------------------|--------|------------------------------|
| [Search 2020-06-30](https://docs.microsoft.com/rest/api/searchservice/index)| Stable | La versión estable más reciente de las API REST del servicio Search, con avances en la puntuación de relevancia y disponibilidad general para el almacén de conocimiento.|
| [Search 2020-06-30-Preview](https://docs.microsoft.com/rest/api/searchservice/index-preview)| Versión preliminar | Versión preliminar asociada a la versión estable. Incluye varias [características en vista previa](search-api-preview.md). |
| Search 2019-05-06 | Stable | Agrega [tipos complejos](search-howto-complex-data-types.md). |
| Search 2019-05-06-Preview | Versión preliminar | Versión preliminar asociada a la versión estable. |
| Search 2017-11-11 | Stable  | Agrega conjuntos de aptitudes y [enriquecimiento con IA](cognitive-search-concept-intro.md). |
| Search 2017-11-11-Preview | Versión preliminar | Versión preliminar asociada a la versión estable. |
| Search 2016-09-01 |Stable | Agrega [indexadores](search-indexer-overview.md). |
| Search 2016-09-01-Preview | Versión preliminar | Versión preliminar asociada a la versión estable.|
| Search 2015-02-28 | Stable  | Primera versión con disponibilidad general.  |
| Search 2015-02-28-Preview | Versión preliminar | Versión preliminar asociada a la versión estable. |
| Search 2014-10-20-Preview | Versión preliminar | Segunda versión preliminar pública. |
| Search 2014-07-31-Preview | Versión preliminar | Primera versión preliminar pública. |

### <a name="management-rest-apis"></a>API de REST de administración

Crea y configura un servicio de búsqueda y administra claves de API.

| Versión&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;   | Estado | Problema de compatibilidad con versiones anteriores |
|-------------------------|--------|------------------------------|
| [Administración 2020-03-13](https://docs.microsoft.com/rest/api/searchmanagement/) | Stable | Versión estable más reciente de las API de REST de administración, con avances en la protección de puntos de conexión. Agrega un [punto de conexión privado](service-create-private-endpoint.md) mediante un vínculo privado y [reglas de IP de red](service-configure-firewall.md) para nuevos servicios. |
| [Administración 2019-10-01-Preview](https://docs.microsoft.com/rest/api/searchmanagement/index-2019-10-01-preview) | Versión preliminar  | A pesar de su número de versión, sigue siendo la versión preliminar actual de las API de REST de administración. No hay ninguna característica en versión preliminar en este momento. Todas las características en versión preliminar han pasado recientemente a disponibilidad general. |
| Administración 2015-08-19  | Stable | La primera versión disponible con carácter general de las API de REST de administración. Proporciona aprovisionamiento de servicios, escalado vertical y administración de claves de API. |
| Administración 2015-08-19-Preview | Versión preliminar | La primera versión preliminar de las API de REST de administración. |

## <a name="azure-sdk-for-net"></a>SDK de Azure para .NET

El historial de versiones del paquete está disponible en NuGet.org. En esta tabla se proporcionan vínculos a cada página del paquete.


| Versión del SDK | Estado | Descripción |
|-------------|--------|------------------------------|
| [Azure.Search.Documents 11.0](https://www.nuget.org/packages/Azure.Search.Documents/1.0.0-preview.4) | Stable | Nueva biblioteca cliente del SDK de .NET de Azure, publicada en julio de 2020. Tiene como destino la API REST de búsqueda API-version = 2020-06-30, pero aún no admite, filtros geográficos ni [FieldBuilder](https://docs.microsoft.com/dotnet/api/microsoft.azure.search.fieldbuilder?view=azure-dotnet). |
| [Microsoft.Azure.Search 10.0](https://www.nuget.org/packages/Microsoft.Azure.Search/) | Stable | Publicada en mayo de 2019. Tiene como destino la API REST del servicio Search api-version=2019-05-06.|
| [Microsoft.Azure.Search 8.0-preview](https://www.nuget.org/packages/Microsoft.Azure.Search/8.0.0-preview) | Versión preliminar | Publicada en abril de 2019. Tiene como destino la API REST del servicio Search api-version=2019-05-06-Preview.|
| [Microsoft.Azure.Management.Search 3.0.0](https://docs.microsoft.com/dotnet/api/overview/azure/search/management?view=azure-dotnet) | Stable | Tiene como destino la API de REST de administración api-version=2015-08-19.  |

## <a name="azure-sdk-for-java"></a>SDK de Azure para Java

| Versión del SDK | Estado | Descripción  |
|-------------|--------|------------------------------|
| [Java azure-search-documents 11](https://azuresdkdocs.blob.core.windows.net/$web/java/azure-search-documents/11.0.0/index.html) | Stable | Nueva biblioteca cliente del SDK de .NET de Azure, publicada en julio de 2020. Tiene como destino la API REST del servicio Search api-version=2019-05-06. |
| [Java Management Client 1.35.0](https://docs.microsoft.com/java/api/overview/azure/search/management?view=azure-java-stable) | Stable | Tiene como destino la API de REST de administración api-version=2015-08-19. |

## <a name="azure-sdk-for-javascript"></a>SDK de Azure para JavaScript

| Versión del SDK | Estado | Descripción  |
|-------------|--------|------------------------------|
| [JavaScript azure-search 11.0](https://azure.github.io/azure-sdk-for-node/azure-search/latest/) | Stable | Nueva biblioteca cliente del SDK de .NET de Azure, publicada en julio de 2020. Tiene como destino la API REST del servicio Search api-version=2016-09-01. |
| [JavaScript azure-arm-search](https://azure.github.io/azure-sdk-for-node/azure-arm-search/latest/) | Stable | Tiene como destino la API de REST de administración api-version=2015-08-19. |

## <a name="azure-sdk-for-python"></a>SDK de Azure para Python

| Versión del SDK | Estado | Descripción  |
|-------------|--------|------------------------------|
| [Python azure-search-documents 11.0](https://azuresdkdocs.blob.core.windows.net/$web/python/azure-search-documents/11.0.0/index.html) | Stable | Nueva biblioteca cliente del SDK de .NET de Azure, publicada en julio de 2020. Tiene como destino la API REST del servicio Search api-version=2019-05-06. |
| [Python azure-mgmt-search 1.0](https://docs.microsoft.com/python/api/overview/azure/search?view=azure-python) | Stable | Tiene como destino la API de REST de administración api-version=2015-08-19. |