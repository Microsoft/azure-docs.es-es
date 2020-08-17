---
title: Puntuaciones de pruebas comparativas de proceso para máquinas virtuales Linux de Azure
description: Comparación de puntuaciones de pruebas comparativas de proceso de CoreMark para máquinas virtuales de Azure con Linux.
ms.service: virtual-machines-linux
ms.topic: conceptual
ms.date: 04/29/2020
ms.reviewer: davberg
ms.openlocfilehash: efbd90eee8d072c1979565209c8c04a4ee5c55f9
ms.sourcegitcommit: 2ff0d073607bc746ffc638a84bb026d1705e543e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87830536"
---
# <a name="compute-benchmark-scores-for-linux-vms"></a>Puntuaciones de pruebas comparativas de proceso para máquinas virtuales Linux
Las siguientes puntuaciones de pruebas comparativas CoreMark muestran el rendimiento de proceso para la alineación de máquinas virtuales de alto rendimiento de Azure con Ubuntu. Las puntuaciones de pruebas comparativas de proceso también están disponibles para las [máquinas virtuales de Windows](../windows/compute-benchmark-scores.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json).

## <a name="standard_das_v4"></a>Standard_Das_v4
(11/12/2019 2:28:52 AM pbi 5851281)

|Tamaño de VM| CPU | vCPU | Nodos NUMA | Memoria (GiB) | Puntuación media | Desarrollo estándar | % desarrollo estándar | N. º de ejecuciones |
|---| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
|Standard_D2as_v4| Procesador de 32 núcleos AMD EPYC 7452 | 2 | 1 | 7.8 | 29 726 | 693 | 2,33 % | 42 |
|Standard_D4as_v4| Procesador de 32 núcleos AMD EPYC 7452 | 4 | 1 | 15,7 | 59 224 | 1595 | 2,69 % | 42 |
|Standard_D8as_v4| Procesador de 32 núcleos AMD EPYC 7452 | 8 | 1 | 31,4 | 116 412 | 3 613 | 3,10 % | 42 |
|Standard_D16as_v4| Procesador de 32 núcleos AMD EPYC 7452 | 16 | 2 | 62,9 | 229 489 | 7 209 | 3,14 % | 35 |
|Standard_D32as_v4| Procesador de 32 núcleos AMD EPYC 7452 | 32 | 4 | 125,9 | 461 916 | 6 746 | 1,46 % | 35 |

## <a name="standard_da_v4"></a>Standard_Da_v4
(12/12/2019 12:01:48 AM pbi 5851281)

| Tamaño de VM | CPU | vCPU | Nodos NUMA | Memoria (GiB) | Puntuación media | Desarrollo estándar | % desarrollo estándar | N. º de ejecuciones |
| --- | --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Standard_D2a_v4 | Procesador de 32 núcleos AMD EPYC 7452 | 2 | 1 | 7.8 | 30 023 | 333 | 1,11 % | 35 |
| Standard_D4a_v4 | Procesador de 32 núcleos AMD EPYC 7452 | 4 | 1 | 15,7 | 59 685 | 1 141 | 1,91 % | 77 |
| Standard_D8a_v4 | Procesador de 32 núcleos AMD EPYC 7452 | 8 | 1 | 31,4 | 118 346 | 1 130 | 0,95 % | 42 |
| Standard_D16a_v4 | Procesador de 32 núcleos AMD EPYC 7452 | 16 | 2 | 62,9 | 231 131 | 3 830 | 1,66 % | 35 |
| Standard_D32a_v4 | Procesador de 32 núcleos AMD EPYC 7452 | 32 | 4 | 125,9 | 457 266 | 10 208 | 2,23 % | 35 |
| Standard_D48a_v4 | Procesador de 32 núcleos AMD EPYC 7452 | 48 | 6 | 188,9 | 664 078 | 17 241 | 2,60 % | 35 |
| Standard_D64a_v4 | Procesador de 32 núcleos AMD EPYC 7452 | 64 | 8 | 251,9 | 863 911 | 24 818 | 2,87 % | 35 |
| Standard_D96a_v4 | Procesador de 32 núcleos AMD EPYC 7452 | 96 | 12 | 377,9 | 1 290 455 | 13 640 | 1,06 % | 35 |

## <a name="standard_eas_v4"></a>Standard_Eas_v4
(11/12/2019 2:28:50 AM pbi 5851281)

| Tamaño de VM | CPU | vCPU | Nodos NUMA | Memoria (GiB) | Puntuación media | Desarrollo estándar | % desarrollo estándar | N. º de ejecuciones |
| --- | --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Standard_E2as_v4 | Procesador de 32 núcleos AMD EPYC 7452 | 2 | 1 | 15,7 | 29 217 | 654 | 2,24 % | 42 |
| Standard_E4as_v4 | Procesador de 32 núcleos AMD EPYC 7452 | 4 | 1 | 31,4 | 58 356 | 480 | 0,82 % | 42 |
| Standard_E8as_v4 | Procesador de 32 núcleos AMD EPYC 7452 | 8 | 1 | 62,9 | 115 943 | 3 526 | 3,04 % | 35 |
| Standard_E16as_v4 | Procesador de 32 núcleos AMD EPYC 7452 | 16 | 2 | 125,9 | 227 383 | 5 619 | 2,47 % | 35 |
| Standard_E32as_v4 | Procesador de 32 núcleos AMD EPYC 7452 | 32 | 4 | 251,9 | 454 609 | 12 746 | 2,80 % | 35 |
| Standard_E48as_v4 | Procesador de 32 núcleos AMD EPYC 7452 | 48 | 6 | 377,9 | 682 769 | 9 257 | 1,36 % | 35 |
| Standard_E64as_v4 | Procesador de 32 núcleos AMD EPYC 7452 | 64 | 8 | 503,9 | 881 311 | 28 357 | 3,22 % | 35 |
| Standard_E96as_v4 | Procesador de 32 núcleos AMD EPYC 7452 | 96 | 12 | 661,4 | 1 299 233 | 14 997 | 1,15 % | 70 |

##  <a name="standard_ea_v4"></a>Standard_Ea_v4
(11/12/2019 2:29:06 AM pbi 5851281)

| Tamaño de VM | CPU | vCPU | Nodos NUMA | Memoria (GiB) | Puntuación media | Desarrollo estándar | % desarrollo estándar | N. º de ejecuciones |
| --- | --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Standard_E2a_v4 | Procesador de 32 núcleos AMD EPYC 7452 | 2 | 1 | 15,7 | 29 561 | 422 | 1,43 % | 42 |
| Standard_E4a_v4 | Procesador de 32 núcleos AMD EPYC 7452 | 4 | 1 | 31,4 | 58 303 | 1 280 | 2,20 % | 42 |
| Standard_E8a_v4 | Procesador de 32 núcleos AMD EPYC 7452 | 8 | 1 | 62,9 | 114 650 | 2 726 | 2,38 % | 42 |
| Standard_E16a_v4 | Procesador de 32 núcleos AMD EPYC 7452 | 16 | 2 | 125,9 | 226 947 | 4 661 | 2,05 % | 35 |
| Standard_E32a_v4 | Procesador de 32 núcleos AMD EPYC 7452 | 32 | 4 | 251,9 | 453 666 | 10 058 | 2,22 % | 42 |
| Standard_E48a_v4 | Procesador de 32 núcleos AMD EPYC 7452 | 48 | 6 | 377,9 | 665 200 | 18 714 | 2,81 % | 35 |
| Standard_E64a_v4 | Procesador de 32 núcleos AMD EPYC 7452 | 64 | 8 | 503,9 | 894 718 | 25 214 | 2,82 % | 35 |
| Standard_E96a_v4 | Procesador de 32 núcleos AMD EPYC 7452 | 96 | 12 | 661,4 | 1 298 074 | 15 948 | 1,23 % | 35 |

## <a name="av2---general-compute"></a>Av2: proceso general
(15/03/2019 12:06:55 AM pbi 3897709)

