---
title: Supervisión de factorías de datos mediante Azure Monitor | Microsoft Docs
description: Aprenda a utilizar Azure Monitor para supervisar las canalizaciones de Azure Data Factory mediante la habilitación de registros de diagnóstico con la información de Data Factory.
services: data-factory
documentationcenter: ''
author: djpmsft
ms.author: daperlov
manager: jroth
ms.reviewer: maghan
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.date: 07/13/2020
ms.openlocfilehash: 6f0e688f3d483536e0d82186dd8e498cdadf97da
ms.sourcegitcommit: 97a0d868b9d36072ec5e872b3c77fa33b9ce7194
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/04/2020
ms.locfileid: "87563558"
---
# <a name="monitor-and-alert-data-factory-by-using-azure-monitor"></a>Alerta y supervisión de Data Factory mediante Azure Monitor

[!INCLUDE[appliesto-adf-xxx-md](includes/appliesto-adf-xxx-md.md)]

Las aplicaciones en la nube son complejas y tienen muchas partes en movimiento. Los supervisores proporcionan datos para garantizar que las aplicaciones permanecen en funcionamiento en un estado correcto. Los supervisores también ayudan a evitar posibles problemas y a solucionar los problemas anteriores. Puede usar datos de supervisión para extraer conclusiones detalladas sobre las aplicaciones. Este conocimiento le ayuda a mejorar el rendimiento y el mantenimiento de las aplicaciones. También le ayuda a automatizar acciones que, de otro modo, requieren intervención manual.

Azure Monitor ofrece métricas y registros de las infraestructuras a nivel básico para la mayoría de los servicios de Azure. Un recurso emite registros de diagnóstico de Azure que proporcionan datos exhaustivos y frecuentes acerca del funcionamiento de ese recurso. Azure Data Factory (ADF) puede escribir registros de diagnóstico en Azure Monitor. Si desea una demostración y una introducción de siete minutos de esta característica, vea el siguiente vídeo:

> [!VIDEO https://channel9.msdn.com/Shows/Azure-Friday/Monitor-Data-Factory-pipelines-using-Operations-Management-Suite-OMS/player]

Para más información, consulte [Introducción a Azure Monitor](https://docs.microsoft.com/azure/monitoring-and-diagnostics/monitoring-overview-azure-monitor).

## <a name="keeping-azure-data-factory-metrics-and-pipeline-run-data"></a>Mantenimiento de métricas de Azure Data Factory y datos de ejecución de la canalización

Data Factory solo almacena los datos de ejecución de canalización durante 45 días. Use Azure Monitor si desea conservar los datos durante más tiempo. Con Monitor, puede enrutar los registros de diagnóstico a varios destinos diferentes.

* **Storage Account** (Cuenta de almacenamiento): Guarde los registro de diagnóstico en una cuenta de almacenamiento para auditarlos o para inspeccionarlos manualmente. Puede usar la configuración de diagnóstico para especificar el tiempo de retención en días.
* **Centro de eventos**: Transmita los registros a Azure Event Hubs. Los registros se convierten en la entrada de un servicio del asociado o una solución de análisis personalizada como Power BI.
* **Log Analytics**: Analice los registros con Log Analytics. La integración de Data Factory con Azure Monitor es útil en los escenarios siguientes:
  * Quiere escribir consultas complejas en un amplio conjunto de métricas que se publican mediante Data Factory en Monitor. Puede crear alertas personalizadas sobre estas consultas mediante Monitor.
  * Quiere realizar la supervisión entre fábricas de datos. Puede enrutar datos desde varias factorías de datos a una única área de trabajo de Monitor.

Puede usar también una cuenta de almacenamiento o un espacio de nombres de centro de eventos que no esté en la misma suscripción que el recurso que emite los registros. El usuario que configura los ajustes debe tener el control de acceso basado en rol (RBAC) adecuado a ambas suscripciones.

## <a name="configure-diagnostic-settings-and-workspace"></a>Configuración de las opciones de diagnóstico y el área de trabajo

Cree o agregue la configuración de diagnóstico de su factoría de datos.

1. En el portal, vaya a Monitor. Seleccione **Configuración** > **Configuración de diagnóstico**.

1. Seleccione la factoría de datos para la que quiere establecer una configuración de diagnóstico.

1. Si no existe ninguna configuración en la factoría de datos seleccionada, se le pedirá que cree una. Seleccione **Activar diagnósticos**.

   ![Creación de una configuración de diagnóstico si no existe ninguna configuración](media/data-factory-monitor-oms/monitor-oms-image1.png)

   Si hay una configuración existente en la factoría de datos, verá una lista de configuraciones ya establecidas en la factoría de datos. Seleccione **Agregar configuración de diagnóstico**.

   ![Adición de una configuración de diagnóstico si existe ninguna configuración](media/data-factory-monitor-oms/add-diagnostic-setting.png)

1. Asigne un nombre de la configuración, seleccione **Enviar a Log Analytics** y, después, seleccione un área de trabajo en **Área de trabajo de Log Analytics**.

    * En el modo _Azure-Diagnostics_, los registros de diagnóstico fluyen a la tabla _AzureDiagnostics_.

    * En el modo _Resource-Specific_, los registros de diagnóstico de Azure Data Factory fluyen a las tablas siguientes:
      - _ADFActivityRun_
      - _ADFPipelineRun_
      - _ADFTriggerRun_
      - _ADFSSISIntegrationRuntimeLogs_
      - _ADFSSISPackageEventMessageContext_
      - _ADFSSISPackageEventMessages_
      - _ADFSSISPackageExecutableStatistics_
      - _ADFSSISPackageExecutionComponentPhases_
      - _ADFSSISPackageExecutionDataStatistics_

      Puede seleccionar varios registros pertinentes para las cargas de trabajo que se van a enviar a las tablas de Log Analytics. Por ejemplo, si no usa SQL Server Integration Services (SSIS), no es necesario seleccionar ningún registro de SSIS. Si desea registrar las operaciones de inicio, detención y mantenimiento de SSIS Integration Runtime (IR), puede seleccionar registros de SSIS IR. Si invoca ejecuciones de paquetes SSIS a través de T-SQL en SQL Server Management Studio (SSMS), el Agente SQL Server u otras herramientas designadas, puede seleccionar los registros de paquetes SSIS. Si invoca ejecuciones de paquetes SSIS mediante actividades Ejecutar paquete SSIS en las canalizaciones de ADF, puede seleccionar todos los registros.

    * Si selecciona _AllMetrics_, varias métricas de ADF estarán disponibles para que pueda supervisarlas o generar alertas sobre ellas, incluidas las métricas de ejecuciones de actividades, canalizaciones y desencadenadores de ADF, así como las operaciones de SSIS IR y las ejecuciones de paquetes SSIS.

   ![Asignación de un nombre a la configuración y selección de un área de trabajo de Log Analytics](media/data-factory-monitor-oms/monitor-oms-image2.png)

    > [!NOTE]
    > Como una tabla de registro de Azure no puede tener más de 500 columnas, se **recomienda encarecidamente** seleccionar el _modo específico de recursos_. Para más información, consulte las [limitaciones conocidas de Log Analytics](../azure-monitor/platform/resource-logs-collect-workspace.md#column-limit-in-azurediagnostics).

1. Seleccione **Guardar**.

Transcurridos unos instantes, el nuevo valor de configuración aparece en la lista de configuraciones para esta factoría de datos. Los registros de diagnóstico se transmiten en esa área de trabajo en cuanto se generan nuevos datos de evento. Pueden pasar hasta 15 minutos entre el momento en que se emite un evento y cuando aparece en Log Analytics.

## <a name="install-azure-data-factory-analytics-solution-from-azure-marketplace"></a>Instalación de la solución Azure Data Factory Analytics desde Azure Marketplace

Esta solución proporciona un resumen del estado general de Data Factory, con opciones para profundizar en los detalles y solucionar problemas de los patrones de comportamiento inesperados. Con las vistas enriquecidas y completas, puede obtener información sobre el procesamiento de claves, entre lo que se incluye:

* Un breve resumen de la canalización, la actividad y las ejecuciones de desencadenadores de factoría de datos
* Capacidad de profundizar en las ejecuciones de actividad de factoría de datos por tipo
* Resumen de los principales errores de actividad y canalización de factoría de datos

1. Vaya a **Azure Marketplace**, elija el filtro **Analytics** y busque **Azure Data Factory Analytics (versión preliminar)** .

   ![Vaya a "Azure Marketplace", escriba "Filtro de Analytics" y seleccione "Azure Data Factory Analytics (versión preliminar)".](media/data-factory-monitor-oms/monitor-oms-image3.png)

1. Detalles sobre **Azure Data Factory Analytics (versión preliminar)**

   ![Detalles sobre "Azure Data Factory Analytics (versión preliminar)"](media/data-factory-monitor-oms/monitor-oms-image4.png)

1. Seleccione **Crear** y, después, cree o seleccione el **Área de trabajo Log Analytics**.

   ![Creación de una solución](media/data-factory-monitor-oms/monitor-log-analytics-image-5.png)

### <a name="monitor-data-factory-metrics"></a>Supervisión de métricas de Data Factory

La instalación de esta solución crea un conjunto predeterminado de vistas dentro de la sección de libros del área de trabajo de Log Analytics elegida. Como resultado, se habilitan las siguientes métricas:

* Ejecuciones de ADF: 1) Ejecuciones de canalización por Data Factory
* Ejecuciones de ADF: 2) Ejecuciones de actividad por Data Factory
* Ejecuciones de ADF: 3) Ejecuciones de desencadenador por Data Factory
* Errores de ADF: 1) 10 errores de canalización principales por Data Factory
* Errores de ADF: 2) 10 ejecuciones de actividad principales por Data Factory
* Errores de ADF: 3) 10 errores de desencadenador principales por Data Factory
* Estadísticas de ADF: 1) Ejecuciones de actividad por tipo
* Estadísticas de ADF: 2) Ejecuciones de desencadenador por tipo
* Estadísticas de ADF: 3) Duración máxima de las ejecuciones de canalización

