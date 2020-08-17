---
title: azcopy bench | Microsoft Docs
description: En este artículo se proporciona información de referencia del comando azcopy bench.
author: normesta
ms.service: storage
ms.topic: reference
ms.date: 07/24/2020
ms.author: normesta
ms.subservice: common
ms.reviewer: zezha-msft
ms.openlocfilehash: c1028d0a4a458746c08fd6fa4f16aa952d9962a2
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87282014"
---
# <a name="azcopy-benchmark"></a>azcopy benchmark

Ejecuta una prueba comparativa de rendimiento al cargar o descargar datos de prueba en o de un destino especificado. Para las cargas, los datos de prueba se generan automáticamente.

El comando de prueba comparativa ejecuta el mismo proceso que "copy", con las siguientes excepciones: 

  - En lugar de requerir los parámetros de origen y de destino, la prueba comparativa toma solo uno. Puede ser el contenedor de blobs, el recurso compartido de Azure Files o el sistema de archivos de Azure Data Lake Storage Gen2 donde desea cargar o descargar.

  - El parámetro "mode" describe si AzCopy debe probar las cargas o las descargas del destino determinado. Los valores válidos son "Upload" y "Download". El valor predeterminado es "Upload".

  - Para las pruebas comparativas de carga, la carga útil se describe mediante parámetros de la línea de comandos, que controlan el número de archivos que se generan automáticamente y el tamaño de los archivos. El proceso de generación se realiza por completo en la memoria. No se utiliza el disco.

  - En el caso de las descargas, la carga útil se compone de los archivos que ya existen en el origen. (Vea el ejemplo siguiente sobre cómo generar archivos de prueba, si es necesario).
  
  - Solo se admiten unos pocos de los parámetros opcionales que están disponibles para el comando copy.
  
  - Se miden y se notifican diagnósticos adicionales.
  
  - En el caso de las cargas, el comportamiento predeterminado es eliminar los datos transferidos al final de la serie de pruebas.  En el caso de las descargas, los datos nunca se guardan de forma local.

El modo de prueba comparativa se ajustará automáticamente al número de conexiones TCP paralelas que proporcione el máximo rendimiento. Mostrará ese número al final. Para evitar el ajuste automático, establezca la variable de entorno AZCOPY_CONCURRENCY_VALUE en un número específico de conexiones. 

Se admiten todos los tipos de autenticación habituales. Sin embargo, el enfoque más conveniente para las pruebas comparativas de carga suele ser crear un contenedor vacío con un token de SAS y usar la autenticación de SAS. (El modo de descarga requiere que haya un conjunto de datos de prueba en el contenedor de destino).

## <a name="examples"></a>Ejemplos

```azcopy
azcopy benchmark [destination] [flags]
```

Ejecución de una prueba comparativa con parámetros predeterminados (adecuada para las pruebas comparativas de redes de hasta 1 Gbps):'

```azcopy
azcopy bench "https://[account].blob.core.windows.net/[container]?<SAS>"
```
Ejecución de una prueba comparativa que carga 100 archivos, cada uno con 2 GiB de tamaño (adecuada para las pruebas comparativas en una red rápida; por ejemplo, 10 Gbps):'

```azcopy
azcopy bench "https://[account].blob.core.windows.net/[container]?<SAS>"--file-count 100 --size-per-file 2G
```
Ejecute una prueba comparativa, pero usa 50 000 archivos, cada uno con 8 MiB de tamaño, y procesa sus códigos hash MD5 (de la misma forma que la marca `--put-md5` lo hace en el comando copy). El propósito de `--put-md5` al realizar pruebas comparativas es comprobar si el cálculo de MD5 afecta a la capacidad de proceso del número y tamaño de archivos seleccionados:

```azcopy
azcopy bench --mode='Upload' "https://[account].blob.core.windows.net/[container]?<SAS>" --file-count 50000 --size-per-file 8M --put-md5
```

Ejecución de una prueba comparativa que descargue los archivos existentes de un destino

```azcopy
azcopy bench --mode='Download' "https://[account].blob.core.windows.net/[container]?<SAS?"
```

Ejecute una carga que no elimine los archivos transferidos. (Estos archivos pueden servir luego como carga útil para una prueba de descarga).

```azcopy
azcopy bench "https://[account].blob.core.windows.net/[container]?<SAS>" --file-count 100 --delete-test-data=false
```

## <a name="options"></a>Opciones

**--blob-type** string. Define el tipo de blob del destino. Se usa para permitir las pruebas comparativas entre distintos tipos de blob. Idéntico al parámetro del mismo nombre en el comando copy (valor predeterminado "Detectar").

**--block-size-mb** float. Utiliza este tamaño de bloque (especificado en MiB). El valor predeterminado se calcula automáticamente en función del tamaño del archivo. Se permiten fracciones decimales, por ejemplo: 0,25. Idéntico al parámetro del mismo nombre del comando copy.

**--check-length** Comprueba la longitud de un archivo en el destino después de la transferencia. Si hay una discrepancia entre el origen y el destino, la transferencia se marca como errónea. (El valor predeterminado es true)

**--delete-test-data**. Si es true, los datos de la prueba comparativa se eliminarán al final de la ejecución de la prueba.  Establézcalo en false si desea mantener los datos en el destino; por ejemplo, para utilizarlos en pruebas manuales fuera del modo de prueba comparativa (el valor predeterminado es true).

**--file-count** uint.  Número de archivos de datos generados automáticamente que se usarán (el valor predeterminado es 100).

**--help**. Ayuda para la prueba.

**--log-level** string. Define el nivel de detalle para el archivo de registro; los niveles disponibles son: INFO (todas las solicitudes y respuestas), WARNING (respuestas lentas), ERROR (solo solicitudes con error) y NONE (sin registros de salida). (Valor predeterminado: "INFO").

**--mode** string. Define si Azcopy debe probar las cargas o descargas desde este destino. Los valores válidos son "upload" y "download". La opción predeterminada es "upload". (Valor predeterminado "upload")

**--number-of-folders** uint. Si es mayor que 0, cree carpetas para dividir los datos.

**--put-md5**. Crea un código hash MD5 de cada archivo y lo guarda como la propiedad Content-MD5 del blob o archivo de destino. (De forma predeterminada, NO se crea el hash). Idéntico al parámetro del mismo nombre del comando copy.

**--size-per-file**  string. Tamaño de cada archivo de datos generado automáticamente. Debe ser un número inmediatamente seguido de K, M o G; por ejemplo: 12k o 200G (el valor predeterminado es "250M").

## <a name="options-inherited-from-parent-commands"></a>Opciones heredadas de comandos primarios

**--cap-mbps float**. Limita la velocidad de transferencia, en megabits por segundo. El rendimiento en un momento dado puede variar ligeramente del límite. Si esta opción se establece en cero o se omite, el rendimiento no se limita.

**--output-type**  string. Formato de la salida del comando. Las opciones incluyen: text, json. El valor predeterminado es "text". (Valor predeterminado: "text").

La cadena **--trusted-microsoft-suffixes**   Especifica sufijos de dominio adicionales en los que se pueden enviar tokens de inicio de sesión de Azure Active Directory.  El valor predeterminado es " *.core.windows.net;* .core.chinacloudapi.cn; *.core.cloudapi.de;* .core.usgovcloudapi.net". Los valores que se muestran aquí se agregan al valor predeterminado. Por seguridad, solo debe poner aquí dominios de Microsoft Azure. Separe las entradas con punto y coma.


## <a name="see-also"></a>Consulte también

- [azcopy](storage-ref-azcopy.md)
