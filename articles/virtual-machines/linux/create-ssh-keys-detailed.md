---
title: Pasos detallados para crear un par de claves de SSH
description: Obtenga información acerca de los pasos detallados para crear y administrar un par de claves SSH pública y privada para VM Linux en Azure.
author: cynthn
ms.service: virtual-machines-linux
ms.topic: how-to
ms.date: 07/31/2020
ms.author: cynthn
ms.openlocfilehash: 34a84ed333172ea0931c529d2dbeee1b774ae8c5
ms.sourcegitcommit: 29400316f0c221a43aff3962d591629f0757e780
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/02/2020
ms.locfileid: "87513201"
---
# <a name="detailed-steps-create-and-manage-ssh-keys-for-authentication-to-a-linux-vm-in-azure"></a>Pasos detallados: crear y administrar claves SSH para la autenticación en una VM Linux en Azure

Con un par de claves SSH puede crear una máquina virtual Linux que use claves SSH para la autenticación. En este artículo se muestra cómo crear y usar rápidamente un par de archivos de claves pública-privada RSA SSH para conexiones cliente SSH.

Si busca comandos rápidos, consulte [Creación de un par de claves SSH pública y privada para VM Linux en Azure](mac-create-ssh-keys.md).

Para crear claves SSH y usarlas para conectarse desde un equipo **Windows**, consulte [Uso de claves SSH con Windows en Azure](ssh-from-windows.md). También puede usar [Azure Portal](../ssh-keys-portal.md) para crear y administrar claves SSH para crear máquinas virtuales en el portal.

[!INCLUDE [virtual-machines-common-ssh-overview](../../../includes/virtual-machines-common-ssh-overview.md)]

[!INCLUDE [virtual-machines-common-ssh-support](../../../includes/virtual-machines-common-ssh-support.md)]

## <a name="ssh-keys-use-and-benefits"></a>Uso y ventajas de las claves SSH

Cuando se crea una VM de Azure mediante la especificación de la clave pública, Azure copia la clave pública (en el formato `.pub`) a la carpeta `~/.ssh/authorized_keys` en la VM. Las claves SSH en `~/.ssh/authorized_keys` se utilizan para presentar un desafío al cliente, que debe proporcionar la clave privada correspondiente en una conexión de inicio de sesión SSH. En una VM Linux de Azure que usa claves SSH para la autenticación, Azure configura el servidor SSHD para no permitir inicios de sesión con contraseña, solo con claves SSH. Mediante la creación una máquina virtual Linux de Azure con claves SSH, puede ayudar a proteger la implementación de la máquina virtual y ahorrarse el paso de configuración que es habitual después de la implementación para deshabilitar las contraseñas en el archivo `sshd_config`.

Si no quiere usar claves de SSH, puede configurar la VM Linux para que use la autenticación de contraseña. Si la máquina virtual no está expuesta a Internet, el uso de contraseñas puede ser suficiente. Aun así, es preciso que administre las contraseñas de cada VM Linux y que mantenga directivas y procedimientos seguros con respecto a las contraseñas, como la elección contraseñas con una longitud mínima y actualizaciones periódicas. 

## <a name="generate-keys-with-ssh-keygen"></a>Generar claves con ssh-keygen

Para crear las claves, se recomienda el comando `ssh-keygen`, disponible con las utilidades de OpenSSH en Azure Cloud Shell, un host de macOS o Linux y Windows 10. `ssh-keygen` realiza una serie de preguntas y, después, escribe una clave privada y una clave pública correspondiente. 

Las claves SSH se mantienen de forma predeterminada en el directorio `~/.ssh`.  Si no tiene un directorio `~/.ssh`, el comando `ssh-keygen` lo crea automáticamente con los permisos correctos.

### <a name="basic-example"></a>Ejemplo básico

El siguiente comando `ssh-keygen` genera archivos RSA de SSH de clave pública y privada de 4096 bits de forma predeterminada en el directorio `~/.ssh`. Si existe un par de claves SSH en la ubicación actual, esos archivos se sobrescribirán.

