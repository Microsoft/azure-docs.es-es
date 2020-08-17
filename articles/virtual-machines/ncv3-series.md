---
title: 'Serie NCv3: Azure Virtual Machines'
description: Especificaciones de las máquinas virtuales de la serie NCv3.
ms.subservice: sizes
author: vikancha-MSFT
ms.service: virtual-machines
ms.topic: conceptual
ms.date: 02/03/2020
ms.author: jushiman
ms.openlocfilehash: de118f7a4a29d306fe351199795a5a8cacb38fcd
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87289746"
---
# <a name="ncv3-series"></a>Serie NCv3

Las máquinas virtuales de la serie NCv3 disponen de la tecnología de las GPU NVIDIA Tesla V100. Estas GPU pueden aumentar 1,5 veces el rendimiento del trabajo de computación de la serie NCv2. Los clientes pueden aprovechar estas GPU actualizadas para cargas de trabajo de HPC tradicionales, como la creación de modelos de embalses, la secuenciación de ADN, el análisis de proteínas, la realización de simulaciones Monte Carlo y otras. La configuración NC24rs v3 proporciona una interfaz de red de baja latencia y alto rendimiento optimizada para cargas de trabajo de computación paralelas estrechamente unidas. Además de las GPU, las máquinas virtuales de la serie NCv3 también están equipadas con CPU Intel Xeon E5-2690 v4 (Broadwell).

Premium Storage:  Compatible

Almacenamiento en caché de Premium Storage:  Compatible

Migración en vivo: No compatible

Actualizaciones con conservación de memoria: No compatible

> [!IMPORTANT]
> Para esta serie de máquinas virtuales, la cuota de vCPU (núcleo) en su suscripción está establecida inicialmente en 0 en cada región. [Solicite un aumento de cuota de vCPU](../azure-portal/supportability/resource-manager-core-quotas-request.md) para esta serie en una [región donde esté disponible](https://azure.microsoft.com/regions/services/).
>
| Size | vCPU | Memoria: GiB | GiB de almacenamiento temporal (SSD) | GPU | Memoria de GPU: GiB | Discos de datos máx. | Rendimiento máximo del disco sin almacenamiento en la caché: IOPS/Mbps | Nº máx. NIC |
|---|---|---|---|---|---|---|---|---|
| Standard_NC6s_v3    | 6  | 112 | 736  | 1 | 16 | 12 | 20000/200 | 4 |
| Standard_NC12s_v3   | 12 | 224 | 1474 | 2 | 32 | 24 | 40000/400 | 8 |
| Standard_NC24s_v3   | 24 | 448 | 2948 | 4 | 64 | 32 | 80000/800 | 8 |
| Standard_NC24rs_v3* | 24 | 448 | 2948 | 4 | 64 | 32 | 80000/800 | 8 |

1 GPU = una tarjeta V100

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
