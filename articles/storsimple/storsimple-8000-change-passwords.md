---
title: Cambio de sus contraseñas de StorSimple | Microsoft Docs
description: Describe cómo usar el servicio StorSimple Device Manager para cambiar sus contraseñas de administrador de dispositivos y de StorSimple Snapshot Manager.
services: storsimple
documentationcenter: NA
author: alkohli
manager: timlt
editor: ''
ms.assetid: ''
ms.service: storsimple
ms.devlang: NA
ms.topic: how-to
ms.tgt_pltfrm: NA
ms.workload: TBD
ms.date: 07/03/2017
ms.author: alkohli
ms.openlocfilehash: 038084ba9ae43e14bc2eb42bf258912be27d062c
ms.sourcegitcommit: 3543d3b4f6c6f496d22ea5f97d8cd2700ac9a481
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/20/2020
ms.locfileid: "86527870"
---
# <a name="use-the-storsimple-device-manager-service-to-change-your-storsimple-passwords"></a>Uso del servicio StorSimple Device Manager para cambiar las contraseñas de StorSimple

## <a name="overview"></a>Información general
La opción **Configuración del dispositivo** de Azure Portal contiene todos los parámetros de dispositivo que puede volver a configurar en un dispositivo StorSimple administrado mediante un servicio StorSimple Device Manager. En este tutorial se explica cómo se puede usar la opción **Seguridad** de **Configuración del dispositivo** para cambiar la contraseña de administrador de dispositivos y de StorSimple Snapshot Manager.

