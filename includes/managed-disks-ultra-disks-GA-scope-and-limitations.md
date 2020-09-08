---
title: archivo de inclusión
description: archivo de inclusión
services: virtual-machines
author: roygara
ms.service: virtual-machines
ms.topic: include
ms.date: 06/25/2020
ms.author: rogarana
ms.custom: include file
ms.openlocfilehash: 546c21802d275fe99ed2acbf00e32d37db3603f2
ms.sourcegitcommit: 152c522bb5ad64e5c020b466b239cdac040b9377
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/14/2020
ms.locfileid: "88225210"
---
Por ahora, los discos Ultra tienen limitaciones adicionales, como se indica a continuación:

Las únicas opciones de redundancia de infraestructura disponibles actualmente para Ultra Disks son las zonas de disponibilidad. Las máquinas virtuales que usan cualquier otra opción de redundancia no pueden conectar discos Ultra Disks.

En la tabla siguiente se describen las regiones en las que los discos Ultra Disks están disponibles, así como las opciones de disponibilidad correspondientes:

> [!NOTE]
> Si una región de la lista siguiente no tiene ninguna zona de disponibilidad compatible con Disco Ultra, las VM de esa región deben implementarse sin ninguna opción de redundancia de infraestructura para poder conectar un Disco Ultra.

|Regions  |Número de zonas de disponibilidad que admiten Discos Ultra  |
|---------|---------|
|US Gov - Virginia     |None         |
|US Gov: Arizona     |None         |
|Centro-sur de EE. UU.     |None         |
|Centro de EE. UU.     |Tres zonas         |
|Oeste de EE. UU.     |None         |
|Oeste de EE. UU. 2    |Tres zonas         |
|Este de EE. UU.     |Tres zonas         |
|Este de EE. UU. 2     |Tres zonas         |
|Sudeste de Asia     |Tres zonas         |
|Este de Asia     |None         |
|Norte de Europa     |Tres zonas          |
|Oeste de Europa     |Tres zonas          |
|Sur de Reino Unido     |Tres zonas          |
|Japón Oriental     |Tres zonas         |
|Centro de Francia    |Dos zonas        |
|Sur de Brasil    |None        |
|Este de Australia    |Tres zonas        |
|Centro de Canadá*    |Tres zonas        |

\* Póngase en contacto con el soporte técnico de Azure para obtener acceso a Availability Zones para esta región.

- Solo se admiten en las siguientes series de máquinas virtuales:
    - [ESv3](../articles/virtual-machines/ev3-esv3-series.md#esv3-series)
    - [Easv4](../articles/virtual-machines/eav4-easv4-series.md#easv4-series)
    - [Edsv4](../articles/virtual-machines/edv4-edsv4-series.md#edsv4-series)
    - [Esv4](../articles/virtual-machines/ev4-esv4-series.md#esv4-series)
    - [DSv3](../articles/virtual-machines/dv3-dsv3-series.md#dsv3-series)
    - [FSv2](../articles/virtual-machines/fsv2-series.md)
    - [LSv2](../articles/virtual-machines/lsv2-series.md)
    - [M](../articles/virtual-machines/workloads/sap/hana-vm-operations-storage.md)
    - [Mv2](../articles/virtual-machines/workloads/sap/hana-vm-operations-storage.md)
- No todos los tamaños de máquina virtual están disponibles en las regiones admitidas con discos Ultra
- Solo están disponibles como discos de datos y solo admiten un tamaño de sector físico de 4000. Debido al tamaño de sector nativo de 4000 de los discos Ultra, hay algunas aplicaciones que no son compatibles con ellos. Un ejemplo sería Oracle Database, que requiere la versión 12.2, o posterior, para admitir discos Ultra.  
- Solo pueden crearse como discos vacíos.  
- Actualmente no admiten instantáneas de disco, imágenes de máquinas virtuales, conjuntos de disponibilidad, instancias de Azure Dedicated Host ni Azure Disk Encryption.
- Actualmente no admiten la integración con Azure Backup o Azure Site Recovery.
- Solo admite lecturas y escrituras sin almacenamiento en caché.
- El límite máximo actual de IOPS en máquinas virtuales de disponibilidad general es 80 000.

De forma predeterminada, los discos Ultra Disks de Azure ofrecen hasta 16 TiB por región y suscripción, pero los discos Ultra admiten una mayor capacidad por solicitud. Para solicitar un aumento del límite, póngase en contacto con Soporte técnico de Azure.