---
title: Ejemplos de PowerShell de Azure Monitor
description: Use PowerShell para acceder a las características de Azure Monitor, como el escalado automático, alertas, webhooks y buscar registros de actividad.
ms.subservice: ''
ms.topic: sample
author: bwren
ms.author: bwren
ms.date: 2/14/2018
ms.openlocfilehash: 520022be8ee2054d6c0c89ee3f027de9094ae1af
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87055263"
---
# <a name="azure-monitor-powershell-samples"></a>Ejemplos de PowerShell de Azure Monitor
En este artículo se muestran comandos de PowerShell de ejemplo para ayudarle a acceder a las características de Azure Monitor.

> [!NOTE]
> Azure Monitor es el nuevo nombre de lo que se conocía como "Azure Insights" hasta el 25 de septiembre de 2016. Sin embargo, los espacios de nombres y, por tanto, los siguientes comandos, aún contienen el término *insights*.

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

## <a name="set-up-powershell"></a>Configurar PowerShell
Si no lo ha hecho ya, configure PowerShell para que se ejecute en el equipo. Para más información, consulte el artículo de [instalación y configuración de PowerShell](/powershell/azure/).

## <a name="examples-in-this-article"></a>Ejemplos de este artículo
Los ejemplos de este artículo muestran cómo puede usar cmdlets de Azure Monitor. También puede consultar toda la lista de cmdlets de PowerShell de Azure Monitor en [Azure Monitor Cmdlets](/powershell/module/az.applicationinsights) (Cmdlets de Azure Monitor).

## <a name="sign-in-and-use-subscriptions"></a>Inicio de sesión y uso de suscripciones
Primero, inicie sesión en la suscripción de Azure.

```powershell
Connect-AzAccount
```

Verá una pantalla de inicio de sesión. Una vez que inicie sesión, se muestran el identificador predeterminado de la suscripción, el identificador de inquilino y la cuenta. Todos los cmdlets de Azure funcionan en el contexto de la suscripción predeterminada. Para ver la lista de suscripciones a las que tiene acceso, use el siguiente comando:

```powershell
Get-AzSubscription
```

Para ver el contexto de trabajo (en qué suscripción se ejecutan los comandos), use el siguiente comando:

```powershell
Get-AzContext
```
Para cambiar el contexto de trabajo a una suscripción diferente, utilice el siguiente comando:

```powershell
Set-AzContext -SubscriptionId <subscriptionid>
```


## <a name="retrieve-activity-log"></a>Recuperación del registro de actividad
Use el cmdlet [Get AzLog](/powershell/module/az.monitor/get-azlog).  A continuación se muestran algunos ejemplos comunes. El registro de actividad contiene los últimos 90 días de operaciones. El uso de las fechas anteriores a ese período provoca un mensaje de error.  

Vea cuál es la fecha y hora actual para comprobar qué períodos se deben usar en el siguiente comando:
```powershell
Get-Date
```

Obtención de entradas de registro desde esta fecha y hora hasta la actual:

```powershell
Get-AzLog -StartTime 2019-03-01T10:30
```

Obtención de entradas de registro entre un intervalo de fecha y hora:

```powershell
Get-AzLog -StartTime 2019-01-01T10:30 -EndTime 2015-01-01T11:30
```

Obtención de entradas de registro de un grupo de recursos específico:

```powershell
Get-AzLog -ResourceGroup 'myrg1'
```

Obtención de entradas de registro de un proveedor de recursos específico entre un intervalo de fecha y hora:

```powershell
Get-AzLog -ResourceProvider 'Microsoft.Web' -StartTime 2015-01-01T10:30 -EndTime 2015-01-01T11:30
```

Obtener todas las entradas de registro con un llamador concreto:

```powershell
Get-AzLog -Caller 'myname@company.com'
```

El comando siguiente recupera los últimos 1000 eventos desde el registro de actividades:

```powershell
Get-AzLog -MaxRecord 10
```

`Get-AzLog` es compatible con muchos otros parámetros. Consulte la referencia `Get-AzLog` para obtener más información.

