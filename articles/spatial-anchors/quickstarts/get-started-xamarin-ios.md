---
title: 'Inicio rápido: Creación de una aplicación Xamarin iOS'
description: En este inicio rápido, aprenderá a compilar una aplicación para iOS con Xamarin mediante Spatial Anchors.
author: craigktreasure
manager: vriveras
services: azure-spatial-anchors
ms.author: crtreasu
ms.date: 07/22/2019
ms.topic: quickstart
ms.service: azure-spatial-anchors
ms.openlocfilehash: 12fb644430d4bfd6444c6bbf09d37b4662ca6fd1
ms.sourcegitcommit: 85eb6e79599a78573db2082fe6f3beee497ad316
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/05/2020
ms.locfileid: "87810129"
---
# <a name="quickstart-create-a-xamarin-ios-app-with-azure-spatial-anchors"></a>Inicio rápido: Creación de una aplicación para iOS en Xamarin con Azure Spatial Anchors

En este inicio rápido se describe cómo crear una aplicación para iOS con Xamarin mediante [Azure Spatial Anchors](../overview.md). Azure Spatial Anchors es un servicio multiplataforma para desarrolladores que le permite crear experiencias de realidad mixta mediante objetos cuya ubicación persiste en todos los dispositivos a lo largo del tiempo. Cuando haya terminado, tendrá una aplicación para iOS que puede guardar y recuperar un anclaje espacial.

Aprenderá a:

> [!div class="checklist"]
> * Crear una cuenta de Spatial Anchors
> * Configurar la clave y el identificador de la cuenta de Spatial Anchors
> * Implementarlo y ejecutarlo en un dispositivo iOS

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Prerrequisitos

Para completar esta guía de inicio rápido, asegúrese de que dispone de lo siguiente:
- Un equipo Mac que ejecute macOS High Sierra (10.13) o posterior con:
  - La versión más reciente de Xcode y el SDK de iOS instalada desde el [App Store](https://itunes.apple.com/us/app/xcode/id497799835?mt=12).
  - Una versión actualizada de <a href="https://docs.microsoft.com/visualstudio/mac/installation?view=vsmac-2019" target="_blank">Visual Studio para Mac 8.1+</a>.
  - <a href="https://git-scm.com/download/mac" target="_blank">Git para macOS</a>
  - <a href="https://git-lfs.github.com/">Git LFS</a>.

[!INCLUDE [Create Spatial Anchors resource](../../../includes/spatial-anchors-get-started-create-resource.md)]

## <a name="open-the-sample-project"></a>Apertura del proyecto de ejemplo

[!INCLUDE [Clone Sample Repo](../../../includes/spatial-anchors-clone-sample-repository.md)]

Abra `Xamarin/SampleXamarin.sln` en Visual Studio.

## <a name="configure-account-identifier-and-key"></a>Configuración del identificador y la clave de la cuenta

El paso siguiente es configurar la aplicación para usar el identificador de cuenta y la clave de cuenta. Los copió en un editor cuando [configuró el recurso de Spatial Anchors](#create-a-spatial-anchors-resource).

Abra `Xamarin/SampleXamarin.Common/AccountDetails.cs`.

Busque el campo `SpatialAnchorsAccountKey` y reemplace `Set me` por la clave de la cuenta.

Busque el campo `SpatialAnchorsAccountId` y reemplace `Set me` por el identificador de la cuenta.

Busque el campo `SpatialAnchorsAccountDomain` y reemplace `Set me` por el dominio de la cuenta.

## <a name="deploy-the-app-to-your-ios-device"></a>Implementación de la aplicación en el dispositivo iOS

Encienda el dispositivo iOS, inicie sesión y conéctelo al equipo mediante un cable USB.

Establezca el proyecto de inicio en **SampleXamarin.iOS**, cambie el valor de **Configuración de solución** a **Versión** y seleccione el dispositivo en el que quiere realizar la implementación en el menú desplegable del selector de dispositivos.

![Configuración de Visual Studio](./media/get-started-xamarin-iOS/visual-studio-macos-configuration.jpg)

Seleccione **Ejecutar** > **Iniciar sin depuración** para implementar e iniciar la aplicación.

En la aplicación, seleccione **Básico** para ejecutar la demostración y siga las instrucciones para colocar y recuperar un delimitador.

> ![Captura de pantalla 1](./media/get-started-xamarin-ios/screenshot-1.jpg)
> ![Captura de pantalla 2](./media/get-started-xamarin-ios/screenshot-2.jpg)
> ![Captura de pantalla 3](./media/get-started-xamarin-ios/screenshot-3.jpg)

[!INCLUDE [Clean-up section](../../../includes/clean-up-section-portal.md)]

[!INCLUDE [Next steps](../../../includes/spatial-anchors-quickstarts-nextsteps.md)]

> [!div class="nextstepaction"]
> [Tutorial: Uso compartido de Spatial Anchors entre dispositivos](../tutorials/tutorial-share-anchors-across-devices.md)
