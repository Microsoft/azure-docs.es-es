---
title: Tutorial de desarrollo del módulo C para Windows en Azure IoT Edge | Microsoft Docs
description: En este tutorial se muestra la creación de un módulo de IoT Edge con código C y su implementación en un dispositivo Windows que ejecuta IoT Edge.
services: iot-edge
author: shizn
manager: philmea
ms.author: xshi
ms.date: 05/28/2019
ms.topic: tutorial
ms.service: iot-edge
ms.custom: mvc
ms.openlocfilehash: 2da31944a58fb3e5834938b7de32348f30ed7e25
ms.sourcegitcommit: 14bf4129a73de2b51a575c3a0a7a3b9c86387b2c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/30/2020
ms.locfileid: "87439810"
---
# <a name="tutorial-develop-a-c-iot-edge-module-for-windows-devices"></a>Tutorial: Desarrollo de un módulo de IoT Edge en C para dispositivos Windows

Use Visual Studio para desarrollar código C e implementarlo en un dispositivo de Windows que ejecute Azure IoT Edge.

Los módulos Azure IoT Edge se pueden usar para implementar código que, a su vez, implementa una lógica de negocios directamente en los dispositivos IoT Edge. En este tutorial, se detallan los pasos para crear e implementar un módulo de IoT Edge que filtra los datos de sensor. En este tutorial, aprenderá a:

> [!div class="checklist"]
>
> * Usar Visual Studio Code para crear un módulo de IoT Edge basado en el SDK de C.
> * Usar Visual Studio y Docker para crear una imagen de Docker y publicarla en el Registro.
> * Implementar el módulo en el dispositivo IoT Edge.
> * Ver datos generados.

El módulo IoT Edge que creó en este tutorial filtra lo datos sobre la temperatura generados por el dispositivo. Solo envía mensajes a los niveles superiores si la temperatura sobrepasa el umbral especificado. Este tipo de análisis perimetral resulta útil para reducir la cantidad de datos que se comunican a la nube y se almacenan en ella.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="solution-scope"></a>Ámbito de la solución

Este tutorial muestra cómo desarrollar un módulo en **C** mediante **Visual Studio 2019** y cómo se implementa en un **dispositivo Windows**. Si va a desarrollar módulos para dispositivos Linux, vaya a [Desarrollo de un módulo de IoT Edge en C para dispositivos Linux](tutorial-c-module.md) en su lugar.

Utilice la tabla siguiente si desea conocer las opciones para desarrollar e implementar módulos de C para dispositivos de Windows:

| C | Visual Studio Code | Visual Studio 2017/2019 |
| -- | ------------------ | ------------------ |
| **Windows AMD64** |  | ![Desarrollo de módulos C para WinAMD64 en Visual Studio](./media/tutorial-c-module/green-check.png) |

## <a name="prerequisites"></a>Prerrequisitos

Antes de comenzar este tutorial, debe haber realizado el anterior para configurar el entorno de desarrollo de contenedores de Windows: [Desarrollo de módulos de IoT Edge para dispositivos Windows](tutorial-develop-for-windows.md). Después de completar este tutorial, se deben cumplir los siguientes requisitos previos:

