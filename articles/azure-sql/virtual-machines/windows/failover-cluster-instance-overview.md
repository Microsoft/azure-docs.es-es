---
title: Instancias de clúster de conmutación por error
description: Obtenga información sobre las instancias de clúster de conmutación por error (FCI) con SQL Server en Azure Virtual Machines.
services: virtual-machines
documentationCenter: na
author: MashaMSFT
editor: monicar
tags: azure-service-management
ms.service: virtual-machines-sql
ms.topic: article
ms.tgt_pltfrm: vm-windows-sql-server
ms.workload: iaas-sql-server
ms.date: 06/02/2020
ms.author: mathoma
ms.openlocfilehash: e5862daa21f8bf0075bb1dee567cbe887ec32d72
ms.sourcegitcommit: 271601d3eeeb9422e36353d32d57bd6e331f4d7b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88653280"
---
# <a name="failover-cluster-instances-with-sql-server-on-azure-virtual-machines"></a>Instancias de clúster de conmutación por error con SQL Server en Azure Virtual Machines
[!INCLUDE[appliesto-sqlvm](../../includes/appliesto-sqlvm.md)]

En este artículo se presentan las diferencias de características cuando se trabaja con instancias de clúster de conmutación por error (FCI) para SQL Server en Azure Virtual Machines (VM). 

## <a name="overview"></a>Información general

SQL Server en VM de Azure usa la funcionalidad de clústeres de conmutación por error de Windows Server (WSFC) para proporcionar una alta disponibilidad local mediante redundancia en el nivel de instancia de servidor: una instancia de clúster de conmutación por error (FCI). Una FCI es una instancia única de SQL Server que se instala en los nodos WSFC (o solo en el clúster) y, posiblemente, en varias subredes. En la red, una FCI parece ser una instancia de SQL Server que se ejecuta en un único equipo. Pero la FCI proporciona conmutación por error de un nodo de WSFC a otro si el nodo actual deja de estar disponible.

El resto del artículo se centra en las diferencias de las instancias de clúster de conmutación por error cuando se usan con SQL Server en VM de Azure. Para más información acerca de la tecnología de clústeres de conmutación por error, consulte: 

