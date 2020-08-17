---
title: Administración de credenciales en Azure Automation
description: En este artículo se explica cómo crear recursos de credenciales y usarlos en un runbook o una configuración de DSC.
services: automation
ms.service: automation
ms.subservice: shared-capabilities
author: mgoedtel
ms.author: magoedte
ms.date: 01/31/2020
ms.topic: conceptual
manager: carmonm
ms.openlocfilehash: 6b95eeaf7dd72c85c3940e3cdc2a71c193c35ff5
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87008615"
---
# <a name="manage-credentials-in-azure-automation"></a>Administración de credenciales en Azure Automation

Un recurso de credencial de Automation incluye un objeto que contiene credenciales de seguridad, como un nombre de usuario y una contraseña. Los runbooks y las configuraciones de DSC usan cmdlets que aceptan un objeto [PSCredential](/dotnet/api/system.management.automation.pscredential?view=pscore-6.2.0) para la autenticación. Como alternativa, pueden extraer el nombre de usuario y la contraseña del objeto `PSCredential` para proporcionarlos a alguna aplicación o servicio que requiera autenticación. 

>[!NOTE]
>Los recursos protegidos en Azure Automation incluyen credenciales, certificados, conexiones y variables cifradas. Estos recursos se cifran y se almacenan en Azure Automation con una clave única que se genera para cada cuenta de Automation. Azure Automation almacena la clave en la instancia de Key Vault administrada por el sistema. Antes de almacenar un recurso seguro, Azure Automation carga la clave desde Key Vault y después la usa para cifrar el recurso. 

[!INCLUDE [gdpr-dsr-and-stp-note.md](../../../includes/gdpr-dsr-and-stp-note.md)]

## <a name="powershell-cmdlets-used-to-access-credentials"></a>Cmdlets de PowerShell usados para acceder a credenciales