| Tamaño de VM | CPU | vCPU | Nodos NUMA | Memoria (GiB) | Puntuación media | Desarrollo estándar | % desarrollo estándar | N. º de ejecuciones |
| --- | --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Standard_A1_v2 | Intel(R) Xeon(R) CPU E5-2660 0 a 2,20 GHz | 1 | 1 | 1.9 | 6483 | 120 | 1,85 % | 273 |
| Standard_A1_v2 | Intel(R) Xeon(R) CPU E5-2673 v3 a 2,40 GHz | 1 | 1 | 1.9 | 6059 | 208 | 3,43 % | 217 |
| Standard_A1_v2 | Intel(R) Xeon(R) CPU E5-2673 v4 a 2,30 GHz | 1 | 1 | 1.9 | 6367 | 453 | 7,12 % | 217 |
| Standard_A2_v2 | Intel(R) Xeon(R) CPU E5-2660 0 a 2,20 GHz | 2 | 1 | 3.9 | 13 161 | 194 | 1,48 % | 266 |
| Standard_A2_v2 | Intel(R) Xeon(R) CPU E5-2673 v3 a 2,40 GHz | 2 | 1 | 3.9 | 12 067 | 401 | 3,32 % | 203 |
| Standard_A2_v2 | Intel(R) Xeon(R) CPU E5-2673 v4 a 2,30 GHz | 2 | 1 | 3.9 | 12 527 | 797 | 6,37 % | 238 |
| Standard_A2m_v2 | Intel(R) Xeon(R) CPU E5-2660 0 a 2,20 GHz | 2 | 1 | 15,7 | 13 167 | 179 | 1,36 % | 273 |
| Standard_A2m_v2 | Intel(R) Xeon(R) CPU E5-2673 v3 a 2,40 GHz | 2 | 1 | 15,7 | 12 133 | 336 | 2,77 % | 210 |
| Standard_A2m_v2 | Intel(R) Xeon(R) CPU E5-2673 v4 a 2,30 GHz | 2 | 1 | 15,7 | 12 401 | 656 | 5,29 % | 224 |
| Standard_A4_v2 | Intel(R) Xeon(R) CPU E5-2660 0 a 2,20 GHz | 4 | 1 | 7.8 | 26 307 | 231 | 0,88 % | 231 |
| Standard_A4_v2 | Intel(R) Xeon(R) CPU E5-2673 v3 a 2,40 GHz | 4 | 1 | 7.8 | 24 552 | 720 | 2,93 % | 224 |
| Standard_A4_v2 | Intel(R) Xeon(R) CPU E5-2673 v4 a 2,30 GHz | 4 | 1 | 7.8 | 24 963 | 1625 | 6,51 % | 252 |
| Standard_A4m_v2 | Intel(R) Xeon(R) CPU E5-2660 0 a 2,20 GHz | 4 | 1 | 31,4 | 26 238 | 292 | 1,11 % | 259 |
| Standard_A4m_v2 | Intel(R) Xeon(R) CPU E5-2673 v3 a 2,40 GHz | 4 | 1 | 31,4 | 24 250 | 491 | 2,02 % | 189 |
| Standard_A4m_v2 | Intel(R) Xeon(R) CPU E5-2673 v4 a 2,30 GHz | 4 | 1 | 31,4 | 24 725 | 1553 | 6,28 % | 259 |
| Standard_A8_v2 | Intel(R) Xeon(R) CPU E5-2660 0 a 2,20 GHz | 8 | 1 | 15,7 | 53 237 | 687 | 1,29 % | 266 |
| Standard_A8_v2 | Intel(R) Xeon(R) CPU E5-2673 v3 a 2,40 GHz | 8 | 1 | 15,7 | 49 655 | 585 | 1,18 % | 147 |
| Standard_A8_v2 | Intel(R) Xeon(R) CPU E5-2673 v4 a 2,30 GHz | 8 | 1 | 15,7 | 49 005 | 2162 | 4,41 % | 294 |
| Standard_A8m_v2 | Intel(R) Xeon(R) CPU E5-2660 0 a 2,20 GHz | 8 | 2 | 62,9 | 52 627 | 902 | 1,71 % | 266 |
| Standard_A8m_v2 | Intel(R) Xeon(R) CPU E5-2673 v3 a 2,40 GHz | 8 | 1 | 62,9 | 49 838 | 633 | 1,27 % | 182 |
| Standard_A8m_v2 | Intel(R) Xeon(R) CPU E5-2673 v4 a 2,30 GHz | 8 | 1 | 62,9 | 49 123 | 2483 | 5,05 % | 259 |

> [!NOTE]
> Las máquinas virtuales de la serie Av2 se pueden implementar en diversos procesadores y tipos de hardware (como se ha visto anteriormente). Las máquinas virtuales de la serie Av2 tienen las configuraciones de memoria y rendimiento de CPU adecuadas para cargas de trabajo de nivel de entrada como desarrollo y pruebas. El tamaño se limita para ofrecer un rendimiento del procesador relativamente coherente para la instancia en ejecución, independientemente del hardware en que esté implementado; sin embargo, el software que aprovecha optimizaciones específicas del procesador nuevo puede ver una variación más significativa en función de los tipos de procesador.

## <a name="b---burstable"></a>B: ampliables
(15/3/2019 12:27:08 AM pbi 3897709) (actualizado 14/6/2019 7:09:29 AM pbi 4777081)

| Tamaño de VM | CPU | vCPU | Nodos NUMA | Memoria (GiB) | Puntuación media | Desarrollo estándar | % desarrollo estándar | N. º de ejecuciones |
| --- | --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Standard_B1ms | Intel(R) Xeon(R) CPU E5-2673 v3 a 2,40 GHz | 1 | 1 | 1.9 | 13 593 | 307 | 2,26 % | 28 |
| Standard_B1ms | Intel(R) Xeon(R) CPU E5-2673 v4 a 2,30 GHz | 1 | 1 | 1.9 | 14 069 | 495 | 3,52 % | 672 |
| Standard_B1s | Intel(R) Xeon(R) CPU E5-2673 v3 a 2,40 GHz | 1 | 1 | 0.9 | 13 736 | 211 | 1,54 % | 28 |
| Standard_B1s | Intel(R) Xeon(R) CPU E5-2673 v4 a 2,30 GHz | 1 | 1 | 0.9 | 13 965 | 457 | 3,27 % | 672 |
| Standard_B2ms | Intel(R) Xeon(R) CPU E5-2673 v3 a 2,40 GHz | 2 | 1 | 7.8 | 27 361 | 1110 | 4,06 % | 28 |
| Standard_B2ms | Intel(R) Xeon(R) CPU E5-2673 v4 a 2,30 GHz | 2 | 1 | 7.8 | 27 432 | 771 | 2,81 % | 672 |
| Standard_B2s | Intel(R) Xeon(R) CPU E5-2673 v3 a 2,40 GHz | 2 | 1 | 3.9 | 27 488 | 822 | 2,99 % | 28 |
| Standard_B2s | Intel(R) Xeon(R) CPU E5-2673 v4 a 2,30 GHz | 2 | 1 | 3.9 | 27 548 | 864 | 3,14 % | 672 |
| Standard_B4ms | Intel(R) Xeon(R) CPU E5-2673 v3 a 2,40 GHz | 4 | 1 | 15,7 | 54 951 | 1868 | 3,40 % | 28 |
| Standard_B4ms | Intel(R) Xeon(R) CPU E5-2673 v4 a 2,30 GHz | 4 | 1 | 15,7 | 54 051 | 1260 | 2,33 % | 672 |
| Standard_B8ms | Intel(R) Xeon(R) CPU E5-2673 v3 a 2,40 GHz | 8 | 1 | 31,4 | 111 929 | 1562 | 1,40 % | 35 |
| Standard_B8ms | Intel(R) Xeon(R) CPU E5-2673 v4 a 2,30 GHz | 8 | 1 | 31,4 | 109 537 | 1354 | 1,24 % | 665 |
| Standard_B12ms | Intel(R) Xeon(R) CPU E5-2673 v3 a 2,40 GHz | 12 | 1 | 47,1 | 170 777 | 3 421 | 2,00 % | 70 |
| Standard_B12ms | Intel(R) Xeon(R) CPU E5-2673 v4 a 2,30 GHz | 12 | 1 | 47,1 | 166 676 | 1 368 | 0,82 % | 70 |
| Standard_B16ms | Intel(R) Xeon(R) CPU E5-2673 v3 a 2,40 GHz | 16 | 1 | 62,9 | 208 373 | 30 383 | 14,58 % | 63 |
| Standard_B16ms | Intel(R) Xeon(R) CPU E5-2673 v4 a 2,30 GHz | 16 | 1 | 62,9 | 223 203 | 1 232 | 0,55 % | 70 |
| Standard_B20ms | Intel(R) Xeon(R) CPU E5-2673 v3 a 2,40 GHz | 20 | 1 | 78,6 | 269 561 | 25 095 | 9,31 % | 77 |
| Standard_B20ms | Intel(R) Xeon(R) CPU E5-2673 v4 a 2,30 GHz | 20 | 1 | 78,6 | 274 007 | 3 669 | 1,34 % | 70 |


