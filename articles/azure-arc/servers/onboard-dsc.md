---
title: Instalación del agente de Connected Machine mediante DSC de Windows PowerShell
description: En este artículo, aprenderá a conectar máquinas a Azure mediante Azure Arc para servidores (versión preliminar) usando DSC de Windows PowerShell.
ms.date: 03/12/2020
ms.topic: conceptual
ms.openlocfilehash: cdda3e6681d3e6912d031c45f5c6da9e92814e8f
ms.sourcegitcommit: b8702065338fc1ed81bfed082650b5b58234a702
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/11/2020
ms.locfileid: "88121007"
---
# <a name="how-to-install-the-connected-machine-agent-using-windows-powershell-dsc"></a>Instalación del agente de Connected Machine mediante DSC de Windows PowerShell

Con [Introducción a la configuración de estado deseado (DSC) para Windows](/powershell/scripting/dsc/getting-started/winGettingStarted?view=powershell-7), puede automatizar la instalación y configuración de software para un equipo Windows. En este artículo se describe cómo usar DSC para instalar el agente de Connected Machine de Azure Arc para servidores en máquinas Windows híbridas.

## <a name="requirements"></a>Requisitos

- Windows PowerShell versión 4.0 o posterior

- El módulo de DSC [AzureConnectedMachineDsc](https://www.powershellgallery.com/packages/AzureConnectedMachineDsc/1.0.1.0)

- Una entidad de servicio para conectar las máquinas a Azure Arc para servidores de forma no interactiva. Siga los pasos descritos en la sección [Creación de una entidad de servicio para la incorporación a escala](onboard-service-principal.md#create-a-service-principal-for-onboarding-at-scale) si aún no ha creado una entidad de servicio para Arc para servidores.

## <a name="install-the-connectedmachine-dsc-module"></a>Instalación del módulo DSC ConnectedMachine

1. Para instalar manualmente el módulo, descargue el código fuente y descomprima el contenido del directorio del proyecto en la `$env:ProgramFiles\WindowsPowerShell\Modules folder`. O bien, ejecute el siguiente comando para realizar la instalación desde la galería de PowerShell mediante PowerShellGet (en PowerShell 5.0):

    ```powershell
    Find-Module -Name AzureConnectedMachineDsc -Repository PSGallery | Install-Module
    ```

2. Para confirmar la instalación, ejecute el comando siguiente y asegúrese de que ve los recursos de DSC de Azure Connected Machine disponibles.

    ```powershell
    Get-DscResource -Module AzureConnectedMachineDsc
    ```

   Debería ver algo parecido a lo siguiente:

   ![Ejemplo de instalación del módulo de DSC de Connected Machine](./media/onboard-dsc/confirm-module-installation.png)

## <a name="install-the-agent-and-connect-to-azure"></a>Instalación del agente y conexión a Azure

Los recursos de este módulo están diseñados para administrar la configuración del agente de Azure Connected Machine. También se incluye un script de PowerShell `AzureConnectedMachineAgent.ps1`, que se encuentra en la carpeta `AzureConnectedMachineDsc\examples`. Utiliza recursos de la comunidad para automatizar la descarga e instalación, y establecer una conexión con Azure Arc. Este script realiza pasos similares que se describen en el artículo [Conexión de máquinas híbridas a Azure desde Azure Portal](onboard-portal.md).

Si la máquina necesita comunicarse mediante un servidor proxy con el servicio, después de instalar el agente, debe ejecutar un comando que se describe [aquí](manage-agent.md#update-or-remove-proxy-settings). Este permite establecer la variable de entorno del sistema del servidor proxy `https_proxy`. En lugar de ejecutar el comando manualmente, puede realizar este paso con DSC mediante el módulo [ComputeManagementDsc](https://www.powershellgallery.com/packages/ComputerManagementDsc/6.0.0.0).

>[!NOTE]
>Para permitir la ejecución de DSC, Windows debe configurarse para recibir comandos remotos de PowerShell incluso cuando se ejecuta una configuración de localhost. Para configurar correctamente el entorno, solo tiene que ejecutar `Set-WsManQuickConfig -Force` en un terminal de PowerShell con privilegios elevados.
>

Los documentos de configuración (archivos MOF) se pueden aplicar a la máquina mediante el cmdlet `Start-DscConfiguration`.

A continuación se muestran los parámetros que se pasan al script de PowerShell que se va a usar.

- `TenantId`: el identificador único (GUID) que representa la instancia dedicada de Azure AD.

- `SubscriptionId`: el identificador de suscripción (GUID) de la suscripción de Azure en la que quiere que estén las máquinas.

- `ResourceGroup`: el nombre del grupo de recursos al que desea que pertenezcan las máquinas conectadas.

- `Location`: consulte [Regiones de Azure compatibles](overview.md#supported-regions). Esta ubicación puede ser la misma u otra diferente que la del grupo de recursos.

- `Tags`: Matriz de cadenas de etiquetas que se deben aplicar al recurso de máquina conectada.

- `Credential`: un objeto de credencial de PowerShell con el valor de **ApplicationId** y de **contraseña** usado para registrar máquinas a escala mediante una [entidad de servicio](onboard-service-principal.md). 

1. En una consola de PowerShell, vaya a la carpeta donde guardó el archivo `.ps1`.

2. Ejecute los siguientes comandos de PowerShell para compilar el documento MOF (para obtener información sobre la compilación de configuraciones de DSC, consulte [Configuraciones de DSC](/powershell/scripting/dsc/configurations/configurations?view=powershell-7):

    ```powershell
    .\`AzureConnectedMachineAgent.ps1 -TenantId <TenantId GUID> -SubscriptionId <SubscriptionId GUID> -ResourceGroup '<ResourceGroupName>' -Location '<LocationName>' -Tags '<Tag>' -Credential <psCredential>
    ```

3. De esta forma creará un `localhost.mof file` en una carpeta nueva denominada `C:\dsc`.

Después de instalar el agente y configurarlo para que se conecte a Azure Arc para servidores (versión preliminar), vaya a Azure Portal para comprobar que el servidor se ha conectado correctamente. Vea las máquinas en [Azure Portal](https://aka.ms/hybridmachineportal).

## <a name="adding-to-existing-configurations"></a>Incorporación a configuraciones existentes

Este recurso se puede agregar a configuraciones de DSC existentes para representar una configuración de un extremo a otro para una máquina. Por ejemplo, puede que desee agregar este recurso a una configuración que establezca una configuración de sistema operativo segura.

El módulo [CompsiteResource](https://www.powershellgallery.com/packages/compositeresource/0.4.0) de la Galería de PowerShell se puede usar para crear un [recurso compuesto](/powershell/scripting/dsc/resources/authoringResourceComposite?view=powershell-7) de la configuración de ejemplo, a fin de simplificar aún más la combinación de configuraciones.

## <a name="next-steps"></a>Pasos siguientes

- Obtenga información sobre cómo administrar la máquina con [Azure Policy](../../governance/policy/overview.md) para, por ejemplo, la [configuración de invitado](../../governance/policy/concepts/guest-configuration.md) de VM, la comprobación de que la máquina informa al área de trabajo de Log Analytics esperada, la habilitación de la supervisión con [Azure Monitor con máquinas virtuales](../../azure-monitor/insights/vminsights-enable-policy.md) y mucho más.

- Más información sobre el [agente de Log Analytics](../../azure-monitor/platform/log-analytics-agent.md). El agente de Log Analytics para Windows y Linux es necesario si desea supervisar de forma proactiva el sistema operativo y las cargas de trabajo que se ejecutan en la máquina, administrarlos mediante runbooks de Automation o soluciones como Update Management, o usar otros servicios de Azure como [Azure Security Center](../../security-center/security-center-intro.md).