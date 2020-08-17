---
title: Consulta de registros de Update Management en Azure Automation
description: En este artículo se explica cómo consultar los registros de Update Management en el área de trabajo de Log Analytics.
services: automation
ms.subservice: update-management
ms.date: 07/28/2020
ms.topic: conceptual
ms.openlocfilehash: 290fb0165038eea8740361a12a6d4bfe2c1bf138
ms.sourcegitcommit: cee72954f4467096b01ba287d30074751bcb7ff4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/30/2020
ms.locfileid: "87449764"
---
# <a name="query-update-management-logs"></a>Consulta de registros de Update Management

Además de los detalles que se proporcionan durante la implementación de Update Management, puede buscar en los registros almacenados en el área de trabajo de Log Analytics. Para buscar los registros desde la cuenta de Automation, seleccione **Update Management** y abra el área de trabajo de Log Analytics asociada a la implementación.

También se pueden personalizar las consultas de registros o usarlas desde clientes distintos. Consulte [Documentación de Log Analytics Search API](https://dev.loganalytics.io/).

## <a name="query-update-records"></a>Consulta de registros de actualización

Update Management recopila los registros para máquinas virtuales Linux y Windows, y los tipos de datos que aparecen en los resultados de la búsqueda de registros. En las secciones siguientes se describen estos registros.

### <a name="query-required-updates"></a>Consulta de actualizaciones necesarias

Se crea un registro con un tipo de `RequiredUpdate` que representa las actualizaciones que necesita un equipo. Estos registros tienen las propiedades de la tabla siguiente:

| Propiedad | Descripción |
|----------|-------------|
| Computer | Nombre de dominio completo del equipo que genera el informe. |
| KBID | Identificador del artículo de Knowledge base para la actualización de Windows. |
| ManagementGroupName | Nombre del grupo de administración de Operations Manager o del área de trabajo de Log Analytics. |
| Producto | Los productos para los que se aplica la actualización. |
| PublishDate | La fecha en la que la actualización está lista para descargarse e instalarse desde Windows Update. |
| Server | | 
| SourceHealthServiceId | Identificador único que representa el identificador del agente de Windows de Log Analytics. |
| SourceSystem | *OperationsManager* |
| TenantId | Identificador único que representa la instancia de la organización de Azure Active Directory. |
| TimeGenerated | Fecha y hora en que se creó el registro. |
| Tipo | *Actualizar* |
| UpdateClassification | Indica el tipo de actualizaciones que se pueden aplicar. Para Windows:<br> *Actualizaciones críticas*<br> *Actualizaciones de seguridad*<br> *Paquetes acumulativos de actualizaciones*<br> *Paquetes de características*<br> *Service Packs*<br> *Actualizaciones de definiciones*<br> *Herramientas*<br> *Actualizaciones*. Para Linux:<br> *Actualizaciones críticas y de seguridad*<br> *Otros* |
| UpdateSeverity | Clasificación de gravedad de la vulnerabilidad. Los valores son:<br> *Critical)* (Crítico)<br> *Importante*<br> *Moderado*<br> *Baja* |
| UpdateTitle | Título de la actualización.|

### <a name="query-update-record"></a>Consulta del registro de la actualización

Se crea un registro con un tipo de `Update` que representa las actualizaciones disponibles y su estado de instalación para un equipo. Estos registros tienen las propiedades de la tabla siguiente:

