---
title: Migración desde AWS y otras plataformas a Managed Disks en Azure
description: Cree VM en Azure con VHD cargados desde otras nubes, como AWS u otras plataformas de virtualización, y aproveche las ventajas de Azure Managed Disks.
author: roygara
manager: twooley
ms.service: virtual-machines-windows
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-windows
ms.topic: conceptual
ms.date: 10/07/2017
ms.author: rogarana
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 1f363b8fc2fec659ee1b101c13d1ba6ac629f29d
ms.sourcegitcommit: 2ff0d073607bc746ffc638a84bb026d1705e543e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87835498"
---
# <a name="migrate-from-amazon-web-services-aws-and-other-platforms-to-managed-disks-in-azure"></a>Migración desde Amazon Web Services (AWS) y otras plataformas a Managed Disks en Azure

Puede cargar archivos de VHD desde AWS o soluciones de virtualización locales en Azure para crear máquinas virtuales que aprovechen las ventajas de Managed Disks. Azure Managed Disks elimina la necesidad de administrar las cuentas de almacenamiento para las VM IaaS de Azure. Solo tiene que especificar el tipo (premium o estándar) y el tamaño del disco que necesita, y Azure lo crea y administra automáticamente. 

Puede cargar VHD generalizados o especializados. 
- **VHD generalizado**: elimina toda la información personal de la cuenta mediante Sysprep. 
- **VHD especializado**: mantiene las cuentas de usuario, las aplicaciones y otros datos de estado de la máquina virtual original. 

> [!IMPORTANT]
> Antes de cargar los VHD en Azure, debe consultar [Preparación de un VHD o un VHDX de Windows antes de cargarlo en Azure](prepare-for-upload-vhd-image.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)
>
>


| Escenario                                                                                                                         | Documentación                                                                                                                       |
|----------------------------------------------------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------|
| Tiene instancias EC2 de AWS existentes que le gustaría migrar a máquinas virtuales de Azure mediante discos administrados.                              | [Migración de una máquina virtual de Amazon Web Services (AWS) a Azure](aws-to-azure.md)                           |
| Tiene una máquina virtual de otra plataforma de virtualización que le gustaría usar como imagen para crear varias máquinas virtuales de Azure. | [Carga de un VHD generalizado y uso de este para crear una nueva máquina virtual en Azure](upload-generalized-managed.md) |
| Cuenta con una VM personalizada de forma exclusiva que quisiera recrear en Azure.                                                      | [Carga de un VHD especializado en Azure y creación de una máquina nueva](create-vm-specialized.md)         |


## <a name="overview-of-managed-disks"></a>Información general de Managed Disks

Azure Managed Disks simplifica la administración de VM al eliminar la necesidad de administrar cuentas de almacenamiento. Managed Disks también aprovecha las ventajas de la mejor confiabilidad de las VM en un conjunto de disponibilidad. Esto garantiza que los discos de las distintas VM de un conjunto de disponibilidad estarán lo suficientemente aislados entre sí como para evitar un único punto de error. Coloca automáticamente discos de distintas VM en un conjunto de disponibilidad en unidades de escalado de almacenamiento diferentes (marcas de tiempo), lo que limita el impacto de errores únicos de unidad de escalado de almacenamiento generados debido a errores de hardware y software.
En virtud de sus necesidades, puede elegir entre cuatro tipos de opciones de almacenamiento. Para información acerca de los tipos de disco disponibles, consulte nuestro artículo [Selección de un tipo de disco](disks-types.md).

## <a name="plan-for-the-migration-to-managed-disks"></a>Planeación de la migración a Managed Disks

Esta sección puede ayudarlo a tomar la mejor decisión sobre los tipos de discos y VM.

