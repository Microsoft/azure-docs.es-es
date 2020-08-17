---
title: Publicación de la aplicación administrada del catálogo de servicios
description: Se explica cómo crear una aplicación administrada de Azure diseñada para los miembros de su organización.
author: tfitzmac
ms.topic: quickstart
ms.custom: subject-armqs, devx-track-azurecli
ms.date: 04/14/2020
ms.author: tomfitz
ms.openlocfilehash: f6a3a16fe7fd6b0036b36520262e85a9066a4e63
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87497861"
---
# <a name="quickstart-create-and-publish-a-managed-application-definition"></a>Inicio rápido: Creación y publicación de una definición de aplicación administrada

En este inicio rápido se proporciona una introducción al uso de [Azure Managed Applications](overview.md). Puede crear y publicar una aplicación administrada que esté diseñada para los miembros de su organización.

Para publicar una aplicación administrada en el catálogo de servicios, debe hacer lo siguiente:

* Cree una plantilla que defina los recursos que va a implementar con la aplicación administrada.
* Defina los elementos de la interfaz de usuario del portal cuando implemente la aplicación administrada.
* Cree un paquete _.zip_ que contenga los archivos de plantilla necesarios.
* Decida qué usuario, grupo o aplicación necesita acceder al grupo de recursos en la suscripción del usuario.
* Cree la definición de aplicación administrada que apunte al paquete _.zip_ y solicite la identidad de acceso.

## <a name="create-the-arm-template"></a>Creación de la plantilla de Resource Manager

Todas las definiciones de aplicaciones administradas incluyen un archivo denominado _mainTemplate.json_. En él, se definen los recursos de Azure que desea implementar. La plantilla no es diferente de una plantilla de Resource Manager normal.

Cree un archivo denominado _mainTemplate.json_. El nombre distingue mayúsculas de minúsculas.

Agregue el siguiente JSON al archivo. Define los parámetros para crear una cuenta de almacenamiento y especifica las propiedades de la cuenta de almacenamiento.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "storageAccountNamePrefix": {
            "type": "string"
        },
        "storageAccountType": {
            "type": "string"
        },
        "location": {
            "type": "string",
            "defaultValue": "[resourceGroup().location]"
        }
    },
    "variables": {
        "storageAccountName": "[concat(parameters('storageAccountNamePrefix'), uniqueString(resourceGroup().id))]"
    },
    "resources": [
        {
            "type": "Microsoft.Storage/storageAccounts",
            "apiVersion": "2019-06-01",
            "name": "[variables('storageAccountName')]",
            "location": "[parameters('location')]",
            "sku": {
                "name": "[parameters('storageAccountType')]"
            },
            "kind": "StorageV2",
            "properties": {}
        }
    ],
    "outputs": {
        "storageEndpoint": {
            "type": "string",
            "value": "[reference(resourceId('Microsoft.Storage/storageAccounts/', variables('storageAccountName')), '2019-06-01').primaryEndpoints.blob]"
        }
    }
}
```

Guarde el archivo _mainTemplate.json_.

## <a name="define-your-create-experience"></a>Definición de la experiencia de creación

Como publicador, defina la experiencia del portal para crear la aplicación administrada. El archivo _createUiDefinition.json_ genera la interfaz del portal. Defina cómo los usuarios proporcionan la entrada para cada parámetro mediante [elementos de control](create-uidefinition-elements.md), incluidas las listas desplegables, los cuadros de texto y los cuadros de contraseña.

Cree un archivo denominado _createUiDefinition.json_ (distingue mayúsculas de minúsculas).

Agregue el siguiente código de inicio JSON y guárdelo.

```json
{
   "$schema": "https://schema.management.azure.com/schemas/0.1.2-preview/CreateUIDefinition.MultiVm.json#",
   "handler": "Microsoft.Azure.CreateUIDef",
   "version": "0.1.2-preview",
   "parameters": {
        "basics": [
            {}
        ],
        "steps": [
            {
                "name": "storageConfig",
                "label": "Storage settings",
                "subLabel": {
                    "preValidation": "Configure the infrastructure settings",
                    "postValidation": "Done"
                },
                "bladeTitle": "Storage settings",
                "elements": [
                    {
                        "name": "storageAccounts",
                        "type": "Microsoft.Storage.MultiStorageAccountCombo",
                        "label": {
                            "prefix": "Storage account name prefix",
                            "type": "Storage account type"
                        },
                        "defaultValue": {
                            "type": "Standard_LRS"
                        },
                        "constraints": {
                            "allowedTypes": [
                                "Premium_LRS",
                                "Standard_LRS",
                                "Standard_GRS"
                            ]
                        }
                    }
                ]
            }
        ],
        "outputs": {
            "storageAccountNamePrefix": "[steps('storageConfig').storageAccounts.prefix]",
            "storageAccountType": "[steps('storageConfig').storageAccounts.type]",
            "location": "[location()]"
        }
    }
}
```

Para más información, consulte [Introducción a CreateUiDefinition](create-uidefinition-overview.md).

## <a name="package-the-files"></a>Empaquetado de los archivos

Agregue los dos archivos a un archivo _.zip_ denominado _app.zip_. Los dos archivos tienen que estar en el nivel raíz del archivo _.zip_. Si los coloca en una carpeta, recibe un error al crear la definición de aplicación administrada que indica que los archivos requeridos no están presentes.

Cargue el paquete en una ubicación accesible desde donde pueda consumirse. Deberá proporcionar un nombre único para la cuenta de almacenamiento.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell-interactive
New-AzResourceGroup -Name storageGroup -Location eastus

$storageAccount = New-AzStorageAccount `
  -ResourceGroupName storageGroup `
  -Name "mystorageaccount" `
  -Location eastus `
  -SkuName Standard_LRS `
  -Kind StorageV2

