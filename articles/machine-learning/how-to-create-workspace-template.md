---
title: Creación de un área de trabajo con una plantilla de Azure Resource Manager
titleSuffix: Azure Machine Learning
description: Aprenda a usar una plantilla de Azure Resource Manager para crear una nueva área de trabajo de Azure Machine Learning.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.custom: how-to, devx-track-azurecli
ms.author: larryfr
author: Blackmist
ms.date: 07/27/2020
ms.openlocfilehash: 06ab819065f96508bcc4ebd26371c743c89b9220
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87487809"
---
# <a name="use-an-azure-resource-manager-template-to-create-a-workspace-for-azure-machine-learning"></a>Uso de una plantilla de Azure Resource Manager para crear un área de trabajo para Azure Machine Learning

[!INCLUDE [aml-applies-to-basic-enterprise-sku](../../includes/aml-applies-to-basic-enterprise-sku.md)]
<br>

En este artículo aprenderá varias formas de crear un área de trabajo de Azure Machine Learning mediante plantillas de Azure Resource Manager. Una plantilla de Resource Manager facilita la creación de recursos en una única operación coordinada. Una plantilla es un documento JSON que define los recursos que son necesarios para una implementación. También puede especificar los parámetros de implementación. Los parámetros se emplean para proporcionar valores de entrada cuando se usa la plantilla.

Para obtener más información, consulte [Implementación de una aplicación con la plantilla de Azure Resource Manager](../azure-resource-manager/templates/deploy-powershell.md).

## <a name="prerequisites"></a>Requisitos previos

