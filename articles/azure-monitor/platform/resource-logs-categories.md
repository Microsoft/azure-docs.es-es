---
title: Servicios y categorías admitidos de los registros de recursos de Azure Monitor
description: Referencia de Azure Monitor para conocer el esquema de los eventos y servicios admitidos para los registros de recursos de Azure.
ms.subservice: logs
ms.topic: reference
ms.date: 06/03/2020
ms.openlocfilehash: 81f79b81c03e7996d7f6d45b002d8160740c3c14
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87318307"
---
# <a name="supported-categories-for-azure-resource-logs"></a>Categorías admitidas en los registros de recursos de Azure

> [!NOTE]
> Los registros de recurso se conocían anteriormente como registros de diagnóstico. El nombre se cambió en octubre de 2019, dado que los tipos de registros recopilados por Azure Monitor evolucionaron para incluir algo más que tan solo el recurso de Azure.

Los [registros de recursos de Azure Monitor](./platform-logs-overview.md) son los registros emitidos por los servicios de Azure que describen el funcionamiento de dichos servicios o recursos. Todos los registros de recursos disponibles a través de Azure Monitor comparten un esquema común de nivel superior, con flexibilidad para que cada servicio emita propiedades únicas para sus propios eventos.

Una combinación del tipo de recurso (disponible en la propiedad `resourceId`) y la `category` identifica un esquema de forma única. Hay un esquema común para todos los registros de recursos con campos específicos de servicio que se han agregado para distintas categorías de registros. Para obtener más información, consulte [Esquema específico de servicio y común para los registros de recursos de Azure]().

## <a name="supported-log-categories-per-resource-type"></a>Categorías de registro admitidas por tipo de recurso

A continuación, se muestra una lista de los tipos de registros disponibles para cada tipo de recurso. 

Es posible que algunas categorías solo se admitan para tipos específicos de recursos. Consulte la documentación específica de recurso si cree que falta un recurso. Por ejemplo, las categorías Microsoft.Sql/servers/databases no están disponibles para todos los tipos de bases de datos. Para más información, consulte la [información sobre el registro de diagnóstico de SQL Database](../../azure-sql/database/metrics-diagnostic-telemetry-logging-streaming-export-configure.md). 

Si todavía falta algo, puede escribir un comentario de GitHub en la parte inferior de este artículo.

## <a name="microsoftaadiamtenants"></a>microsoft.aadiam/tenants

|Category|Nombre para mostrar de categoría|
|---|---|
|Inicio de sesión|Inicio de sesión|


## <a name="microsoftanalysisservicesservers"></a>Microsoft.AnalysisServices/servers

|Category|Nombre para mostrar de categoría|
|---|---|
|Motor|Motor|
|Servicio|Servicio|


## <a name="microsoftapimanagementservice"></a>Microsoft.ApiManagement/service

|Category|Nombre para mostrar de categoría|
|---|---|
|GatewayLogs|Registros relacionados con la puerta de enlace de ApiManagement|


## <a name="microsoftappplatformspring"></a>Microsoft.AppPlatform/Spring

|Category|Nombre para mostrar de categoría|
|---|---|
|ApplicationConsole|Consola de aplicación|
|SystemLogs|Registros del sistema|


## <a name="microsoftautomationautomationaccounts"></a>Microsoft.Automation/automationAccounts

|Category|Nombre para mostrar de categoría|
|---|---|
|JobLogs|Registros de trabajo|
|JobStreams|Flujos de trabajo|
|DscNodeStatus|Estado del nodo de DSC|


## <a name="microsoftbatchbatchaccounts"></a>Microsoft.Batch/batchAccounts

|Category|Nombre para mostrar de categoría|
|---|---|
|ServiceLog|Registros de servicios|


## <a name="microsoftbatchaiworkspaces"></a>Microsoft.BatchAI/workspaces

|Category|Nombre para mostrar de categoría|
|---|---|
|BaiClusterEvent|BaiClusterEvent|
|BaiClusterNodeEvent|BaiClusterNodeEvent|
|BaiJobEvent|BaiJobEvent|


## <a name="microsoftblockchainblockchainmembers"></a>Microsoft.Blockchain/blockchainMembers

