---
title: 'Tutorial: Implementación de una plantilla vinculada'
description: Aprenda a implementar una plantilla vinculada.
ms.date: 03/13/2020
ms.topic: tutorial
ms.author: jgao
ms.custom: devx-track-azurecli
ms.openlocfilehash: 63383f810b6f5643bad9feb86360745cd2f9b000
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87501140"
---
# <a name="tutorial-deploy-a-linked-template"></a>Tutorial: Implementación de una plantilla vinculada

En los [tutoriales anteriores](./deployment-tutorial-local-template.md), aprendió a implementar una plantilla que está almacenada en la máquina local. Para implementar soluciones complejas, puede dividir una plantilla en muchas plantillas y, a continuación, implementar estas mediante una plantilla principal. En este tutorial, aprenderá a implementar una plantilla principal que contiene la referencia a una plantilla vinculada. Cuando se implementa la plantilla principal, se desencadena la implementación de la plantilla vinculada. También aprenderá a almacenar y proteger la plantilla vinculada mediante el token de SAS. Su tiempo de realización es de unos **12 minutos**.

## <a name="prerequisites"></a>Prerrequisitos

Aunque no es obligatorio, se recomienda realizar el tutorial anterior.

## <a name="review-template"></a>Revisión de la plantilla

En los tutoriales anteriores, implementó una plantilla que creaba una cuenta de almacenamiento, un plan de App Service y una aplicación web. La plantilla que se utilizó fue:

:::code language="json" source="~/resourcemanager-templates/get-started-deployment/local-template/azuredeploy.json":::

## <a name="create-a-linked-template"></a>Creación de una plantilla vinculada

Puede separar el recurso de la cuenta de almacenamiento en una plantilla vinculada:

:::code language="json" source="~/resourcemanager-templates/get-started-deployment/linked-template/linkedStorageAccount.json":::

La siguiente plantilla es la plantilla principal.  El objeto resaltado **Microsoft.Resources/deployments** muestra cómo llamar a una plantilla vinculada. La plantilla vinculada no se puede almacenar como un archivo local o un archivo que solo está disponible en la red local. Solo se puede proporcionar un valor de URI que incluya *http* o *https*. Resource Manager debe tener acceso a la plantilla. Una opción es colocar la plantilla vinculada en una cuenta de almacenamiento y usar el URI para dicho elemento. El URI se pasa a la plantilla mediante un parámetro. Consulte la definición del parámetro resaltado.

:::code language="json" source="~/resourcemanager-templates/get-started-deployment/linked-template/azuredeploy.json" highlight="27-32,40-58":::

Guarde una copia de la plantilla principal en la máquina local con la extensión .json, por ejemplo, azuredeploy.json. No es necesario guardar una copia de la plantilla vinculada.  La plantilla vinculada se copiará de un repositorio de GitHub a una cuenta de almacenamiento.

## <a name="store-the-linked-template"></a>Almacenamiento de la plantilla vinculada

El siguiente script de PowerShell crea una cuenta de almacenamiento, crea un contenedor y copia la plantilla vinculada de un repositorio de GitHub en el contenedor. Se almacena una copia de la plantilla vinculada en [GitHub](https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/get-started-deployment/linked-template/linkedStorageAccount.json).

Seleccione **Probar** para abrir Cloud Shell, después, seleccione **Copiar** para copiar el script de PowerShell y haga clic con el botón derecho en el panel del shell para pegar el script:

> [!IMPORTANT]
> Los nombres de cuentas de almacenamiento deben tener entre 3 y 24 caracteres, y usar solo números y letras minúsculas. El nombre debe ser único. En la plantilla, el nombre de la cuenta de almacenamiento es el nombre del proyecto con "store" anexado y el nombre del proyecto debe tener entre 3 y 11 caracteres. Por lo tanto, el nombre del proyecto debe cumplir los requisitos de nombre para la cuenta de almacenamiento y tener menos de 11 caracteres.

```azurepowershell-interactive
$projectName = Read-Host -Prompt "Enter a project name:"   # This name is used to generate names for Azure resources, such as storage account name.
$location = Read-Host -Prompt "Enter a location (i.e. centralus)"

$resourceGroupName = $projectName + "rg"
$storageAccountName = $projectName + "store"
$containerName = "templates" # The name of the Blob container to be created.

$linkedTemplateURL = "https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/get-started-deployment/linked-template/linkedStorageAccount.json" # A completed linked template used in this tutorial.
$fileName = "linkedStorageAccount.json" # A file name used for downloading and uploading the linked template.

# Download the template
Invoke-WebRequest -Uri $linkedTemplateURL -OutFile "$home/$fileName"

# Create a resource group
New-AzResourceGroup -Name $resourceGroupName -Location $location

# Create a storage account
$storageAccount = New-AzStorageAccount `
    -ResourceGroupName $resourceGroupName `
    -Name $storageAccountName `
    -Location $location `
    -SkuName "Standard_LRS"

$context = $storageAccount.Context

# Create a container
New-AzStorageContainer -Name $containerName -Context $context -Permission Container

# Upload the template
Set-AzStorageBlobContent `
    -Container $containerName `
    -File "$home/$fileName" `
    -Blob $fileName `
    -Context $context

Write-Host "Press [ENTER] to continue ..."
```

