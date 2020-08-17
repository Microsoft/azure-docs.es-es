---
title: 'Tutorial: Uso de una imagen de máquina virtual personalizada en un conjunto de escalado con la CLI de Azure'
description: Obtenga información sobre cómo usar la CLI de Azure para crear una imagen de máquina virtual personalizada que puede usar para implementar un conjunto de escalado de máquinas virtuales
author: cynthn
ms.service: virtual-machine-scale-sets
ms.subservice: imaging
ms.topic: tutorial
ms.date: 05/01/2020
ms.author: cynthn
ms.custom: mvc, devx-track-azurecli
ms.reviewer: akjosh
ms.openlocfilehash: dd0cf450ca63349d29aba3d65f3c76f40a44be2a
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87503640"
---
# <a name="tutorial-create-and-use-a-custom-image-for-virtual-machine-scale-sets-with-the-azure-cli"></a>Tutorial: Creación y uso de una imagen personalizada para conjuntos de escalado de máquinas virtuales con la CLI de Azure
Al crear el conjunto de escalado, se especifica la imagen que se usará cuando se implementen las instancias de máquina virtual. Para reducir el número de tareas después de implementar las instancias de máquina virtual, puede usar una imagen de máquina virtual personalizada. Esta imagen de máquina virtual personalizada incluye la instalación o configuración de las aplicaciones necesarias. Las instancias de máquina virtual creadas en el conjunto de escalado usan la imagen de máquina virtual personalizada y están listas para atender el tráfico de la aplicación. En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Creación de una instancia de Shared Image Gallery
> * Crear una definición de imagen especializada
> * Creación de la versión de una imagen
> * Crear un conjunto de escalado a partir de una imagen especializada
> * Compartir una galería de imágenes


Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

Si decide instalar y usar la CLI de forma local, en este tutorial es preciso que ejecute la CLI de Azure de la versión 2.4.0, u otra posterior. Ejecute `az --version` para encontrar la versión. Si necesita instalarla o actualizarla, vea [Instalación de la CLI de Azure]( /cli/azure/install-azure-cli).

## <a name="overview"></a>Información general

Una [galería de imágenes compartidas](shared-image-galleries.md) simplifica el uso compartido de imágenes personalizadas en toda una organización. Las imágenes personalizadas son como las imágenes de Marketplace, pero las puede crear usted mismo. Las imágenes personalizadas pueden usarse para configuraciones de arranque como la carga previa de aplicaciones, configuraciones de aplicaciones y otras configuraciones del sistema operativo. 

Shared Image Gallery le permite compartir sus imágenes de máquina virtual personalizadas con otras personas. Elija las imágenes que desea compartir, qué regiones desea que estén disponibles en ellas y con quién desea compartirlas. 

## <a name="create-and-configure-a-source-vm"></a>Creación y configuración de una máquina virtual de origen

En primer lugar, cree un grupo de recursos con [az group create](/cli/azure/group) y luego cree una máquina virtual con [az vm create](/cli/azure/vm). Esta máquina virtual se usa después como origen de la imagen. En el ejemplo siguiente, se crea una máquina virtual llamada *myVM* en el grupo de recursos llamado *myResourceGroup*:

```azurecli-interactive
az group create --name myResourceGroup --location eastus

az vm create \
  --resource-group myResourceGroup \
  --name myVM \
  --image ubuntults \
  --admin-username azureuser \
  --generate-ssh-keys
```

> [!IMPORTANT]
> El **identificador** de la máquina virtual se muestra en la salida del comando [az vm create](/cli/azure/vm). Cópielo en un lugar seguro para poder usarlo más adelante en este tutorial.

La dirección IP pública de la máquina virtual se muestra también en la salida del comando [az vm create](/cli/azure/vm). Conéctese mediante SSH a la dirección IP pública de la máquina virtual, como se indica a continuación:

```console
ssh azureuser@<publicIpAddress>
```

Para personalizar la máquina virtual, vamos a instalar un servidor web básico. Cuando la instancia de máquina virtual del conjunto de escalado se implemente, tendrá todos los paquetes necesarios para ejecutar una aplicación web. Use `apt-get` para instalar *NGINX* de la siguiente manera:

```bash
sudo apt-get install -y nginx
```

Cuando haya terminado, escriba `exit` para desconectar la sesión SSH.

