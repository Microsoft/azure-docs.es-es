---
title: azcopy list | Microsoft Docs
description: En este artículo se proporciona información de referencia del comando azcopy list.
author: normesta
ms.service: storage
ms.topic: reference
ms.date: 07/24/2020
ms.author: normesta
ms.subservice: common
ms.reviewer: zezha-msft
ms.openlocfilehash: f4d54e4fd8ca0d43cf3d7087006bb51a15a97bcd
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87289220"
---
# <a name="azcopy-list"></a>azcopy list

Enumera las entidades de un recurso determinado.

## <a name="synopsis"></a>Sinopsis

Solo se admiten contenedores de blobs en la versión actual.

```azcopy
azcopy list [containerURL] [flags]
```

## <a name="related-conceptual-articles"></a>Artículos conceptuales relacionados

- [Introducción a AzCopy](storage-use-azcopy-v10.md)
- [Transferencia de datos con AzCopy y Blob Storage](storage-use-azcopy-blobs.md)
- [Transferencia de datos con AzCopy y File Storage](storage-use-azcopy-files.md)
- [Configurar, optimizar y solucionar problemas de AzCopy](storage-use-azcopy-configure.md)

## <a name="examples"></a>Ejemplos

```azcopy
azcopy list [containerURL]
```

## <a name="options"></a>Opciones

|Opción|Descripción|
|--|--|
|-h, --help|Muestra el contenido de la ayuda para el comando list.|
|--machine-readable|Muestra los tamaños de archivo en bytes.|
|--mega-units|Muestra las unidades en grupos de 1000, no de 1024.|
|--running-tally|Cuenta el número total de archivos y sus tamaños.|

## <a name="options-inherited-from-parent-commands"></a>Opciones heredadas de comandos primarios

|Opción|Descripción|
|---|---|
|--cap-mbps número de punto flotante|Limita la velocidad de transferencia, en megabits por segundo. El rendimiento en un momento dado puede variar ligeramente del límite. Si esta opción se establece en cero o se omite, el rendimiento no se limita.|
|--output-type string|Formato de la salida del comando. Las opciones incluyen: text, json. El valor predeterminado es "text".|
|--trusted-microsoft-suffixes string   |Especifica sufijos de dominio adicionales en los que se pueden enviar tokens de inicio de sesión de Azure Active Directory.  El valor predeterminado es " *.core.windows.net;* .core.chinacloudapi.cn; *.core.cloudapi.de;* .core.usgovcloudapi.net". Los valores que se muestran aquí se agregan al valor predeterminado. Por seguridad, solo debe poner aquí dominios de Microsoft Azure. Separe las entradas con punto y coma.|

## <a name="see-also"></a>Consulte también

- [azcopy](storage-ref-azcopy.md)
