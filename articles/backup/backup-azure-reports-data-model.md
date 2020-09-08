---
title: Modelo de datos para eventos de diagnóstico de Azure Backup
description: Este modelo de datos es en referencia al modo específico del recurso de enviar eventos de diagnóstico a Log Analytics (LA).
ms.topic: conceptual
ms.date: 10/30/2019
ms.openlocfilehash: 8cc671152485bc2781a80f96e48b81263dea221b
ms.sourcegitcommit: c6b9a46404120ae44c9f3468df14403bcd6686c1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88892531"
---
# <a name="data-model-for-azure-backup-diagnostics-events"></a>Modelo de datos para eventos de diagnóstico de Azure Backup

## <a name="coreazurebackup"></a>CoreAzureBackup

En esta tabla se proporciona información sobre las entidades de copia de seguridad principales, como almacenes y elementos de copia de seguridad.

| **Campo**                         | **Tipo de datos** | **Descripción**                                              |
| --------------------------------- | ------------- | ------------------------------------------------------------ |
| ResourceId                        | Texto          | Identificador de recursos de datos que se recopilan. Por ejemplo, el id. de recurso del almacén de Recovery Services. |
| OperationName                     | Texto          | Este campo representa el nombre de la operación actual: BackupItem, BackupItemAssociation, o ProtectedContainer. |
| Category                          | Texto          | Este campo representa la categoría de datos de diagnóstico insertados en los registros de Azure Monitor. Por ejemplo, CoreAzureBackup. |
| AgentVersion                      | Texto          | Número de versión del agente de copia de seguridad o del agente de protección (en el caso de SC DPM y MABS). |
| AzureBackupAgentVersion           | Texto          | Versión de Azure Backup Agent en el servidor de administración de copia de seguridad. |
| AzureDataCenter                   | Texto          | Centro de datos donde se encuentra el almacén.                       |
| BackupItemAppVersion              | Texto          | Versión de la aplicación del elemento de copia de seguridad                       |
| BackupItemFriendlyName            | Texto          | Nombre descriptivo del elemento de copia de seguridad.                             |
| BackupItemName                    | Texto          | Nombre del elemento de copia de seguridad.                                      |
| BackupItemProtectionState         | Texto          | Estado de protección del elemento de copia de seguridad                          |
| BackupItemFrontEndSize            | Texto          | Tamaño de front-end del elemento de copia de seguridad.                            |
| BackupItemType                    | Texto          | Tipo de elemento de copia de seguridad. Por ejemplo: VM, FileFolder             |
| BackupItemUniqueId                | Texto          | Identificador único del elemento de copia de seguridad                         |
| BackupManagementServerType        | Texto          | Tipo del servidor de administración de copia de seguridad, como MABS o SC DPM.     |
| BackupManagementServerUniqueId    | Texto          | Campo para identificar de forma única el servidor de administración de copia de seguridad.      |
| BackupManagementType              | Texto          | Tipo de proveedor para el servidor que realiza el trabajo de copia de seguridad. Por ejemplo, IaaSVM o FileFolder |
| BackupManagementServerName        | Texto          | Nombre del servidor de administración de copia de seguridad.                         |
| BackupManagementServerOSVersion   | Texto          | Versión del sistema operativo del servidor de administración de copia de seguridad.                   |
| BackupManagementServerVersion     | Texto          | Versión del servidor de administración de copia de seguridad.                      |
| LatestRecoveryPointLocation       | Texto          | Ubicación del punto de recuperación más reciente para el elemento de copia de seguridad.    |
| LatestRecoveryPointTime           | DateTime      | Fecha y hora del último punto de recuperación para el elemento de copia de seguridad.   |
| OldestRecoveryPointLocation       | Texto          | Ubicación del punto de recuperación más antiguo para el elemento de copia de seguridad.    |
| OldestRecoveryPointTime           | DateTime      | Fecha y hora del último punto de recuperación para el elemento de copia de seguridad.   |
| PolicyUniqueId                    | Texto          | Identificador único que identifica la directiva.                             |
| ProtectedContainerFriendlyName    | Texto          | Nombre descriptivo del servidor protegido.                        |
| ProtectedContainerLocation        | Texto          | Si el contenedor protegido está ubicado en el entorno local o en Azure. |
| ProtectedContainerName            | Texto          | Nombre del contenedor protegido.                            |
| ProtectedContainerOSType          | Texto          | Tipo de sistema operativo del contenedor protegido.                          |
| ProtectedContainerOSVersion       | Texto          | Versión del sistema operativo del contenedor protegido.                        |
| ProtectedContainerProtectionState | Texto          | Estado de protección del contenedor protegido                  |
| ProtectedContainerType            | Texto          | Si el contenedor protegido es un servidor o un contenedor.  |
| ProtectedContainerUniqueId        | Texto          | Identificador único del contenedor protegido para todo excepto las máquinas virtuales copiadas con DPM o MABS. |
| ProtectedContainerWorkloadType    | Texto          | Tipo de la copia de seguridad del contenedor protegido. Por ejemplo, IaaSVMContainer |
| ProtectionGroupName               | Texto          | Nombre del grupo de protección en el que está protegido el elemento de copia de seguridad, para SC DPM y MABS, si procede. |
| ResourceGroupName                 | Texto          | Grupo de recursos correspondiente al recurso (por ejemplo, el almacén de Recovery Services) de los datos que se recopilan. |
| SchemaVersion                     | Texto          | Este campo indica la versión actual del esquema. Es **V2**. |
| SecondaryBackupProtectionState    | Texto          | Si está habilitada la protección secundaria para el elemento de copia de seguridad.  |
| State                             | Texto          | Estado del objeto del elemento de copia de seguridad. Por ejemplo, Activo o Eliminado |
| StorageReplicationType            | Texto          | Tipo de replicación de almacenamiento para el almacén. Por ejemplo, GeoRedundant |
| SubscriptionId                    | Texto          | Identificador de suscripción del recurso (por ejemplo, almacén de Recovery Services) para el que se recopilan los datos. |
| VaultName                         | Texto          | Nombre del almacén                                            |
| VaultTags                         | Texto          | Etiquetas asociadas al recurso de almacén.                    |
| VaultUniqueId                     | Texto          | Identificador único del almacén.                             |
| SourceSystem                      | Texto          | Sistema de origen de los datos actuales: Azure.                  |

