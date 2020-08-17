---
title: Consola serie de Azure para Linux | Microsoft Docs
description: Consola serie bidireccional para máquinas virtuales y conjuntos de escalado de máquinas virtuales de Azure.
services: virtual-machines-linux
documentationcenter: ''
author: asinn826
manager: borisb
editor: ''
tags: azure-resource-manager
ms.service: virtual-machines-linux
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure-services
ms.date: 5/1/2019
ms.author: alsin
ms.openlocfilehash: cacb517c783416994fa95bd0f6a6d15a95a52ab4
ms.sourcegitcommit: e71da24cc108efc2c194007f976f74dd596ab013
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87423463"
---
# <a name="azure-serial-console-for-linux"></a>Consola serie de Azure para Linux

La consola serie de Azure Portal proporciona acceso a una consola basada en texto para las máquinas virtuales Linux y los conjuntos de escalado de máquinas virtuales. Esta conexión serie se conecta al puerto serie ttys0 de la máquina virtual o del conjunto de escalado de máquinas virtuales y ofrece acceso a ellos sin estar relacionada con el estado del sistema operativo o de la red. A la consola serie solo se puede acceder mediante Azure Portal; además, esta solo se permite a los usuarios que tienen un rol de acceso de colaborador o superior en el conjunto de escalado de máquinas virtuales o la máquina virtual.

La consola serie funciona de la misma manera para las máquinas virtuales y las instancias de conjunto de escalado de máquinas virtuales. En este documento, todas las menciones a las máquinas virtuales incluirán implícitamente las instancias de conjunto de escalado de máquinas virtuales, a menos que se indique lo contrario.

La consola serie está disponible con carácter general en regiones de Azure globales y en versión preliminar pública en Azure Government. Aún no está disponible en la nube de Azure en China.

Para la documentación sobre la consola serie para Windows, consulte [Consola serie para Windows](./serial-console-windows.md).

> [!NOTE]
> Actualmente, la consola serie no es compatible con una cuenta de almacenamiento de diagnósticos de arranque administrado. Para usar la consola serie, asegúrese de que está usando una cuenta de almacenamiento personalizada.

## <a name="prerequisites"></a>Requisitos previos

- La máquina virtual o la instancia de conjunto de escalado de máquinas virtuales deben usar el modelo de implementación de Resource Manager. No se admiten las implementaciones clásicas.

