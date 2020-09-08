---
title: Introducción a los almacenes de Recovery Services
description: Información general y comparación entre los almacenes de Recovery Services y los de Azure Backup.
ms.topic: conceptual
ms.date: 08/17/2020
ms.openlocfilehash: 6b6550bc8c5221547d55ad1e81a985b8ecd303e0
ms.sourcegitcommit: c6b9a46404120ae44c9f3468df14403bcd6686c1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88889624"
---
# <a name="recovery-services-vaults-overview"></a>Introducción a los almacenes de Recovery Services

En este artículo se describen las características de un almacén de Recovery Services. Un almacén de Recovery Services es una entidad de almacenamiento de Azure que aloja datos. Normalmente, los datos son copias de datos o información de configuración de máquinas virtuales (VM), cargas de trabajo, servidores o estaciones de trabajo. Puede usar almacenes de Recovery Services para almacenar datos de copia de seguridad de varios servicios de Azure como máquinas virtuales de IaaS (Linux o Windows) y bases de datos de Azure SQL. Los almacenes de Recovery Services son compatibles con System Center DPM, Windows Server, Azure Backup Server y muchos más. Los almacenes de Recovery Services facilitan la tarea de organizar los datos de copia de seguridad, al mismo tiempo que reduce al mínimo su sobrecarga administrativa. Los almacenes de Recovery Services se basan en el modelo de Azure Resource Manager de Azure, que proporciona características como las siguientes:

- **Funcionalidades mejoradas para ayudar a proteger datos de copia de seguridad**: con los almacenes de Recovery Services, Azure Backup proporciona funcionalidades de seguridad para proteger las copias de seguridad en la nube. Estas características de seguridad garantizan que puede proteger las copias de seguridad y recuperar datos de forma segura, incluso si los servidores de producción y copia de seguridad están en peligro. [Más información](backup-azure-security-feature.md)

