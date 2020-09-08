---
title: Modelo de datos de registros de Azure Monitor
description: En este artículo, obtendrá información acerca de los detalles del modelo de datos de Log Analytics de Azure Monitor para los datos de Azure Backup.
ms.topic: conceptual
ms.date: 02/26/2019
ms.openlocfilehash: 7822f88c9ea3e0cd83b7e600d63984a8a51becb1
ms.sourcegitcommit: c6b9a46404120ae44c9f3468df14403bcd6686c1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88890270"
---
# <a name="log-analytics-data-model-for-azure-backup-data"></a>Modelo de datos de Log Analytics para datos de Azure Backup

Use el modelo de datos de Log Analytics para crear alertas personalizadas desde Log Analytics.

[!INCLUDE [azure-monitor-log-analytics-rebrand](../../includes/azure-monitor-log-analytics-rebrand.md)]

> [!NOTE]
>
> Este modelo de datos hace referencia al modo Azure Diagnostics para el envío de eventos de diagnóstico a Log Analytics (LA). Para obtener información sobre el modelo de datos para el nuevo modo específico del recurso, puede consultar el siguiente artículo: [Modelo de datos para eventos de diagnóstico de Azure Backup](./backup-azure-reports-data-model.md)

## <a name="using-azure-backup-data-model"></a>Uso del modelo de datos de Azure Backup

Puede usar los siguientes campos que se proporcionan como parte del modelo de datos para crear objetos visuales, consultas personalizadas y paneles, según sus requisitos.

### <a name="alert"></a>Alerta

Esta tabla proporciona detalles acerca de los campos relacionados con la alerta.

| Campo | Tipo de datos | Descripción |
| --- | --- | --- |
| AlertUniqueId_s |Texto |Identificador único de la alerta generada |
| AlertType_s |Texto |Tipo de alerta, por ejemplo, Copia de seguridad |
| AlertStatus_s |Texto |Estado de la alerta, por ejemplo, Activa |
| AlertOccurrenceDateTime_s |Fecha y hora |Fecha y hora en que se creó la alerta |
| AlertSeverity_s |Texto |Gravedad de la alerta, por ejemplo, Crítica |
|AlertTimeToResolveInMinutes_s    | Number        |Tiempo necesario para resolver una alerta. En blanco para las alertas activas.         |
|AlertConsolidationStatus_s   |Texto         |Identifica si la alerta es o no una alerta consolidada.         |
|CountOfAlertsConsolidated_s     |Number         |Número de alertas consolidadas si se trata de una alerta consolidada.          |
|AlertRaisedOn_s     |Texto         |Tipo de entidad en la que se genera la alerta.         |
|AlertCode_s     |Texto         |Código para identificar de forma única un tipo de alerta.         |
|RecommendedAction_s   |Texto         |Acción recomendada para resolver la alerta.         |
| EventName_s |Texto |Nombre del evento. Siempre AzureBackupCentralReport |
| BackupItemUniqueId_s |Texto |Identificador único del elemento de copia de seguridad asociado con la alerta |
| SchemaVersion_s |Texto |Versión actual del esquema, por ejemplo **V2**. |
| State_s |Texto |Estado actual del objeto de la alerta por ejemplo, Active o Deleted |
| BackupManagementType_s |Texto |Tipo de proveedor para realizar la copia de seguridad, por ejemplo, IaaSVM o FileFolder al que pertenece esta alerta. |
| OperationName |Texto |Nombre de la operación actual, por ejemplo, Alerta |
| Category |Texto |Categoría de datos de diagnóstico insertados en los registros de Azure Monitor. Siempre AzureBackupReport |
| Recurso |Texto |Este es el recurso para el que se están recopilando datos; muestra el nombre del almacén de Recovery Services |
| ProtectedContainerUniqueId_s |Texto |Identificador único del servidor protegido asociado a la alerta (fue ProtectedServerUniqueId_s en V1)|
| VaultUniqueId_s |Texto |Identificador único del almacén protegido asociado a la alerta |
| SourceSystem |Texto |Sistema de origen de los datos actuales: Azure |
| ResourceId |Texto |Identificador único para el recurso sobre el que se recopilan los datos. Por ejemplo, el identificador del recurso del almacén de Recovery Services. |
| SubscriptionId |Texto |Identificador de la suscripción del recurso (p. ej., Almacén de Recovery Services) para el que se recopilan los datos |
| ResourceGroup |Texto |Grupo de recursos del recurso (p. ej., Almacén de Recovery Services) para el que se recopilan los datos |
| ResourceProvider |Texto |Proveedor de recursos para el que se recopilan los datos. Por ejemplo, Microsoft.RecoveryServices. |
| ResourceType |Texto |Tipo de recurso para el que se recopilan los datos. Por ejemplo: almacenes |

### <a name="backupitem"></a>BackupItem

Esta tabla proporciona detalles acerca de los campos relacionados con el elemento de copia de seguridad.

