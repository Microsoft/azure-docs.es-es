---
title: archivo de inclusión
description: archivo de inclusión
services: virtual-machines
author: jonbeck7
ms.service: virtual-machines
ms.topic: include
ms.date: 10/08/2018
ms.author: azcspmt;jonbeck;cynthn
ms.custom: include file
ms.openlocfilehash: 2864946af6bd9ed2a271ef35d3afb385bfa9a71d
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65815543"
---
Los tamaños de VM de uso general proporcionan una relación equilibrada entre CPU y memoria. Ideal para desarrollo y pruebas, bases de datos pequeñas o medianas, y servidores web de tráfico bajo o medio. En este artículo, se proporciona información acerca del número de vCPU, discos de datos y tarjetas de interfaz de red, así como del rendimiento del almacenamiento para cada tamaño de esta agrupación. 

- La [serie DC](#dc-series) es una nueva familia de máquinas virtuales de Azure que puede ayudar a proteger la confidencialidad y la integridad de los datos y del código mientras se están procesando en la nube pública. Estas máquinas están respaldadas por la última generación del procesador Intel XEON E-2176G a 3,7 GHz con la tecnología SGX. Con la tecnología Intel Turbo Boost, estas máquinas pueden llegar hasta 4,7 GHz. Las instancias de la serie DC permiten a los clientes crear aplicaciones seguras basadas en enclave para proteger sus datos y código mientras se usan.

- Las máquinas virtuales de la serie Av2 se pueden implementar en diversos procesadores y tipos de hardware. Las máquinas virtuales de la serie A tienen las configuraciones de memoria y rendimiento de CPU adecuadas para cargas de trabajo de nivel de entrada como desarrollo y pruebas. Según el hardware, el tamaño es una limitación para ofrecer un rendimiento coherente del procesador para la instancia en ejecución, independientemente del hardware en que se implementó. Con el fin de determinar el hardware físico en que se implementó este tamaño, cree una consulta para el hardware virtual desde dentro de la máquina virtual.

  Algunos casos de uso son, por ejemplo, los servidores de desarrollo y pruebas, los servidores web con poco tráfico, las bases de datos de tamaño pequeño a mediano, las pruebas de concepto y los repositorios de código.

- La serie Dv2, una continuación de la serie D original, presenta una CPU más potente y una configuración óptima de la CPU a la memoria, lo que la hace adecuada para la mayoría de las cargas de trabajo de producción. La CPU de la serie Dv2 es un 35 % aproximadamente más rápida que la CPU de la serie D. Se basa en el más reciente generación Intel Xeon® E5-2673 v3 (Haswell) de 2,4 GHz o procesadores de E5-2673 v4 (Broadwell) de 2,3 GHz; y con Intel Turbo Boost Technology 2.0, puede alcanzar los 3,1 GHz. La serie Dv2 tiene las mismas configuraciones de disco y memoria que la serie D.

- La serie Dv3 ofrece el procesador 2.4 GHz Intel Xeon® E5-2673 v3 (Haswell) o el último procesador 2.3 GHz Intel XEON ® E5-2673 v4 (Broadwell) en una configuración con tecnología Hyper-Threading, que proporcionan una mejor propuesta de valor para la mayoría de las cargas de trabajo de uso general.  Se ha ampliado la memoria (de ~3,5 GiB/vCPU a 4 GiB/vCPU), y los límites de disco y red se han ajustado por núcleo para equipararse con el cambio a hyperthreading.  La serie Dv3 ya no dispone de los tamaños de máquina virtual de memoria alta de las familias D/Dv2; se han trasladado a la nueva familia Ev3.

  Casos de uso de la serie D en el ejemplo incluyen aplicaciones empresariales, bases de datos relacionales, análisis y almacenamiento en caché en memoria. 
  
## <a name="b-series"></a>Serie B

Premium Storage:  Compatible

Caching de Premium Storage:  No admitido

Las máquinas virtuales ampliables de la serie B son idóneas para cargas de trabajo que no necesitan un rendimiento completo de la CPU de forma continua, como los servidores web, pequeñas bases de datos y entornos de desarrollo y de prueba. Estas cargas de trabajo suelen necesitar unos requisitos de rendimiento ampliables. La serie B ofrece a estos clientes la posibilidad de comprar un tamaño de máquina virtual con un rendimiento base sensible al precio y que permita a la instancia de la máquina virtual acumular crédito cuando su rendimiento sea inferior al rendimiento base. Cuando la máquina virtual ha acumulado crédito se puede ampliar por encima de la base de referencia de esta con un uso de hasta un 100% de la CPU si la aplicación necesita el mayor rendimiento posible.

Algunos casos de uso son, por ejemplo, los servidores de desarrollo y pruebas, los servidores web con poco tráfico, las bases de datos pequeñas, los servidores para pruebas de concepto y los servidores de compilación.


| Tamaño             | vCPU  | Memoria: GiB | GiB de almacenamiento temporal (SSD) | Rendimiento base de CPU de la máquina virtual | Rendimiento máximo de CPU de la máquina virtual | Crédito inicial | Créditos ingresados / hora | Créditos máximos ingresados | Discos de datos máx. | Rendimiento máximo de almacenamiento temporal y en caché: IOPS / MBps | Rendimiento máximo del disco sin almacenamiento en la caché: IOPS / MBps | Nº máx. NIC |          
|---------------|-------------|----------------|----------------------------|-----------------------|--------------------|--------------------|--------------------|----------------|----------------------------------------|-------------------------------------------|-------------------------------------------|----------|
| Standard_B1ls<sup>1</sup>  | 1           | 0,5              | 4                          | 5 %                   | 100%                   | 30                   | 3                  | 72            | 2                                      | 200 / 10                                  | 160 / 10                                  | 2  |
| Standard_B1s  | 1           | 1              | 4                          | 10%                   | 100%                   | 30                   | 6                  | 144            | 2                        | 400 / 10                                  | 320 / 10                                  | 2  |
| Standard_B1ms | 1           | 2              | 4                          | 20%                   | 100%                   | 30                   | 12                 | 288           | 2                         | 800 / 10                                  | 640 / 10                                  | 2  |
| Standard_B2s  | 2           | 4              | 8                          | 40%                   | 200%                   | 60                   | 24                 | 576            | 4                                      | 1600 / 15                                 | 1280 / 15                                 | 3  |
| Standard_B2ms | 2           | 8              | 16                         | 60%                   | 200%                   | 60                   | 36                 | 864            | 4                                      | 2400 / 22.5                               | 1920 / 22.5                               | 3  |
| Standard_B4ms | 4           | 16             | 32                         | 90%                   | 400%                   | 120                   | 54                 | 1296           | 8                                      | 3600 / 35                                 | 2880 / 35                                 | 4  |
| Standard_B8ms | 8           | 32             | 64                         | 135%                  | 800%                   | 240                   | 81                 | 1944           | 16                                     | 4320 / 50                                 | 4320 / 50                                 | 4  |

<sup>1</sup> B1ls solo se admite en Linux

## <a name="dsv3-series-sup1sup"></a>Serie Dsv3 <sup>1</sup>

ACU: 160-190

Premium Storage:  Compatible

Caching de Premium Storage:  Compatible

Los tamaños de la serie Dsv3 se basan en el procesador Intel Xeon® E5-2673 v3 (Haswell) de 2,4 GHz o el procesador Intel Xeon® E5-2673 v4 (Broadwell) de 2,3 GHz más reciente que pueden llegar a 3,5 GHz con la tecnología Intel Turbo Boost Technology 2.0 y el uso de Premium Storage. Los tamaños de la serie Dsv3 ofrecen una combinación de vCPU, memoria y almacenamiento local adecuados para la mayoría de las cargas de trabajo de producción.


| Tamaño             | vCPU | Memoria: GiB | GiB de almacenamiento temporal (SSD) | Discos de datos máx. | Rendimiento máximo de almacenamiento temporal y en caché: IOPS / MBps (tamaño de caché en GiB) | Rendimiento máximo del disco sin almacenamiento en la caché: IOPS / MBps | Nº máx. de NIC/ancho de banda de red esperado (Mbps) |
|------------------|--------|-------------|----------------|----------------|-----------------------------------------------------------------------|-------------------------------------------|------------------------------------------------|
| Standard_D2s_v3  | 2      | 8           | 16             | 4              | 4 000 / 32 (50)                                                       | 3200 / 48                                | 2 / 1000                                   |
| Standard_D4s_v3  | 4      | 16          | 32             | 8              | 8 000 / 64 (100)                                                      | 6400 / 96                                | 2 / 2000                                   |
| Standard_D8s_v3  | 8      | 32          | 64             | 16             | 16 000 / 128 (200)                                                    | 12 800 / 192                              | 4 / 4000                                      |
| Standard_D16s_v3 | 16     | 64          | 128            | 32             | 32 000 / 256 (400)                                                    | 25 600 / 384                              | 8 / 8000                                      |
| Standard_D32s_v3 | 32     | 128          | 256            | 32             | 64 000 / 512 (800)                                                    | 51 200 / 768                              | 8 / 16 000                                               |
| Standard_D64s_v3 | 64     | 256          | 512            | 32             | 128,000 / 1024 (1600)                                                    | 80 000 / 1200                              | 8 / 30 000                                               |

<sup>1</sup> Las máquinas virtuales de la serie Dsv3 cuentan con la tecnología Hyper-Threading de Intel®.

## <a name="dv3-series-sup1sup"></a>Serie Dv3 <sup>1</sup>

ACU: 160-190

Premium Storage:  No admitido

Caching de Premium Storage:  No admitido

Los tamaños de la serie Dv3 se basan en el procesador Intel Xeon® E5-2673 v3 (Haswell) de 2,4 GHz o el procesador Intel Xeon® E5-2673 v4 (Broadwell) de 2,3 GHz que pueden llegar a 3,5 GHz con la tecnología Intel Turbo Boost Technology 2.0. Los tamaños de la serie Dv3 ofrecen una combinación de vCPU, memoria y almacenamiento local adecuados para la mayoría de las cargas de trabajo de producción.

El almacenamiento en disco de datos se factura de forma independiente a las máquinas virtuales. Para usar discos de Premium Storage, utilice los tamaños Dsv3. El precio y los medidores de facturación para los tamaños Dsv3 son los mismos que para la serie Dv3. 


| Tamaño            | vCPU | Memoria: GiB | GiB de almacenamiento temporal (SSD) | Discos de datos máx. | Rendimiento máximo de almacenamiento temporal: IOPS / MBps de lectura / MBps de escritura | Ancho de banda de red/NIC máx. |
|-----------------|-----------|-------------|----------------|----------------|----------------------------------------------------------|------------------------------|
| Standard_D2_v3  | 2         | 8           | 50             | 4              | 3000/46/23                                               | 2 / 1000                    |
| Standard_D4_v3  | 4         | 16          | 100            | 8              | 6000/93/46                                               | 2 / 2000                    |
| Standard_D8_v3  | 8         | 32          | 200            | 16             | 12000/187/93                                             | 4 / 4000                    |
| Standard_D16_v3 | 16        | 64          | 400            | 32             | 24000/375/187                                            | 8 / 8000                    |
| Standard_D32_v3 | 32        | 128          | 800            | 32             | 48000/750/375                                            | 8 / 16 000                             |
| Standard_D64_v3 | 64        | 256          | 1600            | 32             | 96000/1000/500                                            | 8 / 30 000                             |

<sup>1</sup> Las máquinas virtuales de la serie Dv3 cuentan con la tecnología Hyper-Threading de Intel®.

## <a name="dsv2-series"></a>DSv2-series

ACU: 210-250

Premium Storage:  Compatible

Caching de Premium Storage:  Compatible

| Tamaño | vCPU | Memoria: GiB | GiB de almacenamiento temporal (SSD) | Discos de datos máx. | Rendimiento máximo de almacenamiento temporal y en caché: IOPS / MBps (tamaño de caché en GiB) | Rendimiento máximo del disco sin almacenamiento en la caché: IOPS / MBps | Nº máx. de NIC/ancho de banda de red esperado (Mbps) |
| --- | --- | --- | --- | --- | --- | --- | --- |
| Standard_DS1_v2 |1 |3,5 |7 |4 |4000 / 32 (43) |3200 / 48 |2 / 750 |
| Standard_DS2_v2 |2 |7 |14 |8 |8000 / 64 (86) |6400 / 96 |2 / 1500 |
| Standard_DS3_v2 |4 |14 |28 |16 |16 000 / 128 (172) |12 800 / 192 |4 / 3000 |
| Standard_DS4_v2 |8 |28 |56 |32 |32 000 / 256 (344) |25 600 / 384 |8 / 6000 |
| Standard_DS5_v2 |16 |56 |112 |64 |64 000 / 512 (688) |51 200 / 768 |8 / 12 000 |

## <a name="dv2-series"></a>Serie Dv2

ACU: 210-250

Premium Storage:  No admitido

Caching de Premium Storage:  No admitido

| Tamaño           | vCPU | Memoria: GiB | GiB de almacenamiento temporal (SSD) | Rendimiento máximo de almacenamiento temporal: IOPS / MBps de lectura / MBps de escritura | Discos de datos máx. | Procesamiento: E/S | Nº máx. de NIC/ancho de banda de red esperado (Mbps) |
|----------------|------|-------------|------------------------|------------------------------------------------------------|----------------|------------------|----------------------------------------------|
| Standard_D1_v2 | 1    | 3,5         | 50                     | 3000 / 46 / 23                                             | 4              | 4x500            | 2 / 750                                      |
| Standard_D2_v2 | 2    | 7           | 100                    | 6000 / 93 / 46                                             | 8              | 8x500            | 2 / 1500                                     |
| Standard_D3_v2 | 4    | 14          | 200                    | 12000 / 187 / 93                                           | 16             | 16x500           | 4 / 3000                                       |
| Standard_D4_v2 | 8    | 28          | 400                    | 24000 / 375 / 187                                          | 32             | 32x500           | 8 / 6000                                       |
| Standard_D5_v2 | 16   | 56          | 800                    | 48000 / 750 / 375                                          | 64             | 64x500           | 8 / 12 000                                    |

## <a name="av2-series"></a>Serie Av2

ACU: 100

Premium Storage:  No admitido

Caching de Premium Storage:  No admitido


| Tamaño            | vCPU | Memoria: GiB | GiB de almacenamiento temporal (SSD) | Rendimiento máximo de almacenamiento temporal: IOPS / MBps de lectura / MBps de escritura | Discos de datos máx. / rendimiento: E/S | Nº máx. de NIC/ancho de banda de red esperado (Mbps) | 
|-----------------|-----------|-------------|----------------|----------------------------------------------------------|-----------------------------------|------------------------------|
| Standard_A1_v2  | 1         | 2           | 10             | 1000 / 20 / 10                                           | 2 / 2x500               | 2 / 250                 |
| Standard_A2_v2  | 2         | 4           | 20             | 2000 / 40 / 20                                           | 4 / 4x500               | 2 / 500                 |
| Standard_A4_v2  | 4         | 8           | 40             | 4000 / 80 / 40                                           | 8 / 8x500               | 4 / 1000                     |
| Standard_A8_v2  | 8         | 16          | 80             | 8000 / 160 / 80                                          | 16 / 16x500             | 8 / 2000                     |
| Standard_A2m_v2 | 2         | 16          | 20             | 2000 / 40 / 20                                           | 4 / 4x500               | 2 / 500                 |
| Standard_A4m_v2 | 4         | 32          | 40             | 4000 / 80 / 40                                           | 8 / 8x500               | 4 / 1000                     |
| Standard_A8m_v2 | 8         | 64          | 80             | 8000 / 160 / 80                                          | 16 / 16x500             | 8 / 2000                     |

## <a name="dc-series"></a>Serie DC

Premium Storage: Compatible

Caching de Premium Storage: Compatible



| Tamaño          | vCPU | Memoria: GiB | GiB de almacenamiento temporal (SSD) | Discos de datos máx. | Rendimiento máximo de almacenamiento temporal y en caché: IOPS / MBps (tamaño de caché en GiB) | Rendimiento máximo del disco sin almacenamiento en la caché: IOPS / MBps | Nº máx. de NIC/ancho de banda de red esperado (Mbps) |
|---------------|------|-------------|------------------------|----------------|-------------------------------------------------------------------------|-------------------------------------------|----------------------------------------------|
| Standard_DC2s | 2    | 8           | 100                    | 2              | 4000 / 32 (43)                                                          | 3200 /48                                  | 2 / 1500                                     |
| Standard_DC4s | 4    | 16          | 200                    | 4              | 8000 / 64 (86)                                                          | 6400 /96                                  | 2 / 3000                                     |







