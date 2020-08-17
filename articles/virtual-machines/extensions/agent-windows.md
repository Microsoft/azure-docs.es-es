---
title: Información general del agente de máquina virtual de Azure
description: Información general del agente de máquina virtual de Azure
services: virtual-machines-windows
documentationcenter: virtual-machines
author: mimckitt
manager: gwallace
tags: azure-resource-manager
ms.assetid: 0a1f212e-053e-4a39-9910-8d622959f594
ms.service: virtual-machines-windows
ms.topic: article
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure-services
ms.date: 07/20/2019
ms.author: akjosh
ms.openlocfilehash: 6ff5825f3272f0dadc74147d36e8c5fd8e7838d7
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87010961"
---
# <a name="azure-virtual-machine-agent-overview"></a>Información general del agente de máquina virtual de Azure
El agente de máquina virtual de Microsoft Azure (agente VM) es un proceso ligero y seguro que administra la interacción de máquinas virtuales (VM) con el controlador de tejido de Azure. El agente de VM tiene un rol principal que consiste en habilitar y ejecutar extensiones de máquina virtual de Azure. Las extensiones de máquina virtual habilitan la configuración posterior a la implementación de máquinas virtuales, como la instalación y la configuración de software. Las extensiones de máquina virtual también habilitan características de recuperación, como el restablecimiento de la contraseña administrativa de una máquina virtual. Sin el agente de máquina virtual de Azure, no se pueden ejecutar extensiones de máquina virtual.

En este artículo se describe la instalación y detección del agente de máquina virtual de Azure.

## <a name="install-the-vm-agent"></a>Instalar el agente de VM

### <a name="azure-marketplace-image"></a>Imagen de Azure Marketplace

El agente de máquina virtual de Azure se instala de forma predeterminada en cualquier máquina virtual de Windows implementada a partir de una imagen de Azure Marketplace. Al implementar una imagen de Azure Marketplace desde el portal, PowerShell, la interfaz de la línea de comandos o una plantilla de Azure Resource Manager, también se instalará el agente de máquina virtual de Azure.

El paquete de Windows Guest Agent se divide en dos partes:

- Agente de aprovisionamiento (PA)
- Windows Guest Agent (WinGA)

Para arrancar una máquina virtual debe tener el agente de aprovisionamiento instalado en la máquina virtual, pero no es necesario instalar WinGA. En tiempo de implementación de la máquina virtual, puede seleccionar no instalar WinGA. En el ejemplo siguiente se muestra cómo seleccionar la opción *provisionVmAgent* con una plantilla de Azure Resource Manager:

```json
"resources": [{
"name": "[parameters('virtualMachineName')]",
"type": "Microsoft.Compute/virtualMachines",
"apiVersion": "2016-04-30-preview",
"location": "[parameters('location')]",
"dependsOn": ["[concat('Microsoft.Network/networkInterfaces/', parameters('networkInterfaceName'))]"],
"properties": {
    "osProfile": {
    "computerName": "[parameters('virtualMachineName')]",
    "adminUsername": "[parameters('adminUsername')]",
    "adminPassword": "[parameters('adminPassword')]",
    "windowsConfiguration": {
        "provisionVmAgent": "false"
}
```

Si no tiene instalados los agentes, no puede usar algunos servicios de Azure, como Azure Backup o Azure Security. Estos servicios requieren una extensión para instalarse. Si ha implementado una máquina virtual sin WinGA, puede instalar más tarde la versión más reciente del agente.