|Category|Nombre para mostrar de categoría|
|---|---|
|BlockchainApplication|Aplicación de cadena de bloques|
|Proxy|Proxy|


## <a name="microsoftblockchaincordamembers"></a>Microsoft.Blockchain/cordaMembers

|Category|Nombre para mostrar de categoría|
|---|---|
|BlockchainApplication|Aplicación de cadena de bloques|


## <a name="microsoftcdncdnwebapplicationfirewallpolicies"></a>Microsoft.Cdn/cdnwebapplicationfirewallpolicies

|Category|Nombre para mostrar de categoría|
|---|---|
|WebApplicationFirewallLogs|Registros de firewall de aplicaciones web|


## <a name="microsoftcdnprofiles"></a>Microsoft.Cdn/profiles

|Category|Nombre para mostrar de categoría|
|---|---|
|AzureCdnAccessLog|Registro de acceso a Azure CDN|


## <a name="microsoftcdnprofilesendpoints"></a>Microsoft.Cdn/profiles/endpoints

|Category|Nombre para mostrar de categoría|
|---|---|
|CoreAnalytics|Obtiene las métricas del punto de conexión; por ejemplo, ancho de banda, salida, etc.|


## <a name="microsoftclassicnetworknetworksecuritygroups"></a>Microsoft.ClassicNetwork/networkSecurityGroups

|Category|Nombre para mostrar de categoría|
|---|---|
|Evento de flujo de reglas de grupo de seguridad de red|Evento de flujo de reglas de grupo de seguridad de red|


## <a name="microsoftcognitiveservicesaccounts"></a>Microsoft.CognitiveServices/accounts

|Category|Nombre para mostrar de categoría|
|---|---|
|Auditoría|Registros de auditoría|
|RequestResponse|Registros de solicitud y respuesta|


## <a name="microsoftcontainerregistryregistries"></a>Microsoft.ContainerRegistry/registries

|Category|Nombre para mostrar de categoría|
|---|---|
|ContainerRegistryLoginEvents|Eventos de inicio de sesión|
|ContainerRegistryRepositoryEvents|Registros de RepositoryEvent|


## <a name="microsoftcontainerservicemanagedclusters"></a>Microsoft.ContainerService/managedClusters

|Category|Nombre para mostrar de categoría|
|---|---|
|cluster-autoscaler|Cluster Autoscaler de Kubernetes|
|kube-apiserver|Servidor de la API de Kubernetes|
|kube-audit|Auditoría de Kubernetes|
|kube-controller-manager|Administrador del controlador de Kubernetes|
|kube-scheduler|Programador de Kubernetes|


## <a name="microsoftcustomprovidersresourceproviders"></a>Microsoft.CustomProviders/resourceproviders

|Category|Nombre para mostrar de categoría|
|---|---|
|AuditLogs|Registros de auditoría para llamadas de MiniRP|


## <a name="microsoftdatabricksworkspaces"></a>Microsoft.Databricks/workspaces

|Category|Nombre para mostrar de categoría|
|---|---|
|accounts|Cuentas de Databricks|
|clusters|Clústeres de Databricks|
|dbfs|Sistema de archivos de Databricks|
|instancePools|Grupos de instancias|
|jobs|Trabajos de Databricks|
|notebook|Cuaderno de Databricks|
|secrets|Secretos de Databricks|
|sqlPermissions|Permisos SQL de Databricks|
|ssh|SSH de Databricks|
|área de trabajo|Área de trabajo de Databricks|


## <a name="microsoftdatacatalogdatacatalogs"></a>Microsoft.DataCatalog/datacatalogs

|Category|Nombre para mostrar de categoría|
|---|---|
|ScanStatusLogEvent|ScanStatus|


## <a name="microsoftdatafactoryfactories"></a>Microsoft.DataFactory/factories

|Category|Nombre para mostrar de categoría|
|---|---|
|ActivityRuns|Registro de ejecuciones de actividad de canalización|
|PipelineRuns|Registro de ejecuciones de canalización|
|TriggerRuns|Registro de ejecuciones de desencadenador|


## <a name="microsoftdatalakestoreaccounts"></a>Microsoft.DataLakeStore/accounts

|Category|Nombre para mostrar de categoría|
|---|---|
|Auditoría|Registros de auditoría|
|Requests|Registros de solicitud|


