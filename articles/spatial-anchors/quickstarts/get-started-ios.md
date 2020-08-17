---
title: 'Inicio rápido: Creación de una aplicación iOS'
description: Aprenda a crear una aplicación iOS con Azure Spatial Anchors mediante programación en Swift u Objective-C.
author: craigktreasure
manager: vriveras
services: azure-spatial-anchors
ms.author: crtreasu
ms.date: 02/24/2019
ms.topic: quickstart
ms.service: azure-spatial-anchors
ms.custom: has-adal-ref
ms.openlocfilehash: 2df66979d1e8f400f2630f5e531cd201244fdc6d
ms.sourcegitcommit: 85eb6e79599a78573db2082fe6f3beee497ad316
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/05/2020
ms.locfileid: "87809993"
---
# <a name="quickstart-create-an-ios-app-with-azure-spatial-anchors-in-either-swift-or-objective-c"></a>Inicio rápido: Crear una aplicación iOS con Azure Spatial Anchors en Swift u Objective-C

En esta guía de inicio rápido se describe cómo crear una aplicación iOS con [Azure Spatial Anchors](../overview.md) en Swift o en Objective-C. Azure Spatial Anchors es un servicio multiplataforma para desarrolladores que le permite crear experiencias de realidad mixta mediante objetos cuya ubicación persiste en todos los dispositivos a lo largo del tiempo. Cuando haya terminado, tendrá una aplicación iOS de ARKit que puede guardar y recuperar un delimitador espacial.

Aprenderá a:

> [!div class="checklist"]
> * Crear una cuenta de Spatial Anchors
> * Configurar la clave y el identificador de la cuenta de Spatial Anchors
> * Implementarlo y ejecutarlo en un dispositivo iOS

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Prerrequisitos

Para completar esta guía de inicio rápido, asegúrese de que dispone de lo siguiente:

- Una máquina macOS habilitada para el desarrollo que tenga instaladas las versiones más recientes de <a href="https://geo.itunes.apple.com/us/app/xcode/id497799835?mt=12" target="_blank">Xcode</a> y <a href="https://cocoapods.org" target="_blank">CocoaPods</a>.
- Git instalado mediante HomeBrew:
  1. Escriba el siguiente comando en una sola línea en el terminal: `/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"`. 
  1. Ejecute `brew install git` y `brew install git-lfs`.
  1. Actualice la configuración de Git con `git lfs install` (para el usuario actual) o `git lfs install --system` (para todo el sistema).
- Un dispositivo iOS <a href="https://developer.apple.com/documentation/arkit/verifying_device_support_and_user_permission" target="_blank">compatible con ARKit</a> habilitado para el desarrollo.

[!INCLUDE [Create Spatial Anchors resource](../../../includes/spatial-anchors-get-started-create-resource.md)]

## <a name="open-the-sample-project"></a>Apertura del proyecto de ejemplo

Use Terminal para realizar las acciones siguientes:

[!INCLUDE [Clone Sample Repo](../../../includes/spatial-anchors-clone-sample-repository.md)]

Instale los pods necesarios mediante CocoaPods:

# <a name="swift"></a>[Swift](#tab/openproject-swift)

Vaya a `iOS/Swift/`.

```bash
cd ./iOS/Swift/
```

# <a name="objective-c"></a>[Objective-C](#tab/openproject-objc)

Vaya a `iOS/Objective-C/`.

```bash
cd ./iOS/Objective-C/
```

---

Ejecute `pod install --repo-update` para instalar los pods de CocoaPods para el proyecto.

Ahora abra `.xcworkspace` en Xcode.

