---
title: Solución de problemas de copia de seguridad de recursos compartidos de archivos de Azure
description: Este artículo contiene información para solución de problemas que se producen al proteger recursos compartidos de archivos de Azure.
ms.date: 02/10/2020
ms.topic: troubleshooting
ms.openlocfilehash: 7b007a9ef893bb772929584eb3137c7a5200d756
ms.sourcegitcommit: 3543d3b4f6c6f496d22ea5f97d8cd2700ac9a481
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/20/2020
ms.locfileid: "86524495"
---
# <a name="troubleshoot-problems-while-backing-up-azure-file-shares"></a>Solución de problemas al realizar copias de seguridad de recursos compartidos de archivos de Azure

En este artículo se proporciona información de solución de problemas para cualquier problema que pueda experimentar durante la configuración de la copia de seguridad o la restauración de recursos compartidos de archivos de Azure mediante el servicio Azure Backup.

## <a name="common-configuration-issues"></a>Problemas de configuración comunes

### <a name="could-not-find-my-storage-account-to-configure-backup-for-the-azure-file-share"></a>No se pudo encontrar la cuenta de almacenamiento para configurar la copia de seguridad para los recursos compartidos de archivos de Azure

- Espere a que la detección finalice.
- Compruebe si algún recurso compartido de archivos de la cuenta de almacenamiento ya está protegido con otro almacén de Recovery Services.

  >[!NOTE]
  >Todos los recursos compartidos de archivos de una cuenta de almacenamiento solo se pueden proteger en un almacén de Recovery Services. Puede usar [este script](scripts/backup-powershell-script-find-recovery-services-vault.md) para encontrar el almacén de Recovery Services donde está registrada la cuenta de almacenamiento.

- Asegúrese de que el recurso compartido de archivos no esté presente en ninguna de las cuentas de almacenamiento no admitidas. Puede consultar la [Matriz de compatibilidad de copia de seguridad de recursos compartidos de archivos de Azure](azure-file-share-support-matrix.md) para encontrar las cuentas de almacenamiento admitidas.
- Asegúrese de que la longitud combinada del nombre de la cuenta de almacenamiento y el nombre del grupo de recursos no supere los 84 caracteres en el caso de nuevas cuentas de almacenamiento y 77 caracteres en caso de cuentas de almacenamiento clásicas. 
- Compruebe la configuración del firewall de la cuenta de almacenamiento para asegurarse de que está habilitada la opción de permitir que los servicios de Microsoft de confianza accedan a la cuenta de almacenamiento.

### <a name="error-in-portal-states-discovery-of-storage-accounts-failed"></a>Error en la detección de estados del portal de las cuentas de almacenamiento

Si tiene una suscripción de asociado (habilitada para CSP), haga caso omiso del error. Si la suscripción no está habilitada para CSP y las cuentas de almacenamiento no se pueden detectar, póngase en contacto con el servicio de soporte técnico.

### <a name="selected-storage-account-validation-or-registration-failed"></a>Error de registro o validación de la cuenta de almacenamiento seleccionada

Vuelva a intentar el registro. Si el problema persiste, póngase en contacto con el soporte técnico.

### <a name="could-not-list-or-find-file-shares-in-the-selected-storage-account"></a>No se pudieron mostrar ni encontrar los recursos compartidos de archivos en la cuenta de almacenamiento seleccionada

- Asegúrese de que la cuenta de almacenamiento existe en el grupo de recursos, y de que no se ha eliminado ni movido después de la última validación o el último registro en el almacén.
- Asegúrese de que el recurso compartido de archivos que desea proteger no se ha eliminado.
- Asegúrese de que la cuenta de almacenamiento es una cuenta de almacenamiento admitida para la copia de seguridad de recursos compartidos de archivos. Puede consultar la [Matriz de compatibilidad de copia de seguridad de recursos compartidos de archivos de Azure](azure-file-share-support-matrix.md) para encontrar las cuentas de almacenamiento admitidas.
- Compruebe si el recurso compartido de archivos ya está protegido en el mismo almacén de Recovery Services.

### <a name="backup-file-share-configuration-or-the-protection-policy-configuration-is-failing"></a>Error en la configuración de copia de seguridad del recurso compartido de archivos (o en la configuración de la directiva de protección)

