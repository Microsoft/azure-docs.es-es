---
title: Uso de cloud-init para agregar un usuario a una máquina virtual Linux en Azure
description: Procedimiento para usar cloud-init con el fin de agrear un usuario a una máquina virtual Linux con la CLI de Azure
author: rickstercdn
ms.service: virtual-machines-linux
ms.topic: how-to
ms.date: 11/29/2017
ms.author: rclaus
ms.openlocfilehash: 569ceb4c7158ba9dc08c99c234951fb4507b69f6
ms.sourcegitcommit: f353fe5acd9698aa31631f38dd32790d889b4dbb
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87370077"
---
# <a name="use-cloud-init-to-add-a-user-to-a-linux-vm-in-azure"></a>Uso de cloud-init para agregar a un usuario a una máquina virtual Linux en Azure
En este artículo se muestra el uso de [cloud-init](https://cloudinit.readthedocs.io) para agrear a un usuario en una máquina virtual (VM) o en conjuntos de escalado de máquinas virtuales (VMSS) en el momento del aprovisionamiento en Azure. Este script cloud-init se ejecuta durante el primer arranque una vez que Azure ha aprovisionado los recursos. Para más información acerca del funcionamiento nativo de cloud-init en Azure y las distribuciones de Linux compatibles, consulte la [introducción a cloud-init](using-cloud-init.md).

## <a name="add-a-user-to-a-vm-with-cloud-init"></a>Incorporación de un usuario a una máquina virtual con cloud-init
Una de las primeras tareas en cualquier máquina virtual Linux nueva es agregar un usuario para usted mismo a fin de evitar el uso de *root*. Las claves SSH son un procedimiento recomendado de seguridad y facilidad de uso. Las claves se agregan al archivo *~/.ssh/authorized_keys* con este script cloud-init.

Para agregar un usuario a una máquina virtual Linux, cree un archivo en el shell actual denominado *cloud_init_add_user.txt* y pegue la configuración siguiente. Para este ejemplo, cree el archivo en Cloud Shell, no en la máquina local. Puede utilizar el editor que prefiera. Escriba `sensible-editor cloud_init_add_user.txt` para crear el archivo y ver una lista de editores disponibles. Elija el número 1 para utilizar el editor **nano**. Asegúrese de que todo el archivo cloud-init se copia correctamente, especialmente la primera línea.  Tiene que proporcionar su propia clave pública (por ejemplo, el contenido de *~/.ssh/id_rsa.pub*) para el valor de `ssh-authorized-keys:`, aquí se ha abreviado para simplificar el ejemplo.

```yaml
#cloud-config
users:
  - default
  - name: myadminuser
    groups: sudo
    shell: /bin/bash
    sudo: ['ALL=(ALL) NOPASSWD:ALL']
    ssh-authorized-keys:
      - ssh-rsa AAAAB3<snip>
```
> [!NOTE] 
> El archivo #cloud-config incluye el parámetro `- default` incluido. Esto anexará al usuario al usuario administrador existente creado durante el aprovisionamiento. Si crea un usuario sin el parámetro `- default`, el usuario de administrador generado automáticamente que creó la plataforma Azure se sobrescribirá. 

Antes de implementar esta imagen, debe crear un grupo de recursos con el comando [az group create](/cli/azure/group). Un grupo de recursos de Azure es un contenedor lógico en el que se implementan y se administran los recursos de Azure. En el ejemplo siguiente, se crea un grupo de recursos denominado *myResourceGroup* en la ubicación *eastus*.

```azurecli-interactive 
az group create --name myResourceGroup --location eastus
```

Ahora, cree una máquina virtual con [az vm create](/cli/azure/vm) y especifique el archivo cloud-init con `--custom-data cloud_init_add_user.txt` como se indica a continuación:

```azurecli-interactive 
az vm create \
  --resource-group myResourceGroup \
  --name centos74 \
  --image OpenLogic:CentOS:7-CI:latest \
  --custom-data cloud_init_add_user.txt \
  --generate-ssh-keys 
```

SSH a la dirección IP pública de la máquina virtual que se muestra en la salida del comando anterior. Escriba su propia **dirección IP pública**, como se indica a continuación:

```bash
ssh <publicIpAddress>
```

Para confirmar que se agregó el usuario a la máquina virtual y los grupos especificados, consulte el contenido del archivo */etc/group* como se indica a continuación:

```bash
cat /etc/group
```

En la salida de ejemplo siguiente se muestra que el usuario del archivo *cloud_init_add_user.txt* se agregó a la máquina virtual y al grupo adecuado:

```bash
root:x:0:
<snip />
sudo:x:27:myadminuser
<snip />
myadminuser:x:1000:
```

## <a name="next-steps"></a>Pasos siguientes
Para ejemplos de cloud-init de cambios de configuración adicionales, vea lo siguiente:
 
- [Incorporación de otro usuario de Linux a una máquina virtual](cloudinit-add-user.md)
- [Ejecución de un administrador de paquetes para actualizar los existentes durante el primer arranque](cloudinit-update-vm.md)
- [Cambio del nombre de host de la máquina virtual local](cloudinit-update-vm-hostname.md) 
- [Instalación de un paquete de aplicación, actualización de los archivos de configuración e inserción de claves](tutorial-automate-vm-deployment.md)
