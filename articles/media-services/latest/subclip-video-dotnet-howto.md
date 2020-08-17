---
title: Creación de un subclip de vídeo al codificar con Azure Media Services
description: En este tema se describe cómo crear un subclip de vídeo al codificar mediante Azure Media Services con el SDK de .NET.
services: media-services
documentationcenter: ''
author: Juliako
manager: femila
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: ne
ms.topic: article
ms.date: 06/09/2019
ms.author: juliako
ms.openlocfilehash: 31fa7fe6b16b7ba97b8ce5e6697f1aaaf88aa294
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87091902"
---
# <a name="subclip-a-video-when-encoding-with-media-services---net"></a>Creación de un subclip de vídeo al codificar mediante Azure Media Services: .NET

Puede recortar un vídeo, o crear un subclip de este al codificarlo mediante la opción [Trabajo](/rest/api/media/jobs). Esta funcionalidad se puede usar con cualquier elemento [Transformación](/rest/api/media/transforms) compilado mediante los valores preestablecidos [BuiltInStandardEncoderPreset](/rest/api/media/transforms/createorupdate#builtinstandardencoderpreset) o [StandardEncoderPreset](/rest/api/media/transforms/createorupdate#standardencoderpreset).

En el siguiente código C# de ejemplo se crea un trabajo que recorta un vídeo en un recurso cuando envía un trabajo de codificación. 

## <a name="prerequisites"></a>Prerrequisitos

Para completar los pasos descritos en este tema, ha de:

- [Creación de una cuenta de Azure Media Services](./create-account-howto.md)
- Crear una transformación y recursos de entrada y salida. Puede ver cómo crear una transformación y recursos de entrada y salidos en el tutorial [Carga, codificación y transmisión en secuencias de videos mediante .NET](stream-files-tutorial-with-api.md).
- Revisar el tema [Concepto de codificación](encoding-concept.md).

## <a name="example"></a>Ejemplo

```csharp
/// <summary>
/// Submits a request to Media Services to apply the specified Transform to a given input video.
/// </summary>
/// <param name="client">The Media Services client.</param>
/// <param name="resourceGroupName">The name of the resource group within the Azure subscription.</param>
/// <param name="accountName"> The Media Services account name.</param>
/// <param name="transformName">The name of the transform.</param>
/// <param name="jobName">The (unique) name of the job.</param>
/// <param name="inputAssetName">The name of the input asset.</param>
/// <param name="outputAssetName">The (unique) name of the  output asset that will store the result of the encoding job. </param>
// <SubmitJob>
private static async Task<Job> JobWithBuiltInStandardEncoderWithSingleClipAsync(
    IAzureMediaServicesClient client,
    string resourceGroupName,
    string accountName,
    string transformName,
    string jobName,
    string inputAssetName,
    string outputAssetName)
{
    var jobOutputs = new List<JobOutputAsset>
    {
        new JobOutputAsset(state: JobState.Queued, progress: 0, assetName: outputAssetName)
    };

    var clipStart = new AbsoluteClipTime()
    {
        Time = new TimeSpan(0, 0, 20)
    };

    var clipEnd = new AbsoluteClipTime()
    {
        Time = new TimeSpan(0, 0, 30)
    };

    var jobInput = new JobInputAsset(assetName: inputAssetName, start: clipStart, end: clipEnd);

    Job job = await client.Jobs.CreateAsync(
        resourceGroupName,
        accountName,
        transformName,
        jobName,
        new Job(input: jobInput, outputs: jobOutputs.ToArray(), name: jobName)
        {
            Description = $"A Job with transform {transformName} and single clip.",
            Priority = Priority.Normal,
        });

    return job;
}
```

## <a name="next-steps"></a>Pasos siguientes

[Procedimiento de codificación con una transformación personalizada](customize-encoder-presets-how-to.md) 