$ctx = $storageAccount.Context

New-AzStorageContainer -Name appcontainer -Context $ctx -Permission blob

Set-AzStorageBlobContent `
  -File "D:\myapplications\app.zip" `
  -Container appcontainer `
  -Blob "app.zip" `
  -Context $ctx 
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

```azurecli-interactive
az group create --name storageGroup --location eastus

az storage account create \
    --name mystorageaccount \
    --resource-group storageGroup \
    --location eastus \
    --sku Standard_LRS \
    --kind StorageV2

az storage container create \
    --account-name mystorageaccount \
    --name appcontainer \
    --public-access blob

az storage blob upload \
    --account-name mystorageaccount \
    --container-name appcontainer \
    --name "app.zip" \
    --file "D:\myapplications\app.zip"

```

---

## <a name="create-the-managed-application-definition"></a>Creación de la definición de aplicación administrada

### <a name="create-an-azure-active-directory-user-group-or-application"></a>Creación de una aplicación o grupo de usuarios de Azure Active Directory

El siguiente paso consiste en seleccionar una aplicación, un usuario o un grupo de usuarios para administrar los recursos para el cliente. Esta identidad tiene permisos en el grupo de recursos administrado, según el rol asignado. El rol puede ser cualquier rol integrado en Azure, como Propietario o Colaborador. Para crear un grupo de usuarios de Active Directory, consulte [Creación de un grupo y adición de miembros en Azure Active Directory](../../active-directory/fundamentals/active-directory-groups-create-azure-portal.md).

Necesita el identificador de objeto del grupo de usuarios que se usará para administrar los recursos. 

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell-interactive
$groupID=(Get-AzADGroup -DisplayName mygroup).Id
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

```azurecli-interactive
groupid=$(az ad group show --group mygroup --query objectId --output tsv)
```

---

### <a name="get-the-role-definition-id"></a>Obtención del identificador de definición de rol

A continuación, necesita el identificador de definición de rol para el rol RBAC integrado que desea para conceder acceso al usuario, al grupo de usuarios o a la aplicación. Por lo general, se usa el rol Propietario, Colaborador o Lector. El comando siguiente muestra cómo obtener el identificador de definición de rol para el rol Propietario:

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell-interactive
$ownerID=(Get-AzRoleDefinition -Name Owner).Id
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

```azurecli-interactive
ownerid=$(az role definition list --name Owner --query [].name --output tsv)
```

---

### <a name="create-the-managed-application-definition"></a>Creación de la definición de aplicación administrada

Si ya no dispone de un grupo de recursos para almacenar la definición de aplicación administrada, créelo ahora:

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell-interactive
New-AzResourceGroup -Name appDefinitionGroup -Location westcentralus
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

