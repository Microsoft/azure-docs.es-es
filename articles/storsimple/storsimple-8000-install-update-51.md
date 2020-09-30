---
title: Instalación de Update 5.1 en un dispositivo de la serie StorSimple 8000 | Microsoft Docs
description: Explica cómo instalar Update 5.1 de la serie StorSimple 8000 en un dispositivo de la serie StorSimple 8000.
services: storsimple
documentationcenter: NA
author: alkohli
ms.assetid: ''
ms.service: storsimple
ms.devlang: NA
ms.topic: how-to
ms.tgt_pltfrm: NA
ms.workload: TBD
ms.date: 03/05/2020
ms.author: alkohli
ms.openlocfilehash: 1c9d4747b227c7ff83e4c43292badd65836edfef
ms.sourcegitcommit: 814778c54b59169c5899199aeaa59158ab67cf44
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/13/2020
ms.locfileid: "90052178"
---
# <a name="install-update-51-on-your-storsimple-device"></a>Instalación de Update 5.1 en un dispositivo StorSimple

## <a name="overview"></a>Información general

En este tutorial se explica cómo instalar Update 5.1 en un dispositivo de StorSimple que ejecuta una versión anterior del software mediante Azure Portal. <!--The hotfix method is used when you are trying to install Update 5.1 on a device running pre-Update 3 versions. The hotfix method is also used when a gateway is configured on a network interface other than DATA 0 of the StorSimple device and you are trying to update from a pre-Update 1 software version.-->

Update 5.1 incluye actualizaciones de seguridad sin interrupciones. Las actualizaciones normales o que no provocan interrupciones se pueden aplicar a través de Azure Portal <!--or by the hotfix method-->.

> [!IMPORTANT]
>
> * Update 5.1 es una actualización obligatoria y debe instalarse inmediatamente. Para obtener más información, consulte [Notas de la versión de Update 5.1](storsimple-update51-release-notes.md).
> * En esta actualización se incluye un conjunto de comprobaciones previas que se hace antes de la instalación para determinar el estado del dispositivo en cuanto a la conectividad de red y el estado del hardware. Estas comprobaciones previas solo se realizan si aplica las actualizaciones desde Azure Portal.
> * Si desea instalar mediante el método de revisión, póngase en contacto con [Soporte técnico de Microsoft](mailto:support@microsoft.com).

<!--
> * We strongly recommend that when updating a device running versions prior to Update 3, you install the updates using hotfix method. If you encounter any issues, [log a support ticket](storsimple-8000-contact-microsoft-support.md). 


> * We recommend that you install the software and other regular updates via the Azure portal. You should only go to the Windows PowerShell interface of the device (to install updates) if the pre-update gateway check fails in the portal. Depending upon the version you are updating from, the updates may take 4 hours (or greater) to install. The maintenance mode updates must be installed through the Windows PowerShell interface of the device. As maintenance mode updates are disruptive updates, these result in a down time for your device.


> * If running the optional StorSimple Snapshot Manager, ensure that you have upgraded your Snapshot Manager version to Update 5.1 prior to updating the device.
-->

[!INCLUDE [storsimple-preparing-for-update](../../includes/storsimple-preparing-for-updates.md)]

## <a name="install-update-51-through-the-azure-portal"></a>Instalación de Update 5.1 mediante Azure Portal

Siga los pasos a continuación para actualizar el dispositivo a [Update 5.1](storsimple-update51-release-notes.md).

> [!NOTE]
> Microsoft extrae información de diagnóstico adicional del dispositivo. Como consecuencia, cuando nuestro equipo de operaciones identifica dispositivos que están teniendo problemas, estamos mejor equipados para recopilar información del dispositivo y diagnosticar problemas.

#### <a name="to-install-an-update-from-the-azure-portal"></a>Instalar una actualización desde el Portal de Azure

