---
ms.openlocfilehash: e0c313ea5e5062f725d0a65fefeac444214cf863
ms.sourcegitcommit: 56cbd6d97cb52e61ceb6d3894abe1977713354d9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88682392"
---
A medida que configura los recursos de Azure de este inicio rápido, se copiará un vídeo (corto) de un área de aparcamiento en la máquina virtual Linux en Azure que se usa como dispositivo IoT Edge. Este archivo de vídeo se usará para simular un streaming en vivo para este tutorial.

Abra una aplicación como [VLC Media Player](https://www.videolan.org/vlc/), seleccione Ctrl + N y pegue [este vínculo](https://lvamedia.blob.core.windows.net/public/lots_015.mkv) en el vídeo sobre el área de aparcamiento para iniciar la reproducción. Hacia la marca de 5 segundos, un coche blanco se desplaza por el aparcamiento.

Realice los pasos que se indican a continuación para usar Live Video Analytics en IoT Edge para detectar el movimiento del coche y grabar un clip de vídeo que empiece en torno a la marca de los 5 segundos.
