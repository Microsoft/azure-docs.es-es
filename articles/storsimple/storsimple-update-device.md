---
title: Actualización del dispositivo StorSimple | Microsoft Docs
description: Se explica cómo utilizar la característica de actualización de StorSimple para instalar actualizaciones y revisiones, tanto normales como en modo de mantenimiento.
services: storsimple
documentationcenter: NA
author: alkohli
manager: carmonm
editor: ''
ms.assetid: 786059f5-2a38-4105-941d-0860ce4ac515
ms.service: storsimple
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: TBD
ms.date: 01/23/2018
ms.author: alkohli
ms.openlocfilehash: dec354767875d91c6062cdfbb0009e857dbe2636
ms.sourcegitcommit: 814778c54b59169c5899199aeaa59158ab67cf44
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/13/2020
ms.locfileid: "90056558"
---
# <a name="update-your-storsimple-8000-series-device"></a>Actualización del dispositivo de la serie StorSimple 8000
> [!NOTE]
> El portal clásico para StorSimple está en desuso. Los administradores de dispositivos StorSimple realizarán la transición automáticamente al nuevo Azure Portal según la programación de puesta en desuso. Recibirá un correo electrónico y una notificación del portal en los que se avisa de este paso. Este documento también se retirará pronto. Si tiene alguna pregunta sobre este paso, consulte las [preguntas frecuentes de la migración a Azure Portal](storsimple-8000-move-azure-portal-faq.md).

## <a name="overview"></a>Información general
Las características de actualización de StorSimple permiten mantener fácilmente actualizado el dispositivo de StorSimple. Según el tipo de actualización, puede aplicar las actualizaciones al dispositivo a través del Portal de Azure clásico o mediante la interfaz de Windows PowerShell. En este tutorial se describen los tipos de actualización y cómo instalar cada uno de ellos.

Puede aplicar dos tipos de actualizaciones de dispositivo: 

* Actualizaciones normales (o en modo normal)
* Actualizaciones en modo de mantenimiento

Puede instalar las actualizaciones normales a través del Portal de Azure clásico o de Windows PowerShell; sin embargo, debe usar Windows PowerShell para instalar las actualizaciones en modo de mantenimiento. 

A continuación se describe cada tipo de actualización.

### <a name="regular-updates"></a>Actualizaciones normales
Se trata de actualizaciones no perturbadoras que se pueden instalar cuando el dispositivo está en modo normal. Estas actualizaciones se aplican a través del sitio web Microsoft Update a cada controlador de dispositivo. 

> [!IMPORTANT]
> Durante el proceso de actualización puede producirse una conmutación por error del controlador. Sin embargo, esto no afectará a la disponibilidad ni al funcionamiento del sistema.
> 
> 

