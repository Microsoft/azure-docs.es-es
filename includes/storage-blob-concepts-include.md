---
title: archivo de inclusión
description: archivo de inclusión
services: storage
author: tamram
ms.service: storage
ms.topic: include
ms.date: 11/18/2018
ms.author: tamram
ms.custom: include file
ms.openlocfilehash: 32b9b12c2adf03a4cb0616a5da48dd33fc81fb4f
ms.sourcegitcommit: 5d837a7557363424e0183d5f04dcb23a8ff966bb
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 12/06/2018
ms.locfileid: "52973104"
---
Azure Blob Storage es la solución de almacenamiento de objetos de Microsoft para la nube. Blob Storage está optimizado para el almacenamiento de cantidades masivas de datos no estructurados. Los datos no estructurados son datos que no cumplen un modelo de datos o definición concreta, como texto o datos binarios. 

## <a name="about-blob-storage"></a>Acerca de Blob Storage

Blob Storage está diseñado para:

* Servicio de imágenes o documentos directamente a un explorador.
* Almacenamiento de archivos para acceso distribuido.
* Streaming de audio y vídeo.
* Escribir en archivos de registro.
* Almacenamiento de datos para copia de seguridad y restauración, recuperación ante desastres y archivado.
* Almacenamiento de datos para el análisis en local o en un servicio hospedado de Azure.

Los usuarios o las aplicaciones cliente pueden acceder a objetos en Blob Storage a través de HTTP/HTTPS, desde cualquier lugar del mundo. Se puede acceder a los objetos de Blob Storage mediante la [API REST de Azure Storage](https://docs.microsoft.com/rest/api/storageservices/blob-service-rest-api), [Azure PowerShell](https://docs.microsoft.com/powershell/module/azure.storage), la [CLI de Azure](https://docs.microsoft.com/cli/azure/storage) o una biblioteca de cliente de Azure Storage. Las bibliotecas de cliente están disponibles para numerosos lenguajes, como [.NET](https://docs.microsoft.com/dotnet/api/overview/azure/storage/client), [Java](https://docs.microsoft.com/java/api/overview/azure/storage/client), [Node.js](http://azure.github.io/azure-storage-node), [Python](https://docs.microsoft.com/python/azure/), [Go](https://github.com/azure/azure-storage-blob-go/), [PHP](http://azure.github.io/azure-storage-php/) y [Ruby](http://azure.github.io/azure-storage-ruby).

## <a name="about-azure-data-lake-storage-gen2"></a>Acerca de Azure Data Lake Storage Gen2 

Blob Storage es compatible con Azure Data Lake Storage Gen2, solución de análisis de macrodatos empresariales de Microsoft para la nube. Azure Data Lake Storage Gen2 ofrece un sistema de archivos jerárquico, así como las ventajas de Blob Storage, que incluyen las siguientes funcionalidades: almacenamiento de bajo costo y en niveles, alta disponibilidad, coherencia fuerte y recuperación ante desastres. 

Para más información sobre Data Lake Storage Gen2, consulte [Introduction to Azure Data Lake Storage Gen2 Preview](../articles/storage/data-lake-storage/introduction.md) (Introducción a la versión de preliminar de Azure Data Lake Storage Gen2).
