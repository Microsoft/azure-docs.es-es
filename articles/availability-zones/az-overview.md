---
title: ¿Qué son las zonas de disponibilidad de Azure? | Microsoft Docs
description: Para crear aplicaciones altamente disponibles y resistentes en Azure, las zonas de disponibilidad proporcionan ubicaciones físicamente separadas que puede utilizar para ejecutar sus recursos.
services: ''
documentationcenter: ''
author: cynthn
manager: jeconnoc
editor: ''
tags: ''
ms.assetid: ''
ms.service: azure
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 05/06/2019
ms.author: cynthn
ms.custom: mvc
ms.openlocfilehash: 92ef494efa62a7f4e2d0a7c88ef8efd950dd2472
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65786103"
---
# <a name="what-are-availability-zones-in-azure"></a>¿Qué son las zonas de disponibilidad en Azure?
Las zonas de disponibilidad son una oferta que protege las aplicaciones y datos de los errores del centro de datos. Las zonas de disponibilidad son ubicaciones físicas exclusivas dentro de una región de Azure. Cada zona de disponibilidad consta de uno o varios centros de datos equipados con alimentación, refrigeración y redes independientes. Para garantizar la resistencia, hay tres zonas independientes como mínimo en todas las regiones habilitadas. La separación física de las zonas de disponibilidad dentro de una región protege las aplicaciones y los datos frente a los errores del centro de datos. Los servicios con redundancia de zona replican las aplicaciones y los datos entre zonas de disponibilidad para protegerlos frente a puntos de error únicos. Con las zonas de disponibilidad, Azure ofrece el mejor Acuerdo de Nivel de Servicio del sector de tiempo de actividad de máquina virtual, con un 99,99 %. En el [SLA de Azure](https://azure.microsoft.com/support/legal/sla/virtual-machines/) completo se explica la disponibilidad garantizada de Azure como un conjunto.

Una zona de disponibilidad de una región de Azure es una combinación de un dominio de error y un dominio de actualización. Por ejemplo, si crea tres o más máquinas virtuales en tres zonas de una región de Azure, las máquinas virtuales se distribuyen eficazmente en tres dominios de error y tres dominios de actualización. La plataforma Azure reconoce esta distribución entre dominios de actualización para asegurarse de que las máquinas virtuales de distintas zonas no se actualizan al mismo tiempo.

Coloque sus recursos de proceso, almacenamiento, red y datos dentro de una zona y replíquelos en otras para conseguir una alta disponibilidad en la arquitectura de sus aplicaciones. Los servicios de Azure que admiten zonas de disponibilidad se dividen en dos categorías:

- **Servicios de zona**: ancle el recurso a una zona específica (por ejemplo, máquinas virtuales, discos administrados, direcciones IP)
- **Servicios de redundancia de zona**: la plataforma se replica automáticamente entre zonas (por ejemplo, almacenamiento con redundancia de zona, SQL Database).

Para lograr una continuidad del negocio integral en Azure, cree la arquitectura de aplicación mediante la combinación de zonas de disponibilidad y pares de regiones de Azure. Puede replicar de forma sincrónica las aplicaciones y los datos mediante zonas de disponibilidad dentro de una región de Azure para conseguir alta disponibilidad, y replicar de forma asincrónica entre regiones de Azure para la protección de recuperación ante desastres.
 
![vista conceptual de una zona fuera de servicio en una región](./media/az-overview/az-graphic-two.png)

## <a name="services-support-by-region"></a>Soporte técnico de servicios por región

Las combinaciones de servicios de Azure y regiones que admiten zonas de disponibilidad son:


|                                 |América |              |           |           | Europa |              |          |              | Asia Pacífico |                 |
|----------------------------|----------|----------|---------|---------|--------------|------------|--------|----------|----------|-------------|
|          |Centro de EE. UU.|Este de EE. UU|Este de EE. UU. 2|Oeste de EE. UU. 2|Centro de Francia|Europa del Norte|Sur de Reino Unido 2|Europa occidental|Este de Japón|Sudeste asiático|
| **Proceso**                         |            |              |           |           |                |              |          |             |            |                |
| Linux Virtual Machines          | &#10003;   | &#10003;     | &#10003;  | &#10003;  | &#10003;       | &#10003;     | &#10003; | &#10003;    | &#10003;   | &#10003;       |
| Windows Virtual Machines        | &#10003;   | &#10003;     | &#10003;  | &#10003;  | &#10003;       | &#10003;     | &#10003; | &#10003;    | &#10003;   | &#10003;       |
| Virtual Machine Scale Sets      | &#10003;   | &#10003;     | &#10003;  | &#10003;  | &#10003;       | &#10003;     | &#10003; | &#10003;    | &#10003;   | &#10003;       |
| **Storage**   |            |              |           |           |                |              |          |             |            |                |
| Managed Disks                   | &#10003;   | &#10003;     | &#10003;  | &#10003;  | &#10003;       | &#10003;     | &#10003; | &#10003;    | &#10003;   | &#10003;       |
| Almacenamiento con redundancia de zona          | &#10003;   | &#10003;     | &#10003;  | &#10003;  | &#10003;       | &#10003;     | &#10003; | &#10003;    | &#10003;   | &#10003;       |
| **Redes**                     |            |              |           |           |                |              |          |             |            |                |
| Dirección IP estándar        | &#10003;   | &#10003;     | &#10003;  | &#10003;  | &#10003;       | &#10003;     | &#10003; | &#10003;    | &#10003;   | &#10003;       |
| Standard Load Balancer     | &#10003;   | &#10003;     | &#10003;  | &#10003;  | &#10003;       | &#10003;     | &#10003; | &#10003;    | &#10003;   | &#10003;       |
| VPN Gateway                     | &#10003;   |              | &#10003;  | &#10003;  | &#10003;       | &#10003;     |          | &#10003;    |            | &#10003;       |
| ExpressRoute                    | &#10003;   |              | &#10003;  | &#10003;  | &#10003;       | &#10003;     |          | &#10003;    |            | &#10003;       |
| Application Gateway   | &#10003;   | &#10003;     | &#10003;  | &#10003;  | &#10003;       | &#10003;     |          | &#10003;    | &#10003;       | &#10003;       |
| **Bases de datos**                     |            |              |           |           |                |              |          |             |            |                |
| SQL Database                    | &#10003;   | &#10003;     | &#10003;  | &#10003;  | &#10003;       | &#10003;     | &#10003; | &#10003;    |            | &#10003;       |
| **Analytics**                       |            |              |           |           |                |              |          |             |            |                |
| Event Hubs                      | &#10003;   |              | &#10003;  | &#10003;  | &#10003;       | &#10003;     |          | &#10003;    |            | &#10003;       |
| **Integración**                     |            |              |           |           |                |              |          |             |            |                |
| Service Bus (solo nivel Premium) | &#10003;   |              | &#10003;  | &#10003;  | &#10003;       | &#10003;     |          | &#10003;    |            | &#10003;       |



## <a name="services-resiliency"></a>Resistencia de servicios
Todos los servicios de administración de Azure están diseñados para ser resistente ante errores de nivel de región. En el espectro de errores, uno o más errores de zona de disponibilidad dentro de una región tienen un radio de error más pequeño en comparación con un error de toda la región. Azure puede recuperar de un error de nivel de zona de servicios de administración dentro de la región o en otra región de Azure. Azure realiza una zona de mantenimiento crítico a la vez dentro de una región, para evitar los errores que afectan a los recursos de cliente implementados en zonas de disponibilidad dentro de una región.

## <a name="pricing"></a>Precios
No hay ningún costo adicional asociado a las máquinas virtuales implementadas en una zona de disponibilidad. Cuando dos o más máquinas virtuales se implementan en dos o más zonas de disponibilidad dentro de una región de Azure, se ofrece un Acuerdo de Nivel de Servicio de tiempo de actividad de máquina virtual del 99,99 %. Habrá gastos adicionales de transferencia de datos de máquina virtual a máquina virtual entre zonas de disponibilidad. Para más información, revise la página [Precios de ancho de banda](https://azure.microsoft.com/pricing/details/bandwidth/).


## <a name="get-started-with-availability-zones"></a>Introducción a las zonas de disponibilidad
- [Creación de una máquina virtual](../virtual-machines/windows/create-portal-availability-zone.md)
- [Agregación de un disco administrado mediante PowerShell](../virtual-machines/windows/attach-disk-ps.md#add-an-empty-data-disk-to-a-virtual-machine)
- [Creación de un conjunto de escalado de máquinas virtuales con redundancia de zona](../virtual-machine-scale-sets/virtual-machine-scale-sets-use-availability-zones.md)
- [Equilibrio de carga de máquinas virtuales en distintas zonas con un equilibrador de carga estándar con un front-end con redundancia de zona](../load-balancer/load-balancer-standard-public-zone-redundant-cli.md)
- [Equilibrio de carga de máquinas virtuales dentro de una zona con un equilibrador de carga estándar con un front-end de zona](../load-balancer/load-balancer-standard-public-zonal-cli.md)
- [Almacenamiento con redundancia de zona](../storage/common/storage-redundancy-zrs.md)
- [SQL Database](../sql-database/sql-database-high-availability.md#zone-redundant-configuration)
- [Recuperación ante desastres geográfica de Event Hubs](../event-hubs/event-hubs-geo-dr.md#availability-zones)
- [Recuperación ante desastres geográfica de Service Bus](../service-bus-messaging/service-bus-geo-dr.md#availability-zones)
- [Crear una puerta de enlace de red virtual con redundancia de zona](../vpn-gateway/create-zone-redundant-vnet-gateway.md)


## <a name="next-steps"></a>Pasos siguientes
- [Plantillas de inicio rápido](https://aka.ms/azqs)
