---
title: Línea de base de seguridad de Azure para Batch
description: Línea de base de seguridad de Azure para Batch
author: msmbaldwin
ms.service: security
ms.topic: conceptual
ms.date: 04/09/2020
ms.author: mbaldwin
ms.custom: subject-security-benchmark
ms.openlocfilehash: 1eb24871817f365efe58b8e687563727df74493c
ms.sourcegitcommit: 5a3b9f35d47355d026ee39d398c614ca4dae51c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89400983"
---
# <a name="azure-security-baseline-for-batch"></a>Línea de base de seguridad de Azure para Batch

La línea de base de seguridad de Azure para Batch contiene recomendaciones que le ayudarán a mejorar la postura de seguridad de la implementación.

La base de referencia de este servicio se extrae de la [versión 1.0 de Azure Security Benchmark](../security/benchmarks/overview.md), que proporciona recomendaciones sobre cómo puede proteger las soluciones en la nube en Azure con nuestra guía de procedimientos recomendados.

Para obtener más información, consulte [Introducción a las líneas de base de seguridad de Azure](../security/benchmarks/security-baselines-overview.md).

## <a name="network-security"></a>Seguridad de redes

*Para obtener más información, consulte [Control de seguridad: seguridad de red](../security/benchmarks/security-control-network-security.md).*

### <a name="11-protect-resources-using-network-security-groups-or-azure-firewall-on-your-virtual-network"></a>1.1: Proteja los recursos mediante grupos de seguridad de red o Azure Firewall en su red virtual

**Guía**: implemente grupos de Azure Batch dentro de una red virtual. Para permitir que los nodos de proceso del grupo se comuniquen de manera segura con otras máquinas virtuales o con una red local, puede aprovisionar el grupo en una subred de una red virtual de Azure. Además, la implementación del grupo en una red virtual le proporciona control sobre el grupo de seguridad de red (NSG) que se usa para proteger las interfaces de red (NIC) de los nodos individuales, así como la subred. Configure el NSG para permitir el tráfico solo desde las direcciones IP y ubicaciones de confianza en Internet.

Cómo crear un grupo de Azure Batch en una red virtual:

https://docs.microsoft.com/azure/batch/batch-virtual-network

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="12-monitor-and-log-the-configuration-and-traffic-of-vnets-subnets-and-nics"></a>1.2: Supervisión y registro de la configuración y el tráfico de redes virtuales, subredes y NIC

**Guía**: use Azure Security Center y corrija las recomendaciones de protección de red relacionadas con la red virtual o el grupo de seguridad de red (NSG) asociado al grupo de Batch. Habilite los registros de flujo en el NSG que se usa para proteger el grupo de Batch y envíe los registros a una cuenta de Azure Storage para la auditoría de tráfico. También puede enviar registros de flujo del grupo de seguridad de red a un área de trabajo de Azure Log Analytics y usar Análisis de tráfico de Azure para proporcionar información detallada sobre el flujo de tráfico en la nube de Azure. Algunas de las ventajas de Análisis de tráfico de Azure son la capacidad de visualizar la actividad de la red e identificar las zonas activas, identificar las amenazas de seguridad, comprender los patrones de flujo de tráfico y detectar configuraciones de red incorrectas.

Cómo habilitar los registros de flujo de NSG:

https://docs.microsoft.com/azure/network-watcher/network-watcher-nsg-flow-logging-portal

Habilitación y uso del Análisis de tráfico:

https://docs.microsoft.com/azure/network-watcher/traffic-analytics

Descripción de la seguridad de red proporcionada por Azure Security Center:

https://docs.microsoft.com/azure/security-center/security-center-network-recommendations

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="13-protect-critical-web-applications"></a>1.3: Protección de las aplicaciones web críticas

**Instrucciones**:  No aplicable; esta recomendación está pensada para los recursos de proceso.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="14-deny-communications-with-known-malicious-ip-addresses"></a>1.4: Denegar las comunicaciones con direcciones IP malintencionadas conocidas

**Guía**: habilite la protección estándar de DDoS (denegación de servicio distribuida) de Azure en la red virtual protegiendo el grupo de Azure Batch para la protección frente a ataques DDoS. Use la inteligencia sobre amenazas integrada de Azure Security Center para denegar las comunicaciones con direcciones IP malintencionadas conocidas o no utilizadas.

Cómo configurar la protección contra DDoS:

https://docs.microsoft.com/azure/virtual-network/manage-ddos-protection

Descripción del la inteligencia sobre amenazas integrada de Azure Security Center:

https://docs.microsoft.com/azure/security-center/security-center-alerts-service-layer

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="15-record-network-packets-and-flow-logs"></a>1.5: Registro de los paquetes de red y registros de flujo

**Guía**: habilite los registros de flujo en el grupo de seguridad de red (NSG) que se usa para proteger el grupo de Azure Batch y envíe los registros a una cuenta de Azure Storage para la auditoría de tráfico.

Cómo habilitar los registros de flujo de NSG:

https://docs.microsoft.com/azure/network-watcher/network-watcher-nsg-flow-logging-portal

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="16-deploy-network-based-intrusion-detectionintrusion-prevention-systems"></a>1.6: Implementación de sistemas de prevención de intrusiones y detección de intrusiones basados en la red

**Guía**: si es necesario por motivos de cumplimiento, seleccione una aplicación virtual de red de Azure Marketplace que admita la funcionalidad de sistemas de detección de intrusiones (IDS) y sistemas de prevención de intrusiones (IPS) con funcionalidades de inspección de carga.

Si la detección y/o la prevención de intrusiones basadas en la inspección de carga no son un requisito, se puede usar Azure Firewall con la inteligencia sobre amenazas. El filtrado basado en inteligencia sobre amenazas de Azure Firewall puede alertar y denegar el tráfico desde y hacia los dominios y las direcciones IP malintencionados conocidos. La direcciones IP y los dominios proceden de la fuente Inteligencia sobre amenazas de Microsoft.

Implemente Azure Firewall con una dirección IP pública de la misma red virtual que los nodos del grupo de Azure Batch. Configure reglas de traducción de direcciones de red (NAT) entre las ubicaciones de confianza de Internet y las direcciones IP privadas de los nodos del grupo individuales. En Azure Firewall, en Inteligencia sobre amenazas, configure "Alertar y denegar" para alertar y bloquear el tráfico hacia y desde direcciones IP y dominios malintencionados conocidos. La direcciones IP y los dominios proceden de la fuente de inteligencia sobre amenazas de Microsoft y solo se incluyen los registros de mayor confianza. 

Cómo crear un grupo de Azure Batch en una red virtual:

https://docs.microsoft.com/azure/batch/batch-virtual-network

Cómo implementar Azure Firewall:

https://docs.microsoft.com/azure/firewall/tutorial-firewall-deploy-portal

Azure Marketplace:

https://azuremarketplace.microsoft.com/marketplace/?term=Firewall

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="17-manage-traffic-to-your-web-applications"></a>1.7: Administración del tráfico a las aplicaciones web

**Guía**: no aplicable, la prueba comparativa está pensada para aplicaciones web que se ejecutan en Azure App Service o instancias de IaaS.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="18-minimize-complexity-and-administrative-overhead-of-network-security-rules"></a>1.8: Minimice la complejidad y la sobrecarga administrativa de las reglas de seguridad de red

