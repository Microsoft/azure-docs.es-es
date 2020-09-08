---
title: Línea de base de seguridad de Azure para Service Fabric
description: La línea base de seguridad de Service Fabric proporciona instrucciones de procedimientos y recursos para implementar las recomendaciones de seguridad especificadas en Azure Security Benchmark.
author: msmbaldwin
ms.service: service-fabric
ms.topic: conceptual
ms.date: 07/22/2020
ms.author: mbaldwin
ms.custom: security-benchmark
ms.openlocfilehash: a3641994098834b47412598b25b3effb2be7d276
ms.sourcegitcommit: 2ff0d073607bc746ffc638a84bb026d1705e543e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87836724"
---
# <a name="azure-security-baseline-for-service-fabric"></a>Línea de base de seguridad de Azure para Service Fabric

La línea base de seguridad de Azure para Service Fabric contiene recomendaciones que le ayudarán a mejorar la posición de seguridad de la implementación.

La base de referencia de este servicio se extrae de la [versión 1.0 de Azure Security Benchmark](../security/benchmarks/overview.md), que proporciona recomendaciones sobre cómo puede proteger las soluciones en la nube en Azure con nuestra guía de procedimientos recomendados.

Para obtener más información, consulte [Introducción a las líneas de base de seguridad de Azure](../security/benchmarks/security-baselines-overview.md).



## <a name="network-security"></a>Seguridad de las redes

*Para más información, consulte [Control de seguridad: Seguridad de redes](../security/benchmarks/security-control-network-security.md).*

### <a name="11-protect-azure-resources-within-virtual-networks"></a>1.1: Protección de los recursos de Azure dentro de las redes virtuales

**Instrucciones**: Asegúrese de que todas las implementaciones de subred de Virtual Network tienen un grupo de seguridad de red aplicado con controles de acceso a la red específicos para los puertos y orígenes de confianza de su aplicación.

* [Implementación de Azure Firewall mediante una plantilla](../firewall/deploy-template.md)