## <a name="addonazurebackupalerts"></a>AddonAzureBackupAlerts

Esta tabla proporciona detalles acerca de los campos relacionados con la alerta.

| **Campo**                      | **Tipo de datos** | **Descripción**                                              |
| :----------------------------- | ------------- | ------------------------------------------------------------ |
| ResourceId                     | Texto          | Identificador único para el recurso sobre el que se recopilan los datos. Por ejemplo, el id. de recurso del almacén de Recovery Services. |
| OperationName                  | Texto          | Nombre de la operación actual. Por ejemplo, Alerta            |
| Category                       | Texto          | Categoría de datos de diagnóstico insertados en los registros de Azure Monitor: AddonAzureBackupAlerts. |
| AlertCode                      | Texto          | Código para identificar de forma única un tipo de alerta.                     |
| AlertConsolidationStatus       | Texto          | Identifica si la alerta es o no una alerta consolidada.         |
| AlertOccurrenceDateTime        | DateTime      | Fecha y hora en que se creó la alerta.                     |
| AlertRaisedOn                  | Texto          | Tipo de entidad en la que se genera la alerta.                        |
| AlertSeverity                  | Texto          | Gravedad de la alerta. Por ejemplo, Critical                 |
| AlertStatus                    | Texto          | Estado de la alerta. Por ejemplo, Active                     |
| AlertTimeToResolveInMinutes    | Number        | Tiempo necesario para resolver una alerta. En blanco para las alertas activas.     |
| AlertType                      | Texto          | Tipo de alerta. Por ejemplo, Backup                           |
| AlertUniqueId                  | Texto          | Identificador único de la alerta generada.                    |
| BackupItemUniqueId             | Texto          | Identificador único del elemento de copia de seguridad asociado con la alerta. |
| BackupManagementServerUniqueId | Texto          | Campo para identificar de forma única el servidor de administración de copia de seguridad mediante el que se protege el elemento de copia de seguridad, si procede. |
| BackupManagementType           | Texto          | Tipo de proveedor para el servidor que realiza el trabajo de copia de seguridad, por ejemplo, IaaSVM o FileFolder. |
| CountOfAlertsConsolidated      | Number        | Número de alertas consolidadas si se trata de una alerta consolidada.  |
| ProtectedContainerUniqueId     | Texto          | Identificador único del servidor protegido asociado a la alerta. |
| RecommendedAction              | Texto          | Acción recomendada para resolver la alerta.                      |
| SchemaVersion                  | Texto          | Versión actual del esquema, por ejemplo **V2**.            |
| State                          | Texto          | Estado actual del objeto de la alerta por ejemplo, Active o Deleted |
| StorageUniqueId                | Texto          | Identificador único de la entidad de almacenamiento.                |
| VaultUniqueId                  | Texto          | Id. exclusivo usado para identificar el almacén relacionado con la alerta.    |
| SourceSystem                   | Texto          | Sistema de origen de los datos actuales: Azure                    |

