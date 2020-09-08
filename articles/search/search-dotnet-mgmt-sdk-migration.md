---
title: Actualización al SDK de administración de .NET para Azure Search
titleSuffix: Azure Cognitive Search
description: Actualización al SDK de administración de .NET para Azure Search desde versiones anteriores. Obtenga información sobre las nuevas características y los cambios de código necesarios para la migración.
manager: nitinme
author: brjohnstmsft
ms.author: brjohnst
ms.service: cognitive-search
ms.devlang: dotnet
ms.topic: conceptual
ms.date: 07/08/2020
ms.openlocfilehash: 8648347eb48081389cf360fa949b31bbd0b8c71e
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88936714"
---
# <a name="upgrading-versions-of-the-azure-search-net-management-sdk"></a>Actualización de las versiones del SDK de administración de .NET para Azure Search

En este artículo se explica cómo migrar a versiones sucesivas del SDK de administración de .NET para Azure Search, que se usa para el aprovisionamiento o desaprovisionamiento de servicios de búsqueda, el ajuste de capacidad y la administración de claves de API.

Los SDK de administración están destinados a una versión específica de la API de REST de administración. Para obtener más información sobre los conceptos y las operaciones, vea [Administración de búsquedas (REST)](/rest/api/searchmanagement/).

## <a name="versions"></a>Versiones

| Versión del SDK | Versión correspondiente de la API de REST | Adición de características o cambio de comportamiento |
|-------------|--------------------------------|-------------------------------------|
| [3.0](https://www.nuget.org/packages/Microsoft.Azure.Management.Search/3.0.0) | api-version=2020-30-20 | Agrega seguridad de punto de conexión (firewalls de IP e integración con [Azure Private Link](../private-link/private-endpoint-overview.md)) |
| [2.0](https://www.nuget.org/packages/Microsoft.Azure.Management.Search/2.0.0) | api-version=2019-10-01 | Mejoras de la facilidad de uso. Cambio importante en las [claves de consulta de la lista](/rest/api/searchmanagement/querykeys/listbysearchservice) (GET ya no está disponible). |
| [1.0](https://www.nuget.org/packages/Microsoft.Azure.Management.Search/1.0.1) | api-version=2015-08-19  | Primera versión |

## <a name="how-to-upgrade"></a>Procedimiento de actualización

1. Actualice la referencia de NuGet para `Microsoft.Azure.Management.Search` mediante la Consola del Administrador de paquetes NuGet, o bien haga clic con el botón derecho en las referencias del proyecto y seleccione "Administrar paquetes NuGet" en Visual Studio.

1. Una vez que NuGet ha descargado los nuevos paquetes y sus dependencias, recompile el proyecto. En función de cómo se estructure el código, se podrá volver a compilar correctamente, en cuyo caso habrá terminado.

1. Si se produce un error en la compilación, podría deberse a que implementó algunas de las interfaces del SDK (por ejemplo, para las pruebas unitarias), las que han cambiado. Para resolver esto, deberá implementar métodos más recientes, como `BeginCreateOrUpdateWithHttpMessagesAsync`.

1. Después de corregir los errores de compilación, puede realizar cambios en la aplicación para aprovechar las ventajas de la nueva funcionalidad. 

## <a name="upgrade-to-30"></a>Actualización a 3.0

La versión 3.0 agrega protección de punto de conexión privada mediante la restricción del acceso a los intervalos de IP y, si lo desea, la integración con Azure Private Link para los servicios de búsqueda que no deben estar visibles en la red pública de Internet.

### <a name="new-apis"></a>Nuevas API

| API | Category| Detalles |
|-----|--------|------------------|
| [NetworkRuleSet](/rest/api/searchmanagement/services/createorupdate#networkruleset) | Firewall de dirección IP | Restrinja el acceso a un punto de conexión de servicio a una lista de direcciones IP permitidas. Consulte [Configuración de un firewall de IP](service-configure-firewall.md) para obtener información sobre los conceptos y las instrucciones del portal. |
| [Recurso compartido de Private Link](/rest/api/searchmanagement/sharedprivatelinkresources) | Private Link | Cree un recurso compartido de Private Link para que lo use un servicio de búsqueda.  |
| [Conexiones de punto de conexión privado](/rest/api/searchmanagement/privateendpointconnections) | Private Link | Establezca y administre las conexiones a un servicio de búsqueda a través de un punto de conexión privado. Consulte [Creación de un punto de conexión privado](service-create-private-endpoint.md) para información sobre los conceptos y las instrucciones del portal.|
| [Recursos de Private Link](/rest/api/searchmanagement/privatelinkresources/) | Private Link | En el caso de un servicio de búsqueda que tenga una conexión de punto de conexión privado, obtenga una lista de todos los servicios usados en la misma red virtual. Si la solución de búsqueda incluye indexadores que extraen orígenes de datos de Azure (Azure Storage, Cosmos DB, Azure SQL) o usa Cognitive Services o Key Vault, todos esos recursos deben tener puntos de conexión en la red virtual y esta API debe devolver una lista. |
| [PublicNetworkAccess](/rest/api/searchmanagement/services/createorupdate#publicnetworkaccess)| Private Link | Se trata de una propiedad de las solicitudes de servicio de creación o actualización. Cuando está deshabilitada, el vínculo privado es la única modalidad de acceso. |

### <a name="breaking-changes"></a>Últimos cambios

Ya no puede usar GET en una solicitud [Enumerar claves de consulta](/rest/api/searchmanagement/querykeys/listbysearchservice). En las versiones anteriores podía usar GET o POST, en esta versión y en todas las versiones posteriores, solo se admite POST. 

## <a name="upgrade-to-20"></a>Actualización a 2.0

La versión 2 del SDK de administración de .NET de Azure Search es una actualización menor, por lo que el cambio del código solo debe requerir un esfuerzo mínimo. Los cambios en el SDK son estrictamente cambios del lado cliente para mejorar la facilidad de uso del propio SDK. Estos cambios incluyen los siguientes:

* `Services.CreateOrUpdate` y sus versiones asincrónicas ahora sondean automáticamente el `SearchService` de aprovisionamiento y no se devuelven hasta que el aprovisionamiento de servicio se completa. Esto evita que tenga que escribir el código de sondeo.

* Si de todos modos desea sondear manualmente el aprovisionamiento de servicio, puede usar el método `Services.BeginCreateOrUpdate` nuevo o una de sus versiones asincrónicas.

* Se agregaron los métodos `Services.Update` nuevos y sus versiones asincrónicas al SDK. Estos métodos usan HTTP PATCH para admitir la actualización incremental de un servicio. Por ejemplo, ahora puede escalar un servicio si pasa a esos métodos una instancia `SearchService` que solo contiene las propiedades `partitionCount` y `replicaCount` deseadas. La forma anterior de llamar a `Services.Get`, modificando el elemento `SearchService` devuelto y pasándolo a `Services.CreateOrUpdate`, se sigue admitiendo, pero ya no es necesaria. 

## <a name="next-steps"></a>Pasos siguientes

Si tiene problemas, el mejor foro para publicar preguntas es [Stack Overflow](https://stackoverflow.com/questions/tagged/azure-cognitive-search?tab=Newest). Si encuentra un error, puede enviarlo al [repositorio de GitHub del SDK de .NET para Azure](https://github.com/Azure/azure-sdk-for-net/issues). Asegúrese de que etiqueta el título de la incidencia con "[search]".