---
title: 'Tutorial: Desarrollo del módulo de C# para Windows mediante Azure IoT Edge'
description: En este tutorial se muestra la creación de un módulo de IoT Edge con código C# y su implementación en un dispositivo perimetral Windows IoT Edge.
services: iot-edge
author: kgremban
manager: philmea
ms.author: kgremban
ms.date: 08/03/2020
ms.topic: tutorial
ms.service: iot-edge
ms.custom:
- mvc
- amqp
ms.openlocfilehash: ba438971f3499b0d4f34affee8b4e4d0051ff828
ms.sourcegitcommit: 85eb6e79599a78573db2082fe6f3beee497ad316
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/05/2020
ms.locfileid: "87809941"
---
# <a name="tutorial-develop-a-c-iot-edge-module-for-windows-devices"></a>Tutorial: Desarrollo de un módulo de IoT Edge en C# para dispositivos Windows

Use Visual Studio para desarrollar código C# e implementarlo en un dispositivo de Windows que ejecute Azure IoT Edge.

Los módulos Azure IoT Edge se pueden usar para implementar código que, a su vez, implementa una lógica de negocios directamente en los dispositivos IoT Edge. En este tutorial, se detallan los pasos para crear e implementar un módulo de IoT Edge que filtra los datos de sensor. En este tutorial, aprenderá a:

> [!div class="checklist"]
>
> * Usar Visual Studio Code para crear un módulo de IoT Edge basado en el SDK de C#.
> * Usar Visual Studio y Docker para crear una imagen de Docker y publicarla en el Registro.
> * Implementar el módulo en el dispositivo IoT Edge.
> * Ver datos generados.

El módulo IoT Edge que creó en este tutorial filtra lo datos sobre la temperatura generados por el dispositivo. Solo envía mensajes a los niveles superiores si la temperatura sobrepasa el umbral especificado. Este tipo de análisis perimetral resulta útil para reducir la cantidad de datos que se comunican a la nube y se almacenan en ella.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="solution-scope"></a>Ámbito de la solución

Este tutorial muestra cómo desarrollar un módulo en **C#** mediante **Visual Studio 2019** y cómo se implementa en un **dispositivo Windows**. Si va a desarrollar módulos para los dispositivos de Linux, vaya a [Desarrollo de un módulo de IoT Edge en C# para dispositivos Linux](tutorial-csharp-module.md) en su lugar.

Utilice la tabla siguiente si desea conocer las opciones para desarrollar e implementar módulos de C# para dispositivos Windows:

| C# | Visual Studio Code | Visual Studio 2017/2019 |
| -- | ------------------ | ------------------ |
| **Desarrollo de Windows AMD64** | ![Desarrollo de módulos C# para WinAMD64 en VS Code](./media/tutorial-c-module/green-check.png) | ![Desarrollo de módulos C# para WinAMD64 en Visual Studio](./media/tutorial-c-module/green-check.png) |
| **Depuración de Windows AMD64** |   | ![Depuración de módulos C# para WinAMD64 en Visual Studio](./media/tutorial-c-module/green-check.png) |

## <a name="prerequisites"></a>Prerrequisitos

Antes de comenzar este tutorial, debe haber realizado el anterior para configurar el entorno de desarrollo, [Desarrollo de un módulo IoT Edge para un dispositivo Windows](tutorial-develop-for-windows.md). Después de completar este tutorial, ya debe tener los siguientes requisitos previos:

* Una instancia de [IoT Hub](../iot-hub/iot-hub-create-through-portal.md) de nivel estándar o gratis en Azure.
* Un [dispositivo de Windows que ejecute Azure IoT Edge](quickstart.md).
* Un registro de contenedor, como [Azure Container Registry](https://docs.microsoft.com/azure/container-registry/).
* [Visual Studio 2019](https://docs.microsoft.com/visualstudio/install/install-visual-studio) configurado con la extensión [Azure IoT Edge Tools](https://marketplace.visualstudio.com/items?itemName=vsc-iot.vs16iotedgetools).
* [Docker Desktop](https://docs.docker.com/docker-for-windows/install/) configurado para ejecutar contenedores de Windows.

> [!TIP]
> Si usa Visual Studio 2017 (versión 15.7 o superior), descargue e instale [Azure IoT Edge Tools](https://marketplace.visualstudio.com/items?itemName=vsc-iot.vsiotedgetools) para VS 2017 desde Visual Studio Marketplace.

## <a name="create-a-module-project"></a>Creación de un proyecto de módulo

En los siguientes pasos creará un proyecto de módulo de IoT Edge mediante Visual Studio y la extensión de Azure IoT Edge Tools. Una vez que se crea una plantilla de proyecto, se agrega código nuevo para que el módulo filtre los mensajes en función de sus propiedades notificadas.

### <a name="create-a-new-project"></a>Creación de un nuevo proyecto

Azure IoT Edge Tools proporciona plantillas de proyecto para todos los idiomas admitidos del módulo IoT Edge en Visual Studio. Estas plantillas tienen todos los archivos y el código que se necesita para implementar un módulo de trabajo y probar IoT Edge o le proporcionan a un punto de partida para personalizar la plantilla con su propia lógica de negocios.

1. Inicie Visual Studio 2019 y seleccione **Creación de un proyecto**.

2. Busque **IoT Edge** y seleccione el proyecto **Azure IoT Edge (Windows amd64)** . Haga clic en **Next**.

   ![Creación de un nuevo proyecto de Azure IoT Edge](./media/tutorial-csharp-module-windows/new-project.png)

3. Cambie el nombre del proyecto y de la solución por otro descriptivo como **ApTutorialCSharp**. Haga clic en **Crear** para crear el proyecto.

   ![Configuración de un nuevo proyecto de Azure IoT Edge](./media/tutorial-csharp-module-windows/configure-project.png)

4. Configure el proyecto con los siguientes valores:

   | Campo | Value |
   | ----- | ----- |
   | Seleccione una plantilla: | Seleccione **Módulo C#** . |
   | Nombre del proyecto de módulo | Asigne al módulo el nombre **CSharpModule**. |
   | Repositorio de imágenes de Docker | Un repositorio de imágenes incluye el nombre del registro de contenedor y el nombre de la imagen de contenedor. La imagen de contenedor se rellena previamente con el valor del nombre del proyecto del módulo. Reemplace **localhost:5000** por el valor de **Servidor de inicio de sesión** del registro de contenedor de Azure. Puede recuperar el servidor de inicio de sesión en la página de información general del registro de contenedor en Azure Portal. <br><br> El repositorio de imágenes final es similar a este: \<registry name\>.azurecr.io/cmodule. |

   ![Configuración del proyecto para el dispositivo de destino, el tipo de módulo y el registro de contenedor](./media/tutorial-csharp-module-windows/add-application-and-module.png)

5. Haga clic en **Aceptar** para crear el proyecto.

### <a name="add-your-registry-credentials"></a>Adición de las credenciales del Registro

El manifiesto de implementación comparte las credenciales del registro de contenedor con el entorno de ejecución de Azure IoT Edge. El entorno de ejecución necesita estas credenciales para extraer las imágenes privadas e insertarlas en el dispositivo IoT Edge. Use las credenciales de la sección **Claves de acceso** de Azure Container Registry.

1. En el explorador de soluciones de Visual Studio, abra el archivo **deployment.template.json**.

2. Busque la propiedad **registryCredentials** en las propiedades $edgeAgent que desee. La dirección del registro se rellenará automáticamente con la información que proporcionó al crear el proyecto y, entonces, los campos de nombre de usuario y contraseña contendrán nombres de variable. Por ejemplo:

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

1. En Visual Studio, abra **CSharpModule** > **Program.cs**.

2. En la parte superior del espacio de nombres **CSharpModule**, agregue tres instrucciones **using** para los tipos que se usarán más adelante en:

    ```csharp
    using System.Collections.Generic;     // For KeyValuePair<>
    using Microsoft.Azure.Devices.Shared; // For TwinCollection
    using Newtonsoft.Json;                // For JsonConvert
    ```

3. Agregue la variable **temperatureThreshold** a la clase **Program** tras la variable del contador. La variable temperatureThreshold establece el valor que debe superar la temperatura medida para que los datos se envíen al centro de IoT.

    ```csharp
    static int temperatureThreshold { get; set; } = 25;
    ```

4. Agregue las clases **MessageBody**, **Machine** y **Ambient** a la clase **Program** tras las declaraciones de variables. Estas clases definen el esquema esperado para el cuerpo de los mensajes entrantes.

    ```csharp
    class MessageBody
    {
        public Machine machine {get;set;}
        public Ambient ambient {get; set;}
        public string timeCreated {get; set;}
    }
    class Machine
    {
        public double temperature {get; set;}
        public double pressure {get; set;}
    }
    class Ambient
    {
        public double temperature {get; set;}
        public int humidity {get; set;}
    }
    ```

5. Busque el método **Init**. Este método crea y configura un objeto **ModuleClient**, que permite al módulo conectarse al entorno de ejecución de Azure IoT Edge local para enviar y recibir mensajes. El código también registra una devolución de llamada para recibir mensajes desde el centro de IoT Edge a través del punto de conexión **input1**.

   Reemplace el método Init por el código siguiente:

   ```csharp
   static async Task Init()
   {
       AmqpTransportSettings amqpSetting = new AmqpTransportSettings(TransportType.Amqp_Tcp_Only);
       ITransportSettings[] settings = { amqpSetting };

       // Open a connection to the Edge runtime
       ModuleClient ioTHubModuleClient = await ModuleClient.CreateFromEnvironmentAsync(settings);
       await ioTHubModuleClient.OpenAsync();
       Console.WriteLine("IoT Hub module client initialized.");

       // Read the TemperatureThreshold value from the module twin's desired properties
       var moduleTwin = await ioTHubModuleClient.GetTwinAsync();
       await OnDesiredPropertiesUpdate(moduleTwin.Properties.Desired, ioTHubModuleClient);

       // Attach a callback for updates to the module twin's desired properties.
       await ioTHubModuleClient.SetDesiredPropertyUpdateCallbackAsync(OnDesiredPropertiesUpdate, null);

       // Register a callback for messages that are received by the module.
       await ioTHubModuleClient.SetInputMessageHandlerAsync("input1", FilterMessages, ioTHubModuleClient);
   }
   ```

   Este método Init actualizado todavía configura la conexión con el entorno de ejecución de Azure IoT Edge con ModuleClient, pero también agrega nueva funcionalidad. Lee las propiedades deseadas del gemelo del módulo para recuperar el valor **temperatureThreshold**. A continuación, crea una devolución de llamada que realiza escuchas para las actualizaciones futuras de las propiedades deseadas del módulo gemelo. Con esta devolución de llamada, puede actualizar el umbral de temperatura en el módulo gemelo de forma remota y los cambios se incorporarán al módulo.

   El método Init actualizado también cambia el método existente **SetInputMessageHandlerAsync**. En el código de ejemplo, los mensajes entrantes en *input1* se procesan con la función *PipeMessage*, pero deseamos cambiar este valor para utilizar la función *FilterMessages* que crearemos en los pasos siguientes.

6. Agregue el nuevo método **onDesiredPropertiesUpdate** a la clase **Program**. Este método recibe las actualizaciones sobre las propiedades que se quieren del módulo gemelo y actualiza la variable **temperatureThreshold** para que coincida. Todos los módulos tienen su propio módulo gemelo, que le permite configurar el código que se ejecuta dentro de un módulo directamente desde la nube.

    ```csharp
    static Task OnDesiredPropertiesUpdate(TwinCollection desiredProperties, object userContext)
    {
        try
        {
            Console.WriteLine("Desired property change:");
            Console.WriteLine(JsonConvert.SerializeObject(desiredProperties));

            if (desiredProperties["TemperatureThreshold"]!=null)
                temperatureThreshold = desiredProperties["TemperatureThreshold"];

        }
        catch (AggregateException ex)
        {
            foreach (Exception exception in ex.InnerExceptions)
            {
                Console.WriteLine();
                Console.WriteLine("Error when receiving desired property: {0}", exception);
            }
        }
        catch (Exception ex)
        {
            Console.WriteLine();
            Console.WriteLine("Error when receiving desired property: {0}", ex.Message);
        }
        return Task.CompletedTask;
    }
    ```

7. Quite el método de ejemplo **PipeMessage** y reemplácelo por un nuevo método **FilterMessages**. Se llama a este método cada vez que el módulo recibe un mensaje del centro de IoT Edge. Filtra los mensajes que informan de temperaturas por debajo del umbral de temperatura que se establecen mediante el módulo gemelo. También agrega la propiedad **MessageType** al mensaje con el valor establecido en **Alerta**.

    ```csharp
    static async Task<MessageResponse> FilterMessages(Message message, object userContext)
    {
        var counterValue = Interlocked.Increment(ref counter);
        try
        {
            ModuleClient moduleClient = (ModuleClient)userContext;
            var messageBytes = message.GetBytes();
            var messageString = Encoding.UTF8.GetString(messageBytes);
            Console.WriteLine($"Received message {counterValue}: [{messageString}]");

            // Get the message body.
            var messageBody = JsonConvert.DeserializeObject<MessageBody>(messageString);

            if (messageBody != null && messageBody.machine.temperature > temperatureThreshold)
            {
                Console.WriteLine($"Machine temperature {messageBody.machine.temperature} " +
                    $"exceeds threshold {temperatureThreshold}");
                using(var filteredMessage = new Message(messageBytes))
                {
                    foreach (KeyValuePair<string, string> prop in message.Properties)
                    {
                        filteredMessage.Properties.Add(prop.Key, prop.Value);
                    }

                    filteredMessage.Properties.Add("MessageType", "Alert");
                    await moduleClient.SendEventAsync("output1", filteredMessage);
                }
            }

            // Indicate that the message treatment is completed.
            return MessageResponse.Completed;
        }
        catch (AggregateException ex)
        {
            foreach (Exception exception in ex.InnerExceptions)
            {
                Console.WriteLine();
                Console.WriteLine("Error in sample: {0}", exception);
            }
            // Indicate that the message treatment is not completed.
            var moduleClient = (ModuleClient)userContext;
            return MessageResponse.Abandoned;
        }
        catch (Exception ex)
        {
            Console.WriteLine();
            Console.WriteLine("Error in sample: {0}", ex.Message);
            // Indicate that the message treatment is not completed.
            ModuleClient moduleClient = (ModuleClient)userContext;
            return MessageResponse.Abandoned;
        }
    }
    ```

8. Guarde el archivo Program.cs.

9. Abra el archivo **deployment.template.json** en la solución IoT Edge. Este archivo le indica al agente de IoT Edge qué módulos implementar, en este caso **SimulatedTemperatureSensor** y **CSharpModule**, y a IoT Edge Hub cómo enrutar los mensajes entre ellos.

10. Agregue el módulo gemelo **CSharpModule** al manifiesto de implementación. Inserte el siguiente contenido JSON en la parte inferior de la sección **modulesContent**, después del módulo gemelo **$edgeHub**:

    ```json
       "CSharpModule": {
           "properties.desired":{
               "TemperatureThreshold":25
           }
       }
    ```

    ![Adición de un módulo gemelo a una plantilla de implementación](./media/tutorial-csharp-module-windows/module-twin.png)

11. Guarde el archivo deployment.template.json.

## <a name="build-and-push-your-module"></a>Compilación e inserción del módulo

En la sección anterior se ha creado una solución de IoT Edge y se ha agregado código a **CSharpModule** para filtrar los mensajes en los que la temperatura registrada de la máquina está por debajo del umbral aceptable. Ahora, tiene que compilar la solución como una imagen de contenedor e insertarla en el registro de contenedor.

1. Utilice el siguiente comando para iniciar sesión en Docker en la máquina de desarrollo. Use el nombre de usuario, la contraseña y el servidor de inicio de sesión de Azure Container Registry. Puede recuperar estos valores en la sección **Claves de acceso** del Registro en Azure Portal.

   ```cmd
   docker login -u <ACR username> -p <ACR password> <ACR login server>
   ```

   Puede recibir una advertencia de seguridad en la que se recomiende el uso de `--password-stdin`. Aunque ese procedimiento se recomienda para escenarios de producción, está fuera del ámbito de este tutorial. Para más información, consulte la referencia de [docker login](https://docs.docker.com/engine/reference/commandline/login/#provide-a-password-using-stdin).

2. En el Explorador de soluciones de Visual Studio, haga clic con el botón derecho en el nombre del proyecto que desea compilar. El nombre predeterminado es **AzureIotEdgeApp1** y puesto que va a compilar un módulo de Windows, la extensión debe ser **Windows.Amd64**.

3. Seleccione **Build and Push IoT Edge Modules** (Compilación e inserción de los módulos de IoT Edge).

   El comando de compilación e inserción inicia tres operaciones. En primer lugar, se crea una nueva carpeta en la solución denominada **config** que contiene los archivos del manifiesto de la implementación completa, con la información de la plantilla de implementación y otros archivos de la solución. En segundo lugar, ejecuta `docker build` para generar la imagen de contenedor basándose en el Dockerfile adecuado para la arquitectura de destino. A continuación, ejecuta `docker push` para insertar el repositorio de imágenes en el registro de contenedor.

   Este proceso puede tardar varios minutos la primera vez, pero es más rápido la próxima vez que ejecute los comandos.

## <a name="deploy-modules-to-device"></a>Implementación de módulos en el dispositivo

Utilice el explorador en la nube de Visual Studio y la extensión Azure IoT Edge Tools para implementar el proyecto de módulo en el dispositivo IoT Edge. Ya tiene un manifiesto de implementación preparado para su escenario, el archivo **deployment.windows-amd64.json** de la carpeta config. Ahora todo lo que necesita hacer es seleccionar un dispositivo que reciba la implementación.

Asegúrese de que el dispositivo IoT Edge está en funcionamiento.

1. En el explorador en la nube de Visual Studio, expanda los recursos para ver la lista de dispositivos de IoT.

2. Haga clic con el botón derecho en el nombre del dispositivo IoT Edge que quiere que reciba la implementación.

3. Seleccione **Create Deployment** (Crear implementación).

4. En el explorador de archivos, seleccione el archivo **deployment.windows-amd64.json** en la carpeta de configuración de la solución.

5. Actualice el explorador en la nube para ver los módulos implementados enumerados en el dispositivo.

## <a name="view-generated-data"></a>Visualización de datos generados

Una vez aplicado el manifiesto de implementación al dispositivo de IoT Edge, el entorno de ejecución de IoT Edge del dispositivo recopila la información de implementación nueva y comienza a ejecutarse con ella. Los módulos que se ejecuten en el dispositivo y que no están incluidos en el manifiesto de implementación se detienen. Los módulos que falten en el dispositivo se inician.

Puede usar la extensión IoT Edge Tools para ver los mensajes a medida que llegan a IoT Hub.

1. En el explorador en la nube de Visual Studio, seleccione el nombre del dispositivo de IoT Edge.

2. En la lista **Acciones**, seleccione **Start Monitoring Built-in Event Endpoint** (Iniciar supervisión del punto de conexión del evento integrado).

3. Vea los mensajes que llegan a IoT Hub. Los mensajes pueden tardar un rato en llegar, ya que los cambios realizados en el código CSharpModule esperan hasta que la temperatura de la máquina alcanza los 25 grados antes de enviarlos. También agrega el tipo de mensaje **Alerta** a los mensajes que llegan a ese umbral de temperatura.

   ![Visualización de los mensajes que llegan a IoT Hub](./media/tutorial-csharp-module-windows/view-d2c-message.png)

## <a name="edit-the-module-twin"></a>Edición del módulo gemelo

El módulo gemelo CSharpModule se usa para establecer el umbral de temperatura en 25 grados. Puede usar al módulo gemelo para cambiar la funcionalidad sin tener que actualizar el código del módulo.

1. En Visual Studio, abra el archivo **deployment.windows amd64.json**. (No el archivo deployment.template. Si no ve el manifiesto de implementación en el archivo de configuración del Explorador de soluciones, seleccione el icono **Mostrar todos los archivos** en la barra de herramientas del explorador).

2. Busque el gemelo CSharpModule y cambie el valor del parámetro **temperatureThreshold** por una temperatura 5 a 10 grados mayor que la última notificada.

3. Guarde el archivo **deployment.windows amd64.json**.

4. Siga los pasos de implementación de nuevo para aplicar el manifiesto de implementación actualizado al dispositivo.

5. Supervise los mensajes entrantes del dispositivo a la nube. Debería ver que los mensajes se detienen hasta que se alcanza el nuevo umbral de temperatura.

## <a name="clean-up-resources"></a>Limpieza de recursos

Si prevé seguir con el siguiente artículo recomendado, puede mantener los recursos y las configuraciones que ya ha creado y volverlos a utilizar. También puede seguir usando el mismo dispositivo de IoT Edge como dispositivo de prueba.

En caso contrario, para evitar gastos, puede eliminar las configuraciones locales y los recursos de Azure que creó en este artículo.

[!INCLUDE [iot-edge-clean-up-cloud-resources](../../includes/iot-edge-clean-up-cloud-resources.md)]

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha creado un módulo IoT Edge que contiene código para filtrar datos sin procesar generados por el dispositivo IoT Edge. Cuando esté preparado para crear sus propios módulos, puede aprender más sobre cómo [desarrollar sus propios módulos IoT Edge](module-development.md) o cómo [desarrollar módulos con Visual Studio](how-to-visual-studio-develop-module.md). Para obtener ejemplos de módulos de IoT Edge, incluido el módulo de temperatura simulado, consulte [Ejemplos de módulos de IoT Edge](https://github.com/Azure/iotedge/tree/master/edge-modules).

Puede continuar con los siguientes tutoriales para obtener información sobre cómo Azure IoT Edge puede ayudarle a implementar servicios en la nube de Azure para procesar y analizar datos en el perímetro.

> [!div class="nextstepaction"]
> [Functions](tutorial-deploy-function.md)
> [Stream Analytics](tutorial-deploy-stream-analytics.md)
> [Machine Learning](tutorial-deploy-machine-learning.md)
> [Custom Vision Service](tutorial-deploy-custom-vision.md)
