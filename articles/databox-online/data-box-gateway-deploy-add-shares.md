---
title: Transferencia de datos con Azure Data Box Gateway | Microsoft Docs
description: Obtenga información sobre cómo agregar y conectarse a recursos compartidos en Azure Data Box Gateway y, a continuación, el dispositivo Data Box Gateway puede transferir los datos a Azure.
services: databox
author: alkohli
ms.service: databox
ms.subservice: gateway
ms.topic: tutorial
ms.date: 03/08/2019
ms.author: alkohli
ms.openlocfilehash: 4e54f5254d38bca9c8ff30d5b7cefbe3fde69576
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90904367"
---
# <a name="tutorial-transfer-data-with-azure-data-box-gateway"></a>Tutorial: Transferencia de datos con Azure Data Box Gateway


## <a name="introduction"></a>Introducción

En este artículo se describe cómo agregar recursos compartidos a Data Box Gateway y cómo conectarse a ellos. Después de agregar los recursos compartidos, el dispositivo de Data Box Edge puede transferir datos a Azure.

Este procedimiento tarda aproximadamente 10 minutos en completarse.

En este tutorial, aprenderá a:

> [!div class="checklist"]
>
> * Agregar un recurso compartido
> * Conexión a un recurso compartido

## <a name="prerequisites"></a>Prerrequisitos

Antes de agregar recursos compartidos a Data Box Gateway, asegúrese de que:

- Ha aprovisionado un dispositivo virtual y se ha conectado a él según se detalla en [Aprovisionamiento de Data Box Gateway en Hyper-V](data-box-gateway-deploy-provision-hyperv.md) o en [Aprovisionamiento de Data Box Gateway en VMware](data-box-gateway-deploy-provision-vmware.md).

- Ha activado el dispositivo virtual que se describe en [Conexión y activación de Azure Data Box Gateway](data-box-gateway-deploy-connect-setup-activate.md).

- El dispositivo está listo para crear recursos compartidos y transferir datos.

## <a name="add-a-share"></a>Agregar un recurso compartido

Para crear un recurso compartido, realice el procedimiento siguiente:

