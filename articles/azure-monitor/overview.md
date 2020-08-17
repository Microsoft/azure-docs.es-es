---
title: Introducción a Azure Monitor | Microsoft Docs
description: Introducción a los servicios y funcionalidades de Microsoft que contribuyen a una estrategia de supervisión completa de los servicios y las aplicaciones de Azure.
ms.subservice: ''
ms.topic: overview
author: bwren
ms.author: bwren
ms.date: 10/07/2019
ms.openlocfilehash: 21d980bcaa73af6367908b2f24c0c856d6a6c8ad
ms.sourcegitcommit: 3543d3b4f6c6f496d22ea5f97d8cd2700ac9a481
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/20/2020
ms.locfileid: "86505829"
---
# <a name="azure-monitor-overview"></a>Introducción a Azure Monitor

Azure Monitor maximiza la disponibilidad y el rendimiento de las aplicaciones y servicios con una completa solución que permite recopilar, analizar y administrar datos telemétricos tanto en la nube como en entornos locales. Esta solución le ayudará a entender cómo funcionan las aplicaciones y le permitirá identificar de manera proactiva los problemas que les afectan y los recursos de los que dependen.

Entre los ejemplos de lo que puede hacer con Azure Monitor se incluyen:

- Detección y diagnóstico de problemas en aplicaciones y dependencias con [Application Insights](app/app-insights-overview.md).
- Correlación de problemas de infraestructura con [Azure Monitor para máquinas virtuales](insights/vminsights-overview.md) y [Azure Monitor para contenedores](insights/container-insights-overview.md).
- Profundización en sus datos de supervisión con [Log Analytics](log-query/log-query-overview.md) para la solución de problemas y diagnósticos profundos.
- Soporte técnico de operaciones a escala con [alertas inteligentes](platform/alerts-smartgroups-overview.md) y [acciones automatizadas](platform/alerts-action-rules.md).
- Creación de visualizaciones con [paneles](learn/tutorial-logs-dashboards.md) y [libros](platform/workbooks-overview.md) de Azure.

