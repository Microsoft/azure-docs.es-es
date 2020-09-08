---
title: Preguntas frecuentes sobre Azure Backup Server y DPM
description: En este artículo, encontrará respuestas a preguntas habituales sobre Microsoft Azure Backup Server (MABS) y DPM (Data Protection Manager).
ms.reviewer: srinathv
ms.topic: conceptual
ms.date: 07/05/2019
ms.openlocfilehash: 00eb064aaf549ee0d83d0cf7d9c9b47c2dc5ef78
ms.sourcegitcommit: ac7ae29773faaa6b1f7836868565517cd48561b2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88827212"
---
# <a name="azure-backup-server-and-dpm---faq"></a>Preguntas frecuentes sobre Azure Backup Server y DPM

## <a name="general-questions"></a>Preguntas generales

Este artículo contiene respuestas a algunas preguntas frecuentes sobre Azure Backup Server y DPM.

### <a name="can-i-use-azure-backup-server-to-create-a-bare-metal-recovery-bmr-backup-for-a-physical-server"></a>¿Puedo usar Azure Backup Server para crear una copia de seguridad de reconstrucción completa (BMR) para un servidor físico?

Sí.

### <a name="can-i-register-the-server-to-multiple-vaults"></a>¿Puedo registrar el servidor en varios almacenes?

No. Un servidor DPM o de Azure Backup solo se puede registrar en un único almacén.

### <a name="can-i-use-dpm-to-back-up-apps-in-azure-stack"></a>¿Puedo usar DPM para realizar copias de seguridad de aplicaciones en Azure Stack?

No. Puede usar Azure Backup para proteger Azure Stack; Azure Backup no admite el uso de DPM para realizar copias de seguridad de aplicaciones en Azure Stack.

### <a name="if-ive-installed-azure-backup-agent-to-protect-my-files-and-folders-can-i-install-system-center-dpm-to-back-up-on-premises-workloads-to-azure"></a>Si he instalado el agente de Azure Backup para proteger archivos y carpetas, ¿puedo instalar System Center DPM para realizar copias de seguridad de las cargas de trabajo locales en Azure?

Sí. Pero debe configurar DPM primero y después instalar el agente de Azure Backup.  Instalar los componentes en este orden garantiza que el agente de Azure Backup funcione con DPM. No se recomienda o no se admite la instalación del agente antes que DPM.

### <a name="why-cant-i-add-an-external-dpm-server-after-installing-ur7-and-latest-azure-backup-agent"></a>¿Por qué no puedo agregar un servidor DPM externo después de instalar el paquete acumulativo de actualizaciones 7 y el agente de Azure Backup más reciente?

En el caso de los servidores DPM con orígenes de datos protegidos en la nube (con un paquete acumulativo anterior al Paquete acumulativo de actualizaciones 7), debe esperar al menos un día después de instalar UR7 y el agente de Azure Backup más reciente para empezar a **agregar el servidor DPM externo**. Este período de un día es necesario para cargar los metadatos de los grupos de protección de DPM en Azure. Los metadatos del grupo de protección se cargan la primera vez a través de un trabajo nocturno.

### <a name="are-there-recommendations-for-configuring-exclusions-for-antivirus-software"></a>¿Hay recomendaciones para configurar exclusiones para el software antivirus?

Sí, se recomienda configurar la exclusión del antivirus. Para las exclusiones de DPM, consulte [Ejecución de software antivirus en el servidor DPM](https://docs.microsoft.com/system-center/dpm/run-antivirus-server?view=sc-dpm-2019). Para las exclusiones de MABS, consulte [Configuración del antivirus para el servidor MABS](backup-azure-mabs-troubleshoot.md#configure-antivirus-for-mabs-server).

## <a name="vmware-and-hyper-v-backup"></a>Copia de seguridad de Hyper-V y VMware

### <a name="can-i-back-up-vmware-vcenter-servers-to-azure"></a>¿Puedo hacer copias de seguridad de servidores de VMware vCenter en Azure?

Sí. Se puede usar Azure Backup Server para realizar una copia de seguridad de los hosts de EsXi y VMware vCenter Server en Azure.

- [Obtenga más información](backup-mabs-protection-matrix.md) sobre las versiones compatibles.
- [Siga estos pasos](backup-azure-backup-server-vmware.md) para realizar una copia de seguridad de un servidor de VMware.

### <a name="do-i-need-a-separate-license-to-recover-a-full-on-premises-vmwarehyper-v-cluster"></a>¿Necesito una licencia diferente para recuperar un clúster de Hyper-V o VMware local completo?

No necesita licencias independientes para la protección de Hyper-V o VMware.

- Si es cliente de System Center, utilice System Center Data Protection Manager (DPM) para proteger las máquinas virtuales de VMware.
- Si no es un cliente de System Center, puede usar Azure Backup Server (pago por uso) para proteger las máquinas virtuales de VMware.

## <a name="sharepoint"></a>SharePoint

### <a name="can-i-recover-a-sharepoint-item-to-the-original-location-if-sharepoint-is-configured-by-using-sql-alwayson-with-protection-on-disk"></a>¿Puedo recuperar un elemento de SharePoint en la ubicación original si SharePoint está configurado con SQL AlwaysOn (con protección en disco)?

Sí, se puede recuperar el elemento en el sitio de SharePoint original.

### <a name="can-i-recover-a-sharepoint-database-to-the-original-location-if-sharepoint-is-configured-by-using-sql-alwayson"></a>¿Puedo recuperar una base de datos de SharePoint en la ubicación original si SharePoint está configurada con SQL AlwaysOn?

Dado que las bases de datos de SharePoint están configuradas en SQL AlwaysOn, no se pueden modificar a menos que se quite el grupo de disponibilidad. En consecuencia, DPM no puede restaurar la base de datos en la ubicación original. Puede recuperar una base de datos SQL Server en otra instancia de SQL Server.

## <a name="next-steps"></a>Pasos siguientes

Lea las otras preguntas más frecuentes:

- [Más información](backup-support-matrix-mabs-dpm.md) sobre la matriz de compatibilidad de Azure Backup Server y DPM.
- [Más información](backup-azure-mabs-troubleshoot.md) sobre las guías de solución de problemas de Azure Backup Server y DPM.
