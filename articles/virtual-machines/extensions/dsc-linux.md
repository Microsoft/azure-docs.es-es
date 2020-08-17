---
title: Extensión de Azure DSC para Linux
description: Instala los paquetes OMI y DSC para permitir que una máquina virtual Linux de Azure se configure con Desired State Configuration.
services: virtual-machines-linux
documentationcenter: ''
author: bobbytreed
manager: carmonm
editor: ''
ms.assetid: ''
ms.service: virtual-machines-linux
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure-services
ms.date: 06/12/2018
ms.author: robreed
ms.openlocfilehash: a1a166d12ef753a7a6fc7225d0467ead08514f99
ms.sourcegitcommit: dea88d5e28bd4bbd55f5303d7d58785fad5a341d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87876723"
---
# <a name="dsc-extension-for-linux-microsoftostcextensionsdscforlinux"></a>Extensión de DSC para Linux (Microsoft.OSTCExtensions.DSCForLinux)

Desired State Configuration (DSC) es una plataforma de administración que puede utilizar para administrar la infraestructura de TI y de desarrollo con configuración como código.

> [!NOTE]
> La extensión de DSC para Linux y la [extensión de máquina virtual de Azure Monitor para Linux](./oms-linux.md) presentan actualmente un conflicto y no se admiten en una configuración en paralelo. No use las dos soluciones en la misma máquina virtual.

Microsoft publica y admite la extensión DSCForLinux. La extensión instala los agentes OMI y DSC en las máquinas virtuales de Azure. La extensión DSC también puede realizar las siguientes acciones:

- Registrar la máquina virtual Linux en la cuenta de Azure Automation para extraer las configuraciones del servicio Azure Automation (Register ExtensionAction).
- Insertar las configuraciones de MOF en la máquina virtual Linux (Push ExtensionAction).
- Aplicar la configuración de Meta MOF en la máquina virtual Linux para configurar un servidor de extracción y extraer la configuración de nodo (Pull ExtensionAction).
- Instalar módulos personalizados de DSC en la máquina virtual Linux (Install ExtensionAction).
- Eliminar módulos personalizados de DSC en la máquina virtual Linux (Remove ExtensionAction).

## <a name="prerequisites"></a>Prerrequisitos

### <a name="operating-system"></a>Sistema operativo

Para los nodos que ejecutan Linux, la extensión DSC de Linux admite todas las distribuciones de Linux que se enumeran en la [documentación de DSC de PowerShell](/powershell/scripting/dsc/getting-started/lnxgettingstarted).
 
### <a name="internet-connectivity"></a>Conectividad de Internet

La extensión DSCForlinux requiere que la máquina virtual de destino esté conectada a Internet. Por ejemplo, la extensión Register requiere conectividad con el servicio Automation. En otras acciones, como Pull e Install, se requiere conectividad con Azure Storage y GitHub. Depende de la configuración que proporcione el cliente.

## <a name="extension-schema"></a>Esquema de extensión

### <a name="public-configuration"></a>Configuración pública

Estos son todos los parámetros de configuración pública admitidos:

* `FileUri`: (opcional, cadena) URI del archivo MOF, del archivo Meta MOF o del archivo ZIP de recursos personalizados.
* `ResourceName`: (opcional, cadena) nombre del módulo de recursos personalizados.
* `ExtensionAction`: (opcional, cadena) especifica lo que hace una extensión. Valores válidos: Register, Push, Pull, Install y Remove. Si no se especifica, se considera como acción de inserción de forma predeterminada.
* `NodeConfigurationName`: (opcional, cadena) nombre de una configuración de nodo que se aplicará.
* `RefreshFrequencyMins`: (opcional, entero) especifica la frecuencia (en minutos) en que DSC intenta obtener la configuración del servidor de extracción. 
       Si la configuración del servidor de extracción es diferente de la configuración actual en el nodo de destino, se copia en el almacén pendiente y se aplica.
* `ConfigurationMode`: (opcional, cadena) especifica cómo DSC debe aplicar la configuración. Los valores válidos son: ApplyOnly, ApplyAndMonitor y ApplyAndAutoCorrect.
* `ConfigurationModeFrequencyMins`: (opcional, entero) especifica la frecuencia (en minutos) en que DSC garantiza que la configuración esté en el estado deseado.

