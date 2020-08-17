---
title: azcopy load clfs | Microsoft Docs
titleSuffix: Azure Storage
description: En este artículo se proporciona información de referencia del comando azcopy load clfs.
author: normesta
ms.service: storage
ms.topic: reference
ms.date: 07/24/2020
ms.author: normesta
ms.subservice: common
ms.reviewer: zezha-msft
ms.openlocfilehash: 8f0a8903f90b134c35d9adb35a493d989d414b56
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87293880"
---
# <a name="azcopy-load-clfs"></a>azcopy load clfs

Transfiere datos locales a un contenedor y los almacena en el formato Avere Cloud FileSystem (CLFS) de Microsoft.

## <a name="synopsis"></a>Sinopsis

El comando load copia los datos en contenedores de almacenamiento de blobs de Azure y, a continuación, almacena esos datos en el formato Avere Cloud FileSystem (CLFS) de Microsoft. Los productos Azure HPC Cache y Avere vFXT for Azure usan el formato CLFS propietario.

Para aprovechar este comando, instale la extensión necesaria a través de: pip3 install clfsload~=1.0.23. Asegúrese de que CLFSLoad.py se encuentre en la variable PATH. Para obtener más información sobre este paso, visite [https://aka.ms/azcopy/clfs](https://aka.ms/azcopy/clfs).

Este comando es una opción sencilla para trasladar datos existentes al almacenamiento en la nube para su uso con productos específicos de caché de informática de alto rendimiento de Microsoft. 

Dado que estos productos utilizan un formato de sistema de archivos en la nube propietario para administrar los datos, no se pueden cargar a través del comando copy nativo. 

En su lugar, los datos se deben cargar a través del propio producto de caché o mediante este comando load, que usa el formato propietario correcto.
Este comando le permite transferir datos sin usar la memoria caché. Por ejemplo, para rellenar previamente el almacenamiento o agregar archivos a un espacio de trabajo sin aumentar la carga de la memoria caché.

El destino es un contenedor de Azure Storage vacío. Una vez completada la transferencia, el contenedor de destino se puede usar con una instancia de Azure HPC Cache o un clúster de Avere vFXT for Azure.

> [!NOTE] 
> Se trata de una versión preliminar del comando load. Notifique cualquier problema en el repositorio de GitHub de AzCopy.

```
azcopy load clfs [local dir] [container URL] [flags]
```

## <a name="related-conceptual-articles"></a>Artículos conceptuales relacionados

- [Introducción a AzCopy](storage-use-azcopy-v10.md)
- [Transferencia de datos con AzCopy y Blob Storage](storage-use-azcopy-blobs.md)
- [Transferencia de datos con AzCopy y File Storage](storage-use-azcopy-files.md)
- [Configurar, optimizar y solucionar problemas de AzCopy](storage-use-azcopy-configure.md)

## <a name="examples"></a>Ejemplos

Carga de un directorio completo en un contenedor con una SAS en formato CLFS:

```azcopy
azcopy load clfs "/path/to/dir" "https://[account].blob.core.windows.net/[container]?[SAS]" --state-path="/path/to/state/path"
```

## <a name="options"></a>Opciones

**--compression-type** cadena   especifique el tipo de compresión que se usará para las transferencias. Los valores disponibles son: `DISABLED`,`LZ4`. (`LZ4` por defecto)

**--help**    ayuda sobre el comando `azcopy load clfs`.

**--log-level** cadena   Define el nivel de detalle para el archivo de registro; los niveles disponibles son: `DEBUG`, `INFO`, `WARNING` y `ERROR`. (`INFO` por defecto)

**--max-errors** uint32   Especifica el número máximo de errores de transferencia que se tolerarán. Si se producen suficientes errores, se detiene el trabajo inmediatamente.

**--new-session**   Inicia un nuevo trabajo en lugar de continuar uno existente cuya información de seguimiento se mantiene en `--state-path`. (El valor predeterminado es true)

**--preserve-hardlinks**   Conserva las relaciones de vínculo físico.

**--state-path** cadena   Ruta de acceso obligatoria a un directorio local para el seguimiento del estado del trabajo. La ruta de acceso debe apuntar a un directorio existente para reanudar un trabajo. Debe estar vacío para un nuevo trabajo.

## <a name="options-inherited-from-parent-commands"></a>Opciones heredadas de comandos primarios

|Opción|Descripción|
|---|---|
|--cap-mbps número de punto flotante|Limita la velocidad de transferencia, en megabits por segundo. El rendimiento en un momento dado puede variar ligeramente del límite. Si esta opción se establece en cero o se omite, el rendimiento no se limita.|
|--output-type string|Formato de la salida del comando. Las opciones incluyen: text, json. El valor predeterminado es "text".|
|--trusted-microsoft-suffixes string   | Especifica sufijos de dominio adicionales en los que se pueden enviar tokens de inicio de sesión de Azure Active Directory.  El valor predeterminado es " *.core.windows.net;* .core.chinacloudapi.cn; *.core.cloudapi.de;* .core.usgovcloudapi.net". Los valores que se muestran aquí se agregan al valor predeterminado. Por seguridad, solo debe poner aquí dominios de Microsoft Azure. Separe las entradas con punto y coma.|

## <a name="see-also"></a>Consulte también

- [azcopy](storage-ref-azcopy.md)