* Para más información acerca de cómo instalar actualizaciones normales a través del Portal de Azure clásico, consulte [Instalar actualizaciones normales a través del Portal de Azure clásico](#install-regular-updates-via-the-azure-classic-portal).
* También puede instalar actualizaciones normales a través de Windows PowerShell para StorSimple. Para obtener más información, consulte [Instalación de actualizaciones normales a través de Windows PowerShell para StorSimple](#install-regular-updates-via-windows-powershell-for-storsimple).

### <a name="maintenance-mode-updates"></a>Actualizaciones en modo de mantenimiento
Se trata de actualizaciones con interrupciones, como las actualizaciones de firmware de disco. En estas actualizaciones es necesario que el dispositivo esté en modo de mantenimiento. Para obtener más información, consulte [Paso 2: Acceso al modo de mantenimiento](#step2). No puede usar el Portal de Azure clásico para instalar las actualizaciones en modo de mantenimiento. En su lugar, debe usar Windows PowerShell para StorSimple. 

Para obtener más información sobre cómo instalar las actualizaciones en modo de mantenimiento, consulte [Instalación de actualizaciones en modo de mantenimiento a través de Windows PowerShell para StorSimple](#install-maintenance-mode-updates-via-windows-powershell-for-storsimple).

> [!IMPORTANT]
> Las actualizaciones en modo de mantenimiento deben aplicarse por separado en cada controlador. 
> 
> 

## <a name="install-regular-updates-via-the-azure-classic-portal"></a>Instalar actualizaciones normales a través del Portal de Azure clásico
Puede usar el Portal de Azure clásico para aplicar las actualizaciones en el dispositivo de StorSimple.

[!INCLUDE [storsimple-install-updates-manually](../../includes/storsimple-install-updates-manually.md)]

## <a name="install-regular-updates-via-windows-powershell-for-storsimple"></a>Instalación de actualizaciones normales a través de Windows PowerShell para StorSimple
También puede usar Windows PowerShell para StorSimple para aplicar las actualizaciones normales (en modo normal).

> [!IMPORTANT]
> Aunque puede instalar actualizaciones normales con Windows PowerShell para StorSimple, le recomendamos encarecidamente que las instale a través del Portal de Azure clásico. A partir de la actualización 1, se realizarán comprobaciones previas antes de instalar actualizaciones desde el Portal. Estas comprobaciones previas previenen los errores y garantizan una experiencia con menos complicaciones. 
> 
> 

[!INCLUDE [storsimple-install-regular-updates-powershell](../../includes/storsimple-install-regular-updates-powershell.md)]

## <a name="install-maintenance-mode-updates-via-windows-powershell-for-storsimple"></a>Instalación de actualizaciones en modo de mantenimiento a través de Windows PowerShell para StorSimple
Para aplicar las actualizaciones en modo de mantenimiento en el dispositivo de StorSimple se usa Windows PowerShell para StorSimple. En este modo, todas las solicitudes de E/S se interrumpen. También se detienen servicios como la memoria de acceso aleatorio no volátil (NVRAM) o el servicio de Cluster Server. Al entrar o salir de este modo, se reinician ambos controladores. Al salir de este modo, los servicios se reanudan y deben estar correctos. Esto puede tardar unos minutos.

Si debe aplicar actualizaciones en modo de mantenimiento, recibirá una alerta a través del Portal de Azure clásico que le indicará que existen actualizaciones que se deben instalar. Esta alerta incluye instrucciones para usar Windows PowerShell para StorSimple con el fin de instalar las actualizaciones. Después de actualizar el dispositivo, utilice el mismo procedimiento para cambiarlo a modo normal. Para obtener instrucciones detalladas, consulte [Paso 4: Salir del modo de mantenimiento](#step4).

> [!IMPORTANT]
> * Antes de entrar en modo de mantenimiento, compruebe que ambos controladores del dispositivo funcionen correctamente. Para ello, vaya a **Estado del hardware** en la página **Mantenimiento** del Portal de Azure clásico. Si el controlador no es correcto, póngase en contacto con el servicio de soporte técnico de Microsoft para conocer los pasos siguientes. Para obtener más información, póngase en contacto con el servicio de soporte técnico de Microsoft. 
> * Si está en modo de mantenimiento, debe aplicar primero la actualización en un controlador y, a continuación, en el otro.
> 
> 

### <a name="step-1-connect-to-the-serial-console"></a>Paso 1: Conexión a la consola serie <a name="step1"></a>
En primer lugar, utilice una aplicación como PuTTY para tener acceso a la consola serie. El siguiente procedimiento explica cómo usar PuTTY para conectarse a la consola serie.

[!INCLUDE [storsimple-use-putty](../../includes/storsimple-use-putty.md)]

### <a name="step-2-enter-maintenance-mode"></a>Paso 2: Acceso al modo de mantenimiento <a name="step2"></a>
Una vez esté conectado a la consola, mire a ver si hay actualizaciones que se deban instalar y active el modo de mantenimiento para hacerlo.

[!INCLUDE [storsimple-enter-maintenance-mode](../../includes/storsimple-enter-maintenance-mode.md)]

### <a name="step-3-install-your-updates"></a>Paso 3: Instalación de las actualizaciones <a name="step3"></a>
A continuación, instale las actualizaciones.

[!INCLUDE [storsimple-install-maintenance-mode-updates](../../includes/storsimple-install-maintenance-mode-updates.md)]

### <a name="step-4-exit-maintenance-mode"></a>Paso 4: Salir del modo de mantenimiento <a name="step4"></a>
Por último, salga del modo de mantenimiento.

[!INCLUDE [storsimple-exit-maintenance-mode](../../includes/storsimple-exit-maintenance-mode.md)]

## <a name="install-hotfixes-via-windows-powershell-for-storsimple"></a>Instale las reparaciones mediante Windows PowerShell para StorSimple
A diferencia de las actualizaciones para Microsoft Azure StorSimple, las revisiones se instalan desde una carpeta compartida. Al igual que en el caso de las actualizaciones, hay dos tipos de revisiones: 

* Revisiones normales 
* Revisiones en modo de mantenimiento  

Los siguientes procedimientos explican cómo usar Windows PowerShell para StorSimple con el fin de instalar revisiones normales y en modo de mantenimiento.

[!INCLUDE [storsimple-install-regular-hotfixes](../../includes/storsimple-install-regular-hotfixes.md)]

[!INCLUDE [storsimple-install-maintenance-mode-hotfixes](../../includes/storsimple-install-maintenance-mode-hotfixes.md)]

## <a name="what-happens-to-updates-if-you-perform-a-factory-reset-of-the-device"></a>¿Qué ocurre con las actualizaciones si se realiza un restablecimiento de fábrica del dispositivo?
Si se restablece la configuración de fábrica de un dispositivo, se pierden todas las actualizaciones. Después de registrar y configurar el dispositivo en el que se ha efectuado un restablecimiento de fábrica, deberá instalar manualmente las actualizaciones a través del Portal de Azure clásico o de Windows PowerShell para StorSimple. Para obtener más información acerca de los restablecimientos de fábrica, consulte [Restablecer el dispositivo a los valores predeterminados de fábrica](storsimple-8000-manage-device-controller.md#reset-the-device-to-factory-default-settings).

## <a name="next-steps"></a>Pasos siguientes
* Obtenga más información sobre el [uso de Windows PowerShell para StorSimple para administrar el dispositivo StorSimple](storsimple-windows-powershell-administration.md).
* Obtenga más información sobre el [uso del servicio StorSimple Manager para administrar su dispositivo StorSimple](storsimple-manager-service-administration.md).

