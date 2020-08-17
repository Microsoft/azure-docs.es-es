---
title: Línea de base de seguridad de Azure para API Management
description: Línea de base de seguridad de Azure para API Management
author: msmbaldwin
ms.service: security
ms.topic: conceptual
ms.date: 05/04/2020
ms.author: mbaldwin
ms.custom: security-benchmark
ms.openlocfilehash: 4e951b7eb2d3d5e49ab80af2604d5b44565dd25e
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87008547"
---
# <a name="azure-security-baseline-for-api-management"></a>Línea de base de seguridad de Azure para API Management

La línea de base de seguridad de Azure para API Management contiene recomendaciones que le ayudarán a mejorar la situación de seguridad de una implementación.

La base de referencia de este servicio se extrae de la [versión 1.0 de Azure Security Benchmark](../security/benchmarks/overview.md), que proporciona recomendaciones sobre cómo puede proteger las soluciones en la nube en Azure con nuestra guía de procedimientos recomendados.

Para más información, consulte [Introducción a la línea de base de seguridad de Azure](../security/benchmarks/security-baselines-overview.md).

## <a name="network-security"></a>Seguridad de las redes

*Para más información, consulte [Control de seguridad: Seguridad de redes](../security/benchmarks/security-control-network-security.md).*

### <a name="11-protect-resources-using-network-security-groups-or-azure-firewall-on-your-virtual-network"></a>1.1: Proteja los recursos mediante grupos de seguridad de red o Azure Firewall en su red virtual

**Guía**: Azure API Management se puede implementar dentro de una red virtual (Vnet) de Azure, por lo que puede acceder a los servicios de back-end de la red. El portal para desarrolladores y la puerta de enlace de API Management pueden configurarse para que sean accesibles desde Internet (de forma externa) o solo dentro de la red virtual (de forma interna).
- De forma externa: la puerta de enlace de API Management y el portal para desarrolladores son accesibles públicamente desde Internet con un equilibrador de carga externo. La puerta de enlace puede acceder a recursos dentro de la red virtual.
- De forma interna: la puerta de enlace de API Management y el portal para desarrolladores solo son accesibles desde la red virtual con un equilibrador de carga interno. La puerta de enlace puede acceder a recursos dentro de la red virtual.

El tráfico entrante y saliente en la subred en la que se implementa API Management puede controlarse mediante un grupo de seguridad de red.

* [Usar Azure API Management con redes virtuales](./api-management-using-with-vnet.md)

* [Uso del servicio Azure API Management con una red virtual interna](./api-management-using-with-internal-vnet.md)

* [Integración de API Management en una red virtual interna con Application Gateway](./api-management-howto-integrate-internal-vnet-appgateway.md)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="12-monitor-and-log-the-configuration-and-traffic-of-vnets-subnets-and-nics"></a>1.2: Supervise y registre la configuración y el tráfico de redes virtuales, subredes y NIC

**Guía**: El tráfico entrante y saliente en la subred en la que se implementa API Management puede controlarse mediante grupos de seguridad de red. Implemente un grupo de seguridad de red en la subred de API Management, habilite los registros de flujo de dicho grupo y envíe registros a una cuenta de Azure Storage para la auditoría del tráfico. También puede enviar registros de flujo de grupo de seguridad de red a un área de trabajo de Log Analytics y usar el Análisis de tráfico para proporcionar información detallada sobre el flujo de tráfico en la nube de Azure. Algunas de las ventajas del Análisis de tráfico son la capacidad de visualizar la actividad de la red e identificar las zonas activas, identificar las amenazas de seguridad, comprender los patrones de flujo de tráfico y detectar configuraciones de red incorrectas.

Precaución: Al configurar un grupo de seguridad de red en la subred de API Management, hay un conjunto de puertos que deben estar abiertos. Si alguno de estos puertos no está disponible, es posible que API Management no funcione correctamente y sea inaccesible.

