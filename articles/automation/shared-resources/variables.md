---
title: Administración de variables en Azure Automation
description: En este artículo se explica cómo trabajar con variables en runbooks y configuraciones DSC.
services: automation
ms.service: automation
ms.subservice: shared-capabilities
author: mgoedtel
ms.author: magoedte
ms.date: 05/14/2019
ms.topic: conceptual
manager: carmonm
ms.openlocfilehash: ee49ae905622b4b76d782f6a31e0c2333b6d54be
ms.sourcegitcommit: 269da970ef8d6fab1e0a5c1a781e4e550ffd2c55
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/10/2020
ms.locfileid: "88055299"
---
# <a name="manage-variables-in-azure-automation"></a>Administración de variables en Azure Automation

Los recursos de variables son valores que están disponibles para todos los runbooks y configuraciones de DSC en su cuenta de Automation. Puede administrarlos desde Azure Portal, PowerShell, un runbook o en una configuración de DSC.

Las variables de Automation son útiles para los siguientes escenarios:

- Uso compartido de un valor entre varios runbooks o configuraciones de DSC.

- Uso compartido de un valor entre varios trabajos del mismo runbook o configuración de DSC.

- Administración de un valor que usan los runbooks o las configuraciones de DSC desde el portal o la línea de comandos de PowerShell. Un ejemplo es un conjunto de elementos de configuración comunes, como una lista de nombres de máquinas virtuales específica, un grupo de recursos específico, un nombre de dominio de AD y más.  

Azure Automation conserva las variables y hace que estén disponibles aunque se produzca un error en un runbook o en la configuración de DSC. Este comportamiento permite que un runbook o configuración de DSC establezca un valor para que otro runbook lo use, o bien que lo use el mismo runbook o la misma configuración de DSC la siguiente vez que se ejecute.

