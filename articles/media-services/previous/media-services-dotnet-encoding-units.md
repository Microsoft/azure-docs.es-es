---
title: Escalado de procesamiento multimedia mediante la adición de unidades de codificación - Azure | Microsoft Docs
description: En este artículo se muestra cómo agregar unidades de codificación con Azure Media Services .NET.
services: media-services
documentationcenter: ''
author: juliako
manager: femila
editor: ''
ms.assetid: 33f7625a-966a-4f06-bc09-bccd6e2a42b5
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/18/2019
ms.author: juliako
ms.reviewer: milangada
ms.custom: devx-track-csharp
ms.openlocfilehash: f9293b735d6a752bca30e862a09943bf21583048
ms.sourcegitcommit: bcda98171d6e81795e723e525f81e6235f044e52
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89262419"
---
# <a name="how-to-scale-encoding-with-net-sdk"></a>Escalado de codificación con el SDK de .NET

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

> [!div class="op_single_selector"]
> * [Portal](media-services-portal-scale-media-processing.md)
> * [.NET](media-services-dotnet-encoding-units.md)
> * [REST](/rest/api/media/operations/encodingreservedunittype)
> * [Java](https://github.com/southworkscom/azure-sdk-for-media-services-java-samples)
> * [PHP](https://github.com/Azure/azure-sdk-for-php/tree/master/examples/MediaServices)
> 
> 

## <a name="overview"></a>Información general
> [!IMPORTANT]
> Para más información sobre cómo escalar el procesamiento de medios, consulte este tema de [Introducción](media-services-scale-media-processing-overview.md).
> 
> 

Para cambiar el tipo de unidad reservada y el número de unidades reservadas de codificación mediante el SDK de .NET, haga lo siguiente:

```csharp
IEncodingReservedUnit encodingS1ReservedUnit = _context.EncodingReservedUnits.FirstOrDefault();
encodingS1ReservedUnit.ReservedUnitType = ReservedUnitType.Basic; // Corresponds to S1
encodingS1ReservedUnit.Update();
Console.WriteLine("Reserved Unit Type: {0}", encodingS1ReservedUnit.ReservedUnitType);

encodingS1ReservedUnit.CurrentReservedUnits = 2;
encodingS1ReservedUnit.Update();

Console.WriteLine("Number of reserved units: {0}", encodingS1ReservedUnit.CurrentReservedUnits);
```

## <a name="opening-a-support-ticket"></a>Apertura de una incidencia de soporte técnico

De manera predeterminada, cada cuenta de Media Services puede escalar hasta 10 unidades reservadas de codificación (MRU) S2 o S3 o 25 MRU S1 y 5 unidades reservadas de streaming a petición. Si desea solicitar un límite mayor, abra una incidencia de soporte técnico.

## <a name="media-services-learning-paths"></a>Rutas de aprendizaje de Media Services
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Envío de comentarios
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]