```azurecli-interactive
az group create --name appDefinitionGroup --location westcentralus
```

---

Ahora, cree el recurso de la definición de aplicación administrada.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell-interactive
$blob = Get-AzStorageBlob -Container appcontainer -Blob app.zip -Context $ctx

New-AzManagedApplicationDefinition `
  -Name "ManagedStorage" `
  -Location "westcentralus" `
  -ResourceGroupName appDefinitionGroup `
  -LockLevel ReadOnly `
  -DisplayName "Managed Storage Account" `
  -Description "Managed Azure Storage Account" `
  -Authorization "${groupID}:$ownerID" `
  -PackageFileUri $blob.ICloudBlob.StorageUri.PrimaryUri.AbsoluteUri
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

```azurecli-interactive
blob=$(az storage blob url --account-name mystorageaccount --container-name appcontainer --name app.zip --output tsv)

az managedapp definition create \
  --name "ManagedStorage" \
  --location "westcentralus" \
  --resource-group appDefinitionGroup \
  --lock-level ReadOnly \
  --display-name "Managed Storage Account" \
  --description "Managed Azure Storage Account" \
  --authorizations "$groupid:$ownerid" \
  --package-file-uri "$blob"
```

---

Cuando finalice el comando, tendrá una definición de aplicación administrada en el grupo de recursos.

Algunos de los parámetros usados en el ejemplo anterior son:

* **grupo de recursos**: El nombre del grupo de recursos donde se creó la definición de aplicación administrada.
* **nivel de bloqueo**: El tipo de bloqueo aplicado al grupo de recursos administrado. Impide que el cliente realice operaciones no deseadas en este grupo de recursos. Actualmente, ReadOnly es el único nivel de bloqueo admitido. Cuando se especifica ReadOnly, el cliente solo puede leer los recursos presentes en el grupo de recursos administrado. Las identidades del publicador a las que se concede acceso al grupo de recursos administrados están exentas del bloqueo.
* **authorizations**: Describe el identificador de entidad de seguridad y el identificador de definición de rol que se usan para conceder el permiso al grupo de recursos administrado. Se especifica en el formato `<principalId>:<roleDefinitionId>`. Si se necesita más de un valor, especifíquelos en el formulario `<principalId1>:<roleDefinitionId1>,<principalId2>:<roleDefinitionId2>`. Los valores van separados por una coma.
* **URI del archivo de paquete**: La ubicación de un paquete _.zip_ que contiene los archivos necesarios.

## <a name="bring-your-own-storage-for-the-managed-application-definition"></a>Traiga su propio almacenamiento para la definición de aplicación administrada

Puede optar por almacenar la definición de aplicación administrada en la cuenta de almacenamiento que proporcionara durante la creación para poder administrar totalmente su ubicación y acceso en función de las necesidades regulatorias aplicables.

> [!NOTE]
> La opción para traer su propio almacenamiento solo es compatible con las implementaciones de la plantilla de Resource Manager o de la API REST de la definición de aplicación administrada.

### <a name="select-your-storage-account"></a>Selección de la cuenta de almacenamiento

Debe [crear una cuenta de almacenamiento](../../storage/common/storage-account-create.md) para que contenga la definición de aplicación administrada para su uso con el catálogo de servicios.

Copie el identificador de recurso de la cuenta de almacenamiento. Se usará más adelante al implementar la definición.

### <a name="set-the-role-assignment-for-appliance-resource-provider-in-your-storage-account"></a>Establezca la asignación de roles para "Appliance Resource Provider" en la cuenta de almacenamiento.

Para que la definición de aplicación administrada pueda implementarse en la cuenta de almacenamiento, debe conceder primero permisos de colaborador al rol **Appliance Resource Provider** para que pueda escribir los archivos de definición en el contenedor de la cuenta de almacenamiento.

