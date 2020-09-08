---
title: Convenciones de nomenclatura de tamaños de máquina virtual de Azure
description: Aquí se explican las convenciones de nomenclatura que se usan para los tamaños de las máquinas virtuales de Azure
ms.service: virtual-machines
subservice: sizes
author: mimckitt
ms.topic: conceptual
ms.date: 7/22/2020
ms.author: mimckitt
ms.custom: sttsinar
ms.openlocfilehash: 13894e534dc8d6dd89baf75ea2bd3b6500b718f7
ms.sourcegitcommit: 271601d3eeeb9422e36353d32d57bd6e331f4d7b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88650968"
---
# <a name="azure-virtual-machine-sizes-naming-conventions"></a>Convenciones de nomenclatura de los tamaños de las máquinas virtuales de Azure

En esta página se explican las convenciones de nomenclatura que se usan para las máquinas virtuales de Azure. Las máquinas virtuales usan estas convenciones de nomenclatura para indicar distintas características y especificaciones.

## <a name="naming-convention-explanation"></a>Explicación de convención de nomenclatura

**[Familia]**  +  **[Subfamilia*]**  +  **[N. º de vCPU]**  +  **[Características adicionales]**  +  **[Tipo de acelerador*]**  +  **[Versión]**

|Value | Explicación|
|---|---|
| Familia | Indica la serie de la familia de la máquina virtual| 
| *Subfamilia | Se usa solo para diferenciaciones especializadas de máquinas virtuales|
| N. º de vCPU| Indica el número de vCPU de la máquina virtual |
| Características adicionales | Una o más letras minúsculas indican características adicionales, como: <br> a = procesador basado en AMD <br> d = disco (el disco temporal local está presente); esto es para las máquinas virtuales de Azure más recientes, vea [Series Ddv4 y Ddsv4](./ddv4-ddsv4-series.md) <br> h = compatible con hibernación <br> i = tamaño aislado <br> l = poca memoria; menor cantidad de memoria que el tamaño de memoria intensiva <br> m = memoria intensiva; la mayor cantidad de memoria en un tamaño determinado <br> t = memoria minúscula; la menor cantidad de memoria en un tamaño determinado <br> r = compatible con RDMA <br> s = compatible con Premium Storage, incluido el posible uso de [SSD Ultra](./disks-types.md#ultra-disk) (Nota: Algunos tamaños más recientes sin el atributo s son compatibles con Premium Storage, por ejemplo, M128, M64, etc.)<br> |
| *Tipo de acelerador | Indica el tipo de acelerador de hardware de los SKU especializados o GPU. Solo los nuevos SKU especializados o GPU lanzados a partir del tercer trimestre de 2020 tienen el acelerador de hardware en el nombre. |
| Versión | Indica la versión de la serie de la familia de máquinas virtuales |

## <a name="example-breakdown"></a>Desglose de ejemplo

**[Familia]**  +  **[Subfamilia*]**  +  **[N. º de vCPU]**  +  **[Características adicionales]**  +  **[Tipo de acelerador*]**  +  **[Versión]**

### <a name="example-1-m416ms_v2"></a>Ejemplo 1: M416ms_v2

|Value | Explicación|
|---|---|
| Familia | M | 
| N. º de vCPU | 416 |
| Características adicionales | m = memoria intensiva <br> s = compatible con Premium Storage |
| Versión | v2 |

### <a name="example-2-nv16as_v4"></a>Ejemplo 2: NV16as_v4

|Value | Explicación|
|---|---|
| Familia | N | 
| Subfamilia | V |
| N. º de vCPU | 16 |
| Características adicionales | a = procesador basado en AMD <br> s = compatible con Premium Storage |
| Versión | v4 |

### <a name="example-3-nc4as_t4_v3"></a>Ejemplo 3: NC4as_T4_v3

|Value | Explicación|
|---|---|
| Familia | N | 
| Subfamilia | C |
| N. º de vCPU | 4 |
| Características adicionales | a = procesador basado en AMD <br> s = compatible con Premium Storage |
| Tipo de acelerador | T4 |
| Versión | v3 |

## <a name="next-steps"></a>Pasos siguientes

Obtenga más información sobre los [Tamaños de máquina virtual](./sizes.md) disponibles en Azure. 