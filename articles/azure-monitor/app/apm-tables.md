---
title: Esquema de recursos basado en el área de trabajo de Application Insights de Azure Monitor
description: Obtenga información sobre la nueva estructura de tabla y el esquema parar los nuevos recursos basados en áreas de trabajo de Application Insights de Azure Monitor.
ms.topic: conceptual
author: mrbullwinkle
ms.author: mbullwin
ms.date: 05/09/2020
ms.openlocfilehash: 1d7275c928b4d25e200a3a8d3d690c7575c056e7
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87323186"
---
# <a name="workspace-based-resource-changes-preview"></a>Cambios en los recursos basados en áreas de trabajo (versión preliminar)

Antes de la introducción de los [recursos de Application Insights basados en el área de trabajo](create-workspace-resource.md), los datos de Application Insights se almacenaban en un lugar diferente de otros datos de registro en Azure Monitor. Ambos se basan en Azure Data Explorer y usan el mismo lenguaje de consulta de Kusto (KQL). Esto se describe en los [registros de Azure Monitor](../platform/data-platform-logs.md).

Con los recursos de Application Insights basados en áreas de trabajo, los datos se almacenan en un área de trabajo de Log Analytics con otros datos de supervisión y de aplicaciones. Esto simplifica la configuración, ya que le permite analizar con más facilidad los datos en varias soluciones y aprovechar las capacidades de las áreas de trabajo.

## <a name="table-structure"></a>Estructura de tabla

| Nombre de tabla heredada | Nombre de la nueva tabla | Descripción |
|:---|:---|:---|
| availabilityResults | AppAvailabilityResults |  Datos de resumen de las pruebas de disponibilidad.|
| browserTimings | AppBrowserTimings | Datos sobre el rendimiento del cliente, como el tiempo que se tardan en procesar los datos entrantes.|
| dependencies | AppDependencies | Llamadas desde la aplicación a otros componentes (incluidos componentes externos) registradas a través de TrackDependency(), por ejemplo, llamadas a la API de REST, a una base de datos o a un sistema de archivos.  |
| customEvents | AppEvents | Eventos personalizados creados por la aplicación. |
| customMetrics | AppMetrics | Métricas personalizadas creadas por la aplicación. |
| pageViews | AppPageViews| Datos sobre cada vista del sitio web con información del explorador. |
| performanceCounters | AppPerformanceCounters | Medidas de rendimiento de los recursos de proceso que respaldan la aplicación, por ejemplo, contadores de rendimiento de Windows. |
| Solicitudes | AppRequests | Solicitudes recibidas por su aplicación. Por ejemplo, se registra un registro de solicitud independiente para cada solicitud HTTP que recibe la aplicación web.  |
| exceptions | AppSystemEvents | Excepciones producidas por el tiempo de ejecución de la aplicación, captura las excepciones del lado servidor y cliente (exploradores). |
| traces | AppTraces | Registros detallados (seguimientos) emitidos a través de código de aplicación/plataformas de registro, registrados a través de TrackTrace(). |

## <a name="table-schemas"></a>Esquemas de tabla

En las secciones siguientes se muestra la asignación entre los nombres de propiedades clásicas y los nuevos nombres de propiedad de Application Insights basados en áreas de trabajo.  Utilice esta información para convertir las consultas mediante tablas heredadas.

La mayoría de las columnas tienen el mismo nombre con diferentes mayúsculas y minúsculas. Dado que KQL distingue entre mayúsculas y minúsculas, deberá cambiar cada nombre de columna junto con los nombres de tabla en las consultas existentes. Se resaltan las columnas con cambios además de mayúsculas. Todavía puede usar las consultas clásicas de Application Insights en el panel de **registros** del recurso de Application Insights, incluso si se trata de un recurso basado en el área de trabajo. Los nuevos nombres de propiedad son necesarios cuando se realizan consultas desde el contexto de la experiencia del área de trabajo de Log Analytics.

