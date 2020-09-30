---
title: Implementación de un módulo de GPU en un dispositivo Azure Stack Edge Pro | Microsoft Docs
description: En este artículo se describe cómo habilitar el proceso y preparar el dispositivo Azure Stack Edge Pro para el proceso a través de la interfaz de usuario local.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: how-to
ms.date: 08/31/2020
ms.author: alkohli
ms.openlocfilehash: 1f16ef0ede25f17acb915a7812ae5b15b45f78a4
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90899728"
---
# <a name="deploy-a-gpu-enabled-iot-module-on-azure-stack-edge-pro-gpu-device"></a>Implementación de un módulo de IoT habilitado para GPU en un dispositivo Azure Stack Edge Pro

En este artículo se describe cómo implementar un módulo de IoT Edge habilitado para GPU en un dispositivo Azure Stack Edge Pro con GPU. 

En este artículo aprenderá a:
  - Preparar Azure Stack Edge Pro para ejecutar un módulo GPU.
  - Descargar e instalar código de ejemplo desde un repositorio de Git.
  - Compilar la solución y generar un manifiesto de implementación.
  - Implementar la solución en un dispositivo Azure Stack Edge Pro.
  - Supervisar la salida del módulo.


## <a name="about-sample-module"></a>Información sobre el módulo de ejemplo

El módulo de ejemplo de GPU de este artículo incluye código de ejemplo de pruebas comparativas de PyTorch y TensorFlow entre CPU y GPU.

## <a name="prerequisites"></a>Requisitos previos

Antes de comenzar, asegúrese de que tiene:

