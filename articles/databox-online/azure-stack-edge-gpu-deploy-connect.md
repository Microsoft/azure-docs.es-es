---
title: 'Tutorial: Conexión, configuración y activación de un dispositivo de Azure Stack Edge Pro con GPU en Azure Portal | Microsoft Docs'
description: Obtenga información sobre cómo puede conectarse al dispositivo de Azure Stack Edge con una GPU incorporada mediante la interfaz de usuario web local.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: tutorial
ms.date: 08/29/2020
ms.author: alkohli
Customer intent: As an IT admin, I need to understand how to connect and activate Azure Stack Edge Pro so I can use it to transfer data to Azure.
ms.openlocfilehash: f069f711ccf5672041ea6f3b15a52fa2b3ba2022
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90900041"
---
# <a name="tutorial-connect-to-azure-stack-edge-pro-with-gpu"></a>Tutorial: Conexión a Azure Stack Edge Pro con GPU

En este tutorial se describe cómo puede conectarse al dispositivo de Azure Stack Edge Pro con una GPU incorporada mediante la interfaz de usuario web local.

El proceso de conexión puede tardar unos 5 minutos en completarse.

En este tutorial, obtendrá información sobre lo siguiente:

> [!div class="checklist"]
>
> * Requisitos previos
> * A conectarse a un dispositivo físico


## <a name="prerequisites"></a>Requisitos previos

Antes de configurar e instalar el dispositivo de Azure Stack Edge Pro con GPU, asegúrese de que:

* Ha instalado el dispositivo físico como se detalla en [Instalación de Azure Stack Edge Pro](azure-stack-edge-gpu-deploy-install.md).


## <a name="connect-to-the-local-web-ui-setup"></a>Conexión a la configuración de la interfaz de usuario web local

1. Configure el adaptador Ethernet en el equipo para conectarse al dispositivo de Azure Stack Edge Pro con la dirección IP estática 192.168.100.5 y la subred 255.255.255.0.

2. Conecte el equipo a PUERTO 1 en el dispositivo. Si conecta el equipo directamente al dispositivo (sin un conmutador), use un cable cruzado o un adaptador Ethernet USB. Use la siguiente ilustración para identificar el PUERTO 1 en el dispositivo.

    ![Placa posterior de un dispositivo cableado](./media/azure-stack-edge-gpu-deploy-install/ase-two-pci-slots.png)

    El backplane del dispositivo puede ser ligeramente diferente en función del modelo exacto que haya recibido. Para más información, consulte [Cableado del dispositivo](azure-stack-edge-gpu-deploy-install.md#cable-the-device).


3. Abra una ventana del explorador y acceda a la interfaz de usuario web local del dispositivo en `https://192.168.100.10`.  
    Esta acción puede tardar unos minutos en completarse después de que haya activado el dispositivo.

    Ve un error o advertencia que indica que hay un problema con el certificado de seguridad del sitio web. 
   
    ![Mensaje de error del certificado de seguridad del sitio web](./media/azure-stack-edge-deploy-connect-setup-activate/image2.png)

4. Seleccione **Continue to this webpage**(Continuar a esta página web).  
    Estos pasos pueden variar en función del explorador que se use.

5. Inicie sesión en la interfaz de usuario web del dispositivo. La contraseña predeterminada es *Password1*. 
   
    ![Página de inicio de sesión del dispositivo de Azure Stack Edge Pro](./media/azure-stack-edge-deploy-connect-setup-activate/image3.png)

6. En el aviso, cambie la contraseña del administrador de dispositivos.  
    Recuerde que la nueva contraseña debe tener entre 8 y 16 caracteres. Esta debe contener tres de los siguientes caracteres: mayúsculas, minúsculas, números y caracteres especiales.

Ahora está en la página **Información general** del dispositivo. El siguiente paso es configurar los valores de red del dispositivo.


## <a name="next-steps"></a>Pasos siguientes

En este tutorial, aprendió sobre lo siguiente:

> [!div class="checklist"]
> * Requisitos previos
> * A conectarse a un dispositivo físico


Para información sobre cómo configurar los valores de red del dispositivo de Azure Stack Edge Pro, consulte:

> [!div class="nextstepaction"]
> [Configuración de la red](./azure-stack-edge-gpu-deploy-configure-network-compute-web-proxy.md)