> [!NOTE]
> Si usa una versión anterior a la 2.3, el parámetro mode es el mismo que el de ExtensionAction. El modo parece ser un término sobrecargado. Para evitar confusiones, se usa ExtensionAction a partir de la versión 2.3 en adelante. Para ofrecer compatibilidad con versiones anteriores, la extensión admite tanto el modo como ExtensionAction. 
>

### <a name="protected-configuration"></a>Configuración protegida

Estos son todos los parámetros de configuración protegidos que se admiten:

* `StorageAccountName`: (opcional, cadena) nombre de la cuenta de almacenamiento que contiene el archivo
* `StorageAccountKey`: (opcional, cadena) clave de la cuenta de almacenamiento que contiene el archivo
* `RegistrationUrl`: (opcional, cadena) dirección URL de la cuenta de Azure Automation
* `RegistrationKey`: (opcional, cadena) clave de acceso de la cuenta de Azure Automation


## <a name="scenarios"></a>Escenarios

### <a name="register-an-azure-automation-account"></a>Registro de una cuenta de Azure Automation
protected.json
```json
{
  "RegistrationUrl": "<azure-automation-account-url>",
  "RegistrationKey": "<azure-automation-account-key>"
}
```
public.json
```json
{
  "ExtensionAction" : "Register",
  "NodeConfigurationName" : "<node-configuration-name>",
  "RefreshFrequencyMins" : "<value>",
  "ConfigurationMode" : "<ApplyAndMonitor | ApplyAndAutoCorrect | ApplyOnly>",
  "ConfigurationModeFrequencyMins" : "<value>"
}
```

Formato de PowerShell
```powershell
$privateConfig = '{
  "RegistrationUrl": "<azure-automation-account-url>",
  "RegistrationKey": "<azure-automation-account-key>"
}'

$publicConfig = '{
  "ExtensionAction" : "Register",
  "NodeConfigurationName": "<node-configuration-name>",
  "RefreshFrequencyMins": "<value>",
  "ConfigurationMode": "<ApplyAndMonitor | ApplyAndAutoCorrect | ApplyOnly>",
  "ConfigurationModeFrequencyMins": "<value>"
}'
```

### <a name="apply-an-mof-configuration-file-in-an-azure-storage-account-to-the-vm"></a>Aplicación de un archivo de configuración MOF (de una cuenta de Azure Storage) en la máquina virtual

protected.json
```json
{
  "StorageAccountName": "<storage-account-name>",
  "StorageAccountKey": "<storage-account-key>"
}
```

public.json
```json
{
  "FileUri": "<mof-file-uri>",
  "ExtensionAction": "Push"
}
```

Formato de PowerShell
```powershell
$privateConfig = '{
  "StorageAccountName": "<storage-account-name>",
  "StorageAccountKey": "<storage-account-key>"
}'

$publicConfig = '{
  "FileUri": "<mof-file-uri>",
  "ExtensionAction": "Push"
}'
```


### <a name="apply-an-mof-configuration-file-in-public-storage-to-the-vm"></a>Aplicación de un archivo de configuración MOF (en el almacenamiento público) en la máquina virtual

public.json
```json
{
  "FileUri": "<mof-file-uri>"
}
```

Formato de PowerShell
```powershell
$publicConfig = '{
  "FileUri": "<mof-file-uri>"
}'
```

### <a name="apply-a-meta-mof-configuration-file-in-an-azure-storage-account-to-the-vm"></a>Aplicación de un archivo de configuración Meta MOF (de una cuenta de Azure Storage) en la máquina virtual

protected.json
```json
{
  "StorageAccountName": "<storage-account-name>",
  "StorageAccountKey": "<storage-account-key>"
}
```

public.json
```json
{
  "ExtensionAction": "Pull",
  "FileUri": "<meta-mof-file-uri>"
}
```

Formato de PowerShell
```powershell
$privateConfig = '{
  "StorageAccountName": "<storage-account-name>",
  "StorageAccountKey": "<storage-account-key>"
}'

$publicConfig = '{
  "ExtensionAction": "Pull",
  "FileUri": "<meta-mof-file-uri>"
}'
```

### <a name="apply-a-meta-mof-configuration-file-in-public-storage-to-the-vm"></a>Aplicación de un archivo de configuración Meta MOF (en el almacenamiento público) en la VM
public.json
```json
{
  "FileUri": "<meta-mof-file-uri>",
  "ExtensionAction": "Pull"
}
```
Formato de PowerShell
```powershell
$publicConfig = '{
  "FileUri": "<meta-mof-file-uri>",
  "ExtensionAction": "Pull"
}'
```

