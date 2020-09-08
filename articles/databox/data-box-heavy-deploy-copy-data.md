---
title: Tutorial para copiar datos en Azure Data Box Heavy mediante SMB | Microsoft Docs
description: En este tutorial, aprenderá a conectar y copiar datos del equipo host en Azure Data Box Heavy mediante SMB con la interfaz de usuario web local.
services: databox
author: alkohli
ms.service: databox
ms.subservice: heavy
ms.topic: tutorial
ms.date: 08/29/2019
ms.author: alkohli
ms.localizationpriority: high
ms.openlocfilehash: 6c88a1eaee50676cfe6bb9589779d7156b805e30
ms.sourcegitcommit: 4f1c7df04a03856a756856a75e033d90757bb635
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "87920983"
---
::: zone target = "docs"

# <a name="tutorial-copy-data-to-azure-data-box-heavy-via-smb"></a>Tutorial: Copia de datos a Azure Data Box Heavy mediante SMB

::: zone-end

::: zone target = "chromeless"

## <a name="copy-data-to-azure-data-box-heavy"></a>Copia de datos en Azure Data Box Heavy

::: zone-end

::: zone target = "docs"

En este tutorial se describe cómo conectarse al equipo host y copiar datos desde él mediante la interfaz de usuario web local.

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Conexión a un dispositivo Data Box Heavy
> * Copia de datos a Data Box Heavy

::: zone-end

::: zone target = "chromeless"

Puede copiar datos en su instancia de Data Box desde el servidor de origen a través de SMB, NFS, REST, el servicio de copia de datos o en discos administrados.

En cada caso, asegúrese de que los nombres de los recursos compartidos y de las carpetas, así como el tamaño de los datos, sigan las directrices descritas en los [límites de los servicios Azure Storage y Data Box Heavy](data-box-heavy-limits.md).

::: zone-end

::: zone target = "docs"

## <a name="prerequisites"></a>Requisitos previos

Antes de comenzar, asegúrese de que:

1. Ha completado el [Tutorial: Configuración de Azure Data Box Heavy](data-box-deploy-set-up.md).
2. Ha recibido su dispositivo Data Box Heavy y el estado del pedido en el portal se actualiza a **Delivered** (Entregado).
3. Tiene un equipo host con los datos que desea copiar en su dispositivo Data Box Heavy. El equipo host debe:
    - Ejecutar un [sistema operativo admitido](data-box-system-requirements.md).
    - Estar conectado a una red de alta velocidad. Para obtener velocidades de copia más rápidas, se pueden utilizar dos conexiones de 40 GbE (una por nodo) en paralelo. Si no dispone de una conexión de 40 GbE, le recomendamos que tenga al menos dos conexiones de 10 GbE (una por nodo).
   

## <a name="connect-to-data-box-heavy-shares"></a>Conexión a recursos compartidos de Data Box Heavy

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
| Blobs en bloques de Azure | <li>Ruta de acceso UNC a recursos compartidos: `\\<DeviceIPAddress>\<StorageAccountName_BlockBlob>\<ContainerName>\files\a.txt`</li><li>Dirección URL de Azure Storage: `https://<StorageAccountName>.blob.core.windows.net/<ContainerName>/files/a.txt`</li> |  
| Blobs en páginas de Azure  | <li>Ruta de acceso UNC a recursos compartidos: `\\<DeviceIPAddres>\<StorageAccountName_PageBlob>\<ContainerName>\files\a.txt`</li><li>Dirección URL de Azure Storage: `https://<StorageAccountName>.blob.core.windows.net/<ContainerName>/files/a.txt`</li>   |  
| Azure Files       |<li>Ruta de acceso UNC a recursos compartidos: `\\<DeviceIPAddres>\<StorageAccountName_AzFile>\<ShareName>\files\a.txt`</li><li>Dirección URL de Azure Storage: `https://<StorageAccountName>.file.core.windows.net/<ShareName>/files/a.txt`</li>        |      

Los pasos para realizar la conexión mediante un cliente Windows o Linux son diferentes.

> [!NOTE]
> Siga los mismos pasos para conectarse a ambos nodos del dispositivo en paralelo.

### <a name="connect-on-a-windows-system"></a>Conexión en un sistema Windows

Si usa un equipo host Windows Server, realice los pasos siguientes para conectarse a su dispositivo Data Box Heavy.

