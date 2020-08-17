---
title: Captura de una imagen administrada de una máquina virtual Linux con la CLI de Azure
description: Capture una imagen administrada de una máquina virtual de Azure que se usará para realizar implementaciones masivas mediante la CLI de Azure.
author: cynthn
ms.service: virtual-machines
ms.subservice: imaging
ms.topic: how-to
ms.date: 10/08/2018
ms.author: cynthn
ms.custom: legacy, devx-track-azurecli
ms.openlocfilehash: 376d9d76633060f504454f85841b9c15bafc6685
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87503045"
---
# <a name="how-to-create-a-managed-image-of-a-virtual-machine-or-vhd"></a>Creación de una imagen administrada de una máquina virtual o un disco duro virtual

Para crear varias copias de una máquina virtual para usarla en Azure para desarrollo y prueba, capture una imagen administrada de la máquina virtual o del disco duro virtual del sistema operativo. Para crear, almacenar y compartir imágenes a gran escala, consulte [Instancias de Shared Image Gallery](../shared-images-cli.md).

Una sola imagen administrada admite hasta 20 implementaciones simultáneas. Cuando se intentan crear más de 20 máquinas virtuales simultáneamente, desde una misma imagen administrada, se pueden producir tiempos de espera de aprovisionamiento debido a las limitaciones de rendimiento de almacenamiento de un solo VHD. Para crear más de 20 máquinas virtuales simultáneamente, use una imagen de la [galería de imágenes compartidas](shared-image-galleries.md) configurada con una réplica por cada 20 implementaciones simultáneas de máquina virtual.

Para crear una imagen administrada, deberá quitar la información personal de la cuenta. En los pasos siguientes, se desaprovisiona una máquina virtual existente, se desasigna y se crea una imagen. Puede usar esta imagen para crear VM en cualquier grupo de recursos dentro de su suscripción.

Para crear una copia de la máquina virtual Linux existente para la realización de una copia de seguridad o para su depuración o para cargar un disco duro virtual Linux especializado desde una máquina virtual local, consulte [Carga y creación de una máquina virtual Linux desde una imagen de disco personalizada](upload-vhd.md).  

Puede utilizar el servicio **Azure VM Image Builder (versión preliminar pública)** para crear la imagen personalizada, sin necesidad de aprender las herramientas o configurar las canalizaciones de compilación, simplemente proporcionando una configuración de imagen e Image Builder creará la imagen. Para más información, consulte [Introducción a Azure VM Image Builder](./image-builder-overview.md).

Necesitará los siguientes elementos antes de crear una imagen:

* Una máquina virtual de Azure creada en el modelo de implementación de Resource Manager mediante discos administrados. Si aún no ha creado una máquina virtual Linux, puede usar el [portal](quick-create-portal.md), la [CLI de Azure](quick-create-cli.md) o [plantillas de Resource Manager](create-ssh-secured-vm-from-template.md). Configure la máquina virtual como considere necesario. Por ejemplo, [agregue discos de datos](add-disk.md), aplique actualizaciones e instale aplicaciones. 