> [!NOTE]
> Las máquinas virtuales de la serie B son para cargas de trabajo con requisitos de rendimiento que pueden aumentar. Las instancias de máquina virtual acumulan créditos si se usan menos que su base de referencia. Cuando la máquina virtual ha acumulado crédito, puede superar la base de referencia y llegar a usar el 100 % para cumplir los requisitos de ráfaga corta de la CPU. El tiempo de ráfaga depende de los créditos disponibles, que es una función del tamaño y el tiempo de la máquina virtual.  
>
> CoreMark es una prueba corta que habitualmente finaliza dentro de los créditos de ráfagas disponibles.  Por consiguiente, los números anteriores habitualmente representan el rendimiento de las ráfagas de la máquina virtual, lo que refleja lo que normalmente verá el rendimiento corto, con ráfagas de las cargas de trabajo (típico de la serie B).

## <a name="dsv3---general-compute--premium-storage"></a>DSv3: proceso general y almacenamiento Premium
(12/03/2019 6:52:03 PM pbi 3897709)

| Tamaño de VM | CPU | vCPU | Nodos NUMA | Memoria (GiB) | Puntuación media | Desarrollo estándar | % desarrollo estándar | N. º de ejecuciones |
| --- | --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Standard_D2s_v3 | Intel(R) Xeon(R) CPU E5-2673 v3 a 2,40 GHz | 2 | 1 | 7.8 | 20 153 | 838 | 4,16 % | 147 |
| Standard_D2s_v3 | Intel(R) Xeon(R) CPU E5-2673 v4 a 2,30 GHz | 2 | 1 | 7.8 | 20 903 | 1324 | 6,33 % | 553 |
| Standard_D4s_v3 | Intel(R) Xeon(R) CPU E5-2673 v3 a 2,40 GHz | 4 | 1 | 15,7 | 39 502 | 1257 | 3,18 % | 189 |
| Standard_D4s_v3 | Intel(R) Xeon(R) CPU E5-2673 v4 a 2,30 GHz | 4 | 1 | 15,7 | 40 547 | 1935 | 4,77 % | 511 |
| Standard_D8s_v3 | Intel(R) Xeon(R) CPU E5-2673 v3 a 2,40 GHz | 8 | 1 | 31,4 | 80 191 | 1054 | 1,31 % | 168 |
| Standard_D8s_v3 | Intel(R) Xeon(R) CPU E5-2673 v4 a 2,30 GHz | 8 | 1 | 31,4 | 79 884 | 3073 | 3,85 % | 532 |
| Standard_D16s_v3 | Intel(R) Xeon(R) CPU E5-2673 v3 a 2,40 GHz | 16 | 1 | 62,9 | 160 319 | 1213 | 0,76 % | 105 |
| Standard_D16s_v3 | Intel(R) Xeon(R) CPU E5-2673 v4 a 2,30 GHz | 16 | 1 | 62,9 | 156 325 | 2176 | 1,39 % | 588 |
| Standard_D32s_v3 | Intel(R) Xeon(R) CPU E5-2673 v3 a 2,40 GHz | 32 | 2 | 125,9 | 315 457 | 2647 | 0,84 % | 105 |
| Standard_D32s_v3 | Intel(R) Xeon(R) CPU E5-2673 v4 a 2,30 GHz | 32 | 1 | 125,9 | 312 058 | 1661 | 0,53 % | 595 |
| Standard_D64s_v3 | Intel(R) Xeon(R) CPU E5-2673 v4 a 2,30 GHz | 64 | 2 | 251,9 | 627 378 | 4447 | 0,71 % | 700 |

## <a name="dv3---general-compute"></a>Dv3: proceso general
(12/03/2019 6:54:27 PM pbi 3897709)

| Tamaño de VM | CPU | vCPU | Nodos NUMA | Memoria (GiB) | Puntuación media | Desarrollo estándar | % desarrollo estándar | N. º de ejecuciones |
| --- | --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Standard_D2_v3 | Intel(R) Xeon(R) CPU E5-2673 v3 a 2,40 GHz | 2 | 1 | 7.8 | 20 359 | 799 | 3,93 % | 154 |
| Standard_D2_v3 | Intel(R) Xeon(R) CPU E5-2673 v4 a 2,30 GHz | 2 | 1 | 7.8 | 20 737 | 1422 | 6,86 % | 546 |
| Standard_D4_v3 | Intel(R) Xeon(R) CPU E5-2673 v3 a 2,40 GHz | 4 | 1 | 15,7 | 40 095 | 1501 | 3,74 % | 147 |
| Standard_D4_v3 | Intel(R) Xeon(R) CPU E5-2673 v4 a 2,30 GHz | 4 | 1 | 15,7 | 41 147 | 2706 | 6,58 % | 546 |
| Standard_D8_v3 | Intel(R) Xeon(R) CPU E5-2673 v3 a 2,40 GHz | 8 | 1 | 31,4 | 80 383 | 1486 | 1,85 % | 133 |
| Standard_D8_v3 | Intel(R) Xeon(R) CPU E5-2673 v4 a 2,30 GHz | 8 | 1 | 31,4 | 80 511 | 3916 | 4,86 % | 560 |
| Standard_D16_v3 | Intel(R) Xeon(R) CPU E5-2673 v3 a 2,40 GHz | 16 | 1 | 62,9 | 160 932 | 2200 | 1,37 % | 140 |
| Standard_D16_v3 | Intel(R) Xeon(R) CPU E5-2673 v4 a 2,30 GHz | 16 | 1 | 62,9 | 158 679 | 4550 | 2,87 % | 560 |
| Standard_D32_v3 | Intel(R) Xeon(R) CPU E5-2673 v3 a 2,40 GHz | 32 | 2 | 125,9 | 314 208 | 4250 | 1,35 % | 189 |
| Standard_D32_v3 | Intel(R) Xeon(R) CPU E5-2673 v4 a 2,30 GHz | 32 | 1 | 125,9 | 312 472 | 3173 | 1,02 % | 511 |
| Standard_D64_v3 | Intel(R) Xeon(R) CPU E5-2673 v4 a 2,30 GHz | 64 | 2 | 251,9 | 627 470 | 9651 | 1,54 % | 700 |

## <a name="dsv2---storage-optimized"></a>DSv2: optimizada para almacenamiento
(15/03/2019 12:53:13 AM pbi 3897709)

