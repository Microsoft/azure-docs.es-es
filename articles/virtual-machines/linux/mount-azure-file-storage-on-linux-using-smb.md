---
title: Montaje de Azure File Storage en máquinas virtuales Linux con SMB
description: Procedimientos de montaje de Azure File Storage en máquinas virtuales Linux mediante SMB con la CLI de Azure
author: cynthn
ms.service: virtual-machines-linux
ms.topic: how-to
ms.workload: infrastructure
ms.date: 06/28/2018
ms.author: cynthn
ms.openlocfilehash: 6a5d4f388d9e5f47a13812687489525590c520c3
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87085493"
---
# <a name="mount-azure-file-storage-on-linux-vms-using-smb"></a>Montaje de Azure File Storage en máquinas virtuales Linux con SMB

En este artículo se muestra cómo usar el servicio Azure File Storage en una VM Linux mediante un montaje de SMB con la CLI de Azure. El almacenamiento de archivos de Azure ofrece recursos compartidos de archivos en la nube mediante el protocolo SMB estándar. 

El almacenamiento de archivos ofrece recursos compartidos de archivos en la nube que usan el protocolo SMB estándar. Puede montar un recurso compartido de archivos desde cualquier sistema operativo que admita SMB 3.0. Cuando se usa un montaje de SMB en Linux se obtienen copias de seguridad fáciles en una ubicación de almacenamiento de archivado permanente y sólida que se proporciona mediante un Acuerdo de Nivel de Servicio.

El movimiento de archivos de una VM a un montaje de SMB que se hospeda en File Storage es una excelente manera de depurar registros. El mismo recurso compartido de SMB se puede montar localmente en la estación de trabajo de Windows, Linux o Mac. SMB no es la mejor solución para transmitir registros de aplicación o Linux en tiempo real porque el protocolo SMB no está creado para controlar tareas de registro tan pesadas. Una herramienta de nivel de registro unificado dedicada como Fluentd sería una opción mejor que SMB para recopilar la salida de registro de la aplicación y Linux.

Para esta guía es necesario que ejecute la versión 2.0.4 o superior de la CLI de Azure. Para saber qué versión tiene, ejecute el comando **az --version**. Si necesita instalarla o actualizarla, consulte [Instalación de la CLI de Azure](/cli/azure/install-azure-cli). 


## <a name="create-a-resource-group"></a>Crear un grupo de recursos

Cree un grupo de recursos denominado *myResourceGroup* en la ubicación *Este de EE. UU.*

```azurecli
az group create --name myResourceGroup --location eastus
```

## <a name="create-a-storage-account"></a>Crear una cuenta de almacenamiento

Cree una cuenta de almacenamiento, en el grupo de recursos que ha creado, con [az storage account create](/cli/azure/storage/account). En este ejemplo se crea una cuenta de almacenamiento denominada *mySTORAGEACCT\<random number>* y coloca el nombre de esa cuenta de almacenamiento en la variable **STORAGEACCT**. Los nombres de la cuenta de almacenamiento deben ser únicos, usar `$RANDOM` anexa un número al final para que sea único.

```azurecli
STORAGEACCT=$(az storage account create \
    --resource-group "myResourceGroup" \
    --name "mystorageacct$RANDOM" \
    --location eastus \
    --sku Standard_LRS \
    --query "name" | tr -d '"')
```

## <a name="get-the-storage-key"></a>Obtención de la clave de almacenamiento

Cuando cree una cuenta de almacenamiento, las claves de cuenta de almacenamiento se crean en pares de modo que las claves se pueden girar sin ninguna interrupción del servicio. Cuando cambia a la segunda clave del par, se crea un nuevo par de claves. Las nuevas claves de la cuenta de almacenamiento se crean siempre por pares, por lo que siempre tendrá lista al menos una clave de cuenta de almacenamiento sin usar a la cual cambiar.

Para ver las claves de la cuenta de almacenamiento, use [az storage account keys list](/cli/azure/storage/account/keys). En este ejemplo se almacena el valor de clave 1 en la variable **STORAGEKEY**.

