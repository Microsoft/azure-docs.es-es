---
title: archivo de inclusión
description: archivo de inclusión
services: virtual-machines-linux
author: cynthn
ms.service: virtual-machines-linux
ms.topic: include
ms.date: 02/11/2019
ms.author: cynthn
ms.custom: include file
ms.openlocfilehash: 99c8ee79c51205c33d14328530f527ad8e3ff7b4
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87085474"
---
## <a name="supported-distributions-and-drivers"></a>Distribuciones y controladores admitidos

### <a name="nvidia-cuda-drivers"></a>Controladores NVIDIA CUDA

Los controladores NVIDIA CUDA para máquinas virtuales de las series NC, NCv2, NCv3, ND y NDv2 (y opcionales para la serie NV) solo se admiten en las distribuciones de Linux enumeradas en la tabla siguiente. La información sobre los controladores de CUDA está actualizada en el momento de su publicación. Para obtener los controladores de CUDA y los sistemas operativos compatibles más recientes, visite el sitio web de [NVIDIA](https://developer.nvidia.com/cuda-zone). Asegúrese de instalar o actualizar los controladores más recientes de CUDA para su distribución. 

> [!TIP]
> Como alternativa a la instalación manual de controladores de CUDA en una máquina virtual Linux, puede implementar una imagen de [Data Science Virtual Machine](../articles/machine-learning/data-science-virtual-machine/overview.md) de Azure. Las ediciones de DSVM para Ubuntu 16.04 LTS o CentOS 7.4 preinstalan los controladores NVIDIA CUDA, la biblioteca CUDA Deep Neural Network Library y otras herramientas.


### <a name="nvidia-grid-drivers"></a>Controladores de NVIDIA GRID

Microsoft redistribuye los instaladores del controlador NVIDIA GRID para VM de las series NV y NVv3 que se emplean como estaciones de trabajo virtuales o para aplicaciones virtuales. Instale estos controladores GRID en máquinas virtuales de la serie NV de Azure y solo en los sistemas operativos enumerados en la tabla siguiente. Estos controladores incluyen licencias del software GRID Virtual GPU en Azure. No es necesario configurar un servidor de licencias de software vGPU NVIDIA.

Los controladores de GRID redistribuidos por Azure no funcionan en máquinas virtuales que no son de la serie NV, como las máquinas virtuales de las series NC (NCv2, NCv3, ND y NDv2).

| Distribución | Controlador |
| --- | -- |
|Ubuntu 18.04 LTS<br/><br/>Ubuntu 16.04 LTS<br/><br/>Red Hat Enterprise Linux 7.6 a 7.8, 8.0, 8.1<br/><br/>Basado en CentOS 7.6, 7.7, 8.0 u 8(1911)<br/><br/>SUSE Linux Enterprise Server 12 SP2 | NVIDIA GRID 11, rama de controlador R450|

> [!WARNING] 
> La instalación de software de terceros en productos de Red Hat puede afectar a los términos de soporte técnico de Red Hat. Vea el [artículo de Knowledgebase de Red Hat](https://access.redhat.com/articles/1067).
>
