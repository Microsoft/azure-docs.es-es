---
title: 'Tutorial: Desarrollo del módulo para dispositivos Windows mediante Azure IoT Edge'
description: Este tutorial le guía por la configuración de la máquina de desarrollo y de los recursos en la nube para desarrollar módulos IoT Edge mediante contenedores de Windows para dispositivos Windows.
author: kgremban
manager: philmea
ms.author: kgremban
ms.date: 07/30/2020
ms.topic: tutorial
ms.service: iot-edge
services: iot-edge
ms.custom: mvc
ms.openlocfilehash: d4846a69f548c99de735cc9d9e06bd6bb263b4bd
ms.sourcegitcommit: 14bf4129a73de2b51a575c3a0a7a3b9c86387b2c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/30/2020
ms.locfileid: "87439694"
---
# <a name="tutorial-develop-iot-edge-modules-for-windows-devices"></a>Tutorial: Desarrollo de módulos IoT Edge para dispositivos Windows

Use Visual Studio para desarrollar e implementar código en dispositivos Windows en los que se ejecute IoT Edge.

En el inicio rápido, ha creado un dispositivo IoT Edge mediante una máquina virtual Windows y ha implementado un módulo pregenerado desde Azure Marketplace. En este tutorial se explica todo lo necesario para desarrollar e implementar código propio en un dispositivo IoT Edge. Este tutorial es un requisito previo útil para los restantes tutoriales, en los que se proporcionarán más detalles acerca de lenguajes de programación o servicios de Azure concretos.

En este tutorial se usa el ejemplo de cómo implementar un **módulo de C# en un dispositivo Windows**. Se eligió este ejemplo porque es el escenario de desarrollo más común. Si le interesa desarrollar en un lenguaje distinto o planea implementar los servicios de Azure como módulos, este tutorial de todos modos será útil para aprender sobre las herramientas de desarrollo. Una vez que conozca los conceptos de desarrollo, puede elegir el lenguaje o servicio de Azure que prefiera, con el fin de profundizar en él.

En este tutorial, aprenderá a:

> [!div class="checklist"]
>
> * Configurar una máquina de desarrollo.
> * Usar las herramientas de IoT Edge para Visual Studio 2017 para crear un proyecto.
> * Compilar el proyecto como un contenedor y almacenarlo en un registro de contenedor de Azure.
> * Implementar el código en un dispositivo IoT Edge.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="key-concepts"></a>Conceptos clave

En este tutorial se realiza un recorrido por el desarrollo de un módulo IoT Edge. Un *módulo IoT Edge*, o a veces simplemente *módulo* para abreviar, es un contenedor que alberga código ejecutable. En un dispositivo IoT Edge se pueden implementar uno o varios módulos. Los módulos realizan tareas concretas, como la ingesta de datos de sensores, la realización de análisis de datos o de operaciones de limpieza de datos, o el envío a un centro de IoT. Para más información, consulte [Información sobre los módulos Azure IoT Edge](iot-edge-modules.md).

Cuando se desarrollan módulos IoT Edge, es importante conocer la diferencia entre la máquina de desarrollo y el dispositivo IoT Edge de destino en el que finalmente se implementará el módulo. El contenedor que cree para contener el código del módulo debe coincidir con el sistema operativo del *dispositivo de destino*. Para el desarrollo del contenedor de Windows, este concepto es más sencillo porque los contenedores de Windows solo se ejecutan en sistemas operativos Windows. Pero podría, por ejemplo, usar la máquina de desarrollo Windows para crear módulos para dispositivos IoT Edge en Linux. En ese caso, tendría que asegurarse de que la máquina de desarrollo ejecuta contenedores de Linux. A medida que avance en este tutorial, tenga en cuenta la diferencia entre el *sistema operativo de la máquina de desarrollo* y el *sistema operativo del contenedor*.

Este tutorial va dirigido a dispositivos Windows que ejecutan IoT Edge. Los dispositivos IoT Edge de Windows usan contenedores de Windows. Se recomienda usar Visual Studio 2017 para el desarrollo de dispositivos Windows, así que es lo que se va a usar en este tutorial. También se puede usar Visual Studio Code, aunque hay diferencias en lo relativo a la compatibilidad entre ambas herramientas.