> [!NOTE]
> `Get-AzLog` solo proporciona 15 días de historial. El parámetro **-MaxRecords** permite consultar los últimos eventos N más allá de 15 días. Para acceder a eventos de hace más de 15 días, use el SDK o la API de REST (ejemplo de C# usando el SDK). Si no incluye **StartTime**, el valor predeterminado será **EndTime** menos una hora. Si no incluye **EndTime**, el valor predeterminado será la hora actual. Todas las horas se muestran en UTC.
> 
> 

## <a name="retrieve-alerts-history"></a>Recuperación del historial de alertas
Para ver todos los eventos de alerta, puede consultar los registros de Azure Resource Manager con los ejemplos siguientes.

```powershell
Get-AzLog -Caller "Microsoft.Insights/alertRules" -DetailedOutput -StartTime 2015-03-01
```

Para ver el historial de una regla de alerta específica, puede utilizar el cmdlet `Get-AzAlertHistory`, con lo que se transmitirá el id. de recurso de la regla de alerta.

```powershell
Get-AzAlertHistory -ResourceId /subscriptions/s1/resourceGroups/rg1/providers/microsoft.insights/alertrules/myalert -StartTime 2016-03-1 -Status Activated
```

El cmdlet `Get-AzAlertHistory` admite varios parámetros. Puede obtener más información en [Get-AlertHistory](/previous-versions/azure/mt282453(v=azure.100)).

## <a name="retrieve-information-on-alert-rules"></a>Recuperación de información sobre reglas de alerta
Todos los comandos siguientes se realizan en un grupo de recursos denominado "montest".

Consulta de todas las propiedades de la regla de alerta:

```powershell
Get-AzAlertRule -Name simpletestCPU -ResourceGroup montest -DetailedOutput
```

Recuperación de todas las alertas de un grupo de recursos:

```powershell
Get-AzAlertRule -ResourceGroup montest
```

Recuperación de todas las reglas de alerta de un recurso de destino. Por ejemplo, todas las reglas de alerta se establecen en una máquina virtual.

```powershell
Get-AzAlertRule -ResourceGroup montest -TargetResourceId /subscriptions/s1/resourceGroups/montest/providers/Microsoft.Compute/virtualMachines/testconfig
```

`Get-AzAlertRule` es compatible con otros parámetros. Consulte [Get AlertRule](/previous-versions/azure/mt282459(v=azure.100)) para obtener más información.

## <a name="create-metric-alerts"></a>Creación de alertas de métrica
Puede usar el cmdlet `Add-AlertRule` para crear, actualizar o deshabilitar una regla de alerta.

Puede crear propiedades de correo electrónico y webhook mediante `New-AzAlertRuleEmail` y `New-AzAlertRuleWebhook` respectivamente. En el cmdlet de regla de alerta, asigne estas propiedades como acciones a la propiedad **Actions** de la regla de alerta.

En la tabla siguiente se describen los parámetros y valores utilizados para crear una alerta con una métrica.

| parámetro | value |
| --- | --- |
| Nombre |simpletestdiskwrite |
| Ubicación (Location) de esta regla de alerta |Este de EE. UU. |
| ResourceGroup |montest |
| TargetResourceId |/subscriptions/s1/resourceGroups/montest/providers/Microsoft.Compute/virtualMachines/testconfig |
| Nombre de la métrica (MetricName) de la alerta que se crea |\PhysicalDisk(_Total)\Disk Writes/sec. See the `Get-MetricDefinitions` cmdlet about how to retrieve the exact metric names |
| operator |GreaterThan |
| Valor de umbral (número por segundo para esta métrica) |1 |
| WindowSize (formato hh:mm:ss) |00:05:00 |
| aggregator (estadística de la métrica que, en este caso, usa el recuento medio) |Average |
| mensajes de correo electrónico personalizados (matriz de cadenas) |'foo@example.com','bar@example.com' |
| enviar correo electrónico a los propietarios, colaboradores y lectores |-SendToServiceOwners |

Creación de una acción de correo electrónico

```powershell
$actionEmail = New-AzAlertRuleEmail -CustomEmail myname@company.com
```

Crear una acción de Webhook

```powershell
$actionWebhook = New-AzAlertRuleWebhook -ServiceUri https://example.com?token=mytoken
```

Creación de la regla de alerta de la métrica CPU% en una máquina virtual clásica

```powershell
Add-AzMetricAlertRule -Name vmcpu_gt_1 -Location "East US" -ResourceGroup myrg1 -TargetResourceId /subscriptions/s1/resourceGroups/myrg1/providers/Microsoft.ClassicCompute/virtualMachines/my_vm1 -MetricName "Percentage CPU" -Operator GreaterThan -Threshold 1 -WindowSize 00:05:00 -TimeAggregationOperator Average -Action $actionEmail, $actionWebhook -Description "alert on CPU > 1%"
```

Recuperación de la regla de alerta

```powershell
Get-AzAlertRule -Name vmcpu_gt_1 -ResourceGroup myrg1 -DetailedOutput
```

El cmdlet Add alert también actualiza la regla si ya existe una para las propiedades determinadas. Para deshabilitar una regla de alerta, incluya el parámetro **-DisableRule**.

## <a name="get-a-list-of-available-metrics-for-alerts"></a>Obtención de una lista de métricas disponibles para las alertas
Puede usar el cmdlet `Get-AzMetricDefinition` para ver la lista de todas las métricas de un recurso específico.

```powershell
Get-AzMetricDefinition -ResourceId <resource_id>
```

En el ejemplo siguiente se genera una tabla con el nombre y la unidad de la métrica.

```powershell
Get-AzMetricDefinition -ResourceId <resource_id> | Format-Table -Property Name,Unit
```

Hay disponible una lista completa de opciones para `Get-AzMetricDefinition` en [Get-MetricDefinitions](/previous-versions/azure/mt282458(v=azure.100)).

## <a name="create-and-manage-activity-log-alerts"></a>Creación y administración de alertas del registro de actividades
Puede usar el cmdlet `Set-AzActivityLogAlert` para establecer una alerta de registro de actividad. Una alerta de registro de actividad requiere que primero defina las condiciones como un diccionario de condiciones, y luego cree una alerta que utilice esas condiciones.

```powershell

$condition1 = New-AzActivityLogAlertCondition -Field 'category' -Equal 'Administrative'
$condition2 = New-AzActivityLogAlertCondition -Field 'operationName' -Equal 'Microsoft.Compute/virtualMachines/write'
$additionalWebhookProperties = New-Object "System.Collections.Generic.Dictionary``2[System.String,System.String]"
$additionalWebhookProperties.Add('customProperty', 'someValue')
$actionGrp1 = New-AzActionGroup -ActionGroupId '/subscriptions/<subid>/providers/Microsoft.Insights/actiongr1' -WebhookProperty $additionalWebhookProperties
Set-AzActivityLogAlert -Location 'Global' -Name 'alert on VM create' -ResourceGroupName 'myResourceGroup' -Scope '/subscriptions/<subid>' -Action $actionGrp1 -Condition $condition1, $condition2

```

Las propiedades de webhook adicionales son opcionales. Puede volver el contenido de una alerta de registro de actividad con `Get-AzActivityLogAlert`.

## <a name="create-and-manage-autoscale-settings"></a>Creación y administración de la configuración de escalado automático

> [!NOTE] 
> Para Cloud Services (Microsoft.ClassicCompute), el escalado automático admite un intervalo de tiempo de 5 minutos (PT5M). En el caso de los demás servicios, el escalado automático admite un intervalo de tiempo mínimo de 1 minuto (PT1M).

Los recursos (aplicaciones web, las máquinas virtuales, los servicios en la nube o los conjuntos de escalado de máquinas virtuales) solo pueden tener una configuración de escalado automático.
Sin embargo, cada configuración de escalado automático puede tener varios perfiles. Por ejemplo, uno para un perfil de escalado basado en el rendimiento y otro para uno basado en la programación. Cada perfil puede tener varias reglas configuradas. Para obtener más información sobre el escalado automático, consulte [Escalado automático de una aplicación](../../cloud-services/cloud-services-how-to-scale-portal.md).

Estos son los pasos que debe seguir:

1. Cree reglas.
2. Cree perfiles que asignen las reglas que ha creado anteriormente a los perfiles.
3. Opcional: cree notificaciones de escalado automático configurando las propiedades de Webhook y correo electrónico.
4. Cree una configuración de escalado automático con un nombre en el recurso de destino mediante la asignación de los perfiles y las notificaciones que creó en los pasos anteriores.

En los ejemplos siguientes se muestra cómo crear una configuración de escalado automático para un conjunto de escalado de máquinas virtuales de un sistema operativo de Windows mediante la métrica de uso de CPU.

En primer lugar, cree una regla de escalado horizontal con aumento del recuento de instancias.

```powershell
$rule1 = New-AzAutoscaleRule -MetricName "Percentage CPU" -MetricResourceId /subscriptions/s1/resourceGroups/big2/providers/Microsoft.Compute/virtualMachineScaleSets/big2 -Operator GreaterThan -MetricStatistic Average -Threshold 60 -TimeGrain 00:01:00 -TimeWindow 00:10:00 -ScaleActionCooldown 00:10:00 -ScaleActionDirection Increase -ScaleActionValue 1
```        

Después, cree una regla de reducción de escalado con una disminución del recuento de instancias.

```powershell
$rule2 = New-AzAutoscaleRule -MetricName "Percentage CPU" -MetricResourceId /subscriptions/s1/resourceGroups/big2/providers/Microsoft.Compute/virtualMachineScaleSets/big2 -Operator GreaterThan -MetricStatistic Average -Threshold 30 -TimeGrain 00:01:00 -TimeWindow 00:10:00 -ScaleActionCooldown 00:10:00 -ScaleActionDirection Decrease -ScaleActionValue 1
```

Después, cree un perfil para las reglas.

```powershell
$profile1 = New-AzAutoscaleProfile -DefaultCapacity 2 -MaximumCapacity 10 -MinimumCapacity 2 -Rules $rule1,$rule2 -Name "My_Profile"
```

Cree una propiedad de Webhook.

```powershell
$webhook_scale = New-AzAutoscaleWebhook -ServiceUri "https://example.com?mytoken=mytokenvalue"
```

Cree la propiedad de notificación para la configuración de escalado automático, incluidas las de correo electrónico y Webhook que creó anteriormente.

```powershell
$notification1= New-AzAutoscaleNotification -CustomEmails ashwink@microsoft.com -SendEmailToSubscriptionAdministrators SendEmailToSubscriptionCoAdministrators -Webhooks $webhook_scale
```

Finalmente, cree la configuración de escalado automático para agregar el perfil que creó antes. 

```powershell
Add-AzAutoscaleSetting -Location "East US" -Name "MyScaleVMSSSetting" -ResourceGroup big2 -TargetResourceId /subscriptions/s1/resourceGroups/big2/providers/Microsoft.Compute/virtualMachineScaleSets/big2 -AutoscaleProfiles $profile1 -Notifications $notification1
```

Para obtener más información sobre cómo administrar la configuración de escalado automático, consulte [Get-AutoscaleSetting](/previous-versions/azure/mt282461(v=azure.100)).

## <a name="autoscale-history"></a>Historial de escalado automático
En el ejemplo siguiente se muestra cómo puede ver los eventos de alerta y escalado automático recientes. Utilice la búsqueda de registros de actividades para ver el historial de escalado automático.

```powershell
Get-AzLog -Caller "Microsoft.Insights/autoscaleSettings" -DetailedOutput -StartTime 2015-03-01
```

Puede usar el cmdlet `Get-AzAutoScaleHistory` para recuperar el historial de escalado automático.

```powershell
Get-AzAutoScaleHistory -ResourceId /subscriptions/s1/resourceGroups/myrg1/providers/microsoft.insights/autoscalesettings/myScaleSetting -StartTime 2016-03-15 -DetailedOutput
```

Para obtener más información, consulte [Get-AutoscaleHistory](/previous-versions/azure/mt282464(v=azure.100)).

### <a name="view-details-for-an-autoscale-setting"></a>Consulta de los detalles de una configuración de escalado automático
Puede usar el cmdlet `Get-Autoscalesetting` para recuperar más información sobre la configuración de escalado automático.

En el ejemplo siguiente se muestra información sobre todas las configuraciones de escalado automático del grupo de recursos "myrg1".

```powershell
Get-AzAutoscalesetting -ResourceGroup myrg1 -DetailedOutput
```

En el ejemplo siguiente se muestra información sobre todas las configuraciones de escalado automático del grupo de recursos "myrg1" y, en concreto, la configuración de escalado automático con el nombre "MyScaleVMSSSetting".

```powershell
Get-AzAutoscalesetting -ResourceGroup myrg1 -Name MyScaleVMSSSetting -DetailedOutput
```

### <a name="remove-an-autoscale-setting"></a>Eliminación de una configuración de escalado automático
Puede usar el cmdlet `Remove-Autoscalesetting` para eliminar una configuración de escalado automático.

```powershell
Remove-AzAutoscalesetting -ResourceGroup myrg1 -Name MyScaleVMSSSetting
```

## <a name="manage-log-profiles-for-activity-log"></a>Administración de perfiles de registro para el registro de actividades
Puede crear un *perfil de registro* y exportar los datos de su registro de actividades a una cuenta de almacenamiento y configurar la retención de datos. Si lo desea, también puede transmitir los datos al Centro de eventos. Esta característica se encuentra actualmente en la fase de vista previa y solo puede crear un perfil de registro por suscripción. Puede utilizar los siguientes cmdlets con su suscripción actual para crear y administrar perfiles de registro. También puede elegir una suscripción específica. Aunque PowerShell tiene como valor predeterminado la suscripción actual, puede cambiarla cuando quiera con `Set-AzContext`. Puede configurar el registro de actividades para enrutar los datos a cualquier cuenta de almacenamiento o al centro de eventos de dicha suscripción. Los datos se escriben como archivos blob en formato JSON.

### <a name="get-a-log-profile"></a>Obtención de un perfil de registro
Para capturar los perfiles de registro existentes, use el cmdlet `Get-AzLogProfile` .

### <a name="add-a-log-profile-without-data-retention"></a>Adición de un perfil de registro sin retención de datos
```powershell
Add-AzLogProfile -Name my_log_profile_s1 -StorageAccountId /subscriptions/s1/resourceGroups/myrg1/providers/Microsoft.Storage/storageAccounts/my_storage -Location global,westus,eastus,northeurope,westeurope,eastasia,southeastasia,japaneast,japanwest,northcentralus,southcentralus,eastus2,centralus,australiaeast,australiasoutheast,brazilsouth,centralindia,southindia,westindia
```

### <a name="remove-a-log-profile"></a>Eliminación de un perfil de registro
```powershell
Remove-AzLogProfile -name my_log_profile_s1
```

### <a name="add-a-log-profile-with-data-retention"></a>Adición de un perfil de registro con retención de datos
Puede especificar la propiedad **-RetentionInDays** con el número de días, como un entero positivo, que se conservarán los datos.

```powershell
Add-AzLogProfile -Name my_log_profile_s1 -StorageAccountId /subscriptions/s1/resourceGroups/myrg1/providers/Microsoft.Storage/storageAccounts/my_storage -Location global,westus,eastus,northeurope,westeurope,eastasia,southeastasia,japaneast,japanwest,northcentralus,southcentralus,eastus2,centralus,australiaeast,australiasoutheast,brazilsouth,centralindia,southindia,westindia -RetentionInDays 90
```

### <a name="add-log-profile-with-retention-and-eventhub"></a>Adición de un perfil de registro retención y EventHub
Además de enrutar los datos a la cuenta de almacenamiento, también puede transmitirlos a un Centro de eventos. En esta versión preliminar, la configuración de la cuenta de almacenamiento es obligatoria; sin embargo, la configuración del Centro de eventos es opcional.

```powershell
Add-AzLogProfile -Name my_log_profile_s1 -StorageAccountId /subscriptions/s1/resourceGroups/myrg1/providers/Microsoft.Storage/storageAccounts/my_storage -serviceBusRuleId /subscriptions/s1/resourceGroups/Default-ServiceBus-EastUS/providers/Microsoft.ServiceBus/namespaces/mytestSB/authorizationrules/RootManageSharedAccessKey -Location global,westus,eastus,northeurope,westeurope,eastasia,southeastasia,japaneast,japanwest,northcentralus,southcentralus,eastus2,centralus,australiaeast,australiasoutheast,brazilsouth,centralindia,southindia,westindia -RetentionInDays 90
```

## <a name="configure-diagnostics-logs"></a>Configuración de registros de diagnóstico
Muchos servicios de Azure proporcionan registros adicionales y telemetría que pueden realizar una o varias de las siguientes acciones: 
 - configurarse para guardar los datos en su cuenta de Azure Storage.
 - enviarse a Event Hubs.
 - enviarse a un área de trabajo de Log Analytics. 

La operación solo puede realizarse en un nivel de recursos. La cuenta de almacenamiento o el centro de eventos debe encontrarse en la misma región que el recurso de destino donde se ha ajustado la configuración de diagnóstico.

### <a name="get-diagnostic-setting"></a>Obtención de la configuración de diagnóstico
```powershell
Get-AzDiagnosticSetting -ResourceId /subscriptions/s1/resourceGroups/myrg1/providers/Microsoft.Logic/workflows/andy0315logicapp
```

Deshabilitación de la configuración de diagnóstico

```powershell
Set-AzDiagnosticSetting -ResourceId /subscriptions/s1/resourceGroups/myrg1/providers/Microsoft.Logic/workflows/andy0315logicapp -StorageAccountId /subscriptions/s1/resourceGroups/Default-Storage-WestUS/providers/Microsoft.Storage/storageAccounts/mystorageaccount -Enable $false
```

Habilitación de la configuración de diagnóstico sin retención

```powershell
Set-AzDiagnosticSetting -ResourceId /subscriptions/s1/resourceGroups/myrg1/providers/Microsoft.Logic/workflows/andy0315logicapp -StorageAccountId /subscriptions/s1/resourceGroups/Default-Storage-WestUS/providers/Microsoft.Storage/storageAccounts/mystorageaccount -Enable $true
```

Habilitación la configuración de diagnóstico con retención

```powershell
Set-AzDiagnosticSetting -ResourceId /subscriptions/s1/resourceGroups/myrg1/providers/Microsoft.Logic/workflows/andy0315logicapp -StorageAccountId /subscriptions/s1/resourceGroups/Default-Storage-WestUS/providers/Microsoft.Storage/storageAccounts/mystorageaccount -Enable $true -RetentionEnabled $true -RetentionInDays 90
```

Habilitación de la configuración de diagnóstico con retención para una categoría de registro específica

```powershell
Set-AzDiagnosticSetting -ResourceId /subscriptions/s1/resourceGroups/insights-integration/providers/Microsoft.Network/networkSecurityGroups/viruela1 -StorageAccountId /subscriptions/s1/resourceGroups/myrg1/providers/Microsoft.Storage/storageAccounts/sakteststorage -Categories NetworkSecurityGroupEvent -Enable $true -RetentionEnabled $true -RetentionInDays 90
```

Habilitación de la configuración de diagnóstico para Event Hubs

```powershell
Set-AzDiagnosticSetting -ResourceId /subscriptions/s1/resourceGroups/insights-integration/providers/Microsoft.Network/networkSecurityGroups/viruela1 -serviceBusRuleId /subscriptions/s1/resourceGroups/Default-ServiceBus-EastUS/providers/Microsoft.ServiceBus/namespaces/mytestSB/authorizationrules/RootManageSharedAccessKey -Enable $true
```

Habilitación de la configuración de diagnóstico para Log Analytics

```powershell
Set-AzDiagnosticSetting -ResourceId /subscriptions/s1/resourceGroups/insights-integration/providers/Microsoft.Network/networkSecurityGroups/viruela1 -WorkspaceId /subscriptions/s1/resourceGroups/insights-integration/providers/providers/microsoft.operationalinsights/workspaces/myWorkspace -Enabled $true

```

Tenga en cuenta que la propiedad WorkspaceId usa el *id. de recurso* del área de trabajo. Puede obtener el identificador de recurso de su área de trabajo de Log Analytics con el comando siguiente:

```powershell
(Get-AzOperationalInsightsWorkspace).ResourceId

```

Estos comandos se pueden combinar para enviar datos a varios destinos.
