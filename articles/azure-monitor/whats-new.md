---
title: Novedades en la documentación de Azure Monitor
description: Se realizan actualizaciones significativas de la documentación de Azure Monitor todos los meses.
ms.subservice: ''
ms.topic: overview
author: bwren
ms.author: bwren
ms.date: 07/08/2020
ms.openlocfilehash: b36469a89c7bae39ae3c2d62792c2ccde0fcfda9
ms.sourcegitcommit: fbb66a827e67440b9d05049decfb434257e56d2d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/05/2020
ms.locfileid: "87799334"
---
# <a name="whats-new-in-azure-monitor-documentation"></a>Novedades en la documentación de Azure Monitor

En este artículo se enumeran los artículos de Azure Monitor que son nuevos o que se han actualizado de forma significativa. Se actualizará la primera semana de cada mes para que incluya las actualizaciones de artículos del mes anterior.

## <a name="june-2020"></a>Junio de 2020

### <a name="general"></a>General
- [Implementación de Azure Monitor](deploy-scale.md). Nuevo artículo.
- [Clave administrada por el cliente de Azure Monitor](platform/customer-managed-keys.md). Propiedad billingtype actualizada. Comandos PowerShell incorporados.

### <a name="agents"></a>Agentes
- [Introducción al agente de Log Analytics](platform/log-analytics-agent.md). Requisito de Python 2 incorporado.

### <a name="alerts"></a>Alertas
- [Actualización de reglas de alertas o reglas de acción cuando su recurso de destino se mueve a otra región de Azure](platform/alerts-resource-move.md). Nuevo artículo.
- [Solución de problemas en las alertas de métricas de Azure Monitor](platform/alerts-troubleshoot-metric.md). Nuevo artículo.
- [Solución de problemas de alertas de registro en Azure Monitor](platform/alerts-troubleshoot-metric.md). Nuevo artículo.
  
### <a name="application-insights"></a>Application Insights
- [Azure Application Insights para aplicaciones web de JavaScript](app/javascript.md). Actualización de la sección del SDK de JavaScript. Fragmento de código actualizado para informar sobre errores de carga.
- [Configuración de BYOS (Traiga su propio almacenamiento) para Profiler y Snapshot Debugger](app/profiler-bring-your-own-storage.md). Nuevo artículo.
- [Seguimiento de las solicitudes entrantes en Azure Application Insights con OpenCensus para Python](app/opencensus-python-request.md). Registro y configuración actualizadas para OpenCensus.
- [Supervisión de una aplicación web de ASP.NET en vivo con Azure Application Insights](app/monitor-performance-live-website-now.md): se ha actualizado la fecha de entrada en desuso de Status Monitor v1.
- [Supervisión de servicios de node. js con Azure Application Insights](app/nodejs.md). Varias actualizaciones, incluida la migración desde versiones anteriores y la configuración del SDK
- [Supervisión de aplicaciones de Python con Azure Monitor (versión preliminar)](app/opencensus-python.md). Incorporación de una sección sobre la configuración de los exportadores de Azure Monitor.
- [Supervisión de las aplicaciones sin cambios de código: instrumentación automática para Azure Monitor Application Insights](app/codeless-overview.md). Nuevo artículo.
- [Solución de problemas de errores de carga del SDK para aplicaciones web de JavaScript](app/javascript-sdk-load-failure.md). Nuevo artículo.

### <a name="containers"></a>Contenedores
- [Cómo detener la supervisión del clúster híbrido de Kubernetes](insights/container-insights-optout-hybrid.md). Sección agregada para Kubernetes habilitado para Arc.
- [Configuración de un clúster de Kubernetes habilitado para Azure Arc con Azure Monitor para contenedores](insights/container-insights-enable-arc-enabled-clusters.md). Nuevo artículo.
- [Configuración de la versión 4.x de Red Hat OpenShift en Azure con Azure Monitor para contenedores](insights/container-insights-azure-redhat4-setup.md).Requisitos previos actualizados.
- [Configuración de datos en directo (versión preliminar) de Azure Monitor para contenedores](insights/container-insights-livedata-setup.md). Eliminación de nota sobre la característica que no está disponible en Azure US Government.

### <a name="insights"></a>Información detallada
- [Preguntas más frecuentes: Solución Network Performance Monitor en Azure](insights/network-performance-monitor-faq.md). Incorporación de preguntas más frecuentes sobre Supervisión de ExpressRoute.

### <a name="logs"></a>Registros
- [Eliminación y recuperación de un área de trabajo de Azure Log Analytics](platform/delete-workspace.md). Incorporación de un comando de PowerShell. Actualización de Solución de problemas.
- [Administración de áreas de trabajo de Log Analytics en Azure Monitor](platform/manage-access.md). Incorporación de ejemplo para tablas no permitidas en la sección RBAC.
- [Administración del uso y los costos con los registros de Azure Monitor](platform/manage-cost-storage.md). Detalles adicionales sobre el cálculo del tamaño de los datos. Se actualizó la configuración de alertas de volumen de datos. Detalles sobre los datos de seguridad recopilados por Azure Sentinel. Aclaración sobre el límite de datos.
- [Uso de Azure Monitor Logs con Azure Logic Apps y Power Automate](platform/logicapp-flow-connector.md): se han agregado límites del conector.

### <a name="metrics"></a>Métricas
- [Métricas compatibles de Azure Monitor por tipo de recurso](platform/metrics-supported.md). Actualización de métricas de SQL Server.


### <a name="platform-logs"></a>Registros de plataforma