| Tamaño de VM | CPU | vCPU | Nodos NUMA | Memoria (GiB) | Puntuación media | Desarrollo estándar | % desarrollo estándar | N. º de ejecuciones |
| --- | --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Standard_DS1_v2 | Intel(R) Xeon(R) CPU E5-2673 v3 a 2,40 GHz | 1 | 1 | 3.4 | 14 642 | 600 | 4,10 % | 259 |
| Standard_DS1_v2 | Intel(R) Xeon(R) CPU E5-2673 v4 a 2,30 GHz | 1 | 1 | 3.4 | 14 808 | 904 | 6,10 % | 434 |
| Standard_DS2_v2 | Intel(R) Xeon(R) CPU E5-2673 v3 a 2,40 GHz | 2 | 1 | 6,8 | 28 654 | 877 | 3,06 % | 301 |
| Standard_DS2_v2 | Intel(R) Xeon(R) CPU E5-2673 v4 a 2,30 GHz | 2 | 1 | 6,8 | 29 089 | 1421 | 4,89 % | 406 |
| Standard_DS3_v2 | Intel(R) Xeon(R) CPU E5-2673 v3 a 2,40 GHz | 4 | 1 | 13,7 | 57 255 | 1633 | 2,85 % | 238 |
| Standard_DS3_v2 | Intel(R) Xeon(R) CPU E5-2673 v4 a 2,30 GHz | 4 | 1 | 13,7 | 57 255 | 2265 | 3,96 % | 462 |
| Standard_DS4_v2 | Intel(R) Xeon(R) CPU E5-2673 v3 a 2,40 GHz | 8 | 1 | 27,5 | 116 681 | 1097 | 0,94 % | 231 |
| Standard_DS4_v2 | Intel(R) Xeon(R) CPU E5-2673 v4 a 2,30 GHz | 8 | 1 | 27,5 | 112 512 | 1261 | 1,12 % | 462 |
| Standard_DS5_v2 | Intel(R) Xeon(R) CPU E5-2673 v3 a 2,40 GHz | 16 | 1 | 55,0 | 225 661 | 2370 | 1,05 % | 189 |
| Standard_DS5_v2 | Intel(R) Xeon(R) CPU E5-2673 v3 a 2,40 GHz | 16 | 2 | 55,0 | 229 145 | 2878 | 1,26 % | 21 |
| Standard_DS5_v2 | Intel(R) Xeon(R) CPU E5-2673 v4 a 2,30 GHz | 16 | 1 | 55,0 | 226 818 | 1797 | 0,79 % | 497 |
| Standard_DS11_v2 | Intel(R) Xeon(R) CPU E5-2673 v3 a 2,40 GHz | 2 | 1 | 13,7 | 28 571 | 920 | 3,22 % | 238 |
| Standard_DS11_v2 | Intel(R) Xeon(R) CPU E5-2673 v4 a 2,30 GHz | 2 | 1 | 13,7 | 29 049 | 1614 | 5,56 % | 469 |
| Standard_DS11-1_v2 | Intel(R) Xeon(R) CPU E5-2673 v3 a 2,40 GHz | 1 | 1 | 13,7 | 14 594 | 617 | 4,23 % | 287 |
| Standard_DS11-1_v2 | Intel(R) Xeon(R) CPU E5-2673 v4 a 2,30 GHz | 1 | 1 | 13,7 | 14 951 | 852 | 5,70 % | 413 |
| Standard_DS12_v2 | Intel(R) Xeon(R) CPU E5-2673 v3 a 2,40 GHz | 4 | 1 | 27,5 | 57 503 | 1398 | 2,43 % | 217 |
| Standard_DS12_v2 | Intel(R) Xeon(R) CPU E5-2673 v4 a 2,30 GHz | 4 | 1 | 27,5 | 57 082 | 2372 | 4,16 % | 483 |
| Standard_DS12-1_v2 | Intel(R) Xeon(R) CPU E5-2673 v3 a 2,40 GHz | 1 | 1 | 27,5 | 14 698 | 564 | 3,84 % | 238 |
| Standard_DS12-1_v2 | Intel(R) Xeon(R) CPU E5-2673 v4 a 2,30 GHz | 1 | 1 | 27,5 | 15 127 | 941 | 6,22 % | 462 |
| Standard_DS12-2_v2 | Intel(R) Xeon(R) CPU E5-2673 v3 a 2,40 GHz | 2 | 1 | 27,5 | 28 711 | 981 | 3,42 % | 259 |
| Standard_DS12-2_v2 | Intel(R) Xeon(R) CPU E5-2673 v4 a 2,30 GHz | 2 | 1 | 27,5 | 29 305 | 1241 | 4,24 % | 441 |
| Standard_DS13_v2 | Intel(R) Xeon(R) CPU E5-2673 v3 a 2,40 GHz | 8 | 1 | 55,0 | 116 875 | 1286 | 1,10 % | 203 |
| Standard_DS13_v2 | Intel(R) Xeon(R) CPU E5-2673 v4 a 2,30 GHz | 8 | 1 | 55,0 | 112 318 | 1356 | 1,21 % | 504 |
| Standard_DS13-2_v2 | Intel(R) Xeon(R) CPU E5-2673 v3 a 2,40 GHz | 2 | 1 | 55,0 | 29 105 | 1154 | 3,97 % | 224 |
| Standard_DS13-2_v2 | Intel(R) Xeon(R) CPU E5-2673 v4 a 2,30 GHz | 2 | 1 | 55,0 | 29 936 | 1720 | 5,75 % | 483 |
| Standard_DS13-4_v2 | Intel(R) Xeon(R) CPU E5-2673 v3 a 2,40 GHz | 4 | 1 | 55,0 | 56 992 | 1814 | 3,18 % | 280 |
| Standard_DS13-4_v2 | Intel(R) Xeon(R) CPU E5-2673 v4 a 2,30 GHz | 4 | 1 | 55,0 | 57 781 | 2122 | 3,67 % | 427 |
| Standard_DS14_v2 | Intel(R) Xeon(R) CPU E5-2673 v3 a 2,40 GHz | 16 | 2 | 110,2 | 224 149 | 3450 | 1,54 % | 196 |
| Standard_DS14_v2 | Intel(R) Xeon(R) CPU E5-2673 v4 a 2,30 GHz | 16 | 1 | 110,2 | 227 108 | 1267 | 0,56 % | 504 |
| Standard_DS14-4_v2 | Intel(R) Xeon(R) CPU E5-2673 v3 a 2,40 GHz | 4 | 2 | 110,2 | 56 211 | 2154 | 3,83 % | 189 |
| Standard_DS14-4_v2 | Intel(R) Xeon(R) CPU E5-2673 v4 a 2,30 GHz | 4 | 1 | 110,2 | 59 651 | 2560 | 4,29 % | 518 |
| Standard_DS14-8_v2 | Intel(R) Xeon(R) CPU E5-2673 v3 a 2,40 GHz | 8 | 2 | 110,2 | 112 280 | 4430 | 3,95 % | 196 |
| Standard_DS14-8_v2 | Intel(R) Xeon(R) CPU E5-2673 v4 a 2,30 GHz | 8 | 1 | 110,2 | 113 375 | 1442 | 1,27 % | 511 |
| Standard_DS15_v2 | Intel(R) Xeon(R) CPU E5-2673 v3 a 2,40 GHz | 20 | 2 | 137,7 | 279 359 | 4032 | 1,44 % | 665 |

## <a name="dv2---general-compute"></a>Dv2: proceso general
(12/03/2019 6:53:48 PM pbi 3897709)

| Tamaño de VM | CPU | vCPU | Nodos NUMA | Memoria (GiB) | Puntuación media | Desarrollo estándar | % desarrollo estándar | N. º de ejecuciones |
| --- | --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Standard_D1_v2 | Intel(R) Xeon(R) CPU E5-2673 v3 a 2,40 GHz | 1 | 1 | 3.4 | 14 730 | 663 | 4,50 % | 385 |
| Standard_D1_v2 | Intel(R) Xeon(R) CPU E5-2673 v4 a 2,30 GHz | 1 | 1 | 3.4 | 15 057 | 1319 | 8,76 % | 322 |
| Standard_D2_v2 | Intel(R) Xeon(R) CPU E5-2673 v3 a 2,40 GHz | 2 | 1 | 6,8 | 29 395 | 1073 | 3,65 % | 329 |
| Standard_D2_v2 | Intel(R) Xeon(R) CPU E5-2673 v4 a 2,30 GHz | 2 | 1 | 6,8 | 29 564 | 2145 | 7,26 % | 378 |
| Standard_D3_v2 | Intel(R) Xeon(R) CPU E5-2673 v3 a 2,40 GHz | 4 | 1 | 13,7 | 58 150 | 1340 | 2,30 % | 343 |
| Standard_D3_v2 | Intel(R) Xeon(R) CPU E5-2673 v4 a 2,30 GHz | 4 | 1 | 13,7 | 57 820 | 2944 | 5,09 % | 364 |
| Standard_D4_v2 | Intel(R) Xeon(R) CPU E5-2673 v3 a 2,40 GHz | 8 | 1 | 27,5 | 117 448 | 1612 | 1,37 % | 308 |
| Standard_D4_v2 | Intel(R) Xeon(R) CPU E5-2673 v4 a 2,30 GHz | 8 | 1 | 27,5 | 114 082 | 3369 | 2,95 % | 399 |
| Standard_D5_v2 | Intel(R) Xeon(R) CPU E5-2673 v3 a 2,40 GHz | 16 | 1 | 55,0 | 226 370 | 4722 | 2,09 % | 147 |
| Standard_D5_v2 | Intel(R) Xeon(R) CPU E5-2673 v3 a 2,40 GHz | 16 | 2 | 55,0 | 225 035 | 5026 | 2,23 % | 119 |
| Standard_D5_v2 | Intel(R) Xeon(R) CPU E5-2673 v4 a 2,30 GHz | 16 | 1 | 55,0 | 227 883 | 3259 | 1,43 % | 441 |
| Standard_D11_v2 | Intel(R) Xeon(R) CPU E5-2673 v3 a 2,40 GHz | 2 | 1 | 13,7 | 29 260 | 1012 | 3,46 % | 308 |
| Standard_D11_v2 | Intel(R) Xeon(R) CPU E5-2673 v4 a 2,30 GHz | 2 | 1 | 13,7 | 29 306 | 1763 | 6,02 % | 399 |
| Standard_D12_v2 | Intel(R) Xeon(R) CPU E5-2673 v3 a 2,40 GHz | 4 | 1 | 27,5 | 58 322 | 1391 | 2,39 % | 329 |
| Standard_D12_v2 | Intel(R) Xeon(R) CPU E5-2673 v4 a 2,30 GHz | 4 | 1 | 27,5 | 57 999 | 3533 | 6,09 % | 371 |
| Standard_D13_v2 | Intel(R) Xeon(R) CPU E5-2673 v3 a 2,40 GHz | 8 | 1 | 55,0 | 117 218 | 1514 | 1,29 % | 329 |
| Standard_D13_v2 | Intel(R) Xeon(R) CPU E5-2673 v4 a 2,30 GHz | 8 | 1 | 55,0 | 114 344 | 3307 | 2,89 % | 378 |
| Standard_D14_v2 | Intel(R) Xeon(R) CPU E5-2673 v3 a 2,40 GHz | 16 | 2 | 110,2 | 224 348 | 5477 | 2,44 % | 280 |
| Standard_D14_v2 | Intel(R) Xeon(R) CPU E5-2673 v4 a 2,30 GHz | 16 | 1 | 110,2 | 228 221 | 2733 | 1,20 % | 427 |
| Standard_D15_v2 | Intel(R) Xeon(R) CPU E5-2673 v3 a 2,40 GHz | 20 | 2 | 137,7 | 281 494 | 7976 | 2,83 % | 672 |

