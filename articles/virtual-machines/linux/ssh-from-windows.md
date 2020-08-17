---
title: Uso de claves SSH con Windows para máquinas virtuales Linux
description: Obtenga información acerca de cómo generar y utilizar claves SSH en un equipo de Windows para conectarse a una máquina virtual con Linux en Azure.
author: cynthn
ms.service: virtual-machines-linux
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-linux
ms.topic: how-to
ms.date: 11/26/2018
ms.author: cynthn
ms.openlocfilehash: 4e47b303548707074f51e5f3dd5f8840b3e7b5ed
ms.sourcegitcommit: f353fe5acd9698aa31631f38dd32790d889b4dbb
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87372236"
---
# <a name="how-to-use-ssh-keys-with-windows-on-azure"></a>Uso de claves SSH con Windows en Azure

En este artículo se describen las formas de generar y utilizar claves de *Secure Shell* (SSH) en un equipo Windows para crear y conectarse a una máquina virtual (VM) Linux en Azure. Para usar las claves SSH desde un cliente Linux o macOS, consulte la guía [rápida](mac-create-ssh-keys.md) o [detallada](create-ssh-keys-detailed.md).

[!INCLUDE [virtual-machines-common-ssh-overview](../../../includes/virtual-machines-common-ssh-overview.md)]

[!INCLUDE [virtual-machines-common-ssh-support](../../../includes/virtual-machines-common-ssh-support.md)]

## <a name="windows-packages-and-ssh-clients"></a>Paquetes de Windows y los clientes SSH
Para conectarse a máquinas virtuales Linux de Azure y administraras, se usa un cliente *ssh*. Los equipos que ejecutan Linux o macOS suelen tener un conjunto de comandos SSH para generar y administrar claves SSH, y para establecer conexiones SSH. 

