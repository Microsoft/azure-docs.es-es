---
title: Administración de la red de proceso en Azure Stack Edge Pro para acceder a módulos | Microsoft Docs
description: En este artículo se explica cómo extender la red de proceso en Azure Stack Edge Pro para acceder a los módulos por medio de una dirección IP externa.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: how-to
ms.date: 05/17/2019
ms.author: alkohli
ms.openlocfilehash: 19c92deb58ac51aa882e7123b9a90aa3eae627d0
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90894118"
---
# <a name="enable-compute-network-on-your-azure-stack-edge-pro"></a>Habilitación de la red de proceso en Azure Stack Edge Pro

En este artículo se explica el modo en el que los módulos que se ejecutan en Azure Stack Edge Pro pueden acceder a la red de proceso habilitada en el dispositivo.

Para configurar la red, deberá seguir los siguientes pasos:

- Habilite una interfaz de red en el dispositivo Azure Stack Edge Pro para el proceso.
- Agregue un módulo para acceder a la red de proceso en Azure Stack Edge Pro.
- Compruebe que el módulo pueda acceder a la interfaz de red habilitada.

En este tutorial usará un módulo de aplicaciones de servidor web para demostrar el escenario.

## <a name="prerequisites"></a>Prerrequisitos

Antes de comenzar, necesitará:

- Un dispositivo Azure Stack Edge Pro con la configuración de dispositivo completada.
- Ya completó el paso **Configurar el proceso** según el [tutorial: transformación de datos con Azure Stack Edge Pro](azure-stack-edge-deploy-configure-compute-advanced.md#configure-compute) en el dispositivo. Su dispositivo debe tener un recurso de IoT Hub asociado, un dispositivo IoT y un dispositivo IoT Edge.

## <a name="enable-network-interface-for-compute"></a>Habilitar una interfaz de red para el proceso

Para obtener acceso a los módulos que se ejecutan en el dispositivo a través de una red externa, deberá asignar una dirección IP a una interfaz de red en el dispositivo. Esta configuración del proceso se puede administrar desde la interfaz de usuario web local.

Realice los pasos siguientes en la interfaz de usuario web local para configurar el proceso.

1. En la interfaz de usuario web local, vaya a **Configuración > Configuración del proceso**.  

2. **Habilite** la interfaz de red que quiera usar para conectarse al módulo de procesos que ejecutará en el dispositivo.

    - Si usa direcciones IP estáticas, escriba una dirección IP para la interfaz de red.
    - Si usa DHCP, las direcciones IP se asignan automáticamente. En este ejemplo se usa DHCP.

    ![Habilitar la configuración del proceso 1](media/azure-stack-edge-extend-compute-access-modules/enable-compute-setting-1.png)

3. Seleccione **Aplicar** para aplicar la configuración. Tome nota de la dirección IP asignada a la interfaz de red si usa DHCP.

    ![Habilitación de la configuración del proceso](media/azure-stack-edge-extend-compute-access-modules/enable-compute-setting-2.png)

## <a name="add-webserver-app-module"></a>Agregar el módulo de aplicaciones de servidor web

Siga estos pasos para agregar un módulo de aplicaciones de servidor web al dispositivo Azure Stack Edge Pro.

1. Vaya al recurso de IoT Hub asociado al dispositivo Azure Stack Edge Pro y seleccione **Dispositivo IoT Edge**.
2. Seleccione el dispositivo IoT Edge asociado al dispositivo Azure Stack Edge Pro. En los **detalles del dispositivo**, seleccione **Establecer módulos**. En **Agregar módulos**, seleccione **+ Agregar** y **módulo de IoT Edge**.
3. En la hoja de los **módulos personalizados de IoT Edge**:

    1. Especifique un **nombre** para el módulo de aplicaciones del servidor web que quiera implementar.
    2. Proporcione un **URI de imagen** para la imagen del módulo. Se recuperará un módulo que coincide con el nombre proporcionado y las etiquetas. En ese caso, `nginx:stable` extraerá una imagen nginx estable (etiquetada como estable) del [repositorio público de Docker](https://hub.docker.com/_/nginx/).
    3. En el cuadro **Opciones de creación del contenedor**, puede pegar el ejemplo de código siguiente:  

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

        Esta configuración le permite acceder al módulo usando la IP de la red de proceso sobre *http* en el puerto TCP 8080 (con el puerto del servidor web predeterminado establecido en 80).

        ![Especifique la información del puerto en la hoja de módulo personalizado de IoT Edge.](media/azure-stack-edge-extend-compute-access-modules/module-information.png)

    4. Seleccione **Guardar**.

## <a name="verify-module-access"></a>Comprobar el acceso del módulo

1. Compruebe que el módulo se haya implementado correctamente y se esté ejecutando. En la página de **detalles del dispositivo**, en la pestaña **Módulos**, el estado de ejecución del módulo debe ser **en ejecución**.  
2. Conéctese al módulo de aplicaciones del servidor web. Abra una ventana del navegador y escriba:

    `http://<compute-network-IP-address>:8080`

    Debería ver que la aplicación del servidor web se está ejecutando.

    ![Compruebe la conexión al módulo sobre el puerto especificado.](media/azure-stack-edge-extend-compute-access-modules/verify-connect-module-1.png)

## <a name="next-steps"></a>Pasos siguientes

- Aprenda a [administrar usuarios desde Azure Portal](azure-stack-edge-manage-users.md).
