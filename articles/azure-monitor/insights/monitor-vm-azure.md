---
title: Supervisión de máquinas virtuales de Azure con Azure Monitor
description: Describe cómo recopilar y analizar los datos de supervisión de las máquinas virtuales de Azure mediante Azure Monitor.
ms.service: azure-monitor
ms.subservice: logs
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 05/05/2020
ms.openlocfilehash: 85c4807d5bf71078e3cfb26bbc27e9eecc10c041
ms.sourcegitcommit: 3fc3457b5a6d5773323237f6a06ccfb6955bfb2d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/11/2020
ms.locfileid: "90029468"
---
# <a name="monitoring-azure-virtual-machines-with-azure-monitor"></a>Supervisión de máquinas virtuales de Azure con Azure Monitor
En este artículo se describe cómo usar Azure Monitor para recopilar y analizar datos de supervisión de máquinas virtuales de Azure para mantener su estado. Las máquinas virtuales se pueden supervisar en términos de disponibilidad y rendimiento con Azure Monitor como cualquier [otro](monitor-azure-resource.md) recurso de Azure, pero se diferencian de otros recursos en cuanto que también debe supervisar el sistema operativo invitado y las cargas de trabajo que se ejecutan en él. 

> [!NOTE]
> En este artículo se proporciona información general completa de los conceptos y las opciones para la supervisión de máquinas virtuales en Azure Monitor. Para empezar a supervisar rápidamente las máquinas virtuales sin centrarse en los conceptos subyacentes, consulte [Inicio rápido: Supervisión de máquinas virtuales de Azure con Azure Monitor](../learn/quick-monitor-azure-vm.md).


## <a name="differences-from-other-azure-resources"></a>Diferencias con respecto a otros recursos de Azure
En [Supervisión de recursos de Azure con Azure Monitor](monitor-azure-resource.md) se describen los datos de supervisión generados por los recursos de Azure y cómo puede usar las características de Azure Monitor para analizar y alertar sobre estos datos. Puede recopilar los mismos datos de supervisión de máquinas virtuales de Azure y actuar en ellos con las diferencias siguientes:

-  Las [métricas de plataforma](../platform/data-platform-metrics.md) se recopilan automáticamente para las máquinas virtuales, pero solo para la [máquina virtual anfitriona](#monitoring-data). Necesita un agente para recopilar los datos de rendimiento del sistema operativo invitado. 
- Las máquinas virtuales no generan [registros de recursos](../platform/platform-logs-overview.md) que proporcionen información sobre las operaciones que se realizaron en un recurso de Azure. Se usa un agente para recopilar los datos de registro del sistema operativo invitado.
- Puede crear una [configuración de diagnóstico](../platform/diagnostic-settings.md) para que una máquina virtual envíe métricas de plataforma a otros destinos, como centros de eventos y de almacenamiento, pero no puede establecer esta configuración de diagnóstico en Azure Portal. 

## <a name="monitoring-data"></a>Supervisión de datos
Las máquinas virtuales de Azure generan [registros](../platform/data-platform-logs.md) y [métricas](../platform/data-platform-metrics.md), como se muestra en el diagrama siguiente.

![Información general](media/monitor-vm-azure/logs-metrics.png)


### <a name="virtual-machine-host"></a>Máquina virtual anfitriona
Las máquinas virtuales de Azure generan los datos siguientes para la máquina virtual anfitriona igual que otros recursos de Azure, tal como se describe en [Supervisión de datos](monitor-azure-resource.md#monitoring-data).

- [Métricas de plataforma](../platform/data-platform-metrics.md): valores numéricos que se recopilan automáticamente a intervalos regulares y describen algún aspecto de un recurso en un momento determinado. Las métricas de la plataforma se recopilan para la máquina virtual anfitriona, pero necesita la extensión de diagnóstico para recopilar métricas del sistema operativo invitado.
- [Registro de actividad](../platform/platform-logs-overview.md): proporciona una visión general de las operaciones de cada recurso de Azure de la suscripción desde fuera (en el plano de administración). En el caso de una máquina virtual, esto incluye información como cuándo se inició y cualquier cambio en la configuración.


### <a name="guest-operating-system"></a>Sistema operativo invitado
Para recopilar datos del sistema operativo invitado de una máquina virtual, necesita un agente, que se ejecuta localmente en cada máquina virtual y envía datos a Azure Monitor. Hay varios agentes disponibles para Azure Monitor, y cada uno de ellos recopila datos diferentes y escribe datos en ubicaciones diferentes. Consulte [Introducción a los agentes de Azure Monitor](../platform/agents-overview.md) para ver una comparativa detallada de los diferentes agentes. 

- [Agente de Log Analytics](../platform/agents-overview.md#log-analytics-agent): disponible para máquinas virtuales de Azure, otros entornos de nube y el entorno local. Recopila datos en los registros de Azure Monitor. Admite soluciones de Azure Monitor para VM y supervisión. Este es el mismo agente que se usa para System Center Operations Manager.
- [Dependency Agent](../platform/agents-overview.md#dependency-agent): recopila datos acerca de los procesos que se ejecutan en la máquina virtual y sus dependencias. Se basa en el agente de Log Analytics para transmitir datos a Azure y admite Azure Monitor para VM, Service Map y soluciones de Wire Data 2.0.
- [Extensión de diagnóstico de Azure](../platform/agents-overview.md#azure-diagnostics-extension): disponible solo para máquinas virtuales Azure Monitor. Puede recopilar datos en varias ubicaciones, pero se usa principalmente para recopilar datos de rendimiento del invitado en métricas de Azure Monitor para máquinas virtuales Windows.
- [Agente de Telegraf](../platform/collect-custom-metrics-linux-telegraf.md): recopila datos de rendimiento de máquinas virtuales Linux en métricas de Azure Monitor.


## <a name="configuration-requirements"></a>Requisitos de configuración
Para habilitar todas las características de Azure Monitor para la supervisión de una máquina virtual, debe recopilar datos de supervisión desde la máquina virtual anfitriona y el sistema operativo invitado tanto en [métricas de Azure Monitor](../platform/data-platform-logs.md) como en [registros de Azure Monitor](../platform/data-platform-logs.md). En la tabla siguiente se muestra la configuración que se debe aplicar para habilitar esta colección. Puede optar por no implementar todos estos pasos, en función de sus requisitos particulares.

| Paso de configuración | Acciones completadas | Características habilitadas |
|:---|:---|:---|
| Sin configuración | - Recopilación de las métricas de la plataforma en métricas de Azure Monitor.<br>- Recopilación del registro de actividad. | - Explorador de métricas para el anfitrión.<br>- Alertas de métricas para el anfitrión.<br>- Alertas de registros de actividad. |
| [Habilitar Azure Monitor para VM](#enable-azure-monitor-for-vms) | - Instalación del agente de Log Analytics.<br>- Instalación de Dependency Agent.<br>- Recopilación de datos de rendimiento del invitado en registros de Azure Monitor.<br>- Recopilación de detalles de proceso y dependencia en registros de Azure Monitor. | - Gráficos y libros de rendimiento para los datos de rendimiento del invitado.<br>- Consultas de registro para los datos de rendimiento del invitado.<br>- Alertas de registro para los datos de rendimiento del invitado.<br>- Mapa de dependencias. |
| [Instalación de la extensión de diagnósticos y el agente de Telegraf](#enable-diagnostics-extension-and-telegraf-agent) | - Recopilación de datos de rendimiento del invitado en métricas de Azure Monitor. | - Explorador de métricas para el invitado.<br>- Alertas de métricas para el invitado.  |
| [Configuración de un área de trabajo de Log Analytics](#configure-log-analytics-workspace) | - Recopilación de eventos del invitado. | - Consultas de registro para eventos del invitado.<br>- Alertas de registro para eventos del invitado. |
| [Creación de la configuración de diagnóstico de la máquina virtual](#collect-platform-metrics-and-activity-log) | - Recopilación de métricas de la plataforma en registros de Azure Monitor.<br>- Recopilación del registro de actividad en registros de Azure Monitor. | - Consultas de registros para métricas del anfitrión.<br>- Alertas de registros para métricas del anfitrión.<br>- Consultas de registros para el registro de actividades.

En las secciones siguientes se describen todos estos pasos de configuración.

### <a name="enable-azure-monitor-for-vms"></a>Habilitar Azure Monitor para VM
[Azure Monitor para VM](vminsights-overview.md) es un [punto de información](insights-overview.md) en Azure Monitor que sirve de herramienta principal para supervisar máquinas virtuales en Azure Monitor. Proporciona el siguiente valor adicional con respecto a las características de Azure Monitor estándar.

- Incorporación simplificada del agente de Log Analytics y de Dependency Agent para habilitar la supervisión de un sistema operativo invitado y las cargas de trabajo de una máquina virtual. 
- Libros y gráficas de rendimiento de tendencias predefinidos que permiten analizar las métricas de rendimiento básicas del sistema operativo invitado de la máquina virtual.
- Mapa de dependencias que muestra los procesos que se ejecutan en cada máquina virtual y los componentes interconectados con otras máquinas y orígenes externos.

![Vista Rendimiento de Azure Monitor para VM](media/monitor-vm-azure/vminsights-01.png)

![Vista Asignaciones de Azure Monitor para VM](media/monitor-vm-azure/vminsights-02.png)


Habilite Azure Monitor para VM en la opción **Información** del menú de la máquina virtual de Azure Portal. Vea [Información general sobre la habilitación de Azure Monitor para VM](vminsights-enable-overview.md) para obtener más información y conocer otros métodos de configuración.

![Habilitar Azure Monitor para VM](media/monitor-vm-azure/enable-vminsights.png)

### <a name="configure-log-analytics-workspace"></a>Configuración de un área de trabajo de Log Analytics
El agente de Log Analytics utilizado por Azure Monitor para VM envía datos al [área de trabajo de Log Analytics](../platform/data-platform-logs.md). Puede habilitar la recopilación de datos de rendimiento, eventos y otros datos de supervisión adicionales del agente mediante la configuración del área de trabajo de Log Analytics. Solo debe configurarse una vez, ya que cualquier agente que se conecte al área de trabajo descargará automáticamente la configuración y comenzará inmediatamente a recopilar los datos definidos. 

Puede tener acceso a la configuración del área de trabajo directamente desde Azure Monitor para VM seleccionando **Configuración del área de trabajo** en **Introducción**. Haga clic en el nombre del área de trabajo para abrir su menú.

![Configuración del área de trabajo](media/monitor-vm-azure/workspace-configuration.png)

Seleccione **Configuración avanzada** en el menú del área de trabajo y luego **Datos** para configurar los orígenes de datos. En el caso de los agentes de Windows, seleccione **Registros de eventos de Windows** y agregue registros de eventos comunes como *Sistema* y *Aplicación*. En el caso de los agentes de Linux, seleccione **Syslog** y agregue instalaciones comunes como *Kern* y *demonio*. Consulte [Orígenes de datos de agente en Azure Monitor](../platform/agent-data-sources.md) para obtener una lista de orígenes de datos disponibles e información sobre su configuración. 

![Configuración de eventos](media/monitor-vm-azure/configure-events.png)


> [!NOTE]
> Puede configurar los contadores de rendimiento que se van a recopilar de la configuración del área de trabajo, pero esto puede ser redundante con los contadores de rendimiento recopilados por Azure Monitor para VM. Azure Monitor para VM recopila el conjunto más común de contadores a una frecuencia de una vez por minuto. Configure solo los contadores de rendimiento que se van a recopilar por el área de trabajo si desea recopilar los contadores que no se han recopilado ya por Azure Monitor para VM o si tiene consultas existentes que usan datos de rendimiento.


### <a name="enable-diagnostics-extension-and-telegraf-agent"></a>Habilitación de la extensión de diagnósticos y el agente de Telegraf
Azure Monitor para VM se basa en el agente de Log Analytics que envía datos a un área de trabajo de Log Analytics. Esto admite varias características de Azure Monitor como las [consultas de registro](../log-query/log-query-overview.md), las [alertas de registro](../platform/alerts-log.md) y los [libros](../platform/workbooks-overview.md). La [extensión de diagnósticos](../platform/diagnostics-extension-overview.md) recopila los datos de rendimiento del sistema operativo invitado de las máquinas virtuales de Windows en Azure Storage y, opcionalmente, envía datos de rendimiento a las [métricas Azure Monitor](../platform/data-platform-metrics.md). En el caso de las máquinas virtuales Linux, se necesita el [agente de Telegraf](../platform/collect-custom-metrics-linux-telegraf.md) para enviar datos a las métricas de Azure.  Esto habilita otras características de Azure Monitor, como el [explorador de métricas](../platform/metrics-getting-started.md) y las [alertas de métricas](../platform/alerts-metric.md). También puede configurar la extensión de diagnósticos para enviar eventos y datos de rendimiento fuera de Azure Monitor mediante Azure Event Hubs.

Instale la extensión de diagnósticos para una sola máquina virtual Windows en Azure Portal desde la opción **Configuración de diagnóstico** del menú de la máquina virtual. Seleccione la opción para habilitar **Azure Monitor** en la pestaña **Receptores**. Para habilitar la extensión desde una plantilla o línea de comandos para varias máquinas virtuales, consulte [Instalación y configuración](../platform/diagnostics-extension-overview.md#installation-and-configuration). A diferencia del agente de Log Analytics, los datos que se van a recopilar se definen en la configuración de la extensión en cada máquina virtual.

![Configuración de diagnóstico](media/monitor-vm-azure/diagnostic-setting.png)

Consulte [Instalación y configuración de Telegraf](../platform/collect-custom-metrics-linux-telegraf.md#install-and-configure-telegraf) para obtener más información sobre la configuración de los agentes de Telegraf en máquinas virtuales Linux. La opción de menú **Configuración de diagnóstico** está disponible para Linux, pero solo le permitirá enviar datos a Azure Storage.

### <a name="collect-platform-metrics-and-activity-log"></a>Recopilación de métricas de plataforma y registro de actividad
Puede ver las métricas de la plataforma y el registro de actividad recopilados para cada máquina virtual anfitriona en Azure Portal. Recopile estos datos en la misma área de trabajo de Log Analytics que Azure Monitor para VM para analizarlos con los demás datos de supervisión recopilados para la máquina virtual. Esta colección se establece con una [configuración de diagnóstico](../platform/diagnostic-settings.md). Recopile el registro de actividad con una [configuración de diagnóstico para la suscripción](../platform/diagnostic-settings.md#create-in-azure-portal).

Recopile las métricas de la plataforma con una configuración de diagnóstico para la máquina virtual. A diferencia de otros recursos de Azure, no puede crear una configuración de diagnóstico para una máquina virtual en Azure Portal, sino que debe usar [otro método](../platform/diagnostic-settings.md#create-using-powershell). En los siguientes ejemplos se muestra cómo recopilar métricas para una máquina virtual mediante PowerShell y la CLI.

```powershell
Set-AzDiagnosticSetting -Name vm-diagnostics -ResourceId "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/my-resource-group/providers/Microsoft.Compute/virtualMachines/my-vm" -Enabled $true -MetricCategory AllMetrics -workspaceId "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourcegroups/my-resource-group/providers/microsoft.operationalinsights/workspaces/my-workspace"
```

```CLI
az monitor diagnostic-settings create \
--name VM-Diagnostics 
--resource /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/my-resource-group/providers/Microsoft.Compute/virtualMachines/my-vm \
--metrics '[{"category": "AllMetrics","enabled": true}]' \
--workspace /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourcegroups/my-resource-group/providers/microsoft.operationalinsights/workspaces/my-workspace
```

## <a name="monitoring-in-the-azure-portal"></a>Supervisión en Azure Portal 
Una vez configurada la recopilación de los datos de supervisión para una máquina virtual, tiene varias opciones para tener acceso a ella en Azure Portal:

- Use el menú **Azure Monitor** para acceder a los datos de todos los recursos supervisados. 
- Use Azure Monitor para VM para supervisar conjuntos de máquinas virtuales a escala.
- Analice los datos de una sola máquina virtual desde su menú en Azure Portal. En la tabla siguiente se enumeran las diferentes opciones para supervisar el menú de máquinas virtuales.

![Supervisión en Azure Portal](media/monitor-vm-azure/monitor-menu.png)

| Opción de menú | Descripción |
|:---|:---|
| Información general | Muestra las [métricas de plataforma](../platform/data-platform-metrics.md) para la máquina virtual anfitriona. Haga clic en un gráfico para trabajar con estos datos en el [explorador de métricas](../platform/metrics-getting-started.md). |
| Registro de actividades | Entradas del [registro de actividad](../platform/activity-log.md#view-the-activity-log) filtradas para la máquina virtual actual. |
| Información detallada | Abra [Azure Monitor para VM](./vminsights-overview.md) con la asignación de la máquina virtual actual seleccionada. |
| Alertas | Vea las [alertas](../platform/alerts-overview.md) para la máquina virtual actual.  |
| Métricas | Abra el [explorador de métricas](../platform/metrics-getting-started.md) con el ámbito establecido en la máquina virtual actual. |
| Configuración de diagnóstico | Habilite y configure la [extensión de diagnóstico](../platform/diagnostics-extension-overview.md) para la máquina virtual actual. |
| Recomendaciones de Advisor | Recomendaciones para la máquina virtual actual desde [Azure Advisor](../../advisor/index.yml). |
| Registros | Abra [Log Analytics](../log-query/log-query-overview.md#what-is-log-analytics) con el [ámbito](../log-query/scope.md) establecido en la máquina virtual actual. |
| Monitor de conexión | Abra la [supervisión de conexiones de Network Watcher](../../network-watcher/connection-monitor-preview.md) para supervisar las conexiones entre la máquina virtual actual y otras máquinas virtuales. |


## <a name="analyzing-metric-data"></a>Analizar datos de métricas
Puede analizar las métricas de las máquinas virtuales mediante el explorador de métricas abriendo **Métricas** en el menú de la máquina virtual. Consulte [Introducción al explorador de métricas de Azure](../platform/metrics-getting-started.md) para más información sobre esta herramienta. 

Hay tres espacios de nombres usados por las máquinas virtuales para las métricas:

| Espacio de nombres | Descripción | Requisito |
|:---|:---|:---|
| Máquina virtual anfitriona | Métricas de la anfitriona recopiladas automáticamente para todas las máquinas virtuales de Azure. La lista detallada de métricas en se encuentra en [Microsoft.Compute/virtualMachines](../platform/metrics-supported.md#microsoftcomputevirtualmachines). | Se recopilan automáticamente sin necesidad de configuración. |
| Invitado (clásico) | Conjunto limitado de datos de rendimiento del sistema operativo invitado y de la aplicación. Está disponible en el explorador de métricas, pero no en otras características de Azure Monitor, como las alertas de métricas.  | [Extensión de diagnósticos](../platform/diagnostics-extension-overview.md) instalada. Los datos se leen desde Azure Storage.  |
| Máquina virtual invitada | Datos de rendimiento del sistema operativo invitado y de la aplicación disponibles para todas las características de Azure Monitor mediante métricas. | En Windows, [la extensión de diagnósticos instalada](../platform/diagnostics-extension-overview.md) con el receptor de Azure Monitor habilitado. Para Linux, el [agente de Telegraf instalado](../platform/collect-custom-metrics-linux-telegraf.md). |

![Explorador de métricas en Azure Portal](media/monitor-vm-azure/metrics.png)

## <a name="analyzing-log-data"></a>Analizar datos de registro
Las máquinas virtuales de Azure recopilarán los datos siguientes para los registros de Azure Monitor. 

Azure Monitor para VM habilita la recopilación de un conjunto predeterminado de contadores de rendimiento que se escriben en la tabla *InsightsMetrics*. Se trata de la misma tabla utilizada por [Azure Monitor para contenedores](container-insights-overview.md). 

| Origen de datos | Requisitos | Tablas |
|:---|:---|:---|
| Azure Monitor para máquinas virtuales | Habilite en cada máquina virtual. | InsightsMetrics<br>VMBoundPort<br>VMComputer<br>VMConnection<br>VMProcess<br>Vea [Cómo consultar registros de Azure Monitor para VM](vminsights-log-search.md) para obtener información detallada. |
| Registro de actividades | Configuración de diagnóstico para la suscripción. | AzureActivity |
| Métricas de la anfitriona | Configuración de diagnóstico de la máquina virtual. | AzureMetrics |
| Orígenes de datos del sistema operativo invitado | Habilite el agente de Log Analytics y configure los orígenes de datos. | Consulte la documentación de cada origen de datos. |


> [!NOTE]
> Los datos de rendimiento recopilados por el agente de Log Analytics escriben en la tabla *Perf*, mientras que Azure Monitor para VM los recopilará en la tabla *InsightsMetrics*. Se trata de los mismos datos, pero las tablas tienen una estructura diferente. Si tiene consultas existentes basadas en *Perf*, deberá volver a escribirlas para usar *InsightsMetrics*.


## <a name="alerts"></a>Alertas
Las [alertas](../platform/alerts-overview.md) en Azure Monitor notifican de forma proactiva las condiciones importantes que encuentran en los datos de supervisión y pueden adoptar medidas como iniciar una aplicación lógica o llamar a un webhook. Las reglas de alerta definen la lógica que se usa para determinar cuándo se debe crear una alerta. Azure Monitor recopila los datos usados por las reglas de alertas, pero debe crear reglas para definir las condiciones de alerta en su suscripción de Azure.

En las secciones siguientes se describen los tipos de reglas de alertas y recomendaciones sobre cuándo se debe usar cada una de ellas. Esta recomendación se basa en la funcionalidad y el costo del tipo de regla de alerta. Para más información sobre los precios de las alertas, consulte [Precios de Azure Monitor](https://azure.microsoft.com/pricing/details/monitor/).


### <a name="activity-log-alert-rules"></a>Reglas de alertas de registro de actividad
Las [reglas de alerta del registro de actividad](../platform/alerts-activity-log.md) se activan cuando se crea una entrada que coincide con determinados criterios en el registro de actividad. No tienen ningún costo, por lo que deben ser la primera opción si la lógica que necesita está en el registro de actividad. 

El recurso de destino para las alertas del registro de actividad puede ser una máquina virtual específica, todas las máquinas virtuales de un grupo de recursos o todas las máquinas virtuales de una suscripción.

Por ejemplo, cree una alerta si se detiene una máquina virtual crítica seleccionando *Desconectar máquina virtual* para el nombre de la señal.

![Alerta de registro de actividad](media/monitor-vm-azure/activity-log-alert.png)


### <a name="metric-alert-rules"></a>Reglas de alertas de métricas
Las [reglas de alertas de métricas](../platform/alerts-metric.md) se activan cuando un valor de métrica supera un umbral. Puede definir un valor de umbral específico o permitir que Azure Monitor determine dinámicamente un umbral en función de datos históricos.  Use las alertas de métricas siempre que sea posible con los datos de métricas, ya que tienen un costo menor y tienen más capacidad de respuesta que las reglas de alertas de registro. También tienen estado, lo que significa que se resolverán por sí mismas cuando la métrica caiga por debajo del umbral.

El recurso de destino para las alertas del registro de actividad puede ser una máquina virtual específica o todas las máquinas virtuales de un grupo de recursos.

Por ejemplo, para crear una alerta cuando el procesador de una máquina virtual supera un valor determinado, cree una regla de alertas de métricas con *Porcentaje de CPU* como tipo de señal. Establezca un valor de umbral específico o permita que Azure Monitor establezca un umbral dinámico. 

![Alerta de métrica](media/monitor-vm-azure/metric-alert.png)

### <a name="log-alerts"></a>Alertas de registro
Las [reglas de alertas de registro](../platform/alerts-log.md) se activan cuando los resultados de una consulta de registro programada coinciden con determinados criterios. Las alertas de consulta de registro son las reglas de alerta más caras y con menos capacidad de respuesta, pero tienen acceso a los datos más diversos y pueden aplicar una lógica compleja que no se puede conseguir mediante las demás reglas de alerta. 

El recurso de destino para una consulta de registro es un área de trabajo de Log Analytics. Filtre para obtener los equipos específicos de la consulta.

Por ejemplo, para crear una alerta que compruebe si una máquina virtual de un grupo de recursos determinado está sin conexión, use la siguiente consulta, que devuelve un registro para cada equipo que no ha tenido ningún latido en los últimos diez minutos. Use un umbral de 1 que se desencadene si al menos a un equipo le falta un latido.

```kusto
Heartbeat
| where TimeGenerated < ago(10m)
| where ResourceGroup == "my-resource-group"
| summarize max(TimeGenerated) by Computer
```

![Alerta de registro para latido faltante](media/monitor-vm-azure/log-alert-01.png)

Para crear una alerta si se ha producido un número excesivo de inicios de sesión con errores en las máquinas virtuales Windows de la suscripción, utilice la siguiente consulta, que devuelve un registro para cada evento con un inicio de sesión con errores en la última hora. Use un umbral establecido en el número de inicios de sesión con errores que permitirá. 

```kusto
Event
| where TimeGenerated < ago(1hr)
| where EventID == 4625
```

![Alerta de registro para inicios de sesión con error](media/monitor-vm-azure/log-alert-02.png)


## <a name="system-center-operations-manager"></a>System Center Operations Manager
System Center Operations Manager proporciona una supervisión detallada de las cargas de trabajo en las máquinas virtuales. Consulte la [guía de supervisión en la nube](/azure/cloud-adoption-framework/manage/monitor/) para obtener una comparación de las plataformas de supervisión y las distintas estrategias de implementación.

Si ya tiene un entorno de Operations Manager que quiere seguir usando, puede integrarlo con Azure Monitor para incluir funcionalidad adicional. El agente de Log Analytics que usa Azure Monitor es el mismo que se usa para Operations Manager, de modo que las máquinas virtuales puedan enviar datos a ambos. Todavía necesita agregar el agente a Azure Monitor para VM y configurar el área de trabajo para recopilar datos adicionales, tal y como se especificó anteriormente, pero las máquinas virtuales pueden seguir ejecutando sus módulos de administración existentes en un entorno de Operations Manager sin modificación.

Entre las características de Azure Monitor que aumentan las características de Operations Manager existentes se incluyen las siguientes:

- Use Log Analytics para analizar interactivamente los datos de registro y rendimiento.
- Use alertas de registro para definir condiciones de alerta en varias máquinas virtuales y usar tendencias a largo plazo que no se pueden obtener mediante las alertas de Operations Manager.   

Consulte [Conexión de Operations Manager a Azure Monitor](../platform/om-agents.md) para obtener más información sobre cómo conectar el grupo de administración de Operations Manager existente al área de trabajo de Log Analytics.


## <a name="next-steps"></a>Pasos siguientes

* [Obtenga información sobre el análisis de datos en registros de Azure Monitor mediante consultas de registro.](../log-query/get-started-queries.md)
* [Obtenga información sobre las alertas que usan métricas y registros en Azure Monitor.](../platform/alerts-overview.md)