**Guía**: use etiquetas de servicio de red virtual para definir controles de acceso a la red en los grupos de seguridad de red o las instancias de Azure Firewall asociadas a los grupos de Azure Batch. Puede utilizar etiquetas de servicio en lugar de direcciones IP específicas al crear reglas de seguridad. Al especificar el nombre de la etiqueta de servicio (por ejemplo, ApiManagement) en el campo de origen o destino apropiado de una regla, puede permitir o denegar el tráfico para el servicio correspondiente. Microsoft administra los prefijos de direcciones que la etiqueta de servicio incluye y actualiza automáticamente dicha etiqueta a medida que las direcciones cambian.

Descripción y uso de las etiquetas de servicio:

https://docs.microsoft.com/azure/virtual-network/service-tags-overview

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="19-maintain-standard-security-configurations-for-network-devices"></a>1.9: Mantenimiento de las configuraciones de seguridad estándar para dispositivos de red

**Guía**: defina e implemente configuraciones de seguridad estándar para los recursos de red asociados a los grupos de Azure Batch con Azure Policy. Use alias de Azure Policy en los espacios de nombres "Microsoft.Batch" y "Microsoft.Network" para crear directivas personalizadas con el fin de auditar o aplicar la configuración de red de los grupos de Azure Batch.

Configuración y administración de Azure Policy:

https://docs.microsoft.com/azure/governance/policy/tutorials/create-and-manage

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="110-document-traffic-configuration-rules"></a>1.10: Documentación de las reglas de configuración de tráfico

**Guía**: use etiquetas para los grupos de seguridad de red (NSG) y otros recursos relacionados con la seguridad de red y el flujo de tráfico que están asociados a los grupos de Azure Batch. En el caso de las reglas de NSG individuales, use el campo "Descripción" para especificar las necesidades empresariales o la duración (etc.) de las reglas que permiten que entre o salga el tráfico en una red.

Use cualquiera de las definiciones de Azure Policy integradas relacionadas con el etiquetado, como "Requerir etiqueta y su valor", para asegurarse de que todos los recursos se creen con etiquetas y para notificarle los recursos no etiquetados existentes.

Puede usar Azure PowerShell o la CLI de Azure para buscar o realizar acciones en los recursos en función de sus etiquetas.

Creación y uso de etiquetas:

https://docs.microsoft.com/azure/azure-resource-manager/resource-group-using-tags

Creación de una red virtual:

https://docs.microsoft.com/azure/virtual-network/quick-create-portal

Cómo crear un NSG:

https://docs.microsoft.com/azure/virtual-network/tutorial-filter-network-traffic

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="111-use-automated-tools-to-monitor-network-resource-configurations-and-detect-changes"></a>1.11: Uso de herramientas automatizadas para supervisar las configuraciones de los recursos de red y detectar cambios

**Guía**: use el registro de actividad de Azure para supervisar las configuraciones de los recursos de red y detectar cambios en los recursos de red relacionados con los grupos de Azure Batch. Cree alertas en Azure Monitor que se desencadenarán cuando se produzcan cambios en los recursos de red críticos.

Visualización y recuperación de eventos del registro de actividad de Azure: https://docs.microsoft.com/azure/azure-monitor/platform/activity-log-view 

Creación de alertas en Azure Monitor: https://docs.microsoft.com/azure/azure-monitor/platform/alerts-activity-log

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

## <a name="logging-and-monitoring"></a>Registro y supervisión

*Para obtener más información, consulte [Control de seguridad: registro y supervisión](../security/benchmarks/security-control-logging-monitoring.md).*

### <a name="21-use-approved-time-synchronization-sources"></a>2.1: Uso de orígenes de sincronización de hora aprobados

**Guía**: para Azure Batch, de forma predeterminada, Microsoft proporciona sincronización de hora. Sin embargo, si tiene requisitos de sincronización de hora específicos, puede implementar dichos cambios.

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Microsoft

### <a name="22-configure-central-security-log-management"></a>2.2: Configuración de la administración central de registros de seguridad

**Guía**: incorpore la cuenta de Azure Batch a Azure Monitor para agregar los datos de seguridad generados por los dispositivos del clúster. Aproveche las consultas personalizadas para detectar amenazas en el entorno y responder a ellas.  Para la supervisión en el nivel de recurso de Azure Batch, utilice las API de Batch para supervisar o consultar el estado de los recursos, como los trabajos, las tareas, los nodos y los grupos.

Cómo incorporar una cuenta de Azure Batch a Azure Monitor:

https://docs.microsoft.com/azure/batch/batch-diagnostics

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="23-enable-audit-logging-for-azure-resources"></a>2.3: Habilitación del registro de auditoría para los recursos de Azure

**Guía**: Para la supervisión en el nivel de cuenta de Azure Batch, supervise cada cuenta de Batch con las características de Azure Monitor. Azure Monitor recopila métricas y, opcionalmente, registros de diagnóstico para los recursos del nivel de cuenta de Batch, como grupos, trabajos y tareas. Recopile y consuma estos datos manualmente o mediante programación para supervisar las actividades de la cuenta de Batch y para diagnosticar problemas.

Para la supervisión en el nivel de recurso de Azure Batch, utilice las API de Azure Batch para supervisar o consultar el estado de los recursos, como los trabajos, las tareas, los nodos y los grupos.

Cómo configurar el registro y la supervisión en el nivel de cuenta de Azure Batch:

https://docs.microsoft.com/azure/batch/monitoring-overview

Descripción de la supervisión en el nivel de recurso de Batch:

https://docs.microsoft.com/azure/batch/monitoring-overview#batch-resource-monitoring

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="24-collect-security-logs-from-operating-system"></a>2.4: Recopilación de los registros de seguridad del sistema operativo

**Guía**: Azure Monitor recopila métricas y registros de diagnóstico de los recursos de la cuenta de Azure Batch. Recopile y consuma estos datos de diversas maneras para supervisar la cuenta de Azure Batch y diagnosticar problemas. También puede configurar alertas de métricas para recibir notificaciones cuando una métrica alcance un valor especificado.

Si es necesario, es posible conectarse a los nodos individuales de un grupo mediante Secure Shell (SSH) o Protocolo de escritorio remoto (RDP) para acceder a los registros del sistema operativo local.

Cómo recopilar los registros de diagnóstico de la cuenta de Azure Batch:

https://docs.microsoft.com/azure/batch/batch-diagnostics#batch-diagnostics

Cómo conectarse de forma remota a los nodos del grupo de Azure Batch:

https://docs.microsoft.com/azure/batch/batch-api-basics#error-handling

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="25-configure-security-log-storage-retention"></a>2.5: Configuración de la retención del almacenamiento de registros de seguridad

**Guía**: Incorpore una cuenta de Azure Batch a Azure Monitor. Asegúrese de que el área de trabajo de Azure Log Analytics utilizada tiene el período de retención establecido de acuerdo con la normativa de cumplimiento de la organización.

Cómo configurar el registro y la supervisión de Azure Batch:

https://docs.microsoft.com/azure/batch/monitoring-overview

Cómo configurar el período de retención del área de trabajo de Azure Log Analytics:

https://docs.microsoft.com/azure/azure-monitor/platform/manage-cost-storage

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="26-monitor-and-review-logs"></a>2.6: Supervisión y revisión de registros

**Guía**: cree alertas de métricas de Azure Batch que se desencadenan cuando el valor de una métrica específica cruza un umbral determinado.

Cómo configurar alertas de métricas de Azure Batch:

https://docs.microsoft.com/azure/batch/batch-diagnostics

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="27-enable-alerts-for-anomalous-activity"></a>2.7: Habilitación de alertas para actividades anómalas

**Guía**: cree alertas de métricas de Azure Batch que se desencadenan cuando el valor de una métrica específica cruza un umbral determinado.

Cómo configurar alertas de métricas de Azure Batch:

https://docs.microsoft.com/azure/batch/batch-diagnostics

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="28-centralize-anti-malware-logging"></a>2.8: Centralización del registro antimalware

**Guía**: use Windows Defender en los nodos de Batch individuales en el caso de los sistemas operativos Windows o proporcione su propia solución antimalware si usa Linux.

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="29-enable-dns-query-logging"></a>2.9: Habilitación del registro de consultas de DNS

**Guía**: implemente una solución de terceros para el registro de DNS.

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="210-enable-command-line-audit-logging"></a>2.10: Habilitación del registro de auditoría de la línea de comandos

**Guía**: configure manualmente el registro de consola y la transcripción de PowerShell en cada nodo.

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

## <a name="identity-and-access-control"></a>Identidad y Access Control

*Para obtener más información, consulte [Control de seguridad: identidad y control de acceso](../security/benchmarks/security-control-identity-access-control.md).*

### <a name="31-maintain-inventory-of-administrative-accounts"></a>3.1: Mantenimiento de un inventario de cuentas administrativas

**Guía**: mantenga el registro de la cuenta administrativa local que se crea durante el aprovisionamiento del grupo de Azure Batch, así como de cualquier otra cuenta que cree. Además, si se usa la integración de Azure Active Directory (AAD), AAD tiene roles integrados que se deben asignar explícitamente y, por tanto, se pueden consultar. Use el módulo de PowerShell de AAD para realizar consultas ad hoc para detectar cuentas que son miembros de grupos administrativos.

Además, puede usar las recomendaciones de administración de identidades y acceso de Azure Security Center.

Cómo obtener un rol de directorio en AAD con PowerShell:

https://docs.microsoft.com/powershell/module/azuread/get-azureaddirectoryrole?view=azureadps-2.0

Cómo obtener los miembros de un rol de directorio en AAD con PowerShell:

https://docs.microsoft.com/powershell/module/azuread/get-azureaddirectoryrolemember?view=azureadps-2.0

Cómo supervisar la identidad y el acceso con Azure Security Center:

https://docs.microsoft.com/azure/security-center/security-center-identity-access

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="32-change-default-passwords-where-applicable"></a>3.2: Cambio de las contraseñas predeterminadas cuando proceda

**Guía**: al aprovisionar un grupo de Azure Batch, se le ofrece la opción de crear cuentas de equipo local. No hay contraseñas predeterminadas para cambiar; sin embargo, puede especificar distintas contraseñas para el acceso de Secure Shell (SSH) y Protocolo de escritorio remoto (RDP). Una vez configurado el grupo de Azure Batch, puede generar un usuario aleatorio para los nodos individuales en Azure Portal o mediante la API de Azure Resource Manager.

Cómo agregar un usuario a un nodo de proceso específico:

https://docs.microsoft.com/rest/api/batchservice/computenode/adduser

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="33-ensure-the-use-of-dedicated-administrative-accounts"></a>3.3: Aseguramiento del uso de cuentas administrativas dedicadas

**Guía**: integre la autenticación para las aplicaciones de Azure Batch con Azure Active Directory. Cree directivas y procedimientos en torno al uso de cuentas administrativas dedicadas.

Además, puede usar las recomendaciones de administración de identidades y acceso de Azure Security Center.

Cómo autenticar aplicaciones de Batch con Azure Active Directory:

https://docs.microsoft.com/azure/batch/batch-aad-auth

Cómo supervisar la identidad y el acceso con Azure Security Center:

https://docs.microsoft.com/azure/security-center/security-center-identity-access

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="34-utilize-single-sign-on-sso-with-azure-active-directory"></a>3.4: Uso del inicio de sesión único (SSO) con Azure Active Directory

**Guía**: no es aplicable; aunque Azure Batch admite la autenticación de Azure AD, no se admite el inicio de sesión único.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="35-use-multifactor-authentication-for-all-azure-active-directory-based-access"></a>3.5: Uso de la autenticación multifactor para todo el acceso basado en Azure Active Directory

**Guía**: integre la autenticación para las aplicaciones de Azure Batch con Azure Active Directory (AAD). Habilite la autenticación multifactor (MFA) de AAD y siga las recomendaciones de administración de identidades y acceso de Azure Security Center.

 

Cómo habilitar la MFA en Azure:

https://docs.microsoft.com/azure/active-directory/authentication/howto-mfa-getstarted

Cómo supervisar la identidad y el acceso en Azure Security Center:

https://docs.microsoft.com/azure/security-center/security-center-identity-access

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="36-use-dedicated-machines-privileged-access-workstations-for-all-administrative-tasks"></a>3.6: Uso de máquinas dedicadas (estaciones de trabajo de acceso con privilegios) para todas las tareas administrativas

**Guía**: utilice las PAW (estaciones de trabajo de acceso con privilegios) con la autenticación multifactor (MFA) configurada para iniciar sesión y configurar los recursos de Azure Batch.

Más información sobre las estaciones de trabajo con privilegios de acceso:

https://docs.microsoft.com/windows-server/identity/securing-privileged-access/privileged-access-workstations

Cómo habilitar la MFA en Azure:

https://docs.microsoft.com/azure/active-directory/authentication/howto-mfa-getstarted

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="37-log-and-alert-on-suspicious-activity-from-administrative-accounts"></a>3.7: Registro y alerta de la actividad sospechosa desde cuentas administrativas

**Guía**: si ha integrado la autenticación para las aplicaciones de Azure Batch con Azure Active Directory (AAD), use los informes de seguridad de Azure Active Directory para la generación de registros y alertas cuando se producen actividades sospechosas o no seguras en el entorno. Use Azure Security Center para supervisar la actividad de identidad y acceso.

Cómo identificar usuarios de Azure AD marcados por una actividad de riesgo:

https://docs.microsoft.com/azure/active-directory/reports-monitoring/concept-user-at-risk

Supervisión de la actividad de identidad y acceso de los usuarios en Azure Security Center:

https://docs.microsoft.com/azure/security-center/security-center-identity-access

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="38-manage-azure-resource-from-only-approved-locations"></a>3.8: Administración de los recursos de Azure solo desde ubicaciones aprobadas

**Guía**: si ha integrado la autenticación para las aplicaciones de Azure Batch con Azure Active Directory, puede usar ubicaciones con nombre de acceso condicional para permitir el acceso solo desde agrupaciones lógicas específicas de intervalos de direcciones IP o países o regiones.

Cómo configurar ubicaciones con nombre en Azure:

https://docs.microsoft.com/azure/active-directory/reports-monitoring/quickstart-configure-named-locations

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="39-use-azure-active-directory"></a>3.9: Uso de Azure Active Directory