### <a name="appavailabilityresults"></a>AppAvailabilityResults

Tabla heredada: disponibilidad

|ApplicationInsights|Tipo|LogAnalytics|Tipo|
|:---|:---|:---|:---|
|appId|string|\_ResourceGUID|string|
|application_Version|string|AppVersion|string|
|appName|string|\_ResourceId|string|
|client_Browser|string|ClientBrowser|string|
|client_city|string|ClientCity|string|
|client_CountryOrRegion|string|ClientCountryOrRegion|string|
|client_IP|string|ClientIP|string|
|client_Model|string|ClientModel|string|
|client_OS|string|ClientOS|string|
|client_StateOrProvince|string|ClientStateOrProvince|string|
|client_Type|string|ClientType|string|
|cloud_RoleInstance|string|AppRoleInstance|string|
|cloud_RoleName|string|AppRoleName|string|
|customDimensions|dinámico|Propiedades|Dinámica|
|customMeasurements|dinámico|Medidas|Dinámica|
|duration|real|DurationMs|real|
|`id`|string|`Id`|string|
|iKey|string|IKey|string|
|itemCount|int|ItemCount|int|
|itemId|string|\_ItemId|string|
|itemType|string|Tipo|String|
|ubicación|string|Location|string|
|message|string|Message|string|
|name|string|Nombre|string|
|operation_Id|string|OperationId|string|
|operation_Name|string|OperationName|string|
|operation_ParentId|string|OperationParentId|string|
|operation_SyntheticSource|string|OperationSyntheticSource|string|
|performanceBucket|string|PerformanceBucket|string|
|sdkVersion|string|SdkVersion|string|
|session_Id|string|SessionId|string|
|tamaño|real|Size|real|
|success|string|Correcto|Bool|
|timestamp|datetime|TimeGenerated|datetime|
|user_AccountId|string|UserAccountId|string|
|user_AuthenticatedId|string|UserAuthenticatedId|string|
|user_Id|string|UserId|string|

### <a name="appbrowsertimings"></a>AppBrowserTimings

Tabla heredada: browserTimings

|ApplicationInsights|Tipo|LogAnalytics|Tipo|
|:---|:---|:---|:---|
|appId|string|\_ResourceGUID|string|
|application_Version|string|AppVersion|string|
|appName|string|\_ResourceId|string|
|client_Browser|string|ClientBrowser|string|
|client_city|string|ClientCity|string|
|client_CountryOrRegion|string|ClientCountryOrRegion|string|
|client_IP|string|ClientIP|string|
|client_Model|string|ClientModel|string|
|client_OS|string|ClientOS|string|
|client_StateOrProvince|string|ClientStateOrProvince|string|
|client_Type|string|ClientType|string|
|cloud_RoleInstance|string|AppRoleInstance|string|
|cloud_RoleName|string|AppRoleName|string|
|customDimensions|dinámico|Propiedades|Dinámica|
|customMeasurements|dinámico|Medidas|Dinámica|
|iKey|string|IKey|string|
|itemCount|int|ItemCount|int|
|itemId|string|\_ItemId|string|
|itemType|string|Tipo|string|
|name|string|Nombre|datetime|
|networkDuration|real|NetworkDurationMs|real|
|operation_Id|string|OperationId|string|
|operation_Name|string|OperationName|string|
|operation_ParentId|string|OperationParentId|string|
|operation_SyntheticSource|string|OperationSyntheticSource|string|
|performanceBucket|string|PerformanceBucket|string|
|processingDuration|real|ProcessingDurationMs|real|
|receiveDuration|real|ReceiveDurationMs|real|
|sdkVersion|string|SdkVersion|string|
|sendDuration|real|SendDurationMs|real|
|session_Id|string|SessionId|string|
|timestamp|datetime|TimeGenerated|datetime|
|totalDuration|real|TotalDurationMs|real|
|url|string|Url|string|
|user_AccountId|string|UserAccountId|string|
|user_AuthenticatedId|string|UserAuthenticatedId|string|
|user_Id|string|UserId|string|

