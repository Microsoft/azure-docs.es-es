---
title: Actualizaciones de servicio y actualizaciones de componentes en Azure Site Recovery
description: Se proporciona información general sobre las actualizaciones de servicio y de componentes en Azure Site Recovery.
author: Rajeswari-Mamilla
manager: rochakm
ms.service: site-recovery
ms.topic: conceptual
ms.date: 07/31/2019
ms.author: ramamill
ms.openlocfilehash: a1ea8b6fb9800d796670161288be0d86ce6ffc42
ms.sourcegitcommit: ac5cbef0706d9910a76e4c0841fdac3ef8ed2e82
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/03/2020
ms.locfileid: "89424946"
---
# <a name="service-updates-in-site-recovery"></a>Actualizaciones de servicio en Azure Site Recovery

En este artículo se proporciona información general sobre las actualizaciones de [Azure Site Recovery](site-recovery-overview.md) y se describe cómo actualizar los componentes de Site Recovery.

Site Recovery publica actualizaciones de servicio de forma periódica. Las actualizaciones incluyen nuevas características, mejoras de soporte técnico, actualizaciones de componentes y correcciones de errores. Con el fin de aprovechar las últimas características y correcciones, se recomienda ejecutar las versiones más recientes de los componentes de Site Recovery. 
 
 
## <a name="updates-support"></a>Soporte técnico de actualizaciones

### <a name="support-statement-for-azure-site-recovery"></a>Declaración de soporte técnico de Azure Site Recovery

Se recomienda actualizar siempre a las versiones más recientes de los componentes:

**Con cada nueva versión "N" de un componente de Azure Site Recovery que se publica, todas las versiones anteriores a "N-4" se consideran excluidas del soporte técnico.** 

> [!IMPORTANT]
> El soporte técnico oficial es para la actualización de la versión > N-4 a la versión N. Por ejemplo, si tiene la versión N-6, deberá actualizar primero a N-4 y, después, a N.


### <a name="links-to-currently-supported-update-rollups"></a>Vínculos a paquetes acumulativos de actualizaciones admitidos actualmente

 Consulte el paquete acumulativo de actualizaciones más reciente (versión N) en [este artículo](site-recovery-whats-new.md). Recuerde que Site Recovery proporciona soporte técnico para las versiones N-4.



## <a name="component-expiry"></a>Expiración de componentes

Site Recovery le indica qué componentes están a punto de expirar o ya han expirado por correo electrónico (si está suscrito a las notificaciones por correo electrónico) o en el panel del almacén del portal.

- Además, cuando haya actualizaciones disponibles, en la vista de infraestructura del escenario del portal, aparecerá el botón **Actualización disponible** junto al componente. Este botón le redirige a un vínculo para descargar la versión más reciente del componente.
-  Las notificaciones del panel del almacén no están disponibles si está replicando máquinas virtuales de Hyper-V. 

Las notificaciones de correo electrónico se envían como se indica a continuación.

**Time** | **Frecuencia**
--- | ---
60 días antes de que expire el componente | Una vez cada dos semanas
Siguientes 53 días | Una vez a la semana
Últimos 7 días | Una vez al día
Después de la expiración | Una vez cada dos semanas


### <a name="upgrading-outside-official-support"></a>Actualización fuera del soporte técnico oficial

Si la diferencia entre su versión del componente y la versión más reciente es superior a cuatro versiones, se considera que queda fuera del soporte técnico. En este caso, actualice como se indica a continuación: 

1. Actualice el componente instalado actualmente a su versión actual más cuatro. Por ejemplo, si su versión es la 9.16, actualice a la 9.20.
2. A continuación, actualice a la siguiente versión compatible. Así, en nuestro ejemplo, después de actualizar de la 9.16 a la 9.20, actualizará a la 9.24. 

Siga el mismo proceso para todos los componentes pertinentes.

### <a name="support-for-latest-operating-systemskernels"></a>Soporte técnico para sistemas operativos/kernels más recientes

> [!NOTE]
> Si tiene programado un mantenimiento y, como parte de él, el equipo se va a reiniciar, se recomienda que primero actualice los componentes de Site Recovery y, a continuación, continúe con el resto de las actividades programadas en el mantenimiento.

