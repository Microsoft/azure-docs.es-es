---
title: Comparación del rendimiento de la aplicación en Azure Disk Storage
description: Obtenga información sobre el proceso de comparación de la aplicación en Azure.
author: roygara
ms.author: rogarana
ms.date: 01/11/2019
ms.topic: how-to
ms.service: virtual-machines-linux
ms.subservice: disks
ms.openlocfilehash: 293164413c4c8ecec0295152cb6db3d73718e00d
ms.sourcegitcommit: 56cbd6d97cb52e61ceb6d3894abe1977713354d9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88690789"
---
# <a name="benchmark-your-application-on-azure-disk-storage"></a>Pruebas comparativas de la aplicación en Azure Disk Storage

Las pruebas comparativas consisten en el proceso de simular cargas de trabajo diferentes en la aplicación y medir el rendimiento de las aplicaciones para cada carga de trabajo. Siguiendo los pasos descritos en el [artículo sobre el diseño de alto rendimiento](../premium-storage-performance.md). Al ejecutar las herramientas de pruebas comparativas en las máquinas virtuales en las que se hospeda la aplicación, puede determinar los niveles de rendimiento que la aplicación puede lograr con Premium Storage. En este artículo, se proporcionan ejemplos de pruebas comparativas realizadas con una máquina virtual estándar DS14 aprovisionada con discos de Azure Premium Storage.

Hemos usado las herramientas de pruebas comparativas comunes Iometer y FIO, para Windows y Linux respectivamente. Estas herramientas generan varios subprocesos que simulan una carga de trabajo de producción y miden el rendimiento del sistema. Con estas herramientas, también puede configurar parámetros como la profundidad de la cola y el tamaño de bloque, que normalmente no se puede cambiar de una aplicación. Esto proporciona más flexibilidad para controlar el rendimiento máximo en una máquina virtual a gran escala aprovisionada con discos premium para diferentes tipos de cargas de trabajo de la aplicación. Para más información sobre la herramienta de pruebas comparativas, visite [Iometer](http://www.iometer.org/) y [FIO](http://freecode.com/projects/fio).

Para seguir estos ejemplos, cree una máquina virtual estándar DS14 y conecte 11 discos de Premium Storage a la máquina virtual. De los once discos, configure diez con almacenamiento en caché de host como "None" y secciónelos en un volumen denominado NoCacheWrites. Configure el almacenamiento en caché de host como "ReadOnly" en el disco restante y cree un volumen denominado CacheReads con dicho disco. Con esta configuración, puede ver el rendimiento máximo de lectura y escritura de una máquina virtual estándar DS14. Para obtener instrucciones detalladas sobre cómo crear una máquina virtual DS14 con discos Premium, consulte [Azure Premium Storage: Diseño de alto rendimiento](../premium-storage-performance.md).

[!INCLUDE [virtual-machines-disks-benchmarking](../../../includes/virtual-machines-managed-disks-benchmarking.md)]

## <a name="next-steps"></a>Pasos siguientes

Diríjase al artículo sobre el [diseño para lograr un alto rendimiento](../premium-storage-performance.md).

En dicho artículo, se crea una lista de comprobación similar a la aplicación existente para el prototipo. Con herramientas de pruebas comparativas puede simular las cargas de trabajo y medir el rendimiento de las aplicaciones de prototipo. De este modo, puede determinar qué oferta de disco puede alcanzar o superar los requisitos de rendimiento de las aplicaciones. A continuación, puede implementar las mismas directrices para la aplicación de producción.