---
title: Creación de imágenes de máquina virtual para el dispositivo Azure Stack Edge con GPU
description: Describe cómo crear imágenes de máquinas virtuales Linux o Windows para usarlas con el dispositivo Azure Stack Edge con GPU.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: how-to
ms.date: 08/28/2020
ms.author: alkohli
ms.openlocfilehash: 83332c3bfa0b2b99d7333fa679fb8d398aecf8bd
ms.sourcegitcommit: bcda98171d6e81795e723e525f81e6235f044e52
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89268917"
---
# <a name="create-custom-vm-images-for-your-azure-stack-edge-device"></a>Creación de imágenes personalizadas de máquina virtual para el dispositivo Azure Stack Edge

<!--[!INCLUDE [applies-to-skus](../../includes/azure-stack-edge-applies-to-all-sku.md)]-->

Para implementar máquinas virtuales en el dispositivo Azure Stack Edge, debe ser capaz de crear imágenes personalizadas de máquina virtual que puede usar para crear máquinas virtuales. En este artículo se describen los pasos necesarios para crear imágenes personalizadas de máquinas virtuales Linux o Windows que se pueden usar para implementar máquinas virtuales en el dispositivo Azure Stack Edge.

## <a name="vm-image-workflow"></a>Flujo de trabajo de la imagen de máquina virtual

El flujo de trabajo requiere la creación de una máquina virtual en Azure, la personalización de la máquina virtual, la generalización y la descarga del VHD correspondiente a esa máquina virtual. Este VHD generalizado se carga en Azure Stack Edge, se crea el disco administrado a partir de ese VHD, se crea la imagen a partir del disco administrado y, por último, las máquinas virtuales se crean a partir de esa imagen.   

Para más información, vaya a [Implementación de una máquina virtual en el dispositivo Azure Stack Edge mediante Azure PowerShell](azure-stack-edge-j-series-deploy-virtual-machine-powershell.md).


## <a name="create-a-windows-custom-vm-image"></a>Creación de una imagen de máquina virtual personalizada con Windows

Realice los pasos siguientes para crear una imagen de máquina virtual Windows.

1. Cree una máquina virtual Windows. Para más información, vaya a [Tutorial: Creación y administración de máquinas virtuales Windows con Azure PowerShell](../virtual-machines/windows/tutorial-manage-vm.md).

2. Descargue un disco de sistema operativo existente.

    - Siga los pasos descritos en [Descargar un disco duro virtual](../virtual-machines/windows/download-vhd.md).

    - Use el comando `sysprep` siguiente en lugar de lo que se describe en el procedimiento anterior.
    
        `c:\windows\system32\sysprep\sysprep.exe /oobe /generalize /shutdown /mode:vm`
   
       También puede consultar [Introducción a Sysprep (preparación del sistema)](https://docs.microsoft.com/windows-hardware/manufacture/desktop/sysprep--system-preparation--overview).

Use este VHD para crear e implementar una máquina virtual en el dispositivo Azure Stack Edge.

## <a name="create-a-linux-custom-vm-image"></a>Crear una imagen de máquina virtual personalizada con Linux

Realice los pasos siguientes para crear una imagen de máquina virtual Linux.

1. Creación de una máquina virtual Linux. Para más información, vaya a [Tutorial: Creación y administración de máquinas virtuales Linux con la CLI de Azure](../virtual-machines/linux/tutorial-manage-vm.md).

2. [Descargue un disco de sistema operativo existente](../virtual-machines/linux/download-vhd.md).

Use este VHD para crear e implementar una máquina virtual en el dispositivo Azure Stack Edge. Puede usar las dos imágenes de Azure Marketplace siguientes para crear imágenes personalizadas de Linux:

|Nombre del elemento  |Descripción  |Publicador  |
|---------|---------|---------|
|[Ubuntu Server](https://azuremarketplace.microsoft.com/marketplace/apps/canonical.ubuntuserver) |Ubuntu Server es el sistema operativo Linux más conocido para los entornos de nube.|Canonical|
|[Debian 8 "Jessie"](https://azuremarketplace.microsoft.com/marketplace/apps/credativ.debian) |Debian GNU/Linux es una de las distribuciones de Linux más populares.     |credativ|

Para obtener una lista completa de las imágenes de Azure Marketplace que podrían funcionar (actualmente no probadas), vaya a [Elementos de Azure Marketplace disponibles para Azure Stack Hub](https://docs.microsoft.com/azure-stack/operator/azure-stack-marketplace-azure-items?view=azs-1910).


## <a name="next-steps"></a>Pasos siguientes

[Implementación de máquinas virtuales en el dispositivo Azure Stack Edge](azure-stack-edge-j-series-deploy-virtual-machine-powershell.md).
