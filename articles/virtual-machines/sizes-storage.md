---
title: 'Tamaños de las máquinas virtuales de Azure: almacenamiento | Microsoft Docs'
description: Enumera los tamaños diferentes optimizados para almacenamiento disponibles para las máquinas virtuales en Azure. Se proporciona información sobre el número de unidades vCPU, discos de datos y NIC, así como sobre el rendimiento de almacenamiento y el ancho de banda de red para los tamaños de esta serie.
ms.subservice: sizes
documentationcenter: ''
author: sasha-melamed
ms.service: virtual-machines
ms.topic: conceptual
ms.workload: infrastructure-services
ms.date: 02/03/2020
ms.author: jushiman
ms.openlocfilehash: 5af7b3373993dce1939ecd7534140e58db688579
ms.sourcegitcommit: 2ff0d073607bc746ffc638a84bb026d1705e543e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87835585"
---
# <a name="storage-optimized-virtual-machine-sizes"></a>Tamaños de máquina virtual optimizada para almacenamiento

Los tamaños de VM optimizadas para almacenamiento proporcionan un alto rendimiento de disco y E/S, y son ideales para macrodatos, bases de datos SQL y NoSQL, el almacenamiento de datos y bases de datos transaccionales grandes.  Por ejemplo, Cassandra, MongoDB, Cloudera y Redis. En este artículo, se proporciona información acerca del número de vCPU, discos de datos y NIC, así como del rendimiento del almacenamiento local y del ancho de banda de red para cada tamaño optimizado.

La [serie Lsv2](lsv2-series.md) proporciona un alto rendimiento, baja latencia, almacenamiento NVMe local asignado directamente que se ejecuta en el procesador [AMD EPYC<sup>TM</sup> 7551](https://www.amd.com/en/products/epyc-7000-series) con una potencia en todos los núcleos de 2,55 GHz y una potencia máxima de 3,0 GHz. Las máquinas virtuales de la serie Lsv2 están disponibles en tamaños de 8 a 80 vCPU en una configuración de varios subprocesos simultáneos.  Hay 8 GiB de memoria por vCPU y un dispositivo SSD NVMe M.2 de 1,92 TB por cada 8 vCPU, con hasta 19,2 TB (10 x 1,92 TB) disponibles en la versión L80s v2.

## <a name="other-sizes"></a>Otros tamaños

- [Uso general](sizes-general.md)
- [Proceso optimizado](sizes-compute.md)
- [Memoria optimizada](sizes-memory.md)
- [GPU optimizada](sizes-gpu.md)
- [Proceso de alto rendimiento](sizes-hpc.md)
- [Generaciones anteriores](sizes-previous-gen.md)

## <a name="next-steps"></a>Pasos siguientes

Obtenga más información sobre cómo las [unidades de proceso de Azure (ACU)](acu.md) pueden ayudarlo a comparar el rendimiento en los distintos SKU de Azure.

Aprenda a optimizar el rendimiento en las máquinas virtuales de la serie Lsv2 para [Windows](windows/storage-performance.md) o [Linux](linux/storage-performance.md).

Para obtener más información sobre cómo Azure asigna nombres a las máquinas virtuales, consulte [Convenciones de nomenclatura de los tamaños de las máquinas virtuales de Azure](./vm-naming-conventions.md).