## <a name="esv3---memory-optimized--premium-storage"></a>Esv3: optimizada para memoria y almacenamiento Premium
(12/03/2019 7:17:33 PM pbi 3897709)

| Tamaño de VM | CPU | vCPU | Nodos NUMA | Memoria (GiB) | Puntuación media | Desarrollo estándar | % desarrollo estándar | N. º de ejecuciones |
| --- | --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Standard_E2s_v3 | Intel(R) Xeon(R) CPU E5-2673 v4 a 2,30 GHz | 2 | 1 | 15,7 | 20 957 | 1,200 | 5,73 % | 672 |
| Standard_E4s_v3 | Intel(R) Xeon(R) CPU E5-2673 v4 a 2,30 GHz | 4 | 1 | 31,4 | 40 420 | 1993 | 4,93 % | 672 |
| Standard_E4-2s_v3 | Intel(R) Xeon(R) CPU E5-2673 v4 a 2,30 GHz | 2 | 1 | 31,4 | 20 774 | 1133 | 5,45 % | 672 |
| Standard_E8s_v3 | Intel(R) Xeon(R) CPU E5-2673 v4 a 2,30 GHz | 8 | 1 | 62,9 | 80 153 | 3308 | 4,13 % | 665 |
| Standard_E8-2s_v3 | Intel(R) Xeon(R) CPU E5-2673 v4 a 2,30 GHz | 2 | 1 | 62,9 | 21 178 | 1334 | 6,30 % | 665 |
| Standard_E8-4s_v3 | Intel(R) Xeon(R) CPU E5-2673 v4 a 2,30 GHz | 4 | 1 | 62,9 | 40 614 | 2216 | 5,46 % | 672 |
| Standard_E16s_v3 | Intel(R) Xeon(R) CPU E5-2673 v4 a 2,30 GHz | 16 | 1 | 125,9 | 156 137 | 2160 | 1,38 % | 672 |
| Standard_E16-4s_v3 | Intel(R) Xeon(R) CPU E5-2673 v4 a 2,30 GHz | 4 | 1 | 125,9 | 41 950 | 2309 | 5,50 % | 637 |
| Standard_E16-8s_v3 | Intel(R) Xeon(R) CPU E5-2673 v4 a 2,30 GHz | 8 | 1 | 125,9 | 81 196 | 3179 | 3,91 % | 658 |
| Standard_E20s_v3 | Intel(R) Xeon(R) CPU E5-2673 v4 a 2,30 GHz | 20 | 1 | 157,4 | 196 619 | 1325 | 0,67 % | 672 |
| Standard_E32s_v3 | Intel(R) Xeon(R) CPU E5-2673 v4 a 2,30 GHz | 32 | 2 | 251,9 | 304 707 | 5719 | 1,88 % | 672 |
| Standard_E32-8s_v3 | Intel(R) Xeon(R) CPU E5-2673 v4 a 2,30 GHz | 8 | 2 | 251,9 | 83 576 | 3693 | 4,42 % | 672 |
| Standard_E32-16s_v3 | Intel(R) Xeon(R) CPU E5-2673 v4 a 2,30 GHz | 16 | 2 | 251,9 | 158 023 | 4317 | 2,73 % | 672 |
| Standard_E64s_v3 | Intel(R) Xeon(R) CPU E5-2673 v4 a 2,30 GHz | 64 | 2 | 425,2 | 628 540 | 3982 | 0,63 % | 49 |
| Standard_E64-16s_v3 | Intel(R) Xeon(R) CPU E5-2673 v4 a 2,30 GHz | 16 | 2 | 425,2 | 169 611 | 3265 | 1,92 % | 42 |
| Standard_E64-32s_v3 | Intel(R) Xeon(R) CPU E5-2673 v4 a 2,30 GHz | 32 | 2 | 425,2 | 307 584 | 3569 | 1,16 % | 56 |

## <a name="eisv3---memory-opt--premium-storage-isolated"></a>Eisv3: optimizada para memoria y Premium Storage (aislado)
(11/04/2019 10:07:29 PM pbi 3897709)

| Tamaño de VM | CPU | vCPU | Nodos NUMA | Memoria (GiB) | Puntuación media | Desarrollo estándar | % desarrollo estándar | N. º de ejecuciones |
| --- | --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Standard_E64is_v3 | Intel(R) Xeon(R) CPU E5-2673 v4 a 2,30 GHz | 64 | 2 | 425,2 | 627 745 | 4062 | 0,65 % | 196 |

## <a name="ev3---memory-optimized"></a>Ev3: optimizada para memoria
(12/03/2019 6:52:13 PM pbi 3897709)

| Tamaño de VM | CPU | vCPU | Nodos NUMA | Memoria (GiB) | Puntuación media | Desarrollo estándar | % desarrollo estándar | N. º de ejecuciones |
| --- | --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Standard_E2_v3 | Intel(R) Xeon(R) CPU E5-2673 v4 a 2,30 GHz | 2 | 1 | 15,7 | 21 171 | 1772 | 8,37 % | 693 |
| Standard_E4_v3 | Intel(R) Xeon(R) CPU E5-2673 v4 a 2,30 GHz | 4 | 1 | 31,4 | 41 181 | 3148 | 7,64 % | 700 |
| Standard_E8_v3 | Intel(R) Xeon(R) CPU E5-2673 v4 a 2,30 GHz | 8 | 1 | 62,9 | 81 211 | 5055 | 6,22 % | 700 |
| Standard_E16_v3 | Intel(R) Xeon(R) CPU E5-2673 v4 a 2,30 GHz | 16 | 1 | 125,9 | 158 152 | 4033 | 2,55 % | 700 |
| Standard_E20_v3 | Intel(R) Xeon(R) CPU E5-2673 v4 a 2,30 GHz | 20 | 1 | 157,4 | 197 739 | 2731 | 1,38 % | 693 |
| Standard_E32_v3 | Intel(R) Xeon(R) CPU E5-2673 v4 a 2,30 GHz | 32 | 2 | 251,9 | 307 286 | 8353 | 2,72 % | 700 |
| Standard_E64_v3 | Intel(R) Xeon(R) CPU E5-2673 v4 a 2,30 GHz | 64 | 2 | 425,2 | 628 451 | 9235 | 1,47 % | 707 |

## <a name="eiv3---memory-optimized-isolated"></a>Eiv3: optimizada para memoria (aislado)
(12/03/2019 6:57:51 PM pbi 3897709)

| Tamaño de VM | CPU | vCPU | Nodos NUMA | Memoria (GiB) | Puntuación media | Desarrollo estándar | % desarrollo estándar | N. º de ejecuciones |
| --- | --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Standard_E64i_v3 | Intel(R) Xeon(R) CPU E5-2673 v4 a 2,30 GHz | 64 | 2 | 425,2 | 625 855 | 4881 | 0,78 % | 7 |
| Standard_E64i_v3 | Intel(R) Xeon(R) CPU E5-2673 v4 a 2,30 GHz | 64 | 2 | 425,2 | 629 151 | 9756 | 1,55 % | 217 |

## <a name="fsv2---compute--storage-optimized"></a>Fsv2: proceso y optimizada para almacenamiento
(12/03/2019 6:51:35 PM pbi 3897709)

| Tamaño de VM | CPU | vCPU | Nodos NUMA | Memoria (GiB) | Puntuación media | Desarrollo estándar | % desarrollo estándar | N. º de ejecuciones |
| --- | --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Standard_F2s_v2 | Intel(R) Xeon(R) Platinum 8168 CPU a 2,70 GHz | 2 | 1 | 3.9 | 28 219 | 1843 | 6,53 % | 700 |
| Standard_F4s_v2 | Intel(R) Xeon(R) Platinum 8168 CPU a 2,70 GHz | 4 | 1 | 7.8 | 53 911 | 1002 | 1,86 % | 707 |
| Standard_F8s_v2 | Intel(R) Xeon(R) Platinum 8168 CPU a 2,70 GHz | 8 | 1 | 15,7 | 106 467 | 1101 | 1,03 % | 707 |
| Standard_F16s_v2 | Intel(R) Xeon(R) Platinum 8168 CPU a 2,70 GHz | 16 | 1 | 31,4 | 211 311 | 1724 | 0,82 % | 707 |
| Standard_F32s_v2 | Intel(R) Xeon(R) Platinum 8168 CPU a 2,70 GHz | 32 | 1 | 62,9 | 423 175 | 4346 | 1,03 % | 707 |
| Standard_F64s_v2 | Intel(R) Xeon(R) Platinum 8168 CPU a 2,70 GHz | 64 | 2 | 125,9 | 829 537 | 21 574 | 2,60 % | 707 |
| Standard_F72s_v2 | Intel(R) Xeon(R) Platinum 8168 CPU a 2,70 GHz | 72 | 2 | 141,7 | 933 800 | 26 783 | 2,87 % | 707 |