| Campo | Tipo de datos | Descripción |
| --- | --- | --- |
| EventName_s |Texto |Nombre del evento. Siempre AzureBackupCentralReport |  
| BackupItemUniqueId_s |Texto |Identificador único del elemento de copia de seguridad |
| BackupItemId_s |Texto |Identificador del elemento de copia de seguridad (este campo solo es para el esquema v1) |
| BackupItemName_s |Texto |Nombre de elemento de copia de seguridad |
| BackupItemFriendlyName_s |Texto |Nombre descriptivo del elemento de copia de seguridad |
| BackupItemType_s |Texto |Tipo de elemento de copia de seguridad, por ejemplo, VM o FileFolder |
| BackupItemProtectionState_s |Texto |Estado de protección del elemento de copia de seguridad |
| BackupItemAppVersion_s |Texto |Versión de la aplicación del elemento de copia de seguridad |
| ProtectionState_s |Texto |Estado actual de protección del elemento copia de seguridad, por ejemplo, Protected o ProtectionStopped |
| ProtectionGroupName_s |Texto | Nombre del grupo de protección en el que está protegido el elemento de copia de seguridad, para SC DPM y MABS, si procede.|
| SecondaryBackupProtectionState_s |Texto |Si está habilitada la protección secundaria para el elemento de copia de seguridad.|
| SchemaVersion_s |Texto |Versión del esquema, por ejemplo, **V2**. |
| State_s |Texto |Estado del objeto del elemento de copia de seguridad, por ejemplo, Active o Deleted |
| BackupManagementType_s |Texto |Tipo de proveedor para realizar la copia de seguridad, por ejemplo, IaaSVM o FileFolder al que pertenece este elemento de copia de seguridad. |
| OperationName |Texto |Nombre de la operación, por ejemplo, BackupItem |
| Category |Texto |Categoría de datos de diagnóstico insertados en los registros de Azure Monitor. Siempre AzureBackupReport |
| Recurso |Texto |Recurso para el que se recopilan datos, por ejemplo, nombre de almacén de Recovery Services |
| SourceSystem |Texto |Sistema de origen de los datos actuales: Azure |
| ResourceId |Texto |Identificador de recurso para el que se recopilan datos, por ejemplo, el identificador de recurso del almacén de Recovery Services. |
| SubscriptionId |Texto |Identificador de la suscripción del recurso (p. ej., Almacén de Recovery Services) para los datos que se recopilan |
| ResourceGroup |Texto |Grupo de recursos del recurso (p. ej., Almacén de Recovery Services) para los datos que se recopilan |
| ResourceProvider |Texto |Proveedor de recursos de datos que se recopilan, por ejemplo, Microsoft.RecoveryServices |
| ResourceType |Texto |Tipo del recurso de datos que se recopilan, por ejemplo, los almacenes. |

### <a name="backupitemassociation"></a>BackupItemAssociation

Esta tabla proporciona detalles acerca de las asociaciones de elementos de copia de seguridad con varias entidades.

| Campo | Tipo de datos | Descripción |
| --- | --- | --- |
| EventName_s |Texto |Este campo representa el nombre de este evento. Siempre es AzureBackupCentralReport. |  
| BackupItemUniqueId_s |Texto |Identificador exclusivo del elemento de copia de seguridad. |
| SchemaVersion_s |Texto |Este campo indica la versión actual del esquema. Es **V2**. |
| State_s |Texto |Estado actual del objeto de la asociación del elemento de copia de seguridad, por ejemplo, Active o Deleted |
| BackupManagementType_s |Texto |Tipo de proveedor para el servidor que realiza el trabajo de copia de seguridad, por ejemplo, IaaSVM o FileFolder. |
| BackupItemSourceSize_s |Texto | Tamaño de front-end del elemento de copia de seguridad. |
| BackupManagementServerUniqueId_s |Texto | Campo para identificar de forma única el servidor de administración de copia de seguridad mediante el que se protege el elemento de copia de seguridad, si procede. |
| Category |Texto |Este campo representa la categoría de datos de diagnóstico insertados en Log Analytics. Es AzureBackupReport. |
| OperationName |Texto |Este campo representa el nombre de la operación actual: BackupItemAssociation |
| Resource |Texto |Este es el recurso para el que se están recopilando datos; muestra el nombre del almacén de Recovery Services |
| ProtectedContainerUniqueId_s |Texto |Identificador único del servidor protegido asociado al elemento de copia de seguridad (fue ProtectedServerUniqueId_s en V1) |
| VaultUniqueId_s |Texto |Identificador único del almacén que contiene el elemento de copia de seguridad. |
| SourceSystem |Texto |Sistema de origen de los datos actuales: Azure |
| ResourceId |Texto |Identificador de recursos de datos que se recopilan. Por ejemplo, el identificador del recurso del almacén de Recovery Services. |
| SubscriptionId |Texto |Identificador de la suscripción del recurso (p. ej., Almacén de Recovery Services) para el que se recopilan los datos. |
| ResourceGroup |Texto |Grupo de recursos del recurso (p. ej., Almacén de Recovery Services) para el que se recopilan los datos. |
| ResourceProvider |Texto |Proveedor de recursos de datos que se recopilan, por ejemplo, Microsoft.RecoveryServices |
| ResourceType |Texto |Tipo del recurso de datos que se recopilan, por ejemplo, los almacenes. |

