---
title: Novedades de Microsoft Azure Backup Server
description: Microsoft Azure Backup Server proporciona funcionalidades mejoradas de copia de seguridad para proteger máquinas virtuales, archivos y carpetas, cargas de trabajo, etc.
ms.topic: conceptual
ms.date: 05/24/2020
ms.openlocfilehash: ca06e52c81b909bb19008fdccfe4b3fdeb7e6248
ms.sourcegitcommit: 3fb5e772f8f4068cc6d91d9cde253065a7f265d6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/31/2020
ms.locfileid: "89182146"
---
# <a name="whats-new-in-microsoft-azure-backup-server-mabs"></a>Novedades de Microsoft Azure Backup Server (MABS)

## <a name="whats-new-in-mabs-v3-ur1"></a>Novedades de MABS V3 UR1

La versión 3 de Microsoft Azure Backup Server (MABS) UR1 es la actualización más reciente e incluye correcciones de errores críticos y otras características y mejoras. Para ver la lista de errores corregidos y las instrucciones de instalación de MABS V3 UR1, vea el artículo [KB4534062](https://support.microsoft.com/help/4534062).

>[!NOTE]
>Con MABS V3 UR1, la compatibilidad con el agente de protección de 32 bits está en desuso. Vea [Desuso del agente de protección de 32 bits](#32-bit-protection-agent-deprecation).

### <a name="faster-backups-with-tiered-storage-using-ssds"></a>Copias de seguridad más rápidas con el almacenamiento en capas mediante SSD

En MABS V2 se presentó [Modern Backup Storage](backup-mabs-add-storage.md) (MBS), para mejorar el rendimiento y uso del almacenamiento. En MBS se usa ReFS como sistema de archivos subyacente y está diseñado para utilizar almacenamiento híbrido como el almacenamiento en capas.

Para lograr la escala y el rendimiento mediante MBS, se recomienda usar un pequeño porcentaje (4 % del almacenamiento total) de almacenamiento flash (SSD) con MABS V3 UR1 como un volumen en capas junto con el almacenamiento en disco duros DPM. MABS V3 UR1 con almacenamiento en capas ofrece copias de seguridad entre un 50 y un 70 % más rápidas. Consulte el artículo de DPM [Configuración de MBS con almacenamiento en capas](/system-center/dpm/add-storage?view=sc-dpm-2019#set-up-mbs-with-tiered-storage) para obtener los pasos de configuración el almacenamiento en capas.

### <a name="support-for-refs-volumes"></a>Compatibilidad con volúmenes ReFS

Con MABS V3 UR1, puede realizar una copia de seguridad de los volúmenes y las cargas de trabajo de ReFS implementados en el volumen ReFS. Puede realizar una copia de seguridad de las siguientes cargas de trabajo implementadas en los volúmenes ReFS:

* Sistema operativo (64 bits): Windows Server 2019, 2016, 2012 R2, 2012.
* SQL Server: SQL Server 2019, SQL Server 2017, 2016.
* Exchange: Exchange 2019, 2016.
* SharePoint: SharePoint 2019, 2016 con el Service Pack más reciente.

>[!NOTE]
> La copia de seguridad de máquinas virtuales Hyper-V almacenadas en un volumen ReFS es compatible con MABS V3

>[IMPORTANTE] Hemos detectado algunos problemas con la copia de seguridad de volúmenes de ReFS desduplicados. Estamos trabajando para corregir los problemas y actualizaremos esta sección en cuanto tengamos una solución disponible. Hasta entonces, vamos a retirar la compatibilidad con la copia de seguridad de volúmenes de ReFS desduplicados de MABSv3 UR1.

### <a name="azure-vmware-solution-protection-support"></a>Compatibilidad con la protección de Azure VMware Solution

Con MABS V3 UR1, ahora puede proteger las máquinas virtuales implementadas en [Azure VMware Solution](../azure-vmware/index.yml).

### <a name="vmware-parallel-backups"></a>Copias de seguridad paralelas de VMware

Con MABS V3 UR1, todas las copias de seguridad de máquinas virtuales de VMware de un solo grupo de protección serán paralelas, lo que genera copias de seguridad de máquinas virtuales un 25 % más rápidas.
Con las versiones anteriores de MABS, las copias de seguridad paralelas se realizaban solo entre grupos de protección. Con MABS V3 UR1, los trabajos de replicación diferencial de VMware se ejecutan en paralelo. De forma predeterminada, el número de trabajos que se ejecuta en paralelo está establecido en 8. Obtenga más información sobre las [copias de seguridad paralelas de VMware](backup-azure-backup-server-vmware.md#vmware-parallel-backups).

### <a name="disk-exclusion-for-vmware-vm-backup"></a>Exclusión de discos para copia de seguridad de máquina virtual de VMware

Con MABS V3 UR1, puede excluir discos específicos de una copia de seguridad de máquina virtual de VMware. Obtenga más información sobre la [exclusión de discos de la copia de seguridad de máquina virtual de VMware](backup-azure-backup-server-vmware.md#exclude-disk-from-vmware-vm-backup).  

### <a name="support-for-additional-layer-of-authentication-to-delete-online-backup"></a>Compatibilidad con una capa adicional de autenticación para eliminar la copia de seguridad en línea

Con MABS V3 UR1, se agrega una capa de autenticación adicional a las operaciones críticas. Se le pedirá que escriba un PIN de seguridad al realizar operaciones de tipo **Detener la protección con eliminación de datos**.

### <a name="offline-backup-improvements"></a>Mejoras en la copia de seguridad sin conexión

MABS V3 UR1 mejora la experiencia de copia de seguridad sin conexión con el servicio Azure Import/Export. Para obtener más información, vea los pasos actualizados [aquí](./backup-azure-backup-server-import-export.md).

>[!NOTE]
>La actualización también aporta la versión preliminar para Copia de seguridad sin conexión con Azure Data Box en MABS. Póngase en contacto con [SystemCenterFeedback@microsoft.com](mailto:SystemCenterFeedback@microsoft.com) para obtener más información.

### <a name="new-cmdlet-parameter"></a>Nuevo parámetro de cmdlet

MABS V3 UR1 incluye un nuevo parámetro **[-CheckReplicaFragmentation]** . El nuevo parámetro calcula el porcentaje de fragmentación de una réplica y se incluye en el cmdlet **Copy-DPMDatasourceReplica**.

### <a name="32-bit-protection-agent-deprecation"></a>Desuso del agente de protección de 32 bits

Con MABS V3 UR1, ya no se admite la compatibilidad con el agente de protección de 32 bits. No podrá proteger las cargas de trabajo de 32 bits después de actualizar el servidor MABS V3 a UR1. Los agentes de protección de 32 bits existentes estarán en un estado deshabilitado y se producirá un error en las copias de seguridad programadas con un mensaje en el que se indicará que **el agente está deshabilitado**. Si quiere conservar los datos de copia de seguridad de estos agentes, puede detener la protección con la opción Conservar datos. De lo contrario, se puede quitar el agente de protección.

>[!NOTE]
>Revise la [matriz de protección actualizada](./backup-mabs-protection-matrix.md) para obtener información sobre las cargas de trabajo compatibles para la protección con MABS UR 1.

## <a name="whats-new-in-mabs-v3-rtm"></a>Novedades de MABS V3 RTM

La versión 3 de Microsoft Azure Backup Server (MABS V3) incluye correcciones de errores críticos, compatibilidad con Windows Server 2019, con SQL 2017 y otras características y mejoras. Para ver la lista de errores corregidos y las instrucciones de instalación de MABS V3, consulte el artículo de KB [4457852](https://support.microsoft.com/help/4457852/microsoft-azure-backup-server-v3).

Las siguientes características se incluyen en MABS V3:

### <a name="volume-to-volume-migration"></a>Migración de volumen a volumen

Con Modern Backup Storage (MBS) en MABS V2, anunciábamos almacenamiento con reconocimiento de la carga de trabajo, donde se configuran determinadas cargas de trabajo para la copia de seguridad en almacenamiento específico, según las propiedades de almacenamiento. Sin embargo, después de la configuración, puede que tenga la necesidad de mover copias de seguridad de algunos orígenes de datos a otro almacenamiento para la utilización optimizada de los recursos. MABS V3 le ofrece la funcionalidad de migrar las copias de seguridad y configurarlas para su almacenamiento en un volumen diferente en [tres pasos](https://techcommunity.microsoft.com/t5/system-center-blog/sc-2016-dpm-ur4-migrate-backup-storage-in-3-simple-steps/ba-p/351842).

### <a name="prevent-unexpected-data-loss"></a>Evitar la pérdida inesperada de datos

En las empresas, un equipo de administradores administra MABS. Aunque existen directrices sobre el almacenamiento que debe usarse para las copias de seguridad, el hecho de dar a MABS un volumen incorrecto como almacenamiento de copia de seguridad puede dar lugar a la pérdida de datos críticos. Con MABS V3, puede evitar ese tipo de escenarios si configura esos volúmenes como los que no están disponibles para el almacenamiento mediante [estos cmdlets de PowerShell](./backup-mabs-add-storage.md).

### <a name="custom-size-allocation"></a>Asignación de tamaño personalizada

Modern Backup Storage (MBS) consume almacenamiento fino, como y cuando sea necesario. Para ello, MABS calcula el tamaño de los datos de la copia de seguridad cuando se configura para la protección. Sin embargo, si se realiza la copia de seguridad de muchos archivos y carpetas juntos, como en el caso de un servidor de archivos, el cálculo del tamaño puede llevar mucho tiempo. Con MABS V3, puede configurar MABS para aceptar el tamaño del volumen como valor predeterminado, en lugar de calcular el tamaño de cada archivo, lo que ahorra tiempo.

### <a name="optimized-cc-for-rct-vms"></a>CC optimizado para máquinas virtuales RCT

MABS usa RCT (el seguimiento de cambios nativo de Hyper-V), lo que reduce la necesidad de laboriosas comprobaciones de coherencia en situaciones donde la máquina virtual se bloquea. RCT proporciona una mejor resistencia que el seguimiento de cambios proporcionado por las copias de seguridad basadas en instantáneas de VSS. MABS V3 optimiza el consumo de red y almacenamiento aún más al transferir solo los datos cambiados durante las comprobaciones de coherencia.

### <a name="support-to-tls-12"></a>Compatibilidad con TLS 1.2

TLS 1.2 es el modo seguro de comunicación sugerido por Microsoft con el mejor cifrado de su clase. MABS admite ahora la comunicación TLS 1.2 entre MABS y los servidores protegidos, para la autenticación basada en certificados y las copias de seguridad en la nube.

### <a name="vmware-vm-protection-support"></a>Compatibilidad con la protección de máquinas virtuales de VMware

Ahora, se admite la copia de seguridad de máquinas virtuales de VMware para las implementaciones en producción. MABS V3 ofrece las siguientes características para la protección de máquinas virtuales de VMware:

* Compatibilidad con vCenter y ESXi 6.5, además de compatibilidad con 5.5 y 6.0.
* Protección automática de máquinas virtuales de VMware en la nube. Si se agregan nuevas máquinas virtuales de VMware a una carpeta protegida, se protegen de forma automática en el disco y en la nube.
* Mejoras de la eficiencia de recuperación para la recuperación de ubicación alternativa de VMware.

### <a name="sql-2017-support"></a>Compatibilidad con SQL 2017

MABS V3 se puede instalar con SQL 2017 como base de datos de MABS. Puede actualizar el servidor de SQL Server de SQL 2016 a SQL 2017, o instalarlo de nuevo. También puede hacer una copia de seguridad de la carga de trabajo de SQL 2017 en entornos agrupados y no agrupados con MABS V3.

### <a name="windows-server-2019-support"></a>Compatibilidad con Windows Server 2019

MABS V3 se puede instalar en Windows Server 2019. Para usar MABS V3 con WS2019, puede actualizar el sistema operativo a WS2019 antes de instalar MABS V3 o de actualizar a esta versión. También puede actualizar el sistema operativo después de instalar la versión 3 en WS2016 o de actualizar a esta versión.

MABS V3 es una versión completa y se puede instalar directamente en Windows Server 2016 y Windows Server 2019, o se puede actualizar desde MABS V2. Antes de instalar Backup Server V3 o actualizar a esta versión, lea este artículo sobre los requisitos previos de instalación.
Puede encontrar más información sobre los pasos de instalación o actualización de MABS [aquí](./backup-azure-microsoft-azure-backup.md#software-package).

> [!NOTE]
>
> MABS tiene la misma base de código que System Center Data Protection Manager. MABS v3 es equivalente a Data Protection Manager 1807. MABS V3 UR1 es equivalente a Data Protection Manager 2019 UR1.

## <a name="next-steps"></a>Pasos siguientes

Obtenga información sobre cómo preparar el servidor o empezar a proteger la carga de trabajo:

* [Preparar cargas de trabajo de Backup Server](backup-azure-microsoft-azure-backup.md)
* [Usar Backup Server para hacer una copia de seguridad de un servidor de VMware](backup-azure-backup-server-vmware.md)
* [Usar Backup Server para hacer una copia de seguridad de SQL Server](backup-azure-sql-mabs.md)
* [Usar Modern Backup Storage con Backup Server](backup-mabs-add-storage.md)
