---
title: Tutorial de desarrollo del módulo Node.js para Linux en Azure IoT Edge | Microsoft Docs
description: En este tutorial se muestra cómo crear un módulo de IoT Edge con código de Node.js e implementarlo en un dispositivo perimetral.
services: iot-edge
author: shizn
manager: philmea
ms.author: xshi
ms.date: 07/30/2020
ms.topic: tutorial
ms.service: iot-edge
ms.custom: mvc, devx-track-python, devx-track-javascript
ms.openlocfilehash: b1a24c4e97ba73dc5097e7b3c09c0c1c5a8771ac
ms.sourcegitcommit: dea88d5e28bd4bbd55f5303d7d58785fad5a341d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87875428"
---
# <a name="tutorial-develop-and-deploy-a-nodejs-iot-edge-module-for-linux-devices"></a>Tutorial: Desarrollo e implementación de un módulo de Node.js de IoT Edge para dispositivos Linux

Use Visual Studio Code para desarrollar código de Node.js e implementarlo en un dispositivo Linux que ejecute Azure IoT Edge.

Los módulos de IoT Edge se pueden usar para implementar código que, a su vez, implementa una lógica de negocios directamente en los dispositivos de IoT Edge. En este tutorial, se detallan los pasos para crear e implementar un módulo de IoT Edge que filtra los datos de sensor. Utilizará el dispositivo de IoT Edge simulado que creó en las guías de inicio rápido. En este tutorial, aprenderá a:

> [!div class="checklist"]
>
> * Uso de Visual Studio Code para crear un módulo de Node.js de IoT Edge
> * Uso de Visual Studio Code y Docker para crear una imagen de Docker y publicarla en el Registro
> * Implementación del módulo en el dispositivo IoT Edge
> * Visualización de datos generados

El módulo IoT Edge que creó en este tutorial filtra lo datos sobre la temperatura generados por el dispositivo. Solo envía mensajes a los niveles superiores si la temperatura sobrepasa el umbral especificado. Este tipo de análisis perimetral resulta útil para reducir la cantidad de datos que se comunican a la nube y se almacenan en ella.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="solution-scope"></a>Ámbito de la solución

Este tutorial muestra cómo desarrollar un módulo en **Node.js** mediante **Visual Studio Code** y cómo se implementa en un **dispositivo Linux**. IoT Edge no admite los módulos de Node.js para dispositivos Windows.

Utilice la tabla siguiente si desea conocer las opciones para desarrollar e implementar módulos de Node.js:

| Node.js | Visual Studio Code | Visual Studio 2017/2019 |
| - | ------------------ | ------------------ |
| **Linux AMD64** | ![Usar VS Code para los módulos de Node.js en Linux AMD64](./media/tutorial-c-module/green-check.png) |  |
| **Linux ARM32** | ![Usar VS Code para los módulos de Node.js en Linux ARM32](./media/tutorial-c-module/green-check.png) |  |

## <a name="prerequisites"></a>Requisitos previos

Antes de comenzar este tutorial, debe haber realizado el anterior para configurar el entorno de desarrollo de contenedores de Linux: [Desarrollo de módulos IoT Edge para dispositivos Linux](tutorial-develop-for-linux.md). Al completar cualquiera de estos tutoriales, se deben cumplir los siguientes requisitos previos:

* Una instancia de [IoT Hub](../iot-hub/iot-hub-create-through-portal.md) de nivel estándar o gratis en Azure.
* Un [dispositivo Linux que ejecute Azure IoT Edge](quickstart-linux.md).
* Un registro de contenedor, como [Azure Container Registry](https://docs.microsoft.com/azure/container-registry/).
* [Visual Studio Code](https://code.visualstudio.com/) configurado con [Azure IoT Tools](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-tools).
* [CE de Docker](https://docs.docker.com/install/) configurado para ejecutar contenedores de Linux.

Para desarrollar un módulo de IoT Edge en Node.js, instale los siguientes requisitos previos adicionales en la máquina de desarrollo:

* [Node.js y npm](https://nodejs.org) El paquete npm se distribuye con Node.js, lo que significa que cuando se descarga Node.js, automáticamente se instala npm en el equipo.

## <a name="create-a-module-project"></a>Creación de un proyecto de módulo

En los siguientes pasos, puede ver cómo crear un módulo de Node.js de IoT Edge mediante Visual Studio Code y Azure IoT Tools.

### <a name="create-a-new-project"></a>Creación de un nuevo proyecto

Use **npm** para crear una plantilla de solución de Node.js que se puede compilar a partir de él.

1. En Visual Studio Code, seleccione **Ver** > **Terminal integrado** para abrir el terminal integrado de VS Code.

2. En el terminal integrado, escriba el siguiente comando para instalar **yeoman** y el generador para el módulo de Node.js de Azure IoT Edge:

    ```cmd/sh
    npm install -g yo generator-azure-iot-edge-module
    ```

3. Seleccione **Ver** > **Paleta de comandos** para abrir la paleta de comandos de VS Code.

4. En la paleta de comandos, escriba y ejecute el comando **Azure: Sign in** (Azure: iniciar sesión) y siga las instrucciones para iniciar sesión en la cuenta de Azure. Si ya ha iniciado sesión, puede omitir este paso.

5. En la paleta de comandos, escriba y ejecute el comando **Azure IoT Edge: New IoT Edge solution** (Azure IoT Edge: nueva solución de IoT Edge). Para crear la solución, siga las indicaciones de la paleta de comandos.

   | Campo | Value |
   | ----- | ----- |
   | Seleccionar carpeta | Elija la ubicación en el equipo de desarrollo en la que VS Code creará los archivos de la solución. |
   | Proporcionar un nombre de la solución | Escriba un nombre descriptivo para la solución o acepte el valor predeterminado **EdgeSolution**. |
   | Seleccionar plantilla del módulo | Elija **Módulo Node.js**. |
   | Proporcionar un nombre de módulo | Llame al módulo **NodeModule**. |
   | Proporcionar repositorio de imágenes de Docker del módulo | Un repositorio de imágenes incluye el nombre del registro de contenedor y el nombre de la imagen de contenedor. La imagen de contenedor se rellena previamente con el nombre que proporcionó en el último paso. Reemplace **localhost:5000** por el valor de **Servidor de inicio de sesión** del registro de contenedor de Azure. Puede recuperar el servidor de inicio de sesión en la página de información general del registro de contenedor en Azure Portal. <br><br>El repositorio de imágenes final es similar a \<registry name\>.azurecr.io/nodemodule. |

   ![Especificación del repositorio de imágenes de Docker](./media/tutorial-node-module/repository.png)

### <a name="add-your-registry-credentials"></a>Adición de las credenciales del Registro

El archivo del entorno almacena las credenciales del repositorio de contenedores y las comparte con el entorno de ejecución de IoT Edge. El entorno de ejecución necesita estas credenciales para extraer las imágenes privadas e insertarlas en el dispositivo IoT Edge.

La extensión de IoT Edge intenta extraer de Azure las credenciales del registro del contenedor y rellenar con ellas el archivo de entorno. Compruebe si las credenciales ya están incluidas. Si no lo están, agréguelas ahora:

1. En el explorador de VS Code, abra el archivo **.env**.
2. Actualice los campos con los valores de **nombre de usuario** y **contraseña** que ha copiado del Registro de contenedor de Azure.
3. Guarde este archivo.

### <a name="select-your-target-architecture"></a>Selección de la arquitectura de destino

Actualmente, Visual Studio Code puede desarrollar módulos de Node.js para dispositivos Linux AMD64 y Linux ARM32v7. Debe seleccionar qué arquitectura tiene como destino con cada solución, dado que el contenedor se crea y ejecuta de manera diferente para cada tipo de arquitectura. La opción predeterminada es Linux AMD64.

1. Abra la paleta de comandos y busque **Azure IoT Edge: Set Default Target Platform for Edge Solution** (Establecer la plataforma de destino predeterminada para la solución perimetral), o bien seleccione el icono de acceso directo en la barra lateral en la parte inferior de la ventana.

2. En la paleta de comandos, seleccione la arquitectura de destino en la lista de opciones. Para este tutorial, usamos una máquina virtual Ubuntu como dispositivo IoT Edge, por lo que mantendrá el valor predeterminado **amd64**.

### <a name="update-the-module-with-custom-code"></a>Actualización del módulo con código personalizado

Cada plantilla viene con un código de ejemplo incluido, que toma los datos del sensor simulado del módulo **SimulatedTemperatureSensor** y los enruta a IoT Hub. En esta sección, agregue código para que NodeModule analice los mensajes antes de enviarlos.

1. En el explorador de VS Code, abra **módulos** > **NodeModule** > **app.js**.

2. Agregue una variable de umbral de temperatura por debajo de los módulos de nodo necesarios. El umbral de temperatura establece el valor que debe superar la temperatura medida para que los datos se envíen a IoT Hub.

    ```javascript
    var temperatureThreshold = 25;
    ```

3. Reemplace la función `PipeMessage` por la función `FilterMessage`.

    ```javascript
    // This function filters out messages that report temperatures below the temperature threshold.
    // It also adds the MessageType property to the message with the value set to Alert.
    function filterMessage(client, inputName, msg) {
        client.complete(msg, printResultFor('Receiving message'));
        if (inputName === 'input1') {
            var message = msg.getBytes().toString('utf8');
            var messageBody = JSON.parse(message);
            if (messageBody && messageBody.machine && messageBody.machine.temperature && messageBody.machine.temperature > temperatureThreshold) {
                console.log(`Machine temperature ${messageBody.machine.temperature} exceeds threshold ${temperatureThreshold}`);
                var outputMsg = new Message(message);
                outputMsg.properties.add('MessageType', 'Alert');
                client.sendOutputEvent('output1', outputMsg, printResultFor('Sending received message'));
            }
        }
    }

    ```

4. Reemplace el nombre de la función `pipeMessage` por `filterMessage` en la función `client.on()`.

    ```javascript
    client.on('inputMessage', function (inputName, msg) {
        filterMessage(client, inputName, msg);
        });
    ```

5. Copie el siguiente fragmento de código en la función de devolución de llamada `client.open()`, después `client.on()` dentro de la instrucción `else`. Esta función se invoca cuando se actualicen las propiedades deseadas.

    ```javascript
    client.getTwin(function (err, twin) {
        if (err) {
            console.error('Error getting twin: ' + err.message);
        } else {
            twin.on('properties.desired', function(delta) {
                if (delta.TemperatureThreshold) {
                    temperatureThreshold = delta.TemperatureThreshold;
                }
            });
        }
    });
    ```

6. Guarde el archivo app.js.

7. En el explorador de VS Code, abra el archivo **deployment.template.json** en el área de trabajo de la solución de IoT Edge.

8. Agregue el módulo gemelo NodeModule al manifiesto de implementación. Inserte el siguiente contenido JSON en la parte inferior de la sección `moduleContent`, después del módulo gemelo `$edgeHub`:

   ```json
     "NodeModule": {
         "properties.desired":{
             "TemperatureThreshold":25
         }
     }
   ```

   ![Adición de un módulo gemelo a una plantilla de implementación](./media/tutorial-node-module/module-twin.png)

9. Guarde el archivo deployment.template.json.

## <a name="build-and-push-your-module"></a>Compilación e inserción del módulo

En la sección anterior ha creado una solución IoT Edge y ha agregado código a NodeModule que va a filtrar los mensajes y a eliminar aquellos en los que se notifique que la temperatura del equipo está dentro de los límites aceptables. Ahora, tiene que compilar la solución como una imagen de contenedor e insertarla en el registro de contenedor.

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

4. En el dispositivo, expanda **Módulos** para ver una lista de módulos implementados y en ejecución. Haga clic en el botón Actualizar. Debería ver el nuevo **NodeModule** en ejecución junto con el módulo **SimulatedTemperatureSensor** y el **$edgeAgent** y **$edgeHub**.

    Los módulos pueden tardar unos minutos en iniciarse. El entorno de ejecución de Azure IoT Edge necesita recibir su nuevo manifiesto de implementación, extraer las imágenes de los módulos del entorno de ejecución del contenedor y, después, iniciar cada nuevo módulo.

## <a name="view-the-generated-data"></a>Visualización de los datos generados

Una vez aplicado el manifiesto de implementación al dispositivo de IoT Edge, el entorno de ejecución de IoT Edge del dispositivo recopila la información de implementación nueva y comienza a ejecutarse con ella. Los módulos que se ejecuten en el dispositivo y que no están incluidos en el manifiesto de implementación se detienen. Los módulos que falten en el dispositivo se inician.

Puede ver el estado del dispositivo de IoT Edge con la sección **Azure IoT Hub Devices** (Dispositivos de Azure IoT Hub) del explorador de Visual Studio Code. Expanda los detalles del dispositivo para ver una lista de los módulos implementados y en ejecución.

1. En el explorador de Visual Studio Code, haga clic con el botón derecho en el nombre del dispositivo IoT Edge y seleccione **Start Monitoring Built-in Event Endpoint** (Iniciar supervisión del punto de conexión del evento integrado).

2. Vea los mensajes que llegan a IoT Hub. Los mensajes pueden tardar unos minutos en llegar. El dispositivo IoT Edge tiene que recibir la nueva implementación e iniciar todos los módulos. Después, los cambios realizados en el código NodeModule esperan hasta que la temperatura de la máquina alcanza los 25 grados antes de enviar los mensajes. También agrega el tipo de mensaje **Alerta** a los mensajes que llegan a ese umbral de temperatura.

## <a name="edit-the-module-twin"></a>Edición del módulo gemelo

Usamos el módulo gemelo NodeModule en el manifiesto de implementación para establecer el umbral de temperatura en 25 grados. Puede usar al módulo gemelo para cambiar la funcionalidad sin tener que actualizar el código del módulo.

1. En Visual Studio Code, expanda los detalles en el dispositivo IoT Edge para ver los módulos en ejecución.

2. Haga clic con el botón derecho en **NodeModule** y seleccione **Editar módulo gemelo**.

3. Busque **TemperatureThreshold** en las propiedades deseadas. Cambie su valor por una temperatura nueva de 5 a 10 grados más que la última temperatura registrada.

4. Guarde el archivo del módulo gemelo.

5. Haga clic con el botón derecho en cualquier parte del panel de edición del módulo gemelo y seleccione **Actualizar módulo gemelo**.

6. Supervise los mensajes entrantes del dispositivo a la nube. Debería ver que los mensajes se detienen hasta que se alcanza el nuevo umbral de temperatura.

## <a name="clean-up-resources"></a>Limpieza de recursos

Si prevé seguir con el siguiente artículo recomendado, puede mantener los recursos y las configuraciones que ya ha creado y volverlos a utilizar. También puede seguir usando el mismo dispositivo de IoT Edge como dispositivo de prueba.

En caso contrario, para evitar gastos, puede eliminar las configuraciones locales y los recursos de Azure que creó en este artículo.

[!INCLUDE [iot-edge-clean-up-cloud-resources](../../includes/iot-edge-clean-up-cloud-resources.md)]

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha creado un módulo IoT Edge que contiene código para filtrar los datos sin procesar generados por el dispositivo IoT Edge. Cuando esté preparado para crear sus propios módulos, puede aprender más sobre cómo [desarrollar sus propios módulos IoT Edge](module-development.md) y sobre cómo [desarrollar módulos con Visual Studio Code](how-to-vs-code-develop-module.md). Para obtener ejemplos de módulos de IoT Edge, incluido el módulo de temperatura simulado, consulte [Ejemplos de módulos de IoT Edge](https://github.com/Azure/iotedge/tree/master/edge-modules).

Puede continuar con los siguientes tutoriales para obtener información sobre cómo Azure IoT Edge puede ayudarle a implementar servicios en la nube de Azure para procesar y analizar datos en el perímetro.

> [!div class="nextstepaction"]
> [Functions](tutorial-deploy-function.md)
> [Stream Analytics](tutorial-deploy-stream-analytics.md)
> [Machine Learning](tutorial-deploy-machine-learning.md)
> [Custom Vision Service](tutorial-deploy-custom-vision.md)
