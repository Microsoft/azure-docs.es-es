---
title: Recursos de variables en Azure Automation
description: Los activos de variables son valores que están disponibles para todos los runbooks y configuraciones de DSC en Azure Automation.  En este artículo se explican los detalles de las variables y cómo trabajar con ellas en la creación de texto y gráficos.
services: automation
ms.service: automation
ms.subservice: shared-capabilities
author: georgewallace
ms.author: gwallace
ms.date: 05/14/2019
ms.topic: conceptual
manager: carmonm
ms.openlocfilehash: 0ac34f1d1e7fc2a967c7608f31f3b943f9380d01
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65786206"
---
# <a name="variable-assets-in-azure-automation"></a>Recursos de variables en Azure Automation

Los activos de variables son valores que están disponibles para todos los runbooks y configuraciones de DSC en su cuenta de Automatización. Que pueden administrarse desde Azure portal, PowerShell, un runbook o configuración de DSC. Las variables de Automation son útiles para los siguientes escenarios:

- Compartir un valor entre varios runbooks o configuraciones de DSC.

- Compartir un valor entre varios trabajos del mismo runbook o configuración de DSC.

- Administrar un valor desde el portal o desde la línea de comandos de PowerShell que se usan los runbooks o configuraciones de DSC, como un conjunto de elementos de configuración comunes, como la lista específica de los nombres de máquina virtual, un grupo de recursos específico, un nombre de dominio de AD y mucho más.  

Dado que las variables de Automation se conservan, están disponibles incluso si se produce un error en el runbook o configuración de DSC. Este comportamiento permite que un valor que se establecerá un runbook que, a continuación, se utiliza por otro, o se usa el mismo runbook o configuración de DSC la próxima vez que se ejecute.

Cuando se crea una variable, puede especificar que se almacene cifrada. Las variables cifradas se almacenan de forma segura en Azure Automation y su valor no se puede recuperar desde el [Get-AzureRmAutomationVariable](/powershell/module/AzureRM.Automation/Get-AzureRmAutomationVariable) cmdlet que se incluye como parte del módulo Azure PowerShell. Es la única manera de que se puede recuperar un valor cifrado desde la actividad **Get-AutomationVariable** en un runbook o una configuración de DSC.

>[!NOTE]
>Los recursos protegidos en Azure Automation incluyen credenciales, certificados, conexiones y variables cifradas. Estos recursos se cifran y se almacenan en Azure Automation con una clave única que se genera para cada cuenta de Automation. Esta clave se almacena en una instancia de Key Vault administrada por el sistema. Antes de almacenar un recurso seguro, la clave se carga desde Key Vault y luego se usa para cifrar el recurso. Este proceso se administra mediante Azure Automation.

## <a name="variable-types"></a>Tipos de variables

Cuando se crea una variable con el portal de Azure, debe especificar un tipo de datos en la lista desplegable para que el portal pueda mostrar el control adecuado para escribir el valor de la variable. La variable no está restringida a este tipo de datos. Debe establecer la variable mediante Windows PowerShell si desea especificar un valor de un tipo diferente. Si especifica **no definido**, a continuación, Establece el valor de la variable **$null**, y debe establecer el valor con el [Set-AzureRMAutomationVariable](/powershell/module/AzureRM.Automation/Set-AzureRmAutomationVariable) cmdlet o **Set-AutomationVariable** actividad. No se puede crear o cambiar el valor de un tipo complejo de variable en el portal, pero puede proporcionar un valor de cualquier tipo mediante Windows PowerShell. Los tipos complejos se devolverán como un [PSCustomObject](/dotnet/api/system.management.automation.pscustomobject).

Puede almacenar varios valores en una única variable mediante la creación de una matriz o tabla hash y guardarlos en la variable.

La siguiente es una lista de los tipos de variables disponibles en Automation:

* String
* Integer
* DateTime
* Boolean
* Null

## <a name="azurerm-powershell-cmdlets"></a>Cmdlets de AzureRM PowerShell

En AzureRM, los cmdlets de la tabla siguiente se usan para crear y administrar recursos de credenciales de automatización con Windows PowerShell. Se incluyen como parte de la [módulo Azurerm](/powershell/azure/overview), que está disponible para su uso en los runbooks de Automation y las configuraciones de DSC.

