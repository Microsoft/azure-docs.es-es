---
title: Instalación de Azure IoT Edge para Linux en Windows | Microsoft Docs
description: Instrucciones de instalación de Azure IoT Edge para contenedores de Linux en Windows 10, Windows Server y Windows IoT Core
author: kgremban
manager: philmea
ms.reviewer: veyalla
ms.service: iot-edge
services: iot-edge
ms.topic: conceptual
ms.date: 05/06/2019
ms.author: kgremban
ms.openlocfilehash: c3a23e0c2546da55f977d589eb38607994d3902b
ms.sourcegitcommit: cd0a1ae644b95dbd3aac4be295eb4ef811be9aaa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/19/2020
ms.locfileid: "88611797"
---
# <a name="use-iot-edge-on-windows-to-run-linux-containers"></a>Uso de IoT Edge en Windows para ejecutar contenedores de Linux

Pruebe los módulos de IoT Edge para dispositivos Linux con una máquina Windows.

En un escenario de producción, los dispositivos Windows solo deben ejecutar contenedores de Windows. Sin embargo, un escenario de desarrollo común es usar un equipo Windows para crear módulos de IoT Edge para dispositivos Linux. El entorno de ejecución de Azure IoT Edge para Windows permite ejecutar contenedores de Linux con fines de **prueba y desarrollo**.

En este artículo se enumeran los pasos para instalar el entorno de ejecución de Azure IoT Edge con contenedores de Linux en el sistema Windows x64 (AMD e Intel). Para más información sobre el instalador del entorno de ejecución de Azure IoT Edge, incluidos detalles sobre todos los parámetros de instalación, consulte [Instalación del entorno de ejecución de Azure IoT Edge en Windows](how-to-install-iot-edge-windows.md).

