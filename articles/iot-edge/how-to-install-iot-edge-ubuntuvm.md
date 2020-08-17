---
title: Ejecución de Azure IoT Edge en máquinas virtuales Ubuntu | Microsoft Docs
description: Instrucciones de configuración de Azure IoT Edge en máquinas virtuales de Ubuntu 18.04 LTS
author: toolboc
manager: veyalla
ms.reviewer: kgremban
ms.service: iot-edge
services: iot-edge
ms.topic: conceptual
ms.date: 06/29/2020
ms.author: pdecarlo
ms.custom: devx-track-azurecli
ms.openlocfilehash: e70b22b3edaae96e00306d5d0a93d229e11aac41
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87494084"
---
# <a name="run-azure-iot-edge-on-ubuntu-virtual-machines"></a>Ejecución de Azure IoT Edge en máquinas virtuales Ubuntu

El entorno de ejecución de Azure IoT Edge es lo que convierte a un dispositivo en un dispositivo IoT Edge. El entorno de ejecución se puede implementar en dispositivos tan pequeños como un Raspberry Pi o tan grandes como un servidor industrial. Una vez que un dispositivo está configurado con el entorno de ejecución de IoT Edge, puede empezar a implementar lógica de negocios en él desde la nube.

Para más información acerca de cómo funciona el entorno de ejecución de IoT Edge y los componentes que se incluyen, consulte [Información del entorno de ejecución de Azure IoT Edge y su arquitectura](iot-edge-runtime.md).