> [!NOTE]
> Consulte los pasos de solución de problemas [aquí](#cocoapods-issues-on-macos-catalina-1015) si tiene problemas con CocoaPod después de actualizar a macOS Catalina (10.15).

# <a name="swift"></a>[Swift](#tab/openproject-swift)

```bash
open ./SampleSwift.xcworkspace
```

# <a name="objective-c"></a>[Objective-C](#tab/openproject-objc)

```bash
open ./SampleObjC.xcworkspace
```

---

## <a name="configure-account-identifier-and-key"></a>Configuración del identificador y la clave de la cuenta

El paso siguiente es configurar la aplicación para usar el identificador de cuenta y la clave de cuenta. Los copió en un editor cuando [configuró el recurso de Spatial Anchors](#create-a-spatial-anchors-resource).

# <a name="swift"></a>[Swift](#tab/openproject-swift)

Abra `iOS/Swift/SampleSwift/ViewControllers/BaseViewController.swift`.

Busque el campo `spatialAnchorsAccountKey` y reemplace `Set me` por la clave de la cuenta.

Busque el campo `spatialAnchorsAccountId` y reemplace `Set me` por el identificador de la cuenta.

Busque el campo `spatialAnchorsAccountDomain` y reemplace `Set me` por el dominio de la cuenta.

# <a name="objective-c"></a>[Objective-C](#tab/openproject-objc)

Abra `iOS/Objective-C/SampleObjC/BaseViewController.m`.

Busque el campo `SpatialAnchorsAccountKey` y reemplace `Set me` por la clave de la cuenta.

Busque el campo `SpatialAnchorsAccountId` y reemplace `Set me` por el identificador de la cuenta.

Busque el campo `SpatialAnchorsAccountDomain` y reemplace `Set me` por el dominio de la cuenta.

---

## <a name="deploy-the-app-to-your-ios-device"></a>Implementación de la aplicación en el dispositivo iOS

Conecte el dispositivo iOS al equipo Mac y establezca el **esquema activo** en su dispositivo iOS.

![Selección del dispositivo](./media/get-started-ios/select-device.png)

Seleccione **Build and then run the current scheme** (Compilar y luego ejecutar el esquema actual).

![Implementación y ejecución](./media/get-started-ios/deploy-run.png)

> [!NOTE]
> Si ve un error `library not found for -lPods-SampleObjC`, es probable que haya abierto el archivo `.xcodeproj` en lugar de `.xcworkspace`. Abra `.xcworkspace` e inténtelo de nuevo.

En Xcode, presione **Stop** (Detener) para detener la aplicación.

## <a name="troubleshooting"></a>Solución de problemas

### <a name="cocoapods-issues-on-macos-catalina-1015"></a>Problemas con CocoaPods en macOS Catalina (10.15)

Si ha actualizado recientemente a macOS Catalina (10.15) y tenía CocoaPods instalado con antelación, puede que CocoaPods esté en un estado interrumpido y no pueda configurar correctamente los pods y los archivos de proyecto de `.xcworkspace`. Para resolver este problema, debe volver a instalar CocoaPods ejecutando los siguientes comandos:

```shell
brew update
brew install cocoapods --build-from-source
brew link --overwrite cocoapods
```

### <a name="app-crashes-when-deploying-to-ios-1031-from-a-personal-provisioning-profiledeveloper-account"></a>La aplicación se bloquea cuando se implementa en iOS 10.3.1 desde un perfil de aprovisionamiento personal o una cuenta de desarrollador 

Si implementa la aplicación iOS en iOS 10.3.1 desde un perfil de aprovisionamiento personal o una cuenta de desarrollador, es posible que vea este error: `Library not loaded: @rpath/ADAL...`. 

Para resolver el problema:

- Use un perfil de aprovisionamiento que no sea un perfil de equipo personal (cuenta de desarrollador de pago).
- Implemente la aplicación en un dispositivo iOS que ejecute iOS 13.3 o anterior, o en uno que ejecute la versión beta o de lanzamiento de iOS 13.4.
- Obtenga más información sobre este problema en [Stack Overflow](https://stackoverflow.com/questions/60015309/running-ios-apps-causes-runtime-error-for-frameworks-code-signature-invalid).


[!INCLUDE [Clean-up section](../../../includes/clean-up-section-portal.md)]

[!INCLUDE [Next steps](../../../includes/spatial-anchors-quickstarts-nextsteps.md)]

> [!div class="nextstepaction"]
> [Tutorial: Uso compartido de Spatial Anchors entre dispositivos](../tutorials/tutorial-share-anchors-across-devices.md)
