---
title: Migración de Azure Media Encoder a Media Encoder Standard | Microsoft Docs
description: En este tema se describe cómo migrar del procesador de multimedia de Azure Media Encoder al de Media Encoder Standard.
services: media-services
documentationcenter: ''
author: juliako
manager: femila
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 08/21/2019
ms.author: juliako
ms.custom: devx-track-csharp
ms.openlocfilehash: 32eab0f729280ad25f45853e05ea982b60a6d9f7
ms.sourcegitcommit: bcda98171d6e81795e723e525f81e6235f044e52
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89259343"
---
# <a name="migrate-from-azure-media-encoder-to-media-encoder-standard"></a>Migración de Azure Media Encoder a Media Encoder Standard

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

En este artículo se describen los pasos para migrar del procesador de multimedia heredado Azure Media Encoder (AME), que se va a retirar, al procesador de multimedia Media Encoder Standard. Para ver las fechas de retirada, consulte el tema [componentes heredados](legacy-components.md).

Al codificar archivos con AME, los clientes suelen usar una cadena preestablecida con nombre como `H264 Adaptive Bitrate MP4 Set 1080p`. Para realizar la migración, se debe actualizar el código para que utilice el procesador de multimedia **Media Encoder Standard** en lugar de AME y uno de los [valores preestablecidos del sistema](media-services-mes-presets-overview.md) equivalentes como `H264 Multiple Bitrate 1080p`. 

## <a name="migrating-to-media-encoder-standard"></a>Migración a Media Encoder Standard

A continuación se muestra un ejemplo de código de C# típico que usa el procesador de multimedia heredado. 

```csharp
// Declare a new job. 
IJob job = _context.Jobs.Create("AME Job"); 
// Get a media processor reference, and pass to it the name of the  
// processor to use for the specific task. 
IMediaProcessor processor = GetLatestMediaProcessorByName("Azure Media Encoder"); 

// Create a task with the encoding details, using a string preset. 
// In this case " H264 Adaptive Bitrate MP4 Set 1080p" preset is used. 
ITask task = job.Tasks.AddNew("My encoding task", 
    processor, 
    " H264 Adaptive Bitrate MP4 Set 1080p", 
    TaskOptions.None); 
```

Esta es la versión actualizada que usa Media Encoder Standard.

```csharp
// Declare a new job. 
IJob job = _context.Jobs.Create("Media Encoder Standard Job"); 
// Get a media processor reference, and pass to it the name of the  
// processor to use for the specific task. 
IMediaProcessor processor = GetLatestMediaProcessorByName("Media Encoder Standard"); 

// Create a task with the encoding details, using a string preset. 
// In this case " H264 Multiple Bitrate 1080p" preset is used. 
ITask task = job.Tasks.AddNew("My encoding task", 
    processor, 
    "H264 Multiple Bitrate 1080p", 
    TaskOptions.None); 
```

### <a name="advanced-scenarios"></a>Escenarios avanzados 

Si ha creado su propio valor preestablecido de codificación para AME mediante su esquema, hay un [esquema equivalente para Media Encoder Standard](media-services-mes-schema.md). Si tiene alguna pregunta sobre cómo asignar la configuración anterior al nuevo codificador, póngase en contacto con nosotros en mailto:amshelp@microsoft.com.  
## <a name="known-differences"></a>Diferencias conocidas 

Media Encoder Standard es más sólido, confiable, tiene un mejor rendimiento y genera una salida de mejor calidad que el codificador AME heredado. Además: 

* Media Encoder Standard genera archivos de salida con una convención de nomenclatura diferente de AME.
* Media Encoder Standard genera artefactos, como archivos, que contienen los [metadatos de archivos de entrada](media-services-input-metadata-schema.md) y los [metadatos de archivos de salida](media-services-output-metadata-schema.md).

## <a name="next-steps"></a>Pasos siguientes

* [Componentes heredados](legacy-components.md)
* [Página de precios](https://azure.microsoft.com/pricing/details/media-services/#encoding)
