---
title: 'Tutorial: Instalación de aplicaciones en un conjunto de escalado con Azure PowerShell'
description: Aprenda a usar Azure PowerShell para instalar aplicaciones en conjuntos de escalado de máquinas virtuales con la extensión de script personalizado.
author: ju-shim
ms.author: jushiman
ms.topic: tutorial
ms.service: virtual-machine-scale-sets
ms.subservice: powershell
ms.date: 11/08/2018
ms.reviewer: mimckitt
ms.custom: mimckitt
ms.openlocfilehash: e783f7f0a9be413679e509e4d6124d50bb811821
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87059285"
---
# <a name="tutorial-install-applications-in-virtual-machine-scale-sets-with-azure-powershell"></a>Tutorial: Instalación de aplicaciones en conjuntos de escalado de máquinas virtuales con Azure PowerShell

Para ejecutar aplicaciones en las instancias de máquinas virtuales (VM) de un conjunto de escalado, primero debe instalar los componentes de la aplicación y los archivos necesarios. En un tutorial anterior, aprendió a crear y usar una imagen de máquina virtual personalizada para implementar las instancias de máquina virtual. Esta imagen personalizada incluía instalaciones y configuraciones manuales de aplicaciones. También puede automatizar la instalación de aplicaciones en un conjunto de escalado después de implementar cada instancia de máquina virtual, o actualizar una aplicación que ya se ejecuta en un conjunto de escalado. En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Instalar automáticamente aplicaciones en un conjunto de escalado
> * Usar la extensión de script personalizado de Azure
> * Actualizar una aplicación en ejecución en un conjunto de escalado

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

[!INCLUDE [updated-for-az.md](../../includes/updated-for-az.md)]

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]


## <a name="what-is-the-azure-custom-script-extension"></a>¿Qué es la extensión de script personalizado de Azure?
La extensión de script personalizado descarga y ejecuta scripts en máquinas virtuales de Azure. Esta extensión es útil para la configuración posterior a la implementación, la instalación de software o cualquier otra tarea de configuración o administración. Los scripts se pueden descargar desde Azure Storage o GitHub, o se pueden proporcionar a Azure Portal en el tiempo de ejecución de la extensión.

La extensión de script personalizado se integra con las plantillas de Azure Resource Manager. También se puede usar con la CLI de Azure, Azure PowerShell, Azure Portal o la API REST. Para obtener más información, consulte [Información general de la extensión de script personalizado](../virtual-machines/extensions/custom-script-windows.md).

Para ver la extensión de script personalizado en acción, cree un conjunto de escalado que instala el servidor web de IIS y genera el nombre de host de la instancia de máquina virtual del conjunto de escalado. La definición de extensión de script personalizado descarga un script de ejemplo de GitHub, instala los paquetes necesarios y, luego, escribe el nombre de host de la instancia de máquina virtual en una página HTML básica.


## <a name="create-a-scale-set"></a>Creación de un conjunto de escalado
Ahora, cree un conjunto de escalado de máquinas virtuales con [New-AzVmss](/powershell/module/az.compute/new-azvmss). Para distribuir el tráfico a las instancias individuales de VM, también se crea un equilibrador de carga. El equilibrador de carga incluye reglas para distribuir el tráfico en el puerto TCP 80. También permite tráfico de escritorio remoto en el puerto TCP 3389 y la comunicación remota de PowerShell en el puerto TCP 5985. Cuando se le solicite, puede establecer sus propias credenciales administrativas para las instancias de máquina virtual del conjunto de escalado:

```azurepowershell-interactive
New-AzVmss `
  -ResourceGroupName "myResourceGroup" `
  -VMScaleSetName "myScaleSet" `
  -Location "EastUS" `
  -VirtualNetworkName "myVnet" `
  -SubnetName "mySubnet" `
  -PublicIpAddressName "myPublicIPAddress" `
  -LoadBalancerName "myLoadBalancer" `
  -UpgradePolicyMode "Automatic"
```

Se tardan unos minutos en crear y configurar todos los recursos de conjunto de escalado y máquinas virtuales.