Los equipos Windows no siempre tienen comandos SSH comparables instalados. Las versiones más recientes de Windows 10 proporcionan [comandos del cliente OpenSSH](https://devblogs.microsoft.com/commandline/windows10v1803/) para crear y administrar claves SSH y establecer conexiones SSH desde un símbolo del sistema. Las versiones recientes de Windows 10 también incluyen el [subsistema de Windows para Linux](/windows/wsl/about) para ejecutar utilidades y acceder a ellas, como un cliente SSH nativo en el shell de Bash. 

Otros clientes SSH de Windows comunes que puede instalar se encuentran de forma local en los siguientes paquetes:

* [PuTTY](https://www.chiark.greenend.org.uk/~sgtatham/putty/)
* [Git para Windows](https://git-for-windows.github.io/)
* [MobaXterm](https://mobaxterm.mobatek.net/)
* [Cygwin](https://cygwin.com/)

Asimismo, puede usar las utilidades SSH disponibles en Bash en [Azure Cloud Shell](../../cloud-shell/overview.md). 

* Acceda a Cloud Shell en el explorador web en [https://shell.azure.com](https://shell.azure.com) o en [Azure Portal](https://portal.azure.com). 
* Acceda a Cloud Shell como terminal desde Visual Studio Code mediante la instalación de la [extensión Cuenta de Azure](https://marketplace.visualstudio.com/items?itemName=ms-vscode.azure-account).

## <a name="create-an-ssh-key-pair"></a>Creación de un par de claves SSH
En las secciones siguientes se muestran dos opciones para crear un par de claves SSH en Windows. Puede usar un comando de shell (`ssh-keygen`) o una herramienta de GUI (PuTTYgen). Tenga en cuenta también que, al usar PowerShell para crear una clave, se carga la clave pública en formato ssh.com(SECSH). Cuando use la CLI, convierta la clave en formato OpenSSH antes de la carga. 

### <a name="create-ssh-keys-with-ssh-keygen"></a>Crear claves SSH con ssh-keygen

Si ejecuta un shell de comandos en Windows que admita las herramientas de cliente SSH (o usa Azure Cloud Shell), cree un par de claves SSH mediante el comando `ssh-keygen`. Escribe el siguiente comando y responda a los mensajes. Si existe un par de claves SSH en la ubicación escogida, esos archivos se sobrescribirán. 

```bash
ssh-keygen -t rsa -b 2048
```

Para ver otras consideraciones y obtener información, vea los pasos [rápidos](mac-create-ssh-keys.md) o [detallados](create-ssh-keys-detailed.md) para crear claves SSH con `ssh-keygen`.

### <a name="create-ssh-keys-with-puttygen"></a>Crear claves SSH con PuTTYgen

Si prefiere usar una herramienta basada en GUI para crear claves SSH, puede usar el generador de claves PuTTYgen, incluido con el [paquete de descarga de PuTTY](https://www.chiark.greenend.org.uk/~sgtatham/putty/download.html). 

Para crear un par de claves SSH RSA con PuTTYgen:

1. Inicie PuTTYgen.

2. Haga clic en **Generar**. De forma predeterminada, PuTTYgen genera una clave SSH-2 RSA de 2048 bits.

3. Mueva el mouse en el área en blanco para que la clave sea aleatoria.

4. Una vez generada la clave pública, tiene la opción de escribir y confirmar una frase de contraseña. Se le pedirá la frase de contraseña cuando se autentique en la máquina virtual con la clave SSH. Sin una frase de contraseña, si alguien obtiene su clave privada, podrá iniciar sesión en cualquier máquina virtual o servicio que utilice dicha clave. Por consiguiente, se recomienda crear una frase de contraseña. Sin embargo, si se le olvida la frase de contraseña, no hay forma de recuperarla.

5. La clave pública se muestra en la parte superior de la ventana. Cuando cree una máquina virtual Linux, copie la clave pública completa y péguela en Azure Portal o en una plantilla de Azure Resource Manager. También puede hacer clic en **Save public key** (Guardar clave pública) para guardar una copia en el equipo. Tenga en cuenta que, cuando se guarda en un archivo, PuTTy convierte la clave pública a un formato diferente, [RFC4716](https://tools.ietf.org/html/rfc4716). El formato RFC4716 puede no ser compatible con todas las API. Por tanto, para usarla en Azure Portal, se recomienda copiar la clave pública mostrada en la ventana de PuTTY.

    ![Guardar archivo de clave pública de PuTTY](./media/ssh-from-windows/save-public-key.png)

6. De manera opcional, para guardar la clave privada con el formato de clave privada de PuTTy (archivo .ppk), haga clic en **Save private key** (Guardar clave privada). Necesitará el archivo .ppk si quiere usar PuTTY más adelante para establecer una conexión SSH con la máquina virtual.

    ![Guardar el archivo de clave privada de PuTTY](./media/ssh-from-windows/save-ppk-file.png)

    Si quiere guardar la clave privada con el formato de OpenSSH (este es el formato de clave privada que usan muchos clientes de SSH), seleccione **Conversiones** > **Export OpenSSH key** (Exportar clave OpenSSH).

## <a name="provide-an-ssh-public-key-when-deploying-a-vm"></a>Proporcione una clave pública SSH al implementar una máquina virtual

Para crear una VM Linux que use claves SSH para la autenticación, proporcione la clave pública SSH al crear la máquina virtual mediante Azure Portal u otros métodos.

En el siguiente ejemplo se muestra cómo se copia y pega esta clave pública en Azure Portal al crear una máquina virtual Linux. La clave pública se almacena normalmente en el directorio ~/.ssh/authorized_key, en la nueva máquina virtual.

   ![Usar la clave pública al crear una máquina virtual en Azure Portal](./media/ssh-from-windows/use-public-key-azure-portal.png)


## <a name="connect-to-your-vm"></a>Conexión a la máquina virtual

Una manera de establecer una conexión SSH a la máquina virtual Linux desde Windows es usar un cliente SSH. Este es el método que se prefiere si tiene un cliente SSH instalado en el sistema Windows o si usa herramientas SSH en Bash en Azure Cloud Shell. Si prefiere una herramienta basada en GUI, puede conectarse con PuTTY.  

### <a name="use-an-ssh-client"></a>Uso de un cliente SSH
Con la clave pública implementada en la VM de Azure y la clave privada en su sistema local, aplique SSH en la VM con la dirección IP o nombre DNS de la VM. Reemplace *azureuser* y *myvm.westus.cloudapp.azure.com* en el siguiente comando con el nombre de usuario administrador y el nombre de dominio completo (o dirección IP):

```bash
ssh azureuser@myvm.westus.cloudapp.azure.com
```

Si configuró una frase de contraseña cuando creó el par de claves, escríbala cuando se le solicite durante el proceso de inicio de sesión.

Si la máquina virtual está usando la directiva de acceso Just-In-Time, deberá solicitar acceso antes de poder conectarse a la máquina virtual. Para más información sobre la directiva Just-in-Time, vea [Administración del acceso a máquina virtual mediante Just-In-Time](../../security-center/security-center-just-in-time.md).

### <a name="connect-with-putty"></a>Conexión con PuTTY

Si instaló el [paquete de descarga de PuTTY](https://www.chiark.greenend.org.uk/~sgtatham/putty/download.html) y ya había generado previamente una clave privada de PuTTY (archivo .ppk), puede conectarse a la máquina virtual Linux con PuTTY.

1. Inicie PuTTY.

2. Rellene el nombre de host o la dirección IP de la máquina virtual en Azure Portal:

    ![Abrir conexión nueva de PuTTY](./media/ssh-from-windows/putty-new-connection.png)

3. Seleccione la categoría **Conexión** > **SSH** > **Autenticación**. Busque y seleccione la clave privada de PuTTY (archivo .ppk):

    ![Seleccionar la clave privada de PuTTY para la autenticación](./media/ssh-from-windows/putty-auth-dialog.png)

4. Haga clic en **Open** (Abrir) para conectarse a la máquina virtual.

## <a name="next-steps"></a>Pasos siguientes

* Para obtener pasos detallados, opciones y ejemplos avanzados del uso de claves SSH, consulte los [pasos detallados para crear pares de claves SSH](create-ssh-keys-detailed.md).

* También puede usar Powershell en Azure Cloud Shell para generar claves SSH y establecer conexiones SSH a máquinas virtuales Linux. Consulte [Inicio rápido de PowerShell](../../cloud-shell/quickstart-powershell.md#ssh).

* Si tiene problemas al usar SSH para conectarse a máquinas virtuales Linux, consulte [Troubleshoot SSH connections to an Azure Linux VM](../troubleshooting/troubleshoot-ssh-connection.md?toc=/azure/virtual-machines/linux/toc.json) (Solución de problemas de conexiones SSH a una máquina virtual Linux de Azure).
