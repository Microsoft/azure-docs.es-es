---
title: 'Tutorial: Creación de máquinas virtuales que ejecutan una pila SQL, IIS y .NET en Azure'
description: En este tutorial, aprenderá a instalar la pila Azure SQL, IIS, .NET en una máquina virtual Windows en Azure.
author: cynthn
ms.service: virtual-machines-windows
ms.topic: tutorial
ms.workload: infrastructure
ms.date: 12/05/2018
ms.author: cynthn
ms.custom: mvc
ms.openlocfilehash: 1c53194bd345c18ac582acd538f1e8f8e1e34d54
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87027859"
---
# <a name="tutorial-install-the-sql-iis-net-stack-in-a-windows-vm-with-azure-powershell"></a>Tutorial: Instalación de la pila SQL, IIS y .NET en una máquina virtual Windows con Azure PowerShell

En este tutorial, vamos a instalar una pila SQL, IIS y .NET con Azure PowerShell. Esta pila está formada por dos máquinas virtuales que ejecutan Windows Server 2016, una con IIS y .NET y la otra con SQL Server.

> [!div class="checklist"]
> * Crear una VM 
> * Instalación de IIS y el SDK de .NET Core en la máquina virtual
> * Creación de una máquina virtual que ejecute SQL Server
> * Instalación de la extensión de SQL Server

## <a name="launch-azure-cloud-shell"></a>Inicio de Azure Cloud Shell

Azure Cloud Shell es un shell interactivo gratuito que puede usar para ejecutar los pasos de este artículo. Tiene las herramientas comunes de Azure preinstaladas y configuradas para usarlas en la cuenta. 

Para abrir Cloud Shell, seleccione **Pruébelo** en la esquina superior derecha de un bloque de código. También puede ir a [https://shell.azure.com/powershell](https://shell.azure.com/powershell) para iniciar Cloud Shell en una pestaña independiente del explorador. Seleccione **Copiar** para copiar los bloques de código, péguelos en Cloud Shell y, luego, presione Entrar para ejecutarlos.

## <a name="create-an-iis-vm"></a>Creación de una máquina virtual de IIS 

En este ejemplo, usamos el cmdlet [New-AzVM](/powershell/module/az.compute/new-azvm) en PowerShell Cloud Shell para crear rápidamente una máquina virtual Windows Server 2016 y, a continuación, instalar IIS y .NET Framework. Las máquinas virtuales con IIS y SQL comparten un grupo de recursos y una red virtual, por lo que creamos variables para esos nombres.


```azurepowershell-interactive
$vmName = "IISVM"
$vNetName = "myIISSQLvNet"
$resourceGroup = "myIISSQLGroup"
New-AzVm `
    -ResourceGroupName $resourceGroup `
    -Name $vmName `
    -Location "East US" `
    -VirtualNetworkName $vNetName `
    -SubnetName "myIISSubnet" `
    -SecurityGroupName "myNetworkSecurityGroup" `
    -AddressPrefix 192.168.0.0/16 `
    -PublicIpAddressName "myIISPublicIpAddress" `
    -OpenPorts 80,3389 
```

Instale IIS y .NET Framework utilizando la extensión de script personalizado con el cmdlet [Set-AzVMExtension](/powershell/module/az.compute/set-azvmextension).

```azurepowershell-interactive
Set-AzVMExtension `
    -ResourceGroupName $resourceGroup `
    -ExtensionName IIS `
    -VMName $vmName `
    -Publisher Microsoft.Compute `
    -ExtensionType CustomScriptExtension `
    -TypeHandlerVersion 1.4 `
    -SettingString '{"commandToExecute":"powershell Add-WindowsFeature Web-Server,Web-Asp-Net45,NET-Framework-Features"}' `
    -Location EastUS
```

## <a name="create-another-subnet"></a>Creación de otra subred

Cree una segunda subred para la máquina virtual de SQL. Obtenga la red virtual mediante [Get-AzVirtualNetwork]{/powershell/module/az.network/get-azvirtualnetwork}.

```azurepowershell-interactive
$vNet = Get-AzVirtualNetwork `
   -Name $vNetName `
   -ResourceGroupName $resourceGroup
```

Cree una configuración para la subred con [Add-AzVirtualNetworkSubnetConfig](/powershell/module/az.network/add-azvirtualnetworksubnetconfig).


```azurepowershell-interactive
Add-AzVirtualNetworkSubnetConfig `
   -AddressPrefix 192.168.0.0/24 `
   -Name mySQLSubnet `
   -VirtualNetwork $vNet `
   -ServiceEndpoint Microsoft.Sql
```

Actualice la red virtual con la nueva información de subred mediante [Set-AzVirtualNetwork](/powershell/module/az.network/set-azvirtualnetwork).
   
```azurepowershell-interactive   
$vNet | Set-AzVirtualNetwork
```

## <a name="azure-sql-vm"></a>Máquina virtual de Azure SQL

Utilice una imagen de Marketplace de Azure configurada previamente de un servidor SQL Server para crear la máquina virtual de SQL. Primero creamos la máquina virtual y, a continuación, vamos a instalar la extensión de SQL Server en la máquina virtual. 


```azurepowershell-interactive
New-AzVm `
    -ResourceGroupName $resourceGroup `
    -Name "mySQLVM" `
    -ImageName "MicrosoftSQLServer:SQL2016SP1-WS2016:Enterprise:latest" `
    -Location eastus `
    -VirtualNetworkName $vNetName `
    -SubnetName "mySQLSubnet" `
    -SecurityGroupName "myNetworkSecurityGroup" `
    -PublicIpAddressName "mySQLPublicIpAddress" `
    -OpenPorts 3389,1401 
```

Use [Set-AzVMSqlServerExtension](/powershell/module/az.compute/set-azvmsqlserverextension) para agregar la [extensión de SQL Server](../../azure-sql/virtual-machines/windows/sql-server-iaas-agent-extension-automate-management.md) a la máquina virtual con SQL.

```azurepowershell-interactive
Set-AzVMSqlServerExtension `
   -ResourceGroupName $resourceGroup  `
   -VMName mySQLVM `
   -Name "SQLExtension" `
   -Location "EastUS"
```

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha instalado una pila SQL&#92;IIS&#92;.NET con Azure PowerShell. Ha aprendido a:

> [!div class="checklist"]
> * Crear una VM 
> * Instalación de IIS y el SDK de .NET Core en la máquina virtual
> * Creación de una máquina virtual que ejecute SQL Server
> * Instalación de la extensión de SQL Server

Pase al siguiente tutorial para aprender a proteger un servidor web IIS con certificados TLS/SSL.

> [!div class="nextstepaction"]
> [Protección de un servidor web IIS con certificados TLS/SSL](tutorial-secure-web-server.md)
