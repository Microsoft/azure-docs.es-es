---
title: Restablecimiento de una contraseña local de Windows sin el agente de Azure| Microsoft Docs
description: Restablecimiento de la contraseña de una cuenta de usuario de Windows local cuando el agente invitado de Azure no está instalado o funcionando en una VM
services: virtual-machines-windows
documentationcenter: ''
author: genlin
manager: dcscontentpm
editor: ''
ms.assetid: cf353dd3-89c9-47f6-a449-f874f0957013
ms.service: virtual-machines-windows
ms.topic: troubleshooting
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure-services
ms.date: 04/25/2019
ms.author: genli
ms.openlocfilehash: cb2f08c4788c90f8bdb2af9c6ef95fd1ac43b994
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87028675"
---
# <a name="reset-local-windows-password-for-azure-vm-offline"></a>Restablecimiento de una contraseña de Windows local para VM de Azure sin conexión
Puede restablecer la contraseña de Windows local de una VM en Azure mediante [Azure Portal o Azure PowerShell](reset-rdp.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json) siempre que el agente invitado de Azure esté instalado. Este método es la manera principal de restablecer una contraseña para una VM de Azure. Si tiene problemas con el agente de invitado de Azure, como puede ser que no responda o que no se pueda instalar después de cargar una imagen personalizada, puede restablecer manualmente una contraseña de Windows. En este artículo se detalla cómo restablecer la contraseña de una cuenta local asociando el disco virtual de SO de origen a otra VM. Los pasos descritos en este artículo no se aplican a los controladores de dominio de Windows. 

> [!WARNING]
> Utilice este proceso solamente como último recurso. Intente siempre restablecer una contraseña mediante [Azure Portal o Azure PowerShell](reset-rdp.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json) primero.

## <a name="overview-of-the-process"></a>Información general del proceso
Los pasos principales para realizar un restablecimiento de contraseña para una VM con Windows en Azure cuando no hay acceso al agente invitado de Azure son los siguientes:

1. Detenga la máquina virtual afectada.
1. Cree una instantánea del disco del sistema operativo de la máquina virtual.
1. Cree una copia del disco del sistema operativo desde la instantánea.
1. Conecte y monte el disco del sistema operativo copiado en otra máquina virtual Windows y, a continuación, cree algunos archivos de configuración en el disco. Los archivos le ayudarán a restablecer la contraseña.
1. Desmonte y desasocie el disco del sistema operativo copiado de la máquina virtual de solución de problemas.
1. Cambie el disco del sistema operativo de la máquina virtual afectada.

## <a name="detailed-steps-for-the-vm-with-resource-manager-deployment"></a>Pasos detallados para la máquina virtual con la implementación de Resource Manager

> [!NOTE]
> Los pasos no se aplican a los controladores de dominio de Windows. Solo funcionan en un servidor independiente o en uno que sea miembro de un dominio.

Intente siempre restablecer una contraseña mediante [Azure Portal o Azure PowerShell](reset-rdp.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json) antes de intentar llevar a cabo los pasos siguientes. Asegúrese de que tiene una copia de seguridad de la VM antes de empezar.

1. Realice una instantánea del disco del sistema operativo de la máquina virtual afectada, cree un disco a partir de la instantánea y, a continuación, conecte el disco a una máquina virtual de solución de problemas. Para más información, consulte [Solución de problemas de una máquina virtual Windows mediante la conexión del disco del sistema operativo a una máquina virtual de recuperación mediante Azure Portal](troubleshoot-recovery-disks-portal-windows.md).
2. Conéctese a la máquina virtual de solución de problemas mediante Escritorio remoto.
3. Cree `gpt.ini` en `\Windows\System32\GroupPolicy` en la unidad de la VM de origen (si gpt.ini existe, cambie el nombre a gpt.ini.bak):
   
   > [!WARNING]
   > Asegúrese de que no crea accidentalmente los siguientes archivos en C:\Windows, la unidad del SO para la VM de solución de problemas. Cree los siguientes archivos en la unidad del SO para la VM que se asocia como disco de datos.
   
   * Agregue las líneas siguientes al archivo `gpt.ini` que creó:
     
     ```
     [General]
     gPCFunctionalityVersion=2
     gPCMachineExtensionNames=[{42B5FAAE-6536-11D2-AE5A-0000F87571E3}{40B6664F-4972-11D1-A7CA-0000F87571E3}]
     Version=1
     ```
     
     ![Creación de gpt.ini](./media/reset-local-password-without-agent/create-gpt-ini.png)