### <a name="install-a-custom-resource-module-a-zip-file-in-an-azure-storage-account-to-the-vm"></a>Instalación de un módulo de recursos personalizados (un archivo ZIP en la cuenta de Azure Storage) en la máquina virtual
protected.json
```json
{
  "StorageAccountName": "<storage-account-name>",
  "StorageAccountKey": "<storage-account-key>"
}
```
public.json
```json
{
  "ExtensionAction": "Install",
  "FileUri": "<resource-zip-file-uri>"
}
```

Formato de PowerShell
```powershell
$privateConfig = '{
  "StorageAccountName": "<storage-account-name>",
  "StorageAccountKey": "<storage-account-key>"
}'

$publicConfig = '{
  "ExtensionAction": "Install",
  "FileUri": "<resource-zip-file-uri>"
}'
```

### <a name="install-a-custom-resource-module-a-zip-file-in-public-storage-to-the-vm"></a>Instalación de un módulo de recursos personalizados (un archivo ZIP en el almacenamiento público) en la máquina virtual
public.json
```json
{
  "ExtensionAction": "Install",
  "FileUri": "<resource-zip-file-uri>"
}
```
Formato de PowerShell
```powershell
$publicConfig = '{
  "ExtensionAction": "Install",
  "FileUri": "<resource-zip-file-uri>"
}'
```

### <a name="remove-a-custom-resource-module-from-the-vm"></a>Eliminación de un módulo de recursos personalizados de la VM
public.json
```json
{
  "ResourceName": "<resource-name>",
  "ExtensionAction": "Remove"
}
```
Formato de PowerShell
```powershell
$publicConfig = '{
  "ResourceName": "<resource-name>",
  "ExtensionAction": "Remove"
}'
```

## <a name="template-deployment"></a>Implementación de plantilla

Las extensiones de VM de Azure pueden implementarse con plantillas de Azure Resource Manager. Las plantillas resultan ideales al implementar una o varias máquinas virtuales que requieren configurarse tras la implementación, por ejemplo, para incorporarse a Azure Automation. 

