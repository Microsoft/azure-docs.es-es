---
title: Tutorial sobre cómo copiar datos en Azure Data Box Disk| Microsoft Docs
description: En este tutorial, aprenderá a copiar datos de un equipo host a Azure Data Box Disk y, después, generar sumas de comprobación para comprobar su integridad.
services: databox
author: alkohli
ms.service: databox
ms.subservice: disk
ms.topic: tutorial
ms.date: 09/03/2019
ms.author: alkohli
ms.localizationpriority: high
ms.openlocfilehash: d33f53ef3d6ea0ef6a3040a82ec17b3089075949
ms.sourcegitcommit: 4f1c7df04a03856a756856a75e033d90757bb635
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "87927120"
---
::: zone target="docs"

# <a name="tutorial-copy-data-to-azure-data-box-disk-and-verify"></a>Tutorial: Copia de datos a Azure Data Box Disk y comprobación de los mismos

::: zone-end

::: zone target="chromeless"

## <a name="copy-data-to-azure-data-box-disk-and-validate"></a>Copia de datos a Azure Data Box Disk y validación

Una vez conectados y desbloqueados los discos, puede copiar los datos del servidor de datos de origen a los discos. Una vez completada la copia de datos, debe validar los datos para asegurarse de que se cargarán correctamente en Azure.

::: zone-end

::: zone target="docs"

Este tutorial describe cómo copiar datos desde el equipo host y, a continuación, generar sumas de comprobación para comprobar su integridad.

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Copia de datos a Data Box Disk
> * Comprobación de datos

## <a name="prerequisites"></a>Requisitos previos

