---
title: Serie Av2
description: Especificaciones de las máquinas virtuales de la serie Av2.
author: migerdes
ms.service: virtual-machines
ms.subservice: sizes
ms.topic: conceptual
ms.date: 02/03/2020
ms.author: jushiman
ms.openlocfilehash: fcbcb89505130dddb42d97c153b172a22a9b29b2
ms.sourcegitcommit: 2ff0d073607bc746ffc638a84bb026d1705e543e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87831488"
---
# <a name="av2-series"></a>Serie Av2

Las máquinas virtuales de la serie Av2 se pueden implementar en diversos procesadores y tipos de hardware. Las máquinas virtuales de la serie Av2 tienen las configuraciones de memoria y rendimiento de CPU adecuadas para cargas de trabajo de nivel de entrada como desarrollo y pruebas. El tamaño es una limitación para ofrecer un rendimiento coherente del procesador para la instancia en ejecución, independientemente del hardware en que se implementó. Con el fin de determinar el hardware físico en que se implementó este tamaño, cree una consulta para el hardware virtual desde dentro de la máquina virtual. Algunos casos de uso son, por ejemplo, los servidores de desarrollo y pruebas, los servidores web con poco tráfico, las bases de datos de tamaño pequeño a mediano, las pruebas de concepto y los repositorios de código.

ACU: 100

Premium Storage:  No compatible

Almacenamiento en caché de Premium Storage:  No compatible

Migración en vivo: Compatible

Actualizaciones con conservación de memoria: Compatible


| Size | vCPU | Memoria: GiB | GiB de almacenamiento temporal (SSD) | Rendimiento máximo de almacenamiento temporal: IOPS/Mbps de lectura/Mbps de escritura | Discos de datos máx./rendimiento: E/S | Nº máx. NIC | Ancho de banda de red esperado (Mbps)
|---|---|---|---|---|---|---|---|
| Standard_A1_v2  | 1 | 2  | 10 | 1000/20/10  | 2/2x500   | 2 | 250  |
| Standard_A2_v2  | 2 | 4  | 20 | 2000/40/20  | 4/4x500   | 2 | 500  |
| Standard_A4_v2  | 4 | 8  | 40 | 4000/80/40  | 8/8x500   | 4 | 1000 |
| Standard_A8_v2  | 8 | 16 | 80 | 8000/160/80 | 16/16x500 | 8 | 2000 |
| Standard_A2m_v2 | 2 | 16 | 20 | 2000/40/20  | 4/4x500   | 2 | 500  |
| Standard_A4m_v2 | 4 | 32 | 40 | 4000/80/40  | 8/8x500   | 4 | 1000 |
| Standard_A8m_v2 | 8 | 64 | 80 | 8000/160/80 | 16/16x500 | 8 | 2000 |

[!INCLUDE [virtual-machines-common-sizes-table-defs](../../includes/virtual-machines-common-sizes-table-defs.md)]

## <a name="other-sizes-and-information"></a>Otros tamaños e información

- [Uso general](sizes-general.md)
- [Memoria optimizada](sizes-memory.md)
- [Almacenamiento optimizado](sizes-storage.md)
- [GPU optimizada](sizes-gpu.md)
- [Proceso de alto rendimiento](sizes-hpc.md)
- [Generaciones anteriores](sizes-previous-gen.md)

Calculadora de precios: [Calculadora de precios](https://azure.microsoft.com/pricing/calculator/)

Más información sobre los tipos de disco: [Tipos de disco](./linux/disks-types.md#ultra-disk)

## <a name="next-steps"></a>Pasos siguientes

Obtenga más información sobre cómo las [unidades de proceso de Azure (ACU)](acu.md) pueden ayudarlo a comparar el rendimiento en los distintos SKU de Azure.