---
title: 'Solución de problemas comunes del indexador de búsqueda: Azure Search'
description: Corrija errores y problemas comunes con los indexadores en Azure Search, como la conexión del origen de datos, el firewall y documentos que falten.
author: mgottein
manager: cgronlun
services: search
ms.service: search
ms.devlang: na
ms.topic: conceptual
ms.date: 05/02/2019
ms.author: magottei
ms.custom: seodec2018
ms.openlocfilehash: 256a38320c9b3ca826ee9c12ac0a437957f988e2
ms.sourcegitcommit: f6c85922b9e70bb83879e52c2aec6307c99a0cac
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2019
ms.locfileid: "65539267"
---
# <a name="troubleshooting-common-indexer-issues-in-azure-search"></a>Solución de problemas comunes con el indizador en Azure Search

Los indizadores pueden encontrar una serie de problemas al indexar datos en Azure Search. Las categorías principales de error son las siguientes:

* [Conexión a un origen de datos](#data-source-connection-errors)
* [Procesamiento de documentos](#document-processing-errors)
* [Ingesta de documentos en un índice](#index-errors)

## <a name="data-source-connection-errors"></a>Errores de conexión del origen de datos

### <a name="blob-storage"></a>Blob Storage

#### <a name="storage-account-firewall"></a>Firewall de la cuenta de almacenamiento

Azure Storage ofrece un firewall configurable. De manera predeterminada, el firewall está deshabilitado para que Azure Search se pueda conectar a la cuenta de almacenamiento.

No hay ningún mensaje de error específico cuando el firewall está habilitado. Por lo general, los errores de firewall se verán así: `The remote server returned an error: (403) Forbidden`.

Puede comprobar que el firewall está habilitado en el [portal](https://docs.microsoft.com/azure/storage/common/storage-network-security#azure-portal). Si el firewall está habilitado, tiene dos opciones para evitar este problema:

1. Para deshabilitar el firewall, elija permitir el acceso desde ["Todas las redes"](https://docs.microsoft.com/azure/storage/common/storage-network-security#azure-portal).
1. [Agregue una excepción](https://docs.microsoft.com/azure/storage/common/storage-network-security#managing-ip-network-rules) para la dirección IP del servicio de búsqueda. Para encontrar esta dirección IP, use el comando siguiente:

`nslookup <service name>.search.windows.net`

Las excepciones no funcionan para [búsqueda cognitiva](cognitive-search-concept-intro.md). La única solución alternativa es deshabilitar el firewall.

### <a name="cosmos-db"></a>Cosmos DB

#### <a name="indexing-isnt-enabled"></a>El indexado no está habilitado

Azure Search tiene una dependencia implícita del indexado de Cosmos DB. Si desactiva el indexado automático en Cosmos DB, Azure Search devuelve un estado correcto, pero no puede indexar el contenido del contenedor. Para instrucciones sobre cómo establecer y activar el indexado, consulte [Administración automática en Azure Cosmos DB](https://docs.microsoft.com/azure/cosmos-db/how-to-manage-indexing-policy#manage-indexing-using-azure-portal).

## <a name="document-processing-errors"></a>Errores en el procesamiento de documentos

### <a name="unprocessable-or-unsupported-documents"></a>Documentos no procesables o no compatibles

El indizador de blobs [documenta qué formatos de documento se admiten de manera explícita](search-howto-indexing-azure-blob-storage.md#supported-document-formats). A veces, un contenedor de almacenamiento de blobs contiene documentos no compatibles. En otras ocasiones, puede haber documentos problemáticos. Para evitar que el indizador se detenga en estos documentos, [cambie las opciones de configuración](search-howto-indexing-azure-blob-storage.md#dealing-with-errors):

```
PUT https://[service name].search.windows.net/indexers/[indexer name]?api-version=2019-05-06
Content-Type: application/json
api-key: [admin key]

{
  ... other parts of indexer definition
  "parameters" : { "configuration" : { "failOnUnsupportedContentType" : false, "failOnUnprocessableDocument" : false } }
}
```

### <a name="missing-document-content"></a>Contenido de documento faltante

El indizador de blobs [busca y extrae texto de los blobs de un contenedor](search-howto-indexing-azure-blob-storage.md#how-azure-search-indexes-blobs). Algunos problemas con la extracción de texto son los siguientes:

* El documento solo contiene imágenes escaneadas. Los blobs de PDF que tienen contenido no textual, como imágenes escaneadas (JPG), no generan resultados en una canalización de indexación de blobs estándar. Si tiene contenido de la imagen con los elementos de texto, puede usar [búsqueda cognitiva](cognitive-search-concept-image-scenarios.md) para buscar y extraer el texto.
* El indizador de blobs está configurado para indexar solo metadatos. Para extraer contenido, el indizador de blobs se debe configurar para [extraer tanto contenido como metadatos](search-howto-indexing-azure-blob-storage.md#controlling-which-parts-of-the-blob-are-indexed):

```
PUT https://[service name].search.windows.net/indexers/[indexer name]?api-version=2019-05-06
Content-Type: application/json
api-key: [admin key]

{
  ... other parts of indexer definition
  "parameters" : { "configuration" : { "dataToExtract" : "contentAndMetadata" } }
}
```

## <a name="index-errors"></a>Errores de índice

### <a name="missing-documents"></a>Documentos faltantes

Los indizadores buscan documentos de un [origen de datos](https://docs.microsoft.com/rest/api/searchservice/create-data-source). En algunas ocasiones, un documento del origen de datos que se debería haber indexado aparece como faltante en un índice. Hay un par de razones comunes por las que pueden producirse estos errores:

* El documento no se ha indexado. Revise el portal para ver si el indizador se ejecutó correctamente.
* El documento se actualizó después de la ejecución del indizador. Si el indizador sigue una [programación](https://docs.microsoft.com/rest/api/searchservice/create-indexer#indexer-schedule), a la larga se volverá a ejecutar y recogerá el documento.
* La [consulta](https://docs.microsoft.com/rest/api/searchservice/create-data-source#request-body-syntax) especificada en el origen de datos excluye el documento. Los indizadores no pueden indexar documentos que no forman parte del origen de datos.
* [Asignaciones de campos](https://docs.microsoft.com/rest/api/searchservice/create-indexer#fieldmappings) o [búsqueda cognitiva](https://docs.microsoft.com/azure/search/cognitive-search-concept-intro) ha cambiado el documento y tiene un aspecto diferente de lo esperado.
* Use la [API Buscar documento](https://docs.microsoft.com/rest/api/searchservice/lookup-document) para buscar el documento.
