---
title: 'Tutorial: Implementación de un clasificador de Custom Vision en un dispositivo mediante Azure IoT Edge'
description: En este tutorial, aprenderá cómo hacer que un modelo de Computer Vision se ejecute como un contenedor con Custom Vision y IoT Edge.
services: iot-edge
author: kgremban
manager: philmea
ms.author: kgremban
ms.date: 07/30/2020
ms.topic: tutorial
ms.service: iot-edge
ms.custom: mvc
ms.openlocfilehash: 5d4b87c14422744fd62d42a4d8e5b1ca0f34ffac
ms.sourcegitcommit: 14bf4129a73de2b51a575c3a0a7a3b9c86387b2c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/30/2020
ms.locfileid: "87439729"
---
# <a name="tutorial-perform-image-classification-at-the-edge-with-custom-vision-service"></a>Tutorial: Clasificación de imágenes en el perímetro con Custom Vision Service

Azure IoT Edge puede hacer que su solución de IoT sea más eficaz al mover cargas de trabajo de la nube al perímetro. Esta funcionalidad es adecuada en servicios que procesan una gran cantidad de datos, como los modelos de Computer Vision. [Custom Vision Service](../cognitive-services/custom-vision-service/home.md) le permite crear clasificadores de imágenes personalizados e implementarlos en dispositivos como contenedores. Juntos, estos dos servicios permiten buscar información a partir de imágenes o secuencias de vídeo sin tener que transferir todos los datos fuera del sitio en primer lugar. Custom Vision proporciona un clasificador que compara una imagen con un modelo entrenado para generar información.

Por ejemplo, Custom Vision en un dispositivo IoT Edge podría determinar si una autopista está experimentando un tráfico mayor o menor de lo normal o si un estacionamiento tiene zonas de estacionamiento disponibles en una fila. Esta información se puede compartir con otro servicio para tomar medidas.

En este tutorial, aprenderá a:

> [!div class="checklist"]
>
> * Crear un clasificador de imágenes con Custom Vision.
> * Desarrollar un módulo de IoT Edge que realiza consultas al servidor web de Custom Vision en el dispositivo.
> * Enviar los resultados del clasificador de imágenes a IoT Hub.

<center>

![Diagrama: Tutorial sobre la arquitectura, las fases y la implementación de clasificador](./media/tutorial-deploy-custom-vision/custom-vision-architecture.png)
</center>

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Prerrequisitos