### <a name="appdependencies"></a>AppDependencies

Tabla heredada: dependencias

|ApplicationInsights|Tipo|LogAnalytics|Tipo|
|:---|:---|:---|:---|
|appId|string|\_ResourceGUID|string|
|application_Version|string|AppVersion|string|
|appName|string|\_ResourceId|string|
|client_Browser|string|ClientBrowser|string|
|client_city|string|ClientCity|string|
|client_CountryOrRegion|string|ClientCountryOrRegion|string|
|client_IP|string|ClientIP|string|
|client_Model|string|ClientModel|string|
|client_OS|string|ClientOS|string|
|client_StateOrProvince|string|ClientStateOrProvince|string|
|client_Type|string|ClientType|string|
|cloud_RoleInstance|string|AppRoleInstance|string|
|cloud_RoleName|string|AppRoleName|string|
|customDimensions|dinámico|Propiedades|Dinámica|
|customMeasurements|dinámico|Medidas|Dinámica|
|datos|string|data|string|
|duration|real|DurationMs|real|
|`id`|string|`Id`|string|
|iKey|string|IKey|string|
|itemCount|int|ItemCount|int|
|itemId|string|\_ItemId|string|
|itemType|string|Tipo|String|
|name|string|Nombre|string|
|operation_Id|string|OperationId|string|
|operation_Name|string|OperationName|string|
|operation_ParentId|string|OperationParentId|string|
|operation_SyntheticSource|string|OperationSyntheticSource|string|
|performanceBucket|string|PerformanceBucket|string|
|resultCode|string|ResultCode|string|
|sdkVersion|string|SdkVersion|string|
|session_Id|string|SessionId|string|
|success|string|Correcto|Bool|
|Destino|string|Destino|string|
|timestamp|datetime|TimeGenerated|datetime|
|type|string|DependencyType|string|
|user_AccountId|string|UserAccountId|string|
|user_AuthenticatedId|string|UserAuthenticatedId|string|
|user_Id|string|UserId|string|

### <a name="appevents"></a>AppEvents

Tabla heredada: customEvents

|ApplicationInsights|Tipo|LogAnalytics|Tipo|
|:---|:---|:---|:---|
|appId|string|\_ResourceGUID|string|
|application_Version|string|AppVersion|string|
|appName|string|\_ResourceId|string|
|client_Browser|string|ClientBrowser|string|
|client_city|string|ClientCity|string|
|client_CountryOrRegion|string|ClientCountryOrRegion|string|
|client_IP|string|ClientIP|string|
|client_Model|string|ClientModel|string|
|client_OS|string|ClientOS|string|
|client_StateOrProvince|string|ClientStateOrProvince|string|
|client_Type|string|ClientType|string|
|cloud_RoleInstance|string|AppRoleInstance|string|
|cloud_RoleName|string|AppRoleName|string|
|customDimensions|dinámico|Propiedades|Dinámica|
|customMeasurements|dinámico|Medidas|Dinámica|
|iKey|string|IKey|string|
|itemCount|int|ItemCount|int|
|itemId|string|\_ItemId|string|
|itemType|string|Tipo|string|
|name|string|Nombre|string|
|operation_Id|string|OperationId|string|
|operation_Name|string|OperationName|string|
|operation_ParentId|string|OperationParentId|string|
|operation_SyntheticSource|string|OperationSyntheticSource|string|
|sdkVersion|string|SdkVersion|string|
|session_Id|string|SessionId|string|
|timestamp|datetime|TimeGenerated|datetime|
|user_AccountId|string|UserAccountId|string|
|user_AuthenticatedId|string|UserAuthenticatedId|string|
|user_Id|string|UserId|string|

