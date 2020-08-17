---
title: Solución de problemas de implementación de máquinas virtuales Linux | Microsoft Docs
description: Solución de problemas de implementación de Resource Manager cuando crea una nueva máquina virtual de Linux en Azure
services: virtual-machines-linux, azure-resource-manager
documentationcenter: ''
author: DavidCBerry13
manager: gwallace
editor: ''
tags: top-support-issue, azure-resource-manager
ms.assetid: 906a9c89-6866-496b-b4a4-f07fb39f990c
ms.service: virtual-machines-linux
ms.workload: na
ms.tgt_pltfrm: vm-linux
ms.topic: troubleshooting
ms.date: 09/09/2016
ms.author: daberry
ms.openlocfilehash: f48963a4d18e80cb67bfbbdc532d34f89b8b5d8a
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87028454"
---
# <a name="troubleshoot-resource-manager-deployment-issues-with-creating-a-new-linux-virtual-machine-in-azure"></a>Solución de problemas de la implementación de Resource Manager con la creación de una nueva máquina virtual de Linux en Azure
[!INCLUDE [virtual-machines-troubleshoot-deployment-new-vm-opening](../../../includes/virtual-machines-troubleshoot-deployment-new-vm-opening-include.md)]

[!INCLUDE [support-disclaimer](../../../includes/support-disclaimer.md)]

## <a name="top-issues"></a>Principales problemas
[!INCLUDE [support-disclaimer](../../../includes/virtual-machines-linux-troubleshoot-deploy-vm-top.md)]

Para consultar otros problemas de implementación de máquinas virtuales y preguntas, vea [Troubleshoot deploying Linux virtual machine issues in Azure](troubleshoot-deploy-vm-linux.md) (Solución de problemas de implementación de máquinas virtuales de Linux en Azure).

## <a name="collect-activity-logs"></a>Recopilación de registros de actividad
Para iniciar la solución de problemas, recopile los registros de actividad para identificar el error asociado con el problema. Los vínculos siguientes contienen información detallada sobre el proceso que se debe seguir.

[Ver operaciones de implementación](../../azure-resource-manager/templates/deployment-history.md)

[Ver registros de actividad para administrar recursos de Azure](../../azure-resource-manager/management/view-activity-logs.md)

[!INCLUDE [virtual-machines-troubleshoot-deployment-new-vm-issue1](../../../includes/virtual-machines-troubleshoot-deployment-new-vm-issue1-include.md)]

[!INCLUDE [virtual-machines-linux-troubleshoot-deployment-new-vm-table](../../../includes/virtual-machines-linux-troubleshoot-deployment-new-vm-table.md)]

**Y:** si el sistema operativo es Linux generalizado y se carga o captura con la configuración generalizada, no habrá errores. De forma similar, si el sistema operativo es Linux especializado y se carga o captura con la configuración especializada, no habrá errores.

**Errores de carga:**

**N<sup>1</sup>:** si el sistema operativo es Linux generalizado y se carga como especializado, recibirá un error de tiempo de espera de aprovisionamiento porque la VM se atascará en la fase de aprovisionamiento.

**N<sup>2</sup>:** si el sistema operativo es Linux especializado y se carga como generalizado, recibirá un error de aprovisionamiento porque la nueva VM se ejecuta con el nombre del equipo, el nombre de usuario y la contraseña originales.

**Resolución:**

Para resolver estos errores, cargue el VHD original, disponible en el entorno local, con la misma configuración que para el sistema operativo (generalizado o especializado). Para cargarlo como generalizado, no olvide ejecutarlo o desaprovisionarlo antes.

**Errores de captura:**

**N<sup>3</sup>:** si el sistema operativo es Linux generalizado y se captura como especializado, recibirá un error de tiempo de espera de aprovisionamiento porque la VM original no se puede utilizar, ya que está marcada como generalizada.

**N<sup>4</sup>:** si el sistema operativo es Linux especializado y se captura como generalizado, recibirá un error de aprovisionamiento porque la nueva VM se ejecuta con el nombre del equipo, el nombre de usuario y la contraseña originales. Además, no se puede utilizar la máquina virtual original ya que está marcada como especializada.

**Resolución:**

Para resolver estos errores, elimine la imagen actual del portal y [vuelva a capturarla desde los discos duros virtuales actuales](../linux/capture-image.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) con la misma configuración que para el sistema operativo (generalizada o especializada).

## <a name="issue-custom-gallery-marketplace-image-allocation-failure"></a>Problema: Imagen de galería/marketplace/personalizada; error de asignación
Este error se produce en situaciones en las que la nueva solicitud de máquina virtual está anclada en un clúster que no admite el tamaño de la máquina virtual que se solicita o no tiene espacio libre disponible para alojar la solicitud.

**Causa 1:** El clúster no admite el tamaño de VM solicitado.

**Solución 1:**

* Vuelva a intentar la solicitud con un tamaño de máquina virtual menor.
* Si no se puede cambiar el tamaño de la máquina virtual solicitada:
  * Detenga todas las máquinas virtuales en el conjunto de disponibilidad.
    Haga clic en **Grupos de recursos** > *su grupo de recursos* > **Recursos** > *su conjunto de disponibilidad* > **Virtual Machines** > *su máquina virtual* > **Detener**.
  * Después de detener todas las máquinas virtuales, cree la nueva máquina virtual con el tamaño deseado.
  * Inicie la nueva máquina virtual en primer lugar y luego seleccione cada una de las máquinas virtuales detenidas y haga clic en **Iniciar**.

**Causa 2:** El clúster no tiene recursos disponibles.

**Solución 2:**

* Vuelva a intentar la solicitud más tarde.
* Si la nueva máquina virtual puede formar parte de un conjunto de disponibilidad diferente
  * Cree una nueva máquina virtual en un conjunto de disponibilidad diferente (en la misma región).
  * Agregue la nueva máquina virtual a la misma red virtual.

## <a name="next-steps"></a>Pasos siguientes
Si tiene problemas al iniciar una máquina virtual Linux detenida o al cambiar el tamaño de una máquina virtual Linux existente en Azure, consulte [Solución de problemas de la implementación de Resource Manager con el reinicio o el cambio de tamaño de una máquina virtual de Linux existente en Azure](./troubleshoot-deploy-vm-linux.md?toc=/azure/virtual-machines/linux/toc.json).
