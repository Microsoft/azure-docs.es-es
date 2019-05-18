---
title: Automatizar la adición de un usuario de laboratorio en Azure DevTest Labs | Microsoft Docs
description: Obtenga información sobre cómo automatizar la adición de un usuario de laboratorio a un laboratorio en Azure DevTest Labs.
services: devtest-lab,lab-services
documentationcenter: na
author: spelluru
manager: femila
ms.service: lab-services
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 05/02/2019
ms.author: spelluru
ms.openlocfilehash: 2ad81ae97414abbf3266cc5728febf9abe836151
ms.sourcegitcommit: bb85a238f7dbe1ef2b1acf1b6d368d2abdc89f10
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/10/2019
ms.locfileid: "65522963"
---
# <a name="automate-adding-a-lab-user-to-a-lab-in-azure-devtest-labs"></a>Automatizar la adición de un usuario de laboratorio a un laboratorio en Azure DevTest Labs
Azure DevTest Labs le permite crear rápidamente entornos de desarrollo y pruebas de autoservicio mediante el portal de Azure. Sin embargo, si tiene varios equipos y varias instancias de DevTest Labs, automatizar el proceso de creación puede ahorrar tiempo. [Las plantillas de Azure Resource Manager](https://github.com/Azure/azure-devtestlab/tree/master/ARMTemplates) le permiten crear laboratorios, máquinas virtuales de laboratorio, imágenes personalizadas, fórmulas y agregar usuarios de forma automática. En este artículo se centra específicamente en Agregar usuarios a una instancia de DevTest Labs.

Para agregar un usuario a un laboratorio, agrega el usuario a la **usuario de DevTest Labs** rol para el laboratorio. Este artículo muestra cómo automatizar la adición de un usuario a un laboratorio mediante una de las maneras siguientes:

- Plantillas del Administrador de recursos de Azure
- Cmdlets de Azure PowerShell 
- CLI de Azure.

## <a name="use-azure-resource-manager-templates"></a>Utilización de plantillas del Administrador de recursos de Azure
La siguiente plantilla de Resource Manager de ejemplo especifica un usuario que se agregarán a la **usuario de DevTest Labs** rol de un laboratorio. 

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "principalId": {
      "type": "string",
      "metadata": {
        "description": "The objectId of the user, group, or service principal for the role."
      }
    },
    "labName": {
      "type": "string",
      "metadata": {
        "description": "The name of the lab instance to be created."
      }
    },
    "roleAssignmentGuid": {
      "type": "string",
      "metadata": {
        "description": "Guid to use as the name for the role assignment."
      }
    }
  },
  "variables": {
    "devTestLabUserRoleId": "[concat('/subscriptions/', subscription().subscriptionId, '/providers/Microsoft.Authorization/roleDefinitions/111111111-0000-0000-11111111111111111')]",
    "fullDevTestLabUserRoleName": "[concat(parameters('labName'), '/Microsoft.Authorization/', parameters('roleAssignmentGuid'))]",
    "roleScope": "[concat('/subscriptions/', subscription().subscriptionId, '/resourceGroups/', resourceGroup().name, '/providers/Microsoft.DevTestLab/labs/', parameters('labName'))]"
  },
  "resources": [
    {
      "apiVersion": "2016-05-15",
      "type": "Microsoft.DevTestLab/labs",
      "name": "[parameters('labName')]",
      "location": "[resourceGroup().location]"
    },
    {
      "apiVersion": "2016-07-01",
      "type": "Microsoft.DevTestLab/labs/providers/roleAssignments",
      "name": "[variables('fullDevTestLabUserRoleName')]",
      "properties": {
        "roleDefinitionId": "[variables('devTestLabUserRoleId')]",
        "principalId": "[parameters('principalId')]",
        "scope": "[variables('roleScope')]"
      },
      "dependsOn": [
        "[resourceId('Microsoft.DevTestLab/labs', parameters('labName'))]"
      ]
    }
  ]
}

