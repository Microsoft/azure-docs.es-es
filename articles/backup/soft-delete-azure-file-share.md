---
title: Protección contra la eliminación accidental de los recursos compartidos de archivos de Azure
description: Obtenga información sobre cómo la eliminación temporal puede proteger los recursos compartidos de archivos de Azure contra la eliminación accidental.
ms.topic: conceptual
ms.date: 02/02/2020
ms.custom: references_regions
ms.openlocfilehash: 0ec2d3bf84aed19b608a92b6f21cd1674ba5b7cf
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87282711"
---
# <a name="accidental-delete-protection-for-azure-file-shares-using-azure-backup"></a>Protección contra la eliminación accidental de los recursos compartidos de archivos de Azure mediante Azure Backup

Para proporcionar protección contra los ciberataques o la eliminación accidental, la [eliminación temporal](../storage/files/storage-files-prevent-file-share-deletion.md) está habilitada para todos los recursos compartidos de archivos en una cuenta de almacenamiento cuando se configura la copia de seguridad para cualquier recurso compartido de archivos en la cuenta de almacenamiento correspondiente. Con la eliminación temporal, incluso si un actor malintencionado elimina el recurso compartido de archivos, el contenido del mencionado recurso compartido de archivos y los puntos de recuperación (instantáneas) se conservan durante un mínimo de catorce días adicionales, lo que permite la recuperación de los recursos compartidos de archivos sin pérdida de datos.  La eliminación temporal es compatible con las cuentas de almacenamiento Estándar y Premium, y Azure Backup habilita la opción para todas las cuentas de almacenamiento que hospedan recursos compartidos de archivos con copia de seguridad.

En el siguiente diagrama de flujo se explican los diferentes pasos y estados de un elemento de copia de seguridad cuando la eliminación temporal se habilita para recursos compartidos de archivos en una cuenta de almacenamiento:

 ![Gráfico de flujo de eliminación temporal](./media/soft-delete-afs/soft-delete-flow-chart.png)

## <a name="frequently-asked-questions"></a>Preguntas más frecuentes

### <a name="when-will-soft-delete-be-enabled-for-file-shares-in-my-storage-account"></a>¿Cuándo se habilitará la eliminación temporal para recursos compartidos de archivos en mi cuenta de almacenamiento?

Al configurar la copia de seguridad por primera vez para cualquier recurso compartido de archivos en una cuenta de almacenamiento, el servicio Azure Backup habilita la eliminación temporal para todos los recursos compartidos de archivos de la cuenta de almacenamiento correspondiente.

### <a name="can-i-configure-the-number-of-days-for-which-my-snapshots-and-restore-points-will-be-retained-in-soft-deleted-state-after-i-delete-the-file-share"></a>¿Puedo configurar el número de días durante los que se conservarán mis instantáneas y puntos de restauración en estado de eliminación temporal después de que elimine el recurso compartido de archivos?

Sí, puede establecer el período de retención según sus requisitos. En [este documento](../storage/files/storage-files-enable-soft-delete.md?tabs=azure-portal) se explican los pasos para configurar el período de retención. En el caso de las cuentas de almacenamiento con recursos compartidos de archivos de los que se ha hecho una copia de seguridad, la configuración de retención mínima debe ser de catorce días.

### <a name="does-azure-backup-reset-my-retention-setting-because-i-configured-it-to-less-than-14-days"></a>¿Restablece Azure Backup la configuración de retención porque la definí en menos de catorce días?

Desde el punto de vista de la seguridad, se recomienda tener una retención mínima de catorce días para las cuentas de almacenamiento con recursos compartidos de archivos de los que se ha hecho copia de seguridad. Por lo tanto, en cada ejecución de trabajo de copia de seguridad, si Azure Backup identifica que la configuración es inferior a catorce días, la restablece en catorce días.

### <a name="what-is-the-cost-incurred-during-the-retention-period"></a>¿Cuál es el costo en el que se incurre durante el período de retención?

Durante el período de eliminación temporal, el costo de la instancia protegida y el de almacenamiento de instantáneas permanecerán tal cual.  Además, se le cobrará por la capacidad usada según la tarifa normal para los recursos compartidos de archivos estándar y según tarifa de almacenamiento de instantáneas para los recursos compartidos de archivos premium.