| Propiedad | Descripción |
|----------|-------------|
| ApprovalSource | Solo se aplica al sistema operativo Windows. Origen de la aprobación del registro. El valor es Microsoft Update. |
| Aprobado | True si el registro está aprobado o False en caso contrario. |
| clasificación | Clasificación de aprobación. El valor es Updates (Actualizaciones). |
| Computer | Nombre de dominio completo del equipo que genera el informe. |
| ComputerEnvironment | Entorno. Los valores posibles son Azure y Non-Azure. |
| MSRCBulletinID | Número de identificación del boletín de seguridad. |
| MSRCSeverity | Clasificación de gravedad de la vulnerabilidad. Los valores son:<br> Crítico<br> Importante<br> Moderado<br> Bajo |  
| KBID | Identificador del artículo de Knowledge base para la actualización de Windows. |
| ManagementGroupName | Nombre del grupo de administración de Operations Manager o del área de trabajo de Log Analytics. |
| UpdateID | Identificador único de la actualización de software. |
| RevisionNumber | El número de revisión de una revisión específica de una actualización. |
| Opcional | True si se el registro es opcional o False en caso contrario. |
| RebootBehavior | El comportamiento de reinicio después de instalar o desinstalar una actualización. |
| _ResourceId | Identificador único del recurso asociado al registro. |
| Tipo | Tipo de registro. El valor es Update (Actualización). |
| VMUUID | Identificador único de la máquina virtual. |
| MG | Identificador único del grupo de administración o del área de trabajo de Log Analytics. |
| TenantId | Identificador único que representa la instancia de la organización de Azure Active Directory. |
| SourceSystem | Sistema de origen del registro. El valor es `OperationsManager`. |
| TimeGenerated | Fecha y hora de creación del registro. |
| SourceComputerId | Identificador único que representa el equipo de origen. |
| Título | Título de la actualización. |
| PublishedDate (UTC) | Fecha en la que la actualización está lista para descargarse e instalarse desde Windows Update.  |
| UpdateState | Estado actual de la actualización. |
| Producto | Productos a los que se aplica la actualización. |
| SubscriptionId | Identificador único de la suscripción de Azure. |
| ResourceGroup | Nombre del grupo de recursos al que pertenece el recurso. |
| ResourceProvider | El proveedor de recursos. |
| Resource | Nombre del recurso. |
| ResourceType | El tipo de recurso. |

### <a name="query-update-agent-record"></a>Consulta del registro de agente de la actualización

Se crea un registro con un tipo de `UpdateAgent` que proporciona detalles del agente de actualización en el equipo. Estos registros tienen las propiedades de la tabla siguiente:

| Propiedad | Descripción |
|----------|-------------|
| AgeofOldestMissingRequiredUpdate | |
| AutomaticUpdateEnabled | |
| Computer | Nombre de dominio completo del equipo que genera el informe. |
| DaySinceLastUpdateBucket | |
| ManagementGroupName | Nombre del grupo de administración de Operations Manager o del área de trabajo de Log Analytics. |
| OSVersion | La versión del sistema operativo. |
| Server | |
| SourceHealthServiceId | Identificador único que representa el identificador del agente de Windows de Log Analytics. |
| SourceSystem | Sistema de origen del registro. El valor es `OperationsManager`. |
| TenantId | Identificador único que representa la instancia de la organización de Azure Active Directory. |
| TimeGenerated | Fecha y hora de creación del registro. |
| Tipo | Tipo de registro. El valor es Update (Actualización). |
| WindowsUpdateAgentVersion | Versión del Agente de Windows Update. |
| WSUSServer | Errores en el caso de que el agente de Windows Update tenga un problema, para ayudar en la solución de problemas. |

### <a name="query-update-deployment-status-record"></a>Consulta del registro del estado de implementación de la actualización

Se crea un registro con un tipo de `UpdateRunProgress` que proporciona el estado de implementación de actualización de una implementación programada por equipo. Estos registros tienen las propiedades de la tabla siguiente:

| Propiedad | Descripción |
|----------|-------------|
| Computer | Nombre de dominio completo del equipo que genera el informe. |
| ComputerEnvironment | Entorno. Los valores posibles son Azure y Non-Azure. |
| CorrelationId | Identificador único del trabajo de runbook ejecutado para la actualización. |
| EndTime | La hora a la que ha finalizado el proceso de sincronización. |
| ErrorResult | Código de error de Windows Update generado si no se puede instalar una actualización. |
| InstallationStatus | Los estados de instalación posibles de una actualización en el equipo cliente,<br> `NotStarted`: el trabajo aún no se ha desencadenado.<br> `FailedToStart`: no se puede iniciar el trabajo en la máquina.<br> `Failed`: el trabajo se inició, pero devolvió una excepción.<br> `InProgress`: trabajo en curso.<br> `MaintenanceWindowExceeded`: la ejecución seguía en curso pero se alcanzó el intervalo de la ventana de mantenimiento.<br> `Succeeded`: el trabajo finalizó correctamente.<br> `InstallFailed`: error de instalación de la actualización.<br> `NotIncluded`<br> `Excluded` |
| KBID | Identificador del artículo de Knowledge base para la actualización de Windows. |
| ManagementGroupName | Nombre del grupo de administración de Operations Manager o del área de trabajo de Log Analytics. |
| OSType | Tipo de sistema operativo. Los valores son Windows o Linux. |
| Producto | Productos a los que se aplica la actualización. |
| Resource | Nombre del recurso. |
| ResourceId | Identificador único del recurso asociado al registro. |
| ResourceProvider | El proveedor de recursos. |
| ResourceType | Tipo de recurso. |
| SourceComputerId | Identificador único que representa el equipo de origen. |
| SourceSystem | Sistema de origen del registro. El valor es `OperationsManager`. |
| StartTime | Hora a la que se ha programado la instalación de la actualización. |
| SubscriptionId | Identificador único de la suscripción de Azure. | 
| SucceededOnRetry | Valor que indica si se ha producido un error en la ejecución de la actualización en el primer intento y la operación actual es un reintento. |
| TimeGenerated | Fecha y hora de creación del registro. |
| Título | Título de la actualización. |
| Tipo | Tipo de actualización. El valor es `UpdateRunProgress`. |
| UpdateId | Identificador único de la actualización de software. |
| VMUUID | Identificador único de la máquina virtual. |
| ResourceId | Identificador único del recurso asociado al registro. |

