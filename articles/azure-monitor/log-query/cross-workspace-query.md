---
title: Consulta entre recursos con Azure Monitor | Microsoft Docs
description: En este artículo se describe cómo se puede consultar los recursos de varias áreas de trabajo y en la aplicación de App Insights de la suscripción.
ms.subservice: logs
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 05/01/2020
ms.openlocfilehash: 7cfa3d5652e13ddc88db70674049069a5b391297
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87322132"
---
# <a name="perform-cross-resource-log-queries-in-azure-monitor"></a>Consulta de registros entre recursos en Azure Monitor  

> [!IMPORTANT]
> Si usa un [recurso de Application Insights basado en el área de trabajo](../app/create-workspace-resource.md), la telemetría se almacena en un área de trabajo de Log Analytics con todos los demás datos del registro. Use la expresión log() para escribir una consulta que incluya la aplicación en varias áreas de trabajo. Si tiene varias aplicaciones en la misma área de trabajo, no se necesita realizar una consulta entre áreas de trabajo.

Anteriormente, con Azure Monitor, solo podía analizar datos desde dentro del área de trabajo actual, lo que limitaba su capacidad de consulta a través de varias áreas de trabajo definidas en la suscripción.  Además, solo puede buscar los elementos de telemetría recopilados de la aplicación basada en web con Application Insights directamente en Application Insights o desde Visual Studio. Esto también convirtió en un desafío analizar de manera nativa los datos operativos y de aplicación en conjunto.

Ahora puede consultar no solo a través de varias áreas de trabajo de Log Analytics, sino también datos desde una aplicación específica de Application Insights del mismo grupo de recursos, otro grupo de recursos u otra suscripción. Gracias a esto se consigue una vista total del sistema de datos. Solo puede realizar estos tipos de consultas en [Log Analytics](./log-query-overview.md).

## <a name="cross-resource-query-limits"></a>Límites de la consulta entre recursos 

