---
title: Solución de problemas de instalación de inserción del servicio de movilidad con Azure Site Recovery
description: Solución de problemas de errores de instalación del servicio de movilidad durante la habilitación de la replicación para recuperación ante desastres con Azure Site Recovery.
author: Rajeswari-Mamilla
manager: rochakm
ms.service: site-recovery
ms.topic: conceptual
ms.author: ramamill
ms.date: 04/03/2020
ms.openlocfilehash: 8ee6449f357a578b30809bb03723ac1556e4f459
ms.sourcegitcommit: d39f2cd3e0b917b351046112ef1b8dc240a47a4f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88816202"
---
# <a name="troubleshoot-mobility-service-push-installation"></a>Solución de problemas de instalación de inserción del servicio de movilidad

La instalación del servicio de movilidad es un paso clave para habilitar la replicación. Para realizar este paso correctamente, se deben cumplir los requisitos previos y usar las configuraciones admitidas. Los errores más comunes que pueden aparecer durante la instalación del servicio de movilidad se deben a:

* [Errores de credenciales/privilegios](#credentials-check-errorid-95107--95108)
* [Errores de inicio de sesión](#login-failures-errorid-95519-95520-95521-95522)
* [Errores de conectividad](#connectivity-failure-errorid-95117--97118)
* [Errores con el uso compartido de archivos e impresoras](#file-and-printer-sharing-services-check-errorid-95105--95106)
* [Errores de Instrumental de administración de Windows (WMI)](#windows-management-instrumentation-wmi-configuration-check-error-code-95103)
* [Sistemas operativos no admitidos](#unsupported-operating-systems)
* [Configuraciones de arranque no admitidas](#unsupported-boot-disk-configurations-errorid-95309-95310-95311)
* [Errores de instalación del Servicio de instantáneas de volumen (VSS)](#vss-installation-failures)
* [Uso del nombre del dispositivo en la configuración de GRUB en lugar del UUID del dispositivo](#enable-protection-failed-as-device-name-mentioned-in-the-grub-configuration-instead-of-uuid-errorid-95320)
* [Volumen del Administrador de volúmenes lógicos (LVM)](#lvm-support-from-920-version)
* [Advertencias de reinicio](#install-mobility-service-completed-with-warning-to-reboot-errorid-95265--95266)

Al habilitar la replicación, Azure Site Recovery intenta insertar el agente del servicio de movilidad en su máquina virtual (VM). Como parte de este proceso, el servidor de configuración intenta conectarse con la máquina virtual y copiar el agente. Para permitir la correcta instalación, siga las instrucciones de solución de problemas detalladas.

## <a name="credentials-check-errorid-95107--95108"></a>Comprobación de credenciales (ErrorID: 95107 & 95108)

Verifique si la cuenta de usuario elegida durante la habilitación de la replicación es válida y precisa. Azure Site Recovery necesita una cuenta **raíz** o una cuenta de usuario con **privilegios de administrador** para realizar la instalación de inserción. De lo contrario, dicha instalación se bloqueará en la máquina de origen.

Para Windows (**error 95107**), verifique si la cuenta de usuario tiene acceso administrativo en el equipo de origen, con una cuenta local o cuenta de dominio. Si no utiliza una cuenta de dominio, deberá deshabilitar el control de acceso del usuario remoto en el equipo local.

* Para agregar manualmente una clave del Registro que deshabilite el control de acceso de usuarios remotos:

  * `HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System`
  * Agregue un nuevo `DWORD`: `LocalAccountTokenFilterPolicy`
  * Establezca el valor en `1`

* Para agregar la clave del Registro, desde el símbolo del sistema, ejecute el siguiente comando:

  `REG ADD HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System /v LocalAccountTokenFilterPolicy /t REG_DWORD /d 1`

Para Linux (**error 95108**), debe elegir la cuenta **raíz** para la correcta instalación del agente del servicio de movilidad. Además, los servicios del protocolo de transferencia de archivos SSH (SFTP) deben estar en ejecución. Para habilitar la autenticación del subsistema SFTP y la contraseña en el archivo _sshd_config_:

1. Inicie sesión como usuario **raíz**.
1. Vaya al _archivo /etc/ssh/sshd_config_ y busque la línea que comienza por `PasswordAuthentication`.
1. Quite la marca de comentario de la línea y cambie el valor a `yes`.
1. Busque la línea que comienza por `Subsystem` y quite la marca de comentario.
1. Reinicie el servicio `sshd` .

Si desea modificar las credenciales de la cuenta de usuario elegida, siga [estas instrucciones](vmware-azure-manage-configuration-server.md#modify-credentials-for-mobility-service-installation).

## <a name="insufficient-privileges-failure-errorid-95517"></a>Error de privilegios insuficientes (ErrorID: 95517)

Cuando el usuario que opta por instalar el agente de movilidad no tiene privilegios de administrador, el servidor de configuración o el servidor del proceso de escalabilidad horizontal no podrán copiar el software de agente de movilidad en la máquina de origen. Este error es el resultado de un error de acceso denegado. Asegúrese de que la cuenta de usuario tiene privilegios de administrador.

Si desea modificar las credenciales de la cuenta de usuario elegida, siga [estas instrucciones](vmware-azure-manage-configuration-server.md#modify-credentials-for-mobility-service-installation).

## <a name="insufficient-privileges-failure-errorid-95518"></a>Error de privilegios insuficientes (ErrorID: 95518)

Cuando se produce un error al establecer una relación de confianza de dominio entre el dominio principal y la estación de trabajo al intentar iniciar sesión en la máquina de origen, se produce un error en la instalación del agente de movilidad con el id. de error 95518. Asegúrese de que la cuenta de usuario usada para instalar al agente de movilidad tiene privilegios administrativos para iniciar sesión a través del dominio principal de la máquina de origen.

Si desea modificar las credenciales de la cuenta de usuario elegida, siga [estas instrucciones](vmware-azure-manage-configuration-server.md#modify-credentials-for-mobility-service-installation).

## <a name="login-failures-errorid-95519-95520-95521-95522"></a>Errores de inicio de sesión (ErrorID: 95519, 95520, 95521 y 95522)

En esta sección se describen los mensajes de error de credenciales y de inicio de sesión.

### <a name="credentials-of-the-user-account-have-been-disabled-errorid-95519"></a>Las credenciales de la cuenta de usuario se han deshabilitado (ErrorID: 95519)

La cuenta de usuario elegida durante la habilitación de la replicación se ha deshabilitado. Para habilitar la cuenta de usuario, consulte este [artículo](/powershell/module/Microsoft.PowerShell.LocalAccounts/Enable-LocalUser) o ejecute el siguiente comando reemplazando el texto _username_ con el nombre de usuario real.

`net user 'username' /active:yes`

### <a name="credentials-locked-out-because-of-multiple-failed-login-attempts-errorid-95520"></a>Se han bloqueado las credenciales debido de varios intentos de inicio de sesión incorrectos (ErrorID: 95520)

Varios esfuerzos de reintento erróneos para obtener acceso a una máquina bloquearán la cuenta de usuario. El error puede deberse a alguno de los siguientes motivos:

* Las credenciales proporcionadas durante la configuración son incorrectas.
* La cuenta de usuario elegida durante la habilitación de la replicación es incorrecta.

Modifique las credenciales elegidas siguiendo [estas instrucciones](vmware-azure-manage-configuration-server.md#modify-credentials-for-mobility-service-installation) y reintente la operación.

### <a name="logon-servers-arent-available-on-the-source-machine-errorid-95521"></a>Los servidores de inicio de sesión no están disponibles en la máquina de origen (ErrorID: 95521)

Este error se produce cuando los servidores de inicio de sesión no están disponibles en la máquina de origen. Si los servidores de inicio de sesión no están disponibles, se producirán errores en las solicitudes de inicio de sesión y no se podrá instalar el agente de movilidad. Para iniciar sesión correctamente, asegúrese de que los servidores de inicio de sesión estén disponibles en la máquina de origen e inicie el servicio Netlogon. Para obtener más información, consulte [Escenarios de inicio de sesión de Windows](/windows-server/security/windows-authentication/windows-logon-scenarios).

### <a name="logon-service-isnt-running-on-the-source-machine-errorid-95522"></a>Los servidores de inicio de sesión no se están ejecutando en la máquina de origen (ErrorID: 95522)

El servicio de inicio de sesión no se ejecuta en la máquina de origen y produce errores de solicitud de inicio de sesión. No se puede instalar el agente de movilidad. Para resolver el error, use uno de los métodos siguientes para iniciar el servicio `Netlogon` en la máquina de origen:

* Para iniciar el servicio `Netlogon` desde un símbolo del sistema, ejecute el comando `net start Netlogon`.
* En el Administrador de tareas, inicie el servicio `Netlogon`.

## <a name="connectivity-failure-errorid-95117--97118"></a>Error de conectividad (ErrorID: 95117 y 97118)

El servidor de configuración o el servidor de procesos de escalabilidad horizontal intentan conectarse a la VM de origen para instalar el agente de movilidad. Este error se produce cuando no se puede tener acceso a la máquina de origen debido a problemas de conectividad de red.

Para resolver el error:

* Asegúrese de que puede hacer ping a la máquina de origen desde el servidor de configuración. Si ha elegido el servidor de procesos de escalabilidad horizontal durante la habilitación de la replicación, asegúrese de que puede hacer ping a la máquina de origen desde el servidor de procesos.

* Desde la línea de comandos de la máquina del servidor de origen, use `Telnet` para hacer ping al servidor de configuración o al servidor de procesos de escalabilidad horizontal en el puerto HTTPS 135 como se muestra en el siguiente comando. Este comando comprueba si hay problemas de conectividad de red o problemas de bloqueo de puertos del firewall.

  `telnet <CS/ scale-out PS IP address> <135>`

* Además, para una VM Linux:
  * Compruebe si los paquetes OpenSSH, OpenSSH Server y OpenSSL más recientes estén instalados.
  * Compruebe y asegúrese de que Secure Shell (SSH) está habilitado y se ejecuta en el puerto 22.
  * Los servicios SFTP deben estar en ejecución. Para habilitar la autenticación del subsistema SFTP y la contraseña en el archivo _sshd_config_:

    1. Inicie sesión como usuario **raíz**.
    1. Vaya al _archivo /etc/ssh/sshd_config_ y busque la línea que comienza por `PasswordAuthentication`.
    1. Quite la marca de comentario de la línea y cambie el valor a `yes`.
    1. Busque la línea que comienza por `Subsystem` y quite la marca de comentario.
    1. Reinicie el servicio `sshd` .

* Es posible que se haya producido un error en el intento de conexión si no hay respuestas correctas tras un período de tiempo o si se produjo un error en la conexión establecida porque el host conectado no ha podido responder.
* Puede ser un problema relacionado con la conectividad, la red o el dominio. También podría deberse a un problema de resolución del nombre de DNS o un problema de agotamiento del puerto TCP. Compruebe si hay algún problema conocido en el dominio.

## <a name="connectivity-failure-errorid-95523"></a>Error de conectividad (ErrorID: 95523)

Este error se produce cuando la red en la que reside la máquina de origen no se encuentra o puede que se haya eliminado o que ya no está disponible. La única manera de resolver el error es asegurarse de que la red existe.

## <a name="check-access-for-network-shared-folders-on-source-machine-errorid-9510595523"></a>Compruebe el acceso a las carpetas compartidas de red en la máquina de origen (ErrorID: 95105,95523)

Compruebe si las carpetas compartidas de red de la máquina de origen son accesibles desde el servidor de procesos (PS) de forma remota mediante las credenciales especificadas. Para confirmar el acceso: 

1. Inicie sesión en la máquina del servidor de procesos.
2. Abra el Explorador de archivos. En la barra de direcciones, escriba `\\<SOURCE-MACHINE-IP>\C$` y haga clic en Entrar.

    ![Apertura de la carpeta en PS](./media/vmware-azure-troubleshoot-push-install/open-folder-process-server.PNG)

3. El Explorador de archivos pedirá las credenciales. Escriba el nombre de usuario y la contraseña y haga clic en Aceptar. <br><br/>

    ![Provisión de las credenciales](./media/vmware-azure-troubleshoot-push-install/provide-credentials.PNG)

    >[!NOTE]
    > Si la máquina de origen está unida a un dominio, proporcione el nombre de dominio junto con el nombre de usuario en el formato `<domainName>\<username>`. Si la máquina de origen está en un grupo de trabajo, proporcione solo el nombre de usuario.

4. Si la conexión se establece correctamente, las carpetas de la máquina de origen podrán verse de manera remota desde el servidor de procesos.

    ![Carpetas visibles desde la máquina de origen](./media/vmware-azure-troubleshoot-push-install/visible-folders-from-source.png)

Si la conexión no se establece correctamente, compruebe si se cumplen todos los requisitos previos.

## <a name="file-and-printer-sharing-services-check-errorid-95105--95106"></a>Comprobación de servicios de uso compartido de archivos e impresoras (ErrorID: 95105 y 95106)

Después de la comprobación de conectividad, verifique si está habilitado el servicio de uso compartido de archivos e impresoras en la máquina virtual. Esta configuración es necesaria para copiar el agente de movilidad en la máquina de origen.

Para **Windows 2008 R2 y versiones anteriores**:

* Para habilitar el uso compartido de archivos e impresoras a través del Firewall de Windows,
  1. Abre **Panel de control** > **Sistema y seguridad** > **Firewall de Windows**. En el panel izquierdo, seleccione **Configuración avanzada** > **Reglas de entrada** en el árbol de la consola.
  1. Busque las reglas de uso compartido de archivos e impresoras (sesión NB de entrada) y uso compartido de archivos e impresoras (SMB de entrada).
  1. Para cada regla, haga clic con el botón derecho en la regla y, a continuación, haga clic en **Habilitar regla**.

* Para habilitar el uso compartido de archivos con directiva de grupo:
  1. Vaya a **Inicio**, escriba `gpmc.msc` y busque.
  1. En el panel de navegación, abra las siguientes carpetas: **Directiva de equipo local** > **Configuración de usuario** > **Plantillas administrativas** > **Componentes de Windows** > **Uso compartido de red**.
  1. En el panel de detalles, haga doble clic en **Prevent users from sharing files within their profile** (Evitar que los usuarios compartan archivos con su perfil).

     Para deshabilitar la configuración de directiva de grupo y habilitar la capacidad del usuario para compartir archivos, seleccione **Deshabilitado**.

  1. Seleccione **Aceptar** para guardar los cambios.

  Para más información, consulte [Enable or disable File sharing with Group Policy](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/cc754359(v=ws.10)) (Habilitar o deshabilitar el uso compartido de archivos con directiva de grupo).

Para habilitar el uso compartido de archivos e impresoras para versiones posteriores de Windows o Linux, siga las instrucciones de [Instalación de Mobility Service para la recuperación ante desastres de máquinas virtuales de VMware y servidores físicos](vmware-azure-install-mobility-service.md).

## <a name="windows-management-instrumentation-wmi-configuration-check-error-code-95103"></a>Comprobación de la configuración de Instrumental de administración de Windows (WMI) (Código de error: 95103)

Después de la comprobación de los servicios de archivos e impresoras, habilite el servicio WMI para perfiles privados, públicos y de dominio a través del firewall. Esta configuración es necesaria para completar la ejecución remota en la máquina de origen.

Para habilitar WMI:

1. Vaya al **Panel de Control** > **Seguridad** y seleccione **Firewall de Windows**.
1. Seleccione **Cambiar configuración** y, a continuación, seleccione la pestaña **Excepciones**.
1. En la ventana **Excepciones**, seleccione la casilla para Instrumental de administración de Windows (WMI) para permitir el tráfico de WMI a través del firewall.

También puede habilitar el tráfico de WMI a través del firewall en el símbolo del sistema con el comando siguiente:

`netsh advfirewall firewall set rule group="windows management instrumentation (wmi)" new enable=yes`

Se pueden encontrar otros artículos de solución de problemas de WMI en los siguientes artículos:

* [Basic WMI Testing](https://techcommunity.microsoft.com/t5/ask-the-performance-team/bg-p/AskPerf) (Pruebas básicas de WMI)
* [WMI Troubleshooting](/windows/win32/wmisdk/wmi-troubleshooting) (Solución de problemas de WMI)
* [Troubleshooting problems with WMI scripts and WMI Services](/previous-versions/tn-archive/ff406382(v=msdn.10)) (Solución de problemas con los scripts y servicios de WMI)

## <a name="unsupported-operating-systems"></a>Sistemas operativos no admitidos

Otra causa común de error puede deberse a un sistema operativo no admitido. Utilice un sistema operativo y una versión de kernel admitidos para conseguir una instalación correcta del servicio de movilidad. Evite el uso de revisiones privadas.

Para ver la lista de sistemas operativos y versiones de kernel compatibles con Azure Site Recovery, consulte el [documento de la matriz de soporte técnico](vmware-physical-azure-support-matrix.md#replicated-machines).

## <a name="unsupported-boot-disk-configurations-errorid-95309-95310-95311"></a>Las configuraciones del disco de arranque no son compatibles (ErrorID: 95309, 95310 y 95311)

### <a name="boot-and-system-partitions--volumes-arent-the-same-disk-errorid-95309"></a>Los volúmenes o las particiones de arranque y del sistema no están en el mismo disco (ErrorID: 95309)

Antes de la versión 9.20, la ubicación de los volúmenes o las particiones de arranque y del sistema en discos diferentes era una configuración no admitida. A partir de la [versión 9.20](https://support.microsoft.com/help/4478871/update-rollup-31-for-azure-site-recovery), se admite esta configuración.

### <a name="the-boot-disk-isnt-available-errorid-95310"></a>El disco de arranque no está disponible (ErrorID: 95310)

Una máquina virtual sin disco de arranque no se puede proteger. Un disco de arranque garantiza la correcta recuperación de una máquina virtual durante una operación de conmutación por error. La ausencia de un disco de arranque genera un error al arrancar la máquina después de la conmutación por error. Asegúrese de que la máquina virtual contiene un disco de arranque y reintente la operación. Asimismo, no se admiten varios discos de arranque en la misma máquina.

### <a name="multiple-boot-disks-present-on-the-source-machine-errorid-95311"></a>Hay varias discos de arranque en la máquina de origen (ErrorID: 95311)

Una máquina virtual con varios discos de arranque no es una [configuración admitida](vmware-physical-azure-support-matrix.md#linux-file-systemsguest-storage).

## <a name="system-partition-on-multiple-disks-errorid-95313"></a>Partición del sistema en varios discos (ErrorID: 95313)

Antes de la versión 9.20, el volumen o partición raíz configurado en varios discos era una configuración no admitida. A partir de la [versión 9.20](https://support.microsoft.com/help/4478871/update-rollup-31-for-azure-site-recovery), se admite esta configuración.

## <a name="enable-protection-failed-as-device-name-mentioned-in-the-grub-configuration-instead-of-uuid-errorid-95320"></a>No se pudo habilitar la protección debido a que se ha mencionado el nombre del dispositivo en la configuración de GRUB en lugar del UUID (ErrorID: 95320)

### <a name="possible-cause"></a>Causa posible

Los archivos de configuración de Grand Unified Bootloader (GRUB) ( _/boot/grub/menu.lst_, _/boot/grub/grub.cfg_, _/boot/grub2/grub.cfg_ o _/etc/default/grub_) pueden contener el valor de los parámetros **root** y **resume** como los nombres reales de los dispositivos, en lugar de un identificador único universal (UUID). Site Recovery obliga al enfoque de UUID, ya que los nombres de los dispositivos pueden cambiar en el reinicio de la VM. Por ejemplo, puede que la VM no se conecte con el mismo nombre en la conmutación por error y que esto genere problemas.

Por ejemplo:

- La línea siguiente procede del archivo de GRUB _/boot/grub2/grub.cfg_:

  `linux /boot/vmlinuz-3.12.49-11-default root=/dev/sda2  ${extra_cmdline} resume=/dev/sda1 splash=silent quiet showopts`

- La línea siguiente procede del archivo de GRUB _/boot/grub/menu.lst_:

  `kernel /boot/vmlinuz-3.0.101-63-default root=/dev/sda2 resume=/dev/sda1 splash=silent crashkernel=256M-:128M showopts vga=0x314`

> [!NOTE]
> Las líneas de GRUB contienen nombres de dispositivo reales para los parámetros **root** y **resume** en lugar del UUID.

### <a name="how-to-fix"></a>Solución

Los nombres de dispositivo se deben reemplazar con el UUID correspondiente.

1. Para buscar el UUID del dispositivo, ejecute el comando `blkid \<device name>`.

   Por ejemplo:

   ```shell
   blkid /dev/sda1
   /dev/sda1: UUID="6f614b44-433b-431b-9ca1-4dd2f6f74f6b" TYPE="swap"
   blkid /dev/sda2
   /dev/sda2: UUID="62927e85-f7ba-40bc-9993-cc1feeb191e4" TYPE="ext3"
   ```

1. Ahora, sustituya el nombre del dispositivo por su UUID utilizando el formato `root=UUID=\<UUID>`. Por ejemplo, si reemplazamos los nombres de dispositivo con UUID por los parámetros root y resume mencionados anteriormente en los archivos _/boot/grub2/grub.cfg_, _/boot/grub2/grub.cfg_ o _/etc/default/grub_, las líneas de los archivos serán parecidas a las siguientes:

   `kernel /boot/vmlinuz-3.0.101-63-default root=UUID=62927e85-f7ba-40bc-9993-cc1feeb191e4 resume=UUID=6f614b44-433b-431b-9ca1-4dd2f6f74f6b splash=silent crashkernel=256M-:128M showopts vga=0x314`

1. Reinicie la protección.

## <a name="install-mobility-service-completed-with-warning-to-reboot-errorid-95265--95266"></a>La instalación del servicio de movilidad se ha completado con una advertencia de reinicio (ErrorID: 95265 y 95266)

El servicio de movilidad de Site Recovery tiene muchos componentes, uno de los cuales es el controlador del filtro. El controlador del filtro se carga en la memoria del sistema solo durante un reinicio del sistema. Las correcciones del controlador del filtro solo pueden aplicarse cuando se carga un nuevo controlador del filtro cuando se reinicia el sistema.

> [!IMPORTANT]
> Se trata de una advertencia, la replicación existente funcionará incluso después de que se realice la nueva actualización del agente. Puede elegir reiniciar el sistema en cualquier momento para aprovechar las ventajas del nuevo controlador del filtro, pero, si no reinicia, el antiguo controlador seguirá funcionando. De este modo, si actualiza sin reiniciar, salvo el controlador del filtro, **se aplicarán todas las demás mejoras y correcciones del servicio de movilidad**. Aunque se recomiende, no es obligatorio reiniciar el sistema después de cada actualización. Para obtener información sobre cuándo es obligatorio un reinicio, consulte la sección [Reinicio después de la actualización de Mobility Service](service-updates-how-to.md#reboot-after-mobility-service-upgrade) en Actualizaciones del servicio en Azure Site Recovery.

> [!TIP]
>Para ver los procedimientos recomendados sobre la programación de actualizaciones durante la ventana de mantenimiento, consulte [Soporte técnico para sistemas operativos/kernels más recientes](service-updates-how-to.md#support-for-latest-operating-systemskernels) en Actualizaciones del servicio en Azure Site Recovery.

## <a name="lvm-support-from-920-version"></a>Compatibilidad con LVM desde la versión 9.20

Antes de la versión 9.20, el Administrador de volúmenes lógicos (LVM) solo se admitía para los discos de datos. La partición `/boot` debe estar en una partición de disco y no en un volumen LVM.

A partir de la [versión 9.20](https://support.microsoft.com/help/4478871/update-rollup-31-for-azure-site-recovery), se admite el [disco del sistema operativo en LVM](vmware-physical-azure-support-matrix.md#linux-file-systemsguest-storage).

## <a name="insufficient-space-errorid-95524"></a>Espacio insuficiente (ErrorID: 95524)

Cuando el agente de movilidad se copia en la máquina de origen, se necesita un mínimo de 100 MB de espacio disponible. Asegúrese de que la máquina de origen tenga la cantidad necesaria de espacio disponible y reintente la operación.

## <a name="low-system-resources"></a>Recursos del sistema bajos

Los posibles identificadores de error que se ven para este problema son 95572 y 95573. Este problema se produce cuando el sistema tiene poca memoria disponible y no puede asignar memoria para la instalación del servicio Mobility. Asegúrese de que se ha liberado suficiente memoria para que la instalación continúe y se complete correctamente.

## <a name="vss-installation-failures"></a>Errores de instalación de VSS

La instalación del Servicio de instantáneas de volumen (VSS) es una parte de la instalación del agente de movilidad. Este servicio se usa en el proceso de generar puntos de recuperación coherentes con la aplicación. Pueden producirse errores durante la instalación de VSS debido a varios motivos. Para identificar los errores exactos, consulte _C:\ProgramData\ASRSetupLogs\ASRUnifiedAgentInstaller.log_. En la siguiente sección se destacan algunos de los errores comunes y los pasos para solucionarlos.

### <a name="vss-error--2147023170-0x800706be---exit-code-511"></a>Error de VSS -2147023170 [0x800706BE]: código de salida 511

Este problema se presenta más a menudo cuando software antivirus bloquea las operaciones de servicios de Azure Site Recovery.

Para solucionar este problema:

1. Revise la lista de [exclusiones de carpetas de un programa antivirus](vmware-azure-set-up-source.md#azure-site-recovery-folder-exclusions-from-antivirus-program).
1. Siga las instrucciones publicadas por su proveedor de software antivirus para desbloquear el registro de DLL en Windows.

### <a name="vss-error-7-0x7---exit-code-511"></a>Error de VSS 7 [0 x 7]: código de salida 511

Este es un error de tiempo de ejecución que se debe a que no hay memoria suficiente para instalar VSS. Aumente el espacio en disco para la correcta finalización de esta operación.

### <a name="vss-error--2147023824-0x80070430---exit-code-517"></a>Error de VSS -2147023824 [0x80070430]: código de salida 517

Este error se produce cuando el servicio de proveedor de VSS para Azure Site Recovery está [marcado para su eliminación](/previous-versions/ms838153(v=msdn.10)). Pruebe a instalar VSS manualmente en la máquina de origen mediante la ejecución del siguiente comando:

`"C:\Program Files (x86)\Microsoft Azure Site Recovery\agent\InMageVSSProvider_Install.cmd"`

### <a name="vss-error--2147023841-0x8007041f---exit-code-512"></a>Error de VSS -2147023841 [0x8007041F]: código de salida 512

Este error se produce cuando la base de datos de servicio de proveedor de VSS para Azure Site Recovery está [bloqueada](/previous-versions/ms833798(v=msdn.10)). Pruebe a instalar VSS manualmente en la máquina de origen mediante la ejecución del comando desde un símbolo del sistema:

`"C:\Program Files (x86)\Microsoft Azure Site Recovery\agent\InMageVSSProvider_Install.cmd"`

Cuando tenga un error, compruebe si algún programa antivirus u otros servicios están bloqueados en el estado **Iniciando**. Un proceso en un estado **Iniciando** puede conservar el bloqueo en los servicios de base de datos. Además, dará lugar a errores en la instalación del proveedor de VSS. Asegúrese de que ningún servicio se encuentra en un estado **Iniciando** y luego vuelva a intentar la operación anterior.

### <a name="vss-exit-code-806"></a>Código de salida VSS 806

Este error se produce cuando la cuenta de usuario usada para la instalación no tiene permisos para ejecutar el comando `CSScript`. Proporcione los permisos necesarios para que la cuenta de usuario ejecute el script y vuelva a intentar la operación.

### <a name="other-vss-errors"></a>Otros errores de VSS

Pruebe a instalar el servicio de proveedor de VSS manualmente en la máquina de origen mediante la ejecución del siguiente comando desde un símbolo del sistema:

`"C:\Program Files (x86)\Microsoft Azure Site Recovery\agent\InMageVSSProvider_Install.cmd"`

## <a name="vss-error---0x8004e00f"></a>Error de VSS 0x8004E00F

Este error normalmente se encuentra durante la instalación del agente de movilidad, debido a problemas en `DCOM`, y `DCOM` se encuentra en estado crítico.

Siga el procedimiento a continuación para determinar la causa del error.

### <a name="examine-the-installation-logs"></a>Examen de los registros de instalación

1. Abra el registro de instalación ubicado en _C:\ProgramData\ASRSetupLogs\ASRUnifiedAgentInstaller.log_.
2. La presencia del siguiente error indica este problema:

    ```Output
    Unregistering the existing application...
    Create the catalogue object
    Get the collection of Applications

    ERROR:

    - Error code: -2147164145 [0x8004E00F]
    - Exit code: 802
    ```

Para resolver el problema:

Póngase en contacto con el [equipo de la plataforma de Microsoft Windows](https://aka.ms/Windows_Support) para obtener ayuda para resolver el problema de DCOM.

Cuando se resuelva el problema de DCOM, vuelva a instalar el proveedor de VSS de Azure Site Recovery manualmente mediante el comando siguiente desde un símbolo del sistema:

`"C:\Program Files (x86)\Microsoft Azure Site Recovery\agent\InMageVSSProvider_Install.cmd"`

Si la coherencia de la aplicación no es fundamental para sus requisitos de recuperación ante desastres, puede omitir la instalación del proveedor de VSS.

Para omitir la instalación del proveedor de VSS de Azure Site Recovery e instalar manualmente el proveedor de VSS de Azure Site Recovery después de la instalación:

1. Instale Mobility Service. Se producirá un error en la instalación en el paso: **Configuración posterior a la instalación**.
1. Para omitir la instalación de VSS:
   1. Abra el directorio de instalación de Mobility Service de Azure Site Recovery ubicado en:

      _C:\Archivos de programa (x86)\Microsoft Azure Site Recovery\agent_

   1. Modifique los scripts de instalación del proveedor de VSS de Azure Site Recovery _InMageVSSProvider_Install_ e _InMageVSSProvider_Uninstall.cmd_ para que siempre sean correctos mediante la adición de las líneas siguientes:

      ```plaintext
      rem @echo off
      setlocal
      exit /B 0
      ```

1. Ejecute manualmente la instalación del agente de movilidad.
1. Cuando la instalación se complete correctamente y se mueva al siguiente paso, **Configurar**, quite las líneas que agregó.
1. Para instalar el proveedor de VSS, abra un símbolo del sistema como administrador y ejecute el siguiente comando:

   `"C:\Program Files (x86)\Microsoft Azure Site Recovery\agent\InMageVSSProvider_Install.cmd"`

1. Verifique que el proveedor de VSS de Azure Site Recovery está instalado como un servicio en los servicios de Windows. Abra el servicio de componentes MMC para confirmar que aparece el proveedor de VSS.
1. Si se sigue produciendo un error en la instalación del proveedor de VSS, consulte con el soporte técnico para resolver los errores de permisos en la interfaz de programación de aplicaciones criptográficas (CAPI2).

## <a name="vss-provider-installation-fails-because-the-cluster-service-being-enabled-on-non-cluster-machine"></a>No se puede instalar el proveedor de VSS debido a que el servicio de clúster está habilitando en una máquina sin clústeres

Este problema provoca que se produzca un error en la instalación del agente de movilidad de Azure Site Recovery durante la instalación del proveedor de VSS de Azure Site Recovery. El error se debe a que hay un problema con `COM+` que impide la instalación del proveedor de VSS.

### <a name="to-identify-the-issue"></a>Para identificar el problema

En el registro que se encuentra en el servidor de configuración en _C:\ProgramData\ASRSetupLogs\UploadedLogs\<date-time>UA_InstallLogFile.log_, se encuentra la siguiente excepción:

```plaintext
COM+ was unable to talk to the Microsoft Distributed Transaction Coordinator (Exception from HRESULT: 0x8004E00F)
```

Para resolver el problema:

1. Verifique que esta máquina sea una máquina sin clústeres y que los componentes del clúster no estén en uso.
1. Si los componentes no están en uso, quite los componentes del clúster de la máquina.

## <a name="drivers-are-missing-on-the-source-server"></a>Faltan controladores en el servidor de origen

Si no puede instalar el agente de movilidad, examine los registros de _C:\ProgramData\ASRSetupLogs_ para determinar si faltan algunos de los controladores necesarios en algunos conjuntos de control.

Para resolver el problema:

1. Con un editor del Registro, como `regedit.msc`, abra el Registro.
1. Abre el nodo `HKEY_LOCAL_MACHINE\SYSTEM`.
1. En el nodo `SYSTEM`, busque los conjuntos de control.
1. Abra cada conjunto de control y verifique que los siguientes controladores de Windows estén presentes:

   * Atapi
   * Vmbus
   * Storflt
   * Storvsc
   * Intelide

1. Vuelva a instalar los controladores que falten.

## <a name="next-steps"></a>Pasos siguientes

Obtenga [más información](vmware-azure-tutorial.md) sobre cómo configurar la recuperación ante desastres para VM de VMware.
