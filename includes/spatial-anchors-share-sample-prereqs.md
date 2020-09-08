---
author: ramonarguelles
ms.service: spatial-anchors
ms.topic: include
ms.date: 07/31/2020
ms.author: rgarcia
ms.openlocfilehash: 310c0f547ee11a3243589c364755a30a84be1a25
ms.sourcegitcommit: 85eb6e79599a78573db2082fe6f3beee497ad316
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/05/2020
ms.locfileid: "87810193"
---
## <a name="prerequisites"></a>Prerrequisitos

Para completar este tutorial, asegúrese de disponer de los siguientes elementos:

* Ha leído completamente [Introducción a Azure Spatial Anchors](../articles/spatial-anchors/overview.md).
* Ha completado uno de los [inicios rápidos en 5 minutos](../articles/spatial-anchors/index.yml).
* Conocimiento básico de C# y Unity.
* Conocimiento básico de <a href="https://developers.google.com/ar/discover/" target="_blank">ARCore</a> si desea usar Android, o <a href="https://developer.apple.com/arkit/" target="_blank">ARKit</a> si desea usar iOS.
* Un equipo Windows con <a href="https://www.visualstudio.com/downloads/" target="_blank">Visual Studio 2017</a>, o cualquier versión posterior, instalado, con la carga de trabajo de **desarrollo web y ASP.NET**.
* [SDK de .NET Core 3.1](https://dotnet.microsoft.com/download).
* Uno o varios dispositivos (iOS o Android) en los que implementar y ejecutar una aplicación.
  * Si usa Android, debe tener:
    * <a href="https://developer.android.com/studio/" target="_blank">Android Studio 3.3</a> (o cualquier versión posterior), <a href="https://unity3d.com/get-unity/download" target="_blank">Unity 2019.4 (LTS)</a> y <a href="https://git-scm.com/download/win" target="_blank">Git para Windows</a> instalados en el equipo Windows.
    * Un dispositivo Android <a href="https://developer.android.com/studio/debug/dev-options" target="_blank">habilitado para el desarrollo</a> y <a href="https://developers.google.com/ar/discover/supported-devices" target="_blank">preparado para ARCore</a>.
  * Si usa iOS, debe tener:
    * Un equipo macOS que tenga instalados <a href="https://geo.itunes.apple.com/us/app/xcode/id497799835?mt=12" target="_blank">Xcode 10</a> (o cualquier versión posterior), <a href="https://cocoapods.org" target="_blank">CocoaPods</a> y <a href="https://unity3d.com/get-unity/download" target="_blank">Unity 2019.4 (LTS)</a>.
    * Un dispositivo iOS <a href="https://developer.apple.com/documentation/arkit/verifying_device_support_and_user_permission" target="_blank">compatible con ARKit</a> habilitado para el desarrollo.
    * GIT instalado mediante Homebrew. Escriba el siguiente comando en una sola línea de Terminal: `/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"` A continuación, ejecute `brew install git`.