### <a name="query-update-summary-record"></a>Consulta del registro de resumen de la actualización

Se crea un registro con un tipo de `UpdateSummary` que proporciona el resumen de actualizaciones por equipo. Estos registros tienen las propiedades de la tabla siguiente:

| Propiedad | Descripción |
|----------|-------------|
| Computer | Nombre de dominio completo del equipo que genera el informe. |
| ComputerEnvironment | Entorno. Los valores posibles son Azure y Non-Azure. |
| CriticalUpdatesMissing | Número de actualizaciones críticas aplicables que faltan. |
| ManagementGroupName | Nombre del grupo de administración de Operations Manager o del área de trabajo de Log Analytics. |
| NETRuntimeVersion | Versión de .NET Framework instalada en el equipo Windows. |
| OldestMissingSecurityUpdateBucket | Especificador del depósito de seguridad que falta más antiguo. Los valores son:<br> Recent (Reciente) si el valor es inferior a 30 días<br> Hace 30 días<br> Hace 60 días<br> Hace 90 días<br> Hace 120 días<br> Hace 150 días<br> Hace 180 días<br> Older (Anterior) si el valor supera los 180 días. |
| OldestMissingSecurityUpdateInDays | Número total de días de la actualización más antigua detectada como aplicable que no se ha instalado. |
| OsVersion | La versión del sistema operativo. |
| OtherUpdatesMissing | Recuento de actualizaciones detectadas que faltan. |
| Resource | Nombre del recurso del registro. |
| ResourceGroup | Nombre del grupo de recursos que contiene el recurso. |
| ResourceId | Identificador único del recurso asociado al registro. |
| ResourceProvider | El proveedor de recursos. |
| ResourceType | Tipo de recurso. |
| RestartPending | True si hay un reinicio pendiente o False en caso contrario. |
| SecurityUpdatesMissing | Recuento de actualizaciones de seguridad que faltan que se pueden aplicar.|
| SourceComputerId | Identificador único de la máquina virtual. |
| SourceSystem | Sistema de origen del registro. El valor es `OpsManager`. |
| SubscriptionId | Identificador único de la suscripción de Azure. |
| TimeGenerated | Fecha y hora de creación del registro. |
| TotalUpdatesMissing | Número total de actualizaciones que faltan que se pueden aplicar. |
| Tipo | Tipo de registro. El valor es `UpdateSummary`. |
| VMUUID | Identificador único de la máquina virtual. |
| WindowsUpdateAgentVersion | Versión del Agente de Windows Update. |
| WindowsUpdateSetting | Estado del agente de Windows Update. Los valores posibles son:<br> `Scheduled installation`<br> `Notify before installation`<br> `Error returned from unhealthy WUA agent` |
| WSUSServer | Errores en el caso de que el agente de Windows Update tenga un problema, para ayudar en la solución de problemas. |
| _ResourceId | Identificador único del recurso asociado al registro. |

## <a name="sample-queries"></a>Consultas de ejemplo

En las secciones siguientes se proporcionan ejemplos de consultas de registros para los registros de actualizaciones recopilados por Update Management.

### <a name="confirm-that-non-azure-machines-are-enabled-for-update-management"></a>Confirmación de que las máquinas que no son de Azure están habilitadas para Update Management

Para confirmar que las máquinas conectadas directamente se comunican con los registros de Azure Monitor, ejecute una de las siguientes búsquedas de registros.

