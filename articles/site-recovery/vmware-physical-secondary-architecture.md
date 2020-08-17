---
title: Recuperación ante desastres de servidores físicos o VMware basada en la arquitectura en un sitio secundario con Azure Site Recovery
description: En este artículo se proporciona información general sobre los componentes y la arquitectura usados durante la recuperación ante desastres de máquinas virtuales de VMware locales o servidores físicos de Windows o Linux a en un sitio secundario de VMware con Azure Site Recovery.
author: rayne-wiselman
manager: carmonm
ms.service: site-recovery
ms.topic: conceptual
ms.date: 11/12/2019
ms.author: raynew
ms.openlocfilehash: 7c884ce839523706e67e4278f43e237e1a2b0580
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87496974"
---
# <a name="architecture-for-vmwarephysical-server-replication-to-a-secondary-on-premises-site"></a>Arquitectura para la replicación de VMware y servidores físicos en un sitio local secundario

En este artículo se describe la arquitectura y los procesos usados al configurar la replicación, la conmutación por error y la recuperación de máquinas virtuales (VM) de VMware o de servidores físicos de Windows o Linux en un sitio secundario de VMware durante una recuperación ante desastres, mediante [Azure Site Recovery](site-recovery-overview.md).


## <a name="architectural-components"></a>Componentes de la arquitectura

**Ámbito** | **Componente** | **Detalles**
--- | --- | ---
**Azure** | Este escenario se implementa con InMage Scout. | Para obtener InMage Scout, necesitará una suscripción a Azure.<br/><br/> Después de crear un almacén de Recovery Services descargue InMage Scout e instale las actualizaciones más recientes para configurar la implementación.
**Servidor de proceso** | Ubicado en el sitio principal. | Implemente el servidor de procesos para controlar el almacenamiento en caché, la compresión y la optimización de datos.<br/><br/> También controla la instalación por inserción del agente unificada en las máquinas que desea proteger.
**Servidor de configuración** | Ubicado en el sitio secundario. | El servidor de configuración administra, configura y supervisa la implementación, ya sea mediante el sitio web de administración o la consola de vContinuum.
**Servidor de vContinuum** | Opcional. Se instala en la misma ubicación que el servidor de configuración. | Proporciona una consola para administrar y supervisar su entorno protegido.
**Servidor de destino principal** | Ubicado en el sitio secundario. | El servidor de destino maestro contiene los datos replicados. Recibe los datos del servidor de proceso, crea una máquina de réplica en el sitio secundario y contiene los puntos de retención de datos.<br/><br/> El número de servidores de destino maestros que necesita depende del número de equipos que va a proteger.<br/><br/> Si desea realizar una conmutación por recuperación al sitio principal, necesita un servidor de destino maestro allí también. El agente unificado se instala en este servidor.
**Servidor de VMware ESX/ESXi y vCenter** |  Las máquinas virtuales se hospedan en hosts ESX/ESXi. Los hosts se administran con un servidor vCenter. | Necesita una infraestructura de VMware para replicar máquinas virtuales de VMware.
**Máquinas virtuales y servidores físicos** |  Unified Agent se instala en las máquinas virtuales de VMware y los servidores físicos que desea replicar. | Dicho agente actúa como un proveedor de comunicación entre todos los componentes.

## <a name="set-up-outbound-network-connectivity"></a>Configuración de la conectividad de red saliente

Para que Site Recovery funcione de la forma esperada, debe modificar la conectividad de red saliente para permitir la replicación de su entorno.

> [!NOTE]
> Site Recovery no admite el uso de un proxy de autenticación para controlar la conectividad de la red.

### <a name="outbound-connectivity-for-urls"></a>Conectividad de salida para las direcciones URL

Si usa un proxy de firewall basado en direcciones URL para controlar la conectividad de salida, debe permitir el acceso a ellas:

| **Nombre**                  | **Comercial**                               | **Gobierno**                                 | **Descripción** |
| ------------------------- | -------------------------------------------- | ---------------------------------------------- | ----------- |
| Storage                   | `*.blob.core.windows.net`                  | `*.blob.core.usgovcloudapi.net`              | Permite que los datos se puedan escribir desde la máquina virtual a la cuenta de almacenamiento de caché en la región de origen. |
| Azure Active Directory    | `login.microsoftonline.com`                | `login.microsoftonline.us`                   | Proporciona autorización y autenticación de las direcciones URL del servicio Site Recovery. |
| Replicación               | `*.hypervrecoverymanager.windowsazure.com` | `*.hypervrecoverymanager.windowsazure.com`   | Permite que la máquina virtual se comunique con el servicio Site Recovery. |
| Azure Service Bus               | `*.servicebus.windows.net`                 | `*.servicebus.usgovcloudapi.net`             | Permite que la máquina virtual escriba los datos de diagnóstico y supervisión de Site Recovery. |

## <a name="replication-process"></a>Proceso de replicación

1. Configure los servidores de componentes de cada sitio (configuración, proceso y destino principal) e instale Unified Agent en las máquinas que desee replicar.
2. Después de la replicación inicial, el agente de cada máquina envía los cambios de replicación diferencial al servidor de procesos.
3. Por su parte, el servidor de procesos optimiza los datos y los transfiere al servidor de destino maestro del sitio secundario. El servidor de configuración administra el proceso de replicación.

![Diagrama que muestra la replicación de máquinas virtuales de VMware y de servidores físicos en un centro de datos secundario](./media/site-recovery-components/vmware-to-vmware.png)



## <a name="next-steps"></a>Pasos siguientes

[Configuración](vmware-physical-secondary-disaster-recovery.md) de la recuperación ante desastres de máquinas virtuales de VMware y servidores físicos en un sitio secundario.