* [Creación de redes perimetrales mediante grupos de seguridad de red de Azure (NSG)](../security/fundamentals/service-fabric-best-practices.md#use-network-isolation-and-security-with-azure-service-fabric)

* [Integración del clúster de Azure Service Fabric con una red virtual existente](./service-fabric-patterns-networking.md)

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="12-monitor-and-log-the-configuration-and-traffic-of-virtual-networks-subnets-and-nics"></a>1.2: Supervisión y registro de la configuración y el tráfico de redes virtuales, subredes y NIC

**Guía**: use Azure Security Center y siga las recomendaciones de protección para la red virtual, la subred y el grupo de seguridad de red que se van a usar para proteger el clúster de Service Fabric. Habilite los registros de flujo del grupo de seguridad de red (NSG) y envíelos a una cuenta de Azure Storage para auditar el tráfico. También puede enviar registros de flujo del grupo de seguridad de red a un área de trabajo de Azure Log Analytics y usar Análisis de tráfico de Azure para proporcionar información detallada sobre el flujo de tráfico en la nube de Azure. Algunas de las ventajas de Análisis de tráfico de Azure son la capacidad de visualizar la actividad de la red e identificar las zonas activas, identificar las amenazas de seguridad, comprender los patrones de flujo de tráfico y detectar configuraciones de red incorrectas.

* [Habilitación de los registros de flujo de NSG](../network-watcher/network-watcher-nsg-flow-logging-portal.md)

* [Habilitación y uso de Análisis de tráfico de Azure](../network-watcher/traffic-analytics.md)

* [Descripción de la seguridad de red proporcionada por Azure Security Center](../security-center/security-center-network-recommendations.md)

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="13-protect-critical-web-applications"></a>1.3: Proteja las aplicaciones web críticas

**Guía**: proporcione una puerta de enlace front-end para ofrecer un único punto de entrada para usuarios, dispositivos u otras aplicaciones. Azure API Management se integra directamente con Service Fabric, lo que le permite proteger el acceso a los servicios de back-end, evitar ataques de DOS mediante la limitación y comprobar las claves de API, los tokens JWT, los certificados y otras credenciales.

Considere la posibilidad de implementar el firewall de aplicaciones web (WAF) de Azure delante de las aplicaciones web críticas para la inspección adicional del tráfico entrante. Habilite la configuración de diagnóstico para WAF e ingiera los registros en una cuenta de almacenamiento, un centro de eventos o un área de trabajo de análisis de registros.

* [Introducción a Service Fabric con Azure API Management](./service-fabric-api-management-overview.md)

* [Integración de API Management en una red virtual interna con Application Gateway](../api-management/api-management-howto-integrate-internal-vnet-appgateway.md)

* [Implementación de WAF de Azure](../web-application-firewall/ag/create-waf-policy-ag.md)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="14-deny-communications-with-known-malicious-ip-addresses"></a>1.4: Deniegue las comunicaciones con direcciones IP malintencionadas conocidas

**Instrucciones**: para protegerse de los ataques DDoS, habilite la protección estándar de DDoS de Azure en la red virtual donde está implementada su instancia de Azure Service Fabric. Use la inteligencia sobre amenazas integrada de Azure Security Center para denegar las comunicaciones con direcciones IP malintencionadas conocidas o no utilizadas.

* [Configuración de la protección contra DDoS](../virtual-network/manage-ddos-protection.md)

* [Descripción de la inteligencia sobre amenazas integrada de Azure Security Center](../security-center/threat-protection.md)

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="15-record-network-packets"></a>1.5: Registro de los paquetes de red

**Guía**: habilite los registros de flujo del grupo de seguridad de red (NSG) conectado a la subred que se usa para proteger el clúster de Azure Service Fabric. Anote los registros de flujo del grupo de seguridad de red en una cuenta de Azure Storage para generar registros de flujo. Si es necesario para investigar actividades anómalas, habilite la captura de paquetes de Azure Network Watcher.

* [Habilitación de los registros de flujo de NSG](../network-watcher/network-watcher-nsg-flow-logging-portal.md)

* [Habilitación de Network Watcher](../network-watcher/network-watcher-create.md)

* [Uso de análisis de tráfico para visualizar los registros de flujo de NSG](../network-watcher/traffic-analytics.md)

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="16-deploy-network-based-intrusion-detectionintrusion-prevention-systems-idsips"></a>1.6: Implementación de sistemas de prevención de intrusiones y de detección de intrusiones (IDS/IPS) basados en la red.

**Guía**: Seleccione una oferta de Azure Marketplace que admita la funcionalidad de IDS/IPS con funcionalidades de inspección de carga. Si la detección y/o la prevención de intrusiones basadas en la inspección de carga no es un requisito, se puede usar Azure Firewall con la inteligencia sobre amenazas. El filtrado basado en inteligencia sobre amenazas de Azure Firewall puede alertar y denegar el tráfico desde y hacia los dominios y las direcciones IP malintencionados conocidos. La direcciones IP y los dominios proceden de la fuente Inteligencia sobre amenazas de Microsoft.

Implemente la solución de firewall que prefiera en cada uno de los límites de red de su organización para detectar y/o denegar el tráfico malintencionado.

* [Azure Marketplace](https://azuremarketplace.microsoft.com/marketplace/?term=Firewall)

* [Implementación de Azure Firewall](../firewall/tutorial-firewall-deploy-portal.md)

* [Configuración de alertas con Azure Firewall](../firewall/threat-intel.md)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="17-manage-traffic-to-web-applications"></a>1.7: Administre el tráfico a las aplicaciones web

**Instrucciones**: Implemente Application Gateway de Azure para las aplicaciones web con HTTPS/SSL habilitado para los certificados de confianza.

* [Implementación de Application Gateway](../application-gateway/quick-create-portal.md)

* [Configuración de Application Gateway para usar HTTPS](../application-gateway/create-ssl-portal.md)

* [Descripción del equilibrio de carga de capa 7 con las puertas de enlace de aplicaciones web de Azure](../application-gateway/overview.md)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="18-minimize-complexity-and-administrative-overhead-of-network-security-rules"></a>1.8: Minimice la complejidad y la sobrecarga administrativa de las reglas de seguridad de red

**Guía**: use etiquetas de servicio de red virtual para definir controles de acceso de red en los grupos de seguridad de red (NSG) que están conectados a la subred en la que está implementado el clúster de Azure Service Fabric. Puede utilizar etiquetas de servicio en lugar de direcciones IP específicas al crear reglas de seguridad. Al especificar el nombre de la etiqueta de servicio (por ejemplo, ApiManagement) en el campo de origen o destino apropiado de una regla, puede permitir o denegar el tráfico para el servicio correspondiente. Microsoft administra los prefijos de direcciones que la etiqueta de servicio incluye y actualiza automáticamente dicha etiqueta a medida que las direcciones cambian.

* [Etiquetas de servicio de red virtual](../virtual-network/service-tags-overview.md)

* [Procedimientos recomendados de red de Service Fabric](./service-fabric-best-practices-networking.md)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="19-maintain-standard-security-configurations-for-network-devices"></a>1.9: Mantenga las configuraciones de seguridad estándar para dispositivos de red

**Guía**: defina e implemente configuraciones de seguridad estándar para los recursos de red relacionados con el clúster de Azure Service Fabric. Use alias de Azure Policy en los espacios de nombres "Microsoft.ServiceFabric" y "Microsoft.Network" para crear directivas personalizadas con el fin de auditar o aplicar la configuración de red del clúster de Azure Service Fabric.

También puede usar Azure Blueprints para simplificar las implementaciones de Azure a gran escala mediante el empaquetado de artefactos de entorno clave, como plantillas de Azure Resource Manager, controles de RBAC y directivas, en una única definición de plano técnico. Aplique fácilmente el plano técnico a nuevas suscripciones y entornos, y ajuste el control y la administración mediante el control de versiones.

* [Visualización de los alias de Azure Policy disponibles](/powershell/module/az.resources/get-azpolicyalias?view=azps-3.3.0)

* [Configuración y administración de Azure Policy](../governance/policy/tutorials/create-and-manage.md)

* [Creación de un plano técnico de Azure](../governance/blueprints/create-blueprint-portal.md)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="110-document-traffic-configuration-rules"></a>1.10: Documente las reglas de configuración de tráfico

**Guía**: use etiquetas para el grupo de seguridad de red (NSG) y otros recursos relacionados con la seguridad de red y el flujo de tráfico que están asociados al clúster de Azure Service Fabric. En el caso de las reglas de NSG individuales, use el campo "Descripción" para especificar las necesidades empresariales o la duración (etc.) de las reglas que permiten que entre o salga el tráfico en una red.

Use cualquiera de las definiciones de Azure Policy integradas relacionadas con el etiquetado, como "Requerir etiqueta y su valor", para asegurarse de que todos los recursos se creen con etiquetas y para notificarle los recursos no etiquetados existentes.

Puede usar Azure PowerShell o la interfaz de la línea de comandos (CLI) de Azure para buscar los recursos o realizar acciones en ellos en función de sus etiquetas.

* [Creación y uso de etiquetas](../azure-resource-manager/management/tag-resources.md)

* [Creación de una red virtual](../virtual-network/quick-create-portal.md)

* [Creación de un grupo de seguridad de red con una configuración de seguridad](../virtual-network/tutorial-filter-network-traffic.md)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="111-use-automated-tools-to-monitor-network-resource-configurations-and-detect-changes"></a>1.11: Use herramientas automatizadas para supervisar las configuraciones de recursos de red y detectar cambios

**Guía**: use el registro de actividad de Azure para supervisar las configuraciones de los recursos de red y detectar cambios en estos relacionados con las implementaciones de Azure Service Fabric. Cree alertas en Azure Monitor que se desencadenarán cuando se produzcan cambios en los recursos de red críticos.

* [Visualización y recuperación de eventos del registro de actividad de Azure](../azure-monitor/platform/activity-log.md#view-the-activity-log)

* [Creación de alertas en Azure Monitor](../azure-monitor/platform/alerts-activity-log.md)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

## <a name="logging-and-monitoring"></a>Registro y supervisión

*Para más información, consulte [Control de seguridad: registro y supervisión](../security/benchmarks/security-control-logging-monitoring.md).*

### <a name="21-use-approved-time-synchronization-sources"></a>2.1: Uso de orígenes de sincronización de hora aprobados

**Guía**: Microsoft mantiene los orígenes de hora de los componentes de clústeres de Azure Service Fabric, así que puede actualizar la sincronización de hora en las implementaciones de proceso.

* [Configuración de la sincronización de hora de los recursos de proceso de Azure](../virtual-machines/windows/time-sync.md)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Microsoft

### <a name="22-configure-central-security-log-management"></a>2.2: Configuración de la administración central de registros de seguridad

**Instrucciones**: puede incorporar el clúster de Azure Service Fabric a Azure Monitor para agregar datos de seguridad generados por el clúster. Vea un ejemplo de problemas en el diagnóstico y sus soluciones con Service Fabric.

* [Configuración de la integración de los registros de Azure Monitor con Service Fabric](./service-fabric-diagnostics-oms-setup.md)

* [Configuración de los registros de Azure Monitor para supervisar los contenedores de Azure Service Fabric](./service-fabric-tutorial-monitoring-wincontainers.md)

* [Diagnóstico de escenarios de Service Fabric comunes](./service-fabric-diagnostics-common-scenarios.md)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="23-enable-audit-logging-for-azure-resources"></a>2.3: Habilitación del registro de auditoría para recursos de Azure

**Guía**: habilite Azure Monitor para el clúster de Service Fabric y diríjalo a un área de trabajo de Log Analytics. Esta acción registrará información y métricas del sistema operativo de interés para todos los nodos del clúster de Azure Service Fabric.

* [Configuración de la integración de los registros de Azure Monitor con Service Fabric](./service-fabric-diagnostics-oms-setup.md)

* [Configuración de los registros de Azure Monitor para supervisar los contenedores de Azure Service Fabric](./service-fabric-tutorial-monitoring-wincontainers.md)

* [Implementación del agente de Log Analytics en los nodos](./service-fabric-diagnostics-oms-agent.md)

* [Búsquedas de registros de Log Analytics](../azure-monitor/log-query/log-query-overview.md)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="24-collect-security-logs-from-operating-systems"></a>2.4: Recopilación de registros de seguridad de sistemas operativos

**Guía**: Incorpore el clúster de Azure Service Fabric a Azure Monitor. Asegúrese de que el área de trabajo de Log Analytics usada tenga el período de retención de registros establecido de acuerdo con la normativa de cumplimiento de la organización.

* [Configuración de la integración de los registros de Azure Monitor con Service Fabric](./service-fabric-diagnostics-oms-setup.md)

* [Configuración de los registros de Azure Monitor para supervisar los contenedores de Azure Service Fabric](./service-fabric-tutorial-monitoring-wincontainers.md)

* [Implementación del agente de Log Analytics en los nodos](./service-fabric-diagnostics-oms-agent.md)

* [Configuración del período de retención del área de trabajo de Log Analytics](../azure-monitor/platform/manage-cost-storage.md)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="25-configure-security-log-storage-retention"></a>2.5: Configuración de la retención del almacenamiento de registros de seguridad

**Instrucciones**: Incorpore el clúster de Azure Service Fabric a Azure Monitor. Asegúrese de que el área de trabajo de Log Analytics usada tenga el período de retención de registros establecido de acuerdo con la normativa de cumplimiento de la organización.

* [Configuración de la integración de los registros de Azure Monitor con Service Fabric](./service-fabric-diagnostics-oms-setup.md)

* [Configuración de los registros de Azure Monitor para supervisar los contenedores de Azure Service Fabric](./service-fabric-tutorial-monitoring-wincontainers.md)

* [Implementación del agente de Log Analytics en los nodos](./service-fabric-diagnostics-oms-agent.md)

* [Configuración del período de retención del área de trabajo de Log Analytics](../azure-monitor/platform/manage-cost-storage.md)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="26-monitor-and-review-logs"></a>2.6: Supervisión y revisión de registros

**Guía**: use las consultas del área de trabajo de Azure Log Analytics para consultar los registros de Azure Service Fabric.

* [Búsquedas de registros de Log Analytics](../azure-monitor/log-query/log-query-overview.md)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="27-enable-alerts-for-anomalous-activities"></a>2.7: Habilitación de alertas para actividades anómalas

**Guía**: use el área de trabajo de Azure Log Analytics para supervisar los registros de seguridad y eventos relacionados con el clúster de Azure Service Fabric y generar alertas sobre actividades anómalas en ellos.

* [Administración de alertas de seguridad en Azure Security Center](../security-center/security-center-managing-and-responding-alerts.md)

* [Alertas sobre datos de registro de Log Analytics](../azure-monitor/learn/tutorial-response.md)

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="28-centralize-anti-malware-logging"></a>2.8: Centralización del registro antimalware

**Guía**: De forma predeterminada, Windows Defender está instalado en Windows Server 2016. Consulte su documentación antimalware para obtener reglas de configuración si no utiliza Windows Defender. Windows Defender no es compatible con Linux.

* [Para más información, consulte Antivirus Windows Defender en Windows Server 2016](/windows/security/threat-protection/windows-defender-antivirus/windows-defender-antivirus-on-windows-server-2016).

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="29-enable-dns-query-logging"></a>2.9: Habilitación del registro de consultas DNS

**Guía**: Implemente una solución de terceros para el registro de DNS.

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="210-enable-command-line-audit-logging"></a>2.10: Habilitación del registro de auditoría de la línea de comandos

**Instrucciones**: configure manualmente el registro de la consola por nodo.

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

## <a name="identity-and-access-control"></a>Control de identidades y acceso

*Para más información, consulte [Control de seguridad: Identidad y control de acceso](../security/benchmarks/security-control-identity-access-control.md).*

### <a name="31-maintain-an-inventory-of-administrative-accounts"></a>3.1: Mantenga un inventario de cuentas administrativas

**Instrucciones**: mantenga el registro de la cuenta administrativa local que se crea durante el aprovisionamiento del clúster de Azure Service Fabric, así como de cualquier otra cuenta que cree. Además, si se usa la integración de Azure AD, Azure AD tiene roles integrados que se deben asignar explícitamente y, por tanto, se pueden consultar. Use el módulo de PowerShell de Azure AD para realizar consultas ad hoc a fin de detectar cuentas que son miembros de grupos administrativos.

Además, puede usar las recomendaciones de administración de identidades y acceso de Azure Security Center.

* [Obtención de un rol de directorio en Azure AD con PowerShell](/powershell/module/azuread/get-azureaddirectoryrole?view=azureadps-2.0)

* [Obtención de los miembros de un rol de directorio en Azure AD con PowerShell](/powershell/module/azuread/get-azureaddirectoryrolemember?view=azureadps-2.0)

* [Supervisión de identidad y acceso con Azure Security Center](../security-center/security-center-identity-access.md)

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="32-change-default-passwords-where-applicable"></a>3.2: Cambie las contraseñas predeterminadas cuando proceda

**Guía**: al aprovisionar un clúster, Azure le exige que cree contraseñas para el portal web. No hay contraseñas predeterminadas para cambiar; sin embargo, puede especificar otras contraseñas para el acceso al portal web.

* [Creación en Azure Portal](./service-fabric-cluster-creation-via-portal.md)

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="33-use-dedicated-administrative-accounts"></a>3.3: Use cuentas administrativas dedicadas

**Guía**: integre la autenticación del clúster de Azure Service Fabric con Azure Active Directory. Cree directivas y procedimientos en torno al uso de cuentas administrativas dedicadas.

Además, puede usar las recomendaciones de administración de identidades y acceso de Azure Security Center.

* [Configuración de la autenticación de cliente de Azure Active Directory](./service-fabric-tutorial-create-vnet-and-windows-cluster.md#set-up-azure-active-directory-client-authentication)

* [Supervisión de identidad y acceso con Azure Security Center](../security-center/security-center-identity-access.md)

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="34-use-single-sign-on-sso-with-azure-active-directory"></a>3.4: Use el inicio de sesión único (SSO) con Azure Active Directory

**Guía**: Siempre que sea posible, use el SSO de Azure Active Directory en lugar de configurar credenciales independientes individuales por servicio. Use las recomendaciones de administración de identidades y acceso de Azure Security Center.

* [Descripción del SSO con Azure AD](../active-directory/manage-apps/what-is-single-sign-on.md)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="35-use-multi-factor-authentication-for-all-azure-active-directory-based-access"></a>3.5: Use la autenticación multifactor para todo el acceso basado en Azure Active Directory

**Instrucciones**: Habilite la MFA de Azure AD y siga las recomendaciones de administración de identidades y acceso de Azure Security Center.

* [Procedimiento para habilitar la MFA en Azure](../active-directory/authentication/howto-mfa-getstarted.md)

* [Supervisión de la identidad y el acceso en Azure Security Center](../security-center/security-center-identity-access.md)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="36-use-dedicated-machines-privileged-access-workstations-for-all-administrative-tasks"></a>3.6: Use máquinas dedicadas (estaciones de trabajo de acceso con privilegios) para todas las tareas administrativas

**Guía**: use PAW (estaciones de trabajo de acceso con privilegios) con la autenticación multifactor (MFA) configurada para iniciar sesión en los clústeres de Azure Service Fabric y recursos relacionados, y configurarlos.

* [Más información sobre las estaciones de trabajo con privilegios de acceso](/windows-server/identity/securing-privileged-access/privileged-access-workstations)

* [Procedimiento para habilitar la MFA en Azure](../active-directory/authentication/howto-mfa-getstarted.md)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="37-log-and-alert-on-suspicious-activities-from-administrative-accounts"></a>3.7: Registro y alerta de actividades sospechosas desde cuentas administrativas

**Guía**: Use Privileged Identity Management (PIM) de Azure Active Directory (AD) para la generación de registros y alertas cuando se produzca una actividad sospechosa o insegura en el entorno. Además, use las detecciones de riesgo de Azure AD para ver alertas e informes sobre el comportamiento de los usuarios de riesgo.

* [Cómo implementar Privileged Identity Management (PIM)](../active-directory/privileged-identity-management/pim-deployment-plan.md)

* [Información sobre las detecciones de riesgo de Azure AD](../active-directory/identity-protection/overview-identity-protection.md)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="38-manage-azure-resources-from-only-approved-locations"></a>3.8: Administre los recursos de Azure solo desde ubicaciones aprobadas

**Guía**: Use ubicaciones con nombre de acceso condicional para permitir el acceso solo desde agrupaciones lógicas específicas de intervalos de direcciones IP o países o regiones.

* [Configuración de ubicaciones con nombre en Azure](../active-directory/reports-monitoring/quickstart-configure-named-locations.md)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="39-use-azure-active-directory"></a>3.9: Uso de Azure Active Directory

**Guía**: use Azure Active Directory (AAD) como sistema central de autenticación y autorización para proteger el acceso a los puntos de conexión de administración de los clústeres de Azure Service Fabric. AAD protege los datos mediante un fuerte cifrado para los datos en reposo y en tránsito. AAD también cifra con sal, convierte en hash y almacena de forma segura las credenciales de los usuarios.

* [Creación y configuración de una instancia de AAD](../active-directory/fundamentals/active-directory-access-create-new-tenant.md)

* [Uso de Azure Active Directory para la autenticación de clientes de Service Fabric](./service-fabric-cluster-creation-setup-aad.md)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="310-regularly-review-and-reconcile-user-access"></a>3.10: Revise y concilie regularmente el acceso de los usuarios

**Instrucciones**: use la autenticación de Azure Active Directory (AAD) con el clúster de Azure Service Fabric. AAD proporciona registros para ayudar a descubrir las cuentas obsoletas. Además, use las revisiones de acceso de identidad de Azure para administrar de forma eficiente las pertenencias a grupos, el acceso a las aplicaciones empresariales y las asignaciones de roles. El acceso de los usuarios se puede revisar de forma periódica para asegurarse de que solo las personas adecuadas tengan acceso continuado.

* [Procedimiento para usar las revisiones de acceso de identidad de Azure](../active-directory/governance/access-reviews-overview.md)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="311-alert-on-account-login-behavior-deviation"></a>3.11: Alerte de las desviaciones de comportamiento en los inicios de sesión de las cuentas

**Instrucciones**: use los registros de inicio de sesión y auditoría de Azure Active Directory (AAD) para supervisar los intentos de acceso a cuentas desactivadas; estos registros se pueden integrar en cualquier herramienta de SIEM o de supervisión de terceros.

Para simplificar este proceso, cree una configuración de diagnóstico para las cuentas de usuario de AAD y envíe los registros de auditoría e inicio de sesión a un área de trabajo de Azure Log Analytics. Configure las alertas deseadas en el área de trabajo de Azure Log Analytics.

* [Integración de los registros de actividad de Azure en Azure Monitor](../active-directory/reports-monitoring/howto-integrate-activity-logs-with-log-analytics.md)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="312-alert-on-account-sign-in-behavior-deviation"></a>3.12: Alerta sobre las desviaciones de comportamiento en los inicios de sesión de las cuentas

**Instrucciones**: Use las características de protección de riesgos e identidad de Azure AD para configurar las respuestas automatizadas a las acciones sospechosas detectadas relacionadas con las identidades de los usuarios. También puede hacer que Azure Sentinel ingiera los datos para investigarlos más.

* [Visualización de los inicios de sesión de riesgo de Azure AD](../active-directory/identity-protection/overview-identity-protection.md)

* [Configuración y habilitación de las directivas de riesgo de protección de identidad](../active-directory/identity-protection/howto-identity-protection-configure-risk-policies.md)

* [Incorporación de Azure Sentinel](../sentinel/quickstart-onboard.md)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="313-provide-microsoft-with-access-to-relevant-customer-data-during-support-scenarios"></a>3.13: Proporcione a Microsoft acceso a los datos pertinentes del cliente durante los escenarios de soporte técnico

**Guía**: no disponible; la Caja de seguridad del cliente todavía no se admite en Azure Service Fabric.

* [Lista de servicios admitidos por la Caja de seguridad del cliente](../security/fundamentals/customer-lockbox-overview.md#supported-services-and-scenarios-in-general-availability)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

## <a name="data-protection"></a>Protección de los datos

*Para más información, consulte [Control de seguridad: Protección de datos](../security/benchmarks/security-control-data-protection.md).*

### <a name="41-maintain-an-inventory-of-sensitive-information"></a>4.1: Mantenimiento de un inventario de información confidencial

**Guía**: use etiquetas en los recursos relacionados con las implementaciones de clústeres de Azure Service Fabric para ayudar a realizar un seguimiento de los recursos de Azure que almacenan o procesan información confidencial.

* [Creación y uso de etiquetas](../azure-resource-manager/management/tag-resources.md)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="42-isolate-systems-storing-or-processing-sensitive-information"></a>4.2: Aislamiento de los sistemas que almacenan o procesan información confidencial

**Instrucciones**: Implemente suscripciones y/o grupos de administración independientes para los entornos de desarrollo, prueba y producción. Los recursos deben separarse por red virtual o subred, etiquetarse adecuadamente y estar protegidos por un grupo de seguridad de red o Azure Firewall. Los recursos que almacenan o procesan datos confidenciales deben estar suficientemente aislados. Para las máquinas virtuales que almacenan o procesan datos confidenciales, implemente la directiva y los procedimientos necesarios para desactivarlos cuando no estén en uso.

* [Creación de suscripciones adicionales de Azure](../cost-management-billing/manage/create-subscription.md)

* [Creación de grupos de administración](../governance/management-groups/create.md)

* [Creación y uso de etiquetas](../azure-resource-manager/management/tag-resources.md)

* [Creación de una red virtual](../virtual-network/quick-create-portal.md)

* [Creación de un grupo de seguridad de red con una configuración de seguridad](../virtual-network/tutorial-filter-network-traffic.md)

* [Implementación de Azure Firewall](../firewall/tutorial-firewall-deploy-portal.md)

* [Configuración de alertas o alertas y denegación con Azure Firewall](../firewall/threat-intel.md)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="43-monitor-and-block-unauthorized-transfer-of-sensitive-information"></a>4.3: Supervisión y bloqueo de una transferencia no autorizada de información confidencial

**Guía**: Implemente una herramienta automatizada en los perímetros de red que supervise la transferencia no autorizada de información confidencial y bloquee dichas transferencias al tiempo que alerta a los profesionales de seguridad de la información.

En el caso de la plataforma subyacente administrada por Microsoft, Microsoft trata todo el contenido de los clientes como confidencial y hace grandes esfuerzos para proteger a los clientes contra la pérdida y exposición de sus datos. Para garantizar la seguridad de los datos de los clientes dentro de Azure, Microsoft ha implementado y mantiene un conjunto de controles y funcionalidades eficaces

de protección de datos.

* [Descripción de la protección de datos de los clientes en Azure](../security/fundamentals/protection-customer-data.md)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Compartido

### <a name="44-encrypt-all-sensitive-information-in-transit"></a>4.4: Cifrado de toda la información confidencial en tránsito

**Instrucciones**: Cifre toda la información confidencial en tránsito. Asegúrese de que los clientes que se conectan a los recursos de Azure pueden negociar TLS 1.2 o superior.

Siga las recomendaciones de Azure Security Center para el cifrado en reposo y el cifrado en tránsito, si procede.

* [Descripción del cifrado en tránsito con Azure](../security/fundamentals/encryption-overview.md#encryption-of-data-in-transit)

* [Escenarios de seguridad de los clústeres de Service Fabric](./service-fabric-cluster-security.md)

* [Guía de solución de problemas de Service Fabric para la configuración de TLS](https://github.com/Azure/Service-Fabric-Troubleshooting-Guides/blob/master/Security/TLS%20Configuration.md)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Compartido

### <a name="45-use-an-active-discovery-tool-to-identify-sensitive-data"></a>4.5: Uso de una herramienta de detección activa para identificar datos confidenciales

**Instrucciones**: las características de identificación, clasificación y prevención de pérdida de datos todavía no están disponibles para Azure Storage ni los recursos de proceso. Implemente una solución de terceros, si es necesario, para fines de cumplimiento.

En el caso de la plataforma subyacente administrada por Microsoft, Microsoft trata todo el contenido de los clientes como confidencial y hace grandes esfuerzos para proteger a los clientes contra la pérdida y exposición de sus datos. Para garantizar la seguridad de los datos de los clientes dentro de Azure, Microsoft ha implementado y mantiene un conjunto de controles y funcionalidades eficaces de protección de datos.

* [Descripción de la protección de datos de los clientes en Azure](../security/fundamentals/protection-customer-data.md)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Compartido

### <a name="46-use-role-based-access-control-to-control-access-to-resources"></a>4.6: Uso del control de acceso basado en rol para controlar el acceso a los recursos

**Guía**: No aplicable; esta recomendación está pensada para los recursos que no son de proceso y que están diseñados para almacenar datos.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="47-use-host-based-data-loss-prevention-to-enforce-access-control"></a>4.7: Uso de la prevención de pérdida de datos basada en host para aplicar el control de acceso

**Instrucciones**: en el caso de los clústeres de Azure Service Fabric que almacenan o procesan información confidencial, marque el clúster y los recursos relacionados como confidenciales mediante etiquetas. Las características de identificación, clasificación y prevención de pérdida de datos todavía no están disponibles para Azure Storage ni los recursos de proceso. Implemente una solución de terceros, si es necesario, para fines de cumplimiento.

En el caso de la plataforma subyacente administrada por Microsoft, Microsoft trata todo el contenido de los clientes como confidencial y hace grandes esfuerzos para proteger a los clientes contra la pérdida y exposición de sus datos. Para garantizar la seguridad de los datos de los clientes dentro de Azure, Microsoft ha implementado y mantiene un conjunto de controles y funcionalidades eficaces de protección de datos.

* [Descripción de la protección de datos de los clientes en Azure](../security/fundamentals/protection-customer-data.md)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Compartido

### <a name="48-encrypt-sensitive-information-at-rest"></a>4.8: Cifrado de información confidencial en reposo

**Guía**: Use el cifrado en reposo en todos los recursos de Azure. Microsoft recomienda permitir que Azure administre las claves de cifrado, pero existe la opción de administrar sus propias claves en algunas instancias.

* [Descripción del cifrado en reposo en Azure](../security/fundamentals/encryption-atrest.md)

* [Configuración de las claves de cifrado administradas por el cliente](../storage/common/storage-encryption-keys-portal.md)

* [Habilitación del cifrado de disco para nodos de clústeres de Azure Service Fabric en Windows](./service-fabric-enable-azure-disk-encryption-windows.md)

* [Habilitación del cifrado de disco para nodos de clústeres de Azure Service Fabric en Linux](./service-fabric-enable-azure-disk-encryption-linux.md)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="49-log-and-alert-on-changes-to-critical-azure-resources"></a>4.9: Registro y alerta de cambios en los recursos críticos de Azure

**Guía**: Use Azure Monitor con el registro de actividad de Azure para crear alertas para cuando se produzcan cambios en los recursos críticos de Azure.

* [Creación de alertas para los eventos del registro de actividad de Azure](../azure-monitor/platform/alerts-activity-log.md)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

## <a name="vulnerability-management"></a>Administración de vulnerabilidades

*Para más información, consulte [Control de seguridad: Administración de vulnerabilidades](../security/benchmarks/security-control-vulnerability-management.md).*

### <a name="51-run-automated-vulnerability-scanning-tools"></a>5.1: Ejecute herramientas de análisis de vulnerabilidades automatizado

**Guía**: ejecute regularmente el servicio de análisis de errores de Service Fabric y los servicios de caos para simular errores en todo el clúster con el fin de evaluar la solidez y confiabilidad de los servicios.

Siga las recomendaciones de Azure Security Center sobre cómo realizar evaluaciones de vulnerabilidades en sus máquinas virtuales de Azure e imágenes de contenedor.

Use una solución de terceros para realizar evaluaciones de vulnerabilidades en dispositivos de red y aplicaciones web. Al realizar exámenes remotos, no use una cuenta administrativa única y perpetua. Considere la posibilidad de implementar la metodología de aprovisionamiento JIT para la cuenta de examen. Las credenciales de la cuenta de examen deben protegerse, supervisarse y utilizarse solo para el examen de vulnerabilidades.

* [Introducción al servicio de análisis de errores de Service Fabric](./service-fabric-testability-overview.md)

* [Inducción al caos controlado en clústeres de Service Fabric](./service-fabric-controlled-chaos.md)

* [Implementación de las recomendaciones de evaluación de vulnerabilidades de Azure Security Center](../security-center/security-center-vulnerability-assessment-recommendations.md)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="52-deploy-automated-operating-system-patch-management-solution"></a>5.2: Implemente una solución de administración de revisiones de sistema operativo automatizada

**Instrucciones**: habilite las actualizaciones automáticas de imágenes del SO en los conjuntos de escalado de máquinas virtuales del clúster de Azure Service Fabric.

Como alternativa, para probar primero las revisiones del SO antes de pasar a producción, use el desencadenador manual para las actualizaciones de imágenes del SO del conjunto de escalado. Tenga en cuenta que la opción de desencadenador manual no proporciona reversión integrada. Supervise las revisiones del SO mediante Update Management de Azure Automation.

* [Administración de revisiones para nodos de clúster de Service Fabric](./service-fabric-best-practices-infrastructure-as-code.md#azure-virtual-machine-operating-system-automatic-upgrade-configuration)

* [Actualizaciones automáticas de imágenes del SO con conjuntos de escalado de máquinas virtuales de Azure](../virtual-machine-scale-sets/virtual-machine-scale-sets-automatic-upgrade.md)

* [Actualización de las máquinas virtuales con el modelo de conjunto de escalado más reciente](../virtual-machine-scale-sets/virtual-machine-scale-sets-upgrade-scale-set.md#how-to-bring-vms-up-to-date-with-the-latest-scale-set-model)

* [Introducción a Update Management de Azure Automation](../automation/update-management/update-mgmt-overview.md)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="53-deploy-automated-patch-management-solution-for-third-party-software-titles"></a>5.3: Implementación de una solución de administración de revisiones automatizada de títulos de software de terceros

**Guía**: habilite las actualizaciones automáticas de imágenes del SO en los conjuntos de escalado de máquinas virtuales del clúster de Azure Service Fabric. La aplicación de orquestación de revisiones (POA) es una solución alternativa pensada para clústeres de Service Fabric hospedados fuera de Azure. POA se puede usar con clústeres de Azure, con una sobrecarga de hospedaje adicional.

* [Administración de revisiones para nodos de clúster de Service Fabric](./service-fabric-best-practices-infrastructure-as-code.md#azure-virtual-machine-operating-system-automatic-upgrade-configuration)

* [Actualizaciones automáticas de imágenes del SO con conjuntos de escalado de máquinas virtuales de Azure](../virtual-machine-scale-sets/virtual-machine-scale-sets-automatic-upgrade.md)

* [Configuración de la programación de la aplicación de revisiones del SO para clústeres de Service Fabric](./service-fabric-patch-orchestration-application.md)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="54-compare-back-to-back-vulnerability-scans"></a>5.4: Compare los exámenes de vulnerabilidades opuestos

**Guía**: Exporte los resultados de análisis en intervalos coherentes y compare los resultados para comprobar que se han corregido las vulnerabilidades. Al usar recomendaciones de administración de vulnerabilidades sugeridas por Azure Security Center, puede dinamizar en el portal de la solución seleccionada para ver los datos de análisis históricos.

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="55-use-a-risk-rating-process-to-prioritize-the-remediation-of-discovered-vulnerabilities"></a>5.5: Use un proceso de clasificación de riesgos para priorizar la corrección de las vulnerabilidades detectadas

**Instrucciones**: use un programa de puntuación de riesgos común (por ejemplo, Common Vulnerability Scoring System) o la clasificación de riesgos predeterminada proporcionada por su herramienta de examen de terceros.

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

## <a name="inventory-and-asset-management"></a>Inventario y administración de recursos

*Para más información, consulte [Control de seguridad: Administración de recursos y del inventario](../security/benchmarks/security-control-inventory-asset-management.md).*

### <a name="61-use-automated-asset-discovery-solution"></a>6.1: Uso de la solución de detección de recursos automatizada

**Guía**: Use Azure Resource Graph para consultar o detectar todos los recursos (por ejemplo, proceso, almacenamiento, red, puertos y protocolos, etc.) dentro de las suscripciones. Asegúrese de que tiene los permisos adecuados (lectura) en el inquilino y enumere todas las suscripciones de Azure, así como los recursos de las suscripciones.

Aunque los recursos clásicos de Azure se pueden detectar a través de Resource Graph, se recomienda encarecidamente crear y usar los recursos de Azure Resource Manager que figuran a continuación.

* [Creación de consultas con Azure Resource Graph](../governance/resource-graph/first-query-portal.md)

* [Visualización de las suscripciones de Azure](/powershell/module/az.accounts/get-azsubscription?view=azps-3.0.0)

* [Descripción de Azure RBAC](../role-based-access-control/overview.md)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="62-maintain-asset-metadata"></a>6.2: Mantenimiento de metadatos de recursos

**Instrucciones**: Aplique etiquetas a los recursos de Azure que proporcionan metadatos para organizarlos de forma lógica en una taxonomía.

* [Creación y uso de etiquetas](../azure-resource-manager/management/tag-resources.md)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="63-delete-unauthorized-azure-resources"></a>6.3: Eliminación de recursos de Azure no autorizados

**Instrucciones**: Use el etiquetado, los grupos de administración y las suscripciones independientes, si procede, para organizar y realizar un seguimiento de los recursos. Concilie el inventario periódicamente y asegúrese de que los recursos no autorizados se eliminan de la suscripción de manera oportuna.

* [Creación de suscripciones adicionales de Azure](../cost-management-billing/manage/create-subscription.md)

* [Creación de grupos de administración](../governance/management-groups/create.md)

* [Creación y uso de etiquetas](../azure-resource-manager/management/tag-resources.md)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="64-define-and-maintain-inventory-of-approved-azure-resources"></a>6.4: Definición y mantenimiento de un inventario de los recursos de Azure aprobados

**Guía**: Defina los recursos de Azure y el software aprobados para los recursos de proceso.

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="65-monitor-for-unapproved-azure-resources"></a>6.5: Supervisión de recursos de Azure no aprobados

**Instrucciones**: use Azure Policy para establecer restricciones sobre el tipo de recursos que se pueden crear en las suscripciones del cliente con las siguientes definiciones de directiva integradas:

Tipos de recursos no permitidos

Tipos de recursos permitidos

Use Azure Resource Graph para consultar o detectar recursos dentro de las suscripciones. Asegúrese de que todos los recursos de Azure presentes en el entorno estén aprobados.

* [Configuración y administración de Azure Policy](../governance/policy/tutorials/create-and-manage.md)

* [Creación de consultas con Azure Graph](../governance/resource-graph/first-query-portal.md)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="66-monitor-for-unapproved-software-applications-within-compute-resources"></a>6.6: Supervisión de aplicaciones de software no aprobadas en recursos de proceso

**Instrucciones**: implemente una solución de terceros para supervisar los nodos del clúster en busca de aplicaciones de software no aprobadas.

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="67-remove-unapproved-azure-resources-and-software-applications"></a>6.7: Eliminación de aplicaciones de software y recursos de Azure no aprobadas

**Instrucciones**: use Azure Resource Graph para consultar o detectar todos los recursos (por ejemplo, proceso, almacenamiento, red, puertos y protocolos, etc.), incluidos los clústeres de Azure Service Fabric, dentro de las suscripciones. Quite los recursos de Azure no aprobados que detecte. En el caso de los nodos del clúster de Azure Service Fabric, implemente una solución de terceros para quitar software no aprobado o alertas sobre este.

* [Creación de consultas con Azure Graph](../governance/resource-graph/first-query-portal.md)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="68-use-only-approved-applications"></a>6.8: Uso exclusivo de aplicaciones aprobadas

**Guía**: en el caso de los nodos del clúster de Azure Service Fabric, implemente una solución de terceros para impedir que se ejecute software no autorizado.

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="69-use-only-approved-azure-services"></a>6.9: Uso exclusivo de servicios de Azure aprobados

**Guía**: Use Azure Policy para restringir qué servicios puede aprovisionar en su entorno.

* [Configuración y administración de Azure Policy](../governance/policy/tutorials/create-and-manage.md)

* [Denegación de un tipo de recurso específico con Azure Policy](../governance/policy/samples/index.md)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="610-maintain-an-inventory-of-approved-software-titles"></a>6.10: Mantenimiento de un inventario de títulos de software aprobados

**Guía**: en el caso de los nodos del clúster de Azure Service Fabric, implemente una solución de terceros para impedir que se ejecuten tipos de archivos no autorizados.

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="611-limit-users-ability-to-interact-with-azure-resource-manager"></a>6.11: Limitación de la capacidad de los usuarios para interactuar con Azure Resource Manager

**Instrucciones**: Use el acceso condicional de Azure para limitar la capacidad de los usuarios de interactuar con Azure Resource Manager mediante la configuración de la opción "Bloquear acceso" en la aplicación "Administración de Microsoft Azure".

* [Configuración del acceso condicional para bloquear el acceso a Azure Resource Manager](../role-based-access-control/conditional-access-azure-management.md)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="612-limit-users-ability-to-execute-scripts-within-compute-resources"></a>6.12: Limitación de capacidad de los usuarios para ejecutar scripts en recursos de proceso

**Guía**: Use configuraciones específicas del sistema operativo o recursos de terceros para limitar la capacidad de los usuarios de ejecutar scripts en los recursos de proceso de Azure.

* [Por ejemplo, cómo controlar la ejecución de scripts de PowerShell en entornos Windows](/powershell/module/microsoft.powershell.security/set-executionpolicy?view=powershell-6)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="613-physically-or-logically-segregate-high-risk-applications"></a>6.13: Segregación física o lógica de aplicaciones de alto riesgo

**Guía**: El software que se requiere para las operaciones empresariales, pero que puede suponer un riesgo mayor para la organización, debe aislarse en su propia máquina virtual o red virtual y estar lo suficientemente protegida con Azure Firewall o un grupo de seguridad de red.

* [Creación de una red virtual](../virtual-network/quick-create-portal.md)

* [Creación de un grupo de seguridad de red con una configuración de seguridad](../virtual-network/tutorial-filter-network-traffic.md)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

## <a name="secure-configuration"></a>Configuración segura

*Para más información, consulte [Control de seguridad: Configuración segura](../security/benchmarks/security-control-secure-configuration.md).*

### <a name="71-establish-secure-configurations-for-all-azure-resources"></a>7.1: Establezca configuraciones seguras para todos los recursos de Azure

**Guía**: use alias de Azure Policy en el espacio de nombres "Microsoft.ServiceFabric" para crear directivas personalizadas con el fin de auditar o aplicar la configuración de red del clúster de Service Fabric.

* [Visualización de los alias de Azure Policy disponibles](/powershell/module/az.resources/get-azpolicyalias?view=azps-3.3.0)

* [Configuración y administración de Azure Policy](../governance/policy/tutorials/create-and-manage.md)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="72-establish-secure-operating-system-configurations"></a>7.2: Establezca configuraciones del sistema operativo seguras

**Guía**: Microsoft administra y mantiene las imágenes del sistema operativo de Azure Service Fabric. El cliente es responsable de implementar configuraciones seguras para el sistema operativo de los nodos del clúster.

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="73-maintain-secure-azure-resource-configurations"></a>7.3: Mantenga configuraciones de recursos de Azure seguras

**Instrucciones**: use Azure Policy [denegar] e [impleméntela si no existe] para aplicar la configuración segura a los clústeres de Azure Service Fabric y los recursos relacionados.

* [Configuración y administración de Azure Policy](../governance/policy/tutorials/create-and-manage.md)

* [Descripción de los efectos de Azure Policy](../governance/policy/concepts/effects.md)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="74-maintain-secure-operating-system-configurations"></a>7.4: Mantenga configuraciones del sistema operativo seguras

**Guía**: Microsoft administra y mantiene las imágenes del sistema operativo de los clústeres de Azure Service Fabric. El cliente es responsable de implementar la configuración de estado en el nivel de sistema operativo.

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Compartido

### <a name="75-securely-store-configuration-of-azure-resources"></a>7.5: Almacene de forma segura la configuración de los recursos de Azure

**Guía**: Si usa definiciones de personalizadas de Azure Policy, use Azure DevOps o Azure Repos para almacenar y administrar el código de forma segura.

* [Almacenamiento de código en Azure DevOps](/azure/devops/repos/git/gitworkflow?view=azure-devops)

* [Documentación de Azure Repos](/azure/devops/repos/index?view=azure-devops)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="76-securely-store-custom-operating-system-images"></a>7.6: Almacene imágenes de sistema operativo personalizadas de forma segura

**Guía**: Si usa imágenes personalizadas, use RBAC para asegurarse de que solo los usuarios autorizados puedan tener acceso a las imágenes. En el caso de las imágenes de contenedor, almacénelas en Azure Container Registry y aproveche RBAC para asegurarse de que solo los usuarios autorizados puedan acceder a las imágenes.

* [Descripción de RBAC en Azure](../role-based-access-control/rbac-and-directory-admin-roles.md)

* [Descripción de RBAC para Container Registry](../container-registry/container-registry-roles.md)

* [Configuración de RBAC en Azure](../role-based-access-control/quickstart-assign-role-user-portal.md)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="77-deploy-configuration-management-tools-for-azure-resources"></a>7.7: Implementación de herramientas de administración de configuración para recursos de Azure

**Instrucciones**: use alias de Azure Policy en el espacio de nombres "Microsoft.ServiceFabric" para crear directivas personalizadas con el fin de auditar y aplicar las configuraciones del sistema y enviar alertas sobre ellas. Además, desarrolle un proceso y una canalización para administrar las excepciones de las directivas.

* [Configuración y administración de Azure Policy](../governance/policy/tutorials/create-and-manage.md)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="78-deploy-configuration-management-tools-for-operating-systems"></a>7.8: Implementación de herramientas de administración de la configuración para sistemas operativos

**Instrucciones**: no aplicable; esta guía está pensada para recursos de proceso de IaaS.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="79-implement-automated-configuration-monitoring-for-azure-resources"></a>7.9: Implementación de la supervisión de la configuración automatizada para los recursos de Azure

**Instrucciones**: use alias de Azure Policy en el espacio de nombres "Microsoft.ServiceFabric" para crear directivas personalizadas con el fin de auditar o aplicar la configuración del clúster de Service Fabric.

* [Visualización de los alias de Azure Policy disponibles](/powershell/module/az.resources/get-azpolicyalias?view=azps-3.3.0)

* [Configuración y administración de Azure Policy](../governance/policy/tutorials/create-and-manage.md)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="710-implement-automated-configuration-monitoring-for-operating-systems"></a>7.10: Implemente la supervisión de configuración automatizada para sistemas operativos

**Guía**: Use Azure Security Center para realizar exámenes de base de referencia para la configuración del sistema operativo y de Docker para los contenedores.

* [Descripción de las recomendaciones sobre contenedores de Azure Security Center](../security-center/container-security.md)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="711-manage-azure-secrets-securely"></a>7.11: Administre los secretos de Azure de forma segura

**Instrucciones**: Use Managed Service Identity junto con Azure Key Vault para simplificar y proteger la administración de secretos para las aplicaciones en la nube.

* [Uso de identidades administradas para Azure con Service Fabric](./concepts-managed-identity.md)

* [Configuración de la compatibilidad con la identidad administrada en un clúster nuevo de Service Fabric](./configure-new-azure-service-fabric-enable-managed-identity.md)

* [Uso de identidad administrada con una aplicación de Service Fabric](./how-to-managed-identity-service-fabric-app-code.md)

* [Compatibilidad de KeyVaultReference para aplicaciones de Service Fabric](./service-fabric-keyvault-references.md)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="712-manage-identities-securely-and-automatically"></a>7.12: Administre las identidades de forma segura y automática

**Guía**: las identidades administradas se pueden usar en clústeres de Service Fabric implementados en Azure y para aplicaciones implementadas como recursos de Azure. Las identidades administradas le permiten autenticarse en cualquier servicio que admita la autenticación de Azure AD, incluyendo Key Vault, sin necesidad de credenciales en el código.

* [Uso de identidades administradas para Azure con Service Fabric](./concepts-managed-identity.md)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="713-eliminate-unintended-credential-exposure"></a>7.13: Elimine la exposición de credenciales no intencionada

**Guía**: si usa cualquier código relacionado con la implementación de Azure Service Fabric, puede implementar Credential Scanner para identificar las credenciales en el código. El escáner de credenciales también fomenta el traslado de credenciales detectadas a ubicaciones más seguras, como Azure Key Vault.

Use Azure Key Vault para girar automáticamente los certificados de clúster de Service Fabric.

* [Configuración del escáner de credenciales](https://secdevtools.azurewebsites.net/helpcredscan.html)

* [Administración de certificados en clústeres de Service Fabric](./cluster-security-certificate-management.md#certificate-rotation)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

## <a name="malware-defense"></a>Defensa contra malware

*Para más información, consulte [Control de seguridad: defensa contra malware](../security/benchmarks/security-control-malware-defense.md).*

### <a name="81-use-centrally-managed-anti-malware-software"></a>8.1: Use software antimalware administrado centralmente

**Instrucciones**: De forma predeterminada, el antivirus Windows Defender está instalado en Windows Server 2016. La interfaz de usuario está instalada de forma predeterminada en algunas SKU, pero no es necesaria.

Consulte su documentación antimalware para obtener reglas de configuración si no utiliza Windows Defender. Windows Defender no es compatible con Linux.

* [Información sobre el antivirus Windows Defender en Windows Server 2016](/windows/security/threat-protection/windows-defender-antivirus/windows-defender-antivirus-on-windows-server-2016)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="82-pre-scan-files-to-be-uploaded-to-non-compute-azure-resources"></a>8.2: Examine previamente los archivos que se van a cargar en recursos de Azure que no son de proceso

**Guía**: No aplicable; esta recomendación está pensada para los recursos que no son de proceso y que están diseñados para almacenar datos. Microsoft Antimalware está habilitado en el host subyacente que admite los servicios de Azure (por ejemplo, Service Fabric), pero no se ejecuta en el contenido del cliente.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="83-ensure-anti-malware-software-and-signatures-are-updated"></a>8.3: Asegúrese de que se han actualizado el software y las firmas antimalware

**Guía**: No aplicable; esta recomendación está pensada para los recursos que no son de proceso y que están diseñados para almacenar datos. Microsoft Antimalware está habilitado en el host subyacente que admite los servicios de Azure (por ejemplo, Service Fabric), pero no se ejecuta en el contenido del cliente.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

## <a name="data-recovery"></a>Recuperación de datos

*Para más información, consulte [Control de seguridad: recuperación de datos](../security/benchmarks/security-control-data-recovery.md).*

### <a name="91-ensure-regular-automated-back-ups"></a>9.1: Garantía de copias de seguridad automáticas periódicas

**Instrucciones**: El servicio de copia de seguridad y restauración de Service Fabric permite realizar copias de seguridad de información almacenada en los servicios con estado de forma sencilla y automática. Realizar copias de seguridad de datos de aplicaciones de forma periódica es fundamental para evitar la pérdida de datos y la falta de disponibilidad de servicios. Service Fabric proporciona un servicio de restauración y copia de seguridad opcional, que le permite configurar la copia de seguridad periódica de Reliable Services con estado (incluidos los servicios de actor) sin tener que escribir ningún código adicional. También facilita la restauración de las copias de seguridad realizadas previamente.

* [Restauración y copia de seguridad periódicas en un clúster de Azure Service Fabric](./service-fabric-backuprestoreservice-quickstart-azurecluster.md)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="92-perform-complete-system-backups-and-backup-any-customer-managed-keys"></a>9.2: Copias de seguridad completas del sistema y copia de seguridad de las claves administradas por el cliente

**Guía**: habilite el servicio de restauración de copias de seguridad en el clúster de Service Fabric y cree directivas de copia de seguridad para realizar copias de seguridad de los servicios con estado periódicamente y a petición. Realice una copia de seguridad de las claves administradas por el cliente en Azure Key Vault.

* [Restauración y copia de seguridad periódicas en un clúster de Azure Service Fabric](./service-fabric-backuprestoreservice-quickstart-azurecluster.md)

* [Información sobre la configuración de la copia de seguridad periódica en Azure Service Fabric](./service-fabric-backuprestoreservice-configure-periodic-backup.md)

* [Creación de una copia de seguridad de las claves del almacén de claves en Azure](/powershell/module/azurerm.keyvault/backup-azurekeyvaultkey?view=azurermps-6.13.0)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="93-validate-all-backups-including-customer-managed-keys"></a>9.3: Validación de todas las copias de seguridad, incluidas las claves administradas por el cliente

**Guía**: asegúrese de poder realizar la restauración desde el servicio de restauración de copias de seguridad revisando periódicamente la información de configuración de copia de seguridad y las copias de seguridad disponibles. Pruebe la restauración de la copia de seguridad de las claves administradas por el cliente.

* [Información sobre la configuración de la copia de seguridad periódica en Azure Service Fabric](./service-fabric-backuprestoreservice-configure-periodic-backup.md)

* [Restauración de una copia de seguridad en Azure Service Fabric](./service-fabric-backup-restore-service-trigger-restore.md)

* [Restauración de las claves del almacén de claves en Azure](/powershell/module/azurerm.keyvault/restore-azurekeyvaultkey?view=azurermps-6.13.0)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="94-ensure-protection-of-backups-and-customer-managed-keys"></a>9.4: Garantía de la protección de las copias de seguridad y las claves administradas por el cliente

**Guía**: las copias de seguridad del servicio de restauración de copias de seguridad de Service Fabric usan una cuenta de Azure Storage de la suscripción. Azure Storage cifra todos los datos de las cuentas de almacenamiento en reposo. De manera predeterminada, los datos se cifran con claves administradas por Microsoft. Para tener un mayor control sobre las claves de cifrado, puede proporcionar claves administradas por el cliente para cifrar los datos de almacenamiento.

Si usa claves administradas por el cliente, asegúrese de que la eliminación temporal esté habilitada en Key Vault para evitar la eliminación accidental o malintencionada de las claves.

* [Cifrado de Azure Storage para datos en reposo](../storage/common/storage-service-encryption.md)

* [Habilitación de la eliminación temporal en Key Vault](../storage/blobs/soft-delete-overview.md?tabs=azure-portal)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

## <a name="incident-response"></a>Respuesta a los incidentes

*Para más información, consulte [Control de seguridad: Respuesta a los incidentes](../security/benchmarks/security-control-incident-response.md).*

### <a name="101-create-an-incident-response-guide"></a>10.1: Creación de una guía de respuesta ante incidentes

**Guía**: desarrolle una guía de respuesta a incidentes para su organización. Asegúrese de que haya planes de respuesta a incidentes escritos que definan todos los roles del personal, así como las fases de administración y gestión de los incidentes, desde la detección hasta la revisión posterior a la incidencia.

* [Guía para crear su propio proceso de respuesta a incidentes de seguridad](https://msrc-blog.microsoft.com/2019/07/01/inside-the-msrc-building-your-own-security-incident-response-process/)

* [Anatomía de un incidente del Centro de respuestas de seguridad de Microsoft](https://msrc-blog.microsoft.com/2019/06/27/inside-the-msrc-anatomy-of-a-ssirp-incident/)

* [Uso de la guía de control de incidentes de seguridad de equipos de NIST para la creación de su propio plan de respuesta a incidentes](https://csrc.nist.gov/publications/detail/sp/800-61/rev-2/final)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="102-create-an-incident-scoring-and-prioritization-procedure"></a>10.2: Creación de un procedimiento de priorización y puntuación de incidentes

**Instrucciones**: Azure Security Center asigna una gravedad a cada alerta para ayudarle a priorizar aquellas que se deben investigar en primer lugar. La gravedad se basa en la confianza que tiene Security Center en la búsqueda o en el análisis utilizados para emitir la alerta, así como en el nivel de confianza de que ha habido un intento malintencionado detrás de la actividad que ha provocado la alerta.

Adicionalmente, marque las suscripciones con etiquetas y cree un sistema de nomenclatura para identificar y clasificar los recursos de Azure, especialmente los que procesan datos confidenciales. Es su responsabilidad asignar prioridades a la corrección de las alertas en función de la importancia de los recursos y el entorno de Azure donde se produjo el incidente.

* [Alertas de seguridad en el Centro de seguridad de Azure](../security-center/security-center-alerts-overview.md)

* [Uso de etiquetas para organizar los recursos de Azure](../azure-resource-manager/management/tag-resources.md)

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="103-test-security-response-procedures"></a>10.3: Prueba de los procedimientos de respuesta de seguridad

**Instrucciones**: Realice ejercicios para probar las capacidades de respuesta a los incidentes de los sistemas con regularidad. Identifique puntos débiles y brechas y revise el plan según sea necesario.

* [Guía de NIST para probar, entrenar y ejecutar programas para planes y funcionalidades de TI](https://nvlpubs.nist.gov/nistpubs/Legacy/SP/nistspecialpublication800-84.pdf)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="104-provide-security-incident-contact-details-and-configure-alert-notifications-for-security-incidents"></a>10.4: Provisión de detalles de contacto de incidentes de seguridad y configuración de notificaciones de alerta para incidentes de seguridad

**Instrucciones**: La información de contacto del incidente de seguridad la utilizará Microsoft para ponerse en contacto con usted si Microsoft Security Response Center (MSRC) detecta que un tercero no autorizado o ilegal ha accedido a los datos. Revise los incidentes después del hecho para asegurarse de que se resuelven los problemas.

* [Establecimiento del contacto de seguridad de Azure Security Center](../security-center/security-center-provide-security-contact-details.md)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="105-incorporate-security-alerts-into-your-incident-response-system"></a>10.5: Incorporación de alertas de seguridad en el sistema de respuesta a incidentes

**Instrucciones**: Exporte sus alertas y recomendaciones de Azure Security Center mediante la característica de exportación continua. La exportación continua le permite exportar alertas y recomendaciones de forma manual o continua. Puede usar el conector de datos de Azure Security Center para transmitir las alertas a Sentinel.

* [Configuración de la exportación continua](../security-center/continuous-export.md)

* [Transmisión de alertas a Azure Sentinel](../sentinel/connect-azure-security-center.md)

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="106-automate-the-response-to-security-alerts"></a>10.6: Automatización de la respuesta a las alertas de seguridad

**Instrucciones**: use la característica de automatización del flujo de trabajo de Azure Security Center para desencadenar automáticamente respuestas mediante "Logic Apps" en las alertas y recomendaciones de seguridad.

* [Configuración de la automatización de flujo de trabajo y Logic Apps](../security-center/workflow-automation.md)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

## <a name="penetration-tests-and-red-team-exercises"></a>Pruebas de penetración y ejercicios del equipo rojo

*Para más información, consulte [Control de seguridad: Pruebas de penetración y ejercicios del equipo rojo](../security/benchmarks/security-control-penetration-tests-red-team-exercises.md).*

### <a name="111-conduct-regular-penetration-testing-of-your-azure-resources-and-ensure-remediation-of-all-critical-security-findings"></a>11.1: Realice pruebas de penetración periódicas de los recursos de Azure y asegúrese de corregir todos los resultados de seguridad críticos

**Guía**: Siga las reglas de compromiso de la prueba de penetración de Microsoft Cloud para asegurarse de que las pruebas de penetración no infrinjan las directivas de Microsoft. Use la estrategia de Microsoft y la ejecución de las pruebas de penetración del equipo rojo y sitios activos en la infraestructura de nube, los servicios y las aplicaciones administradas por Microsoft.

* [Reglas de interacción de las pruebas de penetración](https://www.microsoft.com/msrc/pentest-rules-of-engagement?rtc=1)

* [Equipo rojo de Microsoft Cloud](https://gallery.technet.microsoft.com/Cloud-Red-Teaming-b837392e)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Compartido

## <a name="next-steps"></a>Pasos siguientes

- Consulte la [prueba comparativa de seguridad de Azure](../security/benchmarks/overview.md).
- Obtenga más información sobre las [líneas de base de seguridad de Azure](../security/benchmarks/security-baselines-overview.md).