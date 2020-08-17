---
title: Consola serie de Azure para llamadas SysRq y NMI | Microsoft Docs
description: Uso de la consola serie para llamadas SysRq y NMI en máquinas virtuales de Azure.
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
ms.date: 08/14/2018
ms.author: alsin
ms.openlocfilehash: 545399e1d7941351ce861ac98d995d5e57006ea1
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87074366"
---
# <a name="use-the-azure-serial-console-for-sysrq-and-nmi-calls"></a>Uso de la consola serie de Azure para llamadas SysRq y NMI

## <a name="system-request-sysrq"></a>Solicitud del sistema (SysRq)
Una solicitud del sistema, o SysRq, es una secuencia de claves que entiende el kernel del sistema de operaciones Linux, que puede desencadenar un conjunto de acciones predefinidas. Estos comandos se usan a menudo cuando la solución de problemas o la recuperación de una máquina virtual no pueden realizarse mediante la administración tradicional (por ejemplo, si la máquina virtual no responde). Mediante la característica SysRq de la consola serie de Azure se imita la acción de presionar la tecla SysRq y los caracteres escritos en un teclado físico.

Una vez que se entrega la secuencia SysRq, la configuración del kernel controla cómo responde el sistema. Para información sobre cómo habilitar y deshabilitar SysRq, consulte la *guía del administrador de SysRq*[texto](https://aka.ms/kernelorgsysreqdoc) | [markdown](https://aka.ms/linuxsysrq).

La consola serie de Azure se puede usar para enviar un comando SysRq a una máquina virtual de Azure mediante el icono de teclado de la barra de comandos que se muestra a continuación.

![Captura de pantalla de la consola serie de Azure. El icono de teclado se resalta y su menú está visible. Ese menú contiene un elemento Enviar comando SysRq.](../media/virtual-machines-serial-console/virtual-machine-serial-console-command-menu.jpg)

Al elegir "Send SysRq Command" (Enviar comando SysRq) se abre un cuadro de diálogo que proporciona opciones comunes de SysRq o que acepta una secuencia de comandos SysRq escritos en el cuadro de diálogo.  Esto permite que la serie de comandos SysRq realice una operación de alto nivel como un reinicio seguro mediante: `REISUB`.

![Captura de pantalla del cuadro de diálogo Enviar comando SysRq al invitado. Se selecciona la opción para escribir comandos y el cuadro de comandos contiene REISUB.](../media/virtual-machines-serial-console/virtual-machine-serial-console-sysreq_UI.png)

No se puede usar el comando SysRq en máquinas virtuales que estén detenidas o cuyo kernel se encuentre en un estado sin respuesta (por ejemplo, una alerta de pánico del kernel).

### <a name="enable-sysrq"></a>Habilitación de SysRq
Como se describe en la *guía del administrador de SysRq* mencionada anteriormente, SysRq se puede configurar de forma que todos, ninguno o solo ciertos comandos estén disponibles. Puede habilitar todos los comandos SysRq mediante el paso siguiente, pero no sobrevivirán a un reinicio:
```
echo "1" >/proc/sys/kernel/sysrq
```
Para conservar la configuración de SysReq, puede hacer lo siguiente para habilitar todos los comandos SysRq:
1. Agregar esta línea a */etc/sysctl.conf* <br>
    `kernel.sysrq = 1`
1. Reiniciar o actualizar sysctl mediante la ejecución de <br>
    `sysctl -p`

### <a name="command-keys"></a>Teclas de comando
En la guía del administrador de SysRq anterior:

|Get-Help| Función
| ------| ----------- |
|``b``  |   Reiniciará inmediatamente el sistema sin necesidad de sincronizar o desmontar los discos.
|``c``  |   Llevará a cabo un bloqueo del sistema mediante una desreferencia del puntero NULL. Si se configura, se realizará un volcado.
|``d``  |   Muestra todos los bloqueos mantenidos.
|``e``  |   Enviar una señal SIGTERM a todos los procesos, excepto init.
|``f``  |   Llamará a OOM Killer para terminar un proceso de acumulación de memoria, pero no se alarme si no se puede eliminar nada.
|``g``  |   La usa kgdb (depurador de kernel).
|``h``  |   Mostrará ayuda (cualquier otra tecla aparte de las mostradas aquí también mostrará ayuda, pero ``h`` es fácil de recordar :-)
|``i``  |    Enviar una señal SIGKILL a todos los procesos, excepto init.
|``j``  |    Reanudar a la fuerza los sistemas de archivos bloqueados por el IOCTL FIFREEZE.
|``k``  |    La clave de acceso seguro (SAK) termina todos los programas en la consola virtual actual. NOTA:  Vea comentarios importantes más abajo en la sección SAK.
|``l``  |    Muestra un seguimiento regresivo de pila para todas las CPU activas.
|``m``  |    Volcará información actual de memoria a la consola.
|``n``  |    Se usa para realizar tareas de RT compatibles con Nice.
|``o``  |    Apagará el sistema (si está configurado y se admite).
|``p``  |    Volcará los registros y marcas actuales en la consola.
|``q``  |    Volcará listas de todos los temporizadores de alta resolución armados (hrtimer) por CPU (pero NO los temporizadores timer_list normales) e información detallada sobre todos los dispositivos de eventos de reloj.
|``r``  |    Desactiva el modo RAW de teclado y lo establece en XLATE.
|``s``  |    Intentará sincronizar todos los sistemas de archivos montados.
|``t``  |    Volcará una lista de tareas actuales y su información en la consola.
|``u``  |    Intentará volver a montar todos los sistemas de archivos montados de solo lectura.
|``v``  |    Restaura forzosamente la consola de framebuffer.
|``v``  |    Provoca el volcado de memoria del búfer de ETM [específico de ARM].
|``w``  |    Volcar tareas que se encuentran en estado (bloqueado) ininterrumpido.
|``x``  |    La usa la interfaz de XMON en plataformas ppc/powerpc. Mostrar registros de PMU globales en sparc64. Volcar todas las entradas TLB en MIPS.
|``y``  |    Mostrar registros de CPU globales [específicos de SPARC 64].
|``z``  |    Volcar el búfer de ftrace.
|``0``-``9`` | Establece el nivel de registro de la consola, y controla qué mensajes del kernel se imprimen en la consola. (Por ejemplo, ``0`` haría que solo los mensajes de emergencia, como alertas de pánico o problemas se volcaran en la consola).

### <a name="distribution-specific-documentation"></a>Documentación específica de la distribución ###
Para obtener documentación específica de la distribución sobre SysRq y pasos para configurar Linux para crear un volcado de memoria cuando se recibe un comando de bloqueo de SysRq, consulte los siguientes vínculos:

#### <a name="ubuntu"></a>Ubuntu ####
 - [Kernel Crash Dump](https://help.ubuntu.com/lts/serverguide/kernel-crash-dump.html) (Volcado de memoria del kernel)

#### <a name="red-hat"></a>Red Hat ####
- [What is the SysRq Facility and how do I use it?](https://access.redhat.com/articles/231663) (¿Qué es la utilidad SysRq y cómo usarla?)
- [How to use the SysRq facility to collect information from a RHEL server](https://access.redhat.com/solutions/2023) (¿Cómo usar la utilidad SysRq para recopilar información de un servidor de RHEL?)

#### <a name="suse"></a>SUSE ####
- [Configure kernel core dump capture](https://www.suse.com/support/kb/doc/?id=3374462) (Configuración de la captura de volcado de núcleo del kernel)

#### <a name="coreos"></a>CoreOS ####
- [Collecting crash logs](https://coreos.com/os/docs/latest/collecting-crash-logs.html) (Recopilación de registros de bloqueo)

## <a name="non-maskable-interrupt-nmi"></a>Interrupción no enmascarable (NMI)
Una interrupción no enmascarable (NMI) está diseñada para crear una señal que el software de una máquina virtual no pasará por alto. Históricamente, las NMI se han usado para supervisar problemas de hardware en sistemas que necesitaban tiempos de respuesta específicos.  En la actualidad, los administradores del sistema y los programadores suelen usar NMI como mecanismo para depurar o solucionar los problemas de sistemas que no responden.

La consola serie se puede usar para enviar una NMI a una máquina virtual de Azure mediante el icono de teclado de la barra de comandos que se muestra a continuación. Una vez que se entrega la NMI, la configuración de la máquina virtual controla cómo responde el sistema.  Los sistemas operativos Linux se pueden configurar para bloquearse y crear un volcado de memoria cuando el sistema operativo recibe una NMI.

![Captura de pantalla de la consola serie. El icono de teclado se resalta y su menú está visible. Ese menú contiene un elemento Enviar interrupción no enmascarable.](../media/virtual-machines-serial-console/virtual-machine-serial-console-command-menu.jpg) <br>

### <a name="enable-nmi"></a>Habilitar NMI
En el caso de sistemas Linux que admiten sysctl para configurar parámetros del kernel, puede habilitar una alerta de pánico al recibir esta NMI mediante estas acciones:
1. Agregar esta línea a */etc/sysctl.conf* <br>
    `kernel.panic_on_unrecovered_nmi=1`
1. Reiniciar o actualizar sysctl mediante la ejecución de <br>
    `sysctl -p`

Para más información sobre las configuraciones del kernel de Linux, como `unknown_nmi_panic`, `panic_on_io_nmi` y `panic_on_unrecovered_nmi`, consulte: [Documentación de /proc/sys/kernel/*](https://www.kernel.org/doc/Documentation/sysctl/kernel.txt). Para obtener documentación específica de la distribución sobre NMI y pasos para configurar Linux para crear un volcado de memoria cuando se recibe una NMI, consulte los siguientes vínculos:

### <a name="ubuntu"></a>Ubuntu
 - [Kernel Crash Dump](https://help.ubuntu.com/lts/serverguide/kernel-crash-dump.html) (Volcado de memoria del kernel)

### <a name="red-hat"></a>Red Hat
 - [What is an NMI and what can I use it for?](https://access.redhat.com/solutions/4127) (¿Qué es una NMI y para qué puedo usarla?)
 - [How can I configure my system to crash when NMI switch is pushed?](https://access.redhat.com/solutions/125103) (¿Cómo puedo configurar mi sistema para que se bloquee cuando se inserta un modificador de NMI?)
 - [Crash Dump Admin Guide](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/pdf/kernel_crash_dump_guide/kernel-crash-dump-guide.pdf) (Guía del administrador de volcados de memoria)

### <a name="suse"></a>SUSE
- [Configure kernel core dump capture](https://www.suse.com/support/kb/doc/?id=3374462) (Configuración de la captura de volcado de núcleo del kernel)

### <a name="coreos"></a>CoreOS
- [Collecting crash logs](https://coreos.com/os/docs/latest/collecting-crash-logs.html) (Recopilación de registros de bloqueo)

## <a name="next-steps"></a>Pasos siguientes
* La página principal de la documentación de Linux de la consola serie se encuentra [aquí](serial-console-linux.md).
* Use la consola serie para arrancar en [GRUB y entrar en modo de usuario único](serial-console-grub-single-user-mode.md).
* La consola serie también está disponible para VM [Windows](serial-console-windows.md)
* Obtenga más información sobre [Diagnósticos de arranque](boot-diagnostics.md)