>[!TIP]
>Este tutorial es una versión simplificada del proyecto de ejemplo de [Custom Vision y Azure IoT Edge en un Raspberry Pi 3](https://github.com/Azure-Samples/Custom-vision-service-iot-edge-raspberry-pi). Este tutorial se ha diseñado para ejecutarse en una máquina virtual en la nube y usa imágenes estáticas para entrenar y probar el clasificador de imágenes, lo que resulta útil para cualquiera que acabe de empezar a evaluar Custom Vision en IoT Edge. El proyecto de ejemplo usa hardware físico y configura una cámara en directo para entrenar y probar el clasificador de imágenes, que es útil para cualquiera que desee probar un escenario real más detallado.

Antes de comenzar este tutorial es preciso haber realizado el anterior, en el que se configuraba el entorno para el desarrollo de contenedores en Linux: [Desarrollo de módulos IoT Edge para dispositivos Linux](tutorial-develop-for-linux.md). Al completar ese tutorial, se deben cumplir los siguientes requisitos previos:

* Una instancia de [IoT Hub](../iot-hub/iot-hub-create-through-portal.md) de nivel estándar o gratis en Azure.
* Un [dispositivo Linux que ejecute Azure IoT Edge](quickstart-linux.md).
* Un registro de contenedor, como [Azure Container Registry](https://docs.microsoft.com/azure/container-registry/).
* [Visual Studio Code](https://code.visualstudio.com/) configurado con [Azure IoT Tools](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-tools).
* [CE de Docker](https://docs.docker.com/install/) configurado para ejecutar contenedores de Linux.

Para desarrollar un módulo de IoT Edge con el servicio Custom Vision, instale los siguientes requisitos previos adicionales en la máquina de desarrollo:

* [Python](https://www.python.org/downloads/)
* [Git](https://git-scm.com/downloads)
* [Extensión de Python para Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=ms-python.python)

## <a name="build-an-image-classifier-with-custom-vision"></a>Creación de un clasificador de imágenes con Custom Vision

Para crear un clasificador de imágenes, deberá crear un proyecto de Custom Vision y proporcionar imágenes de entrenamiento. Para más información acerca de los pasos que se realizan en esta sección, consulte [Cómo crear un clasificador con Custom Vision](../cognitive-services/custom-vision-service/getting-started-build-a-classifier.md).

Una vez que se crea y se entrena el clasificador de imágenes, puede exportarlo como un contenedor de Docker e implementarlo en un dispositivo IoT Edge.

### <a name="create-a-new-project"></a>Creación de un nuevo proyecto

1. En el explorador web, vaya a la [página web de Custom Vision](https://customvision.ai/).

2. Seleccione **Sign in** (Iniciar sesión) e inicie sesión con la misma cuenta que usa para acceder a los recursos de Azure.

3. Seleccione **New project** (Nuevo proyecto).

4. Cree el proyecto con los valores siguientes:

   | Campo | Value |
   | ----- | ----- |
   | Nombre | Proporcione un nombre para el proyecto, como **EdgeTreeClassifier**. |
   | Descripción | Descripción opcional del proyecto. |
   | Resource | Seleccione uno de sus grupos de recursos de Azure que incluye un recurso de Custom Vision Service o **cree uno** si aún no ha agregado ninguno. |
   | Tipos de proyecto | **Clasificación** |
   | Classification Types (Tipos de clasificación) | **Multiclass (single tag per image)** (Multiclase [etiqueta única por imagen]) |
   | Dominios | **General (compact)** (General [compacto]) |
   | Funcionalidades de exportación | **Basic platforms (Tensorflow, CoreML, ONNX, ...)** (Plataformas básicas [Tensorflow, CoreML, ONNX,...]) |

5. Seleccione **Create project** (Crear proyecto).

### <a name="upload-images-and-train-your-classifier"></a>Carga de imágenes y entrenamiento del clasificador

La creación de un clasificador de imágenes requiere un conjunto de imágenes de entrenamiento, así como imágenes de prueba.

1. Puede clonar o descargar imágenes de ejemplo desde el repositorio [Cognitive-CustomVision-Windows](https://github.com/Microsoft/Cognitive-CustomVision-Windows) en el equipo de desarrollo local.

   ```cmd/sh
   git clone https://github.com/Microsoft/Cognitive-CustomVision-Windows.git
   ```

2. Vuelva al proyecto de Custom Vision y seleccione **Add images** (Agregar imágenes).

3. Busque el repositorio de git que ha clonado localmente y vaya a la primera carpeta de imágenes, **Cognitive-CustomVision-Windows / Samples / Images / Hemlock**. Seleccione las 10 imágenes de la carpeta y, a continuación, seleccione **Open** (Abrir).

4. Agregue la etiqueta **hemlock** (abeto) a este grupo de imágenes y presione **Entrar** para aplicar la etiqueta.

5. Seleccione **Upload 10 files** (Cargar 10 archivos).

   ![Carga de archivos con la etiqueta hemlock en Custom Vision](./media/tutorial-deploy-custom-vision/upload-hemlock.png)

6. Cuando las imágenes se hayan cargado correctamente, seleccione **Done** (Listo).

7. Seleccione **Add images** (Agregar imágenes) de nuevo.

8. Vaya a la segunda carpeta de imágenes, **Cognitive-CustomVision-Windows / Samples / Images / Japanese Cherry**. Seleccione las 10 imágenes de la carpeta y, a continuación, seleccione **Open** (Abrir).

9. Agregue la etiqueta **japanese cherry** (cerezo japonés) a este grupo de imágenes y presione **Entrar** para aplicar la etiqueta.

10. Seleccione **Upload 10 files** (Cargar 10 archivos). Cuando las imágenes se hayan cargado correctamente, seleccione **Done** (Listo).

11. Cuando ambos conjuntos de imágenes estén etiquetados y cargados, seleccione **Train** (Entrenar) para entrenar el clasificador.

### <a name="export-your-classifier"></a>Exportación del clasificador

1. Después de entrenar el clasificador, seleccione **Export** (Exportar) en la página Performance (Rendimiento) del clasificador.

   ![Exportación del clasificador de imágenes entrenado](./media/tutorial-deploy-custom-vision/export.png)

2. Seleccione **DockerFile** para la plataforma. 

3. Seleccione **Linux** para la versión.  

4. Seleccione **Export** (Exportar). 

   ![Exportación como archivo DockerFile con contenedores Linux](./media/tutorial-deploy-custom-vision/export-2.png)

5. Una vez finalizada la exportación, seleccione **Download** (Descargar) y guarde el paquete zip en el equipo. Extraiga todos los archivos del paquete. Estos archivos se usarán para crear un módulo de IoT Edge que contiene el servidor del clasificador de imágenes. 

En este punto, ha terminado de crear y entrenar su proyecto de Custom Vision. Los archivos exportados se utilizarán en la sección siguiente, pero ha terminado con la página web de Custom Vision. 

## <a name="create-an-iot-edge-solution"></a>Creación de una solución de IoT Edge

Ahora tiene los archivos de una versión en contenedor del clasificador de imágenes en el equipo de desarrollo. En esta sección, configurará el contenedor del clasificador de imágenes para su ejecución como un módulo de IoT Edge. También creará un segundo módulo que se implementará junto con el clasificador de imágenes. El segundo módulo envía solicitudes al clasificador y envía los resultados como mensajes a IoT Hub. 

### <a name="create-a-new-solution"></a>Creación de una nueva solución

Una solución es una manera lógica de desarrollar y organizar varios módulos para una sola implementación de IoT Edge. Una solución contiene código para uno o varios módulos, así como el manifiesto de implementación que declara cómo configurarlos en un dispositivo IoT Edge. 

1. En Visual Studio Code, seleccione **Ver** > **Paleta de comandos** para abrir la paleta de comandos de VS Code. 

1. En la paleta de comandos, escriba y ejecute el comando **Azure IoT Edge: New IoT Edge solution** (Azure IoT Edge: nueva solución de IoT Edge). En la paleta de comandos, proporcione la siguiente información para crear la solución: 

   | Campo | Value |
   | ----- | ----- |
   | Seleccionar carpeta | Elija la ubicación en el equipo de desarrollo en la que VS Code creará los archivos de la solución. |
   | Proporcionar un nombre de la solución | Escriba un nombre descriptivo para la solución, como **CustomVisionSolution** o acepte el valor predeterminado. |
   | Seleccionar plantilla del módulo | Elija **Módulo de Python**. |
   | Proporcionar un nombre de módulo | Asigne al módulo el nombre **classifier**.<br><br>Es importante que este nombre de módulo esté en minúsculas. IoT Edge distingue mayúsculas de minúsculas cuando se hace referencia a módulos y esta solución usa una biblioteca que da formato a todas las solicitudes en minúsculas. |
   | Proporcionar repositorio de imágenes de Docker del módulo | Un repositorio de imágenes incluye el nombre del registro de contenedor y el nombre de la imagen de contenedor. La imagen de contenedor se rellena previamente a partir del último paso. Reemplace **localhost:5000** por el valor de **Servidor de inicio de sesión** del registro de contenedor de Azure. Puede recuperar el servidor de inicio de sesión en la página de información general del registro de contenedor en Azure Portal.<br><br>Así es la cadena final: **\<registry name\>.azurecr.io/classifier**. |
 
   ![Especificación del repositorio de imágenes de Docker](./media/tutorial-deploy-custom-vision/repository.png)

El área de trabajo de la solución de IoT Edge se carga en la ventana de Visual Studio Code.

### <a name="add-your-registry-credentials"></a>Adición de las credenciales del Registro

El archivo del entorno almacena las credenciales del registro de contenedor y las comparte con el runtime de IoT Edge. El entorno de ejecución necesita estas credenciales para extraer las imágenes privadas e insertarlas en el dispositivo IoT Edge.

La extensión de IoT Edge intenta extraer de Azure las credenciales del registro del contenedor y rellenar con ellas el archivo de entorno. Compruebe si las credenciales ya están incluidas. Si no lo están, agréguelas ahora:

1. En el explorador de VS Code, abra el archivo .env.
2. Actualice los campos con los valores de **nombre de usuario** y **contraseña** que ha copiado del Registro de contenedor de Azure.
3. Guarde este archivo.

### <a name="select-your-target-architecture"></a>Selección de la arquitectura de destino

Actualmente, Visual Studio Code puede desarrollar módulos para dispositivos Linux AMD64 y Linux ARM32v7. Debe seleccionar qué arquitectura tiene como destino con cada solución, dado que el contenedor se crea y ejecuta de manera diferente para cada tipo de arquitectura. El valor predeterminado es Linux AMD64 y es la que vamos a usar en este tutorial. 

1. Abra la paleta de comandos y busque **Azure IoT Edge: Set Default Target Platform for Edge Solution** (Establecer la plataforma de destino predeterminada para la solución perimetral), o bien seleccione el icono de acceso directo en la barra lateral en la parte inferior de la ventana. 

2. En la paleta de comandos, seleccione la arquitectura de destino en la lista de opciones. Para este tutorial, usamos una máquina virtual Ubuntu como dispositivo IoT Edge, por lo que mantendrá el valor predeterminado **amd64**. 

### <a name="add-your-image-classifier"></a>Adición del clasificador de imágenes

La plantilla del módulo de Python en Visual Studio Code contiene código de ejemplo que se puede ejecutar para probar IoT Edge. No vamos a usar ese código en este escenario. En su lugar, utilice los pasos de esta sección para reemplazar el código de ejemplo por el contenedor del clasificador de imágenes que exportó anteriormente. 

1. En el Explorador de archivos, busque el paquete de Custom Vision que ha descargado y extraído. Copie todo el contenido del paquete extraído. Debería ser dos carpetas, **app** y **azureml** y dos archivos, **Dockerfile** y **README**. 

2. En el Explorador de archivos, vaya al directorio que indicó a Visual Studio Code para crear la solución de IoT Edge. 

3. Abra la carpeta del módulo clasificador. Si utilizó los nombres sugeridos en la sección anterior, la estructura de carpetas es similar a **CustomVisionSolution / modules / classifier**. 

4. Pegue los archivos en la carpeta **classifier**. 

5. Vuelva a la ventana de Visual Studio Code. El área de trabajo de la solución ahora debe mostrar los archivos del clasificador de imágenes en la carpeta del módulo. 

   ![Área de trabajo de la solución con los archivos del clasificador de imágenes](./media/tutorial-deploy-custom-vision/workspace.png)

6. Abra el archivo **module.json** de la carpeta de clasificador. 

7. Actualice el parámetro **platforms** para que apunte al nuevo Dockerfile que ha agregado y quite todas las opciones que no sean AMD64, que es la única arquitectura que vamos a usar para este tutorial. 

   ```json
   "platforms": {
       "amd64": "./Dockerfile"
   }
   ```

8. Guarde los cambios. 

### <a name="create-a-simulated-camera-module"></a>Creación de un módulo de cámara simulada

En una implementación real de Custom Vision, tendría una cámara que proporciona imágenes en directo o secuencias de vídeo. En este escenario, se simula la cámara mediante la creación de un módulo que envía una imagen de prueba al clasificador de imágenes. 

#### <a name="add-and-configure-a-new-module"></a>Adición y configuración de un nuevo módulo

En esta sección, se agrega un nuevo módulo a la misma solución CustomVisionSolution y se proporciona código para crear la cámara simulada. 

1. En la misma ventana de Visual Studio Code, use la paleta de comandos para ejecutar **Azure IoT Edge: Add IoT Edge Module** (Azure IoT Edge: agregar módulo IoT Edge). En la paleta de comandos, proporcione la siguiente información para el nuevo módulo: 

   | Prompt | Value | 
   | ------ | ----- |
   | Seleccionar archivo de la plantilla de implementación | Seleccione el archivo **deployment.template.json** en la carpeta CustomVisionSolution. |
   | Seleccionar plantilla del módulo | Seleccione **Módulo de Python** |
   | Proporcionar un nombre de módulo | Asigne al módulo el nombre **cameraCapture** |
   | Proporcionar repositorio de imágenes de Docker del módulo | Reemplace **localhost:5000** por el valor de **Servidor de inicio de sesión** del registro de contenedor de Azure.<br><br>La cadena final es como esta: **\<registryname\>.azurecr.io/cameracapture**. |

   La ventana de VS Code carga el módulo nuevo en el área de trabajo de la solución y actualiza el archivo deployment.template.json. Ahora debería ver dos carpetas de módulo: classifier y cameraCapture. 

2. Abra el archivo **main.py** de la carpeta **modules** / **cameraCapture**. 

3. Reemplace todo el archivo por el código siguiente. Este código de ejemplo envía solicitudes POST al servicio de procesamiento de imágenes que se ejecuta en el módulo clasificador. Se proporciona este contenedor de módulo con una imagen de ejemplo para usar en las solicitudes. A continuación, se empaqueta la respuesta como un mensaje de IoT Hub y se envía a una cola de salida.  

    ```python
    # Copyright (c) Microsoft. All rights reserved.
    # Licensed under the MIT license. See LICENSE file in the project root for
    # full license information.

    import time
    import sys
    import os
    import requests
    import json
    from azure.iot.device import IoTHubModuleClient, Message

    # global counters
    SENT_IMAGES = 0

    # global client
    CLIENT = None

    # Send a message to IoT Hub
    # Route output1 to $upstream in deployment.template.json
    def send_to_hub(strMessage):
        message = Message(bytearray(strMessage, 'utf8'))
        CLIENT.send_message_to_output(message, "output1")
        global SENT_IMAGES
        SENT_IMAGES += 1
        print( "Total images sent: {}".format(SENT_IMAGES) )

    # Send an image to the image classifying server
    # Return the JSON response from the server with the prediction result
    def sendFrameForProcessing(imagePath, imageProcessingEndpoint):
        headers = {'Content-Type': 'application/octet-stream'}

        with open(imagePath, mode="rb") as test_image:
            try:
                response = requests.post(imageProcessingEndpoint, headers = headers, data = test_image)
                print("Response from classification service: (" + str(response.status_code) + ") " + json.dumps(response.json()) + "\n")
            except Exception as e:
                print(e)
                print("No response from classification service")
                return None

        return json.dumps(response.json())

    def main(imagePath, imageProcessingEndpoint):
        try:
            print ( "Simulated camera module for Azure IoT Edge. Press Ctrl-C to exit." )

            try:
                global CLIENT
                CLIENT = IoTHubModuleClient.create_from_edge_environment()
            except Exception as iothub_error:
                print ( "Unexpected error {} from IoTHub".format(iothub_error) )
                return

            print ( "The sample is now sending images for processing and will indefinitely.")

            while True:
                classification = sendFrameForProcessing(imagePath, imageProcessingEndpoint)
                if classification:
                    send_to_hub(classification)
                time.sleep(10)

        except KeyboardInterrupt:
            print ( "IoT Edge module sample stopped" )

    if __name__ == '__main__':
        try:
            # Retrieve the image location and image classifying server endpoint from container environment
            IMAGE_PATH = os.getenv('IMAGE_PATH', "")
            IMAGE_PROCESSING_ENDPOINT = os.getenv('IMAGE_PROCESSING_ENDPOINT', "")
        except ValueError as error:
            print ( error )
            sys.exit(1)

        if ((IMAGE_PATH and IMAGE_PROCESSING_ENDPOINT) != ""):
            main(IMAGE_PATH, IMAGE_PROCESSING_ENDPOINT)
        else: 
            print ( "Error: Image path or image-processing endpoint missing" )
    ```

4. Guarde el archivo **main.py**. 

5. Abra el archivo **requirements.txt**. 

6. Agregue una nueva línea para una biblioteca que se va a incluir en el contenedor.

   ```Text
   requests
   ```

7. Guarde el archivo **requirements.txt** .


#### <a name="add-a-test-image-to-the-container"></a>Adición de una imagen de prueba al contenedor

En lugar de usar una cámara real para proporcionar fuente de imágenes, en este escenario vamos a usar una única imagen de prueba. En el repositorio de GitHub que descargó anteriormente en este tutorial para las imágenes de entrenamiento se incluye una imagen de prueba. 

1. Vaya a la imagen de prueba, ubicada en **Cognitive-CustomVision-Windows** / **Samples** / **Images** / **Test**. 

2. Copie **test_image.jpg** 

3. Vaya al directorio de la solución de IoT Edge y pegue la imagen de prueba en la carpeta **modules** / **cameraCapture**. La imagen debe estar en la misma carpeta que el archivo main.py que modificó en la sección anterior. 

4. En Visual Studio Code, abra el archivo **Dockerfile.amd64** del módulo cameraCapture.

5. Después de la línea que establece el directorio de trabajo, `WORKDIR /app`, agregue la siguiente línea de código:

   ```Dockerfile
   ADD ./test_image.jpg .
   ```

6. Guarde el archivo Dockerfile.

### <a name="prepare-a-deployment-manifest"></a>Preparación de un manifiesto de implementación

Hasta el momento, en este tutorial ha entrenado un modelo de Custom Vision para clasificar imágenes de árboles y ha empaquetado dicho modelo como un módulo de IoT Edge. A continuación, ha creado un segundo módulo que puede consultar el servidor del clasificador de imágenes y notificar sus resultados a IoT Hub. Ahora, está listo para crear el manifiesto de implementación que indicará a un dispositivo IoT Edge cómo iniciar y ejecutar estos dos módulos juntos. 

La extensión de IoT Edge para Visual Studio Code proporciona una plantilla en cada solución de IoT Edge para ayudarle a crear un manifiesto de implementación. 

1. Abra el archivo **deployment.template.json** de la carpeta de la solución. 

2. Busque la sección **modules**, que debe contener tres módulos: los dos que ha creado, classifier y cameraCapture y un tercero que se incluye de forma predeterminada, SimulatedTemperatureSensor. 

3. Elimine el módulo **SimulatedTemperatureSensor** con todos sus parámetros. Este módulo se incluye para proporcionar datos de ejemplo para escenarios de prueba, pero no se necesita en esta implementación. 

4. Si ha asignado al módulo del clasificador de imágenes un nombre distinto de **classifier**, compruebe el nombre ahora y asegúrese de que está todo en minúsculas. El módulo cameraCapture llama al módulo classifier mediante una biblioteca de solicitudes que da formato a todas las solicitudes en minúsculas e IoT Edge distingue mayúsculas de minúsculas. 

5. Actualice el parámetro **createOptions** del módulo cameraCapture con el siguiente código JSON. Esta información crea variables de entorno en el contenedor del módulo que se recuperan en el proceso main.py. Al incluir esta información en el manifiesto de implementación, puede cambiar la imagen o el punto de conexión sin tener que volver a generar la imagen del módulo. 

    ```json
    "createOptions": "{\"Env\":[\"IMAGE_PATH=test_image.jpg\",\"IMAGE_PROCESSING_ENDPOINT=http://classifier/image\"]}"
    ```

    Si ha asignado al módulo de Custom Vision un nombre distinto de *classifier*, actualice el valor del punto de conexión de procesamiento de imágenes para que coincida. 

6. En la parte inferior del archivo, actualice el parámetro **routes** del módulo $edgeHub. Se desea enrutar los resultados de la predicción desde cameraCapture a IoT Hub.

    ```json
        "routes": {
          "cameraCaptureToIoTHub": "FROM /messages/modules/cameraCapture/outputs/* INTO $upstream"
        },
    ```

    Si ha asignado al segundo módulo un nombre distinto de *cameraCapture*, actualice el valor de la ruta para que coincida. 

7. Guarde el archivo **deployment.template.json**.

## <a name="build-and-push-your-iot-edge-solution"></a>Compilación e inserción de una solución IoT Edge

Con ambos módulos creados y la plantilla del manifiesto de implementación configurada, está listo para crear las imágenes de contenedor e insertarlas en el registro de contenedor.

Una vez que las imágenes se encuentran en el registro, puede implementar la solución en un dispositivo IoT Edge. Puede establecer módulos en un dispositivo con IoT Hub, pero también puede acceder a este y a los dispositivos mediante Visual Studio Code. En esta sección puede configurar el acceso a IoT Hub y, posteriormente, usar VS Code para implementar la solución en el dispositivo IoT Edge.

En primer lugar, compile e inserte la solución en el registro de contenedor.

1. Abra el terminal integrado de VS Code, para lo que debe seleccionar **View** > **Terminal** (Ver > Terminal).

2. Escriba el comando siguiente en el terminal para iniciar sesión en Docker. Inicie sesión con el nombre de usuario, la contraseña y el servidor de inicio de sesión de Azure Container Registry. Puede recuperar estos valores en la sección **Claves de acceso** del Registro en Azure Portal.

   ```bash
   docker login -u <ACR username> -p <ACR password> <ACR login server>
   ```

   Puede recibir una advertencia de seguridad en la que se recomiende el uso de `--password-stdin`. Aunque ese procedimiento se recomienda para escenarios de producción, está fuera del ámbito de este tutorial. Para más información, consulte la referencia de [docker login](https://docs.docker.com/engine/reference/commandline/login/#provide-a-password-using-stdin).

3. En el explorador de VS Code, haga clic con el botón derecho en el archivo **deployment.template.json** y seleccione **Build and Push IoT Edge solution** (Compilar e insertar solución de IoT Edge).

   El comando de compilación e inserción inicia tres operaciones. En primer lugar, se crea una nueva carpeta en la solución llamada **config**, que contiene los archivos del manifiesto de la implementación completa, con la información de la plantilla de implementación y otros archivos de la solución. En segundo lugar, ejecuta `docker build` para generar la imagen de contenedor basándose en el Dockerfile adecuado para la arquitectura de destino. A continuación, ejecuta `docker push` para insertar el repositorio de imágenes en el registro de contenedor.

   Este proceso puede tardar varios minutos la primera vez, pero es más rápido la próxima vez que ejecute los comandos.

## <a name="deploy-modules-to-device"></a>Implementación de módulos en el dispositivo

Utilice el explorador de Visual Studio Code y la extensión Azure IoT Tools para implementar el proyecto de módulo en el dispositivo IoT Edge. Ya tiene un manifiesto de implementación preparado para su escenario, el archivo **deployment.amd64.json** de la carpeta config. Ahora todo lo que necesita hacer es seleccionar un dispositivo que reciba la implementación.

Asegúrese de que el dispositivo IoT Edge está en funcionamiento.

1. En el explorador de Visual Studio Code, en la sección **Azure IoT Hub**, expanda **Dispositivos** para ver la lista de dispositivos IoT.

2. Haga clic con el botón derecho en el nombre del dispositivo IoT Edge y seleccione **Create Deployment for IoT Edge device** (Crear una implementación para un dispositivo individual).

3. Seleccione el archivo **deployment.amd64.json** en la carpeta **config** y, a continuación, haga clic en **Select Edge Deployment Manifest** (Seleccionar manifiesto de implementación de Edge). No utilice el archivo deployment.template.json.

4. En el dispositivo, expanda **Módulos** para ver una lista de módulos implementados y en ejecución. Haga clic en el botón Actualizar. Debería ver que los nuevos módulos **classifier** y **cameraCapture** se ejecutan junto con **$edgeAgent** y **$edgeHub**.  

También puede comprobar que todos los módulos están activados y en ejecución en el propio dispositivo. En el dispositivo IoT Edge, ejecute el siguiente comando para ver el estado de los módulos.

   ```bash
   iotedge list
   ```

Los módulos pueden tardar unos minutos en iniciarse. El entorno de ejecución de Azure IoT Edge necesita recibir su nuevo manifiesto de implementación, extraer las imágenes de los módulos del entorno de ejecución del contenedor y, después, iniciar cada nuevo módulo.

## <a name="view-classification-results"></a>Visualización de los resultados de la clasificación

Hay dos maneras de ver los resultados de los módulos, ya sea en el propio dispositivo, a medida que se generan y envían los mensajes o desde Visual Studio Code, cuando los mensajes llegan a IoT Hub. 

Desde el dispositivo, puede ver los registros del módulo cameraCapture para ver los mensajes enviados y la confirmación de que se recibieron en IoT Hub. 

   ```bash
   iotedge logs cameraCapture
   ```

En Visual Studio Code, haga clic con el botón derecho en el nombre del dispositivo IoT Edge y seleccione **Start Monitoring Built-in Event Endpoint** (Iniciar supervisión del punto de conexión del evento integrado). 

> [!NOTE]
> Al principio es posible que vea ciertos errores de conexión en la salida del módulo cameraCapture. Esto se debe al retraso que se produce entre la implementación y el inicio de los módulos.
>
> El módulo cameraCapture reintentará automáticamente la conexión hasta que se realice correctamente. Después, debería empezar a ver los mensajes de clasificación de imágenes esperados que se describen a continuación.

Los resultados del módulo de Custom Vision, que se envían como mensajes desde el módulo cameraCapture, incluyen la probabilidad de que la imagen sea de un abeto o un cerezo. Dado que la imagen es un abeto, debería ver la probabilidad como 1,0.

## <a name="clean-up-resources"></a>Limpieza de recursos

Si prevé seguir con el siguiente artículo recomendado, puede mantener los recursos y las configuraciones que ya ha creado y volverlos a utilizar. También puede seguir usando el mismo dispositivo de IoT Edge como dispositivo de prueba. 

En caso contrario, para evitar gastos, puede eliminar las configuraciones locales y los recursos de Azure que creó en este artículo. 

[!INCLUDE [iot-edge-clean-up-cloud-resources](../../includes/iot-edge-clean-up-cloud-resources.md)]

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha entrenado un modelo de Custom Vision y lo ha implementado como un módulo en un dispositivo IoT Edge. A continuación, ha creado un módulo que puede consultar el servicio del clasificador de imágenes y notificar sus resultados a IoT Hub. 

Si desea probar una versión más detallada de este escenario con una fuente de cámara en directo, consulte el proyecto de GitHub [Custom Vision y Azure IoT Edge en un dispositivo Raspberry Pi 3](https://github.com/Azure-Samples/Custom-vision-service-iot-edge-raspberry-pi). 

Puede continuar con los siguientes tutoriales para aprender otras formas en las que Azure IoT Edge puede ayudarle a convertir los datos en información empresarial en el perímetro.

> [!div class="nextstepaction"]
> [Almacenamiento de datos en el perímetro con bases de datos de SQL Server](tutorial-store-data-sql-server.md)
