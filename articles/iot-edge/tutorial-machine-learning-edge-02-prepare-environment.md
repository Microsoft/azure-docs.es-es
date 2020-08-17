---
title: 'Tutorial: Configuración del entorno: Machine Learning en Azure IoT Edge'
description: 'Tutorial: Prepare el entorno para desarrollar e implementar módulos de aprendizaje automático en el perímetro.'
author: kgremban
manager: philmea
ms.author: kgremban
ms.date: 3/12/2020
ms.topic: tutorial
ms.service: iot-edge
services: iot-edge
ms.openlocfilehash: 22628af5b3e62edfc96699ad051b6a00cf9c1429
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87078937"
---
# <a name="tutorial-set-up-an-environment-for-machine-learning-on-iot-edge"></a>Tutorial: Configuración de un entorno de aprendizaje automático en IoT Edge

> [!NOTE]
> Este artículo forma parte de un tutorial sobre el uso de Azure Machine Learning en IoT Edge. Si ha llegado directamente a este artículo, le recomendamos que comience con el [primer el artículo](tutorial-machine-learning-edge-01-intro.md) de la serie para obtener los mejores resultados.

Este artículo le ayuda a preparar un entorno para el desarrollo y la implementación. En primer lugar, configure una máquina de desarrollo con todas las herramientas que necesita. A continuación, cree los recursos en la nube necesarios en Azure.

## <a name="set-up-the-development-vm"></a>Configuración de la máquina virtual de desarrollo

Normalmente, este paso está a cargo de un desarrollador de nube. Algunos de los ejemplos de software también pueden resultar útiles para un científico de datos.

