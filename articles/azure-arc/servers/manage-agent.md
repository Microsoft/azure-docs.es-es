---
title: Agente de administración de servidores habilitados para Azure Arc
description: En este artículo se describen las diferentes tareas de administración que normalmente realizará durante el ciclo de vida del agente de Connected Machine de Azure Arc para servidores.
ms.date: 07/30/2020
ms.topic: conceptual
ms.openlocfilehash: 6066226cea224b1e13262763b626c8c646a397d7
ms.sourcegitcommit: 4913da04fd0f3cf7710ec08d0c1867b62c2effe7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/14/2020
ms.locfileid: "88213127"
---
# <a name="managing-and-maintaining-the-connected-machine-agent"></a>Administración y mantenimiento del agente de Connected Machine

Después de la implementación inicial del agente de Connected Machine de Azure Arc para servidores (versión preliminar) para Windows o Linux, debe volver a configurar el agente, actualizarlo o quitarlo del equipo si alcanzó la fase de retirada de su ciclo de vida. Puede administrar fácilmente estas tareas de mantenimiento rutinarias manualmente o de manera automática, lo que reduce los errores de funcionamiento y los gastos.

## <a name="upgrading-agent"></a>Actualizar el agente

El agente de Azure Connected Machine para Windows y Linux se puede actualizar a la versión más reciente de forma manual o automática según sus necesidades. En la tabla siguiente se describen los métodos admitidos para la actualización del agente.

