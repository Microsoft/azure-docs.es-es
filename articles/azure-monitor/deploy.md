---
title: Implementación de Azure Monitor
description: Describe los diferentes pasos necesarios para realizar una implementación completa de Azure Monitor para supervisar todos los recursos de la suscripción de Azure.
ms.subservice: ''
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 07/27/2020
ms.openlocfilehash: 34a048c702b62caeecaf21e710a9dcd9156e4aea
ms.sourcegitcommit: fbb66a827e67440b9d05049decfb434257e56d2d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/05/2020
ms.locfileid: "87801326"
---
# <a name="deploy-azure-monitor"></a>Implementación de Azure Monitor
Habilitar Azure Monitor para supervisar todos los recursos de Azure es una combinación de la configuración de los componentes de Azure Monitor y la configuración de los recursos de Azure para generar datos de supervisión para que Azure Monitor los recopile. En este artículo se describen los diferentes pasos necesarios para realizar una implementación completa de Azure Monitor con una configuración común para supervisar todos los recursos de la suscripción de Azure. Se proporcionan descripciones básicas de cada paso con vínculos a otra documentación para obtener información detallada sobre los requisitos de configuración.

> [!IMPORTANT]
> Las características de Azure Monitor y su configuración variarán en función de los requisitos empresariales que se equilibran con el costo de las características habilitadas. En cada paso siguiente se identificará si existe algún costo potencial y debe evaluar estos costos antes de continuar con ese paso. Consulte [Precios de Azure Monitor](https://azure.microsoft.com/pricing/details/monitor/) para obtener información detallada sobre los precios.

## <a name="configuration-goals"></a>Objetivos de configuración
El objetivo de una implementación completa de Azure Monitor es recopilar todos los datos disponibles de todos sus recursos y aplicaciones en la nube y habilitar tantas características como sea posible en Azure Monitor según los datos.

Los datos recopilados por Azure Monitor se envían a [Métricas de Azure Monitor](platform/data-platform-metrics.md) o [Registros de Azure Monitor](platform/data-platform-logs.md). Cada uno almacena distintos tipos de datos y permite diferentes tipos de análisis y alertas. Consulte [Comparación de las métricas y los registros de Azure Monitor](platform/data-platform.md) para obtener una comparación de ambos e [Información general sobre las alertas en Microsoft Azure](platform/alerts-overview.md) para obtener una descripción de los diferentes tipos de alerta. 

Algunos datos se pueden enviar a las métricas y los registros para poder aprovecharlos con diferentes características. En estos casos, puede que tenga que configurar cada uno por separado. Por ejemplo, los recursos de Azure envían los datos de métricas a las métricas, que admite el explorador de métricas y las alertas de métricas. Debe crear una configuración de diagnóstico para que cada recurso envíe los mismos datos de métricas a los registros, lo que le permite analizar las tendencias de rendimiento con otros datos de registro mediante Log Analytics. En las secciones siguientes se identifica dónde se envían los datos y se incluye cada paso necesario para enviar datos a todas las ubicaciones posibles.

Puede tener requisitos adicionales, como la supervisión de recursos fuera de Azure y el envío de datos fuera de Azure Monitor. Requisitos como estos se pueden lograr con la configuración adicional de las características descritas en este artículo. Siga los vínculos a la documentación en cada paso para obtener opciones de configuración adicionales.

## <a name="collect-data-from-azure-resources"></a>Recopilación de datos a partir de recursos de Azure

> [!NOTE]
> Consulte [Supervisión de recursos de Azure con Azure Monitor](insights/monitor-azure-resource.md) para obtener una guía completa sobre la supervisión de máquinas virtuales con Azure Monitor.

La supervisión de los recursos de Azure está disponible automáticamente sin necesidad de configuración, pero debe realizar los pasos de configuración para recopilar datos de supervisión adicionales. En la tabla siguiente se muestran los pasos de configuración necesarios para recopilar todos los datos disponibles de los recursos de Azure, incluido el paso en el que se envían los datos a Métricas de Azure Monitor y Registros de Azure Monitor. Cada uno de los pasos se describe en detalle en las secciones que aparecen a continuación.

[ ![Supervisión de la implementación de recursos de Azure](media/deploy/deploy-azure-resources.png) ](media/deploy/deploy-azure-resources.png#lightbox)

### <a name="no-configuration"></a>Sin configuración
Las siguientes características de Azure Monitor están habilitadas sin necesidad de configuración al crear una suscripción de Azure. No hay ningún costo asociado con esta supervisión.

[Registros de Azure Active Directory](../active-directory/reports-monitoring/overview-reports.md): proporcionan el historial de la actividad de inicio de sesión y la traza de auditoría de los cambios realizados en Azure Active Directory. Consulte [Informes de actividad de auditoría en el portal de Azure Active Directory](../active-directory/reports-monitoring/concept-audit-logs.md) e [Informes de actividad de inicio de sesión en el portal de Azure Active Directory](../active-directory/reports-monitoring/concept-sign-ins.md) para obtener más información sobre los registros de Azure Active Directory y cómo verlos en Azure Portal.

[Registro de actividad](platform/platform-logs-overview.md): proporciona información sobre el grupo de administración y los eventos que se han producido en Azure en el nivel de la suscripción. Los eventos se escriben automáticamente en el registro de actividad al crear un nuevo recurso de Azure, modificar un recurso o realizar una actividad significativa. Puede ver los eventos en Azure Portal y crear alertas del registro de actividad cuando se crean eventos concretos. Consulte [Registro de actividad de Azure](platform/activity-log.md) para obtener más información sobre el registro de actividad y cómo verlo en Azure Portal.

[Métricas de plataforma](platform/metrics-supported.md): se recopilan automáticamente de los servicios de Azure en las [métricas de Azure Monitor](platform/data-platform-metrics.md). Estos datos se suelen presentar en la página **Información general** de Azure Portal de los distintos servicios. Consulte [Introducción al Explorador de métricas de Azure](platform/metrics-getting-started.md) para obtener más información sobre el análisis de las métricas de la plataforma en Azure Portal. 


### <a name="create-log-analytics-workspace"></a>Creación de un área de trabajo de Log Analytics
Necesita al menos un área de trabajo de Log Analytics para habilitar los [registros de Azure Monitor](platform/data-platform-logs.md), lo que es necesario para recopilar estos datos como registros de recursos de Azure, recopilar datos del sistema operativo invitado de Azure Virtual Machines y para la mayoría de la información de Azure Monitor. Otros servicios, como Azure Sentinel y Azure Security Center, también usan un área de trabajo de Log Analytics y pueden compartir la misma que se usa para Azure Monitor. Puede empezar con una sola área de trabajo para admitir esta supervisión, pero consulte [Diseño de la implementación de registros de Azure Monitor](platform/design-logs-deployment.md) para obtener instrucciones sobre cuándo usar varias áreas de trabajo.

No hay ningún costo por la creación de un área de trabajo de Log Analytics, pero se puede realizar un cargo adicional una vez configurados los datos que se van a recopilar. Consulte [Administración del uso y los costos con los registros de Azure Monitor](platform/manage-cost-storage.md) para más información.  

Consulte [Creación de un área de trabajo de Log Analytics en Azure Portal](learn/quick-create-workspace.md) para crear un área de trabajo de Log Analytics inicial. Consulte [Administración del acceso a los datos de registro y las áreas de trabajo en Azure Monitor](platform/manage-access.md) para configurar el acceso. 

### <a name="create-diagnostic-setting-to-collect-tenant-and-subscription-logs"></a>Creación de una configuración de diagnóstico para recopilar registros de inquilinos y suscripciones
Aunque los [registros de Azure Active Directory](../active-directory/reports-monitoring/overview-reports.md) del inquilino y el [registro de actividad](platform/platform-logs-overview.md) de la suscripción se recopilan automáticamente, al enviarlos a un área de trabajo de Log Analytics, se pueden analizar estos eventos con otros datos de registro mediante consultas de registro en Log Analytics. Esto también le permite crear alertas de consulta de registro, que es la única manera de alertar sobre los registros de Azure Active Directory y proporcionar una lógica más compleja que las alertas del registro de actividad.

No hay ningún costo por enviar el registro de actividad a un área de trabajo, pero hay una ingesta de datos y un cargo de retención para los registros de Azure Active Directory. 

Consulte [Integración de registros de Azure AD con registros de Azure Monitor](../active-directory/reports-monitoring/howto-integrate-activity-logs-with-log-analytics.md) y [Creación de una configuración de diagnóstico para enviar los registros y las métricas de la plataforma a diferentes destinos](platform/diagnostic-settings.md) para crear una configuración de diagnóstico para el inquilino y la suscripción para enviar entradas de registro a su área de trabajo de Log Analytics. 

### <a name="create-diagnostic-setting-to-collect-resource-logs-and-platform-metrics"></a>Creación de una configuración de diagnóstico para recopilar registros de recursos y métricas de la plataforma
Los recursos de Azure generan automáticamente [registros de recursos](platform/platform-logs-overview.md) que proporcionan detalles de las operaciones realizadas dentro del recurso. Sin embargo, a diferencia de las métricas de plataforma, debe configurar los registros de recursos que se van a recopilar. Cree una configuración de diagnóstico para enviarlos a un área de trabajo de Log Analytics para combinarlos con los demás datos usados con los registros de Azure Monitor. Se puede usar la misma configuración de diagnóstico para enviar también las métricas de la plataforma de la mayoría de los recursos a la misma área de trabajo, lo que permite analizar datos de métricas mediante consultas de registro con otros datos recopilados.

Esta recopilación tiene un costo, por lo que debe consultar [Precios de Azure Monitor](https://azure.microsoft.com/pricing/details/monitor/) antes de implementarla en un número significativo de recursos. Consulte también [Administrar el uso y los costos con los registros de Azure Monitor](platform/manage-cost-storage.md) para obtener más información sobre cómo optimizar el costo de la recopilación de registros.

Consulte [Creación de una configuración de diagnóstico para recopilar registros y métricas en Azure](platform/diagnostic-settings.md#create-in-azure-portal) para crear una configuración de diagnóstico para un recurso de Azure. Dado que es necesario crear una configuración de diagnóstico para cada recurso de Azure, consulte [Implementación de Azure Monitor a gran escala](deploy-scale.md) para obtener más información sobre el uso de [Azure Policy](../governance/policy/overview.md) para que la configuración se cree automáticamente cada vez que se crea un recurso de Azure.

### <a name="enable-insights-and-solutions"></a>Habilitación de conclusiones y soluciones
Las conclusiones y soluciones proporcionan una supervisión especializada para un servicio o una solución en concreto. Las conclusiones utilizan las características más recientes de Azure Monitor, como los libros, por lo que debe usar una conclusión si está disponible para su servicio. Están disponibles automáticamente en todas las suscripciones de Azure, pero pueden requerir cierta configuración para una funcionalidad completa. Normalmente usarán los registros de recursos y las métricas de la plataforma que ha configurado previamente y que pueden recopilar datos adicionales.

Las soluciones se deben agregar a cada suscripción, funcionan exclusivamente con datos de registros de Azure Monitor y pueden recopilar datos de registro adicionales.

No hay ningún costo por las soluciones o conclusiones, pero puede que se le cobren los datos que recopilan.

Consulte [¿Qué supervisa Azure Monitor?](monitor-reference.md) para obtener una lista de las conclusiones y soluciones disponibles en Azure Monitor. Consulte la documentación de cada una de las configuraciones o la información de precios. 

## <a name="collect-data-from-virtual-machines"></a>Recopilación de datos de Virtual Machines

> [!NOTE]
> Consulte [Supervisión de máquinas virtuales de Azure con Azure Monitor](insights/monitor-vm-azure.md) para obtener una guía completa sobre la supervisión de máquinas virtuales con Azure Monitor. 

Las máquinas virtuales generan datos similares a otros recursos de Azure, pero necesita un agente para recopilar datos del sistema operativo invitado. Consulte [Introducción a los agentes de Azure Monitor](platform/agents-overview.md) para ver una comparación de los agentes que usa Azure Monitor. 

[Azure Monitor para VM](insights/vminsights-overview.md) usa el agente de Log Analytics y el agente de dependencia para recopilar datos del sistema operativo invitado de las máquinas virtuales, por lo que puede implementar estos agentes como parte de la implementación de esta información. Esto habilita al agente de Log Analytics para otros servicios que lo usan, como Azure Security Center.


[ ![Implementación de VM](media/deploy/deploy-azure-vm.png) ](media/deploy/deploy-azure-vm.png#lightbox)


### <a name="configure-workspace-for-azure-monitor-for-vms"></a>Configuración del área de trabajo de Azure Monitor para VM
Azure Monitor para VM requiere un área de trabajo de Log Analytics, que normalmente será la misma que la creada para recopilar datos de otros recursos de Azure. Antes de habilitar las máquinas virtuales, debe agregar la solución necesaria para Azure Monitor para VM al área de trabajo.

Consulte [Configuración del área de trabajo de Log Analytics para Azure Monitor para VM](insights/vminsights-configure-workspace.md) para obtener más información sobre la configuración del área de trabajo de Log Analytics para Azure Monitor para VM.

### <a name="enable-azure-monitor-for-vms-on-each-virtual-machine"></a>Habilitación de Azure Monitor para VM de cada máquina virtual
Una vez configurada un área de trabajo, puede habilitar cada máquina virtual mediante la instalación del agente de Log Analytics y Dependency Agent. Hay varios métodos para instalar estos agentes, incluido Azure Policy, que permite configurar automáticamente cada máquina virtual a medida que se crea. Los datos de rendimiento y los detalles del proceso recopilados por Azure Monitor para VM se almacenan en registros de Azure Monitor.

Consulte [Información general sobre la habilitación de Azure Monitor para VM](insights/vminsights-enable-overview.md) para ver las opciones de implementación de los agentes en las máquinas virtuales y habilitarlas para la supervisión.

### <a name="configure-workspace-to-collect-events"></a>Configuración del área de trabajo para recopilar eventos
Azure Monitor para VM recopilará los datos de rendimiento y los detalles y las dependencias de los procesos del sistema operativo invitado de cada máquina virtual. El agente de Log Analytics también puede recopilar registros del invitado, incluido el registro de eventos de Windows y Syslog de Linux. Recupera la configuración de estos registros del área de trabajo de Log Analytics a la que está conectado. Solo tiene que configurar el área de trabajo una vez y, cada vez que se conecte un agente, se descargarán todos los cambios de configuración. 

Consulte [Orígenes de datos de agente en Azure Monitor](platform/agent-data-sources.md) para obtener más información sobre la configuración del área de trabajo de Log Analytics para recopilar datos adicionales de las máquinas virtuales del agente.

> [!NOTE]
> También puede configurar el área de trabajo para recopilar contadores de rendimiento, pero lo más probable es que sea redundante con los datos de rendimiento recopilados por Azure Monitor para VM. Los datos de rendimiento recopilados por el área de trabajo se almacenarán en la tabla *Perf*, mientras que los datos de rendimiento recopilados por Azure Monitor para VM se almacenan en la tabla *InsightsMetrics*. Configure la recopilación de rendimiento en el área de trabajo solo si necesita contadores que no están ya recopilados por Azure Monitor para VM.

### <a name="diagnostic-extension-and-telegraf-agent"></a>Extensión de diagnósticos y agente de Telegraf
Azure Monitor para VM usa el agente de Log Analytics que envía datos de rendimiento a un área de trabajo de Log Analytics, pero no a métricas de Azure Monitor. El envío de estos datos a las métricas permite analizarlos con Explorador de métricas y usarlos con las alertas de métricas. Esto requiere la extensión de diagnósticos en Windows y el agente de Telegraf en Linux.

Consulte [Instalación y configuración de la extensión de Azure Diagnostics (WAD) para Windows](platform/diagnostics-extension-windows-install.md) y [Recopilación de métricas personalizadas para una máquina virtual Linux con el agente de InfluxData Telegraf](platform/collect-custom-metrics-linux-telegraf.md) para obtener más información sobre la instalación y configuración de estos agentes.


## <a name="monitor-applications"></a>Supervisión de aplicaciones
Azure Monitor supervisa las aplicaciones personalizadas con [Application Insights](app/app-insights-overview.md), que debe configurar para cada aplicación que quiera supervisar. El proceso de configuración variará en función del tipo de aplicación que se esté supervisando y del tipo de supervisión que quiera realizar. Los datos recopilados por Application Insights se almacenan en las métricas de Azure Monitor, los registros de Azure Monitor y Azure Blob Storage, en función de la característica. Los datos de rendimiento se almacenan en las métricas de Azure Monitor y los registros de Azure Monitor sin necesidad de configuración adicional.

### <a name="create-an-application-resource"></a>Creación de un recurso de aplicación
Debe crear un recurso en Application Insights para cada aplicación que vaya a supervisar. Los datos de registro recopilados por Application Insights se almacenan en los registros de Azure Monitor, pero de forma independiente del área de trabajo de Log Analytics, tal y como se describe en [¿Cómo se estructuran los registros de Azure Monitor?](platform/data-platform-logs.md#how-is-data-in-azure-monitor-logs-structured). Actualmente está en versión preliminar, aunque es la capacidad de almacenar los datos de la aplicación directamente en un área de trabajo de Log Analytics con los demás datos. Esto simplifica la configuración y permite que la aplicación aproveche todas las características de un área de trabajo de Log Analytics.

 Al crear la aplicación, debe seleccionar si quiere usar la versión clásica o basada en el área de trabajo (versión preliminar). Consulte [Creación de recursos en Application Insights](app/create-new-resource.md) para crear una aplicación clásica. Consulte [Recursos de Application Insights basados en área de trabajo (versión preliminar)](app/create-workspace-resource.md) para crear una aplicación basada en el área de trabajo.

### <a name="configure-codeless-or-code-based-monitoring"></a>Configuración de la supervisión no codificada o basada en código
Para habilitar la supervisión de una aplicación, debe decidir si utilizará la supervisión sin código o basada en código. El proceso de configuración variará en función de esta decisión y del tipo de aplicación que vaya a supervisar.

La **supervisión sin código** es más fácil de implementar y se puede configurar después del desarrollo del código. No requiere ninguna actualización en el código. Consulte los siguientes recursos para obtener más información sobre cómo habilitar la supervisión en función de la aplicación.

- [Aplicaciones hospedadas en Azure Web Apps](app/azure-web-apps.md)
- [Aplicaciones de Java](app/java-in-process-agent.md)
- [Aplicaciones ASP.NET hospedadas en IIS en una máquina virtual de Azure o un conjunto de escalado de máquinas virtuales de Azure](app/azure-vm-vmss-apps.md)
- [Aplicaciones ASP.NET hospedadas en una máquina virtual en el entorno local de IIS](app/monitor-performance-live-website-now.md)


La **supervisión basada en código** es más personalizable y recopila telemetría adicional, pero requiere agregar una dependencia al código en los paquetes NuGet del SDK de Application Insights. Consulte los siguientes recursos para obtener más información sobre cómo habilitar la supervisión en función de la aplicación.

- [Aplicaciones ASP.NET](app/asp-net.md)
- [Aplicación ASP.NET Core](app/asp-net-core.md)
- [Aplicaciones de consola .NET](app/console.md)
- [Java](app/java-get-started.md)
- [Node.js](app/nodejs.md)
- [Python](app/opencensus-python.md)
- [Otras plataformas](app/platforms.md)

### <a name="configure-availability-testing"></a>Configuración de las pruebas de disponibilidad
Las pruebas de disponibilidad en Application Insights son pruebas periódicas que supervisan la disponibilidad y la capacidad de respuesta de la aplicación a intervalos regulares desde puntos de todo el mundo. Puede crear una prueba de ping simple de forma gratuita o crear una secuencia de solicitudes web para simular las transacciones de usuario con un costo asociado. 

Consulte [Supervisión de la disponibilidad de un sitio web](app/monitor-web-app-availability.md) para obtener un resumen de los diferentes tipos de pruebas y detalles sobre cómo crearlas.

### <a name="configure-profiler"></a>Configuración de Profiler
Profiler en Application Insights proporciona seguimientos de rendimiento para aplicaciones .NET. Le ayuda a identificar la ruta de acceso "activa" al código que tarda más tiempo cuando se atiende una solicitud web determinada. El proceso para configurar el generador de perfiles varía en función del tipo de aplicación. 

Consulte [Generación de perfiles de aplicaciones de producción en Azure con Application Insights](app/profiler-overview.md) para obtener más información sobre la configuración de Profiler.

### <a name="configure-snapshot-debugger"></a>Configuración de Snapshot Debugger
Snapshot Debugger en Application Insights supervisa la telemetría de excepciones de la aplicación .NET y recopila instantáneas en las principales excepciones que se producen para que tenga la información que necesita para diagnosticar problemas en producción. El proceso para configurar Snapshot Debugger varía en función del tipo de aplicación. 

Consulte [Depurar instantáneas cuando se producen excepciones en aplicaciones de .NET](app/snapshot-debugger.md) para obtener más información sobre la configuración de Snapshot Debugger.


## <a name="visualize-data"></a>Visualización de datos
Las conclusiones y las soluciones incluirán sus propios libros y vistas para analizar sus datos. Además, puede crear sus propias [visualizaciones](visualizations.md), incluidos libros de datos y paneles de Azure Monitor para combinar los datos de Azure Monitor con datos de otros servicios de Azure.


### <a name="create-workbooks"></a>Creación de libros
Los [libros](platform/workbooks-overview.md) de Azure Monitor permiten crear informes visuales enriquecidos en Azure Portal. Puede combinar diferentes conjuntos de datos de métricas de Azure Monitor y registros de Azure Monitor para crear experiencias interactivas unificadas. Puede tener acceso a una galería de libros en la pestaña **Libros** del menú de Azure Monitor. 

Consulte [Libros de Azure Monitor](platform/workbooks-overview.md) para obtener más información sobre la creación de libros personalizados.

### <a name="create-dashboards"></a>Crear paneles
Los [paneles de Azure](../azure-portal/azure-portal-dashboards.md) son la principal tecnología de paneles de Azure y permiten combinar datos de Azure Monitor con datos de otros servicios para proporcionar un panel único a través de la infraestructura de Azure. Consulte [Creación y uso compartido de paneles de datos de Log Analytics](learn/tutorial-logs-dashboards.md) para obtener más información sobre cómo crear un panel que incluya datos de registros de Azure Monitor. 

Consulte [Creación de paneles de indicadores clave de rendimiento (KPI) personalizados con Azure Application Insights](learn/tutorial-app-dashboards.md) para más información sobre cómo crear un panel que incluya datos de Application Insights. 

## <a name="alerts"></a>Alertas
Las alertas de Azure Monitor notifican al usuario proactivamente sobre datos y patrones importantes identificados en los datos de supervisión. Algunas conclusiones generarán alertas sin configuración. En el caso de otros escenarios, debe crear [reglas de alertas](platform/alerts-overview.md) que incluyan los datos que se van a analizar y los criterios sobre cuándo se debe generar una alerta, así como los grupos de acciones que definen la acción que se realizará cuando se genere una alerta. 


### <a name="create-action-groups"></a>Creación de grupos de acciones
Los [grupos de acciones](platform/action-groups.md) son una recopilación de las preferencias de notificación que usan las reglas de alerta para determinar la acción que se realizará cuando se desencadene una alerta. Entre los ejemplos de acciones se incluyen el envío de un correo electrónico o un SMS, la llamada a un webhook o el envío de datos a una herramienta de ITSM. Cada regla de alerta requiere al menos un grupo de acciones, del mismo modo que varias reglas de alerta pueden usar un solo grupo de acciones.

Consulte [Creación y administración de grupos de acciones en Azure Portal](platform/action-groups.md) para más información sobre cómo crear un grupo de acciones y una descripción de las distintas acciones que puede incluir.


### <a name="create-alert-rules"></a>Creación de reglas de alerta
Hay varios tipos de reglas de alerta definidos por el tipo de datos que usan. Cada una tiene capacidades y costos diferentes. La estrategia básica que debe seguir es usar el tipo de regla de alerta con el costo más bajo que proporciona la lógica que necesita.

- [Reglas del registro de actividad](platform/activity-log-alerts.md). Crea una alerta en respuesta a un nuevo evento de registro de actividad que coincide con las condiciones especificadas. No hay ningún costo en estas alertas, por lo que deben ser la primera opción. Consulte [Crear, ver y administrar las alertas del registro de actividad mediante Azure Monitor](platform/alerts-activity-log.md) para más detalles sobre la creación de alertas de registro de actividad.
- [Reglas de alertas de métricas](platform/alerts-metric-overview.md). Crea una alerta en respuesta a uno o varios valores de métrica que superan un umbral. Las alertas de métricas tienen estado, lo que significa que la alerta se cerrará automáticamente cuando el valor caiga por debajo del umbral y solo enviará notificaciones cuando cambie el estado. Hay un costo para las alertas de métricas, pero es significativamente menor que para las alertas de registro. Consulte [Creación, visualización y administración de alertas de métricas mediante Azure Monitor](platform/alerts-metric.md) para más detalles sobre la creación de alertas de métricas.
- [Reglas de alertas de registro](platform/alerts-unified-log.md). Crea una alerta cuando los resultados de una consulta de programación coinciden con los criterios especificados. Son las reglas de alertas más caras, pero permiten los criterios más complejos. Consulte [Creación, visualización y administración de alertas de registro mediante Azure Monitor](platform/alerts-log.md) para más detalles sobre la creación de alertas de consulta de registros.
- Las [alertas de aplicación](app/monitor-web-app-availability.md) permiten realizar pruebas proactivas de rendimiento y disponibilidad de la aplicación web. Puede realizar una prueba de ping simple sin costo alguno, pero hay un costo para las pruebas más complejas. Consulte [Supervisión de la disponibilidad de un sitio web](app/monitor-web-app-availability.md) para obtener una descripción de las diferentes pruebas y detalles sobre cómo crearlas.


## <a name="next-steps"></a>Pasos siguientes

- Consulte [Implementación de Azure Monitor a gran escala mediante Azure Policy](deploy-scale.md).