## <a name="microsoftdbformariadbservers"></a>Microsoft.DBforMariaDB/servers

|Category|Nombre para mostrar de categoría|
|---|---|
|MySqlAuditLogs|Registros de auditoría de MariaDB|
|MySqlSlowLogs|Registros de servidor de MariaDB|


## <a name="microsoftdbformysqlservers"></a>Microsoft.DBforMySQL/servers

|Category|Nombre para mostrar de categoría|
|---|---|
|MySqlAuditLogs|Registros de auditoría de MySQL|
|MySqlSlowLogs|Registros de MySQL Server|


## <a name="microsoftdbforpostgresqlservers"></a>Microsoft.DBforPostgreSQL/servers

|Category|Nombre para mostrar de categoría|
|---|---|
|PostgreSQLLogs|Registros del servidor PostgreSQL|
|QueryStoreRuntimeStatistics|Estadísticas de tiempo de ejecución del almacén de consultas de PostgreSQL|
|QueryStoreWaitStatistics|Estadísticas de espera del almacén de consultas de PostgreSQL|


## <a name="microsoftdbforpostgresqlserversv2"></a>Microsoft.DBforPostgreSQL/serversv2

|Category|Nombre para mostrar de categoría|
|---|---|
|PostgreSQLLogs|Registros del servidor PostgreSQL|


## <a name="microsoftdbforpostgresqlsingleservers"></a>Microsoft.DBforPostgreSQL/singleservers

|Category|Nombre para mostrar de categoría|
|---|---|
|PostgreSQLLogs|Registros del servidor PostgreSQL|


## <a name="microsoftdesktopvirtualizationapplicationgroups"></a>Microsoft.DesktopVirtualization/applicationgroups

|Category|Nombre para mostrar de categoría|
|---|---|
|Punto de control|Punto de control|
|Error|Error|
|Administración|Administración|


## <a name="microsoftdesktopvirtualizationhostpools"></a>Microsoft.DesktopVirtualization/hostpools

|Category|Nombre para mostrar de categoría|
|---|---|
|Punto de control|Punto de control|
|Conexión|Conexión|
|Error|Error|
|HostRegistration|HostRegistration|
|Administración|Administración|


## <a name="microsoftdesktopvirtualizationworkspaces"></a>Microsoft.DesktopVirtualization/workspaces

|Category|Nombre para mostrar de categoría|
|---|---|
|Punto de control|Punto de control|
|Error|Error|
|Fuente|Fuente|
|Administración|Administración|


## <a name="microsoftdevicesiothubs"></a>Microsoft.Devices/IotHubs

|Category|Nombre para mostrar de categoría|
|---|---|
|C2DCommands|Comandos C2D|
|C2DTwinOperations|Operaciones gemelas C2D|
|Configurations|Configurations|
|Conexiones|Conexiones|
|D2CTwinOperations|D2CTwinOperations|
|DeviceIdentityOperations|Operaciones de identidad de dispositivos|
|DeviceStreams|Flujos de dispositivos (versión preliminar)|
|DeviceTelemetry|Telemetría de dispositivo|
|DirectMethods|Métodos directos|
|DistributedTracing|Seguimiento distribuido (versión preliminar)|
|FileUploadOperations|Operaciones de carga de archivos|
|JobsOperations|Operaciones de trabajos|
|Rutas|Rutas|
|TwinQueries|Consultas gemelas|


## <a name="microsoftdevicesprovisioningservices"></a>Microsoft.Devices/provisioningServices

|Category|Nombre para mostrar de categoría|
|---|---|
|DeviceOperations|Operaciones de dispositivo|
|ServiceOperations|Operaciones del servicio|


## <a name="microsoftdocumentdbdatabaseaccounts"></a>Microsoft.DocumentDB/databaseAccounts

|Category|Nombre para mostrar de categoría|
|---|---|
|CassandraRequests|CassandraRequests|
|ControlPlaneRequests|ControlPlaneRequests|
|DataPlaneRequests|DataPlaneRequests|
|MongoRequests|MongoRequests|
|PartitionKeyRUConsumption|PartitionKeyRUConsumption|
|PartitionKeyStatistics|PartitionKeyStatistics|
|QueryRuntimeStatistics|QueryRuntimeStatistics|


