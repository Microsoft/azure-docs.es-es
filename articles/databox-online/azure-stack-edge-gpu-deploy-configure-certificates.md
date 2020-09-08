---
title: Tutorial para configurar certificados para un dispositivo de Azure Stack Edge con GPU en Azure Portal | Microsoft Docs
description: En este tutorial para implementar Azure Stack Edge con GPU, se indica cómo configurar certificados en el dispositivo físico.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: tutorial
ms.date: 08/29/2020
ms.author: alkohli
Customer intent: As an IT admin, I need to understand how to configure certificates for Azure Stack Edge so I can use it to transfer data to Azure.
ms.openlocfilehash: 80a857f80fd2c164637e591fbab43123659cd2f7
ms.sourcegitcommit: bcda98171d6e81795e723e525f81e6235f044e52
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89268183"
---
# <a name="tutorial-configure-certificates-for-your-azure-stack-edge-with-gpu"></a>Tutorial: Configuración de certificados para Azure Stack Edge con GPU

En este tutorial se describe cómo puede configurar certificados para el dispositivo de Azure Stack Edge con una GPU incorporada mediante la interfaz de usuario web local.

El tiempo necesario para este paso puede variar en función de la opción específica que elija y de cómo se establezca el flujo de certificados en su entorno.

En este tutorial, obtendrá información sobre lo siguiente:

> [!div class="checklist"]
>
> * Requisitos previos
> * Configuración de certificados para el dispositivo físico

## <a name="prerequisites"></a>Requisitos previos

Antes de configurar e instalar el dispositivo de Azure Stack Edge con GPU, compruebe lo siguiente:

* Ha instalado el dispositivo físico como se detalla en [Instalación de Azure Stack Edge](azure-stack-edge-gpu-deploy-install.md).
* Si tiene previsto aportar sus propios certificados:
    - Debe tener los certificados preparados en el formato adecuado, incluido el certificado de la cadena de firma.
<!--    - If your device is deployed in Azure Government or Azure Government Secret or Azure Government top secret cloud and not deployed in Azure public cloud, a signing chain certificate is required before you can activate your device. 
    For details on certificate, go to [Manage certificates](azure-stack-edge-j-series-manage-certificates.md).-->


## <a name="configure-certificates-for-device"></a>Configuración de certificados para el dispositivo


1. En el icono **Seguridad**, seleccione **Configurar** para los certificados. 

    ![Página "Certificados" de la interfaz de usuario web local](./media/azure-stack-edge-gpu-deploy-configure-certificates/generate-certificate-1.png)

