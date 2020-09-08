---
title: Servicios y esquemas admitidos de los registros de recursos de Azure
description: Conozca el esquema de los eventos y servicios admitidos para los registros de recurso de Azure.
ms.subservice: logs
ms.topic: reference
ms.date: 06/15/2020
ms.openlocfilehash: fd2dc4030816ab0b31befe46ac60d5e96fdae917
ms.sourcegitcommit: 9c3cfbe2bee467d0e6966c2bfdeddbe039cad029
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/24/2020
ms.locfileid: "88782131"
---
# <a name="common-and-service-specific-schema-for-azure-resource-logs"></a>Esquema específico de servicio y común para los registros de recursos de Azure

> [!NOTE]
> Los registros de recurso se conocían anteriormente como registros de diagnóstico. El nombre se cambió en octubre de 2019, dado que los tipos de registros recopilados por Azure Monitor evolucionaron para incluir algo más que tan solo el recurso de Azure. Además, la lista de las categorías del registro de recursos que se podían recopilar se solía mostrar en este artículo. Se trasladaron a [Categorías del registro de recursos](resource-logs-categories.md). 

Los [registros de recursos de Azure Monitor](./platform-logs-overview.md) son los registros emitidos por los servicios de Azure que describen el funcionamiento de dichos servicios o recursos. Todos los registros de recursos disponibles a través de Azure Monitor comparten un esquema común de nivel superior, con flexibilidad para que cada servicio emita propiedades únicas para sus propios eventos.

Una combinación del tipo de recurso (disponible en la propiedad `resourceId`) y la `category` identifica un esquema de forma única. En este artículo se describe el esquema de nivel superior para los registros de recursos y los vínculos a los esquemas para cada servicio.


## <a name="top-level-common-schema"></a>Esquema común de nivel superior

| Nombre | Obligatorio/opcional | Descripción |
|---|---|---|
| time | Obligatorio | Marca de tiempo (UTC) del evento. |
| resourceId | Obligatorio | Identificador del recurso que ha emitido el evento. Para los servicios de inquilino, presenta la forma /tenants/tenant-id/providers/provider-name. |
| tenantId | Necesario para los registros de inquilinos | El identificador del inquilino de Active Directory al que está asociado este evento. Esta propiedad solo se usa para los registros de nivel de inquilino; no aparece en los registros de nivel de recurso. |
| operationName | Obligatorio | Nombre de la operación representada por este evento. Si el evento representa una operación RBAC, este es el nombre de la operación RBAC (por ejemplo, Microsoft.Storage/storageAccounts/blobServices/blobs/Read). Se suele modelar con la forma de una operación de Resource Manager, incluso si no son operaciones de Resource Manager documentadas reales (`Microsoft.<providerName>/<resourceType>/<subtype>/<Write/Read/Delete/Action>`) |
| operationVersion | Opcional | La versión de API asociada con la operación, si operationName se ha realizado mediante una API (por ejemplo, `http://myservice.windowsazure.net/object?api-version=2016-06-01`). Si no hay ninguna API que se corresponde con esta operación, la versión representa la versión de esa operación en caso de que las propiedades asociadas con la operación cambien en el futuro. |
| category | Obligatorio | Categoría de registro del evento. La categoría es la granularidad con la que se pueden habilitar o deshabilitar los registros en un recurso determinado. Las propiedades que aparecen en el blob de propiedades de un evento son las mismas dentro de una categoría de registro y un tipo de recurso concretos. Las categorías de registro típicas son "Audit", "Operational" "Execution" y "Request". |
| resultType | Opcional | Estado del evento. Entre los valores habituales, se incluyen Started, In Progress, Succeeded, Failed, Active y Resolved. |
| resultSignature | Opcional | Subestado del evento. Si esta operación se corresponde con una llamada API de REST, este campo es el código de estado HTTP de la llamada REST correspondiente. |
| resultDescription | Opcional | Descripción de texto estático de esta operación, por ejemplo, "Obtención de archivo de almacenamiento". |
| durationMs | Opcional | Duración de la operación en milisegundos. |
| callerIpAddress | Opcional | Dirección IP del autor de la llamada, si la operación se corresponde con una llamada API que podría proceder de una entidad con una dirección IP disponible públicamente. |
| correlationId | Opcional | GUID que se usa para agrupar un conjunto de eventos relacionados. Normalmente, si dos eventos tienen el mismo operationName pero dos estados diferentes (por ejemplo, "Iniciado" y "Correcto"), comparten el mismo Id. de correlación. Esto también puede representar otras relaciones entre los eventos. |
| identity | Opcional | Blob JSON que describe la identidad del usuario o la aplicación que realizó la operación. Normalmente esto incluye la autorización y las notificaciones o el token JWT de Active Directory. |
| Nivel | Opcional | Nivel de gravedad del evento. Debe ser uno de entre Informativo, Advertencia, Error o Crítico. |
| ubicación | Opcional | La región del recurso que emite el evento, por ejemplo, "Este de EE. UU." o "Sur de Francia" |
| properties | Opcional | Todas las propiedades extendidas relacionadas con esta categoría de eventos determinada. Todas las propiedades personalizadas o únicas se deben colocar dentro de esta "Parte B" del esquema. |

## <a name="service-specific-schemas"></a>Esquemas específicos por servicio

El esquema para los registros de recursos varía según la categoría de registro y el recurso. En esta lista se muestran los servicios que hacen que los registros de recursos estén disponibles y los vínculos para el servicio y el esquema específico de la categoría cuando estén disponibles. Esta lista cambia todo el tiempo a medida que se agregan nuevos servicios, por lo que si no ve lo que necesita, use un motor de búsqueda para detectar documentación adicional. No dude en abrir una incidencia de GitHub sobre este artículo para que podamos actualizarlo.