### <a name="backupmanagementserver"></a>BackupManagementServer

Esta tabla proporciona detalles acerca de las asociaciones de elementos de copia de seguridad con varias entidades.

| Campo | Tipo de datos | Descripción |
| --- | --- | --- |
|BackupManagementServerName_s     |Texto         |Nombre del servidor de administración de copia de seguridad.        |
|AzureBackupAgentVersion_s     |Texto         |Versión de Azure Backup Agent en el servidor de administración de copia de seguridad.          |
|BackupManagementServerVersion_s     |Texto         |Versión del servidor de administración de copia de seguridad.|
|BackupManagementServerOSVersion_s    |Texto            |Versión del sistema operativo del servidor de administración de copia de seguridad.|
|BackupManagementServerType_s     |Texto         |Tipo del servidor de administración de copia de seguridad, como MABS o SC DPM.|
|BackupManagementServerUniqueId_s     |Texto         |Campo para identificar de forma única el servidor de administración de copia de seguridad.       |
| SourceSystem |Texto |Sistema de origen de los datos actuales: Azure |
| ResourceId |Texto |Identificador de recursos de datos que se recopilan. Por ejemplo, el identificador del recurso del almacén de Recovery Services. |
| SubscriptionId |Texto |Identificador de la suscripción del recurso (p. ej., Almacén de Recovery Services) para el que se recopilan los datos. |
| ResourceGroup |Texto |Grupo de recursos del recurso (p. ej., Almacén de Recovery Services) para el que se recopilan los datos. |
| ResourceProvider |Texto |Proveedor de recursos de datos que se recopilan, por ejemplo, Microsoft.RecoveryServices |
| ResourceType |Texto |Tipo del recurso de datos que se recopilan, por ejemplo, los almacenes. |

### <a name="job"></a>Trabajo

Esta tabla proporciona detalles acerca de los campos relacionados con los trabajos.

| Campo | Tipo de datos | Descripción |
| --- | --- | --- |
| EventName_s |Texto |Nombre del evento. Siempre AzureBackupCentralReport |
| BackupItemUniqueId_s |Texto |Identificador único del elemento de copia de seguridad |
| SchemaVersion_s |Texto |Versión del esquema, por ejemplo, **V2**. |
| State_s |Texto |Estado actual del objeto del trabajo, por ejemplo, Active o Deleted |
| BackupManagementType_s |Texto |Tipo de proveedor para el servidor que realiza el trabajo de copia de seguridad, por ejemplo, IaaSVM o FileFolder. |
| OperationName |Texto |Este campo representa el nombre de la operación actual: Job |
| Category |Texto |Este campo representa la categoría de datos de diagnóstico insertados en los registros de Azure Monitor. Es AzureBackupReport. |
| Resource |Texto |Este es el recurso para el que se están recopilando datos; muestra el nombre del almacén de Recovery Services |
| ProtectedServerUniqueId_s |Texto |Identificador único del servidor protegido asociado al trabajo. |
| ProtectedContainerUniqueId_s |Texto | Identificador único del contenedor protegido donde se ejecuta el trabajo. |
| VaultUniqueId_s |Texto |Identificador único del almacén protegido. |
| JobOperation_s |Texto |Operación para la que se ejecuta el trabajo, por ejemplo, Backup, Restore o Configure Backup |
| JobStatus_s |Texto |Estado del trabajo terminado, por ejemplo, Completed o Failed |
| JobFailureCode_s |Texto |Cadena del código de error por el que produjo el error del trabajo |
| JobStartDateTime_s |Fecha y hora |Fecha y hora en las que comenzó la ejecución del trabajo |
| BackupStorageDestination_s |Texto |Destino de almacenamiento de almacenamiento de copias de seguridad, por ejemplo, Cloud o Disk  |
| AdHocOrScheduledJob_s |Texto | Campo para especificar si el trabajo es ad hoc o programado. |
| JobDurationInSecs_s | Number |Duración del trabajo total en segundos |
| DataTransferredInMB_s | Number |Datos transferidos en MB para este trabajo|
| JobUniqueId_g |Texto |Identificador único que identifica el trabajo. |
| RecoveryJobDestination_s |Texto | Destino de un trabajo de recuperación, donde se recuperan los datos. |
| RecoveryJobRPDateTime_s |DateTime | La fecha y hora cuando se creó el punto de recuperación que se va a recuperar. |
| RecoveryJobRPLocation_s |Texto | La ubicación donde se almacenó el punto de recuperación que se va a recuperar.|
| SourceSystem |Texto |Sistema de origen de los datos actuales: Azure |
| ResourceId |Texto |Identificador de recursos de datos que se recopilan. Por ejemplo, el identificador del recurso del almacén de Recovery Services.|
| SubscriptionId |Texto |Identificador de la suscripción del recurso (p. ej., Almacén de Recovery Services) para el que se recopilan los datos |
| ResourceGroup |Texto |Grupo de recursos del recurso (p. ej., Almacén de Recovery Services) para el que se recopilan los datos |
| ResourceProvider |Texto |Proveedor de recursos para el que se recopilan los datos. Por ejemplo, Microsoft.RecoveryServices. |
| ResourceType |Texto |Tipo de recurso para el que se recopilan los datos. Por ejemplo: almacenes |

