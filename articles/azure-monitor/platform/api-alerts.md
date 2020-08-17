---
title: Uso de la API REST de alertas de Log Analytics
description: Con la API REST de alertas de Log Analytics, se pueden crear y administrar alertas de Log Analytics.  En este artículo encontrará información detallada sobre la API y varios ejemplos para realizar distintas operaciones.
ms.subservice: logs
ms.topic: conceptual
ms.date: 07/29/2018
ms.openlocfilehash: eec7aeab32aa071ce9d4476b15740c89210f0606
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87322336"
---
# <a name="create-and-manage-alert-rules-in-log-analytics-with-rest-api"></a>Creación y administración de reglas de alerta de Log Analytics con la API de REST 

La API REST de alertas de Log Analytics le permite crear y administrar alertas en Log Analytics.  En este artículo encontrará información detallada sobre la API y varios ejemplos para realizar distintas operaciones.

> [!IMPORTANT]
> Como [se anunció anteriormente](https://azure.microsoft.com/updates/switch-api-preference-log-alerts/), las áreas de trabajo de Log Analytics creadas después del *1 de junio de 2019* podrán administrar reglas de alerta **únicamente** mediante la [API REST](/rest/api/monitor/scheduledqueryrules/) scheduledQueryRules de Azure, la [plantilla de Azure Resource Manager](./alerts-log.md#managing-log-alerts-using-azure-resource-template) y el [cmdlet de PowerShell](./alerts-log.md#managing-log-alerts-using-powershell). Los clientes pueden [cambiar fácilmente su forma preferida de administración de reglas de alertas](./alerts-log-api-switch.md#process-of-switching-from-legacy-log-alerts-api) para que las áreas de trabajo anteriores aprovechen scheduledQueryRules de Azure Monitor como valor predeterminado y consigan muchas de las [nuevas ventajas](./alerts-log-api-switch.md#benefits-of-switching-to-new-azure-api), como la posibilidad de usar cmdlets nativos de PowerShell, el aumento del período de retrospectiva en las reglas, la creación de reglas en un grupo de recursos o una suscripción independientes y mucho más.

La API de REST de búsqueda de Log Analytics es de tipo RESTful y se puede obtener acceso a ella a través de la API de REST de Azure Resource Manager. En este documento encontrará ejemplos donde se tiene acceso a la API desde una línea de comandos de PowerShell a través de [ARMClient](https://github.com/projectkudu/ARMClient), una herramienta de línea de comandos de código abierto que simplifica la tarea de invocar a la API de Azure Resource Manager. El uso de ARMClient y PowerShell es una de las muchas opciones para tener acceso a la API de búsqueda de Log Analytics. Con estas herramientas, puede usar la API de RESTful de Azure Resource Manager para realizar llamadas a las áreas de trabajo de Log Analytics y ejecutar comandos de búsqueda dentro de ellas. La API generará resultados de búsqueda, en formato JSON, lo que le permite usar los resultados de búsqueda de muchas formas distintas mediante programación.

## <a name="prerequisites"></a>Prerrequisitos
Actualmente, solo se pueden crear alertas con una búsqueda guardada en Log Analytics.  Para obtener más información, consulte [API de búsqueda de registros de Log Analytics](../log-query/log-query-overview.md) .

## <a name="schedules"></a>Programaciones
Una búsqueda guardada puede tener una o varias programaciones. La programación define la frecuencia con que se realiza la búsqueda y el intervalo de tiempo en el que se identifican los criterios.
Las programaciones tienen las propiedades de la siguiente tabla.

| Propiedad | Descripción |
|:--- |:--- |
| Intervalo |Frecuencia con que se realiza la búsqueda. Se mide en minutos. |
| QueryTimeSpan |Intervalo de tiempo en el que se evalúan los criterios. Debe ser igual o mayor que Intervalo. Se mide en minutos. |
| Versión |Versión de API en uso.  Actualmente, siempre debe estar establecida en 1. |

Por ejemplo, en una consulta de evento con un valor de Intervalo de 15 minutos y un valor de Timespan de 30 minutos, la consulta se ejecutaría cada 15 minutos y se desencadenaría una alerta si los criterios siguieran evaluándose como True durante un intervalo de 30 minutos.

### <a name="retrieving-schedules"></a>Recuperar programaciones
Use el método Get para recuperar todas las programaciones de una búsqueda guardada.

```powershell
armclient get /subscriptions/{Subscription ID}/resourceGroups/{ResourceGroupName}/providers/Microsoft.OperationalInsights/workspaces/{Workspace Name}/savedSearches/{Search  ID}/schedules?api-version=2015-03-20
```

Use el método Get con un identificador de programación para recuperar una programación concreta de una búsqueda guardada.

```powershell
armclient get /subscriptions/{Subscription ID}/resourceGroups/{ResourceGroupName}/providers/Microsoft.OperationalInsights/workspaces/{Workspace Name}/savedSearches/{Subscription ID}/schedules/{Schedule ID}?api-version=2015-03-20
```

La siguiente es una respuesta de ejemplo de una programación.

```json
{
   "value": [{
      "id": "subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/sampleRG/providers/Microsoft.OperationalInsights/workspaces/MyWorkspace/savedSearches/0f0f4853-17f8-4ed1-9a03-8e888b0d16ec/schedules/a17b53ef-bd70-4ca4-9ead-83b00f2024a8",
      "etag": "W/\"datetime'2016-02-25T20%3A54%3A49.8074679Z'\"",
      "properties": {
         "Interval": 15,
         "QueryTimeSpan": 15,
         "Enabled": true,
      }
   }]
}
```

### <a name="creating-a-schedule"></a>Creación de una programación
Use el método Put con un identificador de programación único para crear una programación nueva.  Dos programaciones no pueden tener el mismo identificador, aunque estén asociadas a diferentes búsquedas guardadas.  Al crear una programación en la consola de Log Analytics, se crea un GUID para el identificador de programación.

> [!NOTE]
> El nombre de todas las búsquedas guardadas, programaciones y acciones creadas con Log Analytics API debe estar en minúsculas.

```powershell
$scheduleJson = "{'properties': { 'Interval': 15, 'QueryTimeSpan':15, 'Enabled':'true' } }"
armclient put /subscriptions/{Subscription ID}/resourceGroups/{ResourceGroupName}/providers/Microsoft.OperationalInsights/workspaces/{Workspace Name}/savedSearches/{Search ID}/schedules/mynewschedule?api-version=2015-03-20 $scheduleJson
```

### <a name="editing-a-schedule"></a>Edición de una programación
Utilice el método Put con un identificador de programación existente para modificar la programación de la misma búsqueda guardada. En el ejemplo siguiente, la programación está deshabilitada. El cuerpo de la solicitud debe incluir el valor *etag* de la programación.

```powershell
$scheduleJson = "{'etag': 'W/\"datetime'2016-02-25T20%3A54%3A49.8074679Z'\""','properties': { 'Interval': 15, 'QueryTimeSpan':15, 'Enabled':'false' } }"
armclient put /subscriptions/{Subscription ID}/resourceGroups/{ResourceGroupName}/providers/Microsoft.OperationalInsights/workspaces/{Workspace Name}/savedSearches/{Search ID}/schedules/mynewschedule?api-version=2015-03-20 $scheduleJson
```

### <a name="deleting-schedules"></a>Eliminar programaciones
Para eliminar una programación, use el método Delete con un identificador de programación.

```powershell
armclient delete /subscriptions/{Subscription ID}/resourceGroups/{ResourceGroupName}/providers/Microsoft.OperationalInsights/workspaces/{Workspace Name}/savedSearches/{Subscription ID}/schedules/{Schedule ID}?api-version=2015-03-20
```

## <a name="actions"></a>Acciones
Una programación puede tener varias acciones. Una acción puede definir uno o varios procesos que se van a realizar (como enviar un correo o iniciar un Runbook), o bien puede definir un umbral que determina si los resultados de una búsqueda coinciden con algunos criterios.  Algunas acciones definen ambos aspectos, de forma que los procesos se realizan cuando se alcance el umbral.

Todas las acciones tienen las propiedades de la siguiente tabla.  Los distintos tipos de alertas tienen diferentes propiedades adicionales, descritas aquí.

| Propiedad | Descripción |
|:--- |:--- |
| `Type` |Tipo de la acción.  Actualmente, los valores posibles son Alert y Webhook. |
| `Name` |Nombre para mostrar de la alerta. |
| `Version` |Versión de API en uso.  Actualmente, siempre debe estar establecida en 1. |

### <a name="retrieving-actions"></a>Recuperar acciones

Use el método Get para recuperar todas las acciones de una programación.

```powershell
armclient get /subscriptions/{Subscription ID}/resourceGroups/{ResourceGroupName}/providers/Microsoft.OperationalInsights/workspaces/{Workspace Name}/savedSearches/{Search  ID}/schedules/{Schedule ID}/actions?api-version=2015-03-20
```

Use el método Get con el identificador de una acción para recuperar esa acción concreta para una programación.

```powershell
armclient get /subscriptions/{Subscription ID}/resourceGroups/{ResourceGroupName}/providers/Microsoft.OperationalInsights/workspaces/{Workspace Name}/savedSearches/{Subscription ID}/schedules/{Schedule ID}/actions/{Action ID}?api-version=2015-03-20
```

### <a name="creating-or-editing-actions"></a>Crear o editar acciones
Para crear una acción, use el método Put con un identificador de acción único de la programación.  Al crear una acción en la consola de Log Analytics, se crea un GUID para el identificador de acción.

> [!NOTE]
> El nombre de todas las búsquedas guardadas, programaciones y acciones creadas con Log Analytics API debe estar en minúsculas.

Utilice el método Put con un identificador de acción existente para modificar la programación de la misma búsqueda guardada.  El cuerpo de la solicitud debe incluir el valor etag de la programación.

El formato de solicitud para crear una acción varía según el tipo de acción. En las secciones siguientes se proporcionan ejemplos.

### <a name="deleting-actions"></a>Eliminar acciones

Use el método Delete con el identificador de acción para eliminar esa acción.

```powershell
armclient delete /subscriptions/{Subscription ID}/resourceGroups/{ResourceGroupName}/providers/Microsoft.OperationalInsights/workspaces/{Workspace Name}/savedSearches/{Subscription ID}/schedules/{Schedule ID}/Actions/{Action ID}?api-version=2015-03-20
```

### <a name="alert-actions"></a>Acciones de alerta
Una programación debe tener una acción de alerta única y exclusivamente.  Las acciones de alerta tienen una o varias de las secciones de la siguiente tabla.  Cada una de ellas se describe con más detalle abajo.

| Sección | Descripción | Uso |
|:--- |:--- |:--- |
| Umbral |Criterios para establecer cuándo se va a ejecutar la acción.| Se requiere para cada alerta, ya sea antes o después de extenderlas a Azure. |
| severity |Etiqueta que se usa para clasificar la alerta cuando se desencadena.| Se requiere para cada alerta, ya sea antes o después de extenderlas a Azure. |
| Suprimir |Opción para dejar de recibir notificaciones de la alerta. | Opcional para cada alerta, ya sea antes o después de extenderlas a Azure. |
| Grupos de acciones |Identificadores para Azure ActionGroup donde se especifican las acciones requeridas, como correos electrónicos, mensajes SMS, llamadas de voz, webhooks, runbooks de Automation, conectores ITSM, etc.| Se requiere una vez que las alertas se extienden a Azure|
| Personalizar las acciones|Modifica la salida estándar para seleccionar acciones en ActionGroup| Opcional para cada alerta, se puede usar después de que las alertas se extienden a Azure. |

### <a name="thresholds"></a>Umbrales
Una acción de alerta debe tener un umbral única y exclusivamente.  Cuando los resultados de una búsqueda guardada coinciden con el umbral de una acción asociada a esa búsqueda, se ejecutan los demás procesos de esa acción.  Una acción también puede contener solo un umbral para que pueda usarse con acciones de otros tipos que no contienen umbrales.

Los umbrales tienen las propiedades de la siguiente tabla.

| Propiedad | Descripción |
|:--- |:--- |
| `Operator` |Operador de la comparación de umbral. <br> gt = Mayor que <br> lt = Menor que |
| `Value` |Valor del umbral. |

Por ejemplo, en una consulta de evento con un valor de Intervalo de 15 minutos, un valor de Timespan de 30 minutos y un valor de Threshold mayor que 10, la consulta se ejecutaría cada 15 minutos y se desencadenaría una alerta si devolviera 10 eventos creados durante un intervalo de 30 minutos.

La siguiente es una respuesta de ejemplo de una acción con un solo umbral.  

```json
"etag": "W/\"datetime'2016-02-25T20%3A54%3A20.1302566Z'\"",
"properties": {
   "Type": "Alert",
   "Name": "My threshold action",
   "Threshold": {
      "Operator": "gt",
      "Value": 10
   },
   "Version": 1
}
```

Use el método Put con un identificador de acción único para crear una acción de umbral para una programación.  

```powershell
$thresholdJson = "{'properties': { 'Name': 'My Threshold', 'Version':'1', 'Type':'Alert', 'Threshold': { 'Operator': 'gt', 'Value': 10 } } }"
armclient put /subscriptions/{Subscription ID}/resourceGroups/{ResourceGroupName}/providers/Microsoft.OperationalInsights/workspaces/{Workspace Name}/savedSearches/{Search ID}/schedules/{Schedule ID}/actions/mythreshold?api-version=2015-03-20 $thresholdJson
```

Use el método Put con un identificador de acción existente para modificar una acción de umbral para una programación.  El cuerpo de la solicitud debe incluir el valor etag de la acción.

```powershell
$thresholdJson = "{'etag': 'W/\"datetime'2016-02-25T20%3A54%3A20.1302566Z'\"','properties': { 'Name': 'My Threshold', 'Version':'1', 'Type':'Alert', 'Threshold': { 'Operator': 'gt', 'Value': 10 } } }"
armclient put /subscriptions/{Subscription ID}/resourceGroups/{ResourceGroupName}/providers/Microsoft.OperationalInsights/workspaces/{Workspace Name}/savedSearches/{Search ID}/schedules/{Schedule ID}/actions/mythreshold?api-version=2015-03-20 $thresholdJson
```

#### <a name="severity"></a>severity
Log Analytics permite clasificar las alertas en categorías, para permitir una administración y una evaluación de prioridades más sencillas. La gravedad de la alerta que se define es: informativa, advertencia y crítica. Estos valores se asignan a la escala de gravedad normalizada de Alertas de Azure como:

|Nivel de gravedad de Log Analytics  |Nivel de gravedad de Alertas de Azure  |
|---------|---------|
|`critical` |Gravedad 0|
|`warning` |Gravedad 1|
|`informational` | Gravedad 2|

La siguiente es una respuesta de ejemplo de una acción con un solo umbral y gravedad. 

```json
"etag": "W/\"datetime'2016-02-25T20%3A54%3A20.1302566Z'\"",
"properties": {
   "Type": "Alert",
   "Name": "My threshold action",
   "Threshold": {
      "Operator": "gt",
      "Value": 10
   },
   "Severity": "critical",
   "Version": 1
}
```

Use el método Put con un identificador de acción único para crear una acción para una programación con gravedad.  

```powershell
$thresholdWithSevJson = "{'properties': { 'Name': 'My Threshold', 'Version':'1','Severity': 'critical', 'Type':'Alert', 'Threshold': { 'Operator': 'gt', 'Value': 10 } } }"
armclient put /subscriptions/{Subscription ID}/resourceGroups/{ResourceGroupName}/providers/Microsoft.OperationalInsights/workspaces/{Workspace Name}/savedSearches/{Search ID}/schedules/{Schedule ID}/actions/mythreshold?api-version=2015-03-20 $thresholdWithSevJson
```

Use el método Put con un identificador de acción existente para modificar una acción de gravedad para una programación.  El cuerpo de la solicitud debe incluir el valor etag de la acción.

```powershell
$thresholdWithSevJson = "{'etag': 'W/\"datetime'2016-02-25T20%3A54%3A20.1302566Z'\"','properties': { 'Name': 'My Threshold', 'Version':'1','Severity': 'critical', 'Type':'Alert', 'Threshold': { 'Operator': 'gt', 'Value': 10 } } }"
armclient put /subscriptions/{Subscription ID}/resourceGroups/{ResourceGroupName}/providers/Microsoft.OperationalInsights/workspaces/{Workspace Name}/savedSearches/{Search ID}/schedules/{Schedule ID}/actions/mythreshold?api-version=2015-03-20 $thresholdWithSevJson
```

#### <a name="suppress"></a>Suprimir
Las alertas de consulta basadas en Log Analytics se activarán cada vez que se alcance o supere el umbral. Según la lógica implicada en la consulta, esto puede producir que la alerta se active en una serie de intervalos y, por tanto, también se envíen notificaciones constantemente. Para evitar estos escenarios, un usuario puede establecer la opción de suprimir para indicar a Log Analytics que debe esperar durante un período de tiempo estipulado antes de que la notificación se desencadene por segunda vez para la regla de alertas. Por lo tanto, si la supresión se establece en 30 minutos, la alerta se activará la primera vez y enviará las notificaciones configuradas. Pero, a continuación, espera 30 minutos antes de volver a utilizar la notificación para la regla de alertas. En el período transitorio, la regla de alertas se seguirá ejecutando: Log Analytics solo suprime la notificación durante el tiempo especificado, independientemente de cuántas veces se desencadena la regla de alertas durante este período.

La propiedad de supresión de la regla de alertas de Log Analytics se especifica con el valor *Throttling* y el período de supresión mediante el valor *DurationInMinutes*.

La siguiente es una respuesta de ejemplo de una acción con un solo umbral, la gravedad y la propiedad de supresión.

```json
"etag": "W/\"datetime'2016-02-25T20%3A54%3A20.1302566Z'\"",
"properties": {
   "Type": "Alert",
   "Name": "My threshold action",
   "Threshold": {
      "Operator": "gt",
      "Value": 10
   },
   "Throttling": {
   "DurationInMinutes": 30
   },
   "Severity": "critical",
   "Version": 1
}
```

Use el método Put con un identificador de acción único para crear una acción para una programación con gravedad.  

```powershell
$AlertSuppressJson = "{'properties': { 'Name': 'My Threshold', 'Version':'1','Severity': 'critical', 'Type':'Alert', 'Throttling': { 'DurationInMinutes': 30 },'Threshold': { 'Operator': 'gt', 'Value': 10 } } }"
armclient put /subscriptions/{Subscription ID}/resourceGroups/{ResourceGroupName}/providers/Microsoft.OperationalInsights/workspaces/{Workspace Name}/savedSearches/{Search ID}/schedules/{Schedule ID}/actions/myalert?api-version=2015-03-20 $AlertSuppressJson
```

Use el método Put con un identificador de acción existente para modificar una acción de gravedad para una programación.  El cuerpo de la solicitud debe incluir el valor etag de la acción.

```powershell
$AlertSuppressJson = "{'etag': 'W/\"datetime'2016-02-25T20%3A54%3A20.1302566Z'\"','properties': { 'Name': 'My Threshold', 'Version':'1','Severity': 'critical', 'Type':'Alert', 'Throttling': { 'DurationInMinutes': 30 },'Threshold': { 'Operator': 'gt', 'Value': 10 } } }"
armclient put /subscriptions/{Subscription ID}/resourceGroups/{ResourceGroupName}/providers/Microsoft.OperationalInsights/workspaces/{Workspace Name}/savedSearches/{Search ID}/schedules/{Schedule ID}/actions/myalert?api-version=2015-03-20 $AlertSuppressJson
```

#### <a name="action-groups"></a>Grupos de acciones
Todas las alertas de Azure usan el grupo de acciones como el mecanismo predeterminado para controlar las acciones. Con el grupo de acciones, puede especificar las acciones una vez y, luego, asociar el grupo de acciones a varias alertas en todo Azure, sin la necesidad de declarar repetidamente una y otra vez las mismas acciones. Los grupos de acciones admiten varias acciones, incluido el correo electrónico, SMS, llamadas de voz, conexión ITSM, runbook de Automation, URI de webhook, etc. 

Para los usuarios que han extendido sus alertas a Azure, ahora una programación tendrá los detalles del grupo de acciones junto con el umbral, para así poder crear una alerta. Antes de crear una alerta, es necesario definir los detalles de correo electrónico, las direcciones URL de webhooks, los detalles de automatización de runbooks y otras acciones dentro de un grupo de acciones. Es posible crear un [grupo de acciones desde Azure Monitor](./action-groups.md) en el portal o usar la [API Action Group](/rest/api/monitor/actiongroups).

Para agregar la asociación de un grupo de acciones a una alerta, especifique el identificador de Azure Resource Manager único del grupo de acciones en la definición de la alerta. A continuación, aparece una ilustración de ejemplo:

```json
"etag": "W/\"datetime'2017-12-13T10%3A52%3A21.1697364Z'\"",
"properties": {
   "Type": "Alert",
   "Name": "test-alert",
   "Description": "I need to put a description here",
   "Threshold": {
      "Operator": "gt",
      "Value": 12
   },
   "AzNsNotification": {
      "GroupIds": [
         "/subscriptions/1234a45-123d-4321-12aa-123b12a5678/resourcegroups/my-resource-group/providers/microsoft.insights/actiongroups/test-actiongroup"
      ]
   },
   "Severity": "critical",
   "Version": 1
}
```

Use el método Put con un identificación de acción único para asociar el grupo de acciones ya existente para una programación.  Esta es una ilustración de ejemplo del uso.

```powershell
$AzNsJson = "{'properties': { 'Name': 'test-alert', 'Version':'1', 'Type':'Alert', 'Threshold': { 'Operator': 'gt', 'Value': 12 },'Severity': 'critical', 'AzNsNotification': {'GroupIds': ['subscriptions/1234a45-123d-4321-12aa-123b12a5678/resourcegroups/my-resource-group/providers/microsoft.insights/actiongroups/test-actiongroup']} } }"
armclient put /subscriptions/{Subscription ID}/resourceGroups/{Resource Group Name}/Microsoft.OperationalInsights/workspaces/{Workspace Name}/savedSearches/{Search ID}/schedules/{Schedule ID}/actions/myAzNsaction?api-version=2015-03-20 $AzNsJson
```

Use el método Put con un identificador de acción existente para modificar un grupo de acciones asociado para una programación.  El cuerpo de la solicitud debe incluir el valor etag de la acción.

```powershell
$AzNsJson = "{'etag': 'datetime'2017-12-13T10%3A52%3A21.1697364Z'\"', 'properties': { 'Name': 'test-alert', 'Version':'1', 'Type':'Alert', 'Threshold': { 'Operator': 'gt', 'Value': 12 },'Severity': 'critical', 'AzNsNotification': { 'GroupIds': ['subscriptions/1234a45-123d-4321-12aa-123b12a5678/resourcegroups/my-resource-group/providers/microsoft.insights/actiongroups/test-actiongroup'] } } }"
armclient put /subscriptions/{Subscription ID}/resourceGroups/{Resource Group Name}/Microsoft.OperationalInsights/workspaces/{Workspace Name}/savedSearches/{Search ID}/schedules/{Schedule ID}/actions/myAzNsaction?api-version=2015-03-20 $AzNsJson
```

#### <a name="customize-actions"></a>Personalizar las acciones
En el caso de las acciones predeterminadas, siga la plantilla y el formato estándar para las notificaciones. Pero los usuarios pueden personalizar algunas acciones, incluso si los controlan los grupos de acciones. Actualmente, es posible personalizar el asunto del correo electrónico y la carga del webhook.

##### <a name="customize-e-mail-subject-for-action-group"></a>Personalización del asunto del correo electrónico para el grupo de acciones
De manera predeterminada, el asunto del correo electrónico para las alertas es: Notificación de alerta `<AlertName>` para `<WorkspaceName>`. Pero se puede personalizar, de modo que pueda especificar palabras o etiquetas, para permitirle emplear fácilmente reglas de filtro en la Bandeja de entrada. Los detalles del encabezado del correo electrónico personalizado se deben enviar junto con los detalles de ActionGroup, como se muestra en el ejemplo siguiente.

```json
"etag": "W/\"datetime'2017-12-13T10%3A52%3A21.1697364Z'\"",
"properties": {
   "Type": "Alert",
   "Name": "test-alert",
   "Description": "I need to put a description here",
   "Threshold": {
      "Operator": "gt",
      "Value": 12
   },
   "AzNsNotification": {
      "GroupIds": [
         "/subscriptions/1234a45-123d-4321-12aa-123b12a5678/resourcegroups/my-resource-group/providers/microsoft.insights/actiongroups/test-actiongroup"
      ],
      "CustomEmailSubject": "Azure Alert fired"
   },
   "Severity": "critical",
   "Version": 1
}
```

Use el método Put con un identificación de acción único para asociar el grupo de acciones ya existente con una personalización para una programación.  Esta es una ilustración de ejemplo del uso.

```powershell
$AzNsJson = "{'properties': { 'Name': 'test-alert', 'Version':'1', 'Type':'Alert', 'Threshold': { 'Operator': 'gt', 'Value': 12 },'Severity': 'critical', 'AzNsNotification': {'GroupIds': ['subscriptions/1234a45-123d-4321-12aa-123b12a5678/resourcegroups/my-resource-group/providers/microsoft.insights/actiongroups/test-actiongroup'], 'CustomEmailSubject': 'Azure Alert fired'} } }"
armclient put /subscriptions/{Subscription ID}/resourceGroups/{Resource Group Name}/Microsoft.OperationalInsights/workspaces/{Workspace Name}/savedSearches/{Search ID}/schedules/{Schedule ID}/actions/myAzNsaction?api-version=2015-03-20 $AzNsJson
```

Use el método Put con un identificador de acción existente para modificar un grupo de acciones asociado para una programación.  El cuerpo de la solicitud debe incluir el valor etag de la acción.

```powershell
$AzNsJson = "{'etag': 'datetime'2017-12-13T10%3A52%3A21.1697364Z'\"', 'properties': { 'Name': 'test-alert', 'Version':'1', 'Type':'Alert', 'Threshold': { 'Operator': 'gt', 'Value': 12 },'Severity': 'critical', 'AzNsNotification': {'GroupIds': ['subscriptions/1234a45-123d-4321-12aa-123b12a5678/resourcegroups/my-resource-group/providers/microsoft.insights/actiongroups/test-actiongroup']}, 'CustomEmailSubject': 'Azure Alert fired' } }"
armclient put /subscriptions/{Subscription ID}/resourceGroups/{Resource Group Name}/Microsoft.OperationalInsights/workspaces/{Workspace Name}/savedSearches/{Search ID}/schedules/{Schedule ID}/actions/myAzNsaction?api-version=2015-03-20 $AzNsJson
```

##### <a name="customize-webhook-payload-for-action-group"></a>Personalización de la carga de Webhook para un grupo de acciones
De manera predeterminada, el webhook enviado a través del grupo de acciones para Log Analytics tiene una estructura fija. Sin embargo, un usuario puede personalizar la carga de JSON si usa variables específicas admitidas para cumplir con los requisitos del punto de conexión del webhook. Para más información, consulte [Acciones de webhook para reglas de alertas de registro](./alerts-log-webhook.md). 

Los detalles del webhook personalizado se deben enviar junto con los detalles de ActionGroup y se aplicarán a todo los URI del webhook especificados dentro del grupo de acciones, como se indica en el ejemplo a continuación.

```json
"etag": "W/\"datetime'2017-12-13T10%3A52%3A21.1697364Z'\"",
"properties": {
   "Type": "Alert",
   "Name": "test-alert",
   "Description": "I need to put a description here",
   "Threshold": {
      "Operator": "gt",
      "Value": 12
   },
   "AzNsNotification": {
      "GroupIds": [
         "/subscriptions/1234a45-123d-4321-12aa-123b12a5678/resourcegroups/my-resource-group/providers/microsoft.insights/actiongroups/test-actiongroup"
      ],
   "CustomWebhookPayload": "{\"field1\":\"value1\",\"field2\":\"value2\"}",
   "CustomEmailSubject": "Azure Alert fired"
   },
   "Severity": "critical",
   "Version": 1
},
```

Use el método Put con un identificación de acción único para asociar el grupo de acciones ya existente con una personalización para una programación.  Esta es una ilustración de ejemplo del uso.

```powershell
$AzNsJson = "{'properties': { 'Name': 'test-alert', 'Version':'1', 'Type':'Alert', 'Threshold': { 'Operator': 'gt', 'Value': 12 },'Severity': 'critical', 'AzNsNotification': {'GroupIds': ['subscriptions/1234a45-123d-4321-12aa-123b12a5678/resourcegroups/my-resource-group/providers/microsoft.insights/actiongroups/test-actiongroup'], 'CustomEmailSubject': 'Azure Alert fired','CustomWebhookPayload': '{\"field1\":\"value1\",\"field2\":\"value2\"}'} } }"
armclient put /subscriptions/{Subscription ID}/resourceGroups/{Resource Group Name}/Microsoft.OperationalInsights/workspaces/{Workspace Name}/savedSearches/{Search ID}/schedules/{Schedule ID}/actions/myAzNsaction?api-version=2015-03-20 $AzNsJson
```

Use el método Put con un identificador de acción existente para modificar un grupo de acciones asociado para una programación.  El cuerpo de la solicitud debe incluir el valor etag de la acción.

```powershell
$AzNsJson = "{'etag': 'datetime'2017-12-13T10%3A52%3A21.1697364Z'\"', 'properties': { 'Name': 'test-alert', 'Version':'1', 'Type':'Alert', 'Threshold': { 'Operator': 'gt', 'Value': 12 },'Severity': 'critical', 'AzNsNotification': {'GroupIds': ['subscriptions/1234a45-123d-4321-12aa-123b12a5678/resourcegroups/my-resource-group/providers/microsoft.insights/actiongroups/test-actiongroup']}, 'CustomEmailSubject': 'Azure Alert fired','CustomWebhookPayload': '{\"field1\":\"value1\",\"field2\":\"value2\"}' } }"
armclient put /subscriptions/{Subscription ID}/resourceGroups/{Resource Group Name}/Microsoft.OperationalInsights/workspaces/{Workspace Name}/savedSearches/{Search ID}/schedules/{Schedule ID}/actions/myAzNsaction?api-version=2015-03-20 $AzNsJson
```

## <a name="next-steps"></a>Pasos siguientes

* Use la [API de búsqueda de registros de Log Analytics](../log-query/log-query-overview.md) en Log Analytics.
* Más información sobre las [alertas de registro en Azure Monitor](./alerts-unified-log.md)
* [Creación, edición o administración de reglas de alertas de registro en Azure Monitor](./alerts-log.md)

