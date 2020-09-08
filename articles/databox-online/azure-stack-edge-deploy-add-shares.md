---
title: 'Tutorial: Transferencia de datos a recursos compartidos con Azure Stack Edge | Microsoft Docs'
description: En este tutorial, aprenderá a agregar y conectarse a recursos compartidos en el dispositivo de Azure Stack Edge para que Azure Stack Edge pueda transferir los datos a Azure.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: tutorial
ms.date: 03/21/2019
ms.author: alkohli
Customer intent: As an IT admin, I need to understand how to add and connect to shares on Azure Stack Edge so I can use it to transfer data to Azure.
ms.openlocfilehash: fc47497eeceff20e8357195226c2d44cea64fa88
ms.sourcegitcommit: 4f1c7df04a03856a756856a75e033d90757bb635
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "87926117"
---
# <a name="tutorial-transfer-data-with-azure-stack-edge"></a>Tutorial: Transferencia de datos con Azure Stack Edge

En este tutorial se describe cómo agregar recursos compartidos a un dispositivo de Azure Stack Edge y conectarse a ellos. Después de agregar los recursos compartidos, Azure Stack Edge puede transferir datos a Azure.

Este procedimiento tarda aproximadamente 10 minutos en completarse.

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Agregar un recurso compartido
> * Conexión al recurso compartido

 
## <a name="prerequisites"></a>Prerrequisitos

Antes de agregar recursos compartidos a Azure Stack Edge, compruebe lo siguiente:

- Ha instalado el dispositivo físico tal como se describe en [Instalación de Azure Stack Edge](azure-stack-edge-deploy-install.md).

- Ha activado el dispositivo físico tal como se describe en [Conexión, configuración y activación de Azure Stack Edge](azure-stack-edge-deploy-connect-setup-activate.md).


## <a name="add-a-share"></a>Agregar un recurso compartido

Para crear un recurso compartido, realice el procedimiento siguiente:

1. En [Azure Portal](https://portal.azure.com/), seleccione el recurso de Azure Stack Edge y, después, vaya a **Información general**. El dispositivo debe estar en línea.

   ![Dispositivo en línea](./media/azure-stack-edge-deploy-add-shares/device-online-1.png)

2. Haga clic en **+ Agregar recurso compartido** en la barra de comandos del dispositivo.

   ![Agregar un recurso compartido](./media/azure-stack-edge-deploy-add-shares/select-add-share-1.png)

3. En el panel **Agregar recurso compartido**, lleve a cabo el procedimiento siguiente:

    a. En el cuadro **Nombre**, escriba un nombre único para el recurso compartido.  
    El nombre del recurso compartido solo puede incluir letras minúsculas, números y guiones. Debe tener entre 3 y 63 caracteres y empezar por una letra o un número. Antes y después de los guiones debe haber una letra o un número.
    
    b. Seleccione un **tipo** de recurso compartido.  
    El tipo puede ser **SMB** o **NFS** (SMB es el predeterminado). SMB es el estándar para los clientes de Windows y se usa NFS para los clientes de Linux.  
    Dependiendo de si elige recursos compartidos de SMB o NFS, el resto de las opciones varía ligeramente. 

    c. Proporcione una cuenta de almacenamiento donde residirá el recurso compartido. 

    > [!IMPORTANT]
    > Asegúrese de que la cuenta de Azure Storage que usa no tiene directivas de inmutabilidad establecidas si la usa con un dispositivo de Azure Stack Edge o Data Box Gateway. Para más información, consulte [Establecimiento y administración de directivas de inmutabilidad para el almacenamiento de blobs](https://docs.microsoft.com/azure/storage/blobs/storage-blob-immutability-policies-manage).
    
    d. En la lista desplegable **Servicio de almacenamiento**, seleccione **Blob en bloques**, **Blob en páginas** o **Archivos**.  
    El tipo de servicio que seleccione dependerá del formato que quiere que usen los datos en Azure. En este ejemplo, como queremos almacenar los datos como blobs en bloques en Azure, seleccionamos **Blob en bloques**. Si selecciona **Blob en páginas**, asegúrese de que los datos tienen una alineación de 512 bytes. Por ejemplo, un VHDX siempre tiene una alineación de 512 bytes.

    e. Cree un nuevo contenedor de blobs o use uno ya existente de la lista desplegable. Si crea un contenedor de blobs, proporcione un nombre para este. Si todavía no existe un contenedor, se crea en la cuenta de almacenamiento con el nombre del recurso compartido recién creado.
   
    f. Dependiendo de si creó un recurso compartido de SMB o NFS, haga uno de estos pasos: 
     
    - **Recurso compartido de SMB**: En **Usuario local con todos los privilegios**, seleccione **Crear nuevo** o **Usar existente**. Si crea un usuario local, escriba un nombre de usuario y una contraseña y, luego, confirme la contraseña. Con esta acción se asignan permisos al usuario local. Después de asignar aquí los permisos, puede usar el Explorador de archivos para modificarlos.

        Si activa la casilla de verificación **Permitir operaciones de solo lectura** en los datos de este recurso compartido, puede especificar usuarios de solo lectura.

        ![Incorporación de recurso compartido de SMB](./media/azure-stack-edge-deploy-add-shares/add-share-smb-1.png)
   
    - **Recurso compartido de NFS**: Escriba las direcciones IP de los clientes autorizados que pueden acceder al recurso compartido.

        ![Incorporación de un recurso compartido NFS](./media/azure-stack-edge-deploy-add-shares/add-share-nfs-1.png)
   
4. Seleccione **Crear** para crear el recurso compartido.
    
    Recibe una notificación de que la creación del recurso compartido está en curso. Una vez creado el recurso compartido con la configuración especificada, el icono **Recursos compartidos** se actualiza para reflejar el nuevo recurso compartido.
    

## <a name="connect-to-the-share"></a>Conexión al recurso compartido

Ahora, puede conectarse a uno o varios de los recursos compartidos que creó en el último paso. Dependiendo de si tiene un recurso compartido SMB o NFS, los pasos pueden variar.

### <a name="connect-to-an-smb-share"></a>Conexión a un recurso compartido SMB

En el cliente de Windows Server conectado al dispositivo de Azure Stack Edge, escriba los comandos para conectarse a un recurso compartido de SMB:


1. En una ventana de comandos, escriba:

    `net use \\<IP address of the device>\<share name>  /u:<user name for the share>`

2. Escriba la contraseña del recurso compartido cuando se le pida que lo haga.  
   A continuación se presenta un ejemplo de salida de este comando.

    ```powershell
    Microsoft Windows [Version 10.0.16299.192)
    (c) 2017 Microsoft Corporation. All rights reserved.
    
    C: \Users\DataBoxEdgeUser>net use \\10.10.10.60\newtestuser /u:Tota11yNewUser
    Enter the password for 'TotallyNewUser' to connect to '10.10.10.60':
    The command completed successfully.
    
    C: \Users\DataBoxEdgeUser>
    ```   


3. En el teclado, seleccione Windows + R.

4. En la ventana **Ejecutar**, especifique `\\<device IP address>` y seleccione **Aceptar**.  
   Se abre el Explorador de archivos. Ahora podrá ver los recursos compartidos que creó como carpetas. En el Explorador de archivos, haga doble clic en un recurso compartido (carpeta) para ver el contenido.
 
    ![Conexión a un recurso compartido SMB](./media/azure-stack-edge-deploy-add-shares/connect-to-share2.png)

    Los datos se escriben en estos recursos compartidos según se generan y el dispositivo inserta los datos en la nube.

### <a name="connect-to-an-nfs-share"></a>Conexión a un recurso compartido NFS

En el cliente Linux conectado al dispositivo de Azure Stack Edge, realice el procedimiento siguiente:

1. Asegúrese de que el cliente tiene instalado el cliente NFSv4. Para instalarlo, use el comando siguiente:

   `sudo apt-get install nfs-common`

    Para más información, vaya a [Instalación de cliente NFSv4](https://help.ubuntu.com/community/SettingUpNFSHowTo#NFSv4_client).

2. Una vez instalado el cliente NFS, monte el recurso compartido de NFS que creó en el dispositivo de Azure Stack Edge mediante el comando siguiente:

   `sudo mount -t nfs -o sec=sys,resvport <device IP>:/<NFS shares on device> /home/username/<Folder on local Linux computer>`

    > [!IMPORTANT]
    > El uso de la opción `sync` al montar recursos compartidos de archivos mejora la velocidad de transferencia de los archivos grandes.
    > Antes de montar los recursos compartidos, asegúrese de que los directorios que actuarán como puntos de montaje en la máquina local ya se han creado. Estos directorios no deben contener ningún archivo ni subcarpeta.

    En el ejemplo siguiente se muestra cómo conectarse mediante NFS a un recurso compartido de Azure Stack Edge. La dirección IP del dispositivo es `10.10.10.60`. El recurso compartido `mylinuxshare2` está montado en la máquina ubuntuVM. El punto de montaje del recurso compartido es `/home/databoxubuntuhost/edge`.

    `sudo mount -t nfs -o sec=sys,resvport 10.10.10.60:/mylinuxshare2 /home/databoxubuntuhost/Edge`

> [!NOTE] 
> Las siguientes advertencias son aplicables a esta versión:
> - Una vez que se crea un archivo en el recurso compartido, no se puede cambiar su nombre. 
> - La eliminación de un archivo de un recurso compartido no elimina la entrada en la cuenta de almacenamiento.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha adquirido conocimientos sobre los siguientes temas de Azure Stack Edge:

> [!div class="checklist"]
> * Agregar un recurso compartido
> * Conexión a un recurso compartido

Para aprender a transformar los datos mediante Azure Stack Edge, pase al siguiente tutorial:

> [!div class="nextstepaction"]
> [Transformación de datos con Azure Stack Edge](./azure-stack-edge-deploy-configure-compute.md)


