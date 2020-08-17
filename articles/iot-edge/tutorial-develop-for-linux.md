---
title: 'Tutorial: Desarrollo del módulo para dispositivos Linux mediante Azure IoT Edge'
description: Este tutorial le guía por la configuración de la máquina de desarrollo y de los recursos en la nube para desarrollar módulos IoT Edge mediante contenedores de Linux para dispositivos Linux
author: kgremban
manager: philmea
ms.author: kgremban
ms.date: 07/30/2020
ms.topic: tutorial
ms.service: iot-edge
services: iot-edge
ms.custom: mvc
ms.openlocfilehash: c6cdde3ae63a2f816db7a978557f72b4b60d2677
ms.sourcegitcommit: 14bf4129a73de2b51a575c3a0a7a3b9c86387b2c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/30/2020
ms.locfileid: "87439139"
---
# <a name="tutorial-develop-iot-edge-modules-for-linux-devices"></a>Tutorial: Desarrollo de módulos IoT Edge para dispositivos Linux

Use Visual Studio Code para desarrollar e implementar código en dispositivos Linux en que se ejecute IoT Edge.

En el inicio rápido, creó un dispositivo IoT Edge mediante una máquina virtual Linux e implementó un módulo desde Azure Marketplace. En este tutorial se explica cómo desarrollar e implementar código propio en un dispositivo IoT Edge. Este artículo es un requisito previo útil para los restantes tutoriales, en los que se proporcionarán más detalles acerca de lenguajes de programación o servicios de Azure concretos.

En este tutorial se usa el ejemplo de cómo implementar un **módulo de C# en un dispositivo Linux**. Este ejemplo se ha elegido porque es el escenario de desarrollador más común para las soluciones de IoT Edge. Incluso si planea usar otro lenguaje o implementar un servicio de Azure, este tutorial sigue siendo útil para aprender sobre los conceptos y las herramientas de desarrollo. Complete esta introducción al proceso de desarrollo y, después, elija su lenguaje preferido o el servicio de Azure que prefiera para profundizar en los detalles.

En este tutorial, aprenderá a:

> [!div class="checklist"]
>
> * Configurar una máquina de desarrollo.
> * Usar las herramientas de IoT Edge para Visual Studio Code para crear un proyecto.
> * Compilar el proyecto como un contenedor y almacenarlo en un registro de contenedor de Azure.
> * Implementar el código en un dispositivo IoT Edge.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="key-concepts"></a>Conceptos clave

En este tutorial se realiza un recorrido por el desarrollo de un módulo IoT Edge. Un *módulo IoT Edge*, o a veces simplemente *módulo* para abreviar, es un contenedor con código ejecutable. En un dispositivo IoT Edge se pueden implementar uno o varios módulos. Los módulos realizan tareas concretas, como la ingesta de datos de sensores, la limpieza y el análisis de datos, o el envío a un centro de IoT. Para más información, consulte [Información sobre los módulos Azure IoT Edge](iot-edge-modules.md).

Cuando se desarrollan módulos IoT Edge, es importante conocer la diferencia entre la máquina de desarrollo y el dispositivo IoT Edge de destino en el que finalmente se implementará el módulo. El contenedor que cree para contener el código del módulo debe coincidir con el sistema operativo del *dispositivo de destino*. Por ejemplo, el escenario más común es que alguien desarrolle un módulo en un equipo Windows que pretende tener como destino un dispositivo Linux en el que se ejecuta IoT Edge. En ese caso, el sistema operativo del contenedor sería Linux. A medida que avance en este tutorial, tenga en cuenta la diferencia entre el *sistema operativo de la máquina de desarrollo* y el *sistema operativo del contenedor*.

Este tutorial va dirigido a dispositivos Linux que ejecutan IoT Edge. Puede usar el sistema operativo que prefiera, siempre que dicha máquina ejecute contenedores Linux. Se recomienda usar Visual Studio Code para desarrollar para dispositivos Linux, así que es lo que se va a usar en este tutorial. También se puede usar Visual Studio, aunque hay diferencias en lo relativo a la compatibilidad entre ambas herramientas.

