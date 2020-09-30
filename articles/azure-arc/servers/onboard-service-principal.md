---
title: Conexión de máquinas híbridas a Azure a gran escala
description: En este artículo, obtendrá información sobre cómo conectar máquinas a Azure mediante servidores habilitados para Azure Arc con una entidad de servicio.
ms.date: 09/02/2020
ms.topic: conceptual
ms.openlocfilehash: 545d8abd6dd17e1e413852735c096ddc9261b972
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90908327"
---
# <a name="connect-hybrid-machines-to-azure-at-scale"></a>Conexión de máquinas híbridas a Azure a gran escala

Puede activar los servidores habilitados para Azure Arc para varias máquinas Windows o Linux en su entorno, con varias opciones flexibles, en función de sus requisitos. Con el script de plantilla que proporcionamos, puede automatizar cada paso de la instalación, incluido el establecimiento de la conexión a Azure Arc. Sin embargo, es necesario ejecutar este script de forma interactiva con una cuenta que tenga permisos elevados en la máquina de destino y en Azure. Para conectar las máquinas a los servidores habilitados para Azure Arc, puede usar una [entidad de servicio](../../active-directory/develop/app-objects-and-service-principals.md) de Azure Active Directory en lugar de usar su identidad con privilegios para [conectar interactivamente la máquina](onboard-portal.md). Una entidad de servicio es una identidad de administración limitada especial a la que solo se concede el permiso mínimo necesario para conectar máquinas a Azure con el comando `azcmagent`. Es más seguro que usar una cuenta con más privilegios, como un Administrador de inquilinos, y sigue nuestros procedimientos recomendados de seguridad de control de acceso. La entidad de servicio se usa solo durante la incorporación; no se usa para ningún otro propósito.  

Los métodos de instalación para instalar y configurar el agente Connected Machine requieren que el método automatizado que se use tenga permisos de administrador en las máquinas. En Linux, mediante la cuenta raíz y, en Windows, como miembro del grupo local de administradores.