```bash
ssh-keygen -m PEM -t rsa -b 4096
```

### <a name="detailed-example"></a>Ejemplo detallado
En el siguiente ejemplo, se muestran las opciones de comando adicionales para crear un par de claves RSA de SSH. Si existe un par de claves SSH en la ubicación actual, esos archivos se sobrescribirán. 

```bash
ssh-keygen \
    -m PEM \
    -t rsa \
    -b 4096 \
    -C "azureuser@myserver" \
    -f ~/.ssh/mykeys/myprivatekey \
    -N mypassphrase
```

**Explicación del comando**

`ssh-keygen` = programa usado para crear las claves;

`-m PEM` = formatea la clave como PEM

`-t rsa` = tipo de clave que se va a crear; en este caso, en el formato RSA

`-b 4096` = número de bits de la clave; en este caso, 4096

`-C "azureuser@myserver"` = comentario que se anexa al final del archivo de clave pública para identificarlo fácilmente. Normalmente, se usa una dirección de correo electrónico como comentario, pero utilice lo que funcione mejor para su infraestructura.

`-f ~/.ssh/mykeys/myprivatekey` = nombre del archivo de clave privada, si decide no usar el nombre predeterminado. Se genera un archivo de clave pública correspondiente anexo a `.pub` en el mismo directorio. El directorio debe existir.

`-N mypassphrase` = frase de contraseña adicional que se usa para acceder al archivo de clave privada. 

### <a name="example-of-ssh-keygen"></a>Ejemplo de ssh-keygen

```bash
ssh-keygen -t -m PEM rsa -b 4096 -C "azureuser@myserver"
Generating public/private rsa key pair.
Enter file in which to save the key (/home/azureuser/.ssh/id_rsa):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/azureuser/.ssh/id_rsa.
Your public key has been saved in /home/azureuser/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:vFfHHrpSGQBd/oNdvNiX0sG9Vh+wROlZBktNZw9AUjA azureuser@myserver
The key's randomart image is:
+---[RSA 4096]----+
|        .oE=*B*+ |
|          o+o.*++|
|           .oo++*|
|       .    .B+.O|
|        S   o=BO.|
|         . .o++o |
|        . ... .  |
|         ..  .   |
|           ..    |
+----[SHA256]-----+
```

#### <a name="saved-key-files"></a>Archivos de clave guardados

`Enter file in which to save the key (/home/azureuser/.ssh/id_rsa): ~/.ssh/id_rsa`

El nombre del par de claves en este artículo. Tener un par de claves denominado `id_rsa` es la opción predeterminada y algunas herramientas podrían esperar `id_rsa` como nombre del archivo de la clave privada, así que es buena idea tener uno. El directorio `~/.ssh/` es la ubicación predeterminada para los pares de claves SSH y el archivo de configuración de SSH. Si no se especifica con una ruta de acceso completa, `ssh-keygen` creará las claves en el directorio de trabajo actual, no en el predeterminado `~/.ssh`.

#### <a name="list-of-the-ssh-directory"></a>Lista del directorio `~/.ssh`

```bash
ls -al ~/.ssh
-rw------- 1 azureuser staff  1675 Aug 25 18:04 id_rsa
-rw-r--r-- 1 azureuser staff   410 Aug 25 18:04 id_rsa.pub
```

#### <a name="key-passphrase"></a>Frase de contraseña de la clave

`Enter passphrase (empty for no passphrase):`

Es *muy* recomendable agregar una frase de contraseña a las claves privadas. Sin una frase de contraseña que proteja el archivo de claves, cualquiera que disponga del archivo puede usarlo para iniciar sesión en cualquier servidor que tenga la clave pública correspondiente. El uso de una frase de contraseña ofrece más protección en caso de que alguien obtenga acceso al archivo de clave privada, ya que le daría tiempo para cambiar las claves.

## <a name="generate-keys-automatically-during-deployment"></a>Generar claves automáticamente durante la implementación

