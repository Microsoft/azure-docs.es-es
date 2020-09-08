---
title: Implementación de un ejemplo de Unity en el escritorio
description: Inicio rápido que muestra cómo obtener el ejemplo de Unity en un equipo de escritorio
author: christophermanthei
ms.author: chmant
ms.date: 03/20/2020
ms.topic: quickstart
ms.openlocfilehash: 26531e80292a53400671ac3507b47371c1fbbcad
ms.sourcegitcommit: c5021f2095e25750eb34fd0b866adf5d81d56c3a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88796823"
---
# <a name="quickstart-deploy-unity-sample-to-desktop"></a>Inicio rápido: Implementación de un ejemplo de Unity en el escritorio

En este inicio rápido se explica cómo implementar y ejecutar la aplicación de ejemplo de inicio rápido para Unity en un equipo de escritorio.

Aprenderá lo siguiente:

> [!div class="checklist"]
>
>* Compilación de la aplicación de ejemplo de inicio rápido para escritorio
>* Implementación del ejemplo en un equipo
>* Ejecución del ejemplo en un equipo

## <a name="prerequisites"></a>Requisitos previos

En este inicio rápido, se va a implementar el proyecto de ejemplo de [Inicio rápido: Representación de un modelo con Unity](render-model.md).

Asegúrese de que las credenciales se guardan correctamente con la escena y que puede conectarse a una sesión desde el editor de Unity.

## <a name="disable-virtual-reality-support"></a>Deshabilitación de la compatibilidad con la realidad virtual

Actualmente solo se admiten aplicaciones de escritorio planas en el escritorio, por lo que la compatibilidad con VR debe estar deshabilitada.

1. Abra *Edit > Project Settings...* (Editar > Configuración del proyecto).
1. Seleccione **Player** (Reproductor) a la izquierda.
1. Seleccione la pestaña **Universal Windows Platform settings** (Configuración de Plataforma universal de Windows).
1. Expanda **XR Settings** (Configuración de XR).
1. Deshabilite **Virtual Reality Supported** (Se admite realidad virtual).\
    ![Configuración del reproductor](./media/unity-disable-xr.png)
1. Encima de *XR Settings* (Configuración de XR), expanda **Publishing Settings** (Configuración de publicación).
1. En **Supported Device Families** (Familias de dispositivos admitidas), asegúrese de que la casilla **Desktop** (Escritorio) está activada.

## <a name="build-the-sample-project"></a>Compilación del proyecto de ejemplo

1. Abra *File -> Build Settings* (Archivo -> Configuración de la compilación).
1. Cambie el valor de *Plataforma* a **Plataforma universal de Windows** (**Equipo independiente** también se admite, pero no se usa aquí, consulte las [limitaciones de la plataforma](../reference/limits.md#platform-limitations)).
1. Establezca *Target Device* (Dispositivo de destino) en **PC**.
1. Establezca *Architecture* (Arquitectura) en **x86**.
1. Establezca *Build Type* (Tipo de compilación) en **D3D Project** (Proyecto de D3D).\
  ![Configuración de compilación](./media/unity-build-settings-pc.png)
1. Seleccione **Switch to Platform** (Cambiar a Plataforma).
1. Al presionar **Build** (Compilar) (o "Build And Run" [Compilar y ejecutar]), se le pedirá que seleccione una carpeta donde guardar la solución.
1. Abra el archivo **Quickstart.sln** generado con Visual Studio.
1. Cambie la configuración a **Release** (Versión) y **x86**.
1. Cambie el modo del depurador a **Local Machine** (Máquina local).\
  ![Configuración de la solución](./media/unity-deploy-config-pc.png)
1. Compile la solución.

## <a name="launch-the-sample-project"></a>Inicio del proyecto de ejemplo

Inicie el depurador en Visual Studio (F5). La aplicación se implementará automáticamente en el equipo.

Se iniciará la aplicación de ejemplo y comenzará una nueva sesión. Al cabo de un rato, la sesión está lista y aparece el modelo representado de forma remota.
Si quiere iniciar el ejemplo una segunda vez más tarde, también puede encontrarlo ahora en el menú Inicio.

## <a name="next-steps"></a>Pasos siguientes

En el siguiente inicio rápido, echaremos un vistazo a la conversión de un modelo personalizado.

> [!div class="nextstepaction"]
> [Inicio rápido: Conversión de un modelo para su representación](convert-model.md)
