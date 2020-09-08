---
title: Espacio provisional
description: Describe la configuración del espacio provisional y casos de uso
author: christophermanthei
ms.author: chmant
ms.date: 03/07/2020
ms.topic: article
ms.custom: devx-track-csharp
ms.openlocfilehash: 5809494fb8b619569316a24816a2e5d943dee6b4
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89013136"
---
# <a name="stage-space"></a>Espacio provisional

Cuando se ejecuta ARR en un dispositivo que proporciona datos de seguimiento de la cabeza, como HoloLens 2, la posición de la cabeza se envía tanto a la aplicación de usuario como al servidor. El espacio en el que se define la transformación de la cabeza se llama *espacio provisional*.

Para alinear el contenido local y remoto, se supone que el espacio provisional y el espacio mundial son idénticos en el cliente y el servidor. Si el usuario decide agregar una transformación adicional en la parte superior de la cámara, debe enviarse al servidor también para alinear el contenido local y remoto correctamente.

Los motivos típicos para mover el espacio provisional son las [herramientas de bloqueo de mundo](https://microsoft.github.io/MixedReality-WorldLockingTools-Unity/README.html) u otras técnicas de delimitación del mundo real, así como mover un personaje virtual en la realidad virtual.

## <a name="stage-space-settings"></a>Configuración del espacio provisional

> [!CAUTION]
> EXPERIMENTAL: Esta característica es experimental y cambiará con el tiempo. Por lo tanto, la actualización a versiones más recientes del SDK de cliente puede requerir trabajo adicional para actualizar el código. La implementación actual interrumpirá la alineación del contenido local o remoto durante un breve período de tiempo al cambiar el origen del espacio provisional.
Por lo tanto, actualmente solo está pensada para usarse con fines de bloqueo de mundo, como los delimitadores que solo muestran cambios muy pequeños a lo largo del tiempo.

Para informar al servidor de que se aplica una transformación adicional al espacio provisional, se debe enviar el origen definido por una posición y una rotación en el espacio mundial. Se puede tener acceso a esta configuración a través de la *configuración del espacio provisional*.

> [!IMPORTANT]
> En la [simulación de escritorio](../../concepts/graphics-bindings.md), la ubicación de espacio mundial de la cámara se proporciona por la aplicación de usuario. En este caso, se debe omitir el establecimiento del origen del espacio provisional, ya que ya se ha multiplicado en la transformación de la cámara.

```cs
void ChangeStageSpace(AzureSession session)
{
    StageSpaceSettings settings = session.Actions.StageSpaceSettings;

    // Set position and rotation to the world-space transform of the stage space.
    settings.Position = new Double3(0, 0, 0);
    settings.Rotation = new Quaternion(0, 0, 0, 1);
}
```

```cpp
void ChangeStageSpace(ApiHandle<AzureSession> session)
{
    ApiHandle<StageSpaceSettings> settings = session->Actions()->GetStageSpaceSettings();

    // Set position and rotation to the world-space transform of the stage space.
    settings->SetPosition({0, 0, 0});
    settings->SetRotation({0, 0, 0, 1});
}
```

## <a name="unity-stage-space-script"></a>Script de espacio provisional de Unity

La integración de Unity proporciona un script denominado **StageSpace** que se puede agregar al elemento GameObject principal de la cámara. Una vez presente, este script se encargará de establecer el origen del espacio provisional.

## <a name="next-steps"></a>Pasos siguientes

* [Enlace de gráficos](../../concepts/graphics-bindings.md)
* [Reproyección de fase tardía](late-stage-reprojection.md)
