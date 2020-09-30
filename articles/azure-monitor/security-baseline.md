---
title: Línea de base de seguridad de Azure para Azure Monitor
description: La línea de base de seguridad de Azure Monitor proporciona una guía de procedimientos y recursos para implementar las recomendaciones de seguridad especificadas en Azure Security Benchmark.
author: msmbaldwin
ms.service: azure-monitor
ms.topic: conceptual
ms.date: 09/01/2020
ms.author: mbaldwin
ms.custom: subject-security-benchmark
ms.openlocfilehash: fd68f720f372ee61f7c441ea83bd365bc2a6f36a
ms.sourcegitcommit: 3fc3457b5a6d5773323237f6a06ccfb6955bfb2d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/11/2020
ms.locfileid: "90032748"
---
# <a name="azure-security-baseline-for-azure-monitor"></a>Línea de base de seguridad de Azure para Azure Monitor

Esta línea de base de seguridad aplica las directrices de [Azure Security Benchmark](../security/benchmarks/overview.md) a Azure Monitor. Azure Security Benchmark proporciona recomendaciones sobre cómo puede proteger sus soluciones de nube en Azure. El contenido se agrupa por medio de los **controles de seguridad** definidos por Azure Security Benchmark y las directrices aplicables a Azure Monitor. Se han excluido los **controles** no aplicables a Azure Monitor. Para ver cómo Azure Monitor se adapta por completo a Azure Security Benchmark, consulte el [archivo de asignación de base de referencia de seguridad de Azure Policy completo](https://github.com/MicrosoftDocs/SecurityBenchmarks/tree/master/Azure%20Offer%20Security%20Baselines).

Azure Monitor forma parte de los servicios principales de Azure y no se puede implementar como un servicio por separado. Los componentes de Azure Monitor se pueden implementar con los recursos del usuario, lo cual puede afectar a la posición de seguridad de esos recursos.

## <a name="network-security"></a>Seguridad de las redes

*Para obtener más información, consulte [Azure Security Benchmark: Seguridad de redes](/azure/security/benchmarks/security-control-network-security).*

### <a name="11-protect-azure-resources-within-virtual-networks"></a>1.1: Protección de los recursos de Azure dentro de las redes virtuales

**Instrucciones**: Habilite Azure Private Link para permitir el acceso a los servicios PaaS de Azure (por ejemplo, Azure Monitor) y a los servicios hospedados de clientes o asociados en Azure a través de un punto de conexión privado de la red virtual. El tráfico entre la red virtual y el servicio atraviesa la red troncal de Microsoft, eliminando la exposición a la red pública de Internet.

Para que el tráfico llegue a Azure Monitor, utilice las etiquetas de servicio "AzureMonitor" para permitir el tráfico entrante y saliente a través de grupos de seguridad de red. Para que el tráfico de prueba de supervisión de disponibilidad llegue a Azure Monitor, utilice la etiqueta de servicio "ApplicationInsightsAvailability" para todo el tráfico entrante a través de grupos de seguridad de red.

Las reglas de red virtual permiten que Azure Monitor acepte solo las comunicaciones que se envían desde subredes seleccionadas dentro de una red virtual.

Utilice la puerta de enlace de Log Analytics para enviar datos a un área de trabajo de Log Analytics en Azure Monitor en nombre de los equipos que no se pueden conectar directamente a Internet y evite así la necesidad de que los equipos se conecten a Internet. 

- [Uso de Azure Private Link para conectar redes a Azure Monitor de forma segura](platform/private-link-security.md)

- [Conexión de equipos sin acceso a Internet mediante la puerta de enlace de Log Analytics en Azure Monitor](platform/gateway.md)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="12-monitor-and-log-the-configuration-and-traffic-of-virtual-networks-subnets-and-network-interfaces"></a>1.2: Supervisión y registro de la configuración y el tráfico de redes virtuales, subredes e interfaces de red

**Guía**: Azure Monitor es un servicio principal y no admite la implementación directa en una red virtual. La infraestructura subyacente la administra Microsoft. No obstante, para los recursos que realizan conexiones de red para la oferta Azure Monitor, es preciso proteger su red mediante un grupo de seguridad de red. Habilite los registros de flujo del grupo de seguridad de red y envíelos a una cuenta de almacenamiento para auditar el tráfico. También puede enviar los registros de flujo a un área de trabajo de Log Analytics y utilizar el análisis de tráfico para proporcionar información sobre el flujo del tráfico en la nube de Azure. Algunas de las ventajas del Análisis de tráfico son la capacidad de visualizar la actividad de la red e identificar las zonas activas, identificar las amenazas de seguridad, comprender los patrones de flujo de tráfico y detectar configuraciones de red incorrectas.

Al utilizar Azure Monitor con Private Link, obtendrá acceso al registro de red. Por ejemplo, los datos procesados por el punto de conexión privado (ENTRADA/SALIDA).

- [Requisitos de red para agentes de Azure Monitor](platform/log-analytics-agent.md#network-requirements)

- [Conexión de equipos sin acceso a Internet mediante la puerta de enlace de Log Analytics en Azure Monitor](platform/gateway.md)

- [Habilitación de los registros de flujo de grupos de seguridad de red](../network-watcher/network-watcher-nsg-flow-logging-portal.md)

- [Habilitación y uso del Análisis de tráfico](../network-watcher/traffic-analytics.md)

- [Descripción de la seguridad de red proporcionada por Azure Security Center](../security-center/security-center-network-recommendations.md)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="18-minimize-complexity-and-administrative-overhead-of-network-security-rules"></a>1.8: Minimice la complejidad y la sobrecarga administrativa de las reglas de seguridad de red

**Instrucciones**: Para que el tráfico llegue a Azure Monitor, utilice las etiquetas de servicio "AzureMonitor" para permitir el tráfico entrante y saliente a través de grupos de seguridad de red. Para que el tráfico de prueba de supervisión de disponibilidad llegue a Azure Monitor, utilice la etiqueta de servicio "ApplicationInsightsAvailability" para todo el tráfico entrante a través de grupos de seguridad de red. Microsoft administra los prefijos de direcciones que la etiqueta de servicio incluye y actualiza automáticamente dicha etiqueta a medida que las direcciones cambian.

- [Descripción y uso de las etiquetas de servicio](../virtual-network/service-tags-overview.md) 

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="110-document-traffic-configuration-rules"></a>1.10: Documente las reglas de configuración de tráfico

**Guía**: Azure Monitor forma parte de los servicios principales de Azure y no se puede implementar como un servicio por separado. Los componentes de Azure Monitor, como el agente de Azure Monitor y el SDK de Application Insights, se pueden implementar con los recursos del usuario, lo cual puede afectar a la posición de seguridad de esos recursos.

- [Requisitos de red para agentes de Azure Monitor](platform/log-analytics-agent.md#network-requirements)

- [Conexión de equipos sin acceso a Internet mediante la puerta de enlace de Log Analytics en Azure Monitor](platform/gateway.md) 

- [Introducción a Application Insights](https://docs.microsoft.com/azure/azure-monitor/app/app-insights-overview#get-started)

- [Cómo configurar las pruebas web de disponibilidad](app/monitor-web-app-availability.md)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="111-use-automated-tools-to-monitor-network-resource-configurations-and-detect-changes"></a>1.11: Use herramientas automatizadas para supervisar las configuraciones de recursos de red y detectar cambios

**Instrucciones**: Utilice Azure Activity Log para supervisar las configuraciones de recursos y detectar cambios en los recursos de red relacionados con Azure Monitor. Cree alertas en Azure Monitor para que se desencadenen al producirse cambios en esos recursos de red críticos.

- [Visualización y recuperación de eventos del registro de actividad de Azure](/azure/azure-monitor/platform/activity-log-view)

- [Creación de alertas en Azure Monitor](platform/alerts-activity-log.md)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

## <a name="logging-and-monitoring"></a>Registro y supervisión

*Para obtener más información, consulte [Azure Security Benchmark: registro y supervisión](/azure/security/benchmarks/security-control-logging-monitoring).*

### <a name="22-configure-central-security-log-management"></a>2.2: Configuración de la administración central de registros de seguridad

**Guía**: Azure Monitor utiliza registros de actividad para registrar los cambios que se producen en sus recursos. Puede exportar estos registros a Azure Storage, al centro de eventos o a un área de trabajo de Log Analytics. Recopile registros mediante Azure Monitor para agregar datos de seguridad generados por dispositivos de punto de conexión, recursos de red y otros sistemas de seguridad. En Azure Monitor, puede consultar y analizar los datos, además de utilizar cuentas de Azure Storage para el almacenamiento de registros a largo plazo o con fines de archivo.

Como alternativa, puede habilitar e incorporar datos en Azure Sentinel o en una herramienta SIEM de terceros.

- [Recopilación de registros y métricas de plataforma con Azure Monitor](platform/diagnostic-settings.md)

- [Recopilación de registros de host internos de máquina virtual de Azure con Azure Monitor](learn/quick-collect-azurevm.md)

- [Incorporación de Azure Sentinel](../sentinel/quickstart-onboard.md)

- [Introducción a Azure Monitor e integración con herramienta SIEM de terceros](https://azure.microsoft.com/blog/use-azure-monitor-to-integrate-with-siem-tools/)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="23-enable-audit-logging-for-azure-resources"></a>2.3: Habilitación del registro de auditoría para recursos de Azure

**Guía**: Azure Monitor utiliza registros de actividad, que se habilitan automáticamente y registran las operaciones realizadas en recursos de Azure Monitor (por ejemplo, quién inició una operación, cuándo se produjo, cuál es el estado de esa operación y otra información de auditoría útil). 

- [Recopilación de registros y métricas de plataforma con Azure Monitor](platform/diagnostic-settings.md)

- [Descripción del registro y de los distintos tipos de registro de Azure](platform/platform-logs-overview.md)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="25-configure-security-log-storage-retention"></a>2.5: Configuración de la retención del almacenamiento de registros de seguridad

**Guía**: En Azure Monitor, establezca el período de retención del área de trabajo de Log Analytics de acuerdo con la normativa de cumplimiento de su organización. Utilice las cuentas de Azure Storage para el almacenamiento de registros a largo plazo o con fines de archivo.

- [Cambio del período de retención de datos en Log Analytics](platform/manage-cost-storage.md#change-the-data-retention-period)

- [Configuración de la directiva de retención para los registros de la cuenta de Azure Storage](../storage/common/storage-monitor-storage-account.md#configure-logging)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="26-monitor-and-review-logs"></a>2.6: Supervisión y revisión de registros

**Guía**: Analice y supervise los registros en busca de comportamientos anómalos y revise los resultados con regularidad. Use Azure Monitor y un área de trabajo de Log Analytics para revisar los registros y realizar consultas en los datos de dichos registros.

También puede habilitar e incorporar datos en Azure Sentinel o en una herramienta SIEM de terceros.

- [Incorporación de Azure Sentinel](../sentinel/quickstart-onboard.md)

- [Introducción a las consultas de Log Analytics](log-query/get-started-portal.md)

- [Procedimiento para realizar consultas personalizadas en Azure Monitor](log-query/get-started-queries.md)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="27-enable-alerts-for-anomalous-activities"></a>2.7: Habilitación de alertas para actividades anómalas

**Guía**: Use Azure Security Center con el área de trabajo de Log Analytics para la supervisión y alertas sobre actividades anómalas que se encuentran en los registros y eventos de seguridad. Como alternativa, puede habilitar e incorporar datos a Azure Sentinel.

- [Incorporación de Azure Sentinel](../sentinel/quickstart-onboard.md)

- [Administración de alertas de seguridad en Azure Security Center](../security-center/security-center-managing-and-responding-alerts.md)

- [Alertas sobre datos de registro de Log Analytics](learn/tutorial-response.md)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

## <a name="identity-and-access-control"></a>Control de identidades y acceso

*Para obtener más información, consulte [Azure Security Benchmark: Identidad y control de acceso](/azure/security/benchmarks/security-control-identity-access-control).*

### <a name="31-maintain-an-inventory-of-administrative-accounts"></a>3.1: Mantenga un inventario de cuentas administrativas

**Guía**: El control de acceso basado en roles de Azure (RBAC de Azure) permite administrar el acceso a los recursos de Azure a través de las asignaciones de roles. Puede asignar estos roles a usuarios, grupos de entidades de servicio e identidades administradas. Hay roles integrados predefinidos para determinados recursos, y estos roles se pueden inventariar o consultar mediante herramientas como la CLI de Azure, Azure PowerShell o el Azure Portal.

- [Obtención de un rol de directorio en Azure AD con PowerShell](https://docs.microsoft.com/powershell/module/azuread/get-azureaddirectoryrole?view=azureadps-2.0)

- [Obtención de los miembros de un rol de directorio en Azure AD con PowerShell](https://docs.microsoft.com/powershell/module/azuread/get-azureaddirectoryrolemember?view=azureadps-2.0)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="33-use-dedicated-administrative-accounts"></a>3.3: Use cuentas administrativas dedicadas

**Guía**: Cree procedimientos operativos estándar en torno al uso de cuentas administrativas dedicadas. Use la administración de identidad y acceso de Azure Security Center para supervisar el número de cuentas administrativas.

También puede habilitar el acceso Just-in-Time/Just-Enough usando roles de Azure AD Privileged Identity Management con privilegios para los servicios de Microsoft y Azure Resource Manager. 

- [Información general sobre Azure AD Privileged Identity Management](../active-directory/privileged-identity-management/pim-configure.md)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="34-use-azure-active-directory-single-sign-on-sso"></a>3.4: Uso del inicio de sesión único (SSO) de Azure Active Directory

**Instrucciones**: Siempre que sea posible, use el SSO de Azure Active Directory en lugar de configurar credenciales independientes individuales por servicio. Use las recomendaciones de administración de identidades y acceso de Azure Security Center.

- [Descripción del SSO con Azure AD](../active-directory/manage-apps/what-is-single-sign-on.md)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="35-use-multi-factor-authentication-for-all-azure-active-directory-based-access"></a>3.5: Uso de la autenticación multifactor para todo el acceso basado en Azure Active Directory

**Instrucciones**: Habilite la MFA de Azure AD y siga las recomendaciones de administración de identidades y acceso de Azure Security Center.

- [Procedimiento para habilitar la MFA en Azure](../active-directory/authentication/howto-mfa-getstarted.md)

- [Supervisión de la identidad y el acceso en Azure Security Center](../security-center/security-center-identity-access.md)

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="36-use-dedicated-machines-privileged-access-workstations-for-all-administrative-tasks"></a>3.6: Use máquinas dedicadas (estaciones de trabajo de acceso con privilegios) para todas las tareas administrativas

**Instrucciones**: use una estación de trabajo segura y administrada por Azure (también conocida como una estación de trabajo de acceso con privilegios o PAW) para las tareas administrativas que requieren privilegios elevados.

- [Descripción de las estaciones de trabajo seguras administradas por Azure](../active-directory/devices/concept-azure-managed-workstation.md)

- [Cómo habilitar MFA de Azure AD](../active-directory/authentication/howto-mfa-getstarted.md)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="37-log-and-alert-on-suspicious-activities-from-administrative-accounts"></a>3.7: Registro y alerta de actividades sospechosas desde cuentas administrativas

**Guía**: use la característica de supervisión y los informes de seguridad de Azure Active Directory para detectar cuando se producen actividades sospechosas o no seguras en el entorno. Use Azure Security Center para supervisar la actividad de identidad y acceso. 

- [Procedimiento para identificar usuarios de Azure AD marcados por una actividad de riesgo](/azure/active-directory/reports-monitoring/concept-user-at-risk)

- [Supervisión de la actividad de identidad y acceso de los usuarios en Azure Security Center](../security-center/security-center-identity-access.md)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="38-manage-azure-resources-from-only-approved-locations"></a>3.8: Administre los recursos de Azure solo desde ubicaciones aprobadas

**Guía**: Use ubicaciones con nombre de acceso condicional para permitir el acceso solo desde agrupaciones lógicas específicas de intervalos de direcciones IP o países o regiones.

- [Configuración de ubicaciones con nombre en Azure](../active-directory/reports-monitoring/quickstart-configure-named-locations.md)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="39-use-azure-active-directory"></a>3.9: Uso de Azure Active Directory

**Instrucciones**: Use Azure Active Directory (AD) como sistema central de autenticación y autorización. Azure AD protege los datos mediante un cifrado seguro para los datos en reposo y en tránsito. Azure AD también cifra con sal, convierte en hash y almacena de forma segura las credenciales de los usuarios.

- [Procedimiento para crear y configurar una instancia de Azure AD](../active-directory/fundamentals/active-directory-access-create-new-tenant.md)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="310-regularly-review-and-reconcile-user-access"></a>3.10: Revise y concilie regularmente el acceso de los usuarios

**Guía**: Azure AD proporciona registros para ayudar a descubrir cuentas obsoletas. Además, use las revisiones de acceso de identidad de Azure para administrar de forma eficiente las pertenencias a grupos, el acceso a las aplicaciones empresariales y las asignaciones de roles. El acceso de los usuarios se puede revisar de forma periódica para asegurarse de que solo las personas adecuadas tengan acceso continuado. 

- [Descripción de los informes de Azure AD](/azure/active-directory/reports-monitoring/)

- [Procedimiento para usar las revisiones de acceso de identidad de Azure](../active-directory/governance/access-reviews-overview.md)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="311-monitor-attempts-to-access-deactivated-credentials"></a>3.11: Supervisión de los intentos de acceso a credenciales desactivadas

**Instrucciones**: Tiene acceso a los orígenes de registro de eventos de actividad de inicio de sesión, auditoría y riesgo de Azure AD, que permiten la integración con cualquier herramienta de SIEM o supervisión. Puede simplificar este proceso creando una configuración de diagnóstico para las cuentas de usuario de Azure Active Directory y enviando los registros de auditoría y los registros de inicio de sesión a un área de trabajo de Log Analytics. Puede configurar las alertas deseadas en el área de trabajo de Log Analytics.

- [Integración de los registros de actividad de Azure en Azure Monitor](/azure/active-directory/reports-monitoring/howto-integrate-activity-logs-with-log-analytics)

**Supervisión de Azure Security Center**: Anular. Proporcione un valor para el elemento de trabajo.

**Responsabilidad**: Customer

### <a name="312-alert-on-account-sign-in-behavior-deviation"></a>3.12: Alerta sobre las desviaciones de comportamiento en los inicios de sesión de las cuentas

**Instrucciones**: Use las características de protección de riesgos e identidad de Azure AD para configurar las respuestas automatizadas a las acciones sospechosas detectadas relacionadas con las identidades de los usuarios. También puede hacer que Azure Sentinel ingiera los datos para investigarlos más.

- [Visualización de los inicios de sesión de riesgo de Azure AD](/azure/active-directory/reports-monitoring/concept-risky-sign-ins)

- [Configuración y habilitación de las directivas de riesgo de protección de identidad](../active-directory/identity-protection/howto-identity-protection-configure-risk-policies.md)

- [Incorporación de Azure Sentinel](../sentinel/quickstart-onboard.md)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

## <a name="data-protection"></a>Protección de los datos

*Para obtener más información, consulte [Azure Security Benchmark: Protección de datos](/azure/security/benchmarks/security-control-data-protection).*

### <a name="41-maintain-an-inventory-of-sensitive-information"></a>4.1: Mantenimiento de un inventario de información confidencial

**Guía**: Utilice etiquetas cuando sea posible para facilitar el seguimiento de los recursos de Azure Monitor que almacenan o procesan información confidencial como las áreas de trabajo de Log Analytics.

- [Creación y uso de etiquetas](/azure/azure-resource-manager/resource-group-using-tags)

- [Administración del acceso a los datos de registro y las áreas de trabajo en Azure Monitor](platform/manage-access.md)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="42-isolate-systems-storing-or-processing-sensitive-information"></a>4.2: Aislamiento de los sistemas que almacenan o procesan información confidencial

**Guía**: Implemente el aislamiento mediante suscripciones independientes y grupos de administración para dominios de seguridad individuales, como el tipo de entorno y el nivel de confidencialidad de los datos. Puede restringir el nivel de acceso a los recursos de Azure Monitor y los recursos relacionados que necesitan sus aplicaciones y entornos empresariales. Puede regular el acceso a los recursos de Azure Monitor mediante el control de acceso basado en roles de Azure Active Directory.

- [Creación de suscripciones adicionales de Azure](/azure/billing/billing-create-subscription)

- [Creación de grupos de administración](/azure/governance/management-groups/create)

- [Creación y uso de etiquetas](/azure/azure-resource-manager/resource-group-using-tags)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="44-encrypt-all-sensitive-information-in-transit"></a>4.4: Cifrado de toda la información confidencial en tránsito

**Guía**: De manera predeterminada, Azure Monitor negocia TLS 1.2. Asegúrese de que los clientes que se conectan a los recursos de Azure pueden negociar TLS 1.2 o superior. 

Application Insights y Log Analytics continúan permitiendo la ingesta de datos de TLS 1.1 y TLS 1.0. Los datos se pueden restringir a TLS 1.2 mediante la configuración en el cliente.

- [Cómo enviar datos de forma segura mediante TLS 1.2](platform/data-security.md#sending-data-securely-using-tls-12)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Compartido

### <a name="45-use-an-active-discovery-tool-to-identify-sensitive-data"></a>4.5: Uso de una herramienta de detección activa para identificar datos confidenciales

**Instrucciones**: Las características de identificación, clasificación y prevención de pérdida de datos todavía no están disponibles para Azure Monitor. Implemente una solución de terceros, si es necesario, para fines de cumplimiento.
En el caso de la plataforma subyacente administrada por Microsoft, Microsoft trata todo el contenido de los clientes como confidencial y hace grandes esfuerzos para proteger a los clientes contra la pérdida y exposición de sus datos. Para garantizar la seguridad de los datos de los clientes dentro de Azure, Microsoft ha implementado y mantiene un conjunto de controles y funcionalidades eficaces de protección de datos.

- [Descripción de la protección de datos de los clientes en Azure](../security/fundamentals/protection-customer-data.md)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Compartido

### <a name="46-use-role-based-access-control-to-control-access-to-resources"></a>4.6: Uso del control de acceso basado en rol para controlar el acceso a los recursos

**Guía**: Utilice el control de acceso basado en roles (RBAC) de Azure para administrar el acceso a Azure Monitor.

- [Roles, permisos y seguridad en Azure Monitor](platform/roles-permissions-security.md)

- [Configuración de Azure RBAC](../role-based-access-control/role-assignments-portal.md)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="48-encrypt-sensitive-information-at-rest"></a>4.8: Cifrado de información confidencial en reposo

**Guía**: Azure Monitor garantiza que todos los datos y las consultas guardadas se cifren en reposo mediante claves administradas por Microsoft (MMK). Azure Monitor también proporciona una opción para el cifrado mediante su propia clave almacenada en Azure Key Vault a la que se tiene acceso mediante la autenticación de la identidad administrada asignada por el sistema. Esta clave administrada por el cliente (CMK) se puede proteger mediante software o hardware (HSM).

- [Claves administradas por el cliente de Azure Monitor](platform/customer-managed-keys.md)

- [Seguridad de datos de Log Analytics](platform/data-security.md)

- [Recopilación, retención y almacenamiento de datos en Application Insights](app/data-retention-privacy.md)

- [Descripción del cifrado en reposo en Azure](../security/fundamentals/encryption-atrest.md) 

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="49-log-and-alert-on-changes-to-critical-azure-resources"></a>4.9: Registro y alerta de cambios en los recursos críticos de Azure

**Guía**: Utilice Azure Monitor con Azure Activity Log para crear alertas cuando se produzcan cambios en Azure Monitor y los recursos relacionados.

- [Creación de alertas para los eventos del registro de actividad de Azure](platform/alerts-activity-log.md)

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

## <a name="vulnerability-management"></a>Administración de vulnerabilidades

*Para obtener más información, consulte [Azure Security Benchmark: Administración de vulnerabilidades](/azure/security/benchmarks/security-control-vulnerability-management).*

### <a name="55-use-a-risk-rating-process-to-prioritize-the-remediation-of-discovered-vulnerabilities"></a>5.5: Use un proceso de clasificación de riesgos para priorizar la corrección de las vulnerabilidades detectadas

**Instrucciones**: Use un programa de puntuación de riesgos común (por ejemplo, Common Vulnerability Scoring System) o la clasificación de riesgos predeterminada proporcionada por su herramienta de análisis de terceros.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

## <a name="inventory-and-asset-management"></a>Inventario y administración de recursos

*Para obtener más información, consulte [Azure Security Benchmark: Administración de recursos y del inventario](/azure/security/benchmarks/security-control-inventory-asset-management).*

### <a name="61-use-automated-asset-discovery-solution"></a>6.1: Uso de la solución de detección de recursos automatizada

**Guía**: Utilice la CLI de Azure para consultar y detectar recursos de Azure Monitor en sus suscripciones. Asegúrese de que tiene los permisos adecuados (lectura) en el inquilino y enumere todas las suscripciones de Azure, así como los recursos de las suscripciones.

- [CLI de Azure Monitor](https://docs.microsoft.com/cli/azure/monitor?view=azure-cli-latest)

- [Visualización de las suscripciones de Azure](https://docs.microsoft.com/powershell/module/az.accounts/get-azsubscription?view=azps-3.0.0)

- [Descripción de Azure RBAC](../role-based-access-control/overview.md)

- [Roles, permisos y seguridad en Azure Monitor](platform/roles-permissions-security.md)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="62-maintain-asset-metadata"></a>6.2: Mantenimiento de metadatos de recursos

**Guía**: Aplique etiquetas a los recursos de Azure Monitor que proporcionan metadatos para organizarlos de forma lógica en una taxonomía.

- [Creación y uso de etiquetas](/azure/azure-resource-manager/resource-group-using-tags)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="63-delete-unauthorized-azure-resources"></a>6.3: Eliminación de recursos de Azure no autorizados

**Guía**: Utilice el etiquetado, los grupos de administración y las suscripciones independientes, según corresponda, para organizar y hacer un seguimiento de los recursos relacionados con Azure Monitor. Concilie el inventario periódicamente y asegúrese de que los recursos no autorizados se eliminan de la suscripción de manera oportuna.

- [Creación de suscripciones adicionales de Azure](/azure/billing/billing-create-subscription)

- [Creación de grupos de administración](/azure/governance/management-groups/create)

- [Creación y uso de etiquetas](/azure/azure-resource-manager/resource-group-using-tags)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="64-define-and-maintain-inventory-of-approved-azure-resources"></a>6.4: Definición y mantenimiento de un inventario de los recursos de Azure aprobados

**Guía**: Cree un inventario de los recursos de Azure aprobados y el software aprobado para los recursos de proceso según las necesidades de la organización.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="65-monitor-for-unapproved-azure-resources"></a>6.5: Supervisión de recursos de Azure no aprobados

**Guía**: use Azure Policy para establecer restricciones sobre el tipo de recursos que se pueden crear en las suscripciones.

Use Azure Resource Graph para consultar y detectar recursos dentro de sus suscripciones.  Asegúrese de que todos los recursos de Azure presentes en el entorno estén aprobados.

- [Configuración y administración de Azure Policy](../governance/policy/tutorials/create-and-manage.md)

- [Creación de consultas con Azure Resource Graph Explorer](../governance/resource-graph/first-query-portal.md)

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="67-remove-unapproved-azure-resources-and-software-applications"></a>6.7: Eliminación de aplicaciones de software y recursos de Azure no aprobadas

**Guía**: Concilie el inventario periódicamente y asegúrese de que los recursos no autorizados relacionados con Azure Monitor se eliminen de la suscripción de manera oportuna.  

- [Eliminación de un área de trabajo de Azure Log Analytics](platform/delete-workspace.md)

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="69-use-only-approved-azure-services"></a>6.9: Uso exclusivo de servicios de Azure aprobados

**Guía**: Utilice Azure Policy para restringir los recursos relacionados con Azure Monitor que se pueden aprovisionar en su entorno. 

- [Configuración y administración de Azure Policy](../governance/policy/tutorials/create-and-manage.md) 

- [Denegación de un tipo de recurso específico con Azure Policy](/azure/governance/policy/samples/not-allowed-resource-types)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="611-limit-users-ability-to-interact-with-azure-resource-manager"></a>6.11: Limitación de la capacidad de los usuarios para interactuar con Azure Resource Manager

**Instrucciones**: Use el acceso condicional de Azure para limitar la capacidad de los usuarios de interactuar con Azure Resource Manager mediante la configuración de la opción "Bloquear acceso" en la aplicación "Administración de Microsoft Azure".

- [Configuración del acceso condicional para bloquear el acceso a Azure Resource Manager](../role-based-access-control/conditional-access-azure-management.md)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

## <a name="secure-configuration"></a>Configuración segura

*Para obtener más información, consulte [Azure Security Benchmark: Configuración segura](/azure/security/benchmarks/security-control-secure-configuration).*

### <a name="71-establish-secure-configurations-for-all-azure-resources"></a>7.1: Establezca configuraciones seguras para todos los recursos de Azure

**Guía**: Utilice definiciones personalizadas de Azure Policy para auditar o aplicar la configuración de los recursos relacionados con Azure Monitor. También puede usar definiciones integradas de Azure Policy.

Además, Azure Resource Manager tiene la capacidad de exportar la plantilla en notación de objetos JavaScript (JSON), que se debe revisar para asegurarse de que las configuraciones cumplan o superen los requisitos de seguridad de la organización.

También puede usar las recomendaciones de Azure Security Center como línea de base de configuración segura para los recursos de Azure.

Si utiliza funcionalidades de APM de streaming en vivo, proteja el canal mediante una clave de API secreta, además de la clave de instrumentación.

- [Protección de Live Metrics Stream de APM](app/live-stream.md#secure-the-control-channel)

- [Visualización de los alias de Azure Policy disponibles](https://docs.microsoft.com/powershell/module/az.resources/get-azpolicyalias?view=azps-3.3.0)

- [Tutorial: Creación y administración de directivas para aplicar el cumplimiento](../governance/policy/tutorials/create-and-manage.md)

- [Exportación de uno y varios recursos a una plantilla en Azure Portal](../azure-resource-manager/templates/export-template-portal.md)

- [Guía de referencia sobre las recomendaciones de seguridad](../security-center/recommendations-reference.md)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="73-maintain-secure-azure-resource-configurations"></a>7.3: Mantenga configuraciones de recursos de Azure seguras

**Guía**: Utilice las directivas [deny] y [deploy if not exist] de Azure Policy para aplicar una configuración segura en los recursos relacionados con Azure Monitor.  Además, puede usar plantillas de Azure Resource Manager para mantener la configuración de seguridad de los recursos relacionados con Azure Monitor que requiere su organización.

- [Descripción de los efectos de Azure Policy](../governance/policy/concepts/effects.md)

- [Creación y administración de directivas para aplicar el cumplimiento](../governance/policy/tutorials/create-and-manage.md)

- [Información general sobre plantillas de Azure Resource Manager](../azure-resource-manager/templates/overview.md)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="75-securely-store-configuration-of-azure-resources"></a>7.5: Almacene de forma segura la configuración de los recursos de Azure

**Guía**: Use Azure DevOps para almacenar y administrar de forma segura el código, como directivas personalizadas de Azure y plantillas de Azure Resource Manager. Para acceder a los recursos que administra en Azure DevOps, puede conceder o denegar permisos a usuarios específicos, grupos de seguridad integrados o grupos definidos en Azure Active Directory (Azure AD) si se integran con Azure DevOps, o Active Directory si se integran con TFS.

- [Almacenamiento de código en Azure DevOps](https://docs.microsoft.com/azure/devops/repos/git/gitworkflow?view=azure-devops)

- [Acerca de los permisos y los grupos en Azure DevOps](/azure/devops/organizations/security/about-permissions)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="77-deploy-configuration-management-tools-for-azure-resources"></a>7.7: Implementación de herramientas de administración de configuración para recursos de Azure

**Guía**: Defina e implemente configuraciones de seguridad estándar para los recursos relacionados con Azure Monitor mediante Azure Policy. Utilice definiciones de Azure Policy personalizadas para auditar o aplicar la configuración de seguridad de los recursos relacionados con Azure Monitor. También puede usar definiciones de directivas integradas relacionadas con recursos concretos.

- [Configuración y administración de Azure Policy](../governance/policy/tutorials/create-and-manage.md)

- [Alias de Azure Policy](../governance/policy/concepts/definition-structure.md#aliases)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="79-implement-automated-configuration-monitoring-for-azure-resources"></a>7.9: Implementación de la supervisión de la configuración automatizada para los recursos de Azure

**Guía**: Utilice Azure Security Center para realizar análisis de base de referencia para los recursos relacionados con Azure Monitor.  Además, use Azure Policy para alertar y auditar las configuraciones de recursos de Azure.

- [Corrección de recomendaciones en Azure Security Center](../security-center/security-center-remediate-recommendations.md)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="711-manage-azure-secrets-securely"></a>7.11: Administre los secretos de Azure de forma segura

**Guía**: Utilice Managed Service Identity junto con Azure Key Vault para simplificar y proteger la administración de secretos para los recursos relacionados admitidos de Azure Monitor.

- [Integración con identidades administradas de Azure](../azure-app-configuration/howto-integrate-azure-managed-service-identity.md)

- [Servicios que admiten identidades administradas para recursos de Azure](../active-directory/managed-identities-azure-resources/services-support-managed-identities.md)

- [Creación de un almacén de claves](/azure/key-vault/quick-create-portal)

- [Cómo proporcionar la autenticación de Key Vault con una identidad administrada](/azure/key-vault/managed-identity)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="712-manage-identities-securely-and-automatically"></a>7.12: Administre las identidades de forma segura y automática

**Guía**: Use las identidades administradas para incorporar a los servicios de Azure una identidad administrada automáticamente en Azure AD. Las identidades administradas le permiten autenticarse en cualquier servicio que admita la autenticación de Azure AD, como los recursos de Azure Monitor, sin necesidad de utilizar credenciales en el código. 

- [Configuración de identidades administradas para recursos de Azure](../active-directory/managed-identities-azure-resources/qs-configure-portal-windows-vm.md)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="713-eliminate-unintended-credential-exposure"></a>7.13: Elimine la exposición de credenciales no intencionada

**Instrucciones**: Implemente el escáner de credenciales para identificar las credenciales en el código. El escáner de credenciales también fomenta el traslado de credenciales detectadas a ubicaciones más seguras, como Azure Key Vault.

- [Cómo configurar Credential Scanner](https://secdevtools.azurewebsites.net/helpcredscan.html)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

## <a name="malware-defense"></a>Defensa contra malware

*Para obtener más información, consulte [Azure Security Benchmark: defensa contra malware](/azure/security/benchmarks/security-control-malware-defense).*

### <a name="82-pre-scan-files-to-be-uploaded-to-non-compute-azure-resources"></a>8.2: Examine previamente los archivos que se van a cargar en recursos de Azure que no son de proceso

**Guía**: Microsoft Antimalware está habilitado en el host subyacente que admite servicios de Azure (por ejemplo, recursos relacionados con Azure Monitor), pero no se ejecuta en el contenido del usuario. 

Haga un análisis previo de los archivos que se van a cargar en los recursos relacionados con Azure Monitor (por ejemplo, el área de trabajo de Log Analytics).

Use la detección de amenazas de Azure Security Center para los servicios de datos para detectar malware cargado en cuentas de almacenamiento. 

- [Información sobre Microsoft Antimalware para Azure Cloud Services y Virtual Machines](../security/fundamentals/antimalware.md)

- [Descripción de la detección de amenazas de Azure Security Center para servicios de datos](/azure/security-center/security-center-alerts-data-services)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

## <a name="data-recovery"></a>Recuperación de datos

*Para más información, consulte [Azure Security Benchmark: recuperación de datos](/azure/security/benchmarks/security-control-data-recovery).*

### <a name="91-ensure-regular-automated-back-ups"></a>9.1: Garantía de copias de seguridad automáticas periódicas

**Guía**: Utilice Azure Resource Manager para exportar los recursos relacionados con Azure Monitor en una plantilla de notación de objetos JavaScript (JSON) que se puede usar como copia de seguridad para Azure Monitor y las configuraciones relacionadas.  Use Azure Automation para ejecutar automáticamente los scripts de copia de seguridad. 

- [Administración del área de trabajo de Log Analytics mediante las plantillas de Azure Resource Manager](/azure/azure-monitor/platform/template-workspace-configuration)

- [Exportación de uno y varios recursos a una plantilla en Azure Portal](../azure-resource-manager/templates/export-template-portal.md)

- [Información sobre Azure Automation](../automation/automation-intro.md)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="92-perform-complete-system-backups-and-backup-any-customer-managed-keys"></a>9.2: Copias de seguridad completas del sistema y copia de seguridad de las claves administradas por el cliente

**Guía**: Utilice Azure Resource Manager para exportar los recursos relacionados con Azure Monitor en una plantilla de notación de objetos JavaScript (JSON) que se puede usar como copia de seguridad para Azure Monitor y las configuraciones relacionadas.  Haga una copia de seguridad de las claves administradas por el cliente en Azure Key Vault si los recursos relacionados con Azure Monitor usan claves administradas por el cliente. 

- [Administración del área de trabajo de Log Analytics mediante las plantillas de Azure Resource Manager](/azure/azure-monitor/platform/template-workspace-configuration)

- [Exportación de uno y varios recursos a una plantilla en Azure Portal](../azure-resource-manager/templates/export-template-portal.md)

- [Creación de una copia de seguridad de las claves del almacén de claves en Azure](https://docs.microsoft.com/powershell/module/azurerm.keyvault/backup-azurekeyvaultkey?view=azurermps-6.13.0)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="93-validate-all-backups-including-customer-managed-keys"></a>9.3: Validación de todas las copias de seguridad, incluidas las claves administradas por el cliente

**Guía**: garantice la restauración periódica con copias de seguridad de los archivos de plantilla de Azure Resource Manager.  Pruebe la restauración de la copia de seguridad de las claves administradas por el cliente.

- [Administración del área de trabajo de Log Analytics mediante las plantillas de Azure Resource Manager](/azure/azure-monitor/platform/template-workspace-configuration)

- [Restauración de las claves del almacén de claves en Azure](https://docs.microsoft.com/powershell/module/azurerm.keyvault/restore-azurekeyvaultkey?view=azurermps-6.13.0)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="94-ensure-protection-of-backups-and-customer-managed-keys"></a>9.4: Garantía de la protección de las copias de seguridad y las claves administradas por el cliente

**Guía**: Use Azure DevOps para almacenar y administrar de forma segura el código, como directivas personalizadas de Azure y plantillas de Azure Resource Manager. Para proteger los recursos que administra en Azure DevOps, puede conceder o denegar permisos a usuarios específicos, grupos de seguridad integrados o grupos definidos en Azure Active Directory (Azure AD) si se integran con Azure DevOps, o en Active Directory si se integran con TFS.   Use el control de acceso basado en roles para proteger las claves administradas por el cliente. 

Además, habilite la eliminación temporal y la protección de purga en Key Vault para proteger las claves contra la eliminación accidental o malintencionada. Si Azure Storage se usa para almacenar las plantillas de Azure Resource Manager, habilite la eliminación temporal para guardar y recuperar los datos cuando se eliminen blobs o instantáneas de blobs. 

- [Almacenamiento de código en Azure DevOps](https://docs.microsoft.com/azure/devops/repos/git/gitworkflow?view=azure-devops)

- [Acerca de los permisos y los grupos en Azure DevOps](/azure/devops/organizations/security/about-permissions)

- [Procedimiento para habilitar la eliminación temporal y la protección de purga en Key Vault](https://docs.microsoft.com/azure/storage/blobs/storage-blob-soft-delete?tabs=azure-portal)

- [Eliminación temporal de blobs de Azure Storage](https://docs.microsoft.com/azure/storage/blobs/storage-blob-soft-delete?tabs=azure-portal)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

## <a name="incident-response"></a>Respuesta a los incidentes

*Para obtener más información, consulte [Azure Security Benchmark: Respuesta a los incidentes](/azure/security/benchmarks/security-control-incident-response).*

### <a name="101-create-an-incident-response-guide"></a>10.1: Creación de una guía de respuesta ante incidentes

**Instrucciones**: Cree una guía de respuesta a incidentes para su organización. Asegúrese de que haya planes de respuesta a incidentes escritos que definan todos los roles del personal, así como las fases de administración y gestión de los incidentes, desde la detección hasta la revisión posterior a la incidencia.

- [Guía para crear su propio proceso de respuesta a incidentes de seguridad](https://msrc-blog.microsoft.com/2019/07/01/inside-the-msrc-building-your-own-security-incident-response-process/)

- [Anatomía de un incidente del Centro de respuestas de seguridad de Microsoft](https://msrc-blog.microsoft.com/2019/06/27/inside-the-msrc-anatomy-of-a-ssirp-incident/)

- [Uso de la guía de control de incidentes de seguridad de equipos de NIST para la creación de su propio plan de respuesta a incidentes](https://csrc.nist.gov/publications/detail/sp/800-61/rev-2/final)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="102-create-an-incident-scoring-and-prioritization-procedure"></a>10.2: Creación de un procedimiento de priorización y puntuación de incidentes

**Instrucciones**: Security Center asigna una gravedad a cada alerta para ayudarle a priorizar aquellas que se deben investigar en primer lugar. La gravedad se basa en la confianza que tiene Security Center en la búsqueda o en el análisis utilizados para emitir la alerta, así como en el nivel de confianza de que ha habido un intento malintencionado detrás de la actividad que ha provocado la alerta.

Además, marque claramente las suscripciones (por ejemplo, producción, no producción) con etiquetas y cree un sistema de nomenclatura para identificar y clasificar claramente los recursos de Azure, especialmente los que procesan datos confidenciales.  Es su responsabilidad asignar prioridades a la corrección de las alertas en función de la importancia de los recursos y el entorno de Azure donde se produjo el incidente.

- [Alertas de seguridad en el Centro de seguridad de Azure](../security-center/security-center-alerts-overview.md)

- [Uso de etiquetas para organizar los recursos de Azure](/azure/azure-resource-manager/resource-group-using-tags)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="103-test-security-response-procedures"></a>10.3: Prueba de los procedimientos de respuesta de seguridad

**Guía**: Realice ejercicios para probar las funcionalidades de respuesta a los incidentes de los sistemas periódicamente para ayudar a proteger los recursos de Azure. Identifique puntos débiles y brechas y revise el plan según sea necesario.

- [Publicación de NIST: Guía para probar, entrenar y ejecutar programas para planes y funcionalidades de TI](https://csrc.nist.gov/publications/detail/sp/800-84/final)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="104-provide-security-incident-contact-details-and-configure-alert-notifications-for-security-incidents"></a>10.4: Provisión de detalles de contacto de incidentes de seguridad y configuración de notificaciones de alerta para incidentes de seguridad

**Instrucciones**: La información de contacto del incidente de seguridad la utilizará Microsoft para ponerse en contacto con usted si Microsoft Security Response Center (MSRC) detecta que un tercero no autorizado o ilegal ha accedido a los datos. Revise los incidentes después del hecho para asegurarse de que se resuelven los problemas.

- [Establecimiento del contacto de seguridad de Azure Security Center](../security-center/security-center-provide-security-contact-details.md)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="105-incorporate-security-alerts-into-your-incident-response-system"></a>10.5: Incorporación de alertas de seguridad en el sistema de respuesta a incidentes

**Guía**: Exporte las alertas y recomendaciones de Azure Security Center y mediante la característica de exportación continua para ayudar a identificar los riesgos para los recursos de Azure. La exportación continua le permite exportar alertas y recomendaciones de forma manual o continua. Puede usar el conector de datos de Azure Security Center para transmitir las alertas a Azure Sentinel. 

- [Configuración de la exportación continua](../security-center/continuous-export.md) 

- [Transmisión de alertas a Azure Sentinel](../sentinel/connect-azure-security-center.md)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="106-automate-the-response-to-security-alerts"></a>10.6: Automatización de la respuesta a las alertas de seguridad

**Guía**: Use la característica Automatización de flujo de trabajo de Azure Security Center para desencadenar automáticamente las respuestas a través de Logic Apps en las alertas y recomendaciones de seguridad para proteger los recursos de Azure.

- [Configuración de la automatización de flujo de trabajo y Logic Apps](../security-center/workflow-automation.md)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

## <a name="penetration-tests-and-red-team-exercises"></a>Pruebas de penetración y ejercicios del equipo rojo

*Para obtener más información, consulte [Azure Security Benchmark: Pruebas de penetración y ejercicios del equipo rojo](/azure/security/benchmarks/security-control-penetration-tests-red-team-exercises).*

### <a name="111-conduct-regular-penetration-testing-of-your-azure-resources-and-ensure-remediation-of-all-critical-security-findings"></a>11.1: Realice pruebas de penetración periódicas de los recursos de Azure y asegúrese de corregir todos los resultados de seguridad críticos

**Guía**: Siga las reglas de compromiso de Microsoft para asegurarse de que las pruebas de penetración no infrinjan las directivas de Microsoft. Use la estrategia de Microsoft y la ejecución de pruebas de penetración de sitios activos y ataques simulados en la infraestructura en la nube, los servicios y las aplicaciones administrados por Microsoft.

- [Reglas de interacción de las pruebas de penetración](https://www.microsoft.com/msrc/pentest-rules-of-engagement?rtc=1)

- [Equipo rojo de Microsoft Cloud](https://gallery.technet.microsoft.com/Cloud-Red-Teaming-b837392e)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Compartido

## <a name="next-steps"></a>Pasos siguientes

- Consulte la [prueba comparativa de seguridad de Azure](/azure/security/benchmarks/overview).
- Obtenga más información sobre las [líneas de base de seguridad de Azure](/azure/security/benchmarks/security-baselines-overview).
