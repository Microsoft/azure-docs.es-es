---
title: Series Ev3 y Esv3
description: Especificaciones de las máquinas virtuales de las series Ev3 y Esv3.
author: joelpelley
ms.service: virtual-machines
ms.subservice: sizes
ms.topic: conceptual
ms.date: 09/22/2020
ms.author: lahugh
ms.openlocfilehash: 2fe09d785d282b0f3954e2285b15458c867a9401
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90906187"
---
# <a name="ev3-and-esv3-series"></a>Series Ev3 y Esv3

Las series Ev3 y Esv3 se ejecutan en el procesador Intel® Xeon® Platinum 8272CL (Cascade Lake), Intel® Xeon® 8171M 2.1 GHz (Skylake) o Intel® Xeon® E5-2673 v4 2.3 GHz (Broadwell) en una configuración de Hyper-Threading. Gracias a esto, proporcionan una mejor propuesta de valor para la mayoría de las cargas de trabajo de uso general y la serie v4 se equipara con las máquinas virtuales de uso general de la mayoría de las demás nubes.  Se ha ampliado la memoria (de 7 GiB/vCPU a 8 GiB/vCPU), y los límites de disco y red se han ajustado por núcleo para equipararse con el cambio a hyperthreading. La serie Ev3 es la continuación de los tamaños de máquina virtual de memoria alta de las familias D/Dv2.

## <a name="ev3-series"></a>Serie Ev3

Las instancias de la serie Ev3 se ejecutan en procesadores Intel® Xeon® Platinum 8272CL (Cascade Lake), Intel® Xeon® 8171M 2,1 GHz (Skylake) o Intel® Xeon® E5-2673 V4 2,3 GHz (Broadwell), además de la tecnología Intel Turbo Boost 2.0. Las instancias de la serie Ev3 son ideales para aplicaciones empresariales de uso intensivo de memoria.

El almacenamiento en disco de datos se factura de forma independiente a las máquinas virtuales. Para usar discos de Premium Storage, utilice los tamaños ESv3. Los precios y los medidores de facturación para los tamaños ESv3 son los mismos que para la serie Ev3.

Las máquinas virtuales de la serie Ev3 cuentan con la tecnología Hyper-Threading de Intel®.

ACU: 160 - 190

Premium Storage:  No compatible

Almacenamiento en caché de Premium Storage:  No compatible

Migración en vivo: Compatible

Actualizaciones con conservación de memoria: Compatible

| Size | vCPU | Memoria: GiB | GiB de almacenamiento temporal (SSD) | Discos de datos máx. | Rendimiento máximo de almacenamiento temporal: IOPS / MBps de lectura / MBps de escritura | Ancho de banda de red/NIC máx. |
|---|---|---|---|---|---|---|
| Standard_E2_v3  | 2  | 16  | 50   | 4  | 3000/46/23     | 2/1000  |
| Standard_E4_v3  | 4  | 32  | 100  | 8  | 6000/93/46     | 2/2000  |
| Standard_E8_v3  | 8  | 64  | 200  | 16 | 12000/187/93   | 4/4000  |
| Standard_E16_v3 | 16 | 128 | 400  | 32 | 24000/375/187  | 8/8000  |
| Standard_E20_v3 | 20 | 160 | 500  | 32 | 30000/469/234  | 8/10000 |
| Standard_E32_v3 | 32 | 256 | 800  | 32 | 48000/750/375  | 8/16000 |
| Standard_E48_v3 | 48 | 384 | 1200 | 32 | 96000/1000/500 | 8/24000 |
| Standard_E64_v3 | 64 | 432 | 1600 | 32 | 96000/1000/500 | 8/30000 |
| Standard_E64i_v3 <sup>1,2</sup> | 64 | 432 | 1600 | 32 | 96000/1000/500 | 8/30000 |

<sup>1</sup> Tamaños de núcleos restringidos disponibles.

<sup>2</sup> La instancia está aislada en el hardware dedicado a un solo cliente.

## <a name="esv3-series"></a>Serie Esv3

Las instancias de la serie Esv3 se ejecutan en procesadores Intel® Xeon® Platinum 8272CL (Cascade Lake), Intel® Xeon® 8171M 2,1 GHz (Skylake) o Intel® Xeon® E5-2673 V4 2,3 GHz (Broadwell), además de la tecnología Intel Turbo Boost 2.0, y utilizan Premium Storage. Las instancias de la serie Esv3 son ideales para aplicaciones empresariales de uso intensivo de memoria.

