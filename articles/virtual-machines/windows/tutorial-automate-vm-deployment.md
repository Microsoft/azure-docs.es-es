---
title: 'Tutorial: Instalación de aplicaciones en una máquina virtual Windows en Azure'
description: En este tutorial, aprenderá a usar la extensión de script personalizado para ejecutar scripts e implementar aplicaciones en máquinas virtuales Windows en Azure.
author: cynthn
ms.service: virtual-machines-windows
ms.topic: tutorial
ms.workload: infrastructure
ms.date: 11/29/2018
ms.author: cynthn
ms.custom: mvc
ms.openlocfilehash: 14d0190a97c22a805065ceaf41dcd655b9e8182b
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87065283"
---
# <a name="tutorial---deploy-applications-to-a-windows-virtual-machine-in-azure-with-the-custom-script-extension"></a>Tutorial: Implementación de aplicaciones en una máquina virtual Windows en Azure con la extensión de script personalizado

Para configurar las máquinas virtuales (VM) de una manera rápida y coherente, puede usar la [extensión de script personalizado para Windows](../extensions/custom-script-windows.md). En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Usar la extensión de script personalizada para instalar IIS
> * Crear una máquina virtual que use la extensión de script personalizada
> * Ver un sitio IIS en funcionamiento después de aplicar la extensión

## <a name="launch-azure-cloud-shell"></a>Inicio de Azure Cloud Shell

Azure Cloud Shell es un shell interactivo gratuito que puede usar para ejecutar los pasos de este artículo. Tiene las herramientas comunes de Azure preinstaladas y configuradas para usarlas en la cuenta. 

Para abrir Cloud Shell, seleccione **Pruébelo** en la esquina superior derecha de un bloque de código. También puede ir a [https://shell.azure.com/powershell](https://shell.azure.com/powershell) para iniciar Cloud Shell en una pestaña independiente del explorador. Seleccione **Copiar** para copiar los bloques de código, péguelos en Cloud Shell y, luego, presione Entrar para ejecutarlos.

## <a name="custom-script-extension-overview"></a>Información general de la extensión de script personalizado
La extensión de script personalizado descarga y ejecuta scripts en máquinas virtuales de Azure. Esta extensión es útil para la configuración posterior a la implementación, la instalación de software o cualquier otra tarea de configuración o administración. Los scripts se pueden descargar desde Azure Storage o GitHub, o se pueden proporcionar a Azure Portal en el tiempo de ejecución de la extensión.

La extensión de script personalizado se integra con las plantillas de Azure Resource Manager y también se puede ejecutar mediante la CLI de Azure, PowerShell, Azure Portal o la API de REST de máquina virtual de Azure.

Se puede usar la extensión de script personalizado con máquinas virtuales de Linux y Windows.


## <a name="create-virtual-machine"></a>Crear máquina virtual
Establezca un nombre de usuario de administrador y una contraseña para la máquina virtual con [Get-Credential](/powershell/module/microsoft.powershell.security/get-credential?view=powershell-5.1):

```azurepowershell-interactive
$cred = Get-Credential
```

Ahora puede crear la máquina virtual con [New-AzVM](/powershell/module/az.compute/new-azvm). En el ejemplo siguiente se crea una máquina virtual denominada *myVM* en la ubicación *EastUS*. Si aún no existen, se crean el grupo de recursos *myResourceGroupAutomate* y los recursos de red de soporte. Para permitir el tráfico de web, el cmdlet también abre el puerto *80*.

```azurepowershell-interactive
New-AzVm `
    -ResourceGroupName "myResourceGroupAutomate" `
    -Name "myVM" `
    -Location "East US" `
    -VirtualNetworkName "myVnet" `
    -SubnetName "mySubnet" `
    -SecurityGroupName "myNetworkSecurityGroup" `
    -PublicIpAddressName "myPublicIpAddress" `
    -OpenPorts 80 `
    -Credential $cred
```

Los recursos y la máquina virtual tardan unos minutos en crearse.


## <a name="automate-iis-install"></a>Automatizar la instalación de IIS
Use [Set-AzVMExtension](/powershell/module/az.compute/set-azvmextension) para instalar la extensión de script personalizado. La extensión ejecuta `powershell Add-WindowsFeature Web-Server` para instalar el servidor web de IIS y después actualiza la página *Default.htm* para mostrar el nombre de host de la máquina virtual:

```azurepowershell-interactive
Set-AzVMExtension -ResourceGroupName "myResourceGroupAutomate" `
    -ExtensionName "IIS" `
    -VMName "myVM" `
    -Location "EastUS" `
    -Publisher Microsoft.Compute `
    -ExtensionType CustomScriptExtension `
    -TypeHandlerVersion 1.8 `
    -SettingString '{"commandToExecute":"powershell Add-WindowsFeature Web-Server; powershell Add-Content -Path \"C:\\inetpub\\wwwroot\\Default.htm\" -Value $($env:computername)"}'
```


## <a name="test-web-site"></a>Sitio web de prueba
Obtenga la dirección IP pública del equilibrador de carga con [Get-AzPublicIPAddress](/powershell/module/az.network/get-azpublicipaddress). En el ejemplo siguiente se obtiene la dirección IP de *myPublicIPAddress* que se ha creado anteriormente:

```azurepowershell-interactive
Get-AzPublicIPAddress `
    -ResourceGroupName "myResourceGroupAutomate" `
    -Name "myPublicIPAddress" | select IpAddress
```

A continuación, puede escribir la dirección IP pública en un explorador web. Se muestra el sitio web, incluido el nombre de host de la máquina virtual a la que el equilibrador de carga distribuye el tráfico como en el ejemplo siguiente:

![Ejecución del sitio web de IIS](./media/tutorial-automate-vm-deployment/running-iis-website.png)


## <a name="next-steps"></a>Pasos siguientes

En este tutorial, automatizó la instalación IIS en una máquina virtual. Ha aprendido a:

> [!div class="checklist"]
> * Usar la extensión de script personalizada para instalar IIS
> * Crear una máquina virtual que use la extensión de script personalizada
> * Ver un sitio IIS en funcionamiento después de aplicar la extensión

Avanzar al siguiente tutorial para aprender a crear imágenes de máquina virtual personalizadas.

> [!div class="nextstepaction"]
> [Creación de imágenes personalizadas de máquinas virtuales](./tutorial-custom-images.md)
