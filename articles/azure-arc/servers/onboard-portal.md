---
title: Conexión de máquinas híbridas a Azure desde Azure Portal
description: En este artículo, obtendrá información sobre cómo instalar el agente y conectar máquinas a Azure mediante servidores habilitados para Azure Arc desde Azure Portal.
ms.date: 09/02/2020
ms.topic: conceptual
ms.custom: references_regions
ms.openlocfilehash: 7435256dda68b2689aeb19b237f499d50b418055
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90887630"
---
# <a name="connect-hybrid-machines-to-azure-from-the-azure-portal"></a>Conexión de máquinas híbridas a Azure desde Azure Portal

Con el fin de implementar en su entorno servidores habilitados para Azure Arc para una máquina Windows o Linux, o un número reducido de ellas, debe realizar una serie de pasos manualmente. O bien, puede usar un método automatizado mediante la ejecución de un script de plantilla que le proporcionaremos. Este script automatiza la descarga e instalación de ambos agentes.

Este método requiere que tenga permisos de administrador en la máquina para instalar y configurar el agente. En Linux, mediante la cuenta raíz, y en Windows, usted es miembro del grupo local de administradores.

Antes de comenzar, asegúrese de revisar los [requisitos previos](agent-overview.md#prerequisites) y compruebe que su suscripción y sus recursos los cumplen.

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="generate-the-installation-script-from-the-azure-portal"></a>Generación del script de instalación desde Azure Portal

El script para automatizar la descarga y la instalación, y para establecer la conexión con Azure Arc, está disponible en Azure Portal. Para completar el proceso, haga lo siguiente:

1. En el explorador, vaya a [Azure Portal](https://aka.ms/hybridmachineportal).

1. En la página **Servidores: Azure Arc**, seleccione **Agregar** en la parte superior izquierda.

1. En la página **Seleccionar un método**, seleccione el icono **Agregar servidores mediante un script interactivo** y, después, seleccione **Generar script**.

1. En la página **Generar script**, seleccione la suscripción y el grupo de recursos en el que desea que se administre la máquina en Azure. Seleccione la ubicación de Azure en la que se almacenarán los metadatos de la máquina.

    >[!NOTE]
    >Los servidores habilitados para Azure Arc solo admiten las siguientes regiones:
    >- EastUS
    >- WestUS2
    >- Oeste de Europa
    >- SoutheastAsia
    >
    >Revise las consideraciones adicionales que se deben tener en cuenta al seleccionar una región [aquí](overview.md#supported-regions), en el artículo de información general.

1. En la página **Requisitos previos**, revise la información y, luego, seleccione **Siguiente: Detalles del recurso**.

1. En la página **Detalles del recurso**, proporcione lo siguiente:

    1. En la lista desplegable **Grupo de recursos**, seleccione el grupo de recursos desde el que se administrará la máquina.
    1. En la lista desplegable **Región**, seleccione la región de Azure en la que se almacenarán los metadatos de los servidores.
    1. En la lista desplegable **Sistema operativo**, seleccione el sistema operativo en el que se ejecutará el script.
    1. Si la máquina se comunica mediante un servidor proxy para conectarse a Internet, especifique la dirección IP del servidor proxy o el nombre y el número de puerto que usará la máquina para comunicarse con el servidor proxy. Escriba el valor con el formato `http://<proxyURL>:<proxyport>`.
    1. Seleccione **Siguiente: Etiquetas**.

1. En la página **Etiquetas**, revise las **etiquetas de ubicación física** predeterminadas sugeridas y escriba un valor, o bien especifique una o varias **etiquetas personalizadas** para que cumplan sus estándares.

1. Seleccione **Siguiente: Descargar y ejecutar el script**.

1. En la pestaña **Descargar y ejecutar el script**, revise la información del resumen y seleccione **Descargar**. Si todavía necesita realizar cambios, seleccione **Anterior**.

## <a name="install-and-validate-the-agent-on-windows"></a>Instalación y validación del agente en Windows

### <a name="install-manually"></a>Instalación manual

Para instalar manualmente el agente de Connected Machine, puede ejecutar el paquete de Windows Installer llamado *AzureConnectedMachineAgent.msi*. Puede descargar la versión más reciente del [paquete de Windows Installer con el agente](https://aka.ms/AzureConnectedMachineAgent) del centro de descarga de Microsoft.

>[!NOTE]
>* Para instalar o desinstalar el agente, debe tener permisos de *administrador*.
>* Primero debe descargar y copiar el paquete del instalador en una carpeta en el servidor de destino o desde una carpeta de red compartida. Si ejecuta el paquete del instalador sin opciones, se inicia un asistente para instalación que puede seguir para realizar la instalación del agente de forma interactiva.

Si la máquina necesita comunicarse mediante un servidor proxy con el servicio, después de instalar el agente, debe ejecutar un comando que se describe en los pasos siguientes. Este comando permite establecer la variable de entorno del sistema del servidor proxy `https_proxy`.

Si no está familiarizado con las opciones de la línea de comandos para los paquetes de Windows Installer, consulte [Opciones de la línea de comandos estándar de msiexec](/windows/win32/msi/standard-installer-command-line-options) y [Opciones de la línea de comandos de msiexec](/windows/win32/msi/command-line-options).

Por ejemplo, ejecute el programa de instalación con el parámetro `/?` para revisar la opción de ayuda y referencia rápida.

```dos
msiexec.exe /i AzureConnectedMachineAgent.msi /?
```

1. Para instalar el agente de forma silenciosa y crear un archivo de registro de instalación en la carpeta `C:\Support\Logs`, ejecute el comando siguiente.

    ```dos
    msiexec.exe /i AzureConnectedMachineAgent.msi /qn /l*v "C:\Support\Logs\Azcmagentsetup.log"
    ```

    Si el agente no se inicia una vez completada la instalación, compruebe los registros para obtener información detallada del error. El directorio de registro es *%Programfiles%\AzureConnectedMachineAgentAgent\logs*.

2. Si la máquina necesita comunicarse a través de un servidor proxy, para establecer la variable de entorno del servidor proxy, ejecute el siguiente comando:

    ```powershell
    [Environment]::SetEnvironmentVariable("https_proxy", "http://{proxy-url}:{proxy-port}", "Machine")
    $env:https_proxy = [System.Environment]::GetEnvironmentVariable("https_proxy","Machine")
    # For the changes to take effect, the agent service needs to be restarted after the proxy environment variable is set.
    Restart-Service -Name himds
    ```

    >[!NOTE]
    >El agente no admite la configuración de la autenticación de proxy en esta versión preliminar.
    >

3. Después de instalar el agente, debe configurarlo para que se comunique con el servicio Azure Arc mediante la ejecución del siguiente comando:

    ```dos
    "%ProgramFiles%\AzureConnectedMachineAgent\azcmagent.exe" connect --resource-group "resourceGroupName" --tenant-id "tenantID" --location "regionName" --subscription-id "subscriptionID"
    ```

### <a name="install-with-the-scripted-method"></a>Instalación con el método con script

1. Inicie sesión en el servidor.

1. Abra un símbolo del sistema de PowerShell con privilegios elevados.

    >[!NOTE]
    >El script solo admite la ejecución desde una versión de 64 bits de Windows PowerShell.
    >

1. Cambie a la carpeta o recurso compartido en el que copió el script y ejecútelo en el servidor mediante el script `./OnboardingScript.ps1`.

Si el agente no se inicia una vez completada la instalación, compruebe los registros para obtener información detallada del error. El directorio de registro es *%Programfiles%\AzureConnectedMachineAgentAgent\logs*.

## <a name="install-and-validate-the-agent-on-linux"></a>Instalación y validación del agente en Linux

El agente de Connected Machine de Linux se proporciona en el formato de paquete preferido para la distribución (.RPM o .DEB) que se hospeda en el [repositorio de paquetes](https://packages.microsoft.com/) de Microsoft. Este [ conjunto de scripts de shell `Install_linux_azcmagent.sh`](https://aka.ms/azcmagent) realiza las acciones siguientes:

- Configura la máquina host para descargar el paquete del agente de packages.microsoft.com.
- Instala el paquete del proveedor de recursos híbridos.
- Registra la máquina en Azure Arc

Opcionalmente, puede configurar el agente con la información del proxy incluyendo el parámetro `--proxy "{proxy-url}:{proxy-port}"`.

El script también contiene la lógica para identificar las distribuciones admitidas y no admitidas, y comprueba los permisos necesarios para realizar la instalación.

En el siguiente ejemplo se descarga el agente y se instala:

```bash
# Download the installation package.
wget https://aka.ms/azcmagent -O ~/Install_linux_azcmagent.sh

# Install the connected machine agent.
bash ~/Install_linux_azcmagent.sh
```

Para descargar e instalar el agente, incluyendo el parámetro `--proxy` para configurar el agente para que se comunique mediante el servidor proxy, ejecute los siguientes comandos:

```bash
# Download the installation package.
wget https://aka.ms/azcmagent -O ~/Install_linux_azcmagent.sh

# Install the connected machine agent. 
bash ~/Install_linux_azcmagent.sh --proxy "{proxy-url}:{proxy-port}"
```

## <a name="verify-the-connection-with-azure-arc"></a>Comprobación de la conexión con Azure Arc

Después de instalar el agente y configurarlo para que se conecte a los servidores habilitados para Azure Arc, vaya a Azure Portal para comprobar que el servidor se ha conectado correctamente. Vea las máquinas en [Azure Portal](https://aka.ms/hybridmachineportal).

![Una conexión de servidor correcta](./media/onboard-portal/arc-for-servers-successful-onboard.png)

## <a name="next-steps"></a>Pasos siguientes

- Obtenga información sobre cómo administrar la máquina con [Azure Policy](../../governance/policy/overview.md) para, por ejemplo, la [configuración de invitado](../../governance/policy/concepts/guest-configuration.md) de VM, la comprobación de que la máquina informa al área de trabajo de Log Analytics esperada, la habilitación de la supervisión con [Azure Monitor con máquinas virtuales](../../azure-monitor/insights/vminsights-enable-policy.md) y mucho más.

- Obtenga más información sobre el [[agente de Log Analytics]](../../azure-monitor/platform/log-analytics-agent.md). El agente de Log Analytics para Windows y Linux es necesario si quiere recopilar datos de supervisión del sistema operativo y de las cargas de trabajo y administrarlos con runbooks de Automation o con características como Update Management, o bien mediante otros servicios de Azure, como [Azure Security Center](../../security-center/security-center-intro.md).