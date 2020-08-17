---
title: 'Tutorial: Creación de un conjunto de escalado de máquinas virtuales Windows'
description: Aprenda a usar Azure PowerShell para crear e implementar una aplicación de alta disponibilidad en máquinas virtuales Windows mediante un conjunto de escalado de máquinas virtuales.
author: ju-shim
ms.author: jushiman
ms.topic: tutorial
ms.service: virtual-machine-scale-sets
ms.subservice: windows
ms.date: 11/30/2018
ms.reviewer: mimckitt
ms.custom: mimckitt
ms.openlocfilehash: c5bcc1f2de379eb9a8a49a2259533bcea33328d1
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87265201"
---
# <a name="tutorial-create-a-virtual-machine-scale-set-and-deploy-a-highly-available-app-on-windows-with-azure-powershell"></a>Tutorial: Creación de un conjunto de escalado de máquinas virtuales e implementación de una aplicación de alta disponibilidad en Windows con Azure PowerShell
Un conjunto de escalado de máquinas virtuales le permite implementar y administrar un conjunto de máquinas virtuales de escalado automático idénticas. Puede escalar el número de máquinas virtuales del conjunto de escalado de forma manual. También puede definir reglas de escalado automático en función del uso de recursos tales como la CPU, la demanda de memoria o el tráfico de red. En este tutorial, implementará un conjunto de escalado de máquinas virtuales en Azure y aprenderá cómo:

> [!div class="checklist"]
> * Usar la extensión de script personalizada para definir un sitio IIS para escalar
> * Crear un equilibrador de carga para el conjunto de escalado
> * Crear un conjunto de escalado de máquinas virtuales
> * Aumentar o disminuir el número de instancias en un conjunto de escalado
> * Crear reglas de escalado automático

## <a name="launch-azure-cloud-shell"></a>Inicio de Azure Cloud Shell

Azure Cloud Shell es un shell interactivo gratuito que puede usar para ejecutar los pasos de este artículo. Tiene las herramientas comunes de Azure preinstaladas y configuradas para usarlas en la cuenta. 