## <a name="fs---compute-and-storage-optimized"></a>Fs: proceso y optimizada para almacenamiento
(15/03/2019 12:12:51 AM pbi 3897709)

| Tamaño de VM | CPU | vCPU | Nodos NUMA | Memoria (GiB) | Puntuación media | Desarrollo estándar | % desarrollo estándar | N. º de ejecuciones |
| --- | --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Standard_F1s | Intel(R) Xeon(R) CPU E5-2673 v3 a 2,40 GHz | 1 | 1 | 1.9 | 14 552 | 504 | 3,46 % | 350 |
| Standard_F1s | Intel(R) Xeon(R) CPU E5-2673 v4 a 2,30 GHz | 1 | 1 | 1.9 | 14 784 | 858 | 5,80 % | 357 |
| Standard_F2s | Intel(R) Xeon(R) CPU E5-2673 v3 a 2,40 GHz | 2 | 1 | 3.9 | 28 664 | 895 | 3,12 % | 245 |
| Standard_F2s | Intel(R) Xeon(R) CPU E5-2673 v4 a 2,30 GHz | 2 | 1 | 3.9 | 29 188 | 1228 | 4,21 % | 455 |
| Standard_F4s | Intel(R) Xeon(R) CPU E5-2673 v3 a 2,40 GHz | 4 | 1 | 7.8 | 57 192 | 1700 | 2,97 % | 259 |
| Standard_F4s | Intel(R) Xeon(R) CPU E5-2673 v4 a 2,30 GHz | 4 | 1 | 7.8 | 57 412 | 2215 | 3,86 % | 448 |
| Standard_F8s | Intel(R) Xeon(R) CPU E5-2673 v3 a 2,40 GHz | 8 | 1 | 15,7 | 117 008 | 1139 | 0,97 % | 259 |
| Standard_F8s | Intel(R) Xeon(R) CPU E5-2673 v4 a 2,30 GHz | 8 | 1 | 15,7 | 112 610 | 1595 | 1,42 % | 441 |
| Standard_F16s | Intel(R) Xeon(R) CPU E5-2673 v3 a 2,40 GHz | 16 | 1 | 31,4 | 225 444 | 2328 | 1,03 % | 210 |
| Standard_F16s | Intel(R) Xeon(R) CPU E5-2673 v3 a 2,40 GHz | 16 | 2 | 31,4 | 228 919 | 3380 | 1,48 % | 28 |
| Standard_F16s | Intel(R) Xeon(R) CPU E5-2673 v4 a 2,30 GHz | 16 | 1 | 31,4 | 227 015 | 1543 | 0,68 % | 462 |

## <a name="f---compute-optimized"></a>F: optimizada para proceso
(12/03/2019 6:53:59 PM pbi 3897709)

| Tamaño de VM | CPU | vCPU | Nodos NUMA | Memoria (GiB) | Puntuación media | Desarrollo estándar | % desarrollo estándar | N. º de ejecuciones |
| --- | --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Standard_F1 | Intel(R) Xeon(R) CPU E5-2673 v3 a 2,40 GHz | 1 | 1 | 1.9 | 14 937 | 593 | 3,97 % | 350 |
| Standard_F1 | Intel(R) Xeon(R) CPU E5-2673 v4 a 2,30 GHz | 1 | 1 | 1.9 | 15 460 | 1326 | 8,58 % | 350 |
| Standard_F2 | Intel(R) Xeon(R) CPU E5-2673 v3 a 2,40 GHz | 2 | 1 | 3.9 | 29 324 | 1196 | 4,08 % | 343 |
| Standard_F2 | Intel(R) Xeon(R) CPU E5-2673 v4 a 2,30 GHz | 2 | 1 | 3.9 | 29 299 | 1908 | 6,51 % | 364 |
| Standard_F4 | Intel(R) Xeon(R) CPU E5-2673 v3 a 2,40 GHz | 4 | 1 | 7.8 | 58 314 | 1245 | 2,14 % | 364 |
| Standard_F4 | Intel(R) Xeon(R) CPU E5-2673 v4 a 2,30 GHz | 4 | 1 | 7.8 | 58 280 | 3581 | 6,14 % | 336 |
| Standard_F8 | Intel(R) Xeon(R) CPU E5-2673 v3 a 2,40 GHz | 8 | 1 | 15,7 | 117 516 | 1460 | 1,24 % | 308 |
| Standard_F8 | Intel(R) Xeon(R) CPU E5-2673 v4 a 2,30 GHz | 8 | 1 | 15,7 | 114 361 | 3868 | 3,38 % | 399 |
| Standard_F16 | Intel(R) Xeon(R) CPU E5-2673 v3 a 2,40 GHz | 16 | 1 | 31,4 | 226 487 | 4140 | 1,83 % | 154 |
| Standard_F16 | Intel(R) Xeon(R) CPU E5-2673 v3 a 2,40 GHz | 16 | 2 | 31,4 | 226 683 | 4723 | 2,08 % | 133 |
| Standard_F16 | Intel(R) Xeon(R) CPU E5-2673 v4 a 2,30 GHz | 16 | 1 | 31,4 | 228 592 | 2371 | 1,04 % | 392 |

## <a name="gs---storage-optimized"></a>GS: optimizada para almacenamiento
(12/03/2019 10:22:33 PM pbi 3897709)

| Tamaño de VM | CPU | vCPU | Nodos NUMA | Memoria (GiB) | Puntuación media | Desarrollo estándar | % desarrollo estándar | N. º de ejecuciones |
| --- | --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Standard_GS1 | Intel(R) Xeon(R) CPU E5-2698B v3 a 2,00 GHz | 2 | 1 | 27,5 | 28 835 | 2222 | 7,71 % | 287 |
| Standard_GS2 | Intel(R) Xeon(R) CPU E5-2698B v3 a 2,00 GHz | 4 | 1 | 55,0 | 55 568 | 3139 | 5,65 % | 287 |
| Standard_GS3 | Intel(R) Xeon(R) CPU E5-2698B v3 a 2,00 GHz | 8 | 1 | 110,2 | 106 567 | 2188 | 2,05 % | 287 |
| Standard_GS4 | Intel(R) Xeon(R) CPU E5-2698B v3 a 2,00 GHz | 16 | 1 | 220,4 | 210 586 | 4130 | 1,96 % | 287 |
| Standard_GS4-4 | Intel(R) Xeon(R) CPU E5-2698B v3 a 2,00 GHz | 4 | 1 | 220,4 | 58 598 | 2670 | 4,56 % | 287 |
| Standard_GS4-8 | Intel(R) Xeon(R) CPU E5-2698B v3 a 2,00 GHz | 8 | 1 | 220,4 | 108 234 | 2392 | 2,21 % | 287 |
| Standard_GS5 | Intel(R) Xeon(R) CPU E5-2698B v3 a 2,00 GHz | 32 | 2 | 440,9 | 399 835 | 8694 | 2,17 % | 287 |
| Standard_GS5-8 | Intel(R) Xeon(R) CPU E5-2698B v3 a 2,00 GHz | 8 | 2 | 440,9 | 116 643 | 2354 | 2,02 % | 287 |
| Standard_GS5-16 | Intel(R) Xeon(R) CPU E5-2698B v3 a 2,00 GHz | 16 | 2 | 440,9 | 210 984 | 2995 | 1,42 % | 287 |

## <a name="g---compute-optimized"></a>G: optimizada para proceso
(12/03/2019 10:23:51 PM pbi 3897709)

