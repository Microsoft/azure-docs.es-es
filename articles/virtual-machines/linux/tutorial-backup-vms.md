---
title: 'Tutorial: Copia de seguridad de máquinas virtuales Linux en Azure Portal'
description: En este tutorial, aprenderá a usar Azure Portal para proteger las máquinas virtuales Linux con Azure Backup.
services: virtual-machines-linux
documentationcenter: virtual-machines
author: cynthn
manager: gwallace
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-machines-linux
ms.topic: tutorial
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 07/27/2017
ms.author: cynthn
ms.custom: mvc
ms.openlocfilehash: 1865d307e7b79e7e40b0e98f2dcfa2535b0313fd
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88551714"
---
# <a name="tutorial-back-up-and-restore-files-for-linux-virtual-machines-in-azure"></a>Tutorial: Copia de seguridad y restauración de archivos en máquinas virtuales Linux en Azure

Para proteger sus datos realice copias de seguridad a intervalos regulares. Azure Backup crea puntos de recuperación que se almacenan en almacenes de recuperación con redundancia geográfica. Cuando se realiza una restauración desde un punto de recuperación, se puede restaurar toda la máquina virtual o determinados archivos. En este artículo se explica cómo restaurar un archivo individual a una máquina virtual Linux en la que se ejecuta nginx. Si aún no tiene una máquina virtual que pueda usar, siga las instrucciones que se proporcionan en [Creación de una máquina virtual Linux con la CLI de Azure](quick-create-cli.md). En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Crear una copia de seguridad de una máquina virtual.
> * Programar una copia de seguridad diaria.
> * Restaurar un archivo desde una copia de seguridad.

## <a name="backup-overview"></a>Introducción a Backup

Cuando el servicio Azure Backup inicia una copia de seguridad, desencadena que la extensión de copia de seguridad tome una instantánea de un momento dado. En Linux, el servicio Azure Backup usa la extensión _VMSnapshotLinux_. La extensión se instala cuando se realiza la primera copia de seguridad de la máquina virtual, en caso de que esta esté en ejecución. Si no se está ejecutando la máquina virtual, el servicio Azure Backup toma una instantánea del almacenamiento subyacente (ya que no se produce ninguna escritura de la aplicación mientras se detiene la máquina virtual).

De forma predeterminada, Azure Backup toma una instantánea coherente con el sistema de archivos de la máquina virtual Linux, pero puede configurarse para que tome una [instantánea coherente con la aplicación mediante el marco de script anterior y posterior](../../backup/backup-azure-linux-app-consistent.md). Después de que el servicio Azure Backup toma la instantánea, se transfieren los datos al almacén. Para que el proceso resulte más eficaz, el servicio identifica y transfiere únicamente los bloques de datos que han cambiado desde la última copia de seguridad.

Cuando finaliza la transferencia de datos, se elimina la instantánea y se crea un punto de recuperación.


## <a name="create-a-backup"></a>Creación de una copia de seguridad
Cree una copia de seguridad diaria programada en un almacén de Recovery Services:

1. Inicie sesión en [Azure Portal](https://portal.azure.com/).
2. En el menú de la izquierda, haga clic en **Máquinas virtuales**. 
3. En la lista, seleccione la máquina virtual de la que quiere realizar una copia de seguridad.
4. En la hoja de la máquina virtual, en la sección **Operaciones**, haga clic en **Copia de seguridad**. Se abre la hoja **Habilitar copia de seguridad**.
5. En **Almacén de Recovery Services**, haga clic en **Create new** (Crear nuevo) y especifique el nombre del nuevo almacén. Se crea un nuevo almacén en el grupo de recursos y ubicación en que se encuentra la máquina virtual.
6. Haga clic en **Directiva de copia de seguridad**. En este ejemplo, conserve los valores predeterminados y haga clic en **Aceptar**.
7. En la hoja **Habilitar copia de seguridad**, haga clic en **Habilitar copia de seguridad**. De esta forma se crea una copia de seguridad diaria según la programación predeterminada.
10. Para crear un punto de recuperación inicial, en la hoja **Copia de seguridad** haga clic en **Realizar copia de seguridad ahora**.
11. En la hoja **Realizar copia de seguridad ahora**, haga clic en el icono del calendario, use el control de calendario para seleccionar el último día en que se mantendrá este punto de recuperación y haga clic en **Copia de seguridad**.
12. En la hoja **Copia de seguridad** de la máquina virtual, verá el número de puntos de recuperación completos.

    ![Puntos de recuperación](./media/tutorial-backup-vms/backup-complete.png)

La primera copia de seguridad tarda aproximadamente 20 minutos. Cuando la copia de seguridad finalice, pase a la parte siguiente de este tutorial.

## <a name="restore-a-file"></a>Restauración de un archivo

Si accidentalmente elimina o realiza cambios en un archivo, puede usar Recuperación de archivos para recuperar el archivo del almacén de Backup. La recuperación de archivos usa un script que se ejecuta en la máquina virtual para montar el punto de recuperación como una unidad local. Estas unidades permanecen montadas durante 12 horas para que pueda copiar archivos desde el punto de recuperación y restaurarlos en la máquina virtual.  

En este ejemplo, se muestra cómo recuperar la página web predeterminada de nginx, /var/www/html/index.nginx-debian.html. En este ejemplo, la dirección IP pública de la máquina virtual es *13.69.75.209*. Para encontrar la dirección IP de una máquina virtual, utilice:

 ```azurecli
 az vm show --resource-group myResourceGroup --name myVM -d --query [publicIps] --o tsv
 ```

 
1. En el equipo local, abra un explorador y escriba la dirección IP pública de la máquina virtual para ver la página web predeterminada de nginx.

    ![Página web predeterminada de nginx](./media/tutorial-backup-vms/nginx-working.png)

1. Utilice SSH en la máquina virtual.

    ```bash
    ssh 13.69.75.209
    ```

2. Elimine /var/www/html/index.nginx-debian.html.

    ```bash
    sudo rm /var/www/html/index.nginx-debian.html
    ```
    
4. En el equipo local, actualice el explorador presionando CTRL + F5 para ver que la página predeterminada de nginx ya no existe.

    ![Página web predeterminada de nginx](./media/tutorial-backup-vms/nginx-broken.png)
    
1. En el equipo local, inicie sesión en [Azure Portal](https://portal.azure.com/).
6. En el menú de la izquierda, haga clic en **Máquinas virtuales**. 
7. En la lista, seleccione la máquina virtual.
8. En la hoja de la máquina virtual, en la sección **Configuración**, haga clic en **Copia de seguridad**. Se abre la hoja **Copia de seguridad**. 
9. En el menú de la parte superior de la hoja, seleccione **Recuperación de archivos**. Se abrirá la hoja **Recuperación de archivos**.
10. En **Paso 1: Seleccionar punto de recuperación**, seleccione un punto de recuperación en la lista desplegable.
11. En el **Paso 2: Descargar script para examinar y recuperar archivos**, haga clic en el botón **Descargar ejecutable**. Guarde el archivo descargado en un equipo local.
7. Haga clic en **Descargar script** para descargar el archivo de script localmente.
8. Abra un símbolo del sistema de Bash y escriba lo siguiente, pero reemplace *Linux_myVM_05-05-2017.sh* por la ruta de acceso y el nombre de archivo correctos del script que ha descargado, *azureuser* por el nombre de usuario de la máquina virtual y *13.69.75.209* por la dirección IP pública de la máquina virtual.
    
    ```bash
    scp Linux_myVM_05-05-2017.sh azureuser@13.69.75.209:
    ```
    
9. En el equipo local, abra una conexión SSH a la máquina virtual.

    ```bash
    ssh 13.69.75.209
    ```
    
10. En la máquina virtual, agregue permisos de ejecución al archivo de script.

    ```bash
    chmod +x Linux_myVM_05-05-2017.sh
    ```
    
11. En la máquina virtual, ejecute el script para montar el punto de recuperación como sistema de archivos.

    ```bash
    ./Linux_myVM_05-05-2017.sh
    ```
    
12. La salida del script proporciona la ruta de acceso del punto de montaje. La salida es similar a esta:

    ```output
    Microsoft Azure VM Backup - File Recovery
    ______________________________________________
                          
    Connecting to recovery point using ISCSI service...
    
    Connection succeeded!
    
    Please wait while we attach volumes of the recovery point to this machine...
                         
    ************ Volumes of the recovery point and their mount paths on this machine ************

    Sr.No.  |  Disk  |  Volume  |  MountPath 

    1)  | /dev/sdc  |  /dev/sdc1  |  /home/azureuser/myVM-20170505191055/Volume1

    ************ Open File Explorer to browse for files. ************

    After recovery, to remove the disks and close the connection to the recovery point, please click 'Unmount Disks' in step 3 of the portal.

    Please enter 'q/Q' to exit...
    ```

12. En la máquina virtual, copie la página web predeterminada de nginx desde el punto de montaje al lugar en que se eliminó el archivo.

    ```bash
    sudo cp ~/myVM-20170505191055/Volume1/var/www/html/index.nginx-debian.html /var/www/html/
    ```
    
17. En el equipo local, abra la pestaña del explorador en el que está conectado a la dirección IP de la máquina virtual que muestra la página predeterminada de nginx. Presione CTRL + F5 para actualizar la página del explorador. Ahora debería ver que la página predeterminada vuelve a funcionar.

    ![Página web predeterminada de nginx](./media/tutorial-backup-vms/nginx-working.png)

18. En el equipo local, vuelva a la pestaña del explorador de Azure Portal y, en **Paso 3: Desmontar los discos después de la recuperación**, haga clic en el botón **Desmontar discos**. Si olvida realizar este paso, la conexión al punto de montaje se cierra automáticamente al cabo de 12 horas. A las 12 horas, es preciso que descargue un script nuevo para crear un nuevo punto de montaje.


## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha aprendido a:

> [!div class="checklist"]
> * Crear una copia de seguridad de una máquina virtual.
> * Programar una copia de seguridad diaria.
> * Restaurar un archivo desde una copia de seguridad.

En el siguiente tutorial se explica cómo supervisar máquinas virtuales.

> [!div class="nextstepaction"]
> [Control de máquinas virtuales](tutorial-govern-resources.md)