1. El primer paso es autenticarse e iniciar sesión. Vaya a **Connect and copy** (Conectar y copiar). Haga clic en **Get credentials** (Obtener credenciales) para obtener las credenciales de acceso para los recursos compartidos asociados con la cuenta de almacenamiento.

    ![Obtención de las credenciales del recurso compartido 1](media/data-box-heavy-deploy-copy-data/get-share-credentials-1.png)

2. En el cuadro de diálogo Access share and copy data (Acceder al recurso compartido y copiar datos), copie los valores de **Username** (Nombre de usuario) y **Password** (Contraseña) del recurso compartido. Haga clic en **OK**.
    
    ![Obtención de las credenciales del recurso compartido 1](media/data-box-heavy-deploy-copy-data/get-share-credentials-2.png)

3. Para acceder a los recursos compartidos asociados con la cuenta de almacenamiento (*databoxe2etest* en el ejemplo siguiente) desde el equipo host, abra una ventana Comandos. En el símbolo del sistema, escriba:

    `net use \\<IP address of the device>\<share name>  /u:<user name for the share>`

    Dependiendo del formato de los datos, las rutas de acceso de los recursos compartidos son las siguientes:
    - Blob en bloques de Azure: `\\10.100.10.100\databoxe2etest_BlockBlob`
    - Blob en páginas de Azure: `\\10.100.10.100\databoxe2etest_PageBlob`
    - Azure Files: `\\10.100.10.100\databoxe2etest_AzFile`
    
4. Cuando se le solicite, escriba la contraseña del recurso compartido. En el ejemplo siguiente se muestra la conexión a un recurso compartido con el comando anterior.

    ```
    C:\Users\Databoxuser>net use \\10.100.10.100\databoxe2etest_BlockBlob /u:databoxe2etest
    Enter the password for 'databoxe2etest' to connect to '10.100.10.100':
    The command completed successfully.
    ```

4. Presione Windows + R. En la ventana **Ejecutar**, escriba `\\<device IP address>`. Haga clic en **Aceptar** para abrir el Explorador de archivos.
    
    ![Conexión al recurso compartido mediante el Explorador de archivos 2](media/data-box-heavy-deploy-copy-data/connect-shares-file-explorer-1.png)

    Ahora debería ver los recursos compartidos como carpetas.
    
    ![Conexión al recurso compartido mediante el Explorador de archivos 2](media/data-box-heavy-deploy-copy-data/connect-shares-file-explorer-2.png)

    **Cree siempre una carpeta para los archivos que se va a copiar en el recurso compartido y, después, copie los archivos en ella**. La carpeta que se creó en los recursos compartidos de blob en bloques y blob en páginas representa un contenedor en el que los datos se cargan como blobs. No se pueden copiar los archivos directamente en la carpeta *root* de la cuenta de almacenamiento.
    
### <a name="connect-on-a-linux-system"></a>Conexión en un sistema Linux

Si usa un cliente Linux, utilice el siguiente comando para montar el recurso compartido SMB.

```
sudo mount -t nfs -o vers=2.1 10.126.76.172:/databoxe2etest_BlockBlob /home/databoxubuntuhost/databox
```

El parámetro `vers` es la versión de SMB que admite el host Linux. Conecte la versión adecuada en el comando anterior.