| Sistema operativo | Método de actualización |
|------------------|----------------|
| Windows | Manualmente<br> Windows Update |
| Ubuntu | [Apt](https://help.ubuntu.com/lts/serverguide/apt.html) |
| SUSE Linux Enterprise Server | [zypper](https://en.opensuse.org/SDB:Zypper_usage_11.3) |
| RedHat Enterprise, Amazon, CentOS Linux | [yum](https://wiki.centos.org/PackageManagement/Yum) |

### <a name="windows-agent"></a>Agente de Windows

El paquete de actualización para el agente de Connected Machine para Windows está disponible en:

* Microsoft Update

* [Catálogo de Microsoft Update](https://www.catalog.update.microsoft.com/Home.aspx)

* [Paquete de Windows Installer con el agente](https://aka.ms/AzureConnectedMachineAgent) del centro de descarga de Microsoft.

El agente se puede actualizar a partir de una variedad de métodos para admitir el proceso de administración de actualizaciones de software. Aparte de en Microsoft Update, también se puede ejecutar manualmente desde el símbolo del sistema, un script u otra solución de automatización, o desde el asistente para la interfaz de usuario ejecutando `AzureConnectedMachine.msi`.

> [!NOTE]
> * Para actualizar el agente, debe tener permisos de *administrador*.
> * Para actualizar manualmente, primero debe descargar y copiar el paquete del instalador en una carpeta en el servidor de destino o desde una carpeta de red compartida. 

Si no está familiarizado con las opciones de la línea de comandos para los paquetes de Windows Installer, consulte [Opciones de la línea de comandos estándar de msiexec](/windows/win32/msi/standard-installer-command-line-options) y [Opciones de la línea de comandos de msiexec](/windows/win32/msi/command-line-options).

#### <a name="to-upgrade-using-the-setup-wizard"></a>Para actualizar mediante el Asistente de instalación

1. Inicie sesión en el equipo con una cuenta con derechos administrativos.

2. Ejecute **AzureConnectedMachineAgent.msi** para iniciar el Asistente para la instalación.

El Asistente para la instalación detecta si existe una versión anterior y, a continuación, actualiza el agente automáticamente. Cuando se complete la actualización, el Asistente para la instalación se cerrará automáticamente.

#### <a name="to-upgrade-from-the-command-line"></a>Actualizar desde la línea de comandos

1. Inicie sesión en el equipo con una cuenta con derechos administrativos.

2. Para actualizar el agente de forma silenciosa y crear un archivo de registro de instalación en la carpeta `C:\Support\Logs`, ejecute el comando siguiente.

    ```dos
    msiexec.exe /i AzureConnectedMachineAgent.msi /qn /l*v "C:\Support\Logs\Azcmagentupgradesetup.log"
    ```

### <a name="linux-agent"></a>Agente Linux

Para actualizar el agente en una máquina Linux a la versión más reciente, participan dos comandos. Un comando para actualizar el índice de paquetes locales con la lista de los paquetes disponibles más recientes de los repositorios, y un comando para actualizar el paquete local.

Puede descargar el paquete de agente más reciente del [repositorio de paquetes](https://packages.microsoft.com/) de Microsoft.

> [!NOTE]
> Para actualizar el agente, debe tener permisos de acceso *raíz* o una cuenta que tenga derechos elevados que use Sudo.

#### <a name="upgrade-ubuntu"></a>Actualización de Ubuntu

1. Para actualizar el índice de paquetes locales con los últimos cambios realizados en los repositorios, ejecute el siguiente comando:

    ```bash
    apt update
    ```

2. Para actualizar el sistema, ejecute el siguiente comando:

    ```bash
    apt upgrade
    ```

Las acciones del comando [apt](https://help.ubuntu.com/lts/serverguide/apt.html), como la instalación y la eliminación de paquetes, se registran en el archivo de registro `/var/log/dpkg.log`.

#### <a name="upgrade-red-hatcentosamazon-linux"></a>Actualización de Red Hat/CentOS/Amazon Linux

1. Para actualizar el índice de paquetes locales con los últimos cambios realizados en los repositorios, ejecute el siguiente comando:

    ```bash
    yum check-update
    ```

2. Para actualizar el sistema, ejecute el siguiente comando:

    ```bash
    yum update
    ```

Las acciones del comando [yum](https://access.redhat.com/articles/yum-cheat-sheet), como la instalación y la eliminación de paquetes, se registran en el archivo de registro `/var/log/yum.log`. 

#### <a name="upgrade-suse-linux-enterprise"></a>Actualización de SUSE Linux Enterprise

1. Para actualizar el índice de paquetes locales con los últimos cambios realizados en los repositorios, ejecute el siguiente comando:

    ```bash
    zypper refresh
    ```

2. Para actualizar el sistema, ejecute el siguiente comando:

    ```bash
    zypper update
    ```

Las acciones del comando [zypper](https://en.opensuse.org/Portal:Zypper), como la instalación y la eliminación de paquetes, se registran en el archivo de registro `/var/log/zypper.log`. 

## <a name="about-the-azcmagent-tool"></a>Acerca de la herramienta Azcmagent

La herramienta Azcmagent (Azcmagent.exe) se usa para configurar el agente Connected Machine de Azure Arc para servidores (versión preliminar) durante la instalación o bien para modificar la configuración inicial del agente después de la instalación. Azcmagent.exe proporciona parámetros de línea de comandos para personalizar el agente y ver su estado:

* **Connect**: para conectar la máquina a Azure Arc

* **Disconnect**: para desconectar la máquina de Azure Arc

* **Reconnect**: para volver a conectar una máquina desconectada a Azure Arc

* **Show**: para ver el estado del agente y sus propiedades de configuración (nombre del grupo de recursos, identificador de suscripción, versión, etc.), que puede ayudar a solucionar un problema con el agente.

* **-h o --help**: muestra los parámetros de línea de comandos disponibles

    Por ejemplo, si desea ver ayuda detallada para el parámetro **Reconnect**, escriba `azcmagent reconnect -h`. 

* **-v o --verbose**: habilita el registro detallado

Puede realizar una operación **Connect**, **Disconnect** y **Reconnect** manualmente mientras inicia sesión de forma interactiva, o bien utilizar la misma entidad de servicio que para incorporar varios agentes o con un [token de acceso](../../active-directory/develop/access-tokens.md) de la Plataforma de identidad de Microsoft. Si no usó una entidad de servicio para registrar la máquina con Azure Arc para servidores (versión preliminar), consulte el siguiente [artículo](onboard-service-principal.md#create-a-service-principal-for-onboarding-at-scale) para crear una entidad de servicio.

>[!NOTE]
>Debe tener permisos de acceso *raíz* en máquinas Linux para ejecutar **azcmagent**.

### <a name="connect"></a>Conectar

Este parámetro especifica un recurso en Azure Resource Manager que representa la máquina que se crea en Azure. El recurso está en la suscripción y en el grupo de recursos solicitado, y los datos sobre la máquina se almacenan en la región de Azure especificada por la opción `--location`. El nombre predeterminado del recurso es el nombre de host de esta máquina si no se especifica.

Un certificado correspondiente a la identidad asignada por el sistema de esta máquina se descarga entonces y se almacena localmente. Una vez que este paso se completa, el agente de configuración de invitados y Metadata Service de Azure Connected Machine comienzan a sincronizarse con Azure Arc para servidores (versión preliminar).

Para conectarse con una entidad de servicio, ejecute el siguiente comando:

`azcmagent connect --service-principal-id <serviceprincipalAppID> --service-principal-secret <serviceprincipalPassword> --tenant-id <tenantID> --subscription-id <subscriptionID> --resource-group <ResourceGroupName> --location <resourceLocation>`

Para conectarse mediante un token de acceso, ejecute el siguiente comando:

`azcmagent connect --access-token <> --subscription-id <subscriptionID> --resource-group <ResourceGroupName> --location <resourceLocation>`

Para conectarse con sus credenciales de sesión iniciada elevadas (interactiva), ejecute el siguiente comando:

`azcmagent connect --tenant-id <TenantID> --subscription-id <subscriptionID> --resource-group <ResourceGroupName> --location <resourceLocation>`

### <a name="disconnect"></a>Desconectar

Este parámetro especifica un recurso en Azure Resource Manager que representa la eliminación de la máquina en Azure. No elimina el agente de la máquina, esto debe hacerse como un paso independiente. Una vez que la máquina esté desconectada, si desea volver a registrarla con Azure Arc para servidores (versión preliminar), use `azcmagent connect` para que se cree un nuevo recurso para ella en Azure.

Para desconectarse con una entidad de servicio, ejecute el siguiente comando:

`azcmagent disconnect --service-principal-id <serviceprincipalAppID> --service-principal-secret <serviceprincipalPassword> --tenant-id <tenantID>`

Para desconectarse con un token de acceso, ejecute el siguiente comando:

`azcmagent disconnect --access-token <accessToken>`

Para desconectarse con sus credenciales de sesión iniciada elevadas (interactiva), ejecute el siguiente comando:

`azcmagent disconnect --tenant-id <tenantID>`

### <a name="reconnect"></a>Volver a conectar

> [!WARNING]
> El comando `reconnect` está en desuso y no se debe utilizar. El comando se quitará en una versión futura del agente y los agentes existentes no podrán completar la solicitud de reconexión. En su lugar, [desconecte](#disconnect) la máquina y vuelva a [conectarla](#connect).

Este parámetro vuelve a conectar la máquina ya registrada o conectada con Azure Arc para servidores (versión preliminar). Esto puede ser necesario si la máquina se ha desactivado, al menos 45 días, para que su certificado expire. Este parámetro usa las opciones de autenticación proporcionadas para recuperar las credenciales nuevas correspondientes al recurso de Azure Resource Manager que representa a esta máquina.

Este comando requiere privilegios más altos que el rol [incorporación de máquinas conectadas a Azure](agent-overview.md#required-permissions).

Para volver a conectar con una entidad de servicio, ejecute el siguiente comando:

`azcmagent reconnect --service-principal-id <serviceprincipalAppID> --service-principal-secret <serviceprincipalPassword> --tenant-id <tenantID>`

Para volver a conectarse mediante un token de acceso, ejecute el siguiente comando:

`azcmagent reconnect --access-token <accessToken>`

Para volver a conectar con sus credenciales de sesión iniciada elevadas (interactivo), ejecute el siguiente comando:

`azcmagent reconnect --tenant-id <tenantID>`

## <a name="remove-the-agent"></a>Eliminación del agente

Realice uno de los métodos siguientes para desinstalar el agente de Connected Machine de Windows o Linux desde el equipo. Al quitar el agente no se anula el registro de la máquina con Arc para los servidores (versión preliminar), se trata de un proceso independiente que se realiza cuando ya no es necesario administrar la máquina en Azure.

### <a name="windows-agent"></a>Agente de Windows

Los dos métodos siguientes quitan el agente, pero no quitan la carpeta *C:\Archivos de Files\AzureConnectedMachineAgent* de la máquina.

#### <a name="uninstall-from-control-panel"></a>Desinstalar desde el Panel de control

1. Para desinstalar el agente de Windows de la máquina, haga lo siguiente:

    a. Inicie sesión en el equipo con una cuenta que disponga de permisos de administrador.  
    b. En **Panel de control**, seleccione **Programas y características**.  
    c. En **Programas y características**, seleccione **Agente de Azure Connected Machine**, seleccione **Desinstalar** y, después, seleccione **Sí**.  

    >[!NOTE]
    > También puede ejecutar el asistente para la instalación del agente haciendo doble clic en el paquete del instalador **AzureConnectedMachineAgent.msi**.

#### <a name="uninstall-from-the-command-line"></a>Desinstalar desde la línea de comandos

Para desinstalar el agente manualmente desde el símbolo del sistema o para usar un método automatizado, como un script, puede usar el ejemplo siguiente. En primer lugar, debe recuperar el código del producto, que es un GUID que es el identificador principal del paquete de aplicación, del sistema operativo. La desinstalación se realiza mediante la línea de comandos msiexec.exe: `msiexec /x {Product Code}`.

1. Abra el Editor del Registro.

2. En la clave del Registro `HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Uninstall`, busque y copie el GUID del código del producto.

3. Después, puede desinstalar el agente mediante Msiexec con los ejemplos siguientes:

   * Desde la línea de comandos, escriba:

       ```dos
       msiexec.exe /x {product code GUID} /qn
       ```

   * Puede realizar los mismos pasos con PowerShell:

       ```powershell
       Get-ChildItem -Path HKLM:\Software\Microsoft\Windows\CurrentVersion\Uninstall | `
       Get-ItemProperty | `
       Where-Object {$_.DisplayName -eq "Azure Connected Machine Agent"} | `
       ForEach-Object {MsiExec.exe /x "$($_.PsChildName)" /qn}
       ```

### <a name="linux-agent"></a>Agente Linux

> [!NOTE]
> Para desinstalar el agente, debe tener permisos de acceso *raíz* o una cuenta que tenga derechos elevados que use Sudo.

Para desinstalar el agente de Linux, el comando que se va a usar depende del sistema operativo Linux.

- Para Ubuntu ejecute el siguiente comando:

    ```bash
    sudo apt purge azcmagent
    ```

- Para RHEL, CentOS y Amazon Linux, ejecute el siguiente comando:

    ```bash
    sudo yum remove azcmagent
    ```

- Para SLES, ejecute el siguiente comando:

    ```bash
    sudo zypper remove azcmagent
    ```

## <a name="unregister-machine"></a>Anulación del registro de la máquina

Si tiene previsto dejar de administrar la máquina con los servicios de soporte técnico de Azure, siga estos pasos para anular el registro de la máquina con Arc para servidores (versión preliminar). Puede realizar estos pasos antes o después de haber quitado el agente de Connected Machine de la máquina.

1. Abra Azure Arc para servidores (versión preliminar). Para ello, vaya a [Azure Portal](https://aka.ms/hybridmachineportal).

2. Seleccione la máquina en la lista, seleccione los puntos suspensivos ( **...** ) y, a continuación, seleccione **Eliminar**.

## <a name="update-or-remove-proxy-settings"></a>Actualización o eliminación de la configuración de proxy

Si quiere configurar el agente para comunicarse con el servicio a través de un servidor proxy o quitar esta configuración después de la implementación, use uno de los métodos siguientes para completar esta tarea.

### <a name="windows"></a>Windows

Para establecer la variable de entorno del servidor proxy, ejecute el siguiente comando:

```powershell
# If a proxy server is needed, execute these commands with the proxy URL and port.
[Environment]::SetEnvironmentVariable("https_proxy","http://{proxy-url}:{proxy-port}","Machine")
$env:https_proxy = [System.Environment]::GetEnvironmentVariable("https_proxy","Machine")
# For the changes to take effect, the agent service needs to be restarted after the proxy environment variable is set.
Restart-Service -Name himds
```

Para configurar el agente para que detenga la comunicación a través de un servidor proxy, ejecute el siguiente comando para quitar la variable de entorno del servidor proxy y reiniciar el servicio del agente:

```powershell
[Environment]::SetEnvironmentVariable("https_proxy",$null,"Machine")
$env:https_proxy = [System.Environment]::GetEnvironmentVariable("https_proxy","Machine")
# For the changes to take effect, the agent service needs to be restarted after the proxy environment variable removed.
Restart-Service -Name himds
```

### <a name="linux"></a>Linux

Para establecer el servidor proxy, ejecute el siguiente comando desde el directorio en el que descargó el paquete de instalación del agente:

```bash
# Reconfigure the connected machine agent and set the proxy server.
bash ~/Install_linux_azcmagent.sh --proxy "{proxy-url}:{proxy-port}"
```

Para configurar el agente para que detenga la comunicación a través de un servidor proxy, ejecute el siguiente comando para quitar la configuración de proxy:

```bash
sudo azcmagent_proxy remove
```

## <a name="next-steps"></a>Pasos siguientes

- Obtenga información sobre cómo administrar la máquina con [Azure Policy](../../governance/policy/overview.md) para, por ejemplo, la [configuración de invitado](../../governance/policy/concepts/guest-configuration.md) de VM, la comprobación de que la máquina informa al área de trabajo de Log Analytics esperada, la habilitación de la supervisión con [Azure Monitor con máquinas virtuales](../../azure-monitor/insights/vminsights-enable-policy.md) y mucho más.

- Más información sobre el [agente de Log Analytics](../../azure-monitor/platform/log-analytics-agent.md). El agente de Log Analytics para Windows y Linux es necesario si desea supervisar de forma proactiva el sistema operativo y las cargas de trabajo que se ejecutan en la máquina, administrarlos mediante runbooks de Automation o características como Update Management, o bien usar otros servicios de Azure como [Azure Security Center](../../security-center/security-center-intro.md).