> [!VIDEO https://www.microsoft.com/en-us/videoplayer/embed/RE4qXeL]


[!INCLUDE [azure-lighthouse-supported-service](../../includes/azure-lighthouse-supported-service.md)]

## <a name="overview"></a>Información general

El siguiente diagrama proporciona una visión general de Azure Monitor. En el centro del diagrama están los almacenes de datos de las métricas y los registros, que son los dos tipos fundamentales de datos que se utilizan en Azure Monitor. En la parte izquierda están los [orígenes de datos de supervisión](platform/data-sources.md) que rellenan estos [almacenes de datos](platform/data-platform.md). En la derecha, puede ver las diferentes funciones que realiza Azure Monitor con los datos recopilados, como la realización de análisis, la elaboración de alertas y la transmisión a sistemas externos.

![Introducción a Azure Monitor](media/overview/overview.png)

## <a name="monitoring-data-platform"></a>Supervisión de la plataforma de datos

Todos los datos recopilados por Azure Monitor pueden clasificarse como uno de los dos tipos fundamentales: [métricas y registros](platform/data-platform.md). Las [métricas](platform/data-platform-metrics.md) son valores numéricos que describen algún aspecto de un sistema en un momento dado. Las métricas son ligeras y capaces de admitir escenarios de tiempo casi real. Los [registros](platform/data-platform-logs.md) contienen distintos tipos de datos organizados en grupos de registros, donde cada tipo tiene diferentes conjuntos de propiedades. Los datos de telemetría, como los eventos y los seguimientos, se almacenan como registros junto con los datos de rendimiento para poder analizarlos de forma combinada.

En muchos recursos de Azure, los datos recopilados por Azure Monitor aparecen directamente en la página de información general de Azure Portal. Eche un vistazo a cualquier máquina virtual, por ejemplo, y verá varios gráficos en los que aparecen métricas de rendimiento. Haga clic en cualquiera de los gráficos para abrir los datos en el [explorador de métricas](platform/metrics-charts.md) de Azure Portal, lo que le permitirá crear gráficos con los valores de diversas métricas a lo largo del tiempo.  Puede ver los gráficos de forma interactiva o anclarlos a un panel para verlos con otras visualizaciones.

![Métricas](media/overview/metrics.png)

Los datos de registro recopilados por Azure Monitor se pueden analizar con [consultas](log-query/log-query-overview.md) que recuperan, consolidan y analizan rápidamente los datos recopilados.  Puede crear y probar consultas mediante [Log Analytics](./log-query/log-query-overview.md) en Azure Portal y después analizar los datos directamente con otras herramientas o guardar las consultas para usarlas con las [visualizaciones](visualizations.md) o las [reglas de alertas](platform/alerts-overview.md).

Azure Monitor utiliza una versión del [lenguaje de consulta Kusto](/azure/kusto/query/) utilizado por Azure Data Explorer, que es adecuado para realizar búsquedas de registros simples, pero también dispone de funciones avanzadas, como agregaciones, combinaciones y análisis inteligentes. Puede aprender rápidamente el lenguaje de consulta con [diversas lecciones](log-query/get-started-queries.md).  Se proporciona orientación concreta a los usuarios que ya están familiarizados con [SQL](log-query/sql-cheatsheet.md) y [Splunk](log-query/splunk-cheatsheet.md).

![Registros](media/overview/logs.png)

## <a name="what-data-does-azure-monitor-collect"></a>¿Qué datos recopila Azure Monitor?

Azure Monitor puede recopilar datos de diversos orígenes. Puede pensar en supervisar datos para las aplicaciones en niveles que abarcan desde la aplicación hasta el sistema operativo y los servicios en los que se basa, pasando por la propia plataforma. Azure Monitor recopila datos de cada uno de los siguientes niveles:

- **Datos de supervisión de aplicaciones**: datos sobre el rendimiento y la funcionalidad del código que ha escrito, independientemente de la plataforma.
- **Datos de supervisión del sistema operativo invitado**: datos sobre el sistema operativo en el que se ejecuta la aplicación. La aplicación se puede ejecutar en Azure, en otra nube o en el entorno local. 
- **Supervisión de recursos de Azure**: datos sobre el funcionamiento de un recurso de Azure.
- **Datos de supervisión de la suscripción de Azure**: datos sobre el funcionamiento y la administración de una suscripción de Azure, así como sobre el estado y el funcionamiento del propio Azure. 
- **Datos de supervisión de inquilino de Azure**: datos sobre el funcionamiento de los servicios de Azure en el nivel del inquilino, como Azure Active Directory.

En cuanto crea una suscripción a Azure y empieza a agregar recursos, como máquinas virtuales y aplicaciones web, Azure Monitor comienza a recopilar datos.  Los [registros de actividad](platform/platform-logs-overview.md) registran la creación y modificación de recursos. Las [métricas](platform/data-platform.md) indican cómo está funcionando un recurso y los recursos que consume. 

Amplíe los datos que recopila y obtenga información sobre el funcionamiento real de los recursos [habilitando diagnósticos](platform/platform-logs-overview.md) y [agregando un agente](platform/agent-windows.md) a los recursos de proceso. De este modo, recopilará datos de telemetría sobre el funcionamiento interno de un recurso y podrá configurar diferentes [orígenes de datos](platform/agent-data-sources.md) para recopilar registros y métricas del sistema operativo invitado Windows y Linux. 

Habilite la supervisión de la [aplicación de App Services](app/azure-web-apps.md) o de la [máquina virtual y aplicación del conjunto de escalado de máquinas virtuales](app/azure-vm-vmss-apps.md) para permitir que Application Insights recopile información detallada acerca de la aplicación, como vistas de página, solicitudes de aplicación y excepciones. Compruebe de forma más exhaustiva la disponibilidad de la aplicación configurando una [prueba de disponibilidad](app/monitor-web-app-availability.md) para simular el tráfico de usuarios.

### <a name="custom-sources"></a>Orígenes personalizados

Azure Monitor puede recopilar datos de registro de cualquier cliente de REST mediante [Data Collector API](platform/data-collector-api.md). Esto permite crear escenarios de supervisión personalizados y ampliar la supervisión a los recursos que no exponen datos de telemetría en otros orígenes.

## <a name="insights"></a>Información detallada
Los datos de supervisión solo resultan útiles si aportan una mayor visibilidad sobre el funcionamiento del entorno informático. Azure Monitor cuenta con varias características y herramientas que proporcionan valiosa información sobre las aplicaciones y los recursos de los que dependen. Las características y las [soluciones de supervisión](insights/solutions.md), como [Application Insights](app/app-insights-overview.md) y [Azure Monitor para contenedores](insights/container-insights-overview.md), proporcionan información exhaustiva sobre diferentes aspectos de la aplicación y determinados servicios de Azure. 

### <a name="application-insights"></a>Application Insights
[Application Insights](app/app-insights-overview.md) supervisa la disponibilidad, el rendimiento y el uso de las aplicaciones web, tanto si están hospedadas en la nube como en un entorno local. Esta solución utiliza la eficaz plataforma de análisis de datos de Azure Monitor para proporcionarle información exhaustiva sobre las operaciones de la aplicación y diagnosticar errores sin esperar a que un usuario lo notifique. Application Insights incorpora puntos de conexión con una serie de herramientas de desarrollo y se integra con Visual Studio para admitir los procesos de DevOps.

![Detalles de la aplicación](media/overview/app-insights.png)

### <a name="azure-monitor-for-containers"></a>Azure Monitor para contenedores
[Azure Monitor para contenedores](insights/container-insights-overview.md) es una característica diseñada para supervisar el rendimiento de las cargas de trabajo de contenedor implementadas en clústeres de Kubernetes administrados y hospedados en Azure Kubernetes Service (AKS). Proporciona información sobre el rendimiento recopilando métricas del procesador y de la memoria procedentes de los controladores, los nodos y los contenedores disponibles en Kubernetes mediante Metrics API. También se recopilan registros del contenedor.  Una vez habilitada la supervisión de clústeres de Kubernetes, estas métricas y registros se recopilan automáticamente mediante una versión en contenedor del agente de Log Analytics para Linux.

![Estado de los contenedores](media/overview/container-insights.png)

### <a name="azure-monitor-for-vms"></a>Azure Monitor para máquinas virtuales
La solución [Azure Monitor para VM](insights/vminsights-overview.md) supervisa las máquinas virtuales de Azure a escala y analiza el rendimiento y el estado de las máquinas virtuales de Windows y Linux, incluidos los diferentes procesos y las dependencias interconectadas con otros recursos y con procesos externos. La solución permite supervisar el rendimiento y las dependencias de las aplicaciones en máquinas virtuales hospedadas en el entorno local o en otro proveedor en la nube.  


![VM Insights](media/overview/vm-insights.png)

### <a name="monitoring-solutions"></a>Soluciones de supervisión
Las [soluciones de supervisión](insights/solutions.md) de Azure Monitor son conjuntos empaquetados de lógica que proporcionan información sobre una determinada aplicación o servicio. Incluyen la lógica para recopilar datos de supervisión para la aplicación o servicio, [consultas](log-query/log-query-overview.md) para analizar esos datos y [vistas](./platform/view-designer.md) para su visualización. Las soluciones de supervisión están [disponibles en Microsoft](./monitor-reference.md) y otros asociados, y ofrecen herramientas de supervisión para distintos servicios de Azure y otras aplicaciones.

![Soluciones de supervisión](media/overview/solutions-overview.png)

## <a name="responding-to-critical-situations"></a>Respuesta a situaciones críticas
Además de permitirle analizar de forma interactiva los datos de supervisión, una solución de supervisión eficaz debe ser capaz de responder proactivamente a condiciones críticas que se den en los datos que recopila. Esto podría hacerse enviando un mensaje o correo a un administrador responsable de investigar un problema. O también podría hacerse iniciando un proceso automatizado que intente corregir una condición de error.


### <a name="alerts"></a>Alertas
Las [alertas de Azure Monitor](platform/alerts-overview.md) informan de forma proactiva de los estados críticos e intentan aplicar acciones correctivas. Las reglas de alertas basadas en métricas proporcionan avisos casi en tiempo real que se generan en función de unos valores numéricos, mientras que las reglas basadas en registros permiten aplicar una lógica compleja entre datos de diversos orígenes.

Las reglas de alertas de Azure Monitor utilizan [grupos de acciones](platform/action-groups.md), que contienen diferentes conjuntos de destinatarios y acciones que pueden compartirse entre varias reglas. En función de los requisitos, los grupos de acciones pueden realizar diferentes acciones, como utilizar webhooks para que las alertas inicien acciones externas o se integren con las herramientas de administración de servicios de TI.

![Alertas](media/overview/alerts.png)

### <a name="autoscale"></a>Escalado automático
Gracias al escalado automático, puede ejecutar la cantidad correcta de recursos para administrar la carga de la aplicación. Permite crear reglas que usan las métricas recopiladas por Azure Monitor para determinar cuándo deben agregarse automáticamente recursos que controlen el aumento de la carga y permitan ahorrar dinero quitando recursos inactivos. Tiene que especificar un número mínimo y máximo de instancias y la lógica para decidir cuándo deben aumentar o disminuir los recursos.

![Escalado automático](media/overview/autoscale.png)

## <a name="visualizing-monitoring-data"></a>Visualización de los datos de supervisión
Las [visualizaciones](visualizations.md), como los gráficos y las tablas, son herramientas eficaces para resumir los datos de supervisión y presentarlos a distintos destinatarios. Azure Monitor cuenta con sus propias características para visualizar los datos de supervisión y utiliza otros servicios de Azure para publicarlos ante diferentes destinatarios.

### <a name="dashboards"></a>Paneles
Los [paneles de Azure](../azure-portal/azure-portal-dashboards.md) permiten combinar distintos tipos de datos, como métricas y registros, en un único panel de [Azure Portal](https://portal.azure.com). Si lo desea, también compartir el panel con otros usuarios de Azure. Los elementos que componen Azure Monitor pueden agregarse a un panel de Azure, así como a la salida de cualquier consulta de registro o gráfico de métricas. Por ejemplo, puede crear un panel que contenga diferentes iconos que muestren un gráfico de métricas, una tabla de registros de actividad, un gráfico de uso de Application Insights y la salida de una consulta de registro.

![Panel](media/overview/dashboard.png)

### <a name="views"></a>Vistas
Las [vistas](./platform/view-designer.md) representan visualmente los datos de registro de Azure Monitor.  Cada vista contiene un icono que da acceso a una combinación de visualizaciones, como gráficos de barras y líneas, así como a listas en las que resumen los datos críticos.  Las soluciones de supervisión contienen vistas en las que se resumen los datos de una determinada aplicación. Puede crear sus propias vistas para presentar los datos de cualquier consulta de registros. Al igual que otros elementos de Azure Monitor, las vistas se pueden agregar a los paneles de Azure.

![Ver](media/overview/view.png)

### <a name="power-bi"></a>Power BI
[Power BI](https://powerbi.microsoft.com) es un servicio de análisis empresarial que proporciona visualizaciones interactivas en una serie de orígenes de datos y que constituye un mecanismo eficaz para poner los datos a disposición de personas que están dentro y fuera de la organización. Puede configurar Power BI para que [los datos de registro se importen automáticamente desde Azure Monitor](./platform/powerbi.md) y utilizar estas otras visualizaciones.


![Power BI](media/overview/power-bi.png)


## <a name="integrate-and-export-data"></a>Integración y exportación de datos
A menudo, necesitará integrar Azure Monitor con otros sistemas y crear soluciones personalizadas que utilicen los datos de supervisión. Otros servicios de Azure funcionan con Azure Monitor para proporcionar esta integración.

### <a name="event-hub"></a>Centro de eventos
[Azure Event Hubs](../event-hubs/index.yml) es una plataforma de streaming y un servicio de ingesta de eventos que puede transformar y almacenar los datos utilizando cualquier proveedor de análisis en tiempo real o adaptadores de procesamiento por lotes y almacenamiento de datos. Utilice Event Hubs para [transmitir datos de Azure Monitor](platform/stream-monitoring-data-event-hubs.md) a herramientas de supervisión y administración de eventos e información de seguridad de asociados.


### <a name="logic-apps"></a>Logic Apps
[Logic Apps](https://azure.microsoft.com/services/logic-apps) es un servicio que le permite automatizar tareas y procesos de negocio mediante flujos de trabajo que se integran con diferentes sistemas y servicios. Hay actividades disponibles que leen y escriben registros y métricas en Azure Monitor, lo que le permite crear flujos de trabajo que se integran con muchos otros sistemas.


### <a name="api"></a>API
Existen varias API para leer y escribir métricas y registros en Azure Monitor, que además proporcionan acceso a las alertas generadas. También puede configurar y recuperar alertas. De este modo, dispone de unas posibilidades prácticamente ilimitadas para crear soluciones personalizadas que se integren con Azure Monitor.

## <a name="next-steps"></a>Pasos siguientes
Más información sobre:

* Las [métricas y registros](platform/data-platform.md) de los datos que recopila Azure Monitor.
* Los [orígenes de datos](platform/data-sources.md) de la forma en que los distintos componentes de la aplicación envían la telemetría.
* Las [consultas de registros](log-query/log-query-overview.md) para analizar los datos recopilados.
* [Procedimientos recomendados](/azure/architecture/best-practices/monitoring) para supervisar los servicios y las aplicaciones en la nube.