| Tamaño de VM | CPU | vCPU | Nodos NUMA | Memoria (GiB) | Puntuación media | Desarrollo estándar | % desarrollo estándar | N. º de ejecuciones |
| --- | --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Standard_G1 | Intel(R) Xeon(R) CPU E5-2698B v3 a 2,00 GHz | 2 | 1 | 27,5 | 32 808 | 2679 | 8,17 % | 287 |
| Standard_G2 | Intel(R) Xeon(R) CPU E5-2698B v3 a 2,00 GHz | 4 | 1 | 55,0 | 62 907 | 4465 | 7,10 % | 287 |
| Standard_G3 | Intel(R) Xeon(R) CPU E5-2698B v3 a 2,00 GHz | 8 | 1 | 110,2 | 113 471 | 4346 | 3,83 % | 287 |
| Standard_G4 | Intel(R) Xeon(R) CPU E5-2698B v3 a 2,00 GHz | 16 | 1 | 220,4 | 212 092 | 2857 | 1,35 % | 287 |
| Standard_G5 | Intel(R) Xeon(R) CPU E5-2698B v3 a 2,00 GHz | 32 | 2 | 440,9 | 403 315 | 6947 | 1,72 % | 273 |

## <a name="h---high-performance-compute-hpc"></a>H: Proceso de alto rendimiento (HPC)
(12/03/2019 10:50:51 PM pbi 3897709)

| Tamaño de VM | CPU | vCPU | Nodos NUMA | Memoria (GiB) | Puntuación media | Desarrollo estándar | % desarrollo estándar | N. º de ejecuciones |
| --- | --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Standard_H8 | Intel(R) Xeon(R) CPU E5-2667 v3 a 3,20 GHz | 8 | 1 | 55,0 | 149 859 | 734 | 0,49 % | 175 |
| Standard_H8m | Intel(R) Xeon(R) CPU E5-2667 v3 a 3,20 GHz | 8 | 1 | 110,2 | 149 931 | 657 | 0,44 % | 147 |
| Standard_H16 | Intel(R) Xeon(R) CPU E5-2667 v3 a 3,20 GHz | 16 | 2 | 110,2 | 282 083 | 6738 | 2,39 % | 84 |
| Standard_H16m | Intel(R) Xeon(R) CPU E5-2667 v3 a 3,20 GHz | 16 | 2 | 220,4 | 282 106 | 7013 | 2,49 % | 84 |
| Standard_H16mr | Intel(R) Xeon(R) CPU E5-2667 v3 a 3,20 GHz | 16 | 2 | 220,4 | 282 167 | 6889 | 2,44 % | 84 |
| Standard_H16r | Intel(R) Xeon(R) CPU E5-2667 v3 a 3,20 GHz | 16 | 2 | 110,2 | 280 837 | 6587 | 2,35 % | 84 |

## <a name="lv2---storage-optimized"></a>Lv2: optimizada para almacenamiento
(14/03/2019 5:49:04 PM pbi 3897709)

| Tamaño de VM | CPU | vCPU | Nodos NUMA | Memoria (GiB) | Puntuación media | Desarrollo estándar | % desarrollo estándar | N. º de ejecuciones |
| --- | --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Standard_L8s_v2 | Procesador de 32 núcleos AMD EPYC 7551 | 8 | 1 | 62,9 | 80 528 | 404 | 0,50 % | 119 |
| Standard_L16s_v2 | Procesador de 32 núcleos AMD EPYC 7551 | 16 | 2 | 125,9 | 154 829 | 3708 | 2,40 % | 119 |
| Standard_L32s_v2 | Procesador de 32 núcleos AMD EPYC 7551 | 32 | 4 | 251,9 | 310 811 | 7751 | 2,49 % | 119 |
| Standard_L64s_v2 | Procesador de 32 núcleos AMD EPYC 7551 | 64 | 8 | 503,9 | 595 140 | 14 572 | 2,45 % | 112 |
| Standard_L80s_v2 | Procesador de 32 núcleos AMD EPYC 7551 | 80 | 10 | 629,9 | 773 171 | 19 559 | 2,53 % | 119 |

## <a name="ls---storage-optimized"></a>Ls: optimizada para almacenamiento
(12/03/2019 10:22:29 PM pbi 3897709)

| Tamaño de VM | CPU | vCPU | Nodos NUMA | Memoria (GiB) | Puntuación media | Desarrollo estándar | % desarrollo estándar | N. º de ejecuciones |
| --- | --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Standard_L4s | Intel(R) Xeon(R) CPU E5-2698B v3 a 2,00 GHz | 4 | 1 | 31,4 | 56 488 | 2916 | 5,16 % | 287 |
| Standard_L8s | Intel(R) Xeon(R) CPU E5-2698B v3 a 2,00 GHz | 8 | 1 | 62,9 | 107 017 | 2323 | 2,17 % | 287 |
| Standard_L16s | Intel(R) Xeon(R) CPU E5-2698B v3 a 2,00 GHz | 16 | 1 | 125,9 | 210 865 | 3653 | 1,73 % | 280 |
| Standard_L32s | Intel(R) Xeon(R) CPU E5-2698B v3 a 2,00 GHz | 32 | 2 | 251,9 | 399 963 | 9254 | 2,31 % | 287 |

## <a name="m---memory-optimized"></a>M: optimizada para memoria
(11/04/2019 7:30:39 PM pbi 3897709)

| Tamaño de VM | CPU | vCPU | Nodos NUMA | Memoria (GiB) | Puntuación media | Desarrollo estándar | % desarrollo estándar | N. º de ejecuciones |
| --- | --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Standard_M8-2ms | Intel(R) Xeon(R) CPU E7-8890 v3 a 2,50 GHz | 2 | 1 | 215,2 | 22 605 | 29 | 0,13 % | 42 |
| Standard_M8-4ms | Intel(R) Xeon(R) CPU E7-8890 v3 a 2,50 GHz | 4 | 1 | 215,2 | 44 488 | 183 | 0,41 % | 42 |
| Standard_M16-4ms | Intel(R) Xeon(R) CPU E7-8890 v3 a 2,50 GHz | 4 | 1 | 430,6 | 44 451 | 269 | 0,61 % | 42 |
| Standard_M16-8ms | Intel(R) Xeon(R) CPU E7-8890 v3 a 2,50 GHz | 8 | 1 | 430,6 | 88 238 | 1243 | 1,41 % | 42 |
| Standard_M32-8ms | Intel(R) Xeon(R) CPU E7-8890 v3 a 2,50 GHz | 8 | 1 | 861,2 | 88 521 | 1353 | 1,53 % | 42 |
| Standard_M32-16ms | Intel(R) Xeon(R) CPU E7-8890 v3 a 2,50 GHz | 16 | 1 | 861,2 | 174 674 | 3104 | 1,78 % | 42 |
| Standard_M64 | Intel(R) Xeon(R) CPU E7-8890 v3 a 2,50 GHz | 64 | 2 | 1007,9 | 683 022 | 11 929 | 1,75 % | 42 |
| Standard_M64-16ms | Intel(R) Xeon(R) CPU E7-8890 v3 a 2,50 GHz | 16 | 2 | 1763,9 | 169 386 | 4737 | 2,80 % | 42 |
| Standard_M64-32ms | Intel(R) Xeon(R) CPU E7-8890 v3 a 2,50 GHz | 32 | 2 | 1763,9 | 337 599 | 4738 | 1,40 % | 42 |
| Standard_M64m | Intel(R) Xeon(R) CPU E7-8890 v3 a 2,50 GHz | 64 | 2 | 1763,9 | 677 466 | 14 478 | 2,14 % | 42 |
| Standard_M64ms | Intel(R) Xeon(R) CPU E7-8890 v3 a 2,50 GHz | 64 | 2 | 1763,9 | 675 342 | 16 577 | 2,45 % | 42 |
| Standard_M64s | Intel(R) Xeon(R) CPU E7-8890 v3 a 2,50 GHz | 64 | 2 | 1007,9 | 674 785 | 15 983 | 2,37 % | 42 |
| Standard_M128 | Intel(R) Xeon(R) CPU E7-8890 v3 a 2,50 GHz | 128 | 4 | 2015,9 | 1 334 218 | 21 126 | 1,58 % | 42 |
| Standard_M128-32ms | Intel(R) Xeon(R) CPU E7-8890 v3 a 2,50 GHz | 32 | 4 | 3831,1 | 334 873 | 5005 | 1,49 % | 42 |
| Standard_M128-64ms | Intel(R) Xeon(R) CPU E7-8890 v3 a 2,50 GHz | 64 | 4 | 3831,1 | 667 808 | 18 145 | 2,72 % | 42 |
| Standard_M128m | Intel(R) Xeon(R) CPU E7-8890 v3 a 2,50 GHz | 128 | 4 | 3831,1 | 1 335 873 | 19 642 | 1,47 % | 42 |
| Standard_M128ms | Intel(R) Xeon(R) CPU E7-8890 v3 a 2,50 GHz | 128 | 4 | 3831,1 | 1 329 151 | 24 295 | 1,83 % | 42 |
| Standard_M128s | Intel(R) Xeon(R) CPU E7-8890 v3 a 2,50 GHz | 128 | 4 | 2015,9 | 1 329 923 | 20 117 | 1,51 % | 42 |
| Standard_M16ms | Intel(R) Xeon(R) CPU E7-8890 v3 a 2,50 GHz | 16 | 1 | 430,6 | 174 686 | 2704 | 1,55 % | 35 |
| Standard_M32ls | Intel(R) Xeon(R) CPU E7-8890 v3 a 2,50 GHz | 32 | 1 | 251,9 | 344 069 | 3372 | 0,98 % | 42 |
| Standard_M32ms | Intel(R) Xeon(R) CPU E7-8890 v3 a 2,50 GHz | 32 | 1 | 861,2 | 343 226 | 4884 | 1,42 % | 84 |
| Standard_M32ms | Intel(R) Xeon(R) CPU E7-8890 v3 a 2,50 GHz | 32 | 2 | 861,2 | 336 526 | 4396 | 1,31 % | 7 |
| Standard_M32ts | Intel(R) Xeon(R) CPU E7-8890 v3 a 2,50 GHz | 32 | 1 | 188,9 | 341 112 | 5491 | 1,61 % | 35 |
| Standard_M64ls | Intel(R) Xeon(R) CPU E7-8890 v3 a 2,50 GHz | 64 | 2 | 503,9 | 676 026 | 18 078 | 2,67 % | 42 |
| Standard_M8ms | Intel(R) Xeon(R) CPU E7-8890 v3 a 2,50 GHz | 8 | 1 | 215,2 | 88 066 | 1391 | 1,58 % | 42 |