1. En [Azure Portal](https://portal.azure.com/), seleccione el recurso de Data Box Gateway y, después, vaya a **Información general**. El dispositivo debe estar en línea. Haga clic en **+ Agregar recurso compartido** en la barra de comandos del dispositivo.
   
   ![Agregar un recurso compartido](./media/data-box-gateway-deploy-add-shares/click-add-share.png)

4. En **Agregar recurso compartido**, realice el siguiente procedimiento:

    1. Proporcione un nombre exclusivo para el recurso compartido. Los nombres de recursos compartidos solo pueden contener letras minúsculas, números y guiones. El nombre del recurso compartido debe tener entre 3 y 63 caracteres y empezar por una letra o un número. Antes y después de cada guion debe ir un carácter que no sea otro guión.
    
    2. Seleccione un **tipo** de recurso compartido. El tipo puede ser SMB o NFS, siendo SMB el valor predeterminado. SMB es el estándar para los clientes de Windows y se usa NFS para los clientes de Linux. Dependiendo de si elige recursos compartidos SMB o NFS, las opciones que se presentan son ligeramente diferentes.

    3. Proporcione una cuenta de almacenamiento donde residirá el recurso compartido. Si todavía no existe un contenedor, se crea en la cuenta de almacenamiento con el nombre del recurso compartido recién creado. Si el contenedor ya existe, se utilizará.
       > [!IMPORTANT]
       > Asegúrese de que la cuenta de Azure Storage que usa no tiene directivas de inmutabilidad establecidas si la usa con un dispositivo de Azure Stack Edge Pro o Data Box Gateway. Para más información, consulte [Establecimiento y administración de directivas de inmutabilidad para el almacenamiento de blobs](https://docs.microsoft.com/azure/storage/blobs/storage-blob-immutability-policies-manage).
    
    4. Elija el **servicio de almacenamiento** entre blob en bloques, blobs en páginas o archivos. El tipo de servicio elegido depende de en qué formato desea que los datos residan en Azure. Por ejemplo, en este caso, queremos que los datos residan como blobs en bloque en Azure y, por tanto, seleccionamos esa opción. Si elige blob en páginas, debe asegurarse de que los datos tienen 512 bytes alineados. Por ejemplo, un VHDX siempre tiene una alineación de 512 bytes.
   
    5. Este paso depende de si está creando un recurso compartido SMB o NFS.
     
    - **Recurso compartido SMB**: en **Usuario local con todos los privilegios**, seleccione **Crear nuevo** o **Usar existente**. Si crea un usuario local, escriba un **nombre de usuario** y una **contraseña** y, luego, **confirme la contraseña**. Con esta acción se asignan los permisos al usuario local. Actualmente no se admite la modificación de permisos de nivel de recurso compartido.
    
        ![Incorporación de recurso compartido de SMB](./media/data-box-gateway-deploy-add-shares/add-share-smb-1.png)
        
        Si activa la casilla **Permitir operaciones de solo lectura** para los datos de este recurso compartido, puede especificar usuarios de solo lectura.
        
    - **Recurso compartido NFS**: proporcione las direcciones IP de los clientes que pueden acceder al recurso compartido.

        ![Incorporación de un recurso compartido NFS](./media/data-box-gateway-deploy-add-shares/add-share-nfs-1.png)
   
9. Seleccione **Crear** para crear el recurso compartido.
    
    Recibe una notificación de que la creación del recurso compartido está en curso. Una vez creado el recurso compartido con la configuración especificada, el icono **Recursos compartidos** se actualiza para reflejar el nuevo recurso compartido.
    
    ![Icono de recursos compartidos actualizados](./media/data-box-gateway-deploy-add-shares/updated-list-of-shares.png) 

## <a name="connect-to-the-share"></a>Conexión al recurso compartido

Ahora, puede conectarse a uno o varios de los recursos compartidos que creó en el último paso. Dependiendo de si tiene un recurso compartido SMB o NFS, los pasos pueden variar.

### <a name="connect-to-an-smb-share"></a>Conexión a un recurso compartido SMB

En el cliente de Windows Server conectado a Data Box Gateway, escriba los siguientes comandos para conectarse a un recurso compartido SMB:


1. En una ventana de comandos, escriba:

    `net use \\<IP address of the device>\<share name>  /u:<user name for the share>`

    Cuando se le solicite, escriba la contraseña del recurso compartido. A continuación se presenta un ejemplo de salida de este comando.

    ```powershell
    Microsoft Windows [Version 18.8.16299.192) 
    (c) 2017 microsoft Corporation. All rights reserved . 
    
    C: \Users\GatewayUser>net use \\10.10.10.60\newtestuser /u:Tota11yNewUser 
    Enter the password for 'TotallyNewUser' to connect to '10.10.10.60'  
    The command completed successfully. 
    
    C: \Users\GatewayUser>
    ```   


2. En el teclado, seleccione Windows + R. 
3. En la ventana **Ejecutar**, especifique `\\<device IP address>` y seleccione **Aceptar**. Se abre el Explorador de archivos. Ahora podrá ver los recursos compartidos que creó como carpetas. En el Explorador de archivos, haga doble clic en un recurso compartido (carpeta) para ver el contenido.
 
    ![Conexión a un recurso compartido SMB](./media/data-box-gateway-deploy-add-shares/connect-to-share2.png)-->

    Los datos se escriben en estos recursos compartidos según se generan y el dispositivo inserta los datos en la nube.

### <a name="connect-to-an-nfs-share"></a>Conexión a un recurso compartido NFS

En el cliente Linux conectado al dispositivo de Azure Stack Edge Pro, realice el procedimiento siguiente:

1. Asegúrese de que el cliente tiene instalado el cliente NFSv4. Para instalarlo, use el comando siguiente:

   `sudo apt-get install nfs-common`

    Para más información, vaya a [Instalación de cliente NFSv4](https://help.ubuntu.com/community/SettingUpNFSHowTo#NFSv4_client).

2. Una vez instalado el cliente NFS, use el siguiente comando para montar el recurso compartido NFS que creó en el dispositivo Data Box Gateway:

   `sudo mount -t nfs -o sec=sys,resvport <device IP>:/<NFS shares on device> /home/username/<Folder on local Linux computer>`

    Antes de configurar los montajes, asegúrese de que los directorios que actuarán como puntos de montaje en la máquina local ya se han creado y no contienen archivos ni subcarpetas.

    En el ejemplo siguiente se muestra cómo conectarse mediante NFS a un recurso compartido de Data Box Gateway. La dirección IP del dispositivo virtual es `10.10.10.60`; el recurso compartido `mylinuxshare2` se monta en la máquina virtual ubuntuVM y el punto de montaje es `/home/databoxubuntuhost/gateway`.

    `sudo mount -t nfs -o sec=sys,resvport 10.10.10.60:/mylinuxshare2 /home/databoxubuntuhost/gateway`

> [!NOTE] 
> Las siguientes advertencias son aplicables a esta versión:
> - Una vez que se crea un archivo en los recursos compartidos, no se puede cambiar su nombre.
> - La eliminación de un archivo de un recurso compartido no elimina la entrada en la cuenta de almacenamiento.
> - Si usa `rsync` para copiar los datos, no se admite la opción `rsync -a`.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha obtenido información acerca de varios temas relacionados con Data Box Gateway, como:

> [!div class="checklist"]
> * Agregar un recurso compartido
> * Conexión a un recurso compartido


Pase al siguiente tutorial para aprender a administrar Data Box Gateway.

> [!div class="nextstepaction"]
> [Uso de la interfaz de usuario web local para administrar una instancia de Data Box Gateway](https://aka.ms/dbg-docs)


