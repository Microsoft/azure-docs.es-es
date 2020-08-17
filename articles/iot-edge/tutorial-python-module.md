---
title: 'Creación del módulo personalizado de Python: Azure IoT Edge | Microsoft Docs'
description: En este tutorial, se explica cómo se crea un módulo IoT Edge con código Python y cómo se implementa en un dispositivo perimetral.
services: iot-edge
author: shizn
manager: philmea
ms.reviewer: kgremban
ms.author: xshi
ms.date: 08/04/2020
ms.topic: tutorial
ms.service: iot-edge
ms.custom: mvc
ms.openlocfilehash: 46e5a2119f05f3b4062044533352a6976a4ae4d8
ms.sourcegitcommit: 85eb6e79599a78573db2082fe6f3beee497ad316
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/05/2020
ms.locfileid: "87808788"
---
# <a name="tutorial-develop-and-deploy-a-python-iot-edge-module-for-linux-devices"></a>Tutorial: Desarrollo e implementación de un módulo de Python de IoT Edge para dispositivos Linux

Use Visual Studio Code para desarrollar código de Python e implementarlo en un dispositivo Linux que ejecute Azure IoT Edge.

Los módulos Azure IoT Edge se pueden usar para implementar código que, a su vez, implementa una lógica de negocios directamente en los dispositivos IoT Edge. En este tutorial, se detallan los pasos para crear e implementar un módulo de IoT Edge que filtra los datos de sensor en el dispositivo IoT Edge que configuró en el inicio rápido. En este tutorial, aprenderá a:

> [!div class="checklist"]
>
> * Usar Visual Studio Code para crear un módulo de Python para IoT Edge.
> * Utilizar Visual Studio Code y Docker para crear una imagen de Docker y publicarla en el Registro.
> * Implementar el módulo en el dispositivo IoT Edge.
> * Ver datos generados.

El módulo IoT Edge que creó en este tutorial filtra lo datos sobre la temperatura generados por el dispositivo. Solo envía mensajes a los niveles superiores si la temperatura sobrepasa el umbral especificado. Este tipo de análisis perimetral resulta útil para reducir la cantidad de datos que se comunican a la nube y se almacenan en ella.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="solution-scope"></a>Ámbito de la solución

Este tutorial muestra cómo desarrollar un módulo en **Python** mediante **Visual Studio Code** y cómo se implementa en un **dispositivo Linux**. IoT Edge no admite módulos de Python para dispositivos Windows.

Utilice la tabla siguiente si desea conocer las opciones para desarrollar e implementar módulos de Python en Linux:

| Python | Visual Studio Code | Visual Studio 2017/2019 |
| - | ------------------ | ------------------ |
| **Linux AMD64** | ![Usar VS Code para los módulos de Python en Linux AMD64](./media/tutorial-c-module/green-check.png) |  |
| **Linux ARM32** | ![Usar VS Code para los módulos de Python en Linux ARM32](./media/tutorial-c-module/green-check.png) |  |

## <a name="prerequisites"></a>Prerrequisitos

Antes de comenzar este tutorial, debe haber realizado el anterior para configurar el entorno de desarrollo de contenedores de Linux: [Desarrollo de módulos IoT Edge para dispositivos Linux](tutorial-develop-for-linux.md). Al completar ese tutorial, se deben cumplir los siguientes requisitos previos:

