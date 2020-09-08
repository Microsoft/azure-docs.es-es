---
title: 'Inicio rápido: Configuración de la plataforma para usar C# en Unity con el SDK de Voz: servicio de voz'
titleSuffix: Azure Cognitive Services
description: Use esta guía para configurar la plataforma para usar C# en Unity con el SDK del servicio de voz.
services: cognitive-services
author: markamos
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: include
ms.date: 10/10/2019
ms.author: erhopf
ms.custom: devx-track-csharp
ms.openlocfilehash: 7678ffa1684fb1fe1ea198e9e1836a152cd51b4b
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88926710"
---
En esta guía se muestra cómo instalar el [SDK de Voz](~/articles/cognitive-services/speech-service/speech-sdk.md) para [Unity](https://unity3d.com/).

> [!NOTE]
> El SDK de Voz para Unity es compatible con Windows Desktop (x86 y x64) o con la Plataforma universal de Windows (x86, x64, ARM o ARM64), Android (x86, ARM32/64) e iOS (simulador de x64, ARM32 y ARM64)

[!INCLUDE [License Notice](~/includes/cognitive-services-speech-service-license-notice.md)]

## <a name="prerequisites"></a>Prerrequisitos

Esta guía de inicio rápido requiere:

- [Unity 2018.3 o posterior](https://store.unity.com/) con [Unity 2019.1 que agrega compatibilidad con UWP ARM64](https://blogs.unity3d.com/2019/04/16/introducing-unity-2019-1/#universal).
- [Visual Studio 2019](https://visualstudio.microsoft.com/downloads/). La versión 15.9 o posterior de Visual Studio 2017 también es aceptable.
- Para obtener compatibilidad con Windows ARM64, instale las [herramientas de compilación opcionales para ARM64 y el SDK de Windows 10 para ARM64](https://blogs.windows.com/buildingapps/2018/11/15/official-support-for-windows-10-on-arm-development/).

## <a name="install-the-speech-sdk"></a>Instalación de Speech SDK

Para instalar Speech SDK para Unity, siga estos pasos:

1. Descargue y abra el [SDK de Voz para Unity](https://aka.ms/csspeech/unitypackage), que se incluye como un paquete de recursos de Unity (.unitypackage) y que ya debe estar asociado con Unity. Cuando se abra el paquete de recursos, aparecerá el cuadro de diálogo **Import Unity Package** (Importar paquete de Unity). Es posible que tenga que crear y abrir un proyecto vacío para que funcione este paso.

   [![Cuadro de diálogo Import Unity Package en el editor de Unity](~/articles/cognitive-services/speech-service/media/sdk/qs-csharp-unity-01-import.png)](~/articles/cognitive-services/speech-service/media/sdk/qs-csharp-unity-01-import.png#lightbox)

1. Asegúrese de que se seleccionan todos los archivos y seleccione **Import** (Importar). Transcurridos unos instantes, el paquete de recursos de Unity se importa en el proyecto.

Para más información sobre la importación de paquetes de recursos en Unity, consulte la [documentación de Unity](https://docs.unity3d.com/Manual/AssetPackages.html).

Ahora puede continuar con la sección [Pasos siguientes](#next-steps).

## <a name="next-steps"></a>Pasos siguientes

[!INCLUDE [windows](../quickstart-list.md)]
