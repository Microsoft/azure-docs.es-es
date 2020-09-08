---
title: 'Tutorial: Escalado automático de un conjunto de escalado con Azure PowerShell'
description: Aprenda a escalar automáticamente un conjunto de escalado de máquinas virtuales con Azure PowerShell a medida que la demanda de la CPU aumenta y disminuye
author: ju-shim
ms.author: jushiman
ms.topic: tutorial
ms.service: virtual-machine-scale-sets
ms.subservice: autoscale
ms.date: 03/27/2018
ms.reviewer: avverma
ms.custom: avverma, devx-track-azurepowershell
ms.openlocfilehash: 8ee124f866a5241620671ff84c24f3713f62efe1
ms.sourcegitcommit: 656c0c38cf550327a9ee10cc936029378bc7b5a2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89078476"
---
# <a name="tutorial-automatically-scale-a-virtual-machine-scale-set-with-azure-powershell"></a>Tutorial: Creación de reglas de escalado automático de conjuntos de escalado de máquinas virtuales con Azure PowerShell

[!INCLUDE [requires-azurerm](../../includes/requires-azurerm.md)]

Al crear un conjunto de escalado, puede definir el número de instancias de máquina virtual que quiere ejecutar. A medida que cambia la demanda de las aplicaciones, puede aumentar o reducir automáticamente el número de estas instancias. La posibilidad de realizar el escalado automático le permite satisfacer la demanda del cliente o responder a los cambios de rendimiento de la aplicación a lo largo del ciclo de vida de esta. En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Usar el escalado automático con un conjunto de escalado
> * Crear y usar reglas de escalado automático
> * Realizar pruebas de esfuerzo de instancias de máquina virtual y desencadenar reglas de escalado automático
> * Reducir el escalado automáticamente cuando se reduzca la demanda

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

Existe un problema conocido que afecta a la versión 6.8.1 o posterior del módulo de Azure PowerShell, incluida la versión actual de Azure Cloud Shell. Este tutorial solo se puede realizar con la versión 6.0.0 a 6.8.0 del módulo de Azure PowerShell. Ejecute `Get-Module -ListAvailable AzureRM` para encontrar la versión. Si PowerShell se ejecuta localmente, también debe ejecutar `Connect-AzureRmAccount` para crear una conexión con Azure.


## <a name="create-a-scale-set"></a>Creación de un conjunto de escalado
Para facilitar la creación de las reglas de escalado automático, defina algunas de las variables para el conjunto de escalado. En el siguiente ejemplo se definen variables para el conjunto de escalado denominado *myScaleSet* del grupo de recursos *myResourceGroup* en la región *Este de EE. UU.* El identificador de la suscripción se obtiene con [Get-AzureRmSubscription](/powershell/module/azurerm.profile/get-azurermsubscription). Si tiene varias suscripciones asociadas a su cuenta, solo se devolverá la primera de ellas. Ajuste los nombres y los identificadores de la suscripción como se indica a continuación:

```azurepowershell-interactive
$mySubscriptionId = (Get-AzureRmSubscription)[0].Id
$myResourceGroup = "myResourceGroup"
$myScaleSet = "myScaleSet"
$myLocation = "East US"
```

Ahora, cree un conjunto de escalado de máquinas virtuales con [New-AzureRmVmss](/powershell/module/azurerm.compute/new-azurermvmss). Para distribuir el tráfico a las instancias individuales de VM, también se crea un equilibrador de carga. El equilibrador de carga incluye reglas para distribuir el tráfico en el puerto TCP 80, y permitir el tráfico de Escritorio remoto en el puerto TCP 3389 y la conexión remota de PowerShell en el puerto TCP 5985. Cuando se le solicite, proporcione sus propias credenciales administrativas para las instancias de máquina virtual en el conjunto de escalado:

```azurepowershell-interactive
New-AzureRmVmss `
  -ResourceGroupName $myResourceGroup `
  -VMScaleSetName $myScaleSet `
  -Location $myLocation `
  -VirtualNetworkName "myVnet" `
  -SubnetName "mySubnet" `
  -PublicIpAddressName "myPublicIPAddress" `
  -LoadBalancerName "myLoadBalancer"
