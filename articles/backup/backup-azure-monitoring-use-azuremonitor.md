---
title: Supervisión de Azure Backup con Azure Monitor
description: Supervise las cargas de trabajo de Azure Backup y cree alertas personalizadas mediante Azure Monitor.
ms.topic: conceptual
ms.date: 06/04/2019
ms.assetid: 01169af5-7eb0-4cb0-bbdb-c58ac71bf48b
ms.openlocfilehash: 3f5f663a2f0ed0f91cc414d352e975a2ff3b9649
ms.sourcegitcommit: ac7ae29773faaa6b1f7836868565517cd48561b2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88827161"
---
# <a name="monitor-at-scale-by-using-azure-monitor"></a>Supervisión a escala mediante Azure Monitor

Azure Backup proporciona [funcionalidades de supervisión y alerta integradas](backup-azure-monitoring-built-in-monitor.md) en un almacén de Recovery Services. Estas funcionalidades están disponibles sin ninguna infraestructura de administración adicional. Sin embargo, este servicio integrado está limitado en los escenarios siguientes:

- Si supervisa datos de varios almacenes de Recovery Services entre suscripciones
- Si el canal de notificación preferido *no* es el correo electrónico
- Si los usuarios desean alertas en más escenarios
- Si desea ver la información de un componente local, como System Center Data Protection Manager en Azure, que el portal no muestra en [**Trabajos de copia de seguridad**](backup-azure-monitoring-built-in-monitor.md#backup-jobs-in-recovery-services-vault) o [**Alertas de copia de seguridad**](backup-azure-monitoring-built-in-monitor.md#backup-alerts-in-recovery-services-vault)

## <a name="using-log-analytics-workspace"></a>Uso del área de trabajo de Log Analytics

### <a name="create-alerts-by-using-log-analytics"></a>Creación de alertas mediante Log Analytics

En Azure Monitor, puede crear sus propias alertas en un área de trabajo de Log Analytics. En el área de trabajo, usa *grupos de acciones de Azure* para seleccionar su mecanismo de notificación preferido.

> [!IMPORTANT]
> Para obtener información sobre el costo que supone la creación de esta consulta, consulte [Precios de Azure Monitor](https://azure.microsoft.com/pricing/details/monitor/).

Abra la sección **Registros** del área de trabajo de Log Analytics y cree una consulta para sus propios registros. Al seleccionar **Nueva regla de alertas**, se abre la página de creación de alertas de Azure Monitor, como se muestra en la siguiente imagen.

![Creación de una alerta en un área de trabajo de Log Analytics](media/backup-azure-monitoring-laworkspace/custom-alert.png)

El recurso ya está marcado aquí como área de trabajo de Log Analytics y se ha realizado la integración del grupos de acciones.

![Página de creación de alertas de Log Analytics](media/backup-azure-monitoring-laworkspace/inkedla-azurebackup-createalert.jpg)

#### <a name="alert-condition"></a>Condición de alerta

La característica que define una alerta es su condición desencadenadora. Seleccione **Condición** para cargar automáticamente la consulta de Kusto en la página **Registros**, como se muestra en la siguiente imagen. Aquí podrá editar la condición para satisfacer sus necesidades. Para obtener más información, consulte [Ejemplos de consultas de Kusto](#sample-kusto-queries).

![Configuración de una condición de alerta](media/backup-azure-monitoring-laworkspace/la-azurebackup-alertlogic.png)

Si es necesario, puede editar la consulta de Kusto. Elija un umbral, un período y una frecuencia. El umbral determina cuándo se generará la alerta. El período es el período de tiempo en el que se ejecuta la consulta. Por ejemplo, si el umbral es superior a 0, el período será de 5 minutos y, si la frecuencia equivale a ese tiempo, la regla ejecutará la consulta cada 5 minutos, lo que conlleva que se revisen los 5 minutos anteriores. Si el número de resultados es mayor que 0, se le notificará a través del grupo de acciones seleccionado.

> [!NOTE]
> Para ejecutar la regla de alerta una vez al día, en todos los eventos o registros que se crearon en el día en cuestión, cambie el valor de "período" y "frecuencia" a 1440, es decir, 24 horas.

#### <a name="alert-action-groups"></a>Grupos de acciones de alerta

Use un grupo de acciones para especificar un canal de notificación. Para ver los mecanismos de notificación disponibles, en **Grupos de acciones**, seleccione **Crear nuevo**.

![Mecanismos de notificación disponibles en la ventana "Agregar grupo de acciones"](media/backup-azure-monitoring-laworkspace/LA-AzureBackup-ActionGroup.png)

Puede satisfacer todos los requisitos de supervisión y alertas de Log Analytics solo, o puede usar Log Analytics para complementar las notificaciones integradas.

Para obtener más información, consulte [Creación, visualización y administración de alertas de registro mediante Azure Monitor](../azure-monitor/platform/alerts-log.md) y [Creación y administración de grupos de acciones en Azure Portal](../azure-monitor/platform/action-groups.md).

### <a name="sample-kusto-queries"></a>Ejemplos de consultas de Kusto

Los gráficos predeterminados le proporcionan consultas de Kusto para los escenarios básicos en los que se pueden crear alertas. También puede modificar las consultas para obtener los datos de los que desee recibir alertas. Pegue los siguientes ejemplos de consultas de Kusto en la página **Registros** y, a continuación, cree alertas basadas en las consultas:

- Todos los trabajos de copia de seguridad con resultado satisfactorio

    ````Kusto
    AddonAzureBackupJobs
    | where JobOperation=="Backup"
    | summarize arg_max(TimeGenerated,*) by JobUniqueId
    | where JobStatus=="Completed"
    ````

- Todos los trabajos de copia de seguridad con errores

    ````Kusto
    AddonAzureBackupJobs
    | where JobOperation=="Backup"
    | summarize arg_max(TimeGenerated,*) by JobUniqueId
    | where JobStatus=="Failed"
    ````

- Todos los trabajos de copia de seguridad de máquinas virtuales de Azure con resultado satisfactorio

    ````Kusto
    AddonAzureBackupJobs
    | where JobOperation=="Backup"
    | summarize arg_max(TimeGenerated,*) by JobUniqueId
    | where JobStatus=="Completed"
    | join kind=inner
    (
        CoreAzureBackup
        | where OperationName == "BackupItem"
        | where BackupItemType=="VM" and BackupManagementType=="IaaSVM"
        | distinct BackupItemUniqueId, BackupItemFriendlyName
    )
    on BackupItemUniqueId
    ````

- Todos los trabajos de copia de seguridad del registro de SQL con resultado satisfactorio

    ````Kusto
    AddonAzureBackupJobs
    | where JobOperation=="Backup" and JobOperationSubType=="Log"
    | summarize arg_max(TimeGenerated,*) by JobUniqueId
    | where JobStatus=="Completed"
    | join kind=inner
    (
        CoreAzureBackup
        | where OperationName == "BackupItem"
        | where BackupItemType=="SQLDataBase" and BackupManagementType=="AzureWorkload"
        | distinct BackupItemUniqueId, BackupItemFriendlyName
    )
    on BackupItemUniqueId
    ````

- Todos los trabajos del agente de Azure Backup con resultado satisfactorio

    ````Kusto
    AddonAzureBackupJobs
    | where JobOperation=="Backup"
    | summarize arg_max(TimeGenerated,*) by JobUniqueId
    | where JobStatus=="Completed"
    | join kind=inner
    (
        CoreAzureBackup
        | where OperationName == "BackupItem"
        | where BackupItemType=="FileFolder" and BackupManagementType=="MAB"
        | distinct BackupItemUniqueId, BackupItemFriendlyName
    )
    on BackupItemUniqueId
    ````

- Almacenamiento de copia de seguridad consumido por elemento de copia de seguridad

    ````Kusto
    CoreAzureBackup
    //Get all Backup Items
    | where OperationName == "BackupItem"
    //Get distinct Backup Items
    | distinct BackupItemUniqueId, BackupItemFriendlyName
    | join kind=leftouter
    (AddonAzureBackupStorage
    | where OperationName == "StorageAssociation"
    //Get latest record for each Backup Item
    | summarize arg_max(TimeGenerated, *) by BackupItemUniqueId
    | project BackupItemUniqueId , StorageConsumedInMBs)
    on BackupItemUniqueId
    | project BackupItemUniqueId , BackupItemFriendlyName , StorageConsumedInMBs
    | sort by StorageConsumedInMBs desc
    ````

### <a name="diagnostic-data-update-frequency"></a>Frecuencia de actualización de los datos de diagnóstico

Los datos de diagnóstico del almacén se bombean al área de trabajo de Log Analytics con cierto retraso. Cada evento llega al área de trabajo de Log Analytics *entre 20 y 30 minutos* después de insertarse desde el almacén de Recovery Services. Estos son detalles adicionales sobre el retardo:

- En todas las soluciones, las alertas integradas del servicio de copia de seguridad se insertan en cuanto se crean. Así pues, suelen aparecer en el área de trabajo de Log Analytics tras haber transcurrido de 20 a 30 minutos.
- En todas las soluciones, los trabajos de copia de seguridad a petición y los trabajos de restauración se insertan en cuanto *finalizan*.
- En todas las soluciones salvo la copia de seguridad SQL, los trabajos de copia de seguridad programados se insertan en cuanto *finalizan*.
- En el caso de la copia de seguridad de SQL, como pueden realizarse copias de seguridad de registros cada 15 minutos, de todos los trabajos de copia de seguridad programados completados, incluidos los registros, la información se une en lotes y se inserta cada 6 horas.
- En todas las soluciones, el resto de la información, como el elemento de copia de seguridad, la directiva, los puntos de recuperación, el almacenamiento, etc. se inserta al menos *una vez al día.*
- Un cambio en la configuración de copia de seguridad (como por ejemplo un cambio de directiva o la edición de la directiva) desencadena la inserción de toda la información de copia de seguridad relacionada.

## <a name="using-the-recovery-services-vaults-activity-logs"></a>Uso de los registros de actividad del almacén de Recovery Services

> [!CAUTION]
> Los siguientes pasos se aplican solo a las *copias de seguridad de máquinas virtuales de Azure.* No se pueden utilizar estos pasos para soluciones como el agente de Azure Backup, las copias de seguridad de SQL dentro de Azure o Azure Files.

Los registros de actividad también se pueden usar para recibir notificaciones de eventos, como copias de seguridad correctas. Para comenzar, siga estos pasos:

1. Inicie sesión en Azure Portal.
2. Abra el almacén de Recovery Services pertinente.
3. En las propiedades del almacén, abra la sección **Registro de actividad**.

Para identificar el registro adecuado y crear una alerta:

1. Compruebe que va a recibir registros de actividad por las copias de seguridad correctas aplicando los filtros que se muestran en la imagen siguiente. Cambie el valor **Intervalo de tiempo** según sea necesario para ver los registros.

   ![Filtrado para buscar los registros de actividad de las copias de seguridad de máquinas virtuales de Azure](media/backup-azure-monitoring-laworkspace/activitylogs-azurebackup-vmbackups.png)

2. Seleccione el nombre de la operación para ver los detalles pertinentes.
3. Seleccione **Nueva regla de alertas** para abrir la página **Crear regla**.
4. Cree una alerta siguiendo los pasos en [Crear, ver y administrar las alertas del registro de actividad mediante Azure Monitor](../azure-monitor/platform/alerts-activity-log.md).

   ![Nueva alerta de reglas](media/backup-azure-monitoring-laworkspace/new-alert-rule.png)

Aquí, el recurso es el propio almacén de Recovery Services. Repita los mismos pasos para todos los almacenes en los que desee recibir notificaciones a través de registros de actividad. La condición no tendrá un umbral, un período o una frecuencia, ya que esta alerta está basada en eventos. En cuanto se genera el registro de actividad pertinente, lo hace también la alerta.

## <a name="using-log-analytics-to-monitor-at-scale"></a>Uso de Log Analytics para supervisar a escala

Puede ver todas las alertas creadas a partir de registros de actividad y áreas de trabajo de Log Analytics en Azure Monitor. Solo tiene que abrir el panel **Alertas** situado a la izquierda.

Aunque puede recibir notificaciones a través de registros de actividad, recomendamos encarecidamente que se use Log Analytics en su lugar para supervisar a escala. Aquí se detallan los motivos:

- **Escenarios limitados**: las notificaciones a través de registros de actividad se aplican solo a las copias de seguridad de máquinas virtuales de Azure. Las notificaciones deben configurarse para todos los almacenes de Recovery Services.
- **Ajuste de definición**: la actividad de copia de seguridad programada no se ajusta a la definición más reciente de los registros de actividad. En su lugar, se alinea con los [registros de recursos](../azure-monitor/platform/resource-logs.md#send-to-log-analytics-workspace). Esta alineación da lugar a efectos inesperados al cambiar los datos que fluyen a través del canal del registro de actividad.
- **Problemas con el canal del registro de actividad**: en los almacenes de Recovery Services, los registros de actividad que se bombean desde Azure Backup siguen un nuevo modelo. Desafortunadamente, este cambio afecta a la generación de registros de actividad en Azure Government, Azure Alemania y Azure China 21Vianet. Si los usuarios de estos servicios en la nube crean o configuran cualquier alerta de los registros de actividad en Azure Monitor, no se desencadenarán las alertas. Además, en todas las regiones públicas de Azure, si un usuario [recopila los registros de actividad de Recovery Services en un área de trabajo de Log Analytics](../azure-monitor/platform/activity-log.md), estos registros no aparecerán.

Use un área de trabajo de Log Analytics para la supervisión y generación de alertas a escala en todas las cargas de trabajo que Azure Backup protege.

## <a name="next-steps"></a>Pasos siguientes

Para crear consultas personalizadas, consulte [Modelo de datos de Log Analytics](backup-azure-reports-data-model.md).
