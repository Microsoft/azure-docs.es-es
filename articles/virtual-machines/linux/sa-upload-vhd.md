---
title: Carga de un disco de Linux personalizado con la CLI de Azure
description: Creación y carga en Azure de un disco duro virtual (VHD) mediante el modelo de implementación de Resource Manager y la CLI de Azure
author: cynthn
ms.service: virtual-machines-linux
ms.workload: infrastructure-services
ms.topic: how-to
ms.date: 07/10/2017
ms.author: cynthn
ms.custom: storage accounts
ms.openlocfilehash: d667ff3949fcf58affc72b5d75a40abd8ca0dcb6
ms.sourcegitcommit: 2ff0d073607bc746ffc638a84bb026d1705e543e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87830315"
---
# <a name="upload-and-create-a-linux-vm-from-custom-disk-with-the-azure-cli"></a>Carga y creación de una máquina virtual Linux a partir de un disco personalizado mediante la CLI de Azure

En este artículo se muestra cómo cargar un disco duro virtual (VHD) en una cuenta de Azure Storage mediante la CLI de Azure y cómo crear máquinas virtuales Linux desde este disco personalizado. Esta funcionalidad le permite instalar y configurar una distribución de Linux según sus requisitos y después utilizar ese disco duro virtual para crear rápidamente máquinas virtuales de Azure.

En este tema se usan las cuentas de almacenamiento de los discos duros virtuales finales, pero también puede seguir estos pasos con [discos administrados](upload-vhd.md). 

