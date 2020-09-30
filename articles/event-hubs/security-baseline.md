---
title: Línea de base de seguridad de Azure para Event Hubs
description: Línea de base de seguridad de Azure para Event Hubs
author: msmbaldwin
ms.service: security
ms.topic: conceptual
ms.date: 03/25/2020
ms.author: mbaldwin
ms.custom: subject-security-benchmark
ms.openlocfilehash: f47e9cb93b90c8a2401d1dfe6c0ff7800b0bcf27
ms.sourcegitcommit: 5a3b9f35d47355d026ee39d398c614ca4dae51c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89401136"
---
# <a name="azure-security-baseline-for-event-hubs"></a>Línea de base de seguridad de Azure para Event Hubs

La línea de base de seguridad de Azure para Event Hubs contiene recomendaciones que lo ayudarán a mejorar la posición de seguridad de la implementación.

La base de referencia de este servicio se extrae de la [versión 1.0 de Azure Security Benchmark](../security/benchmarks/overview.md), que proporciona recomendaciones sobre cómo puede proteger las soluciones en la nube en Azure con nuestra guía de procedimientos recomendados.

Para obtener más información, consulte [Introducción a las líneas de base de seguridad de Azure](../security/benchmarks/security-baselines-overview.md).

## <a name="network-security"></a>Seguridad de redes

*Para obtener más información, consulte [Control de seguridad: seguridad de red](../security/benchmarks/security-control-network-security.md).*

### <a name="11-protect-resources-using-network-security-groups-or-azure-firewall-on-your-virtual-network"></a>1.1: Proteja los recursos mediante grupos de seguridad de red o Azure Firewall en su red virtual

**Guía**: La integración de Event Hubs en puntos de conexión de servicio de red virtual permite el acceso seguro a las funcionalidades de mensajería desde cargas de trabajo, tales como máquinas virtuales que están enlazadas a redes virtuales, con una ruta de acceso del tráfico de red que está protegida en ambos extremos.

Una vez que se enlaza con al menos un punto de conexión de servicio de subred de red virtual, el espacio de nombres respectivo de Event Hubs ya solo aceptará el tráfico procedente de redes virtuales autorizadas. Desde la perspectiva de la red virtual, el enlace del espacio de nombres de Event Hubs a un punto de conexión de servicio configura un túnel de redes aislado desde la subred de la red virtual al servicio de mensajería. 

También puede crear un punto de conexión privado, que es una interfaz de red que lo conecta de forma privada y segura al servicio de Azure Event Hubs mediante el servicio de Azure Private Link. El punto de conexión privado usa una dirección IP privada de la red virtual para incorporar el servicio de manera eficaz a su red virtual. Todo el tráfico dirigido al servicio se puede enrutar mediante el punto de conexión privado, por lo que no se necesita ninguna puerta de enlace, dispositivos NAT, conexiones de ExpressRoute o VPN ni direcciones IP públicas. 

También puede proteger el espacio de nombres de Azure Event Hubs mediante firewalls. Azure Event Hubs admite controles de acceso basado en IP para la compatibilidad con el firewall de entrada. Puede establecer reglas de firewall mediante Azure Portal, plantillas de Azure Resource Manager, o a través de la CLI de Azure o Azure PowerShell.

Procedimiento para usar puntos de conexión de servicio de red virtual con Azure Event Hubs: https://docs.microsoft.com/azure/event-hubs/event-hubs-service-endpoints

Para más información, consulte Integración de Azure Event Hubs en Azure Private Link: https://docs.microsoft.com/azure/event-hubs/private-link-service.

Habilitación de la integración y los firewalls de redes virtuales en el espacio de nombres de Event Hubs: https://docs.microsoft.com/azure/event-hubs/event-hubs-tutorial-virtual-networks-firewalls

Procedimiento para configurar reglas de firewall de IP para espacios de nombres de Azure Event Hubs: https://docs.microsoft.com/azure/event-hubs/event-hubs-ip-filtering

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="12-monitor-and-log-the-configuration-and-traffic-of-vnets-subnets-and-nics"></a>1.2: Supervise y registre la configuración y el tráfico de redes virtuales, subredes y NIC

**Instrucciones**: Use Azure Security Center y siga las recomendaciones de protección de redes para ayudar a proteger sus recursos de Event Hubs en Azure. Si usa máquinas virtuales de Azure para obtener acceso a los centros de eventos, habilite los registros de flujo de grupos de seguridad de red (NSG) y envíe los registros a una cuenta de almacenamiento para la auditoría de tráfico.

Cómo habilitar los registros de flujo de NSG: https://docs.microsoft.com/azure/network-watcher/network-watcher-nsg-flow-logging-portal

Descripción de la seguridad de red proporcionada por Azure Security Center: https://docs.microsoft.com/azure/security-center/security-center-network-recommendations

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="13-protect-critical-web-applications"></a>1.3: Proteja las aplicaciones web críticas

**Instrucciones**: No aplicable; esta recomendación está pensada para las aplicaciones web que se ejecutan en Azure App Service o en recursos de proceso.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="14-deny-communications-with-known-malicious-ip-addresses"></a>1.4: Deniegue las comunicaciones con direcciones IP malintencionadas conocidas

**Instrucciones**: Habilite el estándar de DDoS Protection en las redes virtuales asociadas a los centros de eventos para protegerse de los ataques por denegación de servicio distribuidos (DDoS). Use la inteligencia sobre amenazas integrada de Azure Security Center para denegar las comunicaciones con direcciones IP malintencionadas conocidas o no utilizadas.