1. En la página de servicio de StorSimple, seleccione el dispositivo.

    ![Selección del dispositivo](./media/storsimple-8000-install-update-51/update1.png)

2. Vaya a **Configuración del dispositivo** > **Actualizaciones del dispositivo**.

    ![Haga clic en Actualizaciones del dispositivo.](./media/storsimple-8000-install-update-51/update2.png)

3. Aparecerá una notificación si hay nuevas actualizaciones disponibles. O bien, en la hoja **Actualizaciones del dispositivo**, haga clic en **Buscar actualizaciones**. Se crea un trabajo para buscar las actualizaciones disponibles. Se le notificará cuando el trabajo se complete correctamente.

    ![Haga clic en Actualizaciones del dispositivo.](./media/storsimple-8000-install-update-51/update3.png)

4. Se recomienda revisar las notas de la versión antes de aplicar una actualización en el dispositivo. Para aplicar las actualizaciones, haga clic en **Instalar actualizaciones**. En la hoja **Confirmar actualizaciones normales**, revise los requisitos previos que debe completar antes de aplicar las actualizaciones. Seleccione la casilla para indicar que está listo para actualizar el dispositivo y, a continuación, haga clic en **Instalar**.

    ![Haga clic en Actualizaciones del dispositivo.](./media/storsimple-8000-install-update-51/update4.png)

5. Ahora se inicia un conjunto de comprobaciones previas. Estas comprobaciones incluyen:
   
   * **Comprobaciones del estado del controlador** para comprobar que los controladores del dispositivo están en buen estado y en línea.
   * **Comprobaciones de mantenimiento de componentes de hardware** para comprobar que todos los componentes de hardware del dispositivo StorSimple están en buen estado.
   * **Comprobaciones de DATA 0** para comprobar DATA 0 está habilitado en el dispositivo. Si la interfaz no está habilitada, tendrá que habilitarla y, después, volver a intentarlo.

     La actualización se descargará e instalará solo si todas las comprobaciones se completan correctamente. Recibirá una notificación cuando las comprobaciones están en curso. Si se produce un error en la comprobación previa, se le indicarán las causas del error. Resuelva estos problemas e intente realizar de nuevo la operación. Puede que necesite ponerse en contacto con el soporte técnico de Microsoft si no puede resolver estos problemas por sí mismo.

7. Una vez realizadas correctamente las comprobaciones previas, se crea un trabajo de actualización. Recibirá una notificación cuando el trabajo de actualización esté correctamente creado.
   
    ![Creación del trabajo de actualización](./media/storsimple-8000-install-update-51/update6.png)
   
    La actualización se aplica en el dispositivo.

9. La actualización tarda unas horas en completarse. Seleccione el trabajo de actualización y haga clic en **Detalles** para ver los detalles del trabajo en cualquier momento.

    ![Creación del trabajo de actualización](./media/storsimple-8000-install-update-51/update8.png)

     También puede supervisar el progreso del trabajo de actualización desde **Configuración del dispositivo > Trabajos**. En la hoja **Trabajos**, puede ver el progreso de la actualización.

     ![Creación del trabajo de actualización](./media/storsimple-8000-install-update-51/update7.png)

10. Una vez completado el trabajo, vaya a **Configuración del dispositivo > Actualizaciones del dispositivo**. La versión del software ya debería estar actualizada.


Verifique que el dispositivo ejecute **StorSimple 8000 Series Update 5.1 (6.3.9600.17885)** . También se debe modificar la **fecha de última actualización**.
<!-- 5.1 - KB 4542887-->

<!--You will now see that the Maintenance mode updates are available (this message might continue to be displayed for up to 24 hours after you install the updates). The steps to install maintenance mode update are detailed in the next section.

[!INCLUDE [storsimple-8000-install-maintenance-mode-updates](../../includes/storsimple-8000-install-maintenance-mode-updates.md)]

## Install Update 5.1 as a hotfix

