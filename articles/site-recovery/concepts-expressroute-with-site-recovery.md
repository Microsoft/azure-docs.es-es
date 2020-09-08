---
title: Acerca del uso de ExpressRoute con Azure Site Recovery
description: Describe cómo usar Azure ExpressRoute con el servicio Azure Site Recovery para la recuperación ante desastres y la migración.
services: site-recovery
author: mayurigupta13
manager: rochakm
ms.service: site-recovery
ms.topic: conceptual
ms.date: 10/13/2019
ms.author: mayg
ms.openlocfilehash: 46db5f7d3e5d3844fb297e512d8d701e6da79de9
ms.sourcegitcommit: 271601d3eeeb9422e36353d32d57bd6e331f4d7b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88654317"
---
# <a name="azure-expressroute-with-azure-site-recovery"></a>Azure ExpressRoute con Azure Site Recovery

Microsoft Azure ExpressRoute le permite ampliar sus redes locales en la nube de Microsoft a través de una conexión privada que facilita un proveedor de conectividad. Con ExpressRoute, se pueden establecer conexiones con servicios en la nube de Microsoft, como Microsoft Azure, Office 365 y Dynamics 365.

En este artículo, se describe cómo puede usar Azure ExpressRoute con Azure Site Recovery para la recuperación ante desastres y la migración.

## <a name="expressroute-circuits"></a>Circuitos ExpressRoute

Un circuito ExpressRoute representa una conexión lógica entre la infraestructura local y los servicios en la nube de Microsoft a través de un proveedor de conectividad. Puede solicitar varios circuitos ExpressRoute. Cada circuito puede estar en la misma región o en diferentes, y puede estar conectado a su entorno local mediante distintos proveedores de conectividad. Obtenga más información sobre los circuitos ExpressRoute [aquí](../expressroute/expressroute-circuit-peerings.md).

