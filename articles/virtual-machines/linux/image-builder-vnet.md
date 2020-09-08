---
title: Uso de Azure Image Builder para máquinas virtuales Linux que permiten acceder a una red virtual de Azure existente (versión preliminar)
description: Creación de imágenes de máquina virtual Linux con Azure Image Builder que permiten acceder a una red virtual de Azure existente
author: danielsollondon
ms.author: danis
ms.date: 08/10/2020
ms.topic: how-to
ms.service: virtual-machines-linux
ms.subservice: imaging
ms.reviewer: danis
ms.openlocfilehash: f216b6fa3a0e43c1c0313baa4f8414546a74d8f0
ms.sourcegitcommit: d8b8768d62672e9c287a04f2578383d0eb857950
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/11/2020
ms.locfileid: "88068033"
---
# <a name="use-azure-image-builder-for-linux-vms-allowing-access-to-an-existing-azure-vnet"></a>Uso de Azure Image Builder para máquinas virtuales Linux que permiten acceder a una red virtual de Azure existente

En este artículo se muestra cómo puede usar Azure Image Builder para crear una imagen de Linux personalizada básica que tenga acceso a los recursos existentes en una red virtual. La máquina virtual de creación que cree se implementará en una red virtual nueva o existente que especifique en la suscripción. Cuando se usa una red virtual de Azure existente, el servicio Azure Image Builder no requiere conectividad de red pública.

> [!IMPORTANT]
> Actualmente, el generador de imágenes de Azure se encuentra en versión preliminar pública.
> Esta versión preliminar se ofrece sin Acuerdo de Nivel de Servicio y no se recomienda para cargas de trabajo de producción. Es posible que algunas características no sean compatibles o que tengan sus funcionalidades limitadas. Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

## <a name="register-the-features"></a>Registro de las características

En primer lugar, debe registrarse para el servicio Azure Image Builder. El registro concede al servicio permiso para crear, administrar y eliminar un grupo de recursos de almacenamiento provisional. El servicio también tiene derechos para agregar recursos al grupo que se requieren para la creación de la imagen.

```azurecli-interactive
az feature register --namespace Microsoft.VirtualMachineImages --name VirtualMachineTemplatePreview
```

## <a name="set-variables-and-permissions"></a>Establecimiento de variables y permisos 

Usará algunas partes de la información de forma repetida. Cree algunas variables para almacenar esa información.

```azurecli-interactive
# set your environment variables here!!!!

# destination image resource group
imageResourceGroup=aibImageRG01

# location (see possible locations in main docs)
location=WestUS2

# your subscription
# get the current subID : 'az account show | grep id'
subscriptionID=$(az account show | grep id | tr -d '",' | cut -c7-)

# name of the image to be created
imageName=aibCustomLinuxImg01

# image distribution metadata reference name
runOutputName=aibCustLinManImg01ro


# VNET properties (update to match your existing VNET, or leave as-is for demo)
# VNET name
vnetName=myexistingvnet01
# subnet name
subnetName=subnet01
# VNET resource group name
# NOTE! The VNET must always be in the same region as the AIB service region.
vnetRgName=existingVnetRG
# Existing Subnet NSG Name or the demo will create it
nsgName=aibdemoNsg
```

Cree el grupo de recursos.

```azurecli-interactive
az group create -n $imageResourceGroup -l $location
```

## <a name="configure-networking"></a>Configuración de las redes

Si no tiene una red virtual, una subred o un grupo de seguridad de red existente, use el siguiente script para crear uno.

```bash

# Create a resource group

az group create -n $vnetRgName -l $location

# Create VNET

az network vnet create \
    --resource-group $vnetRgName \
    --name $vnetName --address-prefix 10.0.0.0/16 \
    --subnet-name $subnetName --subnet-prefix 10.0.0.0/24

# Create base NSG to simulate an existing NSG

az network nsg create -g $vnetRgName -n $nsgName

az network vnet subnet update \
    --resource-group $vnetRgName \
    --vnet-name $vnetName \
    --name $subnetName \
    --network-security-group $nsgName
    
#  NOTE! The VNET must always be in the same region as the Azure Image Builder service region.
```

### <a name="add-network-security-group-rule"></a>Incorporación de una regla de grupo de seguridad de red

Esta regla permite la conectividad desde el equilibrador de carga de Azure Image Builder con la máquina virtual de proxy. El puerto 60001 es para los sistemas operativos Linux y el puerto 60000 es para los sistemas operativos Windows. La máquina virtual de proxy se conecta a la máquina virtual de creación mediante el puerto 22 para los sistemas operativos Linux o el puerto 5986 para los sistemas operativos Windows.

