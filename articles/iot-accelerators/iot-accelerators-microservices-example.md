---
title: 'Cambio y reimplementación de un microservicio: Azure | Microsoft Docs'
description: En este tutorial se explica cómo cambiar y reimplementar un microservicio en la solución de supervisión remota.
author: dominicbetts
ms.author: dobett
ms.service: iot-accelerators
services: iot-accelerators
ms.date: 04/19/2018
ms.topic: conceptual
ms.custom: devx-track-csharp
ms.openlocfilehash: 9ff3e12721628e244f247e174af101e71ea91191
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "88998329"
---
# <a name="customize-and-redeploy-a-microservice"></a>Personalización y reimplementación de un microservicio

En este tutorial se explica cómo editar uno de los [microservicios](https://azure.com/microservices) en la solución remota, crear una imagen del microservicio, implementar la imagen en el centro de Docker y luego usarla en la solución de supervisión remota. Para introducir este concepto, el tutorial usa un escenario básico donde llamar a una API de microservicio y cambiar el mensaje de estado de "Activo y correcto" a "Las nuevas modificaciones se hacen aquí".

La solución de supervisión remota usa los microservicios compilados con imágenes de Docker extraídas de un centro de Docker. 

En este tutorial, aprenderá a:

>[!div class="checklist"]
> * Editar y compilar un microservicio en la solución de supervisión remota
> * Crear una imagen de Docker
> * Insertar una imagen de Docker en un centro de Docker
> * Extraer una imagen de Docker nueva
> * Visualizar los cambios 

## <a name="prerequisites"></a>Prerequisites

Para utilizar este tutorial, necesitará:

>[!div class="checklist"]
> * [Implementación local del acelerador de soluciones de supervisión remota](iot-accelerators-remote-monitoring-deploy-local.md)
> * [Una cuenta de Docker](https://hub.docker.com/)
> * [Postman](https://www.getpostman.com/): necesario para ver la respuesta de la API

## <a name="call-the-api-and-view-response-status"></a>Llamada API y visualización del estado de la respuesta

En esta parte, llame a la API de microservicio del administrador de IoT Hub predeterminada. La API devuelve un mensaje de estado que se cambia más adelante mediante la personalización del microservicio.

1. Asegúrese de que la solución de supervisión remota se ejecuta localmente en su equipo.
2. Busque el entorno de Postman descargado y ábralo.
3. En Postman, escriba lo siguiente en GET: `http://localhost:8080/iothubmanager/v1/status`.
4. Verá el valor devuelto y también debe ver "Estado": "Activo y correcto".

    ![Mensaje de activo y correcto en Postman](./media/iot-accelerators-microservices-example/postman-alive-well.png)

## <a name="change-the-status-and-build-the-image"></a>Cambio de estado y creación de la imagen

Ahora cambie el mensaje de estado del microservicio del administrador de IoT Hub a "Las nuevas modificaciones se hacen aquí" y luego recompile la imagen de Docker con este nuevo estado. Si experimenta problemas aquí, vea la sección [Solución de problemas](#Troubleshoot).

1. Asegúrese de que el terminal esté abierto y cambie al directorio en que clonó la solución de supervisión remota. 
1. Cambie el directorio a "azure-iot-pcs-remote-monitoring-dotnet/services/iothub-manager/Services".
1. Abra StatusService.cs en el editor de texto o el IDE que le guste. 
1. Busque el código siguiente:

    ```csharp
    var result = new StatusServiceModel(true, "Alive and well!");
    ```

    y cámbielo por el código siguiente y guárdelo.

    ```csharp
    var result = new StatusServiceModel(true, "New Edits Made Here!");
    ```

5. Vuelva al terminal, pero ahora cambie al siguiente directorio: "azure-iot-pcs-remote-monitoring-dotnet/services/iothub-manager/scripts/docker".
6. Para compilar la nueva imagen de Docker, escriba

    ```sh
    sh build
    ```
    
    o en Windows:
    
    ```cmd
    ./build.cmd
    ```

7. Para verificar que la nueva imagen se creó correctamente, escriba

    ```cmd/sh
    docker images 
    ```

El repositorio debe ser "azureiotpcs/iothub-manager-dotnet".

![Imagen de Docker correcta](./media/iot-accelerators-microservices-example/successful-docker-image.png)

## <a name="tag-and-push-the-image"></a>Etiquetado e inserción de la imagen
Antes de poder insertar una nueva imagen de Docker en un centro de Docker, Docker espera el etiquetado de las imágenes. Si experimenta problemas aquí, vea la sección [Solución de problemas](#Troubleshoot).

1. Busque el identificador de la imagen de Docker creada; para ello, escriba:

    ```cmd/sh
    docker images
    ```

2. Para etiquetar la imagen con el tipo "prueba"

    ```cmd/sh
    docker tag [Image ID] [docker ID]/iothub-manager-dotnet:testing 
    ```

3. Para insertar la imagen recién etiquetada con el centro de Docker, escriba

    ```cmd/sh
    docker push [docker ID]/iothub-manager-dotnet:testing
    ```

4. Abra el explorador de Internet y vaya al [centro de Docker](https://hub.docker.com/) e inicie sesión.
5. Ahora debería ver la imagen de Docker recién insertada en el centro de Docker.
![Imagen de Docker en el centro de Docker](./media/iot-accelerators-microservices-example/docker-image-in-docker-hub.png)

## <a name="update-your-remote-monitoring-solution"></a>Actualización de la solución de supervisión remota
Debe actualizar el archivo local docker-compose.yml para extraer la imagen de Docker nueva del centro de Docker. Si experimenta problemas aquí, vea la sección [Solución de problemas](#Troubleshoot).

1. Vuelva al terminal y cambie al siguiente directorio: "azure-iot-pcs-remote-monitoring-dotnet/services/scripts/local".
2. Abra docker-compose.yml en el editor de texto o el IDE que le guste.
3. Busque el código siguiente:

    ```yml
    image: azureiotpcs/iothub-manager-dotnet:testing
    ```

    y cámbielo para que sea similar a la imagen siguiente y después guárdelo.

    ```yml
    image: [docker ID]/iothub-manager-dotnet:testing
    ```

## <a name="view-the-new-response-status"></a>Visualización del nuevo estado de respuesta
Para terminar, reimplemente una instancia local de la solución de supervisión remota y vea la nueva respuesta de estado en Postman.

1. Vuelva al terminal y cambie al siguiente directorio: "azure-iot-pcs-remote-monitoring-dotnet/scripts/local".
2. Inicie la instancia local de la solución de supervisión remota; para ello, escriba el siguiente comando en el terminal:

    ```cmd/sh
    docker-compose up
    ```

3. Busque el entorno de Postman descargado y ábralo.
4. En Postman, escriba la siguiente solicitud en GET: `http://localhost:8080/iothubmanager/v1/status`. Ahora debería ver "Estado": "Correcto. Las nuevas modificaciones se hacen aquí".

![Mensaje de Postman Las nuevas modificaciones se hacen aquí](./media/iot-accelerators-microservices-example/new-postman-message.png)

## <a name="troubleshoot"></a><a name="Troubleshoot"></a>Solución de problemas

Si experimenta problemas, intente eliminar las imágenes y los contenedores de Docker del equipo local.

1. Para quitar todos los contenedores, primero debe detener todos los contenedores en ejecución. Abra el terminal y escriba

    ```cmd/sh
    docker stop $(docker ps -aq)
    docker rm $(docker ps -aq)
    ```
    
2. Para quitar todas las imágenes, abra el terminal y escriba 

    ```cmd/sh
    docker rmi $(docker images -q)
    ```

3. Puede comprobar si hay algún contenedor en la máquina; para ello, escriba

    ```cmd/sh
    docker ps -aq 
    ```

    Si se han quitado correctamente todos los contenedores, no debería aparecer nada.

4. Puede comprobar si hay alguna imagen en la máquina; para ello, escriba

    ```cmd/sh
    docker images
    ```

    Si se han quitado correctamente todos los contenedores, no debería aparecer nada.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, aprenderá a:

<!-- Repeat task list from intro -->
>[!div class="checklist"]
> * Editar y compilar un microservicio en la solución de supervisión remota
> * Crear una imagen de Docker
> * Insertar una imagen de Docker en un centro de Docker
> * Extraer una imagen de Docker nueva
> * Visualizar los cambios 

Lo siguiente que hay que probar es [personalizar el microservicio del simulador de dispositivos en la solución de supervisión remota](iot-accelerators-microservices-example.md).

Para más información sobre la solución de supervisión remota para los desarrolladores, consulte:

* [Guía de referencia para desarrolladores](https://github.com/Azure/azure-iot-pcs-remote-monitoring-dotnet/wiki/Developer-Reference-Guide)
<!-- Next tutorials in the sequence -->

