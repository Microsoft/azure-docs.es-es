---
title: Administración de certificados en Azure Automation
description: En este artículo se explica cómo trabajar con certificados para el acceso mediante runbooks y configuraciones DSC.
services: automation
ms.service: automation
ms.subservice: shared-capabilities
author: mgoedtel
ms.author: magoedte
ms.date: 04/02/2019
ms.topic: conceptual
manager: carmonm
ms.openlocfilehash: 61e2cdf63e5553ba8d796115284dad9a538c2b81
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87056287"
---
# <a name="manage-certificates-in-azure-automation"></a>Administración de certificados en Azure Automation

Azure Automation almacena los certificados de forma segura para que los runbooks o las configuraciones DSC puedan acceder a ellos mediante el cmdlet [Get-AzAutomationCertificate](/powershell/module/Az.Automation/Get-AzAutomationCertificate?view=azps-3.7.0) para recursos de Azure Resource Manager. El almacenamiento seguro de certificados permite crear runbooks y configuraciones DSC que usan certificados para autenticación, o agregarlos a Azure o a recursos de terceros.

>[!NOTE]
>Los recursos protegidos en Azure Automation incluyen credenciales, certificados, conexiones y variables cifradas. Estos recursos se cifran y se almacenan en Automation con una clave única que se genera para cada cuenta de Automation. Automation almacena la clave en el servicio de Key Vault administrado por el sistema. Antes de almacenar un recurso seguro, Automation carga la clave desde Key Vault y después la usa para cifrar el recurso. 

## <a name="powershell-cmdlets-to-access-certificates"></a>Cmdlets de PowerShell para acceder a los certificados