Los cmdlets de la tabla siguiente permiten crear y administrar credenciales de Automation con PowerShell. Se suministran como un componente de los [módulos Az](modules.md#az-modules).

| Cmdlet | Descripción |
|:--- |:--- |
| [Get-AzAutomationCredential](/powershell/module/az.automation/get-azautomationcredential?view=azps-3.3.0) |Recupera un objeto [CredentialInfo](/dotnet/api/microsoft.azure.commands.automation.model.credentialinfo?view=azurerm-ps) que contiene metadatos sobre la credencial. El cmdlet no recupera el objeto `PSCredential` mismo.  |
| [New-AzAutomationCredential](/powershell/module/az.automation/new-azautomationcredential?view=azps-3.3.0) |Crea una nueva credencial de Automation. |
| [Remove-AzAutomationCredential](/powershell/module/az.automation/remove-azautomationcredential?view=azps-3.3.0) |Quita una credencial de Automation. |
| [Set-AzAutomationCredential](/powershell/module/az.automation/set-azautomationcredential?view=azps-3.3.0) |Establece las propiedades para una credencial de Automatización existente. |

## <a name="other-cmdlets-used-to-access-credentials"></a>Otros cmdlets usados para acceder a credenciales

Los cmdlets de la tabla siguiente se usan para acceder a las credenciales en los runbook y las configuraciones de DSC. 

| Cmdlet | Descripción |
|:--- |:--- |
| `Get-AutomationPSCredential` |Obtiene un objeto `PSCredential` para usarlo en un runbook o una configuración de DSC. La mayoría de las veces debe usar este [cmdlet interno](modules.md#internal-cmdlets) en lugar del cmdlet `Get-AzAutomationCredential`, ya que este último solo recupera la información de credenciales. Esta información no suele ser útil para pasar a otro cmdlet. |
| [Get-Credential](/powershell/module/microsoft.powershell.security/get-credential?view=powershell-7) |Obtiene una credencia con una solicitud de nombre de usuario y contraseña. Este cmdlet forma parte del módulo Microsoft.PowerShell.Security predeterminado. Consulte [Módulos predeterminados](modules.md#default-modules).|
| [New-AzureAutomationCredential](/powershell/module/servicemanagement/azure.service/new-azureautomationcredential?view=azuresmps-4.0.0) | Crea un recurso de credencial. Este cmdlet forma parte del módulo de Azure predeterminado. Consulte [Módulos predeterminados](modules.md#default-modules).|

Para recuperar objetos `PSCredential` en el código, debe importar el módulo `Orchestrator.AssetManagement.Cmdlets`. Para más información, consulte [Administración de módulos en Azure Automation](modules.md).

```azurepowershell
Import-Module Orchestrator.AssetManagement.Cmdlets -ErrorAction SilentlyContinue
```

> [!NOTE]
> Debe evitar el uso de variables en el parámetro `Name` de `Get-AutomationPSCredential`. Su uso puede complicar la detección de dependencias entre runbooks o configuraciones de DSC y recursos de credenciales durante el tiempo de diseño.

## <a name="python-2-functions-that-access-credentials"></a>Funciones de Python 2 que tienen acceso a las credenciales

La función de la tabla siguiente se usa para obtener acceso a las credenciales de un runbook de Python 2.

| Función | Descripción |
|:---|:---|
| `automationassets.get_automation_credential` | Recupera información acerca de un recurso de credencial. |

> [!NOTE]
> Para acceder a las funciones del recurso, importe el módulo `automationassets` en la parte superior del runbook de Python.

## <a name="create-a-new-credential-asset"></a>Creación de un nuevo recurso de credencial

Puede crear un recurso de credencial mediante Azure Portal o Windows PowerShell.

### <a name="create-a-new-credential-asset-with-the-azure-portal"></a>Creación de un nuevo recurso de credencial con Azure Portal

1. En la cuenta de Automation, seleccione **Credenciales** en **Recursos compartidos**.
1. Seleccione **Agregar una credencial**.
2. En el panel Nueva credencial, escriba un nombre de credencial adecuado que siga los estándares de nomenclatura. 
3. Escriba el identificador de acceso en el campo **Nombre de usuario**. 
4. En ambos campos de contraseña, escriba la clave de acceso secreta.

    ![Creación de una credencial](../media/credentials/credential-create.png)

5. Si la casilla de autenticación multifactor está activada, desactívela. 
6. Haga clic en **Crear** para guardar el nuevo recurso de credencial.

> [!NOTE]
> Azure Automation no admite cuentas de usuario que usen la autenticación multifactor.

### <a name="create-a-new-credential-asset-with-windows-powershell"></a>Creación de un nuevo recurso de credencial con Windows PowerShell

En el ejemplo siguiente se muestra cómo crear un recurso de credencial de Automation. Primero se crea un objeto `PSCredential` con el nombre y la contraseña y, luego, se usa para crear el recurso de credencial. En su lugar, puede usar el cmdlet `Get-Credential` para solicitar al usuario que escriba un nombre y una contraseña.

```powershell
$user = "MyDomain\MyUser"
$pw = ConvertTo-SecureString "PassWord!" -AsPlainText -Force
$cred = New-Object –TypeName System.Management.Automation.PSCredential –ArgumentList $user, $pw
New-AzureAutomationCredential -AutomationAccountName "MyAutomationAccount" -Name "MyCredential" -Value $cred
```

## <a name="get-a-credential-asset"></a>Obtención de un recurso de credencial

Un runbook o una configuración de DSC recupera un recurso de credencial con el cmdlet `Get-AutomationPSCredential` interno. Este cmdlet recupera un objeto `PSCredential` que puede usar con un cmdlet que requiere una credencial. También puede recuperar las propiedades del objeto de credencial para usarlas individualmente. El objeto tiene propiedades para el nombre de usuario y la contraseña segura. 

> [!NOTE]
> El cmdlet `Get-AzAutomationCredential` no recupera ningún objeto `PSCredential` que se pueda usar para la autenticación. Solo proporciona información acerca de la credencial. Si necesita usar una credencial en un runbook, debe recuperarla como un objeto `PSCredential` mediante `Get-AutomationPSCredential`.

Como alternativa, puede usar el método [GetNetworkCredential](/dotnet/api/system.management.automation.pscredential.getnetworkcredential?view=pscore-6.2.0) para recuperar un objeto [NetworkCredential](/dotnet/api/system.net.networkcredential) que representa una versión no segura de la contraseña.

### <a name="textual-runbook-example"></a>Ejemplo de un runbook textual

En el ejemplo siguiente se muestra cómo usar una credencial de PowerShell en un runbook. Se recupera la credencial y se asigna su nombre de usuario y contraseña a las variables.


```azurepowershell
$myCredential = Get-AutomationPSCredential -Name 'MyCredential'
$userName = $myCredential.UserName
$securePassword = $myCredential.Password
$password = $myCredential.GetNetworkCredential().Password
```

También puede usar una credencial para autenticarse en Azure con [Connect-AzAccount](/powershell/module/az.accounts/connect-azaccount?view=azps-3.3.0). En la mayoría de las circunstancias, deberá usar una [cuenta de ejecución](../manage-runas-account.md) y recuperar la conexión con [Get-AzAutomationConnection](../automation-connections.md).


```azurepowershell
$myCred = Get-AutomationPSCredential -Name 'MyCredential'
$userName = $myCred.UserName
$securePassword = $myCred.Password
$password = $myCred.GetNetworkCredential().Password

$myPsCred = New-Object System.Management.Automation.PSCredential ($userName,$password)

Connect-AzAccount -Credential $myPsCred
```

### <a name="graphical-runbook-example"></a>Ejemplos de un runbook gráfico

Para agregar una actividad para el cmdlet `Get-AutomationPSCredential` interno a un runbook gráfico, haga clic con el botón derecho en la credencial en el panel Biblioteca del editor gráfico y, después, seleccione **Agregar al lienzo**.

![Agregar credencial a lienzo](../media/credentials/credential-add-canvas.png)

La imagen siguiente muestra un ejemplo de cómo usar una credencial en un runbook gráfico. En este caso, la credencial proporciona la autenticación de un runbook en los recursos de Azure, como se describe en [Uso de Azure AD en Azure Automation para autenticarse en Azure](../automation-use-azure-ad.md). La primera actividad recupera la credencial que tiene acceso a la suscripción de Azure. A continuación, la actividad de conexión de la cuenta usa esta credencial para proporcionar autenticación para cualquier actividad que venga después. Aquí se usa un [vínculo de canalización](../automation-graphical-authoring-intro.md#use-links-for-workflow), debido a que `Get-AutomationPSCredential` espera un solo objeto.  

![Agregar credencial a lienzo](../media/credentials/get-credential.png)

## <a name="use-credentials-in-a-dsc-configuration"></a>Uso de credenciales en una configuración de DSC

Aunque las configuraciones de DSC en Azure Automation pueden funcionar con recursos de credenciales mediante `Get-AutomationPSCredential`, también los pueden pasar a través de parámetros. Para obtener más información, consulte [Compilación de configuraciones en DSC de Azure Automation](../automation-dsc-compile.md#credential-assets).

## <a name="use-credentials-in-a-python-2-runbook"></a>Uso de credenciales en un runbook de Python 2

En el ejemplo siguiente se muestra un ejemplo de acceso a credenciales en runbooks de Python 2.


```python
import automationassets
from automationassets import AutomationAssetNotFound

# get a credential
cred = automationassets.get_automation_credential("credtest")
print cred["username"]
print cred["password"]
```

## <a name="next-steps"></a>Pasos siguientes

* Para más información sobre los cmdlets que se usan para acceder a los certificados, consulte [Administración de módulos en Azure Automation](modules.md).
* Para información general sobre los runbooks, consulte [Ejecución de un runbook en Azure Automation](../automation-runbook-execution.md).
* Para obtener más información sobre las configuraciones DSC, consulte [Introducción a State Configuration](../automation-dsc-overview.md). 
