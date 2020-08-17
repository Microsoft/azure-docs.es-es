---
title: Recopilar detalles sobre todas las máquinas virtuales de una suscripción con PowerShell
description: Recopilar detalles sobre todas las máquinas virtuales de una suscripción con PowerShell
services: virtual-machines-windows
documentationcenter: virtual-machines
author: v-miegge
manager: dcscontentpm
editor: v-miegge
tags: azure-service-management
ms.assetid: ''
ms.service: virtual-machines-windows
ms.topic: troubleshooting
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure
ms.date: 07/01/2019
ms.author: v-miegge
ms.custom: mvc
ms.openlocfilehash: 8d61da9c3a3d575fcbce0b77e1f8b2684df58106
ms.sourcegitcommit: d7bd8f23ff51244636e31240dc7e689f138c31f0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/24/2020
ms.locfileid: "87171690"
---
# <a name="collect-details-about-all-vms-in-a-subscription-with-powershell"></a>Recopilar detalles sobre todas las máquinas virtuales de una suscripción con PowerShell

Este script crea un csv que contiene el nombre de la máquina virtual, el nombre del grupo de recursos, la región, el tamaño de VM, la red virtual, la subred, la dirección IP privada, el tipo de sistema operativo y la dirección IP pública de las máquinas virtuales de la suscripción proporcionada.

Si no tiene una [suscripción a Azure](../../guides/developer/azure-developer-guide.md#understanding-accounts-subscriptions-and-billing), cree una [cuenta gratuita](https://azure.microsoft.com/free) antes de empezar.

## <a name="launch-azure-cloud-shell"></a>Inicio de Azure Cloud Shell

Azure Cloud Shell es un shell interactivo gratuito que puede usar para ejecutar los pasos de este artículo. Tiene las herramientas comunes de Azure preinstaladas y configuradas para usarlas en la cuenta. 

Para abrir Cloud Shell, seleccione **Pruébelo** en la esquina superior derecha de un bloque de código. También puede ir a [https://shell.azure.com/powershell](https://shell.azure.com/powershell) para iniciar Cloud Shell en una pestaña independiente del explorador. Seleccione **Copiar** para copiar los bloques de código, péguelos en Cloud Shell y, luego, presione Entrar para ejecutarlos.

## <a name="sample-script"></a>Script de ejemplo

```azurepowershell-interactive
#Provide the subscription Id where the VMs reside
$subscriptionId = "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"

#Provide the name of the csv file to be exported
$reportName = "myReport.csv"

Select-AzSubscription $subscriptionId
$report = @()
$vms = Get-AzVM
$publicIps = Get-AzPublicIpAddress 
$nics = Get-AzNetworkInterface | ?{ $_.VirtualMachine -NE $null} 
foreach ($nic in $nics) { 
    $info = "" | Select VmName, ResourceGroupName, Region, VmSize, VirtualNetwork, Subnet, PrivateIpAddress, OsType, PublicIPAddress, NicName, ApplicationSecurityGroup 
    $vm = $vms | ? -Property Id -eq $nic.VirtualMachine.id 
    foreach($publicIp in $publicIps) { 
        if($nic.IpConfigurations.id -eq $publicIp.ipconfiguration.Id) {
            $info.PublicIPAddress = $publicIp.ipaddress
            } 
        } 
        $info.OsType = $vm.StorageProfile.OsDisk.OsType 
        $info.VMName = $vm.Name 
        $info.ResourceGroupName = $vm.ResourceGroupName 
        $info.Region = $vm.Location 
        $info.VmSize = $vm.HardwareProfile.VmSize
        $info.VirtualNetwork = $nic.IpConfigurations.subnet.Id.Split("/")[-3] 
        $info.Subnet = $nic.IpConfigurations.subnet.Id.Split("/")[-1] 
        $info.PrivateIpAddress = $nic.IpConfigurations.PrivateIpAddress 
        $info.NicName = $nic.Name 
        $info.ApplicationSecurityGroup = $nic.IpConfigurations.ApplicationSecurityGroups.Id 
        $report+=$info 
    } 
$report | ft VmName, ResourceGroupName, Region, VmSize, VirtualNetwork, Subnet, PrivateIpAddress, OsType, PublicIPAddress, NicName, ApplicationSecurityGroup 
$report | Export-CSV "$home/$reportName"
```

## <a name="script-explanation"></a>Explicación del script
Este script usa los siguientes comandos para crear una exportación csv de los detalles de las máquinas virtuales de una suscripción. Cada comando de la tabla crea un vínculo a documentación específica del comando.

|Get-Help|Notas|
|-|-|
|[Select-AzSubscription](/powershell/module/az.accounts/set-azcontext)|Establece el inquilino, la suscripción y el entorno de los cmdlets que se usan en la sesión actual.|
|[Get-AzVM](/powershell/module/az.compute/get-azvm)|Obtiene las propiedades de una máquina virtual.|
|[Get-AzPublicIpAddress](/powershell/module/az.network/get-azpublicipaddress)|Obtiene una dirección IP pública.|
|[Get-AzNetworkInterface](/powershell/module/az.network/get-aznetworkinterface)|Obtiene una interfaz de red.|

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información sobre el módulo de Azure PowerShell, consulte la [documentación de Azure PowerShell](/powershell/azure/).

Encontrará más ejemplos de scripts de Azure PowerShell de máquina virtual en la [documentación sobre máquinas virtuales Windows de Azure](../windows/powershell-samples.md?toc=/azure/virtual-machines/windows/toc.json).
