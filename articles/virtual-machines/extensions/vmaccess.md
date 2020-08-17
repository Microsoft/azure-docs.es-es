---
title: Restablecimiento del acceso a una máquina virtual Linux de Azure
description: Administración de usuarios administrativos y restablecimiento del acceso en máquinas virtuales Linux con la extensión VMAccess y la CLI de Azure
services: virtual-machines-linux
documentationcenter: ''
author: axayjo
manager: gwallace
editor: ''
tags: azure-resource-manager
ms.assetid: 261a9646-1f93-407e-951e-0be7226b3064
ms.service: virtual-machines-linux
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-linux
ms.devlang: azurecli
ms.topic: article
ms.date: 05/10/2018
ms.author: akjosh
ms.openlocfilehash: b3b6c9d5835f6d81ab8641d8a43fb7228b6b53ab
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87085646"
---
# <a name="manage-administrative-users-ssh-and-check-or-repair-disks-on-linux-vms-using-the-vmaccess-extension-with-the-azure-cli"></a>Administración de usuarios administrativos, SSH y comprobación o reparación de discos en máquinas virtuales Linux con la extensión VMAccess y la CLI de Azure
## <a name="overview"></a>Información general
El disco de la máquina virtual de Linux muestra errores. De alguna forma, restableció la contraseña raíz de la máquina virtual de Linux o eliminó por accidente la clave privada SSH. Si esto sucedió en el centro de datos, deberá ir ahí y, luego, abrir el conmutador KVM para llegar a la consola del servidor. Piense en la extensión VMAccess de Azure como ese conmutador KVM que le permite tener acceso a la consola para restablecer el acceso a Linux o realizar el mantenimiento de nivel de disco.

En este artículo se muestra cómo usar la extensión VMAccess de Azure para comprobar o reparar un disco, restablecer el acceso de usuarios administrativos, administrar cuentas de usuario o actualizar la configuración de SSH en Linux cuando se ejecutan como máquinas virtuales de Azure Resource Manager. Si necesita administrar máquinas virtuales clásicas, puede seguir las instrucciones que se encuentran en la documentación [de máquinas virtuales clásicas](/previous-versions/azure/virtual-machines/linux/classic/reset-access-classic). 
 
> [!NOTE]
> Si utiliza la extensión VMAccess para restablecer la contraseña de la máquina virtual después de instalar la extensión de inicio de sesión de AAD, tendrá que volver a ejecutar esta última extensión para habilitar del nuevo el inicio de sesión de AAD en la máquina.

## <a name="prerequisites"></a>Prerequisites
### <a name="operating-system"></a>Sistema operativo

La extensión de acceso a la máquina virtual puede ejecutarse en estas distribuciones de Linux:

| Distribución | Versión |
|---|---|
| Ubuntu | 16.04 LTS, 14.04 LTS y 12.04 LTS |
| Debian | Debian 7.9+, 8.2+ |
| Red Hat | RHEL 6.7+, 7.1+ |
| Oracle Linux | 6.4+, 7.0+ |
| Suse | 11 y 12 |
| OpenSuse | openSUSE Leap 42.2+ |
| CentOS | CentOS 6.3+, 7.0+ |
| CoreOS | 494.4.0+ |

## <a name="ways-to-use-the-vmaccess-extension"></a>Formas de usar la extensión VMAccess
Hay dos formas de usar la extensión VMAccess en las máquinas virtuales Linux:

* Usar la CLI de Azure y los parámetros necesarios.
* [Usando archivos sin formato JSON que procese la extensión VMAccess](#use-json-files-and-the-vmaccess-extension) para realizar acciones en ellos

En los ejemplos siguientes se utilizan comandos [az vm user](/cli/azure/vm/user). Para realizar estos pasos, es preciso tener instalada la [CLI de Azure](/cli/azure/install-az-cli2) más reciente y haber iniciado sesión en una cuenta de Azure mediante [az login](/cli/azure/reference-index).

## <a name="update-ssh-key"></a>Actualizar la clave SSH
En el ejemplo siguiente se actualiza la clave SSH del usuario `azureuser` en la VM denominada `myVM`:

```azurecli-interactive
az vm user update \
  --resource-group myResourceGroup \
  --name myVM \
  --username azureuser \
  --ssh-key-value ~/.ssh/id_rsa.pub
```

> **Nota:** El comando `az vm user update` anexa el nuevo texto de clave pública al archivo `~/.ssh/authorized_keys` para el usuario administrador de la VM. Esto no reemplaza ni quita las claves SSH existentes. Esta acción no quitará las claves anteriores establecidas en el momento de la implementación ni las actualizaciones posteriores a través de la extensión VMAccess.

## <a name="reset-password"></a>Restablecimiento de contraseña
En el ejemplo siguiente se restablece la contraseña del usuario `azureuser` en la máquina virtual denominada "`myVM`":

```azurecli-interactive
az vm user update \
  --resource-group myResourceGroup \
  --name myVM \
  --username azureuser \
  --password myNewPassword
```

## <a name="restart-ssh"></a>Reinicio de SSH
En el ejemplo siguiente se reinicia el demonio de SSH y se restablece la configuración de SSH a los valores predeterminados en una máquina virtual denominada "`myVM`":

```azurecli-interactive
az vm user reset-ssh \
  --resource-group myResourceGroup \
  --name myVM
```

## <a name="create-an-administrativesudo-user"></a>Creación de un usuario administrativo/sudo
En el ejemplo siguiente se crea un usuario denominado `myNewUser` con permisos **sudo**. La cuenta utiliza una clave SSH para la autenticación en la máquina virtual denominada `myVM`. Este método está diseñado para ayudarle a recuperar el acceso a una máquina virtual en caso de que las credenciales actuales se pierdan o se olviden. Se recomienda limitar las cuentas con permisos **sudo**.

```azurecli-interactive
az vm user update \
  --resource-group myResourceGroup \
  --name myVM \
  --username myNewUser \
  --ssh-key-value ~/.ssh/id_rsa.pub
```

## <a name="delete-a-user"></a>Eliminar un usuario
En el ejemplo siguiente se elimina un usuario denominado "`myNewUser`" en la máquina virtual llamada "`myVM`":

```azurecli-interactive
az vm user delete \
  --resource-group myResourceGroup \
  --name myVM \
  --username myNewUser
```

## <a name="use-json-files-and-the-vmaccess-extension"></a>Uso de archivos JSON y de la extensión VMAccess
En los ejemplos siguientes se emplean archivos sin formato JSON. Use [az vm extension set](/cli/azure/vm/extension) y, luego, llame a los archivos JSON. Estos archivos JSON también se pueden llamar desde las plantillas de Azure. 

### <a name="reset-user-access"></a>Restablecimiento del acceso de un usuario
Si perdió el acceso a la raíz de la VM Linux, puede iniciar un script de VMAccess para actualizar la contraseña o la clave SSH de un usuario.

Para actualizar la clave pública SSH de un usuario, cree un archivo denominado "`update_ssh_key.json`" y agregue la configuración en el siguiente formato. Sustituya sus propios valores por los parámetros `username` y `ssh_key`:

```json
{
  "username":"azureuser",
  "ssh_key":"ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQCZ3S7gGp3rcbKmG2Y4vGZFMuMZCwoUzZNGxxxxxx2XV2x9FfAhy8iGD+lF8UdjFX3t5ebMm6BnnMh8fHwkTRdOt3LDQq8o8ElTBrZaKPxZN2thMZnODs5Hlemb2UX0oRIGRcvWqsd4oJmxsXa/Si98Wa6RHWbc9QZhw80KAcOVhmndZAZAGR+Wq6yslNo5TMOr1/ZyQAook5C4FtcSGn3Y+WczaoGWIxG4ZaWk128g79VIeJcIQqOjPodHvQAhll7qDlItVvBfMOben3GyhYTm7k4YwlEdkONm4yV/UIW0la1rmyztSBQIm9sZmSq44XXgjVmDHNF8UfCZ1ToE4r2SdwTmZv00T2i5faeYnHzxiLPA3Enub7xxxxxxwFArnqad7MO1SY1kLemhX9eFjLWN4mJe56Fu4NiWJkR9APSZQrYeKaqru4KUC68QpVasNJHbuxPSf/PcjF3cjO1+X+4x6L1H5HTPuqUkyZGgDO4ynUHbko4dhlanALcriF7tIfQR9i2r2xOyv5gxJEW/zztGqWma/d4rBoPjnf6tO7rLFHXMt/DVTkAfn5wxxtLDwkn5FMyvThRmex3BDf0gujoI1y6cOWLe9Y5geNX0oj+MXg/W0cXAtzSFocstV1PoVqy883hNoeQZ3mIGB3Q0rIUm5d9MA2bMMt31m1g3Sin6EQ== azureuser@myVM"
}
```

Ejecute el script VMAccess con:

```azurecli-interactive
az vm extension set \
  --resource-group myResourceGroup \
  --vm-name myVM \
  --name VMAccessForLinux \
  --publisher Microsoft.OSTCExtensions \
  --version 1.4 \
  --protected-settings update_ssh_key.json
```

Para restablecer la contraseña de un usuario, cree un archivo denominado "`reset_user_password.json`" y agregue la configuración en el siguiente formato. Sustituya sus propios valores por los parámetros `username` y `password`:

```json
{
  "username":"azureuser",
  "password":"myNewPassword" 
}
```

Ejecute el script VMAccess con:

```azurecli-interactive
az vm extension set \
  --resource-group myResourceGroup \
  --vm-name myVM \
  --name VMAccessForLinux \
  --publisher Microsoft.OSTCExtensions \
  --version 1.4 \
  --protected-settings reset_user_password.json
```

### <a name="restart-ssh"></a>Reinicio de SSH
Para reiniciar el demonio de SSH y restablecer la configuración de SSH a los valores predeterminados, cree un archivo denominado "`reset_sshd.json`". Agregue el siguiente contenido:

```json
{
  "reset_ssh": true
}
```

Ejecute el script VMAccess con:

```azurecli-interactive
az vm extension set \
  --resource-group myResourceGroup \
  --vm-name myVM \
  --name VMAccessForLinux \
  --publisher Microsoft.OSTCExtensions \
  --version 1.4 \
  --protected-settings reset_sshd.json
```

### <a name="manage-administrative-users"></a>Administración de usuarios administrativos

Con el objetivo de crear un usuario con permisos **sudo** que utilice una clave SSH para la autenticación, cree un archivo denominado `create_new_user.json` y agregue la configuración en el siguiente formato. Sustituya sus propios valores por los parámetros `username` y `ssh_key`. Este método está diseñado para ayudarle a recuperar el acceso a una máquina virtual en caso de que las credenciales actuales se pierdan o se olviden. Se recomienda limitar las cuentas con permisos **sudo**.

```json
{
  "username":"myNewUser",
  "ssh_key":"ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQCZ3S7gGp3rcbKmG2Y4vGZFMuMZCwoUzZNG1vHY7P2XV2x9FfAhy8iGD+lF8UdjFX3t5ebMm6BnnMh8fHwkTRdOt3LDQq8o8ElTBrZaKPxZN2thMZnODs5Hlemb2UX0oRIGRcvWqsd4oJmxsXa/Si98Wa6RHWbc9QZhw80KAcOVhmndZAZAGR+Wq6yslNo5TMOr1/ZyQAook5C4FtcSGn3Y+WczaoGWIxG4ZaWk128g79VIeJcIQqOjPodHvQAhll7qDlItVvBfMOben3GyhYTm7k4YwlEdkONm4yV/UIW0la1rmyztSBQIm9sZmSq44XXgjVmDHNF8UfCZ1ToE4r2SdwTmZv00T2i5faeYnHzxiLPA3Enub7iUo5IdwFArnqad7MO1SY1kLemhX9eFjLWN4mJe56Fu4NiWJkR9APSZQrYeKaqru4KUC68QpVasNJHbuxPSf/PcjF3cjO1+X+4x6L1H5HTPuqUkyZGgDO4ynUHbko4dhlanALcriF7tIfQR9i2r2xOyv5gxJEW/zztGqWma/d4rBoPjnf6tO7rLFHXMt/DVTkAfn5woYtLDwkn5FMyvThRmex3BDf0gujoI1y6cOWLe9Y5geNX0oj+MXg/W0cXAtzSFocstV1PoVqy883hNoeQZ3mIGB3Q0rIUm5d9MA2bMMt31m1g3Sin6EQ== myNewUser@myVM",
  "password":"myNewUserPassword"
}
```

Ejecute el script VMAccess con:

```azurecli-interactive
az vm extension set \
  --resource-group myResourceGroup \
  --vm-name myVM \
  --name VMAccessForLinux \
  --publisher Microsoft.OSTCExtensions \
  --version 1.4 \
  --protected-settings create_new_user.json
```

Para eliminar un usuario, cree un archivo denominado"`delete_user.json`" y agregue el siguiente contenido. Sustituya su propio valor para el parámetro `remove_user`:

```json
{
  "remove_user":"myNewUser"
}
```

Ejecute el script VMAccess con:

```azurecli-interactive
az vm extension set \
  --resource-group myResourceGroup \
  --vm-name myVM \
  --name VMAccessForLinux \
  --publisher Microsoft.OSTCExtensions \
  --version 1.4 \
  --protected-settings delete_user.json
```

### <a name="check-or-repair-the-disk"></a>Comprobación o reparación del disco
Al utilizar VMAccess, también se puede comprobar y reparar un disco que haya agregado a la VM con Linux.

Para comprobar y reparar después el disco, cree un archivo denominado "`disk_check_repair.json`" y agregue la configuración en el siguiente formato. Sustituya su propio valor para el nombre de `repair_disk`:

```json
{
  "check_disk": "true",
  "repair_disk": "true, mydiskname"
}
```

Ejecute el script VMAccess con:

```azurecli-interactive
az vm extension set \
  --resource-group myResourceGroup \
  --vm-name myVM \
  --name VMAccessForLinux \
  --publisher Microsoft.OSTCExtensions \
  --version 1.4 \
  --protected-settings disk_check_repair.json
```
## <a name="troubleshoot-and-support"></a>Solución de problemas y asistencia

### <a name="troubleshoot"></a>Solución de problemas

Los datos sobre el estado de las implementaciones de extensiones pueden recuperarse desde Azure Portal y mediante la CLI de Azure. Para ver el estado de implementación de las extensiones de una máquina virtual determinada, ejecute el comando siguiente con la CLI de Azure.

```azurecli
az vm extension list --resource-group myResourceGroup --vm-name myVM -o table
```

### <a name="support"></a>Soporte técnico

Si necesita más ayuda con cualquier aspecto de este artículo, puede ponerse en contacto con los expertos de Azure en los [foros de MSDN Azure o Stack Overflow](https://azure.microsoft.com/support/forums/). Como alternativa, puede registrar un incidente de soporte técnico de Azure. Vaya al [sitio de soporte técnico de Azure](https://azure.microsoft.com/support/options/) y seleccione Obtener soporte. Para obtener información sobre el uso del soporte técnico, lea las [Preguntas más frecuentes de soporte técnico de Microsoft Azure](https://azure.microsoft.com/support/faq/).
