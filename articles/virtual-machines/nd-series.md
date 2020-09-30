---
title: 'Serie ND: Azure Virtual Machines'
description: Especificaciones de las máquinas virtuales de la serie ND.
author: vikancha-MSFT
ms.service: virtual-machines
ms.subservice: sizes
ms.topic: conceptual
ms.date: 02/03/2020
ms.author: jushiman
ms.openlocfilehash: 8c229a72b706333a117645095bd96e44abbdf469
ms.sourcegitcommit: 9c262672c388440810464bb7f8bcc9a5c48fa326
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/03/2020
ms.locfileid: "89418898"
---
# <a name="nd-series"></a>Serie ND

Las máquinas virtuales de serie ND son una novedad incorporada a la familia GPU diseñada para cargas de trabajo inteligencia artificial y aprendizaje profundo. Ofrecen un rendimiento excelente para el aprendizaje y la inferencia. Las instancias de ND funcionan con GPU [NVIDIA Tesla P40](https://images.nvidia.com/content/pdf/tesla/184427-Tesla-P40-Datasheet-NV-Final-Letter-Web.pdf) y CPU Intel Xeon E5-2690 v4 (Broadwell). Estas instancias brindan un rendimiento excelente para operaciones de punto flotante de precisión única, para cargas de trabajo de inteligencia artificial que usan Microsoft Cognitive Toolkit, TensorFlow, Caffe y otros marcos. La serie ND también ofrece una memoria de la GPU de un tamaño muy superior (24 GB), lo que permite adaptarse a modelos de redes neurales mucho más grandes. Al igual que la serie NC, la serie ND presenta una configuración con una baja latencia secundaria, una red de alta productividad mediante RDMA y conectividad InfiniBand para que pueda ejecutar trabajos de aprendizaje a gran escala que abarquen muchas GPU.

Premium Storage: Compatible

Almacenamiento en caché de Premium Storage: Compatible

Migración en vivo: No compatible

Actualizaciones con conservación de memoria: No compatible

> [!IMPORTANT]
> Para esta serie de máquinas virtuales, la cuota de vCPU (núcleo) por región en su suscripción está establecida inicialmente en 0. [Solicite un aumento de cuota de vCPU](../azure-portal/supportability/resource-manager-core-quotas-request.md) para esta serie en una [región donde esté disponible](https://azure.microsoft.com/regions/services/).
>
| Size | vCPU | Memoria: GiB | GiB de almacenamiento temporal (SSD) | GPU | Memoria de GPU: GiB | Discos de datos máx. | Rendimiento máximo del disco sin almacenamiento en la caché: IOPS/Mbps | Nº máx. NIC |
|---|---|---|---|---|---|---|---|---|
| Standard_ND6s    | 6  | 112 | 736  | 1 | 24 | 12 | 20000/200 | 4 |
| Standard_ND12s   | 12 | 224 | 1474 | 2 | 48 | 24 | 40000/400 | 8 |
| Standard_ND24s   | 24 | 448 | 2948 | 4 | 24 | 32 | 80000/800 | 8 |
| Standard_ND24rs* | 24 | 448 | 2948 | 4 | 96 | 32 | 80000/800 | 8 |

1 GPU = una tarjeta de P40.

*Compatible con RDMA

[!INCLUDE [virtual-machines-common-sizes-table-defs](../../includes/virtual-machines-common-sizes-table-defs.md)]

## <a name="supported-operating-systems-and-drivers"></a>Sistemas operativos y controladores compatibles

Para aprovechar las funcionalidades de GPU de las máquinas virtuales de la serie N de Azure, deben instalarse controladores de GPU de NVIDIA.

La [extensión de controlador de GPU de NVIDIA](./extensions/hpccompute-gpu-windows.md) instala los controladores CUDA de NVIDIA o GRID adecuados en una máquina virtual de la serie N. Instale o administre la extensión mediante Azure Portal o con herramientas como las plantillas de Azure PowerShell o Azure Resource Manager. Consulte la [documentación de la extensión de controlador de GPU de NVIDIA](./extensions/hpccompute-gpu-windows.md) para los sistemas operativos compatibles y los pasos de implementación. Para una información general sobre las extensiones de máquina virtual, consulte [Características y extensiones de las máquinas virtuales de Azure](./extensions/overview.md).

Si decide instalar manualmente los controladores de GPU de NVIDIA, consulte el artículo sobre la [instalación de controladores GPU de la serie N para Windows](./windows/n-series-driver-setup.md) o el artículo sobre la [instalación de controladores GPU de la serie N para Linux](./linux/n-series-driver-setup.md) para obtener información sobre los sistemas operativos compatibles, los controladores, la instalación y los pasos de verificación.

## <a name="other-sizes"></a>Otros tamaños

- [Uso general](sizes-general.md)
- [Memoria optimizada](sizes-memory.md)
- [Almacenamiento optimizado](sizes-storage.md)
- [GPU optimizada](sizes-gpu.md)
- [Proceso de alto rendimiento](sizes-hpc.md)
- [Generaciones anteriores](sizes-previous-gen.md)

## <a name="next-steps"></a>Pasos siguientes

Obtenga más información sobre cómo las [unidades de proceso de Azure (ACU)](acu.md) pueden ayudarlo a comparar el rendimiento en los distintos SKU de Azure.
