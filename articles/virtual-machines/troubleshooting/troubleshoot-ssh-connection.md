---
title: Solución de problemas de conexión de SSH a una máquina virtual de Azure | Microsoft Docs
description: Cómo solucionar problemas de error de conexión SSH o de conexión SSH rechazada en una máquina virtual de Azure que ejecuta Lunux.
keywords: conexión ssh rechazada, error de ssh azure ssh, error de conexión ssh
services: virtual-machines-linux
documentationcenter: ''
author: genlin
manager: dcscontentpm
tags: top-support-issue,azure-service-management,azure-resource-manager
ms.service: virtual-machines-linux
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-linux
ms.topic: troubleshooting
ms.date: 05/30/2017
ms.author: genli
ms.openlocfilehash: c0f4e02a76044268946a4a482eaeccf5d622b8a7
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87036271"
---
# <a name="troubleshoot-ssh-connections-to-an-azure-linux-vm-that-fails-errors-out-or-is-refused"></a>Solución de problemas de conexiones SSH a una máquina virtual Linux de Azure que producen error o se rechazan.
Este artículo le ayudará a detectar y corregir los problemas que se producen como consecuencia de errores de Secure Shell (SSH), errores de conexión de SSH o que se rechace SSH al intentar conectarse a una máquina virtual Linux. Para solucionar problemas de conexión, puede usar Azure Portal, la CLI de Azure o la extensión de acceso de máquina virtual para Linux.


