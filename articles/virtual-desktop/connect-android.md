---
title: 'Conexión a Windows Virtual Desktop desde Android: Azure'
description: Cómo conectarse a Windows Virtual Desktop mediante el cliente Android.
author: Heidilohr
ms.topic: how-to
ms.date: 03/25/2020
ms.author: helohr
manager: lizross
ms.openlocfilehash: ebf91f68e45148d9a609ff671ffa4683bd74c82c
ms.sourcegitcommit: d68c72e120bdd610bb6304dad503d3ea89a1f0f7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89226121"
---
# <a name="connect-to-windows-virtual-desktop-with-the-android-client"></a>Conexión a Windows Virtual Desktop con el cliente Android

> Se aplica a: Android 4.1 y posteriores, Chromebooks con ChromeOS 53 y posteriores.

>[!IMPORTANT]
>Este contenido se aplica a Windows Virtual Desktop con objetos de Windows Virtual Desktop de Azure Resource Manager. Si usa Windows Virtual Desktop (clásico) sin objetos de Azure Resource Manager, consulte [este artículo](./virtual-desktop-fall-2019/connect-android-2019.md).

Puede acceder a los recursos de Windows Virtual Desktop desde su dispositivo Android con nuestro cliente, que se puede descargar. El cliente Android también se puede usar en dispositivos Chromebook que admitan Google Play Store. En esta guía se le indicará cómo configurar el cliente Android.

## <a name="install-the-android-client"></a>Instalación del cliente Android

Para empezar, [descargue](https://play.google.com/store/apps/details?id=com.microsoft.rdc.androidx) e instale el cliente en su dispositivo Android.

## <a name="subscribe-to-a-feed"></a>Suscripción a una fuente

Suscríbase a la fuente que le proporcionó su administrador para obtener la lista de recursos administrados a los que puede acceder en el dispositivo Android.

Para suscribirse a una fuente:

1. En el centro de conexiones, pulse **+** y, después, pulse **Remote Resource Feed** (Fuente de recursos remotos).
2. Escriba la dirección URL de la fuente en el campo **Feed URL** (Dirección URL de la fuente). La dirección URL de la fuente puede ser una dirección URL o una dirección de correo electrónico.
   - Si usa una dirección URL, use la que le proporcionó el administrador, que lo habitual es que sea <https://rdweb.wvd.microsoft.com/api/arm/feeddiscovery>.
   - Para usar el correo electrónico, escriba su dirección de correo electrónico. El cliente buscará una dirección URL asociada a su dirección de correo electrónico si el administrador configuró el servidor de esa forma.
   - Para conectarse mediante el portal de US Gov, utilice <https://rdweb.wvd.azure.us/api/arm/feeddiscovery>.
3. Pulse **NEXT** (Siguiente).
4. Escriba sus credenciales cuando se le solicite.
   - En **User Name** (Nombre de usuario), asigne al nombre de usuario permiso para acceder a los recursos.
   - En **Password** (Contraseña), escriba la contraseña asociada al nombre de usuario.
   - También se le puede solicitar que proporcione otra información si el administrador configuró la autenticación de esa forma.

Después de suscribirse, el centro de conexiones debería mostrar los recursos remotos.

Una vez que se haya suscrito a una fuente, el contenido de la misma se actualizará automáticamente de forma periódica. Pueden agregarse, cambiarse o quitarse recursos de acuerdo con los cambios hechos por el administrador.

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre cómo usar el cliente Android, consulte [Introducción al cliente de Android](/windows-server/remote/remote-desktop-services/clients/remote-desktop-android/).
