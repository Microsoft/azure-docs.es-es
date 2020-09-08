---
title: 'Inicio rápido: Creación de un plano técnico con la CLI de Azure'
description: En este inicio rápido se usa Azure Blueprints para crear, definir e implementar artefactos mediante la CLI de Azure.
ms.date: 06/02/2020
ms.topic: quickstart
ms.openlocfilehash: c64723befbe3cb8d811179e7062df18606749783
ms.sourcegitcommit: 8a7b82de18d8cba5c2cec078bc921da783a4710e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89051549"
---
# <a name="quickstart-define-and-assign-an-azure-blueprint-with-azure-cli"></a>Inicio rápido: Definición y asignación de un plano técnico de Azure Blueprint con la CLI de Azure

Aprender a crear y asignar planos técnicos permite definir patrones comunes para desarrollar configuraciones reutilizables y de implementación rápida basadas en plantillas de Azure Resource Manager, directivas, seguridad, etc. En este tutorial, aprenderá a usar planos técnicos de Azure Blueprint para realizar algunas de las tareas más comunes relacionadas con la creación, asignación y administración de directivas en toda la organización, como, por ejemplo:

## <a name="prerequisites"></a>Requisitos previos

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free) antes de empezar.

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

## <a name="add-the-blueprint-extension"></a>Adición de la extensión de plano técnico

