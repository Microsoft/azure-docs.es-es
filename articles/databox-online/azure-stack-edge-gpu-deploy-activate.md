---
title: 'Tutorial: Activación de un dispositivo de Azure Stack Edge Pro con GPU en Azure Portal | Microsoft Docs'
description: En este tutorial para implementar Azure Stack Edge Pro con GPU, se indica cómo activar el dispositivo físico.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: tutorial
ms.date: 09/10/2020
ms.author: alkohli
Customer intent: As an IT admin, I need to understand how to activate Azure Stack Edge Pro so I can use it to transfer data to Azure.
ms.openlocfilehash: 15680a4f8228af95e6643478c9262653171912ca
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90903500"
---
# <a name="tutorial-activate-azure-stack-edge-pro-with-gpu"></a>Tutorial: Activación de Azure Stack Edge Pro con GPU

En este tutorial se describe cómo puede activar el dispositivo de Azure Stack Edge Pro con una GPU incorporada mediante la interfaz de usuario web local.

El proceso de activación puede tardar unos 5 minutos en completarse.

En este tutorial, aprendió sobre lo siguiente:

> [!div class="checklist"]
> * Requisitos previos
> * Activación del dispositivo físico

## <a name="prerequisites"></a>Requisitos previos

Antes de configurar e instalar el dispositivo de Azure Stack Edge Pro con GPU, asegúrese de que:

* Para el dispositivo físico: 
    
    - Ha instalado el dispositivo físico como se detalla en [Instalación de Azure Stack Edge Pro](azure-stack-edge-gpu-deploy-install.md).
    - Ha configurado la red y los valores de red de proceso tal como se detalla en [Configuración de la red, la red de proceso y el proxy web](azure-stack-edge-gpu-deploy-configure-network-compute-web-proxy.md)
    - Ha cargado sus propios certificados de dispositivo, o los ha generado, en el dispositivo si ha cambiado el nombre del dispositivo o el dominio DNS a través de la página **Dispositivo**. Si no ha realizado este paso, verá un error durante la activación del dispositivo y se bloqueará la activación. Para obtener más información, vaya a [Configuración de certificados](azure-stack-edge-gpu-deploy-configure-certificates.md).
    
* Tiene la clave de activación del servicio Azure Stack Edge que creó para administrar el dispositivo de Azure Stack Edge Pro. Para más información, vaya a [Preparación de la implementación de Azure Stack Edge Pro](azure-stack-edge-gpu-deploy-prep.md).


## <a name="activate-the-device"></a>Activación del dispositivo

1. En la interfaz de usuario web local del dispositivo, vaya a la página **Introducción**.
2. En el icono **Activación**, seleccione **Activar**. 

    ![Página "Detalles de la nube" de la interfaz de usuario web local](./media/azure-stack-edge-gpu-deploy-activate/activate-1.png)
    
3. En el panel **Activar**, escriba la **Clave de activación** que obtuvo en [Obtención de la clave de activación para Azure Stack Edge Pro](azure-stack-edge-gpu-deploy-prep.md#get-the-activation-key).

4. Seleccione **Aplicar**.

    ![Página "Detalles de la nube" de la interfaz de usuario web local](./media/azure-stack-edge-gpu-deploy-activate/activate-2.png)


5. En primer lugar, se activa el dispositivo. A continuación, se le pedirá que descargue el archivo de clave.
    
    ![Página "Detalles de la nube" de la interfaz de usuario web local](./media/azure-stack-edge-gpu-deploy-activate/activate-3.png)
    
    Seleccione **Download and continue** (Descargar y continuar) y guarde el archivo *device-serial-no.json* en una ubicación segura fuera del dispositivo. **Este archivo de clave contiene las claves de recuperación del disco del sistema operativo y los discos de datos del dispositivo**. Estas claves pueden ser necesarias para facilitar la recuperación del sistema en el futuro.

    Este es el contenido del archivo *json*:

        
    ```json
    {
      "Id": "<Device ID>",
      "DataVolumeBitLockerExternalKeys": {
        "hcsinternal": "<BitLocker key for data disk>",
        "hcsdata": "<BitLocker key for data disk>"
      },
      "SystemVolumeBitLockerRecoveryKey": "<BitLocker key for system volume>",
      "ServiceEncryptionKey": "<Azure service encryption key>"
    }
    ```
        
 
    En la siguiente tabla se explican las diversas claves:
    
    |Campo  |Descripción  |
    |---------|---------|
    |`Id`    | Es el id. para el dispositivo.        |
    |`DataVolumeBitLockerExternalKeys`|Estas son las claves de BitLocker para los discos de datos y se usan para recuperar los datos locales en el dispositivo.|
    |`SystemVolumeBitLockerRecoveryKey`| Esta es la clave de BitLocker para el volumen del sistema. Esta clave ayuda con la recuperación de la configuración del sistema y de los datos del sistema para el dispositivo. |
    |`ServiceEncryptionKey`| Esta clave protege los datos que fluyen a través del servicio de Azure. Esta clave garantiza que, si la seguridad del servicio Azure se pone en peligro, la información almacenada permanecerá segura. |

6. Vaya a la página **Información general**. El estado del dispositivo debería aparecer como **Activado**.

    ![Página "Detalles de la nube" de la interfaz de usuario web local](./media/azure-stack-edge-gpu-deploy-activate/activate-4.png)
 
La activación del dispositivo se ha completado. Ahora puede agregar recursos compartidos en el dispositivo.


## <a name="next-steps"></a>Pasos siguientes

En este tutorial, aprendió sobre lo siguiente:

> [!div class="checklist"]
> * Requisitos previos
> * Activación del dispositivo físico

Para aprender a transferir datos con Azure Stack Edge Pro, consulte:

> [!div class="nextstepaction"]
> [Transferencia de datos con Azure Stack Edge Pro](./azure-stack-edge-j-series-deploy-add-shares.md)
