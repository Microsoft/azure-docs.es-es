---
title: 'Solución de problemas: Windows no pudo acabar de configurar el sistema'
titlesuffix: Azure Virtual Machines
description: En este artículo se proporcionan los pasos para resolver problemas en los que el proceso Sysprep impide el arranque de una máquina virtual de Azure.
services: virtual-machines-windows, azure-resource-manager
documentationcenter: ''
author: v-miegge
manager: dcscontentpm
editor: ''
tags: azure-resource-manager
ms.assetid: 5fc57a8f-5a0c-4b5f-beef-75eecb4d310d
ms.service: virtual-machines-windows
ms.workload: na
ms.tgt_pltfrm: vm-windows
ms.topic: troubleshooting
ms.date: 09/09/2020
ms.author: v-miegge
ms.openlocfilehash: 9c170607d6300c4921285e85ac78db5a8a18ad9b
ms.sourcegitcommit: 1fe5127fb5c3f43761f479078251242ae5688386
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/14/2020
ms.locfileid: "90078790"
---
# <a name="troubleshoot-windows-could-not-finish-configuring-the-system"></a>Solución de problemas: Windows no pudo acabar de configurar el sistema

En este artículo se proporcionan los pasos para resolver problemas en los que el proceso Sysprep impide el arranque de una máquina virtual (VM) de Azure.

## <a name="symptom"></a>Síntoma

Si usa [Diagnósticos de arranque](https://docs.microsoft.com/azure/virtual-machines/troubleshooting/boot-diagnostics) para ver la captura de pantalla de la VM, verá que se muestra un error de instalación de Windows mientras la configuración de Windows inicia los servicios. El error mostrará el mensaje:

`Windows could not finish configuring the system. To attempt to resume configuration, restart the computer. Setup is starting services`

  ![En la figura 1 se muestra un error de instalación de Windows con el mensaje: "Windows no pudo finalizar la configuración del sistema. Para intentar reanudar la configuración, reinicie el equipo. El programa de instalación está iniciando los servicios".](./media/windows-could-not-configure-system/1-windows-error-configure.png)

## <a name="cause"></a>Causa

Este error se produce cuando el sistema operativo (SO) no puede completar el [proceso Sysprep](https://docs.microsoft.com/windows-hardware/manufacture/desktop/sysprep-process-overview). Este error se producirá cuando se intente un arranque inicial de una máquina virtual generalizada. Si se produce este problema, vuelva a crear la imagen generalizada, ya que la imagen se encuentra en un estado no implementable y no se puede recuperar.

## <a name="solution"></a>Solución

Para corregir este problema, siga las [instrucciones de Azure para preparar o capturar una imagen](https://docs.microsoft.com/azure/virtual-machines/windows/upload-generalized-managed) y prepare una nueva imagen generalizada.
