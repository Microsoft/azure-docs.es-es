---
title: Registros de recursos de Azure
description: Aprenda a transmitir registros de Azure Diagnostics a un área de trabajo de Log Analytics en Azure Monitor.
author: bwren
services: azure-monitor
ms.topic: conceptual
ms.date: 07/17/2019
ms.author: bwren
ms.subservice: logs
ms.openlocfilehash: ccf470abadb28919e4fca3c4862b71946a5bb204
ms.sourcegitcommit: fbb66a827e67440b9d05049decfb434257e56d2d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/05/2020
ms.locfileid: "87800507"
---
# <a name="azure-resource-logs"></a>Registros de recursos de Azure
Los registros de recursos de Azure son [registros de plataforma](platform-logs-overview.md) que proporcionan información sobre las operaciones que se realizaron en un recurso de Azure. El contenido de estos registros de recurso varía según el servicio de Azure y el tipo de recurso. Los registros de recurso no se recopilan de forma predeterminada. Debe crear una configuración de diagnóstico para cada recurso de Azure con el fin de enviar sus registros de recursos a un área de trabajo Log Analytics que se usará con [registros de Azure Monitor](data-platform-logs.md), Azure Event Hubs para reenviar fuera de Azure o Azure Storage para el archivado.

Consulte [Creación de una configuración de diagnóstico para enviar registros de plataforma y métricas a diferentes destinos](diagnostic-settings.md) para obtener más información sobre cómo crear una configuración de diagnóstico, así como [Implementación de Azure Monitor a escala mediante Azure Policy](../deploy-scale.md) para obtener más información sobre el uso de Azure Policy para crear automáticamente una configuración de diagnóstico para cada recurso de Azure que cree.

## <a name="send-to-log-analytics-workspace"></a>Envío al área de trabajo de Log Analytics
 Envíe los registros de recursos a un área de trabajo de Log Analytics para habilitar las características de los [registros de Azure Monitor](data-platform-logs.md), entre lo que se incluye lo siguiente:

- Correlacionar los datos del registro de recurso con otros datos de supervisión recopilados por Azure Monitor.
- Consolidar las entradas de registro de varios inquilinos, suscripciones y recursos de Azure en una ubicación para su análisis conjunto.
- Usar las consultas de registro para realizar un análisis complejo y obtener información detallada sobre los datos de registro.
- Usar las alertas de registro con lógicas de alertas complejas.

[Cree una configuración de diagnóstico](diagnostic-settings.md) para enviar los registro de recursos a un área de trabajo de Log Analytics. Estos datos se almacenan en tablas, tal como se describe en [Estructura de registros de Azure Monitor](../log-query/logs-structure.md). Las tablas que usan los registros de recurso dependen del tipo de colección del recurso:

- Azure Diagnostics: todos los datos se escriben en la tabla _AzureDiagnostics_.
- Específicos del recurso: los datos se escriben en una tabla individual para cada categoría del recurso.

### <a name="azure-diagnostics-mode"></a>Modo Azure Diagnostics 
En este modo se recopilan todos los datos de cualquier configuración de diagnóstico en la tabla _AzureDiagnostics_. Este es el método heredado que usa hoy la mayoría de los servicios de Azure. Dado que varios recursos envían datos a la misma tabla, el esquema de esta tabla es el superconjunto de los esquemas de todos los tipos de datos diferentes que se recopilan.

Considere el ejemplo siguiente, donde se recopila la configuración de diagnóstico en la misma área de trabajo para los siguientes tipos de datos:

- Registros de auditoría del servicio 1 (con un esquema que consta de las columnas A, B y C)  
- Registros de errores del servicio 1 (con un esquema que consta de las columnas D, E y F)  
- Registros de auditoría del servicio 2 (con un esquema que consta de las columnas G, H e I)  

La tabla AzureDiagnostics tendrá el siguiente aspecto:  

| ResourceProvider    | Category     | A  | B  | C  | D  | E  | F  | G  | H  | I  |
| -- | -- | -- | -- | -- | -- | -- | -- | -- | -- | -- |
| Microsoft.Service1 | AuditLogs    | x1 | y1 | z1 |    |    |    |    |    |    |
| Microsoft.Service1 | ErrorLogs    |    |    |    | q1 | w1 | e1 |    |    |    |
| Microsoft.Service2 | AuditLogs    |    |    |    |    |    |    | j1 | k1 | l1 |
| Microsoft.Service1 | ErrorLogs    |    |    |    | q2 | w2 | e2 |    |    |    |
| Microsoft.Service2 | AuditLogs    |    |    |    |    |    |    | j3 | k3 | l3 |
| Microsoft.Service1 | AuditLogs    | x5 | y5 | z5 |    |    |    |    |    |    |
| … |