Cómo configurar la protección contra DDoS: [https://docs.microsoft.com/azure/virtual-network/manage-ddos-protection](../virtual-network/manage-ddos-protection.md)

Para obtener más información sobre la inteligencia sobre amenazas integrada de Azure Security Center: https://docs.microsoft.com/azure/security-center/security-center-alerts-service-layer

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="15-record-network-packets-and-flow-logs"></a>1.5: Registre los paquetes de red y registros de flujo

**Guía**: Si usa máquinas virtuales de Azure para obtener acceso a los centros de eventos, habilite los registros de flujo de grupos de seguridad de red (NSG) y envíe los registros a una cuenta de almacenamiento para la auditoría de tráfico. También puede enviar registros de flujo de grupo de seguridad de red a un área de trabajo de Log Analytics y usar el Análisis de tráfico para proporcionar información detallada sobre el flujo de tráfico en la nube de Azure. Algunas de las ventajas del Análisis de tráfico son la capacidad de visualizar la actividad de la red e identificar las zonas activas, identificar las amenazas de seguridad, comprender los patrones de flujo de tráfico y detectar configuraciones de red incorrectas.

Si es necesario para investigar actividades anómalas, habilite la captura de paquetes de Network Watcher.

Cómo habilitar los registros de flujo de NSG: https://docs.microsoft.com/azure/network-watcher/network-watcher-nsg-flow-logging-portal

Habilitación y uso del Análisis de tráfico: https://docs.microsoft.com/azure/network-watcher/traffic-analytics

Cómo habilitar Network Watcher: https://docs.microsoft.com/azure/network-watcher/network-watcher-create

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="16-deploy-network-based-intrusion-detectionintrusion-prevention-systems-idsips"></a>1.6: Implemente sistemas de prevención de intrusiones y detección de intrusiones (IDS/IPS) basados en la red

**Instrucciones**: Si usa máquinas virtuales de Azure para obtener acceso a los centros de eventos, seleccione una oferta de Azure Marketplace que admita la funcionalidad de IDS/IPS con funcionalidades de inspección de carga. Si la detección o prevención de intrusiones basadas en la inspección de la carga no es necesaria en su organización, puede usar la característica de firewall integrada de Azure Event Hubs. Puede restringir el acceso al espacio de nombres de Event Hubs para un intervalo limitado de direcciones IP o una dirección IP específica mediante reglas de firewall.

Azure Marketplace:

https://azuremarketplace.microsoft.com/marketplace/?term=Firewall

Procedimiento para agregar una regla de firewall en Event Hubs para una dirección IP especificada:

 https://docs.microsoft.com/azure/event-hubs/event-hubs-ip-filtering

**Supervisión de Azure Security Center**: No disponible todavía

**Responsabilidad**: Customer

### <a name="17-manage-traffic-to-web-applications"></a>1.7: Administre el tráfico a las aplicaciones web

**Instrucciones**: No aplicable; esta recomendación está pensada para las aplicaciones web que se ejecutan en Azure App Service o en recursos de proceso.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="18-minimize-complexity-and-administrative-overhead-of-network-security-rules"></a>1.8: Minimice la complejidad y la sobrecarga administrativa de las reglas de seguridad de red

**Guía**: No aplicable, esta recomendación está pensada para las aplicaciones web que se ejecutan en Azure App Service o en recursos de proceso.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="19-maintain-standard-security-configurations-for-network-devices"></a>1.9: Mantenga las configuraciones de seguridad estándar para dispositivos de red

**Guía**: Defina e implemente configuraciones de seguridad estándar para los recursos de red asociados a los espacios de nombres de Azure Event Hubs con Azure Policy. Use alias de Azure Policy en los espacios de nombres "Microsoft.EventHub" y "Microsoft.Network" para crear directivas personalizadas con el fin de auditar o aplicar la configuración de red de los espacios de nombres de Event Hubs. También puede usar definiciones de directivas integradas relacionadas con Azure Event Hubs, como:

- El centro de eventos debe usar un punto de conexión del servicio de red virtual.

Cómo configurar y administrar Azure Policy: https://docs.microsoft.com/azure/governance/policy/tutorials/create-and-manage

Directiva integrada de Azure para espacio de nombres de Event Hubs: https://docs.microsoft.com/azure/governance/policy/samples/built-in-policies#event-hub



Ejemplos de Azure Policy para redes: https://docs.microsoft.com/azure/governance/policy/samples/built-in-policies#network



Creación de un plano técnico de Azure: https://docs.microsoft.com/azure/governance/blueprints/create-blueprint-portal

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="110-document-traffic-configuration-rules"></a>1.10: Documente las reglas de configuración de tráfico

**Guía**: Use etiquetas para redes virtuales y otros recursos relacionados con la seguridad de red y el flujo de tráfico que están asociados a los centros de eventos.

Creación y uso de etiquetas: https://docs.microsoft.com/azure/azure-resource-manager/resource-group-using-tags

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="111-use-automated-tools-to-monitor-network-resource-configurations-and-detect-changes"></a>1.11: Use herramientas automatizadas para supervisar las configuraciones de recursos de red y detectar cambios

**Guía**: Use el registro de actividad de Azure para supervisar las configuraciones de los recursos de red y detectar cambios en los recursos de red relacionados con Azure Event Hubs. Cree alertas en Azure Monitor que se desencadenarán cuando se produzcan cambios en los recursos de red críticos.

Visualización y recuperación de eventos del registro de actividad de Azure: https://docs.microsoft.com/azure/azure-monitor/platform/activity-log-view

Creación de alertas en Azure Monitor: https://docs.microsoft.com/azure/azure-monitor/platform/alerts-activity-log

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

## <a name="logging-and-monitoring"></a>Registro y supervisión

*Para obtener más información, consulte [Control de seguridad: registro y supervisión](../security/benchmarks/security-control-logging-monitoring.md).*

### <a name="21-use-approved-time-synchronization-sources"></a>2.1: Uso de orígenes de sincronización de hora aprobados

**Guía**: No aplicable; Microsoft mantiene el origen de la hora usado en los recursos de Azure, como Azure Event Hubs, para las marcas de tiempo de los registros.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Microsoft

### <a name="22-configure-central-security-log-management"></a>2.2: Configuración de la administración central de registros de seguridad

**Instrucciones**: Dentro de Azure Monitor, configure los registros relacionados con Event Hubs en el registro de actividad y la configuración de diagnóstico de Event Hubs para enviar los registros a un área de trabajo de Log Analytics que se va a consultar o a una cuenta de almacenamiento para el almacenamiento de archivo a largo plazo.

Procedimiento para establecer la configuración de diagnóstico para Azure Event Hubs: https://docs.microsoft.com/azure/event-hubs/event-hubs-diagnostic-logs

Descripción del registro de actividad de Azure: https://docs.microsoft.com/azure/azure-monitor/platform/platform-logs-overview

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="23-enable-audit-logging-for-azure-resources"></a>2.3: Habilitación del registro de auditoría para recursos de Azure

**Guía**: Habilite la configuración de diagnóstico para el espacio de nombres de Azure Event Hubs. Existen tres categorías de configuración de diagnóstico para Azure Event Hubs: Registros de archivo, registros operativos y registros de escalabilidad automática. Habilite los registros operativos para capturar información sobre lo que sucede durante las operaciones de Event Hubs, en concreto, el tipo de operación, como la creación del centro de eventos, los recursos usados y el estado de la operación.

Además, puede habilitar la configuración de diagnóstico del registro de actividad de Azure y enviarla a una cuenta de Azure Storage, un centro de eventos o un área de trabajo de Log Analytics. Los registros de actividades proporcionan información sobre las operaciones llevadas a cabo en Azure Event Hubs y otros recursos. Con los registros de actividades, se puede responder a las preguntas "qué, quién y cuándo" de las operaciones de escritura (PUT, POST, DELETE) llevadas a cabo en los espacios de nombres de Azure Event Hubs.

Procedimiento para habilitar la configuración de diagnóstico para Azure Event Hubs: https://docs.microsoft.com/azure/event-hubs/event-hubs-diagnostic-logs

Procedimiento para habilitar la configuración de diagnóstico para el registro de actividad de Azure: https://docs.microsoft.com/azure/azure-monitor/platform/diagnostic-settings-legacy

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="24-collect-security-logs-from-operating-systems"></a>2.4: Recopilación de registros de seguridad de sistemas operativos

**Instrucciones**: No aplicable; esta recomendación está pensada para los recursos de proceso.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="25-configure-security-log-storage-retention"></a>2.5: Configuración de la retención del almacenamiento de registros de seguridad

**Guía**: En Azure Monitor, establezca el período de retención del área de trabajo de Log Analytics de acuerdo con la normativa de cumplimiento de su organización, con el fin de capturar y revisar los incidentes relacionados con los centros de eventos.

Configuración de parámetros de retención de registros de áreas de trabajo de Log Analytics: https://docs.microsoft.com/azure/azure-monitor/platform/manage-cost-storage#change-the-data-retention-period

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="26-monitor-and-review-logs"></a>2.6: Supervisión y registros de revisión

**Guía**: Analice y supervise los registros en busca de comportamientos anómalos y revise periódicamente los resultados relacionados con los centros de eventos. Use Log Analytics de Azure Monitor para revisar los registros y realizar consultas en los datos del registro. Como alternativa, puede habilitar e incorporar datos en Azure Sentinel o en una herramienta SIEM de terceros.
 

Para obtener más información acerca del área de trabajo de Log Analytics, consulte: https://docs.microsoft.com/azure/azure-monitor/log-query/get-started-portal

Procedimiento para realizar consultas personalizadas en Azure Monitor: https://docs.microsoft.com/azure/azure-monitor/log-query/get-started-queries

Procedimiento de incorporación de Azure Sentinel: https://docs.microsoft.com/azure/sentinel/quickstart-onboard

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="27-enable-alerts-for-anomalous-activity"></a>2.7: Habilitación de alertas para actividades anómalas

**Guía**: Dentro de Azure Monitor, configure los registros relacionados con Azure Event Hubs en el registro de actividad y la configuración de diagnóstico de Event Hubs para enviar los registros a un área de trabajo de Log Analytics que se va a consultar o a una cuenta de almacenamiento para el almacenamiento de archivo a largo plazo. Use el área de trabajo de Log Analytics para crear alertas para actividades anómalas que se encuentren en registros y eventos de seguridad.

Como alternativa, puede habilitar e incorporar datos en Azure Sentinel. 

Introducción al registro de actividad de Azure: https://docs.microsoft.com/azure/azure-monitor/platform/platform-logs-overview

Procedimiento para establecer la configuración de diagnóstico para Azure Event Hubs: https://docs.microsoft.com/azure/event-hubs/event-hubs-diagnostic-logs

Alertas sobre datos de registro del área de trabajo de Log Analytics: https://docs.microsoft.com/azure/azure-monitor/learn/tutorial-response

Incorporación de Azure Sentinel: https://docs.microsoft.com/azure/sentinel/quickstart-onboard

**Supervisión de Azure Security Center**: No disponible todavía

**Responsabilidad**: Customer

### <a name="28-centralize-anti-malware-logging"></a>2.8: Centralización del registro antimalware

**Guía**: No aplicable; Event Hubs no procesa el registro antimalware.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="29-enable-dns-query-logging"></a>2.9: Habilitación del registro de consultas DNS

**Guía**: No aplicable; Event Hubs no procesa ni genera registros relacionados con DNS.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="210-enable-command-line-audit-logging"></a>2.10: Habilitación del registro de auditoría de la línea de comandos

**Guía**: No aplicable; esta guía está pensada para recursos de proceso.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

## <a name="identity-and-access-control"></a>Identidad y Access Control

*Para obtener más información, consulte [Control de seguridad: identidad y control de acceso](../security/benchmarks/security-control-identity-access-control.md).*

### <a name="31-maintain-an-inventory-of-administrative-accounts"></a>3.1: Mantenga un inventario de cuentas administrativas

**Instrucciones**: Azure Active Directory (AD) tiene roles integrados que se deben asignar explícitamente y son consultables. Use el módulo de PowerShell de Azure AD para realizar consultas ad hoc para detectar cuentas que son miembros de grupos administrativos. 

Obtención de un rol de directorio en Azure AD con PowerShell: https://docs.microsoft.com/powershell/module/azuread/get-azureaddirectoryrole?view=azureadps-2.0

Obtención de los miembros de un rol de directorio en Azure AD con PowerShell: https://docs.microsoft.com/powershell/module/azuread/get-azureaddirectoryrolemember?view=azureadps-2.0

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="32-change-default-passwords-where-applicable"></a>3.2: Cambie las contraseñas predeterminadas cuando proceda

**Guía**: El acceso del plano de control a Event Hubs se controla a través de Azure Active Directory (AD). Azure AD no tiene el concepto de contraseñas predeterminadas.

El acceso del plano de datos a Event Hubs se controla a través de Azure AD con Identidades administradas o Registros de aplicaciones, así como con firmas de acceso compartido. Los clientes que se conectan a los centros de eventos usan las firmas de acceso compartido, que se pueden volver a generar en cualquier momento.

Introducción a las firmas de acceso compartido para Event Hubs: https://docs.microsoft.com/azure/event-hubs/authenticate-shared-access-signature

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="33-use-dedicated-administrative-accounts"></a>3.3: Use cuentas administrativas dedicadas

**Guía**: Cree procedimientos operativos estándar en torno al uso de cuentas administrativas dedicadas. Use la administración de identidad y acceso de Azure Security Center para supervisar el número de cuentas administrativas.

Además, para ayudarle a realizar un seguimiento de las cuentas administrativas dedicadas, puede seguir las recomendaciones de Azure Security Center o las directivas integradas de Azure Policy, por ejemplo:

- Debe haber más de un propietario asignado a su suscripción

- Las cuentas en desuso con permisos de propietario deben quitarse de la suscripción

- Las cuentas externas con permisos de propietario deben quitarse de la suscripción

Uso de Azure Security Center para supervisar la identidad y el acceso (versión preliminar): https://docs.microsoft.com/azure/security-center/security-center-identity-access

Uso de Azure Policy: https://docs.microsoft.com/azure/governance/policy/tutorials/create-and-manage

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="34-use-single-sign-on-sso-with-azure-active-directory"></a>3.4: Use el inicio de sesión único (SSO) con Azure Active Directory

**Guía**: Microsoft Azure proporciona una administración integrada del control de acceso para recursos y aplicaciones basados en Azure Active Directory (AD). Una ventaja clave de usar Azure AD con Azure Event Hubs es que ya no es necesario almacenar las credenciales en el código. En su lugar, puede solicitar un token de acceso de OAuth 2.0 desde la Plataforma de identidad de Microsoft. El nombre del recurso para solicitar un token es https:\//eventhubs.azure.net/. Azure AD autentica la entidad de seguridad (un usuario, grupo o entidad de servicio) ejecutando la aplicación. Si la autenticación se realiza correctamente, Azure AD devuelve un token de acceso a la aplicación y la aplicación puede entonces usar el token de acceso para autorizar una solicitud de los recursos de Azure Event Hubs.

Procedimiento para autenticar una aplicación con Azure AD para acceder a recursos de Event Hubs: https://docs.microsoft.com/azure/event-hubs/authenticate-application

Descripción del SSO con Azure AD: https://docs.microsoft.com/azure/active-directory/manage-apps/what-is-single-sign-on

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="35-use-multi-factor-authentication-for-all-azure-active-directory-based-access"></a>3.5: Use la autenticación multifactor para todo el acceso basado en Azure Active Directory

**Guía**: Habilite Multi-Factor Authentication (MFA) de Azure Active Directory y siga las recomendaciones de administración de identidades y acceso de Azure Security Center para ayudar a proteger los recursos habilitados para Event Hubs.

Cómo habilitar la MFA en Azure: https://docs.microsoft.com/azure/active-directory/authentication/howto-mfa-getstarted

Cómo supervisar la identidad y el acceso en Azure Security Center: https://docs.microsoft.com/azure/security-center/security-center-identity-access

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="36-use-dedicated-machines-privileged-access-workstations-for-all-administrative-tasks"></a>3.6: Use máquinas dedicadas (estaciones de trabajo de acceso con privilegios) para todas las tareas administrativas

**Guía**: Utilice estaciones de trabajo con acceso con privilegios (PAW) con Multi-Factor Authentication (MFA) configurada para iniciar sesión en recursos habilitados para Event Hubs y configurarlos.

Más información sobre las estaciones de trabajo con privilegios de acceso: https://docs.microsoft.com/windows-server/identity/securing-privileged-access/privileged-access-workstations

Cómo habilitar la MFA en Azure: https://docs.microsoft.com/azure/active-directory/authentication/howto-mfa-getstarted

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="37-log-and-alert-on-suspicious-activity-from-administrative-accounts"></a>3.7: Registre y alerte de la actividad sospechosa desde cuentas administrativas

**Guía**: Use Privileged Identity Management (PIM) de Azure Active Directory (AAD) para la generación de registros y alertas cuando se produzca actividad sospechosa o insegura en el entorno. Use las detecciones de riesgo de Azure AD para ver alertas e informes sobre el comportamiento de los usuarios de riesgo. Para obtener más registros, envíe alertas de detección de Azure Security Center riesgo en Azure Monitor y configure alertas o notificaciones personalizadas con grupos de acciones.

Procedimiento para implementar Privileged Identity Management (PIM): https://docs.microsoft.com/azure/active-directory/privileged-identity-management/pim-deployment-plan

Introducción a las detecciones de riesgo de Azure AD: https://docs.microsoft.com/azure/active-directory/reports-monitoring/concept-risk-events

Procedimiento para configurar grupos de acciones para alertas y notificaciones personalizadas: https://docs.microsoft.com/azure/azure-monitor/platform/action-groups

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="38-manage-azure-resources-from-only-approved-locations"></a>3.8: Administre los recursos de Azure solo desde ubicaciones aprobadas

**Guía**: Use ubicaciones con nombre de acceso condicional para permitir el acceso solo desde agrupaciones lógicas específicas de intervalos de direcciones IP o países o regiones.



Configuración de ubicaciones con nombre en Azure: https://docs.microsoft.com/azure/active-directory/reports-monitoring/quickstart-configure-named-locations

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="39-use-azure-active-directory"></a>3.9: Uso de Azure Active Directory

**Guía**: Use Azure Active Directory (AD) como sistema central de autenticación y autorización para recursos de Azure, como Event Hubs. Esto permite el control de acceso basado en roles (RBAC) para administrar recursos confidenciales.

 Procedimiento para crear y configurar una instancia de Azure AD: https://docs.microsoft.com/azure/active-directory/fundamentals/active-directory-access-create-new-tenant

Para obtener información sobre la manera en que Azure Event Hubs se integra en Azure Active Directory (AAD), consulte Autorización del acceso a recursos de Event Hubs mediante Azure Active Directory: https://docs.microsoft.com/azure/event-hubs/authorize-access-azure-active-directory

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="310-regularly-review-and-reconcile-user-access"></a>3.10: Revise y concilie regularmente el acceso de los usuarios

**Guía**: Azure Active Directory (AD) proporciona registros para ayudar a descubrir cuentas obsoletas. Además, use las revisiones de acceso de identidad de Azure para administrar de forma eficiente las pertenencias a grupos, el acceso a las aplicaciones empresariales y las asignaciones de roles. El acceso de los usuarios se puede revisar de forma periódica para asegurarse de que solo las personas adecuadas tengan acceso continuado.

Además, rote periódicamente las firmas de acceso compartido de Event Hubs.

Introducción a los informes de Azure AD: https://docs.microsoft.com/azure/active-directory/reports-monitoring/

Cómo usar las revisiones de acceso de identidad de Azure: https://docs.microsoft.com/azure/active-directory/governance/access-reviews-overview

Introducción a las firmas de acceso compartido para Event Hubs: https://docs.microsoft.com/azure/event-hubs/authenticate-shared-access-signature

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="311-monitor-attempts-to-access-deactivated-accounts"></a>3.11: Supervise los intentos de acceso a cuentas desactivadas

**Guía**: Tiene acceso a los orígenes de registro de eventos de actividad de inicio de sesión, auditoría y riesgo de Azure Active Directory (AD), que permiten la integración en cualquier herramienta de SIEM o supervisión.

Puede simplificar este proceso creando una configuración de diagnóstico para las cuentas de usuario de Azure AD y enviando los registros de auditoría y los registros de inicio de sesión a un área de trabajo de Log Analytics. Puede configurar las alertas de registro deseadas en Log Analytics.

Procedimiento para integrar los registros de actividad de Azure en Azure Monitor: https://docs.microsoft.com/azure/active-directory/reports-monitoring/howto-integrate-activity-logs-with-log-analytics

Autorización del acceso a recursos de Event Hubs mediante Azure Active Directory: https://docs.microsoft.com/azure/event-hubs/authorize-access-azure-active-directory

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="312-alert-on-account-login-behavior-deviation"></a>3.12: Alerte de las desviaciones de comportamiento en los inicios de sesión de las cuentas

**Guía**: Use las características Identity Protection y de detección de riesgos de Azure Active Directory para configurar respuestas automatizadas a acciones sospechosas detectadas relacionadas con los recursos habilitados para Event Hubs. Debe habilitar las respuestas automatizadas a través de Azure Sentinel para implementar las respuestas de seguridad de su organización.

Procedimiento para ver los inicios de sesión de riesgo de Azure AD: https://docs.microsoft.com/azure/active-directory/reports-monitoring/concept-risky-sign-ins

Configuración y habilitación de las directivas de riesgo de protección de identidad: https://docs.microsoft.com/azure/active-directory/identity-protection/howto-identity-protection-configure-risk-policies

Incorporación de Azure Sentinel: https://docs.microsoft.com/azure/sentinel/quickstart-onboard

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="313-provide-microsoft-with-access-to-relevant-customer-data-during-support-scenarios"></a>3.13: Proporcione a Microsoft acceso a los datos pertinentes del cliente durante los escenarios de soporte técnico

**Instrucciones**: Actualmente no disponible; la Caja de seguridad del cliente todavía no se admite en Event Hubs.

Lista de servicios admitidos por la Caja de seguridad del cliente: https://docs.microsoft.com/azure/security/fundamentals/customer-lockbox-overview#supported-services-and-scenarios-in-general-availability

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: no disponible actualmente

## <a name="data-protection"></a>Protección de datos

*Para obtener más información, consulte [Control de seguridad: protección de datos](../security/benchmarks/security-control-data-protection.md).*

### <a name="41-maintain-an-inventory-of-sensitive-information"></a>4.1: Mantenimiento de un inventario de información confidencial

**Instrucciones**: Use etiquetas en los recursos relacionados con las instancias de Event Hubs para ayudar a realizar un seguimiento de los recursos de Azure que almacenan o procesan información confidencial.

Creación y uso de etiquetas: https://docs.microsoft.com/azure/azure-resource-manager/resource-group-using-tags

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="42-isolate-systems-storing-or-processing-sensitive-information"></a>4.2: Aislamiento de los sistemas que almacenan o procesan información confidencial

**Instrucciones**: Implemente suscripciones y/o grupos de administración independientes para los entornos de desarrollo, prueba y producción. Los espacios de nombres de Event Hubs deben estar separados por una red virtual con puntos de conexión de servicio habilitados y etiquetados adecuadamente.

También puede proteger el espacio de nombres de Azure Event Hubs mediante firewalls. Azure Event Hubs admite controles de acceso basado en IP para la compatibilidad con el firewall de entrada. Puede establecer reglas de firewall mediante Azure Portal, plantillas de Azure Resource Manager, o a través de la CLI de Azure o Azure PowerShell.

Creación de suscripciones adicionales de Azure: https://docs.microsoft.com/azure/billing/billing-create-subscription

Creación de grupos de administración: https://docs.microsoft.com/azure/governance/management-groups/create

Configuración de reglas de firewall de IP para espacios de nombres de Azure Event Hubs: https://docs.microsoft.com/azure/event-hubs/event-hubs-ip-filtering

Procedimiento para crear y usar etiquetas: https://docs.microsoft.com/azure/azure-resource-manager/resource-group-using-tags

Procedimiento para crear una red virtual: https://docs.microsoft.com/azure/virtual-network/quick-create-portal

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="43-monitor-and-block-unauthorized-transfer-of-sensitive-information"></a>4.3: Supervisión y bloqueo de una transferencia no autorizada de información confidencial

**Guía**: Al usar máquinas virtuales para tener acceso a los centros de eventos, aproveche las redes virtuales, los puntos de conexión de servicio, el firewall de Event Hubs, los grupos de seguridad de red y las etiquetas de servicio para mitigar la posibilidad de que se produzca una filtración de datos.

Microsoft administra la infraestructura subyacente para Azure Event Hubs y ha implementado controles estrictos para evitar la pérdida o exposición de los datos de los clientes.

Configuración de reglas de firewall de IP para espacios de nombres de Azure Event Hubs: https://docs.microsoft.com/azure/event-hubs/event-hubs-ip-filtering

Introducción a los puntos de conexión de servicio de red virtual con Azure Event Hubs: https://docs.microsoft.com/azure/event-hubs/event-hubs-service-endpoints

Integración de Azure Event Hubs en Azure Private Link: https://docs.microsoft.com/azure/event-hubs/private-link-service

Introducción a los grupos de seguridad de red y las etiquetas de servicio: https://docs.microsoft.com/azure/virtual-network/security-overview

Descripción de la protección de datos de los clientes en Azure: https://docs.microsoft.com/azure/security/fundamentals/protection-customer-data

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="44-encrypt-all-sensitive-information-in-transit"></a>4.4: Cifrado de toda la información confidencial en tránsito

**Guía**: Azure Event Hubs aplica las comunicaciones cifradas con TLS de forma predeterminada. Actualmente se admiten las versiones de TLS 1.0, 1.1 y 1.2. Sin embargo, TLS 1.0 y 1.1 están en proceso de desuso en todo el sector, por lo que se recomienda TLS 1.2 si es posible.

Para comprender las características de seguridad de Event Hubs, consulte Seguridad de red: https://docs.microsoft.com/azure/event-hubs/network-security

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Microsoft

### <a name="45-use-an-active-discovery-tool-to-identify-sensitive-data"></a>4.5: Uso de una herramienta de detección activa para identificar datos confidenciales

**Instrucciones**: Las características de identificación, clasificación y prevención de pérdida de datos todavía no están disponibles para Azure Event Hubs. Implemente una solución de terceros, si es necesario, para fines de cumplimiento.

En el caso de la plataforma subyacente administrada por Microsoft, Microsoft trata todo el contenido de los clientes como confidencial y hace grandes esfuerzos para proteger a los clientes contra la pérdida y exposición de sus datos. Para garantizar la seguridad de los datos de los clientes dentro de Azure, Microsoft ha implementado y mantiene un conjunto de controles y funcionalidades eficaces de protección de datos.

Descripción de la protección de datos de los clientes en Azure: https://docs.microsoft.com/azure/security/fundamentals/protection-customer-data

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Compartido

### <a name="46-use-azure-rbac-to-control-access-to-resources"></a>4.6: Uso de RBAC de Azure para controlar el acceso a los recursos

**Guía**: Azure Event Hubs admite la autorización de solicitudes a recursos de Event Hubs mediante Azure Active Directory (AD). Con Azure AD, puede usar el control de acceso basado en rol (RBAC) para conceder permisos a una entidad de seguridad, que puede ser un usuario o una entidad de servicio de aplicación.

Introducción a RBAC de Azure AD y los roles disponibles para Azure Event Hubs: https://docs.microsoft.com/azure/event-hubs/authorize-access-azure-active-directory

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="47-use-host-based-data-loss-prevention-to-enforce-access-control"></a>4.7: Uso de la prevención de pérdida de datos basada en host para aplicar el control de acceso

**Guía**: No aplicable; esta guía está pensada para recursos de proceso.

Microsoft administra la infraestructura subyacente para Event Hubs y ha implementado controles estrictos para evitar la pérdida o exposición de los datos de los clientes.

Descripción de la protección de datos de los clientes en Azure: https://docs.microsoft.com/azure/security/fundamentals/protection-customer-data

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="48-encrypt-sensitive-information-at-rest"></a>4.8: Cifrado de información confidencial en reposo

**Guía**: Azure Event Hubs admite la opción de cifrado de datos en reposo con claves administradas por Microsoft o claves administradas por el cliente. Esta característica permite crear, rotar, deshabilitar y revocar el acceso a las claves administradas por el cliente que se usan para cifrar datos en reposo de Azure Event Hubs.

Procedimiento para configurar claves administradas por el cliente para el cifrado de Azure Event Hubs: https://docs.microsoft.com/azure/event-hubs/configure-customer-managed-key

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="49-log-and-alert-on-changes-to-critical-azure-resources"></a>4.9: Registro y alerta de cambios en los recursos críticos de Azure

**Guía**: Use Azure Monitor con el registro de actividad de Azure para crear alertas para cuando se produzcan cambios en las instancias de producción de Azure Event Hubs y otros recursos críticos o relacionados.

Procedimiento para crear alertas para eventos de registro de actividad de Azure: https://docs.microsoft.com/azure/azure-monitor/platform/alerts-activity-log

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

## <a name="vulnerability-management"></a>Administración de vulnerabilidades

*Para obtener más información, consulte [Control de seguridad: administración de vulnerabilidades](../security/benchmarks/security-control-vulnerability-management.md).*

### <a name="51-run-automated-vulnerability-scanning-tools"></a>5.1: Ejecute herramientas de análisis de vulnerabilidades automatizado

**Instrucciones**: No aplicable; Microsoft realiza la administración de vulnerabilidades en los sistemas subyacentes que admiten Event Hubs.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Microsoft

### <a name="52-deploy-automated-operating-system-patch-management-solution"></a>5.2: Implemente una solución de administración de revisiones de sistema operativo automatizada

**Guía**: No aplicable; Microsoft realiza la administración de revisiones en los sistemas subyacentes que admiten Event Hubs.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Microsoft

### <a name="53-deploy-automated-third-party-software-patch-management-solution"></a>5.3: Implemente una solución automatizada de administración de revisiones de software de terceros

**Guía**: No aplicable; la prueba comparativa está pensada para recursos de proceso.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Microsoft

### <a name="54-compare-back-to-back-vulnerability-scans"></a>5.4: Compare los exámenes de vulnerabilidades opuestos

**Instrucciones**: No aplicable; Microsoft realiza la administración de vulnerabilidades en los sistemas subyacentes que admiten Event Hubs.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Microsoft

### <a name="55-use-a-risk-rating-process-to-prioritize-the-remediation-of-discovered-vulnerabilities"></a>5.5: Use un proceso de clasificación de riesgos para priorizar la corrección de las vulnerabilidades detectadas

**Instrucciones**: No aplicable; Microsoft realiza la administración de vulnerabilidades en los sistemas subyacentes que admiten Event Hubs.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Microsoft

## <a name="inventory-and-asset-management"></a>Administración de recursos y del inventario

*Para obtener más información, consulte [Control de seguridad: inventario y administración de recursos](../security/benchmarks/security-control-inventory-asset-management.md).*

### <a name="61-use-azure-asset-discovery"></a>6.1: Uso de la detección de recursos de Azure

**Guía**: Use Azure Resource Graph para consultar y detectar todos los recursos (incluidos los espacios de nombres de Azure Event Hubs) dentro de las suscripciones. Asegúrese de que tiene los permisos adecuados (lectura) en el inquilino y de que puede enumerar todas las suscripciones de Azure, así como los recursos de las suscripciones.

Creación de consultas con Azure Resource Graph: https://docs.microsoft.com/azure/governance/resource-graph/first-query-portal

Visualización de las suscripciones de Azure: https://docs.microsoft.com/powershell/module/az.accounts/get-azsubscription?view=azps-3.0.0

Descripción de Azure RBAC: https://docs.microsoft.com/azure/role-based-access-control/overview

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="62-maintain-asset-metadata"></a>6.2: Mantenimiento de metadatos de recursos

**Instrucciones**: Aplique etiquetas a los recursos de Azure que proporcionan metadatos para organizarlos de forma lógica en una taxonomía.

Creación y uso de etiquetas: https://docs.microsoft.com/azure/azure-resource-manager/resource-group-using-tags

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="63-delete-unauthorized-azure-resources"></a>6.3: Eliminación de recursos de Azure no autorizados

**Guía**: Use el etiquetado, los grupos de administración y las suscripciones independientes, si procede, para organizar y realizar un seguimiento de los espacios de nombres de Azure Event Hubs y los recursos relacionados. Concilie el inventario periódicamente y asegúrese de que los recursos no autorizados se eliminan de la suscripción de manera oportuna.

Creación de suscripciones adicionales de Azure: https://docs.microsoft.com/azure/billing/billing-create-subscription

Creación de grupos de administración: https://docs.microsoft.com/azure/governance/management-groups/create

Creación y uso de etiquetas: https://docs.microsoft.com/azure/azure-resource-manager/resource-group-using-tags

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="64-maintain-an-inventory-of-approved-azure-resources-and-software-titles"></a>6.4: Mantenimiento de un inventario de los recursos de Azure aprobados y títulos de software

**Guía**: No aplicable; esta recomendación está destinada a los recursos de proceso y a Azure en su conjunto.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="65-monitor-for-unapproved-azure-resources"></a>6.5: Supervisión de recursos de Azure no aprobados

**Instrucciones**: use Azure Policy para establecer restricciones sobre el tipo de recursos que se pueden crear en las suscripciones del cliente con las siguientes definiciones de directiva integradas:

- Tipos de recursos no permitidos

- Tipos de recursos permitidos

Además, use Azure Resource Graph para consultar o detectar recursos dentro de las suscripciones.

Cómo configurar y administrar Azure Policy: https://docs.microsoft.com/azure/governance/policy/tutorials/create-and-manage

Creación de consultas con Azure Graph: https://docs.microsoft.com/azure/governance/resource-graph/first-query-portal

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="66-monitor-for-unapproved-software-applications-within-compute-resources"></a>6.6: Supervisión de aplicaciones de software no aprobadas en recursos de proceso

**Instrucciones**: No aplicable; esta recomendación está pensada para los recursos de proceso.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="67-remove-unapproved-azure-resources-and-software-applications"></a>6.7: Eliminación de aplicaciones de software y recursos de Azure no aprobadas

**Guía**: No aplicable; esta recomendación está destinada a los recursos de proceso y a Azure en su conjunto.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="68-use-only-approved-applications"></a>6.8: Uso exclusivo de aplicaciones aprobadas

**Instrucciones**: No aplicable; esta recomendación está pensada para los recursos de proceso.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="69-use-only-approved-azure-services"></a>6.9: Uso exclusivo de servicios de Azure aprobados

**Instrucciones**: use Azure Policy para establecer restricciones sobre el tipo de recursos que se pueden crear en las suscripciones del cliente con las siguientes definiciones de directiva integradas:

- Tipos de recursos no permitidos

- Tipos de recursos permitidos

Cómo configurar y administrar Azure Policy: https://docs.microsoft.com/azure/governance/policy/tutorials/create-and-manage

Denegación de un tipo de recurso específico con Azure Policy: https://docs.microsoft.com/azure/governance/policy/samples/not-allowed-resource-types

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="610-implement-approved-application-list"></a>6.10: Implementación de la lista de aplicaciones aprobadas

**Instrucciones**: No aplicable; esta recomendación está pensada para los recursos de proceso.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="611-divlimit-users-ability-to-interact-with-azure-resource-manager-via-scriptsdiv"></a>6.11: <div>Limitación de la capacidad de los usuarios para interactuar con Azure Resource Manager a través de scripts</div>

**Guía**: Configure el acceso condicional de Azure para limitar la capacidad de los usuarios de interactuar con Azure Resource Manager configurando "Bloquear acceso" en la aplicación Microsoft Azure Management.

Configuración del acceso condicional para bloquear el acceso a Azure Resource Manager: https://docs.microsoft.com/azure/role-based-access-control/conditional-access-azure-management

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="612-limit-users-ability-to-execute-scripts-within-compute-resources"></a>6.12: Limitación de capacidad de los usuarios para ejecutar scripts en recursos de proceso

**Instrucciones**: No aplicable; esta recomendación está pensada para los recursos de proceso.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="613-physically-or-logically-segregate-high-risk-applications"></a>6.13: Segregación física o lógica de aplicaciones de alto riesgo

**Instrucciones**: No aplicable; esta recomendación está pensada para las aplicaciones web que se ejecutan en Azure App Service o en recursos de proceso.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

## <a name="secure-configuration"></a>Configuración segura

*Para obtener más información, consulte [Control de seguridad: configuración segura](../security/benchmarks/security-control-secure-configuration.md).*

### <a name="71-establish-secure-configurations-for-all-azure-resources"></a>7.1: Establezca configuraciones seguras para todos los recursos de Azure

**Guía**: Defina e implemente configuraciones de seguridad estándar para las implementaciones de Azure Event Hubs. Use alias de Azure Policy en el espacio de nombres "Microsoft.EventHub" para crear directivas personalizadas con el fin de auditar o aplicar las configuraciones. También puede usar definiciones de directivas integradas para Azure Event Hubs, como:

- Los registros de diagnóstico del centro de eventos deben estar habilitados

- El centro de eventos debe usar un punto de conexión del servicio de red virtual

Directiva integrada de Azure para espacio de nombres de Event Hubs: https://docs.microsoft.com/azure/governance/policy/samples/built-in-policies#event-hub

Visualización de los alias de Azure Policy disponibles: https://docs.microsoft.com/powershell/module/az.resources/get-azpolicyalias?view=azps-3.3.0

Cómo configurar y administrar Azure Policy: https://docs.microsoft.com/azure/governance/policy/tutorials/create-and-manage

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="72-establish-secure-operating-system-configurations"></a>7.2: Establezca configuraciones del sistema operativo seguras

**Instrucciones**: No aplicable; esta recomendación está pensada para los recursos de proceso.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="73-maintain-secure-azure-resource-configurations"></a>7.3: Mantenga configuraciones de recursos de Azure seguras

**Guía**: Use la directiva de Azure Policy [deny] y [deployifnotexist] para aplicar la configuración segura en los recursos habilitados para Event Hubs. 

Configuración y administración de Azure Policy: https://docs.microsoft.com/azure/governance/policy/tutorials/create-and-manage

 
Para obtener más información, consulte los efectos de Azure Policy: https://docs.microsoft.com/azure/governance/policy/concepts/effects

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="74-maintain-secure-operating-system-configurations"></a>7.4: Mantenga configuraciones del sistema operativo seguras

**Instrucciones**: No aplicable; esta recomendación está pensada para los recursos de proceso.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="75-securely-store-configuration-of-azure-resources"></a>7.5: Almacene de forma segura la configuración de los recursos de Azure

**Guía**: Si usa definiciones personalizadas de Azure Policy para los recursos de Event Hubs u otros relacionados, use Azure Repos para almacenar y administrar el código de forma segura.

Cómo almacenar código en Azure DevOps: https://docs.microsoft.com/azure/devops/repos/git/gitworkflow?view=azure-devops

Documentación de Azure Repos: https://docs.microsoft.com/azure/devops/repos/index?view=azure-devops

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="76-securely-store-custom-operating-system-images"></a>7.6: Almacene imágenes de sistema operativo personalizadas de forma segura

**Instrucciones**: No aplicable; esta recomendación está pensada para los recursos de proceso.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="77-deploy-system-configuration-management-tools"></a>7.7: Implemente herramientas de administración de configuración del sistema

**Guía**: Use alias de Azure Policy en el espacio de nombres "Microsoft.EventHub" para crear directivas personalizadas con el fin de auditar y aplicar las configuraciones del sistema y enviar alertas sobre ellas. Además, desarrolle un proceso y una canalización para administrar las excepciones de las directivas.

Cómo configurar y administrar Azure Policy: https://docs.microsoft.com/azure/governance/policy/tutorials/create-and-manage

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="78-deploy-system-configuration-management-tools-for-operating-systems"></a>7.8: Implemente herramientas de administración de configuración del sistema para sistemas operativos

**Instrucciones**: No aplicable; esta recomendación está pensada para los recursos de proceso.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="79-implement-automated-configuration-monitoring-for-azure-services"></a>7.9: Implemente la supervisión de configuración automatizada para servicios de Azure

**Guía**: Use alias de Azure Policy en el espacio de nombres "Microsoft.EventHub" para crear directivas personalizadas con el fin de auditar y aplicar las configuraciones del sistema y enviar alertas sobre ellas. Use la directiva de Azure Policy [audit], [deny] y [deployifnotexist] para aplicar automáticamente las configuraciones en las implementaciones de Azure Event Hubs y los recursos relacionados.

Cómo configurar y administrar Azure Policy: https://docs.microsoft.com/azure/governance/policy/tutorials/create-and-manage

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="710-implement-automated-configuration-monitoring-for-operating-systems"></a>7.10: Implemente la supervisión de configuración automatizada para sistemas operativos

**Instrucciones**: No aplicable; esta recomendación está pensada para los recursos de proceso.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="711-manage-azure-secrets-securely"></a>7.11: Administre los secretos de Azure de forma segura

**Guía**: En el caso de las máquinas virtuales de Azure o las aplicaciones web que se ejecutan en Azure App Service y se usan para tener acceso a los centros de eventos, use Managed Service Identity junto con Azure Key Vault para simplificar y proteger la administración de firmas de acceso compartido para sus implementaciones de Azure Event Hubs. Asegúrese de que la eliminación temporal de Key Vault esté habilitada.

Autenticación de una identidad administrada con Azure Active Directory para acceder a recursos de Event Hubs: https://docs.microsoft.com/azure/event-hubs/authenticate-managed-identity?tabs=latest

Configuración de claves administradas por el cliente para Event Hubs: https://docs.microsoft.com/azure/event-hubs/configure-customer-managed-key 

Cómo integrar las identidades administradas de Azure: https://docs.microsoft.com/azure/azure-app-configuration/howto-integrate-azure-managed-service-identity

Cómo crear un almacén de claves: https://docs.microsoft.com/azure/key-vault/general/quick-create-portal

Autenticación en Azure Key Vault: https://docs.microsoft.com/azure/key-vault/general/authentication

Asignación de una directiva de acceso de Key Vault: https://docs.microsoft.com/azure/key-vault/general/assign-access-policy-portal

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="712-manage-identities-securely-and-automatically"></a>7.12: Administre las identidades de forma segura y automática

**Guía**: En el caso de las máquinas virtuales de Azure o las aplicaciones web que se ejecutan en Azure App Service y se usan para tener acceso a los centros de eventos, use Managed Service Identity junto con Azure Key Vault para simplificar y proteger Event Hubs de Azure. Asegúrese de que la eliminación temporal de Key Vault esté habilitada.

Use Identidades administradas para proporcionar a los servicios de Azure una identidad administrada automáticamente en Azure Active Directory (AD). Identidades administradas le permite autenticarse en cualquier servicio que admita la autenticación de Azure AD, incluido Azure Key Vault, sin necesidad de credenciales en el código.

Autenticación de una identidad administrada con Azure Active Directory para acceder a recursos de Event Hubs: https://docs.microsoft.com/azure/event-hubs/authenticate-managed-identity?tabs=latest 

Configuración de claves administradas por el cliente para Event Hubs: https://docs.microsoft.com/azure/event-hubs/configure-customer-managed-key 

Cómo configurar las identidades administradas: https://docs.microsoft.com/azure/active-directory/managed-identities-azure-resources/qs-configure-portal-windows-vm

Cómo integrar las identidades administradas de Azure: https://docs.microsoft.com/azure/azure-app-configuration/howto-integrate-azure-managed-service-identity

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="713-eliminate-unintended-credential-exposure"></a>7.13: Elimine la exposición de credenciales no intencionada

**Instrucciones**: Implemente el escáner de credenciales para identificar las credenciales en el código. El escáner de credenciales también fomenta el traslado de credenciales detectadas a ubicaciones más seguras, como Azure Key Vault.

Cómo configurar el escáner de credenciales: https://secdevtools.azurewebsites.net/helpcredscan.html

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

## <a name="malware-defense"></a>Defensa contra malware

*Para obtener más información, consulte [Control de seguridad: defensa contra malware](../security/benchmarks/security-control-malware-defense.md).*

### <a name="81-use-centrally-managed-anti-malware-software"></a>8.1: Use software antimalware administrado centralmente

**Instrucciones**: No aplicable; esta recomendación está pensada para los recursos de proceso.

Microsoft Antimalware está habilitado en el host subyacente que admite los servicios de Azure (por ejemplo, Azure App Service), pero no se ejecuta en el contenido del cliente.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="82-pre-scan-files-to-be-uploaded-to-non-compute-azure-resources"></a>8.2: Examine previamente los archivos que se van a cargar en recursos de Azure que no son de proceso

**Guía**: Examine previamente el contenido que se carga en recursos de Azure que no son de proceso, como Azure Event Hubs, App Service, Data Lake Storage, Blob Storage, Azure Database for PostgreSQL, etc. Microsoft no tiene acceso a los datos de estas instancias.

Microsoft Antimalware está habilitado en el host subyacente que admite los servicios de Azure (por ejemplo, Azure Cache for Redis), pero no se ejecuta en el contenido del cliente.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="83-ensure-anti-malware-software-and-signatures-are-updated"></a>8.3: Asegúrese de que se han actualizado el software y las firmas antimalware

**Instrucciones**: No aplicable; esta recomendación está pensada para los recursos de proceso.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

## <a name="data-recovery"></a>Recuperación de datos

*Para obtener más información, consulte [Control de seguridad: recuperación de datos](../security/benchmarks/security-control-data-recovery.md).*

### <a name="91-ensure-regular-automated-back-ups"></a>9.1: Garantía de copias de seguridad automáticas periódicas

**Instrucciones**: Configure la recuperación ante desastres geográfica para Azure Event Hubs. Cuando hay regiones de Azure completas o centros de datos (si no se utilizan zonas de disponibilidad) que experimentan un tiempo de inactividad, es crucial que el procesamiento de datos siga funcionando en otra región o centro de datos. De esta forma, la recuperación ante desastres geográfica y la replicación geográfica son características importantes para cualquier empresa. Azure Event Hubs admite tanto la recuperación ante desastres con localización geográfica como la replicación geográfica, en el nivel de espacio de nombres. 

Introducción a la recuperación ante desastres geográfica para Azure Event Hubs: https://docs.microsoft.com/azure/event-hubs/event-hubs-geo-dr#availability-zones

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="92-perform-complete-system-backups-and-backup-any-customer-managed-keys"></a>9.2: Realización de copias de seguridad completas del sistema y copia de seguridad de las claves administradas por el cliente

**Guía**: Azure Event Hubs proporciona cifrado de datos en reposo con Azure Storage Service Encryption (Azure SSE). Event Hubs se basa en Azure Storage para almacenar los datos y, de forma predeterminada, todos los datos que se almacenan con Azure Storage se cifran mediante claves administradas por Microsoft. Si usa Azure Key Vault para almacenar claves administradas por el cliente, asegúrese de realizar copias de seguridad automatizadas periódicas de las claves.

Asegúrese de realizar copias de seguridad automatizadas periódicas de sus secretos de Key Vault con el siguiente comando de PowerShell: Backup-AzKeyVaultSecret

Procedimiento para configurar claves administradas por el cliente para el cifrado de datos de Azure Event Hubs en reposo: https://docs.microsoft.com/azure/event-hubs/configure-customer-managed-key

Realización de copias de seguridad de secretos de Key Vault: https://docs.microsoft.com/powershell/module/azurerm.keyvault/backup-azurekeyvaultsecret

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="93-validate-all-backups-including-customer-managed-keys"></a>9.3: Validación de todas las copias de seguridad, incluidas las claves administradas por el cliente

**Guía**: Pruebe la restauración de las claves administradas por el cliente de la copia de seguridad.

 

Restauración de las claves del almacén de claves en Azure: https://docs.microsoft.com/powershell/module/azurerm.keyvault/restore-azurekeyvaultkey?view=azurermps-6.13.0

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="94-ensure-protection-of-backups-and-customer-managed-keys"></a>9.4: Garantía de la protección de las copias de seguridad y las claves administradas del cliente

**Guía**: Habilite la eliminación temporal en Key Vault para proteger las claves contra la eliminación accidental o malintencionada. Azure Event Hubs requiere que las claves administradas por el cliente tengan configurada Eliminación temporal y No purgar.

Configure la eliminación temporal para la cuenta de Azure Storage que se usa para capturar datos de Event Hubs. Tenga en cuenta que esta característica aún no se admite para Azure Data Lake Storage Gen 2.

Habilitación de la eliminación temporal en Key Vault: https://docs.microsoft.com/azure/storage/blobs/storage-blob-soft-delete?tabs=azure-portal

Configuración de un almacén de claves con claves: https://docs.microsoft.com/azure/event-hubs/configure-customer-managed-key

Eliminación temporal de blobs de Azure Storage: https://docs.microsoft.com//azure/storage/blobs/storage-blob-soft-delete?tabs=azure-portal

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

## <a name="incident-response"></a>Respuesta a los incidentes

*Para obtener más información, consulte [Control de seguridad: respuesta ante incidentes](../security/benchmarks/security-control-incident-response.md).*

### <a name="101-create-an-incident-response-guide"></a>10.1: Creación de una guía de respuesta ante incidentes

**Instrucciones**: asegúrese de que haya planes de respuesta ante incidentes escritos que definan los roles del personal, así como las fases de administración y gestión de los incidentes.

Configuración de las automatizaciones del flujo de trabajo en Azure Security Center: https://docs.microsoft.com/azure/security-center/security-center-planning-and-operations-guide



**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="102-create-an-incident-scoring-and-prioritization-procedure"></a>10.2: Creación de un procedimiento de priorización y puntuación de incidentes

**Instrucciones**: Security Center asigna una gravedad a las alertas, que le ayudan a priorizar el orden en el que asiste a cada alerta, de modo que cuando un recurso está en peligro, puede obtenerlo inmediatamente. La gravedad se basa en la confianza que tiene Security Center en la búsqueda o en el análisis utilizados para emitir la alerta, así como en el nivel de confianza de que ha habido un intento malintencionado detrás de la actividad que ha provocado la alerta.

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="103-test-security-response-procedures"></a>10.3: Prueba de los procedimientos de respuesta de seguridad

**Guía**: Realice ejercicios para probar las capacidades de respuesta a los incidentes de los sistemas con regularidad. Identifique puntos débiles y brechas y revise el plan según sea necesario.

Consulte la publicación de NIST: Guía para probar, entrenar y ejecutar programas para planes y capacidades de TI: https://nvlpubs.nist.gov/nistpubs/Legacy/SP/nistspecialpublication800-84.pdf

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="104-provide-security-incident-contact-details-and-configure-alert-notifications-for-security-incidents"></a>10.4: Provisión de detalles de contacto de incidentes de seguridad y configuración de notificaciones de alerta para incidentes de seguridad

**Guía**: La información de contacto del incidente de seguridad la utilizará Microsoft para ponerse en contacto con usted si Microsoft Security Response Center (MSRC) detecta que un tercero no autorizado o ilegal ha accedido a los datos del cliente.  Revise los incidentes después del hecho para asegurarse de que se resuelven los problemas. 

Cómo establecer el contacto de seguridad de Azure Security Center: https://docs.microsoft.com/azure/security-center/security-center-provide-security-contact-details

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="105-incorporate-security-alerts-into-your-incident-response-system"></a>10.5: Incorporación de alertas de seguridad en el sistema de respuesta a incidentes

**Instrucciones**: Exporte sus alertas y recomendaciones de Azure Security Center mediante la característica de exportación continua. La exportación continua le permite exportar alertas y recomendaciones de forma manual o continua. Puede usar el conector de datos de Azure Security Center para transmitir las alertas a Sentinel.

Configuración de la exportación continua: https://docs.microsoft.com/azure/security-center/continuous-export

Transmisión de alertas a Azure Sentinel: https://docs.microsoft.com/azure/sentinel/connect-azure-security-center

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="106-automate-the-response-to-security-alerts"></a>10.6: Automatización de la respuesta a las alertas de seguridad

**Instrucciones**: use la característica de automatización del flujo de trabajo de Azure Security Center para desencadenar automáticamente respuestas mediante "Logic Apps" en las alertas y recomendaciones de seguridad.

Configuración de la automatización de flujo de trabajo y Logic Apps: https://docs.microsoft.com/azure/security-center/workflow-automation

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

## <a name="penetration-tests-and-red-team-exercises"></a>Pruebas de penetración y ejercicios del equipo rojo

*Para obtener más información, consulte [Control de seguridad: Pruebas de penetración y ejercicios del equipo rojo](../security/benchmarks/security-control-penetration-tests-red-team-exercises.md).*

### <a name="111-conduct-regular-penetration-testing-of-your-azure-resources-and-ensure-remediation-of-all-critical-security-findings-within-60-days"></a>11.1: Realice pruebas de penetración periódicas de los recursos de Azure y asegúrese de corregir todos los resultados de seguridad críticos en un plazo de 60 días

**Guía**: siga las reglas de compromiso de Microsoft para asegurarse de que las pruebas de penetración no infrinjan las directivas de Microsoft: https://www.microsoft.com/msrc/pentest-rules-of-engagement?rtc=1.
Puede encontrar más información sobre la estrategia de Microsoft y la ejecución de las pruebas de penetración del equipo rojo y sitios activos en la infraestructura de nube, los servicios y las aplicaciones administradas por Microsoft aquí: https://gallery.technet.microsoft.com/Cloud-Red-Teaming-b837392e

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

## <a name="next-steps"></a>Pasos siguientes

- Consulte la [prueba comparativa de seguridad de Azure](../security/benchmarks/overview.md).
- Obtenga más información sobre las [líneas de base de seguridad de Azure](../security/benchmarks/security-baselines-overview.md).
