---
title: Funcionalidad de restauración instantánea de Azure
description: Funcionalidad de restauración instantánea de Azure y preguntas frecuentes de la pila de copia de seguridad de VM, modelo de implementación de Resource Manager
ms.reviewer: sogup
ms.topic: conceptual
ms.date: 04/23/2019
ms.openlocfilehash: 6ea4c3757da4e24ae0455cf35f119bf57ed644a6
ms.sourcegitcommit: 3d56d25d9cf9d3d42600db3e9364a5730e80fa4a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/03/2020
ms.locfileid: "87531836"
---
# <a name="get-improved-backup-and-restore-performance-with-azure-backup-instant-restore-capability"></a>Rendimiento mejorado de la copia de seguridad y la restauración con la funcionalidad de restauración instantánea de Azure Backup

> [!NOTE]
> De acuerdo con los comentarios de los usuarios, vamos a cambiar el nombre de **pila de copia de seguridad de VM V2** por **restauración instantánea** para reducir la confusión con la funcionalidad de Azure Stack.
> Todos los usuarios de Azure Backup ya se actualizaron a **la restauración instantánea**.

El nuevo modelo de restauración instantánea proporciona las siguientes mejoras en la característica:

* Posibilidad de ver instantáneas tomadas como parte de un trabajo de copia de seguridad que está disponible para la recuperación sin tener que esperar a que finalice la transferencia de datos al almacén. Reduce el tiempo de espera para la copia de instantáneas en el almacén antes de desencadenar la restauración.
* Reduce los tiempos de copia de seguridad y restauración al conservarse las instantáneas localmente durante dos días (de forma predeterminada). Este valor predeterminado de retención de instantánea se puede configurar en cualquier valor entre 1 y 5 días.
* Admite tamaños de disco de hasta 32 TB. No se recomienda cambiar el tamaño de los discos en Azure Backup.
* Admite discos SSD estándar junto con discos HDD estándar y SSD Premium.
* Capacidad de usar cuentas de almacenamiento originales de una VM no administrada (por disco) al restaurar. Esta capacidad existe aun cuando la máquina virtual tenga discos distribuidos entre cuentas de almacenamiento. Acelera las operaciones de restauración para una amplia variedad de configuraciones de máquina virtual.
* En el caso de la copia de seguridad de máquinas virtuales que usan discos Premium sin administrar, con restauración instantánea, se recomienda asignar el *50 %* de espacio libre del espacio de almacenamiento total asignado, que **solo** es necesario para la primera copia de seguridad. El espacio libre del 50 % no es un requisito para las copias de seguridad una vez completada la primera copia de seguridad.

## <a name="whats-new-in-this-feature"></a>Novedades de esta característica

En la actualidad, el trabajo de copia de seguridad consta de dos fases:

1. Toma de una instantánea de máquina virtual.
2. Transferencia de una instantánea de máquina virtual al almacén de Azure Recovery Services.

Un punto de recuperación se considera creado solo después de que se terminan las fases 1 y 2. Como parte de esta actualización, se crea un punto de recuperación en cuanto finaliza la instantánea. Este punto de recuperación de tipo instantánea se puede usar para realizar una restauración con el mismo flujo de restauración. Este punto de recuperación se puede identificar en Azure Portal al usar "instantánea" como tipo de punto de recuperación y, después de transferir la instantánea al almacén, el tipo de punto de recuperación cambia a "instantánea y almacén".

![Trabajo de copia de seguridad de la pila de copia de seguridad de VM, modelo de implementación de Resource Manager, almacenamiento y almacén](./media/backup-azure-vms/instant-rp-flow.png)

De manera predeterminada, las instantáneas de recuperación se conservan durante dos días. Esta característica permite la operación de restauración a partir de estas instantáneas, lo cual reduce los tiempos de restauración. Reduce el tiempo necesario para transformar y copiar datos desde el almacén.

## <a name="feature-considerations"></a>Consideraciones de la característica

