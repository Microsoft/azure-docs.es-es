---
title: Escalado de aplicaciones de HPC (Azure Virtual Machines) | Microsoft Docs
description: Aprenda a escalar aplicaciones de informática de alto rendimiento en máquinas virtuales de Azure.
services: virtual-machines
documentationcenter: ''
author: vermagit
manager: gwallace
editor: ''
tags: azure-resource-manager
ms.service: virtual-machines
ms.workload: infrastructure-services
ms.topic: article
ms.date: 05/15/2019
ms.author: amverma
ms.openlocfilehash: 72178b61d7033167ed48a8ddbb661daad6081df7
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87020122"
---
# <a name="scaling-hpc-applications"></a>Escalado de aplicaciones de HPC

Para que el rendimiento tanto de la escalabilidad vertical como de la horizontal de las aplicaciones de HPC sea óptimo en Azure es preciso realizar experimentos para lograr ajustar el rendimiento y optimizarlas para la carga de trabajo específica. Tanto esta sección como las páginas específicas de la serie de las máquinas virtuales ofrecen una guía general para escalar las aplicaciones.

## <a name="compiling-applications"></a>Compilación de aplicaciones

Aunque no es necesario, la compilación de aplicaciones con las marcas de optimización adecuadas proporciona el mejor rendimiento de escalado vertical en máquinas virtuales de las series HC y HB.

### <a name="amd-optimizing-cc-compiler"></a>Compilador AMD Optimizing C/C++ Compiler

El compilador AMD Optimizing C/C++ Compiler (AOCC) ofrece un alto nivel de optimizaciones avanzadas, subprocesamiento múltiple y compatibilidad de procesador que incluye la optimización global, la vectorización, el análisis entre procedimientos, las transformaciones de bucle y la generación de código. Los archivos binarios del compilador AOCC son adecuados para sistemas Linux que tengan la versión 2.17 de la biblioteca GNU C Library (glibc), y cualquier versión superior. El conjunto del compilador consta de un compilador de C/C++ (clang), un compilador de Fortran (FLANG) y un front-end de Fortran para Clang (Dragon Egg).

### <a name="clang"></a>Clang

Clang es un compilador para C, C++ y Objective-C que se usa para controlar el preprocesamiento, el análisis, la optimización, la generación de código, el ensamblado y la vinculación. Clang admite la marca `-march=znver1`, que permite mejorar y ajustar la generación de código para la arquitectura X86 basada en Zen de AMD.

### <a name="flang"></a>FLANG

El compilador FLANG es una incorporación reciente al conjunto AOCC (se agregó en abril de 2018) y la versión que está a disposición de los programadores para que la descarguen y hagan pruebas es la versión preliminar. Se basa en Fortran 2008 y AMD amplía la versión de GitHub de FLANG (https://github.com/flang-compiler/flang). El compilador FLANG admite todas las opciones del compilador Clang y, además, tiene varias opciones adicionales específicas del compilador FLANG.

### <a name="dragonegg"></a>DragonEgg

DragonEgg es un complemento de gcc que reemplaza los optimizadores y los generadores de código de GCC por los del proyecto LLVM. DragonEgg se incluye con AOCC, funciona con gcc 4.8.x, se ha probado para destinos x86-32/x86-64 y se ha utilizado correctamente en diversas plataformas de Linux.

GFortran es el front-end real para los programas de Fortran responsables del preprocesamiento, la redistribución y el análisis semántico que genera la representación intermedia de GCC GIMPLE (IR). DragonEgg es un complemento de GNU que conecta con el flujo de compilación de GFortran. Implementa la API del complemento de GNU. Con la arquitectura del complemento, DragonEgg se convierte en el controlador del compilador e impulsa las distintas fases de compilación.  Después de seguir las instrucciones de descarga e instalación, DragonEgg se puede invocar mediante: 

```bash
$ gfortran [gFortran flags] 
   -fplugin=/path/AOCC-1.2-Compiler/AOCC-1.2-     
   FortranPlugin/dragonegg.so [plugin optimization flags]     
   -c xyz.f90 $ clang -O3 -lgfortran -o xyz xyz.o $./xyz
```
   
### <a name="pgi-compiler"></a>Compilador PGI
Está confirmado que PGI Community Edition, versión 17 funciona con AMD EPYC. Una versión compilada por PGI de STREAM entrega todo el ancho de banda de la memoria de la plataforma. La más reciente Community Edition 18.10 (noviembre de 2018) también debería funcionar correctamente. A continuación encontrará una CLI de ejemplo para realizar una compilación óptima con el compilador Intel:

```bash
pgcc $(OPTIMIZATIONS_PGI) $(STACK) -DSTREAM_ARRAY_SIZE=800000000 stream.c -o stream.pgi
```

### <a name="intel-compiler"></a>Compilador Intel
Está confirmado que la versión 18 del compilador Intel funciona con AMD EPYC. A continuación encontrará una CLI de ejemplo para realizar una compilación óptima con el compilador Intel.

```bash
icc -o stream.intel stream.c -DSTATIC -DSTREAM_ARRAY_SIZE=800000000 -mcmodel=large -shared-intel -Ofast –qopenmp
```

### <a name="gcc-compiler"></a>Compilador GCC 
En el caso de HPC, AMD la versión 7.3 del compilador GCC, o cualquier versión posterior. Las versiones anteriores, como la 4.8.5, que se incluía en RHEL/CentOS 7.4, no se recomiendan. GCC 7.3, y las versiones posteriores, ofrecerán un rendimiento considerablemente mejor en las pruebas de HPL, HPCG y DGEMM.

```bash
gcc $(OPTIMIZATIONS) $(OMP) $(STACK) $(STREAM_PARAMETERS) stream.c -o stream.gcc
```

## <a name="scaling-applications"></a>Escalado de aplicaciones 

Mediante la aplicación de las siguientes sugerencias se logra una coherencia, rendimiento y eficiencia óptimos en el escalado de aplicaciones:

* Anclar procesos a los núcleos 0-59 mediante un enfoque de anclaje secuencial (en lugar de un enfoque de equilibrio automático). 
* El enlace por Numa/Core/HwThread es mejor que el enlace predeterminado.
* En el caso de aplicaciones híbridas paralelas (OpenMP+MPI), utilice cuatro subprocesos y una clasificación MPI por CCX.
* En el caso de aplicaciones de MPI puras, pruebe con 1-4 clasificaciones MPI por CCX para lograr un rendimiento óptimo.
* Algunas aplicaciones con una sensibilidad extrema al ancho de banda de memoria pueden beneficiarse del uso de un menor número de núcleos por CCX. En el caso de estas aplicaciones, el uso de 2 o 3 núcleos por CCX puede reducir la contención del ancho de banda de memoria y producir mayor rendimiento real o una escalabilidad más coherente. En concreto, MPI Allreduce se puede beneficiar de esto.
* En el caso de ejecuciones a mucho mayor escala, se recomienda usar transportes UD o RC + UD híbridos. Muchas bibliotecas o bibliotecas del runtime de MPI lo hacen internamente (como UCX o MVAPICH2). Compruebe las configuraciones de transporte en las ejecuciones a gran escala.

## <a name="next-steps"></a>Pasos siguientes

Obtenga más información acerca de [HPC](/azure/architecture/topics/high-performance-computing/) en Azure.