Para habilitar la CLI de Azure para administrar definiciones y asignaciones de plano técnico, se debe agregar la extensión.
Esta extensión funciona siempre que se puede usar la CLI de Azure, incluidos [Bash en Windows 10](/windows/wsl/install-win10), [Cloud Shell](https://shell.azure.com) (independiente y dentro del portal), la [imagen de Docker de la CLI de Azure](https://hub.docker.com/r/microsoft/azure-cli/) o instalado localmente.

1. Compruebe que está instalada la versión más reciente de la CLI de Azure (al menos la **2.0.76**). Si todavía no está instalado, siga [estas instrucciones](/cli/azure/install-azure-cli-windows).

1. En el entorno que prefiera de la CLI de Azure, impórtela con el siguiente comando:

   ```azurecli-interactive
   # Add the Blueprint extension to the Azure CLI environment
   az extension add --name blueprint
   ```

1. Compruebe que la extensión se ha instalado y que es la versión esperada (al menos **0.1.0**):

   ```azurecli-interactive
   # Check the extension list (note that you may have other extensions installed)
   az extension list

   # Run help for extension options
   az blueprint -h
   ```

## <a name="create-a-blueprint"></a>Creación de un plano técnico

El primer paso para definir un patrón estándar de cumplimiento es elaborar un plano técnico a partir de los recursos disponibles. Vamos a crear un plano técnico llamado "MyBlueprint" para configurar las asignaciones de roles y de directivas de la suscripción. A continuación, vamos a agregar un grupo de recursos, una plantilla de Resource Manager y una asignación de roles en el grupo de recursos.

> [!NOTE]
> Cuando se usa la CLI de Azure, el objeto _blueprint_ se crea en primer lugar. Para cada  _artefacto_ que se agregue y que tenga parámetros, estos deben definirse de antemano en el _plano técnico_ inicial.

1. Cree el objeto _blueprint_ inicial. El parámetro **parameters** toma un archivo JSON que incluye todos los parámetros de nivel de plano técnico. Los parámetros se establecen durante la asignación y son utilizados por los artefactos que se agregan en pasos posteriores.

   - Archivo JSON: blueprintparms.json

     ```json
     {
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
     }
     ```

   - Comando de la CLI de Azure

     ```azurecli-interactive
     # Login first with az login if not using Cloud Shell

     # Create the blueprint object
     az blueprint create \
        --name 'MyBlueprint' \
        --description 'This blueprint sets tag policy and role assignment on the subscription, creates a ResourceGroup, and deploys a resource template and role assignment to that ResourceGroup.' \
        --parameters blueprintparms.json
     ```

     > [!NOTE]
     > Use el nombre de archivo _blueprint.json_ al importar las definiciones de plano técnico.
     > Este nombre de archivo se usa al llamar a [az blueprint import](/cli/azure/ext/blueprint/blueprint#ext-blueprint-az-blueprint-import).

     El objeto blueprint se crea en la suscripción predeterminada de forma predeterminada. Para especificar el grupo de administración, use el parámetro **managementgroup**. Para especificar la suscripción, use el parámetro **subscription**.

1. Agregue el grupo de recursos para los artefactos de almacenamiento a la definición.

   ```azurecli-interactive
   az blueprint resource-group add \
      --blueprint-name 'MyBlueprint' \
      --artifact-name 'storageRG' \
      --description 'Contains the resource template deployment and a role assignment.'
   ```

1. Agregue una asignación de roles a la suscripción. En el siguiente ejemplo, las identidades de la entidad de servicio a las que se ha asignado el rol especificado se configuran con un parámetro que se establece durante la asignación de planos técnicos. Este ejemplo se usa el rol integrado _Colaborador_ con el GUID `b24988ac-6180-42a0-ab88-20f7382dd24c`.

   ```azurecli-interactive
   az blueprint artifact role create \
      --blueprint-name 'MyBlueprint' \
      --artifact-name 'roleContributor' \
      --role-definition-id '/providers/Microsoft.Authorization/roleDefinitions/b24988ac-6180-42a0-ab88-20f7382dd24c' \
      --principal-ids "[parameters('contributors')]"
   ```

1. Agregue la asignación de directivas a la suscripción. En este ejemplo se usa la directiva integrada _Aplicar una etiqueta y su valor predeterminado a los grupos de recursos_ con el GUID `49c88fc8-6fd1-46fd-a676-f12d1d3a4c71`.

   - Archivo JSON: \artifacts\policyTags.json

     ```json
     {
        "tagName": {
           "value": "[parameters('tagName')]"
        },
        "tagValue": {
           "value": "[parameters('tagValue')]"
        }
     }
     ```

   - Comando de la CLI de Azure

     ```azurecli-interactive
     az blueprint artifact policy create \
        --blueprint-name 'MyBlueprint' \
        --artifact-name 'policyTags' \
        --policy-definition-id '/providers/Microsoft.Authorization/policyDefinitions/49c88fc8-6fd1-46fd-a676-f12d1d3a4c71' \
        --display-name 'Apply tag and its default value to resource groups' \
        --description 'Apply tag and its default value to resource groups' \
        --parameters artifacts\policyTags.json
     ```

1. Agregue otra asignación de directiva para la etiqueta Storage (reutilizando el parámetro _storageAccountType_) en la suscripción. Este artefacto de asignación de directivas adicional muestra que un parámetro definido en el plano técnico lo puede utilizar más de un artefacto. En el ejemplo, **storageAccountType** se usa para establecer una etiqueta en el grupo de recursos. Este valor proporciona información acerca de la cuenta de almacenamiento que se crea en el paso siguiente. En este ejemplo se usa la directiva integrada _Aplicar una etiqueta y su valor predeterminado a los grupos de recursos_ con el GUID `49c88fc8-6fd1-46fd-a676-f12d1d3a4c71`.

   - Archivo JSON: \artifacts\policyStorageTags.json

     ```json
     {
        "tagName": {
           "value": "StorageType"
        },
        "tagValue": {
           "value": "[parameters('storageAccountType')]"
        }
     }
     ```

   - Comando de la CLI de Azure

     ```azurecli-interactive
     az blueprint artifact policy create \
        --blueprint-name 'MyBlueprint' \
        --artifact-name 'policyStorageTags' \
        --policy-definition-id '/providers/Microsoft.Authorization/policyDefinitions/49c88fc8-6fd1-46fd-a676-f12d1d3a4c71' \
        --display-name 'Apply storage tag to resource group' \
        --description 'Apply storage tag and the parameter also used by the template to resource groups' \
        --parameters artifacts\policyStorageTags.json
     ```

1. Agregue la plantilla en el grupo de recursos. El parámetro **template** de una plantilla de Resource Manager incluye los componentes JSON normales de la plantilla. La plantilla también reutiliza los parámetros del plano técnico **storageAccountType**, **tagName** y **tagValue**, cada uno de los cuales se pasa a la plantilla. Los parámetros del plano técnico se ponen a disposición de la plantilla mediante el parámetro **parameters** y dentro de la plantilla JSON en la que se utiliza el par clave-valor para insertar el valor. Los nombres de los parámetros blueprint y template podrían ser los mismos.

   - Archivo de la plantilla de Resource Manager de JSON: artifacts\templateStorage.json

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

   - Archivo de parámetros de la plantilla de Resource Manager de JSON: artifacts\templateStorageParams.json

     ```json
     {
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
     ```

   - Comando de la CLI de Azure

     ```azurecli-interactive
     az blueprint artifact template create \
        --blueprint-name 'MyBlueprint' \
        --artifact-name 'templateStorage' \
        --template artifacts\templateStorage.json \
        --parameters artifacts\templateStorageParams.json \
        --resource-group-art 'storageRG'
     ```

1. Agregue la asignación de roles en el grupo de recursos. Similar a la entrada anterior de asignación de roles, el ejemplo siguiente utiliza el identificador de definición para el rol **Propietario** y le proporciona un parámetro diferente del plano técnico. Este ejemplo se usa el rol integrado _Propietario_ con el GUID `8e3af657-a8ff-443c-a75c-2fe8c4bcb635`.

   ```azurecli-interactive
   az blueprint artifact role create \
      --blueprint-name 'MyBlueprint' \
      --artifact-name 'roleOwner' \
      --role-definition-id '/providers/Microsoft.Authorization/roleDefinitions/8e3af657-a8ff-443c-a75c-2fe8c4bcb635' \
      --principal-ids "[parameters('owners')]" \
      --resource-group-art 'storageRG'
   ```

## <a name="publish-a-blueprint"></a>Publicación de un plano técnico

Ahora que los artefactos se han agregado al plano técnico, es momento de publicarlo. La publicación lo hace disponible para ser asignado a una suscripción.

```azurecli-interactive
az blueprint publish --blueprint-name 'MyBlueprint' --version '{BlueprintVersion}'
```

El valor de `{BlueprintVersion}` es una cadena de letras, números y guiones (sin espacios ni otros caracteres especiales) con una longitud máxima de 20 caracteres. Utilice un nombre único e informativo, como **v20200605-135541**.

## <a name="assign-a-blueprint"></a>Asignación de un plano técnico

Cuando se ha publicado un plano técnico mediante la CLI de Azure, se puede asignar a una suscripción. Asigne el plano técnico creado a una de las suscripciones de la jerarquía del grupo de administración. Si el proyecto se guarda en una suscripción, solo se puede asignar a dicha suscripción. El parámetro **blueprint-name** especifica el plano técnico que se va a asignar. Para proporcionar los parámetros de nombre, ubicación, identidad, bloqueo y plano técnico, use los parámetros de la CLI de Azure coincidentes en el comando `az blueprint assignment create` o proporciónelos en el archivo JSON **parameters**.

1. Ejecute la implementación del plano técnico asignándolo a una suscripción. Dado que los parámetros **contributors** y **owners** requieren una matriz de objectId de las entidades de servicio para que se les conceda la asignación de roles, utilice [Graph API de Azure Active Directory](../../active-directory/develop/active-directory-graph-api.md) para recopilar los elementos objectId que se utilizarán en el archivo **parameters** para sus propios usuarios, grupos o entidades de servicio.

   - Archivo JSON: blueprintAssignment.json

     ```json
     {
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
     ```

   - Comando de la CLI de Azure

     ```azurecli-interactive
     az blueprint assignment create \
        --name 'assignMyBlueprint' \
        --location 'westus' \
        --resource-group-value artifact_name=storageRG name=StorageAccount location=eastus \
        --parameters blueprintAssignment.json
     ```

   - Identidad administrada asignada por el usuario

     Una asignación de plano técnico también puede usar una [identidad administrada asignada por el usuario](../../active-directory/managed-identities-azure-resources/overview.md).
     En este caso, el parámetro **identity-type** se establece en _UserAssigned_ y el parámetro **user-assigned-identities** especifica la identidad. Reemplace `{userIdentity}` por el nombre de la identidad administrada asignada por el usuario.

     ```azurecli-interactive
     az blueprint assignment create \
        --name 'assignMyBlueprint' \
        --location 'westus' \
        --identity-type UserAssigned \
        --user-assigned-identities {userIdentity} \
        --resource-group-value artifact_name=storageRG name=StorageAccount location=eastus \
        --parameters blueprintAssignment.json
     ```

     La **identidad administrada asignada por el usuario** puede estar en cualquier suscripción y grupo de recursos para los que tenga permiso el usuario que asigna el plano técnico.

     > [!IMPORTANT]
     > Azure Blueprints no administra la identidad administrada asignada por el usuario. Los usuarios son responsables de asignar los roles y permisos necesarios o, de lo contrario, se producirá un error en la asignación del plano técnico.

## <a name="clean-up-resources"></a>Limpieza de recursos

### <a name="unassign-a-blueprint"></a>Cancelación de la asignación de un plano técnico

Puede eliminar un plano de una suscripción. A menudo, la eliminación se realiza cuando ya no son necesarios los recursos de artefacto. Cuando se quita un plano técnico, se omiten los artefactos que se asignaron como parte de ese plano técnico. Para eliminar una asignación de plano técnico, use el comando `az blueprint assignment delete`:

```azurecli-interactive
az blueprint assignment delete --name 'assignMyBlueprint'
```

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, ha creado, asignado y eliminado un plano técnico con la CLI de Azure. Para más información sobre Azure Blueprints, vaya el artículo sobre el ciclo de vida de los planos técnicos.

> [!div class="nextstepaction"]
> [Más información acerca del ciclo de vida del plano técnico](./concepts/lifecycle.md)