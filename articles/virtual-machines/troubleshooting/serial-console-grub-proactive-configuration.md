---
title: Configuración proactiva de GRUB en la consola serie de Azure | Microsoft Docs
description: Configure GRUB en diversas distribuciones que permiten el acceso de un solo usuario y el modo de recuperación en máquinas virtuales de Azure.
services: virtual-machines-linux
documentationcenter: ''
author: mimckitt
manager: vashan
editor: ''
tags: azure-resource-manager
ms.service: virtual-machines-linux
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure-services
ms.date: 07/10/2019
ms.author: mimckitt
ms.openlocfilehash: c48ef0321ece2e7e0ffcdfcb8c0907c5f839e738
ms.sourcegitcommit: 2ff0d073607bc746ffc638a84bb026d1705e543e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87831369"
---
# <a name="proactively-ensuring-you-have-access-to-grub-and-sysrq-could-save-you-lots-of-down-time"></a>Garantizar de forma proactiva el acceso a GRUB y a SysRq podría ahorrarle mucho tiempo de inactividad.

El acceso a la consola serie y a GRUB mejorará los tiempos de recuperación de la máquina virtual IaaS con Linux en la mayoría de los casos. GRUB ofrece opciones de recuperación que, de lo contrario, se tardaría más tiempo en recuperar la máquina virtual. 