![Ventana con "Libros (vista previa)" y "AzureDataFactoryAnalytics" resaltados](media/data-factory-monitor-oms/monitor-oms-image6.png)

Puede visualizar las métricas anteriores, examinar las consultas detrás de estas métricas, editar las consultas, crear alertas y realizar otras acciones.

![Representación gráfica de las ejecuciones de canalización por factoría de datos"](media/data-factory-monitor-oms/monitor-oms-image8.png)

> [!NOTE]
> Azure Data Factory Analytics (versión preliminar) envía los registros de diagnóstico a tablas de destino _específicas del recurso_. Puede escribir consultas en las tablas siguientes: _ADFPipelineRun_, _ADFTriggerRun_ y _ADFActivityRun_.

## <a name="data-factory-metrics"></a>Métricas de Data Factory

Con Monitor, puede obtener visibilidad sobre el rendimiento y el estado de las cargas de trabajo de Azure. El tipo más importante de datos de Monitor es la métrica, que también se denomina contador de rendimiento. La mayoría de los recursos de Azure emiten métricas. Monitor proporciona varias maneras de configurar y consumir estas métricas para supervisar y solucionar problemas.

Estas son algunas de las métricas emitidas por Azure Data Factory, versión 2:

| **Métrica**                           | **Nombre de métrica para mostrar**                  | **Unidad** | **Tipo de agregación** | **Descripción**                |
|--------------------------------------|------------------------------------------|----------|----------------------|--------------------------------|
| ActivityCancelledRuns                 | Cancelled activity runs metrics (Métricas de ejecuciones de actividad canceladas)           | Count    | Total                | Número total de ejecuciones de actividad canceladas en un período de minutos. |
| ActivityFailedRuns                   | Métricas de ejecuciones de actividad erróneas             | Count    | Total                | El número total de ejecuciones de actividad en las que se produjo un error en período de minutos. |
| ActivitySucceededRuns                | Métricas de ejecución de actividad realizadas correctamente          | Count    | Total                | El número total de ejecuciones de actividad realizadas correctamente dentro de un período de minutos. |
| PipelineCancelledRuns                 | Cancelled pipeline runs metrics (Métricas de ejecuciones de canalización canceladas)           | Count    | Total                | Número total de ejecuciones de canalización canceladas en un período de minutos. |
| PipelineFailedRuns                   | Métricas de ejecuciones de canalización erróneas             | Count    | Total                | El número total de ejecuciones de canalización con error dentro de una período de minutos. |
| PipelineSucceededRuns                | Las métricas de ejecuciones de canalización se realizaron correctamente          | Count    | Total                | El número total de ejecuciones de canalización realizadas correctamente dentro de un período de minutos. |
| TriggerCancelledRuns                  | Cancelled trigger runs metrics (Métricas de ejecuciones de desencadenador canceladas)            | Count    | Total                | Número total de ejecuciones de desencadenador canceladas en un período de minutos. |
| TriggerFailedRuns                    | Métricas de ejecuciones de desencadenador erróneas              | Count    | Total                | El número total de ejecuciones de desencadenador en las que se produjo un error dentro de un período de minutos. |
| TriggerSucceededRuns                 | Métricas de ejecuciones de desencadenador realizadas correctamente           | Count    | Total                | El número total de ejecuciones de desencadenador realizadas correctamente dentro de un período de minutos. |
| SSISIntegrationRuntimeStartCancelled  | Métricas de inicio de SSIS Integration Runtime cancelado           | Count    | Total                | Número total de inicios de SSIS Integration Runtime cancelados dentro de un período de un minuto. |
| SSISIntegrationRuntimeStartFailed    | Métricas de inicio de SSIS Integration Runtime con errores             | Count    | Total                | El número total de inicios de SSIS Integration Runtime con errores dentro de un período de un minuto. |
| SSISIntegrationRuntimeStartSucceeded | Métricas de inicio de SSIS Integration Runtime correcto          | Count    | Total                | El número total de inicios de SSIS Integration Runtime correctos dentro de un período de un minuto. |
| SSISIntegrationRuntimeStopStuck      | Métricas de detención de SSIS Integration Runtime bloqueada               | Count    | Total                | El número total de detenciones de SSIS Integration Runtime bloqueadas dentro de un período de un minuto. |
| SSISIntegrationRuntimeStopSucceeded  | Métricas de detención de SSIS Integration Runtime correcta           | Count    | Total                | El número total de detenciones de SSIS IR correctas dentro de un período de un minuto. |
| SSISPackageExecutionCancelled         | Métricas de ejecuciones de paquetes SSIS canceladas  | Count    | Total                | Número total de ejecuciones de paquetes SSIS canceladas en un período de minutos. |
| SSISPackageExecutionFailed           | Métricas de ejecución de paquetes SSIS con errores    | Count    | Total                | El número total de ejecuciones de paquetes SSIS con errores en un período de minutos. |
| SSISPackageExecutionSucceeded        | Métricas de ejecución de paquetes SSIS correcta | Count    | Total                | El número total de ejecuciones de paquetes SSIS correctas en un período de minutos. |

Para acceder a las métricas, complete las instrucciones que aparecen en la [plataforma de datos de Azure Monitor](https://docs.microsoft.com/azure/monitoring-and-diagnostics/monitoring-overview-metrics).

> [!NOTE]
> Solo se emiten los eventos de las ejecuciones de actividades y canalizaciones desencadenadas y completadas. Las ejecuciones de depuración y en curso **no** se emiten. Por otro lado, se emiten eventos de **todas** las ejecuciones de paquetes SSIS, incluidas las que se completan y están en curso, independientemente de sus métodos de invocación. Por ejemplo, puede invocar las ejecuciones de paquetes en SQL Server Data Tools (SSDT) habilitado para Azure mediante T-SQL en SSMS, el Agente SQL Server u otras herramientas designadas, y como ejecuciones desencadenadas o de depuración de las actividades de ejecución de paquetes SSIS en canalizaciones de ADF.

## <a name="data-factory-alerts"></a>Alertas de Data Factory

Inicie sesión en Azure Portal y haga clic en **Monitor** > **Alertas** para crear alertas.

![Alertas en el menú del portal](media/monitor-using-azure-monitor/alerts_image3.png)

### <a name="create-alerts"></a>Creación de alertas

1. Haga clic en **+Nueva regla de alertas** para crear una nueva alerta.

    ![Nueva alerta de reglas](media/monitor-using-azure-monitor/alerts_image4.png)

1. Defina la condición de la alerta.

    > [!NOTE]
    > Asegúrese de seleccionar **Todo** en **Filtrar por tipo de recurso**.

    !["Definir condición de la alerta" > "Seleccionar destino", que abre el panel "Seleccionar un recurso" ](media/monitor-using-azure-monitor/alerts_image5.png)

    !["Definir condición de la alerta" > "Agregar criterios", que abre el panel "Configurar lógica de señal"](media/monitor-using-azure-monitor/alerts_image6.png)

    ![Panel "Configurar tipo de señal"](media/monitor-using-azure-monitor/alerts_image7.png)

1. Defina los detalles de la alerta.

    ![Detalles de alertas](media/monitor-using-azure-monitor/alerts_image8.png)

1. Defina el grupo de acciones.

    ![Creación de una regla, con "Nuevo grupo de acciones" resaltado](media/monitor-using-azure-monitor/alerts_image9.png)

    ![Creación de un nuevo grupo de acciones](media/monitor-using-azure-monitor/alerts_image10.png)

    ![Configuración del correo electrónico, SMS, inserciones y voz](media/monitor-using-azure-monitor/alerts_image11.png)

    ![Definición de un grupo de acciones](media/monitor-using-azure-monitor/alerts_image12.png)

## <a name="set-up-diagnostic-logs-via-the-azure-monitor-rest-api"></a>Configuración de registros de diagnóstico mediante la API de REST de Azure Monitor

### <a name="diagnostic-settings"></a>Configuración de diagnóstico

Use la configuración de diagnóstico para configurar los registros de diagnóstico de los recursos que no son de proceso. La configuración de un control de recursos tiene las siguientes características:

* Especifican dónde se envían los registros de diagnóstico. Entre los ejemplos se incluyen una cuenta de almacenamiento de Azure, un centro de eventos de Azure o registros de Monitor.
* Especifican las categorías de registro que se envían.
* Especifican cuánto tiempo se debe conservar cada categoría de registro en una cuenta de almacenamiento.
* Una retención de cero días significa que los registros se conservan de forma indefinida. De lo contrario, el valor puede ser cualquier número de días comprendido entre 1 y 2 147 483 647.
* Si se establecen directivas de retención, pero el almacenamiento de registros en una cuenta de almacenamiento está deshabilitado, las directivas de retención no surten ningún efecto. Por ejemplo, esta condición puede ocurrir cuando solo se seleccionan las opciones de registros de Event Hubs o de Monitor.
* Las directivas de retención se aplican por día. El límite entre los días se produce a la medianoche de la hora universal coordinada (UTC). Al final del día, se eliminan los registros de los días que superan la directiva de retención. Por ejemplo, si tuviera una directiva de retención de un día, se eliminarían los registros de anteayer al principio del día de hoy.

### <a name="enable-diagnostic-logs-via-the-azure-monitor-rest-api"></a>Habilitación de registros de diagnóstico mediante la API REST de Azure Monitor

#### <a name="create-or-update-a-diagnostics-setting-in-the-monitor-rest-api"></a>Creación o actualización de una configuración de diagnóstico en la API REST de Monitor

##### <a name="request"></a>Solicitud

```
PUT
https://management.azure.com/{resource-id}/providers/microsoft.insights/diagnosticSettings/service?api-version={api-version}
```

##### <a name="headers"></a>encabezados

* Reemplace `{api-version}` por `2016-09-01`.
* Reemplace `{resource-id}` por el identificador del recurso para el que quiere editar la configuración de diagnóstico. Para más información, consulte el artículo sobre el [uso de grupos de recursos para administrar los recursos de Azure](../azure-resource-manager/management/manage-resource-groups-portal.md).
* Establezca el encabezado `Content-Type` en `application/json`.
* Establezca el encabezado de autorización en un token web de JSON que se obtiene de Azure Active Directory (AAD). Par más información, consulte [Solicitudes de autenticación](../active-directory/develop/authentication-scenarios.md).

##### <a name="body"></a>Body

```json
{
    "properties": {
        "storageAccountId": "/subscriptions/<subID>/resourceGroups/<resourceGroupName>/providers/Microsoft.Storage/storageAccounts/<storageAccountName>",
        "serviceBusRuleId": "/subscriptions/<subID>/resourceGroups/<resourceGroupName>/providers/Microsoft.EventHub/namespaces/<eventHubName>/authorizationrules/RootManageSharedAccessKey",
        "workspaceId": "/subscriptions/<subID>/resourceGroups/<resourceGroupName>/providers/Microsoft.OperationalInsights/workspaces/<LogAnalyticsName>",
        "metrics": [
        ],
        "logs": [
                {
                    "category": "PipelineRuns",
                    "enabled": true,
                    "retentionPolicy": {
                        "enabled": false,
                        "days": 0
                    }
                },
                {
                    "category": "TriggerRuns",
                    "enabled": true,
                    "retentionPolicy": {
                        "enabled": false,
                        "days": 0
                    }
                },
                {
                    "category": "ActivityRuns",
                    "enabled": true,
                    "retentionPolicy": {
                        "enabled": false,
                        "days": 0
                    }
                }
            ]
    },
    "location": ""
}
```

| Propiedad | Tipo | Descripción |
| --- | --- | --- |
| **storageAccountId** |String | El identificador de recurso de la cuenta de almacenamiento en la que quiere enviar los registros de diagnóstico. |
| **serviceBusRuleId** |String | El identificador de regla de Service Bus para el espacio de nombres de Service Bus donde quiere tener los centros de eventos creados para la transmisión de los registros de diagnóstico. El identificador de la regla tiene el formato `{service bus resource ID}/authorizationrules/{key name}`.|
| **workspaceId** | Tipo complejo | Una matriz de intervalos de agregación de métricas y sus directivas de retención. El valor de esta propiedad está vacío. |
|**métricas**| Valores de parámetros de la ejecución de canalización que se pasan a la canalización invocada| Objeto JSON que asigna nombres de parámetro a los valores de argumento. |
| **logs**| Tipo complejo| El nombre de una categoría de registro de diagnóstico para un tipo de recurso. Para obtener la lista de categorías de registro de diagnóstico para un recurso, realice una operación de configuración de diagnóstico GET. |
| **category**| String| Una matriz de las categorías de registro y sus directivas de retención. |
| **timeGrain** | String | La granularidad de las métricas que se capturan en formato de duración ISO 8601. El valor de propiedad debe ser `PT1M`, que especifica un minuto. |
| **enabled**| Boolean | Especifica si la colección de la categoría de métrica o registro está habilitada para este recurso. |
| **retentionPolicy**| Tipo complejo| Describe la directiva de retención para una categoría de métrica o registro. Esta propiedad se usa solo para las cuentas de almacenamiento. |
|**days**| Int| El número de días para retener las métricas o registros. Si el valor de propiedad es 0, los registros se mantienen indefinidamente. Esta propiedad se usa solo para las cuentas de almacenamiento. |

##### <a name="response"></a>Response

200 CORRECTO.

```json
{
    "id": "/subscriptions/<subID>/resourcegroups/adf/providers/microsoft.datafactory/factories/shloadobetest2/providers/microsoft.insights/diagnosticSettings/service",
    "type": null,
    "name": "service",
    "location": null,
    "kind": null,
    "tags": null,
    "properties": {
        "storageAccountId": "/subscriptions/<subID>/resourceGroups/<resourceGroupName>//providers/Microsoft.Storage/storageAccounts/<storageAccountName>",
        "serviceBusRuleId": "/subscriptions/<subID>/resourceGroups/<resourceGroupName>//providers/Microsoft.EventHub/namespaces/<eventHubName>/authorizationrules/RootManageSharedAccessKey",
        "workspaceId": "/subscriptions/<subID>/resourceGroups/<resourceGroupName>//providers/Microsoft.OperationalInsights/workspaces/<LogAnalyticsName>",
        "eventHubAuthorizationRuleId": null,
        "eventHubName": null,
        "metrics": [],
        "logs": [
            {
                "category": "PipelineRuns",
                "enabled": true,
                "retentionPolicy": {
                    "enabled": false,
                    "days": 0
                }
            },
            {
                "category": "TriggerRuns",
                "enabled": true,
                "retentionPolicy": {
                    "enabled": false,
                    "days": 0
                }
            },
            {
                "category": "ActivityRuns",
                "enabled": true,
                "retentionPolicy": {
                    "enabled": false,
                    "days": 0
                }
            }
        ]
    },
    "identity": null
}
```

#### <a name="get-information-about-diagnostics-settings-in-the-monitor-rest-api"></a>Permite obtener información sobre la configuración de diagnóstico en la API REST de Monitor.

##### <a name="request"></a>Solicitud

```
GET
https://management.azure.com/{resource-id}/providers/microsoft.insights/diagnosticSettings/service?api-version={api-version}
```

##### <a name="headers"></a>encabezados

* Reemplace `{api-version}` por `2016-09-01`.
* Reemplace `{resource-id}` por el identificador del recurso para el que quiere editar la configuración de diagnóstico. Para más información, consulte el artículo sobre el [uso de grupos de recursos para administrar los recursos de Azure](../azure-resource-manager/management/manage-resource-groups-portal.md).
* Establezca el encabezado `Content-Type` en `application/json`.
* Establezca el encabezado de autorización en un token web de JSON que obtuvo de Azure AD. Par más información, consulte [Solicitudes de autenticación](../active-directory/develop/authentication-scenarios.md).

##### <a name="response"></a>Response

200 CORRECTO.

```json
{
    "id": "/subscriptions/<subID>/resourcegroups/adf/providers/microsoft.datafactory/factories/shloadobetest2/providers/microsoft.insights/diagnosticSettings/service",
    "type": null,
    "name": "service",
    "location": null,
    "kind": null,
    "tags": null,
    "properties": {
        "storageAccountId": "/subscriptions/<subID>/resourceGroups/shloprivate/providers/Microsoft.Storage/storageAccounts/azmonlogs",
        "serviceBusRuleId": "/subscriptions/<subID>/resourceGroups/shloprivate/providers/Microsoft.EventHub/namespaces/shloeventhub/authorizationrules/RootManageSharedAccessKey",
        "workspaceId": "/subscriptions/<subID>/resourceGroups/ADF/providers/Microsoft.OperationalInsights/workspaces/mihaipie",
        "eventHubAuthorizationRuleId": null,
        "eventHubName": null,
        "metrics": [],
        "logs": [
            {
                "category": "PipelineRuns",
                "enabled": true,
                "retentionPolicy": {
                    "enabled": false,
                    "days": 0
                }
            },
            {
                "category": "TriggerRuns",
                "enabled": true,
                "retentionPolicy": {
                    "enabled": false,
                    "days": 0
                }
            },
            {
                "category": "ActivityRuns",
                "enabled": true,
                "retentionPolicy": {
                    "enabled": false,
                    "days": 0
                }
            }
        ]
    },
    "identity": null
}
```
Para más información, consulte [Configuración de diagnóstico](https://docs.microsoft.com/rest/api/monitor/diagnosticsettings).

## <a name="schema-of-logs-and-events"></a>Esquema de registros y eventos

### <a name="monitor-schema"></a>Esquema de Monitor

#### <a name="activity-run-log-attributes"></a>Atributos de registro de la ejecución de actividad

```json
{
   "Level": "",
   "correlationId":"",
   "time":"",
   "activityRunId":"",
   "pipelineRunId":"",
   "resourceId":"",
   "category":"ActivityRuns",
   "level":"Informational",
   "operationName":"",
   "pipelineName":"",
   "activityName":"",
   "start":"",
   "end":"",
   "properties":
       {
          "Input": "{
              "source": {
                "type": "BlobSource"
              },
              "sink": {
                "type": "BlobSink"
              }
           }",
          "Output": "{"dataRead":121,"dataWritten":121,"copyDuration":5,
               "throughput":0.0236328132,"errors":[]}",
          "Error": "{
              "errorCode": "null",
              "message": "null",
              "failureType": "null",
              "target": "CopyBlobtoBlob"
        }
    }
}
```

| Propiedad | Tipo | Descripción | Ejemplo |
| --- | --- | --- | --- |
| **Level** |String | El nivel de los registros de diagnóstico. En el caso de los registros de ejecución de actividad, establezca el valor de la propiedad en 4. | `4` |
| **correlationId** |String | El identificador único para realizar el seguimiento de una solicitud determinada. | `319dc6b4-f348-405e-b8d7-aafc77b73e77` |
| **time** | String | La hora del evento en formato UTC de intervalo de tiempo`YYYY-MM-DDTHH:MM:SS.00000Z`. | `2017-06-28T21:00:27.3534352Z` |
|**activityRunId**| String| El identificador de la ejecución de actividad. | `3a171e1f-b36e-4b80-8a54-5625394f4354` |
|**pipelineRunId**| String| El identificador de la ejecución de canalización. | `9f6069d6-e522-4608-9f99-21807bfc3c70` |
|**resourceId**| String | El identificador asociado con el recurso de la fábrica de datos. | `/SUBSCRIPTIONS/<subID>/RESOURCEGROUPS/<resourceGroupName>/PROVIDERS/MICROSOFT.DATAFACTORY/FACTORIES/<dataFactoryName>` |
|**category**| String | La categoría de los registros de diagnóstico. Establezca el valor de propiedad en `ActivityRuns`. | `ActivityRuns` |
|**level**| String | El nivel de los registros de diagnóstico. Establezca el valor de propiedad en `Informational`. | `Informational` |
|**operationName**| String | El nombre de la actividad con su estado. Si la actividad es el latido de inicio, el valor de propiedad es `MyActivity -`. Si la actividad es el latido final, el valor de propiedad es `MyActivity - Succeeded`. | `MyActivity - Succeeded` |
|**pipelineName**| String | El nombre de la canalización. | `MyPipeline` |
|**activityName**| String | El nombre de la actividad. | `MyActivity` |
|**start**| String | La hora de inicio de las ejecuciones de actividad en formato UTC del intervalo de tiempo. | `2017-06-26T20:55:29.5007959Z`|
|**end**| String | La hora de finalización de las ejecuciones de actividad en formato UTC del intervalo de tiempo. Si el registro de diagnóstico muestra que una actividad se ha iniciado pero aún no ha finalizado, el valor de propiedad es `1601-01-01T00:00:00Z`. | `2017-06-26T20:55:29.5007959Z` |

#### <a name="pipeline-run-log-attributes"></a>Atributos de registro de la ejecución de la canalización

```json
{
   "Level": "",
   "correlationId":"",
   "time":"",
   "runId":"",
   "resourceId":"",
   "category":"PipelineRuns",
   "level":"Informational",
   "operationName":"",
   "pipelineName":"",
   "start":"",
   "end":"",
   "status":"",
   "properties":
    {
      "Parameters": {
        "<parameter1Name>": "<parameter1Value>"
      },
      "SystemParameters": {
        "ExecutionStart": "",
        "TriggerId": "",
        "SubscriptionId": ""
      }
    }
}
```

| Propiedad | Tipo | Descripción | Ejemplo |
| --- | --- | --- | --- |
| **Level** |String | El nivel de los registros de diagnóstico. En el caso de los registros de ejecución de actividad, establezca el valor de la propiedad en 4. | `4` |
| **correlationId** |String | El identificador único para realizar el seguimiento de una solicitud determinada. | `319dc6b4-f348-405e-b8d7-aafc77b73e77` |
| **time** | String | La hora del evento en formato UTC de intervalo de tiempo`YYYY-MM-DDTHH:MM:SS.00000Z`. | `2017-06-28T21:00:27.3534352Z` |
|**runId**| String| El identificador de la ejecución de canalización. | `9f6069d6-e522-4608-9f99-21807bfc3c70` |
|**resourceId**| String | El identificador asociado con el recurso de la fábrica de datos. | `/SUBSCRIPTIONS/<subID>/RESOURCEGROUPS/<resourceGroupName>/PROVIDERS/MICROSOFT.DATAFACTORY/FACTORIES/<dataFactoryName>` |
|**category**| String | La categoría de los registros de diagnóstico. Establezca el valor de propiedad en `PipelineRuns`. | `PipelineRuns` |
|**level**| String | El nivel de los registros de diagnóstico. Establezca el valor de propiedad en `Informational`. | `Informational` |
|**operationName**| String | El nombre de la canalización junto con su estado. Una vez finalizada la ejecución de la canalización, el valor de propiedad es `Pipeline - Succeeded`. | `MyPipeline - Succeeded`. |
|**pipelineName**| String | El nombre de la canalización. | `MyPipeline` |
|**start**| String | La hora de inicio de las ejecuciones de actividad en formato UTC del intervalo de tiempo. | `2017-06-26T20:55:29.5007959Z`. |
|**end**| String | La hora de finalización de las ejecuciones de actividad en formato UTC del intervalo de tiempo. Si el registro de diagnóstico muestra que una actividad se ha iniciado pero aún no ha finalizado, el valor de propiedad es `1601-01-01T00:00:00Z`.  | `2017-06-26T20:55:29.5007959Z` |
|**status**| String | El estado final de la ejecución de canalización. Los valores de propiedad posibles son `Succeeded` y `Failed`. | `Succeeded`|

#### <a name="trigger-run-log-attributes"></a>Atributos de registro de la ejecución del desencadenador

```json
{
   "Level": "",
   "correlationId":"",
   "time":"",
   "triggerId":"",
   "resourceId":"",
   "category":"TriggerRuns",
   "level":"Informational",
   "operationName":"",
   "triggerName":"",
   "triggerType":"",
   "triggerEvent":"",
   "start":"",
   "status":"",
   "properties":
   {
      "Parameters": {
        "TriggerTime": "",
       "ScheduleTime": ""
      },
      "SystemParameters": {}
    }
}
```

| Propiedad | Tipo | Descripción | Ejemplo |
| --- | --- | --- | --- |
| **Level** |String | El nivel de los registros de diagnóstico. En el caso de los registros de ejecución de actividad, establezca el valor de la propiedad en 4. | `4` |
| **correlationId** |String | El identificador único para realizar el seguimiento de una solicitud determinada. | `319dc6b4-f348-405e-b8d7-aafc77b73e77` |
| **time** | String | La hora del evento en formato UTC de intervalo de tiempo`YYYY-MM-DDTHH:MM:SS.00000Z`. | `2017-06-28T21:00:27.3534352Z` |
|**triggerId**| String| El identificador de la ejecución del desencadenador. | `08587023010602533858661257311` |
|**resourceId**| String | El identificador asociado con el recurso de la fábrica de datos. | `/SUBSCRIPTIONS/<subID>/RESOURCEGROUPS/<resourceGroupName>/PROVIDERS/MICROSOFT.DATAFACTORY/FACTORIES/<dataFactoryName>` |
|**category**| String | La categoría de los registros de diagnóstico. Establezca el valor de propiedad en `PipelineRuns`. | `PipelineRuns` |
|**level**| String | El nivel de los registros de diagnóstico. Establezca el valor de propiedad en `Informational`. | `Informational` |
|**operationName**| String | El nombre del desencadenador con su estado final, que indica si el desencadenador se activó correctamente. Si el latido se realizó correctamente, el valor de la propiedad es `MyTrigger - Succeeded`. | `MyTrigger - Succeeded` |
|**triggerName**| String | El nombre del desencadenador. | `MyTrigger` |
|**triggerType**| String | El tipo del desencadenador. Los valores de propiedad posibles son `Manual Trigger` y `Schedule Trigger`. | `ScheduleTrigger` |
|**triggerEvent**| String | El evento del desencadenador. | `ScheduleTime - 2017-07-06T01:50:25Z` |
|**start**| String | La hora de inicio del desencadenador que se activa en formato UTC del intervalo de tiempo. | `2017-06-26T20:55:29.5007959Z`|
|**status**| String | El estado final que indica si el desencadenador se activó correctamente. Los valores de propiedad posibles son `Succeeded` y `Failed`. | `Succeeded`|

#### <a name="ssis-integration-runtime-log-attributes"></a>Atributos de los registros de SSIS Integration Runtime

Estos son los atributos de registro de las operaciones de inicio, detención y mantenimiento de SSIS IR.

```json
{
   "time": "",
   "operationName": "",
   "category": "",
   "correlationId": "",
   "dataFactoryName": "",
   "integrationRuntimeName": "",
   "level": "",
   "resultType": "",
   "properties": {
      "message": ""
   },
   "resourceId": ""
}
```

| Propiedad                   | Tipo   | Descripción                                                   | Ejemplo                        |
| -------------------------- | ------ | ------------------------------------------------------------- | ------------------------------ |
| **time**                   | String | Hora del evento en formato UTC: `YYYY-MM-DDTHH:MM:SS.00000Z` | `2017-06-28T21:00:27.3534352Z` |
| **operationName**          | String | Nombre de la operación de SSIS IR                            | `Start/Stop/Maintenance` |
| **category**               | String | Categoría de los registros de diagnóstico                               | `SSISIntegrationRuntimeLogs` |
| **correlationId**          | String | Id. exclusivo para realizar el seguimiento de una operación determinada             | `f13b159b-515f-4885-9dfa-a664e949f785Deprovision0059035558` |
| **dataFactoryName**        | String | Nombre de la instancia de ADF                                          | `MyADFv2` |
| **integrationRuntimeName** | String | Nombre de la instancia de SSIS IR                                      | `MySSISIR` |
| **level**                  | String | Nivel de los registros de diagnóstico                                  | `Informational` |
| **resultType**             | String | Resultado de la operación de SSIS IR                          | `Started/InProgress/Succeeded/Failed` |
| **message**                | String | Mensaje de salida de la operación de SSIS IR                  | `The stopping of your SSIS integration runtime has succeeded.` |
| **resourceId**             | String | Id. exclusivo del recurso de ADF                            | `/SUBSCRIPTIONS/<subscriptionID>/RESOURCEGROUPS/<resourceGroupName>/PROVIDERS/MICROSOFT.DATAFACTORY/FACTORIES/<dataFactoryName>` |

#### <a name="ssis-event-message-context-log-attributes"></a>Atributos de los registros contextuales de los mensajes de eventos de SSIS

Estos son los atributos de registro de las condiciones relacionadas con los mensajes de eventos generados por las ejecuciones de paquetes SSIS en la instancia de SSIS IR. Transmiten información similar a la de la [tabla o vista del contexto de mensajes de eventos del catálogo de SSIS (SSISDB)](https://docs.microsoft.com/sql/integration-services/system-views/catalog-event-message-context?view=sql-server-ver15) que muestra los valores de tiempo de ejecución de muchas propiedades de los paquetes SSIS. Se generan al seleccionar el nivel de registro `Basic/Verbose` y resultan útiles para la depuración o la comprobación de cumplimiento.

```json
{
   "time": "",
   "operationName": "",
   "category": "",
   "correlationId": "",
   "dataFactoryName": "",
   "integrationRuntimeName": "",
   "level": "",
   "properties": {
      "operationId": "",
      "contextDepth": "",
      "packagePath": "",
      "contextType": "",
      "contextSourceName": "",
      "contextSourceId": "",
      "propertyName": "",
      "propertyValue": ""
   },
   "resourceId": ""
}
```

| Propiedad                   | Tipo   | Descripción                                                          | Ejemplo                        |
| -------------------------- | ------ | -------------------------------------------------------------------- | ------------------------------ |
| **time**                   | String | Hora del evento en formato UTC: `YYYY-MM-DDTHH:MM:SS.00000Z`        | `2017-06-28T21:00:27.3534352Z` |
| **operationName**          | String | Se establece en `YourSSISIRName-SSISPackageEventMessageContext`       | `mysqlmissisir-SSISPackageEventMessageContext` |
| **category**               | String | Categoría de los registros de diagnóstico                                      | `SSISPackageEventMessageContext` |
| **correlationId**          | String | Id. exclusivo para realizar el seguimiento de una operación determinada                    | `e55700df-4caf-4e7c-bfb8-78ac7d2f28a0` |
| **dataFactoryName**        | String | Nombre de la instancia de ADF                                                 | `MyADFv2` |
| **integrationRuntimeName** | String | Nombre de la instancia de SSIS IR                                             | `MySSISIR` |
| **level**                  | String | Nivel de los registros de diagnóstico                                         | `Informational` |
| **operationId**            | String | Id. exclusivo para realizar el seguimiento de una operación determinada en SSISDB          | `1` (1 significa operaciones relacionadas con paquetes que **no** están almacenados en SSISDB o no se invocan mediante T-SQL) |
| **contextDepth**           | String | Profundidad del contexto del mensaje del evento                              | `0` (0 significa el contexto antes de que se inicie la ejecución del paquete y 1 significa el contexto cuando se produce un error, que aumenta a medida que el contexto va más allá del error) |
| **packagePath**            | String | Ruta de acceso del objeto de paquete como origen del contexto del mensaje del evento      | `\Package` |
| **contextType**            | String | Tipo de objeto de paquete como origen del contexto del mensaje del evento      | `60`(ver [más tipos de contexto](https://docs.microsoft.com/sql/integration-services/system-views/catalog-event-message-context?view=sql-server-ver15#remarks)) |
| **contextSourceName**      | String | Nombre del objeto de paquete como origen del contexto del mensaje del evento      | `MyPackage` |
| **contextSourceId**        | String | Id. exclusivo del objeto de paquete como origen del contexto del mensaje del evento | `{E2CF27FB-EA48-41E9-AF6F-3FE938B4ADE1}` |
| **propertyName**           | String | Nombre de la propiedad del paquete para el origen del contexto del mensaje del evento   | `DelayValidation` |
| **propertyValue**          | String | Valor de la propiedad del paquete para el origen del contexto del mensaje del evento  | `False` |
| **resourceId**             | String | Id. exclusivo del recurso de ADF                                   | `/SUBSCRIPTIONS/<subscriptionID>/RESOURCEGROUPS/<resourceGroupName>/PROVIDERS/MICROSOFT.DATAFACTORY/FACTORIES/<dataFactoryName>` |

#### <a name="ssis-event-messages-log-attributes"></a>Atributos de los registros de los mensajes de eventos de SSIS

Estos son los atributos de registro de los mensajes de eventos que generan las ejecuciones de paquetes SSIS en la instancia de SSIS IR. Transmiten información similar a la de la [tabla o vista de mensajes de eventos de SSISDB](https://docs.microsoft.com/sql/integration-services/system-views/catalog-event-messages?view=sql-server-ver15) que muestra los metadatos o el texto detallado de los mensajes de los eventos. Se generan en cualquier nivel de registro, excepto en `None`.

```json
{
   "time": "",
   "operationName": "",
   "category": "",
   "correlationId": "",
   "dataFactoryName": "",
   "integrationRuntimeName": "",
   "level": "",
   "properties": {
      "operationId": "",
      "messageTime": "",
      "messageType": "",
      "messageSourceType": "",
      "message": "",
      "packageName": "",
      "eventName": "",
      "messageSourceName": "",
      "messageSourceId": "",
      "subcomponentName": "",
      "packagePath": "",
      "executionPath": "",
      "threadId": ""
   }
}
```

| Propiedad                   | Tipo   | Descripción                                                        | Ejemplo                        |
| -------------------------- | ------ | ------------------------------------------------------------------ | ------------------------------ |
| **time**                   | String | Hora del evento en formato UTC: `YYYY-MM-DDTHH:MM:SS.00000Z`      | `2017-06-28T21:00:27.3534352Z` |
| **operationName**          | String | Se establece en `YourSSISIRName-SSISPackageEventMessages`           | `mysqlmissisir-SSISPackageEventMessages` |
| **category**               | String | Categoría de los registros de diagnóstico                                    | `SSISPackageEventMessages` |
| **correlationId**          | String | Id. exclusivo para realizar el seguimiento de una operación determinada                  | `e55700df-4caf-4e7c-bfb8-78ac7d2f28a0` |
| **dataFactoryName**        | String | Nombre de la instancia de ADF                                               | `MyADFv2` |
| **integrationRuntimeName** | String | Nombre de la instancia de SSIS IR                                           | `MySSISIR` |
| **level**                  | String | Nivel de los registros de diagnóstico                                       | `Informational` |
| **operationId**            | String | Id. exclusivo para realizar el seguimiento de una operación determinada en SSISDB        | `1` (1 significa operaciones relacionadas con paquetes que **no** están almacenados en SSISDB o no se invocan mediante T-SQL) |
| **messageTime**            | String | Hora a la que se crea el mensaje del evento en formato UTC          | `2017-06-28T21:00:27.3534352Z` |
| **messageType**            | String | Tipo de mensaje de evento                                     | `70`(ver [más tipos de mensajes](https://docs.microsoft.com/sql/integration-services/system-views/catalog-operation-messages-ssisdb-database?view=sql-server-ver15#remarks)) |
| **messageSourceType**      | String | Tipo de origen del mensaje del evento                              | `20`(ver [más tipos de orígenes de mensajes](https://docs.microsoft.com/sql/integration-services/system-views/catalog-operation-messages-ssisdb-database?view=sql-server-ver15#remarks)) |
| **message**                | String | Texto del mensaje del evento                                     | `MyPackage:Validation has started.` |
| **packageName**            | String | Nombre del archivo del paquete ejecutado                             | `MyPackage.dtsx` |
| **eventName**              | String | Nombre del evento de tiempo de ejecución relacionado                                 | `OnPreValidate` |
| **messageSourceName**      | String | Nombre del componente del paquete como origen del mensaje del evento         | `Data Flow Task` |
| **messageSourceId**        | String | Id. exclusivo del componente del paquete como origen del mensaje del evento    | `{1a45a5a4-3df9-4f02-b818-ebf583829ad2}    ` |
| **subcomponentName**       | String | Nombre del componente del flujo de datos como el origen del mensaje del evento       | `SSIS.Pipeline` |
| **packagePath**            | String | Ruta de acceso del objeto de paquete como origen del mensaje del evento            | `\Package\Data Flow Task` |
| **executionPath**          | String | Ruta de acceso completa del paquete primario para el componente ejecutado            | `\Transformation\Data Flow Task` (Esta ruta también captura las iteraciones de un componente) |
| **threadId**               | String | Id. exclusivo del subproceso ejecutado cuando se registra el mensaje del evento | `{1a45a5a4-3df9-4f02-b818-ebf583829ad2}    ` |

#### <a name="ssis-executable-statistics-log-attributes"></a>Atributos del registro de estadísticas de archivos ejecutables de SSIS

Estos son los atributos de registro de las estadísticas de los archivos ejecutables que generan las ejecuciones de paquetes SSIS en la instancia de SSIS IR, en la que los archivos ejecutables son contenedores o tareas de los flujos de control de paquetes. Transmiten información similar a la de la [tabla o vista de estadísticas de archivos ejecutables de SSISDB](https://docs.microsoft.com/sql/integration-services/system-views/catalog-executable-statistics?view=sql-server-ver15) que muestra una fila para cada archivo ejecutable en ejecución, incluidas sus iteraciones. Se generan en cualquier nivel de registro, excepto `None`, y resultan útiles para identificar los errores o cuellos de botella a nivel de tarea.

```json
{
   "time": "",
   "operationName": "",
   "category": "",
   "correlationId": "",
   "dataFactoryName": "",
   "integrationRuntimeName": "",
   "level": "",
   "properties": {
      "executionId": "",
      "executionPath": "",
      "startTime": "",
      "endTime": "",
      "executionDuration": "",
      "executionResult": "",
      "executionValue": ""
   },
   "resourceId": ""
}
```

| Propiedad                   | Tipo   | Descripción                                                      | Ejemplo                        |
| -------------------------- | ------ | ---------------------------------------------------------------- | ------------------------------ |
| **time**                   | String | Hora del evento en formato UTC: `YYYY-MM-DDTHH:MM:SS.00000Z`    | `2017-06-28T21:00:27.3534352Z` |
| **operationName**          | String | Se establece en `YourSSISIRName-SSISPackageExecutableStatistics`  | `mysqlmissisir-SSISPackageExecutableStatistics` |
| **category**               | String | Categoría de los registros de diagnóstico                                  | `SSISPackageExecutableStatistics` |
| **correlationId**          | String | Id. exclusivo para realizar el seguimiento de una operación determinada                | `e55700df-4caf-4e7c-bfb8-78ac7d2f28a0` |
| **dataFactoryName**        | String | Nombre de la instancia de ADF                                             | `MyADFv2` |
| **integrationRuntimeName** | String | Nombre de la instancia de SSIS IR                                         | `MySSISIR` |
| **level**                  | String | Nivel de los registros de diagnóstico                                     | `Informational` |
| **executionId**            | String | Id. exclusivo para realizar el seguimiento de una ejecución determinada en SSISDB      | `1` (1 significa ejecuciones relacionadas con paquetes que **no** están almacenados en SSISDB o no se invocan mediante T-SQL) |
| **executionPath**          | String | Ruta de acceso completa del paquete primario para el componente ejecutado          | `\Transformation\Data Flow Task` (Esta ruta también captura las iteraciones de un componente) |
| **startTime**              | String | Hora en que el archivo ejecutable entra en la fase previa a la ejecución en formato UTC  | `2017-06-28T21:00:27.3534352Z` |
| **endTime**                | String | Hora en que el archivo ejecutable entra en la fase posterior a la ejecución en formato UTC | `2017-06-28T21:00:27.3534352Z` |
| **executionDuration**      | String | Tiempo de ejecución del archivo ejecutable en milisegundos                   | `1,125` |
| **executionResult**        | String | Resultado de la ejecución del archivo ejecutable                                 | `0` (0 significa que se ha realizado correctamente, 1 significa que se ha producido un error, 2 significa finalización y 3 significa cancelación) |
| **executionValue**         | String | Valor definido por el usuario devuelto por la ejecución del archivo ejecutable            | `1` |
| **resourceId**             | String | Id. exclusivo del recurso de ADF                               | `/SUBSCRIPTIONS/<subscriptionID>/RESOURCEGROUPS/<resourceGroupName>/PROVIDERS/MICROSOFT.DATAFACTORY/FACTORIES/<dataFactoryName>` |

#### <a name="ssis-execution-component-phases-log-attributes"></a>Atributos del registro de fases de los componentes de ejecución de SSIS

Estos son los atributos de registro de las estadísticas de tiempo de ejecución para los componentes de flujo de datos que generan las ejecuciones de paquetes SSIS en la instancia de SSIS IR. Transmiten información similar a la de la [tabla o vista de las fases de los componentes de ejecución de SSISDB](https://docs.microsoft.com/sql/integration-services/system-views/catalog-execution-component-phases?view=sql-server-ver15) que muestra el tiempo empleado por los componentes de flujo de datos en todas sus fases de ejecución. Se generan al seleccionar el nivel de registro `Performance/Verbose` y resultan útiles para capturar las estadísticas de ejecución del flujo de datos.

```json
{
   "time": "",
   "operationName": "",
   "category": "",
   "correlationId": "",
   "dataFactoryName": "",
   "integrationRuntimeName": "",
   "level": "",
   "properties": {
      "executionId": "",
      "packageName": "",
      "taskName": "",
      "subcomponentName": "",
      "phase": "",
      "startTime": "",
      "endTime": "",
      "executionPath": ""
   },
   "resourceId": ""
}
```

| Propiedad                   | Tipo   | Descripción                                                         | Ejemplo                        |
| -------------------------- | ------ | ------------------------------------------------------------------- | ------------------------------ |
| **time**                   | String | Hora del evento en formato UTC: `YYYY-MM-DDTHH:MM:SS.00000Z`       | `2017-06-28T21:00:27.3534352Z` |
| **operationName**          | String | Se establece en `YourSSISIRName-SSISPackageExecutionComponentPhases` | `mysqlmissisir-SSISPackageExecutionComponentPhases` |
| **category**               | String | Categoría de los registros de diagnóstico                                     | `SSISPackageExecutionComponentPhases` |
| **correlationId**          | String | Id. exclusivo para realizar el seguimiento de una operación determinada                   | `e55700df-4caf-4e7c-bfb8-78ac7d2f28a0` |
| **dataFactoryName**        | String | Nombre de la instancia de ADF                                                | `MyADFv2` |
| **integrationRuntimeName** | String | Nombre de la instancia de SSIS IR                                            | `MySSISIR` |
| **level**                  | String | Nivel de los registros de diagnóstico                                        | `Informational` |
| **executionId**            | String | Id. exclusivo para realizar el seguimiento de una ejecución determinada en SSISDB         | `1` (1 significa ejecuciones relacionadas con paquetes que **no** están almacenados en SSISDB o no se invocan mediante T-SQL) |
| **packageName**            | String | Nombre del archivo del paquete ejecutado                              | `MyPackage.dtsx` |
| **taskName**               | String | Nombre de la tarea Flujo de datos ejecutada                                 | `Data Flow Task` |
| **subcomponentName**       | String | Nombre del componente de flujo de datos                                     | `Derived Column` |
| **phase**                  | String | Nombre de la fase de ejecución                                         | `AcquireConnections` |
| **startTime**              | String | Hora a la que se inicia la fase de ejecución en formato UTC                  | `2017-06-28T21:00:27.3534352Z` |
| **endTime**                | String | Hora de finalización de la fase de ejecución en formato UTC                    | `2017-06-28T21:00:27.3534352Z` |
| **executionPath**          | String | Ruta de ejecución de la tarea Flujo de datos                            | `\Transformation\Data Flow Task` |
| **resourceId**             | String | Id. exclusivo del recurso de ADF                                  | `/SUBSCRIPTIONS/<subscriptionID>/RESOURCEGROUPS/<resourceGroupName>/PROVIDERS/MICROSOFT.DATAFACTORY/FACTORIES/<dataFactoryName>` |

#### <a name="ssis-execution-data-statistics-log-attributes"></a>Atributos del registro de estadísticas de los datos de ejecución de SSIS

Estos son los atributos de registro de los movimientos de datos a través de cada una de las canalizaciones de flujo de datos, desde los componentes ascendentes hasta los descendentes, que generan las ejecuciones de paquetes SSIS en la instancia de SSIS IR. Transmiten información similar a la de la [tabla o vista de estadísticas de datos de ejecución de SSISDB](https://docs.microsoft.com/sql/integration-services/system-views/catalog-execution-data-statistics?view=sql-server-ver15) que muestra los recuentos de filas de los datos que se mueven a través de las tareas Flujo de datos. Se generan al seleccionar el nivel de registro `Verbose` y resultan útiles para calcular el rendimiento del flujo de datos.

```json
{
   "time": "",
   "operationName": "",
   "category": "",
   "correlationId": "",
   "dataFactoryName": "",
   "integrationRuntimeName": "",
   "level": "",
   "properties": {
      "executionId": "",
      "packageName": "",
      "taskName": "",
      "dataflowPathIdString": "",
      "dataflowPathName": "",
      "sourceComponentName": "",
      "destinationComponentName": "",
      "rowsSent": "",
      "createdTime": "",
      "executionPath": ""
   },
   "resourceId": ""
}
```

| Propiedad                     | Tipo   | Descripción                                                        | Ejemplo                        |
| ---------------------------- | ------ | ------------------------------------------------------------------ | ------------------------------ |
| **time**                     | String | Hora del evento en formato UTC: `YYYY-MM-DDTHH:MM:SS.00000Z`      | `2017-06-28T21:00:27.3534352Z` |
| **operationName**            | String | Se establece en `YourSSISIRName-SSISPackageExecutionDataStatistics` | `mysqlmissisir-SSISPackageExecutionDataStatistics` |
| **category**                 | String | Categoría de los registros de diagnóstico                                    | `SSISPackageExecutionDataStatistics` |
| **correlationId**            | String | Id. exclusivo para realizar el seguimiento de una operación determinada                  | `e55700df-4caf-4e7c-bfb8-78ac7d2f28a0` |
| **dataFactoryName**          | String | Nombre de la instancia de ADF                                               | `MyADFv2` |
| **integrationRuntimeName**   | String | Nombre de la instancia de SSIS IR                                           | `MySSISIR` |
| **level**                    | String | Nivel de los registros de diagnóstico                                       | `Informational` |
| **executionId**              | String | Id. exclusivo para realizar el seguimiento de una ejecución determinada en SSISDB        | `1` (1 significa ejecuciones relacionadas con paquetes que **no** están almacenados en SSISDB o no se invocan mediante T-SQL) |
| **packageName**              | String | Nombre del archivo del paquete ejecutado                             | `MyPackage.dtsx` |
| **taskName**                 | String | Nombre de la tarea Flujo de datos ejecutada                                | `Data Flow Task` |
| **dataflowPathIdString**     | String | Id. exclusivo para el seguimiento de la ruta del flujo de datos                          | `Paths[SQLDB Table3.ADO NET Source Output]` |
| **dataflowPathName**         | String | Nombre de la ruta del flujo de datos                                         | `ADO NET Source Output` |
| **sourceComponentName**      | String | Nombre del componente de flujo de datos que envía los datos                    | `SQLDB Table3` |
| **destinationComponentName** | String | Nombre del componente de flujo de datos que recibe los datos                 | `Derived Column` |
| **rowsSent**                 | String | Número de filas enviadas por el componente de origen                        | `500` |
| **createdTime**              | String | Hora a la que se obtienen los valores de fila en formato UTC                | `2017-06-28T21:00:27.3534352Z` |
| **executionPath**            | String | Ruta de ejecución de la tarea Flujo de datos                           | `\Transformation\Data Flow Task` |
| **resourceId**               | String | Id. exclusivo del recurso de ADF                                 | `/SUBSCRIPTIONS/<subscriptionID>/RESOURCEGROUPS/<resourceGroupName>/PROVIDERS/MICROSOFT.DATAFACTORY/FACTORIES/<dataFactoryName>` |

### <a name="log-analytics-schema"></a>Esquema de Log Analytics

Log Analytics hereda el esquema de Monitor con las excepciones siguientes:

* La primera letra de cada nombre de columna se escribe en mayúsculas. Por ejemplo, el nombre de columna "correlationId" en Monitor es "CorrelationId" en Log Analytics.
* No hay ninguna columna "Level".
* La columna de "propiedades" dinámicas se conservará como el tipo de blob de JSON dinámico siguiente.

    | Columna de Azure Monitor | Columna de Log Analytics | Tipo |
    | --- | --- | --- |
    | $.properties.UserProperties | UserProperties | Dinámica |
    | $.properties.Annotations | Anotaciones | Dinámica |
    | $.properties.Input | Entrada | Dinámica |
    | $.properties.Output | Output | Dinámica |
    | $.properties.Error.errorCode | ErrorCode | int |
    | $.properties.Error.message | ErrorMessage | string |
    | $.properties.Error | Error | Dinámica |
    | $.properties.Predecessors | Predecessors | Dinámica |
    | $.properties.Parameters | Parámetros | Dinámica |
    | $.properties.SystemParameters | SystemParameters | Dinámica |
    | $.properties.Tags | Etiquetas | Dinámica |

## <a name="monitor-ssis-operations-with-azure-monitor"></a>Supervisión de operaciones de SSIS con Azure Monitor

Para migrar mediante lift-and-shift las cargas de trabajo de SSIS, puede [aprovisionar una instancia de SSIS IR en ADF](https://docs.microsoft.com/azure/data-factory/tutorial-deploy-ssis-packages-azure) que admita:

- La ejecución de paquetes implementados en el catálogo de SSIS (SSISDB) hospedados por un servidor de Azure SQL Database o por Instancia administrada (modelo de implementación de proyectos)
- La ejecución de paquetes implementados en el sistema de archivos, en Azure Files o en una base de datos de SQL Server (MSDB) hospedados por Instancia administrada de Azure SQL (modelo de implementación de paquetes)

Una vez aprovisionado, puede [comprobar el estado operativo de SSIS IR mediante Azure PowerShell o en el centro de **Monitor** del portal de ADF](https://docs.microsoft.com/azure/data-factory/monitor-integration-runtime#azure-ssis-integration-runtime). Con el modelo de implementación de proyectos, los registros de ejecución de paquetes SSIS se almacenan en tablas o vistas internas de SSISDB, de modo que puede consultarlas, analizarlas y presentarlas visualmente mediante herramientas designadas, como SSMS. Con el modelo de implementación de paquetes, los registros de ejecución de paquetes SSIS se pueden almacenar en el sistema de archivos o en Azure Files como archivos CSV que todavía debe analizar y procesar con otras herramientas designadas antes de que pueda consultarlas, analizarlas y presentarlas visualmente.

Ahora, con la integración de [Azure Monitor](https://docs.microsoft.com/azure/azure-monitor/platform/data-platform), puede consultar, analizar y presentar visualmente todas las métricas y registros generados a partir de operaciones de SSIS IR y ejecuciones de paquetes SSIS en Azure Portal. Además, puede generar alertas sobre dichas métricas.

### <a name="configure-diagnostic-settings-and-workspace-for-ssis-operations"></a>Configuración de las opciones de diagnóstico y del área de trabajo para las operaciones de SSIS

Para enviar todas las métricas y los registros generados por las operaciones de SSIS IR y las ejecuciones de paquetes SSIS a Azure Monitor, debe [configurar las opciones de diagnóstico y el área de trabajo para la instancia de ADF](https://docs.microsoft.com/azure/data-factory/monitor-using-azure-monitor#configure-diagnostic-settings-and-workspace).

### <a name="ssis-operational-metrics"></a>Métricas operativas de SSIS

Las [métricas](https://docs.microsoft.com/azure/azure-monitor/platform/data-platform-metrics) operativas de SSIS son contadores de rendimiento o valores numéricos que describen el estado de las operaciones de inicio y detención de SSIS IR, así como de las ejecuciones de paquetes SSIS en un momento determinado. Forman parte de las [métricas de ADF en Azure Monitor](https://docs.microsoft.com/azure/data-factory/monitor-using-azure-monitor#data-factory-metrics).

Al configurar las opciones de diagnóstico y el área de trabajo de la instancia de ADF en Azure Monitor, al activar la casilla _AllMetrics_, las métricas operativas de SSIS estarán disponibles para los [análisis interactivos con el Explorador de métricas de Azure](https://docs.microsoft.com/azure/azure-monitor/platform/metrics-getting-started), la [presentación en el panel de Azure](https://docs.microsoft.com/azure/azure-monitor/learn/tutorial-app-dashboards) y las [alertas casi en tiempo real](https://docs.microsoft.com/azure/azure-monitor/platform/alerts-metric).

![Asignación de un nombre a la configuración y selección de un área de trabajo de Log Analytics](media/data-factory-monitor-oms/monitor-oms-image2.png)

### <a name="ssis-operational-alerts"></a>Alertas operativas de SSIS

Para generar alertas relativas a las métricas operativas de SSIS desde el portal de ADF, [seleccione la página **Alertas y métricas** del centro **Monitor** de ADF y siga las instrucciones detalladas proporcionadas](https://docs.microsoft.com/azure/data-factory/monitor-visually#alerts).

![Generación de alertas operativas de SSIS desde el portal de ADF](media/data-factory-monitor-oms/data-factory-monitor-alerts-ssis.png)

Para generar alertas relativas a las métricas operativas de SSIS desde Azure Portal, [seleccione la página **Alertas** del centro de Azure **Monitor** y siga las instrucciones detalladas proporcionadas](https://docs.microsoft.com/azure/data-factory/monitor-using-azure-monitor#data-factory-alerts).

![Generación de alertas operativas de SSIS desde Azure Portal](media/data-factory-monitor-oms/azure-monitor-alerts-ssis.png)

### <a name="ssis-operational-logs"></a>Registros operativos de SSIS

Los [registros](https://docs.microsoft.com/azure/azure-monitor/platform/data-platform-logs) operativos de SSIS son eventos generados por las operaciones de SSIS IR y las ejecuciones de paquetes SSIS que proporcionan suficiente contexto sobre los problemas identificados y son útiles para el análisis de la causa principal. 

Al configurar las opciones de diagnóstico y el área de trabajo de la instancia de ADF en Azure Monitor, puede seleccionar los registros operativos de SSIS correspondientes y enviarlos a Log Analytics, que se basa en Azure Data Explorer. Allí, estarán disponibles para [realizar análisis mediante el lenguaje de consulta enriquecido](https://docs.microsoft.com/azure/azure-monitor/log-query/log-query-overview), [presentarlos en el panel de Azure](https://docs.microsoft.com/azure/azure-monitor/learn/tutorial-app-dashboards) y [realizar alertas casi en tiempo real](https://docs.microsoft.com/azure/azure-monitor/platform/alerts-log).

![Asignación de un nombre a la configuración y selección de un área de trabajo de Log Analytics](media/data-factory-monitor-oms/monitor-oms-image2.png)

Los esquemas y el contenido de los registros de ejecución de paquetes SSIS en Azure Monitor y Log Analytics son similares a los esquemas de las tablas o vistas internas de SSISDB.

| Categorías de registros de Azure Monitor          | Tablas de Log Analytics                     | Tablas o vistas internas de SSISDB              |
| ------------------------------------- | ---------------------------------------- | ----------------------------------------- |
| `SSISIntegrationRuntimeLogs`          | `ADFSSISIntegrationRuntimeLogs`          |                                           |
| `SSISPackageEventMessageContext`      | `ADFSSISPackageEventMessageContext`      | `[internal].[event_message_context]`      |
| `SSISPackageEventMessages`            | `ADFSSISPackageEventMessages`            | `[internal].[event_messages]`             |
| `SSISPackageExecutableStatistics`     | `ADFSSISPackageExecutableStatistics`     | `[internal].[executable_statistics]`      |
| `SSISPackageExecutionComponentPhases` | `ADFSSISPackageExecutionComponentPhases` | `[internal].[execution_component_phases]` |
| `SSISPackageExecutionDataStatistics`  | `ADFSSISPackageExecutionDataStatistics`  | `[internal].[execution_data_statistics]`  |

Para más información sobre los atributos y las propiedades de los registros operativos de SSIS, vea [Esquemas de Azure Monitor y Log Analytics para ADF](https://docs.microsoft.com/azure/data-factory/monitor-using-azure-monitor#schema-of-logs-and-events).

Los registros de ejecución de los paquetes SSIS seleccionados siempre se envían a Log Analytics, independientemente de los métodos de invocación. Por ejemplo, puede invocar las ejecuciones de paquetes en SSDT habilitado para Azure mediante T-SQL en SSMS, el Agente SQL Server u otras herramientas designadas, y como ejecuciones desencadenadas o de depuración de las actividades de ejecución de paquetes SSIS en canalizaciones de ADF.

Al consultar los registros de operaciones de SSIS IR en Log Analytics, puede usar las propiedades **OperationName** y **ResultType** que están establecidas en `Start/Stop/Maintenance` y `Started/InProgress/Succeeded/Failed`, respectivamente. 

![Consulta de los registros de operaciones de SSIS IR en Log Analytics](media/data-factory-monitor-oms/log-analytics-query.png)

Al consultar los registros de ejecución de paquetes SSIS en Logs Analytics, puede combinarlos mediante las propiedades **OperationId**/**ExecutionId**/**CorrelationId**. **OperationId**/**ExecutionId** siempre se establecen en `1` para todas las operaciones o ejecuciones relacionadas con los paquetes que **no** están almacenados en SSISDB o se invocan mediante T-SQL.

![Consulta de los registros de ejecución de paquetes SSIS en Log Analytics](media/data-factory-monitor-oms/log-analytics-query2.png)

## <a name="next-steps"></a>Pasos siguientes
[Supervisión y administración de canalizaciones mediante programación](monitor-programmatically.md)
