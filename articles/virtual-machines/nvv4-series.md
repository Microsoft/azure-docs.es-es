---
title: Serie NVv4
description: Especificaciones de las máquinas virtuales de la serie NVv4.
services: virtual-machines
ms.subservice: sizes
author: vikancha-MSFT
ms.service: virtual-machines
ms.topic: conceptual
ms.date: 02/03/2020
ms.author: jushiman
ms.openlocfilehash: 9b841b6422a4314b43a594cb0b22040f884228eb
ms.sourcegitcommit: 8def3249f2c216d7b9d96b154eb096640221b6b9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/03/2020
ms.locfileid: "87543901"
---
# <a name="nvv4-series"></a>Serie NVv4 

Las máquinas virtuales de la serie NVv4 usan la tecnología de las GPU [AMD Radeon instinto MI25](https://www.amd.com/en/products/professional-graphics/instinct-mi25) y las CPU AMD EPYC 7V12 (Rome). Con la serie NVv4, Azure introduce máquinas virtuales con GPU parciales. Elija la máquina virtual del tamaño adecuado para aplicaciones con aceleración gráfica por GPU y escritorios virtuales a partir de un octavo de una GPU con un búfer de cuadros de 2 GiB y hasta una GPU completa con un búfer de cuadros de 16 GiB. Las máquinas virtuales NVv4 actualmente solo admiten el sistema operativo invitado Windows.

<br>

ACU: 230-260

Premium Storage:  Compatible

Almacenamiento en caché de Premium Storage:  Compatible

Migración en vivo: No compatible

Actualizaciones con conservación de memoria: No compatible

| Size | vCPU | Memoria: GiB | GiB de almacenamiento temporal (SSD) | GPU | Memoria de GPU: GiB | Discos de datos máx. | Nº máx. de NIC / ancho de banda de red esperado (MBps) |
| --- | --- | --- | --- | --- | --- | --- | --- |
| Standard_NV4as_v4 |4 |14 |88 | 1/8 | 2 | 4 | 2 / 1000 |
| Standard_NV8as_v4 |8 |28 |176 | 1/4 | 4 | 8 | 4 / 2000 |
| Standard_NV16as_v4 |16 |56 |352 | 1/2 | 8 | 16 | 8 / 4000 |
| Standard_NV32as_v4 |32 |112 |704 | 1 | 16 | 32 | 8 / 8000 |

<sup>1</sup> las máquinas virtuales de la serie NVv4 incluyen tecnología multithreading simultánea de AMD

[!INCLUDE [virtual-machines-common-sizes-table-defs](../../includes/virtual-machines-common-sizes-table-defs.md)]

## <a name="supported-operating-systems-and-drivers"></a>Sistemas operativos y controladores compatibles

Para aprovechar las funcionalidades de GPU de las VM de la serie NVv4 de Azure que ejecutan Windows, deben instalarse controladores de GPU de AMD.

Para instalar manualmente los controladores de GPU de AMD, vea [Instalación de controladores de GPU de AMD de la serie N para Windows](./windows/n-series-amd-driver-setup.md) para obtener información sobre los sistemas operativos y controladores compatibles, así como sobre los pasos de instalación y verificación.

## <a name="other-sizes"></a>Otros tamaños

- [Uso general](sizes-general.md)
- [Memoria optimizada](sizes-memory.md)
- [Almacenamiento optimizado](sizes-storage.md)
- [GPU optimizada](sizes-gpu.md)
- [Proceso de alto rendimiento](sizes-hpc.md)
- [Generaciones anteriores](sizes-previous-gen.md)

## <a name="next-steps"></a>Pasos siguientes

Obtenga más información sobre cómo las [unidades de proceso de Azure (ACU)](acu.md) pueden ayudarlo a comparar el rendimiento en los distintos SKU de Azure.
