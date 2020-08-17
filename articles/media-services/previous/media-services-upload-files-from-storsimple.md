---
title: Carga de archivos en una cuenta de Azure Media Services mediante Azure StorSimple | Microsoft Docs
description: Este artículo ofrece una breve introducción a Azure StorSimple Data Manager. El artículo también incluye vínculos a tutoriales que muestran cómo extraer datos de StorSimple y cargarlos como recursos en una cuenta de Azure Media Services.
services: media-services
documentationcenter: ''
author: Juliako
manager: femila
editor: ''
ms.assetid: 1dd09328-262b-43ef-8099-73241b49a925
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 03/20/2019
ms.author: juliako
ms.openlocfilehash: 5e6a834e98d3b8447a68d149bb40e75d74959f39
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87038501"
---
# <a name="upload-files-into-an-azure-media-services-account-from-azure-storsimple"></a>Carga de archivos en una cuenta de Azure Media Services mediante Azure StorSimple  

> [!NOTE]
> No hay características o funcionalidades nuevas para agregar a Media Services, versión 2. <br/>Finalice la compra de la versión más reciente, [Media Services v3](../latest/index.yml). Consulte también la [guía de migración de la versión v2 a v3](../latest/migrate-from-v2-to-v3.md).
>
> 
> Azure StorSimple Data Manager actualmente está en versión preliminar privada. 
> 

## <a name="overview"></a>Información general

En Media Services, cargue los archivos digitales en un recurso. El recurso puede contener vídeo, audio, imágenes, colecciones de miniaturas, pistas de texto y archivos de subtítulos (y los metadatos de estos archivos). Una vez cargados los archivos, el contenido se almacena de forma segura en la nube para un posterior procesamiento y streaming.

[Azure StorSimple](../../storsimple/index.yml) usa el almacenamiento en la nube como una extensión de la solución local y organiza automáticamente los datos en niveles entre el almacenamiento local y el almacenamiento en la nube. El dispositivo de StorSimple desduplica y comprime los datos antes de enviarlos a la nube, lo que hace que sea muy eficaz para el envío de archivos de gran tamaño a la nube. El servicio [StorSimple Data Manager](../../storsimple/storsimple-data-manager-overview.md) proporciona API que permiten extraer datos de StorSimple y presentarlos como recursos de AMS.

## <a name="get-started"></a>Introducción

1. [Cree una cuenta de Media Services](media-services-portal-create-account.md) a la que desee transferir los recursos.
2. Regístrese para obtener la versión preliminar de Data Manager, tal y como se describe en el artículo [StorSimple Data Manager](../../storsimple/storsimple-data-manager-overview.md).
3. Cree una cuenta de StorSimple Data Manager.
4. Cree un trabajo de transformación de datos que, cuando se ejecute, extraiga datos de un dispositivo de StorSimple y los transfiera como recursos a una cuenta de AMS. 

    Cuando el trabajo comienza a ejecutarse, se crea una cola de almacenamiento. Esta cola se rellena con mensajes de los blobs transformados a medida que están listos. El nombre de esta cola es el mismo que el de la definición del trabajo. Puede usar esta cola para determinar si un recurso está listo y llamar a la operación de Media Services que desee para ejecutarla en él. Por ejemplo, puede usar esta cola para desencadenar una función de Azure que tenga el código de Media Services necesario.

## <a name="see-also"></a>Consulte también

[Uso del SDK de .NET para desencadenar trabajos en Data Manager](../../storsimple/storsimple-data-manager-dotnet-jobs.md)

## <a name="media-services-learning-paths"></a>Rutas de aprendizaje de Media Services
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Envío de comentarios
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

## <a name="next-steps"></a>Pasos siguientes

Ahora puede codificar los recursos cargados. Para más información, consulte [Encode an asset using Media Encoder Standard with the Azure portal](media-services-portal-encode.md)(Codificación de recursos mediante el estándar de codificador multimedia con Azure Portal).