| Servicio | Esquema y documentos |
| --- | --- |
| Azure Active Directory | [Información general](../../active-directory/reports-monitoring/concept-activity-logs-azure-monitor.md), [Esquema de registros de auditoría](../../active-directory/reports-monitoring/reference-azure-monitor-audit-log-schema.md) y [Esquema de inicios de sesión](../../active-directory/reports-monitoring/reference-azure-monitor-sign-ins-log-schema.md) |
| Analysis Services | https://docs.microsoft.com/azure/analysis-services/analysis-services-logging |
| API Management | [Registros de recursos de API Management](../../api-management/api-management-howto-use-azure-monitor.md#resource-logs) |
| Puertas de enlace de aplicaciones |[Registro para Application Gateway](../../application-gateway/application-gateway-diagnostics.md) |
| Azure Automation |[Log Analytics para Azure Automation](../../automation/automation-manage-send-joblogs-log-analytics.md) |
| Azure Batch |[Registro de Azure Batch](../../batch/batch-diagnostics.md) |
| Azure Database for MySQL | [Registros de diagnóstico de Azure Database for MySQL](../../mysql/concepts-server-logs.md#diagnostic-logs) |
| Azure Database for PostgreSQL | [Registros de Azure Database for PostgreSQL](../../postgresql/concepts-server-logs.md#resource-logs) |
| Explorador de datos de Azure | [Registros de Azure Data Explorer](/azure/data-explorer/using-diagnostic-logs) |
| Cognitive Services | [Registro para Azure Cognitive Services](../../cognitive-services/diagnostic-logging.md) |
| Container Registry | [Registro para Azure Container Registry](../../container-registry/container-registry-diagnostics-audit-logs.md) |
| Content Delivery Network | [Registros de Azure para CDN](../../cdn/cdn-azure-diagnostic-logs.md) |
| CosmosDB | [Registro de Azure Cosmos DB](../../cosmos-db/monitor-cosmos-db.md) |
| Data Factory | [Supervisión de factorías de datos mediante Azure Monitor](../../data-factory/monitor-using-azure-monitor.md) |
| Data Lake Analytics |[Acceso a los registros para Azure Data Lake Analytics](../../data-lake-analytics/data-lake-analytics-diagnostic-logs.md) |
| Data Lake Store |[Acceso a los registros para Azure Data Lake Store](../../data-lake-store/data-lake-store-diagnostic-logs.md) |
| Azure Databricks | [Registro de diagnósticos en Azure Databricks](https://github.com/MicrosoftDocs/databricks-pr/blob/live/databricks/administration-guide/account-settings/azure-diagnostic-logs.md) |
| Event Hubs |[Registros de Azure Event Hubs](../../event-hubs/event-hubs-diagnostic-logs.md) |
| ExpressRoute | Esquema no disponible. |
| Azure Firewall | Esquema no disponible. |
| IoT Hub | [Operaciones de IoT Hub](../../iot-hub/iot-hub-monitor-resource-health.md#use-azure-monitor) |
| Key Vault |[Registro de Azure Key Vault](../../key-vault/general/logging.md) |
| Kubernetes Service |[Registro de Kubernetes de Azure](../../aks/view-master-logs.md#log-event-schema) |
| Load Balancer |[Log Analytics para Azure Load Balancer](../../load-balancer/load-balancer-monitor-log.md) |
| Logic Apps |[Esquema de seguimiento personalizado de Logic Apps B2B](../../logic-apps/logic-apps-track-integration-account-custom-tracking-schema.md) |
| Grupos de seguridad de red |[Análisis del registro para grupos de seguridad de red (NSG)](../../virtual-network/virtual-network-nsg-manage-log.md) |
| DDOS Protection | [Administración de Azure DDoS Protection estándar](../../virtual-network/manage-ddos-protection.md) |
| Power BI dedicado | [Registro para Power BI Embedded en Azure](/power-bi/developer/azure-pbie-diag-logs) |
| Recovery Services | [Modelo de datos para Azure Backup](../../backup/backup-azure-reports-data-model.md)|
| Search |[Habilitación y uso de Análisis de tráfico de búsqueda](../../search/search-traffic-analytics.md) |
| Azure Service Bus |[Registros de Azure Service Bus](../../service-bus-messaging/service-bus-diagnostic-logs.md) |
| SQL Database | [Registro de Azure SQL Database](../../azure-sql/database/metrics-diagnostic-telemetry-logging-streaming-export-configure.md) |
| Stream Analytics |[Registros de trabajo](../../stream-analytics/stream-analytics-job-diagnostic-logs.md) |
| Traffic Manager | [Esquema de registro de Traffic Manager](../../traffic-manager/traffic-manager-diagnostic-logs.md) |
| Virtual Networks | Esquema no disponible. |
| Puertas de enlace de red virtual | Esquema no disponible. |


## <a name="next-steps"></a>Pasos siguientes

* [Visualización de las categorías de registro de recursos que se pueden recopilar](resource-logs-categories.md)
* [Más información sobre los registros de recurso](./platform-logs-overview.md)
* [Transmisión de registros de recursos a **Event Hubs**](./resource-logs.md#send-to-azure-event-hubs)
* [Cambio de la configuración de diagnóstico del registro de recursos con la API REST de Azure Monitor](/rest/api/monitor/diagnosticsettings)
* [Análisis de registros desde Azure Storage con Log Analytics](./resource-logs.md#send-to-log-analytics-workspace)

