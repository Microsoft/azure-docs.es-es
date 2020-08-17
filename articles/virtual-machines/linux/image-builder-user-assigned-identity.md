---
title: Creación de una imagen de máquina virtual y usar una identidad administrada asignada por el usuario para acceder a archivos en Azure Storage (versión preliminar)
description: Cree una imagen de máquina virtual mediante el generador de imágenes de Azure que pueda acceder a archivos almacenados en Azure Storage con la identidad administrada asignada por el usuario.
author: cynthn
ms.author: cynthn
ms.date: 05/02/2019
ms.topic: how-to
ms.service: virtual-machines-linux
ms.subservice: imaging
ms.openlocfilehash: 79349f9da45a623581c40276c8e69d490c1dd253
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87085561"
---
# <a name="create-an-image-and-use-a-user-assigned-managed-identity-to-access-files-in-azure-storage"></a>Creación de una imagen y usar una identidad administrada asignada por el usuario para acceder a archivos en Azure Storage 

El generador de imágenes de Azure admite el uso de scripts o la copia de archivos desde varias ubicaciones, como GitHub, Azure Storage, etc. Para utilizarlos, el generador de imágenes de Azure debe haber podido acceder externamente a ellos, pero puede proteger los blobs de Azure Storage mediante tokens de SAS.

En este artículo se muestra cómo crear una imagen personalizada mediante Azure VM Image Builder, donde el servicio usará una [identidad administrada asignada por el usuario](../../active-directory/managed-identities-azure-resources/overview.md) para acceder a los archivos de Azure Storage para la personalización de la imagen, sin tener que dar acceso público a los archivos ni configurar tokens de SAS.

En el ejemplo siguiente, creará dos grupos de recursos: uno se utilizará para la imagen personalizada y el otro hospedará una cuenta de Azure Storage, que contiene un archivo de script. Esto simula un escenario real, en que puede tener los artefactos de compilación o los archivos de imagen en diferentes cuentas de almacenamiento, al margen del generador de imágenes. Creará una identidad asignada por el usuario y, a continuación, concederá permisos de lectura en el archivo de script, pero no establecerá ningún acceso público a ese archivo. Después, usará el personalizador de shell para descargar y ejecutar ese script desde la cuenta de almacenamiento.


> [!IMPORTANT]
> Actualmente, el generador de imágenes de Azure se encuentra en versión preliminar pública.
> Esta versión preliminar se ofrece sin Acuerdo de Nivel de Servicio y no se recomienda para cargas de trabajo de producción. Es posible que algunas características no sean compatibles o que tengan sus funcionalidades limitadas. Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="register-the-features"></a>Registro de las características
Para usar el generador de imágenes de Azure durante la versión preliminar, tendrá que registrar la nueva característica.

```azurecli-interactive
az feature register --namespace Microsoft.VirtualMachineImages --name VirtualMachineTemplatePreview
```

Compruebe el estado del registro de la característica.

```azurecli-interactive
az feature show --namespace Microsoft.VirtualMachineImages --name VirtualMachineTemplatePreview | grep state
```

Compruebe el registro.


```azurecli-interactive
az provider show -n Microsoft.VirtualMachineImages | grep registrationState
az provider show -n Microsoft.KeyVault | grep registrationState
az provider show -n Microsoft.Compute | grep registrationState
az provider show -n Microsoft.Storage | grep registrationState
```

Si no se muestran como registradas, ejecute lo siguiente:

```azurecli-interactive
az provider register -n Microsoft.VirtualMachineImages
az provider register -n Microsoft.Compute
az provider register -n Microsoft.KeyVault
az provider register -n Microsoft.Storage
```


## <a name="create-a-resource-group"></a>Crear un grupo de recursos

Usaremos algunos datos de forma repetida, por lo que crearemos diversas variables para almacenar esa información.


```console
# Image resource group name 
imageResourceGroup=aibmdimsi
# storage resource group
strResourceGroup=aibmdimsistor
# Location 
location=WestUS2
# name of the image to be created
imageName=aibCustLinuxImgMsi01
# image distribution metadata reference name
runOutputName=u1804ManImgMsiro
```

Cree una variable para el id. de suscripción. Puede obtenerlo mediante `az account show | grep id`.

```console
subscriptionID=<Your subscription ID>
```

Cree los grupos de recursos para la imagen y el almacenamiento del script.

```console
# create resource group for image template
az group create -n $imageResourceGroup -l $location
# create resource group for the script storage
az group create -n $strResourceGroup -l $location
```

Cree una identidad asignada por el usuario y establezca permisos en el grupo de recursos.

