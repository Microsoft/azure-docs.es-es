---
title: Preguntas frecuentes sobre la copia de seguridad de archivos de Azure
description: En este artículo, encontrará respuestas a preguntas habituales sobre cómo proteger los recursos compartidos de archivos de Azure con el servicio Azure Backup.
ms.date: 04/22/2020
ms.topic: conceptual
ms.openlocfilehash: 6c2ef95a6303fd061b1ce486e893ba9812b83e14
ms.sourcegitcommit: 5b8fb60a5ded05c5b7281094d18cf8ae15cb1d55
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87382719"
---
# <a name="questions-about-backing-up-azure-files"></a>Preguntas acerca de la copia de seguridad de archivos de Azure

En este artículo se ofrecen respuestas a preguntas habituales acerca de la copia de seguridad de archivos de Azure. En algunas de las respuestas, hay vínculos a artículos que tienen información completa. También se pueden publicar preguntas sobre el servicio Azure Backup en la [página de preguntas y respuestas de Microsoft para su debate](/answers/topics/azure-backup.html).

Para examinar rápidamente las secciones de este artículo, use los vínculos de la derecha que aparecen debajo de **En este artículo**.

## <a name="configuring-the-backup-job-for-azure-files"></a>Configuración del trabajo de copia de seguridad de archivos de Azure

### <a name="why-cant-i-see-some-of-my-storage-accounts-that-i-want-to-protect-which-contain-valid-azure-file-shares"></a>¿Por qué no puedo ver algunas de las cuentas de almacenamiento que quiero proteger, que contienen recursos compartidos de archivos de Azure válidos?

