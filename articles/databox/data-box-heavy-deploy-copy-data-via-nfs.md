---
title: Tutorial de copia de datos en Azure Data Box Heavy mediante NFS| Microsoft Docs
description: En este tutorial, aprenderá a conectar y copiar datos del equipo host en Azure Data Box Heavy mediante NFS con la interfaz de usuario web local.
services: databox
author: alkohli
ms.service: databox
ms.subservice: heavy
ms.topic: tutorial
ms.date: 07/03/2019
ms.author: alkohli
ms.openlocfilehash: a8199039ea803d5919e168edccaaa03c18985725
ms.sourcegitcommit: 4f1c7df04a03856a756856a75e033d90757bb635
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "87921034"
---
# <a name="tutorial-copy-data-to-azure-data-box-heavy-via-nfs"></a>Tutorial: Copia de datos a Azure Data Box Heavy mediante NFS

En este tutorial se describe cómo conectarse al equipo host y copiar datos desde él mediante la interfaz de usuario web local en su dispositivo Azure Data Box Heavy.

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Requisitos previos
> * Conexión a un dispositivo Data Box Heavy
> * Copia de datos a Data Box Heavy

## <a name="prerequisites"></a>Requisitos previos

Antes de comenzar, asegúrese de que:

1. Ha completado el [Tutorial: Configuración de Azure Data Box Heavy](data-box-heavy-deploy-set-up.md).
2. Ha recibido su dispositivo Data Box Heavy y el estado del pedido en el portal se actualiza a **Delivered** (Entregado).
3. Tiene un equipo host con los datos que desea copiar en su dispositivo Data Box Heavy. El equipo host debe:
    - Ejecutar un [sistema operativo admitido](data-box-heavy-system-requirements.md).
    - Estar conectado a una red de alta velocidad. Para obtener velocidades de copia más rápidas, se pueden utilizar dos conexiones de 40 GbE (una por nodo) en paralelo. Si no dispone de una conexión de 40 GbE, le recomendamos que tenga al menos dos conexiones de 10 GbE (una por nodo). 

## <a name="connect-to-data-box-heavy"></a>Conexión a un dispositivo Data Box Heavy

En función de la cuenta de almacenamiento seleccionada, Data Box Heavy crea hasta:
- Tres recursos compartidos para cada cuenta de almacenamiento asociada (GPv1 y GPv2).
- Un recurso compartido para Premium Storage.
- Un recurso compartido para una cuenta de Blob Storage.

Estos recursos compartidos se crean en ambos nodos del dispositivo.

En los recursos compartidos de blobs en bloques y en páginas:
- Las entidades de primer nivel son los contenedores.
- Las entidades de segundo nivel son los blobs.

En los recursos compartidos de Azure Files:
- Las entidades de primer nivel son los recursos compartidos.
- Las entidades de segundo nivel son los archivos.

En la tabla siguiente se muestra la ruta de acceso UNC a los recursos compartidos en la dirección URL de la ruta de acceso de Data Box Heavy y Azure Storage donde se cargan los datos. La dirección URL final de la ruta de acceso de Azure Storage se puede derivar a partir de la ruta de acceso UNC al recurso compartido.
 
| Storage           | Ruta de acceso UNC                                                                       |
|-------------------|--------------------------------------------------------------------------------|
| Blobs en bloques de Azure | <li>Ruta de acceso UNC a recursos compartidos: `//<DeviceIPAddress>/<StorageAccountName_BlockBlob>/<ContainerName>/files/a.txt`</li><li>Dirección URL de Azure Storage: `https://<StorageAccountName>.blob.core.windows.net/<ContainerName>/files/a.txt`</li> |  
| Blobs en páginas de Azure  | <li>Ruta de acceso UNC a recursos compartidos: `//<DeviceIPAddres>/<StorageAccountName_PageBlob>/<ContainerName>/files/a.txt`</li><li>Dirección URL de Azure Storage: `https://<StorageAccountName>.blob.core.windows.net/<ContainerName>/files/a.txt`</li>   |  
| Azure Files       |<li>Ruta de acceso UNC a recursos compartidos: `//<DeviceIPAddres>/<StorageAccountName_AzFile>/<ShareName>/files/a.txt`</li><li>Dirección URL de Azure Storage: `https://<StorageAccountName>.file.core.windows.net/<ShareName>/files/a.txt`</li>        |