#### <a name="linux"></a>Linux

```loganalytics
Heartbeat
| where OSType == "Linux" | summarize arg_max(TimeGenerated, *) by SourceComputerId | top 500000 by Computer asc | render table
```

#### <a name="windows"></a>Windows

```loganalytics
Heartbeat
| where OSType == "Windows" | summarize arg_max(TimeGenerated, *) by SourceComputerId | top 500000 by Computer asc | render table
```

En un equipo Windows, puede revisar la siguiente información para comprobar la conectividad del agente con los registros de Azure Monitor:

1. En el Panel de Control, abra **Microsoft Monitoring Agent**. En la pestaña **Azure Log Analytics**, el agente muestra el mensaje siguiente: **Microsoft Monitoring Agent se ha conectado correctamente a Log Analytics.**

1. Abra el registro de eventos de Windows. Vaya a **Application and Services Logs\Operations Manager** y busque los identificadores de evento 3000 y 5002 desde el **Conector de servicio** de origen. Estos eventos indican que el equipo se ha registrado con el área de trabajo de Log Analytics y está recibiendo la configuración.

Si el agente no puede comunicarse con los registros de Azure Monitor y está configurado para comunicarse con Internet mediante un firewall o un servidor proxy, confirme que estos están configurados correctamente. Para aprender a comprobar que un firewall o un servidor proxy estén correctamente configurados, consulte [Configuración de red para el agente Windows](../../azure-monitor/platform/agent-windows.md) o [Configuración de red para el agente de Linux](../../azure-monitor/learn/quick-collect-linux-computer.md).

> [!NOTE]
> Si los sistemas Linux están configurados para comunicarse con un servidor proxy o una puerta de enlace de Log Analytics y va a habilitar Update Management, actualice los permisos `proxy.conf` a fin de conceder al grupo omiuser permiso de lectura sobre el archivo mediante el uso de los comandos siguientes:
>
> `sudo chown omsagent:omiusers /etc/opt/microsoft/omsagent/proxy.conf`
> `sudo chmod 644 /etc/opt/microsoft/omsagent/proxy.conf`

Los agentes de Linux recién agregados muestran un estado **Actualizado** después de haber realizado una evaluación. Este proceso puede tardar hasta 6 horas.