```

Se tardan unos minutos en crear y configurar todos los recursos de conjunto de escalado y máquinas virtuales.

## <a name="create-a-rule-to-autoscale-out"></a>Creación de una regla de escalado automático horizontal
Si aumenta la demanda de la aplicación, la carga de las instancias de máquina virtual del conjunto de escalado aumenta. Si este aumento de la carga es continuado, en lugar de ser algo puntual, puede configurar reglas de escalado automático para aumentar el número de instancias de máquina virtual en el conjunto de escalado. Cuando se crean estas instancias de máquina virtual y se implementan las aplicaciones, el conjunto de escalado empieza a distribuir el tráfico entre ellas mediante el equilibrador de carga. Puede controlar qué métricas se deben supervisar como, por ejemplo, la CPU o el disco, cuánto tiempo debe cumplir la carga de la aplicación un límite determinado y cuántas instancias de máquina virtual se deben agregar al conjunto de escalado.

Vamos a crear una regla con [New-AzureRmAutoscaleRule](/powershell/module/AzureRM.Insights/New-AzureRmAutoscaleRule) que aumenta el número de instancias de máquina virtual de un conjunto de escalado cuando la carga promedio de la CPU es superior al 70 % durante un período de más de 5 minutos. Cuando se desencadena la regla, aumenta el número de instancias de máquina virtual en tres.

Los siguientes parámetros se utilizan para esta regla:

| Parámetro               | Explicación                                                                                                         | Value          |
|-------------------------|---------------------------------------------------------------------------------------------------------------------|----------------|
| *-MetricName*           | La métrica de rendimiento del conjunto de escalado sobre el que realizar las acciones de supervisión y aplicación.                                                   | Porcentaje de CPU |
| *-TimeGrain*            | Frecuencia de recopilación de las métricas para el análisis.                                                                   | 1 minuto.       |
| *-MetricStatistic*      | Define cómo se deben agregar las métricas recopiladas para el análisis.                                                | Average        |
| *-TimeWindow*           | El periodo durante el que se realiza la supervisión antes de que se comparen los valores de métricas y umbrales.                                   | 5 minutos      |
| *-Operator*             | El operador que se utiliza para comparar los datos de las métricas con los umbrales.                                                     | Mayor que   |
| *-Threshold*            | El valor que hace que la regla de escalado automático desencadene una acción.                                                      | 70%            |
| *-ScaleActionDirection* | Define si el conjunto de escalado debe escalarse o reducirse verticalmente.                                             | Aumento       |
| *-ScaleActionScaleType* | Indica que el número de instancias de máquina virtual debe cambiarse por un valor especificado.                                    | Nº de cambios   |
| *-ScaleActionValue*     | El porcentaje de instancias de máquina virtual se debe cambiar al desencadenarse la regla.                                            | 3              |
| *-ScaleActionCooldown*  | El periodo que hay que esperar hasta que la regla se vuelva a aplicar, para que las acciones de escalado automático tengan tiempo de surtir efecto. | 5 minutos      |

En el ejemplo siguiente se crea un objeto denominado *myRuleScaleOut* que contiene esta regla de escalado vertical. *- MetricResourceId* usa las variables definidas anteriormente para el identificador de la suscripción, el nombre del grupo de recursos y el nombre del conjunto de escalado:

```azurepowershell-interactive
$myRuleScaleOut = New-AzureRmAutoscaleRule `
  -MetricName "Percentage CPU" `
  -MetricResourceId /subscriptions/$mySubscriptionId/resourceGroups/$myResourceGroup/providers/Microsoft.Compute/virtualMachineScaleSets/$myScaleSet `
  -TimeGrain 00:01:00 `
  -MetricStatistic "Average" `
  -TimeWindow 00:05:00 `
  -Operator "GreaterThan" `
  -Threshold 70 `
  -ScaleActionDirection "Increase" `
  -ScaleActionScaleType "ChangeCount" `
  -ScaleActionValue 3 `
  -ScaleActionCooldown 00:05:00
```