## <a name="addonazurebackupprotectedinstance"></a>AddonAzureBackupProtectedInstance

En esta tabla se proporcionan los campos relacionados con las instancias protegidas básicas.

| **Campo**                      | **Tipo de datos** | **Descripción**                                              |
| ------------------------------ | ------------- | ------------------------------------------------------------ |
| ResourceId                     | Texto          | Id. exclusivo para el recurso sobre el que se recopilan los datos. Por ejemplo, el identificador del recurso del almacén de Recovery Services. |
| OperationName                  | Texto          | Nombre de la operación, por ejemplo, ProtectedInstance.         |
| Category                       | Texto          | Categoría de datos de diagnóstico insertados en los registros de Azure Monitor: AddonAzureBackupProtectedInstance. |
| BackupItemUniqueId             | Texto          | Identificador exclusivo del elemento de copia de seguridad.                                 |
| BackupManagementServerUniqueId | Texto          | Campo para identificar de forma única el servidor de administración de copia de seguridad mediante el que se protege el elemento de copia de seguridad, si procede. |
| BackupManagementType           | Texto          | Tipo de proveedor para el servidor que realiza el trabajo de copia de seguridad, por ejemplo, IaaSVM o FileFolder. |
| ProtectedContainerUniqueId     | Texto          | Id. exclusivo para identificar el contenedor protegido donde se ejecuta el trabajo. |
| ProtectedInstanceCount         | Texto          | Recuento de instancias protegidas del elemento de copia de seguridad o del contenedor protegido asociados en esa fecha y hora. |
| SchemaVersion                  | Texto          | Versión actual del esquema, por ejemplo **V2**.            |
| State                          | Texto          | Estado del objeto del elemento de copia de seguridad, por ejemplo, Active o Deleted. |
| VaultUniqueId                  | Texto          | Id. exclusivo del almacén protegido asociado a la instancia protegida. |
| SourceSystem                   | Texto          | Sistema de origen de los datos actuales: Azure                    |

## <a name="addonazurebackupjobs"></a>AddonAzureBackupJobs

Esta tabla proporciona detalles acerca de los campos relacionados con los trabajos.