### <a name="policy"></a>Directiva

Esta tabla proporciona detalles acerca de los campos relacionados con las directivas.

| Campo | Tipo de datos | Versiones aplicables | Descripción |
| --- | --- | --- | --- |
| EventName_s |Texto ||Este campo representa el nombre de este evento. Siempre es AzureBackupCentralReport. |
| SchemaVersion_s |Texto ||Este campo indica la versión actual del esquema. Es **V2**. |
| State_s |Texto ||Estado actual del objeto de la directiva, por ejemplo, Active o Deleted |
| BackupManagementType_s |Texto ||Tipo de proveedor para el servidor que realiza el trabajo de copia de seguridad, por ejemplo, IaaSVM o FileFolder. |
| OperationName |Texto ||Este campo representa el nombre de la operación actual: Policy |
| Category |Texto ||Este campo representa la categoría de datos de diagnóstico insertados en los registros de Azure Monitor. Es AzureBackupReport. |
| Resource |Texto ||Este es el recurso para el que se están recopilando datos; muestra el nombre del almacén de Recovery Services |
| PolicyUniqueId_g |Texto ||Identificador único que identifica la directiva. |
| PolicyName_s |Texto ||Nombre de la directiva definida |
| BackupFrequency_s |Texto ||Frecuencia con la que se ejecutan las copias de seguridad, por ejemplo, a diario o semanalmente |
| BackupTimes_s |Texto ||Fecha y hora en que se programan las copias de seguridad |
| BackupDaysOfTheWeek_s |Texto ||Días de la semana en los que se han programado copias de seguridad |
| RetentionDuration_s |Whole Number ||Duración de retención de las copias de seguridad configuradas |
| DailyRetentionDuration_s |Whole Number ||Duración de retención total de las copias de seguridad configuradas, en días |
| DailyRetentionTimes_s |Texto ||Fecha y hora en que se configuró la retención diaria |
| WeeklyRetentionDuration_s |Decimal Number ||Duración total de la retención semanal de las copias de seguridad configuradas, en semanas |
| WeeklyRetentionTimes_s |Texto ||Fecha y hora en que se ha configurado la retención semanal |
| WeeklyRetentionDaysOfTheWeek_s |Texto ||Días de la semana seleccionados para la retención semanal |
| MonthlyRetentionDuration_s |Decimal Number ||Duración de retención total de las copias de seguridad configuradas, en meses |
| MonthlyRetentionTimes_s |Texto ||Fecha y hora en que se ha configurado la retención mensual |
| MonthlyRetentionFormat_s |Texto ||Tipo de configuración para la retención mensual ,por ejemplo, diariamente si se basa en día, semanalmente si se basa en semana |
| MonthlyRetentionDaysOfTheWeek_s |Texto ||Días de la semana seleccionados para la retención mensual |
| MonthlyRetentionWeeksOfTheMonth_s |Texto ||Semanas del mes en que se ha configurado la retención mensual, por ejemplo, la primera o la última. |
| YearlyRetentionDuration_s |Decimal Number ||Duración total de la retención de las copias de seguridad configuradas, en años |
| YearlyRetentionTimes_s |Texto ||Fecha y hora en que se ha configurado la retención anual |
| YearlyRetentionMonthsOfTheYear_s |Texto ||Meses del año seleccionados para la retención anual |
| YearlyRetentionFormat_s |Texto ||Tipo de configuración para la retención anual ,por ejemplo, diariamente si se basa en día, semanalmente si se basa en semana | |
| YearlyRetentionDaysOfTheMonth_s |Texto ||Fechas del mes seleccionadas para la retención anual |
| SynchronisationFrequencyPerDay_s |Whole Number |v2|Número de veces en un día que se sincroniza una copia de seguridad de archivos para SC DPM y MABS. |
| DiffBackupFormat_s |Texto |v2|Formato para copias de seguridad diferenciales de SQL en la copia de seguridad de máquinas virtuales de Azure. |
| DiffBackupTime_s |Time |v2|Tiempo para copias de seguridad diferenciales de SQL en la copia de seguridad de máquinas virtuales de Azure.|
| DiffBackupRetentionDuration_s |Decimal Number |v2|Duración de retención de las copias de seguridad diferenciales de SQL en la copia de seguridad de máquinas virtuales de Azure.|
| LogBackupFrequency_s |Decimal Number |v2|Frecuencia de las copias de seguridad de registros de SQL.|
| LogBackupRetentionDuration_s |Decimal Number |v2|Duración de retención de las copias de seguridad de registro de SQL en la copia de seguridad de máquinas virtuales de Azure.|
| DiffBackupDaysofTheWeek_s |Texto |v2|Días de la semana de las copias de seguridad diferenciales de SQL en la copia de seguridad de máquinas virtuales de Azure.|
| SourceSystem |Texto ||Sistema de origen de los datos actuales: Azure |
| ResourceId |Texto ||Identificador de recursos de datos que se recopilan. Por ejemplo, el identificador del recurso del almacén de Recovery Services. |
| SubscriptionId |Texto ||Identificador de la suscripción del recurso (p. ej., Almacén de Recovery Services) para el que se recopilan los datos |
| ResourceGroup |Texto ||Grupo de recursos del recurso (p. ej., Almacén de Recovery Services) para el que se recopilan los datos |
| ResourceProvider |Texto ||Proveedor de recursos para el que se recopilan los datos. Por ejemplo, Microsoft.RecoveryServices. |
| ResourceType |Texto ||Tipo de recurso para el que se recopilan los datos. Por ejemplo: almacenes |

