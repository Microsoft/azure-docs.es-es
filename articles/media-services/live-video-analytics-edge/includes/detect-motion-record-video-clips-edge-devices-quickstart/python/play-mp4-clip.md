---
ms.openlocfilehash: f4a1cc432a50a555fe6e050ca318b4cfaf1092d4
ms.sourcegitcommit: 56cbd6d97cb52e61ceb6d3894abe1977713354d9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88682365"
---
Los archivos MP4 se escriben en un directorio del dispositivo perimetral que configuró en el archivo *.env* mediante la clave OUTPUT_VIDEO_FOLDER_ON_DEVICE. Si ha usado el valor predeterminado, los resultados deberían estar en la carpeta */var/media/* .

Para reproducir el clip de MP4:

1. Vaya al grupo de recursos, busque la máquina virtual y conéctese mediante Azure Bastion.

    ![Resource group](../../../media/quickstarts/resource-group.png)
    
    ![máquina virtual](../../../media/quickstarts/virtual-machine.png)
1. Inicie sesión con las credenciales que se generaron al [configurar los recursos de Azure](../../../detect-motion-emit-events-quickstart.md#set-up-azure-resources). 
1. En el símbolo del sistema, vaya al directorio correspondiente. La ubicación predeterminada es */var/media*. Debería ver los archivos MP4 en el directorio.

    ![Output](../../../media/quickstarts/samples-output.png) 

1. Use [Secure Copy (SCP)](../../../../../virtual-machines/linux/copy-files-to-linux-vm-using-scp.md) para copiar los archivos en la máquina local. 
1. Reproduzca los archivos mediante [VLC Media Player](https://www.videolan.org/vlc/) o cualquier otro reproductor MP4.
