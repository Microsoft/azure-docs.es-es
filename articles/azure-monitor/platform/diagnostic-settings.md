---
title: Creación de una configuración de diagnóstico para enviar los registros y las métricas de la plataforma a diferentes destinos
description: Envíe métricas y registros a Azure Monitor, Azure Storage o Azure Event Hubs utilizando la misma configuración de diagnóstico.
author: bwren
ms.author: bwren
services: azure-monitor
ms.topic: conceptual
ms.date: 04/27/2020
ms.subservice: logs
ms.openlocfilehash: ff0df654650bb1c32d5c3e9833ebde2a81e3d65c
ms.sourcegitcommit: fbb66a827e67440b9d05049decfb434257e56d2d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/05/2020
ms.locfileid: "87799963"
---
# <a name="create-diagnostic-settings-to-send-platform-logs-and-metrics-to-different-destinations"></a>Creación de una configuración de diagnóstico para enviar los registros y las métricas de la plataforma a diferentes destinos
Los [registros de plataforma](platform-logs-overview.md) de Azure, como los registros de recursos y los registros de actividad de Azure, proporcionan información de diagnóstico y auditoría detallada sobre los recursos de Azure y la plataforma de Azure de la que dependen. Las [métricas de plataforma](data-platform-metrics.md) se recopilan de forma predeterminada y suelen almacenarse en la base de datos de métricas de Azure Monitor. En este artículo, se explica cómo crear y establecer la configuración de diagnóstico para enviar métricas y registros de plataforma a diferentes destinos.

