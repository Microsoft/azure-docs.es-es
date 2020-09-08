---
title: Tutorial sobre el desempaquetado, la conexión y el desbloqueo de Azure Data Box Disk| Microsoft Docs
description: En este tutorial, aprenderá a desempaquetar un dispositivo Azure Data Box Disk, a conectarse a discos, a obtener la clave de paso y a desbloquear discos de clientes de Windows y Linux.
services: databox
author: alkohli
ms.service: databox
ms.subservice: disk
ms.topic: tutorial
ms.date: 09/04/2019
ms.author: alkohli
ms.localizationpriority: high
ms.openlocfilehash: 570653d620c5e314db9e6a0bc56d731f8acf9296
ms.sourcegitcommit: 4f1c7df04a03856a756856a75e033d90757bb635
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "87925658"
---
::: zone target="docs"

# <a name="tutorial-unpack-connect-and-unlock-azure-data-box-disk"></a>Tutorial: Desempaquetado, conexión y desbloqueo de Azure Data Box Disk

Este tutorial describe cómo desempaquetar, conectar y desbloquear Azure Data Box Disk.

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Desempaquetado de Data Box Disk
> * Conexión a discos y obtención de la clave de paso
> * Desbloqueo de discos en el cliente de Windows
> * Desbloqueo de discos en el cliente de Linux

::: zone-end

::: zone target="chromeless"

## <a name="unpack-connect-and-unlock-azure-data-box-disk"></a>Desempaquetado, conexión y desbloqueo de Azure Data Box Disk

::: zone-end

::: zone target="docs"

## <a name="prerequisites"></a>Prerrequisitos

Antes de comenzar, asegúrese de que:

1. Ha completado el [Tutorial: Pedido de Azure Data Box](data-box-disk-deploy-ordered.md).
2. Ha recibido los discos y el estado del trabajo en el portal se actualiza a **Delivered** (Entregado).
3. Tiene un equipo cliente en el que puede instalar la herramienta de desbloqueo Data Box Disk. El equipo cliente debe:
    - Ejecutar un [sistema operativo admitido](data-box-disk-system-requirements.md#supported-operating-systems-for-clients).
    - Disponer de otro [software necesario](data-box-disk-system-requirements.md#other-required-software-for-windows-clients) instalado si es un cliente de Windows.  

## <a name="unpack-your-disks"></a>Desempaquetado de los discos

 Realice los pasos siguientes para desempaquetar los discos.

1. Data Box Disk se envía por correo en una caja pequeña. Abra la caja cuadro y extraiga su contenido. Compruebe que la caja contiene de 1 a 5 discos de estado sólido (SSD) y un cable USB de conexión por cada disco. Inspeccione la caja para comprobar si hay alguna evidencia de que haya sido alterada o de que haya sufrido algún otro daño evidente.

    ![Paquete de envío de Data Box Disk](media/data-box-disk-deploy-set-up/data-box-disk-ship-package1.png)

2. Si la caja de envío ha sido manipulada o ha sufrido daños graves, no la abra. Póngase en contacto con el Soporte técnico de Microsoft para que le ayude a evaluar si los discos están en buenas condiciones y si debe devolverlos para que se los cambien.
3. Compruebe que la caja tiene una funda clara que contiene una etiqueta de envío (bajo la etiqueta actual) para el envío de devolución. Si esta etiqueta se pierde o se daña, siempre puede descargar e imprimir una nueva desde Azure Portal.

    ![Etiqueta de envío de Data Box Disk](media/data-box-disk-deploy-set-up/data-box-disk-package-ship-label.png)

4. Guarde la caja y la espuma de embalaje para el envío de devolución de los discos.

## <a name="connect-to-disks-and-get-the-passkey"></a>Conexión a discos y obtención de la clave de paso

1. Utilice el cable incluido para conectar el disco al equipo cliente que ejecute un sistema operativo compatible como se indica en los requisitos previos.

    ![Conexión de Data Box Disk](media/data-box-disk-deploy-set-up/data-box-disk-connect-unlock.png)

2. En Azure Portal, vaya al pedido de Data Box Disk. Para buscarlo, desplácese a **General > Todos los recursos** y seleccione el pedido de Data Box Disk. Use el icono de copia para copiar la clave de paso. Esta clave de paso se usará para desbloquear los discos.

    ![Clave de paso de la herramienta de desbloqueo Data Box Disk](media/data-box-disk-deploy-set-up/data-box-disk-get-passkey.png)

Dependiendo de si está conectado a un cliente de Windows o de Linux, los pasos para desbloquear los discos son diferentes.

## <a name="unlock-disks-on-windows-client"></a>Desbloqueo de discos en el cliente de Windows

Realice los pasos siguientes para conectarse a los discos y para desbloquearlos.

1. En Azure Portal, vaya al pedido de Data Box Disk. Para buscarlo, desplácese a **General > Todos los recursos** y seleccione el pedido de Data Box Disk.
2. Descargue el conjunto de herramientas de Data Box Disk que se corresponde con el cliente de Windows. Este conjunto contiene 3 herramientas: Data Box Disk Unlock, Data Box Disk Validation y Data Box Disk Split Copy.

    En este procedimiento, solo usará la herramienta Data Box Disk Unlock. Las otras dos herramientas se usarán más adelante.

    > [!div class="nextstepaction"]
    > [Descargar conjunto de herramientas de Data Box Disk para Windows](https://aka.ms/databoxdisktoolswin)

3. Extraiga el conjunto de herramientas en el mismo equipo que va a utilizar para copiar los datos.
4. Abra una ventana del símbolo del sistema o ejecute Windows PowerShell como administrador en el mismo equipo.
5. (Opcional) Para comprobar que el equipo que va a usar para desbloquear el disco cumple los requisitos del sistema operativo, ejecute el comando de comprobación del sistema. A continuación se muestra una salida de ejemplo.

    ```powershell
    Windows PowerShell
    Copyright (C) Microsoft Corporation. All rights reserved.

    PS C:\DataBoxDiskUnlockTool\DiskUnlock> .\DataBoxDiskUnlock.exe /SystemCheck
    Successfully verified that the system can run the tool.
    PS C:\DataBoxDiskUnlockTool\DiskUnlock>
    ``` 

6. Ejecute `DataBoxDiskUnlock.exe` y proporcione la clave de paso que obtuvo en [Conexión a discos y obtención de la clave de paso](#connect-to-disks-and-get-the-passkey). Se muestra la letra de unidad asignada en el disco. A continuación se muestra una salida de ejemplo.

    ```powershell
    PS C:\WINDOWS\system32> cd C:\DataBoxDiskUnlockTool\DiskUnlock
    PS C:\DataBoxDiskUnlockTool\DiskUnlock> .\DataBoxDiskUnlock.exe
    Enter the passkey :
    testpasskey1

    Following volumes are unlocked and verified.
    Volume drive letters: D:

    PS C:\DataBoxDiskUnlockTool\DiskUnlock>
    ```

7. Repita los pasos de desbloqueo cada vez que tenga que volver a insertar un disco. Use el comando `help` si necesita ayuda con la herramienta de desbloqueo de Data Box Disk.

    ```powershell
    PS C:\DataBoxDiskUnlockTool\DiskUnlock> .\DataBoxDiskUnlock.exe /help
    USAGE:
    DataBoxUnlock /PassKey:<passkey_from_Azure_portal>

    Example: DataBoxUnlock /PassKey:<your passkey>
    Example: DataBoxUnlock /SystemCheck
    Example: DataBoxUnlock /Help

    /PassKey:        Get this passkey from Azure DataBox Disk order. The passkey unlocks your disks.
    /SystemCheck:    This option checks if your system meets the requirements to run the tool.
    /Help:           This option provides help on cmdlet usage and examples.

    PS C:\DataBoxDiskUnlockTool\DiskUnlock>
    ```  

8. Una vez que el disco está desbloqueado, puede ver su contenido.

    ![Contenido de Data Box Disk](media/data-box-disk-deploy-set-up/data-box-disk-content.png)

Si se encuentra con algún problema al desbloquear los discos, consulte cómo [solucionar los problemas de desbloqueo](data-box-disk-troubleshoot-unlock.md).

## <a name="unlock-disks-on-linux-client"></a>Desbloqueo de discos en el cliente de Linux

1. En Azure Portal, vaya a **General > Detalles del dispositivo**.
2. Descargue el conjunto de herramientas de Data Box Disk que se corresponde con el cliente de Linux.  

    > [!div class="nextstepaction"]
    > [Descargar conjunto de herramientas de Data Box Disk para Linux](https://aka.ms/databoxdisktoolslinux)

3. En el cliente Linux, abra un terminal. Vaya a la carpeta en la que descargó el software. Cambie los permisos de archivo para que pueda ejecutar estos archivos. Escriba el siguiente comando:

    `chmod +x DataBoxDiskUnlock_x86_64` 

    `chmod +x DataBoxDiskUnlock_Prep.sh` 

    A continuación se muestra una salida de ejemplo. Una vez que se ejecuta el comando chmod, puede comprobar que los permisos de archivo se han cambiado mediante la ejecución del comando `ls`. 
 
    ```
        [user@localhost Downloads]$ chmod +x DataBoxDiskUnlock_x86_64  
        [user@localhost Downloads]$ chmod +x DataBoxDiskUnlock_Prep.sh   
        [user@localhost Downloads]$ ls -l  
        -rwxrwxr-x. 1 user user 1152664 Aug 10 17:26 DataBoxDiskUnlock_x86_64  
        -rwxrwxr-x. 1 user user 795 Aug 5 23:26 DataBoxDiskUnlock_Prep.sh
    ```

4. Ejecute el script para que instale todos los archivos binarios necesarios para el software de desbloqueo de Data Box Disk. Use `sudo` para ejecutar el comando como raíz. Una vez que los archivos binarios se han instalado correctamente, aparecerá una nota a tal efecto en el terminal.

    `sudo ./DataBoxDiskUnlock_Prep.sh`

    El script comprobará primero si el equipo cliente está ejecutando un sistema operativo compatible. A continuación se muestra una salida de ejemplo. 
 
    ```
    [user@localhost Documents]$ sudo ./DataBoxDiskUnlock_Prep.sh 
        OS = CentOS Version = 6.9 
        Release = CentOS release 6.9 (Final) 
        Architecture = x64 

        The script will install the following packages and dependencies. 
        epel-release 
        dislocker 
        ntfs-3g 
        fuse-dislocker 
        Do you wish to continue? y|n :|
    ```

 
5. Escriba `y` para continuar la instalación. Los paquetes que instala el script son: 
   - **epel-release**: repositorio que contiene los tres paquetes siguientes. 
   - **dislocker y fuse-dislocker**: estas utilidades ayudan a descifrar los discos cifrados de BitLocker. 
   - **ntfs-3g**: paquete que ayuda a montar volúmenes NTFS. 
 
     Una vez que los paquetes se han instalado correctamente el terminal mostrará una notificación al respecto.     
     ```
     Dependency Installed: compat-readline5.x86 64 0:5.2-17.I.el6 dislocker-libs.x86 64 0:0.7.1-8.el6 mbedtls.x86 64 0:2.7.4-l.el6        ruby.x86 64 0:1.8.7.374-5.el6 
     ruby-libs.x86 64 0:1.8.7.374-5.el6 
     Complete! 
     Loaded plugins: fastestmirror, refresh-packagekit, security 
     Setting up Remove Process 
     Resolving Dependencies 
     --> Running transaction check 
     ---> Package epel-release.noarch 0:6-8 will be erased --> Finished Dependency Resolution 
     Dependencies Resolved 
     Package        Architecture        Version        Repository        Size 
     Removing:  epel-release        noarch         6-8        @extras        22 k 
     Transaction Summary                                 
     Remove        1 Package(s) 
     Installed size: 22 k 
     Downloading Packages: 
     Running rpmcheckdebug 
     Running Transaction Test 
     Transaction Test Succeeded 
     Running Transaction 
     Erasing : epel-release-6-8.noarch 
     Verifying : epel-release-6-8.noarch 
     Removed: 
     epel-release.noarch 0:6-8 
     Complete! 
     Dislocker is installed by the script. 
     OpenSSL is already installed.
     ```

6. Ejecute la herramienta de desbloqueo de Data Box Disk. Proporcione la clave de paso de Azure Portal que obtuvo en [Conexión a discos y obtención de la clave de paso](#connect-to-disks-and-get-the-passkey). Opcionalmente, especifique una lista de volúmenes cifrados de BitLocker para desbloquear. La lista de volúmenes y la clave de paso se debe especificar entre comillas simples. 

    Escriba el siguiente comando.
 
    `sudo ./DataBoxDiskUnlock_x86_64 /PassKey:'<Your passkey from Azure portal>'          

    A continuación se muestra la salida de ejemplo. 
 
    ```
    [user@localhost Downloads]$ sudo ./DataBoxDiskUnlock_x86_64 /Passkey:'qwerqwerqwer'  
    
    START: Mon Aug 13 14:25:49 2018 
    Volumes: /dev/sdbl 
    Passkey: qwerqwerqwer 
    
    Volumes for data copy : 
    /dev/sdbl: /mnt/DataBoxDisk/mountVoll/ 
    END: Mon Aug 13 14:26:02 2018
    ```
    Aparece el punto de montaje del volumen en el que puede copiar los datos.

7. Repita los pasos de desbloqueo cada vez que tenga que volver a insertar un disco. Use el comando `help` si necesita ayuda con la herramienta de desbloqueo de Data Box Disk. 
    
    `sudo ./DataBoxDiskUnlock_x86_64 /Help` 

    A continuación se muestra la salida de ejemplo. 
 
    ```
    [user@localhost Downloads]$ sudo ./DataBoxDiskUnlock_x86_64 /Help  
    START: Mon Aug 13 14:29:20 2018 
    USAGE: 
    sudo DataBoxDiskUnlock /PassKey:'<passkey from Azure_portal>' 
    
    Example: sudo DataBoxDiskUnlock /PassKey:'passkey' 
    Example: sudo DataBoxDiskUnlock /PassKey:'passkey' /Volumes:'/dev/sdbl' 
    Example: sudo DataBoxDiskUnlock /Help Example: sudo DataBoxDiskUnlock /Clean 
    
    /PassKey: This option takes a passkey as input and unlocks all of your disks. 
    Get the passkey from your Data Box Disk order in Azure portal. 
    /Volumes: This option is used to input a list of BitLocker encrypted volumes. 
    /Help: This option provides help on the tool usage and examples. 
    /Unmount: This option unmounts all the volumes mounted by this tool. 
   
    END: Mon Aug 13 14:29:20 2018 [user@localhost Downloads]$
    ```
    
8. Una vez que el disco está desbloqueado, puede ir al punto de montaje y ver su contenido. Ahora está listo para copiar los datos en las carpetas *BlockBlob* o *PageBlob*. 

    ![Contenido de Data Box Disk](media/data-box-disk-deploy-set-up/data-box-disk-content-linux.png)


Si se encuentra con algún problema al desbloquear los discos, consulte cómo [solucionar los problemas de desbloqueo](data-box-disk-troubleshoot-unlock.md). 

::: zone-end

::: zone target="chromeless"

1. Desempaquete los discos y use el cable incluido para conectar el disco al equipo cliente.
2. Descargue y extraiga el conjunto de herramientas de Data Box Disk en el mismo equipo que va a usar para copiar los datos.

    > [!div class="nextstepaction"]
    > [Descargar conjunto de herramientas de Data Box Disk para Windows](https://aka.ms/databoxdisktoolswin)

    or
    > [!div class="nextstepaction"]
    > [Descargar conjunto de herramientas de Data Box Disk para Linux](https://aka.ms/databoxdisktoolslinux) 

3. Para desbloquear los discos en un cliente Windows, abra una ventana del símbolo del sistema o ejecute Windows PowerShell como administrador en el mismo equipo:

    - Escriba el siguiente comando en la misma carpeta donde está instalada la herramienta de desbloqueo de Data Box Disk.

        ``` 
        .\DataBoxDiskUnlock.exe
        ```
    -  Obtenga la clave de paso que obtuvo en **General > Detalles del dispositivo** en Azure Portal y proporciónela aquí. Se muestra la letra de unidad asignada en el disco. 
4. Para desbloquear los discos en un cliente Linux, abra un terminal. Vaya a la carpeta en la que descargó el software. Escriba los siguientes comandos para cambiar los permisos de archivo de forma que pueda ejecutar estos archivos: 

    ```
    chmod +x DataBoxDiskUnlock_x86_64
    chmod +x DataBoxDiskUnlock_Prep.sh
    ``` 
    Ejecute el script para instalar todos los archivos binarios necesarios.

    ```
    sudo ./DataBoxDiskUnlock_Prep.sh
    ```
    Ejecute la herramienta de desbloqueo de Data Box Disk. Obtenga la clave de paso que obtuvo en **General > Detalles del dispositivo** en Azure Portal y proporciónela aquí. Opcionalmente, especifique una lista de volúmenes cifrados de BitLocker entre comillas sencillas para desbloquear.

    ```
    sudo ./DataBoxDiskUnlock_x86_64 /PassKey:'<Your passkey from Azure portal>'
    ```      
5. Repita los pasos de desbloqueo cada vez que tenga que volver a insertar un disco. Use el comando de ayuda si necesita ayuda con la herramienta de desbloqueo de Data Box Disk.

Una vez que el disco está desbloqueado, puede ver su contenido.

Para más información sobre cómo configurar y desbloquear discos, vaya a [Configuración de Data Box Disk](data-box-disk-deploy-set-up.md).

::: zone-end

::: zone target="docs"

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha aprendido sobre temas relacionados con Azure Data Box Disk; por ejemplo:

> [!div class="checklist"]
> * Desempaquetado de Data Box Disk
> * Conexión a discos y obtención de la clave de paso
> * Desbloqueo de discos en el cliente de Windows
> * Desbloqueo de discos en el cliente de Linux


Pase al siguiente tutorial para aprender a copiar datos en Data Box Disk.

> [!div class="nextstepaction"]
> [Copia de datos en Data Box Disk](./data-box-disk-deploy-copy-data.md)

::: zone-end

