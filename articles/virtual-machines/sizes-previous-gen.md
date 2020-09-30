---
title: 'Tamaños de máquina virtual de Azure: generaciones anteriores | Microsoft Docs'
description: Se enumeran las generaciones anteriores de tamaños disponibles para máquinas virtuales en Azure. Se proporciona información sobre el número de unidades vCPU, discos de datos y NIC, así como sobre el rendimiento de almacenamiento y el ancho de banda de red para los tamaños de esta serie.
services: virtual-machines
ms.subservice: sizes
author: mimckitt
ms.topic: conceptual
ms.workload: infrastructure-services
ms.date: 02/20/2020
ms.author: jushiman
ms.openlocfilehash: 18389f3058bcc60c22f8598040f4fea4b036f1ee
ms.sourcegitcommit: 58d3b3314df4ba3cabd4d4a6016b22fa5264f05a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89298919"
---
# <a name="previous-generations-of-virtual-machine-sizes"></a>Generaciones anteriores de tamaños de máquina virtual

En esta sección se proporciona información sobre las generaciones anteriores de tamaños de máquina virtual. Se pueden seguir usando estos tamaños, pero hay generaciones más recientes disponibles.

## <a name="f-series"></a>Serie F

La serie F se basa en el procesador Intel Xeon® E5-2673 v3 (Haswell) de 2,4 GHz, que puede alcanzar velocidades de reloj de hasta 3,1 GHz con Intel Turbo Boost Technology 2.0. Este es el mismo rendimiento de CPU que el de la serie Dv2 de máquinas virtuales.  

Las máquinas virtuales de la serie F son una opción excelente para cargas de trabajo que exigen CPU más rápidas, pero que no necesitan tanta memoria o almacenamiento temporal por vCPU.  Las cargas de trabajo, como análisis, servidores de juegos, servidores web y procesamiento por lotes se beneficiarán del valor de la serie F.

ACU: 210 - 250

Premium Storage:  No compatible

Almacenamiento en caché de Premium Storage:  No compatible

| Size | vCPU | Memoria: GiB | GiB de almacenamiento temporal (SSD) | Rendimiento máximo de almacenamiento temporal: IOPS/Mbps de lectura/Mbps de escritura | Discos de datos máx./rendimiento: E/S | Nº máx. de NIC/ancho de banda de red esperado (Mbps) |
|---|---|---|---|---|---|---|
| Standard_F1  | 1  | 2  | 16  | 3000/46/23    | 4/4x500   | 2/750   |
| Standard_F2  | 2  | 4  | 32  | 6000/93/46    | 8/8x500   | 2/1500  |
| Standard_F4  | 4  | 8  | 64  | 12000/187/93  | 16/16x500 | 4/3000  |
| Standard_F8  | 8  | 16 | 128 | 24000/375/187 | 32/32x500 | 8/6000  |
| Standard_F16 | 16 | 32 | 256 | 48000/750/375 | 64/64x500 | 8/12000 |

## <a name="fs-series-sup1sup"></a>Serie Fs <sup>1</sup>

La serie Fs proporciona todas las ventajas de la serie F, además de Premium Storage.

ACU: 210 - 250

Premium Storage:  Compatible

Almacenamiento en caché de Premium Storage:  Compatible

| Size | vCPU | Memoria: GiB | GiB de almacenamiento temporal (SSD) | Discos de datos máx. | Rendimiento máximo de almacenamiento temporal y en caché: IOPS/Mbps (tamaño de caché en GiB) | Rendimiento máximo del disco sin almacenamiento en la caché: IOPS/Mbps | Nº máx. de NIC/ancho de banda de red esperado (Mbps) |
|---|---|---|---|---|---|---|---|
| Standard_F1s  | 1  | 2  | 4  | 4  | 4000/32 (12)    | 3200/48   | 2/750   |
| Standard_F2s  | 2  | 4  | 8  | 8  | 8000/64 (24)    | 6400/96   | 2/1500  |
| Standard_F4s  | 4  | 8  | 16 | 16 | 16000/128 (48)  | 12800/192 | 4/3000  |
| Standard_F8s  | 8  | 16 | 32 | 32 | 32000/256 (96)  | 25600/384 | 8/6000  |
| Standard_F16s | 16 | 32 | 64 | 64 | 64000/512 (192) | 51200/768 | 8/12000 |

