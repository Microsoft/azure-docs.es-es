---
title: Consola serie de Azure para GRUB y el modo de usuario único | Microsoft Docs
description: En este artículo se describe cómo usar la consola serie para GRUB en las máquinas virtuales de Azure.
services: virtual-machines-linux
documentationcenter: ''
author: asinn826
manager: gwallace
editor: ''
tags: azure-resource-manager
ms.service: virtual-machines-linux
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure-services
ms.date: 08/06/2019
ms.author: alsin
ms.openlocfilehash: 5341cc62a7d02c3072df90becf893dec18427ac2
ms.sourcegitcommit: 14bf4129a73de2b51a575c3a0a7a3b9c86387b2c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/30/2020
ms.locfileid: "87439550"
---
# <a name="use-serial-console-to-access-grub-and-single-user-mode"></a>Uso de la consola serie para acceder a GRUB y al modo de usuario único
El cargador de arranque unificado (GRUB) es probablemente lo primero que se ve al arrancar una máquina virtual (VM). Al mostrarse antes de que se haya iniciado el sistema operativo, no se puede acceder al GRUB mediante SSH. Desde GRUB puede modificar la configuración de arranque para arrancar en modo usuario único, entre otras cosas.

El modo de usuario único constituye un entorno mínimo con una funcionalidad mínima. Puede ser útil para investigar los problemas de arranque, los problemas del sistema de archivos o los problemas de red. Se pueden ejecutar menos servicios en segundo plano y, en función del nivel de ejecución, es posible que ni siquiera se monte un sistema de archivos de forma automática.

El modo de usuario único también es útil en situaciones en las que la máquina virtual solo se puede configurar para aceptar claves SSH para el inicio de sesión. En este caso, puede usar el modo de usuario único para crear una cuenta con autenticación de contraseña.

> [!NOTE]
> El servicio de consola serie solo permite a los usuarios con permisos de nivel de *colaborador* (o superiores) acceder a la consola serie de una máquina virtual.

Para entrar en el modo de usuario único, entre en GRUB al arrancar la máquina virtual y modifique la configuración de arranque en GRUB. Consulte las instrucciones detalladas para entrar en GRUB en la siguiente sección. En general, si la máquina virtual está configurada para mostrar GRUB, puede usar el botón de reinicio en la consola serie de la máquina virtual para reiniciar la máquina virtual y mostrar GRUB.

![Botón de reinicio de la consola serie Linux](./media/virtual-machines-serial-console/virtual-machine-serial-console-restart-button-bar.png)

## <a name="general-grub-access"></a>Acceso a GRUB general
Para acceder a GRUB, reinicie la máquina virtual mientras el panel de la consola serie está abierto. Algunas distribuciones requieren la entrada de teclado para mostrar GRUB y otras lo muestran de forma automática durante unos segundos y permiten la entrada de teclado del usuario para cancelar el tiempo de espera.

Para acceder al modo de usuario único, querrá asegurarse de que GRUB está habilitado en la máquina virtual. Según la distribución, puede necesitar algunas operaciones de configuración para asegurarse de que GRUB está habilitado. Para obtener información específica de la distribución, consulte la sección siguiente.

### <a name="restart-your-vm-to-access-grub-in-serial-console"></a>Reinicio de una máquina virtual para acceder a GRUB en la consola serie
Puede reiniciar la máquina virtual desde la consola de serie si mantiene el mouse sobre el botón **Reiniciar** y selecciona **Reiniciar VM**. En la parte inferior del panel se muestra una notificación sobre el reinicio.

También puede reiniciar la máquina virtual al ejecutar el comando SysRq "b" si [SysRq](./serial-console-nmi-sysrq.md) está habilitado. Para obtener información sobre lo que puede esperar de GRUB al reiniciar el equipo, consulte las instrucciones específicas de la distribución en las siguientes secciones.

![Reinicio de la consola serie Linux](./media/virtual-machines-serial-console/virtual-machine-serial-console-restart-button-ubuntu.gif)

## <a name="general-single-user-mode-access"></a>Acceso en modo de usuario único general
Es posible que necesite acceder manualmente al modo de usuario único cuando no se ha configurado una cuenta con la autenticación de contraseña. Debe modificar la configuración de GRUB para entrar manualmente al modo de usuario único. Después de hacerlo, consulte la sección "Uso del modo de usuario único para restablecer o agregar una contraseña" para obtener instrucciones adicionales.

