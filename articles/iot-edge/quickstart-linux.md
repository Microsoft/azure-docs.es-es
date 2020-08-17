---
title: 'Guía de inicio rápido: Creación de un dispositivo Azure IoT Edge en Linux | Microsoft Docs'
description: En esta guía de inicio rápido, aprenderá a crear un dispositivo IoT Edge y a implementar luego el código creado previamente de manera remota desde Azure Portal.
author: kgremban
manager: philmea
ms.author: kgremban
ms.date: 06/30/2020
ms.topic: quickstart
ms.service: iot-edge
services: iot-edge
ms.custom: mvc
ms.openlocfilehash: 36bebe829ccf81ef5b1832b90b2f73d15d5499af
ms.sourcegitcommit: 5b8fb60a5ded05c5b7281094d18cf8ae15cb1d55
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87384810"
---
# <a name="quickstart-deploy-your-first-iot-edge-module-to-a-virtual-linux-device"></a>Inicio rápido: Implementación del primer módulo de IoT Edge en un dispositivo virtual Linux

Pruebe Azure IoT Edge en este inicio rápido mediante la implementación de código en contenedor en un dispositivo de IoT Edge virtual de Linux. IoT Edge permite administrar de forma remota el código de los dispositivos para que pueda enviar más cargas de trabajo al perímetro. En este inicio rápido, se recomienda usar una máquina virtual de Azure para el dispositivo IoT Edge, lo que le permite crear rápidamente una máquina de pruebas con el servicio IoT Edge instalado y, luego, eliminarla cuando haya terminado.

En esta guía de inicio rápido, aprenderá a hacer lo siguiente:
> [!div class="checklist"]
>
> * Cree un centro de IoT Hub.
> * Registre un dispositivo IoT Edge en su instancia de IoT Hub.
> * Instale e inicie el entorno de ejecución de IoT Edge en el dispositivo virtual.
> * Implemente un módulo de manera remota en un dispositivo IoT Edge.

![Diagrama: Inicio rápido de la arquitectura para el dispositivo y la nube](./media/quickstart-linux/install-edge-full.png)

Este inicio rápido le guía en el proceso de creación de una máquina virtual Linux configurada para ser un dispositivo IoT Edge. Después, podrá implementar un módulo de Azure Portal en el dispositivo. El módulo que se usa en este inicio rápido es un sensor simulado que genera datos de temperatura, humedad y presión. Los restantes tutoriales de Azure IoT Edge se basan en el trabajo que se realiza aquí mediante la implementación de módulos adicionales que analizan los datos simulados para obtener información empresarial.

Si no tiene una suscripción activa a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free) antes de comenzar.

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

Usará la CLI de Azure para completar muchos de los pasos de esta guía de inicio rápido; además, Azure IoT cuenta con una extensión para habilitar funcionalidad adicional.

Agregue la extensión de Azure IoT a la instancia de Cloud Shell.

   ```azurecli-interactive
   az extension add --name azure-iot
   ```

[!INCLUDE [iot-hub-cli-version-info](../../includes/iot-hub-cli-version-info.md)]

## <a name="prerequisites"></a>Requisitos previos

Recursos en la nube:

* Un grupo de recursos para administrar todos los recursos que se van a usar en esta guía de inicio rápido. Usamos el nombre del grupo de recursos de ejemplo **IoTEdgeResources** tanto en este inicio rápido como en los siguientes tutoriales.

   ```azurecli-interactive
   az group create --name IoTEdgeResources --location westus2
   ```

## <a name="create-an-iot-hub"></a>Crear un centro de IoT

Para empezar el inicio rápido, cree un centro de IoT con la CLI de Azure.

![Diagrama: Creación de un centro de IoT en la nube](./media/quickstart-linux/create-iot-hub.png)

El nivel gratuito de IoT Hub funciona para esta guía de inicio rápido. Si ha usado IoT Hub en el pasado y ya tiene un centro creado, puede usarlo.

El código siguiente crea un centro **F1** gratis en el grupo de recursos **IoTEdgeResources**. Reemplace `{hub_name}` por un nombre único para su centro de IoT. El centro de IoT puede tardar algunos minutos en crearse.

   ```azurecli-interactive
   az iot hub create --resource-group IoTEdgeResources --name {hub_name} --sku F1 --partition-count 2
   ```

   Si se produce un error porque ya hay un centro gratis en la suscripción, cambie la SKU a **S1**. Cada suscripción no puede tener más de un centro de IoT gratuito. Si recibe un error que le indica que el nombre de IoT Hub no está disponible, significa que alguien más ya tiene un centro con ese nombre. Pruebe con uno nuevo.