## <a name="microsoftenterpriseknowledgegraphservices"></a>Microsoft.EnterpriseKnowledgeGraph/services

|Category|Nombre para mostrar de categoría|
|---|---|
|AuditEvent|Registro de AuditEvent|
|DataIssue|Registro de DataIssue|
|Requests|Registro de configuración|

## <a name="microsofteventgriddomains"></a>Microsoft.EventGrid/domains

|Category|Nombre para mostrar de categoría|
|---|---|
|DeliveryFailures|Registros de errores de entrega|
|PublishFailures|Registros de errores de publicación|


## <a name="microsofteventgridsystemtopics"></a>Microsoft.EventGrid/systemTopics

|Category|Nombre para mostrar de categoría|
|---|---|
|DeliveryFailures|Registros de errores de entrega|


## <a name="microsofteventgridtopics"></a>Microsoft.EventGrid/topics

|Category|Nombre para mostrar de categoría|
|---|---|
|DeliveryFailures|Registros de errores de entrega|
|PublishFailures|Registros de errores de publicación|


## <a name="microsofteventhubnamespaces"></a>Microsoft.EventHub/namespaces

|Category|Nombre para mostrar de categoría|
|---|---|
|ArchiveLogs|Registros de archivo|
|AutoScaleLogs|Registros de escalado automático|
|CustomerManagedKeyUserLogs|Registros de clave administrada por el cliente|
|EventHubVNetConnectionEvent|Registros de conexión de filtrado de red virtual/IP|
|KafkaCoordinatorLogs|Registros de coordinador de Kafka|
|KafkaUserErrorLogs|Registros de error de usuario de Kafka|
|OperationalLogs|Registros operativos|


## <a name="microsofthealthcareapisservices"></a>Microsoft.HealthcareApis/services

|Category|Nombre para mostrar de categoría|
|---|---|
|AuditLogs|Registros de auditoría|


## <a name="microsoftinsightsautoscalesettings"></a>Microsoft.Insights/AutoscaleSettings

|Category|Nombre para mostrar de categoría|
|---|---|
|AutoscaleEvaluations|Evaluaciones de escalabilidad automática|
|AutoscaleScaleActions|Acciones de escalabilidad automática|


## <a name="microsoftinsightscomponents"></a>Microsoft.Insights/Components

|Category|Nombre para mostrar de categoría|
|---|---|
|AppAvailabilityResults|Resultados de disponibilidad|
|AppBrowserTimings|Intervalos de explorador|
|AppDependencies|Dependencias|
|AppEvents|Eventos|
|AppExceptions|Excepciones|
|AppMetrics|Métricas|
|AppPageViews|Vistas de página|
|AppPerformanceCounters|Contadores de rendimiento|
|AppRequests|Requests|
|AppSystemEvents|Eventos del sistema|
|AppTraces|Traces|


## <a name="microsoftiotspacesgraph"></a>Microsoft.IoTSpaces/Graph

|Category|Nombre para mostrar de categoría|
|---|---|
|Auditoría|Auditoría|
|Salida|Salida|
|Entrada|Entrada|
|Operativos|Operativos|
|Seguimiento|Seguimiento|
|UserDefinedFunction|UserDefinedFunction|


## <a name="microsoftkeyvaultvaults"></a>Microsoft.KeyVault/vaults

|Category|Nombre para mostrar de categoría|
|---|---|
|AuditEvent|Registros de auditoría|


## <a name="microsoftkustoclusters"></a>Microsoft.Kusto/Clusters

|Category|Nombre para mostrar de categoría|
|---|---|
|FailedIngestion|Operaciones de ingesta con error|
|SucceededIngestion|Operaciones de ingesta correctas|


## <a name="microsoftlogicintegrationaccounts"></a>Microsoft.Logic/integrationAccounts

|Category|Nombre para mostrar de categoría|
|---|---|
|IntegrationAccountTrackingEvents|Eventos de seguimiento de la cuenta de integración|


## <a name="microsoftlogicworkflows"></a>Microsoft.Logic/workflows

|Category|Nombre para mostrar de categoría|
|---|---|
|WorkflowRuntime|Eventos de diagnóstico en tiempo de ejecución de flujo de trabajo|


