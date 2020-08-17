---
title: 'Inicio rápido: Creación de una aplicación Xamarin Android'
description: En este tutorial, aprenderá a compilar una aplicación para Android con Xamarin mediante Spatial Anchors.
author: craigktreasure
manager: vriveras
services: azure-spatial-anchors
ms.author: crtreasu
ms.date: 07/22/2019
ms.topic: quickstart
ms.service: azure-spatial-anchors
ms.openlocfilehash: 8a7bb300bb634b342b1141d175b0046c7d2d7e2b
ms.sourcegitcommit: 85eb6e79599a78573db2082fe6f3beee497ad316
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/05/2020
ms.locfileid: "87810316"
---
# <a name="quickstart-create-a-xamarin-android-app-with-azure-spatial-anchors"></a>Inicio rápido: Creación de una aplicación para Android en Xamarin con Azure Spatial Anchors

En este inicio rápido se describe cómo crear una aplicación para Android con Xamarin mediante [Azure Spatial Anchors](../overview.md). Azure Spatial Anchors es un servicio multiplataforma para desarrolladores que le permite crear experiencias de realidad mixta mediante objetos cuya ubicación persiste en todos los dispositivos a lo largo del tiempo. Cuando haya terminado, tendrá una aplicación para Android que puede guardar y recuperar un anclaje espacial.

Aprenderá a:

> [!div class="checklist"]
> * Crear una cuenta de Spatial Anchors
> * Configurar la clave y el identificador de la cuenta de Spatial Anchors
> * Realizar la implementación y ejecución en un dispositivo Android

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Prerrequisitos

Para completar esta guía de inicio rápido, asegúrese de que dispone de lo siguiente:
- Un equipo Windows o macOS:
  - Si usa Windows:
    - Una versión actualizada de <a href="https://www.visualstudio.com/downloads/" target="_blank">Visual Studio 2019 16.2+</a>.
    - <a href="https://git-scm.com/download/win" target="_blank">Git para Windows</a>
    - <a href="https://git-lfs.github.com/">Git LFS</a>.
  - Si usa macOS:
    - Una versión actualizada de <a href="https://docs.microsoft.com/visualstudio/mac/installation?view=vsmac-2019" target="_blank">Visual Studio para Mac 8.1+</a>.
    - <a href="https://git-scm.com/download/mac" target="_blank">Git para macOS</a>
    - <a href="https://git-lfs.github.com/">Git LFS</a>.
- La versión más reciente de Xamarin.Android instalada y en ejecución en la plataforma que elija. Para obtener una guía sobre la instalación de Xamarin.Android, consulte las guías de [instalación de Xamarin.Android](https://docs.microsoft.com/xamarin/android/get-started/installation/index).
- Un dispositivo Android <a href="https://developer.android.com/studio/debug/dev-options" target="_blank">habilitado para el desarrollo</a> y <a href="https://developers.google.com/ar/discover/supported-devices" target="_blank">preparado para ARCore</a>.
  - Puede que se necesiten controladores de dispositivo adicionales para que el equipo se comunique con el dispositivo Android. Para más información, consulte [esta página](https://developer.android.com/studio/run/device.html).
- La aplicación debe tener como destino ARCore **1.8**.

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

## <a name="deploy-the-app-to-your-android-device"></a>Implementación de la aplicación en el dispositivo Android

Encienda el dispositivo Android, inicie sesión y conéctelo al equipo mediante un cable USB.

Establezca el proyecto de inicio en **SampleXamarin.Android**, cambie el valor de **Configuración de solución** a **Versión** y seleccione el dispositivo en el que quiere realizar la implementación en el menú desplegable del selector de dispositivos.

# <a name="windows"></a>[Windows](#tab/deploy-windows)

![Configuración de Visual Studio](./media/get-started-xamarin-android/visual-studio-windows-configuration.png)

Seleccione **Depurar** > **Iniciar depuración** para implementar e iniciar la aplicación.

# <a name="macos"></a>[macOS](#tab/deploy-macos)

![Configuración de Visual Studio](./media/get-started-xamarin-android/visual-studio-macos-configuration.jpg)

Seleccione **Ejecutar** > **Iniciar sin depuración** para implementar e iniciar la aplicación.

---

En la aplicación, seleccione **Básico** para ejecutar la demostración y siga las instrucciones para colocar y recuperar un delimitador.

> ![Captura de pantalla 1](./media/get-started-xamarin-android/screenshot-1.jpg)
> ![Captura de pantalla 2](./media/get-started-xamarin-android/screenshot-2.jpg)
> ![Captura de pantalla 3](./media/get-started-xamarin-android/screenshot-3.jpg)

[!INCLUDE [Clean-up section](../../../includes/clean-up-section-portal.md)]

[!INCLUDE [Next steps](../../../includes/spatial-anchors-quickstarts-nextsteps.md)]

> [!div class="nextstepaction"]
> [Tutorial: Uso compartido de Spatial Anchors entre dispositivos](../tutorials/tutorial-share-anchors-across-devices.md)
