---
title: archivo de inclusión
description: archivo de inclusión
services: storage
author: tamram
ms.service: storage
ms.topic: include
ms.date: 11/05/2019
ms.author: tamram
ms.custom: include file
ms.openlocfilehash: 7c3438631dca921989309bb8701e113cb5ce3ff2
ms.sourcegitcommit: d0541eccc35549db6381fa762cd17bc8e72b3423
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/09/2020
ms.locfileid: "89570178"
---
Azure Blob Storage es la solución de almacenamiento de objetos de Microsoft para la nube. Blob Storage está optimizado para el almacenamiento de cantidades masivas de datos no estructurados. Los datos no estructurados son datos que no se ciñen a ningún un modelo de datos o definición concretos, como texto o datos binarios.

## <a name="about-blob-storage"></a>Acerca de Blob Storage

Blob Storage está diseñado para:

* Servicio de imágenes o documentos directamente a un explorador.
* Almacenamiento de archivos para acceso distribuido.
* Streaming de audio y vídeo.
* Escribir en archivos de registro.
* Almacenamiento de datos para copia de seguridad y restauración, recuperación ante desastres y archivado.
* Almacenamiento de datos para el análisis en local o en un servicio hospedado de Azure.

Los usuarios o las aplicaciones cliente pueden acceder a objetos en Blob Storage a través de HTTP/HTTPS, desde cualquier lugar del mundo. Se puede acceder a los objetos de Blob Storage mediante la [API REST de Azure Storage](https://docs.microsoft.com/rest/api/storageservices/blob-service-rest-api), [Azure PowerShell](https://docs.microsoft.com/powershell/module/az.storage), la [CLI de Azure](https://docs.microsoft.com/cli/azure/storage) o una biblioteca de cliente de Azure Storage. Hay bibliotecas de cliente disponibles para distintos lenguajes, entre los que se incluyen:

* [.NET](/dotnet/api/overview/azure/storage?view=azure-dotnet)
* [Java](https://docs.microsoft.com/java/api/overview/azure/storage)
* [Node.js](https://azure.github.io/azure-storage-node)
* [Python](https://docs.microsoft.com/azure/storage/blobs/storage-quickstart-blobs-python)
* [Go](https://github.com/azure/azure-storage-blob-go/)
* [PHP](https://azure.github.io/azure-storage-php/)
* [Ruby](https://azure.github.io/azure-storage-ruby)

## <a name="about-azure-data-lake-storage-gen2"></a>Acerca de Azure Data Lake Storage Gen2

Blob Storage es compatible con Azure Data Lake Storage Gen2, solución de análisis de macrodatos empresarial de Microsoft para la nube. Azure Data Lake Storage Gen2 no solo ofrece un sistema de archivos jerárquico, sino también las ventajas de Blob Storage, entre las que se incluyen:

* Almacenamiento en niveles de bajo costo
* Alta disponibilidad
* Coherencia máxima
* Funcionalidades de recuperación ante desastres

Para más información acerca de Data Lake Storage Gen2, consulte [Introduction to Azure Data Lake Storage Gen2](../articles/storage/data-lake-storage/introduction.md) (Introducción a Azure Data Lake Storage Gen2).