MBps = 10^6 bytes por segundo y GiB = 1024^3 bytes.

<sup>1</sup> El rendimiento de disco máximo (IOPS o MBps) posible con una máquina virtual de la serie Fs puede estar limitado por el número, el tamaño y la fragmentación de los discos conectados.  Para información detallada, consulte [Diseño de alto rendimiento](premium-storage-performance.md).


## <a name="nvv2-series"></a>Serie NVv2

**Recomendación de tamaño más reciente**: [Serie NVv3](nvv3-series.md)

Las máquinas virtuales de la serie NVv2 cuentan con la tecnología de las GPU [Nvidia Test M60](https://images.nvidia.com/content/tesla/pdf/188417-Tesla-M60-DS-A4-fnl-Web.pdf) y la tecnología GRID de NVIDIA con CPU Intel Broadwell. Estas máquinas virtuales están orientadas a escritorios virtuales y aplicaciones gráficas aceleradas mediante GPU donde los clientes desean ver sus datos, simular resultados para verlos, trabajar en CAD o representar y transmitir contenido. Además, estas máquinas virtuales pueden ejecutar cargas de trabajo de precisión única, como la codificación y la representación. Las máquinas virtuales NVv2 son compatibles con Premium Storage y traen el doble de memoria del sistema (RAM) si se comparan con la serie NV anterior.  

Cada GPU de las instancias de NVv2 viene con una licencia de GRID. Esta licencia le ofrece flexibilidad para utilizar una instancia de NV como estación de trabajo virtual para un solo usuario; también se pueden conectar 25 usuarios simultáneos a la VM para un escenario de aplicación virtual.

| Size | vCPU | Memoria: GiB | GiB de almacenamiento temporal (SSD) | GPU | Memoria de GPU: GiB | Discos de datos máx. | Nº máx. NIC | Estaciones de trabajo virtuales | Aplicaciones virtuales |
|---|---|---|---|---|---|---|---|---|---|
| Standard_NV6s_v2  | 6  | 112 | 320  | 1 | 8  | 12 | 4 | 1 | 25  |
| Standard_NV12s_v2 | 12 | 224 | 640  | 2 | 16 | 24 | 8 | 2 | 50  |
| Standard_NV24s_v2 | 24 | 448 | 1280 | 4 | 32 | 32 | 8 | 4 | 100 |

[!INCLUDE [virtual-machines-common-sizes-table-defs](../../includes/virtual-machines-common-sizes-table-defs.md)]

## <a name="older-generations-of-virtual-machine-sizes"></a>Generaciones anteriores de tamaños de máquina virtual

En esta sección se proporciona información sobre las generaciones anteriores de tamaños de máquina virtual. Estos tamaños todavía se admiten, pero no recibirán capacidad adicional. Hay tamaños alternativos o versiones más recientes que están disponibles con carácter general. Consulte [Tamaños de las máquinas virtuales Linux en Azure](./sizes.md) para elegir los tamaños de máquina virtual que mejor se ajusten a sus necesidades.  

Para más información sobre cómo cambiar el tamaño de una máquina virtual Linux, consulte [Cambio de tamaño de una máquina virtual Linux](linux/change-vm-size.md).  

<br>

### <a name="basic-a"></a>A básico  

**Recomendación de tamaño más reciente**: [Serie Av2](av2-series.md)

Premium Storage:  No compatible

Almacenamiento en caché de Premium Storage:  No compatible

Los tamaños de niveles básicos se utilizan sobre todo para cargas de trabajo de desarrollo y otras aplicaciones que no requieren equilibrio de carga, escalado automático o máquinas virtuales de uso intensivo de memoria.

| Tamaño – Tamaño\nombre | vCPU | Memoria|NICs (Máx)| Tamaño máximo del disco temporal | Máx. Discos de datos (1023 GB cada uno)| Máx. E/S (300 por disco) |
|---|---|---|---|---|---|---|
| A0\Basic_A0 | 1 | 768 MB  | 2 | 20 GB  | 1  | 1x300  |
| A1\Basic_A1 | 1 | 1,75 GB | 2 | 40 GB  | 2  | 2x300  |
| A2\Basic_A2 | 2 | 3,5 GB  | 2 | 60 GB  | 4  | 4x300  |
| A3\Basic_A3 | 4 | 7 GB    | 2 | 120 GB | 8  | 8x300  |
| A4\Basic_A4 | 8 | 14 GB   | 2 | 240 GB | 16 | 16x300 |

<br>

### <a name="standard-a0---a4-using-cli-and-powershell"></a>Standard_A0 - A4 con CLI y PowerShell

En el modelo de implementación clásica, algunos nombres de tamaños de VM varían ligeramente en la CLI y en PowerShell:

* Standard_A0 es ExtraSmall
* Standard_A1 es Small
* Standard_A2 es Medium
* Standard_A3 es Large
* Standard_A4 es ExtraLarge

### <a name="a-series"></a>Serie A  

**Recomendación de tamaño más reciente**: [Serie Av2](av2-series.md)

ACU: 50-100

Premium Storage:  No compatible

Almacenamiento en caché de Premium Storage:  No compatible

| Size | vCPU | Memoria: GiB | Almacenamiento temporal (HDD): GiB | Discos de datos máx. | Rendimiento de discos de datos máx.: E/S | Nº máx. de NIC/ancho de banda de red esperado (Mbps) |
| --- | --- | --- | --- | --- | --- | --- |
| Standard_A0&nbsp;<sup>1</sup> | 1 | 0,768 | 20 | 1 | 1x500 | 2/100 |
| Standard_A1 | 1 | 1,75 | 70  | 2  | 2 x 500  | 2/500  |
| Standard_A2 | 2 | 3,5  | 135 | 4  | 4x500  | 2/500  |
| Standard_A3 | 4 | 7    | 285 | 8  | 8x500  | 2/1000 |
| Standard_A4 | 8 | 14   | 605 | 16 | 16x500 | 4/2000 |
| Standard_A5 | 2 | 14   | 135 | 4  | 4x500  | 2/500  |
| Standard_A6 | 4 | 28   | 285 | 8  | 8x500  | 2/1000 |
| Standard_A7 | 8 | 56   | 605 | 16 | 16x500 | 4/2000 |

<sup>1</sup> La suscripción del tamaño A0 es excesiva en el hardware físico. Solo en este tamaño específico, las implementaciones de otros clientes podrían afectar el rendimiento de la carga de trabajo en ejecución. A continuación, se indica el rendimiento relativo como la línea base esperada, sujeta a una variabilidad aproximada de 15 por ciento.

<br>

### <a name="a-series---compute-intensive-instances"></a>Serie A: instancias de proceso intensivo  

**Recomendación de tamaño más reciente**: [Serie Av2](av2-series.md)

ACU: 225

Premium Storage:  No compatible

Almacenamiento en caché de Premium Storage:  No compatible

Los tamaños A8-A11 los de la serie H también se conocen como *instancias de proceso intensivo*. El hardware que ejecuta estos tamaños está diseñado y optimizado para aplicaciones de proceso intensivo que consumen muchos recursos de red, incluidas las aplicaciones de clúster de proceso de alto rendimiento (HPC), el modelado y las simulaciones. La serie A8-A11 utiliza Intel Xeon E5-2670 a 2,6 GHz y la serie H, Intel Xeon E5-2667 v3 a 3,2 GHz.  

| Size | vCPU | Memoria: GiB | Almacenamiento temporal (HDD): GiB | Discos de datos máx. | Rendimiento de discos de datos máx.: E/S | Nº máx. NIC|
|---|---|---|---|---|---|---|
| Standard_A8&nbsp;<sup>1</sup> | 8  | 56  | 382 | 32 | 32x500 | 2 |
| Standard_A9&nbsp;<sup>1</sup> | 16 | 112 | 382 | 64 | 64x500 | 4 |
| Standard_A10 | 8  | 56  | 382 | 32 | 32x500 | 2 |
| Standard_A11 | 16 | 112 | 382 | 64 | 64x500 | 4 |

<sup>1</sup>Para las aplicaciones MPI, la red de back-end RDMA dedicada está habilitada por la red InfiniBand FDR, que ofrece una latencia sumamente baja y un alto ancho de banda.  

> [!NOTE]
> Está previsto que las VM A8-A11 se retiren en 3/2021. Para obtener más información, consulte la [guía de migración de HPC](https://azure.microsoft.com/resources/hpc-migration-guide/).

<br>

### <a name="d-series"></a>Serie D  

**Recomendación de tamaño más reciente**: [Serie Dav4](dav4-dasv4-series.md), [serie Dv4](dv4-dsv4-series.md) y [serie Ddv4](ddv4-ddsv4-series.md)

ACU: 160-250 <sup>1</sup>

Premium Storage:  No compatible

Almacenamiento en caché de Premium Storage:  No compatible

| Size | vCPU | Memoria: GiB | GiB de almacenamiento temporal (SSD) | Rendimiento máximo de almacenamiento temporal: IOPS/Mbps de lectura/Mbps de escritura | Discos de datos máx./rendimiento: E/S | Nº máx. de NIC/ancho de banda de red esperado (Mbps) |
|---|---|---|---|---|---|---|
| Standard_D1  | 1 | 3,5 | 50  | 3000/46/23    | 4/4x500   | 2/500  |
| Standard_D2  | 2 | 7   | 100 | 6000/93/46    | 8/8x500   | 2/1000 |
| Standard_D3  | 4 | 14  | 200 | 12000/187/93  | 16/16x500 | 4/2000 |
| Standard_D4  | 8 | 28  | 400 | 24000/375/187 | 32/32x500 | 8/4000 |

<sup>1</sup> La familia de máquinas virtuales se puede ejecutar en una de las CPU siguientes: 2.2 GHz Intel Xeon® E5-2660 v2, 2,4 GHz Intel Xeon® E5-2673 v3 (Haswell) de 2,3 GHz Intel XEON® o E5-2673 v4 (Broadwell)  

<br>

### <a name="d-series---memory-optimized"></a>Serie D: con optimización para memoria  

**Recomendación de tamaño más reciente**: [Serie Dav4](dav4-dasv4-series.md), [serie Dv4](dv4-dsv4-series.md) y [serie Ddv4](ddv4-ddsv4-series.md)

ACU: 160-250 <sup>1</sup>

Premium Storage:  No compatible

Almacenamiento en caché de Premium Storage:  No compatible

| Size | vCPU | Memoria: GiB | GiB de almacenamiento temporal (SSD) | Rendimiento máximo de almacenamiento temporal: IOPS/Mbps de lectura/Mbps de escritura | Discos de datos máx./rendimiento: E/S | Nº máx. de NIC/ancho de banda de red esperado (Mbps) |
|---|---|---|---|---|---|---|
| Standard_D11 | 2  | 14  | 100 | 6000/93/46    | 8/8x500   | 2/1000 |
| Standard_D12 | 4  | 28  | 200 | 12000/187/93  | 16/16x500 | 4/2000 |
| Standard_D13 | 8  | 56  | 400 | 24000/375/187 | 32/32x500 | 8/4000 |
| Standard_D14 | 16 | 112 | 800 | 48000/750/375 | 64/64x500 | 8/8000 |

<sup>1</sup> La familia de máquinas virtuales se puede ejecutar en una de las CPU siguientes: 2.2 GHz Intel Xeon® E5-2660 v2, 2,4 GHz Intel Xeon® E5-2673 v3 (Haswell) de 2,3 GHz Intel XEON® o E5-2673 v4 (Broadwell)  

<br>

## <a name="preview-dc-series"></a>Vista previa: Serie DC

**Recomendación de tamaño más reciente**: [Serie DCsv2](dcv2-series.md)

Premium Storage: Compatible

Almacenamiento en caché de Premium Storage: Compatible

La serie DC usa la última generación de procesadores Intel XEON E-2176G de 3,7 GHz con la tecnología SGX y, con la tecnología Intel Turbo Boost, puede llegar a 4,7 GHz. 

| Size          | vCPU | Memoria: GiB | GiB de almacenamiento temporal (SSD) | Discos de datos máx. | Rendimiento máximo de almacenamiento temporal y en caché: IOPS / MBps (tamaño de caché en GiB) | Rendimiento máximo del disco sin almacenamiento en la caché: IOPS / MBps | Nº máx. de NIC/ancho de banda de red esperado (Mbps) |
|---------------|------|-------------|------------------------|----------------|-------------------------------------------------------------------------|-------------------------------------------|----------------------------------------------|
| Standard_DC2s | 2    | 8           | 100                    | 2              | 4000 / 32 (43)                                                          | 3200 /48                                  | 2 / 1500                                     |
| Standard_DC4s | 4    | 16          | 200                    | 4              | 8000 / 64 (86)                                                          | 6400 /96                                  | 2 / 3000                                     |

> [!IMPORTANT]
>
> Las VM de la serie DC son [VM de 2.ª generación](./linux/generation-2.md#creating-a-generation-2-vm) y solo admiten imágenes de `Gen2`.


### <a name="ds-series"></a>Serie DS  

**Recomendación de tamaño más reciente**: [Serie Dasv4](dav4-dasv4-series.md), [serie Dsv4](dv4-dsv4-series.md) y [serie Ddsv4](ddv4-ddsv4-series.md)

ACU: 160-250 <sup>1</sup>

Premium Storage:  Compatible

Almacenamiento en caché de Premium Storage:  Compatible

| Size | vCPU | Memoria: GiB | GiB de almacenamiento temporal (SSD) | Discos de datos máx. | Rendimiento máximo de almacenamiento temporal y en caché: IOPS/Mbps (tamaño de caché en GiB) | Rendimiento máximo del disco sin almacenamiento en la caché: IOPS/Mbps | Nº máx. de NIC/ancho de banda de red esperado (Mbps) |
|---|---|---|---|---|---|---|---|
| Standard_DS1 | 1 | 3,5 | 7  | 4  | 4000/32 (43)    | 3200/32   | 2/500  |
| Standard_DS2 | 2 | 7   | 14 | 8  | 8000/64 (86)    | 6400/64   | 2/1000 |
| Standard_DS3 | 4 | 14  | 28 | 16 | 16000/128 (172) | 12800/128 | 4/2000 |
| Standard_DS4 | 8 | 28  | 56 | 32 | 32000/256 (344) | 25600/256 | 8/4000 |

<sup>1</sup> La familia de máquinas virtuales se puede ejecutar en una de las CPU siguientes: 2.2 GHz Intel Xeon® E5-2660 v2, 2,4 GHz Intel Xeon® E5-2673 v3 (Haswell) de 2,3 GHz Intel XEON® o E5-2673 v4 (Broadwell)  

<br>

### <a name="ds-series---memory-optimized"></a>Serie DS: con optimización para memoria  

**Recomendación de tamaño más reciente**: [Serie Dasv4](dav4-dasv4-series.md), [serie Dsv4](dv4-dsv4-series.md) y [serie Ddsv4](ddv4-ddsv4-series.md)

ACU: 160-250 <sup>1,2</sup>

Premium Storage:  Compatible

Almacenamiento en caché de Premium Storage:  Compatible

| Size | vCPU | Memoria: GiB | GiB de almacenamiento temporal (SSD) | Discos de datos máx. | Rendimiento máximo de almacenamiento temporal y en caché: IOPS/Mbps (tamaño de caché en GiB) | Rendimiento máximo del disco sin almacenamiento en la caché: IOPS/Mbps | Nº máx. de NIC/ancho de banda de red esperado (Mbps) |
|---|---|---|---|---|---|---|---|
| Standard_DS11 | 2  | 14  | 28  | 8  | 8000/64 (72)    | 6400/64   | 2/1000 |
| Standard_DS12 | 4  | 28  | 56  | 16 | 16000/128 (144) | 12800/128 | 4/2000 |
| Standard_DS13 | 8  | 56  | 112 | 32 | 32000/256 (288) | 25600/256 | 8/4000 |
| Standard_DS14 | 16 | 112 | 224 | 64 | 64000/512 (576) | 51200/512 | 8/8000 |

<sup>1</sup> El rendimiento de disco máx. (E/S por segundo o Mbps) posible con una VM de la serie DS puede estar limitado por el número, el tamaño y la fragmentación de los discos conectados.  Para información detallada, consulte [Diseño de alto rendimiento](premium-storage-performance.md).
<sup>2</sup> La familia de máquinas virtuales se puede ejecutar en una de las CPU siguientes: 2.2 GHz Intel Xeon® E5-2660 v2, 2,4 GHz Intel Xeon® E5-2673 v3 (Haswell) de 2,3 GHz Intel XEON® o E5-2673 v4 (Broadwell)  

<br>

### <a name="ls-series"></a>Serie Ls

**Recomendación de tamaño más reciente**: [Serie Lsv2](lsv2-series.md)

La serie LS ofrece hasta 32 vCPU, con el [procesador Intel® Xeon® de la familia E5 v3](https://www.intel.com/content/www/us/en/processors/xeon/xeon-e5-solutions.html). La serie LS obtiene el mismo rendimiento de CPU que las series G/GS y viene con 8 GiB de memoria por vCPU.

La serie Ls no permite crear una memoria caché local para aumentar el número de IOPS que pueden alcanzar los discos de datos duraderos. El alto rendimiento y el elevado número de IOPS del disco local hace que las máquinas virtuales de la serie Ls sean ideales para almacenes NoSQL, como Apache Cassandra y MongoDB, que replican datos en diferentes máquinas virtuales para garantizar la persistencia en caso de que se produzca un error en una máquina virtual.

ACU: 180-240

Premium Storage:  Compatible

Almacenamiento en caché de Premium Storage:  No compatible

| Size | vCPU | Memoria (GiB) | Almacenamiento temporal (GiB) | Discos de datos máx. | Rendimiento máximo de almacenamiento temporal (IOPS/Mbps) | Rendimiento de disco no almacenado en caché máx. (IOPS/Mbps) | Nº máx. de NIC/ancho de banda de red esperado (Mbps) |
|---|---|---|---|---|---|---|---|
| Standard_L4s   | 4  | 32  | 678  | 16 | 20000/200 | 5000/125  | 2/4000  |
| Standard_L8s   | 8  | 64  | 1388 | 32 | 40000/400 | 10000/250 | 4/8000  |
| Standard_L16s  | 16 | 128 | 2807 | 64 | 80000/800 | 20000/500 | 8/16000 |
| Standard_L32s&nbsp;<sup>1</sup> | 32 | 256 | 5630 | 64 | 160000/1600 | 40000/1000 | 8/20000 |

El rendimiento máximo de disco que es posible con una VM de la serie Ls puede estar limitado por el número, el tamaño y la fragmentación de cualquier disco asociado. Para información detallada, consulte [Diseño de alto rendimiento](premium-storage-performance.md).

<sup>1</sup> La instancia está aislada en el hardware dedicado a un solo cliente.

### <a name="gs-series"></a>Serie GS

**Recomendación de tamaño más reciente**: [Serie Easv4](eav4-easv4-series.md), [serie Esv4](ev4-esv4-series.md), [serie Edsv4](edv4-edsv4-series.md) y [serie M](m-series.md)

ACU: 180 - 240 <sup>1</sup>

Premium Storage:  Compatible

Almacenamiento en caché de Premium Storage:  Compatible

| Size | vCPU | Memoria: GiB | GiB de almacenamiento temporal (SSD) | Discos de datos máx. | Rendimiento máximo de almacenamiento temporal y en caché: IOPS / MBps (tamaño de caché en GiB) | Rendimiento máximo del disco sin almacenamiento en la caché: IOPS/Mbps | Nº máx. de NIC/ancho de banda de red esperado (Mbps) |
|---|---|---|---|---|---|---|---|
| Standard_GS1 | 2 | 28  | 56  | 8  | 10000/100 (264)  | 5000/ 125  | 2/2000 |
| Standard_GS2 | 4 | 56  | 112 | 16 | 20000/200 (528)  | 10000/ 250 | 2/4000 |
| Standard_GS3 | 8 | 112 | 224 | 32 | 40000/400 (1056) | 20000/ 500 | 4/8000 |
| Standard_GS4&nbsp;<sup>3</sup> | 16 | 224 | 448 | 64 | 80000/800 (2112) | 40000/1000 | 8/16000 |
| Standard_GS5&nbsp;<sup>2,&nbsp;3</sup> | 32 | 448 |896 | 64 |160000/1600 (4224) | 80000/2000 | 8/20000 |

<sup>1</sup> El rendimiento de disco máx. (E/S por segundo o Mbps) posible con una VM de la serie GS puede estar limitado por el número, el tamaño y la fragmentación de los discos conectados. Para información detallada, consulte [Diseño de alto rendimiento](premium-storage-performance.md).

<sup>2</sup> La instancia está aislada en el hardware dedicado a un solo cliente.

<sup>3</sup> Tamaños de núcleos restringidos disponibles.

<br>

### <a name="g-series"></a>Serie G

**Recomendación de tamaño más reciente**: [Serie Eav4](eav4-easv4-series.md), [serie Ev4](ev4-esv4-series.md), [serie Edv4](edv4-edsv4-series.md) y [serie M](m-series.md)

ACU: 180 - 240

Premium Storage:  No compatible

Almacenamiento en caché de Premium Storage:  No compatible

| Size | vCPU | Memoria: GiB | GiB de almacenamiento temporal (SSD) | Rendimiento máximo de almacenamiento temporal: IOPS/Mbps de lectura/Mbps de escritura | Discos de datos máx./rendimiento: E/S | Nº máx. de NIC/ancho de banda de red esperado (Mbps) |
|---|---|---|---|---|---|---|
| Standard_G1  | 2  | 28  | 384  | 6000/93/46    | 8/8x500   | 2/2000  |
| Standard_G2  | 4  | 56  | 768  | 12000/187/93  | 16/16x500 | 2/4000  |
| Standard_G3  | 8  | 112 | 1536 | 24000/375/187 | 32/32x500 | 4/8000  |
| Standard_G4  | 16 | 224 | 3072 | 48000/750/375 | 64/64x500 | 8/16000 |
| Standard_G5&nbsp;<sup>1</sup> | 32 | 448 | 6144 | 96000/1500/750| 64/64x500 | 8/20000 |

<sup>1</sup> La instancia está aislada en el hardware dedicado a un solo cliente.
<br>

## <a name="nv-series"></a>Serie NV
**Recomendación de tamaño más reciente**: [serie NVv3](nvv3-series.md) y [serie NVv4](nvv4-series.md)

Las máquinas virtuales de la serie NV dispone de tarjetas GPU [Tesla M60 de NVIDIA](https://images.nvidia.com/content/tesla/pdf/188417-Tesla-M60-DS-A4-fnl-Web.pdf) y tecnología NVIDIA GRID para aplicaciones aceleradas de escritorio y escritorios virtuales donde los clientes pueden visualizar sus datos o simulaciones. Los usuarios pueden visualizar sus flujos de trabajo con muchos gráficos en las instancias de NV para obtener una excelente funcionalidad gráfica y ejecutar, además, cargas de trabajo de precisión únicas, como la codificación y la representación. Las máquinas virtuales de la serie NV también funcionan con CPU Intel Xeon E5-2690 v3 (Haswell).

Cada GPU de instancias de NV viene con una licencia de GRID. Esta licencia le ofrece flexibilidad para utilizar una instancia de NV como estación de trabajo virtual para un solo usuario; también se pueden conectar 25 usuarios simultáneos a la VM para un escenario de aplicación virtual.

Premium Storage:  No compatible

Almacenamiento en caché de Premium Storage:  No compatible

Migración en vivo: No compatible

Actualizaciones con conservación de memoria: No compatible

| Size | vCPU | Memoria: GiB | GiB de almacenamiento temporal (SSD) | GPU | Memoria de GPU: GiB | Discos de datos máx. | Nº máx. NIC | Estaciones de trabajo virtuales | Aplicaciones virtuales |
|---|---|---|---|---|---|---|---|---|---|
| Standard_NV6  | 6  | 56  | 340  | 1 | 8  | 24 | 1 | 1 | 25  |
| Standard_NV12 | 12 | 112 | 680  | 2 | 16 | 48 | 2 | 2 | 50  |
| Standard_NV24 | 24 | 224 | 1440 | 4 | 32 | 64 | 4 | 4 | 100 |

1 GPU = media tarjeta M60.
<br>

## <a name="other-sizes"></a>Otros tamaños

* [Uso general](sizes-general.md)
* [Proceso optimizado](sizes-compute.md)
* [Memoria optimizada](sizes-memory.md)
* [Almacenamiento optimizado](sizes-storage.md)
* [GPU](sizes-gpu.md)
* [Proceso de alto rendimiento](sizes-hpc.md)

## <a name="next-steps"></a>Pasos siguientes

Obtenga más información sobre cómo las [unidades de proceso de Azure (ACU)](acu.md) pueden ayudarlo a comparar el rendimiento en los distintos SKU de Azure.
