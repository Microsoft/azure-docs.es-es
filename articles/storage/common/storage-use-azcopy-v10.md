---
title: Copiar o mover datos a Azure Storage mediante AzCopy v10 | Microsoft Docs
description: Use la utilidad de línea de comandos AzCopy para mover o copiar datos hacia o desde el contenido del archivo, data lake y blob. Copie datos a Azure Storage desde archivos locales o copie datos en o entre cuentas de almacenamiento. Migre fácilmente sus datos a Azure Storage.
services: storage
author: normesta
ms.service: storage
ms.topic: article
ms.date: 04/23/2019
ms.author: normesta
ms.reviewer: seguler
ms.subservice: common
ms.openlocfilehash: b5a13dfd760f0c94343b151c9b4c1148c949e854
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65790011"
---
# <a name="transfer-data-with-azcopy-v10"></a>Transferencia de datos con AzCopy v10

AzCopy es la utilidad de línea de comandos para copiar datos hacia o desde Microsoft Azure Blob y File storage. AzCopy ofrece una interfaz de línea de comandos rediseñada y transfiere la nueva arquitectura de datos confiable. Mediante el uso de AzCopy, puede copiar datos entre un sistema de archivos y una cuenta de almacenamiento, o entre cuentas de almacenamiento.

## <a name="whats-new-in-azcopy-v10"></a>Novedades de AzCopy v10

