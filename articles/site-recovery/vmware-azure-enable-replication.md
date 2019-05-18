---
title: Habilitar la replicación de máquinas virtuales de VMware para la recuperación ante desastres en Azure con Azure Site Recovery | Microsoft Docs
description: En este artículo se describe cómo habilitar máquinas virtuales de VMware para la replicación en Azure para la recuperación ante desastres con Azure Site Recovery.
author: Rajeswari-Mamilla
ms.service: site-recovery
ms.date: 05/10/2019
ms.topic: conceptual
ms.author: ramamill
ms.openlocfilehash: add0f8252bdae6857b28deeb7de4c1d09973e452
ms.sourcegitcommit: f6c85922b9e70bb83879e52c2aec6307c99a0cac
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2019
ms.locfileid: "65540762"
---
# <a name="enable-replication-to-azure-for-vmware-vms"></a>Habilitación de máquinas virtuales de VMware en Azure

En este artículo se describe cómo habilitar la replicación de máquinas virtuales locales de VMware en Azure.

## <a name="prerequisites"></a>Requisitos previos

En este artículo se supone que ha:

- [Configurar el entorno de origen local](vmware-azure-set-up-source.md).
- [Configurar el entorno de destino en Azure](vmware-azure-set-up-target.md).

## <a name="before-you-start"></a>Antes de comenzar
Al replicar máquinas virtuales de VMware, tenga en cuenta esta información:

