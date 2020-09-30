---
title: Solución de problemas de inicio de máquinas virtuales Linux debido a errores de fstab | Microsoft Docs
description: Se explica por qué no se puede iniciar la máquina virtual Linux y cómo resolver el problema.
services: virtual-machines-linux
documentationcenter: ''
author: v-miegge
manager: dcscontentpm
editor: ''
tags: ''
ms.service: virtual-machines-linux
ms.topic: troubleshooting
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-linux
ms.devlang: azurecli
ms.date: 10/09/2019
ms.author: v-six
ms.openlocfilehash: fd49993e6825c47bbae8f034715c03191e06ab2d
ms.sourcegitcommit: bf1340bb706cf31bb002128e272b8322f37d53dd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/03/2020
ms.locfileid: "89441670"
---
# <a name="troubleshoot-linux-vm-starting-issues-due-to-fstab-errors"></a>Solución de problemas de inicio de máquinas virtuales Linux debido a errores de fstab

No se puede conectar a una máquina virtual (VM) Linux de Azure mediante una conexión Secure Shell (SSH). Al ejecutar la característica [Diagnósticos de arranque](./boot-diagnostics.md) en [Azure Portal](https://portal.azure.com/), verá entradas de registro similares a los ejemplos siguientes:

## <a name="examples"></a>Ejemplos

Los siguientes son ejemplos de posibles errores:

### <a name="example-1-a-disk-is-mounted-by-the-scsi-id-instead-of-the-universally-unique-identifier-uuid"></a>Ejemplo 1: Se monta un disco mediante el identificador SCSI en lugar del identificador único universal (UUID)

```
[K[[1;31m TIME [0m] Timed out waiting for device dev-incorrect.device.
[[1;33mDEPEND[0m] Dependency failed for /data.
[[1;33mDEPEND[0m] Dependency failed for Local File Systems.
…
Welcome to emergency mode! After logging in, type "journalctl -xb" to viewsystem logs, "systemctl reboot" to reboot, "systemctl default" to try again to boot into default mode.
Give root password for maintenance
(or type Control-D to continue)
```

### <a name="example-2-an-unattached-device-is-missing-on-centos"></a>Ejemplo 2: Falta un dispositivo no asociado en CentOS

```
Checking file systems…
fsck from util-linux 2.19.1
Checking all file systems.
/dev/sdc1: nonexistent device ("nofail" fstab option may be used to skip this device)
/dev/sdd1: nonexistent device ("nofail" fstab option may be used to skip this device)
/dev/sde1: nonexistent device ("nofail" fstab option may be used to skip this device)

[/sbin/fsck.ext3 (1) — /CODE] sck.ext3 -a /dev/sdc1
fsck.ext3: No such file or directory while trying to open /dev/sdc1

/dev/sdc1:
The superblock could not be read or does not describe a correct ext2
filesystem. If the device is valid and it really contains an ext2
filesystem (and not swap or ufs or something else), then the superblock
is corrupt, and you might try running e2fsck with an alternate superblock:

e2fsck -b 8193 <device>

[/sbin/fsck.xfs (1) — /GLUSTERDISK] fsck.xfs -a /dev/sdd1
/sbin/fsck.xfs: /dev/sdd1 does not exist
[/sbin/fsck.ext3 (1) — /DATATEMP] fsck.ext3 -a /dev/sde1 fsck.ext3: No such file or directory while trying to open /dev/sde1
```

### <a name="example-3-a-vm-cannot-start-because-of-an-fstab-misconfiguration-or-because-the-disk-is-no-longer-attached"></a>Ejemplo 3: No se puede iniciar una máquina virtual debido a un error de configuración de fstab o porque el disco ya no está asociado

```
The disk drive for /var/lib/mysql is not ready yet or not present.
Continue to wait, or Press S to skip mounting or M for manual recovery
```

### <a name="example-4-a-serial-log-entry-shows-an-incorrect-uuid"></a>Ejemplo 4: Una entrada de registro serie muestra un UUID incorrecto

```
Checking filesystems
Checking all file systems.
[/sbin/fsck.ext4 (1) — /] fsck.ext4 -a /dev/sda1
/dev/sda1: clean, 70442/1905008 files, 800094/7608064 blocks
[/sbin/fsck.ext4 (1) — /datadrive] fsck.ext4 -a UUID="<UUID>"
fsck.ext4: Unable to resolve UUID="<UUID>"
[FAILED

*** An error occurred during the file system check.
*** Dropping you to a shell; the system will reboot
*** when you leave the shell.
*** Warning — SELinux is active
*** Disabling security enforcement for system recovery.
*** Run 'setenforce 1' to reenable.
type=1404 audit(1428047455.949:4): enforcing=0 old_enforcing=1 auid=<AUID> ses=4294967295
Give root password for maintenance
(or type Control-D to continue)
```

Este problema puede producirse si la sintaxis de la tabla del sistema de archivos (fstab) es incorrecta o si un disco de datos necesario que está asignado a una entrada en el archivo "/etc/fstab" no está asociado a la máquina virtual.

## <a name="resolution"></a>Solución

Para resolver este problema, inicie la máquina virtual en modo de emergencia mediante la consola serie de Azure Virtual Machines. Luego, use la herramienta para reparar el sistema de archivos. Si la consola serie no está habilitada en la máquina virtual, vaya a la sección [Reparación de la máquina virtual sin conexión](#repair-the-vm-offline).

## <a name="use-the-serial-console"></a>Uso de la consola serie

### <a name="using-single-user-mode"></a>Uso del Modo de usuario único

1. Conexión a la [consola serie](./serial-console-linux.md).
2. Use la consola serie para realizar el [modo de usuario único](serial-console-grub-single-user-mode.md).
3. Una vez que la máquina virtual se haya iniciado en modo de usuario único. Use el editor de texto que prefiera para abrir el archivo fstab. 

   ```
   # nano /etc/fstab
   ```

4. Revise los sistemas de archivos enumerados. Cada línea del archivo fstab indica un sistema de archivos que se monta cuando se inicia la máquina virtual. Para más información sobre la sintaxis del archivo fstab, ejecute el comando man fstab. Para solucionar los errores de inicio, revise cada línea para asegurarse de que es correcta tanto en estructura como en contenido.

   > [!Note]
   > * Los campos de cada línea están separados por tabulaciones o espacios. Se omiten las líneas en blanco. Las líneas que tienen un signo de número (#) como primer carácter son comentarios. Las líneas comentadas pueden permanecer en el archivo fstab, pero no se procesarán. Se recomienda comentar las líneas fstab sobre las que no está seguro en lugar de quitarlas.
   > * Para que la máquina virtual se recupere y se inicie, las particiones del sistema de archivos deben ser las únicas particiones necesarias. La máquina virtual puede experimentar errores de aplicación sobre particiones comentadas adicionales. Sin embargo, la máquina virtual debe iniciarse sin las particiones adicionales. Posteriormente, puede quitar la marca de comentario de las líneas comentadas.
   > * Se recomienda montar los discos de datos en máquinas virtuales de Azure mediante el UUID de la partición del sistema de archivos. Por ejemplo, ejecute el siguiente comando: ``/dev/sdc1: LABEL="cloudimg-rootfs" UUID="<UUID>" TYPE="ext4" PARTUUID="<PartUUID>"``.
   > * Para determinar el UUID del sistema de archivos, ejecute el comando blkid. Para más información sobre la sintaxis, ejecute el comando man blkid.
   > * La opción nofail garantiza que la máquina virtual se inicia incluso si el sistema de archivos está dañado o no existe durante el inicio. Se recomienda usar la opción nofail en el archivo fstab para permitir que el inicio continúe después de que se produzcan errores en las particiones que no son necesarias para que se inicie la máquina virtual.

5. Cambie o comente cualquier línea incorrecta o innecesaria en el archivo fstab para permitir que la máquina virtual se inicie correctamente.

6. Guarde los cambios en el archivo fstab.

7. Reinicie la máquina virtual con el comando siguiente.
   
   ```
   # reboot -f
   ```
> [!Note]
   > También puede usar el comando "Ctrl + X", que reiniciaría igualmente la máquina virtual.


8. Si el comentario o la corrección de entradas se realizó correctamente, el sistema debe tener acceso al símbolo del sistema de bash en el portal. Compruebe si puede conectarse a la máquina virtual.

### <a name="using-root-password"></a>Uso de la contraseña raíz

1. Conexión a la [consola serie](./serial-console-linux.md).
2. Inicie sesión en el sistema con un usuario local y una contraseña.

   > [!Note]
   > No se puede usar una clave SSH para iniciar sesión en el sistema en la consola serie.

3. Busque el error que indica que el disco no se ha montado. En el ejemplo siguiente, el sistema intentaba asociar un disco que ya no existía:

   ```
   [DEPEND] Dependency failed for /datadisk1.
   [DEPEND] Dependency failed for Local File Systems.
   [DEPEND] Dependency failed for Relabel all filesystems, if necessary.
   [DEPEND] Dependency failed for Migrate local... structure to the new structure.
   Welcome to emergency mode! After logging in, type "journalctl -xb" to view
   system logs, "systemctl reboot" to reboot, "systemctl default" or ^D to try again to boot into default mode.
   Give root password for maintenance
   (or type Control-D to continue):
   ```

4. Conéctese a la máquina virtual con la contraseña raíz (máquinas virtuales basadas en Red Hat).

5. Use el editor de texto que prefiera para abrir el archivo fstab. Después de montar el disco, ejecute el siguiente comando para Nano:

   ```
   $ nano /mnt/troubleshootingdisk/etc/fstab
   ```

6. Revise los sistemas de archivos enumerados. Cada línea del archivo fstab indica un sistema de archivos que se monta cuando se inicia la máquina virtual. Para más información sobre la sintaxis del archivo fstab, ejecute el comando man fstab. Para solucionar los errores de inicio, revise cada línea para asegurarse de que es correcta tanto en estructura como en contenido.

   > [!Note]
   > * Los campos de cada línea están separados por tabulaciones o espacios. Se omiten las líneas en blanco. Las líneas que tienen un signo de número (#) como primer carácter son comentarios. Las líneas comentadas pueden permanecer en el archivo fstab, pero no se procesarán. Se recomienda comentar las líneas fstab sobre las que no está seguro en lugar de quitarlas.
   > * Para que la máquina virtual se recupere y se inicie, las particiones del sistema de archivos deben ser las únicas particiones necesarias. La máquina virtual puede experimentar errores de aplicación sobre particiones comentadas adicionales. Sin embargo, la máquina virtual debe iniciarse sin las particiones adicionales. Posteriormente, puede quitar la marca de comentario de las líneas comentadas.
   > * Se recomienda montar los discos de datos en máquinas virtuales de Azure mediante el UUID de la partición del sistema de archivos. Por ejemplo, ejecute el siguiente comando: ``/dev/sdc1: LABEL="cloudimg-rootfs" UUID="<UUID>" TYPE="ext4" PARTUUID="<PartUUID>"``.
   > * Para determinar el UUID del sistema de archivos, ejecute el comando blkid. Para más información sobre la sintaxis, ejecute el comando man blkid.
   > * La opción nofail garantiza que la máquina virtual se inicia incluso si el sistema de archivos está dañado o no existe durante el inicio. Se recomienda usar la opción nofail en el archivo fstab para permitir que el inicio continúe después de que se produzcan errores en las particiones que no son necesarias para que se inicie la máquina virtual.

7. Cambie o comente cualquier línea incorrecta o innecesaria en el archivo fstab para permitir que la máquina virtual se inicie correctamente.

8. Guarde los cambios en el archivo fstab.

9. Reinicie la máquina virtual.

10. Si el comentario o la corrección de entradas se realizó correctamente, el sistema debe tener acceso al símbolo del sistema de bash en el portal. Compruebe si puede conectarse a la máquina virtual.

11. Cuando pruebe cualquier cambio, compruebe los puntos de montaje con el comando mount –a. Si no hay ningún error, los puntos de montaje son correctos.

## <a name="repair-the-vm-offline"></a>Reparación de la máquina virtual sin conexión

1. Asocie el disco del sistema de la máquina virtual como un disco de datos a una máquina virtual de recuperación (cualquier máquina virtual Linux en funcionamiento). Para ello, puede usar [comandos de la CLI](./troubleshoot-recovery-disks-linux.md) o puede automatizar la configuración de la máquina de recuperación mediante los [comandos de reparación de la máquina virtual](repair-linux-vm-using-azure-virtual-machine-repair-commands.md).

2. Después de montar el disco del sistema como un disco de datos en la máquina virtual de recuperación, haga una copia de seguridad del archivo fstab antes de realizar cambios y, luego, siga estos pasos para corregirlo.

3. Busque el error que indica que el disco no se ha montado. En el ejemplo siguiente, el sistema intentaba asociar un disco que ya no existía:

   ```output
   [DEPEND] Dependency failed for /datadisk1.
   [DEPEND] Dependency failed for Local File Systems.
   [DEPEND] Dependency failed for Relabel all filesystems, if necessary.
   [DEPEND] Dependency failed for Migrate local... structure to the new structure.
   Welcome to emergency mode! After logging in, type "journalctl -xb" to view system logs, "systemctl reboot" to reboot, "systemctl default" or ^D to try again to boot into default mode.
   Give root password for maintenance (or type Control-D to continue):
   ```

4. Conéctese a la máquina virtual con la contraseña raíz (máquinas virtuales basadas en Red Hat).

5. Use el editor de texto que prefiera para abrir el archivo fstab. Después de montar el disco, ejecute el siguiente comando para Nano: Asegúrese de que trabaja en el archivo fstab que se encuentra en el disco montado y no en el de la máquina virtual de rescate.

   ```
   $ nano /mnt/troubleshootingdisk/etc/fstab
   ```

6. Revise los sistemas de archivos enumerados. Cada línea del archivo fstab indica un sistema de archivos que se monta cuando se inicia la máquina virtual. Para más información sobre la sintaxis del archivo fstab, ejecute el comando man fstab. Para solucionar los errores de inicio, revise cada línea para asegurarse de que es correcta tanto en estructura como en contenido.

   > [!Note]
   > * Los campos de cada línea están separados por tabulaciones o espacios. Se omiten las líneas en blanco. Las líneas que tienen un signo de número (#) como primer carácter son comentarios. Las líneas comentadas pueden permanecer en el archivo fstab, pero no se procesarán. Se recomienda comentar las líneas fstab sobre las que no está seguro en lugar de quitarlas.
   > * Para que la máquina virtual se recupere y se inicie, las particiones del sistema de archivos deben ser las únicas particiones necesarias. La máquina virtual puede experimentar errores de aplicación sobre particiones comentadas adicionales. Sin embargo, la máquina virtual debe iniciarse sin las particiones adicionales. Posteriormente, puede quitar la marca de comentario de las líneas comentadas.
   > * Se recomienda montar los discos de datos en máquinas virtuales de Azure mediante el UUID de la partición del sistema de archivos. Por ejemplo, ejecute el siguiente comando: ``/dev/sdc1: LABEL="cloudimg-rootfs" UUID="<UUID>" TYPE="ext4" PARTUUID="<PartUUID>"``.
   > * Para determinar el UUID del sistema de archivos, ejecute el comando blkid. Para más información sobre la sintaxis, ejecute el comando man blkid. Observe que el disco que desea recuperar ahora está montado en una nueva máquina virtual. Aunque los UUID deben ser coherentes, los identificadores de partición del dispositivo (por ejemplo, "/dev/sda1") son diferentes en esta máquina virtual. Las particiones del sistema de archivos de la máquina virtual con errores original que se encuentran en un disco duro virtual que no es del sistema no están disponibles para la máquina virtual de recuperación con los [comandos de la CLI](./troubleshoot-recovery-disks-linux.md).
   > * La opción nofail garantiza que la máquina virtual se inicia incluso si el sistema de archivos está dañado o no existe durante el inicio. Se recomienda usar la opción nofail en el archivo fstab para permitir que el inicio continúe después de que se produzcan errores en las particiones que no son necesarias para que se inicie la máquina virtual.

7. Cambie o comente cualquier línea incorrecta o innecesaria en el archivo fstab para permitir que la máquina virtual se inicie correctamente.

8. Guarde los cambios en el archivo fstab.

9. Reinicie la máquina virtual o recompile la máquina virtual original.

10. Si el comentario o la corrección de entradas se realizó correctamente, el sistema debe tener acceso al símbolo del sistema de bash en el portal. Compruebe si puede conectarse a la máquina virtual.

11. Cuando pruebe cualquier cambio, compruebe los puntos de montaje con el comando mount –a. Si no hay ningún error, los puntos de montaje son correctos.

12. Desmonte y separe el disco duro virtual original y, luego, cree una máquina virtual a partir de él. Para ello, puede usar [comandos de la CLI](troubleshoot-recovery-disks-linux.md) o los [comandos de reparación de la máquina virtual](repair-linux-vm-using-azure-virtual-machine-repair-commands.md) si los ha usado para crear la máquina virtual de recuperación.

13. Después de volver a crear la máquina virtual y de que pueda conectarse a ella a través de SSH, realice las siguientes acciones:
    * Revise cualquiera de las líneas de fstab que se cambiaron o comentaron durante la recuperación.
    * Asegúrese de que usa el UUID y la opción nofail correctamente.
    * Pruebe cualquier cambio de fstab antes de reiniciar la máquina virtual. Para ello, ejecute el siguiente comando: ``$ sudo mount -a``.
    * Cree una copia adicional del archivo fstab corregido para su uso en futuros escenarios de recuperación.

## <a name="next-steps"></a>Pasos siguientes

* [Solución de problemas de una máquina virtual Linux mediante la conexión del disco del sistema operativo a una máquina virtual de recuperación con la CLI de Azure 2.0](./troubleshoot-recovery-disks-linux.md)
* [Solución de problemas de una máquina virtual Linux mediante la conexión del disco del sistema operativo a una máquina virtual de recuperación mediante Azure Portal](./troubleshoot-recovery-disks-portal-linux.md)
