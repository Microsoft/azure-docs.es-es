---
title: Sobre la configuración, proceso y servidores de destino maestros de Azure Site Recovery | Microsoft Docs
description: Este artículo proporciona información general de la configuración, el proceso y los servidores de destino maestro mediante al configurar la recuperación ante desastres de máquinas virtuales de VMware locales en Azure con Azure Site Recovery
author: rayne-wiselman
ms.service: site-recovery
services: site-recovery
ms.topic: conceptual
ms.date: 04/28/2019
ms.author: raynew
ms.openlocfilehash: 6f501e251f0b006bbbb4f64090cac5c3d61b7bf2
ms.sourcegitcommit: bb85a238f7dbe1ef2b1acf1b6d368d2abdc89f10
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/10/2019
ms.locfileid: "65523546"
---
# <a name="about-site-recovery-components-configuration-process-master-target"></a>Acerca de los componentes de Site Recovery (configuración, proceso de destino maestro)

Este artículo describe la configuración, proceso y los servidores de destino maestro usados al replicar máquinas virtuales de VMware y servidores físicos en Azure con el [Site Recovery](site-recovery-overview.md) service.

## <a name="configuration-server"></a>Servidor de configuración

Recuperación ante desastres de máquinas virtuales de VMware locales y servidores físicos, necesita una recuperación del sitio servidor de configuración implementado en el entorno local.

**Configuración** | **Detalles** | **Links**
--- | --- | ---
**Componentes**  | La máquina del servidor de configuración ejecuta todos los componentes de Site Recovery de forma local, que incluyen el servidor de configuración, el servidor de procesos y el servidor de destino maestro.<br/><br/> Al configurar el servidor de configuración, todos los componentes se instalan automáticamente. | [Lectura](vmware-azure-common-questions.md#configuration-server) el servidor de configuración de preguntas más frecuentes.
**Rol** | El servidor de configuración coordina la comunicación entre el entorno local y Azure, además de administrar la replicación de datos. | Más información sobre la arquitectura de [VMware](vmware-azure-architecture.md) y [servidor físico](physical-azure-architecture.md) recuperación ante desastres en Azure.
**Requisitos de VMware** | Recuperación ante desastres de máquinas virtuales de VMware locales, debe instalar y ejecutar el servidor de configuración como una alta disponibilidad de VM de VMware en local. | [Obtenga información sobre](vmware-azure-deploy-configuration-server.md#prerequisites) los requisitos previos.
**Implementación de VMware** | Le recomendamos que implemente el servidor de configuración mediante una plantilla descargada de OVA. Este método proporciona un simplemente para configurar un servidor de configuración que cumpla con todos los requisitos y requisitos previos.<br/><br/> Si por algún motivo no puede implementar una VM de VMware mediante una plantilla de OVA, puede configurar las máquinas de servidor de configuración manualmente, tal como se describe a continuación para la recuperación ante desastres de máquina física. | [Implementar](vmware-azure-deploy-configuration-server.md#deployment-of-configuration-server-through-ova-template) con una plantilla de OVA.
**Requisitos del servidor físico** | Recuperación ante desastres en servidores físicos locales, implementar manualmente el servidor de configuración. | [Obtenga información sobre](physical-azure-set-up-source.md#prerequisites) los requisitos previos.
**Implementación de servidor físico** | Si no se puede instalar como una VM de VMware, se puede instalar en un servidor físico. | [Implementar](physical-azure-set-up-source.md#set-up-the-source-environment) el servidor de configuración manualmente.


## <a name="process-server"></a>Servidor de procesos

**Configuración** | **Detalles** | **Links**
--- | --- | ---
**Implementación**  | Para la recuperación ante desastres y replicación de máquinas virtuales de VMware locales y servidores físicos, necesitará un servidor de proceso local. De forma predeterminada, el servidor de procesos está instalado en el servidor de configuración cuando se implementa. | [Más información](vmware-azure-architecture.md?#architectural-components).
**Rol (de local** | -Recibe datos de replicación de las máquinas habilitadas para replicación.<br/> -Optimiza los datos de replicación con almacenamiento en caché, compresión y cifrado y lo envía al almacenamiento de Azure.<br/> -Realiza una instalación de inserción de Mobility Service de Site Recovery en máquinas virtuales de VMware locales y servidores físicos que desea replicar.<br/> -Realiza la detección automática de máquinas locales. | [Más información](vmware-physical-azure-config-process-server-overview.md#process-server). 
**Rol (la conmutación por recuperación desde Azure)** | Después de la conmutación por error desde su sitio local, configurar un servidor de procesos en Azure, como una máquina virtual de Azure, para controlar la conmutación por recuperación a su ubicación local.<br/><br/> El servidor de procesos en Azure es temporal. Después de realiza la conmutación por recuperación, se puede eliminar la máquina virtual de Azure. | [Más información](vmware-azure-set-up-process-server-azure.md).
**Escalado** | Para implementaciones más grandes, en el entorno local puede configurar servidores de procesos adicional, la escalabilidad horizontal. Servidores adicionales escalan horizontalmente la capacidad, al controlar un mayor número de la replicación de máquinas y mayores volúmenes de tráfico de replicación.<br/><br/> Puede mover máquinas entre dos servidores de procesos, con el fin de equilibrar el tráfico de replicación de carga. | [Obtenga más información](vmware-azure-set-up-process-server-scale.md),


## <a name="master-target-server"></a>Servidor de destino principal

El servidor de destino maestro controla los datos de replicación durante la conmutación por recuperación desde Azure.

- Se instala de forma predeterminada en el servidor de configuración.
- En el caso de las implementaciones de gran tamaño, puede agregar un servidor de destino maestro independiente adicional para la conmutación por recuperación.


## <a name="next-steps"></a>Pasos siguientes
- Revise el [arquitectura](vmware-azure-architecture.md) para la recuperación ante desastres de máquinas virtuales de VMware y servidores físicos.
- Revise el [requisitos y requisitos previos](vmware-physical-azure-support-matrix.md) para la recuperación ante desastres de máquinas virtuales de VMware y servidores físicos en Azure. 
