---
title: Solución de problemas de inicio de máquinas virtuales Linux debido a errores del sistema de archivos | Microsoft Docs
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
ms.openlocfilehash: bae0eb91460bf7ab72abbaa7fa56b2515367d37d
ms.sourcegitcommit: bf1340bb706cf31bb002128e272b8322f37d53dd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/03/2020
ms.locfileid: "89436194"
---
# <a name="troubleshoot-linux-vm-starting-issues-due-to-file-system-errors"></a>Solución de problemas de inicio de máquinas virtuales Linux debido a errores del sistema de archivos

No se puede conectar con una máquina virtual Linux de Azure mediante Secure Shell (SSH). Al ejecutar la característica de diagnóstico de arranque en [Azure Portal](https://portal.azure.com/), verá entradas de registro similares a los ejemplos siguientes.

## <a name="examples"></a>Ejemplos

Los siguientes son ejemplos de posibles errores:

### <a name="example-1"></a>Ejemplo 1 

```
Checking all file systems.
[/sbin/fsck.ext4 (1) — /] fsck.ext4 -a /dev/sda1
/dev/sda1 contains a file system with errors, check forced.
/dev/sda1: Inodes that were part of a corrupted orphan linked list found.
/dev/sda1: UNEXPECTED INCONSISTENCY; RUN fsck MANUALLY
```

### <a name="example-2"></a>Ejemplo 2

```
EXT4-fs (sda1): INFO: recovery required on readonly filesystem
EXT4-fs (sda1): write access will be enabled during recovery
EXT4-fs warning (device sda1): ext4_clear_journal_err:4531: Filesystem error recorded from previous mount: IO failure
EXT4-fs warning (device sda1): ext4_clear_journal_err:4532: Making fs in need of filesystem check.
```

### <a name="example-3"></a>Ejemplo 3

```
[  14.252404] EXT4-fs (sda1): Couldn’t remount RDWR because of unprocessed orphan inode list.  Please unmount/remount instead
An error occurred while mounting /.
```

### <a name="example-4"></a>Ejemplo 4 

En este ejemplo el error se debe a una operación fsck de limpieza. En este caso, también hay discos de datos adicionales asociados a la máquina virtual (/dev/sdc1 y/dev/sde1).

```
Checking all file systems. 
[/sbin/fsck.ext4 (1) — /] fsck.ext4 -a /dev/sda1
/dev/sda1: clean, 65405/1905008 files, 732749/7608064 blocks
[/sbin/fsck.ext4 (1) — /tmp] fsck.ext4 -a /dev/sdc1
[/sbin/fsck.ext4 (2) — /backup] fsck.ext4 -a /dev/sde1
/dev/sdc1: clean, 12/1048576 files, 109842/4192957 blocks
/dev/sde1 : clean, 51/67043328 files, 4259482/268173037 blocks
```

Este problema puede producirse si el sistema de archivos no se apagó correctamente o debido a problemas relacionados con el almacenamiento. Entre los problemas se incluyen errores de hardware o software, problemas con controladores o programas, errores de escritura, etc. Es importante tener siempre una copia de seguridad de los datos críticos. Las herramientas que se describen en este artículo pueden ayudar a recuperar sistemas de archivos, pero es posible que se produzca una pérdida de datos.

Linux cuenta con varios comprobadores de sistemas de archivos. Las más comunes para las distribuciones de Azure son: [FSCK](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/6/html/storage_administration_guide/fsck-fs-specific), [E2FSCK](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/storage_administration_guide/fsck-fs-specific) y [Xfs_repair](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/storage_administration_guide/xfsrepair).

## <a name="resolution"></a>Solución

Para resolver este problema, arranque la máquina virtual en modo de emergencia mediante la [consola serie](./serial-console-linux.md) y use esa herramienta para reparar el sistema de archivos. Si la consola serie no está habilitada en la máquina virtual o no funciona, consulte la sección [Reparación de la máquina virtual sin conexión](#repair-the-vm-offline) de este artículo.

## <a name="use-the-serial-console"></a>Uso de la consola serie

1. Conéctese a la consola serie.

   > [!Note]
   > Para más información sobre el uso de la consola serie para Linux, consulte:
   > * [Uso de la consola serie para acceder a GRUB y al modo de usuario único](serial-console-grub-single-user-mode.md)
   > * [Uso de la consola serie para llamadas SysRq y NMI](./serial-console-nmi-sysrq.md)

2. Seleccione el botón del icono de encendido y, luego, seleccione Reiniciar máquina virtual. (Si la consola serie no está habilitada o no está conectada correctamente, no verá el botón).

   ![IMAGEN](./media/linux-recovery-cannot-ssh-to-linux-vm-due-to-file-system-errors-fsck/restart-vm.png)

3. Arranque la máquina virtual en modo de emergencia.

4. Escriba la contraseña de la cuenta raíz para iniciar sesión en modo de emergencia.

5. Use xfs_repair con la opción -n para detectar los errores en el sistema de archivos. En el ejemplo siguiente, se supone que la partición del sistema es/dev/sda1. Reemplácela por el valor adecuado en su máquina virtual.

   ```
   xfs_repair -n /dev/sda1
   ```

6. Ejecute el siguiente comando para reparar el sistema de archivos:

   ```
   xfs_repair /dev/sda1
   ```

7. Si recibe un mensaje de error que dice que el sistema de archivos tiene cambios de metadatos valiosos en un registro que se debe reproducir, cree un directorio temporal y monte el sistema de archivos:

   ```
   mkdir /temp
   mount /dev/sda1 /temp
   ```

8. Si el disco no se puede montar, ejecute el comando xfs_repair con la opción -L (que fuerza el llenado con ceros del registro):

   ```
   xfs_repair /dev/sda1 -L
   ```

9. A continuación, intente montar el sistema de archivos. Si el disco se monta correctamente, recibirá la siguiente salida:
 
   ```
   XFS (sda1): Mounting V1 Filesystem
   XFS (sda1): Ending clean mount
   ```

10. Reinicie la máquina virtual y, luego, compruebe si se ha resuelto el problema.

    ```
    Reboot -f
    ```

## <a name="repair-the-vm-offline"></a>Reparación de la máquina virtual sin conexión

1. Asocie el disco del sistema de la máquina virtual como un disco de datos a una máquina virtual de recuperación (cualquier máquina virtual Linux en funcionamiento). Para ello, puede usar [comandos de la CLI](./troubleshoot-recovery-disks-linux.md) o puede automatizar la configuración de la máquina de recuperación mediante los [comandos de reparación de la máquina virtual](repair-linux-vm-using-azure-virtual-machine-repair-commands.md).

2. Busque la etiqueta de unidad del disco del sistema que ha asociado. En este caso, se supone que la etiqueta del disco del sistema que ha asociado es/dev/sdc1. Reemplácela por el valor adecuado en su máquina virtual.

3. Use xfs_repair con la opción -n para detectar los errores en el sistema de archivos.

   ```
   xfs_repair -n /dev/sdc1
   ```

4. Ejecute el siguiente comando para reparar el sistema de archivos:

   ```
   xfs_repair /dev/sdc1
   ```

5. Si recibe un mensaje de error que dice que el sistema de archivos tiene cambios de metadatos valiosos en un registro que se debe reproducir, cree un directorio temporal y monte el sistema de archivos:

   ```
   mkdir /temp

   mount /dev/sdc1 /temp
   ```

   Si el disco no se puede montar, ejecute el comando xfs_repair con la opción -L (que fuerza el llenado con ceros del registro):

   ```
   xfs_repair /dev/sdc1 -L
   ```

6. A continuación, intente montar el sistema de archivos. Si el disco se monta correctamente, recibirá la siguiente salida:

   ```
   XFS (sdc1): Mounting V1 Filesystem
   
   XFS (sdc1): Ending clean mount
   ```

7. Desmonte y separe el disco duro virtual original y, luego, cree una máquina virtual a partir de él. Para ello, puede usar [comandos de la CLI](troubleshoot-recovery-disks-linux.md) o los [comandos de reparación de la máquina virtual](repair-linux-vm-using-azure-virtual-machine-repair-commands.md) si los ha usado para crear la máquina virtual de recuperación.

8. Compruebe si el problema se resuelve.

## <a name="next-steps"></a>Pasos siguientes

* [Solución de problemas de una máquina virtual Linux mediante la conexión del disco del sistema operativo a una máquina virtual de recuperación con la CLI de Azure 2.0](./troubleshoot-recovery-disks-linux.md)
* [Uso del portal para conectar un disco de datos a una máquina virtual Linux](../linux/attach-disk-portal.md)