Image Builder usará la [identidad de usuario](../../active-directory/managed-identities-azure-resources/qs-configure-cli-windows-vm.md#user-assigned-managed-identity) proporcionada para insertar la imagen en el grupo de recursos. En este ejemplo, se creará una definición de roles de Azure que tiene las acciones granulares necesarias para realizar la distribución de la imagen. La definición de roles se asignará a la identidad del usuario.

```console
# create user assigned identity for image builder to access the storage account where the script is located
idenityName=aibBuiUserId$(date +'%s')
az identity create -g $imageResourceGroup -n $idenityName

# get identity id
imgBuilderCliId=$(az identity show -g $imageResourceGroup -n $idenityName | grep "clientId" | cut -c16- | tr -d '",')

# get the user identity URI, needed for the template
imgBuilderId=/subscriptions/$subscriptionID/resourcegroups/$imageResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/$idenityName

# download preconfigured role definition example
curl https://raw.githubusercontent.com/danielsollondon/azvmimagebuilder/master/solutions/12_Creating_AIB_Security_Roles/aibRoleImageCreation.json -o aibRoleImageCreation.json

# update the definition
sed -i -e "s/<subscriptionID>/$subscriptionID/g" aibRoleImageCreation.json
sed -i -e "s/<rgName>/$imageResourceGroup/g" aibRoleImageCreation.json

# create role definitions
az role definition create --role-definition ./aibRoleImageCreation.json

# grant role definition to the user assigned identity
az role assignment create \
    --assignee $imgBuilderCliId \
    --role "Azure Image Builder Service Image Creation Role" \
    --scope /subscriptions/$subscriptionID/resourceGroups/$imageResourceGroup
```

Cree el almacenamiento y copie ahí el script de ejemplo de GitHub.

```azurecli-interactive
# script storage account
scriptStorageAcc=aibstorscript$(date +'%s')

# script container
scriptStorageAccContainer=scriptscont$(date +'%s')

# script url
scriptUrl=https://$scriptStorageAcc.blob.core.windows.net/$scriptStorageAccContainer/customizeScript.sh

# create storage account and blob in resource group
az storage account create -n $scriptStorageAcc -g $strResourceGroup -l $location --sku Standard_LRS

az storage container create -n $scriptStorageAccContainer --fail-on-exist --account-name $scriptStorageAcc

# copy in an example script from the GitHub repo 
az storage blob copy start \
    --destination-blob customizeScript.sh \
    --destination-container $scriptStorageAccContainer \
    --account-name $scriptStorageAcc \
    --source-uri https://raw.githubusercontent.com/danielsollondon/azvmimagebuilder/master/quickquickstarts/customizeScript.sh
```

Conceda al generador de imágenes permiso para crear recursos en el grupo de recursos de imagen. El valor `--assignee` es el identificador de identidad del usuario.

```azurecli-interactive
az role assignment create \
    --assignee $imgBuilderCliId \
    --role "Storage Blob Data Reader" \
    --scope /subscriptions/$subscriptionID/resourceGroups/$strResourceGroup/providers/Microsoft.Storage/storageAccounts/$scriptStorageAcc/blobServices/default/containers/$scriptStorageAccContainer 
```




## <a name="modify-the-example"></a>Modificación del ejemplo

Descargue el archivo .json de ejemplo y configúrelo con las variables que ha creado.

```console
curl https://raw.githubusercontent.com/danielsollondon/azvmimagebuilder/master/quickquickstarts/7_Creating_Custom_Image_using_MSI_to_Access_Storage/helloImageTemplateMsi.json -o helloImageTemplateMsi.json
sed -i -e "s/<subscriptionID>/$subscriptionID/g" helloImageTemplateMsi.json
sed -i -e "s/<rgName>/$imageResourceGroup/g" helloImageTemplateMsi.json
sed -i -e "s/<region>/$location/g" helloImageTemplateMsi.json
sed -i -e "s/<imageName>/$imageName/g" helloImageTemplateMsi.json
sed -i -e "s%<scriptUrl>%$scriptUrl%g" helloImageTemplateMsi.json
sed -i -e "s%<imgBuilderId>%$imgBuilderId%g" helloImageTemplateMsi.json
sed -i -e "s%<runOutputName>%$runOutputName%g" helloImageTemplateMsi.json
```

## <a name="create-the-image"></a>Crear la imagen

Envíe la configuración de la imagen al servicio del generador de imágenes de Azure.

```azurecli-interactive
az resource create \
    --resource-group $imageResourceGroup \
    --properties @helloImageTemplateMsi.json \
    --is-full-object \
    --resource-type Microsoft.VirtualMachineImages/imageTemplates \
    -n helloImageTemplateMsi01
```

Inicie la generación de imágenes.

```azurecli-interactive
az resource invoke-action \
     --resource-group $imageResourceGroup \
     --resource-type  Microsoft.VirtualMachineImages/imageTemplates \
     -n helloImageTemplateMsi01 \
     --action Run 
```

Espere a que la compilación finalice. Puede tardar unos 15 minutos.

## <a name="create-a-vm"></a>Crear una VM

Cree una máquina virtual a partir de la imagen. 

```azurecli
az vm create \
  --resource-group $imageResourceGroup \
  --name aibImgVm00 \
  --admin-username aibuser \
  --image $imageName \
  --location $location \
  --generate-ssh-keys
```

Una vez creada la máquina virtual, inicie una sesión de SSH con la máquina virtual.

```console
ssh aibuser@<publicIp>
```

Verá que la imagen se ha personalizado con un mensaje del día en cuanto se ha establecido la conexión SSH.

```output

*******************************************************
**            This VM was built from the:            **
**      !! AZURE VM IMAGE BUILDER Custom Image !!    **
**         You have just been Customized :-)         **
*******************************************************
```

## <a name="clean-up"></a>Limpieza

Cuando haya terminado, puede eliminar los recursos si ya no son necesarios.

```azurecli-interactive

az role definition delete --name "$imageRoleDefName"
```azurecli-interactive
az role assignment delete \
    --assignee $imgBuilderCliId \
    --role "$imageRoleDefName" \
    --scope /subscriptions/$subscriptionID/resourceGroups/$imageResourceGroup
az identity delete --ids $imgBuilderId
az resource delete \
    --resource-group $imageResourceGroup \
    --resource-type Microsoft.VirtualMachineImages/imageTemplates \
    -n helloImageTemplateMsi01
az group delete -n $imageResourceGroup
az group delete -n $strResourceGroup
```

## <a name="next-steps"></a>Pasos siguientes

Si tiene algún problema para trabajar con el generador de imágenes de Azure, consulte [Solucionar problemas](https://github.com/danielsollondon/azvmimagebuilder/blob/master/troubleshootingaib.md?toc=%2fazure%2fvirtual-machines%context%2ftoc.json).