**Guía**: use Azure Active Directory (AAD) como sistema central de autenticación y autorización e integre la autenticación para las aplicaciones de Azure Batch con AAD. AAD protege los datos mediante un fuerte cifrado para los datos en reposo y en tránsito. AAD también cifra con sal, convierte en hash y almacena de forma segura las credenciales de los usuarios.

Cómo crear y configurar una instancia de AAD:

https://docs.microsoft.com/azure/active-directory-domain-services/tutorial-create-instance

Cómo autenticar las aplicaciones de Batch con AAD:

https://docs.microsoft.com/azure/batch/batch-aad-auth

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="310-regularly-review-and-reconcile-user-access"></a>3.10: Revisión y conciliación de manera periódica del acceso de los usuarios

**Guía**: Azure Active Directory(AAD) proporciona registros para ayudar a descubrir cuentas obsoletas. Además, puede utilizar las revisiones de acceso de identidad de Azure para administrar de forma eficiente las pertenencias a grupos, el acceso a las aplicaciones empresariales y las asignaciones de roles. El acceso de los usuarios se puede revisar de forma periódica para asegurarse de que solo los usuarios adecuados tengan acceso continuado.

Cómo usar las revisiones de acceso de identidad de Azure:

https://docs.microsoft.com/azure/active-directory/governance/access-reviews-overview

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="311-monitor-attempts-to-access-deactivated-accounts"></a>3.11: Supervisión de los intentos de acceso a cuentas desactivadas

**Guía**: cree una configuración de diagnóstico para las cuentas de usuario de Azure Active Directory mediante el envío de los registros de auditoría y los registros de inicio de sesión a un área de trabajo de Azure Log Analytics. Configure las alertas deseadas en el área de trabajo de Azure Log Analytics.

Cómo integrar los registros de actividad de Azure en Azure Monitor:

https://docs.microsoft.com/azure/active-directory/reports-monitoring/howto-integrate-activity-logs-with-log-analytics

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="312-alert-on-account-login-behavior-deviation"></a>3.12: Alerta de las desviaciones de comportamiento en los inicios de sesión de las cuentas

**Guía**: use las características de protección de identidad y detección de riesgo de Azure Active Directory (AAD) para configurar respuestas automatizadas a las acciones sospechosas detectadas relacionadas con las identidades de los usuarios. Además, puede ingerir datos en Azure Sentinel para investigarlos más a fondo.

Visualización de inicios de sesión de riesgo de AAD:

https://docs.microsoft.com/azure/active-directory/reports-monitoring/concept-risky-sign-ins

Cómo configurar y habilitar las directivas de riesgo de protección de la identidad:

https://docs.microsoft.com/azure/active-directory/identity-protection/howto-identity-protection-configure-risk-policies

Incorporación de Azure Sentinel:

https://docs.microsoft.com/azure/sentinel/quickstart-onboard

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="313-divprovide-microsoft-with-access-to-relevant-customer-data-during-support-scenariosbrdiv"></a>3.13: <div>Proporcione a Microsoft acceso a los datos pertinentes del cliente durante los escenarios de soporte técnico<br></div>

**Guía**: no disponible; Caja de seguridad del cliente todavía no se admite para Azure Batch.
 
Lista de servicios admitidos por Caja de seguridad del cliente: https://docs.microsoft.com/azure/security/fundamentals/customer-lockbox-overview#supported-services-and-scenarios-in-general-availability



**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

## <a name="data-protection"></a>Protección de datos

*Para obtener más información, consulte [Control de seguridad: protección de datos](../security/benchmarks/security-control-data-protection.md).*

### <a name="41-maintain-an-inventory-of-sensitive-information"></a>4.1: Mantenimiento de un inventario de información confidencial

**Instrucciones**: use etiquetas para ayudar a realizar el seguimiento de los recursos de Azure que almacenan o procesan información confidencial.

Creación y uso de etiquetas:

https://docs.microsoft.com/azure/azure-resource-manager/resource-group-using-tags

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="42-isolate-systems-storing-or-processing-sensitive-information"></a>4.2: Aislamiento de los sistemas que almacenan o procesan información confidencial

**Instrucciones**: Implemente suscripciones y/o grupos de administración independientes para los entornos de desarrollo, prueba y producción. Los grupos de Azure Batch deben separarse mediante una red virtual o subred, etiquetarse adecuadamente y protegerse con grupos de seguridad de red (NSG). Los datos de Azure Batch deben residir en una cuenta de Azure Storage protegida.

Cómo crear un grupo de Azure Batch en una red virtual:

https://docs.microsoft.com/azure/batch/batch-virtual-network

Cómo proteger cuentas de Azure Storage:

https://docs.microsoft.com/azure/storage/common/storage-security-guide

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="43-monitor-and-block-unauthorized-transfer-of-sensitive-information"></a>4.3: Supervisión y bloqueo de transferencias no autorizadas de información confidencial

**Guía**: en el caso de las cuentas de Azure Storage asociadas a los grupos de Azure Batch que contienen información confidencial, márquelas como confidenciales mediante etiquetas y protéjalas con los procedimientos recomendados de Azure.

Las características de identificación, clasificación y prevención de pérdida de datos todavía no están disponibles para Azure Storage ni los recursos de proceso. Implemente una solución de terceros, si es necesario, para fines de cumplimiento.

En el caso de la plataforma subyacente administrada por Microsoft, Microsoft trata todo el contenido de los clientes como confidencial y hace grandes esfuerzos para proteger a los clientes contra la pérdida y exposición de sus datos. Para garantizar la seguridad de los datos de los clientes dentro de Azure, Microsoft ha implementado y mantiene un conjunto de controles y funcionalidades eficaces de protección de datos.

Descripción de la protección de datos de clientes en Azure:

https://docs.microsoft.com/azure/security/fundamentals/protection-customer-data

Cómo proteger cuentas de Azure Storage:

https://docs.microsoft.com/azure/storage/common/storage-security-guide

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Compartido

### <a name="44-encrypt-all-sensitive-information-in-transit"></a>4.4: Cifrado de toda la información confidencial en tránsito

**Instrucciones**: Cifre toda la información confidencial en tránsito. De forma predeterminada, los recursos de Microsoft Azure negociarán TLS 1.2. Asegúrese de que los clientes que se conectan a los grupos de Azure Batch o los almacenes de datos (cuentas de Azure Storage) pueden negociar TLS 1.2 o superior.

Asegúrese de que se requiere HTTPS para acceder a la cuenta de almacenamiento que contiene los datos de Azure Batch.

Descripción del cifrado en tránsito de la cuenta de Azure Storage:

https://docs.microsoft.com/azure/storage/common/storage-security-guide#encryption-in-transit

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Compartido

### <a name="45-use-an-active-discovery-tool-to-identify-sensitive-data"></a>4.5: Uso de una herramienta de detección activa para identificar datos confidenciales

**Guía**: en el caso de las cuentas de Azure Storage asociadas a los grupos de Azure Batch que contienen información confidencial, márquelas como confidenciales mediante etiquetas y protéjalas con los procedimientos recomendados de Azure.

Las características de identificación, clasificación y prevención de pérdida de datos todavía no están disponibles para Azure Storage ni los recursos de proceso. Implemente una solución de terceros, si es necesario, para fines de cumplimiento.

