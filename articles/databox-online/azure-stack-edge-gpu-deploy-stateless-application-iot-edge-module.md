---
title: Implementación de una aplicación sin estado de Kubernetes en Azure Stack Edge Pro con GPU a través del módulo IoT Edge | Microsoft Docs
description: Se describe cómo implementar una aplicación sin estado de Kubernetes en un dispositivo Azure Stack Edge Pro con GPU mediante un módulo IoT Edge al que se tiene acceso a través de una dirección IP externa.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: how-to
ms.date: 08/26/2020
ms.author: alkohli
ms.openlocfilehash: 4bc598080b96886e6734ac3709761465a1a28d49
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90899521"
---
# <a name="use-iot-edge-module-to-run-a-kubernetes-stateless-application-on-your-azure-stack-edge-pro-gpu-device"></a>Uso de un módulo IoT Edge para ejecutar una aplicación sin estado de Kubernetes en un dispositivo Azure Stack Edge Pro con GPU

En este artículo se describe cómo puede usar un módulo IoT Edge para implementar una aplicación sin estado en un dispositivo Azure Stack Edge Pro.

Para implementar la aplicación sin estado, realice los pasos siguientes:

- Asegúrese de que se completan los requisitos previos antes de implementar un módulo IoT Edge.
- Agregue un módulo IoT Edge para acceder a la red de proceso en Azure Stack Edge Pro.
- Compruebe que el módulo pueda acceder a la interfaz de red habilitada.

En este artículo de procedimientos, usará un módulo de aplicación de servidor web para demostrar el escenario.

## <a name="prerequisites"></a>Prerrequisitos

Antes de comenzar, necesitará:

- Un dispositivo Azure Stack Edge Pro. Asegúrese de lo siguiente:

    - Se han configurado las opciones de la red de proceso en el dispositivo.
    - El dispositivo se ha activado según los pasos descritos en [Tutorial: Activación del dispositivo](azure-stack-edge-gpu-deploy-activate.md).
- Ya completó el paso **Configurar el proceso** según el [tutorial: Configure el proceso en su dispositivo Azure Stack Edge Pro](azure-stack-edge-gpu-deploy-configure-compute.md). Su dispositivo debe tener un recurso de IoT Hub asociado, un dispositivo IoT y un dispositivo IoT Edge.


## <a name="add-webserver-app-module"></a>Agregar el módulo de aplicaciones de servidor web

Siga estos pasos para agregar un módulo de aplicaciones de servidor web al dispositivo Azure Stack Edge Pro.

1. En el recurso de IoT Hub asociado con el dispositivo, vaya a **Administración de dispositivos automática > IoT Edge**.
1. Seleccione el dispositivo IoT Edge asociado a su dispositivo Azure Stack Edge Pro y haga clic en él. 

    ![Selección de un dispositivo IoT Edge](media/azure-stack-edge-gpu-deploy-stateless-application-iot-edge-module/select-iot-edge-device-1.png)  

1. Seleccione **Set modules** (Establecer módulos). En **Establecer módulos en el dispositivo**, seleccione **+ Agregar** y **Módulo IoT Edge**.

    ![Selección de módulo IoT Edge](media/azure-stack-edge-gpu-deploy-stateless-application-iot-edge-module/select-iot-edge-module-1.png)

1. En **Agregar módulo IoT Edge**:

    1. Especifique un **nombre** para el módulo de aplicaciones del servidor web que quiera implementar.
    2. En la pestaña **Configuración del módulo**, proporcione un **URI de la imagen** para la imagen del módulo. Se recuperará un módulo que coincide con el nombre proporcionado y las etiquetas. En ese caso, `nginx:stable` extraerá una imagen nginx estable (etiquetada como estable) del [repositorio público de Docker](https://hub.docker.com/_/nginx/).

        ![Agregar módulo IoT Edge](media/azure-stack-edge-gpu-deploy-stateless-application-iot-edge-module/set-module-settings-1.png)    

    3. En la pestaña **Opciones de creación del contenedor**, pegue el ejemplo de código siguiente:  

        ```
        {
            "HostConfig": {
                "PortBindings": {
                    "80/tcp": [
                        {
                            "HostPort": "8080"
                        }
                    ]
                }
            }
        }
        ```

        Esta configuración le permite acceder al módulo usando la IP de la red de proceso sobre *http* en el puerto TCP 8080 (con el puerto del servidor web predeterminado establecido en 80). Seleccione **Agregar**.

        ![Especifique la información del puerto en la hoja de módulo personalizado de IoT Edge.](media/azure-stack-edge-gpu-deploy-stateless-application-iot-edge-module/verify-module-status-1.png)

    4. Seleccione **Revisar + crear**. Revise los detalles del módulo y seleccione **Crear**.

## <a name="verify-module-access"></a>Comprobar el acceso del módulo

1. Compruebe que el módulo se haya implementado correctamente y se esté ejecutando. En la pestaña **Módulos**, el estado en tiempo de ejecución del módulo debe ser **en ejecución**.  

    ![Comprobación de que el estado del módulo es en ejecución](media/azure-stack-edge-gpu-deploy-stateless-application-iot-edge-module/verify-module-status-1.png)

1. Para obtener el punto de conexión externo de la aplicación de servidor web, [acceda al panel de Kubernetes](azure-stack-edge-gpu-monitor-kubernetes-dashboard.md#access-dashboard). 
1. En el panel izquierdo del panel de información, vaya al espacio de nombres **iotedge**. Vaya a **Discovery and Load balancing (Detección y equilibrio de carga) > Servicios**. En la lista de servicios enumerados, busque el punto de conexión externo del módulo de aplicación de servidor web. 

    ![Conexión a la aplicación de servidor web en el punto de conexión externo](media/azure-stack-edge-gpu-deploy-stateless-application-iot-edge-module/connect-external-endpoint-1.png)

1. Seleccione el punto de conexión externo para abrir una nueva ventana del explorador.

    Debería ver que la aplicación del servidor web se está ejecutando.

    ![Compruebe la conexión al módulo sobre el puerto especificado.](media/azure-stack-edge-gpu-deploy-stateless-application-iot-edge-module/verify-webserver-app-1.png)

## <a name="next-steps"></a>Pasos siguientes

- Obtenga información sobre cómo exponer una aplicación con estado a través de un módulo IoT Edge<!--insert link-->.
