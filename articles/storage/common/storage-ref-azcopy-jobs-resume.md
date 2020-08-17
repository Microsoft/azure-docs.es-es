---
title: azcopy jobs resume | Microsoft Docs
description: En este artículo se proporciona información de referencia del comando azcopy jobs resume.
author: normesta
ms.service: storage
ms.topic: reference
ms.date: 07/24/2020
ms.author: normesta
ms.subservice: common
ms.reviewer: zezha-msft
ms.openlocfilehash: dd25bec04d651c01d622f0652a29a65069421786
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87281963"
---
# <a name="azcopy-jobs-resume"></a>azcopy jobs resume

Reanuda el trabajo existente con el identificador de trabajo especificado.

## <a name="synopsis"></a>Sinopsis

```azcopy
azcopy jobs resume [jobID] [flags]
```

## <a name="related-conceptual-articles"></a>Artículos conceptuales relacionados

- [Introducción a AzCopy](storage-use-azcopy-v10.md)
- [Transferencia de datos con AzCopy y Blob Storage](storage-use-azcopy-blobs.md)
- [Transferencia de datos con AzCopy y File Storage](storage-use-azcopy-files.md)
- [Configurar, optimizar y solucionar problemas de AzCopy](storage-use-azcopy-configure.md)

## <a name="options"></a>Opciones

|Opción|Descripción|
|--|--|
|--destination-sas string|SAS de destino del destino para un id. de trabajo determinado.|
|--exclude string|Filtro: excluye estas transferencias con error al reanudar el trabajo. Los archivos deben estar separados por ";".|
|-h, --help|Muestra el contenido de la ayuda para el comando resume.|
|--include string|Filtro: solo excluye estas transferencias con error al reanudar el trabajo. Los archivos deben estar separados por ";".|
|--source-sas string |SAS de origen del origen para un id. de trabajo determinado.|

## <a name="options-inherited-from-parent-commands"></a>Opciones heredadas de comandos primarios

|Opción|Descripción|
|---|---|
|--cap-mbps número de tipo float|Limita la velocidad de transferencia, en megabits por segundo. El rendimiento en un momento dado puede variar ligeramente del límite. Si esta opción se establece en cero o se omite, el rendimiento no se limita.|
|--output-type string|Formato de la salida del comando. Las opciones incluyen: text, json. El valor predeterminado es "text".|
|--trusted-microsoft-suffixes string   |Especifica sufijos de dominio adicionales en los que se pueden enviar tokens de inicio de sesión de Azure Active Directory.  El valor predeterminado es " *.core.windows.net;* .core.chinacloudapi.cn; *.core.cloudapi.de;* .core.usgovcloudapi.net". Los valores que se muestran aquí se agregan al valor predeterminado. Por seguridad, solo debe poner aquí dominios de Microsoft Azure. Separe las entradas con punto y coma.|

## <a name="see-also"></a>Consulte también

- [azcopy jobs](storage-ref-azcopy-jobs.md)
