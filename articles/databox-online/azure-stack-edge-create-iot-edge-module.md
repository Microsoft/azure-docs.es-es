---
title: Módulo de IoT Edge en C# para Azure Stack Edge Pro | Microsoft Docs
description: Aprenda a desarrollar un módulo de IoT Edge en C# que se pueda implementar en Azure Stack Edge Pro.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: how-to
ms.date: 08/06/2019
ms.author: alkohli
ms.custom: devx-track-csharp
ms.openlocfilehash: 8acbc7eec7581adcf0d73ffcd4bb2aa7ab2dd572
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90883496"
---
# <a name="develop-a-c-iot-edge-module-to-move-files-on-azure-stack-edge-pro"></a>Desarrollo de un módulo de IoT Edge en C# para mover archivos de Azure Stack Edge Pro

En este artículo, se explican los pasos para crear un módulo de IoT Edge e implementarlo con el dispositivo Azure Stack Edge Pro. Azure Stack Edge Pro es una solución de almacenamiento que permite procesar datos y enviarlos por la red a Azure.

Puede usar módulos de Azure IoT Edge con Azure Stack Edge Pro para transformar los datos a medida que se migran a Azure. El módulo que se utiliza en este artículo implementa la lógica para copiar un archivo de un recurso compartido local en un recurso compartido en la nube del dispositivo Azure Stack Edge Pro.

En este artículo aprenderá a:

> [!div class="checklist"]
>
> * Cree un registro de contenedor para almacenar y administrar los módulos (imágenes de Docker).
> * Cree un módulo de IoT Edge para implementarlo en el dispositivo Azure Stack Edge Pro. 


## <a name="about-the-iot-edge-module"></a>Acerca del módulo de IoT Edge

El dispositivo Azure Stack Edge Pro puede implementar y ejecutar módulos de IoT Edge. Los módulos de Edge son básicamente contenedores de Docker que realizan una tarea específica, como ingerir un mensaje de un dispositivo, transforman un mensaje o enviar un mensaje a un centro de IoT. En este artículo, va a crear un módulo que copia archivos de un recurso compartido local en un recurso compartido en la nube del dispositivo Azure Stack Edge Pro.

1. Los archivos se escriben en el recurso compartido local del dispositivo Azure Stack Edge Pro.
2. El generador de eventos de archivo crea un evento de archivo para cada archivo escrito en el recurso compartido local. Los eventos de archivo también se generan cuando se modifica un archivo. Los eventos de archivo se envían luego al centro de IoT Edge (en el entorno de ejecución de IoT Edge).
3. El módulo personalizado de IoT Edge procesa el evento de archivo para crear un objeto de evento de archivo que también contiene una ruta de acceso relativa para el archivo. El módulo genera una ruta de acceso absoluta mediante la ruta de acceso de archivo relativa y copia el archivo del recurso compartido local al recurso compartido de nube. El módulo elimina entonces el archivo del recurso compartido local.

![Funcionamiento del módulo de Azure IoT Edge en Azure Stack Edge Pro](./media/azure-stack-edge-create-iot-edge-module/how-module-works-1.png)

Una vez que el archivo está en el recurso compartido en la nube, se carga automáticamente en la cuenta de Azure Storage.

## <a name="prerequisites"></a>Prerrequisitos

Antes de comenzar, asegúrese de que tiene:

- Un dispositivo Azure Stack Edge Pro en ejecución.

    - El dispositivo también tiene un recurso de IoT Hub asociado.
    - El dispositivo tiene configurado el rol de proceso perimetral.
    Para más información, vaya a [Configuración del proceso](azure-stack-edge-deploy-configure-compute.md#configure-compute) en Azure Stack Edge Pro.

- Los siguientes recursos de desarrollo:

    - [Visual Studio Code](https://code.visualstudio.com/).
    - [Extensión de C# para Visual Studio Code (con tecnología de OmniSharp)](https://marketplace.visualstudio.com/items?itemName=ms-dotnettools.csharp)
    - [Extensión de Azure IoT Edge para Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-edge).
    - [SDK de .NET Core 2.1](https://www.microsoft.com/net/download).
    - [Docker CE](https://store.docker.com/editions/community/docker-ce-desktop-windows). Es posible que deba crear una cuenta para descargar e instalar el software.

## <a name="create-a-container-registry"></a>Creación de un Registro de contenedor

Azure Container Registry es un registro privado de Docker en Azure donde se pueden almacenar y administrar las imágenes de contenedor privado de Docker. Los dos servicios de Docker Registry más conocidos disponibles en la nube son Azure Container Registry y Docker Hub. En este artículo se usa Container Registry.

1. Inicie sesión en Azure Portal en [https://portal.azure.com](https://portal.azure.com).
2. Seleccione **Crear un recurso > Contenedores > Container Registry**. Haga clic en **Crear**.
3. Proporcione:

   1. Un **nombre de registro** único en Azure que contenga entre 5 y 50 caracteres alfanuméricos.
   2. Elija una **suscripción**.
   3. Cree un **grupo de recursos** o elija uno ya existente.
   4. Seleccione una **ubicación**. Es conveniente que esta ubicación sea la misma que está asociada con el recurso de Azure Stack Edge.
   5. Establezca **Usuario administrador** en **Habilitar**.
   6. Establezca la SKU en **Básica**.

      ![Crear un registro de contenedor](./media/azure-stack-edge-create-iot-edge-module/create-container-registry-1.png)
 
4. Seleccione **Crear**.
5. Una vez que se haya creado el Registro de contenedor, desplácese hasta él y seleccione **Claves de acceso**.

    ![Obtención de las claves de acceso](./media/azure-stack-edge-create-iot-edge-module/get-access-keys-1.png)
 
6. Copie los valores de **Servidor de inicio de sesión**, **Nombre de usuario** y **Contraseña**. Usará estos valores más adelante al publicar la imagen de Docker en el registro y al agregar las credenciales del registro al entorno de ejecución de Azure IoT Edge.


## <a name="create-an-iot-edge-module-project"></a>Creación de un proyecto de módulo IoT Edge

Mediante los pasos siguientes se crea un proyecto de módulo de IoT Edge basado en el SDK de .NET Core 2.1. El proyecto usa Visual Studio Code y la extensión Azure IoT Edge.

### <a name="create-a-new-solution"></a>Creación de una nueva solución

Cree una plantilla de solución de C que pueda personalizar con su propio código.

1. En Visual Studio Code, seleccione **View > Command Palette** (Ver > Paleta de comandos) para abrir la paleta de comandos de VS Code.
2. En la paleta de comandos, escriba y ejecute el comando **Azure: Sign in** (Azure: iniciar sesión) y siga las instrucciones para iniciar sesión en la cuenta de Azure. Si ya ha iniciado sesión, puede omitir este paso.
3. En la paleta de comandos, escriba y ejecute el comando **Azure IoT Edge: New IoT Edge solution** (Azure IoT Edge: nueva solución de IoT Edge). En la paleta de comandos, proporcione la siguiente información para crear la solución:

    1. Seleccione la carpeta en la que desea crear la solución.
    2. Proporcione un nombre para la solución o acepte el valor predeterminado: **EdgeSolution**.
    
        ![Creación de la solución 1](./media/azure-stack-edge-create-iot-edge-module/create-new-solution-1.png)

    3. Elija el **módulo de C#** como la plantilla del módulo.
    4. Reemplace el nombre de módulo predeterminado por el nombre que quiere asignar, en este caso, **FileCopyModule**.
    
        ![Creación de la solución 2](./media/azure-stack-edge-create-iot-edge-module/create-new-solution-2.png)

    5. Especifique el registro de contenedor que creó en la sección anterior como repositorio de imágenes del primer módulo. Reemplace **localhost:5000** por el valor del servidor de inicio de sesión que copió.

        La cadena final se parece a `<Login server name>/<Module name>`. En este ejemplo, la cadena es `mycontreg2.azurecr.io/filecopymodule`.

        ![Creación de la solución 3](./media/azure-stack-edge-create-iot-edge-module/create-new-solution-3.png)

4. Vaya a **File > Open Folder** (Archivo > Abrir carpeta).

    ![Creación de la solución 4](./media/azure-stack-edge-create-iot-edge-module/create-new-solution-4.png)

5. Examine y seleccione la carpeta **EdgeSolution** que creó anteriormente. La ventana de VS Code carga el área de trabajo de la solución de IoT Edge con sus cinco componentes de nivel superior. En este artículo, no se modificará la carpeta **.vscode**, ni los archivos **.gitignore**, **.env** y **deployment.template.json**.
    
    El único componente que se modificará es la carpeta modules. Esta carpeta contiene el código en C# para el módulo y los archivos de Docker para la compilación de un módulo como una imagen de contenedor.

    ![Creación de la solución 5](./media/azure-stack-edge-create-iot-edge-module/create-new-solution-5.png)

### <a name="update-the-module-with-custom-code"></a>Actualización del módulo con código personalizado

1. En el explorador de VS Code, abra **modules > FileCopyModule > Program.cs**.
2. En la parte superior del espacio de nombres **FileCopyModule**, agregue las siguientes instrucciones using para los tipos que se usarán más adelante. **Microsoft.Azure.Devices.Client.Transport.Mqtt** es un protocolo para enviar mensajes al centro de IoT Edge.

    ```
    namespace FileCopyModule
    {
        using Microsoft.Azure.Devices.Client.Transport.Mqtt;
        using Newtonsoft.Json;
    ```
3. Agregue las variables **InputFolderPath** y **OutputFolderPath** a la clase Program.

    ```
    class Program
        {
            static int counter;
            private const string InputFolderPath = "/home/input";
            private const string OutputFolderPath = "/home/output";
    ```

4. Inmediatamente después del paso anterior, agregue la clase **FileEvent** para definir el cuerpo del mensaje.

    ```
    /// <summary>
    /// The FileEvent class defines the body of incoming messages. 
    /// </summary>
    private class FileEvent
    {
        public string ChangeType { get; set; }

        public string ShareRelativeFilePath { get; set; }

        public string ShareName { get; set; }
    }
    ```

5. En el **método Init**, el código crea y configura un objeto **ModuleClient**. Este objeto permite al módulo conectarse al entorno de ejecución de Azure IoT Edge local mediante el protocolo MQTT para enviar y recibir mensajes. El entorno de ejecución de Azure IoT Edge suministra la cadena de conexión que se usa en el método Init. El código registra una devolución de llamada de FileCopy para recibir mensajes de un centro de IoT Edge a través del punto de conexión **input1**. Reemplace el **método Init** por el código siguiente.

    ```
    /// <summary>
    /// Initializes the ModuleClient and sets up the callback to receive
    /// messages containing file event information
    /// </summary>
    static async Task Init()
    {
        MqttTransportSettings mqttSetting = new MqttTransportSettings(TransportType.Mqtt_Tcp_Only);
        ITransportSettings[] settings = { mqttSetting };

        // Open a connection to the IoT Edge runtime
        ModuleClient ioTHubModuleClient = await ModuleClient.CreateFromEnvironmentAsync(settings);
        await ioTHubModuleClient.OpenAsync();
        Console.WriteLine("IoT Hub module client initialized.");

        // Register callback to be called when a message is received by the module
        await ioTHubModuleClient.SetInputMessageHandlerAsync("input1", FileCopy, ioTHubModuleClient);
    }
    ```

6. Quite el código del **método PipeMessage** y, en su lugar, inserte el código de **FileCopy**.

    ```
        /// <summary>
        /// This method is called whenever the module is sent a message from the IoT Edge Hub.
        /// This method deserializes the file event, extracts the corresponding relative file path, and creates the absolute input file path using the relative file path and the InputFolderPath.
        /// This method also forms the absolute output file path using the relative file path and the OutputFolderPath. It then copies the input file to output file and deletes the input file after the copy is complete.
        /// </summary>
        static async Task<MessageResponse> FileCopy(Message message, object userContext)
        {
            int counterValue = Interlocked.Increment(ref counter);

            try
            {
                byte[] messageBytes = message.GetBytes();
                string messageString = Encoding.UTF8.GetString(messageBytes);
                Console.WriteLine($"Received message: {counterValue}, Body: [{messageString}]");

                if (!string.IsNullOrEmpty(messageString))
                {
                    var fileEvent = JsonConvert.DeserializeObject<FileEvent>(messageString);

                    string relativeFileName = fileEvent.ShareRelativeFilePath.Replace("\\", "/");
                    string inputFilePath = InputFolderPath + relativeFileName;
                    string outputFilePath = OutputFolderPath + relativeFileName;

                    if (File.Exists(inputFilePath))                
                    {
                        Console.WriteLine($"Moving input file: {inputFilePath} to output file: {outputFilePath}");
                        var outputDir = Path.GetDirectoryName(outputFilePath);
                        if (!Directory.Exists(outputDir))
                        {
                            Directory.CreateDirectory(outputDir);
                        }

                        File.Copy(inputFilePath, outputFilePath, true);
                        Console.WriteLine($"Copied input file: {inputFilePath} to output file: {outputFilePath}");
                        File.Delete(inputFilePath);
                        Console.WriteLine($"Deleted input file: {inputFilePath}");
                    } 
                    else
                    {
                        Console.WriteLine($"Skipping this event as input file doesn't exist: {inputFilePath}");   
                    }
                }
            }
            catch (Exception ex)
            {
                Console.WriteLine("Caught exception: {0}", ex.Message);
                Console.WriteLine(ex.StackTrace);
            }

            Console.WriteLine($"Processed event.");
            return MessageResponse.Completed;
        }
    ```

7. Guarde este archivo.
8. También puede [descargar un ejemplo de código existente](https://azure.microsoft.com/resources/samples/data-box-edge-csharp-modules/?cdn=disable) para este proyecto. A continuación, puede validar el archivo guardado en el archivo **program.cs** en este ejemplo.

## <a name="build-your-iot-edge-solution"></a>Compilación de la solución de IoT Edge

En la sección anterior, creó una solución de IoT Edge y agregó código a FileCopyModule para copiar los archivos del recurso compartido local al recurso compartido en la nube. Ahora, tiene que compilar la solución como una imagen de contenedor e insertarla en el registro de contenedor.

1. En VSCode, vaya a Terminal > Nuevo terminal para abrir un nuevo terminal integrado de Visual Studio Code.
2. Escriba el comando siguiente en el terminal integrado para iniciar sesión en Docker.

    `docker login <ACR login server> -u <ACR username>`

    Use el servidor de inicio de sesión y el nombre de usuario que copió del registro de contenedor.

    ![Compilación e inserción de la solución de IoT Edge](./media/azure-stack-edge-create-iot-edge-module/build-iot-edge-solution-1.png)

2. Proporcione la contraseña cuando se le pida. También puede recuperar los valores de servidor de inicio de sesión, nombre de usuario y contraseña de las **claves de acceso** del registro de contenedor en Azure Portal.
 
3. Una vez que se proporcionan las credenciales, puede insertar la imagen del módulo en su registro de contenedor de Azure. En el explorador de VS Code, haga clic con el botón derecho en el archivo **module.json** y seleccione **Build and Push IoT Edge solution** (Compilar e insertar la solución de IoT Edge).

    ![Compilación e inserción de la solución de IoT Edge](./media/azure-stack-edge-create-iot-edge-module/build-iot-edge-solution-2.png)
 
    Cuando le indica a Visual Studio Code que compile la solución, se ejecutan dos comandos en el terminal integrado: docker build y docker push. Estos dos comandos compilan el código, empaquetan CSharpModule.dll en contenedores e insertan el código en el registro de contenedor que especificó cuando inicializó la solución.

    Se le pedirá que elija la plataforma del módulo. Seleccione *amd64*, que corresponde a Linux.

    ![Seleccionar plataforma](./media/azure-stack-edge-create-iot-edge-module/select-platform.png)

    > [!IMPORTANT] 
    > Solamente se admiten módulos de Linux.

    Verá la siguiente advertencia, que puede pasar por alto:

    *Program.cs(77,44): warning CS1998: This async method lacks 'await' operators and will run synchronously (Program.cs(77,44): advertencia CS1998: este método asincrónico carece de operadores "await" y se ejecutará de manera sincrónica). Considere el uso del operador "await" para esperar llamadas API sin bloqueo, o "await Task.Run (...)" para realizar trabajo enlazado con la CPU en un subproceso en segundo plano.*

4. Puede ver la dirección de la imagen de contenedor completa con la etiqueta en el terminal integrado de VS Code. La dirección de la imagen se crea a partir de la información del archivo module.json, con el formato `<repository>:<version>-<platform>`. En este artículo, se debe parecer a `mycontreg2.azurecr.io/filecopymodule:0.0.1-amd64`.

## <a name="next-steps"></a>Pasos siguientes

Para implementar y ejecutar este módulo en Azure Stack Edge Pro, consulte los pasos que se indican en [Adición de un módulo](azure-stack-edge-deploy-configure-compute.md#add-a-module).