### <a name="appmetrics"></a>AppMetrics

Tabla heredada: customMetrics

|ApplicationInsights|Tipo|LogAnalytics|Tipo|
|:---|:---|:---|:---|
|appId|string|\_ResourceGUID|string|
|application_Version|string|AppVersion|string|
|appName|string|\_ResourceId|string|
|client_Browser|string|ClientBrowser|string|
|client_city|string|ClientCity|string|
|client_CountryOrRegion|string|ClientCountryOrRegion|string|
|client_IP|string|ClientIP|string|
|client_Model|string|ClientModel|string|
|client_OS|string|ClientOS|string|
|client_StateOrProvince|string|ClientStateOrProvince|string|
|client_Type|string|ClientType|string|
|cloud_RoleInstance|string|AppRoleInstance|string|
|cloud_RoleName|string|AppRoleName|string|
|customDimensions|dinámico|Propiedades|Dinámica|
|iKey|string|IKey|string|
|itemId|string|\_ItemId|string|
|itemType|string|Tipo|string|
|name|string|Nombre|string|
|operation_Id|string|OperationId|string|
|operation_Name|string|OperationName|string|
|operation_ParentId|string|OperationParentId|string|
|operation_SyntheticSource|string|OperationSyntheticSource|string|
|sdkVersion|string|SdkVersion|string|
|session_Id|string|SessionId|string|
|timestamp|datetime|TimeGenerated|datetime|
|user_AccountId|string|UserAccountId|string|
|user_AuthenticatedId|string|UserAuthenticatedId|string|
|user_Id|string|UserId|string|
|value|real|(se ha quitado)||
|valueCount|int|ValueCount|int|
|valueMax|real|ValueMax|real|
|valueMin|real|ValueMin|real|
|valueStdDev|real|ValueStdDev|real|
|valueSum|real|ValueSum|real|

### <a name="apppageviews"></a>AppPageViews

Tabla heredada: pageViews

|ApplicationInsights|Tipo|LogAnalytics|Tipo|
|:---|:---|:---|:---|
|appId|string|\_ResourceGUID|string|
|application_Version|string|AppVersion|string|
|appName|string|\_ResourceId|string|
|client_Browser|string|ClientBrowser|string|
|client_city|string|ClientCity|string|
|client_CountryOrRegion|string|ClientCountryOrRegion|string|
|client_IP|string|ClientIP|string|
|client_Model|string|ClientModel|string|
|client_OS|string|ClientOS|string|
|client_StateOrProvince|string|ClientStateOrProvince|string|
|client_Type|string|ClientType|string|
|cloud_RoleInstance|string|AppRoleInstance|string|
|cloud_RoleName|string|AppRoleName|string|
|customDimensions|dinámico|Propiedades|Dinámica|
|customMeasurements|dinámico|Medidas|Dinámica|
|duration|real|DurationMs|real|
|`id`|string|`Id`|string|
|iKey|string|IKey|string|
|itemCount|int|ItemCount|int|
|itemId|string|\_ItemId|string|
|itemType|string|Tipo|String|
|name|string|Nombre|string|
|operation_Id|string|OperationId|string|
|operation_Name|string|OperationName|string|
|operation_ParentId|string|OperationParentId|string|
|operation_SyntheticSource|string|OperationSyntheticSource|string|
|performanceBucket|string|PerformanceBucket|string|
|sdkVersion|string|SdkVersion|string|
|session_Id|string|SessionId|string|
|timestamp|datetime|TimeGenerated|datetime|
|url|string|Url|string|
|user_AccountId|string|UserAccountId|string|
|user_AuthenticatedId|string|UserAuthenticatedId|string|
|user_Id|string|UserId|string|

### <a name="appperformancecounters"></a>AppPerformanceCounters

Tabla heredada: performanceCounters

