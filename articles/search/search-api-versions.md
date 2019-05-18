---
title: 'Administración de versiones de API para .NET SDK y API REST: Azure Search'
description: Directiva de versión para las API de REST de Azure Search y la biblioteca de cliente en el SDK de .NET.
author: brjohnstmsft
manager: jlembicz
services: search
ms.service: search
ms.devlang: dotnet
ms.topic: conceptual
ms.date: 05/02/2019
ms.author: brjohnst
ms.openlocfilehash: d72901653e995e811a1d3e89cef8a5f77a9ea8bd
ms.sourcegitcommit: bb85a238f7dbe1ef2b1acf1b6d368d2abdc89f10
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/10/2019
ms.locfileid: "65523804"
---
# <a name="api-versions-in-azure-search"></a>Versiones de API en Azure Search
Azure Search implementa las actualizaciones de características de forma regular. A veces, aunque no siempre, estas actualizaciones requieren una nueva versión de la API para mantener la compatibilidad con versiones anteriores. La publicación de una nueva versión le permite controlar cuándo y cómo integrar en su código las actualizaciones del servicio de búsqueda.

Por lo general, el equipo de Azure Search publica versiones nuevas solo cuando sea necesario, ya que el proceso de actualizar el código para que use una nueva versión de API puede suponer cierto esfuerzo. Solo se necesita una versión nueva si es necesario cambiar algún aspecto de la API de tal manera que se interrumpe la compatibilidad con versiones anteriores. Estos cambios pueden deberse a correcciones en las características existentes o a características nuevas que cambian el área expuesta de la API.

La misma regla se aplica a las actualizaciones del SDK. El SDK de Azure Search sigue las reglas de [Versionamiento semántico](https://semver.org/) , lo que significa que la versión tiene tres partes: principal, secundaria y número de compilación (por ejemplo, 1.1.0). Solo se lanzará una versión principal nueva del SDK en caso de que haya cambios que impidan la compatibilidad con versiones anteriores. En caso de actualizaciones de características que no producen interrupciones, incrementaremos la versión secundaria y, en caso de correcciones de errores, solo aumentaremos la versión de compilación.

> [!NOTE]
> La instancia del servicio Azure Search es compatible con varias versiones de API de REST, incluida la más reciente. Puede seguir usando una versión aunque no sea la más reciente, pero es recomendable que migre el código para usar la versión más actualizada. Cuando se usa la API de REST, debe especificar la versión de API en cada solicitud realizada a través del parámetro api-version. Al usar el SDK. NET, la versión del SDK que usa determina la versión correspondiente de la API de REST. Si usa un SDK anterior, puede seguir ejecutando ese código sin realizar ningún cambio, incluso si el servicio se ha actualizado para admitir una versión más reciente de la API.

## <a name="snapshot-of-current-versions"></a>Instantánea de las versiones actuales
A continuación se incluye una instantánea de las versiones actuales de todas las interfaces de programación para Azure Search.


| Interfaces | Versión principal más reciente | Status |
| --- | --- | --- |
| [.NET SDK](https://aka.ms/search-sdk) |9.0 |Disponibilidad general, lanzado en mayo de 2019 |
| [Versión preliminar del SDK de .NET](https://aka.ms/search-sdk-preview) |8.0-preview |Versión preliminar, publicada en abril de 2019 |
| [API de REST de servicio](https://docs.microsoft.com/rest/api/searchservice/) |2019-05-06 |Disponibilidad general |
| [Servicio REST API 2019-05-06-Preview](search-api-preview.md) |2019-05-06-preview |Preview |
| [SDK de administración de .NET](https://aka.ms/search-mgmt-sdk) |3.0 |Disponibilidad general |
| [API de REST de administración](https://docs.microsoft.com/rest/api/searchmanagement/) |2015-08-19 |Disponibilidad general |

Para las API de REST, es necesario incluir `api-version` en cada llamada. Usando `api-version`, se puede establecer fácilmente una versión específica de destino, como una versión preliminar de API. En el ejemplo siguiente se muestra cómo se especifica el parámetro `api-version` :

    GET https://my-demo-app.search.windows.net/indexes/hotels?api-version=2019-05-06

> [!NOTE]
> Aunque cada solicitud tiene una `api-version`, se recomienda usar la misma versión para todas las solicitudes de API. Esto ocurre especialmente cuando las nuevas versiones de API incorporan atributos u operaciones que las versiones anteriores no reconocen. Debe evitarse la mezcla de versiones de API, ya que puede tener consecuencias no deseadas.
>
> Las versiones de la API de REST de servicio y de la API de REST de administración son independientes entre sí. Cualquier semejanza entre los números de versión es mera coincidencia.

Las API de disponibilidad general (o GA) puede usarse en entornos de producción y están sujetas a contratos de nivel de servicio de Azure. Las versiones preliminares tienen características experimentales que no siempre se migran a una versión de disponibilidad general. **Le recomendamos encarecidamente que no use una versión preliminar de API en aplicaciones de producción.**

## <a name="about-preview-and-generally-available-versions"></a>Acerca de las versiones preliminar y de disponibilidad general
Azure Search siempre publica previamente características experimentales primero a través de la API de REST y, después, a través de las versiones preliminares del SDK de .NET.

Las características en versión preliminar están disponibles para prueba y experimentación con el fin de recabar comentarios sobre la implementación y el diseño de características. Por este motivo, las características en versión preliminar pueden cambiar con el tiempo, posiblemente de maneras que interrumpan la compatibilidad con versiones anteriores. Esto difiere de características en versión de disponibilidad general, que son estables y es poco probable que cambien, con la excepción de las mejoras y correcciones compatibles con versiones anteriores. Además, las características en versión preliminar no siempre terminando pasando a la fase de disponibilidad general.

Por estos motivos, no se recomienda escribir código de producción que dependa de versiones preliminares. Si usa una versión preliminar antigua, se recomienda que migre a la versión de disponibilidad general (GA).

Para .NET SDK: encontrará indicaciones para la migración de código en [Actualización de .NET SDK](search-dotnet-sdk-migration-version-9.md).

La disponibilidad general significa que Azure Search está ahora sujeta al contrato de nivel de servicio. Encontrará dicho contrato en el [Acuerdo de Nivel de Servicio de Azure Search](https://azure.microsoft.com/support/legal/sla/search/v1_0/).