### <a name="policyassociation"></a>PolicyAssociation

Esta tabla proporciona detalles acerca de las asociaciones de directivas con varias entidades.

| Campo | Tipo de datos | Versiones aplicables | Descripción |
| --- | --- | --- | --- |
| EventName_s |Texto ||Este campo representa el nombre de este evento. Siempre es AzureBackupCentralReport. |
| SchemaVersion_s |Texto ||Este campo indica la versión actual del esquema. Es **V2**. |
| State_s |Texto ||Estado actual del objeto de la directiva, por ejemplo, Active o Deleted |
| BackupManagementType_s |Texto ||Tipo de proveedor para el servidor que realiza el trabajo de copia de seguridad, por ejemplo, IaaSVM o FileFolder. |
| OperationName |Texto ||Este campo representa el nombre de la operación actual: PolicyAssociation |
| Category |Texto ||Este campo representa la categoría de datos de diagnóstico insertados en los registros de Azure Monitor. Es AzureBackupReport. |
| Resource |Texto ||Este es el recurso para el que se están recopilando datos; muestra el nombre del almacén de Recovery Services |
| PolicyUniqueId_g |Texto ||Identificador único que identifica la directiva. |
| VaultUniqueId_s |Texto ||Identificador exclusivo del almacén al que pertenece esta directiva. |
| BackupManagementServerUniqueId_s |Texto |v2 |Campo para identificar de forma única el servidor de administración de copia de seguridad mediante el que se protege el elemento de copia de seguridad, si procede.        |
| SourceSystem |Texto ||Sistema de origen de los datos actuales: Azure |
| ResourceId |Texto ||Identificador de recursos de datos que se recopilan. Por ejemplo, el identificador del recurso del almacén de Recovery Services. |
| SubscriptionId |Texto ||Identificador de la suscripción del recurso (p. ej., Almacén de Recovery Services) para el que se recopilan los datos |
| ResourceGroup |Texto ||Grupo de recursos del recurso (p. ej., Almacén de Recovery Services) para el que se recopilan los datos |
| ResourceProvider |Texto ||Proveedor de recursos para el que se recopilan los datos. Por ejemplo, Microsoft.RecoveryServices. |
| ResourceType |Texto ||Tipo de recurso para el que se recopilan los datos. Por ejemplo: almacenes |

### <a name="protected-container"></a>Contenedor protegido

En esta tabla se proporcionan campos básicos sobre los contenedores protegidos. (fue ProtectedServer en v1)

| Campo | Tipo de datos | Descripción |
| --- | --- | --- |
| ProtectedContainerUniqueId_s |Texto | Campo para identificar de forma única un contenedor protegido. |
| ProtectedContainerOSType_s |Texto |Tipo de sistema operativo del contenedor protegido. |
| ProtectedContainerOSVersion_s |Texto |Versión del sistema operativo del contenedor protegido. |
| AgentVersion_s |Texto |Número de versión del agente de copia de seguridad o del agente de protección (en el caso de SC DPM y MABS). |
| BackupManagementType_s |Texto |Tipo de proveedor para realizar una copia de seguridad. Por ejemplo, IaaSVM o FileFolder |
| EntityState_s |Texto |Estado actual del objeto de servidor protegido. Por ejemplo, Activo o Eliminado |
| ProtectedContainerFriendlyName_s |Texto |Nombre descriptivo del servidor protegido |
| ProtectedContainerName_s |Texto |Nombre del contenedor protegido. |
| ProtectedContainerWorkloadType_s |Texto |Tipo de la copia de seguridad del contenedor protegido. Por ejemplo, IaaSVMContainer |
| ProtectedContainerLocation_s |Texto |Si el contenedor protegido está ubicado en el entorno local o en Azure. |
| ProtectedContainerType_s |Texto |Si el contenedor protegido es un servidor o un contenedor. |
| ProtectedContainerProtectionState_s’  |Texto |Estado de protección del contenedor protegido |

### <a name="storage"></a>Storage

Esta tabla proporciona detalles acerca de los campos relacionados con el almacenamiento.

