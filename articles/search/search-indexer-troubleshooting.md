---
title: Solución de problemas comunes del indizador de búsqueda
titleSuffix: Azure Cognitive Search
description: Corrija errores y problemas comunes con los indizadores en Azure Cognitive Search, como la conexión del origen de datos, el firewall y documentos que falten.
manager: nitinme
author: mgottein
ms.author: magottei
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 11/04/2019
ms.openlocfilehash: 05d0cba2a3751ac010dc26f68137a31dd04c62ce
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88935150"
---
# <a name="troubleshooting-common-indexer-issues-in-azure-cognitive-search"></a>Solución de problemas comunes con el indizador en Azure Cognitive Search

Los indizadores pueden encontrar una serie de problemas al indexar datos en Azure Cognitive Search. Las categorías principales de error son las siguientes:

* [Conexión a un origen de datos u otros recursos](#connection-errors)
* [Procesamiento de documentos](#document-processing-errors)
* [Ingesta de documentos en un índice](#index-errors)

## <a name="connection-errors"></a>Errores de conexión

> [!NOTE]
> Los indexadores tienen compatibilidad limitada para el acceso a orígenes de datos y otros recursos protegidos por los mecanismos de seguridad de red de Azure. Actualmente, los indexadores solo pueden tener acceso a los orígenes de datos a través de los mecanismos de restricción de intervalo de direcciones IP o las reglas de NSG cuando corresponda. A continuación puede encontrar información detallada sobre el acceso a cada origen de datos compatible.
>
> Puede averiguar la dirección IP del servicio de búsqueda haciendo ping al nombre de dominio completo (p. ej., `<your-search-service-name>.search.windows.net`).
>
> Puede averiguar el intervalo de direcciones IP de la [etiqueta de servicio](../virtual-network/service-tags-overview.md#available-service-tags) `AzureCognitiveSearch` mediante el uso de [archivos JSON descargables](../virtual-network/service-tags-overview.md#discover-service-tags-by-using-downloadable-json-files) o a través de la [API de detección de etiquetas de servicio](../virtual-network/service-tags-overview.md#use-the-service-tag-discovery-api-public-preview). El intervalo de direcciones IP se actualiza semanalmente.

### <a name="configure-firewall-rules"></a>Configurar reglas de firewall

Azure Storage, CosmosDB y Azure SQL proporcionan un firewall configurable. No hay ningún mensaje de error específico cuando el firewall está habilitado. Normalmente, los errores de firewall son genéricos y se parecen a `The remote server returned an error: (403) Forbidden` o `Credentials provided in the connection string are invalid or have expired`.

Hay dos opciones para permitir que los indexadores tengan acceso a estos recursos en una instancia de este tipo:

* Para deshabilitar el firewall, permita el acceso desde **"Todas las redes"** (si es factible).
* Como alternativa, puede permitir el acceso a la dirección IP del servicio de búsqueda y al intervalo de direcciones IP de la [etiqueta de servicio](../virtual-network/service-tags-overview.md#available-service-tags) `AzureCognitiveSearch` en las reglas de firewall del recurso (restricción del intervalo de direcciones IP).

Los detalles para configurar las restricciones del intervalo de direcciones IP para cada tipo de origen de datos se pueden encontrar en los vínculos siguientes:

* [Almacenamiento de Azure](../storage/common/storage-network-security.md#grant-access-from-an-internet-ip-range)

* [Cosmos DB](../storage/common/storage-network-security.md#grant-access-from-an-internet-ip-range)

* [SQL de Azure](../azure-sql/database/firewall-configure.md#create-and-manage-ip-firewall-rules)

**Limitación**: Como se indica en la documentación anterior para Azure Storage, las restricciones del intervalo de direcciones IP solo funcionarán si el servicio de búsqueda y la cuenta de almacenamiento se encuentran en regiones diferentes.

Azure Functions (que podría usarse como [aptitud de API web personalizada](cognitive-search-custom-skill-web-api.md)) también admite [restricciones de direcciones IP](../azure-functions/ip-addresses.md#ip-address-restrictions). La lista de direcciones IP a configurar sería la dirección IP del servicio de búsqueda y el intervalo de direcciones IP de la etiqueta de servicio `AzureCognitiveSearch`.

Los detalles para acceder a los datos de SQL Server en una máquina virtual de Azure se describen [aquí](search-howto-connecting-azure-sql-iaas-to-azure-search-using-indexers.md).

### <a name="configure-network-security-group-nsg-rules"></a>Configuración de las reglas del grupo de seguridad de red (NSG)

Al acceder a los datos en una instancia administrada de SQL, o cuando se usa una máquina virtual de Azure como URI del servicio web para una [aptitud de API web personalizada](cognitive-search-custom-skill-web-api.md), los clientes no deben preocuparse por direcciones IP específicas.

En tales casos, la máquina virtual de Azure o la instancia administrada de SQL se pueden configurar para que residan en una red virtual. Entonces, se puede configurar un grupo de seguridad de red para filtrar el tipo de tráfico de red que puede entrar y salir de las subredes e interfaces de red de la red virtual.

La etiqueta de servicio `AzureCognitiveSearch` se puede usar directamente en las [reglas de NSG](../virtual-network/manage-network-security-group.md#work-with-security-rules) de entrada sin necesidad de buscar su intervalo de direcciones IP.

Se describen más detalles sobre el acceso a los datos en una instancia administrada de SQL [aquí](search-howto-connecting-azure-sql-mi-to-azure-search-using-indexers.md).

### <a name="cosmosdb-indexing-isnt-enabled"></a>La "indexación" de CosmosDB no está habilitada.

Azure Cognitive Search tiene una dependencia implícita del indexado de Cosmos DB. Si desactiva el indexado automático en Cosmos DB, Azure Cognitive Search devuelve un estado correcto, pero no puede indexar el contenido del contenedor. Para instrucciones sobre cómo establecer y activar el indexado, consulte [Administración automática en Azure Cosmos DB](../cosmos-db/how-to-manage-indexing-policy.md#use-the-azure-portal).

## <a name="document-processing-errors"></a>Errores en el procesamiento de documentos

### <a name="unprocessable-or-unsupported-documents"></a>Documentos no procesables o no compatibles

El indizador de blobs [documenta qué formatos de documento se admiten de manera explícita](search-howto-indexing-azure-blob-storage.md#SupportedFormats). A veces, un contenedor de almacenamiento de blobs contiene documentos no compatibles. En otras ocasiones, puede haber documentos problemáticos. Para evitar que el indizador se detenga en estos documentos, [cambie las opciones de configuración](search-howto-indexing-azure-blob-storage.md#DealingWithErrors):

```
PUT https://[service name].search.windows.net/indexers/[indexer name]?api-version=2020-06-30
Content-Type: application/json
api-key: [admin key]

{
  ... other parts of indexer definition
  "parameters" : { "configuration" : { "failOnUnsupportedContentType" : false, "failOnUnprocessableDocument" : false } }
}
```

### <a name="missing-document-content"></a>Contenido de documento faltante

El indizador de blobs [busca y extrae texto de los blobs de un contenedor](search-howto-indexing-azure-blob-storage.md#how-azure-search-indexes-blobs). Algunos problemas con la extracción de texto son los siguientes:

* El documento solo contiene imágenes escaneadas. Los blobs de PDF que tienen contenido no textual, como imágenes escaneadas (JPG), no generan resultados en una canalización de indexación de blobs estándar. Si tiene contenido de imagen con elementos de texto, puede usar [Cognitive Search](cognitive-search-concept-image-scenarios.md) para buscar y extraer el texto.
* El indizador de blobs está configurado para indexar solo metadatos. Para extraer contenido, el indizador de blobs se debe configurar para [extraer tanto contenido como metadatos](search-howto-indexing-azure-blob-storage.md#controlling-which-parts-of-the-blob-are-indexed):

```
PUT https://[service name].search.windows.net/indexers/[indexer name]?api-version=2020-06-30
Content-Type: application/json
api-key: [admin key]

{
  ... other parts of indexer definition
  "parameters" : { "configuration" : { "dataToExtract" : "contentAndMetadata" } }
}
```

## <a name="index-errors"></a>Errores de índice

### <a name="missing-documents"></a>Documentos faltantes

Los indizadores buscan documentos de un [origen de datos](/rest/api/searchservice/create-data-source). En algunas ocasiones, un documento del origen de datos que se debería haber indexado aparece como faltante en un índice. Hay un par de razones comunes por las que pueden producirse estos errores:

* El documento no se ha indexado. Revise el portal para ver si el indizador se ejecutó correctamente.
* Compruebe el valor del [seguimiento de cambios](/rest/api/searchservice/create-data-source#data-change-detection-policies). Si el valor de límite máximo es una fecha establecida en un momento futuro, el indexador omitirá cualquier documento que tenga una fecha inferior a esta. Puede conocer el estado del seguimiento de cambios del indexador mediante los campos "initialTrackingState" y "finalTrackingState" del [estado del indexador](/rest/api/searchservice/get-indexer-status#indexer-execution-result).
* El documento se actualizó después de la ejecución del indizador. Si el indizador sigue una [programación](/rest/api/searchservice/create-indexer#indexer-schedule), a la larga se volverá a ejecutar y recogerá el documento.
* La [consulta](/rest/api/searchservice/create-data-source) especificada en el origen de datos excluye el documento. Los indizadores no pueden indexar documentos que no forman parte del origen de datos.
* [Asignaciones de campo](/rest/api/searchservice/create-indexer#fieldmappings) o [enriquecimiento de inteligencia artificial](./cognitive-search-concept-intro.md) han cambiado el documento y tiene un aspecto distinto de lo esperado.
* Use la [API Buscar documento](/rest/api/searchservice/lookup-document) para buscar el documento.