- La cuenta que usa una consola serie debe tener el [rol Colaborador de la máquina virtual](../../role-based-access-control/built-in-roles.md#virtual-machine-contributor) y la cuenta de almacenamiento de [diagnósticos de arranque](boot-diagnostics.md).

- La máquina virtual o la instancia de conjunto de escalado de máquinas virtuales deben tener un usuario con contraseña. Puede crear una con la función para [restablecer la contraseña](../extensions/vmaccess.md#reset-password) de la extensión de acceso de máquina virtual. Seleccione **Restablecer contraseña** en la sección **Soporte técnico y solución de problemas**.

- La máquina virtual o el conjunto de escalado de máquinas virtuales deben tener el [diagnóstico de arranque](boot-diagnostics.md) habilitado.

    ![Configuración de los diagnósticos de arranque](./media/virtual-machines-serial-console/virtual-machine-serial-console-diagnostics-settings.png)

- Para conocer la configuración específica de las distribuciones de Linux, consulte [Disponibilidad de distribuciones de Linux para la consola serie](#serial-console-linux-distribution-availability).

- La máquina virtual o el conjunto de escalado de máquinas virtuales debe configurarse para la salida de serie en `ttys0`. Este es el valor predeterminado para las imágenes de Azure, pero querrá volver a comprobar esto en imágenes personalizadas. Más detalles [a continuación](#custom-linux-images).


> [!NOTE]
> La consola serie requiere un usuario local con una contraseña configurada. Las máquinas virtuales o los conjuntos de escalado de máquinas virtuales configurados solo con una clave pública SSH no podrán iniciar sesión en la consola serie. Para crear un usuario local con contraseña, utilice la [extensión VMAccess](../extensions/vmaccess.md) (también está disponible en el portal al seleccionar **Restablecer contraseña**) y cree un usuario local con una contraseña.
> También puede restablecer la contraseña de administrador en su cuenta si [usa GRUB para iniciar el modo de usuario único](./serial-console-grub-single-user-mode.md).

## <a name="serial-console-linux-distribution-availability"></a>Disponibilidad de la distribución de Linux para la consola serie
Para que la consola serie funcione correctamente, el sistema operativo invitado debe configurarse para leer y escribir mensajes de la consola en el puerto serie. La mayoría de [distribuciones de Linux aprobadas por Azure](../linux/endorsed-distros.md) tienen la consola serie configurada de manera predeterminada. Si selecciona la opción **Consola serie** en la sección **Soporte técnico y solución de problemas** de Azure Portal, obtendrá acceso a la consola serie.

> [!NOTE]
> Si no ve nada en la consola serie, asegúrese de que el diagnóstico de arranque está habilitado en la máquina virtual. Presionar **Entrar** a menudo soluciona los problemas en los que no aparece nada en la consola serie.

Distribución      | Acceso a la consola serie
:-----------|:---------------------
Red Hat Enterprise Linux    | El acceso a la consola serie está habilitado de forma predeterminada.
CentOS      | El acceso a la consola serie está habilitado de forma predeterminada.
Debian      | El acceso a la consola serie está habilitado de forma predeterminada.
Ubuntu      | El acceso a la consola serie está habilitado de forma predeterminada.
CoreOS      | El acceso a la consola serie está habilitado de forma predeterminada.
SUSE        | Las imágenes de SLES más nuevas disponibles en Azure tienen habilitado el acceso a la consola serie de forma predeterminada. Si está utilizando versiones anteriores (10 o inferiores) de SLES en Azure, consulte el [artículo de KB](https://www.novell.com/support/kb/doc.php?id=3456486) para habilitar la consola serie.
Oracle Linux        | El acceso a la consola serie está habilitado de forma predeterminada.

### <a name="custom-linux-images"></a>Imágenes personalizadas de Linux
Para habilitar la consola serie para la imagen de VM de Linux personalizada, habilite el acceso de la consola en el archivo */etc/inittab* a fin de ejecutar un terminal en `ttyS0`. Por ejemplo: `S0:12345:respawn:/sbin/agetty -L 115200 console vt102`. También puede ser necesario generar un getty en ttyS0. Esto se puede hacer con `systemctl start serial-getty@ttyS0.service`.

También deberá agregar ttys0 como destino para la salida de serie. Para más información sobre cómo configurar una imagen personalizada para trabajar con la consola serie, consulte los requisitos generales del sistema en el artículo de [Creación y carga de un disco duro virtual Linux en Azure](https://aka.ms/createuploadvhd#general-linux-system-requirements).

Si está compilando un kernel personalizado, considere la posibilidad de habilitar las marcas de kernel `CONFIG_SERIAL_8250=y` y `CONFIG_MAGIC_SYSRQ_SERIAL=y`. El archivo de configuración se encuentra normalmente en la ruta de acceso */boot/* .

## <a name="common-scenarios-for-accessing-the-serial-console"></a>Escenarios habituales de acceso a la consola serie

Escenario          | Acciones en la consola serie
:------------------|:-----------------------------------------
Archivo *FSTAB* dañado | Presione la tecla **Entrar** para continuar y usar un editor de texto a fin de corregir el archivo *FSTAB*. Para ello, es posible que deba estar en modo de usuario único. Para más información, consulte la sección de la consola serie sobre [cómo solucionar problemas de fstab](https://support.microsoft.com/help/3206699/azure-linux-vm-cannot-start-because-of-fstab-errors) y [Use serial console to access GRUB and single user mode](serial-console-grub-single-user-mode.md) (Uso de la consola serie para acceder a GRUB y al modo de usuario único).
Reglas de firewall incorrectas |  Si ha configurado iptables para bloquear la conectividad SSH, puede usar la consola serie para interactuar con la máquina virtual sin necesidad de SSH. Más detalles en la [página del manual de iptables](https://linux.die.net/man/8/iptables).<br>De forma similar, si su firewalld bloquea el acceso SSH, puede acceder a la máquina virtual mediante la consola serie y volver a configurar el firewalld. Se puede encontrar más información en la [documentación del firewalld](https://firewalld.org/documentation/).
Comprobación o daños en el sistema de archivos | Consulte la sección de la consola serie de [Azure VM Linux no se puede iniciar debido a errores de sistema de archivos](https://support.microsoft.com/en-us/help/3213321/linux-recovery-cannot-ssh-to-linux-vm-due-to-file-system-errors-fsck) para más detalles sobre cómo solucionar sistemas de archivos dañados mediante la consola serie.
Problemas de configuración de SSH | Acceda a la consola serie y cambie la configuración. La consola serie puede usarse independientemente de la configuración de SSH de una máquina virtual, ya que no requiere que la máquina virtual tenga conectividad de red para que funcione. Hay una guía de solución de problemas disponible en [Troubleshoot SSH connections to an Azure Linux VM that fails, errors out, or is refused](./troubleshoot-ssh-connection.md) (Solución de problemas de conexión SSH a una máquina virtual Linux de Azure que producen error o se rechazan). Más detalles disponibles en [Detailed SSH troubleshooting steps for issues connecting to a Linux VM in Azure](./detailed-troubleshoot-ssh-connection.md) (Pasos detallados de solución de problemas de SSH para los problemas de conexión a una máquina virtual Linux en Azure).
Interacción con el cargador de arranque | Reinicie la máquina virtual desde dentro de la hoja de la consola serie para acceder a GRUB en la máquina virtual Linux. Para más detalles e información específica de la distribución, consulte [Use serial console to access GRUB and single user mode](serial-console-grub-single-user-mode.md) (Uso de la consola serie para acceder a GRUB y al modo de usuario único).

## <a name="disable-the-serial-console"></a>Deshabilitación de la consola serie

De forma predeterminada, todas las suscripciones tienen el acceso a la consola serie habilitado. Puede deshabilitar la consola serie a nivel de la suscripción o de la máquina virtual o del conjunto de escalado de máquinas virtuales. Para obtener instrucciones detalladas, visite [Habilitación y deshabilitación de la consola serie de Azure](./serial-console-enable-disable.md).

## <a name="serial-console-security"></a>Seguridad de la consola serie

### <a name="access-security"></a>Seguridad de acceso
El acceso a la consola serie está limitado a los usuarios que tienen un rol de acceso de [Colaborador de la máquina Virtual](../../role-based-access-control/built-in-roles.md#virtual-machine-contributor) o superior. Si el inquilino de Azure Active Directory requiere la autenticación multifactor (MFA), el acceso a la consola serie también requerirá MFA porque el acceso a la consola serie se realiza mediante [Azure Portal](https://portal.azure.com).

### <a name="channel-security"></a>Seguridad del canal
Todos los datos enviados y recibidos se cifran en la conexión.

### <a name="audit-logs"></a>Registros de auditoría
Todo el acceso a la consola serie queda registrado en los registros de los [diagnósticos de arranque](./boot-diagnostics.md) de la máquina virtual. El administrador de la máquina virtual de Azure es el propietario y el que controla el acceso a estos registros.

> [!CAUTION]
> Las contraseñas de acceso de la consola no se registran. Sin embargo, si los comandos ejecutados en la consola contienen o generan contraseñas, secretos, nombres de usuario o cualquier otra forma de información de identificación personal, se escribirán en los registros de diagnóstico de arranque de la máquina virtual. Se escribirán junto con el resto del texto visible, como parte de la implementación de la función de desplazamiento de la consola serie. Estos registros son circulares y solo tienen acceso a ellos aquellas personas con permisos de lectura a la cuenta de almacenamiento de diagnósticos. Si va a incluir comandos de datos que contengan secretos o información de identificación personal, se recomienda usar SSH a menos que la consola serie sea absolutamente necesaria.

### <a name="concurrent-usage"></a>Uso simultáneo
Si un usuario se conecta a la consola serie y otro usuario solicita correctamente acceso a esa misma máquina virtual, se desconectará el primer usuario y se conectará el segundo a la misma sesión.

> [!CAUTION]
> Esto significa que no se cerrará la sesión de un usuario que se haya desconectado. La posibilidad de forzar el cierre de sesión con la desconexión (mediante SIGHUP u otro mecanismo similar) sigue en proceso de valoración. En Windows hay un tiempo de espera automático habilitado en la consola administrativa especial (SAC); sin embargo, en Linux puede configurar el valor de tiempo de espera del terminal. Para ello, agregue `export TMOUT=600` en su archivo *.bash_profile* o *.profile* para el usuario que use para iniciar sesión en la consola. Esta configuración agotará el tiempo de espera de la sesión en 10 minutos.

## <a name="accessibility"></a>Accesibilidad
La accesibilidad es un factor clave de la consola serie de Azure. Por este motivo, hemos garantizado que la consola serie sea totalmente accesible.

### <a name="keyboard-navigation"></a>Navegación con el teclado
Use la tecla **Tabulador** del teclado para navegar por la interfaz de la consola serie en Azure Portal. La ubicación se resaltará en pantalla. Para salir del enfoque de la ventana de la consola serie, presione **Control**+**F6** en el teclado.

### <a name="use-serial-console-with-a-screen-reader"></a>Uso de la consola serie con un lector de pantalla
La consola serie tiene integrada la compatibilidad con el lector de pantalla. Si navega con el lector de pantalla activado, el lector de pantalla podrá leer en voz alta el texto alternativo del botón seleccionado actualmente.

## <a name="known-issues"></a>Problemas conocidos
Somos conscientes de que hay algunos problemas con la consola serie y el sistema operativo de la máquina virtual. Esta es una lista de dichos problemas y los pasos que puede realizar para corregirlos en máquinas virtuales Linux. Estos problemas y mitigaciones se aplican tanto a las máquinas virtuales como a las instancias de conjunto de escalado de máquinas virtuales. Si no coinciden con el error que observa, consulte los errores comunes de servicio de la consola serie en [Errores comunes en la consola serie](./serial-console-errors.md).

Incidencia                           |   Mitigación
:---------------------------------|:--------------------------------------------|
Al presionar **Entrar** después del banner de conexión no se muestra ninguna solicitud de inicio de sesión. | GRUB puede no estar configurado correctamente. Ejecute los comandos siguientes: `grub2-mkconfig -o /etc/grub2-efi.cfg` y/o `grub2-mkconfig -o /etc/grub2.cfg`. Para obtener más información, consulte [Hitting enter does nothing](https://github.com/Microsoft/azserialconsole/blob/master/Known_Issues/Hitting_enter_does_nothing.md) (Al presionar Entrar, no se realiza ninguna acción). Este problema puede ocurrir si está ejecutando una máquina virtual personalizada, un dispositivo protegido o una configuración de GRUB que hace que Linux no pueda conectarse al puerto serie.
El texto de la consola serie solo ocupa una parte de la pantalla (a menudo después de usar un editor de texto). | Las consolas serie no admiten operaciones para cambiar el tamaño de la ventana ([RFC 1073](https://www.ietf.org/rfc/rfc1073.txt)), lo que significa que no se enviará ninguna señal SIGWINCH para actualizar el tamaño de la pantalla y la máquina virtual no tendrá conocimiento del tamaño del terminal. Instale xterm o una utilidad similar que le proporcione el comando `resize` y, a continuación, ejecute `resize`.
El hecho de pegar cadenas largas no funciona. | La consola serie limita la longitud de las cadenas pegadas en el terminal a 2048 caracteres a fin de impedir que el ancho de banda del puerto serie se sobrecargue.
Entradas de teclado erráticas en las imágenes de SLES BYOS. La entrada mediante el teclado solo se reconoce esporádicamente. | Se trata de un problema con el paquete de Plymouth. No se debe ejecutar Plymouth en Azure, ya que no necesita una pantalla de presentación y Plymouth afecta la capacidad de la plataforma para usar la consola serie. Quite Plymouth con `sudo zypper remove plymouth` y reinicie el equipo. Como alternativa, modifique la línea de kernel de la configuración GRUB anexando `plymouth.enable=0` al final de esta. Para ello, puede [editar la entrada de arranque en el tiempo de arranque](https://aka.ms/serialconsolegrub#single-user-mode-in-suse-sles) o editar la línea GRUB_CMDLINE_LINUX en `/etc/default/grub`, reconstruir GRUB con `grub2-mkconfig -o /boot/grub2/grub.cfg` y reiniciar el equipo.


## <a name="frequently-asked-questions"></a>Preguntas más frecuentes

**P. ¿Cómo puedo enviar comentarios?**

A. Proporcione comentarios mediante la creación de un problema de GitHub en https://aka.ms/serialconsolefeedback. También tiene la posibilidad (aunque no es tan recomendable) de enviar comentarios a través de azserialhelp@microsoft.com o en la categoría de la máquina virtual de https://feedback.azure.com.

**P. ¿La consola serie admite las operaciones de copiar y pegar?**

A. Sí. Use **Control**+**Mayús**+**C** y **Control**+**Mayús**+**V** para copiar y pegar contenido en el terminal.

**P. ¿Puedo usar la consola serie en lugar de una conexión SSH?**

A. Aunque este uso puede parecer técnicamente posible, la consola serie está pensada para usarse principalmente como una herramienta de solución de problemas en situaciones donde no es posible la conectividad a través de SSH. Se recomienda no usar la consola serie como un reemplazo de SSH por los motivos siguientes:

- La consola serie no tiene el ancho de banda que tiene SSH. Dado que es una conexión de solo texto, las interacciones de GUI más intensivas son complicadas.
- Actualmente, solo se puede acceder a la consola serie con el nombre de usuario y la contraseña. Las claves SSH son mucho más seguras que las combinaciones de nombre de usuario y contraseña; así que, desde una perspectiva de seguridad de inicio de sesión, SSH es más recomendable que la consola serie.

**P. ¿Quién puede habilitar o deshabilitar la consola serie en mi suscripción?**

A. Para habilitar o deshabilitar la consola serie en toda una suscripción, es preciso tener permisos de escritura en la suscripción. Los roles que tienen permiso de escritura son, entre otros, los roles de administrador o propietario. Los roles personalizados también pueden tener permisos de escritura.

**P. ¿Quién puede acceder a la consola serie de mi máquina virtual o conjunto de escalado de máquinas virtuales?**

A. Para acceder a la consola serie, debe tener el rol Colaborador o superior para una máquina virtual o conjunto de escalado de máquinas virtuales.

**P. La consola serie no muestra nada, ¿qué hago?**

A. Es probable que la imagen esté mal configurada para el acceso a la consola serie. Consulte [Disponibilidad de distribuciones de Linux para la consola serie](#serial-console-linux-distribution-availability) para obtener información sobre la configuración de la imagen a fin de habilitar la consola serie.

**P. ¿Está disponible la consola serie en los conjuntos de escalado de máquinas virtuales?**

A. Sí. Consulte [Serial Console for Virtual Machine Scale Sets](serial-console-overview.md#serial-console-for-virtual-machine-scale-sets) (Consola serie para los conjuntos de escalado de máquinas virtuales).

**P. Si configuro mi máquina virtual o conjunto de escalado de máquinas virtuales usando solo la autenticación de clave SSH, ¿puedo seguir usando la consola serie para conectarme a mi máquina virtual o conjunto de escalado de máquinas virtuales?**

A. Sí. La consola serie no requiere claves SSH, por lo que lo único que debe hacer es configurar una combinación de nombre de usuario y contraseña. Para ello, seleccione la opción **Restablecer contraseña** en Azure Portal y use esas credenciales para iniciar sesión en la consola serie.

## <a name="next-steps"></a>Pasos siguientes
* Uso de la consola serie para [acceder a GRUB y al modo de usuario único](serial-console-grub-single-user-mode.md).
* Uso de la consola serie para [llamadas NMI y SysRq](serial-console-nmi-sysrq.md).
* Aprenda a usar la consola serie para [habilitar GRUB en varias distribuciones](serial-console-grub-proactive-configuration.md).
* La consola serie también está disponible para las [VM Windows](./serial-console-windows.md).
* Obtenga más información sobre [diagnósticos de arranque](boot-diagnostics.md).
