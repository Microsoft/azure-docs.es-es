---
title: Tutorial de desarrollo del módulo de C para Linux en Azure IoT Edge | Microsoft Docs
description: En este tutorial se muestra la creación de un módulo IoT Edge con código C y su implementación en un dispositivo Linux que ejecuta IoT Edge.
services: iot-edge
author: shizn
manager: philmea
ms.author: xshi
ms.date: 07/30/2020
ms.topic: tutorial
ms.service: iot-edge
ms.custom: mvc
ms.openlocfilehash: 4bed27654b3482300d3d102d545b44373c53dde0
ms.sourcegitcommit: 14bf4129a73de2b51a575c3a0a7a3b9c86387b2c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/30/2020
ms.locfileid: "87439797"
---
# <a name="tutorial-develop-a-c-iot-edge-module-for-linux-devices"></a>Tutorial: Desarrollo de un módulo IoT Edge con C para dispositivos Linux

Use Visual Studio Code para desarrollar código de C e implementarlo en un dispositivo Linux que ejecute Azure IoT Edge.

Los módulos de IoT Edge se pueden usar para implementar código que, a su vez, implementa una lógica de negocios directamente en los dispositivos de IoT Edge. En este tutorial, se detallan los pasos para crear e implementar un módulo de IoT Edge que filtra los datos de sensor. En este tutorial, aprenderá a:

> [!div class="checklist"]
>
> * Usar Visual Studio Code para crear un módulo de IoT Edge en C.
> * Utilizar Visual Studio Code y Docker para crear una imagen de Docker y publicarla en un contenedor de registro.
> * Implementación del módulo en el dispositivo IoT Edge
> * Visualización de datos generados

El módulo IoT Edge que creó en este tutorial filtra lo datos sobre la temperatura generados por el dispositivo. Solo envía mensajes a los niveles superiores si la temperatura sobrepasa el umbral especificado. Este tipo de análisis perimetral resulta útil para reducir la cantidad de datos que se comunican a la nube y se almacenan en ella.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="solution-scope"></a>Ámbito de la solución

Este tutorial muestra cómo desarrollar un módulo en **C** mediante **Visual Studio Code** y cómo se implementa en un **dispositivo Linux**. Si va a desarrollar módulos para dispositivos Windows, vaya a [Desarrollo de un módulo IoT Edge en C para dispositivos Windows](tutorial-c-module-windows.md) en su lugar.

Utilice la tabla siguiente si desea conocer las opciones para desarrollar e implementar módulos de C en Linux:

| C | Visual Studio Code | Visual Studio |
| - | ------------------ | ------------- |
| **Linux AMD64** | ![Usar VS Code para los módulos de C en Linux AMD64](./media/tutorial-c-module/green-check.png) | ![Usar VS para los módulos de C en Linux AMD64](./media/tutorial-c-module/green-check.png) |
| **Linux ARM32** | ![Usar VS Code para los módulos de C en Linux ARM32](./media/tutorial-c-module/green-check.png) | ![Usar VS para los módulos de C en Linux ARM32](./media/tutorial-c-module/green-check.png) |

## <a name="prerequisites"></a>Prerrequisitos

Antes de comenzar este tutorial, debe haber realizado el anterior para configurar el entorno de desarrollo de contenedores de Linux: [Desarrollo de módulos IoT Edge para dispositivos Linux](tutorial-develop-for-linux.md). Al completar ese tutorial, se deben cumplir los siguientes requisitos previos:

* Una instancia de [IoT Hub](../iot-hub/iot-hub-create-through-portal.md) de nivel estándar o gratis en Azure.
* Un [dispositivo Linux que ejecute Azure IoT Edge](quickstart-linux.md).
* Un registro de contenedor, como [Azure Container Registry](https://docs.microsoft.com/azure/container-registry/).
* [Visual Studio Code](https://code.visualstudio.com/) configurado con [Azure IoT Tools](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-tools).
* [CE de Docker](https://docs.docker.com/install/) configurado para ejecutar contenedores de Linux.

Para desarrollar un módulo IoT Edge en C, instale los siguientes requisitos previos adicionales en la máquina de desarrollo:

* [Extensión de C/C++](https://marketplace.visualstudio.com/items?itemName=ms-vscode.cpptools) para Visual Studio Code.

## <a name="create-a-module-project"></a>Creación de un proyecto de módulo

En los siguientes pasos creará un proyecto de módulo IoT Edge para C mediante Visual Studio Code y la extensión Azure IoT Tools. Una vez que se crea una plantilla de proyecto, se agrega código nuevo para que el módulo filtre los mensajes en función de sus propiedades notificadas.

### <a name="create-a-new-project"></a>Creación de un nuevo proyecto

Cree una plantilla de solución de C que pueda personalizar con su propio código.

1. Seleccione **Ver** > **Paleta de comandos** para abrir la paleta de comandos de VS Code.

2. En la paleta de comandos, escriba y ejecute el comando **Azure: Sign in** (Azure: iniciar sesión) y siga las instrucciones para iniciar sesión en la cuenta de Azure. Si ya ha iniciado sesión, puede omitir este paso.

3. En la paleta de comandos, escriba y ejecute el comando **Azure IoT Edge: New IoT Edge solution** (Azure IoT Edge: nueva solución de IoT Edge). Para crear la solución, siga las indicaciones de la paleta de comandos.

   | Campo | Value |
   | ----- | ----- |
   | Seleccionar carpeta | Elija la ubicación en el equipo de desarrollo en la que VS Code creará los archivos de la solución. |
   | Proporcionar un nombre de la solución | Escriba un nombre descriptivo para la solución o acepte el valor predeterminado **EdgeSolution**. |
   | Seleccionar plantilla del módulo | Elija **Módulo C**. |
   | Proporcionar un nombre de módulo | Llame al módulo **CModule**. |
   | Proporcionar repositorio de imágenes de Docker del módulo | Un repositorio de imágenes incluye el nombre del registro de contenedor y el nombre de la imagen de contenedor. La imagen de contenedor se rellena previamente con el nombre que proporcionó en el último paso. Reemplace **localhost:5000** por el valor de **Servidor de inicio de sesión** del registro de contenedor de Azure. Puede recuperar el servidor de inicio de sesión en la página de información general del registro de contenedor en Azure Portal. <br><br> El repositorio de imágenes final es similar a \<registry name\>.azurecr.io/cmodule. |

   ![Especificación del repositorio de imágenes de Docker](./media/tutorial-c-module/repository.png)

### <a name="add-your-registry-credentials"></a>Adición de las credenciales del Registro

El archivo del entorno almacena las credenciales del registro de contenedor y las comparte con el runtime de IoT Edge. El entorno de ejecución necesita estas credenciales para extraer las imágenes privadas e insertarlas en el dispositivo IoT Edge.

La extensión de IoT Edge intenta extraer de Azure las credenciales del registro del contenedor y rellenar con ellas el archivo de entorno. Compruebe si las credenciales ya están incluidas. Si no lo están, agréguelas ahora:

1. En el explorador de VS Code, abra el archivo .env.
2. Actualice los campos con los valores de **nombre de usuario** y **contraseña** que ha copiado del Registro de contenedor de Azure.
3. Guarde este archivo.

### <a name="select-your-target-architecture"></a>Selección de la arquitectura de destino

Actualmente, Visual Studio Code puede desarrollar módulos de C para dispositivos Linux AMD64 y Linux ARM32v7. Debe seleccionar qué arquitectura tiene como destino con cada solución, dado que el contenedor se crea y ejecuta de manera diferente para cada tipo de arquitectura. La opción predeterminada es Linux AMD64.

1. Abra la paleta de comandos y busque **Azure IoT Edge: Set Default Target Platform for Edge Solution** (Establecer la plataforma de destino predeterminada para la solución perimetral), o bien seleccione el icono de acceso directo en la barra lateral en la parte inferior de la ventana.

2. En la paleta de comandos, seleccione la arquitectura de destino en la lista de opciones. Para este tutorial, usamos una máquina virtual Ubuntu como dispositivo IoT Edge, por lo que mantendrá el valor predeterminado **amd64**.

### <a name="update-the-module-with-custom-code"></a>Actualización del módulo con código personalizado

El código de módulo predeterminado recibe mensajes en una cola de entrada y los pasa por la cola de salida. Vamos a agregar código adicional para que el módulo procese los mensajes en el perímetro antes de reenviarlos a IoT Hub. Actualice el módulo para que analice los datos de temperatura de cada mensaje y solo envíe el mensaje a IoT Hub si la temperatura supera un umbral determinado.

1. En este escenario, los datos del sensor están en formato JSON. Para filtrar los mensajes en formato JSON, importe una biblioteca JSON para C. En este tutorial se usa Parson.

   1. Descargue el [repositorio de Parson GitHub](https://github.com/kgabis/parson). Copie los archivos **parson.c** y **parson.h** en la carpeta **CModule**.

   2. Abra los **módulos** > **CModule** > **CMakeLists.txt**. En la parte superior del archivo, importe los archivos de Parson como una biblioteca denominada **my_parson**.

      ```txt
      add_library(my_parson
          parson.c
          parson.h
      )
      ```

   3. Agregue `my_parson` a la lista de bibliotecas en la función **target_link_libraries** de CMakeLists.txt.

   4. Guarde el archivo **CMakeLists.txt**.

   5. Abra los **módulos** > **CModule** > **main.c**. Al final de la lista de instrucciones include, agregue una nueva para incluir `parson.h` con el fin de lograr compatibilidad con JSON:

      ```c
      #include "parson.h"
      ```

1. En el archivo **main.c**, agregue una variable global denominada `temperatureThreshold` después de la sección de include. Esta variable establece el valor que debe superar la temperatura medida para que los datos se envíen a IoT Hub.

    ```c
    static double temperatureThreshold = 25;
    ```

1. Busque la función `CreateMessageInstance` en main.c. Reemplace la instrucción interna if-else por el código siguiente que agrega unas líneas de funcionalidad:

   ```c
       if ((messageInstance->messageHandle = IoTHubMessage_Clone(message)) == NULL)
       {
           free(messageInstance);
           messageInstance = NULL;
       }
       else
       {
           messageInstance->messageTrackingId = messagesReceivedByInput1Queue;
           MAP_HANDLE propMap = IoTHubMessage_Properties(messageInstance->messageHandle);
           if (Map_AddOrUpdate(propMap, "MessageType", "Alert") != MAP_OK)
           {
              printf("ERROR: Map_AddOrUpdate Failed!\r\n");
           }
       }
   ```

   Las nuevas líneas de código en la instrucción else agregan una nueva propiedad al mensaje, que etiqueta el mensaje como una alerta. Este código etiqueta todos los mensajes como alertas, dado que se agregará una funcionalidad que solo envía mensajes a IoT Hub si notifican altas temperaturas.

1. Reemplace toda la función `InputQueue1Callback` por el código siguiente. Esta función implementa el filtro de mensajería real. Cuando se recibe un mensaje, el filtro comprueba si la temperatura notificada supera el umbral. Si es así, se reenvía el mensaje mediante su cola de salida. De lo contrario, se omite el mensaje.

    ```c
    static unsigned char *bytearray_to_str(const unsigned char *buffer, size_t len)
    {
        unsigned char *ret = (unsigned char *)malloc(len + 1);
        memcpy(ret, buffer, len);
        ret[len] = '\0';
        return ret;
    }

    static IOTHUBMESSAGE_DISPOSITION_RESULT InputQueue1Callback(IOTHUB_MESSAGE_HANDLE message, void* userContextCallback)
    {
        IOTHUBMESSAGE_DISPOSITION_RESULT result;
        IOTHUB_CLIENT_RESULT clientResult;
        IOTHUB_MODULE_CLIENT_LL_HANDLE iotHubModuleClientHandle = (IOTHUB_MODULE_CLIENT_LL_HANDLE)userContextCallback;

        unsigned const char* messageBody;
        size_t contentSize;

        if (IoTHubMessage_GetByteArray(message, &messageBody, &contentSize) == IOTHUB_MESSAGE_OK)
        {
            messageBody = bytearray_to_str(messageBody, contentSize);
        } else
        {
            messageBody = "<null>";
        }

        printf("Received Message [%zu]\r\n Data: [%s]\r\n",
                messagesReceivedByInput1Queue, messageBody);

        // Check if the message reports temperatures higher than the threshold
        JSON_Value *root_value = json_parse_string(messageBody);
        JSON_Object *root_object = json_value_get_object(root_value);
        double temperature;
        if (json_object_dotget_value(root_object, "machine.temperature") != NULL && (temperature = json_object_dotget_number(root_object, "machine.temperature")) > temperatureThreshold)
        {
            printf("Machine temperature %f exceeds threshold %f\r\n", temperature, temperatureThreshold);
            // This message should be sent to next stop in the pipeline, namely "output1".  What happens at "outpu1" is determined
            // by the configuration of the Edge routing table setup.
            MESSAGE_INSTANCE *messageInstance = CreateMessageInstance(message);
            if (NULL == messageInstance)
            {
                result = IOTHUBMESSAGE_ABANDONED;
            }
            else
            {
                printf("Sending message (%zu) to the next stage in pipeline\n", messagesReceivedByInput1Queue);

                clientResult = IoTHubModuleClient_LL_SendEventToOutputAsync(iotHubModuleClientHandle, messageInstance->messageHandle, "output1", SendConfirmationCallback, (void *)messageInstance);
                if (clientResult != IOTHUB_CLIENT_OK)
                {
                    IoTHubMessage_Destroy(messageInstance->messageHandle);
                    free(messageInstance);
                    printf("IoTHubModuleClient_LL_SendEventToOutputAsync failed on sending msg#=%zu, err=%d\n", messagesReceivedByInput1Queue, clientResult);
                    result = IOTHUBMESSAGE_ABANDONED;
                }
                else
                {
                    result = IOTHUBMESSAGE_ACCEPTED;
                }
            }
        }
        else
        {
            printf("Not sending message (%zu) to the next stage in pipeline.\r\n", messagesReceivedByInput1Queue);
            result = IOTHUBMESSAGE_ACCEPTED;
        }

        messagesReceivedByInput1Queue++;
        return result;
    }
    ```

1. Agregue una función `moduleTwinCallback`. Este método recibe las actualizaciones sobre las propiedades que se quieren del módulo gemelo y actualiza la variable **temperatureThreshold** para que coincida. Todos los módulos tienen su propio módulo gemelo, que le permite configurar el código que se ejecuta dentro de un módulo directamente desde la nube.

    ```c
    static void moduleTwinCallback(DEVICE_TWIN_UPDATE_STATE update_state, const unsigned char* payLoad, size_t size, void* userContextCallback)
    {
        printf("\r\nTwin callback called with (state=%s, size=%zu):\r\n%s\r\n",
            MU_ENUM_TO_STRING(DEVICE_TWIN_UPDATE_STATE, update_state), size, payLoad);
        JSON_Value *root_value = json_parse_string(payLoad);
        JSON_Object *root_object = json_value_get_object(root_value);
        if (json_object_dotget_value(root_object, "desired.TemperatureThreshold") != NULL) {
            temperatureThreshold = json_object_dotget_number(root_object, "desired.TemperatureThreshold");
        }
        if (json_object_get_value(root_object, "TemperatureThreshold") != NULL) {
            temperatureThreshold = json_object_get_number(root_object, "TemperatureThreshold");
        }
    }
    ```

1. Busque la función `SetupCallbacksForModule`. Reemplace la función por el siguiente código que agrega una instrucción **else if** para comprobar si se ha actualizado el módulo gemelo.

   ```c
   static int SetupCallbacksForModule(IOTHUB_MODULE_CLIENT_LL_HANDLE iotHubModuleClientHandle)
   {
       int ret;

       if (IoTHubModuleClient_LL_SetInputMessageCallback(iotHubModuleClientHandle, "input1", InputQueue1Callback, (void*)iotHubModuleClientHandle) != IOTHUB_CLIENT_OK)
       {
           printf("ERROR: IoTHubModuleClient_LL_SetInputMessageCallback(\"input1\")..........FAILED!\r\n");
           ret = MU_FAILURE;
       }
       else if (IoTHubModuleClient_LL_SetModuleTwinCallback(iotHubModuleClientHandle, moduleTwinCallback, (void*)iotHubModuleClientHandle) != IOTHUB_CLIENT_OK)
       {
           printf("ERROR: IoTHubModuleClient_LL_SetModuleTwinCallback(default)..........FAILED!\r\n");
           ret = MU_FAILURE;
       }
       else
       {
           ret = 0;
       }

       return ret;
   }
   ```

1. Guarde el archivo main.c.

1. En el explorador de VS Code, abra el archivo **deployment.template.json** en el área de trabajo de la solución de IoT Edge.

1. Agregue el módulo gemelo CModule al manifiesto de implementación. Inserte el siguiente contenido JSON en la parte inferior de la sección `moduleContent`, después del módulo gemelo `$edgeHub`:

   ```json
   "CModule": {
       "properties.desired":{
           "TemperatureThreshold":25
       }
   }
   ```

   ![Adición de un gemelo de CModule gemelo a una plantilla de implementación](./media/tutorial-c-module/module-twin.png)

1. Guarde el archivo **deployment.template.json**.

## <a name="build-and-push-your-module"></a>Compilación e inserción del módulo

En la sección anterior ha creado una solución IoT Edge y ha agregado código a CModule que va a filtrar los mensajes y a eliminar aquellos en los que se notifique que la temperatura del equipo está dentro de los límites aceptables. Ahora, tiene que compilar la solución como una imagen de contenedor e insertarla en el registro de contenedor.

1. Abra el terminal de VS Code, para lo que debe seleccionar **View** > **Terminal** (Ver > Terminal).

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

4. En el dispositivo, expanda **Módulos** para ver una lista de módulos implementados y en ejecución. Haga clic en el botón Actualizar. Debería ver el nuevo **CModule** en ejecución junto con el módulo **SimulatedTemperatureSensor**, así como **$edgeAgent** y **$edgeHub**.

    Los módulos pueden tardar unos minutos en iniciarse. El entorno de ejecución de Azure IoT Edge necesita recibir su nuevo manifiesto de implementación, extraer las imágenes de los módulos del entorno de ejecución del contenedor y, después, iniciar cada nuevo módulo.

## <a name="view-generated-data"></a>Visualización de datos generados

Una vez aplicado el manifiesto de implementación al dispositivo de IoT Edge, el entorno de ejecución de IoT Edge del dispositivo recopila la información de implementación nueva y comienza a ejecutarse con ella. Los módulos que se ejecuten en el dispositivo y que no están incluidos en el manifiesto de implementación se detienen. Los módulos que falten en el dispositivo se inician.

1. En el explorador de Visual Studio Code, haga clic con el botón derecho en el nombre del dispositivo IoT Edge y seleccione **Start Monitoring Built-in Event Endpoint** (Iniciar supervisión del punto de conexión del evento integrado).

2. Vea los mensajes que llegan a IoT Hub. Los mensajes pueden tardar un rato en llegar, ya que el dispositivo IoT Edge tiene que recibir su nueva implementación e iniciar todos los módulos. Después, los cambios realizados en el código CModule esperan hasta que la temperatura de la máquina alcanza los 25 grados antes de enviar los mensajes. También agrega el tipo de mensaje **Alerta** a los mensajes que llegan a ese umbral de temperatura.

   ![Visualización de los mensajes que llegan a IoT Hub](./media/tutorial-c-module/view-d2c-message.png)

## <a name="edit-the-module-twin"></a>Edición del módulo gemelo

Usamos el módulo gemelo CModule en el manifiesto de implementación para establecer el umbral de temperatura en 25 grados. Puede usar al módulo gemelo para cambiar la funcionalidad sin tener que actualizar el código del módulo.

1. En Visual Studio Code, expanda los detalles en el dispositivo IoT Edge para ver los módulos en ejecución.

2. Haga clic con el botón derecho en **CModule** y seleccione **Editar módulo gemelo**.

3. Busque **TemperatureThreshold** en las propiedades deseadas. Cambie su valor por una temperatura nueva de 5 a 10 grados más que la última temperatura registrada.

4. Guarde el archivo del módulo gemelo.

5. Haga clic con el botón derecho en cualquier parte del panel de edición del módulo gemelo y seleccione **Actualizar módulo gemelo**.

6. Supervise los mensajes entrantes del dispositivo a la nube. Debería ver que los mensajes se detienen hasta que se alcanza el nuevo umbral de temperatura.

## <a name="clean-up-resources"></a>Limpieza de recursos

Si prevé seguir con el siguiente artículo recomendado, puede mantener los recursos y las configuraciones que ya ha creado y volverlos a utilizar. También puede seguir usando el mismo dispositivo de IoT Edge como dispositivo de prueba.

En caso contrario, para evitar gastos, puede eliminar las configuraciones locales y los recursos de Azure que creó en este artículo.

[!INCLUDE [iot-edge-clean-up-cloud-resources](../../includes/iot-edge-clean-up-cloud-resources.md)]

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha creado un módulo IoT Edge que contiene código para filtrar los datos sin procesar generados por el dispositivo IoT Edge. Cuando esté preparado para crear sus propios módulos, puede aprender más sobre cómo [desarrollar sus propios módulos IoT Edge](module-development.md) y sobre cómo [desarrollar módulos con Visual Studio Code](how-to-vs-code-develop-module.md). Para obtener ejemplos de módulos de IoT Edge, incluido el módulo de temperatura simulado, consulte [Ejemplos de módulos de IoT Edge](https://github.com/Azure/iotedge/tree/master/edge-modules) y [Ejemplos del SDK de C para IoT](https://github.com/Azure/azure-iot-sdk-c/tree/master/iothub_client/samples).

Puede continuar con los siguientes tutoriales para obtener información sobre cómo Azure IoT Edge puede ayudarle a implementar servicios en la nube de Azure para procesar y analizar datos en el perímetro.

> [!div class="nextstepaction"]
> [Functions](tutorial-deploy-function.md)
> [Stream Analytics](tutorial-deploy-stream-analytics.md)
> [Machine Learning](tutorial-deploy-machine-learning.md)
> [Custom Vision Service](tutorial-deploy-custom-vision.md)