## <a name="ncsv3---gpu-enabled"></a>NCSv3: habilitada para GPU
(21/03/2019 5:48:37 PM pbi 3897709)

| Tamaño de VM | CPU | vCPU | Nodos NUMA | Memoria (GiB) | Puntuación media | Desarrollo estándar | % desarrollo estándar | N. º de ejecuciones |
| --- | --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Standard_NC6s_v3 | Intel(R) Xeon(R) CPU E5-2690 v4 a 2,60 GHz | 6 | 1 | 110,2 | 106 929 | 353 | 0,33 % | 49 |
| Standard_NC12s_v3 | Intel(R) Xeon(R) CPU E5-2690 v4 a 2,60 GHz | 12 | 1 | 220,4 | 213 585 | 875 | 0,41 % | 42 |
| Standard_NC24rs_v3 | Intel(R) Xeon(R) CPU E5-2690 v4 a 2,60 GHz | 24 | 2 | 440,9 | 403 554 | 6705 | 1,66 % | 42 |
| Standard_NC24s_v3 | Intel(R) Xeon(R) CPU E5-2690 v4 a 2,60 GHz | 24 | 2 | 440,9 | 403 874 | 7603 | 1,88 % | 42 |

## <a name="ncsv2---gpu-enabled"></a>NCSv2: habilitada para GPU
(12/03/2019 11:19:19 PM pbi 3897709)

| Tamaño de VM | CPU | vCPU | Nodos NUMA | Memoria (GiB) | Puntuación media | Desarrollo estándar | % desarrollo estándar | N. º de ejecuciones |
| --- | --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Standard_NC6s_v2 | Intel(R) Xeon(R) CPU E5-2690 v4 a 2,60 GHz | 6 | 1 | 110,2 | 107 115 | 321 | 0,30 % | 63 |
| Standard_NC12s_v2 | Intel(R) Xeon(R) CPU E5-2690 v4 a 2,60 GHz | 12 | 1 | 220,4 | 213 814 | 656 | 0,31 % | 63 |
| Standard_NC24rs_v2 | Intel(R) Xeon(R) CPU E5-2690 v4 a 2,60 GHz | 24 | 2 | 440,9 | 401 728 | 6995 | 1,74 % | 63 |
| Standard_NC24s_v2 | Intel(R) Xeon(R) CPU E5-2690 v4 a 2,60 GHz | 24 | 2 | 440,9 | 402 808 | 7923 | 1,97 % | 63 |

## <a name="nc---gpu-enabled"></a>NC: habilitada para GPU
(12/03/2019 11:08:03 PM pbi 3897709)

| Tamaño de VM | CPU | vCPU | Nodos NUMA | Memoria (GiB) | Puntuación media | Desarrollo estándar | % desarrollo estándar | N. º de ejecuciones |
| --- | --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Standard_NC6 | Intel(R) Xeon(R) CPU E5-2690 v3 a 2,60 GHz | 6 | 1 | 55,0 | 102 211 | 658 | 0,64 % | 259 |
| Standard_NC12 | Intel(R) Xeon(R) CPU E5-2690 v3 a 2,60 GHz | 12 | 1 | 110,2 | 203 523 | 2293 | 1,13 % | 259 |
| Standard_NC24 | Intel(R) Xeon(R) CPU E5-2690 v3 a 2,60 GHz | 24 | 2 | 220,4 | 382 897 | 8712 | 2,28 % | 259 |
| Standard_NC24r | Intel(R) Xeon(R) CPU E5-2690 v3 a 2,60 GHz | 24 | 2 | 220,4 | 383 171 | 9166 | 2,39 % | 259 |

## <a name="nds--gpu-enabled"></a>NDs: habilitada para GPU
(12/03/2019 11:19:10 PM pbi 3897709)

| Tamaño de VM | CPU | vCPU | Nodos NUMA | Memoria (GiB) | Puntuación media | Desarrollo estándar | % desarrollo estándar | N. º de ejecuciones |
| --- | --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Standard_ND6s | Intel(R) Xeon(R) CPU E5-2690 v4 a 2,60 GHz | 6 | 1 | 110,2 | 107 095 | 353 | 0,33 % | 63 |
| Standard_ND12s | Intel(R) Xeon(R) CPU E5-2690 v4 a 2,60 GHz | 12 | 1 | 220,4 | 212 298 | 3457 | 1,63 % | 63 |
| Standard_ND24rs | Intel(R) Xeon(R) CPU E5-2690 v4 a 2,60 GHz | 24 | 2 | 440,9 | 402 749 | 7838 | 1,95 % | 56 |
| Standard_ND24s | Intel(R) Xeon(R) CPU E5-2690 v4 a 2,60 GHz | 24 | 2 | 440,9 | 401 822 | 7776 | 1,94 % | 63 |

## <a name="nv---gpu-enabled"></a>NV: habilitada para GPU
(12/03/2019 11:08:13 PM pbi 3897709)

| Tamaño de VM | CPU | vCPU | Nodos NUMA | Memoria (GiB) | Puntuación media | Desarrollo estándar | % desarrollo estándar | N. º de ejecuciones |
| --- | --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Standard_NV6 | Intel(R) Xeon(R) CPU E5-2690 v3 a 2,60 GHz | 6 | 1 | 55,0 | 101 728 | 2094 | 2,06 % | 259 |
| Standard_NV12 | Intel(R) Xeon(R) CPU E5-2690 v3 a 2,60 GHz | 12 | 1 | 110,2 | 203 903 | 1724 | 0,85 % | 252 |
| Standard_NV24 | Intel(R) Xeon(R) CPU E5-2690 v3 a 2,60 GHz | 24 | 2 | 220,4 | 379 879 | 8737 | 2,30 % | 259 |


## <a name="about-coremark"></a>Acerca de CoreMark
Los números de Linux se procesaron ejecutando [CoreMark](https://www.eembc.org/coremark/faq.php) en Ubuntu. CoreMark se configuró con el número de subprocesos establecido en el número de CPU virtuales y la simultaneidad establecida en PThreads. El número objetivo de iteraciones se ajustó basándose en el rendimiento esperado para proporcionar un tiempo de ejecución de al menos 20 segundos (suele ser mucho mayor). La puntuación final representa el número de iteraciones completadas dividido entre el número de segundos que tardó en ejecutarse la prueba. Cada prueba se ejecutó al menos siete veces en cada máquina virtual. Las fechas de la serie de pruebas aparecen más arriba. La prueba se ejecuta en varias máquinas virtuales de las regiones públicas de Azure donde eran compatibles en la fecha de ejecución. Las series A y B (ampliables) no aparecen porque el rendimiento es variable. La serie N no aparece porque se basan en la GPU y Coremark no mide el rendimiento de la GPU.

## <a name="next-steps"></a>Pasos siguientes
* Para más información sobre las capacidades de almacenamiento, los detalles del disco y consideraciones adicionales para seleccionar tamaños de máquinas virtuales, consulte [Tamaños de máquinas virtuales](../sizes.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json).
* Para ejecutar los scripts de CoreMark en máquinas virtuales Linux, descargue el [paquete de scripts de CoreMark](https://download.microsoft.com/download/3/0/5/305A3707-4D3A-4599-9670-AAEB423B4663/AzureCoreMarkScriptPack.zip).