## <a name="deploy-template"></a>Implementar plantilla

Para implementar una plantilla privada en una cuenta de almacenamiento, genere un token de SAS e inclúyalo en el identificador URI de la plantilla. Establezca el tiempo de expiración con un margen suficiente para completar la implementación. El blob que contiene la plantilla solo es accesible para el propietario de la cuenta. Sin embargo, cuando se crea un token de SAS para el blob, el blob es accesible para cualquier persona con ese URI. Si otro usuario intercepta el URI, ese usuario podrá tener acceso a la plantilla. Un token de SAS es una buena forma de limitar el acceso a las plantillas, pero no debe incluir datos confidenciales, como contraseñas, directamente en la plantilla.

Si no ha creado el grupo de recursos, consulte [Creación del grupo de recursos](./deployment-tutorial-local-template.md#create-resource-group).

> [!NOTE]
> En el siguiente código de la CLI de Azure, el parámetro de fecha -d sería un argumento no válido en macOS. Por tanto, para los usuarios de macOS, es necesario usar -v+2H para agregar 2 horas a la hora actual en el terminal de macOS.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell

$projectName = Read-Host -Prompt "Enter a project name:"   # This name is used to generate names for Azure resources, such as storage account name.
$templateFile = Read-Host -Prompt "Enter the main template file and path"

$resourceGroupName="${projectName}rg"
$storageAccountName="${projectName}store"
$containerName = "templates"
$fileName = "linkedStorageAccount.json" # A file name used for downloading and uploading the linked template.

$key = (Get-AzStorageAccountKey -ResourceGroupName $resourceGroupName -Name $storageAccountName).Value[0]
$context = New-AzStorageContext -StorageAccountName $storageAccountName -StorageAccountKey $key

# Generate a SAS token
$linkedTemplateUri = New-AzStorageBlobSASToken `
    -Context $context `
    -Container $containerName `
    -Blob $fileName `
    -Permission r `
    -ExpiryTime (Get-Date).AddHours(2.0) `
    -FullUri

# Deploy the template
New-AzResourceGroupDeployment `
  -Name DeployLinkedTemplate `
  -ResourceGroupName $resourceGroupName `
  -TemplateFile $templateFile `
  -projectName $projectName `
  -linkedTemplateUri $linkedTemplateUri `
  -verbose
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

```azurecli

echo "Enter a project name that is used to generate resource names:"
read projectName
echo "Enter the main template file:"
read templateFile

resourceGroupName="${projectName}rg"
storageAccountName="${projectName}store"
containerName="templates"
fileName="linkedStorageAccount.json"

key=$(az storage account keys list -g $resourceGroupName -n $storageAccountName --query [0].value -o tsv)

linkedTemplateUri=$(az storage blob generate-sas \
  --account-name $storageAccountName \
  --account-key $key \
  --container-name $containerName \
  --name $fileName \
  --permissions r \
  --expiry `date -u -d "120 minutes" '+%Y-%m-%dT%H:%MZ'` \
  --full-uri)

linkedTemplateUri=$(echo $linkedTemplateUri | sed 's/"//g')
az deployment group create \
  --name DeployLinkedTemplate \
  --resource-group $resourceGroupName \
  --template-file $templateFile \
  --parameters projectName=$projectName linkedTemplateUri=$linkedTemplateUri \
  --verbose
```

---

## <a name="clean-up-resources"></a>Limpieza de recursos

Limpie los recursos que implementó eliminando el grupo de recursos.

1. En Azure Portal, seleccione **Grupos de recursos** en el menú de la izquierda.
2. Escriba el nombre del grupo de recursos en el campo **Filtrar por nombre**.
3. Seleccione el nombre del grupo de recursos.
4. Seleccione **Eliminar grupo de recursos** del menú superior.

## <a name="next-steps"></a>Pasos siguientes

Ha aprendido a implementar una plantilla vinculada. En el siguiente tutorial, aprenderá a crear una canalización DevOp para implementar una plantilla.

> [!div class="nextstepaction"]
> [Crear una canalización](./deployment-tutorial-pipeline.md)
