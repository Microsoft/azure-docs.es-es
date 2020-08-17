---
title: Descarga de un VHD de Linux desde Azure
description: Descarga de un VHD de Linux mediante la CLI de Azure y Azure Portal.
author: cynthn
ms.service: virtual-machines-linux
ms.topic: how-to
ms.date: 08/03/2020
ms.author: cynthn
ms.openlocfilehash: 897cae53e589f4058e5499c0e6e941d4f1d9bb2f
ms.sourcegitcommit: 5a37753456bc2e152c3cb765b90dc7815c27a0a8
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/04/2020
ms.locfileid: "87761077"
---
# <a name="download-a-linux-vhd-from-azure"></a>Descarga de un VHD de Linux desde Azure

En este artículo aprenderá a descargar un archivo de disco duro virtual (VHD) de Linux desde Azure mediante Azure Portal. 

## <a name="stop-the-vm"></a>Parada de la máquina virtual

No se puede descargar un VHD desde Azure si está conectado a una máquina virtual en ejecución. Debe detener la máquina virtual para descargar el VHD. 

1.  Inicie sesión en [Azure Portal](https://portal.azure.com/).
2.  En el menú de la izquierda, seleccione **Máquinas virtuales**.
3.  Seleccione la máquina virtual en la lista.
4.  En la página de la máquina virtual, seleccione **Detener**.

    :::image type="content" source="./media/download-vhd/export-stop.PNG" alt-text="Muestra el botón de menú para detener la máquina virtual.":::

## <a name="generate-sas-url"></a>Generación de dirección URL de SAS

Para descargar el archivo de VHD, debe generar una dirección URL de [firma de acceso compartido (SAS)](../../storage/common/storage-sas-overview.md?toc=/azure/virtual-machines/windows/toc.json). Una vez generada la dirección URL, se asigna un tiempo de expiración a la dirección URL.

1. En el menú de la página de la máquina virtual, seleccione **Discos**.
2. Seleccione el disco de sistema operativo de la máquina virtual y luego seleccione **Exportación del disco**.
1. Si es necesario, actualice el valor de **La dirección URL expira en (segundos)** para proporcionar suficiente tiempo para completar la descarga. El valor predeterminado es 3600 segundos (una hora).
3. Seleccione **Generar dirección URL**.
 
      
## <a name="download-vhd"></a>Descarga de VHD

1.  En la dirección URL generada, seleccione **Descargar el archivo VHD**.

    :::image type="content" source="./media/download-vhd/export-download.PNG" alt-text="Muestra el botón para descargar el VHD.":::

2.  Es posible que tenga que seleccionar **Guardar** en el explorador para iniciar la descarga. El nombre predeterminado del archivo de VHD es *abcd*.

## <a name="next-steps"></a>Pasos siguientes

- Obtenga información sobre cómo [cargar y crear una máquina virtual de Linux desde un disco personalizado con la CLI de Azure](upload-vhd.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json). 
- [Administración de discos de Azure con la CLI de Azure](tutorial-manage-disks.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json).
