---
title: Base de referencia de seguridad de Azure para Container Instances
description: Base de referencia de seguridad de Azure para Container Instances
author: msmbaldwin
ms.service: security
ms.topic: conceptual
ms.date: 05/27/2020
ms.author: mbaldwin
ms.custom: subject-security-benchmark
ms.openlocfilehash: a26581b61a4b99bd11f48a3d431a1bb85148d66b
ms.sourcegitcommit: 5a3b9f35d47355d026ee39d398c614ca4dae51c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89393435"
---
# <a name="azure-security-baseline-for-container-instances"></a>Base de referencia de seguridad de Azure para Container Instances

La base de referencia de seguridad de Azure para Container Instances que le ayudará a mejorar la posición de seguridad de la implementación.

La base de referencia de este servicio se extrae de la [versión 1.0 de Azure Security Benchmark](../security/benchmarks/overview.md), que proporciona recomendaciones sobre cómo puede proteger las soluciones en la nube en Azure con nuestra guía de procedimientos recomendados.

Para más información, consulte [Introducción a la línea de base de seguridad de Azure](../security/benchmarks/security-baselines-overview.md).

## <a name="network-security"></a>Seguridad de las redes

*Para más información, consulte [Control de seguridad: Seguridad de redes](../security/benchmarks/security-control-network-security.md).*

### <a name="11-protect-resources-using-network-security-groups-or-azure-firewall-on-your-virtual-network"></a>1.1: Proteja los recursos mediante grupos de seguridad de red o Azure Firewall en su red virtual

**Instrucciones**: Azure Virtual Network proporciona seguridad, acceso de red seguro y privado para recursos locales y de Azure. Integre los grupos de contenedores de Azure Container Instances en una red virtual de Azure. 

* [Escenarios y recursos de red virtual: Azure Container Instances](./container-instances-virtual-network-concepts.md)

* [Implementación de instancias de contenedor en una red virtual de Azure](./container-instances-vnet.md)

* [Creación de un NSG con una configuración de seguridad](../virtual-network/tutorial-filter-network-traffic.md)

* [Implementación y configuración de Azure Firewall](../firewall/tutorial-firewall-deploy-portal.md)


**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="12-monitor-and-log-the-configuration-and-traffic-of-vnets-subnets-and-nics"></a>1.2: Supervise y registre la configuración y el tráfico de redes virtuales, subredes y NIC

**Guía**: Use Azure Security Center y siga las recomendaciones de protección de redes para ayudar a proteger sus recursos de red en Azure. Habilite los registros de flujo de NSG y envíe registros a una cuenta de almacenamiento para la auditoría del tráfico.

* [Habilitación de los registros de flujo del grupo de seguridad de red](../network-watcher/network-watcher-nsg-flow-logging-portal.md)

* [Protección de los recursos de red](../security-center/security-center-network-recommendations.md)

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="13-protect-critical-web-applications"></a>1.3: Proteja las aplicaciones web críticas

**Guía**: implemente el firewall de aplicaciones web (WAF) de Azure delante de las aplicaciones web críticas hospedadas en Azure Container Instances para inspeccionar mejor el tráfico entrante. Habilite la configuración de diagnóstico para WAF e ingiera los registros en una cuenta de almacenamiento, un centro de eventos o un área de trabajo de análisis de registros.

* [Implementación de WAF de Azure](../web-application-firewall/ag/create-waf-policy-ag.md)



**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="14-deny-communications-with-known-malicious-ip-addresses"></a>1.4: Deniegue las comunicaciones con direcciones IP malintencionadas conocidas

**Guía**: Habilite la protección contra DDoS estándar en las redes virtuales de Azure para protegerse frente a ataques DDoS. Use la inteligencia sobre amenazas integrada en Azure Security Center para denegar las comunicaciones con direcciones IP malintencionadas conocidas. Implemente Azure Firewall en cada uno de los límites de red de la organización con la inteligencia de amenazas habilitada y configurada para alertar y denegar el tráfico de red malintencionado. Use el acceso de red Just-in-Time de Azure Security Center para configurar los NSG para limitar la exposición de los puntos de conexión a las direcciones IP aprobadas durante un período limitado. Use la protección de redes adaptativa de Azure Security Center para recomendar configuraciones de NSG que limiten los puertos y las direcciones IP de origen según el tráfico real y la inteligencia sobre amenazas. 

* [Implementación de Azure Firewall](../firewall/tutorial-firewall-deploy-portal.md)

* [Descripción de la inteligencia sobre amenazas integrada de Azure Security Center](../security-center/threat-protection.md)

* [Descripción de la protección de red adaptable de Azure Security Center](../security-center/security-center-adaptive-network-hardening.md)

* [Control de acceso de red Just-in-Time de Azure Security Center](../security-center/security-center-just-in-time.md)

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="15-record-network-packets-and-flow-logs"></a>1.5: Registre los paquetes de red y registros de flujo

**Guía**: si usa grupos de seguridad de red (NSG) con su implementación de red virtual, habilite los registros de flujo del grupo de seguridad de red conectado a la subred delegada en Azure Container Instances. Anote los registros de flujo del grupo de seguridad de red en una cuenta de Azure Storage para generar registros de flujo. Si es necesario para investigar actividades anómalas, habilite la captura de paquetes de Azure Network Watcher.

* [Habilitación de los registros de flujo del grupo de seguridad de red](../network-watcher/network-watcher-nsg-flow-logging-portal.md)

* [Habilitación de Network Watcher](../network-watcher/network-watcher-create.md)

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="16-deploy-network-based-intrusion-detectionintrusion-prevention-systems-idsips"></a>1.6: Implemente sistemas de prevención de intrusiones y detección de intrusiones (IDS/IPS) basados en la red

**Guía**: Seleccione una oferta de Azure Marketplace que admita la funcionalidad de IDS/IPS con funcionalidades de inspección de carga. Si la detección y/o la prevención de intrusiones basadas en la inspección de carga no es un requisito, se puede usar Azure Firewall con la inteligencia sobre amenazas. El filtrado basado en inteligencia sobre amenazas de Azure Firewall puede alertar y denegar el tráfico desde y hacia los dominios y las direcciones IP malintencionados conocidos. La direcciones IP y los dominios proceden de la fuente Inteligencia sobre amenazas de Microsoft.

Implemente la solución de firewall que prefiera en cada uno de los límites de red de su organización para detectar y/o denegar el tráfico malintencionado.