Si usa un equipo host Linux, realice los pasos siguientes para configurar un dispositivo para que pueda acceder a los clientes NFS.

1. Proporcione las direcciones IP de los clientes autorizados que pueden acceder al recurso compartido. En la interfaz de usuario web local, vaya a la página **Connect and copy** (Conectar y copiar). En **NFS settings** (Configuración de NFS), haga clic en **NFS client access** (Acceso de cliente NFS). 

    ![Configuración del acceso de cliente NFS 1](media/data-box-deploy-copy-data/nfs-client-access.png)

2. Proporcione la dirección IP del cliente NFS y haga clic en **Add** (Agregar). Para configurar el acceso para varios clientes NFS, repita este paso. Haga clic en **OK**.

    ![Configuración del acceso de cliente NFS 2](media/data-box-deploy-copy-data/nfs-client-access2.png)

2. Asegúrese de que el equipo host de Linux tiene instalada una [versión admitida](data-box-system-requirements.md) del cliente NFS. Use la versión específica para su distribución de Linux. 

3. Una vez instalado el cliente NFS, use el siguiente comando para montar el recurso compartido NFS en el dispositivo Data Box:

    `sudo mount <Data Box Heavy device IP>:/<NFS share on Data Box Heavy device> <Path to the folder on local Linux computer>`

    En el ejemplo siguiente se muestra cómo conectarse mediante NFS a un recurso compartido de Data Box Heavy. La dirección IP del dispositivo Data Box Heavy es `10.161.23.130`; el recurso compartido `Mystoracct_Blob` se monta en la máquina virtual ubuntuVM y el punto de montaje es `/home/databoxheavyubuntuhost/databoxheavy`.

    `sudo mount -t nfs 10.161.23.130:/Mystoracct_Blob /home/databoxheavyubuntuhost/databoxheavy`
    
    Para los clientes de Mac, debe agregar una opción adicional como sigue: 
    
    `sudo mount -t nfs -o sec=sys,resvport 10.161.23.130:/Mystoracct_Blob /home/databoxheavyubuntuhost/databoxheavy`

    **Cree siempre una carpeta para los archivos que se va a copiar en el recurso compartido y, después, copie los archivos en ella**. La carpeta que se creó en los recursos compartidos de blob en bloques y blob en páginas representa un contenedor en el que los datos se cargan como blobs. No se pueden copiar los archivos directamente en la carpeta *root* de la cuenta de almacenamiento.

## <a name="copy-data-to-data-box-heavy"></a>Copia de datos a Data Box Heavy

Una vez que esté conectado a los recursos compartidos de Data Box Heavy, el siguiente paso es copiar los datos. Antes de comenzar la copia de datos, revise las consideraciones siguientes:

- Es responsabilidad suya asegurarse de que copia los datos en los recursos compartidos que se corresponden con el formato de datos adecuado. Por ejemplo, copie los datos de blobs en bloques en la carpeta para blobs en bloques. Copia de discos duros virtuales en blobs en páginas. Si el formato de los datos no coincide con el recurso compartido correspondiente, la carga de datos en Azure producirá un error más adelante.
-  Al copiar los datos, asegúrese de que el tamaño de los datos se ajusta a los [límites de Azure Storage y Data Box Heavy](data-box-heavy-limits.md). 
- Si los datos que va a cargar el dispositivo Data Box Heavy los están cargando a la vez otras aplicaciones fuera del dispositivo Data Box Heavy, podría provocar errores en el trabajo de carga y daños en los datos.
- Se recomienda no usar SMB y NFS simultáneamente ni copiar los mismos datos al mismo destino final en Azure. En estos casos, no se puede determinar el resultado final.
- **Cree siempre una carpeta para los archivos que se va a copiar en el recurso compartido y, después, copie los archivos en ella**. La carpeta que se creó en los recursos compartidos de blob en bloques y blob en páginas representa un contenedor en el que los datos se cargan como blobs. No se pueden copiar los archivos directamente en la carpeta *root* de la cuenta de almacenamiento.
- Si se ingieren nombres de archivos y directorios que distinguen entre mayúsculas y minúsculas de un recurso compartido de NFS en NFS en Data Box Heavy: 
    - Se conservan las mayúsculas y minúsculas en el nombre.
    - Los archivos no distinguen mayúsculas de minúsculas.
    
    Por ejemplo, si copia `SampleFile.txt` y `Samplefile.Txt`, se conservarán las mayúsculas y minúsculas del nombre cuando se copian en el dispositivo, pero el segundo archivo sobrescribirá el primero porque considera que son el mismo archivo.


