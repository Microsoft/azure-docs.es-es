---
title: Ejecución de Azure IoT Edge en máquinas virtuales de Windows Server | Microsoft Docs
description: Instrucciones de configuración de Azure IoT Edge en máquinas virtuales de Windows Server Marketplace
author: gregman-msft
manager: arjmands
ms.reviewer: kgremban
ms.service: iot-edge
services: iot-edge
ms.topic: conceptual
ms.date: 06/12/2019
ms.author: philmea
ms.custom: devx-track-azurecli
ms.openlocfilehash: 0d8344f5f6b4ccc9f58cc1809f5d625e7e4adf68
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87494067"
---
# <a name="run-azure-iot-edge-on-windows-server-virtual-machines"></a>Ejecución de Azure IoT Edge en máquinas virtuales de Windows Server

El entorno de ejecución de Azure IoT Edge es lo que convierte a un dispositivo en un dispositivo IoT Edge. El entorno de ejecución se puede implementar en dispositivos tan pequeños como un Raspberry Pi o tan grandes como un servidor industrial. Una vez que un dispositivo está configurado con el entorno de ejecución de IoT Edge, puede empezar a implementar lógica de negocios en él desde la nube.

Para más información acerca de cómo funciona el entorno de ejecución de IoT Edge y los componentes que se incluyen, consulte [Información del entorno de ejecución de Azure IoT Edge y su arquitectura](iot-edge-runtime.md).