En este artículo se indica cómo implementar una máquina virtual de Ubuntu 18.04 LTS con el tiempo de ejecución de Azure IoT Edge instalado y configurado usando una cadena de conexión de dispositivo suministrada previamente. La implementación se efectúa a través de una [plantilla de Azure Resource Manager](../azure-resource-manager/templates/overview.md) basada en [cloud-init](../virtual-machines/linux/using-cloud-init.md
) que se conserva en el repositorio [iotedge-vm-deploy](https://github.com/Azure/iotedge-vm-deploy) del proyecto.

En el primer arranque, la máquina virtual de Ubuntu 18.04 LTS [instala la versión más reciente del entorno de ejecución de Azure IoT Edge vía cloud-init](https://github.com/Azure/iotedge-vm-deploy/blob/master/cloud-init.txt). También establece una cadena de conexión proporcionada antes de que se inicie el tiempo de ejecución, lo que permite configurar y conectar fácilmente el dispositivo IoT Edge sin necesidad de iniciar una sesión de SSH o de escritorio remoto. 

## <a name="deploy-using-deploy-to-azure-button"></a>Implementación con el botón Implementar en Azure

Con el botón [Implementar en Azure](../azure-resource-manager/templates/deploy-to-azure-button.md) se puede realizar una implementación simplificada de las [plantillas de Azure Resource Manager](../azure-resource-manager/templates/overview.md) que se conservan en GitHub.  En esta sección se explica el uso del botón Implementar en Azure incluido en el repositorio del proyecto, [iotedge-vm-deploy](https://github.com/Azure/iotedge-vm-deploy).  


1. Implementaremos una máquina virtual Linux habilitada para Azure IoT Edge usando la plantilla iotedge-vm-deploy de Azure Resource Manager.  Para empezar, haga clic en el siguiente botón:

    [![Botón Implementar en Azure de iotedge-vm-deploy](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fazure%2Fiotedge-vm-deploy%2Fmaster%2FedgeDeploy.json)

1. En la ventana que se abre, rellene los campos de formulario disponibles:

    > [!div class="mx-imgBorder"]
    > [![Captura de pantalla de la plantilla iotedge-vm-deploy](./media/how-to-install-iot-edge-ubuntuvm/iotedge-vm-deploy.png)](./media/how-to-install-iot-edge-ubuntuvm/iotedge-vm-deploy.png)

    **Suscripción**: suscripción de Azure activa en la que se va a implementar la máquina virtual.

    **Grupo de recursos**: grupo de recursos existente o recién creado que va a contener la máquina virtual y sus recursos asociados.

    **Prefijo de etiqueta DNS**: valor requerido (de su elección) que se usa para colocar el nombre de host de la máquina virtual a modo de prefijo.

    **Nombre de usuario administrador**: nombre de usuario al que se proporcionarán privilegios raíz en la implementación.

    **Cadena de conexión de dispositivo**: [cadena de conexión de dispositivo](how-to-register-device.md) de un dispositivo que se ha creado en el [IoT Hub](../iot-hub/about-iot-hub.md) de su elección.

    **Tamaño de VM**: [tamaño](../cloud-services/cloud-services-sizes-specs.md) de la máquina virtual que se va a implementar.

    **Versión del SO Ubuntu**: versión del sistema operativo Ubuntu que se va a instalar en la máquina virtual base.

    **Ubicación**: [región geográfica](https://azure.microsoft.com/global-infrastructure/locations/) donde se va a implementar la máquina virtual. Este valor se establece de forma predeterminada en la ubicación del grupo de recursos seleccionado.

    **Tipo de autenticación**: elija **sshPublicKey** o **password**, según cuál sea su preferencia.

    **Contraseña o clave de administrador**: valor de la clave pública SSH o de la contraseña en función del tipo de autenticación que se haya escogido.

    Cuando se hayan rellenado todos los campos, active la casilla situada en la parte inferior de la página para aceptar los términos y seleccione **Adquirir** para iniciar la implementación.

1. Confirme que la implementación finaliza correctamente.  Se debe haber implementado un recurso de máquina virtual en el grupo de recursos seleccionado.  Tome nota del nombre del equipo, que debe tener el formato `vm-0000000000000`. Anote también el **Nombre DNS** asociado, que debe tener el formato `<dnsLabelPrefix>`.`<location>`.cloudapp.azure.com.

    El **Nombre DNS** se puede obtener en la sección **Información general** de la máquina virtual recién implementada en Azure Portal.

    > [!div class="mx-imgBorder"]
    > [![Captura de pantalla con el nombre DNS de la máquina virtual iotedge](./media/how-to-install-iot-edge-ubuntuvm/iotedge-vm-dns-name.png)](./media/how-to-install-iot-edge-ubuntuvm/iotedge-vm-dns-name.png)

1. Si quiere conectarse a través de SSH a la máquina virtual después de la instalación, use el **Nombre DNS** asociado con el comando: `ssh <adminUsername>@<DNS_Name>`.

## <a name="deploy-from-azure-cli"></a>Implementación desde la CLI de Azure

1. Asegúrese de que ha instalado la extensión iot de la CLI de Azure, así:
    ```azurecli-interactive
    az extension add --name azure-iot
    ```

1. Si está usando la CLI de Azure en su escritorio, empiece iniciando sesión:

   ```azurecli-interactive
   az login
   ```

1. Si tiene varias suscripciones, seleccione la que quiera usar:
   1. Muestre las suscripciones:

      ```azurecli-interactive
      az account list --output table
      ```

   1. Copie el campo SubscriptionID de la suscripción que quiera usar.

   1. Establezca la suscripción de trabajo con el identificador que ha copiado:

      ```azurecli-interactive
      az account set -s <SubscriptionId>
      ```

1. Cree un grupo de recursos nuevo (o especifique uno que ya exista en los pasos siguientes):

   ```azurecli-interactive
   az group create --name IoTEdgeResources --location westus2
   ```

1. Creación de una máquina virtual:

    Para usar un **authenticationType** de tipo `password`, vea el siguiente ejemplo:

   ```azurecli-interactive
   az deployment group create \
   --resource-group IoTEdgeResources \
   --template-uri "https://aka.ms/iotedge-vm-deploy" \
   --parameters dnsLabelPrefix='my-edge-vm1' \
   --parameters adminUsername='<REPLACE_WITH_USERNAME>' \
   --parameters deviceConnectionString=$(az iot hub device-identity show-connection-string --device-id <REPLACE_WITH_DEVICE-NAME> --hub-name <REPLACE-WITH-HUB-NAME> -o tsv) \
   --parameters authenticationType='password' \
   --parameters adminPasswordOrKey="<REPLACE_WITH_SECRET_PASSWORD>"
   ```

    Para autenticarse con una clave SSH, puede hacerlo especificando un **authenticationType** de tipo `sshPublicKey` y, tras ello, proporcionando el valor de la clave SSH en el parámetro **adminPasswordOrKey**.  A continuación se muestra un ejemplo.

    ```azurecli-interactive
    #Generate the SSH Key
    ssh-keygen -m PEM -t rsa -b 4096 -q -f ~/.ssh/iotedge-vm-key -N ""  

    #Create a VM using the iotedge-vm-deploy script
    az deployment group create \
    --resource-group IoTEdgeResources \
    --template-uri "https://aka.ms/iotedge-vm-deploy" \
    --parameters dnsLabelPrefix='my-edge-vm1' \
    --parameters adminUsername='<REPLACE_WITH_USERNAME>' \
    --parameters deviceConnectionString=$(az iot hub device-identity show-connection-string --device-id <REPLACE_WITH_DEVICE-NAME> --hub-name <REPLACE-WITH-HUB-NAME> -o tsv) \
    --parameters authenticationType='sshPublicKey' \
    --parameters adminPasswordOrKey="$(< ~/.ssh/iotedge-vm-key.pub)"
    ```

1. Confirme que la implementación finaliza correctamente.  Se debe haber implementado un recurso de máquina virtual en el grupo de recursos seleccionado.  Tome nota del nombre del equipo, que debe tener el formato `vm-0000000000000`. Anote también el **Nombre DNS** asociado, que debe tener el formato `<dnsLabelPrefix>`.`<location>`.cloudapp.azure.com.

    El **Nombre DNS** se puede obtener de la salida con formato JSON del paso anterior, en la sección **outputs** de la entrada **public SSH**.  El valor de esta entrada se puede usar para conectarse a través de SSH en el equipo recién implementado.

    ```bash
    "outputs": {
      "public SSH": {
        "type": "String",
        "value": "ssh <adminUsername>@<DNS_Name>"
      }
    }
    ```

    El **Nombre DNS** también se puede obtener en la sección **Información general** de la máquina virtual recién implementada en Azure Portal.

    > [!div class="mx-imgBorder"]
    > [![Captura de pantalla con el nombre DNS de la máquina virtual iotedge](./media/how-to-install-iot-edge-ubuntuvm/iotedge-vm-dns-name.png)](./media/how-to-install-iot-edge-ubuntuvm/iotedge-vm-dns-name.png)

1. Si quiere conectarse a través de SSH a la máquina virtual después de la instalación, use el **Nombre DNS** asociado con el comando: `ssh <adminUsername>@<DNS_Name>`.

## <a name="next-steps"></a>Pasos siguientes

Ahora que tiene un dispositivo IoT Edge aprovisionado con el entorno de ejecución instalado, puede [implementar módulos de IoT Edge](how-to-deploy-modules-portal.md).

Si tiene problemas con la instalación correcta del entorno de ejecución de Azure IoT Edge, consulte la página de [solución de problemas](troubleshoot.md).

Para actualizar una instalación existente a la versión más reciente de IoT Edge, vea [Actualice el archivo de configuración del demonio de seguridad y el entorno de ejecución de IoT Edge](how-to-update-iot-edge.md).

Si quiere abrir los puertos para acceder a la máquina virtual a través de SSH u otras conexiones entrantes, vea la documentación de Azure Virtual Machines sobre cómo [abrir puertos y puntos de conexión en una máquina virtual Linux](../virtual-machines/linux/nsg-quickstart.md).
