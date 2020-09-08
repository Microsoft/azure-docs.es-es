---
title: Uso de la configuración de diagnóstico en almacenes de Recovery Services
description: En este artículo, se explica cómo se utilizan los nuevos y los antiguos eventos de diagnóstico de Azure Backup.
ms.topic: conceptual
ms.date: 10/30/2019
ms.openlocfilehash: e5f666886dca0959b0f06b799088cadf4593ec39
ms.sourcegitcommit: ac7ae29773faaa6b1f7836868565517cd48561b2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88826674"
---
# <a name="use-diagnostics-settings-for-recovery-services-vaults"></a>Uso de la configuración de diagnóstico en almacenes de Recovery Services

Azure Backup envía eventos de diagnóstico que se pueden recopilar y usar para elaborar análisis, alertas e informes.

Puede usar Azure Portal para configurar los valores de diagnóstico de los almacenes de Recovery Services. Para ello, vaya al almacén y seleccione **Configuración de diagnóstico**. Si selecciona **+ Agregar configuración de diagnóstico**, podrá enviar uno o varios eventos de diagnóstico a una cuenta de almacenamiento, a un centro de eventos o a un área de trabajo de Log Analytics.

![Panel Configuración de diagnóstico](./media/backup-azure-diagnostics-events/diagnostics-settings-blade.png)

## <a name="diagnostics-events-available-for-azure-backup-users"></a>Eventos de diagnóstico disponibles para usuarios de Azure Backup

Azure Backup proporciona los siguientes eventos de diagnóstico. Cada evento proporciona datos detallados sobre un conjunto específico de artefactos relacionados con la copia de seguridad:

* CoreAzureBackup
* AddonAzureBackupAlerts
* AddonAzureBackupProtectedInstance
* AddonAzureBackupJobs
* AddonAzureBackupPolicy
* AddonAzureBackupStorage