| Cmdlets | DESCRIPCIÓN |
|:---|:---|
|[Get-AzureRmAutomationVariable](/powershell/module/AzureRM.Automation/Get-AzureRmAutomationVariable)|Recupera el valor de una variable existente.|
|[New-AzureRmAutomationVariable](/powershell/module/AzureRM.Automation/New-AzureRmAutomationVariable)|Crea una nueva variable y establece su valor.|
|[Remove-AzureRmAutomationVariable](/powershell/module/AzureRM.Automation/Remove-AzureRmAutomationVariable)|Quita una variable existente.|
|[Set-AzureRmAutomationVariable](/powershell/module/AzureRM.Automation/Set-AzureRmAutomationVariable)|Establece el valor de una variable existente.|

## <a name="activities"></a>Actividades

Las actividades de la tabla siguiente se usan para tener acceso a las credenciales de un runbook y configuraciones de DSC.

| Actividades | DESCRIPCIÓN |
|:---|:---|
|Get-AutomationVariable|Recupera el valor de una variable existente.|
|Set-AutomationVariable|Establece el valor de una variable existente.|

> [!NOTE]
> Debe evitar el uso de variables en el parámetro – Name de **Get-AutomationVariable** en un runbook o una configuración de DSC, ya que puede complicar la detección de dependencias entre runbooks o configuraciones de DSC y las variables de Automation en tiempo de diseño.

Las funciones de la tabla siguiente se usan para obtener acceso a las variables de un runbook de Python2 y recuperarlas.

|Funciones de Python2|DESCRIPCIÓN|
|:---|:---|
|automationassets.get_automation_variable|Recupera el valor de una variable existente. |
|automationassets.set_automation_variable|Establece el valor de una variable existente. |

> [!NOTE]
> Debe importar el módulo "automationassets" en la parte superior del runbook de Python para poder tener acceso a las funciones del recurso.

## <a name="creating-a-new-automation-variable"></a>Creación de una nueva variable de Automation

### <a name="to-create-a-new-variable-with-the-azure-portal"></a>Para crear una nueva variable con el Portal de Azure

1. En la cuenta de Automation, haga clic en el icono **Activos** y, en la hoja **Activos**, seleccione **Variables**.
2. En la hoja **Variables**, seleccione **Agregar una variable**.
3. Complete las opciones en la hoja **Nueva variable** y haga clic en **Crear** para guardar la nueva variable.

### <a name="to-create-a-new-variable-with-windows-powershell"></a>Para crear una nueva variable con Windows PowerShell

El cmdlet [New-AzureRmAutomationVariable](/powershell/module/AzureRM.Automation/New-AzureRmAutomationVariable) crea una nueva variable y establece su valor inicial. Puede recuperar el valor mediante [Get-AzureRmAutomationVariable](/powershell/module/AzureRM.Automation/Get-AzureRmAutomationVariable). Si el valor es un tipo simple, se devuelve ese mismo tipo. Si es un tipo complejo, se devuelve **PSCustomObject** .

Los comandos de ejemplo siguientes muestran cómo crear una variable de tipo cadena y que después devuelva su valor.

