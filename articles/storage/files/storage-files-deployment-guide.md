---
title: Implementación de Azure Files | Microsoft Docs
description: Obtenga información sobre cómo implementar Azure Files de principio a fin. Transfiera datos a Azure Files. Montaje automático en los equipos y servidores en que sea necesario.
author: roygara
ms.service: storage
ms.topic: how-to
ms.date: 05/22/2018
ms.author: rogarana
ms.subservice: files
ms.openlocfilehash: 08bcb41dd8d9f4643b03d855960d8632b778ff84
ms.sourcegitcommit: bfeae16fa5db56c1ec1fe75e0597d8194522b396
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/10/2020
ms.locfileid: "88034505"
---
# <a name="how-to-deploy-azure-files"></a>Implementación de Azure Files
[Azure Files](storage-files-introduction.md) ofrece recursos compartidos de archivos en la nube totalmente administrados, a los que se puede acceder mediante el protocolo SMB estándar. Este artículo le mostrará cómo implementar de forma práctica Azure Files dentro de su organización.

Se recomienda encarecidamente leer [Planning for an Azure Files deployment](storage-files-planning.md) (Planear una implementación de Azure Files) antes de seguir los pasos descritos en este artículo.

## <a name="prerequisites"></a>Prerrequisitos
En este artículo se supone que ya ha completado los siguientes pasos:

- Ha creado una cuenta de Azure Storage con las opciones de resistencia y cifrado deseadas en la región que quería. Consulte [Acerca de las cuentas de almacenamiento de Azure](../common/storage-create-storage-account.md?toc=%2fazure%2fstorage%2ffiles%2ftoc.json) para obtener instrucciones paso a paso sobre cómo crear una cuenta de almacenamiento.
- Ha creado un recurso compartido de archivos de Azure con su cuota deseada en su cuenta de almacenamiento. Consulte [Creación de un recurso compartido de archivos en Azure File Storage](storage-how-to-create-file-share.md) para obtener instrucciones paso a paso sobre cómo crear un recurso compartido de archivos.