Hemos creado un script de PowerShell que crea una máquina virtual de Azure con muchos de los requisitos previos ya configurados. La máquina virtual que se cree debe ser capaz de hacer frente a la [virtualización anidada](https://docs.microsoft.com/azure/virtual-machines/windows/nested-virtualization), motivo por el que elegimos el tamaño de máquina [Standard_D8s_v3](../virtual-machines/dv3-dsv3-series.md).

La máquina virtual de desarrollo se configurará con:

* Windows 10
* [Chocolatey](https://chocolatey.org/)
* [Docker Desktop para Windows](https://www.docker.com/products/docker-desktop)
* [Git para Windows](https://gitforwindows.org/)
* [Administrador de credenciales de Git para Windows](https://github.com/Microsoft/Git-Credential-Manager-for-Windows)
* [SDK de .NET Core](https://dotnet.microsoft.com/)
* [Python 3](https://www.python.org/)
* [Visual Studio Code](https://code.visualstudio.com/)
* [Azure PowerShell](https://docs.microsoft.com/powershell/azure/?view=azps-1.1.0)
* [Extensiones de VS Code](https://marketplace.visualstudio.com/search?target=VSCode)
  * [Herramientas de Azure IoT](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-tools)
  * [Python](https://marketplace.visualstudio.com/items?itemName=ms-python.python)
  * [C#](https://marketplace.visualstudio.com/items?itemName=ms-dotnettools.csharp)
  * [Docker](https://marketplace.visualstudio.com/items?itemName=PeterJausovec.vscode-docker)
  * [PowerShell](https://marketplace.visualstudio.com/items?itemName=ms-vscode.PowerShell)

La máquina virtual de desarrollador no es realmente necesaria; todas las herramientas de desarrollo se pueden ejecutar en un equipo local. Sin embargo, es muy recomendable usar la máquina virtual para garantizar condiciones uniformes.

La máquina virtual tarda unos 30 minutos en crearse y configurarse.

1. Clone o descargue el repositorio de ejemplo de [Machine Learning and IoT Edge](https://github.com/Azure-Samples/IoTEdgeAndMlSample) en su equipo local.

1. Abra PowerShell como administrador y vaya al directorio **\IoTEdgeAndMlSample\DevVM**, que se encuentra en el directorio raíz donde descargó el código. Haremos referencia al directorio raíz para el origen de datos como `srcdir`.

    ```powershell
    cd c:\srcdir\IoTEdgeAndMlSample\DevVM
    ```

   El directorio DevVM contiene los archivos necesarios para crear una máquina virtual de Azure adecuada para completar este tutorial.

1. Ejecute el siguiente comando para permitir la ejecución de los scripts. Elija **Sí a todo** cuando se le solicite.

    ```powershell
    Set-ExecutionPolicy Bypass -Scope Process
    ```

1. Ejecute Create-AzureDevVM.ps1.

    ```powershell
    .\Create-AzureDevVm.ps1
    ```

    Cuando se le solicite, proporcione la siguiente información:

    * **Identificador de suscripción de Azure**: su identificador de suscripción, que se puede encontrar en [Suscripciones de Azure](https://ms.portal.azure.com/#blade/Microsoft_Azure_Billing/SubscriptionsBlade) en Azure Portal.
    * **Nombre del grupo de recursos**: nombre de un grupo de recursos nuevo o existente en Azure.
    * **Ubicación**: seleccione la ubicación de Azure donde se creará la máquina virtual. Por ejemplo, "Oeste de EE. UU. 2" o "Norte de Europa". Para obtener más información, consulte [Ubicaciones de Azure](https://azure.microsoft.com/global-infrastructure/locations/).
    * **Nombre de usuario**: Especifique un nombre fácil de recordar para la cuenta de administrador de la máquina virtual.
    * **Contraseña**: Establezca la contraseña de la cuenta de administrador de la máquina virtual.

   El script se ejecuta durante varios minutos mientras realiza los pasos siguientes:

    1. Instala el [módulo Az de Azure PowerShell](https://docs.microsoft.com/powershell/azure/new-azureps-module-az?view=azps-1.1.0).
    1. Le pide que inicie sesión en Azure.
    1. Confirma la información necesaria para crear la máquina virtual. Presione **y** o **ENTRAR** para continuar.
    1. Crea el grupo de recursos, en caso de que no exista.
    1. Implementa la máquina virtual.
    1. Habilita Hyper-V en la máquina virtual.
    1. Instala el software necesario para el desarrollo y clonación del repositorio de ejemplo.
    1. Reinicia la máquina virtual.
    1. Crea un archivo de RDP en el escritorio para conectarse a la máquina virtual.

   Si se le pide el nombre de la máquina virtual para reiniciarla, puede copiarlo de la salida del script. La salida también muestra la ruta de acceso al archivo de RDP para conectarse a la máquina virtual.

### <a name="set-auto-shutdown-schedule"></a>Establecimiento de la programación de apagado automático

Para ayudarle a reducir costos, la máquina virtual de desarrollo se crea con una programación de apagado automático que se establece en las 19:00 PST. Quizá necesite actualizar este valor según su ubicación y programación. Para actualizar la programación de apagado:

1. En Azure Portal, vaya a la máquina virtual que creó el script.

1. En el menú del panel izquierdo, en **Operaciones**, seleccione **Apagado automático**.

1. Ajuste los valores de **Apagado programado** y **Zona horaria** como desee y seleccione **Guardar**.

## <a name="connect-to-the-development-vm"></a>Conexión a la máquina virtual de desarrollo

Ahora que hemos creado una máquina virtual, es necesario finalizar la instalación del software necesario para completar el tutorial.

1. Haga doble clic en el archivo de RDP que el script ha creado en el escritorio.

1. Aparecerá un cuadro de diálogo que indica que el publicador de la conexión remota es desconocido. No pasa nada, así que seleccione **Conectar**.

1. Escriba la contraseña de administrador que especificó para crear la máquina virtual y haga clic en **Aceptar**.

1. Se le pedirá que acepte el certificado para la máquina virtual. Seleccione **Sí**.

## <a name="install-visual-studio-code-extensions"></a>Instalación de extensiones de Visual Studio Code

Ahora que se ha conectado a la máquina de desarrollo, agregue algunas extensiones útiles a Visual Studio Code para facilitar la experiencia de desarrollo.

1. Conéctese a la máquina virtual de desarrollo, abra una ventana de PowerShell y vaya al directorio **C:\source\IoTEdgeAndMlSample\DevVM**. Este directorio lo ha creado el script que creó la máquina virtual.

    ```powershell
    cd C:\source\IoTEdgeAndMlSample\DevVM
    ```

1. Ejecute el siguiente comando para permitir la ejecución de los scripts. Elija **Sí a todo** cuando se le solicite.

    ```powershell
    Set-ExecutionPolicy Bypass -Scope Process
    ```

1. Ejecute el script de extensiones de Visual Studio Code.

    ```powershell
    .\Enable-CodeExtensions.ps1
    ```

1. El script se ejecutará durante unos minutos para instalar las siguientes extensiones de VS Code:

    * Herramientas de Azure IoT
    * Python
    * C#
    * Docker
    * PowerShell

## <a name="set-up-iot-hub-and-storage"></a>Configuración de IoT Hub y Storage

Normalmente, estos pasos están a cargo de un desarrollador de nube.

Azure IoT Hub es el centro de todas las aplicaciones de IoT, ya que controla la comunicación segura entre los dispositivos de IoT y la nube. Es el punto de coordinación principal para el funcionamiento de la solución de aprendizaje automático de IoT Edge.

* IoT Hub utiliza rutas para dirigir los datos entrantes desde los dispositivos IoT hacia otros servicios de bajada. Sacaremos provecho de las rutas de IoT Hub para enviar los datos de los dispositivos a Azure Storage. En Azure Storage, Azure Machine Learning consume estos datos para entrenar a nuestro clasificador de vida útil restante (RUL).

* Más adelante en el tutorial, usaremos IoT Hub para configurar y administrar el dispositivo de Azure IoT Edge.

En esta sección, use un script para crear un centro de IoT de Azure y una cuenta de Azure Storage. Luego, en Azure Portal, configure una ruta que reenvíe los datos que recibe el centro a un contenedor de Azure Storage. Estos pasos tardan aproximadamente 10 minutos en completarse.

1. Conéctese a la máquina virtual de desarrollo, abra una ventana de PowerShell y vaya al directorio **IoTHub**.

    ```powershell
    cd C:\source\IoTEdgeAndMlSample\IoTHub
    ```

1. Ejecute el script de creación. Use los mismos valores de Id. de suscripción, ubicación y grupo de recursos que usó al crear la máquina virtual de desarrollo.

    ```powershell
    .\New-HubAndStorage.ps1 -SubscriptionId <subscription id> -Location <location> -ResourceGroupName <resource group>
    ```

    * Se le pedirá que inicie sesión en Azure.
    * El script confirma la información para crear su cuenta de IoT Hub y Storage. Presione **y** o **ENTRAR** para continuar.

El script tarda dos minutos en ejecutarse. Cuando haya finalizado, el script genera el nombre de la cuenta de almacenamiento y del centro de IoT.

## <a name="review-route-to-storage-in-iot-hub"></a>Revisión de la ruta al almacenamiento en IoT Hub

Como parte de la creación del centro de IoT, el script que se ejecutó en la sección anterior también crea un punto de conexión personalizado y una ruta. Las rutas de IoT Hub constan de una expresión de consulta y un punto de conexión. Si un mensaje coincide con la expresión, los datos se envían a lo largo de la ruta hasta el punto de conexión asociado. Los puntos de conexión pueden ser Event Hubs, colas de Service Bus y temas. En este caso, el punto de conexión es un contenedor de blobs en una cuenta de almacenamiento. Vamos a usar Azure Portal para revisar la ruta que ha creado el script.

1. Abra [Azure Portal](https://portal.azure.com) y vaya al grupo de recursos que utiliza para este tutorial.

1. En la lista de recursos, seleccione el centro de IoT que el script ha creado. Su nombre terminará en caracteres aleatorios, como `IotEdgeAndMlHub-jrujej6de6i7w`.

1. En el menú del panel izquierdo, en **Mensajes**, seleccione **Enrutamiento de mensajes**.

1. En la página **Enrutamiento de mensajes**, seleccione la pestaña **Puntos de conexión personalizados**.

1. Expanda la sección **Storage**:

   ![Compruebe si turbofanDeviceStorage se encuentra en la lista de puntos de conexión personalizados](media/tutorial-machine-learning-edge-02-prepare-environment/custom-endpoints.png)

   Vemos que **turbofanDeviceStorage** se encuentra en dicha lista. Tenga en cuenta las siguientes características de este punto de conexión:

   * Apunta al contenedor de Blob Storage que creó, llamado `devicedata`, tal como se indica en **Nombre de contenedor**.
   * Su **formato de nombre de archivo** tiene la partición como último elemento del nombre. Este formato parece más práctico para las operaciones de archivo que se realizarán con Azure Notebooks más adelante durante el tutorial.
   * El valor de **Estado** debe ser Correcto.

1. Seleccione la pestaña **Rutas**.

1. Seleccione la ruta denominada **turbofanDeviceDataToStorage**.

1. En la página **Detalles de ruta**, observe que el punto de conexión de la ruta es **turbofanDeviceStorage**.

   ![Revisar los detalles de la ruta de turbofanDeviceDataToStorage](media/tutorial-machine-learning-edge-02-prepare-environment/route-details.png)

1. Examine la opción seleccionada **Consulta de enrutamiento**, que es **true**. Este valor significa que todos los mensajes de telemetría de dispositivo coincidirán con esta ruta y, por tanto, todos los mensajes se enviarán al punto de conexión **turbofanDeviceStorage**.

1. Dado que no se ha realizado ninguna modificación, simplemente cierre esta página.

## <a name="next-steps"></a>Pasos siguientes

En este artículo, creamos una instancia de IoT Hub y configuramos una ruta a una cuenta de Azure Storage. A continuación, se enviarán datos de un conjunto de dispositivos simulados a la cuenta de almacenamiento a través de la instancia de IoT Hub. Más adelante en el tutorial, tras configurar al dispositivo y módulos IoT Edge, volveremos a revisar las rutas y profundizaremos un poco más en la consulta de enrutamiento.

Para obtener más información acerca de los pasos descritos en esta parte del tutorial de Machine Learning en IoT Edge, consulte:

* [Aspectos básicos de Azure IoT](https://docs.microsoft.com/azure/iot-fundamentals/)
* [Configuración del enrutamiento de mensajes con IoT Hub](../iot-hub/tutorial-routing.md)
* [Creación de una instancia de IoT Hub mediante Azure Portal](../iot-hub/iot-hub-create-through-portal.md)

Continúe con el siguiente artículo para crear un dispositivo simulado para supervisar.

> [!div class="nextstepaction"]
> [Generación de datos de dispositivo](tutorial-machine-learning-edge-03-generate-data.md)