Si usa la [CLI de Azure](/cli/azure) para crear la máquina virtual, de manera opcional, se pueden generar archivos de claves SSH pública y privada mediante la ejecución del comando [az vm create](/cli/azure/vm) con la opción `--generate-ssh-keys`. Las claves se almacenan en el directorio ~/.ssh. Tenga en cuenta que esta opción de comando no sobrescribe las claves si ya existen en esa ubicación.

## <a name="provide-ssh-public-key-when-deploying-a-vm"></a>Proporcione la clave pública SSH al implementar una VM

Para crear una VM Linux que use claves SSH para la autenticación, proporcione la clave pública SSH al crear la VM mediante el portal de Azure, CLI, plantillas de Resource Manager u otros métodos. Al usar el portal, escriba propia la clave pública. Si usa la [CLI de Azure](/cli/azure) para crear la máquina virtual con una clave pública existente, especifique el valor o la ubicación de esta clave pública mediante la ejecución del comando [az vm create](/cli/azure/vm) con la opción `--ssh-key-value`. 

Si no está familiarizado con el formato de una clave pública SSH, puede ver su clave pública si ejecuta `cat` de la siguiente manera y reemplaza `~/.ssh/id_rsa.pub` por la ubicación de su propio archivo de clave pública:

```bash
cat ~/.ssh/id_rsa.pub
```

Devuelve una salida similar a la siguiente (que se muestra aquí):

```
ssh-rsa XXXXXXXXXXc2EAAAADAXABAAABAXC5Am7+fGZ+5zXBGgXS6GUvmsXCLGc7tX7/rViXk3+eShZzaXnt75gUmT1I2f75zFn2hlAIDGKWf4g12KWcZxy81TniUOTjUsVlwPymXUXxESL/UfJKfbdstBhTOdy5EG9rYWA0K43SJmwPhH28BpoLfXXXXXG+/ilsXXXXXKgRLiJ2W19MzXHp8z3Lxw7r9wx3HaVlP4XiFv9U4hGcp8RMI1MP1nNesFlOBpG4pV2bJRBTXNXeY4l6F8WZ3C4kuf8XxOo08mXaTpvZ3T1841altmNTZCcPkXuMrBjYSJbA8npoXAXNwiivyoe3X2KMXXXXXdXXXXXXXXXXCXXXXX/ azureuser@myserver
```

Si copia y pega el contenido del archivo de clave pública en Azure Portal o en una plantilla de Resource Manager, asegúrese de no copiar ningún espacio en blanco adicional y de no introducir saltos de línea adicionales. Por ejemplo, si utiliza macOS, puede canalizar el archivo de clave pública (de forma predeterminada, `~/.ssh/id_rsa.pub`) hacia **pbcopy** para copiar el contenido (hay otros programas de Linux que hacen lo mismo, como `xclip`).

Si prefiere usar una clave pública que se encuentra en un formato de varias líneas, puede generar una clave con formato RFC4716 en un contenedor de pem de la clave pública que creó anteriormente.

Para crear una clave con formato RFC4716 a partir de una clave pública SSH existente:

```bash
ssh-keygen \
-f ~/.ssh/id_rsa.pub \
-e \
-m RFC4716 > ~/.ssh/id_ssh2.pem
```

## <a name="ssh-to-your-vm-with-an-ssh-client"></a>SSH en la VM con un cliente SSH
Con la clave pública implementada en la VM de Azure y la clave privada en su sistema local, aplique SSH en la VM con la dirección IP o nombre DNS de la VM. Reemplace *azureuser* y *myvm.westus.cloudapp.azure.com* en el siguiente comando con el nombre de usuario administrador y el nombre de dominio completo (o dirección IP):

```bash
ssh azureuser@myvm.westus.cloudapp.azure.com
```

Si proporcionó una frase de contraseña cuando creó el par de claves, escríbala cuando se le solicite durante el proceso de inicio de sesión. (El servidor se agrega a la carpeta `~/.ssh/known_hosts` y no se le pedirá que se conecte de nuevo hasta que la clave pública de la máquina virtual de Azure cambie o se quite el nombre del servidor de `~/.ssh/known_hosts`).

