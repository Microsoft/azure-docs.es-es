---
title: archivo de inclusión
description: archivo de inclusión
services: virtual-machines
author: roygara
ms.service: virtual-machines
ms.topic: include
ms.date: 01/22/2019
ms.author: rogarana
ms.custom: include file
ms.openlocfilehash: c0627dd0833e3b20468eb5f50fbeb9fd9d9ae2b3
ms.sourcegitcommit: b33c9ad17598d7e4d66fe11d511daa78b4b8b330
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88864817"
---
**Transferencias de datos de salida**: [transferencias de datos de salida](https://azure.microsoft.com/pricing/details/bandwidth/) (datos que salen de los centros de datos de Azure) incurren en la facturación por el uso de ancho de banda.

**Transacciones**: se le factura por el número de transacciones que realiza en un disco administrado estándar. Para los discos SSD estándar, cada operación de E/S menor o igual a 256 KiB de rendimiento se considera una sola operación de E/S. Las operaciones de E/S mayores de 256 KiB de rendimiento se consideran varias operaciones de E/S con un tamaño de 256 KiB. En el caso de los discos HDD estándar, cada operación de E/S se considera como una única transacción, independientemente del tamaño de esta.

Para obtener información detallada sobre los precios de Managed Disks, incluidos los costes de las transacciones, consulte [Precios de Managed Disks](https://azure.microsoft.com/pricing/details/managed-disks).

### <a name="ultra-disk-vm-reservation-fee"></a>Precio de reserva de máquina virtual para discos Ultra

Las máquinas virtuales de Azure tienen la funcionalidad para indicar si son compatibles con discos Ultra. Una máquina virtual compatible con SSD Ultra asigna capacidad dedicada de ancho de banda entre la instancia de la máquina virtual de proceso y la unidad de escalado de almacenamiento en bloque, para así poder optimizar el rendimiento y reducir la latencia. Al agregar esta funcionalidad en la máquina virtual, se crea un cargo por reserva que solo se cobra si habilita la funcionalidad de discos Ultra en la máquina virtual sin asociarle uno de estos discos. Cuando un disco Ultra está asociado a la máquina virtual compatible con estos discos, este cargo no se aplicará. Este cargo se calcula según la vCPU aprovisionada en la máquina virtual. 

> [!Note]
> En el caso de los [tamaños principales de máquina virtual restringidos](../articles/virtual-machines/constrained-vcpu.md), la tarifa de la reserva se basa en el número real de vCPU y no en los núcleos restringidos. En el caso de Standard_E32-8s_v3, la tarifa de la reserva se basará en 32 núcleos. 

Para información sobre los precios de los discos Ultra, consulte la [página de precios de discos de Azure](https://azure.microsoft.com/pricing/details/managed-disks/).

### <a name="azure-disk-reservation"></a>Reserva de discos de Azure

La reserva de discos es la opción de adquirir de antemano un año de almacenamiento en discos con descuento, lo que reduce los costos totales. Al comprar una reserva de discos, selecciona una SKU de disco específica en una región de destino, por ejemplo, 10 SSD Premium de P30 (1 TiB) en la región Este de EE. UU. 2 por un plazo de un año. La experiencia de reserva es similar a las instancias reservadas de máquina virtual (VM). Puede agrupar las reservas de discos y máquinas virtuales para maximizar el ahorro. Por ahora, la reserva de discos de Azure ofrece un plan de compromiso de un año para las SKU de SSD Premium de P30 (1 TiB) a P80 (32 TiB) en todas las regiones de producción. Para más información sobre los precios de los discos reservados, consulte la [página de precios de los discos de Azure](https://azure.microsoft.com/pricing/details/managed-disks/).