---
title: 'Tutorial: Personalización del módulo de Java mediante Azure IoT Edge'
description: En este tutorial se muestra la creación de un módulo IoT Edge con código Java y su implementación en un dispositivo perimetral.
services: iot-edge
author: kgremban
manager: philmea
ms.author: kgremban
ms.date: 07/30/2020
ms.topic: tutorial
ms.service: iot-edge
ms.custom:
- mvc
- mqtt
- devx-track-java
ms.openlocfilehash: ef4b42b56ea3e9e73de6d5bc793b2f1b206f34f3
ms.sourcegitcommit: 2ff0d073607bc746ffc638a84bb026d1705e543e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87829992"
---
# <a name="tutorial-develop-a-java-iot-edge-module-for-linux-devices"></a>Tutorial: Desarrollo de un módulo IoT Edge con Java para dispositivos Linux

Los módulos Azure IoT Edge se pueden usar para implementar código que, a su vez, implementa una lógica de negocios directamente en los dispositivos IoT Edge. En este tutorial, se detallan los pasos para crear e implementar un módulo de IoT Edge que filtra los datos de sensor. Utilizará el dispositivo IoT Edge simulado que creó en el inicio rápido Implementación de Azure IoT Edge en un dispositivo simulado en [Linux](quickstart-linux.md). En este tutorial, aprenderá a:

> [!div class="checklist"]
>
> * Use Visual Studio Code para crear un módulo de Java de IoT Edge basado en el paquete de plantillas de Maven de Azure IoT Edge y el SDK de dispositivo de Java de Azure IoT.
> * Utilizar Visual Studio Code y Docker para crear una imagen de Docker y publicarla en el Registro.
> * Implementar el módulo en el dispositivo IoT Edge.
> * Ver datos generados.

El módulo IoT Edge que creó en este tutorial filtra lo datos sobre la temperatura generados por el dispositivo. Solo envía mensajes a los niveles superiores si la temperatura sobrepasa el umbral especificado. Este tipo de análisis perimetral resulta útil para reducir la cantidad de datos que se comunican a la nube y se almacenan en ella.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="solution-scope"></a>Ámbito de la solución

Este tutorial muestra cómo desarrollar un módulo en **Java** mediante **Visual Studio Code** y cómo se implementa en un **dispositivo Linux**. IoT Edge no admite módulos de Java para dispositivos Windows.

Utilice la tabla siguiente si desea conocer las opciones para desarrollar e implementar módulos de Java:

| Java | Visual Studio Code | Visual Studio 2017/2019 |
| - | ------------------ | ------------------ |
| **Linux AMD64** | ![Usar VS Code para los módulos de Java en Linux AMD64](./media/tutorial-c-module/green-check.png) |  |
| **Linux ARM32** | ![Usar VS Code para los módulos de Java en Linux ARM32](./media/tutorial-c-module/green-check.png) |  |

## <a name="prerequisites"></a>Prerrequisitos

Antes de comenzar este tutorial, debe haber realizado el anterior para configurar el entorno de desarrollo de contenedores de Linux: [Desarrollo de módulos IoT Edge para dispositivos Linux](tutorial-develop-for-linux.md). Al completar cualquiera de estos tutoriales, se deben cumplir los siguientes requisitos previos:

* Una instancia de [IoT Hub](../iot-hub/iot-hub-create-through-portal.md) de nivel estándar o gratis en Azure.
* Un [dispositivo Linux que ejecute Azure IoT Edge](quickstart-linux.md).
* Un registro de contenedor, como [Azure Container Registry](https://docs.microsoft.com/azure/container-registry/).
* [Visual Studio Code](https://code.visualstudio.com/) configurado con [Azure IoT Tools](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-tools).
* [CE de Docker](https://docs.docker.com/install/) configurado para ejecutar contenedores de Linux.

Para desarrollar un módulo de IoT Edge en Java, instale los siguientes requisitos previos adicionales en la máquina de desarrollo: 

* [Paquete de extensiones de Java](https://marketplace.visualstudio.com/items?itemName=vscjava.vscode-java-pack) para Visual Studio Code.
* [Java SE Development Kit 10](https://aka.ms/azure-jdks) y [establecer la `JAVA_HOME`variable de entorno](https://docs.oracle.com/cd/E19182-01/820-7851/inst_cli_jdk_javahome_t/) para que apunte a la instalación de JDK.
* [Maven](https://maven.apache.org/)

   >[!TIP]
   >Los procesos de instalación de Java y Maven agregan variables de entorno al sistema. Después de completar la instalación, reinicie las instancias del símbolo del sistema, PowerShell o cualquier terminal de Visual Studio Code abiertas. Este paso garantiza que estas utilidades puedan reconocer los comandos Java y Maven en adelante.

## <a name="create-a-module-project"></a>Creación de un proyecto de módulo

Use los pasos siguientes para crear proyecto de módulo de IoT Edge basado en el paquete de plantillas de Maven de Azure IoT Edge y el SDK de dispositivo de Java de Azure IoT. El proyecto se crea con Visual Studio Code y Azure IoT Tools.

### <a name="create-a-new-project"></a>Creación de un nuevo proyecto

Cree una plantilla de solución de Java que pueda personalizar con su propio código.

1. En Visual Studio Code, seleccione **Ver** > **Paleta de comandos** para abrir la paleta de comandos de VS Code.

2. En la paleta de comandos, escriba y ejecute el comando **Azure IoT Edge: New IoT Edge solution** (Azure IoT Edge: nueva solución de IoT Edge). Para crear la solución, siga las indicaciones de la paleta de comandos.

   | Campo | Value |
   | ----- | ----- |
   | Seleccionar carpeta | Elija la ubicación en el equipo de desarrollo en la que VS Code creará los archivos de la solución. |
   | Proporcionar un nombre de la solución | Escriba un nombre descriptivo para la solución o acepte el valor predeterminado **EdgeSolution**. |
   | Seleccionar plantilla del módulo | Elija **Módulo de Java**. |
   | Proporcionar un nombre de módulo | Llame al módulo **JavaModule**. |
   | Proporcionar repositorio de imágenes de Docker del módulo | Un repositorio de imágenes incluye el nombre del registro de contenedor y el nombre de la imagen de contenedor. La imagen de contenedor se rellena previamente con el nombre que proporcionó en el último paso. Reemplace **localhost:5000** por el valor de **Servidor de inicio de sesión** del registro de contenedor de Azure. Puede recuperar el servidor de inicio de sesión en la página de información general del registro de contenedor en Azure Portal. <br><br>El repositorio de imágenes final es similar a \<registry name\>.azurecr.io/javamodule. |
   | Proporcionar valor de groupId | Escriba un valor de identificador de grupo o acepte el valor predeterminado **com.edgemodule**. |

   ![Especificación del repositorio de imágenes de Docker](./media/tutorial-java-module/repository.png)

Si es la primera vez que crea un módulo de Java, puede que tarde varios minutos en descargar los paquetes de Maven. Cuando la solución esté lista, la ventana de VS Code cargará el área de trabajo de la solución de IoT Edge. El área de trabajo de la solución contiene cinco componentes de nivel superior.

* La carpeta **modules** contiene el código Java del módulo y los archivos de Docker para compilar el módulo como una imagen de contenedor.
* El archivo **\.env** almacena las credenciales del registro de contenedor.
* El archivo **deployment.template.json** contiene la información que usa el entorno de ejecución de IoT Edge para implementar módulos en un dispositivo.
* El archivo **deployment.debug.template.json** contiene la versión de depuración de los módulos.
* En este tutorial no se editarán la carpeta **\.vscode** ni el archivo **\.gitignore**.

Si no especificó un registro de contenedor al crear la solución y aceptó el valor predeterminado localhost:5000, no tendrá un archivo \.env.

### <a name="add-your-registry-credentials"></a>Adición de las credenciales del Registro

El archivo del entorno almacena las credenciales del registro de contenedor y las comparte con el runtime de IoT Edge. El entorno de ejecución necesita estas credenciales para extraer las imágenes privadas e insertarlas en el dispositivo IoT Edge.

La extensión de IoT Edge intenta extraer de Azure las credenciales del registro del contenedor y rellenar con ellas el archivo de entorno. Compruebe si las credenciales ya están incluidas. Si no lo están, agréguelas ahora:

1. En el explorador de VS Code, abra el archivo .env.
2. Actualice los campos con los valores de **nombre de usuario** y **contraseña** que ha copiado del Registro de contenedor de Azure.
3. Guarde este archivo.

### <a name="select-your-target-architecture"></a>Selección de la arquitectura de destino

Actualmente, Visual Studio Code puede desarrollar módulos de Java para dispositivos Linux AMD64 y Linux ARM32v7. Debe seleccionar qué arquitectura tiene como destino con cada solución, dado que el contenedor se crea y ejecuta de manera diferente para cada tipo de arquitectura. La opción predeterminada es Linux AMD64.

1. Abra la paleta de comandos y busque **Azure IoT Edge: Set Default Target Platform for Edge Solution** (Establecer la plataforma de destino predeterminada para la solución perimetral), o bien seleccione el icono de acceso directo en la barra lateral en la parte inferior de la ventana.

2. En la paleta de comandos, seleccione la arquitectura de destino en la lista de opciones. Para este tutorial, usamos una máquina virtual Ubuntu como dispositivo IoT Edge, por lo que mantendrá el valor predeterminado **amd64**.

### <a name="update-the-module-with-custom-code"></a>Actualización del módulo con código personalizado

1. En el Explorador de VS Code, abra **modules** > **JavaModule** > **src** > **main** > **java** > **com** > **edgemodule** > **App.java**.

2. Agregue el código siguiente en la parte superior del archivo para importar nuevas clases a las que se hace referencia.

    ```java
    import java.io.StringReader;
    import java.util.concurrent.atomic.AtomicLong;
    import java.util.HashMap;
    import java.util.Map;

    import javax.json.Json;
    import javax.json.JsonObject;
    import javax.json.JsonReader;

    import com.microsoft.azure.sdk.iot.device.DeviceTwin.Pair;
    import com.microsoft.azure.sdk.iot.device.DeviceTwin.Property;
    import com.microsoft.azure.sdk.iot.device.DeviceTwin.TwinPropertyCallBack;
    ```

3. Agregue la siguiente definición a la clase **App**. Esta variable establece un umbral de temperatura. La temperatura medida en la máquina no se notificará a IoT Hub hasta que supere este valor.

    ```java
    private static final String TEMP_THRESHOLD = "TemperatureThreshold";
    private static AtomicLong tempThreshold = new AtomicLong(25);
    ```

4. Reemplace el método execute de **MessageCallbackMqtt** por el código siguiente. Se llama a este método cada vez que el módulo recibe un mensaje MQTT del centro de IoT Edge. Filtra los mensajes que informan de temperaturas por debajo del umbral de temperatura que se establecen mediante el módulo gemelo.

    ```java
    protected static class MessageCallbackMqtt implements MessageCallback {
        private int counter = 0;
        @Override
        public IotHubMessageResult execute(Message msg, Object context) {
            this.counter += 1;

            String msgString = new String(msg.getBytes(), Message.DEFAULT_IOTHUB_MESSAGE_CHARSET);
            System.out.println(
                   String.format("Received message %d: %s",
                            this.counter, msgString));
            if (context instanceof ModuleClient) {
                try (JsonReader jsonReader = Json.createReader(new StringReader(msgString))) {
                    final JsonObject msgObject = jsonReader.readObject();
                    double temperature = msgObject.getJsonObject("machine").getJsonNumber("temperature").doubleValue();
                    long threshold = App.tempThreshold.get();
                    if (temperature >= threshold) {
                        ModuleClient client = (ModuleClient) context;
                        System.out.println(
                            String.format("Temperature above threshold %d. Sending message: %s",
                            threshold, msgString));
                        client.sendEventAsync(msg, eventCallback, msg, App.OUTPUT_NAME);
                    }
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
            return IotHubMessageResult.COMPLETE;
        }
    }
    ```

5. Agregue las dos clases internas estáticas siguientes a la clase **App**. Estas clases actualizan la variable tempThreshold cuando cambia la propiedad deseada en el módulo gemelo. Todos los módulos tienen su propio módulo gemelo, que le permite configurar el código que se ejecuta dentro de un módulo directamente desde la nube.

    ```java
    protected static class DeviceTwinStatusCallBack implements IotHubEventCallback {
        @Override
        public void execute(IotHubStatusCode status, Object context) {
            System.out.println("IoT Hub responded to device twin operation with status " + status.name());
        }
    }

    protected static class OnProperty implements TwinPropertyCallBack {
        @Override
        public void TwinPropertyCallBack(Property property, Object context) {
            if (!property.getIsReported()) {
                if (property.getKey().equals(App.TEMP_THRESHOLD)) {
                    try {
                        long threshold = Math.round((double) property.getValue());
                        App.tempThreshold.set(threshold);
                    } catch (Exception e) {
                        System.out.println("Faile to set TemperatureThread with exception");
                        e.printStackTrace();
                    }
                }
            }
        }
    }
    ```

6. Agregue las líneas siguientes al método **main** después de **client.open()** para suscribirse a las actualizaciones del módulo gemelo.

    ```java
    client.startTwin(new DeviceTwinStatusCallBack(), null, new OnProperty(), null);
    Map<Property, Pair<TwinPropertyCallBack, Object>> onDesiredPropertyChange = new HashMap<Property, Pair<TwinPropertyCallBack, Object>>() {
        {
            put(new Property(App.TEMP_THRESHOLD, null), new Pair<TwinPropertyCallBack, Object>(new OnProperty(), null));
        }
    };
    client.subscribeToTwinDesiredProperties(onDesiredPropertyChange);
    client.getTwin();
    ```

7. Guarde el archivo App.java.

8. En el explorador de VS Code, abra el archivo **deployment.template.json** en el área de trabajo de la solución de IoT Edge.

9. Agregue el módulo gemelo **JavaModule** al manifiesto de implementación. Inserte el siguiente contenido JSON en la parte inferior de la sección **moduleContent**, después del módulo gemelo **$edgeHub**:

   ```json
     "JavaModule": {
         "properties.desired":{
             "TemperatureThreshold":25
         }
     }
   ```

   ![Adición de un módulo gemelo a una plantilla de implementación](./media/tutorial-java-module/module-twin.png)

10. Guarde el archivo deployment.template.json.

## <a name="build-and-push-your-module"></a>Compilación e inserción del módulo

En la sección anterior, creó una solución de IoT Edge y agregó código a **JavaModule** para filtrar los mensajes en los que la temperatura registrada por la máquina está por debajo del límite aceptable. Ahora, compile la solución como una imagen de contenedor e insertarla en el registro de contenedor.

1. Abra el terminal integrado de VS Code, para lo que debe seleccionar **View** > **Terminal** (Ver > Terminal).

2. Escriba el comando siguiente en el terminal para iniciar sesión en Docker. Inicie sesión con el nombre de usuario, la contraseña y el servidor de inicio de sesión de Azure Container Registry. Puede recuperar estos valores en la sección **Claves de acceso** del Registro en Azure Portal.

   ```bash
   docker login -u <ACR username> -p <ACR password> <ACR login server>
   ```

   Puede recibir una advertencia de seguridad en la que se recomiende el uso de `--password-stdin`. Aunque ese procedimiento se recomienda para escenarios de producción, está fuera del ámbito de este tutorial. Para más información, consulte la referencia de [docker login](https://docs.docker.com/engine/reference/commandline/login/#provide-a-password-using-stdin).

3. En el explorador de VS Code, haga clic con el botón derecho en el archivo **deployment.template.json** y seleccione **Build and Push IoT Edge solution** (Compilar e insertar solución de IoT Edge).

   El comando de compilación e inserción inicia tres operaciones. En primer lugar, se crea una nueva carpeta en la solución llamada **config**, que contiene los archivos del manifiesto de la implementación completa, con la información de la plantilla de implementación y otros archivos de la solución. En segundo lugar, ejecuta `docker build` para generar la imagen de contenedor basándose en el Dockerfile adecuado para la arquitectura de destino. A continuación, ejecuta `docker push` para insertar el repositorio de imágenes en el registro de contenedor.

   Este proceso puede tardar varios minutos la primera vez, pero es más rápido la próxima vez que ejecute los comandos.

## <a name="deploy-modules-to-device"></a>Implementación de módulos en el dispositivo

Utilice el explorador de Visual Studio Code y la extensión Azure IoT Tools para implementar el proyecto de módulo en el dispositivo IoT Edge. Ya tiene un manifiesto de implementación preparado para su escenario, el archivo **deployment.amd64.json** de la carpeta config. Ahora todo lo que necesita hacer es seleccionar un dispositivo que reciba la implementación.

Asegúrese de que el dispositivo IoT Edge está en funcionamiento.

1. En el explorador de Visual Studio Code, en la sección **Azure IoT Hub**, expanda **Dispositivos** para ver la lista de dispositivos IoT.

2. Haga clic con el botón derecho en el nombre del dispositivo IoT Edge y seleccione **Create Deployment for IoT Edge device** (Crear una implementación para un dispositivo individual).

3. Seleccione el archivo **deployment.amd64.json** en la carpeta **config** y, a continuación, haga clic en **Select Edge Deployment Manifest** (Seleccionar manifiesto de implementación de Edge). No utilice el archivo deployment.template.json.

4. En el dispositivo, expanda **Módulos** para ver una lista de módulos implementados y en ejecución. Haga clic en el botón Actualizar. Debería ver el nuevo **JavaModule** en ejecución junto con el módulo **SimulatedTemperatureSensor**, así como **$edgeAgent** y **$edgeHub**.  

    Los módulos pueden tardar unos minutos en iniciarse. El entorno de ejecución de Azure IoT Edge necesita recibir su nuevo manifiesto de implementación, extraer las imágenes de los módulos del entorno de ejecución del contenedor y, después, iniciar cada nuevo módulo.

## <a name="view-the-generated-data"></a>Visualización de los datos generados

Una vez aplicado el manifiesto de implementación al dispositivo de IoT Edge, el entorno de ejecución de IoT Edge del dispositivo recopila la información de implementación nueva y comienza a ejecutarse con ella. Los módulos que se ejecuten en el dispositivo y que no están incluidos en el manifiesto de implementación se detienen. Los módulos que falten en el dispositivo se inician.

1. En el explorador de Visual Studio Code, haga clic con el botón derecho en el nombre del dispositivo IoT Edge y seleccione **Start Monitoring Built-in Event Endpoint** (Iniciar supervisión del punto de conexión del evento integrado).

2. Vea los mensajes que llegan a IoT Hub. Los mensajes pueden tardar unos minutos en llegar. El dispositivo IoT Edge tiene que recibir la nueva implementación e iniciar todos los módulos. Después, los cambios realizados en el código JavaModule esperan hasta que la temperatura de la máquina alcanza los 25 grados antes de enviar los mensajes. También agrega el tipo de mensaje **Alerta** a los mensajes que llegan a ese umbral de temperatura.

## <a name="edit-the-module-twin"></a>Edición del módulo gemelo

Usamos el módulo gemelo JavaModule en el manifiesto de implementación para establecer el umbral de temperatura en 25 grados. Puede usar al módulo gemelo para cambiar la funcionalidad sin tener que actualizar el código del módulo.

1. En Visual Studio Code, expanda los detalles en el dispositivo IoT Edge para ver los módulos en ejecución.

2. Haga clic con el botón derecho en **JavaModule** y seleccione **Editar módulo gemelo**.

3. Busque **TemperatureThreshold** en las propiedades deseadas. Cambie su valor por una temperatura nueva de 5 a 10 grados más que la última temperatura registrada.

4. Guarde el archivo del módulo gemelo.

5. Haga clic con el botón derecho en cualquier parte del panel de edición del módulo gemelo y seleccione **Actualizar módulo gemelo**.

6. Supervise los mensajes entrantes del dispositivo a la nube. Debería ver que los mensajes se detienen hasta que se alcanza el nuevo umbral de temperatura.

## <a name="clean-up-resources"></a>Limpieza de recursos

Si prevé seguir con el siguiente artículo recomendado, puede mantener los recursos y las configuraciones que ya ha creado y volverlos a utilizar. También puede seguir usando el mismo dispositivo de IoT Edge como dispositivo de prueba.

En caso contrario, para evitar gastos, puede eliminar las configuraciones locales y los recursos de Azure que creó en este artículo.

[!INCLUDE [iot-edge-clean-up-cloud-resources](../../includes/iot-edge-clean-up-cloud-resources.md)]

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha creado un módulo de IoT Edge que filtra los datos sin procesar generados por el dispositivo IoT Edge. Cuando esté preparado para crear sus propios módulos, puede aprender más sobre cómo [desarrollar módulos IoT Edge](module-development.md) y sobre cómo [desarrollar módulos con Visual Studio Code](how-to-vs-code-develop-module.md). Consulte [Ejemplos de módulos de IoT Edge](https://github.com/Azure/iotedge/tree/master/edge-modules) para ver ejemplos de código, incluido el módulo de temperatura simulada.

Continúe con los siguientes tutoriales para obtener información sobre cómo Azure IoT Edge puede ayudarle a implementar servicios en la nube de Azure para procesar y analizar datos en el perímetro.

> [!div class="nextstepaction"]
> [Functions](tutorial-deploy-function.md)
> [Stream Analytics](tutorial-deploy-stream-analytics.md)
> [Machine Learning](tutorial-deploy-machine-learning.md)
> [Custom Vision Service](tutorial-deploy-custom-vision.md)
