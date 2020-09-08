---
title: Uso de claves SSH para conectarse a máquinas virtuales Linux
description: Obtenga información acerca de cómo generar y utilizar claves SSH en un equipo de Windows para conectarse a una máquina virtual Linux en Azure.
author: cynthn
ms.service: virtual-machines
ms.workload: infrastructure-services
ms.date: 07/09/2020
ms.topic: how-to
ms.author: cynthn
ms.openlocfilehash: 7e99c9191e93562211f6294cf671f431a5db455d
ms.sourcegitcommit: 2ff0d073607bc746ffc638a84bb026d1705e543e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87825572"
---
# <a name="how-to-use-ssh-keys-with-windows-on-azure"></a>Uso de claves SSH con Windows en Azure

Este artículo está destinado a los usuarios de Windows que desean [crear](#create-an-ssh-key-pair) y usar claves de *shell seguro (SSH)* para [conectar](#connect-to-your-vm) con máquinas virtuales Linux en Azure. También puede [generar y almacenar claves SSH en Azure Portal](../ssh-keys-portal.md) que se van a usar al crear máquinas virtuales en el portal.


Para usar las claves SSH desde un cliente Linux o macOS, consulte la [guía rápida](mac-create-ssh-keys.md). Para más información sobre SSH, consulte [Pasos detallados: crear y administrar claves SSH para la autenticación en una VM Linux en Azure](create-ssh-keys-detailed.md).

## <a name="overview-of-ssh-and-keys"></a>Información general sobre SSH y sus claves

[SSH](https://www.ssh.com/ssh/) es un protocolo de conexión cifrada que permite inicios de sesión seguros a través de conexiones no seguras. SSH es el protocolo de conexión predeterminado de las máquinas virtuales Linux hospedadas en Azure. Aunque el propio SSH proporciona una conexión cifrada, el uso de contraseñas con SSH deja la máquina virtual vulnerable a ataques por fuerza bruta. Se recomienda conectarse a una máquina virtual a través de SSH mediante un par de claves pública y privada, también conocidas como *claves SSH*. 

El par de claves pública y privada es como el bloqueo de la puerta principal. El bloqueo se expone al **público**, es decir, cualquiera que tenga la clave correcta puede abrir la puerta. La clave es **privada** y solo se proporciona a las personas en las que confía porque se puede usar para desbloquear la puerta. 

- El *clave pública* se coloca en la máquina virtual Linux al crear la máquina virtual. 

- La *clave privada* permanece en el sistema local. Esta clave se debe proteger, por lo que no se debe compartir.

Cuando se conecta a la máquina virtual Linux, la máquina virtual prueba el cliente SSH para asegurarse de tener la clave privada correcta. Si es el caso, se le concede acceso a la máquina virtual. 

En función de las directivas de seguridad de la organización, puede volver a usar un par de claves único para obtener acceso a varias máquinas virtuales y servicios de Azure. No es necesario usar ningún par de claves independiente para cada máquina virtual. 

La clave pública se puede compartir con cualquiera. Sin embargo, solo usted (o su infraestructura de seguridad local) debe tener acceso a la clave privada.

[!INCLUDE [virtual-machines-common-ssh-support](../../../includes/virtual-machines-common-ssh-support.md)]

## <a name="ssh-clients"></a>Clientes SSH

Las versiones más recientes de Windows 10 incluyen [comandos del cliente OpenSSH](https://blogs.msdn.microsoft.com/commandline/2018/03/07/windows10v1803/) para crear y administrar claves SSH y establecer conexiones SSH desde PowerShell o un símbolo del sistema. Esta es la manera más fácil de crear una conexión SSH a la máquina virtual Linux desde un equipo Windows. 

También puede usar Bash en [Azure Cloud Shell](../../cloud-shell/overview.md) para conectarse a la máquina virtual. Puede usar Cloud Shell en un [explorador web](https://shell.azure.com/bash), desde [Azure Portal](https://portal.azure.com)o como un terminal en Visual Studio Code con la [extensión de cuenta de Azure](https://marketplace.visualstudio.com/items?itemName=ms-vscode.azure-account).

También puede instalar el [Subsistema de Windows para Linux](/windows/wsl/about) para conectarse a la máquina virtual a través de SSH y usar otras herramientas nativas de Linux en un shell de Bash.

## <a name="create-an-ssh-key-pair"></a>Creación de un par de claves SSH

Cree un par de claves SSH con el comando `ssh-keygen`. Escriba un nombre de archivo o use el valor predeterminado que se muestra entre paréntesis (por ejemplo `C:\Users\username/.ssh/id_rsa`).  Escriba una frase de contraseña para el archivo o deje la frase de contraseña en blanco si no desea usar una frase de contraseña. 

```powershell
ssh-keygen -m PEM -t rsa -b 4096
```

## <a name="create-a-vm-using-your-key"></a>Creación de una máquina virtual con su propia clave

Para crear una máquina virtual Linux que use claves SSH para la autenticación, proporcione la clave pública SSH al crear la máquina virtual.

Con la CLI de Azure, especifique la ruta de acceso y el nombre de archivo de la clave pública con `az vm create` y el parámetro `--ssh-key-value`.

```azurecli
az vm create \
   --resource-group myResourceGroup \
   --name myVM \
   --image UbuntuLTS\
   --admin-username azureuser \
   --ssh-key-value ~/.ssh/id_rsa.pub
```

Con PowerShell, use `New-AzVM` y agregue la clave SSH a la configuración de máquina virtual mediante '. Para ver un ejemplo, consulte [Inicio rápido: Creación de una máquina virtual Linux en Azure con PowerShell](quick-create-powershell.md)

Si realiza muchas implementaciones con el portal, es posible que quiera cargar la clave pública en Azure, donde se puede seleccionar fácilmente al crear una máquina virtual desde el portal. Para más información, consulte [Carga de archivos en una cuenta de Media Services en Azure Portal](../ssh-keys-portal.md#upload-an-ssh-key).


## <a name="connect-to-your-vm"></a>Conexión a la máquina virtual

Con la clave pública implementada en la VM de Azure y la clave privada en su sistema local, aplique SSH en la VM con la dirección IP o nombre DNS de la VM. Reemplace *azureuser* y *10.111.12.123* en el siguiente comando con el nombre de usuario administrador, la dirección IP (o el nombre de dominio completo) y la ruta de acceso a la clave privada:

```bash
ssh -i ~/.ssh/id_rsa.pub azureuser@10.111.12.123
```

Si configuró una frase de contraseña cuando creó el par de claves, escríbala cuando se le solicite.

Si la máquina virtual está usando la directiva de acceso Just-In-Time, deberá solicitar acceso antes de poder conectarse a la máquina virtual. Para más información sobre la directiva Just-in-Time, vea [Administración del acceso a máquina virtual mediante Just-In-Time](../../security-center/security-center-just-in-time.md).


## <a name="next-steps"></a>Pasos siguientes

- Para más información sobre las claves SSH en Azure Portal, consulte [Generar y almacenar claves SSH en Azure Portal](../ssh-keys-portal.md) que se va a usar al crear máquinas virtuales en el portal.

- Para obtener pasos detallados, opciones y ejemplos avanzados del uso de claves SSH, consulte los [pasos detallados para crear pares de claves SSH](create-ssh-keys-detailed.md).

- También puede usar Powershell en Azure Cloud Shell para generar claves SSH y establecer conexiones SSH a máquinas virtuales Linux. Consulte [Inicio rápido de PowerShell](../../cloud-shell/quickstart-powershell.md#ssh).

- Si tiene problemas al usar SSH para conectarse a máquinas virtuales Linux, consulte [Troubleshoot SSH connections to an Azure Linux VM](../troubleshooting/troubleshoot-ssh-connection.md?toc=/azure/virtual-machines/linux/toc.json) (Solución de problemas de conexiones SSH a una máquina virtual Linux de Azure).