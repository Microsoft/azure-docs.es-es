---
title: Características y extensiones de las máquinas virtuales de Azure para Windows
description: Obtenga información acerca de qué extensiones están disponibles para máquinas virtuales de Azure, agrupadas por lo que proporcionan o mejoran.
services: virtual-machines-windows
documentationcenter: ''
author: axayjo
manager: gwallace
editor: ''
tags: azure-service-management,azure-resource-manager
ms.assetid: 999d63ee-890e-432e-9391-25b3fc6cde28
ms.service: virtual-machines-windows
ms.topic: article
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure-services
ms.date: 03/30/2018
ms.author: akjosh
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 611edb06762b96ded7671b70ec0f5d4f07f51848
ms.sourcegitcommit: 2ff0d073607bc746ffc638a84bb026d1705e543e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87829091"
---
# <a name="virtual-machine-extensions-and-features-for-windows"></a>Características y extensiones de las máquinas virtuales para Windows

Las extensiones de máquina virtual de Azure son aplicaciones pequeñas que realizan tareas de automatización y configuración posterior a la implementación en máquinas virtuales de Azure. Por ejemplo, si una máquina virtual necesita que se instale software, protección antivirus o ejecutar un script en ella, se puede usar una extensión de máquina virtual. Las extensiones de máquina virtual de Azure se pueden ejecutar con la CLI de Azure, PowerShell, plantillas de Azure Resource Manager y Azure Portal. Las extensiones se pueden empaquetar con la implementación de una máquina virtual nueva o se pueden ejecutar en cualquier sistema existente.

Este artículo proporciona información general sobre las extensiones de máquina virtual, requisitos previos para usar las extensiones de máquina virtual de Azure e instrucciones sobre cómo detectar, administrar y quitar extensiones de máquina virtual. Este artículo proporciona información generalizada porque muchas extensiones de máquina virtual están disponibles, cada una con una configuración potencialmente única. En cada documento específico de la extensión individual pueden encontrarse detalles específicos de extensión.

 

## <a name="use-cases-and-samples"></a>Casos de uso y ejemplos

Varias extensiones de máquina virtual de Azure diferentes están disponibles, cada una con un caso de uso específico. Estos son algunos ejemplos:

- Aplique configuraciones de estado deseado de PowerShell a una máquina virtual con la extensión DSC para Windows. Para obtener más información, consulte la sección sobre la [extensión de configuración de estado deseado de Azure](dsc-overview.md).
- Configure la supervisión de una máquina virtual con la extensión de máquina virtual del agente de Log Analytics. Para más información, consulte el artículo sobre la [conexión de máquinas virtuales de Azure a registros de Azure Monitor](../../azure-monitor/learn/quick-collect-azurevm.md).
- Configure una máquina virtual de Azure con Chef. Para más información, consulte el artículo sobre [automatización de la implementación de máquinas virtuales de Azure con Chef](/azure/developer/chef/windows-vm-configure).
- Configure la supervisión de su infraestructura de Azure con la extensión de Datadog. Para obtener más información, consulte el [blog de Datadog](https://www.datadoghq.com/blog/introducing-azure-monitoring-with-one-click-datadog-deployment/).


Además de las extensiones específicas de proceso, una extensión de script personalizado está disponible tanto para máquinas virtuales Windows como para máquinas virtuales Linux. La extensión de script personalizado para Windows permite que se ejecute cualquier script de PowerShell en una máquina virtual. Los scripts personalizados resultan útiles para diseñar implementaciones de Azure que requieren una configuración más allá de lo que las herramientas de Azure nativas pueden proporcionar. Para obtener más información, consulte la sección sobre la [extensión de script personalizado de máquina virtual Windows](custom-script-windows.md).

## <a name="prerequisites"></a>Requisitos previos

Para controlar la extensión en la VM, debe tener instalado el Agente Windows de Azure. Algunas extensiones individuales tienen requisitos previos, como el acceso a los recursos o las dependencias.

### <a name="azure-vm-agent"></a>Agente de máquina virtual de Azure

El agente de máquina virtual de Azure administra las interacciones entre una máquina virtual de Azure y el controlador de tejido de Azure. El agente de máquina virtual es responsable de muchos aspectos funcionales de la implementación y administración de las máquinas virtuales de Azure, incluida la ejecución de extensiones de máquina virtual. El agente de máquina virtual de Azure está preinstalado en imágenes de Azure Marketplace y se puede instalar manualmente en sistemas operativos compatibles. El agente de máquina virtual de Azure para Windows se conoce como Windows Guest Agent.

Para obtener información sobre las instrucciones de instalación y los sistemas operativos compatibles, consulte [Agente de máquina virtual de Azure](agent-windows.md).

#### <a name="supported-agent-versions"></a>Versiones de agente compatibles

Para brindar la mejor experiencia posible, se requieren versiones mínimas del agente. Para obtener más información, consulte [este artículo](https://support.microsoft.com/help/4049215/extensions-and-virtual-machine-agent-minimum-version-support).

#### <a name="supported-oses"></a>Sistemas operativos compatibles

Windows Guest Agent se ejecuta en varios sistemas operativos, pero el marco de las extensiones tiene un límite para los sistemas operativos de esas extensiones. Para obtener más información, consulte [este artículo](https://support.microsoft.com/help/4078134/azure-extension-supported-operating-systems
).

Algunas extensiones no son compatibles con todos los sistemas operativos y puede aparecer el *código de error 51, "Sistema operativo no compatible"* . Revise la documentación de cada una de las extensiones para información sobre la compatibilidad.

#### <a name="network-access"></a>Acceso de red

Los paquetes de extensiones se descargan del repositorio de extensiones de Azure Storage y las cargas de estado de las extensiones se publican en Azure Storage. Si usa una versión [compatible](https://support.microsoft.com/help/4049215/extensions-and-virtual-machine-agent-minimum-version-support) de los agentes, no necesita autorizar el acceso a Azure Storage en la región de la máquina virtual, porque puede usar el agente para redirigir la comunicación al controlador de tejido de Azure para las comunicaciones de los agentes (la característica HostGAPlugin a través del canal con privilegios en la dirección IP privada [168.63.129.16](../../virtual-network/what-is-ip-address-168-63-129-16.md)). Si usa una versión no compatible del agente, deberá autorizar el acceso de salida a Azure Storage en esa región desde la máquina virtual.

> [!IMPORTANT]
> Si bloqueó el acceso a *168.63.129.16* con el firewall invitado o con un proxy, las extensiones generarán un error con independencia de lo mencionado anteriormente. Se requieren los puertos 80, 443 y 32526.

Los agentes solo se pueden usar para descargar los paquetes de extensiones e informar el estado. Por ejemplo, si la instalación de una extensión requiere descargar un script de GitHub (script personalizado) o necesita acceso a Azure Storage (Azure Backup), se deben abrir puertos adicionales del firewall o del grupo de seguridad de red. Distintas extensiones tienen distintos requisitos, porque son aplicaciones por sí mismas. En el caso de las extensiones que requieren acceso a Azure Storage o Azure Active Directory, puede permitir el acceso mediante las [etiquetas del servicio NSG de Azure](../../virtual-network/security-overview.md#service-tags) para Storage o Azure Active Directory.

Windows Guest Agent no admite el servidor proxy para redirigir las solicitudes de tráfico del agente, de modo que Windows Guest Agent se basará en el proxy personalizado (si tiene uno) para acceder a recursos en Internet o en el host a través de la dirección IP 168.63.129.16.

## <a name="discover-vm-extensions"></a>Detección de extensiones de máquina virtual

Hay muchas extensiones de máquina virtual diferentes disponibles para su uso con máquinas virtuales de Azure. Para ver una lista completa, use [Get-AzVMExtensionImage](/powershell/module/az.compute/get-azvmextensionimage). En el ejemplo siguiente se muestran todas las extensiones disponibles en la ubicación *WestUS*:

```powershell
Get-AzVmImagePublisher -Location "WestUS" | `
Get-AzVMExtensionImageType | `
Get-AzVMExtensionImage | Select Type, Version
```

## <a name="run-vm-extensions"></a>Ejecución de extensiones de máquina virtual

Las extensiones de máquinas virtuales de Azure se ejecutan en máquinas virtuales existentes, lo que resulta útil si es preciso realizar cambios de configuración o recuperar la conectividad en una máquina virtual ya implementada. Las extensiones de máquina virtual también pueden agruparse con las implementaciones de plantilla de Azure Resource Manager. Al usar las extensiones con plantillas de Resource Manager, las máquinas virtuales de Azure se pueden implementar y configurar sin intervención alguna después de la implementación.

Los métodos siguientes pueden usarse para ejecutar una extensión en una máquina virtual existente.

### <a name="powershell"></a>PowerShell

Existen varios comandos de PowerShell para ejecutar extensiones individuales. Para ver una lista, use [Get-Command](/powershell/module/microsoft.powershell.core/get-command) y filtre por *Extensión*:

```powershell
Get-Command Set-Az*Extension* -Module Az.Compute
```

Esto proporciona una salida similar a la siguiente:

```powershell
CommandType     Name                                          Version    Source
-----------     ----                                          -------    ------
Cmdlet          Set-AzVMAccessExtension                       4.5.0      Az.Compute
Cmdlet          Set-AzVMADDomainExtension                     4.5.0      Az.Compute
Cmdlet          Set-AzVMAEMExtension                          4.5.0      Az.Compute
Cmdlet          Set-AzVMBackupExtension                       4.5.0      Az.Compute
Cmdlet          Set-AzVMBginfoExtension                       4.5.0      Az.Compute
Cmdlet          Set-AzVMChefExtension                         4.5.0      Az.Compute
Cmdlet          Set-AzVMCustomScriptExtension                 4.5.0      Az.Compute
Cmdlet          Set-AzVMDiagnosticsExtension                  4.5.0      Az.Compute
Cmdlet          Set-AzVMDiskEncryptionExtension               4.5.0      Az.Compute
Cmdlet          Set-AzVMDscExtension                          4.5.0      Az.Compute
Cmdlet          Set-AzVMExtension                             4.5.0      Az.Compute
Cmdlet          Set-AzVMSqlServerExtension                    4.5.0      Az.Compute
Cmdlet          Set-AzVmssDiskEncryptionExtension             4.5.0      Az.Compute
```

En el ejemplo siguiente se usa la extensión de script personalizado para descargar un script desde un repositorio de GitHub en la máquina virtual de destino y, a continuación, ejecutar el script. Para obtener más información sobre la extensión de script personalizado, consulte [Introducción a la extensión de script personalizado](custom-script-windows.md).

```powershell
Set-AzVMCustomScriptExtension -ResourceGroupName "myResourceGroup" `
    -VMName "myVM" -Name "myCustomScript" `
    -FileUri "https://raw.githubusercontent.com/neilpeterson/nepeters-azure-templates/master/windows-custom-script-simple/support-scripts/Create-File.ps1" `
    -Run "Create-File.ps1" -Location "West US"
```

En el ejemplo siguiente, la extensión de acceso a la máquina virtual se usa para restablecer la contraseña administrativa de una máquina virtual Windows como contraseña temporal. Para obtener más información sobre la extensión de acceso a la máquina virtual, consulte la sección sobre cómo [restablecer el servicio Escritorio remoto en una máquina virtual Windows](../troubleshooting/reset-rdp.md). Una vez que se ejecute, deberá restablecer la contraseña en el primer inicio de sesión:

```powershell
$cred=Get-Credential

Set-AzVMAccessExtension -ResourceGroupName "myResourceGroup" -VMName "myVM" -Name "myVMAccess" `
    -Location WestUS -UserName $cred.GetNetworkCredential().Username `
    -Password $cred.GetNetworkCredential().Password -typeHandlerVersion "2.0"
```

El comando `Set-AzVMExtension` se puede usar para iniciar cualquier extensión de máquina virtual. Para más información, consulte la [referencia de Set-AzVMExtension](/powershell/module/az.compute/set-azvmextension).


### <a name="azure-portal"></a>Azure portal

Las extensiones de máquina virtual se pueden aplicar a una máquina virtual existente a través de Azure Portal. Seleccione la máquina virtual en el portal, elija **Extensiones** y, luego, **Agregar**. Elija la extensión que quiera en la lista de extensiones disponibles y siga las instrucciones del asistente.

En el ejemplo siguiente se muestra la instalación de la extensión de Microsoft Antimalware desde Azure Portal:

![Instalar la extensión de antimalware](./media/features-windows/installantimalwareextension.png)

### <a name="azure-resource-manager-templates"></a>Plantillas del Administrador de recursos de Azure

Las extensiones de máquina virtual se pueden agregar a una plantilla de Azure Resource Manager y ejecutar con la implementación de la plantilla. Al implementar una extensión con una plantilla, puede crear implementaciones de Azure completamente configuradas. Por ejemplo, el siguiente JSON procede de una plantilla de Resource Manager que implementa un conjunto de máquinas virtuales de carga equilibrada y una instancia de Azure SQL Database y, a continuación, instala una aplicación .NET Core en cada máquina virtual. La extensión de máquina virtual se encarga de la instalación de software.

Para obtener más información, consulte la [plantilla completa de Resource Manager](https://github.com/Microsoft/dotnet-core-sample-templates/tree/master/dotnet-core-music-windows).

```json
{
    "apiVersion": "2015-06-15",
    "type": "extensions",
    "name": "config-app",
    "location": "[resourceGroup().location]",
    "dependsOn": [
    "[concat('Microsoft.Compute/virtualMachines/', variables('vmName'),copyindex())]",
    "[variables('musicstoresqlName')]"
    ],
    "tags": {
    "displayName": "config-app"
    },
    "properties": {
    "publisher": "Microsoft.Compute",
    "type": "CustomScriptExtension",
    "typeHandlerVersion": "1.9",
    "autoUpgradeMinorVersion": true,
    "settings": {
        "fileUris": [
        "https://raw.githubusercontent.com/Microsoft/dotnet-core-sample-templates/master/dotnet-core-music-windows/scripts/configure-music-app.ps1"
        ]
    },
    "protectedSettings": {
        "commandToExecute": "[concat('powershell -ExecutionPolicy Unrestricted -File configure-music-app.ps1 -user ',parameters('adminUsername'),' -password ',parameters('adminPassword'),' -sqlserver ',variables('musicstoresqlName'),'.database.windows.net')]"
    }
    }
}
```

Para más información sobre cómo crear plantillas de Resource Manager, consulte [Authoring Azure Resource Manager templates with Windows VM extensions](../windows/template-description.md#extensions) (Creación de plantillas de Azure Resource Manager con las extensiones de máquinas virtuales Windows).

## <a name="secure-vm-extension-data"></a>Protección de datos de extensión de máquina virtual

Cuando ejecuta una extensión de máquina virtual, puede que sea necesario incluir información confidencial como credenciales, nombres de cuenta de almacenamiento y claves de acceso de cuenta de almacenamiento. Muchas extensiones de máquina virtual incluyen una configuración protegida que cifra datos y los descifra solo dentro de la máquina virtual de destino. Cada extensión tiene un esquema específico de configuración protegida que se detalla de forma individual en la documentación específica de extensión.

En el ejemplo siguiente se muestra una instancia de la extensión de script personalizado para Windows. El comando que se ejecutará incluye un conjunto de credenciales. En este ejemplo, el comando que se ejecutará no está cifrado:

```json
{
    "apiVersion": "2015-06-15",
    "type": "extensions",
    "name": "config-app",
    "location": "[resourceGroup().location]",
    "dependsOn": [
    "[concat('Microsoft.Compute/virtualMachines/', variables('vmName'),copyindex())]",
    "[variables('musicstoresqlName')]"
    ],
    "tags": {
    "displayName": "config-app"
    },
    "properties": {
    "publisher": "Microsoft.Compute",
    "type": "CustomScriptExtension",
    "typeHandlerVersion": "1.9",
    "autoUpgradeMinorVersion": true,
    "settings": {
        "fileUris": [
        "https://raw.githubusercontent.com/Microsoft/dotnet-core-sample-templates/master/dotnet-core-music-windows/scripts/configure-music-app.ps1"
        ],
        "commandToExecute": "[concat('powershell -ExecutionPolicy Unrestricted -File configure-music-app.ps1 -user ',parameters('adminUsername'),' -password ',parameters('adminPassword'),' -sqlserver ',variables('musicstoresqlName'),'.database.windows.net')]"
    }
    }
}
```

Al moverse la propiedad **command to execute** a la configuración **protegida**, se protege la cadena de ejecución tal como se muestra en el ejemplo siguiente:

```json
{
    "apiVersion": "2015-06-15",
    "type": "extensions",
    "name": "config-app",
    "location": "[resourceGroup().location]",
    "dependsOn": [
    "[concat('Microsoft.Compute/virtualMachines/', variables('vmName'),copyindex())]",
    "[variables('musicstoresqlName')]"
    ],
    "tags": {
    "displayName": "config-app"
    },
    "properties": {
    "publisher": "Microsoft.Compute",
    "type": "CustomScriptExtension",
    "typeHandlerVersion": "1.9",
    "autoUpgradeMinorVersion": true,
    "settings": {
        "fileUris": [
        "https://raw.githubusercontent.com/Microsoft/dotnet-core-sample-templates/master/dotnet-core-music-windows/scripts/configure-music-app.ps1"
        ]
    },
    "protectedSettings": {
        "commandToExecute": "[concat('powershell -ExecutionPolicy Unrestricted -File configure-music-app.ps1 -user ',parameters('adminUsername'),' -password ',parameters('adminPassword'),' -sqlserver ',variables('musicstoresqlName'),'.database.windows.net')]"
    }
    }
}
```

En una máquina virtual de IaaS de Azure que use extensiones, en la consola de certificados, es posible que vea certificados que tienen el asunto **_Generador de certificados CRP de Windows Azure_**. En una máquina virtual de RDFE clásica, estos certificados tienen el nombre de asunto **_Administración de servicios de Windows Azure para extensiones_**.

Estos certificados protegen la comunicación entre la máquina virtual y su host durante la transferencia de los valores protegidos (contraseña y otras credenciales) que usan las extensiones. Los certificados se compilan mediante el controlador de tejido de Azure y se pasan al agente de máquina virtual. Si detiene e inicia la máquina virtual cada día, el controlador de tejido podría crear un nuevo certificado. El certificado se almacena en el almacén de certificados personales del equipo. Estos certificados se pueden eliminar. El agente de máquina virtual puede volver a crear certificados si es necesario.

### <a name="how-do-agents-and-extensions-get-updated"></a>¿Cómo se actualizan los agentes y las extensiones?

Los agentes y las extensiones comparten el mismo mecanismo de actualización. Algunas actualizaciones no necesitan reglas adicionales de firewall.

Cuando hay disponible una actualización, solo se instala en la máquina virtual cuando se produce un cambio en las extensiones y otros cambios del modelo de máquina virtual, como:

- Discos de datos.
- Extensiones
- Contenedor de diagnósticos de arranque
- Secretos del sistema operativo invitado
- Tamaño de VM
- Perfil de red

Los publicadores hacen que las actualizaciones estén disponibles en distintos momentos en las diferentes regiones, por lo que es posible que tenga máquinas virtuales en distintas regiones con versiones diferentes.

#### <a name="listing-extensions-deployed-to-a-vm"></a>Lista de extensiones implementadas en una máquina virtual

```powershell
$vm = Get-AzVM -ResourceGroupName "myResourceGroup" -VMName "myVM"
$vm.Extensions | select Publisher, VirtualMachineExtensionType, TypeHandlerVersion
```

```powershell
Publisher             VirtualMachineExtensionType          TypeHandlerVersion
---------             ---------------------------          ------------------
Microsoft.Compute     CustomScriptExtension                1.9
```

#### <a name="agent-updates"></a>Actualizaciones del agente

Windows Guest Agent solo contiene el *código de administración de extensiones*, porque el *código de aprovisionamiento de Windows* es independiente. Puede desinstalar Windows Guest Agent. No puede deshabilitar la actualización automática de Window Guest Agent.

El *código de administración de extensiones* es responsable de comunicarse con el tejido de Azure y administrar las operaciones de extensiones de máquinas virtuales como las instalaciones, informar el estado, actualizar las extensiones individuales y quitarlas. Las actualizaciones contienen revisiones de seguridad, correcciones de errores y mejoras en el *código de administración de extensiones*.

Para comprobar qué versión ejecuta, consulte el artículo sobre [cómo detectar la versión de Windows Guest Agent instalada](agent-windows.md#detect-the-vm-agent).

#### <a name="extension-updates"></a>Actualizaciones de extensiones

Cuando hay disponible una actualización de extensión, Windows Guest Agent la descarga y actualiza. Las actualizaciones de extensiones automáticas son *secundarias* o *revisiones*. Puede decidir si desea recibir o no las actualizaciones *secundarias* de las extensiones cuando aprovisiona la extensión. En el ejemplo siguiente se muestra cómo actualizar automáticamente las versiones secundarias en una plantilla de Resource Manager con *autoUpgradeMinorVersion": true,'* :

```json
    "properties": {
    "publisher": "Microsoft.Compute",
    "type": "CustomScriptExtension",
    "typeHandlerVersion": "1.9",
    "autoUpgradeMinorVersion": true,
    "settings": {
        "fileUris": [
        "https://raw.githubusercontent.com/Microsoft/dotnet-core-sample-templates/master/dotnet-core-music-windows/scripts/configure-music-app.ps1"
        ]
    },
```

Para obtener las correcciones de errores secundarias más recientes, se recomienda encarecidamente que siempre seleccione la actualización automática en las implementaciones de sus extensiones. No es posible optar por no recibir actualizaciones de revisiones que incluyen correcciones de errores clave o revisiones de seguridad.

### <a name="how-to-identify-extension-updates"></a>Identificación de las actualizaciones de extensiones

#### <a name="identifying-if-the-extension-is-set-with-autoupgrademinorversion-on-a-vm"></a>Identificación de si la extensión tiene establecida la opción autoUpgradeMinorVersion en una máquina virtual

En el modelo de la máquina virtual puede ver si la extensión se aprovisionó con "autoUpgradeMinorVersion". Para comprobarlo, use [Get-AzVm](/powershell/module/az.compute/get-azvm) y proporcione el nombre de la VM y el grupo de recursos como se indica a continuación:

```powerShell
 $vm = Get-AzVm -ResourceGroupName "myResourceGroup" -VMName "myVM"
 $vm.Extensions
```

La salida del ejemplo siguiente muestra que la opción *autoUpgradeMinorVersion* está establecida en *true*:

```powershell
ForceUpdateTag              :
Publisher                   : Microsoft.Compute
VirtualMachineExtensionType : CustomScriptExtension
TypeHandlerVersion          : 1.9
AutoUpgradeMinorVersion     : True
```

#### <a name="identifying-when-an-autoupgrademinorversion-occurred"></a>Identificación de cuando se produce un evento de autoUpgradeMinorVersion

Para ver cuando se produce una actualización de la extensión, revise los registros del agente de la máquina virtual en *C:\WindowsAzure\Logs\WaAppAgent.log*

En el ejemplo siguiente, la máquina virtual tenía instalada *Microsoft.Compute.CustomScriptExtension 1.8*. Hubo una revisión disponible para la versión *1.9*:

```powershell
[INFO]  Getting plugin locations for plugin 'Microsoft.Compute.CustomScriptExtension'. Current Version: '1.8', Requested Version: '1.9'
[INFO]  Auto-Upgrade mode. Highest public version for plugin 'Microsoft.Compute.CustomScriptExtension' with requested version: '1.9', is: '1.9'
```

## <a name="agent-permissions"></a>Permisos del agente

Para realizar sus tareas, el agente se debe ejecutar como *sistema local*.

## <a name="troubleshoot-vm-extensions"></a>Solución de problemas de extensiones de máquina virtual

Cada extensión de máquina virtual puede tener unos pasos de solución de problemas específicos de la extensión. Por ejemplo, cuando usa la extensión de script personalizado, se pueden encontrar detalles de ejecución de script localmente en la máquina virtual donde se ejecutó la extensión. Todos los pasos de solución de problemas específicos de extensión aparecen detallados en la documentación específica de extensión.

Los pasos de solución de problemas siguientes se aplican a todas las extensiones de máquina virtual.

1. Para comprobar el registro de Windows Guest Agent, examine la actividad cuando la extensión se aprovisionó en *C:\WindowsAzure\Logs\WaAppAgent.log*.

2. Revise los registros reales de la extensión para obtener más detalles en *C:\WindowsAzure\Logs\Plugins\<extensionName>*

3. Revise las secciones de solución de problemas de la documentación específica de la extensión para ver los códigos de errores, los problemas conocidos, etc.

4. Examine los registros del sistema. Compruebe si hay otras operaciones que puedan haber interferido con la extensión, como una instalación de ejecución prolongada de otra aplicación que requería acceso exclusivo al administrador de paquetes.

### <a name="common-reasons-for-extension-failures"></a>Motivos comunes para los errores de extensiones

1. Las extensiones tienen 20 minutos para ejecutarse (las excepciones son las extensiones CustomScript, Chef y DSC que tienen 90 minutos). Si la implementación supera este tiempo, se marca como tiempo de expiración. La causa de esto puede deberse a máquinas virtuales de bajos recursos, otras tareas de inicio y configuraciones de máquinas virtuales que consumen grandes cantidades de recursos mientras la extensión intenta realizar el aprovisionamiento.

2. No se cumplen los requisitos previos mínimos. Algunas extensiones tienen dependencias de las SKU de la máquina virtual, como las imágenes de HPC. Las extensiones pueden requerir ciertos requisitos de acceso de red, como comunicaciones con Azure Storage o servicios públicos. Otros ejemplos podrían ser el acceso a repositorios de paquetes, quedarse sin espacio en disco o restricciones de seguridad.

3. Acceso exclusivo al administrador de paquetes. En algunos casos, podría encontrar un conflicto entre la configuración de máquina virtual de ejecución prolongada y la instalación de la extensión, donde ambas necesitan acceso exclusivo al administrador de paquetes.

### <a name="view-extension-status"></a>Consulta del estado de la extensión

Después de que una extensión de máquina virtual se ejecuta en una máquina virtual, use [Get-AzVM ](/powershell/module/az.compute/get-azvm) para devolver el estado de la extensión. *Substatuses[0]* muestra que el aprovisionamiento de la extensión se realizó correctamente, lo que significa que se implementó de manera correcta en la máquina virtual, pero la ejecución de la extensión dentro de la máquina virtual presentó un error, *Substatuses[1]* .

```powershell
Get-AzVM -ResourceGroupName "myResourceGroup" -VMName "myVM" -Status
```

La salida es similar a la salida del ejemplo siguiente:

```powershell
Extensions[0]           :
  Name                  : CustomScriptExtension
  Type                  : Microsoft.Compute.CustomScriptExtension
  TypeHandlerVersion    : 1.9
  Substatuses[0]        :
    Code                : ComponentStatus/StdOut/succeeded
    Level               : Info
    DisplayStatus       : Provisioning succeeded
    Message             : Windows PowerShell \nCopyright (C) Microsoft Corporation. All rights reserved.\n
  Substatuses[1]        :
    Code                : ComponentStatus/StdErr/succeeded
    Level               : Info
    DisplayStatus       : Provisioning succeeded
    Message             : The argument 'cseTest%20Scriptparam1.ps1' to the -File parameter does not exist. Provide the path to an existing '.ps1' file as an argument to the

-File parameter.
  Statuses[0]           :
    Code                : ProvisioningState/failed/-196608
    Level               : Error
    DisplayStatus       : Provisioning failed
    Message             : Finished executing command
```

El estado de ejecución de extensión también puede encontrarse en Azure Portal. Para ver el estado de una extensión, seleccione la máquina virtual, elija **Extensiones** y seleccione la extensión deseada.

### <a name="rerun-vm-extensions"></a>Repetición de ejecución de extensiones de máquina virtual

Puede haber casos en los que sea necesario volver a ejecutar una extensión de máquina virtual. Puede volver ejecutar la extensión si la quita y la vuelve a ejecutar con un método de ejecución de su elección. Para quitar una extensión, use [Remove-AzVMExtension](/powershell/module/az.compute/remove-azvmextension) como se indica a continuación:

```powershell
Remove-AzVMExtension -ResourceGroupName "myResourceGroup" -VMName "myVM" -Name "myExtensionName"
```

También puede quitar una extensión en Azure Portal como se indica a continuación:

1. Seleccione una máquina virtual.
2. Elija **Extensiones**.
3. Seleccione la extensión deseada.
4. Elija **Desinstalar**.

## <a name="common-vm-extensions-reference"></a>Referencia de extensiones de máquina virtual comunes
| Nombre de la extensión | Descripción | Más información |
| --- | --- | --- |
| Extensión de la secuencia de comandos personalizada para Windows |Ejecución de scripts en una máquina virtual de Azure |[Extensión de la secuencia de comandos personalizada para Windows](custom-script-windows.md) |
| Extensión de DSC para Windows |Extensión DSC (configuración de estado deseado) de PowerShell |[Extensión DSC para Windows](dsc-overview.md) |
| Extensión de Diagnósticos de Azure |Administración de Diagnósticos de Azure |[Extensión de Diagnósticos de Azure](https://azure.microsoft.com/blog/windows-azure-virtual-machine-monitoring-with-wad-extension/) |
| Extensión de acceso a máquina virtual de Azure |Administración de usuarios y credenciales |[Extensión de acceso a máquina virtual para Linux](https://azure.microsoft.com/blog/using-vmaccess-extension-to-reset-login-credentials-for-linux-vm/) |

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre las extensiones de máquina virtual, consulte [Características y extensiones de las máquinas virtuales de Azure](overview.md).