- Vuelva a intentar la configuración para ver si el problema persiste.
- Asegúrese de que el recurso compartido de archivos que desea proteger no se ha eliminado.
- Si está intentando proteger varios recursos compartidos de archivos a la vez, y algunos de ellos generan error, reintente la configuración de la copia de seguridad en los recursos compartidos de archivos que han dado error.

### <a name="unable-to-delete-the-recovery-services-vault-after-unprotecting-a-file-share"></a>No se puede eliminar el almacén de Recovery Services después de desproteger un recurso compartido de archivos

En Azure Portal, abra su **Almacén** > **Infraestructura de Backup** > **Cuentas de almacenamiento** y haga clic en **Anular el registro** para quitar las cuentas de almacenamiento del almacén de Recovery Services.

>[!NOTE]
>Un almacén de Recovery Services solo se puede eliminar después de anular el registro de todas las cuentas de almacenamiento registradas en el almacén.

## <a name="common-backup-or-restore-errors"></a>Errores de copia de seguridad o restauración comunes

>[!NOTE]
>Consulte [este documento](./backup-rbac-rs-vault.md#minimum-role-requirements-for-the-azure-file-share-backup) para asegurarse de que dispone de los permisos necesarios para realizar operaciones de copia de seguridad o restauración.

### <a name="filesharenotfound--operation-failed-as-the-file-share-is-not-found"></a>FileShareNotFound: no se pudo realizar la operación porque no se encuentra el recurso compartido de archivos

Código de error: FileShareNotFound

Mensaje de error: No se pudo realizar la operación porque no se encuentra el recurso compartido de archivos.

Asegúrese de que el recurso compartido de archivos que intenta proteger no se ha eliminado.

### <a name="usererrorfileshareendpointunreachable--storage-account-not-found-or-not-supported"></a>UserErrorFileShareEndpointUnreachable: la cuenta de almacenamiento no se encuentra o no se admite

Código de error: UserErrorFileShareEndpointUnreachable

Mensaje de error: La cuenta de almacenamiento no se encuentra o no se admite.

- Asegúrese de que la cuenta de almacenamiento existe en el grupo de recursos y de que no se ha eliminado o quitado del grupo de recursos tras la validación final.

- Asegúrese de que la cuenta de almacenamiento es una cuenta de almacenamiento admitida para la copia de seguridad de recursos compartidos de archivos.

### <a name="afsmaxsnapshotreached--you-have-reached-the-max-limit-of-snapshots-for-this-file-share-you-will-be-able-to-take-more-once-the-older-ones-expire"></a>AFSMaxSnapshotReached: ha alcanzado el límite máximo de instantáneas para este recurso compartido de archivos. Podrá tomar más cuando expiren las más antiguas

Código de error: AFSMaxSnapshotReached

Mensaje de error: Ha alcanzado el límite máximo de instantáneas para este recurso compartido de archivos. Podrá tomar más cuando expiren las más antiguas.

- Este error puede producirse al crear varias copias de seguridad a petición para un recurso compartido de archivos.
- Hay un límite de 200 instantáneas por recurso compartido de archivos, incluidas las realizadas por Azure Backup. Las copias de seguridad programadas antiguas (o instantáneas) se borran automáticamente. Las copias de seguridad a petición (o instantáneas) deben eliminarse si se alcanza el límite máximo.

Elimine las copias de seguridad a petición (instantáneas de recurso compartido de archivos de Azure) desde el portal de Azure Files.

>[!NOTE]
> Si elimina las instantáneas creadas por Azure Backup, perderá los puntos de recuperación.

### <a name="usererrorstorageaccountnotfound--operation-failed-as-the-specified-storage-account-does-not-exist-anymore"></a>UserErrorStorageAccountNotFound: no se pudo realizar la operación porque la cuenta de almacenamiento especificada ya no existe

Código de error: UserErrorStorageAccountNotFound

Mensaje de error: No se pudo realizar la operación porque la cuenta de almacenamiento especificada ya no existe.

Asegúrese de que la cuenta de almacenamiento todavía existe y no se ha eliminado.

### <a name="usererrordtsstorageaccountnotfound--the-storage-account-details-provided-are-incorrect"></a>UserErrorDTSStorageAccountNotFound: los detalles de la cuenta de almacenamiento proporcionados son incorrectos

Código de error: UserErrorDTSStorageAccountNotFound

Mensaje de error: Los detalles de cuenta de almacenamiento proporcionados son incorrectos.

Asegúrese de que la cuenta de almacenamiento todavía existe y no se ha eliminado.

### <a name="usererrorresourcegroupnotfound--resource-group-doesnt-exist"></a>UserErrorResourceGroupNotFound: el grupo de recursos no existe

Código de error: UserErrorResourceGroupNotFound

Mensaje de error: El grupo de recursos no existe.

Seleccione un grupo de recursos existente o cree uno nuevo.

### <a name="parallelsnapshotrequest--a-backup-job-is-already-in-progress-for-this-file-share"></a>ParallelSnapshotRequest: ya hay un trabajo de copia de seguridad en curso para el recurso compartido de archivos

Código de error: ParallelSnapshotRequest

Mensaje de error: Ya hay un trabajo de copia de seguridad en curso para el recurso compartido de archivos.

- La copia de seguridad del recurso compartido de archivos no admite solicitudes de instantáneas paralelas en el mismo recurso compartido de archivos.

- Espere a que el trabajo de copia de seguridad existente finalice e inténtelo de nuevo. Si no encuentra un trabajo de copia de seguridad en el almacén de Recovery Services, compruebe otros almacenes de Recovery Services de la misma suscripción.

### <a name="filesharebackupfailedwithazurerprequestthrottling-filesharerestorefailedwithazurerprequestthrottling--file-share-backup-or-restore-failed-due-to-storage-service-throttling-this-may-be-because-the-storage-service-is-busy-processing-other-requests-for-the-given-storage-account"></a>FileshareBackupFailedWithAzureRpRequestThrottling/ FileshareRestoreFailedWithAzureRpRequestThrottling: error de restauración o copia de seguridad de recursos compartidos de archivos debido a la limitación del servicio de almacenamiento. Esto puede ser debido a que el servicio de almacenamiento está ocupado procesando otras solicitudes para la cuenta de almacenamiento dada.

Código de error: FileshareBackupFailedWithAzureRpRequestThrottling/ FileshareRestoreFailedWithAzureRpRequestThrottling

Mensaje de error: Error de restauración o copia de seguridad de recursos compartidos de archivos debido a la limitación del servicio de almacenamiento. Esto puede ser debido a que el servicio de almacenamiento está ocupado procesando otras solicitudes para la cuenta de almacenamiento dada.

Intente realizar la operación de copia de seguridad o restauración en otro momento.

### <a name="targetfilesharenotfound--target-file-share-not-found"></a>TargetFileShareNotFound: no se encontró el recurso compartido de archivos de destino

Código de error: TargetFileShareNotFound

Mensaje de error: No se encontró el recurso compartido de archivos de destino.

- Asegúrese de que existe la cuenta de almacenamiento seleccionada y de que no se ha eliminado el recurso compartido de archivos de destino.

- Asegúrese de que la cuenta de almacenamiento es una cuenta de almacenamiento admitida para la copia de seguridad de recursos compartidos de archivos.

### <a name="usererrorstorageaccountislocked--backup-or-restore-jobs-failed-due-to-storage-account-being-in-locked-state"></a>UserErrorStorageAccountIsLocked:error de los trabajos de copia de seguridad o restauración debido al estado Bloqueado de la cuenta de almacenamiento

Código de error: UserErrorStorageAccountIsLocked

Mensaje de error: Error de los trabajos de copia de seguridad o restauración debido al estado Bloqueado de la cuenta de almacenamiento.

Quite el bloqueo de la cuenta de almacenamiento o use **eliminar bloqueo** en lugar de **leer bloqueo** y reintente la operación de copia de seguridad o restauración.

### <a name="datatransferservicecoflimitreached--recovery-failed-because-number-of-failed-files-are-more-than-the-threshold"></a>DataTransferServiceCoFLimitReached: no se pudo llevar a cabo la recuperación porque el número de archivos con errores es superior al umbral permitido

Código de error: DataTransferServiceCoFLimitReached

Mensaje de error: No se pudo llevar a cabo la recuperación porque el número de archivos con errores es superior al umbral permitido.

- Los motivos de los errores de recuperación se muestran en un archivo (la ruta de acceso se proporciona en los detalles del trabajo). Corrija los errores y vuelva a intentar la operación de restauración solo de los archivos con errores.

- Causas comunes de errores de restauración de archivos:

  - Los archivos con errores están actualmente en uso.
  - Existe un directorio con el mismo nombre que el archivo con errores en el directorio principal.

### <a name="datatransferserviceallfilesfailedtorecover--recovery-failed-as-no-file-could-be-recovered"></a>DataTransferServiceAllFilesFailedToRecover: error en la recuperación porque no se pudo recuperar ningún archivo

Código de error: DataTransferServiceAllFilesFailedToRecover

Mensaje de error: Error de recuperación. No se pudo recuperar ningún archivo.

- Los motivos de los errores de recuperación se muestran en un archivo (la ruta de acceso se proporciona en los detalles del trabajo). Solucione los errores y vuelva a intentar la operación de restauración solo de los archivos con error.

- Causas comunes de errores de restauración de archivos:

  - Los archivos con errores están actualmente en uso.
  - Existe un directorio con el mismo nombre que el archivo con errores en el directorio principal.

### <a name="usererrordtssourceurinotvalid---restore-fails-because-one-of-the-files-in-the-source-does-not-exist"></a>UserErrorDTSSourceUriNotValid: error en la restauración porque uno de los archivos del origen no existe

Código de error: DataTransferServiceSourceUriNotValid

Mensaje de error: Error en la restauración porque uno de los archivos del origen no existe.

- Los elementos seleccionados no están presentes en los datos del punto de recuperación. Para recuperar los archivos, proporcione la lista de archivos correcta.
- La instantánea del recurso compartido de archivos que corresponde al punto de recuperación se elimina manualmente. Seleccione un punto de recuperación diferente y vuelva a intentar la operación de restauración.

### <a name="usererrordtsdestlocked--a-recovery-job-is-in-process-to-the-same-destination"></a>UserErrorDTSDestLocked: ya hay un trabajo de recuperación en curso para el mismo destino

Código de error: UserErrorDTSDestLocked

Mensaje de error: Ya hay un trabajo de recuperación en curso para el mismo destino.

- La copia de seguridad de recursos compartidos de archivos no admite la recuperación en paralelo en el mismo recurso compartido de archivos de destino.

- Espere a que la recuperación existente finalice e inténtelo de nuevo. Si no encuentra un trabajo de recuperación en el almacén de Recovery Services, compruebe otros almacenes de Recovery Services de la misma suscripción.

### <a name="usererrortargetfilesharefull--restore-operation-failed-as-target-file-share-is-full"></a>UserErrorTargetFileShareFull: no se puede realizar la operación de restauración porque el recurso compartido de archivos de destino está lleno

Código de error: UserErrorTargetFileShareFull

Mensaje de error: No se puede realizar la operación de restauración porque el recurso compartido de archivos de destino está lleno.

Aumente la cuota de tamaño del recurso compartido de archivos de destino para albergar los datos de restauración y vuelva a intentar la operación de restauración.

### <a name="usererrortargetfilesharequotanotsufficient--target-file-share-does-not-have-sufficient-storage-size-quota-for-restore"></a>UserErrorTargetFileShareQuotaNotSufficient: no se puede realizar la restauración del recurso compartido de archivos de destino porque la cuota de tamaño de almacenamiento no es suficiente

Código de error: UserErrorTargetFileShareQuotaNotSufficient

Mensaje de error: No se puede realizar la restauración del recurso compartido de archivos de destino porque la cuota de tamaño de almacenamiento no es suficiente.

Aumente la cuota de tamaño del recurso compartido de archivos de destino para albergar los datos de restauración y vuelva a intentar la operación.

### <a name="file-sync-prerestorefailed--restore-operation-failed-as-an-error-occurred-while-performing-pre-restore-operations-on-file-sync-service-resources-associated-with-the-target-file-share"></a>File Sync PreRestoreFailed: la operación de restauración produjo un error porque, a su vez, se produjo un error al realizar las operaciones previas de restauración en los recursos del servicio de sincronización de archivos asociados con el recurso compartido de archivos de destino

Código de error: File Sync PreRestoreFailed

Mensaje de error: La operación de restauración produjo un error porque, a su vez, se produjo un error al realizar las operaciones previas de restauración en los recursos del servicio de sincronización de archivos asociados con el recurso compartido de archivos de destino.

Intente restaurar los datos en otro momento. Si el problema persiste, póngase en contacto con el servicio de soporte técnico de Microsoft.

### <a name="azurefilesyncchangedetectioninprogress--azure-file-sync-service-change-detection-is-in-progress-for-the-target-file-share-the-change-detection-was-triggered-by-a-previous-restore-to-the-target-file-share"></a>AzureFileSyncChangeDetectionInProgress: la detección de cambios en el servicio Azure File Sync está en curso para el recurso compartido de archivos de destino. La detección de cambios se desencadenó mediante una restauración anterior en el recurso compartido de archivos de destino

Código de error: AzureFileSyncChangeDetectionInProgress

Mensaje de error: La detección de cambios en el servicio Azure File Sync está en curso para el recurso compartido de archivos de destino. La detección de cambios se desencadenó mediante una restauración anterior en el recurso compartido de archivos de destino.

Use otro recurso compartido de archivos de destino. Como alternativa, puede esperar a que se complete la detección de cambios en el servicio Azure File Sync para el recurso compartido de archivos de destino antes de volver a intentar la restauración.

### <a name="usererrorafsrecoverysomefilesnotrestored--one-or-more-files-could-not-be-recovered-successfully-for-more-information-check-the-failed-file-list-in-the-path-given-above"></a>UserErrorAFSRecoverySomeFilesNotRestored: uno o varios archivos no se pudieron recuperar correctamente. Para más información, consulte la lista de archivos que han dado error en la ruta de acceso anterior

Código de error: UserErrorAFSRecoverySomeFilesNotRestored

Mensaje de error: Uno o varios archivos no se pudieron recuperar correctamente. Para más información, consulte la lista de archivos que han dado error en la ruta de acceso anterior.

- Los motivos de los errores de recuperación se muestran en un archivo (la ruta de acceso se proporciona en los detalles del trabajo). Corrija los motivos y vuelva a intentar la operación de restauración solo de los archivos con errores.
- Causas comunes de errores de restauración de archivos:

  - Los archivos con errores están actualmente en uso.
  - Existe un directorio con el mismo nombre que el archivo con errores en el directorio principal.

### <a name="usererrorafssourcesnapshotnotfound--azure-file-share-snapshot-corresponding-to-recovery-point-cannot-be-found"></a>UserErrorAFSSourceSnapshotNotFound: no se encuentra la instantánea del recurso compartido de archivos de Azure correspondiente a este punto de recuperación

Código de error: UserErrorAFSSourceSnapshotNotFound

Mensaje de error: No se encuentra la instantánea del recurso compartido de archivos de Azure correspondiente a este punto de recuperación.

- Asegúrese de que la instantánea de recurso compartido de archivos, que se corresponde con el punto de recuperación que está intentando usar para la recuperación, todavía existe.

  >[!NOTE]
  >Si elimina una instantánea de recurso compartido de archivos creada por Azure Backup, los puntos de recuperación correspondientes se volverán inutilizables. Se recomienda no eliminar las instantáneas para garantizar la recuperación.

- Intente seleccionar otro punto de restauración para recuperar los datos.

### <a name="usererroranotherrestoreinprogressonsametarget--another-restore-job-is-in-progress-on-the-same-target-file-share"></a>UserErrorAnotherRestoreInProgressOnSameTarget: hay otro trabajo de restauración en curso en el mismo recurso compartido de archivos de destino

Código de error: UserErrorAnotherRestoreInProgressOnSameTarget

Mensaje de error: Hay otro trabajo de restauración en curso en el mismo recurso compartido de archivos de destino.

Use otro recurso compartido de archivos de destino. Como alternativa, puede cancelar o esperar a que la otra restauración se complete.

## <a name="common-modify-policy-errors"></a>Errores comunes de modificación de directivas

### <a name="bmsusererrorconflictingprotectionoperation--another-configure-protection-operation-is-in-progress-for-this-item"></a>BMSUserErrorConflictingProtectionOperation: hay otra operación de protección de configuración en curso para este elemento

Código de error: BMSUserErrorConflictingProtectionOperation

Mensaje de error: Hay otra operación de protección de configuración en curso para este elemento.

Espere a que termine la operación de modificación de directivas anterior y vuelva a intentarlo en otro momento.

### <a name="bmsusererrorobjectlocked--another-operation-is-in-progress-on-the-selected-item"></a>BMSUserErrorObjectLocked: hay otra operación en curso en el elemento seleccionado

Código de error: BMSUserErrorObjectLocked

Mensaje de error: Hay otra operación en curso en el elemento seleccionado.

Espere a que la otra operación en curso termine y vuelva a intentarlo en otro momento.

## <a name="common-soft-delete-related-errors"></a>Errores comunes relacionados con la eliminación temporal

### <a name="usererrorrestoreafsinsoftdeletestate--this-restore-point-is-not-available-as-the-snapshot-associated-with-this-point-is-in-a-file-share-that-is-in-soft-deleted-state"></a>UserErrorRestoreAFSInSoftDeleteState: este punto de restauración no está disponible porque la instantánea asociada al punto está en un recurso compartido de archivos que está en estado de eliminación temporal

Código de error: UserErrorRestoreAFSInSoftDeleteState

Mensaje de error: Este punto de restauración no está disponible porque la instantánea asociada al punto está en un recurso compartido de archivos que está en estado de eliminación temporal.

No puede realizar una operación de restauración cuando el recurso compartido de archivos se encuentra en estado de eliminación temporal. Restaure el recurso compartido de archivos desde el portal de Files o use el [script Undelete](scripts/backup-powershell-script-undelete-file-share.md) y luego intente realizar la restauración.

### <a name="usererrorrestoreafsindeletestate--listed-restore-points-are-not-available-as-the-associated-file-share-containing-the-restore-point-snapshots-has-been-deleted-permanently"></a>UserErrorRestoreAFSInDeleteState: los puntos de restauración mostrados no están disponibles porque el recurso compartido de archivos asociado que contiene las instantáneas del punto de restauración se ha eliminado permanentemente

Código de error: UserErrorRestoreAFSInDeleteState

Mensaje de error: Los puntos de restauración mostrados no están disponibles porque el recurso compartido de archivos asociado que contiene las instantáneas del punto de restauración se ha eliminado permanentemente.

Compruebe si se ha eliminado el recurso compartido de archivos del que se ha realizado una copia de seguridad. Si se encontraba en un estado de eliminación temporal, compruebe si el período de retención de eliminación temporal es superior y no se ha recuperado. En cualquiera de estos casos, perderá todas las instantáneas de forma permanente y no podrá recuperar los datos.

>[!NOTE]
> Se recomienda no eliminar el recurso compartido de archivos del que se ha realizado una copia de seguridad, o bien, si se encuentra en estado de eliminación temporal, recuperarlo antes de que finalice el período de retención de eliminación temporal, con el fin de evitar que se pierdan todos los puntos de restauración.

### <a name="usererrorbackupafsinsoftdeletestate---backup-failed-as-the-azure-file-share-is-in-soft-deleted-state"></a>UserErrorBackupAFSInSoftDeleteState: no se pudo realizar la copia de seguridad porque el recurso compartido de archivos de Azure está en estado de eliminación temporal

Código de error: UserErrorBackupAFSInSoftDeleteState

Mensaje de error: No se pudo realizar la copia de seguridad porque el recurso compartido de archivos de Azure está en estado de eliminación temporal.

Recupere el recurso compartido de archivos desde el **portal de Files** o mediante el [script Undelete](scripts/backup-powershell-script-undelete-file-share.md) para continuar con la copia de seguridad e impedir la eliminación permanente de los datos.

### <a name="usererrorbackupafsindeletestate--backup-failed-as-the-associated-azure-file-share-is-permanently-deleted"></a>UserErrorBackupAFSInDeleteState: no se pudo realizar la copia de seguridad porque el recurso compartido de archivos asociado de Azure se eliminó permanentemente

Código de error: UserErrorBackupAFSInDeleteState

Mensaje de error: No se pudo realizar la copia de seguridad porque el recurso compartido de archivos asociado de Azure se eliminó permanentemente

Compruebe si se ha eliminado permanentemente el recurso compartido de archivos del que se ha realizado una copia de seguridad. En caso afirmativo, detenga la copia de seguridad de este recurso compartido de archivos para evitar errores repetidos de copia de seguridad. Para obtener información sobre cómo detener la protección, vea [Detención de la protección en un recurso compartido de archivos de Azure](./manage-afs-backup.md#stop-protection-on-a-file-share).

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre la copia de seguridad de recursos compartidos de archivos de Azure, consulte:

- [Copia de seguridad de recursos compartidos de archivos de Azure](backup-afs.md)
- [Preguntas frecuentes acerca de la copia de seguridad de recursos compartidos de archivos de Azure](backup-azure-files-faq.md)