```

Si va a asignar el rol en la misma plantilla que está creando el laboratorio, no olvide agregar una dependencia entre el recurso de la asignación de rol y el laboratorio. Para obtener más información, consulte [definición de dependencias en plantillas de Azure Resource Manager](../azure-resource-manager/resource-group-define-dependencies.md) artículo.

### <a name="role-assignment-resource-information"></a>Información de recursos de asignación de roles
El recurso de la asignación de rol debe especificar el tipo y nombre.

Lo primero que tener en cuenta es que no es el tipo del recurso `Microsoft.Authorization/roleAssignments` como se haría para un grupo de recursos.  En su lugar, el tipo de recurso sigue el patrón `{provider-namespace}/{resource-type}/providers/roleAssignments`. En este caso, será el tipo de recurso `Microsoft.DevTestLab/labs/providers/roleAssignments`.

El propio nombre de asignación de rol debe ser único globalmente.  El nombre de la asignación usa el patrón de `{labName}/Microsoft.Authorization/{newGuid}`. El `newGuid` es un valor de parámetro para la plantilla. Se asegura de que el nombre de la asignación de rol es único. Como no hay ninguna función de plantilla para crear GUID, deberá generar un GUID mediante cualquier herramienta de generador GUID.  

En la plantilla, el nombre de la asignación de rol se define mediante el `fullDevTestLabUserRoleName` variable. Es la línea exacta de la plantilla:

```json
"fullDevTestLabUserRoleName": "[concat(parameters('labName'), '/Microsoft.Authorization/', parameters('roleAssignmentGuid'))]"
```


### <a name="role-assignment-resource-properties"></a>Propiedades de recursos de asignación de rol
Una asignación de roles sí define tres propiedades. Necesita el `roleDefinitionId`, `principalId`, y `scope`.

### <a name="role-definition"></a>Definición de roles
El identificador de definición de rol es el identificador de cadena para la definición de rol existente. La función ID está en el formulario `/subscriptions/{subscription-id}/providers/Microsoft.Authorization/roleDefinitions/{role-definition-id}`. 

La suscripción se obtiene con Id. de `subscription().subscriptionId` función de plantilla.  

Deberá obtener la definición de rol para el `DevTest Labs User` rol integrado. Para obtener el GUID para el [usuario de DevTest Labs](../role-based-access-control/built-in-roles.md#devtest-labs-user) rol, puede usar el [API de REST de las asignaciones de rol](/rest/api/authorization/roleassignments) o [Get AzRoleDefinition](/powershell/module/az.resources/get-azroledefinition?view=azps-1.8.0) cmdlet.

```powershell
$dtlUserRoleDefId = (Get-AzRoleDefinition -Name "DevTest Labs User").Id
```

El identificador de rol se define en la sección de variables y denominado `devTestLabUserRoleId`. En la plantilla, el identificador de rol se establece en: 111111111-0000-0000-11111111111111111. 

```json
"devTestLabUserRoleId": "[concat('/subscriptions/', subscription().subscriptionId, '/providers/Microsoft.Authorization/roleDefinitions/111111111-0000-0000-11111111111111111')]",
```

### <a name="principal-id"></a>ID de la entidad de seguridad
Id. de entidad es el identificador de objeto de que el usuario de Active Directory, grupo o entidad de servicio que desea agregar como un usuario de laboratorio para el laboratorio. La plantilla utiliza el `ObjectId` como un parámetro.

Puede obtener el valor de ObjectId mediante el [Get-AzureRMADUser](/powershell/module/azurerm.resources/get-azurermaduser?view=azurermps-6.13.0), [Get-AzureRMADGroup o [Get-AzureRMADServicePrincipal](/powershell/module/azurerm.resources/get-azurermadserviceprincipal?view=azurermps-6.13.0) cmdlets de PowerShell. Estos cmdlets devuelven un único o listas de objetos de Active Directory que tienen una propiedad de identificador, que es el identificador de objeto que necesita. El ejemplo siguiente muestra cómo obtener el identificador de objeto de un único usuario en una empresa.

```powershell
$userObjectId = (Get-AzureRmADUser -UserPrincipalName ‘email@company.com').Id
```

También puede usar los cmdlets de PowerShell de Azure Active Directory que incluyen [Get-MsolUser](/powershell/module/msonline/get-msoluser?view=azureadps-1.0), [Get-MsolGroup](/powershell/module/msonline/get-msolgroup?view=azureadps-1.0), y [Get-MsolServicePrincipal](/powershell/module/msonline/get-msolserviceprincipal?view=azureadps-1.0).

### <a name="scope"></a>Scope
Ámbito especifica el recurso o grupo de recursos para el que se debe aplicar la asignación de roles. Para los recursos, el ámbito está en el formulario: `/subscriptions/{subscription-id}/resourceGroups/{resource-group-name}/providers/{provider-namespace}/{resource-type}/{resource-name}`. La plantilla utiliza el `subscription().subscriptionId` función para rellenar el `subscription-id` parte y el `resourceGroup().name` función de plantilla para rellenar el `resource-group-name` parte. Uso de estas funciones significa que el laboratorio a la que va a asignar un rol debe existir en la suscripción actual y el mismo grupo de recursos al que se realiza la implementación de plantilla. La última parte, `resource-name`, es el nombre del laboratorio. Este valor se recibe a través del parámetro de plantilla en este ejemplo. 

El ámbito de rol en la plantilla: 

```json
"roleScope": "[concat('/subscriptions/', subscription().subscriptionId, '/resourceGroups/', resourceGroup().name, '/providers/Microsoft.DevTestLab/labs/', parameters('labName'))]"
```

### <a name="deploying-the-template"></a>Implementación de la plantilla
En primer lugar, cree un archivo de parámetros (por ejemplo: azuredeploy.parameters.json) que pasa los valores de parámetros en la plantilla de Resource Manager. 

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "principalId": {
      "value": "11111111-1111-1111-1111-111111111111"
    },
    "labName": {
      "value": "MyLab"
    },
    "roleAssignmentGuid": {
      "value": "22222222-2222-2222-2222-222222222222"
    }
  }
}
```

A continuación, utilice el [New-AzureRmResourceGroupDeployment](/powershell/module/azurerm.resources/new-azurermresourcegroupdeployment?view=azurermps-6.13.0) cmdlet de PowerShell para implementar la plantilla de Resource Manager. El comando de ejemplo siguiente asigna una persona, grupo o una entidad de servicio para el rol de usuario de DevTest Labs para un laboratorio.

```powershell
New-AzureRmResourceGroupDeployment -Name "MyLabResourceGroup-$(New-Guid)" -ResourceGroupName 'MyLabResourceGroup' -TemplateParameterFile .\azuredeploy.parameters.json -TemplateFile .\azuredeploy.json
```

Es importante tener en cuenta que la implementación nombre y la función de asignación de grupo GUID deben ser únicos. Si intenta implementar una asignación de recursos con un GUID no es único, obtendrá un `RoleAssignmentUpdateNotPermitted` error.

Si tiene previsto usar la plantilla varias veces para agregar varios objetos de Active Directory para el rol de usuario de DevTest Labs para el laboratorio, considere la posibilidad de utilizar objetos dinámicos en el comando de PowerShell. En el ejemplo siguiente se usa el [New-Guid](/powershell/module/Microsoft.PowerShell.Utility/New-Guid?view=powershell-5.0) cmdlet para especificar el grupo implementación nombre y la función de asignación de recursos GUID dinámicamente.

```powershell
New-AzureRmResourceGroupDeployment -Name "MyLabResourceGroup-$(New-Guid)" -ResourceGroupName 'MyLabResourceGroup' -TemplateFile .\azuredeploy.json -roleAssignmentGuid "$(New-Guid)" -labName "MyLab" -principalId "11111111-1111-1111-1111-111111111111"
```

## <a name="use-azure-powershell"></a>Uso de Azure PowerShell
Como se describe en la introducción, cree una nueva asignación de rol de Azure para agregar un usuario a la **usuario de DevTest Labs** rol para el laboratorio. En PowerShell, hacerlo mediante el [New-AzureRMRoleAssignment](/powershell/module/azurerm.resources/new-azurermroleassignment?view=azurermps-6.13.0) cmdlet. Este cmdlet tiene muchos parámetros opcionales para permitir una flexibilidad. El `ObjectId`, `SigninName`, o `ServicePrincipalName` puede especificarse como el objeto que se va a conceder permisos.  

Este es un comando de PowerShell de Azure de ejemplo que agrega un usuario al rol de usuario de DevTest Labs en el laboratorio especificado.

```powershell
New-AzureRmRoleAssignment -UserPrincipalName <email@company.com> -RoleDefinitionName 'DevTest Labs User' -ResourceName '<Lab Name>' -ResourceGroupName '<Resource Group Name>' -ResourceType 'Microsoft.DevTestLab/labs'
```

Para especificar el recurso al que se están los permisos concedidos puede especificarse mediante una combinación de `ResourceName`, `ResourceType`, `ResourceGroup` o por el `scope` parámetro. Se utiliza cualquier combinación de parámetros, proporcione suficiente información para el cmdlet para identificar de forma única el objeto de Active Directory (usuario, grupo o entidad de servicio), ámbito (grupo de recursos o recurso) y definición de rol.

## <a name="use-azure-command-line-interface-cli"></a>Usar la interfaz de línea de comandos (CLI) de Azure
En la CLI de Azure, la adición de un usuario de laboratorios a un laboratorio se realiza mediante el uso de la `az role assignment create` comando. Para obtener más información sobre los cmdlets de CLI de Azure, consulte [administrar el acceso a recursos de Azure mediante RBAC y la CLI de Azure](../role-based-access-control/role-assignments-cli.md).

Se puede especificar el objeto que se va a conceder acceso por la `objectId`, `signInName`, `spn` parámetros. El laboratorio a la que se va a conceder acceso el objeto puede identificarse por el `scope` dirección url o una combinación de la `resource-name`, `resource-type`, y `resource-group` parámetros.

En el siguiente ejemplo de CLI de Azure se muestra cómo agregar a una persona a la función de usuario de DevTest Labs del laboratorio de pruebas especificado.  

```azurecli
az role assignment create --roleName "DevTest Labs User" --signInName <email@company.com> -–resource-name "<Lab Name>" --resource-type “Microsoft.DevTestLab/labs" --resource-group "<Resource Group Name>"
```

## <a name="next-steps"></a>Pasos siguientes
Consulte los artículos siguientes:

- [Crear y administrar máquinas virtuales con DevTest Labs mediante la CLI de Azure](devtest-lab-vmcli.md)
- [Crear una máquina virtual con DevTest Labs mediante Azure PowerShell](devtest-lab-vm-powershell.md)
- [Usar herramientas de línea de comandos para iniciar y detener las máquinas virtuales de Azure DevTest Labs](use-command-line-start-stop-virtual-machines.md)