| **Campo**                      | **Tipo de datos** | **Descripción**                                              |
| ------------------------------ | ------------- | ------------------------------------------------------------ |
| ResourceId                     | Texto          | Identificador de recursos de datos que se recopilan. Por ejemplo, el identificador del recurso del almacén de Recovery Services. |
| OperationName                  | Texto          | Este campo representa el nombre de la operación actual: Job    |
| Category                       | Texto          | Este campo representa la categoría de datos de diagnóstico insertados en los registros de Azure Monitor (AddonAzureBackupJobs) |
| AdhocOrScheduledJob            | Texto          | Campo para especificar si el trabajo es ad hoc o programado.           |
| BackupItemUniqueId             | Texto          | Identificador único del elemento de copia de seguridad relacionado con la entidad de almacenamiento. |
| BackupManagementServerUniqueId | Texto          | Identificador único del servidor de administración de copia de seguridad relacionado con la entidad de almacenamiento. |
| BackupManagementType           | Texto          | Tipo de proveedor para realizar la copia de seguridad, por ejemplo, IaaSVM o FileFolder al que pertenece este trabajo |
| DataTransferredInMB            | Number        | Datos transferidos en MB para este trabajo                          |
| JobDurationInSecs              | Number        | Duración del trabajo total en segundos                                |
| JobFailureCode                 | Texto          | Cadena del código de error por el que produjo el error del trabajo    |
| JobOperation                   | Texto          | Operación para la que se ejecuta el trabajo, por ejemplo, Backup, Restore o Configure Backup. |
| JobOperationSubType            | Texto          | Subtipo de la operación de trabajo. Por ejemplo, "Log", en el caso del trabajo de copia de seguridad de registros. |
| JobStartDateTime               | DateTime      | Fecha y hora en las que comenzó la ejecución del trabajo                       |
| Estado del trabajo                      | Texto          | Estado del trabajo terminado, por ejemplo, Completed o Failed   |
| JobUniqueId                    | Texto          | Identificador único que identifica el trabajo.                                |
| ProtectedContainerUniqueId     | Texto          | Identificador único del servidor protegido asociado al trabajo |
| RecoveryJobDestination         | Texto          | Destino de un trabajo de recuperación, donde se recuperan los datos.   |
| RecoveryJobRPDateTime          | DateTime      | La fecha y hora en que se creó el punto de recuperación que se va a recuperar. |
| RecoveryJobLocation            | Texto          | La ubicación donde se almacenó el punto de recuperación que se va a recuperar. |
| RecoveryLocationType           | Texto          | Tipo de la ubicación de recuperación.                                |
| SchemaVersion                  | Texto          | Versión actual del esquema, por ejemplo **V2**.            |
| State                          | Texto          | Estado actual del objeto del trabajo, por ejemplo, Active o Deleted |
| VaultUniqueId                  | Texto          | Identificador único del almacén protegido asociado al trabajo |
| SourceSystem                   | Texto          | Sistema de origen de los datos actuales: Azure                    |

## <a name="addonazurebackuppolicy"></a>AddonAzureBackupPolicy

Esta tabla proporciona detalles acerca de los campos relacionados con las directivas.

