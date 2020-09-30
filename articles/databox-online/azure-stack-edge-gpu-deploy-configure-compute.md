---
title: 'Tutorial: Filtrado y análisis de los datos con proceso en Azure Stack Edge Pro con GPU | Microsoft Docs'
description: Aprenda a configurar el rol de proceso en Azure Stack Edge Pro con GPU y a usarlo para transformar los datos antes de enviarlos a Azure.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: tutorial
ms.date: 08/28/2020
ms.author: alkohli
Customer intent: As an IT admin, I need to understand how to configure compute on Azure Stack Edge Pro so I can use it to transform the data before sending it to Azure.
ms.openlocfilehash: 95c59cff1f47fe720e2dbc65c5b0a69a09be2f2f
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90903180"
---
# <a name="tutorial-configure-compute-on-azure-stack-edge-pro-gpu-device"></a>Tutorial: Configuración del proceso en un dispositivo de Azure Stack Edge Pro con GPU

<!--[!INCLUDE [applies-to-skus](../../includes/azure-stack-edge-applies-to-all-sku.md)]-->

Este tutorial describe cómo configurar el rol de proceso y crear un clúster de Kubernetes en un dispositivo de Azure Stack Edge Pro. 

Este procedimiento tarda aproximadamente entre 20 y 30 minutos en completarse.


En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Configurar el proceso
> * Obtener puntos de conexión de Kubernetes

 
## <a name="prerequisites"></a>Requisitos previos

Antes de configurar un rol de proceso en el dispositivo de Azure Stack Edge Pro, asegúrese de que:

- Ha activado un dispositivo Azure Stack Edge Pro como se describe en [Activación de Azure Stack Edge Pro](azure-stack-edge-gpu-deploy-activate.md).
- Asegúrese de que ha seguido las instrucciones de [Habilitación de la red de proceso](azure-stack-edge-gpu-deploy-configure-network-compute-web-proxy.md#enable-compute-network) y:
    - Ha habilitado una interfaz de red para proceso.
    - Ha asignado direcciones IP de nodo de Kubernetes y direcciones IP de servicio externo de Kubernetes.

## <a name="configure-compute"></a>Configurar el proceso

Para configurar el proceso en Azure Stack Edge Pro, creará un recurso de IoT Hub en Azure Portal.

1. En la instancia de Azure Portal del recurso de Azure Stack Edge, vaya a **Información general**. En el panel derecho, en el icono **Proceso**, seleccione **Comenzar**.

    ![Introducción al proceso](./media/azure-stack-edge-j-series-deploy-configure-compute/configure-compute-1.png)

2. En el icono **Configurar el proceso de Edge**, seleccione **Configurar proceso**.

    ![Configurar el proceso](./media/azure-stack-edge-j-series-deploy-configure-compute/configure-compute-2.png)

3. En la hoja **Configurar el proceso de Edge**, escriba lo siguiente:

   
    |Campo  |Value  |
    |---------|---------|
    |IoT Hub     | Elija entre **Nuevo** o **Existente**. <br> De forma predeterminada, se usa un nivel estándar (S1) para crear un recurso de IoT. Para usar un recurso de IoT de nivel gratuito, cree uno y, a continuación, seleccione el recurso existente. <br> En cualquier caso, el recurso de IoT Hub usa la misma suscripción y el mismo grupo de recursos que el recurso de Azure Stack Edge.     |
    |Nombre     |Escriba un nombre para el recurso de IoT Hub.         |

    ![Introducción al proceso](./media/azure-stack-edge-j-series-deploy-configure-compute/configure-compute-3.png)

4. Seleccione **Crear**. La creación del recurso de IoT Hub tarda varios minutos. Después de crear el recurso de IoT Hub, el icono **Configurar proceso** se actualiza para mostrar la configuración de proceso. 

    ![Introducción al proceso](./media/azure-stack-edge-j-series-deploy-configure-compute/configure-compute-4.png)

5. Para confirmar que se ha configurado el rol de proceso perimetral, seleccione **Ver proceso** en el icono **Configurar proceso**.
    
    ![Introducción al proceso](./media/azure-stack-edge-j-series-deploy-configure-compute/configure-compute-5.png)

    > [!NOTE]
    > Si el cuadro de diálogo **Configurar proceso** se cierra antes de que IoT Hub esté asociado con el dispositivo de Azure Stack Edge Pro, la instancia de IoT Hub se crea, pero no se muestra en la configuración de proceso. 
    
Cuando el rol de proceso de Edge está configurado en el dispositivo de Edge, este crea dos dispositivos: uno IoT y el otro IoT Edge. Ambos se pueden ver en el recurso de IoT Hub. Un entorno de ejecución de IoT Edge también se ejecuta en este dispositivo de IoT Edge. En este momento, solo está disponible la plataforma Linux para el dispositivo IoT Edge.

Puede tardar de 20 a 30 minutos en configurar el proceso, ya que en segundo plano se crean las máquinas virtuales y el clúster de Kubernetes. 

Después de haber configurado correctamente el proceso en Azure Portal, existe un clúster de Kubernetes y un usuario predeterminado asociado al espacio de nombres de IoT (un espacio de nombres del sistema controlado por Azure Stack Edge Pro). 

## <a name="get-kubernetes-endpoints"></a>Obtención de los puntos de conexión de Kubernetes

Para configurar un cliente para obtener acceso al clúster de Kubernetes, necesitará el punto de conexión de Kubernetes. Siga estos pasos para obtener el punto de conexión de la API de Kubernetes desde la interfaz de usuario local del dispositivo de Azure Stack Edge Pro.

1. En la interfaz de usuario web local del dispositivo, vaya a la página **Dispositivos**.
2. En **Puntos de conexión del dispositivo**, copie el punto de conexión del **servicio de API de Kubernetes**. Este punto de conexión es una cadena en el formato siguiente: `https://compute.<device-name>.<DNS-domain>[Kubernetes-cluster-IP-address]`. 

    ![Página del dispositivo en la interfaz de usuario local](./media/azure-stack-edge-j-series-create-kubernetes-cluster/device-kubernetes-endpoint-1.png)

3. Guarde la cadena de punto de conexión. La usará más adelante al configurar un cliente para tener acceso al clúster de Kubernetes a través de kubectl.

4. Mientras está en la interfaz de usuario web local, puede:

    - Ir a la API de Kubernetes, seleccionar **Configuración avanzada** y descargar un archivo de configuración avanzada para Kubernetes. 

        ![Página del dispositivo de la interfaz de usuario local 1](./media/azure-stack-edge-gpu-deploy-configure-compute/download-advanced-config-1.png)

        Si se le ha proporcionado una clave de Microsoft (seleccione los usuarios que pueden tener esto), puede usar este archivo de configuración.

        ![Página del dispositivo de la interfaz de usuario local 2](./media/azure-stack-edge-gpu-deploy-configure-compute/download-advanced-config-2.png)

    - También puede ir al punto de conexión del **panel de Kubernetes** y descargar un archivo de configuración de `aseuser`. 
    
        ![Página del dispositivo de la interfaz de usuario local 3](./media/azure-stack-edge-gpu-deploy-configure-compute/download-aseuser-config-1.png)

        Puede usar este archivo de configuración para iniciar sesión en el panel de Kubernetes o para depurar cualquier problema en el clúster de Kubernetes. Para más información, consulte [Acceso al panel de Kubernetes](azure-stack-edge-gpu-monitor-kubernetes-dashboard.md#access-dashboard). 


## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha aprendido a:

> [!div class="checklist"]
> * Configurar el proceso
> * Obtención de los puntos de conexión de Kubernetes


Para aprender a administrar el dispositivo de Azure Stack Edge Pro, consulte:

> [!div class="nextstepaction"]
> [Uso de la interfaz de usuario web local para administrar Azure Stack Edge Pro](azure-stack-edge-manage-access-power-connectivity-mode.md).