## <a name="microsoftmachinelearningservicesworkspaces"></a>Microsoft.MachineLearningServices/workspaces

|Category|Nombre para mostrar de categoría|
|---|---|
|AmlComputeClusterEvent|AmlComputeClusterEvent|
|AmlComputeClusterNodeEvent|AmlComputeClusterNodeEvent|
|AmlComputeCpuGpuUtilization|AmlComputeCpuGpuUtilization|
|AmlComputeJobEvent|AmlComputeJobEvent|
|AmlRunStatusChangedEvent|AmlRunStatusChangedEvent|


## <a name="microsoftmediamediaservices"></a>Microsoft.Media/mediaservices

|Category|Nombre para mostrar de categoría|
|---|---|
|KeyDeliveryRequests|Solicitudes de entrega de clave|


## <a name="microsoftnetworkapplicationgateways"></a>Microsoft.Network/applicationGateways

|Category|Nombre para mostrar de categoría|
|---|---|
|ApplicationGatewayAccessLog|Registro de acceso de Application Gateway|
|ApplicationGatewayFirewallLog|Registro de Firewall de Application Gateway|
|ApplicationGatewayPerformanceLog|Registro de rendimiento de Application Gateway|


## <a name="microsoftnetworkazurefirewalls"></a>Microsoft.Network/azurefirewalls

|Category|Nombre para mostrar de categoría|
|---|---|
|AzureFirewallApplicationRule|Regla de aplicación de Azure Firewall|
|AzureFirewallNetworkRule|Regla de red de Azure Firewall|


## <a name="microsoftnetworkbastionhosts"></a>Microsoft.Network/bastionHosts

|Category|Nombre para mostrar de categoría|
|---|---|
|BastionAuditLogs|Registros de auditoría de Bastion|


## <a name="microsoftnetworkexpressroutecircuits"></a>Microsoft.Network/expressRouteCircuits

|Category|Nombre para mostrar de categoría|
|---|---|
|PeeringRouteLog|Registros de tablas de rutas de emparejamiento|


## <a name="microsoftnetworkfrontdoors"></a>Microsoft.Network/frontdoors

|Category|Nombre para mostrar de categoría|
|---|---|
|FrontdoorAccessLog|Registro de acceso de Front Door|
|FrontdoorWebApplicationFirewallLog|Registro de firewall de aplicaciones web de Front Door|


## <a name="microsoftnetworkloadbalancers"></a>Microsoft.Network/loadBalancers

|Category|Nombre para mostrar de categoría|
|---|---|
|LoadBalancerAlertEvent|Eventos de alerta de equilibrador de carga|
|LoadBalancerProbeHealthStatus|Estado de mantenimiento de sondeo de equilibrador de carga|


## <a name="microsoftnetworknetworksecuritygroups"></a>Microsoft.Network/networksecuritygroups

|Category|Nombre para mostrar de categoría|
|---|---|
|NetworkSecurityGroupEvent|Evento de grupo de seguridad de red|
|NetworkSecurityGroupFlowEvent|Evento de flujo de reglas de grupo de seguridad de red|
|NetworkSecurityGroupRuleCounter|Contador de reglas de grupo de seguridad de red|


## <a name="microsoftnetworkpublicipaddresses"></a>Microsoft.Network/publicIPAddresses

|Category|Nombre para mostrar de categoría|
|---|---|
|DDoSMitigationFlowLogs|Registros de flujos de las decisiones de mitigación de DDoS|
|DDoSMitigationReports|Informes de las mitigaciones de DDoS|
|DDoSProtectionNotifications|Notificaciones de protección contra DDoS|


## <a name="microsoftnetworktrafficmanagerprofiles"></a>Microsoft.Network/trafficManagerProfiles

|Category|Nombre para mostrar de categoría|
|---|---|
|ProbeHealthStatusEvents|Evento de resultados de estado de sondeo de Traffic Manager|


## <a name="microsoftnetworkvirtualnetworkgateways"></a>Microsoft.Network/virtualNetworkGateways