Los cmdlets de la tabla siguiente permiten crear y administrar certificados de Automation con PowerShell. Se suministran como un componente de los [módulos Az](modules.md#az-modules).

|Cmdlet |Descripción|
| --- | ---|
|[Get-AzAutomationCertificate](/powershell/module/Az.Automation/Get-AzAutomationCertificate?view=azps-3.7.0)|Recupera información sobre un certificado para utilizarlo en un runbook o en la configuración de DSC. Solo puede recuperar el certificado mediante el cmdlet `Get-AutomationCertificate` interno.|
|[New-AzAutomationCertificate](/powershell/module/Az.Automation/New-AzAutomationCertificate?view=azps-3.7.0)|Crea un certificado nuevo en Automation.|
|[Remove-AzAutomationCertificate](/powershell/module/Az.Automation/Remove-AzAutomationCertificate?view=azps-3.7.0)|Quita un certificado de Automation.|
|[Set-AzAutomationCertificate](/powershell/module/Az.Automation/Set-AzAutomationCertificate?view=azps-3.7.0)|Establece las propiedades de un certificado existente, incluidos la carga del archivo de certificado y el establecimiento de la contraseña para un archivo **.pfx**.|

También se puede usar el cmdlet [Add-AzureCertificate](/powershell/module/servicemanagement/azure.service/add-azurecertificate) para cargar un certificado de servicio para el servicio en la nube especificado.

## <a name="internal-cmdlets-to-access-certificates"></a>Cmdlets internos para acceder a los certificados

El cmdlet interno de la tabla siguiente se usa para acceder a los certificados de los runbooks. Este cmdlet se incluye en el módulo global `Orchestrator.AssetManagement.Cmdlets`. Para obtener más información, consulte [Cmdlets internos](modules.md#internal-cmdlets).

| Cmdlet interno | Descripción |
|:---|:---|
|`Get-AutomationCertificate`|Obtiene un certificado para usarlo en un runbook o una configuración de DSC. Devuelve un objeto [System.Security.Cryptography.X509Certificates.X509Certificate2](/dotnet/api/system.security.cryptography.x509certificates.x509certificate2).|

> [!NOTE] 
> Debe evitar el uso de variables en el parámetro `Name` de `Get-AutomationCertificate` en un runbook o configuración de DSC. Estas variables pueden complicar la detección de dependencias entre runbooks o configuraciones DSC y variables de Automation durante el tiempo de diseño.

## <a name="python-2-functions-to-access-certificates"></a>Funciones de Python 2 para acceder a los certificados

La función de la tabla siguiente se usa para acceder a los certificados de un runbook de Python 2.

| Función | Descripción |
|:---|:---|
| `automationassets.get_automation_certificate` | Recupera información de un recurso de certificado. |

> [!NOTE]
> Para acceder a las funciones del recurso, debe importar el módulo `automationassets` al principio del runbook de Python.

## <a name="create-a-new-certificate"></a>Crear un nuevo certificado

Cuando crea un certificado nuevo, debe cargar un archivo .cer o .pfx a Automation. Si marca el certificado como exportable, podrá transferirlo fuera del almacén de certificados de Automation. Si no es exportable, solo se puede usar para firmar dentro del runbook o la configuración de DSC. Automation requiere que el certificado tenga el **proveedor de servicios criptográficos AES y RSA mejorado de Microsoft**.

### <a name="create-a-new-certificate-with-the-azure-portal"></a>Creación de un certificado nuevo con Azure Portal

1. En la cuenta de Automation, seleccione **Recursos** > **Certificados** > **Agregar un certificado**.
1. Escriba un nombre para el certificado en el campo **Nombre**.
1. Para buscar un archivo **.cer** o **.pfx**, elija **Seleccionar un archivo** en **Cargar un archivo de certificado**. Si selecciona un archivo **.pfx**, especifique una contraseña e indique si se puede exportar.
1. Seleccione **Crear** para guardar el recurso de certificado nuevo.

### <a name="create-a-new-certificate-with-powershell"></a>Creación de un certificado nuevo con PowerShell

En el ejemplo siguiente se muestra cómo crear un nuevo certificado de Automation y marcarlo como exportable. En este ejemplo se importa un archivo **.pfx** existente.

```powershell-interactive
$certificateName = 'MyCertificate'
$PfxCertPath = '.\MyCert.pfx'
$CertificatePassword = ConvertTo-SecureString -String 'P@$$w0rd' -AsPlainText -Force
$ResourceGroup = "ResourceGroup01"

New-AzAutomationCertificate -AutomationAccountName "MyAutomationAccount" -Name $certificateName -Path $PfxCertPath –Password $CertificatePassword -Exportable -ResourceGroupName $ResourceGroup
```

### <a name="create-a-new-certificate-with-a-resource-manager-template"></a>Creación de un certificado nuevo con una plantilla de Resource Manager

En el ejemplo siguiente se muestra cómo implementar un certificado en su cuenta de Automation mediante una plantilla de Resource Manager a través de PowerShell:

```powershell-interactive
$AutomationAccountName = "<automation account name>"
$PfxCertPath = '<PFX cert path'
$CertificatePassword = '<password>'
$certificateName = '<certificate name>'
$AutomationAccountName = '<automation account name>'
$flags = [System.Security.Cryptography.X509Certificates.X509KeyStorageFlags]::Exportable `
    -bor [System.Security.Cryptography.X509Certificates.X509KeyStorageFlags]::PersistKeySet `
    -bor [System.Security.Cryptography.X509Certificates.X509KeyStorageFlags]::MachineKeySet
# Load the certificate into memory
$PfxCert = New-Object -TypeName System.Security.Cryptography.X509Certificates.X509Certificate2 -ArgumentList @($PfxCertPath, $CertificatePassword, $flags)
# Export the certificate and convert into base 64 string
$Base64Value = [System.Convert]::ToBase64String($PfxCert.Export([System.Security.Cryptography.X509Certificates.X509ContentType]::Pkcs12))
$Thumbprint = $PfxCert.Thumbprint


$json = @"
{
    '`$schema': 'https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#',
    'contentVersion': '1.0.0.0',
    'resources': [
        {
            'name': '$AutomationAccountName/$certificateName',
            'type': 'Microsoft.Automation/automationAccounts/certificates',
            'apiVersion': '2015-10-31',
            'properties': {
                'base64Value': '$Base64Value',
                'thumbprint': '$Thumbprint',
                'isExportable': true
            }
        }
    ]
}
"@

$json | out-file .\template.json
New-AzResourceGroupDeployment -Name NewCert -ResourceGroupName TestAzureAuto -TemplateFile .\template.json
```

## <a name="get-a-certificate"></a>Obtener un certificado

Para recuperar un certificado, use el cmdlet `Get-AutomationCertificate` interno. No puede usar el cmdlet [Get-AzAutomationCertificate](/powershell/module/Az.Automation/Get-AzAutomationCertificate?view=azps-3.7.0) porque devuelve información sobre el recurso de certificado, pero no el certificado en sí.

### <a name="textual-runbook-example"></a>Ejemplo de un runbook textual

En el ejemplo siguiente se muestra cómo agregar un certificado a un servicio en la nube en un runbook. En este ejemplo, la contraseña se recupera a partir de una variable de automatización cifrada.

```powershell-interactive
$serviceName = 'MyCloudService'
$cert = Get-AutomationCertificate -Name 'MyCertificate'
$certPwd = Get-AzAutomationVariable -ResourceGroupName "ResourceGroup01" `
–AutomationAccountName "MyAutomationAccount" –Name 'MyCertPassword'
Add-AzureCertificate -ServiceName $serviceName -CertToDeploy $cert
```

### <a name="graphical-runbook-example"></a>Ejemplos de un runbook gráfico

Para agregar una actividad para el cmdlet `Get-AutomationCertificate` interno a un runbook gráfico, haga clic con el botón derecho en el certificado en el panel Biblioteca y seleccione **Agregar al lienzo**.

![Captura de pantalla de la incorporación de un certificado al lienzo](../media/certificates/automation-certificate-add-to-canvas.png)

La imagen siguiente muestra un ejemplo de cómo usar un certificado en un runbook gráfico. 

![Captura de pantalla de un ejemplo de creación gráfica](../media/certificates/graphical-runbook-add-certificate.png)

### <a name="python-2-example"></a>Ejemplo de Python 2

En el ejemplo siguiente se muestra cómo acceder a los certificados en runbooks de Python 2.

```python
# get a reference to the Azure Automation certificate
cert = automationassets.get_automation_certificate("AzureRunAsCertificate")

# returns the binary cert content  
print cert
```

## <a name="next-steps"></a>Pasos siguientes

* Para más información sobre los cmdlets que se usan para acceder a los certificados, consulte [Administración de módulos en Azure Automation](modules.md).
* Para información general sobre los runbooks, consulte [Ejecución de un runbook en Azure Automation](../automation-runbook-execution.md).
* Para obtener más información sobre las configuraciones DSC, consulte [Introducción a State Configuration](../automation-dsc-overview.md). 

