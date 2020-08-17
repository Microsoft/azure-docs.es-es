---
author: roygara
ms.service: storage
ms.topic: include
ms.date: 08/10/2020
ms.author: rogarana
ms.openlocfilehash: d704c6026e9d007a7365a3b72649ca509585da4d
ms.sourcegitcommit: 269da970ef8d6fab1e0a5c1a781e4e550ffd2c55
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/10/2020
ms.locfileid: "88057633"
---
#### <a name="additional-premium-file-share-level-limits"></a>Límites del nivel de recurso compartido de archivos prémium adicionales

|Área  |Destino  |
|---------|---------|
|Aumento o disminución del tamaño mínimo    |1 GiB      |
|IOPS base    |1 IOPS por GiB hasta 100 000|
|Ampliación de IOPS    |3 x IOPS por GiB hasta 100 000|
|Velocidad de salida         |60 MiB/s + 0,06 * GiB aprovisionado        |
|Velocidad de entrada| 40 MiB/s + 0,04 * GiB aprovisionado |

#### <a name="file-level-limits"></a>Límites de nivel de archivo

|Área  |Archivo estándar  |Archivo prémium  |
|---------|---------|---------|
|Size     |1 TiB         |4 TiB         |
|IOPS máximas por archivo      |1,000         |5\.000         |
|Identificadores simultáneos     |2\.000         |2\.000         |
|Salida     |Consulte los valores de rendimiento de archivo estándar         |300 MiB/s         |
|Entrada     |Consulte los valores de rendimiento de archivo estándar         |200 MiB/s         |
|Throughput     |Hasta 60 MiB/s         |Consulte los valores de entrada y salida de archivos prémium         |