| **Campo**                       | **Tipo de datos**  | **Descripción**                                              |
| ------------------------------- | -------------- | ------------------------------------------------------------ |
| ResourceId                      | Texto           | Id. exclusivo para el recurso sobre el que se recopilan los datos. Por ejemplo, el identificador del recurso del almacén de Recovery Services. |
| OperationName                   | Texto           | Nombre de la operación, por ejemplo, Policy o PolicyAssociation. |
| Category                        | Texto           | Categoría de datos de diagnóstico insertados en los registros de Azure Monitor: AddonAzureBackupPolicy. |
| BackupDaysOfTheWeek             | Texto           | Días de la semana en los que se han programado copias de seguridad            |
| BackupFrequency                 | Texto           | Frecuencia con la que se ejecutan las copias de seguridad. Por ejemplo, daily o weekly. |
| BackupManagementType            | Texto           | Tipo de proveedor para el servidor que realiza el trabajo de copia de seguridad. Por ejemplo, IaaSVM o FileFolder. |
| BackupManagementServerUniqueId  | Texto           | Campo para identificar de forma única el servidor de administración de copia de seguridad mediante el que se protege el elemento de copia de seguridad, si procede. |
| BackupTimes                     | Texto           | Fecha y hora en que se programan las copias de seguridad                     |
| DailyRetentionDuration          | Whole Number   | Duración de retención total de las copias de seguridad configuradas, en días      |
| DailyRetentionTimes             | Texto           | Fecha y hora en que se configuró la retención diaria            |
| DiffBackupDaysOfTheWeek         | Texto           | Días de la semana de las copias de seguridad diferenciales de SQL en la copia de seguridad de máquinas virtuales de Azure. |
| DiffBackupFormat                | Texto           | Formato para copias de seguridad diferenciales de SQL en la copia de seguridad de máquinas virtuales de Azure.   |
| DiffBackupRetentionDuration     | Decimal Number | Duración de retención de las copias de seguridad diferenciales de SQL en la copia de seguridad de máquinas virtuales de Azure. |
| DiffBackupTime                  | Time           | Tiempo para copias de seguridad diferenciales de SQL en la copia de seguridad de máquinas virtuales de Azure.     |
| LogBackupFrequency              | Decimal Number | Frecuencia de las copias de seguridad de registros de SQL.                            |
| LogBackupRetentionDuration      | Decimal Number | Duración de retención de las copias de seguridad de registro de SQL en la copia de seguridad de máquinas virtuales de Azure. |
| MonthlyRetentionDaysOfTheMonth  | Texto           | Semanas del mes en que se configura la retención mensual.  Por ejemplo, la primera o la última. |
| MonthlyRetentionDaysOfTheWeek   | Texto           | Días de la semana seleccionados para la retención mensual              |
| MonthlyRetentionDuration        | Texto           | Duración de retención total de las copias de seguridad configuradas, en meses    |
| MonthlyRetentionFormat          | Texto           | Tipo de configuración para la retención mensual. Por ejemplo, daily si se hace por días o weekly si se hace por semanas. |
| MonthlyRetentionTimes           | Texto           | Fecha y hora en que se ha configurado la retención mensual           |
| MonthlyRetentionWeeksOfTheMonth | Texto           | Semanas del mes en que se configura la retención mensual.   Por ejemplo, la primera o la última. |
| PolicyName                      | Texto           | Nombre de la directiva definida                                   |
| PolicyUniqueId                  | Texto           | Identificador único que identifica la directiva.                             |
| PolicyTimeZone                  | Texto           | Zona horaria en la que se especifican los campos de hora de directiva en los registros. |
| RetentionDuration               | Texto           | Duración de retención de las copias de seguridad configuradas                    |
| RetentionType                   | Texto           | Tipo de retención.                                            |
| SchemaVersion                   | Texto           | Este campo indica la versión actual del esquema; es **V2**. |
| State                           | Texto           | Estado actual del objeto de directiva. Por ejemplo, Active o Deleted. |
| SynchronisationFrequencyPerDay  | Whole Number   | Número de veces en un día que se sincroniza una copia de seguridad de archivos para SC DPM y MABS. |
| VaultUniqueId                   | Texto           | Id. exclusivo del almacén al que pertenece esta directiva.          |
| WeeklyRetentionDaysOfTheWeek    | Texto           | Días de la semana seleccionados para la retención semanal               |
| WeeklyRetentionDuration         | Decimal Number | Duración total de la retención semanal de las copias de seguridad configuradas, en semanas. |
| WeeklyRetentionTimes            | Texto           | Fecha y hora en que se ha configurado la retención semanal            |
| YearlyRetentionDaysOfTheMonth   | Texto           | Fechas del mes seleccionadas para la retención anual             |
| YearlyRetentionDaysOfTheWeek    | Texto           | Días de la semana seleccionados para la retención anual               |
| YearlyRetentionDuration         | Decimal Number | Duración total de la retención de las copias de seguridad configuradas, en años     |
| YearlyRetentionFormat           | Texto           | Tipo de configuración para la retención anual, por ejemplo, daily si se basa en día, weekly si se basa en semana. |
| YearlyRetentionMonthsOfTheYear  | Texto           | Meses del año seleccionados para la retención anual             |
| YearlyRetentionTimes            | Texto           | Fecha y hora en que se ha configurado la retención anual            |
| YearlyRetentionWeeksOfTheMonth  | Texto           | Semanas del mes seleccionadas para la retención anual.             |
| SourceSystem                    | Texto           | Sistema de origen de los datos actuales: Azure                    |