|Category|Nombre para mostrar de categoría|
|---|---|
|GatewayDiagnosticLog|Registros de diagnóstico de puerta de enlace|
|IKEDiagnosticLog|Registros de IKE Diagnostic|
|P2SDiagnosticLog|Registros de P2S Diagnostic|
|RouteDiagnosticLog|Registros de diagnóstico de ruta|
|TunnelDiagnosticLog|Registros de diagnóstico de túnel|


## <a name="microsoftnetworkvirtualnetworks"></a>Microsoft.Network/virtualNetworks

|Category|Nombre para mostrar de categoría|
|---|---|
|VMProtectionAlerts|Alertas de protección de máquina virtual|


## <a name="microsoftpowerbidedicatedcapacities"></a>Microsoft.PowerBIDedicated/capacities

|Category|Nombre para mostrar de categoría|
|---|---|
|Motor|Motor|


## <a name="microsoftrecoveryservicesvaults"></a>Microsoft.RecoveryServices/Vaults

|Category|Nombre para mostrar de categoría|
|---|---|
|AddonAzureBackupAlerts|Datos de alerta de Azure Backup del complemento|
|AddonAzureBackupJobs|Datos de trabajo de Azure Backup del complemento|
|AddonAzureBackupPolicy|Datos de directiva de Azure Backup del complemento|
|AddonAzureBackupProtectedInstance|Datos de instancia protegida de Azure Backup del complemento|
|AddonAzureBackupStorage|Datos de almacenamiento de Azure Backup del complemento|
|AzureBackupReport|Datos de informes de Azure Backup|
|AzureSiteRecoveryEvents|Eventos de Azure Site Recovery|
|AzureSiteRecoveryJobs|Trabajos de Azure Site Recovery|
|AzureSiteRecoveryProtectedDiskDataChurn|Actividad de los datos del disco protegido de Azure Site Recovery|
|AzureSiteRecoveryRecoveryPoints|Puntos de recuperación de Azure Site Recovery|
|AzureSiteRecoveryReplicatedItems|Elementos replicados de Azure Site Recovery|
|AzureSiteRecoveryReplicationDataUploadRate|Velocidad de carga de datos de replicación de Azure Site Recovery|
|AzureSiteRecoveryReplicationStats|Estadísticas de replicación de Azure Site Recovery|
|CoreAzureBackup|Datos principales de Azure Backup|


## <a name="microsoftrelaynamespaces"></a>Microsoft.Relay/namespaces

|Category|Nombre para mostrar de categoría|
|---|---|
|HybridConnectionsEvent|Eventos de HybridConnections|


## <a name="microsoftsearchsearchservices"></a>Microsoft.Search/searchServices

|Category|Nombre para mostrar de categoría|
|---|---|
|OperationLogs|Registros de operaciones|


## <a name="microsoftservicebusnamespaces"></a>Microsoft.ServiceBus/namespaces

|Category|Nombre para mostrar de categoría|
|---|---|
|OperationalLogs|Registros operativos|


## <a name="microsoftsignalrservicesignalr"></a>Microsoft.SignalRService/SignalR

|Category|Nombre para mostrar de categoría|
|---|---|
|AllLogs|Registros de Azure SignalR Service|


## <a name="microsoftsqlmanagedinstances"></a>Microsoft.Sql/managedInstances

|Category|Nombre para mostrar de categoría|
|---|---|
|DevOpsOperationsAudit|Registros de auditoría de operaciones Devops|
|ResourceUsageStats|Estadísticas de uso de recursos|
|SQLSecurityAuditEvents|Evento de auditoría de seguridad de SQL|


## <a name="microsoftsqlmanagedinstancesdatabases"></a>Microsoft.Sql/managedInstances/databases

|Category|Nombre para mostrar de categoría|
|---|---|
|Errors|Errors|
|QueryStoreRuntimeStatistics|Estadísticas de tiempo de ejecución del Almacén de consultas|
|QueryStoreWaitStatistics|Estadísticas de espera del Almacén de consultas|
|SQLInsights|SQL Insights|


## <a name="microsoftsqlserversdatabases"></a>Microsoft.Sql/servers/databases

