---
title: 'Tutorial: Creación de imágenes de máquina virtual personalizadas con la CLI de Azure'
description: En este tutorial, aprenderá a usar la CLI de Azure para crear una imagen de máquina virtual personalizada en Azure.
author: cynthn
ms.service: virtual-machines-linux
ms.subservice: imaging
ms.topic: tutorial
ms.workload: infrastructure
ms.date: 05/04/2020
ms.author: cynthn
ms.custom: mvc, devx-track-azurecli
ms.reviewer: akjosh
ms.openlocfilehash: 7fff8539f330c7cbc886577eaf6dbe4417411999
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87498147"
---
# <a name="tutorial-create-a-custom-image-of-an-azure-vm-with-the-azure-cli"></a>Tutorial: Creación de una imagen personalizada de una máquina virtual de Azure con la CLI de Azure

Las imágenes personalizadas son como las imágenes de Marketplace, pero las puede crear usted mismo. Las imágenes personalizadas pueden usarse para configuraciones de arranque como la carga previa de aplicaciones, configuraciones de aplicaciones y otras configuraciones del sistema operativo. En este tutorial, creará su propia imagen personalizada de una máquina virtual de Azure. Aprenderá a:

> [!div class="checklist"]
> * Creación de una instancia de Shared Image Gallery
> * Creación de la definición de una imagen
> * Creación de la versión de una imagen
> * Crear una máquina virtual a partir de una imagen 
> * Compartir una galería de imágenes


En este tutorial se usa la CLI dentro de [Azure Cloud Shell](../../cloud-shell/overview.md), que se actualiza constantemente a la versión más reciente. Para abrir Cloud Shell, seleccione **Pruébelo** en la esquina superior de cualquier bloque de código.

Si decide instalar y usar la CLI de forma local, en este tutorial es preciso que ejecute la CLI de Azure de la versión 2.4.0, u otra posterior. Ejecute `az --version` para encontrar la versión. Si necesita instalarla o actualizarla, vea [Instalación de la CLI de Azure]( /cli/azure/install-azure-cli).

## <a name="overview"></a>Información general

Una [galería de imágenes compartidas](shared-image-galleries.md) simplifica el uso compartido de imágenes personalizadas en toda una organización. Las imágenes personalizadas son como las imágenes de Marketplace, pero las puede crear usted mismo. Las imágenes personalizadas pueden usarse para configuraciones de arranque como la carga previa de aplicaciones, configuraciones de aplicaciones y otras configuraciones del sistema operativo. 

Shared Image Gallery le permite compartir sus imágenes de máquina virtual personalizadas con otras personas. Elija las imágenes que desea compartir, qué regiones desea que estén disponibles en ellas y con quién desea compartirlas. 

La característica de galería de imágenes compartidas tiene varios tipos de recursos:

[!INCLUDE [virtual-machines-shared-image-gallery-resources](../../../includes/virtual-machines-shared-image-gallery-resources.md)]

## <a name="before-you-begin"></a>Antes de empezar

Los siguientes pasos explican cómo tomar una máquina virtual existente y convertirla en una imagen personalizada reutilizable que puede usar para crear nuevas instancias de máquinas virtuales.

Para completar el ejemplo de este tutorial, debe tener una máquina virtual. Si es necesario, puede consultar el [inicio rápido de la CLI](quick-create-cli.md) para crear una máquina virtual y usarla en este tutorial. Al trabajar en el tutorial, reemplace los nombres de recursos cuando sea necesario.

## <a name="launch-azure-cloud-shell"></a>Inicio de Azure Cloud Shell

Azure Cloud Shell es un shell interactivo gratuito que puede usar para ejecutar los pasos de este artículo. Tiene las herramientas comunes de Azure preinstaladas y configuradas para usarlas en la cuenta. 