Los motivos para realizar una recuperación de máquina virtual son muchos y se pueden atribuir a escenarios como los siguientes:

   - Sistemas de archivos, kernel o MBR (registro de arranque maestro) dañados
   - Actualizaciones de kernel erróneas
   - Parámetros de kernel de GRUB incorrectos
   - Configuraciones de fstab incorrectas
   - Configuraciones de firewall
   - Contraseña perdida
   - Archivos de configuración de sshd alterados
   - Configuraciones de red

 Muchos otros escenarios, tal como se detalla [aquí](./serial-console-linux.md#common-scenarios-for-accessing-the-serial-console).

Compruebe que puede acceder a GRUB y a la consola serie en las máquinas virtuales implementadas en Azure. 

Si no está familiarizado con la consola serie, consulte [este vínculo](./serial-console-linux.md).

> [!TIP]
> Asegúrese de realizar copias de seguridad de los archivos antes de realizar cambios.

Vea el vídeo siguiente para saber cómo puede recuperar rápidamente la máquina virtual Linux una vez que tenga acceso a GRUB.

[Vídeo de recuperación de máquinas virtuales Linux](https://youtu.be/KevOc3d_SG4)

Hay una serie de métodos para ayudar a recuperar las máquinas virtuales Linux. En un entorno de nube, este proceso ha sido desafiante.
El progreso se realiza continuamente en herramientas y características para asegurarse de que los servicios se recuperan rápidamente.

Con la consola serie de Azure, puede interactuar con la máquina virtual Linux como si estuviera en la consola del sistema.

Puede manipular muchos archivos de configuración, incluido el modo en que se iniciará el kernel. 

Los administradores de sistemas Linux e Unix más experimentados apreciarán los **modos de usuario único** y **de emergencia** que son accesibles a través de la consola serie de Azure, lo que permite el intercambio de discos y la eliminación de máquinas virtuales en muchos escenarios de recuperación redundantes.

El método de recuperación depende del problema que se esté experimentando, por ejemplo, una contraseña perdida o mal colocada se puede restablecer con las opciones de Azure Portal -> **Restablecer contraseña**. La característica **Restablecer contraseña** se conoce como extensión y se comunica con el agente invitado de Linux.

Están disponibles otras extensiones como el script personalizado, pero estas opciones requieren que el **waagent** de Linux esté activo y en un estado correcto, lo que no siempre es el caso.

![estado del agente](./media/virtual-machines-serial-console/agent-status.png)


Asegurarse de que se tiene acceso a la consola serie de Azure y a GRUB significa que se puede rectificar un cambio de contraseña o una configuración incorrecta en cuestión de minutos, en lugar de en horas. Incluso podría forzar a la máquina virtual a iniciarse desde un kernel alternativo si tiene varios kernels en el disco en el escenario en el que el kernel principal está dañado.

![kernel múltiple](./media/virtual-machines-serial-console/more-kernel.png)

## <a name="suggested-order-of-recovery-methods"></a>Orden sugerido de métodos de recuperación:

- Consola serie de Azure

- Intercambio de disco, que se puede automatizar mediante:

   - [Scripts de recuperación de PowerShell](https://github.com/Azure/azure-support-scripts/tree/master/VMRecovery/ResourceManager)
   - [Scripts de recuperación de Bash](https://github.com/sribs/azure-support-scripts)

- Método heredado

## <a name="disk-swap-video"></a>Vídeo de intercambio de discos:

Si no tiene acceso a GRUB, vea en [este](https://youtu.be/m5t0GZ5oGAc) vídeo cómo puede automatizar fácilmente el procedimiento de intercambio de discos para recuperar la máquina virtual

## <a name="challenges"></a>Desafíos:

No todas las máquinas virtuales Linux de Azure están configuradas de forma predeterminada para el acceso de GRUB y ninguna está configurada para que se interrumpan con los comandos SysRq. Algunos distribuciones anteriores, como SLES 11, no están configuradas para mostrar el mensaje de inicio de sesión en la consola serie de Azure.

En este artículo, revisaremos varias configuraciones de documentos y distribuciones de Linux sobre cómo hacer que GRUB esté disponible.




## <a name="how-to-configure-linux-vm-to-accept-sysrq-keys"></a>Configuración de la máquina virtual Linux para aceptar claves SysRq
La clave SysRq está habilitada en algunas distribuciones de Linux más recientes de forma predeterminada, aunque en otras puede estar configurada para aceptar valores solo para determinadas funciones de SysRq.
En distribuciones anteriores, podría deshabilitarse por completo.

La característica SysRq es útil para reiniciar una máquina virtual bloqueada o que no responde directamente desde la consola serie de Azure, también es útil para obtener acceso al menú de GRUB, o bien para reiniciar una máquina virtual desde otra ventana del portal. De lo contrario, la sesión ssh podría anular la conexión de la consola actual, con lo que caducarían los tiempos de expiración de GRUB que se usan para mostrar el menú de GRUB.
La máquina virtual debe estar configurada para aceptar un valor de 1 para el parámetro kernel, que permite todas las funciones de SysRq, o un valor de 128, que permite el reinicio o apagado.


[Vídeo sobre la habilitación de SysRq](https://youtu.be/0doqFRrHz_Mc)


Para configurar la máquina virtual para que acepte un reinicio a través de los comandos de SysRq en Azure Portal, debe establecer un valor de 1 para el parámetro del kernel kernel.sysrq.

Para que esta configuración requiera un reinicio, agregue una entrada al archivo **sysctl.conf**.

`echo kernel.sysrq = 1 >> /etc/sysctl.conf`

Para configurar el parámetro de kernel dinámicamente

`sysctl -w kernel.sysrq=1`

Si no tiene el acceso **raíz** o se interrumpe el acceso a sudo, no será posible configurar SysRq desde un símbolo del sistema de shell.

Puede habilitar SysRq en este escenario mediante Azure Portal. Este método puede ser beneficioso si el archivo **sudors.d/waagent** se ha interrumpido o se ha eliminado.

Con la característica de Azure Portal Operaciones -> Ejecutar comando -> RunShellScript, se requiere que el proceso waagent esté en buen estado para insertar este comando y habilitar SysRq.

`sysctl -w kernel.sysrq=1 ; echo kernel.sysrq = 1 >> /etc/sysctl.conf`

Tal como se muestra aquí: ![habilitar sysrq2](./media/virtual-machines-serial-console/enabling-sysrq-2.png)

Una vez completado, puede intentar acceder a **SysRq** y debería ver que es posible reiniciar.

![habilitar sysrq3](./media/virtual-machines-serial-console/enabling-sysrq-3.png)

Seleccione **Reiniciar** y el comando **Send SysRq**.

![habilitar sysrq4](./media/virtual-machines-serial-console/enabling-sysrq-4.png)

El sistema debe registrar un mensaje de restablecimiento como este:

![habilitar sysrq5](./media/virtual-machines-serial-console/enabling-sysrq-5.png)


## <a name="ubuntu-grub-configuration"></a>Configuración de GRUB de Ubuntu

De forma predeterminada, para tener acceso a GRUB, mantenga presionada la tecla **Esc** durante el arranque de la máquina virtual. Si no aparece el menú de GRUB, puede forzar y mantener el menú de GRUB en la pantalla de la consola serie de Azure mediante una de estas opciones.

**Opción 1**: Forzar a GRUB a mostrarse en la pantalla 

Actualice el archivo /etc/default/grub.d/50-cloudimg-settings.cfg para mantener el menú de GRUB en la pantalla durante el tiempo de espera especificado.
No es necesario que presione **Esc**, ya que GRUB se mostrará inmediatamente.

```
GRUB_TIMEOUT=0

change to

GRUB_TIMEOUT=5
```

**Opción 2**: Permitir que se presione **Esc** antes del arranque

Se puede experimentar un comportamiento similar si se realizan cambios en el archivo /etc/default/grub y se reserva un tiempo de espera de tres segundos para presionar **Esc**.


Comente estas dos líneas:

```
#GRUB_HIDDEN_TIMEOUT=0
#GRUB_HIDDEN_TIMEOUT_QUIET=true
```
y agregue esta línea:

```
GRUB_TIMEOUT_STYLE=countdown
```


## <a name="ubuntu-1204"></a>Ubuntu 12\.04

Ubuntu 12.04 permitirá el acceso a la consola serie, pero no ofrece la capacidad de interactuar. No se ve un aviso **login:** .


Para que la versión 12.04 obtenga el aviso **login:**
1. Cree un archivo denominado /etc/init/ttyS0.conf que contenga el texto siguiente:

    ```
    # ttyS0 - getty
    #
    # This service maintains a getty on ttyS0 from the point the system is
    # started until it is shut down again.
    start on stopped rc RUNLEVEL=[12345]
    stop on runlevel [!12345]
    
    respawn
    exec /sbin/getty -L 115200 ttyS0 vt102
    ```    

2. Pida a upstart que inicie el comando getty:     
    ```
    sudo start ttyS0
    ```
 
[Aquí](https://help.ubuntu.com/community/SerialConsoleHowto) puede encontrar los ajustes necesarios para configurar la consola serie para las versiones de Ubuntu.

## <a name="ubuntu-recovery-mode"></a>Modo de recuperación de Ubuntu

Existen opciones de recuperación y limpieza disponibles para Ubuntu mediante GRUB; sin embargo, solo se puede acceder a ellas si se configuran los parámetros del kernel en consecuencia.
Si no se configura este parámetro de arranque de kernel, se forzará el envío del menú de recuperación a Azure Diagnostics y no a la consola serie de Azure.
Puede obtener acceso al menú de recuperación de Ubuntu con estos pasos:

Interrumpa el proceso de arranque y acceda al menú de GRUB.

Seleccione Advanced Options for Ubuntu (Opciones avanzadas para Ubuntu) y presione Entrar.

![ubunturec1](./media/virtual-machines-serial-console/ubunturec1.png)

Seleccione la línea que muestra *(recovery mode)* [(modo de recuperación)], pero no presione Entrar sino "e".

![ubunturec2](./media/virtual-machines-serial-console/ubunturec2.png)

Busque la línea que cargará el kernel y sustituya el último parámetro **nomodeset** con destino como **console=ttyS0**.

```
linux /boot/vmlinuz-4.15.0-1023-azure root=UUID=21b294f1-25bd-4265-9c4e-d6e4aeb57e97 ro recovery nomodeset

change to

linux /boot/vmlinuz-4.15.0-1023-azure root=UUID=21b294f1-25bd-4265-9c4e-d6e4aeb57e97 ro recovery console=ttyS0
```

![ubunturec3](./media/virtual-machines-serial-console/ubunturec3.png)

Presione **Ctrl-x** para iniciar y cargar el kernel.
Si todo va bien, verá estas otras opciones, lo que puede ayudar a realizar otras opciones de recuperación.

![ubunturec4](./media/virtual-machines-serial-console/ubunturec4.png)


## <a name="red-hat-grub-configuration"></a>Configuración de GRUB para Red Hat

## <a name="red-hat-74-grub-configuration"></a>Configuración de GRUB para Red Hat 7\.4\+
La configuración predeterminada de /etc/default/grub en estas versiones está configurada correctamente.

```
GRUB_TIMEOUT=5
GRUB_DISTRIBUTOR="$(sed 's, release .*$,,g' /etc/system-release)"
GRUB_DEFAULT=saved
GRUB_DISABLE_SUBMENU=true
GRUB_TERMINAL="serial console"
GRUB_SERIAL_COMMAND="serial"
GRUB_CMDLINE_LINUX="console=tty1 console=ttyS0 earlyprintk=ttyS0 rootdelay=300"
GRUB_DISABLE_RECOVERY="true"
```

Habilitación de la clave SysRq

```
sysctl -w kernel.sysrq=1;echo kernel.sysrq = 1 >> /etc/sysctl.conf;sysctl -a | grep -i sysrq
```

## <a name="red-hat-72-and-73-grub-configuration"></a>Configuración de GRUB para Red Hat 7\.2 y 7\.3
El archivo que se va a modificar es /etc/default/grub, una configuración predeterminada es similar a la de este ejemplo:

```
GRUB_TIMEOUT=1
GRUB_DISTRIBUTOR="$(sed 's, release .*$,,g' /etc/system-release)"
GRUB_DEFAULT=saved
GRUB_DISABLE_SUBMENU=true
GRUB_TERMINAL_OUTPUT="console"
GRUB_CMDLINE_LINUX="console=tty1 console=ttyS0 earlyprintk=ttyS0 rootdelay=300"
GRUB_DISABLE_RECOVERY="true"
```

Cambie las líneas siguientes en /etc/default/grub.

```
GRUB_TIMEOUT=1 

to

GRUB_TIMEOUT=5
```


```
GRUB_TERMINAL_OUTPUT="console"

to

GRUB_TERMINAL="serial console"
```

También agregue esta línea:

```
GRUB_SERIAL_COMMAND=”serial –speed=115200 –unit=0 –word=8 –parity=no –stop=1″
```

/etc/default/grub debe ser similar a este ejemplo:

```
GRUB_TIMEOUT=5
GRUB_DISTRIBUTOR="$(sed 's, release .*$,,g' /etc/system-release)"
GRUB_DEFAULT=saved
GRUB_DISABLE_SUBMENU=true
GRUB_TERMINAL="serial console"
GRUB_CMDLINE_LINUX="console=tty1 console=ttyS0 earlyprintk=ttyS0 rootdelay=300"
GRUB_DISABLE_RECOVERY="true"
```
 
Complete y actualice la configuración de GRUB con

`grub2-mkconfig -o /boot/grub2/grub.cfg`

Establezca el parámetro de kernel de SysRq:

`sysctl -w kernel.sysrq = 1;echo kernel.sysrq = 1 >> /etc/sysctl.conf;sysctl -a | grep -i sysrq`

También puede configurar GRUB y SysRq con una sola línea, ya sea en el shell o a través del comando Run. Haga una copia de seguridad de los archivos antes de ejecutar este comando:


`cp /etc/default/grub /etc/default/grub.bak; sed -i 's/GRUB_TIMEOUT=1/GRUB_TIMEOUT=5/g' /etc/default/grub; sed -i 's/GRUB_TERMINAL_OUTPUT="console"/GRUB_TERMINAL="serial console"/g' /etc/default/grub; echo "GRUB_SERIAL_COMMAND=\"serial --speed=115200 --unit=0 --word=8 --parity=no --stop=1\"" >> /etc/default/grub;grub2-mkconfig -o /boot/grub2/grub.cfg;sysctl -w kernel.sysrq=1;echo kernel.sysrq = 1 /etc/sysctl.conf;sysctl -a | grep -i sysrq`


## <a name="red-hat-6x-grub-configuration"></a>Configuración de GRUB para Red Hat 6\.x
El archivo que se va a modificar es /boot/grub/grub.conf. El valor `timeout` determinará durante cuánto tiempo se mostrará GRUB.

```
#boot=/dev/vda
default=0
timeout=15
splashimage=(hd0,0)/grub/splash.xpm.gz
#hiddenmenu
serial --unit=0 --speed=9600
terminal serial
terminal --timeout=5 serial console
```


La última línea *terminal –-timeout=5 serial console* aumentará aún más el tiempo de espera de **GRUB** al agregar una solicitud de cinco segundos que muestra **Presione cualquier tecla para continuar**.

![rh6-1](./media/virtual-machines-serial-console/rh6-1.png)

El menú de GRUB debe aparecer en pantalla durante el tiempo configurado timeout=15 sin necesidad de presionar Esc. Asegúrese de hacer clic en la consola del explorador para activar el menú y seleccionar el kernel necesario.

![rh6-2](./media/virtual-machines-serial-console/rh6-2.png)

## <a name="suse"></a>SuSE

## <a name="sles-12-sp1"></a>SLES 12 sp1
Use el cargador de arranque YaST según los [documentos](./serial-console-grub-single-user-mode.md#grub-access-in-suse-sles) oficiales.

O bien, agregue o cambie a /etc/default/grub los siguientes parámetros:

```
GRUB_TERMINAL=serial
GRUB_TIMEOUT=5
GRUB_SERIAL_COMMAND="serial --unit=0 --speed=9600 --parity=no"

```
Compruebe que ttyS0 se usa en GRUB_CMDLINE_LINUX o GRUB_CMDLINE_LINUX_DEFAULT.

```
GRUB_CMDLINE_LINUX_DEFAULT="console=ttyS0,9600n"
```

Vuelva a crear el archivo grub.cfg.

`grub2-mkconfig -o /boot/grub2/grub.cfg`


## <a name="sles-11-sp4"></a>SLES 11 SP4 
Aparece la consola serie y muestra los mensajes de arranque, pero no muestra el aviso **login:** .

Abra una sesión de ssh en la máquina virtual y actualice el archivo **/etc/inittab** sin comentar esta línea:

```
#S0:12345:respawn:/sbin/agetty -L 9600 ttyS0 vt102
```

A continuación, ejecute el comando 

`telinit q`

Para habilitar GRUB, se deben realizar los siguientes cambios en /boot/grub/menu.lst. 

```
timeout 5
serial --unit=0 --speed=9600 --parity=no
terminal --timeout=5 serial console

root (hd0,0)
kernel /boot/vmlinuz-3.0.101-108.74-default root=/dev/disk/by-uuid/ab6b62bb--
1a8c-45eb-96b1-1fbc535b9265 disk=/dev/sda  USE_BY_UUID_DEVICE_NAMES=1 earlyprinttk=ttyS0 console=ttyS0 rootdelay=300  showopts vga=0x314
```

 Esta configuración habilitará que el mensaje **Presione cualquier tecla para continuar** aparezca en la consola durante cinco segundos. 

Después, se mostrará el menú de GRUB durante otros cinco segundos; si presiona la flecha abajo, interrumpirá el contador y podrá seleccionar el kernel que desee iniciar ya sea anexando la palabra clave **single** para el modo de usuario único que requiere que se establezca la contraseña raíz.

Anexar el comando **init=/bin/bash** cargará el kernel, pero garantizará que el programa init se reemplace por un shell de Bash.

Obtendrá acceso al shell sin tener que escribir una contraseña. Después, puede continuar con la actualización de la contraseña para las cuentas de Linux o realizar otros cambios en la configuración.


## <a name="force-the-kernel-to-a-bash-prompt"></a>Forzar el kernel a un símbolo del sistema de Bash
El acceso a GRUB permite interrumpir el proceso de inicialización y esta interacción resulta útil para muchos procedimientos de recuperación.
Si no tiene la contraseña raíz y el usuario único requiere esa contraseña, puede arrancar el kernel reemplazando el programa init con un símbolo del sistema de Bash: esta interrupción se puede lograr mediante la anexión de init=/bin/bash a la línea de arranque del kernel.

![bash1](./media/virtual-machines-serial-console/bash1.png)

Vuelva a montar el RW del sistema de archivos / (root) con el comando

`mount -o remount,rw /`

![bash2](./media/virtual-machines-serial-console/bash2.png)


Ahora puede realizar el cambio de contraseña raíz o muchos otros cambios de configuración de Linux.

![bash3](./media/virtual-machines-serial-console/bash3.png)

Reinicie la máquina virtual con 

`/sbin/reboot -f`




## <a name="single-user-mode"></a>Modo de usuario único

Como alternativa, puede que necesite acceder a la máquina virtual en modo de emergencia o de usuario único. Seleccione el kernel que desea arrancar o interrumpir con las teclas de dirección.
Escriba el modo deseado mediante la anexión de la palabra clave **single** o **1** a la línea de arranque del kernel. En los sistemas RHEL, también puede anexar **rd.break**.

Para más información sobre cómo acceder al modo de usuario único, consulte [este documento](./serial-console-grub-single-user-mode.md#general-single-user-mode-access). 


![single_user_ubuntu](./media/virtual-machines-serial-console/single-user-ubuntu.png)


## <a name="next-steps"></a>Pasos siguientes
Obtenga más información sobre la [consola serie de Azure]( ./serial-console-linux.md).