| Campo | Tipo de datos | Descripción |
| --- | --- | --- |
| CloudStorageInBytes_s |Decimal Number |Almacenamiento de copia de seguridad en la nube utilizado por las copias de seguridad, calculados basándose en el último valor (este campo solo es para el esquema v1)|
| ProtectedInstances_s |Decimal Number |Número de instancias protegidas que se utilizan para calcular el almacenamiento de front-end en la facturación; se calcula en función del valor más reciente |
| EventName_s |Texto |Este campo representa el nombre de este evento. Siempre es AzureBackupCentralReport. |
| SchemaVersion_s |Texto |Este campo indica la versión actual del esquema. Es **V2**. |
| State_s |Texto |Estado actual del objeto de almacenamiento, por ejemplo, Active o Deleted |
| BackupManagementType_s |Texto |Tipo de proveedor para el servidor que realiza el trabajo de copia de seguridad, por ejemplo, IaaSVM o FileFolder. |
| OperationName |Texto |Este campo representa el nombre de la operación actual: Storage |
| Category |Texto |Este campo representa la categoría de datos de diagnóstico insertados en los registros de Azure Monitor. Es AzureBackupReport. |
| Resource |Texto |Este es el recurso para el que se están recopilando datos; muestra el nombre del almacén de Recovery Services |
| ProtectedServerUniqueId_s |Texto |Identificador exclusivo del servidor protegido para el que se calcula el almacenamiento. |
| VaultUniqueId_s |Texto |Identificador exclusivo del almacén para el que se calcula el almacenamiento. |
| SourceSystem |Texto |Sistema de origen de los datos actuales: Azure |
| ResourceId |Texto |Identificador de recursos de datos que se recopilan. Por ejemplo, el identificador del recurso del almacén de Recovery Services. |
| SubscriptionId |Texto |Identificador de la suscripción del recurso (p. ej., Almacén de Recovery Services) para el que se recopilan los datos |
| ResourceGroup |Texto |Grupo de recursos del recurso (p. ej., Almacén de Recovery Services) para el que se recopilan los datos |
| ResourceProvider |Texto |Proveedor de recursos para el que se recopilan los datos. Por ejemplo, Microsoft.RecoveryServices. |
| ResourceType |Texto |Tipo de recurso para el que se recopilan los datos. Por ejemplo: almacenes |
| StorageUniqueId_s |Texto |Identificador único de la entidad de almacenamiento. |
| StorageType_s |Texto |Tipo de almacenamiento, por ejemplo, Nube, Volumen o Disco |
| StorageName_s |Texto |Nombre de la entidad de almacenamiento, por ejemplo, E:\ |
| StorageTotalSizeInGBs_s |Texto |Tamaño total de almacenamiento, en GB, consumido por la entidad de almacenamiento|

### <a name="storageassociation"></a>StorageAssociation

En esta tabla se proporcionan campos relacionados con almacenamiento básico que conectan el almacenamiento con otras entidades.

| Campo | Tipo de datos | Descripción |
| --- | --- |  --- |
| StorageUniqueId_s |Texto |Identificador único de la entidad de almacenamiento. |
| SchemaVersion_s |Texto |Este campo indica la versión actual del esquema. Es **V2**. |
| BackupItemUniqueId_s |Texto |Identificador único del elemento de copia de seguridad relacionado con la entidad de almacenamiento. |
| BackupManagementServerUniqueId_s |Texto |Identificador único del servidor de administración de copia de seguridad relacionado con la entidad de almacenamiento.|
| VaultUniqueId_s |Texto |Identificador único del almacén relacionado con la entidad de almacenamiento.|
| StorageConsumedInMBs_s |Number|Tamaño de almacenamiento usado por el elemento de copia de seguridad correspondiente en el almacenamiento correspondiente. |
| StorageAllocatedInMBs_s |Number |Tamaño de almacenamiento asignado por el elemento de copia de seguridad correspondiente en el almacenamiento correspondiente del tipo de disco.|

### <a name="vault"></a>Almacén

Esta tabla proporciona detalles acerca de los campos relacionados con el almacén.

| Campo | Tipo de datos | Descripción |
| --- | --- | --- |
| EventName_s |Texto |Este campo representa el nombre de este evento. Siempre es AzureBackupCentralReport. |
| SchemaVersion_s |Texto |Este campo indica la versión actual del esquema. Es **V2**. |
| State_s |Texto |Estado actual del objeto del almacén por ejemplo, Active o Deleted |
| OperationName |Texto |Este campo representa el nombre de la operación actual: Vault |
| Category |Texto |Este campo representa la categoría de datos de diagnóstico insertados en los registros de Azure Monitor. Es AzureBackupReport. |
| Resource |Texto |Este es el recurso para el que se están recopilando datos; muestra el nombre del almacén de Recovery Services |
| VaultUniqueId_s |Texto |Identificador exclusivo del almacén. |
| VaultName_s |Texto |Nombre del almacén |
| AzureDataCenter_s |Texto |Centro de datos donde se encuentra el almacén |
| StorageReplicationType_s |Texto |Tipo de replicación de almacenamiento para el almacén, por ejemplo, GeoRedundant |
| SourceSystem |Texto |Sistema de origen de los datos actuales: Azure |
| ResourceId |Texto |Identificador de recursos de datos que se recopilan. Por ejemplo, el identificador del recurso del almacén de Recovery Services. |
| SubscriptionId |Texto |Identificador de la suscripción del recurso (p. ej., Almacén de Recovery Services) para el que se recopilan los datos |
| ResourceGroup |Texto |Grupo de recursos del recurso (p. ej., Almacén de Recovery Services) para el que se recopilan los datos |
| ResourceProvider |Texto |Proveedor de recursos para el que se recopilan los datos. Por ejemplo, Microsoft.RecoveryServices. |
| ResourceType |Texto |Tipo de recurso para el que se recopilan los datos. Por ejemplo: almacenes |

