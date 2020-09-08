---
title: VHD no se admite al crear una máquina virtual en Azure | Microsoft Docs
description: Este artículo ayuda a corregir los errores de VHD al ejecutar una máquina virtual en Microsoft Azure.
services: virtual-machines
documentationCenter: ''
author: v-miegge
manager: dcscontentpm
editor: ''
ms.service: virtual-machines
ms.assetid: 5488aba9-c3da-435d-b4a5-63470f455b07
ms.topic: troubleshooting
ms.tgt_pltfrm: na
ms.workload: infrastructure
ms.date: 06/29/2020
ms.author: genli
ms.openlocfilehash: 23f31e8d36d6ba615b21a4043c7d39716fc6d89a
ms.sourcegitcommit: 271601d3eeeb9422e36353d32d57bd6e331f4d7b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88653501"
---
# <a name="vhd-is-not-supported-when-you-create-a-virtual-machine-in-azure"></a>VHD no se admite al crear una máquina virtual en Azure

Este artículo ayuda a corregir los errores de VHD al ejecutar una máquina virtual en Windows o Linux.

## <a name="symptoms"></a>Síntomas

Cuando se crea una máquina virtual en Microsoft Azure mediante un VHD cargado, se produce un error en la implementación y se devuelve el mensaje de error siguiente: 

```
New-AzureRmVM : Long running operation failed with status 'Failed'.
ErrorCode: InvalidVhd
ErrorMessage: The specified cookie value in VHD footer indicates that disk 'diskname' with blob https://xxxxxx.blob.core.windows.net/vhds/samplename.vhd is not a supported VHD. Disk is expected to have cookie value 'conectix'.
```

## <a name="cause"></a>Causa

Este problema se produce por uno de los siguientes motivos:

- El disco duro virtual no cumple con la alineación de 1 MB (desplazamiento). El tamaño de disco admitido debe ser 1 MB*N. Por ejemplo, el disco debe ser de 102 401 MB.
- El disco duro virtual está dañado o no se admite. 

## <a name="resolution"></a>Resolución

> [!NOTE]
> Para realizar la siguiente corrección, el cliente tendrá que realizar estos pasos antes de cargar el disco duro virtual en Azure.

Para resolver este problema, cambie el tamaño del disco para que sea compatible con la alineación de 1 MB:

- Para resolver el problema en Windows, use el [cmdlet de PowerShell Resize-VHD](/powershell/module/hyper-v/resize-vhd). Tenga en cuenta que **Resize-VHD** no es un cmdlet de Azure PowerShell.

  1. [Instalación del rol de Hyper-V en Windows Server](/windows-server/virtualization/hyper-v/get-started/install-the-hyper-v-role-on-windows-server)
  1. [Conversión de un disco virtual en un VHD de tamaño fijo](../windows/prepare-for-upload-vhd-image.md#convert-the-virtual-disk-to-a-fixed-size-vhd)

- Para resolver el problema en Linux, use el [comando qemu-img](../linux/create-upload-generic.md).

Para más información sobre cómo crear y cargar un disco duro virtual para crear una VM de Azure, consulte los artículos siguientes:

- [Carga y creación de una VM Linux a partir de una imagen de disco personalizada mediante la CLI de Azure 1.0](../linux/upload-vhd.md)
- [Crear y cargar un VHD de Windows Server a Azure](../windows/upload-generalized-managed.md)

La persistencia de los problemas puede indicar que un disco duro virtual está dañado. En esta situación, se recomienda volver a crear el disco duro virtual desde cero.

Para obtener más información, consulte el artículo siguiente:

- [Acerca de VHD](../managed-disks-overview.md)
