---
title: Supervisión de los recursos de Azure con Azure Monitor | Microsoft Docs
description: Describe cómo recopilar y analizar los datos de supervisión de los recursos de Azure mediante Azure Monitor.
ms.subservice: logs
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 10/08/2019
ms.openlocfilehash: de381b4200e2ec3381f452ac0ec32e61a055a6ec
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87326178"
---
# <a name="monitoring-azure-resources-with-azure-monitor"></a>Supervisión de recursos de Azure con Azure Monitor
Si tiene aplicaciones y procesos empresariales críticos que dependen de recursos de Azure, querrá supervisar esos recursos para su disponibilidad, rendimiento y funcionamiento. En este artículo se describen los datos de supervisión generados por los recursos de Azure y cómo puede usar las características de Azure Monitor para analizar y alertar sobre estos datos.

> [!IMPORTANT]
> Este artículo se aplica a todos los servicios de Azure que usan Azure Monitor. Los recursos de proceso, incluidas las máquinas virtuales y App Service, generan los mismos datos de supervisión que se describen aquí, pero también tienen un sistema operativo invitado que puede generar asimismo registros y métricas. Consulte la documentación de supervisión de estos servicios para más información sobre cómo recopilar y analizar estos datos.

## <a name="what-is-azure-monitor"></a>¿Qué es Azure Monitor?
Azure Monitor es un servicio de supervisión de pila completo de Azure, que proporciona un conjunto completo de características para supervisar los recursos de Azure, además de los recursos locales y en otras nubes. La [Plataforma de datos de Azure Monitor](../platform/data-platform.md) recopila datos en [registros](../platform/data-platform-logs.md) y [métricas](../platform/data-platform-metrics.md), donde se pueden analizar conjuntamente con un conjunto completo de herramientas de supervisión, tal como se describe en las secciones siguientes.