* Necesita tener instalada la versión más reciente de la [CLI de Azure](/cli/azure/install-az-cli2) y haber iniciado sesión en una cuenta de Azure con [az login](/cli/azure/reference-index#az-login).

## <a name="prefer-a-tutorial-instead"></a>¿Prefiere un tutorial en su lugar?

Para una versión simplificada de este artículo, con fines de prueba, evaluación o aprendizaje sobre máquinas virtuales de Azure, consulte [Creación de una imagen personalizada de una máquina virtual de Azure mediante la CLI](tutorial-custom-images.md).  De lo contrario, siga leyendo para obtener la imagen completa.


## <a name="step-1-deprovision-the-vm"></a>Paso 1: Desaprovisionar la máquina virtual
En primer lugar, desaprovisione la máquina virtual con el agente de máquinas virtuales de Azure para eliminar los archivos y datos específicos de la máquina. Use el comando `waagent` con el parámetro `-deprovision+user` en la máquina virtual Linux de origen. Consulte la [Guía de usuario del Agente de Linux de Azure](../extensions/agent-linux.md) para más información.

1. Conéctese a la máquina virtual Linux con un cliente de SSH.
2. En la ventana de SSH, escriba el siguiente comando:
   
    ```bash
    sudo waagent -deprovision+user
    ```
   > [!NOTE]
   > Solo puede ejecutar este comando en una máquina virtual que quiera capturar como imagen. Este comando no garantiza que se haya borrado toda información confidencial de la imagen o que sea adecuada para su redistribución. El parámetro `+user` también elimina la última cuenta de usuario aprovisionada. Para mantener las credenciales de la cuenta de usuario en la máquina virtual, use solo `-deprovision`.
 
3. Escriba **s** para continuar. Puede agregar el parámetro `-force` para evitar este paso de confirmación.
4. Una vez finalizado el comando, escriba **exit** para cerrar el cliente de SSH.  La máquina virtual seguirá ejecutándose en este momento.

## <a name="step-2-create-vm-image"></a>Paso 2: Crear imagen de máquina virtual
Use la CLI de Azure para marcar la máquina virtual como generalizada y capture la imagen. En los ejemplos siguientes, reemplace los nombres de parámetros de ejemplo por los suyos propios. Los nombres de parámetro de ejemplo incluyen *myResourceGroup*, *myVnet* y *myVM*.

1. Desasigne la VM que desaprovisionó con [az vm deallocate](/cli/azure/vm). En el ejemplo siguiente se desasigna la máquina virtual llamada *myVM* en el grupo de recursos *myResourceGroup*.  
   
    ```azurecli
    az vm deallocate \
      --resource-group myResourceGroup \
      --name myVM
    ```
    
    Espere a que la máquina virtual se desasigne por completo antes de moverla. Esta operación puede tardar algunos minutos en completarse.  La máquina virtual se apaga durante la desasignación.

2. Marque la máquina virtual como generalizada con [az vm generalize](/cli/azure/vm). En el ejemplo siguiente se marca como generalizada la máquina virtual llamada *myVM* en el grupo de recursos *myResourceGroup*.
   
    ```azurecli
    az vm generalize \
      --resource-group myResourceGroup \
      --name myVM
    ```

    Una máquina virtual que ha sido generalizada ya no se puede reiniciar.

3. Cree una imagen del recurso de máquina virtual con [az image create](/cli/azure/image#az-image-create). En el ejemplo siguiente se crea una imagen llamada *myImage* en el grupo de recursos llamado *myResourceGroup* por medio del recurso de máquina virtual llamado *myVM*.
   
    ```azurecli
    az image create \
      --resource-group myResourceGroup \
      --name myImage --source myVM
    ```
   
   > [!NOTE]
   > La imagen se crea en el mismo grupo de recursos que la VM de origen. Puede crear VM en cualquier grupo de recursos dentro de la suscripción a partir de esta imagen. Desde una perspectiva de administración, puede que desee crear un grupo de recursos específico para las imágenes y los recursos de VM.
   >
   > Si desea almacenar la imagen en un almacenamiento resistente a zonas, debe crearla en una región que admita [zonas de disponibilidad ](../../availability-zones/az-overview.md) e incluir el parámetro `--zone-resilient true`.
   
Este comando devuelve JSON que describe la imagen de la máquina virtual. Guarde esta salida para su posterior referencia.

## <a name="step-3-create-a-vm-from-the-captured-image"></a>Paso 3: Crear una máquina virtual a partir de la imagen capturada
Cree una máquina virtual con la imagen que ha creado con [az vm create](/cli/azure/vm). En el ejemplo siguiente se crea una máquina virtual llamada *myVMDeployed* a partir de la imagen *myImage*.

```azurecli
az vm create \
   --resource-group myResourceGroup \
   --name myVMDeployed \
   --image myImage\
   --admin-username azureuser \
   --ssh-key-value ~/.ssh/id_rsa.pub
```

### <a name="creating-the-vm-in-another-resource-group"></a>Creación de la máquina virtual en otro grupo de recursos 

Puede crear máquinas virtuales en cualquier grupo de recursos dentro de la suscripción a partir de una imagen. Para crear una VM en un grupo de recursos diferente al de la imagen, especifique el identificador de recurso completo para la imagen. Use [az image list](/cli/azure/image#az-image-list) para ver una lista de imágenes. La salida será similar al del ejemplo siguiente:

```json
"id": "/subscriptions/guid/resourceGroups/MYRESOURCEGROUP/providers/Microsoft.Compute/images/myImage",
   "location": "westus",
   "name": "myImage",
```

En el ejemplo siguiente se utiliza [az vm create](/cli/azure/vm#az-vm-create) para crear una máquina virtual en un grupo de recursos diferente al de la imagen de origen especificando el identificador de recurso de imagen.

```azurecli
az vm create \
   --resource-group myOtherResourceGroup \
   --name myOtherVMDeployed \
   --image "/subscriptions/guid/resourceGroups/MYRESOURCEGROUP/providers/Microsoft.Compute/images/myImage" \
   --admin-username azureuser \
   --ssh-key-value ~/.ssh/id_rsa.pub
```


## <a name="step-4-verify-the-deployment"></a>Paso 4: Comprobar la implementación

Conéctese por SSH a la máquina virtual que creó para comprobar la implementación y empezar a usar la nueva máquina virtual. Para conectarse mediante SSH, busque la dirección IP o el nombre de dominio completo de la máquina virtual con [az vm show](/cli/azure/vm#az-vm-show).

```azurecli
az vm show \
   --resource-group myResourceGroup \
   --name myVMDeployed \
   --show-details
```

## <a name="next-steps"></a>Pasos siguientes
Para crear, almacenar y compartir imágenes a gran escala, consulte [Instancias de Shared Image Gallery](../shared-images-cli.md).
