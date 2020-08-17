---
title: 'Inicio rápido: Creación de una aplicación HoloLens con DirectX'
description: En este inicio rápido, aprenderá a crear una aplicación HoloLens con Spatial Anchors.
author: craigktreasure
manager: virivera
services: azure-spatial-anchors
ms.author: crtreasu
ms.date: 02/24/2019
ms.topic: quickstart
ms.service: azure-spatial-anchors
ms.openlocfilehash: e6d728647bb654ee2645bb1e3eb5968dcab337a8
ms.sourcegitcommit: 85eb6e79599a78573db2082fe6f3beee497ad316
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/05/2020
ms.locfileid: "87810361"
---
# <a name="quickstart-create-a-hololens-app-with-azure-spatial-anchors-in-cwinrt-and-directx"></a>Inicio rápido: Creación de una aplicación HoloLens con Azure Spatial Anchors en C++/WinRT y DirectX

En este inicio rápido, se describe cómo crear una aplicación HoloLens con [Azure Spatial Anchors](../overview.md) en C++/WinRT y DirectX. Azure Spatial Anchors es un servicio multiplataforma para desarrolladores que le permite crear experiencias de realidad mixta mediante objetos cuya ubicación persiste en todos los dispositivos a lo largo del tiempo. Cuando haya terminado, tendrá una aplicación HoloLens que puede guardar y recuperar un delimitador espacial.

Aprenderá a:

> [!div class="checklist"]
> * Crear una cuenta de Spatial Anchors
> * Configurar la clave y el identificador de la cuenta de Spatial Anchors
> * Realizar implementaciones y ejecuciones en un dispositivo HoloLens

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Prerrequisitos

Para completar esta guía de inicio rápido, asegúrese de que dispone de lo siguiente:
- Una máquina Windows con <a href="https://www.visualstudio.com/downloads/" target="_blank">Visual Studio 2019 o versiones posteriores</a> instalado, con la carga de trabajo **Plataforma universal de Windows** y el componente **Windows 10 SDK (10.0.18362.0 o versiones posteriores)** . También se debe instalar <a href="https://git-scm.com/download/win" target="_blank">Git para Windows</a> y <a href="https://git-lfs.github.com/">Git LFS</a>.
- La [extensión de Visual Studio para C++/WinRT (VSIX)](https://aka.ms/cppwinrt/vsix) para Visual Studio instalada desde [Visual Studio Marketplace](https://marketplace.visualstudio.com/).
- Un dispositivo HoloLens con el [modo de desarrollador](https://docs.microsoft.com/windows/mixed-reality/using-visual-studio) habilitado. Este artículo requiere un dispositivo HoloLens con la [actualización de octubre de 2018 de Windows 10](https://docs.microsoft.com/windows/mixed-reality/release-notes-october-2018 ) (también conocida como RS5). Para actualizar a la versión más reciente en HoloLens, abra la aplicación **Settings** (Configuración), vaya a **Update & Security** (Actualización y seguridad) y, a continuación, seleccione el botón **Check for updates** (Buscar actualizaciones).
- La aplicación debe establecer la funcionalidad **spatialPerception** en su manifiesto AppX.

[!INCLUDE [Create Spatial Anchors resource](../../../includes/spatial-anchors-get-started-create-resource.md)]

## <a name="open-the-sample-project"></a>Apertura del proyecto de ejemplo

[!INCLUDE [Clone Sample Repo](../../../includes/spatial-anchors-clone-sample-repository.md)]

Abra `HoloLens\DirectX\SampleHoloLens.sln` en Visual Studio.

## <a name="configure-account-identifier-and-key"></a>Configuración del identificador y la clave de la cuenta

El paso siguiente es configurar la aplicación para usar el identificador de cuenta y la clave de cuenta. Los copió en un editor cuando [configuró el recurso de Spatial Anchors](#create-a-spatial-anchors-resource).

Abra `HoloLens\DirectX\SampleHoloLens\ViewController.cpp`.

Busque el campo `SpatialAnchorsAccountKey` y reemplace `Set me` por la clave de la cuenta.

Busque el campo `SpatialAnchorsAccountId` y reemplace `Set me` por el identificador de la cuenta.

Busque el `SpatialAnchorsAccountDomain` campo y reemplácelo `Set me` por el dominio de la cuenta.

## <a name="deploy-the-app-to-your-hololens"></a>Implementación de la aplicación en el dispositivo HoloLens

Cambiar **Configuración de la solución** a **Versión**, cambie **Plataforma de la solución** a **x86** y seleccione **Dispositivo**  en las opciones de destino de implementación.

Si usa HoloLens 2, utilice **ARM64** como **Plataforma de soluciones**, en lugar de **x86**.

![Configuración de Visual Studio](./media/get-started-hololens/visual-studio-configuration.png)

Encienda el dispositivo HoloLens, inicie sesión y conéctelo al equipo mediante un cable USB.

Seleccione **Depurar** > **Iniciar depuración** para implementar la aplicación y comenzar la depuración.

Siga las instrucciones de la aplicación para colocar y recuperar un delimitador.

En Visual Studio, seleccione **Detener depuración** o presione **Mayús + F5** para detener la depuración.

[!INCLUDE [Clean-up section](../../../includes/clean-up-section-portal.md)]

[!INCLUDE [Next steps](../../../includes/spatial-anchors-quickstarts-nextsteps.md)]

> [!div class="nextstepaction"]
> [Tutorial: Uso compartido de Spatial Anchors entre dispositivos](../tutorials/tutorial-share-anchors-across-devices.md)