Si sigue usando el [evento heredado](#legacy-event) AzureBackupReport, se recomienda cambiar al uso de los eventos anteriores.

Para más información, consulte [Modelo de datos para eventos de diagnóstico de Azure Backup](./backup-azure-reports-data-model.md).

Los datos de estos eventos se pueden enviar a una cuenta de almacenamiento, al área de trabajo de Log Analytics o a un centro de eventos. Si va a enviar estos datos a un área de trabajo de Log Analytics, sitúe el alternador en **Específico del recurso** en la pantalla **Configuración de diagnóstico**. Para más información, consulte las siguientes secciones:

## <a name="use-diagnostics-settings-with-log-analytics"></a>Uso de la configuración de diagnóstico con Log Analytics

Ahora, puede usar Azure Backup para enviar datos de diagnóstico de los almacenes a tablas de Log Analytics dedicadas a la copia de seguridad. Estas tablas se denominan [tablas específicas de recursos](../azure-monitor/platform/resource-logs.md#send-to-log-analytics-workspace).

Para enviar los datos de diagnóstico de un almacén a Log Analytics:

1. Acceda al almacén y seleccione **Configuración de diagnóstico**. Seleccione **+ Agregar conjunto de diagnóstico**.
1. Asigne un nombre a la configuración de diagnóstico.
1. Active la casilla **Enviar a Log Analytics** y seleccione un área de trabajo de Log Analytics.
1. Sitúe el alternador en **Específico del recurso** y seleccione los seis eventos siguientes: **CoreAzureBackup**, **AddonAzureBackupJobs**, **AddonAzureBackupAlerts**, **AddonAzureBackupPolicy**, **AddonAzureBackupStorage** y **AddonAzureBackupProtectedInstance**.
1. Seleccione **Guardar**.

   ![Modo específico del recurso](./media/backup-azure-diagnostics-events/resource-specific-blade.png)

Cuando los datos se transmiten al área de trabajo de Log Analytics, se crean tablas dedicadas para cada uno de esos eventos en el área de trabajo. Puede consultar directamente cualquiera de esas tablas. Si es necesario, también puede realizar combinaciones o uniones entre las tablas.

> [!IMPORTANT]
> En los [informes de Backup](./configure-reports.md), los seis eventos anteriores (CoreAzureBackup, AddonAzureBackupJobs, AddonAzureBackupAlerts, AddonAzureBackupPolicy, AddonAzureBackupStorage y AddonAzureBackupProtectedInstance) *solo* pueden utilizarse en el modo específico del recurso. *Si intenta enviar datos de estos seis eventos en el modo de diagnóstico de Azure, estos datos no podrán verse en los informes de Backup.*

## <a name="legacy-event"></a>Evento heredado

Tradicionalmente, todos los datos de diagnóstico relacionados con las copias de seguridad de un almacén se incluían en un solo evento llamado AzureBackupReport. En realidad, los seis eventos que se describen anteriormente son un desglose de todos los datos de AzureBackupReport.

En la actualidad, aún se permite utilizar el evento AzureBackupReport para garantizar la compatibilidad con versiones anteriores en aquellos casos en los que los usuarios tienen consultas personalizadas existentes relacionadas con este evento; por ejemplo, alertas de registro y visualizaciones personalizadas. *Es conveniente que actualice a los [nuevos eventos](#diagnostics-events-available-for-azure-backup-users) lo antes posible*. Estos nuevos eventos:

* Facilitan enormemente el uso de datos en las consultas de registros.
* Proporcionan una mayor capacidad de detección de los esquemas y su estructura.
* Mejoran el rendimiento tanto de la latencia de ingesta como de los tiempos de consulta.

*El evento heredado del modo de diagnóstico de Azure terminará quedando en desuso. Si decide utilizar los nuevos eventos, evitará las migraciones complejas en el futuro*. Los datos del evento heredado también van a dejar de admitirse en nuestra [solución de generación informes](./configure-reports.md), que utiliza Log Analytics.

### <a name="steps-to-move-to-new-diagnostics-settings-for-a-log-analytics-workspace"></a>Pasos para cambiar a la nueva configuración de diagnóstico en un área de trabajo de Log Analytics

1. Identifique qué almacenes envían datos a las áreas de trabajo de Log Analytics utilizando el evento heredado y las suscripciones a las que pertenece. Ejecute las siguientes consultas en cada una de sus áreas de trabajo para identificar estos almacenes y suscripciones.

    ````Kusto
    let RangeStart = startofday(ago(3d));
    let VaultUnderAzureDiagnostics = (){
        AzureDiagnostics
        | where TimeGenerated >= RangeStart | where Category == "AzureBackupReport" and OperationName == "Vault" and SchemaVersion_s == "V2"
        | summarize arg_max(TimeGenerated, *) by ResourceId
        | project ResourceId, Category};
    let VaultUnderResourceSpecific = (){
        CoreAzureBackup
        | where TimeGenerated >= RangeStart | where OperationName == "Vault"
        | summarize arg_max(TimeGenerated, *) by ResourceId
        | project ResourceId, Category};
        // Some Workspaces will not have AzureDiagnostics Table, so you need to use isFuzzy
    let CombinedVaultTable = (){
        union isfuzzy = true
        (VaultUnderAzureDiagnostics() ),
        (VaultUnderResourceSpecific() )
        | distinct ResourceId, Category};
    CombinedVaultTable | where Category == "AzureBackupReport"
    | join kind = leftanti (
    CombinedVaultTable | where Category == "CoreAzureBackup"
    ) on ResourceId
    | parse ResourceId with * "SUBSCRIPTIONS/" SubscriptionId:string "/RESOURCEGROUPS" * "MICROSOFT.RECOVERYSERVICES/VAULTS/" VaultName:string
    | project ResourceId, SubscriptionId, VaultName
    ````

    A continuación se muestra una captura de pantalla de la consulta en ejecución en una de las áreas de trabajo:

    ![Consulta del área de trabajo](./media/backup-azure-diagnostics-events/workspace-query.png)

2. Use la [directiva de Azure Policy integrada](./azure-policy-configure-diagnostics.md) en Azure Backup para agregar una nueva configuración de diagnóstico en todos los almacenes del ámbito especificado. Esta directiva agrega una nueva configuración de diagnóstico a los almacenes que no cuentan con una configuración de diagnóstico o que solo disponen de una configuración de diagnóstico heredada. Esta directiva se puede asignar a una suscripción entera o a un grupo de recursos cada vez. Tenga en cuenta que necesitará acceso de "propietario" a cada suscripción en la que se asigne la directiva.

Si lo desea, puede optar por utilizar una configuración de diagnóstico diferente para AzureBackupReport y los seis eventos nuevos hasta que haya migrado todas las consultas personalizadas y se utilicen los datos de las nuevas tablas. En la imagen siguiente, se muestra un ejemplo de un almacén que tiene dos configuraciones de diagnóstico. La primera configuración, **Setting1**, envía datos del evento AzureBackupReport a un área de trabajo de Log Analytics en el modo de diagnóstico de Azure. La segunda configuración, **Setting2**, envía datos de los seis nuevos eventos de Azure Backup nuevos a un área de trabajo de Log Analytics en el modo específico del recurso.

![Dos configuraciones](./media/backup-azure-diagnostics-events/two-settings-example.png)

> [!IMPORTANT]
> El evento AzureBackupReport *solo* puede utilizarse en modo de diagnóstico de Azure. *Si intenta enviar datos de este evento en el modo específico del recurso, los datos no se transmitirán al área de trabajo de Log Analytics.*

> [!NOTE]
> El alternador de **Diagnósticos de Azure** o **Específico del recurso** solo aparece si el usuario selecciona **Enviar a Log Analytics**. Para enviar datos a una cuenta de almacenamiento o a un centro de eventos, el usuario solo tiene que seleccionar el destino correspondiente y activar las casillas de los eventos que desee. No es necesario especificar ningún otro dato. Una vez más, le recomendamos que de ahora en adelante no elija el evento heredado AzureBackupReport.

## <a name="send-azure-site-recovery-events-to-log-analytics"></a>Envío de eventos de Azure Site Recovery a Log Analytics

Los eventos de Azure Backup y Azure Site Recovery se envían desde el mismo almacén de Recovery Services. Actualmente, Azure Site Recovery no puede utilizarse con tablas específicas del recurso. Los usuarios que deseen enviar eventos de Azure Site Recovery a Log Analytics *solo* deben utilizar el modo de diagnóstico de Azure, tal y como se muestra en la imagen. *Si elige el modo específico del recurso con los eventos de Azure Site Recovery, no se enviarán los datos al área de trabajo de Log Analytics*.

![Eventos de Site Recovery](./media/backup-azure-diagnostics-events/site-recovery-settings.png)

Resumiendo:

* Si tiene diagnósticos de Log Analytics configurados con Azure Diagnostics y ha creado consultas personalizadas basadas en estos diagnósticos, *no modifique* esta configuración hasta que migre las consultas y usen los datos de los nuevos eventos.
* Si, tal y como se recomienda, también quiere comenzar a utilizar las nuevas tablas, cree una **nueva** configuración de diagnóstico, seleccione **Específico del recurso** y elija los seis eventos nuevos.
* Si actualmente envía eventos de Azure Site Recovery a Log Analytics, *no* elija el modo específico del recurso para estos eventos. De lo contrario, los datos de estos eventos no se enviarán al área de trabajo de Log Analytics. En su lugar, cree otra configuración de diagnóstico, seleccione **Diagnósticos de Azure** y elija los eventos de Azure Site Recovery pertinentes.

En la imagen siguiente, se muestra un ejemplo de un usuario que tiene tres configuraciones de diagnóstico en un almacén. La primera configuración, **Setting1**, envía datos del evento AzureBackupReport a un área de trabajo de Log Analytics en el modo de diagnósticos de Azure. La segunda configuración, **Setting2**, envía datos de los seis nuevos eventos de Azure Backup a un área de trabajo de Log Analytics en el modo específico del recurso. La tercera configuración, **Setting3**, envía datos de los eventos de Azure Site Recovery a un área de trabajo de Log Analytics en el modo de diagnósticos de Azure.

![Tres configuraciones](./media/backup-azure-diagnostics-events/three-settings-example.png)

## <a name="next-steps"></a>Pasos siguientes

[Más información sobre el modelo de datos de Log Analytics para eventos de diagnóstico](./backup-azure-reports-data-model.md)
