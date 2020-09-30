---
title: Creación de imágenes de máquina virtual para el dispositivo Azure Stack Edge Pro con GPU
description: En este artículo se explica cómo crear imágenes de máquinas virtuales Linux o Windows para usarlas con el dispositivo Azure Stack Edge Pro con GPU.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: how-to
ms.date: 09/04/2020
ms.author: alkohli
ms.openlocfilehash: 745d0df07b6d0d01acf0d564df8c242e16f3f56d
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90890986"
---
# <a name="create-custom-vm-images-for-your-azure-stack-edge-pro-device"></a>Creación de imágenes personalizadas de máquina virtual para el dispositivo Azure Stack Edge Pro

<!--[!INCLUDE [applies-to-skus](../../includes/azure-stack-edge-applies-to-all-sku.md)]-->

Para implementar máquinas virtuales en el dispositivo Azure Stack Edge Pro, debe ser capaz de crear imágenes personalizadas de máquina virtual que puede usar para crear máquinas virtuales. En este artículo se describen los pasos necesarios para crear imágenes personalizadas de máquinas virtuales Linux o Windows que se pueden usar para implementar máquinas virtuales en el dispositivo Azure Stack Edge Pro.

## <a name="vm-image-workflow"></a>Flujo de trabajo de la imagen de máquina virtual

El flujo de trabajo requiere la creación de una máquina virtual en Azure, la personalización de la máquina virtual, la generalización y la descarga del VHD correspondiente a esa máquina virtual. Este VHD generalizado se carga en Azure Stack Edge Pro, se crea el disco administrado a partir de ese VHD, se crea la imagen a partir del disco administrado y, por último, las máquinas virtuales se crean a partir de esa imagen.   

Para más información, vaya a [Implementación de una máquina virtual en el dispositivo Azure Stack Edge Pro con Azure PowerShell](azure-stack-edge-j-series-deploy-virtual-machine-powershell.md).


## <a name="create-a-windows-custom-vm-image"></a>Creación de una imagen de máquina virtual personalizada con Windows

Realice los pasos siguientes para crear una imagen de máquina virtual Windows.

1. Cree una máquina virtual Windows. Para más información, vaya a [Tutorial: Creación y administración de máquinas virtuales Windows con Azure PowerShell](../virtual-machines/windows/tutorial-manage-vm.md).

2. Descargue un disco de sistema operativo existente.

    - Siga los pasos descritos en [Descargar un disco duro virtual](../virtual-machines/windows/download-vhd.md).

    - Use el comando `sysprep` siguiente en lugar de lo que se describe en el procedimiento anterior.
    
        `c:\windows\system32\sysprep\sysprep.exe /oobe /generalize /shutdown /mode:vm`
   
       También puede consultar [Introducción a Sysprep (preparación del sistema)](https://docs.microsoft.com/windows-hardware/manufacture/desktop/sysprep--system-preparation--overview).

Use este VHD para crear e implementar una máquina virtual en el dispositivo Azure Stack Edge Pro.

## <a name="create-a-linux-custom-vm-image"></a>Crear una imagen de máquina virtual personalizada con Linux

Realice los pasos siguientes para crear una imagen de máquina virtual Linux.

1. Creación de una máquina virtual Linux. Para más información, vaya a [Tutorial: Creación y administración de máquinas virtuales Linux con la CLI de Azure](../virtual-machines/linux/tutorial-manage-vm.md).

1. Desaprovisione la máquina virtual. Use el agente de máquina virtual de Azure para eliminar archivos y datos específicos de la máquina. Use el comando `waagent` con el parámetro `-deprovision+user` en la máquina virtual Linux de origen. Para obtener más información, consulte [Información y uso del agente de Linux de Azure](../virtual-machines/extensions/agent-linux.md).

    1. Conéctese a la máquina virtual Linux con un cliente de SSH.
    2. En la ventana de SSH, escriba el siguiente comando:
       
        ```bash
        sudo waagent -deprovision+user
        ```
       > [!NOTE]
       > Solo puede ejecutar este comando en una máquina virtual que quiera capturar como imagen. Este comando no garantiza que se haya borrado toda información confidencial de la imagen o que sea adecuada para su redistribución. El parámetro `+user` también elimina la última cuenta de usuario aprovisionada. Para mantener las credenciales de la cuenta de usuario en la máquina virtual, use solo `-deprovision`.
     
    3. Escriba **s** para continuar. Puede agregar el parámetro `-force` para evitar este paso de confirmación.
    4. Una vez finalizado el comando, escriba **exit** para cerrar el cliente de SSH.  La máquina virtual seguirá ejecutándose en este momento.


1. [Descargue un disco de sistema operativo existente](../virtual-machines/linux/download-vhd.md).

Use este VHD para crear e implementar una máquina virtual en el dispositivo Azure Stack Edge Pro. Puede usar las dos imágenes de Azure Marketplace siguientes para crear imágenes personalizadas de Linux:

|Nombre del elemento  |Descripción  |Publicador  |
|---------|---------|---------|
|[Ubuntu Server](https://azuremarketplace.microsoft.com/marketplace/apps/canonical.ubuntuserver) |Ubuntu Server es el sistema operativo Linux más conocido para los entornos de nube.|Canonical|
|[Debian 8 "Jessie"](https://azuremarketplace.microsoft.com/marketplace/apps/credativ.debian) |Debian GNU/Linux es una de las distribuciones de Linux más populares.     |credativ|

Para obtener una lista completa de imágenes de Azure Marketplace que podrían funcionar (aún no sometidas a prueba), vaya a [Elementos de Azure Marketplace disponibles para Azure Stack Hub](https://docs.microsoft.com/azure-stack/operator/azure-stack-marketplace-azure-items?view=azs-1910).


## <a name="next-steps"></a>Pasos siguientes

[Implementación de máquinas virtuales en un dispositivo Azure Stack Edge Pro](azure-stack-edge-j-series-deploy-virtual-machine-powershell.md)