Si la máquina virtual no puede arrancar, algunas distribuciones le ponen automáticamente en modo de usuario único o modo de emergencia. Sin embargo, otras requieren configuración adicional (como la configuración de una contraseña raíz) antes de ponerle en modo de usuario único o de emergencia de forma automática.

### <a name="use-single-user-mode-to-reset-or-add-a-password"></a>Uso del modo de usuario único para restablecer o agregar una contraseña
Una vez en el modo de usuario único, agregue un usuario nuevo con privilegios de sudo mediante los siguientes pasos:
1. Ejecute `useradd <username>` para agregar un usuario.
1. Ejecute `sudo usermod -a -G sudo <username>` para conceder los nuevos privilegios de usuario raíz.
1. Use `passwd <username>` para establecer la contraseña del nuevo usuario. Después, puede iniciar sesión como el nuevo usuario.


## <a name="access-for-red-hat-enterprise-linux-rhel"></a>Acceso a Red Hat Enterprise Linux (RHEL)
Si RHEL no se puede arrancar con normalidad, le pondrá en modo de usuario único automáticamente. Sin embargo, si no ha configurado el acceso raíz para el modo de usuario único, no tiene una contraseña raíz y no puede iniciar sesión. Existe una solución alternativa (consulte la sección "Especificación manual del modo de usuario único en RHEL"), aunque se recomienda configurar inicialmente el acceso raíz.

### <a name="grub-access-in-rhel"></a>Acceso a GRUB en RHEL
RHEL viene con GRUB habilitado listo para su uso. Para entrar a GRUB, reinicie la máquina virtual con el comando `sudo reboot` y presione cualquier tecla. Se debe mostrar el panel de GRUB. Si no se muestra, asegúrese de que las siguientes líneas están presentes en el archivo de GRUB (`/etc/default/grub`):

**Para RHEL 8**

>[!NOTE]
> Red Hat recomienda el uso de Grubby para configurar parámetros de línea de comandos de kernel en RHEL 8+. Actualmente no es posible actualizar el tiempo de espera de GRUB y los parámetros de terminal mediante Grubby. Para modificar la actualización del argumento GRUB_CMDLINE_LINUX para todas las entradas de arranque, ejecute `grubby --update-kernel=ALL --args="console=ttyS0,115200 console=tty1 console=ttyS0 earlyprintk=ttyS0 rootdelay=300"`. [Aquí](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html/managing_monitoring_and_updating_the_kernel/configuring-kernel-command-line-parameters_managing-monitoring-and-updating-the-kernel) hay más detalles disponibles.

```
GRUB_TIMEOUT=5
GRUB_TERMINAL="serial console"
GRUB_CMDLINE_LINUX="console=tty1 console=ttyS0 earlyprintk=ttyS0 rootdelay=300"
```

**Para RHEL 7**

```
GRUB_TIMEOUT=5
GRUB_TERMINAL_OUTPUT="serial console"
GRUB_CMDLINE_LINUX="console=tty1 console=ttyS0,115200n8 earlyprintk=ttyS0,115200 rootdelay=300 net.ifnames=0"
```

