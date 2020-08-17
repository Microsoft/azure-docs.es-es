---
title: Uso de Azure Policy para restringir la instalación de extensiones de VM (Windows)
description: Use Azure Policy para restringir las implementaciones de extensiones.
services: virtual-machines-linux
documentationcenter: ''
author: axayjo
manager: gwallace
editor: ''
ms.service: virtual-machines-linux
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure-services
ms.date: 03/23/2018
ms.author: akjosh
ms.reviewer: cynthn
ms.openlocfilehash: e4959c9dca909afde4bf6d351d79ecca1e4022a0
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87069753"
---
# <a name="use-azure-policy-to-restrict-extensions-installation-on-windows-vms"></a>Uso de Azure Policy para restringir la instalación de extensiones en máquinas virtuales Linux

Si desea impedir el uso o la instalación de determinadas extensiones en las máquinas virtuales Windows, puede crear una definición de Azure Policy mediante PowerShell para restringir las extensiones de las máquinas virtuales dentro de un grupo de recursos. 

En este tutorial se usa Azure PowerShell dentro de Cloud Shell, que se actualiza constantemente a la versión más reciente. 

 

## <a name="create-a-rules-file"></a>Creación de un archivo de reglas

Para restringir qué extensiones se pueden instalar, debe tener una [regla](../../governance/policy/concepts/definition-structure.md#policy-rule) que proporcione la lógica para identificar la extensión.

En este ejemplo se muestra cómo denegar las extensiones publicadas por "Microsoft.OSTCExtensions" mediante la creación de un archivo de reglas en Azure Cloud Shell. Sin embargo, si está trabajando en PowerShell localmente, también puede crear un archivo local y reemplazar la ruta de acceso ($home/clouddrive) por la del archivo local en su máquina.

En una instancia de [Cloud Shell](https://shell.azure.com/powershell), escriba:

```azurepowershell-interactive
nano $home/clouddrive/rules.json
```

Copie y pegue el siguiente archivo .json en el archivo.

```json
{
    "if": {
        "allOf": [
            {
                "field": "type",
                "equals": "Microsoft.Compute/virtualMachines/extensions"
            },
            {
                "field": "Microsoft.Compute/virtualMachines/extensions/publisher",
                "equals": "Microsoft.Compute"
            },
            {
                "field": "Microsoft.Compute/virtualMachines/extensions/type",
                "in": "[parameters('notAllowedExtensions')]"
            }
        ]
    },
    "then": {
        "effect": "deny"
    }
}
```

Cuando haya terminado, presione **Ctrl + O** y, a continuación, **Entrar** para guardar el archivo. Presione **Ctrl + X** para cerrar el archivo y salir.

## <a name="create-a-parameters-file"></a>Creación de un archivo de parámetros

También necesita un archivo de [parámetros](../../governance/policy/concepts/definition-structure.md#parameters) que cree una estructura que pueda usar para pasar una lista de las extensiones que se van a bloquear. 

En este ejemplo se muestra cómo crear un archivo de parámetros para máquinas virtuales en Cloud Shell. Sin embargo, si está trabajando en PowerShell localmente, también puede crear un archivo local y reemplazar la ruta de acceso ($home/clouddrive) por la del archivo local en su máquina.

En [Cloud Shell](https://shell.azure.com/powershell), escriba:

```azurepowershell-interactive
nano $home/clouddrive/parameters.json
```

Copie y pegue el siguiente archivo .json en el archivo.

```json
{
    "notAllowedExtensions": {
        "type": "Array",
        "metadata": {
            "description": "The list of extensions that will be denied.",
            "displayName": "Denied extension"
        }
    }
}
```

Cuando haya terminado, presione **Ctrl + O** y, a continuación, **Entrar** para guardar el archivo. Presione **Ctrl + X** para cerrar el archivo y salir.

## <a name="create-the-policy"></a>Creación de la directiva

Una definición de directiva es un objeto que se usa para almacenar la configuración que le gustaría utilizar. En esta definición se incluyen las reglas y los archivos de parámetros. Cree una definición de directiva mediante el cmdlet [New-AzPolicyDefinition](/powershell/module/az.resources/new-azpolicydefinition).

 Las reglas y los parámetros de la directiva son los archivos creados y almacenados como archivos .json en Cloud Shell.


```azurepowershell-interactive
$definition = New-AzPolicyDefinition `
   -Name "not-allowed-vmextension-windows" `
   -DisplayName "Not allowed VM Extensions" `
   -description "This policy governs which VM extensions that are explicitly denied."   `
   -Policy 'C:\Users\ContainerAdministrator\clouddrive\rules.json' `
   -Parameter 'C:\Users\ContainerAdministrator\clouddrive\parameters.json'
```




## <a name="assign-the-policy"></a>Asignación de la directiva

En este ejemplo se asigna la directiva a un grupo de recursos mediante [New-AzPolicyAssignment](/powershell/module/az.resources/new-azpolicyassignment). Ninguna de las máquinas virtuales creadas en el grupo de recursos **myResourceGroup** podrá instalar las extensiones de agente de acceso de máquina virtual o de script personalizado. 

Use el cmdlet [Get-AzSubscription | Format-Table](/powershell/module/az.accounts/get-azsubscription) para obtener el identificador de suscripción y usarlo para sustituir al del ejemplo.

```azurepowershell-interactive
$scope = "/subscriptions/<subscription id>/resourceGroups/myResourceGroup"
$assignment = New-AzPolicyAssignment `
   -Name "not-allowed-vmextension-windows" `
   -Scope $scope `
   -PolicyDefinition $definition `
   -PolicyParameter '{
    "notAllowedExtensions": {
        "value": [
            "VMAccessAgent",
            "CustomScriptExtension"
        ]
    }
}'
$assignment
```

## <a name="test-the-policy"></a>Prueba de la directiva

Para probar la directiva, intente usar la extensión de acceso de máquina virtual. Se producirá un error con el mensaje "Set-AzVMAccessExtension: Resource 'myVMAccess' was disallowed by policy" (Set-AzVMAccessExtension: la directiva no permitió el recurso "myVMAccess").

```azurepowershell-interactive
Set-AzVMAccessExtension `
   -ResourceGroupName "myResourceGroup" `
   -VMName "myVM" `
   -Name "myVMAccess" `
   -Location EastUS 
```

En el portal, el cambio de contraseña generará un error con el mensaje "The template deployment failed because of policy violation" (No se pudo implementar la plantilla a causa de la infracción de la directiva).

## <a name="remove-the-assignment"></a>Eliminación de la asignación

```azurepowershell-interactive
Remove-AzPolicyAssignment -Name not-allowed-vmextension-windows -Scope $scope
```

## <a name="remove-the-policy"></a>Eliminación de la directiva

```azurepowershell-interactive
Remove-AzPolicyDefinition -Name not-allowed-vmextension-windows
```
    
## <a name="next-steps"></a>Pasos siguientes
Para más información, consulte [Azure Policy](../../governance/policy/overview.md).