* El número de recursos de Application Insights y de áreas de trabajo de Log Analytics que se pueden incluir en una sola consulta se limita a 100.
* No se admite la consulta entre recursos en el Diseñador de vistas. Puede crear una consulta en Log Analytics y anclarla al panel de Azure para [visualizar una consulta de registro](../learn/tutorial-logs-dashboards.md). 
* La consulta entre recursos en las alertas de registro se admite en la nueva [API scheduledQueryRules](/rest/api/monitor/scheduledqueryrules). De forma predeterminada, Azure Monitor usa la [API de alerta heredada de Log Analytics](../platform/api-alerts.md) para crear nuevas reglas de alerta de registro desde Azure Portal, a menos que cambie de [API de alerta de registro heredada](../platform/alerts-log-api-switch.md#process-of-switching-from-legacy-log-alerts-api). Después del cambio, la nueva API se convierte en la predeterminada para las nuevas reglas de alerta en Azure Portal y le permite crear reglas de alertas de registro de consulta entre recursos. Puede crear reglas de alertas de registro de consulta entre recursos sin realizar el cambio mediante la [plantilla de Azure Resource Manager para la API scheduledQueryRules](../platform/alerts-log.md#log-alert-with-cross-resource-query-using-azure-resource-template); sin embargo, esta regla de alertas se puede administrar mediante la [API scheduledQueryRules](/rest/api/monitor/scheduledqueryrules) y no desde Azure Portal.


## <a name="querying-across-log-analytics-workspaces-and-from-application-insights"></a>Consultas a través de áreas de trabajo de Log Analytics y desde Application Insights
Para hacer referencia a otra área de trabajo en la consulta, use el identificador del [*área de trabajo*](./workspace-expression.md), y, para una aplicación de Application Insights, use el identificador de la [*aplicación*](./app-expression.md).  

### <a name="identifying-workspace-resources"></a>Identificación de los recursos del área de trabajo
En los ejemplos siguientes se muestran consultas en las áreas de trabajo de Log Analytics que devuelven recuentos de registros resumidos desde la tabla Update en un área de trabajo denominada *contosoretail-it*. 

La identificación de un área de trabajo se puede lograr de varias maneras:

* Nombre del recurso:- es un nombre legible del área de trabajo; a veces se denomina *nombre del componente*. 

    `workspace("contosoretail-it").Update | count`

* Nombre completo: es el nombre completo del área de trabajo, compuesto por el nombre de la suscripción, el grupo de recursos y el nombre del componente con este formato: *nombreSuscripción/grupoDeRecursos/nombreDeComponente*. 

    `workspace('contoso/contosoretail/contosoretail-it').Update | count`

    >[!NOTE]
    >Dado que los nombres de suscripción de Azure no son únicos, este identificador podría ser ambiguo. 
    >

* Identificador del área de trabajo: es el identificador único e inmutable asignado a cada área de trabajo que se representa como identificador único global (GUID).

    `workspace("b459b4u5-912x-46d5-9cb1-p43069212nb4").Update | count`

* Identificador de recurso de Azure: identidad única definida por Azure del área de trabajo. Se usa cuando el nombre del recurso es ambiguo.  Para las áreas de trabajo, el formato es: */subscriptions/subscriptionId/resourcegroups/resourceGroup/providers/microsoft.OperationalInsights/workspaces/componentName*.  

    Por ejemplo:
    ``` 
    workspace("/subscriptions/e427519-5645-8x4e-1v67-3b84b59a1985/resourcegroups/ContosoAzureHQ/providers/Microsoft.OperationalInsights/workspaces/contosoretail-it").Update | count
    ```

### <a name="identifying-an-application"></a>Identificación de una aplicación
Los ejemplos siguientes devuelven un recuento resumido de las solicitudes realizadas en una aplicación denominada *fabrikamapp* de Application Insights. 

Una aplicación de Application Insights se puede identificar con la expresión *app(Identifier)* .  El argumento *Identifier* especifica la aplicación que usa uno de los siguientes:

* Nombre del recurso: es el nombre legible de la aplicación; a veces se denomina *nombre del componente*.  

    `app("fabrikamapp")`

    >[!NOTE]
    >En la identificación de una aplicación por nombre se da por supuesto que es único en todas las suscripciones accesibles. Si tiene varias aplicaciones con el nombre especificado, la consulta produce un error debido a la ambigüedad. En este caso debe usar uno de los otros identificadores.

* Nombre completo: es el nombre completo de la aplicación, compuesto por el nombre de la suscripción, el grupo de recursos y el nombre del componente con este formato: *nombreSuscripción/grupoDeRecursos/nombreDeComponente*. 

    `app("AI-Prototype/Fabrikam/fabrikamapp").requests | count`

     >[!NOTE]
    >Dado que los nombres de suscripción de Azure no son únicos, este identificador podría ser ambiguo. 
    >

* ID: el identificador único global de la aplicación.

    `app("b459b4f6-912x-46d5-9cb1-b43069212ab4").requests | count`

* Identificador de recurso de Azure: identidad única definida por Azure de la aplicación. Se usa cuando el nombre del recurso es ambiguo. El formato es: */subscriptions/IdSuscripción/resourcegroups/grupoDeRecursos/providers/microsoft.OperationalInsights/components/nombreDeComponente*.  

    Por ejemplo:
    ```
    app("/subscriptions/b459b4f6-912x-46d5-9cb1-b43069212ab4/resourcegroups/Fabrikam/providers/microsoft.insights/components/fabrikamapp").requests | count
    ```

### <a name="performing-a-query-across-multiple-resources"></a>Realizar una consulta en varios recursos
Puede consultar varios recursos desde cualquiera de las instancias de recursos; estas pueden ser áreas de trabajo y aplicaciones combinadas.
    
Ejemplo de consulta en dos áreas de trabajo:    

```
union Update, workspace("contosoretail-it").Update, workspace("b459b4u5-912x-46d5-9cb1-p43069212nb4").Update
| where TimeGenerated >= ago(1h)
| where UpdateState == "Needed"
| summarize dcount(Computer) by Classification
```

## <a name="using-cross-resource-query-for-multiple-resources"></a>Uso de una consulta entre recursos para varios recursos
Al usar las consultas entre recursos para correlacionar datos procedentes de varios recursos del área de trabajo de Log Analytics y de Application Insights, la consulta puede ser compleja y difícil de mantener. Debe aprovechar las [funciones de las consultas de registro de Azure Monitor](functions.md) para separar la lógica de consulta del ámbito de los recursos de la consulta, lo que simplifica la estructura de la consulta. En el ejemplo siguiente se muestra cómo puede supervisar varios recursos de Application Insights y visualizar el número de solicitudes erróneas por nombre de la aplicación. 

Cree una consulta similar a la siguiente que haga referencia al ámbito de recursos de Application Insights. El comando `withsource= SourceApp` agrega una columna que designa el nombre de la aplicación que envió el registro. [Guarde la consulta como función](functions.md#create-a-function) con el alias _applicationsScoping_.

```Kusto
// crossResource function that scopes my Application Insights resources
union withsource= SourceApp
app('Contoso-app1').requests, 
app('Contoso-app2').requests,
app('Contoso-app3').requests,
app('Contoso-app4').requests,
app('Contoso-app5').requests
```



Ahora puede [usar esta función](./functions.md#use-a-function) en una consulta entre recursos similar a la siguiente. El alias de la función _applicationsScoping_ devuelve la unión de la tabla de solicitudes de todas las aplicaciones definidas. A continuación, la consulta filtra las solicitudes erróneas y visualiza las tendencias por aplicación. El operador _parse_ es opcional en este ejemplo. Extrae el nombre de la aplicación de la propiedad _SourceApp_.

```Kusto
applicationsScoping 
| where timestamp > ago(12h)
| where success == 'False'
| parse SourceApp with * '(' applicationName ')' * 
| summarize count() by applicationName, bin(timestamp, 1h) 
| render timechart
```

>[!NOTE]
>Este método no se puede usar con alertas de registro porque la validación de acceso de los recursos para las regla de alertas (incluidas las áreas de trabajo y las aplicaciones) se realiza en el momento de la creación de la alerta. No se admite la adición de nuevos recursos a la función después de crear la alerta. Si prefiere usar la función para el ámbito de los recursos en las alertas de registro, debe editar la regla de alertas en el portal o usar una plantilla de Resource Manager para actualizar los recursos de ámbito. También puede incluir la lista de recursos en la consulta de alerta de registro.


![Gráfica de tiempo](media/cross-workspace-query/chart.png)

## <a name="next-steps"></a>Pasos siguientes

- Consulte [Análisis de los datos de Log Analytics en Azure Monitor](log-query-overview.md) para obtener información general de las consultas de registros y cómo se estructuran los datos del registro de Azure Monitor.
- Consulte las [consultas de registros de Azure Monitor](query-language.md) para ver todos los recursos de las consultas de registros de Azure Monitor.