En la tabla siguiente se enumeran los escenarios de desarrollo compatibles con los **contenedores de Windows** en Visual Studio Code y Visual Studio.

|   | Visual Studio Code | Visual Studio 2017/2019 |
| - | ------------------ | ------------------ |
| **Servicios de Azure** | Azure Functions <br> Azure Stream Analytics |   |
| **Idiomas** | C# (no se admite la depuración) | C <br> C# |
| **Más información** | [Azure IoT Edge para Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-edge) | [Azure IoT Edge Tools para Visual Studio 2017](https://marketplace.visualstudio.com/items?itemName=vsc-iot.vsiotedgetools)<br>[Azure IoT Edge Tools para Visual Studio 2019](https://marketplace.visualstudio.com/items?itemName=vsc-iot.vs16iotedgetools) |

## <a name="prerequisites"></a>Prerrequisitos

Una máquina de desarrollo:

* Windows 10 con actualización 1809 o posterior.
* Puede usar su propio equipo o una máquina virtual, según sus preferencias de desarrollo.
  * Asegúrese de que el equipo de desarrollo admite la virtualización anidada. Esta funcionalidad es necesaria para ejecutar un motor de contenedor, que se instala en la sección siguiente.
* Instale [Git](https://git-scm.com/).

Un dispositivo Azure IoT Edge en Windows:

* Se recomienda no ejecutar IoT Edge en la máquina de desarrollo, sino usar un dispositivo independiente. Esta distinción entre la máquina de desarrollo y el dispositivo IoT Edge refleja con mayor precisión un verdadero escenario de implementación y ayuda a tener claros los distintos conceptos.
* Si no tiene un segundo dispositivo disponible, use el artículo del inicio rápido para crear un dispositivo IoT Edge en Azure con una [máquina virtual Windows](quickstart.md).

Recursos en la nube:

* Una instancia de [IoT Hub](../iot-hub/iot-hub-create-through-portal.md) de nivel estándar o gratis en Azure.

## <a name="install-container-engine"></a>Instalación de un motor de contenedor

Los módulos IoT Edge se empaquetan como contenedores, por lo que necesita un motor de contenedor en la máquina de desarrollo para crear y administrar los contenedores. Se recomienda usar Docker Desktop para el desarrollo debido a gran cantidad de características y su popularidad como motor de contenedor. Con Docker Desktop en un equipo Windows, se puede cambiar entre contenedores de Linux y Windows, con el fin de poder desarrollar fácilmente módulos para los distintos tipos de dispositivos IoT Edge.

Use la documentación de Docker para realizar la instalación en la máquina de desarrollo:

* [Instalación de Docker Desktop para Windows](https://docs.docker.com/docker-for-windows/install/)

  * Cuando se instala Docker Desktop para Windows, se le pregunta si desea utilizar contenedores de Linux o de Windows. Para este tutorial, use los **contenedores de Windows**. Para más información, consulte [Switch between Windows and Linux containers](https://docs.docker.com/docker-for-windows/#switch-between-windows-and-linux-containers) (Cambio entre contenedores de Windows y Linux).

## <a name="set-up-visual-studio-and-tools"></a>Configuración de Visual Studio y sus herramientas

Las extensiones de IoT para Visual Studio lo ayudan a desarrollar módulos de IoT Edge. Estas extensiones proporcionan plantillas de proyecto, automatizan la creación del manifiesto de implementación y permiten supervisar y administrar dispositivos IoT Edge. En esta sección, se instalan Visual Studio y la extensión de IoT Edge y luego se configura una cuenta de Azure para administrar recursos de IoT Hub desde Visual Studio.

En este tutorial se enseñan los pasos del desarrollo en Visual Studio 2019. Si usa Visual Studio 2017 (versión 15.7 o superior), los pasos son muy similares. Si prefiere usar Visual Studio Code, consulte las instrucciones de [Uso de Visual Studio Code para desarrollar y depurar los módulos Azure IoT Edge](how-to-vs-code-develop-module.md).

1. Prepare Visual Studio 2019 en la máquina de desarrollo.

   * Si aún no tiene Visual Studio en la máquina de desarrollo, [instale Visual Studio 2019](https://docs.microsoft.com/visualstudio/install/install-visual-studio) con las cargas de trabajo siguientes:

      * Desarrollo de Azure
      * Desarrollo para el escritorio con C++
      * Desarrollo multiplataforma de .NET Core

   * Si ya tiene instalado Visual Studio 2019 en la máquina de desarrollo, siga los pasos que aparecen en el artículo [Modificación de Visual Studio](https://docs.microsoft.com/visualstudio/install/modify-visual-studio) para agregar las cargas de trabajo necesarias.

2. Descargue e instale la extensión [Azure IoT Edge Tools](https://marketplace.visualstudio.com/items?itemName=vsc-iot.vs16iotedgetools) para Visual Studio 2019.

   Si usa Visual Studio 2017 (versión 15.7 o superior), descargue e instale [Azure IoT Edge Tools para Visual Studio 2017](https://marketplace.visualstudio.com/items?itemName=vsc-iot.vsiotedgetools).

3. Cuando finalicen las instalaciones, abra Visual Studio 2019 y seleccione **Continuar sin código**.

4. Seleccione **Ver** > **Cloud Explorer**.

5. Seleccione el icono del perfil en Cloud Explorer e inicie sesión en la cuenta de Azure si aún no lo ha hecho.

6. Una vez que inicie sesión, se muestran las suscripciones de Azure. Expanda la suscripción que tiene el centro de IoT.

7. En la suscripción, expanda **Centros de IoT** y, luego, su centro de IoT. Debería ver una lista de dispositivos IoT y puede usar este explorador para administrarlos.

   ![Acceso a los recursos de IoT Hub en Cloud Explorer](./media/tutorial-develop-for-windows/cloud-explorer-view-hub.png)

[!INCLUDE [iot-edge-create-container-registry](../../includes/iot-edge-create-container-registry.md)]

## <a name="create-a-new-module-project"></a>Creación de un nuevo proyecto de módulo

La extensión Azure IoT Edge Tools proporciona plantillas de proyecto para todos los idiomas admitidos del módulo IoT Edge en Visual Studio. Estas plantillas tienen todos los archivos y el código que se necesita para implementar un módulo de trabajo y probar IoT Edge o le proporcionan a un punto de partida para personalizar la plantilla con su propia lógica de negocios.

1. Seleccione **Archivo** > **Nuevo** > **Proyecto...**

2. En la ventana del nuevo proyecto, busque **IoT Edge** y seleccione el proyecto **Azure IoT Edge (Windows amd64)** . Haga clic en **Next**.

   ![Creación de un nuevo proyecto de Azure IoT Edge](./media/tutorial-develop-for-windows/new-project.png)

3. En la ventana de configuración del nuevo proyecto, cambie el nombre del proyecto y de la solución por otro descriptivo como **CSharpTutorialApp**. Haga clic en **Crear** para crear el proyecto.

   ![Configuración de un nuevo proyecto de Azure IoT Edge](./media/tutorial-develop-for-windows/configure-project.png)

4. En la ventana Agregar módulo, configure el proyecto con los valores siguientes:

   | Campo | Value |
   | ----- | ----- |
   | Plantilla de Visual Studio | Seleccione **Módulo C#** . |
   | Nombre del módulo | Acepte el valor predeterminado **IotEdgeModule1**. |
   | URL del repositorio | Un repositorio de imágenes incluye el nombre del registro de contenedor y el nombre de la imagen de contenedor. La imagen de contenedor se rellena previamente con el valor del nombre del proyecto del módulo. Reemplace **localhost:5000** por el valor de **Servidor de inicio de sesión** del registro de contenedor de Azure. Puede recuperar el valor de Servidor de inicio de sesión de la página Información general del registro de contenedor en Azure Portal. <br><br> El repositorio de imágenes final es similar a \<registry name\>.azurecr.io/iotedgemodule1. |

      ![Configuración del proyecto para el dispositivo de destino, el tipo de módulo y el registro de contenedor](./media/tutorial-develop-for-windows/add-module-to-solution.png)

5. Para crear el módulo, seleccione **Agregar**.

Cuando el nuevo proyecto se cargue en la ventana de Visual Studio, dedique unos minutos a familiarizarse con los archivos que ha creado:

* Un proyecto de IoT Edge llamado **CSharpTutorialApp**.
  * La carpeta **Modules** contiene punteros a los módulos incluidos en el proyecto. En este caso, debería ser solo IotEdgeModule1.
  * El archivo **.env** oculto almacena las credenciales en el registro de contenedor. Estas credenciales se comparten con el dispositivo IoT Edge para que tenga acceso para extraer las imágenes de contenedor.
  * El archivo **deployment.template.json** es una plantilla para ayudarle a crear un manifiesto de implementación. Un *manifiesto de implementación* es un archivo que define exactamente qué módulos se desean implementar en un dispositivo, cómo se deben configurar y cómo se pueden comunicar tanto entre sí como con la nube.
    > [!TIP]
    > En la sección de credenciales del registro, la dirección se rellena automáticamente a partir de la información que proporcionó al crear la solución. Sin embargo, el nombre de usuario y la contraseña hacen referencia a variables almacenadas en el archivo .env. Esto es por motivos de seguridad, ya que Git ignota el archivo .env, pero no sucede lo mismo con la plantilla de implementación.
* Un proyecto del módulo IoT Edge denominado **IotEdgeModule1**.
  * El archivo **program.cs** contiene el código del módulo de C# predeterminado que se incluye con la plantilla de proyecto. El módulo predeterminado toma la entrada de un origen y lo pasa a la instancia de IoT Hub.
  * El archivo **module.json** contiene detalles sobre el módulo, incluido el repositorio de imágenes completo, la versión de la imagen y el Dockerfile que se va a utilizar para cada plataforma admitida.

### <a name="provide-your-registry-credentials-to-the-iot-edge-agent"></a>Especificación de la credenciales del registro para el agente de IoT Edge

El entorno de ejecución de IoT Edge necesita las credenciales del registro para extraer las imágenes de contenedor en el dispositivo IoT Edge. La extensión de IoT Edge intenta extraer de Azure información del registro del contenedor y rellenar con ella la plantilla de implementación.

1. Abra el archivo **deployment.template.json** en la solución del módulo.

1. Busque la propiedad **registryCredentials** en las propiedades $edgeAgent que desee. La dirección del registro se rellenará automáticamente con la información que proporcionó al crear el proyecto y, entonces, los campos de nombre de usuario y contraseña contendrán nombres de variable. Por ejemplo:

   ```json
   "registryCredentials": {
     "<registry name>": {
       "username": "$CONTAINER_REGISTRY_USERNAME_<registry name>",
       "password": "$CONTAINER_REGISTRY_PASSWORD_<registry name>",
       "address": "<registry name>.azurecr.io"
     }
   }
   ```

1. Abra el archivo **.env** en la solución del módulo. (Está oculto de forma predeterminada en el Explorador de soluciones, por lo que es posible que tenga que seleccionar el botón **Mostrar todos los archivos** para mostrarlo).

1. Agregue los valores de **Nombre de usuario** y **Contraseña** que ha copiado del registro de contenedor de Azure.

1. Guarde los cambios en el archivo .env.

### <a name="review-the-sample-code"></a>Revisión del código de ejemplo

La plantilla de solución que ha creado incluye código de ejemplo de un módulo IoT Edge. Dicho módulo simplemente recibe los mensajes y, después, los pasa. La funcionalidad de canalización muestra un concepto importante de IoT Edge, cómo se comunican los módulos entre sí.

Cada módulo puede tener varias colas de *entrada* y *salida* declaran en su código. El centro de IoT Edge que se ejecuta en el dispositivo enruta los mensajes de la salida de un módulo en la entrada de uno o varios módulos. El código específico para declarar las entradas y salidas varía de un lenguaje a otro, pero el concepto es el mismo en todos los módulos. Para más información acerca del enrutamiento entre módulos, consulte [Declaración de rutas](module-composition.md#declare-routes).

El código de C# de ejemplo que acompaña a la plantilla del proyecto usa la [clase ModuleClient](https://docs.microsoft.com/dotnet/api/microsoft.azure.devices.client.moduleclient?view=azure-dotnet) del SDK de IoT Hub para .NET.

1. En el archivo **program.cs**, busque el método **SetInputMessageHandlerAsync**.

2. El método [SetInputMessageHandlerAsync](https://docs.microsoft.com/dotnet/api/microsoft.azure.devices.client.moduleclient.setinputmessagehandlerasync?view=azure-dotnet) configura una cola de entrada para recibir los mensajes entrantes. Revise este método y vea cómo inicializa una cola de entrada llamada **input1**.

   ![Búsqueda del nombre de la entrada en el constructor SetInputMessageHandlserAsync](./media/tutorial-develop-for-windows/declare-input-queue.png)

3. Luego, busque el método **SendEventAsync**.

4. El método [SendEventAsync](https://docs.microsoft.com/dotnet/api/microsoft.azure.devices.client.moduleclient.sendeventasync?view=azure-dotnet) procesa los mensajes recibidos y configura una cola de salida para difundirlos. Revise dicho método y verá que inicializa una cola de salida llamada **output1**.

   ![Búsqueda del nombre de la salida en el constructor SendEventAsync](./media/tutorial-develop-for-windows/declare-output-queue.png)

5. Abra el archivo **deployment.template.json**.

6. Busque la propiedad **modules** de las propiedades deseadas de $edgeAgent.

   Debería haber dos módulos. Uno es el módulo **SimulatedTemperatureSensor**, que se incluye en todas las plantillas de forma predeterminada para proporcionar datos de temperatura simulados que se pueden usar para probar los módulos. El otro es el módulo **IotEdgeModule1** que creó como parte de este proyecto.

   Esta propiedad modules declara qué módulos se deben incluir en la implementación en el dispositivo o dispositivos.

7. Busque la propiedad **routes** de las propiedades deseadas de $edgeHub.

   Una de las funciones del módulo del centro de IoT Edge es enrutar mensajes entre todos los módulos de una implementación. Revise los valores de la propiedad routes. Una ruta, **IotEdgeModule1ToIoTHub**, usa un carácter comodín ( **\*** ) para incluir todos los mensajes que procedan de cualquiera de las colas de salida del módulo IotEdgeModule1. Dichos mensajes van a *$upstream*, que es un nombre reservado que indica IoT Hub. La otra ruta, **sensorToIotEdgeModule1**, toma los mensajes procedentes del módulo SimulatedTemperatureSensor y los enruta a la cola de entrada *input1* del módulo IotEdgeModule1.

   ![Revisar las rutas de deployment.template.json](./media/tutorial-develop-for-windows/deployment-routes.png)

## <a name="build-and-push-your-solution"></a>Compilación e inserción de la solución

Ha revisado el código del módulo y la plantilla de implementación para conocer algunos conceptos clave de la implementación. Ya está listo para que compilar la imagen de contenedor IotEdgeModule1 e insertarla en el registro de contenedor. Con la extensión de las herramientas de IoT para Visual Studio, este paso también genera el manifiesto de implementación en función de la información del archivo de plantilla y la información del módulo a partir de los archivos de la solución.

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

1. Haga clic con el botón derecho en la carpeta de proyecto **CSharpTutorialApp** y seleccione **Compilación e inserción de módulos IoT Edge**.

   ![Compilación e inserción de módulos IoT Edge](./media/tutorial-develop-for-windows/build-and-push-modules.png)

   El comando de compilación e inserción inicia tres operaciones. En primer lugar, se crea una nueva carpeta en la solución denominada **config** que contiene los archivos del manifiesto de la implementación completa, con la información de la plantilla de implementación y otros archivos de la solución. En segundo lugar, ejecuta `docker build` para generar la imagen de contenedor basándose en el Dockerfile adecuado para la arquitectura de destino. A continuación, ejecuta `docker push` para insertar el repositorio de imágenes en el registro de contenedor.

   Este proceso puede tardar varios minutos la primera vez, pero es más rápido la próxima vez que ejecute los comandos.

2. Abra el archivo **deployment.windows-amd64.json** en la carpeta de configuración recién creada. (Es posible que la carpeta de configuración no aparezca en el Explorador de soluciones de Visual Studio. Si ese es el caso, seleccione el icono **Mostrar todos los archivos** en la barra de tareas del Explorador de soluciones).

3. Busque el parámetro **image** de la sección IotEdgeModule1. Observe que la imagen contiene el repositorio de imágenes completo con el nombre, la versión y la etiqueta de arquitectura del archivo module.json.

4. Abra el archivo **module.json** de la carpeta IotEdgeModule1.

5. Cambie el número de versión de la imagen del módulo. (la versión, no la versión $schema). Por ejemplo, incremente el número de versión de la revisión a **0.0.2** como si hubiéramos hecho una pequeña corrección en el código del módulo.

   >[!TIP]
   >Las versiones de los módulos permiten el control de versiones y le permiten probar los cambios en un pequeño conjunto de dispositivos antes de implementar actualizaciones en la producción. Si no incrementa la versión del módulo antes de la compilación e inserción, entonces sobrescribirá el repositorio en el registro de contenedor.

6. Guarde los cambios en el archivo module.json.

7. Vuelva a hacer clic con el botón derecho en el proyecto **CSharpTutorialApp**, y vuelva a seleccionar **Build and Push IoT Edge Modules** (Compilación e inserción de módulos IoT Edge).

8. Abra el archivo **deployment.windows-amd64.json** de nuevo. Observe que no se creó un nuevo archivo cuando ejecutó el comando de compilación e inserción de nuevo. Más bien, el mismo archivo se actualizó para reflejar los cambios. La imagen de IotEdgeModule1 ahora apunta a la versión 0.0.2 del contenedor. Este cambio en el manifiesto de implementación es cómo se puede saber el dispositivo IoT Edge que hay una nueva versión de un módulo de extracción.

9. Para comprobar mejor lo que hizo el comando de compilación e inserción, vaya a [Azure Portal](https://portal.azure.com) y navegue al registro de contenedor.

10. En el registro de contenedor, seleccione **Repositorios** y después **iotedgemodule1**. Compruebe que ambas versiones de la imagen se hayan insertado en el registro.

    ![Visualización de ambas versiones de la imagen en el registro de contenedor](./media/tutorial-develop-for-windows/view-repository-versions.png)

### <a name="troubleshoot"></a>Solución de problemas

Si encuentra errores al compilar e insertar la imagen del módulo, a menudo tiene que ver con la configuración de Docker en la máquina de desarrollo. Utilice las siguientes comprobaciones para revisar la configuración:

* ¿Ha ejecutado el comando `docker login` mediante las credenciales que copió del registro de contenedor? Estas credenciales son diferentes a las que se utilizan para iniciar sesión en Azure.
* ¿Es el repositorio de contenedor correcto? ¿Tiene el nombre correcto del registro de contenedor y el nombre correcto del módulo? Abra el archivo **module.json** de la carpeta IotEdgeModule1 que se va a comprobar. El valor del repositorio debería ser similar a **\<registry name\>.azurecr.io/iotedgemodule1**.
* Si utilizó un nombre diferente a **IotEdgeModule1** para el módulo, ¿es coherente ese nombre en toda la solución?
* ¿La máquina funciona con el mismo tipo de contenedores que está creando? Este tutorial es para dispositivos Windows IoT Edge, por lo que los archivos de Visual Studio deben tener la extensión **windows-amd64** y Docker Desktop debe ejecutar contenedores Windows.

## <a name="deploy-modules-to-device"></a>Implementación de módulos en el dispositivo

Ha verificado que las imágenes de contenedor creadas se almacenan en el registro de contenedor, por lo que es el momento de implementarlas en un dispositivo. Asegúrese de que el dispositivo IoT Edge está en funcionamiento.

1. Abra Cloud Explorer en Visual Studio y expanda los detalles del centro de IoT.

2. Seleccione el nombre del dispositivo en el que desea realizar la implementación. En la lista **Acciones**, seleccione **Crear implementación**.

   ![Crear una implementación para un dispositivo individual](./media/tutorial-develop-for-windows/create-deployment.png)

3. En el Explorador de archivos, vaya a la carpeta de configuración del proyecto y seleccione el archivo **deployment.windows amd64.json**. Este archivo suele encontrarse en `C:\Users\<username>\source\repos\CSharpTutorialApp\CSharpTutorialApp\config\deployment.windows-amd64.json`

   No utilice el archivo deployment.template.json, que no contiene todos los valores de imagen del módulo.

4. Expanda los detalles para el dispositivo IoT Edge en Cloud Explorer para ver los módulos del dispositivo.

5. Use el botón **Actualizar** para actualizar el estado del dispositivo para ver que los módulos SimulatedTemperatureSensor y IotEdgeModule1 se implementan en el dispositivo.

   ![Visualización de los módulos que se ejecutan en el dispositivo IoT Edge](./media/tutorial-develop-for-windows/view-running-modules.png)

## <a name="view-messages-from-device"></a>Visualización de los mensajes desde el dispositivo

El código IotEdgeModule1 recibe mensajes mediante la cola de entrada y los pasa por la cola de salida. El manifiesto de implementación declaraba las rutas que pasaban los mensajes de SimulatedTemperatureSensor a IotEdgeModule1, y luego enviaban los mensajes de IotEdgeModule1 a la instancia de IoT Hub. Las herramientas Azure IoT Edge Tools para Visual Studio permiten ver los mensajes que llegan a IoT Hub desde dispositivos individuales.

1. En el explorador en la nube de Visual Studio, seleccione el nombre del dispositivo IoT Edge en el que se ha realizado la implementación.

2. En el menú **Acciones**, seleccione **Start Monitoring Built-in Event Endpoint** (Iniciar supervisión del punto de conexión del evento integrado).

3. Consulte la sección **Salida** de Visual Studio para ver los mensajes que llegan al centro de IoT.

   Ambos módulos pueden tardar unos minutos en iniciarse. El entorno de ejecución de Azure IoT Edge necesita recibir su nuevo manifiesto de implementación, extraer las imágenes de los módulos del entorno de ejecución del contenedor y, después, iniciar cada nuevo módulo.

   ![Visualización de los mensajes entrantes del dispositivo a la nube](./media/tutorial-develop-for-windows/view-d2c-messages.png)

## <a name="view-changes-on-device"></a>Visualización de los cambios en el dispositivo

Si desea ver lo que está ocurriendo en su propio dispositivo, utilice los comandos de esta sección para inspeccionar el entorno de ejecución de Azure IoT Edge y los módulos que se ejecutan en el dispositivo.

Los comandos de esta sección son para el dispositivo IoT Edge, no en la máquina de desarrollo. Si usa una máquina virtual para el dispositivo IoT Edge, conéctese a ella ahora. En Azure, vaya a la página de información general de la máquina virtual y seleccione **Conectar** para acceder a la conexión del escritorio remoto. En el dispositivo, abra una ventana de comando o PowerShell para ejecutar los comandos `iotedge`.

* Vea todos los módulos implementados en el dispositivo y compruebe su estado:

   ```cmd
   iotedge list
   ```

   Debería ver cuatro módulos: los dos módulos personalizados del entorno de ejecución de IoT Edge, SimulatedTemperatureSensor e IotEdgeModule1. Los cuatro deben aparecer como en ejecución.

* Inspeccione los registros para un módulo específico:

   ```cmd
   iotedge logs <module name>
   ```

   Los módulos IoT Edge distinguen mayúsculas de minúsculas.

   Los registros de SimulatedTemperatureSensor e IotEdgeModule1 deben mostrar los mensajes que están procesando. El módulo edgeAgent es responsable de iniciar los otros módulos, por lo que sus registros tendrá información sobre cómo implementar el manifiesto de implementación. Si algún módulo no aparece en la lista o no se está ejecutando, es probable que los registros de edgeAgent contengan los errores. El módulo edgeHub es responsable de las comunicaciones entre los módulos e IoT Hub. Si los módulos están en funcionamiento, pero los mensajes no llegan al centro de IoT, los registros de edgeHub probablemente contendrán los errores.

## <a name="clean-up-resources"></a>Limpieza de recursos

Si prevé seguir con el siguiente artículo recomendado, puede mantener los recursos y las configuraciones que ya ha creado y volverlos a utilizar. También puede seguir usando el mismo dispositivo de IoT Edge como dispositivo de prueba.

En caso contrario, para evitar gastos, puede eliminar las configuraciones locales y los recursos de Azure que creó en este artículo.

[!INCLUDE [iot-edge-clean-up-cloud-resources](../../includes/iot-edge-clean-up-cloud-resources.md)]

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha configurado Visual Studio 2019 en la máquina de desarrollo y ha implementado el primer módulo IoT Edge desde él. Ahora que conoce los conceptos básicos, intente agregar funcionalidad a un módulo para que pueda analizar los datos que pasan por él. Elija el lenguaje que prefiera:

> [!div class="nextstepaction"]
> [C](tutorial-c-module-windows.md)
> [C#](tutorial-csharp-module-windows.md)
