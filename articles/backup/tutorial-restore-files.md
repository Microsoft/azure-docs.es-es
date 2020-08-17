---
title: 'Tutorial: Restauración de archivos en una máquina virtual con Azure Backup'
description: Obtenga información acerca de cómo realizar la restauración en el nivel de archivo en una máquina virtual de Azure con Backup and Recovery Services.
ms.topic: tutorial
ms.date: 01/31/2019
ms.custom: mvc
ms.openlocfilehash: e74fd14c0f33c8633165029a18582b80810184a2
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87003515"
---
# <a name="restore-files-to-a-virtual-machine-in-azure"></a>Restauración de archivos en una máquina virtual de Azure

Azure Backup crea puntos de recuperación que se almacenan en almacenes de recuperación con redundancia geográfica. Cuando se realiza una restauración desde un punto de recuperación, se puede restaurar toda una máquina virtual o archivos individuales. En este artículo se detalla cómo restaurar archivos individuales. En este tutorial, aprenderá a:

> [!div class="checklist"]
>
> * Enumerar y seleccionar puntos de recuperación
> * Conectar un punto de recuperación a una máquina virtual
> * Restaurar archivos desde un punto de recuperación

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

Si decide instalar y usar la CLI localmente, para este tutorial es preciso que ejecute la CLI de Azure versión 2.0.18 o posterior. Ejecute `az --version` para encontrar la versión. Si necesita instalarla o actualizarla, consulte [Instalación de la CLI de Azure](/cli/azure/install-azure-cli).

## <a name="prerequisites"></a>Requisitos previos

Para este tutorial se necesita una máquina virtual Linux protegida con Azure Backup. Para simular un proceso de recuperación y la eliminación accidental de archivos, elimine una página desde un servidor web. Si necesita una máquina virtual Linux que ejecute un servidor web y esté protegida con Azure Backup, consulte [Copia de seguridad de una máquina virtual en Azure con la CLI](quick-backup-vm-cli.md).

## <a name="backup-overview"></a>Introducción a Backup

Cuando Azure inicia una copia de seguridad, la extensión de copia de seguridad en la máquina virtual toma una instantánea de un momento dado. La extensión de copia de seguridad se instala en la máquina virtual cuando se solicita la primera copia de seguridad. Azure Backup también puede tomar una instantánea del almacenamiento subyacente si la máquina virtual no se está ejecutando cuando se realiza la copia de seguridad.

De forma predeterminada, Azure Backup toma una copia de seguridad coherente del sistema de archivos. Después de que el servicio Azure Backup tome la instantánea, los datos se transfieren al almacén de Recovery Services. Para que el proceso resulte más eficaz, Azure Backup identifica y transfiere únicamente los bloques de datos que han cambiado desde la última copia de seguridad.

Cuando finaliza la transferencia de datos, se elimina la instantánea y se crea un punto de recuperación.

## <a name="delete-a-file-from-a-vm"></a>Eliminación de un archivo de una máquina virtual

Si elimina un archivo o realiza cambios en un archivo accidentalmente, puede restaurar archivos individuales desde un punto de recuperación. Este proceso permite examinar los archivos de los que se ha realizado una copia de seguridad en un punto de recuperación y restaurar solo los archivos necesarios. En este ejemplo, se elimina un archivo de un servidor web para demostrar el proceso de recuperación de nivel de archivo.