Para ver las versiones de SMB compatibles con Data Box Heavy, consulte [Sistemas de archivos compatibles para clientes Linux](data-box-heavy-system-requirements.md#supported-file-systems-for-linux-clients).

## <a name="copy-data-to-data-box-heavy"></a>Copia de datos a Data Box Heavy

Una vez que esté conectado a los recursos compartidos de Data Box Heavy, el siguiente paso es copiar los datos.

### <a name="copy-considerations"></a>Consideraciones sobre la copia

Antes de comenzar la copia de datos, revise las consideraciones siguientes:

- Asegúrese de que copia los datos en los recursos compartidos que se corresponden con el formato de datos adecuado. Por ejemplo, copie los datos de blobs en bloques en la carpeta para blobs en bloques. Copie los discos duros virtuales en blobs en páginas.

    Si el formato de los datos no coincide con el recurso compartido correspondiente, la carga de datos en Azure producirá un error más adelante.
-  Al copiar los datos, asegúrese de que el tamaño de los datos se ajusta a los límites descritos en los [límites de Azure Storage y de Data Box Heavy](data-box-heavy-limits.md).
- Si los datos que va a cargar el dispositivo Data Box Heavy los están cargando a la vez otras aplicaciones fuera del dispositivo Data Box Heavy, podría provocar errores en el trabajo de carga y daños en los datos.
- Es recomendable que:
    - No use SMB y NFS al mismo tiempo.
    - Copie los mismos datos en el mismo destino final en Azure.
     
  En estos casos, no se puede determinar el resultado final.
- Cree siempre una carpeta para los archivos que se van a copiar en el recurso compartido y, después, copie los archivos en ella. La carpeta que se creó en los recursos compartidos de blob en bloques y blob en páginas representa un contenedor en el que los datos se cargan como blobs. No se pueden copiar los archivos directamente en la carpeta *root* de la cuenta de almacenamiento.

Después de haberse conectado al recurso compartido SMB, inicie una copia de datos.

1. Puede usar cualquier herramienta de copia de archivos compatible con SMB, como Robocopy, para copiar los datos. Con Robocopy se pueden iniciar varios trabajos de copia. Use el comando siguiente:
    
    ```
    robocopy <Source> <Target> * /e /r:3 /w:60 /is /nfl /ndl /np /MT:32 or 64 /fft /Log+:<LogFile>
    ```
    Los atributos se describen en la tabla siguiente.
    
    |Atributo  |Descripción  |
    |---------|---------|
    |/e      |Copia los subdirectorios incluyendo los directorios vacíos.         |
    |/r:     |Especifica el número de reintentos en las copias con errores.         |
    |/w:     |Especifica el tiempo de espera entre reintentos, en segundos.         |
    |/is     |Incluye los mismos archivos.         |
    |/nfl    |Especifica que los nombres de archivo no se han registrado.         |
    |/ndl    |Especifica que los nombres de directorio no se han registrado.        |
    |/np     |Especifica que no se mostrará el progreso de la operación de copia (el número de archivos o directorios copiados hasta el momento). Mostrar el progreso reduce significativamente el rendimiento.         |
    |/MT     | Especifica que se utilice subprocesamiento múltiple; se recomiendan 64 o 32 subprocesos. Esta opción que no se utiliza con los archivos cifrados. Es posible que deba separar archivos cifrados y sin cifrar. Sin embargo, copiar con un solo subproceso disminuye de forma significativa el rendimiento.           |
    |/fft    | Utilice esta opción para reducir la granularidad de la marca de tiempo para cualquier sistema de archivos.        |
    |/b      | Copia los archivos en modo de copia de seguridad.        |
    |/z      | Copia los archivos en modo de reinicio; use esta opción si el entorno es inestable. Esta opción reduce el rendimiento porque realiza un registro adicional.      |
    | /zb    | Usa el modo de reinicio. Si se deniega el acceso, esta opción utiliza el modo de copia de seguridad. Esta opción reduce el rendimiento porque utiliza puntos de control.         |
    |/efsraw | Copia todos los archivos cifrados en el modo sin procesar de EFS. Usar solo con los archivos cifrados.         |
    |log+:\<LogFile>| Anexa la salida al archivo de registro existente.|
    
 
    En el ejemplo siguiente se muestra la salida del comando robocopy para copiar archivos en Data Box Heavy.

    ```   
    C:\Users>Robocopy C:\Git\azure-docs-pr\contributor-guide \\10.100.10.100\devicemanagertest1_AzFile\templates /MT:24
    -------------------------------------------------------------------------------
        ROBOCOPY     ::     Robust File Copy for Windows
    -------------------------------------------------------------------------------
        Started : Thursday, April 4, 2019 2:34:58 PM
        Source : C:\Git\azure-docs-pr\contributor-guide\
        Dest : \\10.100.10.100\devicemanagertest1_AzFile\templates\
        Files : *.*
        Options : *.* /DCOPY:DA /COPY:DAT /MT:24 /R:5 /W:60
    ------------------------------------------------------------------------------
    
    100%        New File                 206        C:\Git\azure-docs-pr\contributor-guide\article-metadata.md
    100%        New File                 209        C:\Git\azure-docs-pr\contributor-guide\content-channel-guidance.md
    100%        New File                 732        C:\Git\azure-docs-pr\contributor-guide\contributor-guide-index.md
    100%        New File                 199        C:\Git\azure-docs-pr\contributor-guide\contributor-guide-pr-criteria.md
                New File                 178        C:\Git\azure-docs-pr\contributor-guide\contributor-guide-pull-request-co100%  .md
                New File                 250        C:\Git\azure-docs-pr\contributor-guide\contributor-guide-pull-request-et100%  e.md
    100%        New File                 174        C:\Git\azure-docs-pr\contributor-guide\create-images-markdown.md
    100%        New File                 197        C:\Git\azure-docs-pr\contributor-guide\create-links-markdown.md
    100%        New File                 184        C:\Git\azure-docs-pr\contributor-guide\create-tables-markdown.md
    100%        New File                 208        C:\Git\azure-docs-pr\contributor-guide\custom-markdown-extensions.md
    100%        New File                 210        C:\Git\azure-docs-pr\contributor-guide\file-names-and-locations.md
    100%        New File                 234        C:\Git\azure-docs-pr\contributor-guide\git-commands-for-master.md
    100%        New File                 186        C:\Git\azure-docs-pr\contributor-guide\release-branches.md
    100%        New File                 240        C:\Git\azure-docs-pr\contributor-guide\retire-or-rename-an-article.md
    100%        New File                 215        C:\Git\azure-docs-pr\contributor-guide\style-and-voice.md
    100%        New File                 212        C:\Git\azure-docs-pr\contributor-guide\syntax-highlighting-markdown.md
    100%        New File                 207        C:\Git\azure-docs-pr\contributor-guide\tools-and-setup.md
    ------------------------------------------------------------------------------
    
                    Total    Copied   Skipped  Mismatch    FAILED    Extras
        Dirs :         1         1         1         0         0         0
        Files :        17        17         0         0         0         0
        Bytes :     3.9 k     3.9 k         0         0         0         0          
    C:\Users>
    ```       

2. Para optimizar el rendimiento, use los siguientes parámetros de robocopy al copiar los datos. (Los números siguientes representan el escenario más optimista).

    | Plataforma    | Archivos pequeños principalmente < 512 KB    | Archivos medianos principalmente, de 512 KB a 1 MB  | Archivos grandes principalmente > 1 MB                             |
    |-------------|--------------------------------|----------------------------|----------------------------|
    | Data Box Heavy | 6 sesiones de Robocopy <br> 24 subprocesos por sesión | 6 sesiones de Robocopy <br> 16 subprocesos por sesión | 6 sesiones de Robocopy <br> 16 subprocesos por sesión |


    Para más información sobre el comando Robocopy, consulte [Robocopy and a few examples](https://social.technet.microsoft.com/wiki/contents/articles/1073.robocopy-and-a-few-examples.aspx) (Robocopy y algunos ejemplos).

3. Abra la carpeta de destino para ver y comprobar los archivos copiados.

    ![Visualización de los archivos copiados](media/data-box-heavy-deploy-copy-data/view-copied-files-1.png)


4. Cuando se copian los datos:

    - Los nombres, tamaños y formatos de archivos se validan para garantizar que cumplen con los límites de almacenamiento y objetos de Azure, así como con las convenciones de nombres de archivo y contenedores de Azure.
    - Para garantizar la integridad de datos, la suma de comprobación se calcula a medida que los datos se copian.

    Si se produce algún error durante el proceso de copia, descargue los archivos de error para solucionar problemas. Seleccione el icono de flecha para descargar los archivos de error.

    ![Descarga de los archivos de error](media/data-box-heavy-deploy-copy-data/download-error-files.png)

    Para más información, consulte [Ver registro de errores durante la copia de datos en Data Box Heavy](data-box-logs.md#view-error-log-during-data-copy). Para obtener una lista detallada de errores durante la copia de datos, consulte [Solución de problemas de Data Box Heavy](data-box-troubleshoot.md).

5. Abra el archivo de error en el Bloc de notas. El archivo de error siguiente indica que los datos no se han alineado correctamente.

    ![Archivo de error abierto](media/data-box-heavy-deploy-copy-data/open-error-file.png)
    
    Para un blob en páginas, los datos deben estar alineados en 512 bytes. Después de quitar estos datos, el error se resuelve tal como se muestra en la captura de pantalla siguiente.

    ![Error resuelto](media/data-box-heavy-deploy-copy-data/error-resolved.png)

6. Una vez completada la copia, vaya a la página **View Dashboard** (Ver panel). Compruebe el espacio utilizado y el espacio disponible en el dispositivo.
    
    ![Comprobación del espacio libre y utilizado en el panel](media/data-box-heavy-deploy-copy-data/verify-used-space-dashboard.png)

Repita los pasos anteriores para copiar los datos en el segundo nodo del dispositivo.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha obtenido información acerca de varios temas relacionados con Azure Data Box Heavy, como:

> [!div class="checklist"]
> * Conexión a un dispositivo Data Box Heavy
> * Copia de datos a Data Box Heavy


En el siguiente tutorial aprenderá a enviar su dispositivo Data Box Heavy a Microsoft.

> [!div class="nextstepaction"]
> [Envío del dispositivo Azure Data Box Heavy a Microsoft](./data-box-heavy-deploy-picked-up.md)

::: zone-end

::: zone target = "chromeless"

### <a name="copy-data-via-smb"></a>Copia de datos mediante SMB

1. Si está utilizando un host de Windows, use el siguiente comando para conectarse a los recursos compartidos de SMB:

    `\\<IP address of your device>\ShareName`

2. Para obtener las credenciales de acceso a los recursos compartidos, vaya a la página **Connect & copy** (Conectar y copiar) de la interfaz de usuario web local de Data Box.

3. Para copiar los datos, puede usar cualquier herramienta de copia de archivos compatible con SMB, como Robocopy.

Para obtener instrucciones paso a paso, consulte [Tutorial: Copia de datos a Azure Data Box Disk mediante SMB](data-box-heavy-deploy-copy-data.md).

### <a name="copy-data-via-nfs"></a>Copia de datos mediante NFS

1. Si usa un host NFS, utilice el siguiente comando para montar los recursos compartidos de NFS:

    `sudo mount <Data Box device IP>:/<NFS share on Data Box device> <Path to the folder on local Linux computer>`

2. Para obtener las credenciales de acceso a los recursos compartidos, vaya a la página **Conectar y copiar** de la interfaz de usuario web local de Data Box Heavy.
3. Use el comando `cp` o `rsync` para copiar los datos. 
4. Repita estos pasos para conectarse y copiar los datos en el segundo nodo de Data Box Heavy.

Para obtener instrucciones paso a paso, consulte [Tutorial: Copia de datos a Azure Data Box Disk mediante NFS](data-box-heavy-deploy-copy-data-via-nfs.md).

### <a name="copy-data-via-rest"></a>Copia de datos con REST

1. Para copiar datos mediante el almacenamiento de blobs de Data Box con las API REST, puede conectarse a través de *http* o *https*.
2. Para copiar datos al almacenamiento de blobs de Data Box, puede usar AzCopy.
3. Repita estos pasos para conectarse y copiar los datos en el segundo nodo de Data Box Heavy.

Para obtener instrucciones paso a paso, consulte [Tutorial: Copia de datos a Azure Data Box Blob Storage mediante API REST](data-box-heavy-deploy-copy-data-via-rest.md).

### <a name="copy-data-via-data-copy-service"></a>Copia de datos mediante el servicio de copia de datos

1. Para copiar datos utilizando el servicio de copia de datos tendrá que crear un trabajo. En la interfaz de usuario web local de Data Box Heavy, vaya a **Administrar > Copiar datos > Crear**.
2. Rellene los parámetros y cree un trabajo.
3. Repita estos pasos para conectarse y copiar los datos en el segundo nodo de Data Box Heavy.

Para obtener instrucciones paso a paso, consulte [Tutorial: Uso del servicio de copia de datos para copiar datos en Azure Data Box Heavy](data-box-heavy-deploy-copy-data-via-copy-service.md).

### <a name="copy-data-to-managed-disks"></a>Copia de datos en discos administrados

1. Al solicitar el dispositivo Data Box, debe haber seleccionado discos administrados como destino de almacenamiento.
2. Puede conectarse a Data Box Heavy mediante los recursos compartidos de SMB o NFS.
3. Después, puede copiar los datos con las herramientas de SMB o NFS.
4. Repita estos pasos para conectarse y copiar los datos en el segundo nodo de Data Box Heavy.

Para obtener instrucciones paso a paso, consulte [Tutorial: Use Data Box Heavy para importar datos como discos administrados en Azure](data-box-heavy-deploy-copy-data-from-vhds.md).

::: zone-end