Si la máquina virtual está usando la directiva de acceso Just-In-Time, deberá solicitar acceso antes de poder conectarse a la máquina virtual. Para más información sobre la directiva Just-in-Time, vea [Administración del acceso a máquina virtual mediante Just-In-Time](../../security-center/security-center-just-in-time.md).

## <a name="use-ssh-agent-to-store-your-private-key-passphrase"></a>Use ssh-agent para almacenar la frase de contraseña de clave privada

Para no tener que escribir la frase de contraseña del archivo de clave privada con cada inicio de sesión SSH, puede usar `ssh-agent` para almacenar en caché la frase de contraseña del archivo de clave privada. Si usa Mac, la aplicación Keychain de macOS almacena de forma segura las frases de contraseña de clave privada al invocar `ssh-agent`.

Compruebe y use `ssh-agent` y `ssh-add` para informar al sistema SSH acerca de los archivos de clave para que no sea necesario usar la frase de contraseña de forma interactiva.

```bash
eval "$(ssh-agent -s)"
```

Ahora, agregue la clave privada a `ssh-agent` mediante el comando `ssh-add`.

```bash
ssh-add ~/.ssh/id_rsa
```

La frase de contraseña de clave privada está almacenada ahora en `ssh-agent`.

## <a name="use-ssh-copy-id-to-copy-the-key-to-an-existing-vm"></a>Use ssh-copy-id para copiar la clave en una VM existente

Si ya ha creado una máquina virtual, puede agregar una clave pública SSH nueva a su máquina virtual Linux mediante `ssh-copy-id`.

```bash
ssh-copy-id -i ~/.ssh/id_rsa.pub azureuser@myserver
```

## <a name="create-and-configure-an-ssh-config-file"></a>Creación y configuración de un archivo de configuración de SSH

Puede crear y configurar un archivo de configuración SSH (`~/.ssh/config`) para acelerar los inicios de sesión y optimizar el comportamiento del cliente SSH. 

En el siguiente ejemplo se muestra una configuración simple que puede utilizar para iniciar sesión rápidamente como un usuario en una VM específica mediante la clave privada de SSH de forma predeterminada. 

Cree el archivo.

```bash
touch ~/.ssh/config
```

Edite el archivo para agregar la nueva configuración de SSH

```bash
vim ~/.ssh/config
```

Agregue valores de configuración adecuados para la VM host. En este ejemplo, el nombre de la máquina virtual es *myvm* y el nombre de la cuenta es *azureuser*.

```bash
# Azure Keys
Host myvm
  Hostname 102.160.203.241
  User azureuser
# ./Azure Keys
```

Puede agregar configuraciones para hosts adicionales para permitir que cada uno use su propio par de claves dedicado. Vea [archivo de configuración SSH](https://www.ssh.com/ssh/config/) para obtener más opciones de configuración avanzada.

Ahora que tiene un par de claves SSH y un archivo de configuración de SSH configurado, puede iniciar sesión en la máquina virtual Linux de forma rápida y segura. Cuando ejecute el siguiente comando, SSH busca y carga los valores de configuración del bloque `Host myvm` en el archivo de configuración SSH.

```bash
ssh myvm
```

La primera vez que inicie sesión en un servidor mediante una clave SSH, el símbolo del sistema le pedirá la frase de contraseña para ese archivo de clave.

## <a name="next-steps"></a>Pasos siguientes

El siguiente paso es crear máquinas virtuales de Linux en Azure con la nueva clave pública SSH. Las máquinas virtuales de Azure que se crean con una clave pública SSH como inicio de sesión están mejor protegidas que las creadas con contraseñas, el método de inicio de sesión predeterminado.

* [Creación de una máquina virtuales Linux desde Azure Portal](quick-create-portal.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)
* [Creación de una máquina virtual Linux con la CLI de Azure](quick-create-cli.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)
* [Creación de una VM Linux mediante una plantilla de Azure](create-ssh-secured-vm-from-template.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)