```azurecli-interactive
az network nsg rule create \
    --resource-group $vnetRgName \
    --nsg-name $nsgName \
    -n AzureImageBuilderNsgRule \
    --priority 400 \
    --source-address-prefixes AzureLoadBalancer \
    --destination-address-prefixes VirtualNetwork \
    --destination-port-ranges 60000-60001 --direction inbound \
    --access Allow --protocol Tcp \
    --description "Allow Image Builder Private Link Access to Proxy VM"
```

### <a name="disable-private-service-policy-on-subnet"></a>Deshabilitación de la directiva de servicio privada en la subred

```azurecli-interactive
az network vnet subnet update \
  --name $subnetName \
  --resource-group $vnetRgName \
  --vnet-name $vnetName \
  --disable-private-link-service-network-policies true 
```

Para más información sobre la conexión en red de Image Builder, consulte [Opciones de conexión en red del servicio Azure Image Builder](image-builder-networking.md).

## <a name="modify-the-example-template-and-create-role"></a>Modificación de la plantilla de ejemplo y creación de un rol

```bash
# download the example and configure it with your vars

curl https://raw.githubusercontent.com/danielsollondon/azvmimagebuilder/master/quickquickstarts/1a_Creating_a_Custom_Linux_Image_on_Existing_VNET/existingVNETLinux.json -o existingVNETLinux.json
curl https://raw.githubusercontent.com/danielsollondon/azvmimagebuilder/master/solutions/12_Creating_AIB_Security_Roles/aibRoleNetworking.json -o aibRoleNetworking.json
curl https://raw.githubusercontent.com/danielsollondon/azvmimagebuilder/master/solutions/12_Creating_AIB_Security_Roles/aibRoleImageCreation.json -o aibRoleImageCreation.json

sed -i -e "s/<subscriptionID>/$subscriptionID/g" existingVNETLinux.json
sed -i -e "s/<rgName>/$imageResourceGroup/g" existingVNETLinux.json
sed -i -e "s/<region>/$location/g" existingVNETLinux.json
sed -i -e "s/<imageName>/$imageName/g" existingVNETLinux.json
sed -i -e "s/<runOutputName>/$runOutputName/g" existingVNETLinux.json

sed -i -e "s/<vnetName>/$vnetName/g" existingVNETLinux.json
sed -i -e "s/<subnetName>/$subnetName/g" existingVNETLinux.json
sed -i -e "s/<vnetRgName>/$vnetRgName/g" existingVNETLinux.json

sed -i -e "s/<subscriptionID>/$subscriptionID/g" aibRoleImageCreation.json
sed -i -e "s/<rgName>/$imageResourceGroup/g" aibRoleImageCreation.json

sed -i -e "s/<subscriptionID>/$subscriptionID/g" aibRoleNetworking.json
sed -i -e "s/<vnetRgName>/$vnetRgName/g" aibRoleNetworking.json

```

## <a name="set-permissions-on-the-resource-group"></a>Establecer permisos en el grupo de recursos