En el caso de la plataforma subyacente administrada por Microsoft, Microsoft trata todo el contenido de los clientes como confidencial y hace grandes esfuerzos para proteger a los clientes contra la pérdida y exposición de sus datos. Para garantizar la seguridad de los datos de los clientes dentro de Azure, Microsoft ha implementado y mantiene un conjunto de controles y funcionalidades eficaces de protección de datos.

Descripción de la protección de datos de clientes en Azure:

https://docs.microsoft.com/azure/security/fundamentals/protection-customer-data

Cómo proteger cuentas de Azure Storage:

https://docs.microsoft.com/azure/storage/common/storage-security-guide

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Compartido

### <a name="46-use-azure-rbac-to-control-access-to-resources"></a>4.6: Uso de RBAC de Azure para controlar el acceso a los recursos

**Guía**: Use el control de acceso basado en rol (RBAC) para controlar el acceso al plano de administración de los recursos de Azure, como la cuenta de Batch, los grupos de Batch y las cuentas de almacenamiento.

Descripción de Azure RBAC:

https://docs.microsoft.com/azure/role-based-access-control/overview

Configuración de RBAC de Azure:

https://docs.microsoft.com/azure/role-based-access-control/role-assignments-portal

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="47-use-host-based-data-loss-prevention-to-enforce-access-control"></a>4.7: Uso de la prevención de pérdida de datos basada en host para aplicar el control de acceso

**Guía**: las características de identificación, clasificación y prevención de pérdida de datos todavía no están disponibles para Azure Storage ni los recursos de proceso. Implemente una solución de terceros, si es necesario, para fines de cumplimiento.

En el caso de la plataforma subyacente administrada por Microsoft, Microsoft trata todo el contenido de los clientes como confidencial y hace grandes esfuerzos para proteger a los clientes contra la pérdida y exposición de sus datos. Para garantizar la seguridad de los datos de los clientes dentro de Azure, Microsoft ha implementado y mantiene un conjunto de controles y funcionalidades eficaces de protección de datos.

Descripción de la protección de datos de los clientes en Azure:

https://docs.microsoft.com/azure/security/fundamentals/protection-customer-data

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Compartido

### <a name="48-encrypt-sensitive-information-at-rest"></a>4.8: Cifrado en reposo de la información confidencial

**Guía**: en el caso de las cuentas de almacenamiento asociadas a la cuenta de Azure Batch, se recomienda permitir que Microsoft administre las claves de cifrado; sin embargo, tiene la opción de administrar sus propias claves si es necesario.

Cómo administrar claves de cifrado para cuentas de Azure Storage:

https://docs.microsoft.com/azure/storage/common/storage-encryption-keys-portal

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="49-log-and-alert-on-changes-to-critical-azure-resources"></a>4.9: Registro y alerta de cambios en los recursos críticos de Azure

**Guía**: use Azure Monitor con el registro de actividad de Azure para crear alertas para cuando se produzcan cambios en los recursos críticos de Azure relacionados o asociados a las cuentas y grupos de Azure Batch.

Configure los valores de diagnóstico para las cuentas de almacenamiento asociadas al grupo de Azure Batch para supervisar y registrar todas las operaciones CRUD en los datos del grupo.

Cómo crear alertas para eventos de registro de actividad de Azure:

https://docs.microsoft.com/azure/azure-monitor/platform/alerts-activity-log

Cómo habilitar el registro o la auditoría adicionales para una cuenta de Azure Storage:

https://docs.microsoft.com/azure/storage/common/storage-monitor-storage-account

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

## <a name="vulnerability-management"></a>Administración de vulnerabilidades

*Para obtener más información, consulte [Control de seguridad: administración de vulnerabilidades](../security/benchmarks/security-control-vulnerability-management.md).*

### <a name="51-run-automated-vulnerability-scanning-tools"></a>5.1: Ejecución de herramientas de análisis de vulnerabilidades automatizadas

**Guía**: en el caso de los nodos del grupo de Azure Batch, usted es responsable de administrar la solución de administración de vulnerabilidades.

Opcionalmente, si dispone de Rapid7, Qualys o cualquier otra suscripción de plataforma de administración de vulnerabilidades, puede instalar manualmente los agentes de evaluación de vulnerabilidades en los nodos del grupo de Batch y administrar los nodos mediante el portal correspondiente.

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="52-deploy-automated-operating-system-patch-management-solution"></a>5.2: Implementación de una solución de administración de revisiones de sistema operativo automatizada

**Guía**: Microsoft mantiene y actualiza las imágenes de nodo del grupo de Azure Batch de base. Asegúrese de que el sistema operativo de los nodos del grupo de Azure Batch permanece revisado a lo largo de la duración del clúster, lo que puede requerir la habilitación de actualizaciones automáticas, la supervisión de los nodos o la realización de reinicios periódicos.


**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Compartido

### <a name="53-deploy-automated-third-party-software-patch-management-solution"></a>5.3: Implementación de una solución automatizada de administración de revisiones de software de terceros

**Guía**: Asegúrese de que las aplicaciones de terceros de los nodos del grupo de Azure Batch permanecen revisadas a lo largo de la duración del clúster, lo que puede requerir la habilitación de actualizaciones automáticas, la supervisión de los nodos o la realización de reinicios periódicos.


**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="54-compare-back-to-back-vulnerability-scans"></a>5.4: Comparación de los exámenes de vulnerabilidades opuestos

**Guía**: Si dispone de Rapid7, Qualys o cualquier otra suscripción de plataforma de administración de vulnerabilidades, puede usar el portal del proveedor para ver y comparar los exámenes de vulnerabilidades opuestos.

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="55-use-a-risk-rating-process-to-prioritize-the-remediation-of-discovered-vulnerabilities"></a>5.5: Use un proceso de clasificación de riesgos para priorizar la corrección de vulnerabilidades detectadas.

**Guía**: use un programa de puntuación de riesgos común (por ejemplo, Common Vulnerability Scoring System) o la clasificación de riesgos predeterminada proporcionada por su herramienta de examen de terceros.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

## <a name="inventory-and-asset-management"></a>Administración de recursos y del inventario

*Para obtener más información, consulte [Control de seguridad: inventario y administración de recursos](../security/benchmarks/security-control-inventory-asset-management.md).*

### <a name="61-use-azure-asset-discovery"></a>6.1: Uso de la detección de recursos de Azure

**Guía**: use Azure Resource Graph para consultar o detectar todos los recursos (por ejemplo, proceso, almacenamiento, red, etc.) dentro de las suscripciones. Asegúrese de que tiene los permisos adecuados (lectura) en el inquilino y de que puede enumerar todas las suscripciones de Azure, así como los recursos de las suscripciones.

Aunque los recursos clásicos de Azure se pueden detectar a través de Resource Graph, se recomienda encarecidamente crear y usar los recursos de Azure Resource Manager (ARM) en adelante.

Creación de consultas con Azure Resource Graph:

https://docs.microsoft.com/azure/governance/resource-graph/first-query-portal

Visualización de las suscripciones de Azure:

https://docs.microsoft.com/powershell/module/az.accounts/get-azsubscription?view=azps-3.0.0

Descripción de Azure RBAC:

https://docs.microsoft.com/azure/role-based-access-control/overview

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="62-maintain-asset-metadata"></a>6.2: Mantenimiento de metadatos de recursos

**Guía**: Aplique etiquetas a los recursos de Azure que proporcionan metadatos para organizarlos de forma lógica en una taxonomía.

Creación y uso de etiquetas:

https://docs.microsoft.com/azure/azure-resource-manager/resource-group-using-tags

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="63-delete-unauthorized-azure-resources"></a>6.3: Eliminación de recursos de Azure no autorizados

**Guía**: Use el etiquetado, los grupos de administración y las suscripciones independientes, si procede, para organizar y realizar un seguimiento de los recursos. Concilie el inventario periódicamente y asegúrese de que los recursos no autorizados se eliminan de la suscripción de manera oportuna.

Creación de suscripciones de Azure adicionales:

https://docs.microsoft.com/azure/billing/billing-create-subscription

Cómo crear grupos de administración:

https://docs.microsoft.com/azure/governance/management-groups/create

Creación y uso de etiquetas:

https://docs.microsoft.com/azure/azure-resource-manager/resource-group-using-tags

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="64-maintain-inventory-of-approved-azure-resources-and-software-titles"></a>6.4: Mantenimiento de un inventario de los recursos de Azure y títulos de software aprobados

**Guía**: defina una lista de los recursos de Azure y el software aprobados para los recursos de proceso.


**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="65-monitor-for-unapproved-azure-resources"></a>6.5: Supervisión de recursos de Azure no aprobados

**Instrucciones**: use Azure Policy para establecer restricciones sobre el tipo de recursos que se pueden crear en las suscripciones del cliente con las siguientes definiciones de directiva integradas:

- Tipos de recursos no permitidos
- Tipos de recursos permitidos

Use Azure Resource Graph para consultar o detectar recursos dentro de las suscripciones. Asegúrese de que todos los recursos de Azure presentes en el entorno estén aprobados.

Cómo configurar y administrar Azure Policy: https://docs.microsoft.com/azure/governance/policy/tutorials/create-and-manage

Creación de consultas con Azure Graph: https://docs.microsoft.com/azure/governance/resource-graph/first-query-portal


**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="66-monitor-for-unapproved-software-applications-within-compute-resources"></a>6.6: Supervisión de aplicaciones de software no aprobadas en recursos de proceso

**Guía**: en el caso de los nodos del grupo de Azure Batch, implemente una solución de terceros para supervisar los nodos del clúster en busca de aplicaciones de software no aprobadas.


**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="67-remove-unapproved-azure-resources-and-software-applications"></a>6.7: Eliminación de aplicaciones de software y recursos de Azure no aprobados

**Guía**: en el caso de los nodos del grupo de Azure Batch, implemente una solución de terceros para supervisar los nodos del clúster en busca de aplicaciones de software no aprobadas.


**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="68-use-only-approved-applications"></a>6.8: Uso exclusivo de aplicaciones aprobadas

**Guía**: en el caso de los nodos del grupo de Azure Batch, implemente una solución de terceros para evitar que se ejecute software no autorizado.


**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="69-use-only-approved-azure-services"></a>6.9: Uso exclusivo de servicios de Azure aprobados

**Guía**: use Azure Policy para establecer restricciones sobre el tipo de recursos que se pueden crear en las suscripciones del cliente con las siguientes definiciones de directiva integradas:

- Tipos de recursos no permitidos
- Tipos de recursos permitidos

Cómo configurar y administrar Azure Policy: https://docs.microsoft.com/azure/governance/policy/tutorials/create-and-manage

Denegación de un tipo de recurso específico con Azure Policy: https://docs.microsoft.com/azure/governance/policy/samples/not-allowed-resource-types


**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="610-implement-approved-application-list"></a>6.10: Implementación de la lista de aplicaciones aprobadas

**Guía**: en el caso de los nodos del grupo de Azure Batch, implemente una solución de terceros para evitar que se ejecuten tipos de archivos no autorizados.


**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="611-divlimit-users-ability-to-interact-with-azure-resource-manager-via-scriptsdiv"></a>6.11: <div>Limitación de la capacidad de los usuarios para interactuar con Azure Resource Manager mediante scripts</div>

**Guía**: use el acceso condicional de Azure para limitar la capacidad de los usuarios de interactuar con Azure Resource Manager mediante la configuración de la opción "Bloquear acceso" en la aplicación "Administración de Microsoft Azure".

Configuración del acceso condicional para bloquear el acceso a Azure Resource Manager: https://docs.microsoft.com/azure/role-based-access-control/conditional-access-azure-management


**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="612-limit-users-ability-to-execute-scripts-within-compute-resources"></a>6.12: Limitación de la capacidad de los usuarios para ejecutar scripts en los recursos de proceso

**Guía**: no aplicable,

esto no es aplicable a Azure Batch, ya que los usuarios (no administradores) de los grupos de Azure Batch no necesitan tener acceso a los nodos individuales para ejecutar trabajos. El administrador del clúster ya tiene acceso raíz a todos los nodos.


**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="613-physically-or-logically-segregate-high-risk-applications"></a>6.13: Segregación física o lógica de aplicaciones de alto riesgo

**Guía**: no aplicable, la prueba comparativa está pensada para aplicaciones web que se ejecutan en Azure App Service o instancias de IaaS.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

## <a name="secure-configuration"></a>Configuración segura

*Para obtener más información, consulte [Control de seguridad: configuración segura](../security/benchmarks/security-control-secure-configuration.md).*

### <a name="71-establish-secure-configurations-for-all-azure-resources"></a>7.1: Establecimiento de configuraciones seguras para todos los recursos de Azure

**Guía**: use alias de Azure Policy en el espacio de nombres "Microsoft.Batch" para crear directivas personalizadas con el fin de auditar o aplicar la configuración de las cuentas y los grupos de Azure Batch.

Consulta de los alias de Azure Policy disponibles:

https://docs.microsoft.com/powershell/module/az.resources/get-azpolicyalias?view=azps-3.3.0

Configuración y administración de Azure Policy:

https://docs.microsoft.com/azure/governance/policy/tutorials/create-and-manage

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="72-establish-secure-configurations-for-your-operating-system"></a>7.2: Establecimiento de configuraciones seguras del sistema operativo

**Guía**: establezca configuraciones seguras para el sistema operativo de los nodos del grupo de Batch.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="73-maintain-secure-configurations-for-all-azure-resources"></a>7.3: Mantenimiento de configuraciones seguras para todos los recursos de Azure

**Instrucciones**: Use [deny] y [deploy if not exist] de Azure Policy para aplicar una configuración segura para los recursos de Azure relacionados con la cuenta y los grupos de Batch (por ejemplo, redes virtuales, subredes, instancias de Azure Firewall, cuentas de Azure Storage, etc.). Puede utilizar alias de Azure Policy procedentes de los siguientes espacios de nombres para crear directivas personalizadas:

- Microsoft.Batch

- Microsoft.Storage

- Microsoft.Network

Cómo configurar y administrar Azure Policy: https://docs.microsoft.com/azure/governance/policy/tutorials/create-and-manage

Descripción de los efectos de Azure Policy: https://docs.microsoft.com/azure/governance/policy/concepts/effects