## <a name="change-the-device-administrator-password"></a>Cambio de la contraseña del administrador de dispositivos
Cuando se usa la interfaz de Windows PowerShell para tener acceso al dispositivo StorSimple, se le pedirá que escriba una contraseña de administrador de dispositivos. Cuando el primer dispositivo StorSimple esté registrado con un servicio, la contraseña predeterminada para esta interfaz será *Password1*. Por seguridad de sus datos, deberá cambiar esta contraseña al final del proceso de registro. No se puede salir del proceso de registro sin cambiar esta contraseña. Para más información, consulte [Paso 3: Configurar y registrar el dispositivo a través de Windows PowerShell para StorSimple](storsimple-8000-deployment-walkthrough-u2.md#step-3-configure-and-register-the-device-through-windows-powershell-for-storsimple).

La primera contraseña que se estableció a través de la interfaz de Windows PowerShell durante el registro puede cambiarse después a través de Azure Portal. Realice los pasos siguientes para cambiar la contraseña del administrador de dispositivos.

#### <a name="to-change-the-device-administrator-password"></a>Para cambiar la contraseña del administrador de dispositivos
1. Vaya al servicio Administrador de dispositivos de StorSimple y haga clic en **Dispositivos**.

2. En la lista tabular de dispositivos, seleccione el dispositivo cuya contraseña tiene intención de cambiar y haga clic en él.

    ![Captura de pantalla del servicio Administrador de dispositivos de StorSimple. En administración está seleccionada la opción Dispositivos. En la lista de dispositivos está seleccionado un dispositivo.](./media/storsimple-8000-change-passwords/changepwd1.png)

3. En la hoja **Configuración**, vaya a **Configuración del dispositivo > Seguridad**.

    ![Captura de pantalla de la hoja Configuración del servicio Administrador de dispositivos. En Configuración de dispositivo está seleccionada la opción Seguridad.](./media/storsimple-8000-change-passwords/changepwd2.png)

4. En la hoja **Configuración de seguridad**, haga clic en **Contraseña** para cambiar la contraseña de administrador de dispositivos.

    ![Captura de pantalla de la hoja Configuración de seguridad. Está resaltado el botón Contraseña.](./media/storsimple-8000-change-passwords/changepwd3.png)

5. En la hoja **Contraseña**, especifique una contraseña de administrador que tenga entre 8 y 15 caracteres. La contraseña debe ser una combinación de tres o más caracteres en mayúsculas, minúsculas, números y caracteres especiales.

6. Confirme la contraseña.

    ![Captura de pantalla de la hoja Contraseña. En Contraseña de administrador de dispositivo, los cuadros Nueva contraseña y Confirmar contraseña están rellenos.](./media/storsimple-8000-change-passwords/changepwd4.png)

7. Haga clic en **Guardar** y, cuando se le pida confirmación, haga clic en **Sí**.

    ![Captura de pantalla de la hoja Contraseña. El botón Guardar está resaltado.](./media/storsimple-8000-change-passwords/changepwd6.png)

Ahora debe actualizarse la contraseña del administrador de dispositivos. Puede usar esta contraseña modificada para tener acceso a la interfaz de Windows PowerShell.

## <a name="set-the-storsimple-snapshot-manager-password"></a>Establecimiento de la contraseña de StorSimple Snapshot Manager
El software StorSimple Snapshot Manager reside en el host de Windows y permite a los administradores administrar copias de seguridad del dispositivo StorSimple en forma de instantáneas locales y en la nube.

Al configurar un dispositivo en StorSimple Snapshot Manager, se le pedirá que proporcione la dirección IP y la contraseña del dispositivo para autenticar el dispositivo de almacenamiento.

La contraseña de StorSimple Snapshot Manager se puede establecer o cambiar a través de Azure Portal. Realice los pasos siguientes para establecer o cambiar la contraseña de StorSimple Snapshot Manager.

#### <a name="to-set-the-storsimple-snapshot-manager-password"></a>Para establecer la contraseña de StorSimple Snapshot Manager
1. Vaya al servicio Administrador de dispositivos de StorSimple y haga clic en **Dispositivos**.

2. En la lista tabular de dispositivos, seleccione el dispositivo cuya contraseña de StorSimple Snapshot Manager tiene intención de establecer o cambiar, y haga clic en él.

     ![Captura de pantalla del servicio Administrador de dispositivos de StorSimple. En Administración está seleccionada la opción Dispositivos. En la lista de dispositivos está seleccionado un dispositivo.](./media/storsimple-8000-change-passwords/changepwd1.png)

3. En la hoja **Configuración**, vaya a **Configuración del dispositivo > Seguridad**.

     ![Captura de pantalla de la hoja Configuración del servicio Administrador de dispositivos. En Configuración de dispositivo está seleccionada la opción Seguridad.](./media/storsimple-8000-change-passwords/changepwd2.png)

4. En la hoja **Configuración de seguridad**, haga clic en **Contraseña** para establecer o cambiar la contraseña de StorSimple Snapshot Manage.

     ![Captura de pantalla de la hoja Configuración de seguridad. Está resaltado el botón Contraseña.](./media/storsimple-8000-change-passwords/changepwd3.png) 

5. En la hoja **Contraseña**, escriba una contraseña que tenga 14 o 15 caracteres. Asegúrese de que la contraseña contiene una combinación de tres o más caracteres en mayúsculas, minúsculas, números y caracteres especiales.

6. Confirme la contraseña.

     ![Captura de pantalla de la hoja Contraseña. En Contraseña de Snapshot Manager los cuadros Nueva contraseña y Confirmar contraseña están rellenos.](./media/storsimple-8000-change-passwords/changepwd5.png)

7. Haga clic en **Guardar** y, cuando se le pida confirmación, haga clic en **Sí**.

     ![Captura de pantalla de la hoja Contraseña. El botón Guardar está resaltado.](./media/storsimple-8000-change-passwords/changepwd6.png)

Ahora se debe actualizar la contraseña de StorSimple Snapshot Manager.

## <a name="next-steps"></a>Pasos siguientes
* Obtenga más información acerca de la [Seguridad de StorSimple](storsimple-8000-security.md).
* Obtenga más información sobre la [modificación de la configuración del dispositivo](storsimple-8000-modify-device-config.md).
* Obtenga más información sobre el [uso del servicio Administrador de dispositivos de StorSimple para administrar el dispositivo StorSimple](storsimple-8000-manager-service-administration.md).