Si su equipo es un host Linux, use una utilidad de copia similar a Robocopy. Algunas de las alternativas disponibles en Linux son [rsync](https://rsync.samba.org/), [FreeFileSync](https://www.freefilesync.org/), [Unison](https://www.cis.upenn.edu/~bcpierce/unison/) o [Ultracopier](https://ultracopier.first-world.info/).  

El comando `cp` es una de las mejores opciones para copiar un directorio. Para más información sobre cómo usarlo, consulte las [páginas sobre cp](http://man7.org/linux/man-pages/man1/cp.1.html).

Si usa la opción rsync para una copia multiproceso, siga estas directrices:

 - Instale el paquete **CIFS Utils** o **NFS Utils** según el sistema de archivos que use el cliente Linux.

    `sudo apt-get install cifs-utils`

    `sudo apt-get install nfs-utils`

 -  Instale **Rsync** y **Parallel** (varía según la versión de la distribución de Linux).

    `sudo apt-get install rsync`
   
    `sudo apt-get install parallel` 

 - Cree un punto de montaje.

    `sudo mkdir /mnt/databoxheavy`

 - Monte el volumen.

    `sudo mount -t NFS4  //Databox-heavy-IP-Address/share_name /mnt/databoxheavy` 

 - Refleje la estructura de directorios de carpetas.  

    `rsync -za --include='*/' --exclude='*' /local_path/ /mnt/databoxheavy`

 - Copie los archivos. 

    `cd /local_path/; find -L . -type f | parallel -j X rsync -za {} /mnt/databoxheavy/{}`

     donde j especifica el número de la paralelización, X = número de copias en paralelo

     Se recomienda empezar con 16 copias en paralelo y aumentar el número de subprocesos según los recursos disponibles.

> [!IMPORTANT]
> No se admiten los siguientes tipos de archivos de Linux: vínculos simbólicos, archivos de caracteres, archivos de bloqueo, sockets y canalizaciones. Estos tipos de archivo generarán errores durante el paso **Preparación para el envío**.

Abra la carpeta de destino para ver y comprobar los archivos copiados. Si se produce algún error durante el proceso de copia, descargue los archivos de error para solucionar problemas. Para más información, consulte [Ver registro de errores durante la copia de datos en Data Box Heavy](data-box-logs.md#view-error-log-during-data-copy). Para obtener una lista detallada de errores durante la copia de datos, consulte [Solución de problemas de Data Box Heavy](data-box-troubleshoot.md).

Para garantizar la integridad de los datos, la suma de comprobación se calcula a medida que los datos se copian. Una vez completada la copia, compruebe el espacio utilizado y el espacio disponible en el dispositivo.
    
   ![Comprobación del espacio libre y utilizado en el panel](media/data-box-deploy-copy-data/verify-used-space-dashboard.png)


## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha obtenido información acerca de varios temas relacionados con Azure Data Box Heavy, como:

> [!div class="checklist"]
> * Requisitos previos
> * Conexión a un dispositivo Data Box Heavy
> * Copia de datos a Data Box Heavy


En el siguiente tutorial aprenderá a enviar su dispositivo Data Box a Microsoft.

> [!div class="nextstepaction"]
> [Envío del dispositivo Azure Data Box Heavy a Microsoft](./data-box-heavy-deploy-picked-up.md)

