---
title: Procedimientos recomendados de configuración de clústeres
description: Obtenga información sobre las configuraciones de clúster admitidas al configurar la alta disponibilidad y la recuperación ante desastres (HADR) para SQL Server en Azure Virtual Machines, por ejemplo, los cuórum admitidos o las opciones de enrutamiento de conexión.
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
ms.openlocfilehash: de773bb2188f09822cae59ce42924a9a49f8087e
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87285635"
---
# <a name="cluster-configuration-best-practices-sql-server-on-azure-vms"></a>Procedimientos recomendados para la configuración de clústeres (SQL Server en máquinas virtuales de Azure)
[!INCLUDE[appliesto-sqlvm](../../includes/appliesto-sqlvm.md)]

Un clúster se usa para conseguir una alta disponibilidad y la recuperación ante desastres (HADR) con SQL Server en Azure Virtual Machines. 

En este artículo se proporcionan procedimientos recomendados de configuración de clústeres tanto para [instancias de clúster de conmutación por error (FCI)](failover-cluster-instance-overview.md) como para [grupos de disponibilidad](availability-group-overview.md) cuando se usan con SQL Server en máquinas virtuales de Azure. 


## <a name="networking"></a>Redes

Use una sola NIC por servidor (nodo de clúster) y una sola subred. La red de Azure tiene redundancia física, lo que hace que las NIC y subredes adicionales sean innecesarias en un clúster invitado de máquina virtual de Azure. El informe de validación de clúster le avisará de que solo se puede tener acceso a los nodos en una sola red. Puede omitir esta advertencia en los clústeres de conmutación por error invitados de máquinas virtuales de Azure.

## <a name="quorum"></a>Quorum

Aunque un clúster de dos nodos funcionará sin un [recurso de cuórum](/windows-server/storage/storage-spaces/understand-quorum), los clientes deben usar un recurso de quórum para tener soporte técnico de producción. La validación del clúster no aprobará ningún clúster sin un recurso de quórum. 

Técnicamente, un clúster de tres nodos puede sobrevivir a la pérdida de un solo nodo (hasta dos) sin un recurso de quórum. Sin embargo, después de que el clúster esté fuera de servicio en los dos nodos, existe el riesgo de que se ejecute en: 

- **Partición en el espacio** (cerebro dividido): los nodos del clúster se separan en la red debido a un problema del servidor, la NIC o el conmutador. 
- **Partición en el tiempo** (amnesia): Un nodo se une o vuelve a unirse al clúster, e intenta reclamar de forma inadecuada la propiedad del grupo de clústeres o de un rol del clúster. 

El recurso de quórum protege el clúster contra cualquiera de estos problemas. 

En la tabla siguiente se enumeran las opciones de cuórum disponibles en el orden de recomendación para uso con una máquina virtual de Azure; el testigo de disco es la opción preferida: 


