---
title: azcopy jobs remove | Microsoft Docs
description: En este artículo se proporciona información de referencia del comando azcopy jobs remove.
author: normesta
ms.service: storage
ms.topic: reference
ms.date: 07/24/2020
ms.author: normesta
ms.subservice: common
ms.reviewer: zezha-msft
ms.openlocfilehash: b1f1b58e9cce061aaa313457ec43256a766e3a2e
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87281980"
---
# <a name="azcopy-jobs-remove"></a>azcopy jobs remove

Quita todos los archivos asociados al identificador de trabajo especificado.

> [!NOTE] 
> Puede personalizar la ubicación donde se guardan los archivos de registro y de plan. Consulte el comando [azcopy env](storage-ref-azcopy-env.md) para obtener más información.

```
azcopy jobs remove [jobID] [flags]
```

## <a name="related-conceptual-articles"></a>Artículos conceptuales relacionados

- [Introducción a AzCopy](storage-use-azcopy-v10.md)
- [Transferencia de datos con AzCopy y Blob Storage](storage-use-azcopy-blobs.md)
- [Transferencia de datos con AzCopy y File Storage](storage-use-azcopy-files.md)
- [Configurar, optimizar y solucionar problemas de AzCopy](storage-use-azcopy-configure.md)

## <a name="examples"></a>Ejemplos

```
  azcopy jobs rm e52247de-0323-b14d-4cc8-76e0be2e2d44
```

## <a name="options"></a>Opciones

**--help** Ayuda para la eliminación.

## <a name="options-inherited-from-parent-commands"></a>Opciones heredadas de comandos primarios

**--cap-mbps float** Limita la velocidad de transferencia, en megabits por segundo. El rendimiento en un momento dado puede variar ligeramente del límite. Si esta opción se establece en cero o se omite, el rendimiento no se limita.

**--output-type** string   Formato de la salida del comando. Las opciones incluyen: text, json. El valor predeterminado es `text`. (`text` de manera predeterminada)

La cadena **--trusted-microsoft-suffixes**   Especifica sufijos de dominio adicionales en los que se pueden enviar tokens de inicio de sesión de Azure Active Directory.  El valor predeterminado es " *.core.windows.net;* .core.chinacloudapi.cn; *.core.cloudapi.de;* .core.usgovcloudapi.net". Los valores que se muestran aquí se agregan al valor predeterminado. Por seguridad, solo debe poner aquí dominios de Microsoft Azure. Separe las entradas con punto y coma.

## <a name="see-also"></a>Consulte también

- [azcopy jobs](storage-ref-azcopy-jobs.md)