Antes de comenzar, asegúrese de revisar los [requisitos previos](agent-overview.md#prerequisites) y compruebe que su suscripción y sus recursos los cumplen.

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

Al final de este proceso, habrá conectado correctamente las máquinas híbridas a los servidores habilitados para Azure Arc.

## <a name="create-a-service-principal-for-onboarding-at-scale"></a>Creación de una entidad de servicio para la incorporación a gran escala

Puede usar [Azure PowerShell](/powershell/azure/install-az-ps) para crear una entidad de servicio con el cmdlet [New-AzADServicePrincipal](/powershell/module/Az.Resources/New-AzADServicePrincipal). O bien, puede seguir los pasos que se indican en [Creación de una entidad de servicio con Azure Portal](../../active-directory/develop/howto-create-service-principal-portal.md) para completar esta tarea.

> [!NOTE]
> Al crear una entidad de servicio, la cuenta debe ser Propietario o Administrador de acceso de los usuarios en la suscripción que se va a usar para la incorporación. Si no tiene permisos suficientes para crear asignaciones de roles, es posible que se cree la entidad de servicio, pero no podrá incorporar máquinas.
>

Para crear la entidad de servicio mediante PowerShell, realice lo siguiente.

1. Ejecute el siguiente comando: Debe almacenar la salida del cmdlet [`New-AzADServicePrincipal`](/powershell/module/az.resources/new-azadserviceprincipal) en una variable o no podrá recuperar la contraseña necesaria en un paso posterior.

    ```azurepowershell-interactive
    $sp = New-AzADServicePrincipal -DisplayName "Arc-for-servers" -Role "Azure Connected Machine Onboarding"
    $sp
    ```

    ```output
    Secret                : System.Security.SecureString
    ServicePrincipalNames : {ad9bcd79-be9c-45ab-abd8-80ca1654a7d1, https://Arc-for-servers}
    ApplicationId         : ad9bcd79-be9c-45ab-abd8-80ca1654a7d1
    ObjectType            : ServicePrincipal
    DisplayName           : Hybrid-RP
    Id                    : 5be92c87-01c4-42f5-bade-c1c10af87758
    Type                  :
    ```

2. Para recuperar la contraseña almacenada en la variable `$sp`, ejecute el siguiente comando:

    ```azurepowershell-interactive
    $credential = New-Object pscredential -ArgumentList "temp", $sp.Secret
    $credential.GetNetworkCredential().password
    ```

3. En la salida, busque el valor de la contraseña en el campo **password** y cópiela. Busque también el valor del campo **ApplicationId** y cópielo también. Guárdelos para más adelante en un lugar seguro. Si olvida o pierde la contraseña de la entidad de servicio, puede restablecerla mediante el cmdlet [`New-AzADSpCredential`](/powershell/module/azurerm.resources/new-azurermadspcredential).

Los valores de las siguientes propiedades se usan con los parámetros que se pasan a `azcmagent`:

* El valor de la propiedad **ApplicationId** se usa para el valor del parámetro `--service-principal-id`.
* El valor de la propiedad **password** se usa para el parámetro `--service-principal-secret` que se usa para conectar el agente.

> [!NOTE]
> Asegúrese de usar la propiedad **ApplicationId** de la entidad de servicio, no la propiedad **Id**.
>

El rol **Incorporación de Azure Connected Machine** contiene solo los permisos necesarios para incorporar una máquina. Puede asignar a la entidad de servicio el permiso para permitir que su ámbito incluya un grupo de recursos o una suscripción. Para agregar una asignación de roles, consulte [Incorporación o eliminación de asignaciones de roles con RBAC de Azure y Azure Portal](../../role-based-access-control/role-assignments-portal.md) o [Incorporación o eliminación de asignaciones de roles con RBAC de Azure y la CLI de Azure](../../role-based-access-control/role-assignments-cli.md).

## <a name="install-the-agent-and-connect-to-azure"></a>Instalación del agente y conexión a Azure

En los pasos siguientes se instala y configura el agente Connected Machine en las máquinas híbridas mediante la plantilla de script, que realiza pasos similares a los que se describen en el artículo [Conexión de máquinas híbridas a Azure desde el Azure Portal](onboard-portal.md). La diferencia radica en el paso final en el que se establece la conexión a Azure Arc con el comando `azcmagent` mediante la entidad de servicio.

A continuación, se muestran los valores que se configuran en el comando `azcmagent` que se va a usar para la entidad de servicio.

* `tenant-id`: el identificador único (GUID) que representa la instancia dedicada de Azure AD.
* `subscription-id`: el identificador de suscripción (GUID) de la suscripción de Azure en la que quiere que estén las máquinas.
* `resource-group`: el nombre del grupo de recursos al que desea que pertenezcan las máquinas conectadas.
* `location`: consulte [Regiones de Azure compatibles](overview.md#supported-regions). Esta ubicación puede ser la misma u otra diferente que la del grupo de recursos.
* `resource-name`: (*opcional*) Se usa para la representación de recursos de Azure de la máquina local. Si no especifica este valor, se usa el nombre de host de la máquina.

Para obtener más información sobre la herramienta de línea de comandos `azcmagent`, revise la [referencia de Azcmagent](./manage-agent.md).

### <a name="windows-installation-script"></a>Script de instalación en Windows

A continuación, se ofrece un ejemplo del script de instalación del agente Connected Machine para Windows que se ha modificado para usar la entidad de servicio y admitir una instalación totalmente automatizada, no interactiva, del agente.

```
 # Download the package
function download() {$ProgressPreference="SilentlyContinue"; Invoke-WebRequest -Uri https://aka.ms/AzureConnectedMachineAgent -OutFile AzureConnectedMachineAgent.msi}
download

# Install the package
msiexec /i AzureConnectedMachineAgent.msi /l*v installationlog.txt /qn | Out-String

# Run connect command
& "$env:ProgramFiles\AzureConnectedMachineAgent\azcmagent.exe" connect `
  --service-principal-id "{serviceprincipalAppID}" `
  --service-principal-secret "{serviceprincipalPassword}" `
  --resource-group "{ResourceGroupName}" `
  --tenant-id "{tenantID}" `
  --location "{resourceLocation}" `
  --subscription-id "{subscriptionID}"
```

>[!NOTE]
>El script solo admite la ejecución desde una versión de 64 bits de Windows PowerShell.
>

### <a name="linux-installation-script"></a>Script de instalación en Linux

A continuación, se ofrece un ejemplo del script de instalación del agente Connected Machine para Linux que se ha modificado para usar la entidad de servicio para admitir una instalación totalmente automatizada, no interactiva, del agente.

```
# Download the installation package
wget https://aka.ms/azcmagent -O ~/install_linux_azcmagent.sh

# Install the hybrid agent
bash ~/install_linux_azcmagent.sh

# Run connect command
azcmagent connect \
  --service-principal-id "{serviceprincipalAppID}" \
  --service-principal-secret "{serviceprincipalPassword}" \
  --resource-group "{ResourceGroupName}" \
  --tenant-id "{tenantID}" \
  --location "{resourceLocation}" \
  --subscription-id "{subscriptionID}"
```

>[!NOTE]
>Debe tener permisos de acceso *raíz* en máquinas Linux para ejecutar **azcmagent**.

Después de instalar el agente y configurarlo para que se conecte a los servidores habilitados para Azure Arc, vaya a Azure Portal a fin de comprobar que el servidor se ha conectado correctamente. Vea las máquinas en [Azure Portal](https://aka.ms/hybridmachineportal).

![Una conexión de servidor correcta](./media/onboard-portal/arc-for-servers-successful-onboard.png)

## <a name="next-steps"></a>Pasos siguientes

- Obtenga información sobre cómo administrar la máquina con [Azure Policy](../../governance/policy/overview.md) para, por ejemplo, la [configuración de invitado](../../governance/policy/concepts/guest-configuration.md) de VM, la comprobación de que la máquina informa al área de trabajo de Log Analytics esperada, la habilitación de la supervisión con [Azure Monitor con máquinas virtuales](../../azure-monitor/insights/vminsights-enable-policy.md) y mucho más.

- Más información sobre el [[agente de Log Analytics]](../../azure-monitor/platform/log-analytics-agent.md). El agente de Log Analytics para Windows y Linux es necesario si quiere recopilar datos de supervisión del sistema operativo y de las cargas de trabajo y administrarlos con runbooks de Automation o con características como Update Management, o bien mediante otros servicios de Azure, como [Azure Security Center](../../security-center/security-center-intro.md).