## <a name="create-a-rule-to-autoscale-in"></a>Creación de una regla de escalado automático horizontal de reducción
La demanda de la aplicación puede reducirse por las tardes o durante los fines de semana. Si esta reducción es constante a lo largo de un período, puede configurar reglas de escalado automático para reducir el número de instancias de máquina virtual del conjunto de escalado. Esta acción de reducción horizontal permite rebajar el costo de ejecutar el conjunto de escalado ya que solo se ejecuta el número de instancias necesario para satisfacer la demanda actual.

Cree otra regla con [New-AzureRmAutoscaleRule](/powershell/module/AzureRM.Insights/New-AzureRmAutoscaleRule) que reduzca el número de instancias de máquina virtual de un conjunto de escalado cuando la carga promedio de la CPU sea inferior al 30 % durante un período de más de 5 minutos. Cuando se desencadena la regla, el número de instancias de máquina virtual se reduce en una. En el ejemplo siguiente se crea un objeto denominado *myRuleScaleOut* que contiene esta regla de reducción vertical. *- MetricResourceId* usa las variables definidas anteriormente para el identificador de la suscripción, el nombre del grupo de recursos y el nombre del conjunto de escalado:

```azurepowershell-interactive
$myRuleScaleIn = New-AzureRmAutoscaleRule `
  -MetricName "Percentage CPU" `
  -MetricResourceId /subscriptions/$mySubscriptionId/resourceGroups/$myResourceGroup/providers/Microsoft.Compute/virtualMachineScaleSets/$myScaleSet `
  -Operator "LessThan" `
  -MetricStatistic "Average" `
  -Threshold 30 `
  -TimeGrain 00:01:00 `
  -TimeWindow 00:05:00 `
  -ScaleActionCooldown 00:05:00 `
  -ScaleActionDirection "Decrease" `
  -ScaleActionScaleType "ChangeCount" `
  -ScaleActionValue 1
```


## <a name="define-an-autoscale-profile"></a>Definición de un perfil de escalado automático
Cree un perfil para asociar las reglas de escalado automático a un conjunto de escalado. El perfil de escalado automático define la capacidad predeterminada, mínima y máxima del conjunto de escalado y asocia las reglas de escalado automático. Cree un perfil de escalado automático con [New-AzureRmAutoscaleProfile](/powershell/module/AzureRM.Insights/New-AzureRmAutoscaleProfile). En el ejemplo siguiente se establece la capacidad predeterminada y la mínima para *2* instancias de máquina virtual y un máximo de *10*. Las reglas de escalado y reducción horizontal creadas en los pasos anteriores se asocian ahora:

```azurepowershell-interactive
$myScaleProfile = New-AzureRmAutoscaleProfile `
  -DefaultCapacity 2  `
  -MaximumCapacity 10 `
  -MinimumCapacity 2 `
  -Rule $myRuleScaleOut,$myRuleScaleIn `
  -Name "autoprofile"
```


## <a name="apply-autoscale-profile-to-a-scale-set"></a>Aplicación del perfil de escalabilidad automática a un conjunto de escalado
El último paso consiste en aplicar el perfil de escalado automático al conjunto de escalado. Posteriormente, el conjunto de escalado se podrá escalar o reducir horizontalmente en función de la demanda de la aplicación. Aplique el perfil de escalado automático con [Add-AzureRmAutoscaleSetting](/powershell/module/AzureRM.Insights/Add-AzureRmAutoscaleSetting) como se indica a continuación:

```azurepowershell-interactive
Add-AzureRmAutoscaleSetting `
  -Location $myLocation `
  -Name "autosetting" `
  -ResourceGroup $myResourceGroup `
  -TargetResourceId /subscriptions/$mySubscriptionId/resourceGroups/$myResourceGroup/providers/Microsoft.Compute/virtualMachineScaleSets/$myScaleSet `
  -AutoscaleProfile $myScaleProfile
