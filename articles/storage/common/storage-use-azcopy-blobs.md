---
title: Transferencia de datos con Azure Blob Storage como origen o destino mediante AzCopy v10 | Microsoft Docs
description: Este artículo contiene una colección de comandos de ejemplo de AzCopy que le ayudarán a crear contenedores, copiar archivos y sincronizar directorios entre sistemas de archivos y contenedores locales.
author: normesta
ms.service: storage
ms.topic: how-to
ms.date: 07/27/2020
ms.author: normesta
ms.subservice: common
ms.reviewer: dineshm
ms.openlocfilehash: f4bf3974cd561626c280dc65aa5fc78d0c9a159b
ms.sourcegitcommit: 269da970ef8d6fab1e0a5c1a781e4e550ffd2c55
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/10/2020
ms.locfileid: "88056506"
---
# <a name="transfer-data-with-azcopy-and-blob-storage"></a>Transferencia de datos con AzCopy y Blob Storage

AzCopy es una utilidad de línea de comandos que puede usar para copiar datos entre cuentas de almacenamiento. Este artículo contiene comandos de ejemplo que funcionan con Blob Storage.

> [!TIP]
> En los ejemplos de este artículo se delimitan los argumentos de ruta de acceso con comillas simples (''). Use comillas simples en todos los shells de comandos excepto en el shell de comandos de Windows (cmd.exe). Si usa un shell de comandos de Windows (cmd.exe), incluya los argumentos de la ruta de acceso entre comillas dobles ("") en lugar de comillas simples ('').

## <a name="get-started"></a>Introducción

Vea el artículo [Introducción a AzCopy](storage-use-azcopy-v10.md) para descargar AzCopy y obtener información sobre las formas de proporcionar credenciales de autorización para el servicio de almacenamiento.

> [!NOTE]
> En los ejemplos de este artículo se supone que ha autenticado la identidad mediante el comando `AzCopy login`. Después, AzCopy usa la cuenta de Azure AD para autorizar el acceso a los datos en Blob Storage.
>
> Si prefiere usar un token de SAS para autorizar el acceso a los datos de blob, puede anexar ese token a la dirección URL de recursos en cada comando AzCopy.
>
> Por ejemplo: `'https://<storage-account-name>.blob.core.windows.net/<container-name><SAS-token>'`.

## <a name="create-a-container"></a>Crear un contenedor

Para crear un contenedor puede usar el comando [azcopy make](storage-ref-azcopy-make.md). En los ejemplos de esta sección se crea un contenedor denominado `mycontainer`.

|    |     |
|--------|-----------|
| **Sintaxis** | `azcopy make 'https://<storage-account-name>.<blob or dfs>.core.windows.net/<container-name>'` |
| **Ejemplo** | `azcopy make 'https://mystorageaccount.blob.core.windows.net/mycontainer'` |
| **Ejemplo** (espacio de nombres jerárquico) | `azcopy make 'https://mystorageaccount.dfs.core.windows.net/mycontainer'` |

Para obtener documentos de referencia detallados, consulte [azcopy make](storage-ref-azcopy-make.md).

## <a name="upload-files"></a>Carga de archivos

Puede usar el comando [azcopy copy](storage-ref-azcopy-copy.md) para cargar archivos y directorios desde el equipo local.

En esta sección se incluyen los ejemplos siguientes:

> [!div class="checklist"]
> * Cargar un archivo
> * Subir un directorio
> * Subir el contenido de un directorio 
> * Carga de archivos específicos

