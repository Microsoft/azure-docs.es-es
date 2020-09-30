---
title: No se puede establecer conexión de forma remota con Azure Virtual Machines porque el adaptador de red está deshabilitado | Microsoft Docs
description: Obtenga información sobre cómo solucionar un problema en el que se produce un error con RDP porque la NIC está deshabilitada en VM de Azure | Microsoft Docs
services: virtual-machines-windows
documentationCenter: ''
author: genlin
manager: dcscontentpm
editor: ''
ms.service: virtual-machines-windows
ms.topic: troubleshooting
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure
ms.date: 11/12/2018
ms.author: genli
ms.openlocfilehash: 0e79efc9de43fc0a3044e9ae1e3959f63bb6e69f
ms.sourcegitcommit: 07166a1ff8bd23f5e1c49d4fd12badbca5ebd19c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/15/2020
ms.locfileid: "90090279"
---
#  <a name="cannot-remote-desktop-to-a-vm-because-the-network-interface-is-disabled"></a>No se puede establecer el escritorio remoto en una VM porque la interfaz de red está deshabilitada

En este artículo se muestra cómo resolver un problema en el que no se puede establecer la conexión del escritorio remoto en Azure Windows Virtual Machines (VM) si la interfaz de red está deshabilitada.


## <a name="symptoms"></a>Síntomas

No puede establecer ninguna conexión RDP ni ningún otro tipo de conexión con ningún otro puerto en una VM en Azure porque la interfaz de red de la VM está deshabilitada.

![Captura de pantalla que muestra una máquina virtual en la que la interfaz de red está desconectada.](./media/troubleshoot-rdp-nic-disabled/disconnected.png)

![Captura de pantalla que muestra una máquina virtual en la que la interfaz de red está deshabilitada.](./media/troubleshoot-rdp-nic-disabled/disabled.png)


## <a name="solution"></a>Solución

Antes de seguir estos pasos, tome una instantánea del disco del sistema operativo de la máquina virtual afectada como copia de seguridad. Para obtener más información, consulte [Instantánea de un disco](../windows/snapshot-copy-managed-disk.md).

Para habilitar la interfaz para la VM, use el control de serie o [restablezca la interfaz de red](#reset-network-interface) para la VM.

### <a name="use-serial-control"></a>Uso del control serie

1. Conéctese a una [consola serie y abra la instancia de CMD](./serial-console-windows.md#use-cmd-or-powershell-in-serial-console). Si la consola serie no está habilitada en la VM, consulte el artículo sobre cómo [restablecer la interfaz de red](#reset-network-interface).
2. Compruebe el estado de la interfaz de red:

    ```console
    netsh interface show interface
    ```

    Anótese el nombre de la interfaz de red deshabilitada.

3. Habilite la interfaz de red:

    ```console
    netsh interface set interface name="interface Name" admin=enabled
    ```

    Por ejemplo, si la interfaz de red se llama "Ethernet 2", ejecute el siguiente comando:

    ```console
    netsh interface set interface name="Ethernet 2" admin=enabled
    ```

4.  Vuelva a comprobar el estado de la interfaz de red para asegurarse de que está habilitada.

    ```console
    netsh interface show interface
    ```

    No tiene que reiniciar la máquina virtual en este momento. La máquina virtual volverá a ser accesible.

5.  Conéctese a la máquina virtual y compruebe si el problema se resuelve.

## <a name="reset-network-interface"></a>Restablecimiento de la interfaz de red

Para restablecer la interfaz de red, cambie la dirección IP a otra dirección IP que esté disponible en la subred. Para ello, use Azure Portal o Azure PowerShell. Para obtener más información, consulte el artículo sobre cómo [restablecer la interfaz de red](reset-network-interface.md).