```azurecli
STORAGEKEY=$(az storage account keys list \
    --resource-group "myResourceGroup" \
    --account-name $STORAGEACCT \
    --query "[0].value" | tr -d '"')
```

## <a name="create-a-file-share"></a>Creación de un recurso compartido de archivos

Crear el recurso compartido de File Storage con [az storage share create](/cli/azure/storage/share). 

Los nombres de recursos compartidos deben estar formados por letras minúsculas, números y guiones sencillos, pero no pueden empezar con un guion. Para obtener detalles completos sobre cómo asignar un nombre a recursos compartidos y archivos, consulte [Asignación de nombres y referencia a recursos compartidos, directorios, archivos y metadatos](/rest/api/storageservices/naming-and-referencing-shares--directories--files--and-metadata).

En este ejemplo se crea un recurso compartido denominado *myshare* con una cuota de 10 GiB. 

```azurecli
az storage share create --name myshare \
    --quota 10 \
    --account-name $STORAGEACCT \
    --account-key $STORAGEKEY
```

## <a name="create-a-mount-point"></a>Creación de un punto de montaje

Para montar el recurso compartido de Azure File en el equipo Linux, deberá asegurarse de tener instalado el paquete **cifs-utils**. Para obtener las instrucciones de instalación, consulte [Instale el paquete cifs-utils correspondiente a su distribución de Linux](../../storage/files/storage-how-to-use-files-linux.md#install-cifs-utils).

Azure Files usa el protocolo SMB, que se comunica a través del puerto TCP 445.  Si tiene dificultades para montar el recurso compartido de Azure File, asegúrese de que el firewall no esté bloqueando el puerto TCP 445.


```bash
mkdir -p /mnt/MyAzureFileShare
```

## <a name="mount-the-share"></a>Montaje del recurso compartido

Monte el recurso compartido de Azure File en el directorio local. 

```bash
sudo mount -t cifs //$STORAGEACCT.file.core.windows.net/myshare /mnt/MyAzureFileShare -o vers=3.0,username=$STORAGEACCT,password=$STORAGEKEY,dir_mode=0777,file_mode=0777,serverino
```

El comando anterior usa el comando [mount](https://linux.die.net/man/8/mount) para montar el recurso compartido de archivos de Azure y opciones específicas de [cifs](https://linux.die.net/man/8/mount.cifs). En concreto, las opciones dir_mode y file_mode establecen el permiso `0777` para archivos y directorios. El permiso `0777` proporciona permiso de lectura, escritura y ejecución a todos los usuarios. Puede cambiar estos permisos reemplazando los valores por otros [permisos chmod](https://en.wikipedia.org/wiki/Chmod). También puede utilizar otras opciones de [cifs](https://linux.die.net/man/8/mount.cifs) como uid o gid. 


## <a name="persist-the-mount"></a>Hacer persistente el montaje

Cuando reinicie la VM de Linux, el recurso compartido de SMB montado se desmontará durante el cierre. Para volver a montar el recurso compartido de SMB al inicio, agregue una línea a /etc/fstab de Linux. Linux utiliza el archivo fstab para enumerar los sistemas de archivos que necesita montar durante el proceso de arranque. Agregue el recurso compartido de SMB para garantizar que el recurso compartido de File Storage es un sistema de archivos montado de manera permanente para la VM Linux. La adición del recurso compartido de SMB de File Storage a una nueva VM es posible cuando se usa cloud-init.

```bash
//myaccountname.file.core.windows.net/mystorageshare /mnt/mymountpoint cifs vers=3.0,username=mystorageaccount,password=myStorageAccountKeyEndingIn==,dir_mode=0777,file_mode=0777
```

Para mayor seguridad en entornos de producción, debe almacenar las credenciales fuera de fstab.

## <a name="next-steps"></a>Pasos siguientes

- [Uso de cloud-init para personalizar una VM de Linux durante la creación](using-cloud-init.md)
- [Adición de un disco a una máquina virtual de Linux](add-disk.md)
- [Azure Disk Encryption para máquinas virtuales Linux](disk-encryption-overview.md)