- [Ejemplos de plantillas de Resource Manager para la configuración de diagnóstico](samples/resource-manager-diagnostic-settings.md). Corrección de la configuración de diagnóstico del registro de actividad.
- [Envío del registro de actividad de Azure al área de trabajo de Log Analytics mediante Azure Portal](learn/quick-collect-activity-log-portal.md). Nuevo artículo.
- [Envío del registro de actividad de Azure al área de trabajo de Log Analytics mediante una plantilla de Azure Resource Manager](learn/quick-collect-activity-log-arm.md). Nuevo artículo.

#### <a name="new-and-updated-articles-from-restructure-and-consolidation-of-platform-log-content"></a>Artículos nuevos y actualizados sobre reestructuración y consolidación del contenido de registros de plataforma
- [Archivado de registros de recursos de Azure en la cuenta de almacenamiento](./platform/resource-logs.md#send-to-azure-storage)
- [Esquema de eventos del registro de actividad de Azure](platform/activity-log-schema.md)
- [Registro de actividad de Azure](platform/activity-log.md)
- [Ejemplos de la CLI de Azure Monitor](samples/cli-samples.md)
- [Ejemplos de PowerShell de Azure Monitor](samples/powershell-samples.md)
- [Tutorial sobre la API de REST de supervisión de Azure](platform/rest-api-walkthrough.md)
- [Servicios y esquemas admitidos de los registros de recursos de Azure](./platform/resource-logs-schema.md)
- [Registros de recursos de Azure](platform/resource-logs.md)
- [Recopilación y análisis del registro de actividad de Azure en Azure Monitor](./platform/activity-log.md)
- [Recopilación de registros de recursos de Azure en el área de trabajo de Log Analytics](./platform/resource-logs.md#send-to-log-analytics-workspace)
- [Creación de una configuración de diagnóstico para enviar registros de plataforma y métricas a diferentes destinos](platform/diagnostic-settings.md)
- [Exportación del registro de actividad de Azure](./platform/activity-log.md#legacy-collection-methods)
- [Introducción a los registros de plataforma Azure](platform/platform-logs-overview.md)
- [Transmisión de registros de plataforma de Azure a un centro de eventos](./platform/resource-logs.md#send-to-azure-event-hubs)
- [Visualización de eventos del registro de actividad de Azure en Azure Monitor](./platform/activity-log.md#view-the-activity-log)

### <a name="virtual-machines"></a>Máquinas virtuales
- [Habilitación de Azure Monitor para VM en Azure Portal](insights/vminsights-enable-single-vm.md). Actualización para incluir Azure Arc.
- [Información general sobre la habilitación de Azure Monitor para VM](insights/vminsights-enable-overview.md). Actualización para incluir Azure Arc.
- [¿Qué es Azure Monitor para VM?](insights/vminsights-overview.md) Actualización para incluir Azure Arc.


### <a name="visualizations"></a>Visualizaciones
- [Orígenes de datos de libros de Azure Monitor](platform/workbooks-data-sources.md). Incorporación de las secciones de Alertas y Punto de conexión personalizado.
- [Solución de problemas de conclusiones basadas en libros de Azure Monitor](insights/troubleshoot-workbooks.md). Nuevo artículo.
- [Actualización de las visualizaciones del panel de Log Analytics](log-query/dashboard-upgrade.md). Nuevo artículo.



## <a name="may-2020"></a>Mayo de 2020

### <a name="general"></a>General

- [Preguntas más frecuentes de Azure Monitor](faq.md): se ha agregado una sección de métricas.
- [Clave administrada por el cliente de Azure Monitor](platform/customer-managed-keys.md): varios cambios en la preparación para la disponibilidad general.
- [Definiciones de directivas integradas para Azure Monitor](samples/policy-samples.md): nuevo artículo.
- [Cuentas de almacenamiento de propiedad del cliente para la ingesta de registros](platform/private-storage.md): nuevo artículo.
- [Administrar el uso y los costos con los registros de Azure Monitor](platform/manage-cost-storage.md): se ha agregado la facturación proporcional de los clústeres.
- [Uso de Azure Private Link para conectar redes a Azure Monitor de forma segura](platform/private-link-security.md): nuevo artículo.


#### <a name="new-resource-manager-template-samples"></a>Nuevos ejemplos de plantillas de Resource Manager 
- [Ejemplos de plantillas de Resource Manager para Azure Monitor](samples/resource-manager-samples.md)
- [Ejemplos de plantillas de Resource Manager para grupos de acciones](samples/resource-manager-action-groups.md)
- [Ejemplos de plantillas de Resource Manager para agentes](samples/resource-manager-agent.md)
- [Ejemplos de plantillas de Resource Manager para Azure Monitor para contenedores](samples/resource-manager-container-insights.md)
- [Ejemplos de plantillas de Resource Manager para Azure Monitor para VM](samples/resource-manager-vminsights.md)
- [Ejemplos de plantillas de Resource Manager para la configuración de diagnóstico](samples/resource-manager-diagnostic-settings.md)
- [Ejemplos de plantillas de Resource Manager para áreas de trabajo de Log Analytics](samples/resource-manager-workspace.md)
- [Ejemplos de plantillas de Resource Manager para consultas de registros](samples/resource-manager-log-queries.md)
- [Ejemplos de plantillas de Resource Manager para reglas de alertas de consultas de registros](samples/resource-manager-alerts-log.md)
- [Ejemplos de plantillas de Resource Manager para reglas de alertas de métricas](samples/resource-manager-alerts-metric.md)
- [Ejemplos de plantillas de Resource Manager para libros](samples/resource-manager-workbooks.md)

### <a name="agents"></a>Agentes
- [Instalación y configuración de la extensión de Diagnósticos de Azure para Windows (WAD)](platform/diagnostics-extension-windows-install.md): detalle agregado sobre la configuración de diagnóstico.
- [Información general del agente de Log Analytics](platform/log-analytics-agent.md): se han agregado versiones de Linux compatibles.

### <a name="application-insights"></a>Application Insights

- [Supervisión de aplicaciones que se ejecutan en Azure Functions con Application Insights: Azure Monitor](app/monitor-functions.md): nuevo artículo.
- [Supervisión de los servicios de Node.js con Azure Application Insights](app/nodejs.md): actualizaciones generales, incluida la nueva sección sobre la migración desde versiones anteriores.
- [Direcciones IP usadas por Application Insights y Log Analytics](app/ip-addresses.md): se han agregado direcciones IP para webhooks y US Government.
- [Supervisión de aplicaciones en Azure Kubernetes Service (AKS) con Application Insights: Azure Monitor](app/kubernetes-codeless.md): nuevo artículo.
- [Solución de problemas cuando no hay datos: Application Insights para .NET](app/asp-net-troubleshoot-no-data.md): se ha agregado una sección sobre la recopilación de registros con dotnet-trace.
- [Uso de Application Change Analysis en Azure Monitor para detectar problemas de la aplicación web](app/change-analysis.md): varias actualizaciones en la característica Change Analysis.

#### <a name="new-and-updated-articles-for-preview-of-workspace-based-applications"></a>Artículos nuevos y actualizados para la versión preliminar de aplicaciones basadas en el área de trabajo
- [Esquema de recursos basado en el área de trabajo de Application Insights de Azure Monitor](app/apm-tables.md)
- [Creación de nuevos recursos basados en área de trabajo de Application Insights de Azure Monitor](app/create-workspace-resource.md)
- [Expresión app() en consultas de registro de Azure Monitor](log-query/app-expression.md)
- [Ámbito de la consulta de registro en Log Analytics de Azure Monitor](log-query/scope.md)
- [Consulta entre recursos con Azure Monitor](log-query/cross-workspace-query.md)
- [Propiedades estándar de los registros de Azure Monitor](platform/log-standard-properties.md)
- [Estructura de registros de Azure Monitor](log-query/logs-structure.md)





### <a name="containers"></a>Contenedores
- [Cómo habilitar Azure Monitor para contenedores](insights/container-insights-onboard.md): se ha actualizado la tabla de configuración del firewall.
- [Cómo actualizar Azure Monitor para contenedores para habilitar métricas](insights/container-insights-update-metrics.md): actualización para usar identidades administradas para recopilar métricas.
- [Costos de supervisión por Azure Monitor para contenedores](insights/container-insights-cost.md): nuevo artículo.
- [Configuración de Azure Monitor para datos en directo de contenedores (versión preliminar)](insights/container-insights-livedata-setup.md): soporte técnico para el nuevo enlace de rol de clúster.

### <a name="insights"></a>Información detallada
- [Azure Monitor para Azure Cache for Redis (versión preliminar)](insights/redis-cache-insights-overview.md): nuevo artículo.
- [Supervisión de Key Vault con Azure Monitor para Key Vault (versión preliminar)](./insights/key-vault-insights-overview.md): nuevo artículo.

### <a name="logs"></a>Registros
- [Creación y configuración de Log Analytics con PowerShell](platform/powershell-workspace-configuration.md): se ha agregado una sección de solución de problemas.
- [Creación de un área de trabajo de Log Analytics en Azure Portal](learn/quick-create-workspace.md): se ha agregado una sección de solución de problemas.
- [Creación de un área de trabajo de Log Analytics mediante la CLI de Azure](learn/quick-create-workspace-cli.md): se ha agregado una sección de solución de problemas.
- [Eliminación y recuperación de un área de trabajo de Azure Log Analytics](platform/delete-workspace.md): se ha actualizado la información sobre la recuperación de un área de trabajo eliminada.
- [Funciones de las consultas de registros de Azure Monitor](log-query/functions.md): se ha quitado la nota sobre las funciones que no contienen otras funciones.
- [Estructura de registros de Azure Monitor](log-query/logs-structure.md): se han aclarado las descripciones de propiedades de la tabla de Application Insights.
- [Uso de Azure Monitor Logs con Azure Logic Apps y Power Automate](platform/logicapp-flow-connector.md): se ha agregado una sección de límites.
- [Uso de PowerShell para crear y configurar un área de trabajo de Log Analytics](platform/powershell-workspace-configuration.md): se ha agregado una sección de solución de problemas.


### <a name="metrics"></a>Métricas
- [Métricas compatibles de Azure Monitor por tipo de recurso](platform/metrics-supported.md): se han aclarado las métricas de invitado y el enrutamiento de métricas. 

### <a name="solutions"></a>Soluciones
- [Optimización del entorno de Active Directory con Azure Monitor](insights/ad-assessment.md): se ha agregado Windows Server 2019 a las versiones compatibles.
- [Optimización del entorno de SQL Server con Azure Monitor](insights/sql-assessment.md): se ha agregado a las versiones compatibles de SQL Server.


### <a name="virtual-machines"></a>Máquinas virtuales
- [Información general sobre la habilitación de Azure Monitor para VM](insights/vminsights-enable-overview.md): se ha agregado a las versiones compatibles de Ubuntu Server. Se han agregado regiones admitidas para el área de trabajo de Log Analytics.
- [Cómo representar el rendimiento en gráficos con Azure Monitor para VM](insights/vminsights-performance.md): se ha agregado una sección de limitaciones para las métricas no disponibles.

### <a name="visualizations"></a>Visualizaciones
- [Libros de Azure Monitor y plantillas de Azure Resource Manager](platform/workbooks-automate.md): se ha agregado una actualización de Resource Manager para implementar una plantilla de libro.
- [Grupos en los libros de Azure Monitor](platform/workbooks-groups.md): nuevo artículo.
- [Libros de Azure Monitor: transformación de datos JSON con JSONPath](platform/workbooks-jsonpath.md): nuevo artículo.


## <a name="april-2020"></a>Abril de 2020

### <a name="general"></a>General

- [Configuración de la clave administrada por el cliente de Azure Monitor](platform/customer-managed-keys.md): se ha agregado una sección sobre operaciones asincrónicas.
- [Administración del acceso a los datos de registro y las áreas de trabajo en Azure Monitor](platform/manage-access.md): se ha actualizado la sección de registros personalizados.

### <a name="alerts"></a>Alertas

- [Reglas de acción (versión preliminar)](platform/alerts-action-rules.md): se ha agregado un vídeo.
- [Información general sobre las alertas en Microsoft Azure](platform/alerts-overview.md): se ha agregado un vídeo.

### <a name="application-insights"></a>Application Insights

- [Mapa de aplicación: Evaluación de prioridades de las aplicaciones distribuidas](app/app-map.md): se ha agregado la configuración de nombres de roles de nube para el agente de Java.
- [Referencia de la API del agente de .NET de Azure Application Insights](app/status-monitor-v2-api-reference.md): se ha consolidado la referencia de la API.
- [Direcciones IP que emplean Application Insights y Log Analytics](app/ip-addresses.md): se han actualizado las direcciones IP para App Insights y las API de Log Analytics, webhooks de grupo de acciones, Azure US Government.
- [Supervisión de aplicaciones de Java en cualquier sitio](app/java-standalone-config.md): nuevo artículo.
- [Supervisión de aplicaciones de Java en cualquier entorno](app/java-in-process-agent.md): nuevo artículo.
- [Supervisión de aplicaciones de Java que se ejecutan en cualquier entorno](app/java-standalone-arguments.md): nuevo artículo.
- [Supervisión de aplicaciones de Java que se ejecutan en el entorno local](app/java-on-premises.md): nuevo artículo.
- [Procedimiento para quitar Application Insights en Visual Studio](app/remove-application-insights.md): nuevo artículo.
- [Muestreo en Application Insights](app/sampling.md): se ha corregido el ejemplo de frecuencia fija en Python.

### <a name="containers"></a>Contenedores

- [Configuración de la versión 4.x de Red Hat OpenShift en Azure con Azure Monitor para contenedores](insights/container-insights-azure-redhat4-setup.md): nuevo artículo.
- [Cómo corregir manualmente los problemas de sincronización de ServiceNow](platform/itsmc-resync-servicenow.md): nuevo artículo.
- [Cómo detener la supervisión del clúster de Azure y Red Hat OpenShift v4](insights/container-insights-optout-openshift-v4.md): nuevo artículo.
- [Cómo detener la supervisión del clúster de Red Hat OpenShift en Azure v3](insights/container-insights-optout-openshift-v3.md): nuevo artículo.
- [Cómo detener la supervisión del clúster de Kubernetes híbrido](insights/container-insights-optout-hybrid.md): nuevo artículo.

### <a name="insights"></a>Información detallada

- [Supervisión de Azure Key Vault con Azure Monitor para Key Vault (versión preliminar)](insights/key-vault-insights-overview.md): nuevo artículo.

### <a name="logs"></a>Registros

- [Límites de servicio de Azure Monitor](service-limits.md): se ha agregado la limitación de consultas de usuario.
- [Administrar el uso y los costos con los registros de Azure Monitor](platform/manage-cost-storage.md): se ha agregado la facturación de los clústeres de registros. Se ha agregado una consulta de Kusto para que los clientes con el plan de tarifa por nodo heredado puedan determinar si deben cambiar a un nivel de reserva de capacidad o por GB.

### <a name="metrics"></a>Métricas

- [Características avanzadas del Explorador de métricas de Azure](platform/metrics-charts.md): se ha agregado la sección de agregación.

### <a name="workbooks"></a>Workbooks

- [Libros de Azure Monitor y plantillas de Azure Resource Manager](platform/workbooks-automate.md): se ha agregado una plantilla de Resource Manager para implementar una plantilla de libro.

## <a name="march-2020"></a>Marzo de 2020

### <a name="general"></a>General

- [Introducción a Azure Monitor](overview.md): se ha agregado un vídeo de introducción a Azure Monitor.
- [Configuración de la clave administrada por el cliente de Azure Monitor](platform/customer-managed-keys.md): actualizaciones generales.
- [Referencia de datos de Azure Monitor](/azure/azure-monitor/reference/): nuevo sitio.

### <a name="alerts"></a>Alertas

- [Crear, ver y administrar las alertas del registro de actividad mediante Azure Monitor](platform/alerts-activity-log.md): explicación adicional de la plantilla de Resource Manager.
- [Descripción del funcionamiento de las alertas de métricas en Azure Monitor.](platform/alerts-metric-overview.md) : se ha actualizado para la compatibilidad con la Administración Pública.
- [Solución de problemas de alertas y notificaciones de Azure Monitor](platform/alerts-troubleshoot.md): nuevo artículo.

### <a name="application-insights"></a>Application Insights

- [Automatización de Azure Application Insights con PowerShell](app/powershell.md): se han agregado ejemplos de ARMClient.
- [Exportación continua de telemetría desde Application Insights](app/export-telemetry.md):se ha agregado una tabla con detalles de la estructura de exportación.
- [Habilitación de Snapshot Debugger para aplicaciones de .NET en Azure App Service](app/snapshot-debugger-appservice.md): se ha agregado una plantilla de Resource Manager de ejemplo.
- [Administración del uso y los costos de Application Insights](app/pricing.md): se ha agregado información sobre la alerta de límite de datos.
- [Supervisión de aplicaciones de Python con Azure Monitor (versión preliminar)](app/opencensus-python.md): se han agregado métricas estándar.
- [Compatibilidad del mapa de origen con las aplicaciones de JavaScript: Application Insights para Azure Monitor](app/source-map-support.md): nuevo artículo.

### <a name="containers"></a>Contenedores

- [Preguntas más frecuentes sobre Azure Monitor](faq.md): actualización de Azure Monitor para contenedores.
- [Configuración de la supervisión de GPU con Azure Monitor para contenedores](insights/container-insights-gpu-monitoring.md): nuevo artículo.

### <a name="insights"></a>Información detallada

- [Solución de administración de Office 365 en Azure](insights/solution-office-365.md): fecha de desuso actualizada.

### <a name="logs"></a>Registros

- [Optimización de las consultas de registro en Azure Monitor](log-query/query-optimization.md): se ha agregado la condición de CPU para el análisis de XML y JSON.
- [Eliminación y recuperación de un área de trabajo de Azure Log Analytics](platform/delete-workspace.md): se ha agregado información de solución de problemas.
- [Uso de registros de Azure Monitor con Azure Logic Apps y Power Automate](platform/logicapp-flow-connector.md): se ha actualizado con un nuevo conector de Azure Monitor.

### <a name="metrics"></a>Métricas

- [Desuso de métricas de disco en Azure Portal](platform/portal-disk-metrics-deprecation.md): nuevo artículo.
- [Tutorial: Creación de un gráfico de métricas en Azure Monitor](learn/tutorial-metrics-explorer.md): se ha agregado un vídeo.

### <a name="platform-logs"></a>Registros de plataforma

- [Recopilación y análisis del registro de actividad de Azure en Azure Monitor](./platform/activity-log.md): se ha redactado de nuevo para explicar mejor la recopilación del registro de actividad con configuración de diagnóstico.

### <a name="virtual-machines"></a>Máquinas virtuales

- [Supervisión de máquinas virtuales de Azure con Azure Monitor](insights/monitor-vm-azure.md): nuevo artículo.
- [Inicio rápido: Supervisión de máquinas virtuales de Azure con Azure Monitor](learn/quick-monitor-azure-vm.md): se ha actualizado para agregar Azure Monitor para máquinas virtuales.
- [Alertas de Azure Monitor para VM](insights/vminsights-alerts.md): nuevo artículo.
- [Información general sobre la habilitación de Azure Monitor para VM](insights/vminsights-enable-overview.md): actualización de los vínculos de descarga de agentes.

Actualizaciones generales para disponibilidad general de Azure Monitor para VM

- [¿Qué es Azure Monitor para VM?](insights/vminsights-overview.md)
- [Preguntas frecuentes sobre Azure Monitor para VM (GA)](insights/vminsights-ga-release-faq.md) 
- [Habilitación de Azure Monitor para VM mediante Azure Policy](insights/vminsights-enable-at-scale-policy.md) 
- [Cómo representar el rendimiento en gráficos con Azure Monitor para VM](insights/vminsights-performance.md)
- [Cómo consultar registros de Azure Monitor para VM](insights/vminsights-log-search.md)
- [Visualización de las dependencias de aplicación con Azure Monitor para VM](insights/vminsights-maps.md) 

### <a name="visualizations"></a>Visualizaciones

- [Visualización de datos de Azure Monitor](visualizations.md): se ha actualizado para indicar el desuso planeado del Diseñador de vistas.

## <a name="february-2020"></a>Febrero de 2020

### <a name="agents"></a>Agentes

Actualizaciones múltiples como parte de la reescritura del contenido de la extensión de diagnóstico.

- [Introducción a los agentes de supervisión de Azure](platform/agents-overview.md): reestructuración de las tablas para aclarar mejor las características únicas de cada agente.
- [Introducción a las extensiones de Azure Diagnostics](platform/diagnostics-extension-overview.md): se ha reescrito en su totalidad.
- [Uso de Blob Storage para IIS y Table Storage para eventos en Azure Monitor](platform/diagnostics-extension-logs.md): reescritura general con fines de actualización y claridad.
- [Instalación y configuración de Windows Azure Diagnostics Extension (WAD)](platform/diagnostics-extension-windows-install.md): artículo nuevo. 
- [Esquema de Windows Diagnostics Extension](platform/diagnostics-extension-schema-windows.md): se ha reorganizado.
- [Envío de datos desde la extensión de Diagnósticos de Windows Azure a Azure Event Hubs](platform/diagnostics-extension-stream-event-hubs.md): se ha reescrito y actualizado por completo.
- [Almacenamiento y visualización de datos de diagnóstico en Azure Storage](../cloud-services/diagnostics-extension-to-storage.md): se ha reescrito y actualizado por completo.
- [Extensión de máquina virtual de Log Analytics para Windows](../virtual-machines/extensions/oms-windows.md): aclara la relación con el agente de Log Analytics.
- [Extensión de máquina virtual de Azure Monitor para Linux](../virtual-machines/extensions/oms-linux.md): aclara la relación con el agente de Log Analytics.

### <a name="application-insights"></a>Application Insights

- [Cadenas de conexión en Azure Application Insights](app/sdk-connection-string.md): artículo nuevo.

### <a name="insights-and-solutions"></a>Conclusiones y soluciones

#### <a name="azure-monitor-for-containers"></a>Azure Monitor para contenedores

- [Integración de Azure Active Directory con Azure Kubernetes Service](../aks/azure-ad-integration-cli.md): se ha agregado una nota para crear una aplicación cliente que admita clústeres habilitados para RBAC con el fin de admitir Azure Monitor para contenedores.

#### <a name="azure-monitor-for-vms"></a>Azure Monitor para máquinas virtuales

- [Preguntas frecuentes sobre Azure Monitor para VM (disponibilidad general)](insights/vminsights-ga-release-faq.md): cambia el modo en que se almacenan los datos de rendimiento.

#### <a name="office-365"></a>Office 365

- [Solución de administración de Office 365 en Azure](insights/solution-office-365.md): fecha de desuso actualizada.


### <a name="logs"></a>Registros

- [Optimización de las consultas de registro en Azure Monitor](log-query/query-optimization.md): artículo nuevo.
- [Administración del uso y los costos de los registros de Azure Monitor](platform/manage-cost-storage.md): mejora de las consultas de ejemplo para ayudarle a comprender el uso.

### <a name="metrics"></a>Métricas

- [Métricas de plataforma de Azure Monitor exportables a través de la configuración del diagnóstico](platform/metrics-supported-export-diagnostic-settings.md): se ha agregado una sección sobre el cambio en el comportamiento de los valores NULL y cero.

### <a name="visualizations"></a>Visualizaciones

Varios artículos nuevos para la guía de conversión del diseñador de vistas a libros.

- [Guía de transición del diseñador de vistas a los libros de Azure Monitor](platform/view-designer-conversion-overview.md): artículo nuevo.
- [Opciones de conversión del diseñador de vistas a los libros de Azure Monitor](platform/view-designer-conversion-options.md): artículo nuevo.
- [Conversión de elementos del diseñador de vistas a los libros de Azure Monitor](platform/view-designer-conversion-tiles.md): artículo nuevo.
- [Resumen y acceso a la conversión del diseñador de vistas a libros de Azure Monitor](platform/view-designer-conversion-access.md): artículo nuevo.
- [Tareas comunes de la conversión del diseñador de vistas a libros de Azure Monitor](platform/view-designer-conversion-tasks.md): artículo nuevo.
- [Ejemplos de conversión del diseñador de vistas a libros de Azure Monitor](platform/view-designer-conversion-examples.md): artículo nuevo.

## <a name="january-2020"></a>Enero de 2020

### <a name="general"></a>General

- [¿Qué supervisa Azure Monitor?](monitor-reference.md) - Nuevo artículo.

### <a name="agents"></a>Agentes

- [Recopilación de datos de registro con el agente de Azure Log Analytics](platform/log-analytics-agent.md): se ha actualizado la tabla de requisitos del firewall de red.

### <a name="alerts"></a>Alertas

- [Creación y administración de grupos de acciones en Azure Portal](platform/action-groups.md): se ha eliminado la configuración de las funciones v2 que ya no es necesaria.
- [Creación de una alerta de métrica con una plantilla de Resource Manager](platform/alerts-metric-create-templates.md): se ha agregado un ejemplo para el parámetro *ignoreDataBefore*.  Se han agregado restricciones sobre las reglas de varios criterios.
- [Uso de la API REST de alertas de Log Analytics](platform/api-alerts.md): se ha corregido el ejemplo de JSON.

### <a name="application-insights"></a>Application Insights

- [Direcciones IP que emplean Application Insights y Log Analytics](app/ip-addresses.md): se ha actualizado la sección de prueba de disponibilidad con la incorporación de una regla de puerto de entrada que permite el tráfico mediante grupos de seguridad de red de Azure.
- [Solución de problemas relacionados con Azure Application Insights Profiler](app/profiler-troubleshooting.md): se ha actualizado la solución de problemas general.
- [Muestreo de telemetría en Azure Application Insights](app/sampling.md): se ha actualizado y reestructurado para mejorar la legibilidad en función de los comentarios de los clientes.

### <a name="data-security"></a>Seguridad de los datos

- [Configuración de la clave administrada por el cliente de Azure Monitor](platform/customer-managed-keys.md): nuevo artículo.

### <a name="insights-and-solutions"></a>Conclusiones y soluciones

#### <a name="azure-monitor-for-containers"></a>Azure Monitor para contenedores

- [Configuración de la recopilación de datos del agente de Azure Monitor para contenedores](insights/container-insights-agent-config.md): se han agregado detalles para la actualización del agente en Red Hat OpenShift en Azure e información adicional para distinguir los métodos de actualización del agente.
- [Creación de alertas de rendimiento para Azure Monitor para contenedores](insights/container-insights-alerts.md): se ha revisado la información y se han actualizado los pasos para crear una alerta en los datos de rendimiento almacenados en el área de trabajo mediante alertas de contexto del área de trabajo.
- [Supervisión de Kubernetes con Azure Monitor para contenedores](insights/container-insights-analyze.md): se ha actualizado el artículo introductorio y el de análisis relacionado con la compatibilidad de los clústeres de Windows Kubernetes.
- [Configuración de clústeres de Red Hat OpenShift en Azure con Azure Monitor para contenedores](insights/container-insights-azure-redhat-setup.md): se han agregado detalles para la actualización del agente en Red Hat OpenShift en Azure e información adicional para distinguir los métodos de actualización del agente.
- [Configuración de los clústeres híbridos de Kubernetes con Azure Monitor para contenedores](insights/container-insights-hybrid-setup.md): se ha actualizado para reflejar la compatibilidad agregada para el puerto seguro 10250 con Kubelet's cAdvisor.
- [Administración de Azure Monitor para el agente de contenedores](insights/container-insights-manage-agent.md): se han actualizado los detalles relacionados con el comportamiento y la configuración de la recopilación de métricas de Red Hat OpenShift en Azure en comparación con otros tipos de clústeres de Kubernetes.
- [Configurar la integración de Prometheus con Azure Monitor para contenedores](insights/container-insights-prometheus-integration.md): se han actualizado los detalles relacionados con el comportamiento y la configuración de la recopilación de métricas de Red Hat OpenShift en Azure en comparación con otros tipos de clústeres de Kubernetes.
- [Actualización de Azure Monitor para contenedores para habilitar métricas](insights/container-insights-update-metrics.md): se han actualizado los detalles relacionados con el comportamiento y la configuración de la recopilación de métricas de Red Hat OpenShift en Azure en comparación con otros tipos de clústeres de Kubernetes.

#### <a name="azure-monitor-for-vms"></a>Azure Monitor para máquinas virtuales

- [Preguntas frecuentes sobre Azure Monitor para VM (disponible de forma general)](insights/vminsights-ga-release-faq.md): se ha agregado información sobre la actualización del área de trabajo y los agentes a la nueva versión.

#### <a name="office-365"></a>Office 365

- [Solución de administración de Office 365 en Azure](insights/solution-office-365.md): se han agregado detalles y preguntas frecuentes sobre la migración a una solución de Office 365 en Azure Sentinel. Se ha eliminado la sección de incorporación.

### <a name="logs"></a>Registros

- [Administración de áreas de trabajo de Log Analytics en Azure Monitor](platform/manage-access.md): se ha actualizado la sección de "No acciones".
- [Administración del uso y los costos de los registros de Azure Monitor](platform/manage-cost-storage.md): se ha agregado claridad al cálculo del volumen de datos en la sección Modelo de precios.
- [Uso de las plantillas de Azure Resource Manager para crear y configurar un área de trabajo de Log Analytics](platform/template-workspace-configuration.md): se ha actualizado la plantilla con nuevos planes de tarifa.

### <a name="platform-logs"></a>Registros de plataforma

- [Recopilación del registro de actividad de Azure con configuración de diagnóstico: Azure Monitor](./platform/activity-log.md): se ha agregado información adicional sobre las propiedades modificadas.
- [Exportación del registro de actividad de Azure](./platform/activity-log.md#legacy-collection-methods): se ha actualizado con los cambios de la interfaz de usuario. 

## <a name="december-2019"></a>Diciembre de 2019

### <a name="agents"></a>Agentes

- [Conexión de equipos Linux a Azure Monitor](platform/agent-linux.md): nuevo artículo.

### <a name="alerts"></a>Alertas

- [Creación de una alerta de métrica con una plantilla de Resource Manager](platform/alerts-metric-create-templates.md): se ha agregado un ejemplo para métricas personalizadas.
- [Creación de alertas con umbrales dinámicos en Azure Monitor](platform/alerts-dynamic-thresholds.md): se ha agregado una sección sobre la interpretación de gráficos de umbrales dinámicos.
- [Información general de alertas y supervisión de notificaciones en Azure](platform/alerts-overview.md): se ha actualizado la consulta de Resource Graph.
- [Recursos compatibles para las alertas de métricas de Azure Monitor](platform/alerts-metric-near-real-time.md): actualización sobre las métricas y dimensiones admitidas.
- [Cambio de la API heredada de alertas de Log Analytics a la nueva API de Alertas de Azure](platform/alerts-log-api-switch.md): se ha agregado una nota sobre el nombre de alerta modificado.
- [Descripción del funcionamiento de las alertas de métricas en Azure Monitor.](platform/alerts-metric-overview.md) se han agregado los tipos de recursos admitidos para la supervisión a escala.

### <a name="application-insights"></a>Application Insights

- [Application Insights para las aplicaciones de servicio de trabajo (aplicaciones sin HTTP)](app/worker-service.md): se ha agregado un nivel de registro predeterminado al código de C#. Se ha actualizado la versión de referencia del paquete.
- [Referencia de ApplicationInsights.config: Azure](app/configuration-with-applicationinsights-config.md): se ha actualizado el código de ejemplo.
- [Automatización de Azure Application Insights con PowerShell](app/powershell.md): se ha actualizado la plantilla de Resource Manager.
- [Paquetes NuGet de Application Insights de Azure Monitor](app/nuget.md): se han agregado otras versiones de paquetes.
- [Creación de un recurso de Azure Application Insights](app/create-new-resource.md): se ha agregado una nota al nombre único global.
- [Diagnóstico con Live Metrics Stream: Azure Application Insights](app/live-stream.md): se ha actualizado el requisito de versión del SDK de ASP.NET Core.
- [Contadores de eventos en Application Insights](app/eventcounters.md): se ha actualizado la categoría y la tabla de customMetrics.
- [Exploración de registros de seguimiento de Java en Azure Application Insights](app/java-trace-logs.md): se ha agregado la configuración del umbral de registro del agente de Java.
- [Direcciones IP que emplean Application Insights y Log Analytics](app/ip-addresses.md): se han actualizado las direcciones IP para Live Metrics Stream.
- [Supervisar el rendimiento de Azure App Services](app/azure-web-apps.md): se ha agregado compatibilidad con ASP.NET Core 3.0. 
- [Supervisión de aplicaciones de Python con Azure Monitor (versión preliminar)](app/opencensus-python.md): se ha agregado una aclaración para la asignación de esquemas de Python para OpenCensus en Azure. Esquema de Monitor
- [Notas de la versión de Azure Application Insights](app/release-notes.md): se han agregado notas para las versiones más antiguas.
- [Canales de telemetría en Azure Application Insights](app/telemetry-channels.md): se ha actualizado la duración de los datos descartados durante el período ampliado de la conexión perdida.
- [Muestreo de telemetría en Azure Application Insights](app/sampling.md): se ha corregido un fragmento de código del elemento TelemetryInitializer.
- [Solución de problemas de Application Insights en un proyecto web de Java](app/java-troubleshoot.md): se ha eliminado una instrucción acerca de la no admisión de la recopilación de dependencias en JDK 9.

### <a name="insights-and-solutions"></a>Conclusiones y soluciones

- [Preguntas más frecuentes sobre Azure Monitor para contenedores](./faq.md): se ha agregado una pregunta sobre los campos Image y Name.
- [Solución Azure SQL Analytics en Azure Monitor](insights/azure-sql.md): se han actualizado los tiempos de espera de la base de datos para la compatibilidad con Instancia administrada.
- [Configuración de la recopilación de datos del agente de Azure Monitor para contenedores](insights/container-insights-agent-config.md): se ha agregado la configuración de enrich_container_logs.
- [Configuración de los clústeres híbridos de Kubernetes con Azure Monitor para contenedores](insights/container-insights-hybrid-setup.md): se ha agregado la sección de solución de problemas.
- [Supervisión de Active Directory Replication Status con Azure Monitor](insights/ad-replication-status.md): se ha actualizado el requisito previo de .NET Framework.
- [Solución Network Performance Monitor en Azure](insights/network-performance-monitor.md): se han agregado regiones compatibles.
- [Optimización del entorno de Active Directory con Azure Monitor](insights/ad-assessment.md): se ha actualizado el requisito previo de .NET Framework.
- [Optimización del entorno de SQL Server con Azure Monitor](insights/sql-assessment.md): se ha actualizado el requisito previo de .NET Framework.
- [Optimización del entorno de System Center Operations Manager con Azure Log Analytics](insights/scom-assessment.md): se ha actualizado el requisito previo de .NET Framework.
- [Conexiones compatibles con IT Service Management Connector en Azure Log Analytics](platform/itsmc-connections.md): se ha agregado "New York" al requisito previo de id. de cliente y secreto de cliente.

### <a name="logs"></a>Registros

- [Eliminación y recuperación de un área de trabajo de Azure Log Analytics](platform/delete-workspace.md): se ha agregado un método con PowerShell.
- [Diseño de la implementación de registros de Azure Monitor](platform/design-logs-deployment.md): se ha aumentado la velocidad de ingesta de un área de trabajo.

### <a name="metrics"></a>Métricas

- [Métricas de plataforma de Azure Monitor que pueden exportarse con la configuración de diagnóstico](platform/metrics-supported-export-diagnostic-settings.md): nuevo artículo.

### <a name="platform-logs"></a>Registros de plataforma

Se han actualizado varios artículos como parte de la reestructuración de contenido de los registros de la plataforma en función de la nueva característica para configurar el registro de actividad mediante la configuración de diagnóstico.

- [Archivado de registros de recursos de Azure en la cuenta de almacenamiento](./platform/resource-logs.md#send-to-azure-storage)
- [Esquema de eventos del registro de actividad de Azure](platform/activity-log-schema.md)
- [Límites de servicio de Azure Monitor](service-limits.md)
- [Recopilación y análisis de los registros de actividad de Azure en un área de trabajo de Log Analytics](./platform/activity-log.md)
- [Recopilación del registro de actividad de Azure con configuración de diagnóstico (versión preliminar): Azure Monitor](./platform/activity-log.md)
- [Recopilación de registros de actividad de Azure en un área de trabajo de Log Analytics entre inquilinos de Azure](platform/activity-log-collect-tenants.md)
- [Recopilación de registros de recursos de Azure en el área de trabajo de Log Analytics](./platform/resource-logs.md#send-to-log-analytics-workspace)
- [Creación de la configuración de diagnóstico en Azure con una plantilla de Resource Manager](platform/diagnostic-settings-template.md)
- [Creación de una configuración de diagnóstico para recopilar registros y métricas en Azure](platform/diagnostic-settings.md)
- [Exportación del registro de actividad de Azure](./platform/activity-log.md#legacy-collection-methods)
- [Introducción a los registros de plataforma Azure](platform/platform-logs-overview.md)
- [Transmisión de datos de supervisión de Azure a un centro de eventos](platform/stream-monitoring-data-event-hubs.md)
- [Transmisión de registros de plataforma de Azure a un centro de eventos](./platform/resource-logs.md#send-to-azure-event-hubs)

### <a name="quickstarts-and-tutorials"></a>Guías de inicio rápido y tutoriales

- [Creación de un gráfico de métricas en Azure Monitor](learn/tutorial-metrics-explorer.md): nuevo artículo.
- [Recopilación de registros de recurso de un recurso de Azure y análisis con Azure Monitor](learn/tutorial-resource-logs.md): nuevo artículo.
- [Supervisión de recursos de Azure con Azure Monitor](learn/quick-monitor-azure-resource.md): nuevo artículo.
   
## <a name="next-steps"></a>Pasos siguientes

- Para colaborar en la documentación de Azure Monitor, consulte la [Guía para colaboradores de Microsoft Docs](/contribute/).
