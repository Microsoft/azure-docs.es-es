---
title: Solución Azure DNS Analytics en Azure Monitor | Microsoft Docs
description: La solución Azure SQL Analytics le ayuda a administrar sus bases de datos de Azure SQL.
ms.subservice: logs
ms.topic: conceptual
author: danimir
ms.author: danil
ms.date: 02/21/2020
ms.reviewer: carlrab
ms.openlocfilehash: c871f5fbbe63747c71e1f6ecf83a47c0cd30970e
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87318035"
---
# <a name="monitor-azure-sql-database-using-azure-sql-analytics-preview"></a>Supervisión de instancias de Azure SQL Database con Azure SQL Analytics (versión preliminar)

![Símbolo de Azure SQL Analytics](./media/azure-sql/azure-sql-symbol.png)

Azure SQL Analytics es una solución de supervisión en la nube que se utiliza para supervisar el rendimiento de todas las bases de datos de Azure SQL Database a escala, entre varias suscripciones y en una vista única. Azure SQL Analytics recopila y visualiza métricas clave del rendimiento con inteligencia integrada para solucionar problemas de rendimiento.

Mediante el uso de estas métricas que se han recopilado, puede crear alertas y reglas de supervisión personalizadas. Azure SQL Analytics también le ayuda a identificar los problemas de cada capa de la pila de la aplicación. Para presentar datos sobre todas las bases de datos de Azure SQL en una sola área de trabajo de Log Analytics, usa las métricas de Azure Diagnostics junto con las vistas de Azure Monitor. Azure Monitor le ayuda a recopilar, correlacionar y visualizar datos estructurados y no estructurados.

Para obtener información general práctica acerca del uso de la solución Azure SQL Analytics y escenarios de uso habituales, vea el vídeo incrustado:

>[!VIDEO https://www.youtube.com/embed/j-NDkN4GIzg]
>

## <a name="connected-sources"></a>Orígenes conectados

Azure SQL Analytics es una solución de supervisión solo en la nube que admite el streaming de telemetría de diagnósticos para las bases de datos de Azure SQL. Teniendo en cuenta que Azure SQL Analytics no utiliza agentes para conectarse a Azure Monitor, no admite la supervisión de instancias de SQL Server hospedadas localmente o en máquinas virtuales.

| Origen conectado | Compatible | Descripción |
| --- | --- | --- |
| [Configuración de diagnóstico](../platform/diagnostic-settings.md) | **Sí** | Azure envía directamente los datos de métricas y registros de Azure a los registros de Azure Monitor. |
| [Cuenta de Almacenamiento de Azure](../platform/resource-logs.md#send-to-log-analytics-workspace) | No | Azure Monitor no lee los datos de una cuenta de almacenamiento. |
| [Agentes de Windows](../platform/agent-windows.md) | No | Azure SQL Analytics no utiliza agentes directos de Windows. |
| [Agentes de Linux](../learn/quick-collect-linux-computer.md) | No | Azure SQL Analytics no utiliza agentes directos de Linux. |
| [Grupo de administración de System Center Operations](../platform/om-agents.md) | No | Azure SQL Analytics no utiliza ninguna conexión directa entre el agente de Operations Manager y Azure Monitor. |

## <a name="azure-sql-analytics-options"></a>Opciones de Azure SQL Analytics

En la siguiente tabla se describen opciones compatibles para dos versiones del panel de Azure SQL Analytics, una para Azure SQL Database y la otra para las bases de datos de Instancia administrada de Azure SQL.

| Opción de Azure SQL Analytics | Descripción | Compatibilidad con SQL Database | Compatibilidad con Instancia administrada de SQL |
| --- | ------- | ----- | ----- |
| Recurso por tipo | Perspectiva que considera todos los recursos supervisados. | Sí | Sí |
| Información detallada | Proporciona un informe detallado jerárquico del rendimiento de Intelligent Insights. | Sí | Sí |
| Errors | Ofrece un informe detallado jerárquico de los errores de SQL que se produjeron en las bases de datos. | Sí | Sí |
| Tiempos de expiración | Ofrece un informe detallado jerárquico de los tiempos de expiración de SQL que se produjeron en las bases de datos. | Sí | No |
| Bloqueos | Ofrece un informe detallado jerárquico de los bloqueos de SQL que se produjeron en las bases de datos. | Sí | No |
| Esperas de la base de datos | Ofrece un informe detallado jerárquico de las estadísticas de espera de SQL en el nivel de base de datos. Incluye resúmenes del tiempo de espera total y el tiempo de espera por tipo de espera. |Sí | No |
| Duración de la consulta | Ofrece un informe detallado jerárquico de las estadísticas de ejecución de consulta, como la duración de la consulta, el uso de CPU, el uso de E/S de datos y el uso de E/S de registro. | Sí | Sí |
| Esperas de consulta | Ofrece un informe detallado jerárquico de las estadísticas de espera de consulta por categoría de espera. | Sí | Sí |

## <a name="configuration"></a>Configuración

Use el proceso que se describe en cómo [agregar soluciones de Azure Monitor desde la Galería de soluciones](./solutions.md) para agregar la solución Azure SQL Analytics (versión preliminar) al área de trabajo de Log Analytics.

### <a name="configure-azure-sql-database-to-stream-diagnostics-telemetry"></a>Configuración de Azure SQL Database para transmitir la telemetría de diagnósticos

Cuando haya creado la solución Azure SQL Analytics en el área de trabajo, tendrá que **configurar cada uno** de los recursos que quiere supervisar para transmitir en secuencias su telemetría de diagnósticos a Azure SQL Analytics. Siga las instrucciones detalladas en esta página:

- Habilite Azure Diagnostics en su base de datos para [transmitir en secuencias la telemetría de diagnósticos a Azure SQL Analytics](../../azure-sql/database/metrics-diagnostic-telemetry-logging-streaming-export-configure.md).

En la página anterior, también se proporcionan instrucciones sobre cómo habilitar la compatibilidad para supervisar varias suscripciones de Azure desde una única área de trabajo de Azure SQL Analytics como un panel único.

## <a name="using-azure-sql-analytics"></a>Uso de Azure SQL Analytics

Cuando se agrega Azure SQL Analytics al área de trabajo, el icono de Azure SQL Analytics se agrega al área de trabajo y aparece en la introducción. Seleccione el vínculo Ver resumen para cargar el contenido del icono.

![Icono de resumen Azure SQL Analytics](./media/azure-sql/azure-sql-sol-tile-01.png)

Una vez cargado, el icono muestra el número de bases de datos y grupos elásticos en SQL Database e instancias y bases de datos de instancias en Instancia administrada de SQL de las que Azure SQL Analytics recibe datos de telemetría de diagnóstico.

![Icono de Azure SQL Analytics](./media/azure-sql/azure-sql-sol-tile-02.png)

Azure SQL Analytics proporciona dos vistas independientes, una para supervisar SQL Database y la otra para supervisar Instancia administrada de SQL.

Para ver el panel de supervisión de Azure SQL Analytics de Azure SQL, haga clic en la parte superior del icono. Para ver el panel de supervisión de Azure SQL Analytics para Instancia administrada de SQL, haga clic en la parte inferior del icono.

### <a name="viewing-azure-sql-analytics-data"></a>Visualización de los datos de Azure SQL Analytics

El panel incluye la información general de todas las bases de datos que se supervisan a través de distintas perspectivas. Para que estas distintas perspectivas funcionen, debe habilitar las métricas o los registros apropiados en los recursos de SQL para que se transmitan al área de trabajo de Log Analytics.

Si no hay transmisión de ninguna métrica o registro en Azure Monitor, los iconos en Azure SQL Analytics no se rellenarán con la información de supervisión.

### <a name="sql-database-view"></a>Vista de SQL Database

Una vez que se selecciona el icono de Azure SQL Analytics para la base de datos, aparece el panel de supervisión.

![Información general de Azure SQL Analytics](./media/azure-sql/azure-sql-sol-overview.png)

Si selecciona cualquiera de los iconos, se abre un informe detallado de la perspectiva específica. Una vez que se ha seleccionado la perspectiva, se abre el informe de la exploración en profundidad.

![Tiempos de expiración de Azure SQL Analytics](./media/azure-sql/azure-sql-sol-metrics.png)

Cada perspectiva de esta vista proporciona resúmenes en los niveles de suscripción, servidor, grupo elástico y base de datos. Además, cada perspectiva muestra a la derecha una perspectiva específica del informe. Seleccione suscripción, servidor, grupo o base de datos en la lista para seguir obteniendo los detalles.

### <a name="sql-managed-instance-view"></a>Vista de Instancia administrada de SQL

Una vez que se selecciona el icono de Azure SQL Analytics para las bases de datos, aparece el panel de supervisión.

![Información general de Azure SQL Analytics](./media/azure-sql/azure-sql-sol-overview-mi.png)

Si selecciona cualquiera de los iconos, se abre un informe detallado de la perspectiva específica. Una vez que se ha seleccionado la perspectiva, se abre el informe de la exploración en profundidad.

Si selecciona la vista de Instancia administrada de SQL, aparecerán los detalles de uso de la instancia, las bases de datos de instancia y los datos de telemetría de las consultas que se han ejecutado en la instancia administrada.

![Tiempos de expiración de Azure SQL Analytics](./media/azure-sql/azure-sql-sol-metrics-mi.png)

### <a name="intelligent-insights-report"></a>Informe de Intelligent Insights

Azure SQL Database [Intelligent Insights](../../azure-sql/database/intelligent-insights-overview.md) le permite saber lo que ocurre con el rendimiento de todas las bases de datos de Azure SQL. Todas las instancias de Intelligent Insights que se recopilan se pueden visualizar y acceder a través de la perspectiva de información detallada.

![Información detallada de Azure SQL Analytics](./media/azure-sql/azure-sql-sol-insights.png)

### <a name="elastic-pools-and-database-reports"></a>Informes de grupos elásticos y bases de datos

Tanto los grupos elásticos como las bases de datos tienen sus propios informes específicos que muestran cómo se recopilan todos los datos para el recurso en el tiempo especificado.

![Base de datos Azure SQL Analytics](./media/azure-sql/azure-sql-sol-database.png)

![Grupo elástico de Azure SQL](./media/azure-sql/azure-sql-sol-pool.png)

### <a name="query-reports"></a>Informes de consulta

Con las perspectivas de duración de consulta y de esperas de consulta, puede correlacionar el rendimiento de cualquier consulta mediante el informe de consulta. En este informe se compara el rendimiento de la consulta en distintas bases de datos y facilita el proceso de identificar las bases de datos que se completan de manera correcta la consulta seleccionada en lugar de las lentas.

![Consultas de Azure SQL Analytics](./media/azure-sql/azure-sql-sol-queries.png)

## <a name="permissions"></a>Permisos

Para usar Azure SQL Analytics, los usuarios necesitan que se les conceda un permiso mínimo del rol Lector en Azure. Pero este rol no permite a los usuarios ver el texto de la consulta ni realizar acciones automáticas de optimización. Otros roles más permisivos en Azure que permiten usar todas las funciones de Azure SQL Analytics son Propietario, Colaborador, Colaborador de base de datos SQL o Colaborador de SQL Server. También puede crear un rol personalizado en el portal con permisos específicos necesarios solo para usar Azure SQL Analytics, sin acceso para administrar otros recursos.

### <a name="creating-a-custom-role-in-portal"></a>Crear un rol personalizado en el portal

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

Al reconocer que algunas organizaciones exigen la implementación de controles estrictos de permisos en Azure, hemos diseñado el siguiente script de PowerShell que permite crear el rol personalizado "Operador de supervisión de SQL Analytics" en Azure Portal con los permisos mínimos de lectura y escritura necesarios para usar todas las funciones de Azure SQL Analytics.

Reemplace "{SubscriptionId}" en el script siguiente por el identificador de su suscripción de Azure y ejecute el script después de iniciar sesión con el rol Propietario o Colaborador en Azure.

   ```powershell
    Connect-AzAccount
    Select-AzSubscription {SubscriptionId}
    $role = Get-AzRoleDefinition -Name Reader
    $role.Name = "SQL Analytics Monitoring Operator"
    $role.Description = "Lets you monitor database performance with Azure SQL Analytics as a reader. Does not allow change of resources."
    $role.IsCustom = $true
    $role.Actions.Add("Microsoft.SQL/servers/databases/read");
    $role.Actions.Add("Microsoft.SQL/servers/databases/topQueries/queryText/*");
    $role.Actions.Add("Microsoft.Sql/servers/databases/advisors/read");
    $role.Actions.Add("Microsoft.Sql/servers/databases/advisors/write");
    $role.Actions.Add("Microsoft.Sql/servers/databases/advisors/recommendedActions/read");
    $role.Actions.Add("Microsoft.Sql/servers/databases/advisors/recommendedActions/write");
    $role.Actions.Add("Microsoft.Sql/servers/databases/automaticTuning/read");
    $role.Actions.Add("Microsoft.Sql/servers/databases/automaticTuning/write");
    $role.Actions.Add("Microsoft.Sql/servers/advisors/read");
    $role.Actions.Add("Microsoft.Sql/servers/advisors/write");
    $role.Actions.Add("Microsoft.Sql/servers/advisors/recommendedActions/read");
    $role.Actions.Add("Microsoft.Sql/servers/advisors/recommendedActions/write");
    $role.Actions.Add("Microsoft.Resources/deployments/write");
    $role.AssignableScopes = "/subscriptions/{SubscriptionId}"
    New-AzRoleDefinition $role
   ```

Después de crear el rol, asígnelo a cada usuario al que necesite conceder permisos personalizados para usar Azure SQL Analytics.

## <a name="analyze-data-and-create-alerts"></a>Análisis de datos y creación de alertas

El análisis de datos en Azure SQL Analytics se basa en el [lenguaje de Log Analytics](../log-query/get-started-queries.md) para los informes y consultas personalizados. Encontrará una descripción de los datos disponibles recopilados del recurso de base de datos para consultas personalizadas en [Métricas y registros disponibles](../../azure-sql/database/metrics-diagnostic-telemetry-logging-streaming-export-configure.md#metrics-and-logs-available).

Para configurar alertas automáticas en Azure SQL Analytics, se escribe una consulta de Log Analytics que desencadena una alerta cuando se cumple una condición. Abajo encontrará varios ejemplos de consultas de Log Analytics en las que se pueden configurar alertas en Azure SQL Analytics.

### <a name="creating-alerts-for-azure-sql-database"></a>Creación de alertas para una base de datos de Azure SQL

Las [alertas se pueden crear fácilmente](../platform/alerts-metric.md) con los datos procedentes de los recursos de Azure SQL Database. Estas son algunas [consultas de registros](../log-query/log-query-overview.md) útiles que puede usar para las alertas de registros:

#### <a name="high-cpu"></a>Uso de CPU elevado

```
AzureMetrics
| where ResourceProvider=="MICROSOFT.SQL"
| where ResourceId contains "/DATABASES/"
| where MetricName=="cpu_percent"
| summarize AggregatedValue = max(Maximum) by bin(TimeGenerated, 5m)
| render timechart
```

> [!NOTE]
>
> - El requisito previo para configurar esta alerta es que las bases de datos supervisadas transmitan métricas básicas a Azure SQL Analytics.
> - Reemplace el valor de MetricName cpu_percent por dtu_consumption_percent para obtener resultados DTU altos.

#### <a name="high-cpu-on-elastic-pools"></a>CPU alta en grupos elásticos

```
AzureMetrics
| where ResourceProvider=="MICROSOFT.SQL"
| where ResourceId contains "/ELASTICPOOLS/"
| where MetricName=="cpu_percent"
| summarize AggregatedValue = max(Maximum) by bin(TimeGenerated, 5m)
| render timechart
```

> [!NOTE]
>
> - El requisito previo para configurar esta alerta es que las bases de datos supervisadas transmitan métricas básicas a Azure SQL Analytics.
> - Reemplace el valor de MetricName cpu_percent por dtu_consumption_percent para obtener resultados DTU altos.

#### <a name="storage-in-average-above-95-in-the-last-1-hr"></a>El promedio de almacenamiento es superior al 95 % en la última hora

```
let time_range = 1h;
let storage_threshold = 95;
AzureMetrics
| where ResourceId contains "/DATABASES/"
| where MetricName == "storage_percent"
| summarize max_storage = max(Average) by ResourceId, bin(TimeGenerated, time_range)
| where max_storage > storage_threshold
| distinct ResourceId
```

> [!NOTE]
>
> - El requisito previo para configurar esta alerta es que las bases de datos supervisadas transmitan métricas básicas a Azure SQL Analytics.
> - Esta consulta requiere que se configure una regla de alerta para activar una alerta cuando existen resultados (> 0 resultados) de la consulta, lo que indica que la condición existe en algunas bases de datos. La salida es una lista de recursos de la base de datos que superan el valor storage_threshold dentro del valor de time_range definido.
> - La salida es una lista de recursos de la base de datos que superan el valor storage_threshold dentro del valor de time_range definido.

#### <a name="alert-on-intelligent-insights"></a>Alerta sobre Intelligent Insights

```
let alert_run_interval = 1h;
let insights_string = "hitting its CPU limits";
AzureDiagnostics
| where Category == "SQLInsights" and status_s == "Active"
| where TimeGenerated > ago(alert_run_interval)
| where rootCauseAnalysis_s contains insights_string
| distinct ResourceId
```

> [!NOTE]
>
> - El requisito previo para configurar esta alerta es que las bases de datos supervisadas transmitan el registro de diagnóstico de SQLInsights a Azure SQL Analytics.
> - Esta consulta requiere que se configure una regla de alerta para que se ejecute con la misma frecuencia que alert_run_interval para evitar resultados duplicados. La regla debe configurarse para activar la alerta cuando existen resultados (> 0 resultados) de la consulta.
> - Personalice alert_run_interval para especificar el intervalo de tiempo para comprobar si la condición ha ocurrido en las bases de datos configuradas para transmitir el registro de SQLInsights a Azure SQL Analytics.
> - Personalice el valor de insights_string para capturar la salida del texto de análisis de la causa principal de Insights. Este es el mismo texto que se muestra en la interfaz de usuario de Azure SQL Analytics y que puede utilizar a partir de las conclusiones existentes. Como alternativa, puede utilizar la consulta siguiente para ver el texto de todas las conclusiones generadas en la suscripción. Utilice la salida de la consulta para recopilar las distintas cadenas para configurar alertas sobre Insights.

```
AzureDiagnostics
| where Category == "SQLInsights" and status_s == "Active"
| distinct rootCauseAnalysis_s
```

### <a name="creating-alerts-for-sql-managed-instance"></a>Creación de alertas para Instancia administrada de SQL

#### <a name="storage-is-above-90"></a>El almacenamiento está por encima del 90 %

```
let storage_percentage_threshold = 90;
AzureDiagnostics
| where Category =="ResourceUsageStats"
| summarize (TimeGenerated, calculated_storage_percentage) = arg_max(TimeGenerated, todouble(storage_space_used_mb_s) *100 / todouble (reserved_storage_mb_s))
   by ResourceId
| where calculated_storage_percentage > storage_percentage_threshold
```

> [!NOTE]
>
> - Como requisito previo para configurar esta alerta, la instancia administrada supervisada necesita tener habilitado el streaming del registro ResourceUsageStats para Azure SQL Analytics.
> - Esta consulta requiere que se configure una regla de alerta para activar una alerta cuando existen resultados (> 0 resultados) de la consulta, lo que indica que la condición existe en la instancia administrada. La salida es el consumo de porcentaje de almacenamiento en la instancia administrada.

#### <a name="cpu-average-consumption-is-above-95-in-the-last-1-hr"></a>El promedio de consumo de CPU es superior al 95 % en la última hora

```
let cpu_percentage_threshold = 95;
let time_threshold = ago(1h);
AzureDiagnostics
| where Category == "ResourceUsageStats" and TimeGenerated > time_threshold
| summarize avg_cpu = max(todouble(avg_cpu_percent_s)) by ResourceId
| where avg_cpu > cpu_percentage_threshold
```

> [!NOTE]
>
> - Como requisito previo para configurar esta alerta, la instancia administrada supervisada necesita tener habilitado el streaming del registro ResourceUsageStats para Azure SQL Analytics.
> - Esta consulta requiere que se configure una regla de alerta para activar una alerta cuando existen resultados (> 0 resultados) de la consulta, lo que indica que la condición existe en la instancia administrada. El resultado es el porcentaje de consumo de uso promedio de CPU en un período definido en la instancia administrada.

### <a name="pricing"></a>Precios

Aunque Azure SQL Analytics puede usarse de forma gratuita, se aplicarán cargos por el uso de la telemetría de diagnósticos que supere las unidades gratuitas de ingesta de datos asignadas cada mes; para más información, consulte los [precios de Log Analytics](https://azure.microsoft.com/pricing/details/monitor). Las unidades gratuitas de ingesta de datos especificadas habilitan la supervisión gratuita de varias bases de datos al mes. Las bases de datos más activas con cargas de trabajo más pesadas ingieren más datos que las bases de datos inactivas. Puede supervisar fácilmente su ingesta de datos en Azure SQL Analytics; para ello seleccione el área de trabajo de OMS en el menú de navegación de Azure SQL Analytics y, después, seleccione Uso y costos estimados.

## <a name="next-steps"></a>Pasos siguientes

- Use las [consultas de registros](../log-query/log-query-overview.md) de Azure Monitor para ver datos detallados de Azure SQL.
- [Cree sus propios paneles](../learn/tutorial-logs-dashboards.md) que muestren datos de Azure SQL.
- [Cree alertas](../platform/alerts-overview.md) cuando se produzcan eventos específicos de Azure SQL.

