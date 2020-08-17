---
title: azcopy jobs show | Microsoft Docs
description: En este artículo se proporciona información de referencia del comando azcopy jobs show.
author: normesta
ms.service: storage
ms.topic: reference
ms.date: 07/24/2020
ms.author: normesta
ms.subservice: common
ms.reviewer: zezha-msft
ms.openlocfilehash: 7ffb16936ec96f30ad7d8e05babbadb85e01a71e
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87281946"
---
# <a name="azcopy-jobs-show"></a>azcopy jobs show

Muestra información detallada del identificador de trabajo especificado.

## <a name="synopsis"></a>Sinopsis

Si solo se proporciona el identificador de trabajo sin ninguna marca, se devolverá el resumen de progreso del trabajo.

Los recuentos en bytes y el porcentaje completado que aparecen al ejecutar este comando solo reflejan los archivos que se completan en el trabajo. No reflejan los archivos completados parcialmente.

Si se establece la marca `with-status`, se mostrará la lista de transferencias del trabajo con el valor especificado.

```azcopy
azcopy jobs show [jobID] [flags]
```

## <a name="related-conceptual-articles"></a>Artículos conceptuales relacionados

- [Introducción a AzCopy](storage-use-azcopy-v10.md)
- [Transferencia de datos con AzCopy y Blob Storage](storage-use-azcopy-blobs.md)
- [Transferencia de datos con AzCopy y File Storage](storage-use-azcopy-files.md)
- [Configurar, optimizar y solucionar problemas de AzCopy](storage-use-azcopy-configure.md)

## <a name="options"></a>Opciones

|Opción|Descripción|
|--|--|
|-h, --help|Muestra el contenido de la ayuda para el comando show.|
|--with-status string|Solo muestra la lista de las transferencias de un trabajo con este estado. Valores disponibles: Started (Iniciado), Success (Correcto), Failed (Con errores)|

## <a name="options-inherited-from-parent-commands"></a>Opciones heredadas de comandos primarios

|Opción|Descripción|
|---|---|
|--cap-mbps float|Limita la velocidad de transferencia, en megabits por segundo. El rendimiento en un momento dado puede variar ligeramente del límite. Si esta opción se establece en cero o se omite, el rendimiento no se limita.|
|--output-type string|Formato de la salida del comando. Las opciones incluyen: text, json. El valor predeterminado es "text".|
|--trusted-microsoft-suffixes string   |Especifica sufijos de dominio adicionales en los que se pueden enviar tokens de inicio de sesión de Azure Active Directory.  El valor predeterminado es " *.core.windows.net;* .core.chinacloudapi.cn; *.core.cloudapi.de;* .core.usgovcloudapi.net". Los valores que se muestran aquí se agregan al valor predeterminado. Por seguridad, solo debe poner aquí dominios de Microsoft Azure. Separe las entradas con punto y coma.|

## <a name="see-also"></a>Consulte también

- [azcopy jobs](storage-ref-azcopy-jobs.md)
