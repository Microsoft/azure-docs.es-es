---
title: azcopy remove | Microsoft Docs
description: En este artículo se proporciona información de referencia del comando azcopy remove.
author: normesta
ms.service: storage
ms.topic: reference
ms.date: 07/24/2020
ms.author: normesta
ms.subservice: common
ms.reviewer: zezha-msft
ms.openlocfilehash: d79b647d216fe28241e5891def574ab598304828
ms.sourcegitcommit: 656c0c38cf550327a9ee10cc936029378bc7b5a2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89078227"
---
# <a name="azcopy-remove"></a>azcopy remove

Elimine blobs o archivos de una cuenta de almacenamiento de Azure.

## <a name="synopsis"></a>Sinopsis

```azcopy
azcopy remove [resourceURL] [flags]
```

## <a name="related-conceptual-articles"></a>Artículos conceptuales relacionados

- [Introducción a AzCopy](storage-use-azcopy-v10.md)
- [Transferencia de datos con AzCopy y Blob Storage](storage-use-azcopy-blobs.md)
- [Transferencia de datos con AzCopy y File Storage](storage-use-azcopy-files.md)
- [Configurar, optimizar y solucionar problemas de AzCopy](storage-use-azcopy-configure.md)

## <a name="examples"></a>Ejemplos

Quite un solo blob con un token de SAS:

```azcopy
azcopy rm "https://[account].blob.core.windows.net/[container]/[path/to/blob]?[SAS]"
```

Quite todo un directorio virtual con un token de SAS:
 
```azcopy
azcopy rm "https://[account].blob.core.windows.net/[container]/[path/to/directory]?[SAS]" --recursive=true
```

Quite solo los blobs dentro de un directorio virtual, pero no quite ningún subdirectorio ni blob dentro de esos subdirectorios:

```azcopy
azcopy rm "https://[account].blob.core.windows.net/[container]/[path/to/virtual/dir]" --recursive=false
```

Quite un subconjunto de blobs de un directorio virtual (por ejemplo, quite solo los archivos .jpg o .pdf o si el nombre del blob es `exactName`):

```azcopy
azcopy rm "https://[account].blob.core.windows.net/[container]/[path/to/directory]?[SAS]" --recursive=true --include-pattern="*.jpg;*.pdf;exactName"
```

Quite un directorio virtual completo, pero con la exclusión de ciertos blobs del ámbito (por ejemplo: todos los blobs que comienzan por "foo" o terminan por "bar"):

```azcopy
azcopy rm "https://[account].blob.core.windows.net/[container]/[path/to/directory]?[SAS]" --recursive=true --exclude-pattern="foo*;*bar"
```

Quite blobs y directorios virtuales específicos colocando sus rutas de acceso relativas (no codificadas como URL) en un archivo:

```azcopy
azcopy rm "https://[account].blob.core.windows.net/[container]/[path/to/parent/dir]" --recursive=true --list-of-files=/usr/bar/list.txt
- file content:
    dir1/dir2
    blob1
    blob2
```
Quite un solo archivo de una cuenta de Blob Storage que tenga un espacio de nombres jerárquico (no se admite incluir/excluir):

```azcopy
azcopy rm "https://[account].dfs.core.windows.net/[container]/[path/to/file]?[SAS]"
```

Quite un solo directorio de una cuenta de Blob Storage que tenga un espacio de nombres jerárquico (no se admite incluir/excluir):

```azcopy
azcopy rm "https://[account].dfs.core.windows.net/[container]/[path/to/directory]?[SAS]"
```

## <a name="options"></a>Opciones

**--delete-snapshots string** De forma predeterminada, se producirá un error en la operación de eliminación si un blob tiene instantáneas. Especifique `include` para quitar el blob raíz y todas sus instantáneas; también puede especificar `only` para quitar solo las instantáneas pero conservar el blob raíz.

**--exclude-path** string   Excluye estas rutas de acceso al quitar. Esta opción no permite caracteres comodín (*). Comprueba el prefijo de ruta de acceso relativa. Por ejemplo: `myFolder;myFolder/subDirName/file.pdf`

**--exclude-pattern** string   Excluye los archivos en los que el nombre coincide con la lista de patrones. Por ejemplo: `*.jpg`;`*.pdf`;`exactName`

**--force-if-read-only**   Al eliminar un archivo o una carpeta de Azure Files, fuerce la eliminación para que funcione incluso si el objeto existente tiene establecido el atributo de solo lectura.

**--help**   Ayuda para la eliminación.

**--include-path** string   Incluye solo estas rutas de acceso al quitar. Esta opción no permite caracteres comodín (*). Comprueba el prefijo de ruta de acceso relativa. Por ejemplo: `myFolder;myFolder/subDirName/file.pdf`

**--include-pattern** string   Incluye solo los archivos en los que el nombre coincide con la lista de patrones. Por ejemplo: *`.jpg`;* `.pdf`;`exactName`

**--list-of-files** string   Define la ubicación de un archivo que contiene la lista de archivos y directorios que se van a eliminar. Las rutas de acceso relativas deben delimitarse mediante saltos de línea y las rutas de acceso NO deben estar codificadas como URL. 

**--list-of-versions** string Especifica un archivo en el que cada identificador de versión aparece en una línea independiente. Asegúrese de que el origen apunte a un único blob y que todos los identificadores de versión especificados en el archivo con esta marca solo pertenezcan al blob de origen. Los identificadores de versión especificados del blob determinado se eliminarán de Azure Storage. 

**--log-level** string   Define el nivel de detalle del registro para el archivo de registro. Los niveles disponibles son: `INFO`(todas las solicitudes/respuestas), `WARNING`(respuestas lentas), `ERROR`(solo solicitudes con errores) y `NONE`(sin registros de salida). (`INFO` predeterminado) (`INFO` predeterminado)

**--recursive**   Busca en los subdirectorios de forma recursiva al sincronizar entre directorios.

## <a name="options-inherited-from-parent-commands"></a>Opciones heredadas de comandos primarios

|Opción|Descripción|
|---|---|
|--cap-mbps número de punto flotante|Limita la velocidad de transferencia, en megabits por segundo. El rendimiento en un momento dado puede variar ligeramente del límite. Si esta opción se establece en cero o se omite, el rendimiento no se limita.|
|--output-type string|Formato de la salida del comando. Las opciones incluyen: text, json. El valor predeterminado es "text".|
|--trusted-microsoft-suffixes string   |Especifica sufijos de dominio adicionales en los que se pueden enviar tokens de inicio de sesión de Azure Active Directory.  El valor predeterminado es " *.core.windows.net;* .core.chinacloudapi.cn; *.core.cloudapi.de;* .core.usgovcloudapi.net". Los valores que se muestran aquí se agregan al valor predeterminado. Por seguridad, solo debe poner aquí dominios de Microsoft Azure. Separe las entradas con punto y coma.|

## <a name="see-also"></a>Consulte también

- [azcopy](storage-ref-azcopy.md)