## <a name="create-an-image-gallery"></a>Creación de una galería de imágenes 

Una galería de imágenes es el recurso principal que se usa para habilitar el uso compartido de imágenes. 

Los caracteres permitidos para el nombre de la Galería son letras mayúsculas o minúsculas, números y puntos. El nombre de la galería no puede contener guiones.   Los nombres de las galerías deben ser únicos dentro de su suscripción. 

Cree una galería de imágenes mediante [az sig az create](/cli/azure/sig#az-sig-create). En el ejemplo siguiente se crea un grupo de recursos denominado *myGalleryRG* en la región *Este de EE. UU.* y una galería denominada *myGallery*.

```azurecli-interactive
az group create --name myGalleryRG --location eastus
az sig create --resource-group myGalleryRG --gallery-name myGallery
```

## <a name="create-an-image-definition"></a>Creación de la definición de una imagen

Las definiciones de imagen crean una agrupación lógica de imágenes. Estas se usan para administrar la información sobre las versiones de la imagen que se crean dentro de ellas. 

Los nombres de las definiciones de imagen pueden estar formados por letras mayúsculas o minúsculas, números, puntos y guiones. 

Asegúrese de que la definición de la imagen sea del tipo correcto. Si ha generalizado la máquina virtual (con Sysprep para Windows o waagent-deprovision para Linux), debe crear una definición de imagen generalizada mediante `--os-state generalized`. Si quiere usar la máquina virtual sin quitar las cuentas de usuario existentes, cree una definición de imagen especializada mediante `--os-state specialized`.

Para más información sobre los valores que se pueden especificar para una definición de imagen, consulte [Definiciones de imagen](../virtual-machines/linux/shared-image-galleries.md#image-definitions).

Cree una definición de imagen en la galería mediante [az sig image-definition create](/cli/azure/sig/image-definition#az-sig-image-definition-create).

En este ejemplo, la definición de la imagen se denomina *myImageDefinition* y es para una imagen del sistema operativo Linux [especializada](../virtual-machines/linux/shared-image-galleries.md#generalized-and-specialized-images). Para crear una definición para las imágenes que usan un sistema operativo Windows, utilice `--os-type Windows`. 

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

> [!IMPORTANT]
> El **identificador** de la definición de la imagen se muestra en la salida del comando. Cópielo en un lugar seguro para poder usarlo más adelante en este tutorial.


## <a name="create-the-image-version"></a>Creación de la versión de la imagen

Cree una versión de la imagen desde la máquina virtual con [az image gallery create-image-version](/cli/azure/sig/image-version#az-sig-image-version-create).  

Los caracteres permitidos para la versión de una imagen son números y puntos. Los números deben estar dentro del rango de un entero de 32 bits. Formato: *VersiónPrincipal*.*VersiónSecundaria*.*Revisión*.

En este ejemplo, la versión de la imagen es *1.0.0* y vamos a crear una réplica en la región *Centro-sur de EE. UU.* y otra réplica en la región *Este de EE. UU. 2*. Las regiones de replicación deben incluir la región donde se encuentra la máquina virtual de origen.

Reemplace el valor de `--managed-image` de este ejemplo por el identificador de la máquina virtual del paso anterior.

```azurecli-interactive 
az sig image-version create \
   --resource-group myGalleryRG \
   --gallery-name myGallery \
   --gallery-image-definition myImageDefinition \
   --gallery-image-version 1.0.0 \
   --target-regions "southcentralus=1" "eastus=1" \
   --managed-image "/subscriptions/<Subscription ID>/resourceGroups/MyResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM"
```

> [!NOTE]
> Deberá esperar a que la versión de la imagen termine de compilarse y replicarse por completo antes de poder usar la misma imagen administrada para crear otra versión de la imagen.
>
> También puede almacenar la imagen en almacenamiento Premium agregando `--storage-account-type  premium_lrs` o en [almacenamiento con redundancia de zona](../storage/common/storage-redundancy.md) agregando `--storage-account-type  standard_zrs` al crear la versión de la imagen.
>




## <a name="create-a-scale-set-from-the-image"></a>Creación de un conjunto de escalado a partir de una imagen
Cree un conjunto de escalado a partir de la imagen especializada mediante [`az vmss create`](/cli/azure/vmss#az-vmss-create). 

Cree el conjunto de escalado mediante [`az vmss create`](/cli/azure/vmss#az-vmss-create) con el parámetro --specialized para indicar que la imagen es una imagen especializada. 

Use el identificador de definición de imagen en `--image` a fin de crear las instancias del conjunto de escalado a partir de la versión más reciente de la imagen que esté disponible. También puede crear las instancias del conjunto de escalado a partir de una versión específica si proporciona el identificador de la versión de la imagen en `--image`. 

Cree un conjunto de escalado denominado *myScaleSet* con la versión más reciente de la imagen *myImageDefinition* que hemos creado anteriormente.

```azurecli
az group create --name myResourceGroup --location eastus
az vmss create \
   --resource-group myResourceGroup \
   --name myScaleSet \
   --image "/subscriptions/<Subscription ID>/resourceGroups/myGalleryRG/providers/Microsoft.Compute/galleries/myGallery/images/myImageDefinition" \
   --specialized
```

Se tardan unos minutos en crear y configurar todos los recursos de conjunto de escalado y máquinas virtuales.


## <a name="test-your-scale-set"></a>Prueba del conjunto de escalado
Para permitir que el tráfico llegue al conjunto de escalado y comprobar que el servidor web funciona correctamente, cree una regla del equilibrador de carga con [az network lb rule create](/cli/azure/network/lb/rule). En el ejemplo siguiente, se crea una regla denominada *myLoadBalancerRuleWeb* que permite tráfico en el puerto *TCP* *80*:

```azurecli-interactive
az network lb rule create \
  --resource-group myResourceGroup \
  --name myLoadBalancerRuleWeb \
  --lb-name myScaleSetLB \
  --backend-pool-name myScaleSetLBBEPool \
  --backend-port 80 \
  --frontend-ip-name loadBalancerFrontEnd \
  --frontend-port 80 \
  --protocol tcp
```

Para ver el conjunto de escalado en acción, obtenga la dirección IP pública del equilibrador de carga con [az network public-ip show](/cli/azure/network/public-ip). En el ejemplo siguiente se obtiene la dirección IP de *myScaleSetLBPublicIP* que se ha creado como parte del conjunto de escalado:

```azurecli-interactive
az network public-ip show \
  --resource-group myResourceGroup \
  --name myScaleSetLBPublicIP \
  --query [ipAddress] \
  --output tsv
```

Escriba la dirección IP pública en un explorador web. Se muestra la página web predeterminada de NGIN, como en el ejemplo siguiente:

![NGINX en ejecución desde la imagen de máquina virtual personalizada](media/tutorial-use-custom-image-cli/default-nginx-website.png)



## <a name="share-the-gallery"></a>Uso compartido de la galería

Puede compartir imágenes entre suscripciones mediante el Control de acceso basado en roles (RBAC). Puede compartir imágenes en la galería, la definición de la imagen o la versión de la imagen. Cualquier usuario que tenga permisos de lectura para una versión de la imagen, incluso entre suscripciones, podrá implementar una máquina virtual con la versión de la imagen.

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

Para más información sobre cómo compartir recursos con RBAC, consulte [Administración del acceso mediante RBAC y la CLI de Azure](../role-based-access-control/role-assignments-cli.md).


## <a name="clean-up-resources"></a>Limpieza de recursos
Para quitar el conjunto de escalado y los recursos adicionales, elimine el grupo de recursos y todos sus recursos con [az group delete](/cli/azure/group). El parámetro `--no-wait` devuelve el control a la petición de confirmación sin esperar a que finalice la operación. El parámetro `--yes` confirma que desea eliminar los recursos sin pedir confirmación adicional.

```azurecli-interactive
az group delete --name myResourceGroup --no-wait --yes
```


## <a name="next-steps"></a>Pasos siguientes
En este tutorial, aprendió a crear y utilizar una imagen de máquina virtual personalizada para los conjuntos de escalado con la CLI de Azure:

> [!div class="checklist"]
> * Creación de una instancia de Shared Image Gallery
> * Creación de una definición de imagen especializada
> * Creación de la versión de una imagen
> * Creación de un conjunto de escalado a partir de una imagen especializada
> * Uso compartido de una galería de imágenes

Vaya al siguiente tutorial para aprender cómo implementar aplicaciones en el conjunto de escalado.

> [!div class="nextstepaction"]
> [Implementación de aplicaciones en los conjuntos de escalado](tutorial-install-apps-cli.md)