## <a name="register-an-iot-edge-device"></a>Registro de un dispositivo de IoT Edge

Registre un dispositivo de IoT Edge con la instancia de IoT Hub recién creada.

![Diagrama: Registro de un dispositivo con una identidad de IoT Hub](./media/quickstart-linux/register-device.png)

Cree una identidad para el dispositivo de IoT Edge, con el fin de que pueda comunicarse con su centro de IoT. La identidad del dispositivo reside en la nube, y se usa una cadena de conexión de dispositivo única para asociar un dispositivo físico a una identidad de dispositivo.

Dado que los dispositivos de IoT Edge se comportan y se pueden administrar de manera diferente a los dispositivos de IoT típicos, declare que esta identidad es para un dispositivo de IoT Edge con la marca `--edge-enabled`.

1. En Azure Cloud Shell, escriba el comando siguiente para crear un dispositivo llamado **myEdgeDevice** en el centro.

   ```azurecli-interactive
   az iot hub device-identity create --device-id myEdgeDevice --edge-enabled --hub-name {hub_name}
   ```

   Si recibe un error acerca de las claves de directiva de iothubowner, asegúrese de que Cloud Shell ejecuta la versión más reciente de la extensión azure-iot.

2. Vea la cadena de conexión del dispositivo, que vincula el dispositivo físico con su identidad en IoT Hub. Contiene el nombre del centro de IoT, el nombre del dispositivo y, después, una clave compartida que autentica las conexiones entre los dos. Volveremos a hacer referencia a esta cadena de conexión en la sección siguiente, en la que configurará un dispositivo IoT Edge.

   ```azurecli-interactive
   az iot hub device-identity show-connection-string --device-id myEdgeDevice --hub-name {hub_name}
   ```

   ![Visualización de la cadena de conexión desde la salida de la CLI](./media/quickstart/retrieve-connection-string.png)

## <a name="configure-your-iot-edge-device"></a>Configuración de un dispositivo de IoT Edge

Cree una máquina virtual con el runtime de Azure IoT Edge en ella.

![Diagrama: Inicio del runtime en el dispositivo](./media/quickstart-linux/start-runtime.png)

El runtime de IoT Edge se implementa en todos los dispositivos de IoT Edge. Tiene tres componentes. El *demonio de seguridad de IoT Edge* se inicia cada vez que se inicia un dispositivo IoT Edge y arranca el dispositivo mediante el inicio del agente de este. El *agente de IoT Edge* facilita la implementación y supervisión de los módulos en el dispositivo IoT Edge, incluido el centro de IoT Edge. El *centro de IoT Edge* administra las comunicaciones entre los módulos del dispositivo de IoT Edge y entre el dispositivo y la instancia de IoT Hub.

Durante la configuración del entorno en tiempo de ejecución, tendrá que proporcionar una cadena de conexión del dispositivo. Esta es la cadena que recuperó de la CLI de Azure. Esta cadena asocia el dispositivo físico con la identidad del dispositivo IoT Edge en Azure.

### <a name="deploy-the-iot-edge-device"></a>Implementación de un dispositivo IoT Edge

En esta sección se usa una plantilla de Azure Resource Manager para crear una máquina virtual e instalar en ella el runtime de IoT Edge. Si desea usar su propio dispositivo Linux, puede seguir los pasos de instalación que se encuentran en el artículo sobre la [instalación del runtime de Azure IoT Edge en Linux](how-to-install-iot-edge-linux.md) y, después, volver a este inicio rápido.

