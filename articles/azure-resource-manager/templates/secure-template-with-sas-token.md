---
title: Implementación de forma segura de una plantilla con el token de SAS
description: Implemente recursos en Azure con una plantilla de Azure Resource Manager que está protegida con el token de SAS. Se muestra Azure PowerShell y la CLI de Azure.
ms.topic: conceptual
ms.date: 08/25/2020
ms.openlocfilehash: 8b35e82da8ebca98ec9fe1fb7441612bf61fb142
ms.sourcegitcommit: b33c9ad17598d7e4d66fe11d511daa78b4b8b330
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88855655"
---
# <a name="deploy-private-arm-template-with-sas-token"></a>Implementación de una plantilla de Resource Manager privada con el token de SAS

Cuando la plantilla de Azure Resource Manager (ARM) se encuentra en una cuenta de almacenamiento, puede restringir el acceso a la plantilla para evitar que se exponga públicamente. Para acceder a una plantilla protegida, cree un token de firma de acceso compartido (SAS) para la plantilla y proporcione ese token durante la implementación. En este artículo se explica cómo usar Azure PowerShell o la CLI de Azure para implementar una plantilla con un token de SAS.

> [!IMPORTANT]
> En lugar de proteger la plantilla con un token de SAS, considere la posibilidad de usar [especificaciones de plantilla](template-specs.md). Con las especificaciones de plantilla, puede compartir sus plantillas con otros usuarios de su organización y administrar el acceso a las plantillas a través de RBAC de Azure.

## <a name="create-storage-account-with-secured-container"></a>Creación de una cuenta de almacenamiento con un contenedor protegido

El script siguiente crea una cuenta de almacenamiento y un contenedor con acceso público desactivado.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell-interactive
New-AzResourceGroup `
  -Name ExampleGroup `
  -Location "Central US"
New-AzStorageAccount `
  -ResourceGroupName ExampleGroup `
  -Name {your-unique-name} `
  -Type Standard_LRS `
  -Location "Central US"
Set-AzCurrentStorageAccount `
  -ResourceGroupName ExampleGroup `
  -Name {your-unique-name}
New-AzStorageContainer `
  -Name templates `
  -Permission Off
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

```azurecli-interactive
az group create \
  --name "ExampleGroup" \
  --location "Central US"
az storage account create \
    --resource-group ExampleGroup \
    --location "Central US" \
    --sku Standard_LRS \
    --kind Storage \
    --name {your-unique-name}
connection=$(az storage account show-connection-string \
    --resource-group ExampleGroup \
    --name {your-unique-name} \
    --query connectionString)
az storage container create \
    --name templates \
    --public-access Off \
    --connection-string $connection
```

---

## <a name="upload-template-to-storage-account"></a>Carga de una plantilla en la cuenta de almacenamiento

Ahora puede cargar la plantilla en la cuenta de almacenamiento. Proporcione la ruta de acceso a la plantilla que quiere usar.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell-interactive
Set-AzStorageBlobContent `
  -Container templates `
  -File c:\Templates\azuredeploy.json
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

```azurecli-interactive
az storage blob upload \
    --container-name templates \
    --file azuredeploy.json \
    --name azuredeploy.json \
    --connection-string $connection
```

---

## <a name="provide-sas-token-during-deployment"></a>Provisión del token de SAS durante la implementación

Para implementar una plantilla privada en una cuenta de almacenamiento, genere un token de SAS e inclúyalo en el identificador URI de la plantilla. Establezca el tiempo de expiración con un margen suficiente para completar la implementación.

> [!IMPORTANT]
> El blob que contiene la plantilla solo es accesible para el propietario de la cuenta. Sin embargo, cuando se crea un token de SAS para el blob, el blob es accesible para cualquier persona con ese URI. Si otro usuario intercepta el URI, ese usuario podrá tener acceso a la plantilla. Un token de SAS es una buena forma de limitar el acceso a las plantillas, pero no debe incluir datos confidenciales, como contraseñas, directamente en la plantilla.
>

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell-interactive
# get the URI with the SAS token
$templateuri = New-AzStorageBlobSASToken `
  -Container templates `
  -Blob azuredeploy.json `
  -Permission r `
  -ExpiryTime (Get-Date).AddHours(2.0) -FullUri

# provide URI with SAS token during deployment
New-AzResourceGroupDeployment `
  -ResourceGroupName ExampleGroup `
  -TemplateUri $templateuri
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

En el siguiente ejemplo se trabaja con el entorno de Bash en Cloud Shell. Otros entornos pueden requerir una sintaxis diferente para crear la hora de expiración del token de SAS.

```azurecli-interactive
expiretime=$(date -u -d '30 minutes' +%Y-%m-%dT%H:%MZ)
connection=$(az storage account show-connection-string \
    --resource-group ExampleGroup \
    --name {your-unique-name} \
    --query connectionString)
token=$(az storage blob generate-sas \
    --container-name templates \
    --name azuredeploy.json \
    --expiry $expiretime \
    --permissions r \
    --output tsv \
    --connection-string $connection)
url=$(az storage blob url \
    --container-name templates \
    --name azuredeploy.json \
    --output tsv \
    --connection-string $connection)
az deployment group create \
  --resource-group ExampleGroup \
  --template-uri $url?$token
```

---

Para obtener un ejemplo del uso de un token de SAS con plantillas vinculadas, consulte [Uso de plantillas vinculadas con Azure Resource Manager](linked-templates.md).


## <a name="next-steps"></a>Pasos siguientes
* Para obtener una introducción a la implementación de plantillas, vea [Implementación de recursos con plantillas de Resource Manager y Azure PowerShell](deploy-powershell.md).
* Para definir parámetros de plantilla, consulte [Creación de plantillas](template-syntax.md#parameters).