- **Supervisión central para el entorno de TI híbrido**: con los almacenes de Recovery Services, puede supervisar no solo sus [máquinas virtuales de IaaS de Azure](backup-azure-manage-vms.md), sino también sus [recursos locales](backup-azure-manage-windows-server.md#manage-backup-items) desde un portal central. [Más información](backup-azure-monitoring-built-in-monitor.md)

- **Control de acceso basado en rol (RBAC)** : RBAC permite un control muy detallado de la administración de acceso en Azure. [Azure proporciona diversas funciones integradas](../role-based-access-control/built-in-roles.md) y Azure Backup tiene tres [roles integrados para administrar puntos de recuperación](backup-rbac-rs-vault.md). Los almacenes de Recovery Services son compatibles con RBAC, que restringe el acceso de copia de seguridad y restauración al conjunto definido de roles de usuario. [Más información](backup-rbac-rs-vault.md)

- **Eliminación temporal**:  Con la eliminación temporal, aunque un actor malintencionado elimine una copia de seguridad (o se eliminen por accidente datos de copia de seguridad), los datos de copia de seguridad se conservan durante 14 días adicionales, lo que permite la recuperación de ese elemento de copia de seguridad sin pérdida de datos. La retención adicional de 14 días de los datos de copia de seguridad con el estado de "eliminación temporal" no tiene costo alguno para el cliente. [Más información](backup-azure-security-feature-cloud.md).

- **Restauración entre regiones**:  la restauración entre regiones (CRR) permite restaurar máquinas virtuales de Azure en una región secundaria, que es una región emparejada de Azure. Si Azure declara un desastre en la región primaria, los datos replicados en la región secundaria estarán disponibles para su restauración en esta última región para mitigar el desastre de tiempo de inactividad real en la región primaria de su entorno. [Más información](backup-azure-arm-restore-vms.md#cross-region-restore).

## <a name="storage-settings-in-the-recovery-services-vault"></a>Configuración de almacenamiento en el almacén de Recovery Services

Un almacén de Recovery Services es una entidad que almacena las copias de seguridad y los puntos de recuperación creados a lo largo del tiempo. También contiene las directivas de copia de seguridad asociadas con las máquinas virtuales protegidas.

- Azure Backup administra automáticamente el almacenamiento para el almacén. Consulte cómo [cambiar la configuración de almacenamiento](./backup-create-rs-vault.md#set-storage-redundancy).

- Para obtener más información acerca de la redundancia de almacenamiento, consulte estos artículos sobre redundancia [geográfica](../storage/common/storage-redundancy.md) y [local](../storage/common/storage-redundancy.md).

## <a name="encryption-settings-in-the-recovery-services-vault"></a>Configuración del cifrado en el almacén de Recovery Services

En esta sección se describen las opciones disponibles para cifrar los datos de copia de seguridad almacenados en el almacén de Recovery Services.

### <a name="encryption-of-backup-data-using-platform-managed-keys"></a>Cifrado de datos de copia de seguridad mediante claves administradas por la plataforma

De forma predeterminada, todos los datos se cifran mediante claves administradas por la plataforma. No es necesario realizar ninguna acción explícita de su parte para habilitar este cifrado. Se aplica a todas las cargas de trabajo de las que se realiza una copia de seguridad en el almacén de Recovery Services.

### <a name="encryption-of-backup-data-using-customer-managed-keys"></a>Cifrado de datos de copia de seguridad mediante claves administradas por el cliente

Puede elegir cifrar los datos mediante las claves de cifrado que posee y administra. Azure Backup le permite usar las claves RSA almacenadas en Azure Key Vault para cifrar las copias de seguridad. La clave de cifrado utilizada para cifrar las copias de seguridad puede ser diferente de la que se usa para el origen. Los datos se protegen mediante una clave de cifrado de datos (DEK) basada en AES 256, que, a su vez, está protegida con las claves del usuario. Esto le proporciona un control total sobre los datos y las claves. Para permitir el cifrado, debe concederse al almacén de Recovery Services el acceso a la clave de cifrado en Azure Key Vault. Puede deshabilitar la clave o revocar el acceso siempre que sea necesario. Sin embargo, debe habilitar el cifrado con las claves antes de intentar proteger los elementos en el almacén.

Obtenga más información acerca de cómo cifrar los datos de copia de seguridad [mediante claves administradas por el cliente](encryption-at-rest-with-cmk.md).

## <a name="azure-advisor"></a>Azure Advisor

[Azure Advisor](../advisor/index.yml) es un consultor de nube personalizado que ayuda a optimizar el uso de Azure. Analiza el uso de Azure y proporciona recomendaciones oportunas para ayudar a optimizar y proteger las implementaciones. Proporciona recomendaciones en cuatro categorías: Alta disponibilidad, Seguridad, Rendimiento y Costo.

Azure Advisor proporciona [recomendaciones](../advisor/advisor-high-availability-recommendations.md#protect-your-virtual-machine-data-from-accidental-deletion) por hora para las máquinas virtuales de las que no se ha realizado una copia de seguridad, por lo que nunca se pierde la copia de seguridad de máquinas virtuales importantes. También puede controlar las recomendaciones al posponerlas.  Puede seleccionar la recomendación y habilitar la copia de seguridad en las máquinas virtuales en línea especificando el almacén (en el que se almacenarán las copias de seguridad) y la directiva de copia de seguridad (programación de copias de seguridad y retención de copias de seguridad).

![Azure Advisor](./media/backup-azure-recovery-services-vault-overview/azure-advisor.png)

## <a name="additional-resources"></a>Recursos adicionales

- [Escenarios admitidos y no admitidos de almacén](backup-support-matrix.md#vault-support)
- [Preguntas más frecuentes de almacén](backup-azure-backup-faq.md)

## <a name="next-steps"></a>Pasos siguientes

Use los artículos siguientes para realizar estos pasos:

- [Copia de seguridad de una máquina virtual de IaaS](backup-azure-arm-vms-prepare.md)
- [Copia de seguridad de Azure Backup Server](backup-azure-microsoft-azure-backup.md)
- [Copia de seguridad de Windows Server](backup-windows-with-mars-agent.md)
