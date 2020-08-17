---
title: Creación y uso de un par de claves SSH para máquinas virtuales Linux en Azure
description: Creación y uso de un par de claves pública-privada SSH para máquinas virtuales Linux en Azure para mejorar la seguridad del proceso de autenticación.
author: cynthn
ms.service: virtual-machines-linux
ms.workload: infrastructure-services
ms.topic: how-to
ms.date: 12/06/2019
ms.author: cynthn
ms.openlocfilehash: 33ba816227db4cf958fd30c9dac1a0745505c504
ms.sourcegitcommit: 29400316f0c221a43aff3962d591629f0757e780
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/02/2020
ms.locfileid: "87513696"
---
# <a name="quick-steps-create-and-use-an-ssh-public-private-key-pair-for-linux-vms-in-azure"></a>Pasos rápidos: Creación y uso de un par de claves pública-privada SSH para máquinas virtuales Linux en Azure

Con un par de claves SSH puede crear una máquinas virtuales en Azure que usen claves SSH para la autenticación. En este artículo se muestra cómo generar y usar rápidamente un par de archivos de claves pública-privada SSH para máquinas virtuales Linux. Estos pasos se pueden completar con Azure Cloud Shell o un host de macOS o de Linux. 

> [!NOTE]
> De forma predeterminada, las máquinas virtuales creadas con claves SSH están configuradas con las contraseñas deshabilitadas, lo que aumenta considerablemente la dificultad para tratar de adivinarlas por fuerza bruta. 

Para más información y ejemplos, consulte los [pasos detallados para crear pares de claves SSH](create-ssh-keys-detailed.md).

Para ver otras formas de generar y usar claves SSH en un equipo con Windows, consulte [Uso de SSH con Windows en Azure](ssh-from-windows.md).

[!INCLUDE [virtual-machines-common-ssh-support](../../../includes/virtual-machines-common-ssh-support.md)]

## <a name="create-an-ssh-key-pair"></a>Creación de un par de claves SSH

Use el comando `ssh-keygen` para generar archivos de clave pública y privada SSH. De forma predeterminada, estos archivos se crean en el directorio ~/.ssh. Puede especificar una ubicación distinta y una contraseña opcional (una *frase de contraseña*) para acceder al archivo de clave privada. Si existe un par de claves SSH con el mismo nombre en la ubicación escogida, estos archivos se sobrescriben.

El siguiente comando crea un par de claves SSH con ayuda del cifrado RSA y una longitud en bits de 4096:

```bash
ssh-keygen -m PEM -t rsa -b 4096
```

