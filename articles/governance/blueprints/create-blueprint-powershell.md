---
title: 'Inicio rápido: Creación de un plano técnico con PowerShell'
description: En este inicio rápido se usa Azure Blueprints para crear, definir e implementar artefactos mediante PowerShell.
ms.date: 08/27/2020
ms.topic: quickstart
ms.openlocfilehash: 339cd4628cda5f469a783db02c10f86259c93941
ms.sourcegitcommit: 8a7b82de18d8cba5c2cec078bc921da783a4710e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89051532"
---
# <a name="quickstart-define-and-assign-an-azure-blueprint-with-powershell"></a>Inicio rápido: Definición y asignación de un plano técnico de Azure Blueprint con PowerShell

Aprender a crear y asignar planos técnicos permite definir patrones comunes para desarrollar configuraciones reutilizables y de implementación rápida basadas en plantillas de Azure Resource Manager, directivas, seguridad, etc. En este tutorial, aprenderá a usar planos técnicos de Azure Blueprint para realizar algunas de las tareas más comunes relacionadas con la creación, asignación y administración de directivas en toda la organización, como, por ejemplo:

## <a name="prerequisites"></a>Requisitos previos

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free) antes de empezar.

Si aún no se ha instalado, siga las instrucciones en [Adición del módulo Az.Blueprint](./how-to/manage-assignments-ps.md#add-the-azblueprint-module) para instalar y validar el módulo **Az.Blueprint** desde la Galería de PowerShell.

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

## <a name="create-a-blueprint"></a>Creación de un plano técnico

El primer paso para definir un patrón estándar de cumplimiento es elaborar un plano técnico a partir de los recursos disponibles. Vamos a crear un plano técnico llamado "MyBlueprint" para configurar las asignaciones de roles y de directivas de la suscripción. A continuación, vamos a agregar un grupo de recursos, una plantilla de Resource Manager y una asignación de roles en el grupo de recursos.

> [!NOTE]
> Cuando se usa PowerShell, el objeto _blueprint_ se crea en primer lugar. Para cada  _artefacto_ que se agregue y que tenga parámetros, estos deben definirse de antemano en el _plano técnico_ inicial.

1. Cree el objeto _blueprint_ inicial. El parámetro **BlueprintFile** toma un archivo JSON que incluye las propiedades sobre el plano técnico, cualquier grupo de recursos que se vaya a crear y todos los parámetros en el nivel del plano técnico. Los parámetros se establecen durante la asignación y son utilizados por los artefactos que se agregan en pasos posteriores.

   - Archivo JSON: blueprint.json

     ```json
     {
         "properties": {
             "description": "This blueprint sets tag policy and role assignment on the subscription, creates a ResourceGroup, and deploys a resource template and role assignment to that ResourceGroup.",
             "targetScope": "subscription",
             "parameters": {
                 "storageAccountType": {
                     "type": "string",
                     "defaultValue": "Standard_LRS",
                     "allowedValues": [
                         "Standard_LRS",
                         "Standard_GRS",
                         "Standard_ZRS",
                         "Premium_LRS"
                     ],
                     "metadata": {
                         "displayName": "storage account type.",
                         "description": null
                     }
                 },
                 "tagName": {
                     "type": "string",
                     "metadata": {
                         "displayName": "The name of the tag to provide the policy assignment.",
                         "description": null
                     }
                 },
                 "tagValue": {
                     "type": "string",
                     "metadata": {
                         "displayName": "The value of the tag to provide the policy assignment.",
                         "description": null
                     }
                 },
                 "contributors": {
                     "type": "array",
                     "metadata": {
                         "description": "List of AAD object IDs that is assigned Contributor role at the subscription",
                         "strongType": "PrincipalId"
                     }
                 },
                 "owners": {
                     "type": "array",
                     "metadata": {
                         "description": "List of AAD object IDs that is assigned Owner role at the resource group",
                         "strongType": "PrincipalId"
                     }
                 }
             },
             "resourceGroups": {
                 "storageRG": {
                     "description": "Contains the resource template deployment and a role assignment."
                 }
             }
         }
     }
     ```

   - Comando de PowerShell

     ```azurepowershell-interactive
     # Login first with Connect-AzAccount if not using Cloud Shell

     # Get a reference to the new blueprint object, we'll use it in subsequent steps
     $blueprint = New-AzBlueprint -Name 'MyBlueprint' -BlueprintFile .\blueprint.json
     ```

     > [!NOTE]
     > Use el nombre de archivo _blueprint.json_ al crear las definiciones de plano técnico mediante programación.
     > Este nombre de archivo se usa al llamar a [Import-AzBlueprintWithArtifact](/powershell/module/az.blueprint/import-azblueprintwithartifact).

     El objeto blueprint se crea en la suscripción predeterminada de forma predeterminada. Para especificar el grupo de administración, use el parámetro **ManagementGroupId**. Para especificar la suscripción, use el parámetro **SubscriptionId**.

1. Agregue una asignación de roles a la suscripción. El **ArtifactFile** define el _tipo_ de artefacto, las propiedades se alinean con el identificador de definición de rol y las identidades principales se pasan como una matriz de valores. En el siguiente ejemplo, las identidades de la entidad de servicio a las que se ha asignado el rol especificado se configuran con un parámetro que se establece durante la asignación de planos técnicos. Este ejemplo se usa el rol integrado _Colaborador_ con el GUID `b24988ac-6180-42a0-ab88-20f7382dd24c`.

   - Archivo JSON: \artifacts\roleContributor.json

     ```json
     {
         "kind": "roleAssignment",
         "properties": {
             "roleDefinitionId": "/providers/Microsoft.Authorization/roleDefinitions/b24988ac-6180-42a0-ab88-20f7382dd24c",
             "principalIds": "[parameters('contributors')]"
         }
     }
     ```

   - Comando de PowerShell

     ```azurepowershell-interactive
     # Use the reference to the new blueprint object from the previous steps
     New-AzBlueprintArtifact -Blueprint $blueprint -Name 'roleContributor' -ArtifactFile .\artifacts\roleContributor.json
     ```

1. Agregue la asignación de directivas a la suscripción. El **ArtifactFile** define el _tipo_ de artefacto, las propiedades que se alinean con una definición de directiva o iniciativa, y configura la asignación de directivas para que utilice los parámetros definidos del plano técnico que se van a configurar durante la asignación del plano técnico. En este ejemplo se usa la directiva integrada _Aplicar una etiqueta y su valor predeterminado a los grupos de recursos_ con el GUID `49c88fc8-6fd1-46fd-a676-f12d1d3a4c71`.

   - Archivo JSON: \artifacts\policyTags.json

     ```json
     {
         "kind": "policyAssignment",
         "properties": {
             "displayName": "Apply tag and its default value to resource groups",
             "description": "Apply tag and its default value to resource groups",
             "policyDefinitionId": "/providers/Microsoft.Authorization/policyDefinitions/49c88fc8-6fd1-46fd-a676-f12d1d3a4c71",
             "parameters": {
                 "tagName": {
                     "value": "[parameters('tagName')]"
                 },
                 "tagValue": {
                     "value": "[parameters('tagValue')]"
                 }
             }
         }
     }
     ```

   - Comando de PowerShell

     ```azurepowershell-interactive
     # Use the reference to the new blueprint object from the previous steps
     New-AzBlueprintArtifact -Blueprint $blueprint -Name 'policyTags' -ArtifactFile .\artifacts\policyTags.json
     ```

1. Agregue otra asignación de directiva para la etiqueta Storage (reutilizando el parámetro _storageAccountType_) en la suscripción. Este artefacto de asignación de directivas adicional muestra que un parámetro definido en el plano técnico lo puede utilizar más de un artefacto. En el ejemplo, **storageAccountType** se usa para establecer una etiqueta en el grupo de recursos. Este valor proporciona información acerca de la cuenta de almacenamiento que se crea en el paso siguiente. En este ejemplo se usa la directiva integrada _Aplicar una etiqueta y su valor predeterminado a los grupos de recursos_ con el GUID `49c88fc8-6fd1-46fd-a676-f12d1d3a4c71`.

   - Archivo JSON: \artifacts\policyStorageTags.json

     ```json
     {
         "kind": "policyAssignment",
         "properties": {
             "displayName": "Apply storage tag to resource group",
             "description": "Apply storage tag and the parameter also used by the template to resource groups",
             "policyDefinitionId": "/providers/Microsoft.Authorization/policyDefinitions/49c88fc8-6fd1-46fd-a676-f12d1d3a4c71",
             "parameters": {
                 "tagName": {
                     "value": "StorageType"
                 },
                 "tagValue": {
                     "value": "[parameters('storageAccountType')]"
                 }
             }
         }
     }
     ```

   - Comando de PowerShell

     ```azurepowershell-interactive
     # Use the reference to the new blueprint object from the previous steps
     New-AzBlueprintArtifact -Blueprint $blueprint -Name 'policyStorageTags' -ArtifactFile .\artifacts\policyStorageTags.json
     ```

1. Agregue la plantilla en el grupo de recursos. El valor de **TemplateFile** de una plantilla de Resource Manager incluye el componente JSON normal de la plantilla. La plantilla también reutiliza los parámetros del plano técnico **storageAccountType**, **tagName** y **tagValue**, cada uno de los cuales se pasa a la plantilla. Los parámetros del plano técnico se ponen a disposición de la plantilla mediante la definición del parámetro **TemplateParameterFile** y dentro de la plantilla JSON que el par clave/valor utiliza para insertar el valor. Los nombres de los parámetros blueprint y template podrían ser los mismos.

   - Archivo de la plantilla de Resource Manager de JSON: \artifacts\templateStorage.json

     ```json
     {
         "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
         "contentVersion": "1.0.0.0",
         "parameters": {
             "storageAccountTypeFromBP": {
                 "type": "string",
                 "metadata": {
                     "description": "Storage Account type"
                 }
             },
             "tagNameFromBP": {
                 "type": "string",
                 "defaultValue": "NotSet",
                 "metadata": {
                     "description": "Tag name from blueprint"
                 }
             },
             "tagValueFromBP": {
                 "type": "string",
                 "defaultValue": "NotSet",
                 "metadata": {
                     "description": "Tag value from blueprint"
                 }
             }
         },
         "variables": {
             "storageAccountName": "[concat(uniquestring(resourceGroup().id), 'standardsa')]"
         },
         "resources": [{
             "type": "Microsoft.Storage/storageAccounts",
             "name": "[variables('storageAccountName')]",
             "apiVersion": "2016-01-01",
             "tags": {
                 "[parameters('tagNameFromBP')]": "[parameters('tagValueFromBP')]"
             },
             "location": "[resourceGroup().location]",
             "sku": {
                 "name": "[parameters('storageAccountTypeFromBP')]"
             },
             "kind": "Storage",
             "properties": {}
         }],
         "outputs": {
             "storageAccountSku": {
                 "type": "string",
                 "value": "[variables('storageAccountName')]"
             }
         }
     }
     ```

   - Archivo de parámetros de la plantilla de Resource Manager de JSON: \artifacts\templateStorageParams.json

     ```json
     {
         "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
         "contentVersion": "1.0.0.0",
         "parameters": {
             "storageAccountTypeFromBP": {
                 "value": "[parameters('storageAccountType')]"
             },
             "tagNameFromBP": {
                 "value": "[parameters('tagName')]"
             },
             "tagValueFromBP": {
                 "value": "[parameters('tagValue')]"
             }
         }
     }
     ```

   - Comando de PowerShell

     ```azurepowershell-interactive
     # Use the reference to the new blueprint object from the previous steps
     New-AzBlueprintArtifact -Blueprint $blueprint -Type TemplateArtifact -Name 'templateStorage' -TemplateFile .\artifacts\templateStorage.json -TemplateParameterFile .\artifacts\templateStorageParams.json -ResourceGroupName storageRG
     ```

1. Agregue la asignación de roles en el grupo de recursos. Similar a la entrada anterior de asignación de roles, el ejemplo siguiente utiliza el identificador de definición para el rol **Propietario** y le proporciona un parámetro diferente del plano técnico. Este ejemplo se usa el rol integrado _Propietario_ con el GUID `8e3af657-a8ff-443c-a75c-2fe8c4bcb635`.

   - Archivo JSON: \artifacts\roleOwner.json

     ```json
     {
         "kind": "roleAssignment",
         "properties": {
             "resourceGroup": "storageRG",
             "roleDefinitionId": "/providers/Microsoft.Authorization/roleDefinitions/8e3af657-a8ff-443c-a75c-2fe8c4bcb635",
             "principalIds": "[parameters('owners')]"
         }
     }
     ```

   - Comando de PowerShell

     ```azurepowershell-interactive
     # Use the reference to the new blueprint object from the previous steps
     New-AzBlueprintArtifact -Blueprint $blueprint -Name 'roleOwner' -ArtifactFile .\artifacts\roleOwner.json
     ```

## <a name="publish-a-blueprint"></a>Publicación de un plano técnico

Ahora que los artefactos se han agregado al plano técnico, es momento de publicarlo. La publicación lo hace disponible para ser asignado a una suscripción.

```azurepowershell-interactive
# Use the reference to the new blueprint object from the previous steps
Publish-AzBlueprint -Blueprint $blueprint -Version '{BlueprintVersion}'
```

El valor de `{BlueprintVersion}` es una cadena de letras, números y guiones (sin espacios ni otros caracteres especiales) con una longitud máxima de 20 caracteres. Utilice un nombre único e informativo, como **v20180622-135541**.

## <a name="assign-a-blueprint"></a>Asignación de un plano técnico

Cuando se ha publicado un plano técnico con PowerShell, se puede asignar a una suscripción. Asigne el plano técnico creado a una de las suscripciones de la jerarquía del grupo de administración. Si el proyecto se guarda en una suscripción, solo se puede asignar a dicha suscripción. El parámetro**Blueprint** especifica el plano técnico que se va a asignar. Para proporcionar los parámetros de nombre, ubicación, identidad, bloqueo y plano técnico, use los parámetros de PowerShell coincidentes en el cmdlet `New-AzBlueprintAssignment` o del archivo JSON del parámetro **AssignmentFile**.

1. Ejecute la implementación del plano técnico asignándolo a una suscripción. Como los parámetros **contributors** y **owners** requieren una serie de objectIds de las entidades de servicio para que se les conceda la asignación de roles, utilice [Graph API de Azure Active Directory](../../active-directory/develop/active-directory-graph-api.md) para recopilar los elementos objectId que se utilizarán en el **AssignmentFile** para sus propios usuarios, grupos o entidades de servicio.

   - Archivo JSON: blueprintAssignment.json

     ```json
     {
         "properties": {
             "blueprintId": "/providers/Microsoft.Management/managementGroups/{YourMG}/providers/Microsoft.Blueprint/blueprints/MyBlueprint",
             "resourceGroups": {
                 "storageRG": {
                     "name": "StorageAccount",
                     "location": "eastus2"
                 }
             },
             "parameters": {
                 "storageAccountType": {
                     "value": "Standard_GRS"
                 },
                 "tagName": {
                     "value": "CostCenter"
                 },
                 "tagValue": {
                     "value": "ContosoIT"
                 },
                 "contributors": {
                     "value": [
                         "7be2f100-3af5-4c15-bcb7-27ee43784a1f",
                         "38833b56-194d-420b-90ce-cff578296714"
                     ]
                 },
                 "owners": {
                     "value": [
                         "44254d2b-a0c7-405f-959c-f829ee31c2e7",
                         "316deb5f-7187-4512-9dd4-21e7798b0ef9"
                     ]
                 }
             }
         },
         "identity": {
             "type": "systemAssigned"
         },
         "location": "westus"
     }
     ```

   - Comando de PowerShell

     ```azurepowershell-interactive
     # Use the reference to the new blueprint object from the previous steps
     New-AzBlueprintAssignment -Blueprint $blueprint -Name 'assignMyBlueprint' -AssignmentFile .\blueprintAssignment.json
     ```

   - Identidad administrada asignada por el usuario

     Una asignación de plano técnico también puede usar una [identidad administrada asignada por el usuario](../../active-directory/managed-identities-azure-resources/overview.md).
     En este caso, la parte de **identidad** del archivo de asignación JSON cambia del modo siguiente. Reemplace `{tenantId}`, `{subscriptionId}`, `{yourRG}` y `{userIdentity}` por los valores del identificador de inquilino, el identificador de suscripción, el nombre del grupo de recursos y el nombre de su identidad administrada asignada por el usuario, respectivamente.

     ```json
     "identity": {
         "type": "userAssigned",
         "tenantId": "{tenantId}",
         "userAssignedIdentities": {
             "/subscriptions/{subscriptionId}/resourceGroups/{yourRG}/providers/Microsoft.ManagedIdentity/userAssignedIdentities/{userIdentity}": {}
         }
     },
     ```

     La **identidad administrada asignada por el usuario** puede estar en cualquier suscripción y grupo de recursos para los que tenga permiso el usuario que asigna el plano técnico.

     > [!IMPORTANT]
     > Azure Blueprints no administra la identidad administrada asignada por el usuario. Los usuarios son responsables de asignar los roles y permisos necesarios o, de lo contrario, se producirá un error en la asignación del plano técnico.

## <a name="clean-up-resources"></a>Limpieza de recursos

### <a name="unassign-a-blueprint"></a>Cancelación de la asignación de un plano técnico

Puede eliminar un plano de una suscripción. A menudo, la eliminación se realiza cuando ya no son necesarios los recursos de artefacto. Cuando se quita un plano técnico, se omiten los artefactos que se asignaron como parte de ese plano técnico. Para eliminar una asignación de plano técnico, use el cmdlet `Remove-AzBlueprintAssignment`:

assignMyBlueprint

```azurepowershell-interactive
Remove-AzBlueprintAssignment -Name 'assignMyBlueprint'
```

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, ha creado, asignado y eliminado un plano técnico con PowerShell. Para más información sobre Azure Blueprints, vaya el artículo sobre el ciclo de vida de los planos técnicos.

> [!div class="nextstepaction"]
> [Más información acerca del ciclo de vida del plano técnico](./concepts/lifecycle.md)