* Una instancia de [IoT Hub](../iot-hub/iot-hub-create-through-portal.md) de nivel estándar o gratis en Azure.
* Un [dispositivo de Windows que ejecute Azure IoT Edge](quickstart.md).
* Un registro de contenedor, como [Azure Container Registry](https://docs.microsoft.com/azure/container-registry/).
* [Visual Studio 2019](https://docs.microsoft.com/visualstudio/install/install-visual-studio) configurado con la extensión [Azure IoT Edge Tools](https://marketplace.visualstudio.com/items?itemName=vsc-iot.vs16iotedgetools).
* [Docker Desktop](https://docs.docker.com/docker-for-windows/install/) configurado para ejecutar contenedores de Windows.
* Instale el SDK de C para Azure IoT para Windows x64 a mediante vcpkg:

   ```powershell
   git clone https://github.com/Microsoft/vcpkg
   cd vcpkg
   .\bootstrap-vcpkg.bat
   .\vcpkg install azure-iot-sdk-c:x64-windows
   .\vcpkg --triplet x64-windows integrate install
   ```

> [!TIP]
> Si usa Visual Studio 2017 (versión 15.7 o superior), descargue e instale [Azure IoT Edge Tools](https://marketplace.visualstudio.com/items?itemName=vsc-iot.vsiotedgetools) para VS 2017 desde Visual Studio Marketplace.

## <a name="create-a-module-project"></a>Creación de un proyecto de módulo

En los siguientes pasos creará un proyecto de módulo de IoT Edge basado en el SDK de C mediante Visual Studio y la extensión Azure IoT Edge Tools. Una vez que se crea una plantilla de proyecto, se agrega código nuevo para que el módulo filtre los mensajes en función de sus propiedades notificadas.

### <a name="create-a-new-project"></a>Creación de un nuevo proyecto

Cree una plantilla de solución de C que pueda personalizar con su propio código.

1. Inicie Visual Studio 2019 y seleccione **Creación de un proyecto**.

2. Busque **IoT Edge** y seleccione el proyecto **Azure IoT Edge (Windows amd64)** . Haga clic en **Next**.

   ![Creación de un nuevo proyecto de Azure IoT Edge](./media/tutorial-c-module-windows/new-project.png)

3. Cambie el nombre del proyecto y de la solución por otro descriptivo como **CTutorialApp**. Haga clic en **Crear** para crear el proyecto.

   ![Configuración de un nuevo proyecto de Azure IoT Edge](./media/tutorial-c-module-windows/configure-project.png)

4. Configure el proyecto con los siguientes valores:

   | Campo | Value |
   | ----- | ----- |
   | Seleccione una plantilla: | Seleccione **Módulo C**. |
   | Nombre del proyecto de módulo | Llame al módulo **CModule**. |
   | Repositorio de imágenes de Docker | Un repositorio de imágenes incluye el nombre del registro de contenedor y el nombre de la imagen de contenedor. La imagen de contenedor se rellena previamente con el valor del nombre del proyecto del módulo. Reemplace **localhost:5000** por el valor de **Servidor de inicio de sesión** del registro de contenedor de Azure. Puede recuperar el servidor de inicio de sesión en la página de información general del registro de contenedor en Azure Portal. <br><br> El repositorio de imágenes final es similar a \<registry name\>.azurecr.io/cmodule. |

   ![Configuración del proyecto para el dispositivo de destino, el tipo de módulo y el registro de contenedor](./media/tutorial-c-module-windows/add-application-and-module.png)

5. Haga clic en **Aceptar** para crear el proyecto.

### <a name="add-your-registry-credentials"></a>Adición de las credenciales del Registro

El manifiesto de implementación comparte las credenciales del registro de contenedor con el entorno de ejecución de Azure IoT Edge. El entorno de ejecución necesita estas credenciales para extraer las imágenes privadas e insertarlas en el dispositivo IoT Edge. Use las credenciales de la sección **Claves de acceso** de Azure Container Registry.

1. En el explorador de soluciones de Visual Studio, abra el archivo **deployment.template.json**.

2. Busque la propiedad **registryCredentials** en las propiedades $edgeAgent que desee. La dirección del registro se rellenará automáticamente con la información que proporcionó al crear el proyecto. Los campos de nombre de usuario y contraseña deben contener nombres de variable. Por ejemplo:

   ```json
   "registryCredentials": {
     "<registry name>": {
       "username": "$CONTAINER_REGISTRY_USERNAME_<registry name>",
       "password": "$CONTAINER_REGISTRY_PASSWORD_<registry name>",
       "address": "<registry name>.azurecr.io"
     }
   }
   ```

3. Abra el archivo **.env** en la solución del módulo. (Está oculto de forma predeterminada en el Explorador de soluciones, por lo que es posible que tenga que seleccionar el botón **Mostrar todos los archivos** para mostrarlo). El archivo .env debe contener las mismas variables de nombre de usuario y contraseña que vio en el archivo deployment.template.json.

4. Agregue los valores de **Nombre de usuario** y **Contraseña** del registro de contenedor de Azure.

5. Guarde los cambios en el archivo .env.

### <a name="update-the-module-with-custom-code"></a>Actualización del módulo con código personalizado

El código de módulo predeterminado recibe mensajes en una cola de entrada y los pasa por la cola de salida. Vamos a agregar código adicional para que el módulo procese los mensajes en el perímetro antes de reenviarlos a IoT Hub. Actualice el módulo para que analice los datos de temperatura de cada mensaje y solo envíe el mensaje a IoT Hub si la temperatura supera un umbral determinado.

1. En este escenario, los datos del sensor están en formato JSON. Para filtrar los mensajes en formato JSON, importe una biblioteca JSON para C. En este tutorial se usa Parson.

   1. Descargue el [repositorio de Parson GitHub](https://github.com/kgabis/parson). Copie los archivos **parson.c** y **parson.h** en el proyecto **CModule**.

   2. En Visual Studio, abra el archivo **CMakeLists.txt** desde la carpeta del proyecto CModule. En la parte superior del archivo, importe los archivos de Parson como una biblioteca denominada **my_parson**.

      ```txt
      add_library(my_parson
          parson.c
          parson.h
      )
      ```

   3. Agregue `my_parson` a la lista de bibliotecas de la sección **target_link_libraries** del archivo CMakeLists.txt.

   4. Guarde el archivo **CMakeLists.txt**.

   5. Abra **CModule** > **main.c**. Al final de la lista de instrucciones include, agregue una nueva para incluir `parson.h` con el fin de lograr compatibilidad con JSON:

      ```c
      #include "parson.h"
      ```

2. En el archivo **main.c**, agregue una variable global denominada `temperatureThreshold` junto a la variable messagesReceivedByInput1Queue. Esta variable establece el valor que debe superar la temperatura medida para que los datos se envíen a IoT Hub.

    ```c
    static double temperatureThreshold = 25;
    ```

3. Busque la función `CreateMessageInstance` en main.c. Reemplace la instrucción interna if-else por el código siguiente que agrega unas líneas de funcionalidad:

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

4. Busque la función `InputQueue1Callback` y reemplace toda la función por el código siguiente. Esta función implementa el filtro de mensajería real. Cuando se recibe un mensaje, el filtro comprueba si la temperatura notificada supera el umbral. Si es así, se reenvía el mensaje mediante su cola de salida. De lo contrario, se omite el mensaje.

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

        // If temperature exceeds threshold, send to output1
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
        // If message does not exceed threshold, do not forward
        else
        {
            printf("Not sending message (%zu) to the next stage in pipeline.\r\n", messagesReceivedByInput1Queue);
            result = IOTHUBMESSAGE_ACCEPTED;
        }

        messagesReceivedByInput1Queue++;
        return result;
    }
    ```

5. Agregue una función `moduleTwinCallback`. Este método recibe las actualizaciones sobre las propiedades que se quieren del módulo gemelo y actualiza la variable **temperatureThreshold** para que coincida. Todos los módulos tienen su propio módulo gemelo, que le permite configurar el código que se ejecuta dentro de un módulo directamente desde la nube.

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

6. Busque la función `SetupCallbacksForModule`. Reemplace la función por el siguiente código que agrega una instrucción **else if** para comprobar si se ha actualizado el módulo gemelo.

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

7. Guarde el archivo main.c.

8. Abra el archivo **deployment.template.json**.

9. Agregue el módulo gemelo CModule al manifiesto de implementación. Inserte el siguiente contenido JSON en la parte inferior de la sección `moduleContent`, después del módulo gemelo `$edgeHub`:

   ```json
   "CModule": {
       "properties.desired":{
           "TemperatureThreshold":25
       }
   }
   ```

   ![Adición de un gemelo de CModule gemelo a una plantilla de implementación](./media/tutorial-c-module-windows/module-twin.png)

10. Guarde el archivo **deployment.template.json**.

## <a name="build-and-push-your-module"></a>Compilación e inserción del módulo

En la sección anterior, creó una solución de IoT Edge y agregó código a **CModule** para filtrar los mensajes en los que la temperatura registrada por la máquina está por debajo del umbral aceptable. Ahora, tiene que compilar la solución como una imagen de contenedor e insertarla en el registro de contenedor.

### <a name="sign-in-to-docker"></a>Inicio de sesión en Docker

Especifique las credenciales del registro de contenedor para Docker en el equipo de desarrollo, con el fin de que pueda insertar la imagen de contenedor para que se almacene en el registro.

1. Abra PowerShell o un símbolo del sistema.

2. Inicie sesión en Docker con las credenciales del registro de contenedor de Azure que guardó después de crear el registro.

   ```cmd
   docker login -u <ACR username> -p <ACR password> <ACR login server>
   ```

   Puede recibir una advertencia de seguridad en la que se recomiende el uso de `--password-stdin`. Aunque ese procedimiento se recomienda para escenarios de producción, está fuera del ámbito de este tutorial. Para más información, consulte la referencia de [docker login](https://docs.docker.com/engine/reference/commandline/login/#provide-a-password-using-stdin).

### <a name="build-and-push"></a>Compilación e inserción

La máquina de desarrollo ahora tiene acceso al registro de contenedores y también los dispositivos IoT Edge. Es hora de convertir el código del proyecto en una imagen de contenedor.

1. En el Explorador de soluciones de Visual Studio, haga clic con el botón derecho en el nombre del proyecto que desea compilar. El nombre predeterminado es **AzureIotEdgeApp1**. En este tutorial, se ha elegido el nombre **CTutorialApp**. Puesto que va a compilar un módulo de Windows, la extensión debe ser **Windows.Amd64**.

2. Seleccione **Build and Push IoT Edge Modules** (Compilación e inserción de los módulos de IoT Edge).

   El comando de compilación e inserción inicia tres operaciones. En primer lugar, se crea una nueva carpeta en la solución denominada **config** que contiene los archivos del manifiesto de la implementación completa, con la información de la plantilla de implementación y otros archivos de la solución. En segundo lugar, ejecuta `docker build` para generar la imagen de contenedor basándose en el Dockerfile adecuado para la arquitectura de destino. A continuación, ejecuta `docker push` para insertar el repositorio de imágenes en el registro de contenedor.

   Este proceso puede tardar varios minutos la primera vez, pero es más rápido la próxima vez que ejecute los comandos.

## <a name="deploy-modules-to-device"></a>Implementación de módulos en el dispositivo

Utilice el explorador en la nube de Visual Studio y la extensión Azure IoT Edge Tools para implementar el proyecto de módulo en el dispositivo IoT Edge. Ya tiene un manifiesto de implementación preparado para su escenario, el archivo **deployment.windows-amd64.json** de la carpeta config. Ahora todo lo que necesita hacer es seleccionar un dispositivo que reciba la implementación.

Asegúrese de que el dispositivo IoT Edge está en funcionamiento.

1. En el explorador en la nube de Visual Studio, expanda los recursos para ver la lista de dispositivos de IoT.

2. Haga clic con el botón derecho en el nombre del dispositivo IoT Edge que quiere que reciba la implementación.

3. Seleccione **Create Deployment** (Crear implementación).

4. En el explorador de archivos, seleccione el archivo **deployment.windows-amd64** en la carpeta de configuración de la solución.

5. Actualice el explorador en la nube para ver los módulos implementados enumerados en el dispositivo.

## <a name="view-generated-data"></a>Visualización de datos generados

Una vez aplicado el manifiesto de implementación al dispositivo de IoT Edge, el entorno de ejecución de IoT Edge del dispositivo recopila la información de implementación nueva y comienza a ejecutarse con ella. Los módulos que se ejecuten en el dispositivo y que no están incluidos en el manifiesto de implementación se detienen. Los módulos que falten en el dispositivo se inician.

Puede usar la extensión IoT Edge Tools para ver los mensajes a medida que llegan a IoT Hub.

1. En el explorador en la nube de Visual Studio, seleccione el nombre del dispositivo de IoT Edge.

2. En la lista **Acciones**, seleccione **Start Monitoring Built-in Event Endpoint** (Iniciar supervisión del punto de conexión del evento integrado).

3. Vea los mensajes que llegan a IoT Hub. Los mensajes pueden tardar un rato en llegar, ya que el dispositivo IoT Edge tiene que recibir su nueva implementación e iniciar todos los módulos. Después, los cambios realizados en el código CModule esperan hasta que la temperatura de la máquina alcanza los 25 grados antes de enviar los mensajes. También agrega el tipo de mensaje **Alerta** a los mensajes que llegan a ese umbral de temperatura.

   ![Visualización de los mensajes que llegan a IoT Hub](./media/tutorial-c-module-windows/view-d2c-message.png)

## <a name="edit-the-module-twin"></a>Edición del módulo gemelo

El módulo gemelo CModule se usa para establecer el umbral de temperatura en 25 grados. Puede usar al módulo gemelo para cambiar la funcionalidad sin tener que actualizar el código del módulo.

1. En Visual Studio, abra el archivo **deployment.windows amd64.json**. (No el archivo deployment.template. Si no ve el manifiesto de implementación en el archivo de configuración del Explorador de soluciones, seleccione el icono **Mostrar todos los archivos** en la barra de herramientas del explorador).

2. Busque el gemelo CModule y cambie el valor del parámetro **temperatureThreshold** por una temperatura 5 a 10 grados mayor que la última notificada.

3. Guarde el archivo **deployment.windows amd64.json**.

4. Siga los pasos de implementación de nuevo para aplicar el manifiesto de implementación actualizado al dispositivo.

5. Supervise los mensajes entrantes del dispositivo a la nube. Debería ver que los mensajes se detienen hasta que se alcanza el nuevo umbral de temperatura.

## <a name="clean-up-resources"></a>Limpieza de recursos

Si prevé seguir con el siguiente artículo recomendado, puede mantener los recursos y las configuraciones que ya ha creado y volverlos a utilizar. También puede seguir usando el mismo dispositivo de IoT Edge como dispositivo de prueba.

En caso contrario, para evitar gastos, puede eliminar las configuraciones locales y los recursos de Azure que creó en este artículo.

[!INCLUDE [iot-edge-clean-up-cloud-resources](../../includes/iot-edge-clean-up-cloud-resources.md)]

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha creado un módulo IoT Edge que contiene código para filtrar datos sin procesar generados por el dispositivo IoT Edge. Cuando esté preparado para crear sus propios módulos, puede aprender más sobre cómo [desarrollar sus propios módulos IoT Edge](module-development.md) o cómo [desarrollar módulos con Visual Studio](how-to-visual-studio-develop-module.md). Para obtener ejemplos de módulos de IoT Edge, incluido el módulo de temperatura simulado, consulte [Ejemplos de módulos de IoT Edge](https://github.com/Azure/iotedge/tree/master/edge-modules) y [Ejemplos del SDK de C para IoT](https://github.com/Azure/azure-iot-sdk-c/tree/master/iothub_client/samples).

Puede continuar con los siguientes tutoriales para obtener información sobre cómo Azure IoT Edge puede ayudarle a implementar servicios en la nube de Azure para procesar y analizar datos en el perímetro.

> [!div class="nextstepaction"]
> [Functions](tutorial-deploy-function.md)
> [Stream Analytics](tutorial-deploy-stream-analytics.md)
> [Machine Learning](tutorial-deploy-machine-learning.md)
> [Custom Vision Service](tutorial-deploy-custom-vision.md)
