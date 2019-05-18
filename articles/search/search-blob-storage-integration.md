---
title: 'Incorporación de búsqueda de texto completo a Azure Blob Storage: Azure Search'
description: Rastreo de contenido de texto en Azure Blob Storage para la indexación de Azure Search en código mediante la API REST de HTTP.
services: search
ms.service: search
ms.topic: conceptual
ms.date: 03/01/2019
author: mgottein
manager: cgronlun
ms.author: magottei
ms.custom: seodec2018
ms.openlocfilehash: b6bb70e4c56adb162006d2597d301c73b12d2a8a
ms.sourcegitcommit: f6c85922b9e70bb83879e52c2aec6307c99a0cac
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2019
ms.locfileid: "65540887"
---
# <a name="searching-blob-storage-with-azure-search"></a>Búsqueda en Blob Storage con Azure Search

La búsqueda en los diversos tipos de contenido almacenado en Azure Blob Storage puede ser un problema difícil de resolver. Pero gracias a Azure Search, es posible indexar y buscar en el contenido de los blobs con tan solo unos clics. Para buscar en Blob Storage, es necesario aprovisionar un servicio de Azure Search. Encontrará los diversos límites de los servicios y los planes de tarifa de Azure Search en la [página de precios](https://aka.ms/azspricing).

## <a name="what-is-azure-search"></a>¿Qué es Azure Search?
[Azure Search](https://aka.ms/whatisazsearch) es un servicio de búsqueda que permite que los programadores agreguen fácilmente experiencias eficaces de búsqueda de texto completo para aplicaciones web y móviles. Como servicio, Azure Search elimina la necesidad de administrar las infraestructuras de búsqueda al mismo tiempo que ofrece un [tiempo de actividad del 99,9 % a través del contrato de nivel de servicio](https://aka.ms/azuresearchsla).

## <a name="index-and-search-enterprise-document-formats"></a>Indexación y búsqueda de formatos de documentos empresariales
Con compatibilidad para la [extracción de documentos](https://aka.ms/azsblobindexer) en Azure Blob Storage, puede indexar el contenido siguiente:

[!INCLUDE [search-blob-data-sources](../../includes/search-blob-data-sources.md)]

Mediante la extracción de texto y metadatos de estos tipos de archivos, puede realizar búsquedas en varios formatos de archivo con una sola búsqueda. 

## <a name="search-through-your-blob-metadata"></a>Buscar en los metadatos del blob
Una práctica común que permite ordenar fácilmente blobs con cualquier tipo de contenido consiste en indexar los metadatos personalizados, así como las propiedades del sistema para cada uno de los blobs. De esta manera, la información de todos los blobs se indexa independientemente del tipo de documento. A continuación, puede continuar con la ordenación, filtrado y búsqueda de contenido mediante facetas en todo el contenido de Blob Storage.

[Obtenga más información sobre la indexación de metadatos del blob.](https://aka.ms/azsblobmetadataindexing)

## <a name="image-search"></a>Búsqueda de imágenes
Ahora, la búsqueda de texto completo, la navegación por facetas y las funcionalidades de ordenación de Azure Search se pueden aplicar a los metadatos de las imágenes almacenadas en blobs.

Búsqueda cognitiva incluye capacidades de procesamiento de imagen como [reconocimiento óptico de caracteres (OCR)](cognitive-search-skill-ocr.md) e identificación de [características visuales](cognitive-search-skill-image-analysis.md) que hacen posible indizar el contenido visual encontrado en cada uno imagen.

## <a name="index-and-search-through-json-blobs"></a>Indexar y buscar en blobs JSON
Es posible configurar Azure Search para que extraiga el contenido estructurado que se encuentra en los blobs que contienen JSON. Azure Search puede leer blobs JSON y analizar el contenido estructurado en los campos correspondientes de un documento de Azure Search. Azure Search también puede tomar los blobs que contienen una matriz de objetos JSON y asignar cada elemento a un documento independiente de Azure Search.

Actualmente, el análisis de JSON no se puede configurar en el portal. [Obtenga más información sobre el análisis de JSON en Azure Search.](https://aka.ms/azsjsonblobindexing)

## <a name="quick-start"></a>Inicio rápido
Es posible agregar Azure Search a blobs directamente desde la página del portal de Blob Storage.

![](./media/search-blob-storage-integration/blob-blade.png)

Haga clic en **Agregar Azure Search** para iniciar un flujo en el que puede seleccionar un servicio existente de Azure Search o crear uno nuevo. Si crea un servicio, saldrá de la experiencia del portal de su cuenta de almacenamiento. Puede regresar a la página del portal de almacenamiento y volver a seleccionar la opción **Agregar Azure Search**, donde puede seleccionar el servicio existente.

## <a name="next-steps"></a>Pasos siguientes
Obtenga más información sobre el indexador de blobs de Azure Search en la [documentación](https://aka.ms/azsblobindexer) completa.