- Sincroniza los sistemas de archivos a Azure Blob storage o viceversa. Mediante `azcopy sync <source> <destination>`. Ideal para escenarios de copia incremental.
- Admite las API de Azure Data Lake Storage Gen2. Use `myaccount.dfs.core.windows.net` como un URI para llamar a las API de Data Lake Storage Gen2.
- Admite la copia de una cuenta completa (solo el Blob service) a otra cuenta.
- Permite copiar datos desde un cubo de S3 de Amazon Web Services.
- Usa el nuevo [Put Block de dirección URL](https://docs.microsoft.com/rest/api/storageservices/put-block-from-url) API para admitir la copia de la cuenta de cuenta. La transferencia de datos es más rápida, ya que no se requiere la transferencia al cliente.
- Incluye o quita archivos y blobs en una ruta de acceso dada.
- Admite patrones de caracteres comodín en una ruta de acceso y--marcadores de exclusión.
- Crea un pedido de trabajo y un archivo de registro relacionados con cada instancia de AzCopy. Puede ver y reiniciar los trabajos anteriores y reanudar trabajos con errores. AzCopy también reintenta automáticamente una transferencia tras un error.
- Mejoras de rendimiento general de características.

## <a name="download-and-install-azcopy"></a>Descarga e instalación de AzCopy

### <a name="latest-production-version-v10"></a>Versión de producción más reciente (v10)

Descargue la versión más reciente de AzCopy:
- [Windows](https://aka.ms/downloadazcopy-v10-windows) (zip)
- [Linux](https://aka.ms/downloadazcopy-v10-linux) (tar)
- [MacOS](https://aka.ms/downloadazcopy-v10-mac) (zip)

### <a name="latest-azcopy-supporting-table-storage-service-v73"></a>AzCopy más reciente que admiten el servicio de almacenamiento de tabla (v7.3)

Descargue [v7.3 AzCopy que admite la copia de datos desde y hacia el servicio Table Storage de Microsoft Azure](https://aka.ms/downloadazcopynet).

## <a name="post-installation-steps"></a>Pasos posteriores a la instalación

AzCopy no requiere una instalación. Abra la aplicación de línea de comandos que prefiera y vaya a la carpeta donde `azcopy.exe` se encuentra. Si es necesario, puede agregar la ubicación de carpeta de AzCopy a la ruta del sistema para facilitar su uso.

## <a name="authentication-options"></a>Opciones de autenticación

AzCopy admite las siguientes opciones al autenticarse con Azure Storage:
- **Azure Active Directory** (compatible con **servicios Blob y Data Lake Storage Gen2**). Use ```.\azcopy login``` para iniciar sesión con Azure Active Directory.  El usuario debe tener [rol "Colaborador de datos de Blob de almacenamiento" asignado](https://docs.microsoft.com/azure/storage/common/storage-auth-aad-rbac) para escribir en el almacenamiento de blobs con autenticación de Azure Active Directory. Para la autenticación a través de las identidades administradas para los recursos de Azure, use `azcopy login --identity`.
- **Compartir los tokens de firma de acceso [admitidos para los servicios Blob y File]**. Anexe el token de acceso compartido (SAS) de firma a la ruta de acceso de blob en la línea de comandos para usarlo. Puede generar tokens de SAS con Azure portal, [Explorador de Storage](https://blogs.msdn.microsoft.com/jpsanders/2017/10/12/easily-create-a-sas-to-download-a-file-from-azure-storage-using-azure-storage-explorer/), [PowerShell](https://docs.microsoft.com/powershell/module/az.storage/new-azstorageblobsastoken), u otras herramientas que prefiera. Para más información, consulte los [ejemplos](https://docs.microsoft.com/azure/storage/blobs/common/storage-dotnet-shared-access-signature-part-1).

## <a name="getting-started"></a>Introducción

> [!TIP]
> **¿Prefiere una interfaz gráfica de usuario?**
>
> [Explorador de Azure Storage](https://azure.microsoft.com/features/storage-explorer/), un cliente de escritorio que simplifica la administración de los datos de almacenamiento de Azure, ahora usa AzCopy para acelerar la transferencia de datos a y desde Azure Storage.
>
> Habilitar AzCopy en el Explorador de Storage en el **Preview** menú.
> ![Habilitar AzCopy como un motor de transferencia en el Explorador de Storage de Azure](media/storage-use-azcopy-v10/enable-azcopy-storage-explorer.jpg)

AzCopy v10 tiene una sintaxis documentada automáticamente. Cuando haya iniciado sesión en Azure Active Directory, la sintaxis general es similar al siguiente:

```azcopy
.\azcopy <command> <arguments> --<flag-name>=<flag-value>

# Examples if you have logged into the Azure Active Directory:
.\azcopy copy <source path> <destination path> --<flag-name>=<flag-value>
.\azcopy cp "C:\local\path" "https://account.blob.core.windows.net/container" --recursive=true
.\azcopy cp "C:\local\path\myfile" "https://account.blob.core.windows.net/container/myfile"
.\azcopy cp "C:\local\path\*" "https://account.blob.core.windows.net/container"

# Examples if you're using SAS tokens to authenticate:
.\azcopy cp "C:\local\path" "https://account.blob.core.windows.net/container?st=2019-04-05T04%3A10%3A00Z&se=2019-04-13T04%3A10%3A00Z&sp=rwdl&sv=2018-03-28&sr=c&sig=Qdihej%2Bsbg4AiuyLVyQZklm9pSuVGzX27qJ508wi6Es%3D" --recursive=true
.\azcopy cp "C:\local\path\myfile" "https://account.blob.core.windows.net/container/myfile?st=2019-04-05T04%3A10%3A00Z&se=2019-04-13T04%3A10%3A00Z&sp=rwdl&sv=2018-03-28&sr=c&sig=Qdihej%2Bsbg4AiuyLVyQZklm9pSuVGzX27qJ508wi6Es%3D"
```

Así puede obtener una lista de los comandos disponibles:

```azcopy
.\azcopy --help
# To use the alias instead
.\azcopy -h
```

Para ver la página de ayuda y los ejemplos de un comando específico, ejecute el siguiente comando:

```azcopy
.\azcopy <cmd> --help
# Example:
.\azcopy cp -h
```

## <a name="create-a-blob-container-or-file-share"></a>Crear un recurso compartido de archivo o contenedor de blob 

**Crear un contenedor de blobs**

```azcopy
.\azcopy make "https://account.blob.core.windows.net/container-name"
```

**Creación de un recurso compartido de archivos**

```azcopy
.\azcopy make "https://account.file.core.windows.net/share-name"
```

**Crear un contenedor de blobs mediante Azure Data Lake Storage Gen2**

Si ha habilitado los espacios de nombres jerárquico en su cuenta de almacenamiento de blobs, puede usar el comando siguiente para crear un nuevo contenedor de blobs para cargar archivos.

```azcopy
.\azcopy make "https://account.dfs.core.windows.net/top-level-resource-name"
```

## <a name="copy-data-to-azure-storage"></a>Copia de datos a Azure Storage

Utilice el comando de copia para transferir datos desde el origen al destino. El origen o destino puede ser:
- El sistema de archivos local
- Blob/directorio virtual/URI de contenedor de Azure
- Archivo/directorio/URI de recurso compartido de archivos de Azure
- Directorio/URI de archivo/sistema de archivos de Azure Data Lake Storage Gen2

```azcopy
.\azcopy copy <source path> <destination path> --<flag-name>=<flag-value>
# Using the alias instead 
.\azcopy cp <source path> <destination path> --<flag-name>=<flag-value>
```

El siguiente comando carga todos los archivos en la carpeta `C:\local\path` recursivamente en el contenedor `mycontainer1`, creando `path` directorio en el contenedor. Cuando `--put-md5` se proporciona la marca, AzCopy calcula y almacena el hash de md5 de cada archivo en `Content-md5` propiedad del blob correspondiente para su uso posterior.

```azcopy
.\azcopy cp "C:\local\path" "https://account.blob.core.windows.net/mycontainer1<sastoken>" --recursive=true --put-md5
```

El siguiente comando carga todos los archivos en la carpeta `C:\local\path` (sin recurrencia en los subdirectorios) en el contenedor `mycontainer1`:

```azcopy
.\azcopy cp "C:\local\path\*" "https://account.blob.core.windows.net/mycontainer1<sastoken>" --put-md5
```

Para obtener más ejemplos, use el siguiente comando:

```azcopy
.\azcopy cp -h
```

## <a name="copy-blob-data-between-two-storage-accounts"></a>Copiar datos de blobs entre dos cuentas de almacenamiento

Copiar datos entre dos cuentas de almacenamiento utiliza la [colocar bloque From URL](https://docs.microsoft.com/rest/api/storageservices/put-block-from-url) API y no usa el ancho de banda de red del equipo cliente. Se copian datos entre dos servidores de Azure Storage directamente, mientras que AzCopy simplemente Orquesta la operación de copia. Esta opción solo está disponible para el almacenamiento de blobs.

Para copiar todos los datos entre dos cuentas de almacenamiento de Blob, use el siguiente comando:
```azcopy
.\azcopy cp "https://myaccount.blob.core.windows.net/<sastoken>" "https://myotheraccount.blob.core.windows.net/<sastoken>" --recursive=true
```

Para copiar un contenedor de blobs en otro contenedor de blobs, use el siguiente comando:
```azcopy
.\azcopy cp "https://myaccount.blob.core.windows.net/mycontainer/<sastoken>" "https://myotheraccount.blob.core.windows.net/mycontainer/<sastoken>" --recursive=true
```

## <a name="copy-a-vhd-image-to-a-storage-account"></a>Copia de una imagen VHD en una cuenta de almacenamiento

Forma predeterminada, AzCopy carga datos en blobs en bloques. Para cargar archivos como Blobs en anexos o Blobs en páginas use la marca `--blob-type=[BlockBlob|PageBlob|AppendBlob]`.

```azcopy
.\azcopy cp "C:\local\path\mydisk.vhd" "https://myotheraccount.blob.core.windows.net/mycontainer/mydisk.vhd<sastoken>" --blob-type=PageBlob
```

## <a name="sync-incremental-copy-and-delete-blob-storage-only"></a>Sync: copia y eliminación incrementales (solo Blob Storage)

El comando sync sincroniza el contenido de un directorio de origen en un directorio en el destino, comparar los nombres de archivo y se modificó por última vez las marcas de tiempo. Esta operación incluye la eliminación de archivos de destino opcional si no existe en el origen cuando el `--delete-destination=prompt|true` se proporciona la marca. De forma predeterminada, el comportamiento de eliminación está deshabilitado. 

> [!NOTE] 
> Use el `--delete-destination` marca con cuidado. Habilitar la [eliminación temporal](https://docs.microsoft.com/azure/storage/blobs/storage-blob-soft-delete) características antes de habilitar el comportamiento de eliminación en sincronización para evitar eliminaciones accidentales en su cuenta. 
>
> Cuando `--delete-destination` está establecido en true, AzCopy eliminará los archivos que no existen en el origen de destino sin ningún símbolo del sistema para el usuario. Si desea que se le pida confirmación, utilice `--delete-destination=prompt`.

Para sincronizar su sistema de archivos local en una cuenta de almacenamiento, use el siguiente comando:

```azcopy
.\azcopy sync "C:\local\path" "https://account.blob.core.windows.net/mycontainer1<sastoken>" --recursive=true
```

También puede sincronizar un contenedor de blobs hasta un sistema de archivos local:

```azcopy
# The SAS token isn't required for Azure Active Directory authentication.
.\azcopy sync "https://account.blob.core.windows.net/mycontainer1" "C:\local\path" --recursive=true
```

Este comando sincroniza incrementalmente el origen al destino basándose en las últimas marcas de tiempo modificada. Si agrega o elimina un archivo en el origen, AzCopy hará lo mismo en el destino. Antes de la eliminación, AzCopy le pedirá que confirme la operación.

## <a name="copy-data-from-amazon-web-services-aws-s3"></a>Copiar datos de Amazon Web Services (AWS) S3

Para autenticarse con un cubo de AWS S3, establezca las variables de entorno siguientes:

```
# For Windows:
set AWS_ACCESS_KEY_ID=<your AWS access key>
set AWS_SECRET_ACCESS_KEY=<AWS secret access key>
# For Linux:
export AWS_ACCESS_KEY_ID=<your AWS access key>
export AWS_SECRET_ACCESS_KEY=<AWS secret access key>
# For MacOS
export AWS_ACCESS_KEY_ID=<your AWS access key>
export AWS_SECRET_ACCESS_KEY=<AWS secret access key>
```

Para copiar el depósito en un contenedor de blobs, emita el comando siguiente:

```
.\azcopy cp "https://s3.amazonaws.com/mybucket" "https://myaccount.blob.core.windows.net/mycontainer?<sastoken>" --recursive
```

Para obtener más información acerca de cómo copiar datos desde AWS S3 mediante AzCopy, consulte la página [aquí](https://github.com/Azure/azure-storage-azcopy/wiki/Copy-from-AWS-S3).

## <a name="advanced-configuration"></a>Configuración avanzada

### <a name="configure-proxy-settings"></a>Configuración de los valores de proxy

Para configurar la configuración de proxy para AzCopy v10, establezca el https_proxy variable de entorno mediante el comando siguiente:

```cmd
# For Windows:
set https_proxy=<proxy IP>:<proxy port>
# For Linux:
export https_proxy=<proxy IP>:<proxy port>
# For MacOS
export https_proxy=<proxy IP>:<proxy port>
```

### <a name="optimize-throughput"></a>Optimización del rendimiento

Establezca la variable de entorno AZCOPY_CONCURRENCY_VALUE para configurar el número de solicitudes simultáneas y para controlar el consumo de recursos y el rendimiento de rendimiento. De forma predeterminada, el valor se establece en 300. Si se reduce el valor, limitará el ancho de banda y el valor CPU usados por AzCopy v10.

```cmd
# For Windows:
set AZCOPY_CONCURRENCY_VALUE=<value>
# For Linux:
export AZCOPY_CONCURRENCY_VALUE=<value>
# For MacOS
export AZCOPY_CONCURRENCY_VALUE=<value>
# To check the current value of the variable on all the platforms
.\azcopy env
# If the value is blank then the default value is currently in use
```

### <a name="change-the-location-of-the-log-files"></a>Cambie la ubicación del archivo de registro.

Puede cambiar la ubicación de los archivos de registro si es necesario o para evitar llenar el disco del sistema operativo.

```cmd
# For Windows:
set AZCOPY_LOG_LOCATION=<value>
# For Linux:
export AZCOPY_LOG_LOCATION=<value>
# For MacOS
export AZCOPY_LOG_LOCATION=<value>
# To check the current value of the variable on all the platforms
.\azcopy env
# If the value is blank, then the default value is currently in use
```
### <a name="change-the-default-log-level"></a>Cambiar el nivel de registro predeterminado 

De forma predeterminada, el nivel de registro de AzCopy se establece en INFO. Si desea reducir el nivel de detalle del registro para ahorrar espacio en disco, sobrescriba el valor utilizando la opción ``--log-level``. Los niveles de registro disponibles son: DEPURACIÓN, información, advertencia, ERROR, pánico y FATAL.

### <a name="review-the-logs-for-errors"></a>Revisión de los registros en busca errores

El siguiente comando obtiene todos los errores con el estado UPLOADFAILED desde el registro 04dc9ca9-158f-7945-5933-564021086c79:

```azcopy
cat 04dc9ca9-158f-7945-5933-564021086c79.log | grep -i UPLOADFAILED
```
## <a name="troubleshooting"></a>solución de problemas

AzCopy crea archivos de registro y archivos de plan para cada trabajo. Puede usar los registros para investigar y solucionar los problemas potenciales. Los registros contendrá el estado de error (UPLOADFAILED, COPYFAILED y DOWNLOADFAILED), la ruta de acceso completa y el motivo del error. Los registros de trabajo y los archivos del plan se encuentran en la carpeta %USERPROFILE\\.azcopy en Windows o $HOME\\.azcopy en Mac y Linux.

> [!IMPORTANT]
> Al enviar una solicitud a Microsoft Support (o para solucionar el problema de que impliquen a ningún tercero), compartir la versión censurada del comando que desea ejecutar. Esto garantiza que la SAS accidentalmente no se comparte con nadie. Puede encontrar la versión censurada al principio del archivo de registro.

### <a name="view-and-resume-jobs"></a>Visualización y reanudación de trabajos

Cada operación de transferencia creará un trabajo de AzCopy. Use el siguiente comando para ver el historial de trabajos:

```azcopy
.\azcopy jobs list
```

Para ver las estadísticas del trabajo, use el siguiente comando:

```azcopy
.\azcopy jobs show <job-id>
```

Para filtrar a las transferencias por estado, use el siguiente comando:

```azcopy
.\azcopy jobs show <job-id> --with-status=Failed
```

Use el comando siguiente para reanudar un trabajo de error/cancelado. Este comando usa su identificador, junto con el token de SAS, ya que no es persistente por motivos de seguridad:

```azcopy
.\azcopy jobs resume <jobid> --source-sas="<sastokenhere>"
.\azcopy jobs resume <jobid> --destination-sas="<sastokenhere>"
```

## <a name="next-steps"></a>Pasos siguientes

Si tiene preguntas, problemas o comentarios generales, enviarlas [en GitHub](https://github.com/Azure/azure-storage-azcopy).