2. En función de si cambió el nombre del dispositivo o el dominio DNS en el icono **Instalación del dispositivo**, puede elegir una de las siguientes opciones para los certificados.

    - Si no ha cambiado el nombre del dispositivo o el dominio DNS en el paso anterior y no desea aportar sus propios certificados, puede omitir este paso y continuar en el paso siguiente. El dispositivo ha generado automáticamente certificados autofirmados para comenzar. 

        ![Página "Certificados" de la interfaz de usuario web local](./media/azure-stack-edge-gpu-deploy-configure-certificates/generate-certificate-2.png)

    - Si cambió el nombre del dispositivo o el dominio DNS, para activar correctamente el dispositivo, puede elegir una de las siguientes opciones: 
    
        - **Generación de todos los certificados del dispositivo**. Los certificados del dispositivo solo se deben usar para las pruebas y no con las cargas de trabajo de producción. ** Para más información, vaya a [Generación de certificados de dispositivo en Azure Stack Edge](#generate-device-certificates).

        - **Aportación de sus propios certificados**. Puede aportar sus propios certificados de punto de conexión firmados y las cadenas de firma correspondientes. Primero debe agregar la cadena de firma y, a continuación, cargar los certificados de punto de conexión. **Se recomienda que siempre aporte sus propios certificados para cargas de trabajo de producción.** Para más información, vaya a [Aportación de sus propios certificados en el dispositivo de Azure Stack Edge](#bring-your-own-certificates).
    
        - Puede aportar algunos de sus propios certificados y generar algunos certificados de dispositivo. La opción **Generar certificados** solo volverá a generar los certificados del dispositivo.

    - Si cambió el nombre del dispositivo o el dominio DNS, y no genera certificados ni aporta sus propios certificados, se bloqueará la activación.


### <a name="generate-device-certificates"></a>Generación de certificados de dispositivo

Siga estos pasos para generar certificados de dispositivo.

Siga estos pasos para volver a generar y descargar los certificados de dispositivo de Azure Stack Edge:

1. En la interfaz de usuario local del dispositivo, vaya a **Configuración > Certificados**. Seleccione **Generar certificados**.

    ![Generación y descarga del certificado 1](./media/azure-stack-edge-gpu-deploy-configure-certificates/generate-certificate-3.png)

2. En **Generar certificados** del dispositivo, seleccione **Generar**.

    ![Generación y descarga del certificado 2](./media/azure-stack-edge-gpu-deploy-configure-certificates/generate-certificate-4.png)

    Se generan y aplican los certificados de dispositivo. 
    
    > [!IMPORTANT]
    > Mientras la operación de generación de certificados está en curso, no aporte sus propios certificados e intente agregarlos a través de la opción **+ Agregar certificado**.

    Recibirá una notificación cuando la operación se haya completado correctamente. Para evitar posibles problemas de caché, reinicie el explorador. 
    
    ![Generación y descarga del certificado 4](./media/azure-stack-edge-gpu-deploy-configure-certificates/generate-certificate-5.png)

3. En la página **Certificados**, verá que la columna **Descargar** se ha rellenado y están disponibles los vínculos para descargar los certificados regenerados. 

    ![Generación y descarga del certificado 5](./media/azure-stack-edge-gpu-deploy-configure-certificates/generate-certificate-6.png)

4. Seleccione el vínculo de descarga de un certificado y, cuando se le pida, guarde el certificado. 

    ![Generación y descarga del certificado 6](./media/azure-stack-edge-gpu-deploy-configure-certificates/generate-certificate-7.png)

5. Repita este proceso para todos los certificados que desee descargar. 
    
    ![Generación y descarga del certificado 7](./media/azure-stack-edge-gpu-deploy-configure-certificates/generate-certificate-8.png)

    Los certificados generados por el dispositivo se guardan como certificados DER con el siguiente formato de nombre: 

    `<Device name>_<Endpoint name>.cer`. Estos certificados contienen la clave pública para los certificados correspondientes instalados en el dispositivo. 

Deberá instalar estos certificados en el sistema cliente que usa para acceder a los puntos de conexión en el dispositivo ASE. Estos certificados establecen confianza entre el cliente y el dispositivo.

Para importar e instalar estos certificados en el cliente que usa para acceder al dispositivo, siga los pasos descritos en [Importación de certificados en los clientes que tienen acceso al dispositivo de Azure Stack Edge](azure-stack-edge-j-series-manage-certificates.md#import-certificates-on-the-client-accessing-the-device). 

Si usa el Explorador de Azure Storage, tendrá que instalar los certificados en el cliente en formato PEM y deberá convertir los certificados generados por el dispositivo a formato PEM. 

> [!IMPORTANT]
> - El vínculo de descarga solo está disponible para los certificados generados por el dispositivo y no si aporta sus propios certificados.
> - Puede optar por una combinación de certificados generados por el dispositivo y sus propios certificados, siempre y cuando se cumplan otros requisitos de certificado. Para más información, vaya a [Requisitos de certificado](azure-stack-edge-j-series-certificate-requirements.md).
    

### <a name="bring-your-own-certificates"></a>Aportación de sus propios certificados

Siga estos pasos para agregar sus propios certificados, incluida la cadena de firma.

1. Para cargar el certificado, en la página **Certificado**, seleccione **+ Agregar certificado**.

    ![Página "Certificados" de la interfaz de usuario web local](./media/azure-stack-edge-gpu-deploy-configure-certificates/add-certificate-1.png)

2. Cargue primero la cadena de firma y seleccione **Validar y agregar**.

    ![Página "Certificados" de la interfaz de usuario web local](./media/azure-stack-edge-gpu-deploy-configure-certificates/add-certificate-2.png)

3. Ahora puede cargar otros certificados. Por ejemplo, puede cargar los certificados de punto de conexión de Blob Storage y Azure Resource Manager.

    ![Página "Certificados" de la interfaz de usuario web local](./media/azure-stack-edge-gpu-deploy-configure-certificates/add-certificate-3.png)

    También puede cargar el certificado de la interfaz de usuario web local. Después de cargar este certificado, se le pedirá que inicie el explorador y borre la memoria caché. A continuación, tendrá que conectarse a la interfaz de usuario web local del dispositivo.  

    ![Página "Certificados" de la interfaz de usuario web local](./media/azure-stack-edge-gpu-deploy-configure-certificates/add-certificate-5.png)

    También puede cargar el certificado de nodo.

    ![Página "Certificados" de la interfaz de usuario web local](./media/azure-stack-edge-gpu-deploy-configure-certificates/add-certificate-4.png)

    La página del certificado debe actualizarse para reflejar los certificados recién agregados.

    ![Página "Certificados" de la interfaz de usuario web local](./media/azure-stack-edge-gpu-deploy-configure-certificates/add-certificate-6.png)  

    > [!NOTE]
    > A excepción de la nube pública de Azure, es necesario que los certificados de la cadena de firma se agreguen antes de la activación de todas las configuraciones de nube (Azure Government o Azure Stack).


El dispositivo ya está listo para activarse.


## <a name="next-steps"></a>Pasos siguientes

En este tutorial, obtendrá información sobre lo siguiente:

> [!div class="checklist"]
>
> * Requisitos previos
> * Configuración de certificados para el dispositivo físico

Para aprender a administrar el dispositivo de Azure Stack Edge, consulte:

> [!div class="nextstepaction"]
> [Activación del dispositivo de Azure Stack Edge](./azure-stack-edge-gpu-deploy-activate.md)
