---
title: Base de referencia de seguridad de Azure para Azure Functions
description: Base de referencia de seguridad de Azure para Azure Functions
author: msmbaldwin
ms.service: security
ms.topic: conceptual
ms.date: 05/04/2020
ms.author: mbaldwin
ms.custom: security-benchmark
ms.openlocfilehash: 32905383b046148893492640fe42fc721d1944dd
ms.sourcegitcommit: 85eb6e79599a78573db2082fe6f3beee497ad316
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/05/2020
ms.locfileid: "87810248"
---
# <a name="azure-security-baseline-for-azure-functions"></a>Base de referencia de seguridad de Azure para Azure Functions

La base de referencia de seguridad de Azure para Azure Functions contiene recomendaciones que le ayudarán a mejorar la estrategia de seguridad de la implementación.

La base de referencia de este servicio se extrae de la [versión 1.0 de Azure Security Benchmark](../security/benchmarks/overview.md), que proporciona recomendaciones sobre cómo puede proteger las soluciones en la nube en Azure con nuestra guía de procedimientos recomendados.

Para más información, consulte [Introducción a la línea de base de seguridad de Azure](../security/benchmarks/security-baselines-overview.md).

## <a name="network-security"></a>Seguridad de las redes

*Para más información, consulte [Control de seguridad: Seguridad de redes](../security/benchmarks/security-control-network-security.md).*

### <a name="11-protect-resources-using-network-security-groups-or-azure-firewall-on-your-virtual-network"></a>1.1: Proteja los recursos mediante grupos de seguridad de red o Azure Firewall en su red virtual

**Guía**: integre las aplicaciones de Azure Functions con una red virtual de Azure. Las aplicaciones de Functions que se ejecutan en el plan Premium tienen las mismas funcionalidades de hospedaje que las aplicaciones web de Azure App Service, entre las que se incluye la característica "Integración con red virtual".  Las redes virtuales de Azure le permiten colocar cualquier recurso de Azure, como Azure Functions, en una red enrutable que no sea Internet.

- [Integración de Functions con una red virtual de Azure](./functions-create-vnet.md)

- [Descripción de la característica Integración con red virtual de Azure Functions y Azure App Service](../app-service/web-sites-integrate-with-vnet.md)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="12-monitor-and-log-the-configuration-and-traffic-of-vnets-subnets-and-nics"></a>1.2: Supervise y registre la configuración y el tráfico de redes virtuales, subredes y NIC

**Guía**: utilice Azure Security Center y siga las recomendaciones de protección de redes para ayudar a salvaguardar las configuraciones y recursos de red relacionados con las aplicaciones de Azure Functions.

Si usa grupos de seguridad de red (NSG) con su implementación de Azure Functions, habilite los registros de flujo de estos grupos y envíe registros a una cuenta de Azure Storage para realizar auditorías del tráfico. También puede enviar registros de flujo de grupo de seguridad de red a un área de trabajo de Log Analytics y usar el Análisis de tráfico para proporcionar información detallada sobre el flujo de tráfico en la nube de Azure. Algunas de las ventajas del Análisis de tráfico son la capacidad de visualizar la actividad de la red e identificar las zonas activas, identificar las amenazas de seguridad, comprender los patrones de flujo de tráfico y detectar configuraciones de red incorrectas.

- [Descripción de la seguridad de red proporcionada por Azure Security Center](../security-center/security-center-network-recommendations.md)

- [Habilitación de los registros de flujo de NSG](../network-watcher/network-watcher-nsg-flow-logging-portal.md)

- [Habilitación y uso del Análisis de tráfico](../network-watcher/traffic-analytics.md)

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="13-protect-critical-web-applications"></a>1.3: Proteja las aplicaciones web críticas

**Guía**: Para proteger totalmente los puntos de conexión de Azure Functions en producción, considere la posibilidad de implementar una de las siguientes opciones de seguridad en el nivel de aplicación de Functions:
- active la autenticación o autorización de App Service en la aplicación de Functions,
- utilice Azure API Management (APIM) para autenticar las solicitudes o
- implemente la aplicación de Functions en una instancia de Azure App Service Environment.

Además, debe asegurarse de que la depuración remota esté deshabilitada en la instancia de producción de Azure Functions. Asimismo, el uso compartido de recursos entre orígenes (CORS) no debe permitir que todos los dominios accedan a la aplicación de Azure Functions. Solo los dominios necesarios deben tener permiso para interactuar con la aplicación de Azure Functions.

Considere la posibilidad de implementar el firewall de aplicaciones web (WAF) de Azure durante la configuración de las redes para realizar una inspección adicional del tráfico entrante. Habilite la configuración de diagnóstico para WAF e ingiera los registros en una cuenta de almacenamiento, un centro de eventos o un área de trabajo de análisis de registros. 

