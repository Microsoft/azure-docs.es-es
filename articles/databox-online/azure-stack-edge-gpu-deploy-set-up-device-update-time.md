---
title: 'Tutorial: Conexión, configuración y activación de un dispositivo de Azure Stack Edge Pro con GPU en Azure Portal | Microsoft Docs'
description: En este tutorial para implementar Azure Stack Edge Pro con GPU, se indica cómo conectar, configurar y activar el dispositivo físico.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: tutorial
ms.date: 09/10/2020
ms.author: alkohli
Customer intent: As an IT admin, I need to understand how to connect and activate Azure Stack Edge Pro so I can use it to transfer data to Azure.
ms.openlocfilehash: 1f86b0fc847ade3153c8eaddb0d82bd968913b46
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90899593"
---
# <a name="tutorial-configure-device-settings-for-azure-stack-edge-pro-with-gpu"></a>Tutorial: Configuración de los valores del dispositivo para Azure Stack Edge Pro con GPU

En este tutorial se describe cómo se configuran los valores relacionados con el dispositivo de Azure Stack Edge Pro con una GPU incorporada. Puede configurar el nombre de dispositivo, el servidor de actualización y el servidor horario a través de la interfaz de usuario web local.

La configuración del dispositivo puede tardar aproximadamente de 5 a 7 minutos en completarse.

En este tutorial, obtendrá información sobre lo siguiente:

> [!div class="checklist"]
>
> * Requisitos previos
> * Configuración de las opciones de dispositivo
> * Configuración de la actualización 
> * Configuración de la hora

## <a name="prerequisites"></a>Requisitos previos

Antes de configurar los valores relacionados con el dispositivo de Azure Stack Edge Pro con GPU, asegúrese de que:

* Para el dispositivo físico:

    - Ha instalado el dispositivo físico como se detalla en [Instalación de Azure Stack Edge Pro](azure-stack-edge-gpu-deploy-install.md).
    - Ha configurado la red y ha habilitado y configurado la red de proceso en el dispositivo, tal y como se detalla en [Tutorial: Configuración de la red para Azure Stack Edge Pro con GPU](azure-stack-edge-gpu-deploy-configure-network-compute-web-proxy.md).


## <a name="configure-device-settings"></a>Configuración de las opciones de dispositivo

Siga estos pasos para configurar los valores relacionadas con el dispositivo:

1. En la página **Dispositivo**, realice los pasos siguientes:

    1. Escriba un nombre descriptivo para el dispositivo. El nombre descriptivo debe contener entre 1 y 13 caracteres, que pueden ser letras, números y guiones.

    2. Proporcione un **Dominio DNS** para el dispositivo. Este dominio se usa para configurar el dispositivo como servidor de archivos.

    3. Seleccione **Aplicar** para validar y aplicar los valores de dispositivo configurados.

        ![Página "Dispositivo" de la interfaz de usuario web local 1](./media/azure-stack-edge-gpu-deploy-set-up-device-update-time/device-2.png)

        Si ha cambiado el nombre del dispositivo y el dominio DNS, los certificados autofirmados generados automáticamente en el dispositivo no funcionarán. Por lo tanto, debe elegir una de las siguientes opciones al configurar los certificados: 
        
        - Genere y descargue los certificados del dispositivo. 
        - Aporte sus propios certificados para el dispositivo, incluida la cadena de firma.
    

        ![Página "Dispositivo" de la interfaz de usuario web local 2](./media/azure-stack-edge-gpu-deploy-set-up-device-update-time/device-3.png)

    4. Cuando se cambia el nombre del dispositivo y el dominio DNS, se crea el punto de conexión SMB.  

    5. Una vez aplicada la configuración, seleccione **Siguiente: Servidor de actualización**.

        ![Página "Dispositivo" de la interfaz de usuario web local](./media/azure-stack-edge-gpu-deploy-set-up-device-update-time/device-4.png)

## <a name="configure-update"></a>Configuración de la actualización

1. Ahora puede configurar la ubicación desde la que se van a descargar las actualizaciones para el dispositivo en la página **Actualizar**.  

    - Puede obtener las actualizaciones directamente desde el **servidor de Microsoft Update**.

        ![Página "Servidor de actualización" de la interfaz de usuario web local](./media/azure-stack-edge-gpu-deploy-set-up-device-update-time/update-2.png)

        También puede optar por implementar las actualizaciones desde **Windows Server Update Services** (WSUS). Proporcione la ruta de acceso al servidor WSUS.
        
        ![Página "Servidor de actualización" de la interfaz de usuario web local](./media/azure-stack-edge-gpu-deploy-set-up-device-update-time/update-3.png)

        > [!NOTE] 
        > Si se configura un servidor de Windows Update independiente y se elige la conexión a través de *https* (en lugar de *http*), se necesitan los certificados de cadena de firma requeridos para conectarse al servidor de actualización. Para información sobre cómo crear y cargar certificados, vaya a [Administrar certificados](azure-stack-edge-j-series-manage-certificates.md). 

2. Seleccione **Aplicar**.
3. Una vez configurado el servidor de actualización, seleccione **Siguiente: Hora**.
    

## <a name="configure-time"></a>Configuración de la hora

Siga estos pasos para configurar los valores de hora en el dispositivo. 

> [!IMPORTANT]
> Aunque la configuración de la hora es opcional, se recomienda encarecidamente configurar un NTP principal y un servidor NTP secundario en la red local para el dispositivo. Si el servidor local no está disponible, se pueden configurar servidores NTP públicos.

Se requieren servidores NTP, ya que el dispositivo debe sincronizar la hora para que pueda autenticarse con los proveedores de servicios en la nube.

1. Puede seleccionar la zona horaria y los servidores NTP principal y secundario para el dispositivo en la página **Hora**.  
    
    1. En la lista desplegable **Zona horaria**, seleccione la correspondiente a la ubicación geográfica en que se va a implementar el dispositivo.
        La zona horaria predeterminada para el dispositivo es la hora del Pacífico (PST). El dispositivo usará esta zona horaria para todas las operaciones programadas.

    2. En el cuadro **Servidor NTP principal**, especifique el servidor principal del dispositivo o acepte el valor predeterminado de time.windows.com.  
        Asegúrese de que la red permite que el tráfico NTP pase del centro de datos a Internet.

    3. Si lo desea, en el cuadro **Servidor NTP secundario**, escriba un servidor secundario para el dispositivo.

    4. Seleccione **Aplicar** para validar y aplicar la configuración horaria.

        ![Página "Hora" de la interfaz de usuario web local](./media/azure-stack-edge-gpu-deploy-set-up-device-update-time/time-2.png)

2. Una vez aplicada la configuración, seleccione **Siguiente: Certificados**.


## <a name="next-steps"></a>Pasos siguientes

En este tutorial, obtendrá información sobre lo siguiente:

> [!div class="checklist"]
>
> * Requisitos previos
> * Configuración de las opciones de dispositivo
> * Configuración de la actualización 
> * Configuración de la hora

Para información sobre cómo configurar certificados para el dispositivo Azure Stack Edge Pro, consulte:

> [!div class="nextstepaction"]
> [Configurar certificados](./azure-stack-edge-gpu-deploy-configure-certificates.md)
