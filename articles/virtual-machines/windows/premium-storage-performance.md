---
title: 'Azure Premium Storage: diseño de alto rendimiento'
description: Diseñe aplicaciones de alto rendimiento mediante discos administrados SSD Prémium de Azure. Premium Storage le ofrece compatibilidad con discos de alto rendimiento y baja latencia para cargas de trabajo con un uso intensivo de E/S, que se ejecutan en Azure Virtual Machines.
author: roygara
ms.service: virtual-machines-windows
ms.topic: conceptual
ms.date: 06/27/2017
ms.author: rogarana
ms.subservice: disks
ms.openlocfilehash: 46c917a2d27f5efaa1e902db0d5da5375578b8b8
ms.sourcegitcommit: 3543d3b4f6c6f496d22ea5f97d8cd2700ac9a481
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/20/2020
ms.locfileid: "86526049"
---
# <a name="azure-premium-storage-design-for-high-performance"></a>Azure Premium Storage: diseño de alto rendimiento
[!INCLUDE [virtual-machines-common-premium-storage-introduction](../../../includes/virtual-machines-common-premium-storage-introduction.md)]

> [!NOTE]
> A veces, lo que parece ser un problema de rendimiento del disco es realmente un cuello de botella de red. En estos casos, debería optimizarse el [rendimiento de la red](../../virtual-network/virtual-network-optimize-network-bandwidth.md).
>
> Si desea realizar pruebas comparativas de su disco, consulte nuestro artículo sobre cómo [realizar pruebas comparativas realizadas de un disco](disks-benchmarks.md).
>
> Si la VM admite redes aceleradas, debe asegurarse de que esta opción esté habilitada. Si no está habilitada, puede habilitarla en VM ya implementadas, tanto en [Windows](../../virtual-network/create-vm-accelerated-networking-powershell.md#enable-accelerated-networking-on-existing-vms) como en [Linux](../../virtual-network/create-vm-accelerated-networking-cli.md#enable-accelerated-networking-on-existing-vms).

Antes de comenzar, si es la primera vez que usa Premium Storage, lea antes los artículos acerca de la [Selección de un tipo de disco para máquinas virtuales IaaS de Azure](disks-types.md) y [Objetivos de escalabilidad de las cuentas de almacenamiento de blob en páginas Premium](../../storage/blobs/scalability-targets-premium-page-blobs.md).

[!INCLUDE [virtual-machines-common-premium-storage-performance.md](../../../includes/virtual-machines-common-premium-storage-performance.md)]

Si desea realizar pruebas comparativas de su disco, consulte nuestro artículo sobre cómo [realizar pruebas comparativas realizadas de un disco](disks-benchmarks.md).

Más información sobre los tipos de disco disponibles: [Selección de un tipo de disco](disks-types.md)  

Para los usuarios de SQL Server, lea artículos sobre procedimientos recomendados para SQL Server:

* [Procedimientos recomendados para SQL Server en Azure Virtual Machines](../../azure-sql/virtual-machines/windows/performance-guidelines-best-practices.md)
* [Azure Premium Storage proporciona el máximo rendimiento para SQL Server en una máquina virtual de Azure](https://cloudblogs.microsoft.com/sqlserver/2015/04/23/azure-premium-storage-provides-highest-performance-for-sql-server-in-azure-vm/)