## <a name="create-custom-script-extension-definition"></a>Creación de una definición de extensión de script personalizado
Azure PowerShell usa una tabla hash para almacenar el archivo que se va a descargar y el comando que se va a ejecutar. En el ejemplo siguiente, se utiliza un script de ejemplo de GitHub. En primer lugar, cree este objeto de configuración de la siguiente manera:

```azurepowershell-interactive
$customConfig = @{
  "fileUris" = (,"https://raw.githubusercontent.com/Azure-Samples/compute-automation-configurations/master/automate-iis.ps1");
  "commandToExecute" = "powershell -ExecutionPolicy Unrestricted -File automate-iis.ps1"
}
```


Ahora, aplique la extensión de script personalizado con [Add-AzVmssExtension](/powershell/module/az.Compute/Add-azVmssExtension). El objeto de configuración previamente definido se pasa a la extensión. Actualice y ejecute la extensión en las instancias de máquina virtual con [Update-AzVmss](/powershell/module/az.compute/update-azvmss).


```azurepowershell-interactive
# Get information about the scale set
$vmss = Get-AzVmss `
          -ResourceGroupName "myResourceGroup" `
          -VMScaleSetName "myScaleSet"

# Add the Custom Script Extension to install IIS and configure basic website
$vmss = Add-AzVmssExtension `
  -VirtualMachineScaleSet $vmss `
  -Name "customScript" `
  -Publisher "Microsoft.Compute" `
  -Type "CustomScriptExtension" `
  -TypeHandlerVersion 1.9 `
  -Setting $customConfig

# Update the scale set and apply the Custom Script Extension to the VM instances
Update-AzVmss `
  -ResourceGroupName "myResourceGroup" `
  -Name "myScaleSet" `
  -VirtualMachineScaleSet $vmss
```

Cada instancia de máquina virtual del conjunto de escalado descarga y ejecuta el script de GitHub. En un ejemplo más complejo, se podrían instalar varios componentes y archivos de aplicaciones. Si el conjunto de escalado se escala verticalmente, las nuevas instancias de máquina virtual aplican automáticamente la misma definición de extensión de script personalizado e instalan la aplicación necesaria.


## <a name="allow-traffic-to-application"></a>Permitir tráfico a la aplicación

Para permitir el acceso a la aplicación web básica, cree un grupo de seguridad de red con [New-AzNetworkSecurityRuleConfig](/powershell/module/az.network/new-aznetworksecurityruleconfig) y [New-AzNetworkSecurityGroup](/powershell/module/az.network/new-aznetworksecuritygroup). Para más información, consulte [Redes para conjuntos de escalado de máquinas virtuales de Azure](virtual-machine-scale-sets-networking.md).

```azurepowershell-interactive

#Create a rule to allow traffic over port 80
$nsgFrontendRule = New-AzNetworkSecurityRuleConfig `
  -Name myFrontendNSGRule `
  -Protocol Tcp `
  -Direction Inbound `
  -Priority 200 `
  -SourceAddressPrefix * `
  -SourcePortRange * `
  -DestinationAddressPrefix * `
  -DestinationPortRange 80 `
  -Access Allow

#Create a network security group and associate it with the rule
$nsgFrontend = New-AzNetworkSecurityGroup `
  -ResourceGroupName  "myResourceGroup" `
  -Location EastUS `
  -Name myFrontendNSG `
  -SecurityRules $nsgFrontendRule

$vnet = Get-AzVirtualNetwork `
  -ResourceGroupName  "myResourceGroup" `
  -Name myVnet

$frontendSubnet = $vnet.Subnets[0]

$frontendSubnetConfig = Set-AzVirtualNetworkSubnetConfig `
  -VirtualNetwork $vnet `
  -Name mySubnet `
  -AddressPrefix $frontendSubnet.AddressPrefix `
  -NetworkSecurityGroup $nsgFrontend

Set-AzVirtualNetwork -VirtualNetwork $vnet

```



