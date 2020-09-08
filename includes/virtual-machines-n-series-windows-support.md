---
title: archivo de inclusión
description: archivo de inclusión
services: virtual-machines-windows
author: cynthn
ms.service: virtual-machines-windows
ms.topic: include
ms.date: 02/11/2019
ms.author: cynthn
ms.custom: include file
ms.openlocfilehash: 00661043d1ec9769adbf4119a2c9c1925dcd29fa
ms.sourcegitcommit: c28fc1ec7d90f7e8b2e8775f5a250dd14a1622a6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/13/2020
ms.locfileid: "88186358"
---
## <a name="supported-operating-systems-and-drivers"></a>Sistemas operativos y controladores compatibles

### <a name="nvidia-tesla-cuda-drivers"></a>Controladores NVIDIA Tesla (CUDA)

Los controladores NVIDIA Tesla (CUDA) para máquinas virtuales de las series NC, NCv2, NCv3, NCasT4_v3, ND y NDv2 (opcionales para la serie NV) solo se admiten en las distribuciones de sistemas operativos enumeradas en la tabla siguiente. Los vínculos de descarga de controladores están actualizados en el momento de la publicación. Para ver los controladores más recientes, visite el sitio web de [NVIDIA](https://www.nvidia.com/).

> [!TIP]
> Como alternativa a la instalación manual de controladores de CUDA en una máquina virtual de Windows Server, puede implementar una imagen de [Data Science Virtual Machine](../articles/machine-learning/data-science-virtual-machine/overview.md) de Azure. Las ediciones de DSVM para Windows Server 2016 preinstalan los controladores NVIDIA CUDA, la biblioteca CUDA Deep Neural Network Library y otras herramientas.


| SO | Controlador |
| -------- |------------- |
| Windows Server 2019 | [451.82](http://us.download.nvidia.com/tesla/451.82/451.82-tesla-desktop-winserver-2019-2016-international.exe) (.exe) |
| Windows Server 2016 | [451.82](http://us.download.nvidia.com/tesla/451.82/451.82-tesla-desktop-winserver-2019-2016-international.exe) (.exe) |

### <a name="nvidia-grid-drivers"></a>Controladores de NVIDIA GRID

Microsoft redistribuye los instaladores del controlador NVIDIA GRID para VM de las series NV y NVv3 que se emplean como estaciones de trabajo virtuales o para aplicaciones virtuales. Instale estos controladores GRID en VM de la serie NV de Azure y solo en los sistemas operativos enumerados en la tabla siguiente. Estos controladores incluyen licencias del software GRID Virtual GPU en Azure. No es necesario configurar un servidor de licencias de software vGPU NVIDIA.

Los controladores de GRID redistribuidos por Azure no funcionan en máquinas virtuales que no son de la serie NV, como las máquinas virtuales de las series NC (NCv2, NCv3, ND y NDv2).

Tenga en cuenta que la extensión de NVIDIA siempre instalará el controlador más reciente. Aquí se proporcionan vínculos a la versión anterior para los clientes que tienen dependencias de una versión anterior.

En Windows Server 2019, Windows Server 2016 y Windows 10 (hasta la compilación 2004):
- [GRID 11 (451.48)](https://go.microsoft.com/fwlink/?linkid=874181) (.exe)
- [GRID 10.1 (442.06)](https://download.microsoft.com/download/b/8/f/b8f5ecec-b8f9-47de-b007-ac40adc88dc8/442.06_grid_win10_64bit_international_whql.exe) (.exe) 

En Windows Server 2012 R2: 
- [GRID 11 (451.48)](https://go.microsoft.com/fwlink/?linkid=874184) (.exe)
- [GRID 10.1 (442.66)](https://download.microsoft.com/download/4/3/3/4330fd5c-c685-4ca1-abca-3b2fb3c11d2e/442.06_grid_win8_win7_64bit_international_whql.exe) (.exe)  