## <a name="addonazurebackupstorage"></a>AddonAzureBackupStorage

Esta tabla proporciona detalles acerca de los campos relacionados con el almacenamiento.

| **Campo**                      | **Tipo de datos** | **Descripción**                                              |
| ------------------------------ | ------------- | ------------------------------------------------------------ |
| ResourceId                     | Texto          | Identificador de recursos de datos que se recopilan. Por ejemplo, el id. de recurso del almacén de Recovery Services. |
| OperationName                  | Texto          | Este campo representa el nombre de la operación actual: Storage o StorageAssociation. |
| Category                       | Texto          | Este campo representa la categoría de datos de diagnóstico insertados en los registros de Azure Monitor; es AddonAzureBackupStorage. |
| BackupItemUniqueId             | Texto          | Id. exclusivo del elemento de copia de seguridad de las máquinas virtuales cuya copia de seguridad se ha realizado con DPM o MABS. |
| BackupManagementServerUniqueId | Texto          | Campo para identificar de forma única el servidor de administración de copia de seguridad mediante el que se protege el elemento de copia de seguridad, si procede. |
| BackupManagementType           | Texto          | Tipo de proveedor para el servidor que realiza el trabajo de copia de seguridad. Por ejemplo, IaaSVM o FileFolder. |
| PreferredWorkloadOnVolume      | Texto          | Carga de trabajo para la que este volumen es el almacenamiento preferido.      |
| ProtectedContainerUniqueId     | Texto          | Identificador único del contenedor protegido asociado al elemento de copia de seguridad. |
| SchemaVersion                  | Texto          | Versión del esquema. Por ejemplo, **V2**.                   |
| State                          | Texto          | Estado del objeto del elemento de copia de seguridad. Por ejemplo, Active o Deleted. |
| StorageAllocatedInMBs          | Number        | Tamaño de almacenamiento asignado por el elemento de copia de seguridad correspondiente en el almacenamiento correspondiente del tipo de disco. |
| StorageConsumedInMBs           | Number        | Tamaño de almacenamiento consumido por el elemento de copia de seguridad correspondiente en el almacenamiento correspondiente. |
| StorageName                    | Texto          | Nombre de la entidad de almacenamiento. Por ejemplo, E:\.                      |
| StorageTotalSizeInGBs          | Texto          | Tamaño total de almacenamiento, en GB, consumido por la entidad de almacenamiento     |
| StorageType                    | Texto          | Tipo de almacenamiento, por ejemplo, Nube, Volumen o Disco             |
| StorageUniqueId                | Texto          | Identificador único de la entidad de almacenamiento.                |
| VaultUniqueId                  | Texto          | Id. exclusivo del almacén relacionado con la entidad de almacenamiento. |
| VolumeFriendlyName             | Texto          | Nombre descriptivo del volumen de almacenamiento.                          |
| SourceSystem                   | Texto          | Sistema de origen de los datos actuales: Azure                    |

## <a name="next-steps"></a>Pasos siguientes

- [Más información sobre cómo enviar datos a Log Analytics](./backup-azure-diagnostic-events.md)
- [Más información sobre cómo escribir consultas en las tablas específicas del recurso](./backup-azure-monitoring-use-azuremonitor.md#sample-kusto-queries)