|Category|Nombre para mostrar de categoría|
|---|---|
|AutomaticTuning|Ajuste automático|
|Blocks|Blocks|
|DatabaseWaitStatistics|Estadísticas de espera de base de datos|
|Interbloqueos|Interbloqueos|
|DevOpsOperationsAudit|Registros de auditoría de operaciones Devops|
|DmsWorkers|Trabajo de DMS|
|Errors|Errors|
|ExecRequests|Solicitudes de ejecución|
|QueryStoreRuntimeStatistics|Estadísticas de tiempo de ejecución del Almacén de consultas|
|QueryStoreWaitStatistics|Estadísticas de espera del Almacén de consultas|
|RequestSteps|Pasos de la solicitud|
|SQLInsights|SQL Insights|
|SqlRequests|Solicitudes de SQL|
|SQLSecurityAuditEvents|Evento de auditoría de seguridad de SQL|
|Tiempos de expiración|Tiempos de expiración|
|Esperas|Esperas|


## <a name="microsoftstoragestorageaccountsblobservices"></a>Microsoft.Storage/storageAccounts/blobServices

|Category|Nombre para mostrar de categoría|
|---|---|
|StorageDelete|StorageDelete|
|StorageRead|StorageRead|
|StorageWrite|StorageWrite|


## <a name="microsoftstoragestorageaccountsfileservices"></a>Microsoft.Storage/storageAccounts/fileServices

|Category|Nombre para mostrar de categoría|
|---|---|
|StorageDelete|StorageDelete|
|StorageRead|StorageRead|
|StorageWrite|StorageWrite|


## <a name="microsoftstoragestorageaccountsqueueservices"></a>Microsoft.Storage/storageAccounts/queueServices

|Category|Nombre para mostrar de categoría|
|---|---|
|StorageDelete|StorageDelete|
|StorageRead|StorageRead|
|StorageWrite|StorageWrite|


## <a name="microsoftstoragestorageaccountstableservices"></a>Microsoft.Storage/storageAccounts/tableServices

|Category|Nombre para mostrar de categoría|
|---|---|
|StorageDelete|StorageDelete|
|StorageRead|StorageRead|
|StorageWrite|StorageWrite|


## <a name="microsoftstreamanalyticsstreamingjobs"></a>Microsoft.StreamAnalytics/streamingjobs

|Category|Nombre para mostrar de categoría|
|---|---|
|Creación|Creación|
|Ejecución|Ejecución|


## <a name="microsoftwebhostingenvironments"></a>microsoft.web/hostingenvironments

|Category|Nombre para mostrar de categoría|
|---|---|
|AppServiceEnvironmentPlatformLogs|Registros de plataforma de App Service Environment|


## <a name="microsoftwebsites"></a>microsoft.web/sites

|Category|Nombre para mostrar de categoría|
|---|---|
|AppServiceAppLogs|Registros de aplicaciones de App Service|
|AppServiceAuditLogs|Acceso a registros de auditoría|
|AppServiceConsoleLogs|Registros de consola de App Service|
|AppServiceFileAuditLogs|Registros de auditoría de cambios de contenido de sitio|
|AppServiceHTTPLogs|Registros de HTTP|
|FunctionAppLogs|Registros de aplicaciones de función|
|ScanLogs|Registros de examen antivirus|


## <a name="microsoftwebsitesslots"></a>microsoft.web/sites/slots


|Category|Nombre para mostrar de categoría|
|---|---|
|AppServiceAppLogs|Registros de aplicaciones de App Service|
|AppServiceAuditLogs|Acceso a registros de auditoría|
|AppServiceConsoleLogs|Registros de consola de App Service|
|AppServiceFileAuditLogs|Registros de auditoría de cambios de contenido de sitio|
|AppServiceHTTPLogs|Registros de HTTP|
|FunctionAppLogs|Registros de aplicaciones de función|
|ScanLogs|Registros de examen antivirus|


## <a name="next-steps"></a>Pasos siguientes

* [Más información sobre los registros de recurso](./platform-logs-overview.md)
* [Transmisión de registros de recursos a **Event Hubs**](./resource-logs.md#send-to-azure-event-hubs)
* [Cambio de la configuración de diagnóstico del registro de recursos con la API REST de Azure Monitor](/rest/api/monitor/diagnosticsettings)
* [Análisis de registros desde Azure Storage con Log Analytics](./resource-logs.md#send-to-log-analytics-workspace)

