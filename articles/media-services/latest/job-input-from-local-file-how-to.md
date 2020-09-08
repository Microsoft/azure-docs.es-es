---
title: Creación de una entrada de trabajo de Azure Media Services desde un archivo local | Microsoft Docs
description: En este artículo se muestra cómo crear una entrada de trabajo de Azure Media Services desde un archivo local.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.service: media-services
ms.workload: ''
ms.topic: how-to
ms.date: 08/31/2020
ms.author: inhenkel
ms.openlocfilehash: 02aaae89f7544392765b46361b1617e0fd7dae8c
ms.sourcegitcommit: 58d3b3314df4ba3cabd4d4a6016b22fa5264f05a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89296814"
---
# <a name="create-a-job-input-from-a-local-file"></a>Creación de una entrada de trabajo a partir de un archivo local

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

En Media Services v3, cuando se envían trabajos para procesar los vídeos, es necesario indicar a Media Services dónde encontrar el vídeo de entrada. El vídeo de entrada se puede almacenar como un recurso de Media Services, en cuyo caso se crea un recurso de entrada basado en un archivo (almacenado localmente o en Azure Blob Storage). En este tema se muestra cómo crear una entrada de trabajo a partir de un archivo local. Para un ejemplo completo, consulte este [ejemplo de GitHub](https://github.com/Azure-Samples/media-services-v3-dotnet-tutorials/blob/master/AMSV3Tutorials/UploadEncodeAndStreamFiles/Program.cs).

## <a name="prerequisites"></a>Prerrequisitos 

* [Cree una cuenta de Media Services](./create-account-howto.md).

## <a name="net-sample"></a>Ejemplo de .NET

En el código siguiente se muestra cómo crear un recurso de entrada y usarlo como la entrada del trabajo. La función CreateInputAsset realiza las acciones siguientes:

* Crea el recurso.
* Obtiene una [dirección URL de SAS](../../storage/common/storage-sas-overview.md) que se puede escribir en el [contenedor de almacenamiento](../../storage/blobs/storage-quickstart-blobs-dotnet.md#upload-blobs-to-a-container) del recurso.
* Carga el archivo en el contenedor de almacenamiento mediante la dirección URL de SAS

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/UploadEncodeAndStreamFiles/Program.cs#CreateInputAsset)]

En el fragmento de código siguiente se crea un recurso de salida si aún no existe:

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/UploadEncodeAndStreamFiles/Program.cs#CreateOutputAsset)]

El siguiente fragmento de código envía un trabajo de codificación:

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/UploadEncodeAndStreamFiles/Program.cs#SubmitJob)]

## <a name="job-error-codes"></a>Códigos de error de trabajo

Consulte [Códigos de error](/rest/api/media/jobs/get#joberrorcode).

## <a name="next-steps"></a>Pasos siguientes

[Creación de una entrada de trabajo a partir de una dirección URL de HTTP(s)](job-input-from-http-how-to.md).
