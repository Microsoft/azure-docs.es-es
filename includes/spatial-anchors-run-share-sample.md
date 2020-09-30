---
author: ramonarguelles
ms.service: spatial-anchors
ms.topic: include
ms.date: 8/27/2020
ms.author: rgarcia
ms.openlocfilehash: 8e3f58a5a88495263396c4e915a6363ad065c202
ms.sourcegitcommit: c52e50ea04dfb8d4da0e18735477b80cafccc2cf
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/08/2020
ms.locfileid: "89570010"
---
## <a name="android"></a>[Android](#tab/Android)

El ejemplo de Android de Java admite el uso compartido entre dispositivos.
Abra el archivo `SharedActivity.java` de la carpeta de ejemplos en Android Studio. Escriba la dirección URL que obtuvo en el paso anterior (de la implementación de Azure de la aplicación web de ASP.NET) como valor de `SharingAnchorsServiceUrl` en el archivo `SharedActivity.java`. Reemplace `index.html` en la dirección URL por `api/anchors`. Debería ser parecido a este: `https://<app_name>.azurewebsites.net/api/anchors`.

[!INCLUDE [Run shared sample](spatial-anchors-run-sample.md)]

## <a name="ios"></a>[iOS](#tab/iOS)

El ejemplo de iOS de Objective-C admite el uso compartido entre dispositivos.
Abra el archivo `SharedDemoViewController.m` de la carpeta de ejemplos. Escriba la dirección URL que obtuvo en el paso anterior (de la implementación de Azure de la aplicación web de ASP.NET) como valor de `SharingAnchorsServiceUrl` en el archivo `SharedDemoViewController.m`. Reemplace `index.html` en la dirección URL por `api/anchors`. Debería ser parecido a este: `https://<app_name>.azurewebsites.net/api/anchors`.

[!INCLUDE [Run shared sample](spatial-anchors-run-sample.md)]

## <a name="xamarin"></a>[Xamarin](#tab/Xamarin)

Los ejemplos de Xamarin Android e iOS admiten el uso compartido entre dispositivos.
Abra el archivo `AccountDetails.cs` de la carpeta de ejemplos. Escriba la dirección URL que obtuvo en el paso anterior (de la implementación de Azure de la aplicación web de ASP.NET) como valor de `AnchorSharingServiceUrl` en el archivo `AccountDetails.cs`. Reemplace `index.html` en la dirección URL por `api/anchors`. Debería ser parecido a este: `https://<app_name>.azurewebsites.net/api/anchors`.

[!INCLUDE [Run shared sample](spatial-anchors-run-sample.md)]

## <a name="unity"></a>[Unity](#tab/Unity)

[!INCLUDE [Open Unity Project](spatial-anchors-open-unity-project.md)]

### <a name="set-up-an-android-device"></a>Configuración de un dispositivo Android

[!INCLUDE [Android Unity Build Settings](spatial-anchors-unity-android-build-settings.md)]

### <a name="set-up-an-ios-device"></a>Configuración de un dispositivo iOS

[!INCLUDE [iOS Unity Build Settings](spatial-anchors-unity-ios-build-settings.md)]

[!INCLUDE [Configure Unity Scene](spatial-anchors-unity-configure-scene.md)]

En el panel del **proyecto**, vaya a `Assets\AzureSpatialAnchors.Examples\Resources`. Seleccione `SpatialAnchorSamplesConfig`. Además, en el panel **Inspector**, escriba `Sharing Anchors Service url` (de la implementación en Azure de la aplicación web de ASP.NET) como el valor de `Base Sharing Url`, que reemplaza `index.html` por `api/anchors`. Debería ser parecido a este: `https://<app_name>.azurewebsites.net/api/anchors`.

Para guardar la escena, seleccione **File** > **Save** (Archivo > Guardar).

## <a name="deploy-to-your-device"></a>Implementación en el dispositivo

### <a name="deploy-to-android-device"></a>Implementación en un dispositivo Android

Inicie sesión en el dispositivo Android y conéctelo a un equipo mediante un cable USB.

Seleccione **File** > **Build Settings** (Archivo > Configuración de compilación) para abrir **Build Settings** (Configuración de compilación).

En **Scenes In Build** (Escenas en compilación), asegúrese de que haya una marca de verificación junto a cada escena.

Asegúrese de que **Export Project** (Exportar proyecto) no tiene una marca de verificación. Seleccione **Build And Run** (Compilar y ejecutar). Se le pedirá que guarde el archivo `.apk`. Puede elegir cualquier nombre.

[!INCLUDE [Run shared sample](spatial-anchors-run-sample.md)]

### <a name="deploy-to-an-ios-device"></a>Implementación en un dispositivo iOS

Seleccione **File** > **Build Settings** (Archivo > Configuración de compilación) para abrir **Build Settings** (Configuración de compilación).

En **Scenes In Build** (Escenas en compilación), asegúrese de que haya una marca de verificación junto a cada escena.

[!INCLUDE [Configure Xcode](spatial-anchors-unity-ios-xcode.md)]

[!INCLUDE [Run shared sample](spatial-anchors-run-sample.md)]

En Xcode, seleccione **Stop** (Detener) para detener la aplicación.