## <a name="test-your-scale-set"></a>Prueba del conjunto de escalado
Para ver el servidor web en acción, obtenga la dirección IP pública del equilibrador de carga con [Get-AzPublicIpAddress](/powershell/module/az.network/get-azpublicipaddress). En el ejemplo siguiente se muestra la dirección IP que se creó en el grupo de recursos *myResourceGroup*:

```azurepowershell-interactive
Get-AzPublicIpAddress -ResourceGroupName "myResourceGroup" | Select IpAddress
```

Escriba la dirección IP pública del equilibrador de carga en un explorador web. El equilibrador de carga distribuye el tráfico a una de las instancias de VM, como se muestra en el ejemplo siguiente:

![Página web básica en IIS](media/tutorial-install-apps-powershell/running-iis.png)

Deje el explorador web abierto para que pueda ver una versión actualizada en el paso siguiente.


## <a name="update-app-deployment"></a>Actualización de la implementación de aplicaciones
A lo largo del ciclo de vida de un conjunto de escalado, puede que deba implementar una versión actualizada de la aplicación. Con la extensión de script personalizado, puede hacer referencia a un script de implementación actualizado y, luego, volver a aplicar la extensión al conjunto de escalado. Cuando se creó el conjunto de escalado en un paso anterior, el parámetro `-UpgradePolicyMode` se estableció en *Automatic*. Esta configuración permite que las instancias de máquina virtual del conjunto de escalado se actualicen y apliquen automáticamente la versión más reciente de la aplicación.

Cree una nueva definición de configuración llamada *customConfigv2*. Esta definición ejecuta una versión *v2* actualizada del script de instalación de la aplicación:

```azurepowershell-interactive
$customConfigv2 = @{
  "fileUris" = (,"https://raw.githubusercontent.com/Azure-Samples/compute-automation-configurations/master/automate-iis-v2.ps1");
  "commandToExecute" = "powershell -ExecutionPolicy Unrestricted -File automate-iis-v2.ps1"
}
```

Actualice la configuración de la extensión de script personalizado en las instancias de las máquinas virtuales del conjunto de escalado. La definición *customConfigv2* se usa para aplicar la versión actualizada de la aplicación:

```azurepowershell-interactive
$vmss = Get-AzVmss `
          -ResourceGroupName "myResourceGroup" `
          -VMScaleSetName "myScaleSet"
 
$vmss.VirtualMachineProfile.ExtensionProfile[0].Extensions[0].Settings = $customConfigv2
 
Update-AzVmss `
  -ResourceGroupName "myResourceGroup" `
  -Name "myScaleSet" `
  -VirtualMachineScaleSet $vmss
```

Todas las instancias de máquina virtual del conjunto de escalado se actualizan automáticamente con la versión más reciente de la página web de ejemplo. Para ver la versión actualizada, actualice el sitio web en el explorador:

![Página web actualizada en IIS](media/tutorial-install-apps-powershell/running-iis-updated.png)


## <a name="clean-up-resources"></a>Limpieza de recursos
Para quitar el conjunto de escalado y los recursos adicionales, elimine el grupo de recursos y todos sus recursos con [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup). El parámetro `-Force` confirma que desea eliminar los recursos sin pedir confirmación adicional. El parámetro `-AsJob` devuelve el control a la petición de confirmación sin esperar a que finalice la operación.

```azurepowershell-interactive
Remove-AzResourceGroup -Name "myResourceGroup" -Force -AsJob
```


## <a name="next-steps"></a>Pasos siguientes
En este tutorial, aprendió a instalar y actualizar automáticamente aplicaciones en un conjunto de escalado con Azure PowerShell:

> [!div class="checklist"]
> * Instalar automáticamente aplicaciones en un conjunto de escalado
> * Usar la extensión de script personalizado de Azure
> * Actualizar una aplicación en ejecución en un conjunto de escalado

Vaya al siguiente tutorial para aprender cómo se escala automáticamente el conjunto de escalado.

> [!div class="nextstepaction"]
> [Escalado automático de los conjuntos de escalado](tutorial-autoscale-powershell.md)
