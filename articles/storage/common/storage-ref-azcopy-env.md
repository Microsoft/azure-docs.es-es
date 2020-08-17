---
title: azcopy env | Microsoft Docs
description: En este artículo se proporciona información de referencia del comando azcopy env.
author: normesta
ms.service: storage
ms.topic: reference
ms.date: 07/24/2020
ms.author: normesta
ms.subservice: common
ms.reviewer: zezha-msft
ms.openlocfilehash: 197d8ee1ea1d389a26548fa65adee83c6a92cf96
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87275979"
---
# <a name="azcopy-env"></a>azcopy env

Muestra las variables de entorno que pueden configurar el comportamiento de AzCopy.

## <a name="synopsis"></a>Sinopsis

```azcopy
azcopy env [flags]
```

> [!IMPORTANT]
> Si establece una variable de entorno mediante la línea de comandos, esa variable se podrá leer en el historial de la línea de comandos. Considere la posibilidad de borrar las variables que contengan credenciales del historial de la línea de comandos. Para evitar que aparezcan variables en el historial, puede usar un script para pedir al usuario sus credenciales y establecer la variable de entorno.

## <a name="related-conceptual-articles"></a>Artículos conceptuales relacionados

- [Introducción a AzCopy](storage-use-azcopy-v10.md)
- [Transferencia de datos con AzCopy y Blob Storage](storage-use-azcopy-blobs.md)
- [Transferencia de datos con AzCopy y File Storage](storage-use-azcopy-files.md)
- [Configurar, optimizar y solucionar problemas de AzCopy](storage-use-azcopy-configure.md)

## <a name="options"></a>Opciones

|Opción|Descripción|
|--|--|
|-h, --help|Muestra el contenido de la ayuda para el comando env. |
|--show-sensitive|Muestra las variables de entorno confidenciales o secretas.|

## <a name="options-inherited-from-parent-commands"></a>Opciones heredadas de comandos primarios

|Opción|Descripción|
|---|---|
|--cap-mbps float|Limita la velocidad de transferencia, en megabits por segundo. El rendimiento en un momento dado puede variar ligeramente del límite. Si esta opción se establece en cero o se omite, el rendimiento no se limita.|
|--output-type string|Formato de la salida del comando. Las opciones incluyen: text, json. El valor predeterminado es "text".|
|--trusted-microsoft-suffixes string  | Especifica sufijos de dominio adicionales en los que se pueden enviar tokens de inicio de sesión de Azure Active Directory.  El valor predeterminado es " *.core.windows.net;* .core.chinacloudapi.cn; *.core.cloudapi.de;* .core.usgovcloudapi.net". Los valores que se muestran aquí se agregan al valor predeterminado. Por seguridad, solo debe poner aquí dominios de Microsoft Azure. Separe las entradas con punto y coma.|

## <a name="see-also"></a>Consulte también

- [azcopy](storage-ref-azcopy.md)
