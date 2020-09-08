---
title: 'Tutorial: Conmutación por error del dispositivo físico StorSimple en el mismo dispositivo'
description: Conozca los pasos necesarios para conmutar por error un dispositivo físico de la serie 8000 de StorSimple en sí mismo en caso de desastre.
services: storsimple
documentationcenter: ''
author: alkohli
manager: timlt
editor: ''
ms.assetid: ''
ms.service: storsimple
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 06/23/2017
ms.author: alkohli
ms.openlocfilehash: 17c116194aa52a82246bcee9114824e8a918ebbd
ms.sourcegitcommit: faeabfc2fffc33be7de6e1e93271ae214099517f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/13/2020
ms.locfileid: "88184336"
---
# <a name="fail-over-your-storsimple-physical-device-to-same-device"></a>Conmutación por error del dispositivo físico StorSimple en el mismo dispositivo

## <a name="overview"></a>Información general

En este tutorial se describen los pasos necesarios para conmutar por error un dispositivo físico de la serie StorSimple 8000 en sí mismo en caso de desastre. StorSimple usa la característica de conmutación por error de dispositivos para migrar los datos desde un dispositivo físico de origen en el centro de datos a otro dispositivo físico. Las instrucciones de este tutorial se aplican a los dispositivos físicos de la serie StorSimple 8000 que ejecutan las versiones de software Update 3 y posteriores.

Para aprender más sobre la conmutación por error de dispositivos y cómo se usa para recuperarse de un desastre, vaya a [Conmutación por error y recuperación ante desastres para dispositivos de la serie StorSimple 8000](storsimple-8000-device-failover-disaster-recovery.md).

Para conmutar por error un dispositivo físico a otro dispositivo físico, vaya a [Fail over to the same StorSimple physical device](storsimple-8000-device-failover-physical-device.md) (Conmutación por error al mismo dispositivo físico StorSimple). Para conmutar por error un dispositivo físico StorSimple a un dispositivo StorSimple Cloud Appliance, vaya a [Fail over to a StorSimple Cloud Appliance](storsimple-8000-device-failover-cloud-appliance.md) (Conmutación por error a un dispositivo StorSimple Cloud Appliance).


## <a name="prerequisites"></a>Prerrequisitos

- Asegúrese de haber revisado las consideraciones sobre la conmutación por error de dispositivos. Para más información, vaya a [Consideraciones comunes para la conmutación por error de dispositivo](storsimple-8000-device-failover-disaster-recovery.md).


## <a name="steps-to-fail-over-to-the-same-device"></a>Pasos para conmutar por error al mismo dispositivo

Realice los pasos siguientes si necesita conmutar por error al mismo dispositivo.

1. Tome instantáneas en la nube de todos los volúmenes del dispositivo. Para más información, vaya a [Use StorSimple Device Manager service to create backups](storsimple-8000-manage-backup-policies-u2.md) (Uso del servicio StorSimple Device Manager para crear copias de seguridad).
2. Restablezca el dispositivo a los valores predeterminados de fábrica. Siga las instrucciones detalladas de [Cómo restablecer un dispositivo StorSimple a los valores predeterminados de fábrica](storsimple-8000-manage-device-controller.md#reset-the-device-to-factory-default-settings).
3. Vaya al servicio StorSimple Device Manager y, después, haga clic en **Dispositivos**. En la hoja **Dispositivos**, el dispositivo antiguo debe aparecer como **Sin conexión**.

    ![Dispositivo de origen sin conexión](./media/storsimple-8000-device-failover-disaster-recovery/failover-single-dev2.png)

4. Configure el dispositivo y vuelva a registrarlo con el servicio StorSimple Device Manager. El dispositivo recién registrado debe aparecer como **Listo para configurar**. El nombre del nuevo dispositivo es el mismo que el del dispositivo antiguo, pero se le ha anexado un número para indicar que el dispositivo se ha restablecido a sus valores predeterminados de fábrica y se ha registrado de nuevo.

    ![Dispositivo recién registrado y listo para configurar](./media/storsimple-8000-device-failover-disaster-recovery/failover-single-dev3.png)
5. Para el nuevo dispositivo, complete la configuración del mismo. Para más información, vaya a [Paso 4: Completar el programa de instalación mínima del dispositivo](storsimple-8000-deployment-walkthrough-u2.md#step-4-complete-minimum-device-setup). En la hoja **Dispositivos**, el estado del dispositivo cambia a **En línea**.

   > [!IMPORTANT]
   > **Complete la configuración mínima primero, o se puede producir un error en la recuperación ante desastres**.

    ![Dispositivo recién registrado en línea](./media/storsimple-8000-device-failover-disaster-recovery/failover-single-dev7.png)

6. Seleccione el dispositivo antiguo (con el estado sin conexión) y, en la barra de comandos, haga clic en **Conmutación por error**. En la hoja **Conmutación por error**, seleccione el dispositivo antiguo como origen y especifique el dispositivo de destino como el dispositivo recién registrado.

    ![Resumen de la conmutación por error](./media/storsimple-8000-device-failover-disaster-recovery/failover-single-dev11.png)

    Para obtener instrucciones detalladas, consulte Conmutación por error a otro dispositivo físico.

7. Se crea un trabajo de restauración de dispositivo que puede supervisar desde la hoja **Trabajos**.

8. Después de que el trabajo se haya completado correctamente, acceda al dispositivo nuevo y navegue hasta la hoja **Contenedores de volúmenes**. Compruebe que todos los contenedores de volúmenes del dispositivo antiguo se han migrado al nuevo.

   ![Contenedores de volúmenes migrados](./media/storsimple-8000-device-failover-disaster-recovery/failover-single-dev13.png)

9. Una vez completada la conmutación por error, puede desactivar y eliminar el dispositivo antiguo del portal. Seleccione el dispositivo antiguo (sin conexión), haga clic con el botón derecho y, a continuación, seleccione **Desactivar**. Una vez desactivado, se actualiza el estado del dispositivo.

     ![Dispositivo de origen desactivado](./media/storsimple-8000-device-failover-disaster-recovery/failover-single-dev14.png)

10. Seleccione el dispositivo desactivado, haga clic con el botón derecho y, a continuación, seleccione **Eliminar**. Con ello, eliminará el dispositivo de la lista de dispositivos.

    ![Dispositivo de origen eliminado](./media/storsimple-8000-device-failover-disaster-recovery/failover-single-dev15.png)



## <a name="next-steps"></a>Pasos siguientes

* Después de haber realizado una conmutación por error, puede que necesite [desactivar o eliminar el dispositivo StorSimple](storsimple-8000-deactivate-and-delete-device.md).
* Para más información sobre cómo usar el servicio StorSimple Device Manager, vaya a [Use the StorSimple Device Manager service to administer your StorSimple device](storsimple-8000-manager-service-administration.md) (Uso del servicio StorSimple Device Manager para administrar el dispositivo StorSimple).