En este artículo se enumeran los pasos para ejecutar el entorno de ejecución de Azure IoT Edge en una máquina virtual de Windows Server 2019 con la oferta de Azure Marketplace de [Windows Server](https://www.microsoft.com/cloud-platform/windows-server-pricing). Siga las instrucciones de [Instalación del entorno de ejecución de Azure IoT Edge en Windows](how-to-install-iot-edge-windows.md) para usarlo con otras versiones.

## <a name="deploy-from-the-azure-marketplace"></a>Implementación desde Azure Marketplace

1. Vaya a la oferta de Azure Marketplace de [Windows Server](https://www.microsoft.com/cloud-platform/windows-server-pricing) o busque "Windows Server" en [Azure Marketplace](https://azuremarketplace.microsoft.com/).
2. Seleccione **OBTENERLA AHORA**.
3. En **Plan de Software**, busque "Windows Server 2019 Datacenter Server Core with Containers" y seleccione **Continuar** en el cuadro de diálogo siguiente.
    * También puede seguir estas instrucciones con otras versiones de Windows Server con contenedores.
4. Una vez en Azure Portal, seleccione **Crear** y siga el asistente para implementar la máquina virtual.
    * Si es la primera vez que prueba una máquina virtual, es más fácil usar una contraseña y habilitar RDP y SSH en el menú del puerto de entrada público.
    * Si tiene una carga de trabajo con uso intensivo de recursos, debe actualizar el tamaño de máquina virtual mediante la adición de más CPU o memoria.
5. Una vez implementada la máquina virtual, configúrela para conectarse a IoT Hub:
    1. Copie la cadena de conexión del dispositivo IoT Edge creado en IoT Hub. Vea el procedimiento [Recuperar la cadena de conexión en Azure Portal](how-to-register-device.md#retrieve-the-connection-string-in-the-azure-portal).
    1. Seleccione el recurso de máquina virtual recién creado en Azure Portal y abra la opción **Ejecución de comando**.
    1. Seleccione la opción **RunPowerShellScript**.
    1. Copie este script en la ventana de comandos con la cadena de conexión del dispositivo:

        ```powershell
        . {Invoke-WebRequest -useb aka.ms/iotedge-win} | Invoke-Expression; `
        Install-IoTEdge -Manual -DeviceConnectionString '<connection-string>'
        ```

    1. Ejecute el script para instalar el entorno de ejecución de IoT Edge y establezca la cadena de conexión mediante la selección de **Ejecutar**.
    1. Transcurrido un minuto o dos, debería ver un mensaje sobre la instalación del entorno de ejecución de Edge y su correcto aprovisionamiento.

## <a name="deploy-from-the-azure-portal"></a>Implementación desde Azure Portal

1. Desde Azure Portal, busque "Windows Server" y seleccione **Windows Server 2019 Datacenter** para iniciar el flujo de trabajo de creación de la máquina virtual.
2. En **Seleccionar un plan de software**, elija "Windows Server 2019 Datacenter Server Core with Containers" y seleccione **Crear**.
3. Siga el paso 5 de las instrucciones de "Implementación desde Azure Marketplace" anteriores.

## <a name="deploy-from-azure-cli"></a>Implementación desde la CLI de Azure

1. Si está usando la CLI de Azure en el escritorio, empiece por iniciar sesión:

   ```azurecli-interactive
   az login
   ```

1. Si tiene varias suscripciones, seleccione la que desea utilizar:
   1. Muestre las suscripciones:

      ```azurecli-interactive
      az account list --output table
      ```

   1. Copie el campo SubscriptionID para la suscripción que desea utilizar.
   1. Ejecute este comando con el identificador que acaba de copiar:

      ```azurecli-interactive
      az account set -s {SubscriptionId}
      ```

1. Cree un grupo de recursos nuevo (o especifique uno que ya exista en los pasos siguientes):

   ```azurecli-interactive
   az group create --name IoTEdgeResources --location westus2
   ```

1. Creación de una máquina virtual:

   ```azurecli-interactive
   az vm create -g IoTEdgeResources -n EdgeVM --image MicrosoftWindowsServer:WindowsServer:2019-Datacenter-Core-with-Containers:latest  --admin-username azureuser --generate-ssh-keys --size Standard_DS1_v2
   ```

   * Este comando le solicitará contraseña, pero puede agregar la opción `--admin-password` para establecerla más fácilmente en un script.
   * La imagen de Windows Server Core admite línea de comandos solo con Escritorio remoto, por lo tanto, si quiere la experiencia de escritorio completa, debe especificar `MicrosoftWindowsServer:WindowsServer:2019-Datacenter-with-Containers:latest` como imagen.

1. Establezca la cadena de conexión del dispositivo (puede seguir el procedimiento [Recuperar la cadena de conexión con la CLI de Azure](how-to-register-device.md#retrieve-the-connection-string-with-the-azure-cli) si no está familiarizado con este proceso):

   ```azurecli-interactive
   az vm run-command invoke -g IoTEdgeResources -n EdgeVM --command-id RunPowerShellScript --script ". {Invoke-WebRequest -useb aka.ms/iotedge-win} | Invoke-Expression; `Install-IoTEdge -Manual -DeviceConnectionString '<connection-string>'"
   ```

## <a name="next-steps"></a>Pasos siguientes

Ahora que tiene un dispositivo IoT Edge aprovisionado con el entorno de ejecución instalado, puede [implementar módulos de IoT Edge](how-to-deploy-modules-portal.md).

Si tiene problemas con la instalación correcta del entorno de ejecución de Edge, consulte la página de [solución de problemas](troubleshoot.md).

Para actualizar una instalación existente a la versión más reciente de IoT Edge, vea [Actualice el archivo de configuración del demonio de seguridad y el entorno de ejecución de IoT Edge](how-to-update-iot-edge.md).

Más información sobre el uso de las máquinas virtuales Windows en la [Documentación sobre máquinas virtuales Windows](https://docs.microsoft.com/azure/virtual-machines/windows/).

Si desea conectarse mediante SSH a la máquina virtual después de la instalación, siga la guía de [instalación de OpenSSH para Windows Server](https://docs.microsoft.com/windows-server/administration/openssh/openssh_install_firstuse#installing-openssh-with-powershell) con Escritorio remoto o Powershell remoto.