## <a name="quick-commands"></a>Comandos rápidos
Si necesita realizar rápidamente la tarea, en la siguiente sección se detallan los comandos base para cargar un disco duro virtual en Azure. Se puede encontrar información más detallada y contexto para cada paso en el resto del documento, [comenzando aquí](#requirements).

Asegúrese de que tiene la versión más reciente de la [CLI de Azure](/cli/azure/install-az-cli2) instalada y de que ha iniciado sesión en una cuenta de Azure con [az login](/cli/azure/reference-index).

En los ejemplos siguientes, reemplace los nombres de parámetros de ejemplo por los suyos propios. Nombres de parámetros de ejemplo incluidos `myResourceGroup`, `mystorageaccount` y `mydisks`.

En primer lugar, cree un grupo de recursos con [az group create](/cli/azure/group). En el ejemplo siguiente se crea un grupo de recursos denominado `myResourceGroup` en la ubicación `WestUs`:

```azurecli
az group create --name myResourceGroup --location westus
```

Cree una cuenta de almacenamiento que contenga los discos virtuales con [az storage account create](/cli/azure/storage/account). En el ejemplo siguiente se crea una cuenta de almacenamiento denominada `mystorageaccount`:

```azurecli
az storage account create --resource-group myResourceGroup --location westus \
  --name mystorageaccount --kind Storage --sku Standard_LRS
```

Cree una lista de las claves de acceso de la cuenta de almacenamiento con [az storage account keys list](/cli/azure/storage/account/keys). Tome nota de `key1`:

```azurecli
az storage account keys list --resource-group myResourceGroup --account-name mystorageaccount
```

Cree un contenedor en su cuenta de almacenamiento mediante la clave de almacenamiento que obtuvo con [az storage container create](/cli/azure/storage/container). En el ejemplo siguiente se crea un contenedor denominado `mydisks` con el valor de clave de almacenamiento de `key1`:

```azurecli
az storage container create --account-name mystorageaccount \
    --account-key key1 --name mydisks
```

Por último, cargue el disco duro virtual en el contenedor que creó con [az storage blob upload](/cli/azure/storage/blob). Especifique la ruta de acceso local para el disco duro virtual en `/path/to/disk/mydisk.vhd`:

```azurecli
az storage blob upload --account-name mystorageaccount \
    --account-key key1 --container-name mydisks --type page \
    --file /path/to/disk/mydisk.vhd --name myDisk.vhd
```

Especifique el URI del disco (`--image`) con [az vm create](/cli/azure/vm). En el ejemplo siguiente se crea una máquina virtual denominada `myVM` con el disco virtual cargado anteriormente:

```azurecli
az vm create --resource-group myResourceGroup --location westus \
    --name myVM --storage-account mystorageaccount --os-type linux \
    --admin-username azureuser --ssh-key-value ~/.ssh/id_rsa.pub \
    --image https://mystorageaccount.blob.core.windows.net/mydisk/myDisks.vhd \
    --use-unmanaged-disk
```

La cuenta de almacenamiento de destino debe ser la misma cuenta donde cargó su disco virtual. También debe especificar todos los parámetros adicionales requeridos, o responder a sus avisos, por el comando **az vm create**, como la red virtual, la dirección IP pública, el nombre de usuario y las claves SSH. Puede obtener más información sobre los [parámetros disponibles de Resource Manager de la CLI clásica](/previous-versions/azure/virtual-machines/azure-cli-arm-commands#virtual-machines).

## <a name="requirements"></a>Requisitos
Para completar los pasos siguientes, necesita:

* **Sistema operativo Linux instalado en un archivo .vhd**: instale una [distribución de Linux aprobada por Azure](endorsed-distros.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) (o consulte la [información para distribuciones no aprobadas](create-upload-generic.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)) en un disco virtual en el formato VHD. Existen varias herramientas para crear una máquina virtual y archivos VHD:
  * Instale y configure [QEMU](https://en.wikibooks.org/wiki/QEMU/Installing_QEMU) o [KVM](https://www.linux-kvm.org/page/RunningKVM), teniendo cuidado de usar VHD como formato de imagen. Si es necesario, puede [convertir una imagen](https://en.wikibooks.org/wiki/QEMU/Images#Converting_image_formats) con `qemu-img convert`.
  * También puede usar Hyper-V [en Windows 10](/virtualization/hyper-v-on-windows/quick-start/enable-hyper-v) o [en Windows Server 2012/2012 R2](/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/hh846766(v=ws.11)).

> [!NOTE]
> el reciente formato VHDX no se admite en Azure. Al crear una máquina virtual, especifique un VHD como formato. Si es necesario, puede convertir discos VHDX a VHD mediante [`qemu-img convert`](https://en.wikibooks.org/wiki/QEMU/Images#Converting_image_formats) o el cmdlet de PowerShell [`Convert-VHD`](/powershell/module/hyper-v/convert-vhd?view=win10-ps). Además, Azure no permite cargar VHD dinámicos, por lo que tendrá que convertir estos discos a VHD estáticos antes de cargarlos. Puede usar herramientas como, por ejemplo, [Azure VHD Utilities for GO](https://github.com/Microsoft/azure-vhd-utils-for-go) para convertir discos dinámicos durante el proceso de carga en Azure.
> 
> 

* Las máquinas virtuales creadas desde el disco personalizado deben residir en la misma cuenta de almacenamiento que el propio disco.
  * Cree una cuenta de almacenamiento y un contenedor para almacenar el disco personalizado y las máquinas virtuales creadas.
  * Una vez que haya creado todas las máquinas virtuales, puede eliminar el disco de forma segura.

Asegúrese de que tiene la versión más reciente de la [CLI de Azure](/cli/azure/install-az-cli2) instalada y de que ha iniciado sesión en una cuenta de Azure con [az login](/cli/azure/reference-index).

En los ejemplos siguientes, reemplace los nombres de parámetros de ejemplo por los suyos propios. Nombres de parámetros de ejemplo incluidos `myResourceGroup`, `mystorageaccount` y `mydisks`.

<a id="prepimage"> </a>

## <a name="prepare-the-disk-to-be-uploaded"></a>Preparación del disco que se va a cargar
Azure admite varias distribuciones Linux (consulte [Distribuciones aprobadas](endorsed-distros.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)). Los artículos siguientes le guiarán en el proceso de preparación de las distintas distribuciones de Linux admitidas en Azure:

* **[Distribuciones basadas en CentOS](create-upload-centos.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)**
* **[Debian Linux](debian-create-upload-vhd.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)**
* **[Oracle Linux](oracle-create-upload-vhd.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)**
* **[Red Hat Enterprise Linux](redhat-create-upload-vhd.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)**
* **[SLES y openSUSE](suse-create-upload-vhd.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)**
* **[Ubuntu](create-upload-ubuntu.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)**
* **[Otras distribuciones no aprobadas](create-upload-generic.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)**

Consulte también las **[notas de instalación de Linux](create-upload-generic.md#general-linux-installation-notes)** para más sugerencias generales sobre la preparación de imágenes de Linux para Azure.

> [!NOTE]
> El [Acuerdo de Nivel de Servicio de la plataforma Azure](https://azure.microsoft.com/support/legal/sla/virtual-machines/) se aplica a las máquinas virtuales que ejecutan Linux solo cuando una de las distribuciones aprobadas se use con los detalles de la configuración según se indica en la sección "Versiones admitidas" en [Linux en distribuciones aprobadas por Azure](endorsed-distros.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json).
> 
> 

## <a name="create-a-resource-group"></a>Crear un grupo de recursos
Los grupos de recursos reúnen de forma lógica todos los recursos de Azure que admiten sus máquinas virtuales, como el almacenamiento y las redes virtuales. Para más información acerca de los grupos de recursos, consulte [Información general de Azure Resource Manager](../../azure-resource-manager/management/overview.md). Antes de cargar el disco personalizado y crear máquinas virtuales, es preciso crear un grupo de recursos con [az group create](/cli/azure/group).

En el ejemplo siguiente se crea un grupo de recursos denominado `myResourceGroup` en la ubicación `westus`:

```azurecli
az group create --name myResourceGroup --location westus
```

## <a name="create-a-storage-account"></a>Crear una cuenta de almacenamiento

Cree una cuenta de almacenamiento para el disco personalizado y las máquinas virtuales con [az storage account create](/cli/azure/storage/account). Todas las máquinas virtuales que cree con discos no administrados deben estar en la misma cuenta de almacenamiento que ese disco. 

En el ejemplo siguiente se crea una cuenta de almacenamiento denominada `mystorageaccount` en el grupo de recursos que creó anteriormente:

```azurecli
az storage account create --resource-group myResourceGroup --location westus \
  --name mystorageaccount --kind Storage --sku Standard_LRS
```

## <a name="list-storage-account-keys"></a>Enumerar claves de cuentas de almacenamiento
Azure genera dos claves de acceso de 512 bits para cada cuenta de almacenamiento. Estas claves de acceso se usan al autenticar en la cuenta de almacenamiento, como la realización de operaciones de escritura. Para obtener más información sobre las claves de acceso de las cuentas de almacenamiento, vea [Administración de claves de acceso de cuentas de almacenamiento](../../storage/common/storage-account-keys-manage.md). Vea las teclas de acceso con [az storage account keys list](/cli/azure/storage/account/keys).

Consulte las claves de acceso de la cuenta de almacenamiento que ha creado:

```azurecli
az storage account keys list --resource-group myResourceGroup --account-name mystorageaccount
```

La salida es parecida a esta:

```output
info:    Executing command storage account keys list
+ Getting storage account keys
data:    Name  Key                                                                                       Permissions
data:    ----  ----------------------------------------------------------------------------------------  -----------
data:    key1  d4XAvZzlGAgWdvhlWfkZ9q4k9bYZkXkuPCJ15NTsQOeDeowCDAdB80r9zA/tUINApdSGQ94H9zkszYyxpe8erw==  Full
data:    key2  Ww0T7g4UyYLaBnLYcxIOTVziGAAHvU+wpwuPvK4ZG0CDFwu/mAxS/YYvAQGHocq1w7/3HcalbnfxtFdqoXOw8g==  Full
info:    storage account keys list command OK
```

Tome nota de `key1` , pues el objetivo de su uso será interactuar con su cuenta de almacenamiento en los siguientes pasos.

## <a name="create-a-storage-container"></a>Creación de un contenedor de almacenamiento
De la misma manera que crea directorios distintos para organizar de forma lógica el sistema de archivos local, crea contenedores dentro de una cuenta de almacenamiento para organizar los discos. Una cuenta de almacenamiento puede contener un número cualquiera de contenedores. Cree un contenedor con [az storage container create](/cli/azure/storage/container).

En el ejemplo siguiente se crea un contenedor llamado `mydisks`:

```azurecli
az storage container create \
    --account-name mystorageaccount \
    --name mydisks
```

## <a name="upload-vhd"></a>Carga de VHD
Ahora puede cargar su disco personalizado con [az storage blob upload](/cli/azure/storage/blob). Cargue y almacene el disco personalizado como un blob en páginas.

Especifique la clave de acceso, el contenedor que creó en el paso anterior y, a continuación, la ruta de acceso al disco personalizado del equipo local:

```azurecli
az storage blob upload --account-name mystorageaccount \
    --account-key key1 --container-name mydisks --type page \
    --file /path/to/disk/mydisk.vhd --name myDisk.vhd
```

## <a name="create-the-vm"></a>Creación de la máquina virtual
Para crear una máquina virtual con discos no administrados, especifique el URI en el disco (`--image`) con [az vm create](/cli/azure/vm). En el ejemplo siguiente se crea una máquina virtual denominada `myVM` con el disco virtual cargado anteriormente:

Especifique el parámetro `--image` con [az vm create](/cli/azure/vm) para apuntar al disco personalizado. Asegúrese de que `--storage-account` coincide con la cuenta de almacenamiento donde se almacena el disco personalizado. No es necesario utilizar el mismo contenedor que disco personalizado para almacenar las máquinas virtuales. Asegúrese de crear contenedores adicionales siguiendo los pasos anteriores antes de cargar el disco personalizado.

En el ejemplo siguiente se crea una máquina virtual denominada `myVM` a partir del disco personalizado:

```azurecli
az vm create --resource-group myResourceGroup --location westus \
    --name myVM --storage-account mystorageaccount --os-type linux \
    --admin-username azureuser --ssh-key-value ~/.ssh/id_rsa.pub \
    --image https://mystorageaccount.blob.core.windows.net/mydisks/myDisk.vhd \
    --use-unmanaged-disk
```

También debe especificar todos los parámetros adicionales que requiere el comando **az vm create**, como el nombre de usuario y las claves SSH, así como responder a sus indicaciones.


## <a name="resource-manager-template"></a>Plantilla de Resource Manager
Las plantillas de Azure Resource Manager son archivos de Notación de objetos JavaScript (JSON) que definen el entorno que desea crear. Las plantillas se desglosan en distintos proveedores de recursos, tales como proceso o red. Puede usar las plantillas existentes o escribir las suyas propias. Lea más sobre el [uso de Resource Manager y las plantillas](../../azure-resource-manager/management/overview.md).

Dentro del proveedor `Microsoft.Compute/virtualMachines` de la plantilla, tendrá un nodo `storageProfile` que contiene los detalles de configuración de la máquina virtual. Los dos parámetros principales para modificar son los URI `image` y `vhd` que apuntan al disco personalizado y el disco virtual de la nueva máquina virtual. A continuación se muestra un ejemplo de JSON para el uso de un disco personalizado:

```json
"storageProfile": {
          "osDisk": {
            "name": "myVM",
            "osType": "Linux",
            "caching": "ReadWrite",
            "createOption": "FromImage",
            "image": {
              "uri": "https://mystorageaccount.blob.core.windows.net/mydisks/myDisk.vhd"
            },
            "vhd": {
              "uri": "https://mystorageaccount.blob.core.windows.net/vhds/newvmname.vhd"
            }
          }
```

Puede usar [esta plantilla existente para crear una máquina virtual desde una imagen personalizada](https://github.com/Azure/azure-quickstart-templates/tree/master/101-vm-from-user-image) o leer más sobre la [creación de sus propias plantillas de Azure Resource Manager](../../azure-resource-manager/templates/template-syntax.md). 

Una vez que tenga una plantilla configurada, utilice [az group deployment create](/cli/azure/group/deployment) para crear las máquinas virtuales. Especifique el URI de la plantilla JSON con el parámetro `--template-uri` :

```azurecli
az group deployment create --resource-group myNewResourceGroup \
  --template-uri https://uri.to.template/mytemplate.json
```

Si tiene un archivo JSON almacenado localmente en el equipo, puede usar el parámetro `--template-file` en su lugar:

```azurecli
az group deployment create --resource-group myNewResourceGroup \
  --template-file /path/to/mytemplate.json
```


## <a name="next-steps"></a>Pasos siguientes
Después de haber preparado y cargado el disco virtual personalizado, puede leer más sobre el [uso de Resource Manager y las plantillas](../../azure-resource-manager/management/overview.md). También es posible que quiera [agregar un disco de datos](add-disk.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) a las nuevas máquinas virtuales. Si tiene aplicaciones que se ejecutan en las máquinas virtuales a las que necesite tener acceso, asegúrese de [abrir puertos y puntos de conexión](nsg-quickstart.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json).