> [!IMPORTANT]
> Antes de crear una configuración de diagnóstico para el registro de actividad, debe deshabilitar cualquier configuración heredada. Para más información, consulte [Métodos de recopilación heredados](activity-log.md#legacy-collection-methods).

Cada recurso de Azure necesita su propia configuración de diagnóstico, que establece los siguientes criterios:

- Categorías de los datos de los registros y las métricas que se envían a los destinos definidos en la configuración. Las categorías disponibles varían para los distintos tipos de recursos.
- Uno o más destinos para enviar los registros. Los destinos actuales incluyen el área de trabajo de Log Analytics, Event Hubs y Azure Storage.

Cada configuración de diagnóstico puede definir un único destino. Si desea enviar datos a más de un tipo de destino determinado (por ejemplo, dos áreas de trabajo de Log Analytics diferentes), cree varias configuraciones. Cada recurso puede tener hasta 5 configuraciones de diagnóstico.

El siguiente vídeo describe cómo enrutar registros de plataforma con una configuración de diagnóstico.
> [!VIDEO https://www.microsoft.com/en-us/videoplayer/embed/RE4AvVO]

> [!NOTE]
> Las [métricas de plataforma](metrics-supported.md) se envían automáticamente a las [métricas de Azure Monitor](data-platform-metrics.md). Las configuraciones de diagnóstico se pueden usar para enviar métricas de determinados servicios de Azure a los registros de Azure Monitor a fin de analizarlas con otros datos de supervisión mediante [consultas de registro](../log-query/log-query-overview.md) con algunas limitaciones. 
>  
>  
> Actualmente no se admite el envío de métricas de varias dimensiones a través de la configuración de diagnóstico. Las métricas con dimensiones se exportan como métricas unidimensionales planas agregadas a través de los valores de dimensión. *Por ejemplo*: La métrica 'IOReadBytes' de una cadena de bloques puede consultarse y representarse individualmente en cada nodo. Sin embargo, cuando se exporta utilizando la configuración de diagnóstico, la métrica exportada representa los bytes de lectura de todos los nodos. Además, debido a limitaciones internas, no todas las métricas se exportan a los registros de Azure Monitor o Log Analytics. Para más información, consulte la [lista de métricas exportables](metrics-supported-export-diagnostic-settings.md). 
>  
>  
> Para solucionar estas limitaciones de métricas específicas, se recomienda extraerlas manualmente mediante la [API REST de métricas](/rest/api/monitor/metrics/list) e importarlas a los registros de Azure Monitor con [Data Collector API de Azure Monitor](data-collector-api.md).  


## <a name="destinations"></a>Destinations

Los registros y las métricas de plataforma se pueden enviar a los destinos de la tabla siguiente. Siga los vínculos de la tabla siguiente para más información sobre cómo enviar datos a ese destino.

| Destination | Descripción |
|:---|:---|
| [Área de trabajo de Log Analytics](#log-analytics-workspace) | El envío de registros y métricas a un área de trabajo de Log Analytics le permite analizar con otros datos de supervisión recopilados por Azure Monitor mediante consultas de registro eficaces, junto con otras características de Azure Monitor, como las alertas y las visualizaciones. |
| [Event Hubs](#event-hub) | El envío de registros y métricas a Event Hubs permite transmitir datos a sistemas externos, como SIEM de terceros y otras soluciones de análisis de registros. |
| [Cuenta de Almacenamiento de Azure](#azure-storage) | Archivar los registros y las métricas en una cuenta de almacenamiento de Azure resulta útil para realizar auditorías, análisis estáticos o copias de seguridad. En comparación con los registros de Azure Monitor y las áreas de trabajo de Log Analytics, Azure Storage es más económico y los registros se pueden mantener indefinidamente. |


## <a name="prerequisites"></a>Requisitos previos
Se deben crear todos los destinos para la configuración de diagnóstico con los permisos necesarios. Consulte las secciones siguientes para conocer los requisitos previos de cada destino.

### <a name="log-analytics-workspace"></a>Área de trabajo de Log Analytics
[Cree un área de trabajo nueva](../learn/quick-create-workspace.md) si no tiene ya una. El área de trabajo no tiene que estar en la misma suscripción que la del recurso que envía los registros, siempre que el usuario que realiza la configuración tenga el acceso RBAC adecuado a ambas suscripciones.

### <a name="event-hub"></a>Centro de eventos
Si no lo tiene, [cree un centro de eventos](../../event-hubs/event-hubs-create.md). El espacio de nombres de Event Hubs no tiene que estar necesariamente en la misma suscripción que la suscripción que emite los registros, siempre que el usuario que configure el valor tenga el acceso RBAC adecuado a ambas suscripciones y estas se encuentren en el mismo inquilino.

La directiva de acceso compartido del espacio de nombres define los permisos que tiene el mecanismo de transmisión. Para transmitir a Event Hubs, se necesitan permisos de administración, envío y escucha. Puede crear o modificar directivas de acceso compartido en la pestaña Configurar de Azure Portal para su espacio de nombres de Event Hubs. Para actualizar la configuración de diagnóstico para incluir la transmisión, debe tener el permiso ListKey sobre esa regla de autorización de Event Hubs. 


### <a name="azure-storage"></a>Almacenamiento de Azure
Si no la tiene, [cree una cuenta de Azure Storage](../../storage/common/storage-account-create.md). La cuenta de almacenamiento no tiene que estar en la misma suscripción que la del recurso que envía los registros, siempre que el usuario que realiza la configuración tenga el acceso RBAC adecuado a ambas suscripciones.

No debe utilizar una cuenta de almacenamiento existente que tenga otros datos sin supervisión almacenados en ella, para que pueda controlar mejor el acceso a los datos. Sin embargo, si va a archivar el registro de actividad y los registros de recurso, puede que tenga sentido utilizar esa misma cuenta de almacenamiento para mantener todos los datos de supervisión en una ubicación central.

Para enviar los datos al almacenamiento inmutable, establezca la directiva de inmutabilidad para la cuenta de almacenamiento tal como se escribe en [Establecimiento y administración de directivas de inmutabilidad para el almacenamiento de blobs](../../storage/blobs/storage-blob-immutability-policies-manage.md). Debe seguir todos los pasos que aparecen en este artículo, incluida la habilitación de las escrituras de blobs en anexos protegidos.

> [!NOTE]
> Actualmente, las cuentas de Azure Data Lake Storage Gen2 no se admiten como destino de la configuración de diagnóstico, aunque pueden aparecer como una opción válida en Azure Portal.



## <a name="create-in-azure-portal"></a>Creación en Azure Portal

Puede realizar configuraciones de diagnóstico en Azure Portal desde el menú de Azure Monitor o desde el menú del recurso.

1. El lugar de Azure Portal en el que se realiza la configuración de diagnóstico depende del recurso.

   - Si se trata de un único recurso, en el menú de este recurso, haga clic en la opción **Configuración de diagnóstico** de **Supervisar**.

        ![Configuración de diagnóstico](media/diagnostic-settings/menu-resource.png)

   - Si se trata de uno o varios recursos, en el menú de Azure Monitor, haga clic en la opción **Configuración de diagnóstico** de **Configuración** y después en el recurso.

      ![Configuración de diagnóstico](media/diagnostic-settings/menu-monitor.png)

   - En el registro de actividad, haga clic en la opción **Registro de actividad** del menú **Azure Monitor** y después en **Configuración de diagnóstico**. No olvide deshabilitar cualquier configuración heredada del registro de actividad. Consulte [Deshabilitar la configuración existente](./activity-log.md#legacy-collection-methods) para más información.

        ![Configuración de diagnóstico](media/diagnostic-settings/menu-activity-log.png)

2. Si no existe ninguna configuración en el recurso que ha seleccionado, se le pide que cree una. Haga clic en **Agregar configuración de diagnóstico**.

   ![Agregar configuración de diagnóstico: sin configuración actual](media/diagnostic-settings/add-setting.png)

   Si ya hay una configuración en el recurso, verá una lista de opciones que ya están configuradas. Puede hacer clic en **Agregar configuración de diagnóstico** para agregar una nueva configuración o **Editar configuración** para modificar una existente. Cada configuración no puede tener más de uno de los tipos de destino.

   ![Agregar configuración de diagnóstico: configuración actual](media/diagnostic-settings/edit-setting.png)

3. Asigne un nombre a la configuración, si aún no lo tiene.

    ![Incorporación de la configuración de diagnóstico](media/diagnostic-settings/setting-new-blank.png)

4. **Detalles de la categoría (qué enrutar)** : active la casilla de cada categoría de datos que desee enviar a los destinos que especificará más adelante. La lista de categorías es diferente en cada servicio de Azure.

     - **AllMetrics** enruta las métricas de plataforma de un recurso al almacén de registros de Azure, pero en forma de registro. Normalmente, estas métricas solo se envían a la base de datos de series temporales de métricas de Azure Monitor. Si se envían al almacén de registros de Azure Monitor (que se puede buscar mediante Log Analytics), tendrá que integrarlas en consultas que realicen búsquedas en otros registros. Es posible que esta opción no esté disponible en todos los tipos de recursos. Cuando sí está disponible, en las [métricas compatibles con Azure Monitor](metrics-supported.md), se indica qué métricas se recopilan para cada tipo de recurso.

       > [!NOTE]
       > Consulte las limitaciones para enrutar métricas a los registros de Azure Monitor anteriormente en este artículo.  


     - En **Registros**, se muestran las distintas categorías disponibles en función del tipo de recurso. Seleccione las categorías que desee enrutar a un destino.

5. **Detalles del destino**: active la casilla de cada destino. Cuando se activa una casilla, aparecen opciones que le permiten agregar información adicional.

      ![Envío a Log Analytics o centros de eventos](media/diagnostic-settings/send-to-log-analytics-event-hubs.png)

    1. **Log Analytics**: especifique la suscripción y el área de trabajo.  Si no tiene ningún área de trabajo, tendrá que [crear una antes de continuar](../learn/quick-create-workspace.md).

    1. **Centro de eventos**: especifique los criterios siguientes:
       - La suscripción de la que forma parte el centro de eventos.
       - El espacio de nombres del centro de eventos. Si aún no lo ha hecho, tendrá que [crear uno](../../event-hubs/event-hubs-create.md).
       - El nombre del centro de eventos (opcional) al que se van a enviar todos los datos. Si no especifica un nombre, se crea un centro de eventos para cada categoría de registro. Si va a enviar varias categorías, puede especificar un nombre para limitar el número de centros de eventos creados. Consulte [Cuotas y límites de Azure Event Hubs](../../event-hubs/event-hubs-quotas.md) para más detalles.
       - La directiva del centro de eventos (opcional), una directiva que defina los permisos que tiene el mecanismo de transmisión. Para más información, consulte este artículo sobre las [características de los centros de eventos](../../event-hubs/event-hubs-features.md#publisher-policy).

    1. **Storage**: elija una suscripción, una cuenta de almacenamiento y una directiva de retención.

        ![Enviar a Storage](media/diagnostic-settings/storage-settings-new.png)

        > [!TIP]
        > Considere la posibilidad de establecer la directiva de retención en 0 y eliminar manualmente los datos de almacenamiento utilizando un trabajo programado para evitar posibles confusiones en el futuro.
        >
        > En primer lugar, si está usando el almacenamiento para archivar, querrá conservar los datos durante más de 365 días. En segundo lugar, si elige una directiva de retención que sea mayor que 0, la fecha de expiración se asociará a los registros cuando se almacenen. No se puede cambiar la fecha de los registros una vez almacenados.
        >
        > Si establece la directiva de retención de *WorkflowRuntime* en 180 días y 24 horas después la establece en 365 días, los registros almacenados durante las primeras 24 horas se eliminarán automáticamente a los 180 días, mientras que todos los registros posteriores de ese mismo tipo se eliminarán automáticamente después de 365 días. Aunque posteriormente se modifique la directiva de retención, los registros que se almacenaron durante las 24 primeras horas no se conservarán 365 días.

6. Haga clic en **Save**(Guardar).

Transcurridos unos instantes, la nueva configuración aparece en la lista de configuraciones para este recurso y los registros se transmiten a los destinos especificados en cuanto se generan nuevos datos de eventos. Pueden pasar hasta quince minutos desde que se emite un evento hasta que [aparece en un área de trabajo de Log Analytics](data-ingestion-time.md).

## <a name="create-using-powershell"></a>Creación mediante PowerShell

Use el cmdlet [Set-AzDiagnosticSetting](/powershell/module/az.monitor/set-azdiagnosticsetting) para crear una configuración de diagnóstico con [Azure PowerShell](../samples/powershell-samples.md). Consulte la documentación de este cmdlet para las descripciones de sus parámetros.

> [!IMPORTANT]
> No puede usar este método con el registro de actividad de Azure. En su lugar, siga las indicaciones de [Creación de la configuración de diagnóstico en Azure Monitor con una plantilla de Resource Manager](diagnostic-settings-template.md) para crear una plantilla de Resource Manager e implementarla con PowerShell.

A continuación se muestra un cmdlet de PowerShell de ejemplo para crear una configuración de diagnóstico con los tres destinos.

```powershell
Set-AzDiagnosticSetting -Name KeyVault-Diagnostics -ResourceId /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/myresourcegroup/providers/Microsoft.KeyVault/vaults/mykeyvault -Category AuditEvent -MetricCategory AllMetrics -Enabled $true -StorageAccountId /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/myresourcegroup/providers/Microsoft.Storage/storageAccounts/mystorageaccount -WorkspaceId /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourcegroups/oi-default-east-us/providers/microsoft.operationalinsights/workspaces/myworkspace  -EventHubAuthorizationRuleId /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/myresourcegroup/providers/Microsoft.EventHub/namespaces/myeventhub/authorizationrules/RootManageSharedAccessKey
```

## <a name="create-using-azure-cli"></a>Creación mediante la CLI de Azure

Use el comando [az monitor diagnostic-settings create](/cli/azure/monitor/diagnostic-settings?view=azure-cli-latest#az-monitor-diagnostic-settings-create) para crear una configuración de diagnóstico con la [CLI de Azure](/cli/azure/monitor?view=azure-cli-latest). Consulte la documentación de este comando para las descripciones de sus parámetros.

> [!IMPORTANT]
> No puede usar este método con el registro de actividad de Azure. En su lugar, siga las indicaciones de [Creación de la configuración de diagnóstico en Azure Monitor con una plantilla de Resource Manager](diagnostic-settings-template.md) para crear una plantilla de Resource Manager e implementarla con la CLI.

A continuación se muestra un comando de la CLI de ejemplo para crear una configuración de diagnóstico con los tres destinos.

```azurecli
az monitor diagnostic-settings create  \
--name KeyVault-Diagnostics \
--resource /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/myresourcegroup/providers/Microsoft.KeyVault/vaults/mykeyvault \
--logs    '[{"category": "AuditEvent","enabled": true}]' \
--metrics '[{"category": "AllMetrics","enabled": true}]' \
--storage-account /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/myresourcegroup/providers/Microsoft.Storage/storageAccounts/mystorageaccount \
--workspace /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourcegroups/oi-default-east-us/providers/microsoft.operationalinsights/workspaces/myworkspace \
--event-hub-rule /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/myresourcegroup/providers/Microsoft.EventHub/namespaces/myeventhub/authorizationrules/RootManageSharedAccessKey
```

## <a name="create-using-resource-manager-template"></a>Creación mediante una plantilla de Resource Manager
Consulte [Ejemplos de plantillas de Resource Manager para la configuración de diagnóstico en Azure Monitor](../samples/resource-manager-diagnostic-settings.md) para crear o actualizar la configuración de diagnóstico con una plantilla de Resource Manager.

## <a name="create-using-rest-api"></a>Creación mediante la API REST
Consulte [Configuración de diagnóstico](/rest/api/monitor/diagnosticsettings) para crear o actualizar configuraciones de diagnóstico mediante la [API REST de Azure Monitor](/rest/api/monitor/).

## <a name="create-using-azure-policy"></a>Creación mediante Azure Policy
Dado que es necesario crear una configuración de diagnóstico para cada recurso de Azure, se puede usar Azure Policy para crear automáticamente una configuración de diagnóstico a medida que se crea cada recurso. Para más información, consulte [Implementación de Azure Monitor a escala mediante Azure Policy](../deploy-scale.md).


## <a name="next-steps"></a>Pasos siguientes

- [Más información sobre los registros de plataforma de Azure](platform-logs-overview.md)