## <a name="transfer-data-into-azure-files"></a>Transferencia de datos a Azure Files
Puede migrar los recursos compartidos de archivos existentes, como los almacenados localmente, al nuevo recurso compartido de archivos de Azure. En esta sección se mostrará cómo mover datos a un recurso compartido de archivos de Azure mediante varios métodos populares que se detallan en la [guía de planeamiento](storage-files-planning.md#migration).

### <a name="azure-file-sync"></a>Azure File Sync
Azure File Sync le permite centralizar los recursos compartidos de archivos de su organización en Azure Files sin renunciar a la flexibilidad, el rendimiento y la compatibilidad de un servidor de archivos local. Para ello, transforma los servidores de Windows Server en una caché rápida del recurso compartido de archivos de Azure. Puede usar cualquier protocolo disponible en Windows Server para tener acceso a los datos localmente (incluidos SMB, NFS y FTPS) y puede tener tantas cachés según sea necesario en todo el mundo.

Azure File Sync se puede usar para migrar datos a un recurso compartido de archivos de Azure, aunque el mecanismo de sincronización no se desee para usarlo a largo plazo. Se puede encontrar más información acerca de cómo usar Azure File Sync para transferir datos a un recurso compartido de archivos de Azure en [Planeamiento de una implementación de Azure File Sync (versión preliminar)](storage-sync-files-planning.md) e [Implementación de Azure File Sync](storage-sync-files-deployment-guide.md).

### <a name="azure-importexport"></a>Azure Import/Export
El servicio Azure Import/Export permite transferir de forma segura grandes cantidades de datos a un recurso compartido de archivos de Azure mediante el envío de unidades de disco duro a un centro de datos de Azure. Consulte [Uso del servicio Microsoft Azure Import/Export para transferir datos a Blob Storage](../common/storage-import-export-service.md?toc=%2fazure%2fstorage%2ffiles%2ftoc.json) para obtener una descripción más detallada del servicio.

> [!Note]  
> Actualmente, el servicio Azure Import/Export no admite la exportación de archivos desde un recurso compartido de archivos de Azure.

Los pasos siguientes importarán datos desde una ubicación local a un recurso compartido de archivos de Azure.

1. Obtenga el número necesario de discos duros para correo en Azure. Los discos duros pueden tener cualquier tamaño de disco, pero deben ser un SSD o HHD de 2,5" o 3,5" que admita el estándar SATA II o SATA III. 

2. Conecte y monte cada disco en el servidor o PC que realice la transferencia de datos. Para obtener un rendimiento óptimo, se recomienda ejecutar el trabajo de exportación local internamente en el servidor que contiene los datos. En algunos casos, por ejemplo, cuando el servidor de archivos que proporciona los datos es un dispositivo NAS, esto puede no ser posible. En ese caso, es totalmente aceptable montar cada disco en un equipo.

3. Asegúrese de que cada unidad de disco está en línea, inicializada y tiene asignada una letra de unidad. Para poner en línea e inicializar una unidad y asignarla una letra de unidad, abra el complemento MMC de Administración de discos (diskmgmt.msc).

    - Para conectar un disco en línea (si aún no lo está), haga clic con el botón derecho en el disco en el panel inferior de MMC Administración de discos y seleccione "En línea".
    - Para inicializar un disco, haga clic con el botón derecho en el disco en el panel inferior (después de que el disco esté en línea) y seleccione "Inicializar". No olvide seleccionar "GPT" cuando se le pida.

        ![Captura de pantalla del menú Inicializar disco en MMC Administración de discos](media/storage-files-deployment-guide/transferdata-importexport-1.PNG)

    - Para asignar una letra de unidad en el disco, haga clic con el botón derecho en el espacio "No asignado" del disco en línea e inicializado y haga clic en "Nuevo volumen simple". Esto le permitirá asignar una letra de unidad. Tenga en cuenta que no es necesario formatear el volumen, ya que esta acción se realizará más tarde.

        ![Captura de pantalla del Asistente para nuevo volumen simple en MMC Administración de discos](media/storage-files-deployment-guide/transferdata-importexport-2.png)

4. Cree el archivo CSV de conjunto de datos. El archivo CSV del conjunto de datos es una asignación entre la ruta de acceso a los datos locales y el recurso compartido de archivos de Azure deseado en el que se deben copiar los datos. Por ejemplo, el siguiente archivo CSV del conjunto de datos asigna un recurso compartido de archivos local ("F:\shares\scratch") a un recurso compartido de archivos de Azure ("MyAzureFileShare"):
    
    ```
    BasePath,DstItemPathOrPrefix,ItemType,Disposition,MetadataFile,PropertiesFile
    "F:\shares\scratch\","MyAzureFileShare/",file,rename,"None",None
    ```

    Se pueden especificar varios recursos compartidos con una cuenta de almacenamiento. Vea [Preparación del archivo CSV de conjunto de datos](../common/storage-import-export-tool-preparing-hard-drives-import.md?toc=%2fazure%2fstorage%2ffiles%2ftoc.json) para más información.

5. Cree el archivo CSV de conjunto de unidades. El archivo CSV de conjunto de unidades enumera los discos disponibles para el agente de exportación local. Por ejemplo, el siguiente archivo CSV de conjunto de unidades enumera las unidades `X:`, `Y:` y `Z:` que se utilizarán en el trabajo de exportación local:

    ```
    DriveLetter,FormatOption,SilentOrPromptOnFormat,Encryption,ExistingBitLockerKey
    X,Format,SilentMode,Encrypt,
    Y,Format,SilentMode,Encrypt,
    Z,Format,SilentMode,Encrypt,
    ```
    
    Consulte [Preparación del archivo CSV InitialDriveSet o AdditionalDriveSet](../common/storage-import-export-tool-preparing-hard-drives-import.md?toc=%2fazure%2fstorage%2ffiles%2ftoc.json) para más información.

6. Utilice la herramienta [WAImportExport](https://www.microsoft.com/download/details.aspx?id=55280) para copiar sus datos a uno o varios discos duros.

    ```
    WAImportExport.exe PrepImport /j:<JournalFile> /id:<SessionId> [/logdir:<LogDirectory>] [/sk:<StorageAccountKey>] [/silentmode] [/InitialDriveSet:<driveset.csv>] DataSet:<dataset.csv>
    ```

    > [!Warning]  
    > No modifique los datos de las unidades de disco duro o el archivo de diario después de completar la preparación del disco.

7. [Cree un trabajo de importación](../common/storage-import-export-data-to-files.md#step-2-create-an-import-job).
    
### <a name="robocopy"></a>Robocopy
Robocopy es una herramienta de copia conocida que se incluye con Windows y Windows Server. Robocopy puede usarse para transferir datos a Azure Files al montar el recurso compartido de archivos localmente y luego usar la ubicación montada como destino en el comando de Robocopy. El uso de Robocopy es bastante sencillo:

1. [Monte un recurso compartido de archivos de Azure](storage-how-to-use-files-windows.md). Para obtener un rendimiento óptimo, es aconsejable montar el recurso compartido de archivos de Azure localmente, en el servidor que contiene los datos. En algunos casos, por ejemplo, cuando el servidor de archivos que proporciona los datos es un dispositivo NAS, esto puede no ser posible. En ese caso, es totalmente aceptable montar el recurso compartido de archivos de Azure en un equipo. En este ejemplo, `net use` se utiliza en la línea de comandos para montar el recurso compartido de archivos:

    ```
    net use <desired-drive-letter>: \\<storage-account-name>.file.core.windows.net\<share-name> <storage-account-key> /user:Azure\<storage-account-name>
    ```

2. Use `robocopy` en la línea de comandos para mover datos al recurso compartido de archivos de Azure.

    ```
    robocopy <path-to-local-share> <path-to-azure-file-share> /E /Z /MT:32
    ```
    
    Robocopy tiene un número considerable de opciones para modificar el comportamiento de la copia según sea necesario. Para más información, consulte la página del manual de [Robocopy](https://technet.microsoft.com/library/cc733145.aspx).

### <a name="azcopy"></a>AzCopy
AzCopy es una utilidad de línea de comandos diseñada para copiar datos a y desde Azure Files, así como Azure Blob Storage, mediante sencillos comandos con un rendimiento óptimo. Uso de AzCopy:

1. Descargue la [versión más reciente de AzCopy en Windows](https://aka.ms/downloadazcopy) o [Linux](https://docs.microsoft.com/previous-versions/azure/storage/storage-use-azcopy-linux#download-and-install-azcopy).
2. Use `azcopy` en la línea de comandos para mover datos al recurso compartido de archivos de Azure. La sintaxis en Windows es la siguiente: 

    ```
    azcopy /Source:<path-to-local-share> /Dest:https://<storage-account>.file.core.windows.net/<file-share>/ /DestKey:<storage-account-key> /S
    ```

    En Linux, la sintaxis del comando es un poco diferente:

    ```
    azcopy --source <path-to-local-share> --destination https://<storage-account>.file.core.windows.net/<file-share>/ --dest-key <storage-account-key> --recursive
    ```

    AzCopy tiene un número considerable de opciones para modificar el comportamiento de la copia según sea necesario. Para más información, consulte [AzCopy en Windows](../common/storage-use-azcopy.md?toc=%2fazure%2fstorage%2ffiles%2ftoc.json) y [AzCopy en Linux](../common/storage-use-azcopy-linux.md?toc=%2fazure%2fstorage%2ffiles%2ftoc.json).

## <a name="automatically-mount-on-needed-pcsservers"></a>Montaje automático en los equipos y servidores necesarios
Para reemplazar un recurso compartido de archivos local, es útil montar previamente los recursos compartidos en las máquinas en las que se usará. Esto puede hacerse automáticamente en una lista de máquinas.

> [!Note]  
> Montar un recurso compartido de archivos de Azure requiere el uso de la clave de la cuenta de almacenamiento como contraseña, por lo que solo se recomienda hacerlo en entornos de confianza. 

### <a name="windows"></a>Windows
Puede usar PowerShell para ejecutar el comando de montaje en varios equipos. En el ejemplo siguiente, `$computers` es rellenar de forma manual, pero puede generar la lista de equipos para montar automáticamente. Por ejemplo, puede rellenar esta variable con resultados de Active Directory.

```powershell
$computer = "MyComputer1", "MyComputer2", "MyComputer3", "MyComputer4"
$computer | ForEach-Object { Invoke-Command -ComputerName $_ -ScriptBlock { net use <desired-drive-letter>: \\<storage-account-name>.file.core.windows.net\<share-name> <storage-account-key> /user:Azure\<storage-account-name> /PERSISTENT:YES } }
```

### <a name="linux"></a>Linux
Un script de Bash simple combinado con SSH puede producir el mismo resultado en el ejemplo siguiente. De forma similar, la variable `$computer` se deja para que la rellene el usuario:

```
computer = ("MyComputer1" "MyComputer2" "MyComputer3" "MyComputer4")
for item in "${computer[@]}"
do
    ssh $item "sudo bash -c 'echo \"//<storage-account-name>.file.core.windows.net/<share-name> /mymountpoint cifs vers=3.0,username=<storage-account-name>,password=<storage-account-key>,dir_mode=0777,file_mode=0777,serverino\" >> /etc/fstab'", "sudo mount -a"
done
```

## <a name="next-steps"></a>Pasos siguientes
- [Planeamiento de una implementación de Azure Files Sync](storage-sync-files-planning.md)
- [Solución de problemas de Azure Files en Windows](storage-troubleshoot-windows-file-connection-problems.md)
- [Solución de problemas de Azure Files en Linux](storage-troubleshoot-linux-file-connection-problems.md)