- [Protección de los puntos de conexión de Azure Functions en producción](./functions-bindings-http-webhook-trigger.md?tabs=csharp#secure-an-http-endpoint-in-production)

- [Implementación de WAF de Azure](../web-application-firewall/ag/create-waf-policy-ag.md)

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="14-deny-communications-with-known-malicious-ip-addresses"></a>1.4: Deniegue las comunicaciones con direcciones IP malintencionadas conocidas

**Guía**: habilite el estándar DDoS Protection en las redes virtuales asociadas a las aplicaciones de Functions para protegerse de los ataques de denegación de servicio distribuido (DDoS). Use la inteligencia sobre amenazas integrada en Azure Security Center para denegar las comunicaciones con direcciones IP públicas malintencionadas conocidas o no utilizadas.
Asimismo, configure una puerta de enlace front-end, como el firewall de aplicaciones web de Azure, para autenticar todas las solicitudes entrantes y filtrar el tráfico malintencionado. El firewall de aplicaciones web de Azure puede ayudarle a proteger las aplicaciones de Azure Functions inspeccionando el tráfico web entrante para bloquear las inyecciones de código SQL, los scripts de sitios, las cargas de malware y los ataques de denegación de servicio distribuido. Para incorporar un WAF, se necesita una instancia de App Service Environment o deben usarse puntos de conexión privados (versión preliminar). Asegúrese de que los puntos de conexión privados no tengan ya el estado de versión preliminar antes de usarlos con las cargas de trabajo de producción.

- [Opciones de redes de Azure Functions](./functions-networking-options.md)

- [Plan Premium de Azure Functions](./functions-scale.md#premium-plan)

- [Introducción a App Service Environment](../app-service/environment/intro.md)

- [Consideraciones de red para un entorno de App Service Environment](../app-service/environment/network-info.md)

- [Configuración de la protección contra DDoS](../virtual-network/manage-ddos-protection.md)

- [Implementación de Azure Firewall](../firewall/tutorial-firewall-deploy-portal.md)

- [Descripción de la inteligencia sobre amenazas integrada de Azure Security Center](../security-center/threat-protection.md)

- [Descripción de la protección de red adaptable de Azure Security Center](../security-center/security-center-adaptive-network-hardening.md)

- [Descripción del control de acceso de red Just-in-Time de Azure Security Center](../security-center/security-center-just-in-time.md)

- [Uso de puntos de conexión privados con Azure Functions](../app-service/networking/private-endpoint.md)

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="15-record-network-packets-and-flow-logs"></a>1.5: Registre los paquetes de red y registros de flujo

**Guía**: Si usa grupos de seguridad de red (NSG) con su implementación de Azure Functions, habilite los registros de flujo en estos grupos y envíe dichos registros a una cuenta de almacenamiento para auditar el tráfico. También puede enviar los registros de flujo a un área de trabajo de Log Analytics y usar Análisis de tráfico para obtener conclusiones sobre el flujo del tráfico en la nube de Azure. Algunas de las ventajas del Análisis de tráfico son la capacidad de visualizar la actividad de la red e identificar las zonas activas, identificar las amenazas de seguridad, comprender los patrones de flujo de tráfico y detectar configuraciones de red incorrectas.

- [Habilitación de los registros de flujo de NSG](../network-watcher/network-watcher-nsg-flow-logging-portal.md)

- [Habilitación y uso del Análisis de tráfico](../network-watcher/traffic-analytics.md)

- [Habilitación de Network Watcher](../network-watcher/network-watcher-create.md)

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="16-deploy-network-based-intrusion-detectionintrusion-prevention-systems-idsips"></a>1.6: Implemente sistemas de prevención de intrusiones y detección de intrusiones (IDS/IPS) basados en la red

**Guía**: Configure una puerta de enlace front-end como firewall de aplicaciones web de Azure para autenticar todas las solicitudes entrantes y filtrar el tráfico malintencionado. Los firewalls de aplicaciones web de Azure ayudan a proteger las aplicaciones de Functions, ya que inspeccionan el tráfico web entrante para bloquear las inyecciones de código SQL, los scripts de sitios, las cargas de malware y los ataques de denegación de servicio distribuido. Para incorporar un WAF, se necesita una instancia de App Service Environment o deben usarse puntos de conexión privados (versión preliminar). Asegúrese de que los puntos de conexión privados no tengan ya el estado de versión preliminar antes de usarlos con las cargas de trabajo de producción.

Existen también otras opciones en el mercado, como el WAF de Barracuda para Azure, que están disponibles en Azure Marketplace con las características IDS/IPS.

- [Opciones de redes de Azure Functions](./functions-networking-options.md)

- [Plan Premium de Azure Functions](./functions-scale.md#premium-plan)

- [Introducción a App Service Environment](../app-service/environment/intro.md)

- [Consideraciones de red para un entorno de App Service Environment](../app-service/environment/network-info.md)

- [Descripción del firewall de aplicaciones web de Azure](../web-application-firewall/index.yml)

- [Uso de puntos de conexión privados con Azure Functions](../app-service/networking/private-endpoint.md)

- [Descripción del servicio en la nube Barracuda WAF](../app-service/environment/app-service-app-service-environment-web-application-firewall.md#configuring-your-barracuda-waf-cloud-service)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="17-manage-traffic-to-web-applications"></a>1.7: Administre el tráfico a las aplicaciones web

**Instrucciones**: Configure una puerta de enlace front-end para la red, como el firewall de aplicaciones web de Azure con cifrado TLS de un extremo a otro. Para incorporar un WAF, se necesita una instancia de App Service Environment o deben usarse puntos de conexión privados (versión preliminar). Asegúrese de que los puntos de conexión privados no tengan ya el estado de versión preliminar antes de usarlos con las cargas de trabajo de producción.

- [Opciones de redes de Azure Functions](./functions-networking-options.md)

- [Plan Premium de Azure Functions](./functions-scale.md#premium-plan)

- [Introducción a App Service Environment](../app-service/environment/intro.md)

- [Consideraciones de red para un entorno de App Service Environment](../app-service/environment/network-info.md)

- [Descripción del firewall de aplicaciones web de Azure](../web-application-firewall/index.yml)

- [Configuración de TLS de un extremo a otro con Application Gateway mediante el portal](../application-gateway/end-to-end-ssl-portal.md)

- [Uso de puntos de conexión privados con Azure Functions](../app-service/networking/private-endpoint.md)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="18-minimize-complexity-and-administrative-overhead-of-network-security-rules"></a>1.8: Minimice la complejidad y la sobrecarga administrativa de las reglas de seguridad de red

**Instrucciones**: puede usar las etiquetas del servicio Red virtual para definir los controles de acceso a la red en los grupos de seguridad de red o Azure Firewall. Puede utilizar etiquetas de servicio en lugar de direcciones IP específicas al crear reglas de seguridad. Al especificar el nombre de la etiqueta de servicio (por ejemplo, AzureAppService) en el campo de origen o destino apropiado de una regla, puede permitir o denegar el tráfico en el servicio correspondiente. Microsoft administra los prefijos de direcciones que la etiqueta de servicio incluye y actualiza automáticamente dicha etiqueta a medida que las direcciones cambian.

- [Más información sobre el uso de etiquetas de servicio](../virtual-network/service-tags-overview.md)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="19-maintain-standard-security-configurations-for-network-devices"></a>1.9: Mantenga las configuraciones de seguridad estándar para dispositivos de red

**Guía**: defina e implemente configuraciones de seguridad estándar para los recursos de red relacionados con Azure Functions. Utilice alias de Azure Policy en los espacios de nombres "Microsoft.Web" y "Microsoft.Network" para crear directivas personalizadas que permitan auditar o aplicar la configuración de red de Azure Functions. También puede usar definiciones de directiva integradas en Azure Functions, como:
- CORS no debe permitir que todos los recursos accedan a Function App
- Acceso a Function App solo a través de HTTPS
- La versión más reciente de TLS debe usarse en la aplicación de funciones.

También puede utilizar Azure Blueprints para simplificar las implementaciones de Azure a gran escala mediante el empaquetado de artefactos de entorno clave, como plantillas de Azure Resource Manager (ARM), control de acceso basado en rol (RBAC) y directivas, en una única definición de plano técnico. Aplique fácilmente el plano técnico a nuevas suscripciones y entornos, y ajuste el control y la administración mediante el control de versiones.

- [Configuración y administración de Azure Policy](../governance/policy/tutorials/create-and-manage.md)

- [Creación de un plano técnico de Azure](../governance/blueprints/create-blueprint-portal.md)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="110-document-traffic-configuration-rules"></a>1.10: Documente las reglas de configuración de tráfico

**Instrucciones**: Si usa grupos de seguridad de red (NSG) con su implementación de Azure Functions, use etiquetas para estos grupos y otros recursos relacionados con la seguridad de red y el flujo del tráfico. En el caso de las reglas de NSG individuales, use el campo "Descripción" para especificar las necesidades empresariales o la duración (etc.) de las reglas que permiten que entre o salga el tráfico en una red.

Use cualquiera de las definiciones de directiva de Azure integradas relacionadas con el etiquetado, como "Requerir etiqueta y su valor" para asegurarse de que todos los recursos se crean con etiquetas y para notificarle los recursos no etiquetados existentes.

Puede usar Azure PowerShell o la CLI de Azure para buscar o realizar acciones en los recursos en función de sus etiquetas.

- [Creación y uso de etiquetas](../azure-resource-manager/management/tag-resources.md)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="111-use-automated-tools-to-monitor-network-resource-configurations-and-detect-changes"></a>1.11: Use herramientas automatizadas para supervisar las configuraciones de recursos de red y detectar cambios

**Guía**: use el registro de actividad de Azure para supervisar las configuraciones de los recursos de red y detectar cambios en las opciones y recursos de red relacionados con las implementaciones de Azure Functions. Cree alertas en Azure Monitor que se desencadenen cuando se produzcan cambios en la configuraciones o recursos de red críticos. 

- [Visualización y recuperación de eventos del registro de actividad de Azure](../azure-monitor/platform/activity-log.md#view-the-activity-log)

- [Creación de alertas en Azure Monitor](../azure-monitor/platform/alerts-activity-log.md)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

## <a name="logging-and-monitoring"></a>Registro y supervisión

*Para más información, consulte [Control de seguridad: registro y supervisión](../security/benchmarks/security-control-logging-monitoring.md).*

### <a name="21-use-approved-time-synchronization-sources"></a>2.1: Uso de orígenes de sincronización de hora aprobados

**Guía**: Microsoft utiliza el mismo origen que los recursos de Azure, como Azure Functions, para establecer las marcas de tiempo en los registros.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Microsoft

### <a name="22-configure-central-security-log-management"></a>2.2: Configuración de la administración central de registros de seguridad

**Guía**: Para el registro de la auditoria del plano de control, habilite la configuración de diagnóstico del registro de actividad de Azure y envíela a un área de trabajo de Log Analytics, un centro de eventos de Azure o una cuenta de almacenamiento de Azure para su archivo. Con los datos del registro de actividad de Azure, puede responder a las preguntas "qué, quién y cuándo" de las operaciones de escritura (PUT, POST, DELETE) llevadas a cabo en el nivel del plano de control para los recursos de Azure.

Azure Functions también puede integrarse directamente con Azure Application Insights para supervisar las funciones. Application Insights recopila información del registro, el rendimiento y los errores. Detecta automáticamente anomalías en el rendimiento e incluye herramientas de análisis eficaces que ayudan a diagnosticar problemas y comprender cómo se usan las funciones.

Si tiene un registro de auditoría o seguridad personalizado integrado en la aplicación de Azure Functions, habilite la opción de diagnóstico "FunctionAppLogs" y envíe los registros a un área de trabajo de Log Analytics, a un centro de eventos de Azure o a una cuenta de almacenamiento de Azure para archivarlos. 

Si lo desea, también puede habilitar e incorporar datos en Azure Sentinel o en una herramienta de Administración de eventos e información de seguridad (SIEM) de terceros. 

- [Habilitación de la configuración de diagnóstico para el registro de actividad de Azure](../azure-monitor/platform/activity-log.md)

- [Configuración de Azure Functions con Azure Application Insights](./functions-monitoring.md)

- [Activación de la configuración de diagnóstico (registros generados por el usuario) en Azure Functions](./functions-monitor-log-analytics.md)

- [Incorporación de Azure Sentinel](../sentinel/quickstart-onboard.md)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="23-enable-audit-logging-for-azure-resources"></a>2.3: Habilitación del registro de auditoría para recursos de Azure

**Guía**: Para el registro de la auditoria del plano de control, habilite la configuración de diagnóstico del registro de actividad de Azure y envíela a un área de trabajo de Log Analytics, un centro de eventos de Azure o una cuenta de almacenamiento de Azure para su archivo. Con los datos del registro de actividad de Azure, puede responder a las preguntas "qué, quién y cuándo" de las operaciones de escritura (PUT, POST, DELETE) llevadas a cabo en el nivel del plano de control para los recursos de Azure.

Si tiene un registro de auditoría o seguridad personalizado integrado en la aplicación de Azure Functions, habilite la opción de diagnóstico "FunctionAppLogs" y envíe los registros a un área de trabajo de Log Analytics, a un centro de eventos de Azure o a una cuenta de almacenamiento de Azure para archivarlos. 

- [Habilitación de la configuración de diagnóstico para el registro de actividad de Azure](../azure-monitor/platform/activity-log.md)

- [Activación de la configuración de diagnóstico (registros generados por el usuario) en Azure Functions](./functions-monitor-log-analytics.md)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="24-collect-security-logs-from-operating-systems"></a>2.4: Recopilación de registros de seguridad de sistemas operativos

**Guía**: no aplicable; esta guía está pensada para recursos de proceso de IaaS.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="25-configure-security-log-storage-retention"></a>2.5: Configuración de la retención del almacenamiento de registros de seguridad

**Guía**: En Azure Monitor, establezca el período de retención de registros en las áreas de trabajo de Log Analytics vinculadas a las aplicaciones de Azure Functions atendiendo a las normativas de cumplimiento de su organización.

- [Establecimiento de parámetros de retención de registros](../azure-monitor/platform/manage-cost-storage.md#change-the-data-retention-period)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="26-monitor-and-review-logs"></a>2.6: Supervisión y registros de revisión

**Guía**: Habilite la configuración de diagnóstico del registro de actividad de Azure, así como la configuración de diagnóstico de la aplicación Azure Functions, y envíe los registros a un área de trabajo de Log Analytics. Realice consultas en Log Analytics para buscar términos, identificar tendencias, analizar patrones y proporcionar otra información detallada basada en los datos recopilados.

Habilite Application Insights para que las aplicaciones de Azure Functions puedan recopilar datos de registro, rendimiento y error. Puede ver los datos de telemetría recopilados por Application Insights en Azure Portal.

Si tiene un registro de auditoría o seguridad personalizado integrado en la aplicación de Azure Functions, habilite la opción de diagnóstico "FunctionAppLogs" y envíe los registros a un área de trabajo de Log Analytics, a un centro de eventos de Azure o a una cuenta de almacenamiento de Azure para archivarlos. 

Si lo desea, también puede habilitar e incorporar datos en Azure Sentinel o en una herramienta de Administración de eventos e información de seguridad (SIEM) de terceros. 

- [Habilitación de la configuración de diagnóstico para el registro de actividad de Azure](../azure-monitor/platform/activity-log.md)

- [Activación de la configuración de diagnóstico en Azure Functions](./functions-monitor-log-analytics.md)

- [Configuración de Azure Functions con Azure Application Insights y visualización de los datos de telemetría](./functions-monitoring.md)

- [Incorporación de Azure Sentinel](../sentinel/quickstart-onboard.md)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="27-enable-alerts-for-anomalous-activity"></a>2.7: Habilitación de alertas para actividades anómalas

**Guía**: Habilite la configuración de diagnóstico del registro de actividad de Azure, así como la configuración de diagnóstico de la aplicación Azure Functions, y envíe los registros a un área de trabajo de Log Analytics. Realice consultas en Log Analytics para buscar términos, identificar tendencias, analizar patrones y proporcionar otra información detallada basada en los datos recopilados. Puede crear alertas basadas en las consultas del área de trabajo de Log Analytics.

Habilite Application Insights para que las aplicaciones de Azure Functions puedan recopilar datos de registro, rendimiento y error. Puede ver los datos de telemetría recopilados por Application Insights y crear alertas en Azure Portal.

Si lo desea, también puede habilitar e incorporar datos en Azure Sentinel o en una herramienta de Administración de eventos e información de seguridad (SIEM) de terceros. 

- [Habilitación de la configuración de diagnóstico para el registro de actividad de Azure](../azure-monitor/platform/activity-log.md)

- [Activación de la configuración de diagnóstico en Azure Functions](./functions-monitor-log-analytics.md)

- [Activación de Application Insights para Azure Functions](./functions-monitoring.md#enable-application-insights-integration)

- [Creación de alertas en Azure](../azure-monitor/learn/tutorial-response.md)

- [Incorporación de Azure Sentinel](../sentinel/quickstart-onboard.md)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="28-centralize-anti-malware-logging"></a>2.8: Centralización del registro antimalware

**Guía**: no aplicable; las aplicaciones de Azure Functions no procesan ni generan registros relacionados con malware.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="29-enable-dns-query-logging"></a>2.9: Habilitación del registro de consultas DNS

**Guía**: no aplicable; las aplicaciones de Azure Functions no procesan ni generan registros relacionados con DNS accesibles por el usuario.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="210-enable-command-line-audit-logging"></a>2.10: Habilitación del registro de auditoría de la línea de comandos

**Guía**: no aplicable; esta guía está pensada para recursos de proceso de IaaS.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

## <a name="identity-and-access-control"></a>Control de identidades y acceso

*Para más información, consulte [Control de seguridad: Identidad y control de acceso](../security/benchmarks/security-control-identity-access-control.md).*

### <a name="31-maintain-an-inventory-of-administrative-accounts"></a>3.1: Mantenga un inventario de cuentas administrativas

**Instrucciones**: Azure Active Directory (AD) tiene roles integrados que se deben asignar explícitamente y son consultables. Use el módulo de PowerShell de Azure AD para realizar consultas ad hoc para detectar cuentas que son miembros de grupos administrativos. 

- [Obtención de un rol de directorio en Azure AD con PowerShell](/powershell/module/azuread/get-azureaddirectoryrole?view=azureadps-2.0)

- [Obtención de los miembros de un rol de directorio en Azure AD con PowerShell](/powershell/module/azuread/get-azureaddirectoryrolemember?view=azureadps-2.0)

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="32-change-default-passwords-where-applicable"></a>3.2: Cambie las contraseñas predeterminadas cuando proceda

**Guía**: El acceso del plano de control a Azure Functions se controla mediante Azure Active Directory (AD). Azure AD no tiene el concepto de contraseñas predeterminadas.

El acceso al plano de datos se puede controlar a través de diversos medios, como las claves de autorización, las restricciones de red y la validación de una identidad de AAD. Las claves de autorización las utilizan los clientes que se conectan a los puntos de conexión HTTP de Azure Functions y se pueden volver a generar en cualquier momento. Estas claves se generan de forma predeterminada para los nuevos puntos de conexión HTTP.

Hay diversos métodos de implementación disponibles para las aplicaciones de Functions, algunos de los cuales pueden hacer uso de un conjunto de credenciales generadas. Revise los métodos de implementación que se usarán con la aplicación.

- [Protección de un punto de conexión HTTP](./functions-bindings-http-webhook-trigger.md?tabs=csharp#secure-an-http-endpoint-in-production)

- [Obtención y regeneración de claves de autorización](./functions-bindings-http-webhook-trigger.md?tabs=csharp#obtaining-keys)

- [Tecnologías de implementación en Azure Functions](./functions-deployment-technologies.md)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="33-use-dedicated-administrative-accounts"></a>3.3: Use cuentas administrativas dedicadas

**Guía**: Cree procedimientos operativos estándar en torno al uso de cuentas administrativas dedicadas. Use la administración de identidad y acceso de Azure Security Center para supervisar el número de cuentas administrativas.

Además, para ayudarle a realizar un seguimiento de las cuentas administrativas dedicadas, puede seguir las recomendaciones de Azure Security Center o las directivas integradas de Azure Policy, por ejemplo: Debería haber más de un propietario asignado a la suscripción. Las cuentas en desuso con permisos de propietario deben quitarse de la suscripción. Las cuentas externas con permisos de propietario deben quitarse de la suscripción.

- [Uso de Azure Security Center para supervisar la identidad y el acceso (versión preliminar)](../security-center/security-center-identity-access.md)

- [Uso de Azure Policy](../governance/policy/tutorials/create-and-manage.md)

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="34-use-single-sign-on-sso-with-azure-active-directory"></a>3.4: Use el inicio de sesión único (SSO) con Azure Active Directory

**Guía**: siempre que sea posible, use el SSO de Azure Active Directory en lugar de configurar diferentes credenciales individuales en la aplicación de Functions para acceder a los datos. Use las recomendaciones de administración de identidades y acceso de Azure Security Center. Implemente el inicio de sesión único en las aplicaciones de Azure Functions utilizando la característica de autenticación y autorización de App Service.

- [Descripción de la autenticación y la autorización en Azure Functions](../app-service/overview-authentication-authorization.md#identity-providers)

- [Descripción del SSO con Azure AD](../active-directory/manage-apps/what-is-single-sign-on.md)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="35-use-multi-factor-authentication-for-all-azure-active-directory-based-access"></a>3.5: Use la autenticación multifactor para todo el acceso basado en Azure Active Directory

**Guía**: Habilite Multi-Factor Authentication (MFA) de Azure Active Directory (AD) y siga las recomendaciones de administración de identidades y acceso de Azure Security Center.

- [Procedimiento para habilitar la MFA en Azure](../active-directory/authentication/howto-mfa-getstarted.md)

- [Supervisión de la identidad y el acceso en Azure Security Center](../security-center/security-center-identity-access.md)

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="36-use-dedicated-machines-privileged-access-workstations-for-all-administrative-tasks"></a>3.6: Use máquinas dedicadas (estaciones de trabajo de acceso con privilegios) para todas las tareas administrativas

**Instrucciones**: Utilice estaciones de trabajo de acceso con privilegios (PAW), con Multi-Factor Authentication (MFA) configurado para iniciar sesión en recursos de Azure y configurarlos.

- [Más información sobre las estaciones de trabajo con privilegios de acceso](/windows-server/identity/securing-privileged-access/privileged-access-workstations)

- [Procedimiento para habilitar la MFA en Azure](../active-directory/authentication/howto-mfa-getstarted.md)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="37-log-and-alert-on-suspicious-activity-from-administrative-accounts"></a>3.7: Registre y alerte de la actividad sospechosa desde cuentas administrativas

**Guía**: Use Privileged Identity Management (PIM) de Azure Active Directory (AD) para la generación de registros y alertas cuando se produzca una actividad sospechosa o insegura en el entorno.

Además, use las detecciones de riesgo de Azure AD para ver alertas e informes sobre el comportamiento de los usuarios de riesgo.

- [Cómo implementar Privileged Identity Management (PIM)](../active-directory/privileged-identity-management/pim-deployment-plan.md)

- [Información sobre las detecciones de riesgo de Azure AD](../active-directory/identity-protection/overview-identity-protection.md)

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="38-manage-azure-resources-from-only-approved-locations"></a>3.8: Administre los recursos de Azure solo desde ubicaciones aprobadas

**Guía**: El cliente debe usar ubicaciones con nombre de acceso condicional para permitir el acceso a Azure Portal solo desde agrupaciones lógicas específicas de intervalos de direcciones IP o países o regiones.

- [Configuración de ubicaciones con nombre en Azure](../active-directory/reports-monitoring/quickstart-configure-named-locations.md)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="39-use-azure-active-directory"></a>3.9: Uso de Azure Active Directory

**Guía**: Use Azure Active Directory (AD) como sistema central de autenticación y autorización en las aplicaciones de Azure Functions. Azure AD protege los datos mediante un cifrado seguro para los datos en reposo y en tránsito. Azure AD también cifra con sal, convierte en hash y almacena de forma segura las credenciales de los usuarios.

- [Configuración de una aplicación de App Service o Azure Functions para usar el inicio de sesión de Azure AD](../app-service/configure-authentication-provider-aad.md)

- [Creación y configuración de una instancia de AAD](../active-directory/fundamentals/active-directory-access-create-new-tenant.md)

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="310-regularly-review-and-reconcile-user-access"></a>3.10: Revise y concilie regularmente el acceso de los usuarios

**Guía**: Azure Active Directory (AD) proporciona registros para ayudar a descubrir cuentas obsoletas. Además, use las revisiones de acceso de identidad de Azure para administrar de forma eficiente las pertenencias a grupos, el acceso a las aplicaciones empresariales y las asignaciones de roles. El acceso de los usuarios se puede revisar de forma periódica para asegurarse de que solo las personas adecuadas tengan acceso continuado. 

- [Descripción de los informes de Azure AD](../active-directory/reports-monitoring/index.yml)

- [Procedimiento para usar las revisiones de acceso de identidad de Azure](../active-directory/governance/access-reviews-overview.md)

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="311-monitor-attempts-to-access-deactivated-accounts"></a>3.11: Supervise los intentos de acceso a cuentas desactivadas

**Guía**: use Azure Active Directory (AD) como sistema central de autenticación y autorización de las aplicaciones de Azure Functions. Azure AD protege los datos mediante un cifrado seguro para los datos en reposo y en tránsito. Azure AD también cifra con sal, convierte en hash y almacena de forma segura las credenciales de los usuarios.

Tiene acceso a los orígenes de registro de eventos de actividad de inicio de sesión, auditoría y riesgo de Azure AD, que permiten la integración en Azure Sentinel o SIEM de terceros.

Para simplificar este proceso, cree una configuración de diagnóstico para las cuentas de usuario de Azure AD y envíe los registros de auditoría y los registros de inicio de sesión a un área de trabajo de Log Analytics. Puede configurar las alertas de registro deseadas en Log Analytics.

- [Configuración de una aplicación de App Service o Azure Functions para usar el inicio de sesión de Azure AD](../app-service/configure-authentication-provider-aad.md)

- [Integración de los registros de actividad de Azure en Azure Monitor](../active-directory/reports-monitoring/howto-integrate-activity-logs-with-log-analytics.md)

- [Incorporación de Azure Sentinel](../sentinel/quickstart-onboard.md)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="312-alert-on-account-login-behavior-deviation"></a>3.12: Alerte de las desviaciones de comportamiento en los inicios de sesión de las cuentas

**Guía**: Use Azure Active Directory (AD) como sistema central de autenticación y autorización en las aplicaciones de Azure Functions. En caso de que se produzcan desviaciones en el comportamiento de inicio de sesión de la cuenta del plano de control (Azure Portal), use Azure Active Directory (AD) Identity Protection y las características de detección de riesgo para configurar respuestas automatizadas a las acciones sospechosas que se detecten en relación con las identidades de los usuarios. También puede hacer que Azure Sentinel ingiera los datos para investigarlos más.

- [Visualización de los inicios de sesión de riesgo de Azure AD](../active-directory/identity-protection/overview-identity-protection.md)

- [Configuración y habilitación de las directivas de riesgo de protección de identidad](../active-directory/identity-protection/howto-identity-protection-configure-risk-policies.md)

- [Incorporación de Azure Sentinel](../sentinel/quickstart-onboard.md)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="313-provide-microsoft-with-access-to-relevant-customer-data-during-support-scenarios"></a>3.13: Proporcione a Microsoft acceso a los datos pertinentes del cliente durante los escenarios de soporte técnico

**Guía**: no está disponible en este momento; Caja de seguridad del cliente no es compatible actualmente con Azure Functions.

- [Lista de servicios admitidos por Caja de seguridad del cliente](../security/fundamentals/customer-lockbox-overview.md#supported-services-and-scenarios-in-general-availability)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

## <a name="data-protection"></a>Protección de los datos

*Para más información, consulte [Control de seguridad: Protección de datos](../security/benchmarks/security-control-data-protection.md).*

### <a name="41-maintain-an-inventory-of-sensitive-information"></a>4.1: Mantenimiento de un inventario de información confidencial

**Instrucciones**: use etiquetas para ayudar a realizar el seguimiento de los recursos de Azure que almacenan o procesan información confidencial.

- [Creación y uso de etiquetas](../azure-resource-manager/management/tag-resources.md)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="42-isolate-systems-storing-or-processing-sensitive-information"></a>4.2: Aislamiento de los sistemas que almacenan o procesan información confidencial

**Instrucciones**: Implemente suscripciones y/o grupos de administración independientes para los entornos de desarrollo, prueba y producción. Las aplicaciones de Azure Functions deben estar separadas en cada red virtual o subred, y estar debidamente etiquetadas.

También se puede usar puntos de conexión privados para realizar el aislamiento de red. Un punto de conexión privado de Azure es una interfaz de red que permite conectarse de forma privada y segura a un servicio (por ejemplo, punto de conexión HTTPS de la aplicación de Azure Functions) con la tecnología de Azure Private Link. El punto de conexión privado usa una dirección IP privada de la red virtual, colocando el servicio de manera eficaz en su red virtual. Los puntos de conexión privados se encuentran en versión preliminar en las aplicaciones de Functions que se ejecutan en el plan Premium. Asegúrese de que los puntos de conexión privados no tengan ya el estado de versión preliminar antes de usarlos con las cargas de trabajo de producción.

- [Creación de suscripciones adicionales de Azure](../cost-management-billing/manage/create-subscription.md)

- [Creación de grupos de administración](../governance/management-groups/create.md)

- [Creación y uso de etiquetas](../azure-resource-manager/management/tag-resources.md)

- [Opciones de redes de Azure Functions](./functions-networking-options.md)

- [Plan Premium de Azure Functions](./functions-scale.md#premium-plan)

- [Descripción de puntos de conexión privados](../private-link/private-endpoint-overview.md)

- [Uso de puntos de conexión privados con Azure Functions](../app-service/networking/private-endpoint.md)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="43-monitor-and-block-unauthorized-transfer-of-sensitive-information"></a>4.3: Supervisión y bloqueo de una transferencia no autorizada de información confidencial

**Guía**: Implemente una herramienta automatizada en los perímetros de red que supervise la transferencia no autorizada de información confidencial y bloquee dichas transferencias al tiempo que alerta a los profesionales de seguridad de la información. 

Microsoft administra la infraestructura subyacente de Azure Functions y ha implementado controles estrictos para evitar la pérdida o exposición de los datos de los clientes.

- [Descripción de la protección de datos de los clientes en Azure](../security/fundamentals/protection-customer-data.md)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: No aplicable

### <a name="44-encrypt-all-sensitive-information-in-transit"></a>4.4: Cifrado de toda la información confidencial en tránsito

**Guía**: En la instancia de Azure Portal de las aplicaciones de Azure Functions, en "Características de la plataforma:  Redes: SSL ", habilite la opción"Solo HTTPS" y establezca la versión mínima de TLS en 1.2.

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="45-use-an-active-discovery-tool-to-identify-sensitive-data"></a>4.5: Uso de una herramienta de detección activa para identificar datos confidenciales

**Instrucciones**: no está disponible en estos momentos; las características de identificación, clasificación y prevención de pérdida de datos no están disponibles actualmente en Azure Functions. Etiquete las aplicaciones de Functions que podrían procesar información confidencial e implemente una solución de terceros, si es necesario por motivos de cumplimiento.

En el caso de la plataforma subyacente administrada por Microsoft, Microsoft trata todo el contenido de los clientes como confidencial y hace grandes esfuerzos para proteger a los clientes contra la pérdida y exposición de sus datos. Para garantizar la seguridad de los datos de los clientes dentro de Azure, Microsoft ha implementado y mantiene un conjunto de controles y funcionalidades eficaces de protección de datos.

- [Descripción de la protección de datos de los clientes en Azure](../security/fundamentals/protection-customer-data.md)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="46-use-azure-rbac-to-control-access-to-resources"></a>4.6: Uso de RBAC de Azure para controlar el acceso a los recursos

**Guía**: Use el control de acceso basado en roles (RBAC) de Azure Active Directory (AD) para controlar el acceso al plano de control de Azure Functions (Azure Portal). 

- [Configuración de RBAC en Azure](../role-based-access-control/role-assignments-portal.md)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="47-use-host-based-data-loss-prevention-to-enforce-access-control"></a>4.7: Uso de la prevención de pérdida de datos basada en host para aplicar el control de acceso

**Guía**: no aplicable; esta recomendación está diseñada para recursos de proceso de IaaS.

Microsoft administra la infraestructura subyacente de Azure Functions y ha implementado controles estrictos para evitar la pérdida o exposición de los datos de los clientes.

- [Descripción de la protección de datos de los clientes en Azure](../security/fundamentals/protection-customer-data.md)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="48-encrypt-sensitive-information-at-rest"></a>4.8: Cifrado de información confidencial en reposo

**Guía**: Al crear una aplicación de funciones, debe crear o vincular una cuenta de Azure Storage de uso general compatible con Blob, Queue y Table Storage. El motivo es que Azure Functions usa Azure Storage con ciertas operaciones; por ejemplo, para administrar los desencadenadores y ejecutar las funciones de registro. Azure Storage cifra todos los datos de las cuentas de almacenamiento en reposo. De manera predeterminada, los datos se cifran con claves administradas por Microsoft. Para tener un mayor control sobre las claves de cifrado, puede proporcionar claves administradas por el cliente que pueden usarse para cifrar datos de archivos y blobs. Estas claves deben estar presentes en Azure Key Vault para que la aplicación de Azure Functions pueda acceder a la cuenta de almacenamiento.

- [Consideraciones de almacenamiento de Azure Functions](./storage-considerations.md)

- [Cifrado de Azure Storage para datos en reposo](../storage/common/storage-service-encryption.md)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Compartido

### <a name="49-log-and-alert-on-changes-to-critical-azure-resources"></a>4.9: Registro y alerta de cambios en los recursos críticos de Azure

**Guía**: Use Azure Monitor con el registro de actividad de Azure para crear alertas cuando se produzcan cambios en las aplicaciones de producción de Azure Functions y en otros recursos críticos o relacionados.

- [Creación de alertas para los eventos del registro de actividad de Azure](../azure-monitor/platform/alerts-activity-log.md)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

## <a name="vulnerability-management"></a>Administración de vulnerabilidades

*Para más información, consulte [Control de seguridad: Administración de vulnerabilidades](../security/benchmarks/security-control-vulnerability-management.md).*

### <a name="51-run-automated-vulnerability-scanning-tools"></a>5.1: Ejecute herramientas de análisis de vulnerabilidades automatizado

**Instrucciones**: Adopte un enfoque DevSecOps (desarrollo, seguridad y operaciones) para garantizar que las aplicaciones de Azure Functions son seguras y que se van a mantener lo más protegidas posible durante su ciclo de vida útil. El enfoque DevSecOps involucra al equipo de seguridad de la organización y sus funcionalidades en las prácticas de DevOps, lo que convierte la seguridad en una responsabilidad de todos los usuarios del equipo.

Siga también las recomendaciones de Azure Security Center para ayudar a proteger las aplicaciones de Azure Functions.

- [Incorporación de un mecanismo continuo para validar la seguridad en la canalización de CI/CD](/azure/devops/migrate/security-validation-cicd-pipeline?view=azure-devops)

- [Implementación de las recomendaciones de evaluación de vulnerabilidades de Azure Security Center](../security-center/security-center-vulnerability-assessment-recommendations.md)

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="52-deploy-an-automated-operating-system-patch-management-solution"></a>5.2: Implementación de una solución automatizada para la administración de las revisiones del sistema

**Guía**: no aplicable; esta recomendación está diseñada para recursos de proceso de IaaS.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="53-deploy-automated-third-party-software-patch-management-solution"></a>5.3: Implemente una solución automatizada de administración de revisiones de software de terceros

**Guía**: no aplicable; esta recomendación está diseñada para recursos de proceso de IaaS.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="54-compare-back-to-back-vulnerability-scans"></a>5.4: Compare los exámenes de vulnerabilidades opuestos

**Guía**: no aplicable; esta recomendación está diseñada para recursos de proceso de IaaS.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="55-use-a-risk-rating-process-to-prioritize-the-remediation-of-discovered-vulnerabilities"></a>5.5: Use un proceso de clasificación de riesgos para priorizar la corrección de las vulnerabilidades detectadas

**Guía**: Microsoft se encarga de administrar las vulnerabilidades en los sistemas subyacentes que admiten Azure Functions. Sin embargo, puede utilizar la gravedad de las recomendaciones de Azure Security Center y la puntuación de seguridad para medir el riesgo de su entorno. La puntuación segura se basa en el número de recomendaciones de Security Center que ha mitigado. Para dar prioridad a las recomendaciones que deben resolverse en primer lugar, tenga en cuenta la gravedad de cada una.

- [Guía de referencia sobre las recomendaciones de seguridad](../security-center/recommendations-reference.md)

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Compartido

## <a name="inventory-and-asset-management"></a>Inventario y administración de recursos

*Para más información, consulte [Control de seguridad: Administración de recursos y del inventario](../security/benchmarks/security-control-inventory-asset-management.md).*

### <a name="61-use-azure-asset-discovery"></a>6.1: Uso de la detección de recursos de Azure

**Guía**: Use Azure Resource Graph para consultar o detectar todos los recursos (por ejemplo, proceso, almacenamiento, red, puertos y protocolos, etc.) dentro de las suscripciones.  Asegúrese de que tiene los permisos adecuados (lectura) en el inquilino y enumere todas las suscripciones de Azure, así como los recursos de las suscripciones.

Aunque los recursos clásicos de Azure se pueden detectar a través de Resource Graph, se recomienda encarecidamente crear y usar los recursos de Azure Resource Manager que figuran a continuación.

- [Creación de consultas con Azure Resource Graph](../governance/resource-graph/first-query-portal.md)

- [Visualización de las suscripciones de Azure](/powershell/module/az.accounts/get-azsubscription?view=azps-3.0.0)

- [Descripción de Azure RBAC](../role-based-access-control/overview.md)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="62-maintain-asset-metadata"></a>6.2: Mantenimiento de metadatos de recursos

**Instrucciones**: Aplique etiquetas a los recursos de Azure que proporcionan metadatos para organizarlos de forma lógica en una taxonomía.

- [Creación y uso de etiquetas](../azure-resource-manager/management/tag-resources.md)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="63-delete-unauthorized-azure-resources"></a>6.3: Eliminación de recursos de Azure no autorizados

**Guía**: use el etiquetado, los grupos de administración y las suscripciones independientes, si procede, para organizar y realizar un seguimiento de los recursos de Azure. Concilie el inventario periódicamente y asegúrese de que los recursos no autorizados se eliminan de la suscripción de manera oportuna.

Además, use Azure Policy para establecer restricciones sobre el tipo de recursos que se pueden crear en las suscripciones del cliente con las siguientes definiciones de directiva integradas: tipos de recursos no permitidos y tipos de recursos permitidos

- [Creación de suscripciones adicionales de Azure](../cost-management-billing/manage/create-subscription.md)

- [Creación de grupos de administración](../governance/management-groups/create.md)

- [Creación y uso de etiquetas](../azure-resource-manager/management/tag-resources.md)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="64-maintain-an-inventory-of-approved-azure-resources-and-software-titles"></a>6.4: Mantenimiento de un inventario de los recursos de Azure aprobados y títulos de software

**Guía**: Defina los recursos de Azure y el software aprobados para los recursos de proceso.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="65-monitor-for-unapproved-azure-resources"></a>6.5: Supervisión de recursos de Azure no aprobados

**Guía**: Use Azure Policy para establecer restricciones en el tipo de recursos que se pueden crear en sus suscripciones. 

Use Azure Resource Graph para consultar o detectar recursos dentro de sus suscripciones.  Asegúrese de que todos los recursos de Azure presentes en el entorno estén aprobados. 

- [Configuración y administración de Azure Policy](../governance/policy/tutorials/create-and-manage.md)

- [Creación de consultas con Azure Graph](../governance/resource-graph/first-query-portal.md)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="66-monitor-for-unapproved-software-applications-within-compute-resources"></a>6.6: Supervisión de aplicaciones de software no aprobadas en recursos de proceso

**Guía**: no aplicable; esta recomendación está diseñada para recursos de proceso de IaaS.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="67-remove-unapproved-azure-resources-and-software-applications"></a>6.7: Eliminación de aplicaciones de software y recursos de Azure no aprobadas

**Guía**: no aplicable; esta recomendación está diseñada para recursos de proceso de IaaS.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="68-use-only-approved-applications"></a>6.8: Uso exclusivo de aplicaciones aprobadas

**Guía**: no aplicable; esta recomendación está diseñada para recursos de proceso de IaaS.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="69-use-only-approved-azure-services"></a>6.9: Uso exclusivo de servicios de Azure aprobados

**Guía**: use Azure Policy para establecer restricciones sobre el tipo de recursos que se pueden crear en las suscripciones del cliente con las siguientes definiciones de directiva integradas: Tipos de recursos no permitidos Tipos de recursos permitidos

- [Configuración y administración de Azure Policy](../governance/policy/tutorials/create-and-manage.md)

- [Denegación de un tipo de recurso específico con Azure Policy](../governance/policy/samples/index.md)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="610-implement-approved-application-list"></a>6.10: Implementación de la lista de aplicaciones aprobadas

**Guía**: no aplicable; esta recomendación está diseñada para recursos de proceso de IaaS.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="611-limit-users-ability-to-interact-with-azure-resources-manager-via-scripts"></a>6.11: Limitación de la capacidad de los usuarios para interactuar con Azure Resource Manager mediante scripts

**Guía**: Configure el acceso condicional de Azure para limitar la capacidad de los usuarios de interactuar con Azure Resource Manager configurando "Bloquear acceso" en la aplicación Microsoft Azure Management.

- [Configuración del acceso condicional para bloquear el acceso a Azure Resource Manager](../role-based-access-control/conditional-access-azure-management.md)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="612-limit-users-ability-to-execute-scripts-within-compute-resources"></a>6.12: Limitación de capacidad de los usuarios para ejecutar scripts en recursos de proceso

**Guía**: no aplicable; esta recomendación está diseñada para recursos de proceso de IaaS.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="613-physically-or-logically-segregate-high-risk-applications"></a>6.13: Segregación física o lógica de aplicaciones de alto riesgo

**Guía**: En el caso de las aplicaciones de Azure Functions que son sensibles o de alto riesgo, implemente suscripciones o grupos de administración independientes para proporcionar aislamiento.

Implemente aplicaciones de Azure Functions de alto riesgo en su propia red virtual (VNet). En Azure Functions, la seguridad perimetral se lleva a cabo utilizando redes virtuales. Las funciones que se ejecutan en el plan Premium o App Service Environment (ASE) se pueden integrar con redes virtuales. Elija la mejor arquitectura para su caso de uso.

- [Opciones de redes de Azure Functions](./functions-networking-options.md)

- [Plan Premium de Azure Functions](./functions-scale.md#premium-plan)

- [Consideraciones de red para un entorno de App Service Environment](../app-service/environment/network-info.md)

- [Creación de una instancia externa de App Service Environment](../app-service/environment/create-external-ase.md)

Creación de un ASE interno:

- [https://docs.microsoft.com/azure/app-service/environment/create-ilb-as](../virtual-network/quick-create-portal.md)

- [Creación de un grupo de seguridad de red con una configuración de seguridad](../virtual-network/tutorial-filter-network-traffic.md)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

## <a name="secure-configuration"></a>Configuración segura

*Para más información, consulte [Control de seguridad: Configuración segura](../security/benchmarks/security-control-secure-configuration.md).*

### <a name="71-establish-secure-configurations-for-all-azure-resources"></a>7.1: Establezca configuraciones seguras para todos los recursos de Azure

**Instrucciones**: Defina e implemente configuraciones de seguridad estándar para su aplicación de Azure Functions con Azure Policy. Utilice los alias de Azure Policy en el espacio de nombres "Microsoft.Batch" para crear directivas personalizadas que permitan auditar o aplicar la configuración de las aplicaciones de Azure Functions. También puede utilizar definiciones de directivas integradas como:
- La identidad administrada debe usarse en la aplicación de funciones.
- Se debe desactivar la depuración remota para Function App
- Acceso a Function App solo a través de HTTPS

- [Visualización de los alias de Azure Policy disponibles](/powershell/module/az.resources/get-azpolicyalias?view=azps-3.3.0)

- [Configuración y administración de Azure Policy](../governance/policy/tutorials/create-and-manage.md)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="72-establish-secure-operating-system-configurations"></a>7.2: Establezca configuraciones del sistema operativo seguras

**Instrucciones**: no aplicable; esta guía está pensada para recursos de proceso de IaaS.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="73-maintain-secure-azure-resource-configurations"></a>7.3: Mantenga configuraciones de recursos de Azure seguras

**Instrucciones**: Use la directiva de Azure [denegar] e [implementar si no existe] para aplicar la configuración segura en los recursos de Azure.

- [Configuración y administración de Azure Policy](../governance/policy/tutorials/create-and-manage.md)

- [Descripción de los efectos de Azure Policy](../governance/policy/concepts/effects.md)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="74-maintain-secure-operating-system-configurations"></a>7.4: Mantenga configuraciones del sistema operativo seguras

**Instrucciones**: no aplicable; aunque se pueden implementar funciones locales, esta guía está pensada para los recursos de proceso de IaaS. Si implementa funciones locales, la responsabilidad de crear una configuración segura en el entorno recaerá sobre usted.

- [Descripción de las funciones locales](./functions-runtime-install.md)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="75-securely-store-configuration-of-azure-resources"></a>7.5: Almacene de forma segura la configuración de los recursos de Azure

**Instrucciones**: almacene y administre plantillas de ARM y definiciones de directivas personalizadas de Azure de forma segura en el control de código fuente.

- [¿Qué es la infraestructura como código?](/azure/devops/learn/what-is-infrastructure-as-code)

- [Diseño de flujos de trabajo de directiva como código](../governance/policy/concepts/policy-as-code.md)

- [Almacenamiento de código en Azure DevOps](/azure/devops/repos/git/gitworkflow?view=azure-devops)

- [Documentación de Azure Repos](/azure/devops/repos/index?view=azure-devops)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="76-securely-store-custom-operating-system-images"></a>7.6: Almacene imágenes de sistema operativo personalizadas de forma segura

**Guía**: no aplicable; esta guía está pensada para recursos de proceso de IaaS.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="77-deploy-system-configuration-management-tools"></a>7.7: Implemente herramientas de administración de configuración del sistema

**Guía**: Use las definiciones integradas en Azure Policy junto con los alias de esta misma aplicación en el espacio de nombres "Microsoft.Web" para crear directivas personalizadas que permitan auditar y aplicar las configuraciones, así como enviar alertas sobre ellas. Además, desarrolle un proceso y una canalización para administrar las excepciones de las directivas.

- [Configuración y administración de Azure Policy](../governance/policy/tutorials/create-and-manage.md)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="78-deploy-system-configuration-management-tools-for-operating-systems"></a>7.8: Implemente herramientas de administración de configuración del sistema para sistemas operativos

**Guía**: no aplicable; esta guía está pensada para recursos de proceso de IaaS.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="79-implement-automated-configuration-monitoring-for-azure-services"></a>7.9: Implemente la supervisión de configuración automatizada para servicios de Azure

**Guía**: Use las definiciones integradas en Azure Policy junto con los alias de esta misma aplicación en el espacio de nombres "Microsoft.Web" para crear directivas personalizadas que permitan auditar y aplicar las configuraciones, así como enviar alertas sobre ellas. Use la directiva de Azure [audit], [deny] y [deploy if not exist] para aplicar automáticamente las configuraciones en los recursos de Azure.

- [Configuración y administración de Azure Policy](../governance/policy/tutorials/create-and-manage.md)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="710-implement-automated-configuration-monitoring-for-operating-systems"></a>7.10: Implemente la supervisión de configuración automatizada para sistemas operativos

**Guía**: no aplicable; esta guía está pensada para recursos de proceso de IaaS.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="711-manage-azure-secrets-securely"></a>7.11: Administre los secretos de Azure de forma segura

**Guía**: Utilice identidades administradas con Azure Key Vault para simplificar y proteger la administración de secretos de las aplicaciones en la nube. Las identidades administradas permiten que la aplicación de Functions se autentique en cualquier servicio que admita la autenticación de Azure AD, como Key Vault, sin necesidad de usar credenciales en el código.

- [Creación de un almacén de claves](../key-vault/secrets/quick-create-portal.md)

- [Cómo usar identidades administradas para App Service y Azure Functions](../app-service/overview-managed-identity.md)

- [Cómo proporcionar la autenticación de Key Vault con una identidad administrada](../key-vault/general/managed-identity.md)

- [Uso de referencias de Key Vault para App Service y Azure Functions](../app-service/app-service-key-vault-references.md)

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="712-manage-identities-securely-and-automatically"></a>7.12: Administre las identidades de forma segura y automática

**Guía**: Use las identidades administradas para proporcionar a la aplicación de Azure Functions una identidad administrada automáticamente en Azure AD. Las identidades administradas le permiten autenticarse en cualquier servicio que admita la autenticación de Azure AD, incluyendo Key Vault, sin necesidad de credenciales en el código.

- [Cómo usar identidades administradas para App Service y Azure Functions](../app-service/overview-managed-identity.md)

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="713-eliminate-unintended-credential-exposure"></a>7.13: Elimine la exposición de credenciales no intencionada

**Instrucciones**: Implemente el escáner de credenciales para identificar las credenciales en el código. El escáner de credenciales también fomenta el traslado de credenciales detectadas a ubicaciones más seguras, como Azure Key Vault. 

- [Configuración del escáner de credenciales](https://secdevtools.azurewebsites.net/helpcredscan.html)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

## <a name="malware-defense"></a>Defensa contra malware

*Para más información, consulte [Control de seguridad: defensa contra malware](../security/benchmarks/security-control-malware-defense.md).*

### <a name="81-use-centrally-managed-anti-malware-software"></a>8.1: Use software antimalware administrado centralmente

**Guía**: no aplicable; esta guía está pensada para recursos de proceso de IaaS.

Microsoft Antimalware está habilitado en el host subyacente que admite los servicios de Azure (como Azure Functions), pero no se ejecuta en el contenido del cliente.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Microsoft

### <a name="82-pre-scan-files-to-be-uploaded-to-non-compute-azure-resources"></a>8.2: Examine previamente los archivos que se van a cargar en recursos de Azure que no son de proceso

**Guía**: No aplicable; esta recomendación está pensada para los recursos que no son de proceso y que están diseñados para almacenar datos.


**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="83-ensure-anti-malware-software-and-signatures-are-updated"></a>8.3: Asegúrese de que se han actualizado el software y las firmas antimalware

**Guía**: No aplicable; esta recomendación está pensada para los recursos que no son de proceso y que están diseñados para almacenar datos.

Microsoft Antimalware está habilitado en el host subyacente que admite los servicios de Azure (como Azure Functions), pero no se ejecuta en el contenido del cliente.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

## <a name="data-recovery"></a>Recuperación de datos

*Para más información, consulte [Control de seguridad: recuperación de datos](../security/benchmarks/security-control-data-recovery.md).*

### <a name="91-ensure-regular-automated-back-ups"></a>9.1: Garantía de copias de seguridad automáticas periódicas

**Guía**: utilice la característica Copias de seguridad y restauración para programar copias de seguridad periódicas de la aplicación. Las aplicaciones de Functions que se ejecutan en el plan Premium tienen las mismas funcionalidades de hospedaje que las aplicaciones web de Azure App Service, como la característica de copia de seguridad y restauración.

También puede usar una solución de control de código fuente como Azure Repos y Azure DevOps para almacenar y administrar el código de forma segura. Azure DevOps Services utiliza muchas de las características de almacenamiento de Azure para garantizar la disponibilidad de los datos en caso de un error de hardware, una interrupción del servicio o un desastre en la región. Además, el equipo de Azure DevOps sigue los procedimientos para proteger los datos contra la eliminación accidental o malintencionada.

- [Realizar una copia de seguridad de la aplicación en Azure](../app-service/manage-backup.md)

- [Descripción de la disponibilidad de los datos en Azure DevOps](/azure/devops/organizations/security/data-protection?view=azure-devops#data-availability)

- [Almacenamiento de código en Azure DevOps](/azure/devops/repos/git/gitworkflow?view=azure-devops)

- [Documentación de Azure Repos](/azure/devops/repos/index?view=azure-devops)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="92-perform-complete-system-backups-and-backup-any-customer-managed-keys"></a>9.2: Realización de copias de seguridad completas del sistema y copia de seguridad de las claves administradas por el cliente

**Guía**: utilice la característica Copias de seguridad y restauración para programar copias de seguridad periódicas de la aplicación. Las aplicaciones de Functions que se ejecutan en el plan Premium tienen las mismas funcionalidades de hospedaje que las aplicaciones web de Azure App Service, como la característica de copia de seguridad y restauración. Realice una copia de seguridad de las claves administradas del cliente con Azure Key Vault.

También puede usar una solución de control de código fuente como Azure Repos y Azure DevOps para almacenar y administrar el código de forma segura. Azure DevOps Services utiliza muchas de las características de almacenamiento de Azure para garantizar la disponibilidad de los datos en caso de un error de hardware, una interrupción del servicio o un desastre en la región. Además, el equipo de Azure DevOps sigue los procedimientos para proteger los datos contra la eliminación accidental o malintencionada.

- [Realizar una copia de seguridad de la aplicación en Azure](../app-service/manage-backup.md)

- [Creación de una copia de seguridad de las claves del almacén de claves en Azure](/powershell/module/azurerm.keyvault/backup-azurekeyvaultkey)

- [Descripción de la disponibilidad de los datos en Azure DevOps](/azure/devops/organizations/security/data-protection?view=azure-devops#data-availability)

- [Almacenamiento de código en Azure DevOps](/azure/devops/repos/git/gitworkflow?view=azure-devops)

- [Documentación de Azure Repos](/azure/devops/repos/index?view=azure-devops)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="93-validate-all-backups-including-customer-managed-keys"></a>9.3: Validación de todas las copias de seguridad, incluidas las claves administradas por el cliente

**Guía**: garantice la capacidad para restaurar los datos periódicamente desde la característica Copias de seguridad y restauración. Si usa otra ubicación sin conexión para realizar una copia de seguridad del código, no olvide realizar periódicamente restauraciones completas. Pruebe la restauración de las claves administradas por el cliente de la copia de seguridad.

- [Restauración de una aplicación de Azure desde una copia de seguridad](../app-service/web-sites-restore.md)

- [Restauración de una aplicación en Azure desde una instantánea](../app-service/app-service-web-restore-snapshots.md)

- [Restauración de las claves del almacén de claves en Azure](/powershell/module/azurerm.keyvault/restore-azurekeyvaultkey?view=azurermps-6.13.0)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="94-ensure-protection-of-backups-and-customer-managed-keys"></a>9.4: Garantía de la protección de las copias de seguridad y las claves administradas del cliente

**Guía**: Las copias de seguridad de la característica Copias de seguridad y restauración usan una cuenta de Azure Storage de la suscripción. Azure Storage cifra todos los datos de las cuentas de almacenamiento en reposo. De manera predeterminada, los datos se cifran con claves administradas por Microsoft. Para tener un mayor control sobre las claves de cifrado, puede proporcionar claves administradas por el cliente para cifrar los datos de almacenamiento.

Si usa claves administradas por el cliente, asegúrese de que la eliminación temporal esté habilitada en Key Vault para evitar la eliminación accidental o malintencionada de las claves.

- [Cifrado de Azure Storage para datos en reposo](../storage/common/storage-service-encryption.md)

- [Habilitación de la eliminación temporal en Key Vault](../storage/blobs/soft-delete-overview.md?tabs=azure-portal)

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Compartido

## <a name="incident-response"></a>Respuesta a los incidentes

*Para más información, consulte [Control de seguridad: Respuesta a los incidentes](../security/benchmarks/security-control-incident-response.md).*

### <a name="101-create-an-incident-response-guide"></a>10.1: Creación de una guía de respuesta ante incidentes

**Instrucciones**: Cree una guía de respuesta a incidentes para su organización. Asegúrese de que haya planes de respuesta a incidentes escritos que definan todos los roles del personal, así como las fases de administración y gestión de los incidentes, desde la detección hasta la revisión posterior a la incidencia.

- [Configuración de las automatizaciones de flujos de trabajo en Azure Security Center](../security-center/security-center-planning-and-operations-guide.md)

- [Guía para crear su propio proceso de respuesta a incidentes de seguridad](https://msrc-blog.microsoft.com/2019/07/01/inside-the-msrc-building-your-own-security-incident-response-process/)

- [Anatomía de un incidente del Centro de respuestas de seguridad de Microsoft](https://msrc-blog.microsoft.com/2019/07/01/inside-the-msrc-building-your-own-security-incident-response-process/)

- [El cliente también puede usar la guía de control de incidentes de seguridad de equipos de NIST como referencia para la creación de su propio plan de respuesta a incidentes](https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-61r2.pdf)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="102-create-an-incident-scoring-and-prioritization-procedure"></a>10.2: Creación de un procedimiento de priorización y puntuación de incidentes

**Instrucciones**: Security Center asigna una gravedad a cada alerta para ayudarle a priorizar aquellas que se deben investigar en primer lugar. La gravedad se basa en la confianza que tiene Security Center en la búsqueda o en el análisis utilizados para emitir la alerta, así como en el nivel de confianza de que ha habido un intento malintencionado detrás de la actividad que ha provocado la alerta.

Además, marque claramente las suscripciones (por ejemplo, producción, no producción) y cree un sistema de nomenclatura para identificar y clasificar claramente los recursos de Azure.

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Compartido

### <a name="103-test-security-response-procedures"></a>10.3: Prueba de los procedimientos de respuesta de seguridad

**Instrucciones**: Realice ejercicios para probar las capacidades de respuesta a los incidentes de los sistemas con regularidad. Identifique puntos débiles y brechas y revise el plan según sea necesario.

- [Consulte la publicación de NIST: Guía para probar, entrenar y ejecutar programas para planes y funcionalidades de TI](https://nvlpubs.nist.gov/nistpubs/Legacy/SP/nistspecialpublication800-84.pdf)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="104-provide-security-incident-contact-details-and-configure-alert-notifications-for-security-incidents"></a>10.4: Provisión de detalles de contacto de incidentes de seguridad y configuración de notificaciones de alerta para incidentes de seguridad

**Guía**: La información de contacto del incidente de seguridad la utilizará Microsoft para ponerse en contacto con usted si Microsoft Security Response Center (MSRC) detecta que un tercero no autorizado o ilegal ha accedido a los datos del cliente.  Revise los incidentes después del hecho para asegurarse de que se resuelven los problemas.

- [Establecimiento del contacto de seguridad de Azure Security Center](../security-center/security-center-provide-security-contact-details.md)

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="105-incorporate-security-alerts-into-your-incident-response-system"></a>10.5: Incorporación de alertas de seguridad en el sistema de respuesta a incidentes

**Instrucciones**: Exporte sus alertas y recomendaciones de Azure Security Center mediante la característica de exportación continua. La exportación continua le permite exportar alertas y recomendaciones de forma manual o continua. Puede usar el conector de datos de Azure Security Center para transmitir las alertas a Azure Sentinel.

- [Configuración de la exportación continua](../security-center/continuous-export.md)

- [Transmisión de alertas a Azure Sentinel](../sentinel/connect-azure-security-center.md)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="106-automate-the-response-to-security-alerts"></a>10.6: Automatización de la respuesta a las alertas de seguridad

**Guía**: utilice la característica Automatización de flujo de trabajo de Azure Security Center para desencadenar automáticamente respuestas a las alertas y recomendaciones de seguridad con Logic Apps.

- [Configuración de la automatización de flujo de trabajo y Logic Apps](../security-center/workflow-automation.md)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

## <a name="penetration-tests-and-red-team-exercises"></a>Pruebas de penetración y ejercicios del equipo rojo

*Para más información, consulte [Control de seguridad: Pruebas de penetración y ejercicios del equipo rojo](../security/benchmarks/security-control-penetration-tests-red-team-exercises.md).*

### <a name="111-conduct-regular-penetration-testing-of-your-azure-resources-and-ensure-remediation-of-all-critical-security-findings"></a>11.1: Realice pruebas de penetración periódicas de los recursos de Azure y asegúrese de corregir todos los resultados de seguridad críticos

**Guía**: Siga las reglas de compromiso de Microsoft para asegurarse de que las pruebas de penetración no infrinjan las directivas de Microsoft. Use la estrategia de Microsoft y la ejecución de pruebas de penetración de sitios activos y ataques simulados en la infraestructura en la nube, los servicios y las aplicaciones administrados por Microsoft.

- [Reglas de interacción de las pruebas de penetración](https://www.microsoft.com/msrc/pentest-rules-of-engagement?rtc=1)

- [Equipo rojo de Microsoft Cloud](https://gallery.technet.microsoft.com/Cloud-Red-Teaming-b837392e)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Compartido

## <a name="next-steps"></a>Pasos siguientes

- Consulte la [prueba comparativa de seguridad de Azure](../security/benchmarks/overview.md).
- Obtenga más información sobre las [líneas de base de seguridad de Azure](../security/benchmarks/security-baselines-overview.md).