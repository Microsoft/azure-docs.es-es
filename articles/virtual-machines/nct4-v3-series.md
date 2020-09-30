---
title: Serie NCas T4 v3
description: Especificaciones de las VM de la serie NCas T4 v3.
services: virtual-machines
ms.subservice: sizes
author: vikancha-MSFT
ms.service: virtual-machines
ms.topic: conceptual
ms.date: 08/10/2020
ms.author: vikancha
ms.openlocfilehash: af9f7eb21d533bc5fb365e7cbf1fb8fc18184fa7
ms.sourcegitcommit: 3246e278d094f0ae435c2393ebf278914ec7b97b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89375234"
---
# <a name="ncast4_v3-series-in-preview"></a>Serie NCasT4_v3 (en versión preliminar) 

Las máquinas virtuales de la serie NCas T4_v3 usan la tecnología de las GPU [Nvidia Tesla T4](https://www.nvidia.com/en-us/data-center/tesla-t4/) y las CPU AMD EPYC 7V12 (Rome). Las VM incluyen hasta 4 GPU de NVIDIA T4 con 16 GB de memoria cada una, hasta 64 núcleos de procesador de AMD EPYC 7V12 (Rome) sin multiproceso, y 440 GiB de memoria del sistema. Estas máquinas virtuales son ideales para ejecutar cargas de trabajo de aprendizaje automático e inteligencia artificial con CUDA, TensorFlow, Pytorch, Caffe y otros marcos, o las cargas de trabajo gráficas mediante la tecnología de NVIDIA GRID. La serie NCasT4_v3 es idónea para ejecutar cargas de trabajo de inferencia.

Puede [enviar una solicitud](https://aka.ms/NCT4v3Preview) para que forme parte del programa de versión preliminar.

<br>

ACU: 230-260

Premium Storage:  Compatible

Almacenamiento en caché de Premium Storage:  Compatible

Migración en vivo: No compatible

Actualizaciones con conservación de memoria: No compatible

| Size | vCPU | Memoria: GiB | GiB de almacenamiento temporal (SSD) | GPU | Memoria de GPU: GiB | Discos de datos máx. | Nº máx. NIC |
| --- | --- | --- | --- | --- | --- | --- | --- |
| Standard_NC4as_T4_v3 |4 |28 |180 | 1 | 16 | 8 | 2 |
| Standard_NC8as_T4_v3 |8 |56 |360 | 1 | 16 | 16 | 4  |
| Standard_NC16as_T4_v3 |16 |110 |360 | 1 | 16 | 32 | 8  |
| Standard_NC64as_T4_v3 |64 |440 |2880 | 4 | 64 | 32 | 8  |


[!INCLUDE [virtual-machines-common-sizes-table-defs](../../includes/virtual-machines-common-sizes-table-defs.md)]

## <a name="supported-operating-systems-and-drivers"></a>Sistemas operativos y controladores compatibles

Para aprovechar las funcionalidades de GPU de las VM de la serie NCasT4_v3 de Azure que ejecutan Windows o Linux, deben instalarse controladores de GPU de NVIDIA.

Para instalar manualmente los controladores de GPU de NVIDIA, consulte [Instalación de controladores de GPU de la serie N para Windows](./windows/n-series-driver-setup.md) para obtener información sobre los sistemas operativos y controladores compatibles, así como sobre los pasos de instalación y verificación.

## <a name="other-sizes"></a>Otros tamaños

- [Uso general](sizes-general.md)
- [Memoria optimizada](sizes-memory.md)
- [Almacenamiento optimizado](sizes-storage.md)
- [GPU optimizada](sizes-gpu.md)
- [Proceso de alto rendimiento](sizes-hpc.md)
- [Generaciones anteriores](sizes-previous-gen.md)

## <a name="next-steps"></a>Pasos siguientes

Obtenga más información sobre cómo las [unidades de proceso de Azure (ACU)](acu.md) pueden ayudarlo a comparar el rendimiento en los distintos SKU de Azure.
