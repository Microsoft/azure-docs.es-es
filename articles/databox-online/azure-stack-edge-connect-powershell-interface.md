---
title: Conexión a un dispositivo Microsoft Azure Stack Edge Pro y administración mediante la interfaz de Windows PowerShell | Microsoft Docs
description: Se describe cómo conectarse a Azure Stack Edge Pro y cómo administrarlo mediante la interfaz de Windows PowerShell.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: how-to
ms.date: 06/25/2019
ms.author: alkohli
ms.openlocfilehash: b6b0fe7e9e096b252d33d25c4a70305e57d206b1
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90894417"
---
# <a name="manage-an-azure-stack-edge-pro-device-via-windows-powershell"></a>Administración de un dispositivo Azure Stack Edge Pro mediante Windows PowerShell

La solución Azure Stack Edge Pro permite procesar datos y enviarlos a través de la red a Azure. En este artículo se describen algunas de las tareas de configuración y administración del dispositivo Azure Stack Edge Pro. Puede usar Azure Portal, la interfaz de usuario web local o la interfaz de Windows PowerShell para administrar su dispositivo.

Este artículo se centra en las tareas que realizará mediante la interfaz de PowerShell. 

Asimismo, en este artículo se incluyen las secciones siguientes:

- Conectarse a la interfaz de PowerShell
- Crear un paquete de soporte
- Carga del certificado
- Restablecer el dispositivo
- Ver la información del dispositivo
- Obtener los registros de proceso
- Supervisar y solucionar problemas de los módulos de proceso

## <a name="connect-to-the-powershell-interface"></a>Conectarse a la interfaz de PowerShell

[!INCLUDE [Connect to admin runspace](../../includes/data-box-edge-gateway-connect-minishell.md)]

## <a name="create-a-support-package"></a>Crear un paquete de soporte

[!INCLUDE [Create a support package](../../includes/data-box-edge-gateway-create-support-package.md)]

## <a name="upload-certificate"></a>Carga del certificado

[!INCLUDE [Upload certificate](../../includes/data-box-edge-gateway-upload-certificate.md)]

También puede cargar certificados de IoT Edge para habilitar una conexión segura entre el dispositivo IoT Edge y los dispositivos de nivel inferior que pueden conectarse a él. Hay tres certificados de IoT Edge (con formato *.pem*) que necesita instalar:

- Certificado de entidad de certificación raíz o el propietario de la entidad de certificación
- Certificado de entidad de certificación de dispositivo
- Certificado clave del dispositivo

En el siguiente ejemplo se muestra el uso de este cmdlet para instalar certificados de IoT Edge:

```
Set-HcsCertificate -Scope IotEdge -RootCACertificateFilePath "\\hcfs\root-ca-cert.pem" -DeviceCertificateFilePath "\\hcfs\device-ca-cert.pem\" -DeviceKeyFilePath "\\hcfs\device-key-cert.pem" -Credential "username"
```
Cuando ejecute este cmdlet, se le solicitará que proporcione la contraseña para el recurso compartido de red.

Para obtener más información sobre los certificados, vaya a [Certificados de Azure IoT Edge](https://docs.microsoft.com/azure/iot-edge/iot-edge-certs) o a [Instalación de los certificados en la puerta de enlace](https://docs.microsoft.com/azure/iot-edge/how-to-create-transparent-gateway).

## <a name="view-device-information"></a>Ver la información del dispositivo
 
[!INCLUDE [View device information](../../includes/data-box-edge-gateway-view-device-info.md)]

## <a name="reset-your-device"></a>Restablecer el dispositivo

[!INCLUDE [Reset your device](../../includes/data-box-edge-gateway-deactivate-device.md)]

## <a name="get-compute-logs"></a>Obtener los registros de proceso

Si el rol de proceso está configurado en su dispositivo, también puede obtener los registros del proceso gracias a la interfaz de PowerShell.

1. [Conéctese a la interfaz de PowerShell](#connect-to-the-powershell-interface).
2. Use `Get-AzureDataBoxEdgeComputeRoleLogs` para obtener los registros del proceso del dispositivo.

    En el ejemplo siguiente se muestra el uso de este cmdlet:

    ```powershell
    Get-AzureDataBoxEdgeComputeRoleLogs -Path "\\hcsfs\logs\myacct" -Credential "username" -FullLogCollection
    ```

    Aquí tiene hay una descripción de los parámetros que se usan en el cmdlet:
    - `Path`: proporcione una ruta de red al recurso compartido en el que quiere crear el paquete de registros del proceso.
    - `Credential`: proporcione el nombre de usuario del recurso compartido de red. Cuando ejecute este cmdlet, deberá proporcionar la contraseña del recurso compartido.
    - `FullLogCollection`: este parámetro garantiza que el paquete de registros contendrá todos los registros del proceso. De forma predeterminada, el paquete de registros contiene solo un subconjunto de registros.

## <a name="monitor-and-troubleshoot-compute-modules"></a>Supervisar y solucionar problemas de los módulos de proceso

[!INCLUDE [Monitor and troubleshoot compute modules](../../includes/azure-stack-edge-monitor-troubleshoot-compute.md)]

## <a name="exit-the-remote-session"></a>Salir de la sesión remota

Para salir de la sesión remota de PowerShell, cierre la ventana de PowerShell.

## <a name="next-steps"></a>Pasos siguientes

- Implemente [Azure Stack Edge Pro](azure-stack-edge-deploy-prep.md) en Azure Portal.