### <a name="resource-specific"></a>Específico del recurso
En este modo se crean tablas individuales en el área de trabajo seleccionada para cada categoría seleccionada en la configuración de diagnóstico. Se recomienda este método porque facilita el trabajo con los datos en las consultas de registro, proporciona mejor capacidad de detección de esquemas y su estructura, mejora el rendimiento en la latencia de ingesta y en los tiempos de consulta, y ofrece la capacidad de conceder derechos de RBAC en un tabla específica. Todos los servicios de Azure se migrarán finalmente al modo específico del recurso. 

En el ejemplo anterior, esto daría lugar a la creación de tres tablas:
 
- Tabla *Service1AuditLogs*:

    | Proveedor de recursos | Category | A | B | C |
    | -- | -- | -- | -- | -- |
    | Service1 | AuditLogs | x1 | y1 | z1 |
    | Service1 | AuditLogs | x5 | y5 | z5 |
    | … |

- Tabla *Service1ErrorLogs*:  

    | Proveedor de recursos | Category | D | E | F |
    | -- | -- | -- | -- | -- | 
    | Service1 | ErrorLogs |  q1 | w1 | e1 |
    | Service1 | ErrorLogs |  q2 | w2 | e2 |
    | … |

- Tabla *Service2AuditLogs*:  

    | Proveedor de recursos | Category | G | H | I |
    | -- | -- | -- | -- | -- |
    | Service2 | AuditLogs | j1 | k1 | l1|
    | Service2 | AuditLogs | j3 | k3 | l3|
    | … |



### <a name="select-the-collection-mode"></a>Selección del modo de recopilación
La mayoría de los recursos de Azure escribirán los datos en el área de trabajo en modo **Azure Diagnostics** o **específico del recurso** sin que usted pueda elegir. Consulte la [documentación para cada servicio](./resource-logs-schema.md) para más información sobre el modo que usa. Todos los servicios de Azure usarán finalmente el modo específico del recurso. Como parte de esta transición, algunos recursos permitirán seleccionar un modo en la configuración de diagnóstico. Especifique el modo específico del recurso para las configuraciones de diagnóstico nuevas, ya que esto facilita la administración de los datos y puede ayudar a evitar migraciones complejas posteriores.
  
   ![Selector del modo de configuración de diagnósticos](media/resource-logs-collect-workspace/diagnostic-settings-mode-selector.png)