Consulte la [matriz de compatibilidad para la copia de seguridad de recursos compartidos de archivos de Azure](azure-file-share-support-matrix.md) para asegurarse de que la cuenta de almacenamiento pertenece a uno de los tipos de cuenta de almacenamiento admitidos. También es posible que la cuenta de almacenamiento que está buscando ya esté protegida o registrada con otro almacén. [Anule el registro de la cuenta de almacenamiento](manage-afs-backup.md#unregister-a-storage-account) desde el almacén para descubrir la cuenta de almacenamiento en otros almacenes para la protección.

### <a name="why-cant-i-see-some-of-my-azure-file-shares-in-the-storage-account-when-im-trying-to-configure-backup"></a>¿Por qué no veo algunos de mis recursos compartidos de archivos de Azure en la cuenta de almacenamiento cuando intento configurar la copia de seguridad?

Compruebe si el recurso compartido de archivos de Azure ya está protegido en el mismo almacén de Recovery Services o si se ha eliminado recientemente.

### <a name="can-i-protect-file-shares-connected-to-a-sync-group-in-azure-files-sync"></a>¿Puedo proteger recursos compartidos de archivos conectado a un grupo de sincronización en Azure File Sync?

Sí. La protección de recursos compartidos de archivos de Azure conectados a grupos de sincronización está habilitada.

### <a name="when-trying-to-back-up-file-shares-i-clicked-on-a-storage-account-for-discovering-the-file-shares-in-it-however-i-didnt-protect-them-how-do-i-protect-these-file-shares-with-any-other-vault"></a>Al intentar realizar la copia de seguridad de recursos compartidos de archivos, hice clic en una cuenta de almacenamiento para detectar los recursos compartidos de archivos de ella. Sin embargo, no los he protegido. ¿Cómo puedo proteger estos recursos compartidos de archivos con cualquier otro almacén?

Al intentar realizar una copia de seguridad, si selecciono una cuenta de almacenamiento para detectar los recursos compartidos que hay en ella, se registra la cuenta de almacenamiento con el almacén desde el que se realiza esto. Si elige proteger los recursos compartidos de archivos con un almacén diferente, [anule el registro](manage-afs-backup.md#unregister-a-storage-account) de la cuenta de almacenamiento seleccionada de este almacén.

### <a name="why-cant-i-change-the-vault-to-configure-backup-for-the-file-share"></a>¿Por qué no puedo cambiar el almacén para configurar la copia de seguridad del recurso compartido de archivos?

Si la cuenta de almacenamiento ya está registrada con un almacén u otros recursos compartidos de archivos de la cuenta de almacenamiento están protegidos mediante un almacén, no se le ofrece la opción de cambiarla, ya que todos los recursos compartidos de archivos de una cuenta de almacenamiento solo se pueden proteger en el mismo almacén. En caso de que quiera cambiar el almacén, deberá [detener la protección de todos los recursos compartidos de archivos de la cuenta de almacenamiento](manage-afs-backup.md#stop-protection-on-a-file-share) desde el almacén conectado, [anular el registro](manage-afs-backup.md#unregister-a-storage-account) de la cuenta de almacenamiento y, a continuación, elegir otro almacén para la protección.

### <a name="can-i-change-the-vault-to-which-i-back-up-my-file-shares"></a>¿Puedo cambiar el almacén en el que hago la copia de seguridad de mis recursos compartidos de archivos?

Sí. Sin embargo, deberá [detener la protección en un recurso compartido de archivos](manage-afs-backup.md#stop-protection-on-a-file-share) desde el almacén conectado, [anular el registro](manage-afs-backup.md#unregister-a-storage-account) de esta cuenta de almacenamiento y protegerla desde un almacén distinto.

### <a name="can-i-protect-two-different-file-shares-from-the-same-storage-account-to-different-vaults"></a>¿Puedo proteger dos recursos compartidos de archivos diferentes desde la misma cuenta de almacenamiento en almacenes diferentes?

No. Todos los recursos compartidos de archivos de una cuenta de almacenamiento solo se pueden proteger en el mismo almacén.

## <a name="backup"></a>Copia de seguridad

### <a name="what-should-i-do-if-my-backups-start-failing-due-to-the-maximum-limit-reached-error"></a>¿Qué debo hacer si las copias de seguridad empiezan a generar errores debido al error de límite máximo alcanzado?

En cualquier momento dado, puede tener hasta 200 instantáneas de un recurso compartido de archivos. El límite incluye las instantáneas que toma Azure Backup como se define en la directiva. Si las copias de seguridad empiezan a generar errores después de alcanzar el límite, elimine las instantáneas a petición para que las próximas copias de seguridad sean correctas.

## <a name="restore"></a>Restauración

### <a name="can-i-recover-from-a-deleted-azure-file-share"></a>¿Se pueden recuperar los recursos compartidos de archivos de Azure eliminados?

Si el estado del recurso compartido de archivos es eliminado temporalmente, primero debe recuperar el recurso compartido de archivos para poder realizar la operación de restauración. La operación de recuperación devolverá el recurso compartido de archivos al estado activo, donde puede realizar la restauración a cualquier momento dado. Para obtener información sobre cómo recuperar el recurso compartido de archivos, visite [este vínculo](../storage/files/storage-files-enable-soft-delete.md?tabs=azure-portal#restore-soft-deleted-file-share) o consulte [Script para recuperar un recurso compartido de archivos](./scripts/backup-powershell-script-undelete-file-share.md). Si el recurso compartido de archivos se eliminó de forma permanente, no podrá restaurar el contenido ni las instantáneas.

### <a name="can-i-restore-from-backups-if-i-stopped-protection-on-an-azure-file-share"></a>¿Puedo restaurar las copias de seguridad si detuve la protección en un recurso compartido de archivos de Azure?

Sí. Si ha elegido **Retener datos de copia de seguridad** cuando detuvo la protección, puede restaurarlas desde todos los puntos de restauración existentes.

### <a name="what-happens-if-i-cancel-an-ongoing-restore-job"></a>¿Qué ocurre si se cancela un trabajo de restauración en curso?

Si se cancela un trabajo de restauración en curso, el proceso de restauración se detiene y todos los archivos restaurados antes de la cancelación permanecen en el destino configurado (ubicación original o alternativa) sin las reversiones.

## <a name="manage-backup"></a>Administración de copias de seguridad

### <a name="can-i-use-powershell-to-configuremanagerestore-backups-of-azure-file-shares"></a>¿Se puede usar PowerShell para configurar/administrar/restaurar copias de seguridad de los recursos compartidos de Azure File?

Sí. Consulte [aquí](backup-azure-afs-automation.md) la documentación detallada.

### <a name="can-i-access-the-snapshots-taken-by-azure-backups-and-mount-them"></a>¿Puedo acceder a las instantáneas realizadas por las instancias de Azure Backup y montarlas?

Es posible acceder a todas las instantáneas realizadas por Azure Backup desde el portal, PowerShell o la CLI. Para más información acerca de las instantáneas de recurso compartido de Azure Files, consulte [Información general de instantáneas de recurso compartido de Azure Files](../storage/files/storage-snapshots-files.md).

### <a name="what-is-the-maximum-retention-i-can-configure-for-backups"></a>¿Cuál es la retención máxima que puedo configurar para las copias de seguridad?

Consulte la [matriz de compatibilidad](azure-file-share-support-matrix.md) para obtener más información sobre la retención máxima. Azure Backup hace un cálculo en tiempo real del número de instantáneas cuando se escriben los valores de retención al configurar la directiva de copia de seguridad. En cuanto el número de instantáneas correspondientes a los valores de retención definidos supera las 200, el portal mostrará una advertencia que le solicitará que ajuste los valores de retención. Esto es para que no supere el límite del número máximo de instantáneas que admite Azure Files para cualquier recurso compartido de archivos en cualquier momento.

### <a name="what-is-the-impact-on-existing-recovery-points-and-snapshots-when-i-modify-the-backup-policy-for-an-azure-file-share-to-switch-from-daily-policy-to-gfs-policy"></a>¿Cuál es el efecto en las instantáneas y los puntos de recuperación existentes cuando se modifica la Directiva de copia de seguridad de un recurso compartido de archivos de Azure para pasar de la "directiva diaria" a la "directiva GFS"?

Cuando se modifica una directiva de copia de seguridad diaria a la directiva GFS (agregando retención semanal/mensual/anual), el comportamiento es el siguiente:

- **Retención**: Si va a agregar una retención semanal/mensual/anual como parte de la modificación de la directiva, todos los puntos de recuperación futuros creados como parte de la copia de seguridad programada se etiquetarán según la nueva directiva. Todos los puntos de recuperación existentes se seguirán considerando puntos de recuperación diarios y por eso no se etiquetarán como semanal/mensual/anual.

- **Limpieza de instantáneas y puntos de recuperación**:

  - Si se amplía la retención diaria, la fecha de expiración de los puntos de recuperación existentes se actualiza según el valor de retención diaria configurado en la nueva Directiva.
  - Si se reduce la retención diaria, las instantáneas y los puntos de recuperación existentes se marcan para su eliminación en el siguiente trabajo de ejecución de limpieza según el valor de retención diaria configurado en la nueva directiva y después se eliminan.

Este es un ejemplo de su funcionamiento:

#### <a name="existing-policy-p1"></a>Directiva existente [P1]

|Tipo de retención |Programación |Retención  |
|---------|---------|---------|
|Diario    |    Todos los días a las 20:00 horas    |  100 días       |

#### <a name="new-policy-modified-p1"></a>Nueva directiva [P1 modificada]

| Tipo de retención | Programación                       | Retención |
| -------------- | ------------------------------ | --------- |
| Diario          | Todos los días a las 21:00 horas              | 50 días   |
| Semanal         | El domingo a las 21:00 horas              | 3 semanas   |
| Mensual        | El último lunes a las 21:00 horas         | 1 mes   |
| Anual         | El tercer domingo de enero a las 21:00 horas | 4 años   |

#### <a name="impact"></a>Impacto

1. La fecha de expiración de los puntos de recuperación existentes se ajustará según el valor de retención diaria de la nueva directiva: es decir, 50 días. Por lo que los puntos de recuperación anteriores a 50 días se marcarán para su eliminación.

2. Los puntos de recuperación existentes no se etiquetarán como semanal/mensual/anual según la nueva directiva.

3. Todas las copias de seguridad futuras se desencadenarán de acuerdo con la nueva programación: es decir, a las 21:00 horas.

4. La fecha de expiración de todos los puntos de recuperación futuros se alineará con la nueva directiva.

>[!NOTE]
>Los cambios de directiva solo afectarán a los puntos de recuperación creados como parte de la ejecución del trabajo de copia de seguridad programada. En el caso de las copias de seguridad a petición, la retención viene determinada por el valor de **Conservar copia de seguridad hasta** especificado en el momento de realizar la copia de seguridad.

### <a name="what-is-the-impact-on-existing-recovery-points-when-i-modify-an-existing-gfs-policy"></a>¿Cuál es el efecto en los puntos de recuperación existentes cuando se modifica una directiva GFS existente?

Cuando se aplica una nueva directiva en recursos compartidos de archivos, todas las copias de seguridad programadas futuras se realizarán según la programación configurada en la directiva modificada.  La retención de todos los puntos de recuperación existentes se alinea según los nuevos valores de retención configurados. Por lo que, si se amplía la retención, los puntos de recuperación existentes se marcan para que se conserven según la nueva directiva. Si se reduce la retención, se marcan para su limpieza en el siguiente trabajo de limpieza y luego se eliminan.

Este es un ejemplo de su funcionamiento:

#### <a name="existing-policy-p2"></a>Directiva existente [P2]

| Tipo de retención | Programación           | Retención |
| -------------- | ------------------ | --------- |
| Diario          | Todos los días a las 20:00 horas | 50 días   |
| Semanal         | El lunes a las 20:00 horas  | 3 semanas   |

#### <a name="new-policy-modified-p2"></a>Nueva directiva [P2 modificada]

| Tipo de retención | Programación               | Retención |
| -------------- | ---------------------- | --------- |
| Diario          | Todos los días a las 21:00 horas     | 10 días   |
| Semanal         | El lunes a las 21:00 horas      | Dos semanas   |
| Mensual        | El último lunes a las 21:00 horas | 2 meses  |

#### <a name="impact-of-change"></a>Impacto del cambio

1. La fecha de expiración de los puntos de recuperación diarios existentes se alineará según el nuevo valor de retención diaria, es decir, 10 días. Por lo que los puntos de recuperación diarios anteriores a 10 días se eliminarán.

2. La fecha de expiración de los puntos de recuperación semanales existentes se alineará según el nuevo valor de retención semanal, es decir, dos semanas. Por lo que los puntos de recuperación semanales anteriores a dos semanas se eliminarán.

3. Los puntos de recuperación mensuales solo se crearán como parte de copias de seguridad futuras según la nueva configuración de directiva.

4. La fecha de expiración de todos los puntos de recuperación futuros se alineará con la nueva directiva.

>[!NOTE]
>Los cambios de directiva solo afectarán a los puntos de recuperación creados como parte de la copia de seguridad programada. En el caso de las copias de seguridad a petición, la retención viene determinada por el valor de **Conservar copia de seguridad hasta** especificado en el momento de realizar la copia de seguridad.

## <a name="next-steps"></a>Pasos siguientes

- [Solución de problemas al realizar copias de seguridad de recursos compartidos de archivos de Azure](troubleshoot-azure-files.md)