* [Descripción de las configuraciones de grupos de seguridad de red para Azure API Management](./api-management-using-with-vnet.md#-common-network-configuration-issues)

* [Habilitación de los registros de flujo de NSG](../network-watcher/network-watcher-nsg-flow-logging-portal.md)

* [Habilitación y uso del Análisis de tráfico](../network-watcher/traffic-analytics.md)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="13-protect-critical-web-applications"></a>1.3: Proteja las aplicaciones web críticas

**Guía**: Para proteger las API Web y HTTP esenciales, configure API Management dentro de una red virtual (Vnet) en modo interno y configure una instancia de Azure Application Gateway. Application Gateway es un servicio PaaS. Actúa como un proxy inverso y proporciona equilibrio de carga L7, enrutamiento, firewall de aplicaciones web (WAF) y otros servicios.

La combinación de una instancia de API Management aprovisionada en una red virtual interna con el front-end de Application Gateway permite los siguientes escenarios:
- Uso de un único recurso de API Management para exponer todas las API a los consumidores internos y externos.
- Uso de un único recurso de API Management para exponer un subconjunto de API a los consumidores externos.
- Proporcionar una forma de activar y desactivar el acceso a API Management desde la red pública de Internet.

Nota: Esta característica está disponible únicamente en los niveles Premium y Desarrollador de API Management.

* [Integración de API Management en una red virtual interna con Application Gateway](./api-management-howto-integrate-internal-vnet-appgateway.md)

* [¿Qué es Azure Application Gateway?](../application-gateway/index.yml)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="14-deny-communications-with-known-malicious-ip-addresses"></a>1.4: Deniegue las comunicaciones con direcciones IP malintencionadas conocidas

**Guía**: Configure API Management dentro de una red virtual (Vnet) en modo interno y configure una instancia de Azure Application Gateway. Application Gateway es un servicio PaaS. Actúa como un proxy inverso y proporciona equilibrio de carga L7, enrutamiento, firewall de aplicaciones web (WAF) y otros servicios.

La combinación de una instancia de API Management aprovisionada en una red virtual interna con el front-end de Application Gateway permite los siguientes escenarios:
- Uso de un único recurso de API Management para exponer todas las API a los consumidores internos y externos.
- Uso de un único recurso de API Management para exponer un subconjunto de API a los consumidores externos.
- Proporcionar una forma de activar y desactivar el acceso a API Management desde la red pública de Internet.

Nota: Esta característica está disponible únicamente en los niveles Premium y Desarrollador de API Management.

Habilite Azure DDoS Protection Estándar en la red virtual asociada a su implementación de API Management para protegerse frente a ataques de denegación de servicio distribuido (DDoS).

Use la inteligencia sobre amenazas integrada de Azure Security Center para denegar las comunicaciones con direcciones IP malintencionadas conocidas o no utilizadas.

* [Integración de API Management en una red virtual interna con Application Gateway](./api-management-howto-integrate-internal-vnet-appgateway.md)

* [¿Qué es Azure Application Gateway?](../application-gateway/index.yml)

* [Configuración de Azure DDoS Protection Estándar](../virtual-network/manage-ddos-protection.md)

* [Descripción de la inteligencia sobre amenazas integrada de Azure Security Center](../security-center/threat-protection.md)

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="15-record-network-packets-and-flow-logs"></a>1.5: Registre los paquetes de red y registros de flujo

**Guía**: El tráfico entrante y saliente en la subred en la que se implementa API Management puede controlarse mediante grupos de seguridad de red. Implemente un grupo de seguridad de red en la subred de API Management, habilite los registros de flujo de dicho grupo y envíe registros a una cuenta de Azure Storage para la auditoría del tráfico. También puede enviar registros de flujo de grupo de seguridad de red a un área de trabajo de Log Analytics y usar el Análisis de tráfico para proporcionar información detallada sobre el flujo de tráfico en la nube de Azure. Algunas de las ventajas del Análisis de tráfico son la capacidad de visualizar la actividad de la red e identificar las zonas activas, identificar las amenazas de seguridad, comprender los patrones de flujo de tráfico y detectar configuraciones de red incorrectas.

Precaución: Al configurar un grupo de seguridad de red en la subred de API Management, hay un conjunto de puertos que deben estar abiertos. Si alguno de estos puertos no está disponible, es posible que API Management no funcione correctamente y sea inaccesible.

* [Descripción de las configuraciones de grupos de seguridad de red para Azure API Management](./api-management-using-with-vnet.md#-common-network-configuration-issues)

* [Habilitación de los registros de flujo de NSG](../network-watcher/network-watcher-nsg-flow-logging-portal.md)

* [Habilitación y uso del Análisis de tráfico](../network-watcher/traffic-analytics.md)

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="16-deploy-network-based-intrusion-detectionintrusion-prevention-systems-idsips"></a>1.6: Implemente sistemas de prevención de intrusiones y detección de intrusiones (IDS/IPS) basados en la red

**Guía**: Configure API Management dentro de una red virtual (Vnet) en modo interno y configure una instancia de Azure Application Gateway. Application Gateway es un servicio PaaS. Actúa como un proxy inverso y proporciona equilibrio de carga L7, enrutamiento, firewall de aplicaciones web (WAF) y otros servicios. El WAF de Application Gateway proporciona protección contra puntos vulnerables de seguridad y ataques de seguridad comunes, y puede ejecutarse en los dos modos siguientes:
- Modo de detección: supervisa y registra todas las alertas de amenazas. Puede activar el registro de diagnósticos de Application Gateway en la sección Diagnósticos. Debe asegurarse de que el registro de WAF está seleccionado y activado. El firewall de aplicaciones web no bloquea solicitudes entrantes cuando se ejecuta en modo de detección.
- Modo de prevención: bloquea las intrusiones y los ataques que detectan las reglas. El atacante recibe una excepción de "403: acceso no autorizado" y se cierra la conexión. El modo de prevención registra dichos ataques en los registros del WAF.

La combinación de una instancia de API Management aprovisionada en una red virtual interna con el front-end de Application Gateway permite los siguientes escenarios:
- Uso de un único recurso de API Management para exponer todas las API a los consumidores internos y externos.
- Uso de un único recurso de API Management para exponer un subconjunto de API a los consumidores externos.
- Proporcionar una forma de activar y desactivar el acceso a API Management desde la red pública de Internet.

Nota: Esta característica está disponible únicamente en los niveles Premium y Desarrollador de API Management.

* [Integración de API Management en una red virtual interna con Application Gateway](./api-management-howto-integrate-internal-vnet-appgateway.md)

* [Firewall de aplicaciones web de Azure en Azure Application Gateway](../web-application-firewall/ag/ag-overview.md)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="17-manage-traffic-to-web-applications"></a>1.7: Administre el tráfico a las aplicaciones web

**Instrucciones**: Para administrar el tráfico que fluye a las API Web o HTTP, implemente API Management en una red virtual (Vnet) asociada con App Service Environment en modo externo o interno.

En modo interno, configure una instancia de Azure Application Gateway delante de API Management. Application Gateway es un servicio PaaS. Actúa como un proxy inverso y proporciona equilibrio de carga L7, enrutamiento, firewall de aplicaciones web (WAF) y otros servicios. El WAF de Application Gateway proporciona protección contra puntos vulnerables y problemas de seguridad comunes.

La combinación de una instancia de API Management aprovisionada en una red virtual interna con el front-end de Application Gateway permite los siguientes escenarios:
- Uso de un único recurso de API Management para exponer todas las API a los consumidores internos y externos.
- Uso de un único recurso de API Management para exponer un subconjunto de API a los consumidores externos.
- Proporcionar una forma de activar y desactivar el acceso a API Management desde la red pública de Internet.

Nota: Esta característica está disponible únicamente en los niveles Premium y Desarrollador de API Management.

* [Publicación de API privadas para consumidores externos](/azure/architecture/example-scenario/apps/publish-internal-apis-externally)

* [Uso de Azure API Management con una red virtual](./api-management-using-with-vnet.md)

* [Firewall de aplicaciones web de Azure en Azure Application Gateway](../web-application-firewall/ag/ag-overview.md)

* [¿Qué es Azure Application Gateway?](../application-gateway/overview.md)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="18-minimize-complexity-and-administrative-overhead-of-network-security-rules"></a>1.8: Minimice la complejidad y la sobrecarga administrativa de las reglas de seguridad de red

**Guía**: Use etiquetas de servicio de Virtual Network (Vnet) para definir controles de acceso a la red en los grupos de seguridad de red que se usan en las subredes de API Management. Puede utilizar etiquetas de servicio en lugar de direcciones IP específicas al crear reglas de seguridad. Al especificar el nombre de la etiqueta de servicio (por ejemplo, ApiManagement) en el campo de origen o destino apropiado de una regla, puede permitir o denegar el tráfico para el servicio correspondiente. Microsoft administra los prefijos de direcciones que la etiqueta de servicio incluye y actualiza automáticamente dicha etiqueta a medida que las direcciones cambian.

Precaución: Al configurar un grupo de seguridad de red en la subred de API Management, hay un conjunto de puertos que deben estar abiertos. Si alguno de estos puertos no está disponible, es posible que API Management no funcione correctamente y sea inaccesible.

* [Descripción y uso de las etiquetas de servicio](../virtual-network/service-tags-overview.md)

* [Puertos necesarios para API Management](./api-management-using-with-vnet.md#-common-network-configuration-issues)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="19-maintain-standard-security-configurations-for-network-devices"></a>1.9: Mantenga las configuraciones de seguridad estándar para dispositivos de red

**Guía**: Defina e implemente configuraciones de seguridad estándar para la configuración de red relacionada con las implementaciones de Azure API Management. Use alias de Azure Policy en los espacios de nombres "Microsoft.ApiManagement" y "Microsoft.Network" para crear directivas personalizadas con el fin de auditar o aplicar la configuración de red de las implementaciones de Azure API Management y los recursos relacionados. También puede usar definiciones de directivas integradas para Azure Virtual Network, como:
- Se debe habilitar DDoS Protection estándar

También puede utilizar Azure Blueprints para simplificar las implementaciones de Azure a gran escala mediante el empaquetado de artefactos de entorno clave, como plantillas de Azure Resource Manager (ARM), control de acceso basado en rol (RBAC) y directivas, en una única definición de plano técnico. Aplique fácilmente el plano técnico a nuevas suscripciones y entornos, y ajuste el control y la administración mediante el control de versiones.

* [Configuración y administración de Azure Policy](../governance/policy/tutorials/create-and-manage.md)

* [Creación de un plano técnico de Azure](../governance/blueprints/create-blueprint-portal.md)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="110-document-traffic-configuration-rules"></a>1.10: Documente las reglas de configuración de tráfico

**Guía**: Use etiquetas para grupos de seguridad de red y otros recursos relacionados con la seguridad de red y el flujo de tráfico. En el caso de las reglas de grupo de seguridad de red individuales, puede usar el campo "Descripción" para especificar las necesidades empresariales o la duración (etc.) de las reglas que permiten que entre o salga el tráfico en una red.

* [Creación y uso de etiquetas](../azure-resource-manager/management/tag-resources.md)

* [Creación de una red virtual](../virtual-network/quick-create-portal.md)

* [Creación de un grupo de seguridad de red con una configuración de seguridad](../virtual-network/tutorial-filter-network-traffic.md)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="111-use-automated-tools-to-monitor-network-resource-configurations-and-detect-changes"></a>1.11: Use herramientas automatizadas para supervisar las configuraciones de recursos de red y detectar cambios

**Guía**: Use el registro de actividad de Azure para supervisar las configuraciones de los recursos de red y detectar cambios en estos asociados a las implementaciones de Azure API Management. Cree alertas en Azure Monitor que se desencadenarán cuando se produzcan cambios en los recursos de red críticos.

* [Visualización y recuperación de eventos del registro de actividad de Azure](../azure-monitor/platform/activity-log.md#view-the-activity-log)

* [Creación de alertas en Azure Monitor](../azure-monitor/platform/alerts-activity-log.md)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

## <a name="logging-and-monitoring"></a>Registro y supervisión

*Para más información, consulte [Control de seguridad: registro y supervisión](../security/benchmarks/security-control-logging-monitoring.md).*

### <a name="21-use-approved-time-synchronization-sources"></a>2.1: Uso de orígenes de sincronización de hora aprobados

**Guía**: Microsoft mantiene los orígenes de hora para Azure API Management.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Microsoft

### <a name="22-configure-central-security-log-management"></a>2.2: Configuración de la administración central de registros de seguridad

**Guía**: En Azure Monitor, use las áreas de trabajo de Log Analytics para consultar y realizar análisis, enviar registros a Azure Storage para el almacenamiento de archivos a largo plazo o para el análisis sin conexión, o exportar registros a otra solución de análisis en Azure y en otros entornos con Azure Event Hubs. De forma predeterminada, Azure API Management genera registros y métricas para Azure Monitor. El nivel de detalle del registro se puede configurar en todo el servicio y para cada API.

Además de Azure Monitor, Azure API Management puede integrarse con uno o varios servicios de Azure Application Insights. La configuración del registro para Application Insights se puede configurar en función de cada servicio o para cada API.

Como alternativa, puede habilitar e incorporar datos en Azure Sentinel o en una herramienta de Administración de eventos e información de seguridad (SIEM) de terceros.

* [Configuración del diagnóstico](../azure-monitor/platform/diagnostic-settings.md#create-in-azure-portal)

* [Incorporación de Azure Sentinel](../sentinel/quickstart-onboard.md)

* [Introducción a Azure Monitor e integración con herramienta SIEM de terceros](https://azure.microsoft.com/blog/use-azure-monitor-to-integrate-with-siem-tools/)

* [Creación de una canalización de análisis y registros personalizados](./api-management-howto-log-event-hubs.md)

* [Cómo integrar Azure API Management con Azure Application Insights](./api-management-howto-app-insights.md)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="23-enable-audit-logging-for-azure-resources"></a>2.3: Habilitación del registro de auditoría para recursos de Azure

**Guía**: Para el registro de auditoría de planos de control, habilite la configuración de diagnóstico del registro de actividad de Azure y envíe registros de actividad a un área de trabajo de Log Analytics para los informes y los análisis, a Azure Storage para la protección a largo plazo, a Azure Event Hubs para la exportación a otras soluciones de análisis en Azure y otros entornos. Con los datos del registro de actividad de Azure, puede responder a las preguntas "qué, quién y cuándo" de las operaciones de escritura (PUT, POST, DELETE) llevadas a cabo en el nivel del plano de control para el servicio de Azure API Management.

Para el registro de auditoría de planos de datos, los registros de diagnóstico proporcionan información valiosa acerca de las operaciones y los errores que son importantes para la auditoría, así como para solucionar problemas. Los registros de diagnóstico son diferentes de los registros de actividad. Los registros de actividad proporcionan información sobre las operaciones llevadas a cabo en los recursos de Azure. Los registros de diagnóstico proporcionan información detallada acerca de las operaciones que el recurso ha realizado.

* [Habilitación de la configuración de diagnóstico para el registro de actividad de Azure](../azure-monitor/platform/activity-log.md)

* [Habilitación de la configuración de diagnóstico para Azure API Management](./api-management-howto-use-azure-monitor.md#activity-logs)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="24-collect-security-logs-from-operating-systems"></a>2.4: Recopilación de registros de seguridad de sistemas operativos

**Instrucciones**: No aplicable; esta recomendación está pensada para los recursos de proceso.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="25-configure-security-log-storage-retention"></a>2.5: Configuración de la retención del almacenamiento de registros de seguridad

**Guía**: En Azure Monitor, establezca el período de retención del área de trabajo de Log Analytics de acuerdo con la normativa de cumplimiento de su organización. Use cuentas de Azure Storage para el almacenamiento de archivos a largo plazo.

* [Configuración de parámetros de retención de registros de áreas de trabajo de Log Analytics](../azure-monitor/platform/manage-cost-storage.md#change-the-data-retention-period)

* [Archivado de registros en una cuenta de Azure Storage](../azure-monitor/platform/resource-logs.md#send-to-azure-storage)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="26-monitor-and-review-logs"></a>2.6: Supervisión y registros de revisión

**Guía**: Azure API Management emite continuamente registros y métricas a Azure Monitor, lo que le permite tener una visibilidad casi en tiempo real del estado general y también del estado de las API. Con las áreas de trabajo de Azure Monitor y Log Analytics, puede revisar alertas, consultarlas, visualizarlas, enrutarlas, archivarlas y configurarlas, así como realizar acciones en las métricas y los registros procedentes de API Management y los recursos relacionados. analice y supervise los registros en busca de comportamientos anómalos y revise los resultados con regularidad.

Si lo desea, puede integrar API Management con Azure Application Insights y usarla como herramienta principal o secundaria de supervisión, seguimiento, informes y alertas.

* [Supervisión y revisión de los registros de Azure API Management](./api-management-howto-use-azure-monitor.md)

* [Procedimiento para realizar consultas personalizadas en Azure Monitor](../azure-monitor/log-query/get-started-queries.md)

* [Descripción del área de trabajo de Log Analytics](../azure-monitor/log-query/get-started-portal.md)

* [Cómo integrar Azure API Management con Azure Application Insights](./api-management-howto-app-insights.md)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="27-enable-alerts-for-anomalous-activity"></a>2.7: Habilitación de alertas para actividades anómalas

**Guía**: Habilite la configuración de diagnóstico del registro de actividad de Azure, así como la configuración de diagnóstico de las instancias de Azure API Management y envíe los registros a un área de trabajo de Log Analytics. Realice consultas en Log Analytics para buscar términos, identificar tendencias, analizar patrones y proporcionar otra información detallada basada en los datos recopilados. Puede crear alertas basadas en las consultas del área de trabajo de Log Analytics.

Cree alertas de métricas que le avisen cuando ocurra algo inesperado. Por ejemplo, reciba notificaciones cuando la instancia de Azure API Management haya superado la capacidad máxima esperada durante un período de tiempo determinado o si ha habido un cierto número de solicitudes de puerta de enlace no autorizadas o errores durante un período de tiempo predefinido.

Si lo desea, puede integrar API Management con Azure Application Insights y usarla como herramienta principal o secundaria de supervisión, seguimiento, informes y alertas.

Si lo desea, también puede habilitar e incorporar datos en Azure Sentinel o en una herramienta de Administración de eventos e información de seguridad (SIEM) de terceros.

* [Habilitación de la configuración de diagnóstico para el registro de actividad de Azure](../azure-monitor/platform/activity-log.md)

* [Habilitación de la configuración de diagnóstico para Azure API Management](./api-management-howto-use-azure-monitor.md#activity-logs)

* [Configuración de una regla de alerta para solicitudes no autorizadas](./api-management-howto-use-azure-monitor.md#set-up-an-alert-rule-for-unauthorized-request)

* [Visualización de las métricas de capacidad de una instancia de Azure API Management](./api-management-capacity.md)

* [Cómo integrar Azure API Management con Azure Application Insights](./api-management-howto-app-insights.md)

* [Incorporación de Azure Sentinel](../sentinel/quickstart-onboard.md)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="28-centralize-anti-malware-logging"></a>2.8: Centralización del registro antimalware

**Guía**: No aplicable; Azure API Management no procesa ni genera registros relacionados con antimalware.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="29-enable-dns-query-logging"></a>2.9: Habilitación del registro de consultas DNS

**Guía**: No aplicable; Azure API Management no procesa ni genera registros relacionados con DNS a los que el usuario pueda acceder.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="210-enable-command-line-audit-logging"></a>2.10: Habilitación del registro de auditoría de la línea de comandos

**Instrucciones**: No aplicable; esta recomendación está pensada para los recursos de proceso.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

## <a name="identity-and-access-control"></a>Control de identidades y acceso

*Para más información, consulte [Control de seguridad: Identidad y control de acceso](../security/benchmarks/security-control-identity-access-control.md).*

### <a name="31-maintain-an-inventory-of-administrative-accounts"></a>3.1: Mantenga un inventario de cuentas administrativas

**Guía**: Mantenga un inventario de cuentas que tengan acceso administrativo al plano de control de Azure API Management (Azure Portal).

Azure Active Directory (AD) tiene roles integrados que se deben asignar explícitamente y son consultables. Azure API Management se basa en estos roles y en el control de acceso basado en rol para habilitar la administración de acceso detallada para servicios y entidades de API Management.

Además, API Management contiene un grupo de administradores integrado en el sistema de usuario de API Management. Los grupos de API Management controlan la visibilidad de las API en el portal para desarrolladores y los miembros del grupo de administradores pueden ver todas las API.

Siga las recomendaciones de Azure Security Center para la administración y el mantenimiento de las cuentas administrativas.

* [Uso del control de acceso basado en rol en Azure API Management](./api-management-role-based-access-control.md)

* [Obtención de una lista de usuarios en una instancia de Azure API Management](/powershell/module/az.apimanagement/get-azapimanagementuser?view=azps-3.1.0)

* [Obtención de una lista de usuarios asignados a un rol de directorio en Azure AD con PowerShell](/powershell/module/az.resources/get-azroleassignment?view=azps-3.7.0)

* [Obtención de un rol de directorio en Azure AD con PowerShell](/powershell/module/az.resources/get-azroledefinition?view=azps-3.7.0)

* [Recomendaciones de identidad y acceso de Azure Security Center](../security-center/recommendations-reference.md#recs-identity)

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="32-change-default-passwords-where-applicable"></a>3.2: Cambie las contraseñas predeterminadas cuando proceda

**Guía**: Azure API Management no tiene el concepto de contraseñas o claves predeterminadas.

Sin embargo, las suscripciones de Azure API Management, que son un medio para proteger el acceso a las API, incluyen un par de claves de suscripción generadas. Los clientes pueden volver a generar estas claves de suscripción en cualquier momento.

* [Suscripciones en Azure API Management](./api-management-subscriptions.md)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="33-use-dedicated-administrative-accounts"></a>3.3: Use cuentas administrativas dedicadas

**Guía**: Cree procedimientos operativos estándar en torno al uso de cuentas administrativas dedicadas. Use la administración de identidad y acceso de Azure Security Center para supervisar el número de cuentas administrativas.

Además, para ayudarle a realizar un seguimiento de las cuentas administrativas dedicadas, puede seguir las recomendaciones de Azure Security Center o las directivas integradas de Azure Policy, por ejemplo:
- Debe haber más de un propietario asignado a su suscripción
- Las cuentas en desuso con permisos de propietario deben quitarse de la suscripción
- Las cuentas externas con permisos de propietario deben quitarse de la suscripción

* [Uso de Azure Security Center para supervisar la identidad y el acceso (versión preliminar)](../security-center/security-center-identity-access.md)

* [Uso de Azure Policy](../governance/policy/tutorials/create-and-manage.md)

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="34-use-single-sign-on-sso-with-azure-active-directory"></a>3.4: Use el inicio de sesión único (SSO) con Azure Active Directory

**Guía**: Azure API Management puede configurarse para aprovechar Azure Active Directory como proveedor de identidades a fin de autenticar a los usuarios en el portal para desarrolladores y beneficiarse de las funcionalidades de SSO que Azure AD ofrece. Una vez configurados, los nuevos usuarios del portal para desarrolladores pueden optar por seguir en primer lugar el proceso de registro integrado mediante la autenticación con Azure AD y, a continuación, completar el proceso de registro en el portal una vez autenticados.

* [Autorización de cuentas de desarrollador mediante Azure Active Directory en Azure API Management](./api-management-howto-aad.md)

Como alternativa, el proceso de inicio de sesión o registro se puede personalizar aún más mediante la delegación. La delegación le permite usar su sitio web actual para controlar el inicio de sesión y la suscripción de los desarrolladores, así como sus suscripciones a productos, en contraposición al uso de la funcionalidad integrada en el portal para desarrolladores. Permite que el sitio web sea propietario de los datos de usuario y realice la validación de estos pasos de forma personalizada.

* [Delegación de registros de usuario y suscripciones a producto](./api-management-howto-setup-delegation.md)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="35-use-multi-factor-authentication-for-all-azure-active-directory-based-access"></a>3.5: Use la autenticación multifactor para todo el acceso basado en Azure Active Directory

**Guía**: Habilite Multi-Factor Authentication (MFA) de Azure Active Directory (AD) y siga las recomendaciones de administración de identidades y acceso de Azure Security Center.

* [Procedimiento para habilitar la MFA en Azure](../active-directory/authentication/howto-mfa-getstarted.md)

* [Supervisión de la identidad y el acceso en Azure Security Center](../security-center/security-center-identity-access.md)

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="36-use-dedicated-machines-privileged-access-workstations-for-all-administrative-tasks"></a>3.6: Use máquinas dedicadas (estaciones de trabajo de acceso con privilegios) para todas las tareas administrativas

**Instrucciones**: Utilice estaciones de trabajo de acceso con privilegios (PAW), con Multi-Factor Authentication (MFA) configurado para iniciar sesión en recursos de Azure y configurarlos.

* [Más información sobre las estaciones de trabajo con privilegios de acceso](/windows-server/identity/securing-privileged-access/privileged-access-workstations)

* [Procedimiento para habilitar la MFA en Azure](../active-directory/authentication/howto-mfa-getstarted.md)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="37-log-and-alert-on-suspicious-activity-from-administrative-accounts"></a>3.7: Registre y alerte de la actividad sospechosa desde cuentas administrativas

**Guía**: Use Privileged Identity Management (PIM) de Azure Active Directory (AD) para la generación de registros y alertas cuando se produzca una actividad sospechosa o insegura en el entorno.

Además, use las detecciones de riesgo de Azure AD para ver alertas e informes sobre el comportamiento de los usuarios de riesgo.

* [Cómo implementar Privileged Identity Management (PIM)](../active-directory/privileged-identity-management/pim-deployment-plan.md)

* [Información sobre las detecciones de riesgo de Azure AD](../active-directory/identity-protection/overview-identity-protection.md)

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="38-manage-azure-resources-from-only-approved-locations"></a>3.8: Administre los recursos de Azure solo desde ubicaciones aprobadas

**Guía**: El cliente debe usar ubicaciones con nombre de acceso condicional para permitir el acceso a Azure Portal solo desde agrupaciones lógicas específicas de intervalos de direcciones IP o países o regiones.

* [Configuración de ubicaciones con nombre en Azure](../active-directory/reports-monitoring/quickstart-configure-named-locations.md)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="39-use-azure-active-directory"></a>3.9: Uso de Azure Active Directory

**Guía**: Siempre que sea posible, use Azure AD como sistema central de autenticación y autorización. AAD protege los datos mediante un fuerte cifrado para los datos en reposo y en tránsito. Azure AD también cifra con sal, convierte en hash y almacena de forma segura las credenciales de los usuarios.

Configure el portal para desarrolladores de Azure API Management a fin de autenticar las cuentas de desarrollador mediante Azure Active Directory.

Configure la instancia de Azure API Management para proteger las API mediante el protocolo OAuth 2.0 con Azure Active Directory (Azure AD).

* [Autorización de las cuentas de desarrollador mediante Azure Active Directory en Azure API Management](./api-management-howto-aad.md)

* [Protección de una API mediante OAuth 2.0 con Azure Active Directory API Management](./api-management-howto-protect-backend-with-aad.md)

* [Creación y configuración de una instancia de AAD](../active-directory/fundamentals/active-directory-access-create-new-tenant.md)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="310-regularly-review-and-reconcile-user-access"></a>3.10: Revise y concilie regularmente el acceso de los usuarios

**Guía**: Azure Active Directory proporciona registros para ayudar a descubrir cuentas obsoletas. Los clientes pueden usar las revisiones de acceso de identidad de Azure para administrar de forma eficiente las pertenencias a grupos, el acceso a las aplicaciones empresariales y las asignaciones de roles. El acceso de los usuarios se puede revisar periódicamente para asegurarse de que solo los usuarios apropiados continúan teniendo el acceso adecuado.

Los clientes pueden mantener el inventario de cuentas de usuario de API Management y conciliar el acceso según sea necesario. En API Management, los desarrolladores son los usuarios de las API que se exponen en este servicio. De forma predeterminada, las cuentas de desarrollador recién creadas se encuentran en estado activo y se asocian al grupo de desarrolladores. Las cuentas de desarrollador que se encuentran en estado activo se pueden usar para acceder a todas las API para las que tienen suscripciones.

Los administradores pueden crear grupos personalizados o aprovechar los grupos externos en los inquilinos de Azure Active Directory asociados. Los grupos personalizados y externos pueden usarse junto con grupos del sistema en la concesión a los desarrolladores de visibilidad y acceso a productos de la API.

* [Administración de cuentas de usuario en Azure API Management](./api-management-howto-create-or-invite-developers.md)

* [Obtención de una lista de usuarios de API Management](/powershell/module/az.apimanagement/get-azapimanagementuser?view=azps-3.1.0)

* [Creación y uso de grupos para administrar cuentas de desarrollador en Azure API Management](./api-management-howto-create-groups.md)

* [Procedimiento para usar las revisiones de acceso de identidad de Azure](../active-directory/governance/access-reviews-overview.md)

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="311-monitor-attempts-to-access-deactivated-accounts"></a>3.11: Supervise los intentos de acceso a cuentas desactivadas

**Guía**: Configure la instancia de Azure API Management para autenticar las cuentas de desarrollador mediante Azure Active Directory como proveedor de identidades en Azure API Management.

Configure la instancia de Azure API Management para proteger las API mediante el protocolo OAuth 2.0 con Azure Active Directory (Azure AD).

Configure la directiva de validación de JWT para las solicitudes de API entrantes a fin de ayudar a exigir la existencia y la validez de un token válido.

Cree una configuración de diagnóstico para las cuentas de usuario de Azure AD y envíe los registros de auditoría y los registros de inicio de sesión a un área de trabajo de Log Analytics. Configure las alertas deseadas en el área de trabajo de Log Analytics. Además, puede incorporar el área de trabajo de Log Analytics a Azure Sentinel o a una herramienta SIEM de terceros.

Configure la supervisión avanzada con API Management mediante la directiva `log-to-eventhub`, capture cualquier información de contexto adicional necesaria para el análisis de seguridad y envíela a Azure Sentinel o a una herramienta SIEM de terceros.

* [Autorización de las cuentas de desarrollador mediante Azure Active Directory en Azure API Management](./api-management-howto-aad.md)

* [Protección de una API mediante OAuth 2.0 con Azure Active Directory API Management](./api-management-howto-protect-backend-with-aad.md)

* [Directivas de restricción de acceso de API Management](./api-management-access-restriction-policies.md)

* [Integración de registros de Azure AD con registros de Azure Monitor](../active-directory/reports-monitoring/howto-integrate-activity-logs-with-log-analytics.md)

* [Incorporación de Azure Sentinel](../sentinel/quickstart-onboard.md)

* [Supervisión avanzada de API](./api-management-log-to-eventhub-sample.md)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="312-alert-on-account-login-behavior-deviation"></a>3.12: Alerte de las desviaciones de comportamiento en los inicios de sesión de las cuentas

**Guía**: En caso de que se produzcan desviaciones en el comportamiento de inicio de sesión de la cuenta del plano de control (Azure Portal), use Azure Active Directory (AD) Identity Protection y las características de detección de riesgo para configurar respuestas automatizadas a las acciones sospechosas que se detecten en relación con las identidades de los usuarios. También puede hacer que Azure Sentinel ingiera los datos para investigarlos más.

* [Visualización de los inicios de sesión de riesgo de Azure AD](../active-directory/identity-protection/overview-identity-protection.md)

* [Configuración y habilitación de las directivas de riesgo de protección de identidad](../active-directory/identity-protection/howto-identity-protection-configure-risk-policies.md)

* [Incorporación de Azure Sentinel](../sentinel/quickstart-onboard.md)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="313-provide-microsoft-with-access-to-relevant-customer-data-during-support-scenarios"></a>3.13: Proporcione a Microsoft acceso a los datos pertinentes del cliente durante los escenarios de soporte técnico

**Guía**: No está disponible actualmente; Caja de seguridad del cliente no se admite actualmente en Azure API Management.

* [Lista de servicios admitidos por la Caja de seguridad del cliente](../security/fundamentals/customer-lockbox-overview.md#supported-services-and-scenarios-in-general-availability)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

## <a name="data-protection"></a>Protección de los datos

*Para más información, consulte [Control de seguridad: Protección de datos](../security/benchmarks/security-control-data-protection.md).*

### <a name="41-maintain-an-inventory-of-sensitive-information"></a>4.1: Mantenimiento de un inventario de información confidencial

**Instrucciones**: use etiquetas para ayudar a realizar el seguimiento de los recursos de Azure que almacenan o procesan información confidencial.

* [Creación y uso de etiquetas](../azure-resource-manager/management/tag-resources.md)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="42-isolate-systems-storing-or-processing-sensitive-information"></a>4.2: Aislamiento de los sistemas que almacenan o procesan información confidencial

**Instrucciones**: Implemente suscripciones y/o grupos de administración independientes para los entornos de desarrollo, prueba y producción. Las instancias de Azure API Management deben estar separadas por la red o la subred virtual y estar etiquetadas correctamente.

* [Creación de suscripciones adicionales de Azure](../cost-management-billing/manage/create-subscription.md)

* [Creación de grupos de administración](../governance/management-groups/create.md)

* [Creación y uso de etiquetas](../azure-resource-manager/management/tag-resources.md)

* [Usar Azure API Management con redes virtuales](./api-management-using-with-vnet.md)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="43-monitor-and-block-unauthorized-transfer-of-sensitive-information"></a>4.3: Supervisión y bloqueo de una transferencia no autorizada de información confidencial

**Guía**: Actualmente no disponible; las características de identificación, clasificación y prevención de pérdida de datos no están disponibles en este momento en Azure API Management.

Microsoft administra la infraestructura subyacente para Azure API Management y ha implementado controles estrictos para evitar la pérdida o exposición de los datos de los clientes.

* [Descripción de la protección de datos de los clientes en Azure](../security/fundamentals/protection-customer-data.md)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="44-encrypt-all-sensitive-information-in-transit"></a>4.4: Cifrado de toda la información confidencial en tránsito

**Guía**: Las llamadas del plano de administración se hacen mediante Azure Resource Manager con TLS. Se requiere un token web de JSON (JWT) válido. Las llamadas del plano de datos pueden estar protegidas con TLS y uno de los mecanismos de autenticación admitidos (por ejemplo, el certificado de cliente o JWT).

* [Protección de los datos en Azure API Management](./api-management-security-controls.md#data-protection)

* [Administración de la configuración de TLS en Azure API Management](./api-management-howto-manage-protocols-ciphers.md)

* [Protección de API en Azure API Management con Azure Active Directory](./api-management-howto-protect-backend-with-aad.md)

* [Protección de API en Azure API Management con Azure Active Directory B2C](./howto-protect-backend-frontend-azure-ad-b2c.md)

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Compartido

### <a name="45-use-an-active-discovery-tool-to-identify-sensitive-data"></a>4.5: Uso de una herramienta de detección activa para identificar datos confidenciales

**Guía**: Actualmente no disponible; las características de identificación, clasificación y prevención de pérdida de datos no están disponibles en este momento en Azure API Management. Etiquete las instancias de Azure API Management y los recursos relacionados que podrían procesar información confidencial como tal e implemente una solución de terceros, si es necesario, para fines de cumplimiento.

En el caso de la plataforma subyacente administrada por Microsoft, Microsoft trata todo el contenido de los clientes como confidencial y hace grandes esfuerzos para proteger a los clientes contra la pérdida y exposición de sus datos. Para garantizar la seguridad de los datos de los clientes dentro de Azure, Microsoft ha implementado y mantiene un conjunto de controles y funcionalidades eficaces de protección de datos.

* [Descripción de la protección de datos de los clientes en Azure](../security/fundamentals/protection-customer-data.md)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="46-use-azure-rbac-to-control-access-to-resources"></a>4.6: Uso de RBAC de Azure para controlar el acceso a los recursos

**Guía**: Use el control de acceso basado en roles para controlar el acceso a Azure API Management. Azure API Management se basa en el control de acceso basado en rol (RBAC) de Azure para permitir una administración de acceso pormenorizada de servicios y entidades de API Management (por ejemplo, API y directivas).

* [Uso del control de acceso basado en rol en Azure API Management](./api-management-role-based-access-control.md)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="47-use-host-based-data-loss-prevention-to-enforce-access-control"></a>4.7: Uso de la prevención de pérdida de datos basada en host para aplicar el control de acceso

**Instrucciones**: No aplicable; esta recomendación está pensada para los recursos de proceso.

Microsoft administra la infraestructura subyacente para Azure API Management y ha implementado controles estrictos para evitar la pérdida o exposición de los datos de los clientes.

* [Descripción de la protección de datos de los clientes en Azure](../security/fundamentals/protection-customer-data.md)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="48-encrypt-sensitive-information-at-rest"></a>4.8: Cifrado de información confidencial en reposo

**Guía**: Los datos confidenciales, como certificados, claves y valores con nombre secreto, se cifran con claves que administra el servicio por instancia de servicio. Todas las claves de cifrado son por instancia de servicio y las administra el servicio.

* [Descripción de la protección de datos y el cifrado en reposo con Azure API Management](./api-management-security-controls.md#data-protection)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Microsoft

### <a name="49-log-and-alert-on-changes-to-critical-azure-resources"></a>4.9: Registro y alerta de cambios en los recursos críticos de Azure

**Guía**: Use Azure Monitor con el registro de actividad de Azure a fin de crear alertas para cuando se produzcan cambios en las aplicaciones de Azure Functions y otros recursos esenciales o relacionados.

* [Creación de alertas para los eventos del registro de actividad de Azure](../azure-monitor/platform/alerts-activity-log.md)

* [Uso de Azure Monitor y el registro de actividad de Azure en Azure API Management](./api-management-howto-use-azure-monitor.md)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

## <a name="vulnerability-management"></a>Administración de vulnerabilidades

*Para más información, consulte [Control de seguridad: Administración de vulnerabilidades](../security/benchmarks/security-control-vulnerability-management.md).*

### <a name="51-run-automated-vulnerability-scanning-tools"></a>5.1: Ejecute herramientas de análisis de vulnerabilidades automatizado

**Guía**: No está disponible actualmente; la evaluación de puntos vulnerables en Azure Security Center no está disponible en este momento para Azure API Management.

Plataforma subyacente examinada y revisada por Microsoft. Revise los controles de seguridad disponibles para reducir los puntos vulnerables relacionados con la configuración de los servicios.

* [Controles de seguridad disponibles para Azure API Management](./api-management-security-controls.md)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="52-deploy-automated-operating-system-patch-management-solution"></a>5.2: Implemente una solución de administración de revisiones de sistema operativo automatizada

**Instrucciones**: No aplicable; esta recomendación está pensada para los recursos de proceso.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="53-deploy-automated-third-party-software-patch-management-solution"></a>5.3: Implemente una solución automatizada de administración de revisiones de software de terceros

**Instrucciones**: No aplicable; esta recomendación está pensada para los recursos de proceso.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="54-compare-back-to-back-vulnerability-scans"></a>5.4: Compare los exámenes de vulnerabilidades opuestos

**Instrucciones**: No aplicable; esta recomendación está pensada para los recursos de proceso.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="55-use-a-risk-rating-process-to-prioritize-the-remediation-of-discovered-vulnerabilities"></a>5.5: Use un proceso de clasificación de riesgos para priorizar la corrección de las vulnerabilidades detectadas

**Guía**: No está disponible actualmente; la evaluación de puntos vulnerables en Azure Security Center no está disponible en este momento para Azure API Management.

Plataforma subyacente examinada y revisada por Microsoft. El cliente debe revisar los controles de seguridad disponibles para reducir los puntos vulnerables relacionados con la configuración de los servicios.

* [Controles de seguridad disponibles para Azure API Management](./api-management-security-controls.md)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

## <a name="inventory-and-asset-management"></a>Inventario y administración de recursos

*Para más información, consulte [Control de seguridad: Administración de recursos y del inventario](../security/benchmarks/security-control-inventory-asset-management.md).*

### <a name="61-use-azure-asset-discovery"></a>6.1: Uso de la detección de recursos de Azure

**Guía**: Use Azure Resource Graph para consultar o detectar todos los recursos (por ejemplo, proceso, almacenamiento, red, puertos y protocolos, etc.) dentro de las suscripciones. Asegúrese de que tiene los permisos adecuados (lectura) en el inquilino y enumere todas las suscripciones de Azure, así como los recursos de las suscripciones.

Aunque los recursos clásicos de Azure se pueden detectar a través de Resource Graph, se recomienda encarecidamente crear y usar los recursos de Azure Resource Manager que figuran a continuación.

* [Creación de consultas con Azure Resource Graph](../governance/resource-graph/first-query-portal.md)

* [Visualización de las suscripciones de Azure](/powershell/module/az.accounts/get-azsubscription?view=azps-3.0.0)

* [Descripción de Azure RBAC](../role-based-access-control/overview.md)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="62-maintain-asset-metadata"></a>6.2: Mantenimiento de metadatos de recursos

**Instrucciones**: Aplique etiquetas a los recursos de Azure que proporcionan metadatos para organizarlos de forma lógica en una taxonomía.

* [Creación y uso de etiquetas](../azure-resource-manager/management/tag-resources.md)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="63-delete-unauthorized-azure-resources"></a>6.3: Eliminación de recursos de Azure no autorizados

**Guía**: use el etiquetado, los grupos de administración y las suscripciones independientes, si procede, para organizar y realizar un seguimiento de los recursos de Azure. Concilie el inventario periódicamente y asegúrese de que los recursos no autorizados se eliminan de la suscripción de manera oportuna.

Además, use Azure Policy para establecer restricciones sobre el tipo de recursos que se pueden crear en las suscripciones del cliente con las siguientes definiciones de directiva integradas:
- Tipos de recursos no permitidos
- Tipos de recursos permitidos

* [Creación de suscripciones adicionales de Azure](../cost-management-billing/manage/create-subscription.md)

* [Creación de grupos de administración](../governance/management-groups/create.md)

* [Creación y uso de etiquetas](../azure-resource-manager/management/tag-resources.md)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="64-maintain-an-inventory-of-approved-azure-resources-and-software-titles"></a>6.4: Mantenimiento de un inventario de los recursos de Azure aprobados y títulos de software

**Guía**: No aplicable; esta recomendación está pensada para los recursos de proceso.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="65-monitor-for-unapproved-azure-resources"></a>6.5: Supervisión de recursos de Azure no aprobados

**Guía**: Use Azure Policy para establecer restricciones sobre el tipo de recursos que se pueden crear en las suscripciones con las siguientes definiciones de directiva integradas:
- Tipos de recursos no permitidos
- Tipos de recursos permitidos

Use Azure Resource Graph para consultar o detectar recursos dentro de sus suscripciones. Asegúrese de que todos los recursos de Azure presentes en el entorno estén aprobados.

* [Configuración y administración de Azure Policy](../governance/policy/tutorials/create-and-manage.md)

* [Creación de consultas con Azure Graph](../governance/resource-graph/first-query-portal.md)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="66-monitor-for-unapproved-software-applications-within-compute-resources"></a>6.6: Supervisión de aplicaciones de software no aprobadas en recursos de proceso

**Instrucciones**: No aplicable; esta recomendación está pensada para los recursos de proceso.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="67-remove-unapproved-azure-resources-and-software-applications"></a>6.7: Eliminación de aplicaciones de software y recursos de Azure no aprobadas

**Instrucciones**: No aplicable; esta recomendación está pensada para los recursos de proceso.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="68-use-only-approved-applications"></a>6.8: Uso exclusivo de aplicaciones aprobadas

**Instrucciones**: No aplicable; esta recomendación está pensada para los recursos de proceso.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="69-use-only-approved-azure-services"></a>6.9: Uso exclusivo de servicios de Azure aprobados

**Guía**: use Azure Policy para establecer restricciones sobre el tipo de recursos que se pueden crear en las suscripciones del cliente con las siguientes definiciones de directiva integradas:
- Tipos de recursos no permitidos
- Tipos de recursos permitidos

* [Configuración y administración de Azure Policy](../governance/policy/tutorials/create-and-manage.md)

* [Denegación de un tipo de recurso específico con Azure Policy](/azure/governance/policy/samples/not-allowed-resource-types)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="610-implement-approved-application-list"></a>6.10: Implementación de la lista de aplicaciones aprobadas

**Instrucciones**: No aplicable; esta recomendación está pensada para los recursos de proceso.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="611-limit-users-ability-to-interact-with-azure-resources-manager-via-scripts"></a>6.11: Limitación de la capacidad de los usuarios para interactuar con Azure Resource Manager mediante scripts

**Guía**: Configure el acceso condicional de Azure para limitar la capacidad de los usuarios de interactuar con Azure Resource Manager configurando "Bloquear acceso" en la aplicación Microsoft Azure Management.

* [Configuración del acceso condicional para bloquear el acceso a Azure Resource Manager](../role-based-access-control/conditional-access-azure-management.md)

* [Uso del control de acceso basado en rol en Azure API Management](./api-management-role-based-access-control.md)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="612-limit-users-ability-to-execute-scripts-within-compute-resources"></a>6.12: Limitación de capacidad de los usuarios para ejecutar scripts en recursos de proceso

**Instrucciones**: No aplicable; esta recomendación está pensada para los recursos de proceso.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="613-physically-or-logically-segregate-high-risk-applications"></a>6.13: Segregación física o lógica de aplicaciones de alto riesgo

**Instrucciones**: No aplicable; esta recomendación está pensada para las aplicaciones web que se ejecutan en Azure App Service o en recursos de proceso.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

## <a name="secure-configuration"></a>Configuración segura

*Para más información, consulte [Control de seguridad: Configuración segura](../security/benchmarks/security-control-secure-configuration.md).*

### <a name="71-establish-secure-configurations-for-all-azure-resources"></a>7.1: Establezca configuraciones seguras para todos los recursos de Azure

**Guía**: Defina e implemente configuraciones de seguridad estándar para las instancias de Azure API Management con Azure Policy. Use alias de Azure Policy en el espacio de nombres "Microsoft.ApiManagement" para crear directivas personalizadas con el fin de auditar o aplicar la configuración de los servicios de Azure API Management.

* [Visualización de los alias de Azure Policy disponibles](/powershell/module/az.resources/get-azpolicyalias?view=azps-3.3.0)

* [Configuración y administración de Azure Policy](../governance/policy/tutorials/create-and-manage.md)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="72-establish-secure-operating-system-configurations"></a>7.2: Establezca configuraciones del sistema operativo seguras

**Instrucciones**: No aplicable; esta recomendación está pensada para los recursos de proceso.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="73-maintain-secure-azure-resource-configurations"></a>7.3: Mantenga configuraciones de recursos de Azure seguras

**Guía**: Defina e implemente configuraciones de seguridad estándar para los servicios de Azure API Management con Azure Policy. Use alias de Azure Policy en el espacio de nombres "Microsoft.ApiManagement" para crear directivas personalizadas con el fin de auditar o aplicar la configuración de las instancias de Azure API Management. Use la directiva de Azure [denegar] e [implementar si no existe] para aplicar la configuración segura en los recursos de Azure.

* [Configuración y administración de Azure Policy](../governance/policy/tutorials/create-and-manage.md)

* [Descripción de los efectos de Azure Policy](../governance/policy/concepts/effects.md)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="74-maintain-secure-operating-system-configurations"></a>7.4: Mantenga configuraciones del sistema operativo seguras

**Instrucciones**: No aplicable; esta recomendación está pensada para los recursos de proceso.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Compartido

### <a name="75-securely-store-configuration-of-azure-resources"></a>7.5: Almacene de forma segura la configuración de los recursos de Azure

**Guía**: Si usa definiciones de directivas personalizadas de Azure, use Azure DevOps o Azure Repos para almacenar y administrar de forma segura la configuración de servicios de Azure API Management.

* [Almacenamiento de código en Azure DevOps](/azure/devops/repos/git/gitworkflow?view=azure-devops)

* [Documentación de Azure Repos](/azure/devops/repos/index?view=azure-devops)

* [Descripción del kit de recursos de DevOps de Azure API Management](./api-management-security-controls.md#configuration-management)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="76-securely-store-custom-operating-system-images"></a>7.6: Almacene imágenes de sistema operativo personalizadas de forma segura

**Instrucciones**: No aplicable; esta recomendación está pensada para los recursos de proceso.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="77-deploy-system-configuration-management-tools"></a>7.7: Implemente herramientas de administración de configuración del sistema

**Guía**: Defina e implemente configuraciones de seguridad estándar para los servicios de Azure API Management con Azure Policy. Use alias de Azure Policy en el espacio de nombres "Microsoft.ApiManagement" para crear directivas personalizadas con el fin de auditar o aplicar la configuración de las instancias de Azure API Management. Use la directiva de Azure [denegar] e [implementar si no existe] para aplicar la configuración segura en los recursos de Azure.

* [Configuración y administración de Azure Policy](../governance/policy/tutorials/create-and-manage.md)

* [Descripción de los efectos de Azure Policy](../governance/policy/concepts/effects.md)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="78-deploy-system-configuration-management-tools-for-operating-systems"></a>7.8: Implemente herramientas de administración de configuración del sistema para sistemas operativos

**Instrucciones**: No aplicable; esta recomendación está pensada para los recursos de proceso.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="79-implement-automated-configuration-monitoring-for-azure-services"></a>7.9: Implemente la supervisión de configuración automatizada para servicios de Azure

**Guía**: Use el kit de recursos de DevOps de Azure API Management para realizar la administración de la configuración de Azure API Management.

Además, defina e implemente configuraciones de seguridad estándar para los servicios de Azure API Management con Azure Policy. Use alias de Azure Policy en el espacio de nombres "Microsoft.ApiManagement" para crear directivas personalizadas con el fin de auditar o aplicar la configuración de las instancias de Azure API Management. Use la directiva de Azure [denegar] e [implementar si no existe] para aplicar la configuración segura en los recursos de Azure.

* [Descripción del kit de recursos de DevOps de Azure API Management](./api-management-security-controls.md#configuration-management)

* [Configuración y administración de Azure Policy](../governance/policy/tutorials/create-and-manage.md)

* [Descripción de los efectos de Azure Policy](../governance/policy/concepts/effects.md)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="710-implement-automated-configuration-monitoring-for-operating-systems"></a>7.10: Implemente la supervisión de configuración automatizada para sistemas operativos

**Instrucciones**: No aplicable; esta recomendación está pensada para los recursos de proceso.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="711-manage-azure-secrets-securely"></a>7.11: Administre los secretos de Azure de forma segura

**Guía**: Use Key Vault para administrar certificados y establecer que roten automáticamente. Si usa Azure Key Vault para administrar el certificado SSL de dominio personalizado, asegúrese de que el certificado se inserta en Key Vault como un certificado, no como un secreto.

* [Establecimiento de nombres de dominio personalizados con instrucciones para la rotación de claves de Key Vault](./configure-custom-domain.md)

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Microsoft

### <a name="712-manage-identities-securely-and-automatically"></a>7.12: Administre las identidades de forma segura y automática

**Guía**: Use una identidad de servicio administrada generada por Azure Active Directory (Azure AD) para permitir a la instancia de API Management acceder de forma fácil y segura a otros recursos protegidos de Azure AD, como Azure Key Vault.

* [Creación de una identidad administrada para una instancia de API Management](./api-management-howto-use-managed-service-identity.md)

* [Directiva de autenticación con una identidad administrada](./api-management-authentication-policies.md#ManagedIdentity)

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="713-eliminate-unintended-credential-exposure"></a>7.13: Elimine la exposición de credenciales no intencionada

**Instrucciones**: Implemente el escáner de credenciales para identificar las credenciales en el código. El escáner de credenciales también fomenta el traslado de credenciales detectadas a ubicaciones más seguras, como Azure Key Vault.

* [Configuración del escáner de credenciales](https://secdevtools.azurewebsites.net/helpcredscan.html)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

## <a name="malware-defense"></a>Defensa contra malware

*Para más información, consulte [Control de seguridad: defensa contra malware](../security/benchmarks/security-control-malware-defense.md).*

### <a name="81-use-centrally-managed-anti-malware-software"></a>8.1: Use software antimalware administrado centralmente

**Instrucciones**: No aplicable; esta recomendación está pensada para los recursos de proceso.

Microsoft Antimalware está habilitado en el host subyacente que admite los servicios de Azure (por ejemplo, Azure API Management), pero no se ejecuta en el contenido del cliente.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="82-pre-scan-files-to-be-uploaded-to-non-compute-azure-resources"></a>8.2: Examine previamente los archivos que se van a cargar en recursos de Azure que no son de proceso

**Guía**: No aplicable; esta recomendación está pensada para los recursos que no son de proceso y que están diseñados para almacenar datos.

Microsoft Antimalware está habilitado en el host subyacente que admite los servicios de Azure (por ejemplo, Azure API Management), pero no se ejecuta en el contenido del cliente.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="83-ensure-anti-malware-software-and-signatures-are-updated"></a>8.3: Asegúrese de que se han actualizado el software y las firmas antimalware

**Guía**: No aplicable; esta recomendación está pensada para los recursos que no son de proceso y que están diseñados para almacenar datos.

Microsoft Antimalware está habilitado en el host subyacente que admite los servicios de Azure (por ejemplo, Azure API Management), pero no se ejecuta en el contenido del cliente.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

## <a name="data-recovery"></a>Recuperación de datos

*Para más información, consulte [Control de seguridad: recuperación de datos](../security/benchmarks/security-control-data-recovery.md).*

### <a name="91-ensure-regular-automated-back-ups"></a>9.1: Garantía de copias de seguridad automáticas periódicas

**Instrucciones**: Si publica y administra las API a través de Azure API Management, podrá aprovechar numerosas funcionalidades de infraestructura y tolerancia a errores que, de lo contrario, debería diseñar, implementar y administrar de manera manual. API Management admite la implementación en varias regiones, lo que hace que el plano de datos sea inmune a los errores regionales sin necesidad de agregar ninguna sobrecarga operativa.

Las características de copia de seguridad y restauración de API Management ofrecen los bloques de creación necesarios para implementar una estrategia de recuperación ante desastres. Las operaciones de copia de seguridad y restauración se pueden realizar de forma manual o automatizada.

* [Implementación de un plano de datos de API Management en varias regiones](./api-management-howto-deploy-multi-region.md)

* [Procedimiento para implementar la recuperación ante desastres mediante copias de seguridad y restauración del servicio en Azure API Management](./api-management-howto-disaster-recovery-backup-restore.md#calling-the-backup-and-restore-operations)

* [Llamada a la operación de copia de seguridad de API Management](/rest/api/apimanagement/2019-12-01/apimanagementservice/backup)

* [Llamada a la operación de restauración de API Management](/rest/api/apimanagement/2019-12-01/apimanagementservice/restore)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="92-perform-complete-system-backups-and-backup-any-customer-managed-keys"></a>9.2: Realización de copias de seguridad completas del sistema y copia de seguridad de las claves administradas por el cliente

**Guía**: Las operaciones de copia de seguridad y restauración proporcionadas por Azure API Management realizan copias de seguridad y restauraciones completas del sistema.

Las identidades administradas se pueden utilizar para obtener certificados de Azure Key Vault para los nombres de dominio personalizados de API Management. Realice una copia de seguridad de todos los certificados que se almacenan en Azure Key Vault.

* [Procedimiento para implementar la recuperación ante desastres mediante copias de seguridad y restauración del servicio en Azure API Management](./api-management-howto-disaster-recovery-backup-restore.md#calling-the-backup-and-restore-operations)

* [Realización de copias de seguridad de certificados de Azure Key Vault](/powershell/module/azurerm.keyvault/backup-azurekeyvaultcertificate?view=azurermps-6.13.0)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="93-validate-all-backups-including-customer-managed-keys"></a>9.3: Validación de todas las copias de seguridad, incluidas las claves administradas por el cliente

**Instrucciones**: Compruebe las copias de seguridad; para ello, haga una restauración de prueba del servicio y los certificados a partir de las copias de seguridad.

* [Llamada a la operación de restauración de API Management](/rest/api/apimanagement/2019-12-01/apimanagementservice/restore)

* [Restauración de certificados de Azure Key Vault](/powershell/module/azurerm.keyvault/restore-azurekeyvaultcertificate?view=azurermps-6.13.0)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="94-ensure-protection-of-backups-and-customer-managed-keys"></a>9.4: Garantía de la protección de las copias de seguridad y las claves administradas del cliente

**Guía**: Azure API Management escribe copias de seguridad en las cuentas de Azure Storage propiedad del cliente. Siga las recomendaciones de seguridad de Azure Storage para proteger la copia de seguridad.

* [Procedimiento para implementar la recuperación ante desastres mediante copias de seguridad y restauración del servicio en Azure API Management](./api-management-howto-disaster-recovery-backup-restore.md#calling-the-backup-and-restore-operations)

* [Recomendaciones de seguridad para Blob Storage](../storage/blobs/security-recommendations.md)

Habilite la eliminación temporal en Key Vault para proteger las claves contra la eliminación accidental o malintencionada.

* [Habilitación de la eliminación temporal en Key Vault](../storage/blobs/soft-delete-overview.md?tabs=azure-portal)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

## <a name="incident-response"></a>Respuesta a los incidentes

*Para más información, consulte [Control de seguridad: Respuesta a los incidentes](../security/benchmarks/security-control-incident-response.md).*

### <a name="101-create-an-incident-response-guide"></a>10.1: Creación de una guía de respuesta ante incidentes

**Instrucciones**: Cree una guía de respuesta a incidentes para su organización. Asegúrese de que haya planes de respuesta a incidentes escritos que definan todos los roles del personal, así como las fases de administración y gestión de los incidentes, desde la detección hasta la revisión posterior a la incidencia.

* [Guía para crear su propio proceso de respuesta a incidentes de seguridad](https://msrc-blog.microsoft.com/2019/07/01/inside-the-msrc-building-your-own-security-incident-response-process/)

* [Anatomía de un incidente del Centro de respuestas de seguridad de Microsoft](https://msrc-blog.microsoft.com/2019/06/27/inside-the-msrc-anatomy-of-a-ssirp-incident/)

* [Uso de la guía de control de incidentes de seguridad de equipos de NIST para la creación de su propio plan de respuesta a incidentes](https://csrc.nist.gov/publications/detail/sp/800-61/rev-2/final)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="102-create-an-incident-scoring-and-prioritization-procedure"></a>10.2: Creación de un procedimiento de priorización y puntuación de incidentes

**Instrucciones**: Security Center asigna una gravedad a cada alerta para ayudarle a priorizar aquellas que se deben investigar en primer lugar. La gravedad se basa en la confianza que tiene Security Center en la búsqueda o en el análisis utilizados para emitir la alerta, así como en el nivel de confianza de que ha habido un intento malintencionado detrás de la actividad que ha provocado la alerta.

Además, marque claramente las suscripciones (por ejemplo, producción, no producción) con etiquetas y cree un sistema de nomenclatura para identificar y clasificar claramente los recursos de Azure, especialmente los que procesan datos confidenciales. Es su responsabilidad asignar prioridades a la corrección de las alertas en función de la importancia de los recursos y el entorno de Azure donde se produjo el incidente.

* [Alertas de seguridad en el Centro de seguridad de Azure](../security-center/security-center-alerts-overview.md)

* [Uso de etiquetas para organizar los recursos de Azure](../azure-resource-manager/management/tag-resources.md)

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="103-test-security-response-procedures"></a>10.3: Prueba de los procedimientos de respuesta de seguridad

**Guía**: Realice ejercicios para probar las funcionalidades de respuesta a los incidentes de los sistemas periódicamente para ayudar a proteger los recursos de Azure. Identifique puntos débiles y brechas y revise el plan según sea necesario.

* [Publicación de NIST: Guía para probar, entrenar y ejecutar programas para planes y funcionalidades de TI](https://csrc.nist.gov/publications/detail/sp/800-84/final)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="104-provide-security-incident-contact-details-and-configure-alert-notifications-for-security-incidents"></a>10.4: Provisión de detalles de contacto de incidentes de seguridad y configuración de notificaciones de alerta para incidentes de seguridad

**Instrucciones**: La información de contacto del incidente de seguridad la utilizará Microsoft para ponerse en contacto con usted si Microsoft Security Response Center (MSRC) detecta que un tercero no autorizado o ilegal ha accedido a los datos. Revise los incidentes después del hecho para asegurarse de que se resuelven los problemas.

* [Establecimiento del contacto de seguridad de Azure Security Center](../security-center/security-center-provide-security-contact-details.md)

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="105-incorporate-security-alerts-into-your-incident-response-system"></a>10.5: Incorporación de alertas de seguridad en el sistema de respuesta a incidentes

**Guía**: Exporte las alertas y recomendaciones de Azure Security Center y mediante la característica de exportación continua para ayudar a identificar los riesgos para los recursos de Azure. La exportación continua le permite exportar alertas y recomendaciones de forma manual o continua. Puede usar el conector de datos de Azure Security Center para transmitir las alertas a Azure Sentinel.

* [Configuración de la exportación continua](../security-center/continuous-export.md)

* [Transmisión de alertas a Azure Sentinel](../sentinel/connect-azure-security-center.md)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="106-automate-the-response-to-security-alerts"></a>10.6: Automatización de la respuesta a las alertas de seguridad

**Guía**: Use la característica Automatización de flujo de trabajo de Azure Security Center para desencadenar automáticamente respuestas mediante "Logic Apps" en las alertas y recomendaciones de seguridad.

* [Configuración de la automatización de flujo de trabajo y Logic Apps](../security-center/workflow-automation.md)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

## <a name="penetration-tests-and-red-team-exercises"></a>Pruebas de penetración y ejercicios del equipo rojo

*Para más información, consulte [Control de seguridad: Pruebas de penetración y ejercicios del equipo rojo](../security/benchmarks/security-control-penetration-tests-red-team-exercises.md).*

### <a name="111-conduct-regular-penetration-testing-of-your-azure-resources-and-ensure-to-remediate-all-critical-security-findings-within-60-days"></a>11.1: Realización de pruebas de penetración periódicas de los recursos de Azure y garantía de la corrección de todos los resultados de seguridad críticos en un plazo de 60 días

**Guía**: * [Siga las reglas de compromiso de Microsoft para asegurarse de que las pruebas de penetración no infrinjan las directivas de Microsoft](https://www.microsoft.com/msrc/pentest-rules-of-engagement?rtc=1.).

* [Aquí encontrará más información sobre la estrategia y puesta en marcha de un equipo rojo de Microsoft, y las pruebas de penetración en sitios activos de la infraestructura en la nube, las aplicaciones y los servicios administrados por Microsoft](https://gallery.technet.microsoft.com/Cloud-Red-Teaming-b837392e).

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Compartido

## <a name="next-steps"></a>Pasos siguientes

- Consulte la [prueba comparativa de seguridad de Azure](../security/benchmarks/overview.md).
- Obtenga más información sobre las [líneas de base de seguridad de Azure](../security/benchmarks/security-baselines-overview.md).