Image Builder usará la [identidad de usuario](https://docs.microsoft.com/azure/active-directory/managed-identities-azure-resources/qs-configure-cli-windows-vm#user-assigned-managed-identity) proporcionada para insertar la imagen en la instancia de Azure Shared Image Gallery (SIG). En este ejemplo, se creará una definición de roles de Azure que tiene las acciones granulares necesarias para realizar la distribución de la imagen a la instancia de SIG. La definición de roles se asignará a la identidad del usuario.

```bash
# create user assigned identity for image builder
idenityName=aibBuiUserId$(date +'%s')
az identity create -g $imageResourceGroup -n $idenityName

# get identity id
imgBuilderCliId=$(az identity show -g $imageResourceGroup -n $idenityName | grep "clientId" | cut -c16- | tr -d '",')

# get the user identity URI, needed for the template
imgBuilderId=/subscriptions/$subscriptionID/resourcegroups/$imageResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/$idenityName

# update the template
sed -i -e "s%<imgBuilderId>%$imgBuilderId%g" existingVNETLinux.json

# make role name unique, to avoid clashes in the same Azure Active Directory domain
imageRoleDefName="Azure Image Builder Image Def"$(date +'%s')
netRoleDefName="Azure Image Builder Network Def"$(date +'%s')

# update the definitions
sed -i -e "s/Azure Image Builder Service Image Creation Role/$imageRoleDefName/g" aibRoleImageCreation.json
sed -i -e "s/Azure Image Builder Service Networking Role/$netRoleDefName/g" aibRoleNetworking.json
```

En lugar de conceder a Image Builder una granularidad más baja y un mayor privilegio, puede crear dos roles. Uno proporciona permisos de creador para crear una imagen y el otro permite conectar la máquina virtual de creación y el equilibrador de carga a la red virtual.

```bash
# create role definitions
az role definition create --role-definition ./aibRoleImageCreation.json
az role definition create --role-definition ./aibRoleNetworking.json

# grant role definition to the user assigned identity
az role assignment create \
    --assignee $imgBuilderCliId \
    --role $imageRoleDefName \
    --scope /subscriptions/$subscriptionID/resourceGroups/$imageResourceGroup

az role assignment create \
    --assignee $imgBuilderCliId \
    --role $netRoleDefName \
    --scope /subscriptions/$subscriptionID/resourceGroups/$vnetRgName
```

Para más información sobre los permisos, consulte [Configuración de los permisos del Azure Image Builder la mediante CLI de Azure](image-builder-permissions-cli.md) o [Configuración de los permisos del servicio Azure Image Builder mediante PowerShell](image-builder-permissions-powershell.md).

## <a name="create-the-image"></a>Crear la imagen

Envíe la configuración de la imagen al servicio del generador de imágenes de Azure.

```azurecli-interactive
az resource create \
    --resource-group $imageResourceGroup \
    --properties @existingVNETLinux.json \
    --is-full-object \
    --resource-type Microsoft.VirtualMachineImages/imageTemplates \
    -n existingVNETLinuxTemplate01

# Wait approximately 1-3 mins (validation, permissions etc.)
```

Inicie la generación de imágenes.

```azurecli-interactive
az resource invoke-action \
     --resource-group $imageResourceGroup \
     --resource-type  Microsoft.VirtualMachineImages/imageTemplates \
     -n existingVNETLinuxTemplate01 \
     --action Run 

# Wait approximately 15 mins
```

La creación de la imagen y su replicación en las dos regiones puede llevar un tiempo. Espere a que termine esta parte antes de pasar a la creación de una máquina virtual.


## <a name="create-the-vm"></a>Creación de la máquina virtual

Cree una máquina virtual a partir la versión de la imagen creada por Azure Image Builder.

```azurecli-interactive
az vm create \
  --resource-group $imageResourceGroup \
  --name aibImgVm0001 \
  --admin-username aibuser \
  --image $imageName \
  --location $location \
  --generate-ssh-keys
```

Conéctese mediante SSH a la máquina virtual.

```bash
ssh aibuser@<publicIpAddress>
```

Debería ver que la imagen se ha personalizado con un *mensaje del día* en cuanto se establece la conexión SSH.

```console
*******************************************************
**            This VM was built from the:            **
**      !! AZURE VM IMAGE BUILDER Custom Image !!    **
**         You have just been Customized :-)         **
*******************************************************
```

## <a name="clean-up-resources"></a>Limpieza de recursos

Si ahora quiere volver a personalizar la versión de la imagen para crear una nueva de la misma imagen, omita los siguientes pasos y vaya a [Uso de Azure Image Builder para crear otra versión de la imagen](image-builder-gallery-update-image-version.md).


Lo siguiente elimina la imagen que se ha creado, junto con todos los demás archivos de recursos. Asegúrese de que haya terminado con esta implementación antes de eliminar los recursos.

Al eliminar los recursos de la galería de imágenes, tendrá que eliminar todas las versiones de la imagen antes de poder eliminar la definición de la imagen que se ha usado para crearlas. Para eliminar una galería, primero tiene que haber eliminado todas las definiciones de imagen de la galería.

Elimine la plantilla de Azure Image Builder.

```azurecli
az resource delete \
    --resource-group $imageResourceGroup \
    --resource-type Microsoft.VirtualMachineImages/imageTemplates \
    -n existingVNETLinuxTemplate01
```

Elimine las asignaciones de permisos, los roles y la identidad.
```azurecli-interactive
az role assignment delete \
    --assignee $imgBuilderCliId \
    --role $imageRoleDefName \
    --scope /subscriptions/$subscriptionID/resourceGroups/$imageResourceGroup

az role assignment delete \
    --assignee $imgBuilderCliId \
    --role $netRoleDefName \
    --scope /subscriptions/$subscriptionID/resourceGroups/$vnetRgName


az role definition delete --name "$imageRoleDefName"
az role definition delete --name "$netRoleDefName"

az identity delete --ids $imgBuilderId
```

Elimine el grupo de recursos.

```azurecli-interactive
az group delete -n $imageResourceGroup
```

Si ha creado una red virtual para este inicio rápido, puede eliminarla red virtual si ya no se usa.

## <a name="next-steps"></a>Pasos siguientes

Obtenga más información sobre las [galerías de imágenes compartidas de Azure](shared-image-galleries.md).