||[Testigo de disco](/windows-server/failover-clustering/manage-cluster-quorum#configure-the-cluster-quorum)  |[Testigo en la nube](/windows-server/failover-clustering/deploy-cloud-witness)  |[Testigo de recurso compartido de archivos](/windows-server/failover-clustering/manage-cluster-quorum#configure-the-cluster-quorum)  |
|---------|---------|---------|---------|
|**Sistema operativo admitido**| All |Windows Server 2016+| Windows Server 2012+|




### <a name="disk-witness"></a>Testigo de disco

Se trata de un pequeño disco agrupado en el grupo de almacenamiento disponible del clúster. Este disco presenta una alta disponibilidad y puede conmutar por error entre nodos. Contiene una copia de la base de datos del clúster, con un tamaño predeterminado que suele ser inferior a 1 GB. El testigo de disco es la opción de cuórum preferida para una máquina virtual de Azure, ya que puede resolver el problema de la partición en el tiempo, a diferencia del testigo de nube y el testigo de recurso compartido de archivos. 

Configure un disco compartido de Azure como el testigo de disco. 

Para empezar, consulte [Configuración de un testigo de disco](/windows-server/failover-clustering/manage-cluster-quorum#configure-the-cluster-quorum).


**Sistema operativo compatible**: All   


### <a name="cloud-witness"></a>Testigo en la nube

Se trata de un tipo de testigo de cuórum de clúster de conmutación por error que usa Microsoft Azure para proporcionar un voto en el quórum de clúster. El tamaño predeterminado es de aproximadamente 1 MB y solo contiene la marca de tiempo. Un testigo en la nube resulta ideal para implementaciones en varios sitios, en varias zonas y en varias regiones.

Para empezar, consulte [Configuración de un testigo en la nube](/windows-server/failover-clustering/deploy-cloud-witness#CloudWitnessSetUp).


**Sistema operativo compatible**: Windows Server 2016 y posteriores   


### <a name="file-share-witness"></a>Testigo de recurso compartido de archivos

Se trata de un recurso compartido de archivos SMB que se suele configurar en un servidor de archivos que ejecuta Windows Server. Este testigo mantiene la información de la agrupación en clústeres en un archivo witness.log, pero no almacena una copia de la base de datos del clúster. En Azure, puede configurar un [recurso compartido de archivos de Azure](../../../storage/files/storage-how-to-create-file-share.md) para usarlo como testigo de recurso compartido de archivos o en una máquina virtual independiente.

Si va a usar un recurso compartido de archivos de Azure, puede montarlo con el mismo proceso que usó para el [recurso compartido de archivos Premium](failover-cluster-instance-premium-file-share-manually-configure.md#mount-premium-file-share). 

Para empezar, consulte [Configuración de un testigo de recurso compartido de archivos](/windows-server/failover-clustering/manage-cluster-quorum#configure-the-cluster-quorum).


**Sistema operativo compatible**: Windows Server 2012 y posteriores   

## <a name="connectivity"></a>Conectividad

En un entorno de red local tradicional, una instancia de clúster de conmutación por error de SQL Server parece ser una instancia única de SQL Server que se ejecuta en un único equipo. Dado que la instancia de clúster de conmutación por error conmuta por error de nodo a nodo, el nombre de red virtual (VNN) de la instancia proporciona un punto de conexión unificado y permite que las aplicaciones se conecten a la instancia de SQL Server sin conocer qué nodo está activo actualmente. Cuando se produce una conmutación por error, el nombre de red virtual se registra en el nuevo nodo activo después de iniciarse. Este proceso pasa desapercibido para el cliente o aplicación que se conecta a SQL Server y reduce al mínimo el tiempo de inactividad que la aplicación o el cliente experimenta durante un error. 

Use un nombre de red virtual (VNN) con Azure Load Balancer o un nombre de red distribuida (DNN) para enrutar el tráfico al VNN de la instancia de clúster de conmutación por error con SQL Server en máquinas virtuales de Azure. La característica DNN actualmente solo está disponible para SQL Server 2019 CU2 y versiones posteriores en una máquina virtual de Windows Server 2016 (o posterior). 

En la tabla siguiente se compara la compatibilidad de la conexión de HADR: 

| |**Nombre de red virtual (VNN)**  |**Nombre de red distribuida (DNN)**  |
|---------|---------|---------|
|**Versión mínima de sistema operativo**| Windows Server 2012 | Windows Server 2016|
|**Versión de SQL Server mínima** |SQL Server 2012 |SQL Server 2019 CU2|
|**Solución HADR compatible** | Instancia de clúster de conmutación por error <br/> grupo de disponibilidad | Instancia de clúster de conmutación por error|


### <a name="virtual-network-name-vnn"></a>Nombre de red virtual (VNN)

Dado que el punto de acceso a la dirección IP virtual funciona de forma diferente en Azure, debe configurar [Azure Load Balancer](../../../load-balancer/index.yml) para enrutar el tráfico a la dirección IP de los nodos de FCI. En máquinas virtuales de Azure, un equilibrador de carga hospeda la dirección IP del nombre de red virtual en el que se basan los recursos de SQL Server en clúster. El equilibrador de carga distribuye los flujos de entrada que llegan al front-end y, a continuación, enruta ese tráfico a las instancias definidas por el grupo de back-end. El flujo de tráfico se configura mediante reglas de equilibrio de carga y sondeos de estado. Con FCI de SQL Server, las instancias del grupo de back-end son las máquinas virtuales de Azure que ejecutan SQL Server. 

Hay un ligero retraso en la conmutación por error cuando se usa el equilibrador de carga, ya que el sondeo de estado realiza comprobaciones activas cada 10 segundos de forma predeterminada. 

Para empezar, aprenda a [configurar Azure Load Balancer para una FCI](hadr-vnn-azure-load-balancer-configure.md). 

**Sistema operativo compatible**: Windows Server 2012 y posteriores   
**Versión de SQL compatible**: SQL Server 2012 y posterior   
**Solución HADR admitida**: Instancia del clúster de conmutación por error y grupo de disponibilidad 


### <a name="distributed-network-name-dnn"></a>Nombre de red distribuida (DNN)

El nombre de red distribuida es una nueva característica de Azure para SQL Server 2019 CU2. El DNN proporciona una manera alternativa para que los clientes de SQL Server se conecten a la instancia de clúster de conmutación por error de SQL Server sin usar un equilibrador de carga. 

Cuando se crea un recurso DNN, el clúster enlaza el nombre DNS con las direcciones IP de todos los nodos del clúster. El cliente SQL intentará conectarse a cada dirección IP de esta lista para buscar el nodo en el que se está ejecutando actualmente la instancia de clúster de conmutación por error. Puede acelerar este proceso si especifica `MultiSubnetFailover=True` en la cadena de conexión. Esta configuración indica al proveedor que pruebe todas las direcciones IP en paralelo, por lo que el cliente puede conectarse a la FCI al instante. 

Se recomienda un nombre de red distribuida a través de un equilibrador de carga cuando sea posible porque: 
- La solución de un extremo a otro es más sólida, dado que ya no tiene que mantener el recurso del equilibrador de carga. 
- La eliminación de los sondeos del equilibrador de carga reduce al mínimo la duración de la conmutación por error. 
- El nombre de red distribuida simplifica el aprovisionamiento y la administración de la instancia de clúster de conmutación por error con SQL Server en máquinas virtuales de Azure. 

La mayoría de las características de SQL Server funcionan de manera transparente con FCI. En esos casos, basta con reemplazar el nombre DNS de VNN existente con el nombre DNS de DNN o con establecer el valor de DNN con el nombre DNS de VNN existente. Sin embargo, algunos componentes en el lado del servidor requieren un alias de red que asigne el nombre VNN al nombre DNN. Algunos casos específicos pueden requerir el uso explícito del nombre DNS de DNN, por ejemplo, al definir ciertas direcciones URL en una configuración de servidor. 

Para empezar, aprenda a [configurar un recurso DNN para una FCI](hadr-distributed-network-name-dnn-configure.md). 

**Sistema operativo compatible**: Windows Server 2016 y posteriores   
**Versión de SQL compatible**: SQL Server 2019 y versiones posteriores   
**Solución HADR admitida**: Solo una instancia de clúster de conmutación por error


## <a name="limitations"></a>Limitaciones

Tenga en cuenta las siguientes limitaciones cuando vaya a trabajar con FCI o con grupos de disponibilidad y SQL Server en Azure Virtual Machines. 

### <a name="msdtc"></a>MSDTC 
Azure Virtual Machines admite el Coordinador de transacciones distribuidas de Microsoft (MSDTC) en Windows Server 2019 con almacenamiento en Volúmenes compartidos en clúster (CSV) y [Azure Standard Load Balancer](../../../load-balancer/load-balancer-standard-overview.md).

En Azure Virtual Machines, MSDTC no se admite para Windows Server 2016 ni versiones anteriores porque:

- El recurso MSDTC en clúster no puede configurarse para usar almacenamiento compartido. En Windows Server 2016, si crea un recurso MSDTC, no mostrará ningún almacenamiento compartido disponible para su uso, incluso si el almacenamiento está disponible. Este problema se ha corregido en Windows Server 2019.
- El equilibrador de carga básico no controla los puertos RPC.


## <a name="next-steps"></a>Pasos siguientes

Una vez que haya determinado los procedimientos recomendados adecuados para su solución, empiece por [preparar la máquina virtual con SQL Server para FCI](failover-cluster-instance-prepare-vm.md). También puede crear el grupo de disponibilidad mediante la [CLI de Azure](availability-group-az-cli-configure.md) o las [plantillas de inicio rápido de Azure ](availability-group-quickstart-template-configure.md). 

