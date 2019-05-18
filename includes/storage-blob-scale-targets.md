---
author: tamram
ms.service: storage
ms.topic: include
ms.date: 4/20/2019
ms.author: tamram
ms.openlocfilehash: aab17966862c57a52f252b3c4e9b757673078b0a
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65787597"
---
| Recurso | Destino        |
|----------|---------------|
| Tamaño máximo del contenedor de blob único | Igual que la capacidad de la cuenta de almacenamiento máximo |
| Número máximo de bloques en un bloque de blobs o blob en anexos | 50 000 bloques |
| Tamaño máximo de un bloque en un blob en bloques | 100 MiB |
| Tamaño máximo de un blob en bloques | 50 000 x 100 MiB (4,75 TiB aproximadamente) |
| Tamaño máximo de un bloque en un blob en anexos | 4 MiB |
| Tamaño máximo de un blob en anexos | 50 000 x 4 MiB (195 GiB aproximadamente) |
| Tamaño máximo de un blob en páginas | 8 TiB |
| Número máximo de directivas de acceso almacenadas por contenedor de blobs | 5 |
|Rendimiento de un blob |Hasta los límites de entrada/salida de cuenta de almacenamiento<sup>1</sup> |

<sup>1</sup> rendimiento único objeto depende de varios factores, incluidos, pero sin limitarse a: simultaneidad, el tamaño de la solicitud, nivel de rendimiento, velocidad de origen para las cargas y de destino para las descargas. Para aprovechar las ventajas de [blob en bloques de alto rendimiento](https://azure.microsoft.com/blog/high-throughput-with-azure-blob-storage/) mejoras de rendimiento, utilice un tamaño de la solicitud Put Blob o Put Block de > 4 MiB (> 256 KiB para blob storage en bloques de rendimiento de premium o para Data Lake Storage Gen2).