Para abrir Cloud Shell, seleccione **Pruébelo** en la esquina superior derecha de un bloque de código. También puede ir a [https://shell.azure.com/powershell](https://shell.azure.com/powershell) para iniciar Cloud Shell en una pestaña independiente del explorador. Seleccione **Copiar** para copiar los bloques de código, péguelos en Cloud Shell y, luego, presione Entrar para ejecutarlos.

## <a name="scale-set-overview"></a>Introducción al conjunto de escalado
Un conjunto de escalado de máquinas virtuales le permite implementar y administrar un conjunto de máquinas virtuales de escalado automático idénticas. Las máquinas virtuales de un conjunto de escalado se distribuyen en dominios lógicos de error y de actualización en uno o más *grupos de selección de ubicación*. Los grupos de selección de ubicación son grupos de máquinas virtuales configuradas de manera similar, al igual que los [conjuntos de disponibilidad](tutorial-availability-sets.md).

Las máquinas virtuales se crean según sea necesario en un conjunto de escalado. Defina reglas de escalado automático para controlar cómo y cuándo se agregan o se quitan las máquinas virtuales del conjunto de escalado. Estas reglas se pueden desencadenar en función de métricas como la carga de la CPU, el uso de la memoria o el tráfico de red.

Los conjuntos de escalado admiten hasta 1000 máquinas virtuales cuando se usa una imagen de la plataforma de Azure. Para las cargas de trabajo con requisitos de personalización de VM o instalación significativos, puede que desee [crear una imagen de VM personalizada](tutorial-custom-images.md). Puede crear hasta 600 máquinas virtuales en un conjunto de escalado al usar una imagen personalizada.


## <a name="create-a-scale-set"></a>Creación de un conjunto de escalado
Cree un conjunto de escalado de máquinas virtuales con [New-AzVmss](/powershell/module/az.compute/new-azvmss). En el ejemplo siguiente, se crea un conjunto de escalado denominado *myScaleSet* que usa la imagen de plataforma *Windows Server 2016 Datacenter*. Los recursos de red de Azure para una red virtual, una dirección IP pública y un equilibrador de carga se crean automáticamente. Cuando se le solicite, puede establecer sus propias credenciales administrativas para las instancias de máquina virtual del conjunto de escalado:

```azurepowershell-interactive
New-AzVmss `
  -ResourceGroupName "myResourceGroupScaleSet" `
  -Location "EastUS" `
  -VMScaleSetName "myScaleSet" `
  -VirtualNetworkName "myVnet" `
  -SubnetName "mySubnet" `
  -PublicIpAddressName "myPublicIPAddress" `
  -LoadBalancerName "myLoadBalancer" `
  -UpgradePolicyMode "Automatic"
```

Se tardan unos minutos en crear y configurar todos los recursos de conjunto de escalado y máquinas virtuales.


## <a name="deploy-sample-application"></a>Implementación de una aplicación de ejemplo
Para probar el conjunto de escalado, instale una aplicación web básica. La extensión de script personalizado de Azure se usa para descargar y ejecutar un script que instala IIS en las instancias de máquina virtual. Esta extensión es útil para la configuración posterior a la implementación, la instalación de software o cualquier otra tarea de configuración o administración. Para obtener más información, consulte [Información general de la extensión de script personalizado](../extensions/custom-script-windows.md).

Use la extensión de script personalizado para instalar un servidor web de IIS básico. Aplique la extensión de script personalizada que instala IIS según se indica a continuación:

```azurepowershell-interactive
# Define the script for your Custom Script Extension to run
$publicSettings = @{
    "fileUris" = (,"https://raw.githubusercontent.com/Azure-Samples/compute-automation-configurations/master/automate-iis.ps1");
    "commandToExecute" = "powershell -ExecutionPolicy Unrestricted -File automate-iis.ps1"
}

# Get information about the scale set
$vmss = Get-AzVmss `
  -ResourceGroupName "myResourceGroupScaleSet" `
  -VMScaleSetName "myScaleSet"

# Use Custom Script Extension to install IIS and configure basic website
Add-AzVmssExtension -VirtualMachineScaleSet $vmss `
  -Name "customScript" `
  -Publisher "Microsoft.Compute" `
  -Type "CustomScriptExtension" `
  -TypeHandlerVersion 1.8 `
  -Setting $publicSettings

# Update the scale set and apply the Custom Script Extension to the VM instances
Update-AzVmss `
  -ResourceGroupName "myResourceGroupScaleSet" `
  -Name "myScaleSet" `
  -VirtualMachineScaleSet $vmss
```

## <a name="allow-traffic-to-application"></a>Permitir tráfico a la aplicación

Para permitir el acceso a la aplicación web básica, cree un grupo de seguridad de red con [New-AzNetworkSecurityRuleConfig](/powershell/module/az.network/new-aznetworksecurityruleconfig) y [New-AzNetworkSecurityGroup](/powershell/module/az.network/new-aznetworksecuritygroup). Para más información, consulte [Redes para conjuntos de escalado de máquinas virtuales de Azure](../../virtual-machine-scale-sets/virtual-machine-scale-sets-networking.md).

```azurepowershell-interactive
# Get information about the scale set
$vmss = Get-AzVmss `
  -ResourceGroupName "myResourceGroupScaleSet" `
  -VMScaleSetName "myScaleSet"

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
  -ResourceGroupName  "myResourceGroupScaleSet" `
  -Location EastUS `
  -Name myFrontendNSG `
  -SecurityRules $nsgFrontendRule

$vnet = Get-AzVirtualNetwork `
  -ResourceGroupName  "myResourceGroupScaleSet" `
  -Name myVnet

$frontendSubnet = $vnet.Subnets[0]

$frontendSubnetConfig = Set-AzVirtualNetworkSubnetConfig `
  -VirtualNetwork $vnet `
  -Name mySubnet `
  -AddressPrefix $frontendSubnet.AddressPrefix `
  -NetworkSecurityGroup $nsgFrontend

Set-AzVirtualNetwork -VirtualNetwork $vnet

# Update the scale set and apply the Custom Script Extension to the VM instances
Update-AzVmss `
  -ResourceGroupName "myResourceGroupScaleSet" `
  -Name "myScaleSet" `
  -VirtualMachineScaleSet $vmss
```

## <a name="test-your-scale-set"></a>Prueba del conjunto de escalado
Para ver el conjunto de escalado en acción, obtenga la dirección IP pública del equilibrador de carga con [Get-AzPublicIPAddress](/powershell/module/az.network/get-azpublicipaddress). En el ejemplo siguiente se muestra la dirección IP de *myPublicIP* que se ha creado como parte del conjunto de escalado:

```azurepowershell-interactive
Get-AzPublicIPAddress `
  -ResourceGroupName "myResourceGroupScaleSet" `
  -Name "myPublicIPAddress" | select IpAddress
```

Escriba la dirección IP pública en un explorador web. Se muestra la aplicación web, incluido el nombre de host de la máquina virtual a la que el equilibrador de carga distribuye el tráfico:

![Ejecución del sitio de IIS](./media/tutorial-create-vmss/running-iis-site.png)

Para ver el conjunto de escalado en funcionamiento, realice una actualización forzada del explorador web para ver cómo el equilibrador de carga distribuye el tráfico entre las máquinas virtuales del conjunto de escalado que ejecutan la aplicación.


## <a name="management-tasks"></a>Tareas de administración
Durante el ciclo de vida del conjunto de escalado, debe ejecutar una o varias tareas de administración. Además, puede crear scripts para automatizar varias tareas de ciclo de vida. Azure PowerShell proporciona una forma rápida de realizar esas tareas. A continuación, presentamos algunas tareas comunes.

### <a name="view-vms-in-a-scale-set"></a>Visualización de máquinas virtuales en un conjunto de escalado
Para ver una lista de las instancias de máquina virtual en un conjunto de escalado, use [Get-AzVmssVM](/powershell/module/az.compute/get-azvmssvm) de la forma siguiente:

```azurepowershell-interactive
Get-AzVmssVM `
  -ResourceGroupName "myResourceGroupScaleSet" `
  -VMScaleSetName "myScaleSet"
```

La salida del ejemplo siguiente muestra dos instancias de máquina virtual del conjunto de escalado:

```powershell
ResourceGroupName                 Name Location             Sku InstanceID ProvisioningState
-----------------                 ---- --------             --- ---------- -----------------
MYRESOURCEGROUPSCALESET   myScaleSet_0   eastus Standard_DS1_v2          0         Succeeded
MYRESOURCEGROUPSCALESET   myScaleSet_1   eastus Standard_DS1_v2          1         Succeeded
```

Para ver información adicional acerca de una instancia específica de la máquina virtual, agregue el parámetro `-InstanceId` a [Get-AzVmssVM](/powershell/module/az.compute/get-azvmssvm). En el ejemplo siguiente, se ve información sobre la instancia de máquina virtual *1*:

```azurepowershell-interactive
Get-AzVmssVM `
  -ResourceGroupName "myResourceGroupScaleSet" `
  -VMScaleSetName "myScaleSet" `
  -InstanceId "1"
```


### <a name="increase-or-decrease-vm-instances"></a>Aumento o disminución de instancias de máquina virtual
Para ver el número de instancias que tiene actualmente en un conjunto de escalado, use [Get-AzVmss](/powershell/module/az.compute/get-azvmss) y realice consultas a *sku.capacity*:

```azurepowershell-interactive
Get-AzVmss -ResourceGroupName "myResourceGroupScaleSet" `
  -VMScaleSetName "myScaleSet" | `
  Select -ExpandProperty Sku
```

A continuación, puede aumentar o reducir manualmente el número de máquinas virtuales del conjunto de escalado con [Update-AzVmss](/powershell/module/az.compute/update-azvmss). En el ejemplo siguiente se establece el número de máquinas virtuales en el conjunto de escalado en *3*:

```azurepowershell-interactive
# Get current scale set
$scaleset = Get-AzVmss `
  -ResourceGroupName "myResourceGroupScaleSet" `
  -VMScaleSetName "myScaleSet"

# Set and update the capacity of your scale set
$scaleset.sku.capacity = 3
Update-AzVmss -ResourceGroupName "myResourceGroupScaleSet" `
    -Name "myScaleSet" `
    -VirtualMachineScaleSet $scaleset
```

Se tarda unos minutos en actualizar el número especificado de instancias en el conjunto de escalado.


### <a name="configure-autoscale-rules"></a>Configuración de reglas de escalado automático
En lugar de escalar manualmente el número de instancias en el conjunto de escalado, defina reglas de escalado automático. Estas reglas supervisan las instancias en el conjunto de escalado y responden según corresponda, basándose en las métricas y los umbrales que defina. En el ejemplo siguiente se escala horizontalmente el número de instancias en uno cuando la carga de CPU media es mayor que el 60 % durante un período de cinco minutos. Si la carga de CPU media se sitúa por debajo del 30 % durante un período de cinco minutos, las instancias se reducen horizontalmente en una instancia:

```azurepowershell-interactive
# Define your scale set information
$mySubscriptionId = (Get-AzSubscription)[0].Id
$myResourceGroup = "myResourceGroupScaleSet"
$myScaleSet = "myScaleSet"
$myLocation = "East US"
$myScaleSetId = (Get-AzVmss -ResourceGroupName $myResourceGroup -VMScaleSetName $myScaleSet).Id 

# Create a scale up rule to increase the number instances after 60% average CPU usage exceeded for a 5-minute period
$myRuleScaleUp = New-AzAutoscaleRule `
  -MetricName "Percentage CPU" `
  -MetricResourceId $myScaleSetId `
  -Operator GreaterThan `
  -MetricStatistic Average `
  -Threshold 60 `
  -TimeGrain 00:01:00 `
  -TimeWindow 00:05:00 `
  -ScaleActionCooldown 00:05:00 `
  -ScaleActionDirection Increase `
  -ScaleActionValue 1

# Create a scale down rule to decrease the number of instances after 30% average CPU usage over a 5-minute period
$myRuleScaleDown = New-AzAutoscaleRule `
  -MetricName "Percentage CPU" `
  -MetricResourceId $myScaleSetId `
  -Operator LessThan `
  -MetricStatistic Average `
  -Threshold 30 `
  -TimeGrain 00:01:00 `
  -TimeWindow 00:05:00 `
  -ScaleActionCooldown 00:05:00 `
  -ScaleActionDirection Decrease `
  -ScaleActionValue 1

# Create a scale profile with your scale up and scale down rules
$myScaleProfile = New-AzAutoscaleProfile `
  -DefaultCapacity 2  `
  -MaximumCapacity 10 `
  -MinimumCapacity 2 `
  -Rule $myRuleScaleUp,$myRuleScaleDown `
  -Name "autoprofile"

# Apply the autoscale rules
Add-AzAutoscaleSetting `
  -Location $myLocation `
  -Name "autosetting" `
  -ResourceGroup $myResourceGroup `
  -TargetResourceId $myScaleSetId `
  -AutoscaleProfile $myScaleProfile
```

Para más información de diseño sobre el uso del escalado automático, consulte los [procedimientos recomendados de escalado automático](/azure/architecture/best-practices/auto-scaling).


## <a name="next-steps"></a>Pasos siguientes
En este tutorial, ha creado un conjunto de escalado de máquinas virtuales. Ha aprendido a:

> [!div class="checklist"]
> * Usar la extensión de script personalizada para definir un sitio IIS para escalar
> * Crear un equilibrador de carga para el conjunto de escalado
> * Crear un conjunto de escalado de máquinas virtuales
> * Aumentar o disminuir el número de instancias en un conjunto de escalado
> * Crear reglas de escalado automático

Pase al siguiente tutorial para obtener más información sobre el concepto de equilibrio de carga de las máquinas virtuales.

> [!div class="nextstepaction"]
> [Load balance virtual machines](tutorial-load-balancer.md) (Equilibrio de carga de máquinas virtuales)