> [!NOTE]
> Para ver un ejemplo de configuración del modo de recopilación mediante una plantilla de Resource Manager, consulte [Ejemplos de plantillas de Resource Manager para la configuración de diagnóstico en Azure Monitor](../samples/resource-manager-diagnostic-settings.md#diagnostic-setting-for-recovery-services-vault).


En el modo específico del recurso se puede modificar una configuración de diagnóstico existente. En este caso, los datos que ya se han recopilado permanecerán en la tabla _AzureDiagnostics_ hasta que se eliminen de acuerdo con la configuración de retención del área de trabajo. Los nuevos datos se recopilarán en la tabla dedicada. Use el operador [union](/azure/kusto/query/unionoperator) para consultar datos en ambas tablas.

Puede continuar por el blog [Actualizaciones de Azure](https://azure.microsoft.com/updates/) para ver noticias sobre los servicios de Azure que admiten el modo específico del recurso.

### <a name="column-limit-in-azurediagnostics"></a>Límite de columnas en AzureDiagnostics
En los registros de Azure Monitor hay un límite de 500 propiedades por tabla. Una vez alcanzado este límite, cualquier fila que contenga datos con cualquier propiedad fuera de las primeras 500 se eliminará en el momento de la ingesta. Estos límites afectan particularmente a la tabla *AzureDiagnostics*, ya que incluye propiedades para todos los servicios de Azure que escriben en ella.

Si va a recopilar registros de recursos de varios servicios, _AzureDiagnostics_ puede superar este límite, y se omitirán datos. Hasta que todos los servicios de Azure admitan el modo específico del recurso, debe configurar los recursos para escribir en varias áreas de trabajo con el fin de reducir la posibilidad de alcanzar el límite de 500 columnas.

### <a name="azure-data-factory"></a>Azure Data Factory
Azure Data Factory, debido a un conjunto detallado de registros, es un servicio del cual se sabe que escribe un gran número de columnas y que puede provocar que _AzureDiagnostics_ supere el límite. Para cualquier configuración de diagnóstico configurada antes de que se habilitara el modo específico del recurso, se creará una nueva columna para cada parámetro de usuario con nombre único en cualquier actividad. Se crearán más columnas debido a la naturaleza detallada de las entradas y salidas de la actividad.
 
Debe migrar los registros para usar el modo específico del recurso lo antes posible. Si no puede hacerlo inmediatamente, una solución alternativa provisional es aislar los registros de Azure Data Factory en su propia área de trabajo para minimizar la posibilidad de que estos registros afecten a otros tipos de registro que se recopilan en las áreas de trabajo.


## <a name="send-to-azure-event-hubs"></a>Envío a Azure Event Hubs
Envíe los registros de recursos al centro de eventos para enviarlos fuera de Azure, por ejemplo, a un SIEM de terceros o a otras soluciones de Log Analytics. Los registros de recursos de los centros de eventos se consumen en formato JSON con un elemento `records` que contiene los registros de cada carga. El esquema depende del tipo de recurso, tal y como se describe en [Esquema específico del servicio y común para los registros de recursos de Azure](resource-logs-schema.md).

A continuación se muestran datos de salida de ejemplo de Event Hubs para un registro de recursos:

```json
{
    "records": [
        {
            "time": "2019-07-15T18:00:22.6235064Z",
            "workflowId": "/SUBSCRIPTIONS/00000000-0000-0000-0000-000000000000/RESOURCEGROUPS/JOHNKEMTEST/PROVIDERS/MICROSOFT.LOGIC/WORKFLOWS/JOHNKEMTESTLA",
            "resourceId": "/SUBSCRIPTIONS/00000000-0000-0000-0000-000000000000/RESOURCEGROUPS/JOHNKEMTEST/PROVIDERS/MICROSOFT.LOGIC/WORKFLOWS/JOHNKEMTESTLA/RUNS/08587330013509921957/ACTIONS/SEND_EMAIL",
            "category": "WorkflowRuntime",
            "level": "Error",
            "operationName": "Microsoft.Logic/workflows/workflowActionCompleted",
            "properties": {
                "$schema": "2016-04-01-preview",
                "startTime": "2016-07-15T17:58:55.048482Z",
                "endTime": "2016-07-15T18:00:22.4109204Z",
                "status": "Failed",
                "code": "BadGateway",
                "resource": {
                    "subscriptionId": "00000000-0000-0000-0000-000000000000",
                    "resourceGroupName": "JohnKemTest",
                    "workflowId": "243aac67fe904cf195d4a28297803785",
                    "workflowName": "JohnKemTestLA",
                    "runId": "08587330013509921957",
                    "location": "westus",
                    "actionName": "Send_email"
                },
                "correlation": {
                    "actionTrackingId": "29a9862f-969b-4c70-90c4-dfbdc814e413",
                    "clientTrackingId": "08587330013509921958"
                }
            }
        },
        {
            "time": "2019-07-15T18:01:15.7532989Z",
            "workflowId": "/SUBSCRIPTIONS/00000000-0000-0000-0000-000000000000/RESOURCEGROUPS/JOHNKEMTEST/PROVIDERS/MICROSOFT.LOGIC/WORKFLOWS/JOHNKEMTESTLA",
            "resourceId": "/SUBSCRIPTIONS/00000000-0000-0000-0000-000000000000/RESOURCEGROUPS/JOHNKEMTEST/PROVIDERS/MICROSOFT.LOGIC/WORKFLOWS/JOHNKEMTESTLA/RUNS/08587330012106702630/ACTIONS/SEND_EMAIL",
            "category": "WorkflowRuntime",
            "level": "Information",
            "operationName": "Microsoft.Logic/workflows/workflowActionStarted",
            "properties": {
                "$schema": "2016-04-01-preview",
                "startTime": "2016-07-15T18:01:15.5828115Z",
                "status": "Running",
                "resource": {
                    "subscriptionId": "00000000-0000-0000-0000-000000000000",
                    "resourceGroupName": "JohnKemTest",
                    "workflowId": "243aac67fe904cf195d4a28297803785",
                    "workflowName": "JohnKemTestLA",
                    "runId": "08587330012106702630",
                    "location": "westus",
                    "actionName": "Send_email"
                },
                "correlation": {
                    "actionTrackingId": "042fb72c-7bd4-439e-89eb-3cf4409d429e",
                    "clientTrackingId": "08587330012106702632"
                }
            }
        }
    ]
}
```

## <a name="send-to-azure-storage"></a>Envío a Azure Storage
Envíe registros de recursos a Azure Storage para conservarlos para archivarlos. Una vez creada la configuración de diagnóstico, en cuanto se produzca un evento en una de las categorías de registro que haya habilitado se creará un contenedor de almacenamiento en la cuenta de almacenamiento. Los blobs del contenedor usan la siguiente convención de nomenclatura:

```
insights-logs-{log category name}/resourceId=/SUBSCRIPTIONS/{subscription ID}/RESOURCEGROUPS/{resource group name}/PROVIDERS/{resource provider name}/{resource type}/{resource name}/y={four-digit numeric year}/m={two-digit numeric month}/d={two-digit numeric day}/h={two-digit 24-hour clock hour}/m=00/PT1H.json
```

Por ejemplo, el blob de un grupo de seguridad de red puede tener un nombre similar al siguiente:

```
insights-logs-networksecuritygrouprulecounter/resourceId=/SUBSCRIPTIONS/00000000-0000-0000-0000-000000000000/RESOURCEGROUPS/TESTRESOURCEGROUP/PROVIDERS/MICROSOFT.NETWORK/NETWORKSECURITYGROUP/TESTNSG/y=2016/m=08/d=22/h=18/m=00/PT1H.json
```

Cada blob PT1H.json contiene un blob JSON de eventos que se produjeron en la hora especificada en la dirección URL del blob (por ejemplo h=12). Durante la hora en cuestión, los eventos se anexan al archivo PT1H.json a medida que se producen. El valor en minutos (m = 00) siempre es 00, ya que los eventos del registro de recurso se dividen en blobs individuales por hora.

En el archivo PT1H.json, los eventos se almacenan con este formato; con un esquema general común, pero único para cada uno de los servicios de Azure, tal y como se describe en [Esquema de los registros de recurso](./resource-logs-schema.md).

``` JSON
{"time": "2016-07-01T00:00:37.2040000Z","systemId": "46cdbb41-cb9c-4f3d-a5b4-1d458d827ff1","category": "NetworkSecurityGroupRuleCounter","resourceId": "/SUBSCRIPTIONS/s1id1234-5679-0123-4567-890123456789/RESOURCEGROUPS/TESTRESOURCEGROUP/PROVIDERS/MICROSOFT.NETWORK/NETWORKSECURITYGROUPS/TESTNSG","operationName": "NetworkSecurityGroupCounters","properties": {"vnetResourceGuid": "{12345678-9012-3456-7890-123456789012}","subnetPrefix": "10.3.0.0/24","macAddress": "000123456789","ruleName": "/subscriptions/ s1id1234-5679-0123-4567-890123456789/resourceGroups/testresourcegroup/providers/Microsoft.Network/networkSecurityGroups/testnsg/securityRules/default-allow-rdp","direction": "In","type": "allow","matchedConnections": 1988}}
```

> [!NOTE]
> Los registros de plataforma se escriben en el almacenamiento de blobs en formato [JSON Lines](http://jsonlines.org/), donde cada evento es una línea y el carácter de nueva línea indica un evento nuevo. Este formato se implementó en noviembre de 2018. Antes de esta fecha, los registros se escribían en el almacenamiento de blobs como matriz JSON de registros, tal como se describe en [Preparación para el cambio de formato a los registros de plataforma de Azure Monitor archivados en una cuenta de almacenamiento](resource-logs-blob-format.md).


## <a name="next-steps"></a>Pasos siguientes

* [Más información sobre los registros de recurso](platform-logs-overview.md).
* [Creación de una configuración de diagnóstico para enviar registros de plataforma y métricas a diferentes destinos](diagnostic-settings.md).