4. Cree `scripts.ini` en `\Windows\System32\GroupPolicy\Machine\Scripts\`. Asegúrese de que se muestran las carpetas ocultas. Si es necesario, cree las carpetas `Machine` o `Scripts`.
   
   * Agregue las líneas siguientes al archivo `scripts.ini` que creó:
     
     ```
     [Startup]
     0CmdLine=C:\Windows\System32\FixAzureVM.cmd
     0Parameters=
     ```
     
     ![Creación de scripts.ini](./media/reset-local-password-without-agent/create-scripts-ini.png)

5. Crear `FixAzureVM.cmd` en `\Windows\System32` con el contenido siguiente, reemplazando `<username>` y `<newpassword>` por sus propios valores:
   
    ```
    net user <username> <newpassword> /add
    net localgroup administrators <username> /add
    net localgroup "remote desktop users" <username> /add
    ```

    ![Creación de FixAzureVM.cmd](./media/reset-local-password-without-agent/create-fixazure-cmd.png)
   
    Al definir la nueva contraseña debe cumplir los requisitos de complejidad de contraseña configurada para la VM.

6. En Azure Portal, desasocie el disco de la máquina virtual de solución de problemas.

7. [Cambie el disco del sistema operativo de la máquina virtual afectada](troubleshoot-recovery-disks-portal-windows.md#swap-the-os-disk-for-the-vm).

8. Después de que la CM se ejecute, conéctese a la VM mediante Escritorio remoto con la nueva contraseña que especificó en el script `FixAzureVM.cmd`.

9. En la sesión remota respecto a la nueva VM, quite los archivos siguientes para limpiar el entorno:
    
    * De %windir%\System32
      * quite FixAzureVM.cmd
    * De %windir%\System32\GroupPolicy\Machine\Scripts
      * quite scripts.ini
    * De %windir%\System32\GroupPolicy
      * quite gpt.ini (si gpt.ini existía antes y cambió su nombre a gpt.ini.bak, vuelva a cambiar el nombre del archivo .bak a gpt.ini)

## <a name="detailed-steps-for-classic-vm"></a>Pasos detallados para la máquina virtual clásica

[!INCLUDE [classic-vm-deprecation](../../../includes/classic-vm-deprecation.md)]

> [!NOTE]
> Los pasos no se aplican a los controladores de dominio de Windows. Solo funcionan en un servidor independiente o en uno que sea miembro de un dominio.

Intente siempre restablecer una contraseña mediante [Azure Portal o Azure PowerShell](/previous-versions/azure/virtual-machines/windows/classic/reset-rdp) antes de intentar llevar a cabo los pasos siguientes. Asegúrese de que tiene una copia de seguridad de la VM antes de empezar. 

1. Elimine la VM afectada de Azure Portal. La eliminación de la VM, solo elimina los metadatos, la referencia de la VM dentro de Azure. Los discos virtuales se conservan cuando se elimina la VM:
   
   * Seleccione la VM en Azure Portal y luego haga clic en *Eliminar*:
     
     ![Eliminación de la VM existente](./media/reset-local-password-without-agent/delete-vm-classic.png)

2. Asocie el disco del SO de la VM de origen a la VM de solución de problemas. La VM de solución de problemas debe estar en la misma región que el disco del SO de la VM de origen (como `West US`):
   
   1. Seleccione la VM de solución de problemas en Azure Portal. Haga clic en *Discos* | *Asociar existente*:
     
      ![Conectar disco existente](./media/reset-local-password-without-agent/disks-attach-existing-classic.png)
     
   2. Seleccione *Archivo VHD* y seleccione la cuenta de almacenamiento que contiene su VM de origen:
     
      ![Selección de la cuenta de almacenamiento](./media/reset-local-password-without-agent/disks-select-storage-account-classic.png)
     
   3. Active la casilla *Mostrar cuentas de almacenamiento clásico* y, después, seleccione el contenedor de origen. El contenedor de origen suele ser *vhds*:
     
      ![Selección del contenedor de almacenamiento](./media/reset-local-password-without-agent/disks-select-container-classic.png)

      ![Selección del contenedor de almacenamiento](./media/reset-local-password-without-agent/disks-select-container-vhds-classic.png)
     
   4. Seleccione el disco duro virtual de SO para asociar. Haga clic en *Seleccionar* para completar el proceso:
     
      ![Seleccione el disco virtual de origen](./media/reset-local-password-without-agent/disks-select-source-vhd-classic.png)

   5. Haga clic en Aceptar para conectar el disco

      ![Conectar disco existente](./media/reset-local-password-without-agent/disks-attach-okay-classic.png)

3. Conéctese a la VM de solución de problemas mediante Escritorio remoto y asegúrese de que el disco del SO de la VM de origen está visible:

   1. Seleccione la VM de solución de problemas en Azure Portal y haga clic en *Conectar*.

   2. Abra el archivo RDP que descarga. Escriba el nombre de usuario y la contraseña de la VM de solución de problemas.

   3. En el Explorador de archivos, busque el disco de datos que se ha asociado. Si el origen de disco duro virtual de la VM es el único disco de datos asociado a la VM de solución de problemas, debe ser la unidad F:
     
      ![Visualización del disco de datos conectado](./media/reset-local-password-without-agent/troubleshooting-vm-file-explorer-classic.png)

4. Cree `gpt.ini` en `\Windows\System32\GroupPolicy` en la unidad de la VM de origen (si `gpt.ini` existe, cambie el nombre a `gpt.ini.bak`):
   
   > [!WARNING]
   > Asegúrese de que no crea accidentalmente los siguientes archivos en `C:\Windows`, la unidad del SO para la VM de solución de problemas. Cree los siguientes archivos en la unidad del SO para la VM que se asocia como disco de datos.
   
   * Agregue las líneas siguientes al archivo `gpt.ini` que creó:
     
     ```
     [General]
     gPCFunctionalityVersion=2
     gPCMachineExtensionNames=[{42B5FAAE-6536-11D2-AE5A-0000F87571E3}{40B6664F-4972-11D1-A7CA-0000F87571E3}]
     Version=1
     ```
     
     ![Creación de gpt.ini](./media/reset-local-password-without-agent/create-gpt-ini-classic.png)

5. Cree `scripts.ini` en `\Windows\System32\GroupPolicy\Machines\Scripts\`. Asegúrese de que se muestran las carpetas ocultas. Si es necesario, cree las carpetas `Machine` o `Scripts`.
   
   * Agregue las líneas siguientes al archivo `scripts.ini` que creó:

     ```
     [Startup]
     0CmdLine=C:\Windows\System32\FixAzureVM.cmd
     0Parameters=
     ```
     
     ![Creación de scripts.ini](./media/reset-local-password-without-agent/create-scripts-ini-classic.png)

6. Crear `FixAzureVM.cmd` en `\Windows\System32` con el contenido siguiente, reemplazando `<username>` y `<newpassword>` por sus propios valores:
   
    ```
    net user <username> <newpassword> /add
    net localgroup administrators <username> /add
    net localgroup "remote desktop users" <username> /add
    ```

    ![Creación de FixAzureVM.cmd](./media/reset-local-password-without-agent/create-fixazure-cmd-classic.png)
   
    Al definir la nueva contraseña debe cumplir los requisitos de complejidad de contraseña configurada para la VM.

7. En Azure Portal, desasocie el disco de la VM de solución de problemas:
   
   1. Seleccione la VM de solución de problemas en Azure Portal y haga clic en *Discos*.
   
   2. Seleccione el disco de datos asociado en el paso 2, haga clic en **Desasociar:** y luego en **Aceptar**.

     ![Desasociación del disco](./media/reset-local-password-without-agent/data-disks-classic.png)
     
     ![Desasociación del disco](./media/reset-local-password-without-agent/detach-disk-classic.png)

8. Cree una VM desde el disco del SO de la VM de origen:
   
     ![Creación de una VM a partir de una plantilla](./media/reset-local-password-without-agent/create-new-vm-from-template-classic.png)

     ![Creación de una VM a partir de una plantilla](./media/reset-local-password-without-agent/choose-subscription-classic.png)

     ![Creación de una VM a partir de una plantilla](./media/reset-local-password-without-agent/create-vm-classic.png)

## <a name="complete-the-create-virtual-machine-experience"></a>Completar la experiencia de Crear máquina virtual

1. Después de que la CM se ejecute, conéctese a la VM mediante Escritorio remoto con la nueva contraseña que especificó en el script `FixAzureVM.cmd`.

2. En la sesión remota respecto a la nueva VM, quite los archivos siguientes para limpiar el entorno:
    
    * De `%windir%\System32`
      * quite `FixAzureVM.cmd`
    * De `%windir%\System32\GroupPolicy\Machine\Scripts`
      * quite `scripts.ini`
    * De `%windir%\System32\GroupPolicy`
      * quite `gpt.ini` (si `gpt.ini` existía antes y le cambió el nombre a `gpt.ini.bak`, vuelva a cambiar el nombre del archivo `.bak` a `gpt.ini`).

## <a name="next-steps"></a>Pasos siguientes
Si sigue sin poder conectarse mediante Escritorio remoto, consulte la [guía de solución de problemas de RDP](troubleshoot-rdp-connection.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json). La [guía detallada de solución de problemas de RDP](detailed-troubleshoot-rdp.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json) examina los métodos de solución de problemas en lugar de pasos específicos. También puede [abrir una solicitud de soporte técnico de Azure](https://azure.microsoft.com/support/options/) para obtener ayuda práctica.
