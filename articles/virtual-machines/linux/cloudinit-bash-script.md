---
title: Uso de cloud-init para ejecutar un script de Bash en una máquina virtual Linux en Azure
description: Cómo usar cloud-init para ejecutar un script de bash en una máquina virtual Linux durante la creación con la CLI de Azure
author: rickstercdn
ms.service: virtual-machines-linux
ms.topic: how-to
ms.date: 11/29/2017
ms.author: rclaus
ms.openlocfilehash: 0370ddeafd1f695292ba122bfebd6197a360f560
ms.sourcegitcommit: f353fe5acd9698aa31631f38dd32790d889b4dbb
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87372814"
---
# <a name="use-cloud-init-to-run-a-bash-script-in-a-linux-vm-in-azure"></a>Uso de cloud-init para ejecutar un script de Bash en una máquina virtual Linux en Azure
En este artículo se muestra el uso de [cloud-init](https://cloudinit.readthedocs.io) para ejecutar un script ya existente de Bash en una máquina virtual Linux (VM) o en conjuntos de escalado de máquinas virtuales (VMSS) en el momento del aprovisionamiento en Azure. Estos scripts de cloud-init se ejecutan durante el primer arranque una vez que Azure ha aprovisionado los recursos. Para obtener más información acerca del funcionamiento nativo de cloud-init en Azure y las distribuciones de Linux compatibles, consulte la [introducción a cloud-init](using-cloud-init.md).

## <a name="run-a-bash-script-with-cloud-init"></a>Ejecución de un script de Bash con cloud-init
Con cloud-init no es necesario convertir los scripts existentes en un archivo cloud-config; cloud-init acepta varios tipos de entrada, uno de los cuales es los scripts de Bash.

Si se ha estado utilizando la extensión de Azure de script personalizado de Linux para ejecutar los scripts, puede migrarlos para que usen cloud-init. De todas formas, las extensiones de Azure han integrado la creación de informes para alertar sobre los errores de script, una implementación de una imagen de cloud-init NO fallará si valla el script.

Para ver esta funcionalidad en acción, cree un script de bash simple para realizar pruebas. Al igual que el archivo cloud-init `#cloud-config`, este script tiene que ser local en relación a la ubicación en la que se ejecutan los comandos AzureCLI para aprovisionar la máquina virtual.  Para este ejemplo, cree el archivo en Cloud Shell, no en la máquina local. Puede utilizar el editor que prefiera. Escriba `sensible-editor simple_bash.sh` para crear el archivo y ver una lista de editores disponibles. Elija el número 1 para utilizar el editor **nano**. Asegúrese de que todo el archivo cloud-init se copia correctamente, especialmente la primera línea.  

```bash
#!/bin/sh
echo "this has been written via cloud-init" + $(date) >> /tmp/myScript.txt
```

Antes de implementar esta imagen, debe crear un grupo de recursos con el comando [az group create](/cli/azure/group). Un grupo de recursos de Azure es un contenedor lógico en el que se implementan y se administran los recursos de Azure. En el ejemplo siguiente, se crea un grupo de recursos denominado *myResourceGroup* en la ubicación *eastus*.

```azurecli-interactive 
az group create --name myResourceGroup --location eastus
```

Ahora, cree una máquina virtual con [az vm create](/cli/azure/vm) y especifique el archivo de script bash con `--custom-data simple_bash.sh` como se indica a continuación:

```azurecli-interactive 
az vm create \
  --resource-group myResourceGroup \
  --name centos74 \
  --image OpenLogic:CentOS:7-CI:latest \
  --custom-data simple_bash.sh \
  --generate-ssh-keys 
```
## <a name="verify-bash-script-has-run"></a>Compruebe que se ha ejecutado el script de bash
SSH a la dirección IP pública de la máquina virtual que se muestra en la salida del comando anterior. Escriba su propia **dirección IP pública**, como se indica a continuación:

```bash
ssh <publicIpAddress>
```

Cambie al directorio **/tmp** y compruebe que el archivo myScript.txt existe y tiene el texto adecuado en su interior.  Si no es así, puede comprobar **/var/log/cloud-init.log** para más detalles.  Busque la siguiente entrada:

```bash
Running config-scripts-user using lock Running command ['/var/lib/cloud/instance/scripts/part-001']
```

## <a name="next-steps"></a>Pasos siguientes
Para ejemplos de cloud-init de cambios de configuración adicionales, vea lo siguiente:
 
- [Incorporación de otro usuario de Linux a una máquina virtual](cloudinit-add-user.md)
- [Ejecución de un administrador de paquetes para actualizar los existentes durante el primer arranque](cloudinit-update-vm.md)
- [Cambio del nombre de host de la máquina virtual local](cloudinit-update-vm-hostname.md) 
- [Instalación de un paquete de aplicación, actualización de los archivos de configuración e inserción de claves](tutorial-automate-vm-deployment.md)