* Una instancia de [IoT Hub](../iot-hub/iot-hub-create-through-portal.md) de nivel estándar o gratis en Azure.
* Un [dispositivo Linux que ejecute Azure IoT Edge](quickstart-linux.md).
* Un registro de contenedor, como [Azure Container Registry](https://docs.microsoft.com/azure/container-registry/).
* [Visual Studio Code](https://code.visualstudio.com/) configurado con [Azure IoT Tools](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-tools).
* [CE de Docker](https://docs.docker.com/install/) configurado para ejecutar contenedores de Linux.

Para desarrollar un módulo de IoT Edge en Python, instale los siguientes requisitos previos adicionales en la máquina de desarrollo:

* [Extensión de Python](https://marketplace.visualstudio.com/items?itemName=ms-python.python) para Visual Studio Code.
* [Python](https://www.python.org/downloads/)
* [Pip](https://pip.pypa.io/en/stable/installing/#installation) para instalar paquetes de Python (normalmente, se incluye con la instalación de Python).

>[!Note]
>Asegúrese de que su carpeta `bin` se encuentra en la ruta de acceso de su plataforma. Normalmente `~/.local/` para UNIX y macOS, o `%APPDATA%\Python` en Windows.

## <a name="create-a-module-project"></a>Creación de un proyecto de módulo

En los siguientes pasos creará un módulo de Python para IoT Edge mediante Visual Studio Code y Azure IoT Tools.

### <a name="create-a-new-project"></a>Creación de un nuevo proyecto

Cree una plantilla de solución de Python que pueda personalizar con su propio código.

1. En Visual Studio Code, seleccione **Ver** > **Paleta de comandos** para abrir la paleta de comandos de VS Code.

2. En la paleta de comandos, escriba y ejecute el comando **Azure: Sign in** (Azure: iniciar sesión) y siga las instrucciones para iniciar sesión en la cuenta de Azure. Si ya ha iniciado sesión, puede omitir este paso.

3. En la paleta de comandos, escriba y ejecute el comando **Azure IoT Edge: New IoT Edge solution** (Azure IoT Edge: nueva solución de IoT Edge). Siga los mensajes y proporcione la siguiente información para crear la solución:

   | Campo | Value |
   | ----- | ----- |
   | Seleccionar carpeta | Elija la ubicación en el equipo de desarrollo en la que VS Code creará los archivos de la solución. |
   | Proporcionar un nombre de la solución | Escriba un nombre descriptivo para la solución o acepte el valor predeterminado **EdgeSolution**. |
   | Seleccionar plantilla del módulo | Elija **Módulo de Python**. |
   | Proporcionar un nombre de módulo | Llame al módulo **PythonModule**. |
   | Proporcionar repositorio de imágenes de Docker del módulo | Un repositorio de imágenes incluye el nombre del registro de contenedor y el nombre de la imagen de contenedor. La imagen de contenedor se rellena previamente con el nombre que proporcionó en el último paso. Reemplace **localhost:5000** por el valor de **Servidor de inicio de sesión** del registro de contenedor de Azure. Puede recuperar el servidor de inicio de sesión en la página de información general del registro de contenedor en Azure Portal. <br><br>El repositorio de imágenes final es similar a este: \<registry name\>.azurecr.io/pythonmodule. |

   ![Especificación del repositorio de imágenes de Docker](./media/tutorial-python-module/repository.png)

### <a name="add-your-registry-credentials"></a>Adición de las credenciales del Registro

El archivo del entorno almacena las credenciales del repositorio de contenedores y las comparte con el entorno de ejecución de IoT Edge. El entorno de ejecución necesita estas credenciales para extraer las imágenes privadas e insertarlas en el dispositivo IoT Edge.

La extensión de IoT Edge intenta extraer de Azure las credenciales del registro del contenedor y rellenar con ellas el archivo de entorno. Compruebe si las credenciales ya están incluidas. Si no lo están, agréguelas ahora:

1. En el explorador de VS Code, abra el archivo **.env**.
2. Actualice los campos con los valores de **nombre de usuario** y **contraseña** que ha copiado del Registro de contenedor de Azure.
3. Guarde el archivo .env.

### <a name="select-your-target-architecture"></a>Selección de la arquitectura de destino

Actualmente, Visual Studio Code puede desarrollar módulos de Python para dispositivos Linux AMD64 y Linux ARM32v7. Debe seleccionar qué arquitectura tiene como destino con cada solución, dado que el contenedor se crea y ejecuta de manera diferente para cada tipo de arquitectura. La opción predeterminada es Linux AMD64.

1. Abra la paleta de comandos y busque **Azure IoT Edge: Set Default Target Platform for Edge Solution** (Establecer la plataforma de destino predeterminada para la solución perimetral), o bien seleccione el icono de acceso directo en la barra lateral en la parte inferior de la ventana.

2. En la paleta de comandos, seleccione la arquitectura de destino en la lista de opciones. Para este tutorial, usamos una máquina virtual Ubuntu como dispositivo IoT Edge, por lo que mantendrá el valor predeterminado **amd64**.

### <a name="update-the-module-with-custom-code"></a>Actualización del módulo con código personalizado

Cada plantilla incluye un código de ejemplo, que toma los datos del sensor simulado del módulo **SimulatedTemperatureSensor** y los enruta al centro de IoT. En esta sección, agregue el código que expande **PythonModule** para analizar los mensajes antes de enviarlos.

1. En el explorador de VS Code, abra **módulos**  > **PythonModule** > **main.py**.

2. En la parte superior del archivo **main.py**, importe la biblioteca **json**:

    ```python
    import json
    ```

3. Agregue las variables **TEMPERATURE_THRESHOLD** y **TWIN_CALLBACKS** bajo los contadores globales. El umbral de temperatura establece el valor que debe superar la temperatura de la máquina para que los datos se envíen al centro de IoT.

    ```python
    # global counters
    TEMPERATURE_THRESHOLD = 25
    TWIN_CALLBACKS = 0
    RECEIVED_MESSAGES = 0
    ```

4. Reemplace la función **input1_listener** por el código siguiente:

    ```python
        # Define behavior for receiving an input message on input1
        # Because this is a filter module, we forward this message to the "output1" queue.
        async def input1_listener(module_client):
            global RECEIVED_MESSAGES
            global TEMPERATURE_THRESHOLD
            while True:
                try:
                    input_message = await module_client.receive_message_on_input("input1")  # blocking call
                    message = input_message.data
                    size = len(message)
                    message_text = message.decode('utf-8')
                    print ( "    Data: <<<%s>>> & Size=%d" % (message_text, size) )
                    custom_properties = input_message.custom_properties
                    print ( "    Properties: %s" % custom_properties )
                    RECEIVED_MESSAGES += 1
                    print ( "    Total messages received: %d" % RECEIVED_MESSAGES )
                    data = json.loads(message_text)
                    if "machine" in data and "temperature" in data["machine"] and data["machine"]["temperature"] > TEMPERATURE_THRESHOLD:
                        custom_properties["MessageType"] = "Alert"
                        print ( "Machine temperature %s exceeds threshold %s" % (data["machine"]["temperature"], TEMPERATURE_THRESHOLD))
                        await module_client.send_message_to_output(input_message, "output1")
                except Exception as ex:
                    print ( "Unexpected error in input1_listener: %s" % ex )

        # twin_patch_listener is invoked when the module twin's desired properties are updated.
        async def twin_patch_listener(module_client):
            global TWIN_CALLBACKS
            global TEMPERATURE_THRESHOLD
            while True:
                try:
                    data = await module_client.receive_twin_desired_properties_patch()  # blocking call
                    print( "The data in the desired properties patch was: %s" % data)
                    if "TemperatureThreshold" in data:
                        TEMPERATURE_THRESHOLD = data["TemperatureThreshold"]
                    TWIN_CALLBACKS += 1
                    print ( "Total calls confirmed: %d\n" % TWIN_CALLBACKS )
                except Exception as ex:
                    print ( "Unexpected error in twin_patch_listener: %s" % ex )
    ```

5. Actualice los **clientes de escucha** para escuchar también las actualizaciones gemelas.

    ```python
        # Schedule task for C2D Listener
        listeners = asyncio.gather(input1_listener(module_client), twin_patch_listener(module_client))

        print ( "The sample is now waiting for messages. ")
    ```

6. Guarde el archivo main.py.

7. En el explorador de VS Code, abra el archivo **deployment.template.json** en el área de trabajo de la solución de IoT Edge.

8. Agregue el módulo gemelo de **PythonModule** que se corresponde con el manifiesto de implementación. Inserte el siguiente contenido JSON en la parte inferior de la sección **moduleContent**, después del módulo gemelo **$edgeHub**:

   ```json
       "PythonModule": {
           "properties.desired":{
               "TemperatureThreshold":25
           }
       }
   ```

   ![Adición de un módulo gemelo a una plantilla de implementación](./media/tutorial-python-module/module-twin.png)

9. Guarde el archivo deployment.template.json.

## <a name="build-and-push-your-module"></a>Compilación e inserción del módulo

En la sección anterior ha creado una solución IoT Edge y ha agregado código a PythonModule que va a filtrar los mensajes y a eliminar aquellos en los que se notifique que la temperatura del equipo está dentro de los límites aceptables. Ahora, tiene que compilar la solución como una imagen de contenedor e insertarla en el registro de contenedor.

1. Abra el terminal integrado de VS Code, para lo que debe seleccionar **View** > **Terminal** (Ver > Terminal).

2. Escriba el comando siguiente en el terminal para iniciar sesión en Docker. Inicie sesión con el nombre de usuario, la contraseña y el servidor de inicio de sesión de Azure Container Registry. Puede recuperar estos valores en la sección **Claves de acceso** del Registro en Azure Portal.

   ```bash
   docker login -u <ACR username> -p <ACR password> <ACR login server>
   ```

   Puede recibir una advertencia de seguridad en la que se recomiende el uso de `--password-stdin`. Aunque ese procedimiento se recomienda para escenarios de producción, está fuera del ámbito de este tutorial. Para más información, consulte la referencia de [docker login](https://docs.docker.com/engine/reference/commandline/login/#provide-a-password-using-stdin).

3. En el explorador de VS Code, haga clic con el botón derecho en el archivo **deployment.template.json** y seleccione **Build and Push IoT Edge solution** (Compilar e insertar solución de IoT Edge).

   El comando de compilación e inserción inicia tres operaciones. En primer lugar, se crea una nueva carpeta en la solución denominada **config** que contiene los archivos del manifiesto de la implementación completa, con la información de la plantilla de implementación y otros archivos de la solución. En segundo lugar, ejecuta `docker build` para generar la imagen de contenedor basándose en el Dockerfile adecuado para la arquitectura de destino. A continuación, ejecuta `docker push` para insertar el repositorio de imágenes en el registro de contenedor.

   Este proceso puede tardar varios minutos la primera vez, pero es más rápido la próxima vez que ejecute los comandos.

## <a name="deploy-modules-to-device"></a>Implementación de módulos en el dispositivo

Utilice el explorador de Visual Studio Code y la extensión Azure IoT Tools para implementar el proyecto de módulo en el dispositivo IoT Edge. Ya tiene un manifiesto de implementación preparado para su escenario, el archivo **deployment.amd64.json** de la carpeta config. Ahora todo lo que necesita hacer es seleccionar un dispositivo que reciba la implementación.

Asegúrese de que el dispositivo IoT Edge está en funcionamiento.

1. En el explorador de Visual Studio Code, en la sección **Azure IoT Hub**, expanda **Dispositivos** para ver la lista de dispositivos IoT.

2. Haga clic con el botón derecho en el nombre del dispositivo IoT Edge y seleccione **Create Deployment for IoT Edge device** (Crear una implementación para un dispositivo individual).

3. Seleccione el archivo **deployment.amd64.json** en la carpeta **config** y, a continuación, haga clic en **Select Edge Deployment Manifest** (Seleccionar manifiesto de implementación de Edge). No utilice el archivo deployment.template.json.

4. En el dispositivo, expanda **Módulos** para ver una lista de módulos implementados y en ejecución. Haga clic en el botón Actualizar. Debería ver el nuevo **PythonModule** en ejecución junto con el módulo **SimulatedTemperatureSensor**, así como **$edgeAgent** y **$edgeHub**.

    Los módulos pueden tardar unos minutos en iniciarse. El entorno de ejecución de Azure IoT Edge necesita recibir su nuevo manifiesto de implementación, extraer las imágenes de los módulos del entorno de ejecución del contenedor y, después, iniciar cada nuevo módulo.

## <a name="view-the-generated-data"></a>Visualización de los datos generados

Una vez aplicado el manifiesto de implementación al dispositivo de IoT Edge, el entorno de ejecución de IoT Edge del dispositivo recopila la información de implementación nueva y comienza a ejecutarse con ella. Los módulos que se ejecuten en el dispositivo y que no están incluidos en el manifiesto de implementación se detienen. Los módulos que falten en el dispositivo se inician.

Puede ver el estado del dispositivo de IoT Edge con la sección **Azure IoT Hub Devices** (Dispositivos de Azure IoT Hub) del explorador de Visual Studio Code. Expanda los detalles del dispositivo para ver una lista de los módulos implementados y en ejecución.

1. En el explorador de Visual Studio Code, haga clic con el botón derecho en el nombre del dispositivo IoT Edge y seleccione **Start Monitoring Built-in Event Endpoint** (Iniciar supervisión del punto de conexión del evento integrado).

2. Vea los mensajes que llegan a IoT Hub. Los mensajes pueden tardar unos minutos en llegar. El dispositivo IoT Edge tiene que recibir la nueva implementación e iniciar todos los módulos. Después, los cambios realizados en el código PythonModule esperan hasta que la temperatura de la máquina alcanza los 25 grados antes de enviar los mensajes. También agrega el tipo de mensaje **Alerta** a los mensajes que llegan a ese umbral de temperatura.

## <a name="edit-the-module-twin"></a>Edición del módulo gemelo

Usamos el módulo gemelo PythonModule en el manifiesto de implementación para establecer el umbral de temperatura en 25 grados. Puede usar al módulo gemelo para cambiar la funcionalidad sin tener que actualizar el código del módulo.

1. En Visual Studio Code, expanda los detalles en el dispositivo IoT Edge para ver los módulos en ejecución.

2. Haga clic con el botón derecho en **PythonModule** y seleccione **Editar módulo gemelo**.

3. Busque **TemperatureThreshold** en las propiedades deseadas. Cambie su valor por una temperatura nueva de 5 a 10 grados más que la última temperatura registrada.

4. Guarde el archivo del módulo gemelo.

5. Haga clic con el botón derecho en cualquier parte del panel de edición del módulo gemelo y seleccione **Actualizar módulo gemelo**.

6. Supervise los mensajes entrantes del dispositivo a la nube. Debería ver que los mensajes se detienen hasta que se alcanza el nuevo umbral de temperatura.

## <a name="clean-up-resources"></a>Limpieza de recursos

Si prevé seguir con el siguiente artículo recomendado, puede mantener los recursos y las configuraciones que ya ha creado y volverlos a utilizar. También puede seguir usando el mismo dispositivo de IoT Edge como dispositivo de prueba.

En caso contrario, para evitar gastos, puede eliminar las configuraciones locales y los recursos de Azure que creó en este artículo.

[!INCLUDE [iot-edge-clean-up-cloud-resources](../../includes/iot-edge-clean-up-cloud-resources.md)]

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha creado un módulo IoT Edge que contiene código para filtrar los datos sin procesar generados por el dispositivo IoT Edge. Cuando esté preparado para crear sus propios módulos, puede aprender más sobre cómo [desarrollar sus propios módulos IoT Edge](module-development.md) y sobre cómo [desarrollar módulos con Visual Studio Code](how-to-vs-code-develop-module.md). Para obtener ejemplos de módulos de IoT Edge, incluido el módulo de temperatura simulado, consulte [Ejemplos de módulos de IoT Edge](https://github.com/Azure/iotedge/tree/master/edge-modules) y [Ejemplos del SDK de Python para IoT](https://github.com/Azure/azure-iot-sdk-python/tree/master/azure-iot-device/samples/async-edge-scenarios).

Puede continuar con los siguientes tutoriales para obtener información sobre cómo Azure IoT Edge puede ayudarle a implementar servicios en la nube de Azure para procesar y analizar datos en el perímetro.

> [!div class="nextstepaction"]
> [Functions](tutorial-deploy-function.md)
> [Stream Analytics](tutorial-deploy-stream-analytics.md)
> [Machine Learning](tutorial-deploy-machine-learning.md)
> [Custom Vision Service](tutorial-deploy-custom-vision.md)