En la tabla siguiente se enumeran los escenarios de desarrollo compatibles con los **contenedores de Linux** en Visual Studio Code y Visual Studio.

|   | Visual Studio Code | Visual Studio 2017/2019 |
| - | ------------------ | ------------------ |
| **Arquitectura de dispositivos Linux** | Linux AMD64 <br> Linux ARM32 | Linux AMD64 <br> Linux ARM32 |
| **Servicios de Azure** | Azure Functions <br> Azure Stream Analytics <br> Azure Machine Learning |   |
| **Idiomas** | C <br> C# <br> Java <br> Node.js <br> Python | C <br> C# |
| **Más información** | [Azure IoT Edge para Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-edge) | [Azure IoT Edge Tools para Visual Studio 2017](https://marketplace.visualstudio.com/items?itemName=vsc-iot.vsiotedgetools) <br> [Azure IoT Edge Tools para Visual Studio 2019](https://marketplace.visualstudio.com/items?itemName=vsc-iot.vs16iotedgetools) |

>[!NOTE]
>La compatibilidad con dispositivos ARM64 de Linux está disponible en [versión preliminar pública](https://azure.microsoft.com/support/legal/preview-supplemental-terms/). Para más información, consulte [Desarrollo y depuración de módulos ARM64 IoT Edge en Visual Studio Code (versión preliminar)](https://devblogs.microsoft.com/iotdev/develop-and-debug-arm64-iot-edge-modules-in-visual-studio-code-preview).

En este tutorial se enseñan los pasos del desarrollo en Visual Studio Code. Si prefiere usar Visual Studio, consulte las instrucciones de [Uso de Visual Studio 2019 para desarrollar y depurar los módulos Azure IoT Edge](how-to-visual-studio-develop-module.md).

## <a name="prerequisites"></a>Prerrequisitos

Una máquina de desarrollo:

* Puede usar su propio equipo o una máquina virtual, según sus preferencias de desarrollo.
  * Asegúrese de que el equipo de desarrollo admite la virtualización anidada. Esta funcionalidad es necesaria para ejecutar un motor de contenedor, que se instala en la sección siguiente.
* La mayor parte de los sistemas operativos que pueden ejecutar un motor de contenedor se pueden usar para desarrollar módulos IoT Edge para dispositivos Linux. En este tutorial se usa un equipo Windows, pero se indican las diferencias conocidas con macOS o Linux.
* Instale [Git](https://git-scm.com/) para extraer paquetes de plantillas de módulos más adelante.  
* [Extensión de C# para Visual Studio Code (con tecnología de OmniSharp)](https://marketplace.visualstudio.com/items?itemName=ms-dotnettools.csharp)
* [SDK de .NET Core 2.1](https://www.microsoft.com/net/download).

Un dispositivo Azure IoT Edge en Linux:

* Se recomienda no ejecutar IoT Edge en la máquina de desarrollo, sino usar un dispositivo independiente. Esta distinción entre la máquina de desarrollo y el dispositivo IoT Edge refleja con mayor precisión un verdadero escenario de implementación y ayuda a tener claros los distintos conceptos.
* Si no tiene un segundo dispositivo disponible, use el artículo del inicio rápido para crear un dispositivo IoT Edge en Azure con una [máquina virtual Linux](quickstart-linux.md).

Recursos en la nube:

* Una instancia de [IoT Hub](../iot-hub/iot-hub-create-through-portal.md) de nivel estándar o gratis en Azure.

## <a name="install-container-engine"></a>Instalación de un motor de contenedor

Los módulos IoT Edge se empaquetan como contenedores, por lo que necesita un motor de contenedor en la máquina de desarrollo para crear y administrar los contenedores. Se recomienda Docker Desktop para el desarrollo debido a su popularidad y la compatibilidad de las características. Docker Desktop en Windows permite cambiar entre contenedores Linux y Windows para poder desarrollar fácilmente módulos para los distintos tipos de dispositivos IoT Edge.

Use la documentación de Docker para realizar la instalación en la máquina de desarrollo:

* [Instalación de Docker Desktop para Windows](https://docs.docker.com/docker-for-windows/install/)

  * Cuando se instala Docker Desktop para Windows, se le pregunta si desea utilizar contenedores de Linux o de Windows. Esta decisión se puede cambiar en cualquier momento mediante un cambio fácil. En este tutorial se usan contenedores Linux porque nuestros módulos van dirigidos a dispositivos Linux. Para más información, consulte [Switch between Windows and Linux containers](https://docs.docker.com/docker-for-windows/#switch-between-windows-and-linux-containers) (Cambio entre contenedores de Windows y Linux).

* [Instalación de Docker Desktop para Mac](https://docs.docker.com/docker-for-mac/install/)

* Lea [Acerca de CE de Docker](https://docs.docker.com/install/) para obtener información acerca de la instalación en varias plataformas Linux.
  * En el caso del Subsistema de Windows para Linux (WSL), instale Docker Desktop para Windows.

## <a name="set-up-vs-code-and-tools"></a>Configuración de VS Code y de herramientas

Use las extensiones de IoT para Visual Studio Code para desarrollar módulos IoT Edge. Estas extensiones proporcionan plantillas de proyecto, automatizan la creación del manifiesto de implementación y permiten supervisar y administrar dispositivos IoT Edge. En esta sección, se instalan Visual Studio Code y la extensión de IoT y luego se configurar una cuenta de Azure para administrar recursos de IoT Hub desde dentro de Visual Studio Code.

1. Instale [Visual Studio Code](https://code.visualstudio.com/) en la máquina de desarrollo.

2. Una vez que finalice la instalación, seleccione **Ver** > **Extensiones**.

3. Busque **Azure IoT Tools**, que es realmente una colección de extensiones que le ayudan a interactuar con dispositivos IoT e IoT Hub, así como desarrollar módulos IoT Edge.

4. Seleccione **Instalar**. Cada extensión incluida se instala individualmente.

5. Cuando las extensiones se realizan mediante la instalación, abra la paleta de comandos, para lo que debe seleccionar **Ver** > **Paleta de comandos**.

6. En la paleta de comandos, busque y seleccione **Azure: Iniciar sesión**. Siga las indicaciones para iniciar sesión en su cuenta de Azure.

7. En la paleta de comandos, busque y seleccione **Azure IoT Hub: Select IoT Hub** (Azure IoT Hub: seleccione IoT Hub). Siga las instrucciones para seleccionar una suscripción de Azure y un centro de IoT.

8. Abra la sección del explorador de Visual Studio Code, para lo que puede seleccionar el icono pertinente en la barra de actividad de la izquierda, o bien seleccionar **Ver** > **Explorador**.

9. En la parte inferior de dicha sección, expanda el menú **Azure IoT Hub / Devices** (Azure IoT Hub/Dispositivos). Debería ver los dispositivos y los dispositivos IoT Edge asociados con el centro de IoT que seleccionó en la paleta de comandos.

   ![Visualización de los dispositivos de un centro de IoT](./media/tutorial-develop-for-linux/view-iot-hub-devices.png)

[!INCLUDE [iot-edge-create-container-registry](../../includes/iot-edge-create-container-registry.md)]

## <a name="create-a-new-module-project"></a>Creación de un nuevo proyecto de módulo

La extensión Azure IoT Tools proporciona plantillas de proyecto para todos los lenguajes del módulo IoT Edge admitidos que haya en Visual Studio Code. Estas plantillas tienen todos los archivos y el código que se necesita para implementar un módulo de trabajo y probar IoT Edge o le proporcionan a un punto de partida para personalizar la plantilla con su propia lógica de negocios.

En este tutorial, usamos la plantilla del módulo de C# porque es la plantilla que se usa con más frecuencia.

### <a name="create-a-project-template"></a>Creación de una plantilla de proyecto

En la paleta de comandos de Visual Studio Code, busque y seleccione **Azure IoT Edge: New IoT Edge solution** (Nueva solución de IoT Edge). Siga las instrucciones y use los siguientes valores para crear la solución:

   | Campo | Value |
   | ----- | ----- |
   | Seleccionar carpeta | Elija la ubicación en el equipo de desarrollo en la que VS Code creará los archivos de la solución. |
   | Proporcionar un nombre de la solución | Escriba un nombre descriptivo para la solución o acepte el valor predeterminado **EdgeSolution**. |
   | Seleccionar plantilla del módulo | Elija **Módulo C#** . |
   | Proporcionar un nombre de módulo | Acepte el valor predeterminado, **SampleModule**. |
   | Proporcionar repositorio de imágenes de Docker del módulo | Un repositorio de imágenes incluye el nombre del registro de contenedor y el nombre de la imagen de contenedor. La imagen de contenedor se rellena previamente con el nombre que proporcionó en el último paso. Reemplace **localhost:5000** por el valor de **Servidor de inicio de sesión** del registro de contenedor de Azure. Puede recuperar el valor de Servidor de inicio de sesión de la página Información general del registro de contenedor en Azure Portal. <br><br> El repositorio de imágenes final es similar a \<registry name\>.azurecr.io/samplemodule. |

   ![Especificación del repositorio de imágenes de Docker](./media/tutorial-develop-for-linux/image-repository.png)

Cuando la nueva solución se cargue en la ventana de Visual Studio Code, dedique unos minutos a familiarizarse con los archivos que ha creado:

* La carpeta **.vscode** contiene un archivo denominado **launch.json**, que se usa para la depuración de módulos.
* La carpeta **modules** contiene una carpeta para cada módulo de la solución. En este momento, solo debería ser **SampleModule**, o el nombre que haya dado al módulo. La carpeta SampleModule contiene el código de programa principal, los metadatos del módulo y varios archivos de Docker.
* El archivo **.env** almacena las credenciales en el registro de contenedor. Estas credenciales se comparten con el dispositivo IoT Edge para que tenga acceso para extraer las imágenes de contenedor.
* Tanto el archivo **deployment.debug.template.json** como el archivo **deployment.template.json** son plantillas que le ayudan a crear un manifiesto de implementación. Un *manifiesto de implementación* es un archivo que define exactamente qué módulos se desean implementar en un dispositivo, cómo se deben configurar y cómo se pueden comunicar tanto entre sí como con la nube. Los archivos de la plantilla utilizan punteros para algunos valores. Cuando la plantilla se transforma en un verdadero manifiesto de implementación, los punteros se reemplazan por valores que se han tomado de otros archivos de solución. Busque los dos marcadores de posición comunes en la plantilla de implementación:

  * En la sección de credenciales del registro, la dirección se rellena automáticamente a partir de la información que proporcionó al crear la solución. Sin embargo, el nombre de usuario y la contraseña hacen referencia a las variables almacenadas en el archivo .env. Esta configuración es por motivos de seguridad, ya que Git ignora el archivo .env, pero no sucede lo mismo con la plantilla de implementación.
  * En la sección SampleModule, la imagen de contenedor no se rellena, aunque haya especificado el repositorio de imágenes al crear la solución. Este marcador de posición apunta al archivo **module.json** de la carpeta SampleModule. Si va a dicho archivo, verá que el campo de imagen contiene el repositorio, sino también un valor de etiqueta, que se compone de la versión y la plataforma del contenedor. La versión se puede iterar manualmente como parte del ciclo de desarrollo y se puede seleccionar la plataforma de contenedor mediante un modificador que se presentará en esta misma sección más adelante.

### <a name="provide-your-registry-credentials-to-the-iot-edge-agent"></a>Especificación de la credenciales del registro para el agente de IoT Edge

El archivo del entorno almacena las credenciales del registro de contenedor y las comparte con el runtime de IoT Edge. El entorno de ejecución necesita estas credenciales para extraer las imágenes de contenedor e insertarlas en el dispositivo IoT Edge.

La extensión de IoT Edge intenta extraer de Azure las credenciales del registro del contenedor y rellenar con ellas el archivo de entorno. Compruebe si las credenciales ya están incluidas. Si no lo están, agréguelas ahora:

1. Abra el archivo **.env** en la solución del módulo.
2. Agregue los valores de **nombre de usuario** y **contraseña** que ha copiado del registro de contenedor de Azure.
3. Guarde los cambios en el archivo .env.

### <a name="select-your-target-architecture"></a>Selección de la arquitectura de destino

Actualmente, Visual Studio Code puede desarrollar módulos de C# para dispositivos Linux AMD64 y ARM32v7. Debe seleccionar qué arquitectura tiene como destino con cada solución, ya que ello afecta a la forma en que se crea y ejecuta el contenedor. La opción predeterminada es Linux AMD64.

1. Abra la paleta de comandos y busque **Azure IoT Edge: Set Default Target Platform for Edge Solution** (Establecer la plataforma de destino predeterminada para la solución perimetral), o bien seleccione el icono de acceso directo en la barra lateral en la parte inferior de la ventana.

   ![Seleccionar el icono de la arquitectura en la barra lateral](./media/tutorial-develop-for-linux/select-architecture.png)

2. En la paleta de comandos, seleccione la arquitectura de destino en la lista de opciones. Para este tutorial, usamos una máquina virtual Ubuntu como dispositivo IoT Edge, por lo que mantendrá el valor predeterminado **amd64**.

### <a name="review-the-sample-code"></a>Revisión del código de ejemplo

La plantilla de solución que ha creado incluye código de ejemplo de un módulo IoT Edge. Dicho módulo simplemente recibe los mensajes y, después, los pasa. La funcionalidad de canalización muestra un concepto importante de IoT Edge, cómo se comunican los módulos entre sí.

Cada módulo puede tener varias colas de *entrada* y *salida* declaran en su código. El centro de IoT Edge que se ejecuta en el dispositivo enruta los mensajes de la salida de un módulo en la entrada de uno o varios módulos. El código específico para declarar las entradas y salidas varía de un lenguaje a otro, pero el concepto es el mismo en todos los módulos. Para más información acerca del enrutamiento entre módulos, consulte [Declaración de rutas](module-composition.md#declare-routes).

El código de C# de ejemplo que acompaña a la plantilla del proyecto usa la [clase ModuleClient](https://docs.microsoft.com/dotnet/api/microsoft.azure.devices.client.moduleclient?view=azure-dotnet) del SDK de IoT Hub para .NET.

1. Abra el archivo **Program.cs**, que está dentro de la carpeta **modules/SampleModule/** .

2. En program.cs, busque el método **SetInputMessageHandlerAsync**.

3. El método [SetInputMessageHandlerAsync](https://docs.microsoft.com/dotnet/api/microsoft.azure.devices.client.moduleclient.setinputmessagehandlerasync?view=azure-dotnet) configura una cola de entrada para recibir los mensajes entrantes. Revise este método y vea cómo inicializa una cola de entrada llamada **input1**.

   ![Buscar el nombre de entrada en el constructor de SetInputMessageCallback](./media/tutorial-develop-for-linux/declare-input-queue.png)

4. Luego, busque el método **SendEventAsync**.

5. El método [SendEventAsync](https://docs.microsoft.com/dotnet/api/microsoft.azure.devices.client.moduleclient.sendeventasync?view=azure-dotnet) procesa los mensajes recibidos y configura una cola de salida para difundirlos. Revise dicho método y verá que inicializa una cola de salida llamada **output1**.

   ![Buscar el nombre de salida en SendEventToOutputAsync](./media/tutorial-develop-for-linux/declare-output-queue.png)

6. Abra el archivo **deployment.template.json**.

7. Busque la propiedad **modules** de las propiedades deseadas de $edgeAgent.

   Debería haber dos módulos. Uno es el módulo **SimulatedTemperatureSensor**, que se incluye en todas las plantillas de forma predeterminada para proporcionar datos de temperatura simulados que se pueden usar para probar los módulos. El otro es el módulo **SampleModule** que creó como parte de esta solución.

8. En la parte inferior del archivo, busque las propiedades deseadas del módulo **$edgeHub**.

   Una de las funciones del módulo del centro de IoT Edge es enrutar mensajes entre todos los módulos de una implementación. Revise los valores de la propiedad **routes**. Una ruta, **SampleModuleToIoTHub**, usa un carácter comodín ( **\*** ) para indicar cualquier mensaje que proceda de cualquier cola de salida del módulo SampleModule. Dichos mensajes van a *$upstream*, que es un nombre reservado que indica IoT Hub. La otra ruta, **sensorToSampleModule**, toma los mensajes que proceden del módulo SimulatedTemperatureSensor y los enruta a la cola de entrada *input1*, que ha visto que se inicializaba en el código de SampleModule.

   ![Revisar las rutas de deployment.template.json](./media/tutorial-develop-for-linux/deployment-routes.png)

## <a name="build-and-push-your-solution"></a>Compilación e inserción de la solución

Ha revisado el código del módulo y la plantilla de implementación para conocer algunos conceptos clave de la implementación. Ya está listo para que compilar la imagen de contenedor SampleModule e insertarla en el registro de contenedor. Con la extensión de las herramientas de IoT para Visual Studio Code, este paso también genera el manifiesto de implementación en función de la información del archivo de plantilla y la información del módulo a partir de los archivos de la solución.

### <a name="sign-in-to-docker"></a>Inicio de sesión en Docker

Especifique las credenciales del registro de contenedor a Docker para que puede insertar la imagen de contenedor para que se almacene en el registro.

1. Abra el terminal integrado de Visual Studio Code, para lo que debe seleccionar **Ver** > **Terminal**.

2. Inicie sesión en Docker con las credenciales del registro de contenedor de Azure que guardó después de crear el registro.

   ```cmd/sh
   docker login -u <ACR username> -p <ACR password> <ACR login server>
   ```

   Puede recibir una advertencia de seguridad en la que se recomiende el uso de `--password-stdin`. Aunque ese procedimiento se recomienda para escenarios de producción, está fuera del ámbito de este tutorial. Para más información, consulte la referencia de [docker login](https://docs.docker.com/engine/reference/commandline/login/#provide-a-password-using-stdin).
   
3. Inicio de sesión en Azure Container Registry

   ```cmd/sh
   az acr login -n <ACR registry name>
   ```

### <a name="build-and-push"></a>Compilación e inserción

Visual Studio Code ahora tiene acceso a su registro de contenedor, por lo que es el momento de convertir el código de la solución en una imagen de contenedor.

1. En el explorador de Visual Studio Code, haga clic con el botón derecho en el archivo **deployment.template.json** y seleccione **Build and Push IoT Edge solution** (Compilar e insertar solución IoT Edge).

   ![Compilación e inserción de módulos IoT Edge](./media/tutorial-develop-for-linux/build-and-push-modules.png)

   El comando de compilación e inserción inicia tres operaciones. En primer lugar, se crea una nueva carpeta en la solución denominada **config** que contiene los archivos del manifiesto de la implementación completa, con la información de la plantilla de implementación y otros archivos de la solución. En segundo lugar, ejecuta `docker build` para generar la imagen de contenedor basándose en el Dockerfile adecuado para la arquitectura de destino. A continuación, ejecuta `docker push` para insertar el repositorio de imágenes en el registro de contenedor.

   Este proceso puede tardar varios minutos la primera vez, pero es más rápido la próxima vez que ejecute los comandos.

2. Abra el archivo **deployment.amd64.json** en la carpeta de configuración recién creada. El nombre de archivo refleja la arquitectura de destino, por lo que será diferente si ha elegido una otra arquitectura.

3. Tenga en cuenta que los dos parámetros que tenían marcadores de posición ahora están rellenos con los valores adecuados. La sección **registryCredentials** tiene el nombre de usuario y la contraseña del registro que se extrajeron del archivo .env. **SampleModule** tiene el repositorio de imágenes completo con el nombre, la versión y la etiqueta de arquitectura del archivo module.json.

4. Abra el archivo **module.json** de la carpeta SampleModule.

5. Cambie el número de versión de la imagen del módulo. (la versión, no la versión $schema). Por ejemplo, incremente el número de versión de la revisión a **0.0.2** como si hubiéramos hecho una pequeña corrección en el código del módulo.

   >[!TIP]
   >Las versiones de los módulos permiten el control de versiones y le permiten probar los cambios en un pequeño conjunto de dispositivos antes de implementar actualizaciones en la producción. Si no incrementa la versión del módulo antes de la compilación e inserción, entonces sobrescribirá el repositorio en el registro de contenedor.

6. Guarde los cambios en el archivo module.json.

7. Vuelva a hacer clic con el botón derecho en el archivo **deployment.template.json** y seleccione **Build and Push IoT Edge solution** (Compilar e insertar solución IoT Edge) de nuevo.

8. Vuelva a abrir el archivo **deployment.amd64.json** de nuevo. Observe que no se creó un nuevo archivo cuando ejecutó el comando de compilación e inserción de nuevo. Más bien, el mismo archivo se actualizó para reflejar los cambios. La imagen de SampleModule ahora apunta a la versión 0.0.2 del contenedor.

9. Para comprobar mejor lo que hizo el comando de compilación e inserción, vaya a [Azure Portal](https://portal.azure.com) y navegue al registro de contenedor.

10. En el registro de contenedor, seleccione **Repositorios** y después **samplemodule**. Compruebe que ambas versiones de la imagen se hayan insertado en el registro.

    ![Visualización de ambas versiones de la imagen en el registro de contenedor](./media/tutorial-develop-for-linux/view-repository-versions.png)

<!--Alternative steps: Use VS Code Docker tools to view ACR images with tags-->

### <a name="troubleshoot"></a>Solución de problemas

Si encuentra errores al compilar e insertar la imagen del módulo, a menudo tiene que ver con la configuración de Docker en la máquina de desarrollo. Utilice las siguientes comprobaciones para revisar la configuración:

* ¿Ha ejecutado el comando `docker login` mediante las credenciales que copió del registro de contenedor? Estas credenciales son diferentes a las que se utilizan para iniciar sesión en Azure.
* ¿Es el repositorio de contenedor correcto? ¿Tiene el nombre correcto del registro de contenedor y el nombre correcto del módulo? Abra el archivo **module.json** de la carpeta SampleModule para comprobarlo. El valor del repositorio debería ser similar a **\<registry name\>.azurecr.io/samplemodule**.
* Si utilizó un nombre diferente a **SampleModule** para el módulo, ¿es coherente ese nombre en toda la solución?
* ¿La máquina funciona con el mismo tipo de contenedores que está creando? Este tutorial es para dispositivos IoT Edge de Linux, por lo que Visual Studio Code debería indicar **amd64** o **arm32v7** en la barra lateral y Docker Desktop debe ejecutar contenedores Linux.  

## <a name="deploy-modules-to-device"></a>Implementación de módulos en el dispositivo

Ha verificado que las imágenes de contenedor creadas se almacenan en el registro de contenedor, por lo que es el momento de implementarlas en un dispositivo. Asegúrese de que el dispositivo IoT Edge está en funcionamiento.

1. En el explorador de Visual Studio Code, en la sección **Azure IoT Hub**, expanda **Dispositivos** para ver la lista de dispositivos IoT.

2. Haga clic con el dispositivo IoT Edge que desea implementar y seleccione **Create Deployment for IoT Edge device** (Crear una implementación para un dispositivo individual).

   ![Crear una implementación para un dispositivo individual](./media/tutorial-develop-for-linux/create-deployment.png)

3. En el Explorador de archivos, vaya a la carpeta **config** y seleccione el archivo **deployment.amd64.json**.

   No utilice el archivo deployment.template.json, que no contiene todos las credenciales del registro de contenedor o los valores de imagen del módulo. Si el destino es un dispositivo ARM32 con Linux, el manifiesto de implementación se denominará deployment.arm32v7.json.

4. En el dispositivo, expanda **Módulos** para ver una lista de módulos implementados y en ejecución. Haga clic en el botón Actualizar. Debería ver que los nuevos módulos SimulatedTemperatureSensor y SampleModule se ejecutan en el dispositivo.

   Los módulos pueden tardar unos minutos en iniciarse. El entorno de ejecución de Azure IoT Edge necesita recibir su nuevo manifiesto de implementación, extraer las imágenes de los módulos del entorno de ejecución del contenedor y, después, iniciar cada nuevo módulo.

   ![Visualización de los módulos que se ejecutan en el dispositivo IoT Edge](./media/tutorial-develop-for-linux/view-running-modules.png)

## <a name="view-messages-from-device"></a>Visualización de los mensajes desde el dispositivo

El código SampleModule recibe mensajes mediante la cola de entrada y los pasa por la cola de salida. El manifiesto de implementación declaraba las rutas que pasaban a SampleModule los mensajes de SimulatedTemperatureSensor, y luego reenviaba los mensajes de SampleModule a la instancia de IoT Hub. Las herramientas de Azure IoT para Visual Studio Code permiten ver los mensajes que llegan a IoT Hub desde dispositivos individuales.

1. En el explorador de Visual Studio Code, haga clic con el botón derecho en el dispositivo IoT Edge que desea supervisar y seleccione **Start Monitoring Built-in Event Endpoint** (Iniciar supervisión del punto de conexión del evento integrado).

2. En la ventana de salida podrá ver si llegan mensajes al centro de IoT.

   ![Visualización de los mensajes entrantes del dispositivo a la nube](./media/tutorial-develop-for-linux/view-d2c-messages.png)

## <a name="view-changes-on-device"></a>Visualización de los cambios en el dispositivo

Si desea ver lo que está ocurriendo en su propio dispositivo, utilice los comandos de esta sección para inspeccionar el entorno de ejecución de Azure IoT Edge y los módulos que se ejecutan en el dispositivo.

Los comandos de esta sección son para el dispositivo IoT Edge, no en la máquina de desarrollo. Si usa una máquina virtual para el dispositivo IoT Edge, conéctese a ella ahora. En Azure, vaya a la página de información general de la máquina virtual y seleccione **Conectar** para acceder a la conexión de Secure Shell.

* Vea todos los módulos implementados en el dispositivo y compruebe su estado:

   ```bash
   iotedge list
   ```

   Debería ver cuatro módulos: los dos módulos personalizados del entorno de ejecución de IoT Edge, SimulatedTemperatureSensor y SampleModule. Los cuatro deben aparecer como en ejecución.

* Inspeccione los registros para un módulo específico:

   ```bash
   iotedge logs <module name>
   ```

   Los módulos IoT Edge distinguen mayúsculas de minúsculas.

   Los registros de SimulatedTemperatureSensor y SampleModule deben mostrar los mensajes que están procesando. El módulo edgeAgent es responsable de iniciar los otros módulos, por lo que sus registros tendrá información sobre cómo implementar el manifiesto de implementación. Si algún módulo no aparece en la lista o no se está ejecutando, es probable que los registros de edgeAgent contengan los errores. El módulo edgeHub es responsable de las comunicaciones entre los módulos e IoT Hub. Si los módulos están en funcionamiento, pero los mensajes no llegan al centro de IoT, los registros de edgeHub probablemente contendrán los errores.

## <a name="clean-up-resources"></a>Limpieza de recursos

Si prevé seguir con el siguiente artículo recomendado, puede mantener los recursos y las configuraciones que ya ha creado y volverlos a utilizar. También puede seguir usando el mismo dispositivo de IoT Edge como dispositivo de prueba.

En caso contrario, para evitar gastos, puede eliminar las configuraciones locales y los recursos de Azure que creó en este artículo.

[!INCLUDE [iot-edge-clean-up-cloud-resources](../../includes/iot-edge-clean-up-cloud-resources.md)]

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha configurado Visual Studio Code en la máquina de desarrollo y ha implementado el primer módulo IoT Edge desde él. Ahora que conoce los conceptos básicos, intente agregar funcionalidad a un módulo para que pueda analizar los datos que pasan por él. Elija el lenguaje que prefiera:

> [!div class="nextstepaction"]
> [C](tutorial-c-module.md)
> [C#](tutorial-csharp-module.md)
> [Java](tutorial-java-module.md)
> [Node.js](tutorial-node-module.md)
> [Python](tutorial-python-module.md)