* Una **suscripción de Azure**. Si no tiene una ya, pruebe la [versión gratuita o de pago de Azure Machine Learning](https://aka.ms/AMLFree).

* Para usar una plantilla desde una CLI, necesita [Azure PowerShell](https://docs.microsoft.com/powershell/azure/?view=azps-1.2.0) o la [CLI de Azure](https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest).

## <a name="workspace-resource-manager-template"></a>Plantilla de Resource Manager de área de trabajo

La plantilla de Azure Resource Manager utilizada en este documento se encuentra en el directorio [201-machine-learning-advanced](https://github.com/Azure/azure-quickstart-templates/blob/master/201-machine-learning-advanced/azuredeploy.json) del repositorio de GitHub de plantillas de inicio rápido de Azure.

Esta plantilla crea los siguientes servicios de Azure:

* Cuenta de Azure Storage
* Azure Key Vault
* Azure Application Insights
* Azure Container Registry
* Área de trabajo de Azure Machine Learning

El grupo de recursos es el contenedor que contiene los servicios. El área de trabajo de Azure Machine Learning requiere los diversos servicios.

La plantilla de ejemplo tiene dos parámetros **necesarios**:

* La **ubicación** en la que se crearán los recursos.

    La plantilla utilizará la ubicación que seleccione para la mayoría de los recursos. La excepción es el servicio Application Insights que no está disponible en todas las ubicaciones en las que están los demás servicios. Si selecciona una ubicación donde no está disponible, el servicio se creará en la ubicación Centro-sur de EE. UU.

* El **nombre del área de trabajo**, que es el nombre descriptivo del área de trabajo de Azure Machine Learning.

    > [!NOTE]
    > El nombre del área de trabajo no distingue mayúsculas de minúsculas.

    Los nombres de los demás servicios se generan aleatoriamente.

> [!TIP]
> Aunque la plantilla asociada a este documento crea un nuevo registro de contenedor de Azure, también puede crear una nueva área de trabajo sin necesidad de generar un registro de contenedor. Se creará uno cuando se realice una operación que lo requiera. Por ejemplo, al entrenar o implementar un modelo.
>
> También se puede hacer referencia a un registro de contenedor o una cuenta de almacenamiento existentes en la plantilla de Azure Resource Manager, en lugar de crear otros nuevos. Sin embargo, el registro de contenedor que use debe tener la __cuenta de administrador__ habilitada. Para información sobre cómo habilitar la cuenta de administrador, consulte [Cuenta de administrador](/azure/container-registry/container-registry-authentication#admin-account).

[!INCLUDE [machine-learning-delete-acr](../../includes/machine-learning-delete-acr.md)]

Para más información sobre las plantillas, consulte los siguientes artículos:

* [Creación de plantillas del Administrador de recursos de Azure](../azure-resource-manager/templates/template-syntax.md)
* [Implementación de una aplicación con las plantillas de Azure Resource Manager](../azure-resource-manager/templates/deploy-powershell.md)
* [Tipos de recurso de Microsoft.MachineLearningServices](https://docs.microsoft.com/azure/templates/microsoft.machinelearningservices/allversions)

## <a name="deploy-template"></a>Implementar plantilla

Para implementar la plantilla, debe crear un grupo de recursos.

Vea la sección [Azure Portal](#use-the-azure-portal) si prefiere usar la interfaz gráfica de usuario.

# <a name="azure-cli"></a>[CLI de Azure](#tab/azcli)

```azurecli
az group create --name "examplegroup" --location "eastus"
```

# <a name="azure-powershell"></a>[Azure PowerShell](#tab/azpowershell)

```azurepowershell
New-AzResourceGroup -Name "examplegroup" -Location "eastus"
```

---

Una vez creado correctamente el grupo de recursos, implemente la plantilla con el siguiente comando:

# <a name="azure-cli"></a>[CLI de Azure](#tab/azcli)

```azurecli
az deployment group create \
    --name "exampledeployment" \
    --resource-group "examplegroup" \
    --template-uri "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/201-machine-learning-advanced/azuredeploy.json" \
    --parameters workspaceName="exampleworkspace" location="eastus"
```

# <a name="azure-powershell"></a>[Azure PowerShell](#tab/azpowershell)

```azurepowershell
New-AzResourceGroupDeployment `
  -Name "exampledeployment" `
  -ResourceGroupName "examplegroup" `
  -TemplateUri "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/201-machine-learning-advanced/azuredeploy.json" `
  -workspaceName "exampleworkspace" `
  -location "eastus"
```

---

De forma predeterminada, todos los recursos creados como parte de la plantilla son nuevos. Sin embargo, también tiene la opción de usar los recursos existentes. Si proporciona parámetros adicionales a la plantilla, puede usar los recursos existentes. Por ejemplo, si desea usar una cuenta de almacenamiento existente, establezca el valor **storageAccountOption** en **existing** y proporcione el nombre de la cuenta de almacenamiento en el parámetro **storageAccountName**.

> [!IMPORTANT]
> Si quiere usar una cuenta de Azure Storage existente, no puede ser una cuenta Premium (Premium_LRS y Premium_GRS). Tampoco puede tener un espacio de nombres jerárquico (se usa con Azure Data Lake Storage Gen2). No se admite Premium Storage ni el espacio de nombres jerárquico con la cuenta de almacenamiento predeterminada del área de trabajo.

# <a name="azure-cli"></a>[CLI de Azure](#tab/azcli)

```azurecli
az deployment group create \
    --name "exampledeployment" \
    --resource-group "examplegroup" \
    --template-uri "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/201-machine-learning-advanced/azuredeploy.json" \
    --parameters workspaceName="exampleworkspace" \
      location="eastus" \
      storageAccountOption="existing" \
      storageAccountName="existingstorageaccountname"
```

# <a name="azure-powershell"></a>[Azure PowerShell](#tab/azpowershell)

```azurepowershell
New-AzResourceGroupDeployment `
  -Name "exampledeployment" `
  -ResourceGroupName "examplegroup" `
  -TemplateUri "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/201-machine-learning-advanced/azuredeploy.json" `
  -workspaceName "exampleworkspace" `
  -location "eastus" `
  -storageAccountOption "existing" `
  -storageAccountName "existingstorageaccountname"
```

---

## <a name="deploy-an-encrypted-workspace"></a>Implementación y cifrado del área de trabajo

En la plantilla de ejemplo siguiente se muestra cómo crear un área de trabajo con tres configuraciones:

* Habilitar la configuración de alta confidencialidad para el área de trabajo
* Habilitar el cifrado para el área de trabajo
* Usar una instancia de Azure Key Vault existente para recuperar las claves administradas por el cliente

> [!IMPORTANT]
> Una vez que se ha creado un área de trabajo, no se puede cambiar la configuración de datos confidenciales, cifrado, identificador de almacén de claves o identificadores de claves. Para cambiar estos valores, debe crear una nueva área de trabajo con los nuevos valores.

Para obtener más información, consulte [Cifrado en reposo](concept-enterprise-security.md#encryption-at-rest).

> [!IMPORTANT]
> Hay algunos requisitos específicos que la suscripción debe cumplir antes de usar esta plantilla:
>
> * La aplicación __Azure Machine Learning__ debe ser un __colaborador__ para su suscripción de Azure.
> * Debe tener un almacén Azure Key Vault existente que contenga una clave de cifrado.
> * Debe tener una directiva de acceso en el almacén Azure Key Vault que conceda el acceso __obtener__, __ajustar__ y __desajustar__ a la aplicación __Azure Cosmos DB__.
> * El almacén Azure Key Vault debe estar en la misma región en la que se planea crear el área de trabajo de Azure Machine Learning.

__Para agregar la aplicación de Azure Machine Learning como un colaborador__, use los comandos siguientes:

1. Inicie sesión en la cuenta de Azure y obtenga el id. de suscripción. Esta suscripción debe ser la misma que la que contiene el área de trabajo de Azure Machine Learning.  

    # <a name="azure-cli"></a>[CLI de Azure](#tab/azcli)

    ```azurecli
    az account list --query '[].[name,id]' --output tsv
    ```

    > [!TIP]
    > Para seleccionar otra suscripción, utilice el comando `az account set -s <subscription name or ID>` y especifique el nombre o identificador de la suscripción a los que desea cambiar. Para obtener más información sobre la selección de la suscripción, consulte [Uso de varias suscripciones de Azure](https://docs.microsoft.com/cli/azure/manage-azure-subscriptions-azure-cli?view=azure-cli-latest). 

    # <a name="azure-powershell"></a>[Azure PowerShell](#tab/azpowershell)

    ```azurepowershell
    Get-AzSubscription
    ```

    > [!TIP]
    > Para seleccionar otra suscripción, utilice el comando `Az-SetContext -SubscriptionId <subscription ID>` y especifique el nombre o identificador de la suscripción a los que desea cambiar. Para obtener más información sobre la selección de la suscripción, consulte [Uso de varias suscripciones de Azure](https://docs.microsoft.com/powershell/azure/manage-subscriptions-azureps?view=azps-4.3.0).

    ---

1. Para obtener el id. de objeto de la aplicación de Azure Machine Learning, use el comando siguiente. El valor puede ser distinto para cada una de las suscripciones de Azure:

    # <a name="azure-cli"></a>[CLI de Azure](#tab/azcli)

    ```azurecli
    az ad sp list --display-name "Azure Machine Learning" --query '[].[appDisplayName,objectId]' --output tsv
    ```

    # <a name="azure-powershell"></a>[Azure PowerShell](#tab/azpowershell)

    ```azurepowershell
    Get-AzADServicePrincipal --DisplayName "Azure Machine Learning" | select-object DisplayName, Id
    ```

    ---
    Este comando devuelve el id. de objeto, que es un GUID.

1. Para agregar el id. de objeto como colaborador a la suscripción, use el comando siguiente. Reemplace `<object-ID>` por el id. de objeto de la entidad de servicio. Reemplace `<subscription-ID>` por el nombre o id. de la suscripción de Azure:

    # <a name="azure-cli"></a>[CLI de Azure](#tab/azcli)

    ```azurecli
    az role assignment create --role 'Contributor' --assignee-object-id <object-ID> --subscription <subscription-ID>
    ```

    # <a name="azure-powershell"></a>[Azure PowerShell](#tab/azpowershell)

    ```azurepowershell
    New-AzRoleAssignment --ObjectId <object-ID> --RoleDefinitionName "Contributor" -Scope /subscriptions/<subscription-ID>
    ```

    ---

1. Para generar una clave en una instancia existente de Azure Key Vault, use uno de los siguientes comandos. Reemplace `<keyvault-name>` por el nombre del almacén de claves. Reemplace `<key-name>` por el nombre que va a usar para la clave:

    # <a name="azure-cli"></a>[CLI de Azure](#tab/azcli)

    ```azurecli
    az keyvault key create --vault-name <keyvault-name> --name <key-name> --protection software
    ```

    # <a name="azure-powershell"></a>[Azure PowerShell](#tab/azpowershell)

    ```azurepowershell
    Add-AzKeyVaultKey -VaultName <keyvault-name> -Name <key-name> -Destination 'Software'
    ```
    --- 

__Para agregar una directiva de acceso al almacén de claves, use los comandos siguientes__:

1. Para obtener el id. de objeto de la aplicación de Azure Cosmos DB, use el comando siguiente. El valor puede ser distinto para cada una de las suscripciones de Azure:

    # <a name="azure-cli"></a>[CLI de Azure](#tab/azcli)

    ```azurecli
    az ad sp list --display-name "Azure Cosmos DB" --query '[].[appDisplayName,objectId]' --output tsv
    ```

    # <a name="azure-powershell"></a>[Azure PowerShell](#tab/azpowershell)

    ```azurepowershell
    Get-AzADServicePrincipal --DisplayName "Azure Cosmos DB" | select-object DisplayName, Id
    ```
    ---

    Este comando devuelve el id. de objeto, que es un GUID. Guárdelo para más adelante.

1. Para establecer una directiva, use el comando siguiente. Reemplace `<keyvault-name>` por el nombre del almacén Azure Key Vault existente. Reemplace `<object-ID>` por el GUID del paso anterior:

    # <a name="azure-cli"></a>[CLI de Azure](#tab/azcli)

    ```azurecli
    az keyvault set-policy --name <keyvault-name> --object-id <object-ID> --key-permissions get unwrapKey wrapKey
    ```

    # <a name="azure-powershell"></a>[Azure PowerShell](#tab/azpowershell)
    
    ```azurepowershell
    Set-AzKeyVaultAccessPolicy -VaultName <keyvault-name> -ObjectId <object-ID> -PermissionsToKeys get, unwrapKey, wrapKey
    ```
    ---    

__A fin de obtener los valores__ para `cmk_keyvault` (id. de Key Vault) y los parámetros de `resource_cmk_uri` (URI de clave) necesarios para esta plantilla, siga los pasos siguientes:

1. Para obtener el id. de Key Vault, use el comando siguiente:

    # <a name="azure-cli"></a>[CLI de Azure](#tab/azcli)

    ```azurecli
    az keyvault show --name <keyvault-name> --query 'id' --output tsv
    ```

    # <a name="azure-powershell"></a>[Azure PowerShell](#tab/azpowershell)

    ```azurepowershell
    Get-AzureRMKeyVault -VaultName '<keyvault-name>'
    ```
    ---

    Este comando devuelve un valor similar a `/subscriptions/{subscription-guid}/resourceGroups/<resource-group-name>/providers/Microsoft.KeyVault/vaults/<keyvault-name>`.

1. Para obtener el valor del URI de la clave administrada por el cliente, use el comando siguiente:

    # <a name="azure-cli"></a>[CLI de Azure](#tab/azcli)

    ```azurecli
    az keyvault key show --vault-name <keyvault-name> --name <key-name> --query 'key.kid' --output tsv
    ```

    # <a name="azure-powershell"></a>[Azure PowerShell](#tab/azpowershell)

    ```azurepowershell
    Get-AzureKeyVaultKey -VaultName '<keyvault-name>' -KeyName '<key-name>'
    ```
    ---

    Este comando devuelve un valor similar a `https://mykeyvault.vault.azure.net/keys/mykey/{guid}`.

> [!IMPORTANT]
> Una vez que se ha creado un área de trabajo, no se puede cambiar la configuración de datos confidenciales, cifrado, identificador de almacén de claves o identificadores de claves. Para cambiar estos valores, debe crear una nueva área de trabajo con los nuevos valores.

Una vez que haya completado correctamente los pasos anteriores, implemente la plantilla como lo haría normalmente. Para habilitar el uso de claves administradas por el cliente, establezca los parámetros siguientes:

* **encryption_status** en **Enabled**.
* **cmk_keyvault** en el valor `cmk_keyvault` obtenido en los pasos anteriores.
* **resource_cmk_uri** en el valor `resource_cmk_uri` obtenido en los pasos anteriores.

# <a name="azure-cli"></a>[CLI de Azure](#tab/azcli)

```azurecli
az deployment group create \
    --name "exampledeployment" \
    --resource-group "examplegroup" \
    --template-uri "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/201-machine-learning-advanced/azuredeploy.json" \
    --parameters workspaceName="exampleworkspace" \
      location="eastus" \
      encryption_status="Enabled" \
      cmk_keyvault="/subscriptions/{subscription-guid}/resourceGroups/<resource-group-name>/providers/Microsoft.KeyVault/vaults/<keyvault-name>" \
      resource_cmk_uri="https://mykeyvault.vault.azure.net/keys/mykey/{guid}" \
```

# <a name="azure-powershell"></a>[Azure PowerShell](#tab/azpowershell)

```azurepowershell
New-AzResourceGroupDeployment `
  -Name "exampledeployment" `
  -ResourceGroupName "examplegroup" `
  -TemplateUri "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/201-machine-learning-advanced/azuredeploy.json" `
  -workspaceName "exampleworkspace" `
  -location "eastus" `
  -encryption_status "Enabled" `
  -cmk_keyvault "/subscriptions/{subscription-guid}/resourceGroups/<resource-group-name>/providers/Microsoft.KeyVault/vaults/<keyvault-name>" `
  -resource_cmk_uri "https://mykeyvault.vault.azure.net/keys/mykey/{guid}"
```
---

Cuando se usa una clave administrada por el cliente, Azure Machine Learning crea un grupo de recursos secundario que contiene la instancia de Cosmos DB. Para más información, consulte [Cifrado en reposo: Cosmos DB](concept-enterprise-security.md#encryption-at-rest).

Una configuración adicional que puede proporcionar para los datos es establecer el parámetro **confidential_data** en **true**. Con ello, hace lo siguiente:

* Inicia el cifrado del disco temporal local para los clústeres de proceso de Azure Machine Learning, siempre que no haya creado ningún clúster anterior en la suscripción. Si ha creado previamente un clúster en la suscripción, abra una incidencia de soporte técnico para habilitar el cifrado del disco temporal para los clústeres de proceso.
* Limpia el disco temporal local entre ejecuciones.
* Pasa de forma segura las credenciales de la cuenta de almacenamiento, el registro de contenedor y la cuenta SSH desde la capa de ejecución a los clústeres de proceso mediante el almacén de claves.
* Habilita el filtrado de IP para asegurarse de que los servicios externos que no sean AzureMachineLearningService no puedan llamar a los grupos de lotes subyacentes.

    > [!IMPORTANT]
    > Una vez que se ha creado un área de trabajo, no se puede cambiar la configuración de datos confidenciales, cifrado, identificador de almacén de claves o identificadores de claves. Para cambiar estos valores, debe crear una nueva área de trabajo con los nuevos valores.

  Para obtener más información, consulte [Cifrado en reposo](concept-enterprise-security.md#encryption-at-rest).

## <a name="deploy-workspace-behind-a-virtual-network"></a>Implementación del área de trabajo detrás de una red virtual

Al establecer el valor del parámetro `vnetOption` en `new` o `existing`, podrá crear los recursos usados por un área de trabajo detrás de una red virtual.

> [!IMPORTANT]
> En el caso del registro de contenedor, solo se admite la SKU "Premium".

> [!IMPORTANT]
> Application Insights no admite la implementación detrás de una red virtual.

### <a name="only-deploy-workspace-behind-private-endpoint"></a>Implementación del área de trabajo solo detrás de un punto de conexión privado

Si los recursos asociados no están detrás de una red virtual, puede establecer el parámetro **privateEndpointType** en `AutoAproval` o `ManualApproval` para implementar el área de trabajo detrás de un punto de conexión privado. Esto puede hacerse tanto en áreas de trabajo nuevas como existentes. Al actualizar un área de trabajo existente, rellene los parámetros de la plantilla con la información del área de trabajo existente.

> [!IMPORTANT]
> La implementación solo es válida en las regiones que admiten puntos de conexión privados.

# <a name="azure-cli"></a>[CLI de Azure](#tab/azcli)

```azurecli
az deployment group create \
    --name "exampledeployment" \
    --resource-group "examplegroup" \
    --template-uri "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/201-machine-learning-advanced/azuredeploy.json" \
    --parameters workspaceName="exampleworkspace" \
      location="eastus" \
      privateEndpointType="AutoApproval"
```

# <a name="azure-powershell"></a>[Azure PowerShell](#tab/azpowershell)

```azurepowershell
New-AzResourceGroupDeployment `
  -Name "exampledeployment" `
  -ResourceGroupName "examplegroup" `
  -TemplateUri "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/201-machine-learning-advanced/azuredeploy.json" `
  -workspaceName "exampleworkspace" `
  -location "eastus" `
  -privateEndpointType "AutoApproval"
```

---

### <a name="use-a-new-virtual-network"></a>Uso de una nueva red virtual

Para implementar un recurso detrás de una nueva red virtual, establezca **vnetOption** en **new** junto con la configuración de red virtual para el recurso correspondiente. En la implementación siguiente se muestra cómo implementar un área de trabajo con el recurso de la cuenta de almacenamiento detrás de una nueva red virtual.

# <a name="azure-cli"></a>[CLI de Azure](#tab/azcli)

```azurecli
az deployment group create \
    --name "exampledeployment" \
    --resource-group "examplegroup" \
    --template-uri "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/201-machine-learning-advanced/azuredeploy.json" \
    --parameters workspaceName="exampleworkspace" \
      location="eastus" \
      vnetOption="new" \
      vnetName="examplevnet" \
      storageAccountBehindVNet="true"
      privateEndpointType="AutoApproval"
```

# <a name="azure-powershell"></a>[Azure PowerShell](#tab/azpowershell)

```azurepowershell
New-AzResourceGroupDeployment `
  -Name "exampledeployment" `
  -ResourceGroupName "examplegroup" `
  -TemplateUri "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/201-machine-learning-advanced/azuredeploy.json" `
  -workspaceName "exampleworkspace" `
  -location "eastus" `
  -vnetOption "new" `
  -vnetName "examplevnet" `
  -storageAccountBehindVNet "true"
  -privateEndpointType "AutoApproval"
```

---

Como alternativa, puede implementar varios o todos los recursos dependientes detrás de una red virtual.

# <a name="azure-cli"></a>[CLI de Azure](#tab/azcli)

```azurecli
az deployment group create \
    --name "exampledeployment" \
    --resource-group "examplegroup" \
    --template-uri "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/201-machine-learning-advanced/azuredeploy.json" \
    --parameters workspaceName="exampleworkspace" \
      location="eastus" \
      vnetOption="new" \
      vnetName="examplevnet" \
      storageAccountBehindVNet="true" \
      keyVaultBehindVNet="true" \
      containerRegistryBehindVNet="true" \
      containerRegistryOption="new" \
      containerRegistrySku="Premium"
      privateEndpointType="AutoApproval"
```

# <a name="azure-powershell"></a>[Azure PowerShell](#tab/azpowershell)

```azurepowershell
New-AzResourceGroupDeployment `
  -Name "exampledeployment" `
  -ResourceGroupName "examplegroup" `
  -TemplateUri "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/201-machine-learning-advanced/azuredeploy.json" `
  -workspaceName "exampleworkspace" `
  -location "eastus" `
  -vnetOption "new" `
  -vnetName "examplevnet" `
  -storageAccountBehindVNet "true"
  -keyVaultBehindVNet "true" `
  -containerRegistryBehindVNet "true" `
  -containerRegistryOption "new" `
  -containerRegistrySku "Premium"
  -privateEndpointType "AutoApproval"
```

---

<!-- Workspaces need a private endpoint when associated resources are behind a virtual network to work properly. To set up a private endpoint for the workspace with a new virtual network:

> [!IMPORTANT]
> The deployment is only valid in regions which support private endpoints.

# [Azure CLI](#tab/azcli)

```azurecli
az deployment group create \
    --name "exampledeployment" \
    --resource-group "examplegroup" \
    --template-uri "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/201-machine-learning-advanced/azuredeploy.json" \
    --parameters workspaceName="exampleworkspace" \
      location="eastus" \
      vnetOption="new" \
      vnetName="examplevnet" \
      privateEndpointType="AutoApproval"
```

# [Azure PowerShell](#tab/azpowershell)

```azurepowershell
New-AzResourceGroupDeployment `
  -Name "exampledeployment" `
  -ResourceGroupName "examplegroup" `
  -TemplateUri "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/201-machine-learning-advanced/azuredeploy.json" `
  -workspaceName "exampleworkspace" `
  -location "eastus" `
  -vnetOption "new" `
  -vnetName "examplevnet" `
  -privateEndpointType "AutoApproval"
```

--- -->

### <a name="use-an-existing-virtual-network--resources"></a>Uso de una red virtual y recursos existentes

Para implementar un área de trabajo con recursos asociados existentes, debe establecer el parámetro **vnetOption** en **existing** junto con los parámetros de subred. Sin embargo, debe crear puntos de conexión de servicio en la red virtual para cada uno de los recursos **antes** de la implementación. Al igual que con las implementaciones de nueva red virtual, puede tener uno o todos los recursos detrás de una red virtual.

> [!IMPORTANT]
> La subred debe tener el punto de conexión de servicio `Microsoft.Storage`.

> [!IMPORTANT]
> Las subredes no permiten la creación de puntos de conexión privados. Deshabilite el punto de conexión privado para habilitar la subred.

1. Habilite los puntos de conexión de servicio para los recursos.

    # <a name="azure-cli"></a>[CLI de Azure](#tab/azcli)

    ```azurecli
    az network vnet subnet update --resource-group "examplegroup" --vnet-name "examplevnet" --name "examplesubnet" --service-endpoints "Microsoft.Storage"
    az network vnet subnet update --resource-group "examplegroup" --vnet-name "examplevnet" --name "examplesubnet" --service-endpoints "Microsoft.KeyVault"
    az network vnet subnet update --resource-group "examplegroup" --vnet-name "examplevnet" --name "examplesubnet" --service-endpoints "Microsoft.ContainerRegistry"
    ```

    # <a name="azure-powershell"></a>[Azure PowerShell](#tab/azpowershell)

    ```azurepowershell
    Get-AzVirtualNetwork -ResourceGroupName "examplegroup" -Name "examplevnet" | Set-AzVirtualNetworkSubnetConfig -Name "examplesubnet" -AddressPrefix "<subnet prefix>" -ServiceEndpoint "Microsoft.Storage" | Set-AzVirtualNetwork
    Get-AzVirtualNetwork -ResourceGroupName "examplegroup" -Name "examplevnet" | Set-AzVirtualNetworkSubnetConfig -Name "examplesubnet" -AddressPrefix "<subnet prefix>" -ServiceEndpoint "Microsoft.KeyVault" | Set-AzVirtualNetwork
    Get-AzVirtualNetwork -ResourceGroupName "examplegroup" -Name "examplevnet" | Set-AzVirtualNetworkSubnetConfig -Name "examplesubnet" -AddressPrefix "<subnet prefix>" -ServiceEndpoint "Microsoft.ContainerRegistry" | Set-AzVirtualNetwork
    ```

    ---

1. Implemente el área de trabajo.

    # <a name="azure-cli"></a>[CLI de Azure](#tab/azcli)

    ```azurecli
    az deployment group create \
    --name "exampledeployment" \
    --resource-group "examplegroup" \
    --template-uri "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/201-machine-learning-advanced/azuredeploy.json" \
    --parameters workspaceName="exampleworkspace" \
      location="eastus" \
      vnetOption="existing" \
      vnetName="examplevnet" \
      vnetResourceGroupName="examplegroup" \
      storageAccountBehindVNet="true" \
      keyVaultBehindVNet="true" \
      containerRegistryBehindVNet="true" \
      containerRegistryOption="new" \
      containerRegistrySku="Premium" \
      subnetName="examplesubnet" \
      subnetOption="existing"
      privateEndpointType="AutoApproval"
    ```

    # <a name="azure-powershell"></a>[Azure PowerShell](#tab/azpowershell)
    ```azurepowershell
    New-AzResourceGroupDeployment `
      -Name "exampledeployment" `
      -ResourceGroupName "examplegroup" `
      -TemplateUri "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/201-machine-learning-advanced/azuredeploy.json" `
      -workspaceName "exampleworkspace" `
      -location "eastus" `
      -vnetOption "existing" `
      -vnetName "examplevnet" `
      -vnetResourceGroupName "examplegroup" `
      -storageAccountBehindVNet "true"
      -keyVaultBehindVNet "true" `
      -containerRegistryBehindVNet "true" `
      -containerRegistryOption "new" `
      -containerRegistrySku "Premium" `
      -subnetName "examplesubnet" `
      -subnetOption "existing"
      -privateEndpointType "AutoApproval"
    ```
    ---

<!-- Workspaces need a private endpoint when associated resources are behind a virtual network to work properly. To set up a private endpoint for the workspace with an existing virtual network:

> [!IMPORTANT]
> The deployment is only valid in regions which support private endpoints.

# [Azure CLI](#tab/azcli)

```azurecli
az deployment group create \
    --name "exampledeployment" \
    --resource-group "examplegroup" \
    --template-uri "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/201-machine-learning-advanced/azuredeploy.json" \
    --parameters workspaceName="exampleworkspace" \
      location="eastus" \
      vnetOption="existing" \
      vnetName="examplevnet" \
      vnetResourceGroupName="rg" \
      privateEndpointType="AutoApproval" \
      subnetName="subnet" \
      subnetOption="existing"
```

# [Azure PowerShell](#tab/azpowershell)

```azurepowershell
New-AzResourceGroupDeployment `
  -Name "exampledeployment" `
  -ResourceGroupName "examplegroup" `
  -TemplateUri "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/201-machine-learning-advanced/azuredeploy.json" `
  -workspaceName "exampleworkspace" `
  -location "eastus" `
  -vnetOption "existing" `
  -vnetName "examplevnet" `
  -vnetResourceGroupName "rg"
  -privateEndpointType "AutoApproval"
  -subnetName "subnet"
  -subnetOption "existing"
```

--- -->

## <a name="use-the-azure-portal"></a>Uso de Azure Portal

1. Siga los pasos que se indican en [Implementación de recursos desde plantilla personalizada](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-template-deploy-portal#deploy-resources-from-custom-template). Cuando llegue a la pantalla __Seleccionar una plantilla__, elija la plantilla **201-machine-learning-advanced** en la lista desplegable.
1. Seleccione __Seleccionar plantilla__ para usar la plantilla. Proporcione la siguiente información necesaria y cualquier otro parámetro en función del escenario de implementación.

   * Suscripción: Seleccione la suscripción de Azure que va a usar para estos recursos.
   * Grupo de recursos: Seleccione o cree un grupo de recursos que contenga los servicios.
   * Región: Seleccione la región de Azure en la que se crearán los recursos.
   * Nombre del área de trabajo: El nombre que se usará para el área de trabajo de Azure Machine Learning que se va a crear. El nombre del área de trabajo debe tener entre 3 y 33 caracteres. Solo puede contener caracteres alfanuméricos y "-".
   * Ubicación: Seleccione la ubicación en la que se crearán los recursos.
1. Seleccione __Revisar + crear__.
1. En la pantalla __Revisar y crear__, acepte los términos y condiciones de la lista y seleccione __Crear__.

Para más información, vea [Implementación de recursos desde plantilla personalizada](../azure-resource-manager/templates/deploy-portal.md#deploy-resources-from-custom-template).

## <a name="troubleshooting"></a>Solución de problemas

### <a name="resource-provider-errors"></a>Errores del proveedor de recursos

[!INCLUDE [machine-learning-resource-provider](../../includes/machine-learning-resource-provider.md)]

### <a name="azure-key-vault-access-policy-and-azure-resource-manager-templates"></a>Directiva de acceso de Azure Key Vault y plantillas de Azure Resource Manager

Una plantilla de Azure Resource Manager se usa para crear el área de trabajo y los recursos asociados (incluido Azure Key Vault) varias veces. Por ejemplo, con el uso de la plantilla varias veces con los mismos parámetros como parte de una canalización de implementación e integración continuas.

La mayoría de las operaciones de creación de recursos mediante plantillas son idempotentes, pero Key Vault borra las directivas de acceso cada vez que se usa la plantilla. Al borrar las directivas de acceso, se interrumpe el acceso al almacén de claves para las áreas de trabajo existentes que lo estén usando. Por ejemplo, las funcionalidades de detección y creación de Azure Notebooks pueden producir un error.  

Para evitar este problema, se recomienda uno de los siguientes enfoques:

* No implemente la plantilla de más de una vez con los mismos parámetros. O bien, elimine los recursos existentes antes de usar la plantilla para volver a crearlos.

* Examine las directivas de acceso de Key Vault y, luego, use estas directivas para establecer la propiedad `accessPolicies` de la plantilla. Para ver las directivas de acceso, use el siguiente comando de CLI de Azure:

    ```azurecli
    az keyvault show --name mykeyvault --resource-group myresourcegroup --query properties.accessPolicies
    ```

    Para más información sobre el uso de la `accessPolicies` sección de la plantilla, consulte la referencia del objeto [AccessPolicyEntry](https://docs.microsoft.com/azure/templates/Microsoft.KeyVault/2018-02-14/vaults#AccessPolicyEntry).

* Compruebe si ya existe el recurso de Key Vault. Si es así, no lo vuelva a crear con la plantilla. Por ejemplo, para usar el Key Vault existente en lugar de crear uno nuevo, realice los siguientes cambios en la plantilla:

    * **Agregue** un parámetro que acepte el identificador de un recurso de Key Vault existente:

        ```json
        "keyVaultId":{
          "type": "string",
          "metadata": {
            "description": "Specify the existing Key Vault ID."
          }
        }
      ```

    * **Elimine** la sección que crea un recurso de Key Vault:

        ```json
        {
          "type": "Microsoft.KeyVault/vaults",
          "apiVersion": "2018-02-14",
          "name": "[variables('keyVaultName')]",
          "location": "[parameters('location')]",
          "properties": {
            "tenantId": "[variables('tenantId')]",
            "sku": {
              "name": "standard",
              "family": "A"
            },
            "accessPolicies": [
            ]
          }
        },
        ```

    * **Elimine** la línea de la `"[resourceId('Microsoft.KeyVault/vaults', variables('keyVaultName'))]",` sección `dependsOn` del área de trabajo. También **cambie** la `keyVault` entrada en la sección `properties` del área de trabajo para hacer referencia al `keyVaultId` parámetro:

        ```json
        {
          "type": "Microsoft.MachineLearningServices/workspaces",
          "apiVersion": "2019-11-01",
          "name": "[parameters('workspaceName')]",
          "location": "[parameters('location')]",
          "dependsOn": [
            "[resourceId('Microsoft.Storage/storageAccounts', variables('storageAccountName'))]",
            "[resourceId('Microsoft.Insights/components', variables('applicationInsightsName'))]"
          ],
          "identity": {
            "type": "systemAssigned"
          },
          "sku": {
            "tier": "[parameters('sku')]",
            "name": "[parameters('sku')]"
          },
          "properties": {
            "friendlyName": "[parameters('workspaceName')]",
            "keyVault": "[parameters('keyVaultId')]",
            "applicationInsights": "[resourceId('Microsoft.Insights/components',variables('applicationInsightsName'))]",
            "storageAccount": "[resourceId('Microsoft.Storage/storageAccounts/',variables('storageAccountName'))]"
          }
        }
        ```

    Después de estos cambios, puede especificar el identificador del recurso de Key Vault existente al ejecutar la plantilla. La plantilla establecerá entonces la propiedad `keyVault` del área de trabajo en su identificador para reutilizar el almacén de claves.

    Para obtener el identificador de la Key Vault, puede hacer referencia a la salida de la ejecución de la plantilla original o usar el CLI de Azure. El siguiente comando es un ejemplo de uso de la CLI de Azure para obtener el identificador de recurso de Key Vault:

    ```azurecli
    az keyvault show --name mykeyvault --resource-group myresourcegroup --query id
    ```

    Este comando devuelve un valor similar al siguiente texto:

    ```text
    /subscriptions/{subscription-guid}/resourceGroups/myresourcegroup/providers/Microsoft.KeyVault/vaults/mykeyvault
    ```

## <a name="next-steps"></a>Pasos siguientes

* [Implementación de recursos con las plantillas de Resource Manager y la API REST de Resource Manager](../azure-resource-manager/templates/deploy-rest.md).
* [Creación e implementación de grupos de recursos de Azure mediante Visual Studio](../azure-resource-manager/templates/create-visual-studio-deployment-project.md).
* [Para ver otras plantillas relacionadas con Azure Machine Learning, consulte el repositorio de plantillas de inicio rápido de Azure](https://github.com/Azure/azure-quickstart-templates)