1. Antes de actualizar las versiones del kernel o el sistema operativo, compruebe si la versión de destino es compatible con Site Recovery. 

    - Compatibilidad con [máquina virtual de Azure](azure-to-azure-support-matrix.md#replicated-machine-operating-systems).
    - Compatibilidad con [servidores físicos o de VMware](vmware-physical-azure-support-matrix.md#replicated-machines).
    - Compatibilidad con [Hyper-V](hyper-v-azure-support-matrix.md#replicated-vms).
2. Revise las [actualizaciones disponibles](site-recovery-whats-new.md) para averiguar qué desea actualizar.
3. Actualice a la versión más reciente de Site Recovery.
4. Actualice el sistema operativo o el kernel a las versiones necesarias.
5. Reinicie.


Este proceso garantiza que el sistema operativo o el kernel de la máquina se actualicen a la versión más reciente y que se carguen en la máquina los cambios de Site Recovery más recientes necesarios para admitir la nueva versión.

## <a name="azure-vm-disaster-recovery-to-azure"></a>Recuperación ante desastres de máquinas virtuales de Azure en Azure

En este escenario, es muy recomendable que [habilite las actualizaciones automáticas](azure-to-azure-autoupdate.md). Puede permitir que Site Recovery administre las actualizaciones de la siguiente manera:

- Al habilitar el proceso de replicación.
- Definiendo la configuración de actualización de la extensión dentro del almacén.

Si desea administrar las actualizaciones manualmente, puede elegir una de las siguientes opciones:

1. Cuando hay disponible una nueva actualización del agente, Site Recovery proporciona una notificación en el almacén hacia la parte superior de la página. En el almacén > **Elementos replicados**, haga clic en esta notificación en la parte superior de la pantalla: 
    
    **Hay una nueva actualización del agente de replicación de Site Recovery disponible. Haga clic para instalarla ->** <br/><br/>Seleccione las máquinas virtuales a las que desea aplicar la actualización y haga clic en **Aceptar**.

2. En la página de información general de recuperación ante desastres de la máquina virtual, encontrará el campo "Estado del agente", que indicará "Actualización crítica" si el agente va a expirar. Haga clic en él y siga las instrucciones posteriores para actualizar manualmente la máquina virtual.

## <a name="vmware-vmphysical-server-disaster-recovery-to-azure"></a>Recuperación ante desastres de un servidor físico/VM de VMware en Azure

1. En función de la versión actual y de conformidad con la [declaración de soporte técnico](#support-statement-for-azure-site-recovery), instale primero la actualización en el servidor de configuración local con [estas instrucciones](vmware-azure-deploy-configuration-server.md#upgrade-the-configuration-server). 
2. Si tiene servidores de procesos de escalado horizontal, actualícelos a continuación con [estas instrucciones](vmware-azure-manage-process-server.md#upgrade-a-process-server).
3. Para actualizar el agente de movilidad en cada máquina protegida, consulte [este](vmware-physical-manage-mobility-service.md#update-mobility-service-from-azure-portal) artículo.

### <a name="reboot-after-mobility-service-upgrade"></a>Reinicio después de la actualización de Mobility Service

Se recomienda reiniciar después de cada actualización de Mobility Service para asegurarse de que se cargan todos los cambios más recientes en la máquina de origen.

No es obligatorio reiniciar, a menos que durante el último reinicio la diferencia entre la versión del agente y la versión actual sea mayor que 4.

En el ejemplo de la tabla se muestra cómo funciona esto.

|**Versión del agente (último reinicio)** | **Actualizar a** | **¿Reinicio obligatorio?**|
|---------|---------|---------|
|9.16 |  9.18 | No es obligatorio|
|9.16 | 9.19 | No es obligatorio|
| 9.16 | 9.20 | No es obligatorio
 | 9.16 | 9.21 | Mandatory.<br/><br/> Actualice a la versión 9.20 y, a continuación, reinicie antes de actualizar a la 9.21.

## <a name="hyper-v-vm-disaster-recovery-to-azure"></a>Recuperación ante desastres de máquinas virtuales de Hyper-V en Azure

### <a name="between-a-hyper-v-site-and-azure"></a>Entre un sitio Hyper-V y Azure

1. Descargue la actualización del proveedor de Microsoft Azure Site Recovery.
2. Instale el proveedor en cada servidor de Hyper-V registrado en Site Recovery. Si está ejecutando un clúster, actualice en todos los nodos del clúster.


## <a name="between-an-on-premises-vmm-site-and-azure"></a>Entre un sitio VMM local y Azure
1. Descargue la actualización del proveedor de Microsoft Azure Site Recovery.
2. Instale el proveedor en el servidor VMM. Si VMM está implementado en un clúster, instale el proveedor en todos los nodos del clúster.
3. Instale el agente de Microsoft Azure Recovery Services más reciente en cada host de Hyper-V o nodo del clúster.


## <a name="between-two-on-premises-vmm-sites"></a>Entre dos sitios VMM locales
1. Descargue la actualización más reciente del proveedor de Microsoft Azure Site Recovery.
2. Instale el proveedor más reciente en el servidor VMM que administra el sitio de recuperación secundario. Si VMM está implementado en un clúster, instale el proveedor en todos los nodos del clúster.
3. Una vez actualizado el sitio de recuperación, instale el proveedor en el servidor VMM que administra el sitio principal.

## <a name="next-steps"></a>Pasos siguientes

Consulte nuestra página [Actualizaciones de Azure](https://azure.microsoft.com/updates/?product=site-recovery) para conocer las nuevas actualizaciones y versiones.
