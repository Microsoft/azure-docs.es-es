---
title: Recursos
titleSuffix: Azure Media Services
description: Conozca qué son los recursos y cómo se usan en Azure Media Services.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.service: media-services
ms.workload: ''
ms.topic: conceptual
ms.date: 08/31/2020
ms.author: inhenkel
ms.custom: seodec18
ms.openlocfilehash: 99101f31ec955c822c784180dd8b04bfb8e5f73c
ms.sourcegitcommit: 58d3b3314df4ba3cabd4d4a6016b22fa5264f05a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89291609"
---
# <a name="assets-in-azure-media-services-v3"></a>Recursos de la versión 3 de Azure Media Services

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

En Azure Media Services, un [recurso](/rest/api/media/assets) es un concepto básico. Es donde se introducen elementos multimedia (por ejemplo, a través de la carga o la ingesta en vivo), se generan elementos multimedia (a partir de la salida de un trabajo) y desde el que se publican elementos multimedia (para streaming). 

Un recurso se asigna a un contenedor de blobs en la [cuenta de Azure Storage](storage-account-concept.md) y los archivos del recurso se almacenan como blobs en bloques en ese contenedor. Los recursos contienen información sobre archivos digitales almacenados en Azure Storage (como vídeos, audio, imágenes, colecciones de miniaturas, pistas de texto y subtítulos).

Azure Media Services admite los niveles de blob cuando la cuenta usa el almacenamiento de uso general v2 (GPv2). Con GPv2, puede mover los archivos al [almacenamiento de acceso esporádico o al almacenamiento en frío](../../storage/blobs/storage-blob-storage-tiers.md). El almacenamiento de **archivos** es adecuado para guardar archivos de origen cuando ya no son necesarios (por ejemplo, una vez codificados).

El nivel de almacenamiento de **archivos** solo se recomienda para archivos de origen muy grandes que ya se hayan codificado y cuya salida del trabajo de codificación se haya colocado en un contenedor de blobs de salida. Los blobs del contenedor de salida que quiera asociar con un recurso y usar para transmitir o analizar contenido deben existir en un nivel de almacenamiento **frecuente** o **esporádico**.

## <a name="naming"></a>Nomenclatura 

### <a name="assets"></a>Recursos

Los nombres de los recursos tienen que ser únicos. Los nombres de recursos de Media Services v3 (por ejemplo, recursos, trabajos y transformaciones) están sujetos a las restricciones de nomenclatura de Azure Resource Manager. Para más información, consulte [Convenciones de nomenclatura](media-services-apis-overview.md#naming-conventions).

### <a name="blobs"></a>Datos BLOB

Los nombres de los archivos o blobs dentro de un recurso deben seguir los [requisitos para los nombres de blobs](/rest/api/storageservices/naming-and-referencing-containers--blobs--and-metadata) y los [requisitos para los nombres NTFS](/windows/win32/fileio/naming-a-file). La razón de estos requisitos es que los archivos se puedan copiar desde Blob Storage a un disco NTFS local para su procesamiento.

## <a name="next-steps"></a>Pasos siguientes

[Introducción a Media Services](media-services-overview.md)

## <a name="see-also"></a>Consulte también

[Diferencias entre la versión v2 y v3 de Media Services](migrate-from-v2-to-v3.md)