### <a name="can-i-perform-a-restore-operation-when-my-data-is-in-soft-deleted-state"></a>¿Puedo realizar una operación de restauración si mis datos están en estado de eliminación temporal?

Primero debe recuperar el recurso compartido de archivos eliminado temporalmente para realizar operaciones de restauración. La operación de recuperación devolverá el recurso compartido de archivos al estado de copia de seguridad donde puede realizar la restauración a cualquier momento dado. Para obtener información sobre cómo recuperar el recurso compartido de archivos, visite [este vínculo](../storage/files/storage-files-enable-soft-delete.md?tabs=azure-portal#restore-soft-deleted-file-share) o consulte [Script para recuperar un recurso compartido de archivos](./scripts/backup-powershell-script-undelete-file-share.md).

### <a name="how-can-i-purge-the-data-of-a-file-share-in-a-storage-account-that-has-at-least-one-protected-file-share"></a>¿Cómo puedo purgar los datos de un recurso compartido de archivos en una cuenta de almacenamiento que tiene al menos un recurso compartido de archivos protegido?

Si tiene al menos un recurso compartido de archivos protegido en una cuenta de almacenamiento, la eliminación temporal está habilitada para todos los recursos compartidos de archivos en esa cuenta y los datos se retendrán durante catorce días después de la operación de eliminación. Pero si desea purgar los datos inmediatamente y no desea que se retengan, siga estos pasos:

1. Si ya ha eliminado el recurso compartido de archivos mientras estaba habilitada la eliminación temporal, recupere primero el recurso compartido de archivos del [portal de archivos](../storage/files/storage-files-enable-soft-delete.md?tabs=azure-portal#restore-soft-deleted-file-share) o mediante el [script para recuperar un recurso compartido de archivos](./scripts/backup-powershell-script-undelete-file-share.md).
2. Deshabilite la eliminación temporal para recursos compartidos de archivos en su cuenta de almacenamiento siguiendo los pasos mencionados en [este documento](../storage/files/storage-files-enable-soft-delete.md?tabs=azure-portal#disable-soft-delete).
3. Ahora, elimine el recurso compartido de archivos cuyo contenido desea purgar inmediatamente.

>[!NOTE]
>Debe realizar el paso 2 antes de que se ejecute el siguiente trabajo de copia de seguridad programada en el recurso compartido de archivos protegido de la cuenta de almacenamiento. El motivo es que, cada vez que se ejecuta el trabajo de copia de seguridad, se vuelve a habilitar la eliminación temporal para todos los recursos compartidos de archivos de la cuenta de almacenamiento.

>[!WARNING]
>Después de deshabilitar la eliminación temporal en el paso 2, cualquier operación de eliminación realizada en los recursos compartidos de archivos es una operación de eliminación permanente. Esto significa que, si elimina accidentalmente el recurso compartido del que se ha hecho una copia de seguridad después de deshabilitar la eliminación temporal, perderá todas las instantáneas y no podrá recuperar los datos.

### <a name="in-the-context-of-a-file-shares-soft-delete-setting-what-changes-does-azure-backup-do-when-i-unregister-a-storage-account"></a>En el contexto de la configuración de eliminación temporal de un recurso compartido de archivos, ¿qué cambios hace Azure Backup cuando anulo el registro de una cuenta de almacenamiento?

En el momento de la anulación del registro, Azure Backup comprueba la configuración del período de retención para los recursos compartidos de archivos y, si es superior o inferior a catorce días, deja el valor de retención como está. Sin embargo, si la retención es de catorce días, se considerará como habilitada por Azure Backup y, por tanto, deshabilitaremos la eliminación temporal durante el proceso de anulación del registro. Si desea anular el registro de la cuenta de almacenamiento manteniendo la configuración de retención tal y como está, habilítela de nuevo desde el panel de la cuenta de almacenamiento después de finalizar la anulación del registro. Puede consultar [este vínculo](../storage/files/storage-files-enable-soft-delete.md?tabs=azure-portal#restore-soft-deleted-file-share) para ver los pasos de configuración.

## <a name="next-steps"></a>Pasos siguientes

Obtenga información sobre cómo [hacer copias de seguridad de los recursos compartidos de archivos de Azure desde Azure Portal](backup-afs.md).