### <a name="manual-installation"></a>Instalación manual
El agente de máquina virtual de Windows puede instalarse manualmente con un paquete de Windows Installer. Es posible que sea necesaria la instalación manual cuando se crea una imagen de máquina virtual personalizada que se implementa en Azure. Para instalar manualmente el agente de máquina virtual de Windows, [descargue el instalador del agente de máquina virtual](https://go.microsoft.com/fwlink/?LinkID=394789). El Agente de máquina virtual solo se admite en Windows Server 2008 (64 bits) y posterior.

> [!NOTE]
> Es importante actualizar la opción AllowExtensionOperations después de instalar manualmente VMAgent en una máquina virtual que se ha implementado a partir de una imagen sin ProvisionVMAgent habilitado.

```powershell
$vm.OSProfile.AllowExtensionOperations = $true
$vm | Update-AzVM
```

### <a name="prerequisites"></a>Requisitos previos
- El Agente de máquina virtual de Windows necesita como mínimo Windows Server 2008 (64 bits) para ejecutarse, con .NET Framework 4.0. Consulte [Versión mínima admitida para los agentes de la máquina virtual en Azure](https://support.microsoft.com/en-us/help/4049215/extensions-and-virtual-machine-agent-minimum-version-support).

- Asegúrese de que la máquina virtual tenga acceso a la dirección IP 168.63.129.16. Para más información, vea [¿Qué es la dirección IP 168.63.129.16?](../../virtual-network/what-is-ip-address-168-63-129-16.md).

## <a name="detect-the-vm-agent"></a>Detección del agente de VM

### <a name="powershell"></a>PowerShell

El módulo de PowerShell de Azure Resource Manager puede usarse para recuperar información sobre máquinas virtuales de Azure. Para ver información sobre una VM, como el estado de aprovisionamiento para el agente de VM de Azure, use [Get-AzVM](/powershell/module/az.compute/get-azvm):

```powershell
Get-AzVM
```

El siguiente ejemplo condensado de salida muestra la propiedad *ProvisionVMAgent* anidada dentro de *OSProfile*. Esta propiedad se puede usar para determinar si el agente de VM se ha implementado en la VM:

```powershell
OSProfile                  :
  ComputerName             : myVM
  AdminUsername            : myUserName
  WindowsConfiguration     :
    ProvisionVMAgent       : True
    EnableAutomaticUpdates : True
```

El siguiente script se puede usar para devolver una lista concisa de nombres de máquinas virtuales y el estado del agente de máquina virtual:

```powershell
$vms = Get-AzVM

foreach ($vm in $vms) {
    $agent = $vm | Select -ExpandProperty OSProfile | Select -ExpandProperty Windowsconfiguration | Select ProvisionVMAgent
    Write-Host $vm.Name $agent.ProvisionVMAgent
}
```

### <a name="manual-detection"></a>Detección manual

Cuando inicia sesión en una máquina virtual de Windows, el Administrador de tareas se puede usar para examinar los procesos en ejecución. Para comprobar el agente de máquina virtual de Azure, abra el Administrador de tareas, haga clic en la pestaña *Detalles* y busque el nombre de proceso **WindowsAzureGuestAgent.exe**. La presencia de este proceso indica que el agente de VM está instalado.


## <a name="upgrade-the-vm-agent"></a>Actualización del agente de VM
El agente de VM de Azure para Windows se actualiza automáticamente. A medida que se implementan nuevas máquinas virtuales en Azure, reciben el agente de máquina virtual más reciente en tiempo de aprovisionamiento de máquina virtual. Las imágenes de máquina virtual personalizadas se deben actualizar manualmente para que incluyan el nuevo agente de máquina virtual en tiempo de creación de la imagen.

## <a name="windows-guest-agent-automatic-logs-collection"></a>Recopilación de registros automáticos de Windows Guest Agent
Windows Guest Agent tiene una característica para recopilar automáticamente algunos registros. Esta característica está controlada por el proceso CollectGuestLogs.exe. Existe tanto para los servicios en la nube PaaS como para las máquinas virtuales de IaaS, y su objetivo es recopilar rápida y automáticamente algunos registros de diagnóstico de una máquina virtual, de modo que se puedan usar para realizar un análisis sin conexión. Los registros recopilados son registros de eventos, registros del sistema operativo, registros de Azure y algunas claves del Registro. Se genera un archivo ZIP que se transfiere al host de la máquina virtual. Este archivo ZIP puede ser consultado por los equipos de ingeniería y los profesionales de soporte técnico para investigar problemas en la solicitud del cliente propietario de la máquina virtual.

## <a name="next-steps"></a>Pasos siguientes
Para más información sobre las extensiones de máquina virtual, consulte [Características y extensiones de las máquinas virtuales de Azure](overview.md).