Para abrir Cloud Shell, seleccione **Pruébelo** en la esquina superior derecha de un bloque de código. También puede ir a [https://shell.azure.com/powershell](https://shell.azure.com/powershell) para iniciar Cloud Shell en una pestaña independiente del explorador. Seleccione **Copiar** para copiar los bloques de código, péguelos en Cloud Shell y, luego, presione Entrar para ejecutarlos.

## <a name="create-an-image-gallery"></a>Creación de una galería de imágenes 

Una galería de imágenes es el recurso principal que se usa para habilitar el uso compartido de imágenes. 

Los caracteres permitidos para el nombre de la Galería son letras mayúsculas o minúsculas, números y puntos. El nombre de la galería no puede contener guiones.   Los nombres de las galerías deben ser únicos dentro de su suscripción. 

Cree una galería de imágenes mediante [az sig az create](/cli/azure/sig#az-sig-create). En el ejemplo siguiente se crea un grupo de recursos denominado *myGalleryRG* en la región *Este de EE. UU.* y una galería denominada *myGallery*.

```azurecli-interactive
az group create --name myGalleryRG --location eastus
az sig create --resource-group myGalleryRG --gallery-name myGallery
```

## <a name="get-information-about-the-vm"></a>Obtención de información acerca de la máquina virtual

Puede ver una lista de las máquinas virtuales que están disponibles mediante [az vm list](/cli/azure/vm#az-vm-list). 

```azurecli-interactive
az vm list --output table
```

Una vez que conozca el nombre de la máquina virtual y el grupo de recursos en el que se encuentra, obtenga el identificador de la máquina virtual con [az vm get-instance-view](/cli/azure/vm#az-vm-get-instance-view). 

```azurecli-interactive
az vm get-instance-view -g MyResourceGroup -n MyVm --query id
```

Copie el identificador de la máquina virtual para usarlo más adelante.

## <a name="create-an-image-definition"></a>Creación de la definición de una imagen

Las definiciones de imagen crean una agrupación lógica de imágenes. Estas se usan para administrar la información sobre las versiones de la imagen que se crean dentro de ellas. 

Los nombres de las definiciones de imagen pueden estar formados por letras mayúsculas o minúsculas, números, puntos y guiones. 

Para más información sobre los valores que se pueden especificar para una definición de imagen, consulte [Definiciones de imagen](./shared-image-galleries.md#image-definitions).

Cree una definición de imagen en la galería mediante [az sig image-definition create](/cli/azure/sig/image-definition#az-sig-image-definition-create). 

En este ejemplo, la definición de la imagen se denomina *myImageDefinition* y es para una imagen del sistema operativo Linux [especializada](./shared-image-galleries.md#generalized-and-specialized-images). 

```azurecli-interactive 
az sig image-definition create \
   --resource-group myGalleryRG \
   --gallery-name myGallery \
   --gallery-image-definition myImageDefinition \
   --publisher myPublisher \
   --offer myOffer \
   --sku mySKU \
   --os-type Linux \
   --os-state specialized
```

Copie el identificador de la definición de imagen de la salida para usarla más adelante.

## <a name="create-the-image-version"></a>Creación de la versión de la imagen

Cree una versión de la imagen a partir de la máquina virtual con [az image gallery create-image-version](/cli/azure/sig/image-version#az-sig-image-version-create).  

Los caracteres permitidos para la versión de una imagen son números y puntos. Los números deben estar dentro del rango de un entero de 32 bits. Formato: *VersiónPrincipal*.*VersiónSecundaria*.*Revisión*.

En este ejemplo, es la versión de la imagen es la *1.0.0* y vamos a crear dos réplicas en la región *Centro-oeste de EE. UU.* , una réplica en la región *Centro-sur de EE. UU.* y una réplica en la región *Este de EE. UU. 2* región mediante almacenamiento con redundancia de zona. Las regiones de replicación deben incluir la región donde se encuentra la máquina virtual de origen.

Reemplace el valor de `--managed-image` en este ejemplo por el identificador de la máquina virtual del paso anterior.

```azurecli-interactive 
az sig image-version create \
   --resource-group myGalleryRG \
   --gallery-name myGallery \
   --gallery-image-definition myImageDefinition \
   --gallery-image-version 1.0.0 \
   --target-regions "westcentralus" "southcentralus=1" "eastus=1=standard_zrs" \
   --replica-count 2 \
   --managed-image "/subscriptions/<Subscription ID>/resourceGroups/MyResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM"
```

> [!NOTE]
> Deberá esperar a que la versión de la imagen termine de compilarse y replicarse por completo antes de poder usar la misma imagen administrada para crear otra versión de la imagen.
>
> También puede almacenar la imagen en almacenamiento Premium agregando `--storage-account-type  premium_lrs` o en [almacenamiento con redundancia de zona](../../storage/common/storage-redundancy.md) agregando `--storage-account-type  standard_zrs` al crear la versión de la imagen.
>

 
## <a name="create-the-vm"></a>Creación de la máquina virtual

Cree la máquina virtual con [az vm create](/cli/azure/vm#az-vm-create) mediante el parámetro --specialized para indicar que la imagen es una imagen especializada. 

Use el identificador de la definición de imagen en `--image` a fin de crear la máquina virtual a partir de la versión más reciente de la imagen que está disponible. También puede crear la máquina virtual a partir de una versión específica si proporciona el identificador de la versión de la imagen en `--image`. 

En este ejemplo, se creará una máquina virtual a partir de la versión más reciente de la imagen *myImageDefinition*.

```azurecli
az group create --name myResourceGroup --location eastus
az vm create --resource-group myResourceGroup \
    --name myVM \
    --image "/subscriptions/<Subscription ID>/resourceGroups/myGalleryRG/providers/Microsoft.Compute/galleries/myGallery/images/myImageDefinition" \
    --specialized
```

## <a name="share-the-gallery"></a>Uso compartido de la galería

Puede compartir imágenes entre suscripciones mediante el Control de acceso basado en roles (RBAC). Puede compartir imágenes en el nivel de la galería, la definición de imagen o la versión de imagen. Cualquier usuario que tenga permisos de lectura para una versión de la imagen, incluso entre suscripciones, podrá implementar una VM con la versión de la imagen.

Se recomienda que el uso compartido con otros usuarios se realice en el nivel de la galería. Para obtener el identificador de objeto de la galería, use [az sig show](/cli/azure/sig#az-sig-show).

```azurecli-interactive
az sig show \
   --resource-group myGalleryRG \
   --gallery-name myGallery \
   --query id
```

Use el identificador de objeto como ámbito, junto con una dirección de correo electrónico y [az role assignment create](/cli/azure/role/assignment#az-role-assignment-create) para dar a un usuario acceso a la galería de imágenes compartidas. Reemplace `<email-address>` y `<gallery iD>` por su propia información.

```azurecli-interactive
az role assignment create \
   --role "Reader" \
   --assignee <email address> \
   --scope <gallery ID>
```

Para más información sobre cómo compartir recursos con RBAC, consulte [Administración del acceso mediante RBAC y la CLI de Azure](../../role-based-access-control/role-assignments-cli.md).

## <a name="azure-image-builder"></a>Azure Image Builder

Azure también ofrece un servicio, basado en Packer, [Azure VM Image Builder](./image-builder-overview.md). Solo tiene que describir sus personalizaciones en una plantilla y esta servirá para crear la imagen. 

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha creado una imagen de máquina virtual personalizada. Ha aprendido a:

> [!div class="checklist"]
> * Creación de una instancia de Shared Image Gallery
> * Creación de la definición de una imagen
> * Creación de la versión de una imagen
> * Crear una máquina virtual a partir de una imagen 
> * Compartir una galería de imágenes

Avance al siguiente tutorial para obtener información sobre máquinas virtuales de alta disponibilidad.

> [!div class="nextstepaction"]
> [Creación de máquinas virtuales de alta disponibilidad](tutorial-availability-sets.md)