```


## <a name="generate-cpu-load-on-scale-set"></a>Generación de carga de CPU en el conjunto de escalado
Para probar las reglas de escalado automático, genere cargas de CPU en las instancias de máquina virtual del conjunto de escalado. Esta carga de CPU simulada hace que las reglas de escalado automático escalen horizontalmente y aumenten el número de instancias de máquina virtual. Como la carga simulada de la CPU se reduce, las reglas de escalado automático reducen horizontalmente el número de instancias de máquina virtual.

Para obtener una lista de los puertos NAT para conectarse a instancias de máquina virtual en un conjunto de escalado, primero obtenga el objeto de equilibrador de carga con el comando [Get-AzureRmLoadBalancer](/powershell/module/AzureRM.Network/Get-AzureRmLoadBalancer). Después, consulte las reglas NAT entrantes con [Get-AzureRmLoadBalancerInboundNatRuleConfig](/powershell/module/AzureRM.Network/Get-AzureRmLoadBalancerInboundNatRuleConfig):

```azurepowershell-interactive
# Get the load balancer object
$lb = Get-AzureRmLoadBalancer -ResourceGroupName "myResourceGroup" -Name "myLoadBalancer"

# View the list of inbound NAT rules
Get-AzureRmLoadBalancerInboundNatRuleConfig -LoadBalancer $lb | Select-Object Name,Protocol,FrontEndPort,BackEndPort
```

La salida del ejemplo siguiente muestra el nombre de instancia, la dirección IP pública del equilibrador de carga y el número de puerto al que las reglas NAT reenvían el tráfico:

```powershell
Name        Protocol FrontendPort BackendPort
----        -------- ------------ -----------
myRDPRule.0 Tcp             50001        3389
myRDPRule.1 Tcp             50002        3389
```

El *nombre* de la regla se alinea con el nombre de la instancia de máquina virtual como se muestra en un comando [Get-AzureRmVmssVM](/powershell/module/azurerm.compute/get-azurermvmssvm) anterior. Por ejemplo, para conectarse a la instancia de máquina virtual *0*, use *myRDPRule.0* y conéctese al puerto *50001*. Para conectarse a la instancia de máquina virtual *1*, utilice el valor de *myRDPRule.1* y conéctese al puerto *50002*.

Consulte la dirección IP pública del equilibrador de carga con [Get-AzureRmPublicIPAddress](/powershell/module/AzureRM.Network/Get-AzureRmPublicIpAddress):

```azurepowershell-interactive
Get-AzureRmPublicIpAddress -ResourceGroupName "myResourceGroup" -Name myPublicIP | Select IpAddress
```

Salida de ejemplo:

```powershell
IpAddress
---------
52.168.121.216
```

Cree una conexión remota a la primera instancia de máquina virtual. Especifique su dirección IP pública y el número de puerto de la instancia de máquina virtual necesaria, tal como se muestra en los comandos anteriores. Cuando se le solicite, escriba las credenciales que ha utilizado al crear el conjunto de escalado (de forma predeterminada en los comandos de ejemplo, *azureuser* y *P\@ssw0rd!* ). Si se utiliza Azure Cloud Shell, realice este paso desde un símbolo del sistema local de PowerShell o un cliente de Escritorio remoto. En el ejemplo siguiente se realiza una conexión a la instancia de máquina virtual *0*:

```powershell
mstsc /v 52.168.121.216:50001
```

Con la sesión iniciada, abra Internet Explorer desde la barra de tareas.

- Seleccione **Aceptar** para aceptar *Usar la configuración recomendada de compatibilidad, privacidad y seguridad* en el símbolo del sistema.
- Escriba *http://download.sysinternals.com/files/CPUSTRES.zip* en la barra de direcciones.
- Como la configuración de seguridad mejorada de Internet Explorer está habilitada, elija **Agregar** el dominio *http://download.sysinternals.com* a la lista de sitios de confianza.
- Cuando se le solicite la descarga del archivo, seleccione **Abrir**, y seleccione y haga clic en **Ejecutar** para ejecutar la herramienta *CPUSTRES.EXE*.

Para generar carga de la CPU, marque dos casillas de subprocesos **activos**. Desde el menú desplegable **Actividad** de ambos subprocesos, seleccione *Máximo*. Puede abrir el Administrador de tareas para confirmar que la carga de CPU en la máquina virtual está al 100 %.

![Generación de carga en la instancia de máquina virtual con la utilidad CPU stress](media/tutorial-autoscale-powershell/cpu-stress-load.PNG)

Deje abierta la sesión de conexión a Escritorio remoto y abra otra. Conéctese a la segunda instancia de máquina virtual con el número de puerto del cmdlet [Get-AzureRmLoadBalancerInboundNatRuleConfig](/powershell/module/AzureRM.Network/Get-AzureRmLoadBalancerInboundNatRuleConfig) anterior:

```powershell
mstsc /v 52.168.121.216:50002
```

Una vez iniciada la sesión en la segunda instancia de máquina virtual, repita los pasos anteriores para descargar y ejecutar *CPUSTRES.EXE*. De nuevo, inicie dos subprocesos **activos** y establezca la actividad en *Máximo*.

Para permitir que la herramienta **CPU Stress** continúe ejecutándose, deje abierta ambas sesiones de conexión a Escritorio remoto.


## <a name="monitor-the-active-autoscale-rules"></a>Supervisión de las reglas de escalado automático activas
Para supervisar el número de instancias de máquina virtual del conjunto de escalado, use **while**. Las reglas de escalado automático tardan 5 minutos en empezar el proceso de escalado horizontal en respuesta a la carga de CPU generada por **CPUStress** en cada instancia de máquina virtual:

```azurepowershell-interactive
while (1) {Get-AzureRmVmssVM `
    -ResourceGroupName $myResourceGroup `
    -VMScaleSetName $myScaleSet; sleep 10}
