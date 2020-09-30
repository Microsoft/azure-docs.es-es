---
author: alkohli
ms.service: databox
ms.topic: include
ms.date: 08/03/2020
ms.author: alkohli
ms.openlocfilehash: 7ce49873b4e59bcf474deaea4420f56a72c9c589
ms.sourcegitcommit: 656c0c38cf550327a9ee10cc936029378bc7b5a2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89082552"
---
El tamaño de la máquina virtual determina la cantidad de recursos de proceso, como memoria, CPU y GPU, que están disponibles para la máquina virtual. Las máquinas virtuales deben crearse con un tamaño de máquina virtual adecuado para la carga de trabajo. Aunque todas las máquinas se ejecutarán en el mismo hardware, los tamaños de máquina tienen límites diferentes para el acceso al disco, lo que puede ayudarle a administrar el acceso total al disco en todas las máquinas virtuales. Si esta aumenta, también se puede cambiar el tamaño de las máquinas virtuales existentes.

Se admiten las siguientes máquinas virtuales de la serie Standard Dv2 para la creación en un dispositivo de Azure Stack Edge.

### <a name="dv2-series"></a>Serie Dv2
|Size     |vCPU     |Memoria (GiB) | Almacenamiento temporal (GiB)  | Rendimiento de discos del SO máx. (IOPS) | Rendimiento máximo de almacenamiento temporal (IOPS) | Discos de datos máx. / rendimiento (IOPS) | Nº máx. NIC |
|-------------------|----|----|-----|----|------|------------|---------|
|**Standard_D1_v2** |1   |3,5 |50   |500 |3000  |4 / 4x500   |2 |
|**Standard_D2_v2** |2   |7   |100  |500 |6000  |8 / 8x500   |2 |
|**Standard_D3_v2** |4   |14  |200  |500 |12000 |16 / 16x500 |4 |
|**Standard_D4_v2** |8   |28  |400  |500 |24000 |32 / 32x500 |8 |
|**Standard_D5_v2** |16  |56  |800  |500 |48000 |64 / 64x500 |8 |

### <a name="dsv2-series"></a>DSv2-series
|Size     |vCPU     |Memoria (GiB) | Almacenamiento temporal (GiB)  | Rendimiento de discos del SO máx. (IOPS) | Rendimiento máximo de almacenamiento temporal (IOPS) | Discos de datos máx. / rendimiento (IOPS) | Nº máx. NIC |
|--------------------|----|----|----|-----|------|-------------|---------|
|**Standard_DS1_v2** |1   |3,5 |7   |1000 |4000  |4 / 4x2300   |2 |
|**Standard_DS2_v2** |2   |7   |14  |1000 |8000  |8 / 8x2300   |2 |
|**Standard_DS3_v2** |4   |14  |28  |1000 |16000 |16 / 16x2300 |4 |
|**Standard_DS4_v2** |8   |28  |56  |1000 |32000 |32 / 32x2300 |8 |
|**Standard_DS5_v2** |16  |56  |112 |1000 |64000 |64 / 64x2300 |8 |

### <a name="dv2-series"></a>Serie Dv2
|Size     |vCPU     |Memoria (GiB) | Almacenamiento temporal (GiB)  | Rendimiento de discos del SO máx. (IOPS) | Rendimiento máximo de almacenamiento temporal (IOPS) | Discos de datos máx. / rendimiento (IOPS) | Nº máx. NIC |
|--------------------|----|----|-----|----|-------|-------------|---------|
|**Standard_D11_v2** |2   |14  |100  |500 |6000   |8 / 8x500    |2 |
|**Standard_D12_v2** |4   |28  |200  |500 |12000  |16 / 16x500  |4 |
|**Standard_D13_v2** |8   |56  |400  |500 |24000  |32 / 32x500  |8 |
|**Standard_D14_v2** |16  |112 |800  |500 |48000  |64 / 64x500  |8 |


### <a name="dsv2-series"></a>DSv2-series
|Size     |vCPU     |Memoria (GiB) | Almacenamiento temporal (GiB)  | Rendimiento de discos del SO máx. (IOPS) | Rendimiento máximo de almacenamiento temporal (IOPS) | Discos de datos máx. / rendimiento (IOPS) | Nº máx. NIC |
|---------------------|----|----|-----|-----|-------|--------------|---------|
|**Standard_DS11_v2** |2   |14  |28   |1000 |8000   |4 / 4x2300    |2 |
|**Standard_DS12_v2** |4   |28  |56   |1000 |16000  |8 / 8x2300    |4 |
|**Standard_DS13_v2** |8   |56  |112  |1000 |32000  |16 / 16x2300  |8 |
|**Standard_DS14_v2** |16  |112 |224  |1000 |64000  |32 / 32x2300  |8 |

Para más información, vaya a los [tamaños de máquina virtual de uso general de la serie Dv2](../articles/virtual-machines/dv2-dsv2-series.md#dv2-series).