* [Azure Marketplace](https://azuremarketplace.microsoft.com/marketplace/?term=Firewall)

* [Implementación de Azure Firewall](../firewall/tutorial-firewall-deploy-portal.md)

* [Configuración de alertas con Azure Firewall](../firewall/threat-intel.md)

* [Implementación en una red virtual: Azure Container Instances](./container-instances-vnet.md) 



**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="17-manage-traffic-to-web-applications"></a>1.7: Administre el tráfico a las aplicaciones web

**Instrucciones**: Implemente Application Gateway de Azure para las aplicaciones web con HTTPS/SSL habilitado para los certificados de confianza.

* [Implementación de Application Gateway](../application-gateway/quick-create-portal.md)

* [Configuración de Application Gateway para usar HTTPS](../application-gateway/create-ssl-portal.md) 

* [Descripción del equilibrio de carga de capa 7 con las puertas de enlace de aplicaciones web de Azure](../application-gateway/overview.md)

* [Exposición de una dirección IP estática para un grupo de contenedores](./container-instances-application-gateway.md)

* [Implementación en una red virtual: Azure Container Instances](./container-instances-vnet.md)



**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="18-minimize-complexity-and-administrative-overhead-of-network-security-rules"></a>1.8: Minimice la complejidad y la sobrecarga administrativa de las reglas de seguridad de red

**Guía**: Puede usar etiquetas de servicio de Virtual Network para definir controles de acceso a la red en grupos de seguridad de red o Azure Firewall. Puede utilizar etiquetas de servicio en lugar de direcciones IP específicas al crear reglas de seguridad. Al especificar el nombre de la etiqueta de servicio (por ejemplo, ApiManagement) en el campo de origen o destino apropiado de una regla, puede permitir o denegar el tráfico para el servicio correspondiente. Microsoft administra los prefijos de direcciones que la etiqueta de servicio incluye y actualiza automáticamente dicha etiqueta a medida que las direcciones cambian. 

También puede usar grupos de seguridad de aplicaciones para simplificar una configuración de seguridad compleja. Los grupos de seguridad de aplicaciones le permiten configurar la seguridad de red como una extensión natural de la estructura de una aplicación, lo que le permite agrupar máquinas virtuales y directivas de seguridad de red basadas en esos grupos. 

* [Descripción y uso de las etiquetas de servicio](../virtual-network/service-tags-overview.md) 

* [Descripción y uso de los grupos de seguridad de aplicaciones](../virtual-network/security-overview.md#application-security-groups)

* [Implementación en una red virtual: Azure Container Instances](./container-instances-vnet.md)



**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="19-maintain-standard-security-configurations-for-network-devices"></a>1.9: Mantenga las configuraciones de seguridad estándar para dispositivos de red

**Guía**: También puede usar Azure Blueprints para simplificar las implementaciones de Azure a gran escala mediante el empaquetado de artefactos de entorno clave, como plantillas de Azure Resource Manager, controles de Azure RBAC y directivas, en una única definición de plano técnico. Puede aplicar el plano técnico a nuevas suscripciones y ajustar el control y la administración a través del control de versiones. 

* [Configuración y administración de Azure Policy](../governance/policy/tutorials/create-and-manage.md)

* [Ejemplos de Azure Policy para redes](/azure/governance/policy/samples/#network)

* [Creación de un plano técnico de Azure](../governance/blueprints/create-blueprint-portal.md)



**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="110-document-traffic-configuration-rules"></a>1.10: Documente las reglas de configuración de tráfico

**Guía**: Use etiquetas para NSG y otros recursos relacionados con la seguridad de red y el flujo de tráfico. En el caso de las reglas de NSG individuales, use el campo "Descripción" para especificar las necesidades empresariales o la duración (etc.) de las reglas que permiten que entre o salga el tráfico en una red.

Use cualquiera de las definiciones de directiva de Azure integradas relacionadas con el etiquetado, como "Requerir etiqueta y su valor" para asegurarse de que todos los recursos se crean con etiquetas y para notificarle los recursos no etiquetados existentes.

Puede usar Azure PowerShell o la CLI de Azure para buscar o realizar acciones en los recursos en función de sus etiquetas.

* [Creación y uso de etiquetas](../azure-resource-manager/management/tag-resources.md)

* [Implementación en una red virtual: Azure Container Instances](./container-instances-vnet.md)

* [Creación de un grupo de seguridad de red con una configuración de seguridad](../virtual-network/tutorial-filter-network-traffic.md)


**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="111-use-automated-tools-to-monitor-network-resource-configurations-and-detect-changes"></a>1.11: Use herramientas automatizadas para supervisar las configuraciones de recursos de red y detectar cambios

**Guía**: use el registro de actividad de Azure para supervisar las configuraciones de los recursos de red y detectar cambios en ellos relacionados con las instancias de contenedor. Cree alertas en Azure Monitor que se desencadenarán cuando se produzcan cambios en los recursos de red críticos.

* [Visualización y recuperación de eventos del registro de actividad de Azure](../azure-monitor/platform/activity-log.md#view-the-activity-log)

* [Creación de alertas en Azure Monitor](../azure-monitor/platform/alerts-activity-log.md)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

## <a name="logging-and-monitoring"></a>Registro y supervisión

*Para más información, consulte [Control de seguridad: registro y supervisión](../security/benchmarks/security-control-logging-monitoring.md).*

### <a name="21-use-approved-time-synchronization-sources"></a>2.1: Uso de orígenes de sincronización de hora aprobados

**Guía**: Microsoft mantiene los orígenes de hora de los recursos de Azure; sin embargo, tiene la opción de administrar la configuración de la sincronización de hora de los recursos de proceso. Por ejemplo, ejecute un comando de sincronización de hora en un contenedor en ejecución.

* [Configuración de la sincronización de hora de los recursos de proceso de Azure](../virtual-machines/windows/time-sync.md)

* [Ejecución de un comando en una instancia de contenedor de Azure en ejecución](./container-instances-exec.md)



**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Microsoft

### <a name="22-configure-central-security-log-management"></a>2.2: Configuración de la administración central de registros de seguridad

**Guía**: ingiera registros a través de Azure Monitor para agregar datos de seguridad generados por un grupo de contenedores de Azure. En Azure Monitor, use áreas de trabajo de Log Analytics para realizar consultas y análisis, y utilice cuentas de Azure Storage para el almacenamiento de archivos a largo plazo.

* [Registro de instancias y grupos de contenedores con registros de Azure Monitor](./container-instances-log-analytics.md)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="23-enable-audit-logging-for-azure-resources"></a>2.3: Habilitación del registro de auditoría para recursos de Azure

**Instrucciones**: Azure Monitor recopila registros de recursos (anteriormente denominados registros de diagnóstico) de eventos controlados por el usuario en el registro. Azure Container Instances incluye compatibilidad integrada con el envío de registros de grupo de contenedores, datos de eventos y registros de contenedor a registros de Azure Monitor.

* [Registro de instancias y grupos de contenedores con registros de Azure Monitor](../container-registry/container-registry-diagnostics-audit-logs.md)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="24-collect-security-logs-from-operating-systems"></a>2.4: Recopilación de registros de seguridad de sistemas operativos

**Guía**: No aplicable. Esta guía está pensada para recursos de proceso IaaS.


**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="25-configure-security-log-storage-retention"></a>2.5: Configuración de la retención del almacenamiento de registros de seguridad

**Guía**: En Azure Monitor, establezca el período de retención del área de trabajo de Log Analytics de acuerdo con la normativa de cumplimiento de su organización. Use cuentas de Azure Storage para el almacenamiento de archivos a largo plazo.

* [Configuración de parámetros de retención de registros de áreas de trabajo de Log Analytics](../azure-monitor/platform/manage-cost-storage.md#change-the-data-retention-period)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="26-monitor-and-review-logs"></a>2.6: Supervisión y registros de revisión

**Guía**: Analice y supervise los registros en busca de comportamientos anómalos y revise los resultados con regularidad. Use el área de trabajo de Log Analytics de Azure Monitor para revisar los registros y realizar consultas en los datos del registro. 

* [Registro de instancias y grupos de contenedores con registros de Azure Monitor](./container-instances-log-analytics.md)

* [Descripción del área de trabajo de Log Analytics](../azure-monitor/log-query/get-started-portal.md)

* [Procedimiento para realizar consultas personalizadas en Azure Monitor](../azure-monitor/log-query/get-started-queries.md)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="27-enable-alerts-for-anomalous-activity"></a>2.7: Habilitación de alertas para actividades anómalas

**Guía**: use el área de trabajo de Log Analytics para supervisar las actividades anómalas encontradas en los registros y eventos de seguridad y avisar sobre ellas. 

* [Registro de instancias y grupos de contenedores con registros de Azure Monitor](./container-instances-log-analytics.md)

* [Alertas sobre datos de registro de Log Analytics](../azure-monitor/learn/tutorial-response.md)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="28-centralize-anti-malware-logging"></a>2.8: Centralización del registro antimalware

**Guía**: si es necesario, proporcione su propia solución antimalware y recopilación de eventos para ejecutarlos en un contenedor. 


**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="29-enable-dns-query-logging"></a>2.9: Habilitación del registro de consultas DNS

**Guía**: si es necesario, proporcione su propia solución para consultar los registros DNS en un contenedor.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="210-enable-command-line-audit-logging"></a>2.10: Habilitación del registro de auditoría de la línea de comandos

**Guía**: si es necesario, configure el registro de la consola en una instancia de contenedor en ejecución.

* [Ejecución de un comando en una instancia de contenedor de Azure en ejecución](./container-instances-exec.md)



**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

## <a name="identity-and-access-control"></a>Control de identidades y acceso

*Para más información, consulte [Control de seguridad: Identidad y control de acceso](../security/benchmarks/security-control-identity-access-control.md).*

### <a name="31-maintain-an-inventory-of-administrative-accounts"></a>3.1: Mantenga un inventario de cuentas administrativas

**Guía**: Azure Active Directory (Azure AD) tiene roles integrados que se deben asignar explícitamente y son consultables. Use el módulo de PowerShell de Azure AD para realizar consultas ad hoc para detectar cuentas que son miembros de grupos administrativos.

Si usa un registro de contenedor de Azure con Azure Container Instances, para cada registro de contenedor de Azure, compruebe si la cuenta de administrador integrada está habilitada o deshabilitada. Deshabilite la cuenta cuando no esté en uso.

* [Obtención de un rol de directorio en Azure AD con PowerShell](/powershell/module/azuread/get-azureaddirectoryrole?view=azureadps-2.0)

* [Obtención de los miembros de un rol de directorio en Azure AD con PowerShell](/powershell/module/azuread/get-azureaddirectoryrolemember?view=azureadps-2.0)

* [Cuenta de administrador de Azure Container Registry](../container-registry/container-registry-authentication.md#admin-account)

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="32-change-default-passwords-where-applicable"></a>3.2: Cambie las contraseñas predeterminadas cuando proceda

**Guía**: Azure Active Directory (Azure AD) no tiene el concepto de contraseñas predeterminadas. Otros recursos de Azure que requieren una contraseña obligan a crear una contraseña con requisitos de complejidad y una longitud mínima, que cambia en función del servicio. Es responsable de las aplicaciones de terceros y de los servicios de Marketplace que pueden usar contraseñas predeterminadas.

Si usa un registro de contenedor de Azure con Azure Container Instances y la cuenta de administrador predeterminada de dicho registro está habilitada, se crean automáticamente contraseñas complejas, que se deben rotar. Deshabilite la cuenta cuando no esté en uso.

* [Cuenta de administrador de Azure Container Registry](../container-registry/container-registry-authentication.md#admin-account)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="33-use-dedicated-administrative-accounts"></a>3.3: Use cuentas administrativas dedicadas

**Guía**: Cree procedimientos operativos estándar en torno al uso de cuentas administrativas dedicadas. Use la administración de identidad y acceso de Azure Security Center para supervisar el número de cuentas administrativas.

Si usa un registro de contenedor de Azure con Azure Container Instances, cree procedimientos para habilitar la cuenta de administrador integrada de un registro de contenedor. Deshabilite la cuenta cuando no esté en uso.

* [Descripción de identidad y acceso en Azure Security Center](../security-center/security-center-identity-access.md)

* [Cuenta de administrador de Azure Container Registry](../container-registry/container-registry-authentication.md#admin-account)

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="34-use-single-sign-on-sso-with-azure-active-directory"></a>3.4: Use el inicio de sesión único (SSO) con Azure Active Directory

**Instrucciones**: Siempre que sea posible, use el SSO de Azure Active Directory en lugar de configurar credenciales independientes individuales por servicio. Use las recomendaciones de administración de identidades y acceso de Azure Security Center.

* [Descripción del SSO con Azure AD](../active-directory/manage-apps/what-is-single-sign-on.md)


**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="35-use-multi-factor-authentication-for-all-azure-active-directory-based-access"></a>3.5: Use la autenticación multifactor para todo el acceso basado en Azure Active Directory

**Guía**: Habilite la autenticación multifactor (MFA) de Azure Active Directory (Azure AD) y siga las recomendaciones de administración de identidades y acceso de Azure Security Center.

* [Procedimiento para habilitar la MFA en Azure](../active-directory/authentication/howto-mfa-getstarted.md)

* [Supervisión de la identidad y el acceso en Azure Security Center](../security-center/security-center-identity-access.md)

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="36-use-dedicated-machines-privileged-access-workstations-for-all-administrative-tasks"></a>3.6: Use máquinas dedicadas (estaciones de trabajo de acceso con privilegios) para todas las tareas administrativas

**Guía**: Utilice PAW (estaciones de trabajo de acceso privilegiado) con la MFA configurada para iniciar la sesión y configurar recursos de Azure.

* [Más información sobre las estaciones de trabajo con privilegios de acceso](/windows-server/identity/securing-privileged-access/privileged-access-workstations)

* [Procedimiento para habilitar la MFA en Azure](../active-directory/authentication/howto-mfa-getstarted.md)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="37-log-and-alert-on-suspicious-activity-from-administrative-accounts"></a>3.7: Registre y alerte de la actividad sospechosa desde cuentas administrativas

**Guía**: Use los informes de seguridad de Azure Active Directory (Azure AD) para la generación de registros y alertas cuando se produzcan actividades sospechosas o no seguras en el entorno. Use Azure Security Center para supervisar la actividad de identidad y acceso.

* [Procedimiento para identificar usuarios de Azure AD marcados por una actividad de riesgo](../active-directory/identity-protection/overview-identity-protection.md)

* [Supervisión de la actividad de identidad y acceso de los usuarios en Azure Security Center](../security-center/security-center-identity-access.md)

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="38-manage-azure-resources-from-only-approved-locations"></a>3.8: Administre los recursos de Azure solo desde ubicaciones aprobadas

**Guía**: Use ubicaciones con nombre de acceso condicional para permitir el acceso solo desde agrupaciones lógicas específicas de intervalos de direcciones IP o países o regiones.

* [Configuración de ubicaciones con nombre en Azure](../active-directory/reports-monitoring/quickstart-configure-named-locations.md)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="39-use-azure-active-directory"></a>3.9: Uso de Azure Active Directory

**Instrucciones**: Use Azure Active Directory (AD) como sistema central de autenticación y autorización. Azure AD protege los datos mediante un cifrado seguro para los datos en reposo y en tránsito. Azure AD también cifra con sal, convierte en hash y almacena de forma segura las credenciales de los usuarios.

* [Procedimiento para crear y configurar una instancia de Azure AD](../active-directory/fundamentals/active-directory-access-create-new-tenant.md)

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="310-regularly-review-and-reconcile-user-access"></a>3.10: Revise y concilie regularmente el acceso de los usuarios

**Instrucciones**: Azure Active Directory (Azure AD) proporciona registros para ayudar a descubrir cuentas obsoletas. Además, use las revisiones de acceso de identidad de Azure para administrar de forma eficiente las pertenencias a grupos, el acceso a las aplicaciones empresariales y las asignaciones de roles. El acceso de los usuarios se puede revisar de forma periódica para asegurarse de que solo las personas adecuadas tengan acceso continuado.

* [Descripción de los informes de Azure AD](../active-directory/reports-monitoring/index.yml)

* [Procedimiento para usar las revisiones de acceso de identidad de Azure](../active-directory/governance/access-reviews-overview.md)

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="311-monitor-attempts-to-access-deactivated-accounts"></a>3.11: Supervise los intentos de acceso a cuentas desactivadas

**Guía**: Puede usar orígenes de registro de actividad de inicio de sesión de Azure Active Directory (Azure AD), auditoría y riesgos para la supervisión, lo que permite realizar la integración con cualquier herramienta de administración de eventos e información de seguridad (SIEM) y de supervisión.

Puede simplificar este proceso creando una configuración de diagnóstico para las cuentas de usuario de Azure Active Directory y enviando los registros de auditoría y los registros de inicio de sesión a un área de trabajo de Log Analytics. Puede configurar las alertas deseadas en el área de trabajo de Log Analytics.

* [Integración de los registros de actividad de Azure en Azure Monitor](../active-directory/reports-monitoring/howto-integrate-activity-logs-with-log-analytics.md)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="312-alert-on-account-login-behavior-deviation"></a>3.12: Alerte de las desviaciones de comportamiento en los inicios de sesión de las cuentas

**Guía**: Use las características de protección de identidad y detección de riesgo de Azure Active Directory (Azure AD) para configurar respuestas automatizadas a las acciones sospechosas que se detecten relacionadas con las identidades de los usuarios.

* [Visualización de los inicios de sesión de riesgo de Azure AD](../active-directory/identity-protection/overview-identity-protection.md)

* [Configuración y habilitación de las directivas de riesgo de protección de identidad](../active-directory/identity-protection/howto-identity-protection-configure-risk-policies.md)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="313-provide-microsoft-with-access-to-relevant-customer-data-during-support-scenarios"></a>3.13: Proporcione a Microsoft acceso a los datos pertinentes del cliente durante los escenarios de soporte técnico

**Guía**: no está disponible en este momento; Caja de seguridad del cliente no es compatible actualmente con Azure Container Instances.

* [Lista de servicios admitidos por la Caja de seguridad del cliente](../security/fundamentals/customer-lockbox-overview.md#supported-services-and-scenarios-in-general-availability)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

## <a name="data-protection"></a>Protección de los datos

*Para más información, consulte [Control de seguridad: Protección de datos](../security/benchmarks/security-control-data-protection.md).*

### <a name="41-maintain-an-inventory-of-sensitive-information"></a>4.1: Mantenimiento de un inventario de información confidencial

**Guía**: use etiquetas de recursos para ayudar a realizar el seguimiento de las instancias de contenedor de Azure que almacenan o procesan información confidencial. 

Aplique control de versiones a las imágenes de contenedor y etiquételas para ayudar a realizar el seguimiento de las imágenes que almacenan o procesan información confidencial.

* [Creación y uso de etiquetas](../azure-resource-manager/management/tag-resources.md)

* [Recomendaciones para el etiquetado y el control de versiones de las imágenes de contenedor](../container-registry/container-registry-image-tag-version.md)


**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="42-isolate-systems-storing-or-processing-sensitive-information"></a>4.2: Aislamiento de los sistemas que almacenan o procesan información confidencial

**Guía**: implemente suscripciones o grupos de administración independientes para los entornos de desarrollo, pruebas y producción. Los recursos deben separarse por red virtual o subred, etiquetarse adecuadamente y estar protegidos por un grupo de seguridad de red o Azure Firewall. Los recursos que almacenan o procesan datos confidenciales deben estar suficientemente aislados.

* [Ejecución de un comando en una instancia de contenedor de Azure en ejecución](./container-instances-exec.md)

* [Creación de suscripciones adicionales de Azure](../cost-management-billing/manage/create-subscription.md)

* [Creación de grupos de administración](../governance/management-groups/create.md)

* [Creación y uso de etiquetas](../azure-resource-manager/management/tag-resources.md)

* [Implementación en una red virtual: Azure Container Instances](./container-instances-vnet.md) 
* [Creación de un grupo de seguridad de red con una configuración de seguridad](../virtual-network/tutorial-filter-network-traffic.md)

* [Implementación de Azure Firewall](../firewall/tutorial-firewall-deploy-portal.md)

* [Configuración de alertas o alertas y denegación con Azure Firewall](../firewall/threat-intel.md)


**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="43-monitor-and-block-unauthorized-transfer-of-sensitive-information"></a>4.3: Supervisión y bloqueo de una transferencia no autorizada de información confidencial

**Guía**: Implemente una herramienta automatizada en los perímetros de red que supervise la transferencia no autorizada de información confidencial y bloquee dichas transferencias al tiempo que alerta a los profesionales de seguridad de la información. Supervise y bloquee la transferencia de información no autorizada desde recursos compartidos de archivos de Azure y otros volúmenes montados en instancias de contenedor.

En el caso de la plataforma subyacente administrada por Microsoft, Microsoft trata todo el contenido de los clientes como confidencial y hace grandes esfuerzos para proteger a los clientes contra la pérdida y exposición de sus datos. Para garantizar la seguridad de los datos de los clientes dentro de Azure, Microsoft ha implementado y mantiene un conjunto de controles y funcionalidades eficaces de protección de datos.

* [Descripción de la protección de datos de los clientes en Azure](../security/fundamentals/protection-customer-data.md)

* [Implementación en una red virtual: Azure Container Instances](./container-instances-vnet.md) 

* [Montaje de un recurso compartido de archivos de Azure en Azure Container Instances](./container-instances-volume-azure-files.md)


**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Compartido

### <a name="44-encrypt-all-sensitive-information-in-transit"></a>4.4: Cifrado de toda la información confidencial en tránsito

**Instrucciones**: asegúrese de que los clientes que se conectan a los grupos de contenedores de Azure pueden negociar TLS 1.2 o superior. De forma predeterminada, los recursos de Microsoft Azure negociarán TLS 1.2.

Siga las recomendaciones de Azure Security Center para el cifrado en reposo y el cifrado en tránsito, si procede.

* [Descripción del cifrado en tránsito con Azure](../security/fundamentals/encryption-overview.md#encryption-of-data-in-transit)

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Compartido

### <a name="45-use-an-active-discovery-tool-to-identify-sensitive-data"></a>4.5: Uso de una herramienta de detección activa para identificar datos confidenciales

**Guía**: las características de identificación, clasificación y prevención de pérdida de datos no están disponibles actualmente en Azure Container Instances. Etiquete los grupos de contenedores que podrían procesar información confidencial e implemente una solución de terceros, si se requiere por motivos de cumplimiento.

En el caso de la plataforma subyacente administrada por Microsoft, Microsoft trata todo el contenido de los clientes como confidencial y hace grandes esfuerzos para proteger a los clientes contra la pérdida y exposición de sus datos. Para garantizar la seguridad de los datos de los clientes dentro de Azure, Microsoft ha implementado y mantiene un conjunto de controles y funcionalidades eficaces de protección de datos.

* [Descripción de la protección de datos de los clientes en Azure](../security/fundamentals/protection-customer-data.md)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Compartido

### <a name="46-use-azure-rbac-to-control-access-to-resources"></a>4.6: Uso de RBAC de Azure para controlar el acceso a los recursos

**Guía**: Utilice el control de acceso basado en rol de Azure (Azure RBAC) para controlar el acceso a los datos y recursos de Azure Container Instances. 

* [Configuración de Azure RBAC](../role-based-access-control/role-assignments-portal.md)


**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="47-use-host-based-data-loss-prevention-to-enforce-access-control"></a>4.7: Uso de la prevención de pérdida de datos basada en host para aplicar el control de acceso

**Guía**: no aplicable; esta recomendación está diseñada para recursos de proceso de IaaS.

En el caso de la plataforma subyacente administrada por Microsoft, Microsoft trata todo el contenido de los clientes como confidencial y hace grandes esfuerzos para proteger a los clientes contra la pérdida y exposición de sus datos. Para garantizar la seguridad de los datos de los clientes dentro de Azure, Microsoft ha implementado y mantiene un conjunto de controles y funcionalidades eficaces de protección de datos.

* [Descripción de la protección de datos de los clientes en Azure](../security/fundamentals/protection-customer-data.md)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Compartido

### <a name="48-encrypt-sensitive-information-at-rest"></a>4.8: Cifrado de información confidencial en reposo

**Guía**: de forma predeterminada, todos los datos de implementación de Azure Container Instances se cifran en reposo mediante claves administradas por Microsoft. También tiene la opción de administrar el cifrado con su propia clave (clave administrada por el cliente).

* [Descripción del cifrado en reposo en Azure](../security/fundamentals/encryption-atrest.md)

* [Cifrado de datos de implementación con Azure Container Instances](./container-instances-encrypt-data.md)



**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="49-log-and-alert-on-changes-to-critical-azure-resources"></a>4.9: Registro y alerta de cambios en los recursos críticos de Azure

**Instrucciones**: use Azure Monitor con el registro de actividad de Azure para crear alertas para cuando se produzcan cambios en los grupos de contenedores y las instancias de contenedor. 

* [Creación de alertas para los eventos del registro de actividad de Azure](../azure-monitor/platform/alerts-activity-log.md)


**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

## <a name="vulnerability-management"></a>Administración de vulnerabilidades

*Para más información, consulte [Control de seguridad: Administración de vulnerabilidades](../security/benchmarks/security-control-vulnerability-management.md).*

### <a name="51-run-automated-vulnerability-scanning-tools"></a>5.1: Ejecute herramientas de análisis de vulnerabilidades automatizado

**Guía**: implemente soluciones para examinar imágenes de contenedor de un registro privado e identificar las posibles vulnerabilidades. Siga las recomendaciones de Azure Security Center sobre cómo realizar evaluaciones de vulnerabilidades en las imágenes de contenedor almacenadas en Azure Container Registry. Opcionalmente, puede implementar soluciones de terceros de Azure Marketplace para realizar evaluaciones de vulnerabilidades de imagen.

* [Consideraciones de seguridad para Azure Container Instances](./container-instances-image-security.md)

* [Integración de Azure Container Registry con Security Center](../security-center/azure-container-registry-integration.md)

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="52-deploy-automated-operating-system-patch-management-solution"></a>5.2: Implemente una solución de administración de revisiones de sistema operativo automatizada

**Instrucciones**: Microsoft realiza la administración de revisiones en los sistemas subyacentes que admiten contenedores en ejecución.

Use una solución personalizada o de terceros para aplicar revisiones a las imágenes de contenedor. Si almacena imágenes de contenedor en Azure Container Registry, ejecute tareas de Azure Container Registry para automatizar las actualizaciones de las imágenes de aplicación de un registro de contenedor en función de las revisiones de seguridad u otras actualizaciones de las imágenes base del sistema operativo.

* [Consideraciones de seguridad para Azure Container Instances](./container-instances-image-security.md)

* [Acerca de las actualizaciones de imagen base para ACR Tasks](../container-registry/container-registry-tasks-base-images.md)


**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Compartido

### <a name="53-deploy-automated-third-party-software-patch-management-solution"></a>5.3: Implemente una solución automatizada de administración de revisiones de software de terceros

**Guía**: use una solución personalizada o de terceros para aplicar revisiones a las imágenes de contenedor. Si almacena imágenes de contenedor en Azure Container Registry, ejecute tareas de Azure Container Registry para automatizar las actualizaciones de las imágenes de aplicación de un registro de contenedor en función de las revisiones de seguridad u otras actualizaciones de las imágenes base del sistema operativo.

* [Consideraciones de seguridad para Azure Container Instances](./container-instances-image-security.md)

* [Acerca de las actualizaciones de imagen base para ACR Tasks](../container-registry/container-registry-tasks-base-images.md)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="54-compare-back-to-back-vulnerability-scans"></a>5.4: Compare los exámenes de vulnerabilidades opuestos

**Guía**: exporte los resultados del examen de las imágenes en intervalos coherentes y compare los resultados para comprobar que se han corregido las vulnerabilidades. Si almacena imágenes de contenedor en Azure Container Registry, integre el registro con Azure Security Center para permitir el análisis periódico de las imágenes de contenedor a fin de detectar vulnerabilidades. Opcionalmente, puede implementar soluciones de terceros de Azure Marketplace para realizar evaluaciones de vulnerabilidades de imagen.

* [Consideraciones de seguridad para Azure Container Instances](./container-instances-image-security.md)

* [Integración de Azure Container Registry con Security Center](../security-center/azure-container-registry-integration.md)

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="55-use-a-risk-rating-process-to-prioritize-the-remediation-of-discovered-vulnerabilities"></a>5.5: Use un proceso de clasificación de riesgos para priorizar la corrección de las vulnerabilidades detectadas

**Guía**:  si almacena imágenes de contenedor en Azure Container Registry, integre el registro con Azure Security Center para permitir el examen periódico de las imágenes de contenedor a fin de detectar las vulnerabilidades y clasificar los riesgos. Opcionalmente, puede implementar soluciones de terceros de Azure Marketplace para realizar evaluaciones de vulnerabilidades de imagen y clasificación de riesgos.

* [Consideraciones de seguridad para Azure Container Instances](./container-instances-image-security.md)

* [Integración de Azure Container Registry con Security Center](../security-center/azure-container-registry-integration.md)

**Supervisión de Azure Security Center**: Sí

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

**Guía**: aplique etiquetas a Azure Container Instances y recursos relacionados y proporcione metadatos para organizarlos de forma lógica en una taxonomía.

* [Creación y uso de etiquetas](../azure-resource-manager/management/tag-resources.md)


**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="63-delete-unauthorized-azure-resources"></a>6.3: Eliminación de recursos de Azure no autorizados

**Instrucciones**: Use el etiquetado, los grupos de administración y las suscripciones independientes, si procede, para organizar y realizar un seguimiento de los recursos. Concilie el inventario periódicamente y asegúrese de que los recursos no autorizados se eliminan de la suscripción de manera oportuna.

* [Creación de suscripciones adicionales de Azure](../cost-management-billing/manage/create-subscription.md)

* [Creación de grupos de administración](../governance/management-groups/create.md)

* [Creación y uso de etiquetas](../azure-resource-manager/management/tag-resources.md)


**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="64-maintain-an-inventory-of-approved-azure-resources-and-software-titles"></a>6.4: Mantenimiento de un inventario de los recursos de Azure aprobados y títulos de software

**Guía**: Tendrá que crear un inventario de los recursos de Azure aprobados y el software aprobado para los recursos de proceso según las necesidades de la organización.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="65-monitor-for-unapproved-azure-resources"></a>6.5: Supervisión de recursos de Azure no aprobados

**Guía**: Use Azure Policy para establecer restricciones en el tipo de recursos que se pueden crear en sus suscripciones.

Use Azure Resource Graph para consultar o detectar recursos dentro de sus suscripciones. Asegúrese de que todos los recursos de Azure presentes en el entorno estén aprobados.

* [Auditoría del cumplimiento de los registros de contenedor de Azure mediante Azure Policy](../container-registry/container-registry-azure-policy.md)

* [Configuración y administración de Azure Policy](../governance/policy/tutorials/create-and-manage.md)

* [Creación de consultas con Azure Graph](../governance/resource-graph/first-query-portal.md)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="66-monitor-for-unapproved-software-applications-within-compute-resources"></a>6.6: Supervisión de aplicaciones de software no aprobadas en recursos de proceso

**Guía**: implemente su propia solución, o una solución de terceros, para inventariar el software de las aplicaciones en contenedores aprobadas. 

implemente soluciones para examinar imágenes de contenedor de un registro privado e identificar las posibles vulnerabilidades. Siga las recomendaciones de Azure Security Center para realizar evaluaciones de vulnerabilidades en las imágenes de contenedor almacenadas en Azure Container Registry. Opcionalmente, puede implementar soluciones de terceros de Azure Marketplace para realizar evaluaciones de vulnerabilidades de imagen.

Supervise los registros de Azure Container Instances en busca de comportamientos anómalos y revise los resultados con regularidad. Use el área de trabajo de Log Analytics de Azure Monitor para revisar los registros y realizar consultas en los datos del registro.

* [Registro de instancias y grupos de contenedores con registros de Azure Monitor](./container-instances-log-analytics.md)

* [Descripción del área de trabajo de Log Analytics](../azure-monitor/log-query/get-started-portal.md)

* [Procedimiento para realizar consultas personalizadas en Azure Monitor](../azure-monitor/log-query/get-started-queries.md)

* [Consideraciones de seguridad para Azure Container Instances](./container-instances-image-security.md)
* [Integración de Azure Container Registry con Security Center](../security-center/azure-container-registry-integration.md)



**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="67-remove-unapproved-azure-resources-and-software-applications"></a>6.7: Eliminación de aplicaciones de software y recursos de Azure no aprobadas

**Guía**: Azure Automation proporciona un control completo durante la implementación, las operaciones y la retirada de las cargas de trabajo y recursos. Puede implementar su propia solución para quitar aplicaciones de software y recursos de Azure no autorizados.

* [Introducción a Azure Automation](../automation/automation-intro.md)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="68-use-only-approved-applications"></a>6.8: Uso exclusivo de aplicaciones aprobadas

**Guía**: aplique control de versiones a las imágenes de contenedor y etiquételas para ayudar a realizar el seguimiento de las imágenes que se ejecutan en aplicaciones aprobadas.
* [Recomendaciones para el etiquetado y el control de versiones de las imágenes de contenedor](../container-registry/container-registry-image-tag-version.md)


**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="69-use-only-approved-azure-services"></a>6.9: Uso exclusivo de servicios de Azure aprobados

**Guía**: Use Azure Policy para restringir qué servicios puede aprovisionar en su entorno.

* [Configuración y administración de Azure Policy](../governance/policy/tutorials/create-and-manage.md)

* [Denegación de un tipo de recurso específico con Azure Policy](/azure/governance/policy/samples/not-allowed-resource-types)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="610-implement-approved-application-list"></a>6.10: Implementación de la lista de aplicaciones aprobadas

**Guía**: aplique control de versiones a las imágenes de contenedor y etiquételas para ayudar a realizar el seguimiento de las imágenes que se ejecutan en aplicaciones aprobadas.
* [Recomendaciones para el etiquetado y el control de versiones de las imágenes de contenedor](../container-registry/container-registry-image-tag-version.md)


**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="611-divlimit-users-ability-to-interact-with-azure-resource-manager-via-scriptsbrdiv"></a>6.11: <div>Limitación de la capacidad de los usuarios para interactuar con Azure Resource Manager a través de scripts<br></div>

**Instrucciones**: Use el acceso condicional de Azure para limitar la capacidad de los usuarios de interactuar con Azure Resource Manager mediante la configuración de la opción "Bloquear acceso" en la aplicación "Administración de Microsoft Azure". 

* [Configuración del acceso condicional para bloquear el acceso a Azure Resource Manager](../role-based-access-control/conditional-access-azure-management.md)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="612-limit-users-ability-to-execute-scripts-within-compute-resources"></a>6.12: Limitación de capacidad de los usuarios para ejecutar scripts en recursos de proceso

**Guía**: todos los usuarios con acceso a Azure Container Instances pueden ejecutar scripts dentro de contenedores.

Administre los recursos de Azure Container Instances y revise su acceso mediante distintas suscripciones o grupos de administración de Azure; también puede aislar los recursos mediante redes virtuales y grupos de seguridad de red o Azure Firewall.

* [Ejecución de un comando en una instancia de contenedor de Azure en ejecución](./container-instances-exec.md)

* [Creación de suscripciones adicionales de Azure](../cost-management-billing/manage/create-subscription.md)

* [Creación de grupos de administración](../governance/management-groups/create.md)

* [Implementación en una red virtual: Azure Container Instances](./container-instances-vnet.md)

* [Creación de un grupo de seguridad de red con una configuración de seguridad](../virtual-network/tutorial-filter-network-traffic.md)

* [Implementación de Azure Firewall](../firewall/tutorial-firewall-deploy-portal.md)



**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="613-physically-or-logically-segregate-high-risk-applications"></a>6.13: Segregación física o lógica de aplicaciones de alto riesgo

**Guía**: el software necesario para las operaciones empresariales, pero que puede suponer un riesgo mayor para la organización, debe aislarse en su propia red virtual y estar lo suficientemente protegida con Azure Firewall o un grupo de seguridad de red.

* [Implementación en una red virtual: Azure Container Instances](./container-instances-vnet.md) 

* [Creación de un grupo de seguridad de red con una configuración de seguridad](../virtual-network/tutorial-filter-network-traffic.md)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

## <a name="secure-configuration"></a>Configuración segura

*Para más información, consulte [Control de seguridad: Configuración segura](../security/benchmarks/security-control-secure-configuration.md).*

### <a name="71-establish-secure-configurations-for-all-azure-resources"></a>7.1: Establezca configuraciones seguras para todos los recursos de Azure

**Guía**: mantenga una configuración de grupo de contenedores aprobada mediante una plantilla de Azure Resource Manager o la exportación a un archivo YAML. Use Azure Policy para mantener las configuraciones de seguridad de los recursos de Azure relacionados.

* [Grupos de contenedores en Azure Container Instances](container-instances-container-groups.md#deployment)

* [Configuración y administración de Azure Policy](../governance/policy/tutorials/create-and-manage.md)


**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="72-establish-secure-operating-system-configurations"></a>7.2: Establezca configuraciones del sistema operativo seguras

**Guía**: use una solución personalizada o de terceros para aplicar revisiones a las imágenes de contenedor. Si almacena imágenes de contenedor en Azure Container Registry, ejecute tareas de Azure Container Registry para automatizar las actualizaciones de las imágenes de aplicación de un registro de contenedor en función de las revisiones de seguridad u otras actualizaciones de las imágenes base del sistema operativo. 

* [Acerca de las actualizaciones de imagen base para ACR Tasks](../container-registry/container-registry-tasks-base-images.md)



**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="73-maintain-secure-azure-resource-configurations"></a>7.3: Mantenga configuraciones de recursos de Azure seguras

**Instrucciones**: Use la directiva de Azure [denegar] e [implementar si no existe] para aplicar la configuración segura en los recursos de Azure.

* [Configuración y administración de Azure Policy](../governance/policy/tutorials/create-and-manage.md)

* [Descripción de los efectos de Azure Policy](../governance/policy/concepts/effects.md)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="74-maintain-secure-operating-system-configurations"></a>7.4: Mantenga configuraciones del sistema operativo seguras

**Instrucciones**: implemente soluciones para examinar las imágenes de contenedor de un registro privado e identificar posibles vulnerabilidades en las configuraciones del sistema operativo. Siga las recomendaciones de Azure Security Center para realizar evaluaciones de vulnerabilidades en las imágenes de contenedor almacenadas en Azure Container Registry. Opcionalmente, puede implementar soluciones de terceros de Azure Marketplace para realizar evaluaciones de vulnerabilidades de imagen.

Use una solución personalizada o de terceros para aplicar revisiones a las imágenes de contenedor. Si almacena imágenes de contenedor en Azure Container Registry, ejecute tareas de Azure Container Registry para automatizar las actualizaciones de las imágenes de aplicación de un registro de contenedor en función de las revisiones de seguridad u otras actualizaciones de las imágenes base del sistema operativo. 

* [Recomendaciones de seguridad de supervisión y examen de contenedores para Azure Container Instances](./container-instances-image-security.md)

* [Integración de Azure Container Registry con Security Center](../security-center/azure-container-registry-integration.md)
* [Acerca de las actualizaciones de imagen base para ACR Tasks](../container-registry/container-registry-tasks-base-images.md)



**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="75-securely-store-configuration-of-azure-resources"></a>7.5: Almacene de forma segura la configuración de los recursos de Azure

**Guía**: almacene y administre plantillas de ARM, archivos YAML y definiciones de directivas personalizadas de Azure de forma segura en el control de código fuente.

* [Almacenamiento de código en Azure DevOps](/azure/devops/repos/git/gitworkflow?view=azure-devops)

* [Documentación de Azure Repos](/azure/devops/repos/index?view=azure-devops)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="76-securely-store-custom-operating-system-images"></a>7.6: Almacene imágenes de sistema operativo personalizadas de forma segura

**Guía**: Guarde imágenes de contenedor en Azure Container Registry y utilice Azure RBAC para garantizar que solo los usuarios autorizados puedan acceder a ellas.

* [Descripción de Azure RBAC](../role-based-access-control/rbac-and-directory-admin-roles.md)

* [Descripción de Azure RBAC para Container Registry](../container-registry/container-registry-roles.md)

* [Configuración de Azure RBAC](../role-based-access-control/quickstart-assign-role-user-portal.md)


**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="77-deploy-system-configuration-management-tools"></a>7.7: Implemente herramientas de administración de configuración del sistema

**Instrucciones**: Use Azure Policy para alertar, auditar y aplicar las configuraciones del sistema. Además, desarrolle un proceso y una canalización para administrar las excepciones de las directivas.

* [Configuración y administración de Azure Policy](../governance/policy/tutorials/create-and-manage.md)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="78-deploy-system-configuration-management-tools-for-operating-systems"></a>7.8: Implemente herramientas de administración de configuración del sistema para sistemas operativos

**Guía**: no aplicable; esta guía está pensada para recursos de proceso de IaaS.


**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="79-implement-automated-configuration-monitoring-for-azure-services"></a>7.9: Implemente la supervisión de configuración automatizada para servicios de Azure

**Guía**: Use Azure Security Center para realizar análisis de base de referencia para los recursos de Azure.

Use Azure Policy para establecer restricciones en el tipo de recursos que se pueden crear en sus suscripciones.

* [Corrección de recomendaciones en Azure Security Center](../security-center/security-center-remediate-recommendations.md)

* [Auditoría del cumplimiento de los registros de contenedor de Azure mediante Azure Policy](../container-registry/container-registry-azure-policy.md)

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="710-implement-automated-configuration-monitoring-for-operating-systems"></a>7.10: Implemente la supervisión de configuración automatizada para sistemas operativos

**Guía**: si usa Azure Container Registry para almacenar imágenes de contenedor, utilice Azure Security Center para examinar la base de referencia y comprobar la configuración del sistema operativo y de Docker de los contenedores.

* [Descripción de las recomendaciones sobre contenedores de Azure Security Center](../security-center/container-security.md)


**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="711-manage-azure-secrets-securely"></a>7.11: Administre los secretos de Azure de forma segura

**Instrucciones**: Use Managed Service Identity junto con Azure Key Vault para simplificar y proteger la administración de secretos para las aplicaciones en la nube.

* [Integración con identidades administradas de Azure](../azure-app-configuration/howto-integrate-azure-managed-service-identity.md)

* [Creación de un almacén de claves](../key-vault/secrets/quick-create-portal.md)

* [Autenticación en Azure Key Vault](../key-vault/general/authentication.md)

* [Asignación de una directiva de acceso de Azure Key Vault](../key-vault/general/assign-access-policy-portal.md)

* [Cómo utilizar una identidad administrada con Azure Container Instances](./container-instances-managed-identity.md)

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="712-manage-identities-securely-and-automatically"></a>7.12: Administre las identidades de forma segura y automática

**Guía**: Use las identidades administradas para proporcionar a los servicios de Azure una identidad administrada automáticamente en Azure AD. Las identidades administradas le permiten autenticarse en cualquier servicio que admita la autenticación de Azure AD, como Key Vault, sin necesidad de credenciales en el código.

* [Configuración de las identidades administradas](../active-directory/managed-identities-azure-resources/qs-configure-portal-windows-vm.md)

* [Cómo utilizar una identidad administrada con Azure Container Instances](./container-instances-managed-identity.md)



**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="713-eliminate-unintended-credential-exposure"></a>7.13: Elimine la exposición de credenciales no intencionada

**Instrucciones**: Implemente el escáner de credenciales para identificar las credenciales en el código. El escáner de credenciales también fomenta el traslado de credenciales detectadas a ubicaciones más seguras, como Azure Key Vault.

* [Configuración del escáner de credenciales](https://secdevtools.azurewebsites.net/helpcredscan.html)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

## <a name="malware-defense"></a>Defensa contra malware

*Para más información, consulte [Control de seguridad: defensa contra malware](../security/benchmarks/security-control-malware-defense.md).*

### <a name="81-use-centrally-managed-anti-malware-software"></a>8.1: Use software antimalware administrado centralmente

**Instrucciones**: no aplicable; esta guía está pensada para recursos de proceso de IaaS.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="82-pre-scan-files-to-be-uploaded-to-non-compute-azure-resources"></a>8.2: Examine previamente los archivos que se van a cargar en recursos de Azure que no son de proceso

**Guía**: Microsoft Antimalware está habilitado en el host subyacente que admite los servicios de Azure (por ejemplo, Azure Container Instances), pero no se ejecuta en el contenido del cliente.

Examine previamente los archivos que se cargan en recursos de Azure que no son de proceso, como App Service, Data Lake Storage, Blob Storage, etc. 


**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="83-ensure-anti-malware-software-and-signatures-are-updated"></a>8.3: Asegúrese de que se han actualizado el software y las firmas antimalware

**Guía**: no aplicable; esta recomendación está diseñada para recursos de proceso de IaaS.

Microsoft Antimalware está habilitado en el host subyacente que admite los servicios de Azure (por ejemplo, Azure Container Instances), pero no se ejecuta en el contenido del cliente.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

## <a name="data-recovery"></a>Recuperación de datos

*Para más información, consulte [Control de seguridad: recuperación de datos](../security/benchmarks/security-control-data-recovery.md).*

### <a name="91-ensure-regular-automated-back-ups"></a>9.1: Garantía de copias de seguridad automáticas periódicas

**Guía**: habilite Azure Backup y configure el origen de la copia de seguridad (por ejemplo, un recurso compartido de archivos montado en grupos de contenedores), así como la frecuencia deseada y el período de retención. 

* [Habilitación de Azure Backup](../backup/index.yml)

* [Montaje de un recurso compartido de archivos de Azure en Azure Container Instances](./container-instances-volume-azure-files.md)


**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="92-perform-complete-system-backups-and-backup-any-customer-managed-keys"></a>9.2: Realización de copias de seguridad completas del sistema y copia de seguridad de las claves administradas por el cliente

**Guía**: Realice una copia de seguridad de las claves administradas por el cliente en Azure Key Vault con las herramientas de línea de comandos o los SDK de Azure.

Opcionalmente, haga una copia de seguridad de las imágenes de contenedor importando de un registro a otro.
* [Creación de una copia de seguridad de las claves del almacén de claves en Azure](/powershell/module/azurerm.keyvault/backup-azurekeyvaultkey?view=azurermps-6.13.0)

* [Importación de imágenes de contenedor en un registro de contenedor](../container-registry/container-registry-import-images.md)



**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="93-validate-all-backups-including-customer-managed-keys"></a>9.3: Validación de todas las copias de seguridad, incluidas las claves administradas por el cliente

**Guía**: Realice una copia de seguridad de las claves administradas por el cliente en Azure Key Vault con las herramientas de línea de comandos o los SDK de Azure.

* [Restauración de las claves de Azure Key Vault en Azure](/powershell/module/azurerm.keyvault/restore-azurekeyvaultkey?view=azurermps-6.13.0)

* [Cifrado de datos de implementación: Azure Container Instances](./container-instances-encrypt-data.md)



**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="94-ensure-protection-of-backups-and-customer-managed-keys"></a>9.4: Garantía de la protección de las copias de seguridad y las claves administradas del cliente

**Guía**: Habilite la eliminación temporal en Azure Key Vault para proteger las claves contra la eliminación accidental o malintencionada.

* [Habilitación de la eliminación temporal en Key Vault](../storage/blobs/soft-delete-overview.md?tabs=azure-portal)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

## <a name="incident-response"></a>Respuesta a los incidentes

*Para más información, consulte [Control de seguridad: Respuesta a los incidentes](../security/benchmarks/security-control-incident-response.md).*

### <a name="101-create-an-incident-response-guide"></a>10.1: Creación de una guía de respuesta ante incidentes

**Instrucciones**: Cree una guía de respuesta a incidentes para su organización. Asegúrese de que haya planes de respuesta a incidentes escritos que definan todos los roles del personal, así como las fases de administración y gestión de los incidentes, desde la detección hasta la revisión posterior a la incidencia.

El cliente también puede usar la guía de control de incidentes de seguridad de equipos de NIST como referencia para la creación de su propio plan de respuesta a incidentes.

* [Configuración de las automatizaciones de flujos de trabajo en Azure Security Center](../security-center/security-center-planning-and-operations-guide.md)

* [Guía para crear su propio proceso de respuesta a incidentes de seguridad](https://msrc-blog.microsoft.com/2019/07/01/inside-the-msrc-building-your-own-security-incident-response-process/)

* [Anatomía de un incidente del Centro de respuestas de seguridad de Microsoft](https://msrc-blog.microsoft.com/2019/07/01/inside-the-msrc-building-your-own-security-incident-response-process/)

* [Guía del NIST para controlar incidentes de seguridad en equipos](https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-61r2.pdf)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="102-create-an-incident-scoring-and-prioritization-procedure"></a>10.2: Creación de un procedimiento de priorización y puntuación de incidentes

**Instrucciones**: Azure Security Center asigna una gravedad a cada alerta para ayudarle a priorizar aquellas que se deben investigar en primer lugar. La gravedad se basa en la confianza que tiene Security Center en la búsqueda o en el análisis utilizados para emitir la alerta, así como en el nivel de confianza de que ha habido un intento malintencionado detrás de la actividad que ha provocado la alerta.

Además, marque claramente las suscripciones (por ejemplo, producción, no producción) y cree un sistema de nomenclatura para identificar y clasificar claramente los recursos de Azure.


**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="103-test-security-response-procedures"></a>10.3: Prueba de los procedimientos de respuesta de seguridad

**Guía**: Realice ejercicios para probar las capacidades de respuesta a los incidentes de los sistemas con regularidad. Identifique puntos débiles y brechas y revise el plan según sea necesario.

* [Consulte la publicación de NIST: Guía para probar, entrenar y ejecutar programas para planes y funcionalidades de TI](https://nvlpubs.nist.gov/nistpubs/Legacy/SP/nistspecialpublication800-84.pdf)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="104-provide-security-incident-contact-details-and-configure-alert-notifications-for-security-incidents"></a>10.4: Provisión de detalles de contacto de incidentes de seguridad y configuración de notificaciones de alerta para incidentes de seguridad

**Guía**: La información de contacto del incidente de seguridad la utilizará Microsoft para ponerse en contacto con usted si Microsoft Security Response Center (MSRC) detecta que un tercero no autorizado o ilegal ha accedido a los datos del cliente. Revise los incidentes después del hecho para asegurarse de que se resuelven los problemas.

* [Establecimiento del contacto de seguridad de Azure Security Center](../security-center/security-center-provide-security-contact-details.md)

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="105-incorporate-security-alerts-into-your-incident-response-system"></a>10.5: Incorporación de alertas de seguridad en el sistema de respuesta a incidentes

**Instrucciones**: Exporte sus alertas y recomendaciones de Azure Security Center mediante la característica de exportación continua. La exportación continua le permite exportar alertas y recomendaciones de forma manual o continua. Puede usar el conector de datos de Azure Security Center para transmitir las alertas a Sentinel.

* [Configuración de la exportación continua](../security-center/continuous-export.md)

* [Transmisión de alertas a Azure Sentinel](../sentinel/connect-azure-security-center.md)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="106-automate-the-response-to-security-alerts"></a>10.6: Automatización de la respuesta a las alertas de seguridad

**Instrucciones**: use la característica de automatización del flujo de trabajo de Azure Security Center para desencadenar automáticamente respuestas mediante "Logic Apps" en las alertas y recomendaciones de seguridad.

* [Configuración de la automatización de flujo de trabajo y Logic Apps](../security-center/workflow-automation.md)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

## <a name="penetration-tests-and-red-team-exercises"></a>Pruebas de penetración y ejercicios del equipo rojo

*Para más información, consulte [Control de seguridad: Pruebas de penetración y ejercicios del equipo rojo](../security/benchmarks/security-control-penetration-tests-red-team-exercises.md).*

### <a name="111-conduct-regular-penetration-testing-of-your-azure-resources-and-ensure-remediation-of-all-critical-security-findings-within-60-days"></a>11.1: Realice pruebas de penetración periódicas de los recursos de Azure y asegúrese de corregir todos los resultados de seguridad críticos en un plazo de 60 días

**Guía**: Siga las reglas de compromiso de Microsoft para asegurarse de que las pruebas de penetración no infrinjan las directivas de Microsoft. Use la estrategia de Microsoft y la ejecución de pruebas de penetración de sitios activos y ataques simulados en la infraestructura en la nube, los servicios y las aplicaciones administrados por Microsoft.

* [Reglas de interacción de las pruebas de penetración](https://www.microsoft.com/msrc/pentest-rules-of-engagement?rtc=1)

* [Equipo rojo de Microsoft Cloud](https://gallery.technet.microsoft.com/Cloud-Red-Teaming-b837392e)


**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Compartido

## <a name="next-steps"></a>Pasos siguientes

- Consulte la [prueba comparativa de seguridad de Azure](../security/benchmarks/overview.md).
- Obtenga más información sobre las [líneas de base de seguridad de Azure](../security/benchmarks/security-baselines-overview.md).