**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="74-maintain-secure-configurations-for-operating-systems"></a>7.4: Mantenimiento de configuraciones seguras para los sistemas operativos

**Guía**: Microsoft administra y mantiene las imágenes del sistema operativo del grupo de Azure Batch. Usted es responsable de implementar la configuración de estado en el nivel de sistema operativo.

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Compartido

### <a name="75-securely-store-configuration-of-azure-resources"></a>7.5: Almacenamiento seguro de la configuración de los recursos de Azure

**Instrucciones**: Si usa definiciones personalizadas de Azure Policy para las cuentas y los grupos de Azure Batch, o para los recursos relacionados, use Azure Repos para almacenar y administrar el código de forma segura.

Cómo almacenar código en Azure DevOps:

https://docs.microsoft.com/azure/devops/repos/git/gitworkflow?view=azure-devops

Documentación de Azure Repos:

https://docs.microsoft.com/azure/devops/repos/index?view=azure-devops

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="76-securely-store-custom-operating-system-images"></a>7.6: Almacenamiento seguro de las imágenes de sistema operativo personalizadas

**Instrucciones**: Si usa imágenes personalizadas para los grupos de Azure Batch, use el control de acceso basado en rol de Azure (RBAC de Azure) para asegurarse de que solo los usuarios autorizados pueden acceder a las imágenes.

Descripción de Azure RBAC:

https://docs.microsoft.com/azure/role-based-access-control/rbac-and-directory-admin-roles

Configuración de RBAC de Azure:

https://docs.microsoft.com/azure/role-based-access-control/quickstart-assign-role-user-portal

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="77-deploy-system-configuration-management-tools"></a>7.7: Implementación de herramientas de administración de la configuración del sistema

**Guía**: use definiciones de Azure Policy integradas para alertar, auditar y aplicar las configuraciones de los recursos relacionados con Azure Batch.  use alias de Azure Policy en el espacio de nombres "Microsoft.Batch" para crear directivas personalizadas para las cuentas y los grupos de Azure Batch. Además, desarrolle un proceso y una canalización para administrar las excepciones de las directivas.

Configuración y administración de Azure Policy:

https://docs.microsoft.com/azure/governance/policy/tutorials/create-and-manage

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="78-deploy-system-configuration-management-tools-for-operating-systems"></a>7.8: Implementación de herramientas de administración de la configuración del sistema para sistemas operativos

**Guía**: implemente una solución de terceros para mantener el estado deseado de los sistemas operativos de los nodos del grupo de Azure Batch.


**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="79-implement-automated-configuration-monitoring-for-azure-services"></a>7.9: Implementación de la supervisión de la configuración automatizada para los servicios de Azure

**Guía**: use alias de Azure Policy en el espacio de nombres "Microsoft.Batch" para crear directivas personalizadas con el fin de auditar o aplicar la configuración de la instancia de Azure Batch. También puede usar cualquier directiva integrada creada específicamente para Azure Batch o los recursos usados por Azure Batch, como:

- Las subredes deben estar asociadas con un grupo de seguridad de red.

Las cuentas de almacenamiento deben usar un punto de conexión del servicio de red virtual

- Se deben habilitar los registros de diagnóstico en las cuentas de Batch

Consulta de los alias de Azure Policy disponibles: https://docs.microsoft.com/powershell/module/az.resources/get-azpolicyalias?view=azps-3.3.0

Cómo configurar y administrar Azure Policy: https://docs.microsoft.com/azure/governance/policy/tutorials/create-and-manage



**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="710-implement-automated-configuration-monitoring-for-operating-systems"></a>7.10: Implementación de la supervisión de la configuración automatizada para los sistemas operativos

**Guía**: implemente una solución de terceros para supervisar el estado de los sistemas operativos de los nodos del grupo de Azure Batch.


**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="711-securely-manage-azure-secrets"></a>7.11: Administración segura de los secretos de Azure

**Guía**: puede usar Azure Key Vault con las implementaciones de Azure Batch para administrar las claves para el almacenamiento del grupo en cuentas de Azure Storage.

Cómo integrar las identidades administradas de Azure:

https://docs.microsoft.com/azure/azure-app-configuration/howto-integrate-azure-managed-service-identity

Cómo crear un almacén de claves:

https://docs.microsoft.com/azure/key-vault/general/quick-create-portal

Autenticación en Key Vault:

https://docs.microsoft.com/azure/key-vault/general/authentication

Asignación de una directiva de acceso de Key Vault:

https://docs.microsoft.com/azure/key-vault/general/assign-access-policy-portal

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="712-securely-and-automatically-manage-identities"></a>7.12: Administración de identidades de forma segura y automática

**Guía**: no disponible, Azure Batch no admite identidades de servicio administrado


**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="713-eliminate-unintended-credential-exposure"></a>7.13: Elimine la exposición de credenciales no intencionada

**Instrucciones**: Implemente el escáner de credenciales para identificar las credenciales en el código. El escáner de credenciales también fomenta el traslado de credenciales detectadas a ubicaciones más seguras, como Azure Key Vault. 

Cómo configurar el escáner de credenciales: https://secdevtools.azurewebsites.net/helpcredscan.html

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

## <a name="malware-defense"></a>Defensa contra malware

*Para obtener más información, consulte [Control de seguridad: defensa contra malware](../security/benchmarks/security-control-malware-defense.md).*

### <a name="81-use-centrally-managed-anti-malware-software"></a>8.1: Uso de software antimalware administrado centralmente

**Guía**: use Windows Defender en los nodos del grupo de Azure Batch individuales en el caso de los sistemas operativos Windows o proporcione su propia solución antimalware si usa Linux.


**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="82-pre-scan-files-to-be-uploaded-to-non-compute-azure-resources"></a>8.2: Examine previamente los archivos que se van a cargar en recursos de Azure que no son de proceso

**Guía**: Microsoft Antimalware está habilitado en el host subyacente que admite los servicios de Azure (por ejemplo, Azure Batch), pero no se ejecuta en el contenido del cliente.

Examine previamente los archivos que se cargan en recursos de Azure que no son de proceso, como App Service, Data Lake Storage, Blob Storage, etc. Microsoft no tiene acceso a los datos del cliente en estas instancias.

Descripción de Microsoft Antimalware para Azure Cloud Services y Virtual Machines:

https://docs.microsoft.com/azure/security/fundamentals/antimalware

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Compartido

### <a name="83-ensure-anti-malware-software-and-signatures-are-updated"></a>8.3: Aseguramiento de la actualización del software y las firmas antimalware

**Guía**: use Windows Defender en los nodos individuales del grupo de Azure Batch en el caso de los sistemas operativos Windows y asegúrese de que está habilitada la actualización automática. Proporcione su propia solución antimalware si usa Linux.


**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

## <a name="data-recovery"></a>Recuperación de datos

*Para obtener más información, consulte [Control de seguridad: recuperación de datos](../security/benchmarks/security-control-data-recovery.md).*

### <a name="91-ensure-regular-automated-back-ups"></a>9.1: Garantía de copias de seguridad automáticas periódicas

**Guía**: Al usar una cuenta de Azure Storage para el almacén de datos del grupo de Azure Batch, elija la opción de redundancia adecuada (LRS,ZRS, GRS, RA-GRS). 