Si planea migrar de discos no administrados a discos administrados, debe saber que los usuarios con el rol [Colaborador de máquina virtual](../../role-based-access-control/built-in-roles.md#virtual-machine-contributor) no podrán cambiar el tamaño de la máquina virtual (como podían hacer antes de la conversión). El motivo es que las máquinas virtuales con discos administrados requieren que el usuario tenga el permiso de escritura/discos/Microsoft.Compute para los discos del sistema operativo.

### <a name="location"></a>Location

Elija una ubicación donde Azure Managed Disks esté disponible. Si va a migrar a Managed Disks Premium, asegúrese que Premium Storage también está disponible en la región a la que planea migrar. Consulte [Servicios de Azure por región](https://azure.microsoft.com/regions/#services) para obtener información actualizada sobre las ubicaciones disponibles.

### <a name="vm-sizes"></a>Tamaños de VM

Si va a migrar a Managed Disks Premium, debe actualizar el tamaño de la VM a un tamaño compatible con Premium Storage disponible en la región donde se ubica la VM. Revise los tamaños de VM compatibles con Premium Storage. Las especificaciones de tamaño de las máquinas virtuales de Azure se muestran en [Tamaños de máquinas virtuales](../sizes.md).
Repase las características de rendimiento de las máquinas virtuales que trabajan con Premium Storage y elija el tamaño de máquina virtual que se mejor se ajuste a su carga de trabajo. Procure que haya suficiente ancho de banda disponible en la máquina virtual para dirigir el tráfico de disco.

### <a name="disk-sizes"></a>Tamaños de disco

**Managed Disks Premium**

Hay siete tipos de Managed Disks premium que se pueden usar con la máquina virtual y cada uno de ellos tiene sus límites específicos de rendimiento y E/S por segundo. Tenga presentes estos límites a la hora de elegir el tipo de disco Premium para la VM según las necesidades de capacidad, rendimiento, escalabilidad y cargas máximas de la aplicación.

| Tipo de discos Premium  | P4    | P6    | P10   | P15   | P20   | P30   | P40   | P50   | 
|---------------------|-------|-------|-------|-------|-------|-------|-------|-------|
| Tamaño del disco           | 32 GB| 64 GB| 128 GB| 256 GB|512 GB | 1\.024 GB (1 TB)    | 2048 GB (2 TB)    | 4095 GB (4 TB)    | 
| IOPS por disco       | 120   | 240   | 500   | 1100  |2300              | 5000              | 7500              | 7500              | 
| Rendimiento de disco. | 25 MB por segundo  | 50 MB por segundo  | 100 MB por segundo | 125 MB por segundo |150 MB por segundo | 200 MB por segundo | 250 MB por segundo | 250 MB por segundo |

**Discos administrados Estándar**

Hay siete tipos de Managed Disks estándar que se pueden usar con la máquina virtual. Cada uno de ellos tiene una capacidad distinta, pero los mismos límites de rendimiento y E/S por segundo. Elija el tipo de disco administrado Estándar según las necesidades de capacidad de la aplicación.

| Tipo de disco Estándar  | S4               | S6               | S10              | S15              | S20              | S30              | S40              | S50              | 
|---------------------|------------------|------------------|------------------|------------------|------------------|------------------|------------------|------------------| 
| Tamaño del disco           | 30 GB            | 64 GB            | 128 GB           | 256 GB           |512 GB           | 1\.024 GB (1 TB)   | 2048 GB (2 TB)    | 4095 GB (4 TB)   | 
| IOPS por disco       | 500              | 500              | 500              | 500              |500              | 500              | 500             | 500              | 
| Rendimiento de disco. | 60 MB por segundo | 60 MB por segundo | 60 MB por segundo | 60 MB por segundo |60 MB por segundo | 60 MB por segundo | 60 MB por segundo | 60 MB por segundo | 

### <a name="disk-caching-policy"></a>Directiva de almacenamiento en caché de disco 

**Managed Disks Premium**

De forma predeterminada, la directiva de almacenamiento en caché de los discos es *Solo lectura* para todos los discos de datos Premium y *Lectura y Escritura* para el disco de sistema operativo Premium conectado a la máquina virtual. Recomendamos esta opción de configuración para lograr el rendimiento óptimo de E/S de la aplicación. Para discos de datos de solo escritura o de gran cantidad de escritura (por ejemplo, archivos de registro de SQL Server), deshabilite el almacenamiento en caché de disco para lograr un mejor rendimiento de la aplicación.

### <a name="pricing"></a>Precios

Revise el [precio de Managed Disks](https://azure.microsoft.com/pricing/details/managed-disks/). Los precios de Managed Disks Premium son iguales que los de Unmanaged Disks Premium. Sin embargo, los precios de Managed Disks Estándar son distintos a los de los Unmanaged Disks Estándar.


## <a name="next-steps"></a>Pasos siguientes

- Antes de cargar los VHD en Azure, debe consultar [Preparación de un VHD o un VHDX de Windows antes de cargarlo en Azure](prepare-for-upload-vhd-image.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)