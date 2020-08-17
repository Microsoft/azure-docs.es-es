---
title: Características y extensiones de las máquinas virtuales de Azure
description: Más información sobre las extensiones de VM de Azure
services: virtual-machines
author: amjads1
ms.service: virtual-machines
ms.topic: article
ms.workload: infrastructure-services
ms.date: 08/03/2020
ms.author: amjads
ms.openlocfilehash: 73738f339bea3e8e075530e5de564c1c0854d283
ms.sourcegitcommit: 1b2d1755b2bf85f97b27e8fbec2ffc2fcd345120
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/04/2020
ms.locfileid: "87552057"
---
# <a name="azure-virtual-machine-extensions-and-features"></a>Características y extensiones de las máquinas virtuales de Azure
Las extensiones son aplicaciones pequeñas que proporcionan automatización y configuración posterior a la implementación en VM de Azure. La plataforma de Azure hospeda numerosas extensiones, que abarcan aplicaciones de configuración, supervisión, seguridad y utilidad de VM. Los editores toman una aplicación, la encapsulan en una extensión y simplifican la instalación. Solo debe proporcionar parámetros obligatorios. 

## <a name="how-can-i-find-what-extensions-are-available"></a>¿Cómo puedo encontrar qué extensiones están disponibles?
Para ver las extensiones disponibles, seleccione una VM y, a continuación, seleccione **Extensiones** en el menú de la izquierda. Para extraer una lista completa de extensiones, consulte [Detección de extensiones de VM para Linux](features-linux.md) y [Detección de extensiones de VM para Windows](features-windows.md).

## <a name="how-can-i-install-an-extension"></a>¿Cómo puedo instalar una extensión?
Las extensiones de VM de Azure se pueden administrar con la CLI de Azure, PowerShell, plantillas de Resource Manager y Azure Portal. Para probar una extensión, vaya a Azure Portal, seleccione la extensión de script personalizado y pase un comando o script para ejecutar la extensión.

Para obtener más información, consulte [Extensión de script personalizado de Windows](custom-script-windows.md) y [Extensión de script personalizado de Linux](custom-script-linux.md).

## <a name="how-do-i-manage-extension-application-lifecycle"></a>¿Cómo administro el ciclo de vida de las aplicaciones de extensión?
No es necesario conectarse directamente a una VM para instalar o eliminar una extensión. El ciclo de vida de la extensión de Azure se administra fuera de la VM y se integra en la plataforma de Azure.

## <a name="anything-else-i-should-be-thinking-about-for-extensions"></a>¿Debo tener en cuenta algo más sobre las extensiones?
Algunas aplicaciones de extensión de máquina virtual individuales pueden tener sus propios requisitos previos de entorno, como el acceso a un punto de conexión. Cada extensión tiene un artículo en el que se explican los requisitos previos, incluidos los sistemas operativos compatibles.

## <a name="troubleshoot-extensions"></a>Extensiones de solución de problemas

Puede encontrar información de solución de problemas para cada extensión en la sección **Solución de problemas y soporte técnico** de la información general de la extensión. A continuación se enumera la información de solución de problemas disponible:

| Espacio de nombres | Solución de problemas |
|-----------|-----------------|
| microsoft.azure.monitoring.dependencyagent.dependencyagentlinux | [Dependencia de Azure Monitor para Linux](agent-dependency-linux.md#troubleshoot-and-support) |
| microsoft.azure.monitoring.dependencyagent.dependencyagentwindows | [Dependencia de Azure Monitor para Windows](agent-dependency-windows.md#troubleshoot-and-support) |
| microsoft.azure.security.azurediskencryptionforlinux | [Azure Disk Encryption para Linux](azure-disk-enc-linux.md#troubleshoot-and-support) |
| microsoft.azure.security.azurediskencryption | [Azure Disk Encryption para Windows](azure-disk-enc-windows.md#troubleshoot-and-support) |
| microsoft.compute.customscriptextension | [Script personalizado para Windows](custom-script-windows.md#troubleshoot-and-support) |
| microsoft.ostcextensions.customscriptforlinux | [ para Linux](dsc-linux.md#troubleshoot-and-support) |
| microsoft.powershell.dsc | [Desired State Configuration para Windows](dsc-windows.md#troubleshoot-and-support) |
| microsoft.hpccompute.nvidiagpudriverlinux | [Extensión del controlador de GPU de NVIDIA para Linux](hpccompute-gpu-linux.md#troubleshoot-and-support) |
| microsoft.hpccompute.nvidiagpudriverwindows | [Extensión del controlador de GPU de NVIDIA para Windows](hpccompute-gpu-windows.md#troubleshoot-and-support) |
| microsoft.azure.security.iaasantimalware | [Extensión de Antimalware para Windows](iaas-antimalware-windows.md#troubleshoot-and-support) |
| microsoft.enterprisecloud.monitoring.omsagentforlinux | [Azure Monitor para Linux](oms-linux.md#troubleshoot-and-support)
| microsoft.enterprisecloud.monitoring.microsoftmonitoringagent | [Azure Monitor para Windows](oms-windows.md#troubleshoot-and-support) |
| stackify.linuxagent.extension.stackifylinuxagentextension | [Stackify Retrace para Linux](stackify-retrace-linux.md#troubleshoot-and-support) |
| vmaccessforlinux.microsoft.ostcextensions | [Restablecimiento de contraseña para Linux](vmaccess.md#troubleshoot-and-support) |
| microsoft.recoveryservices.vmsnapshot | [Instantánea para Linux](vmsnapshot-linux.md#troubleshoot-and-support) |
| microsoft.recoveryservices.vmsnapshot | [Instantánea para Windows](vmsnapshot-windows.md#troubleshoot-and-support) |


## <a name="next-steps"></a>Pasos siguientes
* Para obtener más información sobre cómo funcionan las extensiones y el agente de Linux, vea [Características y extensiones de VM de Azure para Linux](features-linux.md).
* Para más información sobre cómo funcionan las extensiones de Windows y Windows Guest Agent, consulte [Características y extensiones de VM de Azure para Windows](features-windows.md).  
* Para instalar Windows Guest Agent, vea [Información general del agente de máquina virtual de Azure](agent-windows.md).  
* Para instalar el agente de Linux, vea [Información y uso del agente de Linux de Azure](agent-linux.md).  