Use el siguiente comando de la CLI para crear un dispositivo IoT Edge basado en la plantilla pregenerada [iotedge-vm-deploy](https://github.com/Azure/iotedge-vm-deploy).

* Para los usuarios de Bash o Cloud Shell, copie el siguiente comando en un editor de texto, reemplace el texto del marcador de posición por su información y, después, cópielo en la ventana de Bash o de Cloud Shell:

   ```azurecli-interactive
   az deployment group create --resource-group IoTEdgeResources --template-uri "https://aka.ms/iotedge-vm-deploy" --parameters dnsLabelPrefix='my-edge-vm' --parameters adminUsername='azureUser' --parameters deviceConnectionString=$(az iot hub device-identity show-connection-string --device-id myEdgeDevice --hub-name <REPLACE_WITH_HUB_NAME> -o tsv) --parameters authenticationType='password' --parameters adminPasswordOrKey="<REPLACE_WITH_PASSWORD>"
   ```

* Para los usuarios de PowerShell, copie el siguiente comando en la ventana de PowerShell y, después, reemplace el texto del marcador de posición por su propia información:

   ```azurecli
   az deployment group create `
   --resource-group IoTEdgeResources `
   --template-uri "https://aka.ms/iotedge-vm-deploy" `
   --parameters dnsLabelPrefix='my-edge-vm1' `
   --parameters adminUsername='azureUser' `
   --parameters deviceConnectionString=$(az iot hub device-identity show-connection-string --device-id myEdgeDevice --hub-name <REPLACE_WITH_HUB_NAME> -o tsv) `
   --parameters authenticationType='password' `
   --parameters adminPasswordOrKey="<REPLACE_WITH_PASSWORD>"
   ```

La plantilla usa los siguientes parámetros:

| Parámetro | Descripción |
| --------- | ----------- |
| **resource-group** | El grupo de recursos en el que se creará el recurso. Use el valor predeterminado **IoTEdgeResources** que hemos usado en este artículo, o bien especifique el nombre de un grupo de recursos existente en la suscripción. |
| **template-uri** | Puntero a la plantilla de Resource Manager que se usa. |
| **dnsLabelPrefix** | Cadena que se usará para crear el nombre de host de la máquina virtual. Use la cadena **my-edge-vm** del ejemplo o especifique una nueva. |
| **adminUsername** | Nombre de usuario de la cuenta de administrador de la máquina virtual. Use el nombre de usuario **azureUser** del ejemplo o especifique uno nuevo. |
| **deviceConnectionString** | Cadena de conexión de la identidad del dispositivo en IoT Hub, que se usa para configurar el entorno de ejecución de Azure IoT Edge en la máquina virtual. El comando de la CLI que hay dentro de este parámetro toma la cadena de conexión automáticamente. Sustituya el texto del marcador de posición por su nombre de IoT Hub. |
| **authenticationType** | El método de autenticación de la cuenta de administrador. En este inicio rápido se usa la autenticación mediante **contraseña**, pero también puede establecer este parámetro en **sshPublicKey**. |
| **adminPasswordOrKey** | La contraseña o el valor de la clave SSH de la cuenta de administrador. Sustituya el texto del marcador de posición por una contraseña segura. La contraseña debe tener una longitud mínima de doce caracteres y, al menos, tres de los siguientes: caracteres en minúsculas, caracteres en mayúsculas, dígitos y caracteres especiales. |

Una vez completada la implementación, debe recibir la salida con formato JSON en la CLI que contiene la información de SSH para conectarse a la máquina virtual. Copie el valor de la entrada de **public SSH** de la sección **outputs**:

   ![Recuperación del valor de ssh de la salida](./media/quickstart-linux/outputs-public-ssh.png)

### <a name="view-the-iot-edge-runtime-status"></a>Visualización del estado del entorno de ejecución de Azure IoT Edge

El resto de los comandos de este inicio rápido tienen lugar en el propio dispositivo de IoT Edge, con el fin de que pueda ver lo que sucede en el dispositivo. Si usa una máquina virtual, conéctese a ella ahora con el nombre de usuario de administrador que configuró y el nombre DNS que el comando de implementación ha generado. También puede encontrar el nombre DNS en la página de información general de la máquina virtual en Azure Portal. Use el siguiente comando para conectarse a la máquina virtual. Reemplace `{admin username}` y `{DNS name}` con sus propios valores.

   ```console
   ssh {admin username}@{DNS name}
   ```

Una vez que se haya conectado a su máquina virtual, compruebe que el runtime se ha instalado y configurado correctamente en el dispositivo de IoT Edge.

1. Compruebe que el demonio de seguridad de IoT Edge se ejecuta como un servicio del sistema.

   ```bash
   sudo systemctl status iotedge
   ```

   ![Comprobación de cómo el demonio de IoT Edge se ejecuta como un servicio del sistema](./media/quickstart-linux/iotedged-running.png)

   >[!TIP]
   >Necesita privilegios elevados para ejecutar comandos `iotedge`. Cuando cierre la sesión en su máquina y la inicie de nuevo por primera vez después de instalar el entorno de ejecución de IoT Edge, sus permisos se actualizarán automáticamente. Hasta entonces, use `sudo` delante de los comandos.

2. Si necesita solucionar problemas del servicio, recupere los registros del servicio.

   ```bash
   journalctl -u iotedge
   ```

3. Vea todos los módulos que se ejecutan en el dispositivo IoT Edge. Como el servicio se acaba de iniciar por primera vez, solo verá la ejecución del módulo **edgeAgent**. El módulo edgeAgent se ejecuta de forma predeterminada y le ayuda a instalar e iniciar todos los módulos adicionales que implemente en el dispositivo.

   ```bash
   sudo iotedge list
   ```

   ![Visualización de un módulo en el dispositivo](./media/quickstart-linux/iotedge-list-1.png)

El dispositivo de IoT Edge está ya configurado. Está preparado para ejecutar módulos implementados en la nube.

## <a name="deploy-a-module"></a>Implementación de un módulo

Administre el dispositivo Azure IoT Edge desde la nube para implementar un módulo que enviará datos de telemetría a IoT Hub.

![Diagrama: implementación del módulo desde la nube al dispositivo](./media/quickstart-linux/deploy-module.png)

[!INCLUDE [iot-edge-deploy-module](../../includes/iot-edge-deploy-module.md)]

## <a name="view-generated-data"></a>Visualización de datos generados

En esta guía de inicio rápido, ha creado un nuevo dispositivo de IoT Edge y ha instalado el runtime de IoT Edge en él. Luego, ha usado Azure Portal para implementar un módulo de IoT Edge para que se ejecute en el dispositivo sin tener que realizar cambios en el propio dispositivo.

En este caso, el módulo que ha insertado genera los datos del entorno de ejemplo que puede usar posteriormente para las pruebas. El sensor simulado está supervisando una máquina y el entorno alrededor de esta. Por ejemplo, este sensor podría estar en una sala de servidores, en una fábrica o en un aerogenerador. El mensaje incluye la temperatura y la humedad ambiental, la temperatura y la presión de la máquina, y una marca de tiempo. Los tutoriales de IoT Edge usan los datos creados por este módulo como datos de prueba con fines de análisis.

Abra de nuevo el símbolo del sistema en su dispositivo IoT Edge, o utilice la conexión SSH de la CLI de Azure. Confirme que el módulo implementado desde la nube se está ejecutando en el dispositivo IoT Edge:

   ```bash
   sudo iotedge list
   ```

   ![Ver tres módulos en el dispositivo](./media/quickstart-linux/iotedge-list-2.png)

Vea los mensajes que se envían desde el módulo del sensor de temperatura:

   ```bash
   sudo iotedge logs SimulatedTemperatureSensor -f
   ```

   >[!TIP]
   >Los comandos de IoT Edge distinguen mayúsculas de minúsculas en los nombres de los módulos.

   ![Ver los datos desde el módulo](./media/quickstart-linux/iotedge-logs.png)

También puede ver los mensajes que llegan a su centro de IoT mediante la [extensión de Azure IoT Hub para Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-toolkit).

## <a name="clean-up-resources"></a>Limpieza de recursos

Si desea continuar con los tutoriales de IoT Edge, puede usar el dispositivo que ha registrado y configurado en esta guía de inicio rápido. En caso contrario, puede eliminar los recursos de Azure que creó para evitar gastos.

Si ha creado una máquina virtual y un centro de IoT en un nuevo grupo de recursos, puede eliminar dicho grupo y todos los recursos asociados. Vuelva a comprobar el contenido del grupo de recursos para asegurarse de que no haya nada que desee conservar. Si no desea eliminar todo el grupo, puede eliminar recursos individuales en su lugar.

> [!IMPORTANT]
> La eliminación de un grupo de recursos es irreversible.

Quite el grupo **IoTEdgeResources**. La eliminación de un grupo de recursos puede tardar unos minutos.

```azurecli-interactive
az group delete --name IoTEdgeResources
```

Puede confirmar que se ha eliminado el grupo de recursos mediante la visualización de la lista de grupos de recursos.

```azurecli-interactive
az group list
```

## <a name="next-steps"></a>Pasos siguientes

En esta guía de inicio rápido, ha creado un dispositivo IoT Edge y ha usado la interfaz en la nube de Azure IoT Edge para implementar el código en el dispositivo. Ahora tiene un dispositivo de prueba que genera datos sin procesar acerca de su entorno.

El siguiente paso es configurar el entorno de desarrollo local para que pueda empezar a crear módulos de IoT Edge que ejecutan la lógica de negocios.

> [!div class="nextstepaction"]
> [Empezar a desarrollar módulos de IoT Edge para dispositivos Linux](tutorial-develop-for-linux.md)