|ApplicationInsights|Tipo|LogAnalytics|Tipo|
|:---|:---|:---|:---|
|appId|string|\_ResourceGUID|string|
|application_Version|string|AppVersion|string|
|appName|string|\_ResourceId|string|
|category|string|Category|string|
|client_Browser|string|ClientBrowser|string|
|client_city|string|ClientCity|string|
|client_CountryOrRegion|string|ClientCountryOrRegion|string|
|client_IP|string|ClientIP|string|
|client_Model|string|ClientModel|string|
|client_OS|string|ClientOS|string|
|client_StateOrProvince|string|ClientStateOrProvince|string|
|client_Type|string|ClientType|string|
|cloud_RoleInstance|string|AppRoleInstance|string|
|cloud_RoleName|string|AppRoleName|string|
|counter|string|(se ha quitado)||
|customDimensions|dinámico|Propiedades|Dinámica|
|iKey|string|IKey|string|
|instance|string|Instancia|string|
|itemId|string|\_ItemId|string|
|itemType|string|Tipo|string|
|name|string|Nombre|string|
|operation_Id|string|OperationId|string|
|operation_Name|string|OperationName|string|
|operation_ParentId|string|OperationParentId|string|
|operation_SyntheticSource|string|OperationSyntheticSource|string|
|sdkVersion|string|SdkVersion|string|
|session_Id|string|SessionId|string|
|timestamp|datetime|TimeGenerated|datetime|
|user_AccountId|string|UserAccountId|string|
|user_AuthenticatedId|string|UserAuthenticatedId|string|
|user_Id|string|UserId|string|
|value|real|Value|real|

### <a name="apprequests"></a>AppRequests

Tabla heredada: solicitudes

|ApplicationInsights|Tipo|LogAnalytics|Tipo|
|:---|:---|:---|:---|
|appId|string|\_ResourceGUID|string|
|application_Version|string|AppVersion|string|
|appName|string|\_ResourceId|string|
|client_Browser|string|ClientBrowser|string|
|client_city|string|ClientCity|string|
|client_CountryOrRegion|string|ClientCountryOrRegion|string|
|client_IP|string|ClientIP|string|
|client_Model|string|ClientModel|string|
|client_OS|string|ClientOS|string|
|client_StateOrProvince|string|ClientStateOrProvince|string|
|client_Type|string|ClientType|string|
|cloud_RoleInstance|string|AppRoleInstance|string|
|cloud_RoleName|string|AppRoleName|string|
|customDimensions|dinámico|Propiedades|Dinámica|
|customMeasurements|dinámico|Medidas|Dinámica|
|duration|real|DurationMs|Real|
|`id`|string|`Id`|String|
|iKey|string|IKey|string|
|itemCount|int|ItemCount|int|
|itemId|string|\_ItemId|string|
|itemType|string|Tipo|String|
|name|string|Nombre|String|
|operation_Id|string|OperationId|string|
|operation_Name|string|OperationName|string|
|operation_ParentId|string|OperationParentId|string|
|operation_SyntheticSource|string|OperationSyntheticSource|string|
|performanceBucket|string|PerformanceBucket|String|
|resultCode|string|ResultCode|String|
|sdkVersion|string|SdkVersion|string|
|session_Id|string|SessionId|string|
|source|string|Source|String|
|success|string|Correcto|Bool|
|timestamp|datetime|TimeGenerated|datetime|
|url|string|Url|String|
|user_AccountId|string|UserAccountId|string|
|user_AuthenticatedId|string|UserAuthenticatedId|string|
|user_Id|string|UserId|string|

### <a name="appsystemevents"></a>AppSystemEvents

Tabla heredada: excepciones