Azure Automation almacena cada variable cifrada de forma segura. Al crear una variable, puede especificar su cifrado y almacenamiento por Azure Automation como recurso seguro. Después de crear la variable, no se puede cambiar su estado de cifrado sin volver a crearla. Una recomendación de Azure Security Center es cifrar todas las variables de Azure Automation como se describe en [Las variables de la cuenta de Automation deben estar cifradas](../../security-center/recommendations-reference.md#recs-computeapp). 

>[!NOTE]
>Los recursos protegidos en Azure Automation incluyen credenciales, certificados, conexiones y variables cifradas. Estos recursos se cifran y se almacenan en Azure Automation con una clave única que se genera para cada cuenta de Automation. Azure Automation almacena la clave en la instancia de Key Vault administrada por el sistema. Antes de almacenar un recurso seguro, Azure Automation carga la clave desde Key Vault y después la usa para cifrar el recurso. 

## <a name="variable-types"></a>Tipos de variables

Cuando se crea una variable con Azure Portal, debe especificar un tipo de datos en la lista desplegable para que el portal pueda mostrar el control adecuado para escribir el valor de la variable. Los siguientes tipos de variable están disponibles en Azure Automation:

* String
* Entero
* DateTime
* Boolean
* Null

La variable no se limita al tipo de datos especificado. Debe establecer la variable mediante Windows PowerShell si se desea especificar un valor de un tipo diferente. Si indica `Not defined`, el valor de la variable se establece en NULL. Debe establecer el valor con el cmdlet [Set-AzAutomationVariable](/powershell/module/az.automation/set-azautomationvariable?view=azps-3.5.0) o el cmdlet interno `Set-AutomationVariable`.

Azure Portal no se puede usar para crear o cambiar el valor de un tipo de variable complejo. Sin embargo, puede proporcionar un valor de cualquier tipo mediante Windows PowerShell. Los tipos complejos se recuperan como un objeto [PSCustomObject](/dotnet/api/system.management.automation.pscustomobject).

Puede almacenar varios valores en una única variable mediante la creación de una matriz o tabla hash y guardarlos en la variable.

>[!NOTE]
>Las variables de nombre de máquina virtual pueden tener un máximo de 80 caracteres. Las variables de grupo de recursos pueden tener un máximo de 90 caracteres. Consulte [Reglas y restricciones de nomenclatura para los recursos de Azure](../../azure-resource-manager/management/resource-name-rules.md).

## <a name="powershell-cmdlets-to-access-variables"></a>Cmdlets de PowerShell para acceder a las variables

Los cmdlets de la tabla siguiente permiten crear y administrar variables de Automation con PowerShell. Se suministran como un componente de los [módulos Az](modules.md#az-modules).

| Cmdlet | Descripción |
|:---|:---|
|[Get-AzAutomationVariable](/powershell/module/az.automation/get-azautomationvariable?view=azps-3.5.0) | Recupera el valor de una variable existente. Si el valor es de un tipo simple, se recupera ese mismo tipo. Si es un tipo complejo, se recupera un tipo `PSCustomObject`. <br>**Nota:**  Este cmdlet no se puede usar para recuperar el valor de una variable cifrada. La única forma de hacerlo es usando el cmdlet `Get-AutomationVariable` interno en un runbook o una configuración de DSC. Consulte [Cmdlets internos para acceder a las variables](#internal-cmdlets-to-access-variables). |
|[New-AzAutomationVariable](/powershell/module/az.automation/new-azautomationvariable?view=azps-3.5.0) | Crea una nueva variable y establece su valor.|
|[Remove-AzAutomationVariable](/powershell/module/az.automation/remove-azautomationvariable?view=azps-3.5.0)| Quita una variable existente.|
|[Set-AzAutomationVariable](/powershell/module/az.automation/set-azautomationvariable?view=azps-3.5.0)| Establece el valor de una variable existente. |

## <a name="internal-cmdlets-to-access-variables"></a>Cmdlets internos para acceder a las variables

Los cmdlets internos de la tabla siguiente se usan para acceder a las variables en los runbooks y las configuraciones de DSC. Estos cmdlets se presentan con el módulo global `Orchestrator.AssetManagement.Cmdlets`. Para más información, consulte [Cmdlets internos](modules.md#internal-cmdlets).

| Cmdlet interno | Descripción |
|:---|:---|
|`Get-AutomationVariable`|Recupera el valor de una variable existente.|
|`Set-AutomationVariable`|Establece el valor de una variable existente.|

> [!NOTE]
> Evite usar variables en el parámetro `Name` de `Get-AutomationVariable` en un runbook o configuración de DSC. El uso de las variables puede complicar la detección de dependencias entre runbooks y variables de Automation durante el tiempo de diseño.

`Get-AutomationVariable` no funciona en PowerShell, solo en un runbook o una configuración de DSC. Por ejemplo, para ver el valor de una variable cifrada, puede crear un runbook para obtener la variable y escribirla en el flujo de salida:
 
```powershell
$mytestencryptvar = Get-AutomationVariable -Name TestVariable
Write-output "The encrypted value of the variable is: $mytestencryptvar"
```

## <a name="python-2-functions-to-access-variables"></a>Funciones de Python 2 para acceder a las variables

Las funciones de la siguiente tabla sirven para acceder a las variables de un runbook de Python 2.

|Funciones de Python 2|Descripción|
|:---|:---|
|`automationassets.get_automation_variable`|Recupera el valor de una variable existente. |
|`automationassets.set_automation_variable`|Establece el valor de una variable existente. |

> [!NOTE]
> Para acceder a las funciones del activo, debe importar el módulo `automationassets` en la parte superior del runbook de Python.

## <a name="create-and-get-a-variable"></a>Creación y obtención de una variable

>[!NOTE]
>Si va a quitar el cifrado de una variable, debe eliminarla y volver a crearla sin cifrado.

### <a name="create-and-get-a-variable-using-the-azure-portal"></a>Creación y obtención de una variable con Azure Portal

1. En la cuenta de Automation, haga clic en el mosaico **Activos** y en la hoja **Activos** y, luego, seleccione **Variables**.
2. En la hoja **Variables**, seleccione **Agregar una variable**.
3. Complete las opciones en la hoja **Nueva variable** y después haga clic en **Crear** para guardar la nueva variable.

> [!NOTE]
> Una vez que una variable cifrada se guarde, no podrá verse en el portal. Solo se podrá actualizar.

### <a name="create-and-get-a-variable-in-windows-powershell"></a>Creación y obtención de una variable en Windows PowerShell

El runbook o la configuración de DSC usa el cmdlet `New-AzAutomationVariable` para crear una nueva variable y establecer su valor inicial. Si la variable está cifrada, la llamada debe usar el parámetro `Encrypted`. El script puede recuperar el valor de la variable mediante `Get-AzAutomationVariable`. 

>[!NOTE]
>Un script de PowerShell no puede recuperar un valor cifrado. La única forma de hacerlo es usar el cmdlet `Get-AutomationVariable` interno.

En el siguiente ejemplo se muestra cómo crear una variable de tipo cadena y después se devuelve su valor.

```powershell
New-AzAutomationVariable -ResourceGroupName "ResourceGroup01" 
–AutomationAccountName "MyAutomationAccount" –Name 'MyStringVariable' `
–Encrypted $false –Value 'My String'
$string = (Get-AzAutomationVariable -ResourceGroupName "ResourceGroup01" `
–AutomationAccountName "MyAutomationAccount" –Name 'MyStringVariable').Value
```

En el siguiente ejemplo se muestra cómo crear una variable con un tipo complejo y después se recuperan sus propiedades. En este caso, se usa un objeto de máquina virtual de [Get-AzVM](/powershell/module/Az.Compute/Get-AzVM?view=azps-3.5.0).

```powershell
$vm = Get-AzVM -ResourceGroupName "ResourceGroup01" –Name "VM01"
New-AzAutomationVariable –AutomationAccountName "MyAutomationAccount" –Name "MyComplexVariable" –Encrypted $false –Value $vm

$vmValue = (Get-AzAutomationVariable -ResourceGroupName "ResourceGroup01" `
–AutomationAccountName "MyAutomationAccount" –Name "MyComplexVariable").Value
$vmName = $vmValue.Name
$vmIpAddress = $vmValue.IpAddress
```

## <a name="textual-runbook-examples"></a>Ejemplo de un runbook textual

### <a name="retrieve-and-set-a-simple-value-from-a-variable"></a>Recuperación y establecimiento de un valor simple de una variable

Los comandos de ejemplo siguientes muestran cómo establecer y recuperar una variable en un runbook textual. En este ejemplo se da por hecho que se han creado las variables de entero `NumberOfIterations` y `NumberOfRunnings`, así como una variable de cadena denominada `SampleMessage`.

```powershell
$NumberOfIterations = Get-AzAutomationVariable -ResourceGroupName "ResourceGroup01" –AutomationAccountName "MyAutomationAccount" -Name 'NumberOfIterations'
$NumberOfRunnings = Get-AzAutomationVariable -ResourceGroupName "ResourceGroup01" –AutomationAccountName "MyAutomationAccount" -Name 'NumberOfRunnings'
$SampleMessage = Get-AutomationVariable -Name 'SampleMessage'

Write-Output "Runbook has been run $NumberOfRunnings times."

for ($i = 1; $i -le $NumberOfIterations; $i++) {
    Write-Output "$i`: $SampleMessage"
}
Set-AzAutomationVariable -ResourceGroupName "ResourceGroup01" –AutomationAccountName "MyAutomationAccount" –Name NumberOfRunnings –Value ($NumberOfRunnings += 1)
```

### <a name="retrieve-and-set-a-variable-in-a-python-2-runbook"></a>Recuperación y establecimiento de una variable en un runbook de Python 2

En el siguiente ejemplo se muestra cómo obtener una variable, establecerla y administrar una excepción de una variable inexistente en un runbook de Python 2.

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
    value = automationassets.get_automation_variable("nonexisting variable")
except AutomationAssetNotFound:
    print "variable not found"
```

## <a name="graphical-runbook-examples"></a>Ejemplos de runbook gráfico

En un runbook gráfico, puede agregar actividades para los cmdlets internos `Get-AutomationVariable` o `Set-AutomationVariable`. Basta con hacer clic con el botón derecho en cada variable en el panel Biblioteca del editor gráfico y seleccionar la actividad que se va a agregar.

![Adición de una variable al lienzo](../media/variables/runbook-variable-add-canvas.png)

La imagen siguiente muestra las actividades de ejemplo para actualizar una variable con un valor simple en un runbook gráfico. En este ejemplo, la actividad para `Get-AzVM` recupera una única máquina virtual de Azure y guarda el nombre del equipo en una variable de cadena de Automation existente. No importa si el [vínculo es una canalización o una secuencia](../automation-graphical-authoring-intro.md#use-links-for-workflow), ya que el código solo espera un único objeto en la salida.

![Establecimiento de una variable simple](../media/variables/runbook-set-simple-variable.png)

## <a name="next-steps"></a>Pasos siguientes

* Para más información sobre los cmdlets que se usan para acceder a las variables, consulte [Administración de módulos en Azure Automation](modules.md).
* Para información general sobre los runbooks, consulte [Ejecución de un runbook en Azure Automation](../automation-runbook-execution.md).
* Para obtener más información sobre las configuraciones DSC, consulte [Introducción a State Configuration](../automation-dsc-overview.md).
