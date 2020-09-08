---
title: 'Serie DC: Azure Virtual Machines'
description: Especificaciones de las máquinas virtuales de la serie DC.
author: susaxen
ms.service: virtual-machines
ms.subservice: sizes
ms.topic: conceptual
ms.date: 02/20/2020
ms.author: jushiman
ms.openlocfilehash: 41dbb42637db98dcd97e3487f252e88f2ba5f9a3
ms.sourcegitcommit: 271601d3eeeb9422e36353d32d57bd6e331f4d7b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88649897"
---
# <a name="dcsv2-series"></a>Serie DCsv2


La serie DCsv2 puede ayudar a proteger la confidencialidad y la integridad de los datos y del código mientras se están procesando en la nube pública. Estas máquinas están respaldadas por la última generación del procesador Intel XEON E-2288G con la tecnología SGX. Con la tecnología Intel Turbo Boost, estas máquinas pueden llegar hasta 5,0 GHz. Las instancias de la serie DCsv2 permiten a los clientes compilar aplicaciones seguras basadas en enclave para proteger su código y sus datos mientras se usan.

Entre los casos de uso de ejemplo se incluyen el uso compartido de datos confidenciales entre varias partes, la detección de fraudes, el blanqueo de dinero, la cadena de bloques, el análisis de uso confidencial, el análisis de la inteligencia y el aprendizaje automático confidencial.

Premium Storage: Admitido*

Almacenamiento en caché de Premium Storage: Admitido*

Migración en vivo: No compatible

Actualizaciones con conservación de memoria: No compatible

*Excepto para Standard_DC8_v2



| Size             | vCPU | Memoria: GiB | GiB de almacenamiento temporal (SSD) | Discos de datos máx. | Rendimiento máximo de almacenamiento temporal y en caché: IOPS / MBps (tamaño de caché en GiB) | Nº máx. de NIC / ancho de banda de red esperado (MBps) | Memoria EPC (MiB) |
|------------------|------|-------------|------------------------|----------------|-------------------------------------------------------------------------|----------------------------------------------|---------------------|
| Standard_DC1s_v2 | 1    | 4           | 50                     | 1              | 2000/16                                                                                               | 2   | 28                                      |
| Standard_DC2s_v2 | 2    | 8           | 100                    | 2              | 4000/32                                                                                               | 2  | 56                                          |
| Standard_DC4s_v2 | 4    | 16          | 200                    | 4              | 8000/64                                                                                               | 2  | 112                                          |
| Standard_DC8_v2  | 8   | 32          | 400                    | 8              | 16000/128                                                                                         | 2   | 168                                         |

- Las VM de la serie DCsv2 son [VM de 2.ª generación](./linux/generation-2.md#creating-a-generation-2-vm) y solo admiten imágenes de `Gen2`.
- Actualmente está disponible en las regiones enumeradas [aquí](https://azure.microsoft.com/global-infrastructure/services/?products=virtual-machines&regions=all).
- Generación anterior de máquinas virtuales de proceso confidencial: [Serie DC](sizes-previous-gen.md#preview-dc-series)
- Creación de máquinas virtuales DCsv2 mediante [Azure Portal](./linux/quick-create-portal.md) o [Azure Marketplace](https://azuremarketplace.microsoft.com/marketplace/apps/microsoft-azure-compute.acc-virtual-machine-v2?tab=overview)



## <a name="other-sizes-and-information"></a>Otros tamaños e información

- [Uso general](sizes-general.md)
- [Memoria optimizada](sizes-memory.md)
- [Almacenamiento optimizado](sizes-storage.md)
- [GPU optimizada](sizes-gpu.md)
- [Proceso de alto rendimiento](sizes-hpc.md)
- [Generaciones anteriores](sizes-previous-gen.md)

Calculadora de precios: [Calculadora de precios](https://azure.microsoft.com/pricing/calculator/)

Más información sobre los tipos de disco: [Tipos de disco](./disks-types.md#ultra-disk)

## <a name="next-steps"></a>Pasos siguientes

Obtenga más información sobre cómo las [unidades de proceso de Azure (ACU)](acu.md) pueden ayudarlo a comparar el rendimiento en los distintos SKU de Azure.