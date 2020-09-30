---
title: Grupos de seguridad de red con Azure Site Recovery | Microsoft Docs
description: Describe cómo usar los grupos de seguridad de red con Azure Site Recovery para la recuperación ante desastres y la migración
author: Harsha-CS
manager: rochakm
ms.service: site-recovery
ms.topic: conceptual
ms.date: 04/08/2019
ms.author: harshacs
ms.openlocfilehash: 904bc63ed2a135cdcadad75e96acd6fe3ca39039
ms.sourcegitcommit: 1fe5127fb5c3f43761f479078251242ae5688386
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/14/2020
ms.locfileid: "90069686"
---
# <a name="network-security-groups-with-azure-site-recovery"></a>Grupos de seguridad de red con Azure Site Recovery

Los grupos de seguridad de red sirven para limitar el tráfico a los recursos en una red virtual. Los [grupos de seguridad de red (NSG)](../virtual-network/security-overview.md#network-security-groups) contienen una lista de reglas de seguridad que permiten o deniegan el tráfico entrante o saliente en función de las direcciones IP de origen o destino, el puerto y el protocolo.

En el modelo de implementación del Administrador de recursos, los NSG se pueden asociar a subredes o a interfaces de red individuales. Cuando un grupo de seguridad de red está asociado a una subred, las reglas se aplican a todos los recursos conectados a la subred. El tráfico se puede restringir aún más mediante la asociación adicional de un grupo de seguridad de red a interfaces de red individuales dentro de una subred que ya tienen otro asociado.

En este artículo se describe el uso de los grupos de seguridad de red con Azure Site Recovery.

## <a name="using-network-security-groups"></a>Uso de los grupos de seguridad de red

Una subred individual puede o no tener un grupo de seguridad de red asociado. Una interfaz de red individual también puede o no tener un grupo de seguridad de red asociado. Por lo tanto, puede restringir eficazmente el tráfico dual para una máquina virtual mediante la asociación de un NSG, primero a una subred y, a continuación, otro NSG a la interfaz de red de la VM. En este caso, la aplicación de reglas de grupo de seguridad de red depende de la dirección del tráfico y la prioridad de las reglas de seguridad aplicadas.

Consideremos un ejemplo sencillo con una máquina virtual como se indica a continuación:
-    La máquina virtual se encuentra en la **subred Contoso**.
-    La **subred Contoso** está asociada al **grupo de seguridad de red de la subred**.
-    La interfaz de red de la máquina virtual también está asociada a un **grupo de seguridad de red de máquina virtual**.

![Grupo de seguridad de red con Site Recovery](./media/concepts-network-security-group-with-site-recovery/site-recovery-with-network-security-group.png)

En este ejemplo, para el tráfico entrante, primero se evalúa el grupo de seguridad de red de la subred. A continuación, el grupo de seguridad de red de la máquina virtual evalúa el tráfico permitido mediante el grupo de seguridad de red de la subred. Lo contrario es aplicable para el tráfico saliente, el grupo de seguridad de red de la máquina virtual se evalúa primero. A continuación, el grupo de seguridad de red de la subred evalúa el tráfico permitido mediante el grupo de seguridad de red de la máquina virtual.

Esto permite la aplicación de la regla de seguridad específica. Por ejemplo, puede que desee permitir el acceso entrante a Internet para algunas máquinas virtuales de aplicación (por ejemplo, las de front-end) en una subred, pero restringir el acceso a Internet para otras máquinas virtuales (por ejemplo, las de base de datos o de back-end). En este caso puede tener una regla más flexible en el grupo de seguridad de red de la subred que permita el tráfico de Internet y restringir el acceso para máquinas virtuales específicas al denegar el acceso en el grupo de seguridad de red de la máquina virtual. Lo mismo se aplica al tráfico saliente.

Al configurar estas configuraciones de grupo de seguridad de red, asegúrese de que las prioridades correctas se aplican a las [reglas de seguridad](../virtual-network/security-overview.md#security-rules). Las reglas se procesan en orden de prioridad. Se procesan primero las reglas con los números más bajos ya que estos tienen más prioridad. Si el tráfico coincide con una regla, se detiene el procesamiento. Como resultado, las reglas con menor prioridad (números más altos) que tengan los mismos atributos que las reglas con una prioridad mayor no se procesarán.

Puede que no siempre sepa cuándo se aplican los grupos de seguridad de red a una interfaz de red y a una subred. Puede verificar las reglas agregadas que se aplican a una interfaz de red mediante la visualización de las [reglas de seguridad vigentes](../virtual-network/virtual-network-network-interface.md#view-effective-security-rules) de una interfaz de red. También puede usar la funcionalidad [Comprobación del flujo de IP](../network-watcher/diagnose-vm-network-traffic-filtering-problem.md) de [Azure Network Watcher](../network-watcher/network-watcher-monitoring-overview.md) para determinar si se permite la comunicación hacia una interfaz de red o desde esta. La herramienta le indica si se permite la comunicación y qué regla de seguridad de red permite o deniega el tráfico.

## <a name="on-premises-to-azure-replication-with-nsg"></a>Replicación desde un sitio local en Azure con un grupo de seguridad de red

Azure Site Recovery permite la recuperación ante desastres y la migración a Azure para las [máquinas virtuales de Hyper-V](hyper-v-azure-architecture.md), [máquinas virtuales de VMware](vmware-azure-architecture.md) y [servidores físicos](physical-azure-architecture.md) locales. Para todos los escenarios de replicación de sitios locales a Azure, los datos de replicación se envían a una cuenta de Azure Storage y se almacenan en ella. Durante la replicación, no paga ningún cargo de las máquinas virtuales. Cuando se ejecuta una conmutación por error en Azure, Site Recovery crea automáticamente máquinas virtuales IaaS de Azure.

Una vez que se han creado las máquinas virtuales después de la conmutación por error a Azure, pueden usarse grupos de seguridad de red para limitar el tráfico a la red virtual y las máquinas virtuales. Site Recovery no crea grupos de seguridad de red como parte de la operación de conmutación por error. Se recomienda crear los grupos de seguridad de red de Azure necesarios antes de iniciar la conmutación por error. Se pueden asociar grupos de seguridad de red para incluir VM automáticamente en la conmutación por error mediante scripts de automatización con los eficaces [planes de recuperación](site-recovery-create-recovery-plans.md) de Site Recovery.

Por ejemplo, si la configuración de la máquina virtual tras la conmutación por error es similar al [escenario de ejemplo](concepts-network-security-group-with-site-recovery.md#using-network-security-groups) descrito anteriormente:
-    Puede crear la **red virtual Contoso** y la **subred Contoso** como parte del planeamiento de la recuperación ante desastres en la región de Azure de destino.
-    También puede crear y configurar el **grupo de seguridad de red de la subred** y el **grupo de seguridad de red de la máquina virtual** como parte del mismo planeamiento.
-    Después, el **grupo de seguridad de red de la subred** se puede asociar inmediatamente a la **subred Contoso**, ya que están disponibles tanto el grupo de seguridad de red como la subred.
-    El **grupo de seguridad de red de la máquina virtual** se puede asociar con máquinas virtuales durante la conmutación por error mediante planes de recuperación.

Una vez creados y configurados los grupos de seguridad de red, se recomienda ejecutar una [conmutación por error de prueba](site-recovery-test-failover-to-azure.md) para verificar las asociaciones del grupo de seguridad de red generado por script y la conectividad de la máquina virtual tras la conmutación por error.

## <a name="azure-to-azure-replication-with-nsg"></a>Replicación de Azure a Azure con un grupo de seguridad de red

Azure Site Recovery permite la recuperación ante desastres de las [máquinas virtuales de Azure](azure-to-azure-architecture.md). Al habilitar la replicación de máquinas virtuales de Azure, Site Recovery puede crear las redes virtuales de réplica (subredes y subredes de puerta de enlace incluidas) en la región de destino y las asignaciones necesarias entre las redes virtuales de origen y destino. También puede crear previamente las redes y subredes en el lado de destino y usarlas al habilitar la replicación. Site Recovery no crea máquinas virtuales en la región de Azure de destino antes de la [conmutación por error](azure-to-azure-tutorial-failover-failback.md).

Para la replicación de máquinas virtuales de Azure, asegúrese de que las reglas de grupo de seguridad de red de la región de Azure de origen permiten la [conectividad saliente](azure-to-azure-about-networking.md#outbound-connectivity-using-service-tags) para el tráfico de replicación. También puede probar y verificar estas reglas necesarias mediante este [ejemplo de configuración de grupo de seguridad de red](azure-to-azure-about-networking.md#example-nsg-configuration).

Site Recovery no crea ni replica grupos de seguridad de red como parte de la operación de conmutación por error. Se recomienda crear los grupos de seguridad de red de Azure necesarios en la región de Azure de destino antes de iniciar la conmutación por error. Se pueden asociar grupos de seguridad de red para incluir VM automáticamente en la conmutación por error mediante scripts de automatización con los eficaces [planes de recuperación](site-recovery-create-recovery-plans.md) de Site Recovery.

Teniendo en cuenta el [escenario de ejemplo](concepts-network-security-group-with-site-recovery.md#using-network-security-groups) descrito anteriormente:
-    Site Recovery puede crear réplicas de la **red virtual Contoso** y la **subred Contoso** en la región de Azure de destino cuando la replicación está habilitada para la máquina virtual.
-    Puede crear las réplicas deseadas del **grupo de seguridad de red de la subred** y del **grupo de seguridad de red de la máquina virtual** (denominados, por ejemplo, **grupo de seguridad de red de la subred de destino** y **grupo de seguridad de red de la máquina virtual de destino**, respectivamente) en la región de Azure de destino, lo cual permite cualquier regla adicional necesaria en la región de destino.
-    Después, el **grupo de seguridad de red de la subred de destino** se puede asociar inmediatamente a la subred de la región de destino, ya que están disponibles tanto el grupo de seguridad de red como la subred.
-    El **grupo de seguridad de red de la máquina virtual de destino** se puede asociar con máquinas virtuales durante la conmutación por error mediante planes de recuperación.

Una vez creados y configurados los grupos de seguridad de red, se recomienda ejecutar una [conmutación por error de prueba](azure-to-azure-tutorial-dr-drill.md) para verificar las asociaciones del grupo de seguridad de red generado por script y la conectividad de la máquina virtual tras la conmutación por error.

## <a name="next-steps"></a>Pasos siguientes
-    Más información sobre los [grupos de seguridad de red](../virtual-network/security-overview.md#network-security-groups).
-    Más información acerca de las [reglas de seguridad](../virtual-network/security-overview.md#security-rules) de los grupos de seguridad de red.
-    Más información sobre las [reglas de seguridad eficaces](../virtual-network/diagnose-network-traffic-filter-problem.md) para los grupos de seguridad de red.
-    Más información sobre los [planes de recuperación](site-recovery-create-recovery-plans.md) para automatizar la conmutación por error de las aplicaciones.