* Las instantáneas se almacenan junto con los discos para acelerar la creación de puntos de recuperación y las operaciones de restauración. Como resultado, verá los costos de almacenamiento que corresponden a las instantáneas tomadas durante este período.
* Las instantáneas incrementales se almacenan como blobs en páginas. A todos los usuarios que usen discos no administrados se les cobrará por las instantáneas almacenadas en su cuenta de almacenamiento local. Como las colecciones de puntos de restauración utilizadas por las copias de seguridad de máquinas virtuales administradas usan instantáneas de blobs en el nivel de almacenamiento subyacente, para discos administrados, verá los costos correspondientes a los precios de instantáneas de blobs, que son incrementales.
* Para las cuentas de almacenamiento premium, las instantáneas tomadas para los puntos de recuperación de instantánea se consideran en el límite de 10 TB de espacio asignado.
* Puede obtener la capacidad de configurar la retención de instantáneas según las necesidades de restauración. En función de los requisitos, puede configurar la retención de instantáneas en un día como mínimo en la hoja de la directiva de copia de seguridad como se explica a continuación. Esto le ayuda a ahorrar costos de retención de instantáneas si no lleva a cabo restauraciones con frecuencia.
* Es una actualización unidireccional; una vez actualizada a la restauración instantánea, no se puede volver atrás.

>[!NOTE]
>Con esta actualización instantánea de la restauración, la duración de la retención de instantáneas de todos los clientes (**los nuevos y los ya existentes**) se establecerá en un valor predeterminado de dos días. Pese a ello, puede establecer la duración según sus necesidades en cualquier valor entre 1 y 5 días.

## <a name="cost-impact"></a>Impacto sobre los costos