Las máquinas virtuales de la serie Esv3 cuentan con la tecnología Hyper-Threading de Intel®.

ACU: 160-190

Premium Storage:  Compatible

Almacenamiento en caché de Premium Storage:  Compatible

Migración en vivo: Compatible

Actualizaciones con conservación de memoria: Compatible

| Size | vCPU | Memoria: GiB | GiB de almacenamiento temporal (SSD) | Discos de datos máx. | Rendimiento máximo de almacenamiento temporal y en caché: IOPS/Mbps (tamaño de caché en GiB) | Rendimiento de almacenamiento en caché y almacenamiento temporal expandidos: IOPS/MBps<sup>3</sup> | Rendimiento máximo del disco sin almacenamiento en la caché: IOPS/Mbps |  Rendimiento del disco sin almacenamiento en la caché expandido: IOPS/MBps<sup>3</sup>| Nº máx. de NIC/ancho de banda de red esperado (Mbps) |
|---|---|---|---|---|---|---|---|---|---|
| Standard_E2s_v3                | 2  | 16  | 32  | 4  | 4000/32 (50)       | 4000/100    | 3200/48    | 4000/100 | 2/1000 |
| Standard_E4s_v3 <sup>1</sup>   | 4  | 32  | 64  | 8  | 8000/64 (100)      | 8000/200    | 6400/96    | 8000/200 | 2/2000 |
| Standard_E8s_v3 <sup>1</sup>   | 8  | 64  | 128 | 16 | 16000/128 (200)    | 16 000/400   | 12800/192  | 16 000/400 | 4/4000 |
| Standard_E16s_v3 <sup>1</sup>  | 16 | 128 | 256 | 32 | 32000/256 (400)    | 32 000/800   | 25600/384  | 32 000/800 | 8/8000 |
| Standard_E20s_v3               | 20 | 160 | 320 | 32 | 40000/320 (400)    | 40000/1000  | 32000/480  | 40000/1000 | 8/10000 |
| Standard_E32s_v3 <sup>1</sup>  | 32 | 256 | 512 | 32 | 64000/512 (800)    | 64 000/1600  | 51200/768  | 64 000/1600 | 8/16000 |
| Standard_E48s_v3 <sup>1</sup>  | 48 | 384 | 768 | 32 | 96000/768 (1200)   | 96 000/2000  | 76800/1152 | 80000/2000 | 8/24000 |
| Standard_E64s_v3 <sup>1</sup>  | 64 | 432 | 864 | 32 | 128000/1024 (1600) | 128 000/2000 | 80000/1200 | 80000/2000 | 8/30000 |
| Standard_E64is_v3 <sup>2</sup> | 64 | 432 | 864 | 32 | 128000/1024 (1600) | 128 000/2000 | 80000/1200 | 80000/2000 | 8/30000 |

<sup>1</sup> Tamaños de núcleos restringidos disponibles.

<sup>2</sup> La instancia está aislada en el hardware dedicado a un solo cliente.

<sup>3</sup> Las máquinas virtuales de la serie Esv3 pueden [expandir](linux/disk-bursting.md) su rendimiento de disco y llegar a una expansión máxima de hasta 30 minutos cada vez.

[!INCLUDE [virtual-machines-common-sizes-table-defs](../../includes/virtual-machines-common-sizes-table-defs.md)]

## <a name="other-sizes-and-information"></a>Otros tamaños e información

- [Uso general](sizes-general.md)
- [Memoria optimizada](sizes-memory.md)
- [Almacenamiento optimizado](sizes-storage.md)
- [GPU optimizada](sizes-gpu.md)
- [Proceso de alto rendimiento](sizes-hpc.md)
- [Generaciones anteriores](sizes-previous-gen.md)
- [Calculadora de precios](https://azure.microsoft.com/pricing/calculator/)
- Para obtener más información sobre los tipos de discos, vea [¿Qué tipos de disco están disponibles en Azure?](disks-types.md).

## <a name="next-steps"></a>Pasos siguientes

Obtenga más información sobre cómo las [unidades de proceso de Azure (ACU)](acu.md) pueden ayudarlo a comparar el rendimiento en los distintos SKU de Azure.