- [¿Qué puede hacer con las métricas de Azure Monitor?](../platform/data-platform-metrics.md#what-can-you-do-with-azure-monitor-metrics)
- [¿Qué puede hacer con los registros de Azure Monitor?](../platform/data-platform-logs.md#what-can-you-do-with-azure-monitor-logs)

Al crear un recurso de Azure, se habilita Azure Monitor, y se inicia la recopilación de métricas y registros de actividad que puede [ver y analizar en Azure Portal](#monitoring-in-the-azure-portal). Con algunas opciones de configuración, puede recopilar datos de supervisión adicionales y habilitar características adicionales. Consulte [Supervisión de datos](#monitoring-data) a continuación para más información sobre los requisitos de configuración.


## <a name="costs-associated-with-monitoring"></a>Costos asociados con la supervisión
El análisis de los datos de supervisión que se recopilan de forma predeterminada no supone ningún costo. Incluye lo siguiente:

- Recopilación de métricas de la plataforma y su análisis con el Explorador de métricas.
- Recopilación del registro de actividad y su análisis en Azure Portal.
- Creación de una regla de alerta de registro de actividad.

No hay ningún costo de Azure Monitor para recopilar y exportar registros y métricas, pero puede haber costos relacionados asociados con el destino:

- Costos asociados con la ingesta de datos y la retención al recopilar registros y métricas en el área de trabajo de Log Analytics. Consulte [Precios de Azure Monitor para Log Analytics](https://azure.microsoft.com/pricing/details/monitor/).
- Costos asociados con el almacenamiento de datos al recopilar registros y métricas en una cuenta de almacenamiento de Azure. Consulte [Precios de Azure Storage para Blob Storage](https://azure.microsoft.com/pricing/details/storage/blobs/).
- Costos asociados con el streaming de Event Hubs al reenviar registros y métricas a Azure Event Hubs. Consulte [Precios de Azure Event Hubs](https://azure.microsoft.com/pricing/details/event-hubs/).

Puede haber costos de Azure Monitor asociados con lo siguiente. Consulte [Precios de Azure Monitor](https://azure.microsoft.com/pricing/details/monitor/):

- Ejecución de una consulta de registro.
- Creación de una regla de alerta de consulta de registro o métrica.
- Envío de una notificación desde cualquier regla de alerta.
- Acceso a las métricas mediante API.

## <a name="monitoring-data"></a>Supervisión de datos
Los recursos de Azure generan [registros](../platform/data-platform-logs.md) y [métricas](../platform/data-platform-metrics.md) que aparecen en el diagrama siguiente. Consulte la documentación de cada uno de los servicios de Azure para ver los datos específicos que generan y cualquier solución o información que proporcionen.

![Información general](media/monitor-azure-resource/logs-metrics.png)



- [Métricas de plataforma](../platform/data-platform-metrics.md): valores numéricos que se recopilan automáticamente a intervalos regulares y describen algún aspecto de un recurso en un momento determinado. 
- [Registros de recurso](../platform/platform-logs-overview.md): proporcionan información de las operaciones realizadas dentro del mismo recurso de Azure (en el plano de datos), por ejemplo, obtención de un secreto de un almacén de claves o realización de una solicitud a una base de datos. El contenido y la estructura de estos registros de recurso varían según el servicio de Azure y el tipo de recurso.
- [Registro de actividad](../platform/platform-logs-overview.md): proporciona información general de las operaciones de cada recurso de Azure de la suscripción desde fuera (en el plano de administración), por ejemplo, creación de un nuevo recurso o inicio de una máquina virtual. Esta información es sobre los interrogantes qué, quién y cuándo para las operaciones de escritura (PUT, POST, DELETE) en los recursos de la suscripción.


## <a name="configuration-requirements"></a>Requisitos de configuración

### <a name="configure-monitoring"></a>Configuración de la supervisión
Algunos datos de supervisión se recopilan automáticamente, pero es posible que tenga que configurar algunos valores en función de sus requisitos. Consulte la información siguiente y vea la información específica para cada tipo de datos de supervisión.

- [Métricas de plataforma](../platform/data-platform-metrics.md): las métricas de plataforma se recopilan automáticamente en [Métricas en Azure Monitor](../platform/data-platform-metrics.md) sin necesidad de configuración. Cree una configuración de diagnóstico para enviar entradas a registros de Azure Monitor o reenviarlas fuera de Azure.
- [Registros de recurso](../platform/platform-logs-overview.md): los recursos de Azure generan automáticamente registros de recurso, pero estos no se recopilan sin una configuración de diagnóstico.  Cree una configuración de diagnóstico para enviar entradas a registros de Azure Monitor o reenviarlas fuera de Azure.
- [Registro de actividad](../platform/platform-logs-overview.md): el registro de actividad se recopila automáticamente sin necesidad de configuración y puede verse en Azure Portal. Cree una configuración de diagnóstico para copiarla en registros de Azure Monitor o reenviarla fuera de Azure.

### <a name="log-analytics-workspace"></a>Área de trabajo de Log Analytics
La recopilación de datos en los registros de Azure Monitor requiere un área de trabajo de Log Analytics. Puede empezar a supervisar rápidamente el servicio creando una nueva área de trabajo, pero también puede resultar útil usar un área de trabajo que recopila datos de otros servicios. Consulte [Creación de un área de trabajo de Log Analytics en Azure Portal](../learn/quick-create-workspace.md), para conocer más detalles sobre cómo crear un área de trabajo, y [Diseño de la implementación de registros de Azure Monitor](../platform/design-logs-deployment.md), como ayuda para determinar el mejor diseño de área de trabajo para sus requisitos. Si usa un área de trabajo existente en la organización, necesitará los permisos adecuados, tal como se describe en [Administración del acceso a los datos de registro y las áreas de trabajo en Azure Monitor](../platform/manage-access.md). 





## <a name="diagnostic-settings"></a>Configuración de diagnóstico
La configuración de diagnóstico define dónde se deben enviar los registros de recurso y las métricas para un recurso determinado. Los posibles destinos son:

- [Área de trabajo de Log Analytics](../platform/resource-logs.md#send-to-log-analytics-workspace), que permite analizar datos con otros datos de supervisión recopilados por Azure Monitor mediante unas consultas de registro eficaces y también aprovechar otras características de Azure Monitor, como alertas de registros y visualizaciones. 
- [Event Hubs](../platform/resource-logs.md#send-to-azure-event-hubs), para transmitir datos a sistemas externos, como SIEM de terceros y otras soluciones de análisis de registros. 
- [Cuenta de almacenamiento de Azure](../platform/resource-logs.md#send-to-azure-storage), que resulta útil para la auditoría, el análisis estático o la copia de seguridad.

Siga el procedimiento de [Creación de una configuración de diagnóstico para recopilar registros de plataforma y métricas en Azure](../platform/diagnostic-settings.md) para crear y administrar la configuración de diagnóstico mediante Azure Portal. Consulte [Creación de la configuración de diagnóstico en Azure con una plantilla de Resource Manager](../platform/diagnostic-settings-template.md) para definirla en una plantilla y habilitar la supervisión completa para un recurso cuando se crea.


## <a name="monitoring-in-the-azure-portal"></a>Supervisión en Azure Portal
 Puede acceder a los datos de supervisión para la mayoría de los recursos de Azure en el menú del recurso en Azure Portal. Esto le proporcionará acceso a los datos de un solo recurso con las herramientas de Azure Monitor estándar. Algunos servicios de Azure proporcionarán opciones diferentes, por lo que debe consultar la documentación de ese servicio para obtener información adicional. Use el menú **Azure Monitor** para analizar los datos de todos los recursos supervisados. 

### <a name="overview"></a>Información general
Muchos servicios incluirán datos de supervisión en su página **Información general** para una vista rápida de su funcionamiento. Esto se basará, normalmente, en un subconjunto de métricas de plataforma almacenadas en Métricas de Azure Monitor. En general, habrá otras opciones de supervisión disponibles en una sección **Supervisión** del menú de servicio.

![Página de información general](media/monitor-azure-resource/overview-page.png)


### <a name="insights-and-solutions"></a>Insights y soluciones 
Algunos servicios proporcionarán herramientas más allá de las características estándar de Azure Monitor. [Insights](./insights-overview.md) proporciona una experiencia de supervisión personalizada basada en la plataforma de datos de Azure Monitor y las características estándar. Las [soluciones](./solutions.md) proporcionan lógica de supervisión predefinida basada en los registros de Azure Monitor. 

Si un servicio tiene una conclusión de Azure Monitor, puede acceder a ella desde la opción **Supervisión** en el menú de cada recurso. Puede acceder a todas las conclusiones y las soluciones desde el menú de **Azure Monitor**.

![Información detallada](media/monitor-azure-resource/insights.png)

### <a name="metrics"></a>Métricas
Analice las métricas en Azure Portal mediante el [explorador de métricas](../platform/metrics-getting-started.md), que está disponible en el elemento de menú **Métricas** de la mayoría de los servicios. Esta herramienta permite trabajar con métricas individuales o combinar varias para identificar correlaciones y tendencias. 

- Consulte [Introducción al Explorador de métricas de Azure](../platform/metrics-getting-started.md) para más información sobre el Explorador de métricas.
- Consulte [Características avanzadas del Explorador de métricas de Azure](../platform/metrics-charts.md) para ver características avanzadas del Explorador de métricas, como el uso de varias métricas, y la aplicación de filtros y división.

![Métricas](media/monitor-azure-resource/metrics.png)


### <a name="activity-log"></a>Registro de actividades 
Vea las entradas del registro de actividad en Azure Portal con el filtro inicial establecido en el recurso actual. Copie el registro de actividad en un área de trabajo de Log Analytics para acceder a él, y usarlo en consultas y libros de registro. 

- Consulte [Visualización y recuperación de eventos del registro de actividad de Azure](../platform/activity-log.md#view-the-activity-log) para más información sobre la visualización del registro de actividad y la recuperación de entradas mediante varios métodos.
- Consulte la documentación del servicio de Azure para conocer los eventos específicos que se registran.

![Registro de actividad](media/monitor-azure-resource/activity-log.png)

### <a name="azure-monitor-logs"></a>Registros de Azure Monitor
Los registros de Azure Monitor consolidan los registros y las métricas de varios servicios, y otros orígenes de datos para su análisis con una herramienta de consulta eficaz. Como se ha descrito anteriormente, cree una configuración de diagnóstico para recopilar métricas de la plataforma, el registro de actividad y los registros de recursos en un área de trabajo de Log Analytics en Azure Monitor.

[Log Analytics](../log-query/get-started-portal.md) le permite trabajar con las [consultas de registros](../log-query/log-query-overview.md), que es una característica eficaz de Azure Monitor que ayuda a realizar un análisis avanzado de los datos de registro mediante un lenguaje de consulta muy completo. Abra Log Analytics en **Registros** en el menú **Supervisión** para que un recurso de Azure funcione con las consultas de registro que usan el recurso como [ámbito de consulta](../log-query/scope.md#query-scope). Esto le permite analizar datos en varias tablas solo para ese recurso. Use **Registros** en el menú de Azure Monitor para acceder a los registros de todos los recursos. 

- Consulte [Introducción a las consultas de registro en Azure Monitor](../log-query/get-started-queries.md) para ver un tutorial sobre el uso del lenguaje de consulta que se utiliza para escribir consultas de registro.
- Consulte [Recopilación de registros de recurso de Azure en el área de trabajo de Log Analytics en Azure Monitor](../platform/resource-logs.md#send-to-log-analytics-workspace) para información sobre cómo se recopilan los registros de recurso en los registros de Azure Monitor y detalles sobre cómo obtener acceso a ellos en una consulta.
- Consulte [Modo de recopilación](../platform/resource-logs.md#send-to-log-analytics-workspace) para ver una explicación de cómo se estructuran los datos del registro de recursos en los registros de Azure Monitor.
- Consulte la documentación de cada servicio de Azure para más información sobre su tabla en los registros de Azure Monitor.

![Registros](media/monitor-azure-resource/logs.png)

## <a name="monitoring-from-command-line"></a>Supervisión desde la línea de comandos
Puede acceder a los datos de supervisión recopilados en el recurso desde una línea de comandos o incluirlos en un script mediante [Azure PowerShell](/powershell/azure/) o la [interfaz de la línea de comandos de Azure](/cli/azure/). 

- Consulte la [referencia de métricas de la CLI](/cli/azure/monitor/metrics) para acceder a los datos de métricas de la CLI.
- Consulte la [referencia de Log Analytics de la CLI](/cli/azure/ext/log-analytics/monitor/log-analytics) para acceder a los datos de los registros de Azure Monitor mediante una consulta de registro desde la CLI.
- Consulte la [referencia de métricas de Azure PowerShell](/powershell/module/azurerm.insights/get-azurermmetric) para acceder a los datos de métricas desde Azure PowerShell.
- Consulte la [referencia de consulta de registro de Azure PowerShell](/powershell/module/az.operationalinsights/Invoke-AzOperationalInsightsQuery) para acceder a los datos de los registros de Azure Monitor mediante una consulta de registro desde Azure PowerShell.

## <a name="monitoring-from-rest-api"></a>Supervisión desde una API REST
Incluya datos de supervisión recopilados desde el recurso en una aplicación personalizada mediante una API REST.

- Consulte [Tutorial sobre la API REST de supervisión de Azure](../platform/rest-api-walkthrough.md) para más información sobre el acceso a las métricas desde la API REST de Azure Monitor.
- Consulte el artículo sobre la [API REST de Azure Log Analytics](https://dev.loganalytics.io/) para obtener información sobre cómo acceder a los datos de los registros de Azure Monitor mediante una consulta de registro desde Azure PowerShell.

## <a name="alerts"></a>Alertas
Las [alertas](../platform/alerts-overview.md) le informan de forma proactiva y pueden tomar medidas cuando se detectan condiciones importantes en los datos que supervisa. Cree una regla de alerta que defina un destino para la alerta, las condiciones para la creación de una alerta y las acciones que se deben realizar en respuesta.

Se usan diferentes tipos de datos de supervisión para distintos tipos de reglas de alerta.

- [Alerta de registro de actividad](../platform/alerts-activity-log.md): cree una alerta al crear una entrada en el registro de actividad que coincida con criterios específicos. Esto le permite recibir una notificación, por ejemplo, cuando se crea un tipo determinado de recurso o si se produce un error en un cambio de configuración.
- [Alerta de métrica](../platform/alerts-metric.md): cree una alerta cuando un valor de métrica supere un umbral determinado. Las alertas de métricas tienen más capacidad de respuesta que otras alertas y se pueden resolver automáticamente cuando se corrige el problema.
- [Alerta de consulta de registro](../platform/alerts-log.md): ejecute una consulta de registro a intervalos regulares y cree una alerta si se encuentra una condición determinada. Esto le permite realizar análisis complejos en varios conjuntos de datos.

Use **Alertas** en el menú de un recurso para ver alertas y administrar las reglas de alerta para ese recurso. Solo las alertas del registro de actividad y las alertas de métricas utilizan recursos individuales de Azure como destino. Las alertas de consulta de registro usan un área de trabajo de Log Analytics como destino y se basan en una consulta que puede acceder a los registros almacenados en esa área de trabajo. Use el menú de Azure Monitor para ver y administrar las alertas de todos los recursos, y administrar las reglas de alerta de consultas de registro.

- Consulte los artículos de los diferentes tipos de alerta anteriores para más información sobre la creación de reglas de alerta.
- Consulte [Creación y administración de grupos de acciones en Azure Portal](../platform/action-groups.md) para más información sobre cómo crear un grupo de acciones que le permita administrar las respuestas a las alertas.



## <a name="next-steps"></a>Pasos siguientes

* Consulte [Servicios, esquemas y categorías admitidos en los registros de recursos de Azure](../platform/resource-logs-schema.md) para más información sobre los registros de recursos para los distintos servicios de Azure.  