- Tiene acceso a un dispositivo Azure Stack Edge Pro con un nodo habilitado para GPU. Este dispositivo se activa con un recurso en Azure. Consulte [Activación del dispositivo](azure-stack-edge-gpu-deploy-activate.md).
- Ha configurado el proceso en este dispositivo. Siga los pasos del [Tutorial: Configuración del proceso en un dispositivo Azure Stack Edge Pro](azure-stack-edge-gpu-deploy-configure-compute.md).
- Un registro de Azure Container Registry (ACR). Vaya a la hoja **Claves de acceso** y anote la contraseña, el nombre de usuario y el servidor de inicio de sesión de ACR. Para obtener más información, vaya a [Inicio rápido: Creación de un registro de contenedor privado con Azure Portal](../container-registry/container-registry-get-started-portal.md#create-a-container-registry).
- Los siguientes recursos de desarrollo en un cliente Windows:
    - [CLI de Azure 2.0 o posterior](https://aka.ms/installazurecliwindows)
    - [Docker CE](https://store.docker.com/editions/community/docker-ce-desktop-windows). Es posible que deba crear una cuenta para descargar e instalar el software.
    - [Visual Studio Code](https://code.visualstudio.com/)  
    - [Extensión de Azure IoT Edge para Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-edge).    
    - [Extensión de Python para Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=ms-python.python)    
    - [Python 3](https://www.python.org/)    
    - Pip para instalar paquetes de Python (normalmente, se incluye con la instalación de Python)

## <a name="get-the-sample-code"></a>Obtención del código de ejemplo

1. Vaya a [Patrones de Azure Intelligent Edge en ejemplos de Azure](https://github.com/azure-samples/azure-intelligent-edge-patterns). Clone o descargue el archivo .zip del código. 

    ![Descargar el archivo ZIP](media/azure-stack-edge-gpu-deploy-sample-module/download-zip-file-1.png)

    Extraiga los archivos del archivo .zip. También puede clonar los ejemplos.

    ```json
    git clone https://github.com/Azure-Samples/azure-intelligent-edge-patterns.git
    ```

## <a name="build-and-deploy-module"></a>Compilación e implementación del módulo

1. Abra la carpeta **GpuReferenceModules** en Visual Studio Code.

    ![Apertura de GPUReferenceModules en Visual Studio Code](media/azure-stack-edge-gpu-deploy-sample-module/open-folder-gpu-sample-1.png)

2. Abra el archivo *deployment.template.json* e identifique los parámetros a los que se hacen referencia para el registro de contenedor. En el siguiente archivo, se usan CONTAINER_REGISTRY_USERNAME, CONTAINER_REGISTRY_PASSWORD y CONTAINER_REGISTRY_NAME.

    ```json
        {
      "$schema-template": "2.0.0",
      "modulesContent": {
        "$edgeAgent": {
          "properties.desired": {
            "schemaVersion": "1.0",
            "runtime": {
              "type": "docker",
              "settings": {
                "minDockerVersion": "v1.25",
                "loggingOptions": "",
                "registryCredentials": {
                  "${CONTAINER_REGISTRY_NAME}":{
                  "username": "$CONTAINER_REGISTRY_USERNAME",
                  "password": "$CONTAINER_REGISTRY_PASSWORD",
                  "address": "${CONTAINER_REGISTRY_NAME}.azurecr.io"
                  }
                }
              }
            },
    ```
3. Cree un archivo. Rellene los valores de los parámetros del registro de contenedor (use los identificados en el paso anterior) de la manera siguiente: 

    ```json
    CONTAINER_REGISTRY_NAME=<YourContainerRegistryName>
    CONTAINER_REGISTRY_USERNAME=<YourContainerRegistryUserName>
    CONTAINER_REGISTRY_PASSWORD=<YourContainerRegistryPassword>
    ```
    A continuación, se muestra un archivo de ejemplo *.env*:
    
    ![Creación y guardado del archivo .env](media/azure-stack-edge-gpu-deploy-sample-module/create-save-env-file-1.png)

4. Guarde el archivo como *.env* en la carpeta **SampleSolution**.

5. Escriba el comando siguiente en el terminal integrado de Visual Studio Code para iniciar sesión en Docker. 

    ```json
    docker login -u <CONTAINER_REGISTRY_USERNAME> -p <CONTAINER_REGISTRY_PASSWORD> <CONTAINER_REGISTRY_NAME>
    ```
    Vaya a la sección **Claves de acceso** del contenedor registro en Azure Portal. Copie y use el servidor de inicio de sesión, la contraseña y el nombre del registro.

    ![Claves de acceso del contenedor de registro](media/azure-stack-edge-gpu-deploy-sample-module/container-registry-access-keys-1.png)

    Una vez que se proporcionan las credenciales, el inicio de sesión se realiza correctamente.

    ![Inicio de sesión correcto](media/azure-stack-edge-gpu-deploy-sample-module/successful-sign-in-1.png)

6. Inserte la imagen en el registro de contenedor de Azure. En el explorador de Visual Studio Code, haga clic con el botón derecho en el archivo **deployment.template.json** y seleccione **Build and Push IoT Edge solution** (Compilar e insertar solución de IoT Edge). 

    ![Compilación e inserción de la solución de IoT Edge](media/azure-stack-edge-gpu-deploy-sample-module/build-push-iot-edge-solution-1.png)   

    Si no se instalan las extensiones de Python y Python, se hará al compilar e insertar la solución. Sin embargo, se producirían tiempos de compilación más largos. 

    Una vez completado este paso, verá el módulo en el registro de contenedor.

    ![Módulo en el registro de contenedor](media/azure-stack-edge-gpu-deploy-sample-module/module-container-registry-1.png)    


7. Para crear un manifiesto de implementación, haga clic con el botón derecho en el archivo **deployment.template.json** y seleccione **Generate IoT Edge Deployment Manifest** (Generar manifiesto de implementación de IoT Edge). 

    ![Generación de un manifiesto de implementación de IoT Edge](media/azure-stack-edge-gpu-deploy-sample-module/generate-iot-edge-deployment-manifest-1.png)  

    La notificación informa de la ruta de acceso en la que se generó el manifiesto de implementación. El manifiesto es el archivo `deployment.amd64.json` generado en la carpeta **config**. 

8. Seleccione el archivo **deployment.amd64.json** en la carpeta **config** y, a continuación, elija **Create Deployment for Single Device** (Crear una implementación para un dispositivo individual). No utilice el archivo **deployment.template.json**. 

    ![Crear una implementación para un dispositivo individual](media/azure-stack-edge-gpu-deploy-sample-module/create-deployment-single-device-1.png)  

    En la ventana **Salida**, debería ver un mensaje que indica que la implementación se realizó correctamente.

    ![Implementación correcta en Salida](media/azure-stack-edge-gpu-deploy-sample-module/deployment-succeeded-output-1.png) 

## <a name="monitor-the-module"></a>Supervisión del módulo  

1. En la paleta de comandos de VS Code, ejecute **Azure IoT Hub: Select IoT Hub**.

2. Elija la suscripción y la instancia de IoT Hub que contienen el dispositivo IoT Edge que desea configurar. En este caso, seleccione la suscripción que se ha usado para implementar el dispositivo Azure Stack Edge Pro y seleccione el dispositivo IoT Edge creado para el dispositivo Azure Stack Edge Pro. Esto sucede cuando se configura el proceso mediante Azure Portal en los pasos anteriores.

3. En el explorador de Visual Studio Code, expanda la sección Azure IoT Hub. En **Dispositivos**, debería ver el dispositivo IoT Edge correspondiente a su dispositivo Azure Stack Edge Pro. 

    1. Seleccione ese dispositivo, haga clic con el botón derecho en **Iniciar la supervisión del punto de conexión de eventos integrado**.
  
        ![Iniciar supervisión](media/azure-stack-edge-gpu-deploy-sample-module/monitor-builtin-event-endpoint-1.png)  

    2. Vaya a **Dispositivos > Módulos** y debería ver el **módulo de GPU** en ejecución.

        ![Módulo en IoT Hub](media/azure-stack-edge-gpu-deploy-sample-module/module-iot-hub-1.png)  

    3. El terminal de Visual Studio Code también debería mostrar los eventos de IoT Hub como salida de supervisión para el dispositivo Azure Stack Edge Pro.

        ![Supervisión de la salida](media/azure-stack-edge-gpu-deploy-sample-module/monitor-events-output-1.png) 

        Puede ver que el tiempo necesario para ejecutar el mismo conjunto de operaciones (5000 iteraciones de transformación de forma) por GPU es mucho menor que el de la CPU.

## <a name="next-steps"></a>Pasos siguientes

- Obtenga más información sobre cómo [configurar la GPU para usar un módulo](azure-stack-edge-j-series-configure-gpu-modules.md).
