---
title: 'Ingesta de datos de Azure HPC Cache: msrsync'
description: Cómo usar msrsync para mover datos a un destino de Blob Storage en Azure HPC Cache
author: ekpgh
ms.service: hpc-cache
ms.topic: how-to
ms.date: 10/30/2019
ms.author: v-erkel
ms.openlocfilehash: 323ecd6a2dd001c3c8df1b2ec15f0ae8402ec70c
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87092412"
---
# <a name="azure-hpc-cache-data-ingest---msrsync-method"></a>Ingesta de datos de Azure HPC Cache: método msrsync

En este artículo se proporcionan instrucciones detalladas sobre el uso de la utilidad ``msrsync`` para copiar datos en un contenedor de Azure Blob Storage y usarlo con Azure HPC Cache.

Para más información sobre cómo mover datos a Blob Storage para Azure HPC Cache, consulte [Traslado de datos a Azure Blob Storage](hpc-cache-ingest.md).

La herramienta ``msrsync`` se puede usar para mover datos a un destino de almacenamiento de back-end para Azure HPC Cache. Además, esta herramienta está diseñada para optimizar el uso del ancho de banda ejecutando varios procesos de tipo ``rsync`` en paralelo. Está disponible en GitHub en https://github.com/jbd/msrsync.

``msrsync`` divide el directorio de origen en "cubos" separados y luego ejecuta procesos de tipo ``rsync`` individuales en cada cubo.

Las pruebas preliminares que se realizan con una máquina virtual de cuatro núcleos mostraron una eficacia mejor al usar 64 procesos. Use la opción de ``msrsync````-p`` para establecer el número de procesos en 64.

Tenga en cuenta que ``msrsync`` solo se puede escribir en y desde volúmenes locales. El origen y el destino deben ser accesibles como montajes locales en la estación de trabajo que se usa para emitir el comando.

Siga estas instrucciones para usar ``msrsync`` y rellenar Azure Blob Storage con Azure HPC Cache:

1. Instale ``msrsync`` y sus requisitos previos (``rsync`` y Python 2.6 o posterior).
1. Determine el número total de archivos y directorios que se copiarán.

   Por ejemplo, use la utilidad ``prime.py`` con los argumentos ```prime.py --directory /path/to/some/directory``` (disponibles mediante la descarga de <https://github.com/Azure/Avere/blob/master/src/clientapps/dataingestor/prime.py>).

   Si no usa ``prime.py``, puede calcular el número de elementos con la herramienta ``find`` de GNU, como se muestra a continuación:

   ```bash
   find <path> -type f |wc -l         # (counts files)
   find <path> -type d |wc -l         # (counts directories)
   find <path> |wc -l                 # (counts both)
   ```

1. Divida el número de elementos por 64 para determinar el número de elementos por proceso. Use este número con la opción ``-f`` para establecer el tamaño de los cubos cuando ejecute el comando.

1. Ejecute el comando ``msrsync`` para copiar los archivos:

   ```bash
   msrsync -P --stats -p64 -f<ITEMS_DIV_64> --rsync "-ahv --inplace" <SOURCE_PATH> <DESTINATION_PATH>
   ```

   Por ejemplo, este comando está diseñado para mover 11 000 archivos en 64 procesos desde /test/source-repository hasta /mnt/vfxt/repository:

   ``mrsync -P --stats -p64 -f170 --rsync "-ahv --inplace" /test/source-repository/ /mnt/hpccache/repository``