Si usa la [CLI de Azure](/cli/azure) para crear la máquina virtual con el comando [az vm create](/cli/azure/vm#az-vm-create), tiene la opción de generar archivos de clave pública y privada SSH con la opción `--generate-ssh-keys`. Los archivos de claves se almacenan en el directorio ~/.ssh a menos que se especifique lo contrario con la opción `--ssh-dest-key-path`. Si ya existe un par de claves SSH y se usa la opción `--generate-ssh-keys`, no se generará otro par de claves, sino que se usará el existente. En el siguiente comando, reemplace *VMname* y *RGname* por sus propios valores:

```azurecli
az vm create --name VMname --resource-group RGname --image UbuntuLTS --generate-ssh-keys 
```

## <a name="provide-an-ssh-public-key-when-deploying-a-vm"></a>Proporcione una clave pública SSH al implementar una máquina virtual

Si desea crear una máquina virtual con Linux que utilice claves SSH para la autenticación, especifique la clave pública SSH al crear la máquina virtual utilizando Azure Portal, la CLI de Azure, las plantillas de Resource Manager u otros métodos:

* [Creación de una máquina virtuales Linux desde Azure Portal](quick-create-portal.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)
* [Creación de una máquina virtual Linux con la CLI de Azure](quick-create-cli.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)
* [Creación de una VM Linux mediante una plantilla de Azure](create-ssh-secured-vm-from-template.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)

Si no está familiarizado con el formato de una clave pública SSH, puede mostrarla con el siguiente comando `cat`, reemplazando `~/.ssh/id_rsa.pub` por la ruta de acceso y el nombre de su propio archivo de clave pública, si es necesario:

```bash
cat ~/.ssh/id_rsa.pub
```

Normalmente, los valores de clave pública tienen el aspecto del siguiente ejemplo:

```
ssh-rsa AAAAB3NzaC1yc2EAABADAQABAAACAQC1/KanayNr+Q7ogR5mKnGpKWRBQU7F3Jjhn7utdf7Z2iUFykaYx+MInSnT3XdnBRS8KhC0IP8ptbngIaNOWd6zM8hB6UrcRTlTpwk/SuGMw1Vb40xlEFphBkVEUgBolOoANIEXriAMvlDMZsgvnMFiQ12tD/u14cxy1WNEMAftey/vX3Fgp2vEq4zHXEliY/sFZLJUJzcRUI0MOfHXAuCjg/qyqqbIuTDFyfg8k0JTtyGFEMQhbXKcuP2yGx1uw0ice62LRzr8w0mszftXyMik1PnshRXbmE2xgINYg5xo/ra3mq2imwtOKJpfdtFoMiKhJmSNHBSkK7vFTeYgg0v2cQ2+vL38lcIFX4Oh+QCzvNF/AXoDVlQtVtSqfQxRVG79Zqio5p12gHFktlfV7reCBvVIhyxc2LlYUkrq4DHzkxNY5c9OGSHXSle9YsO3F1J5ip18f6gPq4xFmo6dVoJodZm9N0YMKCkZ4k1qJDESsJBk2ujDPmQQeMjJX3FnDXYYB182ZCGQzXfzlPDC29cWVgDZEXNHuYrOLmJTmYtLZ4WkdUhLLlt5XsdoKWqlWpbegyYtGZgeZNRtOOdN6ybOPJqmYFd2qRtb4sYPniGJDOGhx4VodXAjT09omhQJpE6wlZbRWDvKC55R2d/CSPHJscEiuudb+1SG2uA/oik/WQ== username@domainname
```

Si copia y pega el contenido del archivo de clave pública para usarlo en Azure Portal o en una plantilla de Resource Manager, asegúrese de no copiar ningún espacio en blanco al final. Para copiar una clave pública en macOS, puede canalizar el archivo de clave pública a `pbcopy`. Del mismo modo, en Linux puede canalizar el archivo de clave pública a programas como `xclip`.

De forma predeterminada, la clave pública que se guarda en la máquina virtual Linux de Azure se almacena en ~/.ssh/id_rsa.pub, a menos que se especificara otra ubicación cuando creó el par de claves. Si desea usar la [CLI de Azure 2.0](/cli/azure) para crear la máquina virtual con una clave pública existente, especifique el valor de esta clave pública y, de forma opcional, su ubicación ejecutando el comando [az vm create](/cli/azure/vm#az-vm-create) con la opción `--ssh-key-values`. En el siguiente comando, reemplace *myVM*, *myResourceGroup*, *UbuntuLTS*, *azureuser* y *mysshkey.pub* con sus propios valores:


```azurecli
az vm create \
  --resource-group myResourceGroup \
  --name myVM \
  --image UbuntuLTS \
  --admin-username azureuser \
  --ssh-key-values mysshkey.pub
```

Si desea usar varias claves SSH con la máquina virtual, puede escribirlas en una lista separada por espacios, como esta `--ssh-key-values sshkey-desktop.pub sshkey-laptop.pub`.


## <a name="ssh-into-your-vm"></a>Conexión SSH con la máquina virtual

Con la clave pública implementada en la máquina virtual de Azure y la clave privada en el sistema local, aplique SSH en la máquina virtual con la dirección IP o el nombre DNS de la máquina virtual. En el comando siguiente, reemplace *azureuser* y *myvm.westus.cloudapp.azure.com* por el nombre de usuario del administrador y el nombre de dominio completo (o la dirección IP):

```bash
ssh azureuser@myvm.westus.cloudapp.azure.com
```

Si especificó una frase de contraseña al crear el par de claves, escríbala cuando se lo soliciten durante el proceso de inicio de sesión. La máquina virtual se agrega al archivo ~/.ssh/known_hosts y no se le pedirá que se conecte de nuevo hasta que se modifique la clave pública de la máquina virtual de Azure o se quite el nombre del servidor de ~/.ssh/known_hosts.

Si la máquina virtual está usando la directiva de acceso Just-In-Time, deberá solicitar acceso antes de poder conectarse a la máquina virtual. Para más información sobre la directiva Just-in-Time, vea [Administración del acceso a máquina virtual mediante Just-In-Time](../../security-center/security-center-just-in-time.md).

## <a name="next-steps"></a>Pasos siguientes

* Para más información acerca de cómo trabajar con pares de claves SSH, consulte los [pasos detallados para crear y administrar pares de claves SSH](create-ssh-keys-detailed.md).

* Si tiene problemas en las conexiones SSH con una máquina virtual de Azure, consulte [Solución de problemas de conexiones SSH con una máquina virtual Linux de Azure](../troubleshooting/troubleshoot-ssh-connection.md).