Para obtener información acerca de lo que se incluye en la versión más reciente de IoT Edge, vea las [versiones de Azure IoT Edge](https://github.com/Azure/azure-iotedge/releases).

## <a name="prerequisites"></a>Prerequisites

Utilice esta sección para revisar si el dispositivo Windows puede admitir IoT Edge y para prepararlo para un motor de contenedor antes de la instalación.

### <a name="supported-windows-versions"></a>Versiones de Windows admitidas

Azure IoT Edge con contenedores de Linux puede ejecutarse en cualquier versión de Windows que cumpla con los [requisitos de Docker Desktop](https://docs.docker.com/docker-for-windows/install/#what-to-know-before-you-install)

Si quiere instalar IoT Edge en una máquina virtual, habilite la virtualización anidada y asigne al menos 2 GB de memoria. La manera que habilita la virtualización anidada difiere en función del hipervisor que usa. En el caso de Hyper-V, las máquinas virtuales de generación 2 tienen la virtualización anidada habilitada de manera predeterminada. En VMware, hay un botón de alternancia para habilitar la característica en la máquina virtual.

### <a name="prepare-the-container-engine"></a>Preparación del motor de contenedor

Azure IoT Edge se basa en un motor de contenedor [compatible con OCI](https://www.opencontainers.org/). La principal diferencia de configuración entre ejecutar contenedores de Windows y de Linux en una máquina Windows es que la instalación de IoT Edge incluye un entorno de ejecución de contenedores de Windows, pero es necesario proporcionar un entorno de ejecución propio para los contenedores de Linux antes de instalar IoT Edge.

Si quiere configurar una máquina Windows para desarrollar y probar contenedores para dispositivos Linux, puede usar [Docker Desktop](https://www.docker.com/docker-windows) como el motor de contenedor. Necesita instalar Docker y configurarlo para [usar los contenedores de Linux](https://docs.docker.com/docker-for-windows/#switch-between-windows-and-linux-containers) antes de instalar IoT Edge.  

Si su dispositivo IoT Edge es un equipo Windows, compruebe que cumple los [requisitos del sistema](https://docs.microsoft.com/virtualization/hyper-v-on-windows/reference/hyper-v-requirements) para Hyper-V.

## <a name="install-iot-edge-on-a-new-device"></a>Instalación de IoT Edge en un dispositivo nuevo

>[!NOTE]
>Los paquetes de software de Azure IoT Edge están sujetos a los términos de licencia ubicados en los paquetes (en el directorio LICENSE). Lea los términos de licencia antes de usar el paquete. La instalación y uso del paquete constituye la aceptación de estos términos. Si no acepta los términos de licencia, no utilice el paquete.

Un script de PowerShell descarga e instala el demonio de seguridad de Azure IoT Edge. Después, el demonio de seguridad inicia el primero de los dos módulos en tiempo de ejecución, el agente de IoT Edge, que permite implementaciones remotas de otros módulos.

Cuando instale por primera vez el tiempo de ejecución de IoT Edge en un dispositivo, debe aprovisionar el dispositivo con una identidad de un IoT hub. Un único dispositivo de IoT Edge se puede aprovisionar manualmente mediante una cadena de conexiones de dispositivo proporcionada por el centro de IoT. O bien, puede usar el servicio Device Provisioning para aprovisionar dispositivos automáticamente, lo que resulta útil cuando tiene muchos dispositivos para configurar.

Puede leer más sobre los distintos parámetros y opciones de instalación en el artículo [Instalación del entorno de ejecución de Azure IoT Edge en Windows](how-to-install-iot-edge-windows.md). Una vez que Docker Desktop está instalado y configurado para los contenedores de Linux, la principal diferencia de instalación es declarar Linux con el parámetro **-ContainerOs**. Por ejemplo:

1. Si aún no lo ha hecho, registre un nuevo dispositivo IoT Edge y recupere la cadena de conexión del dispositivo. Copie la cadena de conexión para usarla más adelante en esta sección. Este paso se puede completar con las siguientes herramientas:

   * [Azure Portal](how-to-register-device.md#register-in-the-azure-portal)
   * [CLI de Azure](how-to-register-device.md#register-with-the-azure-cli)
   * [Visual Studio Code](how-to-register-device.md#register-with-visual-studio-code)

2. Ejecute PowerShell como administrador.

   >[!NOTE]
   >Use una sesión de AMD64 de PowerShell para instalar IoT Edge, no PowerShell (x86). Si no está seguro de qué tipo de sesión usa, ejecute el comando siguiente:
   >
   >```powershell
   >(Get-Process -Id $PID).StartInfo.EnvironmentVariables["PROCESSOR_ARCHITECTURE"]
   >```

3. El comando **Deploy-IoTEdge** comprueba si la versión del equipo Windows es compatible, activa la característica de contenedores y descarga tanto el entorno de ejecución de Moby (que no se usa para los contenedores de Linux) como el de IoT Edge. El comando se establece de manera predeterminada en los contenedores de Windows, por lo que debe declarar a Linux como el sistema operativo de contenedor deseado.

   ```powershell
   . {Invoke-WebRequest -useb aka.ms/iotedge-win} | Invoke-Expression; `
   Deploy-IoTEdge -ContainerOs Linux
   ```

4. En este momento, los dispositivos IoT Core pueden reiniciarse automáticamente. Es posible que otros dispositivos Windows 10 o Windows Server soliciten su reinicio. En ese caso, reinícielo ahora. Una vez que el dispositivo esté listo, vuelva a ejecutar PowerShell como administrador.

5. El comando **Initialize-IoTEdge** configura el entorno de ejecución de Azure IoT Edge en el equipo. El comando tiene como valor predeterminado el aprovisionamiento manual con una cadena de conexión del dispositivo. Vuelva a declarar a Linux como el sistema operativo de contenedor deseado.

   ```powershell
   . {Invoke-WebRequest -useb aka.ms/iotedge-win} | Invoke-Expression; `
   Initialize-IoTEdge -ContainerOs Linux
   ```

6. Cuando se solicite, especifique la cadena de conexión del dispositivo que recuperó en el paso 1. La cadena de conexión del dispositivo asocia el dispositivo físico con un identificador de dispositivo de IoT Hub.

   La cadena de conexión del dispositivo adopta el formato siguiente y no debe incluir comillas: `HostName={IoT hub name}.azure-devices.net;DeviceId={device name};SharedAccessKey={key}`

## <a name="verify-successful-installation"></a>Comprobación de instalación correcta

Compruebe el estado del servicio IoT Edge:

```powershell
Get-Service iotedge
```

Examine los registros del servicio de los últimos 5 minutos:

```powershell
. {Invoke-WebRequest -useb aka.ms/iotedge-win} | Invoke-Expression; Get-IoTEdgeLog
```

Ejecute una comprobación automatizada para detectar los errores de configuración y red más comunes:

```powershell
iotedge check
```

Enumere los módulos en ejecución. Después de una instalación nueva, el único módulo que debería en ejecución es **edgeAgent**. Después de [implementar los módulos de IoT Edge](how-to-deploy-modules-portal.md) por primera vez, el otro módulo del sistema, **edgeHub**, se iniciará también en el dispositivo.

```powershell
iotedge list
```

## <a name="next-steps"></a>Pasos siguientes

Ahora que tiene un dispositivo IoT Edge aprovisionado con el entorno de ejecución instalado, puede [implementar módulos de IoT Edge](how-to-deploy-modules-portal.md).

Si tiene problemas con la instalación correcta de IoT Edge, vea la página de [solución de problemas](troubleshoot.md).

Para actualizar una instalación existente a la versión más reciente de IoT Edge, vea [Actualice el archivo de configuración del demonio de seguridad y el entorno de ejecución de IoT Edge](how-to-update-iot-edge.md).
