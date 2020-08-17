---
title: Desarrollo y depuración de módulos para Azure IoT Edge | Microsoft Docs
description: Uso de Visual Studio Code para desarrollar, compilar y depurar un módulo de Azure IoT Edge mediante C#, Python, Node.js, Java o C
services: iot-edge
keywords: ''
author: shizn
ms.author: xshi
ms.date: 08/07/2019
ms.topic: conceptual
ms.service: iot-edge
ms.custom: devx-track-javascript
ms.openlocfilehash: 848f617aaf74960aeecd856016321ac7607eca00
ms.sourcegitcommit: e71da24cc108efc2c194007f976f74dd596ab013
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87422902"
---
# <a name="use-visual-studio-code-to-develop-and-debug-modules-for-azure-iot-edge"></a>Uso de Visual Studio Code para desarrollar y depurar módulos para Azure IoT Edge

Puede convertir la lógica de negocios en módulos de Azure IoT Edge. En este artículo se muestra cómo usar Visual Studio Code como herramienta principal para desarrollar y depurar módulos.

Hay dos maneras de depurar los módulos escritos en C#, Node.js o Java en Visual Studio Code: Puede agregar un proceso a un contenedor de módulo o iniciar el código del módulo en modo de depuración. Para depurar los módulos escritos en Python o C, solo puede agregar un proceso en contenedores de Linux amd64.

