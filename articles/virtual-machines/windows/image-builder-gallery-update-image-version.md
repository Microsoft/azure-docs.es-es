---
title: Creación de una versión de imagen a partir de otra ya existente con Azure Image Builder (versión preliminar)
description: Cree una versión de una imagen de una máquina virtual a partir de otra existente mediante Azure Image Builder.
author: cynthn
ms.author: cynthn
ms.date: 05/05/2020
ms.topic: how-to
ms.service: virtual-machines-windows
ms.openlocfilehash: bdae608d573bc411242b4bf66ea033a7df421cfc
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87005708"
---
# <a name="preview-create-a-new-vm-image-version-from-an-existing-image-version-using-azure-image-builder-in-windows"></a>Vista previa: Creación de una versión de una imagen de una máquina virtual a partir de otra existente mediante Azure Image Builder en Windows

En este artículo se muestra cómo tomar una versión de imagen existente en una [galería de imágenes compartidas](shared-image-galleries.md), actualizarla y publicarla como una nueva versión de imagen en la galería.

Se usará una plantilla .json de ejemplo para configurar la imagen. El archivo .json que se usa aquí es: [helloImageTemplateforSIGfromWinSIG.json](https://raw.githubusercontent.com/danielsollondon/azvmimagebuilder/master/quickquickstarts/2_Creating_a_Custom_Win_Shared_Image_Gallery_Image_from_SIG/helloImageTemplateforSIGfromWinSIG.json). 

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


## <a name="set-variables-and-permissions"></a>Establecimiento de variables y permisos

Si ha usado [Crear una imagen y distribuirla a una galería de imágenes compartidas](image-builder-gallery.md) para crear la galería de imágenes compartidas, ya se han creado las variables necesarias. Si no es así, configure algunas variables para usarlas en este ejemplo.

Para la versión preliminar, el generador de imágenes solo admitirá la creación de imágenes personalizadas en el mismo grupo de recursos que la imagen administrada de origen. Actualice el nombre del grupo de recursos de este ejemplo para que coincida con el de la imagen administrada de origen.

```azurecli-interactive
# Resource group name - we are using ibsigRG in this example
sigResourceGroup=myIBWinRG
# Datacenter location - we are using West US 2 in this example
location=westus
# Additional region to replicate the image to - we are using East US in this example
additionalregion=eastus
# name of the shared image gallery - in this example we are using myGallery
sigName=my22stSIG
# name of the image definition to be created - in this example we are using myImageDef
imageDefName=winSvrimages
# image distribution metadata reference name
runOutputName=w2019SigRo
# User name and password for the VM
username="user name for the VM"
vmpassword="password for the VM"
```

Cree una variable para el id. de suscripción. Puede obtenerlo mediante `az account show | grep id`.

```azurecli-interactive
subscriptionID=<Subscription ID>
```

Obtenga la versión de la imagen que quiera actualizar.

```azurecli-interactive
sigDefImgVersionId=$(az sig image-version list \
   -g $sigResourceGroup \
   --gallery-name $sigName \
   --gallery-image-definition $imageDefName \
   --subscription $subscriptionID --query [].'id' -o json | grep 0. | tr -d '"' | tr -d '[:space:]')
```

## <a name="create-a-user-assigned-identity-and-set-permissions-on-the-resource-group"></a>Creación de una identidad asignada por el usuario y establecimiento de los permisos en el grupo de recursos
Como ha establecido la identidad del usuario en el ejemplo anterior, solo tiene que obtener el identificador de recurso correspondiente, que se anexará a la plantilla.

```azurecli-interactive
#get identity used previously
imgBuilderId=$(az identity list -g $sigResourceGroup --query "[?contains(name, 'aibBuiUserId')].id" -o tsv)
```

Si ya tiene una galería de imágenes compartidas propia y no ha seguido el ejemplo anterior, tendrá que asignar permisos a Image Builder para acceder al grupo de recursos, para que pueda acceder a la galería. Revise los pasos del ejemplo [Crear una imagen y distribuirla en Shared Image Gallery](image-builder-gallery.md).


## <a name="modify-helloimage-example"></a>Modificación del ejemplo helloImage
Para revisar el ejemplo que se va a usar, abra el archivo .json aquí: [helloImageTemplateforSIGfromSIG.json](https://raw.githubusercontent.com/danielsollondon/azvmimagebuilder/master/quickquickstarts/2_Creating_a_Custom_Linux_Shared_Image_Gallery_Image_from_SIG/helloImageTemplateforSIGfromSIG.json) junto con la [referencia de plantillas de Image Builder](../linux/image-builder-json.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json). 


Descargue el ejemplo de .json y configúrelo con las variables. 

```azurecli-interactive
curl https://raw.githubusercontent.com/danielsollondon/azvmimagebuilder/master/quickquickstarts/8_Creating_a_Custom_Win_Shared_Image_Gallery_Image_from_SIG/helloImageTemplateforSIGfromWinSIG.json -o helloImageTemplateforSIGfromWinSIG.json
sed -i -e "s/<subscriptionID>/$subscriptionID/g" helloImageTemplateforSIGfromWinSIG.json
sed -i -e "s/<rgName>/$sigResourceGroup/g" helloImageTemplateforSIGfromWinSIG.json
sed -i -e "s/<imageDefName>/$imageDefName/g" helloImageTemplateforSIGfromWinSIG.json
sed -i -e "s/<sharedImageGalName>/$sigName/g" helloImageTemplateforSIGfromWinSIG.json
sed -i -e "s%<sigDefImgVersionId>%$sigDefImgVersionId%g" helloImageTemplateforSIGfromWinSIG.json
sed -i -e "s/<region1>/$location/g" helloImageTemplateforSIGfromWinSIG.json
sed -i -e "s/<region2>/$additionalregion/g" helloImageTemplateforSIGfromWinSIG.json
sed -i -e "s/<runOutputName>/$runOutputName/g" helloImageTemplateforSIGfromWinSIG.json
sed -i -e "s%<imgBuilderId>%$imgBuilderId%g" helloImageTemplateforSIGfromWinSIG.json
```

## <a name="create-the-image"></a>Crear la imagen

Envíe la configuración de la imagen al servicio de generador de imágenes de máquina virtual.

```azurecli-interactive
az resource create \
    --resource-group $sigResourceGroup \
    --properties @helloImageTemplateforSIGfromWinSIG.json \
    --is-full-object \
    --resource-type Microsoft.VirtualMachineImages/imageTemplates \
    -n imageTemplateforSIGfromWinSIG01
```

Inicie la generación de imágenes.

```azurecli-interactive
az resource invoke-action \
     --resource-group $sigResourceGroup \
     --resource-type  Microsoft.VirtualMachineImages/imageTemplates \
     -n imageTemplateforSIGfromWinSIG01 \
     --action Run 
```

Espere hasta que se haya creado la imagen y la replicación antes de continuar con el paso siguiente.


## <a name="create-the-vm"></a>Creación de la máquina virtual

```azurecli-interactive
az vm create \
  --resource-group $sigResourceGroup \
  --name aibImgWinVm002 \
  --admin-username $username \
  --admin-password $vmpassword \
  --image "/subscriptions/$subscriptionID/resourceGroups/$sigResourceGroup/providers/Microsoft.Compute/galleries/$sigName/images/$imageDefName/versions/latest" \
  --location $location
```

## <a name="verify-the-customization"></a>Comprobación de la personalización
Cree una conexión de Escritorio remoto a la máquina virtual con el nombre de usuario y la contraseña que ha establecido al crear la máquina virtual. Dentro de la máquina virtual, abra un símbolo del sistema y escriba lo siguiente:

```console
dir c:\
```

Ahora debería ver dos directorios:
- `buildActions` que se ha creado en la primera versión de imagen.
- `buildActions2` que se ha creado como parte de la actualización de la primera versión de imagen para crear la segunda.


## <a name="next-steps"></a>Pasos siguientes

Para obtener más información sobre los componentes del archivo .json que se usan en este artículo, vea [Referencia de la plantilla de generador de imágenes](../linux/image-builder-json.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json).