|ApplicationInsights|Tipo|LogAnalytics|Tipo|
|:---|:---|:---|:---|
|appId|string|\_ResourceGUID|string|
|application_Version|string|AppVersion|string|
|appName|string|\_ResourceId|string|
|ensamblado|string|Assembly|string|
|client_Browser|string|ClientBrowser|string|
|client_city|string|ClientCity|string|
|client_CountryOrRegion|string|ClientCountryOrRegion|string|
|client_IP|string|ClientIP|string|
|client_Model|string|ClientModel|string|
|client_OS|string|ClientOS|string|
|client_StateOrProvince|string|ClientStateOrProvince|string|
|client_Type|string|ClientType|string|
|cloud_RoleInstance|string|AppRoleInstance|string|
|cloud_RoleName|string|AppRoleName|string|
|customDimensions|dinámico|Propiedades|dinámico|
|customMeasurements|dinámico|Medidas|dinámico|
|detalles|dinámico|Detalles|dinámico|
|handledAt|string|HandledAt|string|
|iKey|string|IKey|string|
|innermostAssembly|string|InnermostAssembly|string|
|innermostMessage|string|InnermostMessage|string|
|innermostMethod|string|InnermostMethod|string|
|innermostType|string|InnermostType|string|
|itemCount|int|ItemCount|int|
|itemId|string|\_ItemId|string|
|itemType|string|Tipo|string|
|message|string|Message|string|
|method|string|Método|string|
|operation_Id|string|OperationId|string|
|operation_Name|string|OperationName|string|
|operation_ParentId|string|OperationParentId|string|
|operation_SyntheticSource|string|OperationSyntheticSource|string|
|outerAssembly|string|OuterAssembly|string|
|outerMessage|string|OuterMessage|string|
|outerMethod|string|OuterMethod|string|
|outerType|string|OuterType|string|
|problemId|string|ProblemId|string|
|sdkVersion|string|SdkVersion|string|
|session_Id|string|SessionId|string|
|severityLevel|int|SeverityLevel|int|
|timestamp|datetime|TimeGenerated|datetime|
|type|string|ExceptionType|string|
|user_AccountId|string|UserAccountId|string|
|user_AuthenticatedId|string|UserAuthenticatedId|string|
|user_Id|string|UserId|string|

### <a name="apptraces"></a>AppTraces

Tabla heredada: seguimientos

|ApplicationInsights|Tipo|LogAnalytics|Tipo|
|:---|:---|:---|:---|
|appId|string|\_ResourceGUID|string|
|application_Version|string|AppVersion|string|
|appName|string|\_ResourceId|string|
|client_Browser|string|ClientBrowser|string|
|client_city|string|ClientCity|string|
|client_CountryOrRegion|string|ClientCountryOrRegion|string|
|client_IP|string|ClientIP|string|
|client_Model|string|ClientModel|string|
|client_OS|string|ClientOS|string|
|client_StateOrProvince|string|ClientStateOrProvince|string|
|client_Type|string|ClientType|string|
|cloud_RoleInstance|string|AppRoleInstance|string|
|cloud_RoleName|string|AppRoleName|string|
|customDimensions|dinámico|Propiedades|dinámico|
|customMeasurements|dinámico|Medidas|dinámico|
|iKey|string|IKey|string|
|itemCount|int|ItemCount|int|
|itemId|string|\_ItemId|string|
|itemType|string|Tipo|string|
|message|string|Message|string|
|operation_Id|string|OperationId|string|
|operation_Name|string|OperationName|string|
|operation_ParentId|string|OperationParentId|string|
|operation_SyntheticSource|string|OperationSyntheticSource|string|
|sdkVersion|string|SdkVersion|string|
|session_Id|string|SessionId|string|
|severityLevel|int|SeverityLevel|int|
|timestamp|datetime|TimeGenerated|datetime|
|user_AccountId|string|UserAccountId|string|
|user_AuthenticatedId|string|UserAuthenticatedId|string|
|user_Id|string|UserId|string|

## <a name="next-steps"></a>Pasos siguientes

* [Exploración de métricas](../platform/metrics-charts.md)
* [Escribir consultas de Analytics](../log-query/log-query-overview.md)