Antes de comenzar, asegúrese de que:
- Ha completado el [Tutorial: Instalación y operación configuración de Azure Data Box Disk](data-box-disk-deploy-set-up.md).
- Los discos se desbloquean y se conectan a un equipo cliente.
- El equipo cliente que se utiliza para copiar datos en los discos debe ejecutar un [sistema operativo compatible](data-box-disk-system-requirements.md#supported-operating-systems-for-clients).
- Asegúrese de que el tipo de almacenamiento previsto para los datos coincide con los [tipos de almacenamiento admitidos](data-box-disk-system-requirements.md#supported-storage-types-for-upload).
- Revise los [límites de los discos administrados en los límites de tamaño de objeto de Azure](data-box-disk-limits.md#azure-object-size-limits).


## <a name="copy-data-to-disks"></a>Copia de datos a discos

Antes de copiar los datos en los discos, revise las siguientes consideraciones:

- Es responsabilidad suya asegurarse de que copia los datos en las carpetas que se corresponden con el formato de datos adecuado. Por ejemplo, copie los datos de blobs en bloques en la carpeta para blobs en bloques. Si el formato de los datos no coincide con la carpeta correspondiente (tipo de almacenamiento), en un paso posterior, la carga de datos en Azure produce un error.
- Al copiar los datos, asegúrese de que su tamaño se ajusta a los límites descritos en los [límites de almacenamiento de Azure y Data Box Disk](data-box-disk-limits.md).
- Si los datos que carga Data Box Disk están siendo cargados a la vez por otras aplicaciones fuera de Data Box Disk, podría provocar errores en el trabajo de carga y daños en los datos.

   > [!IMPORTANT]
   >  Si se especificaron discos administrados como uno de los destinos de almacenamiento durante la creación del pedido, continúe con la siguiente sección.

- Solo puede tener un disco administrado con un nombre determinado en un grupo de recursos en las carpetas que se crearon previamente y en Data Box Disk. Esto implica que los VHD cargados en las carpetas creadas previamente deben tener nombres únicos. Asegúrese de que el nombre indicado no coincida con un disco administrado ya existente en un grupo de recursos. Si los discos duros virtuales tienen los mismos nombres, solo uno se convierte en un disco administrado con ese nombre. Los otros discos duros virtuales se cargan como blobs en páginas en la cuenta de almacenamiento provisional.
- Copie siempre los VHD en una de las carpetas creadas previamente. Si copia los VHD fuera de estas carpetas o en una carpeta que usted mismo haya creado, se cargarán en la cuenta de Azure Storage como blobs en páginas y no como discos administrados.
- Solo se pueden cargar VHD fijos para crear discos administrados. No se admiten los discos duros virtuales dinámicos, los de diferenciación ni los archivos VHDX.


Realice los pasos siguientes para conectarse y copiar datos desde el equipo a Data Box Disk.

1. Vea el contenido de la unidad desbloqueada. La lista de las carpetas y subcarpetas creadas previamente en la unidad difiere según las opciones seleccionadas al realizar el pedido de Data Box Disk.

    |Destino de almacenamiento seleccionado  |Tipo de cuenta de almacenamiento|Tipo de cuenta de almacenamiento provisional |Carpetas y subcarpetas  |
    |---------|---------|---------|------------------|
    |Cuenta de almacenamiento     |GPv1 o GPv2                 | N/D | BlockBlob <br> PageBlob <br> AzureFile        |
    |Cuenta de almacenamiento     |Cuenta de Blob Storage         | N/D | BlockBlob        |
    |Discos administrados     |N/D | GPv1 o GPv2         | ManagedDisk<ul> <li>PremiumSSD</li><li>StandardSSD</li><li>StandardHDD</li></ul>        |
    |Cuenta de almacenamiento <br> Discos administrados     |GPv1 o GPv2 | GPv1 o GPv2         |BlockBlob <br> PageBlob <br> AzureFile <br> ManagedDisk<ul> <li> PremiumSSD </li><li>StandardSSD</li><li>StandardHDD</li></ul>         |
    |Cuenta de almacenamiento <br> Discos administrados    |Cuenta de Blob Storage | GPv1 o GPv2         |BlockBlob <br> ManagedDisk<ul> <li>PremiumSSD</li><li>StandardSSD</li><li>StandardHDD</li></ul>         |

    A continuación se muestra una captura de pantalla de ejemplo de pedido donde se especifica una cuenta de almacenamiento GPv2:

    ![Contenido de la unidad de disco](media/data-box-disk-deploy-copy-data/data-box-disk-content.png)
 
2. Copie los datos que deban importarse como blobs en bloques en la carpeta *BlockBlob*. De forma similar, puede copiar datos del tipo VHD/VHDX en la carpeta *PageBlob* y otros datos en la carpeta *AzureFile*.

    Se crea un contenedor en la cuenta de almacenamiento de Azure para cada subcarpeta bajo las carpetas BlockBlob y PageBlob. Todos los archivos bajo las carpetas BlockBlob y PageBlob se copian en un contenedor predeterminado `$root` bajo la cuenta de Azure Storage. Todos los archivos del contenedor `$root` siempre se cargan como blobs en bloques.

   Copie los archivos en una subcarpeta de la carpeta *AzureFile*. Una subcarpeta dentro de *AzureFile* crea un recurso compartido de archivos. Los archivos que se copien directamente en la carpeta *AzureFile* presentan errores y se cargan como blobs en bloques.

    Si existen archivos y carpetas en el directorio raíz, debe pasarlos a una carpeta diferente antes de comenzar la copia de los datos.

    > [!IMPORTANT]
    > Todos los contenedores, blobs y nombres de archivos deben cumplir las [convenciones de nomenclatura de Azure](data-box-disk-limits.md#azure-block-blob-page-blob-and-file-naming-conventions). Si no se siguen estas reglas, se producirá un error en la carga de datos en Azure.

3. Al copiar archivos, asegúrese de que estos no superen ~4,7 TiB (blobs en bloques), ~8 TiB (blobs en páginas) y ~1 TiB (Azure Files). 
4. Puede usar arrastrar y colocar con el Explorador de archivos para copiar los datos. También puede usar cualquier herramienta de copia de archivos compatible con SMB, como Robocopy, para copiar los datos. Con el siguiente comando de Robocopy, se pueden iniciar varios trabajos de copia:

    `Robocopy <source> <destination>  * /MT:64 /E /R:1 /W:1 /NFL /NDL /FFT /Log:c:\RobocopyLog.txt` 
    
    Los parámetros y las opciones para el comando se tabulan como sigue:
    
    |Parámetros/opciones  |Descripción |
    |--------------------|------------|
    |Source            | Especifica la ruta de acceso del directorio de origen.        |
    |Destination       | Especifica la ruta de acceso del directorio de destino.        |
    |/E                  | Copia los subdirectorios incluyendo los directorios vacíos. |
    |/MT[:N]             | Crea copias multiproceso con N subprocesos, donde N es un entero entre 1 y 128. <br>El valor predeterminado de N es 8.        |
    |/R: \<N>             | Especifica el número de reintentos en las copias con errores. El valor predeterminado de N es 1 000 000 (un millón de reintentos).        |
    |/W: \<N>             | Especifica el tiempo de espera entre reintentos, en segundos. El valor predeterminado de N es 30 (30 segundos de tiempo de espera).        |
    |/NFL                | Especifica que los nombres de archivo no se van a registrar.        |
    |/NDL                | Especifica que los nombres de directorio no se van a registrar.        |
    |/FFT                | Se da por supuesto que se trata de tiempos de archivos FAT (precisión de dos segundos).        |
    |/Log:\<Log File>     | Escribe la salida del estado en el archivo de registro (sobrescribe el archivo de registro existente).         |

    Se pueden usar varios discos en paralelo con varios trabajos que se ejecutan en cada disco.

6. Se comprueba el estado de la copia cuando el trabajo está en curso. El ejemplo siguiente muestra la salida del comando robocopy para copiar archivos en Data Box Disk.

    ```
    C:\Users>robocopy
        -------------------------------------------------------------------------------
       ROBOCOPY     ::     Robust File Copy for Windows
    -------------------------------------------------------------------------------
    
      Started : Thursday, March 8, 2018 2:34:53 PM
           Simple Usage :: ROBOCOPY source destination /MIR
    
                 source :: Source Directory (drive:\path or \\server\share\path).
            destination :: Destination Dir  (drive:\path or \\server\share\path).
                   /MIR :: Mirror a complete directory tree.
    
        For more usage information run ROBOCOPY /?    
    
    ****  /MIR can DELETE files as well as copy them !
    
    C:\Users>Robocopy C:\Git\azure-docs-pr\contributor-guide \\10.126.76.172\devicemanagertest1_AzFile\templates /MT:64 /E /R:1 /W:1 /FFT 
    -------------------------------------------------------------------------------
       ROBOCOPY     ::     Robust File Copy for Windows
    -------------------------------------------------------------------------------
    
      Started : Thursday, March 8, 2018 2:34:58 PM
       Source : C:\Git\azure-docs-pr\contributor-guide\
         Dest : \\10.126.76.172\devicemanagertest1_AzFile\templates\
    
        Files : *.*
    
      Options : *.* /DCOPY:DA /COPY:DAT /MT:8 /R:1000000 /W:30
    
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
       Times :   0:00:05   0:00:00                       0:00:00   0:00:00
        
       Speed :                5620 Bytes/sec.
       Speed :               0.321 MegaBytes/min.
       Ended : Thursday, March 8, 2018 2:34:59 PM
        
    C:\Users>
    ```
 
    Para optimizar el rendimiento, use los siguientes parámetros de robocopy al copiar los datos.

    |    Plataforma    |    Archivos pequeños principalmente < 512 KB                           |    Archivos medianos principalmente, de 512 KB a 1 MB                      |    Archivos grandes principalmente > 1 MB                             |   
    |----------------|--------------------------------------------------------|--------------------------------------------------------|--------------------------------------------------------|
    |    Data Box Disk        |    4 sesiones de Robocopy* <br> 16 subprocesos por sesión    |    2 sesiones de Robocopy* <br> 16 subprocesos por sesión    |    2 sesiones de Robocopy* <br> 16 subprocesos por sesión    |
    
    **Cada sesión de Robocopy puede tener un máximo de 7000 directorios y 150 millones de archivos.*
    
    >[!NOTE]
    > Los parámetros que se han sugerido anteriormente se basan en el entorno que se ha usado en las pruebas internas.
    
    Para más información sobre el comando Robocopy, consulte [Robocopy and a few examples](https://social.technet.microsoft.com/wiki/contents/articles/1073.robocopy-and-a-few-examples.aspx) (Robocopy y algunos ejemplos).

6. Abra la carpeta de destino para ver y comprobar los archivos copiados. Si se produce algún error durante el proceso de copia, descargue los archivos de registro para solucionar problemas. Los archivos de registro se ubican tal como se especifica en el comando robocopy.
 
### <a name="split-and-copy-data-to-disks"></a>División y copia de datos en discos

Este procedimiento opcional se puede usar cuando se utilizan varios discos y se tiene un conjunto de datos grande que es necesario dividir y copiar entre todos los discos. La herramienta Data Box Split Copy ayuda a dividir y copia los datos en un equipo Windows.

>[!IMPORTANT]
> La herramienta de copia y división de Data Box también le permite validar los datos. Si usa la herramienta de copia y división de Data Box para copiar datos, puede omitir el [paso de validación](#validate-data).
> Esta herramienta de copia y división no es compatible con los discos administrados.

1. En el equipo Windows, asegúrese de que se ha descargado la herramienta Data Box Split Copy y que se ha extraído en una carpeta local. Esta herramienta se descargó cuando se descargó el conjunto de herramientas de Data Box Disk para Windows.
2. Abra el Explorador de archivos. Anote la unidad del origen de datos y las letras de unidad asignadas a Data Box Disk. 

     ![Dividir datos de copia](media/data-box-disk-deploy-copy-data/split-copy-1.png)
 
3. Identificar los datos de origen que se copian. Por ejemplo, en este caso:
    - Se han identificado los siguientes datos del blob en bloques.

         ![Dividir datos de copia](media/data-box-disk-deploy-copy-data/split-copy-2.png)    

    - Se han identificado los siguientes datos del blob en páginas.

         ![Dividir datos de copia](media/data-box-disk-deploy-copy-data/split-copy-3.png)
 
4. Vaya a la carpeta en la que se extrae el software. Busque el archivo `SampleConfig.json` en esa carpeta. Se trata de un archivo de solo lectura que puede modificar y guardar.

   ![Dividir datos de copia](media/data-box-disk-deploy-copy-data/split-copy-4.png)
 
5. Modifique el archivo `SampleConfig.json`.
 
   - Especifique el nombre de un trabajo. Se crea una carpeta en Data Box Disk y se convierte en el contenedor de la cuenta de Azure Storage asociada con estos discos. El nombre del trabajo debe seguir las convenciones de nomenclatura de contenedores de Azure. 
   - Especifique una ruta de acceso de origen anotando el formato de la ruta de acceso de `SampleConfigFile.json`. 
   - Escriba las letras de unidad correspondiente a los discos de destino. Los datos se toman de la ruta de acceso de origen y se copian en varios discos.
   - Especifique una ruta de acceso para los archivos de registro. De forma predeterminada, se envía al directorio actual en el que se encuentra el archivo `.exe`.

     ![Dividir datos de copia](media/data-box-disk-deploy-copy-data/split-copy-5.png)

6. Para validar el formato de archivo, vaya a `JSONlint`. Guarde el archivo como `ConfigFile.json`. 

     ![Dividir datos de copia](media/data-box-disk-deploy-copy-data/split-copy-6.png)
 
7. Abra una ventana de símbolo del sistema. 

8. Ejecute `DataBoxDiskSplitCopy.exe`. Tipo

    `DataBoxDiskSplitCopy.exe PrepImport /config:<Your-config-file-name.json>`

     ![Dividir datos de copia](media/data-box-disk-deploy-copy-data/split-copy-7.png)
 
9. Entre para continuar el script.

    ![Dividir datos de copia](media/data-box-disk-deploy-copy-data/split-copy-8.png)
  
10. Cuando el conjunto de datos se divide y se copia, se presenta el resumen de la herramienta Split Copy para la sesión de copia. A continuación se muestra una salida de ejemplo.

    ![Dividir datos de copia](media/data-box-disk-deploy-copy-data/split-copy-9.png)
 
11. Compruebe que los datos se dividen entre los discos de destino. 
 
    ![Dividir datos de copia](media/data-box-disk-deploy-copy-data/split-copy-10.png)
    ![Split copy data](media/data-box-disk-deploy-copy-data/split-copy-11.png)
     
    Si examina más a fondo el contenido de la unidad `n:`, verá que se crean dos subcarpetas correspondientes a los datos con formato de blob en bloques y blob en páginas.
    
     ![Dividir datos de copia](media/data-box-disk-deploy-copy-data/split-copy-12.png)

12. Si se produce un error en la sesión de copia, para recuperar los datos y reanudarla, use el siguiente comando:

    `DataBoxDiskSplitCopy.exe PrepImport /config:<configFile.json> /ResumeSession`

Si ve errores al usar la herramienta de copia y división, consulte cómo [solucionar los errores de la herramienta de copia y división](data-box-disk-troubleshoot-data-copy.md).

Una vez completada la copia de datos, puede continuar para validar los datos. Si ha usado la herramienta de copia y división, omita la validación (esta herramienta también valida) y vaya al siguiente tutorial.


## <a name="validate-data"></a>Validación de datos

Si no usó la herramienta de copia y división para copiar los datos, tendrá que validarlos. Para comprobar los datos, realice los pasos siguientes.

1. Ejecute `DataBoxDiskValidation.cmd` para la validación de la suma de comprobación en la carpeta *DataBoxDiskImport* de la unidad. Solo está disponible para el entorno Windows. Los usuarios de Linux deben validar que los datos de origen que se copian en el disco cumplan los [requisitos previos](https://docs.microsoft.com/azure/databox/data-box-disk-limits).
    
    ![Salida de la herramienta de validación Data Box Disk](media/data-box-disk-deploy-copy-data/data-box-disk-validation-tool-output.png)

2. Elija la opción más adecuada. **Se recomienda validar siempre los archivos y generar sumas de comprobación mediante la selección de la opción 2**. Según el tamaño de los datos, este paso puede tardar un rato. Una vez completado el script, salga de la ventana de comandos. Si se produce algún error durante la validación y la generación de la suma de comprobación, se le notificará y también se le proporcionará un vínculo a los registros de errores.

    ![Salida de la suma de comprobación](media/data-box-disk-deploy-copy-data/data-box-disk-checksum-output.png)

    > [!TIP]
    > - Restablezca la herramienta entre dos ejecuciones.
    > - Utilice la opción 1 si trata con conjuntos de datos grandes que contienen archivos pequeños (~ KB). Esta opción solo valida los archivos, la generación de la suma de comprobación puede llevar mucho tiempo y el rendimiento puede ser muy lento.

3. Si usa varios discos, ejecute el comando para cada disco.

Si ve errores durante la validación, consulte la [resolución de los errores de validación](data-box-disk-troubleshoot.md).

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha aprendido sobre temas relacionados con Azure Data Box Disk; por ejemplo:

> [!div class="checklist"]
> * Copia de datos a Data Box Disk
> * Comprobar la integridad de los datos

Pase al siguiente tutorial para obtener información sobre cómo devolver Data Box Disk y comprobar la carga de datos en Azure.

> [!div class="nextstepaction"]
> [Devolución de Azure Data Box Disk a Microsoft](./data-box-disk-deploy-picked-up.md)

::: zone-end

::: zone target="chromeless"

### <a name="copy-data-to-disks"></a>Copia de datos a discos

Realice los pasos siguientes para conectarse y copiar datos desde el equipo a Data Box Disk.

1. Vea el contenido de la unidad desbloqueada. La lista de las carpetas y subcarpetas creadas previamente en la unidad difiere según las opciones seleccionadas al realizar el pedido de Data Box Disk.
2. Copie los datos en carpetas que se correspondan con el formato de datos adecuado. Por ejemplo, copie los datos no estructurados en la carpeta para *BlockBlob*, los datos VHD o VHDX en la carpeta *PageBlob* y los archivos en *AzureFile*. Si el formato de los datos no coincide con la carpeta apropiada (tipo de almacenamiento), más adelante, en un paso posterior, la carga de datos en Azure produce un error.

    - Asegúrese de que todos los contenedores, blobs y archivos deben cumplir las [convenciones de nomenclatura de Azure](data-box-disk-limits.md#azure-block-blob-page-blob-and-file-naming-conventions) y los [límites de tamaño de objeto de Azure](data-box-disk-limits.md#azure-object-size-limits). Si no se siguen estas reglas o límites, se producirá un error en la carga de datos en Azure.     
    - Si el pedido tiene Managed Disks como uno de los destinos de almacenamiento, consulte las convenciones de nomenclatura de [discos administrados](data-box-disk-limits.md#managed-disk-naming-conventions).
    - Se crea un contenedor en la cuenta de almacenamiento de Azure para cada subcarpeta bajo las carpetas BlockBlob y PageBlob. Todos los archivos en las carpetas *BlockBlob* y *PageBlob* se copian en un contenedor predeterminado $root en la cuenta de Azure Storage. Todos los archivos del contenedor $root siempre se cargan como blobs en bloques.
    - Cree una subcarpeta en la carpeta *AzureFile*. Esta subcarpeta se asigna a un recurso compartido en la nube. Copie los archivos a la subcarpeta. Los archivos que se copien directamente en la carpeta *AzureFile* presentan errores y se cargan como blobs en bloques.
    - Si existen archivos y carpetas en el directorio raíz, debe pasarlos a una carpeta diferente antes de comenzar la copia de los datos.

3. Puede arrastrar y colocar con el Explorador de archivos o utilizar cualquier herramienta de copia de archivos compatible con SMB, como Robocopy, para copiar los datos. Con el siguiente comando, se pueden iniciar varios trabajos de copia:

    ```
    Robocopy <source> <destination>  * /MT:64 /E /R:1 /W:1 /NFL /NDL /FFT /Log:c:\RobocopyLog.txt
    ```
4. Abra la carpeta de destino para ver y comprobar los archivos copiados. Si se produce algún error durante el proceso de copia, descargue los archivos de registro para solucionar problemas. Los archivos de registro se ubican tal como se especifica en el comando robocopy.

Use el procedimiento opcional de [división y copia](data-box-disk-deploy-copy-data.md#split-and-copy-data-to-disks) cuando se esté utilizando varios discos y tenga un conjunto de datos grande que tiene dividir y copiar en todos los discos.

### <a name="validate-data"></a>Validación de datos

Siga estos pasos para comprobar los datos.

1. Ejecute `DataBoxDiskValidation.cmd` para la validación de la suma de comprobación en la carpeta *DataBoxDiskImport* de la unidad.
2. Use la opción 2 para validar los archivos y generar sumas de comprobación. Según el tamaño de los datos, este paso puede tardar un rato. Si se produce algún error durante la validación y la generación de la suma de comprobación, se le notificará y también se le proporcionará un vínculo a los registros de errores.

    Para más información sobre la validación de datos, consulte, [Validación de datos](https://docs.microsoft.com/azure/databox/data-box-disk-deploy-copy-data#validate-data). Si experimenta errores durante la validación, consulte la [resolución de los errores de validación](data-box-disk-troubleshoot.md).

::: zone-end