Si necesita más ayuda con cualquier aspecto de este artículo, puede ponerse en contacto con los expertos de Azure en [los foros de MSDN Azure o Stack Overflow](https://azure.microsoft.com/support/forums/). Como alternativa, puede registrar un incidente de soporte técnico de Azure. Vaya al [sitio de soporte técnico de Azure](https://azure.microsoft.com/support/options/) y seleccione **Obtener soporte**. Para obtener información sobre el uso del soporte técnico, lea las [Preguntas más frecuentes de soporte técnico de Microsoft Azure](https://azure.microsoft.com/support/faq/).

## <a name="quick-troubleshooting-steps"></a>Pasos rápidos para solucionar problemas
Después de cada paso de solución de problemas, intente volver a conectarse a la máquina virtual.

1. [Restablezca la configuración de SSH](#reset-config).
2. [Restablezca las credenciales](#reset-credentials) del usuario.
3. Compruebe las reglas del [grupo de seguridad de red](../../virtual-network/security-overview.md) que permitan el tráfico SSH.
   * Asegúrese de que existe una [regla de grupo de seguridad de red](#security-rules) para permitir el tráfico SSH (de forma predeterminada, el puerto TCP 22).
   * No se puede usar el redireccionamiento o la asignación de puertos sin utilizar un equilibrador de carga de Azure.
4. Compruebe el [estado de los recursos de la máquina virtual](../../service-health/resource-health-overview.md).
   * Asegúrese de que el estado de la máquina virtual se notifica como correcto.
   * Si tiene [habilitado el diagnóstico de arranque](boot-diagnostics.md), compruebe que la máquina virtual no notifica errores de arranque en los registros.
5. [Reinicie la máquina virtual](#restart-vm).
6. [Vuelva a implementar la máquina virtual](#redeploy-vm).

Siga leyendo para conocer pasos y soluciones más detallados de solución de problemas.

## <a name="available-methods-to-troubleshoot-ssh-connection-issues"></a>Métodos disponibles para la solución de problemas de conexiones de SSH
Puede restablecer las credenciales o la configuración de SSH mediante uno de los métodos siguientes:

* [Azure Portal](#use-the-azure-portal): este método funciona muy bien si necesita restablecer rápidamente la configuración o la clave de SSH y no tiene instaladas las herramientas de Azure.
* [Consola serie de máquina virtual en Azure](https://aka.ms/serialconsolelinux): la consola serie de máquina virtual funcionará independientemente de la configuración de SSH y proporcionará una consola interactiva a la máquina virtual. De hecho, la consola serie se ha diseñado para resolver específicamente las situaciones de tipo "no se puede SSH". Más detalles a continuación.
* [CLI de Azure](#use-the-azure-cli): si ya está en la línea de comandos, restablezca rápidamente la configuración o las credenciales de SSH. Si está trabajando con una máquina virtual clásica, puede usar la [CLI clásica de Azure](#use-the-azure-classic-cli).
* [Extensión Azure VMAccessForLinux](#use-the-vmaccess-extension): permite crear y reutilizar archivos de definición json para restablecer la configuración o las credenciales de usuario de SSH.

Después de cada paso de solución de problemas, intente conectarse de nuevo a la máquina virtual. Si sigue sin poder conectarse, pruebe el paso siguiente.

## <a name="use-the-azure-portal"></a>Uso de Azure Portal
Azure Portal proporciona una forma rápida de restablecer la configuración o las credenciales de usuario de SSH sin necesidad de instalar ninguna herramienta en el equipo local.

Para empezar, seleccione la máquina virtual en Azure Portal. Desplácese hacia abajo hasta la sección **Soporte técnico y solución de problemas** y seleccione **Restablecer contraseña** como en el ejemplo siguiente:

![Restablecer la configuración o las credenciales de SSH en Azure Portal](./media/troubleshoot-ssh-connection/reset-credentials-using-portal.png)

### <a name="reset-the-ssh-configuration"></a><a id="reset-config" />Restablecimiento de la configuración de SSH
Para restablecer la configuración de SSH, seleccione `Reset configuration only` en la sección **Modo** como se muestra en la captura de pantalla anterior y, a continuación, seleccione **Actualizar**. Una vez completada esta acción, intente acceder de nuevo a la máquina virtual.

### <a name="reset-ssh-credentials-for-a-user"></a><a id="reset-credentials" />Restablecimiento de las credenciales de SSH de un usuario
Para restablecer las credenciales de un usuario existente, seleccione `Reset SSH public key` o `Reset password` en la sección **Modo** como en la captura de pantalla anterior. Especifique el nombre de usuario y una clave de SSH o una nueva contraseña y, después, seleccione **Actualizar**.

Desde este menú también puede crear un usuario con privilegios de sudo en la máquina virtual. Escriba un nuevo nombre de usuario y la contraseña o clave SSH asociada y, a continuación, seleccione **Actualizar**.

### <a name="check-security-rules"></a><a id="security-rules" />Comprobación de reglas de seguridad

Use la [verificación del flujo IP](../../network-watcher/diagnose-vm-network-traffic-filtering-problem.md) para confirmar si una regla de un grupo de seguridad de red está bloqueando el tráfico hacia o desde una máquina virtual. También puede revisar cómo crear reglas de grupo de seguridad eficaces para garantizar que exista la regla NSG "Permitir" de entrada y tenga prioridad para el puerto SSH (valor predeterminado 22). Para más información, consulte [Uso de las reglas de seguridad vigentes para solucionar problemas de flujo de tráfico de máquinas virtuales](../../virtual-network/diagnose-network-traffic-filter-problem.md).

### <a name="check-routing"></a>Comprobación del enrutamiento

Use la funcionalidad [Próximo salto](../../network-watcher/diagnose-vm-network-routing-problem.md) de Network Watcher para confirmar que una ruta no impide que el tráfico se enrute hacia una máquina virtual o desde esta. También puede revisar las rutas efectivas para ver todas las rutas efectivas de una interfaz de red. Para más información, consulte [Uso de rutas eficaces para solucionar problemas de flujo de tráfico de máquinas virtuales](../../virtual-network/diagnose-network-routing-problem.md).

## <a name="use-the-azure-vm-serial-console"></a>Uso de la consola serie de máquina virtual en Azure
La [consola serie de máquina virtual en Azure](./serial-console-linux.md) ofrece acceso a una consola basada en texto para máquinas virtuales de Linux. Puede usar la consola para solucionar problemas de la conexión SSH en un shell interactivo. Asegúrese de que se han cumplido los [requisitos previos](./serial-console-linux.md#prerequisites) para usar la consola serie y pruebe los siguientes comandos para tratar de solucionar los problemas de conectividad SSH.

### <a name="check-that-ssh-is-running"></a>Comprobación de que SSH se está ejecutando
Puede usar el siguiente comando para comprobar si SSH se está ejecutando en la máquina virtual:

```console
ps -aux | grep ssh
```

Si hay algún resultado, SSH se está ejecutando.

### <a name="check-which-port-ssh-is-running-on"></a>Comprobación del puerto en el que se está ejecutando SSH

Puede usar el siguiente comando para comprobar el puerto en el que se está ejecutando SSH:

```console
sudo grep Port /etc/ssh/sshd_config
```

El resultado tendrá un aspecto similar al siguiente:

```output
Port 22
```

## <a name="use-the-azure-cli"></a>Uso de la CLI de Azure
Si todavía no la tiene, instale la [CLI de Azure](/cli/azure/install-az-cli2) más reciente e inicie sesión en una cuenta de Azure con [az login](/cli/azure/reference-index).

Si ha creado y cargado una imagen de disco Linux personalizada, asegúrese de que el [agente Linux de Microsoft Azure](../extensions/agent-linux.md) versión 2.0.5 o posterior esté instalado. Para máquinas virtuales creadas mediante imágenes de la galería, esta extensión de acceso ya está instalada y configurada automáticamente.

### <a name="reset-ssh-configuration"></a>Restablecer la configuración de SSH
Incialmente también puede tratar de restablecer la configuración de SSH a los valores predeterminados y reiniciar el servidor SSH en la máquina virtual. Esto no cambia el nombre de la cuenta de usuario, la contraseña ni las claves SSH.
En el ejemplo siguiente se utiliza [az vm user reset-ssh](/cli/azure/vm/user) para restablecer la configuración de SSH en la máquina virtual denominada `myVM` en `myResourceGroup`. Use sus propios valores, como se indica a continuación:

```azurecli
az vm user reset-ssh --resource-group myResourceGroup --name myVM
```

### <a name="reset-ssh-credentials-for-a-user"></a>Restablecimiento de las credenciales de SSH de un usuario
En el ejemplo siguiente se usa [az vm user update](/cli/azure/vm/user) para restablecer las credenciales de `myUsername` al valor especificado en `myPassword`, en la máquina virtual llamada `myVM` en `myResourceGroup`. Use sus propios valores, como se indica a continuación:

```azurecli
az vm user update --resource-group myResourceGroup --name myVM \
     --username myUsername --password myPassword
```

Si usa la autenticación de claves SSH, puede restablecer la clave SSH de un usuario determinado: En el ejemplo siguiente se usa **az vm access set-linux-user** para actualizar la clave SSH almacenada en `~/.ssh/id_rsa.pub` para el usuario llamado `myUsername`, en la máquina virtual llamada `myVM` en `myResourceGroup`. Use sus propios valores, como se indica a continuación:

```azurecli
az vm user update --resource-group myResourceGroup --name myVM \
    --username myUsername --ssh-key-value ~/.ssh/id_rsa.pub
```

## <a name="use-the-vmaccess-extension"></a>Uso de la extensión VMAccess
La extensión de acceso de máquina virtual de Linux lee un archivo json que define las acciones que se llevarán a cabo. Estas acciones incluyen el restablecimiento de SSHD o de una clave SSH o la adición de un usuario. Seguirá usando la CLI de Azure para llamar a la extensión VMAccess, pero puede reutilizar los archivos json entre varias máquinas virtuales si así lo desea. Este enfoque permite crear un repositorio de archivos json que luego se pueden llamar en escenarios determinados.

### <a name="reset-sshd"></a>Restablecer SSHD
Cree un archivo llamado `settings.json` con el siguiente contenido:

```json
{
    "reset_ssh":True
}
```

Con la CLI de Azure, luego puede llamar a la extensión `VMAccessForLinux` para restablecer la conexión de SSHD mediante la especificación de archivo json. En el ejemplo siguiente se utiliza [az vm extension set](/cli/azure/vm/extension) para restablecer SSHD en la máquina virtual denominada `myVM` en `myResourceGroup`. Use sus propios valores, como se indica a continuación:

```azurecli
az vm extension set --resource-group philmea --vm-name Ubuntu \
    --name VMAccessForLinux --publisher Microsoft.OSTCExtensions --version 1.2 --settings settings.json
```

### <a name="reset-ssh-credentials-for-a-user"></a>Restablecimiento de las credenciales de SSH de un usuario
Si SSHD parece funcionar correctamente, puede restablecer las credenciales de un usuario dado. Para restablecer la contraseña de un usuario, cree un archivo llamado `settings.json`. En el ejemplo siguiente se restablecen las credenciales de `myUsername` en el valor especificado en `myPassword`. Escriba las líneas siguientes en el archivo `settings.json`, usando sus propios valores:

```json
{
    "username":"myUsername", "password":"myPassword"
}
```

O bien, para restablecer la clave SSH de un usuario, primero cree un archivo llamado `settings.json`. En el ejemplo siguiente se restablecen las credenciales para `myUsername` al valor especificado en `myPassword`, en la máquina virtual llamada `myVM` en `myResourceGroup`. Escriba las líneas siguientes en el archivo `settings.json`, usando sus propios valores:

```json
{
    "username":"myUsername", "ssh_key":"mySSHKey"
}
```

Después de crear el archivo json, use la CLI de Azure para llamar a la extensión `VMAccessForLinux` y restablecer las credenciales de usuario SSH mediante la especificación del archivo json. En el ejemplo siguiente se restablecen las credenciales en la máquina virtual llamada `myVM` en `myResourceGroup`. Use sus propios valores, como se indica a continuación:

```azurecli
az vm extension set --resource-group philmea --vm-name Ubuntu \
    --name VMAccessForLinux --publisher Microsoft.OSTCExtensions --version 1.2 --settings settings.json
```

## <a name="use-the-azure-classic-cli"></a>Uso de la CLI de Azure clásica
Si todavía no la tiene, [instale la CLI de Azure clásica y conéctese a su suscripción de Azure](/cli/azure/install-classic-cli). Asegúrese de que está usando el modo de Resource Manager como se indica:

```azurecli
azure config mode arm
```

Si ha creado y cargado una imagen de disco Linux personalizada, asegúrese de que el [agente Linux de Microsoft Azure](../extensions/agent-linux.md) versión 2.0.5 o posterior esté instalado. Para máquinas virtuales creadas mediante imágenes de la galería, esta extensión de acceso ya está instalada y configurada automáticamente.

### <a name="reset-ssh-configuration"></a>Restablecer la configuración de SSH
La configuración de SSHD podría no ser correcta o que el servicio haya encontrado un error. Puede restablecer SSHD para asegurarse de que la configuración de SSH sea válida. El restablecimiento de SSHD debe ser el primer paso de solución de problemas que debe realizar.

En el ejemplo siguiente se restablece SSHD en una máquina virtual llamada `myVM` en el grupo de recursos llamado `myResourceGroup`. Use sus propios nombres de grupo de recursos y máquina virtual, como se indica a continuación:

```azurecli
azure vm reset-access --resource-group myResourceGroup --name myVM \
    --reset-ssh
```

### <a name="reset-ssh-credentials-for-a-user"></a>Restablecimiento de las credenciales de SSH de un usuario
Si SSHD parece funcionar correctamente, puede restablecer la contraseña de un usuario dado. En el ejemplo siguiente se restablecen las credenciales para `myUsername` al valor especificado en `myPassword`, en la máquina virtual llamada `myVM` en `myResourceGroup`. Use sus propios valores, como se indica a continuación:

```azurecli
azure vm reset-access --resource-group myResourceGroup --name myVM \
     --user-name myUsername --password myPassword
```

Si usa la autenticación de claves SSH, puede restablecer la clave SSH de un usuario determinado: En el ejemplo siguiente se actualiza la clave SSH almacenada en `~/.ssh/id_rsa.pub` para el usuario llamado `myUsername`, en la máquina virtual llamada `myVM` en `myResourceGroup`. Use sus propios valores, como se indica a continuación:

```azurecli
azure vm reset-access --resource-group myResourceGroup --name myVM \
    --user-name myUsername --ssh-key-file ~/.ssh/id_rsa.pub
```

## <a name="restart-a-vm"></a><a id="restart-vm" />Reinicio de una máquina virtual
Si ha restablecido la configuración y las credenciales de usuario de SSH, o ha encontrado un error al hacerlo, puede intentar reiniciar la máquina virtual para solucionar los problemas de procesos subyacentes.

### <a name="azure-portal"></a>Portal de Azure
Para reiniciar una máquina virtual mediante Azure Portal, seleccione la máquina virtual y, después, seleccione **Reiniciar**, como en el ejemplo siguiente:

![Reiniciar una máquina virtual en Azure Portal](./media/troubleshoot-ssh-connection/restart-vm-using-portal.png)

### <a name="azure-cli"></a>Azure CLI
En el ejemplo siguiente se usa [az vm restart](/cli/azure/vm) para reiniciar la máquina virtual llamada `myVM` en el grupo de recursos llamado `myResourceGroup`. Use sus propios valores, como se indica a continuación:

```azurecli
az vm restart --resource-group myResourceGroup --name myVM
```

### <a name="azure-classic-cli"></a>CLI de Azure clásica

[!INCLUDE [classic-vm-deprecation](../../../includes/classic-vm-deprecation.md)]

En el ejemplo siguiente se reinicia la máquina virtual llamada `myVM` en el grupo de recursos llamado `myResourceGroup`: Use sus propios valores, como se indica a continuación:

```console
azure vm restart --resource-group myResourceGroup --name myVM
```

## <a name="redeploy-a-vm"></a><a id="redeploy-vm" />Reimplementación de una máquina virtual
Puede volver a implementar una máquina virtual en otro nodo dentro de Azure, lo que podría corregir los problemas de red subyacentes. Para más información sobre cómo volver a implementar una máquina virtual, consulte [Nueva implementación de la máquina virtual en un nuevo nodo de Azure](./redeploy-to-new-node-windows.md?toc=/azure/virtual-machines/windows/toc.json).

> [!NOTE]
> Cuando finalice esta operación, se pierden datos de disco efímeros y se actualizan las direcciones IP dinámicas que están asociadas a la máquina virtual.
>
>

### <a name="azure-portal"></a>Portal de Azure
Para volver a implementar una máquina virtual mediante Azure Portal, seleccione la máquina virtual y desplácese hacia abajo hasta la sección **Soporte técnico y solución de problemas**. Seleccione **Volver a implementar**, como en el ejemplo siguiente:

![Nueva implementación de una máquina virtual en Azure Portal](./media/troubleshoot-ssh-connection/redeploy-vm-using-portal.png)

### <a name="azure-cli"></a>Azure CLI
En el ejemplo siguiente se usa [az vm redeploy](/cli/azure/vm) para volver a implementar la máquina virtual llamada `myVM` en el grupo de recursos llamado `myResourceGroup`. Use sus propios valores, como se indica a continuación:

```azurecli
az vm redeploy --resource-group myResourceGroup --name myVM
```

### <a name="azure-classic-cli"></a>CLI de Azure clásica

En el ejemplo siguiente se vuelve a implementar la máquina virtual llamada `myVM` en el grupo de recursos llamado `myResourceGroup`. Use sus propios valores, como se indica a continuación:

```console
azure vm redeploy --resource-group myResourceGroup --name myVM
```

## <a name="vms-created-by-using-the-classic-deployment-model"></a>Máquinas virtuales creadas con el modelo de implementación clásica

[!INCLUDE [classic-vm-deprecation](../../../includes/classic-vm-deprecation.md)]

Siga estos pasos para intentar resolver los errores de conexión SSH más habituales en las máquinas virtuales creadas con el modelo de implementación clásica: Después de cada paso, pruebe a conectarse a la máquina virtual.

* Restablezca el acceso remoto desde [Azure Portal](https://portal.azure.com). En Azure Portal, seleccione su máquina virtual y, después, seleccione **Restablecer acceso remoto...** .
* Reinicie la VM. En [Azure Portal](https://portal.azure.com), seleccione su máquina virtual y, después, seleccione **Restablecer**.

* Implemente de nuevo la máquina virtual en un nuevo nodo de Azure. Para más información sobre cómo volver a implementar una máquina virtual, consulte [Nueva implementación de la máquina virtual en un nuevo nodo de Azure](./redeploy-to-new-node-windows.md?toc=/azure/virtual-machines/windows/toc.json).

    Cuando finalice esta operación, se perderán los datos de disco efímeros y se actualizarán las direcciones IP dinámicas que están asociadas a la máquina virtual.
* Siga las instrucciones que se indican en [Restablecimiento de una contraseña o SSH para máquinas virtuales Linux](/previous-versions/azure/virtual-machines/linux/classic/reset-access-classic) para:

  * Restablecer la contraseña o la clave de SSH.
  * Crear una nueva cuenta de usuario de *sudo*.
  * Restablecer la configuración de SSH.
* Compruebe el estado de los recursos de la máquina virtual para ver si hay algún problema en la plataforma.<br>
     Seleccione la máquina virtual y desplácese hacia abajo hasta **Configuración** > **Comprobar estado**.

## <a name="additional-resources"></a>Recursos adicionales
* Si sigue sin poder establecer una conexión SSH a su máquina virtual después de seguir los pasos anteriores, puede examinar [pasos más detallados de solución de problemas](detailed-troubleshoot-ssh-connection.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) para ver pasos adicionales para resolver su problema.
* Para más información sobre cómo solucionar problemas de acceso a las aplicaciones, consulte [Solución de problemas de acceso a una aplicación que se ejecuta en una máquina virtual de Azure](./troubleshoot-app-connection.md?toc=/azure/virtual-machines/linux/toc.json).
* Para más información sobre cómo solucionar problemas de máquinas virtuales que se crearon mediante el modelo de implementación clásica, consulte [Restablecimiento de una contraseña o clave SSH para máquinas virtuales Linux](/previous-versions/azure/virtual-machines/linux/classic/reset-access-classic).