Para confirmar que un grupo de administración de Operations Manager se comunica con los registros de Azure Monitor, consulte [Validación de la integración de Operations Manager con los registros de Azure Monitor](../../azure-monitor/platform/om-agents.md#validate-operations-manager-integration-with-azure-monitor).

### <a name="single-azure-vm-assessment-queries-windows"></a>Consultas individuales de evaluación de Azure VM (Windows)

Reemplace el valor de VMUUID con el GUID de la máquina virtual en la que va a realizar la consulta. Para encontrar el VMUUID que se debe usar, ejecute la siguiente consulta en los registros de Azure Monitor: `Update | where Computer == "<machine name>" | summarize by Computer, VMUUID`

#### <a name="missing-updates-summary"></a>Falta resumen de actualizaciones

```loganalytics
Update
| where TimeGenerated>ago(14h) and OSType!="Linux" and (Optional==false or Classification has "Critical" or Classification has "Security") and VMUUID=~"b08d5afa-1471-4b52-bd95-a44fea6e4ca8"
| summarize hint.strategy=partitioned arg_max(TimeGenerated, UpdateState, Classification, Approved) by Computer, SourceComputerId, UpdateID
| where UpdateState=~"Needed" and Approved!=false
| summarize by UpdateID, Classification
| summarize allUpdatesCount=count(), criticalUpdatesCount=countif(Classification has "Critical"), securityUpdatesCount=countif(Classification has "Security"), otherUpdatesCount=countif(Classification !has "Critical" and Classification !has "Security")
```

#### <a name="missing-updates-list"></a>Falta lista de actualizaciones

```loganalytics
Update
| where TimeGenerated>ago(14h) and OSType!="Linux" and (Optional==false or Classification has "Critical" or Classification has "Security") and VMUUID=~"8bf1ccc6-b6d3-4a0b-a643-23f346dfdf82"
| summarize hint.strategy=partitioned arg_max(TimeGenerated, UpdateState, Classification, Title, KBID, PublishedDate, Approved) by Computer, SourceComputerId, UpdateID
| where UpdateState=~"Needed" and Approved!=false
| project-away UpdateState, Approved, TimeGenerated
| summarize computersCount=dcount(SourceComputerId, 2), displayName=any(Title), publishedDate=min(PublishedDate), ClassificationWeight=max(iff(Classification has "Critical", 4, iff(Classification has "Security", 2, 1))) by id=strcat(UpdateID, "_", KBID), classification=Classification, InformationId=strcat("KB", KBID), InformationUrl=iff(isnotempty(KBID), strcat("https://support.microsoft.com/kb/", KBID), ""), osType=2
| sort by ClassificationWeight desc, computersCount desc, displayName asc
| extend informationLink=(iff(isnotempty(InformationId) and isnotempty(InformationUrl), toobject(strcat('{ "uri": "', InformationUrl, '", "text": "', InformationId, '", "target": "blank" }')), toobject('')))
| project-away ClassificationWeight, InformationId, InformationUrl
```

### <a name="single-azure-vm-assessment-queries-linux"></a>Consultas individuales de evaluación de Azure VM (Linux)

En algunas distribuciones de Linux hay una falta de concordancia de la [codificación Endiannes](https://en.wikipedia.org/wiki/Endianness) con el valor de VMUUID que proviene de Azure Resource Manager y lo que está almacenado en los registros de Azure Monitor. La siguiente consulta busca una concordancia en cualquier codificación Endiannes. Reemplace los valores de VMUUID por el formato de big-endian y little-endian del GUID para devolver correctamente los resultados. Para encontrar el VMUUID que se debe usar, ejecute la siguiente consulta en los registros de Azure Monitor: `Update | where Computer == "<machine name>"
| summarize by Computer, VMUUID`

#### <a name="missing-updates-summary"></a>Falta resumen de actualizaciones

```loganalytics
Update
| where TimeGenerated>ago(5h) and OSType=="Linux" and (VMUUID=~"625686a0-6d08-4810-aae9-a089e68d4911" or VMUUID=~"a0865662-086d-1048-aae9-a089e68d4911")
| summarize hint.strategy=partitioned arg_max(TimeGenerated, UpdateState, Classification) by Computer, SourceComputerId, Product, ProductArch
| where UpdateState=~"Needed"
| summarize by Product, ProductArch, Classification
| summarize allUpdatesCount=count(), criticalUpdatesCount=countif(Classification has "Critical"), securityUpdatesCount=countif(Classification has "Security"), otherUpdatesCount=countif(Classification !has "Critical" and Classification !has "Security")
```

#### <a name="missing-updates-list"></a>Falta lista de actualizaciones

```loganalytics
Update
| where TimeGenerated>ago(5h) and OSType=="Linux" and (VMUUID=~"625686a0-6d08-4810-aae9-a089e68d4911" or VMUUID=~"a0865662-086d-1048-aae9-a089e68d4911")
| summarize hint.strategy=partitioned arg_max(TimeGenerated, UpdateState, Classification, BulletinUrl, BulletinID) by Computer, SourceComputerId, Product, ProductArch
| where UpdateState=~"Needed"
| project-away UpdateState, TimeGenerated
| summarize computersCount=dcount(SourceComputerId, 2), ClassificationWeight=max(iff(Classification has "Critical", 4, iff(Classification has "Security", 2, 1))) by id=strcat(Product, "_", ProductArch), displayName=Product, productArch=ProductArch, classification=Classification, InformationId=BulletinID, InformationUrl=tostring(split(BulletinUrl, ";", 0)[0]), osType=1
| sort by ClassificationWeight desc, computersCount desc, displayName asc
| extend informationLink=(iff(isnotempty(InformationId) and isnotempty(InformationUrl), toobject(strcat('{ "uri": "', InformationUrl, '", "text": "', InformationId, '", "target": "blank" }')), toobject('')))
| project-away ClassificationWeight, InformationId, InformationUrl

```

### <a name="multi-vm-assessment-queries"></a>Consultas de evaluación de máquinas virtuales múltiples

#### <a name="computers-summary"></a>Resumen de equipos

```loganalytics
Heartbeat
| where TimeGenerated>ago(12h) and OSType=~"Windows" and notempty(Computer)
| summarize arg_max(TimeGenerated, Solutions) by SourceComputerId
| where Solutions has "updates"
| distinct SourceComputerId
| join kind=leftouter
(
    Update
    | where TimeGenerated>ago(14h) and OSType!="Linux"
    | summarize hint.strategy=partitioned arg_max(TimeGenerated, UpdateState, Approved, Optional, Classification) by SourceComputerId, UpdateID
    | distinct SourceComputerId, Classification, UpdateState, Approved, Optional
    | summarize WorstMissingUpdateSeverity=max(iff(UpdateState=~"Needed" and (Optional==false or Classification has "Critical" or Classification has "Security") and Approved!=false, iff(Classification has "Critical", 4, iff(Classification has "Security", 2, 1)), 0)) by SourceComputerId
)
on SourceComputerId
| extend WorstMissingUpdateSeverity=coalesce(WorstMissingUpdateSeverity, -1)
| summarize computersBySeverity=count() by WorstMissingUpdateSeverity
| union (Heartbeat
| where TimeGenerated>ago(12h) and OSType=="Linux" and notempty(Computer)
| summarize arg_max(TimeGenerated, Solutions) by SourceComputerId
| where Solutions has "updates"
| distinct SourceComputerId
| join kind=leftouter
(
    Update
    | where TimeGenerated>ago(5h) and OSType=="Linux"
    | summarize hint.strategy=partitioned arg_max(TimeGenerated, UpdateState, Classification) by SourceComputerId, Product, ProductArch
    | distinct SourceComputerId, Classification, UpdateState
    | summarize WorstMissingUpdateSeverity=max(iff(UpdateState=~"Needed", iff(Classification has "Critical", 4, iff(Classification has "Security", 2, 1)), 0)) by SourceComputerId
)
on SourceComputerId
| extend WorstMissingUpdateSeverity=coalesce(WorstMissingUpdateSeverity, -1)
| summarize computersBySeverity=count() by WorstMissingUpdateSeverity)
| summarize assessedComputersCount=sumif(computersBySeverity, WorstMissingUpdateSeverity>-1), notAssessedComputersCount=sumif(computersBySeverity, WorstMissingUpdateSeverity==-1), computersNeedCriticalUpdatesCount=sumif(computersBySeverity, WorstMissingUpdateSeverity==4), computersNeedSecurityUpdatesCount=sumif(computersBySeverity, WorstMissingUpdateSeverity==2), computersNeedOtherUpdatesCount=sumif(computersBySeverity, WorstMissingUpdateSeverity==1), upToDateComputersCount=sumif(computersBySeverity, WorstMissingUpdateSeverity==0)
| summarize assessedComputersCount=sum(assessedComputersCount), computersNeedCriticalUpdatesCount=sum(computersNeedCriticalUpdatesCount),  computersNeedSecurityUpdatesCount=sum(computersNeedSecurityUpdatesCount), computersNeedOtherUpdatesCount=sum(computersNeedOtherUpdatesCount), upToDateComputersCount=sum(upToDateComputersCount), notAssessedComputersCount=sum(notAssessedComputersCount)
| extend allComputersCount=assessedComputersCount+notAssessedComputersCount
```

#### <a name="missing-updates-summary"></a>Falta resumen de actualizaciones

```loganalytics
Update
| where TimeGenerated>ago(5h) and OSType=="Linux" and SourceComputerId in ((Heartbeat
| where TimeGenerated>ago(12h) and OSType=="Linux" and notempty(Computer)
| summarize arg_max(TimeGenerated, Solutions) by SourceComputerId
| where Solutions has "updates"
| distinct SourceComputerId))
| summarize hint.strategy=partitioned arg_max(TimeGenerated, UpdateState, Classification) by Computer, SourceComputerId, Product, ProductArch
| where UpdateState=~"Needed"
| summarize by Product, ProductArch, Classification
| union (Update
| where TimeGenerated>ago(14h) and OSType!="Linux" and (Optional==false or Classification has "Critical" or Classification has "Security") and SourceComputerId in ((Heartbeat
| where TimeGenerated>ago(12h) and OSType=~"Windows" and notempty(Computer)
| summarize arg_max(TimeGenerated, Solutions) by SourceComputerId
| where Solutions has "updates"
| distinct SourceComputerId))
| summarize hint.strategy=partitioned arg_max(TimeGenerated, UpdateState, Classification, Approved) by Computer, SourceComputerId, UpdateID
| where UpdateState=~"Needed" and Approved!=false
| summarize by UpdateID, Classification )
| summarize allUpdatesCount=count(), criticalUpdatesCount=countif(Classification has "Critical"), securityUpdatesCount=countif(Classification has "Security"), otherUpdatesCount=countif(Classification !has "Critical" and Classification !has "Security")
```

#### <a name="computers-list"></a>Lista de equipos

```loganalytics
Heartbeat
| where TimeGenerated>ago(12h) and OSType=="Linux" and notempty(Computer)
| summarize arg_max(TimeGenerated, Solutions, Computer, ResourceId, ComputerEnvironment, VMUUID) by SourceComputerId
| where Solutions has "updates"
| extend vmuuId=VMUUID, azureResourceId=ResourceId, osType=1, environment=iff(ComputerEnvironment=~"Azure", 1, 2), scopedToUpdatesSolution=true, lastUpdateAgentSeenTime=""
| join kind=leftouter
(
    Update
    | where TimeGenerated>ago(5h) and OSType=="Linux" and SourceComputerId in ((Heartbeat
    | where TimeGenerated>ago(12h) and OSType=="Linux" and notempty(Computer)
    | summarize arg_max(TimeGenerated, Solutions) by SourceComputerId
    | where Solutions has "updates"
    | distinct SourceComputerId))
    | summarize hint.strategy=partitioned arg_max(TimeGenerated, UpdateState, Classification, Product, Computer, ComputerEnvironment) by SourceComputerId, Product, ProductArch
    | summarize Computer=any(Computer), ComputerEnvironment=any(ComputerEnvironment), missingCriticalUpdatesCount=countif(Classification has "Critical" and UpdateState=~"Needed"), missingSecurityUpdatesCount=countif(Classification has "Security" and UpdateState=~"Needed"), missingOtherUpdatesCount=countif(Classification !has "Critical" and Classification !has "Security" and UpdateState=~"Needed"), lastAssessedTime=max(TimeGenerated), lastUpdateAgentSeenTime="" by SourceComputerId
    | extend compliance=iff(missingCriticalUpdatesCount > 0 or missingSecurityUpdatesCount > 0, 2, 1)
    | extend ComplianceOrder=iff(missingCriticalUpdatesCount > 0 or missingSecurityUpdatesCount > 0 or missingOtherUpdatesCount > 0, 1, 3)
)
on SourceComputerId
| project id=SourceComputerId, displayName=Computer, sourceComputerId=SourceComputerId, scopedToUpdatesSolution=true, missingCriticalUpdatesCount=coalesce(missingCriticalUpdatesCount, -1), missingSecurityUpdatesCount=coalesce(missingSecurityUpdatesCount, -1), missingOtherUpdatesCount=coalesce(missingOtherUpdatesCount, -1), compliance=coalesce(compliance, 4), lastAssessedTime, lastUpdateAgentSeenTime, osType=1, environment=iff(ComputerEnvironment=~"Azure", 1, 2), ComplianceOrder=coalesce(ComplianceOrder, 2)
| union(Heartbeat
| where TimeGenerated>ago(12h) and OSType=~"Windows" and notempty(Computer)
| summarize arg_max(TimeGenerated, Solutions, Computer, ResourceId, ComputerEnvironment, VMUUID) by SourceComputerId
| where Solutions has "updates"
| extend vmuuId=VMUUID, azureResourceId=ResourceId, osType=2, environment=iff(ComputerEnvironment=~"Azure", 1, 2), scopedToUpdatesSolution=true, lastUpdateAgentSeenTime=""
| join kind=leftouter
(
    Update
    | where TimeGenerated>ago(14h) and OSType!="Linux" and SourceComputerId in ((Heartbeat
    | where TimeGenerated>ago(12h) and OSType=~"Windows" and notempty(Computer)
    | summarize arg_max(TimeGenerated, Solutions) by SourceComputerId
    | where Solutions has "updates"
    | distinct SourceComputerId))
    | summarize hint.strategy=partitioned arg_max(TimeGenerated, UpdateState, Classification, Title, Optional, Approved, Computer, ComputerEnvironment) by Computer, SourceComputerId, UpdateID
    | summarize Computer=any(Computer), ComputerEnvironment=any(ComputerEnvironment), missingCriticalUpdatesCount=countif(Classification has "Critical" and UpdateState=~"Needed" and Approved!=false), missingSecurityUpdatesCount=countif(Classification has "Security" and UpdateState=~"Needed" and Approved!=false), missingOtherUpdatesCount=countif(Classification !has "Critical" and Classification !has "Security" and UpdateState=~"Needed" and Optional==false and Approved!=false), lastAssessedTime=max(TimeGenerated), lastUpdateAgentSeenTime="" by SourceComputerId
    | extend compliance=iff(missingCriticalUpdatesCount > 0 or missingSecurityUpdatesCount > 0, 2, 1)
    | extend ComplianceOrder=iff(missingCriticalUpdatesCount > 0 or missingSecurityUpdatesCount > 0 or missingOtherUpdatesCount > 0, 1, 3)
)
on SourceComputerId
| project id=SourceComputerId, displayName=Computer, sourceComputerId=SourceComputerId, scopedToUpdatesSolution=true, missingCriticalUpdatesCount=coalesce(missingCriticalUpdatesCount, -1), missingSecurityUpdatesCount=coalesce(missingSecurityUpdatesCount, -1), missingOtherUpdatesCount=coalesce(missingOtherUpdatesCount, -1), compliance=coalesce(compliance, 4), lastAssessedTime, lastUpdateAgentSeenTime, osType=2, environment=iff(ComputerEnvironment=~"Azure", 1, 2), ComplianceOrder=coalesce(ComplianceOrder, 2) )
| order by ComplianceOrder asc, missingCriticalUpdatesCount desc, missingSecurityUpdatesCount desc, missingOtherUpdatesCount desc, displayName asc
| project-away ComplianceOrder
```

#### <a name="missing-updates-list"></a>Falta lista de actualizaciones

```loganalytics
Update
| where TimeGenerated>ago(5h) and OSType=="Linux" and SourceComputerId in ((Heartbeat
| where TimeGenerated>ago(12h) and OSType=="Linux" and notempty(Computer)
| summarize arg_max(TimeGenerated, Solutions) by SourceComputerId
| where Solutions has "updates"
| distinct SourceComputerId))
| summarize hint.strategy=partitioned arg_max(TimeGenerated, UpdateState, Classification, BulletinUrl, BulletinID) by SourceComputerId, Product, ProductArch
| where UpdateState=~"Needed"
| project-away UpdateState, TimeGenerated
| summarize computersCount=dcount(SourceComputerId, 2), ClassificationWeight=max(iff(Classification has "Critical", 4, iff(Classification has "Security", 2, 1))) by id=strcat(Product, "_", ProductArch), displayName=Product, productArch=ProductArch, classification=Classification, InformationId=BulletinID, InformationUrl=tostring(split(BulletinUrl, ";", 0)[0]), osType=1
| union(Update
| where TimeGenerated>ago(14h) and OSType!="Linux" and (Optional==false or Classification has "Critical" or Classification has "Security") and SourceComputerId in ((Heartbeat
| where TimeGenerated>ago(12h) and OSType=~"Windows" and notempty(Computer)
| summarize arg_max(TimeGenerated, Solutions) by SourceComputerId
| where Solutions has "updates"
| distinct SourceComputerId))
| summarize hint.strategy=partitioned arg_max(TimeGenerated, UpdateState, Classification, Title, KBID, PublishedDate, Approved) by Computer, SourceComputerId, UpdateID
| where UpdateState=~"Needed" and Approved!=false
| project-away UpdateState, Approved, TimeGenerated
| summarize computersCount=dcount(SourceComputerId, 2), displayName=any(Title), publishedDate=min(PublishedDate), ClassificationWeight=max(iff(Classification has "Critical", 4, iff(Classification has "Security", 2, 1))) by id=strcat(UpdateID, "_", KBID), classification=Classification, InformationId=strcat("KB", KBID), InformationUrl=iff(isnotempty(KBID), strcat("https://support.microsoft.com/kb/", KBID), ""), osType=2)
| sort by ClassificationWeight desc, computersCount desc, displayName asc
| extend informationLink=(iff(isnotempty(InformationId) and isnotempty(InformationUrl), toobject(strcat('{ "uri": "', InformationUrl, '", "text": "', InformationId, '", "target": "blank" }')), toobject('')))
| project-away ClassificationWeight, InformationId, InformationUrl
```

## <a name="next-steps"></a>Pasos siguientes

* Para obtener más información sobre los registros de Azure Monitor, consulte [Registros de Azure Monitor](../../azure-monitor/log-query/log-query-overview.md).
* Para obtener ayuda con las alertas, consulte [Configuración de alertas](update-mgmt-configure-alerts.md).