### <a name="backup-management-server"></a>Servidor de administración de copia de seguridad

En esta tabla se proporcionan campos básicos sobre los servidores de administración de copia de seguridad.

|Campo  |Tipo de datos  | Descripción  |
|---------|---------|----------|
|BackupManagementServerName_s     |Texto         |Nombre del servidor de administración de copia de seguridad.        |
|AzureBackupAgentVersion_s     |Texto         |Versión de Azure Backup Agent en el servidor de administración de copia de seguridad.          |
|BackupManagementServerVersion_s     |Texto         |Versión del servidor de administración de copia de seguridad.|
|BackupManagementServerOSVersion_s     |Texto            |Versión del sistema operativo del servidor de administración de copia de seguridad.|
|BackupManagementServerType_s     |Texto         |Tipo del servidor de administración de copia de seguridad, como MABS o SC DPM.|
|BackupManagementServerUniqueId_s     |Texto         |Campo para identificar de forma única el servidor de administración de copia de seguridad.       |

### <a name="preferredworkloadonvolume"></a>PreferredWorkloadOnVolume

En esta tabla se especifican las cargas de trabajo a las que está asociado un volumen.

| Campo | Tipo de datos | Descripción |
| --- | --- | --- |
| StorageUniqueId_s |Texto |Identificador único de la entidad de almacenamiento. |
| BackupItemType_s |Texto |Las cargas de trabajo para las que este volumen es el almacenamiento preferido.|

### <a name="protectedinstance"></a>ProtectedInstance

En esta tabla se proporcionan los campos relacionados con las instancias protegidas básicas.

| Campo | Tipo de datos |Versiones aplicables | Descripción |
| --- | --- | --- | --- |
| BackupItemUniqueId_s |Texto |v2|Identificador único del elemento de copia de seguridad de máquinas virtuales copiadas con DPM o MABS.|
| ProtectedContainerUniqueId_s |Texto |v2|Identificador único del contenedor protegido para todo excepto las máquinas virtuales copiadas con DPM o MABS.|
| ProtectedInstanceCount_s |Texto |v2|Recuento de instancias protegidas del elemento de copia de seguridad o del contenedor protegido asociados en esa fecha y hora.|

### <a name="recoverypoint"></a>RecoveryPoint

En esta tabla se proporcionan los campos básicos relacionados con el punto de recuperación.

| Campo | Tipo de datos | Descripción |
| --- | --- | --- |
| BackupItemUniqueId_s |Texto |Identificador único del elemento de copia de seguridad de máquinas virtuales copiadas con DPM o MABS.|
| OldestRecoveryPointTime_s |Texto |Fecha y hora del punto de recuperación más antiguo para el elemento de copia de seguridad.|
| OldestRecoveryPointLocation_s |Texto |Ubicación del punto de recuperación más antiguo para el elemento de copia de seguridad.|
| LatestRecoveryPointTime_s |Texto |Fecha y hora del último punto de recuperación para el elemento de copia de seguridad.|
| LatestRecoveryPointLocation_s |Texto |Ubicación del punto de recuperación más reciente para el elemento de copia de seguridad.|

## <a name="sample-kusto-queries"></a>Ejemplos de consultas de Kusto

A continuación se muestran algunos ejemplos que le ayudarán a escribir consultas sobre los datos de Azure Backup que residen en la tabla Azure Diagnostics:

- Todos los trabajos de copia de seguridad con resultado satisfactorio

    ````Kusto
    AzureDiagnostics
    | where Category == "AzureBackupReport"
    | where SchemaVersion_s == "V2"
    | where OperationName == "Job" and JobOperation_s == "Backup"
    | where JobStatus_s == "Completed"
    ````

- Todos los trabajos de copia de seguridad con errores

    ````Kusto
    AzureDiagnostics
    | where Category == "AzureBackupReport"
    | where SchemaVersion_s == "V2"
    | where OperationName == "Job" and JobOperation_s == "Backup"
    | where JobStatus_s == "Failed"
    ````

- Todos los trabajos de copia de seguridad de máquinas virtuales de Azure con resultado satisfactorio

    ````Kusto
    AzureDiagnostics
    | where Category == "AzureBackupReport"
    | where SchemaVersion_s == "V2"
    | extend JobOperationSubType_s = columnifexists("JobOperationSubType_s", "")
    | where OperationName == "Job" and JobOperation_s == "Backup" and JobStatus_s == "Completed" and JobOperationSubType_s != "Log" and JobOperationSubType_s != "Recovery point_Log"
    | join kind=inner
    (
        AzureDiagnostics
        | where Category == "AzureBackupReport"
        | where OperationName == "BackupItem"
        | where SchemaVersion_s == "V2"
        | where BackupItemType_s == "VM" and BackupManagementType_s == "IaaSVM"
        | distinct BackupItemUniqueId_s, BackupItemFriendlyName_s
        | project BackupItemUniqueId_s , BackupItemFriendlyName_s
    )
    on BackupItemUniqueId_s
    | extend Vault= Resource
    | project-away Resource
    ````

