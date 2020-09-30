---
title: Base de referencia de seguridad de Azure para Azure Application Gateway
description: Base de referencia de seguridad de Azure para Azure Application Gateway
author: msmbaldwin
ms.service: security
ms.topic: conceptual
ms.date: 06/05/2020
ms.author: mbaldwin
ms.custom: subject-security-benchmark
ms.openlocfilehash: b470fa72518f805c10403931280bdec96bda0fbe
ms.sourcegitcommit: d68c72e120bdd610bb6304dad503d3ea89a1f0f7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89226546"
---
# <a name="azure-security-baseline-for-azure-application-gateway"></a>Base de referencia de seguridad de Azure para Azure Application Gateway

La base de referencia de seguridad de Azure para Azure Application Gateway contiene recomendaciones que le ayudarán a mejorar la estrategia de seguridad de la implementación.

La base de referencia de este servicio se extrae de la [versión 1.0 de Azure Security Benchmark](https://docs.microsoft.com/azure/security/benchmarks/overview), que proporciona recomendaciones sobre cómo puede proteger las soluciones en la nube en Azure con nuestra guía de procedimientos recomendados.

Para más información, consulte [Introducción a la línea de base de seguridad de Azure](https://docs.microsoft.com/azure/security/benchmarks/security-baselines-overview).

## <a name="network-security"></a>Seguridad de las redes

*Para más información, consulte [Control de seguridad: Seguridad de redes](https://docs.microsoft.com/azure/security/benchmarks/security-control-network-security).*

### <a name="11-protect-azure-resources-within-virtual-networks"></a>1.1: Protección de los recursos de Azure dentro de las redes virtuales

**Guía**: asegúrese de que todas las implementaciones de subred de Azure Application Gateway de Virtual Network tienen un grupo de seguridad de red (NSG) aplicado con controles de acceso a la red específicos para los puertos y orígenes de confianza de la aplicación. Aunque los grupos de seguridad de red se admiten en Azure Application Gateway, hay restricciones y requisitos que se deben cumplir para que Azure Application Gateway y NSG funcionen según lo esperado.

* [Descripción de las restricciones y los requisitos del uso de NSG con Azure Application Gateway](https://docs.microsoft.com/azure/application-gateway/configuration-overview)

* [Creación de un NSG con una configuración de seguridad](https://docs.microsoft.com/azure/virtual-network/tutorial-filter-network-traffic)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="12-monitor-and-log-the-configuration-and-traffic-of-virtual-networks-subnets-and-nics"></a>1.2: Supervisión y registro de la configuración y el tráfico de redes virtuales, subredes y NIC

**Guía**: en los grupos de seguridad de red (NSG) asociados a las subredes de Azure Application Gateway, habilite los registros de flujo de estos grupos y envíe registros a una cuenta de almacenamiento para realizar auditorías del tráfico. También puede enviar registros de flujo de grupo de seguridad de red a un área de trabajo de Log Analytics y usar el Análisis de tráfico para proporcionar información detallada sobre el flujo de tráfico en la nube de Azure. Algunas de las ventajas del Análisis de tráfico son la capacidad de visualizar la actividad de la red e identificar las zonas activas, identificar las amenazas de seguridad, comprender los patrones de flujo de tráfico y detectar configuraciones de red incorrectas.

Nota: Hay algunos casos en los que los registros de flujo de NSG asociados a las subredes de Azure Application Gateway no muestran el tráfico que se ha permitido. Si la configuración coincide con el siguiente escenario, no verá el tráfico permitido en los registros de flujo de NSG:
- Ha implementado Application Gateway V2.
- Tiene un NSG en la subred de Application Gateway.
- Ha habilitado los registros de flujo de NSG en ese NSG.

* [Habilitación de los registros de flujo de NSG](https://docs.microsoft.com/azure/network-watcher/network-watcher-nsg-flow-logging-portal)

* [Habilitación y uso del Análisis de tráfico](https://docs.microsoft.com/azure/network-watcher/traffic-analytics)

* [Descripción de la seguridad de red proporcionada por Azure Security Center](https://docs.microsoft.com/azure/security-center/security-center-network-recommendations)

* [Preguntas más frecuentes sobre el diagnóstico y el registro de Azure Application Gateway](https://docs.microsoft.com/azure/application-gateway/application-gateway-faq#diagnostics-and-logging)

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="13-protect-critical-web-applications"></a>1.3: Proteja las aplicaciones web críticas

**Guía**: Implemente el firewall de aplicaciones web (WAF) de Azure delante de las aplicaciones web críticas para la inspección adicional del tráfico entrante. El firewall de aplicaciones web (WAF) es un servicio (característica de Azure Application Gateway) que proporciona a las aplicaciones web una protección centralizada contra vulnerabilidades de seguridad comunes. El firewall de aplicaciones web de Azure puede ayudarle a proteger las aplicaciones web de Azure App Service al inspeccionar el tráfico web entrante a fin de bloquear ataques como inyecciones de código SQL, scripts de sitios, cargas de malware y ataques de denegación de servicio distribuido. El firewall de aplicaciones web se basa en las reglas contenidas en los conjuntos de reglas básicas de OWASP (Open Web Application Security Project) 3.1 (solo WAF_v2), 3.0 y 2.2.9.

* [Características de Azure Application Gateway](https://docs.microsoft.com/azure/application-gateway/features)

* [WAF de Azure](https://docs.microsoft.com/azure/web-application-firewall/ag/ag-overview)

* [Implementación de WAF de Azure](https://docs.microsoft.com/azure/web-application-firewall/ag/create-waf-policy-ag)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="14-deny-communications-with-known-malicious-ip-addresses"></a>1.4: Deniegue las comunicaciones con direcciones IP malintencionadas conocidas

**Instrucciones**: habilite la protección contra DDoS estándar en las redes virtuales de Azure asociadas a las instancias de producción de Azure Application Gateway para protegerse frente a ataques de DDoS. Use la inteligencia sobre amenazas integrada en Azure Security Center para denegar las comunicaciones con direcciones IP malintencionadas conocidas.

* [Configuración de la protección contra DDoS](https://docs.microsoft.com/azure/virtual-network/manage-ddos-protection)

* [Descripción de la inteligencia sobre amenazas integrada de Azure Security Center](https://docs.microsoft.com/azure/security-center/security-center-alerts-service-layer)

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="15-record-network-packets"></a>1.5: Registro de los paquetes de red

**Instrucciones**: en los grupos de seguridad de red (NSG) asociados a las subredes de Azure Application Gateway, habilite los registros de flujo de estos grupos y envíe registros a una cuenta de almacenamiento para realizar auditorías del tráfico. También puede enviar registros de flujo de grupo de seguridad de red a un área de trabajo de Log Analytics y usar el Análisis de tráfico para proporcionar información detallada sobre el flujo de tráfico en la nube de Azure. Algunas de las ventajas del Análisis de tráfico son la capacidad de visualizar la actividad de la red e identificar las zonas activas, identificar las amenazas de seguridad, comprender los patrones de flujo de tráfico y detectar configuraciones de red incorrectas.

Nota: Hay algunos casos en los que los registros de flujo de NSG asociados a las subredes de Azure Application Gateway no muestran el tráfico que se ha permitido. Si la configuración coincide con el siguiente escenario, no verá el tráfico permitido en los registros de flujo de NSG:
- Ha implementado Application Gateway V2.
- Tiene un NSG en la subred de Application Gateway.
- Ha habilitado los registros de flujo de NSG en ese NSG.

* [Habilitación de los registros de flujo de NSG](https://docs.microsoft.com/azure/network-watcher/network-watcher-nsg-flow-logging-portal)

* [Habilitación y uso del Análisis de tráfico](https://docs.microsoft.com/azure/network-watcher/traffic-analytics)

* [Descripción de la seguridad de red proporcionada por Azure Security Center](https://docs.microsoft.com/azure/security-center/security-center-network-recommendations)

* [Preguntas más frecuentes sobre el diagnóstico y el registro de Azure Application Gateway](https://docs.microsoft.com/azure/application-gateway/application-gateway-faq#diagnostics-and-logging)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="16-deploy-network-based-intrusion-detectionintrusion-prevention-systems-idsips"></a>1.6: Implemente sistemas de prevención de intrusiones y detección de intrusiones (IDS/IPS) basados en la red

**Guía**: Implemente el firewall de aplicaciones web (WAF) de Azure delante de las aplicaciones web críticas para la inspección adicional del tráfico entrante. El firewall de aplicaciones web (WAF) es un servicio (característica de Azure Application Gateway) que proporciona a las aplicaciones web una protección centralizada contra vulnerabilidades de seguridad comunes. El firewall de aplicaciones web de Azure puede ayudarle a proteger las aplicaciones web de Azure App Service al inspeccionar el tráfico web entrante a fin de bloquear ataques como inyecciones de código SQL, scripts de sitios, cargas de malware y ataques de denegación de servicio distribuido. El firewall de aplicaciones web se basa en las reglas contenidas en los conjuntos de reglas básicas de OWASP (Open Web Application Security Project) 3.1 (solo WAF_v2), 3.0 y 2.2.9.

Existen también otras opciones en el mercado, como el WAF de Barracuda para Azure, que están disponibles en Azure Marketplace con las características IDS/IPS.

* [Características de Azure Application Gateway](https://docs.microsoft.com/azure/application-gateway/features)

* [WAF de Azure](https://docs.microsoft.com/azure/web-application-firewall/ag/ag-overview)

* [Implementación de WAF de Azure](https://docs.microsoft.com/azure/web-application-firewall/ag/create-waf-policy-ag)

* [Descripción del servicio en la nube Barracuda WAF](https://docs.microsoft.com/azure/app-service/environment/app-service-app-service-environment-web-application-firewall#configuring-your-barracuda-waf-cloud-service)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="17-manage-traffic-to-web-applications"></a>1.7: Administre el tráfico a las aplicaciones web

**Instrucciones**: Implemente Application Gateway de Azure para las aplicaciones web con HTTPS/SSL habilitado para los certificados de confianza.

* [Implementación de Application Gateway](https://docs.microsoft.com/azure/application-gateway/quick-create-portal)

* [Configuración de Application Gateway para usar HTTPS](https://docs.microsoft.com/azure/application-gateway/create-ssl-portal)

* [Descripción del equilibrio de carga de capa 7 con las puertas de enlace de aplicaciones web de Azure](https://docs.microsoft.com/azure/application-gateway/overview)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="18-minimize-complexity-and-administrative-overhead-of-network-security-rules"></a>1.8: Minimice la complejidad y la sobrecarga administrativa de las reglas de seguridad de red

**Guía**: Puede usar etiquetas de servicio de Virtual Network para definir controles de acceso a la red en grupos de seguridad de red o Azure Firewall. Puede utilizar etiquetas de servicio en lugar de direcciones IP específicas al crear reglas de seguridad. Al especificar el nombre de la etiqueta de servicio (por ejemplo, GatewayManager) en el campo de origen o destino apropiado de una regla, puede permitir o denegar el tráfico en el servicio correspondiente. Microsoft administra los prefijos de direcciones que la etiqueta de servicio incluye y actualiza automáticamente dicha etiqueta a medida que las direcciones cambian.

En el caso de los grupos de seguridad de red (NSG) asociados a las subredes de Azure Application Gateway, debe permitir el tráfico entrante de Internet en los puertos TCP 65503-65534 para la SKU de Application Gateway v1, y los puertos TCP 65200-65535 para la SKU de v2 con la subred de destino como Cualquiera y el origen como etiqueta de servicio de GatewayManager. Este intervalo de puertos es necesario para la comunicación de la infraestructura de Azure. Estos puertos están protegidos (bloqueados) mediante certificados de Azure. Las entidades externas, incluidos los clientes de esas puertas de enlace, no pueden comunicarse en estos puntos de conexión.

* [Descripción y uso de las etiquetas de servicio](https://docs.microsoft.com/azure/virtual-network/service-tags-overview)

* [Introducción a la configuración de Azure Application Gateway](https://docs.microsoft.com/azure/application-gateway/configuration-overview)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="19-maintain-standard-security-configurations-for-network-devices"></a>1.9: Mantenga las configuraciones de seguridad estándar para dispositivos de red

**Guía**: defina e implemente configuraciones de seguridad estándar para la configuración de red relacionada con las implementaciones de Azure Application Gateway. Use alias de Azure Policy en el espacio de nombres "Microsoft.Network" para crear directivas personalizadas que permitan auditar o aplicar la configuración de red de los grupos de seguridad de red, Azure Application Gateways y Azure Virtual Networks. También puede usar definiciones de directivas integradas.

También puede utilizar Azure Blueprints para simplificar las implementaciones de Azure a gran escala mediante el empaquetado de artefactos de entorno clave, como las plantillas de Azure Resource Manager, el control de acceso basado en rol de Azure (Azure RBAC) y las directivas, en una única definición de un plano técnico. Aplique fácilmente el plano técnico a nuevas suscripciones y entornos, y ajuste el control y la administración mediante el control de versiones.

* [Configuración y administración de Azure Policy](https://docs.microsoft.com/azure/governance/policy/tutorials/create-and-manage)

* [Creación de un plano técnico de Azure](https://docs.microsoft.com/azure/governance/blueprints/create-blueprint-portal)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="110-document-traffic-configuration-rules"></a>1.10: Documente las reglas de configuración de tráfico

**Guía**: use etiquetas para los grupos de seguridad de red (NSG) asociados con la subred de Azure Application Gateway, así como cualquier otro recurso relacionado con la seguridad de red y el flujo de tráfico. En el caso de las reglas de NSG individuales, use el campo "Descripción" para especificar las necesidades empresariales o la duración (etc.) de las reglas que permiten que entre o salga el tráfico en una red.

Use cualquiera de las definiciones de directiva de Azure integradas relacionadas con el etiquetado, como "Requerir etiqueta y su valor" para asegurarse de que todos los recursos se crean con etiquetas y para notificarle los recursos no etiquetados existentes.

Puede usar Azure PowerShell o la CLI de Azure para buscar o realizar acciones en los recursos en función de sus etiquetas.

* [Creación y uso de etiquetas](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-using-tags)

* [Creación de una red virtual](https://docs.microsoft.com/azure/virtual-network/quick-create-portal)

* [Creación de un grupo de seguridad de red con una configuración de seguridad](https://docs.microsoft.com/azure/virtual-network/tutorial-filter-network-traffic)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="111-use-automated-tools-to-monitor-network-resource-configurations-and-detect-changes"></a>1.11: Use herramientas automatizadas para supervisar las configuraciones de recursos de red y detectar cambios

**Guía**: use el registro de actividad de Azure para supervisar las configuraciones de los recursos de red y detectar cambios en las opciones y recursos de red relacionados con las implementaciones de Azure Application Gateway. Cree alertas en Azure Monitor que se desencadenen cuando se produzcan cambios en la configuraciones o recursos de red críticos.

* [Visualización y recuperación de eventos del registro de actividad de Azure](https://docs.microsoft.com/azure/azure-monitor/platform/activity-log-view)

* [Creación de alertas en Azure Monitor](https://docs.microsoft.com/azure/azure-monitor/platform/alerts-activity-log)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

## <a name="logging-and-monitoring"></a>Registro y supervisión

*Para más información, consulte [Control de seguridad: registro y supervisión](https://docs.microsoft.com/azure/security/benchmarks/security-control-logging-monitoring).*

### <a name="21-use-approved-time-synchronization-sources"></a>2.1: Uso de orígenes de sincronización de hora aprobados

**Guía**: Microsoft mantiene el origen de la hora usado en los recursos de Azure, como Azure App Service.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Microsoft

### <a name="22-configure-central-security-log-management"></a>2.2: Configuración de la administración central de registros de seguridad

**Guía**: para el registro de la auditoria del plano de control, habilite la configuración de diagnóstico del registro de actividad de Azure y envíela a un área de trabajo de Log Analytics, un centro de eventos de Azure o una cuenta de almacenamiento de Azure. Con los datos del registro de actividad de Azure, puede determinar el "qué, quién y cuándo" de las operaciones de escritura (PUT, POST, DELETE) realizadas en el nivel de plano de control para Azure Application Gateway y recursos seleccionados, como los grupos de seguridad de red (NSG), que se usan para proteger la subred de Azure Application Gateway.

Además de los registros de actividad, puede configurar los valores de diagnóstico para las implementaciones de Azure Application Gateway. La configuración de diagnóstico se usa para configurar la exportación de streaming de registros y métricas de la plataforma para un recurso al destino que elija (Storage Accounts, Event Hubs y Log Analytics).

Azure Application Gateway también puede integrarse directamente con Azure Application Insights. Application Insights recopila información del registro, el rendimiento y los errores. Application Insights detecta automáticamente anomalías en el rendimiento e incluye herramientas de análisis eficaces que ayudan a diagnosticar problemas y comprender cómo se usan las aplicaciones web. Puede habilitar la exportación continua para exportar datos de telemetría desde Application Insights a una ubicación centralizada para mantener los datos durante más tiempo que el período de retención estándar.

* [Habilitación de la configuración de diagnóstico para el registro de actividad de Azure](https://docs.microsoft.com/azure/azure-monitor/platform/diagnostic-settings-legacy)

* [Habilitación de la configuración de diagnóstico de Azure Application Gateway](https://docs.microsoft.com/azure/application-gateway/application-gateway-diagnostics)

* [Habilitación de Application Insights](https://docs.microsoft.com/azure/azure-monitor/app/app-insights-overview)

* [Configuración de la exportación continua](https://docs.microsoft.com/azure/azure-monitor/app/export-telemetry)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="23-enable-audit-logging-for-azure-resources"></a>2.3: Habilitación del registro de auditoría para recursos de Azure

**Guía**: para el registro de la auditoria del plano de control, habilite la configuración de diagnóstico del registro de actividad de Azure y envíela a un área de trabajo de Log Analytics, un centro de eventos de Azure o una cuenta de almacenamiento de Azure. Con los datos del registro de actividad de Azure, puede determinar el "qué, quién y cuándo" de las operaciones de escritura (PUT, POST, DELETE) realizadas en el nivel de plano de control para Azure Application Gateway y recursos seleccionados, como los grupos de seguridad de red (NSG), que se usan para proteger la subred de Azure Application Gateway.

Además de los registros de actividad, puede configurar los valores de diagnóstico para las implementaciones de Azure Application Gateway. La configuración de diagnóstico se usa para configurar la exportación de streaming de registros y métricas de la plataforma para un recurso al destino que elija (Storage Accounts, Event Hubs y Log Analytics).

Azure Application Gateway también puede integrarse directamente con Azure Application Insights. Application Insights recopila información del registro, el rendimiento y los errores. Application Insights detecta automáticamente anomalías en el rendimiento e incluye herramientas de análisis eficaces que ayudan a diagnosticar problemas y comprender cómo se usan las aplicaciones web. Puede habilitar la exportación continua para exportar datos de telemetría desde Application Insights a una ubicación centralizada para mantener los datos durante más tiempo que el período de retención estándar.

* [Habilitación de la configuración de diagnóstico para el registro de actividad de Azure](https://docs.microsoft.com/azure/azure-monitor/platform/diagnostic-settings-legacy)

* [Habilitación de la configuración de diagnóstico de Azure Application Gateway](https://docs.microsoft.com/azure/application-gateway/application-gateway-diagnostics)

* [Habilitación de Application Insights](https://docs.microsoft.com/azure/azure-monitor/app/app-insights-overview)

* [Configuración de la exportación continua](https://docs.microsoft.com/azure/azure-monitor/app/export-telemetry)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="24-collect-security-logs-from-operating-systems"></a>2.4: Recopilación de registros de seguridad de sistemas operativos

**Guía**: no aplicable; esta recomendación está diseñada para recursos de proceso de IaaS.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="25-configure-security-log-storage-retention"></a>2.5: Configuración de la retención del almacenamiento de registros de seguridad

**Guía**: En Azure Monitor, establezca el período de retención del área de trabajo de Log Analytics de acuerdo con la normativa de cumplimiento de su organización. Use cuentas de Azure Storage para el almacenamiento de archivos a largo plazo.

* [Configuración de parámetros de retención de registros de áreas de trabajo de Log Analytics](https://docs.microsoft.com/azure/azure-monitor/platform/manage-cost-storage#change-the-data-retention-period)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="26-monitor-and-review-logs"></a>2.6: Supervisión y registros de revisión

**Guía**: habilite la configuración de diagnóstico del registro de actividad de Azure, así como la configuración de diagnóstico de Azure Application Gateway y envíe los registros a un área de trabajo de Log Analytics. Realice consultas en Log Analytics para buscar términos, identificar tendencias, analizar patrones y proporcionar otra información detallada basada en los datos recopilados.

Use Azure Monitor para redes para una vista completa del estado y las métricas de todos los recursos de red implementados, incluidos los Azure Application Gateways.

Si lo desea, también puede habilitar e incorporar datos en Azure Sentinel o en una herramienta de Administración de eventos e información de seguridad (SIEM) de terceros.

* [Habilitación de la configuración de diagnóstico para el registro de actividad de Azure](https://docs.microsoft.com/azure/azure-monitor/platform/diagnostic-settings-legacy)

* [Habilitación de la configuración de diagnóstico de Azure Application Gateway](https://docs.microsoft.com/azure/application-gateway/application-gateway-diagnostics)

* [Utilización de Azure Monitor para redes](https://docs.microsoft.com/azure/azure-monitor/insights/network-insights-overview)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="27-enable-alerts-for-anomalous-activities"></a>2.7: Habilitación de alertas para actividades anómalas

**Guía**: implemente el firewall de aplicaciones web (WAF) v2 SKU de Azure delante de las aplicaciones web críticas para la inspección adicional del tráfico entrante. El firewall de aplicaciones web (WAF) es un servicio (característica de Azure Application Gateway) que proporciona a las aplicaciones web una protección centralizada contra vulnerabilidades de seguridad comunes. El firewall de aplicaciones web de Azure puede ayudarle a proteger las aplicaciones web de Azure App Service al inspeccionar el tráfico web entrante a fin de bloquear ataques como inyecciones de código SQL, scripts de sitios, cargas de malware y ataques de denegación de servicio distribuido. El firewall de aplicaciones web se basa en las reglas contenidas en los conjuntos de reglas básicas de OWASP (Open Web Application Security Project) 3.1 (solo WAF_v2), 3.0 y 2.2.9.

Habilite la configuración de diagnóstico del registro de actividad de Azure, así como la configuración de diagnóstico de Azure WAF, y envíe los registros a un área de trabajo de Log Analytics. Realice consultas en Log Analytics para buscar términos, identificar tendencias, analizar patrones y proporcionar otra información detallada basada en los datos recopilados. Puede crear alertas basadas en las consultas del área de trabajo de Log Analytics.

Use Azure Monitor para redes para una vista completa del estado y las métricas de todos los recursos de red implementados, incluidos los Azure Application Gateways. En la consola de Azure Monitor para redes, puede ver y crear alertas para Azure Application Gateway.

* [Implementación de WAF de Azure](https://docs.microsoft.com/azure/web-application-firewall/ag/create-waf-policy-ag)

* [Habilitación de la configuración de diagnóstico para el registro de actividad de Azure](https://docs.microsoft.com/azure/azure-monitor/platform/diagnostic-settings-legacy)

* [Habilitación de la configuración de diagnóstico de Azure Application Gateway](https://docs.microsoft.com/azure/application-gateway/application-gateway-diagnostics)

* [Utilización de Azure Monitor para redes](https://docs.microsoft.com/azure/azure-monitor/insights/network-insights-overview)

* [Creación de alertas en Azure](https://docs.microsoft.com/azure/azure-monitor/learn/tutorial-response)

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="28-centralize-anti-malware-logging"></a>2.8: Centralización del registro antimalware

**Guía**: implemente el firewall de aplicaciones web (WAF) v2 de Azure delante de las aplicaciones web críticas para la inspección adicional del tráfico entrante. El firewall de aplicaciones web (WAF) es un servicio (característica de Azure Application Gateway) que proporciona a las aplicaciones web una protección centralizada contra vulnerabilidades de seguridad comunes. El firewall de aplicaciones web de Azure puede ayudarle a proteger las aplicaciones web de Azure App Service al inspeccionar el tráfico web entrante a fin de bloquear ataques como inyecciones de código SQL, scripts de sitios, cargas de malware y ataques de denegación de servicio distribuido.

Configure los valores de diagnóstico para las implementaciones de Azure Application Gateway. La configuración de diagnóstico se usa para configurar la exportación de streaming de registros y métricas de la plataforma para un recurso al destino que elija (Storage Accounts, Event Hubs y Log Analytics).

* [Implementación de WAF de Azure](https://docs.microsoft.com/azure/web-application-firewall/ag/create-waf-policy-ag)

* [Configuración de los valores de diagnóstico para Azure WAF](https://docs.microsoft.com/azure/application-gateway/application-gateway-diagnostics)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="29-enable-dns-query-logging"></a>2.9: Habilitación del registro de consultas DNS

**Guía**: no aplicable; Azure Application Gateway no procesa ni genera registros relacionados con DNS a los que el usuario pueda acceder.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="210-enable-command-line-audit-logging"></a>2.10: Habilitación del registro de auditoría de la línea de comandos

**Guía**: no aplicable; esta guía está pensada para recursos de proceso de IaaS.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

## <a name="identity-and-access-control"></a>Control de identidades y acceso

*Para más información, consulte [Control de seguridad: Identidad y control de acceso](https://docs.microsoft.com/azure/security/benchmarks/security-control-identity-access-control).*

### <a name="31-maintain-an-inventory-of-administrative-accounts"></a>3.1: Mantenga un inventario de cuentas administrativas

**Instrucciones**: Azure Active Directory (AD) tiene roles integrados que se deben asignar explícitamente y son consultables. Use el módulo de PowerShell de Azure AD para realizar consultas ad hoc para detectar cuentas que son miembros de grupos administrativos.

* [Obtención de un rol de directorio en Azure AD con PowerShell](https://docs.microsoft.com/powershell/module/azuread/get-azureaddirectoryrole?view=azureadps-2.0)

* [Obtención de los miembros de un rol de directorio en Azure AD con PowerShell](https://docs.microsoft.com/powershell/module/azuread/get-azureaddirectoryrolemember?view=azureadps-2.0)

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="32-change-default-passwords-where-applicable"></a>3.2: Cambie las contraseñas predeterminadas cuando proceda

**Guía**: el acceso del plano de control a Azure Application Gateway se controla mediante Azure Active Directory (AD). Azure AD no tiene el concepto de contraseñas predeterminadas.

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="33-use-dedicated-administrative-accounts"></a>3.3: Use cuentas administrativas dedicadas

**Guía**: Cree procedimientos operativos estándar en torno al uso de cuentas administrativas dedicadas. Use la administración de identidad y acceso de Azure Security Center para supervisar el número de cuentas administrativas.

Además, para ayudarle a realizar un seguimiento de las cuentas administrativas dedicadas, puede seguir las recomendaciones de Azure Security Center o las directivas integradas de Azure Policy, por ejemplo:
- Debe haber más de un propietario asignado a su suscripción
- Las cuentas en desuso con permisos de propietario deben quitarse de la suscripción
- Las cuentas externas con permisos de propietario deben quitarse de la suscripción

* [Uso de Azure Security Center para supervisar la identidad y el acceso (versión preliminar)](https://docs.microsoft.com/azure/security-center/security-center-identity-access)

* [Uso de Azure Policy](https://docs.microsoft.com/azure/governance/policy/tutorials/create-and-manage)

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="34-use-single-sign-on-sso-with-azure-active-directory"></a>3.4: Use el inicio de sesión único (SSO) con Azure Active Directory

**Guía**: use un registro de aplicaciones de Azure (entidad de servicio) para recuperar un token que se pueda usar para interactuar con los Azure Application Gateways a través de llamadas API.

* [Llamada a las API REST de Azure](https://docs.microsoft.com/rest/api/azure/#how-to-call-azure-rest-apis-with-postman)

* [Registro de la aplicación cliente (entidad de servicio) con Azure AD](https://docs.microsoft.com/rest/api/azure/#register-your-client-application-with-azure-ad)

* [Información de API de Azure Recovery Services](https://docs.microsoft.com/rest/api/recoveryservices/)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="35-use-multi-factor-authentication-for-all-azure-active-directory-based-access"></a>3.5: Use la autenticación multifactor para todo el acceso basado en Azure Active Directory

**Instrucciones**: Habilite la MFA de Azure AD y siga las recomendaciones de administración de identidades y acceso de Azure Security Center.

* [Procedimiento para habilitar la MFA en Azure](https://docs.microsoft.com/azure/active-directory/authentication/howto-mfa-getstarted)

* [Supervisión de la identidad y el acceso en Azure Security Center](https://docs.microsoft.com/azure/security-center/security-center-identity-access)

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="36-use-dedicated-machines-privileged-access-workstations-for-all-administrative-tasks"></a>3.6: Use máquinas dedicadas (estaciones de trabajo de acceso con privilegios) para todas las tareas administrativas

**Guía**: Utilice PAW (estaciones de trabajo de acceso privilegiado) con la MFA configurada para iniciar la sesión y configurar recursos de Azure.

* [Más información sobre las estaciones de trabajo con privilegios de acceso](https://docs.microsoft.com/windows-server/identity/securing-privileged-access/privileged-access-workstations)

* [Procedimiento para habilitar la MFA en Azure](https://docs.microsoft.com/azure/active-directory/authentication/howto-mfa-getstarted)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="37-log-and-alert-on-suspicious-activities-from-administrative-accounts"></a>3.7: Registro y alerta de actividades sospechosas desde cuentas administrativas

**Guía**: Use los informes de seguridad de Azure Active Directory para la generación de registros y alertas cuando se producen actividades sospechosas o no seguras en el entorno. Use Azure Security Center para supervisar la actividad de identidad y acceso.

* [Procedimiento para identificar usuarios de Azure AD marcados por una actividad de riesgo](https://docs.microsoft.com/azure/active-directory/reports-monitoring/concept-user-at-risk)

* [Supervisión de la actividad de identidad y acceso de los usuarios en Azure Security Center](https://docs.microsoft.com/azure/security-center/security-center-identity-access)

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="38-manage-azure-resources-from-only-approved-locations"></a>3.8: Administre los recursos de Azure solo desde ubicaciones aprobadas

**Guía**: Use ubicaciones con nombre de acceso condicional para permitir el acceso solo desde agrupaciones lógicas específicas de intervalos de direcciones IP o países o regiones.

* [Configuración de ubicaciones con nombre en Azure](https://docs.microsoft.com/azure/active-directory/reports-monitoring/quickstart-configure-named-locations)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="39-use-azure-active-directory"></a>3.9: Uso de Azure Active Directory

**Instrucciones**: Use Azure Active Directory (AAD) como el sistema central de autenticación y autorización. AAD protege los datos mediante un fuerte cifrado para los datos en reposo y en tránsito. AAD también cifra con sal, convierte en hash y almacena de forma segura las credenciales de los usuarios.

* [Creación y configuración de una instancia de AAD](https://docs.microsoft.com/azure/active-directory/fundamentals/active-directory-access-create-new-tenant)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="310-regularly-review-and-reconcile-user-access"></a>3.10: Revise y concilie regularmente el acceso de los usuarios

**Guía**: Azure AD proporciona registros para ayudar a descubrir cuentas obsoletas. Además, use las revisiones de acceso de identidad de Azure para administrar de forma eficiente las pertenencias a grupos, el acceso a las aplicaciones empresariales y las asignaciones de roles. El acceso de los usuarios se puede revisar de forma periódica para asegurarse de que solo las personas adecuadas tengan acceso continuado.

* [Descripción de los informes de Azure AD](https://docs.microsoft.com/azure/active-directory/reports-monitoring/)

* [Procedimiento para usar las revisiones de acceso de identidad de Azure](https://docs.microsoft.com/azure/active-directory/governance/access-reviews-overview)

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="311-monitor-attempts-to-access-deactivated-credentials"></a>3.11: Supervisión de los intentos de acceso a credenciales desactivadas

**Guía**: Tiene acceso a los orígenes de registro de eventos de actividad de inicio de sesión, auditoría y riesgo de Azure AD, que permiten la integración con cualquier herramienta de SIEM o supervisión.

Puede simplificar este proceso creando una configuración de diagnóstico para las cuentas de usuario de Azure Active Directory y enviando los registros de auditoría y los registros de inicio de sesión a un área de trabajo de Log Analytics. Puede configurar las alertas deseadas en el área de trabajo de Log Analytics.

* [Integración de los registros de actividad de Azure en Azure Monitor](https://docs.microsoft.com/azure/active-directory/reports-monitoring/howto-integrate-activity-logs-with-log-analytics)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="312-alert-on-account-login-behavior-deviation"></a>3.12: Alerte de las desviaciones de comportamiento en los inicios de sesión de las cuentas

**Instrucciones**: use las características de detección de riesgos y Azure AD Identity Protection para configurar las respuestas automatizadas a las acciones sospechosas detectadas relacionadas con las identidades de los usuarios. También puede hacer que Azure Sentinel ingiera los datos para investigarlos más.

* [Visualización de los inicios de sesión de riesgo de Azure AD](https://docs.microsoft.com/azure/active-directory/reports-monitoring/concept-risky-sign-ins)

* [Configuración y habilitación de las directivas de riesgo de protección de identidad](https://docs.microsoft.com/azure/active-directory/identity-protection/howto-identity-protection-configure-risk-policies)

* [Incorporación de Azure Sentinel](https://docs.microsoft.com/azure/sentinel/quickstart-onboard)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="313-provide-microsoft-with-access-to-relevant-customer-data-during-support-scenarios"></a>3.13: Proporcione a Microsoft acceso a los datos pertinentes del cliente durante los escenarios de soporte técnico

**Guía**: no aplicable; Caja de seguridad del cliente no es aplicable a Azure Application Gateway.

* [Lista de servicios admitidos por la Caja de seguridad del cliente](https://docs.microsoft.com/azure/security/fundamentals/customer-lockbox-overview#supported-services-and-scenarios-in-general-availability)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: No aplicable

## <a name="data-protection"></a>Protección de los datos

*Para más información, consulte [Control de seguridad: Protección de datos](https://docs.microsoft.com/azure/security/benchmarks/security-control-data-protection).*

### <a name="41-maintain-an-inventory-of-sensitive-information"></a>4.1: Mantenimiento de un inventario de información confidencial

**Guía**: Use etiquetas para ayudar a realizar el seguimiento de los recursos de Azure que almacenan o procesan información confidencial.

* [Creación y uso de etiquetas](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-using-tags)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="42-isolate-systems-storing-or-processing-sensitive-information"></a>4.2: Aislamiento de los sistemas que almacenan o procesan información confidencial

**Instrucciones**: Implemente suscripciones y/o grupos de administración independientes para los entornos de desarrollo, prueba y producción. Asegúrese de que todas las implementaciones de subred de Azure Application Gateway de Virtual Network tienen un grupo de seguridad de red (NSG) aplicado con controles de acceso a la red específicos para los puertos y orígenes de confianza de la aplicación. Aunque los grupos de seguridad de red se admiten en Azure Application Gateway, hay restricciones y requisitos que se deben cumplir para que Azure Application Gateway y NSG funcionen según lo esperado.

* [Creación de suscripciones adicionales de Azure](https://docs.microsoft.com/azure/billing/billing-create-subscription)

* [Creación de grupos de administración](https://docs.microsoft.com/azure/governance/management-groups/create)

* [Creación y uso de etiquetas](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-using-tags)

* [Descripción de las restricciones y los requisitos del uso de NSG con Azure Application Gateway](https://docs.microsoft.com/azure/application-gateway/configuration-overview)

* [Creación de un NSG con una configuración de seguridad](https://docs.microsoft.com/azure/virtual-network/tutorial-filter-network-traffic)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="43-monitor-and-block-unauthorized-transfer-of-sensitive-information"></a>4.3: Supervisión y bloqueo de una transferencia no autorizada de información confidencial

**Guía**: asegúrese de que todas las implementaciones de subred de Azure Application Gateway de Virtual Network tienen un grupo de seguridad de red (NSG) aplicado con controles de acceso a la red específicos para los puertos y orígenes de confianza de la aplicación. Restrinja el tráfico de salida a solo ubicaciones de confianza para ayudar a mitigar la amenaza de la exfiltración de datos. Aunque los grupos de seguridad de red se admiten en Azure Application Gateway, hay restricciones y requisitos que se deben cumplir para que Azure Application Gateway y NSG funcionen según lo esperado.

* [Descripción de las restricciones y los requisitos del uso de NSG con Azure Application Gateway](https://docs.microsoft.com/azure/application-gateway/configuration-overview)

* [Creación de un NSG con una configuración de seguridad](https://docs.microsoft.com/azure/virtual-network/tutorial-filter-network-traffic)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Compartido

### <a name="44-encrypt-all-sensitive-information-in-transit"></a>4.4: Cifrado de toda la información confidencial en tránsito

**Guía**: configure el cifrado de un extremo a otro con TLS para Azure Application Gateways.

* [Configuración de TLS de un extremo a otro con Azure Application Gateway](https://docs.microsoft.com/azure/application-gateway/end-to-end-ssl-portal)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Compartido

### <a name="45-use-an-active-discovery-tool-to-identify-sensitive-data"></a>4.5: Uso de una herramienta de detección activa para identificar datos confidenciales

**Guía**: no es aplicable, Azure Application Gateway no almacena datos en reposo de clientes.

Microsoft administra la infraestructura subyacente para Azure Application Gateway y ha implementado controles estrictos para evitar la pérdida o exposición de los datos de los clientes.

* [Descripción de la protección de datos de los clientes en Azure](https://docs.microsoft.com/azure/security/fundamentals/protection-customer-data)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="46-use-azure-rbac-to-control-access-to-resources"></a>4.6: Uso de RBAC de Azure para controlar el acceso a los recursos

**Guía**: use el control de acceso basado en roles de Azure (Azure RBAC) para controlar el acceso al plano de control de Azure Application Gateway (Azure Portal).

* [Configuración de Azure RBAC](https://docs.microsoft.com/azure/role-based-access-control/role-assignments-portal)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="47-use-host-based-data-loss-prevention-to-enforce-access-control"></a>4.7: Uso de la prevención de pérdida de datos basada en host para aplicar el control de acceso

**Guía**: no aplicable; esta recomendación está diseñada para recursos de proceso de IaaS.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="48-encrypt-sensitive-information-at-rest"></a>4.8: Cifrado de información confidencial en reposo

**Guía**: no es aplicable, Azure Application Gateway no almacena datos de clientes.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="49-log-and-alert-on-changes-to-critical-azure-resources"></a>4.9: Registro y alerta de cambios en los recursos críticos de Azure

**Guía**: use Azure Monitor con el registro de actividad de Azure para crear alertas cuando se produzcan cambios en las aplicaciones de producción de Azure Application Gateway y en otros recursos críticos o relacionados.

* [Creación de alertas para los eventos del registro de actividad de Azure](https://docs.microsoft.com/azure/azure-monitor/platform/alerts-activity-log)

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

## <a name="vulnerability-management"></a>Administración de vulnerabilidades

*Para más información, consulte [Control de seguridad: Administración de vulnerabilidades](https://docs.microsoft.com/azure/security/benchmarks/security-control-vulnerability-management).*

### <a name="51-run-automated-vulnerability-scanning-tools"></a>5.1: Ejecute herramientas de análisis de vulnerabilidades automatizado

**Guía**: actualmente no está disponible; la evaluación de vulnerabilidades en Azure Security Center todavía no se admite para Azure Application Gateway.

Plataforma subyacente examinada y revisada por Microsoft. Revise los controles de seguridad disponibles para Azure Application Gateway para reducir las vulnerabilidades relacionadas con la configuración del servicio.

* [Cobertura de características (incluida la evaluación de vulnerabilidades) para los servicios PaaS de Azure](https://docs.microsoft.com/azure/security-center/features-paas)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="52-deploy-automated-operating-system-patch-management-solution"></a>5.2: Implemente una solución de administración de revisiones de sistema operativo automatizada

**Guía**: no aplicable; esta recomendación está diseñada para recursos de proceso de IaaS.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="53-deploy-automated-patch-management-solution-for-third-party-software-titles"></a>5.3: Implementación de una solución de administración de revisiones automatizada de títulos de software de terceros

**Instrucciones**: no aplicable; esta recomendación está diseñada para recursos de proceso de IaaS.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="54-compare-back-to-back-vulnerability-scans"></a>5.4: Compare los exámenes de vulnerabilidades opuestos

**Guía**: todavía no está disponible; la evaluación de vulnerabilidades en Azure Security Center todavía no se admite para Azure Application Gateway.

Plataforma subyacente examinada y revisada por Microsoft. Revise los controles de seguridad disponibles para Azure Application Gateway para reducir las vulnerabilidades relacionadas con la configuración del servicio.

* [Cobertura de características (incluida la evaluación de vulnerabilidades) para los servicios PaaS de Azure](https://docs.microsoft.com/azure/security-center/features-paas)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="55-use-a-risk-rating-process-to-prioritize-the-remediation-of-discovered-vulnerabilities"></a>5.5: Use un proceso de clasificación de riesgos para priorizar la corrección de las vulnerabilidades detectadas

**Guía**: todavía no está disponible; la evaluación de vulnerabilidades en Azure Security Center todavía no se admite para Azure Application Gateway.

Plataforma subyacente examinada y revisada por Microsoft. Revise los controles de seguridad disponibles para Azure Application Gateway para reducir las vulnerabilidades relacionadas con la configuración del servicio.

* [Cobertura de características (incluida la evaluación de vulnerabilidades) para los servicios PaaS de Azure](https://docs.microsoft.com/azure/security-center/features-paas)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

## <a name="inventory-and-asset-management"></a>Inventario y administración de recursos

*Para más información, consulte [Control de seguridad: Administración de recursos y del inventario](https://docs.microsoft.com/azure/security/benchmarks/security-control-inventory-asset-management).*

### <a name="61-use-automated-asset-discovery-solution"></a>6.1: Uso de la solución de detección de recursos automatizada

**Guía**: Use Azure Resource Graph para consultar o detectar todos los recursos (por ejemplo, proceso, almacenamiento, red, puertos y protocolos, etc.) dentro de las suscripciones. Asegúrese de que tiene los permisos adecuados (lectura) en el inquilino y enumere todas las suscripciones de Azure, así como los recursos de las suscripciones.

Aunque los recursos clásicos de Azure se pueden detectar a través de Resource Graph, se recomienda encarecidamente crear y usar los recursos de Azure Resource Manager que figuran a continuación.

* [Creación de consultas con Azure Resource Graph](https://docs.microsoft.com/azure/governance/resource-graph/first-query-portal)

* [Visualización de las suscripciones de Azure](https://docs.microsoft.com/powershell/module/az.accounts/get-azsubscription?view=azps-3.0.0)

* [Descripción de Azure RBAC](https://docs.microsoft.com/azure/role-based-access-control/overview)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="62-maintain-asset-metadata"></a>6.2: Mantenimiento de metadatos de recursos

**Instrucciones**: Aplique etiquetas a los recursos de Azure que proporcionan metadatos para organizarlos de forma lógica en una taxonomía.

* [Creación y uso de etiquetas](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-using-tags)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="63-delete-unauthorized-azure-resources"></a>6.3: Eliminación de recursos de Azure no autorizados

**Guía**: use el etiquetado, los grupos de administración y las suscripciones independientes, si procede, para organizar y realizar un seguimiento de los recursos de Azure. Concilie el inventario periódicamente y asegúrese de que los recursos no autorizados se eliminan de la suscripción de manera oportuna.

Además, use Azure Policy para establecer restricciones sobre el tipo de recursos que se pueden crear en las suscripciones del cliente con las siguientes definiciones de directiva integradas:
- Tipos de recursos no permitidos
- Tipos de recursos permitidos

* [Creación de suscripciones adicionales de Azure](https://docs.microsoft.com/azure/billing/billing-create-subscription)

* [Creación de grupos de administración](https://docs.microsoft.com/azure/governance/management-groups/create)

* [Creación y uso de etiquetas](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-using-tags)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="64-define-and-maintain-an-inventory-of-approved-azure-resources"></a>6.4: Definición y mantenimiento de un inventario de los recursos de Azure aprobados

**Guía**: defina los recursos de Azure aprobados.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="65-monitor-for-unapproved-azure-resources"></a>6.5: Supervisión de recursos de Azure no aprobados

**Guía**: Use Azure Policy para establecer restricciones en el tipo de recursos que se pueden crear en sus suscripciones.

Use Azure Resource Graph para consultar o detectar recursos dentro de sus suscripciones. Asegúrese de que todos los recursos de Azure presentes en el entorno estén aprobados.

* [Configuración y administración de Azure Policy](https://docs.microsoft.com/azure/governance/policy/tutorials/create-and-manage)

* [Creación de consultas con Azure Graph](https://docs.microsoft.com/azure/governance/resource-graph/first-query-portal)

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

**Instrucciones**: use Azure Policy para establecer restricciones sobre el tipo de recursos que se pueden crear en las suscripciones del cliente con las siguientes definiciones de directiva integradas:
- Tipos de recursos no permitidos
- Tipos de recursos permitidos

* [Configuración y administración de Azure Policy](https://docs.microsoft.com/azure/governance/policy/tutorials/create-and-manage)

* [Denegación de un tipo de recurso específico con Azure Policy](https://docs.microsoft.com/azure/governance/policy/samples/not-allowed-resource-types)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="610-maintain-an-inventory-of-approved-software-titles"></a>6.10: Mantenimiento de un inventario de títulos de software aprobados

**Guía**: no aplicable; esta recomendación está diseñada para recursos de proceso de IaaS.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="611-limit-users-ability-to-interact-with-azure-resource-manager"></a>6.11: Limitación de la capacidad de los usuarios para interactuar con Azure Resource Manager

**Instrucciones**: Configure el acceso condicional de Azure para limitar la capacidad de los usuarios de interactuar con Azure Resource Manager configurando "Bloquear acceso" en la aplicación Microsoft Azure Management.

* [Configuración del acceso condicional para bloquear el acceso a Azure Resource Manager](https://docs.microsoft.com/azure/role-based-access-control/conditional-access-azure-management)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="612-limit-users-ability-to-execute-scripts-within-compute-resources"></a>6.12: Limitación de capacidad de los usuarios para ejecutar scripts en recursos de proceso

**Guía**: no aplicable; esta recomendación está diseñada para recursos de proceso de IaaS.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="613-physically-or-logically-segregate-high-risk-applications"></a>6.13: Segregación física o lógica de aplicaciones de alto riesgo

**Guía**: Implemente suscripciones y/o grupos de administración independientes para los entornos de desarrollo, prueba y producción. Asegúrese de que todas las implementaciones de subred de Azure Application Gateway de Virtual Network tienen un grupo de seguridad de red (NSG) aplicado con controles de acceso a la red específicos para los puertos y orígenes de confianza de la aplicación. Aunque los grupos de seguridad de red se admiten en Azure Application Gateway, hay restricciones y requisitos que se deben cumplir para que Azure Application Gateway y NSG funcionen según lo esperado.

* [Creación de suscripciones adicionales de Azure](https://docs.microsoft.com/azure/billing/billing-create-subscription)

* [Creación de grupos de administración](https://docs.microsoft.com/azure/governance/management-groups/create)

* [Creación y uso de etiquetas](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-using-tags)

* [Descripción de las restricciones y los requisitos del uso de NSG con Azure Application Gateway](https://docs.microsoft.com/azure/application-gateway/configuration-overview)

* [Creación de un NSG con una configuración de seguridad](https://docs.microsoft.com/azure/virtual-network/tutorial-filter-network-traffic)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

## <a name="secure-configuration"></a>Configuración segura

*Para más información, consulte [Control de seguridad: Configuración segura](https://docs.microsoft.com/azure/security/benchmarks/security-control-secure-configuration).*

### <a name="71-establish-secure-configurations-for-all-azure-resources"></a>7.1: Establezca configuraciones seguras para todos los recursos de Azure

**Guía**: defina e implemente configuraciones de seguridad estándar para la configuración de red relacionada con las implementaciones de Azure Application Gateway. Use alias de Azure Policy en el espacio de nombres "Microsoft.Network" para crear directivas personalizadas que permitan auditar o aplicar la configuración de red de los grupos de seguridad de red, Azure Application Gateways y Azure Virtual Networks. También puede usar definiciones de directivas integradas.

* [Visualización de los alias de Azure Policy disponibles](https://docs.microsoft.com/powershell/module/az.resources/get-azpolicyalias?view=azps-3.3.0)

* [Configuración y administración de Azure Policy](https://docs.microsoft.com/azure/governance/policy/tutorials/create-and-manage)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="72-establish-secure-operating-system-configurations"></a>7.2: Establezca configuraciones del sistema operativo seguras

**Guía**: no aplicable; esta recomendación está diseñada para recursos de proceso de IaaS.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="73-maintain-secure-azure-resource-configurations"></a>7.3: Mantenga configuraciones de recursos de Azure seguras

**Instrucciones**: Use la directiva de Azure [denegar] e [implementar si no existe] para aplicar la configuración segura en los recursos de Azure.

* [Configuración y administración de Azure Policy](https://docs.microsoft.com/azure/governance/policy/tutorials/create-and-manage)

* [Descripción de los efectos de Azure Policy](https://docs.microsoft.com/azure/governance/policy/concepts/effects)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="74-maintain-secure-operating-system-configurations"></a>7.4: Mantenga configuraciones del sistema operativo seguras

**Guía**: no aplicable; esta recomendación está diseñada para recursos de proceso de IaaS.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="75-securely-store-configuration-of-azure-resources"></a>7.5: Almacene de forma segura la configuración de los recursos de Azure

**Instrucciones**: Si usa definiciones de directivas personalizadas de Azure, use Azure DevOps o Azure Repos para almacenar y administrar el código de forma segura.

* [Almacenamiento de código en Azure DevOps](https://docs.microsoft.com/azure/devops/repos/git/gitworkflow?view=azure-devops)

* [Documentación de Azure Repos](https://docs.microsoft.com/azure/devops/repos/index?view=azure-devops)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="76-securely-store-custom-operating-system-images"></a>7.6: Almacene imágenes de sistema operativo personalizadas de forma segura

**Guía**: no aplicable; esta recomendación está diseñada para recursos de proceso de IaaS.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="77-deploy-configuration-management-tools-for-azure-resources"></a>7.7: Implementación de herramientas de administración de configuración para recursos de Azure

**Guía**: use las definiciones integradas en Azure Policy junto con los alias de esta misma aplicación en el espacio de nombres "Microsoft.Network" para crear directivas personalizadas que permitan auditar y aplicar las configuraciones, así como enviar alertas sobre ellas. Además, desarrolle un proceso y una canalización para administrar las excepciones de las directivas.

* [Configuración y administración de Azure Policy](https://docs.microsoft.com/azure/governance/policy/tutorials/create-and-manage)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="78-deploy-configuration-management-tools-for-operating-systems"></a>7.8: Implementación de herramientas de administración de la configuración para sistemas operativos

**Guía**: no aplicable; esta recomendación está diseñada para recursos de proceso de IaaS.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="79-implement-automated-configuration-monitoring-for-azure-resources"></a>7.9: Implementación de la supervisión de la configuración automatizada para los recursos de Azure

**Guía**: use las definiciones integradas en Azure Policy junto con los alias de esta misma aplicación en el espacio de nombres "Microsoft.Network" para crear directivas personalizadas que permitan auditar y aplicar las configuraciones, así como enviar alertas sobre ellas. Use la directiva de Azure [audit], [deny] y [deploy if not exist] para aplicar automáticamente las configuraciones en los recursos de Azure.

* [Configuración y administración de Azure Policy](https://docs.microsoft.com/azure/governance/policy/tutorials/create-and-manage)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="710-implement-automated-configuration-monitoring-for-operating-systems"></a>7.10: Implemente la supervisión de configuración automatizada para sistemas operativos

**Instrucciones**: no aplicable; esta recomendación está diseñada para recursos de proceso de IaaS.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="711-manage-azure-secrets-securely"></a>7.11: Administre los secretos de Azure de forma segura

**Guía**: use las identidades administradas para proporcionar a Azure Application Gateway una identidad administrada automáticamente en Azure Active Directory (AD). Las identidades administradas le permiten autenticarse en cualquier servicio que admita la autenticación de Azure AD, incluyendo Key Vault, sin necesidad de credenciales en el código.

Use Azure Key Vault para almacenar certificados de forma segura. Azure Key Vault es un almacén de secretos administrado por la plataforma que puede usar para proteger los secretos, las claves y los certificados SSL. Azure Application Gateway admite la integración con Key Vault para certificados de servidor adjuntos a clientes de escucha con HTTPS habilitado. Esta compatibilidad solo es aplicable a la versión 2 de la SKU de Application Gateway.

* [Configuración de la terminación SSL con certificados de Key Vault mediante Azure PowerShell](https://docs.microsoft.com/azure/application-gateway/configure-keyvault-ps)

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="712-manage-identities-securely-and-automatically"></a>7.12: Administre las identidades de forma segura y automática

**Guía**: use las identidades administradas para proporcionar a Azure Application Gateway una identidad administrada automáticamente en Azure Active Directory (AD). Las identidades administradas le permiten autenticarse en cualquier servicio que admita la autenticación de Azure AD, incluyendo Key Vault, sin necesidad de credenciales en el código.

Use Azure Key Vault para almacenar certificados de forma segura. Azure Key Vault es un almacén de secretos administrado por la plataforma que puede usar para proteger los secretos, las claves y los certificados SSL. Azure Application Gateway admite la integración con Key Vault para certificados de servidor adjuntos a clientes de escucha con HTTPS habilitado. Esta compatibilidad solo es aplicable a la versión 2 de la SKU de Application Gateway.

* [Configuración de la terminación SSL con certificados de Key Vault mediante Azure PowerShell](https://docs.microsoft.com/azure/application-gateway/configure-keyvault-ps)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="713-eliminate-unintended-credential-exposure"></a>7.13: Elimine la exposición de credenciales no intencionada

**Instrucciones**: Implemente el escáner de credenciales para identificar las credenciales en el código. El escáner de credenciales también fomenta el traslado de credenciales detectadas a ubicaciones más seguras, como Azure Key Vault.

* [Configuración del escáner de credenciales](https://secdevtools.azurewebsites.net/helpcredscan.html)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

## <a name="malware-defense"></a>Defensa contra malware

*Para más información, consulte [Control de seguridad: defensa contra malware](https://docs.microsoft.com/azure/security/benchmarks/security-control-malware-defense).*

### <a name="81-use-centrally-managed-anti-malware-software"></a>8.1: Use software antimalware administrado centralmente

**Guía**: implemente el firewall de aplicaciones web (WAF) v2 de Azure delante de las aplicaciones web críticas para la inspección adicional del tráfico entrante. El firewall de aplicaciones web (WAF) es un servicio (característica de Azure Application Gateway) que proporciona a las aplicaciones web una protección centralizada contra vulnerabilidades de seguridad comunes. El firewall de aplicaciones web de Azure puede ayudarle a proteger las aplicaciones web de Azure App Service al inspeccionar el tráfico web entrante a fin de bloquear ataques como inyecciones de código SQL, scripts de sitios, cargas de malware y ataques de denegación de servicio distribuido.

Configure los valores de diagnóstico para las implementaciones de Azure Application Gateway. La configuración de diagnóstico se usa para configurar la exportación de streaming de registros y métricas de la plataforma para un recurso al destino que elija (Storage Accounts, Event Hubs y Log Analytics).

* [Implementación de WAF de Azure](https://docs.microsoft.com/azure/web-application-firewall/ag/create-waf-policy-ag)

* [Configuración de los valores de diagnóstico para Azure WAF](https://docs.microsoft.com/azure/application-gateway/application-gateway-diagnostics)

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="82-pre-scan-files-to-be-uploaded-to-non-compute-azure-resources"></a>8.2: Examine previamente los archivos que se van a cargar en recursos de Azure que no son de proceso

**Guía**: no es aplicable, Azure Application Gateway no almacena datos de clientes.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="83-ensure-anti-malware-software-and-signatures-are-updated"></a>8.3: Asegúrese de que se han actualizado el software y las firmas antimalware

**Guía**: al usar el firewall de aplicaciones web (WAF) de Azure, puede configurar las directivas de WAF. Una directiva de WAF consta de dos tipos de reglas de seguridad: reglas personalizadas creadas por el cliente y conjuntos de reglas administrados, una colección de conjuntos de reglas configurados previamente y administrados por Azure. Los conjuntos de reglas administrados por Azure proporcionan una forma fácil de implementar la protección frente a un conjunto común de amenazas de seguridad. Dado que Azure administra estos conjuntos de reglas, las reglas se actualizan según sea necesario para protegerse frente a nuevas firmas de ataque.

* [Descripción de los conjuntos de reglas de WAF administrados por Azure](https://docs.microsoft.com/azure/web-application-firewall/ag/ag-overview#waf-policy)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Compartido

## <a name="data-recovery"></a>Recuperación de datos

*Para más información, consulte [Control de seguridad: recuperación de datos](https://docs.microsoft.com/azure/security/benchmarks/security-control-data-recovery).*

### <a name="91-ensure-regular-automated-back-ups"></a>9.1: Garantía de copias de seguridad automáticas periódicas

**Guía**: Azure Application Gateway no almacena datos de clientes. Sin embargo, si usa definiciones de directivas personalizadas de Azure, use Azure DevOps o Azure Repos para almacenar y administrar el código de forma segura.

Azure DevOps Services utiliza muchas de las características de almacenamiento de Azure para garantizar la disponibilidad de los datos en caso de un error de hardware, una interrupción del servicio o un desastre en la región. Además, el equipo de Azure DevOps sigue los procedimientos para proteger los datos contra la eliminación accidental o malintencionada.

* [Descripción de la disponibilidad de los datos en Azure DevOps](https://docs.microsoft.com/azure/devops/organizations/security/data-protection?view=azure-devops#data-availability)

* [Almacenamiento de código en Azure DevOps](https://docs.microsoft.com/azure/devops/repos/git/gitworkflow?view=azure-devops)

* [Documentación de Azure Repos](https://docs.microsoft.com/azure/devops/repos/index?view=azure-devops)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="92-perform-complete-system-backups-and-backup-any-customer-managed-keys"></a>9.2: Realización de copias de seguridad completas del sistema y copia de seguridad de las claves administradas por el cliente

**Guía**: realice una copia de seguridad de los certificados administrados por el cliente en Azure Key Vault.

* [Realización de copias de seguridad de certificados del almacén de claves en Azure](https://docs.microsoft.com/powershell/module/azurerm.keyvault/backup-azurekeyvaultcertificate)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="93-validate-all-backups-including-customer-managed-keys"></a>9.3: Validación de todas las copias de seguridad, incluidas las claves administradas por el cliente

**Guía**: pruebe la restauración de la copia de seguridad de los certificados administrados por el cliente.

* [Restauración de certificados del almacén de claves](https://docs.microsoft.com/powershell/module/azurerm.keyvault/restore-azurekeyvaultcertificate)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="94-ensure-protection-of-backups-and-customer-managed-keys"></a>9.4: Garantía de la protección de las copias de seguridad y las claves administradas del cliente

**Instrucciones**: Asegúrese de que la eliminación temporal está habilitada para Azure Key Vault. La eliminación temporal permite la recuperación de almacenes de claves y objetos de almacén eliminados, como claves, secretos y certificados.

* [Uso de la eliminación temporal de Azure Key Vault](https://docs.microsoft.com/azure/key-vault/key-vault-soft-delete-powershell)

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

## <a name="incident-response"></a>Respuesta a los incidentes

*Para más información, consulte [Control de seguridad: Respuesta a los incidentes](https://docs.microsoft.com/azure/security/benchmarks/security-control-incident-response).*

### <a name="101-create-an-incident-response-guide"></a>10.1: Creación de una guía de respuesta ante incidentes

**Instrucciones**: Cree una guía de respuesta a incidentes para su organización. Asegúrese de que haya planes de respuesta a incidentes escritos que definan todos los roles del personal, así como las fases de administración y gestión de los incidentes, desde la detección hasta la revisión posterior a la incidencia.

* [Configuración de las automatizaciones de flujos de trabajo en Azure Security Center](https://docs.microsoft.com/azure/security-center/security-center-planning-and-operations-guide)

* [Guía para crear su propio proceso de respuesta a incidentes de seguridad](https://msrc-blog.microsoft.com/2019/07/01/inside-the-msrc-building-your-own-security-incident-response-process/)

* [Anatomía de un incidente del Centro de respuestas de seguridad de Microsoft](https://msrc-blog.microsoft.com/2019/07/01/inside-the-msrc-building-your-own-security-incident-response-process/)

* [El cliente también puede usar la guía de control de incidentes de seguridad de equipos de NIST como referencia para la creación de su propio plan de respuesta a incidentes](https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-61r2.pdf)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="102-create-an-incident-scoring-and-prioritization-procedure"></a>10.2: Creación de un procedimiento de priorización y puntuación de incidentes

**Instrucciones**: Security Center asigna una gravedad a cada alerta para ayudarle a priorizar aquellas que se deben investigar en primer lugar. La gravedad se basa en la confianza que tiene Security Center en la búsqueda o en el análisis utilizados para emitir la alerta, así como en el nivel de confianza de que ha habido un intento malintencionado detrás de la actividad que ha provocado la alerta.

Además, marque claramente las suscripciones (por ejemplo, producción, no producción) y cree un sistema de nomenclatura para identificar y clasificar claramente los recursos de Azure.

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="103-test-security-response-procedures"></a>10.3: Prueba de los procedimientos de respuesta de seguridad

**Instrucciones**: Realice ejercicios para probar las capacidades de respuesta a los incidentes de los sistemas con regularidad. Identifique puntos débiles y brechas y revise el plan según sea necesario.

* [Consulte la publicación de NIST: Guía para probar, entrenar y ejecutar programas para planes y funcionalidades de TI](https://nvlpubs.nist.gov/nistpubs/Legacy/SP/nistspecialpublication800-84.pdf)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="104-provide-security-incident-contact-details-and-configure-alert-notifications-for-security-incidents"></a>10.4: Provisión de detalles de contacto de incidentes de seguridad y configuración de notificaciones de alerta para incidentes de seguridad

**Guía**: La información de contacto del incidente de seguridad la utilizará Microsoft para ponerse en contacto con usted si Microsoft Security Response Center (MSRC) detecta que un tercero no autorizado o ilegal ha accedido a los datos del cliente. Revise los incidentes después del hecho para asegurarse de que se resuelven los problemas.

* [Establecimiento del contacto de seguridad de Azure Security Center](https://docs.microsoft.com/azure/security-center/security-center-provide-security-contact-details)

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="105-incorporate-security-alerts-into-your-incident-response-system"></a>10.5: Incorporación de alertas de seguridad en el sistema de respuesta a incidentes

**Instrucciones**: Exporte sus alertas y recomendaciones de Azure Security Center mediante la característica de exportación continua. La exportación continua le permite exportar alertas y recomendaciones de forma manual o continua. Puede usar el conector de datos de Azure Security Center para transmitir las alertas a Sentinel.

* [Configuración de la exportación continua](https://docs.microsoft.com/azure/security-center/continuous-export)

* [Transmisión de alertas a Azure Sentinel](https://docs.microsoft.com/azure/sentinel/connect-azure-security-center)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="106-automate-the-response-to-security-alerts"></a>10.6: Automatización de la respuesta a las alertas de seguridad

**Instrucciones**: use la característica de automatización del flujo de trabajo de Azure Security Center para desencadenar automáticamente respuestas mediante "Logic Apps" en las alertas y recomendaciones de seguridad.

* [Configuración de la automatización de flujo de trabajo y Logic Apps](https://docs.microsoft.com/azure/security-center/workflow-automation)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

## <a name="penetration-tests-and-red-team-exercises"></a>Pruebas de penetración y ejercicios del equipo rojo

*Para más información, consulte [Control de seguridad: Pruebas de penetración y ejercicios del equipo rojo](https://docs.microsoft.com/azure/security/benchmarks/security-control-penetration-tests-red-team-exercises).*

### <a name="111-conduct-regular-penetration-testing-of-your-azure-resources-and-ensure-remediation-of-all-critical-security-findings"></a>11.1: Realice pruebas de penetración periódicas de los recursos de Azure y asegúrese de corregir todos los resultados de seguridad críticos

**Guía**: 

* [Siga las reglas de involucración de Microsoft para asegurarse de que las pruebas de penetración no infrinjan las directivas de Microsoft](https://www.microsoft.com/msrc/pentest-rules-of-engagement?rtc=1)

* [Puede encontrar más información sobre la estrategia y ejecución de Microsoft en las pruebas de penetración del equipo rojo y los sitios activos en la infraestructura de nube, los servicios y las aplicaciones administradas por Microsoft aquí.](https://gallery.technet.microsoft.com/Cloud-Red-Teaming-b837392e)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Compartido

## <a name="next-steps"></a>Pasos siguientes

- Consulte la [prueba comparativa de seguridad de Azure](https://docs.microsoft.com/azure/security/benchmarks/overview).
- Obtenga más información sobre las [líneas de base de seguridad de Azure](https://docs.microsoft.com/azure/security/benchmarks/security-baselines-overview).
