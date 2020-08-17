---
title: Descargar un VHD de Windows desde Azure
description: Descargue un VHD de Windows mediante Azure Portal.
author: cynthn
manager: gwallace
ms.service: virtual-machines-windows
ms.subservice: disks
ms.workload: infrastructure-services
ms.topic: how-to
ms.date: 01/13/2019
ms.author: cynthn
ms.openlocfilehash: 5c57ac0fb0ab377f00770e3c9c03a413dcb345bc
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87291029"
---
# <a name="download-a-windows-vhd-from-azure"></a>Descargar un VHD de Windows desde Azure

En este artículo aprenderá a descargar un archivo de disco duro virtual (VHD) de Windows desde Azure mediante Azure Portal.

## <a name="optional-generalize-the-vm"></a>Opcional: Generalización de la máquina virtual

Si quiere usar el VHD como una [imagen](tutorial-custom-images.md) para crear otras máquinas virtuales, debe usar [Sysprep](/windows-hardware/manufacture/desktop/sysprep--generalize--a-windows-installation) para generalizar el sistema operativo. 

Para usar el VHD como imagen para crear otras máquinas virtuales, generalice la máquina virtual.

1. Si aún no lo ha hecho, inicie sesión en el [Azure Portal](https://portal.azure.com/).
2. [Conecte a la máquina virtual](connect-logon.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json). 
3. En la máquina virtual, abra la ventana del símbolo del sistema como administrador.
4. Cambie el directorio a *%windir%\system32\sysprep* y ejecute sysprep.exe.
5. En el cuadro de diálogo Herramienta de preparación del sistema, seleccione **Iniciar la configuración rápida (OOBE) del sistema** y asegúrese de que **Generalizar** está activado.
6. En Opciones de apagado, seleccione **Apagar** y luego haga clic en **Aceptar**. 


## <a name="stop-the-vm"></a>Parada de la máquina virtual

No se puede descargar un VHD desde Azure si está conectado a una máquina virtual en ejecución. Debe detener la máquina virtual para descargar un VHD. 

1. En el menú Concentrador de Azure Portal, haga clic en **Máquinas virtuales**.
1. Seleccione la máquina virtual en la lista.
1. En la hoja de la máquina virtual, haga clic en **Detener**.


## <a name="generate-download-url"></a>Generación de la dirección URL de descarga

Para descargar el archivo de VHD, debe generar una dirección URL de [firma de acceso compartido (SAS)](../../storage/common/storage-sas-overview.md?toc=/azure/virtual-machines/windows/toc.json). Una vez generada la dirección URL, se asigna un tiempo de expiración a la dirección URL.

1. En la página de la máquina virtual, haga clic en **Discos** en el menú de la izquierda.
1. Seleccione el disco de sistema operativo de la máquina.
1. En la página del disco, seleccione **Exportación del disco** en el menú de la izquierda.
1. El tiempo de expiración predeterminado de la dirección URL es de *3600* segundos. Auméntelo a **36000** para los discos del sistema operativo Windows.
1. Haga clic en **Generar dirección URL**.

> [!NOTE]
> El tiempo de expiración se aumenta con respecto al valor predeterminado para proporcionar tiempo suficiente para descargar el archivo de VHD grande de un sistema operativo Windows Server. Es de esperar que un archivo de VHD que contenga el sistema operativo Windows Server tarde varias horas en descargarse según la conexión. Si va a descargar un VHD de un disco de datos, el tiempo predeterminado es suficiente. 
> 
> 

## <a name="download-vhd"></a>Descarga de VHD

1. En la dirección URL generada, haga clic en Descargar el archivo VHD.
1. Es posible que tenga que hacer clic en **Guardar** en el explorador para iniciar la descarga. El nombre predeterminado del archivo de VHD es *abcd*.

## <a name="next-steps"></a>Pasos siguientes

- Más información sobre cómo [cargar un archivo de VHD en Azure](upload-generalized-managed.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json). 
- [Create managed disks from unmanaged disks in a storage account](attach-disk-ps.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json) (Creación de discos administrados a partir de discos no administrados en una cuenta de almacenamiento).
- [Manage Azure disks with PowerShell](tutorial-manage-data-disk.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json) (Administrar discos de Azure con PowerShell).