Cómo configurar la redundancia del almacenamiento para cuentas de Azure Storage:

https://docs.microsoft.com/azure/storage/common/storage-redundancy

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="92-perform-complete-system-backups-and-backup-any-customer-managed-keys"></a>9.2: Realización de copias de seguridad completas del sistema y copia de seguridad de las claves administradas por el cliente

**Guía**: Al usar una cuenta de Azure Storage para el almacén de datos del grupo de Azure Batch, elija la opción de redundancia adecuada (LRS,ZRS, GRS, RA-GRS).  Si usa Azure Key Vault en alguna parte de la implementación de Azure Batch, asegúrese de que se realizan copias de seguridad de las claves.

Cómo configurar la redundancia del almacenamiento para cuentas de Azure Storage:

https://docs.microsoft.com/azure/storage/common/storage-redundancy

Realización de una copia de seguridad de las claves del almacén de claves en Azure:

https://docs.microsoft.com/powershell/module/azurerm.keyvault/backup-azurekeyvaultkey?view=azurermps-6.13.0

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="93-validate-all-backups-including-customer-managed-keys"></a>9.3: Validación de todas las copias de seguridad, incluidas las claves administradas por el cliente

**Guía**: si administra sus propias claves para las cuentas de Azure Storage o cualquier otro recurso relacionado con la implementación de Azure Batch, pruebe de forma periódica la restauración de las claves de las que ha realizado copia de seguridad.

Realización de una copia de seguridad de las claves del almacén de claves en Azure:

https://docs.microsoft.com/powershell/module/azurerm.keyvault/backup-azurekeyvaultkey?view=azurermps-6.13.0

Cómo restaurar una clave administrada por el cliente con PowerShell:

https://docs.microsoft.com/powershell/module/azurerm.keyvault/restore-azurekeyvaultkey?view=azurermps-6.13.0

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="94-ensure-protection-of-backups-and-customer-managed-keys"></a>9.4: Aseguramiento de la protección de las copias de seguridad y las claves administradas por el cliente

**Guía**: Si se usa Azure Key Vault para contener las claves relacionadas con las cuentas de almacenamiento del grupo de Azure Batch, habilite la eliminación temporal en Azure Key Vault para proteger las claves contra la eliminación accidental o malintencionada.

Cómo habilitar la eliminación temporal en Azure Key Vault:

https://docs.microsoft.com/azure/key-vault/key-vault-soft-delete-powershell

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

## <a name="incident-response"></a>Respuesta a los incidentes

*Para obtener más información, consulte [Control de seguridad: respuesta ante incidentes](../security/benchmarks/security-control-incident-response.md).*

### <a name="101-create-incident-response-guide"></a>10.1: Creación de una guía de respuesta ante incidentes

**Guía**: Asegúrese de que haya planes de respuesta ante incidentes escritos que definan los roles del personal, así como las fases de administración y gestión de los incidentes.

Configuración de las automatizaciones del flujo de trabajo en Azure Security Center:

https://docs.microsoft.com/azure/security-center/security-center-planning-and-operations-guide

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="102-create-incident-scoring-and-prioritization-procedure"></a>10.2: Creación de un procedimiento de priorización y puntuación de incidentes

**Guía**: Security Center asigna una gravedad a las alertas, que le ayudan a priorizar el orden en el que asiste a cada alerta, de modo que cuando un recurso está en peligro, puede obtenerlo inmediatamente. La gravedad se basa en la confianza que tiene Security Center en la búsqueda o en el análisis utilizados para emitir la alerta, así como en el nivel de confianza de que ha habido un intento malintencionado detrás de la actividad que ha provocado la alerta.

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="103-test-security-response-procedures"></a>10.3: Prueba de los procedimientos de respuesta de seguridad

**Instrucciones**: Realice ejercicios para probar las capacidades de respuesta a los incidentes de los sistemas con regularidad. Identifique puntos débiles y brechas y revise el plan según sea necesario.

Consulte la publicación de NIST: Guía para probar, entrenar y ejecutar programas para planes y capacidades de TI: https://nvlpubs.nist.gov/nistpubs/Legacy/SP/nistspecialpublication800-84.pdf

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="104-provide-security-incident-contact-details-and-configure-alert-notifications-nbspfor-security-incidents"></a>10.4: Provisión de detalles de contacto de incidentes de seguridad y configuración de notificaciones de alerta&nbsp; para incidentes de seguridad

**Guía**: la información de contacto del incidente de seguridad la utilizará Microsoft para ponerse en contacto con usted si Microsoft Security Response Center (MSRC) detecta que un tercero no autorizado o ilegal ha accedido a los datos.

Establecimiento del contacto de seguridad de Azure Security Center:

https://docs.microsoft.com/azure/security-center/security-center-provide-security-contact-details

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="105-incorporate-security-alerts-into-your-incident-response-system"></a>10.5: Incorporación de alertas de seguridad en el sistema de respuesta a incidentes

**Instrucciones**: Exporte sus alertas y recomendaciones de Azure Security Center mediante la característica de exportación continua. La exportación continua le permite exportar alertas y recomendaciones de forma manual o continua. Puede usar el conector de datos de Azure Security Center para transmitir las alertas a Azure Sentinel.

Configuración de la exportación continua:

https://docs.microsoft.com/azure/security-center/continuous-export

Transmisión de alertas a Azure Sentinel:

https://docs.microsoft.com/azure/sentinel/connect-azure-security-center

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="106-automate-the-response-to-security-alerts"></a>10.6: Automatización de la respuesta a las alertas de seguridad

**Instrucciones**: use la característica de automatización del flujo de trabajo de Azure Security Center para desencadenar automáticamente respuestas mediante "Logic Apps" en las alertas y recomendaciones de seguridad.

Configuración de la automatización de flujo de trabajo y Logic Apps:

https://docs.microsoft.com/azure/security-center/workflow-automation

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

## <a name="penetration-tests-and-red-team-exercises"></a>Pruebas de penetración y ejercicios del equipo rojo

*Para obtener más información, consulte [Control de seguridad: Pruebas de penetración y ejercicios del equipo rojo](../security/benchmarks/security-control-penetration-tests-red-team-exercises.md).*

### <a name="111-conduct-regular-penetration-testing-of-your-azure-resources-and-ensure-to-remediate-all-critical-security-findings-within-60-days"></a>11.1: Realización de pruebas de penetración periódicas de los recursos de Azure y aseguramiento de la corrección de todos los resultados de seguridad críticos en un plazo de 60 días

**Guía**: siga las reglas de compromiso de Microsoft para asegurarse de que las pruebas de penetración no infrinjan las directivas de Microsoft:

https://www.microsoft.com/msrc/pentest-rules-of-engagement?rtc=1.

Puede encontrar más información sobre la estrategia de Microsoft y la ejecución de las pruebas de penetración de equipo rojo y sitios activos en la nube, los servicios y las aplicaciones administradas por Microsoft aquí: 

https://gallery.technet.microsoft.com/Cloud-Red-Teaming-b837392e

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Compartido

## <a name="next-steps"></a>Pasos siguientes

- Consulte la [prueba comparativa de seguridad de Azure](../security/benchmarks/overview.md).
- Obtenga más información sobre las [líneas de base de seguridad de Azure](../security/benchmarks/security-baselines-overview.md).
