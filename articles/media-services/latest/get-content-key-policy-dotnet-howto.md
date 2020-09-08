---
title: Obtención de una clave de firma de una directiva mediante .NET de Azure Media Services v3
description: En este tema se muestra cómo obtener una clave de firma de la directiva existente mediante el SDK de .NET de Media Services v3.
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
ms.custom: seodec18
ms.openlocfilehash: 7f46c77d463873a5cdd5d8c4ac1b28f1b7d0b9eb
ms.sourcegitcommit: 58d3b3314df4ba3cabd4d4a6016b22fa5264f05a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89299004"
---
# <a name="get-a-signing-key-from-the-existing-policy"></a>Obtención de una clave de firma de la directiva existente

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

Uno de los principales principios de diseño de la versión v3 de la API es hacerla más segura. Las API de la versión v3 no devuelven secretos o credenciales en las operaciones **Get** o **List**. Consulte la explicación detallada aquí: Para más información, consulte [RBAC and Media Services accounts](rbac-overview.md) (Cuentas de Media Services y RBAC)

En el ejemplo de este artículo se muestra cómo usar .NET para obtener una clave de firma a partir de la directiva existente. 
 
## <a name="download"></a>Descargar 

Clone un repositorio de GitHub que contenga el ejemplo de .NET completo en la máquina mediante el siguiente comando:  

 ```bash
 git clone https://github.com/Azure-Samples/media-services-v3-dotnet-tutorials.git
 ```
 
El ejemplo de ContentKeyPolicy con secretos se encuentra en la carpeta [EncryptWithDRM](https://github.com/Azure-Samples/media-services-v3-dotnet-tutorials/tree/master/AMSV3Tutorials/EncryptWithDRM).

## <a name="get-contentkeypolicy-with-secrets"></a>Obtención de ContentKeyPolicy con secretos 

Para obtener la clave, use **GetPolicyPropertiesWithSecretsAsync**, tal y como se muestra en el ejemplo siguiente.

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/EncryptWithDRM/Program.cs#GetOrCreateContentKeyPolicy)]

## <a name="next-steps"></a>Pasos siguientes

[Diseño del sistema de protección de contenido con DRM múltiple con control de acceso](design-multi-drm-system-with-access-control.md) 