Si no está familiarizado con las funcionalidades de depuración de Visual Studio Code, consulte más información sobre [depuración](https://code.visualstudio.com/Docs/editor/debugging).

En este artículo se proporcionan instrucciones para desarrollar y depurar módulos en varios lenguajes y para varias arquitecturas. Actualmente, Visual Studio Code proporciona compatibilidad con módulos escritos en C#, C, Python, node.js y Java. Las arquitecturas de dispositivo compatibles son X64 y ARM32. Para obtener más información sobre los sistemas operativos, los lenguajes y las arquitecturas compatibles, consulte [Compatibilidad de lenguaje y arquitectura](module-development.md#language-and-architecture-support).

>[!NOTE]
>El soporte técnico del desarrollo y depuración para dispositivos ARM64 de Linux está en [versión preliminar pública](https://azure.microsoft.com/support/legal/preview-supplemental-terms/). Para más información, consulte [Desarrollo y depuración de módulos ARM64 IoT Edge en Visual Studio Code (versión preliminar)](https://devblogs.microsoft.com/iotdev/develop-and-debug-arm64-iot-edge-modules-in-visual-studio-code-preview).

## <a name="prerequisites"></a>Requisitos previos

Puede usar un equipo o máquina virtual con Windows, macOS o Linux como equipo de desarrollo. En equipos Windows, puede desarrollar módulos de Windows o Linux. Para desarrollar módulos de Windows, use un equipo Windows que ejecute la versión 1809, la compilación 17763 o una más reciente. Para desarrollar módulos de Linux, use un equipo Windows que cumpla los [requisitos de Docker Desktop](https://docs.docker.com/docker-for-windows/install/#what-to-know-before-you-install).

Instale [Visual Studio Code](https://code.visualstudio.com/) y después agregue las siguientes extensiones:

- [Herramientas de Azure IoT](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-tools)
- [Extensión de Docker](https://marketplace.visualstudio.com/items?itemName=PeterJausovec.vscode-docker)
- Extensiones de Visual Studio específicas del lenguaje en el que esté desarrollando:
  - C#, incluyendo Azure Functions: [Extensión de C#](https://marketplace.visualstudio.com/items?itemName=ms-dotnettools.csharp)
  - Python: [Extensión de Python](https://marketplace.visualstudio.com/items?itemName=ms-python.python)
  - Java: [Paquete de extensiones de Java para Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=vscjava.vscode-java-pack)
  - C. [Extensión de C/C++](https://marketplace.visualstudio.com/items?itemName=ms-vscode.cpptools)

También deberá instalar algunas herramientas específicas del lenguaje adicionales para desarrollar el módulo:

- C#, incluido Azure Functions: [SDK de .NET Core 2.1](https://www.microsoft.com/net/download)

- Python: [Python](https://www.python.org/downloads/) y [Pip](https://pip.pypa.io/en/stable/installing/#installation) para instalar paquetes de Python (normalmente, se incluye con la instalación de Python).

- Node.js: [Node.js](https://nodejs.org). También le interesará instalar [Yeoman](https://www.npmjs.com/package/yo) y el [generador de módulo Node.js de Azure IoT Edge](https://www.npmjs.com/package/generator-azure-iot-edge-module).

- Java: [Java SE Development Kit 10](https://aka.ms/azure-jdks) y [Maven](https://maven.apache.org/). Deberá [establecer la `JAVA_HOME`variable de entorno](https://docs.oracle.com/cd/E19182-01/820-7851/inst_cli_jdk_javahome_t/) para que apunte a la instalación de JDK.

Para compilar e implementar la imagen del módulo, necesita Docker para compilar la imagen del módulo y un registro de contenedor para guardarla:

- [Docker Community Edition](https://docs.docker.com/install/) en la máquina de desarrollo

- [Azure Container Registry](https://docs.microsoft.com/azure/container-registry/) o [Docker Hub](https://docs.docker.com/docker-hub/repos/#viewing-repository-tags)

    > [!TIP]
    > Puede usar un registro de Docker local con fines de prototipo y prueba en lugar de un registro en la nube.

A menos que esté desarrollando el módulo en C, también necesitará la [herramienta de desarrollo de Azure IoT EdgeHub](https://pypi.org/project/iotedgehubdev/) basada en Python con el fin de configurar el entorno de desarrollo local para depurar, ejecutar y probar la solución de IoT Edge. Si aún no lo ha hecho, instale [Python (2.7/3.6/3.7) y Pip](https://www.python.org/) y después **iotedgehubdev** mediante la ejecución de este comando en el terminal.

   ```cmd
   pip install --upgrade iotedgehubdev
   ```
   
> [!NOTE]
> Actualmente, iotedgehubdev usa una biblioteca docker-py que no es compatible con Python 3.8.
>
> Si tiene varias versiones de Python, incluida la versión Python 2.7 preinstalada (por ejemplo, en Ubuntu o macOS), asegúrese de que usa los valores `pip` o `pip3` correctos para instalar **iotedgehubdev**.

Para probar el módulo en un dispositivo, necesitará una instancia de IoT Hub con al menos un dispositivo IoT Edge. Para usar el equipo como un dispositivo IoT Edge, siga los pasos descritos en el inicio rápido para [Linux](quickstart-linux.md) o [Windows](quickstart.md). Si ejecuta el demonio de IoT Edge en el equipo de desarrollo, es posible que deba detener EdgeHub y EdgeAgent antes de ir al paso siguiente.

## <a name="create-a-new-solution-template"></a>Creación de una plantilla de solución

En los siguientes pasos se muestra cómo crear un módulo de IoT Edge en su lenguaje de desarrollo preferido (incluido Azure Functions, escrito en C#) mediante Visual Studio Code y Herramientas de Azure IoT. Primero debe crear una solución y, a continuación, generar el primer módulo de esa solución. Cada solución puede contener varios módulos.

1. Seleccione **Ver** > **Paleta de comandos**.

1. En la paleta de comandos, escriba y ejecute el comando **Azure IoT Edge: New IoT Edge solution** (Nueva solución de IoT Edge).

   ![Ejecución de la nueva solución IoT Edge](./media/how-to-develop-csharp-module/new-solution.png)

1. Vaya a la carpeta en la que desea crear la solución y después haga clic en **Seleccionar carpeta**.

1. Escriba un nombre para la solución.

1. Seleccione una plantilla de módulo para que su lenguaje de desarrollo preferido sea el primer módulo de la solución.

1. Escriba un nombre para el módulo. Elija un nombre que sea único dentro del registro de contenedor.

1. Proporcione el nombre del repositorio de imágenes del módulo. Visual Studio Code completa automáticamente el nombre del módulo como **localhost:5000 /<el nombre del módulo\>** . Reemplácelo por su propia información de registro. Si usa un registro de Docker local para realizar pruebas, **localhost** puede valer. Si va a usar Azure Container Registry, utilice el servidor de inicio de sesión de la configuración del registro. El servidor de inicio de sesión se parece a **_\<registry name\>_ .azurecr.io**. Reemplace solo la sección **localhost:5000** de la cadena para que el resultado final se parezca a **\<*registry name*\>.azurecr.io/ _\<your module name\>_**.

   ![Especificación del repositorio de imágenes de Docker](./media/how-to-develop-csharp-module/repository.png)

Visual Studio Code toma la información que le ha proporcionado, crea una solución IoT Edge y la carga en una nueva ventana.

Hay cuatro elementos dentro de la solución:

- Una carpeta **.vscode** que contiene configuraciones de depuración.

- Una carpeta **modules** que contiene subcarpetas para cada módulo.  Dentro de la carpeta de cada módulo hay un archivo, **module.json**, que controla cómo se compilan e implementan los módulos.  Es necesario modificar este archivo para cambiar el registro del contenedor de la implementación del módulo de localhost a un registro remoto. En este momento, solo tiene un módulo.  Sin embargo, podría agregar más en la paleta de comandos con el comando **Azure IoT Edge: Add IoT Edge Module** (Azure IoT Edge: agregar módulo IoT Edge).

- Un archivo **.env** enumera las variables de entorno. Si Azure Container Registry es su registro, tendrá en él un nombre de usuario y una contraseña de Azure Container Registry.

  > [!NOTE]
  > Solo se crea el archivo de entorno si proporciona un repositorio de imágenes para el módulo. Si aceptó los valores predeterminados de localhost para probar y depurar localmente, no es necesario declarar las variables de entorno.

- Un archivo **deployment.template.json** muestra el nuevo módulo junto con un módulo **SimulatedTemperatureSensor** de ejemplo que simula los datos que puede usar para las pruebas. Para más información sobre cómo funcionan los manifiestos de implementación, consulte [cómo usar los manifiestos de implementación para implementar módulos y establecer rutas](module-composition.md).

Para ver cómo funciona el módulo de temperatura simulada, vea el [código fuente de SimulatedTemperatureSensor.csproj](https://github.com/Azure/iotedge/tree/master/edge-modules/SimulatedTemperatureSensor).

## <a name="add-additional-modules"></a>Agregar módulos adicionales

Para agregar módulos adicionales a la solución, ejecute el comando **Azure IoT Edge: Add IoT Edge Module** (Azure IoT Edge: Agregar módulo IoT Edge) desde la paleta de comandos. También puede hacer clic en la carpeta **módulos** o el archivo `deployment.template.json` en la vista Explorador de Visual Studio Code y después seleccionar **Agregar módulo IoT Edge**.

## <a name="develop-your-module"></a>Desarrollo de su módulo

El código del módulo predeterminado que se incluye con la solución se encuentra en la siguiente ubicación:

- Azure Function (C#): **módulos > *&lt;el nombre del módulo&gt;*  >  *&lt;el nombre del módulo&gt;* .cs**
- C#: **módulos > *&lt;el nombre del módulo&gt;* > Program.cs**
- Python: **módulos > *&lt;el nombre del módulo&gt;* > main.py**
- Node.js: **módulos > *&lt;el nombre del módulo&gt;* > app.js**
- Java: **módulos > *&lt;el nombre del módulo&gt;* > src > main > java > com > edgemodulemodules > App.java**
- C: **módulos > *&lt;el nombre del módulo&gt;* > main.c**

El módulo y el archivo deployment.template.json se configuran para que pueda compilar la solución, insertarla en el registro de contenedor e implementarla en un dispositivo para iniciar la prueba sin tocar el código. El módulo se compila para recibir entradas de un origen (en este caso, el módulo SimulatedTemperatureSensor que simula datos) y canalizarlo a IoT Hub.

Cuando tenga todo listo para personalizar la plantilla de C con su propio código, use los [SDK de Azure IoT Hub](../iot-hub/iot-hub-devguide-sdks.md) para compilar módulos que aborden las necesidades principales de las soluciones de IoT, como seguridad, administración de dispositivos y confiabilidad.

## <a name="debug-a-module-without-a-container-c-nodejs-java"></a>Depurar un módulo sin un contenedor (C#, Node.js, Java)

Si está desarrollando en C#, Node.js o Java, el módulo requiere el uso de un objeto **ModuleClient** en el código del módulo predeterminado para poder iniciar, ejecutar y enrutar los mensajes. También usará el canal de entrada predeterminado **input1** para realizar una acción cuando el módulo reciba los mensajes.

### <a name="set-up-iot-edge-simulator-for-iot-edge-solution"></a>Configuración del simulador de IoT Edge para la solución de IoT Edge

En el equipo de desarrollo, puede iniciar un simulador de IoT Edge en lugar de instalar el demonio de seguridad de IoT Edge de forma que pueda ejecutar la solución de IoT Edge.

1. En el explorador de dispositivos del lado izquierdo, haga clic con el botón derecho en el identificador de dispositivo IoT Edge y después seleccione **Setup IoT Edge Simulator** (Configurar el simulador de IoT Edge) para iniciar el simulador con la cadena de conexión del dispositivo.
1. Puede ver que el simulador de IoT Edge se ha configurado correctamente leyendo la información de progreso en el terminal integrado.

### <a name="set-up-iot-edge-simulator-for-single-module-app"></a>Configuración del simulador de IoT Edge para la aplicación de módulo único

Para configurar e iniciar el simulador, ejecute el comando **Azure IoT Edge: Start IoT Edge Hub Simulator for Single Module** (Azure IoT Edge: Iniciar el simulador de centro de IoT Edge para módulo único) desde la paleta de comandos de Visual Studio Code. Cuando se le pida, use el valor **input1** del código de módulo predeterminado (o el valor equivalente de su código) como el nombre de entrada para la aplicación. El comando desencadena la CLI **iotedgehubdev** y después inicia el simulador de IoT Edge y un contenedor de módulo de herramienta para pruebas. Puede ver las siguientes salidas en el terminal integrado si el simulador se ha iniciado correctamente en modo de módulo único. También puede ver un comando `curl` para ayudar a enviar el mensaje. Lo usará más adelante.

   ![Configuración del simulador de IoT Edge para la aplicación de módulo único](media/how-to-develop-csharp-module/start-simulator-for-single-module.png)

   Puede usar la vista Explorador de Docker en Visual Studio Code para ver el estado de ejecución del módulo.

   ![Estado del módulo simulador](media/how-to-develop-csharp-module/simulator-status.png)

   El contenedor **edgeHubDev** es el núcleo del simulador de IoT Edge local. Puede ejecutarse en el equipo de desarrollo sin el demonio de seguridad de IoT Edge y proporcionar la configuración del entorno para la aplicación de módulo nativa o los contenedores de módulo. El contenedor de **entrada** expone las API REST para ayudar a que los mensajes lleguen al canal de entrada de destino en el módulo.

### <a name="debug-module-in-launch-mode"></a>Depuración del módulo en el modo de inicio

1. Prepare el entorno de depuración según los requisitos de su lenguaje de desarrollo, establezca un punto de interrupción en el módulo y seleccione la configuración de depuración para usar:
   - **C#**
     - En el terminal integrado de Visual Studio Code, cambie el directorio a la carpeta con su ***&lt;nombre de módulo&gt;*** y después ejecute el siguiente comando para compilar la aplicación de .Net Core.

       ```cmd
       dotnet build
       ```

     - Abra el archivo `Program.cs` y agregue un punto de interrupción.

     - Vaya a la vista Depuración de Visual Studio Code seleccionando **Vista > Depurar**. Seleccione la configuración de depuración ***&lt;nombre del módulo&gt;* Local Debug (.NET Core)** en la lista desplegable.

        > [!NOTE]
        > Si su `TargetFramework` de .Net Core no es coherente con la ruta de acceso del programa en `launch.json`, deberá actualizar manualmente la ruta de acceso del programa en `launch.json` para que coincida con el `TargetFramework` en el archivo .csproj para que Visual Studio Code pueda iniciar correctamente este programa.

   - **Node.js**
     - En el terminal integrado de Visual Studio Code, cambie el directorio a la carpeta ***&lt;nombre del módulo&gt;*** y después ejecute el siguiente comando para instalar paquetes Node.

       ```cmd
       npm install
       ```

     - Abra el archivo `app.js` y agregue un punto de interrupción.

     - Vaya a la vista Depuración de Visual Studio Code seleccionando **Vista > Depurar**. Seleccione la configuración de depuración ***&lt;nombre del módulo&gt;* Local Debug (Node.js)** en la lista desplegable.
   - **Java**
     - Abra el archivo `App.java` y agregue un punto de interrupción.

     - Vaya a la vista Depuración de Visual Studio Code seleccionando **Vista > Depurar**. Seleccione la configuración de depuración ***&lt;nombre del módulo&gt;* Local Debug (Java)** en la lista desplegable.

1. Haga clic en **Iniciar depuración** o presione **F5** para iniciar la sesión de depuración.

1. En el terminal integrado de Visual Studio Code, ejecute el siguiente comando para enviar un mensaje de **Hola mundo** al módulo. Este es el comando que se mostró en pasos anteriores cuando configuró el simulador de IoT Edge.

    ```bash
    curl --header "Content-Type: application/json" --request POST --data '{"inputName": "input1","data":"hello world"}' http://localhost:53000/api/v1/messages
    ```

   > [!NOTE]
   > Si usa Windows, asegúrese de que el shell del terminal integrado de Visual Studio Code es **Git Bash** o **WSL Bash**. No puede ejecutar el comando `curl` en PowerShell ni desde el símbolo del sistema.
   > [!TIP]
   > También puede usar [PostMan](https://www.getpostman.com/) u otras herramientas de API para enviar mensajes en lugar de `curl`.

1. En la vista Depuración de Visual Studio Code, verá las variables en el panel izquierdo.

1. Para detener la sesión de depuración, haga clic en el botón Detener o presione **MAYÚS + F5** y después ejecute **Azure IoT Edge: Stop IoT Edge Simulator** (Azure IoT Edge: Detener el simulador de IoT Edge) en la paleta de comandos para detener el simulador y limpiar.

## <a name="debug-in-attach-mode-with-iot-edge-simulator-c-nodejs-java-azure-functions"></a>Depuración en modo de agregación con el simulador de IoT Edge (C#, Node.js, Java, Azure Functions)

La solución predeterminada contiene dos módulos, uno es un módulo de sensor de temperatura simulado y el otro es el módulo de canalización. El sensor de temperatura simulado manda mensajes al módulo de canalización, y después los mensajes se canalizan a IoT Hub. En la carpeta de módulo que se creó, hay varios archivos de Docker para diferentes tipos de contenedor. Use cualquiera de estos archivos que acaban con la extensión **.debug** para compilar el módulo de pruebas.

Actualmente, la depuración en modo de agregación solo es compatible como sigue:

- Los módulos de C#, incluyendo los de Azure Functions, admiten la depuración en contenedores de Linux amd64
- Los módulos de Node.js admiten la depuración en contenedores de Linux amd64 y arm32v7, y en contenedores de Windows amd64
- Los módulos de Java admiten la depuración en contenedores de Linux amd64 y arm32v7

> [!TIP]
> Puede cambiar las opciones de la plataforma predeterminada para la solución de IoT Edge si hace clic en el elemento en la barra de estado de Visual Studio Code.

### <a name="set-up-iot-edge-simulator-for-iot-edge-solution"></a>Configuración del simulador de IoT Edge para la solución de IoT Edge

En el equipo de desarrollo, puede iniciar un simulador de IoT Edge en lugar de instalar el demonio de seguridad de IoT Edge de forma que pueda ejecutar la solución de IoT Edge.

1. En el explorador de dispositivos del lado izquierdo, haga clic con el botón derecho en el identificador de dispositivo IoT Edge y después seleccione **Setup IoT Edge Simulator** (Configurar el simulador de IoT Edge) para iniciar el simulador con la cadena de conexión del dispositivo.

1. Puede ver que el simulador de IoT Edge se ha configurado correctamente leyendo la información de progreso en el terminal integrado.

### <a name="build-and-run-container-for-debugging-and-debug-in-attach-mode"></a>Compilación y ejecución del contenedor para la depuración y depuración en modo de agregación

1. Abra el archivo de módulo (`Program.cs`, `app.js`, `App.java` o `<your module name>.cs`) y agregue un punto de interrupción.

1. En la vista Explorador de Visual Studio Code, haga clic con el botón derecho en el archivo `deployment.debug.template.json` para la solución y después seleccione **Compilar y ejecutar solución de IoT Edge en el Simulador**. Puede inspeccionar todos los registros del contenedor del módulo en la misma ventana. También puede navegar hasta la vista Docker para observar el estado del contenedor.

   ![Inspección de variables](media/how-to-vs-code-develop-module/view-log.png)

1. Vaya a la vista Depuración de Visual Studio Code y seleccione el archivo de configuración de depuración del módulo. El nombre de la opción de depuración debe ser similar a ***&lt;nombre del módulo&gt;* Remote Debug**

1. Haga clic en **Iniciar depuración** o presione **F5**. Seleccione el proceso al que desea asociarlo.

1. En la vista Depuración de Visual Studio Code, verá las variables en el panel izquierdo.

1. Para detener la sesión de depuración, primero haga clic en el botón Detener o presione **MAYÚS + F5**, y después ejecute **Azure IoT Edge: Stop IoT Edge Simulator** (Azure IoT Edge: Detener módulo IoT Edge) desde la paleta de comandos.

> [!NOTE]
> En el ejemplo anterior se muestra cómo depurar módulos de IoT Edge en contenedores. Ha agregado puertos expuestos a la configuración `createOptions` del contenedor del módulo. Una vez finalizada la depuración de los módulos, se recomienda quitar estos puertos expuestos de los módulos de IoT Edge listos para entornos de producción.
>
> Para módulos escritos en C#, incluyendo los de Azure Functions, este ejemplo se basa en la versión de depuración de `Dockerfile.amd64.debug`, que incluye el depurador de línea de comandos de .NET Core (VSDBG) en la imagen de contenedor al realizar la compilación. Una vez finalizada la depuración de los módulos de C#, se recomienda usar directamente Dockerfile sin VSDBG para módulos de IoT Edge listos para producción.

## <a name="debug-a-module-with-the-iot-edge-runtime"></a>Depuración de un módulo con el entorno de tiempo de ejecución de IoT Edge

En cada carpeta de módulo, hay varios archivos de Docker para diferentes tipos de contenedor. Use cualquiera de estos archivos que acaban con la extensión **.debug** para compilar el módulo de pruebas.

Al depurar los módulos con este método, los módulos se ejecutan sobre el entorno de ejecución de IoT Edge. El dispositivo de IoT Edge y Visual Studio Code pueden estar en la misma máquina, o como es más habitual, Visual Studio Code se encuentra en el equipo de desarrollo y el entorno de tiempo de ejecución y los módulos de IoT Edge se ejecutan en otro equipo físico. Para depurar desde Visual Studio Code, debe:

- Configurar el dispositivo de IoT Edge, compilar módulos de IoT Edge con el Dockerfile **.debug** y, después, implementar en el dispositivo de IoT Edge.
- Exponer la dirección IP y el puerto del módulo para adjuntar el depurador.
- Actualiza el archivo `launch.json` para que Visual Studio Code pueda adjuntarse al proceso del contenedor en el equipo remoto. Este archivo se encuentra en la carpeta `.vscode` del área de trabajo y se actualiza cada vez que agrega un nuevo módulo que admite la depuración.

### <a name="build-and-deploy-your-module-to-the-iot-edge-device"></a>Compilación e implementación del módulo en el dispositivo de IoT Edge

1. En Visual Studio Code, abra el archivo `deployment.debug.template.json`, que contiene la versión de depuración de las imágenes del módulo con valores de `createOptions` adecuados establecidos.

1. Si está desarrollando su módulo de Python, siga estos pasos antes de continuar:
   - Abra el archivo `main.py` y agregue este código después de la sección de importación:

      ```python
      import ptvsd
      ptvsd.enable_attach(('0.0.0.0',  5678))
      ```

   - Agregue la siguiente línea de código única a la devolución de llamada que quiera depurar:

      ```python
      ptvsd.break_into_debugger()
      ```

     Por ejemplo, si quiere depurar la función `receive_message_listener`, insertaría esa línea de código tal como se muestra a continuación:

      ```python
      def receive_message_listener(client):
          ptvsd.break_into_debugger()
          global RECEIVED_MESSAGES
          while True:
              message = client.receive_message_on_input("input1")   # blocking call
              RECEIVED_MESSAGES += 1
              print("Message received on input1")
              print( "    Data: <<{}>>".format(message.data) )
              print( "    Properties: {}".format(message.custom_properties))
              print( "    Total calls received: {}".format(RECEIVED_MESSAGES))
              print("Forwarding message to output1")
              client.send_message_to_output(message, "output1")
              print("Message successfully forwarded")
      ```

1. En la paleta de comandos de Visual Studio Code:
   1. Ejecute el comando **Azure IoT Edge: Build and Push IoT Edge solution (Azure IoT Edge: Compilación e inserción de la solución de IoT Edge)** .

   1. Seleccione el archivo `deployment.debug.template.json` para su solución.

1. En la sección **Azure IoT Hub Devices** (Dispositivos de Azure IoT Hub) de la vista Explorador de Visual Studio Code:
   1. Haga clic con el botón derecho en un Id. de dispositivo de IoT Edge y seleccione **Create Deployment for IoT Edge device** (Crear una implementación para un dispositivo individual).

      > [!TIP]
      > Para confirmar que el dispositivo que ha elegido es un dispositivo IoT Edge, selecciónelo para expandir la lista de módulos y comprobar la presencia de **$edgeHub** y **$edgeAgent**. Todos los dispositivos IoT Edge incluyen estos dos módulos.

   1. Vaya a la carpeta **config** de la solución, seleccione el archivo `deployment.debug.amd64.json` y haga clic en **Select Edge Deployment Manifest** (Seleccionar manifiesto de implementación de Edge).

Verá que la implementación se ha creado correctamente con un identificador de implementación en el terminal integrado.

Puede comprobar el estado del contenedor mediante la ejecución del comando `docker ps` en el terminal. Si el entorno de tiempo de ejecución de IoT Edge y Visual Studio Code se ejecutan en el mismo equipo, también puede comprobar el estado en la vista de Docker de Visual Studio Code.

### <a name="expose-the-ip-and-port-of-the-module-for-the-debugger"></a>Exposición de la dirección IP y el puerto del módulo para el depurador

Puede omitir esta sección si los módulos se ejecutan en el mismo equipo que Visual Studio Code, ya que está usando localhost para adjuntar al contenedor y ya tiene la configuración de puerto correcta en el Dockerfile **.debug**, en la configuración `createOptions` del contenedor del módulo y en el archivo `launch.json`. Si los módulos y Visual Studio Code se ejecutan en máquinas independientes, siga los pasos de su lenguaje de desarrollo.

- **C#, incluyendo Azure Functions**

  [Configure el canal SSH en el equipo de desarrollo y el dispositivo de IoT Edge](https://github.com/OmniSharp/omnisharp-vscode/wiki/Attaching-to-remote-processes) y, después, edite el archivo `launch.json` para adjuntar.

- **Node.js**

  - Asegúrese de que el módulo en el equipo que se va a depurar está ejecutándose y está listo para adjuntar los depuradores, y de que el puerto 9229 es accesible externamente. Puede comprobarlo abriendo `http://<target-machine-IP>:9229/json` en la máquina que se va a depurar. Esta dirección URL debe mostrar información sobre el módulo Node.js que se quiere depurar.
  
  - En el equipo de desarrollo, abra Visual Studio Code y, después, edite el archivo `launch.json` para que el valor de la dirección del perfil ***&lt;nombre del módulo&gt;* Remote Debug (Node.js)** (o el perfil ***&lt;nombre del módulo&gt;* Remote Debug [Node js en un contenedor Windows]** si el módulo se está ejecutando como un contenedor Windows) sea la dirección IP de la máquina que se va a depurar.

- **Java**

  - Compile un túnel SSH para la máquina que se quiere depurar con la ejecución de `ssh -f <username>@<target-machine> -L 5005:127.0.0.1:5005 -N`.
  
  - En el equipo de desarrollo, abra Visual Studio Code y edite el perfil ***&lt;nombre del módulo&gt;* Remote Debug (Java)** en `launch.json` para que pueda adjuntar al equipo de destino. Para más información sobre cómo editar `launch.json` y depurar Java con Visual Studio Code, consulte la sección sobre [cómo configurar el depurador](https://code.visualstudio.com/docs/java/java-debugging#_configuration).

- **Python**

  - Asegúrese de que el puerto 5678 del equipo que se va a depurar está abierto y es accesible.

  - En el código `ptvsd.enable_attach(('0.0.0.0', 5678))` que insertó anteriormente en `main.py`, cambie **0.0.0.0** a la dirección IP de la máquina que quiere depurar. Compile, inserte y vuelva a implementar los módulos de IoT Edge.

  - En el equipo de desarrollo, abra Visual Studio Code y, después, edite `launch.json` para que el valor `host` del perfil ***&lt;nombre del módulo&gt;* Remote Debug (Python)** use la dirección IP de la máquina de destino en lugar de `localhost`.

### <a name="debug-your-module"></a>Depuración del módulo

1. En la vista Depuración de Visual Studio Code seleccione el archivo de configuración de depuración del módulo. El nombre de la opción de depuración debe ser similar a ***&lt;nombre del módulo&gt;* Remote Debug**

1. Abra el archivo del módulo para su lenguaje de desarrollo y agregue un punto de interrupción:

   - **Azure Function (C#)** : agregue el punto de interrupción en el archivo `<your module name>.cs`.
   - **C#** : agregue el punto de interrupción en el archivo `Program.cs`.
   - **Node.js**: agregue el punto de interrupción en el archivo `app.js`.
   - **Java**: agregue el punto de interrupción en el archivo `App.java`.
   - **Python**: agregue el punto de interrupción al archivo `main.py` en el método de devolución de llamada donde agregó la línea `ptvsd.break_into_debugger()`.
   - **C**: agregue el punto de interrupción en el archivo `main.c`.

1. Seleccione **Iniciar depuración** o presione **F5**. Seleccione el proceso al que desea asociarlo.

1. En la vista Depuración de Visual Studio Code, verá las variables en el panel izquierdo.

> [!NOTE]
> En el ejemplo anterior se muestra cómo depurar módulos de IoT Edge en contenedores. Ha agregado puertos expuestos a la configuración `createOptions` del contenedor del módulo. Una vez finalizada la depuración de los módulos, se recomienda quitar estos puertos expuestos de los módulos de IoT Edge listos para entornos de producción.

## <a name="build-and-debug-a-module-remotely"></a>Compilación y depuración de un módulo de forma remota

Con los recientes cambios en los motores de Docker y Moby para admitir conexiones SSH y una nueva configuración de las herramientas de Azure IoT que permite la inyección de configuración de entorno en la paleta de comandos de Visual Studio Code y en los terminales de Azure IoT Edge, ahora puede compilar y depurar módulos en dispositivos remotos.

Consulte la [entrada del blog para desarrolladores de IoT](https://devblogs.microsoft.com/iotdev/easily-build-and-debug-iot-edge-modules-on-your-remote-device-with-azure-iot-edge-for-vs-code-1-9-0/) para obtener más información e instrucciones paso a paso.

## <a name="next-steps"></a>Pasos siguientes

Después de compilar el módulo, aprenda cómo [implementar módulos de Azure IoT Edge desde Visual Studio Code](how-to-deploy-modules-vscode.md).

Para desarrollar módulos para los dispositivos IoT Edge, consulte [Información y uso de los SDK de Azure IoT Hub](../iot-hub/iot-hub-devguide-sdks.md).