> [!NOTE]
> Red Hat también proporciona documentación para arrancar en modo de rescate, modo de emergencia, modo de depuración y restablecer la contraseña raíz. Para obtener instrucciones, consulte [Edición de los menús de terminal durante el arranque](https://aka.ms/rhel7grubterminal).

### <a name="set-up-root-access-for-single-user-mode-in-rhel"></a>Configuración del acceso raíz para el modo de usuario único en RHEL
El usuario raíz está deshabilitado de manera predeterminada. El modo de usuario único en RHEL requiere que el usuario raíz esté habilitado. Si necesita habilitar el modo de usuario único, siga estas instrucciones:

1. Inicie sesión en el sistema de Red Hat a través de SSH.
1. Cambie a raíz.
1. Habilite la contraseña del usuario raíz mediante los siguientes pasos:
    * Ejecute `passwd root` (establezca una contraseña de raíz segura).
1. Asegúrese de que el usuario raíz solo puede iniciar sesión a través de ttyS0 mediante los siguientes pasos: a. Ejecute `edit /etc/ssh/sshd_config` y asegúrese de que PermitRootLogIn está establecido en `no`.
    b. Ejecute `edit /etc/securetty file` para permitir el inicio de sesión solo a través de ttyS0.

Si ahora el sistema arranca en modo de usuario único puede iniciar sesión con la contraseña raíz.

Como alternativa, en el caso de RHEL 7.4 y versiones posteriores o 6.9 y versiones posteriores, puede habilitar el modo de usuario único en las solicitudes de GRUB. Para ello, consulte [Arranque en modo de usuario único](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/5/html/installation_guide/s1-rescuemode-booting-single).

### <a name="manually-enter-single-user-mode-in-rhel"></a>Especificación manual del modo de usuario único en RHEL
Si ha configurado GRUB y el acceso raíz con las instrucciones anteriores, puede entrar al modo de usuario único mediante los siguientes pasos:

1. Para entrar en GRUB, presione Esc cuando reinicie la máquina virtual.
1. En GRUB, presione E para editar el sistema operativo que quiere arrancar. El sistema operativo suele aparecer en la primera línea.
1. Busque la línea de kernel. En Azure, comienza por *linux16*.
1. Presione Ctrl + E para ir al final de la línea.
1. Al final de la línea, agregue *systemd.unit=rescue.target*.

    Esta acción arranca el sistema en modo de usuario único. Si quiere usar el modo de emergencia, agregue *systemd.unit=emergency.target* al final de la línea (en lugar de *systemd.unit=rescue.target*).

1. Presione Ctrl + X para salir y reiniciar con la configuración aplicada.

   Se le pedirá la contraseña de administrador para que pueda entrar en el modo de usuario único. Esta contraseña es la que creó en las instrucciones anteriores.

    ![Imagen animada que muestra una interfaz de línea de comandos. El usuario selecciona un servidor, localiza el final de la línea del kernel y, a continuación, escribe el texto especificado.](../media/virtual-machines-serial-console/virtual-machine-linux-serial-console-rhel-enter-emergency-shell.gif)

### <a name="enter-single-user-mode-without-root-account-enabled-in-rhel"></a>Especificación del modo de usuario único sin una cuenta raíz habilitada en RHEL
Si no ha habilitado el usuario raíz con las instrucciones anteriores, puede restablecer la contraseña raíz mediante los siguientes pasos:

> [!NOTE]
> Si está usando SELinux, cuando restablezca la contraseña raíz asegúrese de seguir los pasos adicionales que se describen en la [documentación de Red Hat](https://aka.ms/rhel7grubterminal).

1. Para entrar en GRUB, presione Esc cuando reinicie la máquina virtual.

1. En GRUB, presione E para editar el sistema operativo que quiere arrancar. El sistema operativo suele aparecer en la primera línea.
1. Busque la línea de kernel. En Azure, comienza por *linux16*.
1. Al final de la línea, agregue *rd.break*. Deje un espacio entre la línea de kernel y *rd.break*.

    Esta acción interrumpirá el proceso de arranque antes de que el control pase de `initramfs` a `systemd`, como se indica en la [documentación de Red Hat](https://aka.ms/rhel7rootpassword).
1. Presione Ctrl + X para salir y reiniciar con la configuración aplicada.

   Después de arrancar, se le pondrá en modo de emergencia con un sistema de archivos de solo lectura.

1. En el shell, escriba `mount -o remount,rw /sysroot` para volver a montar el sistema de archivos raíz con permisos de lectura/escritura.
1. Después de arrancar en modo de usuario único, escriba `chroot /sysroot` para cambiar a la celda `sysroot`.
1. Ahora está en la raíz. Para restablecer la contraseña raíz puede usar `passwd` y, a continuación, seguir las instrucciones anteriores para especificar el modo de usuario único.
1. Cuando haya terminado, escriba `reboot -f` para reiniciar el equipo.

![Imagen animada que muestra una interfaz de línea de comandos. El usuario selecciona un servidor, localiza el final de la línea del kernel y escribe los comandos especificados.](../media/virtual-machines-serial-console/virtual-machine-linux-serial-console-rhel-emergency-mount-no-root.gif)

> [!NOTE]
> Si ejecuta las instrucciones anteriores se le pondrá en el shell de emergencia, de tal forma que pueda también realizar tareas como editar `fstab`. Sin embargo, normalmente sugerimos que restablezca la contraseña raíz y la use para entrar en modo de usuario único.

## <a name="access-for-centos"></a>Acceso para CentOS
De forma similar a Red Hat Enterprise Linux, el modo de usuario único en CentOS requiere que GRUB y el usuario raíz estén habilitados.

### <a name="grub-access-in-centos"></a>Acceso a GRUB en CentOS
CentOS viene con GRUB habilitado listo para su uso. Para entrar a GRUB, reinicie la máquina virtual con el comando `sudo reboot` y presione cualquier tecla. Esta acción muestra el panel de GRUB.

### <a name="single-user-mode-in-centos"></a>Modo de usuario único en CentOS
Para habilitar el modo de usuario único en CentOS, siga las instrucciones anteriores para RHEL.

## <a name="access-for-ubuntu"></a>Acceso para Ubuntu
Las imágenes de Ubuntu no requieren una contraseña raíz. Si el sistema arranca en modo de usuario único, puede usarlo sin necesidad de otras credenciales.

### <a name="grub-access-in-ubuntu"></a>Acceso a GRUB en Ubuntu
Para acceder a GRUB, mantenga presionado Esc mientras arranca la máquina virtual.

Puede que las imágenes de Ubuntu no muestren de forma predeterminada el panel de GRUB. Puede cambiar la configuración mediante los siguientes pasos:
1. En un editor de texto, abra el archivo */etc/default/grub.d/50-cloudimg-settings.cfg*.

1. Cambie el valor de `GRUB_TIMEOUT` por un valor distinto de cero.
1. En un editor de texto, abra */etc/default/grub*.
1. Marque como comentario la línea `GRUB_HIDDEN_TIMEOUT=1`.
1. Asegúrese de que hay una línea `GRUB_TIMEOUT_STYLE=menu`.
1. Ejecute `sudo update-grub`.

### <a name="single-user-mode-in-ubuntu"></a>Modo de usuario único en Ubuntu
Si Ubuntu no se puede arrancar con normalidad, le pondrá en modo de usuario único automáticamente. Para especificar el modo de usuario único manualmente, haga lo siguiente:

1. Desde GRUB, presione E para editar la entrada de arranque (la entrada de Ubuntu).
1. Busque la línea que empieza por *linux* y, a continuación, busque *ro*.
1. Agregue *single* después de *ro* y asegúrese de que hay un espacio antes y después de *single*.
1. Presione Ctrl + X para reiniciar el equipo con esta configuración y especificar el modo de usuario único.

### <a name="use-grub-to-invoke-bash-in-ubuntu"></a>Uso de GRUB para invocar Bash en Ubuntu
Después de probar las instrucciones anteriores, puede haber situaciones (por ejemplo, una contraseña raíz olvidada) en las que todavía no pueda acceder al modo de usuario único en la VM de Ubuntu. También puede indicar al kernel que ejecute `/bin/bash` como inicialización, en lugar de una inicialización de sistema. Esta acción le proporciona un shell de bash y permite dar mantenimiento al sistema. Use las siguientes instrucciones:

1. Desde GRUB, presione E para editar la entrada de arranque (la entrada de Ubuntu).

1. Busque la línea que empieza por *linux* y, a continuación, busque *ro*.
1. Reemplace *ro* por *rw init=/bin/bash*.

    Esta acción monta el sistema de archivos como de lectura y escritura y usa `/bin/bash` como proceso de inicialización.
1. Presione Control + X para reiniciar el equipo con esta configuración.

## <a name="access-for-coreos"></a>Acceso para CoreOS
El modo de usuario único en CoreOS requiere que GRUB esté habilitado.

### <a name="grub-access-in-coreos"></a>Acceso a GRUB en CoreOS
Para acceder a GRUB, presione cualquier tecla mientras arranca la máquina virtual.

### <a name="single-user-mode-in-coreos"></a>Modo de usuario único en CoreOS
Si CoreOS no se puede arrancar con normalidad, le pondrá en modo de usuario único automáticamente. Para especificar el modo de usuario único manualmente, haga lo siguiente:

1. Desde GRUB, presione E para editar la entrada de arranque.

1. Busque la línea que empieza por *linux$* . Debe haber dos instancias de la línea, cada una encapsulada en una cláusula *if... else*.
1. Agregue *coreos.autologin=ttyS0* al final de cada una de las líneas *linux$* .
1. Presione Ctrl + X para reiniciar el equipo con esta configuración y especificar el modo de usuario único.

## <a name="access-for-suse-sles"></a>Acceso para SUSE Linux Enterprise Server
Las imágenes más recientes de SLES 12 SP3+ permiten el acceso a través de la consola serie si el sistema arranca en modo de emergencia.

### <a name="grub-access-in-suse-sles"></a>Acceso a GRUB en SUSE Linux Enterprise Server
El acceso a GRUB en SUSE Linux Enterprise Server requiere la configuración del cargador de arranque a través de YaST. Para crear la configuración, haga lo siguiente:

1. Use SSH para iniciar sesión en la máquina virtual SLES y después ejecute `sudo yast bootloader`. Presione la tecla de tabulación, presione entrar y, a continuación, use las teclas de dirección para navegar por el menú.

1. Vaya a **Kernel Parameters** (Parámetros de kernel) y después marque la casilla de verificación **Use serial console** (Usar consola serial).
1. Agregue `serial --unit=0 --speed=9600 --parity=no` a los argumentos de **Consola**.
1. Presione F10 para guardar la configuración y salir.
1. Para entrar en GRUB, reinicie la máquina virtual y presione cualquier tecla durante la secuencia de arranque para que el panel de GRUB permanezca en pantalla.

    El tiempo de espera predeterminado para GRUB es **1 s**. Puede modificar esta configuración si cambia la variable `GRUB_TIMEOUT` en el archivo */etc/default/grub*.

![Inicialización de la configuración del cargador de arranque](../media/virtual-machines-serial-console/virtual-machine-linux-serial-console-sles-yast-grub-config.gif)

### <a name="single-user-mode-in-suse-sles"></a>Modo de usuario único en SUSE SLES
Si SLES no puede arrancar con normalidad, se le pondrá automáticamente en el shell de emergencia. Para entrar en el shell de emergencia manualmente, haga lo siguiente:

1. Desde GRUB, presione E para editar la entrada de arranque (la entrada de SLES).

1. Busque la línea del kernel que empieza por *linux*.
1. Agregue *systemd.unit=emergency.target* al final de la línea del kernel.
1. Presione Ctrl + X para reiniciar el equipo con esta configuración y entrar al shell de emergencia.

   > [!NOTE]
   > Esta acción lo pone en el shell de emergencia con un sistema de archivos de solo lectura. Para editar los archivos, vuelva a montar el sistema de archivos con permisos de lectura y escritura. Para ello, escriba `mount -o remount,rw /` en el shell.

## <a name="access-for-oracle-linux"></a>Acceso para Oracle Linux
De forma similar a Red Hat Enterprise Linux, el modo de usuario único en Oracle Linux requiere que GRUB y el usuario raíz estén habilitados.

### <a name="grub-access-in-oracle-linux"></a>Acceso a GRUB en Oracle Linux
Oracle Linux viene con GRUB habilitado listo para su uso. Para entrar a GRUB, reinicie la máquina virtual con el comando `sudo reboot` y presione la tecla Esc. Esta acción muestra el panel de GRUB. Si no se muestra el panel de GRUB, asegúrese de que el valor de la línea `GRUB_TERMINAL` contiene *serial console* (es decir, la línea es `GRUB_TERMINAL="serial console"`). Vuelva a generar GRUB con `grub2-mkconfig -o /boot/grub/grub.cfg`.

### <a name="single-user-mode-in-oracle-linux"></a>Modo de usuario único en Oracle Linux
Para habilitar el modo de usuario único en Oracle Linux, siga las instrucciones anteriores para RHEL.

## <a name="next-steps"></a>Pasos siguientes
Para obtener más información acerca de la consola serie, consulte la siguiente información:
* [Documentación de la consola serie de Linux](serial-console-linux.md)
* [Uso de la consola serie para habilitar GRUB en varias distribuciones](http://linuxonazure.azurewebsites.net/why-proactively-ensuring-you-have-access-to-grub-and-sysrq-in-your-linux-vm-could-save-you-lots-of-down-time/)
* [Uso de la consola serie para llamadas NMI y SysRq](serial-console-nmi-sysrq.md)
* [Consola serie para máquinas virtuales Windows](serial-console-windows.md)
* [Diagnósticos de arranque](boot-diagnostics.md)