La plantilla de Resource Manager de ejemplo es [201-dsc-linux-azure-storage-on-ubuntu](https://github.com/Azure/azure-quickstart-templates/tree/master/201-dsc-linux-azure-storage-on-ubuntu) y [201-dsc-linux-public-storage-on-ubuntu](https://github.com/Azure/azure-quickstart-templates/tree/master/201-dsc-linux-public-storage-on-ubuntu).

Para más información sobre la plantilla de Resource Manager, consulte [Creación de plantillas de Azure Resource Manager](../../azure-resource-manager/templates/template-syntax.md).


## <a name="azure-cli-deployment"></a>Implementación de la CLI de Azure

### <a name="use-azure-cliazure-cli"></a>Uso de la [CLI de Azure] [azure-cli]
Antes de implementar la extensión DSCForLinux, configure `public.json` y `protected.json`, según los diferentes escenarios de la sección 3.

#### <a name="classic"></a>Clásico

[!INCLUDE [classic-vm-deprecation](../../../includes/classic-vm-deprecation.md)]

El modo de implementación clásica también se denomina modo de administración de servicios de Azure. Puede cambiarlo mediante la ejecución de:
```
$ azure config mode asm
```

Puede implementar la extensión DSCForLinux mediante la ejecución de:
```
$ azure vm extension set <vm-name> DSCForLinux Microsoft.OSTCExtensions <version> \
--private-config-path protected.json --public-config-path public.json
```

Para obtener información sobre la última versión disponible de la extensión, ejecute:
```
$ azure vm extension list
```

#### <a name="resource-manager"></a>Resource Manager
Cambie al modo Azure Resource Manager mediante la ejecución de:
```
$ azure config mode arm
```

Puede implementar la extensión DSCForLinux mediante la ejecución de:
```
$ azure vm extension set <resource-group> <vm-name> \
DSCForLinux Microsoft.OSTCExtensions <version> \
--private-config-path protected.json --public-config-path public.json
```
> [!NOTE]
> En el modo de Azure Resource Manager, `azure vm extension list` no está disponible por ahora.
>

### <a name="use-azure-powershellazure-powershell"></a>Uso de [Azure PowerShell][azure-powershell]

#### <a name="classic"></a>Clásico

Puede iniciar sesión en su cuenta de Azure (modo de administración de servicios de Azure) mediante la ejecución de:

```powershell>
Add-AzureAccount
```

Asimismo, puede implementar la extensión DSCForLinux mediante la ejecución de:

```powershell>
$vmname = '<vm-name>'
$vm = Get-AzureVM -ServiceName $vmname -Name $vmname
$extensionName = 'DSCForLinux'
$publisher = 'Microsoft.OSTCExtensions'
$version = '< version>'
```

Cambie el contenido de $privateConfig y $publicConfig según los diferentes escenarios de la sección anterior.
```
$privateConfig = '{
  "StorageAccountName": "<storage-account-name>",
  "StorageAccountKey": "<storage-account-key>"
}'
```

```
$publicConfig = '{
  "ExtensionAction": "Push",
  "FileUri": "<mof-file-uri>"
}'
```

```
Set-AzureVMExtension -ExtensionName $extensionName -VM $vm -Publisher $publisher `
  -Version $version -PrivateConfiguration $privateConfig `
  -PublicConfiguration $publicConfig | Update-AzureVM
```

#### <a name="resource-manager"></a>Resource Manager

Puede iniciar sesión en su cuenta de Azure con el modo de Azure Resource Manager mediante la ejecución de:

```powershell>
Login-AzAccount
```

Para más información sobre cómo usar Azure PowerShell con Azure Resource Manager, consulte [Administración de recursos de Azure con Azure PowerShell](../../azure-resource-manager/management/manage-resources-powershell.md).

Puede implementar la extensión DSCForLinux mediante la ejecución de:

```powershell>
$rgName = '<resource-group-name>'
$vmName = '<vm-name>'
$location = '< location>'
$extensionName = 'DSCForLinux'
$publisher = 'Microsoft.OSTCExtensions'
$version = '< version>'
```

Cambie el contenido de $privateConfig y $publicConfig según los diferentes escenarios de la sección anterior.
```
$privateConfig = '{
  "StorageAccountName": "<storage-account-name>",
  "StorageAccountKey": "<storage-account-key>"
}'
```

```
$publicConfig = '{
  "ExtensionAction": "Push",
  "FileUri": "<mof-file-uri>"
}'
```

```
Set-AzVMExtension -ResourceGroupName $rgName -VMName $vmName -Location $location `
  -Name $extensionName -Publisher $publisher -ExtensionType $extensionName `
  -TypeHandlerVersion $version -SettingString $publicConfig -ProtectedSettingString $privateConfig
```

## <a name="troubleshoot-and-support"></a>Solución de problemas y asistencia

### <a name="troubleshoot"></a>Solución de problemas

Los datos sobre el estado de las implementaciones de extensiones pueden recuperarse desde Azure Portal y mediante la CLI de Azure. Para ver el estado de implementación de las extensiones de una máquina virtual determinada, ejecute el comando siguiente con la CLI de Azure.

```azurecli
az vm extension list --resource-group myResourceGroup --vm-name myVM -o table
```

El resultado de la ejecución de las extensiones se registra en el archivo siguiente:

```
/var/log/azure/<extension-name>/<version>/extension.log file.
```

Código de error: 51 representa una distribución o una acción de extensión no admitidas.
En algunos casos, la extensión DSC de Linux no puede instalar OMI cuando ya existe una versión posterior de OMI en la máquina. [Respuesta de error: (000003) Downgrade not allowed (Cambio a una versión anterior no permitido)]



### <a name="support"></a>Soporte técnico

Si necesita más ayuda con cualquier aspecto de este artículo, póngase en contacto con los expertos de Azure en los [foros de MSDN Azure o Stack Overflow](https://azure.microsoft.com/support/community/). Como alternativa, puede registrar un incidente de soporte técnico de Azure. Vaya al [sitio de soporte técnico de Azure](https://azure.microsoft.com/support/options/) y seleccione **Soporte técnico**. Para obtener información sobre el uso del soporte técnico de Azure, lea las [Preguntas más frecuentes del soporte técnico de Microsoft Azure](https://azure.microsoft.com/support/faq/).

## <a name="next-steps"></a>Pasos siguientes
Para más información sobre las extensiones, consulte el artículo [Características y extensiones de las máquinas virtuales para Linux](features-linux.md).