Un circuito ExpressRoute tiene asociados varios dominios de enrutamiento. Obtenga más información y una comparación sobre los dominios de enrutamiento de ExpressRoute [aquí](../expressroute/expressroute-circuit-peerings.md#peeringcompare).

## <a name="on-premises-to-azure-replication-with-expressroute"></a>Replicación desde un sitio local en Azure con ExpressRoute

Azure Site Recovery permite la recuperación ante desastres y la migración a Azure para las [máquinas virtuales de Hyper-V](hyper-v-azure-architecture.md), [máquinas virtuales de VMware](vmware-azure-architecture.md) y [servidores físicos](physical-azure-architecture.md) locales. Para todos los escenarios de replicación de sitios locales a Azure, los datos de replicación se envían a una cuenta de Azure Storage y se almacenan en ella. Durante la replicación, no paga ningún cargo de las máquinas virtuales. Cuando se ejecuta una conmutación por error en Azure, Site Recovery crea automáticamente máquinas virtuales IaaS de Azure.

Site Recovery replica datos a una cuenta de Azure Storage o a un disco administrado de réplica en la región de Azure de destino a través de un punto de conexión público. Para usar ExpressRoute para el tráfico de replicación de Site Recovery, puede utilizar el [emparejamiento de Microsoft](../expressroute/expressroute-circuit-peerings.md#microsoftpeering) o un [emparejamiento público](../expressroute/about-public-peering.md) existente (en desuso para las creaciones nuevas). El emparejamiento de Microsoft es el dominio de enrutamiento recomendado para la replicación. Observe que no se admite la replicación a través del emparejamiento privado.

Asegúrese de que también se cumplan los [requisitos de red](vmware-azure-configuration-server-requirements.md#network-requirements) para el servidor de configuración. El servidor de configuración requiere conectividad a direcciones URL específicas para la orquestación de la replicación de Site Recovery. Para esta conectividad no se puede usar ExpressRoute. 

En caso de que use el proxy de forma local y desee utilizar ExpressRoute para el tráfico de replicación, debe configurar la lista de omisión del proxy en el servidor de configuración y los servidores de proceso. Para hacerlo, siga estos pasos:

- Descargue la herramienta PsExec desde [aquí](https://aka.ms/PsExec) para acceder al contexto de usuario del sistema.
- Abra Internet Explorer en el contexto de usuario del sistema mediante la ejecución de la siguiente línea de comandos: psexec -s -i "%programfiles%\Internet Explorer\iexplore.exe"
- Adición de la configuración de proxy en Internet Explorer
- En la lista de omisión, agregue la dirección URL de Azure *.blob.core.windows.net.

Así se asegurará de que solo el tráfico de replicación fluye a través de ExpressRoute, mientras que la comunicación puede ir a través del proxy.

Una vez que las máquinas virtuales o los servidores conmutan por error en una red virtual de Azure, puede acceder a ellos a través del [emparejamiento privado](../expressroute/expressroute-circuit-peerings.md#privatepeering). 

El escenario combinado se representa en el siguiente diagrama: ![Del entorno local a Azure con ExpressRoute](./media/concepts-expressroute-with-site-recovery/site-recovery-with-expressroute.png)

## <a name="azure-to-azure-replication-with-expressroute"></a>Replicación de Azure a Azure con ExpressRoute

Azure Site Recovery permite la recuperación ante desastres de las [máquinas virtuales de Azure](azure-to-azure-architecture.md). En función de si las máquinas virtuales de Azure usan [Azure Managed Disks](../virtual-machines/managed-disks-overview.md), los datos de replicación se envían a una cuenta de Azure Storage o a una instancia de Managed Disks de réplica en la región de Azure de destino. De forma predeterminada, aunque los puntos de conexión de la replicación son públicos, el tráfico de replicación de las máquinas virtuales de Azure no atraviesa Internet, independientemente de la región de Azure en la que esté la red virtual de origen. Puede invalidar la ruta del sistema predeterminada de Azure para el prefijo de dirección 0.0.0.0/0 con una [ruta personalizada](../virtual-network/virtual-networks-udr-overview.md#custom-routes) y desviar el tráfico de la máquina virtual a una aplicación virtual de red (NVA) local, pero esta configuración no se recomienda para la replicación de Site Recovery. Si va a usar rutas personalizadas, debe [crear un punto de conexión de servicio de red virtual](azure-to-azure-about-networking.md#create-network-service-endpoint-for-storage) en su red virtual de "Storage" para que el tráfico de replicación no salga de los límites de Azure.

De forma predeterminada, para la recuperación ante desastres de máquinas virtuales de Azure no se necesita ExpressRoute para la replicación. Una vez que las máquinas virtuales se conmutan por error en la región de Azure de destino, puede acceder a ellas mediante el [emparejamiento privado](../expressroute/expressroute-circuit-peerings.md#privatepeering). Tenga en cuenta que los precios de transferencia de datos se aplican independientemente del modo de replicación de datos entre regiones de Azure.

Si ya usa ExpressRoute para conectarse desde el centro de datos local a las máquinas virtuales de Azure en la región de origen, puede planear el restablecimiento de la conectividad de ExpressRoute en la región de destino de la conmutación por error. Puede usar el mismo circuito ExpressRoute para conectarse a la región de destino a través de una nueva conexión de red virtual o usar un circuito ExpressRoute independiente y una conexión para la recuperación ante desastres. [Aquí](azure-vm-disaster-recovery-with-expressroute.md#fail-over-azure-vms-when-using-expressroute) se describen los posibles escenarios.

Puede replicar máquinas virtuales de Azure en cualquier región de Azure del mismo clúster geográfico como se detalla [aquí](../site-recovery/azure-to-azure-support-matrix.md#region-support). Si la región de Azure de destino elegida no está en la misma región geopolítica que la de origen, es posible que tenga que habilitar ExpressRoute Premium. Para obtener más información, consulte [Ubicaciones de ExpressRoute](../expressroute/expressroute-locations.md) y [Precios de ExpressRoute](https://azure.microsoft.com/pricing/details/expressroute/).

## <a name="next-steps"></a>Pasos siguientes
- Obtenga más información sobre los [circuitos ExpressRoute](../expressroute/expressroute-circuit-peerings.md).
- Obtenga más información sobre los [dominios de enrutamiento de ExpressRoute](../expressroute/expressroute-circuit-peerings.md#peeringcompare).
- Obtenga más información sobre las [ubicaciones de ExpressRoute](../expressroute/expressroute-locations.md).
- Más información sobre la recuperación ante desastres de [máquinas virtuales de Azure con ExpressRoute](azure-vm-disaster-recovery-with-expressroute.md).