```powershell
New-AzureRmAutomationVariable -ResourceGroupName "ResourceGroup01" 
–AutomationAccountName "MyAutomationAccount" –Name 'MyStringVariable' `
–Encrypted $false –Value 'My String'
$string = (Get-AzureRmAutomationVariable -ResourceGroupName "ResourceGroup01" `
–AutomationAccountName "MyAutomationAccount" –Name 'MyStringVariable').Value
```

Los comandos de ejemplo siguientes muestran cómo crear una variable con un tipo complejo y que después devuelva su valor. En este caso, se usa un objeto de máquina virtual desde **Get-AzureRmVm**.

```powershell
$vm = Get-AzureRmVm -ResourceGroupName "ResourceGroup01" –Name "VM01"
New-AzureRmAutomationVariable –AutomationAccountName "MyAutomationAccount" –Name "MyComplexVariable" –Encrypted $false –Value $vm

$vmValue = (Get-AzureRmAutomationVariable -ResourceGroupName "ResourceGroup01" `
–AutomationAccountName "MyAutomationAccount" –Name "MyComplexVariable").Value
$vmName = $vmValue.Name
$vmIpAddress = $vmValue.IpAddress
```

## <a name="using-a-variable-in-a-runbook-or-dsc-configuration"></a>Uso de una variable en un runbook o una configuración de DSC

Use la actividad **Set-AutomationVariable** para establecer el valor de una variable de Automation en un runbook de PowerShell o una configuración de DSC, y la actividad **Get-AutomationVariable** para recuperarlo. No debe usar los cmdlets **Set-AzureRMAutomationVariable** o **Get-AzureRMAutomationVariable** en un runbook o una configuración de DSC, ya que son menos eficientes que las actividades de flujo de trabajo. Tampoco puede recuperar el valor de variables seguras con **Get-AzureRMAutomationVariable**. La única forma de crear una nueva variable desde un runbook o una configuración de DSC es usar el cmdlet [New-AzureRMAutomationVariable](/powershell/module/AzureRM.Automation/New-AzureRmAutomationVariable).

### <a name="textual-runbook-samples"></a>Ejemplos de runbook textual

#### <a name="setting-and-retrieving-a-simple-value-from-a-variable"></a>Establecimiento y recuperación de un valor simple de una variable

Los comandos de ejemplo siguientes muestran cómo establecer y recuperar una variable en un runbook textual. En este ejemplo, se supone que las variables de tipo entero denominan *NumberOfIterations* y *NumberOfRunnings* y una variable de tipo string llamada *SampleMessage* tiene se ha creado.

```powershell
$NumberOfIterations = Get-AzureRmAutomationVariable -ResourceGroupName "ResourceGroup01" –AutomationAccountName "MyAutomationAccount" -Name 'NumberOfIterations'
$NumberOfRunnings = Get-AzureRmAutomationVariable -ResourceGroupName "ResourceGroup01" –AutomationAccountName "MyAutomationAccount" -Name 'NumberOfRunnings'
$SampleMessage = Get-AutomationVariable -Name 'SampleMessage'

Write-Output "Runbook has been run $NumberOfRunnings times."

for ($i = 1; $i -le $NumberOfIterations; $i++) {
    Write-Output "$i`: $SampleMessage"
}
Set-AzureRmAutomationVariable -ResourceGroupName "ResourceGroup01" –AutomationAccountName "MyAutomationAccount" –Name NumberOfRunnings –Value ($NumberOfRunnings += 1)
```

#### <a name="setting-and-retrieving-a-variable-in-python2"></a>Establecimiento y recuperación de una variable en Python2

En el siguiente código de ejemplo se muestra cómo usar una variable, establecer una variable y controlar una excepción de una variable inexistente en un runbook de Python2.

```python
import automationassets
from automationassets import AutomationAssetNotFound

# get a variable
value = automationassets.get_automation_variable("test-variable")
print value

# set a variable (value can be int/bool/string)
automationassets.set_automation_variable("test-variable", True)
automationassets.set_automation_variable("test-variable", 4)
automationassets.set_automation_variable("test-variable", "test-string")

# handle a non-existent variable exception
try:
    value = automationassets.get_automation_variable("non-existing variable")
except AutomationAssetNotFound:
    print "variable not found"
```

### <a name="graphical-runbook-samples"></a>Ejemplos de runbook gráfico

En un runbook gráfico, agregue **Get-AutomationVariable** o **Set-AutomationVariable** haciendo clic con el botón derecho en la variable en el panel Biblioteca del editor gráfico y seleccionando la actividad que desee.

![Adición de una variable al lienzo](../media/variables/runbook-variable-add-canvas.png)

#### <a name="setting-values-in-a-variable"></a>Configuración de valores de una variable

La imagen siguiente muestra las actividades de ejemplo para actualizar una variable con un valor simple en un runbook gráfico. En este ejemplo, **Get-AzureRmVM** recupera una sola máquina virtual de Azure y el nombre del equipo se guarda en una variable de Automation existente con un tipo de cadena. No importa si el [vínculo es una canalización o una secuencia](../automation-graphical-authoring-intro.md#links-and-workflow), puesto que solo esperamos un único objeto en la salida.

![Establecimiento de una variable simple](../media/variables/runbook-set-simple-variable.png)

## <a name="next-steps"></a>Pasos siguientes

- Para obtener más información sobre cómo conectar actividades en la creación gráfica, consulte [Vínculos y flujo de trabajo en Creación gráfica en Automatización de Azure](../automation-graphical-authoring-intro.md#links-and-workflow)
- Para empezar a trabajar con cuadernos gráficos, consulte [Mi primer runbook gráfico](../automation-first-runbook-graphical.md)