> [!TIP]
> Puede modificar las operaciones de carga mediante marcas opcionales. Estos son algunos ejemplos.
>
> |Escenario|Marca|
> |---|---|
> |Cargar los archivos como blobs en anexos o blobs en páginas.|**--blob-type**=\[BlockBlob\|PageBlob\|AppendBlob\]|
> |Cargar en un nivel de acceso específico (como el nivel de archivo).|**--block-blob-tier**=\[None\|Hot\|Cool\|Archive\]|
> 
> Para obtener una lista completa, vea las [opciones](storage-ref-azcopy-copy.md#options).

### <a name="upload-a-file"></a>Cargar un archivo

|    |     |
|--------|-----------|
| **Sintaxis** | `azcopy copy '<local-file-path>' 'https://<storage-account-name>.<blob or dfs>.core.windows.net/<container-name>/<blob-name>'` |
| **Ejemplo** | `azcopy copy 'C:\myDirectory\myTextFile.txt' 'https://mystorageaccount.blob.core.windows.net/mycontainer/myTextFile.txt'` |
| **Ejemplo** (espacio de nombres jerárquico) | `azcopy copy 'C:\myDirectory\myTextFile.txt' 'https://mystorageaccount.dfs.core.windows.net/mycontainer/myTextFile.txt'` |

También puede cargar un archivo mediante un símbolo comodín (*) en cualquier lugar de la ruta de acceso o del nombre de archivo. Por ejemplo: `'C:\myDirectory\*.txt'` o `C:\my*\*.txt`.

### <a name="upload-a-directory"></a>Subir un directorio

En este ejemplo se copia un directorio (y todos sus archivos) en un contenedor de blobs. El resultado es un directorio en el contenedor con el mismo nombre.

|    |     |
|--------|-----------|
| **Sintaxis** | `azcopy copy '<local-directory-path>' 'https://<storage-account-name>.<blob or dfs>.core.windows.net/<container-name>' --recursive` |
| **Ejemplo** | `azcopy copy 'C:\myDirectory' 'https://mystorageaccount.blob.core.windows.net/mycontainer' --recursive` |
| **Ejemplo** (espacio de nombres jerárquico) | `azcopy copy 'C:\myDirectory' 'https://mystorageaccount.dfs.core.windows.net/mycontainer' --recursive` |

Para copiar un directorio dentro el contenedor, simplemente especifique el nombre de ese directorio en la cadena de comandos.

|    |     |
|--------|-----------|
| **Ejemplo** | `azcopy copy 'C:\myDirectory' 'https://mystorageaccount.blob.core.windows.net/mycontainer/myBlobDirectory' --recursive` |
| **Ejemplo** (espacio de nombres jerárquico) | `azcopy copy 'C:\myDirectory' 'https://mystorageaccount.dfs.core.windows.net/mycontainer/myBlobDirectory' --recursive` |

Si especifica el nombre de un directorio que no existe en el contenedor, AzCopy crea un directorio con ese nombre.

### <a name="upload-the-contents-of-a-directory"></a>Subir el contenido de un directorio

Puede cargar el contenido de un directorio sin copiar el propio directorio contenedor mediante el carácter comodín (*).

|    |     |
|--------|-----------|
| **Sintaxis** | `azcopy copy '<local-directory-path>\*' 'https://<storage-account-name>.<blob or dfs>.core.windows.net/<container-name>/<directory-path>'` |
| **Ejemplo** | `azcopy copy 'C:\myDirectory\*' 'https://mystorageaccount.blob.core.windows.net/mycontainer/myBlobDirectory'` |
| **Ejemplo** (espacio de nombres jerárquico) | `azcopy copy 'C:\myDirectory\*' 'https://mystorageaccount.dfs.core.windows.net/mycontainer/myBlobDirectory'` |

> [!NOTE]
> Anexe la marca `--recursive` para cargar archivos en todos los subdirectorios.

### <a name="upload-specific-files"></a>Carga de archivos específicos

Puede cargar archivos específicos mediante nombres de archivo completos, nombres parciales con caracteres comodín (*) o fechas y horas.

#### <a name="specify-multiple-complete-file-names"></a>Especificación de varios nombres de archivo completos

Use el comando [azcopy copy](storage-ref-azcopy-copy.md) con la opción `--include-path`. Separe los nombres de archivo individuales mediante punto y coma (`;`).

|    |     |
|--------|-----------|
| **Sintaxis** | `azcopy copy '<local-directory-path>' 'https://<storage-account-name>.<blob or dfs>.core.windows.net/<container-name>' --include-path <semicolon-separated-file-list>` |
| **Ejemplo** | `azcopy copy 'C:\myDirectory' 'https://mystorageaccount.blob.core.windows.net/mycontainer' --include-path 'photos;documents\myFile.txt' --recursive` |
| **Ejemplo** (espacio de nombres jerárquico) | `azcopy copy 'C:\myDirectory' 'https://mystorageaccount.dfs.core.windows.net/mycontainer' --include-path 'photos;documents\myFile.txt' --recursive` |

En este ejemplo, AzCopy transfiere el directorio `C:\myDirectory\photos` y el archivo `C:\myDirectory\documents\myFile.txt`. Debe incluir la opción `--recursive` para transferir todos los archivos del directorio `C:\myDirectory\photos`.

También puede excluir archivos mediante la opción `--exclude-path`. Para más información, consulte los documentos de referencia de [azcopy copy ](storage-ref-azcopy-copy.md).

#### <a name="use-wildcard-characters"></a>Uso de caracteres comodín

Use el comando [azcopy copy](storage-ref-azcopy-copy.md) con la opción `--include-pattern`. Especifique nombres parciales que incluyan los caracteres comodín. Separe los nombres con punto y coma (`;`). 

|    |     |
|--------|-----------|
| **Sintaxis** | `azcopy copy '<local-directory-path>' 'https://<storage-account-name>.<blob or dfs>.core.windows.net/<container-name>' --include-pattern <semicolon-separated-file-list-with-wildcard-characters>` |
| **Ejemplo** | `azcopy copy 'C:\myDirectory' 'https://mystorageaccount.blob.core.windows.net/mycontainer' --include-pattern 'myFile*.txt;*.pdf*'` |
| **Ejemplo** (espacio de nombres jerárquico) | `azcopy copy 'C:\myDirectory' 'https://mystorageaccount.dfs.core.windows.net/mycontainer' --include-pattern 'myFile*.txt;*.pdf*'` |

También puede excluir archivos mediante la opción `--exclude-pattern`. Para más información, consulte los documentos de referencia de [azcopy copy ](storage-ref-azcopy-copy.md).

Las opciones `--include-pattern` y `--exclude-pattern` solo se aplican a los nombres de archivo, no a la ruta de acceso.  Si quiere copiar todos los archivos de texto que existen en un árbol de directorios, use la opción `–recursive` para obtener todo el árbol de directorios y, a continuación, use el `–include-pattern` y especifique `*.txt` para obtener todos los archivos de texto.

#### <a name="upload-files-that-were-modified-after-a-date-and-time"></a>Carga de archivos modificados después de una fecha y hora 

Use el comando [azcopy copy](storage-ref-azcopy-copy.md) con la opción `--include-after`. Especifique una fecha y hora en formato ISO-8601 (por ejemplo: `2020-08-19T15:04:00Z`). 

|    |     |
|--------|-----------|
| **Sintaxis** | `azcopy copy '<local-directory-path>\*' 'https://<storage-account-name>.<blob or dfs>.core.windows.net/<container-or-directory-name>'  --include-after <Date-Time-in-ISO-8601-format>` |
| **Ejemplo** | `azcopy copy 'C:\myDirectory\*' 'https://mystorageaccount.blob.core.windows.net/mycontainer/FileDirectory'  --include-after '2020-08-19T15:04:00Z'` |
| **Ejemplo** (espacio de nombres jerárquico) | `azcopy copy 'C:\myDirectory\*' 'https://mystorageaccount.dfs.core.windows.net/mycontainer/FileDirectory'   --include-after '2020-08-19T15:04:00Z'` |

Para ver una referencia detallada, consulte la documentación de referencia de [azcopy copy](storage-ref-azcopy-copy.md).

## <a name="download-files"></a>Descarga de archivos

Puede usar el comando [azcopy copy](storage-ref-azcopy-copy.md) para descargar blobs, directorios y contenedores en un equipo local.

En esta sección se incluyen los ejemplos siguientes:

> [!div class="checklist"]
> * Descarga de un archivo
> * Descargar un directorio
> * Descargar el contenido de un directorio
> * Descarga de archivos específicos

> [!TIP]
> Puede modificar las operaciones de descarga mediante marcas opcionales. Estos son algunos ejemplos.
>
> |Escenario|Marca|
> |---|---|
> |Descomprimir archivos automáticamente.|**DECOMPRESS**|
> |Especifique el grado de detalles que quiere que sean las entradas de registro relacionadas con la copia.|**--log-level**=\[WARNING\|ERROR\|INFO\|NONE\]|
> |Especifique si desea sobrescribir los archivos y blobs en conflicto en el destino, y cómo hacerlo.|**--overwrite**=\[true\|false\|ifSourceNewer\|prompt\]|
> 
> Para obtener una lista completa, vea las [opciones](storage-ref-azcopy-copy.md#options).

> [!NOTE]
> Si el valor de la propiedad `Content-md5` de un blob contiene un hash, AzCopy calcula un hash MD5 para los datos descargados y comprueba que el hash MD5 almacenado en la propiedad `Content-md5` del blob coincide con el hash calculado. Si estos valores no coinciden, se produce un error en la descarga a menos que invalide este comportamiento mediante la anexión de `--check-md5=NoCheck` o `--check-md5=LogOnly` al comando de copia.

### <a name="download-a-file"></a>Descarga de un archivo

|    |     |
|--------|-----------|
| **Sintaxis** | `azcopy copy 'https://<storage-account-name>.<blob or dfs>.core.windows.net/<container-name>/<blob-path>' '<local-file-path>'` |
| **Ejemplo** | `azcopy copy 'https://mystorageaccount.blob.core.windows.net/mycontainer/myTextFile.txt' 'C:\myDirectory\myTextFile.txt'` |
| **Ejemplo** (espacio de nombres jerárquico) | `azcopy copy 'https://mystorageaccount.dfs.core.windows.net/mycontainer/myTextFile.txt' 'C:\myDirectory\myTextFile.txt'` |

### <a name="download-a-directory"></a>Descargar un directorio

|    |     |
|--------|-----------|
| **Sintaxis** | `azcopy copy 'https://<storage-account-name>.<blob or dfs>.core.windows.net/<container-name>/<directory-path>' '<local-directory-path>' --recursive` |
| **Ejemplo** | `azcopy copy 'https://mystorageaccount.blob.core.windows.net/mycontainer/myBlobDirectory' 'C:\myDirectory'  --recursive` |
| **Ejemplo** (espacio de nombres jerárquico) | `azcopy copy 'https://mystorageaccount.dfs.core.windows.net/mycontainer/myBlobDirectory' 'C:\myDirectory'  --recursive` |

En este ejemplo se crea un directorio denominado `C:\myDirectory\myBlobDirectory` que contiene todos los archivos descargados.

### <a name="download-the-contents-of-a-directory"></a>Descargar el contenido de un directorio

Puede descargar el contenido de un directorio sin copiar el propio directorio contenedor mediante el carácter comodín (*).

> [!NOTE]
> En la actualidad, este escenario solo se admite para las cuentas que no tienen un espacio de nombres jerárquico.

|    |     |
|--------|-----------|
| **Sintaxis** | `azcopy copy 'https://<storage-account-name>.blob.core.windows.net/<container-name>/*' '<local-directory-path>/'` |
| **Ejemplo** | `azcopy copy 'https://mystorageaccount.blob.core.windows.net/mycontainer/myBlobDirectory/*' 'C:\myDirectory'` |

> [!NOTE]
> Anexe la marca `--recursive` para descargar archivos en todos los subdirectorios.

### <a name="download-specific-files"></a>Descarga de archivos específicos

Puede descargar archivos específicos mediante nombres de archivo completos, nombres parciales con caracteres comodín (*) o fechas y horas. 

#### <a name="specify-multiple-complete-file-names"></a>Especificación de varios nombres de archivo completos

Use el comando [azcopy copy](storage-ref-azcopy-copy.md) con la opción `--include-path`. Separe los nombres de archivo individuales mediante un punto y coma (`;`).

|    |     |
|--------|-----------|
| **Sintaxis** | `azcopy copy 'https://<storage-account-name>.<blob or dfs>.core.windows.net/<container-or-directory-name>' '<local-directory-path>'  --include-path <semicolon-separated-file-list>` |
| **Ejemplo** | `azcopy copy 'https://mystorageaccount.blob.core.windows.net/mycontainer/FileDirectory' 'C:\myDirectory'  --include-path 'photos;documents\myFile.txt' --recursive` |
| **Ejemplo** (espacio de nombres jerárquico) | `azcopy copy 'https://mystorageaccount.dfs.core.windows.net/mycontainer/FileDirectory' 'C:\myDirectory'  --include-path 'photos;documents\myFile.txt'--recursive` |

En este ejemplo, AzCopy transfiere el directorio `https://mystorageaccount.blob.core.windows.net/mycontainer/FileDirectory/photos` y el archivo `https://mystorageaccount.blob.core.windows.net/mycontainer/FileDirectory/documents/myFile.txt`. Debe incluir la opción `--recursive` para transferir todos los archivos del directorio `https://mystorageaccount.blob.core.windows.net/mycontainer/FileDirectory/photos`.

También puede excluir archivos mediante la opción `--exclude-path`. Para más información, consulte los documentos de referencia de [azcopy copy ](storage-ref-azcopy-copy.md).

#### <a name="use-wildcard-characters"></a>Uso de caracteres comodín

Use el comando [azcopy copy](storage-ref-azcopy-copy.md) con la opción `--include-pattern`. Especifique nombres parciales que incluyan los caracteres comodín. Separe los nombres con punto y coma (`;`).

|    |     |
|--------|-----------|
| **Sintaxis** | `azcopy copy 'https://<storage-account-name>.<blob or dfs>.core.windows.net/<container-or-directory-name>' '<local-directory-path>' --include-pattern <semicolon-separated-file-list-with-wildcard-characters>` |
| **Ejemplo** | `azcopy copy 'https://mystorageaccount.blob.core.windows.net/mycontainer/FileDirectory' 'C:\myDirectory'  --include-pattern 'myFile*.txt;*.pdf*'` |
| **Ejemplo** (espacio de nombres jerárquico) | `azcopy copy 'https://mystorageaccount.dfs.core.windows.net/mycontainer/FileDirectory' 'C:\myDirectory'  --include-pattern 'myFile*.txt;*.pdf*'` |

También puede excluir archivos mediante la opción `--exclude-pattern`. Para más información, consulte los documentos de referencia de [azcopy copy ](storage-ref-azcopy-copy.md).

Las opciones `--include-pattern` y `--exclude-pattern` solo se aplican a los nombres de archivo, no a la ruta de acceso.  Si quiere copiar todos los archivos de texto que existen en un árbol de directorios, use la opción `–recursive` para obtener todo el árbol de directorios y, a continuación, use el `–include-pattern` y especifique `*.txt` para obtener todos los archivos de texto.

#### <a name="download-files-that-were-modified-after-a-date-and-time"></a>Descarga de archivos modificados después de una fecha y hora 

Use el comando [azcopy copy](storage-ref-azcopy-copy.md) con la opción `--include-after`. Especifique una fecha y hora en formato ISO-8601 (por ejemplo: `2020-08-19T15:04:00Z`). 

|    |     |
|--------|-----------|
| **Sintaxis** | `azcopy copy 'https://<storage-account-name>.<blob or dfs>.core.windows.net/<container-or-directory-name>/*' '<local-directory-path>' --include-after <Date-Time-in-ISO-8601-format>` |
| **Ejemplo** | `azcopy copy 'https://mystorageaccount.blob.core.windows.net/mycontainer/FileDirectory/*' 'C:\myDirectory'  --include-after '2020-08-19T15:04:00Z'` |
| **Ejemplo** (espacio de nombres jerárquico) | `azcopy copy 'https://mystorageaccount.dfs.core.windows.net/mycontainer/FileDirectory/*' 'C:\myDirectory'  --include-after '2020-08-19T15:04:00Z'` |

Para ver una referencia detallada, consulte la documentación de referencia de [azcopy copy](storage-ref-azcopy-copy.md).

## <a name="copy-blobs-between-storage-accounts"></a>Copia de blobs entre cuentas de almacenamiento

Puede usar AzCopy para copiar blobs a otras cuentas de almacenamiento. La operación de copia es sincrónica, por lo que cuando el comando devuelve un resultado, eso indica que se han copiado todos los archivos. 

AzCopy usa interfaces [API](https://docs.microsoft.com/rest/api/storageservices/put-page-from-url)[de servidor a servidor](https://docs.microsoft.com/rest/api/storageservices/put-block-from-url), por lo que los datos se copian directamente entre servidores de almacenamiento. En estas operaciones de copia no se usa el ancho de banda de red del equipo. Para aumentar el rendimiento de estas operaciones puede establecer el valor de la variable de entorno `AZCOPY_CONCURRENCY_VALUE`. Para obtener más información, consulte [Optimización del rendimiento](storage-use-azcopy-configure.md#optimize-throughput).

> [!NOTE]
> Este escenario tiene las siguientes limitaciones en la versión actual.
>
> - Tiene que anexar un token de SAS a cada dirección URL de origen. Si proporciona credenciales de autorización mediante Azure Active Directory (AD), puede omitir el token de SAS solo de la dirección URL de destino. Asegúrese de que ha configurado los roles adecuados en su cuenta de destino. Consulte [Opción 1: Uso de Azure Active Directory](storage-use-azcopy-v10.md?toc=/azure/storage/blobs/toc.json#option-1-use-azure-active-directory).
>-  Las cuentas de almacenamiento de blobs en bloques Premium no admiten niveles de acceso. Omita el nivel de acceso de un blob de la operación de copia. Para ello, establezca `s2s-preserve-access-tier` en `false` (por ejemplo: `--s2s-preserve-access-tier=false`).

En esta sección se incluyen los ejemplos siguientes:

> [!div class="checklist"]
> * Copia de un blob a otra cuenta de almacenamiento
> * Copia de un directorio a otra cuenta de almacenamiento
> * Copia de un contenedor a otra cuenta de almacenamiento
> * Copia de todos los contenedores, directorios y archivos a otra cuenta de almacenamiento

Estos ejemplos también funcionan con las cuentas que tienen un espacio de nombres jerárquico. El [acceso multiprotocolo en Data Lake Storage](../blobs/data-lake-storage-multi-protocol-access.md) le permite usar la misma sintaxis de URL (`blob.core.windows.net`) en esas cuentas.

> [!TIP]
> Puede modificar las operaciones de copia mediante marcas opcionales. Estos son algunos ejemplos.
>
> |Escenario|Marca|
> |---|---|
> |Copie blobs como blobs en bloques, en páginas o en anexos.|**--blob-type**=\[BlockBlob\|PageBlob\|AppendBlob\]|
> |Copiar en un nivel de acceso específico (como el nivel de archivo).|**--block-blob-tier**=\[None\|Hot\|Cool\|Archive\]|
> |Descomprimir archivos automáticamente.|**--decompress**=\[gzip\|deflate\]|
> 
> Para obtener una lista completa, vea las [opciones](storage-ref-azcopy-copy.md#options).

### <a name="copy-a-blob-to-another-storage-account"></a>Copia de un blob a otra cuenta de almacenamiento

Use la misma sintaxis de URL (`blob.core.windows.net`) para las cuentas que tienen un espacio de nombres jerárquico.

|    |     |
|--------|-----------|
| **Sintaxis** | `azcopy copy 'https://<source-storage-account-name>.blob.core.windows.net/<container-name>/<blob-path><SAS-token>' 'https://<destination-storage-account-name>.blob.core.windows.net/<container-name>/<blob-path>'` |
| **Ejemplo** | `azcopy copy 'https://mysourceaccount.blob.core.windows.net/mycontainer/myTextFile.txt?sv=2018-03-28&ss=bfqt&srt=sco&sp=rwdlacup&se=2019-07-04T05:30:08Z&st=2019-07-03T21:30:08Z&spr=https&sig=CAfhgnc9gdGktvB=ska7bAiqIddM845yiyFwdMH481QA8%3D' 'https://mydestinationaccount.blob.core.windows.net/mycontainer/myTextFile.txt'` |
| **Ejemplo** (espacio de nombres jerárquico) | `azcopy copy 'https://mysourceaccount.blob.core.windows.net/mycontainer/myTextFile.txt?sv=2018-03-28&ss=bfqt&srt=sco&sp=rwdlacup&se=2019-07-04T05:30:08Z&st=2019-07-03T21:30:08Z&spr=https&sig=CAfhgnc9gdGktvB=ska7bAiqIddM845yiyFwdMH481QA8%3D' 'https://mydestinationaccount.blob.core.windows.net/mycontainer/myTextFile.txt'` |

### <a name="copy-a-directory-to-another-storage-account"></a>Copia de un directorio a otra cuenta de almacenamiento

Use la misma sintaxis de URL (`blob.core.windows.net`) para las cuentas que tienen un espacio de nombres jerárquico.

|    |     |
|--------|-----------|
| **Sintaxis** | `azcopy copy 'https://<source-storage-account-name>.blob.core.windows.net/<container-name>/<directory-path><SAS-token>' 'https://<destination-storage-account-name>.blob.core.windows.net/<container-name>' --recursive` |
| **Ejemplo** | `azcopy copy 'https://mysourceaccount.blob.core.windows.net/mycontainer/myBlobDirectory?sv=2018-03-28&ss=bfqt&srt=sco&sp=rwdlacup&se=2019-07-04T05:30:08Z&st=2019-07-03T21:30:08Z&spr=https&sig=CAfhgnc9gdGktvB=ska7bAiqIddM845yiyFwdMH481QA8%3D' 'https://mydestinationaccount.blob.core.windows.net/mycontainer' --recursive` |
| **Ejemplo** (espacio de nombres jerárquico)| `azcopy copy 'https://mysourceaccount.blob.core.windows.net/mycontainer/myBlobDirectory?sv=2018-03-28&ss=bfqt&srt=sco&sp=rwdlacup&se=2019-07-04T05:30:08Z&st=2019-07-03T21:30:08Z&spr=https&sig=CAfhgnc9gdGktvB=ska7bAiqIddM845yiyFwdMH481QA8%3D' 'https://mydestinationaccount.blob.core.windows.net/mycontainer' --recursive` |

### <a name="copy-a-container-to-another-storage-account"></a>Copia de un contenedor a otra cuenta de almacenamiento

Use la misma sintaxis de URL (`blob.core.windows.net`) para las cuentas que tienen un espacio de nombres jerárquico.

|    |     |
|--------|-----------|
| **Sintaxis** | `azcopy copy 'https://<source-storage-account-name>.blob.core.windows.net/<container-name><SAS-token>' 'https://<destination-storage-account-name>.blob.core.windows.net/<container-name>' --recursive` |
| **Ejemplo** | `azcopy copy 'https://mysourceaccount.blob.core.windows.net/mycontainer?sv=2018-03-28&ss=bfqt&srt=sco&sp=rwdlacup&se=2019-07-04T05:30:08Z&st=2019-07-03T21:30:08Z&spr=https&sig=CAfhgnc9gdGktvB=ska7bAiqIddM845yiyFwdMH481QA8%3D' 'https://mydestinationaccount.blob.core.windows.net/mycontainer' --recursive` |
| **Ejemplo** (espacio de nombres jerárquico)| `azcopy copy 'https://mysourceaccount.blob.core.windows.net/mycontainer?sv=2018-03-28&ss=bfqt&srt=sco&sp=rwdlacup&se=2019-07-04T05:30:08Z&st=2019-07-03T21:30:08Z&spr=https&sig=CAfhgnc9gdGktvB=ska7bAiqIddM845yiyFwdMH481QA8%3D' 'https://mydestinationaccount.blob.core.windows.net/mycontainer' --recursive` |

### <a name="copy-all-containers-directories-and-blobs-to-another-storage-account"></a>Copia de todos los contenedores, directorios y blobs en otra cuenta de almacenamiento

Use la misma sintaxis de URL (`blob.core.windows.net`) para las cuentas que tienen un espacio de nombres jerárquico.

|    |     |
|--------|-----------|
| **Sintaxis** | `azcopy copy 'https://<source-storage-account-name>.blob.core.windows.net/<SAS-token>' 'https://<destination-storage-account-name>.blob.core.windows.net/' --recursive` |
| **Ejemplo** | `azcopy copy 'https://mysourceaccount.blob.core.windows.net/?sv=2018-03-28&ss=bfqt&srt=sco&sp=rwdlacup&se=2019-07-04T05:30:08Z&st=2019-07-03T21:30:08Z&spr=https&sig=CAfhgnc9gdGktvB=ska7bAiqIddM845yiyFwdMH481QA8%3D' 'https://mydestinationaccount.blob.core.windows.net' --recursive` |
| **Ejemplo** (espacio de nombres jerárquico)| `azcopy copy 'https://mysourceaccount.blob.core.windows.net/?sv=2018-03-28&ss=bfqt&srt=sco&sp=rwdlacup&se=2019-07-04T05:30:08Z&st=2019-07-03T21:30:08Z&spr=https&sig=CAfhgnc9gdGktvB=ska7bAiqIddM845yiyFwdMH481QA8%3D' 'https://mydestinationaccount.blob.core.windows.net' --recursive` |

## <a name="synchronize-files"></a>Sincronización de archivos

Puede sincronizar el contenido de un sistema de archivos local con un contenedor de blobs. También puede sincronizar los contenedores y directorios virtuales entre sí. La sincronización es unidireccional. En otras palabras, tendrá que elegir cuál de estos dos puntos de conexión es el origen y cuál es el destino. La sincronización también usa las API de servidor a servidor. Los ejemplos que se presentan en esta sección también funcionan con las cuentas que tienen un espacio de nombres jerárquico. 

> [!NOTE]
> La versión actual de AzCopy no sincroniza entre otros orígenes y destinos (por ejemplo: Almacenamiento de archivos o cubos de Amazon Web Services (AWS) S3).

El comando [sync](storage-ref-azcopy-sync.md) compara nombres de archivo y las marcas de tiempo de la última modificación. Establezca la marca opcional `--delete-destination` en un valor de `true` o `prompt` para eliminar archivos en el directorio de destino si esos archivos ya no existen en el directorio de origen.

Si establece la marca `--delete-destination` en `true`, AzCopy elimina los archivos sin proporcionar un mensaje. Si quiere que aparezca un mensaje antes de que AzCopy elimine un archivo, establezca la marca `--delete-destination` en `prompt`.

> [!NOTE]
> Para evitar eliminaciones accidentales, asegúrese de habilitar la característica de [eliminación temporal](https://docs.microsoft.com/azure/storage/blobs/storage-blob-soft-delete) antes de usar la marca `--delete-destination=prompt|true`.

> [!TIP]
> Puede modificar las operaciones de sincronización mediante marcas opcionales. Estos son algunos ejemplos.
>
> |Escenario|Marca|
> |---|---|
> |Especificar cómo de estrictamente se deben validar los hashes MD5 al descargarse.|**--check-md5**=\[NoCheck\|LogOnly\|FailIfDifferent\|FailIfDifferentOrMissing\]|
> |Excluir archivos en función de un patrón.|**--exclude-path**|
> |Especifique el grado de detalles que quiere que sean las entradas de registro relacionadas con la sincronización.|**--log-level**=\[WARNING\|ERROR\|INFO\|NONE\]|
> 
> Para obtener una lista completa, vea las [opciones](storage-ref-azcopy-sync.md#options).

### <a name="update-a-container-with-changes-to-a-local-file-system"></a>Actualización de un contenedor con los cambios realizados en un sistema de archivos local

En este caso, el contenedor es el destino y el sistema de archivos local es el origen. 

|    |     |
|--------|-----------|
| **Sintaxis** | `azcopy sync '<local-directory-path>' 'https://<storage-account-name>.blob.core.windows.net/<container-name>' --recursive` |
| **Ejemplo** | `azcopy sync 'C:\myDirectory' 'https://mystorageaccount.blob.core.windows.net/mycontainer' --recursive` |

### <a name="update-a-local-file-system-with-changes-to-a-container"></a>Actualización de un sistema de archivos local con los cambios realizados en un contenedor

En este caso, el sistema de archivos local es el destino y el contenedor es el origen.

|    |     |
|--------|-----------|
| **Sintaxis** | `azcopy sync 'https://<storage-account-name>.blob.core.windows.net/<container-name>' 'C:\myDirectory' --recursive` |
| **Ejemplo** | `azcopy sync 'https://mystorageaccount.blob.core.windows.net/mycontainer' 'C:\myDirectory' --recursive` |

### <a name="update-a-container-with-changes-in-another-container"></a>Actualización de un contenedor con cambios en otro contenedor

El primer contenedor que aparece en este comando es el contenedor origen. El segundo es el destino.

|    |     |
|--------|-----------|
| **Sintaxis** | `azcopy sync 'https://<source-storage-account-name>.blob.core.windows.net/<container-name>' 'https://<destination-storage-account-name>.blob.core.windows.net/<container-name>' --recursive` |
| **Ejemplo** | `azcopy sync 'https://mysourceaccount.blob.core.windows.net/mycontainer' 'https://mydestinationaccount.blob.core.windows.net/mycontainer' --recursive` |

### <a name="update-a-directory-with-changes-to-a-directory-in-another-file-share"></a>Actualización de un directorio con cambios en un directorio de otro recurso compartido de archivos

El primer directorio que aparece en este comando es el origen. El segundo es el destino.

|    |     |
|--------|-----------|
| **Sintaxis** | `azcopy sync 'https://<source-storage-account-name>.blob.core.windows.net/<container-name>/<directory-name>' 'https://<destination-storage-account-name>.blob.core.windows.net/<container-name>/<directory-name>' --recursive` |
| **Ejemplo** | `azcopy sync 'https://mysourceaccount.blob.core.windows.net/<container-name>/myDirectory' 'https://mydestinationaccount.blob.core.windows.net/mycontainer/myDirectory' --recursive` |

## <a name="next-steps"></a>Pasos siguientes

Puede encontrar más ejemplos en cualquiera de estos artículos:

- [Introducción a AzCopy](storage-use-azcopy-v10.md)

- [Tutorial: Migración de datos locales al almacenamiento en la nube mediante AzCopy](storage-use-azcopy-migrate-on-premises-data.md)

- [Transferencia de datos con AzCopy y File Storage](storage-use-azcopy-files.md)

- [Transferencia de datos con AzCopy y cubos de Amazon S3](storage-use-azcopy-s3.md)

- [Configurar, optimizar y solucionar problemas de AzCopy](storage-use-azcopy-configure.md)