- Todos los trabajos de copia de seguridad del registro de SQL con resultado satisfactorio

    ````Kusto
    AzureDiagnostics
    | where Category == "AzureBackupReport"
    | where SchemaVersion_s == "V2"
    | extend JobOperationSubType_s = columnifexists("JobOperationSubType_s", "")
    | where OperationName == "Job" and JobOperation_s == "Backup" and JobStatus_s == "Completed" and JobOperationSubType_s == "Log"
    | join kind=inner
    (
        AzureDiagnostics
        | where Category == "AzureBackupReport"
        | where OperationName == "BackupItem"
        | where SchemaVersion_s == "V2"
        | where BackupItemType_s == "SQLDataBase" and BackupManagementType_s == "AzureWorkload"
        | distinct BackupItemUniqueId_s, BackupItemFriendlyName_s
        | project BackupItemUniqueId_s , BackupItemFriendlyName_s
    )
    on BackupItemUniqueId_s
    | extend Vault= Resource
    | project-away Resource
    ````

- Todos los trabajos del agente de Azure Backup con resultado satisfactorio

    ````Kusto
    AzureDiagnostics
    | where Category == "AzureBackupReport"
    | where SchemaVersion_s == "V2"
    | extend JobOperationSubType_s = columnifexists("JobOperationSubType_s", "")
    | where OperationName == "Job" and JobOperation_s == "Backup" and JobStatus_s == "Completed" and JobOperationSubType_s != "Log" and JobOperationSubType_s != "Recovery point_Log"
    | join kind=inner
    (
        AzureDiagnostics
        | where Category == "AzureBackupReport"
        | where OperationName == "BackupItem"
        | where SchemaVersion_s == "V2"
        | where BackupItemType_s == "FileFolder" and BackupManagementType_s == "MAB"
        | distinct BackupItemUniqueId_s, BackupItemFriendlyName_s
        | project BackupItemUniqueId_s , BackupItemFriendlyName_s
    )
    on BackupItemUniqueId_s
    | extend Vault= Resource
    | project-away Resource
    ````

## <a name="v1-schema-vs-v2-schema"></a>Esquema v1 frente a esquema V2

Anteriormente, los datos de diagnóstico para el agente de Azure Backup y la copia de seguridad de VM de Azure se enviaban a una tabla de Azure Diagnostics en un esquema denominado ***esquema V1***. Posteriormente, se agregaron nuevas columnas para admitir otros escenarios y cargas de trabajo, y los datos de diagnóstico se insertaron en un nuevo esquema denominado ***esquema V2***.  

Por motivos de compatibilidad con versiones anteriores, los datos de diagnóstico para el agente de Azure Backup y la copia de seguridad de VM de Azure se envían actualmente a la tabla de Azure Diagnostics en el esquema V1 y V2 (con el esquema v1 ahora en una ruta de degradación). Puede identificar qué registros de Log Analytics son del esquema V1 filtrando los registros para SchemaVersion_s=="v1" en las consultas de registro.

Vea la tercera columna "Descripción" del [modelo de datos](#using-azure-backup-data-model) descrito arriba para identificar las columnas que pertenecen solo al esquema V1.

### <a name="modifying-your-queries-to-use-the-v2-schema"></a>Modificación de las consultas para usar el esquema v2

Dado que el esquema V1 está en proceso de degradación, se recomienda usar solo el esquema V2 en todas las consultas personalizadas en los datos de diagnóstico de Azure Backup. A continuación se muestra un ejemplo de cómo actualizar las consultas para quitar la dependencia del esquema v1:

1. Identifique si la consulta está usando cualquier campo que solo sea aplicable al esquema v1. Suponga que tiene una consulta para enumerar todos los elementos de copia de seguridad y sus servidores protegidos asociados como se indica a continuación:

    ````Kusto
    AzureDiagnostics
    | where Category=="AzureBackupReport"
    | where OperationName=="BackupItemAssociation"
    | distinct BackupItemUniqueId_s, ProtectedServerUniqueId_s
    ````

    La consulta anterior usa el campo ProtectedServerUniqueId_s, que solo es aplicable al esquema V1. El esquema v2 equivalente de este campo es ProtectedContainerUniqueId_s (consulte las tablas anteriores). El campo BackupItemUniqueId_s es aplicable incluso al esquema v2 y se puede usar el mismo campo en esta consulta.

2. Actualice la consulta para usar los nombres de campo de esquema v2. Es un procedimiento recomendado usar el filtro **where SchemaVersion_s=="V2"** en todas las consultas, de modo que solo los registros correspondientes al esquema V2 se analicen mediante la consulta:

    ````Kusto
    AzureDiagnostics
    | where Category=="AzureBackupReport"
    | where OperationName=="BackupItemAssociation"
    | where SchemaVersion_s=="V2"
    | distinct BackupItemUniqueId_s, ProtectedContainerUniqueId_s
    ````

## <a name="next-steps"></a>Pasos siguientes

Después de revisar el modelo de datos, puede comenzar a [crear consultas personalizadas](../azure-monitor/learn/tutorial-logs-dashboards.md) en los registros de Azure Monitor para elaborar su propio panel.