Las instantáneas incrementales se almacenan en la cuenta de almacenamiento de la máquina virtual, que se usa para la recuperación instantánea. Una "instantánea incremental" significa que el espacio ocupado por ella es igual que el espacio ocupado por las páginas escritas una vez creada la instantánea. La facturación se realiza aún por GB de espacio que ocupa la instantánea y el precio por GB es el que se menciona en la [página de precios](https://azure.microsoft.com/pricing/details/managed-disks/). En el caso de las máquinas virtuales que usan discos no administrados, las instantáneas se pueden visualizar en el menú del archivo VHD de cada disco. En el caso de discos administrados, las instantáneas se almacenan en un recurso de recopilación de puntos de restauración de un grupo de recursos designado y las instantáneas en sí no son directamente visibles.

>[!NOTE]
> Se ha corregido la retención de instantáneas a 5 días para las directivas semanales.

## <a name="configure-snapshot-retention"></a>Configuración de la retención de instantáneas

### <a name="using-azure-portal"></a>Uso de Azure Portal

En Azure Portal puede ver que se ha agregado un campo a la hoja **Directiva de copia de seguridad de máquina virtual** en la sección **Restauración instantánea**. Puede cambiar la duración de la retención de instantáneas en la hoja **Directiva de copia de seguridad de máquina virtual** para todas las máquinas virtuales asociadas con la directiva de copia de seguridad específica.

![Funcionalidad de restauración instantánea](./media/backup-azure-vms/instant-restore-capability.png)

### <a name="using-powershell"></a>Usar PowerShell

>[!NOTE]
> Desde Az PowerShell 1.6.0 y versiones posteriores, puede actualizar el período de retención de instantáneas para la restauración instantánea en la directiva mediante PowerShell

```powershell
$bkpPol = Get-AzureRmRecoveryServicesBackupProtectionPolicy -WorkloadType "AzureVM"
$bkpPol.SnapshotRetentionInDays=5
Set-AzureRmRecoveryServicesBackupProtectionPolicy -policy $bkpPol
```

La retención de instantáneas predeterminada para cada directiva está establecida en dos días. El usuario puede cambiar el valor a un mínimo de uno y un máximo de cinco días. Para las directivas semanales, se ha corregido la retención de instantáneas a cinco días.

## <a name="frequently-asked-questions"></a>Preguntas más frecuentes

### <a name="what-are-the-cost-implications-of-instant-restore"></a>¿Cuáles son las implicaciones sobre el costo de la restauración instantánea?

Las instantáneas se almacenan junto con los discos para acelerar la creación de puntos de recuperación y las operaciones de restauración. Como resultado, verá los costos de almacenamiento que se corresponden con el período de retención de la instantánea seleccionada como parte de la directiva de copia de seguridad de máquina virtual.

### <a name="in-premium-storage-accounts-do-the-snapshots-taken-for-instant-recovery-point-occupy-the-10-tb-snapshot-limit"></a>En las cuentas de Premium Storage, ¿las instantáneas que se toman para el punto de recuperación instantáneo ocupan el límite de 10 TB para instantáneas?

Sí, en el caso de las cuentas de Premium Storage, las instantáneas que se toman para el punto de recuperación instantáneo ocupan los 10 TB de espacio asignado.

### <a name="how-does-the-snapshot-retention-work-during-the-five-day-period"></a>¿Cómo funciona la retención de instantáneas durante el período de cinco días?

Cada día se toma una instantánea nueva, por lo tanto, hay cinco instantáneas incrementales individuales. El tamaño de la instantánea depende de la actividad de datos que, en la mayoría de los casos, es aproximadamente del 2 al 7 %.

### <a name="is-an-instant-restore-snapshot-an-incremental-snapshot-or-full-snapshot"></a>¿Es una instantánea de la restauración instantánea incremental o completa?

Las instantáneas realizadas como parte de la funcionalidad de restauración instantánea son incrementales.

### <a name="how-can-i-calculate-the-approximate-cost-increase-due-to-instant-restore-feature"></a>¿Cómo puedo calcular el aumento del costo aproximado debido a la característica de restauración instantánea?

Depende de la actividad de la máquina virtual. En un estado estable, podemos suponer que el aumento del costo es = actividad diaria del período de retención de instantáneas por costo de almacenamiento de máquina virtual por GB.

### <a name="if-the-recovery-type-for-a-restore-point-is-snapshot-and-vault-and-i-perform-a-restore-operation-which-recovery-type-will-be-used"></a>Si el tipo de recuperación de un punto de restauración es "Instantánea y almacén" y realizo una operación de restauración, ¿qué tipo de recuperación se usará?

Si el tipo de recuperación es "Instantánea y almacén", la restauración se realizará automáticamente desde la instantánea local, que se comparará con más rapidez con la restauración desde el almacén.

### <a name="what-happens-if-i-select-retention-period-of-restore-point-tier-2-less-than-the-snapshot-tier1-retention-period"></a>¿Qué ocurre si selecciono un período de retención de punto de restauración (nivel 2) inferior al de la instantánea (nivel 1)?

El nuevo modelo no permite eliminar el punto de restauración (nivel 2), a menos que se elimine la instantánea (nivel 1). Se recomienda programar un período de retención del punto de restauración (nivel 2) superior al período de retención de instantáneas.

### <a name="why-is-my-snapshot-existing-even-after-the-set-retention-period-in-backup-policy"></a>¿Por qué continúa existiendo mi instantánea incluso una vez expirado el tiempo de retención establecido en la directiva de copia de seguridad?

Si el punto de recuperación tiene instantáneas y que es el más reciente disponible, se conserva hasta el momento en que se realice la copia de seguridad siguiente correctamente. Esto atiende a lo estipulado en el diseño de la actual directiva de "recolección de elementos no utilizados" (GC), que exige que haya al menos un punto de recuperación siempre presente en caso de que se produzca un error en todas las copias de seguridad por un problema en la máquina virtual. En escenarios normales, los puntos de recuperación se eliminan en un máximo de 24 horas después de su expiración.

### <a name="i-dont-need-instant-restore-functionality-can-it-be-disabled"></a>No necesito la característica de restauración instantánea. ¿Se puede deshabilitar?

La característica de restauración instantánea está habilitada para todos los usuarios y no se puede deshabilitar. Puede reducir la retención de instantáneas a un mínimo de un día.