```

Cuando se alcanza el umbral de la CPU, las reglas de escalado automático aumentan el número de instancias de máquina virtual en el conjunto de escalado. La siguiente salida muestra tres máquinas virtuales que se crean durante el escalado horizontal automático:

```powershell
ResourceGroupName         Name Location          Sku Capacity InstanceID ProvisioningState
-----------------         ---- --------          --- -------- ---------- -----------------
MYRESOURCEGROUP   myScaleSet_2   eastus Standard_DS2                   2         Succeeded
MYRESOURCEGROUP   myScaleSet_3   eastus Standard_DS2                   3         Succeeded
MYRESOURCEGROUP   myScaleSet_4   eastus Standard_DS2                   4          Creating
MYRESOURCEGROUP   myScaleSet_5   eastus Standard_DS2                   5          Creating
MYRESOURCEGROUP   myScaleSet_6   eastus Standard_DS2                   6          Creating
```

Cierre la herramienta **CPU Stress** en todas las sesiones de conexión a Escritorio remoto de las instancias de máquina virtual. La carga de CPU media del conjunto de escalado vuelve al estado normal. Después de otros 5 minutos, las reglas de escalado automático reducen horizontalmente el número de instancias de máquina virtual. Con las acciones de reducción horizontal se eliminan instancias de máquina virtual en orden, empezando por los identificadores mayores. Cuando un conjunto de escalado usa conjuntos de disponibilidad o zonas de disponibilidad, las acciones de reducción horizontal se distribuyen por igual entre esas instancias de máquina virtual. En la salida de ejemplo siguiente se muestra la eliminación de una instancia de máquina virtual con la reducción horizontal del conjunto de escalado:

```powershell
MYRESOURCEGROUP   myScaleSet_6   eastus Standard_DS2                   6          Deleting
```

Salga de *while* con `Ctrl-c`. El conjunto de escalado continúa la reducción horizontal cada 5 minutos y quita una instancia de máquina virtual hasta que se alcanza el mínimo de dos instancias.


## <a name="clean-up-resources"></a>Limpieza de recursos
Para quitar el conjunto de escalado y los recursos adicionales, elimine el grupo de recursos y todos sus recursos con [Remove-AzureRmResourceGroup](/powershell/module/azurerm.resources/remove-azurermresourcegroup). El parámetro `-Force` confirma que desea eliminar los recursos sin pedir confirmación adicional. El parámetro `-AsJob` devuelve el control a la petición de confirmación sin esperar a que finalice la operación.

```azurepowershell-interactive
Remove-AzureRmResourceGroup -Name "myResourceGroup" -Force -AsJob
```


## <a name="next-steps"></a>Pasos siguientes
En este tutorial aprendió cómo escalar y reducir horizontalmente de forma automática con Azure PowerShell:

> [!div class="checklist"]
> * Usar el escalado automático con un conjunto de escalado
> * Crear y usar reglas de escalado automático
> * Realizar pruebas de esfuerzo de instancias de máquina virtual y desencadenar reglas de escalado automático
> * Reducir el escalado automáticamente cuando se reduzca la demanda
