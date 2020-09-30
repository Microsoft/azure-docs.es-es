---
title: Línea de base de seguridad de Azure para Security Center
description: La línea de base de seguridad de Security Center proporciona una guía con procedimientos y recursos para implementar las recomendaciones de seguridad especificadas en Azure Security Benchmark.
author: msmbaldwin
ms.service: security-center
ms.topic: conceptual
ms.date: 08/05/2020
ms.author: mbaldwin
ms.custom: subject-security-benchmark
ms.openlocfilehash: f41dfa0889b7f0c34486eee5d16362f4c3c7360d
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90894595"
---
# <a name="azure-security-baseline-for-security-center"></a>Línea de base de seguridad de Azure para Security Center

Esta línea de base de seguridad aplica la guía de [Azure Security Benchmark](../security/benchmarks/overview.md) a Azure Security Center. Azure Security Benchmark proporciona recomendaciones sobre cómo puede proteger sus soluciones de nube en Azure. El contenido se agrupa por los **controles de seguridad** definidos en Azure Security Benchmark y la guía relacionada que se aplica a Azure Security Center. Se han excluido los **controles** no aplicables a Azure Security Center. Para ver cómo se asigna Azure Security Center por completo a Azure Security Benchmark, consulte el [archivo de asignación de línea de base de seguridad de Azure Security Center completo](https://github.com/MicrosoftDocs/SecurityBenchmarks/tree/master/Azure%20Offer%20Security%20Baselines).

## <a name="network-security"></a>Seguridad de las redes

*Para obtener más información, consulte [Azure Security Benchmark: Seguridad de redes](/azure/security/benchmarks/security-control-network-security).*

### <a name="11-protect-azure-resources-within-virtual-networks"></a>1.1: Protección de los recursos de Azure dentro de las redes virtuales

**Guía**: Azure Security Center es una oferta básica de Azure. No se pueden asociar redes virtuales, subredes ni grupos de seguridad de red directamente a Security Center. Si habilita la recopilación de datos para los recursos de proceso y Security Center almacena después los datos que recopila a través de un área de trabajo Log Analytics, podrá configurar esa área de trabajo con el objetivo de usar Private Link para acceder a los datos de dicha área de trabajo a través de un punto de conexión privado de la red virtual. Además, al usar la recopilación de datos, Security Center se basará en el agente de Log Analytics que se implementa en los servidores para recopilar datos de seguridad y proporcionar protección a estos recursos de proceso. El agente de Log Analytics requiere que se abran determinados puertos y protocolos para que funcione correctamente con Security Center. Bloquee las redes para permitir solo estos puertos y protocolos necesarios y agregue únicamente las reglas adicionales que la aplicación necesita para funcionar correctamente a través de grupos de seguridad de red.

- [Recopilación de datos en Azure Security Center](security-center-enable-data-collection.md)

- [Filtrado del tráfico de red con un grupo de seguridad de red](../virtual-network/tutorial-filter-network-traffic.md)

- [Requisitos de firewall para usar el agente de Log Analytics](../azure-monitor/platform/log-analytics-agent.md#firewall-requirements)

- [¿Qué es Azure Private Link?](../private-link/private-link-overview.md) 

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="19-maintain-standard-security-configurations-for-network-devices"></a>1.9: Mantenga las configuraciones de seguridad estándar para dispositivos de red

**Guía**: La oferta Azure Security Center no se integra directamente con una red virtual, pero puede recopilar datos de servidores configurados con el agente de Log Analytics que se implementan en las redes. Los servidores que están configurados para enviar datos a Security Center requieren que determinados puertos y protocolos puedan comunicarse correctamente. Defina e implemente configuraciones de seguridad estándar para esos recursos de red con Azure Policy.

También puede usar Azure Blueprints para simplificar las implementaciones de Azure a gran escala empaquetando artefactos de entorno clave, como plantillas de Azure Resource Manager, asignaciones de roles y asignaciones de Azure Policy, en una única definición de plano técnico. Puede aplicar el plano técnico a nuevas suscripciones para implementar configuraciones de Security Center y recursos de red relacionados de forma coherente y segura.

- [Recopilación de datos en Azure Security Center](security-center-enable-data-collection.md)

- [Requisitos de firewall para usar el agente de Log Analytics](../azure-monitor/platform/log-analytics-agent.md#firewall-requirements)

- [Configuración y administración de Azure Policy](../governance/policy/tutorials/create-and-manage.md) 

- [Ejemplos de Azure Policy para redes](../governance/policy/samples/built-in-policies.md#network)

- [Creación de un plano técnico de Azure](../governance/blueprints/create-blueprint-portal.md)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="110-document-traffic-configuration-rules"></a>1.10: Documente las reglas de configuración de tráfico

**Guía**: La oferta Azure Security Center no se integra directamente con una red virtual, pero puede recopilar datos de servidores configurados con el agente de Log Analytics que se implementan en las redes. Los servidores que están configurados para enviar datos a Security Center requieren que determinados puertos y protocolos puedan comunicarse correctamente. Defina e implemente configuraciones de seguridad estándar para esos recursos de red con Azure Policy.

Use etiquetas de recursos para los grupos de seguridad de red y otros recursos, como los servidores de las redes que están configurados para enviar registros de seguridad a Azure Security Center. En el caso de las reglas de grupo de seguridad de red individuales, use el campo "Descripción" para documentar las reglas que permiten con origen y destino en una red. 

Use cualquiera de las definiciones de Azure Policy integradas relacionadas con el etiquetado, como "Requerir etiqueta y su valor", para asegurarse de que todos los recursos se creen con etiquetas y para notificarle los recursos no etiquetados existentes.

Puede usar Azure PowerShell o la CLI de Azure para buscar o realizar acciones en los recursos en función de sus etiquetas. 

- [Recopilación de datos en Azure Security Center](security-center-enable-data-collection.md)

- [Requisitos de firewall para usar el agente de Log Analytics](../azure-monitor/platform/log-analytics-agent.md#firewall-requirements)

- [Creación y uso de etiquetas](/azure/azure-resource-manager/resource-group-using-tags) 

- [Creación de una red virtual de Azure](../virtual-network/quick-create-portal.md) 

- [Filtrado del tráfico de red con reglas de grupos de seguridad de red](../virtual-network/tutorial-filter-network-traffic.md)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="111-use-automated-tools-to-monitor-network-resource-configurations-and-detect-changes"></a>1.11: Use herramientas automatizadas para supervisar las configuraciones de recursos de red y detectar cambios

**Guía**: use el registro de actividad de Azure para supervisar las configuraciones de los recursos de red y detectar cambios en los recursos de red relacionados con Azure Security Center. Cree alertas en Azure Monitor para recibir una notificación cuando se produzcan cambios en recursos críticos.

- [Visualización y recuperación de eventos del registro de actividad de Azure](/azure/azure-monitor/platform/activity-log-view) 

- [Creación de alertas en Azure Monitor](../azure-monitor/platform/alerts-activity-log.md)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

## <a name="logging-and-monitoring"></a>Registro y supervisión

*Para obtener más información, consulte [Azure Security Benchmark: registro y supervisión](/azure/security/benchmarks/security-control-logging-monitoring).*

### <a name="22-configure-central-security-log-management"></a>2.2: Configuración de la administración central de registros de seguridad

**Guía**: agregue los datos de seguridad que ha generado Azure Security Center y sus orígenes conectados usando un área de trabajo central de Log Analytics. 

Configure la recopilación de datos de Security Center para enviar eventos y datos de seguridad desde los recursos de proceso de Azure conectados a un área de trabajo central de Log Analytics. Además de la recopilación de datos, use la característica de exportación continua para enviar las alertas de seguridad y recomendaciones que genera Security Center al área de trabajo central de Log Analytics. En Azure Monitor, puede consultar y realizar análisis en los datos de seguridad generados a partir de Security Center y los recursos de Azure conectados. 

Como alternativa, puede enviar los datos generados por Security Center a Azure Sentinel o a una herramienta SIEM de terceros.

- [Exportar alertas y recomendaciones de seguridad (versión preliminar)](continuous-export.md)

- [Recopilación de datos en Azure Security Center](security-center-enable-data-collection.md)

- [Incorporación de Azure Sentinel](../sentinel/quickstart-onboard.md) 

- [Recopilación de registros y métricas de plataforma con Azure Monitor](../azure-monitor/platform/diagnostic-settings.md) 

- [Recopilación de registros de host internos de máquina virtual de Azure con Azure Monitor](../azure-monitor/learn/quick-collect-azurevm.md)

- [Introducción a Azure Monitor e integración con herramienta SIEM de terceros](https://azure.microsoft.com/blog/use-azure-monitor-to-integrate-with-siem-tools/)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="23-enable-audit-logging-for-azure-resources"></a>2.3: Habilitación del registro de auditoría para recursos de Azure

**Guía**: Los registros de actividad de Azure Monitor están disponibles de forma automática y contienen todas las operaciones de escritura para el recurso, como Azure Security Center, incluidas las operaciones que se realizaron, quién inició la operación y cuándo. Envíe los registros de actividad de Azure a un área de trabajo de Log Analytics para consolidarlos y aumentar la retención.

- [Recopilación de registros y métricas de plataforma con Azure Monitor](../azure-monitor/platform/diagnostic-settings.md) 

- [Descripción del registro y de los distintos tipos de registro de Azure](../azure-monitor/platform/platform-logs-overview.md)

- [Envío de registros de actividad a un área de trabajo de Log Analytics](../azure-monitor/platform/activity-log.md#send-to-log-analytics-workspace)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="25-configure-security-log-storage-retention"></a>2.5: Configuración de la retención del almacenamiento de registros de seguridad

**Guía**: En Azure Monitor, establezca el período de retención del área de trabajo de Log Analytics de acuerdo con la normativa de cumplimiento de su organización. Use cuentas de Azure Storage para el almacenamiento de archivos y a largo plazo. 

- [Cambio del período de retención de datos en Log Analytics](../azure-monitor/platform/manage-cost-storage.md#change-the-data-retention-period) 

- [Configuración de la directiva de retención para los registros de la cuenta de Azure Storage](../storage/common/storage-monitor-storage-account.md#configure-logging)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="26-monitor-and-review-logs"></a>2.6: Supervisión y revisión de registros

**Instrucciones**: Analice y supervise los registros generados por Azure Security Center y sus orígenes conectados para detectar comportamientos anómalos y revise los resultados con regularidad. Use Azure Monitor y un área de trabajo de Log Analytics para revisar los registros y realizar consultas en los datos de dichos registros.

También puede habilitar e incorporar datos en Azure Sentinel o en una herramienta SIEM de terceros. 

- [Incorporación de Azure Sentinel](../sentinel/quickstart-onboard.md) 

- [Introducción a las consultas de Log Analytics](../azure-monitor/log-query/get-started-portal.md) 

- [Procedimiento para realizar consultas personalizadas en Azure Monitor](../azure-monitor/log-query/get-started-queries.md)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="27-enable-alerts-for-anomalous-activities"></a>2.7: Habilitación de alertas para actividades anómalas

**Guía**: configure alertas de registro de Azure Monitor para consultar actividades anómalas o no deseadas que se agregan al registro de actividad o a los datos generados por Azure Security Center. Configure los grupos de acciones para que se notifique a su organización y pueda tomar medidas si se inicia una alerta de registro de actividad anómala. Use la característica de automatización de flujos de trabajo de Security Center para desencadenar aplicaciones lógicas en alertas de seguridad y recomendaciones. Los flujos de trabajo de Security Center se pueden usar para notificar a los usuarios la respuesta a los incidentes o para tomar medidas con el fin de corregir errores en los recursos en función de la información de las alertas.

También puede habilitar e incorporar los datos relacionados con Azure Security Center (o los generados por este servicio) en Azure Sentinel. Azure Sentinel es compatible con los cuadernos de estrategias, que permiten respuestas automatizadas a los problemas relacionados con la seguridad.

- [Automatización de flujos de trabajo Azure Security Center](workflow-automation.md)

- [Administración de alertas de seguridad en Azure Security Center](security-center-managing-and-responding-alerts.md) 

- [Alertas sobre datos de registro de Log Analytics](../azure-monitor/learn/tutorial-response.md)

- [Configuración de respuestas automatizadas frente a amenazas en Azure Sentinel](../sentinel/tutorial-respond-threats-playbook.md)

- [Alertas de registro en Azure Monitor](../azure-monitor/platform/alerts-unified-log.md)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

## <a name="identity-and-access-control"></a>Control de identidades y acceso

*Para obtener más información, consulte [Azure Security Benchmark: Identidad y control de acceso](/azure/security/benchmarks/security-control-identity-access-control).*

### <a name="31-maintain-an-inventory-of-administrative-accounts"></a>3.1: Mantenga un inventario de cuentas administrativas

**Guía**: El control de acceso basado en roles de Azure (RBAC de Azure) permite administrar el acceso a los recursos de Azure a través de las asignaciones de roles. Puede asignar estos roles a usuarios, grupos de entidades de servicio e identidades administradas. Hay roles integrados predefinidos para determinados recursos, y estos roles se pueden inventariar o consultar mediante herramientas como la CLI de Azure, Azure PowerShell o el Azure Portal. Azure Security Center tiene los roles integrados de "Lector de seguridad" o "Administrador de seguridad", que permiten a los usuarios leer o actualizar directivas de seguridad y descartar alertas y recomendaciones.

- [Permisos en Azure Security Center](security-center-permissions.md)

- [Obtención de un rol de directorio en Azure AD con PowerShell](https://docs.microsoft.com/powershell/module/azuread/get-azureaddirectoryrole?view=azureadps-2.0) 

- [Obtención de los miembros de un rol de directorio en Azure AD con PowerShell](https://docs.microsoft.com/powershell/module/azuread/get-azureaddirectoryrolemember?view=azureadps-2.0)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="33-use-dedicated-administrative-accounts"></a>3.3: Use cuentas administrativas dedicadas

**Guía**: cree procedimientos operativos estándar en torno al uso de cuentas administrativas dedicadas para la plataforma Azure o específicas de la oferta Azure Security Center. Use la administración de acceso e identidades de Azure Security Center para supervisar el número de cuentas administrativas en Azure Active Directory. Security Center también tiene roles integrados de "Administrador de seguridad", que permiten a los usuarios actualizar las directivas de seguridad y descartar alertas y recomendaciones; asegúrese de revisar y reconciliar de forma periódica los usuarios que tengan esta asignación de roles.

Además, para ayudarle a realizar un seguimiento de las cuentas administrativas dedicadas, puede seguir las recomendaciones de Azure Security Center o las directivas integradas de Azure Policy, por ejemplo:

- Debe haber más de un propietario asignado a su suscripción
- Las cuentas en desuso con permisos de propietario deben quitarse de la suscripción
- Las cuentas externas con permisos de propietario deben quitarse de la suscripción

- [Permisos en Azure Security Center](security-center-permissions.md)

- [Uso de Azure Security Center para supervisar la identidad y el acceso (versión preliminar)](security-center-identity-access.md)

- [Uso de Azure Policy](../governance/policy/tutorials/create-and-manage.md)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="34-use-single-sign-on-sso-with-azure-active-directory"></a>3.4: Use el inicio de sesión único (SSO) con Azure Active Directory

**Instrucciones**: Siempre que sea posible, use el SSO de Azure Active Directory en lugar de configurar credenciales independientes individuales por servicio. Use las recomendaciones de identidades y acceso de Azure Security Center.

- [Descripción del SSO con Azure AD](../active-directory/manage-apps/what-is-single-sign-on.md)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="35-use-multi-factor-authentication-for-all-azure-active-directory-based-access"></a>3.5: Use la autenticación multifactor para todo el acceso basado en Azure Active Directory

**Guía**: habilite la MFA de Azure Active Directory para acceder Azure Security Center y Azure Portal, y siga las recomendaciones de identidad y acceso de Security Center. 

- [Procedimiento para habilitar la MFA en Azure](../active-directory/authentication/howto-mfa-getstarted.md) 

- [Supervisión de la identidad y el acceso en Azure Security Center](security-center-identity-access.md)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="36-use-dedicated-machines-privileged-access-workstations-for-all-administrative-tasks"></a>3.6: Use máquinas dedicadas (estaciones de trabajo de acceso con privilegios) para todas las tareas administrativas

**Instrucciones**: use una estación de trabajo segura y administrada por Azure (también conocida como una estación de trabajo de acceso con privilegios o PAW) para las tareas administrativas que requieren privilegios elevados.

- [Descripción de las estaciones de trabajo seguras administradas por Azure](../active-directory/devices/concept-azure-managed-workstation.md)

- [Cómo habilitar MFA de Azure AD](../active-directory/authentication/howto-mfa-getstarted.md)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="37-log-and-alert-on-suspicious-activities-from-administrative-accounts"></a>3.7: Registro y alerta de actividades sospechosas desde cuentas administrativas

**Guía**: use la característica de supervisión y los informes de seguridad de Azure Active Directory para detectar cuando se producen actividades sospechosas o no seguras en el entorno. Use Azure Security Center para supervisar la actividad de identidad y acceso.

- [Procedimiento para identificar usuarios de Azure AD marcados por una actividad de riesgo](/azure/active-directory/reports-monitoring/concept-user-at-risk) 

- [Supervisión de la actividad de identidad y acceso de los usuarios en Azure Security Center](security-center-identity-access.md)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="38-manage-azure-resources-from-only-approved-locations"></a>3.8: Administre los recursos de Azure solo desde ubicaciones aprobadas

**Guía**: use ubicaciones con nombre de Azure AD para permitir el acceso solo desde agrupaciones lógicas específicas de intervalos de direcciones IP o países o regiones. 

- [Configuración de ubicaciones con nombre de Azure AD](../active-directory/reports-monitoring/quickstart-configure-named-locations.md)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="39-use-azure-active-directory"></a>3.9: Uso de Azure Active Directory

**Guía**: use Azure Active Directory (Azure AD) como sistema central de autenticación y autorización al usar Azure Security Center. Azure AD protege los datos mediante un cifrado seguro para los datos en reposo y en tránsito. Azure AD también cifra con sal, convierte en hash y almacena de forma segura las credenciales de los usuarios. Azure Security Center tiene roles integrados que se pueden asignar, como "Administrador de seguridad", que permite a los usuarios actualizar directivas de seguridad y descartar alertas y recomendaciones.

- [Permisos en Azure Security Center](security-center-permissions.md)

- [Procedimiento para crear y configurar una instancia de Azure AD](../active-directory/fundamentals/active-directory-access-create-new-tenant.md)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="310-regularly-review-and-reconcile-user-access"></a>3.10: Revise y concilie regularmente el acceso de los usuarios

**Guía**: Azure Active Directory proporciona registros para ayudar a descubrir cuentas obsoletas. Además, use las revisiones de acceso e identidades de Azure AD para administrar de forma eficiente las pertenencias a grupos, el acceso a las aplicaciones empresariales y las asignaciones de roles. El acceso de los usuarios relacionado con Azure Security Center se puede revisar de forma periódica para asegurarse de que solo las personas adecuadas tengan acceso continuado. 

- [Descripción de los informes de Azure AD](/azure/active-directory/reports-monitoring/) 

- [Procedimiento para usar las revisiones de acceso e identidades de Azure AD](../active-directory/governance/access-reviews-overview.md)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="311-monitor-attempts-to-access-deactivated-credentials"></a>3.11: Supervisión de los intentos de acceso a credenciales desactivadas

**Guía**: tiene acceso a los orígenes de registro de eventos de actividad de inicio de sesión, auditoría y riesgo de Azure AD, que permiten la integración con cualquier herramienta SIEM o de supervisión. 

Para simplificar este proceso, cree una configuración de diagnóstico para las cuentas de usuario de Azure AD y envíe los registros de auditoría y los registros de inicio de sesión a un área de trabajo de Log Analytics. Puede configurar las alertas deseadas en el área de trabajo de Log Analytics.  

- [Integración de los registros de actividad de Azure en Azure Monitor](/azure/active-directory/reports-monitoring/howto-integrate-activity-logs-with-log-analytics)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="312-alert-on-account-login-behavior-deviation"></a>3.12: Alerte de las desviaciones de comportamiento en los inicios de sesión de las cuentas 

**Guía**: use las características de Azure Active Directory Identity Protection para configurar las respuestas automatizadas a las acciones sospechosas detectadas relacionadas con las identidades de los usuarios. También puede hacer que Azure Sentinel ingiera los datos para investigarlos más. 

- [Visualización de los inicios de sesión de riesgo de Azure AD](/azure/active-directory/reports-monitoring/concept-risky-sign-ins) 

- [Configuración y habilitación de las directivas de riesgo de protección de identidad](../active-directory/identity-protection/howto-identity-protection-configure-risk-policies.md) 

- [Incorporación de Azure Sentinel](../sentinel/quickstart-onboard.md)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

## <a name="data-protection"></a>Protección de los datos

*Para obtener más información, consulte [Azure Security Benchmark: Protección de datos](/azure/security/benchmarks/security-control-data-protection).*

### <a name="41-maintain-an-inventory-of-sensitive-information"></a>4.1: Mantenimiento de un inventario de información confidencial

**Guía**: use etiquetas para ayudar a realizar un seguimiento de los recursos de Azure como el área de trabajo de Log Analytics, que almacena información de seguridad confidencial de Azure Security Center.

- [Creación y uso de etiquetas](/azure/azure-resource-manager/resource-group-using-tags)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="42-isolate-systems-storing-or-processing-sensitive-information"></a>4.2: Aislamiento de los sistemas que almacenan o procesan información confidencial

**Guía**: Implemente el aislamiento mediante suscripciones independientes y grupos de administración para dominios de seguridad individuales, como el tipo de entorno y el nivel de confidencialidad de los datos. Puede restringir el nivel de acceso a los recursos de Azure que necesitan sus aplicaciones y entornos empresariales. Puede controlar el acceso a los recursos de Azure a través de RBAC de Azure.

De forma predeterminada, los datos de Azure Security Center se almacenan en el servicio back-end de Security Center. Si su organización ha agregado requisitos para almacenar estos datos en sus propios recursos, puede configurar un área de trabajo de Log Analytics para almacenar datos, alertas y recomendaciones de Security Center. Al usar su propia área de trabajo, puede agregar más separación configurando diferentes áreas de trabajo según el entorno en el que se originaron los datos.

- [Exportar alertas y recomendaciones de seguridad (versión preliminar)](continuous-export.md)

- [Recopilación de datos en Azure Security Center](security-center-enable-data-collection.md)

- [Creación de suscripciones adicionales de Azure](/azure/billing/billing-create-subscription) 

- [Creación de grupos de administración](../governance/management-groups/create.md) 

- [Creación y uso de etiquetas](/azure/azure-resource-manager/resource-group-using-tags)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="44-encrypt-all-sensitive-information-in-transit"></a>4.4: Cifrado de toda la información confidencial en tránsito

**Instrucciones**: Cifre toda la información confidencial en tránsito. Asegúrese de que los clientes que se conectan a los recursos de Azure pueden negociar TLS 1.2 o superior. Todas las máquinas virtuales configuradas con el agente de Log Analytics y para enviar datos a Azure Security Center deben configurarse para usar TLS 1.2.

Siga las recomendaciones de Azure Security Center para el cifrado en reposo y el cifrado en tránsito, si procede. 

- [Envío de datos de forma segura a Log Analytics](../azure-monitor/platform/data-security.md#sending-data-securely-using-tls-12)

- [Descripción del cifrado en tránsito con Azure](../security/fundamentals/encryption-overview.md#encryption-of-data-in-transit)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Compartido

### <a name="46-use-azure-rbac-to-control-access-to-resources"></a>4.6: Uso de RBAC de Azure para controlar el acceso a los recursos 

**Guía**: Use el control de acceso basado en roles de Azure (RBAC de Azure) para administrar el acceso a los recursos y los datos relacionados con Azure Security Center. Azure Security Center tiene los roles integrados de "Lector de seguridad" o "Administrador de seguridad", que permiten a los usuarios leer o actualizar directivas de seguridad y descartar alertas y recomendaciones. El área de trabajo de Log Analytics que almacena los datos recopilados por Security Center también tiene roles integrados que puede asignar, como "Lector de Log Analytics", "Colaborador de Log Analytics" y otros. Asigne el rol de menos permisos que los usuarios necesiten para realizar sus tareas. Por ejemplo, asigne el rol Lector a los usuarios que solo necesiten ver información sobre el estado de seguridad de los recursos, pero no llevar a cabo acciones como aplicar recomendaciones o editar directivas.

- [Permisos para el área de trabajo de Azure Log Analytics](../role-based-access-control/built-in-roles.md#log-analytics-reader)

- [Permisos en Azure Security Center](security-center-permissions.md)

- [Configuración de Azure RBAC](../role-based-access-control/role-assignments-portal.md)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="48-encrypt-sensitive-information-at-rest"></a>4.8: Cifrado de información confidencial en reposo

**Guía**: Azure Security Center usa un área de trabajo de Log Analytics configurada para almacenar los datos, las alertas y las recomendaciones que genera. Configure una clave administrada por el cliente (CMK) para el área de trabajo que ha configurado para la recopilación de datos de Security Centers. CMK permite cifrar todos los datos guardados o enviados en el área de trabajo con una clave de Azure Key Vault que usted mismo puede crear y que será de su propiedad. 

- [Clave administrada por el cliente de Azure Monitor](../azure-monitor/platform/customer-managed-keys.md)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="49-log-and-alert-on-changes-to-critical-azure-resources"></a>4.9: Registro y alerta de cambios en los recursos críticos de Azure

**Guía**: Use Azure Monitor para crear alertas cuando se realicen cambios en recursos críticos de Azure relacionados con Azure Security Center. Estos cambios pueden incluir cualquier acción que modifique configuraciones relacionadas con Security Center, como la deshabilitación de alertas o recomendaciones, o la actualización o eliminación de almacenes de datos.

- [Creación de alertas para los eventos del registro de actividad de Azure](../azure-monitor/platform/alerts-activity-log.md)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

## <a name="vulnerability-management"></a>Administración de vulnerabilidades

*Para obtener más información, consulte [Azure Security Benchmark: Administración de vulnerabilidades](/azure/security/benchmarks/security-control-vulnerability-management).*

### <a name="55-use-a-risk-rating-process-to-prioritize-the-remediation-of-discovered-vulnerabilities"></a>5.5: Use un proceso de clasificación de riesgos para priorizar la corrección de las vulnerabilidades detectadas

**Instrucciones**: Use un programa de puntuación de riesgos común (por ejemplo, Common Vulnerability Scoring System) o la clasificación de riesgos predeterminada proporcionada por su herramienta de análisis de terceros.

- [Publicación de NIST: sistema de puntuación común de vulnerabilidades](https://www.nist.gov/publications/common-vulnerability-scoring-system)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

## <a name="inventory-and-asset-management"></a>Inventario y administración de recursos

*Para obtener más información, consulte [Azure Security Benchmark: Administración de recursos y del inventario](/azure/security/benchmarks/security-control-inventory-asset-management).*

### <a name="61-use-automated-asset-discovery-solution"></a>6.1: Uso de la solución de detección de recursos automatizada

**Guía**: use Azure Resource Graph para consultar y detectar todos los recursos relacionados con Azure Security Center en las suscripciones. Asegúrese de que tiene los permisos adecuados (lectura) en el inquilino y enumere todas las suscripciones de Azure para detectar los recursos de Security Center. 

- [Creación de consultas con Azure Resource Graph Explorer](../governance/resource-graph/first-query-portal.md) 

- [Visualización de las suscripciones de Azure](https://docs.microsoft.com/powershell/module/az.accounts/get-azsubscription?view=azps-3.0.0) 

- [Descripción de Azure RBAC](../role-based-access-control/overview.md)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="62-maintain-asset-metadata"></a>6.2: Mantenimiento de metadatos de recursos

**Guía**: use etiquetas para ayudar a realizar un seguimiento de los recursos de Azure como el área de trabajo de Log Analytics, que almacena información de seguridad confidencial de Azure Security Center.

- [Creación y uso de etiquetas](/azure/azure-resource-manager/resource-group-using-tags)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="63-delete-unauthorized-azure-resources"></a>6.3: Eliminación de recursos de Azure no autorizados

**Guía**: use el etiquetado, los grupos de administración y las suscripciones independientes, si procede, para organizar y realizar un seguimiento de los recursos de Azure Security Center. Concilie el inventario periódicamente y asegúrese de que los recursos no autorizados se eliminan de la suscripción de manera oportuna.

Además, use Azure Policy para establecer restricciones sobre el tipo de recursos que se pueden crear en las suscripciones del cliente con las siguientes definiciones de directiva integradas:

- Tipos de recursos no permitidos
- Tipos de recursos permitidos

- [Creación de suscripciones adicionales de Azure](/azure/billing/billing-create-subscription)

- [Creación de grupos de administración](../governance/management-groups/create.md)

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

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="67-remove-unapproved-azure-resources-and-software-applications"></a>6.7: Eliminación de aplicaciones de software y recursos de Azure no aprobadas

**Guía**: quite los recursos de Azure relacionados con Azure Security Center cuando ya no sean necesarios como parte del inventario y del proceso de revisión de su organización.

- [Eliminación de grupos de recursos y recursos de Azure](../azure-resource-manager/management/delete-resource-group.md)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="69-use-only-approved-azure-services"></a>6.9: Uso exclusivo de servicios de Azure aprobados

**Guía**: use Azure Policy para establecer restricciones sobre el tipo de recursos que se pueden crear en las suscripciones con las siguientes definiciones de directiva integradas:

- Tipos de recursos no permitidos
- Tipos de recursos permitidos

- [Configuración y administración de Azure Policy](../governance/policy/tutorials/create-and-manage.md)

- [Denegación de un tipo de recurso específico con Azure Policy](/azure/governance/policy/samples/not-allowed-resource-types)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="611-limit-users-ability-to-interact-with-azure-resource-manager"></a>6.11: Limitación de la capacidad de los usuarios para interactuar con Azure Resource Manager

**Instrucciones**: Configure el acceso condicional de Azure para limitar la capacidad de los usuarios de interactuar con Azure Resource Manager configurando "Bloquear acceso" en la aplicación Microsoft Azure Management.

- [Configuración del acceso condicional para bloquear el acceso a Azure Resource Manager](../role-based-access-control/conditional-access-azure-management.md)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

## <a name="secure-configuration"></a>Configuración segura

*Para obtener más información, consulte [Azure Security Benchmark: Configuración segura](/azure/security/benchmarks/security-control-secure-configuration).*

### <a name="71-establish-secure-configurations-for-all-azure-resources"></a>7.1: Establezca configuraciones seguras para todos los recursos de Azure

**Guía**: defina e implemente configuraciones de seguridad estándar para Azure Security Center y su área de trabajo conectada a través de Azure Policy. Use los alias de Azure Policy en los espacios de nombres "Microsoft.OperationalInsights" y "Microsoft.Security" para crear definiciones de Azure Policy personalizadas con el fin de auditar o aplicar la configuración de Security Center y su área de trabajo de Log Analytics.

- [Visualización de los alias de Azure Policy disponibles](https://docs.microsoft.com/powershell/module/az.resources/get-azpolicyalias?view=azps-3.3.0)

- [Configuración y administración de Azure Policy](../governance/policy/tutorials/create-and-manage.md)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="73-maintain-secure-azure-resource-configurations"></a>7.3: Mantenga configuraciones de recursos de Azure seguras

**Instrucciones**: Utilice las directivas "deny" y "deploy if not exist" de Azure Policy para aplicar una configuración segura en los recursos de Azure. Además, puede usar plantillas de Azure Resource Manager para mantener la configuración de seguridad de los recursos de Azure que requiere su organización. 

- [Descripción de los efectos de Azure Policy](../governance/policy/concepts/effects.md) 

- [Creación y administración de directivas para aplicar el cumplimiento](../governance/policy/tutorials/create-and-manage.md) 

- [Información general sobre plantillas de Azure Resource Manager](../azure-resource-manager/templates/overview.md)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="75-securely-store-configuration-of-azure-resources"></a>7.5: Almacene de forma segura la configuración de los recursos de Azure

**Guía**: Use Azure DevOps para almacenar y administrar de forma segura el código, como definiciones de Azure Policy personalizadas, plantillas de Azure Resource Manager y scripts de Desired State Configuration. Para acceder a los recursos que administra en Azure DevOps, puede conceder o denegar permisos a usuarios específicos, grupos de seguridad integrados o grupos definidos en Azure Active Directory (Azure AD) si se integran con Azure DevOps, o Active Directory si se integran con TFS. 

- [Almacenamiento de código en Azure DevOps](https://docs.microsoft.com/azure/devops/repos/git/gitworkflow?view=azure-devops) 

- [Acerca de los permisos y los grupos en Azure DevOps](/azure/devops/organizations/security/about-permissions)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="77-deploy-configuration-management-tools-for-azure-resources"></a>7.7: Implementación de herramientas de administración de configuración para recursos de Azure

**Guía**: defina e implemente configuraciones de seguridad estándar para los recursos de Azure con Azure Policy. Use los alias de Azure Policy para crear directivas personalizadas con el fin de auditar o aplicar la configuración de los recursos relacionados con Azure Security Center. Además, puede usar Azure Automation para implementar los cambios de configuración. 

- [Configuración y administración de Azure Policy](../governance/policy/tutorials/create-and-manage.md) 

- [Uso de alias](../governance/policy/concepts/definition-structure.md#aliases)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="79-implement-automated-configuration-monitoring-for-azure-resources"></a>7.9: Implementación de la supervisión de la configuración automatizada para los recursos de Azure

**Guía**: use definiciones de Azure Policy integradas, así como alias de Azure Policy en los espacios de nombres "Microsoft.OperationalInsights" y "Microsoft. Security" para crear directivas personalizadas con el fin de alertar, auditar y aplicar configuraciones de recursos de Azure. Use las directivas "audit", "deny" y "deploy if not exist" de Azure Policy para aplicar automáticamente las configuraciones en los recursos de Azure.

- [Configuración y administración de Azure Policy](../governance/policy/tutorials/create-and-manage.md)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="711-manage-azure-secrets-securely"></a>7.11: Administre los secretos de Azure de forma segura

**Guía**: Azure Security Center usa un área de trabajo de Log Analytics configurada para almacenar los datos, las alertas y las recomendaciones que genera. Configure una clave administrada por el cliente (CMK) para el área de trabajo que ha configurado para la recopilación de datos de Security Centers. CMK permite cifrar todos los datos guardados o enviados en el área de trabajo con una clave de Azure Key Vault que usted mismo puede crear y que será de su propiedad. 

- [Clave administrada por el cliente de Azure Monitor](../azure-monitor/platform/customer-managed-keys.md)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="713-eliminate-unintended-credential-exposure"></a>7.13: Elimine la exposición de credenciales no intencionada

**Instrucciones**: Implemente el escáner de credenciales para identificar las credenciales en el código. El escáner de credenciales también fomenta el traslado de credenciales detectadas a ubicaciones más seguras, como Azure Key Vault.

- [Cómo configurar Credential Scanner](https://secdevtools.azurewebsites.net/helpcredscan.html)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

## <a name="malware-defense"></a>Defensa contra malware

*Para obtener más información, consulte [Azure Security Benchmark: defensa contra malware](/azure/security/benchmarks/security-control-malware-defense).*

### <a name="82-pre-scan-files-to-be-uploaded-to-non-compute-azure-resources"></a>8.2: Examine previamente los archivos que se van a cargar en recursos de Azure que no son de proceso

**Guía**: Azure Security Center no está diseñado para almacenar ni procesar archivos. Es su responsabilidad realizar un examen previo del contenido que se carga en los recursos de Azure que no son de proceso, como el área de trabajo de Log Analytics.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

## <a name="data-recovery"></a>Recuperación de datos

*Para obtener más información, consulte [Azure Security Benchmark: recuperación de datos](/azure/security/benchmarks/security-control-data-recovery).*

### <a name="91-ensure-regular-automated-back-ups"></a>9.1: Garantía de copias de seguridad automáticas periódicas 

**Guía**: siga un enfoque de infraestructura como código (IAC) y use Azure Resource Manager para implementar los recursos relacionados con Azure Security Center en una plantilla de notación de objetos JavaScript (JSON) que se puede usar como copia de seguridad para las configuraciones relacionadas con los recursos.

- [Exportación de uno y varios recursos a una plantilla en Azure Portal](../azure-resource-manager/templates/export-template-portal.md)

- [Plantillas de Azure Resource Manager para recursos de seguridad](/azure/templates/microsoft.security/allversions)

- [Información sobre Azure Automation](../automation/automation-intro.md)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="92-perform-complete-system-backups-and-backup-any-customer-managed-keys"></a>9.2: Copias de seguridad completas del sistema y copia de seguridad de las claves administradas por el cliente

**Instrucciones**: Azure Security Center usa un área de trabajo Log Analytics para almacenar los datos, las alertas y las recomendaciones que genera. Puede configurar Azure Monitor y el área de trabajo que Security Center usa para habilitar una clave administrada por el cliente. Si usa Key Vault para almacenar las claves administradas por el cliente, asegúrese de que se realizan con regularidad copias de seguridad automatizadas de las claves.

- [Procedimiento para realizar copias de seguridad de claves de Key Vault](/powershell/module/azurerm.keyvault/backup-azurekeyvaultkey)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="93-validate-all-backups-including-customer-managed-keys"></a>9.3: Validación de todas las copias de seguridad, incluidas las claves administradas por el cliente

**Guía**: garantice la restauración periódica con copias de seguridad de los archivos de plantilla de Azure Resource Manager. Pruebe la restauración de la copia de seguridad de las claves administradas por el cliente.

- [Administración del área de trabajo de Log Analytics mediante las plantillas de Azure Resource Manager](../azure-monitor/platform/template-workspace-configuration.md)

- [Restauración de las claves del almacén de claves en Azure](https://docs.microsoft.com/powershell/module/azurerm.keyvault/restore-azurekeyvaultkey?view=azurermps-6.13.0)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="94-ensure-protection-of-backups-and-customer-managed-keys"></a>9.4: Garantía de la protección de las copias de seguridad y las claves administradas por el cliente

**Guía**: Use Azure DevOps para almacenar y administrar de forma segura el código, como definiciones de Azure Policy personalizadas y plantillas de Azure Resource Manager. Para proteger los recursos que administra en Azure DevOps, puede conceder o denegar permisos a usuarios específicos, grupos de seguridad integrados o grupos definidos en Azure Active Directory (Azure AD) si se integran con Azure DevOps, o en Active Directory si se integran con TFS. Use el control de acceso basado en roles para proteger las claves administradas por el cliente.

Además, habilite la eliminación temporal y la protección de purga en Key Vault para proteger las claves contra la eliminación accidental o malintencionada.  Si Azure Storage se usa para almacenar las plantillas de Azure Resource Manager, habilite la eliminación temporal para guardar y recuperar los datos cuando se eliminen blobs o instantáneas de blobs. 

- [Almacenamiento de código en Azure DevOps](https://docs.microsoft.com/azure/devops/repos/git/gitworkflow?view=azure-devops)

- [Acerca de los permisos y los grupos en Azure DevOps](/azure/devops/organizations/security/about-permissions)

- [Procedimiento para habilitar la eliminación temporal y la protección de purga en Key Vault](https://docs.microsoft.com/azure/storage/blobs/storage-blob-soft-delete?tabs=azure-portal) 

- [Eliminación temporal de blobs de Azure Storage](https://docs.microsoft.com/azure/storage/blobs/storage-blob-soft-delete?tabs=azure-portal)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

## <a name="incident-response"></a>Respuesta a los incidentes

*Para obtener más información, consulte [Azure Security Benchmark: Respuesta a los incidentes](/azure/security/benchmarks/security-control-incident-response).*

### <a name="101-create-an-incident-response-guide"></a>10.1: Creación de una guía de respuesta ante incidentes

**Guía**: desarrolle una guía de respuesta a incidentes para su organización. Asegúrese de que haya planes de respuesta a incidentes escritos que definan todos los roles del personal, así como las fases de administración y gestión de los incidentes, desde la detección hasta la revisión posterior a la incidencia. 

- [Guía para crear su propio proceso de respuesta a incidentes de seguridad](https://msrc-blog.microsoft.com/2019/07/01/inside-the-msrc-building-your-own-security-incident-response-process/) 

- [Anatomía de un incidente del Centro de respuestas de seguridad de Microsoft](https://msrc-blog.microsoft.com/2019/06/27/inside-the-msrc-anatomy-of-a-ssirp-incident/) 

- [Uso de la guía de control de incidentes de seguridad de equipos de NIST para la creación de su propio plan de respuesta a incidentes](https://csrc.nist.gov/publications/detail/sp/800-61/rev-2/final)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="102-create-an-incident-scoring-and-prioritization-procedure"></a>10.2: Creación de un procedimiento de priorización y puntuación de incidentes

**Instrucciones**: Azure Security Center asigna una gravedad a cada alerta para ayudarle a priorizar aquellas que se deben investigar en primer lugar. La gravedad se basa en la confianza que tiene Security Center en la búsqueda o en el análisis utilizados para emitir la alerta, así como en el nivel de confianza de que ha habido un intento malintencionado detrás de la actividad que ha provocado la alerta.

Adicionalmente, marque las suscripciones con etiquetas y cree un sistema de nomenclatura para identificar y clasificar los recursos de Azure, especialmente los que procesan datos confidenciales.  Es su responsabilidad asignar prioridades a la corrección de las alertas en función de la importancia de los recursos y el entorno de Azure donde se produjo el incidente. 

- [Alertas de seguridad en el Centro de seguridad de Azure](security-center-alerts-overview.md) 

- [Uso de etiquetas para organizar los recursos de Azure](/azure/azure-resource-manager/resource-group-using-tags)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="103-test-security-response-procedures"></a>10.3: Prueba de los procedimientos de respuesta de seguridad

**Instrucciones**: Realice ejercicios para probar las funcionalidades de respuesta a los incidentes de los sistemas periódicamente; así, ayudará a proteger los recursos de Azure. Identifique puntos débiles y brechas y después revise el plan de respuesta según sea necesario. 

- [Publicación de NIST: Guía para probar, entrenar y ejecutar programas para planes y funcionalidades de TI](https://csrc.nist.gov/publications/detail/sp/800-84/final)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="104-provide-security-incident-contact-details-and-configure-alert-notifications-for-security-incidents"></a>10.4: Provisión de detalles de contacto de incidentes de seguridad y configuración de notificaciones de alerta para incidentes de seguridad

**Instrucciones**: La información de contacto del incidente de seguridad la utilizará Microsoft para ponerse en contacto con usted si Microsoft Security Response Center (MSRC) detecta que un tercero no autorizado o ilegal ha accedido a los datos. Revise los incidentes después del hecho para asegurarse de que se resuelven los problemas. 

- [Establecimiento del contacto de seguridad de Azure Security Center](security-center-provide-security-contact-details.md)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="105-incorporate-security-alerts-into-your-incident-response-system"></a>10.5: Incorporación de alertas de seguridad en el sistema de respuesta a incidentes

**Instrucciones**: exporte las alertas y recomendaciones de Azure Security Center y mediante la característica de exportación continua para ayudar a identificar riesgos en los recursos de Azure. La exportación continua permite exportar alertas y recomendaciones de forma manual o continua. Puede usar el conector de datos de Azure Security Center para enviar las alertas a Azure Sentinel. 

- [Configuración de la exportación continua](continuous-export.md) 

- [Transmisión de alertas a Azure Sentinel](../sentinel/connect-azure-security-center.md)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="106-automate-the-response-to-security-alerts"></a>10.6: Automatización de la respuesta a las alertas de seguridad

**Guía**: use la característica de automatización de flujos de trabajo de Azure Security Center para desencadenar automáticamente las alertas y recomendaciones de seguridad con el fin de proteger los recursos de Azure. 

- [Configuración de la automatización de flujos de trabajo en Security Center](workflow-automation.md)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

## <a name="penetration-tests-and-red-team-exercises"></a>Pruebas de penetración y ejercicios del equipo rojo

*Para obtener más información, consulte [Azure Security Benchmark: Pruebas de penetración y ejercicios del equipo rojo](/azure/security/benchmarks/security-control-penetration-tests-red-team-exercises).*

### <a name="111-conduct-regular-penetration-testing-of-your-azure-resources-and-ensure-remediation-of-all-critical-security-findings"></a>11.1: Realice pruebas de penetración periódicas de los recursos de Azure y asegúrese de corregir todos los resultados de seguridad críticos

**Guía**: Siga las reglas de compromiso de la prueba de penetración de Microsoft Cloud para asegurarse de que las pruebas de penetración no infrinjan las directivas de Microsoft. Use la estrategia de Microsoft y la ejecución de las pruebas de penetración del equipo rojo y sitios activos en la infraestructura de nube, los servicios y las aplicaciones administradas por Microsoft. 

- [Reglas de interacción de las pruebas de penetración](https://www.microsoft.com/msrc/pentest-rules-of-engagement?rtc=1) 

- [Equipo rojo de Microsoft Cloud](https://gallery.technet.microsoft.com/Cloud-Red-Teaming-b837392e)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Compartido

## <a name="next-steps"></a>Pasos siguientes

- Consulte la [prueba comparativa de seguridad de Azure](/azure/security/benchmarks/overview).
- Obtenga más información sobre las [líneas de base de seguridad de Azure](/azure/security/benchmarks/security-baselines-overview).