The software versions that can be upgraded using the hotfix method are:

* Update 0.1, 0.2, 0.3
* Update 1, 1.1, 1.2
* Update 2, 2.1, 2.2
* Update 3, 3.1
* Update 4
* Update 5

> [!NOTE]
> The recommended method to install Update 5.1 is through the Azure portal when trying to update from Update 3 and later version. When updating a device running versions prior to Update 3, use this procedure. You can also use this procedure if you fail the gateway check when trying to install the updates through the Azure portal. The check fails when you have a gateway assigned to a non-DATA 0 network interface and your device is running a software version earlier than Update 1.

The hotfix method involves the following three steps:

1. Download the hotfixes from the Microsoft Update Catalog.
2. Install and verify the regular mode hotfixes.
3. Install and verify the maintenance mode hotfix.

#### Download updates for your device

You must download and install the following hotfixes in the prescribed order and the suggested folders:

| Order | KB | Description | Update type | Install time |Install in folder|
| --- | --- | --- | --- | --- | --- |
| 1. |KB4037264 |Software update<br> Download both _HcsSoftwareUpdate.exe_ and _CisMSDAgent.exe_ |Regular <br></br>Non-disruptive |~ 25 mins |FirstOrderUpdate|

If updating from a device running Update 4, you only need to install the OS cumulative updates as second order updates.

| Order | KB | Description | Update type | Install time |Install in folder|
| --- | --- | --- | --- | --- | --- |
| 2A. |KB4025336 |OS cumulative updates package <br> Download Windows Server 2012 R2 version |Regular <br></br>Non-disruptive |- |SecondOrderUpdate|

If installing from a device running Update 3 or earlier, install the following in addition to the cumulative updates.

| Order | KB | Description | Update type | Install time |Install in folder|
| --- | --- | --- | --- | --- | --- |
| 2B. |KB4011841 <br> KB4011842 |LSI driver and firmware updates <br> USM firmware update (version 3.38) |Regular <br></br>Non-disruptive |~ 3 hrs <br> (includes 2A. + 2B. + 2C.)|SecondOrderUpdate|
| 2C. |KB3139398 <br> KB3142030 <br> KB3108381 <br> KB3153704 <br> KB3174644 <br> KB3139914   |OS security updates package <br> Download Windows Server 2012 R2 version |Regular <br></br>Non-disruptive |- |SecondOrderUpdate|
| 2D. |KB3146621 <br> KB3103616 <br> KB3121261 <br> KB3123538 |OS updates package <br> Download Windows Server 2012 R2 version |Regular <br></br>Non-disruptive |- |SecondOrderUpdate|



You may also need to install disk firmware updates on top of all the updates shown in the preceding tables. You can verify whether you need the disk firmware updates by running the `Get-HcsFirmwareVersion` cmdlet. If you are running these firmware versions: `XMGJ`, `XGEG`, `KZ50`, `F6C2`, `VR08`, `N003`, `0107`, then you do not need to install these updates.

| Order | KB | Description | Update type | Install time | Install in folder|
| --- | --- | --- | --- | --- | --- |
| 3. |KB4037263 |Disk firmware |Maintenance <br></br>Disruptive |~ 30 mins | ThirdOrderUpdate |

<br></br>

> [!IMPORTANT]
> * If updating from Update 4, the total install time is close to 4 hours.
> * Before using this procedure to apply the update, make sure that both the device controllers are online and all the hardware components are healthy.

Perform the following steps to download and install the hotfixes.

[!INCLUDE [storsimple-install-update5-hotfix](../../includes/storsimple-install-update5-hotfix.md)]
-->
<!--
[!INCLUDE [storsimple-8000-install-troubleshooting](../../includes/storsimple-8000-install-troubleshooting.md)]
-->

## <a name="next-steps"></a>Pasos siguientes

Más información sobre la [versión de Update 5.1](storsimple-update51-release-notes.md).