* Su cuenta de usuario de Azure debe tener ciertos [permisos](site-recovery-role-based-linked-access-control.md#permissions-required-to-enable-replication-for-new-virtual-machines) para habilitar la replicación de una nueva máquina virtual en Azure.
* Las máquinas virtuales de VMware se detectan cada 15 minutos. Se pueden tardar 15 minutos o más máquinas virtuales que aparezcan en el portal de Azure después de la detección. Del mismo modo, la detección puede tardar 15 minutos o más cuando se agrega un nuevo host de vCenter server o vSphere.
* Se pueden tardar 15 minutos o más cambios en el entorno en la máquina virtual (por ejemplo, la instalación de herramientas de VMware) en actualizarse en el portal.
* Puede comprobar el tiempo detectados por el último para máquinas virtuales de VMware: Consulte la **último contacto a las** campo el **servidores de configuración** página para el servidor vCenter/host vSphere.
* Para agregar las máquinas virtuales para la replicación sin esperar a que la detección programada, resalte el servidor de configuración (pero no haga clic en él) y seleccione **actualizar**.
* Al habilitar la replicación, si la máquina virtual está preparada, el servidor de procesos instala automáticamente el servicio Azure Site Recovery Mobility en ella.

## <a name="enable-replication"></a>Habilitar replicación

Antes de seguir los pasos descritos en esta sección, tenga en cuenta la siguiente información:
* Azure Site Recovery replica ahora directamente a discos administrados para todas las replicaciones de nuevo. El servidor de procesos escribe los registros de replicación en una cuenta de almacenamiento de caché en la región de destino. Estos registros se utilizan para crear puntos de recuperación en discos administrados de réplica.
* En el momento de la conmutación por error, el punto de recuperación que seleccione se utiliza para crear el disco administrado de destino.
* Las máquinas virtuales que se habían configuradas previamente para replicar en cuentas de almacenamiento de destino no se ven afectadas.
* Replicación en cuentas de almacenamiento para una nueva máquina virtual solo está disponible a través de un Representational State Transfer (REST) API y Powershell. Use la API de REST de Azure versión 2016-08-10 o 2018-01-10 para la replicación en cuentas de almacenamiento.

1. Vaya a **paso 2: Replicar la aplicación** > **Origen**. Después de habilitar la replicación por primera vez, seleccione **+ replicar** en el almacén para habilitar la replicación de máquinas virtuales adicionales.
2. En la página **Origen** > **Origen**, seleccione el servidor de configuración.
3. Para **tipo de máquina**, seleccione **máquinas virtuales** o **máquinas físicas**.
4. En **vCenter/vSphere Hypervisor**, seleccione el servidor vCenter que administra el host de vSphere, o bien seleccione el host. Esta configuración no es relevante si va a replicar los equipos físicos.
5. Seleccione el servidor de procesos. Si no hay ningún servidor de procesos adicionales creados, servidor de procesos integrada del servidor de configuración estará disponible en la lista desplegable. Se indica el estado de mantenimiento de cada servidor de procesos según los límites recomendados y otros parámetros. Elija un servidor de procesos en buen estado. Un [críticos](vmware-physical-azure-monitor-process-server.md#process-server-alerts) no se puede elegir el servidor de procesos. Puede [localizar y solucionar](vmware-physical-azure-troubleshoot-process-server.md) los errores **o** configurar un [servidor de procesos de escalado horizontal](vmware-azure-set-up-process-server-scale.md).
    ![Habilitar ventana de origen de replicación](media/vmware-azure-enable-replication/ps-selection.png)

> [!NOTE]
> Desde [9.24 versiones](service-updates-how-to.md#links-to-currently-supported-update-rollups), se presentan las alertas adicionales para mejorar las alertas de estado del servidor de procesos. Actualizar componentes de Site Recovery 9.24 versiones o versiones posteriores para que todas las alertas que se genere.

6. Para **destino**, seleccione la suscripción y grupo de recursos donde desea crear máquinas virtuales conmutadas por error. Elija el modelo de implementación que desea usar en Azure para las máquinas virtuales conmutadas por error.
2. Seleccione la red de Azure y la subred que se conectarán las máquinas virtuales de Azure después de la conmutación por error. La red debe estar en la misma región que el almacén del servicio Site Recovery.

   Seleccione **configurar ahora para las máquinas seleccionadas** para aplicar la configuración de red para todas las máquinas virtuales que seleccionó para la protección. Seleccione **configurar más adelante** para seleccionar la red de Azure por máquina virtual. Si no dispone de una red, debe crear una. Para crear una red mediante el uso de Azure Resource Manager, seleccione **crear nuevo**. Seleccione una subred si es aplicable y, a continuación, seleccione **Aceptar**.
   
   ![Habilitar ventana de destino de replicación](./media/vmware-azure-enable-replication/enable-rep3.png)

1. Para **máquinas virtuales** > **seleccionar máquinas virtuales**, seleccione cada máquina virtual que desea replicar. Solo puede seleccionar las máquinas virtuales para el que se puede habilitar la replicación. Después seleccione **Aceptar**. Si no se puede ver o seleccionar cualquier máquina virtual en concreto, consulte [máquina de origen no aparece en el portal de Azure](https://aka.ms/doc-plugin-VM-not-showing) para resolver el problema.

    ![Habilitar ventana Seleccionar máquinas virtuales de replicación](./media/vmware-azure-enable-replication/enable-replication5.png)

1. Para **propiedades** > **configurar propiedades**, seleccione la cuenta que usa el servidor de procesos para instalar automáticamente el servicio Site Recovery Mobility en la máquina virtual. Además, elija el tipo de disco administrado de destino para replicar en basados en los datos de los patrones de actividad.
10. De forma predeterminada, se replican todos los discos de una máquina virtual de origen. Para excluir discos de replicación, desactive la **Include** casilla de verificación para todos los discos que no quiera replicar. Después seleccione **Aceptar**. Puede establecer propiedades adicionales más adelante. Obtenga más información sobre [excluir discos](vmware-azure-exclude-disk.md).

    ![Habilitar la ventana Propiedades de configuración de replicación](./media/vmware-azure-enable-replication/enable-replication6.png)

1. En **la configuración de replicación** > **establecer configuración de replicación**, compruebe que la directiva de replicación correcto está seleccionada. Puede modificar la configuración de directiva de replicación en **configuración** > **directivas de replicación** > ***nombre de la directiva***  >  **Editar la configuración**. Los cambios que se aplican a una directiva también se aplican a máquinas virtuales nuevas y replicadas.
1. Habilitar **coherencia de múltiples VM** si desea recopilar las máquinas virtuales en un grupo de replicación. Especifique un nombre para el grupo y, a continuación, seleccione **Aceptar**.

    > [!NOTE]
    >    * Las máquinas virtuales en un grupo de replicación se replican al mismo tiempo y comparten puntos de recuperación coherente con los bloqueos y coherentes con la aplicación cuando conmutan.
    >    * Recopile las máquinas virtuales y los servidores físicos juntos para que reflejen las cargas de trabajo. La habilitación de la coherencia entre varias VM puede afectar al rendimiento de la carga de trabajo. Hacer esto sólo si las máquinas virtuales ejecutan la misma carga de trabajo y necesita coherencia.

    ![Habilitar período de replicación](./media/vmware-azure-enable-replication/enable-replication7.png)
    
1. Seleccione **Habilitar replicación**. Puede realizar un seguimiento del progreso de la **Habilitar protección** del trabajo en **configuración** > **trabajos** > **trabajos de Site Recovery**. La máquina virtual estará preparada para la conmutación por error después de que finalice el trabajo de **Finalizar protección**.

## <a name="view-and-manage-vm-properties"></a>Visualización y administración de las propiedades de la máquina virtual

A continuación, compruebe las propiedades de la máquina virtual de origen. Recuerde que el nombre de la máquina virtual de Azure debe cumplir los [requisitos para las máquinas virtuales de Azure](vmware-physical-azure-support-matrix.md#replicated-machines).

1. Vaya a **configuración** > **elementos replicados**y, a continuación, seleccione la máquina virtual. El **Essentials** página muestra información sobre la configuración y el estado de la máquina virtual.
1. En **Propiedades** puede ver la información de replicación y conmutación por error de la máquina virtual.
1. En **proceso y red** > **propiedades de proceso**, puede cambiar varias propiedades de la máquina virtual. 

    ![Ventana de propiedades de proceso y red](./media/vmware-azure-enable-replication/vmproperties.png)

    * Nombre de máquina virtual de Azure: Modifique el nombre para satisfacer los requisitos de Azure, si es necesario.
    * Tamaño de máquina virtual de destino o tipo de máquina virtual: El tamaño de máquina virtual predeterminada se selecciona basándose en unos cuantos parámetros que incluyen el número de discos, número de tarjetas NIC, recuento de núcleos de CPU, memoria y tamaños de rol VM disponibles en la región de Azure de destino. Azure Site Recovery toma el primer tamaño de máquina virtual disponible que cumple todos los criterios. Puede seleccionar un tamaño de máquina virtual diferente en función de sus necesidades en cualquier momento antes de la conmutación por error. Tenga en cuenta que el tamaño del disco de máquina virtual también se basa en el tamaño del disco de origen, y solo puede cambiarse después de la conmutación por error. Más información sobre los tamaños de disco y las tasas de IOPS en [objetivos de escalabilidad y rendimiento para discos de máquinas virtuales en Windows](../virtual-machines/windows/disk-scalability-targets.md).

    *  Grupo de recursos: Puede seleccionar un [grupo de recursos](https://docs.microsoft.com/azure/virtual-machines/windows/infrastructure-resource-groups-guidelines), desde que una máquina virtual se convierte en una parte de una conmutación por error posterior. Puede cambiar esta configuración en cualquier momento antes de la conmutación por error. Después de la conmutación por error, si migra la máquina virtual a otro grupo de recursos, interrumpe la configuración de protección para esa máquina virtual.
    * Conjunto de disponibilidad: Puede seleccionar un [conjunto de disponibilidad](https://docs.microsoft.com/azure/virtual-machines/windows/infrastructure-availability-sets-guidelines) si la máquina virtual debe ser una parte de una conmutación por error posterior. Cuando se selecciona un conjunto de disponibilidad, tenga en cuenta la siguiente información:

        * Se muestran solo los conjuntos de disponibilidad que pertenecen al grupo de recursos especificado.  
        * Las máquinas virtuales que se encuentran en distintas redes virtuales no pueden ser una parte del mismo conjunto de disponibilidad.
        * Solo las máquinas virtuales del mismo tamaño pueden formar parte de un conjunto de disponibilidad.
1. También puede agregar información acerca de la red de destino, la subred y la dirección IP que se asigna a la máquina virtual de Azure.
2. En **Discos** puede ver los discos de datos y del sistema operativo en la máquina virtual que se va a replicar.

### <a name="configure-networks-and-ip-addresses"></a>Configuración de redes y direcciones IP

Puede establecer la dirección IP de destino. Si no proporciona una dirección, la máquina virtual conmutada por error usará DHCP. Si establece una dirección que no esté disponible en el momento de la conmutación por error, esta operación no funcionará. Si la dirección está disponible en la red de conmutación por error de prueba, puede usar la misma dirección IP de destino para la conmutación por error de prueba.

El número de adaptadores de red viene determinado por el tamaño que especifique para la máquina virtual de destino, como se indica a continuación:

- Si el número de adaptadores de red en la máquina virtual de origen es menor o igual que el número de adaptadores que se permiten para el tamaño de la máquina virtual de destino, el destino tiene el mismo número de adaptadores que el origen.
- Si el número de adaptadores para la máquina virtual de origen supera el número permitido para el tamaño de la máquina virtual de destino, se utiliza el tamaño máximo de destino. Por ejemplo, si una máquina virtual de origen tiene dos adaptadores de red y el tamaño de la máquina virtual de destino admite cuatro, la máquina virtual de destino tiene dos adaptadores. Si la máquina virtual de origen tiene dos adaptadores pero el tamaño de destino solo admite uno, la máquina virtual de destino tiene solo un adaptador.
- Si la máquina virtual tiene varios adaptadores de red, todos ellos se conectan a la misma red. Además, se convierte en el primer adaptador que se muestra en la lista el *predeterminada* adaptador de red en la máquina virtual de Azure. 

### <a name="azure-hybrid-benefit"></a>Ventaja híbrida de Azure

Los clientes de Microsoft Software Assurance pueden usar la ventaja híbrida de Azure para ahorrar en costos de licencia para los equipos de Windows Server que se migran a Azure. La ventaja también se aplica a la recuperación ante desastres de Azure. Si está autorizado, puede asignar la ventaja a la máquina virtual que Site Recovery crea si se produce una conmutación por error. Para ello, sigue estos pasos:
1. Vaya a la **propiedades de red y equipo** de la máquina virtual replicada.
2. Responder cuando se le pregunte si tiene una licencia de Windows Server que le permite ser apto para la ventaja híbrida de Azure.
3. Confirme que tiene una licencia de Windows Server con Software Assurance que puede usar para aplicar la ventaja a la máquina virtual que se crearán en la conmutación por error.
4. Guarde la configuración de la máquina virtual replicada.

Más información sobre la [Ventaja híbrida de Azure](https://aka.ms/azure-hybrid-benefit-pricing).

## <a name="resolve-common-issues"></a>Resolver problemas comunes

* Cada disco debe ser inferior a 4 TB.
* El disco del sistema operativo debe ser un disco básico, no un disco dinámico.
* 2/compatible con UEFI máquinas virtuales de generación, la familia de sistema operativo debe ser Windows y el disco de arranque debe ser menor que 300 GB.

## <a name="next-steps"></a>Pasos siguientes

Una vez que la máquina virtual alcanza un estado protegido, pruebe una [conmutación por error](site-recovery-failover.md) para comprobar si la aplicación aparece en Azure.

* Consulte cómo [borrar el registro y la configuración de la protección](site-recovery-manage-registration-and-protection.md) para deshabilitar la replicación.
* Obtenga información sobre cómo [automatizar la replicación para las máquinas virtuales mediante Powershell](vmware-azure-disaster-recovery-powershell.md).