- [Tecnologías de clúster de Windows](https://docs.microsoft.com/windows-server/failover-clustering/failover-clustering-overview)
- [Instancias del clúster de conmutación por error de SQL Server](https://docs.microsoft.com/sql/sql-server/failover-clusters/windows/always-on-failover-cluster-instances-sql-server)

## <a name="quorum"></a>Quorum

Las instancias de clúster de conmutación por error con SQL Server en Azure Virtual Machines admiten el uso de un testigo de disco, un testigo en la nube o un testigo de recurso compartido de archivos para el cuórum del clúster.

Para más información, consulte [Procedimientos recomendados de cuórum con VM mediante SQL Server en Azure](hadr-cluster-best-practices.md#quorum). 


## <a name="storage"></a>Storage

En entornos en clúster tradicionales en el entorno local, un clúster de conmutación por error de Windows utiliza una red de área de almacenamiento (SAN) a la que ambos nodos pueden acceder como almacenamiento compartido. Los archivos de SQL Server se hospedan en el almacenamiento compartido y solo el nodo activo puede acceder a ellos al mismo tiempo. 

SQL Server en VM de Azure ofrece varias opciones como solución de almacenamiento compartido para una implementación de instancias de clúster de conmutación por error de SQL Server: 

||[Discos compartidos de Azure](../../../virtual-machines/windows/disks-shared.md)|[Recursos compartidos de archivos Premium](../../../storage/files/storage-how-to-create-premium-fileshare.md) |[Espacios de almacenamiento directo (S2D)](/windows-server/storage/storage-spaces/storage-spaces-direct-overview)|
|---------|---------|---------|---------|
|**Versión mínima de sistema operativo**| All |Windows Server 2012|Windows Server 2016|
|**Versión de SQL Server mínima**|All|SQL Server 2012|SQL Server 2016|
|**Disponibilidad de VM admitida** |Conjuntos de disponibilidad con grupos de selección de ubicación de proximidad |Conjuntos y zonas de disponibilidad|Conjuntos de disponibilidad |
|**Admite FileStream**|Sí|No|Sí |
|**Caché de blobs de Azure**|No|No|Sí|

En el resto de esta sección se enumeran las ventajas y las limitaciones de cada opción de almacenamiento disponible para SQL Server en las máquinas virtuales de Azure. 

### <a name="azure-shared-disks"></a>Discos compartidos de Azure

[Los discos compartidos de Azure](../../../virtual-machines/windows/disks-shared.md) son una característica de [Azure Managed Disks](../../../virtual-machines/managed-disks-overview.md). Los clústeres de conmutación por error de Windows Server admiten el uso de discos compartidos de Azure con una instancia de clúster de conmutación por error. 

**Sistema operativo compatible**: All   
**Versión de SQL compatible**: All     

**Ventajas** 
- Resulta útil para las aplicaciones que desean migrar a Azure al tiempo que mantienen su arquitectura de alta disponibilidad y recuperación ante desastres (HADR) tal cual. 
- Puede migrar las aplicaciones en clúster a Azure tal cual gracias a la compatibilidad con las reservas persistentes de SCSI (SCSI PR). 
- Admite Azure SSD Premium compartido para todas las versiones de SQL Server y Almacenamiento en disco Ultra de Azure compartido para SQL Server 2019. 
- Puede usar un único disco compartido o seccionar varios para crear un grupo de almacenamiento compartido. 
- Admite FileStream.


**Limitaciones**: 
- Las máquinas virtuales se deben colocar en el mismo conjunto de disponibilidad y grupo de ubicación de proximidad.
- Availability Zones no se admite.
- El almacenamiento en caché de disco SSD Premium no se admite.
 
Para empezar, consulte [Creación de una FCI con discos compartidos de Azure (SQL Server en VM de Azure)](failover-cluster-instance-azure-shared-disks-manually-configure.md). 

### <a name="storage-spaces-direct"></a>Espacios de almacenamiento directo

[Espacios de almacenamiento directo](/windows-server/storage/storage-spaces/storage-spaces-direct-overview) es una característica de Windows Server compatible con los clústeres de conmutación por error en Azure Virtual Machines. Proporciona una SAN virtual basada en software.

**Sistema operativo compatible**: Windows Server 2016 y posteriores   
**Versión de SQL compatible**: SQL Server 2016 y posterior   


**Ventajas:** 
- El ancho de banda de red suficiente permite una solución de almacenamiento compartido sólida y de alto rendimiento. 
- Admite la memoria caché de blobs de Azure, por lo que las lecturas se pueden atender localmente desde ella. (Las actualizaciones se replican simultáneamente en ambos nodos). 
- Admite FileStream. 

**Limitaciones:**
- Solo está disponible para Windows Server 2016 y versiones posteriores. 
- Availability Zones no se admite.
- Requiere la misma capacidad de disco conectado a ambas máquinas virtuales. 
- Se requiere un ancho de banda de red alto para lograr un gran rendimiento debido a la replicación de disco en curso. 
- Necesita un tamaño de máquina virtual mayor y un doble pago por almacenamiento, ya que el almacenamiento está conectado a cada máquina virtual. 

Para empezar, consulte [Instancia de clúster de conmutación por error de SQL Server con Espacios de almacenamiento directo](failover-cluster-instance-storage-spaces-direct-manually-configure.md). 

### <a name="premium-file-share"></a>Recurso compartido de archivos Premium

[Los recursos compartidos de archivos Premium](../../../storage/files/storage-how-to-create-premium-fileshare.md) son una característica de [Azure Files](../../../storage/files/index.yml). Los recursos compartidos de archivos Premium están respaldados por SSD y tienen una latencia baja constante. Son totalmente compatibles con las instancias de clúster de conmutación por error para SQL Server 2012 o posterior en Windows Server 2012 o posterior. Los recursos compartidos de archivos Premium ofrecen mayor flexibilidad, lo que le permite cambiar el tamaño y escalar un recurso compartido de archivos sin tiempo de inactividad.

**Sistema operativo compatible**: Windows Server 2012 y posteriores   
**Versión de SQL compatible**: SQL Server 2012 y posterior   

**Ventajas:** 
- Solo una solución de almacenamiento compartido para las máquinas virtuales distribuidas en varias zonas de disponibilidad. 
- Sistema de archivos totalmente administrado con latencias de un solo dígito y rendimiento de E/S que se puede aumentar. 

**Limitaciones:**
- Solo está disponible para Windows Server 2012 y versiones posteriores. 
- No se admite FileStream. 


Para empezar, consulte [Instancia de clúster de conmutación por error de SQL Server con el recurso compartido de archivos Premium](failover-cluster-instance-premium-file-share-manually-configure.md). 

### <a name="partner"></a>Asociado

Hay soluciones de agrupación en clústeres de asociados con almacenamiento compatible. 

**Sistema operativo compatible**: All   
**Versión de SQL compatible**: All   

En un ejemplo se usa SIOS DataKeeper como almacenamiento. Para más información, consulte la entrada del blog [Clúster de conmutación por error y SIOS DataKeeper](https://azure.microsoft.com/blog/high-availability-for-a-file-share-using-wsfc-ilb-and-3rd-party-software-sios-datakeeper/).

### <a name="iscsi-and-expressroute"></a>iSCSI y ExpressRoute

También puede exponer un almacenamiento en bloque compartido de destino iSCSI a través de Azure ExpressRoute. 

**Sistema operativo compatible**: All   
**Versión de SQL compatible**: All   

Por ejemplo, NetApp Private Storage (NPS) expone un destino iSCSI a través de ExpressRoute con Equinix a las máquinas virtuales de Azure.

Para las soluciones de almacenamiento compartido y de replicación de datos de asociados de Microsoft, debe ponerse en contacto con el proveedor en caso de que surja algún problema relacionado con el acceso a los datos en la conmutación por error.

## <a name="connectivity"></a>Conectividad

Las instancias de clúster de conmutación por error con SQL Server en Azure Virtual Machines usan un [nombre de red distribuida (DNN)](hadr-distributed-network-name-dnn-configure.md) o un [nombre de red virtual (VNN) con Azure Load Balancer](hadr-vnn-azure-load-balancer-configure.md) para enrutar el tráfico a la instancia de SQL Server, independientemente del nodo que posea actualmente los recursos en clúster. Hay consideraciones adicionales cuando se usan ciertas características y el nombre de red distribuida con una FCI de SQL Server. Consulte [Interoperabilidad de DNN con FCI de SQL Server](failover-cluster-instance-dnn-interoperability.md) para más información. 

Para más información sobre las opciones de conectividad de clústeres, consulte [Enrutamiento de conexiones de HADR a SQL Server en máquinas virtuales de Azure](hadr-cluster-best-practices.md#connectivity). 

## <a name="limitations"></a>Limitaciones

Tenga en cuenta las limitaciones siguientes de las instancias de clúster de conmutación por error con SQL Server en Azure Virtual Machines. 

### <a name="lightweight-resource-provider"></a>Proveedor de recursos ligeros   
Actualmente, las instancias del clúster de conmutación por error de SQL Server en las máquinas virtuales de Azure solo se admiten con el modo de administración [ligera](sql-vm-resource-provider-register.md#management-modes) de la [extensión del agente de IaaS de SQL Server](sql-server-iaas-agent-extension-automate-management.md). Para cambiar del modo de extensión total a extensión ligera, elimine el recurso **Máquina virtual con SQL** de las máquinas virtuales correspondientes y regístrelas con el proveedor de recursos VM con SQL en el modo ligero. Cuando elimine el recurso **Máquina virtual con SQL** desde Azure Portal, desactive la casilla de la máquina virtual correcta. 

La extensión completa admite características como la copia de seguridad automatizada, y la aplicación de revisiones y la administración avanzada del portal. Estas características no funcionarán para las VM con SQL Server después de que el agente se vuelva a instalar en modo de administración ligera.

### <a name="msdtc"></a>MSDTC   
Azure Virtual Machines admite MSDTC en Windows Server 2019 con almacenamiento en volúmenes compartidos en clúster (CSV) y [Azure Standard Load Balancer](../../../load-balancer/load-balancer-standard-overview.md).

En Azure Virtual Machines, MSDTC no se admite para Windows Server 2016 o versiones anteriores porque:

- El recurso MSDTC en clúster no puede configurarse para usar almacenamiento compartido. En Windows Server 2016, si crea un recurso MSDTC, no mostrará ningún almacenamiento compartido disponible para su uso, incluso si el almacenamiento está disponible. Este problema se ha corregido en Windows Server 2019.
- El equilibrador de carga básico no controla los puertos RPC.


## <a name="next-steps"></a>Pasos siguientes

Revise los [procedimientos recomendados de configuración de clústeres](hadr-cluster-best-practices.md) y, después, puede [preparar la VM con SQL Server para la FCI](failover-cluster-instance-prepare-vm.md). 

Para más información, consulte: 

- [Tecnologías de clúster de Windows](/windows-server/failover-clustering/failover-clustering-overview)   
- [Instancias del clúster de conmutación por error de SQL Server](/sql/sql-server/failover-clusters/windows/always-on-failover-cluster-instances-sql-server)