1. En [Azure Portal](https://portal.azure.com), vaya a la cuenta de almacenamiento.
1. Seleccione **Control de acceso (IAM)** para mostrar la configuración del control de acceso de la cuenta de almacenamiento. Seleccione la pestaña **Asignaciones de roles** para ver la lista de asignaciones de roles.
1. En la ventana **Agregar asignación de roles**, seleccione el rol **Colaborador**. 
1. En el campo **Asignar acceso a**, seleccione **Usuario, grupo o entidad de servicio de Azure AD**.
1. En **Seleccionar**, busque el rol **Appliance Resource Provider** y selecciónelo.
1. Guarde la asignación de roles.

### <a name="deploy-the-managed-application-definition-with-an-arm-template"></a>Implementación de la definición de aplicación administrada con una plantilla de Resource Manager

Use la plantilla de Resource Manager siguiente para implementar la aplicación administrada empaquetada como una definición de aplicación administrada nueva en el catálogo de servicios cuyos archivos de definición se guardan y mantienen en la cuenta de almacenamiento del usuario:
   
```json
    {
    "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "location": {
            "type": "string",
            "defaultValue": "[resourceGroup().location]"
        },
        "applicationName": {
            "type": "string",
            "metadata": {
                "description": "Managed Application name"
            }
        },
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
        "description": "Storage Account type"
      }
    },
        "definitionStorageResourceID": {
            "type": "string",
            "metadata": {
                "description": "Storage account resource ID for where you're storing your definition"
            }
        },
        "_artifactsLocation": {
            "type": "string",
            "metadata": {
                "description": "The base URI where artifacts required by this template are located."
            }
        }
    },
    "variables": {
        "lockLevel": "None",
        "description": "Sample Managed application definition",
        "displayName": "Sample Managed application definition",
        "managedApplicationDefinitionName": "[parameters('applicationName')]",
        "packageFileUri": "[parameters('_artifactsLocation')]",
        "defLocation": "[parameters('definitionStorageResourceID')]",
        "managedResourceGroupId": "[concat(subscription().id,'/resourceGroups/', concat(parameters('applicationName'),'_managed'))]",
        "applicationDefinitionResourceId": "[resourceId('Microsoft.Solutions/applicationDefinitions',variables('managedApplicationDefinitionName'))]"
    },
    "resources": [
        {
            "type": "Microsoft.Solutions/applicationDefinitions",
            "apiVersion": "2019-07-01",
            "name": "[variables('managedApplicationDefinitionName')]",
            "location": "[parameters('location')]",
            "properties": {
                "lockLevel": "[variables('lockLevel')]",
                "description": "[variables('description')]",
                "displayName": "[variables('displayName')]",
                "packageFileUri": "[variables('packageFileUri')]",
                "storageAccountId": "[variables('defLocation')]"
            }
        }
    ],
    "outputs": {}
}
```

Se ha agregado una nueva propiedad llamada `storageAccountId` a las propiedades de `applicationDefinitions` y se proporciona como valor el identificador de la cuenta de almacenamiento en donde almacenar la definición:

Puede comprobar que los archivos de definición de aplicación se guardan en la cuenta de almacenamiento proporcionada en un contenedor titulado `applicationDefinitions`.

> [!NOTE]
> Para mayor seguridad, puede crear una definición de aplicación administrada y almacenarla en un [blob de cuenta de almacenamiento de Azure con el cifrado habilitado](../../storage/common/storage-service-encryption.md). El contenido de la definición se cifra mediante las opciones de cifrado de la cuenta de almacenamiento. Solo los usuarios con permisos para el archivo pueden ver la definición en el catálogo de servicios.

## <a name="make-sure-users-can-see-your-definition"></a>Asegurarse de que los usuarios pueden ver la definición

Tiene acceso a la definición de la aplicación administrada, pero desea asegurarse de que otros usuarios de su organización puedan acceder a ella. Concédales al menos el rol de lector en la definición. Puede haber heredado este nivel de acceso de la suscripción o del grupo de recursos. Para comprobar quién tiene acceso a la definición y agregar usuarios o grupos, consulte [Uso del control de acceso basado en rol para administrar el acceso a los recursos de la suscripción de Azure](../../role-based-access-control/role-assignments-portal.md).

## <a name="next-steps"></a>Pasos siguientes

Ha publicado la definición de aplicación administrada. Ahora, aprenda a implementar una instancia de esa definición.

> [!div class="nextstepaction"]
> [Inicio rápido: Implementación de la aplicación de catálogo de servicios](deploy-service-catalog-quickstart.md)