1. Para conectarse a su máquina virtual, obtenga su dirección IP con [az vm show](/cli/azure/vm?view=azure-cli-latest#az-vm-show):

     ```azurecli-interactive
     az vm show --resource-group myResourceGroup --name myVM -d --query [publicIps] --o tsv
     ```

2. Para comprobar que el sitio web funciona actualmente, abra un explorador web en la dirección IP pública de la máquina virtual. Deje abierta la ventana del explorador web.

    ![Página web predeterminada de NGINX](./media/tutorial-restore-files/nginx-working.png)

3. Conéctese a la máquina virtual mediante SSH. Reemplace *publicIpAddress* por la dirección IP pública que obtuvo en un comando anterior:

    ```bash
    ssh publicIpAddress
    ```

4. Elimine la página predeterminada del servidor web en */var/www/html/index.nginx-debian.html* como se indica a continuación:

    ```bash
    sudo rm /var/www/html/index.nginx-debian.html
    ```

5. En el explorador web, actualice la página web. El sitio web ya no carga la página, tal como se muestra en el ejemplo siguiente:

    ![El sitio web de NGINX ya no carga la página predeterminada](./media/tutorial-restore-files/nginx-broken.png)

6. Cierre la sesión de SSH de la máquina virtual de la manera siguiente:

    ```bash
    exit
    ```

## <a name="generate-file-recovery-script"></a>Generar un script de recuperación de archivos

Para restaurar sus archivos, Azure Backup proporciona un script que se ejecuta en la máquina virtual y que conecta el punto de recuperación como una unidad local. Puede examinar esta unidad local, restaurar archivos en la propia máquina virtual y, a continuación, desconectar el punto de recuperación. Azure Backup continúa realizando la copia seguridad de los datos de acuerdo con la directiva de retención y programación asignada.

1. Para enumerar los puntos de recuperación de la máquina virtual, use [az backup recoverypoint list](/cli/azure/backup/recoverypoint?view=azure-cli-latest#az-backup-recoverypoint-list). En este ejemplo, se selecciona el punto de recuperación más reciente de la máquina virtual denominada *myVM* que está protegida en *myRecoveryServicesVault*:

    ```azurecli-interactive
    az backup recoverypoint list \
        --resource-group myResourceGroup \
        --vault-name myRecoveryServicesVault \
        --container-name myVM \
        --item-name myVM \
        --query [0].name \
        --output tsv
    ```

2. Para obtener el script que conecta el punto de recuperación a la máquina virtual o lo monta en esta, use [az backup restore files mount-rp](/cli/azure/backup/restore/files?view=azure-cli-latest#az-backup-restore-files-mount-rp). En el ejemplo siguiente se obtiene el script de la máquina virtual denominada *myVM* que está protegida en *myRecoveryServicesVault*.

    Reemplace *myRecoveryPointName* por el nombre del punto de recuperación que obtuvo en el comando anterior:

    ```azurecli-interactive
    az backup restore files mount-rp \
        --resource-group myResourceGroup \
        --vault-name myRecoveryServicesVault \
        --container-name myVM \
        --item-name myVM \
        --rp-name myRecoveryPointName
    ```

    El script se descarga y se muestra una contraseña, como en el ejemplo siguiente:

    ```output
    File downloaded: myVM_we_1571974050985163527.sh. Use password c068a041ce12465
    ```

3. Para transferir el script a la máquina virtual, use Secure Copy (SCP). Proporcione el nombre del script descargado y reemplace *publicIpAddress* por la dirección IP pública de la máquina virtual. Asegúrese de incluir el signo `:` final al final del comando SCP como se indica a continuación:

    ```bash
    scp myVM_we_1571974050985163527.sh 52.174.241.110:
    ```

## <a name="restore-file-to-your-vm"></a>Restaurar archivos en la máquina virtual

Con el script de recuperación copiado en la máquina virtual, ahora puede conectar el punto de recuperación y restaurar archivos.

>[!NOTE]
> Consulte [aquí](backup-azure-restore-files-from-vm.md#selecting-the-right-machine-to-run-the-script) si puede ejecutar el script en la máquina virtual antes de continuar.

1. Conéctese a la máquina virtual mediante SSH. Reemplace *publicIpAddress* por la dirección IP pública de la máquina virtual, como se indica a continuación:

    ```bash
    ssh publicIpAddress
    ```

2. Para permitir que el script se ejecute correctamente, agregue permisos con **chmod**. Escriba el nombre de su propio script:

    ```bash
    chmod +x myVM_we_1571974050985163527.sh
    ```

3. Para montar el punto de recuperación, ejecute el script. Escriba el nombre de su propio script:

    ```bash
    ./myVM_we_1571974050985163527.sh
    ```

    Cuando se ejecute el script, se le pedirá que escriba una contraseña para acceder al punto de recuperación. Escriba la contraseña que se muestra en la salida del comando [az backup restore files mount-rp](/cli/azure/backup/restore/files?view=azure-cli-latest#az-backup-restore-files-mount-rp) anterior que generó el script de recuperación.

    La salida del script proporciona la ruta de acceso del punto de recuperación. La siguiente salida de ejemplo muestra que el punto de recuperación está montado en */home/azureuser/myVM-20170919213536/Volume1*:

    ```output
    Microsoft Azure VM Backup - File Recovery
    ______________________________________________
    Please enter the password as shown on the portal to securely connect to the recovery point. : c068a041ce12465

    Connecting to recovery point using ISCSI service...

    Connection succeeded!

    Please wait while we attach volumes of the recovery point to this machine...

    ************ Volumes of the recovery point and their mount paths on this machine ************

    Sr.No.  |  Disk  |  Volume  |  MountPath

    1)  | /dev/sdc  |  /dev/sdc1  |  /home/azureuser/myVM-20170919213536/Volume1

    ************ Open File Explorer to browse for files. ************
    ```

4. Use **cp** para volver a copiar la página web predeterminada de NGINX del punto de recuperación montado a la ubicación del archivo original. Reemplace el punto de montaje */home/azureuser/myVM-20170919213536/Volume1* por su propia ubicación:

    ```bash
    sudo cp /home/azureuser/myVM-20170919213536/Volume1/var/www/html/index.nginx-debian.html /var/www/html/
    ```

5. En el explorador web, actualice la página web. El sitio web se vuelve a cargar ahora correctamente, tal como se muestra en el ejemplo siguiente:

    ![el sitio web de NGINX se carga ahora correctamente](./media/tutorial-restore-files/nginx-restored.png)

6. Cierre la sesión de SSH de la máquina virtual de la manera siguiente:

    ```bash
    exit
    ```

7. Desmonte el punto de recuperación de su máquina virtual con [az backup restore files unmount-rp](/cli/azure/backup/restore/files?view=azure-cli-latest#az-backup-restore-files-unmount-rp). En el ejemplo siguiente se desmonta el punto de recuperación de la máquina virtual denominada *myVM* en *myRecoveryServicesVault*.

    Reemplace *myRecoveryPointName* por el nombre del punto de recuperación que obtuvo en los comandos anteriores:

    ```azurecli-interactive
    az backup restore files unmount-rp \
        --resource-group myResourceGroup \
        --vault-name myRecoveryServicesVault \
        --container-name myVM \
        --item-name myVM \
        --rp-name myRecoveryPointName
    ```

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, conectó un punto de recuperación a una máquina virtual y restauró los archivos de un servidor web. Ha aprendido a:

> [!div class="checklist"]
>
> * Enumerar y seleccionar puntos de recuperación
> * Conectar un punto de recuperación a una máquina virtual
> * Restaurar archivos desde un punto de recuperación

Avance al siguiente tutorial para obtener información acerca de cómo realizar copias de seguridad de Windows Server en Azure.

> [!div class="nextstepaction"]
> [Hacer copias de seguridad de Windows Server en Azure](tutorial-backup-windows-server-to-azure.md)
